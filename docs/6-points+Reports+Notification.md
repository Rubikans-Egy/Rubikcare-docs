# 📘 `06-POINTS` — v7.0 (Unified Edition)

**قرأتُ `26-Rubika-Points-System` v6.0 كاملاً. الآن سأدمج مع التصحيحات الفعلية (14-15 أكتوبر) + فصل `07-REPORTS` + `08-NOTIFICATIONS`.**

---

## 🎯 مقدمة

**⚠️ ملاحظة:** سأُدمج `26-Points` (v6.0) مع migrations الفعلية (14-15 أكتوبر) التي لم تكن في `26`.

**⚠️ سيتم فصل:**
- `07-REPORTS` — وثيقة مستقلة.
- `08-NOTIFICATIONS` — وثيقة مستقلة.

---

```markdown
# 06 — POINTS (نظام النقاط وروبيكا)

**الإصدار:** 7.0 (Campaign Reference + Extended Beneficiaries)
**آخر تحديث:** 17 أكتوبر 2026
**الحالة:** ✅ Backend مكتمل — Frontend جزئي
**الجمهور:** فريق التطوير + AI Assistants
**المرجع الإلزامي:** يُقرأ قبل أي تعديل على النقاط

---

## 📌 الجزء صفر: الفلسفة والرؤية (نص المؤلف الحرفي)

> **⚠️ ملاحظة:**
> هذا القسم يحتوي على **نص الكاتب الحرفي** — دون إعادة صياغة.

### 0.1 الرؤية العامة لنظام النقاط

> **نص المؤلف الحرفي (من جلسة 10 أكتوبر 2026):**
>
> "نظام النقاط مصمم ليتم إضافته ليس لخدمة ما ولكن لحملة تسويقية داخل الخدمة تنشأها المنظمة المستفيدة من الخدمة. على سبيل المثال كل برنامج دعم سنعتبر حملة تسويقية. وستجد أن النظام الحالي مرتبط بشكل منفصل بكل برنامج."

**الترجمة التقنية — مُحدَّثة (13 أكتوبر 2026):**
- **الرابط الفعلي** = `SourceEntityID` + `PointSystemTypeID`.
- **`SourceEntityID`** — معرّف الحملة المصدر.
- **`PointSystemTypeID`** — تصنيف (PSP=1, B2B=2, B2C=3, PLATFORM_MKT=4, RUBIKA=5).

**⚠️ `ProgramID` — محذوف نهائياً.**

### 0.2 منطق الاستفادة من نظام النقاط كنظام مستقل

> **نص المؤلف الحرفي:**
>
> "أولاً منطق الاستفادة من نظام النقاط كنظام مستقل عن أي خدمة مقدمة أو عدة خدمات. فمثلاً من المخطط له عمل نظام إدارة عيادات باشتراكات متنوعة، فيتم إلحاق نظام النقاط كميزة ملحقة فيه.
>
> من المخطط عمل شبكة دعم لوجيستي كاملة متعددة المراحل، ما بين شركات الأدوية ومخازن جملة ثم مخازن تجزئة ثم من الأخيرة إلى الصيدليات ثم من الصيدليات إلى المستهلك. فكل نقطة مما ذكرت سيتم إدارة وتغير value chain وتدفق عمليات وأنشطة تجارية وتسويقية تتناسب وطبيعة كل خطوة. ومطلوب إدراج نظام النقاط في كل عملية بينية من ال saler to payer in every stage.
>
> كذلك من المقرر تطوير وتفعيل Medical CRM لإدارة علاقات العملاء ومندوبي الدعاية medical reps visits to doctors & MR calls & visits plans وغيرها. ويمكن تحويل نظام النقاط من قبل الشركة إلى نظام حساب KPI's for MR's، point system for doctors & pharmacies، وهكذا والأمثلة متعددة."

**الترجمة التقنية:**
- **بنية أفقية** — تخدم أي خدمة عبر `PointSystemTypeID`.
- **كل خدمة تُنشئ كياناتها** — تُعطي `SourceEntityID`.
- **النقاط تُمنح في كل مرحلة** — `saler → payer`.
- **المستفيد يختلف** حسب المرحلة.

### 0.3 Rubika Currency — النظام الموازي

> **نص المؤلف الحرفي:**
>
> "وكما شرحت أنت من قبل أن هناك نظام موازي من المنصة لكل العملاء والمنظمات في كل الخدمات. سيستخدم أحياناً كوحدة خدمية في حالة المحاسبة بنظام العمولة.
>
> يقوم الطبيب بشحن رصيد روبيكا فيتمكن من استقبال طلبات الحجز أون لاين. ويقوم الصيدلي بشحن رصيد روبيكا فيتمكن من استقبال طلبات شراء دواء.
>
> ويمكن للمنصة أن تمنح لكل منهما نقاط مقابل referring platform to others أو أي نشاط يدعم توسع عمل المنصة وانتشارها تسويقياً.
>
> وكذلك يتم منح النقاط للمستخدم العادي أياً كان دوره أو حتى لو لم يكن له دور فهو في النهاية مريض محتمل. حيث أن المرض في المنصة ليس دور ولكنه حالة مؤقتة قد يمر بها الطبيب والمندوب والصيدلي وغيره.
>
> فيحصل مقابل أنشطة تحددها المنصة ويمكنه تحويلها من رصيده إلى العيادات للحصول على خصم مقابلها أو للصيدلي للحصول على خصم مقابل شراء الأدوية."

**الترجمة التقنية:**
- **Rubika** = طبقة مستقلة — `PointSystemTypeID = 5`.
- **المرض ليس دوراً** — بل حالة.
- **كل مستخدم = مريض محتمل** (`BeneficiaryTypeID = 4`).

### 0.4 منطق الحملات التسويقية

> **نص المؤلف الحرفي:**
>
> "أنا أقترح أيضاً أن يتم ربط نظام النقاط في كل خدمة بحملة تسويقية معينة يتم إنشاؤها من قبل المنظمة المستفيدة..."

**الترجمة التقنية:**
- **حملة** = كيان داخل كل خدمة.
- **PSP:** `PSPProgram` — `PointSystemTypeID = 1`.
- **B2B:** `PurchaseOrder` — `PointSystemTypeID = 2`.
- **قرار:** **لا جدول `Campaigns` عام**.

### 0.5 تعدد Schemes داخل نفس البرنامج

> **نص المؤلف الحرفي:**
>
> "مطلوب أن تتمكن المنظمة من وضع عدة أنظمة نقاط schema داخل نفس البرنامج. فمثلاً في برامج الدعم تتمكن من تصميم نظام للعيادات - الصيدليات - المرضى - العاملين في أي منظمة وفق نوعها. العاملين بها (المندوبين الداخليين المستخدمين للمنصة)."

**الترجمة التقنية:**
- `TargetBeneficiaryTypeID` في `PSPProgramPointScheme` (nullable).
- **Filtered Unique Indexes** — منذ 10 أكتوبر.

### 0.6 نمط "الفئة المستهدفة" (نص المؤلف الحرفي)

> **نص المؤلف الحرفي:**
>
> "لاحظ أن كل نوع من المستفيدين يمكن جلبه من جدول محدد ولهذا تم تحديد 6 أنواع فقط. والجداول هي أنواع المنظمات + جدول المنظمات لتحديد المنظمة المستفيدة. جدول userprofile إن كانت النقاط للمريض. جدول orgmembership + customjobtitle لتحديد المستخدم إن كان عضو داخلي أو خارجي في المنظمة."

**القاعدة الذهبية:**

> **`BeneficiaryTypeID` يحدد الجدول الذي سيُجلب منه المستفيد، ولا يحمل بنفسه هوية المستفيد.**

| `BeneficiaryTypeID` | الجدول المصدر | المعرّف الفعلي |
|:---:|:---|:---|
| **1** = ORGANIZATION | `Organizations` | `OrganizationID` |
| **2** = ORG_MEMBER_INTERNAL | `OrgMemberships` + `CustomJobTitles` | `UserProfileID` |
| **3** = ORG_MEMBER_EXTERNAL | `OrgMemberships` + `CustomJobTitles` | `UserProfileID` |
| **4** = REGULAR_USER | `UserProfiles` | `UserProfileID` |
| **5** = PATIENT | `UserProfiles` + `PSPParticipations` | `UserProfileID` |
| **6** = GENERIC | (مستقبلاً) | — |

**⚠️ `OrganizationTypeFilterID`** (بدلاً من `TargetOrganizationTypeID`):

| Rule | `BeneficiaryTypeID` | `OrganizationTypeFilterID` |
|:---|:---:|:---:|
| #1 | 1 (ORGANIZATION) | 4 (CLINIC) → النقاط للعيادات |
| #2 | 5 (PATIENT) | NULL → للمريض |
| #3 | 2 (ORG_MEMBER_INTERNAL) | NULL → للمندوب |

**⚠️ الخلاصة:** `BeneficiaryTypeID` + `OrganizationTypeFilterID` = الفئة المستهدفة بدقة.

### 0.7 إدارة الجداول المرجعية

> **نص المؤلف الحرفي (من جلسة 10 أكتوبر 2026):**
>
> "ملاحظتي الآن أن هناك جداول كثيرة تم إضافتها وكلها seed data، ولم نضع خطة لإضافة واجهات لتحديث هذه الجداول ببيانات تشغيلية."

**الجداول التي تحتاج واجهة إدارية:**
- `PSPActivityTypes` (7 قيم).
- `PointSystemTypes` (5 قيم).
- `PointBeneficiaryTypes` (6 قيم).
- `PointSystemTypeBeneficiaryTypes` (M:N).

**الجمهور:** Super Admin.
**الأولوية:** 🟠 متوسطة.

---

## 🎯 الجزء الأول: نظرة عامة تقنية

**نظامان، محرك واحد.**

| البُعد | Program Points | Rubika Currency |
|:---|:---:|:---:|
| **الطبيعة** | مكافآت تحفيزية | عملة داخلية |
| **المُصدر** | شركة / مؤسسة | RubikCare |
| **الدورة** | كسب → صرف | شحن → استخدام |
| **`PointSystemTypeID`** | 1, 2, 3, 4 | 5 |
| **الجداول** | `PSPProgramPoint*` | `RubikaWallet` + `RubikaTransactions` |

**المبدأ:** **نفس المحرك (`IPointEngineService`)، تنفيذان مختلفان.**

---

## 🎯 الجزء الثاني: Campaign Reference Architecture

### 2.1 مفهوم Campaign

> **"الحملة التسويقية = كيان داخل كل خدمة. لا يوجد جدول `Campaigns` عام."**

**⚠️ المشكلة القديمة:**
- `PSPProgramPointSchemes.ProgramID` — FK صلب.
- يمنع استخدام النظام في خدمات أخرى.

**⚠️ الحل الجديد:**
- `SourceEntityID` — معرّف الحملة.
- `PointSystemTypeID` — يميّز نوع الخدمة.
- **لا FK إلى `PSPPrograms`**.

### 2.2 `SourceEntityID` + `PointSystemTypeID`

**البنية الجديدة لـ `PSPProgramPointScheme`:**

| العمود | النوع | الوصف |
|:---|:---:|:---|
| `SchemeID` | `INT PK` | المعرف |
| **`PointSystemTypeID`** | `INT` | نوع نظام النقاط |
| **`SourceEntityID`** | `INT NOT NULL` | معرّف الحملة المصدر |
| `TargetBeneficiaryTypeID` | `INT?` | نوع المستفيد المستهدف |
| **`OrganizationTypeFilterID`** | `INT?` | ⭐ **جديد (14 أكتوبر)** — فلترة نوع المؤسسة |
| `ServiceProductID` | `INT?` | ربط بميزة مسعَّرة |

**⚠️ العلاقة:**

```
SourceEntityID + PointSystemTypeID = الحملة الفعلية
```

**⚠️ جدول توضيحي:**

| `PointSystemTypeID` | `SourceEntityID` يُمثّل |
|:---:|:---|
| 1 (PSP) | `PSPProgram.ProgramID` |
| 2 (B2B) | `PurchaseOrder.ID` (مستقبلاً) |
| 3 (B2C) | `Campaign.ID` (مستقبلاً) |
| 4 (PLATFORM_MKT) | `Campaign.ID` (مستقبلاً) |
| 5 (RUBIKA) | `Wallet.ID` أو `null` |

### 2.3 `ServiceProductID`

- **`NULL`** — غير مرتبط بميزة.
- **`X`** — مرتبط بميزة مثل `LOYALTY_POINTS_SYSTEM`.

**⚠️ الحالة:** موجود — غير مُستخدم.

---

## 🎯 الجزء الثالث: البنية المعمارية

### 3.1 المستويات الأربعة

```
Layer 1: Reference Layer
  ├── PointSystemTypes (5 قيم Seed)
  ├── PSPActivityTypes (7 قيم Seed)
  ├── PointBeneficiaryTypes (6 قيم Seed)
  ├── PSPProgramPointTransactionTypes (4 قيم Seed)
  └── PointSystemTypeOrganizationTypes ⭐ جديد (14 أكتوبر)

