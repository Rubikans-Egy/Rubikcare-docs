
# 14 — PWA (تطبيق الويب التقدمي)

**الإصدار:** 2.0 (Unified Edition)
**آخر تحديث:** 17 أكتوبر 2026
**الحالة:** ✅ مرجع شامل — يدمج 22 + إضافات
**الجمهور:** فريق التطوير + AI Assistants
**المرجع الإلزامي:** يُقرأ قبل أي تعديل على PWA

---

## 📌 الجزء صفر: الفلسفة والرؤية

### 0.1 ما هو RubikCare.PWA؟

**RubikCare.PWA** هو تطبيق ويب تقدمي (Progressive Web App) مبني بتقنية **Blazor WebAssembly** في إطار **.NET 10**. يمثل **العميل الثالث** في منظومة RubikCare:

| المشروع | التقنية | الجمهور | الحالة |
|---------|---------|---------|--------|
| **RubikCare.Web** | Blazor Server | الإدارة والموظفون | ✅ منشور |
| **RubikCare.Mobile** | .NET MAUI + BlazorWebView | المرضى والمهنيون | ✅ منشور على Google Play |
| **RubikCare.PWA** ⭐ | Blazor WebAssembly | المرضى والمهنيون (ويب متقدم) | 🟠 **~95%** |

### 0.2 الأهداف الاستراتيجية

1. **تجربة موحدة:** نفس تجربة الموبايل (تصميم بطاقات على تدرج لوني) وليس نمط الويب (Split-Panel).
2. **إعادة الاستخدام القصوى:** تشغيل مكونات `Shared.UI` الموجودة بالفعل.
3. **قابلية التثبيت:** تثبيت التطبيق على الشاشة الرئيسية (iOS/Android/Desktop).
4. **مصدر حقيقة واحد:** بيانات الجلسة تُجلب مرة واحدة وتُشارك.
5. **🆕 مصادقة موحدة:** دعم تسجيل الدخول التقليدي و Google OAuth.

---

## 🏗️ الجزء الأول: البنية المعمارية

### 1.1 الموقع في Clean Architecture

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

### 1.2 التبعيات المسموحة

```
RubikCare.PWA
├── ✅ يعتمد على: RubikCare.Shared.UI (مكونات + خدمات)
├── ✅ يتواصل مع: Api.Web عبر HTTP فقط
├── 🆕 يدعم: Google.Apis.Auth (للتحقق من ID Token)
├── ❌ لا يعتمد على: Infrastructure
├── ❌ لا يعتمد على: Domain مباشرة
└── ❌ لا يعتمد على: DbContext
```

### 1.3 ملفات الربط الحرجة (الحالية)

| الملف | الدور | الحالة |
|-------|-------|--------|
| `PWA/Program.cs` | تسجيل الخدمات | ✅ مكتمل |
| `PWA/Layout/MainLayout.razor` | الهيكل الرئيسي + تحميل الجلسة | ✅ مكتمل + محسّن |
| `PWA/Layout/NavMenu.razor` | القائمة الجانبية الديناميكية | ✅ مكتمل |
| `PWA/Services/UserSessionState.cs` | حالة الجلسة المشتركة | ✅ مكتمل |
| `PWA/Services/WebApiService.cs` | تنفيذ `IApiService` عبر HTTP | ✅ مكتمل |
| `PWA/Services/WebTranslationService.cs` | تنفيذ `ISharedTranslationService` | ✅ مكتمل |
| 🆕 `PWA/wwwroot/cache-reset.html` | صفحة تنظيف الكاش التلقائية | ✅ مكتمل |
| 🆕 `PWA/wwwroot/js/googleAuth.js` | تكامل Google Identity Services | ✅ مكتمل |

### 1.4 الخدمات المسجلة في `Program.cs`

**⚠️ من الجرد الفعلي:**

| الخدمة | التنفيذ | الحالة |
|--------|---------|--------|
| `IApiService` | `WebApiService` | ✅ |
| `ITranslationService` | `WebTranslationService` | ✅ |
| `ITranslationState` | `SharedTranslationState` | ✅ |
| `ILocalNotificationService` | `WebLocalNotificationService` | ✅ |
| `UserSessionState` | — | ✅ |
| `CurrentOrganizationState` | — | ✅ |
| `AuthenticationStateProvider` | `CustomAuthenticationStateProvider` | ✅ |

---

## 🔐 الجزء الثاني: نظام المصادقة

### 2.1 طرق تسجيل الدخول المدعومة

| الطريقة | الحالة | الـ Endpoint |
|---------|--------|--------------|
| **تقليدي** (Email + Password) | ✅ مكتمل | `POST /api/auth/login` |
| **Google OAuth** (ID Token) | ✅ مكتمل | `POST /api/auth/google-login` |
| **تسجيل حساب جديد** | ✅ مكتمل | `POST /api/auth/register` |

