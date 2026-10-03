# 📘 وثيقة مشروع RubikCare.PWA

**آخر تحديث:** 3 أكتوبر 2026  
**الحالة:** مرحلة التطوير النشط (Alpha) — تشغيل الميزات الأساسية  
**الإصدار:** 0.3.0

---

## 🎯 1. المقدمة والهدف

### ما هو مشروع RubikCare.PWA؟

**RubikCare.PWA** هو تطبيق ويب تقدمي (Progressive Web App) مبني بتقنية **Blazor WebAssembly** في إطار **.NET 10**. يمثل **العميل الثالث** في منظومة RubikCare:

| المشروع | التقنية | الجمهور | الحالة |
|---------|---------|---------|--------|
| **RubikCare.Web** | Blazor Server | الإدارة والموظفون | ✅ منشور |
| **RubikCare.Mobile** | .NET MAUI + BlazorWebView | المرضى والمهنيون | ✅ منشور على Google Play |
| **RubikCare.PWA** ⭐ | Blazor WebAssembly | المرضى والمهنيون (ويب متقدم) | 🚧 قيد التطوير (35%) |

### الأهداف الاستراتيجية

1. **تجربة موحدة:** نفس تجربة الموبايل (تصميم بطاقات على تدرج لوني) وليس نمط الويب (Split-Panel)
2. **إعادة الاستخدام القصوى:** تشغيل مكونات `Shared.UI` الموجودة بالفعل دون إعادة بنائها
3. **قابلية التثبيت:** تثبيت التطبيق على الشاشة الرئيسية (iOS/Android/Desktop)
4. **مصدر حقيقة واحد:** بيانات الجلسة تُجلب مرة واحدة وتُشارك عبر جميع الصفحات
5. **🆕 مصادقة موحدة:** دعم تسجيل الدخول التقليدي و Google OAuth

---

## 🏗️ 2. البنية المعمارية والربط مع الحل

### الموقع في Clean Architecture

```
┌─────────────────────────────────────────────────────────┐
│                    RubikCare.Api.Web                     │
│              (نقطة الدخول الوحيدة للبيانات)              │
└─────────────────────┬───────────────────────────────────┘
                      │ HTTP/REST + JWT
        ┌─────────────┼─────────────┐
        ▼             ▼             ▼
   ┌────────┐   ┌──────────┐   ┌──────────┐
   │  Web   │   │  Mobile  │   │  PWA ⭐  │
   │ Server │   │  (MAUI)  │   │  (WASM)  │
   └────────┘   └──────────┘   └──────────┘
        │             │             │
        └─────────────┼─────────────┘
                      ▼
              ┌───────────────┐
              │  Shared.UI    │
              │    (RCL)      │
              └───────────────┘
```

### التبعيات المسموحة

```
RubikCare.PWA
├── ✅ يعتمد على: RubikCare.Shared.UI (مكونات + خدمات)
├── ✅ يتواصل مع: Api.Web عبر HTTP فقط
├── 🆕 يدعم: Google.Apis.Auth (للتحقق من ID Token)
├── ❌ لا يعتمد على: Infrastructure
├── ❌ لا يعتمد على: Domain مباشرة
└── ❌ لا يعتمد على: DbContext
```

### ملفات الربط الحرجة (الحالية)

| الملف | الدور | الحالة |
|-------|-------|--------|
| `PWA/Program.cs` | تسجيل الخدمات (`IApiService`, `WebApiService`, `WebTranslationService`, `UserSessionState`) | ✅ مكتمل |
| `PWA/Layout/MainLayout.razor` | الهيكل الرئيسي + تحميل الجلسة مرة واحدة | ✅ مكتمل + محسّن |
| `PWA/Layout/NavMenu.razor` | القائمة الجانبية الديناميكية بالترجمات | ✅ مكتمل + Logout مُصحح |
| `PWA/Services/UserSessionState.cs` | حالة الجلسة المشتركة (تستدعي `api/user/session-bootstrap`) | ✅ مكتمل |
| `PWA/Services/WebApiService.cs` | تنفيذ `IApiService` عبر HTTP مع التوكن | ✅ مكتمل |
| `PWA/Services/WebTranslationService.cs` | تنفيذ `ISharedTranslationService` عبر API | ✅ مكتمل |
| 🆕 `PWA/wwwroot/cache-reset.html` | صفحة تنظيف الكاش التلقائية | ✅ مكتمل |
| 🆕 `PWA/wwwroot/js/googleAuth.js` | تكامل Google Identity Services | ✅ مكتمل |

