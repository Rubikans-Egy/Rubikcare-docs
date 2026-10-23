# 13 — MAUI (تطوير MAUI + BlazorWebView)

**الإصدار:** 2.0 (Unified Edition)
**آخر تحديث:** 17 أكتوبر 2026
**الحالة:** ✅ مرجع شامل — يدمج 10 + 11
**الجمهور:** فريق التطوير + AI Assistants
**المرجع الإلزامي:** يُقرأ قبل أي تطوير لصفحة MAUI

---

## 📌 الجزء صفر: الفلسفة

### 0.1 لماذا BlazorWebView؟

| الميزة | XAML التقليدي | BlazorWebView |
|--------|---------------|---------------|
| **لغة التطوير** | لغة مختلفة عن Web | نفس HTML/CSS المستخدم في Web |
| **ربط البيانات** | مشاكل Binding متكررة | لا Binding - استخدام Razor |
| **مشاركة المكونات** | صعبة | مشاركة مباشرة عبر RCL |
| **منحنى التعلم** | منفصل تماماً | نفس مهارات Blazor |

### 0.2 المبادئ الأساسية

1. **مشاركة المكونات** — `Shared.UI` مشترك بين Web + PWA + MAUI.
2. **CustomBlazorWebView** — استخدمه دائماً (لا `BlazorWebView` العادي).
3. **Action بدلاً من EventCallback** — نمط MAUI.
4. **لا `@inject` للخدمات MAUI** — استخدم `[Parameter]`.
5. **`SafeAsyncAction`** — لكل `Action<T>` مع `async`.

---

## 🏗️ الجزء الأول: هيكل مشروع MAUI

### 1.1 الهيكل العام

```
RubikCare.Mobile/
├── App.xaml / App.xaml.cs              # نقطة بداية التطبيق
├── AppShell.xaml / AppShell.xaml.cs    # التنقل الرئيسي (Shell)
├── MauiProgram.cs                      # تسجيل الخدمات
│
├── Controls/
│   └── CustomBlazorWebView.cs          # ⭐ Wrapper لكل BlazorWebView
│
├── Features/                           # الميزات مقسمة حسب المجال
│   ├── Auth/
│   │   ├── Views/
│   │   │   ├── LoginView.xaml
│   │   │   └── RegisterPage.xaml
│   │   └── ViewModels/
│   │
│   ├── PSP/
│   │   ├── Doctor/
│   │   │   ├── Views/
│   │   │   │   ├── PspSearchPage.xaml
│   │   │   │   ├── ProgramDetailsPage.xaml     # نمط 5 (Hybrid)
│   │   │   │   ├── PointsDashboardPage.xaml    # نمط 2 (QueryProperty string)
│   │   │   │   └── InvitePatientPage.xaml
│   │   │   └── ViewModels/
│   │   ├── Patient/
│   │   │   ├── Views/
│   │   │   │   ├── PspEntryPage.xaml
│   │   │   │   └── PspDetailPage.xaml
│   │   │   └── ViewModels/
│   │   └── Pharmacist/
│   │       ├── Views/
│   │       │   ├── ScanTokenPage.xaml
│   │       │   └── VerifyTokenPage.xaml
│   │       └── ViewModels/
│   │
│   ├── PublicUser/
│   │   ├── Views/
│   │   │   ├── DashboardPage.xaml
│   │   │   └── ProfilePage.xaml
│   │   └── ViewModels/
│   │
│   ├── Shared/Views/                    # حاويات MAUI للصفحات المشتركة
│   │   ├── PharmacySearchFlow.xaml
│   │   ├── PharmacyDetailFlow.xaml
│   │   └── ...
│   │
│   └── ProfessionalOnboarding/Views/
│       ├── ProfessionalStatusFlow.xaml
│       └── ProfessionalLicenseFlow.xaml
│
├── Infrastructure/
│   └── Services/
│       ├── ApiService.cs               # التواصل مع Api.Web
│       ├── AuthService.cs              # المصادقة وإدارة التوكن
│       ├── CachedUserSessionService.cs # ⭐ كاش الجلسة المحلي
│       └── AppStateService.cs          # حالة التطبيق المشتركة
│
├── ViewModels/
│   └── AppShellViewModel.cs            # ⭐ ViewModel القائمة الجانبية
│
├── Converters/                         # محولات XAML
├── Resources/                          # موارد التطبيق
│   ├── Fonts/
│   ├── Images/
│   ├── Styles/
│   └── raw/                            # ⭐ Android Raw Resources
│
└── Platforms/                          # كود خاص بكل منصة
    ├── Android/
    └── iOS/
```

### 1.2 ملف `AppConfig.cs`

```csharp
public static class AppConfig
{
#if DEBUG
    public static string ApiBaseUrl = "http://localhost:5235";
#else
    public static string ApiBaseUrl = "https://api.rubikcare.com";
#endif

    public static int RequestTimeout = 30; // ثواني
    public static int MaxRetryAttempts = 3;
}
```

---

## 🚨 الجزء الثاني: مشكلة Dependency Injection (DI)

### 2.1 المشكلة الجذرية (اكتشاف: 13 يونيو 2026)

**`BlazorWebView` في MAUI يستخدم حاوية DI مستقلة عن حاوية MAUI الرئيسية.** هذا يعني أن الخدمات المسجلة في `MauiProgram.cs` **ليست متاحة تلقائياً** لمكونات Blazor الداخلية.

### 2.2 ❌ العرض: فشل صامت للمكون

**⚠️ عندما يستخدم مكون Blazor `@inject` لخدمة غير مسجلة في حاوية Blazor الداخلية، يفشل المكون في التحميل تماماً بدون أي خطأ في Debug Output. الصفحة تظهر فارغة أو لا تظهر على الإطلاق.**

> **⚠️ تحذير:** هذا الفشل **صامت تماماً** — لا exception، لا خطأ في الـ Console، لا أي مؤشر. `OnInitializedAsync` لا تُستدعى أبداً.

### 2.3 📊 الخدمات التي تعمل وتفشل مع `@inject`

| الخدمة | `@inject` في Blazor | التفسير |
|--------|---------------------|---------|
| `ISharedTranslationService` | ✅ يعمل | مسجلة في حاوية Blazor |
| `ISharedTranslationState` | ✅ يعمل | مسجلة في حاوية Blazor |
| `NavigationManager` | ✅ يعمل | مسجلة تلقائياً من Blazor runtime |
| `HttpClient` | ❌ يفشل بصمت | غير مسجلة في حاوية Blazor |
| `ApiService` | ❌ يفشل بصمت | غير مسجلة في حاوية Blazor |
| `IMobileTranslationService` | ❌ يفشل بصمت | غير مسجلة في حاوية Blazor |

### 2.4 ✅ الحل: تمرير الخدمات كـ Parameters

**في مكون Blazor (`.razor`):**

```razor
@* ❌ خطأ - سيفشل بصمت في MAUI *@
@inject HttpClient Http
@inject ApiService ApiService

@* ✅ صحيح - استخدم Parameter *@
@using RubikCare.Shared.UI.Services

@code {
    [Parameter] public IApiService? ApiService { get; set; }
}
```

**في صفحة MAUI المضيفة (`.xaml.cs`):**

```csharp
private readonly ApiService _apiService;

public MyFlowPage()
{
    InitializeComponent();
    _apiService = IPlatformApplication.Current!.Services
        .GetRequiredService<ApiService>();

    var parameters = new Dictionary<string, object?>
    {
        { "ApiService", (IApiService)_apiService }
    };

    blazorWebView.RootComponents.Add(new RootComponent
    {
        Selector = "#app",
        ComponentType = typeof(MyBlazorPage),
        Parameters = parameters
    });
}
```

### 2.5 🔍 Checklist تشخيص مشكلة DI

عندما يفشل مكون Blazor في التحميل داخل MAUI:

- [ ] هل المكون يعمل في Web Blazor لكن ليس في MAUI؟
- [ ] هل تستخدم `@inject HttpClient`؟ → **احذفه فوراً**.
- [ ] هل تستخدم `@inject` لأي خدمة غير `ISharedTranslation*` أو `NavigationManager`؟
- [ ] هل `OnInitializedAsync` لا تُستدعى (لا يظهر Debug Output)؟
- [ ] هل الصفحة فارغة تماماً بدون أي خطأ؟