### 2.2 تدفق Google OAuth في PWA

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

### 2.3 Client IDs المتعددة

**النظام يدعم Client IDs متعددة لنفس المشروع:**

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

### 2.4 Logout الموحّد

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

**⚠️ لماذا `window.location.href` بدلاً من `Navigation.NavigateTo`؟**
- `Navigation.NavigateTo("/login", forceLoad: true)` يسبب 404 في PWA.
- `window.location.href` يعمل بشكل صحيح مع URL Rewrite.

---

## 🧹 الجزء الثالث: نظام تنظيف الكاش التلقائي

### 3.1 المشكلة التي يحلها

**عند نشر نسخة جديدة من PWA، قد يبقى Service Worker القديم في المتصفح ويخدّم ملفات `.wasm` بـ hashes قديمة، مما يسبب:**

```
Failed to find a valid digest in the 'integrity' attribute
SRI's integrity checks failed
```

### 3.2 الحل: `cache-reset.html`

**صفحة HTML ثابتة (ليست Blazor) تُفتح تلقائياً عند فشل التحميل، وتقوم بـ:**

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

3. **مسح localStorage/sessionStorage**.

4. **إعادة التوجيه التلقائي للصفحة الرئيسية.**

### 3.3 الكشف التلقائي عن الفشل

**في `index.html`، يتم الكشف عن أخطاء التحميل:**

```javascript
window.addEventListener('unhandledrejection', function (event) {
    var msg = event.reason?.message?.toLowerCase() || '';
    if (msg.indexOf('integrity') !== -1 ||
        msg.indexOf('failed to fetch') !== -1) {
        window.location.href = '/cache-reset.html';
    }
});
```

### 3.4 الحماية من حلقة التوجيه اللانهائية

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

## 🎨 الجزء الرابع: تحسينات واجهة المستخدم

### 4.1 TopBar المحسّن

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

### 4.2 حل مشكلة CSS Specificity

**المشكلة:** ملف CSS عام (`_CoreBundle.css`) يُعرّف `.topbar-logo` بحجم ثابت `36px` مع specificity أعلى.

**الحل:** استخدام `!important` في `MainLayout.razor.css`:

```css
.topbar-logo {
    height: 98px !important;
    width: 98px !important;
}
```

---

## 📱 الجزء الخامس: نظام المحادثات

### 5.1 تدفق الحجز مع المحادثة

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

### 5.2 صفحات المحادثات المُفعّلة

| الصفحة | المسار | الحالة |
|--------|--------|--------|
| `DoctorSearchPage` | `/doctor-search` | ✅ مكتمل |
| `DoctorProfilePage` | `/doctorprofile/{clinicId}` | ✅ مكتمل + ربط بالمحادثة |
| `ChatPage` | `/chat/{conversationId}` | ✅ مكتمل |
| `MessagingHubPage` | `/messaging-hub` | ⏳ قيد التطوير |

---

## 🛡️ الجزء السادس: حلول المشاكل الشائعة

### 6.1 خطأ: `Illegal invocation` مع Google GIS

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

### 6.2 خطأ: `404 Not Found` عند تسجيل الخروج

**المشكلة:**
```
Navigation.NavigateTo("/login", forceLoad: true) → 404
```

**السبب:** `forceLoad: true` يطلب `/login` كملف HTML، لكن IIS لا يجده.

**الحل:** استخدام `window.location.href` بدلاً من `Navigation.NavigateTo`.

### 6.3 خطأ: تضارب المسارات (Duplicate Routes)

**المشكلة:**
```
The following routes are ambiguous:
'pharmacy/patient-orders' in 'PatientOrders.razor'
'pharmacy/patient-orders' in 'PharmacyPatientOrdersPage.razor'
```

**الحل:** حذف الملف المكرر والاحتفاظ بالأوضح (`PharmacyPatientOrdersPage.razor`).

---

## 📊 الجزء السابع: حالة PWA — نسبة الإنجاز

### 7.1 نسبة الإنجاز الفعلية (~95%)

| العنصر | النسبة |
|--------|:------:|
| **البنية التحتية** | ✅ 100% |
| **صفحات المصادقة** | ✅ 100% |
| **صفحات PSP** | ✅ 100% |
| **صفحات المريض** | ✅ 100% |
| **صفحات الطبيب** | ✅ 100% |
| **صفحات الصيدلية** | ✅ 100% |
| **صفحات المندوب** | ✅ 100% |
| **نظام النقاط** | ✅ 100% |
| **نظام المحادثات** | ✅ 90% |
| **الإشعارات Push** | ✅ 100% |
| **Google Login** | ✅ 100% |
| **الإجمالي** | **~95%** |

