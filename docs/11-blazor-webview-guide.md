
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

---

## 🚨 القسم الثالث: تحديث مكون Blazor من MAUI بعد التحميل

### ⚠️ المشكلة (اكتشاف: 13 يونيو 2026)

بعد إضافة المكون في الكونستركتور، لا يمكن إعادة تحميله عبر `RootComponents.Clear()` ثم `RootComponents.Add()`. لكن هناك حالات تحتاج فيها الحاوية لتحديث المكون من الخارج:

- **FilePicker:** المستخدم يختار ملفاً من MAUI Native UI
- **Push Notification:** تحديث البيانات عند وصول إشعار
- **Deep Link:** فتح الصفحة ببيانات من رابط خارجي

### ❌ أنماط فاشلة

```csharp
// ❌ لن يعمل - Clear ثم Add لا يعيد تحميل المكون
private async Task UpdateComponentAfterFilePick()
{
    var file = await PickFileAsync();
    blazorWebView.RootComponents.Clear();
    blazorWebView.RootComponents.Add(new RootComponent
    {
        Parameters = new Dictionary<string, object?> { { "File", file } }
    });
    // النتيجة: صفحة فارغة
}
```

### ✅ نمط OnComponentReady — الحل الوحيد

**الفكرة:** يحتفظ MAUI Host بمرجع مباشر لـ Blazor Component، ويستدعي `public methods` عليه مباشرة.

#### في مكون Blazor (`ProfessionalLicensePage.razor`):

```razor
@code {
    [Parameter] public Action<ProfessionalLicensePage>? OnComponentReady { get; set; }

    protected override void OnAfterRender(bool firstRender)
    {
        if (firstRender)
        {
            // ⭐ أرسل مرجع this للحاوية بمجرد أن يصبح المكون جاهزاً
            OnComponentReady?.Invoke(this);
        }
    }

    // ⭐ Public method تستدعى من الحاوية
    public void SetFileData(string fileName, byte[] fileBytes)
    {
        _fileName = fileName;
        _selectedFileBytes = fileBytes;
        InvokeAsync(StateHasChanged);
    }
}
```

#### في حاوية MAUI (`ProfessionalLicenseFlow.xaml.cs`):

```csharp
public partial class ProfessionalLicenseFlow : ContentPage
{
    private ProfessionalLicensePage? _licensePageRef;  // ⭐ مرجع للمكون

    private void LoadLicensePage()
    {
        blazorWebView.RootComponents.Add(new RootComponent
        {
            ComponentType = typeof(ProfessionalLicensePage),
            Parameters = new Dictionary<string, object?>
            {
                { "ApiService", (IApiService)_apiService },
                { "OnComponentReady", new Action<ProfessionalLicensePage>(page =>
                    {
                        _licensePageRef = page;  // ⭐ احفظ المرجع
                        System.Diagnostics.Debug.WriteLine("✅ Component ref captured");
                    })
                }
            }
        });
    }

    private async Task PickFileAndUpdateAsync()
    {
        var result = await FilePicker.PickAsync(...);
        if (result == null) return;

        var bytes = await ReadFileBytesAsync(result);
        var fileName = result.FileName;

        // ⭐ استدعِ public method مباشرة — لا Clear/Add!
        await MainThread.InvokeOnMainThreadAsync(() =>
        {
            _licensePageRef?.SetFileData(fileName, bytes);
        });
    }

    protected override void OnDisappearing()
    {
        base.OnDisappearing();
        _licensePageRef = null;  // ⭐ تنظيف المرجع
        try { blazorWebView.RootComponents.Clear(); } catch { }
    }
}
```

### 📊 مقارنة: Clear/Add vs OnComponentReady

| المعيار | Clear/Add | OnComponentReady |
|---------|-----------|------------------|
| إعادة تحميل المكون | ❌ لا يعمل | ✅ نعم |
| الحفاظ على State | ❌ يفقد كل شيء | ✅ يحافظ على الحالة |
| استدعاء API | ❌ لا حاجة (المكون جديد) | ✅ عبر public methods |
| تعقيد | بسيط لكن فاشل | بسيط وناجح |

### ⚠️ متى تستخدم هذا النمط

- ✅ FilePicker يحتاج لتحديث المكون بالملف المختار
- ✅ Push Notification يحتاج لتحديث بيانات معروضة
- ✅ أي سيناريو فيه MAUI native UI ثم العودة لـ Blazor

### ❌ متى لا تستخدمه

