# 01 - البنية التحتية والتسجيلات الأساسية (Program.cs & App.razor)

**آخر تحديث:** 14 يوليو 2026  
**الإصدار:** 1.3

---

## مقدمة

هذا المرجع يغطي قلب النظام: ملف `Program.cs` وملف `App.razor`. أي خطأ في هذه الملفات يؤثر على كل صفحة في التطبيق. اقرأه بتركيز قبل البدء بأي تطوير جديد.

---

## 1. ملف Program.cs - مصدر الحقيقة الوحيد للتسجيلات

### الغرض

هذا الملف هو المصدر الوحيد لكل الخدمات والإعدادات في النظام. أي خدمة تريد استخدامها في صفحتك يجب أن تكون مسجلة هنا.

### 1.1 منطقة الأساسيات ودعم SSR

```csharp
builder.Services.AddRazorComponents()
    .AddInteractiveServerComponents();
```

### 1.2 منطقة إدارة حالة المصادقة

```csharp
builder.Services.AddCascadingAuthenticationState();
builder.Services.AddScoped<IdentityRedirectManager>();
builder.Services.AddScoped<AuthenticationStateProvider, IdentityRevalidatingAuthenticationStateProvider>();

// تسجيل الدخول عبر Google
builder.Services.AddAuthentication()
    .AddGoogle(options =>
    {
        options.ClientId = googleClientId;
        options.ClientSecret = googleClientSecret;
    });
```

⚠️ **تحذير:** تأكد من وجود `ClientId` و `ClientSecret` في `appsettings.json`.

### 1.3 منطقة قاعدة البيانات (الأهم والأكثر خطأً)

```csharp
var connectionString = builder.Configuration.GetConnectionString("DefaultConnection");

// التسجيل القديم (للصفحات القديمة فقط)
builder.Services.AddDbContext<BusinessDbContext>(options =>
    options.UseSqlServer(connectionString),
    ServiceLifetime.Scoped);

// التسجيل الحديث والصحيح (للصفحات الجديدة)
builder.Services.AddDbContextFactory<BusinessDbContext>(options =>
    options.UseSqlServer(connectionString),
    ServiceLifetime.Scoped); // ⭐ هذا ما كان ناقصاً ويسبب أخطاء
```

⚠️ **تحذير حرج:**
- تأكد من وجود `ServiceLifetime.Scoped` في `AddDbContextFactory`
- لا تحذف `AddDbContext` إلا بعد ترحيل كل الصفحات القديمة
- `AddDbContextFactory` هو الموصى به للصفحات الجديدة لمنع مشكلة "A second operation started"

### 1.4 منطقة الهوية (Identity)

```csharp
builder.Services.AddIdentity<ApplicationUser, IdentityRole>(options =>
{
    options.Password.RequireDigit = false;
    options.Password.RequiredLength = 6;
    options.SignIn.RequireConfirmedAccount = false;
})
.AddEntityFrameworkStores<BusinessDbContext>()
.AddDefaultTokenProviders();

builder.Services.AddScoped<IGenericService<UserProfile>, GenericService<UserProfile>>();
builder.Services.AddScoped<IUserRoleService, UserRoleService>();
```

### 1.5 منطقة خدمات النظام الإضافية

```csharp
// الأداء
builder.Services.AddMemoryCache(); // ⭐ ضروري جداً للترجمة والقوائم

// حماية البيانات
builder.Services.AddDataProtection()
    .PersistKeysToFileSystem(new DirectoryInfo(Path.Combine(builder.Environment.ContentRootPath, "DataProtection-Keys")))
    .SetApplicationName("RubikCare");

// الخدمة الأساسية (CRUD)
builder.Services.AddScoped(typeof(IGenericService<>), typeof(GenericService<>));

// خدمات السياق والمساعدة (الأكثر استخداماً)
builder.Services.AddScoped<DbContextFactoryService>();
builder.Services.AddScoped<UserContextService>();
builder.Services.AddScoped<DynamicMenuService>();
builder.Services.AddScoped<IUserSessionService, UserSessionService>();
builder.Services.AddHttpContextAccessor();

// نظام الترجمة
builder.Services.AddScoped<LocalizationService>();
builder.Services.AddScoped<ILocalizationService, LocalizationService>();
builder.Services.AddScoped<TranslationStateService>();
builder.Services.AddScoped<ILayoutDirectionService, LayoutDirectionService>();

// خدمات Excel والصور
builder.Services.AddScoped<ExcelService>();
builder.Services.AddScoped<IImageService, ImageService>();
```

