# 01 - البنية التحتية والتسجيلات الأساسية (Program.cs & App.razor)

**آخر تحديث:** 18 يوليو 2026

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

- **التهيئة المبكرة للغة:** بدون استدعاء `TranslationState.InitializeAsync()` هنا، ستعرض الصفحة الأولى اللغة الافتراضية ثم "تومض" عند تحميل اللغة الحقيقية.
- **التوجيه الموحد:** `DefaultLayout="@typeof(MainLayout)"` يضمن أن كل الصفحات ترث التخطيط نفسه.

---

## 3. 🎯 مصفوفة اختيار الخدمة - مرجع اتخاذ القرار (⭐ جديد - 18 يوليو 2026)

### لماذا هذا القسم مهم؟
في المشروع الحالي، يوجد عدة طرق للوصول إلى البيانات. السؤال الأكثر تكراراً من المطورين الجدد هو: **"أي خدمة أستخدم لصفحتي الجديدة؟"**. هذا القسم يجيب على هذا السؤال بشكل قاطع.

### 📊 المصفوفة الكاملة للخدمات

#### 3.1 خدمات أساسية (ضرورية لكل صفحة)

| الخدمة | الواجهة | متى تستخدمها؟ | مثال |
|--------|---------|---------------|------|
| `GenericService<T>` | `IGenericService<T>` | **CRUD بسيط** على كيان واحد (عرض/إضافة/تعديل/حذف) | قائمة الأدوية، تعديل اسم منظمة |
| `DbContextFactoryService` | - | **استعلامات قراءة مخصصة** معقدة (تقارير، فلترة متعددة) | لوحة التحكم، تقارير المبيعات |
| `UserContextService` | `IUserContextService` | **معلومات المستخدم الحالي** (ID، الأدوار، الصلاحيات) | أي صفحة تحتاج معرفة من هو المسجل |

#### 3.2 خدمات الترجمة (لأي صفحة تدعم اللغتين)

| الخدمة | الواجهة | متى تستخدمها؟ | مثال |
|--------|---------|---------------|------|
| `TranslationStateService` | - | معرفة **اللغة الحالية** والاشتراك في تغييراتها | أي صفحة تدعم AR/EN |
| `ILocalizationService` | `ILocalizationService` | جلب **الترجمات من قاعدة البيانات** | عرض النصوص المترجمة |
| `LayoutDirectionService` | `ILayoutDirectionService` | إدارة **اتجاه الصفحة** (RTL/LTR) | ضبط `dir` attribute |

#### 3.3 خدمات المستخدم والسياق

| الخدمة | الواجهة | متى تستخدمها؟ | مثال |
|--------|---------|---------------|------|
| `UserContextService` | `IUserContextService` | ID المستخدم، الأدوار، الصلاحيات | التحقق من صلاحية الوصول |
| `IUserSessionService` | `IUserSessionService` | localStorage + إدارة الجلسات | حفظ تفضيلات الجلسة |
| `IUserRoleService` | `IUserRoleService` | **إدارة الأدوار الموسعة** (AspNetUserRoles) | إضافة/تعديل/حذف أدوار المستخدمين |

#### 3.4 خدمات التنقل والقوائم

| الخدمة | الواجهة | متى تستخدمها؟ | مثال |
|--------|---------|---------------|------|
| `DynamicMenuService` | - | جلب **القوائم الديناميكية** للمستخدم | عرض القائمة الجانبية |

#### 3.5 خدمات مساعدة

| الخدمة | الواجهة | متى تستخدمها؟ | مثال |
|--------|---------|---------------|------|
| `ExcelService` | `IExcelService<T>` | استيراد/تصدير Excel | تصدير قائمة مرضى |
| `IImageService` | `IImageService` | رفع ومعالجة الصور | رفع صورة البروفايل |

---

## 4. 🧭 شجرة قرار اختيار الخدمة (⭐ جديد - 18 يوليو 2026)

### السؤال الأول: هل الصفحة جديدة أم قديمة؟

```
هل الصفحة جديدة؟
│
├── نعم → انتقل للسؤال الثاني
│
└── لا → هل تستخدم @inject BusinessDbContext مباشرة؟
          │
          ├── نعم → خطط لترحيلها تدريجياً
          │         (ابدأ بـ IGenericService ثم UseCase)
          │
          └── لا → اتركها كما هي إذا كانت تعمل
```

### السؤال الثاني: هل العملية مشتركة بين Web و Mobile؟

```
هل العملية ستُستخدم في كل من Web و Mobile؟
│
├── نعم → استخدم UseCase في Application + API في Api.Web
│         (مثال: SyncCompanyMedications, CreatePsp)
│         📖 راجع [00 - شجرة القرار المعماري](00-architecture-overview.md)
│
└── لا → انتقل للسؤال الثالث
```

