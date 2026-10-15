# 📘 09 — API GUIDE v2.0 (النسخة النهائية المُحدَّثة)

**آخر تحديث:** 16 أكتوبر 2026
**الإصدار:** 2.0 (Full Controllers Edition — Verified)
**الحالة:** ✅ مرجع شامل — موثّق من الكود الفعلي
**الجمهور:** فريق التطوير، تكامل العملاء (Web, PWA, Mobile)، AI Assistants

---

## 📌 مقدمة

هذا المرجع هو **الدليل الكامل** لجميع APIs في منصة RubikCare.

**الأرقام الفعلية (16 أكتوبر 2026) — مُتحقَّق منها من الكود:**

| المقياس | القيمة |
|:---|:---:|
| **عدد Controllers** | 35+ |
| **عدد Endpoints** | ~290 |
| **عدد المجموعات** | 18 |
| **المشاريع المعتمدة** | 9 |

---

## 📊 الجزء الأول: النظرة العامة

### 1.1 الهيكل الكامل

```
Api.Web/Controllers/
├── AuthController.cs                      (13 endpoints)
├── ClinicController.cs                    (15 endpoints)
├── DashboardController.cs                 (1 endpoint)
├── DoctorInvitationsController.cs         (1 endpoint)
├── DrugFormsController.cs                 (1 endpoint)
├── HealthController.cs                    (1 endpoint)
├── LocalizationController.cs              (2 endpoints)
├── MedicationCategoriesController.cs      (1 endpoint)
├── NotificationsController.cs             (9 endpoints)
├── PatientMedicationsController.cs        (10 endpoints)
├── ProfessionalController.cs              (3 endpoints)
├── UserController.cs                      (7 endpoints)
│
├── Base/
│   └── BaseApiController.cs               (8 endpoints — Helper)
│
├── Clinical/
│   ├── OrganizationTypesController.cs     (1 endpoint)
│   └── SpecialitiesController.cs          (4 endpoints)
│
├── GeoLocations/
│   ├── AreasController.cs                 (2 endpoints)
│   ├── CitiesController.cs                (3 endpoints)
│   └── CountriesController.cs             (1 endpoint)
│
├── Messaging/
│   ├── ConversationsController.cs         (7 endpoints)
│   └── MessagesController.cs              (9 endpoints)
│
├── Organizations/
│   ├── OrganizationJobTitlesController.cs      (2 endpoints)
│   ├── OrganizationMembersController.cs        (3 endpoints)
│   ├── OrganizationMemberTitlesController.cs   (5 endpoints)
│   └── OrganizationsController.cs              (3 endpoints)
│
├── Patient/
│   └── PatientOrdersController.cs         (3 endpoints)
│
├── Pharmacy/
│   ├── DispenseController.cs              (14 endpoints) ⭐ موثّق
│   ├── MedicationRequestController.cs     (6 endpoints) ⭐ موثّق
│   └── PharmacyController.cs              (7 endpoints) ⭐ موثّق
│
├── PSP/
│   ├── DispensationPlansController.cs     (5 endpoints)
│   ├── PharmaCompanyMedicationsController.cs  (18 endpoints)
│   ├── PspController.cs                   (0 — Base)
│   ├── PspController.Programs.cs          (4 endpoints) ⭐ موثّق
│   ├── PspController.Invitations.cs       (2 endpoints) ⭐ موثّق
│   ├── PspController.RepInvitations.cs    (2 endpoints) ⭐ موثّق
│   ├── PspController.Enrollment.cs        (2 endpoints) ⭐ موثّق
│   ├── PspController.Unsubscribe.cs       (1 endpoint) ⭐ موثّق
│   ├── PspController.Entry.cs             (1 endpoint) ⭐ موثّق
│   ├── PspController.DispensationPlans.cs (6 endpoints) ⭐ موثّق
│   ├── PspController.Links.cs             (3 endpoints) ⭐ موثّق
│   ├── PspController.Tests.cs             (38 endpoints) ⭐ موثّق
│   ├── Pspcontroller.patientdetails.cs    (1 endpoint) ⭐ موثّق
│   ├── PspController.Helpers.cs           (0 — Helper) ⭐ موثّق
│   ├── RemindersController.cs             (8 endpoints) ⭐ موثّق
│   │
│   └── Points/
│       ├── PointBalancesController.cs     (7 endpoints) ⭐ موثّق
│       ├── PointEarningController.cs      (1 endpoint) ⭐ موثّق
│       ├── PointLookupsController.cs      (4 endpoints) ⭐ موثّق
│       ├── PointRedeemController.cs       (8 endpoints) ⭐ موثّق
│       ├── PointRulesController.cs        (5 endpoints) ⭐ موثّق
│       └── PointSchemesController.cs      (13 endpoints) ⭐ موثّق
│
├── Rep/
│   └── RepController.cs                   (12 endpoints)
│
└── Reports/
    └── ReportsController.cs               (8 endpoints)
```

### 1.2 جدول شامل للمجموعات

| # | المجموعة | Controllers | Endpoints | الأولوية |
|:-:|:---|:---:|:---:|:---:|
| 1 | **Auth & Identity** | 2 | 20 | 🔴 |
| 2 | **User & Profile** | 2 | 10 | 🔴 |
| 3 | **Organizations** | 4 | 13 | 🔴 |
| 4 | **Clinical** | 2 | 5 | 🟠 |
| 5 | **GeoLocations** | 3 | 6 | 🟢 |
| 6 | **Messaging** | 2 | 16 | 🟠 |
| 7 | **Notifications** | 1 | 9 | 🔴 |
| 8 | **Clinic** | 1 | 15 | 🔴 |
| 9 | **Professional** | 1 | 3 | 🟠 |
| 10 | **Patient** | 2 | 13 | 🔴 |
| 11 | **Pharmacy** | 3 | 27 | 🔴 |
| 12 | **PSP** | 15 | 68 | 🔴 |
| 13 | **Points** | 6 | 38 | 🔴 |
| 14 | **Rep** | 1 | 12 | 🔴 |
| 15 | **Reports** | 1 | 8 | 🔴 |
| 16 | **Dashboard** | 1 | 1 | 🟠 |
| 17 | **Misc** | 3 | 3 | 🟢 |
| 18 | **Base** | 1 | 8 | — |
| | **المجموع** | **~50 ملف** | **~290** | |

---

## 🔐 الجزء الثاني: المصادقة والهوية

### 2.1 `AuthController` (13 endpoints)

**المسار:** `Api.Web/Controllers/AuthController.cs`

