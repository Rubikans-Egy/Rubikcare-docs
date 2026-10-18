
# 04 — TRANSLATION (نظام الترجمة الشامل + الاتجاه)

**الإصدار:** 2.0 (Unified Edition)
**آخر تحديث:** 16 أكتوبر 2026
**الحالة:** ✅ مرجع شامل — يدمج 15 + 17 + 19 + الكود الفعلي
**الجمهور:** فريق التطوير + AI Assistants

---

## 📌 الجزء صفر: الفلسفة والرؤية (نص المؤلف الحرفي)

> **⚠️ ملاحظة:**
> هذا القسم يحتوي على **نص الكاتب الحرفي** — دون إعادة صياغة.

### 0.1 الرؤية الأساسية

> **"API واحد للترجمة + واجهة موحّدة + سلوك مختلف لكل منصة."**

- **API واحد:** `/api/localization/page/{domain}?lang={lang}`.
- **واجهة موحّدة:** `ISharedTranslationService` + `ISharedTranslationState`.
- **سلوك مختلف:**
  - **Web:** Cookie + LocalStorage.
  - **Mobile:** Preferences (Native).

### 0.2 المسارات الثلاثة

| # | المسار | التقنية | الاستخدام |
|---|--------|---------|-----------|
| **1** | **Web Dashboard** | Blazor Server (MainLayout + InteractiveMenu) | التطبيق الداخلي |
| **2** | **Marketing Site** | Blazor Server (MarketingLayout + MarketingHeader) | الموقع التسويقي |
| **3** | **Mobile** | XAML + BlazorWebView | تطبيق MAUI |

### 0.3 حالة الاستقرار — مقارنة Web vs Mobile

**⚠️ ملاحظة حرجة (من جلسة 16 أكتوبر):**

> "أريد توثيق أن هناك **استقرار وأداء جيد في الويب** بشكل عام في هذه الجزئية — سواء الاتجاه أو الترجمة.
> **عكس الوضع في الموبايل**، حيث أن هناك مشكلة تتمثل في **ظهور مفاتيح الترجمة في أحيان كثيرة**، وأعتقد أنها مشكلة خاصة بالتخزين المحلي وتوقيت تحميل الترجمات، وسنحتاج علاجها في وقت لاحق."

**⚠️ التقييم:**

| المنصة | الترجمة | الاتجاه | الأداء | الحالة |
|:---:|:---:|:---:|:---:|:---|
| **Web** | ✅ مستقر | ✅ مستقر | ✅ جيد | 🟢 |
| **Marketing** | ✅ مستقر | ✅ مستقر | ✅ جيد | 🟢 |
| **Mobile** | ⚠️ مشاكل متقطعة | ✅ مستقر | ⚠️ متوسط | 🟡 |

**⚠️ مشكلة Mobile المحددة:**

- **الأعراض:** ظهور مفاتيح الترجمة (`PSP.FORM.EGP` بدلاً من "جنيه مصري").
- **السبب المُرجَّح:** توقيت تحميل الترجمات + التخزين المحلي (`Preferences`).
- **الحالة:** 🟡 مؤجّلة — تحتاج جلسة منفصلة.

---

## 📌 الجزء الأول: الأساس المشترك (Shared.UI)

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
| `SHARED.` | مشترك بين Web + Mobile | `SHARED.SUPPORT.TITLE` |
| `WEB.` | الويب فقط | `WEB.DASHBOARD.WELCOME` |
| `HOME.` | الموقع التسويقي — الرئيسية | `HOME.HERO.TITLE.LINE1` |
| `MANUFACTURERS.` | الموقع التسويقي — شركات الأدوية | `MANUFACTURERS.HERO.TITLE` |
| `PATIENTS.` | الموقع التسويقي — المرضى | `PATIENTS.HERO.TITLE` |
| `DOCTORS.` | الموقع التسويقي — الأطباء | `DOCTORS.HERO.TITLE` |
| `PHARMACIES.` | الموقع التسويقي — الصيدليات | `PHARMACIES.HERO.TITLE` |
| `COMPLIANCE.` | الموقع التسويقي — الامتثال | `COMPLIANCE.HERO.TITLE` |
| `DEMO.` | الموقع التسويقي — طلب عرض | `DEMO.FORM.TITLE` |
| `MOBILE.` | الموبايل فقط (XAML) | `MOBILE.LOGIN.EMAIL` |
| `COMMON.` | نصوص عامة | `COMMON.SAVE` |

### 1.3 جدول `Resources` — البنية الفعلية

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

**⭐ استخدم `MERGE` بدلًا من INSERT/UPDATE:**

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

## 📌 الجزء الثاني: Web Dashboard (Blazor Server)

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
- `T("key")` جاهز في كل صفحة.
- `OnLanguageChanged` يعيد الرسم تلقائيًا.
- Cache على مستوى السيرفر.

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
- `MarketingLayout.razor` (Static).
- `MarketingHeader.razor` (Interactive).
- `LanguageSwitcher.razor` (Interactive).

### 2.4 خدمة `TranslationStateService` — الكود الفعلي

**المسار:** `RubikCare.Application/Services/TranslationStateService.cs`

**المسؤوليات:**
1. قراءة اللغة من **Cookie** (`.RubikCare.Language`).
2. حفظ اللغة في **LocalStorage** + **Cookie**.
3. إطلاق `OnLanguageChanged` عند التغيير.
4. **قراءة فورية من Cookie عند كل استدعاء** — لمنع الانقلاب.

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
                var cookieValue = httpContext.Request.Cookies[".RubikCare.Language"]
                               ?? httpContext.Request.Cookies["RubikCare.Language"];

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

        // الافتراضي
        return "en";
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

## 📌 الجزء الثالث: Marketing Site (Blazor Server)

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

**هذه القاعدة تحل مشكلة "القفزات" في الاتجاه واللغة.**

**⭐ في `App.razor` — السكربت الأولي (الكود الفعلي):**

