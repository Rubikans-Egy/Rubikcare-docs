
# 02 — DOCTOR JOURNEY (رحلة الطبيب)

الإصدار: 1.0
آخر تحديث: 15 نوفمبر 2026
الحالة: ✅ موثق
الجمهور: فريق التطوير + AI Assistants + فريق دعم الأطباء + صانعو الفيديوهات

## 🎯 نظرة سريعة

| العنصر | القيمة |
|---|---|
| **Persona** | Doctor (الطبيب) |
| **الحساب** | `UserProfile` + دور `Doctor` |
| **العضوية** | `OrgMembership` (OrganizationTypeID = 4 = Clinic) |
| **Dashboard** | `ClinicDashboardPage` |
| **المنصات** | Web + PWA + MAUI |
| **البرامج المدعومة** | PSP + Clinic OS (مستقبلاً) |

## 📊 الرحلة الكاملة (6 Phases)

```
PHASE 1: REGISTRATION + PROFESSIONAL LICENSE
    ↓
PHASE 2: CLINIC MEMBERSHIP (عضوية العيادة)
    ↓
PHASE 3: CLINIC DASHBOARD (اليومي)
    ↓
PHASE 4: PSP PROGRAM ENROLLMENT
    ↓
PHASE 5: PATIENT INVITATION (دعوة المرضى)
    ↓
PHASE 6: PATIENT MANAGEMENT + REPORTS
```

---

## 🚀 PHASE 1: REGISTRATION + PROFESSIONAL LICENSE

### 📱 الخطوات

1. **التسجيل الأساسي**
   - مثل أي مستخدم — `OnboardingPage` → `RegisterPage`.
   - بعد التسجيل — يبدأ في Personal Mode.

2. **التسجيل المهني (Professional Registration)**
   - **من Personal Dashboard:** بطاقة "التسجيل المهني".
   - **من `SettingsPage`:** قسم "التسجيل المهني" (`ProfessionalSection`).
   - **من `AppShell`:** "التسجيل المهني" في الـ Flyout.

3. **رفع المستندات**
   - **الصفحة:** `ProfessionalStatusFlow` → `ProfessionalLicenseFlow`.
   - **الحقول:**
     - نوع المهنة (Doctor / Pharmacist).
     - رقم الترخيص.
     - التخصص الرئيسي (Speciality).
     - التخصص الفرعي.
     - صورة الترخيص.
     - صورة البطاقة الشخصية.

4. **الموافقة (من Admin)**
   - **الحالة الأولية:** `PENDING`.
   - **من Admin Dashboard:**
     - `AdminPendingLicenses.razor`.
     - عرض الطلبات.
     - زر "قبول" / "رفض".
   - **بعد الموافقة:**
     - الحالة = `APPROVED`.
     - النظام يُنشئ عضوية في عيادة (إذا طلب).

5. **بعد الموافقة**
   - **Push Notification:** "تمت الموافقة على طلبك".
   - **`AppShellViewModel.UserRole`** = "Doctor".
   - **`ShowProfessionalSection`** = true (يظهر "عيادتي" في Flyout).
   - **زر "الدخول إلى عيادتي"** في Dashboard.

### 🔌 Endpoints

| Endpoint | Method | الغرض |
|---|---|---|
| `/api/professional/register` | POST | التسجيل المهني |
| `/api/professional/check-status` | GET | حالة الترخيص |
| `/api/professional/upload-license` | POST | رفع صورة الترخيص |
| `/api/admin/pending-licenses` | GET | (Admin) قائمة الطلبات |
| `/api/admin/approve-license` | POST | (Admin) الموافقة |
| `/api/admin/reject-license` | POST | (Admin) الرفض |

### 📂 الملفات المتأثرة

- `ProfessionalStatusFlow.xaml` + `.xaml.cs`
- `ProfessionalLicenseFlow.xaml` + `.xaml.cs`
- `ProfessionalLicensePage.razor`
- `ProfessionalStatusPage.razor`
- `SettingsPage.razor` — `ProfessionalSection`
- `AdminPendingLicenses.razor` (Web)

---

## 🏥 PHASE 2: CLINIC MEMBERSHIP (عضوية العيادة)

### 📱 الخطوات

