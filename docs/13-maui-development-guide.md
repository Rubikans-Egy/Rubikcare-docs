
# 05 — PSP (نظام برامج دعم المرضى)

الإصدار: 2.1 (Unified Edition — Gap-Aware) 
آخر تحديث: 24 نوفمبر 2026 
الحالة: ✅ مرجع شامل — يدمج 07 + User Flow + الكود الفعلي + الكيانات المنفّذة 
الجمهور: فريق التطوير + AI Assistants

---

## 📌 الجزء صفر: نظرة عامة

### 0.1 ما هو نظام PSP؟

PSP = Patient Support Programs = برامج دعم المرضى.

الهدف: تقديم دعم (مالي، طبي، تعليمي) للمرضى من خلال شركات الأدوية، مع تتبع كامل للرحلة العلاجية.

### 0.2 المكونات الأساسية

| # | المكوّن | الوصف | الحالة |
|---|---|---|---|
| 1 | Program | البرنامج (علاج السكري، علاج الأنيميا...) | ✅ |
| 2 | ProgramMedication | الأدوية المرتبطة بالبرنامج | ✅ |
| 3 | DispensationPlan | خطة الصرف | ✅ |
| 4 | DispensationPlanPhase | ⭐ مراحل خطة الصرف | ✅ |
| 5 | Dispensation | عمليات الصرف الفعلية | ✅ |
| 6 | Patient | المرضى المشاركين (CaseID) | ✅ |
| 7 | Invitation | الدعوات (للمنظمات) | ✅ |
| 8 | Participation | مشاركات المنظمات (عيادة/صيدلية) | ✅ |
| 9 | eRX | الروشتات الإلكترونية | ✅ |
| 10 | eRXItem | ⭐ تفاصيل الأدوية في روشتة الدعم | ✅ **جديد** |
| 11 | Prescription | ⭐ الوصفة العامة (Clinic Domain) | ✅ **جديد** |
| 12 | PrescriptionItem | ⭐ تفاصيل الوصفة العامة | ✅ **جديد** |
| 13 | ClinicProgramReferral | ⭐ كود إحالة ثابت (عيادة + برنامج) | ✅ **جديد** |
| 14 | PSPRequiredTest | التحاليل المطلوبة | ✅ |
| 15 | PSPTestResult | نتائج التحاليل | ✅ |
| 16 | PSPRequiredDataEntry | ⭐ البيانات المطلوبة (تعريف) | ✅ |
| 17 | PSPPatientDataEntry | ⭐ إجابات المريض على البيانات المطلوبة | ⚠️ **يحتاج تأكيد** |
| 18 | DoctorNote | ملاحظات الطبيب | ✅ (كيان) — ❌ (UI) |

---

## 🗄️ الجزء الأول: هيكل الكيانات (Domain Entities)

### 1.1 المسار

```
RubikCare.Domain\Entities\PSP\
├── Config\          ← إعدادات
│   ├── PSPRequiredDataEntry.cs
│   ├── PSPRequiredFollowUp.cs
│   └── PSPRequiredTest.cs
│
├── Core\            ← القلب
│   ├── ClinicProgramReferral.cs                  ⭐ جديد
│   ├── LinkTargetAudience.cs
│   ├── PSPDispensation.cs
│   ├── PSPDispensationPlan.cs
│   ├── PSPDispensationPlanMedication.cs
│   ├── PSPDispensationPlanPhase.cs               ⭐
│   ├── PSPDispensationPlanPhaseMedication.cs     ⭐
│   ├── PSPeRX.cs
│   ├── PSPeRXItem.cs                             ⭐ جديد
│   ├── PSPInvitation.cs
│   ├── PSPParticipation.cs
│   ├── PSPPatient.cs
│   ├── PSPProgram.cs
│   ├── PSPProgramLink.cs
│   ├── PSPProgramMedication.cs
│   └── PSPProgramSpeciality.cs
│
├── Doctor\
│   └── DoctorNote.cs
│
├── Execution\
│   ├── PSPFollowUpRecord.cs
│   └── PSPTestResult.cs
│
└── Patient\
    └── PSPAdverseEvent.cs
```

**ملاحظة**: `Prescription.cs` و`PrescriptionItem.cs` يقعان في `RubikCare.Domain\Entities\Clinic\` (Clinic Domain) — انظر §v2.

### 1.2 خريطة العلاقات (محدّثة)

```
PSPProgram (البرنامج)
    │
    ├── ProgramID ──┬──> PSPProgramMedications (أدوية البرنامج)
    │               │         │
    │               │         └── ProgramMedicationID ──┐
    │               │                                   │
    │               └──> PSPDispensationPlans (خطط الصرف)
    │                         │
    │                         └── PlanID ──┐
    │                                      │
    │                                      ▼
    │                          PSPDispensationPlanMedications
    │                          (Legacy — دواء في الخطة مباشرة)
    │
    ├── ProgramID ──> PSPRequiredTests / PSPRequiredFollowUps / PSPRequiredDataEntries
    │
    └── ProgramID ──> ClinicProgramReferrals ⭐ (كود إحالة ثابت لكل عيادة)
```

### 1.3 ⭐ مع Phase System

```
PSPProgram
  └── PSPDispensationPlan
        ├── PSPDispensationPlanMedication (Legacy — للتوافق)
        └── PSPDispensationPlanPhase ⭐
              └── PSPDispensationPlanPhaseMedication ⭐
                    └── PSPProgramMedication
```

### 1.4 ⭐ مع eRX System (محدّث)

```
UserProfile (شخص واحد — PatientProfileID)
    │
    └── PSPPatient (سجل واحد — CaseID) ⭐ UNIQUE على PatientProfileID
            │
            ├── PSPParticipation (مشاركة المنظمة: عيادة/صيدلية)  ⚠️ ليس اشتراك المريض
            │
            ├── PSPeRX (روشتة دعم — ProgramID إلزامي)
            │     └── PSPeRXItem ⭐ (MedicationID nullable)
            │           ├── ProgramMedicationID (FK إلزامي)
            │           ├── OverrideTimesPerDay?
            │           ├── OverrideQuantityPerDispense?
            │           └── IsOverridden
            │
            ├── Prescription (وصفة عامة — Clinic Domain)
            │     └── PrescriptionItem
            │           └── LinkedPspERxItemID? ──> PSPeRXItem
            │
            └── PSPDispensations (التوكنات)
