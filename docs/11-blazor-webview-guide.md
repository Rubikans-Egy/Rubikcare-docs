# 11 - دليل BlazorWebView والأنماط الناجحة

**آخر تحديث: 13 يونيو 2026**

---

## 📌 مقدمة

هذا المرجع يوثق **الأنماط الناجحة** وما يجب تجنبه عند استخدام BlazorWebView داخل تطبيق MAUI. الهدف هو مشاركة المكونات بين Web و Mobile مع تجنب المشاكل المعروفة.

---

## لماذا BlazorWebView؟

| XAML التقليدي | BlazorWebView |
|---------------|---------------|
| لغة مختلفة عن Web | نفس HTML/CSS المستخدم في Web |
| مشاكل Binding متكررة | لا Binding - استخدام Razor |
| صعوبة مشاركة المكونات | مشاركة مباشرة عبر RCL |
| منحنى تعلم منفصل | نفس مهارات Blazor |

---

## 🚨 القسم الأول: مشكلة Dependency Injection في BlazorWebView

### ⚠️ المشكلة الجذرية (اكتشاف: 13 يونيو 2026)

`BlazorWebView` في MAUI يستخدم حاوية Dependency Injection **مستقلة** عن حاوية MAUI الرئيسية. هذا يعني أن الخدمات المسجلة في `MauiProgram.cs` قد لا تكون متاحة لمكونات Blazor الداخلية.

### ❌ العرض: فشل صامت للمكون

عندما يستخدم مكون Blazor `@inject` لخدمة غير مسجلة في حاوية Blazor الداخلية، **يفشل المكون في التحميل تماماً بدون أي خطأ في Debug Output**. الصفحة تظهر فارغة أو لا تظهر على الإطلاق.

> **ملاحظة مهمة:** هذا الفشل صامت تماماً — لا exception، لا خطأ في الـ console، لا أي مؤشر. `OnInitializedAsync` لا تُستدعى أبداً.

### 📊 الخدمات التي تعمل وتفشل مع `@inject`

| الخدمة | `@inject` في Blazor | التفسير |
|--------|---------------------|---------|
| `ISharedTranslationService` | ✅ يعمل | مسجل في حاوية Blazor |
| `ISharedTranslationState` | ✅ يعمل | مسجل في حاوية Blazor |
| `NavigationManager` | ✅ يعمل | مسجل تلقائياً من Blazor runtime |
| `HttpClient` | ❌ يفشل بصمت | غير مسجل في حاوية Blazor |
| `ApiService` | ❌ يفشل بصمت | غير مسجل في حاوية Blazor |
| `IMobileTranslationService` | ❌ يفشل بصمت | غير مسجل في حاوية Blazor |

### ✅ الحل: تمرير الخدمات كـ Parameters

بدلاً من `@inject`، استخدم `[Parameter]` ومرر الخدمة من صفحة MAUI المضيفة عبر `IApiService` (Interface موجود في `Shared.UI`):

```razor
@* ❌ خطأ - في مكون Blazor - سيفشل بصمت في MAUI *@
@inject HttpClient Http
@inject ApiService ApiService

@* ✅ صحيح - في مكون Blazor *@
@using RubikCare.Shared.UI.Services
[Parameter] public IApiService? ApiService { get; set; }
```

```csharp
// ✅ في صفحة MAUI المضيفة - مرر الخدمة كـ Parameter
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
- [ ] هل تستخدم `@inject HttpClient Http`؟ → **احذفه فوراً**
- [ ] هل تستخدم `@inject` لأي خدمة غير `ISharedTranslation*` أو `NavigationManager`؟
- [ ] هل `OnInitializedAsync` لا تُستدعى أبداً (لا يظهر Debug Output)؟
- [ ] هل الصفحة فارغة تماماً بدون أي خطأ؟

**إذا أجبت بنعم على أي سؤال، استبدل `@inject` بـ `[Parameter]`.**

---

## 🚨 القسم الثاني: قيود RootComponents في BlazorWebView

### ⚠️ السبب الجذري لقيود التوقيت

`BlazorWebView` على Windows يستخدم `WebView2` كمحرك تحته. هذا المحرك **يربط الـ DOM بمكونات Blazor فقط أثناء دورة حياة الكونستركتور** للـ ContentPage المضيفة.

أي محاولة لإضافة `RootComponents` بعد انتهاء الكونستركتور — سواء في `OnAppearing`، أو بعد `await`، أو في `Task` منفصل — **لن تُحمَّل في الـ DOM** وستبقى الصفحة فارغة بدون أي خطأ.

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
    blazorWebView.RootComponents.Add(...); // ← متأخر جداً، WebView2 لم يعد ينتظر
}

// ❌ النمط 2: في OnAppearing
protected override void OnAppearing()
{
    base.OnAppearing();
    blazorWebView.RootComponents.Add(...); // ← خارج دورة الكونستركتور — لن يعمل
}

// ❌ النمط 3: Clear ثم Add بعد تحميل البيانات
private async Task RefreshWithData()
{
    var data = await apiService.GetDataAsync();
    blazorWebView.RootComponents.Clear();
    blazorWebView.RootComponents.Add(...); // ← لن يُعاد تحميل المكون
}
```

