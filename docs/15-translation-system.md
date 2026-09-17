# 🌐 Translation System (نظام الترجمة الشامل)

آخر تحديث: 17 سبتمبر 2026 | الأولوية: 🔴 حرج

## 📌 مقدمة

هذا المرجع يوثق **نظام الترجمة الكامل** في مشروع RubikCare، ويغطي **ثلاثة مسارات**:

| # | المسار | التقنية | الاستخدام |
|---|--------|---------|-----------|
| **1** | **Web Dashboard** | Blazor Server (MainLayout + InteractiveMenu) | التطبيق الداخلي |
| **2** | **Marketing Site** | Blazor Server (MarketingLayout + MarketingHeader) | الموقع التسويقي |
| **3** | **Mobile** | XAML + BlazorWebView | تطبيق MAUI |

**المبدأ الأساسي:**

> **API واحد للترجمة** (`/api/localization/page/{domain}?lang={lang}`)
> **واجهة موحّدة** (`ISharedTranslationService` + `ISharedTranslationState`)
> **سلوك مختلف لكل منصة** (Cookie + LocalStorage للويب، Preferences للموبايل)

---

## 🏗️ الجزء الأول: الأساس المشترك (Shared.UI)

**هذا الجزء إلزامي لكل المسارات الثلاثة.**

### 1.1 الواجهات المشتركة

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

### 1.2 قواعد التسمية (Naming Convention)

| البادئة | المنصة | مثال |
|---------|--------|------|
| `SHARED.` | مشترك بين Web + Mobile (BlazorWebView) | `SHARED.SUPPORT.TITLE` |
| `WEB.` | الويب فقط | `WEB.DASHBOARD.WELCOME` |
| `HOME.` | الموقع التسويقي — الصفحة الرئيسية | `HOME.HERO.TITLE.LINE1` |
| `MANUFACTURERS.` | الموقع التسويقي — شركات الأدوية | `MANUFACTURERS.HERO.TITLE` |
| `PATIENTS.` | الموقع التسويقي — المرضى | `PATIENTS.HERO.TITLE` |
| `DOCTORS.` | الموقع التسويقي — الأطباء | `DOCTORS.HERO.TITLE` |
| `PHARMACIES.` | الموقع التسويقي — الصيدليات | `PHARMACIES.HERO.TITLE` |
| `COMPLIANCE.` | الموقع التسويقي — الامتثال | `COMPLIANCE.HERO.TITLE` |
| `DEMO.` | الموقع التسويقي — طلب عرض | `DEMO.FORM.TITLE` |
| `MOBILE.` | الموبايل فقط (XAML) | `MOBILE.LOGIN.EMAIL` |
| `COMMON` | نصوص عامة | `COMMON.SAVE` |

### 1.3 قواعد البيانات

**جدول `Resources` — الأعمدة:**

| العمود | النوع | ملاحظات |
|--------|------|---------|
| `ResourceKey` | `nvarchar` | PRIMARY KEY — فريد |
| `ResourceValueAr` | `nvarchar` | القيمة العربية — **يجب `N''`** |
| `ResourceValueEn` | `nvarchar` | القيمة الإنجليزية |
| `Module` | `nvarchar` | `HOME` / `SHARED` / `MOBILE` / `COMMON` |
| `ResourceType` | `nvarchar` | `Title` / `Text` / `Button` / `Label` |
| `IsActive` | `bit` | 1 = نشط |
| `CreatedDate` | `datetime` | `GETDATE()` |
| `LastModifiedDate` | `datetime` | عند التحديث |

### 1.4 SQL — قواعد صارمة

**⭐ استخدم `N''` للعربية دائمًا:**

```sql
-- ❌ خطأ — يُنتج ????
INSERT INTO Resources (ResourceValueAr, ...) VALUES ('عنوان', ...);

-- ✅ صحيح
INSERT INTO Resources (ResourceValueAr, ...) VALUES (N'عنوان', ...);
```

**⭐ استخدم `MERGE` بدلًا من INSERT/UPDATE منفصلة:**

