**آخر تحديث: 21 مايو 2026** | **الأولوية: 🟡 مهم**

---

## 📌 مقدمة

هذا المرجع يوثق نظام الترجمة الكامل لتطبيق الموبايل، ويغطي **مسارين مختلفين**:

- **مسار XAML** — الصفحات التي تستخدم XAML وViewModels
- **مسار BlazorWebView** — الصفحات في `RubikCare.Shared.UI`

**المبدأ الأساسي:** نفس الـ API الموجود في الويب (`/api/localization/page/{domain}?lang={lang}`) يُستخدم في الموبايل. الفرق في **طريقة تخزين اللغة** وطريقة **إبلاغ الصفحات** بالتغيير.

| | الويب | الموبايل |
|---|---|---|
| **تخزين اللغة** | Browser Cookie / localStorage | `Preferences` (Native MAUI Storage) |
| **إبلاغ الصفحات** | SignalR / Blazor Circuit | C# Event (`OnLanguageChanged`) |
| **API الترجمة** | `/api/localization/page/...` | نفسه ✅ |
| **دالة T("key")** | في كل صفحة | نفسها ✅ |

---

## 🏗️ الجزء الأول: الأساس المشترك (يُبنى أولاً)

هذا الجزء **إلزامي** قبل أي عمل على XAML أو Blazor. كلا المسارين يعتمدان عليه.

---

### 1.1 Interfaces المشتركة (Shared.UI)

هذه الـ interfaces تسمح لـ `Shared.UI` باستخدام خدمات الترجمة دون الاعتماد على مشروع `Mobile` مباشرة.

**المسار:** `Shared.UI/Services/ITranslationService.cs`
```csharp
namespace RubikCare.Shared.UI.Services;

public interface ISharedTranslationService
{
    Task<Dictionary<string, string>> GetPageTranslationsAsync(string pageDomain, string? lang = null);
    string GetCurrentLanguage();
}
```

**المسار:** `Shared.UI/Services/ITranslationState.cs`
```csharp
namespace RubikCare.Shared.UI.Services;

public interface ISharedTranslationState
{
    string CurrentLanguage { get; }
    event Action? OnLanguageChanged;
    void SetLanguage(string lang);
}
```

---

### 1.2 MobileTranslationService

**المسار:** `Mobile/Services/MobileTranslationService.cs`

**المسؤولية:** يتواصل مع الـ API ويجلب الترجمات. يستخدم `MemoryCache` لتجنب الطلبات المتكررة.

```csharp
using Microsoft.Extensions.Caching.Memory;
using RubikCare.Mobile.Infrastructure.Services;
using RubikCare.Shared.UI.Services;

namespace RubikCare.Mobile.Services;

public interface IMobileTranslationService : ISharedTranslationService { }

public class MobileTranslationService : IMobileTranslationService
{
    private readonly ApiService _apiService;
    private readonly IMemoryCache _cache;
    private const string DefaultLang = "ar";
    private const string LangKey = "RubikCare:Language";

    public MobileTranslationService(ApiService apiService, IMemoryCache cache)
    {
        _apiService = apiService;
        _cache = cache;
    }

    public string GetCurrentLanguage()
    {
        return Preferences.Get(LangKey, DefaultLang);
    }

    public async Task<Dictionary<string, string>> GetPageTranslationsAsync(
        string pageDomain, string? lang = null)
    {
        var currentLang = lang ?? GetCurrentLanguage();
        var cacheKey = $"mobile_translations_{pageDomain}_{currentLang}";

        if (_cache.TryGetValue(cacheKey, out Dictionary<string, string>? cached) && cached != null)
            return cached;

        try
        {
            var result = await _apiService.GetAsync<Dictionary<string, string>>(
                $"/api/localization/page/{pageDomain}?lang={currentLang}");

            var translations = result ?? new Dictionary<string, string>();
            _cache.Set(cacheKey, translations, TimeSpan.FromMinutes(30));
            return translations;
        }
        catch (Exception ex)
        {
            Console.WriteLine($"[Translation] Failed to load {pageDomain}: {ex.Message}");
            return new Dictionary<string, string>();
        }
    }
}
```

