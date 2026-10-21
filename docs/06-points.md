
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
---
## 🎯 الجزء الحادي عشر: دورة حياة النقاط (Points Lifecycle) — v3.0 (21 أكتوبر 2026)

> ⚠️ **هذا القسم يوثق التصميم الحالي بعد إصلاح `PatientID → CaseID` في جلسة #136.**
> **لا تعدل هذا القسم إلا بعد اختبار E2E كامل.**

### 11.1 المبادئ الأساسية (Core Principles)

#### 11.1.1 `PatientProfileID` = "الشخص" (Person)
- **لا يتكرر أبداً.** هو `UserProfileID` من `AspNetUsers`.
- هو المرجع الأساسي لأي عملية تخص "الشخص" (وليس "الحالة").

#### 11.1.2 `CaseID` = "الحالة" (Case)
- **يتكرر** لكل برنامج يشترك فيه الشخص.
- هو `PSPPatients.CaseID` (المفتاح الأساسي).
- **مثال:** shadyelzaher@rubikcare.com قد يكون لديه `CaseID=18` (Infertility) و `CaseID=1083` (pregnant case).

#### 11.1.3 `BeneficiaryID` في `PointsContext` = `PatientProfileID`
- **هذا هو المفتاح الحاسم.** `EarnPointsHandler` يبحث عن `Balance` باستخدام:
  - `BeneficiaryTypeID = 5` (Patient).
  - `BeneficiaryID = context.CaseID` (الذي يجب أن يكون `PatientProfileID`).
  - `SchemeID = scheme.SchemeID`.
- **النتيجة:** كل شخص لديه **رصيد واحد** لكل `Scheme`، بغض النظر عن عدد الـ `CaseID`.

#### 11.1.4 `Scheme` = "نظام النقاط" لكل برنامج
- كل `ProgramID` قد يكون له `Scheme` خاص (أو مشترك).
- **مثال:**
  - `Scheme 1` → `ProgramID = 1` (pregnant case) → 1 ج.م/نقطة.
  - `Scheme 7` → `ProgramID = 2` (Infertility) → 1 ج.م/نقطة.
  - `Scheme 6` → `ProgramID = 4` (devartpoint) → 5 ج.م/نقطة.

---

### 11.2 خريطة الأحداث (Activity Types) — 11 حدثاً

| ID | ActivityTypeCode | الاسم | يُمنح عند | السيناريو |
|:--:|------------------|-------|-----------|-----------|
| 1 | `PATIENT_INVITED` | دعوة مريض | إنشاء دعوة (Doctor → Patient) | `CreateInvitationAsync` |
| 2 | `PATIENT_ENROLLED` | تسجيل مريض | قبول الدعوة | `AcceptInvitationAsync` / `EnrollUserInProgramAsync` |
| 3 | `ERX_CREATED` | إنشاء روشتة | إنشاء `PSPeRX` | `AcceptInvitationAsync` (لكل دواء) |
| 4 | `DISPENSATION_COMPLETED` | إتمام صرف | `ConfirmDispense` / `ScanAndDispense` | `DispenseController` |
| 5 | `REFILL_COMPLETED` | تكرار صرف | `ConfirmDispense` (RefillNumber > 0) | `DispenseController` |
| 6 | `PROGRAM_COMPLETED` | إكمال البرنامج | ⏸️ مؤجل | — |
| 7 | `LAB_TEST_UPLOADED` | رفع تحليل طبي | ⏸️ مؤجل | — |
| **8** | **`DAILY_DOSE_TAKEN`** | **تأكيد تناول الجرعة** | **`UpdateScheduleStatusAsync` (Status = "Taken")** | **`PatientMedicationService`** |
| 9 | `CONTENT_VIEWED_VIDEO` | مشاهدة فيديو تعليمي | ⏸️ مؤجل | — |
| 10 | `CONTENT_READ_ARTICLE` | قراءة مقال تعليمي | ⏸️ مؤجل | — |
| 11 | `CONTENT_DOWNLOADED` | تحميل ملف تعليمي | ⏸️ مؤجل | — |

---
---

### 11.3 تدفق الكسب (Earning Flow)

#### 11.3.1 حدث `DAILY_DOSE_TAKEN` (نقاط التناول اليومي)


