# 00 — OVERVIEW (نظرة عامة على رحلات المستخدمين)

الإصدار: 1.0
آخر تحديث: 15 نوفمبر 2026
الحالة: ✅ موثق
الجمهور: فريق التطوير + AI Assistants + فريق دعم المرضى + صانعو الفيديوهات

## 🎯 الغرض

هذه الوثيقة تقدم **خريطة شاملة** لرحلات المستخدمين الخمسة في RubikCare:

1. من هم المستخدمون؟
2. كيف يسجّلون؟
3. ما هي لوحات التحكم الرئيسية؟
4. كيف يتفاعلون مع بعضهم؟
5. ما هي الميزات المشتركة؟

## 👥 5 Personas

### 🟢 Persona 1: Patient (المريض)

**من هو؟**
- مستخدم مسجّل على المنصة.
- لديه `UserProfile` (شخص واحد) — وليس `PatientProfile`.
- قد يكون عضواً في برامج دعم (PSP) متعددة.

**كيف يسجل؟**
- من `OnboardingPage` → `RegisterPage` → `LoginView`.
- بعد التسجيل — يبدأ مباشرة في وضع **Personal**.

**Dashboards:**
- `DashboardFlowPage` (Personal Dashboard).

**رحلته الأساسية:**
1. تسجيل حساب.
2. استلام دعوة من طبيب/مندوب (`InvitationCode`).
3. الدخول إلى `/psp/entry` — إدخال الكود.
4. الانضمام لبرنامج PSP.
5. استلام توكن الصرف (Token).
6. إرسال التوكن للصيدلية.
7. تأكيد استلام الدواء.
8. كسب نقاط من الصرف + الالتزام اليومي.
9. استخدام النقاط لتجديد التوكن.

**رحلة مفصلة:** [01-patient.md](01-patient.md)

---

### 🔵 Persona 2: Doctor (الطبيب)

**من هو؟**
- مستخدم مسجّل + دور `Doctor` (في `AspNetRoles`).
- عضو في عيادة واحدة على الأقل (`OrgMembership` مع `OrganizationTypeID = 4`).

**كيف يسجل؟**
- `OnboardingPage` → `RegisterPage`.
- بعد التسجيل — يطلب التسجيل المهني (`/professional-status` → `/professional-license`).
- بعد الموافقة — يصبح عضواً في عيادة (عن طريق دعوة من مندوب أو إنشاء).

**Dashboards:**
- `ClinicDashboardPage` (لوحة العيادة).
- `DashboardFlowPage` (Personal — عند العودة للوضع الشخصي).

**رحلته الأساسية:**
1. تسجيل + التسجيل المهني.
2. الموافقة على الترخيص (من Admin).
3. الدخول إلى Clinic Dashboard.
4. البحث عن برامج PSP (`PspSearchPage`).
5. الاشتراك في برنامج.
6. دعوة مرضى (`InvitePatientPage`).
7. متابعة المرضى (`ClinicPatientsPage`).
8. عرض التحاليل + الوصفات.

**رحلة مفصلة:** [02-doctor.md](02-doctor.md)

---

### 🟡 Persona 3: Pharmacist (الصيدلي)

**من هو؟**
- مستخدم مسجّل + دور `Pharmacist`.
- عضو في صيدلية واحدة على الأقل (`OrgMembership` مع `OrganizationTypeID = 3`).

**كيف يسجل؟**
- مثل الطبيب — `OnboardingPage` → `RegisterPage` → التسجيل المهني.

**Dashboards:**
- `PharmacyDashboardPage` (لوحة الصيدلية).
- `DashboardFlowPage` (Personal).

**رحلته الأساسية:**
1. تسجيل + التسجيل المهني.
2. الموافقة على الترخيص.
3. الدخول إلى Pharmacy Dashboard.
4. استلام توكن من المريض.
5. التحقق من التوكن (`TokenRedemptionCard`).
6. تأكيد الصرف.
7. طلب أدوية من شركة الأدوية (`CommerceFlowPage`).
8. متابعة طلبات المرضى (`PharmacyOrdersPage`).

**رحلة مفصلة:** [03-pharmacist.md](03-pharmacist.md)

---

### 🟠 Persona 4: Rep (المندوب)

**من هو؟**
- مستخدم مسجّل + flag `IsRep = true`.
- عضو في شركة أدوية (`OrgMembership` مع `OrganizationTypeID = 2`).

**كيف يسجل؟**
- مثل الآخرين — لكن العضوية في شركة الأدوية تأتي من Admin الشركة (أو دعوة).

**Dashboards:**
- `PharmaCompanyDashboardPage` (لوحة شركة الأدوية).
- `RepDashboard` (لوحة المندوب — Programs).
- `DashboardFlowPage` (Personal).