### ✅ النمط الصحيح الوحيد

```csharp
// ✅ RootComponents.Add بشكل متزامن في الكونستركتور
public MyFlowPage()
{
    InitializeComponent();
    _apiService = IPlatformApplication.Current!.Services
        .GetRequiredService<ApiService>();

    // الخطوة 1: بيانات محلية (متزامن)
    LoadCachedData();

    // الخطوة 2: إضافة المكون (متزامن - إلزامي في الكونستركتور)
    LoadBlazorComponent();

    // الخطوة 3: تحديث من API (غير متزامن - لا يعيد تحميل المكون)
    _ = RefreshFromApiAsync();
}
```

### ⚠️ تحذير: مشكلة `autostart="false"` مع `window.__blazorStarted`

إذا كان `index.html` يستخدم هذا النمط:

```html
<script src="_framework/blazor.webview.js" autostart="false"></script>
<script>
    window.addEventListener('DOMContentLoaded', function () {
        setTimeout(function () {
            if (typeof Blazor !== 'undefined' && !window.__blazorStarted) {
                window.__blazorStarted = true; // ← هذا هو المشكلة
                Blazor.start();
            }
        }, 100);
    });
</script>
```

فإن **أول صفحة Blazor تعمل**، لكن أي صفحة Blazor تُفتح لاحقاً في نفس الجلسة **تفشل بصمت** لأن `window.__blazorStarted = true` يمنع استدعاء `Blazor.start()` مرة ثانية.

**الحل:** استخدم `autostart` الافتراضي بدون flag:

```html
<!-- ✅ سطر واحد بدون flag -->
<script src="_framework/blazor.webview.js"></script>
```

> **ملاحظة:** في RubikCare تم التحقق من أن هذه المشكلة **ليست** السبب الفعلي للأعطال الحالية (السبب كان `@inject HttpClient`)، لكنها موثقة هنا لتجنبها مستقبلاً.

---

## 🔴 تشخيص: الصفحة تظهر فارغة تماماً

هذا هو أكثر سيناريو مربك لأنه **لا يعطي أي خطأ**. اتبع هذه الخطوات بالترتيب:

### الخطوة 1: تأكد أن MAUI layer يعمل

أضف Debug في الكونستركتور:

```csharp
public MyFlowPage()
{
    InitializeComponent();
    System.Diagnostics.Debug.WriteLine("🔵 Constructor START");

    // ... كودك ...

    System.Diagnostics.Debug.WriteLine($"🟢 RootComponents: {blazorWebView.RootComponents.Count}");
}
```

- **لا يظهر "Constructor START"** → مشكلة في تسجيل الصفحة أو الـ routing
- **يظهر لكن Count = 0** → مشكلة في `RootComponents.Add`
- **Count = 1** → انتقل للخطوة 2

### الخطوة 2: تأكد أن Blazor runtime يعمل

استبدل مكونك مؤقتاً بـ TestPage بسيطة **بدون أي `@inject`**:

```razor
@namespace RubikCare.Shared.UI.Components

<h1 style="color:red;font-size:48px;">✅ BLAZOR WORKS</h1>

@code {
    protected override void OnInitialized()
    {
        System.Diagnostics.Debug.WriteLine("🔴 TestPage: OnInitialized CALLED");
    }
}
```

