# 01 — ARCHITECTURE (الهيكل المعماري الشامل + Program.cs)

**الإصدار:** 2.0 (Unified Edition)
**آخر تحديث:** 16 أكتوبر 2026
**الحالة:** ✅ محدّث — يدمج 00 + 01 + 24
**الجمهور:** فريق التطوير + AI Assistants
**المرجع الإلزامي:** يُقرأ قبل أي تطوير جديد

---

## 📌 الجزء صفر: فلسفة البناء (نص المؤلف الحرفي)

> **⚠️ ملاحظة:**
> هذا القسم يحتوي على **نص الكاتب الحرفي** — دون إعادة صياغة.
> الهدف: حفظ الرؤية الكاملة كما صيغت.

نظام **RubikCare** هو منصة صحية متكاملة، صُممت لتوحيد إدارة المرضى، ومقدمي الرعاية (أطباء، صيادلة، مندوبين)، وبرامج الدعم (PSP) في منظومة رقمية واحدة. يعتمد النظام على **Clean Architecture** لضمان فصل واضح للمسؤوليات، وقابلية عالية للتطوير، وسهولة في الصيانة والاختبار.

تمتد المنصة عبر **تسعة مشاريع برمجية**، تنقسم إلى طبقات أساسية (Core) وتطبيقات طرفية (Clients)، جميعها تشترك في مكونات UI موحدة وتتواصل عبر واجهة برمجية (API) مركزية.

---

## 🗺️ الجزء الأول: الخريطة المعمارية للمشاريع التسعة

### 1.1 المخطط المعماري

```mermaid
graph TD
    subgraph "Core Layers"
        Domain["RubikCare.Domain<br/>Entities, Enums, Interfaces"]
        App["RubikCare.Application<br/>Use Cases, DTOs, Services"]
        Infra["RubikCare.Infrastructure<br/>DbContext, Repositories"]
    end

    subgraph "Presentation & API"
        Api["RubikCare.Api.Web<br/>Central REST API Gateway"]
    end

    subgraph "Client Applications"
        Web["RubikCare.Web<br/>Blazor Server App"]
        PWA["RubikCare.PWA<br/>Blazor WebAssembly App"]
        Mobile["RubikCare.Mobile<br/>MAUI App"]
    end

    subgraph "Shared & Testing"
        SharedUI["RubikCare.Shared.UI<br/>Razor Component Library"]
        Tests["RubikCare.Tests<br/>Unit & Integration Tests"]
    end

    App --> Domain
    Infra --> App & Domain
    Api --> App & Infra

    Web -- "SignalR / HTTP" --> Api
    PWA -- "HTTP / REST" --> Api
    Mobile -- "HTTP / REST" --> Api

    SharedUI -.-> Web & PWA & Mobile
    Tests -.-> Api & App & Infra & Domain & Web & PWA & Mobile & SharedUI
```

### 1.2 الجدول التنفيذي للمشاريع

| المشروع | التقنية الأساسية | المسؤولية الرئيسية | يعتمد على (مباشر) |
| :--- | :--- | :--- | :--- |
| **Domain** | .NET 10 Class Library | الكيانات الأساسية، التعدادات، وواجهات النطاق المجردة. | لا يعتمد على أي مشروع. |
| **Application** | .NET 10 Class Library | منطق الأعمال، حالات الاستخدام (Use Cases)، وخدمات التطبيق. | `Domain` فقط. |
| **Infrastructure** | .NET 10, EF Core 10, SQL Server | تنفيذ واجهات الـ `Application`، الوصول للبيانات، الترحيلات (Migrations). | `Application` و `Domain`. |
| **Api.Web** | ASP.NET Core 10 | نقطة النهاية الموحدة (REST API) لجميع العملاء. | `Application` و `Infrastructure`. |
| **Web** | Blazor Server 10 | تطبيق الويب الرئيسي (تفاعلي مع SignalR). | `Api.Web` (ضمنياً) و `Shared.UI`. |
| **PWA** | Blazor WebAssembly 10 | تطبيق ويب تقدمي (نسخة كربونية من Mobile). | `Api.Web` (عبر HTTP) و `Shared.UI`. |
| **Mobile** | .NET MAUI 10 | تطبيق الهواتف الذكية (Android, iOS). | `Api.Web` (عبر HTTP) و `Shared.UI`. |
| **Shared.UI** | Razor Class Library (RCL) | مكونات Razor قابلة لإعادة الاستخدام + خدمات مشتركة. | لا يعتمد على تطبيقات محددة. |
| **Tests** | xUnit, Moq | اختبارات الوحدة والتكامل والمعمارية. | جميع المشاريع (حسب الحاجة). |

---

## 🧩 الجزء الثاني: مسؤوليات كل مشروع بالتفصيل

### 2.1 `RubikCare.Domain` — القلب النابض 🫀

**أكثر طبقة استقراراً ونقاءً.**

لا تعرف هذه الطبقة شيئاً عن قواعد البيانات، أو واجهات المستخدم، أو حتى كيفية نقل البيانات. همها الوحيد هو تمثيل مفاهيم العمل (Business Concepts).

- **المحتويات**: الكيانات (Entities) مثل `ApplicationUser`, `Patient`, `Medication`، التعدادات (Enums)، كائنات القيمة (Value Objects) كـ `Address`، وأحداث النطاق (Domain Events).
- **إضافة حديثة**: كيانات نظام النقاط (`PSPProgramPointScheme`, `PSPProgramPointRule`, إلخ).
- **قاعدة ذهبية**: **ممنوع** إضافة أي مرجعيات لـ Entity Framework أو مكتبات خارجية متعلقة بالبيانات.

### 2.2 `RubikCare.Application` — عقل النظام 🧠

**هنا يُتخذ القرار.**