1. المريض يضغط "تم التناول" في MedicationSchedulePage
   ↓
2. PUT /api/patient/medications/status
   {
     "ScheduleID": 12345,
     "Status": "Taken",
     "TakenDateTime": "2026-10-21T15:30:00"
   }
   ↓
3. PatientMedicationsController.UpdateScheduleStatus
   ↓
4. PatientMedicationService.UpdateScheduleStatusAsync
   ↓
5. if (dto.Status == "Taken")
     ↓
6. TryAwardDailyDosePointsAsync(schedule)
     ├── جلب PSPPatient حيث PatientProfileID = schedule.PatientProfileID
     │   و Participation.ProgramID = schedule.ProgramID
     │   → targetPatient (CaseID = 1083, ProfileID = 15)
     │
     ├── بناء PointsContext:
     │   OrganizationID = targetPatient.InvitedByOrganizationID (1267)
     │   CaseID = targetPatient.PatientProfileID (15)  ← مهم جداً
     │   UserID = targetPatient.PatientProfileID (15)
     │
     └── AwardByCampaignAsync(
           campaignId: schedule.ProgramID (1),
           pointSystemTypeID: 1,
           activityTypeId: 8 (DAILY_DOSE_TAKEN),
           context,
           referenceType: "SCHEDULE",
           referenceId: schedule.ScheduleID
         )
         ↓
7. EarnPointsOrchestrator.AwardAsync
   ├── جلب Scheme حيث SourceEntityID = 1 و PointSystemTypeID = 1
   │   → Scheme 1 (PSP.CUP)
   ├── جلب Rules النشطة حيث SchemeID = 1 و ActivityTypeID = 8
   │   → Rule (Points = 5, BeneficiaryTypeID = 5)
   ├── context.GetBeneficiaryIdFor(5) → PatientID = 15
   └── EarnPointsHandler.HandleAsync(...)
       ├── جلب/إنشاء Balance حيث BeneficiaryID = 15, SchemeID = 1
       ├── Balance += 5
       └── إنشاء PSPProgramPointTransaction (Points = +5)

---
**⚠️ نقاط حساسة:**
- **`CaseID = PatientProfileID`** في `PointsContext` (وليس `CaseID` من `PSPPatient`).
- **`Scheme` يُحدد تلقائياً** من `campaignId` (وليس من `CaseID`).
- **`BeneficiaryID` في `Balance`** هو `PatientProfileID` (وليس `CaseID`).

#### 11.3.2 حدث `DISPENSATION_COMPLETED` (نقاط الصرف)

```
1. الصيدلي يضغط "تأكيد الصرف" أو "مسح QR"
   ↓
2. POST /api/dispense/confirm أو POST /api/dispense/scan-and-dispense
   ↓
3. DispenseController.ConfirmDispense / ScanAndDispense
   ├── تحديث PSPDispensation.Status = "USED"
   ├── حساب PSPCoverage و PatientPayment
   └── TryAwardPointsViaOrchestratorAsync(
         patientId: eRx.PatientID,        ← CaseID
         programId: eRx.ProgramID,
         activityTypeId: 4 (DISPENSATION_COMPLETED),
         referenceType: "DISPENSATION",
         referenceId: dispensation.DispensationID
       )
       ↓
4. (نفس الخطوة 7 أعلاه) → منح النقاط
```

#### 11.3.3 حدث `REFILL_COMPLETED` (نقاط تكرار الصرف)

```
(نفس DISPENSATION_COMPLETED، لكن:)
- activityTypeId = 5 (REFILL_COMPLETED)
- الشرط: (dispensation.RefillNumber ?? 0) > 0
```

#### 11.3.4 أحداث `PATIENT_ENROLLED` و `ERX_CREATED` (نقاط التسجيل)

```
1. المريض يقبل الدعوة
   ↓
2. POST /api/psp/accept-invitation
   ↓
3. PSPEnrollmentService.AcceptInvitationAsync
   ├── إنشاء/جلب PSPPatient (CaseID)
   ├── TryAwardPointsViaOrchestratorAsync(
   │     activityTypeId: 2 (PATIENT_ENROLLED),
   │     referenceType: "PATIENT",
   │     referenceId: pspPatient.CaseID
   │   )
   ├── إنشاء PSPeRX لكل دواء
   └── TryAwardPointsViaOrchestratorAsync(
         activityTypeId: 3 (ERX_CREATED),
         referenceType: "ERX",
         referenceId: eRx.ERxID
       ) ← لكل eRx
```

