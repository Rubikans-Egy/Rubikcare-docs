# 11 - دليل BlazorWebView والأنماط الناجحة

**آخر تحديث: 8 أكتوبر 2026 (الإصدار 2.0)**
**الحالة: ✅ محدّث بالأنماط الفعلية + دروس مستفادة من جلسة نظام النقاط**

---

## 📌 مقدمة

هذا المرجع يوثق **الأنماط الفعلية الناجحة** وما يجب تجنبه عند استخدام `BlazorWebView` داخل تطبيق MAUI. الهدف هو مشاركة المكونات بين Web و PWA و Mobile مع تجنب المشاكل المعروفة، وتوحيد أسلوب العمل بين أعضاء الفريق.

**⚠️ ملاحظة مهمة:** هذا المستند يعكس **ما هو مُتبع فعلياً في المشروع**، وليس ما هو نظري. بعض الأنماط هنا **تعمل بثبات** رغم تحذيرات قديمة في نسخ سابقة من هذا المستند.

**🆕 جديد في الإصدار 2.0:**
- قسم `[QueryProperty]` مع `Nullable<T>` (القسم 2-ب)
- قسم `Action<T>` مع `async` (القسم 5-ب)
- تحديث شجرة القرار (القسم 4)
- قصة نجاح `PointsDashboardPage` (القسم 12)
- تحديث النقاط التي تحتاج توثيقاً (القسم 11)

---

## 🧠 لماذا BlazorWebView؟

| الميزة | XAML التقليدي | BlazorWebView |
|--------|---------------|---------------|
| لغة التطوير | لغة مختلفة عن Web | نفس HTML/CSS المستخدم في Web |
| ربط البيانات | مشاكل Binding متكررة | لا Binding - استخدام Razor |
| مشاركة المكونات | صعبة | مشاركة مباشرة عبر RCL |
| منحنى التعلم | منفصل تماماً | نفس مهارات Blazor |

---

## 🚨 القسم الأول: مشكلة Dependency Injection (DI)

### ⚠️ المشكلة الجذرية (اكتشاف: 13 يونيو 2026)

`BlazorWebView` في MAUI يستخدم حاوية Dependency Injection **مستقلة** عن حاوية MAUI الرئيسية. هذا يعني أن الخدمات المسجلة في `MauiProgram.cs` **ليست متاحة تلقائياً** لمكونات Blazor الداخلية.

### ❌ العرض: فشل صامت للمكون

عندما يستخدم مكون Blazor `@inject` لخدمة غير مسجلة في حاوية Blazor الداخلية، **يفشل المكون في التحميل تماماً بدون أي خطأ** في Debug Output. الصفحة تظهر فارغة أو لا تظهر على الإطلاق.

> **⚠️ تحذير:** هذا الفشل **صامت تماماً** — لا exception، لا خطأ في الـ Console، لا أي مؤشر. `OnInitializedAsync` لا تُستدعى أبداً.

### 📊 الخدمات التي تعمل وتفشل مع `@inject`

| الخدمة | `@inject` في Blazor | التفسير |
|--------|---------------------|---------|
| `ISharedTranslationService` | ✅ يعمل | مسجلة في حاوية Blazor |
| `ISharedTranslationState` | ✅ يعمل | مسجلة في حاوية Blazor |
| `NavigationManager` | ✅ يعمل | مسجلة تلقائياً من Blazor runtime |
| `HttpClient` | ❌ يفشل بصمت | غير مسجلة في حاوية Blazor |
| `ApiService` | ❌ يفشل بصمت | غير مسجلة في حاوية Blazor |
| `IMobileTranslationService` | ❌ يفشل بصمت | غير مسجلة في حاوية Blazor |

### ✅ الحل: تمرير الخدمات كـ Parameters

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

### 🔍 Checklist تشخيص مشكلة DI

عندما يفشل مكون Blazor في التحميل داخل MAUI:

- [ ] هل المكون يعمل في Web Blazor لكن ليس في MAUI؟
- [ ] هل تستخدم `@inject HttpClient`؟ → **احذفه فوراً**
- [ ] هل تستخدم `@inject` لأي خدمة غير `ISharedTranslation*` أو `NavigationManager`؟
- [ ] هل `OnInitializedAsync` لا تُستدعى (لا يظهر Debug Output)؟
- [ ] هل الصفحة فارغة تماماً بدون أي خطأ؟

