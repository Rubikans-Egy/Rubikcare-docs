
# 26 — نظام النقاط وروبيكا
# (Rubika Points & Currency System)

**الإصدار:** 6.0 (Campaign Reference Architecture Edition)
**آخر تحديث:** 13 أكتوبر 2026
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

**الترجمة التقنية — مُحدَّثة (13 أكتوبر 2026):**
- **الرابط الفعلي** = `SourceEntityID` + `PointSystemTypeID`.
- **`SourceEntityID`** — معرّف الحملة المصدر (PSP → `ProgramID`، B2B → `PurchaseOrderID`).
- **`PointSystemTypeID`** — تصنيف (PSP=1, B2B_SUPPLY=2, B2C_PHARMACY=3, PLATFORM_MKT=4, RUBIKA=5).
- **`TargetBeneficiaryTypeID`** — اختياري، يحدد نوع المستفيد المستهدف في Scheme.

**⚠️ ملاحظة:** `ProgramID` — **محذوف نهائياً** (جلسة 13 أكتوبر 2026).

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

**الترجمة التقنية — مُحدَّثة:**
- نظام النقاط **بنية أفقية** — يخدم أي خدمة عبر `PointSystemTypeID`.
- كل خدمة تُنشئ **كياناتها الخاصة** (Programs, Campaigns, Orders, ...) — تُعطي `SourceEntityID`.
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
- **Rubika Currency** = نظام موازٍ (طبقة مستقلة) — `PointSystemTypeID = 5`.
- **المرض ليس دوراً** — بل حالة.
- **كل مستخدم** = مريض محتمل (`BeneficiaryTypeID = 4`).
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

**الترجمة التقنية — مُحدَّثة:**
- **حملة تسويقية** = كيان داخل كل خدمة — يُعطي `SourceEntityID`.
- **PSP**: الحملة = `PSPProgram` — `PointSystemTypeID = 1`.
- **B2B**: الحملة = `PurchaseOrder` / `B2BOrder` — `PointSystemTypeID = 2`.
- **CRM**: الحملة = `MRCampaign` — `PointSystemTypeID` (مستقبلاً).
- **الربط**: `Scheme.SourceEntityID` + `Scheme.PointSystemTypeID`.

**⚠️ قرار معتمد:** **لا جدول `Campaigns` عام.**

---

### 0.5 تعدد Schemes داخل نفس البرنامج

> **نص المؤلف الحرفي:**
>
> "مطلوب أن تتمكن المنظمة من وضع عدة أنظمة نقاط schema داخل نفس البرنامج.
>
> فمثلاً في برامج الدعم تتمكن من تصميم نظام للعيادات - الصيدليات - المرضى - العاملين في أي منظمة وفق نوعها.
>
> العاملين بها (المندوبين الداخليين المستخدمين للمنصة)."

**الترجمة التقنية — مُحدَّثة:**
- **`TargetBeneficiaryTypeID`** في `PSPProgramPointScheme` (nullable).
- **Filtered Unique Indexes الجديدة:**
  - `IX_PSPProgramPointSchemes_Source_System_General` — `(SourceEntityID, PointSystemTypeID)` UNIQUE WHERE `TargetBeneficiaryTypeID IS NULL`.
  - `IX_PSPProgramPointSchemes_Source_System_Target` — `(SourceEntityID, PointSystemTypeID, TargetBeneficiaryTypeID)` UNIQUE WHERE `TargetBeneficiaryTypeID IS NOT NULL`.
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

**الترجمة التقنية — مُحدَّثة:**
- **Vertical Slices**: كل خدمة = Module مستقل.
- **Reusable Services**: Notifications, Translation, Messaging, **Points**, Invoicing, Orders, Inventory, Checkout, AI.
- **API Client Layer**: `IXxxClient` Interfaces (In-Process + HTTP) — **مثال حي:** `IPointSchemeClientService`.

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
| `PointSystemTypeID` | 1, 2, 3, 4 | 5 |
| الطبقة | Services | Value |
| الجداول | `PSPProgramPoint*` | `RubikaWallet` + `RubikaTransactions` |

**المبدأ:** **نفس المحرك (`IPointEngineService`)، تنفيذان مختلفان.**

---

## 🎯 الجزء الثاني: Campaign Reference Architecture ⭐ جديد

> **⚠️ هذا القسم دُمج من وثيقة `28-campaign-reference-architecture.md`.**
> **يُوثّق القرار المعماري الأساسي لجلسة 13 أكتوبر 2026.**

### 2.1 مفهوم Campaign

> **"الحملة التسويقية = كيان داخل كل خدمة. لا يوجد جدول `Campaigns` عام."**

**⚠️ المشكلة القديمة:**
- `PSPProgramPointSchemes.ProgramID` — FK صلب إلى `PSPPrograms`.
- يمنع استخدام النظام في خدمات أخرى (B2B, Clinic, CRM).
- الاسم `ProgramID` — مضلل — يوحي بأنه PSP-only.

**⚠️ الحل الجديد:**
- `SourceEntityID` — معرّف الحملة المصدر (مفتوح).
- `PointSystemTypeID` — يميّز نوع الخدمة.
- **لا FK إلى `PSPPrograms`** — الضمان في Service Layer.

### 2.2 `SourceEntityID` + `PointSystemTypeID`

**⚠️ البنية الجديدة لـ `PSPProgramPointScheme`:**

