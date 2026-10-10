## 📄 وثيقة 26: `26-rubika-points-system.md`

**المحتوى الكامل (نسخة جديدة — مع نصوصك الحرفية):**

```markdown
# 26 — نظام النقاط وروبيكا (Rubika Points & Currency System)

**آخر تحديث:** 10 أكتوبر 2026
**الإصدار:** 5.0 (Vision Edition — بعد توثيق Backend الكامل)
**الحالة:** ✅ Backend مكتمل — Frontend قيد الانتظار
**المؤلف:** فريق RubikCare

---

## 📌 الجزء صفر: الفلسفة والرؤية (نص المؤلف الحرفي)

> **⚠️ ملاحظة:**
> هذا القسم يحتوي على **نص الكاتب الحرفي** — دون إعادة صياغة.
> الهدف: حفظ الرؤية الكاملة كما صيغت.

---

### 0.1 الرؤية العامة لنظام النقاط

> **نص المؤلف الحرفي (من جلسة 10 أكتوبر 2026):**
> 
> "نظام النقاط مصمم ليتم إضافته ليس لخدمة ما ولكن لحملة تسويقية داخل الخدمة تنشأها المنظمة المستفيدة من الخدمة. على سبيل المثال كل برنامج دعم سنعتبر حملة تسويقية. وستجد أن النظام الحالي مرتبط بشكل منفصل بكل برنامج."

**الترجمة التقنية:**
- **الرابط الفعلي** = `ProgramID` (أو `SchemeID` = نظام نقاط محدد).
- **`PointSystemType`** = تصنيف مساعد (PSP, B2B, B2C, ...)، ليس رابطاً.
- **`TargetBeneficiaryTypeID`** = اختياري، يحدد نوع المستفيد المستهدف في Scheme.

---

### 0.2 منطق الاستفادة من نظام النقاط كنظام مستقل

> **نص المؤلف الحرفي:**
> 
> "أولاً منطق الاستفادة من نظام النقاط كنظام مستقل عن أي خدمة مقدمة أو عدة خدمات.
> 
> فمثلاً من المخطط له عمل نظام إدارة عيادات باشتراكات متنوعة، فيتم إلحاق نظام النقاط كميزة ملحقة فيه.
> 
> من المخطط عمل شبكة دعم لوجيستي كاملة متعددة المراحل، ما بين شركات الأدوية ومخازن جملة ثم مخازن تجزئة ثم من الأخيرة إلى الصيدليات ثم من الصيدليات إلى المستهلك.
> 
> فكل نقطة مما ذكرت سيتم إدارة وتغير value chain وتدفق عمليات وأنشطة تجارية وتسويقية تتناسب وطبيعة كل خطوة.
> 
> ومطلوب إدراج نظام النقاط في كل عملية بينية من ال saler to payer in every stage.
> 
> كذلك من المقرر تطوير وتفعيل Medical CRM لإدارة علاقات العملاء ومندوبي الدعاية medical reps visits to doctors & MR calls & visits plans وغيرها.
> 
> ويمكن تحويل نظام النقاط من قبل الشركة إلى نظام حساب KPI's for MR's، point system for doctors & pharmacies، وهكذا والأمثلة متعددة."

**الترجمة التقنية:**
- نظام النقاط **بنية أفقية** — يخدم أي خدمة.
- كل خدمة تُنشئ **كياناتها الخاصة** (Programs, Campaigns, Orders, ...).
- النقاط تُمنح في **كل مرحلة** (saler → payer).
- المستفيد يختلف حسب المرحلة (`BeneficiaryTypeID` ديناميكي).

---

### 0.3 Rubika Currency — النظام الموازي

> **نص المؤلف الحرفي:**
> 
> "وكما شرحت أنت من قبل أن هناك نظام موازي من المنصة لكل العملاء والمنظمات في كل الخدمات.
> 
> سيستخدم أحياناً كوحدة خدمية في حالة المحاسبة بنظام العمولة.
> 
> يقوم الطبيب بشحن رصيد روبيكا فيتمكن من استقبال طلبات الحجز أون لاين.
> 
> ويقوم الصيدلي بشحن رصيد روبيكا فيتمكن من استقبال طلبات شراء دواء.
> 
> ويمكن للمنصة أن تمنح لكل منهما نقاط مقابل referring platform to others أو أي نشاط يدعم توسع عمل المنصة وانتشارها تسويقياً.
> 
> وكذلك يتم منح النقاط للمستخدم العادي أياً كان دوره أو حتى لو لم يكن له دور فهو في النهاية مريض محتمل.
> 
> حيث أن المرض في المنصة ليس دور ولكنه حالة مؤقتة قد يمر بها الطبيب والمندوب والصيدلي وغيره.
> 
> فيحصل مقابل أنشطة تحددها المنصة ويمكنه تحويلها من رصيده إلى العيادات للحصول على خصم مقابلها أو للصيدلي للحصول على خصم مقابل شراء الأدوية."

**الترجمة التقنية:**
- **Rubika Currency** = نظام موازٍ (طبقة مستقلة).
- **المرض ليس دوراً** — بل حالة.
- **كل مستخدم** = مريض محتمل.
- **التحويل** بين الأدوار مدعوم.

---

### 0.4 منطق الحملات التسويقية

> **نص المؤلف الحرفي:**
> 
> "أنا أقترح أيضاً أن يتم ربط نظام النقاط في كل خدمة بحملة تسويقية معينة يتم إنشاؤها من قبل المنظمة المستفيدة.
> 
> وهنا عند الرغبة في عمل نظام نقاط، يظهر لمنظمة أ مثلاً مشتركة معنا في نظام PSP - B2B - CRM أن تقرر تصميم حملات تسويقية في كل قطاع تستخدمه.
> 
> وعند إنشاء نظام نقاط يظهر لها قائمة منسدلة تحدد الخدمة المطلوب ربطها، ثم داخل الخدمة نفسها يظهر لها الحملات التسويقية الحالية فيتم اختيار حملة ما."

**الترجمة التقنية:**
- **حملة تسويقية** = كيان داخل كل خدمة.
- **PSP**: الحملة = `PSPProgram`.
- **B2B**: الحملة = `PurchaseOrder` / `B2BOrder`.
- **CRM**: الحملة = `MRCampaign`.
- **الربط**: `Scheme.ProgramID` (أو `Scheme.SourceEntityID` مستقبلاً).

---

### 0.5 تعدد Schemes داخل نفس البرنامج

> **نص المؤلف الحرفي:**
> 
> "مطلوب أن تتمكن المنظمة من وضع عدة أنظمة نقاط schema داخل نفس البرنامج.
> 
> فمثلاً في برامج الدعم تتمكن من تصميم نظام للعيادات - الصيدليات - المرضى - العاملين في أي منظمة وفق نوعها.
> 
> العاملين بها (المندوبين الداخليين المستخدمين للمنصة)."

**الترجمة التقنية:**
- **`TargetBeneficiaryTypeID`** في `PSPProgramPointScheme` (nullable).
- **Filtered Unique Indexes**:
  - `(ProgramID) UNIQUE WHERE TargetBeneficiaryTypeID IS NULL` (عام).
  - `(ProgramID, TargetBeneficiaryTypeID) UNIQUE WHERE TargetBeneficiaryTypeID IS NOT NULL` (مخصص).
- **Rules** تُحدد `BeneficiaryTypeID` لكل حدث.

---

### 0.6 الاستقلالية والرؤية الأفقية لكل المنصة

> **نص المؤلف الحرفي:**
> 
> "ما أفكر فيه هو تصميم المشروع كله بهذا النمط. بحيث يمكن إعادة استخدام أي ميزة لاحقاً مثلاً التنبيهات، الإشعارات، الترجمة، المحادثات، طبقة الAI عند إضافتها لاحقاً.
> 
> كذلك نظام إدارة الفواتير والطلبات والمخزون والفوترة وصفحات checkout في كل المنصة. بمبدأ إعادة الاستخدام وعدم تكرار العمل، فيصبح إضافة ميزة جديدة بمثابة إضافة ثروة مخزنة يعاد استخدامها. وهو فكر مشابه لفكرة miniservices ولكن بنمط مختلف بعض الشيء ربما.
> 
> كذلك يمكن من خلال هذا الفكر تطوير مكتبة API client لإعادة استخدام هذه القطع الخدمية المستقلة داخلياً وخارجياً."

**الترجمة التقنية:**
- **Vertical Slices**: كل خدمة = Module مستقل.
- **Reusable Services**: Notifications, Translation, Messaging, Points, Invoicing, Orders, Inventory, Checkout, AI.
- **API Client Layer**: `IXxxClient` Interfaces (In-Process + HTTP).

---

### 0.7 إدارة الجداول المرجعية

> **نص المؤلف الحرفي (من جلسة 10 أكتوبر 2026):**
> 
> "ملاحظتي الآن أن هناك جداول كثيرة تم إضافتها وكلها seed data، ولم نضع خطة لإضافة واجهات لتحديث هذه الجداول ببيانات تشغيلية."

**الترجمة التقنية:**
- **الجداول التي تحتاج واجهة إدارية:**
  - `PSPActivityTypes` (7 قيم).
  - `PointSystemTypes` (5 قيم).
  - `PointBeneficiaryTypes` (6 قيم).
  - `PointSystemTypeBeneficiaryTypes` (M:N).
- **الجمهور:** Super Admin (فريق RubikCare).
- **المكان:** لوحة تحكم المنصة (`/admin/`).
- **الأولوية:** 🟠 متوسطة (بعد Frontend UI).

---

## 🎯 الجزء الأول: نظرة عامة تقنية

**نظام النقاط وروبيكا** = نظامان، محرك واحد.

| البُعد | Program Points | Rubika Currency |
|--------|:--------------:|:---------------:|
| الطبيعة | مكافآت تحفيزية | عملة داخلية |
| المُصدر | شركة / مؤسسة | RubikCare |
| الدورة | كسب → صرف | شحن → استخدام |
| الطبقة | Services | Value |
| الجداول | `PSPProgramPoint*` | `RubikaWallet` + `RubikaTransactions` |

**المبدأ:** **نفس المحرك (`IPointEngineService`)، تنفيذان مختلفان.**

---

## 🎯 الجزء الثاني: البنية المعمارية

### 2.1 المستويات الأربعة

```
Layer 1: Reference Layer
  ├── PointSystemTypes (5 قيم Seed)
  ├── PointActivityTypes (7 قيم Seed)
  ├── PointBeneficiaryTypes (6 قيم Seed)
  └── PSPProgramPointTransactionTypes (4 قيم Seed)

