# 14 - نظام الكاش الموحد (Unified Caching System)

**آخر تحديث:** 14 يوليو 2026  
**الإصدار:** 1.2

---

## مقدمة

نظام الكاش في RubikCare هو أحد أكثر الأجزاء حساسية في المشروع. أي خطأ في تصميمه يؤدي إلى:
- ظهور بيانات المستخدم السابق بعد تسجيل الخروج
- تضارب البيانات بين الجلسات المختلفة
- استهلاك مفرط للذاكرة
- صعوبة في الصيانة والتطوير

هذا المرجع يوثق الفلسفة، البنية، والقواعد الصارمة لنظام الكاش الموحد.

---

## 🎯 فلسفة النظام: مصدر واحد للكاش

### المبدأ الأساسي

```text
قبل الإصلاح (❌):
UserSessionService ← كاش 1
DynamicMenuService ← كاش 2
CachedUserSessionService ← كاش 3 (الموبايل)
SecureStorage ← تخزين 4
LocalizationService ← يستخدم IMemoryCache + LocalizationCacheService (ازدواجية غير مبررة)

بعد الإصلاح (✅):
UserSessionService ← الكاش الوحيد للبيانات الحساسة (الخادم)
CachedUserSessionService ← كاش محلي (الموبايل)
SecureStorage ← تخزين التوكن فقط
LocalizationService (Web) ← يستخدم IMemoryCache القياسي فقط للترجمات
TranslationCacheService (Mobile) ← يدمج IMemoryCache مع Preferences للترجمات
```

### القاعدة الذهبية

> **"كل نوع من البيانات له مصدر كاش واحد فقط. لا تكرار، لا ازدواجية."**

---

## 🌐 كاش الترجمات (Localization Caching)

لتجنب ازدواجية الكاش وضمان الأداء الأمثل، يتم التعامل مع ترجمات النظام كالتالي بناءً على بيئة التشغيل:

### في تطبيق الويب (Web):
- ✅ **الصحيح:** `LocalizationService` يستخدم `IMemoryCache` القياسي مباشرة لتخزين الترجمات.
- ❌ **ممنوع:** استخدام `LocalizationCacheService` (المبني على `Dictionary` + `SemaphoreSlim`) لأنه يسبب ازدواجية غير مبررة في استهلاك الذاكرة وتعقيداً في منطق الكاش دون إضافة قيمة.

### في تطبيق الموبايل (Mobile):
- ✅ **الصحيح:** `TranslationCacheService` يجمع بين `IMemoryCache` (للوصول السريع أثناء التشغيل) و `Preferences` (للحفظ المحلي الدائم عند إغلاق التطبيق).
- **لماذا هذا الاستثناء؟** طبيعة دورة حياة تطبيق الموبايل (App Lifecycle) تتطلب الوصول للترجمات حتى دون اتصال بالإنترنت، وهو ما لا ينطبق على تطبيق الويب الذي يعتمد على الخادم بشكل دائم.

---

## 📦 طبقات الكاش

| الطبقة | المسؤول | الموقع | المفاتيح | المدة |
|--------|---------|--------|----------|-------|
| **الجلسة** | `UserSessionService` | الخادم | `UserSession_{userId}` | ساعتين |
| **التفضيلات** | `UserSessionService` | الخادم | `UserPrefs_{userId}` | ساعة |
| **المعلومات الأساسية** | `UserSessionService` | الخادم | `UserBasic_{userId}` | ساعة |
| **ترجمات الويب** | `LocalizationService` | الخادم | `translation_{lang}_{key}` | 30 دقيقة |
| **ترجمات الموبايل** | `TranslationCacheService` | الموبايل | `translations_all_{lang}` | دائم (حتى Logout) |
| **كاش الموبايل** | `CachedUserSessionService` | الموبايل | `_cachedSession` | ساعتين |

---

## 🏗️ هيكل النظام

### 1. UserSessionService (الخادم - المصدر الرئيسي)

**المسار:** `RubikCare.Application/Services/Session/UserSessionService.cs`

**المسؤوليات:**
- تخزين بيانات الجلسة الكاملة (`UserSessionData`)
- تخزين تفضيلات المستخدم (`UserPreferences`)
- تخزين المعلومات الأساسية (`UserBasicInfo`)
- إدارة انتهاء الصلاحية (TTL)
- مسح الكاش عند Logout