```sql
MERGE Resources AS target
USING (VALUES
    (N'HOME.HERO.TITLE.LINE1', N'تحصل على دوائها.', N'She gets her medicine.', N'HOME', N'Title'),
    (N'HOME.HERO.TITLE.LINE2', N'كل شهر', N'Every month', N'HOME', N'Title'),
    (N'HOME.HERO.BADGE',       N'برامج دعم المرضى · مصر', N'Patient Support Programs · Egypt', N'HOME', N'Badge')
) AS source (ResourceKey, ResourceValueAr, ResourceValueEn, Module, ResourceType)
ON target.ResourceKey = source.ResourceKey AND target.Module = source.Module
WHEN MATCHED THEN
    UPDATE SET
        ResourceValueAr = source.ResourceValueAr,
        ResourceValueEn = source.ResourceValueEn,
        LastModifiedDate = GETDATE()
WHEN NOT MATCHED THEN
    INSERT (ResourceKey, ResourceValueAr, ResourceValueEn, Module, ResourceType, IsActive, CreatedDate)
    VALUES (source.ResourceKey, source.ResourceValueAr, source.ResourceValueEn, source.Module, source.ResourceType, 1, GETDATE());
```

**⚠️ ملاحظة مهمة:** `MERGE` يحتاج **مفتاحين** للتطابق (`ResourceKey` + `Module`) — لأن نفس المفتاح قد يتكرر في modules مختلفة.

**⭐ التحقق قبل الإدراج:**

```sql
SELECT ResourceKey, ResourceValueAr, ResourceValueEn, Module
FROM Resources
WHERE ResourceKey LIKE 'HOME.%'
ORDER BY ResourceKey;
```

---

## 🖥️ الجزء الثاني: Web Dashboard (Blazor Server)

### 2.1 المكوّنات الأساسية

| الملف | الدور |
|-------|------|
| `BasePage.cs` | الفئة الأساسية للصفحات — تحمّل الترجمات |
| `TranslationStateService` | إدارة حالة اللغة (Cookie + LocalStorage) |
| `WebTranslationState` | تنفيذ `ISharedTranslationState` للويب |
| `WebTranslationService` | تنفيذ `ISharedTranslationService` للويب |
| `ILocalizationService` | خدمة الوصول لقاعدة البيانات + Cache |

### 2.2 نمط `BasePage`

**المسار:** `Rubikcare.Web/Components/Base/BasePage.cs`

**كل صفحة في الويب يجب أن ترث `BasePage`:**

```razor
@page "/dashboard"
@inherits BasePage
@layout MainLayout
@rendermode InteractiveServer

<PageTitle>@T("WEB.DASHBOARD.TITLE")</PageTitle>

<h1>@T("WEB.DASHBOARD.WELCOME")</h1>

@code {
    protected override string GetPageDomain() => "WEB.DASHBOARD";
}
```

**المزايا:**
- `T("key")` جاهز في كل صفحة
- `OnLanguageChanged` يعيد الرسم تلقائيًا
- Cache على مستوى السيرفر

### 2.3 نمط المكوّن التفاعلي داخل Layout ثابت

**المشكلة:** `LayoutComponentBase` **لا يقبل `@rendermode`** (بسبب `RenderFragment Body`).

**الحل:** استخراج الجزء التفاعلي في **مكوّن فرعي**.

**مثال — `MainLayout.razor` (Static) + `InteractiveMenu.razor` (Interactive):**

```razor
@* MainLayout.razor — Static SSR *@
@inherits LayoutComponentBase

<InteractiveMenu />
<main>@Body</main>
```

```razor
@* InteractiveMenu.razor — Interactive *@
@rendermode InteractiveServer
@inject TranslationStateService TranslationState
```

**نفس النمط في الموقع التسويقي:**
- `MarketingLayout.razor` (Static)
- `MarketingHeader.razor` (Interactive)
- `LanguageSwitcher.razor` (Interactive)

### 2.4 خدمة `TranslationStateService`

**المسار:** `RubikCare.Application/Services/TranslationStateService.cs`

**المسؤوليات:**
1. قراءة اللغة من **Cookie** (`RubikCare.Language`)
2. حفظ اللغة في **LocalStorage** + **Cookie**
3. إطلاق `OnLanguageChanged` عند التغيير
4. **قراءة فورية من Cookie عند كل استدعاء** — لمنع الانقلاب