1. **الحصول على عضوية عيادة**
   - **السيناريو 1 — دعوة من Rep:**
     - مندوب يرسل دعوة لـ "عيادة" → `InviteDoctorPage`.
     - الطبيب يقبل الدعوة → عضوية في العيادة.
   - **السيناريو 2 — إنشاء عيادة جديدة:**
     - من `create-organization` → `CreateOrganizationFlow`.
     - ملء بيانات العيادة.
     - Admin يوافق.

2. **التبديل إلى Clinic Mode**
   - **من Flyout:** "عيادتي" (`NavigateToMyClinicCommand`).
   - **من Dashboard:** زر "عيادتي".
   - **`AppShellViewModel.SwitchToProfessionalMode("Doctor")`:**
     - جلب `session.Clinics.FirstOrDefault()`.
     - `_appState.CurrentOrganizationId = clinic.OrganizationId`.
     - `CurrentOrganizationState.SetOrganizationId(clinic.OrganizationId)`.
     - `GoToAsync("//ClinicDashboardPage?ClinicId=...")`.

3. **بعد التبديل**
   - `CurrentMode = AppMode.Doctor`.
   - `IsInProfessionalMode = true`.
   - الـ Flyout يعرض "الوضع المهني - طبيب".
   - زر "العودة للوضع الشخصي" يظهر.

### 🔌 Endpoints

| Endpoint | Method | الغرض |
|---|---|---|
| `/api/clinic/my-clinics` | GET | عيادات المستخدم |
| `/api/clinic/{id}/details` | GET | تفاصيل العيادة |
| `/api/organizations/create` | POST | إنشاء عيادة جديدة (إن وجد) |
| `/api/rep/invitations/create` | POST | (Rep) إرسال دعوة للعيادة |
| `/api/rep/invitations/accept` | POST | قبول دعوة |

### 📂 الملفات المتأثرة

- `AppShellViewModel.cs` — `SwitchToProfessionalMode`
- `AppShell.xaml` — FlyoutItem for Clinic
- `ClinicDashboardPage.xaml` + `.xaml.cs`
- `CreateOrganizationFlow.xaml` + `.xaml.cs`

---

## 🏥 PHASE 3: CLINIC DASHBOARD (اليومي)

### 📱 الصفحة الرئيسية

**`ClinicDashboardPage` → `ClinicDashboard.razor`:**

1. **Hero Section**
   - اسم العيادة (`_stats?.ClinicName`).
   - اسم الطبيب (`_stats?.DoctorName`).
   - **Pills:**
     - عدد المرضى (`PatientsCount`).
     - عدد البرامج النشطة (`ActivePrograms`).
   - **Stats Row:**
     - المرضى.
     - البرامج.
     - مواعيد اليوم (`TodayAppointments`).

2. **Quick Actions**
   - **عيادتي** → `MyClinicPage`.
   - **برامج PSP** → `PspSearchPage`.
   - **قائمة المرضى** → `ClinicPatientsPage`.
   - **دعواتي** → `MyPatientInvitationsFlow`.

3. **Next Appointment Section**
   - الموعد القادم.
   - زر "تأكيد" (مستقبلاً).

4. **Reminders Section**
   - دعوات معلقة (`PendingInvitations`).

### 🔌 Endpoints

| Endpoint | Method | الغرض |
|---|---|---|
| `/api/clinic/dashboard/stats?clinicId={id}` | GET | إحصائيات العيادة |

### 📂 الملفات المتأثرة

- `ClinicDashboard.razor`
- `ClinicDashboardPage.xaml` + `.xaml.cs`

---

## 🌿 PHASE 4: PSP PROGRAM ENROLLMENT (الانضمام لبرنامج)

### 📱 الخطوات

1. **البحث عن برامج PSP**
   - **من Quick Actions:** "برامج PSP".
   - **الصفحة:** `PspSearchPage.razor` (Shared.UI).
   - **المحتوى:**
     - قائمة البرامج المتاحة.
     - فلترة حسب الشركة، الفئة، إلخ.

2. **الاطلاع على تفاصيل البرنامج**
   - **الصفحة:** `ProgramDetailsWrapper.razor` → `ProgramDetailsComponent.razor`.
   - **المحتوى:**
     - اسم البرنامج.
     - الوصف.
     - الشركة الداعمة.
     - الأدوية المشمولة.
     - الروابط الخارجية.
     - **بطاقة "نقاطي"** — إذا كان للمنظمة `Scheme`.
     - زر "اشترك" — إذا الطبيب مؤهل.