| # | Method | Endpoint | الوظيفة |
|:-:|:---:|:---|:---|
| 1 | `POST` | `/api/auth/login` | تسجيل دخول تقليدي |
| 2 | `POST` | `/api/auth/register` | تسجيل مستخدم جديد |
| 3 | `POST` | `/api/auth/google-login` | تسجيل عبر Google (ID Token) |
| 4 | `POST` | `/api/auth/logout` | تسجيل الخروج + مسح الكاش |
| 5 | `POST` | `/api/auth/change-password` | تغيير كلمة المرور |
| 6 | `POST` | `/api/auth/forgot-password` | نسيت كلمة المرور |
| 7 | `POST` | `/api/auth/reset-password` | إعادة تعيين كلمة المرور |
| 8 | `POST` | `/api/auth/confirm-email` | تأكيد البريد |
| 9 | `POST` | `/api/auth/resend-confirmation` | إعادة إرسال التأكيد |
| 10 | `POST` | `/api/auth/refresh-token` | تجديد JWT |
| 11 | `GET` | `/api/auth/me` | المستخدم الحالي |
| 12 | `POST` | `/api/auth/complete-profile` | إكمال الملف الشخصي |
| 13 | `POST` | `/api/auth/verify-code` | التحقق من الرمز |

### 2.2 `UserController` (7 endpoints)

| # | Method | Endpoint | الوظيفة |
|:-:|:---:|:---|:---|
| 1 | `GET` | `/api/user/profile` | جلب الملف الشخصي |
| 2 | `PUT` | `/api/user/profile` | تحديث الملف الشخصي |
| 3 | `POST` | `/api/user/upload-picture` | رفع صورة البروفايل |
| 4 | `GET` | `/api/user/session-bootstrap` | تهيئة الجلسة (Mobile) |
| 5 | `GET` | `/api/user/memberships` | عضويات المستخدم |
| 6 | `GET` | `/api/user/preferences` | تفضيلات المستخدم |
| 7 | `PUT` | `/api/user/preferences` | تحديث التفضيلات |

#### 📌 مثال: `POST /api/auth/login`

**Request:**
```json
{
  "email": "user@example.com",
  "password": "Password123!"
}
```

**Response (نجاح):**
```json
{
  "success": true,
  "token": "eyJhbGciOiJIUzI1NiIs...",
  "user": {
    "id": "abc123",
    "email": "user@example.com",
    "displayName": "أحمد محمد",
    "userProfileId": 12,
    "roles": ["Doctor"]
  }
}
```

---

## 👥 الجزء الثالث: المنظمات

### 3.1 `OrganizationsController` (3 endpoints)

| # | Method | Endpoint | الوظيفة |
|:-:|:---:|:---|:---|
| 1 | `GET` | `/api/organizations/{id}` | جلب منظمة |
| 2 | `POST` | `/api/organizations` | إنشاء منظمة |
| 3 | `PUT` | `/api/organizations/{id}` | تحديث منظمة |

### 3.2 `OrganizationMembersController` (3 endpoints)

| # | Method | Endpoint | الوظيفة |
|:-:|:---:|:---|:---|
| 1 | `GET` | `/api/organizations/{orgId}/members` | أعضاء المنظمة |
| 2 | `POST` | `/api/organizations/{orgId}/members` | إضافة عضو |
| 3 | `DELETE` | `/api/organizations/{orgId}/members/{memberId}` | حذف عضو |

### 3.3 `OrganizationJobTitlesController` (2 endpoints)

| # | Method | Endpoint | الوظيفة |
|:-:|:---:|:---|:---|
| 1 | `GET` | `/api/organizations/{orgId}/job-titles` | المسميات الوظيفية |
| 2 | `POST` | `/api/organizations/{orgId}/job-titles` | إضافة مسمى |

### 3.4 `OrganizationMemberTitlesController` (5 endpoints)

| # | Method | Endpoint | الوظيفة |
|:-:|:---:|:---|:---|
| 1 | `GET` | `/api/organizations/{orgId}/member-titles` | ألقاب الأعضاء |
| 2 | `POST` | `/api/organizations/{orgId}/member-titles` | إضافة لقب |
| 3 | `PUT` | `/api/organizations/{orgId}/member-titles/{id}` | تحديث |
| 4 | `DELETE` | `/api/organizations/{orgId}/member-titles/{id}` | حذف |
| 5 | `POST` | `/api/organizations/{orgId}/member-titles/assign` | تعيين لقب لعضو |

---

## 🏥 الجزء الرابع: Clinical & Geo

### 4.1 `OrganizationTypesController` (1 endpoint)

| # | Method | Endpoint | الوظيفة |
|:-:|:---:|:---|:---|
| 1 | `GET` | `/api/clinical/organization-types` | أنواع المنظمات |

### 4.2 `SpecialitiesController` (4 endpoints)

| # | Method | Endpoint | الوظيفة |
|:-:|:---:|:---|:---|
| 1 | `GET` | `/api/clinical/specialities` | التخصصات |
| 2 | `GET` | `/api/clinical/specialities/{id}` | تخصص واحد |
| 3 | `POST` | `/api/clinical/specialities` | إضافة تخصص |
| 4 | `DELETE` | `/api/clinical/specialities/{id}` | حذف تخصص |

### 4.3 `CountriesController` (1 endpoint)

| # | Method | Endpoint | الوظيفة |
|:-:|:---:|:---|:---|
| 1 | `GET` | `/api/geo/countries` | قائمة الدول |

### 4.4 `CitiesController` (3 endpoints)

| # | Method | Endpoint | الوظيفة |
|:-:|:---:|:---|:---|
| 1 | `GET` | `/api/geo/cities` | كل المدن |
| 2 | `GET` | `/api/geo/countries/{countryId}/cities` | مدن دولة |
| 3 | `GET` | `/api/geo/cities/{id}` | مدينة واحدة |

### 4.5 `AreasController` (2 endpoints)

| # | Method | Endpoint | الوظيفة |
|:-:|:---:|:---|:---|
| 1 | `GET` | `/api/geo/cities/{cityId}/areas` | مناطق مدينة |
| 2 | `GET` | `/api/geo/areas/{id}` | منطقة واحدة |

---

## 💬 الجزء الخامس: Messaging

### 5.1 `ConversationsController` (7 endpoints)

| # | Method | Endpoint | الوظيفة |
|:-:|:---:|:---|:---|
| 1 | `GET` | `/api/messaging/conversations` | قائمة المحادثات |
| 2 | `POST` | `/api/messaging/conversations` | إنشاء محادثة |
| 3 | `GET` | `/api/messaging/conversations/{id}` | تفاصيل محادثة |
| 4 | `PUT` | `/api/messaging/conversations/{id}` | تحديث محادثة |
| 5 | `DELETE` | `/api/messaging/conversations/{id}` | حذف محادثة |
| 6 | `POST` | `/api/messaging/conversations/{id}/read` | تعليم كمقروءة |
| 7 | `POST` | `/api/messaging/conversations/{id}/archive` | أرشفة |