تحتوي على حالات الاستخدام (Use Cases) التي تمثل متطلبات المستخدم الفعلية (مثل: `EnrollPatientInPspUseCase`). تقوم بتنسيق تدفق البيانات بين الـ `Api.Web` والـ `Infrastructure`، وتطبق منطق الأعمال.

- **المحتويات**: الخدمات (Services)، واجهات للبنية التحتية (Interfaces)، كائنات نقل البيانات (DTOs)، محولات (Mappers)، ومُحققّات (Validators) باستخدام FluentValidation.
- **إضافات حديثة**:
  - `PSPEnrollmentService` — تسجيل المرضى في برامج الدعم.
  - `PSPPointSchemeService` — قراءة نظام النقاط.
- **قاعدة ذهبية**: لا تعتمد إلا على `Domain` ولا تعرف كيف أو من أين تأتي البيانات (مستودع، خدمة خارجية، إلخ).

### 2.3 `RubikCare.Infrastructure` — الجسر إلى العالم الخارجي 🌉

**التفاصيل التقنية هنا.**

هذا هو مكان تنفيذ الواجهات المجردة التي عرّفها الـ `Application`. مسؤول عن التواصل مع قواعد البيانات، وإرسال البريد الإلكتروني، والخدمات السحابية.

- **المحتويات**: `BusinessDbContext` (EF Core)، تنفيذ المستودعات (Repositories)، ملفات الترحيل (Migrations)، وخدمات خارجية (مثل `EmailService`).
- **إضافات حديثة**:
  - **19 Handler** لنظام النقاط في `UseCases/PSP/Points/`.
  - **`PSPEnrollmentService`** في `Services/PSP/` (يستخدم `IGenericService<T>`).
- **قاعدة ذهبية**: يُسمح لها بالاعتماد على `Application` و `Domain` فقط، ولا يُستدعى مباشرة من قبل التطبيقات العميلة (Web, PWA, Mobile).

### 2.4 `RubikCare.Api.Web` — بوابة النظام الموحدة 🚪

**نقطة الدخول الوحيدة للبيانات.**

هي واجهة برمجية (REST API) مركزية تخدم جميع العملاء (Web, PWA, Mobile). تتولى المصادقة (JWT)، والتحقق من الصلاحيات، وتنسيق الطلبات.

- **المحتويات**: وحدات التحكم (Controllers)، `Program.cs`، وMiddleware (معالجة الأخطاء، التسجيل).
- **Controllers حديثة**:
  - `PointSchemesController` (4 endpoints)
  - `PointRulesController` (5 endpoints)
  - `PointBalancesController` (3 endpoints)
  - `PointEarningController` (1 endpoint)
  - `PointRedeemController` (5 endpoints)
- **قاعدة ذهبية**: وحدات التحكم يجب أن تكون **رفيعة جداً** (Thin Controllers)؛ كل المنطق يُفوض إلى الخدمات في طبقة `Application` أو إلى Handlers في `Infrastructure`.

### 2.5 `RubikCare.Web` — التطبيق التفاعلي الغني 💻

**تطبيق Blazor Server.**

يُستخدم في بيئات الإدارة واللوحات التي تتطلب تفاعلاً فورياً. يحافظ على اتصال مستمر (SignalR) مع الخادم، مما يوفر تجربة مستخدم سلسة.

- **المحتويات**: صفحات (Pages)، مكونات خاصة بالويب، وخدمات محلية لإدارة الحالة.
- **إضافات حديثة**:
  - **`PSPStep2b_PointScheme.razor`** — واجهة إدارة نظام النقاط للشركة.
  - **قسم "نظام النقاط"** في `PSPProgramsDetails.razor`.
- **قاعدة ذهبية**: يتواصل مع الـ `Api.Web` عبر HTTP، ولا يتفاعل مباشرة مع قواعد البيانات.

### 2.6 `RubikCare.PWA` — التطبيق الخفيف متعدد المنصات 📱

**تطبيق Blazor WebAssembly — نسخة كربونية من MAUI.**

تمت إضافته ليكون بديلاً لتطبيق الويب (Blazor Server) على الأجهزة التي لا تتعامل بكفاءة مع SignalR (مثل أجهزة iPhone)، وأيضاً ليكون حلاً سريعاً ومستقراً للوصول إلى النظام من أي متصفح.

- **طبيعة التشغيل**: يعمل كلياً داخل متصفح المستخدم بعد تحميل ملفات `WebAssembly`. يُنشر كملفات **ثابتة** (Static Files) على خادم ويب (مثل IIS).
- **المحتويات**:
  - صفحات Razor مكافئة لـ `RubikCare.Mobile`.
  - خدمات للتواصل مع الـ API (مثل `WebApiService`).
  - `googleAuth.js` — تكامل Google Identity Services.
  - `cache-reset.html` — لتنظيف Service Worker تلقائياً.
- **إضافات حديثة**:
  - **`PointDashboard.razor`** — محفظة النقاط.
- **الرابط المرجعي**: [دليل PWA](./14-PWA.md).

### 2.7 `RubikCare.Mobile` — تجربة الهواتف الذكية 📲

**تطبيق MAUI الأصلي.**

يستخدم `BlazorWebView` لعرض مكونات `Shared.UI`، مما يسمح بإعادة استخدام كبير للكود مع تطبيقات الويب. يوفر إمكانية الوصول إلى ميزات الجهاز (الكاميرا، GPS، الإشعارات المحلية).

- **المحتويات**: صفحات XAML، ViewModels، وخدمات خاصة بالمنصة (مثل `MauiLocalNotificationService`).
- **إضافات حديثة**:
  - **`PointsDashboardPage.xaml`** — Container لعرض `PointDashboard.razor`.
