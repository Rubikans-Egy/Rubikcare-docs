
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

## 🚨 القسم الجديد: مشكلة Dependency Injection في BlazorWebView

### ⚠️ المشكلة الجذرية (اكتشاف: 13 يونيو 2026)

`BlazorWebView` في MAUI يستخدم حاوية Dependency Injection **مستقلة** عن حاوية MAUI الرئيسية. هذا يعني أن الخدمات المسجلة في `MauiProgram.cs` قد لا تكون متاحة لمكونات Blazor الداخلية.

### ❌ العرض: فشل صامت للمكون

عندما يستخدم مكون Blazor `@inject` لخدمة غير مسجلة في حاوية Blazor الداخلية، **يفشل المكون في التحميل تماماً بدون أي خطأ في Debug Output**. الصفحة تظهر فارغة أو لا تظهر على الإطلاق.

### 📊 الخدمات التي تعمل وتفشل مع `@inject`

| الخدمة | `@inject` في Blazor | التفسير |
|--------|---------------------|---------|
| `ISharedTranslationService` | ✅ يعمل | مسجل في حاوية Blazor |
| `ISharedTranslationState` | ✅ يعمل | مسجل في حاوية Blazor |
| `HttpClient` | ❌ يفشل بصمت | غير مسجل في حاوية Blazor |
| `ApiService` | ❌ يفشل بصمت | غير مسجل في حاوية Blazor |
| `IMobileTranslationService` | ❌ يفشل بصمت | غير مسجل في حاوية Blazor |

### ✅ الحل: تمرير الخدمات كـ Parameters

بدلاً من `@inject`، استخدم `[Parameter]` ومرر الخدمة من صفحة MAUI المضيفة:

```csharp
// ❌ خطأ - في مكون Blazor
@inject HttpClient Http
@inject ApiService ApiService

// ✅ صحيح - في مكون Blazor
[Parameter] public IApiService? ApiService { get; set; }

// ✅ في حاوية MAUI - مرر الخدمة
private readonly ApiService _apiService;

public MyFlowPage()
{
    InitializeComponent();
    _apiService = IPlatformApplication.Current!.Services.GetRequiredService<ApiService>();
    
    var parameters = new Dictionary<string, object?>
    {
        { "ApiService", _apiService }
    };
    
    blazorWebView.RootComponents.Add(new RootComponent
    {
        Selector = "#app",
        ComponentType = typeof(MyBlazorPage),
        Parameters = parameters
    });
}
```

### 🔍 Checklist تشخيص المشكلة

عندما يفشل مكون Blazor في التحميل داخل MAUI:

- [ ] هل المكون يعمل في Web Blazor لكن ليس في MAUI؟
- [ ] هل تستخدم `@inject` لخدمة غير `ISharedTranslation*` أو `NavigationManager`؟
- [ ] هل `OnInitializedAsync` لا تُستدعى أبداً (لا يظهر Debug Output)؟
- [ ] هل الصفحة فارغة تماماً بدون أي خطأ؟

**إذا أجبت بنعم على أي سؤال، استبدل `@inject` بـ `[Parameter]`.**

---

## هيكل المشروع مع BlazorWebView

```
RubikCare.Mobile/
├── Features/
│   ├── Shared/Views/                    # حاويات MAUI للصفحات المشتركة
│   │   ├── SettingsFlow.xaml           # حاوية + كود خلفي
│   │   ├── ProfessionalStatusFlow.xaml
│   │   └── CreateOrganizationFlow.xaml
│   │
│   └── XamlPages/                       # صفحات XAML الموجودة
│       └── ... (تبقى كما هي)
│
└── Shared.UI/                           # RCL للمكونات المشتركة
    └── Components/
        ├── Patient/Settings/
        │   ├── SettingsPage.razor       # مكون Blazor مشترك
        │   └── ProfessionalStatusPage.razor
        └── OrganizationManagement/
            └── CreateOrganizationPage.razor
```

---

## الأنماط الناجحة (يجب اتباعها)

### 1. النمط القياسي لصفحة BlazorWebView جديدة

**الخطوات الإلزامية بالترتيب:**

```csharp
public partial class MyFlowPage : ContentPage
{
    private readonly ApiService _apiService;

    public MyFlowPage()
    {
        InitializeComponent();
        _apiService = IPlatformApplication.Current!.Services.GetRequiredService<ApiService>();

        // الخطوة 1: تحميل البيانات المكاشة محلياً (متزامن)
        LoadCachedData();

        // الخطوة 2: إضافة المكون مع البيانات (متزامن - إلزامي!)
        LoadBlazorComponent();

        // الخطوة 3: تحديث من API في الخلفية (غير متزامن)
        _ = RefreshFromApiAsync();
    }

    private void LoadBlazorComponent()
    {
        blazorWebView.RootComponents.Clear();

        var parameters = new Dictionary<string, object?>
        {
            { "Param1", localValue },
            { "ApiService", _apiService },  // ⭐ مرر ApiService كـ Parameter
            { "OnAction", EventCallback.Factory.Create(...) }
        };

        blazorWebView.RootComponents.Add(new RootComponent
        {
            Selector = "#app",
            ComponentType = typeof(MyBlazorPage),
            Parameters = parameters
        });
    }
}
```