3. **الاشتراك في البرنامج**
   - **Endpoint:** `POST /api/psp/participate`.
   - **Payload:**
     ```json
     {
       "ProgramId": 1,
       "OrganizationId": 1267,
       "JoinMethod": "DIRECT",  // أو "REP_INVITATION"
       "ParticipationType": "Clinic",
       "Notes": "اشتراك عبر التطبيق"
     }
     ```
   - **بعد الاشتراك:**
     - `_isSubscribed = true`.
     - زر "اشترك" → "إلغاء الاشتراك".
     - زر "إعادة الاشتراك" — إذا `CanResubscribe`.

### 🔌 Endpoints

| Endpoint | Method | الغرض |
|---|---|---|
| `/api/psp/programs` | GET | قائمة البرامج |
| `/api/psp/programs/{id}?organizationId={orgId}` | GET | تفاصيل البرنامج |
| `/api/psp/participate` | POST | الاشتراك |
| `/api/psp/unsubscribe` | POST | إلغاء الاشتراك |

### 📂 الملفات المتأثرة

- `PspSearchPage.razor`
- `ProgramDetailsWrapper.razor`
- `ProgramDetailsComponent.razor`

---

## 💌 PHASE 5: PATIENT INVITATION (دعوة المرضى)

### 📱 الخطوات

1. **فتح صفحة الدعوة**
   - **من Clinic Dashboard:** "دعوة مريض".
   - **من Quick Actions:** `ClinicNavigationBridge.NavigateToInvitePatient`.
   - **الصفحة:** `InvitePatientPage.xaml` (MAUI) → `InvitePatientViewModel.cs`.
   - **أو** `InvitePatientComponent.razor` (Shared.UI — للـ PWA).

2. **اختيار البرنامج**
   - إذا لم يكن هناك برنامج محدد:
     - تحميل قائمة البرامج (`LoadProgramsAsync`).
     - اختيار برنامج → `SelectProgramForInvitation`.

3. **إنشاء الدعوة**
   - **Endpoint:** `POST /api/psp/create-invitation`.
   - **Payload:**
     ```json
     {
       "ProgramId": 1,
       "ClinicId": 1267
     }
     ```
   - **Response:**
     ```json
     {
       "Success": true,
       "InvitationId": 456,
       "InvitationToken": "PSP-2026-XXXX"
     }
     ```

4. **عرض الدعوة**
   - **QR Code:** 160×160.
   - **TokenCode:** PSP-2026-XXXX.
   - **زر "نسخ".**
   - **زر "مشاركة".**
   - **زر "إنشاء وصفة طبية"** (مستقبلاً).

### 🔌 Endpoints

| Endpoint | Method | الغرض |
|---|---|---|
| `/api/psp/create-invitation` | POST | إنشاء كود دعوة |
| `/api/rep/programs?companyId={id}` | GET | (إن كان الطبيب مندوباً أيضاً) |

### 📂 الملفات المتأثرة

- `InvitePatientPage.xaml` + `.xaml.cs`
- `InvitePatientViewModel.cs`
- `InvitePatientComponent.razor`

---

## 👥 PHASE 6: PATIENT MANAGEMENT + REPORTS

### 📱 الخطوات

1. **عرض قائمة المرضى**
   - **الصفحة:** `ClinicPatientsPage.xaml` + `.xaml.cs`.
   - **المحتوى:**
     - قائمة المرضى (`ClinicPatientsList`).
     - بحث + فلترة.
     - حالة كل مريض (Active / Pending / Inactive).

2. **عرض تفاصيل مريض**
   - **الصفحة:** `ClinicPatientDetailsPage.xaml` + `.xaml.cs`.
   - **المحتوى:**
     - بيانات المريض.
     - البرامج المسجل فيها.
     - الوصفات (`PSPeRX`).
     - التحاليل (`PSPTestResult`).
     - المواعيد.

3. **عرض التحاليل**
   - **من تفاصيل المريض:** زر "التحاليل".
   - **الصفحة:** `PatientTestsFlow` → `PatientTests.razor`.
   - **المحتوى:**
     - قائمة التحاليل (`RequiredTest`).
     - حالة كل تحليل (Completed / Active / Locked).
     - تحميل نتائج (من المريض).
     - عرض النتائج.