**⚠️ قاعدة حرجة — `CurrentLanguage` يجب أن تقرأ Cookie فورًا:**

```csharp
public string CurrentLanguage
{
    get
    {
        if (!string.IsNullOrEmpty(_currentLanguage))
            return _currentLanguage;

        // ⭐ قراءة فورية من Cookie
        try
        {
            var httpContext = _httpContextAccessor?.HttpContext;
            if (httpContext != null)
            {
                var cookieValue = httpContext.Request.Cookies[".RubikCare.Language"];
                if (!string.IsNullOrEmpty(cookieValue))
                {
                    var decoded = Uri.UnescapeDataString(cookieValue);
                    var match = Regex.Match(decoded, @"c=([a-z]{2})");
                    if (match.Success && (match.Groups[1].Value == "ar" || match.Groups[1].Value == "en"))
                    {
                        _currentLanguage = match.Groups[1].Value;
                        return _currentLanguage;
                    }
                }
            }
        }
        catch { }

        return "en";  // الافتراضي
    }
}
```

**⚠️ بدون هذا — تحدث قفزات بين SSR و Interactive.**

### 2.5 ⚠️ خطأ شائع: قلب المنطق

**هذه الأخطاء الثلاثة أدت إلى "توقف زر التبديل" أو "انقلاب اللغة" — تأكد من صحتها دائمًا:**

```csharp
// ❌ خطأ 1: رفض "ar"
if (languageCode != "en" && languageCode != "en") throw ...
// ✅ صحيح
if (languageCode != "ar" && languageCode != "en") throw ...

// ❌ خطأ 2: كلا الحالتين en
var inferredLang = currentDir == "ltr" ? "en" : "en";
// ✅ صحيح
var inferredLang = currentDir == "rtl" ? "ar" : "en";

// ❌ خطأ 3: قبول en فقط
if (match.Success && (value == "en" || value == "en")) ...
// ✅ صحيح
if (match.Success && (value == "ar" || value == "en")) ...
```

### 2.6 `LocalizationService` — قواعد اختيار اللغة

**⚠️ الترتيب مهم في كل استعلام:**

```csharp
// ⭐ إذا طلبت "ar" → ResourceValueAr، وإلا → ResourceValueEn
var value = language.ToLower() == "ar"
    ? resource.ResourceValueAr
    : resource.ResourceValueEn;
```

**⚠️ القيم الافتراضية يجب أن تكون `"en"`:**

```csharp
// ✅ في كل الدوال
public async Task<Dictionary<string, string>> GetPageTranslationsAsync(
    string pageDomain, string language = "en")  // ← "en" وليس "ar"
```

### 2.7 إخفاء الترجمة أثناء التحميل (منع الوميض)

**⚠️ لا تستخدم `T("COMMON.LOADING")` في صفحة تحمّل الترجمة — لأن المفتاح نفسه لن يكون مترجمًا بعد.**

**الحل:**

```razor
@if (!_translations.Any())
{
    <div class="spinner"></div>
}
else
{
    <div class="page-content">
        @* محتوى الصفحة *@
    </div>
}
```

**السبب:** في أول زيارة، `_translations` فارغة → عرض المفتاح الخام → وميض.

**الحل البديل (عند الحاجة):** النص المباشر باللغة:

```razor
<p>@(_currentLanguage == "ar" ? "جاري التحميل..." : "Loading...")</p>
```

---

## 🌐 الجزء الثالث: Marketing Site (Blazor Server)

### 3.1 الفرق عن Dashboard

| الجانب | Dashboard | Marketing |
|--------|-----------|-----------|
| **اللغة الافتراضية** | `"ar"` (السوق المصري) | **`"en"`** (شركات الأدوية العالمية) |
| **القائمة** | ديناميكية (من DB) | ثابتة |
| **التخطيط** | `MainLayout` | `MarketingLayout` |
| **الرأس** | `InteractiveMenu` | `MarketingHeader` |
| **الألوان** | `--rubik-*` | `--hp-*` |
| **الخطوط** | Cairo | Cairo + Fraunces + IBM Plex |