---

## 🔐 3. نظام المصادقة (Authentication System)

### 3.1 طرق تسجيل الدخول المدعومة

| الطريقة | الحالة | الـ Endpoint |
|---------|--------|--------------|
| **تقليدي** (Email + Password) | ✅ مكتمل | `POST /api/auth/login` |
| **Google OAuth** (ID Token) | ✅ مكتمل | `POST /api/auth/google-login` |
| **تسجيل حساب جديد** | ✅ مكتمل | `POST /api/auth/register` |

### 3.2 تدفق Google OAuth في PWA

```
المستخدم يضغط "Sign in with Google"
    ↓
Google Identity Services (GIS) يفتح نافذة اختيار الحساب
    ↓
Google يُرجع ID Token (JWT موقّع من Google)
    ↓
PWA يرسل ID Token إلى API: POST /api/auth/google-login
    ↓
API يتحقق من التوقيع باستخدام Google.Apis.Auth
    ↓
API يُرجع JWT token خاص بالتطبيق
    ↓
PWA يحفظ Token في localStorage
    ↓
UserSessionState يجلب بيانات الجلسة
    ↓
المستخدم يدخل للوحة التحكم ✅
```

### 3.3 Client IDs المتعددة

النظام يدعم **Client IDs متعددة** لنفس المشروع:

| المنصة | Client ID | الاستخدام |
|--------|-----------|-----------|
| MAUI (Android/iOS) | `369163319733-8229...` | Native Apps |
| PWA (Web) | `369163319733-9rt7...` | Web Browser |
| Web (Server) | `369163319733-abc1...` | Blazor Server |

**التحقق في الـ API:**
```csharp
var settings = new GoogleJsonWebSignature.ValidationSettings
{
    Audience = new[] { 
        _configuration["Google:ClientId"],      // MAUI
        _configuration["Google:PwaClientId"]    // PWA
    }
};
```

### 3.4 Logout الموحّد

```csharp
private async Task Logout()
{
    // 1. مسح token من localStorage
    await JS.InvokeVoidAsync("localStorage.removeItem", "auth_token");

    // 2. مسح حالة المصادقة
    if (AuthStateProvider is CustomAuthenticationStateProvider customProvider)
    {
        await customProvider.MarkUserAsLoggedOut();
    }

    // 3. مسح حالة المنظمة
    OrgState.SetOrganizationId(0);

    // 4. إعادة التوجيه باستخدام JavaScript (يتجنب 404 في PWA)
    await JS.InvokeVoidAsync("eval", "window.location.href = '/login'");
}
```

**لماذا `window.location.href` بدلاً من `Navigation.NavigateTo`؟**
- `Navigation.NavigateTo("/login", forceLoad: true)` يسبب 404 في PWA
- `window.location.href` يعمل بشكل صحيح مع URL Rewrite

---

## 🧹 4. نظام تنظيف الكاش التلقائي

### 4.1 المشكلة التي يحلها

عند نشر نسخة جديدة من PWA، قد يبقى **Service Worker القديم** في المتصفح ويخدّم ملفات `.wasm` بـ hashes قديمة، مما يسبب:

```
Failed to find a valid digest in the 'integrity' attribute
SRI's integrity checks failed
```

### 4.2 الحل: `cache-reset.html`

صفحة HTML ثابتة (ليست Blazor) تُفتح تلقائياً عند فشل التحميل، وتقوم بـ:

1. **إلغاء Service Workers:**
```javascript
const registrations = await navigator.serviceWorker.getRegistrations();
for (const registration of registrations) {
    await registration.unregister();
}
```

2. **مسح جميع الكاشات:**
```javascript
const cacheNames = await caches.keys();
for (const cacheName of cacheNames) {
    await caches.delete(cacheName);
}
```

3. **مسح localStorage/sessionStorage**

4. **إعادة التوجيه التلقائي للصفحة الرئيسية**

### 4.3 الكشف التلقائي عن الفشل