**إذا أجبت بنعم على أي سؤال، استبدل `@inject` بـ `[Parameter]`.**

---

## 🚨 الجزء الثالث: تسجيل صفحات BlazorWebView — قاعدة ذهبية

**⚠️ عند إنشاء صفحة MAUI جديدة تحتوي على `BlazorWebView`:**

### 3.1 الخطوة 1: `MauiProgram.cs`

**⚠️ لا تسجل الصفحة بـ `AddTransient` — هذا يسبب انهيار Android عند Splash!**

### 3.2 الخطوة 2: `AppShell.xaml.cs`

```csharp
Routing.RegisterRoute("PageName", typeof(PageName));
```

### 3.3 نمط XAML الصحيح

```xml
<webview:BlazorWebView x:Name="blazorWebView" HostPage="wwwroot/index.html" />
```

**⚠️ بدون `RootComponent` في XAML.**

### 3.4 نمط Code-behind الصحيح

```csharp
private bool _isLoaded;

protected override void OnAppearing()
{
    base.OnAppearing();
    if (_isLoaded) return;
    _isLoaded = true;

    blazorWebView.RootComponents.Clear();
    blazorWebView.RootComponents.Add(new RootComponent
    {
        Selector = "#app",
        ComponentType = typeof(MyComponent),
        Parameters = new Dictionary<string, object?> { { "Param", value } }
    });
}

protected override void OnDisappearing()
{
    base.OnDisappearing();
    _isLoaded = false;
    blazorWebView.RootComponents.Clear();
}
```

### 3.5 ⚠️ الخطأ الشائع: `jumpToEnd`

**إذا ظهر خطأ `No view found for id 0x7f0800ff (jumpToEnd)`:**

1. **احذف `AddTransient<PageName>()`** من `MauiProgram.cs`.
2. **قم بـ Clean + حذف `bin/obj` + إعادة بناء**.

---

## 🚨 الجزء الرابع: قيود RootComponents في BlazorWebView

### 4.1 ⚠️ السبب الجذري

**`BlazorWebView` على Windows يستخدم `WebView2` كمحرك. هذا المحرك يربط الـ DOM بمكونات Blazor فقط أثناء دورة حياة الكونستركتور للـ `ContentPage` المضيفة.**

**⚠️ استثناء مهم:** في بيئة المشروع الفعلية، `RootComponents.Clear()` ثم `Add` **يعمل بنجاح** في بعض السيناريوهات (خاصة عند استخدام `CustomBlazorWebView`). هذا مُوثَّق في نمط "QueryProperty + Setter" أدناه.

### 4.2 ❌ أنماط فاشلة موثقة

```csharp
// ❌ النمط 1: async في الكونستركتور
public MyFlowPage()
{
    InitializeComponent();
    _ = LoadAndShowAsync(); // Task تنتهي بعد الكونستركتور — لن يعمل
}

// ❌ النمط 2: في OnAppearing (للمكونات التي لم تُضف في Constructor)
protected override void OnAppearing()
{
    base.OnAppearing();
    blazorWebView.RootComponents.Add(...); // ← خارج الكونستركتور
}

// ❌ النمط 3: Clear ثم Add بعد تحميل البيانات (بدون CustomBlazorWebView)
private async Task RefreshWithData()
{
    var data = await apiService.GetDataAsync();
    blazorWebView.RootComponents.Clear();
    blazorWebView.RootComponents.Add(...); // ← لن يُعاد تحميل المكون
}
```

### 4.3 ✅ النمط الصحيح (الأساسي)

```csharp
// ✅ RootComponents.Add بشكل متزامن في الكونستركتور
public MyFlowPage()
{
    InitializeComponent();
    _apiService = IPlatformApplication.Current!.Services
        .GetRequiredService<ApiService>();

    // 1. بيانات محلية (متزامن)
    LoadCachedData();

    // 2. إضافة المكون (متزامن - إلزامي في الكونستركتور)
    LoadBlazorComponent();

    // 3. تحديث من API (غير متزامن - لا يعيد تحميل المكون)
    _ = RefreshFromApiAsync();
}
```

---

## 🚨 الجزء الخامس: فخ `[QueryProperty]` مع Nullable Types

### 5.1 ⚠️ المشكلة (اكتشاف: 8 أكتوبر 2026)

**عند استخدام `[QueryProperty]` مع خاصية من نوع `int?` (أو أي `Nullable<T>`)، يفشل MAUI Shell في تحويل القيمة القادمة (String) إلى `Nullable<int>` — ويظهر الخطأ التالي:**

```
System.InvalidCastException: Invalid cast from 'System.String' to 
'System.Nullable`1[[System.Int32, System.Private.CoreLib, Version=10.0.0.0, 
Culture=neutral, PublicKeyToken=7cec85d7bea7798e]]'.
```

### 5.2 🎯 السبب الجذري

**`[QueryProperty]` يستخدم `Convert.ChangeType` داخلياً، وهذا لا يدعم `Nullable<T>`.** عندما يمرر Shell القيمة `"1"` (String) إلى خاصية `int?`، يفشل التحويل.

### 5.3 ❌ الكود الفاشل

```csharp
[QueryProperty(nameof(ProgramId), "programId")]
public partial class PointsDashboardPage : ContentPage
{
    public int? ProgramId { get; set; }   // ❌ يفشل بـ InvalidCastException
}
```

### 5.4 ✅ الحل: استقبل كـ String ثم حوّل يدوياً

```csharp
[QueryProperty(nameof(ProgramIdRaw), "programId")]
public partial class PointsDashboardPage : ContentPage
{
    // ⭐ نستقبلها كـ String
    public string? ProgramIdRaw { get; set; }

    // ⭐ خاصية محسوبة (Computed Property) للتحويل اليدوي
    private int? ProgramId => int.TryParse(ProgramIdRaw, out var id) ? id : null;
}
```

### 5.5 📌 القاعدة الذهبية

> **`[QueryProperty]` يستخدم `Convert.ChangeType` داخلياً، وهذا لا يدعم `Nullable<T>`.**
>
> **دائماً استقبل QueryProperty كـ `string`، ثم حوّل يدوياً.**

### 5.6 📊 جدول: أنواع Property ونتائجها

| النوع | يعمل؟ | ملاحظات |
|-------|:---:|---------|
| `string` | ✅ | الأكثر أماناً — استخدمه دائماً |
| `string?` | ✅ | يعمل (مع nullable reference types) |
| `int` (non-nullable) | ✅ | يعمل، لكن يفشل إذا كانت القيمة فارغة |
| `int?` (nullable) | ❌ | **يفشل بـ InvalidCastException** |
| `DateTime?` | ❌ | نفس المشكلة |
| `bool` (non-nullable) | ✅ | يعمل |
| `bool?` (nullable) | ❌ | نفس المشكلة |

### 5.7 ⚠️ فخ إضافي: `[QueryProperty]` يُعيَّن بعد Constructor

**المشكلة:**

```
Constructor (LoadBlazorComponent) → QueryProperty setter → OnAppearing
                                       ↑
                          ⚠️ يُعيَّن هنا، بعد أن يكون المكون قد بُني
```

**النتيجة:** إذا استدعيت `LoadBlazorComponent()` في Constructor، فإن `ProgramId` سيكون `null` أو `0` عند بناء المكون.

**الحل:** انقل `LoadBlazorComponent()` إلى `OnAppearing` (مع flag `_componentLoaded`):

```csharp
private bool _componentLoaded = false;