### السؤال الثالث: ما مستوى تعقيد العملية؟

```
هل العملية معقدة (منطق أعمال متعدد الخطوات، تحويلات، تحقق متعدد)؟
│
├── نعم → استخدم UseCase في Application
│         (مثال: EnrollPatientUseCase, ProcessOrderUseCase)
│
└── لا → هل العملية CRUD بسيطة؟
          │
          ├── نعم → استخدم IGenericService<T>
          │         (مثال: عرض قائمة أدوية، تعديل اسم)
          │
          └── لا → استخدم DbContextFactoryService
                    (للاستعلامات المخصصة المعقدة)
```

---

## 5. 💡 أمثلة عملية على اختيار الخدمة الصحيحة

### ✅ مثال 1: صفحة CRUD بسيطة (عرض قائمة أدوية)

**القرار:** استخدم `IGenericService<T>`

```csharp
// في Web/Pages/Admin/Medications.razor
@inject IGenericService<Medication> MedicationService

@code {
    private List<Medication> medications = new();
    
    protected override async Task OnInitializedAsync()
    {
        medications = await MedicationService.GetAllAsync().ToListAsync();
    }
    
    private async Task DeleteMedication(int id)
    {
        await MedicationService.DeleteAsync(id);
    }
}
```

**السبب:** عملية قراءة/حذف بسيطة، لا تحتاج UseCase.

---

### ✅ مثال 2: عملية معقدة (مزامنة أدوية الشركة)

**القرار:** استخدم UseCase

```csharp
// في Application/UseCases/Medication/SyncCompanyMedicationsUseCase.cs
public class SyncCompanyMedicationsUseCase
{
    private readonly IMedicationRepository _repository;
    
    public async Task<Result> ExecuteAsync(int companyId)
    {
        // منطق معقد: مزامنة، تحقق، تحويل
        var companyMeds = await _repository.GetCompanyMedicationsAsync(companyId);
        // ... منطق الأعمال ...
        await _repository.SyncToMainTableAsync(companyMeds);
        return Result.Success();
    }
}

// في Web/Pages/Admin/SyncMedications.razor
@inject SyncCompanyMedicationsUseCase SyncUseCase

@code {
    private async Task SyncNow()
    {
        var result = await SyncUseCase.ExecuteAsync(CurrentCompanyId);
        // ...
    }
}
```

**السبب:** عملية معقدة متعددة الخطوات، تحتاج UseCase.

---

### ✅ مثال 3: استعلام قراءة معقد (لوحة التحكم)

**القرار:** استخدم `DbContextFactoryService`

```csharp
// في Web/Pages/Dashboard.razor
@inject DbContextFactoryService DbFactory

@code {
    private DashboardStats? stats;
    
    protected override async Task OnInitializedAsync()
    {
        await using var context = await DbFactory.CreateDbContextAsync();
        
        stats = new DashboardStats
        {
            TotalPatients = await context.UserProfiles
                .CountAsync(up => up.IsActive),
            ActivePrograms = await context.PSPs
                .CountAsync(p => p.IsActive),
            RecentOrders = await context.Orders
                .OrderByDescending(o => o.CreatedDate)
                .Take(10)
                .ToListAsync()
        };
    }
}
```

**السبب:** استعلام مخصص معقد، لا يناسب `IGenericService<T>`.

---

### ✅ مثال 4: عملية مشتركة (Web + Mobile)

**القرار:** استخدم UseCase + API

```csharp
// 1. في Application/UseCases/Medication/SyncCompanyMedicationsUseCase.cs
// (نفس UseCase أعلاه)

// 2. في Api.Web/Controllers/MedicationController.cs
[HttpPost("sync")]
public async Task<IActionResult> Sync([FromBody] SyncRequest request)
{
    var result = await _syncUseCase.ExecuteAsync(request.CompanyId);
    return Ok(result);
}

// 3. في Mobile - يستدعي API
var result = await ApiService.PostAsync<SyncResult>(
    "api/medication/sync", 
    new { CompanyId = 123 });

// 4. في Web - يستدعي UseCase مباشرة (لأنه على نفس الخادم)
var result = await SyncUseCase.ExecuteAsync(123);
```

**السبب:** العملية مشتركة، نضع UseCase في Application ونستدعيها بطرق مختلفة.

---

## 6. 🎯 مصفوفة "متى تستخدم ماذا" - مرجع سريع (⭐ جديد - 18 يوليو 2026)