Layer 2: Beneficiary Layer
  ├── PointSchemeBeneficiaryTypes (M:N)
  └── PointSystemTypeBeneficiaryTypes (M:N)

Layer 3: Organization Structure
  ├── Organizations
  ├── OrganizationGroups
  └── OrganizationGroupMembers

Layer 4: Value Engines
  ├── Program Points Engine (PSPProgramPoint*)
  └── Rubika Currency Engine (RubikaWallet + RubikaTransactions)
```

### 2.2 الطبقات الأربع (مبسّطة)

| الطبقة | المسؤولية | الجداول الأساسية |
|--------|-----------|-----------------|
| **Reference** | التصنيفات | PointSystemTypes, PointActivityTypes, PointBeneficiaryTypes |
| **Beneficiary** | من يستفيد | PointSchemeBeneficiaryTypes, PointSystemTypeBeneficiaryTypes |
| **Structure** | المؤسسات | Organizations, OrganizationGroups |
| **Value** | المحركات | PSPProgramPoint*, RubikaWallet, RubikaTransactions |

---

## 🎯 الجزء الثالث: المستفيدون (PointBeneficiaryTypes)

### 3.1 القيم الست

| ID | Code | NameAr | IsOrganization |
|:--:|------|--------|:--------------:|
| 1 | ORGANIZATION | مؤسسة | ✅ |
| 2 | ORG_MEMBER_INTERNAL | عضو داخلي | ❌ |
| 3 | ORG_MEMBER_EXTERNAL | عضو خارجي | ❌ |
| 4 | REGULAR_USER | مستخدم عادي | ❌ |
| 5 | PATIENT | مريض | ❌ |
| 6 | GENERIC | عام | ❌ |

### 3.2 Constants

**الموقع:** `RubikCare.Domain/Constants/PSP/Points/PointBeneficiaryTypeIds.cs`

```csharp
public const int Organization = 1;
public const int OrgMemberInternal = 2;
public const int OrgMemberExternal = 3;
public const int RegularUser = 4;
public const int Patient = 5;
public const int Generic = 6;
```

**⚠️ ملاحظة:** هذه القيم **6 قيم نهائية** — مذكورة في الوثيقة الأصلية. **لا تُعدّل** إلا بـ Migration.

---

## 🎯 الجزء الرابع: Backend المُنفَّذ

### 4.1 المكوّنات الجديدة

| # | المكوّن | الموقع | الدور |
|:-:|--------|--------|-------|
| 1 | `EarnPointsCommand` | `Application/UseCases/PSP/Points/Earning/` | الأمر (BeneficiaryTypeID + BeneficiaryID + SchemeID) |
| 2 | `EarnPointsHandler` | `Infrastructure/UseCases/PSP/Points/Earning/` | التنفيذ الديناميكي |
| 3 | `PointsContext` | `Application/UseCases/PSP/Points/Earning/` | سياق الحدث |
| 4 | `IEarnPointsOrchestrator` + `EarnPointsOrchestrator` | Application + Infrastructure | الموزّع |
| 5 | `IPointEngineService` + `PointEngineService` | Application + Infrastructure | **Facade الموحّد** |

### 4.2 نمط العمل

```
PSPEnrollmentService / DispenseController
        ↓