Layer 2: Beneficiary Layer
  ├── PointSchemeBeneficiaryTypes (M:N)
  └── PointSystemTypeBeneficiaryTypes (M:N)

Layer 3: Organization Structure
  ├── Organizations
  ├── OrganizationGroups (موجود — غير مُستخدم)
  └── OrganizationGroupMembers

Layer 4: Value Engines
  ├── Program Points Engine (PSPProgramPoint*)
  └── Rubika Currency Engine (RubikaWallet + RubikaTransactions)
```

### 3.2 الجداول الفعلية

#### 🔹 المرجعية (Reference)

**`PointSystemTypes` — 5 قيم:**

| ID | TypeCode | NameAr | IssuerEntityType | SupportsCashRedeem |
|:--:|:---|:---|:---|:---:|
| 1 | PSP | نقاط برامج الدعم | PHARMA | ✅ |
| 2 | B2B_SUPPLY | نقاط التوريد B2B | PHARMA,WAREHOUSE | ❌ |
| 3 | B2C_PHARMACY | نقاط الصيدلية للجمهور | PHARMACY | ✅ |
| 4 | PLATFORM_MKT | نقاط المنصة التسويقية | PLATFORM | ✅ |
| 5 | RUBIKA | روبيكا | PLATFORM | ✅ |

**`PSPActivityTypes` — 7 قيم:**

| Code | الحدث |
|:---|:---|
| `PATIENT_INVITED` | دعوة مريض |
| `PATIENT_ENROLLED` | تسجيل مريض |
| `ERX_CREATED` | إنشاء وصفة |
| `DISPENSATION_COMPLETED` | صرف دواء |
| `REFILL_COMPLETED` | إعادة صرف |
| `PROGRAM_COMPLETED` | إكمال برنامج |
| `LAB_TEST_UPLOADED` | رفع تحليل |

**`PointBeneficiaryTypes` — 6 قيم:**

| ID | Code | NameAr | IsOrganization |
|:--:|:---|:---|:---:|
| 1 | ORGANIZATION | مؤسسة | ✅ |
| 2 | ORG_MEMBER_INTERNAL | عضو داخلي | ❌ |
| 3 | ORG_MEMBER_EXTERNAL | عضو خارجي | ❌ |
| 4 | REGULAR_USER | مستخدم عادي | ❌ |
| 5 | PATIENT | مريض | ❌ |
| 6 | GENERIC | عام | ❌ |

**⭐ `PointSystemTypeOrganizationTypes` — جدول جديد (14 أكتوبر):**

**⚠️ يربط `PointSystemType` بـ `OrganizationType`:**
- **الغرض:** فلترة أنواع المؤسسات المسموحة لكل نظام نقاط.
- **مثال:** `PointSystemType = PSP (1)` → مسموح `CLINIC`, `PHARMACY`, `HOSPITAL`.

#### 🔹 الطبقة التشغيلية (Value Engine)

| الجدول | الدور |
|:---|:---|
| `PSPProgramPointSchemes` | نظام نقاط (Scheme) |
| `PSPProgramPointRules` | قواعد منح النقاط |
| `PSPProgramPointBalances` | أرصدة المستفيدين |
| `PSPProgramPointTransactions` | سجل المعاملات |
| `PSPProgramPointRedeemRequests` | طلبات الاسترداد |
| `PSPProgramPointRedeemStatuses` | حالات طلبات الاسترداد |
| `PSPProgramPointTransactionTypes` | أنواع المعاملات |

#### 🔹 الجداول M:N

- `PointSchemeBeneficiaryTypes` — يربط Schemes بأنواع المستفيدين.
- `PointSystemTypeBeneficiaryTypes` — يربط أنواع النظم بأنواع المستفيدين.

#### 🔹 Rubika Currency

- `RubikaWallets` — محافظ العملة.
- `RubikaTransactions` — المعاملات.
- `RubikaValueSettings` — إعدادات القيمة.

### 3.3 مخطط ERD

```
Organizations
  ├── PSPPrograms (حملة PSP)
  │   └── PSPProgramPointSchemes
  │       ├── PointSystemTypeID → PointSystemTypes
  │       ├── SourceEntityID → (مفتوح)
  │       ├── TargetBeneficiaryTypeID → PointBeneficiaryTypes
  │       ├── OrganizationTypeFilterID → OrganizationTypes ⭐
  │       ├── ServiceProductID → ServiceProducts
  │       ├── PSPProgramPointRules
  │       │   ├── PSPActivityTypes
  │       │   ├── PointBeneficiaryTypes
  │       │   └── OrganizationTypeFilterID ⭐
  │       ├── PSPProgramPointBalances
  │       │   ├── PSPProgramPointTransactions
  │       │   └── PSPProgramPointRedeemRequests
  │       └── PointSchemeBeneficiaryTypes
  │
  └── (B2B / Clinic / CRM — مستقبلاً)

