# 01 — PATIENT JOURNEY (رحلة المريض)

الإصدار: 2.0 (Live State Edition)
آخر تحديث: 15 نوفمبر 2026
الحالة: ✅ موثق
الجمهور: فريق التطوير + AI Assistants + فريق دعم المرضى + صانعو الفيديوهات
الوثيقة الأم: [20-USER JOURNEY.md](../20-USER%20JOURNEY.md) — النسخة القديمة (v1.0)

## 🎯 نظرة سريعة

| العنصر | القيمة |
|---|---|
| **Persona** | Patient (المريض) |
| **الحساب** | `UserProfile` (شخص واحد) |
| **Dashboard** | `DashboardFlowPage` (Personal) |
| **الأدوار** | User |
| **المنصات** | Web + PWA + MAUI |
| **البرامج المدعومة** | PSP (Phase System) + B2C Commerce (مستقبلاً) |

## 📊 الرحلة الكاملة (8 Phases)

```
PHASE 1: ENROLLMENT (التسجيل)
    ↓
PHASE 2: PSP ENROLLMENT (الانضمام لبرنامج)
    ↓
PHASE 3: INITIAL TOKEN (التوكن الأول)
    ↓
PHASE 4: DISPENSATION (الصرف)
    ↓
PHASE 5: CONFIRMATION (تأكيد الاستلام)
    ↓
PHASE 6: DAILY ADHERENCE (الالتزام اليومي)
    ↓
PHASE 7: POINTS REDEMPTION (استخدام النقاط)
    ↓
PHASE 8: POINTS DASHBOARD (لوحة النقاط)
```

---

## 🚀 PHASE 1: ENROLLMENT (التسجيل)

### 📱 الخطوات (من منظور المستخدم)

1. **فتح التطبيق لأول مرة**
   - **الصفحة:** `OnboardingPage`.
   - **المحتوى:** 3-4 شرائح تعريفية (Swipe).
   - **الأزرار:** "تخطي" + "ابدأ".

2. **الضغط على "ابدأ" أو "إنشاء حساب"**
   - **الصفحة:** `RegisterPage`.
   - **الحقول:**
     - الاسم الكامل (عربي/إنجليزي).
     - البريد الإلكتروني.
     - رقم الهاتف.
     - كلمة المرور.
     - تأكيد كلمة المرور.
   - **التحقق:**
     - البريد الإلكتروني — فريد.
     - رقم الهاتف — فريد.
     - كلمة المرور — 6+ أحرف.

3. **إرسال OTP (إذا مفعل)**
   - **الصفحة:** `EmailVerificationPage`.
   - **الحقول:** 6 أرقام.
   - **Timer:** 60 ثانية لإعادة الإرسال.

4. **تسجيل الدخول**
   - **الصفحة:** `LoginView`.
   - **الحقول:** البريد الإلكتروني + كلمة المرور.
   - **خيارات:** "تذكرني" + "نسيت كلمة المرور".

5. **بعد تسجيل الدخول**
   - **التوجيه:** `//DashboardFlow` (Personal).
   - **`AppShell.InitializeAsync`:**
     - قراءة `auth_token` من `SecureStorage`.
     - `POST /api/user/session-bootstrap` — جلب الجلسة.
     - حفظ `user_profile_id` + `user_full_name` في `Preferences`.
     - التوجيه إلى Personal Dashboard.

### 🔌 Endpoints

| Endpoint | Method | الغرض |
|---|---|---|
| `/api/auth/register` | POST | إنشاء حساب |
| `/api/auth/login` | POST | تسجيل الدخول + JWT |
| `/api/user/session-bootstrap` | POST | جلب بيانات الجلسة (UserProfile + Memberships) |
| `/api/auth/send-otp` | POST | إرسال OTP |
| `/api/auth/verify-otp` | POST | التحقق من OTP |
| `/api/auth/forgot-password` | POST | نسيت كلمة المرور |

### 📂 الملفات المتأثرة

- `OnboardingPage.xaml` + `.xaml.cs`
- `RegisterPage.razor`
- `LoginView.razor`
- `EmailVerificationPage.xaml` + `.xaml.cs`
- `AppShell.xaml.cs` — `InitializeAsync`
- `AppShellViewModel.cs` — `LoadUserDataAsync`
- `AuthService.cs` — `LogoutAsync`