- **قاعدة ذهبية**: لا يعتمد على أي مشروع آخر باستثناء `Shared.UI`؛ يتواصل مع النظام حصراً عبر `Api.Web` باستخدام HTTP.

### 2.8 `RubikCare.Shared.UI` — مستودع المكونات المشتركة 🧩

**اللبنة الأساسية لواجهات المستخدم.**

مكتبة تحتوي على كل المكونات البصرية والخدمات المساعدة التي تستخدمها التطبيقات العميلة الثلاثة (Web, PWA, Mobile)، مما يضمن توحيد الشكل والأداء.

- **المحتويات**:
  - **68** مكون Razor (مثل `RubikSmartTable`, `RubikButton`).
  - **120+** ملف CSS منظم.
  - **25** خدمة/واجهة مشتركة (مثل `ITranslationService`).
- **إضافات حديثة**:
  - **`PointDashboard.razor`** + `PointDashboardNavigationBridge.cs` + `PointModels.cs`.
  - **`point-dashboard.css`**.
- **قاعدة ذهبية**: تحتوي على **مكونات** قابلة لإعادة الاستخدام، وليس صفحات كاملة، لتجنب تعقيد التبعيات.

### 2.9 `RubikCare.Tests` — شبكة الأمان الجودة 🛡️

**ضمان استقرار المعمارية.**

يحتوي على اختبارات الوحدة (Unit Tests) لكل طبقة، واختبارات تكامل (Integration Tests) بين الطبقات، بالإضافة إلى اختبارات معماريّة باستخدام `NetArchTest` للتأكد من عدم كسر قواعد التبعية.

- **عدد الاختبارات**: 33 اختبار ناجح (حتى الآن).
- **قاعدة ذهبية**: جزء لا يتجزأ من عملية التطوير.

---

## 🧭 الجزء الثالث: شجرة القرار المعماري (دليل المطور)

مع وجود ثلاثة تطبيقات عميلة وطبقات متعددة، قد يحتار المطور في النهج الصحيح لتطوير صفحة جديدة. هذا القسم يوفر إرشادات عملية.

### 3.1 مستويات "النقاء المعماري" (من العملي إلى النظري)

| المستوى | الوصف | النهج |
| :--- | :--- | :--- |
| **LVL 1** | **مقبول للصفحات القديمة جداً** (يُنصح بالترقية). | `Blazor Page` ← `DbContext` مباشرة. |
| **LVL 2** | **مناسب للعمليات البسيطة (CRUD)**. | `Blazor Page` ← `IGenericService<T>` ← `DbContext`. |
| **LVL 3** | **الموصى به للعمليات المعقدة والجديدة**. | `Blazor/MAUI/PWA Page` ← `UseCase` (في `Application`) ← `Repository` (في `Infrastructure`). |
| **LVL 4** | **الأنقى للعمليات المشتركة بين العملاء**. | `Client (Web/PWA/Mobile)` → (HTTP) → `Api.Web` → `UseCase`. |

### 3.2 القاعدة الذهبية: اختر النهج حسب التعقيد والمشاركة

| نوع الصفحة/الوظيفة | النهج الأمثل | مثال تطبيقي |
| :--- | :--- | :--- |
| **CRUD بسيط** (إضافة/تعديل/حذف لكيان واحد) | **LVL 2**: `IGenericService<T>` | عرض قائمة الأدوية، تعديل ملف المستخدم. |
| **منطق أعمال معقد** (متعدد الخطوات والشروط) | **LVL 3**: `UseCase` (في `Application`) | تسجيل مريض في برنامج دعم (PSP)، عملية مزامنة مع نظام خارجي. |
| **وظيفة مشتركة بين Web و PWA و Mobile** | **LVL 4**: `UseCase` (في `Application`) + `API` | إرسال إشعار، إنشاء تقرير، البحث المتقدم. |
| **عمليات حساسة جداً** (مالية، مصادقة) | **LVL 4**: `UseCase` + `API` مع تحقق إضافي (Validation). | معالجة الدفع، تغيير كلمة المرور. |
| **استعلامات تقارير معقدة** (قراءة فقط) | **استخدام** `DbContextFactoryService` مباشرة في الصفحة (مع الحذر). | لوحة التحكم (Dashboard) بإحصائيات متعددة. |

### 3.3 ⚠️ تحذيرات معمارية صارمة

- **لا تكسر تبعية الطبقات**: ممنوع استدعاء `Infrastructure` أو `DbContext` مباشرة من `Web` أو `PWA` أو `Mobile`.
- **الـ `API` للجميع**: تطبيقات `Mobile` و `PWA` **لا تعتمد** على أي مشروع آخر (`Domain`, `Application`...)، وتتواصل فقط مع `Api.Web` عبر HTTP.
- **`Shared.UI` للمكونات**: لا تضع صفحات كاملة (مثل `Dashboard.razor`) في `Shared.UI`، بل ضعها في المشاريع العميلة نفسها (`Web`, `PWA`, `Mobile`).
- **استخدم DTOs**: لا تُرجع كائنات `Domain` (Entities) مباشرة من الـ `API`، استخدم DTOs لمنع تسرب تفاصيل النطاق وتقليل حجم البيانات.

---

## 📋 الجزء الرابع: العلاقات والتبعيات (بصيغة مبسطة)

```mermaid
flowchart LR
    subgraph Clients
        Web[Web - Blazor Server] -->|SignalR/HTTP| API
        PWA[PWA - Blazor WASM] -->|HTTP| API
        Mobile[Mobile - MAUI] -->|HTTP| API
    end

    subgraph Core
        API[Api.Web] --> App[Application]
        App --> Dom[Domain]
        Inf[Infrastructure] --> App & Dom
    end

    subgraph Shared
        UI[Shared.UI] -.-> Web & PWA & Mobile
    end
```