**مثال على الاستخدام:**
```csharp
// ✅ صحيح - الحصول على بيانات الجلسة
var session = await _userSessionService.GetUserSessionAsync(user);

// ✅ صحيح - تحديث تفضيلات المستخدم
await _userSessionService.UpdateSessionLanguageAsync(userId, "ar");

// ✅ صحيح - مسح الكاش عند Logout
await _userSessionService.RefreshUserSessionAndCacheAsync(userId);
```

---

### 2. CachedUserSessionService (الموبايل - كاش محلي)

**المسار:** `RubikCare.Mobile/Infrastructure/Services/CachedUserSessionService.cs`

**المسؤوليات:**
- تخزين الجلسة محلياً في الذاكرة (`_cachedSession`)
- التحقق من صلاحية الكاش (2 ساعة)
- تحديث الكاش من API عند الحاجة
- مسح الكاش عند Logout

**مثال على الاستخدام:**
```csharp
// ✅ صحيح - الحصول على الجلسة (من الكاش أولاً)
var session = await _cachedSessionService.GetSessionAsync();

// ✅ صحيح - التحقق من الصلاحية
var isValid = await _cachedSessionService.IsSessionValidAsync();

// ✅ صحيح - مسح الكاش عند Logout
await _cachedSessionService.ClearSessionAsync();
```

---

### 3. LocalizationService (الويب - كاش الترجمات)

**المسار:** `RubikCare.Application/Services/LocalizationService.cs`

**المسؤوليات:**
- تخزين الترجمات في `IMemoryCache` القياسي
- إدارة انتهاء الصلاحية (30 دقيقة)
- جلب الترجمات من قاعدة البيانات عند الحاجة

**مثال على الاستخدام:**
```csharp
// ✅ صحيح - الحصول على ترجمة
var text = await _localizationService.GetTranslationAsync("COMMON.WELCOME", "ar");

// ✅ صحيح - الحصول على ترجمات صفحة كاملة
var translations = await _localizationService.GetPageTranslationsAsync("SHARED.DASHBOARD", "ar");
```

---

### 4. TranslationCacheService (الموبايل - كاش الترجمات)

**المسار:** `RubikCare.Mobile/Services/TranslationCacheService.cs`

**المسؤوليات:**
- تخزين الترجمات في `IMemoryCache` (للوصول السريع)
- تخزين الترجمات في `Preferences` (للحفظ الدائم)
- إدارة انتهاء الصلاحية (ساعة واحدة للذاكرة، دائم لـ Preferences)
- مسح الكاش عند تغيير اللغة

**مثال على الاستخدام:**
```csharp
// ✅ صحيح - الحصول على الترجمات من الكاش
var translations = await _cacheService.GetCachedAsync("ar");

// ✅ صحيح - حفظ الترجمات في الكاش
_cacheService.SetCache("ar", translations);

// ✅ صحيح - مسح كاش لغة معينة
_cacheService.ClearCache("ar");
```

---

## 🔴 القواعد الإلزامية

### ممنوعات

1. **لا تستخدم `IMemoryCache` للبيانات الحساسة (مثل الجلسات أو القوائم) خارج `UserSessionService`.** 
   *(الاستثناء الوحيد المسموح به والمطلوب: يُشترط استخدام `IMemoryCache` القياسي داخل `LocalizationService` للترجمات، مع إهمال أي أنظمة كاش مخصصة مثل `LocalizationCacheService` التي تعتمد على Dictionary).*

2. **لا تنشئ نظام كاش مخصص (Dictionary + SemaphoreSlim) إذا كان `IMemoryCache` يكفي.**

3. **لا تخزن بيانات المستخدم في `Static` fields** - هذا يسبب تسرب البيانات بين المستخدمين.

4. **لا تنسَ مسح الكاش عند Logout** - على الخادم (`UserSessionService`) وعلى العميل (`SecureStorage` + `_cachedSession`).

5. **لا تستخدم `IMemoryCache` للموبايل** - استخدم `Preferences` أو `SecureStorage` بدلاً منه.

### إلزاميات

1. **استخدم `UserSessionService` كمصدر وحيد لبيانات الجلسة على الخادم.**

2. **استخدم `CachedUserSessionService` في الموبايل للوصول السريع للجلسة.**

3. **استخدم `IMemoryCache` القياسي في `LocalizationService` على الخادم.**

4. **استخدم `TranslationCacheService` في الموبايل لإدارة كاش الترجمات.**

5. **حدد مدة صلاحية (TTL) واضحة لكل نوع من البيانات.**

6. **امسح الكاش عند أي تغيير في بيانات المستخدم** (تعديل ملف شخصي، تغيير دور، إلخ).

---

## 🔄 دورة حياة الكاش

### عند تسجيل الدخول