**إذا أجبت بنعم على أي سؤال، استبدل `@inject` بـ `[Parameter]`.**

---

## 🚨 القسم الثاني: قيود RootComponents في BlazorWebView

### ⚠️ السبب الجذري

`BlazorWebView` على Windows يستخدم `WebView2` كمحرك. هذا المحرك **يربط الـ DOM بمكونات Blazor فقط أثناء دورة حياة الكونستركتور** للـ `ContentPage` المضيفة.

**⚠️ استثناء مهم مُوثَّق:** في بيئة المشروع الفعلية، `RootComponents.Clear()` ثم `Add` **يعمل بنجاح** في بعض السيناريوهات (خاصة عند استخدام `CustomBlazorWebView` — انظر القسم التالي). هذا مُوثَّق في نمط "QueryProperty + Setter" أدناه.

### ❌ أنماط فاشلة موثقة

```csharp
// ❌ النمط 1: async في الكونستركتور
public MyFlowPage()
{
    InitializeComponent();
    _ = LoadAndShowAsync(); // Task تنتهي بعد الكونستركتور — لن يعمل
}

private async Task LoadAndShowAsync()
{
    await LoadDataAsync();
    blazorWebView.RootComponents.Add(...); // ← متأخر جداً
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

### ✅ النمط الصحيح (الأساسي)

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

## 🚨 القسم الثاني-ب: فخ `[QueryProperty]` مع Nullable Types

### ⚠️ المشكلة (اكتشاف: 8 أكتوبر 2026)

عند استخدام `[QueryProperty]` مع خاصية من نوع `int?` (أو أي `Nullable<T>`)، يفشل MAUI Shell في تحويل القيمة القادمة (String) إلى `Nullable<int>` — ويظهر الخطأ التالي:

```
System.InvalidCastException: Invalid cast from 'System.String' to 
'System.Nullable`1[[System.Int32, System.Private.CoreLib, Version=10.0.0.0, 
Culture=neutral, PublicKeyToken=7cec85d7bea7798e]]'.
   at System.Convert.DefaultToType(IConvertible value, Type targetType, IFormatProvider provider)
   at System.String.System.IConvertible.ToType(Type type, IFormatProvider provider)
   at System.Convert.ChangeType(Object value, Type conversionType, IFormatProvider provider)
   at Microsoft.Maui.Controls.ShellContent.ApplyQueryAttributes(Object content, 
       ShellRouteParameters query, ShellRouteParameters oldQuery)