protected override void OnAppearing()
{
    base.OnAppearing();
    if (!_componentLoaded)
    {
        LoadBlazorComponent();
        _componentLoaded = true;
    }
}
```

**هذا يضمن أن `QueryProperty` تم تعيينه قبل بناء المكون.**

### 5.8 📊 متى تستخدم `IQueryAttributable` بدلاً من `[QueryProperty]`؟

| الحالة | `[QueryProperty]` | `IQueryAttributable` |
|--------|:------------------:|:-------------------:|
| 1-2 معاملات | ✅ مُوصى | يعمل |
| 3+ معاملات | ❌ وميض | ✅ مُوصى |
| `Nullable<T>` | ❌ يحتاج حيلة (string) | ✅ يدعم أصلاً |
| إعادة إنشاء المكون | لا | لا |
| `LoadBlazorComponent` في Constructor | ✅ | يحتاج `_queryReady` flag |
| ترتيب الاستدعاء | Setter في Constructor | `ApplyQueryAttributes` (بعد Constructor، قبل OnAppearing) |

**القاعدة:**
- `[QueryProperty]` **للبساطة** (1-2 معاملات، `string`).
- `IQueryAttributable` **للمرونة** (3+ معاملات، `Nullable<T>`، تحكم كامل).

**⚠️ ملاحظة:** النمط 2 (`PharmacyDetailFlow`) يعتمد على `[QueryProperty]` لتفعيل الـ Setter — وهو مُتبع ومستقر. **لكن لا يستخدمه مع `Nullable<T>`.**

---

## 🎯 الجزء السادس: الأنماط الأربعة الفعلية في المشروع

**النمط الأساسي أعلاه هو نقطة البداية، لكن المشروع طوّر 4 أنماط متمايزة تناسب سيناريوهات مختلفة. كلها مُتبعة فعلياً في المشروع، ولكل منها استخدامه الصحيح.**

### 6.1 🌟 النمط 1: Constructor + LoadComponent (الأساسي — مستقر)

**متى يُستخدم:**
- عند **عدم وجود معاملات** أو وجود معاملات بسيطة جداً.
- عند الحاجة لتحميل المكون **مرة واحدة** عند فتح الصفحة.

**المثال:** `PharmacySearchFlow.xaml.cs`

```csharp
public partial class MyFlowPage : ContentPage
{
    public MyFlowPage()
    {
        InitializeComponent();
        _apiService = IPlatformApplication.Current!.Services
            .GetRequiredService<ApiService>();

        LoadBlazorComponent();  // ⭐ في الـ Constructor
    }

    private void LoadBlazorComponent()
    {
        blazorWebView.RootComponents.Clear();
        blazorWebView.RootComponents.Add(new RootComponent
        {
            Selector = "#app",
            ComponentType = typeof(MyBlazorComponent),
            Parameters = new Dictionary<string, object?>
            {
                { "ApiService", (IApiService)_apiService }
            }
        });
    }

    protected override void OnDisappearing()
    {
        base.OnDisappearing();
        try { blazorWebView.RootComponents.Clear(); } catch { }
    }
}
```

**المزايا:**
- ✅ الأكثر استقراراً وبساطة.
- ✅ لا وميض (المكون يُضاف مرة واحدة).
- ✅ الأداء الأمثل.

**العيوب:**
- ⚠️ لا يستقبل `QueryProperty` تلقائياً.
- ⚠️ لا يُحدَّث عند تغيير الصفحة (يحتاج `OnAppearing` مع `_isLoaded` flag).

### 6.2 🌟 النمط 2: QueryProperty + Setter → LoadComponent (للمعاملات القليلة)

**متى يُستخدم:**
- عند وجود **1-2 معاملات** فقط (`QueryProperty`).
- عندما لا تكون البيانات ثقيلة جداً.
- **⚠️ لا يُنصح به لأكثر من 2-3 معاملات** (يسبب وميض).
- **⚠️ لا تستخدمه مع `Nullable<T>`** — استخدم `string` + تحويل يدوي.

**المثال:** `PharmacyDetailFlow.xaml.cs`

```csharp
[QueryProperty(nameof(PharmacyId), "pharmacyId")]
[QueryProperty(nameof(TokenCode), "token")]
public partial class PharmacyDetailFlow : ContentPage, IBackButtonHandler
{
    private int _pharmacyId;
    public int PharmacyId
    {
        get => _pharmacyId;
        set
        {
            _pharmacyId = value;
            LoadComponent();   // ⭐ إعادة تحميل المكون في الـ Setter
        }
    }

    private string? _tokenCode;
    public string? TokenCode
    {
        get => _tokenCode;
        set
        {
            _tokenCode = value;
            LoadComponent();   // ⭐ إعادة تحميل المكون في الـ Setter
        }
    }

    public PharmacyDetailFlow()
    {
        InitializeComponent();  // لا شيء آخر هنا
    }

    private void LoadComponent()
    {
        blazorWebView.RootComponents.Clear();
        blazorWebView.RootComponents.Add(new RootComponent
        {
            Selector = "#app",
            ComponentType = typeof(PharmacyDetailPage),
            Parameters = new Dictionary<string, object?>
            {
                { "PharmacyId", _pharmacyId },
                { "TokenCode", _tokenCode ?? "" },
                { "OnGoBackAction", new Action(() => GoBack()) }
            }
        });
    }
}
```

**المزايا:**
- ✅ يتعامل مع `QueryProperty` تلقائياً.
- ✅ **يعمل بثبات في المشروع** — خاصة مع `CustomBlazorWebView`.
- ✅ مكونات Blazor تعيد التحميل تلقائياً عند وصول المعاملات.

**العيوب:**
- ⚠️ **يسبب وميضاً** عند إعادة التحميل.
- ⚠️ **مع 3+ معاملات**: `LoadComponent()` تُستدعى 3+ مرات → وميض شديد + طلبات API متكررة.
- ⚠️ **يحتاج `CustomBlazorWebView`** ليعمل بثبات.

### 6.3 🌟 النمط 3: Constructor + Refresh in Background (مع Cache)

**متى يُستخدم:**
- عند وجود **بيانات محلية** (Preferences/SecureStorage) يمكن عرضها فوراً.
- عند الحاجة لتحديث شفاف في الخلفية.

**المثال:** `ProfessionalStatusFlow.xaml.cs`

```csharp
public partial class ProfessionalStatusFlow : ContentPage
{
    private static string _statusCode = "";
    private static string _professionalRole = "";

    public ProfessionalStatusFlow()
    {
        InitializeComponent();
        _apiService = IPlatformApplication.Current!.Services
            .GetRequiredService<ApiService>();

        // 1. تحميل البيانات من الذاكرة المحلية
        LoadCachedData();

        // 2. تحميل UI فوراً
        LoadProfessionalStatusPage();

        // 3. تحديث من API في الخلفية
        _ = RefreshFromApiAsync();
    }

    private void LoadCachedData()
    {
        _statusCode = Preferences.Get("license_status_code", "");
        _professionalRole = Preferences.Get("license_role_type", "");
    }

    private async Task RefreshFromApiAsync()
    {
        var session = await _cachedSessionService.GetSessionAsync();
        Preferences.Set("license_status_code", newStatusCode);

        if (changed)
        {
            await MainThread.InvokeOnMainThreadAsync(() =>
            {
                LoadProfessionalStatusPage();  // ← Clear + Add
            });
        }
    }
}
```

**المزايا:**
- ✅ **لا وميض** — UI يظهر فوراً من الـ Cache.
- ✅ تحديث شفاف.
- ✅ تجربة مستخدم ممتازة.

**العيوب:**
- ⚠️ يستخدم `static` variables (ليس نظيفاً معمارياً).
- ⚠️ يعيد إنشاء `RootComponent` عند التحديث (قد يسبب وميضاً بسيطاً).
- ⚠️ يحتاج `OnDisappearing` للتنظيف.

### 6.4 🌟 النمط 4: Constructor + OnComponentReady (للتفاعل MAUI ↔ Component)

**متى يُستخدم:**
- عندما يحتاج **MAUI لاستدعاء methods** على المكون (مثل FilePicker).
- عند وجود **تفاعل مستمر** بين MAUI والمكوّن.

**المثال:** `ProfessionalLicenseFlow.xaml.cs`

```csharp
public partial class ProfessionalLicenseFlow : ContentPage
{
    private ProfessionalLicensePage? _licensePageRef;

    public ProfessionalLicenseFlow()
    {
        InitializeComponent();
        _apiService = IPlatformApplication.Current!.Services
            .GetRequiredService<ApiService>();

        LoadLicensePage();  // في Constructor
    }

    private void LoadLicensePage()
    {
        blazorWebView.RootComponents.Clear();
        blazorWebView.RootComponents.Add(new RootComponent
        {
            Selector = "#app",
            ComponentType = typeof(ProfessionalLicensePage),
            Parameters = new Dictionary<string, object?>
            {
                { "ApiService", (IApiService)_apiService },
                { "OnPickFileRequested", new Action(async () => await PickFileAndUpdateAsync()) },
                { "OnComponentReady", new Action<ProfessionalLicensePage>(page =>
                    {
                        _licensePageRef = page;
                        Debug.WriteLine("✅ LicensePage ref captured");
                    })
                }
            }
        });
    }

