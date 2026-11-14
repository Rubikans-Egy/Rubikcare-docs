
# 04 — REP JOURNEY (رحلة المندوب)

الإصدار: 1.0
آخر تحديث: 15 نوفمبر 2026
الحالة: ✅ موثق
الجمهور: فريق التطوير + AI Assistants + فريق دعم المندوبين + صانعو الفيديوهات

## 🎯 نظرة سريعة

| العنصر | القيمة |
|---|---|
| **Persona** | Rep (المندوب) |
| **الحساب** | `UserProfile` + flag `IsRep = true` |
| **العضوية** | `OrgMembership` (OrganizationTypeID = 2 = PharmaCompany) |
| **Dashboards** | `PharmaCompanyDashboardPage` + `RepDashboard` |
| **المنصات** | Web + PWA + MAUI |
| **البرامج المدعومة** | PSP (Invitations) + B2B Commerce (Tracking) |

## 📊 الرحلة الكاملة (6 Phases)

```
PHASE 1: REGISTRATION + COMPANY MEMBERSHIP
    ↓
PHASE 2: PHARMA COMPANY DASHBOARD
    ↓
PHASE 3: REP DASHBOARD (Programs)
    ↓
PHASE 4: INVITE DOCTOR / PHARMACY
    ↓
PHASE 5: MY NETWORK
    ↓
PHASE 6: MY INVITATIONS + TRACKING
```

---

## 🚀 PHASE 1: REGISTRATION + COMPANY MEMBERSHIP

### 📱 الخطوات

1. **التسجيل الأساسي**
   - مثل أي مستخدم — `OnboardingPage` → `RegisterPage`.

2. **العضوية في شركة أدوية**
   - **السيناريو 1 — Admin الشركة يضيفه:**
     - Admin الشركة يستخدم `MembersManagementFlow` لإضافة المندوب.
   - **السيناريو 2 — دعوة:**
     - Admin الشركة يرسل دعوة → `InviteRepPage` (مستقبلاً).

3. **بعد العضوية**
   - **`AppShellViewModel.IsRep`** = true.
   - **`AvailableModes`** تتضمن `AppMode.Rep`.
   - **التبديل إلى Rep Mode** — من Flyout ("لوحة المندوب").

4. **التبديل إلى Rep Mode**
   - **`AppShellViewModel.SwitchToProfessionalMode("Rep")`:**
     - جلب `session.Memberships.FirstOrDefault(m => m.OrganizationTypeInfo?.TypeId == 2)`.
     - `GoToAsync("//PharmaCompanyDashboardPage?companyId=...")`.
   - **بعد التبديل:**
     - `CurrentMode = AppMode.Rep`.
     - Flyout يعرض "الوضع الوظيفي - مندوب".

### 🔌 Endpoints

| Endpoint | Method | الغرض |
|---|---|---|
| `/api/auth/register` | POST | التسجيل |
| `/api/user/session-bootstrap` | POST | جلب الجلسة (بما فيها `IsRep`) |
| `/api/organization/members` | GET | أعضاء الشركة |

### 📂 الملفات المتأثرة

- `AppShellViewModel.cs` — `SwitchToProfessionalMode`
- `PharmaCompanyDashboardPage.xaml` + `.xaml.cs`
- `MembersManagementFlow.xaml` + `.xaml.cs`

---

## 🏢 PHASE 2: PHARMA COMPANY DASHBOARD

### 📱 الصفحة الرئيسية

**`PharmaCompanyDashboardPage` → `PharmaCompanyDashboard.razor`:**

1. **Hero Section**
   - اسم الشركة (`_stats?.CompanyName`).
   - اسم المندوب (`_stats?.RepName`).
   - **Pills:**
     - عدد الأطباء المدعوين (`TotalDoctorsInvited`).
     - عدد الصيدليات المدعوة (`TotalPharmaciesInvited`).
   - **Stats Row:**
     - الأطباء.
     - الصيدليات.
     - معدل التحويل (`ConversionRate`).

2. **Quick Actions (4 بطاقات)**
   - **البرامج** → `RepDashboard` (`NavigateToRepDashboardWithCompany`).
   - **دعوة جديدة** → `InviteDoctorPage` (`NavigateToInvite`).
   - **شبكتي** → `MyNetworkPage` (`NavigateToDoctors`).
   - **سجل الدعوات** → `MyInvitationsPage` (`NavigateToPharmacies`).