4. **عرض المواعيد**
   - **من Clinic Dashboard:** "المواعيد".
   - **الصفحة:** `ClinicDashboardPage` → قسم Next Appointment.

5. **التقارير**
   - **الصفحة:** `ClinicReportsPage` (مستقبلاً).
   - **المحتوى:**
     - عدد المرضى.
     - البرامج النشطة.
     - معدل الالتزام.
     - الأداء المالي.

### 🔌 Endpoints

| Endpoint | Method | الغرض |
|---|---|---|
| `/api/clinic/patients?clinicId={id}` | GET | قائمة المرضى |
| `/api/clinic/patients/{patientId}/details` | GET | تفاصيل مريض |
| `/api/psp/patients/{id}/programs` | GET | برامج المريض |
| `/api/psp/patients/{id}/tests-with-status` | GET | التحاليل |

### 📂 الملفات المتأثرة

- `ClinicPatientsPage.xaml` + `.xaml.cs`
- `ClinicPatientDetailsPage.xaml` + `.xaml.cs`
- `PatientTestsFlow.xaml` + `.xaml.cs`
- `MyPatientInvitationsFlow.xaml` + `.xaml.cs`

---

## 🌐 الميزات المشتركة في رحلة الطبيب

### 1. Offline Mode
- **يعمل:**
  - الترجمات.
  - جلسة المستخدم + عضوية العيادة.
  - بيانات آخر تحميل (Clinic Dashboard).
- **لا يعمل:**
  - الإحصائيات (تظهر 0).
  - دعوة مرضى جديدة.
  - تحميل قائمة المرضى.

### 2. Central Back Button
- **في `ClinicDashboardPage`** — `AppShell.OnBackButtonPressed`:
  - إذا `NavigationStack > 1` → ارجع.
  - إذا `ClinicDashboardPage` → "هل تريد الخروج؟" → `App.Quit()`.

### 3. Start Page Preference (مستقبلاً)
- **⚠️ TODO:** الطبيب يختار `ClinicDashboardPage` كصفحة بدء → يتخطى Personal Dashboard.

### 4. Messaging
- **Doctor ←→ Patient** (حجز، استفسار).
- **Doctor ←→ Pharmacist** (وصفة، توكن).

### 5. Notifications
- **Push:**
  - عند قبول دعوة.
  - عند اشتراك مريض جديد.
  - عند صرف دواء لمريض.
- **In-App:** زر الإشعارات.

---

## 📊 جدول المراحل (ملخص)

| # | المرحلة | Endpoint | الحالة |
|---|---|---|---|
| 1 | Onboarding | (شاشات) | ✅ |
| 2 | Registration | `POST /api/auth/register` | ✅ |
| 3 | Professional License | `POST /api/professional/register` | ✅ |
| 4 | Session Bootstrap | `POST /api/user/session-bootstrap` | ✅ |
| 5 | Clinic Membership | `GET /api/clinic/my-clinics` | ✅ |
| 6 | Clinic Dashboard | `GET /api/clinic/dashboard/stats` | ✅ |
| 7 | PSP Search | `GET /api/psp/programs` | ✅ |
| 8 | PSP Enroll | `POST /api/psp/participate` | ✅ |
| 9 | Invite Patient | `POST /api/psp/create-invitation` | ✅ |
| 10 | Patient List | `GET /api/clinic/patients` | ✅ |
| 11 | Patient Tests | `GET /api/psp/patients/{id}/tests-with-status` | ✅ |

---

## 📂 الملفات المتأثرة (كاملة)

### Mobile (MAUI)
- `ProfessionalStatusFlow.xaml` + `.xaml.cs`
- `ProfessionalLicenseFlow.xaml` + `.xaml.cs`
- `ClinicDashboardPage.xaml` + `.xaml.cs`
- `MyClinicPage.xaml` + `.xaml.cs`
- `ClinicPatientsPage.xaml` + `.xaml.cs`
- `ClinicPatientDetailsPage.xaml` + `.xaml.cs`
- `MyPatientInvitationsFlow.xaml` + `.xaml.cs`
- `PspSearchPage.xaml` + `.xaml.cs`
- `ProgramDetailsPage.xaml` + `.xaml.cs`
- `PatientTestsFlow.xaml` + `.xaml.cs`
- `InvitePatientPage.xaml` + `.xaml.cs`
- `InvitePatientViewModel.cs`