PointSystemTypes
  └── PointSystemTypeOrganizationTypes ⭐ (M:N)
      └── OrganizationTypes
```

---

## 🎯 الجزء الرابع: Backend المُنفَّذ

### 4.1 المكوّنات الأساسية

| # | المكوّن | الموقع |
|:-:|:---|:---|
| 1 | `EarnPointsCommand` | `Application/UseCases/PSP/Points/Earning/` |
| 2 | `EarnPointsHandler` | `Infrastructure/UseCases/Points/Earning/` |
| 3 | `PointsContext` | `Application/UseCases/PSP/Points/Earning/` |
| 4 | `IEarnPointsOrchestrator` | Application + Infrastructure |
| 5 | `IPointEngineService` | Application + Infrastructure (Facade) |
| 6 | `IPSPPointSchemeService` | Application (in-process) |
| 7 | `IPointSchemeClientService` | Web (HTTP) |

### 4.2 نمط العمل

```
PSPEnrollmentService / DispenseController
    ↓
TryAwardPointsViaOrchestratorAsync
    ↓
PointsContext (OrganizationID + PatientID + UserID)
    ↓
IEarnPointsOrchestrator.AwardByCampaignAsync(
    campaignId,
    pointSystemTypeID: 1,  // ⭐ PSP
    activityTypeId,
    context,
    ...
)
    ↓
اقرأ Rules النشطة (SchemeID + ActivityTypeID)
    ↓
لكل Rule (BeneficiaryTypeID مختلف)
    ↓
IEarnPointsHandler.HandleAsync
    ↓