**رحلته الأساسية:**
1. تسجيل + العضوية في شركة.
2. الدخول إلى PharmaCompany Dashboard.
3. عرض برامج الشركة (`RepDashboard`).
4. دعوة أطباء (`InviteDoctorPage`).
5. دعوة صيادلة (`InviteDoctorPage` — وضع Pharmacy).
6. متابعة الشبكة (`MyNetworkPage`).
7. عرض الدعوات (`MyInvitationsPage`).

**رحلة مفصلة:** [04-rep.md](04-rep.md)

---

### 🔵 Persona 5: Public User (المستخدم العام)

**من هو؟**
- مستخدم مسجّل على المنصة.
- **ليس** عضواً في أي مؤسسة.
- يستخدم المنصة للبحث والحجز.

**كيف يسجل؟**
- `OnboardingPage` → `RegisterPage`.
- مباشرة في وضع Personal.

**Dashboards:**
- `DashboardFlowPage` (Personal).

**رحلته الأساسية:**
1. تسجيل حساب.
2. البحث عن طبيب (`DoctorSearchPage`).
3. عرض تفاصيل الطبيب (`DoctorProfilePage`).
4. حجز موعد (إرسال رسالة للطبيب).
5. البحث عن صيدلية (`PharmacySearchPage`).
6. عرض تفاصيل الصيدلية (`PharmacyDetailPage`).
7. البحث عن صيدليات قريبة (`NearbyPharmaciesPage`).
8. المحادثات (`MessagingHubPage`).

**رحلة مفصلة:** [05-public-user.md](05-public-user.md)

---

## 🏢 المؤسسات في المنصة (Organizations)

| النوع | TypeID | مثال | يستخدمها |
|---|---|---|---|
| **Personal** | 1 | المستخدم الشخصي | جميع Personas |
| **PharmaCompany** | 2 | Devartlab Pharmaceuticals | Rep |
| **Pharmacy** | 3 | صيدلية روبيك كير | Pharmacist |
| **Clinic** | 4 | عيادة السلام | Doctor |
| **DrugWarehouse** | 5 | مخزن الأدوية | (مستقبلاً) |

**⚠️ ملاحظة:** المستخدم الواحد قد يكون عضواً في **عدة مؤسسات** (Clinic + Pharmacy + Company). يمكنه التبديل بينها من `AppShellViewModel.SwitchToProfessionalMode`.

---

## 🔐 الأدوار والصلاحيات

### الأدوار الحالية (AspNetRoles)
- **User** — افتراضي.
- **Doctor** — يفتح Clinic Dashboard.
- **Pharmacist** — يفتح Pharmacy Dashboard.
- **Rep** — يفتح PharmaCompany Dashboard (عبر `IsRep`).
- **Admin** — يفتح Admin Dashboard.

### الصلاحيات (مستقبلاً — `19-PERMISSIONS.md`)
- **4 طبقات:** Platform → Subscription → Organization → JobTitle.
- **⚠️ TODO:** UI لإدارة الصلاحيات — جلسة منفصلة.

---

## 🔄 التفاعلات بين Personas

```
[Rep] ─── يدعو ───► [Doctor] ─── يدعو ───► [Patient]
  │                    │                       │
  │                    │                       │
  └── يدعو ──────────► [Pharmacist] ◄── يرسل ──┘
                              │
                              │
                              ▼
                        [يصرف الدواء]
                              │
                              │
                              ▼
                       [Patient يُكمل]
```

**التدفق:**
1. **Rep** يدعو **Doctor** للانضمام إلى برنامج PSP.
2. **Rep** يدعو **Pharmacist** للانضمام إلى برنامج PSP.
3. **Doctor** يدعو **Patient** للانضمام إلى برنامج PSP.
4. **Patient** يستلم توكن — يرسله لـ **Pharmacist**.
5. **Pharmacist** يصرف الدواء — يؤكد.
6. **Patient** يؤكد الاستلام — يكسب نقاط.

---

## 🎯 الميزات المشتركة (Cross-Cutting)

كل Persona تشترك في:

### 1. AppShell (Navigation)
- **Flyout Menu** — ديناميكي حسب الدور.
- **Banner "غير متصل"** — في `FlyoutContent` + `TitleView`.
- **زر Back مركزي** — في `AppShell.OnBackButtonPressed`.

### 2. Translation (AR/EN) + RTL
- **اللغة الافتراضية:** العربية.
- **التخزين:** `Preferences` (Mobile) / `localStorage` (Web/PWA).
- **Offline:** 4 مستويات Fallback.

### 3. Offline Mode
- **ما يعمل offline:**
  - الترجمات.
  - جلسة المستخدم.
  - القائمة الجانبية.
- **ما لا يعمل offline:**
  - الإحصائيات (حتى يُضاف Cache).
  - طلبات جديدة.
  - المحادثات الحية.