**خلاصة القاعدة**: السهم يشير إلى "يعتمد على". التطبيقات العميلة تعتمد على الـ API، والـ API يعتمد على الـ Application، والـ Application على الـ Domain، والـ Infrastructure على الـ Application والـ Domain. `Shared.UI` في المنتصف يستخدمه الجميع.

---

## 🔧 الجزء الخامس: Program.cs — مصدر الحقيقة الوحيد للتسجيلات

### 5.1 الغرض

هذا الملف هو **المصدر الوحيد** لكل الخدمات والإعدادات في النظام. أي خدمة تريد استخدامها في صفحتك **يجب** أن تكون مسجلة هنا.

### 5.2 منطقة الأساسيات ودعم SSR

```csharp
builder.Services.AddRazorComponents()
    .AddInteractiveServerComponents();
```

### 5.3 منطقة إدارة حالة المصادقة

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

### 5.4 منطقة قاعدة البيانات (الأهم والأكثر خطأً)

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

**⚠️ تحذير حرج:**
- تأكد من وجود `ServiceLifetime.Scoped` في `AddDbContextFactory`
- لا تحذف `AddDbContext` إلا بعد ترحيل كل الصفحات القديمة
- `AddDbContextFactory` هو الموصى به للصفحات الجديدة لمنع مشكلة "A second operation started"

### 5.5 منطقة الهوية (Identity)

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

### 5.6 منطقة خدمات النظام الإضافية

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

### 5.7 إعدادات Middleware (الترتيب مهم جداً)

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

## 📄 الجزء السادس: App.razor — تدفق التطبيق

### 6.1 الكود الكامل

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

### 6.2 النقاط الحرجة

- **التهيئة المبكرة للغة:** بدون استدعاء `TranslationState.InitializeAsync()` هنا، ستعرض الصفحة الأولى اللغة الافتراضية ثم "تومض" عند تحميل اللغة الحقيقية.
- **التوجيه الموحد:** `DefaultLayout="@typeof(MainLayout)"` يضمن أن كل الصفحات ترث التخطيط نفسه.

---

## 🎯 الجزء السابع: مصفوفة اختيار الخدمة

### 7.1 لماذا هذا القسم مهم؟

في المشروع الحالي، يوجد عدة طرق للوصول إلى البيانات. السؤال الأكثر تكراراً من المطورين الجدد هو: **"أي خدمة أستخدم لصفحتي الجديدة؟"**. هذا القسم يجيب على هذا السؤال بشكل قاطع.

### 7.2 📊 المصفوفة الكاملة للخدمات

#### 7.2.1 خدمات أساسية (ضرورية لكل صفحة)

| الخدمة | الواجهة | متى تستخدمها؟ | مثال |
|--------|---------|---------------|------|
| `GenericService<T>` | `IGenericService<T>` | **CRUD بسيط** على كيان واحد (عرض/إضافة/تعديل/حذف) | قائمة الأدوية، تعديل اسم منظمة |
| `DbContextFactoryService` | - | **استعلامات قراءة مخصصة** معقدة (تقارير، فلترة متعددة) | لوحة التحكم، تقارير المبيعات |
| `UserContextService` | `IUserContextService` | **معلومات المستخدم الحالي** (ID، الأدوار، الصلاحيات) | أي صفحة تحتاج معرفة من هو المسجل |

#### 7.2.2 خدمات الترجمة (لأي صفحة تدعم اللغتين)

| الخدمة | الواجهة | متى تستخدمها؟ | مثال |
|--------|---------|---------------|------|
| `TranslationStateService` | - | معرفة **اللغة الحالية** والاشتراك في تغييراتها | أي صفحة تدعم AR/EN |
| `ILocalizationService` | `ILocalizationService` | جلب **الترجمات من قاعدة البيانات** | عرض النصوص المترجمة |
| `LayoutDirectionService` | `ILayoutDirectionService` | إدارة **اتجاه الصفحة** (RTL/LTR) | ضبط `dir` attribute |

#### 7.2.3 خدمات المستخدم والسياق

| الخدمة | الواجهة | متى تستخدمها؟ | مثال |
|--------|---------|---------------|------|
| `UserContextService` | `IUserContextService` | ID المستخدم، الأدوار، الصلاحيات | التحقق من صلاحية الوصول |
| `IUserSessionService` | `IUserSessionService` | localStorage + إدارة الجلسات | حفظ تفضيلات الجلسة |
| `IUserRoleService` | `IUserRoleService` | **إدارة الأدوار الموسعة** (AspNetUserRoles) | إضافة/تعديل/حذف أدوار المستخدمين |

#### 7.2.4 خدمات التنقل والقوائم

| الخدمة | الواجهة | متى تستخدمها؟ | مثال |
|--------|---------|---------------|------|
| `DynamicMenuService` | - | جلب **القوائم الديناميكية** للمستخدم | عرض القائمة الجانبية |

#### 7.2.5 خدمات مساعدة

| الخدمة | الواجهة | متى تستخدمها؟ | مثال |
|--------|---------|---------------|------|
| `ExcelService` | `IExcelService<T>` | استيراد/تصدير Excel | تصدير قائمة مرضى |
| `IImageService` | `IImageService` | رفع ومعالجة الصور | رفع صورة البروفايل |

---

## 🧭 الجزء الثامن: شجرة قرار اختيار الخدمة

### 8.1 السؤال الأول: هل الصفحة جديدة أم قديمة؟

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

### 8.2 السؤال الثاني: هل العملية مشتركة بين Web و Mobile؟

```
هل العملية ستُستخدم في كل من Web و Mobile؟
│
├── نعم → استخدم UseCase في Application + API في Api.Web
│         (مثال: SyncCompanyMedications, CreatePsp)
│         📖 راجع [00 - شجرة القرار المعماري](./01-ARCHITECTURE.md#-الجزء-الثالث-شجرة-القرار-المعماري-دليل-المطور)
│
└── لا → انتقل للسؤال الثالث
```