---

### 11.4 تدفق الاستخدام (Redeeming Flow)

#### 11.4.1 حدث `REDEEM_DISPENSATION` (خصم النقاط على الصرف)

```
1. المريض يضغط "استخدم نقاطي" في PspDetail
   ↓
2. POST /api/dispense/apply-points (الخيار ب — التوكن الحالي)
   أو
   POST /api/dispense/generate-next-token (مع PointsToRedeem > 0)
   ↓
3. DispenseController.ApplyPointsToCurrentToken / GenerateNextToken
   ├── جلب Scheme (SourceEntityID = programId)
   ├── جلب Balance (BeneficiaryID = patientId, SchemeID = scheme.SchemeID)
   ├── التحقق: Balance >= PointsToRedeem
   ├── حساب Discount = PointsToRedeem × Scheme.MonetaryValuePerPoint
   ├── Balance -= PointsToRedeem
   ├── TotalRedeemed += PointsToRedeem
   ├── إنشاء PSPProgramPointTransaction (
   │     TransactionTypeID: 9 (REDEEM_DISPENSATION),
   │     Points: -PointsToRedeem,
   │     ReferenceType: "DISPENSATION",
   │     ReferenceID: newDispensation.DispensationID
   │   )
   └── تحديث PSPDispensation:
         PSPCoverage += Discount
         PatientPayment -= Discount
```

**⚠️ قواعد صارمة:**
- `TransactionTypeID = 9` (REDEEM_DISPENSATION) — **وليس 2 (REDEEM_REQUEST)**.
- `ReferenceType = "DISPENSATION"` + `ReferenceID = DispensationID`.
- **Idempotency:** التحقق من `AnyAsync(ReferenceType = "DISPENSATION" && ReferenceID = id && TransactionTypeID = 9)` قبل الخصم.

#### 11.4.2 `RedeemRequest` (طلب استرداد نقدي) — منفصل

- هذا **ليس** خصم على الدواء.
- هو طلب استرداد نقدي (Cash Redeem).
- يستخدم `TransactionTypeID = 2, 3, 4, 5, 6, 7, 8` (حسب حالة الطلب).
- **لا تخلط بينه وبين `REDEEM_DISPENSATION`**.

---

### 11.5 مخططات التكامل (Integration Diagrams)

#### 11.5.1 منح النقاط (Earning)

```
┌──────────────────────────────────────────────────────────────┐
│  PSP Event (enrollment, dispensation, daily dose)            │
└──────────────┬───────────────────────────────────────────────┘
               │
               ▼
┌──────────────────────────────────────────────────────────────┐
│  TryAwardPointsViaOrchestratorAsync / TryAwardDailyDose...   │
│  (يحسب PointsContext + BeneficiaryID)                        │
└──────────────┬───────────────────────────────────────────────┘
               │
               ▼
┌──────────────────────────────────────────────────────────────┐
│  IEarnPointsOrchestrator.AwardByCampaignAsync                │
│  (يجد Scheme + Rules)                                        │
└──────────────┬───────────────────────────────────────────────┘
               │
               ▼
┌──────────────────────────────────────────────────────────────┐
│  EarnPointsHandler.HandleAsync                               │
│  (ينشئ/يحدّث Balance + PSPProgramPointTransaction)           │
└──────────────┬───────────────────────────────────────────────┘
               │
               ▼
┌──────────────────────────────────────────────────────────────┐
│  PSPProgramPointBalance  +  PSPProgramPointTransaction       │
│  (Balance += Points, Transaction.Points = +Points)           │
└──────────────────────────────────────────────────────────────┘
```

#### 11.5.2 استخدام النقاط (Redeeming)