    private async Task PickFileAndUpdateAsync()
    {
        var result = await FilePicker.PickAsync(...);
        if (result == null) return;

        using var stream = await result.OpenReadAsync();
        using var ms = new MemoryStream();
        await stream.CopyToAsync(ms);
        var bytes = ms.ToArray();
        var fileName = result.FileName;

        await MainThread.InvokeOnMainThreadAsync(() =>
        {
            _licensePageRef?.SetFileData(fileName, bytes);
        });
    }

    protected override void OnDisappearing()
    {
        base.OnDisappearing();
        _licensePageRef = null;
        try { blazorWebView.RootComponents.Clear(); } catch { }
    }
}
```

**في المكون (`.razor`):**

```razor
@code {
    [Parameter] public Action<MyComponent>? OnComponentReady { get; set; }

    protected override void OnAfterRender(bool firstRender)
    {
        if (firstRender)
        {
            OnComponentReady?.Invoke(this);
        }
    }

    public void SetFileData(string fileName, byte[] fileBytes)
    {
        _fileName = fileName;
        _selectedFileBytes = fileBytes;
        InvokeAsync(StateHasChanged);
    }
}
```

**المزايا:**
- ✅ يسمح بـ **التفاعل MAUI ↔ Component** بدون إعادة تحميل.
- ✅ لا وميض.
- ✅ يحافظ على State المكون.

**العيوب:**
- ⚠️ يتطلب `public methods` على المكون.
- ⚠️ يحتاج `Dispose` لتنظيف المرجع.
- ⚠️ `OnComponentReady` يُستدعى مرة واحدة فقط عند `firstRender`.

---

# [#68] 📘 `13-MAUI` — v2.0 (Part 2)

---

## 📌 الجزء السابع: شجرة القرار — أي نمط أستخدم؟

```
هل تحتاج لتمرير معاملات (QueryProperty)؟
├── لا
│   ├── هل تحتاج لبيانات محلية (Cache)؟
│   │   ├── نعم → 🌟 النمط 3 (Constructor + Refresh in Background)
│   │   └── لا
│   │       ├── هل تحتاج لتفاعل MAUI → Component؟
│   │       │   ├── نعم → 🌟 النمط 4 (Constructor + OnComponentReady)
│   │       │   └── لا → 🌟 النمط 1 (Constructor + LoadComponent)
│
└── نعم
    ├── هل عدد المعاملات ≤ 2؟
    │   ├── نعم → 🌟 النمط 2 (QueryProperty + Setter → LoadComponent)
    │   │         ⚠️ تحقق: هل Property من نوع Nullable<T>؟
    │   │         ├── نعم → ❌ ممنوع! استقبلها كـ string وحوّل يدوياً
    │   │         └── لا → تمام
    │   └── لا (3+ معاملات)
    │       ├── ⚠️ تجنب النمط 2 (يسبب وميض)
    │       └── استخدم: النمط 5 (هجين) + IQueryAttributable
```

### 7.1 جدول ملخص لاختيار النمط

| السيناريو | النمط المُوصى به | ملاحظات |
|-----------|:----------------:|---------|
| صفحة بدون معاملات، بدون تفاعل | النمط 1 | الأبسط والأكثر استقراراً |
| صفحة بمعامل واحد أو اثنين | النمط 2 | يعمل بثبات (بفضل `CustomBlazorWebView`) |
| صفحة ببيانات Cache محلية | النمط 3 | تجربة مستخدم ممتازة |
| صفحة بتفاعل MAUI → Component | النمط 4 | يحتاج `public methods` |
| صفحة بـ 3+ معاملات | **النمط 5 (هجين)** | انظر القسم الثامن |
| صفحة بـ Nullable QueryProperty | **النمط 2 مع string** | استقبل كـ string، حوّل يدوياً |

---

## 🔀 الجزء الثامن: النمط 5 (هجين) — `IQueryAttributable` + `OnComponentReady` + `OnAppearing`

### 8.1 متى يُستخدم

- عند وجود **3+ معاملات** (تجنب النمط 2).
- عند الحاجة لتحديث المكون بعد وصول المعاملات.
- عند الرغبة في تجنب الوميض.

### 8.2 البنية الكاملة

```csharp
public partial class ProgramDetailsPage : ContentPage, IQueryAttributable
{
    private ProgramDetailsWrapper? _wrapperRef;
    private readonly IApiService _apiService;
    private bool _queryReady = false;

    public ProgramDetailsPage()
    {
        InitializeComponent();
        _apiService = IPlatformApplication.Current!.Services
            .GetRequiredService<IApiService>();

        LoadBlazorComponent();  // ⭐ بدون QueryProperties (بقيم افتراضية)
    }

    private void LoadBlazorComponent()
    {
        blazorWebView.RootComponents.Clear();
        blazorWebView.RootComponents.Add(new RootComponent
        {
            Selector = "#app",
            ComponentType = typeof(ProgramDetailsWrapper),
            Parameters = new Dictionary<string, object?>
            {
                { "ApiService", (IApiService)_apiService },
                { "OnComponentReady", new Action<ProgramDetailsWrapper>(w =>
                    {
                        _wrapperRef = w;
                        Debug.WriteLine("✅ ProgramDetailsWrapper ref captured");
                        TryLoadProgram();   // ⭐ محاولة فورية
                    })
                },
                { "OnGoBack", SafeAsyncAction(GoBackAsync) },
                { "OnGoToPointsDashboard", SafeAsyncAction<int>(GoToPointsDashboardAsync) },
                // ... باقي الـ Bridges
            }
        });
    }

    // ⭐ يُستدعى بعد تعيين كل QueryProperties
    public void ApplyQueryAttributes(IDictionary<string, object> query)
    {
        if (query.TryGetValue("ProgramId", out var pid))
            _programId = Convert.ToInt32(pid);
        if (query.TryGetValue("Source", out var src))
            _source = src?.ToString() ?? "Search";
        // ... باقي المعاملات

        _queryReady = true;
        TryLoadProgram();   // ⭐ محاولة فورية
    }

    // ⭐ تُنفَّذ فقط عندما يكون الاثنان جاهزين
    private void TryLoadProgram()
    {
        if (!_queryReady || _wrapperRef == null)
        {
            Debug.WriteLine($"⏸️ [TryLoadProgram] Waiting: QueryReady={_queryReady}, " +
                            $"WrapperRef={_wrapperRef != null}");
            return;
        }

        Debug.WriteLine("🚀 [TryLoadProgram] Both ready — invoking LoadProgram");
        MainThread.BeginInvokeOnMainThread(async () =>
        {
            try
            {
                await _wrapperRef.LoadProgram(_programId, _source, _isSubscribed,
                                              _organizationId, _invitationToken);
            }
            catch (Exception ex)
            {
                Debug.WriteLine($"❌ [TryLoadProgram] LoadProgram error: {ex}");
            }
        });
    }

    // ⭐ Helper لتقليل التكرار
    private Action<T> SafeAsyncAction<T>(Func<T, Task> asyncAction) =>
        (arg) => MainThread.BeginInvokeOnMainThread(async () =>
        {
            try { await asyncAction(arg); }
            catch (Exception ex) { Debug.WriteLine($"❌ [SafeAsyncAction] {ex}"); }
        });

    private Action SafeAsyncAction(Func<Task> asyncAction) =>
        () => MainThread.BeginInvokeOnMainThread(async () =>
        {
            try { await asyncAction(); }
            catch (Exception ex) { Debug.WriteLine($"❌ [SafeAsyncAction] {ex}"); }
        });

    protected override void OnDisappearing()
    {
        base.OnDisappearing();
        _wrapperRef = null;
        _queryReady = false;
        try { blazorWebView.RootComponents.Clear(); } catch { }
    }
}
```

### 8.3 المزايا

- ✅ **يستقبل كل المعاملات مرة واحدة** (لا وميض).
- ✅ **لا يعيد تحميل المكون** (لا طلبات API متكررة).
- ✅ **يحافظ على State المكون**.
- ✅ **✅ تم التحقق منه في المشروع** (8 أكتوبر 2026 — `ProgramDetailsPage`).

### 8.4 العيوب

- ⚠️ **يحتاج `public method` على المكون** (`LoadProgram`).
- ⚠️ **يحتاج flag `_queryReady`** لمنع الاستدعاء قبل وصول الـ Query.

---

## 🚨 الجزء التاسع: تجنّب `Action<T>` مع `async`

### 9.1 ⚠️ المشكلة (اكتشاف: 8 أكتوبر 2026)

**تمرير `async` lambda كـ `Action<int>` يُخفي الـ Exception الحقيقي ويحوّله إلى `JavaProxyThrowable` المبهم على Android — مما يجعل التشخيص شبه مستحيل.**

### 9.2 ❌ النمط الخاطئ

```csharp
// في ProgramDetailsPage.xaml.cs
{ "OnGoToPointsDashboard", new Action<int>(async (programId) =>
    await GoToPointsDashboardAsync(programId)) },
