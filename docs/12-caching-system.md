
# 12 — CACHING (نظام الكاش الموحد)

**الإصدار:** 2.0 (Unified Edition)
**آخر تحديث:** 17 أكتوبر 2026
**الحالة:** ✅ مرجع شامل — يدمج 14 + الكود الفعلي
**الجمهور:** فريق التطوير + AI Assistants
**المرجع الإلزامي:** يُقرأ قبل أي تعديل على الكاش

---

## 📌 الجزء صفر: الفلسفة والرؤية

### 0.1 المبدأ الأساسي

> **"كل نوع من البيانات له مصدر كاش واحد فقط. لا تكرار، لا ازدواجية."**

### 0.2 لماذا هذا المرجع؟

نظام الكاش في RubikCare هو أحد أكثر الأجزاء حساسية في المشروع. أي خطأ في تصميمه يؤدي إلى:

- **ظهور بيانات المستخدم السابق** بعد تسجيل الخروج.
- **تضارب البيانات** بين الجلسات المختلفة.
- **استهلاك مفرط للذاكرة**.
- **صعوبة في الصيانة والتطوير**.

### 0.3 قبل / بعد الإصلاح

```
❌ قبل الإصلاح:
UserSessionService ← كاش 1
DynamicMenuService ← كاش 2
CachedUserSessionService ← كاش 3 (الموبايل)
SecureStorage ← تخزين 4
LocalizationService ← IMemoryCache + LocalizationCacheService (ازدواجية)

✅ بعد الإصلاح:
UserSessionService ← الكاش الوحيد للبيانات الحساسة (الخادم)
CachedUserSessionService ← كاش محلي (الموبايل)
SecureStorage ← تخزين التوكن فقط
LocalizationService (Web) ← IMemoryCache القياسي فقط للترجمات
TranslationCacheService (Mobile) ← يدمج IMemoryCache مع Preferences
```

---

## 📌 الجزء الأول: طبقات الكاش

### 1.1 نظرة عامة

| الطبقة | المسؤول | الموقع | المفاتيح | المدة |
|--------|---------|--------|----------|-------|
| **الجلسة** | `UserSessionService` | الخادم | `UserSession_{userId}` | ساعتين |
| **التفضيلات** | `UserSessionService` | الخادم | `UserPrefs_{userId}` | ساعة |
| **المعلومات الأساسية** | `UserSessionService` | الخادم | `UserBasic_{userId}` | ساعة |
| **ترجمات الويب** | `LocalizationService` | الخادم | `translation_{lang}_{key}` | 30 دقيقة |
| **ترجمات الموبايل** | `TranslationCacheService` | الموبايل | `translations_all_{lang}` | دائم (حتى Logout) |
| **كاش الموبايل** | `CachedUserSessionService` | الموبايل | `_cachedSession` | ساعتين |

### 1.2 🌐 كاش الترجمات — استثناء مُوثَّق

**لتجنب ازدواجية الكاش وضمان الأداء الأمثل:**

#### في تطبيق الويب (Web)

- ✅ **الصحيح:** `LocalizationService` يستخدم `IMemoryCache` القياسي مباشرة.
- ❌ **ممنوع:** استخدام `LocalizationCacheService` (مبني على `Dictionary` + `SemaphoreSlim`).

**⚠️ السبب:** يسبب ازدواجية غير مبررة في استهلاك الذاكرة وتعقيداً في منطق الكاش.

#### في تطبيق الموبايل (Mobile)

- ✅ **الصحيح:** `TranslationCacheService` يجمع بين `IMemoryCache` (للوصول السريع) + `Preferences` (للحفظ الدائم).

**⚠️ لماذا هذا الاستثناء؟** طبيعة دورة حياة تطبيق الموبايل (App Lifecycle) تتطلب الوصول للترجمات حتى دون اتصال بالإنترنت — وهو ما لا ينطبق على تطبيق الويب الذي يعتمد على الخادم بشكل دائم.

### 1.3 الكود الفعلي: `LocalizationService` — Cache Keys

**⚠️ من الفحص الفعلي:**

```csharp
private readonly TimeSpan _cacheDuration = TimeSpan.FromMinutes(30);
private static readonly string _cacheVersion = "v2";  // ⭐ غيّرها مع كل نشر

// مفاتيح الكاش الفعلية:
string cacheKey = $"translation_{language}_{resourceKey}";
string cacheKey = $"all_translations_{language}";
string cacheKey = $"page_translations_{_cacheVersion}_{language}_{pageDomain}";
```

**⚠️ ميزة `_cacheVersion`:** تغييرها يُلغي الكاش القديم تلقائياً — **يُستخدم مع كل نشر**.

---

## 📌 الجزء الثاني: هيكل النظام

### 2.1 `UserSessionService` (الخادم — المصدر الرئيسي)

