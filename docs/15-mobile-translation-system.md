
# 📱 Mobile Translation System (نظام ترجمة الموبايل)

**آخر تحديث: 14 يونيو 2026** | **الأولوية: 🟡 مهم**

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

### 1.2 MobileTranslationService (⭐ محدث)

**المسار:** `Mobile/Services/MobileTranslationService.cs`

**المسؤولية:** يتواصل مع الـ API ويجلب الترجمات. يستخدم `MemoryCache` و `Preferences` لتجنب الطلبات المتكررة.

**⚠️ تحديث مهم (14 يونيو 2026):** تم تعديل الدالة لتستدعي الـ API مباشرة.

```csharp
using Microsoft.Extensions.Caching.Memory;
using RubikCare.Mobile.Infrastructure.Services;
using RubikCare.Shared.UI.Services;

namespace RubikCare.Mobile.Services;

public interface IMobileTranslationService : ISharedTranslationService
{
    Task<Dictionary<string, string>> GetAllTranslationsAsync(string? lang = null);
    Task PreloadCommonTranslationsAsync();
}

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

    /// <summary>
    /// ترجمة صفحة معينة - الإصدار النهائي
    /// </summary>
    public async Task<Dictionary<string, string>> GetPageTranslationsAsync(
        string pageDomain, string? lang = null)
    {
        var currentLang = lang ?? GetCurrentLanguage();
        var cacheKey = $"mobile_translations_{pageDomain}_{currentLang}";

        // Memory Cache
        if (_cache.TryGetValue(cacheKey, out Dictionary<string, string>? cached) && cached != null)
            return cached;

        // ⭐ مباشرة من API (تجاوز Preferences مؤقتاً)
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
            System.Diagnostics.Debug.WriteLine($"[Translation] Failed to load {pageDomain}: {ex.Message}");
            return new Dictionary<string, string>();
        }
    }

    /// <summary>
    /// تحميل كل الترجمات مرة واحدة من السيرفر
    /// </summary>
    public async Task<Dictionary<string, string>> GetAllTranslationsAsync(string? lang = null)
    {
        var currentLang = lang ?? GetCurrentLanguage();
        var cacheKey = $"all_translations_{currentLang}";

        if (_cache.TryGetValue(cacheKey, out Dictionary<string, string>? cached) && cached != null)
            return cached;

        try
        {
            var result = await _apiService.GetAsync<Dictionary<string, string>>(
                $"/api/localization/all?lang={currentLang}");

            var translations = result ?? new Dictionary<string, string>();

            if (translations.Any())
            {
                _cache.Set(cacheKey, translations, TimeSpan.FromHours(1));
            }

            return translations;
        }
        catch (Exception ex)
        {
            Console.WriteLine($"[Translation] Failed to load all translations: {ex.Message}");
            return new Dictionary<string, string>();
        }
    }

    public async Task PreloadCommonTranslationsAsync()
    {
        try
        {
            var currentLang = GetCurrentLanguage();
            Console.WriteLine($"🔄 Preloading translations for: {currentLang}");

            var allTranslations = await GetAllTranslationsAsync(currentLang);

            Console.WriteLine($"✅ Preloaded {allTranslations.Count} translations for {currentLang}");
        }
        catch (Exception ex)
        {
            Console.WriteLine($"❌ PreloadCommonTranslationsAsync error: {ex.Message}");
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

### 1.4 TranslationCacheService (⭐ جديد)

**المسار:** `Mobile/Services/TranslationCacheService.cs`

**المسؤولية:** يدير تخزين الترجمات في `Preferences` و `MemoryCache` لتحسين الأداء وتقليل طلبات API.

```csharp
using System.Text.Json;
using Microsoft.Extensions.Caching.Memory;

namespace RubikCare.Mobile.Services;

public class TranslationCacheService
{
    private readonly IMemoryCache _memoryCache;
    private const string PrefsPrefix = "translations_all_";

    public TranslationCacheService(IMemoryCache memoryCache)
    {
        _memoryCache = memoryCache;
    }

    public async Task<Dictionary<string, string>?> GetCachedAsync(string lang)
    {
        var memKey = $"all_translations_{lang}";
        if (_memoryCache.TryGetValue(memKey, out Dictionary<string, string>? cached) && cached != null)
            return cached;

        var prefsKey = $"{PrefsPrefix}{lang}";
        var json = Preferences.Get(prefsKey, "");
        if (!string.IsNullOrEmpty(json))
        {
            try
            {
                var translations = JsonSerializer.Deserialize<Dictionary<string, string>>(json);
                if (translations != null && translations.Any())
                {
                    _memoryCache.Set(memKey, translations, TimeSpan.FromMinutes(30));
                    return translations;
                }
            }
            catch { }
        }

        return null;
    }