### 1.6 إعدادات Middleware (الترتيب مهم جداً)

```csharp
var app = builder.Build();

if (app.Environment.IsDevelopment())
{
    app.UseMigrationsEndPoint();
}
else
{
    app.UseExceptionHandler("/Error", createScopeForErrors: true);
    app.UseHsts();
}

app.UseStaticFiles(new StaticFileOptions
{
    FileProvider = new PhysicalFileProvider(
        Path.Combine(Directory.GetCurrentDirectory(), "wwwroot", "uploads")),
    RequestPath = "/uploads"
});

// ⭐ ترتيب Middleware إلزامي - لا تغيره أبداً
app.UseHttpsRedirection();
app.UseStaticFiles();
app.UseRouting();
app.UseAuthentication();    // ⭐ قبل Authorization
app.UseAuthorization();
app.UseAntiforgery();

// ضبط الثقافة الافتراضية
var defaultCulture = new CultureInfo("en");
CultureInfo.DefaultThreadCurrentCulture = defaultCulture;
CultureInfo.DefaultThreadCurrentUICulture = defaultCulture;

// ربط نقاط النهاية
app.MapAdditionalIdentityEndpoints();
app.MapRazorComponents<App>()
    .AddInteractiveServerRenderMode();

// API مخصص للترجمة
app.MapGet("/api/localization/page/{page}", async (
    string page,
    string lang,
    [FromServices] ILocalizationService locService) =>
{
    var translations = await locService.GetPageTranslationsAsync(page, lang);
    return Results.Ok(translations);
});

app.Run();
```

⚠️ **ترتيب Middleware إلزامي:** يجب أن يكون `UseAuthentication` قبل `UseAuthorization`. أي تبديل يؤدي إلى فشل المصادقة.

---

## 2. ملف App.razor - تدفق التطبيق

### الكود الكامل (المحدث - يوليو 2026)

```razor
<!DOCTYPE html>
@using MudBlazor
<html>
<head>
    <meta charset="utf-8" />
    <meta name="viewport" content="width=device-width, initial-scale=1.0" />
    <base href="/" />
    
    <link rel="stylesheet" href="https://cdn.jsdelivr.net/npm/bootstrap@5.3.2/dist/css/bootstrap.min.css" />
    <link href="_content/MudBlazor/MudBlazor.min.css" rel="stylesheet" />
    <link href="_content/RubikCare.Shared.UI/css/main.css" rel="stylesheet" />
    
    <HeadOutlet />
</head>
<body>
    <Routes />
    <ReconnectModal />
    
    <MudPopoverProvider />
    <MudThemeProvider />
    <MudDialogProvider />
    <MudSnackbarProvider />
    
    <script src="_framework/blazor.web.js"></script>
</body>
</html>

@code {
    // ⭐ استخدام الواجهة (Interface) وليس الكلاس الملموس
    [Inject] private RubikCare.Application.Services.TranslationStateService TranslationState { get; set; } = default!;
    [Inject] private RubikCare.Application.Interfaces.ILayoutDirectionService LayoutDirection { get; set; } = default!;

    protected override async Task OnInitializedAsync()
    {
        // ⭐ مهم جداً: تهيئة اللغة والاتجاه قبل أي شيء لمنع وميض SSR
        await TranslationState.InitializeAsync();
        await LayoutDirection.EnsureInitializedAsync();
    }
}
```