```text
1. المصادقة عبر Identity
2. إنشاء UserSessionData
3. حفظ في UserSessionService (الخادم)
4. حفظ في CachedUserSessionService (الموبايل)
5. تحميل الترجمات وحفظها في TranslationCacheService (الموبايل)
```

### أثناء الاستخدام

```text
1. التحقق من صلاحية الكاش
2. إذا صالح → إرجاع البيانات من الكاش
3. إذا منتهي → تحديث من قاعدة البيانات / API
4. حفظ البيانات المحدثة في الكاش
```

### عند تسجيل الخروج

```text
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

## 🧪 أمثلة عملية

### مثال 1: الحصول على بيانات المستخدم في صفحة Dashboard

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

---

### مثال 2: الحصول على الترجمات

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

---

### مثال 3: مسح الكاش عند Logout

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

## 📋 CHECKLIST: عند التعامل مع الكاش

### عند إنشاء خدمة جديدة

- [ ] هل حددت مصدر الكاش الصحيح (UserSessionService / CachedUserSessionService)؟
- [ ] هل حددت مدة صلاحية (TTL) مناسبة؟
- [ ] هل أضفت منطق التحقق من صلاحية الكاش؟
- [ ] هل أضفت منطق تحديث الكاش عند انتهاء الصلاحية؟
- [ ] هل أضفت منطق مسح الكاش عند Logout؟

### عند التعامل مع بيانات المستخدم

- [ ] هل استخدمت `UserSessionService` (الخادم) أو `CachedUserSessionService` (الموبايل)؟
- [ ] هل تجنبت تخزين البيانات الحساسة في `Static` fields؟
- [ ] هل مسحت الكاش عند أي تغيير في بيانات المستخدم؟

### عند التعامل مع الترجمات

- [ ] هل استخدمت `LocalizationService` مع `IMemoryCache` في الويب؟
- [ ] هل استخدمت `TranslationCacheService` في الموبايل؟
- [ ] ⭐ هل تم إهمال `LocalizationCacheService` في الويب والاعتماد حصرياً على `IMemoryCache` داخل `LocalizationService`؟
- [ ] هل مسحت كاش الترجمات عند تغيير اللغة؟

### عند تسجيل الخروج

- [ ] هل استدعيت `ClearUserCacheUseCase` على الخادم؟
- [ ] هل مسحت `CachedUserSessionService` في الموبايل؟
- [ ] هل مسحت `TranslationCacheService` في الموبايل؟
- [ ] هل مسحت `SecureStorage` في الموبايل؟

---

## ⚠️ تحذيرات ومحاذير

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

---

## 🔗 روابط ذات صلة

- [00 - الهيكل المعماري](00-architecture-overview.md)
- [01 - Program.cs والتسجيلات الأساسية](01-program-cs-foundation.md)
- [02 - نظام الهوية والمصادقة](02-identity-system.md)
- [15 - نظام الترجمة في الموبايل](15-mobile-translation-system.md)
- [الملحق أ - مسرد المصطلحات](appendix-a-glossary.md)

---

## 📝 سجل التحديثات

| التاريخ | الإصدار | التغييرات |
|---------|---------|-----------|
| 24 مايو 2026 | 1.0 | الإصدار الأولي - توحيد نظام الكاش |
| 14 يوليو 2026 | 1.2 | إضافة قسم "كاش الترجمات"، توضيح استثناء `IMemoryCache` للترجمات، إهمال `LocalizationCacheService`، إضافة بند جديد في CHECKLIST |
```

---

## ✅ ملخص التحديثات المضافة في هذه النسخة

1. **تحديث التاريخ والإصدار** إلى 14 يوليو 2026 / 1.2
2. **تحديث قسم "مصدر واحد للكاش"** ليشمل حالة الترجمات قبل وبعد الإصلاح
3. **إضافة قسم جديد كامل: "كاش الترجمات (Localization Caching)"** يوضح الفرق بين الويب والموبايل
4. **تحديث القاعدة رقم 1** في "ممنوعات" لتوضيح استثناء `LocalizationService`
5. **إضافة صفين جديدين** في جدول "طبقات الكاش" لترجمات الويب والموبايل
6. **إضافة قسمين جديدين** في "هيكل النظام" لـ `LocalizationService` و `TranslationCacheService`
7. **إضافة مثال عملي** لكيفية التعامل مع الترجمات في الويب والموبايل
8. **إضافة بند جديد** في CHECKLIST لإهمال `LocalizationCacheService`
9. **إضافة تحذيرين جديدين** في جدول المحاذير
10. **تحديث سجل التحديثات** لتوثيق التغييرات