---

## 🎯 PHASE 2: PSP ENROLLMENT (الانضمام لبرنامج PSP)

### 📱 الخطوات

1. **استلام كود دعوة (InvitationCode)**
   - **المصدر:** Doctor (أو Rep) — يرسله عبر:
     - **SMS / WhatsApp** (يدوياً).
     - **QR Code** (من `InvitePatientPage`).
     - **صفحة "دعواتي"** (للمريض المسجل).

2. **الدخول إلى "برامج الدعم"**
   - **من Dashboard:** زر "برامج دعم المرضى" (`QuickActionsGrid`).
   - **من Flyout:** عنصر "بحث عن برامج" (`PspSearchLabel`).
   - **الصفحة:** `PspEntryPage` → `PspEntry` Component.

3. **إدخال الكود**
   - **الصفحة:** `PspEntry` Component.
   - **طريقتان:**
     - **QR Scan** (إذا كان الكود QR).
     - **إدخال يدوي** (PSP-XXXX-XXXX).
   - **التحقق:**
     - `POST /api/psp/accept-invitation`.

4. **بعد الانضمام**
   - **النظام يُنشئ:**
     - `PSPPatient` (CaseID) — إن لم يكن موجوداً.
     - `PSPParticipation` — للبرنامج الجديد.
     - `PSPeRX` — لكل دواء.
   - **التوجيه:** `PspDetailPage` (تفاصيل البرنامج).

### 🔌 Endpoints

| Endpoint | Method | الغرض |
|---|---|---|
| `/api/psp/accept-invitation` | POST | قبول كود الدعوة |
| `/api/psp/patient-details` | GET | تفاصيل البرنامج (للمريض) |
| `/api/psp/patients/{id}/programs` | GET | قائمة البرامج النشطة |

### 📂 الملفات المتأثرة

- `PspEntry.razor` + `PspEntry.razor.cs`
- `PspEntryPage.xaml` + `.xaml.cs`
- `PspDetailComponent.razor`
- `PspDetailFlow.xaml` + `.xaml.cs`

---

## 🎫 PHASE 3: INITIAL TOKEN (التوكن الأول)

### 📱 الخطوات

1. **النظام يولّد التوكن تلقائياً**
   - **بعد الانضمام:**
     - النظام يجلب `DispensationPlan` للبرنامج.
     - يُنشئ `PSPDispensation` (Status = "GENERATED").
     - يُولّد `TokenCode` (RC-YYYYMMDD-XXXX).
     - يُرسل Push Notification للمريض.

2. **المريض يرى التوكن**
   - **الصفحة:** `PspDetailPage`.
   - **المحتوى:**
     - QR Code (160×160).
     - `TokenCode` (نص).
     - تاريخ الانتهاء.
     - الحالة ("صالح" / "منتهي").
   - **الأزرار:**
     - **نسخ** — `CopyToken()`.
     - **مشاركة** — `ShareToken()`.
     - **إرسال للصيدلية** — `SendToPharmacy()`.

### 🔌 Endpoints

| Endpoint | Method | الغرض |
|---|---|---|
| (تلقائي) | — | توليد التوكن الأول |
| `/api/psp/patient-details` | GET | جلب التفاصيل (بما فيها TokenCode) |

### 📂 الملفات المتأثرة

- `PspDetailComponent.razor` — قسم "TOKEN SECTION"
- `TokenRedemptionCard.razor` (المستخدم من Pharmacist)
- `PharmacyDetailPage.razor` — يستقبل `TokenCode` عبر Query

---

## 💊 PHASE 4: DISPENSATION (الصرف)

### 📱 الخطوات (من منظور المريض)

1. **المريض يذهب للصيدلية**
   - **الخيارات:**
     - **صيدلية قريبة** — `NearbyPharmaciesPage`.
     - **صيدلية محددة** — `PharmacySearchPage`.
     - **صيدلية مقترحة** — من `PspDetailPage` → "ابحث عن صيدلية".

2. **يُظهر التوكن للصيدلي**
   - **QR Code** — يمسحه الصيدلي.
   - **TokenCode** — يُدخله يدوياً.