| نوع الصفحة | النهج الموصى به | الخدمة | مثال |
|-----------|----------------|--------|------|
| **CRUD بسيط** (إضافة/تعديل/حذف) | `IGenericService<T>` | GenericService | قائمة أدوية، تعديل اسم |
| **صفحة معقدة** (تقارير، عمليات تجارية) | **UseCase** | UseCase في Application | إنشاء PSP، تسجيل مريض |
| **صفحة مشتركة بين Web و Mobile** | **API** | UseCase + API | المزامنة، الإشعارات |
| **عمليات حساسة** (دفع، مصادقة) | **UseCase + API** | UseCase + Validation | الدفع، تغيير كلمة المرور |
| **تقارير ولوحات تحكم** | `DbContextFactoryService` | DbFactory | Dashboard، إحصائيات |
| **صفحات قديمة** | ترحيل تدريجي | - | استخدم IGenericService بدلاً من DbContext |

---

## 7. CHECKLIST: قبل كتابة أي صفحة جديدة

### التحقق من التسجيلات

- [ ] هل الخدمة التي سأستخدمها مسجلة في `Program.cs`؟
- [ ] إذا كانت خدمة جديدة، هل أضفتها باستخدام `AddScoped` أو `AddSingleton`؟
- [ ] هل تأكدت من عدم وجود تسجيل مزدوج لنفس الخدمة؟

### التحقق من قاعدة البيانات

- [ ] هل النموذج (Model) موجود في مجلد `Data/Models`؟
- [ ] هل تم إضافة `DbSet<MyEntity>` في `BusinessDbContext`؟
- [ ] هل تم إنشاء `Migration` جديدة بعد إضافة النموذج؟

### التحقق من نمط الصفحة (⭐ محدّث - 18 يوليو 2026)

- [ ] **هل الصفحة جديدة؟** → استخدم `IGenericService<T>` و `DbContextFactory`
- [ ] **هل العملية معقدة؟** → استخدم UseCase في Application
- [ ] **هل العملية مشتركة بين Web و Mobile؟** → استخدم UseCase + API
- [ ] **هل الصفحة قديمة وتستخدم `@inject BusinessDbContext` مباشرة؟** → خطط لترحيلها
- [ ] **هل راجعت شجرة القرار في [00 - الهيكل المعماري](00-architecture-overview.md)؟**

### التحقق من الترجمة

- [ ] هل أضفت مفاتيح الترجمة في قاعدة البيانات مع `N'' prefix` للعربية؟
- [ ] هل تستخدم `T()` المساعدة بدلاً من `<LocalizedText />`؟

---

## 8. تحذيرات ومحاذير

| # | التحذير |
|---|---------|
| 1 | لا تغير ترتيب Middleware - `UseAuthentication` قبل `UseAuthorization` دائماً |
| 2 | لا تنسى `ServiceLifetime.Scoped` في `AddDbContextFactory` |
| 3 | استخدم `DbContextFactory` للصفحات الجديدة - لا تحقن `DbContext` مباشرة |
| 4 | سجل خدماتك قبل `builder.Build()` - أي تسجيل بعد البناء لن يعمل |
| 5 | تأكد من وجود `AddMemoryCache()` - ضروري للترجمة والقوائم |
| 6 | ⭐ **لا تستخدم `IGenericService<T>` للعمليات المعقدة** - استخدم UseCase |
| 7 | ⭐ **لا تضع منطق أعمال في Controllers** - Controllers للتنسيق فقط |
| 8 | ⭐ **Mobile لا يعتمد على أي مشروع** - التواصل عبر HTTP فقط |

---

## 🔗 روابط ذات صلة

- [00 - الهيكل المعماري](00-architecture-overview.md) - يحتوي على شجرة القرار المعماري الكاملة ⭐
- [02 - نظام الهوية والمصادقة](02-identity-system.md)
- [05 - إنشاء الصفحات والمكونات](05-page-creation-checklist.md)
- [09 - دليل API](09-api-guide.md)
- [الملحق ب - فهرس الخدمات](../appendix-b-service-index.md)

---
```

---

## 📊 ملخص التحديثات في هذه الوثيقة

| القسم | التغيير |
|-------|---------|
| **المحتوى الأصلي (الأقسام 1-2)** | ✅ محفوظ بالكامل دون أي حذف |
| **القسم 3: مصفوفة اختيار الخدمة** | ✅ تحديث شامل مع عمود "متى تستخدمها؟" |
| **القسم 4: شجرة قرار اختيار الخدمة** | ✅ إضافة جديدة بالكامل |
| **القسم 5: أمثلة عملية** | ✅ 4 أمثلة كود جاهزة لكل حالة |
| **القسم 6: مصفوفة "متى تستخدم ماذا"** | ✅ جدول مرجعي سريع |
| **القسم 7: CHECKLIST** | ✅ تحديث نقاط التحقق لتشمل القرارات المعمارية |
| **القسم 8: تحذيرات** | ✅ إضافة 3 تحذيرات جديدة |
| **الروابط** | ✅ إضافة رابط للوثيقة 00 المحدثة |