### 3.2 مكوّن `LanguageSwitcher`

**المسار:** `Rubikcare.Web/Components/Layout/LanguageSwitcher.razor`

```razor
@implements IDisposable
@rendermode InteractiveServer

@using RubikCare.Shared.UI.Services
@inject ISharedTranslationState TranslationState

<button type="button" class="lang-switch" @onclick="ToggleLanguageAsync">
    @(_currentLanguage == "ar" ? "EN | ع" : "ع | EN")
</button>

@code {
    private string _currentLanguage = "ar";

    protected override void OnInitialized()
    {
        _currentLanguage = TranslationState.CurrentLanguage ?? "en";
        TranslationState.OnLanguageChanged += OnLanguageChangedHandler;
    }

    private async Task ToggleLanguageAsync()
    {
        var newLang = _currentLanguage == "ar" ? "en" : "ar";
        TranslationState.SetLanguage(newLang);
        _currentLanguage = newLang;
        await Task.CompletedTask;
    }

    private void OnLanguageChangedHandler()
    {
        _currentLanguage = TranslationState.CurrentLanguage ?? "en";
        InvokeAsync(StateHasChanged);
    }

    public void Dispose()
    {
        TranslationState.OnLanguageChanged -= OnLanguageChangedHandler;
    }
}
```

### 3.3 ⚠️ قاعدة جوهرية: مصدر واحد للغة

**هذه القاعدة تحل مشكلة "القفزات" في الاتجاه واللغة:**

**⭐ في `App.razor` — السكربت الأولي:**

```html
<script>
    (function() {
        try {
            var lang = null;

            // 1. LocalStorage (الأولوية القصوى)
            var stored = localStorage.getItem('RubikCare:Language');
            if (stored === 'ar' || stored === 'en') lang = stored;

            // 2. Cookie
            if (!lang) {
                var cookieMatch = document.cookie.match(/(?:^|;\s*)\.RubikCare\.Language=([^;]*)/);
                if (cookieMatch) {
                    var decoded = decodeURIComponent(cookieMatch[1]);
                    var langMatch = decoded.match(/c=([a-z]{2})/);
                    if (langMatch && (langMatch[1] === 'ar' || langMatch[1] === 'en')) {
                        lang = langMatch[1];
                    }
                }
            }

            // 3. الافتراضي
            if (!lang) lang = 'en';

            var dir = lang === 'ar' ? 'rtl' : 'ltr';
            document.documentElement.setAttribute('dir', dir);
            document.documentElement.setAttribute('lang', lang);
            if (document.body) document.body.setAttribute('dir', dir);
        } catch (e) {
            document.documentElement.setAttribute('dir', 'ltr');
            document.documentElement.setAttribute('lang', 'en');
        }
    })();
</script>
```

**⭐ في `direction-protector.js`:**

```javascript
function getCurrentLanguage() {
    try {
        // 1. LocalStorage
        var stored = localStorage.getItem(STORAGE_KEY);
        if (stored === 'ar' || stored === 'en') return stored;

        // 2. Cookie
        var cookieMatch = document.cookie.match(/(?:^|;\s*)\.RubikCare\.Language=([^;]*)/);
        if (cookieMatch) {
            var decoded = decodeURIComponent(cookieMatch[1]);
            var langMatch = decoded.match(/c=([a-z]{2})/);
            if (langMatch && (langMatch[1] === 'ar' || langMatch[1] === 'en')) {
                return langMatch[1];
            }
        }

        // 3. html lang
        var htmlLang = document.documentElement.getAttribute('lang');
        if (htmlLang === 'ar' || htmlLang === 'en') return htmlLang;

        return DEFAULT_LANG;
    } catch (e) { return DEFAULT_LANG; }
}
```

**⭐ في `TranslationStateService.CurrentLanguage`:** (مذكور في القسم 2.4)

**⭐ في `WebTranslationState`:** استخدم `ISharedTranslationState`