    public void SetCache(string lang, Dictionary<string, string> translations)
    {
        if (translations == null || !translations.Any()) return;

        _memoryCache.Set($"all_translations_{lang}", translations, TimeSpan.FromHours(1));

        try
        {
            var json = JsonSerializer.Serialize(translations);
            Preferences.Set($"{PrefsPrefix}{lang}", json);
            System.Diagnostics.Debug.WriteLine($"✅ Saved {translations.Count} translations to Preferences for {lang}");
        }
        catch (Exception ex)
        {
            System.Diagnostics.Debug.WriteLine($"❌ TranslationCache Set error: {ex.Message}");
        }
    }

    public void ClearCache(string lang)
    {
        _memoryCache.Remove($"all_translations_{lang}");
        Preferences.Remove($"{PrefsPrefix}{lang}");
    }
}
```

---

### 1.5 التسجيل في MauiProgram.cs

```csharp
// في MauiProgram.cs

// ⭐ ضروري للـ Cache في Translation
builder.Services.AddMemoryCache();

// ⭐ خدمات الترجمة للموبايل
builder.Services.AddSingleton<MobileTranslationState>();
builder.Services.AddSingleton<IMobileTranslationService, MobileTranslationService>();
builder.Services.AddSingleton<TranslationCacheService>();

// ⭐ سجّل الـ interfaces بعد ما الخدمات الفعلية تكون جاهزة
builder.Services.AddSingleton<ISharedTranslationState>(sp => sp.GetRequiredService<MobileTranslationState>());
builder.Services.AddSingleton<ISharedTranslationService>(sp => sp.GetRequiredService<IMobileTranslationService>());
```

**ملاحظة:** الخدمات `Singleton` لأن اللغة المختارة يجب أن تكون واحدة على مستوى التطبيق كله.

---

### 1.6 تسجيل ILocalizationService في Api.Web

```csharp
// في Api.Web\Program.cs — منطقة APPLICATION SERVICES
builder.Services.AddScoped<Rubikcare.Application.Interfaces.Localization.ILocalizationService, RubikCare.Application.Services.LocalizationService>();
```

**بدون هذا التسجيل، الـ API endpoint `/api/localization/page/{domain}` لن يعمل.**

---

## 📱 الجزء الثاني: مسار XAML

---

### 2.1 النمط القياسي لـ ViewModel (مع Dependency Injection)

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

### 2.2 النمط البديل لـ ViewModel (بدون Dependency Injection)

بعض ViewModels (مثل `PspSearchViewModel`) لا تستخدم DI مباشرة، وتحتاج نمطاً مختلفاً:

```csharp
using RubikCare.Mobile.Services;
using RubikCare.Shared.UI.Services;

namespace RubikCare.Mobile.Features.PSP.Doctor.ViewModels;

public class PspSearchViewModel : INotifyPropertyChanged, IDisposable
{
    private readonly IMobileTranslationService _translationService;
    private readonly MobileTranslationState _translationState;
    private Dictionary<string, string> _translations = new();
    private const string PageDomain = "SHARED.PSP_SEARCH";

    public PspSearchViewModel()
    {
        // ⭐ الحصول على الخدمات يدوياً من DI container
        _translationService = IPlatformApplication.Current?.Services?.GetRequiredService<IMobileTranslationService>()
            ?? throw new InvalidOperationException("IMobileTranslationService not registered");
        _translationState = IPlatformApplication.Current?.Services?.GetRequiredService<MobileTranslationState>()
            ?? throw new InvalidOperationException("MobileTranslationState not registered");

        _translationState.OnLanguageChanged += HandleLanguageChanged;
        LoadTranslationsAndApply();
    }

    // ⭐ Properties يدوية مع INotifyPropertyChanged
    private string _pageTitle = "برامج الدعم";
    public string PageTitle
    {
        get => _pageTitle;
        set { _pageTitle = value; OnPropertyChanged(); }
    }

    private async void LoadTranslationsAndApply()
    {
        await LoadTranslationsAsync();
        ApplyTranslations();
    }

    private async Task LoadTranslationsAsync()
    {
        var pageTrans = await _translationService.GetPageTranslationsAsync(PageDomain);
        var commonTrans = await _translationService.GetPageTranslationsAsync("COMMON");
        _translations = commonTrans.Concat(pageTrans).ToDictionary(k => k.Key, v => v.Value);
    }