```

### 1.5 ⭐ إصلاح الجذور المعمارية: `CaseID` (وليس `PatientProfileID`)

#### 1.5.1 المشكلة الجذرية (Root Cause) — جلسة #136

**قبل الإصلاح:**
- كان `PSPPatient` يُنشأ **جديداً** لكل مشاركة في برنامج جديد.
- أدى إلى وجود **عدة `PatientID` لنفس الشخص** (`UserProfileID`).
- مثال حقيقي: `shadyelzaher@rubikcare.com` كان لديه `PatientID = 18` (Infertility) و`PatientID = 1081` (pregnant case).
- **النتيجة:** فوضى في نظام النقاط (النقاط تذهب لـ `PatientID` عشوائي)، والتقارير، والمواعيد.

#### 1.5.2 الإصلاح الجذري (Root Fix)

**البنية الجديدة:**
```
UserProfile (شخص واحد)
└── PSPPatient (سجل واحد — CaseID)
├── PSPParticipation (البرنامج 1)
├── PSPParticipation (البرنامج 2)
└── PSPParticipation (البرنامج 3)
```

**التغييرات الرئيسية:**

| # | التغيير | الملف | التفاصيل |
|---|---|---|---|
| 1 | `PatientID` → `CaseID` | كل الكيانات (`PSPPatient`, `PSPeRX`, `PSPReminder`, `PSPTestResult`, `PSPFollowUpRecord`, `PSPAdverseEvent`, `DoctorNote`) | إعادة تسمية عبر Migration (RenameColumn) |
| 2 | Migration `RenamePatientIdToCaseId` | `RubikCare.Infrastructure/Migrations/` | RenameColumn + تحديث الـ FKs |
| 3 | `UNIQUE INDEX` على `PatientProfileID` | `PSPPatients` | منع تكرار المريض نهائياً |
| 4 | `PSPEnrollmentService` | `RubikCare.Application/Services/PSP/` | البحث عن `PSPPatient` بـ `PatientProfileID` (وليس إنشاء جديد) |
| 5 | `PspController.GetUserActivePrograms` | `Api.Web/Controllers/PSP/PspController.Helpers.cs` | إرجاع كل البرامج حتى لو لم يكن لها `eRx` أو `Dispensation` |
| 6 | `TryAwardDailyDosePointsAsync` | `PatientMedicationService.cs` | البحث عن `CaseID` الصحيح بناءً على `ProgramID` |

#### 1.5.3 القواعد الصارمة (Rules) — لا تكسرها أبداً

| # | القاعدة | السبب |
|---|---|---|
| **R1** | **لا تنشئ `PSPPatient` جديداً إذا كان `PatientProfileID` موجوداً.** | هذا هو أصل الفوضى. استخدم `Participations` للتعدد. |
| **R2** | **استخدم `PatientProfileID` (وليس `CaseID`) للبحث عن "الشخص".** | `CaseID` يمثل "حالة"، وليس "شخصاً". |
| **R3** | **في `PointsContext`، استخدم `CaseID` كـ `BeneficiaryID` (لأن `EarnPointsHandler` يعتمد عليه).** | هذا هو التصميم الحالي. |
| **R4** | **`PSPPatient.ParticipationID` يشير إلى "أول مشاركة" فقط (موروث).** | لا تعتمد عليه للبحث عن "كل المشاركات". |
| **R5** | **في `GetUserActivePrograms`، لا تتخطى البرنامج إذا لم يجد `eRx` أو `Dispensation`.** | البرنامج موجود حتى لو لم يُصرف بعد. |
| **R6** | **`PSPParticipation` = مشاركة منظمة (عيادة/صيدلية)، وليست اشتراك مريض.** | ⭐ توضيح جديد — لا تخلط بينهما. |
| **R7** | **`ClinicProgramReferral` ≠ `PSPInvitation`.** | ⭐ `Referral` كود ثابت متعدد الاستخدام، `Invitation` دعوة فردية لمرة واحدة. |

#### 1.5.4 كيف يعمل نظام النقاط الآن (بعد الإصلاح)

```
تأكيد التناول (ScheduleID من البرنامج 1)
↓
UpdateScheduleStatusAsync
↓
TryAwardDailyDosePointsAsync
├── جلب PSPPatient (CaseID=1083, ProgramID=1)
├── بناء PointsContext (CaseID=15, OrganizationID=1267)
└── AwardByCampaignAsync(campaignId=1, ...)
↓
Scheme 1 (PSP.CUP)
↓
Rule (ActivityTypeID=8, Points=5)
↓
EarnPointsHandler
├── Balance(BeneficiaryID=15, SchemeID=1)
└── PSPProgramPointTransaction (Points=+5)
```

#### 1.5.5 سيناريوهات اختبار القبول (Acceptance Tests)

| # | السيناريو | النتيجة المتوقعة |
|---|---|---|
| 1 | المريض مشترك في برنامجين | `PSPPatients` يحتوي سجلاً واحداً فقط (`CaseID` واحد) |
| 2 | فتح PspEntry | يظهر **كل** البرامج النشطة (حتى لو لم تُصرف) |
| 3 | تأكيد التناول في البرنامج 1 | النقاط تزيد في `Scheme 1` (وليس `Scheme 7`) |
| 4 | تأكيد التناول في البرنامج 2 | النقاط تزيد في `Scheme 7` (وليس `Scheme 1`) |
| 5 | صرف الدواء | النقاط تزيد بناءً على `ActivityTypeID` للصرف |
| 6 | استخدام النقاط (ApplyPoints) | `Balance` ينخفض، `PatientPayment` ينخفض |

#### 1.5.6 الملفات المتأثرة (للمراجعة المستقبلية)

| الطبقة | الملفات |
|---|---|
| **Domain** | `PSPPatient.cs`, `PSPeRX.cs`, `PSPReminder.cs`, `PSPTestResult.cs`, `PSPFollowUpRecord.cs`, `PSPAdverseEvent.cs`, `DoctorNote.cs`, `PointsContext.cs` |
| **Infrastructure** | `BusinessDbContext.cs` (ConfigurePSPModels), `EarnPointsHandler.cs` |
| **Application** | `PSPEnrollmentService.cs`, `PatientMedicationService.cs` |
| **API** | `PspController.Helpers.cs`, `PspController.patientdetails.cs`, `DispenseController.cs`, `PatientMedicationsController.cs` |
| **Migrations** | `20261021XXXXXX_RenamePatientIdToCaseId.cs` |
| **Shared.UI** | `PspDetailComponent.razor`, `MedicationSchedulePage.razor`, `PspEntry.razor` |

---

## 🎯 الجزء الثاني: نظام مراحل خطة الصرف (Phase System)

### 2.1 الهدف

كل خطة صرف ممكن تحتوي على مراحل متعددة — كل مرحلة لها:
- مدة زمنية (3 شهور، 6 شهور...).
- تبدأ بعد كام شهر من بداية البرنامج.
- أدوية خاصة بها.
- تفاصيل كاملة: كام علبة/شهر، التقسيمة، الخصم.

### 2.2 السيناريو

```
برنامج: علاج السكري
  └── خطة الصرف: "الخطة الأساسية"
        ├── المرحلة الأولى (0-3 شهور)
        │     ├── Metformin — 2 علبة/شهر — 3 مرات يومياً
        │     └── Insulin — 1 علبة/شهر — 2 مرات يومياً
        └── المرحلة الثانية (3-6 شهور)
              ├── Metformin — 3 علبة/شهر — 4 مرات يومياً
              └── Glimepiride — 1 علبة/شهر — 1 مرة يومياً
```

### 2.3 الكيانات

#### 2.3.1 `PSPDispensationPlanPhase`

المسار: `RubikCare.Domain\Entities\PSP\Core\PSPDispensationPlanPhase.cs`

```csharp
[Table("PSPDispensationPlanPhases")]
public class PSPDispensationPlanPhase
{
    [Key]
    public int PhaseID { get; set; }

    public int PlanID { get; set; }  // FK → PSPDispensationPlans (CASCADE)

    public string PhaseName { get; set; }   // "المرحلة الأولى"
    public int PhaseOrder { get; set; }     // 1, 2, 3...
    public int DurationMonths { get; set; } // 3, 6...
    public int StartAfterMonths { get; set; } // 0 = من بداية البرنامج
    public string? Description { get; set; }

    public bool IsActive { get; set; } = true;
    public DateTime CreatedDate { get; set; }
    public DateTime? LastModifiedDate { get; set; }