Balance + Transaction
```

### 4.3 `IPointEngineService` — Facade

```csharp
public interface IPointEngineService
{
    Task<EarnPointsResponse> EarnAsync(EarnPointsRequest request, CancellationToken ct = default);
    Task<OrchestratorResult> EarnMultiAsync(EarnMultiRequest request, CancellationToken ct = default);
    Task<GetBalanceResponse> GetBalanceAsync(GetBalanceRequest request, CancellationToken ct = default);
    Task<GetTransactionsResponse> GetTransactionsAsync(GetTransactionsRequest request, CancellationToken ct = default);
}
```

### 4.4 `IPSPPointSchemeService` vs `IPointSchemeClientService`

**⚠️ ازدواجية موثَّقة:**
- **`IPSPPointSchemeService`** (Application) — قراءة in-process.
- **`IPointSchemeClientService`** (Web) — قراءة عبر HTTP.
- **`GetPointSchemeByProgramHandler`** (Infrastructure) — قراءة عبر API.

**⚠️ الدين تقني** — يحتاج تنظيف.

### 4.5 Controllers — محدَّثة (بعد 14-15 أكتوبر)

**`PointSchemesController` — 13 Endpoints:**

| Method | Endpoint |
|:---|:---|
| POST | `/api/psp/points/schemes` |
| PUT | `/api/psp/points/schemes/{schemeId}` |
| GET | `/api/psp/points/schemes/by-program/{programId}` |
| POST | `/api/psp/points/schemes/{schemeId}/deactivate` |
| GET | `/api/psp/points/schemes/list-by-program/{programId}` |
| GET | `/api/psp/points/schemes/programs/{programId}/overview` |
| GET | `/api/psp/points/schemes/programs/{programId}/clinics` |
| GET | `/api/psp/points/schemes/by-organization/{organizationId}` |
| GET | `/api/psp/points/schemes/{schemeId:int}` |
| GET | `/api/psp/points/schemes/{schemeId:int}/clinics` |
| GET | `/api/psp/points/schemes/{schemeId:int}/beneficiary-types` |
| POST | `/api/psp/points/schemes/{schemeId:int}/beneficiary-types` |
| DELETE | `/api/psp/points/schemes/{schemeId:int}/beneficiary-types/{id:int}` |

**`PointRulesController` — 5 Endpoints.**
**`PointBalancesController` — 7 Endpoints.**
**`PointEarningController` — 1 Endpoint.**
**`PointRedeemController` — 8 Endpoints.**
**`PointLookupsController` — 4 Endpoints.** ⭐ جديد (14 أكتوبر)

**⚠️ التفاصيل الكاملة في `09-API-GUIDE`.**

---

## 🎯 الجزء الخامس: Migrations المُنفَّذة

### 5.1 Migrations (13 → 15 أكتوبر)

**⚠️ جدول كامل:**

| # | Migration | التاريخ | الوصف |
|:-:|:---|:---:|:---|
| 1 | `AddPSPProgramPointsTables` | 29 سبتمبر | الجداول الأساسية |
| 2 | `AddPointSystemTypesAndBeneficiary` | 7 أكتوبر | PointSystemTypes |
| 3 | `AddPointBeneficiaryAndOrganizationGroups` | 9 أكتوبر | Beneficiary + OrganizationGroups |
| 4 | `RemoveLegacyBeneficiaryColumns` | 9 أكتوبر | حذف الأعمدة القديمة |
| 5 | `AddTargetBeneficiaryTypeToScheme` | 10 أكتوبر | `TargetBeneficiaryTypeID` |
| 6 | `UseSourceEntityIdInPointSchemes` | 13 أكتوبر | `ProgramID` → `SourceEntityID` |
| 7 | **`AddTargetOrganizationTypeIdToRules`** | **14 أكتوبر** ⭐ | `TargetOrganizationTypeId` في Rules |
| 8 | **`AddPointSystemTypeOrganizationTypes`** | **14 أكتوبر** ⭐ | M:N جديد |
| 9 | **`AllowMultipleSchemesPerCampaign`** | **15 أكتوبر** ⭐ | إزالة Unique على Single Scheme |
| 10 | **`IncludeTargetOrgTypeInRuleUniqueIndex`** | **15 أكتوبر** ⭐ | تعديل Unique Index |

**⚠️ ملاحظة:** `26-Points` v6.0 كانت متأخرة عن 14-15 أكتوبر — الآن مُحدَّثة.

### 5.2 Migration 1: `UseSourceEntityIdInPointSchemes`

**المُعدَّل:**
- DropForeign على `ProgramID`.
- DropIndex على `IX_..._ProgramID_General` و `IX_..._ProgramID_TargetBeneficiary`.
- DropColumn `ProgramID`.
- AlterColumn `SourceEntityID` → NOT NULL.
- AddColumn `ServiceProductID` (int?).
- CreateIndex على `(SourceEntityID, PointSystemTypeID)` — Filtered Unique.
- CreateIndex على `(SourceEntityID, PointSystemTypeID, TargetBeneficiaryTypeID)` — Filtered Unique.

### 5.3 Migration 2: `AddTargetOrganizationTypeIdToRules`

**⚠️ جديد (14 أكتوبر):**
- **إضافة `TargetOrganizationTypeId`** إلى `PSPProgramPointRules`.
- **الغرض:** فلترة نوع المؤسسة في القواعد.
- **الاستخدام:** ربط بـ `PointSystemTypeOrganizationTypes`.

### 5.4 Migration 3: `AddPointSystemTypeOrganizationTypes`

**⚠️ جديد (14 أكتوبر):**
- **إنشاء جدول `PointSystemTypeOrganizationTypes`** (M:N).
- **يربط:** `PointSystemType` ↔ `OrganizationType`.
- **الغرض:** تحديد أنواع المؤسسات المسموحة لكل نظام نقاط.
- **Seed Data:** PSP → [CLINIC, PHARMACY, HOSPITAL, ...].

### 5.5 Migration 4: `AllowMultipleSchemesPerCampaign`

**⚠️ جديد (15 أكتوبر):**
- **إزالة Unique Index** على `(SourceEntityID, PointSystemTypeID)`.
- **السبب:** السماح بـ Schemes متعددة لنفس الحملة.
- **الفائدة:** يمكن للشركة تعريف 3 Schemes (للعيادة، للصيدلية، للمريض).

### 5.6 Migration 5: `IncludeTargetOrgTypeInRuleUniqueIndex`

**⚠️ جديد (15 أكتوبر):**
- **تعديل Unique Index** على Rules.
- **السبب:** إضافة `TargetOrganizationTypeId` كجزء من المفتاح.

---

## 🎯 الجزء السادس: خطة Frontend

### 6.1 `LoyaltyHub.razor` — ✅ يعمل

**المسار:** `Web/Components/Pages/Loyalty/LoyaltyHub.razor`.

**الوظائف:**
- Dashboard — 4 Stats (Schemes + Rules + Balance + Pending Redeems).
- Quick Actions — Schemes, Balances, Redeem, Settings.
- Recent Schemes.

### 6.2 الصفحات المتبقية

| # | الصفحة | المسار | الحالة |
|:-:|:---|:---|:---:|
| 1 | `LoyaltySchemesList.razor` | `/loyalty/schemes` | ⏳ |
| 2 | `LoyaltySchemeDetails.razor` | `/loyalty/schemes/{id}` | ⏳ |
| 3 | `LoyaltySchemeEdit.razor` | `/loyalty/schemes/{id}/edit` | ⏳ |
| 4 | `LoyaltySchemeNew.razor` | `/loyalty/schemes/new` | ⏳ |
| 5 | `LoyaltyBalances.razor` | `/loyalty/balances` | ⏳ |
| 6 | `LoyaltyRedeemRequests.razor` | `/loyalty/redeem-requests` | ⏳ |
| 7 | `LoyaltySettings.razor` | `/loyalty/settings` | ⏳ |

### 6.3 واجهات إدارة الجداول المرجعية

**⚠️ مطلوبة:**
- إدارة `PSPActivityTypes`.
- إدارة `PointSystemTypes`.
- إدارة `PointBeneficiaryTypes`.
- إدارة `PointSystemTypeBeneficiaryTypes`.

### 6.4 واجهات `OrganizationGroups`

**⚠️ تأسيس فقط — غير مُستخدَم.**

---

## 🎯 الجزء السابع: خطة التوسع

### 7.1 Rubika Currency Engine

**⚠️ Schema موجود — Engine غير مُنفَّذ.**

**الخطوة التالية:** `IRubikaEngineService` في جلسة منفصلة.

### 7.2 الحملات في خدمات أخرى

| الخدمة | الحملة = | `PointSystemTypeID` |
|:---|:---|:---:|
| PSP | `PSPProgram` | 1 — ✅ |
| B2B_SUPPLY | `PurchaseOrder` | 2 — 🟡 |
| B2C_PHARMACY | `PharmacyCampaign` | 3 — 🟡 |
| PLATFORM_MKT | `MarketingCampaign` | 4 — 🟡 |
| RUBIKA | (نظام عام) | 5 — 🟡 |
| CRM | `MRCampaign` | 6+ — 🟡 |

### 7.3 KPI System for MRs

**⚠️ الفكرة:** استخدام نظام النقاط لحساب مؤشرات أداء مندوبي الدعاية.

**`PointSystemTypeID`:** `CRM = 6` — أو `PLATFORM_MKT = 4`.

**`BeneficiaryTypeID`:** `ORG_MEMBER_INTERNAL = 2`.

### 7.4 إضافة خدمات جديدة

```
1. إضافة PointSystemType جديد
   ↓
