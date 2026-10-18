
# 07 — REPORTS (نظام التقارير)

**الإصدار:** 2.0 (Unified Edition)
**آخر تحديث:** 17 أكتوبر 2026
**الحالة:** ✅ مكتمل ومُختبر
**الجمهور:** فريق التطوير + مديرو المشروع + AI Assistants

---

## 📌 الجزء صفر: الفلسفة والرؤية

### 0.1 الهدف

نظام تقارير متكامل لشركة الأدوية، يتيح:

- **قياس أداء البرامج** من الدعوة إلى الإتمام.
- **تحليل المندوبين والعيادات والصيدليات**.
- **متابعة الالتزام الدوائي** للمرضى.
- **التقارير المالية والتعويضات**.
- **نظرة شاملة** عبر لوحة تحكم مركزية.

### 0.2 المبادئ الأساسية

| المبدأ | التطبيق |
|:---|:---|
| **Clean Architecture** | Handlers في Infrastructure، DTOs في Application |
| **نمط موحد** | كل تقرير يتبع نفس البنية |
| **الأداء** | تحميل تدريجي (Summary دائماً، تفاصيل عند الطلب) |
| **الأمان** | إخفاء بيانات المرضى افتراضياً |
| **التكامل** | Funnel + Financial + Adherence في كل تقرير |

### 0.3 الجمهور المستهدف

- **مديرو البرامج** في شركات الأدوية.
- **المندوبون** لمتابعة أدائهم.
- **المحاسبون** للتقارير المالية.
- **الفريق الطبي** لمتابعة الالتزام.

---

## 📌 الجزء الأول: الهيكل المعماري

### 1.1 الطبقات

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

### 1.2 مواقع الملفات

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
    ├── PatientFunnel/
    ├── PharmacyDispensation/
    ├── PharmacyFinancial/
    ├── Adherence/
    ├── ProgramLifecycle/
    ├── Dashboard/
    └── Filters/

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

## 📌 الجزء الثاني: البنية التحتية المشتركة

### 2.1 `ReportFilterBase.cs`

```csharp
public class ReportFilterBase
{
    public int? PharmaCompanyId { get; set; }
    public int? ProgramId { get; set; }
    public int? RepUserProfileId { get; set; }
    public int? ClinicId { get; set; }
    public int? PharmacyId { get; set; }
    public string? SourceType { get; set; }
    public string? InvitedOrganizationType { get; set; }
    public string? Status { get; set; }
    public DateTime? FromDate { get; set; }
    public DateTime? ToDate { get; set; }
    public List<int>? CountryIds { get; set; }
    public List<int>? CityIds { get; set; }
    public List<int>? AreaIds { get; set; }
    public string? SearchTerm { get; set; }
}
```

### 2.2 DTOs المشتركة

| الملف | الوصف |
|:---|:---|
| `PharmaCompanyDto` | شركة الأدوية |
| `RepDto` | المندوب |
| `OrganizationLookupDto` | العيادة/الصيدلية |
| `LookupItemDto` | الدول/المدن/المناطق |

### 2.3 `GetReportFiltersHandler`

**يوفر كل قوائم الفلاتر — مع فلترة تلقائية حسب `PharmaCompanyId`.**

---

## 📌 الجزء الثالث: التقارير الستة بالتفصيل

### 3.1 تقرير معدل تحويل الدعوات (Provider Funnel)

**الغرض:** قياس أداء المندوبين في دعوة الأطباء والصيدليات.

**المدخلات:** `programId`, `pharmaCompanyId`, `repUserProfileId`, `sourceType`, `invitedOrganizationType`, `status`, `fromDate`, `toDate`.

**المخرجات:**

```csharp
public class GetProviderFunnelResponse
{
    public List<ProviderFunnelProgramDto> Programs { get; set; }
    public ProviderFunnelSummaryDto Summary { get; set; }
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
   └── الفلترة الجغرافية
3. جلب المشاركات
4. التجميع: Program → SourceType → InvitedOrganizationType
5. حساب ActualParticipations
6. حساب النسب
```

### 3.2 تقرير قمع تحويل المريض (Patient Funnel)

**الغرض:** قياس رحلة المريض من الدعوة إلى الاشتراك والصرف.

**منطق Handler:**

```
1. جلب البرامج
2. جلب الدعوات (DOCTOR_TO_PATIENT + DOCTOR_REFERRAL)
3. جلب المرضى المسجلين (JOIN مع PSPParticipation)
4. جلب الصرفات
5. جلب إتمام البرامج
6. التجميع: حسب البرنامج
7. التفاصيل: حسب العيادة
```