---

### 1.3 MobileTranslationState

**المسار:** `Mobile/Services/MobileTranslationState.cs`

**المسؤولية:** يحفظ اللغة الحالية ويُبلغ **كل** الصفحات عند تغييرها عبر event.

```csharp
using RubikCare.Shared.UI.Services;

namespace RubikCare.Mobile.Services;

public class MobileTranslationState : ISharedTranslationState
{
    private const string LangKey = "RubikCare:Language";
    private const string DefaultLang = "ar";

    public event Action? OnLanguageChanged;

    public string CurrentLanguage => Preferences.Get(LangKey, DefaultLang);

    public void SetLanguage(string lang)
    {
        if (lang == CurrentLanguage) return;
        Preferences.Set(LangKey, lang);
        OnLanguageChanged?.Invoke();
    }
}
```

---

### 1.4 التسجيل في MauiProgram.cs

```csharp
// في MauiProgram.cs

// ⭐ ضروري للـ Cache في Translation
builder.Services.AddMemoryCache();

// ⭐ خدمات الترجمة للموبايل
builder.Services.AddSingleton<MobileTranslationState>();
builder.Services.AddSingleton<IMobileTranslationService, MobileTranslationService>();

// ⭐ سجّل الـ interfaces بعد ما الخدمات الفعلية تكون جاهزة
builder.Services.AddSingleton<ISharedTranslationState>(sp => sp.GetRequiredService<MobileTranslationState>());
builder.Services.AddSingleton<ISharedTranslationService>(sp => sp.GetRequiredService<IMobileTranslationService>());
```

**ملاحظة:** الخدمات `Singleton` لأن اللغة المختارة يجب أن تكون واحدة على مستوى التطبيق كله.

---

### 1.5 تسجيل ILocalizationService في Api.Web

```csharp
// في Api.Web\Program.cs — منطقة APPLICATION SERVICES
builder.Services.AddScoped<Rubikcare.Application.Interfaces.Localization.ILocalizationService, RubikCare.Application.Services.LocalizationService>();
```

**بدون هذا التسجيل، الـ API endpoint `/api/localization/page/{domain}` لن يعمل.**

---

## 📱 الجزء الثاني: مسار XAML

---

### 2.1 النمط القياسي لـ ViewModel

كل ViewModel في صفحة XAML تحتاج ترجمة تتبع هذا النمط:

```csharp
using RubikCare.Mobile.Services;

namespace RubikCare.Mobile.Features.Example.ViewModels;

public partial class ExampleViewModel : ObservableObject
{
    private readonly IMobileTranslationService _translationService;
    private readonly MobileTranslationState _translationState;
    private Dictionary<string, string> _translations = new();

    [ObservableProperty] private string _titleText = string.Empty;
    [ObservableProperty] private string _submitButtonText = string.Empty;

    public ExampleViewModel(
        IMobileTranslationService translationService,
        MobileTranslationState translationState)
    {
        _translationService = translationService;
        _translationState = translationState;

        // ⭐ الاشتراك في تغييرات اللغة
        _translationState.OnLanguageChanged += OnLanguageChangedHandler;
    }

    public async Task InitializeAsync()
    {
        await LoadTranslationsAsync();
    }

    private async Task LoadTranslationsAsync()
    {
        var lang = _translationState.CurrentLanguage;

        var pageTrans = await _translationService.GetPageTranslationsAsync("MOBILE.EXAMPLE", lang);
        var commonTrans = await _translationService.GetPageTranslationsAsync("COMMON", lang);

        _translations.Clear();
        foreach (var item in commonTrans) _translations[item.Key] = item.Value;
        foreach (var item in pageTrans) _translations[item.Key] = item.Value;

        ApplyTranslations();
    }

    private void ApplyTranslations()
    {
        TitleText = T("MOBILE.EXAMPLE.TITLE");
        SubmitButtonText = T("COMMON.SAVE");
    }

    private string T(string key) =>
        _translations.TryGetValue(key, out var value) ? value : key;

    private async void OnLanguageChangedHandler()
    {
        await LoadTranslationsAsync();
    }

    public void Dispose()
    {
        _translationState.OnLanguageChanged -= OnLanguageChangedHandler;
    }
}
```