### النقاط الحرجة

1. **التهيئة المبكرة للغة:** بدون استدعاء `TranslationState.InitializeAsync()` هنا، ستعرض الصفحة الأولى اللغة الافتراضية ثم "تومض" عند تحميل اللغة الحقيقية.

2. **استخدام الواجهة (Interface):** استخدم `ILayoutDirectionService` بدلاً من `LayoutDirectionService` لأن الخدمة مسجلة كواجهة في `Program.cs`.

3. **التوجيه الموحد:** `DefaultLayout="@typeof(MainLayout)"` يضمن أن كل الصفحات ترث التخطيط نفسه.

---

## 3. ⭐ مشاكل SSR الشائعة وحلولها (جديد - يوليو 2026)

### المشكلة 1: وميض اللغة عند التحميل الأولي

**الأعراض:**
- الصفحة تظهر بالإنجليزية ثم تتحول للعربية فجأة
- المستخدم يرى وميضاً مزعجاً عند كل تحميل

**السبب:**
عدم تهيئة اللغة في `App.razor` قبل بدء الـ Rendering.

**الحل:**
```razor
@code {
    [Inject] private TranslationStateService TranslationState { get; set; } = default!;
    
    protected override async Task OnInitializedAsync()
    {
        await TranslationState.InitializeAsync();
    }
}
```

---

### المشكلة 2: خطأ "JavaScript interop calls cannot be issued"

**الأعراض:**
```
System.InvalidOperationException: JavaScript interop calls cannot be issued at this time. 
This is because the component is being statically rendered.
```

**السبب:**
استدعاء `IJSRuntime` في `OnInitializedAsync` (مرحلة SSR قبل اتصال SignalR).

**الحل:**
استخدام `OnAfterRenderAsync` لأي استدعاء JavaScript:

```csharp
protected override async Task OnAfterRenderAsync(bool firstRender)
{
    if (firstRender)
    {
        // ✅ آمن: المتصفح متصل الآن
        await JSRuntime.InvokeVoidAsync("localStorage.setItem", "key", "value");
    }
}
```

**قاعدة ذهبية:**
- `OnInitializedAsync` → لا تستخدم `IJSRuntime`
- `OnAfterRenderAsync` → آمن لاستخدام `IJSRuntime`

---

### المشكلة 3: خطأ CS0120 - "An object reference is required"

**الأعراض:**
```
Error CS0120: An object reference is required for the non-static field, method, 
or property 'JSRuntimeExtensions.InvokeVoidAsync(...)'
```

**السبب:**
استخدام صيغة خاطئة للحقن خارج كتلة `@code`:

```razor
❌ خطأ:
[Inject] IJSRuntime JSRuntime { get; set; }

✅ صحيح:
@inject IJSRuntime JSRuntime
```

**الحل:**
استخدام `@inject` في أعلى الملف (خارج `@code`):

```razor
@inject IJSRuntime JSRuntime
@inject NavigationManager Navigation

@code {
    // الكود هنا
}
```

---

### المشكلة 4: خطأ CS1061 - "does not contain a definition for"

**الأعراض:**
```
Error CS1061: 'ILayoutDirectionService' does not contain a definition for 
'EnsureInitializedAsync'
```

**السبب:**
الدالة موجودة في الكلاس الملموس لكنها غير معرّفة في الواجهة.

**الحل:**
إضافة تعريف الدالة في الواجهة:

```csharp
// ILayoutDirectionService.cs
public interface ILayoutDirectionService
{
    string CurrentDirection { get; }
    Task<string> GetDirectionAsync();
    Task<string> GetEffectiveDirectionAsync();
    Task SetDirectionAsync(string direction);
    
    // ⭐ هذا السطر المفقود
    Task EnsureInitializedAsync();
    
    event Action? OnDirectionChanged;
}
```

---

### المشكلة 5: انتهاك Clean Architecture - استخدام IHttpContextAccessor في Application Layer