3. **Recent Activity**
   - آخر نشاط (`LatestActivityTitle`).
   - تفاصيل (`LatestActivityDescription`).
   - تاريخ (`LatestActivityDate`).

4. **Reminders**
   - دعوات معلقة (`PendingInvitations`).

### 🔌 Endpoints

| Endpoint | Method | الغرض |
|---|---|---|
| `/api/rep/dashboard/stats?companyId={id}` | GET | إحصائيات المندوب |

### 📂 الملفات المتأثرة

- `PharmaCompanyDashboard.razor`
- `PharmaCompanyDashboardPage.xaml` + `.xaml.cs`
- `PharmaCompanyNavigationBridge.cs` (Bridges)

---

## 📊 PHASE 3: REP DASHBOARD (Programs)

### 📱 الخطوات

1. **فتح Rep Dashboard**
   - **من PharmaCompany Dashboard:** Quick Action "البرامج".
   - **أو:** Flyout → "لوحة المندوب".
   - **الصفحة:** `RepDashboard.razor` (Shared.UI).

2. **المحتوى**
   - **Hero:**
     - عنوان "لوحة المندوب".
     - زر "دعوة جديدة".
   - **Stats Row:**
     - الأطباء المدعوون.
     - الصيدليات المدعوة.
     - معدل التحويل.
   - **Programs Grid:**
     - قائمة برامج PSP النشطة للشركة.
     - كل برنامج → `RepProgramDto`:
       - `ProgramName`.
       - `ProgramCode`.
   - **عند اختيار برنامج:**
     - `RepNavigationBridge.OnProgramDetailsRequested?.Invoke(program.ProgramId)`.
     - يفتح `InviteDoctorPage` مع `ProgramId` في `QueryProperty`.

### 🔌 Endpoints

| Endpoint | Method | الغرض |
|---|---|---|
| `/api/rep/programs?companyId={id}` | GET | برامج الشركة |

### 📂 الملفات المتأثرة

- `RepDashboard.razor`
- `RepNavigationBridge.cs`

---

## 📨 PHASE 4: INVITE DOCTOR / PHARMACY

### 📱 الخطوات

1. **فتح صفحة الدعوة**
   - **من PharmaCompany Dashboard:** "دعوة جديدة".
   - **من Rep Dashboard:** "دعوة جديدة".
   - **الصفحة:** `InviteDoctorPage.xaml` (MAUI) → `InviteDoctorViewModel.cs`.
   - **أو:** `InviteDoctor.razor` (Shared.UI).

2. **اختيار النوع**
   - **Toggle:** Doctor / Pharmacist.
   - `SelectType(bool isDoctor)`.

3. **اختيار البرنامج (إن لم يكن محدداً)**
   - تحميل قائمة البرامج (`LoadProgramsAsync`).
   - اختيار برنامج → `SelectProgramForInvitation`.

4. **البحث عن الطبيب / الصيدلية**
   - **حقل البحث:** `_searchText`.
   - **Endpoint:**
     - Doctor: `GET /api/rep/doctors/search?q={term}&programId={id}`.
     - Pharmacy: `GET /api/organizations/pharmacies?searchTerm={term}&pageSize=20`.

5. **إرسال الدعوة**
   - **Button:** "دعوة" (`SendInvitation`).
   - **Endpoint:** `POST /api/rep/invitations/create`.
   - **Payload:**
     ```json
     {
       "ProgramId": 1,
       "InvitedOrganizationId": 456,
       "RecipientType": "DOCTOR",
       "CommunicationMethod": "IN_APP",
       "CustomMessage": "دعوة للانضمام لبرنامج X"
     }
     ```
   - **Response:**
     ```json
     {
       "Success": true,
       "InvitationToken": "PSP-2026-XXXX",
       "ExpiryDate": "2026-12-15"
     }
     ```

6. **عرض الدعوة**
   - **Modal** — Token + Expiry + Copy + Share.
   - **`RepNavigationBridge.ShareToken`** — للمشاركة عبر WhatsApp / Email.

7. **دعوة عامة (بدون منظمة محددة)**
   - **زر:** "دعوة عامة" (`CreateGeneralInvitation`).
   - **Payload:** `InvitedOrganizationId = 0`.
   - **الاستخدام:** رابط عام يُشارك على وسائل التواصل.