| العمود | النوع | الوصف |
|:---|:---:|:---|
| `SchemeID` | `INT PK` | المعرف |
| **`PointSystemTypeID`** | `INT` | نوع نظام النقاط (PSP=1, B2B=2, ...) |
| **`SourceEntityID`** | `INT NOT NULL` | معرّف الحملة المصدر |
| `TargetBeneficiaryTypeID` | `INT?` | نوع المستفيد المستهدف (اختياري) |
| `ServiceProductID` | `INT?` | ربط بميزة مسعَّرة (اختياري) |
| `IsPlatformIssued` | `BIT` | هل الحملة من المنصة نفسها؟ |
| ... | | باقي الحقول |

**⚠️ العلاقة:**

```
SourceEntityID + PointSystemTypeID = الحملة الفعلية
```

**⚠️ جدول توضيحي:**

| `PointSystemTypeID` | `TypeCode` | `SourceEntityID` يُمثّل | جدول المصدر |
|:---:|:---|:---|:---|
| 1 | PSP | `PSPProgram.ProgramID` | `PSPPrograms` |
| 2 | B2B_SUPPLY | `PurchaseOrder.ID` | (مستقبلاً) |
| 3 | B2C_PHARMACY | `Campaign.ID` | (مستقبلاً) |
| 4 | PLATFORM_MKT | `Campaign.ID` | (مستقبلاً) |
| 5 | RUBIKA | `Wallet.ID` أو `null` | `RubikaWallets` |

### 2.3 دعم كل خدمة

#### 🔹 PSP — ✅ مُنفَّذ

```
PointSystemTypeID = 1
SourceEntityID = PSPProgram.ProgramID
TargetBeneficiaryTypeID = (1, 2, 5) — اختياري
```

#### 🔹 B2B_SUPPLY — 🟡 مُخطَّط

```
PointSystemTypeID = 2
SourceEntityID = PurchaseOrder.ID
TargetBeneficiaryTypeID = 1 (ORGANIZATION)
```

**⚠️ التطبيق:** كل عملية توريد ناجحة = نقاط للمخزن والصيدلية.

#### 🔹 B2C_PHARMACY — 🟡 مُخطَّط

```
PointSystemTypeID = 3
SourceEntityID = PharmacyCampaign.ID
TargetBeneficiaryTypeID = (4, 5) — REGULAR_USER + PATIENT
```

#### 🔹 CRM — 🟡 مُخطَّط

```
PointSystemTypeID = 4
SourceEntityID = MRCampaign.ID
TargetBeneficiaryTypeID = 2 (ORG_MEMBER_INTERNAL)
```

**⚠️ التطبيق:** KPI's for MRs.

#### 🔹 RUBIKA — 🟡 مُخطَّط

```
PointSystemTypeID = 5
SourceEntityID = null (نظام عام)
TargetBeneficiaryTypeID = (1, 4)
```

### 2.4 لا جدول Campaigns

> **قرار معتمد (نص المؤلف):**
>
> "قد لا نحتاج جدول Campaigns عام أبداً.
> فكل خدمة سيكون بداخلها كياناتها التي تصلح للربط، ونقرر وقتها هذا الأمر.
> سبب قولي اعتقادي أنه سيكون أقل تعقيداً، إلا لو كان ضرورة."

**⚠️ المبدأ:**

| ❌ لا نفعل | ✅ نفعل |
|:---|:---|
| جدول `Campaigns` عام | `SourceEntityID` + `PointSystemTypeID` |
| FK إلى جدول موحّد | FK غير مُحدَّد (مفتوح) |
| قيود مسبقة | الضمان في `Service Layer` |

### 2.5 `ServiceProductID`

**⚠️ الغرض:** ربط نظام النقاط بميزة مسعَّرة.

**⚠️ الاستخدام:**
- `NULL` — غير مرتبط بميزة (الوضع الحالي).
- `X` — مرتبط بميزة مثل `LOYALTY_POINTS_SYSTEM` في `ServiceProducts`.

**⚠️ الحالة:** ✅ موجود في Entity — 🟡 غير مُستخدم بعد.

**⚠️ الفائدة:**
- عند بناء نظام الاشتراكات — يمكن ربط `PointScheme` بميزة مسعَّرة.
- المنظمة تشترك في `SubscriptionPlan` يحتوي على `LOYALTY_POINTS_SYSTEM` → تُفعَّل الميزة تلقائياً.

---

## 🎯 الجزء الثالث: البنية المعمارية

### 3.1 المستويات الأربعة