3. **الصيدلي يُدخل التوكن في النظام**
   - **الصفحة:** `TokenRedemptionCard` (داخل `PharmacyDetailPage`).
   - **الخطوات (من منظور الصيدلي):**
     - إدخال `TokenCode` أو مسح QR.
     - `POST /api/dispense/validate-token` — التحقق.
     - `POST /api/dispense/submit-for-approval` — إرسال للموافقة (Auto-Refresh كل 5s).
     - عند الموافقة — `POST /api/dispense/confirm` — تأكيد الصرف.

4. **بعد الصرف**
   - **Status = "DISPENSED"** (أو "APPROVED").
   - **`PspDetailPage` يُحدّث:**
     - قسم "متابعة الطلب" (زر لصفحة "طلباتي").
     - إخفاء قسم التوكن (لأنه استُخدم).

### 🔌 Endpoints

| Endpoint | Method | الغرض |
|---|---|---|
| `/api/dispense/validate-token` | POST | التحقق من التوكن |
| `/api/dispense/submit-for-approval` | POST | إرسال طلب الموافقة |
| `/api/dispense/status/{dispensationId}` | GET | حالة الموافقة (Polling) |
| `/api/dispense/confirm` | POST | تأكيد الصرف |
| `/api/dispense/can-dispense-token` | GET | التحقق من تعاقد الصيدلية |

### 📂 الملفات المتأثرة

- `TokenRedemptionCard.razor`
- `PharmacyDetailPage.razor`
- `PspDetailComponent.razor`

---

## ✅ PHASE 5: CONFIRMATION (تأكيد الاستلام)

### 📱 الخطوات

1. **المريض يستلم Push Notification**
   - **المحتوى:** "تم صرف دوائك — يرجى تأكيد الاستلام".
   - **عند الضغط:** يفتح `PspDetailPage` (أو `/my-orders`).

2. **المريض يفتح "طلباتي"**
   - **الصفحة:** `PatientOrdersBlazorPage` (من `/my-orders`).
   - **المحتوى:**
     - قائمة التوكنات.
     - الحالة ("تم الصرف" / "تم الاستلام").
     - زر "تأكيد الاستلام".

3. **المريض يضغط "تأكيد الاستلام"**
   - `POST /api/dispense/confirm-receipt`.
   - **بعد التأكيد:**
     - Status = "RECEIVED".
     - النظام **يولّد تلقائياً** توكن جديد (إذا كان هناك RemainingTokens).
     - المريض يرى توكن جديد في `PspDetailPage`.

### 🔌 Endpoints

| Endpoint | Method | الغرض |
|---|---|---|
| `/api/patient/medications/orders` | GET | قائمة طلبات المريض |
| `/api/dispense/confirm-receipt` | POST | تأكيد الاستلام |
| `/api/dispense/generate-next-token` | POST | توليد التوكن التالي (تلقائي) |

### 📂 الملفات المتأثرة

- `PatientOrdersFlow.xaml` + `.xaml.cs`
- `PatientOrdersBlazorPage.razor`
- `PspDetailComponent.razor`

---

## 📅 PHASE 6: DAILY ADHERENCE (الالتزام اليومي)

### 📱 الخطوات

1. **المريض يستلم تذكير يومي**
   - **Push Notification:**
     - **النوع:** `PSP_REMINDER` — **Data-Only** (يعمل في MAUI).
     - **المحتوى:** "حان موعد دواء [اسم الدواء]".
   - **عند الضغط:** يفتح `MedicineGuidePage` أو `MedicationScheduleFlow`.

2. **المريض يفتح "جدول الأدوية"**
   - **الصفحة:** `MedicationScheduleFlow` (MAUI) أو `/medication-schedule` (PWA/Web).
   - **المحتوى:**
     - قائمة الأدوية (اليوم).
     - الحالة ("لم تؤخذ" / "تم الأخذ").
     - زر "تناولت الجرعة".

3. **المريض يضغط "تناولت الجرعة"**
   - `PUT /api/patient/medications/status`.
   - **النظام:**
     - يُحدّث `PatientMedicationSchedule.Status = "Taken"`.
     - يُحدّث `PSPReminder.Status = "Completed"`.
     - **يمنح نقاط** — عبر `IEarnPointsOrchestrator.AwardByCampaignAsync`:
       - `ActivityTypeID = PointActivityTypeIds.DailyDoseTaken`.
       - `ReferenceType = "SCHEDULE"`.
       - `ReferenceID = scheduleId`.
     - **Idempotency:** فحص مسبق لمنع التكرار.