في `index.html`، يتم الكشف عن أخطاء التحميل:

```javascript
window.addEventListener('unhandledrejection', function (event) {
    var msg = event.reason?.message?.toLowerCase() || '';
    if (msg.indexOf('integrity') !== -1 ||
        msg.indexOf('failed to fetch') !== -1) {
        window.location.href = '/cache-reset.html';
    }
});
```

### 4.4 الحماية من حلقة التوجيه اللانهائية

```javascript
const MAX_ATTEMPTS = 3;
const attempts = parseInt(sessionStorage.getItem('rubik_reset_attempts') || '0');

if (attempts >= MAX_ATTEMPTS) {
    // عرض رسالة خطأ بدلاً من إعادة التوجيه
    errorBox.classList.add('visible');
    return;
}
```

---

## 🎨 5. تحسينات واجهة المستخدم

### 5.1 TopBar المحسّن

**الشعار قابل للنقر:**
```razor
<div class="topbar-brand clickable-brand" @onclick="GoToHome" title="الصفحة الرئيسية">
    <img src="_content/RubikCare.Shared.UI/Images/own/RubickLogo.png"
         class="topbar-logo" />
    <span class="topbar-brand-text">RubikCare</span>
</div>
```

**صورة المستخدم قابلة للنقر:**
```razor
<div class="topbar-user clickable-user" @onclick="GoToProfile" title="البروفايل الشخصي">
    <div class="user-avatar">
        @if (!string.IsNullOrEmpty(SessionState.CurrentSession?.ProfilePictureUrl))
        {
            <img src="@fullImageUrl" />
        }
        else
        {
            <span>@GetInitials(SessionState.CurrentSession?.FullNameAr)</span>
        }
    </div>
    <span class="user-name">@SessionState.CurrentSession?.FullNameAr</span>
</div>
```

### 5.2 حل مشكلة CSS Specificity

**المشكلة:** ملف CSS عام (`_CoreBundle.css`) يُعرّف `.topbar-logo` بحجم ثابت `36px` مع specificity أعلى.

**الحل:** استخدام `!important` في `MainLayout.razor.css`:

```css
.topbar-logo {
    height: 98px !important;
    width: 98px !important;
}
```

---

## 📱 6. نظام المحادثات (Messaging)

### 6.1 تدفق الحجز مع المحادثة

```
المريض يبحث عن عيادة (DoctorSearchPage)
    ↓
يختار عيادة → يفتح DoctorProfilePage
    ↓
يملأ نموذج الحجز (يوم + سبب + ملاحظات)
    ↓
يضغط "إرسال طلب الحجز"
    ↓
DoctorProfilePage:
    1. ينشئ محادثة: POST /api/messaging/conversations
    2. يرسل رسالة الحجز: POST /api/messaging/messages
    3. يوجّه لصفحة المحادثة: Navigation.NavigateTo($"/chat/{conversationId}")
    ↓
ChatPage يفتح ويعرض المحادثة ✅
```

### 6.2 صفحات المحادثات المُفعّلة

| الصفحة | المسار | الحالة |
|--------|--------|--------|
| `DoctorSearchPage` | `/doctor-search` | ✅ مكتمل |
| `DoctorProfilePage` | `/doctorprofile/{clinicId}` | ✅ مكتمل + ربط بالمحادثة |
| `ChatPage` | `/chat/{conversationId}` | ✅ مكتمل |
| `MessagingHubPage` | `/messaging-hub` | ⏳ قيد التطوير |

---

## 🛡️ 7. حلول المشاكل الشائعة

### 7.1 خطأ: `Illegal invocation` مع Google GIS

**المشكلة:**
```
TypeError: Failed to execute 'query' on 'Permissions': Illegal invocation
```

**السبب:** مكتبة Google GIS تستدعي `navigator.permissions.query` بطريقة تفقد السياق `this`.

**الحل:** Polyfill في `index.html` قبل تحميل مكتبة Google:

```javascript
if (navigator.permissions && navigator.permissions.query) {
    var originalQuery = navigator.permissions.query.bind(navigator.permissions);
    navigator.permissions.query = function (parameters) {
        try {
            return originalQuery(parameters);
        } catch (e) {
            return Promise.reject(e);
        }
    };
}
```