```

### 🎯 السبب الجذري

`[QueryProperty]` يستخدم `Convert.ChangeType` داخلياً، وهذا **لا يدعم `Nullable<T>`**. عندما يمرر Shell القيمة `"1"` (String) إلى خاصية `int?`، يفشل التحويل.

### ❌ الكود الفاشل

```csharp
[QueryProperty(nameof(ProgramId), "programId")]
public partial class PointsDashboardPage : ContentPage
{
    public int? ProgramId { get; set; }   // ❌ يفشل بـ InvalidCastException
}
```

### ✅ الحل: استقبل كـ String ثم حوّل يدوياً

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

### 📌 القاعدة الذهبية

> **`[QueryProperty]` يستخدم `Convert.ChangeType` داخلياً، وهذا لا يدعم `Nullable<T>`.**
>
> **دائماً استقبل QueryProperty كـ `string`، ثم حوّل يدوياً.**

### 📊 جدول: أنواع Property ونتائجها

| النوع | يعمل؟ | ملاحظات |
|-------|:---:|---------|
| `string` | ✅ | الأكثر أماناً — استخدمه دائماً |
| `string?` | ✅ | يعمل (مع nullable reference types) |
| `int` (non-nullable) | ✅ | يعمل، لكن يفشل إذا كانت القيمة فارغة |
| `int?` (nullable) | ❌ | **يفشل بـ InvalidCastException** |
| `DateTime?` | ❌ | نفس المشكلة |
| `bool` (non-nullable) | ✅ | يعمل |
| `bool?` (nullable) | ❌ | نفس المشكلة |

### ⚠️ فخ إضافي: `[QueryProperty]` يُعيَّن **بعد** Constructor

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

---

## 🎯 القسم الثالث: الأنماط الأربعة الفعلية في المشروع

النمط **الأساسي** أعلاه هو نقطة البداية، لكن المشروع طوّر **4 أنماط متمايزة** تناسب سيناريوهات مختلفة. **كلها مُتبعة فعلياً في المشروع**، ولكل منها استخدامه الصحيح.

### 🌟 النمط 1: Constructor + LoadComponent (الأساسي — مستقر)

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

---

### 🌟 النمط 2: QueryProperty + Setter → LoadComponent (للمعاملات القليلة)

**متى يُستخدم:**
- عند وجود **1-2 معاملات** فقط (`QueryProperty`).
- عندما لا تكون البيانات ثقيلة جداً.
- **⚠️ لا يُنصح به لأكثر من 2-3 معاملات** (يسبب وميض).
- **⚠️ لا تستخدمه مع Nullable<T>** — استخدم `string` + تحويل يدوي.

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
- ✅ لا يحتاج `IQueryAttributable`.
- ✅ **يعمل بثبات في المشروع** — خاصة مع `CustomBlazorWebView`.
- ✅ مكونات Blazor تعيد التحميل تلقائياً عند وصول المعاملات.

**العيوب:**
- ⚠️ **يسبب وميضاً** عند إعادة التحميل.
- ⚠️ **مع 3+ معاملات**: `LoadComponent()` تُستدعى 3+ مرات → وميض شديد + طلبات API متكررة.
- ⚠️ **يحتاج `CustomBlazorWebView`** ليعمل بثبات.

---

### 🌟 النمط 3: Constructor + Refresh in Background (مع Cache)

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
        // ...
    }

    private async Task RefreshFromApiAsync()
    {
        // جلب من API
        var session = await _cachedSessionService.GetSessionAsync();

        // تحديث Preferences
        Preferences.Set("license_status_code", newStatusCode);

        // إذا تغيّر شيء، أعد تحميل UI
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
- ✅ تجربة مستخدم سلسة.

**العيوب:**
- ⚠️ يستخدم `static` variables (ليس نظيفاً معمارياً).
- ⚠️ يعيد إنشاء `RootComponent` عند التحديث (قد يسبب وميضاً بسيطاً).
- ⚠️ يحتاج `OnDisappearing` للتنظيف.

---

### 🌟 النمط 4: Constructor + OnComponentReady (للتفاعل MAUI ↔ Component)

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

    // ⭐ استدعاء method على المكون
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
            _licensePageRef?.SetFileData(fileName, bytes);  // ⭐ public method
        });
    }

    protected override void OnDisappearing()
    {
        base.OnDisappearing();
        _licensePageRef = null;   // تنظيف
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
            OnComponentReady?.Invoke(this);   // ⭐ إرسال مرجع this
        }
    }

    // ⭐ public method تستدعى من MAUI
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
- ⚠️ **الحذر**: `OnComponentReady` يُستدعى مرة واحدة فقط عند `firstRender`.

---

## 🌳 القسم الرابع: شجرة قرار — أي نمط أستخدم؟

```
هل تحتاج لتمرير معاملات (QueryProperty)؟
├── لا
│   ├── هل تحتاج لبيانات محلية (Cache)؟
│   │   ├── نعم → 🌟 النمط 3 (Constructor + Refresh in Background)
│   │   └── لا
│   │       ├── هل تحتاج لتفاعل MAUI → Component؟
│   │       │   ├── نعم → 🌟 النمط 4 (Constructor + OnComponentReady)
│   │       │   └── لا → 🌟 النمط 1 (Constructor + LoadComponent)
│   └── ...
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
    │
    └── ...