### 🔌 Endpoints

| Endpoint | Method | الغرض |
|---|---|---|
| `/api/patient/medications/schedules/{userProfileId}?date=YYYY-MM-DD` | GET | جدول اليوم |
| `/api/patient/medications/status` | PUT | تحديث حالة الجرعة |
| (تلقائي) | — | منح نقاط الالتزام |

### 📂 الملفات المتأثرة

- `MedicationScheduleFlow.xaml` + `.xaml.cs`
- `MedicineGuideFlow.xaml` + `.xaml.cs`
- `MedicineGuidePage.razor` (Shared.UI)
- `AcknowledgeReminderHandler.cs` (Backend)
- `IEarnPointsOrchestrator.cs`

---

## 💎 PHASE 7: POINTS REDEMPTION (استخدام النقاط)

### 📱 الخطوات

1. **المريض يفتح "لوحة النقاط"**
   - **من Flyout:** "محفظة النقاط" → `patient/points` أو `PointsDashboardPage`.
   - **من PspDetail:** بطاقة "محفظة النقاط" في الأعلى.
   - **الصفحة:** `PointDashboard` (Shared.UI).

2. **المحتوى:**
   - الرصيد الإجمالي.
   - إجمالي المكتسب.
   - إجمالي المستخدم.
   - الالتزام (%).
   - Badges.
   - Streak.
   - الخصم الديناميكي.

3. **المريض يقرر استخدام النقاط**
   - **السيناريو 1 — تجديد التوكن:**
     - من `PspDetailPage` — بطاقة "استخدم نقاطي".
     - **يفتح:** `RedeemDialog`.
     - **يُدخل:** عدد النقاط.
     - **يضغط:** "تطبيق وتجديد التوكن".
     - **Endpoint:**
       - إذا التوكن الحالي نشط (ACTIVE / GENERATED) → `POST /api/dispense/apply-points`.
       - إذا التوكن منتهي (EXPIRED / RECEIVED) → `POST /api/dispense/generate-next-token` (مع `PointsToRedeem`).
     - **بعد التأكيد:**
       - خصم X نقطة من `PSPProgramPointBalance`.
       - `PSPProgramPointTransaction` (Points = -X, Type = 9 = RedeemDispensation).
       - تحديث `PatientPayment` (ينخفض).
       - تحديث `PSPCoverage` (يرتفع).
       - تحديث `Dispensation` (PointsUsedInCurrentToken = true).

### 🔌 Endpoints

| Endpoint | Method | الغرض |
|---|---|---|
| `/api/psp/points/balances/patient-dashboard/{userProfileId}` | GET | لوحة النقاط |
| `/api/dispense/apply-points` | POST | تطبيق النقاط على التوكن الحالي |
| `/api/dispense/generate-next-token` | POST | تجديد + استخدام النقاط |

### 📂 الملفات المتأثرة

- `PointDashboard.razor`
- `PatientPointsFlow.xaml` + `.xaml.cs`
- `PspDetailComponent.razor` — قسم Redeem
- `PointsDashboardPage.xaml` + `.xaml.cs`

---

## 🏆 PHASE 8: POINTS DASHBOARD (لوحة النقاط)

### 📱 الخطوات (أي وقت)

1. **المريض يفتح "لوحة النقاط"**
   - **من Personal Dashboard** — بطاقة "محفظة النقاط" (Hero Stats).
   - **من Flyout** — "النقاط".
   - **من `PspDetailPage`** — بطاقة "محفظة النقاط".
   - **الصفحة:** `patient/points` أو `PointsDashboardPage`.

2. **المحتوى:**
   - **Hero Stats:**
     - الرصيد الإجمالي.
     - القيمة النقدية.
     - الالتزام.
   - **قائمة البرامج:**
     - كل برنامج برصيده.
     - Progress bar للالتزام.
   - **Badges / Streak:**
     - من `PointDashboard`.
   - **الـ Redemption History:**
     - كل معاملة (Earn / Redeem).

