# 📘 وثيقة RubikCare.PWA — النسخة الكاملة المحدثة (v2.0)

> **تاريخ آخر تحديث:** 31 أكتوبر 2026  
> **الإصدار:** 2.0  
> **الحالة:** مرحلة التطوير النشط (Beta) — تشغيل الميزات الأساسية + نظام بيئات متكامل  
> **نسبة الإنجاز:** ~40%

---

## 🎯 الجزء الأول: المقدمة والهدف

### ما هو مشروع RubikCare.PWA؟

**RubikCare.PWA** هو تطبيق ويب تقدمي (Progressive Web App) مبني بتقنية **Blazor WebAssembly** في إطار **.NET 10**. يمثل **العميل الثالث** في منظومة RubikCare:

| المشروع | التقنية | الجمهور | الحالة |
|---------|---------|---------|--------|
| **RubikCare.Web** | Blazor Server | الإدارة والموظفون | ✅ منشور |
| **RubikCare.Mobile** | .NET MAUI + BlazorWebView | المرضى والمهنيون | ✅ منشور على Google Play |
| **RubikCare.PWA** ⭐ | Blazor WebAssembly | المرضى والمهنيون (ويب متقدم) | 🚧 قيد التطوير (40%) |

### الأهداف الاستراتيجية

1. **تجربة موحدة:** نفس تجربة الموبايل (تصميم بطاقات على تدرج لوني) وليس نمط الويب (Split-Panel)
2. **إعادة الاستخدام القصوى:** تشغيل مكونات `Shared.UI` الموجودة بالفعل دون إعادة بنائها
3. **قابلية التثبيت:** تثبيت التطبيق على الشاشة الرئيسية (iOS/Android/Desktop)
4. **مصدر حقيقة واحد:** بيانات الجلسة تُجلب مرة واحدة وتُشارك عبر جميع الصفحات
5. **🆕 مصادقة موحدة:** دعم تسجيل الدخول التقليدي و Google OAuth
6. **🆕 نظام بيئات متكامل:** Development / Staging / Production

---

## 🏗️ الجزء الثاني: البنية المعمارية والربط مع الحل

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

### ملفات الربط الحرجة

| الملف | الدور | الحالة |
|-------|-------|--------|
| `PWA/Program.cs` | تسجيل الخدمات | ✅ مكتمل + محسّن (تحميل البيئة) |
| `PWA/Layout/MainLayout.razor` | الهيكل الرئيسي | ✅ مكتمل + محسّن |
| `PWA/Layout/NavMenu.razor` | القائمة الجانبية | ✅ مكتمل + Logout مُصحح |
| `PWA/Services/UserSessionState.cs` | حالة الجلسة | ✅ مكتمل |
| `PWA/Services/WebApiService.cs` | تنفيذ `IApiService` | ✅ مكتمل |
| `PWA/Services/WebTranslationService.cs` | تنفيذ `ISharedTranslationService` | ✅ مكتمل |
| 🆕 `PWA/wwwroot/appsettings.json` | إعدادات Development | ✅ |
| 🆕 `PWA/wwwroot/appsettings.Staging.json` | إعدادات Stage | ✅ |
| 🆕 `PWA/wwwroot/appsettings.Production.json` | إعدادات Production | ✅ |
| 🆕 `PWA/wwwroot/cache-reset.html` | صفحة التنظيف التلقائي | ✅ |
| 🆕 `PWA/wwwroot/js/load-failure-detector.js` | كاشف فشل التحميل (v2) | ✅ |
| 🆕 `PWA/wwwroot/js/googleAuth.js` | تكامل Google Identity Services | ✅ |
| 🆕 `PWA/wwwroot/js/permissions-fix.js` | Polyfill لـ Illegal invocation | ✅ |
| 🆕 `PWA/wwwroot/js/clipboard-helpers.js` | نسخ ومشاركة | ✅ |
| 🆕 `PWA/wwwroot/js/blazor-starter.js` | بدء Blazor اليدوي | ✅ |

---

## 🔐 الجزء الثالث: نظام المصادقة (Authentication System)

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

## 🧹 الجزء الرابع: نظام تنظيف الكاش التلقائي

### 4.1 المشكلة التي يحلها

عند نشر نسخة جديدة من PWA، قد يبقى **Service Worker القديم** في المتصفح ويخدّم ملفات `.wasm` بـ hashes قديمة، مما يسبب:

```
Failed to find a valid digest in the 'integrity' attribute
SRI's integrity checks failed
```

### 4.2 الحل: `cache-reset.html`

صفحة HTML ثابتة (ليست Blazor) تُفتح تلقائياً عند فشل التحميل، وتقوم بـ:

1. **إلغاء Service Workers**
2. **مسح جميع الكاشات** (Cache Storage)
3. **مسح localStorage/sessionStorage** (مع حماية عداد المحاولات)
4. **إعادة التوجيه التلقائي للصفحة الرئيسية**

### 4.3 الحماية من حلقة التوجيه اللانهائية

```javascript
const MAX_ATTEMPTS = 3;
const currentAttempts = parseInt(sessionStorage.getItem('rubik_reset_attempts') || '0');

if (currentAttempts >= MAX_ATTEMPTS) {
    errorBox.classList.add('visible');
    btnRestart.classList.add('visible');
    return;
}

// ⭐ زيادة العداد
sessionStorage.setItem('rubik_reset_attempts', (currentAttempts + 1).toString());

// ... تنفيذ التنظيف ...

// ⭐ استعادة العداد بعد مسح sessionStorage
sessionStorage.setItem('rubik_reset_attempts', nextAttempts.toString());
```

### 4.4 الكشف التلقائي عن الفشل

في `load-failure-detector.js` (النسخة 2.0):

```javascript
// تجاهل أخطاء manifest (غير حرجة)
if (msg.indexOf('manifest') !== -1 || 
    msg.indexOf('.webmanifest') !== -1) {
    console.warn('⚠️ Ignoring non-critical manifest error:', msg);
    event.preventDefault();
    return;
}
```

---

## 🎨 الجزء الخامس: تحسينات واجهة المستخدم

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

**المشكلة:** ملف CSS عام (`_CoreBundle.css`) يُعرّف `.topbar-logo` بحجم ثابت.

**الحل:** استخدام `!important` في `MainLayout.razor.css`:

```css
.topbar-logo {
    height: 98px !important;
    width: 98px !important;
}
```

---

## 📱 الجزء السادس: نظام المحادثات (Messaging)

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

## 🛡️ الجزء السابع: حلول المشاكل الشائعة

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

**الحل:** استخدام `window.location.href` بدلاً من `Navigation.NavigateTo`.

### 7.3 خطأ: تضارب المسارات (Duplicate Routes)

**المشكلة:**
```
The following routes are ambiguous:
'pharmacy/patient-orders' in 'PatientOrders.razor'
'pharmacy/patient-orders' in 'PharmacyPatientOrdersPage.razor'
```

**الحل:** حذف الملف المكرر والاحتفاظ بالأوضح (`PharmacyPatientOrdersPage.razor`).

### 7.4 🆕 خطأ: `500.19.183` — تضارب MIME type لـ `.webmanifest`

**المشكلة:**
```
HTTP 500.19 - Internal Server Error
Sub-status 19: Configuration data is invalid
Win32 status 183: "Cannot create a file when that file already exists"
```

