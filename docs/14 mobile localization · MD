# 14 - نظام الترجمة في تطبيق الموبايل (XAML + BlazorWebView)

**آخر تحديث: 20 مايو 2026** | **الأولوية: 🟡 مهم**

---

## مقدمة

هذا المرجع يوثق نظام الترجمة الكامل لتطبيق الموبايل، ويغطي **مسارين مختلفين**:

- **مسار XAML** — الصفحات القديمة التي تستخدم XAML وViewModels
- **مسار BlazorWebView** — الصفحات الجديدة في `RubikCare.Shared.UI`

**المبدأ الأساسي:** نفس الـ API الموجود في الويب (`/api/localization/page/{domain}?lang={lang}`) يُستخدم في الموبايل بالظبط. الفرق الوحيد في **طريقة تخزين اللغة** وطريقة **إبلاغ الصفحات** بالتغيير.

| | الويب | الموبايل |
|---|---|---|
| **تخزين اللغة** | Browser Cookie / localStorage | `Preferences` (Native MAUI Storage) |
| **إبلاغ الصفحات** | SignalR / Blazor Circuit | C# Event (`OnLanguageChanged`) |
| **API الترجمة** | `/api/localization/page/...` | نفسه ✅ |
| **دالة T("key")** | في كل صفحة | نفسها ✅ |

---

## الجزء الأول: الأساس المشترك (يُبنى أولاً)

هذا الجزء **إلزامي** قبل أي عمل على XAML أو Blazor. كلا المسارين يعتمدان عليه.

---

### 1.1 MobileTranslationService

**المسار:** `RubikCare.Mobile/Services/MobileTranslationService.cs`

**المسؤولية:** يتواصل مع الـ API ويجلب الترجمات. يستخدم `MemoryCache` لتجنب الطلبات المتكررة.

```csharp
using Microsoft.Extensions.Caching.Memory;
using RubikCare.Mobile.Services; // ApiService

namespace RubikCare.Mobile.Services;

public interface IMobileTranslationService
{
    Task<Dictionary<string, string>> GetPageTranslationsAsync(string pageDomain, string? lang = null);
    string GetCurrentLanguage();
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

    public async Task<Dictionary<string, string>> GetPageTranslationsAsync(
        string pageDomain, string? lang = null)
    {
        var currentLang = lang ?? GetCurrentLanguage();
        var cacheKey = $"mobile_translations_{pageDomain}_{currentLang}";

        if (_cache.TryGetValue(cacheKey, out Dictionary<string, string>? cached) && cached != null)
            return cached;

        try
        {
            // ⭐ نفس الـ API endpoint الموجود في الويب
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

### 1.2 MobileTranslationState

**المسار:** `RubikCare.Mobile/Services/MobileTranslationState.cs`

**المسؤولية:** يحفظ اللغة الحالية ويُبلغ **كل** الصفحات عند تغييرها عبر event.

```csharp
namespace RubikCare.Mobile.Services;

public class MobileTranslationState
{
    private const string LangKey = "RubikCare:Language";
    private const string DefaultLang = "ar";

    // ⭐ Event يُطلق عند كل تغيير في اللغة — الصفحات تشترك فيه
    public event Action? OnLanguageChanged;

    public string CurrentLanguage => Preferences.Get(LangKey, DefaultLang);

    public void SetLanguage(string lang)
    {
        if (lang == CurrentLanguage) return;

        Preferences.Set(LangKey, lang);

        // ⭐ إبلاغ كل المشتركين
        OnLanguageChanged?.Invoke();
    }

    public bool IsArabic => CurrentLanguage == "ar";
}
```

---

### 1.3 التسجيل في MauiProgram.cs

أضف السطرين التاليين في **منطقة خدمات النظام**، بعد تسجيل `ApiService` مباشرةً:

```csharp
// في MauiProgram.cs — بعد: builder.Services.AddSingleton<ApiService>();

builder.Services.AddMemoryCache(); // ⭐ ضروري للـ Cache في Translation

// ⭐ خدمات الترجمة للموبايل
builder.Services.AddSingleton<MobileTranslationState>();
builder.Services.AddSingleton<IMobileTranslationService, MobileTranslationService>();
```

**ملاحظة:** كلاهما `Singleton` لأن اللغة المختارة يجب أن تكون واحدة على مستوى التطبيق كله.

---

## الجزء الثاني: مسار XAML

---

### 2.1 النمط القياسي لـ ViewModel

كل ViewModel في صفحة XAML تحتاج ترجمة تتبع هذا النمط:

```csharp
// مثال: LoginViewModel.cs
using RubikCare.Mobile.Services;