### 7.2 إحصائيات المشروع

| المقياس | القيمة |
|---------|--------|
| **مكونات Shared.UI المتاحة** | 68 |
| **صفحات @page مُفعّلة** | 15+ |
| **خدمات تحتاج تنفيذ في PWA** | ~8 |

---

## 📱 الجزء الثامن: Web Push Notifications (PWA)

### 8.1 نظامان منفصلان

**⚠️ من `08-NOTIFICATIONS`:**

| النظام | الجدول | الاستخدام |
|--------|--------|-----------|
| **FCM Native** | `DeviceTokens` | تطبيق MAUI |
| **Web Push** | `UserWebPushSubscriptions` | PWA |

### 8.2 ⚠️ المشكلة الحرجة: ملفان Service Worker

**⚠️ مشاريع Blazor WASM PWA فيها ملفان:**

| الملف | الاستخدام |
|-------|-----------|
| `wwwroot/service-worker.js` | **Dev فقط** |
| `wwwroot/service-worker.published.js` | **القالب الفعلي** |

**⚠️ عند `dotnet publish` — يُحل `service-worker.published.js` محل `service-worker.js`.**

**⚠️ الخطأ الشائع:** التعديل في `service-worker.js` — يضيع مع `publish`.

### 8.3 الحل: `service-worker.published.js`

**⚠️ يجب التعديل هنا.**

**⚠️ التفاصيل الكاملة في `08-NOTIFICATIONS`.**

### 8.4 المسار الكامل (End-to-End)

```
1. المستخدم يفتح PWA → MainLayout.razor → InitializePushSafelyAsync()
2. push-notifications.js → subscribeToPush(vapidPublicKey)
3. POST /api/notifications/register-web-push
   ↓
══════════════ (انتهى التسجيل) ══════════════
   ↓
4. Trigger (send-test / حدث حقيقي)
5. NotificationsController → يحفظ Notification في DB
6. _pushNotificationService.SendToUserAsync() → FCM (MAUI)
7. _webPushSenderService.SendToUserAsync() → Web Push (PWA)
8. WebPushClient.SendNotificationAsync() مع VAPID → fcm.googleapis.com
9. service-worker.js (النسخة المنشورة الفعلية) → addEventListener('push')
10. self.registration.showNotification() → 🔔 الإشعار يظهر
```

**⚠️ التفاصيل الكاملة في `08-NOTIFICATIONS`.**

---

## 🎯 الجزء التاسع: خطة تشغيل مكونات Shared.UI في PWA

### 9.1 المرحلة 1: الصفحات الأساسية ✅ مكتملة

- [x] صفحات المصادقة (Login, Register, ForgotPassword).
- [x] Dashboard (لوحة التحكم الأساسية).
- [x] الصفحات القانونية (Terms, Privacy, Support).
- [x] Profile (البروفايل الشخصي).

### 9.2 المرحلة 2: الميزات الأساسية ✅ مكتملة

- [x] Google OAuth.
- [x] Logout الموحّد.
- [x] Doctor Search + Booking.
- [x] Messaging (ChatPage).
- [x] TopBar المحسّن.

### 9.3 المرحلة 3: الميزات المتقدمة 🔄 قيد التطوير

- [ ] MessagingHubPage (مركز المحادثات الكامل).
- [ ] MedicationSchedulePage (جدولة الأدوية).
- [ ] NearbyPharmacies (الصيدليات القريبة - تحتاج GPS).
- [ ] PspGateway (بوابة برامج الدعم).

### 9.4 المرحلة 4: لوحات التحكم المهنية ⏳ قادمة

- [ ] ClinicDashboard (لوحة الطبيب).
- [ ] PharmacyDashboard (لوحة الصيدلية).
- [ ] RepDashboard (لوحة المندوب).

---

## 🎓 الجزء العاشر: الدروس المستفادة

### 10.1 نشر PWA يتطلب مسح Service Worker

**الخطأ:** نسخ الملفات الجديدة فوق القديمة بدون مسح المجلد.

**الصحيح:**
```powershell
Get-ChildItem "C:\WebSite\PU_RubicCareStage" -Exclude "web.config" | 
    Remove-Item -Recurse -Force
```

### 10.2 Blazor يستخدم filenames مجزأة (hashed)

**الخطأ:** افتراض أن أسماء ملفات `.wasm` ثابتة.

**الصحيح:** كل `publish` يولّد أسماء جديدة مثل `RubikCare.PWA.abc123.wasm`.

### 10.3 CSS Specificity يتغلب على الترتيب

**الخطأ:** افتراض أن آخر ملف CSS يُحمّل يتغلب.