```

### جدول ملخص لاختيار النمط

| السيناريو | النمط المُوصى به | ملاحظات |
|-----------|:----------------:|---------|
| صفحة بدون معاملات، بدون تفاعل | النمط 1 | الأبسط والأكثر استقراراً |
| صفحة بمعامل واحد أو اثنين | النمط 2 | يعمل بثبات (بفضل `CustomBlazorWebView`) |
| صفحة ببيانات Cache محلية | النمط 3 | تجربة مستخدم ممتازة |
| صفحة بتفاعل MAUI → Component | النمط 4 | يحتاج `public methods` |
| صفحة بـ 3+ معاملات | **النمط 5 (هجين)** | انظر القسم التالي |
| **صفحة بـ Nullable QueryProperty** | **النمط 2 مع string** | استقبل كـ string، حوّل يدوياً |

---

## 🔀 القسم الخامس: الأنماط الهجينة

### النمط 5 (هجين): `IQueryAttributable` + `OnComponentReady` + `OnAppearing`

**متى يُستخدم:**
- عند وجود **3+ معاملات** (تجنب النمط 2).
- عند الحاجة لتحديث المكون بعد وصول المعاملات.
- عند الرغبة في تجنب الوميض.

**البنية:**

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
                }
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

    protected override void OnDisappearing()
    {
        base.OnDisappearing();
        _wrapperRef = null;
        _queryReady = false;
        try { blazorWebView.RootComponents.Clear(); } catch { }
    }
}
```

**المزايا:**
- ✅ **يستقبل كل المعاملات مرة واحدة** (لا وميض).
- ✅ **لا يعيد تحميل المكون** (لا طلبات API متكررة).
- ✅ **يحافظ على State المكون**.
- ✅ **✅ تم التحقق منه في المشروع** (8 أكتوبر 2026 — `ProgramDetailsPage`).

**العيوب:**
- ⚠️ **يحتاج `public method` على المكون** (`LoadProgram`).
- ⚠️ **يحتاج flag `_queryReady`** لمنع الاستدعاء قبل وصول الـ Query.

**✅ متى يعمل هذا النمط:**
- عندما يُستدعى `OnComponentReady` و `ApplyQueryAttributes` — **بأي ترتيب** — فإن `TryLoadProgram()` تُنفَّذ عند اكتمال الاثنين.

---

## 🚨 القسم الخامس-ب: تجنّب `Action<T>` مع `async`

### ⚠️ المشكلة (اكتشاف: 8 أكتوبر 2026)

تمرير `async` lambda كـ `Action<int>` يُخفي الـ Exception الحقيقي ويحوّله إلى `JavaProxyThrowable` المبهم على Android — مما يجعل التشخيص شبه مستحيل.

### ❌ النمط الخاطئ

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

**لا يخبرك بشيء.** لا السطر، لا السبب، لا الـ Inner Exception.

### ✅ النمط الصحيح

```csharp
// ⭐ لفّها في MainThread.BeginInvokeOnMainThread + try/catch
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
            // أو أظهر Alert للمستخدم
        }
    });
}) },
```

### 📌 القاعدة الذهبية

> **كل `Action<T>` يحتوي على `async` يجب أن:**
> 1. **يُلفّ في `MainThread.BeginInvokeOnMainThread`**
> 2. **يحتوي على `try/catch` صريح**
> 3. **يطبع الـ Exception الكامل** (`ex.ToString()` وليس `ex.Message`)

**هذا يحوّل `JavaProxyThrowable` المبهم إلى Exception واضح يمكن تشخيصه.**

### 📊 مقارنة: قبل/بعد

| قبل | بعد |
|-----|-----|
| `JavaProxyThrowable` فقط | Exception حقيقي مع Stack Trace كامل |
| لا نعرف السبب | نعرف السطر والسبب |
| ساعات من التشخيص | دقائق من التشخيص |

---

## 🚨 القسم السادس: `CustomBlazorWebView`

### 📌 ما هو؟

`CustomBlazorWebView` هو Wrapper مخصص حول `BlazorWebView` مُسجَّل في `RubikCare.Mobile.Controls`. يستخدمه **كل المشروع تقريباً** بدلاً من `BlazorWebView` العادي.

**المسار:** `RubikCare.Mobile/Controls/CustomBlazorWebView.cs`

### 📌 لماذا؟

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

### 📌 الاستخدام القياسي

```xml
<ContentPage xmlns="http://schemas.microsoft.com/dotnet/2021/maui"
             xmlns:x="http://schemas.microsoft.com/winfx/2009/xaml"
             xmlns:controls="clr-namespace:RubikCare.Mobile.Controls"
             x:Class="RubikCare.Mobile.Features.MyFlow">
    <controls:CustomBlazorWebView x:Name="blazorWebView" HostPage="wwwroot/index.html" />
</ContentPage>
```