    private void ApplyTranslations()
    {
        PageTitle = T("SHARED.PSP_SEARCH.TITLE");
        // ... باقي الخصائص
    }

    private string T(string key) => _translations.GetValueOrDefault(key, key);

    private async void HandleLanguageChanged()
    {
        await LoadTranslationsAsync();
        ApplyTranslations();
    }

    // ⭐ INotifyPropertyChanged implementation
    public event PropertyChangedEventHandler? PropertyChanged;
    protected void OnPropertyChanged([CallerMemberName] string? propertyName = null)
    {
        PropertyChanged?.Invoke(this, new PropertyChangedEventArgs(propertyName));
    }

    public void Dispose() => _translationState.OnLanguageChanged -= HandleLanguageChanged;
}
```

---

### 2.3 الـ XAML Binding

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

### 2.4 Code-Behind للصفحة

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

### 2.5 CHECKLIST لكل صفحة XAML جديدة

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

### 3.2 صفحات تم تطبيقها بالفعل (⭐ محدث)

| الصفحة | المسار | Domain | الحالة |
|--------|--------|--------|--------|
| `SupportPage.razor` | `Shared.UI/Components/Support/` | `SHARED.SUPPORT` | ✅ مكتملة |
| `SettingsPage.razor` | `Shared.UI/Components/Patient/` | `SHARED.SETTINGS_V2` | ✅ مكتملة |
| `RepDashboard.razor` | `Shared.UI/Components/Rep/` | `SHARED.REP_DASHBOARD` | ✅ مكتملة |
| `PharmaCompanyDashboard.razor` | `Shared.UI/Components/Rep/` | `SHARED.PHARMA_DASHBOARD` | ✅ مكتملة |
| `MyNetwork.razor` | `Shared.UI/Components/Rep/` | `SHARED.MY_NETWORK` | ✅ مكتملة |
| `MyInvitations.razor` | `Shared.UI/Components/Rep/` | `SHARED.MY_INVITATIONS` | ✅ مكتملة |
| `InviteDoctor.razor` | `Shared.UI/Components/Rep/` | `SHARED.INVITE_DOCTOR` | ✅ مكتملة |

---

### 3.3 CHECKLIST لكل صفحة BlazorWebView جديدة (⭐ محدث)

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
- [ ] **⚠️ استخدام علامات التنصيص المفردة في `@onclick` مع دوال تأخذ معامل**

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
| الإعدادات | `SHARED.SETTINGS_V2` | BlazorWebView |
| الإشعارات | `SHARED.NOTIFICATIONS` | BlazorWebView |
| الملف الشخصي | `SHARED.PROFILE` | BlazorWebView |
| لوحة المندوب | `SHARED.REP_DASHBOARD` | BlazorWebView |
| شبكتي (مندوب) | `SHARED.MY_NETWORK` | BlazorWebView |
| سجل الدعوات (مندوب) | `SHARED.MY_INVITATIONS` | BlazorWebView |
| دعوة طبيب/صيدلي | `SHARED.INVITE_DOCTOR` | BlazorWebView |
| بحث البرامج | `SHARED.PSP_SEARCH` | XAML |
| تسجيل الدخول | `MOBILE.LOGIN` | XAML |
| التسجيل | `MOBILE.REGISTER` | XAML |
| نصوص مشتركة | `COMMON` | كلاهما |

---

## 🗄️ الجزء السادس: إضافة مفاتيح الترجمة في قاعدة البيانات

### 6.1 إضافة مفاتيح جديدة (INSERT)

```sql
-- ⭐ لا تنسَ N'' prefix للعربية دائماً
-- الأعمدة الإجبارية: ResourceKey, ResourceValueAr, ResourceValueEn, Module, ResourceType, IsActive, CreatedDate

INSERT INTO Resources (ResourceKey, ResourceValueAr, ResourceValueEn, Module, ResourceType, IsActive, CreatedDate)
VALUES
    (N'SHARED.EXAMPLE.TITLE',  N'عنوان المثال',  N'Example Title',  N'SHARED', N'Title', 1, GETDATE()),
    (N'MOBILE.EXAMPLE.TITLE',  N'عنوان الجوال',  N'Mobile Title',   N'MOBILE', N'Title', 1, GETDATE()),
    (N'COMMON.SAVE',           N'حفظ',           N'Save',           N'COMMON', N'Button', 1, GETDATE());
```

### 6.2 تحديث مفاتيح موجودة (UPDATE)

```sql
-- ⚠️ استخدم UPDATE لو المفاتيح موجودة بالفعل