```
Layer 1: Reference Layer
  ├── PointSystemTypes (5 قيم Seed)
  ├── PSPActivityTypes (7 قيم Seed)
  ├── PointBeneficiaryTypes (6 قيم Seed)
  └── PSPProgramPointTransactionTypes (4 قيم Seed)

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

### 3.2 الطبقات الأربع (مبسّطة)

| الطبقة | المسؤولية | الجداول الأساسية |
|--------|-----------|-----------------|
| **Reference** | التصنيفات | PointSystemTypes, PSPActivityTypes, PointBeneficiaryTypes |
| **Beneficiary** | من يستفيد | PointSchemeBeneficiaryTypes, PointSystemTypeBeneficiaryTypes |
| **Structure** | المؤسسات | Organizations, OrganizationGroups |
| **Value** | المحركات | PSPProgramPoint*, RubikaWallet, RubikaTransactions |

### 3.3 الجداول الفعلية لنظام النقاط

#### 🔹 المرجعية (Reference)

**`PointSystemTypes`** — 5 قيم:

| ID | TypeCode | NameAr | IssuerEntityType | SupportsCashRedeem |
|:--:|:---|:---|:---|:---:|
| 1 | PSP | نقاط برامج الدعم | PHARMA | ✅ |
| 2 | B2B_SUPPLY | نقاط التوريد B2B | PHARMA,WAREHOUSE | ❌ |
| 3 | B2C_PHARMACY | نقاط الصيدلية للجمهور | PHARMACY | ✅ |
| 4 | PLATFORM_MKT | نقاط المنصة التسويقية | PLATFORM | ✅ |
| 5 | RUBIKA | روبيكا | PLATFORM | ✅ |

**`PSPActivityTypes`** — 7 قيم:

| Code | الحدث |
|:---|:---|
| `PATIENT_INVITED` | دعوة مريض |
| `PATIENT_ENROLLED` | تسجيل مريض |
| `ERX_CREATED` | إنشاء وصفة |
| `DISPENSATION_COMPLETED` | صرف دواء |
| `REFILL_COMPLETED` | إعادة صرف |
| `PROGRAM_COMPLETED` | إكمال برنامج |
| `LAB_TEST_UPLOADED` | رفع تحليل |

**`PointBeneficiaryTypes`** — 6 قيم:

| ID | Code | NameAr | IsOrganization |
|:--:|:---|:---|:---:|
| 1 | ORGANIZATION | مؤسسة | ✅ |
| 2 | ORG_MEMBER_INTERNAL | عضو داخلي | ❌ |
| 3 | ORG_MEMBER_EXTERNAL | عضو خارجي | ❌ |
| 4 | REGULAR_USER | مستخدم عادي | ❌ |
| 5 | PATIENT | مريض | ❌ |
| 6 | GENERIC | عام | ❌ |

#### 🔹 الطبقة التشغيلية (Value Engine)

| الجدول | الدور |
|:---|:---|
| `PSPProgramPointSchemes` | نظام نقاط (Scheme) — واحد أو أكثر لكل حملة |
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
- `PremiumServices` — الخدمات المدفوعة.
- `RewardActivities` — أنشطة المكافآت.

### 3.4 مخطط ERD — مبسّط

```
Organizations
  ├── PSPPrograms (حملة PSP)
  │   └── PSPProgramPointSchemes
  │       ├── PSPProgramPointRules
  │       │   ├── PSPActivityTypes
  │       │   └── PointBeneficiaryTypes
  │       ├── PSPProgramPointBalances
  │       │   ├── PSPProgramPointTransactions
  │       │   └── PSPProgramPointRedeemRequests
  │       └── PointSchemeBeneficiaryTypes
  │
  └── (B2B / Clinic / CRM — مستقبلاً)

PSPProgramPointSchemes
  ├── PointSystemTypeID → PointSystemTypes
  ├── SourceEntityID → (مفتوح — يُفسَّر حسب PointSystemTypeID)
  ├── TargetBeneficiaryTypeID → PointBeneficiaryTypes (nullable)
  └── ServiceProductID → ServiceProducts (nullable)
```

---
[#104] 📄 وثيقة 26 — الإصدار 6.0 (الجزء الثاني)

---

```markdown
## 🎯 الجزء الرابع: Backend المُنفَّذ

### 4.1 المكوّنات الجديدة

| # | المكوّن | الموقع | الدور |
|:-:|--------|--------|-------|
| 1 | `EarnPointsCommand` | `Application/UseCases/PSP/Points/Earning/` | الأمر (BeneficiaryTypeID + BeneficiaryID + CampaignID + PointSystemTypeID) |
| 2 | `EarnPointsHandler` | `Infrastructure/UseCases/Points/Earning/` | التنفيذ الديناميكي |
| 3 | `PointsContext` | `Application/UseCases/PSP/Points/Earning/` | سياق الحدث |
| 4 | `IEarnPointsOrchestrator` + `EarnPointsOrchestrator` | Application + Infrastructure | الموزّع |
| 5 | `IPointEngineService` + `PointEngineService` | Application + Infrastructure | **Facade الموحّد** |
| 6 | **`IPSPPointSchemeService`** + `PSPPointSchemeService` | Application | **خدمة قراءة in-process** |
| 7 | **`IPointSchemeClientService`** + `PointSchemeClientService` | Web | **خدمة عميل HTTP** |

### 4.2 نمط العمل — مُحدَّث

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

**⚠️ التغيير الحاسم:** `AwardByProgramAsync` → `AwardByCampaignAsync` + `pointSystemTypeID`.

**⚠️ الفائدة:**
- **ديناميكي:** كل Rule يُطبَّق على المرشح المناسب.
- **مرن:** الشركة تُحدد من يستفيد (Rules).
- **قابل للتوسع:** إضافة BeneficiaryTypeID جديد = إضافة Rule.
- **دعم خدمات متعددة:** B2B + Clinic + CRM — عبر `PointSystemTypeID`.

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

**⚠️ الحالة:**
- **`EarnAsync`** — يستخدم `EarnPointsHandler` — مع `CampaignID` + `PointSystemTypeID`.
- **`EarnMultiAsync`** — يستخدم `AwardByCampaignAsync`.
- **`GetBalanceAsync`** — لم يتغير.
- **`GetTransactionsAsync`** — يستخدم `CampaignID`.

**⚠️ الاستخدام:**
- **العملاء الخارجيون** (`PSPEnrollmentService`, `DispenseController`) يستخدمون **`IEarnPointsOrchestrator` مباشرة** (للأداء).
- **العملاء الجدد** يستخدمون **`IPointEngineService`** (للبساطة).

### 4.4 `IPSPPointSchemeService` — خدمة القراءة In-Process

**⚠️ الموقع:** `Application/Interfaces/PSP/IPSPPointSchemeService.cs`.

