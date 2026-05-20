# 01 - البنية التحتية والتسجيلات الأساسية (Program.cs & App.razor)

**آخر تحديث: 17 مايو 2026**

---

## مقدمة

هذا المرجع يغطي **قلب النظام**: ملف `Program.cs` وملف `App.razor`. أي خطأ في هذه الملفات يؤثر على **كل صفحة** في التطبيق. اقرأه بتركيز قبل البدء بأي تطوير جديد.

---

## 1. ملف Program.cs - مصدر الحقيقة الوحيد للتسجيلات

### الغرض

هذا الملف هو **المصدر الوحيد** لكل الخدمات والإعدادات في النظام. أي خدمة تريد استخدامها في صفحتك **يجب** أن تكون مسجلة هنا.

---

### 1.1 منطقة الأساسيات ودعم SSR

```csharp
builder.Services.AddRazorComponents()
    .AddInteractiveServerComponents();
```

---

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

---

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

---

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

---

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

---

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

### الكود الكامل

```razor
<Router AppAssembly="@typeof(App).Assembly">
    <Found Context="routeData">
        <RouteView RouteData="@routeData" DefaultLayout="@typeof(MainLayout)" />
        <FocusOnNavigate RouteData="@routeData" Selector="h1" />
    </Found>
    <NotFound>
        <PageTitle>الصفحة غير موجودة</PageTitle>
        <LayoutView Layout="@typeof(MainLayout)">
            <div class="text-center py-5">
                <h1>404</h1>
                <p>عذراً، الصفحة التي تبحث عنها غير موجودة.</p>
                <NavLink href="/">العودة للرئيسية</NavLink>
            </div>
        </LayoutView>
    </NotFound>
</Router>

@code {
    [Inject] private TranslationStateService TranslationState { get; set; }
    [Inject] private LayoutDirectionService LayoutDirection { get; set; }

    protected override async Task OnInitializedAsync()
    {
        // ⭐ مهم جداً: تهيئة اللغة قبل أي شيء
        await TranslationState.InitializeAsync();
        await LayoutDirection.EnsureInitializedAsync();
    }
}
```

### النقاط الحرجة

1. **التهيئة المبكرة للغة:** بدون استدعاء `TranslationState.InitializeAsync()` هنا، ستعرض الصفحة الأولى اللغة الافتراضية ثم "تومض" عند تحميل اللغة الحقيقية.

2. **التوجيه الموحد:** `DefaultLayout="@typeof(MainLayout)"` يضمن أن كل الصفحات ترث التخطيط نفسه.

---

## 3. مصفوفة الخدمات - مرجع سريع

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
| `IUserSessionService` | `IUserSessionService` | localStorage |
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

## 4. CHECKLIST: قبل كتابة أي صفحة جديدة

### التحقق من التسجيلات
- [ ] هل الخدمة التي سأستخدمها مسجلة في `Program.cs`؟
- [ ] إذا كانت خدمة جديدة، هل أضفتها باستخدام `AddScoped` أو `AddSingleton`؟
- [ ] هل تأكدت من عدم وجود تسجيل مزدوج لنفس الخدمة؟

### التحقق من قاعدة البيانات
- [ ] هل النموذج (Model) موجود في مجلد `Data/Models`؟
- [ ] هل تم إضافة `DbSet<MyEntity>` في `BusinessDbContext`؟
- [ ] هل تم إنشاء `Migration` جديدة بعد إضافة النموذج؟

### التحقق من نمط الصفحة
- [ ] هل الصفحة **جديدة**؟ → استخدم `IGenericService<T>` و `DbContextFactory`
- [ ] هل الصفحة **قديمة** وتستخدم `@inject BusinessDbContext` مباشرة؟ → خطط لترحيلها

### التحقق من الترجمة
- [ ] هل أضفت مفاتيح الترجمة في قاعدة البيانات مع `N'' prefix` للعربية؟
- [ ] هل تستخدم `T()` المساعدة بدلاً من `<LocalizedText />`؟

---

## 5. تحذيرات ومحاذير

| # | التحذير |
|---|---------|
| 1 | **لا تغير ترتيب Middleware** - `UseAuthentication` قبل `UseAuthorization` دائماً |
| 2 | **لا تنسى `ServiceLifetime.Scoped`** في `AddDbContextFactory` |
| 3 | **استخدم `DbContextFactory`** للصفحات الجديدة - لا تحقن `DbContext` مباشرة |
| 4 | **سجل خدماتك قبل `builder.Build()`** - أي تسجيل بعد البناء لن يعمل |
| 5 | **تأكد من وجود `AddMemoryCache()`** - ضروري للترجمة والقوائم |

---

## 🔗 روابط ذات صلة

- [00 - الهيكل المعماري](00-architecture-overview.md)
- [02 - نظام الهوية والمصادقة](02-identity-system.md)
- [05 - إنشاء الصفحات والمكونات](05-page-creation-checklist.md)
- [الملحق ب - فهرس الخدمات](../appendix-b-service-index.md)
```

---