**المسار:** `RubikCare.Application/Services/Session/UserSessionService.cs`

**المسؤوليات:**
- تخزين بيانات الجلسة الكاملة (`UserSessionData`).
- تخزين تفضيلات المستخدم (`UserPreferences`).
- تخزين المعلومات الأساسية (`UserBasicInfo`).
- إدارة انتهاء الصلاحية (TTL).
- مسح الكاش عند Logout.

**مثال على الاستخدام:**

```csharp
// ✅ صحيح - الحصول على بيانات الجلسة
var session = await _userSessionService.GetUserSessionAsync(user);

// ✅ صحيح - تحديث تفضيلات المستخدم
await _userSessionService.UpdateSessionLanguageAsync(userId, "ar");

// ✅ صحيح - مسح الكاش عند Logout
await _userSessionService.RefreshUserSessionAndCacheAsync(userId);
```

### 2.2 `CachedUserSessionService` (الموبايل — كاش محلي)

**المسار:** `RubikCare.Mobile/Infrastructure/Services/CachedUserSessionService.cs`

**المسؤوليات:**
- تخزين الجلسة محلياً في الذاكرة (`_cachedSession`).
- التحقق من صلاحية الكاش (2 ساعة).
- تحديث الكاش من API عند الحاجة.
- مسح الكاش عند Logout.

**مثال:**

```csharp
// ✅ صحيح - الحصول على الجلسة (من الكاش أولاً)
var session = await _cachedSessionService.GetSessionAsync();

// ✅ صحيح - التحقق من الصلاحية
var isValid = await _cachedSessionService.IsSessionValidAsync();

// ✅ صحيح - مسح الكاش عند Logout
await _cachedSessionService.ClearSessionAsync();
```

### 2.3 `LocalizationService` (الويب — كاش الترجمات)

**المسار:** `RubikCare.Application/Services/LocalizationService.cs`

**المسؤوليات:**
- تخزين الترجمات في `IMemoryCache` القياسي.
- إدارة انتهاء الصلاحية (30 دقيقة).
- جلب الترجمات من قاعدة البيانات عند الحاجة.

**مثال:**

```csharp
// ✅ صحيح - الحصول على ترجمة
var text = await _localizationService.GetTranslationAsync("COMMON.WELCOME", "ar");

// ✅ صحيح - الحصول على ترجمات صفحة كاملة
var translations = await _localizationService.GetPageTranslationsAsync("SHARED.DASHBOARD", "ar");
```

### 2.4 `TranslationCacheService` (الموبايل — كاش الترجمات)

**المسار:** `RubikCare.Mobile/Services/TranslationCacheService.cs`

**المسؤوليات:**
- تخزين الترجمات في `IMemoryCache` (للوصول السريع).
- تخزين الترجمات في `Preferences` (للحفظ الدائم).
- إدارة انتهاء الصلاحية (ساعة للذاكرة، دائم لـ `Preferences`).
- مسح الكاش عند تغيير اللغة.

**مثال:**

```csharp
// ✅ صحيح - الحصول على الترجمات من الكاش
var translations = await _cacheService.GetCachedAsync("ar");

// ✅ صحيح - حفظ الترجمات في الكاش
_cacheService.SetCache("ar", translations);

// ✅ صحيح - مسح كاش لغة معينة
_cacheService.ClearCache("ar");
```

---

## 📌 الجزء الثالث: 🔴 القواعد الإلزامية

### 3.1 ممنوعات

1. **لا تستخدم `IMemoryCache` للبيانات الحساسة** (مثل الجلسات أو القوائم) **خارج `UserSessionService`.**
   *(الاستثناء الوحيد المسموح: `IMemoryCache` القياسي داخل `LocalizationService` للترجمات).*

2. **لا تنشئ نظام كاش مخصص (Dictionary + SemaphoreSlim)** إذا كان `IMemoryCache` يكفي.

3. **لا تخزن بيانات المستخدم في `Static` fields** — يسبب تسرب البيانات.

4. **لا تنسَ مسح الكاش عند Logout** — على الخادم + العميل.

5. **لا تستخدم `IMemoryCache` للموبايل** — استخدم `Preferences` أو `SecureStorage`.

### 3.2 إلزاميات

1. **استخدم `UserSessionService` كمصدر وحيد لبيانات الجلسة على الخادم.**

2. **استخدم `CachedUserSessionService` في الموبايل للوصول السريع للجلسة.**

3. **استخدم `IMemoryCache` القياسي في `LocalizationService` على الخادم.**

4. **استخدم `TranslationCacheService` في الموبايل لإدارة كاش الترجمات.**

5. **حدد مدة صلاحية (TTL) واضحة** لكل نوع من البيانات.

6. **امسح الكاش عند أي تغيير في بيانات المستخدم.**

---