**⚠️ الترتيب ثابت في كل مكان:**
1. `localStorage` (الأحدث — يُكتب عند الضغط)
2. `Cookie` (يُقرأ من الخادم)
3. `html lang` (احتياطي)
4. **`"en"`** (الافتراضي النهائي)

### 3.4 CSS — دعم الاتجاهين

**استخدم `inset-inline-start/end` بدلًا من `left/right`:**

```css
/* ✅ صحيح — يعمل في RTL و LTR تلقائيًا */
.hero__image {
    position: absolute;
    inset-inline-end: 0;   /* يمين في RTL، يسار في LTR */
}

/* ❌ خطأ — ثابت */
.hero__image {
    right: 0;
}
```

**للصور التي تحتاج انعكاسًا:**

```css
.home-page[dir="ltr"] .hero__image {
    transform: scaleX(-1);
}
```

**⚠️ لا تستخدم الانعكاس إذا كانت الصورة تحتوي على نصوص.**

---

## 📱 الجزء الرابع: Mobile (XAML + BlazorWebView)

### 4.1 الفرق عن الويب

| الجانب | Web | Mobile |
|--------|-----|--------|
| **التخزين** | Cookie + LocalStorage | `Preferences` (Native) |
| **الإبلاغ** | SignalR / Blazor Circuit | `OnLanguageChanged` event |
| **API الترجمة** | `/api/localization/page/{domain}` | **نفس الـ API** |
| **`T("key")`** | في `BasePage` | في الـ ViewModel |

### 4.2 `MobileTranslationService`

**المسار:** `Mobile/Services/MobileTranslationService.cs`

```csharp
public interface IMobileTranslationService : ISharedTranslationService
{
    Task<Dictionary<string, string>> GetAllTranslationsAsync(string? lang = null);
    Task PreloadCommonTranslationsAsync();
}

public class MobileTranslationService : IMobileTranslationService
{
    private readonly ApiService _apiService;
    private readonly IMemoryCache _cache;
    private const string DefaultLang = "en";
    private const string LangKey = "RubikCare:Language";

    public string GetCurrentLanguage() => Preferences.Get(LangKey, DefaultLang);

    public async Task<Dictionary<string, string>> GetPageTranslationsAsync(
        string pageDomain, string? lang = null)
    {
        var currentLang = lang ?? GetCurrentLanguage();
        var cacheKey = $"mobile_translations_{pageDomain}_{currentLang}";

        if (_cache.TryGetValue(cacheKey, out Dictionary<string, string>? cached) && cached != null)
            return cached;

        var result = await _apiService.GetAsync<Dictionary<string, string>>(
            $"/api/localization/page/{pageDomain}?lang={currentLang}");

        var translations = result ?? new Dictionary<string, string>();
        _cache.Set(cacheKey, translations, TimeSpan.FromMinutes(30));
        return translations;
    }
}
```

### 4.3 `MobileTranslationState`

**المسار:** `Mobile/Services/MobileTranslationState.cs`

```csharp
public class MobileTranslationState : ISharedTranslationState
{
    private const string LangKey = "RubikCare:Language";
    private const string DefaultLang = "en";

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

### 4.4 التسجيل في `MauiProgram.cs`

```csharp
builder.Services.AddMemoryCache();

builder.Services.AddSingleton<MobileTranslationState>();
builder.Services.AddSingleton<IMobileTranslationService, MobileTranslationService>();
builder.Services.AddSingleton<TranslationCacheService>();

builder.Services.AddSingleton<ISharedTranslationState>(sp => sp.GetRequiredService<MobileTranslationState>());
builder.Services.AddSingleton<ISharedTranslationService>(sp => sp.GetRequiredService<IMobileTranslationService>());
```

**⚠️ Singleton:** لأن اللغة يجب أن تكون واحدة على مستوى التطبيق.

### 4.5 نمط ViewModel (XAML)

```csharp
public partial class ExampleViewModel : ObservableObject, IDisposable
{
    private readonly IMobileTranslationService _translationService;
    private readonly MobileTranslationState _translationState;
    private Dictionary<string, string> _translations = new();

    [ObservableProperty] private string _titleText = string.Empty;