2. ربط BeneficiaryTypes المسموحة
   ↓
3. ربط OrganizationTypes المسموحة ⭐ (14 أكتوبر)
   ↓
4. إضافة ActivityTypes خاصة (إن لزم)
   ↓
5. إنشاء Scheme جديد — SourceEntityID + PointSystemTypeID
   ↓
6. (اختياري) ربط Scheme بـ ServiceProductID
```

### 7.5 MenuVisibilityRules + Subscriptions

**⚠️ الربط بنظام القوائم:**
- `LOYALTY_MENU` له `MenuType = 'FEATURE_BASED'`.
- عند تفعيل الاشتراكات:
  - `SubscriptionPlan` يحتوي على `LOYALTY_POINTS_SYSTEM`.
  - `OrganizationSubscription` يربط المنظمة بالخطة.
  - `MenuVisibilityRule` (RuleType=Subscription) يظهر القائمة.

**⚠️ الحالة:** 🟡 مُخطَّطة — تعتمد على `03-MENUS` + `19-PERMISSIONS`.

---

## 📌 نهاية Part 1

**⚠️ Part 2 سيحتوي على:**

-# [#63] 📘 `06-POINTS` — v7.0 (Part 2) + `07-REPORTS` + `08-NOTIFICATIONS`

---

## 📌 الجزء الثامن: ملخص الحالة

### 8.1 مكتمل ✅

#### 🔹 Domain Layer

| # | المكوّن | العدد |
|:-:|--------|:---:|
| 1 | كيانات `PSPProgramPoint*` | 8 |
| 2 | كيانات Beneficiary | 3 |
| 3 | كيانات Rubika | 3 |
| 4 | كيانات OrganizationGroups | 2 |
| 5 | `PointSystemType` | 1 |
| 6 | **`PointSystemTypeOrganizationType`** ⭐ | 1 |
| 7 | `ServiceProduct` (من نظام الخدمات) | 1 |
| 8 | Constants | 3 |
| 9 | Enum (`RuleType`) | 1 |

#### 🔹 Application Layer

| # | المكوّن | العدد |
|:-:|--------|:---:|
| 1 | Commands / Queries / DTOs | 25+ |
| 2 | Interfaces (`IEarnPointsHandler`, `IEarnPointsOrchestrator`, `IPointEngineService`) | 3 |
| 3 | Services (`PSPEnrollmentService`, `PSPPointSchemeService`) | 2 |

#### 🔹 Infrastructure Layer

| # | المكوّن | العدد |
|:-:|--------|:---:|
| 1 | Handlers | 20+ |
| 2 | Services (`EarnPointsOrchestrator`, `PointEngineService`) | 2 |

#### 🔹 Api.Web Layer

| # | المكوّن | العدد |
|:-:|--------|:---:|
| 1 | Controllers | 6 ⭐ (منهم PointLookups) |
| 2 | Endpoints | ~38 |

#### 🔹 Web Layer

| # | المكوّن | العدد |
|:-:|--------|:---:|
| 1 | Client Services (`IPointSchemeClientService`) | 1 |
| 2 | Pages (`LoyaltyHub`, `PSPStep2b_PointScheme`, `PSPProgramManage`, `PSPProgramsDetails`) | 4 |

#### 🔹 Migrations (10 migrations)

**⚠️ التفاصيل في القسم 5.**

### 8.2 قيد الانتظار ⏳

#### 🔴 أولوية عالية

| # | المهمة | السبب |
|:-:|--------|:---|
| 1 | **صفحات Loyalty المتبقية** (7 صفحات) | إكمال Frontend |
| 2 | **ربط `PSPStep2b_PointScheme` بـ `LoyaltySchemeEdit`** | إعادة استخدام |
| 3 | **اختبار E2E شامل** | ضمان الاستقرار |

#### 🟠 أولوية متوسطة

| # | المهمة | السبب |
|:-:|--------|:---|
| 1 | **واجهات إدارة الجداول المرجعية** | Super Admin |
| 2 | **تنظيف `PSPPointSchemeService`** | دين تقني |
| 3 | **`Rubika Currency Engine`** | نظام موازٍ |
| 4 | **ربط `PROGRAM_COMPLETED`** | حدث نقاط |
| 5 | **ربط `LAB_TEST_UPLOADED`** | حدث نقاط |
| 6 | **`OrganizationTypeFilterID` UI** ⭐ | من 14 أكتوبر |

#### 🟡 أولوية منخفضة

| # | المهمة | السبب |
|:-:|--------|:---|
| 1 | **`OrganizationGroups` UI** | تأسيس فقط |
| 2 | **توسعة MAUI/PWA** | قوائم ديناميكية |

---

## 📌 الجزء التاسع: EF Core Migration — ملاحظات حرجة

### 9.1 المبدأ الأساسي

> **"Migration المُولَّد صحيح — فقط عندما يكون Snapshot متزامناً مع DB."**

**⚠️ EF Core يقارن: Snapshot + Model Current. لا يفحص DB.**

### 9.2 مثال حي — جلسة 13 أكتوبر

**⚠️ ما حدث:**

1. **تعديل `OnModelCreating`** — تحويل `ProgramID` → `SourceEntityID`.
2. **`Add-Migration UseSourceEntityIdInPointSchemes`** — حاول `DropIndex("IX_..._ProgramID")`.
3. **`Update-Database`** — فشل — لأن DB يحتوي `IX_..._ProgramID_General` وليس `IX_..._ProgramID`.
4. **السبب:** Snapshot كان يعكس حالة **قبل** Migration سابق.
5. **الحل:**
   - **SQL مباشر:** `UPDATE PSPProgramPointSchemes SET SourceEntityID = ProgramID`.
   - **تعديل Snapshot يدوياً.**
   - **`Add-Migration` جديد** — يعكس الفرق الحقيقي.
   - **`Update-Database`** — نجح.
6. **النتيجة:** Model = Snapshot = DB.

### 9.3 القواعد المُعتمَدة

| # | القاعدة | السبب |
|:-:|:---|:---|
| 1 | **لا تعديل Migration files يدوياً** | تُلغى عند أول `Add-Migration` |
| 2 | **لا تعديل `ModelSnapshot.cs` كروتين** | يُلغى عند أول `Add-Migration` |
| 3 | **SQL مباشر للبيانات (Data Migration)** — مسموح | مثل `UPDATE SourceEntityID = ProgramID` |
| 4 | **تعديل Snapshot يدوياً** — استثناء نادر | لمرة واحدة — مع تحقق فوري |
| 5 | **Backup DB** قبل أي Migration | حماية |
| 6 | **مراجعة Migration المُولَّد** قبل `Update-Database` | اكتشاف الأخطاء مبكراً |

### 9.4 Checklist قبل Migration على نظام النقاط

- [ ] Backup DB كامل.
- [ ] Snapshot = DB؟
- [ ] `OnModelCreating` يُطابق Entity؟
- [ ] `PointSystemTypeID` مُعرَّف؟
- [ ] `SourceEntityID` NOT NULL في Entity + DB؟
- [ ] `OrganizationTypeFilterID` ⭐ (14 أكتوبر)؟
- [ ] `PointSystemTypeOrganizationTypes` (M:N)؟
- [ ] `Add-Migration` — مراجعة Migration المُولَّد.
- [ ] `Update-Database` — مراقبة الأخطاء.

---

## 📌 الجزء العاشر: روابط + سجل التغييرات

### 10.1 روابط ذات صلة

- [00 — MASTER-INDEX](./00-MASTER-INDEX.md)
- [00B — PROJECT-BACKLOG](./00B-PROJECT-BACKLOG.md)
- [01 — ARCHITECTURE](./01-ARCHITECTURE.md)
- [02 — IDENTITY](./02-IDENTITY.md)
- [03 — MENUS](./03-MENUS.md)
- [04 — TRANSLATION](./04-TRANSLATION.md)
- [05 — PSP](./05-PSP.md)
- [07 — REPORTS](./07-REPORTS.md)
- [08 — NOTIFICATIONS](./08-NOTIFICATIONS.md)
- [09 — API-GUIDE](./09-API-GUIDE.md)
- [19 — PERMISSIONS](./19-PERMISSIONS.md)

### 10.2 سجل التغييرات

| الإصدار | التاريخ | التغييرات |
|:---:|:---|:---|
| **1.0** | **29 سبتمبر 2026** | الإصدار الأولي |
| **2.0** | **3 أكتوبر 2026** | التفاصيل الكاملة |
| **3.0** | **8 أكتوبر 2026** | Beneficiary Types + Organization Groups + Rubika |
| **4.0** | **9 أكتوبر 2026** | Stage 1-3 منفّذة |
| **5.0** | **10 أكتوبر 2026** | نص المؤلف الحرفي + Backend الكامل |
| **6.0** | **13 أكتوبر 2026** | Campaign Reference Architecture |
| **7.0** | **17 أكتوبر 2026** | **`06-POINTS` v7.0** — دمج `26` v6.0 + migrations 14-15 أكتوبر (4 جديدة) + فصل `07-REPORTS` + `08-NOTIFICATIONS` |

---

**© 2026 RubikCare — للاستخدام الداخلي**

**نهاية وثيقة `06-POINTS` ✅**
```