```html
<script>
    (function () {
        try {
            var lang = null;

            // 1. localStorage (الأولوية القصوى)
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

**⭐ في `direction-protector.js` — الكود الفعلي:**

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

**⭐ في `TranslationStateService.CurrentLanguage`:** (مذكور في القسم 2.4).

**⭐ في `WebTranslationState`:** استخدم `ISharedTranslationState`.

**⚠️ الترتيب ثابت في كل مكان:**
1. `localStorage` (الأحدث — يُكتب عند الضغط).
2. `Cookie` (يُقرأ من الخادم).
3. `html lang` (احتياطي).
4. **`"en"`** (الافتراضي النهائي).

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

## 📌 الجزء الرابع: Mobile (MAUI) — مع المشاكل الحالية

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

### 4.6 ⚠️ المشاكل المعروفة في Mobile

**⚠️ من ملاحظتك (16 أكتوبر):**

> "هناك مشكلة تتمثل في ظهور مفاتيح الترجمة في أحيان كثيرة، وأعتقد أنها مشكلة خاصة بالتخزين المحلي وتوقيت تحميل الترجمات."

**⚠️ الأعراض:**
- ظهور `PSP.FORM.EGP` بدلاً من "جنيه مصري".
- ظهور `MOBILE.LOGIN.EMAIL` بدلاً من "البريد الإلكتروني".
- **متقطع** — ليس دائمًا.

**⚠️ الأسباب المُرجَّحة:**

| # | السبب | التفصيل |
|:-:|:---|:---|
| 1 | **توقيت التحميل** | `LoadTranslationsAsync()` قد لا تُكتمل قبل الرسم |
| 2 | **التخزين المحلي** | `Preferences` قد لا يُقرأ فورًا |
| 3 | **Cache متأخر** | `IMemoryCache` قد يُعيد قيمة فارغة |
| 4 | **عدم انتظار API** | `_apiService.GetAsync` قد يفشل صامتًا |

**⚠️ الحالة:** 🟡 **مؤجّلة** — تحتاج جلسة منفصلة.

**⚠️ توصيات مبدئية:**
- **`PreloadCommonTranslationsAsync`** في `MauiProgram` — لتحميل الترجمات الشائعة قبل الحاجة.
- **Cache Synchronous** — قراءة من `Preferences` فورًا عند الإقلاع.
- **Fallback**: إن لم يُوجد مفتاح → أظهر مفتاح بديل مفهوم.

### 4.7 Checklist لكل صفحة XAML

- [ ] ViewModel يحقن `IMobileTranslationService` و `MobileTranslationState`.
- [ ] الاشتراك في `OnLanguageChanged` في constructor.
- [ ] `LoadTranslationsAsync()` تحمّل `domain` الصفحة + `COMMON` دفعة واحدة.
- [ ] `ApplyTranslations()` تُحدِّث كل Properties.
- [ ] XAML يستخدم `{Binding TitleText}` — لا نصوص ثابتة.
- [ ] `OnAppearing` تستدعي `InitializeAsync()`.
- [ ] `OnDisappearing` تستدعي `Dispose()`.
- [ ] مفاتيح الترجمة موجودة في `Resources` مع `N''`.

---# [#54] 📘 `04-TRANSLATION` — v2.0 (Part 2)

---

## 📌 الجزء الخامس: Web Localization Direction (من 19)

### 5.1 لماذا كانت المشكلة صعبة؟

**Blazor Server يعمل في مرحلتين منفصلتين لكل طلب صفحة:**

```
المرحلة 1: SSR (Static Server Rendering)
  → الخادم يُنشئ HTML كاملاً
  → يُرسل للمتصفح فوراً
  → المكونات تعمل لكن بدون JavaScript

المرحلة 2: Interactive (WebSocket)
  → Blazor يتصل بالخادم عبر WebSocket
  → ينشئ Scope جديد لكل الخدمات (Scoped)
  → المكونات تصبح تفاعلية
```

**⚠️ المشكلة الجذرية:** `TranslationStateService` هو **Scoped** — يعني في كل مرحلة ينشئ Blazor **instance جديد** يبدأ من صفر بـ `_currentLanguage = null` (= "en" افتراضياً).

### 5.2 التسلسل الزمني الكامل لما يحدث عند Refresh

```
1. المتصفح يرسل طلب GET /settings
   → الـ cookie: .RubikCare.Language=c%3Den%7Cuic%3Den