### ✅ القاعدة

**استخدم `CustomBlazorWebView` دائماً** — لا `BlazorWebView` العادي.

---

## 🧠 القسم السابع: قاعدة ذهبية — لا تفترض، تحقق

**قبل كتابة أي كود يتفاعل مع `RootComponent` أو `BlazorWebView`**، تحقق من:

1. هل الخاصية/الطريقة موجودة في الـ IntelliSense؟
2. هل هذا النمط موثق في هذا الملف؟
3. هل هذا النمط يعمل في مشاريع MAUI الأخرى في الحل؟

**أمثلة على افتراضات فاشلة:**

| الافتراض | النتيجة |
|----------|---------|
| `RootComponent.Component` | ❌ غير موجود |
| `EventCallback.Factory.Create` في MAUI | ⚠️ يعمل أحياناً — يُفضَّل `Action` |
| `StateHasChanged()` من خارج الـ Component | ❌ غير قابل للوصول |
| `[QueryProperty]` يعمل مع `int?` | ❌ يفشل بـ InvalidCastException |

---

## 🔴 القسم الثامن: تشخيص الصفحة الفارغة

هذا هو أكثر سيناريو مربك لأنه **لا يعطي أي خطأ**. اتبع هذه الخطوات بالترتيب:

### الخطوة 1: تأكد أن MAUI layer يعمل

```csharp
public MyFlowPage()
{
    InitializeComponent();
    Debug.WriteLine("🔵 Constructor START");
    // ...
    Debug.WriteLine($"🟢 RootComponents: {blazorWebView.RootComponents.Count}");
}
```

| النتيجة | المشكلة |
|---------|---------|
| لا يظهر "Constructor START" | مشكلة في تسجيل الصفحة أو الـ Routing |
| يظهر لكن Count = 0 | مشكلة في `RootComponents.Add` |
| Count = 1 | انتقل للخطوة 2 |

### الخطوة 2: تأكد أن Blazor runtime يعمل

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

### الخطوة 3: عزّل مشكلة `@inject`

أضف `@inject` الخاصة بمكونك واحداً واحداً لـ TestPage حتى تجد المشكلة.

**توقفت عند خدمة معينة** → هذه الخدمة غير مسجلة في حاوية Blazor → حوّلها إلى `[Parameter]`.

### الخطوة 4: تأكد من `OnInitializedAsync`

```csharp
protected override async Task OnInitializedAsync()
{
    Debug.WriteLine("🔵 OnInitializedAsync START");
    // ...
    Debug.WriteLine("🟢 OnInitializedAsync END");
}
```

| النتيجة | المشكلة |
|---------|---------|
| لا يظهر START | مشكلة DI (الخطوة 3) |
| يظهر START لكن لا يظهر END | Exception في المنتصف — ابحث عن try/catch مفقود |

### الخطوة 5 (جديدة): إذا ظهر `JavaProxyThrowable`

**السبب المحتمل:** `Action<T>` مع `async` — راجع القسم 5-ب.

**الحل:** لفّ الـ `Action` في `MainThread.BeginInvokeOnMainThread` + `try/catch`.

---

## 📁 القسم التاسع: هيكل المشروع مع BlazorWebView

```
RubikCare.Mobile/
├── Controls/
│   └── CustomBlazorWebView.cs          # ⭐ Wrapper لكل BlazorWebView
│
├── Features/
│   ├── Shared/Views/                    # حاويات MAUI للصفحات المشتركة
│   │   ├── PharmacySearchFlow.xaml
│   │   ├── PharmacyDetailFlow.xaml
│   │   └── ...
│   │
│   ├── PSP/Doctor/Views/
│   │   ├── ProgramDetailsPage.xaml     # نمط 5 (Hybrid)
│   │   └── PointsDashboardPage.xaml    # نمط 5 مع QueryProperty (string)
│   │
│   └── ProfessionalOnboarding/Views/
│       ├── ProfessionalStatusFlow.xaml
│       └── ProfessionalLicenseFlow.xaml
│
RubikCare.Shared.UI/                     # RCL للمكونات المشتركة
└── Components/
    ├── Patient/Settings/
    │   ├── SettingsPage.razor
    │   ├── ProfessionalStatusPage.razor
    │   └── ProfessionalLicensePage.razor
    ├── PharmacySearch/
    │   └── PharmacySearchPage.razor
    ├── Pharmacy/
    │   └── PharmacyDetailPage.razor
    └── PSP/
        ├── ProgramDetailsComponent.razor
        ├── ProgramDetailsWrapper.razor  # ⭐ يستقبل Parameters + Bridges
        └── Points/
            └── PointDashboard.razor
```