```

**النتيجة:** عند حدوث Exception في `GoToPointsDashboardAsync`، يظهر:

```
Android.Runtime.JavaProxyThrowable
  HResult=0x80131500
  Source=<Cannot evaluate the exception source>
  StackTrace:
<Cannot evaluate the exception stack trace>
```

**لا يخبرك بشيء.**

### 9.3 ✅ النمط الصحيح

#### الحل 1: لفّ مباشر

```csharp
{ "OnGoToPointsDashboard", new Action<int>((programId) =>
{
    MainThread.BeginInvokeOnMainThread(async () =>
    {
        try
        {
            await GoToPointsDashboardAsync(programId);
        }
        catch (Exception ex)
        {
            Debug.WriteLine($"❌ [OnGoToPointsDashboard] {ex}");
        }
    });
}) },
```

#### الحل 2: Helper Method (مُوصى به)

```csharp
// ⭐ Helper يقلل التكرار
private Action<T> SafeAsyncAction<T>(Func<T, Task> asyncAction) =>
    (arg) => MainThread.BeginInvokeOnMainThread(async () =>
    {
        try { await asyncAction(arg); }
        catch (Exception ex) { Debug.WriteLine($"❌ [SafeAsyncAction] {ex}"); }
    });

private Action SafeAsyncAction(Func<Task> asyncAction) =>
    () => MainThread.BeginInvokeOnMainThread(async () =>
    {
        try { await asyncAction(); }
        catch (Exception ex) { Debug.WriteLine($"❌ [SafeAsyncAction] {ex}"); }
    });

// الاستخدام:
{ "OnGoToPointsDashboard", SafeAsyncAction<int>(GoToPointsDashboardAsync) },
{ "OnGoBack", SafeAsyncAction(GoBackAsync) },
```

### 9.4 📌 القاعدة الذهبية

> **كل `Action<T>` يحتوي على `async` يجب أن:**
> 1. **يُلفّ في `MainThread.BeginInvokeOnMainThread`**.
> 2. **يحتوي على `try/catch` صريح**.
> 3. **يطبع الـ Exception الكامل** (`ex.ToString()`).

**هذا يحوّل `JavaProxyThrowable` المبهم إلى Exception واضح.**

### 9.5 📊 مقارنة: قبل/بعد

| قبل | بعد |
|-----|-----|
| `JavaProxyThrowable` فقط | Exception حقيقي مع Stack Trace كامل |
| لا نعرف السبب | نعرف السطر والسبب |
| ساعات من التشخيص | دقائق من التشخيص |

---

## 🚨 الجزء العاشر: `CustomBlazorWebView`

### 10.1 📌 ما هو؟

**`CustomBlazorWebView` هو Wrapper مخصص حول `BlazorWebView` مُسجَّل في `RubikCare.Mobile.Controls`.** يستخدمه **كل المشروع تقريباً** بدلاً من `BlazorWebView` العادي.

**المسار:** `RubikCare.Mobile/Controls/CustomBlazorWebView.cs`

### 10.2 📌 لماذا؟

**⚠️ نقطة تحتاج توثيقاً:** السبب الدقيق غير موثق. لكن بناءً على استخدامه في:
- `PharmacySearchFlow`
- `PharmacyDetailFlow`
- `ProgramDetailsPage`
- `PointsDashboardPage`
- صفحات أخرى كثيرة

**الاحتمالات:**
1. يحل مشكلة `jumpToEnd` على Android.
2. يوفر `OnPageAppearing()` method.
3. يعالج `IBackButtonHandler` تلقائياً.

### 10.3 📌 الاستخدام القياسي

```xml
<ContentPage xmlns="http://schemas.microsoft.com/dotnet/2021/maui"
             xmlns:x="http://schemas.microsoft.com/winfx/2009/xaml"
             xmlns:controls="clr-namespace:RubikCare.Mobile.Controls"
             x:Class="RubikCare.Mobile.Features.MyFlow">
    <controls:CustomBlazorWebView x:Name="blazorWebView" HostPage="wwwroot/index.html" />
</ContentPage>
```

### 10.4 ✅ القاعدة

**استخدم `CustomBlazorWebView` دائماً** — لا `BlazorWebView` العادي.

---

## 🔴 الجزء الحادي عشر: تشخيص الصفحة الفارغة

**هذا هو أكثر سيناريو مربك لأنه لا يعطي أي خطأ. اتبع هذه الخطوات بالترتيب:**

### 11.1 الخطوة 1: تأكد أن MAUI layer يعمل

```csharp
public MyFlowPage()
{
    InitializeComponent();
    Debug.WriteLine("🔵 Constructor START");
    Debug.WriteLine($"🟢 RootComponents: {blazorWebView.RootComponents.Count}");
}
```

| النتيجة | المشكلة |
|---------|---------|
| لا يظهر "Constructor START" | مشكلة في تسجيل الصفحة أو الـ Routing |
| يظهر لكن Count = 0 | مشكلة في `RootComponents.Add` |
| Count = 1 | انتقل للخطوة 2 |

### 11.2 الخطوة 2: تأكد أن Blazor runtime يعمل

**استبدل مكونك مؤقتاً بـ TestPage بسيطة بدون أي `@inject`:**

```razor
@namespace RubikCare.Shared.UI.Components

<h1 style="color:red;font-size:48px;">✅ BLAZOR WORKS</h1>

@code {
    protected override void OnInitialized()
    {
        Debug.WriteLine("🔴 TestPage: OnInitialized CALLED");
    }
}
```

| النتيجة | المشكلة |
|---------|---------|
| لم تظهر TestPage | مشكلة في WebView2 أو `index.html` أو تسجيل الخدمات |
| ظهرت TestPage | انتقل للخطوة 3 |

### 11.3 الخطوة 3: عزّل مشكلة `@inject`

**أضف `@inject` الخاصة بمكونك واحداً واحداً لـ TestPage حتى تجد المشكلة.**

**توقفت عند خدمة معينة → هذه الخدمة غير مسجلة في حاوية Blazor → حوّلها إلى `[Parameter]`.**

### 11.4 الخطوة 4: تأكد من `OnInitializedAsync`

```csharp
protected override async Task OnInitializedAsync()
{
    Debug.WriteLine("🔵 OnInitializedAsync START");
    Debug.WriteLine("🟢 OnInitializedAsync END");
}
```

| النتيجة | المشكلة |
|---------|---------|
| لا يظهر START | مشكلة DI (الخطوة 3) |
| يظهر START لكن لا يظهر END | Exception في المنتصف |

### 11.5 الخطوة 5: إذا ظهر `JavaProxyThrowable`

**السبب المحتمل:** `Action<T>` مع `async` — راجع القسم التاسع.

**الحل:** لفّ الـ `Action` في `MainThread.BeginInvokeOnMainThread` + `try/catch` (أو `SafeAsyncAction`).

---

## 🔗 الجزء الثاني عشر: نظام الملاحة (Navigation) في MAUI Shell

### 12.1 📌 القواعد الذهبية لتسجيل الصفحات

#### 1. الصفحات الرئيسية (Root Pages) — 6 صفحات فقط

| الصفحة | Route | الملف |
|--------|-------|-------|
| تسجيل الدخول | `LoginPage` | `AppShell.xaml` |
| إنشاء حساب | `RegisterPage` | `AppShell.xaml` |
| الرئيسية (Personal) | `MainDashboard` | `AppShell.xaml` |
| لوحة العيادة | `ClinicDashboardPage` | `AppShell.xaml` |
| لوحة الصيدلية | `PharmacyDashboardPage` | `AppShell.xaml` |
| لوحة شركة الأدوية | `PharmaCompanyDashboardPage` | `AppShell.xaml` |

**تسجل في:** `AppShell.xaml` **فقط** كـ `<ShellContent>`.
**تستدعى بـ:** `//Route` (شرطتان مائلتان).