## 📌 الجزء الرابع: دورة حياة الكاش

### 4.1 عند تسجيل الدخول

```
1. المصادقة عبر Identity
2. إنشاء UserSessionData
3. حفظ في UserSessionService (الخادم)
4. حفظ في CachedUserSessionService (الموبايل)
5. تحميل الترجمات وحفظها في TranslationCacheService (الموبايل)
```

### 4.2 أثناء الاستخدام

```
1. التحقق من صلاحية الكاش
2. إذا صالح → إرجاع البيانات من الكاش
3. إذا منتهي → تحديث من قاعدة البيانات / API
4. حفظ البيانات المحدثة في الكاش
```

### 4.3 عند تسجيل الخروج

```
1. استدعاء ClearUserCacheUseCase (الخادم)
   ├── UserSessionService.RefreshUserSessionAndCacheAsync(userId)
   ├── IUserMenuService.ClearCacheAsync(userId)
   └── مسح جميع المفاتيح المرتبطة بـ userId

2. في الموبايل:
   ├── CachedUserSessionService.ClearSessionAsync()
   ├── TranslationCacheService.ClearAllCache()
   └── SecureStorage.RemoveAll()
```

---

## 📌 الجزء الخامس: أمثلة عملية

### 5.1 مثال 1: الحصول على بيانات المستخدم في Dashboard

```csharp
// ✅ صحيح - في Web
@inject IUserSessionService UserSessionService

@code {
    private UserSessionData? _session;
    
    protected override async Task OnInitializedAsync()
    {
        var authState = await AuthProvider.GetAuthenticationStateAsync();
        _session = await UserSessionService.GetUserSessionAsync(authState.User);
    }
}
```

```csharp
// ✅ صحيح - في Mobile
@inject ICachedUserSessionService CachedSessionService

@code {
    private UserSessionData? _session;
    
    protected override async Task OnInitializedAsync()
    {
        _session = await CachedSessionService.GetSessionAsync();
    }
}
```

### 5.2 مثال 2: الحصول على الترجمات

```csharp
// ✅ صحيح - في Web
@inject ILocalizationService LocalizationService

@code {
    private string _welcomeText = "";
    
    protected override async Task OnInitializedAsync()
    {
        _welcomeText = await LocalizationService.GetTranslationAsync(
            "COMMON.WELCOME", "ar");
    }
}
```

```csharp
// ✅ صحيح - في Mobile
@inject IMobileTranslationService TranslationService

@code {
    private Dictionary<string, string> _translations = new();
    
    protected override async Task OnInitializedAsync()
    {
        _translations = await TranslationService.GetPageTranslationsAsync(
            "SHARED.DASHBOARD");
    }
}
```

### 5.3 مثال 3: مسح الكاش عند Logout

```csharp
// ✅ صحيح - في Web (AuthController)
[HttpPost("logout")]
[Authorize]
public async Task<IActionResult> Logout([FromServices] ClearUserCacheUseCase clearCache)
{
    var userId = User.FindFirst("userId")?.Value;
    if (!string.IsNullOrEmpty(userId))
        await clearCache.ExecuteAsync(userId);
    
    await _signInManager.SignOutAsync();
    return Ok(new { success = true });
}
```

```csharp
// ✅ صحيح - في Mobile (LoginPage)
private async Task LogoutAsync()
{
    // مسح الكاش المحلي
    await _cachedSessionService.ClearSessionAsync();
    _translationCacheService.ClearAllCache();
    
    // مسح SecureStorage
    SecureStorage.RemoveAll();
    
    // استدعاء API Logout
    await _apiService.PostAsync("api/auth/logout", new { });
    
    // الانتقال لصفحة الدخول
    await Shell.Current.GoToAsync("//LoginPage");
}
```

---

## 📌 الجزء السادس: CHECKLIST

### 6.1 عند إنشاء خدمة جديدة

- [ ] هل حددت مصدر الكاش الصحيح (`UserSessionService` / `CachedUserSessionService`)؟
- [ ] هل حددت مدة صلاحية (TTL) مناسبة؟
- [ ] هل أضفت منطق التحقق من صلاحية الكاش؟
- [ ] هل أضفت منطق تحديث الكاش عند انتهاء الصلاحية؟
- [ ] هل أضفت منطق مسح الكاش عند Logout؟

### 6.2 عند التعامل مع بيانات المستخدم

- [ ] هل استخدمت `UserSessionService` (الخادم) أو `CachedUserSessionService` (الموبايل)؟
- [ ] هل تجنبت تخزين البيانات الحساسة في `Static` fields؟
- [ ] هل مسحت الكاش عند أي تغيير في بيانات المستخدم؟

### 6.3 عند التعامل مع الترجمات