```
┌──────────────────────────────────────────────────────────────┐
│  Patient clicks "استخدم نقاطي"                                │
└──────────────┬───────────────────────────────────────────────┘
               │
               ▼
┌──────────────────────────────────────────────────────────────┐
│  POST /api/dispense/apply-points                             │
│  أو POST /api/dispense/generate-next-token (PointsToRedeem)  │
└──────────────┬───────────────────────────────────────────────┘
               │
               ▼
┌──────────────────────────────────────────────────────────────┐
│  Atomic Transaction (BeginTransaction + Commit)              │
│  ├── قفل Balance (UPDLOCK)                                   │
│  ├── Idempotency Check                                       │
│  ├── Balance -= PointsToRedeem                               │
│  ├── TotalRedeemed += PointsToRedeem                         │
│  ├── Transaction (Type=9, ReferenceType="DISPENSATION")      │
│  ├── Update PSPDispensation (PSPCoverage += Discount)        │
│  └── Update PSPDispensation (PatientPayment -= Discount)     │
└──────────────┬───────────────────────────────────────────────┘
               │
               ▼
┌──────────────────────────────────────────────────────────────┐
│  PSPProgramPointBalance  +  PSPProgramPointTransaction       │
│  (Balance -= Points, Transaction.Points = -Points)           │
└──────────────────────────────────────────────────────────────┘
```

---

### 11.6 قواعد Idempotency (منع التكرار)

| # | العملية | المفتاح | كيف نمنع التكرار |
|---|---------|---------|-------------------|
| 1 | `DAILY_DOSE_TAKEN` | `SCHEDULE/{scheduleId}` | التحقق: `AnyAsync(ReferenceType="SCHEDULE" && ReferenceID=scheduleId && ActivityTypeID=8)` |
| 2 | `DISPENSATION_COMPLETED` | `DISPENSATION/{dispensationId}` | التحقق: `AnyAsync(ReferenceType="DISPENSATION" && ReferenceID=dispensationId && ActivityTypeID=4)` |
| 3 | `REFILL_COMPLETED` | `DISPENSATION/{dispensationId}` | التحقق: `AnyAsync(ReferenceType="DISPENSATION" && ReferenceID=dispensationId && ActivityTypeID=5)` |
| 4 | `PATIENT_ENROLLED` | `PATIENT/{caseId}` | التحقق: `AnyAsync(ReferenceType="PATIENT" && ReferenceID=caseId && ActivityTypeID=2)` |
| 5 | `REDEEM_DISPENSATION` | `DISPENSATION/{dispensationId}` | التحقق: `AnyAsync(ReferenceType="DISPENSATION" && ReferenceID=dispensationId && TransactionTypeID=9)` |

**⚠️ القاعدة:** أي حدث يخصم أو يمنح نقاطاً، **يجب** أن يكون له Idempotency Check.

---

### 11.7 المشاكل المعروفة والحلول (Troubleshooting)

#### مشكلة 1: "لا توجد قواعد نشطة لهذا الحدث"

**الأعراض:**
- `AwardByCampaignAsync` يرجع `TotalAwarded = 0`.
- الرسالة: "لا توجد قواعد نشطة لهذا الحدث".

**الأسباب المحتملة:**
1. `Rule` مفقود لـ `ActivityTypeID` في `Scheme` المحدد.
2. `Rule.IsActive = 0`.
3. `Rule.Points <= 0`.

**الحل:**
```sql
-- التحقق من Rules
SELECT RuleID, ActivityTypeID, Points, BeneficiaryTypeID, IsActive
FROM PSPProgramPointRules
WHERE SchemeID = @SchemeId AND ActivityTypeID = @ActivityTypeId;

-- إذا لم توجد، أضف:
INSERT INTO PSPProgramPointRules 
    (SchemeID, ActivityTypeID, Points, BeneficiaryTypeID, IsActive, DisplayOrder, CreatedDate)
VALUES 
    (@SchemeId, @ActivityTypeId, @Points, 5, 1, 100, GETDATE());
```

#### مشكلة 2: "النقاط لا تزيد بعد تأكيد التناول"

**الأعراض:**
- المريض يؤكد التناول، لكن الرصيد لا يتغير.

**الأسباب المحتملة:**
1. `schedule.ProgramID` مختلف عن `Scheme.SourceEntityID` (مثلاً، الموعد للبرنامج 1، لكن `Scheme 7` للبرنامج 2).
2. `PointsContext.CaseID` خطأ (يجب أن يكون `PatientProfileID`).
3. `Balance` غير موجود (يجب أن يُنشأ تلقائياً).