#### 2. الصفحات الفرعية (Detail Pages) — كل ما عدا الستة

**تسجل في:** `AppShell.xaml.cs` **فقط** باستخدام `Routing.RegisterRoute`.
**تستدعى بـ:** `Route` (بدون شرطات).

#### 3. جدول أنواع التنقل الكامل

| الرمز | النوع | التأثير | يستخدم مع | مثال |
|-------|-------|---------|-----------|------|
| `//Route` | Absolute | يستبدل الصفحة الحالية في المكدس | الصفحات الرئيسية | `//MainDashboard` |
| `///Route` | Absolute كامل | يمسح المكدس بالكامل | ❌ ممنوع الاستخدام | - |
| `Route` | Relative | يضيف صفحة فوق المكدس | الصفحات الفرعية | `doctorprofile` |
| `..` | Back | يرجع صفحة للخلف | أي صفحة | `..` |
| `Route?param=value` | Relative + معاملات | يضيف صفحة مع بيانات | الصفحات الفرعية | `doctorprofile?clinicId=5` |

### 12.2 🔴 أخطاء التنقل الشائعة

#### الخطأ 1: `Ambiguous routes matched for...`

**الرسالة:**
```
System.ArgumentException: Ambiguous routes matched for: 
//.../pageName matches found: //.../pageName, //.../pageName
```

**ماذا يعني:** الصفحة مسجلة مرتين - في `AppShell.xaml` و `AppShell.xaml.cs` معاً.

**الحل:** احذف التسجيل من أحد الملفين.

#### الخطأ 2: `JavaProxyThrowable`

**متى يظهر:** عند العودة من صفحة إلى أخرى (زر الرجوع).

**الأسباب المحتملة:**
1. تعارض إصدارات `WebView.Maui` مع `MAUI.Controls` (الأكثر شيوعاً).
2. `Ambiguous Routes` (يظهر معه في نفس الوقت).

**الحل:** تأكد من تطابق إصدارات MAUI في `.csproj`:
```xml
<PackageReference Include="Microsoft.Maui.Controls" Version="10.0.20" />
<PackageReference Include="Microsoft.AspNetCore.Components.WebView.Maui" Version="10.0.20" />
```

#### الخطأ 3: `No view found for id 0x7f0800ff (jumpToEnd)`

**متى يظهر:** بعد Splash Screen مباشرة، قبل تحميل أي صفحة. التطبيق ينهار فوراً.

**ماذا يعني:** استخدام `///` مع صفحة رئيسية أدى إلى تدمير `Fragment` الداخلي لـ Shell نفسه.

**الأسباب المحتملة:**

| # | السبب | مثال خاطئ |
|---|-------|-----------|
| 1 | `///` مع صفحة رئيسية | `GoToAsync("///ClinicDashboardPage")` |
| 2 | مسار نسبي مع صفحة رئيسية | `GoToAsync("MainDashboard")` بدون `//` |
| 3 | حذف كود XAML معلق دون تنظيف | ترك `xmlns` references مكسورة في الملف |
| 4 | تسجيل رئيسية في `AppShell.xaml.cs` | `Routing.RegisterRoute("MainDashboard", ...)` |

**الحل:**
1. استبدل كل `///Route` بـ `//Route` للصفحات الرئيسية.
2. استبدل كل مسار نسبي لصفحة رئيسية بـ `//Route`.
3. نفذ تنظيف عميق:
```bash
rmdir /s /q bin
rmdir /s /q obj
# احذف التطبيق من الهاتف
dotnet restore
dotnet build -f net10.0-android -c Debug -t:Install
```

#### الخطأ 4: `Global routes currently cannot be the only page on the stack`

**ماذا يعني:** تحاول استخدام `//` مع صفحة **فرعية** (مسجلة في `AppShell.xaml.cs`).

**الحل:** احذف `//` واستخدم المسار النسبي فقط.

#### الخطأ 5: `unable to figure out route for...`

**ماذا يعني:** الصفحة غير مسجلة في `AppShell.xaml.cs` أو أن المسار غير معروف.

**الحل:** تحقق من `Routing.RegisterRoute` موجود.

### 12.3 📊 مصفوفة التشخيص السريع

| الخطأ | أول ملف تراجعه | ثاني ملف | ثالث ملف |
|-------|---------------|---------|----------|
| `Ambiguous routes` | `AppShell.xaml` | `AppShell.xaml.cs` | - |
| `JavaProxyThrowable` | `AppShell.xaml.cs` | `.csproj` | `AppShell.xaml` |
| `jumpToEnd` | `AppShellViewModel.cs` | `AppShell.xaml` | `AppShell.xaml.cs` |
| `Global routes...` | `ViewModel` المعني | `AppShell.xaml.cs` | - |
| `unable to figure out` | `AppShell.xaml.cs` | `ViewModel` المعني | - |

### 12.4 🛡️ إجراءات الوقاية

1. **قبل أي Commit:** تأكد من عدم وجود `///` في المشروع كله.
2. **بعد حذف كود XAML معلق:** تنظيف عميق إجباري.
3. **عند إضافة صفحة جديدة:** اسأل: "هل هي من الـ 6 الرئيسية؟"
4. **القاعدة النهائية:**
   ```
   رئيسية؟ → AppShell.xaml → //Route
   فرعية؟  → AppShell.xaml.cs → Route
   رجوع؟   → ..
   ///     → ممنوع نهائياً
   ```

---

## 📖 الجزء الثالث عشر: قصة نجاح `PointsDashboardPage`

### 13.1 السياق

**صفحة `PointsDashboardPage` كانت لا تفتح في MAUI (تعمل في PWA). الخطأ الظاهر: `JavaProxyThrowable` المبهم.**

### 13.2 التشخيص (بعد ساعات من التحقيق)

**3 مشاكل متراكبة:**

1. **`ProgramId` لم يُستقبل أصلاً** — كان `[QueryProperty]` يُعيَّن بعد `LoadBlazorComponent()`.
2. **`[QueryProperty]` مع `int?`** — فشل التحويل.
3. **`Action<int>` + `async`** — بدون `try/catch`.

### 13.3 الحل النهائي

**في `PointsDashboardPage.xaml.cs`:**

```csharp
[QueryProperty(nameof(ProgramIdRaw), "programId")]
public partial class PointsDashboardPage : ContentPage
{
    public string? ProgramIdRaw { get; set; }
    private int? ProgramId => int.TryParse(ProgramIdRaw, out var id) ? id : null;
    private bool _componentLoaded = false;

    protected override void OnAppearing()
    {
        base.OnAppearing();
        if (!_componentLoaded)
        {
            LoadBlazorComponent();
            _componentLoaded = true;
        }
    }
}
```

**في `ProgramDetailsPage.xaml.cs`:**

```csharp
{ "OnGoToPointsDashboard", SafeAsyncAction<int>(GoToPointsDashboardAsync) },
```

### 13.4 النتيجة

✅ الصفحة تفتح بنجاح في MAUI.
✅ الـ Log يظهر القيم الصحيحة (`OrgId=1247`, `ProgramId=1`).
✅ لا `JavaProxyThrowable`.

### 13.5 الدروس المستفادة

1. **`[QueryProperty]` ≠ `Nullable<T>`** — استخدم `string`.
2. **`[QueryProperty]` يُعيَّن بعد Constructor** — استخدم `OnAppearing` مع flag.
3. **`Action<T>` + `async` = `JavaProxyThrowable`** — لفّها بـ `try/catch`.
4. **"الفشل الصامت" في MAUI** — قد يكون DI، قد يكون QueryProperty، قد يكون Action.
5. **`SafeAsyncAction` Helper** — يوحد النمط، يقلل التكرار.

---

## 🔧 الجزء الرابع عشر: التحديات المعروفة وحلولها

### 14.1 مشكلة Binding في XAML

**المشكلة:** `InvalidCastException` عند binding لخصائص معقدة.

**الحل:** استخدام خصائص بسيطة في ViewModel.

```csharp
// ❌ خطأ
public PspProgram SelectedProgram { get; set; }

// ✅ صحيح
public string ProgramName { get; set; }
public string ProgramDescription { get; set; }
```

### 14.2 مشكلة UI Thread

**المشكلة:** تحديث UI من خيط غير رئيسي يسبب استثناء.

**الحل:** استخدام `MainThread.BeginInvokeOnMainThread`.