---

### 2.2 الـ XAML Binding

```xml
<ContentPage xmlns="http://schemas.microsoft.com/dotnet/2021/maui"
             x:Class="RubikCare.Mobile.Features.Example.Views.ExamplePage">
    <VerticalStackLayout>
        <Label Text="{Binding TitleText}" />
        <Button Text="{Binding SubmitButtonText}" Command="{Binding SubmitCommand}" />
    </VerticalStackLayout>
</ContentPage>
```

---

### 2.3 Code-Behind للصفحة

```csharp
namespace RubikCare.Mobile.Features.Example.Views;

public partial class ExamplePage : ContentPage
{
    private readonly ExampleViewModel _viewModel;

    public ExamplePage(ExampleViewModel viewModel)
    {
        InitializeComponent();
        _viewModel = viewModel;
        BindingContext = _viewModel;
    }

    protected override async void OnAppearing()
    {
        base.OnAppearing();
        await _viewModel.InitializeAsync();
    }

    protected override void OnDisappearing()
    {
        base.OnDisappearing();
        _viewModel.Dispose();
    }
}
```

---

### 2.4 CHECKLIST لكل صفحة XAML جديدة

- [ ] ViewModel يحقن `IMobileTranslationService` و `MobileTranslationState`
- [ ] ViewModel يشترك في `OnLanguageChanged` في الـ constructor
- [ ] `LoadTranslationsAsync()` تُحمّل domain الصفحة + COMMON دفعة واحدة
- [ ] `ApplyTranslations()` تحدث كل Properties
- [ ] XAML يعمل Binding على Properties وليس نصوص ثابتة
- [ ] `OnAppearing` تستدعي `InitializeAsync()`
- [ ] `OnDisappearing` تستدعي `Dispose()` لإلغاء الاشتراك
- [ ] مفاتيح الترجمة موجودة في قاعدة البيانات مع `N''` prefix للعربية

---

## 🌐 الجزء الثالث: مسار BlazorWebView

---

### 3.1 النمط القياسي لصفحة BlazorWebView

```razor
@page "/example"
@namespace RubikCare.Shared.UI.Components.Example
@implements IDisposable

@inject ISharedTranslationService TranslationService
@inject ISharedTranslationState TranslationState

<div dir="@(_currentLang == "ar" ? "rtl" : "ltr")">
    <h1>@T("SHARED.EXAMPLE.TITLE")</h1>
    <button @onclick="ToggleLanguage">
        @(_currentLang == "ar" ? "English" : "العربية")
    </button>
</div>

@code {
    private string _currentLang = "ar";
    private Dictionary<string, string> _translations = new();
    private const string PageDomain = "SHARED.EXAMPLE";

    protected override async Task OnInitializedAsync()
    {
        _currentLang = TranslationState.CurrentLanguage;
        TranslationState.OnLanguageChanged += HandleLanguageChanged;
        await LoadTranslations();
    }

    private async Task LoadTranslations()
    {
        var lang = TranslationState.CurrentLanguage;
        var pageTrans = await TranslationService.GetPageTranslationsAsync(PageDomain, lang);
        var commonTrans = await TranslationService.GetPageTranslationsAsync("COMMON", lang);

        _translations = commonTrans.Concat(pageTrans).ToDictionary(k => k.Key, v => v.Value);
        _currentLang = lang;
    }

    private async void HandleLanguageChanged()
    {
        await LoadTranslations();
        await InvokeAsync(StateHasChanged);
    }

    private string T(string key) =>
        _translations.TryGetValue(key, out var value) ? value : key;

    private void ToggleLanguage()
    {
        var newLang = _currentLang == "ar" ? "en" : "ar";
        TranslationState.SetLanguage(newLang);
    }

    public void Dispose()
    {
        TranslationState.OnLanguageChanged -= HandleLanguageChanged;
    }
}
```