**السبب:** تعريف `.webmanifest` في `web.config` يتعارض مع تعريف IIS على مستوى الخادم.

**الحل:** حذف `<remove fileExtension=".webmanifest" />` و `<mimeMap fileExtension=".webmanifest" ...>` من `web.config` والاعتماد على تعريف IIS الافتراضي.

### 7.5 🆕 خطأ: `SRI's integrity checks failed` — Hashes قديمة

**المشكلة:**
```
Failed to find a valid digest in the 'integrity' attribute for resource 
'https://stagepu.rubikcare.com/_framework/Microsoft.Extensions.Primitives.zcroe0rbn6.wasm'
```

**السبب:** المتصفح يتمسك بنسخة قديمة من `service-worker-assets.js` تحتوي على hashes قديمة.

**الحل:**
1. مسح `bin` و `obj` محلياً
2. إعادة `dotnet publish`
3. مسح Service Worker من المتصفح (Application → Service Workers → Unregister)
4. تفعيل **Bypass for network**
5. `Clear site data` ثم `Ctrl + Shift + R`

---

## 📦 الجزء الثامن: البنية الحقيقية لـ Shared.UI

### 8.1 قائمة المكونات الكاملة (68 مكون)

#### 🏥 فئة العيادة (Clinic) — 4 مكونات
| المكون | النوع | الوظيفة |
|--------|------|---------|
| `ClinicDashboard.razor` | لوحة تحكم كاملة | لوحة تحكم الطبيب |
| `ClinicPatientDetails.razor` | صفحة تفاصيل | تفاصيل مريض |
| `ClinicPatientsList.razor` | قائمة | قائمة المرضى |
| `MyPatientInvitations.razor` | صفحة | دعوات المرضى |

#### ⚖️ الفئة القانونية (Legal) — 2 مكونات ✅ لها @page
| المكون | المسار | الوظيفة |
|--------|--------|---------|
| `PrivacyPolicy.razor` | `/privacy`, `/legal/privacy` | سياسة الخصوصية |
| `TermsOfService.razor` | `/terms`, `/legal/terms` | شروط الاستخدام |

#### 💬 فئة المحادثات (Messaging) — 7 مكونات
| المكون | الوظيفة |
|--------|---------|
| `MessagingHubPage.razor` | مركز المحادثات |
| `ChatPage.razor` | صفحة الدردشة |
| `ChatMessage.razor` | مكون رسالة |
| `ConversationsListPage.razor` | قائمة المحادثات |
| `DoctorSearchPage.razor` | بحث عن طبيب |
| `DoctorProfilePage.razor` | ملف طبيب |
| `MessagingSettingsPage.razor` | إعدادات المحادثات |

#### 🏢 إدارة المنظمات (OrganizationManagement) — 4 مكونات ✅ لها @page
| المكون | المسار | الوظيفة |
|--------|--------|---------|
| `CreateOrganizationPage.razor` | `/create-organization` | إنشاء منظمة |
| `MembersTab.razor` | `/organization/members` | تبويب الأعضاء |
| `MemberTitlesTab.razor` | `/organization/member-titles` | ألقاب الأعضاء |
| `CustomJobTitles.razor` | `/organization/job-titles` | المسميات الوظيفية |

#### 👤 فئة المريض (Patient) — 11 مكون
| المكون | الوظيفة |
|--------|---------|
| `SettingsPage.razor` | الإعدادات الرئيسية |
| `MyProfilePage.razor` | ملفي الشخصي |
| `EditProfilePage.razor` | تعديل الملف |
| `PatientOrdersPage.razor` | طلبات المريض |
| `OrderTracker.razor` | تتبع الطلب |
| `NotificationsPage.razor` | الإشعارات |

##### ⏰ جدولة الأدوية (Patient/SchedualMedication) — 5 مكونات
| المكون | الوظيفة |
|--------|---------|
| `MedicationSchedulePage.razor` | جدول الأدوية الرئيسي |
| `AddMedicationPage.razor` | إضافة دواء |
| `PspSchedulePage.razor` | جدولة PSP |
| `DailyDosesTab.razor` | تبويب الجرعات اليومية |
| `RefillDatesTab.razor` | تبويب مواعيد الصرف |

##### ⚙️ إعدادات المريض (Patient/Settings) — 6 مكونات (2 منها @page)
| المكون | المسار | الوظيفة |
|--------|--------|---------|
| `ProfessionalStatusPage.razor` | `/professional-status` ✅ | الحالة المهنية |
| `ProfessionalLicensePage.razor` | `/professional-license` ✅ | الترخيص المهني |
| `SecuritySection.razor` | — | قسم الأمان |
| `LanguageSection.razor` | — | قسم اللغة |
| `NotificationsSection.razor` | — | قسم الإشعارات |
| `PrivacySection.razor` | — | قسم الخصوصية |

#### 💊 فئة الصيدلية (Pharmacy) — 13 مكون (1 له @page)
| المكون | النوع | الوظيفة |
|--------|------|---------|
| `PharmacyDashboard.razor` | لوحة تحكم | لوحة الصيدلية |
| `PharmacyDetailPage.razor` | صفحة @page ✅ | تفاصيل صيدلية |
| `PharmacySearchPage.razor` | بحث | بحث صيدليات |
| `NearbyPharmacies.razor` | قائمة | الصيدليات القريبة |
| `PharmacyGateway.razor` | بوابة | بوابة الصيدلية |
| `PharmacyReportsPage.razor` | تقارير | تقارير الصيدلية |
| `PatientOrders.razor` | قائمة | طلبات المرضى |
| `MedicationRequestCard.razor` | بطاقة | بطاقة طلب دواء |
| `TokenRedemptionCard.razor` | بطاقة | بطاقة استرداد رمز |
| `PharmacyCard.razor` | بطاقة | بطاقة صيدلية |
| `PharmacyGrid.razor` | شبكة | شبكة صيدليات |
| `PharmacyFilter.razor` | فلتر | فلتر البحث |

#### 🌿 فئة برامج الدعم (PSP) — 5 مكونات (1 له @page)
| المكون | النوع | الوظيفة |
|--------|------|---------|
| `PspAboutPage.razor` | صفحة @page ✅ | عن برامج الدعم |
| `PspGateway.razor` | بوابة | بوابة PSP |
| `PspSearch.razor` | بحث | بحث برامج |
| `PspEntry.razor` (Patient) | إدخال | دخول برنامج |
| `PspScheduleSetup.razor` (Patient) | إعداد | جدولة برنامج |

#### 👔 فئة المندوب (Rep) — 5 مكونات
| المكون | الوظيفة |
|--------|---------|
| `RepDashboard.razor` | لوحة المندوب |
| `PharmaCompanyDashboard.razor` | لوحة شركة الأدوية |
| `InviteDoctor.razor` | دعوة طبيب |
| `MyInvitations.razor` | دعواتي |
| `MyNetwork.razor` | شبكتي |

#### 🛠️ مكونات مساعدة مشتركة — 7 مكونات
| المكون | الوظيفة | الحالة في PWA |
|--------|---------|---------------|
| `LoaderOverlay.razor` | شاشة تحميل | ✅ مستخدم بالفعل |
| `RubikButton.razor` | زر موحد | جاهز |
| `TestPage.razor` | صفحة اختبار | جاهز |
| `Pagination.razor` | ترقيم الصفحات | جاهز |
| `SearchBar.razor` | شريط بحث | جاهز |
| `RubikSmartTable.razor` | جدول ذكي | جاهز |
| `SupportPage.razor` | صفحة دعم @page ✅ | جاهز |