```csharp
MainThread.BeginInvokeOnMainThread(() =>
{
    IsLoading = false;
    Programs = new ObservableCollection<PspProgram>(data);
});
```

### 14.3 مشكلة SecureStorage

**المشكلة:** `SecureStorage` يفشل أحياناً على بعض أجهزة Android.

**الحل:** استخدام `Preferences` كبديل مع تشفير القيمة.

```csharp
public async Task SaveTokenAsync(string token)
{
    try
    {
        await SecureStorage.SetAsync("auth_token", token);
    }
    catch
    {
        // fallback لـ Preferences
        Preferences.Set("auth_token", token);
    }
}
```

### 14.4 مشكلة الاتصال بالإنترنت

**المشكلة:** التطبيق يتجمد عند فقدان الاتصال.

**الحل:** التحقق من الاتصال قبل كل استدعاء API.

```csharp
public async Task<T> SafeApiCallAsync<T>(Func<Task<T>> apiCall)
{
    if (Connectivity.Current.NetworkAccess != NetworkAccess.Internet)
    {
        await ShowError("لا يوجد اتصال بالإنترنت");
        return default;
    }

    try
    {
        return await apiCall();
    }
    catch (HttpRequestException)
    {
        await ShowError("خطأ في الاتصال بالخادم");
        return default;
    }
}
```

### 14.5 ⭐ مشكلة: منظمات المستخدم لا تظهر في القائمة الجانبية

**السبب:** `AppShellViewModel.LoadUserDataAsync()` لم تكن تُستدعى من `DashboardPage`.

**الحل:** إضافة استدعاء `appShell.RefreshUserDataAsync()` في `DashboardPage.OnAppearing`.

### 14.6 ⭐ مشكلة: ظهور بيانات المستخدم السابق بعد Logout

**السبب:** الكاش المحلي (`_cachedSession`) لم يكن يُمسح عند Logout.

**الحل:** استدعاء `_cachedSessionService.ClearSessionAsync()` في `AppShellViewModel.LogoutAsync`.

### 14.7 ⭐ مشكلة: نظام الملاحة

**⚠️ التفاصيل الكاملة في القسم الثاني عشر (نظام الملاحة).**

---

## 📱 الجزء الخامس عشر: إدارة الجلسات والكاش في الموبايل

### 15.1 نظرة عامة

**إدارة الجلسات في الموبايل تعتمد على 3 طبقات:**

| الطبقة | المسؤول | الموقع |
|--------|---------|--------|
| **الخادم** | `UserSessionService` (Application) | Api.Web |
| **محلي** | `CachedUserSessionService` | الموبايل |
| **تخزين آمن** | `SecureStorage` | الموبايل |

### 15.2 `CachedUserSessionService`

**المسار:** `RubikCare.Mobile/Infrastructure/Services/CachedUserSessionService.cs`

**الوظائف الرئيسية:**

- `GetSessionAsync()` — جلب الجلسة من الكاش المحلي أو من API.
- `RefreshSessionAsync()` — تحديث الجلسة من `POST api/user/session-bootstrap`.
- `ClearSessionAsync()` — ⭐ مسح الكاش المحلي (يجب استدعاؤها عند Logout).
- `IsSessionValidAsync()` — التحقق من صلاحية الجلسة.

### 15.3 `AppShellViewModel`

**المسار:** `RubikCare.Mobile/ViewModels/AppShellViewModel.cs`

**الوظائف الرئيسية:**

- `LoadUserDataAsync()` — تحميل بيانات المستخدم ومنظماته.
- `LogoutAsync()` — تسجيل الخروج ومسح جميع البيانات.
- `NavigateToOrganization(org)` — التنقل لمنظمة محددة.

**⚠️ نقطة حرجة:** `AppShellViewModel.LoadUserDataAsync()` **يجب أن تُستدعى صراحة** من `DashboardPage.OnAppearing`:

```csharp
// في DashboardPage.xaml.cs
protected override async void OnAppearing()
{
    base.OnAppearing();

    // ⭐ تحديث القائمة الجانبية
    if (Shell.Current is AppShell appShell)
    {
        await appShell.RefreshUserDataAsync();
    }

    // تحميل بيانات Dashboard
    if (BindingContext is DashboardPageViewModel vm)
    {
        await vm.LoadUserDataAsync();
    }
}
```

### 15.4 `AuthService` — المصادقة والخروج

**المسار:** `RubikCare.Mobile/Infrastructure/Services/AuthService.cs`

**تسلسل Logout الصحيح:**

```csharp
public async Task LogoutAsync()
{
    // 1. إعلام الخادم (يمسح كاش UserSessionService)
    try
    {
        await _apiService.PostAsync<object>("api/auth/logout", null);
    }
    catch (Exception ex)
    {
        Debug.WriteLine($"❌ Logout API error: {ex.Message}");
    }

    // 2. مسح التخزين المحلي
    SecureStorage.Remove("auth_token");
    SecureStorage.Remove("user_profile");
    SecureStorage.Remove("userProfileId");
}
```

---

## 🎨 الجزء السادس عشر: الأنماط المعمارية في MAUI

### 16.1 نمط MVVM

```csharp
// ViewModel مثال:
public class LoginViewModel : BaseViewModel
{
    private readonly IAuthService _authService;
    private readonly INavigationService _navigationService;

    private string _email;
    public string Email
    {
        get => _email;
        set { _email = value; OnPropertyChanged(); }
    }

    private string _password;
    public string Password
    {
        get => _password;
        set { _password = value; OnPropertyChanged(); }
    }

    private bool _isLoading;
    public bool IsLoading
    {
        get => _isLoading;
        set { _isLoading = value; OnPropertyChanged(); }
    }

    public ICommand LoginCommand { get; }

    public LoginViewModel(IAuthService authService, INavigationService navigationService)
    {
        _authService = authService;
        _navigationService = navigationService;
        LoginCommand = new Command(async () => await LoginAsync());
    }

    private async Task LoginAsync()
    {
        if (IsLoading) return;
        IsLoading = true;

        try
        {
            var result = await _authService.LoginAsync(Email, Password);
            if (result.Success)
            {
                await _navigationService.GoToMainPage();
            }
            else
            {
                await ShowError(result.Message);
            }
        }
        finally
        {
            IsLoading = false;
        }
    }
}
```

### 16.2 `BaseViewModel`

```csharp
public class BaseViewModel : INotifyPropertyChanged
{
    public event PropertyChangedEventHandler PropertyChanged;

    protected void OnPropertyChanged([CallerMemberName] string propertyName = null)
    {
        PropertyChanged?.Invoke(this, new PropertyChangedEventArgs(propertyName));
    }

    protected async Task ShowError(string message)
    {
        await Application.Current.MainPage.DisplayAlert("خطأ", message, "موافق");
    }

    protected async Task ShowSuccess(string message)
    {
        await Application.Current.MainPage.DisplayAlert("نجاح", message, "موافق");
    }
}
```

---

## 🔗 الجزء السابع عشر: Bridges (التواصل بين Blazor و MAUI)

### 17.1 إنشاء Bridge

**في `Shared.UI/Components/Rep/RepNavigationBridge.cs`:**

```csharp
public static class RepNavigationBridge
{
    public static Action<int>? OnProgramSelected { get; set; }
    public static Func<string, string, Task>? ShareToken { get; set; }
}
```

### 17.2 تسجيل Bridge في XAML.cs

```csharp
RepNavigationBridge.ShareToken = async (token, programName) =>
{
    var message = $"🎫 كود الدعوة: {token}";
    await Share.Default.RequestAsync(new ShareTextRequest
    {
        Text = message,
        Title = $"دعوة لبرنامج {programName}"
    });
};
```

### 17.3 استخدام Bridge في Razor

```csharp
private async Task ShareToken()
{
    if (RepNavigationBridge.ShareToken != null)
        await RepNavigationBridge.ShareToken(_token, _programName);
}
```
### 17.4 ✅ ربط جميع دوال الـ Bridge في OnAppearing

**القاعدة الذهبية:** أي دالة في `PspDetailNavigationBridge` (أو أي Bridge آخر) **يجب** ربطها من الصفحة المضيفة (`.xaml.cs`) قبل استخدامها في المكون. ربط بعض الدوال ونسيان البعض الآخر هو سبب شائع لـ "زر لا يعمل".

