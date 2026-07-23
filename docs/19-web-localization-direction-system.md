## وثيقة: نظام الترجمة والاتجاه في تطبيق الويب

**المسار المقترح:** `docs/17-web-localization-direction-system.md`

---

### المشكلة الجذرية: فجوة SSR ↔ Interactive

Blazor يعمل في مرحلتين منفصلتين لكل طلب:

```
المرحلة 1 — SSR (Server Side Rendering)
├── يحدث على الخادم قبل إرسال HTML للمتصفح
├── HttpContext متاح → يمكن قراءة Cookies
├── JavaScript غير متاح → لا يمكن قراءة localStorage
├── TranslationStateService ينشأ (Scoped جديد)
└── OnInitializedAsync تُستدعى هنا

المرحلة 2 — Interactive (Blazor Server via SignalR)
├── يحدث بعد اتصال WebSocket
├── HttpContext غير متاح → Cookies لا تُقرأ
├── JavaScript متاح → يمكن قراءة localStorage
├── TranslationStateService ينشأ من جديد (Scoped جديد مختلف)
└── OnAfterRenderAsync تُستدعى هنا فقط
```

**النتيجة:** `TranslationStateService` في المرحلة الثانية يبدأ دائماً بـ `_currentLanguage = null` → يرجع الافتراضي `"ar"` → تظهر الصفحة بالعربية حتى لو المستخدم اختار الإنجليزية.

---

### الحل المطبّق: طبقات ثلاث متكاملة

```
الطبقة 1: السكربت المبكر في <head> (App.razor)
    → يعمل قبل Blazor بالكامل
    → يقرأ Cookie أو localStorage
    → يضبط html[dir] و html[lang] فوراً
    → يمنع الوميض البصري

الطبقة 2: TranslationStateService.InitializeAsync
    → تُستدعى في SSR
    → تقرأ Cookie من HttpContext
    → تضبط _currentLanguage للـ SSR render

الطبقة 3: InteractiveMenu.OnAfterRenderAsync
    → تُستدعى بعد اكتمال WebSocket
    → تستدعي SyncFromLocalStorageAsync
    → تقرأ localStorage وتصحح _currentLanguage
    → تُطلق OnLanguageChanged لتحديث كل المكونات
```

---

### تدفق البيانات الكامل

```
المستخدم يختار لغة
    ↓
WebTranslationState.SetLanguage(lang)
    ↓
TranslationStateService.SetLanguageAsync(lang)
    ├── localStorage.setItem("RubikCare:Language", lang)
    ├── document.cookie = ".RubikCare.Language=..."
    └── OnLanguageChanged?.Invoke()

عند Refresh
    ↓
السكربت المبكر في <head>
    ├── يقرأ .RubikCare.Language من Cookie
    └── يضبط html[dir] فوراً (بدون انتظار Blazor)
    ↓
SSR: TranslationStateService.InitializeAsync
    └── يقرأ Cookie من HttpContext → _currentLanguage = "en"
    ↓
Interactive: InteractiveMenu.OnAfterRenderAsync
    └── SyncFromLocalStorageAsync → يقرأ localStorage → يصحح إذا اختلف
```

---

### الملفات وأدوارها

| الملف | الدور |
|-------|-------|
| `App.razor` — `<head>` script | يضبط `html[dir]` فوراً قبل Blazor |
| `TranslationStateService.cs` — `InitializeAsync` | يقرأ Cookie في SSR |
| `TranslationStateService.cs` — `SyncFromLocalStorageAsync` | يقرأ localStorage في Interactive |
| `TranslationStateService.cs` — `SetLanguageAsync` | يحفظ في localStorage + Cookie + يُطلق الحدث |
| `InteractiveMenu.razor` — `OnAfterRenderAsync` | يستدعي Sync بعد اكتمال WebSocket |
| `WebTranslationState.cs` — `SetLanguage` | واجهة للمكونات، تستدعي `applyRubikDirection()` |
| `direction-protector.js` | يراقب DOM ويمنع Blazor من تغيير `html[dir]` خطأً |
| `_sidebar.css` | يقرأ `html[dir]` فقط لتحديد موضع القائمة |

---

### قواعد إلزامية

**ممنوعات:**
- لا تضع `dir` على `main-wrapper` أو أي عنصر Blazor — الـ CSS يقرأ `html[dir]` فقط
- لا تستدعي JS في `OnInitializedAsync` — JS غير متاح في SSR
- لا تستدعي `SyncFromLocalStorageAsync` في مكون SSR — تحتاج Interactive

**إلزاميات:**
- المكون الذي يستدعي `SyncFromLocalStorageAsync` يجب أن يكون `@rendermode InteractiveServer`
- اسم الـ Cookie يجب أن يطابق بالضبط: `.RubikCare.Language`
- قيمة الـ Cookie بالصيغة: `c=en|uic=en` (مُشفّرة بـ URL)
- اسم localStorage: `RubikCare:Language`

---

### تشخيص المشاكل

| العرض | السبب | الحل |
|-------|-------|-------|
| القائمة في مكان خاطئ عند الفتح | `html[dir]` لم يُضبط قبل Blazor | تأكد من وجود السكربت المبكر في `<head>` |
| اللغة تتغير للافتراضي بعد Refresh | `SyncFromLocalStorageAsync` لا تُستدعى في Interactive | تأكد من استدعائها في `OnAfterRenderAsync` لمكون Interactive |
| تغيير اللغة لا يحرك القائمة | `_sidebar.css` يعتمد على شيء غير `html[dir]` | راجع CSS وتأكد أنه يقرأ `html[dir]` فقط |
| الـ Cookie لا يُقرأ في SSR | اسم الـ Cookie مختلف | تحقق من الاسم بالضبط في Console: `document.cookie` |