### Shared.UI
- `ClinicDashboard.razor`
- `PspSearchPage.razor`
- `ProgramDetailsWrapper.razor`
- `ProgramDetailsComponent.razor`
- `InvitePatientComponent.razor`
- `PatientTests.razor`
- `SettingsPage.razor` — `ProfessionalSection`

### Backend (للمرجعية)
- `ProfessionalController.cs`
- `ClinicController.cs`
- `PspController.cs`
- `PatientTestsController.cs`

---

## 🔗 روابط سريعة

- **[00-overview.md](00-overview.md)**
- **[01-patient.md](01-patient.md)**
- **[03-pharmacist.md](03-pharmacist.md)**
- **[06-cross-cutting.md](06-cross-cutting.md)**
- **[05-PSP.md](../05-PSP.md)**

© 2026 RubikCare — للاستخدام الداخلي
```

---

## 📄 `docs/20-user-journeys/03-pharmacist.md`

**أنشئ الملف:**

```markdown
# 03 — PHARMACIST JOURNEY (رحلة الصيدلي)

الإصدار: 1.0
آخر تحديث: 15 نوفمبر 2026
الحالة: ✅ موثق
الجمهور: فريق التطوير + AI Assistants + فريق دعم الصيادلة + صانعو الفيديوهات

## 🎯 نظرة سريعة

| العنصر | القيمة |
|---|---|
| **Persona** | Pharmacist (الصيدلي) |
| **الحساب** | `UserProfile` + دور `Pharmacist` |
| **العضوية** | `OrgMembership` (OrganizationTypeID = 3 = Pharmacy) |
| **Dashboard** | `PharmacyDashboardPage` |
| **المنصات** | Web + PWA + MAUI |
| **البرامج المدعومة** | PSP + B2B Commerce + Pharmacy OS (مستقبلاً) |

## 📊 الرحلة الكاملة (6 Phases)

```
PHASE 1: REGISTRATION + PROFESSIONAL LICENSE
    ↓
PHASE 2: PHARMACY MEMBERSHIP
    ↓
PHASE 3: PHARMACY DASHBOARD
    ↓
PHASE 4: TOKEN REDEMPTION (صرف توكنات)
    ↓
PHASE 5: MEDICATION REQUESTS (طلبات أدوية)
    ↓
PHASE 6: B2B COMMERCE + REPORTS
```

---

## 🚀 PHASE 1: REGISTRATION + PROFESSIONAL LICENSE

**⚠️ نفس رحلة الطبيب تماماً — انظر [02-doctor.md](02-doctor.md).**

**الفرق:** بدلاً من "Doctor" — يختار "Pharmacist".

---

## 🏥 PHASE 2: PHARMACY MEMBERSHIP

### 📱 الخطوات

1. **الحصول على عضوية صيدلية**
   - **السيناريو 1 — دعوة من Rep:**
     - مندوب يرسل دعوة لـ "صيدلية".
   - **السيناريو 2 — إنشاء صيدلية جديدة:**
     - `create-organization` → `CreateOrganizationFlow`.

2. **التبديل إلى Pharmacy Mode**
   - **من Flyout:** "صيدليتي".
   - **`AppShellViewModel.SwitchToProfessionalMode("Pharmacist")`:**
     - جلب `session.Pharmacies.FirstOrDefault()`.
     - `CurrentOrganizationState.SetOrganizationId()`.
     - `GoToAsync("//PharmacyDashboardPage?PharmacyId=...")`.

3. **بعد التبديل**
   - `CurrentMode = AppMode.Pharmacist`.
   - Flyout يعرض "الوضع المهني - صيدلي".
   - زر "العودة للوضع الشخصي".

### 🔌 Endpoints

| Endpoint | Method | الغرض |
|---|---|---|
| `/api/clinic/my-pharmacies` | GET | صيدليات المستخدم |
| `/api/pharmacy/details/{id}` | GET | تفاصيل الصيدلية |

---

## 🏥 PHASE 3: PHARMACY DASHBOARD

### 📱 الصفحة الرئيسية

**`PharmacyDashboardPage` → `PharmacyDashboard.razor`:**

1. **Hero Section**
   - اسم الصيدلية.
   - اسم الصيدلي.
   - **Pills:**
     - صرف اليوم (`TodayDispenses`).
     - البرامج النشطة (`ActivePrograms`).
   - **Stats Row:**
     - صرف اليوم.
     - طلبات معلقة.
     - المرضى.