    // Navigation
    public virtual PSPDispensationPlan? Plan { get; set; }
    public virtual ICollection<PSPDispensationPlanPhaseMedication> PhaseMedications { get; set; }
}
```

#### 2.3.2 `PSPDispensationPlanPhaseMedication`

المسار: `RubikCare.Domain\Entities\PSP\Core\PSPDispensationPlanPhaseMedication.cs`

```csharp
[Table("PSPDispensationPlanPhaseMedications")]
public class PSPDispensationPlanPhaseMedication
{
    [Key]
    public int PhaseMedicationID { get; set; }

    public int PhaseID { get; set; }              // FK → PSPDispensationPlanPhases (CASCADE)
    public int ProgramMedicationID { get; set; }  // FK → PSPProgramMedications (NO_ACTION)

    public int QuantityPerMonth { get; set; }     // كام علبة/شهر
    public int? TimesPerDay { get; set; }         // 3 = ثلاث مرات يومياً
    public int? HoursBetweenDoses { get; set; }   // 8 = كل 8 ساعات
    public decimal DiscountPercentage { get; set; } // نسبة الخصم

    public bool IsActive { get; set; } = true;
    public DateTime CreatedDate { get; set; }
    public DateTime? LastModifiedDate { get; set; }

    // Navigation
    public virtual PSPDispensationPlanPhase? Phase { get; set; }
    public virtual PSPProgramMedication? ProgramMedication { get; set; }
}
```

### 2.4 ⚠️ قواعد الـ Foreign Keys

| العلاقة | OnDelete | السبب |
|---|---|---|
| Phase.PlanID → Plan | CASCADE | لو الخطة اتمسحت، المراحل تتمسح |
| PhaseMedication.PhaseID → Phase | CASCADE | لو المرحلة اتمسحت، الأدوية تتمسح |
| PhaseMedication.ProgramMedicationID → ProgramMedication | NO_ACTION | يمنع مسح الدواء لو فيه مراحل بتشير ليه |

**ProgramMedication هو بيانات مرجعية (Master Data)، مش بيانات تشغيلية.**

### 2.5 الـ Migration

ID: `20260923150028_AddPSPDispensationPlanPhases`

الجداول الجديدة:
- `PSPDispensationPlanPhases`.
- `PSPDispensationPlanPhaseMedications`.

الأعمدة الجديدة:
- `PSPDispensationPlanMedications.PhaseID` (nullable) — للتوافق.

---

## 🎯 الجزء الثالث: نظام التحاليل (Tests System)

⚠️ هذا الجزء مُستخلَص من `PspController.Tests.cs` — 38 endpoint.

### 3.1 نظرة عامة

نظام التحاليل = 8 مجموعات:

| # | المجموعة | الوصف |
|---|---|---|
| 1 | LabTest (مرجعية) | قائمة التحاليل المتاحة (Master Data) |
| 2 | RequiredTest | التحاليل المطلوبة في برنامج |
| 3 | TestResult | نتائج التحاليل |
| 4 | Attachment | المرفقات (PDF, صور) |
| 5 | PharmaView | عرض الشركة (Privacy) |
| 6 | Reminder | التذكيرات |
| 7 | PhaseRequiredTest | ربط بمراحل |
| 8 | Patient Mobile | للموبايل |

### 3.2 ⚠️ منطق RequiredTiming (5 قيم)

**تحديث**: `PSPRequiredDataEntry` يحتوي على `RequiredTiming` بـ **5 قيم** (بما فيها `OnEvent`)، وليس 4 كما كان موثقاً سابقاً.

| القيمة | المعنى | المنطق |
|---|---|---|
| 1 | Before (قبل البدء) | يجب رفعها قبل StartDate |
| 2 | During (أثناء) | متاحة بعد StartDate |
| 3 | After (بعد N أيام) | بعد StartDate + RequiredAtDay |
| 4 | Periodic (دورية) | كل FrequencyDays |
| 5 | OnEvent (عند حدث) | ⭐ جديد — مرتبطة بـ `TriggerEvent` (Dispensation/AdverseEvent/FollowUp) |

- Before — يمنع بدء الرحلة العلاجية حتى رفعها.
- During — متاحة بعد البدء.
- After — مؤجلة N أيام بعد البدء.
- Periodic — متكررة.
- OnEvent — تُطلب عند وقوع حدث محدد.

### 3.3 ⚠️ منطق EditWindow

| نوع التحليل | مدة التعديل |
|---|---|
| Mandatory (إلزامي) | 24 ساعة |
| غير Mandatory | 168 ساعة (7 أيام) |

- `ComputeEditWindow(dto, lastResult, isMandatory)`.
- `CanEdit = elapsed.TotalHours <= editWindowHours`.

### 3.4 ⚠️ نظام Consent (موافقة المريض)

```csharp
ConsentGiven = true,
ConsentDate = DateTime.UtcNow,
ConsentText = dto.ConsentText ?? "أوافق على مشاركة هذا التحليل..."
```

انظر `27-REPORTS` (Privacy + PDPL).

### 3.5 ⚠️ Privacy في PharmaView

```csharp
PatientAnonymousId = $"PAT-{r.PatientID:D6}",
PatientAge = CalculateAge(...),
PatientGender = ...,
// ❌ لا اسم، لا هاتف
```

انظر `07-REPORTS` (Adherence Report).

### 3.6 ⚠️ 3 خيارات لـ ShowTo*

- `ShowToPatient`
- `ShowToDoctor`
- `ShowToPharma`

### 3.7 ⚠️ 4 أنواع مرفقات

| النوع | Endpoint |
|---|---|
| Form File | `POST attachments` (multipart) |
| Base64 | `POST attachments-base64` (Mobile) |
| Download | `GET attachments/{id}/download` |
| View Base64 | `GET attachments/{id}/view-base64` (Modal) |

`view-base64` — محمي بـ Auth + 5MB limit.

### 3.8 4 حالات للـ Test Status

| الحالة | متى |
|---|---|
| Completed | توجد نتيجة |
| Active | متاح للرفع |
| Locked | لم يأن الوقت |
| Pending | (مذكور في كود الحالة) |

### 3.9 StartTreatment — منطق ذكي

```
1. التحقق: StartDate == null؟
2. جلب mandatory قبل البدء
3. التحقق: كلها مرفوعة؟
4. لو ناقص → BadRequest + أسماء التحاليل الناقصة
5. لو كامل → StartDate = UtcNow
```

---

## 🎯 الجزء الرابع: تدفق المستخدمين (User Flow)

⚠️ هذا الجزء مُستخلَص من `USERS STEP FLOW`.

### 4.1 تدفق المندوب (Rep)

الدور: المستخدم الأول في المنصة، يمثل شركة الأدوية.

| الخطوة | الوصف | المسؤول | الصفحة / API |
|---|---|---|---|
| 1 | إنشاء حساب (تسجيل) | المندوب | `RegisterPage` |
| 2 | تسجيل الدخول | المندوب | `LoginPage` |
| 3 | الوصول إلى لوحة التحكم | المندوب | `PharmaCompanyDashboardPage` |
| 4 | إنشاء دعوة (لطبيب أو صيدلي) | المندوب | `POST api/rep/invitations/create` |
| 5 | مشاركة كود الدعوة مع الطبيب/الصيدلي | المندوب | مشاركة يدوية (واتساب / بريد) |

🔚 نهاية تدفق المندوب

### 4.2 تدفق الطبيب (Doctor)

الدور: يستقبل دعوة من المندوب للاشتراك في برنامج دعم المريض (PSP).

| الخطوة | الوصف | المسؤول | الصفحة / API |
|---|---|---|---|
| 1 | استلام كود الدعوة من المندوب | الطبيب | يدويًا |
| 2 | تحميل تطبيق RubikCare | الطبيب | — |
| 3 | إنشاء حساب (تسجيل) | الطبيب | `RegisterPage` |
| 4 | تسجيل الدخول | الطبيب | `LoginPage` |
| 5 | اختيار الدور المهني "طبيب" | الطبيب | `ProfessionalStatusFlow` |
| 6 | إدخال ترخيص العيادة + رفع مستندات | الطبيب | `ProfessionalLicenseFlow` |
| 7 | انتظار موافقة المنصة | النظام | `ProfessionalLicenseFlow` (Pending) |
| 8 | إنشاء عيادة (بعد الموافقة) | الطبيب | `create-organization` (BlazorWebView) |
| 9 | البحث عن برنامج PSP باستخدام كود الدعوة | الطبيب | `PspEntryPage` |
| 10 | الاشتراك في البرنامج | النظام | `POST api/psp/entry` |
| 11 | الوصول إلى لوحة الطبيب | الطبيب | `ClinicDashboardPage` |
| 12 | (لاحقًا) إنشاء دعوات للمرضى | الطبيب | `POST api/psp/create-invitation` |
| 13 | ⭐ **توليد كود إحالة ثابت (QR + ShortCode)** | الطبيب | `ClinicProgramReferral` **جديد** |

🔚 نهاية تدفق الطبيب

### 4.3 تدفق الصيدلي (Pharmacist)

الدور: يستقبل دعوة من المندوب، ومسؤول عن تأكيد صرف الأدوية للمرضى.

| الخطوة | الوصف | المسؤول | الصفحة / API |
|---|---|---|---|
| 1 | استلام كود الدعوة من المندوب | الصيدلي | يدويًا |
| 2 | تحميل تطبيق RubikCare | الصيدلي | — |
| 3 | إنشاء حساب (تسجيل) | الصيدلي | `RegisterPage` |
| 4 | تسجيل الدخول | الصيدلي | `LoginPage` |
| 5 | اختيار الدور المهني "صيدلي" | الصيدلي | `ProfessionalStatusFlow` |
| 6 | إدخال ترخيص الصيدلية + رفع مستندات | الصيدلي | `ProfessionalLicenseFlow` |
| 7 | انتظار موافقة المنصة | النظام | `ProfessionalLicenseFlow` (Pending) |
| 8 | إنشاء صيدلية (بعد الموافقة) | الصيدلي | `create-organization` (BlazorWebView) |
| 9 | البحث عن برنامج PSP باستخدام كود الدعوة | الصيدلي | `PspEntryPage` |
| 10 | الاشتراك في البرنامج | النظام | `POST api/psp/entry` |
| 11 | الوصول إلى لوحة الصيدلي | الصيدلي | `PharmacyDashboardPage` |
| 12 | تأكيد صرف الأدوية للمرضى | الصيدلي | `POST api/dispense/validate-token + confirm` |

🔚 نهاية تدفق الصيدلي

### 4.4 تدفق المريض (Patient)

الدور: يستقبل دعوة من الطبيب (أو كود إحالة ثابت)، ويحصل على الدواء من الصيدلية.

| الخطوة | الوصف | المسؤول | الصفحة / API |
|---|---|---|---|
| 1 | استلام كود الدعوة من الطبيب (أو كود الإحالة) | المريض | يدويًا |
| 2 | تحميل تطبيق RubikCare | المريض | — |
| 3 | إنشاء حساب (تسجيل) | المريض | `RegisterPage` |
| 4 | تسجيل الدخول | المريض | `LoginPage` |
| 5 | إدخال كود الدعوة/الإحالة للبحث عن برنامج PSP | المريض | `PspEntryPage` / `JoinPage` |
| 6 | الاشتراك في البرنامج | النظام | `POST api/psp/entry` |
| 7 | ⭐ **تسجيل البيانات المطلوبة (PSPRequiredDataEntry)** | المريض | `PatientRequiredDataEntryPage` **جديد** |
| 8 | استلام رمز الصرف (TokenCode) | النظام | Response من `api/psp/entry` |
| 9 | مشاركة رمز الصرف مع الصيدلي | المريض | يدويًا |
| 10 | فتح صفحة التذكيرات (بعد تأكيد الصرف) | المريض | `MedicationScheduleFlow` |
| 11 | تعيين موعد الجرعة الأولى | المريض | `MedicationScheduleFlow` |
| 12 | توليد مواعيد الصرف التالية تلقائيًا | النظام | تلقائيًا |
| 13 | ⭐ **استلام الوصفة (eRX) من الطبيب** | المريض | `PrescriptionViewPage` **جديد** |

🔚 نهاية تدفق المريض

### 4.5 حالات الدعوة (Invitation Status)

| الحالة | المعنى | وقت الحدوث |
|---|---|---|
| PENDING | دعوة مرسلة ولم تُستخدم بعد | بعد إنشاء الدعوة مباشرة |
| ACCEPTED | تم قبول الدعوة (قبل إكمال التسجيل) | عند استخدام الكود في `PspEntryPage` |
| USED | تم استخدام الدعوة بالكامل | بعد اكتمال الاشتراك في البرنامج |
| EXPIRED | انتهت صلاحية الدعوة | بعد 30 يوم من الإنشاء |

### 4.6 قاعدة البيانات — تحديث العلاقات

عند قبول الدعوة (الخطوة 10 للطبيب/الصيدلي)، يتم تحديث:

```sql
-- تحديث الدعوة
UPDATE PSPInvitations 
SET Status = 'USED',
    UsedByUserID = @UserProfileId,
    UsedDate = GETDATE()