### 🔌 Endpoints

| Endpoint | Method | الغرض |
|---|---|---|
| `/api/psp/points/balances/patient-dashboard/{userProfileId}` | GET | كل بيانات لوحة النقاط |

---

## 🌐 الميزات المشتركة في رحلة المريض

### 1. Offline Mode
- **يعمل:**
  - الترجمات (من Cache).
  - جلسة المستخدم (من `Preferences`).
  - القائمة الجانبية.
  - عرض بيانات آخر تحميل (Dashboard, PspDetail).
- **لا يعمل:**
  - الإحصائيات الحية (تظهر 0).
  - إرسال التوكن للصيدلية.
  - تحديث النقاط.
- **Banner "غير متصل":** يظهر في كل الصفحات (Flyout + TitleView).

### 2. Translation (AR/EN) + RTL
- **اللغة الافتراضية:** العربية.
- **مفتاح التبديل:** في `SettingsPage` → `LanguageSection`.
- **RTL:** يعمل تلقائياً (CSS `inset-inline-start/end`).
- **Cache:** 4 مستويات Fallback (Memory → API → Page Cache → All Cache).

### 3. Central Back Button
- **في `AppShell.OnBackButtonPressed`:**
  - إذا `NavigationStack > 1` → ارجع.
  - إذا `Personal Dashboard` → "هل تريد الخروج؟" → `App.Quit()`.
- **في `ChatPage`:** زر Back → قائمة المحادثات.

### 4. Messaging
- **المريض ←→ Doctor** (حجز موعد).
- **المريض ←→ Pharmacy** (استفسار، توكن).
- **الصفحات:**
  - `MessagingHubPage` — تبويب المحادثات.
  - `ChatPage` — محادثة محددة.
  - `ConversationsListPage` — القائمة.
- **الاسم المعروض:** اسم الطرف الآخر (`DisplayName`).
- **الإنشاء:** عند إرسال حجز من `DoctorProfilePage` → `POST /api/messaging/conversations` + `POST /api/messaging/messages`.

### 5. Notifications
- **Push (FCM):**
  - عند توليد توكن.
  - عند الموافقة من الصيدلية.
  - عند صرف الدواء.
  - تذكيرات يومية (PSP_REMINDER — Data-Only).
- **In-App:**
  - زر الإشعارات (🔔) في `Shell.TitleView`.
  - Counter (HasUnreadNotifications).
- **Local (MAUI فقط):**
  - تذكيرات الجرعات.

---

## 📊 جدول المراحل (ملخص)

| # | المرحلة | Endpoint | الحالة |
|---|---|---|---|
| 1 | Onboarding + Registration | `POST /api/auth/register` | ✅ |
| 2 | Login | `POST /api/auth/login` | ✅ |
| 3 | Session Bootstrap | `POST /api/user/session-bootstrap` | ✅ |
| 4 | PSP Enrollment | `POST /api/psp/accept-invitation` | ✅ |
| 5 | View Token | `GET /api/psp/patient-details` | ✅ |
| 6 | Send Token | `POST /api/dispense/validate-token` | ✅ |
| 7 | Dispense (Pharmacist) | `POST /api/dispense/confirm` | ✅ |
| 8 | Confirm Receipt | `POST /api/dispense/confirm-receipt` | ✅ |
| 9 | Daily Adherence | `PUT /api/patient/medications/status` | ✅ |
| 10 | Points Dashboard | `GET /api/psp/points/balances/patient-dashboard/{id}` | ✅ |
| 11 | Points Redemption | `POST /api/dispense/apply-points` | ✅ |

---

## 🎯 السيناريوهات الشائعة

### سيناريو 1: مريض جديد + برنامج واحد
```
1. Onboarding (3 شرائح)
2. Register (email + password + OTP)
3. Login → DashboardFlow
4. من Doctor → استلام كود دعوة
5. PspEntry → إدخال الكود
6. PspDetail → رؤية التوكن
7. Send To Pharmacy
8. Pharmacist Dispenses → Push Notification
9. Confirm Receipt → Token جديد
10. Daily Adherence (يومياً)
11. Points → Redeem (اختياري)
```

**الوقت المتوقع:** ~5 دقائق (Online).