### 4. Notifications
- **Push (FCM)** — عبر Firebase.
- **Local** — للتذكيرات.
- **In-App** — عبر `NotificationBell`.

### 5. Messaging
- **محادثات بين Patient و Organization**.
- **محادثات بين Patient و Doctor**.
- **محادثات B2B** (Rep ↔ Pharmacy).

### 6. Commerce (B2B)
- **Pharmacist يطلب من PharmaCompany**.
- **Rep يتابع الطلبات**.

### 7. Start Page Preference (مستقبلاً)
- **⚠️ TODO:** المستخدم يختار صفحة البدء (Personal / Clinic / Pharmacy / Company).

**التفاصيل:** [06-cross-cutting.md](06-cross-cutting.md)

---

## 📋 جدول المراحل (ملخص شامل)

| # | المرحلة | Persona | Endpoint | الحالة |
|---|---|---|---|---|
| 1 | Onboarding | كل الـPersonas | (شاشات فقط) | ✅ |
| 2 | Registration | كل الـPersonas | `POST /api/auth/register` | ✅ |
| 3 | Login | كل الـPersonas | `POST /api/auth/login` | ✅ |
| 4 | Session Bootstrap | كل الـPersonas | `POST /api/user/session-bootstrap` | ✅ |
| 5 | Professional Registration | Doctor + Pharmacist | `POST /api/professional/register` | ✅ |
| 6 | Rep Invites Doctor | Rep → Doctor | `POST /api/rep/invitations/create` | ✅ |
| 7 | Doctor Invites Patient | Doctor → Patient | `POST /api/psp/create-invitation` | ✅ |
| 8 | Patient Enrolls in PSP | Patient | `POST /api/psp/accept-invitation` | ✅ |
| 9 | Patient Receives Token | Patient | (تلقائي) | ✅ |
| 10 | Patient Sends Token | Patient → Pharmacist | `POST /api/dispense/validate-token` | ✅ |
| 11 | Pharmacist Dispenses | Pharmacist | `POST /api/dispense/confirm` | ✅ |
| 12 | Patient Confirms Receipt | Patient | `POST /api/dispense/confirm-receipt` | ✅ |
| 13 | Patient Earns Points | (تلقائي) | `IEarnPointsOrchestrator` | ✅ |
| 14 | Patient Uses Points | Patient | `POST /api/dispense/apply-points` | ✅ |
| 15 | Daily Adherence | Patient | `PUT /api/patient/medications/status` | ✅ |
| 16 | Doctor Searches Program | Doctor | `GET /api/psp/programs` | ✅ |
| 17 | Doctor Subscribes to Program | Doctor | `POST /api/psp/participate` | ✅ |
| 18 | Public User Searches Doctor | Public User | `GET /api/organizations/doctors/search` | ✅ |
| 19 | Public User Books Appointment | Public User | `POST /api/messaging/conversations` + `POST /api/messaging/messages` | ✅ |
| 20 | Pharmacist Orders from Company | Pharmacist → Rep | `POST /api/commerce/supply-orders` | ✅ |
| 21 | Rep Tracks Orders | Rep | `GET /api/commerce/supply-orders/company/{id}` | ✅ |

---

## 🎯 كيف تستخدم هذه الوثيقة؟

### للمطورين:
1. ابدأ بـ **README.md** — لفهم الهيكل.
2. ثم **00-overview.md** (هذه) — لفهم الشخصيات.
3. ثم الوثيقة الخاصة بالشخصية التي تعمل عليها.

### لـ AI Support Bot:
- **الوثائق كاملة** — تُغذي الـ Bot.
- **الـ Endpoints** — تُستخدم للإجابة على "كيف أفعل X؟".
- **المراحل** — تُستخدم لتتبع تقدم المستخدم.

### لصانعي الفيديوهات:
- كل وثيقة = سكريبت فيديو محتمل.
- **العنوان** + **الخطوات** + **الصور المقترحة**.

---

## 📊 إحصائيات

| المقياس | القيمة |
|---|---|
| Personas | 5 |
| Organization Types | 5 (Personal, Pharma, Pharmacy, Clinic, Warehouse) |
| Endpoints (إجمالي) | ~290 |
| Endpoints موثقة هنا | 21 |
| Cross-Cutting Concerns | 7 |

---

## 🔗 روابط سريعة

- **Patient:** [01-patient.md](01-patient.md)
- **Doctor:** [02-doctor.md](02-doctor.md)
- **Pharmacist:** [03-pharmacist.md](03-pharmacist.md)
- **Rep:** [04-rep.md](04-rep.md)
- **Public User:** [05-public-user.md](05-public-user.md)
- **Cross-Cutting:** [06-cross-cutting.md](06-cross-cutting.md)

© 2026 RubikCare — للاستخدام الداخلي
```