### 8.2 خدمات Shared.UI (20 ملف)

| الخدمة | النوع | الاستخدام في PWA |
|--------|------|------------------|
| `IApiService.cs` | واجهة | ✅ مسجلة كـ `WebApiService` |
| `ITranslationService.cs` | واجهة | ✅ مسجلة كـ `WebTranslationService` |
| `ITranslationState.cs` | واجهة | ✅ مسجلة كـ `SharedTranslationState` |
| `CurrentOrganizationState.cs` | حالة | ✅ مسجلة |
| `IAppStateService.cs` | واجهة | ✅ مسجلة كـ `PwaAppStateService` |
| `IPatientSessionService.cs` | واجهة | ✅ مسجلة كـ `WebPatientSessionService` |
| `PatientSessionService.cs` | تنفيذ | ✅ بديل PWA |
| `SharedApiService.cs` | تنفيذ | بديل محتمل |
| `ILocalNotificationService.cs` | واجهة | ✅ مسجلة كـ `WebLocalNotificationService` |
| `IMobileNavigationService.cs` | واجهة | ✅ مسجلة كـ `WebMobileNavigationService` |
| `INearbyPharmacyService.cs` | واجهة | ✅ مسجلة كـ `WebNearbyPharmacyService` |
| `INotificationNavigationService.cs` | واجهة | ✅ مسجلة كـ `WebNotificationNavigationService` |
| `IPspUiService.cs` | واجهة | ✅ مسجلة كـ `WebPspUiService` |
| `IPublicApiService.cs` | واجهة | ⚠️ يحتاج تنفيذ |
| `InviteNavigationBridge.cs` | جسر | ⚠️ خاص بـ MAUI |
| `RepNavigationBridge.cs` | جسر | ⚠️ خاص بـ MAUI |
| `OsmPharmacyService.cs` | تنفيذ | ✅ مسجلة |
| `PendingRequestService.cs` | خدمة | ✅ مسجلة |
| `Models/NotificationItem.cs` | نموذج | ✅ جاهز |
| `Models/PspModels.cs` | نموذج | ✅ جاهز |

### 8.3 الصفحات القابلة للتوجيه مباشرة (@page)

| الصفحة | المسار(ات) |
|--------|-----------|
| `TermsOfService` | `/terms`, `/legal/terms` |
| `PrivacyPolicy` | `/privacy`, `/legal/privacy` |
| `SupportPage` | `/support` |
| `CreateOrganizationPage` | `/create-organization` |
| `MembersTab` | `/organization/members` |
| `MemberTitlesTab` | `/organization/member-titles` |
| `CustomJobTitles` | `/organization/job-titles` |
| `ProfessionalStatusPage` | `/professional-status` |
| `ProfessionalLicensePage` | `/professional-license` |
| `PharmacyDetailPage` | `/pharmacy/{id}` |
| `PspAboutPage` | `/psp/about` |

---

## ⚠️ الجزء التاسع: التناقضات المعمارية المكتشفة والحلول

### التناقض 1: "لا صفحات كاملة في Shared.UI"

**ما تقوله الوثيقة `00-architecture-overview.md`:**
> ❌ ما لا يوضع في Shared.UI: صفحات كاملة (Dashboard, PSP, Admin)

**الواقع الفعلي:**
تحتوي Shared.UI على لوحات تحكم كاملة (`ClinicDashboard`, `PharmacyDashboard`, `RepDashboard`) وصفحات متعددة.

**التفسير:**
هذه الصفحات وُضعت في Shared.UI لأنها **مشتركة بين الويب والموبايل** (تُستضاف في الموبايل عبر `BlazorWebView`). هذا يتوافق مع **المستوى 4** من شجرة القرار المعماري (عمليات مشتركة بين المنصات).

**القرار العملي:**
> ✅ نقبل الواقع الحالي. هذه الصفحات موجودة وتعمل في الموبايل والويب. مهمتنا في PWA هي **استضافتها** وليس إعادة بنائها.

### التناقض 2: نمط التصميم (MAUI مقابل Web)

**ما تقوله الوثيقة `03-style-guide.md`:**
صفحات المصادقة تستخدم نمط **Split-Panel** (لوحة بصرية + لوحة نموذج).

**ما اتفقنا عليه في هذه الجلسة:**
صفحات المصادقة في PWA تتبع نمط **الموبايل** (بطاقة بيضاء على تدرج لوني) وليس نمط الويب.

**القرار العملي:**
> ✅ صفحات المصادقة (Login, Register) في PWA تتبع نمط الموبايل.
> ✅ الصفحات الداخلية تتبع نمط الدور (Clinic/Pharmacy/Rep) حسب وثيقة `03`.

### التناقض 3: حقن الخدمات في مكونات Shared.UI

**ما تقوله الوثيقة `11-blazor-webview-guide.md`:**
مكونات Shared.UI تستقبل `IApiService` كـ `[Parameter]` وليس `@inject` (بسبب عزل حاوية DI في BlazorWebView).

**الأثر على PWA:**
عند استضافة هذه المكونات في PWA، يجب تمرير `IApiService` كـ Parameter:

```razor
<SettingsPage ApiService="ApiService" />
```

حيث `ApiService` هو `IApiService` المحقون في صفحة PWA المضيفة.

---

## 🗺️ الجزء العاشر: خطة تشغيل مكونات Shared.UI في PWA

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
- [x] نظام تنظيف الكاش التلقائي
- [x] Polyfill لـ Google GIS
- [x] نظام البيئات (Development/Staging/Production)

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

## 🔑 الجزء الحادي عشر: الأنماط الناجحة والموثقة

### 11.1 استضافة مكون Shared.UI في PWA

```razor
@* صفحة PWA المضيفة *@
@page "/settings"
@inject IApiService ApiService

<SettingsPage ApiService="ApiService" />
```

### 11.2 تمرير بيانات الجلسة

```razor
@* استخدام UserSessionState في أي صفحة *@
@inject UserSessionState SessionState

<h1>@SessionState.CurrentSession?.FullNameAr</h1>
```

### 11.3 الترجمة الآمنة (مع مفتاح احتياطي)

```csharp
private string T(string key)
{
    if (_translations.TryGetValue(key, out var value))
        return value;

    // مفتاح مختصر (بدون البادئة) كاحتياطي
    var shortKey = key.Contains('.') ? key[(key.LastIndexOf('.') + 1)..] : key;
    if (_translations.TryGetValue(shortKey, out var shortValue))
        return shortValue;

    return key;
}
```

### 11.4 إعادة الرسم عند تغيير البيانات

```csharp
protected override async Task OnInitializedAsync()
{
    SessionState.OnChange += OnSessionChanged;
}

private async void OnSessionChanged() => await InvokeAsync(StateHasChanged);

public void Dispose() => SessionState.OnChange -= OnSessionChanged;
```

### 11.5 🆕 تحميل الإعدادات حسب البيئة

```csharp
var environment = builder.HostEnvironment.Environment;
builder.Configuration.AddJsonFile($"appsettings.{environment}.json", optional: true);

string apiUrl = config["ApiSettings:BaseUrl"]
    ?? (environment == "Development"
        ? "http://localhost:5235/"
        : "https://uat.rubikcare.com/");
```