### 5.2 `MessagesController` (9 endpoints)

| # | Method | Endpoint | الوظيفة |
|:-:|:---:|:---|:---|
| 1 | `GET` | `/api/messaging/conversations/{convId}/messages` | رسائل محادثة |
| 2 | `POST` | `/api/messaging/conversations/{convId}/messages` | إرسال رسالة |
| 3 | `GET` | `/api/messaging/messages/{id}` | رسالة واحدة |
| 4 | `PUT` | `/api/messaging/messages/{id}` | تعديل رسالة |
| 5 | `DELETE` | `/api/messaging/messages/{id}` | حذف رسالة |
| 6 | `POST` | `/api/messaging/messages/{id}/read` | تعليم كمقروءة |
| 7 | `POST` | `/api/messaging/messages/{id}/attachment` | رفع مرفق |
| 8 | `GET` | `/api/messaging/messages/unread-count` | عدد غير المقروءة |
| 9 | `POST` | `/api/messaging/messages/mark-all-read` | تعليم الكل كمقروء |

---

## 🔔 الجزء السادس: Notifications

### 6.1 `NotificationsController` (9 endpoints)

| # | Method | Endpoint | الوظيفة |
|:-:|:---:|:---|:---|
| 1 | `GET` | `/api/notifications` | قائمة الإشعارات |
| 2 | `GET` | `/api/notifications/unread-count` | عدد غير المقروءة |
| 3 | `POST` | `/api/notifications/{id}/read` | تعليم كمقروء |
| 4 | `POST` | `/api/notifications/mark-all-read` | تعليم الكل كمقروء |
| 5 | `DELETE` | `/api/notifications/{id}` | حذف إشعار |
| 6 | `POST` | `/api/notifications/register-device` | تسجيل FCM Token |
| 7 | `POST` | `/api/notifications/register-web-push` | تسجيل Web Push Subscription |
| 8 | `POST` | `/api/notifications/unregister-device` | إلغاء تسجيل FCM |
| 9 | `POST` | `/api/notifications/send-test` | إرسال إشعار تجريبي (Dev) |

**⚠️ نظامان منفصلان:**
- `DeviceTokens` — FCM (MAUI)
- `UserWebPushSubscriptions` — Web Push (PWA)

---

## 🏥 الجزء السابع: Clinic

### 7.1 `ClinicController` (15 endpoints)

| # | Method | Endpoint | الوظيفة |
|:-:|:---:|:---|:---|
| 1 | `GET` | `/api/clinic/dashboard` | لوحة تحكم العيادة |
| 2 | `GET` | `/api/clinic/patients` | مرضى العيادة |
| 3 | `GET` | `/api/clinic/patients/{id}` | تفاصيل مريض |
| 4 | `POST` | `/api/clinic/patients` | إضافة مريض |
| 5 | `PUT` | `/api/clinic/patients/{id}` | تحديث مريض |
| 6 | `DELETE` | `/api/clinic/patients/{id}` | حذف مريض |
| 7 | `GET` | `/api/clinic/invitations` | دعوات العيادة |
| 8 | `POST` | `/api/clinic/invitations` | إنشاء دعوة |
| 9 | `POST` | `/api/clinic/invitations/{id}/resend` | إعادة إرسال |
| 10 | `GET` | `/api/clinic/psp-programs` | برامج PSP الخاصة بالعيادة |
| 11 | `GET` | `/api/clinic/psp-participations` | مشاركات العيادة |
| 12 | `GET` | `/api/clinic/schedule` | جدول العيادة |
| 13 | `POST` | `/api/clinic/schedule` | إضافة موعد |
| 14 | `PUT` | `/api/clinic/schedule/{id}` | تحديث موعد |
| 15 | `DELETE` | `/api/clinic/schedule/{id}` | حذف موعد |

---

## 👨‍⚕️ الجزء الثامن: Professional

### 8.1 `ProfessionalController` (3 endpoints)

| # | Method | Endpoint | الوظيفة |
|:-:|:---:|:---|:---|
| 1 | `GET` | `/api/professional/status` | حالة الترخيص |
| 2 | `POST` | `/api/professional/submit-license` | تقديم طلب ترخيص |
| 3 | `POST` | `/api/professional/upload-document` | رفع مستند |

### 8.2 `DoctorInvitationsController` (1 endpoint)

| # | Method | Endpoint | الوظيفة |
|:-:|:---:|:---|:---|
| 1 | `GET` | `/api/doctor-invitations/pending` | دعوات معلقة |

---

## 👤 الجزء التاسع: Patient

### 9.1 `PatientMedicationsController` (10 endpoints)

| # | Method | Endpoint | الوظيفة |
|:-:|:---:|:---|:---|
| 1 | `GET` | `/api/patient/medications` | أدوية المريض |
| 2 | `GET` | `/api/patient/medications/{id}` | دواء واحد |
| 3 | `POST` | `/api/patient/medications` | إضافة دواء |
| 4 | `PUT` | `/api/patient/medications/{id}` | تحديث دواء |
| 5 | `DELETE` | `/api/patient/medications/{id}` | حذف دواء |
| 6 | `GET` | `/api/patient/schedules` | جدول الجرعات |
| 7 | `POST` | `/api/patient/schedules` | إضافة موعد جرعة |
| 8 | `PUT` | `/api/patient/schedules/{id}/status` | تحديث حالة الجرعة |
| 9 | `GET` | `/api/patient/reminders` | التذكيرات |
| 10 | `POST` | `/api/patient/reminders/sync` | مزامنة التذكيرات |

### 9.2 `PatientOrdersController` (3 endpoints)

| # | Method | Endpoint | الوظيفة |
|:-:|:---:|:---|:---|
| 1 | `GET` | `/api/patient/orders` | طلبات المريض |
| 2 | `GET` | `/api/patient/orders/{id}` | تفاصيل طلب |
| 3 | `POST` | `/api/patient/orders/{id}/confirm-receipt` | تأكيد الاستلام |

---

## 💊 الجزء العاشر: Pharmacy (مُوثَّق بدقة)

### 10.1 `PharmacyController` (7 endpoints)

**المسار:** `Api.Web/Controllers/Pharmacy/PharmacyController.cs`
**Route:** `[Route("api/pharmacy")]`

| # | Method | Endpoint | الوظيفة | Auth |
|:-:|:---:|:---|:---|:---:|
| 1 | `GET` | `/api/pharmacy/dashboard/stats` | إحصائيات اللوحة | 🔒 |
| 2 | `GET` | `/api/pharmacy/reports` | تقارير الصيدلية | 🔒 |
| 3 | `GET` | `/api/pharmacy/nearby` | الصيدليات القريبة (GPS) | ⚠️ Anonymous |
| 4 | `PUT` | `/api/pharmacy/update-location` | تحديث موقع الصيدلية | 🔒 |
| 5 | `GET` | `/api/pharmacy/details/{id}` | تفاصيل صيدلية | 🔒 |
| 6 | `GET` | `/api/pharmacy/{pharmacyId}/programs` | برامج الصيدلية | 🔒 |
| 7 | `GET` | `/api/pharmacy/medication-requests` | طلبات الدواء | 🔒 |