TryAwardPointsViaOrchestratorAsync
        ↓
PointsContext (OrganizationID + PatientID + UserID)
        ↓
IEarnPointsOrchestrator.AwardByProgramAsync
        ↓
اقرأ Rules النشطة (SchemeID + ActivityTypeID)
        ↓
لكل Rule (BeneficiaryTypeID مختلف)
        ↓
IEarnPointsHandler.HandleAsync
        ↓
Balance + Transaction
```

**الفائدة:**
- **ديناميكي:** كل Rule يُطبَّق على المرشح المناسب.
- **مرن:** الشركة تُحدد من يستفيد (Rules).
- **قابل للتوسع:** إضافة BeneficiaryTypeID جديد = إضافة Rule.

### 4.3 الـ Facade المُوحَّد (`IPointEngineService`)

```csharp
public interface IPointEngineService
{
    // منح نقاط — فردي
    Task<EarnPointsResponse> EarnAsync(EarnPointsRequest request, CancellationToken ct = default);

    // منح نقاط — متعدد (Orchestrator)
    Task<OrchestratorResult> EarnMultiAsync(EarnMultiRequest request, CancellationToken ct = default);

    // جلب رصيد
    Task<GetBalanceResponse> GetBalanceAsync(GetBalanceRequest request, CancellationToken ct = default);