UPDATE Resources 
SET ResourceValueAr = N'📊 لوحة المندوب', 
    ResourceValueEn = N'📊 Rep Dashboard'
WHERE ResourceKey = N'SHARED.REP_DASHBOARD.TITLE';
```

### 6.3 إدراج أو تحديث آمن (MERGE)

```sql
-- ⭐ أفضل طريقة: MERGE يضيف لو مش موجود، يعدل لو موجود

MERGE Resources AS target
USING (VALUES
    (N'SHARED.REP_DASHBOARD.TITLE', N'📊 لوحة المندوب', N'📊 Rep Dashboard', N'SHARED', N'Title'),
    (N'SHARED.REP_DASHBOARD.LOADING', N'جاري التحميل...', N'Loading...', N'SHARED', N'Text')
) AS source (ResourceKey, ResourceValueAr, ResourceValueEn, Module, ResourceType)
ON target.ResourceKey = source.ResourceKey
WHEN MATCHED THEN
    UPDATE SET 
        ResourceValueAr = source.ResourceValueAr,
        ResourceValueEn = source.ResourceValueEn
WHEN NOT MATCHED THEN
    INSERT (ResourceKey, ResourceValueAr, ResourceValueEn, Module, ResourceType, IsActive, CreatedDate)
    VALUES (source.ResourceKey, source.ResourceValueAr, source.ResourceValueEn, source.Module, source.ResourceType, 1, GETDATE());
```

### 6.4 التحقق من المفاتيح

```sql
SELECT ResourceKey, ResourceValueAr, ResourceValueEn, Module, ResourceType
FROM Resources 
WHERE ResourceKey LIKE 'SHARED.REP_DASHBOARD%'
ORDER BY ResourceKey;
```

### 6.5 الأعمدة الإجبارية في جدول Resources

| العمود | النوع | ملاحظات |
|--------|------|---------|
| `ResourceKey` | `nvarchar` | PRIMARY KEY - فريد لكل ترجمة |
| `ResourceValueAr` | `nvarchar` | القيمة العربية (**استخدم N'' prefix دائماً**) |
| `ResourceValueEn` | `nvarchar` | القيمة الإنجليزية |
| `Module` | `nvarchar` | `SHARED` / `MOBILE` / `WEB` |
| `ResourceType` | `nvarchar` | `Title` / `Text` / `Button` / `Label` / `Placeholder` |
| `IsActive` | `bit` | `1` للتفعيل |
| `CreatedDate` | `datetime` | `GETDATE()` |

---

## 📊 الجزء السابع: مقارنة سريعة بين XAML و BlazorWebView

| العنصر | XAML | BlazorWebView |
|--------|------|---------------|
| **الـ Service** | `IMobileTranslationService` | `ISharedTranslationService` |
| **الـ State** | `MobileTranslationState` | `ISharedTranslationState` |
| **الحقن** | Constructor Injection أو `GetRequiredService` | `@inject` في Razor |
| **تحميل الترجمات** | `LoadTranslationsAsync()` + `ApplyTranslations()` | `LoadTranslations()` + `StateHasChanged()` |
| **تحديث الـ UI** | تلقائي عبر `[ObservableProperty]` | يدوي عبر `InvokeAsync(StateHasChanged)` |
| **RTL/LTR** | تلقائي مع `I18nManager` | يدوي عبر `dir` attribute أو CSS class |
| **تنظيف الموارد** | `Dispose()` في ViewModel + `OnDisappearing` | `Dispose()` في `@code` |

---

## ⚠️ الجزء الثامن: تحذيرات ومحاذير

### 🔴 ممنوعات مطلقة

| # | الممنوع | البديل |
|---|---------|--------|
| 1 | نصوص Hardcoded في XAML أو Razor | استخدم `{Binding TitleText}` أو `@T("KEY")` |
| 2 | نسيان `Dispose()` في Blazor | Memory Leak يتراكم مع كل فتح صفحة |
| 3 | نسيان إلغاء الاشتراك في XAML ViewModel | نفس المشكلة |
| 4 | استدعاء الـ API في كل Render | حمّل مرة في `OnInitializedAsync` فقط |
| 5 | SQL بدون `N''` prefix للعربية | تظهر `????` في قاعدة البيانات |
| 6 | حقن `MobileTranslationService` مباشرة في Shared.UI | استخدم `ISharedTranslationService` |
| 7 | استخدام INSERT عندما المفتاح موجود مسبقاً | استخدم UPDATE أو MERGE |

### 🟡 تنبيهات مهمة

- `MobileTranslationState` هو `Singleton` — تغيير اللغة يؤثر على التطبيق كله فوراً
- الـ Cache مدته 30 دقيقة — تغيير الترجمات في DB يحتاج إعادة تشغيل التطبيق
- في XAML، تحديث الـ UI يتم عبر `[ObservableProperty]` تلقائياً
- في Blazor، تحديث الـ UI **يحتاج** `InvokeAsync(StateHasChanged)` صريحة
- لا تنسَ تسجيل `ILocalizationService` في `Api.Web\Program.cs`
- ViewModels التي لا تستخدم DI تحتاج `GetRequiredService` يدوياً

---

## 🟠 الجزء التاسع: المشاكل المعروفة والحلول (Known Issues)

### 9.1 بعض الصفحات تظهر مفاتيح الترجمة بدلاً من النصوص

**السبب:** `GetPageTranslationsAsync` كان يبحث في `Preferences` عن المفاتيح، ولكن `Preferences` لم تكن محملة مسبقاً.

**الحل:** 
- الخيار 1: استدعاء `PreloadCommonTranslationsAsync()` عند بدء التطبيق
- الخيار 2: تعديل `GetPageTranslationsAsync` لاستدعاء API مباشرة (تم التنفيذ)

### 9.2 ترجمة جزئية (بعض الأزرار مترجمة وبعضها لا)

**السبب:** بعض المفاتيح موجودة في `COMMON` domain وليس في domain الصفحة.

**الحل:** دائماً قم بتحميل `COMMON` مع domain الصفحة:
```csharp
var pageTrans = await TranslationService.GetPageTranslationsAsync(PageDomain, lang);
var commonTrans = await TranslationService.GetPageTranslationsAsync("COMMON", lang);
_translations = commonTrans.Concat(pageTrans).ToDictionary(k => k.Key, v => v.Value);
```

### 9.3 `@onclick` مع دالة تأخذ معامل لا يعمل في Blazor

**السبب:** تداخل علامات التنصيص المزدوجة.

**الحل:** استخدم علامات التنصيص **المفردة**:
```razor
// ❌ لا يعمل
<button @onclick="() => MyFunction("value")">نص</button>