#### 📌 مثال: `GET /api/pharmacy/dashboard/stats`

**Query Parameters:**
- `pharmacyId` (int, optional)

**Response:**
```json
{
  "pharmacyId": 1245,
  "pharmacyName": "صيدلية الشفاء",
  "pharmacistName": "د. أحمد",
  "todayDispenses": 12,
  "pendingDispenses": 3,
  "totalPatients": 45,
  "activePrograms": 0,
  "pendingRefills": 0,
  "latestDispenseDate": "2026-10-16T14:30:00",
  "latestDispensePatient": "",
  "latestDispenseMedication": ""
}
```

#### 📌 مثال: `GET /api/pharmacy/nearby`

**Query Parameters:**
- `lat` (double, required) — خط العرض
- `lng` (double, required) — خط الطول
- `radiusKm` (double, default: 10)

**Response:**
```json
[
  {
    "organizationID": 1245,
    "organizationNameAr": "صيدلية الشفاء",
    "fullAddressAr": "شارع الرئيسي، القاهرة",
    "phone": "01234567890",
    "latitude": "30.0444",
    "longitude": "31.2357",
    "distanceKm": 1.2,
    "areaNameAr": "وسط البلد",
    "cityNameAr": "القاهرة"
  }
]
```

#### 📌 مثال: `GET /api/pharmacy/{pharmacyId}/programs`

**Query Parameters:**
- `status` (string, default: `"ALL"`) — `"ACCEPTED"` / `"USED"` / `"ALL"`

**Response:**
```json
[
  {
    "programId": 2,
    "programName": "برنامج دعم السكري",
    "companyName": "شركة ديفارت لاب",
    "status": "USED",
    "joinDate": "2026-09-15T10:00:00",
    "invitationId": 42,
    "canInvitePatients": true
  }
]
```

### 10.2 `DispenseController` (14 endpoints)

**المسار:** `Api.Web/Controllers/Pharmacy/DispenseController.cs`
**Route:** `[Route("api/dispense")]`

**⚠️ بعض الـ endpoints `[AllowAnonymous]` — تحتاج مراجعة أمنية.**

| # | Method | Endpoint | الوظيفة | Auth |
|:-:|:---:|:---|:---|:---:|
| 1 | `POST` | `/api/dispense/validate-token` | التحقق من رمز الصرف | ⚠️ Anonymous |
| 2 | `POST` | `/api/dispense/confirm` | تأكيد الصرف (+ منح النقاط) | ⚠️ Anonymous |
| 3 | `GET` | `/api/dispense/history` | سجل الصرف | 🔒 |
| 4 | `POST` | `/api/dispense/send-to-pharmacy` | إرسال توكن لصيدلية | 🔒 |
| 5 | `POST` | `/api/dispense/confirm-receipt` | تأكيد الاستلام | 🔒 |
| 6 | `POST` | `/api/dispense/generate-next-token` | توليد توكن جديد | 🔒 |
| 7 | `POST` | `/api/dispense/submit-for-approval` | المريض يرسل للصيدلي | 🔒 |
| 8 | `POST` | `/api/dispense/approve` | الصيدلي يوافق | 🔒 |
| 9 | `POST` | `/api/dispense/scan-and-dispense` | مسح QR → صرف (+ نقاط) | 🔒 |
| 10 | `GET` | `/api/dispense/pending-approvals` | طلبات معلقة | 🔒 |
| 11 | `GET` | `/api/dispense/status/{id}` | حالة الصرف | 🔒 |
| 12 | `GET` | `/api/dispense/status-by-token/{tokenCode}` | حالة بالتوكن | ⚠️ Anonymous |
| 13 | `GET` | `/api/dispense/my-psp-orders` | طلبات PSP للمريض | 🔒 |
| 14 | `POST` | `/api/dispense/refill-completed` | إتمام Refill | 🔒 |

#### 📌 مثال: `POST /api/dispense/validate-token`

**Request:**
```json
{ "tokenCode": "RC-20260323-1234", "pharmacyId": 1260 }
```

**Response (صالحة):**
```json
{
  "isValid": true,
  "message": "الرمز صالح",
  "dispensationId": 42,
  "tokenCode": "RC-20260323-1234",
  "patientId": 100,
  "patientName": "أحمد محمد",
  "patientPhone": "01234567890",
  "medicationId": 5,
  "medicationName": "Oxy Free",
  "dosage": "500mg",
  "programId": 2,
  "programName": "Infertility Support Program",
  "originalPrice": 200,
  "discountedPrice": 150,
  "patientPayment": 150,
  "expiryDate": "2026-11-15T00:00:00"
}
```

#### 📌 مثال: `POST /api/dispense/confirm` ⭐ (مع منح النقاط)

**Request:**
```json
{
  "tokenCode": "RC-20260323-1234",
  "dispensationId": 42,
  "pharmacyId": 1260,
  "pharmacistId": 55,
  "quantityDispensed": 20
}
```

**Response:**
```json
{
  "success": true,
  "message": "تم صرف الدواء بنجاح",
  "transactionId": 42,
  "dispenseDate": "2026-10-16T14:30:00"
}
```

**⚠️ ما يحدث داخلياً:**
1. **حساب المبالغ المالية** (`UnitPrice × Quantity`).
2. **تحديث `PSPDispensation`** (`TokenUsed = true`).
3. **⭐ منح النقاط** عبر `IEarnPointsOrchestrator.AwardByCampaignAsync`:
   - `PointActivityTypeIds.DispensationCompleted` (أول صرف)
   - `PointActivityTypeIds.RefillCompleted` (إعادة صرف)
4. **إشعار المريض** (DB + FCM + Web Push).

### 10.3 `MedicationRequestController` (6 endpoints)

**المسار:** `Api.Web/Controllers/Pharmacy/MedicationRequestController.cs`
**Route:** `[Route("api/pharmacy")]`

| # | Method | Endpoint | الوظيفة | Auth |
|:-:|:---:|:---|:---|:---:|
| 1 | `POST` | `/api/pharmacy/medication-request` | إنشاء طلب دواء | 🔒 |
| 2 | `GET` | `/api/pharmacy/medication-requests` | طلبات الصيدلية | 🔒 |
| 3 | `PUT` | `/api/pharmacy/medication-request/{id}/approve` | موافقة | 🔒 |
| 4 | `PUT` | `/api/pharmacy/medication-request/{id}/reject` | رفض | 🔒 |
| 5 | `GET` | `/api/pharmacy/my-requests` | طلبات المريض | 🔒 |
| 6 | `PUT` | `/api/pharmacy/medication-request/{id}/cancel` | إلغاء | 🔒 |