namespace RubikCare.Mobile.Features.Auth.ViewModels;

public partial class LoginViewModel : ObservableObject
{
    private readonly IMobileTranslationService _translationService;
    private readonly MobileTranslationState _translationState;
    private Dictionary<string, string> _translations = new();

    // ⭐ Properties مترجمة — XAML يعمل Bind عليها
    [ObservableProperty] private string _titleText = string.Empty;
    [ObservableProperty] private string _emailPlaceholder = string.Empty;
    [ObservableProperty] private string _loginButtonText = string.Empty;
    // ... باقي النصوص

    public LoginViewModel(
        IMobileTranslationService translationService,
        MobileTranslationState translationState)
    {
        _translationService = translationService;
        _translationState = translationState;

        // ⭐ الاشتراك في تغييرات اللغة
        _translationState.OnLanguageChanged += OnLanguageChangedHandler;
    }

    // ⭐ يُستدعى عند فتح الصفحة
    public async Task InitializeAsync()
    {
        await LoadTranslationsAsync();
    }

    private async Task LoadTranslationsAsync()
    {
        var lang = _translationState.CurrentLanguage;

        // ⭐ تحميل domain الصفحة + COMMON دفعة واحدة
        var pageTrans = await _translationService.GetPageTranslationsAsync("LOGIN", lang);
        var commonTrans = await _translationService.GetPageTranslationsAsync("COMMON", lang);

        _translations.Clear();
        foreach (var item in commonTrans) _translations[item.Key] = item.Value;
        foreach (var item in pageTrans) _translations[item.Key] = item.Value;

        ApplyTranslations();
    }

    private void ApplyTranslations()
    {
        // ⭐ تطبيق الترجمات على Properties — XAML Binding يتحدث تلقائياً
        TitleText = T("LOGIN.TITLE");
        EmailPlaceholder = T("LOGIN.EMAIL_PLACEHOLDER");
        LoginButtonText = T("COMMON.LOGIN");
        // ... باقي النصوص
    }

    // ⭐ دالة T() المساعدة — نفس فكرة الويب
    private string T(string key) =>
        _translations.TryGetValue(key, out var value) ? value : key;

    private async void OnLanguageChangedHandler()
    {
        await LoadTranslationsAsync();
    }

    // ⭐ إلزامي: إلغاء الاشتراك عند التخلص من ViewModel
    public void Dispose()
    {
        _translationState.OnLanguageChanged -= OnLanguageChangedHandler;
    }
}
```

---

### 2.2 الـ XAML Binding

في ملف `.xaml` استخدم Binding عادي على Properties الـ ViewModel:

```xml
<!-- LoginPage.xaml -->
<ContentPage xmlns="http://schemas.microsoft.com/dotnet/2021/maui"
             x:Class="RubikCare.Mobile.Features.Auth.Views.LoginPage">

    <VerticalStackLayout>
        <!-- ⭐ Binding على Property في ViewModel -->
        <Label Text="{Binding TitleText}" />

        <Entry Placeholder="{Binding EmailPlaceholder}" />

        <Button Text="{Binding LoginButtonText}"
                Command="{Binding LoginCommand}" />
    </VerticalStackLayout>
</ContentPage>
```

---

### 2.3 Code-Behind للصفحة

```csharp
// LoginPage.xaml.cs
namespace RubikCare.Mobile.Features.Auth.Views;

public partial class LoginPage : ContentPage
{
    private readonly LoginViewModel _viewModel;

    public LoginPage(LoginViewModel viewModel)
    {
        InitializeComponent();
        _viewModel = viewModel;
        BindingContext = _viewModel;
    }

    protected override async void OnAppearing()
    {
        base.OnAppearing();
        // ⭐ تحميل الترجمات عند ظهور الصفحة
        await _viewModel.InitializeAsync();
    }

    protected override void OnDisappearing()
    {
        base.OnDisappearing();
        // ⭐ إلغاء الاشتراك
        _viewModel.Dispose();
    }
}
```

---

### 2.4 زر تغيير اللغة في AppShell

أضف `ToolbarItem` في `AppShell.xaml` لتغيير اللغة من أي مكان في التطبيق:

```xml
<!-- AppShell.xaml -->
<Shell.ToolbarItems>
    <ToolbarItem Text="EN | AR"
                 Command="{Binding ToggleLanguageCommand}"
                 Order="Primary"
                 Priority="0" />