// ✅ يعمل
<button @onclick='() => MyFunction("value")'>نص</button>
```

### 9.4 تغيير اللغة لا يؤثر على الصفحة الحالية

**السبب:** نسيان استدعاء `InvokeAsync(StateHasChanged)` في `HandleLanguageChanged`.

**الحل:**
```csharp
private async void HandleLanguageChanged()
{
    await LoadTranslations();
    await InvokeAsync(StateHasChanged);  // ⭐ مهم جداً
}
```

### 9.5 تسرب الذاكرة (Memory Leak) عند التنقل بين الصفحات

**السبب:** نسيان إلغاء الاشتراك من `OnLanguageChanged` في `Dispose()`.

**الحل:**
```csharp
public void Dispose()
{
    TranslationState.OnLanguageChanged -= HandleLanguageChanged;
}
```

---

## 📁 هيكل الملفات الكامل

```
📁 Mobile/Services/
├── MobileTranslationService.cs
├── MobileTranslationState.cs
└── TranslationCacheService.cs          (⭐ جديد)

📁 Shared.UI/Services/
├── ITranslationService.cs
└── ITranslationState.cs

📁 Shared.UI/Components/
├── Support/SupportPage.razor           ✅ مترجمة
├── Patient/SettingsPage.razor          ✅ مترجمة
├── Rep/RepDashboard.razor              ✅ مترجمة
├── Rep/PharmaCompanyDashboard.razor    ✅ مترجمة
├── Rep/MyNetwork.razor                 ✅ مترجمة
├── Rep/MyInvitations.razor             ✅ مترجمة
└── Rep/InviteDoctor.razor              ✅ مترجمة
```

---

## 🔗 روابط ذات صلة

- [00 - Architecture Overview](00-architecture-overview.md)
- [01 - Program.cs والتسجيلات الأساسية](01-program-cs-foundation.md)
- [05 - إنشاء الصفحات والمكونات](05-page-creation-checklist.md)
- [11 - دليل BlazorWebView](11-blazor-webview-guide.md)
- [13 - Clean Architecture Enforcement](13-clean-architecture-enforcement.md)
- [06 - منهجية حل المشاكل](06-troubleshooting-methodology.md)

---

**آخر تحديث:** 14 يونيو 2026
**الملف:** `15-mobile-translation-system.md`
``