---

## 💚 الجزء الحادي عشر: PSP (مُوثَّق بدقة من الكود)

### 11.1 `PspController` (12 ملف — 62 endpoints)

**المسار:** `Api.Web/Controllers/PSP/`

#### 📌 `PspController.Programs.cs` (4 endpoints)

| # | Method | Endpoint | الوظيفة | Auth |
|:-:|:---:|:---|:---|:---:|
| 1 | `GET` | `/api/psp/programs` | قائمة البرامج (Pagination) | 🔒 |
| 2 | `GET` | `/api/psp/programs-temp` | ⚠️ **Temp** — قائمة البرامج | ⚠️ Anonymous |
| 3 | `GET` | `/api/psp/programs/{id}` | تفاصيل برنامج | 🔒 |
| 4 | `GET` | `/api/psp/program-details-temp/{id}` | ⚠️ **Temp** — تفاصيل | ⚠️ Anonymous |

**⚠️ ملاحظات:**
- `programs-temp` و `program-details-temp` — **endpoints مؤقتة للاختبار**.
- `GetProgramById` يستقبل `organizationId` كـ `[FromQuery]` لحساب `IsSubscribed` + `CanResubscribe`.

#### 📌 `PspController.Invitations.cs` (2 endpoints)

| # | Method | Endpoint | الوظيفة |
|:-:|:---:|:---|:---|
| 1 | `POST` | `/api/psp/create-invitation` | إنشاء دعوة مريض (Doctor → Patient) |
| 2 | `POST` | `/api/psp/accept-invitation` | قبول دعوة المريض |

**⚠️ كلاهما يستدعي `IPSPEnrollmentService` (مع منح النقاط).**

#### 📌 `PspController.RepInvitations.cs` (2 endpoints)

| # | Method | Endpoint | الوظيفة |
|:-:|:---:|:---|:---|
| 1 | `GET` | `/api/psp/invitation/by-token?token=X` | البحث عن دعوة بالكود |
| 2 | `POST` | `/api/psp/accept-rep-invitation` | قبول دعوة المندوب (Rep → Org) |

#### 📌 `PspController.Enrollment.cs` (2 endpoints)

| # | Method | Endpoint | الوظيفة |
|:-:|:---:|:---|:---|
| 1 | `POST` | `/api/psp/participate` | اشتراك/إعادة اشتراك (UseCase) |
| 2 | `POST` | `/api/psp/unsubscribe` | ⚠️ **قديم** — إلغاء اشتراك مباشر |

**⚠️ `Participate` مربوط بـ `ResubscribeToProgramUseCase` — يدعم 3 سيناريوهات:**
1. اشتراك جديد.
2. إعادة اشتراك (`WITHDRAWN`).
3. رفض المشترك بالفعل.

**⚠️ تضارب:** `unsubscribe` موجود في `Enrollment.cs` و `Unsubscribe.cs`.

#### 📌 `PspController.Unsubscribe.cs` (1 endpoint) ⭐ جديد

| # | Method | Endpoint | الوظيفة |
|:-:|:---:|:---|:---|
| 1 | `POST` | `/api/psp/unsubscribe` | إلغاء اشتراك (UseCase) |

**⚠️ يستدعي `UnsubscribeUseCase`.**

#### 📌 `PspController.Entry.cs` (1 endpoint)

| # | Method | Endpoint | الوظيفة |
|:-:|:---:|:---|:---|
| 1 | `POST` | `/api/psp/entry` | نقطة دخول المريض |

**Request:**
```json
{ "invitationCode": "INV-5CWEFHEJ" }
```

**حالات الاستجابة:**

| الحالة | المعنى |
|:---|:---|
| `NEW_ENROLLMENT` | تم الاشتراك بنجاح |
| `ACTIVE_PROGRAMS` | برامج نشطة بالفعل |
| `ALREADY_ENROLLED` | مسجل مسبقاً |
| `CODE_INVALID` | كود غير صالح |
| `CODE_EXPIRED` | كود منتهي |
| `CODE_USED` | مستخدم من آخر |
| `NO_CODE` | لا يوجد كود |
| `UNAUTHORIZED` | غير مصرح |

#### 📌 `PspController.DispensationPlans.cs` (6 endpoints)

| # | Method | Endpoint | الوظيفة |
|:-:|:---:|:---|:---|
| 1 | `GET` | `/api/psp/plans/{planId}/phases` | مراحل خطة |
| 2 | `GET` | `/api/psp/phases/{phaseId}` | مرحلة واحدة |
| 3 | `POST` | `/api/psp/plans/{planId}/phases` | إضافة مرحلة |
| 4 | `PUT` | `/api/psp/phases/{phaseId}` | تعديل مرحلة |
| 5 | `DELETE` | `/api/psp/phases/{phaseId}` | حذف مرحلة |
| 6 | `GET` | `/api/psp/plans/{planId}/available-medications` | أدوية متاحة |

#### 📌 `PspController.Links.cs` (3 endpoints)

| # | Method | Endpoint | الوظيفة |
|:-:|:---:|:---|:---|
| 1 | `POST` | `/api/psp/links` | إضافة رابط |
| 2 | `PUT` | `/api/psp/links/{id}` | تحديث رابط |
| 3 | `DELETE` | `/api/psp/links/{id}` | حذف رابط |

**⚠️ قاعدة:** `MaxLinksPerProgram = 6` روابط لكل برنامج.

**⚠️ `LinkTargetAudience` Enum** — لتحديد الجمهور.

#### 📌 `Pspcontroller.patientdetails.cs` (1 endpoint)

| # | Method | Endpoint | الوظيفة |
|:-:|:---:|:---|:---|
| 1 | `GET` | `/api/psp/patient-details?patientId=X&participationId=Y` | تفاصيل برنامج المريض |

**⚠️ ملاحظة:** `PharmaciesCount` + `NearestPharmacy` + `PharmacyDistance` — **Hardcoded حالياً** (تحتاج ربط بـ `NearbyPharmacyService`).

#### 📌 `PspController.Tests.cs` (38 endpoints) ⭐

**8 مجموعات فرعية:**

| المجموعة | Endpoints |
|:---|:---:|
| LabTest (مرجعية) | 1 |
| RequiredTest | 5 |
| TestResult | 8 |
| Attachment | 5 |
| PharmaView (Privacy) | 1 |
| Reminder | 3 |
| PhaseRequiredTest | 3 |
| Patient Mobile | 5 |

**⚠️ منطق `RequiredTiming` (4 قيم):**
- `1` = Before (قبل البدء)
- `2` = During (أثناء)
- `3` = After N days
- `4` = Periodic (كل N يوم)