---

## 📘 `07-REPORTS` — نظام التقارير (وثيقة مستقلة)

```markdown
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

## 📘 `08-NOTIFICATIONS` — نظام الإشعارات (وثيقة مستقلة)

```markdown
# 08 — NOTIFICATIONS (نظام الإشعارات والتذكيرات)

**الإصدار:** 2.0 (Unified Edition)
**آخر تحديث:** 17 أكتوبر 2026
**الحالة:** 🟡 شغال — نقاط مفتوحة
**الجمهور:** فريق التطوير + AI Assistants
**المرجع الإلزامي:** يُقرأ قبل أي تعديل على الإشعارات

---

## 📌 الجزء صفر: نظرة عامة

### 0.1 نظامان منفصلان تماماً

**⚠️ مهم جداً: عدم الخلط بينهما.**

| # | النظام | الآلية | الحالة |
|:-:|:---|:---|:---:|
| **1** | **إشعارات الطلبات المهنية** (Push/FCM) | Firebase (FCM) — يحتاج إنترنت + سيرفر | ✅ شغال |
| **2** | **تذكيرات مواعيد الأدوية** (Local Alarms) | `Plugin.LocalNotification` — محلي على الجهاز | ✅ شغال — ملاحظتان مفتوحتان |
| **3** | **Web Push Notifications** (PWA) | Web Push + VAPID — Service Worker | ✅ شغال — (ملاحظة) |

**⚠️ 3 أنظمة منفصلة — ليس 2.**

---

## 📌 الجزء الأول: نظام إشعارات الطلبات المهنية (Push/FCM)

### 1.1 الحالة

✅ **شغال بالكامل.**

### 1.2 الوظيفة

- **إشعار للأدمن** — عند "طلب مهني" (فتح عيادة/صيدلية).
- **إشعار لصاحب الطلب** — عند الموافقة.
- **قابل للضغط** — التوجيه للصفحة المناسبة.

### 1.3 الملفات المسؤولة

```
Mobile\Services\MauiNotificationNavigationService.cs        ← تنفيذ التوجيه على الموبايل
Rubikcare.Web\Services\WebNotificationNavigationService.cs  ← تنفيذ التوجيه على الويب
Shared.UI\Services\INotificationNavigationService.cs        ← Interface مشترك
```

### 1.4 Commits ذات صلة

```
16129b9 — 2026-07-28 — ProfessionalMode Notification for Admin & user
a864b8b — 2026-07-26 — Push Notification After Add Member to Organization
b71c635 — 2026-07-26 — Push Notification From Fire BASE
```

**⚠️ لا حاجة للعمل على هذا النظام حاليًا — مغلق ومستقر.**

---

## 📌 الجزء الثاني: نظام تذكيرات مواعيد الأدوية (Local Alarms)

### 2.1 الحالة

✅ **الأساسيات شغالة — نقطتان مفتوحتان.**

### 2.2 البنية المعمارية