</Shell.ToolbarItems>
```

```csharp
// AppShellViewModel.cs — أضف هذا الكود
private readonly MobileTranslationState _translationState;

[RelayCommand]
private void ToggleLanguage()
{
    var newLang = _translationState.IsArabic ? "en" : "ar";
    _translationState.SetLanguage(newLang);
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

## الجزء الثالث: مسار BlazorWebView

هذا المسار **أقرب للويب** لأن الصفحات مكتوبة بـ Razor. الفرق الوحيد أن `MobileTranslationState` يحل محل `TranslationStateService`.

---

### 3.1 النمط القياسي لصفحة Blazor في Shared.UI

```razor
@* مثال: PspDashboard.razor في RubikCare.Shared.UI *@
@page "/psp/dashboard"
@implements IDisposable

@inject IMobileTranslationService TranslationService
@inject MobileTranslationState TranslationState

<h1>@T("PSP.DASHBOARD_TITLE")</h1>

<p>@T("PSP.WELCOME_MESSAGE")</p>

<button @onclick="ToggleLanguage">
    @(_currentLang == "ar" ? "English" : "العربية")
</button>

@code {
    private string _currentLang = "ar";
    private Dictionary<string, string> _translations = new();
    private const string PageDomain = "PSP_DASHBOARD";

    protected override async Task OnInitializedAsync()
    {
        _currentLang = TranslationState.CurrentLanguage;

        // ⭐ الاشتراك في تغييرات اللغة
        TranslationState.OnLanguageChanged += HandleLanguageChanged;

        await LoadTranslations();
    }

    private async Task LoadTranslations()
    {
        var lang = TranslationState.CurrentLanguage;

        // ⭐ تحميل دفعة واحدة — نفس نمط الويب
        var pageTrans = await TranslationService.GetPageTranslationsAsync(PageDomain, lang);
        var commonTrans = await TranslationService.GetPageTranslationsAsync("COMMON", lang);

        _translations.Clear();
        foreach (var item in commonTrans) _translations[item.Key] = item.Value;
        foreach (var item in pageTrans) _translations[item.Key] = item.Value;

        _currentLang = lang;
    }

    private async void HandleLanguageChanged()
    {
        await LoadTranslations();
        await InvokeAsync(StateHasChanged); // ⭐ يحدث الـ UI
    }

    // ⭐ دالة T() المساعدة — نفس الويب حرفياً
    private string T(string key) =>
        _translations.TryGetValue(key, out var value) ? value : key;

    private void ToggleLanguage()
    {
        var newLang = _currentLang == "ar" ? "en" : "ar";
        TranslationState.SetLanguage(newLang);
    }

    // ⭐ إلزامي: إلغاء الاشتراك عند مغادرة الصفحة
    public void Dispose()
    {
        TranslationState.OnLanguageChanged -= HandleLanguageChanged;
    }
}
```

---

### 3.2 تسجيل الخدمات لـ BlazorWebView

في صفحات `Shared.UI` داخل الموبايل، الخدمات تُحقن من خلال نفس الـ DI container. لا يلزم تسجيل إضافي إذا تم التسجيل في `MauiProgram.cs` (راجع 1.3).

⚠️ **تنبيه:** إذا كانت صفحات `Shared.UI` تُستخدم أيضاً في مشروع الويب، تأكد أن الـ interface `IMobileTranslationService` مسجل في الويب أيضاً أو استخدم interface مشترك.

---

### 3.3 Layout مشترك لصفحات BlazorWebView

إذا أردت زر اللغة في كل صفحات BlazorWebView، أضفه في الـ Layout:

```razor
@* MobileLayout.razor في Shared.UI *@
@inherits LayoutComponentBase
@inject MobileTranslationState TranslationState

<div dir="@(_isArabic ? "rtl" : "ltr")" lang="@TranslationState.CurrentLanguage">
    <header>
        <button @onclick="ToggleLanguage" class="lang-btn">
            @(_isArabic ? "English" : "العربية")
        </button>
    </header>

    <main>
        @Body
    </main>
</div>

@code {
    private bool _isArabic => TranslationState.CurrentLanguage == "ar";

    private void ToggleLanguage()
    {
        var newLang = _isArabic ? "en" : "ar";
        TranslationState.SetLanguage(newLang);
    }
}
```

---

### 3.4 CHECKLIST لكل صفحة BlazorWebView جديدة

- [ ] `@implements IDisposable` في أعلى الصفحة
- [ ] `@inject IMobileTranslationService` و `@inject MobileTranslationState`
- [ ] الاشتراك في `OnLanguageChanged` في `OnInitializedAsync`
- [ ] `LoadTranslations()` تحمّل domain الصفحة + COMMON دفعة واحدة
- [ ] `HandleLanguageChanged` تستدعي `InvokeAsync(StateHasChanged)`
- [ ] `T("key")` في كل النصوص — لا نصوص ثابتة
- [ ] `Dispose()` يلغي الاشتراك — **منع Memory Leak**
- [ ] مفاتيح الترجمة موجودة في قاعدة البيانات

---

## الجزء الرابع: مرجع سريع للـ Domains

استخدم هذا الجدول كمرجع عند إضافة مفاتيح ترجمة جديدة:

| الصفحة / المجال | Domain المقترح | نوع الصفحة |
|---|---|---|
| تسجيل الدخول | `MOBILE_LOGIN` | XAML |
| التسجيل | `MOBILE_REGISTER` | XAML |
| لوحة تحكم الطبيب | `MOBILE_DOCTOR_DASHBOARD` | XAML |
| دعوة مريض | `MOBILE_INVITE_PATIENT` | XAML |
| البحث عن برنامج PSP | `MOBILE_PSP_SEARCH` | BlazorWebView |
| تفاصيل البرنامج | `MOBILE_PSP_DETAIL` | BlazorWebView |
| صرف الدواء | `MOBILE_DISPENSE` | BlazorWebView |
| نصوص مشتركة | `COMMON` | كلاهما |

---

## الجزء الخامس: إضافة مفاتيح الترجمة في قاعدة البيانات

```sql
-- ⭐ لا تنسَ N'' prefix للعربية دائماً
INSERT INTO Resources (ResourceKey, ResourceValueAr, ResourceValueEn, Module, IsActive)
VALUES
    (N'MOBILE_LOGIN.TITLE',            N'تسجيل الدخول',     N'Login',           N'MOBILE_LOGIN',  1),
    (N'MOBILE_LOGIN.EMAIL_PLACEHOLDER',N'البريد الإلكتروني', N'Email',           N'MOBILE_LOGIN',  1),
    (N'MOBILE_LOGIN.PASSWORD',         N'كلمة المرور',       N'Password',        N'MOBILE_LOGIN',  1),
    (N'COMMON.LOGIN',                  N'دخول',              N'Sign In',         N'COMMON',        1),
    (N'COMMON.CANCEL',                 N'إلغاء',             N'Cancel',          N'COMMON',        1),
    (N'COMMON.SAVE',                   N'حفظ',               N'Save',            N'COMMON',        1);
```

---

## الجزء السادس: تحذيرات ومحاذير

### 🔴 ممنوعات مطلقة

| # | الممنوع | البديل |
|---|---------|--------|
| 1 | نصوص Hardcoded في XAML أو Razor | استخدم `{Binding TitleText}` أو `@T("KEY")` |
| 2 | نسيان `Dispose()` في Blazor | Memory Leak يتراكم مع كل فتح صفحة |
| 3 | نسيان `_translationState.OnLanguageChanged -= ...` في XAML ViewModel | نفس المشكلة |
| 4 | استدعاء الـ API في كل رسم (Render) | حمّل مرة في `OnInitializedAsync` فقط |
| 5 | SQL بدون `N''` prefix للعربية | تظهر `????` في قاعدة البيانات |

### 🟡 تنبيهات مهمة

- `MobileTranslationState` هو `Singleton` — تغيير اللغة يؤثر على التطبيق كله فوراً
- الـ Cache مدته 30 دقيقة — تغيير الترجمات في DB يحتاج إعادة تشغيل التطبيق أو مسح الـ Cache
- في XAML، تحديث الـ UI يتم عبر `[ObservableProperty]` + `OnPropertyChanged` تلقائياً
- في Blazor، تحديث الـ UI **يحتاج** `InvokeAsync(StateHasChanged)` صريحة

---

## 🔗 روابط ذات صلة

- [01 - Program.cs والتسجيلات الأساسية](01-program-cs-foundation.md)
- [05 - إنشاء الصفحات والمكونات](05-page-creation-checklist.md)
- [11 - دليل BlazorWebView](11-blazor-webview-guide.md)
- [06 - منهجية حل المشاكل](06-troubleshooting-methodology.md)