WHERE InvitationToken = @InvitationCode;

-- تحديث المشاركة بربطها بالدعوة
UPDATE PSPParticipations 
SET InvitationID = @InvitationId
WHERE ParticipationID = @ParticipationId;
```

**ملاحظة**: `PSPInvitation` يحتوي على `[InverseProperty("Invitation")] ResultingParticipation` — العلاقة صحيحة في الكيان، لكن الـ SQL أعلاه لا يستخدمها. يُفضّل الاعتماد على EF Core Navigation بدلاً من SQL المباشر.

### 4.7 المهام الفنية المطلوبة

| المهمة | الوصف | الحالة |
|---|---|---|
| تحديث `POST api/psp/entry` | ربط الدعوة بالمشاركة تلقائيًا | ❌ غير مكتمل |
| تحديث `POST api/rep/invitations/create` | توليد كود دعوة صالح | ✅ مكتمل |
| إنشاء صفحة `MyNetworkPage` | شبكة المندوب | ✅ مكتمل |
| إنشاء صفحة `MyInvitationsPage` | سجل الدعوات | ❌ لم تبدأ |
| تعديل `GetMyDoctors` و`GetMyPharmacies` | قراءة الدعوات ذات الحالة USED | ❌ تحتاج تعديل |
| ⭐ **إنشاء صفحة `WritePrescriptionPage`** | كتابة eRX متعدد الأدوية | ❌ لم تبدأ |
| ⭐ **إنشاء صفحة `PatientRequiredDataEntryPage`** | إدخال البيانات المطلوبة | ❌ لم تبدأ |
| ⭐ **إنشاء `DoctorNotesTab`** | ملاحظات الطبيب | ❌ لم تبدأ |

---

# [#61] 📘05-PSP — v2.0 (Part 2)

## 📌 الجزء الخامس: نظام النقاط في PSP (Points Integration)

### 5.1 نظرة عامة

انظر `06-POINTS` (المُدمج من 26).

نظام النقاط في PSP = 5 أحداث منفذة + 2 مؤجلة (من أصل 7):

| # | الحدث | الموقع | الحالة |
|---|---|---|---|
| 1 | PATIENT_INVITED | `PSPEnrollmentService` | ✅ |
| 2 | PATIENT_ENROLLED | `PSPEnrollmentService` | ✅ |
| 3 | ERX_CREATED | `PSPEnrollmentService` | ✅ |
| 4 | DISPENSATION_COMPLETED | `DispenseController` | ✅ |
| 5 | REFILL_COMPLETED | `DispenseController` | ✅ |
| 6 | PROGRAM_COMPLETED | مؤجل | ⏸️ |
| 7 | LAB_TEST_UPLOADED | مؤجل | ⏸️ |

### 5.2 كيف تُمنح النقاط؟

```
1. حدث (مثل: PATIENT_ENROLLED)
   ↓