---

### 3.2 صفحات تم تطبيقها بالفعل

| الصفحة | المسار | Domain | الحالة |
|--------|--------|--------|--------|
| `SupportPage.razor` | `Shared.UI/Components/Support/` | `SHARED.SUPPORT` | ✅ مكتملة |
| `SettingsPage.razor` | `Shared.UI/Components/Patient/` | `SHARED.SETTINGS` | ✅ مكتملة (تبويب اللغة) |

---

### 3.3 صفحات جاهزة لتطبيق الترجمة

| الصفحة | المسار |
|--------|--------|
| `NotificationsPage.razor` | `Shared.UI/Components/Pages/` |
| `EditProfilePage.razor` | `Shared.UI/Components/Patient/` |
| `MedicationSchedulePage.razor` | `Shared.UI/Components/Patient/` |
| `MyProfilePage.razor` | `Shared.UI/Components/Patient/` |
| `PrivacyPolicy.razor` | `Shared.UI/Components/Legal/` |
| `TermsOfService.razor` | `Shared.UI/Components/Legal/` |
| `PspEntry.razor` | `Shared.UI/Components/PSP/Patient/` |
| `PspScheduleSetup.razor` | `Shared.UI/Components/PSP/Patient/` |
| `PharmacySearchPage.razor` | `Shared.UI/Components/PharmacySearch/` |
| `RepDashboard.razor` | `Shared.UI/Components/Rep/` |
| `InviteDoctor.razor` | `Shared.UI/Components/Rep/` |
| صفحات المراسلة (6) | `Shared.UI/Components/Messaging/` |
| صفحات إدارة المنظمة (3) | `Shared.UI/Components/OrganizationManagement/` |

---

### 3.4 CHECKLIST لكل صفحة BlazorWebView جديدة

- [ ] `@implements IDisposable` في أعلى الصفحة
- [ ] `@inject ISharedTranslationService TranslationService`
- [ ] `@inject ISharedTranslationState TranslationState`
- [ ] `PageDomain` بالبادئة الصحيحة (`SHARED.` للصفحات المشتركة)
- [ ] الاشتراك في `OnLanguageChanged` في `OnInitializedAsync`
- [ ] `LoadTranslations()` تحمّل domain الصفحة + COMMON دفعة واحدة
- [ ] `HandleLanguageChanged` تستدعي `InvokeAsync(StateHasChanged)`
- [ ] `T("key")` في كل النصوص — لا نصوص ثابتة
- [ ] `Dispose()` يلغي الاشتراك — **منع Memory Leak**
- [ ] مفاتيح الترجمة موجودة في قاعدة البيانات مع `N''` prefix

---

## 🏷️ الجزء الرابع: نظام تسمية مفاتيح الترجمة (Naming Convention)

| البادئة | المنصة | مثال |
|---------|--------|------|
| `SHARED.` | صفحات مشتركة (Web + Mobile BlazorWebView) | `SHARED.SUPPORT.TITLE` |
| `WEB.` | صفحات الويب فقط (Blazor Server) | `WEB.DASHBOARD.WELCOME` |
| `MOBILE.` | صفحات الموبايل فقط (XAML) | `MOBILE.LOGIN.EMAIL` |
| `COMMON` | نصوص عامة مشتركة بين الكل | `COMMON.SAVE` |

---

## 📊 الجزء الخامس: مرجع سريع للـ Domains

| الصفحة / المجال | Domain المقترح | النوع |
|---|---|---|
| الدعم الفني | `SHARED.SUPPORT` | BlazorWebView |
| الإعدادات | `SHARED.SETTINGS` | BlazorWebView |
| الإشعارات | `SHARED.NOTIFICATIONS` | BlazorWebView |
| الملف الشخصي | `SHARED.PROFILE` | BlazorWebView |
| تسجيل الدخول | `MOBILE.LOGIN` | XAML |
| التسجيل | `MOBILE.REGISTER` | XAML |
| نصوص مشتركة | `COMMON` | كلاهما |