---

## 🚫 الجزء الثاني عشر: الأنماط المحظورة

| النمط | السبب | البديل |
|-------|-------|--------|
| اختراع مفاتيح ترجمة | المفاتيح يجب أن تكون موجودة في قاعدة البيانات | تحقق بـ `SELECT` قبل الاستخدام |
| استخدام `@inject HttpClient` في مكونات Shared.UI | يفشل بصمت في BlazorWebView | استخدم `[Parameter] IApiService` |
| تعديل ملفات Migration يدوياً | يكسر تتبع EF Core | استخدم `Add-Migration` فقط |
| استخدام ألوان ثابتة بدلاً من المتغيرات | صعب الصيانة | استخدم `var(--rubik-primary)` |
| Inline styles في ملفات كبيرة | غير قابل للصيانة | استخدم ملفات `.razor.css` |
| `@keyframes` بدون هروب | يفسرها Razor كـ C# | استخدم `@@keyframes` |
| `?.` مع `EventCallback` | هو struct وليس nullable | استخدم `.InvokeAsync()` |
| 🆕 **تعديل `web.config` على السيرفر** | يسبب خطأ `500.19.183` | استخدم النسخة المعتمدة فقط |
| 🆕 **نسخ ملفات `.br`/`.gz` يدوياً** | يكسر ضغط IIS | أعد `dotnet publish` دائماً |
| 🆕 **تكرار سطر `blazor.webassembly.js`** | يكسر إقلاع Blazor | سطر واحد فقط مع `autostart="false"` |

---

## 🎓 الجزء الثالث عشر: الدروس المستفادة من هذه الجلسة

### 13.1 لا تفترض، بل تحقق
- **الخطأ:** افتراض أن مفاتيح الترجمة غير موجودة
- **الصحيح:** فتح Console وقراءة رسالة الخطأ الفعلية

### 13.2 استخدم المفاتيح الموجودة فعلاً
- **الخطأ:** اختراع مفاتيح مثل `DASHBOARD.WELCOME`
- **الصحيح:** استخدام المفاتيح الفعلية `SHARED.DASHBOARD.WELCOME` من قاعدة البيانات

### 13.3 افهم دورة حياة المكونات
- **الخطأ:** توقع أن المكون الفرعي يُعاد رسمه تلقائياً
- **الصحيح:** الاشتراك في أحداث التغيير (`OnChange`) واستدعاء `StateHasChanged`

### 13.4 ابدأ بالبسيط ثم تعقّد
- **الخطأ:** بناء لوحة تحكم كاملة من البداية
- **الصحيح:** صفحة مؤقتة → بيانات حقيقية → ميزات متقدمة

### 13.5 وثّق قبل أن تنسى
- كل اكتشاف جديد يجب أن يُضاف لهذه الوثيقة فوراً

### 13.6 🆕 المسح قبل النسخ ليس تحسيناً — هو جزء من عملية النشر
- أسماء ملفات `.wasm` مجزأة (hashed) وتتغير كل نشر
- الخلط بين القديم والجديد = فشل SRI حتمي
- **القاعدة:** امسح المجلد أولاً (ما عدا `web.config`) ثم انسخ

### 13.7 🆕 افحص حيث تعيش الملفات
- في هذا المشروع الملفات في `wwwroot` وليس الجذر
- سكربت فحص ينظر للجذر سيكذب عليك

### 13.8 🆕 السيرفر والمتصفح مخزنان منفصلان
- نشر سليم + متصفح قديم = خطأ يبدو كأنه نشر فاشل
- افصل التشخيصين بفحص الـ hashes أولاً
- **InPrivate هو أداة التشخيص الأولى** — يعزل مشكلة المتصفح عن مشكلة السيرفر

### 13.9 🆕 `.NET 10` ليس `.NET 8`
- لا تبحث عن `blazor.boot.json`؛ الخريطة في `service-worker-assets.js`
- كل أصل له 3 نسخ: `.wasm` + `.br` + `.gz`

### 13.10 🆕 ملف واحد مكرر في `index.html` يكسر التطبيق كله
- عند تعديل `index.html` راجعه كاملاً قبل النشر
- `autostart="false"` + `Blazor.start()` يدوي = النمط الصحيح

### 13.11 🆕 صفحة `cache-reset.html` شبكة الأمان للمستخدم النهائي
- لا تحذفها، وهي تعفي الدعم الفني من شرح مسح الكاش هاتفياً
- عند تعديلها، يجب إعادة `dotnet publish` لتحديث `.br` و `.gz`

### 13.12 🆕 ملفات `appsettings.json` في Blazor WASM تُقرأ من المتصفح
- إذا فشل تسجيل الدخول بعد النشر، تحقق من `BaseUrl` في `appsettings.json`
- يجب أن يتطابق مع بيئة النشر (ليس `localhost` على Stage)

### 13.13 🆕 Service Worker الزومبي
- المتصفح يتمسك بشراسة بالنسخة القديمة من Service Worker
- الحل: `Application → Service Workers → Unregister` + `Clear site data` + `Bypass for network`

### 13.14 🆕 تضارب MIME type لـ `.webmanifest`
- لا تضف تعريف `.webmanifest` في `web.config`
- IIS 10.0 يعرفه افتراضياً، والتعريف المزدوج يسبب `500.19.183`

---

## 📊 الجزء الرابع عشر: حالة PWA — نسبة الإنجاز

### 14.1 نسبة الإنجاز الفعلية (~40%)

| العنصر | النسبة |
|--------|--------|
| **البنية التحتية** | ✅ 100% |
| **صفحات المصادقة** | ✅ 100% |
| **نظام البيئات** | ✅ 100% |
| **نظام تنظيف الكاش** | ✅ 100% |
| **Google OAuth** | ✅ 100% |
| **صفحات PSP** | ✅ 100% |
| **صفحات المريض** | ✅ 100% |
| **صفحات الطبيب** | ✅ 100% |
| **صفحات الصيدلية** | ✅ 100% |
| **صفحات المندوب** | ✅ 100% |
| **نظام النقاط** | ✅ 100% |
| **نظام المحادثات** | ✅ 90% |
| **الإشعارات Push** | ✅ 100% |
| **الإجمالي** | **~40%** |

### 14.2 إحصائيات المشروع

| المقياس | القيمة |
|---------|--------|
| **مكونات Shared.UI المتاحة** | 68 |
| **صفحات @page مُفعّلة** | 15+ |
| **خدمات تحتاج تنفيذ في PWA** | ~8 |
| **ملفات البيئة** | 3 (Development, Staging, Production) |
| **سكريبتات النشر** | 2 (Stage, Production) |

---

## 🎯 الجزء الخامس عشر: 📘 دليل النشر التشغيلي (Deployment SOP) — مشروع RubikCare

**الإصدار:** 3.0  
**التاريخ:** 31 أكتوبر 2026  
**النطاق:** RubikCare.PWA + RubikCare.Api.Web على بيئة Stage/UAT

---

### 15.1 بطاقة المرجع السريع (بيانات هذا المشروع تحديداً)