**⚠️ الغرض:** قراءة `Schemes` + `Rules` من DB — **بدون HTTP** — لصفحات Blazor Server.

```csharp
public interface IPSPPointSchemeService
{
    // جلب أول Scheme (عام → نشط)
    Task<PointSchemeWithRules?> GetSchemeWithRulesAsync(
        int campaignId, 
        int pointSystemTypeID = 1);

    // جلب كل Schemes
    Task<List<PointSchemeWithRules>> GetSchemesWithRulesByCampaignAsync(
        int campaignId, 
        int pointSystemTypeID = 1);

    // إحصائيات
    Task<PointSchemeStats> GetSchemeStatsAsync(int schemeId);
}
```

**⚠️ المستخدِم:** `PSPProgramsDetails.razor` (Blazor Page) — يستدعي `GetSchemeWithRulesAsync(ProgramID, pointSystemTypeID: 1)`.

### 4.5 `IPointSchemeClientService` — خدمة عميل HTTP

**⚠️ الموقع:** `Web/Data/Services/IPointSchemeClientService.cs`.

**⚠️ الغرض:** استدعاء API `PointSchemesController` عبر HTTP.

**⚠️ الوظائف:**
- `GetSchemeByProgramAsync(int programId)` — يجلب Scheme واحد.
- `GetSchemesByProgramAsync(int programId, ...)` — يجلب قائمة Schemes.
- `GetSchemesByOrganizationAsync(int organizationId, ...)` — حسب المنظمة.
- `CreateSchemeAsync`, `UpdateSchemeAsync`, `DeactivateSchemeAsync`.
- `GetProgramOverviewAsync`, `GetSchemeClinicsAsync`.
- `GetRedeemRequestsAsync`, `AcceptRedeemRequestAsync`, `RejectRedeemRequestAsync`, `ResolveRedeemDisputeAsync`, `GetRedeemRequestDetailsAsync`.

**⚠️ المستخدِم:** `PSPStep2b_PointScheme.razor`, `PSPProgramManage.razor`, `LoyaltyHub.razor`, `LoyaltySchemesList.razor`.

### 4.6 ⚠️ ازدواجية موثَّقة

**⚠️ مشكلة معمارية:**

- **`IPSPPointSchemeService`** (Application) — قراءة in-process.
- **`IPointSchemeClientService`** (Web) — قراءة عبر HTTP.
- **`GetPointSchemeByProgramHandler`** (Infrastructure) — قراءة عبر API.

**⚠️ الثلاثة يفعلون نفس الشيء تقريباً.**

**⚠️ القرار:** **موثَّق كدين تقني** — سيُنظَّف في جلسة منفصلة (راجع `27-technical-debt.md`).

**⚠️ التوصية المستقبلية:**
- **توحيد الوصول:** `IPSPPointSchemeService` يصبح هو المصدر الوحيد للقراءة in-process.
- **`IPointSchemeClientService`** يبقى للـ UI الذي يستدعي API.
- **`GetPointSchemeByProgramHandler`** — استخدامه للـ API فقط.

### 4.7 Controllers — مُحدَّثة

**⚠️ `PointSchemesController`** — 5 Endpoints:

| Method | Endpoint | الغرض |
|:---|:---|:---|
| `POST` | `/api/psp/points/schemes` | إنشاء Scheme |
| `PUT` | `/api/psp/points/schemes/{schemeId}` | تحديث Scheme |
| `GET` | `/api/psp/points/schemes/by-program/{programId}` | جلب Scheme واحد |
| `POST` | `/api/psp/points/schemes/{schemeId}/deactivate` | تعطيل Scheme |
| `GET` | `/api/psp/points/schemes/list-by-program/{programId}` | جلب كل Schemes |
| `GET` | `/api/psp/points/schemes/programs/{programId}/overview` | نظرة عامة |
| `GET` | `/api/psp/points/schemes/programs/{programId}/clinics` | عيادات البرنامج |
| `GET` | `/api/psp/points/schemes/by-organization/{organizationId}` | حسب المنظمة |

**⚠️ `PointBalancesController`** — 6 Endpoints:

| Method | Endpoint |
|:---|:---|
| `GET` | `/api/psp/points/balances/by-organization` |
| `GET` | `/api/psp/points/balances/by-scheme/{schemeId}` |
| `GET` | `/api/psp/points/balances/organization-summary/{organizationId}` |
| `PUT` | `/api/psp/points/balances/visibility` |
| `GET` | `/api/psp/points/balances/visibility/check` |
| `PUT` | `/api/psp/points/balances/visibility/all/{schemeId}` |

**⚠️ `PointRulesController`** — 5 Endpoints.

**⚠️ `PointEarningController`** — 1 Endpoint.

**⚠️ `PointRedeemController`** — 5 Endpoints.

**⚠️ `DispenseController`** — يستخدم `AwardByCampaignAsync`.

---

## 🎯 الجزء الخامس: Migrations المُنفَّذة

### 5.1 Migration 1: `AddTargetBeneficiaryTypeToScheme`

**⚠️ التاريخ:** 10 أكتوبر 2026.

**⚠️ ما تم:**

| # | التعديل |
|:-:|--------|
| 1 | إضافة `TargetBeneficiaryTypeID` (`int?`) إلى `PSPProgramPointSchemes` |
| 2 | إزالة UNIQUE Index القديم على `ProgramID` |
| 3 | إضافة UNIQUE Filtered Index `IX_PSPProgramPointSchemes_ProgramID_General` |
| 4 | إضافة UNIQUE Filtered Index `IX_PSPProgramPointSchemes_ProgramID_TargetBeneficiary` |
| 5 | إضافة Index عادي على `TargetBeneficiaryTypeID` |
| 6 | إضافة FK إلى `PointBeneficiaryTypes` |