### سيناريو 2: مريض لديه 3 برامج
- `DashboardFlowPage` يعرض 3 بطاقات.
- `PspDetailPage` — لكل برنامج توكن منفصل.
- `PointDashboard` — رصيد لكل برنامج (Scheme).

### سيناريو 3: مريض Offline
- **يفتح التطبيق:** Spinner → Dashboard (من Cache).
- **Banner:** "أنت غير متصل".
- **القائمة:** تعمل.
- **الإحصائيات:** 0.
- **عند إرسال توكن:** فشل — "تأكد من الاتصال".

---

## 📂 الملفات المتأثرة (كاملة)

### Mobile (MAUI)
- `OnboardingPage.xaml` + `.xaml.cs`
- `LoginView.xaml` + `.xaml.cs` (أو `LoginView.razor`)
- `RegisterPage.xaml` + `.xaml.cs` (أو `RegisterPage.razor`)
- `EmailVerificationPage.xaml` + `.xaml.cs`
- `AppShell.xaml` + `.xaml.cs`
- `AppShellViewModel.cs`
- `DashboardFlowPage.xaml` + `.xaml.cs`
- `PspEntryPage.xaml` + `.xaml.cs`
- `PspDetailFlow.xaml` + `.xaml.cs`
- `PatientPointsFlow.xaml` + `.xaml.cs`
- `PointsDashboardPage.xaml` + `.xaml.cs`
- `MedicationScheduleFlow.xaml` + `.xaml.cs`
- `MedicineGuideFlow.xaml` + `.xaml.cs`
- `PatientOrdersFlow.xaml` + `.xaml.cs`
- `MyPatientInvitationsFlow.xaml` + `.xaml.cs`

### Shared.UI
- `DashboardPage.razor`
- `PspEntry.razor` + `.razor.cs`
- `PspDetailComponent.razor`
- `ProgramDetailsWrapper.razor`
- `ProgramDetailsComponent.razor`
- `PointDashboard.razor`
- `PatientOrdersBlazorPage.razor`
- `MedicineGuidePage.razor`
- `TokenRedemptionCard.razor` (يظهر في Pharmacy)
- `MessagingHubPage.razor`
- `ChatPage.razor`
- `ConversationsListPage.razor`

### Backend (للمرجعية)
- `AuthController.cs`
- `PspController.cs`
- `DispenseController.cs`
- `PatientMedicationsController.cs`
- `PointsController.cs`
- `MessagingController.cs`
- `FirebaseNotificationService.cs`
- `IEarnPointsOrchestrator.cs`

---

## 🎯 ملاحظات للمطورين

### قاعدة `PatientProfileID` vs `CaseID`
- **`PatientProfileID`** = `UserProfileID` (الشخص) — لا يتكرر.
- **`CaseID`** = `PSPPatients.CaseID` — يتكرر لكل برنامج.
- **في `PointsContext`:** استخدم `CaseID = PatientProfileID` (لأن `EarnPointsHandler` يبحث عن `Balance` بـ `PatientProfileID`).
- **في `PSPDispensation`:** استخدم `ERxID → PSPeRX.CaseID → PSPPatient.CaseID`.

### قاعدة `T()` في Mobile
- **لا تُرجع المفتاح أبداً** — `string.Empty` أفضل.
- **`_isTranslationsLoaded` flag** — إلزامي.
- **`InvokeAsync(StateHasChanged)`** — بعد `await`.

### قاعدة Offline
- **Cache-first via `Preferences`** — للترجمات + الجلسة.
- **لا Cache للإحصائيات** (حتى Sprint مستقبلي).

---

## 🔗 روابط سريعة

- **[00-overview.md](00-overview.md)** — نظرة عامة.
- **[02-doctor.md](02-doctor.md)** — رحلة الطبيب.
- **[03-pharmacist.md](03-pharmacist.md)** — رحلة الصيدلي.
- **[06-cross-cutting.md](06-cross-cutting.md)** — الميزات المشتركة.
- **[05-PSP.md](../05-PSP.md)** — نظام PSP (الوثيقة التقنية).
- **[06-POINTS.md](../06-POINTS.md)** — نظام النقاط (الوثيقة التقنية).

© 2026 RubikCare — للاستخدام الداخلي
```