### 8.3 السؤال الثالث: ما مستوى تعقيد العملية؟

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

## 💡 الجزء التاسع: أمثلة عملية على اختيار الخدمة الصحيحة

### 9.1 ✅ مثال 1: صفحة CRUD بسيطة (عرض قائمة أدوية)

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

### 9.2 ✅ مثال 2: عملية معقدة (مزامنة أدوية الشركة)

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

### 9.3 ✅ مثال 3: استعلام قراءة معقد (لوحة التحكم)

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

### 9.4 ✅ مثال 4: عملية مشتركة (Web + Mobile)

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

## 🎯 الجزء العاشر: مصفوفة "متى تستخدم ماذا" — مرجع سريع

| نوع الصفحة | النهج الموصى به | الخدمة | مثال |
|-----------|----------------|--------|------|
| **CRUD بسيط** (إضافة/تعديل/حذف) | `IGenericService<T>` | GenericService | قائمة أدوية، تعديل اسم |
| **صفحة معقدة** (تقارير، عمليات تجارية) | **UseCase** | UseCase في Application | إنشاء PSP، تسجيل مريض |
| **صفحة مشتركة بين Web و Mobile** | **API** | UseCase + API | المزامنة، الإشعارات |
| **عمليات حساسة** (دفع، مصادقة) | **UseCase + API** | UseCase + Validation | الدفع، تغيير كلمة المرور |
| **تقارير ولوحات تحكم** | `DbContextFactoryService` | DbFactory | Dashboard، إحصائيات |
| **صفحات قديمة** | ترحيل تدريجي | - | استخدم IGenericService بدلاً من DbContext |

---

## ✅ الجزء الحادي عشر: CHECKLIST — قبل كتابة أي صفحة جديدة

### 11.1 التحقق من التسجيلات

- [ ] هل الخدمة التي سأستخدمها مسجلة في `Program.cs`؟
- [ ] إذا كانت خدمة جديدة، هل أضفتها باستخدام `AddScoped` أو `AddSingleton`؟
- [ ] هل تأكدت من عدم وجود تسجيل مزدوج لنفس الخدمة؟

### 11.2 التحقق من قاعدة البيانات

- [ ] هل النموذج (Model) موجود في مجلد `Data/Models`؟
- [ ] هل تم إضافة `DbSet<MyEntity>` في `BusinessDbContext`؟
- [ ] هل تم إنشاء `Migration` جديدة بعد إضافة النموذج؟

### 11.3 التحقق من نمط الصفحة