---

## 📋 القسم العاشر: Checklist عند إنشاء صفحة BlazorWebView جديدة

### MAUI Layer (صفحة الحاوية)

- [ ] هل تستخدم `CustomBlazorWebView` بدلاً من `BlazorWebView` العادي؟
- [ ] هل اخترت النمط الصحيح من القسم الرابع (شجرة القرار)؟
- [ ] هل `RootComponents.Add` يُستدعى بشكل متزامن في الكونستركتور (أو Setter للنمط 2)؟
- [ ] هل `ApiService` يُمرر كـ `(IApiService)_apiService`؟
- [ ] هل تستخدم `Action` بدلاً من `EventCallback` لتمرير الدوال (نمط MAUI)؟
- [ ] هل `OnDisappearing` تستدعي `RootComponents.Clear()` وتنظف `_pageRef`؟
- [ ] **هل `[QueryProperty]` من نوع `string` وليس `Nullable<T>`؟**
- [ ] **هل `Action<T>` مع `async` ملفوف في `MainThread.BeginInvokeOnMainThread` + `try/catch`؟**
- [ ] **إذا كان `[QueryProperty]` مستخدماً — هل `LoadBlazorComponent` في `OnAppearing` وليس Constructor؟**

### Blazor Component (المكون)

- [ ] هل المكون **لا يستخدم** `@inject HttpClient` أو `@inject ApiService`؟
- [ ] هل كل خدمة MAUI تُستخدم عبر `[Parameter]` وليس `@inject` (ما عدا الترجمة و `NavigationManager`)؟
- [ ] هل `OnInitializedAsync` تُستدعى؟ (تأكد من Debug Output)
- [ ] هل المكون يعرض Skeleton/Loading أثناء جلب البيانات؟
- [ ] هل توجد `public methods` للتحديث من الحاوية عند الحاجة (النمط 4/5)؟
- [ ] **هل الـ Parameters من نوع `Action<T>` (وليس `EventCallback`)؟**

---

## 📝 القسم الحادي عشر: نقاط تحتاج توثيقاً (TODO)

النقاط التالية تحتاج تأكيداً من الفريق:

1. **لماذا يعمل النمط 2 (`QueryProperty + Setter`) رغم تحذير القسم الثاني؟**
   - هل بسبب `CustomBlazorWebView`؟
   - هل بسبب `RootComponents.Clear()` الذي يعمل فعلياً؟

2. **ما الفرق الدقيق بين `CustomBlazorWebView` و `BlazorWebView`؟**
   - ما الميزات الإضافية؟
   - ما المشاكل التي يحلها؟

3. **هل `static variables` في النمط 3 مقصودة؟**
   - ما الفائدة المعمارية؟

4. **✅ النمط الهجين (5) — تم التحقق منه (8 أكتوبر 2026)**
   - يعمل بنجاح في `ProgramDetailsPage` مع `IQueryAttributable`.
   - الشرط: `OnComponentReady` و `ApplyQueryAttributes` — بأي ترتيب — ثم `TryLoadProgram()`.
   - الإصلاح: flag `_queryReady` + `_wrapperRef`.
   - **لم يعد نقطة مفتوحة.**

5. **ما هو `IBackButtonHandler`؟**
   - كيف يعمل؟
   - متى نستخدمه؟

---

## 📖 القسم الثاني عشر: قصة نجاح موثقة — `PointsDashboardPage`

### السياق

صفحة `PointsDashboardPage` كانت لا تفتح في MAUI (تعمل في PWA). الخطأ الظاهر: `JavaProxyThrowable` المبهم.

### التشخيص (بعد ساعات من التحقيق)

**3 مشاكل متراكبة:**