### 🔌 Endpoints

| Endpoint | Method | الغرض |
|---|---|---|
| `/api/rep/doctors/search` | GET | البحث عن أطباء |
| `/api/organizations/pharmacies` | GET | البحث عن صيدليات |
| `/api/rep/invitations/create` | POST | إنشاء دعوة |
| `/api/rep/invitations/stats?programId={id}&companyId={id}` | GET | إحصائيات الدعوات |

### 📂 الملفات المتأثرة

- `InviteDoctorPage.xaml` + `.xaml.cs`
- `InviteDoctorViewModel.cs`
- `InviteDoctor.razor`
- `RepNavigationBridge.cs` — `ShareToken`

---

## 👥 PHASE 5: MY NETWORK

### 📱 الخطوات

1. **فتح "شبكتي"**
   - **من PharmaCompany Dashboard:** Quick Action "شبكتي".
   - **الصفحة:** `MyNetworkPage.xaml` (MAUI) → `MyNetwork.razor` (Shared.UI).

2. **المحتوى**
   - **Hero:**
     - اسم المندوب.
     - **Pills:**
       - عدد الأطباء.
       - عدد الصيدليات.
   - **Tabs:**
     - **الأطباء** (`doctors`).
     - **الصيدليات** (`pharmacies`).
   - **Program Filter (جديد):**
     - Dropdown لاختيار برنامج.
     - `_selectedProgramId`.
   - **Search.**

3. **قائمة الأطباء**
   - كل طبيب → `MyDoctorDto`:
     - الاسم.
     - العيادة.
     - البرنامج.
     - InvitationStatus.
     - ReferralsCount.
     - EnrolledSince.

4. **قائمة الصيدليات**
   - كل صيدلية → `MyPharmacyDto`:
     - الاسم.
     - العنوان.
     - البرنامج.
     - DispensationsCount.

5. **عرض تفاصيل**
   - **`PharmaCompanyNavigationBridge.NavigateToNetworkDetails?.Invoke(orgId, name)`.**
   - **الصفحة:** `RepNetworkDetailsPage.xaml` → `RepNetworkDetails.razor`.

### 🔌 Endpoints

| Endpoint | Method | الغرض |
|---|---|---|
| `/api/rep/my-doctors?companyId={id}` | GET | أطباء الشبكة |
| `/api/rep/my-pharmacies?companyId={id}` | GET | صيدليات الشبكة |
| `/api/rep/invitation-history/{orgId}?companyId={id}` | GET | تاريخ الدعوات (لمنظمة) |

### 📂 الملفات المتأثرة

- `MyNetworkPage.xaml` + `.xaml.cs`
- `MyNetwork.razor`
- `RepNetworkDetailsPage.xaml` + `.xaml.cs`
- `RepNetworkDetails.razor`

---

## 📋 PHASE 6: MY INVITATIONS + TRACKING

### 📱 الخطوات

1. **فتح "دعواتي"**
   - **من PharmaCompany Dashboard:** "سجل الدعوات".
   - **الصفحة:** `MyInvitationsPage.xaml` → `MyInvitations.razor` (Shared.UI).

2. **المحتوى**
   - **Hero:**
     - "مرحباً" + عنوان.
     - **Pills:**
       - Total.
       - Pending.
       - Accepted.
   - **Stats Row.**
   - **Filter Tabs:**
     - ALL / PENDING / ACCEPTED / USED / EXPIRED.
   - **Search.**

3. **قائمة الدعوات**
   - كل دعوة → `InvitationWithStatusDto`:
     - البرنامج.
     - RecipientName.
     - RecipientType.
     - Status.
     - InvitationDate.
     - ExpiryDate.
     - InvitationToken.
   - **Actions (للمعلقة):**
     - نسخ الكود.
     - إعادة إرسال (`ResendInvitation`).

4. **عرض تفاصيل منظمة في الشبكة**
   - **الصفحة:** `RepNetworkDetailsPage` → `RepNetworkDetails.razor`.
   - **المحتوى:**
     - اسم المنظمة.
     - النوع (Pharmacy / Clinic).
     - قائمة الدعوات (Historique).
     - Filter + Search.

### 🔌 Endpoints