**الأعراض:**
```
Error CS0246: The type or namespace name 'IHttpContextAccessor' could not be found
```

**السبب:**
محاولة استخدام `IHttpContextAccessor` في طبقة `Application`، وهذا انتهاك لقاعدة Clean Architecture.

**القاعدة:**
طبقة `Application` يجب ألا تعرف شيئاً عن بيئة التشغيل (Web/Mobile).

**الحل:**
استخدام `CultureInfo.CurrentCulture` بدلاً من `IHttpContextAccessor`:

```csharp
// ❌ خطأ (Application Layer)
public class TranslationStateService
{
    private readonly IHttpContextAccessor _httpContextAccessor; // ⚠️ انتهاك!
}

// ✅ صحيح (Application Layer)
public class TranslationStateService
{
    public async Task InitializeAsync(bool isInteractive = false)
    {
        if (!isInteractive)
        {
            // قراءة لغة النظام الحالية (آمن في SSR)
            var serverLang = CultureInfo.CurrentCulture.TwoLetterISOLanguageName;
            _currentLanguage = (serverLang == "ar" || serverLang == "en") ? serverLang : "ar";
        }
    }
}
```

---

## 4. ⭐ أنماط التحميل الأمثل (جديد - يوليو 2026)

### النمط 1: منع التحميل المتكرر

**المشكلة:**
`OnInitializedAsync` يُستدعى عدة مرات → تحميل مكرر للبيانات → استعلامات قاعدة بيانات زائدة.

**الحل:**
استخدام علم (Flag) للتحميل مرة واحدة:

```csharp
private bool _isDataLoaded = false;

protected override async Task OnInitializedAsync()
{
    if (_isDataLoaded) return; // ⭐ منع إعادة التحميل
    
    await LoadDataAsync();
    _isDataLoaded = true; // ⭐ تأكيد اكتمال التحميل
}
```

---

### النمط 2: استعادة الحالة دون فرض التنقل

**المشكلة:**
دالة `RestoreMenuStateAsync` تستدعي `Navigation.NavigateTo` → إعادة توجيه المستخدم للصفحة الرئيسية عند تحديث الصفحة (F5).

**الحل:**
تحديث الحالة الداخلية فقط دون استدعاء `Navigation`:

```csharp
private async Task RestoreMenuStateAsync()
{
    var savedMode = await JSRuntime.InvokeAsync<string>("localStorage.getItem", "mode");
    
    // ✅ تحديث الحالة فقط (بدون Navigation)
    _currentViewMode = savedMode == "Organization" 
        ? MenuViewMode.Organization 
        : MenuViewMode.Personal;
    
    StateHasChanged();
}
```

**متى تستخدم `Navigation.NavigateTo`؟**
- فقط عند النقر الفعلي من المستخدم (في `@onclick`)
- ليس في `OnAfterRenderAsync` أو `RestoreMenuStateAsync`

---

### النمط 3: استخدام forceLoad: true لتبديل الأوضاع

**المشكلة:**
عند التبديل بين الأوضاع (شخصي ↔ مؤسسة)، تتغير القائمة لكن الصفحة لا تتغير.

**السبب:**
Blazor يحتفظ بحالة المكونات القديمة ولا يعيد رسم الصفحة الجديدة بشكل كامل.

**الحل:**
استخدام `forceLoad: true` لإعادة تحميل الصفحة بالكامل:

```csharp
private async Task SwitchToOrganization(OrganizationMembershipSession org)
{
    _currentViewMode = MenuViewMode.Organization;
    _selectedOrganization = org;
    
    StateHasChanged();
    
    var dashboardUrl = GetDashboardUrlByOrgType(org);
    
    // ⭐ استخدام forceLoad: true لضمان تحميل الصفحة الرئيسية للوضع الجديد
    Navigation.NavigateTo(dashboardUrl, forceLoad: true);
}
```

---

### النمط 4: تطبيق الاتجاه بشكل ديناميكي