### 3.3 تقرير صرف الصيدليات (Pharmacy Dispensation)

**الغرض:** تحليل أداء الصيدليات في عمليات الصرف (تشغيلي).

**مؤشرات الوقت:**

| المؤشر | الشرط |
|:---|:---|
| `FastDispenses` | `TotalHours < 24` |
| `NormalDispenses` | `24 <= TotalHours <= 72` |
| `SlowDispenses` | `TotalHours > 72` |

### 3.4 التقرير المالي - الصيدليات (Pharmacy Financial)

**الغرض:** تحليل مالي كامل لعمليات الصرف.

**المعادلات المالية:**

```
TotalAmount = UnitPrice × Quantity
PSPCoverage = TotalAmount × (SupportPercentage / 100)
PatientPayment = TotalAmount - PSPCoverage
```

### 3.5 تقرير الالتزام بالجرعات (Adherence)

**الغرض:** قياس التزام المرضى بالجرعات اليومية.

**⭐ الميزة الأمنية — إخفاء بيانات المرضى:**

```csharp
public class AdherencePatientDto
{
    public string PatientName { get; set; }
    public string? PatientPhone { get; set; }
    public bool HasSharedContactConsent { get; set; } = false;

    public string DisplayName => HasSharedContactConsent
        ? PatientName
        : "مريض ****";

    public string? DisplayPhone => HasSharedContactConsent
        ? PatientPhone
        : null;
}
```

### 3.6 تقرير دورة حياة البرنامج (Program Lifecycle)

**الغرض:** التقرير الشامل الذي يجمع كل شيء.

**8 أقسام:**

1. `Summary` (الملخص).
2. `Funnel` (القمع — 7 مراحل).
3. `Programs` (حسب البرنامج).
4. `Regions` (حسب المنطقة).
5. `TopClinics` (أفضل العيادات).
6. `TopPharmacies` (أفضل الصيدليات).
7. `Financial` (الملخص المالي).
8. `Monthly` (التحليل الشهري).

---

## 📌 الجزء الرابع: لوحة التحكم المركزية

### 4.1 الموقع

`Rubikcare.Web/Components/Pages/PharmaDashboard.razor`

**URL:** `/organization/{OrganizationID:int}/psp/dashboard`

### 4.2 الأقسام

1. **Hero Section** — اسم الشركة + فلتر زمني (30D/90D/YEAR/ALL).
2. **KPIs دائرية — نظرة عامة** (تراكمي) — البرامج، العيادات، الصيدليات، المرضى.
3. **KPIs دائرية — أداء الفترة** — الدعوات، الصرفات، المبيعات، التعويضات.
4. **Quick Access Cards (6)** — Program Lifecycle, Provider Funnel, Patient Funnel, Pharmacy Dispensation, Pharmacy Financial, Adherence.
5. **Charts** — Bar Chart: الدعوات الشهرية + أفضل العيادات.
6. **Funnel Preview + Adherence Distribution**.
7. **آخر الأنشطة (5)**.

### 4.3 `GetDashboardSummaryHandler`

**استعلام واحد خفيف يجمع كل شيء:**

```csharp
public class DashboardKpisDto
{
    public int ActivePrograms { get; set; }
    public int ActiveClinics { get; set; }
    public int ActivePharmacies { get; set; }
    public int TotalActivePatients { get; set; }
    // ... + KPIs الفترة
}
```

---

## 📌 الجزء الخامس: نظام الفلترة الموحد

### 5.1 مجموعات الفلاتر (3 مجموعات)

1. **المندوب والمنطقة** (أزرق) — `#E8F3F9` → `#FFFFFF`.
2. **البرنامج والتصنيف** (أخضر) — `#EAF4EF` → `#FFFFFF`.
3. **الفترة الزمنية** (برتقالي) — `#FFF3E0` → `#FFFFFF`.

### 5.2 الفلترة المتسلسلة

```
الدولة → المدينة → المنطقة
```

### 5.3 الفلترة التلقائية حسب الشركة

**كل الفلاتر تُفلتر تلقائياً بـ `PharmaCompanyId`.**

### 5.4 نمط `@oninput` (بدون `@bind`)

**⚠️ `@bind:after` مع `select` يسبب خطأ CS0121.**

**الحل:** `@oninput` + `ChangeEventArgs`.

---

## 📌 الجزء السادس: إصلاح البيانات المالية

### 6.1 المشكلة

`PSPDispensations` كانت فارغة ماليًا.

### 6.2 الحل 1: إصلاح `ConfirmDispense`