2. **Quick Actions**
   - **صيدليتي** → `MyPharmacyPage`.
   - **برامج PSP** → `PharmacyGatewayFlow`.
   - **التقارير** → `PharmacyReportsFlow`.
   - **طلبات المرضى** → `PatientOrdersFlow`.
   - **طلبات التوريد (B2B)** → `CommerceFlowPage`.

3. **Location Section (جديد)**
   - Latitude + Longitude.
   - زر "تحديد موقعي" (GPS).
   - زر "حفظ".

4. **Recent Dispenses**
   - آخر صرف.
   - زر "عرض الكل".

5. **Today Reminders**
   - طلبات Refills معلقة.
   - أو "لا توجد تذكيرات".

### 🔌 Endpoints

| Endpoint | Method | الغرض |
|---|---|---|
| `/api/pharmacy/dashboard/stats?pharmacyId={id}` | GET | إحصائيات الصيدلية |
| `/api/pharmacy/details/{id}` | GET | تفاصيل الصيدلية (بما فيها Location) |
| `/api/pharmacy/update-location` | PUT | حفظ الموقع |

### 📂 الملفات المتأثرة

- `PharmacyDashboard.razor`
- `PharmacyDashboardPage.xaml` + `.xaml.cs`
- `NearbyPharmacyService.cs` — `GetCurrentLocationAsync`

---

## 💊 PHASE 4: TOKEN REDEMPTION (صرف توكنات)

### 📱 الخطوات

1. **استلام التوكن من المريض**
   - **QR Code** — يمسحه الصيدلي.
   - **TokenCode** — يدخله يدوياً.

2. **فتح صفحة الصرف**
   - **المسار:** المريض → "إرسال للصيدلية" → `PharmacyDetailFlow?pharmacyId=X&token=Y`.
   - **أو:** الصيدلي يفتح `PharmacySearchPage` → يختار صيدلية → `PharmacyDetailFlow`.
   - **الصفحة:** `PharmacyDetailPage.razor` (Shared.UI).

3. **صرف التوكن (`TokenRedemptionCard`)**
   - **الخطوة 1 — إدخال التوكن:**
     - إدخال `TokenCode` أو مسح QR.
     - `POST /api/dispense/validate-token`.
     - إذا `Status = "APPROVED"` → `ShowDetails`.
     - إذا `Status = "PENDING_APPROVAL"` → `WaitingApproval`.
   - **الخطوة 2 — الانتظار (Auto-Refresh كل 5s):**
     - `GET /api/dispense/status/{dispensationId}`.
     - إذا `APPROVED` → `ShowDetails`.
     - إذا `REJECTED` → `EnterToken` + رسالة رفض.
   - **الخطوة 3 — عرض التفاصيل + تأكيد:**
     - تفاصيل الوصفة (المريض، الدواء، الكمية).
     - زر "تأكيد الاستلام".
     - `POST /api/dispense/confirm`.
   - **الخطوة 4 — النجاح:**
     - `Status = "DISPENSED"`.
     - Push Notification للمريض.

4. **التحقق من التعاقد**
   - إذا `TokenCode` غير موجود — `can-dispense-token`.
   - إذا الصيدلية غير متعاقدة → تحذير ("غير متعاقدة").

### 🔌 Endpoints

| Endpoint | Method | الغرض |
|---|---|---|
| `/api/dispense/can-dispense-token` | GET | التحقق من التعاقد |
| `/api/dispense/validate-token` | POST | التحقق من التوكن |
| `/api/dispense/submit-for-approval` | POST | إرسال الموافقة |
| `/api/dispense/status/{id}` | GET | حالة الموافقة |
| `/api/dispense/confirm` | POST | تأكيد الصرف |
| `/api/dispense/apply-points` | POST | تطبيق نقاط المريض |

### 📂 الملفات المتأثرة

- `PharmacyDetailPage.razor`
- `PharmacyDetailFlow.xaml` + `.xaml.cs`
- `TokenRedemptionCard.razor`

---

## 💊 PHASE 5: MEDICATION REQUESTS (طلبات أدوية)

### 📱 الخطوات