2. PSPEnrollmentService يستدعي:
   TryAwardPointsViaOrchestratorAsync
   ↓
3. بناء PointsContext:
   - OrganizationID
   - CaseID (وليس PatientProfileID — انظر R3)
   - UserID
   ↓
4. IEarnPointsOrchestrator.AwardByCampaignAsync(
      campaignId: programId,
      pointSystemTypeID: 1,  // PSP
      activityTypeId: PATIENT_ENROLLED,
      context: PointsContext,
      ...
   )
   ↓
5. لكل Rule نشطة:
   - Balance + Transaction
```

**CampaignID = ProgramID** — لأن كل برنامج PSP = حملة تسويقية.

### 5.3 الكود الفعلي (من `PSPEnrollmentService`)

```csharp
// في PSPEnrollmentService
private async Task TryAwardPointsViaOrchestratorAsync(
    int patientId, int programId, int activityTypeId,
    string referenceType, int referenceId, int? currentUserId)
{
    try
    {
        // 1. جلب المريض
        var patient = await _patientService.GetByIdAsync(patientId);
        if (patient == null) return;

        // 2. تحديد OrganizationID
        int? organizationId = patient.InvitedByOrganizationID;
        if (organizationId == null && patient.ParticipationID > 0)
        {
            var participation = await _dbFactory.ExecuteReadOnlyAsync(...);
            organizationId = participation?.ParticipantOrganizationID;
        }

        if (organizationId == null) return;

        // 3. بناء PointsContext
        var context = new PointsContext
        {
            OrganizationID = organizationId.Value,
            CaseID = patient.CaseID,        // ⭐ توضيح: القيمة هنا PatientProfileID
            UserID = patient.PatientProfileID
        };

        // 4. استدعاء Orchestrator
        var result = await _pointsOrchestrator.AwardByCampaignAsync(
            campaignId: programId,
            pointSystemTypeID: 1,  // PSP
            activityTypeId: activityTypeId,
            context: context,
            referenceType: referenceType,
            referenceId: referenceId,
            currentUserId: currentUserId);
    }
    catch (Exception ex)
    {
        _logger.LogWarning(ex, "⚠️ [Points] Failed to award points");
    }
}
```

### 5.4 الـ Controllers المُتَعلقة

`Api.Web/Controllers/PSP/Points/`:

| Controller | Endpoints | الوظيفة |
|---|---|---|
| `PointSchemesController` | 13 | إدارة Schemes |
| `PointRulesController` | 5 | إدارة Rules |
| `PointBalancesController` | 7 | أرصدة |
| `PointEarningController` | 1 | سجل معاملات |
| `PointRedeemController` | 8 | طلبات استرداد |
| `PointLookupsController` | 4 | Lookups |
| **الإجمالي** | **38** | |

انظر `06-POINTS`.

---

## 📌 الجزء السادس: الـ API Endpoints (ملخص من `09-API-GUIDE`)

### 6.1 `PspController` (12 ملف — 60 endpoint)

انظر `09-API-GUIDE`.

| الملف | Endpoints | التفاصيل |
|---|---|---|
| `PspController.Programs.cs` | 4 | البرامج (2 عام + 2 Temp) |
| `PspController.Invitations.cs` | 2 | دعوات المرضى (Doctor → Patient) |
| `PspController.RepInvitations.cs` | 2 | دعوات المندوب (Rep → Org) |
| `PspController.Enrollment.cs` | 2 | Participate + Unsubscribe |
| `PspController.Unsubscribe.cs` | 1 | Unsubscribe (UseCase) |
| `PspController.Entry.cs` | 1 | PspEntry |
| `PspController.DispensationPlans.cs` | 6 | مراحل خطة الصرف |
| `PspController.Links.cs` | 3 | روابط البرنامج |
| `PspController.patientdetails.cs` | 1 | تفاصيل برنامج المريض |
| `PspController.Tests.cs` | 38 | نظام التحاليل الكامل |
| `PspController.Helpers.cs` | 0 | Helpers |
| **الإجمالي** | **60** | |

**ملاحظة**: العدد 60 وليس 62 كما كان موثقاً سابقاً.

### 6.2 Controllers مساعدة

| Controller | Endpoints |
|---|---|
| `DispensationPlansController` | 5 |
| `PharmaCompanyMedicationsController` | 18 |
| `RemindersController` | 8 |

---

## 🆕 v2: تكامل العيادات والوصفات متعددة الأدوية (Clinic Integration & Multi-Medication eRX)

**تاريخ الإضافة**: نوفمبر 2026

**الحالة**: ✅ **منفّذ جزئياً** — الكيانات + Migrations جاهزة، الواجهات ناقصة.

**ملاحظة**: هذا القسم يضيف وظائف جديدة ولا يلغي أو يعدل أي مما ورد في الأقسام السابقة.

### 🎯 الأهداف

- تمكين الطبيب من كتابة وصفة تحتوي على أدوية متعددة (مدعومة وغير مدعومة) في سجل واحد.
- فصل "الوصفة الطبية العامة" عن "وصفة الدعم" مع إمكانية الربط بينهما.
- توفير كود إحالة ثابت للعيادة يقلل الاحتكاك في عملية اشتراك المرضى.
- دعم سيناريوهات المرضى الثلاثة: بدون حساب / حساب جديد / مريض قديم.

### 📦 الكيانات الجديدة (كلها منفّذة)

#### `ClinicProgramReferral` ⭐ — ✅ منفّذ

كود إحالة ثابت يربط عيادة ببرنامج دعم.

- **الحقول**: `ClinicID`, `ProgramID`, `ReferralCode` (فريد), `QrData`, `IsActive`
- **الدور**: نقطة الدخول الموحدة لجميع سيناريوهات المرضى. عند مسح المريض لهذا الكود، يتعرف النظام على العيادة والبرنامج تلقائياً.
- **العلاقة مع `PSPInvitation`**: لا يحل محله، بل يعمل بالتوازي معه. `PSPInvitation` للدعوات الفردية، و`ClinicProgramReferral` للكود الثابت المتعدد الاستخدام.

#### `Prescription` (رأس الوصفة العامة — Clinic Domain) ⭐ — ✅ منفّذ

وصفة طبية مستقلة عن برامج الدعم، تابعة للعيادة.

- **الحقول**: `PrescriptionID`, `PrescriptionCode` (للمشاركة عبر واتساب/QR), `CaseID`, `DoctorID`, `ClinicID`, `PrescriptionDate`, `Notes`
- **الرؤية المستقبلية**: سينتقل هذا الكيان لقاعدة بيانات العيادة المستقلة عند فصل وحدة أعمال إدارة العيادات.

#### `PrescriptionItem` (تفاصيل الوصفة العامة) ⭐ — ✅ منفّذ

- **الحقول**: `PrescriptionItemID`, `PrescriptionID` (FK), `MedicationID`, `Dosage`, `Frequency`, `Instructions`, `TotalQuantity`
- **الربط الاختياري بالدعم**: `LinkedPspERxItemID` (int? nullable) → إذا كان الدواء مدعوماً، يتم ربطه بسجل `PSPeRXItem` المقابل.

#### `PSPeRX` (رأس وصفة الدعم — PSP Domain) ⭐ مُعدّل — ✅ منفّذ

- **التعديل عن النسخة السابقة**: `MedicationID` أصبح **nullable** (كان إلزامياً `[Required]`).
- **الحقول**: `ERxID`, `ERxCode`, `CaseID`, `DoctorID`, `ProgramID` (إلزامي), `PrescriptionDate`, `Notes`, `Status`
- **الدور**: يمثل الوصفة المرتبطة ببرنامج دعم محدد. يحتفظ بجميع علاقاته القائمة مع `PSPDispensation` و`PSPPatient`.

#### `PSPeRXItem` ⭐ — ✅ منفّذ (تفاصيل الأدوية المدعومة)

- **الحقول**: `ERxItemID`, `ERxID` (FK إلى `PSPeRX`), `ProgramMedicationID` (FK إلى `PSPProgramMedication`), `Dosage`, `Frequency`, `Instructions`, `Quantity`
- **تعديل نظام التناول (Override)**:
  - `OverrideTimesPerDay` (int? nullable)
  - `OverrideQuantityPerDispense` (int? nullable)
  - `IsOverridden` (bool) → لتتبع ما إذا كان الطبيب قد عدّل الإعدادات الافتراضية المحددة من شركة الأدوية.
- **القيد**: لا يمكن إضافة دواء غير موجود في `PSPProgramMedication` للبرنامج المختار. المصدر الحقيقي للأدوية المدعومة ونظام صرفها يظل `PSPProgramMedication` و`PSPDispensationPlanPhaseMedication`.

### 🔄 تدفق عمل الطبيب (Workflow) — الحالة الفعلية

1. الطبيب يفتح صفحة كتابة وصفة لمريض. ⚠️ **الصفحة غير موجودة بعد** — `InvitePatient.razor` فيه زر "Prescription" (`OnPrescriptionClick`) لكنه غير مربوط.
2. يختار المريض من قائمة مرضاه في العيادة. ✅ (`ClinicPatientsList.razor` موجود)
3. للأدوية العادية (غير المدعومة): يضيفها مباشرة → تُنشأ `PrescriptionItem` فقط (`LinkedPspERxItemID = null`).
4. للأدوية المدعومة:
   - يختار برنامج دعم من قائمة برامعه المشتركة (`PSPParticipation`).
   - تظهر له قائمة `PSPProgramMedication` الخاصة بهذا البرنامج، مع تفاصيل الصرف والتناول المحددة من الشركة (`PSPDispensationPlanPhaseMedication`).
   - يختار دواءً أو أكثر من هذه القائمة (ليس جميعها بالضرورة).
   - إذا سمح البرنامج بتعديل نظام التناول، تظهر حقول `TimesPerDay` و`QuantityPerDispense` قابلة للتعديل.
5. عند الحفظ:
   - تُنشأ `PSPeRX` مرتبطة بالبرنامج والمريض.
   - تُنشأ `PSPeRXItems` للأدوية المختارة.
   - تُنشأ `PrescriptionItems` مقابلة، كل منها مرتبط بـ `LinkedPspERxItemID`.

**الحالة الفعلية**: الخطوات 1، 3، 4، 5 **غير منفّذة** في الواجهة. الكيانات + Migrations جاهزة، لكن منطق الحفظ في الـ API غير مؤكد.

### 👥 سيناريوهات المرضى

| السيناريو | الإجراء التقني | الكيانات المعنية |
|---|---|---|
| مريض بدون حساب | الطبيب يكتب وصفة → يولد `PrescriptionCode` للمشاركة → المريض يستخدمه لاحقاً للتسجيل | `Prescription`, `PrescriptionItem` |
| مريض جديد (داخل العيادة) | يمسح `ClinicProgramReferral` QR → يظهر البرنامج والعيادة → يشترك → يحجز موعداً فورياً → يظهر للطبيب → يكتب الوصفة | `ClinicProgramReferral`, `PSPPatient`, `Prescription`, `PSPeRX` |
| مريض قديم (متابعة) | يمسح نفس QR → النظام يتعرف عليه عبر `UserProfileID` → يجد اشتراكه فعالاً في `PSPPatient` → يفتح شاشة حجز متابعة → الطبيب يكتب وصفة جديدة مرتبطة بنفس `CaseID` | `ClinicProgramReferral`, `PSPPatient`, `Prescription`, `PSPeRX` |

### 🔮 الرؤية المستقبلية (Scalability)

- `Prescription` و`PrescriptionItem` سينتقلان لقاعدة بيانات العيادة المستقلة عند اكتمال وحدة أعمال إدارة العيادات.
- `PSPeRX` و`PSPeRXItem` سيبقيان في قاعدة PSP المركزية.
- الربط بين القاعدتين سيتم عبر `LinkedPspERxItemID` كـ Foreign Key اختياري، أو عبر API/Event Bus بعد الفصل الكامل.
- تفعيل `DoctorNote` و`PSPRequiredDataEntry` في واجهة الطبيب والمريض ضمن نفس التدفق. ⚠️ **لا يزال غير منفّذ في الواجهة**.

### ⚠️ الفجوات المعروفة (بعد المراجعة)

| # | الفجوة | الأولوية |
|---|---|---|
| 1 | `POST api/psp/entry` لا يربط الدعوة/الإحالة بالمشاركة | 🔴 |
| 2 | `Join.razor` لا يكمل الاشتراك (يعرض الكود فقط) | 🔴 |
| 3 | `WritePrescriptionPage` غير موجودة | 🔴 |
| 4 | `DoctorNotesTab` غير موجود | 🔴 |
| 5 | `PatientRequiredDataEntryPage` غير موجودة | 🔴 |
| 6 | `ClinicReferralCard.razor` (QR + ShortCode) غير مؤكد وجوده | 🟠 |
| 7 | Endpoint تقارير الإحالة (`referral-reports`) غير مؤكد | 🟡 |
| 8 | كيان `PSPPatientDataEntry` (إجابات المريض) غير مؤكد | 🔴 |

---

## 📌 الجزء السابع: الـ UI

### 7.1 المكونات في `Shared.UI`

| # | المكوّن | الوظيفة | الحالة |
|---|---|---|---|
| 1 | `PspAboutPage.razor` | عن برامج الدعم | ✅ |
| 2 | `PspGateway.razor` | بوابة PSP | ✅ |
| 3 | `PspSearch.razor` | بحث برامج | ✅ |
| 4 | `PspEntry.razor` | دخول برنامج | ✅ |
| 5 | `PspScheduleSetup.razor` | إعداد جدولة | ✅ |
| 6 | `PspDetailComponent.razor` | تفاصيل برنامج | ✅ |
| 7 | `ProgramDetailsComponent.razor` | تفاصيل برنامج (طبيب) | ✅ |
| 8 | `PointDashboard.razor` ⭐ | محفظة النقاط | ✅ |
| 9 | `ClinicDashboard.razor` ⭐ | لوحة تحكم الطبيب | ✅ |
| 10 | `ClinicPatientDetails.razor` ⭐ | تفاصيل مريض | ✅ |
| 11 | `ClinicPatientsList.razor` ⭐ | قائمة مرضى العيادة | ✅ |
| 12 | `MyClinic.razor` ⭐ | إدارة العيادة | ✅ |
| 13 | `MyPatientInvitations.razor` ⭐ | سجل دعوات المرضى | ✅ |
| 14 | `InvitePatient.razor` ⭐ | دعوة مريض (QR + ShortCode) | ✅ |
| 15 | `Join.razor` ⭐ (`/join`) | استقبال `?token=` | ⚠️ لا يكمل الاشتراك |
| 16 | `WritePrescriptionPage.razor` ⭐ | كتابة eRX متعدد الأدوية | ❌ **غير موجود** |
| 17 | `PrescriptionViewPage.razor` ⭐ | عرض الوصفة للمريض | ❌ **غير موجود** |
| 18 | `DoctorNotesTab.razor` ⭐ | ملاحظات الطبيب | ❌ **غير موجود** |
| 19 | `PatientRequiredDataEntryPage.razor` ⭐ | إدخال البيانات المطلوبة | ❌ **غير موجود** |
| 20 | `ClinicReferralCard.razor` ⭐ | كارت QR + ShortCode للإحالة | ❓ **غير مؤكد** |

### 7.2 الصفحات في `Rubikcare.Web`

| # | الصفحة | المسار |
|---|---|---|
| 1 | `PSPProgramsList.razor` | `/psp/programs` |
| 2 | `PSPProgramDetails.razor` | `/psp/programs/{id}` |
| 3 | `PSPProgramEdit.razor` | `/psp/programs/{id}/edit` |
| 4 | `PSPStep0_BasicInfo.razor` | خطوة 0 |
| 5 | `PSPStep1_SpecsMeds.razor` | خطوة 1 |
| 6 | `PSPStep2_Budget.razor` | خطوة 2 |
| 7 | `PSPStep2_DispensationPlans.razor` | خطوة 2 (خطط الصرف) |
| 8 | `PSPStep2b_PointScheme.razor` | خطوة 2ب (نظام النقاط) |
| 9 | `PSPStep3_RequiredTests.razor` | خطوة 3 (التحاليل) — ⚠️ قد لا يكون موجوداً |
| 10 | `PSPStep3_Review.razor` | خطوة 3 (مراجعة) |
| 11 | `PSPStep_Links.razor` | روابط |

### 7.3 الصفحات في `PWA`

| # | الصفحة | المسار |
|---|---|---|
| 1 | `PspEntryPage.razor` | `/psp/entry` |
| 2 | `PspGatewayPage.razor` | `/psp/gateway` |
| 3 | `PspSearchPage.razor` | `/psp/search` |
| 4 | `PspDetailPage.razor` | `/psp/{id}` |
| 5 | `PointsDashboardPage.razor` | `/psp/points` |

### 7.4 الصفحات في `MAUI`

| # | الصفحة | المسار |
|---|---|---|
| 1 | `PspEntryPage.xaml` | — |
| 2 | `PspDetailPage.xaml` | — |
| 3 | `PointsDashboardPage.xaml` | — |

### 7.5 الـ CSS

انظر `10-UI-STYLE-GUIDE`.

الملفات في `Shared.UI/wwwroot/css/_pages/Organization/PSP/`:

```
PSP/
├── psp-detail.css
├── PSPProgramsList.css
├── PSPProgramDetails.css
├── PSPProgramsEdit.css
├── psp-search.css
├── PSPEditSteps/
│   ├── PSPStep0_BasicInfo.css
│   ├── PSPStep1_SpecsMeds.css
│   ├── PSPStep2_Budget.css
│   ├── PSPStep2_DispensationPlans.css
│   ├── PSPStep2b_PointScheme.css
│   ├── PSPStep_Links.css
│   ├── PSPStep3_Review.css
│   └── PSPRequiredTestsSection.css
├── program-details.css
├── invite-patient.css
├── join.css
└── Points/
    ├── point-dashboard.css
    ├── ppm-program-manage.css
    └── beneficiary-type-selector.css