```csharp
var quantity = request.QuantityDispensed > 0 ? request.QuantityDispensed : 1;
var medication = await db.Set<PharmaCompanyMedication>()
    .Where(m => m.OriginalMedicationID == eRx.MedicationID)
    .FirstOrDefaultAsync();
var supportPercentage = planMedication?.DiscountPercentage > 0
    ? planMedication.DiscountPercentage
    : programMedication?.SupportValue ?? 0;
var unitPrice = medication?.Price ?? 0;
var grossAmount = unitPrice * quantity;
var pspCoverage = grossAmount * (supportPercentage / 100);
var patientPayment = grossAmount - pspCoverage;
```

### 6.3 الحل 2: Migration للبيانات القديمة

**⚠️ استخدم `UPDATE` مع `JOIN` لتعبئة البيانات القديمة.**

### 6.4 النتائج

| المؤشر | القيمة |
|:---|:---:|
| إجمالي الصرفات | 42 |
| إجمالي العلب | 42 |
| إجمالي قيمة المبيعات | 11,268 ج.م |
| إجمالي التعويضات | 3,192 ج.م |
| إجمالي مدفوعات المرضى | 8,076 ج.م |

**⚠️ التحقق:** `11,268 - 3,192 = 8,076` ✅.

---

## 📌 الجزء السابع: الميزات الأمنية

### 7.1 إخفاء بيانات المرضى

**الحل:** إخفاء افتراضي + حقل موافقة (`HasSharedContactConsent`).

### 7.2 الامتثال لـ PDPL

**صفحة الامتثال:** `https://test.rubikcare.com/compliance`

**المتطلبات المُحققة:**
- ✅ موافقة مكتوبة وصريحة.
- ✅ وصول مقيّد وسجل تدقيق.
- ✅ حقوق صاحب البيانات.
- ✅ الاحتفاظ والمحو.

### 7.3 الخطوات التالية

| # | المهمة | الحالة |
|:-:|:---|:---:|
| 1 | إنشاء `PatientConsent` | ⏳ |
| 2 | واجهة موافقة للمريض | ⏳ |
| 3 | ربط `HasSharedContactConsent` | ⏳ |
| 4 | سجل تدقيق الوصول | ⏳ |

---

## 📌 الجزء الثامن: الأنماط والتصميم

### 8.1 التحميل التدريجي

```
1. فتح الصفحة
   ├── تحميل الفلاتر
   └── تحميل Summary فقط (includeDetails=false)
   ↓
2. عرض البطاقات + زر "عرض التفاصيل"
   ↓ (عند الطلب)
3. تحميل التفاصيل (includeDetails=true)
   └── عرض الجدول الكامل
```

### 8.2 الدوائر (Circular KPIs)

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
}
```

### 8.3 `@rendermode InteractiveServer`

**⚠️ إلزامي في كل صفحة.**

### 8.4 مشاكل `BLS.Charts`

| المشكلة | الحل |
|:---|:---|
| `LineSeries` vs `BarSeries` | استخدام `List<BarSeries>` |
| `@rendermode` | إلزامي |
| Overflow في RTL | `overflow: hidden` على `.ch-card-body` |

---

## 📌 الجزء التاسع: مفاتيح الترجمة

### 9.1 البنية

```
DASHBOARD.*   → 52 مفتاحاً (لوحة التحكم)
REPORTS.*     → ~150 مفتاحاً (التقارير)
COMMON.*      → نصوص عامة
```

### 9.2 التحميل في `PharmaDashboard`

**⚠️ تحميل 3 وحدات معاً:**

```csharp
var dashboardTrans = await TranslationService.GetPageTranslationsAsync("DASHBOARD");
var reportsTrans = await TranslationService.GetPageTranslationsAsync("REPORTS");
var commonTrans = await TranslationService.GetPageTranslationsAsync("COMMON");

_translations = dashboardTrans
    .Concat(reportsTrans)
    .Concat(commonTrans)
    .GroupBy(kv => kv.Key)
    .ToDictionary(g => g.Key, g => g.First().Value);