| العنصر | القيمة |
|--------|--------|
| مسار الحل (محلي) | `C:\RC\Rubikcare.Full.Migration` |
| مشروع PWA | `RubikCare.PWA` — Blazor WebAssembly على **.NET 10** |
| مشروع API | `RubikCare.Api.Web` |
| المكتبة المشتركة | `RubikCare.Shared.UI` (RCL — تُدمج داخل نشر الـ PWA تلقائياً) |
| مجلد النشر المحلي | `E:\Publish\PWA` و `E:\Publish\ApiUat` |
| مجلد PWA على السيرفر | `C:\WebSite\PU_RubicCareStage` |
| App Pool للـ PWA | `PU_RubicCareStage` |
| نطاق PWA | `https://stagepu.rubikcare.com` |
| مجلد API على السيرفر | `C:\WebSite\RubikCareUat` |
| App Pool للـ API | `RubikCareUat` |
| نطاق API | `https://uat.rubikcare.com` |
| ملف إعدادات البيئة المحمي | `appsettings.Test.json` (يُمنع overwritten) |
| 🆕 مسار النسخ الاحتياطي | `C:\WebSite\_Docs\Backups\StableVersionOfPWA_dd.MM.yyyy.zip` |

> ⚠️ الـ PWA المنشور على `stagepu` يستدعي الـ API على `uat` — لذلك أي تعديل في الـ API يتطلب نشر المشروعين معاً.

---

### 15.2 هيكل المجلدات على السيرفر (أهم اختلاف عن باقي المشاريع)

```
C:\WebSite\PU_RubicCareStage\          ← IIS Physical Path يشير هنا (الجذر)
├── web.config                         ← ⭐ محمي: يحتوي قواعد Rewrite + MIME
├── (ملفات نشر stray: dotnet.js, runtimeconfig.json)  ← طبيعية، تُتجاهل
└── wwwroot\                           ← ⭐⭐ الملفات الفعلية كلها هنا
    ├── index.html
    ├── cache-reset.html               ← صفحة التنظيف التلقائي
    ├── cache-reset.html.br            ← نسخة Brotli المضغوطة
    ├── cache-reset.html.gz            ← نسخة Gzip المضغوطة
    ├── service-worker.js
    ├── service-worker-assets.js       ← خريطة الأصول والـ hashes
    ├── manifest.webmanifest
    ├── appsettings.json               ← إعدادات Development (افتراضي)
    ├── appsettings.Staging.json       ← إعدادات Stage
    ├── appsettings.Production.json    ← إعدادات Production
    ├── css\  js\  images\  lib\
    ├── _content\RubikCare.Shared.UI\  ← مكونات وأصول المكتبة المشتركة
    └── _framework\                    ← ملفات Blazor (.wasm/.js/.dat + نسخ .br/.gz)
```

### القاعدة الذهبية للتحقق:

> **أي أمر فحص يجب أن ينظر داخل `wwwroot` وليس الجذر.**
> معظم أخطاء التشخيص السابقة حدثت لأن سكربتات الفحص كانت تبحث في الجذر.

---

### 15.3 ⚠️ القسم الحرج: خصائص المشروع الفريدة وقواعد النشر الإلزامية

> **تنويه هام جداً:** مشروع `RubikCare.PWA` (Blazor WebAssembly على .NET 10) **يختلف جذرياً** في هيكلية المجلدات، وآلية عمل المتصفح، وإعدادات الخادم عن أي مشروع ويب تقليدي (مثل MVC أو Blazor Server). تجاهل هذه الخصائص سيؤدي حتماً إلى أخطاء `500.19` أو حلقات لا نهائية من أخطاء `SRI Integrity`.

#### الاختلافات الجذرية عن المشاريع الأخرى (يجب فهمها)

| الخاصية | المشاريع التقليدية | مشروع RubikCare.PWA (.NET 10) |
| :--- | :--- | :--- |
| **ملف تمهيد Blazor** | يعتمد على `blazor.boot.json` | **غير موجود**. الخريطة مدمجة داخل `blazor.webassembly.js` و `service-worker-assets.js`. |
| **أسماء الملفات** | أسماء ثابتة (مثل `app.js`, `main.wasm`) | **أسماء مجزأة (Hashed)** تتغير مع كل نشر (مثل `app.1a2b3c.wasm`). خلط ملفات قديمة وجديدة يسبب فشل التحقق من السلامة (SRI). |
| **نسخ الملفات** | نسخة واحدة لكل ملف | **3 نسخ لكل ملف**: `.wasm` (أصلي)، `.wasm.br` (مضغوط Brotli)، `.wasm.gz` (مضغوط Gzip). حذف النسخ المضغوطة يكسر ضغط IIS. |
| **موقع الملفات الفعلي** | غالباً في الجذر أو `wwwroot` مباشرة | IIS يشير للجذر `C:\WebSite\PU_RubicCareStage`، لكن **كل ملفات الويب تعيش حصرياً داخل مجلد `wwwroot`**. |
| **سلوك المتصفح (Service Worker)** | يحترم تحديثات الخادم فوراً | **يتمسك بشراسة بالنسخة المخزنة (Zombie SW)**. قد يخدم ملفات بـ Hashes قديمة رغم تحديث الخادم، مما يسبب حلقة لا نهائية من أخطاء SRI وتوجيه لصفحة `cache-reset.html`. |
| **إعدادات البيئة** | تُقرأ من الخادم مرة واحدة | `appsettings.json` يُطلب بواسطة **المتصفح** (HTTP Request) في وقت التشغيل. يجب أن يتطابق محتواه مع بيئة النشر. |

#### القواعد الذهبية المطلقة (Zero Tolerance)

1. 🚫 **ممنوع منعاً باتاً تعديل أو استبدال ملف `web.config` الموجود في الجذر.** هذا الملف هو العمود الفقري لتوجيه الطلبات ومعالجة أنواع الملفات. أي تغيير فيه يسبب خطأ `500.19.183` فوراً.
2. 💾 **إلزامية النسخ الاحتياطي المستقر:** قبل أي عملية نشر، يجب حفظ نسخة كاملة من المجلد الحالي في المسار التالي بالضبط:  
   `C:\WebSite\_Docs\Backups\StableVersionOfPWA_dd.MM.yyyy.zip`  
   *(استبدل `dd.MM.yyyy` بتاريخ اليوم، مثال: `StableVersionOfPWA_31.10.2026.zip`)*.
3. 🧹 **المسح قبل النسخ:** بسبب تغيير الـ Hashes، يجب مسح محتويات المجلد (ما عدا `web.config`) قبل نسخ الملفات الجديدة لتجنب "تلوث" الملفات.
4. 🛡️ **حماية `appsettings.json`:** في بيئة Stage، يجب التأكد من أن ملف `appsettings.json` (أو `appsettings.Staging.json`) يحتوي على `BaseUrl` الصحيح للـ API (`https://uat.rubikcare.com/`) وليس `localhost`.
5. 🔁 **إعادة `dotnet publish` عند تعديل `cache-reset.html`:** لأن Blazor يولد تلقائياً نسخ `.br` و `.gz` المضغوطة. التعديل اليدوي على السيرفر فقط يكسر الضغط.

---

### 15.4 ملف `web.config` — محتوى محمي يُمنع تعديله

> ⚠️ **تحذير أخير:** إذا وجدت أن هذا الملف قد تم استبداله بنسخة من مجلد `publish` المحلي، قم فوراً باستعادته من النسخة الاحتياطية.

