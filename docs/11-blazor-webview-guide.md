# 11 - دليل BlazorWebView والأنماط الناجحة

**آخر تحديث: 8 يوليو 2026**

---

## 📌 مقدمة

هذا المرجع يوثق **الأنماط الناجحة** وما يجب تجنبه عند استخدام `BlazorWebView` داخل تطبيق MAUI. الهدف هو مشاركة المكونات بين Web و Mobile مع تجنب المشاكل المعروفة، وتوحيد أسلوب العمل بين أعضاء الفريق.

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

أي محاولة لإضافة `RootComponents` بعد انتهاء الكونستركتور — سواء في `OnAppearing`، أو بعد `await`، أو في `Task` منفصل — **لن تُحمَّل في الـ DOM** وستبقى الصفحة فارغة.

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

// ❌ النمط 2: في OnAppearing
protected override void OnAppearing()
{
    base.OnAppearing();
    blazorWebView.RootComponents.Add(...); // ← خارج الكونستركتور
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

    // 1. بيانات محلية (متزامن)
    LoadCachedData();

    // 2. إضافة المكون (متزامن - إلزامي في الكونستركتور)
    LoadBlazorComponent();

    // 3. تحديث من API (غير متزامن - لا يعيد تحميل المكون)
    _ = RefreshFromApiAsync();
}
```

---

## 🧠 قاعدة ذهبية: لا تفترض، تحقق

**قبل كتابة أي كود يتفاعل مع `RootComponent` أو `BlazorWebView`**، تحقق من:

1. هل الخاصية/الطريقة موجودة في الـ IntelliSense؟
2. هل هذا النمط موثق في ملفات `11-blazor-webview-guide.md` أو `10-maui-development-guide.md`؟
3. هل هذا النمط يعمل في مشاريع MAUI الأخرى في الحل؟

**أمثلة على افتراضات فاشلة:**

| الافتراض | النتيجة |
|----------|---------|
| `RootComponent.Component` | ❌ غير موجود |
| `EventCallback.Factory.Create` في MAUI | ❌ لا يعمل كما في Blazor Server |
| `StateHasChanged()` من خارج الـ Component | ❌ غير قابل للوصول |

---

## 🚨 القسم الثالث: تحديث مكون Blazor من MAUI بعد التحميل

### ⚠️ المشكلة

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

**في مكون Blazor (`.razor`):**

```razor
@code {
    [Parameter] public Action<MyBlazorPage>? OnComponentReady { get; set; }

    protected override void OnAfterRender(bool firstRender)
    {
        if (firstRender)
        {
            // ⭐ أرسل مرجع this للحاوية
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

**في حاوية MAUI (`.xaml.cs`):**

```csharp
public partial class MyFlowPage : ContentPage
{
    private MyBlazorPage? _pageRef;  // ⭐ مرجع للمكون

    private void LoadBlazorComponent()
    {
        blazorWebView.RootComponents.Add(new RootComponent
        {
            ComponentType = typeof(MyBlazorPage),
            Parameters = new Dictionary<string, object?>
            {
                { "ApiService", (IApiService)_apiService },
                { "OnComponentReady", new Action<MyBlazorPage>(page =>
                    {
                        _pageRef = page;
                        Debug.WriteLine("✅ Component ref captured");
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
            _pageRef?.SetFileData(fileName, bytes);
        });
    }

    protected override void OnDisappearing()
    {
        base.OnDisappearing();
        _pageRef = null;  // ⭐ تنظيف المرجع
        try { blazorWebView.RootComponents.Clear(); } catch { }
    }
}
```

### 📊 مقارنة: Clear/Add vs OnComponentReady

| المعيار | Clear/Add | OnComponentReady |
|---------|-----------|------------------|
| إعادة تحميل المكون | ❌ لا يعمل | ✅ نعم |
| الحفاظ على State | ❌ يفقد كل شيء | ✅ يحافظ على الحالة |
| استدعاء API | ❌ لا حاجة (مكون جديد) | ✅ عبر public methods |
| التعقيد | بسيط لكن فاشل | بسيط وناجح |

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

```csharp
// في الكونستركتور مؤقتاً
blazorWebView.RootComponents.Add(new RootComponent
{
    Selector = "#app",
    ComponentType = typeof(TestPage)
});
```

| النتيجة | المشكلة |
|---------|---------|
| لم تظهر TestPage | مشكلة في WebView2 أو `index.html` أو تسجيل الـ Services |
| ظهرت TestPage | انتقل للخطوة 3 |

### الخطوة 3: عزّل مشكلة `@inject`

أضف `@inject` الخاصة بمكونك واحداً واحداً لـ TestPage حتى تجد المشكل:

```razor
@inject ISharedTranslationService TranslationService  @* ← هل تظهر؟ *@
@inject ISharedTranslationState TranslationState      @* ← هل تظهر؟ *@
@inject HttpClient Http                               @* ← على الأرجح هنا المشكلة *@
```

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

---

## 📁 هيكل المشروع مع BlazorWebView

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

## 📋 الأنماط الناجحة (يجب اتباعها)

### 1. النمط القياسي الكامل لصفحة BlazorWebView جديدة

```csharp
public partial class MyFlowPage : ContentPage
{
    private readonly ApiService _apiService;

    public MyFlowPage()
    {
        InitializeComponent();
        _apiService = IPlatformApplication.Current!.Services
            .GetRequiredService<ApiService>();
        LoadBlazorComponent();
    }

    private void LoadBlazorComponent()
    {
        var parameters = new Dictionary<string, object?>
        {
            { "ApiService", (IApiService)_apiService },
            { "OnCreateOrganizationAction", new Action<CreateOrgData>(async (data) =>
                {
                    await CreateOrganizationAsync(data);
                })
            }
        };

        blazorWebView.RootComponents.Clear();
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
@* ✅ مسموح - هذه الخدمات مسجلة في حاوية Blazor *@
@inject ISharedTranslationService TranslationService
@inject ISharedTranslationState TranslationState

@code {
    // ✅ صحيح - الخدمات غير المسجلة تُمرر كـ Parameters
    [Parameter] public IApiService? ApiService { get; set; }
    [Parameter] public Action<MyData>? OnAction { get; set; }

    protected override async Task OnInitializedAsync()
    {
        Debug.WriteLine("🔵 OnInitializedAsync START");
        if (ApiService != null) await LoadDataAsync();
    }
}
```

### 3. تمرير دوال (Actions/Callbacks) من MAUI إلى Blazor

في MAUI Hybrid، **`Action` يعمل بشكل موثوق**، بينما `EventCallback` قد يفشل في بعض السيناريوهات.

```csharp
// ✅ النمط الناجح في MAUI
var parameters = new Dictionary<string, object?>
{
    { "OnCreateOrganizationAction", new Action<CreateOrgData>(async (data) =>
        {
            await CreateOrganizationAsync(data);
        })
    }
};

// في Blazor Component
[Parameter] public Action<CreateOrgData>? OnCreateOrganizationAction { get; set; }
```

---

## ✅ CHECKLIST: عند إنشاء صفحة BlazorWebView جديدة

### MAUI Layer (صفحة الحاوية)

- [ ] هل `RootComponents.Add` يُستدعى **بشكل متزامن** في الكونستركتور؟
- [ ] هل `ApiService` يُمرر كـ `(IApiService)_apiService`؟
- [ ] هل تستخدم `Action` بدلاً من `EventCallback` لتمرير الدوال؟
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

**آخر تحديث:** 8 يوليو 2026 | **الملف:** `11-blazor-webview-guide.md`
```

---

## ✅ **ملخص التحديثات والتنسيق:**

1.  **إضافة قسم "قاعدة ذهبية"** بعد القسم الثاني مباشرة، لتأكيد مبدأ التحقق من الوثائق وعدم الافتراض.
2.  **تحديث قسم تمرير الدوال** ليصبح نمطاً مستقلاً وموثقاً، مع التأكيد على أن `Action` هو النمط الناجح في MAUI Hybrid.
3.  **تحسين التنسيق العام**:
    - استخدام `code blocks` مع تحديد اللغة (```csharp, ```razor, ```xml, ```markdown).
    - استخدام `Emojis` لتوضيح الحالات (✅ ❌ ⚠️).
    - تنظيم الجداول والنقاط لتكون أكثر وضوحاً.
    - إضافة `Checklist` محدثة بنقاط جديدة من التجارب العملية.
4.  **تحديث تاريخ الملف** إلى 8 يوليو 2026 ليعكس آخر التغييرات.