**المشكلة:**
عند تغيير اللغة، لا يتغير اتجاه الصفحة فوراً.

**الحل:**
استدعاء `ApplyDirectionAsync` في `OnAfterRenderAsync`:

```csharp
protected override async Task OnAfterRenderAsync(bool firstRender)
{
    if (firstRender)
    {
        var savedLang = await JSRuntime.InvokeAsync<string>("localStorage.getItem", "RubikCare:Language");
        
        if (!string.IsNullOrEmpty(savedLang))
        {
            if (TranslationState.CurrentLanguage == savedLang)
            {
                // ✅ اللغة متطابقة، نطبق الاتجاه فقط
                await ApplyDirectionAsync(savedLang);
            }
            else
            {
                // اللغة مختلفة، نحدثها (يحدث مرة واحدة عند أول دخول)
                await TranslationState.SetLanguageAsync(savedLang);
            }
        }
    }
}

private async Task ApplyDirectionAsync(string lang)
{
    var direction = lang == "ar" ? "rtl" : "ltr";
    await JSRuntime.InvokeVoidAsync("eval", $@"
        document.documentElement.setAttribute('dir', '{direction}');
        document.documentElement.setAttribute('lang', '{lang}');
        document.body.setAttribute('dir', '{direction}');
    ");
}
```

---

## 5. مصفوفة الخدمات - مرجع سريع

### خدمات أساسية (ضرورية لكل صفحة)

| الخدمة | الواجهة | الاستخدام |
|--------|---------|-----------|
| `GenericService<T>` | `IGenericService<T>` | لأي عملية CRUD على أي جدول |
| `DbContextFactoryService` | - | لإنشاء سياقات آمنة (للخدمات المخصصة) |
| `UserContextService` | `IUserContextService` | معلومات المستخدم الحالي |

### خدمات الترجمة (لأي صفحة تدعم اللغتين)

| الخدمة | الواجهة | الاستخدام |
|--------|---------|-----------|
| `TranslationStateService` | - | معرفة اللغة الحالية والاشتراك في تغييراتها |
| `ILocalizationService` | `ILocalizationService` | جلب الترجمات من قاعدة البيانات |
| `LayoutDirectionService` | `ILayoutDirectionService` | إدارة اتجاه الصفحة (RTL/LTR) |

### خدمات المستخدم والسياق

| الخدمة | الواجهة | الاستخدام |
|--------|---------|-----------|
| `UserContextService` | `IUserContextService` | ID المستخدم، الأدوار، الصلاحيات |
| `IUserSessionService` | `IUserSessionService` | إدارة الجلسات والكاش |
| `IUserRoleService` | `IUserRoleService` | إدارة الأدوار الموسعة |

### خدمات التنقل والقوائم

| الخدمة | الواجهة | الاستخدام |
|--------|---------|-----------|
| `DynamicMenuService` | - | جلب القوائم الديناميكية للمستخدم |

### خدمات مساعدة

| الخدمة | الواجهة | الاستخدام |
|--------|---------|-----------|
| `ExcelService` | `IExcelService<T>` | استيراد/تصدير Excel |
| `IImageService` | `IImageService` | رفع ومعالجة الصور |

---

## 6. CHECKLIST: قبل كتابة أي صفحة جديدة

### التحقق من التسجيلات

- [ ] هل الخدمة التي سأستخدمها مسجلة في `Program.cs`؟
- [ ] إذا كانت خدمة جديدة، هل أضفتها باستخدام `AddScoped` أو `AddSingleton`؟
- [ ] هل تأكدت من عدم وجود تسجيل مزدوج لنفس الخدمة؟
- [ ] هل استخدمت الواجهة (Interface) للحقن بدلاً من الكلاس الملموس؟

### التحقق من قاعدة البيانات

- [ ] هل النموذج (Model) موجود في مجلد `Data/Models`؟
- [ ] هل تم إضافة `DbSet<MyEntity>` في `BusinessDbContext`؟
- [ ] هل تم إنشاء `Migration` جديدة بعد إضافة النموذج؟