**Checklist سريعة:**
- [ ] افتح ملف الـ Bridge (`PspDetailNavigationBridge.cs`).
- [ ] لكل `Action` أو `Func` في الـ Bridge، تأكد من وجود سطر ربط مقابل في `OnAppearing` في الصفحة المضيفة.
- [ ] استخدم `Dispatcher.DispatchAsync` أو `MainThread.BeginInvokeOnMainThread` داخل كل `Action` لتجنب `JavaProxyThrowable`.
- [ ] تأكد من أن المسارات (`medication-schedule`, `pharmacy-search`) مسجلة في `AppShell.xaml.cs`.

**مثال:**
```csharp
// في PspDetailFlow.xaml.cs - OnAppearing
PspDetailNavigationBridge.GoToSchedule = async () =>
{
    await Dispatcher.DispatchAsync(async () =>
    {
        try { await Shell.Current.GoToAsync("medication-schedule"); }
        catch (Exception ex) { Debug.WriteLine($"❌ {ex}"); }
    });
};

PspDetailNavigationBridge.GoToPharmacySearch = async (token) =>
{
    await Dispatcher.DispatchAsync(async () =>
    {
        try
        {
            var parameters = new Dictionary<string, object> { { "token", token } };
            await Shell.Current.GoToAsync("pharmacy-search", parameters);
        }
        catch (Exception ex) { Debug.WriteLine($"❌ {ex}"); }
    });
};


---
---
```
### 📊 الخلاصة

| العنصر | الحالة |
|--------|--------|
| **صفحة النقاط** | ✅ تعمل (بعد حل `InvalidCastException` بتحويل `userId` إلى `string`) |
| **زر جدول الأدوية** | ❌ لا يعمل — يحتاج ربط `GoToSchedule` في `PspDetailFlow` |
| **زر البحث عن صيدلية** | ❌ لا يعمل — يحتاج ربط `GoToPharmacySearch` في `PspDetailFlow` |
| **التوثيق** | ✅ موجود جزئيًا (Bridges + Dispatcher) — يحتاج إضافة "ربط جميع الدوال" في Checklist |

**أضف ربط الزرين في `PspDetailFlow.xaml.cs`، وتأكد من المسارات، ثم جرب.** 🚀
---
---

## 🚀 الجزء الثامن عشر: النشر (Publishing)

### 18.1 Android (APK/AAB)

```bash
# إنشاء APK
dotnet publish -f net10.0-android -c Release /p:AndroidPackageFormat=apk

# إنشاء AAB (Google Play)
dotnet publish -f net10.0-android -c Release /p:AndroidPackageFormat=aab
```

### 18.2 iOS

```bash
# إنشاء IPA
dotnet publish -f net10.0-ios -c Release
```

---

## ✅ الجزء التاسع عشر: CHECKLIST

### 19.1 عند إضافة صفحة BlazorWebView جديدة

#### MAUI Layer (صفحة الحاوية)

- [ ] هل تستخدم `CustomBlazorWebView` بدلاً من `BlazorWebView` العادي؟
- [ ] هل اخترت النمط الصحيح من القسم السابع (شجرة القرار)؟
- [ ] هل `RootComponents.Add` يُستدعى بشكل متزامن في الكونستركتور (أو Setter للنمط 2)؟
- [ ] هل `ApiService` يُمرر كـ `(IApiService)_apiService`؟
- [ ] هل تستخدم `Action` بدلاً من `EventCallback` لتمرير الدوال؟
- [ ] هل `OnDisappearing` تستدعي `RootComponents.Clear()` وتنظف `_pageRef`؟
- [ ] هل `[QueryProperty]` من نوع `string` وليس `Nullable<T>`؟
- [ ] هل `Action<T>` مع `async` ملفوف في `MainThread.BeginInvokeOnMainThread` + `try/catch` (أو `SafeAsyncAction`)؟
- [ ] إذا كان `[QueryProperty]` مستخدماً — هل `LoadBlazorComponent` في `OnAppearing` وليس Constructor؟

#### Blazor Component (المكون)

- [ ] هل المكون **لا يستخدم** `@inject HttpClient` أو `@inject ApiService`؟
- [ ] هل كل خدمة MAUI تُستخدم عبر `[Parameter]` وليس `@inject` (ما عدا الترجمة و `NavigationManager`)؟
- [ ] هل `OnInitializedAsync` تُستدعى؟ (تأكد من Debug Output)
- [ ] هل المكون يعرض Skeleton/Loading أثناء جلب البيانات؟
- [ ] هل توجد `public methods` للتحديث من الحاوية عند الحاجة (النمط 4/5)؟
- [ ] هل الـ Parameters من نوع `Action<T>` (وليس `EventCallback`)؟

### 19.2 عند إضافة صفحة MAUI جديدة

- [ ] هل أنشأت مجلداً في `Features/{Domain}/Views/`؟
- [ ] هل أنشأت ViewModel مطابقاً في `Features/{Domain}/ViewModels/`؟
- [ ] هل ورثت من `BaseViewModel`؟
- [ ] هل استخدمت `SafeApiCallAsync` لاستدعاءات API؟
- [ ] هل تعاملت مع حالة عدم الاتصال بالإنترنت؟
- [ ] هل اختبرت على Android و iOS؟
- [ ] هل تعاملت مع UI Thread بشكل صحيح؟

### 19.3 عند التعامل مع الجلسات والكاش

- [ ] هل `AppShellViewModel.LoadUserDataAsync()` تُستدعى من `DashboardPage.OnAppearing`؟
- [ ] هل `LogoutAsync` يمسح `_cachedSession` + `SecureStorage` + يستدعي `api/auth/logout`؟
- [ ] هل `HasClinic`/`HasPharmacy` تستخدمان `Memberships` وليس `Clinics`/`Pharmacies`؟

---

## 📝 الجزء العشرون: نقاط تحتاج توثيقاً (TODO)

1. **لماذا يعمل النمط 2 (`QueryProperty + Setter`)** رغم تحذير القسم الرابع؟
   - هل بسبب `CustomBlazorWebView`؟
   - هل بسبب `RootComponents.Clear()` الذي يعمل فعلياً؟

2. **ما الفرق الدقيق بين `CustomBlazorWebView` و `BlazorWebView`؟**
   - ما الميزات الإضافية؟
   - ما المشاكل التي يحلها؟

3. **هل `static variables` في النمط 3 مقصودة؟**
   - ما الفائدة المعمارية؟

4. **✅ النمط الهجين (5) — تم التحقق منه (8 أكتوبر 2026).**

5. **ما هو `IBackButtonHandler`؟**
   - كيف يعمل؟
   - متى نستخدمه؟

---

## 🔗 الجزء الحادي والعشرون: روابط ذات صلة

- [00 — MASTER-INDEX](./00-MASTER-INDEX.md)
- [00B — PROJECT-BACKLOG](./00B-PROJECT-BACKLOG.md)
- [01 — ARCHITECTURE](./01-ARCHITECTURE.md)
- [04 — TRANSLATION](./04-TRANSLATION.md)
- [10 — UI-STYLE-GUIDE](./10-UI-STYLE-GUIDE.md)
- [11 — PAGE-CREATION](./11-PAGE-CREATION.md)
- [12 — CACHING](./12-CACHING.md)
- [14 — PWA](./14-PWA.md)
- [16 — TROUBLESHOOTING](./16-TROUBLESHOOTING.md)

---

## 📝 الجزء الثاني والعشرون: سجل التغييرات

| الإصدار | التاريخ | التغييرات |
|:---:|:---|:---|
| **1.0** | **24 مايو 2026** | الإصدار الأولي — `10-maui-development-guide.md` |
| **1.0** | **8 يوليو 2026** | الإصدار الأولي — `11-blazor-webview-guide.md` v1.0 |
| **2.0** | **8 أكتوبر 2026** | `[QueryProperty]` + Nullable + `Action<T>` + async + قصة نجاح `PointsDashboardPage` |
| **3.0** | **10 أكتوبر 2026** | `SafeAsyncAction` Helper + جدول مقارنة `[QueryProperty]` vs `IQueryAttributable` |
| **2.0** | **17 أكتوبر 2026** | **`13-MAUI` v2.0** — دمج `10` + `11` + قاعدة تسجيل الصفحات + Bridges + إشارات |

---

**© 2026 RubikCare — للاستخدام الداخلي**

**نهاية الوثيقة ✅**
```