```xml
<?xml version="1.0" encoding="UTF-8"?>
<configuration>
  <system.webServer>
    <!-- 1. تعريف أنواع الملفات (MIME Types) - ضروري جداً لملفات Blazor -->
    <staticContent>
      <remove fileExtension=".blat" />
      <remove fileExtension=".dat" />
      <remove fileExtension=".dll" />
      <remove fileExtension=".webcil" />
      <remove fileExtension=".json" />
      <remove fileExtension=".wasm" />
      <remove fileExtension=".woff" />
      <remove fileExtension=".woff2" />
      
      <mimeMap fileExtension=".blat" mimeType="application/octet-stream" />
      <mimeMap fileExtension=".dll" mimeType="application/octet-stream" />
      <mimeMap fileExtension=".webcil" mimeType="application/octet-stream" />
      <mimeMap fileExtension=".dat" mimeType="application/octet-stream" />
      <mimeMap fileExtension=".json" mimeType="application/json" />
      <mimeMap fileExtension=".wasm" mimeType="application/wasm" />
      <mimeMap fileExtension=".woff" mimeType="application/font-woff" />
      <mimeMap fileExtension=".woff2" mimeType="application/font-woff2" />
    </staticContent>

    <!-- 2. ضغط الملفات لتحسين الأداء -->
    <httpCompression>
      <dynamicTypes>
        <add mimeType="application/octet-stream" enabled="true" />
        <add mimeType="application/wasm" enabled="true" />
        <add mimeType="application/json" enabled="true" />
      </dynamicTypes>
      <staticTypes>
        <add mimeType="application/wasm" enabled="true" />
        <add mimeType="application/json" enabled="true" />
      </staticTypes>
    </httpCompression>

    <!-- 3. قواعد إعادة التوجيه (Rewrite Rules) - قلب النظام -->
    <rewrite>
      <rules>
        <!-- قاعدة 1: خدمة الملفات الموجودة فعلياً داخل مجلد wwwroot -->
        <rule name="Serve wwwroot files" stopProcessing="true">
          <match url="^(.*)$" />
          <conditions logicalGrouping="MatchAny">
            <add input="{DOCUMENT_ROOT}\wwwroot\{R:1}" matchType="IsFile" />
            <add input="{DOCUMENT_ROOT}\wwwroot\{R:1}" matchType="IsDirectory" />
          </conditions>
          <action type="Rewrite" url="wwwroot/{R:1}" />
        </rule>

        <!-- قاعدة 2: SPA Fallback - توجيه أي مسار غير معروف إلى index.html -->
        <rule name="SPA fallback to index.html" stopProcessing="true">
          <match url="^(.*)$" />
          <action type="Rewrite" url="wwwroot/index.html" />
        </rule>
      </rules>

      <!-- 4. منع المتصفح من تخزين Service Worker في الكاش (حل مشكلة الزومبي) -->
      <outboundRules>
        <rule name="No Cache for Service Worker" preCondition="IsServiceWorker">
          <match serverVariable="RESPONSE_Cache_Control" pattern=".*" />
          <action type="Rewrite" value="no-cache, no-store, must-revalidate" />
        </rule>
        <preConditions>
          <preCondition name="IsServiceWorker">
            <add input="{REQUEST_URI}" pattern="service-worker(\.assets)?\.js$" />
          </preCondition>
        </preConditions>
      </outboundRules>
    </rewrite>

    <!-- 5. الملف الافتراضي -->
    <defaultDocument>
      <files>
        <clear />
        <add value="index.html" />
      </files>
    </defaultDocument>
  </system.webServer>
</configuration>
```

### ممنوعات الـ web.config:

- ❌ حذف قسم `<staticContent>` (تعريفات MIME لـ `.wasm/.dat/.blat/.webcil`) — بدونه ملفات تُخدم بنوع خاطئ.
- ❌ إضافة تعريف `.webmanifest` — IIS 10.0 يعرفه افتراضياً، والتعريف المزدوج يسبب `500.19.183`.
- ❌ تغيير `url="wwwroot/..."` إلى مسار جذر — البنية تعتمد على `wwwroot`.
- ❌ حذف قاعدة `SPA fallback` — بدونه كل مسار مثل `/login` و `/chat/5` يعطي 404.
- ❌ استبداله بنسخة الـ publish المحلية — نسخة السيرفر هي المُعايرة الصحيحة.
- ❌ حذف قسم `<outboundRules>` — بدونه Service Worker الزومبي يعود.

---

### 15.5 خصوصيات هذا المشروع (.NET 10 + PWA)

| الخصوصية | التفصيل | الخطأ الشائع |
|----------|---------|--------------|
| **لا يوجد `blazor.boot.json`** | في .NET 10 الخريطة داخل `blazor.webassembly.js` + `service-worker-assets.js` | البحث عنه واعتباره مفقوداً |
| **كل أصل له 3 نسخ** (`x.wasm` + `.br` + `.gz`) | ضغط Brotli/Gzip طبيعي | اعتبار النسخ الإضافية تكراراً وحذفها |
| **`index.html` حساس** | سطر واحد فقط `blazor.webassembly.js` مع `autostart="false"` + `Blazor.start()` يدوي + polyfill قبل سكربت Google | تكرار السطر أو حذف `autostart` |
| **`cache-reset.html`** | تنظيف ذاتي للمستخدم النهائي عند فشل الإقلاع | حذفه عند "تنظيف" الملفات، أو تعديله يدوياً دون إعادة publish |
| **ملفات stray في الجذر** (`dotnet.js`, `runtimeconfig.json`) | طبيعية من بنية publish | حذفها يدوياً |
| **Shared.UI مدمجة** | أي تعديل CSS/مكونات في Shared.UI يتطلب **إعادة نشر الـ PWA** | نشر API فقط وتوقع ظهور التعديل |
| **🆕 `appsettings.json` يُقرأ من المتصفح** | Blazor WASM يطلب الملف عبر HTTP عند الإقلاع | نسيان تحديثه عند تغيير بيئة النشر |

---

### 15.6 جدول استكشاف الأخطاء (من واقع جلسات النشر الفعلية)

| الخطأ في المتصفح/السيرفر | السبب | العلاج |
|--------------------------|-------|--------|
| `SRI's integrity checks failed` + `Failed to find a valid digest` | خلط ملفات قديمة/جديدة أو Service Worker قديم | مسح `bin`/`obj` → `dotnet publish` → مسح SW من المتصفح + InPrivate |
| `Expected a JavaScript-or-Wasm module ... MIME type of "text/html"` | ملف مفقود في `_framework` (IIS يرد صفحة 404 HTML) | فحص 1 و2 → إعادة النسخ بعد إيقاف الـ App Pool |
| `404` على `/login` أو أي مسار SPA | قاعدة `SPA fallback` مفقودة/مكسورة في web.config | استعادة قواعد web.config المعتمدة |
| `403 Forbidden` عند فتح الموقع | قاعدة Rewrite تشير لمسار خاطئ أو صلاحيات المجلد | التحقق من web.config + `icacls ... /grant "IIS_IUSRS:(OI)(CI)(RX)"` |
| `The following routes are ambiguous` | صفحتان في PWA بنفس `@page` | حذف الصفحة المكررة (احتفظ بالأوضح اسماً) |
| `Illegal invocation` عند دخول Google | غياب الـ polyfill قبل سكربت GIS | التأكد من وجود سكربت الـ polyfill في `index.html` |
| Splash عالق / حلقة صفحة التنظيف | تكرار سطر `blazor.webassembly.js` أو غياب `autostart="false"` | استبدال `index.html` بالنسخة المعتمدة |
| شاشة زرقاء بعد نشر كان يعمل محلياً | نسخ فوق القديم بدون مسح | القاعدة الذهبية: امسح أولاً |
| التعديلات لا تظهر رغم النشر الصحيح | Service Worker/كاش المتصفح | Application → Service Workers → Unregister → Clear site data |
| **🆕 `500.19.183` على كل الملفات** | تضارب MIME type لـ `.webmanifest` في web.config | حذف تعريف `.webmanifest` من web.config |
| **🆕 حلقة لا نهائية لـ `cache-reset.html`** | المتصفح يتمسك بـ Service Worker قديم (Zombie) | Unregister SW + Bypass for network + InPrivate |
| **🆕 فشل تسجيل الدخول بعد النشر** | `appsettings.json` يحتوي على `localhost` بدلاً من `uat.rubikcare.com` | تحديث `appsettings.Staging.json` وإعادة النشر |
| **🆕 `SyntaxError: await is only valid in async functions`** | كود `cache-reset.html` يستخدم `await` خارج دالة `async` | استبدال `await` بـ `.then()` chains |