### التحقق من نمط الصفحة

- [ ] هل الصفحة جديدة؟ → استخدم `IGenericService<T>` و `DbContextFactory`
- [ ] هل الصفحة قديمة وتستخدم `@inject BusinessDbContext` مباشرة؟ → خطط لترحيلها

### التحقق من الترجمة

- [ ] هل أضفت مفاتيح الترجمة في قاعدة البيانات مع `N'' prefix` للعربية؟
- [ ] هل تستخدم `T()` المساعدة بدلاً من `<LocalizedText />`؟

### التحقق من SSR والأداء (جديد - يوليو 2026)

- [ ] هل أضفت `@code` block في `App.razor` مع `InitializeAsync()`؟
- [ ] هل تستخدم `OnAfterRenderAsync` لاستدعاءات JavaScript؟
- [ ] هل استخدمت علم `_isDataLoaded` لمنع التحميل المتكرر؟
- [ ] هل تجنب استدعاء `Navigation.NavigateTo` في `OnAfterRenderAsync`؟
- [ ] هل استخدمت `forceLoad: true` عند تبديل الأوضاع؟

---

## 7. تحذيرات ومحاذير

| # | التحذير |
|---|---------|
| 1 | لا تغير ترتيب Middleware - `UseAuthentication` قبل `UseAuthorization` دائماً |
| 2 | لا تنسى `ServiceLifetime.Scoped` في `AddDbContextFactory` |
| 3 | استخدم `DbContextFactory` للصفحات الجديدة - لا تحقن `DbContext` مباشرة |
| 4 | سجل خدماتك قبل `builder.Build()` - أي تسجيل بعد البناء لن يعمل |
| 5 | تأكد من وجود `AddMemoryCache()` - ضروري للترجمة والقوائم |
| 6 | استخدم `@inject` في أعلى الملف وليس `[Inject]` داخل `@code` |
| 7 | لا تستخدم `IJSRuntime` في `OnInitializedAsync` - استخدم `OnAfterRenderAsync` |
| 8 | لا تستخدم `IHttpContextAccessor` في Application Layer - استخدم `CultureInfo` |
| 9 | استخدم الواجهة (Interface) للحقن وليس الكلاس الملموس |
| 10 | لا تستدعي `Navigation.NavigateTo` في `OnAfterRenderAsync` أو `RestoreMenuStateAsync` |

---

## 🔗 روابط ذات صلة

- [00 - الهيكل المعماري](00-architecture-overview.md)
- [02 - نظام الهوية والمصادقة](02-identity-system.md)
- [05 - إنشاء الصفحات والمكونات](05-page-creation-checklist.md)
- [الملحق ب - فهرس الخدمات](../appendix-b-service-index.md)

---

## 📝 سجل التحديثات

| التاريخ | الإصدار | التغييرات |
|---------|---------|-----------|
| 17 مايو 2026 | 1.0 | الإصدار الأولي |
| 24 مايو 2026 | 1.1 | إضافة نظام الكاش والجلسات |
| 14 يوليو 2026 | 1.3 | إضافة مشاكل SSR وحلولها، أنماط التحميل الأمثل، تحذيرات معمارية |
```

---

## ✅ ملخص التحديثات المضافة

1. **قسم جديد: "مشاكل SSR الشائعة وحلولها"** - يغطي 5 مشاكل واجهناها فعلياً
2. **قسم جديد: "أنماط التحميل الأمثل"** - 4 أنماط لتحسين الأداء
3. **تحديث مثال `App.razor`** - ليشمل استخدام الواجهة
4. **تحديث قائمة التحقق (CHECKLIST)** - إضافة فقرة SSR والأداء
5. **إضافة 5 تحذيرات جديدة** - بناءً على الأخطاء التي واجهناها
6. **سجل التحديثات** - لتتبع التغييرات