    public ExampleViewModel(
        IMobileTranslationService translationService,
        MobileTranslationState translationState)
    {
        _translationService = translationService;
        _translationState = translationState;

        _translationState.OnLanguageChanged += OnLanguageChangedHandler;
    }

    public async Task InitializeAsync() => await LoadTranslationsAsync();

    private async Task LoadTranslationsAsync()
    {
        var lang = _translationState.CurrentLanguage;
        var pageTrans = await _translationService.GetPageTranslationsAsync("MOBILE.EXAMPLE", lang);
        var commonTrans = await _translationService.GetPageTranslationsAsync("COMMON", lang);

        _translations = commonTrans
            .Concat(pageTrans)
            .GroupBy(x => x.Key)
            .ToDictionary(g => g.Key, g => g.First().Value);

        TitleText = T("MOBILE.EXAMPLE.TITLE");
    }

    private string T(string key) =>
        _translations.TryGetValue(key, out var value) ? value : key;

    private async void OnLanguageChangedHandler() => await LoadTranslationsAsync();

    public void Dispose() => _translationState.OnLanguageChanged -= OnLanguageChangedHandler;
}
```

### 4.6 Checklist لكل صفحة XAML

- [ ] ViewModel يحقن `IMobileTranslationService` و `MobileTranslationState`
- [ ] الاشتراك في `OnLanguageChanged` في constructor
- [ ] `LoadTranslationsAsync()` تحمّل `domain` الصفحة + `COMMON` دفعة واحدة
- [ ] `ApplyTranslations()` تُحدِّث كل Properties
- [ ] XAML يستخدم `{Binding TitleText}` — لا نصوص ثابتة
- [ ] `OnAppearing` تستدعي `InitializeAsync()`
- [ ] `OnDisappearing` تستدعي `Dispose()`
- [ ] مفاتيح الترجمة موجودة في `Resources` مع `N''`

---

## 📊 جدول مقارنة شامل

| العنصر | Web Dashboard | Marketing Site | Mobile |
|--------|---------------|----------------|--------|
| **اللغة الافتراضية** | `ar` | `en` | `en` |
| **التخزين** | Cookie + LocalStorage | Cookie + LocalStorage | Preferences |
| **الحقن** | `[Inject] TranslationStateService` | `[Inject] ISharedTranslationState` | Constructor Injection |
| **`BasePage`** | ✅ | ✅ | ❌ (ViewModels) |
| **`T("key")`** | في `BasePage` | في `BasePage` | في ViewModel |
| **تحديث UI** | `StateHasChanged` | `StateHasChanged` | `[ObservableProperty]` |
| **RTL/LTR** | `dir` attribute | `dir` attribute | `I18nManager` |
| **التنظيف** | `Dispose()` | `Dispose()` | `Dispose()` + `OnDisappearing` |
| **الأنماط** | `--rubik-*` | `--hp-*` | `--rubik-*` |

---

## ⚠️ الجزء السادس: تحذيرات ومحاذير

### 🔴 ممنوعات مطلقة

| # | الممنوع | البديل |
|---|---------|--------|
| **1** | نصوص Hardcoded | `@T("KEY")` |
| **2** | نسيان `Dispose()` | Memory Leak |
| **3** | استدعاء API في كل Render | `OnInitializedAsync` فقط |
| **4** | SQL بدون `N''` | تظهر `????` |
| **5** | حقن `MobileTranslationService` في Shared.UI | `ISharedTranslationService` |
| **6** | `INSERT` بدلًا من `MERGE` | `MERGE` |
| **7** | قلب منطق اللغة (انظر 2.5) | تحقق من كل شرط |
| **8** | `T("COMMON.LOADING")` في صفحة تحمّل الترجمة | spinner بدون نص |
| **9** | `left/right` في CSS | `inset-inline-start/end` |
| **10** | `@rendermode` على `LayoutComponentBase` | مكوّن فرعي Interactive |

### 🟡 تنبيهات مهمة

- **`TranslationStateService` (Web):** Singleton مُسجَّل، يُشارك بين كل الجلسات
- **`MobileTranslationState`:** Singleton — تغيير اللغة يؤثر على التطبيق كله فورًا
- **Cache:** 30 دقيقة — تعديل DB يحتاج Restart
- **`LocalizationCacheService`:** Singleton — يُشارك بين Circuits
- **`PersistentComponentState`:** إذا استُخدم، يجب أن يكون في `BasePage`

---

## 🟠 الجزء السابع: المشاكل المعروفة والحلول

### 7.1 بعض الصفحات تعرض مفاتيح بدلًا من النصوص

**السبب:** `GetPageTranslationsAsync` يُرجع فاضي.
**الحل:** تحقق من `Module` في جدول `Resources`.

### 7.2 ترجمة جزئية

**السبب:** المفاتيح موجودة في `COMMON` وليس في `domain` الصفحة.
**الحل:**

```csharp
var pageTrans = await TranslationService.GetPageTranslationsAsync(PageDomain, lang);
var commonTrans = await TranslationService.GetPageTranslationsAsync("COMMON", lang);
_translations = commonTrans.Concat(pageTrans).ToDictionary(k => k.Key, v => v.Value);
```

### 7.3 تغيير اللغة لا يؤثر على الصفحة الحالية

**السبب:** نسيان `InvokeAsync(StateHasChanged)`.
**الحل:**

```csharp
private async void HandleLanguageChanged()
{
    await LoadTranslations();
    await InvokeAsync(StateHasChanged);  // ⭐ إلزامي
}
```

### 7.4 تسرب الذاكرة

**السبب:** نسيان إلغاء الاشتراك.
**الحل:**

```csharp
public void Dispose()
{
    TranslationState.OnLanguageChanged -= HandleLanguageChanged;
}
```

### 7.5 ⭐ القفزات بين SSR و Interactive (مشكلة محلولة)

**السبب:** `TranslationStateService.CurrentLanguage` **لم تكن تقرأ Cookie فورًا** — كانت تُرجع `"en"` (الافتراضي) → انقلاب.

**الحل:** `CurrentLanguage` **تقرأ Cookie عند كل استدعاء** (انظر 2.4).

**⚠️ هذه كانت مشكلة متجذرة — تأكد من عدم إعادة كسرها.**

### 7.6 زر التبديل لا يعمل

**السبب:** شرط `SetLanguageAsync` يرفض `"ar"`.
**الحل:** تحقق من الشرط `languageCode != "ar" && languageCode != "en"`.

### 7.7 وميض `COMMON.LOADING` في أول زيارة

**السبب:** الصفحة تعرض `T("COMMON.LOADING")` قبل تحميل الترجمات.
**الحل:** Spinner بدون نص (انظر 2.7).

---

## 📁 هيكل الملفات الكامل

```
📁 Shared.UI/Services/
├── ITranslationService.cs
└── ITranslationState.cs