```

`_pspbundle.css` → `main.css`.

---

## 📌 الجزء الثامن: مفاتيح الترجمة

### 8.1 Module: `PSP`

انظر `07 v1`:

| البادئة | الوصف |
|---|---|
| `PSP.DP.MODAL.TAB_*` | تابين الـ Modal |
| `PSP.DP.PHASES.*` | قسم المراحل |
| `PSP.DP.PHASE_MODAL.*` | Modal المرحلة |
| `PSP.DP.MED_MODAL.*` | Modal الدواء |
| `PSP.DP.MSG.*` | رسائل |
| `PSP.DP.UNIT.*` | وحدات |

### 8.2 مفاتيح إضافية (من Tests System)

| المفتاح | القيمة |
|---|---|
| `PSP.TESTS.TITLE` | "التحاليل" |
| `PSP.TESTS.REQUIRED` | "مطلوب" |
| `PSP.TESTS.OPTIONAL` | "اختياري" |
| `PSP.TESTS.TIMING.BEFORE` | "قبل البدء" |
| `PSP.TESTS.TIMING.DURING` | "أثناء الرحلة" |
| `PSP.TESTS.TIMING.AFTER` | "بعد N أيام" |
| `PSP.TESTS.TIMING.PERIODIC` | "دوري" |
| `PSP.TESTS.STATUS.ACTIVE` | "متاح" |
| `PSP.TESTS.STATUS.LOCKED` | "مقفل" |
| `PSP.TESTS.STATUS.COMPLETED` | "مكتمل" |
| `PSP.TESTS.CONSENT.TEXT` | "أوافق على مشاركة..." |
| `PSP.TESTS.UPLOAD` | "رفع ملف" |
| `PSP.TESTS.VIEW` | "عرض" |

### 8.3 قاعدة صارمة

انظر `04-TRANSLATION`:
- `N''` prefix للعربية.
- `MERGE` بدلاً من `INSERT/UPDATE`.
- `ResourceKey` + `Module` = مفتاح مركب.

---

## 📌 الجزء التاسع: الاختبارات + Troubleshooting

### 9.1 سيناريوهات E2E المطلوبة

انظر `23-TECHNICAL_DEBT` (المُدمج في `00B`):

| # | السيناريو | الحالة |
|---|---|---|
| 1 | Rep → Doctor: إنشاء دعوة → قبول | ✅ |
| 2 | Rep → Pharmacy: إنشاء دعوة → قبول | ✅ |
| 3 | Doctor → Patient: إنشاء دعوة → قبول → Token | ⚠️ |
| 4 | Patient → Dispense: Token → Validate → Confirm | ✅ |
| 5 | Points: منح النقاط في 5 أحداث | ✅ |
| 6 | Tests: رفع تحليل → موافقة → عرض للشركة | ⚠️ |
| 7 | ⭐ **Clinic → Patient: مسح QR إحالة → اشتراك → إدخال بيانات مطلوبة** | ❌ **جديد** |
| 8 | ⭐ **Doctor → Patient: كتابة eRX متعدد الأدوية → مشاركة → عرض** | ❌ **جديد** |
| 9 | ⭐ **Doctor → Patient: إضافة DoctorNote → ظهورها** | ❌ **جديد** |

### 9.2 المشاكل المعروفة

انظر `23-TECHNICAL_DEBT`:

| # | المشكلة | الحالة |
|---|---|---|
| 1 | `POST api/psp/entry` — لا يربط الدعوة بالمشاركة | ❌ مفتوح |
| 2 | `ResultingParticipationParticipationID` — خطأ إملائي | ❌ مفتوح (لكن `InverseProperty` صحيحة في الكيان) |
| 3 | `programs-temp` + `program-details-temp` — endpoints مؤقتة | ⚠️ |
| 4 | `Unsubscribe` مكرر (في `Enrollment.cs` + `Unsubscribe.cs`) | ⚠️ |
| 5 | `PharmaciesCount` Hardcoded في `patientdetails` | ⚠️ |
| 6 | `PSPStep3_RequiredTests.razor` — قد لا يكون موجوداً | ⚠️ |
| 7 | `PSPEnrollmentService` يستدعي `IGenericService<T>` (LVL 2) | ⚠️ |
| 8 | ⭐ **`Join.razor` لا يكمل الاشتراك** | ❌ **جديد** |
| 9 | ⭐ **`WritePrescriptionPage` غير موجودة** | ❌ **جديد** |
| 10 | ⭐ **`DoctorNotesTab` غير موجود** | ❌ **جديد** |
| 11 | ⭐ **`PatientRequiredDataEntryPage` غير موجودة** | ❌ **جديد** |
| 12 | ⭐ **كيان `PSPPatientDataEntry` غير مؤكد** | ❓ **جديد** |

### 9.3 Troubleshooting — مشاكل شائعة

#### مشكلة: "التحليل لا يظهر للمريض"

- **السبب المحتمل**: `ShowToPatient = false`.
- **الحل**: تحقق من `PSPRequiredTest.ShowToPatient`.

#### مشكلة: "النقاط لا تُمنح"

- **السبب المحتمل**: `PSPProgramPointScheme` غير موجود للبرنامج.
- **الحل**:
  ```sql
  SELECT * FROM PSPProgramPointSchemes WHERE SourceEntityID = @ProgramId;
  ```

#### مشكلة: "الصيدلي لا يرى الطلب"

- **السبب المحتمل**: `Dispensation.TokenUsed = true` أو `Status != "ACTIVE"`.
- **الحل**: تحقق من `PSPDispensation.Status`.

#### ⭐ مشكلة: "المريض يمسح QR الإحالة ولا يكمل الاشتراك"

- **السبب المحتمل**: `Join.razor` يعرض الكود فقط، ولا يستدعي `POST api/psp/entry`.
- **الحل**: تعديل `Join.razor` لاستدعاء الـ endpoint بعد عرض الكود (أو دمجه).

#### ⭐ مشكلة: "DoctorNote موجودة في DB لكن لا تظهر في صفحة الطبيب"

- **السبب المحتمل**: لا يوجد `DoctorNotesTab.razor` مربوط بـ `ClinicPatientDetails`.
- **الحل**: إنشاء المكون + Endpoints (`POST/GET api/clinic/patients/{caseId}/notes`).

---

## 📌 الجزء العاشر: Roadmap + روابط

### 10.1 Roadmap المُتبقي

| # | المهمة | الأولوية | الحالة |
|---|---|---|---|
| 1 | إكمال Patient Mobile (Token) | 🔴 | 🟡 |
| 2 | حل مشكلة `POST api/psp/entry` (ربط الدعوة/الإحالة) | 🔴 | ❌ |
| 3 | `MyInvitationsPage` (سجل الدعوات) | 🟠 | ❌ |
| 4 | تعديل `GetMyDoctors` (USED فقط) | 🟠 | ❌ |
| 5 | حذف `programs-temp` | 🟠 | ❌ |
| 6 | حذف `Unsubscribe` المكرر | 🟠 | ❌ |
| 7 | ربط `PharmaciesCount` | 🟡 | ❌ |
| 8 | `PSPStep3_RequiredTests.razor` (UI) | 🟠 | ❌ |
| 9 | ربط `PROGRAM_COMPLETED` | 🟠 | ⏸️ |
| 10 | ربط `LAB_TEST_UPLOADED` | 🟠 | ⏸️ |
| 11 | تحسين `PSPEnrollmentService` → UseCase | 🟡 | ❌ |
| 12 | ⭐ **إصلاح `Join.razor` لإكمال الاشتراك** | 🔴 | ❌ **جديد** |
| 13 | ⭐ **إنشاء `WritePrescriptionPage`** | 🔴 | ❌ **جديد** |
| 14 | ⭐ **إنشاء `DoctorNotesTab`** | 🔴 | ❌ **جديد** |
| 15 | ⭐ **إنشاء `PatientRequiredDataEntryPage`** | 🔴 | ❌ **جديد** |
| 16 | ⭐ **تأكيد/إنشاء كيان `PSPPatientDataEntry`** | 🔴 | ❓ **جديد** |
| 17 | ⭐ **إنشاء `ClinicReferralCard`** (إن لم يكن موجوداً) | 🟠 | ❓ **جديد** |
| 18 | ⭐ **Endpoint `referral-reports`** | 🟡 | ❓ **جديد** |

### 10.2 الاعتمادات

- `POST api/psp/entry` — مستقل.
- Patient Mobile — يحتاج Token Flow.
- `PSPStep3_RequiredTests` — يحتاج Tests System (موجود في API).
- ⭐ `WritePrescriptionPage` — يحتاج `Prescription` + `PSPeRXItem` (منفّذان).
- ⭐ `DoctorNotesTab` — يحتاج `DoctorNote` (منفّذ) + Endpoints.
- ⭐ `PatientRequiredDataEntryPage` — يحتاج `PSPRequiredDataEntry` (منفّذ) + `PSPPatientDataEntry` (غير مؤكد).

### 10.3 روابط ذات صلة

- [00B - Project Backlog](./00B-PROJECT-BACKLOG.md)
- [01 - Architecture](./01-ARCHITECTURE.md)
- [06 - Points](./06-POINTS.md)
- [07 - Reports](./07-REPORTS.md)
- [09 - API Guide](./09-API-GUIDE.md)
- [10 - UI Style Guide](./10-UI-STYLE-GUIDE.md)
- [13 - MAUI](./13-MAUI.md)
- [23 - Technical Debt](./23-TECHNICAL_DEBT.md)

### 10.4 سجل التغييرات

| الإصدار | التاريخ | التغييرات |
|---|---|---|
| 1.0 | 18 يوليو 2026 | الإصدار الأولي — `07-psp-system.md` |
| 1.0 | أكتوبر 2026 | الإصدار الأولي — `USERS STEP FLOW` |
| 2.0 | 17 أكتوبر 2026 | `05-PSP v2.0` — دمج 07 + User Flow + Tests System + Phase System + Points Integration |
| **2.1** | **24 نوفمبر 2026** | **توحيد `CaseID` (حذف `PatientProfileID` كاسم حقل)؛ إضافة `ClinicProgramReferral`, `PSPeRXItem`, `Prescription`, `PrescriptionItem` إلى §0.2 و§1.1؛ تصحيح `RequiredTiming` إلى 5 قيم (بما فيها `OnEvent`)؛ تصحيح `PspController` إلى 60 endpoint؛ تصحيح Test Status إلى 4 حالات؛ إضافة R6/R7؛ إضافة §4.2 خطوة 13 و§4.4 خطوة 13؛ إضافة فجوات معروفة في §v2؛ إضافة مكونات UI جديدة في §7.1 مع حالتها؛ إضافة مشاكل 8–12 في §9.2؛ إضافة مهام 12–18 في §10.1؛ إضافة Troubleshooting لـ QR وDoctorNote؛ إصلاح عناوين مقطوعة (`2.3.1`, `3.9`, `7.1`)؛ إكمال جداول مقطوعة.** |

---

© 2026 RubikCare — للاستخدام الداخلي

**نهاية الوثيقة** ✅
```

---