    // سجل المعاملات
    Task<GetTransactionsResponse> GetTransactionsAsync(GetTransactionsRequest request, CancellationToken ct = default);
}
```

**الاستخدام:**
- **العملاء الخارجيون** (PSPEnrollmentService, DispenseController) يستخدمون **الـ Handlers مباشرة** (للأداء).
- **العملاء الجدد** يستخدمون **`IPointEngineService`** (للبساطة).

---

## 🎯 الجزء الخامس: Migration المُنفَّذ

### 5.1 Migration `AddTargetBeneficiaryTypeToScheme`

**ما تم:**

| # | التعديل |
|:-:|--------|
| 1 | إضافة `TargetBeneficiaryTypeID` (`int?`) إلى `PSPProgramPointSchemes` |
| 2 | إزالة UNIQUE Index القديم على `ProgramID` |
| 3 | إضافة UNIQUE Filtered Index `IX_...ProgramID_General` |
| 4 | إضافة UNIQUE Filtered Index `IX_...ProgramID_TargetBeneficiary` |
| 5 | إضافة Index عادي على `TargetBeneficiaryTypeID` |
| 6 | إضافة FK إلى `PointBeneficiaryTypes` |

### 5.2 الفهارس الحالية

```sql
-- عام
IX_PSPProgramPointSchemes_ProgramID_General
  UNIQUE WHERE [TargetBeneficiaryTypeID] IS NULL

-- مخصص
IX_PSPProgramPointSchemes_ProgramID_TargetBeneficiary
  UNIQUE (ProgramID, TargetBeneficiaryTypeID) WHERE [TargetBeneficiaryTypeID] IS NOT NULL