**⚠️ منطق `EditWindow`:**
- Mandatory → 24 ساعة
- غير Mandatory → 168 ساعة (7 أيام)

**⚠️ منطق `Consent`:** موافقة المريض مطلوبة لكل رفع.

#### 📌 `PspController.Helpers.cs` (0 endpoints — NonAction)

**Helpers:**
- `GetCurrentUserId()` — من claim `userProfileId`
- `GenerateTokenCode()` — `RC-yyyyMMdd-NNNN`
- `GenerateERxCode()` — `PSP-ERX-yyyyMMdd-XXXXXX`
- `GenerateDispensationCode()` — `DISP-yyyyMMdd-XXXXXX`
- `GenerateInvitationToken()` — `INV-XXXXXXXX`
- `GetUserActivePrograms()` — `[NonAction]`
- `EnrollUserInProgram()` — `[NonAction]`

### 11.2 `RemindersController` (8 endpoints)

**المسار:** `Api.Web/Controllers/PSP/RemindersController.cs`
**Route:** `[Route("api/psp/reminders")]`

**✅ نظيف — UseCase Pattern.**

| # | Method | Endpoint | الوظيفة | Handler |
|:-:|:---:|:---|:---|:---|
| 1 | `POST` | `/api/psp/reminders` | إنشاء تذكير | `CreateReminderHandler` |
| 2 | `GET` | `/api/psp/reminders` | التذكيرات | `GetRemindersHandler` |
| 3 | `GET` | `/api/psp/reminders/{id}` | تذكير واحد | `GetReminderByIdHandler` |
| 4 | `PATCH` | `/api/psp/reminders/{id}/sent` | تعليم كـ Sent | `MarkReminderAsSentHandler` |
| 5 | `POST` | `/api/psp/reminders/{id}/take` | المريض ضغط "تناول" | `AcknowledgeReminderHandler` |
| 6 | `PATCH` | `/api/psp/reminders/{id}/complete` | إكمال | `CompleteReminderHandler` |
| 7 | `PATCH` | `/api/psp/reminders/{id}/cancel` | إلغاء | `CancelReminderHandler` |
| 8 | `POST` | `/api/psp/reminders/{id}/repeat` | تكرار | `RepeatReminderHandler` |

### 11.3 `DispensationPlansController` (5 endpoints)

**Route:** `[Route("api/[controller]")]` → `/api/dispensationplans`

| # | Method | Endpoint | الوظيفة | Auth |
|:-:|:---:|:---|:---|:---:|
| 1 | `GET` | `/api/dispensationplans/program/{programId}` | خطط برنامج | ⚠️ Anonymous |
| 2 | `GET` | `/api/dispensationplans/{planId}` | تفاصيل خطة | ⚠️ Anonymous |
| 3 | `POST` | `/api/dispensationplans` | إنشاء خطة | 🔒 |
| 4 | `PUT` | `/api/dispensationplans/{planId}` | تحديث | 🔒 |
| 5 | `DELETE` | `/api/dispensationplans/{planId}` | حذف | 🔒 |

### 11.4 `PharmaCompanyMedicationsController` (18 endpoints)

**Route:** `[Route("api/[controller]")]` → `/api/pharmacompanymedications`

**⚠️ بعض الـ endpoints `[AllowAnonymous]`.**

| # | Method | Endpoint | الوظيفة | Auth |
|:-:|:---:|:---|:---|:---:|
| 1 | `GET` | `/api/pharmacompanymedications` | قائمة الأدوية (Pagination) | ⚠️ Anonymous |
| 2 | `GET` | `/api/pharmacompanymedications/{id}` | تفاصيل | 🔒 |
| 3 | `POST` | `/api/pharmacompanymedications` | إنشاء | 🔒 |
| 4 | `PUT` | `/api/pharmacompanymedications/{id}` | تحديث | 🔒 |
| 5 | `DELETE` | `/api/pharmacompanymedications/{id}` | حذف (Soft) | 🔒 |
| 6 | `POST` | `/api/pharmacompanymedications/{id}/image/medication` | رفع صورة الدواء | 🔒 |
| 7 | `POST` | `/api/pharmacompanymedications/{id}/image/package` | رفع صورة العبوة | 🔒 |
| 8 | `GET` | `/api/pharmacompanymedications/search` | بحث | ⚠️ Anonymous |
| 9 | `GET` | `/api/pharmacompanymedications/by-category/{categoryId}` | حسب الفئة | 🔒 |
| 10 | `GET` | `/api/pharmacompanymedications/by-speciality/{specialityId}` | حسب التخصص | 🔒 |
| 11 | `GET` | `/api/pharmacompanymedications/{id}/specialities` | تخصصات الدواء | 🔒 |
| 12 | `POST` | `/api/pharmacompanymedications/{id}/specialities/{specialityId}` | إضافة تخصص | 🔒 |
| 13 | `DELETE` | `/api/pharmacompanymedications/{id}/specialities/{specialityId}` | حذف تخصص | 🔒 |
| 14 | `PUT` | `/api/pharmacompanymedications/{id}/specialities` | تحديث التخصصات | 🔒 |
| 15 | `POST` | `/api/pharmacompanymedications/{id}/approve` | موافقة | 🔒 |
| 16 | `POST` | `/api/pharmacompanymedications/{id}/reject` | رفض | 🔒 |
| 17 | `PATCH` | `/api/pharmacompanymedications/{id}/toggle-active` | تفعيل/تعطيل | 🔒 |
| 18 | `GET` | `/api/pharmacompanymedications/statistics/{companyId}` | إحصائيات | 🔒 |

**⚠️ ملاحظة:** `SharedStorage` — الصور تُحفظ في مسار مشترك:
```
{SolutionPath}/SharedStorage/uploads/
```

---

## 🎯 الجزء الثاني عشر: Points (مُوثَّق بدقة)

### 12.1 `PointSchemesController` (13 endpoints)

**Route:** `[Route("api/psp/points/schemes")]`