**الحل:**
- تحقق من `schedule.ProgramID` → `Scheme.SourceEntityID`.
- تحقق من `PSPProgramPointBalances` للمريض.
- راجع سجلات `dailydose_debug.log`.

#### مشكلة 3: "النقاط تذهب لمريض آخر"

**الأعراض:**
- المريض A يؤكد التناول، لكن النقاط تذهب لمريض B.

**السبب:**
- `PointsContext.CaseID` يستخدم `FirstOrDefault()` بدلاً من `FirstOrDefault(p => p.Participation.ProgramID == schedule.ProgramID)`.

**الحل:**
```csharp
var targetPatient = pspPatients.FirstOrDefault(p =>
    p.Participation != null &&
    p.Participation.ProgramID == schedule.ProgramID.Value);
```

#### مشكلة 4: "الخصم على الدواء لا يظهر في الفاتورة"

**الأعراض:**
- المريض استخدم النقاط، لكن `PatientPayment` لم ينخفض.

**السبب:**
- `ConfirmDispense` يعيد حساب `PatientPayment` من الصفر (بدون احترام `PointsDiscount`).

**الحل:**
- اقرأ `PSPProgramPointTransaction` حيث `ReferenceType="DISPENSATION" && ReferenceID=dispensationId && TransactionTypeID=9`.
- احسب `pointsDiscount = (-transaction.Points) × scheme.MonetaryValuePerPoint`.
- اطرح `pointsDiscount` من `PatientPayment` وأضفه إلى `PSPCoverage`.

#### مشكلة 5: "زر استخدم نقاطي لا يظهر"

**الأسباب المحتملة:**
1. `TotalPoints = 0` (لا يوجد رصيد).
2. `PointsUsedInCurrentToken = true` (تم استخدام النقاط بالفعل).
3. `ShowRedeemCard = false` (منطق الزر في `PspDetailComponent`).

**الحل:**
- تحقق من `PatientProgramDetailsResponse` (TotalPoints, PointsUsedInCurrentToken).
- تحقق من `ShowRedeemCard` في `PspDetailComponent.razor`.

---

### 11.8 Checklist لأي تعديل مستقبلي على نظام النقاط

قبل أي تعديل على نظام النقاط، تأكد من:

- [ ] قراءة هذا القسم بالكامل.
- [ ] فهم `PatientProfileID` vs `CaseID` vs `BeneficiaryID`.
- [ ] التحقق من وجود `Scheme` و `Rules` لكل حدث.
- [ ] إضافة Idempotency Check لكل خصم/منح.
- [ ] استخدام `TransactionTypeID` الصحيح:
  - `1` = EARN.
  - `9` = REDEEM_DISPENSATION.
  - `2-8` = REDEEM_REQUEST lifecycle.
- [ ] استخدام `ReferenceType` + `ReferenceID` الصحيحين:
  - `"DISPENSATION"` + `DispensationID` (للصرف).
  - `"SCHEDULE"` + `ScheduleID` (للتناول).
  - `"PATIENT"` + `CaseID` (للتسجيل).
  - `"ERX"` + `ERxID` (للروشتة).
- [ ] اختبار E2E لكل حدث (Earning + Redeeming).
- [ ] تحديث `dailydose_debug.log` (إذا لزم).

---

---

| 11.9 سجل التغييرات | ✅ |

**⏭️ الخطوة التالية:** انسخ القسم إلى `06-POINTS.md`، ثم قل لي إذا كنت تريد أن أرسل تعديلات `01-ARCHITECTURE.md` و `20-USER-JOURNEY.md` و `00B-PROJECT-BACKLOG.md`. 🚀

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
| **3.0** | **21 أكتوبر 2026** | **إضافة "دورة حياة النقاط" (11.1-11.9) + إصلاح `PatientID → CaseID` + `DAILY_DOSE_TAKEN` + Idempotency + Troubleshooting** |

---

**© 2026 RubikCare — للاستخدام الداخلي**

**نهاية وثيقة `06-POINTS` ✅**
```

---