```
Shared.UI\Services\ILocalNotificationService.cs          ← Interface مشترك
Mobile\Infrastructure\Services\MauiLocalNotificationService.cs  ← التنفيذ (Mobile فقط)
Rubikcare.Web\Services\WebLocalNotificationService.cs     ← No-Op (الويب)
Mobile\MauiProgram.cs                                      ← Plugin + Channel
Mobile\Platforms\Android\MainActivity.cs                   ← الصلاحيات + Exact Alarm
Shared.UI\Components\Patient\SchedulePage.razor            ← نقطة الاستدعاء
```

### 2.3 آلية العمل

1. **أي تغيير في المواعيد** → `LoadAllData()`.
2. **`NotificationService.SyncReminders(_schedules)`** — في نهاية `LoadAllData()`.
3. **`SyncReminders`** — لكل موعد:
   - لو `Status == "Pending"` + في المستقبل → `ScheduleDoseReminder`.
   - غير كده → `CancelReminder`.
4. **`ScheduleDoseReminder`** — `NotifyTime = ScheduledTime - reminderMinutesBefore`.

### 2.4 الكود (مختصر)

**`ILocalNotificationService.cs`:**

```csharp
public interface ILocalNotificationService
{
    Task ScheduleDoseReminder(int scheduleId, string medicationName, string dosage, DateTime scheduledTime, int reminderMinutesBefore = 0);
    Task CancelReminder(int scheduleId);
    Task SyncReminders(IEnumerable<MedicationScheduleDto> schedules, int reminderMinutesBefore = 0);
}
```

**`MauiLocalNotificationService.cs`:**

```csharp
public class MauiLocalNotificationService : ILocalNotificationService
{
    public async Task ScheduleDoseReminder(...)
    {
        var notifyTime = scheduledTime.AddMinutes(-reminderMinutesBefore);
        if (notifyTime <= DateTime.Now) return;
        
        var notification = new NotificationRequest
        {
            NotificationId = scheduleId,
            Title = "💊 تذكير دواء",
            Description = body,
            CategoryType = NotificationCategoryType.Alarm,
            Schedule = new NotificationRequestSchedule { NotifyTime = notifyTime },
            Android = new AndroidOptions
            {
                ChannelId = "rubikcare_medication",
                Priority = AndroidPriority.High
            }
        };
        await LocalNotificationCenter.Current.Show(notification);
    }
    
    // ... Cancel + Sync
}
```

**`MauiProgram.cs` — التسجيل:**

```csharp
.UseLocalNotification(config =>
{
    config.AddAndroid(android =>
    {
        android.AddChannel(new NotificationChannelRequest
        {
            Id = "rubikcare_medication",
            Name = "تذكيرات الأدوية",
            Description = "تنبيهات مواعيد جرعات الأدوية",
            Importance = AndroidImportance.High,
            Sound = "custom_sound.wav", // ⚠️ ملاحظة
            LockScreenVisibility = AndroidVisibilityType.Public,
            VibrationPattern = new long[] { 0, 250, 250, 250 }
        });
    });
})
```

**الصلاحيات (AndroidManifest.xml):**

```xml
<uses-permission android:name="android.permission.POST_NOTIFICATIONS" />
<uses-permission android:name="android.permission.SCHEDULE_EXACT_ALARM" />
<uses-permission android:name="android.permission.USE_EXACT_ALARM" />
<uses-permission android:name="android.permission.FOREGROUND_SERVICE" />
<uses-permission android:name="android.permission.FOREGROUND_SERVICE_SPECIAL_USE" />
<uses-permission android:name="android.permission.RECEIVE_BOOT_COMPLETED" />
<uses-permission android:name="android.permission.VIBRATE" />
<uses-permission android:name="android.permission.WAKE_LOCK" />
```

### 2.5 ⚠️ النقاط المفتوحة

#### 🟡 نقطة 1: فرق التوقيت

**الوصف:** `reminderMinutesBefore = 0` افتراضي (من 10 سابقاً).

**المطلوب:**
- ✅ تأكيد فعلي (باختبار حقيقي) إن التنبيه يظهر في نفس الميعاد.
- لو لسه فيه فرق — مراجعة `Schedule.NotifyTime` + Cache.

#### 🔴 نقطة 2: عدم استقرار على ColorOS

**الوصف:**
- التنبيه اشتغل في أول اختبار.
- توقف في اختبارات لاحقة — نفس الكود.

**التشخيص:** ColorOS (Oppo/Realme/OnePlus) يقيّد Background Alarms.

**الحل المُقترَح:**

**1. تحقق يدوي أولاً:**
- `Settings → Battery → App Battery Management → RubikCare → No restrictions`.
- `Settings → App Management → Auto-launch → تفعيل RubikCare يدويًا`.

**2. لو المشكلة اتحلت → إضافة شاشة إرشادية داخل التطبيق.**

**3. لو لم تُحل → مراجعة `WakeLock` + `AlarmManager.setExactAndAllowWhileIdle` مباشرة.**

#### 🟢 ملاحظة إضافية

`Sound = "custom_sound.wav"` — ملف غير موجود فعليًا.

**الحل:** نسخ ملف صوت لـ `Mobile\Platforms\Android\Resources\raw\`.

### 2.6 خطة العمل

```
1. تأكيد التوقيت (نقطة 1)
2. التحقق اليدوي من إعدادات ColorOS (نقطة 2)
3. بناءً على النتيجة:
   - لو اتحل → إضافة شاشة إرشادية
   - لو لم يُحل → مراجعة أعمق