### 2. استخدام RCL للمكونات المشتركة

```razor
@* في RubikCare.Shared.UI - مكون مشترك *@
@* Components/RubikButton.razor *@

<button class="btn @ColorClass" @onclick="OnClick">
    @if (!string.IsNullOrEmpty(Icon))
    {
        <i class="@Icon me-1"></i>
    }
    @Text
</button>

@code {
    [Parameter] public string Text { get; set; }
    [Parameter] public string Icon { get; set; }
    [Parameter] public string ColorClass { get; set; } = "btn-primary";
    [Parameter] public EventCallback OnClick { get; set; }
}
```

### 3. التعامل مع الترجمة

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
        .ToDictionary(kvp => kvp.Key, kvp => kvp.Value);
}
```

---

## ما يجب تجنبه (Anti-Patterns)

### ❌ 1. استخدام `@inject` لخدمات غير مسجلة في Blazor

```csharp
// ❌ خطأ - سيفشل المكون بصمت في MAUI
@inject HttpClient Http
@inject ApiService ApiService
@inject IMobileTranslationService MobileTranslation

// ✅ صحيح - استخدم Parameters
[Parameter] public IApiService? ApiService { get; set; }
```

### ❌ 2. إضافة RootComponents بشكل غير متزامن

```csharp
// ❌ خطأ - المكون لن يُحمّل
public MyFlowPage()
{
    InitializeComponent();
    _ = LoadAndShowAsync();  // async - لن يعمل!
}

private async Task LoadAndShowAsync()
{
    await LoadDataAsync();
    blazorWebView.RootComponents.Add(...);  // متأخر جداً
}

// ✅ صحيح - RootComponents.Add في الكونستركتور مباشرة
public MyFlowPage()
{
    InitializeComponent();
    LoadBlazorComponent();  // متزامن - يعمل!
    _ = RefreshFromApiAsync();  // غير متزامن - للتحديث
}
```

### ❌ 3. استدعاء Native APIs مباشرة من Blazor

```csharp
// ❌ خطأ - سيسبب استثناء
await SecureStorage.SetAsync("key", "value");

// ✅ صحيح - استخدم خدمة وسيطة تمرر كـ Parameter
[Parameter] public IStorageService? StorageService { get; set; }
await StorageService.SecureSetAsync("key", "value");
```

### ❌ 4. Hardcoding المسارات

```csharp
// ❌ خطأ
Navigation.NavigateTo("/psp/patient/5/details");

// ✅ صحيح - استخدم Constants
Navigation.NavigateTo($"{Routes.PspPatientDetails}/{patientId}");
```

---

## التكامل مع XAML

### فتح صفحة Blazor من XAML

```csharp
await Shell.Current.GoToAsync("blazorPageRoute");
```

### فتح صفحة XAML من Blazor

```csharp
// عبر EventCallback
[Parameter] public EventCallback OnNavigateToXaml { get; set; }
await OnNavigateToXaml.InvokeAsync();
```

---

## CHECKLIST: عند إنشاء صفحة BlazorWebView جديدة

- [ ] هل المكون موجود في `RubikCare.Shared.UI`؟
- [ ] هل `RootComponents.Add` يُستدعى **بشكل متزامن** في الكونستركتور؟
- [ ] هل الخدمات تُمرر كـ `[Parameter]` وليس `@inject` (باستثناء `ISharedTranslation*`)؟
- [ ] هل الصفحة تتعامل مع حالة "لا بيانات" (Skeleton Loader داخل المكون)؟
- [ ] هل `OnInitializedAsync` تُستدعى (تأكد من Debug Output)؟
- [ ] هل مفاتيح الترجمة مسجلة في قاعدة البيانات؟
- [ ] هل الصفحة متجاوبة مع أحجام الشاشات المختلفة؟

---

## 🔗 روابط ذات صلة

- [00 - الهيكل المعماري](00-architecture-overview.md)
- [05 - إنشاء الصفحات والمكونات](05-page-creation-checklist.md)
- [10 - دليل تطوير MAUI](10-maui-development-guide.md)
- [13 - Clean Architecture Enforcement](13-clean-architecture-enforcement.md)
- [15 - نظام ترجمة الموبايل](15-mobile-translation-system.md)

---

**آخر تحديث:** 13 يونيو 2026
**الملف:** `11-blazor-webview-guide.md`