```csharp
// في الكونستركتور مؤقتاً
blazorWebView.RootComponents.Add(new RootComponent
{
    Selector = "#app",
    ComponentType = typeof(TestPage),
    Parameters = new Dictionary<string, object?>()
});
```

- **لم تظهر TestPage** → مشكلة في WebView2 أو `index.html` أو تسجيل الـ services
- **ظهرت TestPage** → انتقل للخطوة 3

### الخطوة 3: عزّل مشكلة `@inject`

أضف `@inject` الخاصة بمكونك واحداً واحداً لـ TestPage حتى تجد المشكل:

```razor
@* اختبر واحداً في كل مرة *@
@inject ISharedTranslationService TranslationService  @* ← هل تظهر؟ *@
@inject ISharedTranslationState TranslationState      @* ← هل تظهر؟ *@
@inject HttpClient Http                               @* ← على الأرجح هنا المشكلة *@
```

- **توقفت عند خدمة معينة** → هذه الخدمة غير مسجلة في حاوية Blazor → حوّلها إلى `[Parameter]`

### الخطوة 4: تأكد من `OnInitializedAsync`

```csharp
protected override async Task OnInitializedAsync()
{
    System.Diagnostics.Debug.WriteLine("🔵 OnInitializedAsync START");
    // ...
    System.Diagnostics.Debug.WriteLine("🟢 OnInitializedAsync END");
}
```

- **لا يظهر START** → مشكلة DI (الخطوة 3)
- **يظهر START لكن لا يظهر END** → exception في المنتصف — ابحث عن try/catch مفقود

---

## هيكل المشروع مع BlazorWebView

```
RubikCare.Mobile/
├── Features/
│   ├── Shared/Views/                    # حاويات MAUI للصفحات المشتركة
│   │   ├── SettingsFlow.xaml
│   │   ├── ProfessionalStatusFlow.xaml
│   │   └── CreateOrganizationFlow.xaml
│   │
│   └── XamlPages/                       # صفحات XAML الموجودة (تبقى كما هي)
│
RubikCare.Shared.UI/                     # RCL للمكونات المشتركة
└── Components/
    ├── Patient/Settings/
    │   ├── SettingsPage.razor
    │   └── ProfessionalStatusPage.razor
    └── OrganizationManagement/
        └── CreateOrganizationPage.razor
```

---

## الأنماط الناجحة (يجب اتباعها)

### 1. النمط القياسي الكامل لصفحة BlazorWebView جديدة

```csharp
public partial class MyFlowPage : ContentPage
{
    private readonly ApiService _apiService;

    // بيانات محلية مؤقتة
    private string _cachedValue = "";

    public MyFlowPage()
    {
        InitializeComponent();
        _apiService = IPlatformApplication.Current!.Services
            .GetRequiredService<ApiService>();

        // الخطوة 1: تحميل البيانات المكاشة (متزامن)
        LoadCachedData();

        // الخطوة 2: إضافة المكون مع البيانات (متزامن - إلزامي!)
        LoadBlazorComponent();

        // الخطوة 3: تحديث من API في الخلفية (غير متزامن)
        _ = RefreshFromApiAsync();
    }

    private void LoadCachedData()
    {
        _cachedValue = Preferences.Get("my_cached_key", "");
    }

    private void LoadBlazorComponent()
    {
        blazorWebView.RootComponents.Clear();

        var parameters = new Dictionary<string, object?>
        {
            { "Param1", _cachedValue },
            { "ApiService", (IApiService)_apiService }, // ⭐ مرر عبر Interface
            { "OnAction", EventCallback.Factory.Create<MyDto>(this, HandleAction) }
        };

        blazorWebView.RootComponents.Add(new RootComponent
        {
            Selector = "#app",
            ComponentType = typeof(MyBlazorPage),
            Parameters = parameters
        });
    }

    private async Task RefreshFromApiAsync()
    {
        try
        {
            var data = await _apiService.GetAsync<MyDto>("api/my-endpoint");
            if (data != null)
            {
                Preferences.Set("my_cached_key", data.Value);
                // ملاحظة: لا تُعد إضافة RootComponent هنا
                // المكون يتعامل مع تحديث البيانات داخلياً عبر ApiService Parameter
            }
        }
        catch (Exception ex)
        {
            System.Diagnostics.Debug.WriteLine($"❌ Refresh error: {ex.Message}");
        }
    }

    private async Task HandleAction(MyDto data)
    {
        // معالجة الـ callback من المكون
    }

    protected override void OnDisappearing()
    {
        base.OnDisappearing();
        try { blazorWebView.RootComponents.Clear(); } catch { }
    }
}
```