| # | Method | Endpoint | الوظيفة |
|:-:|:---:|:---|:---|
| 1 | `POST` | `/api/psp/points/schemes` | إنشاء Scheme (CampaignID + PointSystemTypeID) |
| 2 | `PUT` | `/api/psp/points/schemes/{schemeId}` | تحديث |
| 3 | `GET` | `/api/psp/points/schemes/by-program/{programId}` | Scheme واحد (PSP) |
| 4 | `POST` | `/api/psp/points/schemes/{schemeId}/deactivate` | تعطيل |
| 5 | `GET` | `/api/psp/points/schemes/list-by-program/{programId}` | قائمة Schemes |
| 6 | `GET` | `/api/psp/points/schemes/programs/{programId}/overview` | نظرة عامة |
| 7 | `GET` | `/api/psp/points/schemes/programs/{programId}/clinics` | عيادات البرنامج |
| 8 | `GET` | `/api/psp/points/schemes/by-organization/{organizationId}` | حسب المنظمة |
| 9 | `GET` | `/api/psp/points/schemes/{schemeId:int}` | ⭐ تفاصيل |
| 10 | `GET` | `/api/psp/points/schemes/{schemeId:int}/clinics` | ⭐ عيادات Scheme |
| 11 | `GET` | `/api/psp/points/schemes/{schemeId:int}/beneficiary-types` | ⭐ أنواع المستفيدين |
| 12 | `POST` | `/api/psp/points/schemes/{schemeId:int}/beneficiary-types` | ⭐ إضافة نوع |
| 13 | `DELETE` | `/api/psp/points/schemes/{schemeId:int}/beneficiary-types/{id:int}` | ⭐ حذف نوع |

### 12.2 `PointRulesController` (5 endpoints)

**Route:** `[Route("api/psp/points/rules")]`

| # | Method | Endpoint | الوظيفة |
|:-:|:---:|:---|:---|
| 1 | `POST` | `/api/psp/points/rules` | إنشاء قاعدة |
| 2 | `PUT` | `/api/psp/points/rules/{ruleId}` | تحديث |
| 3 | `DELETE` | `/api/psp/points/rules/{ruleId}` | حذف |
| 4 | `GET` | `/api/psp/points/rules/by-scheme/{schemeId}` | قواعد Scheme |
| 5 | `POST` | `/api/psp/points/rules/{ruleId}/toggle` | تفعيل/تعطيل |

### 12.3 `PointBalancesController` (7 endpoints)

**Route:** `[Route("api/psp/points/balances")]`

| # | Method | Endpoint | الوظيفة |
|:-:|:---:|:---|:---|
| 1 | `GET` | `/api/psp/points/balances/by-organization` | حسب المنظمة |
| 2 | `GET` | `/api/psp/points/balances/by-scheme/{schemeId}` | حسب Scheme |
| 3 | `GET` | `/api/psp/points/balances/organization-summary/{organizationId}` | ملخص |
| 4 | `PUT` | `/api/psp/points/balances/visibility` | إعدادات الظهور |
| 5 | `GET` | `/api/psp/points/balances/visibility/check` | فحص |
| 6 | `PUT` | `/api/psp/points/balances/visibility/all/{schemeId}` | الكل |
| 7 | `GET` | `/api/psp/points/balances/by-company/{companyOrganizationId:int}` | ⭐ أرصدة الشركة |

### 12.4 `PointEarningController` (1 endpoint)

**Route:** `[Route("api/psp/points/transactions")]`

| # | Method | Endpoint | الوظيفة |
|:-:|:---:|:---|:---|
| 1 | `GET` | `/api/psp/points/transactions` | سجل المعاملات |

**⚠️ ملاحظة:** **لا يوجد Endpoint لـ `POST /earn`** — `EarnPointsHandler` يُستدعى **داخلياً** من Handlers أخرى (`CreatePSPPatientHandler`, `DispenseController`).

**⚠️ حد أقصى للـ `take`:** 200.

### 12.5 `PointRedeemController` (8 endpoints)

**Route:** `[Route("api/psp/points/redeem-requests")]`

| # | Method | Endpoint | الوظيفة |
|:-:|:---:|:---|:---|
| 1 | `POST` | `/api/psp/points/redeem-requests` | إنشاء طلب (الطبيب) |
| 2 | `POST` | `/api/psp/points/redeem-requests/{id}/accept` | قبول (الشركة) |
| 3 | `POST` | `/api/psp/points/redeem-requests/{id}/reject` | رفض (الشركة) |
| 4 | `POST` | `/api/psp/points/redeem-requests/{id}/confirm` | تأكيد (الطبيب) |
| 5 | `POST` | `/api/psp/points/redeem-requests/{id}/dispute` | اعتراض (الطبيب) |
| 6 | `POST` | `/api/psp/points/redeem-requests/{id}/resolve` | حل نزاع |
| 7 | `GET` | `/api/psp/points/redeem-requests` | قائمة (Pagination) |
| 8 | `GET` | `/api/psp/points/redeem-requests/{id}` | تفاصيل |

**⚠️ حد أقصى للـ `take`:** 200.

### 12.6 `PointLookupsController` (4 endpoints) ⭐ جديد (14 أكتوبر 2026)

**Route:** `[Route("api/psp/points/lookups")]`

| # | Method | Endpoint | الوظيفة |
|:-:|:---:|:---|:---|
| 1 | `GET` | `/api/psp/points/lookups/organization-types` | أنواع المنظمات (Lookup) |
| 2 | `GET` | `/api/psp/points/lookups/beneficiary-types` | أنواع المستفيدين |
| 3 | `GET` | `/api/psp/points/lookups/activity-types` | أنواع الأنشطة |
| 4 | `GET` | `/api/psp/points/lookups/programs` | ⭐ برامج الشركة |

**⚠️ `organization-types` و `beneficiary-types`** يدعمان فلترة حسب `pointSystemTypeID`.

---

## 👔 الجزء الثالث عشر: Rep (المندوب)

### 13.1 `RepController` (12 endpoints)

**Route:** `[Route("api/rep")]`

| # | Method | Endpoint | الوظيفة |
|:-:|:---:|:---|:---|
| 1 | `GET` | `/api/rep/programs` | برامج المندوب |
| 2 | `GET` | `/api/rep/my-doctors` | أطباء المندوب (USED) |
| 3 | `GET` | `/api/rep/my-pharmacies` | صيدليات المندوب (USED) |
| 4 | `GET` | `/api/rep/my-invitations` | سجل الدعوات |
| 5 | `POST` | `/api/rep/invitations/create` | إنشاء دعوة |
| 6 | `POST` | `/api/rep/invitations/{id}/resend` | إعادة إرسال |
| 7 | `GET` | `/api/rep/dashboard/stats` | إحصائيات اللوحة |
| 8 | `GET` | `/api/rep/my-network` | شبكتي |
| 9 | `GET` | `/api/rep/current-info` | معلومات المندوب |
| 10 | `GET` | `/api/rep/psp-programs` | برامج PSP للشركة |
| 11 | `GET` | `/api/rep/company-programs` | برامج شركة المندوب |
| 12 | `GET` | `/api/rep/invitations/{id}` | تفاصيل دعوة |

#### 📌 مثال: `POST /api/rep/invitations/create`

**Request:**
```json
{
  "programId": 2,
  "invitedOrganizationId": 1247,
  "recipientType": "DOCTOR",
  "customMessage": "رسالة اختيارية"
}
```

---

## 📊 الجزء الرابع عشر: Reports

### 14.1 `ReportsController` (8 endpoints)