### 5.2 Migration 2: `UseSourceEntityIdInPointSchemes` ⭐ جديد

**⚠️ التاريخ:** 13 أكتوبر 2026.

**⚠️ السياق:** جزء من **Campaign Reference Architecture**.

**⚠️ ما تم:**

| # | التعديل | السبب |
|:-:|--------|:---|
| 1 | `DropForeignKey` على `ProgramID` | تحرير `Scheme` من `PSPPrograms` |
| 2 | `DropIndex` على `IX_..._ProgramID_General` | حذف Index قديم |
| 3 | `DropIndex` على `IX_..._ProgramID_TargetBeneficiary` | حذف Index قديم |
| 4 | `DropColumn` `ProgramID` | **حذف نهائي** — الاعتماد على `SourceEntityID` |
| 5 | `AlterColumn` `SourceEntityID` → NOT NULL | فرض NOT NULL |
| 6 | `AddColumn` `ServiceProductID` (`int?`) | ربط بميزة مسعَّرة |
| 7 | `CreateIndex` على `ServiceProductID` | للأداء |
| 8 | `CreateIndex` على `(SourceEntityID, PointSystemTypeID)` — Filtered Unique — General | Index جديد |
| 9 | `CreateIndex` على `(SourceEntityID, PointSystemTypeID, TargetBeneficiaryTypeID)` — Filtered Unique — Target | Index جديد |
| 10 | `AddForeignKey` على `ServiceProductID` | FK جديد |

**⚠️ الحالة النهائية لـ `PSPProgramPointSchemes`:**

```
SchemeID (PK)
PointSystemTypeID (NOT NULL, default=1)
SourceEntityID (NOT NULL)
TargetBeneficiaryTypeID (nullable)
ServiceProductID (nullable)
IsPlatformIssued (bit)
PointsNameAr, PointsNameEn, PointsCode
MonetaryValuePerPoint (decimal)
Currency
Description, ValidFrom, ValidTo
IsActive, CreatedBy, CreatedDate, LastModifiedDate
MinimumRedeemPoints, MaximumRedeemPoints, AllowPartialRedeem

Indexes:
  IX_..._PointsCode (Unique)
  IX_..._PointSystemTypeID
  IX_..._CreatedBy
  IX_..._ServiceProductID
  IX_..._TargetBeneficiaryTypeID
  IX_..._Source_System_General (Unique — Filtered)
  IX_..._Source_System_Target (Unique — Filtered)

FKs:
  FK_..._PointSystemTypes_PointSystemTypeID
  FK_..._ServiceProducts_ServiceProductID
  FK_..._TargetBeneficiaryType
  FK_..._UserProfiles_CreatedBy
```

### 5.3 الحالة الحالية

**⚠️ Model = Snapshot = DB — متزامنان.** ✅

**⚠️ Migration History محدَّث.**

**⚠️ البيانات سليمة:** `SchemeID=1, SourceEntityID=1, PointSystemTypeID=1`.

---

## 🎯 الجزء السادس: خطة Frontend

### 6.1 واجهات إدارة Schemes

**⚠️ الملف المستهدف:** `PSPStep2b_PointScheme.razor`.

**⚠️ التحديثات المطلوبة:**
- **اختيار `TargetBeneficiaryTypeID`** (nullable) — من `PointBeneficiaryTypes`.
- **عرض Schemes المتعددة** لنفس الحملة.
- **Rules متعددة** لكل Scheme.
- **استخدام `CampaignID`** بدل `ProgramID`.
- **استخدام `PointSystemTypeID = 1`** تلقائياً لـ PSP.

### 6.2 LoyaltyHub.razor — ✅ يعمل

**⚠️ الملف:** `Web/Components/Pages/Loyalty/LoyaltyHub.razor`.

**⚠️ الوظائف:**
- Dashboard — يعرض 4 Stats (Schemes + Rules + Balance + Pending Redeems).
- Quick Actions — Schemes, Balances, Redeem, Settings.
- Recent Schemes.

**⚠️ الحالة:** ✅ يعمل — يُفتح من القائمة الجانبية (`/loyalty`).

**⚠️ التسمية:** "نظام الولاء" (`Loyalty System`).

### 6.3 الصفحات المتبقية — قيد التنفيذ

| # | الصفحة | المسار | الحالة |
|:-:|:---|:---|:---:|
| 1 | `LoyaltySchemesList.razor` | `/loyalty/schemes` | ⏳ |
| 2 | `LoyaltySchemeDetails.razor` | `/loyalty/schemes/{id}` | ⏳ |
| 3 | `LoyaltySchemeEdit.razor` | `/loyalty/schemes/{id}/edit` | ⏳ |
| 4 | `LoyaltySchemeNew.razor` | `/loyalty/schemes/new` | ⏳ |
| 5 | `LoyaltySchemeManage.razor` | `/loyalty/schemes/{id}/manage` | ⏳ |
| 6 | `LoyaltyBalances.razor` | `/loyalty/balances` | ⏳ |
| 7 | `LoyaltyRedeemRequests.razor` | `/loyalty/redeem-requests` | ⏳ |
| 8 | `LoyaltySettings.razor` | `/loyalty/settings` | ⏳ |

### 6.4 واجهات إدارة الجداول المرجعية — 🟡 مُخطَّطة