- [ ] هل استخدمت `LocalizationService` مع `IMemoryCache` في الويب؟
- [ ] هل استخدمت `TranslationCacheService` في الموبايل؟
- [ ] ⭐ هل تم إهمال `LocalizationCacheService` في الويب والاعتماد حصرياً على `IMemoryCache` داخل `LocalizationService`؟
- [ ] هل مسحت كاش الترجمات عند تغيير اللغة؟

### 6.4 عند تسجيل الخروج

- [ ] هل استدعيت `ClearUserCacheUseCase` على الخادم؟
- [ ] هل مسحت `CachedUserSessionService` في الموبايل؟
- [ ] هل مسحت `TranslationCacheService` في الموبايل؟
- [ ] هل مسحت `SecureStorage` في الموبايل؟

---

## 📌 الجزء السابع: ⚠️ تحذيرات ومحاذير

| # | التحذير |
|---|---------|
| 1 | لا تنشئ `IMemoryCache` جديد في خدماتك - استخدم الموجود المسجل في `Program.cs` |
| 2 | لا تستخدم `Static` fields لتخزين بيانات المستخدم - يسبب تسرب البيانات |
| 3 | لا تنسَ مسح الكاش عند Logout - على الخادم والعميل |
| 4 | حدد TTL مناسب لكل نوع بيانات (جلسات: ساعتين، ترجمات: 30 دقيقة) |
| 5 | استخدم `IMemoryCache` القياسي في الويب - لا تنشئ أنظمة مخصصة |
| 6 | في الموبايل، استخدم `Preferences` للبيانات الدائمة و `IMemoryCache` للبيانات المؤقتة |
| 7 | امسح كاش الترجمات عند تغيير اللغة لتجنب عرض ترجمات قديمة |
| 8 | لا تستخدم `LocalizationCacheService` (المبني على Dictionary) - استخدم `IMemoryCache` مباشرة |
| 9 | ⭐ **`_cacheVersion`** — غيّرها مع كل نشر (لإلغاء الكاش القديم تلقائياً) |
| 10 | ⭐ **`UserSessionService`** — مصدر وحيد للبيانات الحساسة (لا تكرار) |

---

## 📌 الجزء الثامن: العلاقة بالوثائق الأخرى

### 8.1 `01-ARCHITECTURE`

**⚠️ التسجيلات في `Program.cs`:**

```csharp
builder.Services.AddMemoryCache();  // ⭐ ضروري جداً
builder.Services.AddScoped<UserContextService>();
builder.Services.AddScoped<IUserSessionService, UserSessionService>();
builder.Services.AddScoped<LocalizationService>();
builder.Services.AddScoped<ILocalizationService, LocalizationService>();
```

### 8.2 `02-IDENTITY`

**⚠️ `ClearUserCacheUseCase`** — يُستدعى في `AuthController.Logout`.

**⚠️ التفاصيل في `02-IDENTITY`.**

### 8.3 `04-TRANSLATION`

**⚠️ `TranslationStateService`** — يستخدم Cookie + LocalStorage.

**⚠️ `LocalizationService`** — يستخدم `IMemoryCache` (هذه الوثيقة).

**⚠️ التفاصيل في `04-TRANSLATION`.**

### 8.4 `03-MENUS`

**⚠️ `DynamicMenuService`** — **لا يستخدم `IMemoryCache`** — تمت إزالته.

**⚠️ الكاش المركزي في `UserSessionService`.**

**⚠️ التفاصيل في `03-MENUS`.**

---

## 📌 الجزء التاسع: روابط ذات صلة

- [00 — MASTER-INDEX](./00-MASTER-INDEX.md)
- [00B — PROJECT-BACKLOG](./00B-PROJECT-BACKLOG.md)
- [01 — ARCHITECTURE](./01-ARCHITECTURE.md)
- [02 — IDENTITY](./02-IDENTITY.md)
- [03 — MENUS](./03-MENUS.md)
- [04 — TRANSLATION](./04-TRANSLATION.md)
- [11 — PAGE-CREATION](./11-PAGE-CREATION.md)

---

## 📌 الجزء العاشر: سجل التحديثات

| التاريخ | الإصدار | التغييرات |
|---------|---------|-----------|
| 24 مايو 2026 | 1.0 | الإصدار الأولي — توحيد نظام الكاش |
| 14 يوليو 2026 | 1.2 | إضافة قسم "كاش الترجمات"، توضيح استثناء `IMemoryCache` للترجمات، إهمال `LocalizationCacheService` |
| **17 أكتوبر 2026** | **2.0** | **`12-CACHING` v2.0** — دمج `14` v1.2 + الكود الفعلي (`_cacheVersion` + Cache Keys) + إشارات `01` + `02` + `03` + `04` |

---

**© 2026 RubikCare — للاستخدام الداخلي**

**نهاية الوثيقة ✅**
```

---
**⏳ بانتظار قرارك.** 🚀