1. **إنشاء طلب دواء**
   - **الصفحة:** `MedicationRequestCard.razor` (Shared.UI).
   - **الخطوات:**
     - البحث عن دواء (`SearchMedications`).
     - اختيار الدواء (Checkbox).
     - تحديد الكمية + ملاحظات.
     - `SubmitRequest` → `POST /api/pharmacy/medication-request`.

2. **عرض طلباتي**
   - **من Quick Actions:** "طلبات المرضى".
   - **الصفحة:** `PatientOrdersFlow` → `PatientOrdersBlazorPage`.
   - **المحتوى:**
     - قائمة الطلبات.
     - حالة كل طلب (Pending / Approved / Delivered).
     - تفاصيل الطلب.

3. **عرض تفاصيل طلب**
   - **الصفحة:** `OrderDetailsPage.razor`.
   - **المحتوى:**
     - رقم الطلب.
     - بيانات المريض.
     - الأدوية.
     - زر "تتبع" → `OrderTrackingPage`.
     - زر "قبول" / "رفض" (حسب الصلاحية).

### 🔌 Endpoints

| Endpoint | Method | الغرض |
|---|---|---|
| `/api/pharmacy/medication-request` | POST | إنشاء طلب |
| `/api/commerce/supply-orders/pharmacy/{id}` | GET | طلبات الصيدلية |
| `/api/commerce/supply-orders/{id}` | GET | تفاصيل طلب |

### 📂 الملفات المتأثرة

- `MedicationRequestCard.razor`
- `PharmacyOrdersPage.razor`
- `PatientOrdersFlow.xaml` + `.xaml.cs`
- `OrderDetailsPage.razor`
- `OrderTrackingPage.razor`

---

## 🛒 PHASE 6: B2B COMMERCE + REPORTS

### 📱 الخطوات

1. **فتح Commerce**
   - **من Quick Actions:** "طلبات التوريد (B2B)".
   - **الصفحة:** `CommerceFlowPage.razor` (Shared.UI).

2. **تصفح الكتالوج**
   - **الصفحة:** `PharmacyCatalogPage.razor`.
   - **المحتوى:**
     - اختيار شركة الأدوية (`_selectedCompanyId`).
     - قائمة الأدوية.
     - بحث + Grid/List.
     - إضافة للسلة (`AddToCart`).

3. **عرض تفاصيل منتج**
   - **الصفحة:** `ProductDetailsPage.razor`.
   - **المحتوى:**
     - معلومات الدواء.
     - الأسعار.
     - الخصومات.
     - البونص (`BonusDto`).
     - زر "إضافة للسلة".

4. **عرض السلة**
   - **الصفحة:** `PharmacyCartPage.razor`.
   - **المحتوى:**
     - قائمة الأدوية.
     - تعديل الكميات.
     - حساب الإجمالي (SubTotal + Discount + Tax).
     - زر "تأكيد الطلب" → `SubmitOrderAsync`.
     - **Endpoint:** `POST /api/commerce/supply-orders`.
     - **ثم:** `POST /api/commerce/supply-orders/{id}/submit`.

5. **متابعة الطلبات**
   - **الصفحة:** `PharmacyOrdersPage.razor`.
   - **المحتوى:**
     - قائمة الطلبات.
     - فلترة حسب الحالة.
     - تفاصيل كل طلب.

6. **تتبع الطلب**
   - **الصفحة:** `OrderTrackingPage.razor`.
   - **المحتوى:**
     - Timeline (Draft → Submitted → Confirmed → Preparing → Shipped → Delivered).
     - زر "رجوع".

7. **التقارير**
   - **الصفحة:** `PharmacyReportsFlow` (مستقبلاً).
   - **المحتوى:**
     - صرف اليوم / الأسبوع / الشهر.
     - الأدوية الأكثر صرفاً.
     - الأرباح.
     - الالتزام.

### 🔌 Endpoints

| Endpoint | Method | الغرض |
|---|---|---|
| `/api/commerce/catalog/companies` | GET | شركات الأدوية المتاحة |
| `/api/commerce/catalog/{companyId}` | GET | كتالوج الشركة |
| `/api/commerce/catalog/{companyId}/search?term={term}` | GET | البحث |
| `/api/commerce/bonuses/company/{companyId}` | GET | البونص |
| `/api/commerce/supply-orders` | POST | إنشاء طلب |
| `/api/commerce/supply-orders/{id}/submit` | POST | إرسال الطلب |
| `/api/commerce/supply-orders/pharmacy/{id}` | GET | طلبات الصيدلية |