### 2. النمط القياسي لمكون Blazor في Shared.UI

```razor
@namespace RubikCare.Shared.UI.Components.MyFeature
@using RubikCare.Shared.UI.Services
@implements IDisposable

@inject ISharedTranslationService TranslationService   @* ✅ مسموح *@
@inject ISharedTranslationState TranslationState       @* ✅ مسموح *@
@* ❌ لا تضف @inject HttpClient أو أي خدمة MAUI هنا *@

<div>
    @if (IsLoading)
    {
        <div class="skeleton-loader"></div>
    }
    else
    {
        @* المحتوى الفعلي *@
    }
</div>

@code {
    // ✅ Parameters من MAUI
    [Parameter] public string SomeParam { get; set; } = "";
    [Parameter] public IApiService? ApiService { get; set; }         @* ✅ عبر Interface *@
    [Parameter] public EventCallback<MyDto> OnAction { get; set; }

    // State داخلي
    private bool IsLoading = true;
    private List<MyItemDto> _items = new();

    protected override async Task OnInitializedAsync()
    {
        System.Diagnostics.Debug.WriteLine("🔵 MyPage: OnInitializedAsync START");

        await LoadTranslations();

        if (ApiService != null)
        {
            await LoadDataAsync();
        }

        IsLoading = false;
        System.Diagnostics.Debug.WriteLine($"🟢 MyPage: Done, Items={_items.Count}");
    }

    private async Task LoadDataAsync()
    {
        try
        {
            var result = await ApiService!.GetAsync<List<MyItemDto>>("api/my-endpoint");
            if (result != null) _items = result;
        }
        catch (Exception ex)
        {
            System.Diagnostics.Debug.WriteLine($"❌ LoadData: {ex.Message}");
        }
    }

    private async Task LoadTranslations()
    {
        var lang = TranslationState.CurrentLanguage;
        var trans = await TranslationService.GetPageTranslationsAsync("MY_PAGE", lang);
        // ...
    }

    public void Dispose()
    {
        TranslationState.OnLanguageChanged -= HandleLanguageChanged;
    }
}
```

### 3. استخدام RCL للمكونات المشتركة

```razor
@* في RubikCare.Shared.UI *@
@* Components/RubikButton.razor *@

<button class="btn @ColorClass" @onclick="OnClick">
    @if (!string.IsNullOrEmpty(Icon))
    {
        <i class="@Icon me-1"></i>
    }
    @Text
</button>

@code {
    [Parameter] public string Text { get; set; } = "";
    [Parameter] public string Icon { get; set; } = "";
    [Parameter] public string ColorClass { get; set; } = "btn-primary";
    [Parameter] public EventCallback OnClick { get; set; }
}
```

### 4. التعامل مع الترجمة

```csharp
// ⭐ استخدم ISharedTranslationService (مدعوم في BlazorWebView)
@inject ISharedTranslationService TranslationService
@inject ISharedTranslationState TranslationState

private Dictionary<string, string> _translations = new();
private string T(string key) =>
    _translations.TryGetValue(key, out var value) ? value : key;

protected override async Task OnInitializedAsync()
{
    var lang = TranslationState.CurrentLanguage;
    var pageTrans = await TranslationService.GetPageTranslationsAsync(PageDomain, lang);
    var commonTrans = await TranslationService.GetPageTranslationsAsync("COMMON", lang);
    _translations = commonTrans.Concat(pageTrans)
        .GroupBy(x => x.Key)
        .ToDictionary(g => g.Key, g => g.First().Value);
}
```

---

## ما يجب تجنبه (Anti-Patterns)

### ❌ 1. استخدام `@inject` لخدمات غير مسجلة في Blazor

```razor
@* ❌ خطأ - سيفشل المكون بصمت في MAUI بدون أي خطأ *@
@inject HttpClient Http
@inject ApiService ApiService
@inject IMobileTranslationService MobileTranslation

@* ✅ صحيح - استخدم Parameters *@
[Parameter] public IApiService? ApiService { get; set; }
```

### ❌ 2. إضافة RootComponents بشكل غير متزامن