- ❌ جلب بيانات من API — دع المكون يحمّلها بنفسه في `OnInitializedAsync`
- ❌ تغيير حالة بسيطة — استخدم `EventCallback` العادي

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
│   └── ProfessionalOnboarding/Views/    # صفحات التوجيه المهني
│       ├── ProfessionalLicenseFlow.xaml
│       └── CreateOrganizationFlow.xaml
│
RubikCare.Shared.UI/                     # RCL للمكونات المشتركة
└── Components/
    ├── Patient/Settings/
    │   ├── SettingsPage.razor
    │   ├── ProfessionalStatusPage.razor
    │   └── ProfessionalLicensePage.razor
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
    private string _cachedValue = "";

    public MyFlowPage()
    {
        InitializeComponent();
        _apiService = IPlatformApplication.Current!.Services.GetRequiredService<ApiService>();
        LoadCachedData();
        LoadBlazorComponent();
        _ = RefreshFromApiAsync();
    }

    private void LoadBlazorComponent()
    {
        blazorWebView.RootComponents.Clear();

        var parameters = new Dictionary<string, object?>
        {
            { "ApiService", (IApiService)_apiService },
            { "OnAction", EventCallback.Factory.Create<MyDto>(this, HandleAction) }
        };

        blazorWebView.RootComponents.Add(new RootComponent
        {
            Selector = "#app",
            ComponentType = typeof(MyBlazorPage),
            Parameters = parameters
        });
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
@inject ISharedTranslationService TranslationService   @* ✅ مسموح *@
@inject ISharedTranslationState TranslationState       @* ✅ مسموح *@

@code {
    [Parameter] public IApiService? ApiService { get; set; }
    [Parameter] public EventCallback<MyDto> OnAction { get; set; }

    protected override async Task OnInitializedAsync()
    {
        System.Diagnostics.Debug.WriteLine("🔵 OnInitializedAsync START");
        if (ApiService != null) await LoadDataAsync();
    }
}
```

---

## CHECKLIST: عند إنشاء صفحة BlazorWebView جديدة

### MAUI Layer (صفحة الحاوية)
- [ ] هل `RootComponents.Add` يُستدعى **بشكل متزامن** في الكونستركتور؟
- [ ] هل `ApiService` يُمرر كـ `(IApiService)_apiService`؟
- [ ] هل تستخدم `OnComponentReady` لتحديث المكون من MAUI (بدلاً من Clear/Add)؟
- [ ] هل `OnDisappearing` تستدعي `RootComponents.Clear()` وتنظف `_pageRef`؟

### Blazor Component (المكون)
- [ ] هل المكون **لا يستخدم** `@inject HttpClient` أو `@inject ApiService`؟
- [ ] هل كل خدمة MAUI تُستخدم عبر `[Parameter]` وليس `@inject` (ما عدا الترجمة)؟
- [ ] هل `OnInitializedAsync` تُستدعى؟ (تأكد من Debug Output)
- [ ] هل المكون يعرض Skeleton/Loading أثناء جلب البيانات؟
- [ ] هل توجد `public methods` للتحديث من الحاوية عند الحاجة؟

---

## 🔗 روابط ذات صلة

- [00 - الهيكل المعماري](https://github.com/Rubikans-Egy/Rubikcare-docs/blob/main/docs/00-architecture-overview.md)
- [05 - إنشاء الصفحات والمكونات](https://github.com/Rubikans-Egy/Rubikcare-docs/blob/main/docs/05-page-creation-checklist.md)
- [10 - دليل تطوير MAUI](https://github.com/Rubikans-Egy/Rubikcare-docs/blob/main/docs/10-maui-development-guide.md)
- [13 - Clean Architecture Enforcement](https://github.com/Rubikans-Egy/Rubikcare-docs/blob/main/docs/13-clean-architecture-enforcement.md)
- [15 - نظام ترجمة الموبايل](https://github.com/Rubikans-Egy/Rubikcare-docs/blob/main/docs/15-mobile-translation-system.md)

---

**آخر تحديث:** 13 يونيو 2026 | **الملف:** `11-blazor-webview-guide.md`
```

---

## 📋 **ملخص التحديثات الجديدة:**

| القسم | الحالة |
|--------|--------|
| القسم الأول: مشكلة DI | ✅ موجود |
| القسم الثاني: قيود RootComponents | ✅ موجود |
| **القسم الثالث: OnComponentReady** | ✅ **جديد** |
| جدول مقارنة Clear/Add vs OnComponentReady | ✅ **جديد** |
| تشخيص الصفحة الفارغة (4 خطوات) | ✅ موجود |
| CHECKLIST محدثة | ✅ **محدثة** |