- [ ] **هل الصفحة جديدة؟** → استخدم `IGenericService<T>` و `DbContextFactory`
- [ ] **هل العملية معقدة؟** → استخدم UseCase في Application
- [ ] **هل العملية مشتركة بين Web و Mobile؟** → استخدم UseCase + API
- [ ] **هل الصفحة قديمة وتستخدم `@inject BusinessDbContext` مباشرة؟** → خطط لترحيلها
- [ ] **هل راجعت شجرة القرار في [00 - الهيكل المعماري](./01-ARCHITECTURE.md#-الجزء-الثالث-شجرة-القرار-المعماري-دليل-المطور)؟**

### 11.4 التحقق من الترجمة

- [ ] هل أضفت مفاتيح الترجمة في قاعدة البيانات مع `N'' prefix` للعربية؟
- [ ] هل تستخدم `T()` المساعدة بدلاً من `<LocalizedText />`؟

---

## ⚠️ الجزء الثاني عشر: تحذيرات ومحاذير

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

## 📦 الجزء الثالث عشر: جرد المشاريع (Inventory)

### 13.1 نظرة عامة على المشاريع

| المشروع | التقنية | الغرض | عدد المكونات/الصفحات |
|---------|---------|-------|----------------------|
| **Shared.UI** | Razor Class Library (RCL) | مكونات UI مشتركة بين جميع المنصات | **68** مكون Razor + **120+** ملف CSS + **25** خدمة |
| **Mobile** | .NET MAUI + BlazorWebView | تطبيق الموبايل (Android + iOS) | **57** صفحة XAML + **16** ViewModel + **20+** خدمة |
| **PWA** | Blazor WebAssembly | نسخة كربونية من MAUI للمتصفح | **25+** صفحة Razor |

### 13.2 العلاقة بين المشاريع الثلاثة

```
┌─────────────────────────────────────────────────────────────────┐
│                    RubikCare.Shared.UI                          │
│              (مكتبة المكونات المشتركة - RCL)                    │
│  • 68 مكون Razor جاهز                                            │
│  • 120+ ملف CSS منظم حسب الأدوار                                │
│  • 25 خدمة/واجهة                                                │
└─────────────────────────────────────────────────────────────────┘
                                      │
        ┌─────────────────────────────┼─────────────────────────────┐
        │                             │                             │
        ▼                             ▼                             ▼
┌─────────────────┐          ┌─────────────────┐          ┌─────────────────┐
│   Mobile (MAUI)  │          │   Web (Blazor)   │          │      PWA         │
│  57 صفحة XAML    │          │   (Blazor Server)│          │  Blazor WASM     │
│  BlazorWebView   │          │                  │          │  نسخة كربونية    │
└─────────────────┘          └─────────────────┘          └─────────────────┘
```

### 13.3 جرد شامل: Shared.UI (68 مكون)

#### 🏥 فئة العيادة (Clinic) — 4 مكونات

| المكون | الوظيفة |
|--------|---------|
| `ClinicDashboard.razor` | لوحة تحكم الطبيب |
| `ClinicPatientDetails.razor` | تفاصيل مريض |
| `ClinicPatientsList.razor` | قائمة المرضى |
| `MyPatientInvitations.razor` | دعوات المرضى |

#### 💊 فئة الصيدلية (Pharmacy) — 12 مكون

| المكون | الوظيفة |
|--------|---------|
| `PharmacyDashboard.razor` | لوحة تحكم الصيدلية |
| `PharmacyDetailPage.razor` | تفاصيل صيدلية |
| `PharmacySearchPage.razor` | بحث صيدليات |
| `NearbyPharmacies.razor` | الصيدليات القريبة |
| `PharmacyGateway.razor` | بوابة الصيدلية |
| `PharmacyReportsPage.razor` | تقارير الصيدلية |
| `PatientOrders.razor` | طلبات المرضى |
| `MedicationRequestCard.razor` | بطاقة طلب دواء |
| `TokenRedemptionCard.razor` | بطاقة استرداد رمز |
| `PharmacyCard.razor` | بطاقة صيدلية |
| `PharmacyGrid.razor` | شبكة صيدليات |
| `PharmacyFilter.razor` | فلتر البحث |

#### 👔 فئة المندوب (Rep) — 5 مكونات

| المكون | الوظيفة |
|--------|---------|
| `RepDashboard.razor` | لوحة المندوب |
| `PharmaCompanyDashboard.razor` | لوحة شركة الأدوية |
| `InviteDoctor.razor` | دعوة طبيب |
| `MyInvitations.razor` | دعواتي |
| `MyNetwork.razor` | شبكتي |

#### 👤 فئة المريض (Patient) — 16 مكون

| المكون | الوظيفة |
|--------|---------|
| `SettingsPage.razor` | الإعدادات الرئيسية |
| `MyProfilePage.razor` | ملفي الشخصي |
| `EditProfilePage.razor` | تعديل الملف |
| `PatientOrdersPage.razor` | طلبات المريض |
| `OrderTracker.razor` | تتبع الطلب |
| `NotificationsPage.razor` | الإشعارات |
| `MedicationSchedulePage.razor` | جدول الأدوية |
| `AddMedicationPage.razor` | إضافة دواء |
| `PspSchedulePage.razor` | جدولة PSP |
| `DailyDosesTab.razor` | تبويب الجرعات اليومية |
| `RefillDatesTab.razor` | تبويب مواعيد الصرف |
| `ProfessionalStatusPage.razor` | الحالة المهنية |
| `ProfessionalLicensePage.razor` | الترخيص المهني |
| `SecuritySection.razor` | قسم الأمان |
| `LanguageSection.razor` | قسم اللغة |
| `NotificationsSection.razor` | قسم الإشعارات |
| `PrivacySection.razor` | قسم الخصوصية |

#### 💬 فئة المحادثات (Messaging) — 7 مكونات

| المكون | الوظيفة |
|--------|---------|
| `MessagingHubPage.razor` | مركز المحادثات |
| `ChatPage.razor` | صفحة الدردشة |
| `ChatMessage.razor` | مكون رسالة |
| `ConversationsListPage.razor` | قائمة المحادثات |
| `DoctorSearchPage.razor` | بحث عن طبيب |
| `DoctorProfilePage.razor` | ملف طبيب |
| `MessagingSettingsPage.razor` | إعدادات المحادثات |

#### 🌿 فئة برامج الدعم (PSP) — 8 مكونات

| المكون | الوظيفة |
|--------|---------|
| `PspAboutPage.razor` | عن برامج الدعم |
| `PspGateway.razor` | بوابة PSP |
| `PspSearch.razor` | بحث برامج |
| `PspEntry.razor` | دخول برنامج |
| `PspScheduleSetup.razor` | إعداد جدولة |
| `PspDetailComponent.razor` | تفاصيل برنامج |
| `ProgramDetailsComponent.razor` | تفاصيل برنامج (طبيب) |
| **`PointDashboard.razor`** ⭐ | **محفظة النقاط (جديد)** |

#### 🏢 إدارة المنظمات (OrganizationManagement) — 4 مكونات

| المكون | الوظيفة |
|--------|---------|
| `CreateOrganizationPage.razor` | إنشاء منظمة |
| `MembersTab.razor` | تبويب الأعضاء |
| `MemberTitlesTab.razor` | ألقاب الأعضاء |
| `CustomJobTitles.razor` | المسميات الوظيفية |

#### ⚖️ الفئة القانونية (Legal) — 2 مكونات

| المكون | الوظيفة |
|--------|---------|
| `PrivacyPolicy.razor` | سياسة الخصوصية |
| `TermsOfService.razor` | شروط الاستخدام |

#### 🛠️ مكونات مساعدة مشتركة — 7 مكونات

| المكون | الوظيفة |
|--------|---------|
| `LoaderOverlay.razor` | شاشة تحميل |
| `RubikButton.razor` | زر موحد |
| `TestPage.razor` | صفحة اختبار |
| `Pagination.razor` | ترقيم الصفحات |
| `SearchBar.razor` | شريط بحث |
| `RubikSmartTable.razor` | جدول ذكي |
| `SupportPage.razor` | صفحة دعم |

### 13.4 خدمات وواجهات Shared.UI (25 ملف)

| الخدمة/الواجهة | النوع | الحالة |
|-----------------|-------|--------|
| `IApiService.cs` | واجهة | ✅ |
| `ITranslationService.cs` | واجهة | ✅ |
| `ITranslationState.cs` | واجهة | ✅ |
| `CurrentOrganizationState.cs` | حالة | ✅ |
| `IAppStateService.cs` | واجهة | ⚠️ |
| `IPatientSessionService.cs` | واجهة | ⚠️ |
| `PatientSessionService.cs` | تنفيذ | ⚠️ |
| `SharedApiService.cs` | تنفيذ | ✅ |
| `ILocalNotificationService.cs` | واجهة | ✅ |
| `IMobileNavigationService.cs` | واجهة | ⚠️ |
| `INearbyPharmacyService.cs` | واجهة | ⚠️ |
| `INotificationNavigationService.cs` | واجهة | ✅ |
| `IPspUiService.cs` | واجهة | ✅ |
| `IPublicApiService.cs` | واجهة | ⚠️ |
| `InviteNavigationBridge.cs` | جسر | ✅ |
| `RepNavigationBridge.cs` | جسر | ✅ |
| `PspGatewayNavigationBridge.cs` | جسر | ✅ |
| **`PointDashboardNavigationBridge.cs`** ⭐ | **جسر (جديد)** | ✅ |
| `OsmPharmacyService.cs` | تنفيذ | ⚠️ |
| `PendingRequestService.cs` | خدمة | ⚠️ |
| `Models/NotificationItem.cs` | نموذج | ✅ |
| `Models/PspModels.cs` | نموذج | ✅ |
| `Models/LicenseSubmitData.cs` | نموذج | ✅ |
| `Models/MedicationRequestStatus.cs` | نموذج | ✅ |
| **`Models/PointModels.cs`** ⭐ | **نموذج (جديد)** | ✅ |

### 13.5 حالة الـ PWA — نسبة الإنجاز

| العنصر | النسبة |
|--------|:------:|
| **البنية التحتية** | ✅ 100% |
| **صفحات المصادقة** | ✅ 100% |
| **صفحات PSP** | ✅ 100% |
| **صفحات المريض** | ✅ 100% |
| **صفحات الطبيب** | ✅ 100% |
| **صفحات الصيدلية** | ✅ 100% |
| **صفحات المندوب** | ✅ 100% |
| **نظام النقاط** | ✅ 100% |
| **نظام المحادثات** | ✅ 90% |
| **الإشعارات Push** | ✅ 100% |
| **Google Login** | ✅ 100% |
| **الإجمالي** | **~95%** |

### 13.6 جرد نظام النقاط

#### 13.6.1 الموقع في المشاريع الثلاثة

| الطبقة | المكوّن | العدد | الموقع |
|--------|---------|:-----:|--------|
| **Domain** | كيانات | 8 | `Entities/PSP/Points/` |
| **Domain** | Lookups | 3 | `Entities/PSP/Points/Lookups/` |
| **Domain** | Constants | 3 | `Constants/PSP/Points/` |
| **Application** | Services | 2 | `Services/PSP/` |
| **Infrastructure** | Handlers | 19 | `UseCases/PSP/Points/` |
| **Api.Web** | Controllers | 5 | `Controllers/PSP/Points/` |
| **Shared.UI** | Components | 1 | `Components/PSP/Points/` |
| **Shared.UI** | Bridge | 1 | `Components/PSP/Points/` |
| **Shared.UI** | Models | 1 | `Components/PSP/Points/` |
| **Web** | Razor | 1 | `Components/Pages/Professional/PSPSteps/` |
| **PWA** | Page | 1 | `Pages/PSP/Points/` |
| **MAUI** | Container | 1 | `Features/PSP/Doctor/Views/` |

#### 13.6.2 الكيانات (Domain) — 8

| # | الكيان | الغرض |
|---|--------|-------|
| 1 | `PSPActivityType` | Lookup — أنواع الأحداث (7) |
| 2 | `PSPProgramPointTransactionType` | Lookup — أنواع المعاملات (4) |
| 3 | `PSPProgramPointRedeemStatus` | Lookup — حالات Redeem (4) |
| 4 | `PSPProgramPointScheme` | نظام النقاط لكل برنامج |
| 5 | `PSPProgramPointRule` | قواعد المنح |
| 6 | `PSPProgramPointBalance` | أرصدة العيادات |
| 7 | `PSPProgramPointTransaction` | سجل المعاملات |
| 8 | `PSPProgramPointRedeemRequest` | طلبات الـ Redeem |

#### 13.6.3 Handlers (Infrastructure) — 19

| المجموعة | العدد | الملفات |
|----------|:-----:|---------|
| **Schemes** | 4 | `CreatePointSchemeHandler`, `UpdatePointSchemeHandler`, `GetPointSchemeByProgramHandler`, `DeactivatePointSchemeHandler` |
| **Rules** | 5 | `CreatePointRuleHandler`, `UpdatePointRuleHandler`, `DeletePointRuleHandler`, `GetPointRulesBySchemeHandler`, `TogglePointRuleHandler` |
| **Balances** | 3 | `GetBalanceByOrganizationHandler`, `GetAllBalancesBySchemeHandler`, `GetOrganizationPointSummaryHandler` |
| **Earning** | 2 | **`EarnPointsHandler`** ⭐, `GetPointTransactionsHandler` |
| **Redeem** | 5 | `CreateRedeemRequestHandler` ⭐, `AcceptRedeemRequestHandler` ⭐, `RejectRedeemRequestHandler`, `GetRedeemRequestsHandler`, `GetRedeemRequestDetailsHandler` |

**⚠️ الـ Transaction مُطبَّق في:** `EarnPointsHandler`, `CreateRedeemRequestHandler`, `AcceptRedeemRequestHandler`.

#### 13.6.4 Controllers (Api.Web) — 5

| Controller | Endpoints |
|-----------|:---------:|
| `PointSchemesController` | 4 |
| `PointRulesController` | 5 |
| `PointBalancesController` | 3 |
| `PointEarningController` | 1 |
| `PointRedeemController` | 5 |

#### 13.6.5 ربط الأحداث — 5 من 7

| # | الحدث | الموقع | الحالة |
|---|-------|--------|:------:|
| 1 | `PATIENT_INVITED` | `PSPEnrollmentService` | ✅ |
| 2 | `PATIENT_ENROLLED` | `PSPEnrollmentService` | ✅ |
| 3 | `ERX_CREATED` | `PSPEnrollmentService` | ✅ |
| 4 | `DISPENSATION_COMPLETED` | `DispenseController` | ✅ |
| 5 | `REFILL_COMPLETED` | `DispenseController` | ✅ |
| 6 | `PROGRAM_COMPLETED` | مؤجل | ⏸️ |
| 7 | `LAB_TEST_UPLOADED` | مؤجل | ⏸️ |

---

## 🎯 الجزء الرابع عشر: Reusable Services (الرؤية الأفقية)

### 14.1 المفهوم

> **"كل ميزة = خدمة مستقلة قابلة لإعادة الاستخدام."**

**⚠️ من `08-Strategic-Roadmap`:**

- **التنبيهات**
- **الإشعارات**
- **الترجمة**
- **المحادثات**
- **نظام النقاط**
- **AI** (لاحقاً)
- **الفواتير + الطلبات + المخزون + الفوترة + Checkout** (لاحقاً)

### 14.2 أمثلة حية

| الخدمة | الحالة | الموقع |
|--------|:------:|--------|
| **`IPointEngineService`** | ✅ منفّذ | `Application + Infrastructure` |
| **`INotificationService`** | ✅ منفّذ | `Application + Infrastructure` |
| **`ITranslationService`** | ✅ منفّذ | `Application + Infrastructure` |
| **`IMessagingService`** | ✅ منفّذ | `Application + Infrastructure` |
| **`IAIService`** | ⏸️ مستقبلاً | — |
| **`IInvoicingService`** | ⏸️ مستقبل | — |

### 14.3 كيف تبني Reusable Service؟

**⚠️ الخطوات:**

1. **`Domain`** — كيان + Enums.
2. **`Application`** — `IXxxService` + DTOs.
3. **`Infrastructure`** — `XxxService` (Implementation).
4. **`Api.Web`** — Controller (Thin).
5. **`Shared.UI`** — Components (اختياري).
6. **`Client`** — `IXxxClient` (اختياري).

---

## 🔧 الجزء الخامس عشر: EF Core Migration — ملاحظات حرجة

### 15.1 المبدأ الأساسي

> **"Code-First إلزامي. لا تعديل يدوي على Migration files."**

### 15.2 القاعدة الصارمة

> **"قبل أي `Add-Migration` — تأكد أن Snapshot = DB."**

**⚠️ EF Core لا يفحص DB — بل يثق أن Snapshot يعكس DB.**

### 15.3 الممنوعات المطلقة

| # | الممنوع | السبب |
|:-:|:---|:---|
| 1 | تعديل Migration files يدوياً | تُلغى عند أول `Add-Migration` |
| 2 | تعديل `ModelSnapshot.cs` كروتين | يفقد الاتساق |
| 3 | SQL مباشر لتعديل DB — كبديل | يخلق فجوة |
| 4 | حذف Migration file من DB بدون `Remove-Migration` | يكسر `__EFMigrationsHistory` |
| 5 | تعديل DB دون Migration — كروتين | يُفسد التوازن |

### 15.4 الاستثناءات المسموحة

| # | الاستثناء | الشرط |
|:-:|:---|:---|
| 1 | **SQL مباشر للبيانات** (Data Migration) | عند نقل بيانات |
| 2 | **تعديل Snapshot يدوياً** | لمرة واحدة — مع تحقق فوري |
| 3 | **`Remove-Migration`** | لحذف Migration غير مُطبَّق |
| 4 | **Backup DB** | قبل أي Migration حساس |

### 15.5 Checklist قبل أي Migration

- [ ] Backup DB كامل.
- [ ] Snapshot = DB؟
- [ ] `OnModelCreating` يُطابق Entity؟
- [ ] `Add-Migration` — مراجعة Migration المُولَّد.
- [ ] `Update-Database` — مراقبة الأخطاء.

---

## 🔗 الجزء السادس عشر: روابط ذات صلة

### 📚 وثائق أساسية

- [00 — MASTER-INDEX](./00-MASTER-INDEX.md)
- [00B — PROJECT-BACKLOG](./00B-PROJECT-BACKLOG.md)
- [02 — IDENTITY](./02-IDENTITY.md)
- [03 — MENUS](./03-MENUS.md)
- [04 — TRANSLATION](./04-TRANSLATION.md)
- [09 — API-GUIDE](./09-API-GUIDE.md)
- [11 — PAGE-CREATION](./11-PAGE-CREATION.md)
- [13 — MAUI](./13-MAUI.md)
- [14 — PWA](./14-PWA.md)
- [16 — TROUBLESHOOTING](./16-TROUBLESHOOTING.md)
- [17 — STRATEGIC-ROADMAP](./17-STRATEGIC-ROADMAP.md)
- [19 — PERMISSIONS](./19-PERMISSIONS.md)

---

## 📝 الجزء السابع عشر: سجل التغييرات

| الإصدار | التاريخ | التغييرات |
|:---:|:---|:---|
| **1.0** | **18 يوليو 2026** | الإصدار الأولي — `01-program-cs-foundation.md` |
| **1.0** | **3 أكتوبر 2026** | الإصدار الأولي — `00-architecture-overview.md` |
| **1.0** | **3 أكتوبر 2026** | الإصدار الأولي — `24-project-structure-inventory.md` |
| **2.0** | **16 أكتوبر 2026** | **`01-ARCHITECTURE` v2.0** — دمج `00` + `01` + `24` — مع الحفاظ على كل التفاصيل + إضافة Reusable Services + EF Core ملاحظات |

---

**© 2026 RubikCare — للاستخدام الداخلي**

**نهاية الوثيقة ✅**
```

---