```

### 9.3 `MERGE` بدلاً من `INSERT`

**⚠️ مع `N''` للعربية.**

---

## 📌 الجزء العاشر: خطوات التنفيذ

### 10.1 لكل تقرير جديد

| # | الخطوة | الملف |
|:-:|:---|:---|
| 1 | `IGetXxxHandler.cs` | `Application/UseCases/Reports/Xxx/` |
| 2 | `GetXxxQuery.cs` | نفس المجلد |
| 3 | `GetXxxResponse.cs` | نفس المجلد |
| 4 | `GetXxxHandler.cs` | `Infrastructure/UseCases/Reports/Xxx/` |
| 5 | endpoint في `ReportsController` | `Api.Web/Controllers/Reports/` |
| 6 | تسجيل في `InfrastructureExtensions.cs` | Infrastructure |
| 7 | `XxxReport.razor` | `Shared.UI/Components/Reports/` |
| 8 | `Xxx.razor` (صفحة) | `Rubikcare.Web/Components/Pages/Reports/` |
| 9 | `xxx.css` | `Shared.UI/wwwroot/css/_pages/Reports/` |
| 10 | `@import` في `_PagesBundle.css` | Shared.UI |
| 11 | SQL: `MenuItems` + `Resources` | — |

### 10.2 تسجيل الخدمات

**⚠️ في `InfrastructureExtensions.cs`:**

```csharp
services.AddScoped<IGetProviderFunnelHandler, GetProviderFunnelHandler>();
// ... إلخ
```

### 10.3 القائمة الديناميكية

**⚠️ `MenuItems` — 6 تقارير في `PHARMA_PSP` (MenuID = 4).**

---

## 📌 الجزء الحادي عشر: الدروس المستفادة

### 11.1 الأخطاء الشائعة وحلولها

| # | الخطأ | الحل |
|:-:|:---|:---|
| 1 | `\"` في `@onclick` | دوال مباشرة: `@onclick="NavigateToX"` |
| 2 | `@bind:after` مع `select` | `@oninput` + `ChangeEventArgs` |
| 3 | `LineSeries` vs `BarSeries` | `List<BarSeries>` |
| 4 | `TimeSpan?` لا يحتوي `TotalHours` | `.Value` |
| 5 | `@rendermode` مفقود | إلزامي في كل صفحة |
| 6 | نصوص عربية في Handler | `T()` في الواجهة |
| 7 | تحميل وحدة ترجمة واحدة | 3 وحدات: DASHBOARD + REPORTS + COMMON |
| 8 | `OrganizationID` مباشر | `CurrentOrganizationId` من `UserSession` |

### 11.2 أفضل الممارسات

1. **البطاقات دائماً** — الجدول عند الطلب (`includeDetails`).
2. **التجميع حسب البرنامج أولاً**.
3. **الفلاتر تُفلتر تلقائياً** بـ `PharmaCompanyId`.
4. **إخفاء بيانات المرضى** افتراضياً.
5. **الفلترة الجغرافية** عبر `Organization.AreaID`.
6. **`[Parameter] IApiService`** في `Shared.UI` (لا `@inject`).
7. **`@rendermode InteractiveServer`** إلزامي.
8. **`MERGE`** بدلاً من `INSERT/UPDATE`.
9. **`N''`** للعربية في SQL.
10. **التحقق من البناء** بعد كل تعديل.

### 11.3 القواعد الذهبية

```
✅ Clean Architecture: Handlers في Infrastructure
✅ نمط موحد: Query + Response + Handler لكل تقرير
✅ التحميل التدريجي: Summary دائماً، تفاصيل عند الطلب
✅ الفلترة التلقائية: حسب PharmaCompanyId
✅ الأمان: إخفاء بيانات المرضى افتراضياً
✅ الترجمة: 3 وحدات معاً
✅ @rendermode InteractiveServer: إلزامي
✅ MERGE + N'': في SQL
```

---

## 📌 خاتمة

**نظام التقارير المُنجز:**

| المقياس | القيمة |
|:---|:---:|
| **عدد التقارير** | 6 + لوحة تحكم |
| **عدد Endpoints** | 8 |
| **عدد Handlers** | 8 |
| **عدد ملفات Razor** | 13 |
| **عدد ملفات CSS** | 6 |
| **عدد DTOs** | ~30 |
| **مفاتيح الترجمة** | ~200 |
| **الحالة** | ✅ مكتمل ومُختبر |

**⚠️ هذا النظام يُعتبر الأساس القوي لنظام النقاط (Rubika) القادم.**

---

## 📌 سجل التغييرات

| الإصدار | التاريخ | التغييرات |
|:---:|:---|:---|
| **1.0** | **16 أكتوبر 2026** | الإصدار الأولي — `27-reports-system.md` |
| **2.0** | **17 أكتوبر 2026** | **`07-REPORTS` v2.0** — دمج + ترقيم جديد + إشارات |

---

**© 2026 RubikCare — للاستخدام الداخلي**

**نهاية وثيقة `07-REPORTS` ✅**
```

---