### 📂 الملفات المتأثرة

- `CommerceFlowPage.razor`
- `PharmacyCatalogPage.razor`
- `ProductDetailsPage.razor`
- `PharmacyCartPage.razor`
- `PharmacyOrdersPage.razor`
- `OrderTrackingPage.razor`

---

## 🌐 الميزات المشتركة في رحلة الصيدلي

### 1. Offline Mode
- **يعمل:**
  - الترجمات.
  - جلسة المستخدم + عضوية الصيدلية.
- **لا يعمل:**
  - الإحصائيات.
  - صرف توكنات جديدة.
  - طلبات B2B جديدة.

### 2. Central Back Button
- **في `PharmacyDashboardPage`** — `AppShell.OnBackButtonPressed`.
- **في `PharmacyDetailFlow`** — زر Back → `pharmacy-search` (عبر `IBackButtonHandler`).

### 3. Messaging
- **Pharmacist ←→ Patient** (استفسارات، توكن).
- **Pharmacist ←→ Rep** (طلبات توريد).

### 4. Notifications
- **Push:**
  - عند استلام طلب جديد.
  - عند صرف دواء بنجاح.
  - تذكيرات Refills.

---

## 📊 جدول المراحل (ملخص)

| # | المرحلة | Endpoint | الحالة |
|---|---|---|---|
| 1 | Onboarding | (شاشات) | ✅ |
| 2 | Registration | `POST /api/auth/register` | ✅ |
| 3 | Professional License | `POST /api/professional/register` | ✅ |
| 4 | Session Bootstrap | `POST /api/user/session-bootstrap` | ✅ |
| 5 | Pharmacy Membership | `GET /api/clinic/my-pharmacies` | ✅ |
| 6 | Pharmacy Dashboard | `GET /api/pharmacy/dashboard/stats` | ✅ |
| 7 | Validate Token | `POST /api/dispense/validate-token` | ✅ |
| 8 | Confirm Dispense | `POST /api/dispense/confirm` | ✅ |
| 9 | Medication Request | `POST /api/pharmacy/medication-request` | ✅ |
| 10 | B2B Order | `POST /api/commerce/supply-orders` | ✅ |
| 11 | Track Order | `GET /api/commerce/supply-orders/{id}` | ✅ |

---

## 📂 الملفات المتأثرة (كاملة)

### Mobile (MAUI)
- `ProfessionalStatusFlow.xaml` + `.xaml.cs`
- `ProfessionalLicenseFlow.xaml` + `.xaml.cs`
- `PharmacyDashboardPage.xaml` + `.xaml.cs`
- `MyPharmacyPage.xaml` + `.xaml.cs`
- `PharmacyGatewayFlow.xaml` + `.xaml.cs`
- `PharmacyReportsFlow.xaml` + `.xaml.cs`
- `PatientOrdersFlow.xaml` + `.xaml.cs`
- `PharmacySearchFlow.xaml` + `.xaml.cs`
- `PharmacyDetailFlow.xaml` + `.xaml.cs`
- `CommerceFlowPage.xaml` + `.xaml.cs`

### Shared.UI
- `PharmacyDashboard.razor`
- `PharmacySearchPage.razor`
- `PharmacyFilter.razor`
- `PharmacyDetailPage.razor`
- `TokenRedemptionCard.razor`
- `MedicationRequestCard.razor`
- `CommerceFlowPage.razor`
- `PharmacyCatalogPage.razor`
- `ProductDetailsPage.razor`
- `PharmacyCartPage.razor`
- `PharmacyOrdersPage.razor`
- `OrderTrackingPage.razor`

### Backend (للمرجعية)
- `DispenseController.cs`
- `PharmacyController.cs`
- `CommerceController.cs`
- `PharmacyMedicationRequestController.cs`

---

## 🔗 روابط سريعة

- **[00-overview.md](00-overview.md)**
- **[01-patient.md](01-patient.md)**
- **[02-doctor.md](02-doctor.md)**
- **[06-cross-cutting.md](06-cross-cutting.md)**
- **[05-PSP.md](../05-PSP.md)**
- **[23-COMMERCE-ENGINE.md](../23-COMMERCE-ENGINE.md)**

© 2026 RubikCare — للاستخدام الداخلي
```