4. حسم موضوع ملف الصوت (اختياري)
5. إزالة زرار "Test Notification" قبل الإصدار
```

### 2.7 أداة التشخيص المؤقتة

**⚠️ `TestNotificationNow()` — يجب إزالتها قبل الإصدار.**

---

## 📌 الجزء الثالث: نظام Web Push (PWA)

### 3.1 نظرة عامة

**⚠️ من `21-Web-Push`:**

**نظام Web Push** = إشعارات المتصفح (Desktop + Mobile) عبر **Service Worker** + **VAPID**.

### 3.2 نظامان منفصلان

| النظام | الجدول | الاستخدام |
|:---|:---|:---|
| **FCM Native** | `DeviceTokens` | تطبيق MAUI |
| **Web Push** | `UserWebPushSubscriptions` | PWA |

### 3.3 ⚠️ المشكلة الحرجة: ملفان Service Worker

**⚠️ مشاريع Blazor WASM PWA فيها ملفان:**

| الملف | الاستخدام |
|:---|:---|
| `wwwroot/service-worker.js` | **Dev فقط** |
| `wwwroot/service-worker.published.js` | **القالب الفعلي** |

**⚠️ عند `dotnet publish` — يُحل `service-worker.published.js` محل `service-worker.js`.**

**⚠️ الخطأ الشائع:** التعديل في `service-worker.js` — يضيع مع `publish`.

### 3.4 الحل: `service-worker.published.js`

**⚠️ يجب التعديل هنا.**

### 3.5 الحالة

**✅ الإشعارات تعمل على اللابتوب والموبايل.**

**🟡 مشكلة علامات الاستفهام في النص العربي — قيد الفحص.**

### 3.6 التشخيص

**⚠️ تفاصيل كاملة في `21-Web-Push`.**

---

## 📌 الجزء الرابع: القواعد الذهبية

### 4.1 🔴 ممنوعات مطلقة

| # | الممنوع |
|:-:|:---|
| 1 | الخلط بين الأنظمة الثلاثة (FCM/Local/Web Push). |
| 2 | تعديل `service-worker.js` — عدّل `service-worker.published.js`. |
| 3 | نقل ملف الـ Service Worker وحده (لازم مجلد `wwwroot` كامل). |
| 4 | اختبار في المتصفح العادي بعد النشر — استخدم InPrivate. |
| 5 | افتراض `success: true` معناها الإشعار وصل — اعتمد على logs. |
| 6 | اختبار بمسار نسبي (`/api/...`) — استخدم URL كامل. |
| 7 | افتراض نجاح اللابتوب = نجاح الموبايل — إذن + اشتراك منفصلان. |

### 4.2 🟡 قواعد صارمة

| # | القاعدة |
|:-:|:---|
| 1 | `N''` + `MERGE` في SQL. |
| 2 | `@rendermode InteractiveServer` في صفحات Blazor. |
| 3 | Cache 30 دقيقة — تعديل DB يحتاج Restart. |
| 4 | `IsTokenValid` قبل إرسال التوكن. |

---

## 📌 الجزء الخامس: Troubleshooting

### 5.1 نظام Local Alarms

#### مشكلة: التنبيه لا يظهر

**الأسباب المحتملة:**
- **إعدادات الجهاز** (ColorOS/Xiaomi) تقيّد Background Alarms.
- **الصلاحيات** — `POST_NOTIFICATIONS` غير ممنوحة.
- **`USE_EXACT_ALARM`** غير موجود في Manifest.

**الحل:** تحقق من الإعدادات اليدوية.

#### مشكلة: التنبيه يظهر متأخراً

**الأسباب:**
- `reminderMinutesBefore = 10` (قيمة قديمة).
- Cache قديم.
- AlarmManager مقيد.

### 5.2 نظام Web Push

#### مشكلة: الإشعار الافتراضي "This site has been updated in the background"

**السبب:** الـ Service Worker الحالي لا ينادي `showNotification()`.

**الحل:**
1. تحديث Service Worker فعلياً.
2. إعادة تشغيل Chrome بالكامل.

#### مشكلة: علامات استفهام في النص العربي

**الأسباب المحتملة:**
- Encoding.ASCII في pipeline.
- Middleware مخصص.

**⚠️ الحالة:** قيد الفحص.

---

## 📌 الجزء السادس: Roadmap

### 6.1 ما هو مطلوب

| # | المهمة | الأولوية | الحالة |
|:-:|:---|:---:|:---:|
| 1 | **تأكيد توقيت Local Alarms** | 🔴 عالية | ⏳ |
| 2 | **حل مشكلة ColorOS** | 🔴 عالية | ⏳ |
| 3 | **حل مشكلة النص العربي في Web Push** | 🟠 متوسطة | ⏳ |
| 4 | **إضافة شاشة إرشادية للمستخدمين** | 🟠 متوسطة | ❌ |
| 5 | **حسم ملف الصوت المخصص** | 🟢 منخفضة | ❌ |
| 6 | **إزالة Test Notification** | 🟠 متوسطة | ❌ |
| 7 | **حل مشكلة Mobile translation** (من `04-TRANSLATION`) | 🟠 متوسطة | ⏳ |

### 6.2 الاعتمادات

- **نقطة 1** — مستقلة.
- **نقطة 2** — تحتاج اختبار ميداني.
- **نقطة 3** — تحتاج فحص pipeline.
- **نقطة 4** — تحتاج UI.
- **نقطة 7** — تحتاج جلسة منفصلة.

---

## 📌 سجل التغييرات

| الإصدار | التاريخ | التغييرات |
|:---:|:---|:---|
| **1.0** | **31 يوليو 2026** | الإصدار الأولي — `20-notifications-status.md` |
| **1.0** | **16 سبتمبر 2026** | الإصدار الأولي — `21-web-push.md` |
| **2.0** | **17 أكتوبر 2026** | **`08-NOTIFICATIONS` v2.0** — دمج `20` + `21` + إشارات |

---

## 📌 روابط ذات صلة

- [00 — MASTER-INDEX](./00-MASTER-INDEX.md)
- [00B — PROJECT-BACKLOG](./00B-PROJECT-BACKLOG.md)
- [01 — ARCHITECTURE](./01-ARCHITECTURE.md)
- [04 — TRANSLATION](./04-TRANSLATION.md)
- [06 — POINTS](./06-POINTS.md)
- [07 — REPORTS](./07-REPORTS.md)
- [14 — PWA](./14-PWA.md)

---

**© 2026 RubikCare — للاستخدام الداخلي**

**نهاية وثيقة `08-NOTIFICATIONS` ✅**
```

---

## 📌 ملاحظات

### ✅ ما تم إنجازه:

**3 وثائق كاملة:**

1. **`06-POINTS`** v7.0 — دمج `26` v6.0 + 4 migrations جديدة (14-15 أكتوبر).
2. **`07-REPORTS`** v2.0 — ترقيم جديد.
3. **`08-NOTIFICATIONS`** v2.0 — دمج `20` + `21` + إشارات.

### ⚠️ ما تم تصحيحه:

| # | التصحيح | في |
|---|--------|----|
| 1 | **`ProgramID` → `SourceEntityID`** | `06-POINTS` |
| 2 | **`OrganizationTypeFilterID`** (14 أكتوبر) | `06-POINTS` |
| 3 | **`PointSystemTypeOrganizationTypes`** (14 أكتوبر) | `06-POINTS` |
| 4 | **`AllowMultipleSchemesPerCampaign`** (15 أكتوبر) | `06-POINTS` |
| 5 | **`IncludeTargetOrgTypeInRuleUniqueIndex`** (15 أكتوبر) | `06-POINTS` |
| 6 | **3 أنظمة إشعارات** (ليس 2) | `08-NOTIFICATIONS` |
| 7 | **Web Push Service Worker المزدوج** | `08-NOTIFICATIONS` |

---

## 🎯 الخطوة التالية

**⚠️ الوثيقة التالية:**

- **A)** `11-PAGE-CREATION` — دمج `05` + `13`.
- **B)** `12-CACHING` — تحديث `14`.
- **C)** `13-MAUI` — دمج `10` + `11`.

**⚠️ توصيتي:**

**`11-PAGE-CREATION`** — لأنه أساسي + يدمج `13`.

**⏳ بانتظار قرارك.** 🚀
