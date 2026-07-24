# 19- ظام الترجمة والاتجاهات في تطبيق الويب (Web Localization & Direction)

**آخر تحديث: يوليو 2026**

---

## مقدمة: لماذا كانت المشكلة صعبة؟

Blazor Server يعمل في **مرحلتين منفصلتين** لكل طلب صفحة:

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

**المشكلة الجذرية:** `TranslationStateService` هو `Scoped`، يعني في كل مرحلة ينشئ Blazor **instance جديد** يبدأ من صفر بـ `_currentLanguage = null` (= "ar" افتراضياً).

---

## التسلسل الزمني الكامل لما يحدث عند Refresh

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
   → _currentLanguage = null = "ar" افتراضياً
   → App.razor.OnInitializedAsync → InitializeAsync()
   → httpContext = null (لأننا في Interactive mode)
   → يرجع للافتراضي "ar" ❌
   → المكونات تُعاد رسمها بـ lang=ar ❌

5. بعد اكتمال الاتصال:
   → InteractiveMenu.OnAfterRenderAsync يُنفَّذ
   → SyncFromLocalStorageAsync() تقرأ localStorage = "en"
   → تُطلق OnLanguageChanged
   → جميع المكونات تُعاد رسمها بـ lang=en ✅
   → لكن هذا يحدث بعد لحظة من الرسم الخاطئ (flash مرئي)
```

---

## مخطط الملفات المتداخلة

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

---

## الحل الكامل المطبق

### الطبقة 1: السكربت المبكر في `App.razor` (أسرع طبقة)

يعمل **قبل** أي JavaScript آخر، يضبط `html[dir]` فوراً:

```html
<!-- في <head> مباشرة قبل HeadOutlet -->
<script>
    (function() {
        try {
            var lang = 'ar';
            // 1. محاولة القراءة من الكوكيز أولاً
            var cookieMatch = document.cookie.match(
                /(?:^|;\s*)\.RubikCare\.Language=([^;]*)/);
            if (cookieMatch) {
                var decoded = decodeURIComponent(cookieMatch[1]);
                var langMatch = decoded.match(/c=([a-z]{2})/);
                if (langMatch) lang = langMatch[1];
            } else {
                // 2. القراءة من localStorage
                var stored = localStorage.getItem('RubikCare:Language');
                if (stored === 'ar' || stored === 'en') lang = stored;
            }
            // 3. تطبيق الاتجاه فوراً
            var dir = lang === 'ar' ? 'rtl' : 'ltr';
            document.documentElement.setAttribute('dir', dir);
            document.documentElement.setAttribute('lang', lang);
            document.body.setAttribute('dir', dir);
        } catch (e) {
            document.documentElement.setAttribute('dir', 'rtl');
            document.documentElement.setAttribute('lang', 'ar');
        }
    })();
</script>
```

**ما يحله:** يضبط `html[dir]` الصحيح قبل رسم أي محتوى — يمنع الـ flash الأولي.

---

### الطبقة 2: `direction-protector.js` (الحارس النشط)

يراقب أي تغيير على `html[dir]` ويُعيده للقيمة الصحيحة:

```javascript
const observer = new MutationObserver((mutations) => {
    const correctLang = getCurrentLanguage(); // من localStorage
    const correctDir = getDirection(correctLang);

    mutations.forEach((mutation) => {
        if (mutation.type === 'attributes' && mutation.attributeName === 'dir') {
            const currentDir = mutation.target.getAttribute('dir');
            if (currentDir !== correctDir) {
                mutation.target.setAttribute('dir', correctDir);
            }
        }
    });
});

observer.observe(document.documentElement, { attributes: true, attributeFilter: ['dir'] });
observer.observe(document.body, { attributes: true, attributeFilter: ['dir'] });
```

**ما يحله:** يمنع Blazor من تغيير `html[dir]` عند إعادة الرسم.

---

### الطبقة 3: `TranslationStateService.InitializeAsync` (SSR)

تقرأ الـ cookie في مرحلة SSR فقط (حيث `HttpContext` متاح):

```csharp
public async Task InitializeAsync()
{
    if (_isInitialized) return;

    try
    {
        string detectedLang = "ar";

        var httpContext = _httpContextAccessor?.HttpContext;
        if (httpContext != null) // SSR فقط
        {
            var cookieValue = httpContext.Request.Cookies[".RubikCare.Language"];
            if (string.IsNullOrEmpty(cookieValue))
                cookieValue = httpContext.Request.Cookies["RubikCare.Language"];

            if (!string.IsNullOrEmpty(cookieValue))
            {
                var decoded = Uri.UnescapeDataString(cookieValue);
                var match = Regex.Match(decoded, @"c=([a-z]{2})");
                if (match.Success &&
                    (match.Groups[1].Value == "en" || match.Groups[1].Value == "ar"))
                    detectedLang = match.Groups[1].Value;
            }
        }
        // ⚠️ لا تحاول قراءة JS هنا — لا تعمل في OnInitializedAsync

        _currentLanguage = detectedLang;
    }
    finally
    {
        _isInitialized = true;
    }
}
```

**ما يحله:** يضمن أن الـ HTML المرسل من SSR يحمل اللغة الصحيحة.

---

### الطبقة 4: `SyncFromLocalStorageAsync` في `InteractiveMenu` (Interactive) ⭐

هذا هو **المفتاح الأساسي** لحل مشكلة Refresh:

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

**لماذا `InteractiveMenu` وليس `MainLayout` أو `App.razor`؟**

| المكون | الـ RenderMode | OnAfterRenderAsync يعمل؟ |
|--------|---------------|--------------------------|
| `App.razor` | SSR (لا rendermode) | ❌ لا |
| `MainLayout.razor` | SSR (لا rendermode) | ❌ لا |
| `InteractiveMenu.razor` | `@rendermode InteractiveServer` | ✅ نعم |

`OnAfterRenderAsync` لا تُنفَّذ إلا في مكونات Interactive — لذا `InteractiveMenu` هو المكان الوحيد الصحيح.

---

### `SyncFromLocalStorageAsync` في `TranslationStateService`

```csharp
public async Task SyncFromLocalStorageAsync()
{
    try
    {
        var storedLang = await _jsRuntime.InvokeAsync<string>(
            "localStorage.getItem", "RubikCare:Language");

        if (storedLang == "en" || storedLang == "ar")
        {
            if (_currentLanguage != storedLang)
            {
                _currentLanguage = storedLang;
                OnLanguageChanged?.Invoke(); // يُحدّث جميع المكونات المشتركة
            }
        }
    }
    catch (Exception ex)
    {
        Console.WriteLine($"❌ SyncFromLocalStorageAsync failed: {ex.Message}");
    }
}
```

---

### CSS: يعتمد على `html[dir]` فقط — لا `main-wrapper[dir]`

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

**السبب:** `html[dir]` يُضبط بـ JavaScript (السكربت المبكر + direction-protector) قبل Blazor، بينما `main-wrapper[dir]` يعتمد على Blazor الذي يبدأ بـ "ar" دائماً.

وعليه يجب **حذف** `dir` من `main-wrapper` في `MainLayout.razor`:

```razor
<!-- ❌ قبل الإصلاح -->
<div class="main-wrapper" dir="@(TranslationState.CurrentLanguage == "ar" ? "rtl" : "ltr")">