```csharp
// ❌ خطأ - المكون لن يُحمّل
public MyFlowPage()
{
    InitializeComponent();
    _ = LoadAndShowAsync(); // async - لن يعمل!
}

// ✅ صحيح
public MyFlowPage()
{
    InitializeComponent();
    LoadBlazorComponent(); // متزامن - يعمل!
    _ = RefreshFromApiAsync(); // غير متزامن - للتحديث فقط
}
```

### ❌ 3. إعادة إضافة RootComponent لتحديث البيانات

```csharp
// ❌ خطأ - لن يُعاد تحميل المكون
private async Task UpdateData()
{
    var newData = await _apiService.GetAsync<MyDto>("api/data");
    blazorWebView.RootComponents.Clear();
    blazorWebView.RootComponents.Add(...); // المكون لن يظهر
}

// ✅ صحيح - المكون يحمّل بياناته بنفسه عبر ApiService Parameter
// لا تعيد إضافة RootComponent بعد الكونستركتور
```

### ❌ 4. استدعاء Native APIs مباشرة من Blazor

```csharp
// ❌ خطأ - سيسبب استثناء
await SecureStorage.SetAsync("key", "value");
var pref = Preferences.Get("key", "");

// ✅ صحيح - استخدم خدمة وسيطة تُمرر كـ Parameter
[Parameter] public IStorageService? StorageService { get; set; }
await StorageService!.SecureSetAsync("key", "value");
```

### ❌ 5. Hardcoding المسارات

```csharp
// ❌ خطأ
Navigation.NavigateTo("/psp/patient/5/details");

// ✅ صحيح
Navigation.NavigateTo($"{Routes.PspPatientDetails}/{patientId}");
```

---

## التكامل مع XAML

### فتح صفحة Blazor من XAML

```csharp
await Shell.Current.GoToAsync("blazorPageRoute");
```

### فتح صفحة XAML من Blazor

```razor
@* عبر EventCallback من الـ Parameter *@
[Parameter] public EventCallback OnNavigateToXaml { get; set; }

<button @onclick="OnNavigateToXaml">انتقل</button>
```

---

## CHECKLIST: عند إنشاء صفحة BlazorWebView جديدة

### MAUI Layer (صفحة الحاوية)
- [ ] هل المكون موجود في `RubikCare.Shared.UI`؟
- [ ] هل `RootComponents.Add` يُستدعى **بشكل متزامن** في الكونستركتور؟
- [ ] هل `ApiService` يُمرر كـ `(IApiService)_apiService`؟
- [ ] هل `OnDisappearing` تستدعي `RootComponents.Clear()`؟

### Blazor Component (المكون)
- [ ] هل المكون **لا يستخدم** `@inject HttpClient`؟
- [ ] هل كل خدمة MAUI تُستخدم عبر `[Parameter]` وليس `@inject`؟
- [ ] هل `OnInitializedAsync` تُستدعى؟ (تأكد من Debug Output)
- [ ] هل المكون يتعامل مع حالة `ApiService == null`؟
- [ ] هل المكون يعرض Skeleton/Loading أثناء جلب البيانات؟
- [ ] هل مفاتيح الترجمة مسجلة في قاعدة البيانات؟
- [ ] هل الصفحة متجاوبة مع أحجام الشاشات المختلفة؟

---

## 🔗 روابط ذات صلة

- [00 - الهيكل المعماري](https://github.com/Rubikans-Egy/Rubikcare-docs/blob/main/docs/00-architecture-overview.md)
- [05 - إنشاء الصفحات والمكونات](https://github.com/Rubikans-Egy/Rubikcare-docs/blob/main/docs/05-page-creation-checklist.md)
- [10 - دليل تطوير MAUI](https://github.com/Rubikans-Egy/Rubikcare-docs/blob/main/docs/10-maui-development-guide.md)
- [13 - Clean Architecture Enforcement](https://github.com/Rubikans-Egy/Rubikcare-docs/blob/main/docs/13-clean-architecture-enforcement.md)
- [15 - نظام ترجمة الموبايل](https://github.com/Rubikans-Egy/Rubikcare-docs/blob/main/docs/15-mobile-translation-system.md)

---

**آخر تحديث:** 13 يونيو 2026 | **الملف:** `11-blazor-webview-guide.md`