**Route:** `[Route("api/reports")]`

| # | Method | Endpoint | الوظيفة |
|:-:|:---:|:---|:---|
| 1 | `GET` | `/api/reports/provider-funnel` | معدل تحويل الدعوات |
| 2 | `GET` | `/api/reports/patient-funnel` | قمع تحويل المريض |
| 3 | `GET` | `/api/reports/pharmacy-dispensation` | صرف الصيدليات |
| 4 | `GET` | `/api/reports/pharmacy-financial` | التقرير المالي |
| 5 | `GET` | `/api/reports/adherence` | الالتزام بالجرعات |
| 6 | `GET` | `/api/reports/program-lifecycle` | دورة حياة البرنامج |
| 7 | `GET` | `/api/reports/dashboard-summary` | لوحة التحكم |
| 8 | `GET` | `/api/reports/filters` | الفلاتر المتاحة |

---

## 📈 الجزء الخامس عشر: Dashboard & Misc

### 15.1 `DashboardController` (1 endpoint)

| # | Method | Endpoint | الوظيفة |
|:-:|:---:|:---|:---|
| 1 | `GET` | `/api/dashboard` | لوحة التحكم العامة |

### 15.2 `HealthController` (1 endpoint)

| # | Method | Endpoint | الوظيفة |
|:-:|:---:|:---|:---|
| 1 | `GET` | `/health` | Health Check |

### 15.3 `DrugFormsController` (1 endpoint)

| # | Method | Endpoint | الوظيفة |
|:-:|:---:|:---|:---|
| 1 | `GET` | `/api/drug-forms` | الأشكال الدوائية |

### 15.4 `MedicationCategoriesController` (1 endpoint)

| # | Method | Endpoint | الوظيفة |
|:-:|:---:|:---|:---|
| 1 | `GET` | `/api/medication-categories` | فئات الأدوية |

### 15.5 `LocalizationController` (2 endpoints)

| # | Method | Endpoint | الوظيفة |
|:-:|:---:|:---|:---|
| 1 | `GET` | `/api/localization/page/{page}` | ترجمات صفحة |
| 2 | `GET` | `/api/localization/key/{key}` | ترجمة واحدة |

---

## 🔧 الجزء السادس عشر: أنماط مشتركة

### 16.1 Authentication

**⚠️ معظم الـ endpoints تتطلب:**

```
Authorization: Bearer eyJhbGciOiJIUzI1NiIs...
```

**JWT Claims:**
- `userId` — معرف المستخدم (`AspNetUsers.Id`)
- `userProfileId` — معرف الملف الشخصي (`UserProfiles.UserProfileID`)
- `roles` — الأدوار النظامية
- `exp` — تاريخ الانتهاء

### 16.2 Response Format

**نجاح:**
```json
{
  "success": true,
  "data": { ... }
}
```

**فشل:**
```json
{
  "success": false,
  "message": "وصف الخطأ",
  "errors": {
    "Email": ["البريد الإلكتروني مطلوب"]
  }
}
```

### 16.3 Status Codes

| الكود | المعنى |
|:---:|:---|
| 200 | نجاح |
| 201 | تم الإنشاء |
| 400 | خطأ في البيانات |
| 401 | غير مصرح |
| 403 | ممنوع |
| 404 | غير موجود |
| 500 | خطأ داخلي |

### 16.4 Thin Controllers

**⚠️ القاعدة الذهبية:**

> **"Controllers للتنسيق فقط. المنطق في UseCase أو Handler."**

```csharp
// ✅ صحيح
[HttpPost]
public async Task<IActionResult> Create([FromBody] CreateRequest request)
{
    var result = await _createUseCase.ExecuteAsync(request);
    return result.IsSuccess ? Ok(result) : BadRequest(result);
}
```

---

## 🚨 الجزء السابع عشر: قائمة `AllowAnonymous` (تحتاج مراجعة أمنية)

| Controller | Endpoint | الحالة |
|:---|:---|:---:|
| `PspController.Programs` | `programs-temp` | ⚠️ **Temp** |
| `PspController.Programs` | `program-details-temp/{id}` | ⚠️ **Temp** |
| `DispensationPlansController` | `program/{programId}` | ⚠️ |
| `DispensationPlansController` | `{planId}` | ⚠️ |
| `DispenseController` | `validate-token` | ⚠️ |
| `DispenseController` | `confirm` | ⚠️ |
| `DispenseController` | `status-by-token/{tokenCode}` | ⚠️ |
| `PharmaCompanyMedicationsController` | `GetAll` | ⚠️ |
| `PharmaCompanyMedicationsController` | `Search` | ⚠️ |
| `PharmacyController` | `nearby` | ⚠️ |

---

## 📋 الجزء الثامن عشر: جدول الـ Endpoints الكامل

| المجموعة | عدد Controllers | عدد Endpoints |
|:---|:---:|:---:|
| Auth & User | 2 | 20 |
| Organizations | 4 | 13 |
| Clinical | 2 | 5 |
| GeoLocations | 3 | 6 |
| Messaging | 2 | 16 |
| Notifications | 1 | 9 |
| Clinic | 1 | 15 |
| Professional | 1 | 3 |
| Patient | 2 | 13 |
| **Pharmacy** | **3** | **27** |
| **PSP** | **15** | **68** |
| **Points** | **6** | **38** |
| Rep | 1 | 12 |
| Reports | 1 | 8 |
| Dashboard | 1 | 1 |
| Misc | 3 | 3 |
| Base | 1 | 8 |
| **المجموع** | **~50 ملف** | **~290** |

---

## 🔗 الجزء التاسع عشر: روابط ذات صلة

- [00 — Architecture Overview](./00-architecture-overview.md)
- [01 — Program.cs Foundation](./01-program-cs-foundation.md)
- [02 — Identity System](./02-identity-system.md)
- [04 — Dynamic Menus](./04-dynamic-menus.md)
- [07 — PSP System](./07-psp-system.md)
- [09 — API Guide (النسخة السابقة)](./09-api-guide.md)
- [13 — Clean Architecture](./13-clean-architecture-enforcement.md)
- [26 — Rubika Points System](./26-rubika-points-system.md)
- [27 — Reports System](./27-reports-system.md)

---

## 📝 الجزء العشرون: سجل التغييرات

| الإصدار | التاريخ | التغييرات |
|:---:|:---|:---|
| **1.0** | **17 مايو 2026** | الإصدار الأولي — 7 Controllers رئيسية |
| **2.0** | **16 أكتوبر 2026** | **شامل** — ~290 endpoint موثّق من الكود الفعلي: PSP (62) + Points (38) + Pharmacy (27) + Reminders (8) + PharmaCompanyMedications (18) |

---

**© 2026 RubikCare — للاستخدام الداخلي**

**نهاية الوثيقة ✅**

---