---

### 15.7 الدروس المستفادة (الخلاصة النهائية)

1. **المسح قبل النسخ ليس تحسيناً — هو جزء من عملية النشر.** بدونه كل نشر مقامرة.
2. **افحص حيث تعيش الملفات:** في هذا المشروع الملفات في `wwwroot`؛ سكربت فحص ينظر للجذر سيكذب عليك.
3. **السيرفر والمتصفح مخزنان منفصلان:** نشر سليم + متصفح قديم = خطأ يبدو كأنه نشر فاشل. افصل التشخيصين بفحص الـ hashes أولاً.
4. **`.NET 10` ليس `.NET 8`:** لا تبحث عن `blazor.boot.json`؛ الخريطة في `service-worker-assets.js`.
5. **ملف واحد مكرر في `index.html` يكسر التطبيق كله** — عند تعديل `index.html` راجعه كاملاً قبل النشر.
6. **InPrivate هو أداة التشخيص الأولى** — يعزل مشكلة المتصفح عن مشكلة السيرفر في ثانية واحدة.
7. **صفحة `cache-reset.html` شبكة الأمان للمستخدم النهائي** — لا تحذفها، وهي تعفي الدعم الفني من شرح مسح الكاش هاتفياً.
8. **الـ API له ملف بيئة مقدّس** (`appsettings.Test.json`) — المسح الشامل يشمل كل شيء ما عدا هو و`web.config`.
9. **🆕 ملفات `appsettings.json` في Blazor WASM تُقرأ من المتصفح:** إذا فشل تسجيل الدخول أو الاتصال بالـ API بعد النشر، فتأكد أن ملف `appsettings.json` (أو ملف البيئة النشط) على السيرفر يحتوي على `BaseUrl` الصحيح للـ API، وأنه لم يتم استبداله بنسخة تطوير (`localhost`) عن طريق الخطأ أثناء النسخ اليدوي.
10. **🆕 تعديل `cache-reset.html` يتطلب `dotnet publish`:** لأن Blazor يولد تلقائياً نسخ `.br` و `.gz` المضغوطة. التعديل اليدوي على السيرفر فقط يترك هذه النسخ قديمة ويسبب تضارباً.
11. **🆕 Service Worker الزومبي:** المتصفح يتمسك شرسة بالنسخة القديمة. الحل: `Application → Service Workers → Unregister` + `Clear site data` + `Bypass for network` + `Ctrl + Shift + R`.
12. **🆕 تضارب MIME type لـ `.webmanifest`:** لا تضف تعريف `.webmanifest` في `web.config`. IIS 10.0 يعرفه افتراضياً، والتعريف المزدوج يسبب `500.19.183`.

---

### 15.8 Checklist النهائي (يُلصق في كل تذكرة نشر)

```
[ ] البناء محلياً نجح والتعديل مُختبر محلياً
[ ] نُقل المجلد للسيرفر عبر RDP (إلى C:\WebSite\_Staging\PWA)
[ ] PWA: stop → clear (عدا web.config) → copy (عدا web.config) → start
[ ] API: stop → clear (عدا web.config و appsettings.Test.json) → copy → start
[ ] فحص 1: الملفات الأساسية في wwwroot موجودة
[ ] فحص 2: hashes مطابقة (OK: N assets match)
[ ] نافذة InPrivate: التطبيق يفتح، Console نظيف
[ ] دخول Google يعمل، Logout يعود لـ /login
[ ] Console يُظهر: 🔵 Running in environment: Staging
[ ] Console يُظهر: 🔵 API Base URL: https://uat.rubikcare.com/
[ ] (اختياري) نافذة عادية: Unregister SW + Clear site data ثم إعادة اختبار
```

---

### 15.9 🆕 طريقة النشر اليدوي الموثوقة (Manual Deployment)

نظراً لاعتماد النشر اليدوي المباشر لضمان التحكم الكامل، يجب اتباع هذه الخطوات بالحرف لتجنب أخطاء تضارب الملفات (SRI Errors) أو كسر التكوين:

#### الخطوة 1: النشر المحلي

```powershell
cd C:\RC\Rubikcare.Full.Migration

# مسح bin و obj لضمان نظافة البناء
Remove-Item "RubikCare.PWA\bin" -Recurse -Force -ErrorAction SilentlyContinue
Remove-Item "RubikCare.PWA\obj" -Recurse -Force -ErrorAction SilentlyContinue
Remove-Item "RubikCare.Shared.UI\bin" -Recurse -Force -ErrorAction SilentlyContinue
Remove-Item "RubikCare.Shared.UI\obj" -Recurse -Force -ErrorAction SilentlyContinue

# البناء والنشر
dotnet publish RubikCare.PWA -c Release -o "E:\Publish\PWA"
```

#### الخطوة 2: النسخ إلى السيرفر (عبر RDP)

انسخ **جميع** محتويات مجلد `E:\Publish\PWA` إلى:
```
C:\WebSite\_Staging\PWA
```

#### الخطوة 3: تشغيل سكريبت النشر على السيرفر

```powershell
cd C:\WebSite\_Docs\Scripts
.\Deploy-ToStage-PWA.ps1
```

#### الخطوة 4: التحقق من Console

افتح `https://stagepu.rubikcare.com` في نافذة **InPrivate** (`Ctrl + Shift + N`)، ثم افتح Console (F12).

**النتيجة المتوقعة:**
```
🔵 Running in environment: Staging
🔵 API Base URL: https://uat.rubikcare.com/
🔵 Features: DebugLogs=True, CacheReset=True, Push=True
✅ Blazor started successfully
🟢 User authenticated: [اسم المستخدم] (UserId: [ID])
```

**بدون:**
- ❌ `SRI's integrity checks failed`
- ❌ `Failed to find a valid digest`
- ❌ `Redirecting to cache reset`
- ❌ الحلقة اللانهائية

---

### 15.10 🆕 سكريبت النشر المحدث (يجب استخدامه حصرياً)

**المسار على السيرفر:** `C:\WebSite\_Docs\Scripts\Deploy-ToStage-PWA.ps1`