2. SSR Phase:
   → TranslationStateService (Instance #1) يُنشأ
   → App.razor.OnInitializedAsync → InitializeAsync()
   → يقرأ cookie → يجد "en" → _currentLanguage = "en" ✅
   → المكونات تُرسم بـ lang=en
   → HTML يُرسل للمتصفح

3. المتصفح يستلم HTML:
   → السكربت المبكر في <head> يقرأ cookie/localStorage
   → يضبط html[dir] = "ltr" و html[lang] = "en" ✅
   → direction-protector.js يبدأ المراقبة

4. Interactive Phase (WebSocket يتصل):
   → TranslationStateService (Instance #2) يُنشأ جديد ❌
   → _currentLanguage = null = "en" افتراضياً
   → App.razor.OnInitializedAsync → InitializeAsync()
   → httpContext = null (لأننا في Interactive mode)
   → يرجع للافتراضي "en" ❌
   → المكونات تُعاد رسمها بـ lang=ar ❌

5. بعد اكتمال الاتصال:
   → InteractiveMenu.OnAfterRenderAsync يُنفَّذ
   → SyncFromLocalStorageAsync() تقرأ localStorage = "en"
   → تُطلق OnLanguageChanged
   → جميع المكونات تُعاد رسمها بـ lang=en ✅
   → لكن هذا يحدث بعد لحظة من الرسم الخاطئ (flash مرئي)
```

### 5.3 مخطط الملفات المتداخلة

```
App.razor
├── السكربت المبكر (inline في <head>)
│   └── يقرأ cookie/localStorage ويضبط html[dir] فوراً
│
├── direction-protector.js
│   └── يراقب html[dir] ويمنع Blazor من تغييره
│
├── TranslationStateService (Scoped)
│   ├── InitializeAsync() ← تقرأ cookie في SSR فقط
│   ├── SyncFromLocalStorageAsync() ← تقرأ localStorage في Interactive
│   └── SetLanguageAsync() ← تحفظ في localStorage + cookie + تطلق حدث
│
├── MainLayout.razor (SSR)
│   └── يقرأ TranslationState.CurrentLanguage للنصوص فقط
│       لكن لا يملك OnAfterRenderAsync فعّال (SSR)
│
└── InteractiveMenu.razor (@rendermode InteractiveServer) ⭐
    └── OnAfterRenderAsync → SyncFromLocalStorageAsync()
        هذا هو المكان الوحيد الذي يُصحح اللغة في Interactive
```

### 5.4 الحل الكامل المطبق — 4 طبقات

#### 🔹 الطبقة 1: السكربت المبكر في `App.razor` (أسرع طبقة)

**يعمل قبل أي JavaScript آخر — يضبط `html[dir]` فوراً:**

```html
<!-- في <head> مباشرة قبل HeadOutlet -->
<script>
    (function () {
        try {
            var lang = null;

            // 1. localStorage (الأولوية القصوى)
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

**⚠️ ما يحله:** يضبط `html[dir]` الصحيح قبل رسم أي محتوى — يمنع الـ flash الأولي.

**⚠️ ملاحظة حرجة (من `App.razor` الفعلي):** يوجد أيضاً سكربت إضافي لـ `data-rubik-lang`:

```html
<script>
    (function () {
        try {
            var lang = localStorage.getItem('RubikCare:Language') || 'en';
            if (lang !== 'ar' && lang !== 'en') lang = 'en';
            document.documentElement.setAttribute('data-rubik-lang', lang);
        } catch (e) { }
    })();
</script>
```

#### 🔹 الطبقة 2: `direction-protector.js` — الحارس النشط (v2)

**⚠️ الملف الفعلي — النسخة المُحسَّنة (بدون Loop):**

```javascript
(function () {
    'use strict';

    const STORAGE_KEY = 'RubikCare:Language';
    const DEFAULT_LANG = 'en';

    // ⭐ Flag لمنع التكرار
    let _isApplying = false;
    let _lastApplied = { lang: null, dir: null };

    function getCurrentLanguage() {
        try {
            // الأولوية 1: localStorage
            var stored = localStorage.getItem(STORAGE_KEY);
            if (stored === 'ar' || stored === 'en') return stored;

            // الأولوية 2: Cookie
            var cookieMatch = document.cookie.match(/(?:^|;\s*)\.RubikCare\.Language=([^;]*)/);
            if (cookieMatch) {
                var decoded = decodeURIComponent(cookieMatch[1]);
                var langMatch = decoded.match(/c=([a-z]{2})/);
                if (langMatch && (langMatch[1] === 'ar' || langMatch[1] === 'en')) {
                    return langMatch[1];
                }
            }

            // الأولوية 3: html lang
            var htmlLang = document.documentElement.getAttribute('lang');
            if (htmlLang === 'ar' || htmlLang === 'en') return htmlLang;

            return DEFAULT_LANG;
        } catch (e) {
            return DEFAULT_LANG;
        }
    }

    function getDirection(lang) {
        return lang === 'en' ? 'ltr' : 'rtl';
    }

    function applyDirection(silent) {
        // ⭐ منع التكرار
        if (_isApplying) return;
        _isApplying = true;

        try {
            const lang = getCurrentLanguage();
            const dir = getDirection(lang);

            // ⭐ لو نفس القيم اللي اتطبقت قبل كده، متعملش حاجة
            if (_lastApplied.lang === lang && _lastApplied.dir === dir) {
                _isApplying = false;
                return;
            }

            const html = document.documentElement;
            const body = document.body;

            if (html) {
                if (html.getAttribute('dir') !== dir) html.setAttribute('dir', dir);
                if (html.getAttribute('lang') !== lang) html.setAttribute('lang', lang);
            }
            if (body) {
                if (body.getAttribute('dir') !== dir) body.setAttribute('dir', dir);
            }

            _lastApplied.lang = lang;
            _lastApplied.dir = dir;

            if (!silent) {
                console.log(`🧭 [direction-protector] Applied: ${dir} (${lang})`);
            }
        } catch (e) {
            console.warn('⚠️ [direction-protector] Failed:', e);
        } finally {
            _isApplying = false;
        }
    }

    // 1. التطبيق الفوري
    if (document.readyState === 'loading') {
        document.addEventListener('DOMContentLoaded', () => applyDirection());
    } else {
        applyDirection();
    }

    // 2. ⭐ الحارس النشط: مراقبة تغييرات dir (بدون Loop)
    const observer = new MutationObserver((mutations) => {
        if (_isApplying) return;  // ⭐ تجاهل التغييرات اللي عملناها بنفسنا

        const correctLang = getCurrentLanguage();
        const correctDir = getDirection(correctLang);

        mutations.forEach((mutation) => {
            if (mutation.type === 'attributes' && mutation.attributeName === 'dir') {
                const currentDir = mutation.target.getAttribute('dir');

                if (currentDir && currentDir !== correctDir) {
                    console.warn(`⚠️ [direction-protector] Wrong dir detected: ${currentDir}, correcting to ${correctDir}`);
                    applyDirection(true);  // silent
                }
            }
        });
    });

    // مراقبة <html> فقط (لتجنب الأداء السيئ + Loop)
    if (document.documentElement) {
        observer.observe(document.documentElement, { attributes: true, attributeFilter: ['dir'] });
    }

    // 3. الاستماع لتغييرات localStorage
    window.addEventListener('storage', (e) => {
        if (e.key === STORAGE_KEY) {
            _lastApplied = { lang: null, dir: null };  // إعادة تعيين
            applyDirection();
        }
    });

    // 4. تصدير الدالة
    window.applyRubikDirection = () => {
        _lastApplied = { lang: null, dir: null };
        applyDirection();
    };

    console.log('✅ [direction-protector] Active Guardian v2 initialized');
})();
```

**⚠️ ملاحظات مهمة من الكود:**
- **v2** — إيقاف الحلقة اللانهائية.
- **`_isApplying` flag** — لمنع إعادة الدخول.
- **`_lastApplied` cache** — لتجنب التكرار.
- **مراقبة `<html>` فقط** — ليس `<body>`.
- **`window.applyRubikDirection`** — للاستدعاء من Blazor.

#### 🔹 الطبقة 3: `TranslationStateService.InitializeAsync` (SSR)

**تقرأ الـ cookie في مرحلة SSR فقط (حيث `HttpContext` متاح):**

```csharp
public async Task InitializeAsync()
{
    if (_isInitialized) return;

    try
    {
        string detectedLang = "en";

        try
        {
            var httpContext = _httpContextAccessor?.HttpContext;
            if (httpContext != null)  // SSR فقط
            {
                var cookieValue = httpContext.Request.Cookies[".RubikCare.Language"];
                if (string.IsNullOrEmpty(cookieValue))
                    cookieValue = httpContext.Request.Cookies["RubikCare.Language"];

                if (!string.IsNullOrEmpty(cookieValue))
                {
                    var decoded = Uri.UnescapeDataString(cookieValue);
                    var match = Regex.Match(decoded, @"c=([a-z]{2})");
                    if (match.Success && (match.Groups[1].Value == "ar" || match.Groups[1].Value == "en"))
                        detectedLang = match.Groups[1].Value;
                }
            }
        }
        catch { }

        _currentLanguage = detectedLang;
        Console.WriteLine($"🌐 [TranslationState] InitializeAsync → lang={_currentLanguage}, hasHttpContext={_httpContextAccessor?.HttpContext != null}");
    }
    finally
    {
        _isInitialized = true;
    }
}
```

**⚠️ ما يحله:** يضمن أن الـ HTML المرسل من SSR يحمل اللغة الصحيحة.

**⚠️ ملاحظة:** لا تحاول قراءة JavaScript هنا — لا يعمل في `OnInitializedAsync`.

#### 🔹 الطبقة 4: `SyncFromLocalStorageAsync` في `InteractiveMenu` (Interactive) ⭐

**هذا هو المفتاح الأساسي لحل مشكلة Refresh:**

```csharp
// في InteractiveMenu.razor
protected override async Task OnAfterRenderAsync(bool firstRender)
{
    if (firstRender && _isFirstRender)
    {
        _isFirstRender = false;
        _dotNetRef = DotNetObjectReference.Create(this);

        // ⭐ يجب أن يكون أول شيء — يصحح اللغة قبل أي عملية أخرى
        await TranslationState.SyncFromLocalStorageAsync();

        try
        {
            _isMobileViewport = await JSRuntime.InvokeAsync<bool>(
                "eval", "window.innerWidth <= 768");
            await JSRuntime.InvokeVoidAsync("rubikMenu.initResizeListener", _dotNetRef);
            await JSRuntime.InvokeVoidAsync("rubikMenu.initMobileClose");
        }
        catch { }

        StateHasChanged();
    }
}
```

**⚠️ لماذا `InteractiveMenu` وليس `MainLayout` أو `App.razor`؟**

| المكون | الـ RenderMode | `OnAfterRenderAsync` يعمل؟ |
|--------|---------------|--------------------------|
| `App.razor` | SSR (لا rendermode) | ❌ لا |
| `MainLayout.razor` | SSR (لا rendermode) | ❌ لا |
| `InteractiveMenu.razor` | `@rendermode InteractiveServer` | ✅ نعم |

**⚠️ `OnAfterRenderAsync` لا تُنفَّذ إلا في مكونات Interactive — لذا `InteractiveMenu` هو المكان الوحيد الصحيح.**

#### `SyncFromLocalStorageAsync` — الكود الفعلي

```csharp
public async Task SyncFromLocalStorageAsync()
{
    try
    {
        var storedLang = await _jsRuntime.InvokeAsync<string>(
            "localStorage.getItem", "RubikCare:Language");

        if (storedLang == "ar" || storedLang == "en")
        {
            if (_currentLanguage != storedLang)
            {
                Console.WriteLine($"🔄 [Interactive] Syncing language from {_currentLanguage} to {storedLang}");
                _currentLanguage = storedLang;
                OnLanguageChanged?.Invoke(); // يُحدّث جميع المكونات المشتركة
            }
        }
        else
        {
            // ⭐ Fallback: infer from direction
            try
            {
                var currentDir = await _jsRuntime.InvokeAsync<string>("eval", "document.documentElement.getAttribute('dir')");
                var inferredLang = currentDir == "rtl" ? "ar" : "en";

                if (_currentLanguage != inferredLang)
                {
                    Console.WriteLine($"🔄 [Interactive] Inferring language from direction: {currentDir} → {inferredLang}");
                    _currentLanguage = inferredLang;
                    await _jsRuntime.InvokeVoidAsync("localStorage.setItem", "RubikCare:Language", inferredLang);
                    OnLanguageChanged?.Invoke();
                }
            }
            catch (Exception ex)
            {
                Console.WriteLine($"⚠️ Failed to infer language from direction: {ex.Message}");
            }
        }
    }
    catch (Exception ex)
    {
        Console.WriteLine($"❌ SyncFromLocalStorageAsync failed: {ex.Message}");
    }
}
```

### 5.5 CSS: يعتمد على `html[dir]` فقط — لا `main-wrapper[dir]`

```css
/* ✅ الصحيح: يقرأ من html مباشرة */
html[dir="rtl"] .interactive-sidebar {
    right: 0 !important;
    left: auto !important;
}

html:not([dir="rtl"]) .interactive-sidebar {
    left: 0 !important;
    right: auto !important;
}

/* ❌ الخاطئ: يعتمد على Blazor لتحديث dir */
.main-wrapper[dir="rtl"] .interactive-sidebar { ... }
```

**⚠️ السبب:** `html[dir]` يُضبط بـ JavaScript (السكربت المبكر + `direction-protector`) قبل Blazor، بينما `main-wrapper[dir]` يعتمد على Blazor الذي يبدأ بـ "en" دائماً.

**⚠️ وعليه يجب حذف `dir` من `main-wrapper` في `MainLayout.razor`:**

```razor
<!-- ❌ قبل الإصلاح -->
<div class="main-wrapper" dir="@(TranslationState.CurrentLanguage == "ar" ? "rtl" : "ltr")">

<!-- ✅ بعد الإصلاح (الكود الفعلي) -->
<div class="main-wrapper">
```

**⚠️ الكود الفعلي في `MainLayout.razor` (16 أكتوبر):**

```razor
<div class="main-wrapper">
    <AuthorizeView>
        <Authorized>
            <InteractiveMenu />
        </Authorized>
        <NotAuthorized>
            <header class="interactive-menu-header text-center justify-content-center">
                <span class="text-white">@_welcomeText</span>
            </header>
        </NotAuthorized>
    </AuthorizeView>
    ...
</div>
```

**⚠️ لاحظ:** `MainLayout` **لا يضع `dir`** — لكنه **يستدعي `SyncFromLocalStorageAsync`** أيضاً في `OnAfterRenderAsync`:

```csharp
protected override async Task OnAfterRenderAsync(bool firstRender)
{
    if (firstRender)
    {
        Console.WriteLine($"🔵 [MainLayout] OnAfterRenderAsync - CurrentLang BEFORE: {TranslationState.CurrentLanguage}");
        await TranslationState.SyncFromLocalStorageAsync();
        Console.WriteLine($"🟢 [MainLayout] OnAfterRenderAsync - CurrentLang AFTER: {TranslationState.CurrentLanguage}");
        StateHasChanged();
    }
}
```

**⚠️ هذا إجراء احتياطي (belt-and-suspenders)** — رغم أن `InteractiveMenu` يقوم بنفس الشيء.

### 5.6 تدفق تغيير اللغة يدوياً

**عندما يضغط المستخدم على زر تغيير اللغة في `LanguageSection.razor`:**

```
1. TranslationState.SetLanguage("en") يُستدعى
   → WebTranslationState.SetLanguage()
   → TranslationStateService.SetLanguageAsync("en")
      → localStorage.setItem("RubikCare:Language", "en")
      → document.cookie = ".RubikCare.Language=c%3Den..."
      → _currentLanguage = "en"
      → OnLanguageChanged?.Invoke()

2. OnLanguageChanged يُطلق الحدث:
   → InteractiveMenu.OnLanguageChanged → StateHasChanged
   → MainLayout.OnLanguageChangedHandler → StateHasChanged
   → SettingsPage.HandleLanguageChanged → LoadTranslations

3. window.applyRubikDirection() يُستدعى:
   → document.documentElement.setAttribute('dir', 'ltr')
   → direction-protector يُراقب ويُثبّت
```

**⚠️ الكود الفعلي في `WebTranslationState.SetLanguage`:**

```csharp
public async void SetLanguage(string lang)
{
    if (lang == CurrentLanguage) return;

    try
    {
        // 1. تحديث الحالة الداخلية في Blazor أولاً
        await _translationStateService.SetLanguageAsync(lang);

        // 2. حفظ اللغة في localStorage (المصدر الوحيد للاتجاه)
        await _jsRuntime.InvokeVoidAsync("localStorage.setItem", "RubikCare:Language", lang);

        // 3. ⭐ استدعاء دالة JavaScript لتطبيق الاتجاه فوراً ومنع الوميض
        await _jsRuntime.InvokeVoidAsync("eval", "window.applyRubikDirection()");
    }
    catch (Exception ex)
    {
        Console.WriteLine($"⚠️ Failed to save language or apply direction: {ex.Message}");
    }
}
```

### 5.7 جدول الملفات المتأثرة

| الملف | الدور | ملاحظة |
|-------|-------|--------|
| `App.razor` | السكربت المبكر في `<head>` | يضبط `html[dir]` قبل كل شيء |
| `direction-protector.js` | الحارس النشط v2 | يراقب `html[dir]` ويمنع التغيير الخاطئ |
| `TranslationStateService.cs` | مصدر الحقيقة للغة في C# | `InitializeAsync` للـ SSR، `SyncFromLocalStorageAsync` للـ Interactive |
| `LayoutDirectionService.cs` | مشتق من `TranslationStateService` | الاتجاه دائماً مشتق من اللغة — **لا تخزين منفصل** |
| `WebTranslationState.cs` | وسيط بين Blazor و `TranslationStateService` | يستدعي `applyRubikDirection()` عند التغيير |
| `WebTranslationService.cs` | تنفيذ `ISharedTranslationService` | يستدعي `ILocalizationService` |
| `InteractiveMenu.razor` | نقطة التصحيح الأولى في Interactive | يستدعي `SyncFromLocalStorageAsync` في `OnAfterRenderAsync` |
| `MainLayout.razor` | لا يضع `dir` على `main-wrapper` | يعتمد على `html[dir]` فقط — احتياطياً يستدعي Sync |
| `_sidebar.css` | CSS يقرأ من `html[dir]` فقط | لا يعتمد على `.main-wrapper[dir]` |

### 5.8 `LayoutDirectionService` — الكود الفعلي

**المسار:** `RubikCare.Application/Services/LayoutDirectionService.cs`

**⚠️ ملاحظة حرجة:** الاتجاه **مشتق دائماً وأبداً من اللغة الحالية. لا تخزين منفصل.**

```csharp
public class LayoutDirectionService : ILayoutDirectionService
{
    private readonly TranslationStateService _translationState;
    private readonly ILogger<LayoutDirectionService> _logger;
    private bool _isInitialized = false;

    public event Action? OnDirectionChanged;

    // ⭐ الاتجاه مشتق دائماً وأبداً من اللغة الحالية. لا تخزين منفصل.
    public string CurrentDirection => _translationState.CurrentLanguage == "ar" ? "rtl" : "ltr";

    public LayoutDirectionService(
        TranslationStateService translationState,
        ILogger<LayoutDirectionService> logger)
    {
        _translationState = translationState;
        _logger = logger;

        // الاستماع لتغييرات اللغة لتحديث الاتجاه فوراً
        _translationState.OnLanguageChanged += OnLanguageChangedHandler;
    }

    private void OnLanguageChangedHandler()
    {
        OnDirectionChanged?.Invoke();
    }

    public Task<string> GetDirectionAsync() => Task.FromResult(CurrentDirection);
    public Task<string> GetEffectiveDirectionAsync() => Task.FromResult(CurrentDirection);

    // تطبيق الواجهة: بدلاً من الحفظ في localStorage منفصل، نقوم بتغيير اللغة
    // هذا يضمن أن تغيير الاتجاه يغير اللغة والعكس صحيح (مصدر واحد للحقيقة)
    public async Task SetDirectionAsync(string direction)
    {
        if (direction != "rtl" && direction != "ltr")
            throw new ArgumentException("Direction must be 'rtl' or 'ltr'");

        string targetLang = direction == "rtl" ? "ar" : "en";

        // نستخدم خدمة اللغة لتغيير الحالة، وهي ستطلق الأحداث تلقائياً
        await _translationState.SetLanguageAsync(targetLang);
    }

    public async Task EnsureInitializedAsync()
    {
        if (_isInitialized) return;

        try
        {
            await _translationState.EnsureInitializedAsync();
            _logger.LogInformation("Layout direction initialized based on language: {Direction}", CurrentDirection);
        }
        catch (Exception ex)
        {
            _logger.LogError(ex, "Error initializing layout direction");
        }
        finally
        {
            _isInitialized = true;
        }
    }

    public void Dispose()
    {
        _translationState.OnLanguageChanged -= OnLanguageChangedHandler;
    }
}
```

**⚠️ قواعد ذهبية في هذا الـ Service:**

- **لا تخزين منفصل للاتجاه** — دائماً مشتق من اللغة.
- **`SetDirectionAsync`** = `SetLanguageAsync` بشكل غير مباشر.
- **`OnDirectionChanged`** = مشتق من `OnLanguageChanged`.

### 5.9 قواعد إلزامية (لا تكسرها)

1. **لا تضع `dir` على أي عنصر Blazor SSR** — فقط على `<html>` عبر JavaScript.
2. **`SyncFromLocalStorageAsync` تُستدعى فقط في مكونات `@rendermode InteractiveServer`** — لا تستدعها في `App.razor` أو `MainLayout.razor` (رغم أن `MainLayout` يحتوي عليها كاحتياطي).
3. **`InitializeAsync` لا تستدعي JavaScript** — تعمل في SSR حيث JS غير متاح.
4. **CSS يقرأ `html[dir]` فقط** — لا `.main-wrapper[dir]` أو أي عنصر Blazor آخر.
5. **اسم الـ cookie الصحيح هو `.RubikCare.Language`** (بنقطة في البداية) — تأكد من تطابقه.
6. **`localStorage` هو المصدر الوحيد الموثوق في Interactive mode** — الـ cookie يُستخدم في SSR فقط.

### 5.10 CHECKLIST: عند إضافة مكون جديد يحتاج الترجمة

- [ ] هل المكون يستخدم `TranslationState.CurrentLanguage` لقراءة اللغة؟
- [ ] هل اشترك في `TranslationState.OnLanguageChanged`؟
- [ ] هل يُلغي الاشتراك في `Dispose()`؟
- [ ] إذا كان المكون `@rendermode InteractiveServer` ويحتاج تصحيح اللغة فوراً — هل يستدعي `SyncFromLocalStorageAsync`؟ (في الغالب يكفي الاشتراك في `OnLanguageChanged`).

### 5.11 CHECKLIST: عند إضافة صفحة Layout جديدة

- [ ] هل تجنبت وضع `dir` على الـ wrapper الرئيسي؟
- [ ] هل CSS يعتمد على `html[dir]` فقط؟
- [ ] إذا كانت الصفحة تحتوي على مكون Interactive — هل يستدعي ذلك المكون `SyncFromLocalStorageAsync` في `OnAfterRenderAsync`؟

---

## 📌 الجزء السادس: Sidebar Direction Stability (من 17)

### 6.1 الغرض من هذا القسم

**توثيق السبب الجذري والحل النهائي لمشكلة "قفز القائمة الجانبية لليسار" عند التنقل أو تبديل الأوضاع في وضع اللغة العربية (RTL)، وتحديد الملفات المحظور تعديلها لضمان استقرار النظام.**

### 6.2 المشكلة الموثقة (الأعراض)

**عندما يكون التطبيق في وضع اللغة العربية:**

1. المحتوى الرئيسي يظهر بشكل صحيح على اليمين (RTL).
2. عند الضغط على أي عنصر في القائمة الجانبية، أو عند تبديل الوضع (من شخصي إلى مؤسسة)، **تقفز القائمة الجانبية فجأة إلى اليسار**.
3. تظل القائمة عالقة في اليسار حتى يتم تغيير اللغة يدوياً مرتين لإجبار النظام على إعادة تقييم الاتجاه.
4. اللغة نفسها (محتوى النصوص) تظل عربية ومستقرة، فقط **موقع القائمة** هو ما يتأثر.

### 6.3 السبب الجذري (Root Cause)

**المشكلة لم تكن في فقدان حالة اللغة (Language State)، بل في فقدان سياق الاتجاه (Direction Context) أثناء إعادة رسم Blazor للمكونات (Re-render).**

1. قواعد CSS في ملف `_sidebar.css` مصممة لتقرأ سمة الاتجاه من الغلاف الرئيسي: `.main-wrapper[dir="rtl"]`.
2. في الإصدارات السابقة، كان ملف `MainLayout.razor` يرسم `<div class="main-wrapper">` **بدون** سمة `dir`.
3. عندما يعيد Blazor رسم المكون (بسبب التنقل أو تغيير الحالة)، يفقد المتصفح مؤقتاً سياق الاتجاه لهذا العنصر المحدد، فيعود للوضع الافتراضي (LTR)، مما يجعل CSS يضع القائمة في `left: 0`.
4. في نفس الوقت، وسم `<html>` العالمي كان يحتفظ بـ `dir="rtl"`، مما يفسر لماذا كان المحتوى الرئيسي صحيحاً بينما القائمة فقط هي التي تقفز.


### 6.4 الحل النهائي المعتمد (مُحدَّث)

**⚠️ ملاحظة حرجة (16 أكتوبر):** الحل القديم كان يعتمد على إلزام Blazor بكتابة سمة `dir` على `.main-wrapper`. **هذا الحل تم إلغاؤه** لصالح الاعتماد الكامل على `html[dir]` فقط.

**لماذا؟**
- `html[dir]` يُضبط بواسطة JavaScript (السكربت المبكر + `direction-protector`) **قبل** أن يتصل Blazor.
- إذا اعتمدنا على Blazor لتحديث `dir`، فسوف يتأخر التحديث بسبب دورة حياة Blazor (SSR → Interactive).
- `html[dir]` هو المصدر الوحيد الذي يضمن استقرار الاتجاه من أول لحظة.

**الكود الفعلي في `MainLayout.razor`:**
```razor
<!-- ✅ بعد الإصلاح (الكود الفعلي) -->
<div class="main-wrapper">
    <!-- ... محتوى القائمة ... -->
</div>


### 6.5 تقسيم المسؤوليات: لماذا نستخدم JavaScript مع Blazor هنا؟

**يعتمد الحل النهائي على نموذج هجين (Hybrid Model) يضمن استقرار الاتجاه بنسبة 100% دون حدوث "حرب" بين الخادم والمتصفح.**

#### 1. دور Blazor (مدير الحالة والرسم الديناميكي)

- **المسؤولية:** قراءة اللغة الحالية من `TranslationState` وكتابة سمة `dir` ديناميكياً على عنصر `.main-wrapper` في كل مرة يتم فيها إعادة رسم المكون (Re-render).
- **⚠️ ملاحظة (16 أكتوبر):** هذا **لم يعد صحيحاً** — Blazor لم يعد يضع `dir`.

#### 2. دور `direction-protector.js` (الحارس الأولي والاحتياطي)

- **المسؤولية:**
  1. قراءة تفضيل اللغة من `localStorage` **فوراً** عند تحميل الصفحة (قبل أن يتصل Blazor بالخادم).
  2. تطبيق سمة `dir` على وسمي `<html>` و `<body>` لضمان أن الرسم الأولي (First Paint) صحيح ولا يظهر وميض.
  3. الاستماع لحدث `storage` لمزامنة الاتجاه إذا تم تغيير اللغة من تبويب متصفح آخر.
- **⚠️ الجديد في v2:** منع الحلقات اللانهائية + التحقق من التغيير الفعلي.

#### 3. دور `login-localization.js` (معزول تماماً)

- **المسؤولية:** إدارة ترجمات صفحات الدخول والتسجيل التقليدية (Static Pages) فقط.
- **القاعدة:** هذا الملف **لا يتدخل** أبداً في صفحات Blazor الداخلية أو في اتجاه القائمة الجانبية.

**💡 الخلاصة المعمارية:**

> **ملف JS يضمن صحة البيئة عند البدء (Initialization)، بينما Blazor يدير الاستمرارية والتفاعل (Interactivity).**
> **⚠️ محدّث (16 أكتوبر):** `html[dir]` الآن المصدر الوحيد للاتجاه. لا `main-wrapper[dir]`.

### 6.6 تحذيرات معمارية صارمة (ممنوع العبث)

| الملف | القاعدة الصارمة | السبب |
|-------|----------------|-------|
| **`MainLayout.razor`** | **ممنوع** إضافة `dir` على `.main-wrapper`. | `html[dir]` هو المصدر الوحيد للاتجاه. |
| **`_sidebar.css`** | **ممنوع** استخدام `.main-wrapper[dir="rtl"]`. | القواعد الحالية تعتمد على `html[dir]` فقط. |
| **`InteractiveMenu.razor`** | **ممنوع** استخدام `Navigation.NavigateTo(url, forceLoad: true)` في عناصر القائمة. | `forceLoad: true` يدمر دائرة Blazor ويمسح الحالة المؤقتة. |
| **`direction-protector.js`** | **ممنوع** إضافة `MutationObserver` عدواني أو فحص دوري (`setInterval`) يغير `dir` بشكل قسري. | يسبب "حرباً" مع Blazor. |

### 6.7 Testing Checklist

**قبل اعتماد أي تحديث يتعلق بالواجهة أو التنقل، يجب اجتياز هذه الاختبارات:**

- [ ] **تسجيل الدخول باللغة العربية:** القائمة يجب أن تكون على اليمين فوراً بدون وميض.
- [ ] **التنقل الداخلي:** الضغط على 3 عناصر مختلفة في القائمة يجب ألا يغير مكان القائمة.
- [ ] **تبديل الأوضاع:** الانتقال من "الشخصي" إلى "مؤسسة" والعكس يجب أن يحافظ على اللغة العربية ومكان القائمة (يمين).
- [ ] **تحديث الصفحة (F5):** يجب أن تستعيد الصفحة اللغة والاتجاه الصحيحين من `localStorage` / Cookie بدون قفزات.
- [ ] **التبديل للإنجليزية:** القائمة يجب أن تنتقل لليسار بسلاسة، والمحتوى لليسار، بدون أي أخطاء.

---

## 📌 الجزء السابع: جدول مقارنة شامل

| العنصر | Web Dashboard | Marketing Site | Mobile |
|--------|---------------|----------------|--------|
| **اللغة الافتراضية** | `en` | `en` | `en` |
| **التخزين** | Cookie + LocalStorage | Cookie + LocalStorage | Preferences |
| **الحقن** | `[Inject] TranslationStateService` | `[Inject] ISharedTranslationState` | Constructor Injection |
| **`BasePage`** | ✅ | ✅ | ❌ (ViewModels) |
| **`T("key")`** | في `BasePage` | في `BasePage` | في ViewModel |
| **تحديث UI** | `StateHasChanged` | `StateHasChanged` | `[ObservableProperty]` |
| **RTL/LTR** | `html[dir]` attribute | `html[dir]` attribute | `I18nManager` |
| **التنظيف** | `Dispose()` | `Dispose()` | `Dispose()` + `OnDisappearing` |
| **الأنماط** | `--rubik-*` | `--hp-*` | `--rubik-*` |
| **الحالة** | 🟢 مستقر | 🟢 مستقر | 🟡 مشاكل متقطعة |

---

## 📌 الجزء الثامن: تحذيرات ومحاذير

### 8.1 🔴 ممنوعات مطلقة

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
| **11** | `dir` على أي عنصر Blazor SSR | `html[dir]` فقط |
| **12** | `main-wrapper[dir="rtl"]` في CSS | `html[dir="rtl"]` |

### 8.2 🟡 تنبيهات مهمة

- **`TranslationStateService` (Web):** Singleton مسجَّل، يُشارك بين كل الجلسات.
- **`MobileTranslationState`:** Singleton — تغيير اللغة يؤثر على التطبيق كله فورًا.
- **Cache:** 30 دقيقة — تعديل DB يحتاج Restart.
- **`PersistentComponentState`:** إذا استُخدم، يجب أن يكون في `BasePage`.
- **`HTML lang` vs `data-rubik-lang`:** يوجد كلاهما في `App.razor` — الأول للاتجاه، الثاني للـ CSS.

---

## 📌 الجزء التاسع: المشاكل المعروفة والحلول

### 9.1 بعض الصفحات تعرض مفاتيح بدلًا من النصوص

**السبب:** `GetPageTranslationsAsync` يُرجع فاضي.
**الحل:** تحقق من `Module` في جدول `Resources`.

### 9.2 ترجمة جزئية

**السبب:** المفاتيح موجودة في `COMMON` وليس في `domain` الصفحة.
**الحل:**

```csharp
var pageTrans = await TranslationService.GetPageTranslationsAsync(PageDomain, lang);
var commonTrans = await TranslationService.GetPageTranslationsAsync("COMMON", lang);
_translations = commonTrans.Concat(pageTrans).ToDictionary(k => k.Key, v => v.Value);
```

### 9.3 تغيير اللغة لا يؤثر على الصفحة الحالية

**السبب:** نسيان `InvokeAsync(StateHasChanged)`.
**الحل:**

```csharp
private async void HandleLanguageChanged()
{
    await LoadTranslations();
    await InvokeAsync(StateHasChanged);  // ⭐ إلزامي
}
```

### 9.4 تسرب الذاكرة

**السبب:** نسيان إلغاء الاشتراك.
**الحل:**

```csharp
public void Dispose()
{
    TranslationState.OnLanguageChanged -= HandleLanguageChanged;
}
```

### 9.5 ⭐ القفزات بين SSR و Interactive (مشكلة محلولة)

**السبب:** `TranslationStateService.CurrentLanguage` **لم تكن تقرأ Cookie فورًا** — كانت تُرجع `"en"` (الافتراضي) → انقلاب.

**الحل:** `CurrentLanguage` **تقرأ Cookie عند كل استدعاء** (انظر 2.4).

**⚠️ هذه كانت مشكلة متجذرة — تأكد من عدم إعادة كسرها.**

### 9.6 زر التبديل لا يعمل

**السبب:** شرط `SetLanguageAsync` يرفض `"ar"`.
**الحل:** تحقق من الشرط `languageCode != "ar" && languageCode != "en"`.

### 9.7 وميض `COMMON.LOADING` في أول زيارة

**السبب:** الصفحة تعرض `T("COMMON.LOADING")` قبل تحميل الترجمات.
**الحل:** Spinner بدون نص (انظر 2.7).

### 9.8 ⚠️ مشكلة Mobile — مفاتيح الترجمة في أحيان كثيرة (مفتوحة)

**الأعراض:**
- ظهور `PSP.FORM.EGP` بدلاً من "جنيه مصري".
- ظهور `MOBILE.LOGIN.EMAIL` بدلاً من "البريد الإلكتروني".
- متقطع — ليس دائمًا.

**السبب المُرجَّح:**
- **توقيت التحميل** — `LoadTranslationsAsync()` قد لا تُكتمل قبل الرسم.
- **التخزين المحلي** — `Preferences` قد لا يُقرأ فورًا.
- **Cache متأخر** — `IMemoryCache` قد يُعيد قيمة فارغة.
- **عدم انتظار API** — `_apiService.GetAsync` قد يفشل صامتًا.

**الحالة:** 🟡 **مؤجّلة** — تحتاج جلسة منفصلة.

**توصيات مبدئية:**
- **`PreloadCommonTranslationsAsync`** في `MauiProgram` — لتحميل الترجمات الشائعة قبل الحاجة.
- **Cache Synchronous** — قراءة من `Preferences` فورًا عند الإقلاع.
- **Fallback**: إن لم يُوجد مفتاح → أظهر مفتاح بديل مفهوم.

---

## 📌 الجزء العاشر: هيكل الملفات الكامل

```
📁 Shared.UI/Services/
├── ITranslationService.cs
└── ITranslationState.cs

📁 RubikCare.Application/Services/
├── TranslationStateService.cs        ⭐ Web + Marketing
├── LayoutDirectionService.cs         ⭐ مشتق من اللغة
├── LocalizationService.cs
└── ...

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

📁 Rubikcare.Web/Controllers/
└── LocalizationController.cs         ⭐ API نقطة النهاية

📁 Rubikcare.Web/wwwroot/Assets/js/
├── direction-protector.js            ⭐ الحارس النشط v2
├── login-localization.js             ⭐ للصفحات الستاتيك
├── accessibility.js
├── observability.js
├── utils.js
└── ...

📁 Rubikcare.Web/wwwroot/js/
├── rubik-menu.js
├── marketing-reveal.js
├── register.js
├── push-notifications.js
└── ...

📁 Mobile/Services/
├── MobileTranslationService.cs
├── MobileTranslationState.cs
└── TranslationCacheService.cs
```

---

## 📌 الجزء الحادي عشر: EF Core Migration — للترجمة

### 11.1 جدول `Resources`

**البنية (من الفحص الفعلي):**

```sql
CREATE TABLE Resources (
    ResourceKey NVARCHAR(200) NOT NULL,
    ResourceValueAr NVARCHAR(MAX) NULL,
    ResourceValueEn NVARCHAR(MAX) NULL,
    Module NVARCHAR(50) NULL,
    ResourceType NVARCHAR(50) NULL,
    IsActive BIT NOT NULL DEFAULT 1,
    CreatedDate DATETIME2 NOT NULL DEFAULT GETDATE(),
    LastModifiedDate DATETIME2 NULL,
    PRIMARY KEY (ResourceKey)
);
```

### 11.2 Indexes

**Migration `AddIndexesOnResourcesTable` (5 سبتمبر 2026):**

- Index على `ResourceKey`.
- Index على `Module`.
- Index على `IsActive`.

### 11.3 Data Seeding

**استخدم `MERGE` دائماً — مع `N''` للعربية.**

---

## 📌 الجزء الثاني عشر: Roadmap

### 12.1 ما هو مطلوب

| # | المهمة | الأولوية | الحالة |
|---|--------|---------|--------|
| 1 | **علاج مشكلة Mobile** (مفاتيح الترجمة) | 🔴 عالية | 🟡 مؤجّلة |
| 2 | **`data-rubik-lang` للـ CSS** | 🟢 منخفضة | ✅ موجود |
| 3 | **Preload Translations في Mobile** | 🟠 متوسطة | ❌ |
| 4 | **Fallback للمفاتيح المفقودة** | 🟠 متوسطة | ❌ |

### 12.2 الاعتمادات

- **مشكلة Mobile** — تحتاج جلسة منفصلة.
- **الباقي** — يمكن تنفيذه تدريجياً.

---

## 📌 الجزء الثالث عشر: سجل التغييرات + روابط

### 13.1 سجل التغييرات

| الإصدار | التاريخ | التغييرات |
|---------|---------|-----------|
| **1.0** | **17 سبتمبر 2026** | الإصدار الأولي — `15-translation-system.md` |
| **1.0** | **15 يوليو 2026** | الإصدار الأولي — `17-sidebar-direction-stability.md` |
| **1.0** | **يوليو 2026** | الإصدار الأولي — `19-web-localization-direction-system.md` |
| **2.0** | **16 أكتوبر 2026** | **`04-TRANSLATION` v2.0** — دمج `15` + `17` + `19` + الكود الفعلي + ملاحظات Mobile |

### 13.2 روابط ذات صلة

- [00 — MASTER-INDEX](./00-MASTER-INDEX.md)
- [00B — PROJECT-BACKLOG](./00B-PROJECT-BACKLOG.md)
- [01 — ARCHITECTURE](./01-ARCHITECTURE.md)
- [02 — IDENTITY](./02-IDENTITY.md)
- [03 — MENUS](./03-MENUS.md)
- [05 — PAGE-CREATION](./11-PAGE-CREATION.md)
- [09 — API-GUIDE](./09-API-GUIDE.md)
- [14 — CACHING](./12-CACHING.md)

---

**© 2026 RubikCare — للاستخدام الداخلي**

**نهاية الوثيقة ✅**