**الصحيح:** `!important` أو specificity أعلى يتغلب على الترتيب.

### 10.4 Navigation.NavigateTo قد يفشل في PWA

**الخطأ:** استخدام `forceLoad: true` في PWA.

**الصحيح:** استخدام `window.location.href` للتوجيه الكامل.

### 10.5 Client IDs متعددة مطلوبة لمنصات مختلفة

**الخطأ:** استخدام نفس Client ID لـ MAUI و PWA.

**الصحيح:** كل منصة (Android, iOS, Web) تحتاج Client ID خاص بها.

---

## 📝 الجزء الحادي عشر: ملاحظات النشر

### 11.1 خطوات النشر الصحيحة

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

### 11.2 مسح Service Worker من المتصفح

**بعد النشر، يجب على المستخدم:**

1. فتح `https://stagepu.rubikcare.com`.
2. الضغط على `F12` → **Application**.
3. **Service Workers** → **Unregister**.
4. **Storage** → **Clear site data**.
5. أو استخدام نافذة **InPrivate** (`Ctrl + Shift + N`).

**⚠️ التفاصيل الكاملة في `15-DEPLOYMENT`.**

---

## 🔧 الجزء الثاني عشر: المشاكل المعروفة

### 12.1 قائمة المشاكل

| # | المشكلة | الحالة |
|:-:|:---|:---:|
| 1 | `MessagingHubPage` — قيد التطوير | 🟡 |
| 2 | تجربة شاملة (E2E) — لم تُنفَّذ | ⏳ |
| 3 | `MedicationSchedulePage` — غير مُنفَّذ | ⏳ |
| 4 | `NearbyPharmacies` — يحتاج GPS | ⏳ |
| 5 | `PspGateway` — غير مُنفَّذ | ⏳ |
| 6 | لوحات مهنية (Clinic/Pharmacy/Rep) | ⏳ |
| 7 | علامات استفهام في النص العربي (Web Push) | 🟡 |

**⚠️ التفاصيل الكاملة في `08-NOTIFICATIONS` (للمشكلة 7).**

---

## 📊 الجزء الثالث عشر: الفرق بين PWA و Mobile

### 13.1 الفروقات الجوهرية

| البُعد | PWA | Mobile (MAUI) |
|:---|:---|:---|
| **التقنية** | Blazor WASM | .NET MAUI + BlazorWebView |
| **التشغيل** | في المتصفح | تطبيق أصلي |
| **الإشعارات** | Web Push (Service Worker) | FCM (Device Tokens) |
| **التخزين** | localStorage / IndexedDB | SecureStorage / Preferences |
| **الإصدار** | GitHub Pages / IIS (Static Files) | Google Play / App Store |
| **التحديث** | Service Worker (تلقائي) | Google Play (يدوي) |
| **الحجم** | ~2-3 MB (بدون Assets) | ~15-20 MB |

### 13.2 التكامل

**⚠️ كلاهما يستخدم:**
- `Shared.UI` (المكونات).
- `Api.Web` (الـ Backend).
- نفس نظام الترجمة (`04-TRANSLATION`).
- نفس نظام النقاط (`06-POINTS`).

---

## 📝 الجزء الرابع عشر: سجل التغييرات

### 14.1 سجل التحديثات

| الإصدار | التاريخ | التغييرات |
|:---:|:---|:---|
| **0.1.0** | **أغسطس 2026** | الإصدار الأولي |
| **0.2.0** | **26 أغسطس 2026** | Google OAuth + Cache Reset |
| **0.3.0** | **3 أكتوبر 2026** | TopBar محسّن + Messaging + Polyfill |
| **2.0** | **17 أكتوبر 2026** | **`14-PWA` v2.0** — دمج `22` v0.3.0 + إشارات `08-NOTIFICATIONS` + `06-POINTS` + `04-TRANSLATION` |

### 14.2 روابط ذات صلة

- [00 — MASTER-INDEX](./00-MASTER-INDEX.md)
- [00B — PROJECT-BACKLOG](./00B-PROJECT-BACKLOG.md)
- [01 — ARCHITECTURE](./01-ARCHITECTURE.md)
- [04 — TRANSLATION](./04-TRANSLATION.md)
- [06 — POINTS](./06-POINTS.md)
- [08 — NOTIFICATIONS](./08-NOTIFICATIONS.md)
- [10 — UI-STYLE-GUIDE](./10-UI-STYLE-GUIDE.md)
- [13 — MAUI](./13-MAUI.md)
- [15 — DEPLOYMENT](./15-DEPLOYMENT.md)

---

**© 2026 RubikCare — للاستخدام الداخلي**

**نهاية الوثيقة ✅**
```

---
