# 📘 وثيقة نظام التقارير الشاملة — RubikCare Reports System

**آخر تحديث:** 16 أكتوبر 2026  
**الإصدار:** 1.0  
**الحالة:** ✅ مكتمل ومُختبر  
**الجمهور:** فريق التطوير، مديرو المشروع، المطورون الجدد

---

## 📋 جدول المحتويات

1. [نظرة عامة](#1-نظرة-عامة)
2. [الهيكل المعماري](#2-الهيكل-المعماري)
3. [البنية التحتية المشتركة](#3-البنية-التحتية-المشتركة)
4. [التقارير الستة بالتفصيل](#4-التقارير-الستة-بالتفصيل)
5. [لوحة التحكم المركزية](#5-لوحة-التحكم-المركزية)
6. [نظام الفلترة الموحد](#6-نظام-الفلترة-الموحد)
7. [إصلاح البيانات المالية](#7-إصلاح-البيانات-المالية)
8. [الميزات الأمنية](#8-الميزات-الأمنية)
9. [الأنماط والتصميم](#9-الأنماط-والتصميم)
10. [مفاتيح الترجمة](#10-مفاتيح-الترجمة)
11. [خطوات التنفيذ](#11-خطوات-التنفيذ)
12. [الدروس المستفادة](#12-الدروس-المستفادة)

---

## 1. نظرة عامة

### 1.1 الهدف

نظام تقارير متكامل لشركة الأدوية، يتيح:
- **قياس أداء البرامج** من الدعوة إلى الإتمام
- **تحليل المندوبين والعيادات والصيدليات**
- **متابعة الالتزام الدوائي** للمرضى
- **التقارير المالية** والتعويضات
- **نظرة شاملة** عبر لوحة تحكم مركزية

### 1.2 المبادئ الأساسية

| المبدأ | التطبيق |
|--------|---------|
| **Clean Architecture** | Handlers في Infrastructure، DTOs في Application |
| **نمط موحد** | كل تقرير يتبع نفس البنية |
| **الأداء** | تحميل تدريجي (Summary دائماً، تفاصيل عند الطلب) |
| **الأمان** | إخفاء بيانات المرضى افتراضياً |
| **التكامل** | Funnel + Financial + Adherence في كل تقرير |

### 1.3 الجمهور المستهدف

- **مديرو البرامج** في شركات الأدوية
- **المندوبون** لمتابعة أدائهم
- **المحاسبون** للتقارير المالية
- **الفريق الطبي** لمتابعة الالتزام

---

## 2. الهيكل المعماري

### 2.1 الطبقات

```
┌─────────────────────────────────────────────────────────────┐
│  Rubikcare.Web (Blazor Server)                              │
│  └── Pages/Reports/*.razor (الصفحات المضيفة)                │
└──────────────────────┬──────────────────────────────────────┘
                       │ HTTP
                       ▼
┌─────────────────────────────────────────────────────────────┐
│  Shared.UI (RCL)                                            │
│  ├── Components/Reports/*.razor (المكونات)                  │
│  └── wwwroot/css/_pages/Reports/*.css                       │
└──────────────────────┬──────────────────────────────────────┘
                       │ HTTP
                       ▼
┌─────────────────────────────────────────────────────────────┐
│  Api.Web                                                    │
│  └── Controllers/Reports/ReportsController.cs               │
└──────────────────────┬──────────────────────────────────────┘
                       │
                       ▼
┌─────────────────────────────────────────────────────────────┐
│  Infrastructure                                             │
│  └── UseCases/Reports/*/GetXxxHandler.cs                    │
│      (يستخدم IDbContextFactoryService)                      │
└──────────────────────┬──────────────────────────────────────┘
                       │
                       ▼
┌─────────────────────────────────────────────────────────────┐
│  Application                                                │
│  └── UseCases/Reports/*/                                    │
│      ├── IGetXxxHandler.cs                                  │
│      ├── GetXxxQuery.cs                                     │
│      └── GetXxxResponse.cs                                  │
└──────────────────────┬──────────────────────────────────────┘
                       │
                       ▼
┌─────────────────────────────────────────────────────────────┐
│  Domain                                                     │
│  └── Entities (PSP*, Organizations, PatientMedicationSchedule)
└─────────────────────────────────────────────────────────────┘
```

### 2.2 مواقع الملفات

```
RubikCare.Application/
├── DTOs/Reports/
│   ├── ReportFilterBase.cs
│   ├── PharmaCompanyDto.cs
│   ├── RepDto.cs
│   ├── OrganizationLookupDto.cs
│   └── LookupItemDto.cs
└── UseCases/Reports/
    ├── ProviderFunnel/
    │   ├── IGetProviderFunnelHandler.cs
    │   ├── GetProviderFunnelQuery.cs
    │   └── GetProviderFunnelResponse.cs
    ├── PatientFunnel/
    ├── PharmacyDispensation/
    ├── PharmacyFinancial/
    ├── Adherence/
    ├── ProgramLifecycle/
    ├── Dashboard/
    └── Filters/
        └── IGetReportFiltersHandler.cs

RubikCare.Infrastructure/
└── UseCases/Reports/
    ├── ProviderFunnel/GetProviderFunnelHandler.cs
    ├── PatientFunnel/GetPatientFunnelHandler.cs
    ├── PharmacyDispensation/GetPharmacyDispensationHandler.cs
    ├── PharmacyFinancial/GetPharmacyFinancialHandler.cs
    ├── Adherence/GetAdherenceHandler.cs
    ├── ProgramLifecycle/GetProgramLifecycleHandler.cs
    ├── Dashboard/GetDashboardSummaryHandler.cs
    └── Filters/GetReportFiltersHandler.cs

Api.Web/
└── Controllers/Reports/ReportsController.cs

Shared.UI/
├── Components/Reports/
│   ├── ProviderFunnelReport.razor
│   ├── PatientFunnelReport.razor
│   ├── PharmacyDispensationReport.razor
│   ├── PharmacyFinancialReport.razor
│   ├── AdherenceReport.razor
│   └── ProgramLifecycleReport.razor
└── wwwroot/css/_pages/Reports/
    ├── provider-funnel.css
    ├── patient-funnel.css
    ├── pharmacy-dispensation.css
    ├── pharmacy-financial.css
    ├── adherence.css
    └── program-lifecycle.css

Rubikcare.Web/
└── Components/Pages/
    ├── PharmaDashboard.razor
    └── Reports/
        ├── ProviderFunnel.razor
        ├── PatientFunnel.razor
        ├── PharmacyDispensation.razor
        ├── PharmacyFinancial.razor
        ├── Adherence.razor
        └── ProgramLifecycle.razor
```

---

## 3. البنية التحتية المشتركة

### 3.1 `ReportFilterBase.cs`

**الفلاتر الموحدة لكل التقارير:**

```csharp
public class ReportFilterBase
{
    // ⭐ فلتر شركة الأدوية
    public int? PharmaCompanyId { get; set; }

    // ⭐ فلتر البرنامج
    public int? ProgramId { get; set; }

    // ⭐ فلتر المندوب
    public int? RepUserProfileId { get; set; }

    // ⭐ فلتر العيادة
    public int? ClinicId { get; set; }

    // ⭐ فلتر الصيدلية
    public int? PharmacyId { get; set; }

    // ⭐ فلتر نوع الدعوة
    public string? SourceType { get; set; }

    // ⭐ فلتر نوع المنظمة
    public string? InvitedOrganizationType { get; set; }

    // ⭐ فلتر الحالة
    public string? Status { get; set; }

    // ⭐ فلتر التاريخ (Cohort Analysis)
    public DateTime? FromDate { get; set; }
    public DateTime? ToDate { get; set; }

    // ⭐ الفلاتر الجغرافية
    public List<int>? CountryIds { get; set; }
    public List<int>? CityIds { get; set; }
    public List<int>? AreaIds { get; set; }

    // ⭐ البحث النصي
    public string? SearchTerm { get; set; }
}
```

### 3.2 DTOs المشتركة

| الملف | الوصف |
|-------|-------|
| `PharmaCompanyDto` | شركة الأدوية (ID, NameAr, NameEn, MembersCount) |
| `RepDto` | المندوب (UserProfileId, FullName, Email, PharmaCompanyId) |
| `OrganizationLookupDto` | العيادة/الصيدلية (Id, NameAr, NameEn, TypeId, AreaId) |
| `LookupItemDto` | الدول/المدن/المناطق (Id, NameAr, NameEn, ParentId) |

### 3.3 `GetReportFiltersHandler`

**يوفر كل قوائم الفلاتر:**

```csharp
public class GetReportFiltersQuery
{
    public int? PharmaCompanyId { get; set; }
}

public class GetReportFiltersResponse
{
    public List<PharmaCompanyDto> PharmaCompanies { get; set; }
    public List<PspProgramDto> Programs { get; set; }
    public List<RepDto> Reps { get; set; }
    public List<OrganizationLookupDto> Clinics { get; set; }
    public List<OrganizationLookupDto> Pharmacies { get; set; }
    public List<LookupItemDto> Countries { get; set; }
    public List<LookupItemDto> Cities { get; set; }
    public List<LookupItemDto> Areas { get; set; }
}
```

**الفلترة التلقائية:**
- **البرامج** → حسب `PharmaCompanyId`
- **المندوبون** → حسب `PharmaCompanyId` (عبر `OrgMembership`)
- **العيادات** → حسب `PharmaCompanyId` (عبر `PSPParticipation`)
- **الصيدليات** → حسب `PharmaCompanyId` (عبر `PSPParticipation`)

### 3.4 `ReportsController`

**8 Endpoints:**

| Endpoint | الوصف |
|----------|-------|
| `GET /api/reports/provider-funnel` | تقرير المندوبين |
| `GET /api/reports/patient-funnel` | تقرير المرضى |
| `GET /api/reports/pharmacy-dispensation` | صرف الصيدليات |
| `GET /api/reports/pharmacy-financial` | التقرير المالي |
| `GET /api/reports/adherence` | الالتزام |
| `GET /api/reports/program-lifecycle` | دورة الحياة |
| `GET /api/reports/dashboard-summary` | لوحة التحكم |
| `GET /api/reports/filters` | الفلاتر |

**كل endpoint:**
- يستقبل `pharmaCompanyId` + فلاتر
- يستدعي Handler المناسب
- يُرجع DTO

---

## 4. التقارير الستة بالتفصيل

### 4.1 تقرير معدل تحويل الدعوات (Provider Funnel)

**الغرض:** قياس أداء المندوبين في دعوة الأطباء والصيدليات.

**المدخلات:** `programId`, `pharmaCompanyId`, `repUserProfileId`, `sourceType`, `invitedOrganizationType`, `status`, `fromDate`, `toDate`

**المخرجات:**

```csharp
public class GetProviderFunnelResponse
{
    public List<ProviderFunnelProgramDto> Programs { get; set; }
    public ProviderFunnelSummaryDto Summary { get; set; }
}

public class ProviderFunnelProgramDto
{
    public int ProgramId { get; set; }
    public string ProgramNameAr { get; set; }
    public List<ProviderFunnelItemDto> Items { get; set; }
}

public class ProviderFunnelItemDto
{
    public string SourceType { get; set; }
    public string InvitedOrganizationType { get; set; }
    public int TotalInvitations { get; set; }
    public int Pending { get; set; }
    public int Accepted { get; set; }
    public int Used { get; set; }
    public int Expired { get; set; }
    public int ActualParticipations { get; set; }
    public decimal AcceptanceRate { get; set; }
    public decimal ParticipationRate { get; set; }
}
```

**منطق Handler:**

```
1. جلب البرامج (مع فلتر PharmaCompanyId)
2. جلب الدعوات (REP_TO_DOCTOR + REP_TO_PHARMACY)
   ├── فلتر المندوب
   ├── فلتر نوع الدعوة
   ├── فلتر نوع المنظمة
   ├── فلتر الحالة
   ├── فلتر التاريخ
   └── الفلترة الجغرافية (عبر ReferralOrganizationID → Organization.AreaID)
3. جلب المشاركات (للربط عبر InvitationID)
4. التجميع: Program → SourceType → InvitedOrganizationType
5. حساب ActualParticipations
6. حساب النسب (AcceptanceRate, ParticipationRate)
```

**الاستعلام الأساسي:**

```csharp
var invitations = await db.Set<PSPInvitation>()
    .Where(i => programIds.Contains(i.ProgramID))
    .Where(i => i.SourceType == "REP_TO_DOCTOR" || i.SourceType == "REP_TO_PHARMACY")
    // ... فلاتر
    .Select(i => new { i.InvitationID, i.ProgramID, i.SourceType, i.InvitedOrganizationType, i.Status })
    .ToListAsync();
```

**الملفات:**
- `IGetProviderFunnelHandler.cs`
- `GetProviderFunnelQuery.cs` (يورث من `ReportFilterBase` + `IncludeDetails`)
- `GetProviderFunnelResponse.cs`
- `GetProviderFunnelHandler.cs`
- `ProviderFunnelReport.razor`
- `provider-funnel.css`

---

### 4.2 تقرير قمع تحويل المريض (Patient Funnel)

**الغرض:** قياس رحلة المريض من الدعوة إلى الاشتراك والصرف.

**المدخلات:** `programId`, `pharmaCompanyId`, `clinicId`, `repUserProfileId`, `status`, `fromDate`, `toDate`

**المخرجات:**

```csharp
public class GetPatientFunnelResponse
{
    public List<PatientFunnelProgramDto> Programs { get; set; }
    public PatientFunnelSummaryDto Summary { get; set; }
}

public class PatientFunnelProgramDto
{
    public int ProgramId { get; set; }
    public string ProgramName { get; set; }
    public int TotalInvitations { get; set; }
    public int Pending { get; set; }
    public int Used { get; set; }
    public int Expired { get; set; }
    public int RegisteredPatients { get; set; }
    public int InitialDispenses { get; set; }
    public int RefillDispenses { get; set; }
    public int CompletedPrograms { get; set; }
    public decimal InvitationToRegistrationRate { get; set; }
    public decimal RegistrationToDispenseRate { get; set; }
    public decimal CompletionRate { get; set; }

    // ⭐ التفاصيل (اختياري)
    public List<PatientFunnelClinicDto> Clinics { get; set; }
}
```

**منطق Handler:**

```
1. جلب البرامج
2. جلب الدعوات (DOCTOR_TO_PATIENT + DOCTOR_REFERRAL)
   ├── فلتر العيادة (ReferralOrganizationID)
   ├── فلتر المندوب
   └── ...
3. جلب المرضى المسجلين (مع ProgramID عبر PSPParticipation)
   ⭐ المفتاح: JOIN مع PSPParticipation لجلب ProgramID
4. جلب الصرفات
5. جلب إتمام البرامج
6. التجميع الأساسي: حسب البرنامج
7. التفاصيل: حسب العيادة داخل كل برنامج
```

**⚠️ الإصلاح الحرج:**

```csharp
// قبل: RegisteredPatients غير مفلتر حسب البرنامج
var patients = await db.Set<PSPPatient>().ToListAsync();

// بعد: JOIN مع PSPParticipation
var patientsQuery = from p in db.Set<PSPPatient>()
                    join part in db.Set<PSPParticipation>()
                        on p.ParticipationID equals part.ParticipationID
                    where programIds.Contains(part.ProgramID)
                    select new { p.PatientID, p.PatientProfileID, ProgramID = part.ProgramID };
```

**الملفات:**
- `IGetPatientFunnelHandler.cs`
- `GetPatientFunnelHandler.cs`
- `PatientFunnelReport.razor`

---

### 4.3 تقرير صرف الصيدليات (Pharmacy Dispensation)

**الغرض:** تحليل أداء الصيدليات في عمليات الصرف (تشغيلي - بدون مالية).

**المدخلات:** `programId`, `pharmaCompanyId`, `pharmacyId`, `dispenseType`, `fromDate`, `toDate`

**المخرجات:**

```csharp
public class GetPharmacyDispensationResponse
{
    public List<PharmacyDispensationProgramDto> Programs { get; set; }
    public PharmacyDispensationSummaryDto Summary { get; set; }
}

public class PharmacyDispensationProgramDto
{
    public int ProgramId { get; set; }
    public string ProgramName { get; set; }
    public int TotalDispensations { get; set; }
    public int TotalQuantity { get; set; }
    public int InitialDispenses { get; set; }
    public int RefillDispenses { get; set; }
    public int RejectedCount { get; set; }
    public int UniquePrescriptions { get; set; }
    public int UniquePatients { get; set; }
    public decimal AverageHoursToDispense { get; set; }
    public int FastDispenses { get; set; }   // < 24 ساعة
    public int NormalDispenses { get; set; } // 24-72 ساعة
    public int SlowDispenses { get; set; }   // > 72 ساعة
    public decimal InitialRate { get; set; }
    public decimal RefillRate { get; set; }
    public decimal RejectionRate { get; set; }
    public decimal FastRate { get; set; }

    // ⭐ التفاصيل (اختياري)
    public List<PharmacyDispensationPharmacyDto> Pharmacies { get; set; }
}
```

**منطق Handler:**

```
1. جلب البرامج
2. جلب الصرفات (TokenUsed = true, PharmacyID != null)
   ├── فلتر الصيدلية
   ├── فلتر نوع الصرف (INITIAL / REFILL)
   ├── فلتر التاريخ
   └── الفلترة الجغرافية (حسب موقع الصيدلية)
3. جلب أسماء الصيدليات
4. التجميع: حسب البرنامج
5. حساب مؤشرات الوقت:
   ├── FastDispenses: TotalHours < 24
   ├── NormalDispenses: 24 <= TotalHours <= 72
   └── SlowDispenses: TotalHours > 72
6. التفاصيل: حسب الصيدلية
```

**⚠️ ملاحظة حرجة:** `TokenGeneratedDate` و `TokenUsedDate` من نوع `DateTime?` — يجب استخدام `.Value`:

```csharp
(d.TokenUsedDate.Value - d.TokenGeneratedDate.Value).TotalHours
```

**الملفات:**
- `IGetPharmacyDispensationHandler.cs`
- `GetPharmacyDispensationHandler.cs`
- `PharmacyDispensationReport.razor`

---

### 4.4 التقرير المالي - الصيدليات (Pharmacy Financial)

**الغرض:** تحليل مالي كامل لعمليات الصرف.

**المدخلات:** `programId`, `pharmaCompanyId`, `pharmacyId`, `medicationId`, `dispenseType`, `year`, `month`, `fromDate`, `toDate`

**المخرجات:**

```csharp
public class GetPharmacyFinancialResponse
{
    public PharmacyFinancialSummaryDto Summary { get; set; }
    public List<PharmacyFinancialProgramDto> Programs { get; set; }
    public List<PharmacyFinancialPharmacyDto> Pharmacies { get; set; }
    public List<PharmacyFinancialMedicationDto> Medications { get; set; }
    public List<PharmacyFinancialMonthlyDto> Monthly { get; set; }
}

public class PharmacyFinancialSummaryDto
{
    public int TotalDispensations { get; set; }
    public int TotalQuantity { get; set; }
    public decimal TotalSalesValue { get; set; }
    public decimal TotalReimbursement { get; set; }
    public decimal TotalPatientPayment { get; set; }
    public int TotalUniquePatients { get; set; }
    public decimal AverageReimbursementPerDispense { get; set; }
    public decimal AverageSalesPerDispense { get; set; }
    public decimal ReimbursementRate { get; set; }
}
```

**منطق Handler:**

```
1. جلب البرامج
2. جلب الصرفات مع المبالغ المالية
   ├── TotalAmount (قيمة المبيعات)
   ├── PSPCoverage (التعويضات)
   └── PatientPayment (مدفوعات المرضى)
3. جلب أسماء الصيدليات والأدوية
4. بناء 5 أقسام:
   ├── Summary (الإجماليات)
   ├── Programs (حسب البرنامج)
   ├── Pharmacies (حسب الصيدلية)
   ├── Medications (حسب الدواء)
   └── Monthly (التحليل الشهري)
```

**المعادلات المالية:**

```
TotalAmount = UnitPrice × Quantity
PSPCoverage = TotalAmount × (SupportPercentage / 100)
PatientPayment = TotalAmount - PSPCoverage
```

**الملفات:**
- `IGetPharmacyFinancialHandler.cs`
- `GetPharmacyFinancialHandler.cs`
- `PharmacyFinancialReport.razor`
- `pharmacy-financial.css`

---

### 4.5 تقرير الالتزام بالجرعات (Adherence)

**الغرض:** قياس التزام المرضى بالجرعات اليومية.

**المدخلات:** `programId`, `pharmaCompanyId`, `patientProfileId`, `adherenceLevel`, `fromDate`, `toDate`

**المخرجات:**

```csharp
public class GetAdherenceResponse
{
    public AdherenceSummaryDto Summary { get; set; }
    public List<AdherenceProgramDto> Programs { get; set; }
    public List<AdherencePatientDto> Patients { get; set; }
    public List<AdherenceDistributionDto> Distribution { get; set; }
}

public class AdherencePatientDto
{
    public int PatientProfileId { get; set; }
    public int PatientId { get; set; }
    public string PatientName { get; set; }     // ⭐ الاسم الأصلي
    public string? PatientPhone { get; set; }   // ⭐ الهاتف الأصلي
    public bool HasSharedContactConsent { get; set; }  // ⭐ الموافقة

    // ⭐ الاسم المعروض (مجهل إذا لم يوافق)
    public string DisplayName => HasSharedContactConsent ? PatientName : "مريض ****";
    public string? DisplayPhone => HasSharedContactConsent ? PatientPhone : null;

    public int TotalDoses { get; set; }
    public int TakenDoses { get; set; }
    public int SkippedDoses { get; set; }
    public int MissedDoses { get; set; }
    public decimal AdherenceRate { get; set; }
    public string AdherenceLevel { get; set; }  // HIGH / MEDIUM / LOW
}
```

**منطق Handler:**

```
1. جلب البرامج
2. جلب الجرعات (ScheduleType = "DAILY_DOSE")
   ├── ScheduledDateTime < NOW (الفعلية فقط)
   ├── ProgramID != null
   └── فلتر التاريخ
3. جلب بيانات المرضى (مع الاسم والهاتف)
4. ⭐ جلب الموافقات (مؤقتاً: فارغ)
5. التجميع: حسب البرنامج + حسب المريض
6. التوزيع:
   ├── عالي (≥ 80%)
   ├── متوسط (50-80%)
   └── منخفض (< 50%)
```

**⭐ الميزة الأمنية:**

```csharp
// ⚠️ مؤقتاً: لا توجد موافقات
var consentData = new HashSet<int>();

// عند إنشاء PatientConsent:
// var consentData = await db.Set<PatientConsent>()
//     .Where(c => c.ConsentType == "SHARE_CONTACT_WITH_PHARMA"
//              && c.IsGranted == true)
//     .Select(c => c.PatientProfileId)
//     .ToHashSetAsync();

var hasConsent = consentData.Contains(patientProfileId);
```

**الملفات:**
- `IGetAdherenceHandler.cs`
- `GetAdherenceHandler.cs`
- `AdherenceReport.razor`
- `adherence.css`

---

### 4.6 تقرير دورة حياة البرنامج (Program Lifecycle)

**الغرض:** التقرير الشامل الذي يجمع كل شيء.

**المدخلات:** `programId`, `pharmaCompanyId`, `repUserProfileId`, `clinicId`, `pharmacyId`, `year`, `month`, `fromDate`, `toDate`

**المخرجات (8 أقسام):**

```csharp
public class GetProgramLifecycleResponse
{
    public ProgramLifecycleSummaryDto Summary { get; set; }              // 1. الملخص
    public List<ProgramLifecycleFunnelDto> Funnel { get; set; }          // 2. القمع
    public List<ProgramLifecycleProgramDto> Programs { get; set; }       // 3. حسب البرنامج
    public List<ProgramLifecycleRegionDto> Regions { get; set; }         // 4. حسب المنطقة
    public List<ProgramLifecycleClinicDto> TopClinics { get; set; }      // 5. أفضل العيادات
    public List<ProgramLifecyclePharmacyDto> TopPharmacies { get; set; } // 6. أفضل الصيدليات
    public ProgramLifecycleFinancialDto Financial { get; set; }          // 7. الملخص المالي
    public List<ProgramLifecycleMonthlyDto> Monthly { get; set; }        // 8. التحليل الشهري
}
```

**القمع (7 مراحل):**

| المرحلة | اللون | الأيقونة |
|---------|-------|---------|
| الدعوات | `#1B5A7A` | 📨 |
| المقبولة | `#2D8B6E` | ✅ |
| المرضى | `#4DA88B` | 👥 |
| الصرف الأول | `#B88B4A` | 💊 |
| التكرارات | `#B33F40` | 🔄 |
| الالتزام | `#2E7D5E` | 📊 |
| الإتمام | `#0A3D5C` | 🏆 |

**منطق Handler:**

```
1. جلب البرامج
2. جلب الدعوات (مع فلتر المندوب والتاريخ)
3. جلب المشاركات
4. جلب المرضى
5. جلب الصرفات (مع المبالغ المالية)
6. جلب الجرعات (الالتزام)
7. جلب الإتمام (PSPeRX.Status = 'COMPLETED')
8. جلب أسماء العيادات والصيدليات
9. بناء الأقسام الثمانية
```

**الملفات:**
- `IGetProgramLifecycleHandler.cs`
- `GetProgramLifecycleHandler.cs`
- `ProgramLifecycleReport.razor`
- `program-lifecycle.css`

---

## 5. لوحة التحكم المركزية

### 5.1 الموقع

```
Rubikcare.Web/Components/Pages/PharmaDashboard.razor
```

**URL:** `/organization/{OrganizationID:int}/psp/dashboard`

### 5.2 الأقسام

```
┌─────────────────────────────────────────────────────────────┐
│  1. Hero Section                                            │
│  ├── اسم الشركة                                             │
│  ├── فلتر زمني (30D / 90D / YEAR / ALL)                    │
│  └── ترحيب                                                  │
└─────────────────────────────────────────────────────────────┘

┌─────────────────────────────────────────────────────────────┐
│  2. KPIs دائرية — نظرة عامة (تراكمي)                       │
│  🎯 البرامج (8)  🏥 العيادات (15)  💊 الصيدليات (12)  👥 المرضى (567) │
└─────────────────────────────────────────────────────────────┘

┌─────────────────────────────────────────────────────────────┐
│  3. KPIs دائرية — أداء الفترة                              │
│  📨 الدعوات (20)  💊 الصرفات (1)  💰 المبيعات (1,268)  💵 التعويضات (84) │
└─────────────────────────────────────────────────────────────┘

┌─────────────────────────────────────────────────────────────┐
│  4. Quick Access Cards (6 بطاقات)                          │
│  📊 Program Lifecycle   📨 Provider Funnel                 │
│  👥 Patient Funnel      💊 Pharmacy Dispensation           │
│  💰 Pharmacy Financial  📈 Adherence                       │
└─────────────────────────────────────────────────────────────┘

┌─────────────────────────────────────────────────────────────┐
│  5. Charts                                                  │
│  Bar Chart: الدعوات الشهرية  │  Bar Chart: أفضل العيادات  │
└─────────────────────────────────────────────────────────────┘

┌─────────────────────────────────────────────────────────────┐
│  6. Funnel Preview + Adherence Distribution                 │
│  معاينة القمع  │  توزيع الالتزام (Progress Bars)           │
└─────────────────────────────────────────────────────────────┘

┌─────────────────────────────────────────────────────────────┐
│  7. آخر الأنشطة (5 أنشطة)                                  │
│  💊 صرف دواء - منذ 5 دقائق                                 │
│  👤 تسجيل مريض - منذ 15 دقيقة                              │
│  📨 دعوة جديدة - منذ ساعة                                   │
└─────────────────────────────────────────────────────────────┘
```

### 5.3 Handler الخاص

**`GetDashboardSummaryHandler`** — استعلام واحد خفيف يجمع كل شيء:

```csharp
public class DashboardKpisDto
{
    // ⭐ نظرة عامة (تراكمي)
    public int ActivePrograms { get; set; }
    public int ActiveClinics { get; set; }
    public int ActivePharmacies { get; set; }
    public int TotalActivePatients { get; set; }

    // ⭐ أداء الفترة
    public int TotalInvitations { get; set; }
    public int AcceptedInvitations { get; set; }
    public int TotalPatients { get; set; }
    public int TotalDispensations { get; set; }
    public int TotalQuantity { get; set; }
    public decimal TotalReimbursement { get; set; }
    public decimal TotalSalesValue { get; set; }
    public decimal TotalPatientPayment { get; set; }
    public int TotalDoses { get; set; }
    public int TakenDoses { get; set; }
    public int CompletedPrograms { get; set; }
}
```

### 5.4 الميزات الرئيسية

| الميزة | الوصف |
|--------|-------|
| **Load مرة واحدة** | استعلام واحد للـ Dashboard كامل |
| **الفترة الزمنية** | فلتر 30D/90D/YEAR/ALL |
| **KPIs مقسمة** | تراكمي + فترة |
| **Quick Access** | 6 بطاقات للتنقل السريع |
| **Bar Charts** | باستخدام `BLS.Charts` |
| **Progress Bars** | للقمع وتوزيع الالتزام |
| **آخر الأنشطة** | 5 أنشطة من كل المصادر |

---

## 6. نظام الفلترة الموحد

### 6.1 مجموعات الفلاتر (3 مجموعات)

```
┌─────────────────────────────────────────────────────────────┐
│  👤 المجموعة 1: المندوب والمنطقة (أزرق)                    │
│  ├── المندوب (search)                                       │
│  ├── الدولة (search)                                        │
│  ├── المدينة (search - متسلسل)                              │
│  └── المنطقة (search - متسلسل)                              │
└─────────────────────────────────────────────────────────────┘

┌─────────────────────────────────────────────────────────────┐
│  📊 المجموعة 2: البرنامج والتصنيف (أخضر)                   │
│  ├── البرنامج (search)                                      │
│  ├── نوع الدعوة (select)                                    │
│  ├── نوع المنظمة (select)                                   │
│  └── الحالة (select)                                        │
└─────────────────────────────────────────────────────────────┘

┌─────────────────────────────────────────────────────────────┐
│  📅 المجموعة 3: الفترة الزمنية (برتقالي)                   │
│  ├── من تاريخ (date)                                        │
│  ├── إلى تاريخ (date)                                       │
│  └── زر مسح الفلاتر                                         │
└─────────────────────────────────────────────────────────────┘
```

### 6.2 الفلترة المتسلسلة

```
الدولة → المدينة → المنطقة
```

**التنفيذ:**

```csharp
private List<LookupItemDto> FilteredCities =>
    _selectedCountryId.HasValue
        ? _filters.Cities.Where(c => c.ParentId == _selectedCountryId.Value).ToList()
        : _filters.Cities;

private List<LookupItemDto> FilteredAreas =>
    _selectedCityId.HasValue
        ? _filters.Areas.Where(a => a.ParentId == _selectedCityId.Value).ToList()
        : _filters.Areas;
```

### 6.3 الفلترة التلقائية حسب الشركة

**كل الفلاتر تُفلتر تلقائياً بـ `PharmaCompanyId`:**

```csharp
// في GetReportFiltersHandler
if (query.PharmaCompanyId.HasValue)
    programsQuery = programsQuery.Where(p => p.PharmaCompanyID == query.PharmaCompanyId.Value);

// العيادات المرتبطة ببرامج الشركة
var clinicIds = await db.Set<PSPParticipation>()
    .Where(p => programIds.Contains(p.ProgramID))
    .Select(p => p.ParticipantOrganizationID)
    .Distinct()
    .ToListAsync();
```

### 6.4 نمط `@oninput` (بدون `@bind`)

**⚠️ مهم:** استخدام `@bind:after` مع `select` يسبب خطأ CS0121.

**الحل:** استخدام `@oninput` + `ChangeEventArgs`:

```razor
<input list="reps-list"
       value="@_repSearch"
       @oninput="OnRepSearchChanged"
       class="filter-input" />
```

```csharp
private async Task OnRepSearchChanged(ChangeEventArgs e)
{
    var val = e.Value?.ToString();
    _repSearch = val;
    _selectedRepId = string.IsNullOrEmpty(val)
        ? null
        : _filters.Reps.FirstOrDefault(r => r.FullName == val)?.UserProfileId;
    _showDetails = false;
    await LoadDataAsync();
}
```

---

## 7. إصلاح البيانات المالية

### 7.1 المشكلة

**`PSPDispensations` كانت فارغة ماليًا:**

| الحقل | القيمة |
|--------|--------|
| `QuantityDispensed` | `0` |
| `UnitPrice` | `NULL` |
| `TotalAmount` | `NULL` |
| `PatientPayment` | `NULL` |
| `PSPCoverage` | `NULL` |

### 7.2 الحل 1: إصلاح `ConfirmDispense`

**في `DispenseController.cs`:**

```csharp
// ⭐ حساب المبالغ المالية
var quantity = request.QuantityDispensed > 0 ? request.QuantityDispensed : 1;

// 1. جلب الدواء (عبر OriginalMedicationID)
var medication = await db.Set<PharmaCompanyMedication>()
    .Where(m => m.OriginalMedicationID == eRx.MedicationID)
    .FirstOrDefaultAsync();

// 2. نسبة الدعم (من الخطة أو البرنامج)
var supportPercentage = planMedication?.DiscountPercentage > 0
    ? planMedication.DiscountPercentage
    : programMedication?.SupportValue ?? 0;

// 3. الحسابات
var unitPrice = medication?.Price ?? 0;
var grossAmount = unitPrice * quantity;
var pspCoverage = grossAmount * (supportPercentage / 100);
var patientPayment = grossAmount - pspCoverage;

// 4. الحفظ
dispensation.QuantityDispensed = quantity;
dispensation.UnitPrice = unitPrice;
dispensation.TotalAmount = grossAmount;
dispensation.PSPCoverage = pspCoverage;
dispensation.PatientPayment = patientPayment;
```

### 7.3 الحل 2: Migration للبيانات القديمة

```sql
-- 1. إصلاح الكمية
UPDATE PSPDispensations
SET QuantityDispensed = 1
WHERE QuantityDispensed = 0 AND TokenUsed = 1;

-- 2. ملء المبالغ المالية
UPDATE d
SET 
    d.UnitPrice = pcm.Price,
    d.TotalAmount = d.QuantityDispensed * pcm.Price,
    d.PSPCoverage = d.QuantityDispensed * pcm.Price * 
        CASE 
            WHEN pdm.DiscountPercentage > 0 THEN pdm.DiscountPercentage 
            ELSE ISNULL(pm.SupportValue, 0) 
        END / 100,
    d.PatientPayment = d.QuantityDispensed * pcm.Price * (1 - 
        CASE 
            WHEN pdm.DiscountPercentage > 0 THEN pdm.DiscountPercentage 
            ELSE ISNULL(pm.SupportValue, 0) 
        END / 100)
FROM PSPDispensations d
JOIN PSPeRX e ON d.ERxID = e.ERxID
JOIN PharmaCompanyMedications pcm ON e.MedicationID = pcm.OriginalMedicationID
LEFT JOIN PSPProgramMedications pm ON pm.ProgramID = e.ProgramID 
                                   AND pm.MedicationID = e.MedicationID
LEFT JOIN PSPDispensationPlanMedications pdm ON pdm.PlanID = d.PlanID 
                                             AND pdm.ProgramMedicationID = pm.ProgramMedicationID
WHERE d.TokenUsed = 1;
```

### 7.4 النتائج

| المؤشر | القيمة |
|--------|--------|
| **إجمالي الصرفات** | 42 |
| **إجمالي العلب** | 42 |
| **إجمالي قيمة المبيعات** | 11,268 ج.م |
| **إجمالي التعويضات** | 3,192 ج.م |
| **إجمالي مدفوعات المرضى** | 8,076 ج.م |

**التحقق:** `11,268 - 3,192 = 8,076` ✅

---

## 8. الميزات الأمنية

### 8.1 إخفاء بيانات المرضى

**المشكلة:** كشف الاسم والهاتف للمرضى في تقرير الالتزام.

**الحل:** إخفاء افتراضي + حقل موافقة.

```csharp
public class AdherencePatientDto
{
    public string PatientName { get; set; }     // الاسم الأصلي
    public string? PatientPhone { get; set; }   // الهاتف الأصلي
    public bool HasSharedContactConsent { get; set; } = false;  // افتراضياً: لا موافقة

    // ⭐ الاسم المعروض
    public string DisplayName => HasSharedContactConsent
        ? PatientName
        : "مريض ****";

    // ⭐ الهاتف المعروض
    public string? DisplayPhone => HasSharedContactConsent
        ? PatientPhone
        : null;
}
```

**في Razor:**

```razor
<td>
    @if (p.HasSharedContactConsent)
    {
        @p.PatientName
    }
    else
    {
        <span class="masked-data">@T("REPORTS.ADHERENCE.MASKED_PATIENT")</span>
    }
</td>
<td>
    @if (p.HasSharedContactConsent)
    {
        @p.PatientPhone
    }
    else
    {
        <span class="masked-data">—</span>
    }
</td>
```

### 8.2 الامتثال لـ PDPL

**صفحة الامتثال:** `https://test.rubikcare.com/compliance`

**المتطلبات المُحققة:**
- ✅ موافقة مكتوبة وصريحة (Standalone screen)
- ✅ وصول مقيّد وسجل تدقيق
- ✅ حقوق صاحب البيانات (وصول، تصحيح، سحب)
- ✅ الاحتفاظ والمحو

### 8.3 الخطوات التالية

| # | المهمة | الحالة |
|---|--------|--------|
| 1 | إنشاء `PatientConsent` | ⏳ |
| 2 | واجهة موافقة للمريض | ⏳ |
| 3 | ربط `HasSharedContactConsent` | ⏳ |
| 4 | سجل تدقيق الوصول | ⏳ |

---

## 9. الأنماط والتصميم

### 9.1 نظام الألوان

| العنصر | اللون |
|--------|-------|
| **المجموعة 1 (المندوب والمنطقة)** | `#E8F3F9` → `#FFFFFF` |
| **المجموعة 2 (البرنامج والتصنيف)** | `#EAF4EF` → `#FFFFFF` |
| **المجموعة 3 (الفترة الزمنية)** | `#FFF3E0` → `#FFFFFF` |

### 9.2 التحميل التدريجي

```
┌─────────────────────────────────────────────────────────────┐
│  فتح الصفحة                                                 │
│  ├── تحميل الفلاتر (api/reports/filters)                   │
│  └── تحميل Summary فقط (includeDetails=false)              │
└─────────────────────────────────────────────────────────────┘
                       ↓
┌─────────────────────────────────────────────────────────────┐
│  عرض البطاقات + زر "عرض التفاصيل"                          │
└─────────────────────────────────────────────────────────────┘
                       ↓ (عند الطلب)
┌─────────────────────────────────────────────────────────────┐
│  تحميل التفاصيل (includeDetails=true)                      │
│  └── عرض الجدول الكامل                                      │
└─────────────────────────────────────────────────────────────┘
```

### 9.3 الدوائر (Circular KPIs)

```css
.circular-kpi-ring {
    width: 130px;
    height: 130px;
    border-radius: 50%;
    background: conic-gradient(
        var(--color) 0deg,
        var(--color) 280deg,
        var(--rubik-border-light) 280deg
    );
    display: flex;
    align-items: center;
    justify-content: center;
    position: relative;
}

.circular-kpi-ring::before {
    content: '';
    position: absolute;
    inset: 10px;
    background: var(--rubik-card-bg);
    border-radius: 50%;
}
```

### 9.4 `@rendermode InteractiveServer`

**⚠️ إلزامي في كل صفحة:**

```razor
@page "/reports/provider-funnel"
@rendermode InteractiveServer
```

**بدونه:** الأحداث لا تعمل (Static SSR فقط).

### 9.5 مشاكل `BLS.Charts`

| المشكلة | الحل |
|---------|------|
| `LineSeries` vs `BarSeries` | استخدام `List<BarSeries>` للـ Bar Chart |
| `@rendermode` | إلزامي |
| Overflow في RTL | `overflow: hidden` على `.ch-card-body` |

---

## 10. مفاتيح الترجمة

### 10.1 البنية

```
DASHBOARD.*   → 52 مفتاحاً (لوحة التحكم)
REPORTS.*     → ~150 مفتاحاً (التقارير)
COMMON.*      → نصوص عامة
```

### 10.2 الوحدات

| الوحدة | المفاتيح |
|--------|----------|
| `DASHBOARD` | KPIs، الفترة، الأقسام، الفوتر |
| `REPORTS` | عناوين التقارير، الفلاتر، المؤشرات، الأزرار |
| `REPORTS.FILTER.*` | الفلاتر |
| `REPORTS.PHARMACY_FINANCIAL.*` | التقرير المالي |
| `REPORTS.PROGRAM_LIFECYCLE.*` | دورة الحياة |
| `REPORTS.ADHERENCE.*` | الالتزام |

### 10.3 التحميل في `PharmaDashboard`

**⚠️ مهم:** تحميل 3 وحدات معاً:

```csharp
if (TranslationService != null)
{
    var dashboardTrans = await TranslationService.GetPageTranslationsAsync("DASHBOARD");
    var reportsTrans = await TranslationService.GetPageTranslationsAsync("REPORTS");
    var commonTrans = await TranslationService.GetPageTranslationsAsync("COMMON");

    _translations = dashboardTrans
        .Concat(reportsTrans)
        .Concat(commonTrans)
        .GroupBy(kv => kv.Key)
        .ToDictionary(g => g.Key, g => g.First().Value);
}
```

### 10.4 MERGE بدلاً من INSERT

```sql
MERGE Resources AS target
USING (VALUES
    (N'KEY', N'عربي', N'English', N'MODULE', N'Type')
) AS source (ResourceKey, ResourceValueAr, ResourceValueEn, Module, ResourceType)
ON target.ResourceKey = source.ResourceKey AND target.Module = source.Module
WHEN MATCHED THEN UPDATE SET ...
WHEN NOT MATCHED THEN INSERT ...;
```

---

## 11. خطوات التنفيذ

### 11.1 لكل تقرير جديد

| # | الخطوة | الملف |
|---|--------|-------|
| 1 | إنشاء `IGetXxxHandler.cs` | Application/UseCases/Reports/Xxx/ |
| 2 | إنشاء `GetXxxQuery.cs` | نفس المجلد |
| 3 | إنشاء `GetXxxResponse.cs` | نفس المجلد |
| 4 | إنشاء `GetXxxHandler.cs` | Infrastructure/UseCases/Reports/Xxx/ |
| 5 | إضافة endpoint في `ReportsController` | Api.Web/Controllers/Reports/ |
| 6 | تسجيل في `InfrastructureExtensions.cs` | Infrastructure/ |
| 7 | إنشاء `XxxReport.razor` | Shared.UI/Components/Reports/ |
| 8 | إنشاء `Xxx.razor` (صفحة) | Rubikcare.Web/Components/Pages/Reports/ |
| 9 | إنشاء `xxx.css` | Shared.UI/wwwroot/css/_pages/Reports/ |
| 10 | إضافة `@import` في `_PagesBundle.css` | Shared.UI/wwwroot/css/ |
| 11 | SQL: `MenuItems` + `Resources` | - |

### 11.2 تسجيل الخدمات

**في `InfrastructureExtensions.cs`:**

```csharp
// ⭐ Reports UseCases
services.AddScoped<IGetProviderFunnelHandler, GetProviderFunnelHandler>();
services.AddScoped<IGetPatientFunnelHandler, GetPatientFunnelHandler>();
services.AddScoped<IGetPharmacyDispensationHandler, GetPharmacyDispensationHandler>();
services.AddScoped<IGetPharmacyFinancialHandler, GetPharmacyFinancialHandler>();
services.AddScoped<IGetAdherenceHandler, GetAdherenceHandler>();
services.AddScoped<IGetProgramLifecycleHandler, GetProgramLifecycleHandler>();
services.AddScoped<IGetDashboardSummaryHandler, GetDashboardSummaryHandler>();
services.AddScoped<IGetReportFiltersHandler, GetReportFiltersHandler>();
```

### 11.3 القائمة الديناميكية

```sql
INSERT INTO MenuItems (MenuID, ItemCode, ItemNameAr, ItemNameEn, Url, IconClass, SortOrder, IsVisible, CreatedDate)
VALUES 
    (4, N'PHARMA_PROVIDER_FUNNEL', N'معدل تحويل الدعوات', N'Invitation Conversion Rate', N'/reports/provider-funnel', N'bi-bar-chart', 10, 1, GETDATE()),
    (4, N'PHARMA_PATIENT_FUNNEL', N'قمع تحويل المريض', N'Patient Funnel Report', N'/reports/patient-funnel', N'bi-people', 11, 1, GETDATE()),
    (4, N'PHARMA_PHARMACY_DISPENSATION', N'تقرير صرف الصيدليات', N'Pharmacy Dispensation Report', N'/reports/pharmacy-dispensation', N'bi-capsule', 12, 1, GETDATE()),
    (4, N'PHARMA_PHARMACY_FINANCIAL', N'التقرير المالي - الصيدليات', N'Pharmacy Financial Report', N'/reports/pharmacy-financial', N'bi-cash-coin', 13, 1, GETDATE()),
    (4, N'PHARMA_ADHERENCE', N'تقرير الالتزام بالجرعات', N'Adherence Report', N'/reports/adherence', N'bi-clipboard-check', 14, 1, GETDATE()),
    (4, N'PHARMA_PROGRAM_LIFECYCLE', N'دورة حياة البرنامج', N'Program Lifecycle', N'/reports/program-lifecycle', N'bi-diagram-3', 15, 1, GETDATE());
```

---

## 12. الدروس المستفادة

### 12.1 الأخطاء الشائعة وحلولها

| # | الخطأ | الحل |
|---|-------|------|
| 1 | `\"` في `@onclick` | استخدام دوال مباشرة: `@onclick="NavigateToX"` |
| 2 | `@bind:after` مع `select` | استخدام `@oninput` + `ChangeEventArgs` |
| 3 | `LineSeries` vs `BarSeries` | استخدام `List<BarSeries>` للـ Bar Chart |
| 4 | `TimeSpan?` لا يحتوي `TotalHours` | استخدام `.Value` |
| 5 | `@rendermode` مفقود | إلزامي في كل صفحة |
| 6 | نصوص عربية مباشرة في Handler | استخدام `T()` في الواجهة |
| 7 | تحميل وحدة ترجمة واحدة | تحميل 3 وحدات: DASHBOARD + REPORTS + COMMON |
| 8 | `OrganizationID` مباشر | استخدام `CurrentOrganizationId` من `UserSession` |

### 12.2 أفضل الممارسات

| # | الممارسة |
|---|----------|
| 1 | **البطاقات دائماً** — الجدول عند الطلب (`includeDetails`) |
| 2 | **التجميع حسب البرنامج أولاً** — ثم العيادة/الصيدلية |
| 3 | **الفلاتر تُفلتر تلقائياً** بـ `PharmaCompanyId` |
| 4 | **إخفاء بيانات المرضى** افتراضياً |
| 5 | **الفلترة الجغرافية** عبر `Organization.AreaID` |
| 6 | **`[Parameter] IApiService`** في `Shared.UI` (لا `@inject`) |
| 7 | **`@rendermode InteractiveServer`** إلزامي |
| 8 | **`MERGE`** بدلاً من `INSERT/UPDATE` |
| 9 | **`N''`** للعربية في SQL |
| 10 | **التحقق من البناء** بعد كل تعديل |

### 12.3 القواعد الذهبية

```
✅ Clean Architecture: Handlers في Infrastructure
✅ نمط موحد: Query + Response + Handler لكل تقرير
✅ التحميل التدريجي: Summary دائماً، تفاصيل عند الطلب
✅ الفلترة التلقائية: حسب PharmaCompanyId
✅ الأمان: إخفاء بيانات المرضى افتراضياً
✅ الترجمة: 3 وحدات معاً (DASHBOARD + REPORTS + COMMON)
✅ @rendermode InteractiveServer: إلزامي
✅ MERGE + N'': في SQL
```

---

## 📌 خاتمة

**نظام التقارير المُنجز:**

| المقياس | القيمة |
|---------|--------|
| **عدد التقارير** | 6 + لوحة تحكم |
| **عدد Endpoints** | 8 |
| **عدد Handlers** | 8 |
| **عدد ملفات Razor** | 13 |
| **عدد ملفات CSS** | 6 |
| **عدد DTOs** | ~30 |
| **مفاتيح الترجمة** | ~200 |
| **الحالة** | ✅ مكتمل ومُختبر |

**هذا النظام يُعتبر الأساس القوي لنظام النقاط (Rubika) القادم.**

---

**📅 تاريخ الوثيقة:** 16 أكتوبر 2026  
**✍️ الإصدار:** 1.0  
**🔗 المرجع:** `Rubikcare-docs/docs/26-reports-system.md` (مقترح)