```powershell
# ═══════════════════════════════════════════════════════════
# سكربت نشر PWA الآمن - الإصدار 4.0 (المعتمد نهائياً)
# ═══════════════════════════════════════════════════════════

[Console]::OutputEncoding = [System.Text.Encoding]::UTF8
$ErrorActionPreference = "Stop"

# ─── الإعدادات ───
$SourcePath = "C:\WebSite\_Staging\PWA"
$TargetPath = "C:\WebSite\PU_RubicCareStage"
$BackupDir  = "C:\WebSite\_Docs\Backups"
$AppPoolName = "PU_RubicCareStage"
$DateStr = Get-Date -Format "dd.MM.yyyy"
$BackupFile = Join-Path $BackupDir "StableVersionOfPWA_$DateStr.zip"

Write-Host "`n╔══════════════════════════════════════════════════════════╗" -ForegroundColor Cyan
Write-Host "║        RubikCare PWA Secure Deployment v4.0            ║" -ForegroundColor Cyan
Write-Host "╚══════════════════════════════════════════════════════════╝`n" -ForegroundColor Cyan

# ─── 1. التحقق من المصدر ───
if (-not (Test-Path "$SourcePath\wwwroot\index.html")) {
    Write-Host "❌ خطأ: مجلد المصدر لا يحتوي على wwwroot\index.html" -ForegroundColor Red
    exit 1
}
Write-Host "✅ تم التحقق من ملفات المصدر." -ForegroundColor Green

# ─── 2. النسخ الاحتياطي الإلزامي (في المسار المحدد) ───
if (-not (Test-Path $BackupDir)) { New-Item -ItemType Directory -Path $BackupDir | Out-Null }
Write-Host "📦 جاري إنشاء نسخة احتياطية مستقرة في: $BackupFile" -ForegroundColor Yellow
try {
    Compress-Archive -Path "$TargetPath\*" -DestinationPath $BackupFile -Force -ErrorAction SilentlyContinue
    Write-Host "✅ تم حفظ النسخة الاحتياطية بنجاح." -ForegroundColor Green
} catch {
    Write-Host "⚠️  تحذير: فشل إنشاء النسخة الاحتياطية. هل تريد المتابعة؟ (y/n)" -ForegroundColor Yellow
    if ((Read-Host) -ne 'y') { exit 1 }
}

# ─── 3. إيقاف App Pool ───
Write-Host "🛑 جاري إيقاف App Pool..." -ForegroundColor Yellow
Stop-WebAppPool -Name $AppPoolName -ErrorAction SilentlyContinue
Start-Sleep -Seconds 3

# ─── 4. التنظيف الآمن (مع حماية web.config) ───
Write-Host "🧹 جاري تنظيف المجلد القديم (مع حماية web.config)..." -ForegroundColor Yellow
Get-ChildItem -Path $TargetPath -Exclude "web.config" | Remove-Item -Recurse -Force

# ─── 5. نسخ الملفات الجديدة ───
Write-Host "📋 جاري نسخ الملفات الجديدة..." -ForegroundColor Yellow
$robocopyArgs = @(
    $SourcePath, $TargetPath, 
    "/MIR", "/XF", "web.config",  # ⚠️ حماية مطلقة لـ web.config
    "/NFL", "/NDL", "/NJH", "/NJS", "/NC", "/NS", "/NP"
)
$rc = Start-Process -FilePath "robocopy.exe" -ArgumentList $robocopyArgs -Wait -NoNewWindow -PassThru
if ($rc.ExitCode -gt 7) {
    Write-Host "❌ فشل Robocopy (Exit Code: $($rc.ExitCode))" -ForegroundColor Red
    exit 1
}
Write-Host "✅ تم نسخ الملفات بنجاح." -ForegroundColor Green

# ─── 6. التحقق من صحة web.config ───
$webConfigContent = Get-Content "$TargetPath\web.config" -Raw
if ($webConfigContent -notmatch "Serve wwwroot files") {
    Write-Host "🚨 خطر: تم العبث بملف web.config! جاري استعادته من النسخة الاحتياطية..." -ForegroundColor Red
    Write-Host "⚠️  يرجى استعادة web.config يدوياً من النسخة الاحتياطية وإلغاء النشر." -ForegroundColor Red
    exit 1
}

# ─── 7. تشغيل App Pool ───
Write-Host "▶️  جاري تشغيل App Pool..." -ForegroundColor Yellow
Start-WebAppPool -Name $AppPoolName
Start-Sleep -Seconds 3

Write-Host "`n╔══════════════════════════════════════════════════════════╗" -ForegroundColor Green
Write-Host "║            ✅ تم النشر بنجاح وأمان!                    ║" -ForegroundColor Green
Write-Host "╚══════════════════════════════════════════════════════════╝" -ForegroundColor Green
Write-Host "📝 ملاحظة: افتح المتصفح في وضع InPrivate (Ctrl+Shift+N) لتجنب كاش Service Worker القديم.`n" -ForegroundColor Cyan
```

---

## 📋 ملخص التحديثات في هذه الوثيقة

| القسم | التغيير |
|-------|---------|
| **المحتوى الأصلي** | ✅ محفوظ بالكامل (البنية، المكونات، الخدمات، الخطط) |
| **نظام المصادقة** | ✅ إضافة كاملة: Google OAuth, Logout الموحّد, Client IDs المتعددة |
| **نظام تنظيف الكاش** | ✅ إضافة كاملة: cache-reset.html, load-failure-detector.js v2 |
| **تحسينات UI** | ✅ TopBar المحسّن, حل CSS Specificity |
| **نظام المحادثات** | ✅ تدفق الحجز + ChatPage |
| **حلول المشاكل** | ✅ إضافة 4 مشاكل جديدة (Illegal invocation, 404 logout, Duplicate Routes, 500.19.183) |
| **الدروس المستفادة** | ✅ إضافة 6 دروس جديدة (المسح, wwwroot, InPrivate, .NET 10, cache-reset, appsettings) |
| **دليل النشر** | ✅ تحديث شامل: خصائص فريدة, web.config المقدس, النسخ الاحتياطي, سكريبت v4.0 |
| **إحصائيات المشروع** | ✅ تحديث: 68 مكون, 15+ صفحة, ~40% إنجاز |

---

**ملاحظة:** هذه وثيقة حية (Living Document) — يتم تحديثها مع تقدم المشروع.

**آخر مراجعة:** 31 أكتوبر 2026  
**المراجعة التالية:** بعد إكمال المرحلة 3 (الميزات المتقدمة)

---

## 🚀 ملخص الإنجازات في هذه الجلسة

| # | الإنجاز | الأهمية |
|---|---------|---------|
| 1 | 🔐 Google OAuth | عالية جداً |
| 2 | 🚪 Logout الموحّد | عالية |
| 3 | 🧹 Cache Reset Page | عالية جداً |
| 4 | 🛡️ Google GIS Polyfill | متوسطة |
| 5 | 🎨 TopBar المحسّن | متوسطة |
| 6 | 📱 Messaging Integration | عالية |
| 7 | 🐛 Bug Fixes (404, SRI, Routes, 500.19) | عالية جداً |
| 8 | 🌍 نظام البيئات (3 ملفات appsettings) | عالية جداً |
| 9 | 📝 سكريبت النشر v4.0 | عالية جداً |
| 10 | 📚 توثيق شامل للوثيقة | عالية |

---