| Endpoint | Method | الغرض |
|---|---|---|
| `/api/rep/my-invitations?status=ALL&companyId={id}` | GET | دعوات المندوب |
| `/api/rep/invitations/{id}/resend` | POST | إعادة إرسال |
| `/api/rep/invitation-history/{orgId}` | GET | تاريخ الدعوات |

### 📂 الملفات المتأثرة

- `MyInvitationsPage.xaml` + `.xaml.cs`
- `MyInvitations.razor`
- `RepNetworkDetailsPage.xaml` + `.xaml.cs`
- `RepNetworkDetails.razor`

---

## 🌐 الميزات المشتركة في رحلة المندوب

### 1. Offline Mode
- **يعمل:**
  - الترجمات.
  - جلسة المستخدم + عضوية الشركة.
  - بيانات آخر تحميل (PharmaCompany Dashboard).
- **لا يعمل:**
  - الإحصائيات.
  - دعوات جديدة.
  - قائمة الشبكة.

### 2. Central Back Button
- **في `PharmaCompanyDashboardPage`** — `AppShell.OnBackButtonPressed`.

### 3. Messaging
- **Rep ←→ Pharmacy** (B2B).
- **Rep ←→ Doctor** (استفسار).

### 4. Notifications
- **Push:**
  - عند قبول دعوة.
  - عند استلام طلب من صيدلية (B2B).

### 5. B2B Commerce Tracking
- **Rep يرى طلبات الصيدليات** → `GET /api/commerce/supply-orders/company/{id}`.
- **يمكنه تحديث الحالة** (Confirm, Prepare, Ship, Deliver).

---

## 📊 جدول المراحل (ملخص)

| # | المرحلة | Endpoint | الحالة |
|---|---|---|---|
| 1 | Onboarding | (شاشات) | ✅ |
| 2 | Registration | `POST /api/auth/register` | ✅ |
| 3 | Company Membership | (Admin) | ✅ |
| 4 | Session Bootstrap | `POST /api/user/session-bootstrap` | ✅ |
| 5 | PharmaCompany Dashboard | `GET /api/rep/dashboard/stats` | ✅ |
| 6 | Rep Dashboard (Programs) | `GET /api/rep/programs` | ✅ |
| 7 | Invite Doctor/Pharmacy | `POST /api/rep/invitations/create` | ✅ |
| 8 | My Network | `GET /api/rep/my-doctors` + `my-pharmacies` | ✅ |
| 9 | My Invitations | `GET /api/rep/my-invitations` | ✅ |
| 10 | Resend Invitation | `POST /api/rep/invitations/{id}/resend` | ✅ |
| 11 | Track B2B Orders | `GET /api/commerce/supply-orders/company/{id}` | ✅ |

---

## 📂 الملفات المتأثرة (كاملة)

### Mobile (MAUI)
- `PharmaCompanyDashboardPage.xaml` + `.xaml.cs`
- `RepDashboardPage.xaml` + `.xaml.cs`
- `InviteDoctorPage.xaml` + `.xaml.cs`
- `InviteDoctorViewModel.cs`
- `MyNetworkPage.xaml` + `.xaml.cs`
- `MyInvitationsPage.xaml` + `.xaml.cs`
- `RepNetworkDetailsPage.xaml` + `.xaml.cs`
- `MembersManagementFlow.xaml` + `.xaml.cs`
- `JobTitlesManagementFlow.xaml` + `.xaml.cs`

### Shared.UI
- `PharmaCompanyDashboard.razor`
- `RepDashboard.razor`
- `InviteDoctor.razor`
- `MyNetwork.razor`
- `MyInvitations.razor`
- `RepNetworkDetails.razor`

### Backend (للمرجعية)
- `RepController.cs`
- `RepInvitationsController.cs`
- `OrganizationController.cs`
- `CommerceController.cs` (B2B)

---

## 🔗 روابط سريعة

- **[00-overview.md](00-overview.md)**
- **[01-patient.md](01-patient.md)**
- **[02-doctor.md](02-doctor.md)**
- **[03-pharmacist.md](03-pharmacist.md)**
- **[06-cross-cutting.md](06-cross-cutting.md)**
- **[23-COMMERCE-ENGINE.md](../23-COMMERCE-ENGINE.md)**

© 2026 RubikCare — للاستخدام الداخلي
```

---