1. **`ProgramId` لم يُستقبل أصلاً** — كان `[QueryProperty]` يُعيَّن بعد `LoadBlazorComponent()`، فيبقى `ProgramId = null` عند بناء المكون.

2. **`[QueryProperty]` مع `int?`** — كان `public int? ProgramId`، ففشل Shell في تحويل `"1"` إلى `int?` مع `InvalidCastException`.

3. **`Action<int>` + `async`** — كانت `GoToPointsDashboardAsync` تُلفّ في `Action<int>(async ...)` بدون `try/catch`، فتحوّل الخطأ إلى `JavaProxyThrowable`.

### الحل النهائي

**في `PointsDashboardPage.xaml.cs`:**

```csharp
[QueryProperty(nameof(ProgramIdRaw), "programId")]   // ✅ string وليس int?
public partial class PointsDashboardPage : ContentPage
{
    public string? ProgramIdRaw { get; set; }
    
    private int? ProgramId => int.TryParse(ProgramIdRaw, out var id) ? id : null;
    
    private bool _componentLoaded = false;
    
    protected override void OnAppearing()   // ✅ هنا وليس في Constructor
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
{ "OnGoToPointsDashboard", new Action<int>((programId) =>
{
    MainThread.BeginInvokeOnMainThread(async () =>
    {
        try { await GoToPointsDashboardAsync(programId); }
        catch (Exception ex) { Debug.WriteLine($"❌ {ex}"); }
    });
}) },
```

### النتيجة

✅ الصفحة تفتح بنجاح في MAUI.
✅ الـ Log يظهر القيم الصحيحة (`OrgId=1247, ProgramId=1`).
✅ لا `JavaProxyThrowable`.
✅ البطاقة تعمل بعد إصلاح `CheckPointsVisibility` (Fallback لـ `OrganizationId`).

### الدروس المستفادة

1. **`[QueryProperty]` ≠ `Nullable<T>`** — استخدم `string` دائماً.
2. **`[QueryProperty]` يُعيَّن بعد Constructor** — استخدم `OnAppearing` مع flag.
3. **`Action<T>` + `async` = `JavaProxyThrowable`** — لفّها بـ `try/catch`.
4. **"الفشل الصامت" في MAUI** — قد يكون DI، قد يكون QueryProperty، قد يكون Action.

---

## 🔗 روابط ذات صلة

- [00 - الهيكل المعماري](./00-architecture-overview.md)
- [05 - إنشاء الصفحات والمكونات](./05-page-creation-checklist.md)
- [10 - دليل تطوير MAUI](./10-maui-development-guide.md)
- [13 - Clean Architecture Enforcement](./13-clean-architecture-enforcement.md)
- [15 - نظام ترجمة الموبايل](./15-translation-system.md)
- [27 - الديون التقنية](./27-technical-debt.md)

---

**آخر تحديث:** 8 أكتوبر 2026 | **الإصدار:** 2.0 | **الملف:** `11-blazor-webview-guide.md`
```

---

## ✅ ملخص ما تغيّر في الإصدار 2.0

| # | التغيير | القسم |
|:---:|:---|:---|
| **1** | 🆕 قسم كامل — `[QueryProperty]` مع `Nullable<T>` | القسم 2-ب |
| **2** | 🆕 قسم كامل — `Action<T>` مع `async` | القسم 5-ب |
| **3** | 🆕 قصة نجاح موثقة — `PointsDashboardPage` | القسم 12 |
| **4** | تحديث شجرة القرار — إضافة سؤال Nullable | القسم 4 |
| **5** | تحديث القسم 11 — النمط 5 مُتحقَّق منه | القسم 11 |
| **6** | تحديث Checklist — إضافة 3 فحوصات جديدة | القسم 10 |
| **7** | تحديث الخطوة 5 في تشخيص الصفحة الفارغة | القسم 8 |
| **8** | تحديث هيكل المشروع — إضافة `PointsDashboardPage` | القسم 9 |
| **9** | تحديث قائمة الافتراضات الفاشلة | القسم 7 |

---


**هذه الوثيقة الآن "ذهبية" — تمنع كل الأخطاء التي واجهناها اليوم.** 🚀

**هل تريد تحديثاً مشابهاً لأي وثيقة أخرى؟**