### 7.2 خطأ: `404 Not Found` عند تسجيل الخروج

**المشكلة:**
```
Navigation.NavigateTo("/login", forceLoad: true) → 404
```

**السبب:** `forceLoad: true` يطلب `/login` كملف HTML، لكن IIS لا يجده.

**الحل:** استخدام `window.location.href` بدلاً من `Navigation.NavigateTo`.

### 7.3 خطأ: تضارب المسارات (Duplicate Routes)

**المشكلة:**
```
The following routes are ambiguous:
'pharmacy/patient-orders' in 'PatientOrders.razor'
'pharmacy/patient-orders' in 'PharmacyPatientOrdersPage.razor'
```

**الحل:** حذف الملف المكرر والاحتفاظ بالأوضح (`PharmacyPatientOrdersPage.razor`).

---

## 📦 8. البنية الحقيقية لـ Shared.UI

*(نفس القسم 3 من الوثيقة القديمة - لم يتغير)*

---

## 🗺️ 9. خطة تشغيل مكونات Shared.UI في PWA

### المرحلة 1: الصفحات الأساسية ✅ مكتملة

- [x] صفحات المصادقة (Login, Register, ForgotPassword)
- [x] Dashboard (لوحة التحكم الأساسية)
- [x] الصفحات القانونية (Terms, Privacy, Support)
- [x] Profile (البروفايل الشخصي)

### المرحلة 2: الميزات الأساسية ✅ مكتملة

- [x] Google OAuth
- [x] Logout الموحّد
- [x] Doctor Search + Booking
- [x] Messaging (ChatPage)
- [x] TopBar المحسّن

### المرحلة 3: الميزات المتقدمة 🔄 قيد التطوير

- [ ] MessagingHubPage (مركز المحادثات الكامل)
- [ ] MedicationSchedulePage (جدولة الأدوية)
- [ ] NearbyPharmacies (الصيدليات القريبة - تحتاج GPS)
- [ ] PspGateway (بوابة برامج الدعم)

### المرحلة 4: لوحات التحكم المهنية ⏳ قادمة

- [ ] ClinicDashboard (لوحة الطبيب)
- [ ] PharmacyDashboard (لوحة الصيدلية)
- [ ] RepDashboard (لوحة المندوب)

---

## 📊 10. إحصائيات المشروع

| المقياس | القيمة (أغسطس) | القيمة (أكتوبر) | التحسن |
|---------|----------------|-----------------|--------|
| نسبة الإنجاز | ~20% | ~35% | +15% |
| صفحات @page مُفعّلة | 11 | 15+ | +4 |
| أنظمة المصادقة | 1 (تقليدي) | 2 (+ Google) | +1 |
| أنظمة الحماية | 0 | 2 (Cache Reset + Polyfill) | +2 |
| صفحات المحادثات | 2 | 4 | +2 |

---

## 🎓 11. الدروس المستفادة من هذه الجلسة

### 11.1 نشر PWA يتطلب مسح Service Worker

**الخطأ:** نسخ الملفات الجديدة فوق القديمة بدون مسح المجلد.

**الصحيح:**
```powershell
Get-ChildItem "C:\WebSite\PU_RubicCareStage" -Exclude "web.config" | 
    Remove-Item -Recurse -Force
```

### 11.2 Blazor يستخدم filenames مجزأة (hashed)

**الخطأ:** افتراض أن أسماء ملفات `.wasm` ثابتة.

**الصحيح:** كل `publish` يولّد أسماء جديدة مثل `RubikCare.PWA.abc123.wasm`.

### 11.3 CSS Specificity يتغلب على الترتيب

**الخطأ:** افتراض أن آخر ملف CSS يُحمّل يتغلب.

**الصحيح:** `!important` أو specificity أعلى يتغلب على الترتيب.

### 11.4 Navigation.NavigateTo قد يفشل في PWA

**الخطأ:** استخدام `forceLoad: true` في PWA.

**الصحيح:** استخدام `window.location.href` للتوجيه الكامل.

### 11.5 Client IDs متعددة مطلوبة لمنصات مختلفة

**الخطأ:** استخدام نفس Client ID لـ MAUI و PWA.

**الصحيح:** كل منصة (Android, iOS, Web) تحتاج Client ID خاص بها.