---

## 🗄️ الجزء السادس: إضافة مفاتيح الترجمة في قاعدة البيانات

```sql
-- ⭐ لا تنسَ N'' prefix للعربية دائماً
-- الأعمدة الإجبارية: ResourceKey, ResourceValueAr, ResourceValueEn, Module, ResourceType, IsActive, CreatedDate

INSERT INTO Resources (ResourceKey, ResourceValueAr, ResourceValueEn, Module, ResourceType, IsActive, CreatedDate)
VALUES
    (N'SHARED.EXAMPLE.TITLE',  N'عنوان المثال',  N'Example Title',  N'SHARED', N'Title', 1, GETDATE()),
    (N'MOBILE.EXAMPLE.TITLE',  N'عنوان الجوال',  N'Mobile Title',   N'MOBILE', N'Title', 1, GETDATE()),
    (N'COMMON.SAVE',           N'حفظ',           N'Save',           N'COMMON', N'Button', 1, GETDATE());
```

**الأعمدة الإجبارية في جدول Resources:**
- `ResourceKey`, `ResourceValueAr`, `ResourceType`, `Module`, `IsActive`, `CreatedDate`

---

## ⚠️ الجزء السابع: تحذيرات ومحاذير

### 🔴 ممنوعات مطلقة

| # | الممنوع | البديل |
|---|---------|--------|
| 1 | نصوص Hardcoded في XAML أو Razor | استخدم `{Binding TitleText}` أو `@T("KEY")` |
| 2 | نسيان `Dispose()` في Blazor | Memory Leak يتراكم مع كل فتح صفحة |
| 3 | نسيان إلغاء الاشتراك في XAML ViewModel | نفس المشكلة |
| 4 | استدعاء الـ API في كل Render | حمّل مرة في `OnInitializedAsync` فقط |
| 5 | SQL بدون `N''` prefix للعربية | تظهر `????` في قاعدة البيانات |
| 6 | حقن `MobileTranslationService` مباشرة في Shared.UI | استخدم `ISharedTranslationService` |

### 🟡 تنبيهات مهمة

- `MobileTranslationState` هو `Singleton` — تغيير اللغة يؤثر على التطبيق كله فوراً
- الـ Cache مدته 30 دقيقة — تغيير الترجمات في DB يحتاج إعادة تشغيل التطبيق
- في XAML، تحديث الـ UI يتم عبر `[ObservableProperty]` تلقائياً
- في Blazor، تحديث الـ UI **يحتاج** `InvokeAsync(StateHasChanged)` صريحة
- لا تنسَ تسجيل `ILocalizationService` في `Api.Web\Program.cs`

---

## 📁 هيكل الملفات الكامل

```
📁 Mobile/Services/
├── MobileTranslationService.cs   (IMobileTranslationService + ISharedTranslationService)
└── MobileTranslationState.cs     (ISharedTranslationState)

📁 Shared.UI/Services/
├── ITranslationService.cs        (ISharedTranslationService)
└── ITranslationState.cs          (ISharedTranslationState)

📁 Shared.UI/Components/
├── Support/SupportPage.razor     ✅ مترجمة
└── Patient/SettingsPage.razor    ✅ مترجمة
```

---

## 🔗 روابط ذات صلة

- [01 - Program.cs والتسجيلات الأساسية](01-program-cs-foundation.md)
- [05 - إنشاء الصفحات والمكونات](05-page-creation-checklist.md)
- [11 - دليل BlazorWebView](11-blazor-webview-guide.md)
- [06 - منهجية حل المشاكل](06-troubleshooting-methodology.md)

---

**آخر تحديث:** 21 مايو 2026
**الملف:** `14-mobile-translation-system.md`