**⚠️ الجمهور:** Super Admin.

**⚠️ الواجهات المقترحة:**
- إدارة `PSPActivityTypes`.
- إدارة `PointSystemTypes`.
- إدارة `PointBeneficiaryTypes`.
- إدارة `PointSystemTypeBeneficiaryTypes`.

**⚠️ الأولوية:** 🟠 متوسطة — تُنفَّذ بعد Frontend UI الرئيسي.

### 6.5 واجهات OrganizationGroups — 🟡 مُخطَّطة

**⚠️ الجمهور:** Super Admin.

**⚠️ الفكرة:** إدارة المجموعات (مستشفيات، سلاسل صيدليات، ...).

**⚠️ الحالة:** **تأسيس فقط** — `OrganizationGroups` + `OrganizationGroupMembers` موجودان — لكن **غير مُستخدَمين**.

**⚠️ الأولوية:** 🟡 منخفضة.

---

## 🎯 الجزء السابع: خطة التوسع المستقبلي

### 7.1 Rubika Currency Engine

**⚠️ الحالة:** Schema موجود (`RubikaWallet` + `RubikaTransactions` + `RubikaValueSettings`) — Engine غير موجود.

**⚠️ الخطوة التالية:** `IRubikaEngineService` في جلسة منفصلة.

**⚠️ الفكرة:**
- **`RubikaWallet`** — محفظة لكل `UserProfile` أو `Organization`.
- **`RubikaTransactions`** — سجل المعاملات.
- **`RubikaValueSettings`** — إعدادات القيمة (صرف، تحويل).

### 7.2 الحملات التسويقية في خدمات أخرى

| الخدمة | الحملة = | `PointSystemTypeID` |
|:---|:---|:---:|
| **PSP** | `PSPProgram` | 1 — ✅ |
| **B2B_SUPPLY** | `PurchaseOrder` / `B2BOrder` | 2 — 🟡 |
| **B2C_PHARMACY** | `PharmacyCampaign` | 3 — 🟡 |
| **PLATFORM_MKT** | `MarketingCampaign` | 4 — 🟡 |
| **RUBIKA** | (نظام عام) | 5 — 🟡 |
| **CRM** | `MRCampaign` | 6+ — 🟡 |

**⚠️ المبدأ:** كل خدمة تُصمم كيانها — **لا جدول `Campaigns` عام.**

### 7.3 KPI System for MRs

**⚠️ الفكرة:** استخدام نظام النقاط لحساب مؤشرات أداء مندوبي الدعاية.

**⚠️ التنفيذ:** نفس البنية (`Scheme + Rules + Balance`).

**⚠️ `PointSystemTypeID` جديد:** `CRM = 6` — أو `PLATFORM_MKT = 4`.

**⚠️ `BeneficiaryTypeID`:** `ORG_MEMBER_INTERNAL = 2` (المندوب).

### 7.4 إضافة خدمات جديدة

**⚠️ الخطوات:**

```
1. إضافة PointSystemType جديد في PointSystemTypes
   ↓
2. ربط BeneficiaryTypes المسموحة في PointSystemTypeBeneficiaryTypes
   ↓
3. إضافة ActivityTypes خاصة (إن لزم)
   ↓
4. إنشاء Scheme جديد لهذا النوع — عبر SourceEntityID + PointSystemTypeID
   ↓
5. (اختياري) ربط Scheme بـ ServiceProductID — للاشتراكات
```

**⚠️ المبدأ:** **الأساس موجود — لا نبني من الصفر.**

### 7.5 OrganizationGroups — التوسع

**⚠️ الفكرة:** المستشفى كمجموعة من المنظمات (عيادات + صيدلية + معمل + ...).

**⚠️ الجداول:** `OrganizationGroups` + `OrganizationGroupMembers`.

**⚠️ الحالة:** **موجودان — لكن غير مُستخدَمين.**

**⚠️ الأولوية:** 🟡 منخفضة — Phase 5+.

### 7.6 MenuVisibilityRules + Subscriptions

**⚠️ الربط بنظام القوائم:**

- **`LOYALTY_MENU`** له `MenuType = 'FEATURE_BASED'`.
- **عند تفعيل الاشتراكات:**
  - `SubscriptionPlan` يحتوي على `LOYALTY_POINTS_SYSTEM`.
  - `OrganizationSubscription` يربط المنظمة بالخطة.
  - `MenuVisibilityRule` (RuleType=Subscription) يظهر القائمة.

**⚠️ الحالة:** 🟡 مُخطَّطة — تعتمد على `04-dynamic-menus.md` + نظام الاشتراكات.