-- FK
IX_PSPProgramPointSchemes_TargetBeneficiaryTypeID
```

---

## 🎯 الجزء السادس: خطة Frontend

### 6.1 واجهات إدارة Schemes

**الملف المستهدف:** `PSPStep2b_PointScheme.razor`

**التحديثات:**
- اختيار `TargetBeneficiaryTypeID` (nullable).
- عرض Schemes المتعددة لنفس Program.
- Rules متعددة لكل Scheme.

### 6.2 واجهات إدارة الجداول المرجعية

**الجمهور:** Super Admin.

**الواجهات المقترحة:**
- إدارة `PSPActivityTypes`.
- إدارة `PointSystemTypes`.
- إدارة `PointBeneficiaryTypes`.
- إدارة `PointSystemTypeBeneficiaryTypes`.

**الأولوية:** 🟠 متوسطة — تُنفَّذ بعد Frontend UI الرئيسي.

### 6.3 واجهات OrganizationGroups

**الجمهور:** Super Admin.

**الفكرة:** إدارة المجموعات (مستشفيات، سلاسل صيدليات، ...).

**الأولوية:** 🟡 منخفضة — تأسيس فقط (لا تشغيل).

---

## 🎯 الجزء السابع: خطة التوسع المستقبلي

### 7.1 Rubika Currency Engine

**الحالة:** Schema موجود، Engine غير موجود.

**الخطوة التالية:** `IRubikaEngineService` في جلسة منفصلة.

### 7.2 الحملات التسويقية في خدمات أخرى

**PSP** = الحملة = `PSPProgram`. ✅
**B2B** = الحملة = `PurchaseOrder` / `B2BOrder` (مستقبلاً).
**CRM** = الحملة = `MRCampaign` (مستقبلاً).

**المبدأ:** كل خدمة تُصمم كيانها — لا جدول `Campaigns` عام.

### 7.3 KPI System for MRs

**الفكرة:** استخدام نظام النقاط لحساب مؤشرات أداء مندوبي الدعاية.

**التنفيذ:** نفس البنية (`Scheme + Rules + Balance`).

---

## 🎯 الجزء الثامن: ملخص الحالة

### 8.1 مكتمل (✅)

| # | المكوّن |
|:-:|--------|
| 1 | 8 جداول PSP Program Points |
| 2 | 5 جداول جديدة (Beneficiary + Groups) |
| 3 | 19 Handler |
| 4 | 5 Controllers |
| 5 | `PSPEnrollmentService` |
| 6 | 3 جداول Rubika |
| 7 | `PointBeneficiaryTypeIds` Constants |
| 8 | Migration Stage 1-3 |
| 9 | **Migration `TargetBeneficiaryTypeToScheme`** ⭐ |
| 10 | **`IPointEngineService` + `PointEngineService`** ⭐ |
| 11 | **`IEarnPointsOrchestrator` + `EarnPointsOrchestrator`** ⭐ |
| 12 | **`PointsContext`** ⭐ |

### 8.2 قيد الانتظار (⏳)

| # | المهمة | الأولوية |
|:-:|--------|:--------:|
| 1 | Frontend UI — اختيار نوع المستفيد | 🔴 |
| 2 | إدارة الجداول المرجعية (واجهات) | 🟠 |
| 3 | OrganizationGroups UI | 🟡 |
| 4 | Rubika Currency Engine | 🟠 |
| 5 | ربط `PROGRAM_COMPLETED` | 🟡 |
| 6 | ربط `LAB_TEST_UPLOADED` | 🟡 |
| 7 | اختبار شامل E2E | 🔴 |

---

## 🔗 الجزء التاسع: روابط ذات صلة

- [08 - Strategic Roadmap](./08-Strategic-Roadmap.md)
- [07 - نظام PSP](./07-psp-system.md)
- [15 - إدارة القوائم متعددة الطبقات](./15-multi-layer-menu-control.md)
- [00 - الهيكل المعماري](./00-architecture-overview.md)

---

## 📝 الجزء العاشر: سجل التغييرات

| الإصدار | التاريخ | التغييرات |
|---------|---------|-----------|
| 1.0 | 29 سبتمبر 2026 | الإصدار الأولي |
| 2.0 | 3 أكتوبر 2026 | التفاصيل الكاملة |
| 3.0 | 8 أكتوبر 2026 | Beneficiary Types + Organization Groups + Rubika |
| 4.0 | 9 أكتوبر 2026 | Stage 1-3 منفّذة |
| **5.0** | **10 أكتوبر 2026** | **نص المؤلف الحرفي + Backend الكامل (`IPointEngineService` + Orchestrator + PointsContext + Migration)** |

---

**© 2026 RubikCare — للاستخدام الداخلي**
```

---

## 📋 الخطوة التالية

**بعد تطبيق وثيقة 26، ننتقل إلى وثيقة 08.**

**هل:**
1. **تُطبّق وثيقة 26 الآن، ثم أواصل 08؟**
2. **أو تريد مراجعة 26 أولاً قبل 08؟**

**بانتظار قرارك.**