---

## 📝 12. ملاحظات النشر (Deployment Notes)

### 12.1 خطوات النشر الصحيحة

```powershell
# 1. بناء المشروع
dotnet publish RubikCare.PWA -c Release -o E:\Publish\PWA

# 2. إيقاف App Pool
C:\Windows\System32\inetsrv\appcmd stop apppool "PU_RubicCareStage"

# 3. مسح المجلد (ما عدا web.config)
Get-ChildItem "C:\WebSite\PU_RubicCareStage" -Exclude "web.config" | 
    Remove-Item -Recurse -Force

# 4. نسخ الملفات الجديدة
Copy-Item -Path "E:\Publish\PWA\*" `
    -Destination "C:\WebSite\PU_RubicCareStage\" -Recurse -Force

# 5. تشغيل App Pool
C:\Windows\System32\inetsrv\appcmd start apppool "PU_RubicCareStage"
```

### 12.2 مسح Service Worker من المتصفح

بعد النشر، يجب على المستخدم:

1. فتح `https://stagepu.rubikcare.com`
2. الضغط على `F12` → **Application**
3. **Service Workers** → **Unregister**
4. **Storage** → **Clear site data**
5. أو استخدام نافذة **InPrivate** (`Ctrl + Shift + N`)

---

## ✅ 13. الحالة الحالية والخطوات التالية

### ✅ ما تم إنجازه (0.3.0)

- [x] البنية التحتية للـ PWA (Program.cs, Services)
- [x] صفحات المصادقة (Login, Register) بنمط الموبايل
- [x] **Google OAuth** (تسجيل الدخول عبر Google)
- [x] الهيكل الرئيسي (MainLayout, NavMenu, EmptyLayout)
- [x] **TopBar محسّن** (شعار + صورة مستخدم قابلان للنقر)
- [x] نظام الترجمة الكامل (عربي/إنجليزي)
- [x] نظام الجلسة الموحدة (UserSessionState)
- [x] لوحة تحكم أساسية (Dashboard) ببيانات حقيقية
- [x] القائمة الجانبية الديناميكية بالبيانات الحقيقية
- [x] **Doctor Search + Booking**
- [x] **Messaging (ChatPage)**
- [x] **صفحة تنظيف الكاش التلقائية**
- [x] **Logout الموحّد**
- [x] **Polyfill لـ Google GIS**

### 🔄 الخطوة التالية

**المرحلة 3:** الميزات المتقدمة
- MessagingHubPage (مركز المحادثات الكامل)
- MedicationSchedulePage (جدولة الأدوية)
- NearbyPharmacies (الصيدليات القريبة)

### 📊 إحصائيات المشروع

| المقياس | القيمة |
|---------|--------|
| مكونات Shared.UI المتاحة | 66 |
| صفحات @page مُفعّلة | 15+ |
| خدمات تحتاج تنفيذ في PWA | ~8 |
| نسبة الإنجاز الحالية | **~35%** |

---

**ملاحظة:** هذه وثيقة حية (Living Document) — يتم تحديثها مع تقدم المشروع.

**آخر مراجعة:** 3 أكتوبر 2026  
**المراجعة التالية:** بعد إكمال المرحلة 3 (الميزات المتقدمة)

---

## ✅ ملخص التحديثات

تم تحديث الوثيقة لتشمل:

1. ✅ **تاريخ التحديث:** 3 أكتوبر 2026 (بدلاً من 26 أغسطس)
2. ✅ **الإصدار:** 0.3.0 (بدلاً من 0.2.0)
3. ✅ **نسبة الإنجاز:** 35% (بدلاً من 20%)
4. ✅ **قسم جديد:** نظام المصادقة (Google OAuth)
5. ✅ **قسم جديد:** نظام تنظيف الكاش التلقائي
6. ✅ **قسم جديد:** تحسينات واجهة المستخدم
7. ✅ **قسم جديد:** نظام المحادثات
8. ✅ **قسم جديد:** حلول المشاكل الشائعة
9. ✅ **قسم جديد:** ملاحظات النشر
10. ✅ **قسم جديد:** الدروس المستفادة من هذه الجلسة

**الوثيقة جاهزة للاستخدام!** 🎉