📁 RubikCare.Application/Services/
├── TranslationStateService.cs        ⭐ Web + Marketing
└── LocalizationService.cs

📁 Rubikcare.Web/Services/
├── WebTranslationState.cs
└── WebTranslationService.cs

📁 Rubikcare.Web/Components/Base/
└── BasePage.cs

📁 Rubikcare.Web/Components/Layout/
├── MainLayout.razor                  (Dashboard)
├── InteractiveMenu.razor             (Dashboard header)
├── MarketingLayout.razor             (Marketing)
├── MarketingHeader.razor             (Marketing header)
└── LanguageSwitcher.razor            (Marketing toggle)

📁 Rubikcare.Web/wwwroot/Assets/js/
└── direction-protector.js

📁 Mobile/Services/
├── MobileTranslationService.cs
├── MobileTranslationState.cs
└── TranslationCacheService.cs
```

---

## 🔗 روابط ذات صلة

- [00 - الهيكل المعماري](../00-architecture.md)
- [02 - نظام الهوية والمصادقة](../02-identity-system.md)
- [05 - إنشاء الصفحات والمكونات](../05-page-creation-checklist.md)
- [09 - دليل API](../09-api-guide.md)

---

**آخر تحديث:** 17 سبتمبر 2026
**الملف:** `15-translation-system.md`
**الحالة:** ✅ مستقر — لا تعدّل منطق اللغة دون مراجعة 7.5
```