<!-- ✅ بعد الإصلاح -->
<div class="main-wrapper">
```

---

## تدفق تغيير اللغة يدوياً

عندما يضغط المستخدم على زر تغيير اللغة في `LanguageSection.razor`:

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

---

## جدول الملفات المتأثرة

| الملف | الدور | ملاحظة |
|-------|-------|--------|
| `App.razor` | السكربت المبكر في `<head>` | يضبط `html[dir]` قبل كل شيء |
| `direction-protector.js` | الحارس النشط | يراقب `html[dir]` ويمنع التغيير الخاطئ |
| `TranslationStateService.cs` | مصدر الحقيقة للغة في C# | `InitializeAsync` للـ SSR، `SyncFromLocalStorageAsync` للـ Interactive |
| `WebTranslationState.cs` | وسيط بين Blazor و TranslationStateService | يستدعي `applyRubikDirection()` عند التغيير |
| `InteractiveMenu.razor` | نقطة التصحيح الأولى في Interactive | يستدعي `SyncFromLocalStorageAsync` في `OnAfterRenderAsync` |
| `MainLayout.razor` | لا يضع `dir` على `main-wrapper` | يعتمد على `html[dir]` فقط |
| `_sidebar.css` | CSS يقرأ من `html[dir]` فقط | لا يعتمد على `.main-wrapper[dir]` |

---

## قواعد إلزامية (لا تكسرها)

1. **لا تضع `dir` على أي عنصر Blazor SSR** — فقط على `<html>` عبر JavaScript.

2. **`SyncFromLocalStorageAsync` تُستدعى فقط في مكونات `@rendermode InteractiveServer`** — لا تستدعها في `App.razor` أو `MainLayout.razor`.

3. **`InitializeAsync` لا تستدعي JavaScript** — تعمل في SSR حيث JS غير متاح.

4. **CSS يقرأ `html[dir]` فقط** — لا `.main-wrapper[dir]` أو أي عنصر Blazor آخر.

5. **اسم الـ cookie الصحيح هو `.RubikCare.Language`** (بنقطة في البداية) — تأكد من تطابقه في جميع الملفات.

6. **localStorage هو المصدر الوحيد الموثوق في Interactive mode** — الـ cookie يُستخدم في SSR فقط.

---

## CHECKLIST: عند إضافة مكون جديد يحتاج الترجمة

- [ ] هل المكون يستخدم `TranslationState.CurrentLanguage` لقراءة اللغة؟
- [ ] هل اشترك في `TranslationState.OnLanguageChanged`؟
- [ ] هل يُلغي الاشتراك في `Dispose()`؟
- [ ] إذا كان المكون `@rendermode InteractiveServer` ويحتاج تصحيح اللغة فوراً — هل يستدعي `SyncFromLocalStorageAsync`؟ (في الغالب يكفي الاشتراك في `OnLanguageChanged`)

---

## CHECKLIST: عند إضافة صفحة Layout جديدة

- [ ] هل تجنبت وضع `dir` على الـ wrapper الرئيسي؟
- [ ] هل CSS يعتمد على `html[dir]` فقط؟
- [ ] إذا كانت الصفحة تحتوي على مكون Interactive — هل يستدعي ذلك المكون `SyncFromLocalStorageAsync` في `OnAfterRenderAsync`؟

---

## سجل المشاكل المحلولة

| التاريخ | المشكلة | الحل |
|---------|---------|------|
| يوليو 2026 | القائمة في اليسار دائماً رغم اللغة العربية | CSS انتقل من `.main-wrapper[dir]` إلى `html[dir]` |
| يوليو 2026 | اللغة تتحول للعربية عند Refresh | `SyncFromLocalStorageAsync` في `InteractiveMenu.OnAfterRenderAsync` |
| يوليو 2026 | الصفحة الأولى تفتح بالعربية رغم localStorage=en | السكربت المبكر في `App.razor` يضبط `html[dir]` قبل Blazor |