---
[#105] 📄 وثيقة 26 — الإصدار 6.0 (الجزء الثالث والأخير)

---

```markdown
## 🎯 الجزء الثامن: ملخص الحالة

### 8.1 مكتمل ✅

#### 🔹 Domain Layer

| # | المكوّن | العدد |
|:-:|--------|:---:|
| 1 | كيانات `PSPProgramPoint*` | 8 |
| 2 | كيانات Beneficiary (`PointBeneficiaryType`, `PointSchemeBeneficiaryType`, `PointSystemTypeBeneficiaryType`) | 3 |
| 3 | كيانات Rubika (`RubikaWallet`, `RubikaTransaction`, `RubikaValueSetting`) | 3 |
| 4 | كيانات OrganizationGroups | 2 |
| 5 | `PointSystemType` (Lookup) | 1 |
| 6 | `ServiceProduct` (من نظام الخدمات) | 1 |
| 7 | Constants (`PointBeneficiaryTypeIds`, `PointActivityTypeIds`, `PointTransactionTypeIds`) | 3 |
| 8 | Enum (`RuleType`) | 1 |

#### 🔹 Application Layer

| # | المكوّن | العدد |
|:-:|--------|:---:|
| 1 | Commands / Queries / DTOs | 25+ |
| 2 | Interfaces (`IEarnPointsHandler`, `IEarnPointsOrchestrator`, `IPointEngineService`) | 3 |
| 3 | Services (`PSPEnrollmentService`, `PSPPointSchemeService`) | 2 |

#### 🔹 Infrastructure Layer

| # | المكوّن | العدد |
|:-:|--------|:---:|
| 1 | Handlers (Schemes, Rules, Balances, Earning, Redeem, Manage) | 20+ |
| 2 | Services (`EarnPointsOrchestrator`, `PointEngineService`) | 2 |

#### 🔹 Api.Web Layer

| # | المكوّن | العدد |
|:-:|--------|:---:|
| 1 | Controllers (`PointSchemes`, `PointBalances`, `PointRules`, `PointEarning`, `PointRedeem`) | 5 |
| 2 | Endpoints | ~25 |

#### 🔹 Web Layer

| # | المكوّن | العدد |
|:-:|--------|:---:|
| 1 | Client Services (`IPointSchemeClientService`) | 1 |
| 2 | Pages (`LoyaltyHub`, `PSPStep2b_PointScheme`, `PSPProgramManage`, `PSPProgramsDetails`) | 4 |

#### 🔹 Migrations

| # | Migration | التاريخ |
|:-:|--------|:---|
| 1 | `AddPSPProgramPointsTables` | 29 سبتمبر |
| 2 | `AddPointSystemTypesAndBeneficiary` | 7 أكتوبر |
| 3 | `AddPointBeneficiaryAndOrganizationGroups` | 9 أكتوبر |
| 4 | `RemoveLegacyBeneficiaryColumns` | 9 أكتوبر |
| 5 | `AddTargetBeneficiaryTypeToScheme` | 10 أكتوبر |
| 6 | **`UseSourceEntityIdInPointSchemes`** | **13 أكتوبر** ⭐ |

#### 🔹 الميزات المُنفَّذة

| # | الميزة | الحالة |
|:-:|--------|:---:|
| 1 | نظام نقاط ديناميكي (`IPointEngineService`) | ✅ |
| 2 | Orchestrator متعدد المستفيدين | ✅ |
| 3 | `SourceEntityID` + `PointSystemTypeID` | ✅ |
| 4 | `ServiceProductID` (للاشتراكات) | ✅ |
| 5 | إدارة Schemes + Rules | ✅ |
| 6 | إدارة Balances + Redeem Requests | ✅ |
| 7 | `LoyaltyHub` (Dashboard) | ✅ |
| 8 | `MenuAssignment` لـ `LOYALTY_MENU` | ✅ |

### 8.2 قيد الانتظار ⏳

#### 🔴 أولوية عالية

| # | المهمة | السبب |
|:-:|--------|:---|
| 1 | **صفحات Loyalty المتبقية** (SchemesList, Details, Edit, Balances, Redeem, Settings) | إكمال Frontend |
| 2 | **ربط `PSPStep2b_PointScheme` بـ `LoyaltySchemeEdit`** | إعادة استخدام |
| 3 | **اختبار E2E شامل** | ضمان الاستقرار |

#### 🟠 أولوية متوسطة

| # | المهمة | السبب |
|:-:|--------|:---|
| 1 | **واجهات إدارة الجداول المرجعية** (ActivityTypes, SystemTypes, BeneficiaryTypes) | Super Admin |
| 2 | **`MenuVisibilityRules` UI** (Super Admin) | إدارة الميزات |
| 3 | **تنظيف `PSPPointSchemeService`** (ازدواجية) | دين تقني |
| 4 | **`Rubika Currency Engine`** (`IRubikaEngineService`) | نظام موازٍ |
| 5 | **ربط `PROGRAM_COMPLETED`** | حدث نقاط |
| 6 | **ربط `LAB_TEST_UPLOADED`** | حدث نقاط |

#### 🟡 أولوية منخفضة

| # | المهمة | السبب |
|:-:|--------|:---|
| 1 | **`OrganizationGroups` UI** | تأسيس فقط |
| 2 | **توسعة MAUI/PWA** | قوائم ديناميكية |
| 3 | **`Features` + `Plans` + `Subscriptions`** | اشتراكات |

---

## 🎯 الجزء التاسع: EF Core Migration — ملاحظات حرجة

> **⚠️ المرجع الأساسي:** [04-dynamic-menus.md — الجزء الثامن](./04-dynamic-menus.md#-الجزء-الثامن-ef-core-migration--ملاحظات-حرجة).

**⚠️ هذا القسم يُوجز القواعد + يذكر أمثلة من نظام النقاط.**

### 9.1 المبدأ الأساسي

> **"Migration المُولَّد صحيح — فقط عندما يكون Snapshot متزامناً مع DB."**

**⚠️ EF Core يقارن: Snapshot + Model Current. لا يفحص DB.**

### 9.2 مثال حي — جلسة 13 أكتوبر

**⚠️ ما حدث:**

1. **تعديل `OnModelCreating`** — تحويل `ProgramID` → `SourceEntityID`.
2. **`Add-Migration UseSourceEntityIdInPointSchemes`** — Migration حاول `DropIndex("IX_..._ProgramID")`.
3. **`Update-Database`** — فشل — لأن DB يحتوي `IX_..._ProgramID_General` وليس `IX_..._ProgramID`.
4. **السبب:** Snapshot كان يعكس حالة **قبل** Migration سابق — لا يعكس DB.
5. **الحل المُطبَّق:**
   - **SQL مباشر:** `UPDATE PSPProgramPointSchemes SET SourceEntityID = ProgramID`.
   - **تعديل Snapshot يدوياً** — ليعكس DB (بأسماء Indexes الفعلية).
   - **`Add-Migration UseSourceEntityIdInPointSchemes`** — جديد — يعكس الفرق الحقيقي.
   - **`Update-Database`** — نجح.
6. **النتيجة:** Model = Snapshot = DB — متزامن.

### 9.3 القواعد المُعتمَدة

| # | القاعدة | السبب |
|:-:|:---|:---|
| 1 | **لا تعديل Migration files يدوياً** | تُلغى عند أول `Add-Migration` آخر |
| 2 | **لا تعديل `ModelSnapshot.cs` كروتين** | يُلغى عند أول `Add-Migration` |
| 3 | **SQL مباشر للبيانات (Data Migration)** — مسموح | مثل `UPDATE SourceEntityID = ProgramID` |
| 4 | **تعديل Snapshot يدوياً** — استثناء نادر | لمرة واحدة — لإصلاح تعارض تاريخي |
| 5 | **Backup DB** قبل أي Migration | حماية |
| 6 | **مراجعة Migration المُولَّد** قبل `Update-Database` | اكتشاف الأخطاء مبكراً |

### 9.4 Checklist قبل Migration على نظام النقاط

- [ ] Backup DB كامل.
- [ ] Snapshot = DB؟ (تحقق من `PSPProgramPointSchemes`).
- [ ] `OnModelCreating` يُطابق Entity؟
- [ ] `PointSystemTypeID` مُعرَّف في `OnModelCreating`؟
- [ ] `SourceEntityID` NOT NULL في Entity + DB؟
- [ ] `Add-Migration` — مراجعة Migration المُولَّد.
- [ ] `Update-Database` — مراقبة الأخطاء.

---

## 🔗 الجزء العاشر: روابط ذات صلة

### 📚 وثائق أساسية

- [00 - الهيكل المعماري](./00-architecture-overview.md)
- [01 - Program.cs والتسجيلات الأساسية](./01-program-cs-foundation.md)
- [02 - نظام الهوية والمصادقة](./02-identity-system.md)
- [04 - نظام القوائم والصلاحيات](./04-dynamic-menus.md) ⭐ محدَّث
- [07 - نظام PSP](./07-psp-system.md)
- [08 - Strategic Roadmap](./08-Strategic-Roadmap.md)
- [14 - نظام الكاش الموحد](./14-caching-system.md)
- [26 - نظام النقاط وروبيكا](./26-rubika-points-system.md) (هذه الوثيقة)
- [27 - الديون التقنية](./27-technical-debt.md)

### 📚 وثائق ذات صلة

- [05 - إنشاء الصفحات والمكونات](./05-page-creation-checklist.md)
- [11 - دليل BlazorWebView](./11-blazor-webview-guide.md)

### 📚 وثائق مُدمَجة

- ~~`28-campaign-reference-architecture.md`~~ — **دُمج في هذه الوثيقة (الجزء الثاني).**

---

## 📝 الجزء الحادي عشر: سجل التغييرات

| الإصدار | التاريخ | التغييرات |
|---------|---------|-----------|
| **1.0** | **29 سبتمبر 2026** | الإصدار الأولي |
| **2.0** | **3 أكتوبر 2026** | التفاصيل الكاملة |
| **3.0** | **8 أكتوبر 2026** | Beneficiary Types + Organization Groups + Rubika |
| **4.0** | **9 أكتوبر 2026** | Stage 1-3 منفّذة |
| **5.0** | **10 أكتوبر 2026** | نص المؤلف الحرفي + Backend الكامل (`IPointEngineService` + Orchestrator + PointsContext + Migration) |
| **6.0** | **13 أكتوبر 2026** | **Campaign Reference Architecture Edition** — دمج `28` + تحديث `ProgramID → SourceEntityID` + `PointSystemTypeID` + `ServiceProductID` + Migration `UseSourceEntityIdInPointSchemes` + تحديث `AwardByCampaignAsync` + `IPSPPointSchemeService` + قسم EF Core Migration |

---

## 🎯 خاتمة

### 📌 الرؤية النهائية

**نظام النقاط وروبيكا** ليس مجرد نظام مكافآت — بل **بنية أفقية (Reusable Service)** تخدم:
- **PSP** — نقاط برامج الدعم.
- **B2B Supply Chain** — نقاط التوريد.
- **B2C E-commerce** — نقاط التجزئة.
- **Clinic OS** — نقاط العيادات.
- **Medical CRM** — KPI's for MRs.
- **Rubika Currency** — عملة المنصة الداخلية.

### 📌 المبادئ الأساسية

1. **Campaign Reference Architecture** — لا جدول `Campaigns` عام.
2. **`SourceEntityID` + `PointSystemTypeID`** — التمييز بين الخدمات.
3. **`IPointEngineService`** — Facade موحّد.
4. **Orchestrator + Rules** — ديناميكي ومرن.
5. **Reusable Service** — يخدم أي خدمة حالية أو مستقبلية.

### 📌 القاعدة الذهبية

> **"كل ميزة = خدمة مستقلة قابلة لإعادة الاستخدام."**

---

**© 2026 RubikCare — للاستخدام الداخلي**

**نهاية الوثيقة ✅**
```

---
