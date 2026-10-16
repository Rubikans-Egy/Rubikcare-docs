# 03 — MENUS (نظام القوائم الديناميكية والصلاحيات)

**الإصدار:** 4.0 (Unified Edition)
**آخر تحديث:** 16 أكتوبر 2026
**الحالة:** ✅ محدّث بالتنفيذ الفعلي + رؤية التكامل
**الجمهور:** فريق التطوير، المعماريون، AI Assistants
**المرجع الإلزامي:** يُقرأ قبل أي تعديل على القوائم أو الخدمات

---

## 📌 الجزء صفر: الفلسفة والرؤية (نص المؤلف الحرفي)

> **⚠️ ملاحظة:**
> هذا القسم يحتوي على **نص الكاتب الحرفي** — دون إعادة صياغة.
> الهدف: حفظ الرؤية الكاملة كما صيغت.

---

### 0.1 الرؤية الأصلية للنظام

> **نص المؤلف الحرفي:**
>
> "راجع هذه الوثيقة جيداً والخاصة بتحويل جذري لإدارة القوائم في المنصة.
> وذلك بهدف إظهار أو إخفاء هذه الميزات بشكل ديناميكي من لوحة التحكم في المنصة وليس عبر الكود، وفقاً لنظام اشتراكات أو خطط تسويق مختلفة ومرنة.
>
> لاحظ أن النظام موجود في الويب ولكن مطلوب تعميمه ليكون في منصات العرض كلها مع تحكم تام مركزي، مرن أيضاً بنفس نمط تفكيري."

**الترجمة التقنية:**

- **مصدر الحقيقة** = قاعدة البيانات (لا كود).
- **التعميم** = Web + PWA + MAUI (نفس النظام).
- **المرونة** = Subscriptions + Plans + Dynamic Visibility.
- **التحكم المركزي** = لوحة تحكم Super Admin.

---

### 0.2 الرؤية المُوسَّعة (نظام الخدمات والميزات)

> **نص المؤلف الحرفي:**
>
> "كل ميزة أريد أن يتم الوصول إليها عبر عنصر يتم التحكم في ظهوره أو إخفاءه في القوائم الجانبية لكل نوع من أنواع المنظمات، ولكل منظمة عبر رقمها (مستويين للتنظيم).
>
> نوع المنظمة يتحكم فيما يعتبر من خصائص هذا النوع، مثلاً العيادات يكون ضمن قوائمها نظام الحجز والكشف أونلاين وإدارة العيادات والسجلات الطبية للمرضى.
>
> كل من هذه عبارة عن عنصر يتم تسجيله في القوائم المندرجة تحت نوع المنظمة هذه.
>
> وهناك ميزات عامة مثل نظام النقاط والإشعارات وغيرها يمكن إعادة استخدامها عبر كل خدمة. فمثلاً يمكن أن تُستخدم في PSP - B2B - B2C - Clinic OS وهكذا.
>
> كل هذه العناصر يتم تسجيلها في القوائم بالنظام الحالي، ويتم ربطها عن طريق تسجيلها كميزات وخدمات في جداولها بحيث يمكن ربطها بالتسعير والمنظمات المستفيدة وفقاً لنوعها واشتراكاتها.
>
> عندما يفتح مستخدم المنصة، تظهر له قوائم المنظمة التي له عضوية بها. وكذلك يتم التحكم في الصلاحيات عبر نفس القوائم.
>
> قد تكون صلاحيات عمومية تحجب ميزة أو خاصية كاملة، أو ربما نطوّر لاحقاً نظام صلاحيات داخلي لكل منظمة يتم إعداده عبر أدمن كل منظمة."

**الترجمة التقنية:**

| # | المفهوم | التنفيذ |
|:-:|:---|:---|
| 1 | كل ميزة = عنصر في القائمة | `MenuItem` |
| 2 | التحكم في الظهور | `MenuVisibilityRule` |
| 3 | مستوى نوع المنظمة | `OrganizationTypeID` |
| 4 | مستوى المنظمة | `OrganizationID` |
| 5 | الميزات العامة | `ServiceProduct` مع `ServiceNature = FEATURE_BASED` |
| 6 | الميزات الخاصة | `ServiceProduct` مع `ServiceNature = SUBSCRIPTION_BASED` |
| 7 | الربط بالتسعير | `ServicePricingOption` + `SubscriptionPlan` |
| 8 | الربط بالاشتراك | `OrganizationSubscription` |
| 9 | الصلاحيات | عبر نفس `MenuVisibilityRule` (لاحقاً: `CustomJobTitleId`) |

---

### 0.3 الربط بنظام النقاط

> **نص المؤلف الحرفي:**
>
> "لماذا أخبرك بهذه الجزئية الآن؟ أخبرك بها لأننا سنحتاج بعد الانتهاء من backend ما ننجزه الآن أن نفكر في مراعاة هذه الجزئية المذكورة في تصميم الواجهات، وتعديل الوضع الحالي ليسمح بمرونة أكثر، بدلاً من كونها ميزة مدمجة في داخل برامج الدعم.
>
> فلا نحتاج مستقبلاً لتطوير واجهات مخصصة لكل خدمة نريد إضافة نظام نقاط لها."

**الترجمة التقنية:**

**المشكلة الحالية:**
- نظام النقاط مدمج داخل PSP.
- لو أردنا نظام نقاط في B2B → نسخ كود + تعديلات.

**بعد التنفيذ:**
- نظام النقاط يصبح **ميزة (Feature)**.
- المنظمة تشترك في **Plan** يحتوي على `POINTS_SYSTEM`.
- **لا كود مخصص لكل خدمة.**
- **نفس الميزة، نفس الواجهة، نفس المحرك.**

---

### 0.4 تعميم النظام على كل المنصات

> **نص المؤلف الحرفي:**
>
> "لاحظ أن النظام موجود في الويب ولكن مطلوب تعميمه ليكون في منصات العرض كلها مع تحكم تام مركزي."

**الترجمة التقنية:**

| المنصة | الحالة الحالية | الهدف |
|:------:|:--------------:|:-----:|
| **Web** | ✅ النظام مُطبَّق | يبقى |
| **PWA** | 🟡 قائمة ثابتة | تُطبَّق الطبقات |
| **MAUI** | 🟡 AppShell ثابت | يُستبدل بـ `DynamicMenuComponent` |

**الحل:** `DynamicMenuComponent` في `Shared.UI` — يُستخدم في **الثلاث منصات**.

---

### 0.5 فلسفة "التطوير التراكمي الآمن"

> **نص المؤلف الحرفي:**
>
> "في RubikCare، لا نؤمن بـ التغيير الجذري المفاجئ. بدلاً من ذلك، نتبع منهجية التطوير التراكمي الآمن."

```
1. فهم النظام الحالي
   ↓
2. إضافة الجديد بجانب القديم
   ↓
3. تحويل المستخدمين تدريجياً
   ↓
4. جمع الملاحظات والتحسين
   ↓
5. إزالة القديم بعد الاستقرار
```

**النتيجة:**
- **لا توقف في الخدمة** أبداً.
- **اكتشاف المشاكل مبكراً** على نطاق محدود.
- **قابلية التراجع** الفوري إذا ظهرت مشكلة.

---

## 📌 الملخص التنفيذي

هذه الوثيقة تُعرّف نظامًا معماريًا موحّدًا للتحكم في **القوائم** و**الصلاحيات** و**الميزات** و**الاشتراكات** عبر المنصات الثلاث (Blazor Web + PWA + MAUI)، من خلال:

- **4 طبقات تحكم** مستقلة ومتكاملة.
- **آلية Rules** ديناميكية.
- **نظام خدمات وميزات** قابل للربط.
- **نظام اشتراكات** وفق خطط تسويقية.
- **نظام جروبات منظمات** للمستشفيات وسلاسل الصيدليات.

**الهدف:** توحيد مصدر الحقيقة، إزالة الفوضى، تمكين المنظمات من التحكم الذاتي، وتفعيل نظام الاشتراكات والصلاحيات في حل واحد.

---

## 🏛️ الجزء الأول: البنية المعمارية

### 1.1 المبدأ الأساسي

> **"لا نبني نظامًا جديدًا، بل نُوسّع ونُوحّد الموجود."**

### 1.2 الطبقات الأربع للتحكم

```
┌─────────────────────────────────────────────────────────┐
│  الطبقة 1: المنصة (Platform)                             │
│  من يتحكم؟ Super Admin                                   │
│  ماذا؟ القوائم الرئيسية لكل نوع منظمة                    │
│  المعيار: OrganizationTypeID + MenuAssignments           │
└─────────────────────────────────────────────────────────┘
                          ↓
┌─────────────────────────────────────────────────────────┐
│  الطبقة 2: الاشتراكات (Subscriptions)                    │
│  من يتحكم؟ Super Admin                                   │
│  ماذا؟ الميزات المفعّلة في اشتراك المنظمة                 │
│  المعيار: MenuVisibilityRules (SubscriptionPlanID)       │
└─────────────────────────────────────────────────────────┘
                          ↓
┌─────────────────────────────────────────────────────────┐
│  الطبقة 3: المنظمة (Organization)                        │
│  من يتحكم؟ Admin المنظمة                                 │
│  ماذا؟ ما سمحت به المنظمة لعناصر قوائمها                 │
│  المعيار: MenuVisibilityRules (OrganizationID)           │
└─────────────────────────────────────────────────────────┘
                          ↓
┌─────────────────────────────────────────────────────────┐
│  الطبقة 4: الوظيفة (JobTitle) — 🟡 مُخطَّطة              │
│  من يتحكم؟ Admin المنظمة                                 │
│  ماذا؟ ما تراه كل وظيفة داخل المنظمة                     │
│  المعيار: MenuVisibilityRules (CustomJobTitleId)         │
└─────────────────────────────────────────────────────────┘
```

**⚠️ الحالة الفعلية:**

| الطبقة | الحالة | الملاحظة |
|:---:|:---:|:---|
| **الطبقة 1** | ✅ **مُنفَّذة** | `MenuAssignments` + `SystemMenus` + `MenuItems` |
| **الطبقة 2** | 🟡 **مُنفَّذة جزئياً** | `SubscriptionPlanID` موجود — Rules فارغة |
| **الطبقة 3** | 🟡 **مُنفَّذة جزئياً** | `OrganizationID` موجود — Rules فارغة |
| **الطبقة 4** | 🟡 **مُخطَّطة** | `CustomJobTitleId` يحتاج Migration |

### 1.3 ترتيب الأولويات بين الطبقات

```
1. MenuAssignments → تحديد القوائم المتاحة (OrganizationTypeID + OrganizationID)
   ↓
2. MenuVisibilityRules → تطبيق القواعد (RuleType + Priority)
   ↓
3. القائمة النهائية للمستخدم
```

**قاعدة الأولوية:**

> **"Priority الأصغر = الأولوية الأعلى."**

**مثال:**

- Rule 1: `RuleType=All`, `Priority=100`, `IsVisible=true` → يظهر للجميع.
- Rule 2: `RuleType=Organization`, `OrganizationID=5`, `Priority=50`, `IsVisible=false` → يخفي عن Org 5.
- **النتيجة لـ Org 5:** **الاثنان مطابقان** → **الأعلى أولوية (Rule 2, Priority=50) يفوز** → **مخفي**.

### 1.4 الطبقة 1: المنصة (Platform Control) — ✅ مُنفَّذة

#### الوصف
- **من يتحكم؟** Super Admin (RubikCare).
- **ماذا يتحكم؟** القوائم الرئيسية + عناصرها لكل `OrganizationType` + `Organization`.
- **المعيار:** `MenuAssignments` — بمعيار `OrganizationTypeID` أولاً، ثم `OrganizationID`.

#### الوظائف
- إضافة قائمة رئيسية.
- إضافة عناصر للقائمة.
- إخفاء/إظهار عنصر عبر `MenuItems.IsVisible`.
- نقل عنصر بين قوائم.
- إظهار عنصر في عدة قوائم.

#### الجداول المستخدمة
- `SystemMenus`
- `MenuItems`
- `MenuAssignments`

#### آلية العرض — مثال حي

```
OrganizationTypeID = 2 → PHARMA_PSP (ORG_TYPE_BASED)
OrganizationTypeID = 2 → LOYALTY (FEATURE_BASED)

OrganizationTypeID = 3 → PHARMACY_MENU (ORG_TYPE_BASED)
OrganizationTypeID = 4 → CLINIC_MENU (ORG_TYPE_BASED)
OrganizationTypeID = 5 → DRUG_WAREHOUSE_MENU (ORG_TYPE_BASED) ⭐
```

- **المرحلة الأولى:** جلب `MenuAssignments` بـ `OrganizationTypeID = 2`.
- **النتيجة:** `PHARMA_PSP` + `LOYALTY`.
- **المرحلة الثانية:** فلترة `MenuItems` بـ `IsVisible = 1`.

### 1.5 الطبقة 2: الاشتراكات (Subscriptions) — 🟡 جزئياً مُنفَّذة

#### الوصف
- **من يتحكم؟** Super Admin.
- **ماذا يتحكم؟** الميزات المفعّلة في اشتراك كل منظمة.
- **المعيار:** `MenuVisibilityRules` — مع `SubscriptionPlanID`.

#### ما هو موجود فعلاً

| العنصر | الحالة |
|:---|:---:|
| `MenuVisibilityRules.SubscriptionPlanID` | ✅ موجود |
| `SubscriptionPlan` (Table) | ✅ موجود |
| `PlanService` (M:N) | ✅ موجود |
| `ServiceProduct` (Table) | ✅ موجود |
| `ServicePricingOption` | ✅ موجود |
| **UI لإدارة الاشتراكات** | ✅ موجود (`SubscriptionPlans.razor`) |
| **OrganizationSubscription** | ❌ **مفقود** — يحتاج إنشاء |
| **Rules فعلية في DB** | ❌ **فارغة (0)** |
| **UI لربط الخطة بالقوائم** | ❌ **مفقود** |

#### آلية العمل — التصميم

عند بناء القائمة لمستخدم:
1. يُحدد اشتراك المنظمة النشطة (`OrganizationSubscription`).
2. تُحدد الميزات المفعّلة (`PlanService` → `ServiceProduct`).
3. تُفلتر `MenuItems` بحيث تظهر فقط العناصر المرتبطة بميزات مفعّلة — عبر `MenuVisibilityRules (SubscriptionPlanID)`.

### 1.6 الطبقة 3: المنظمة (Organization Control) — 🟡 جزئياً مُنفَّذة

#### الوصف
- **من يتحكم؟** Admin المنظمة.
- **ماذا يتحكم؟** ما سمحت به المنظمة لعناصر قوائمها (إظهار/إخفاء).
- **المعيار:** `MenuVisibilityRules` — مع `OrganizationID`.

#### ما هو موجود فعلاً

| العنصر | الحالة |
|:---|:---:|
| `MenuVisibilityRules.OrganizationID` | ✅ موجود |
| **UI لإدارة Rules المنظمة** | ❌ **مفقود** |
| **Rules فعلية** | ❌ **فارغة** |

#### الفرق بين الطبقة 1 والطبقة 3

| البند | الطبقة 1 (المنصة) | الطبقة 3 (المنظمة) |
|:---|:---|:---|
| **من يتحكم؟** | Super Admin | Admin المنظمة |
| **النطاق** | كل المنظمات | منظمة واحدة |
| **الوظائف** | إضافة/نقل/إخفاء | إظهار/إخفاء فقط |
| **المستوى** | `MenuAssignment` + `MenuItems.IsVisible` | `MenuVisibilityRules (OrganizationID)` |

### 1.7 الطبقة 4: الوظيفة (JobTitle) — 🟡 مُخطَّطة

#### الوصف
- **من يتحكم؟** Admin المنظمة.
- **ماذا يتحكم؟** ما تراه كل وظيفة داخل المنظمة.
- **المعيار:** `MenuVisibilityRules` — مع `CustomJobTitleId`.

#### ما يحتاج (Migration مطلوبة)

- إضافة `CustomJobTitleId INT NULL` إلى `MenuVisibilityRules`.
- ربط `CustomJobTitle` بـ `OrgMemberships` (موجود).
- UI لإدارة صلاحيات الوظائف.

#### ملاحظة مهمة

- **`RoleId`** (في `AspNetRoles`) — **لا يُستخدم لإدارة القوائم** — لأنه للأدوار المهنية (`Doctor`, `Pharmacist`).
- **`CustomJobTitleId`** — هو المعيار الصحيح — لأنه يحدد **الوظيفة داخل المنظمة**.

### 1.8 ترتيب الأولويات بين الطبقات

عند بناء القائمة لمستخدم، تُطبّق الطبقات بالترتيب التالي:

```
1. الطبقة 1: تحديد القوائم المتاحة (MenuAssignments)
   ↓
2. الطبقة 2: فلترة القوائم وفق الاشتراك (MenuVisibilityRules — SubscriptionPlanID)
   ↓
3. الطبقة 3: فلترة العناصر وفق إعدادات المنظمة (MenuVisibilityRules — OrganizationID)
   ↓
4. الطبقة 4: فلترة العناصر وفق وظيفة المستخدم (MenuVisibilityRules — CustomJobTitleId)
   ↓
5. القائمة النهائية
```

#### قاعدة الأولوية

> **"الطبقة الأدق تتجاوز الأعلى عمومية."**

- إذا أظهرت الطبقة 3 عنصرًا، لكن الطبقة 4 حجبته عن وظيفة المستخدم → **لا يظهر**.
- إذا حجبت الطبقة 3 عنصرًا، لكن الطبقة 4 سمحت به → **لا يظهر** (لأن الطبقة 3 أعلى).
- **الاستثناء:** `UserProfileID` في `MenuAssignment` (تخصيص مباشر) — **مستقبلاً**.

---

## 🗄️ الجزء الثاني: نموذج البيانات الكامل

### 2.1 الجداول الأساسية — ✅ موجودة

| # | الجدول | الدور |
|:-:|--------|-------|
| 1 | `SystemMenus` | القوائم الرئيسية |
| 2 | `MenuItems` | عناصر القوائم (Items) |
| 3 | `MenuAssignments` | ربط القوائم بالمنظمات وأنواعها |
| 4 | `MenuVisibilityRules` | قواعد الإظهار/الإخفاء الديناميكية |
| 5 | `CustomJobTitles` | وظائف الأعضاء |
| 6 | `OrgMemberships` | عضويات المستخدمين |
| 7 | **`ServiceProduct`** | ⭐ **الخدمات/الميزات** |
| 8 | **`ServiceCategory`** | فئات الخدمات |
| 9 | **`SubscriptionPlan`** | ⭐ **خطط الاشتراك** |
| 10 | **`PlanService`** | M:N بين Plan و ServiceProduct |
| 11 | **`ServicePricingOption`** | خيارات التسعير |
| 12 | **`ServiceTarget`** | الجمهور المستهدف |
| 13 | **`PremiumService`** | الخدمات المميزة |
| 14 | **`OrganizationGroup`** | مجموعات المنظمات |
| 15 | **`OrganizationGroupMember`** | أعضاء المجموعة |
| 16 | **`RubikaWallet`** | محفظة Rubika |
| 17 | **`RubikaTransaction`** | معاملات Rubika |
| 18 | **`RubikaValueSetting`** | إعدادات القيمة |

### 2.2 `SystemMenus` — البنية الفعلية

```sql
CREATE TABLE SystemMenus (
    MenuID INT PRIMARY KEY IDENTITY(1,1),
    MenuCode NVARCHAR(50) NOT NULL,
    MenuNameAr NVARCHAR(200) NOT NULL,
    MenuNameEn NVARCHAR(200) NULL,
    MenuType NVARCHAR(50) NOT NULL,   -- BASE | ROLE_BASED | ORG_TYPE_BASED | FEATURE_BASED
    IconClass NVARCHAR(100) NULL,
    ColorCode NVARCHAR(50) NULL,
    SortOrder INT NOT NULL DEFAULT 100,
    Description NVARCHAR(500) NULL,
    IsActive BIT NOT NULL DEFAULT 1,
    ShowInSidebar BIT NOT NULL DEFAULT 1,
    ShowInTopbar BIT NOT NULL DEFAULT 0,
    CreatedBy INT NULL,
    CreatedDate DATETIME2 NOT NULL DEFAULT GETDATE(),
    LastModifiedDate DATETIME2 NULL
);
```

**القيم الفعلية (16 أكتوبر 2026):**

| MenuID | MenuCode | MenuNameAr | MenuType | SortOrder | ملاحظة |
|:------:|----------|------------|----------|:---------:|:---|
| 2 | `USER_BASE` | القائمة الشخصية | `BASE` | 10 | عام |
| 13 | `LOYALTY` | نظام النقاط | `FEATURE_BASED` | 15 | ⭐ |
| 3 | `ADMIN_MENU` | لوحة تحكم الإدارة | `ROLE_BASED` | 20 | للمسؤولين |
| 4 | `PHARMA_PSP` | شركات الأدوية | `ORG_TYPE_BASED` | 30 | Pharma |
| 9 | `CLINIC_MENU` | العيادة | `ORG_TYPE_BASED` | 40 | Clinic |
| 10 | `PHARMACY_MENU` | الصيدلية | `ORG_TYPE_BASED` | 45 | Pharmacy |
| 11 | **`DRUG_WAREHOUSE_MENU`** | **مخزن الأدوية** | `ORG_TYPE_BASED` | 50 | ⭐ **مُصحَّح من `HOSPITAL_MENU`** |

### 2.3 `MenuType` — القيم المُعتمَدة

| القيمة | الغرض | مثال |
|:---:|:---|:---|
| **`BASE`** | قوائم عامة لكل المستخدمين | `USER_BASE` |
| **`ROLE_BASED`** | قوائم مرتبطة بالدور النظامي | `ADMIN_MENU` |
| **`ORG_TYPE_BASED`** | قوائم مرتبطة بنوع المنظمة | `PHARMA_PSP`, `CLINIC_MENU` |
| **`FEATURE_BASED`** | قوائم مرتبطة بميزة (اشتراك/خطة) | `LOYALTY` |

**⚠️ ملاحظة:**
- هذه القيم **4 نهائية**.
- **قيم مستقبلية محتملة:** `USER_SPECIFIC`, `HYBRID`.

### 2.4 `MenuItems` — البنية الفعلية

```sql
CREATE TABLE MenuItems (
    ItemID INT PRIMARY KEY IDENTITY(1,1),
    MenuID INT NOT NULL FOREIGN KEY REFERENCES SystemMenus(MenuID),
    ParentItemID INT NULL FOREIGN KEY REFERENCES MenuItems(ItemID),
    ItemCode NVARCHAR(100) NOT NULL UNIQUE,
    ItemNameAr NVARCHAR(200) NOT NULL,
    ItemNameEn NVARCHAR(200) NULL,
    Url NVARCHAR(500) NOT NULL,
    IconClass NVARCHAR(100) NULL,
    SortOrder INT NOT NULL DEFAULT 100,
    IsVisible BIT NOT NULL DEFAULT 1,
    RequiresPermission BIT NOT NULL DEFAULT 0,
    PermissionCode NVARCHAR(100) NULL,
    OpenInNewTab BIT NOT NULL DEFAULT 0,
    Tooltip NVARCHAR(300) NULL,
    CreatedDate DATETIME2 NOT NULL DEFAULT GETDATE(),
    LastModifiedDate DATETIME2 NULL
);
```

**⚠️ ملاحظات:**
- **`ItemCode`** — كود فريد لكل عنصر.
- **`IsVisible`** — بديل عن `IsActive`.
- **`RequiresPermission`** — bit — مع `PermissionCode`.
- **`ParentItemID`** — يدعم Hierarchy نظرياً — **غير مستخدم حالياً**.

### 2.5 `MenuAssignments` — البنية الفعلية

```sql
CREATE TABLE MenuAssignments (
    AssignmentID INT PRIMARY KEY IDENTITY(1,1),
    MenuID INT NOT NULL FOREIGN KEY REFERENCES SystemMenus(MenuID),
    OrganizationTypeID INT NULL FOREIGN KEY REFERENCES OrganizationTypes(OrganizationTypeID),
    OrganizationID INT NULL FOREIGN KEY REFERENCES Organizations(OrganizationID),
    IsActive BIT NOT NULL DEFAULT 1,
    AssignedDate DATETIME2 NOT NULL DEFAULT GETUTCDATE(),
    ValidTo DATETIME2 NULL,
    Notes NVARCHAR(500) NULL,
    CreatedBy INT NULL,
    LastModifiedDate DATETIME2 NULL
);
```

**⚠️ ملاحظات مهمة:**
- **لا يوجد `AssignmentType`.**
- **لا يوجد `UserProfileID` ولا `RoleId`.**
- **⚠️ دين تقني:** عمود `SystemMenuMenuID` (Shadow Property) — **يحتاج حذف**.

**القيم الفعلية (16 أكتوبر 2026):**

| AssignmentID | MenuID | OrganizationTypeID | OrganizationID | ملاحظة |
|:---:|:---:|:---:|:---:|:---|
| 2 | 3 | NULL | 1 | ADMIN_MENU — لمؤسسة 1 |
| 3 | 4 | NULL | 91 | PHARMA_PSP — لمؤسسة 91 |
| 4 | 4 | 2 | NULL | PHARMA_PSP — لكل شركات الأدوية |
| 5 | 2 | NULL | NULL | USER_BASE — عام |
| 10 | 6 | 3 | NULL | (MenuID=6 غير موجود في SystemMenus) |
| 11 | 10 | 3 | NULL | PHARMACY_MENU |
| 12 | 11 | 5 | NULL | DRUG_WAREHOUSE_MENU |
| 13 | 9 | 4 | NULL | CLINIC_MENU |
| 14 | 13 | 2 | NULL | LOYALTY — لكل شركات الأدوية |

### 2.6 `MenuVisibilityRules` — البنية الفعلية

```sql
CREATE TABLE MenuVisibilityRules (
    RuleID INT PRIMARY KEY IDENTITY(1,1),
    ItemID INT NOT NULL FOREIGN KEY REFERENCES MenuItems(ItemID),
    RuleType INT NOT NULL,                       -- Enum (int)
    OrganizationID INT NULL,
    OrganizationTypeID INT NULL,
    SubscriptionPlanID INT NULL,
    RoleId NVARCHAR(450) NULL,                   -- ⚠️ سيُستبدل بـ CustomJobTitleId
    IsVisible BIT NOT NULL DEFAULT 1,
    EffectiveFrom DATETIME2 NOT NULL DEFAULT GETUTCDATE(),
    EffectiveTo DATETIME2 NULL,
    Priority INT NOT NULL,                       -- الأصغر = الأعلى
    IsActive BIT NOT NULL DEFAULT 1,
    CreatedDate DATETIME2 NOT NULL DEFAULT GETUTCDATE(),
    CreatedBy NVARCHAR(450) NULL,
    LastModifiedDate DATETIME2 NULL,
    LastModifiedBy NVARCHAR(450) NULL
);
```

**⚠️ الحالة الحالية:** **0 سجل** — الجدول فارغ.

**⚠️ التغييرات المُخطَّطة:**
- **إضافة `CustomJobTitleId INT NULL`.**
- **إزالة `RoleId`** — بعد الانتقال الكامل إلى `CustomJobTitleId`.

### 2.7 `RuleType` — Enum

```csharp
public enum RuleType
{
    All = 0,                // للجميع
    Organization = 1,       // منظمة محددة
    OrganizationType = 2,   // نوع منظمة
    Subscription = 3,       // خطة اشتراك
    Role = 4,               // دور مستخدم (سيتحول إلى JobTitle = 4)
    User = 5                // مستخدم محدد (مستقبلاً)
}
```

**⚠️ ملاحظة:**
- `RuleType` = **Enum** — مُخزَّن كـ `int` — **لا جدول مرجعي**.
- `Role = 4` — سيُعاد تسميته إلى `JobTitle` — بعد Migration.

### 2.8 ⭐ نظام الخدمات والميزات (Services & Features)

**⚠️ هذا النظام مُصمَّم من بدايات المشروع — لكن لم يُستخدَم بعد.**

#### `ServiceProduct` — الخدمة/الميزة

```csharp
[Table("ServiceProducts")]
public class ServiceProduct
{
    public int ServiceProductID { get; set; }
    public string SKU_Code { get; set; }              // مثال: "PSP-BASIC"
    public string ServiceNameAr { get; set; }
    public string? ServiceNameEn { get; set; }
    public int? CategoryID { get; set; }
    public string ServiceNature { get; set; } = "FEATURE_BASED";  // ⭐
    public decimal? BasePrice_InRubika { get; set; }
    public decimal? FeatureUnlockPrice { get; set; }
    public decimal? CommissionRate { get; set; }
    public bool IsActive { get; set; }
    public bool RequiresApproval { get; set; }
    public bool IsRewardService { get; set; }
    public bool IsIncludedInFreePlan { get; set; }
    // ... + ServiceTargets, PricingOptions, PlanServices
}
```

**⚠️ القيم الحالية في DB (10 خدمات):**

| ID | SKU | الاسم | CategoryID | ServiceNature |
|:--:|:---|:---|:---:|:---|
| 5 | `PSP-BASIC` | برنامج دعم أساسي | 2 | `SUBSCRIPTION_BASED` |
| 6 | `PSP-PREMIUM` | برنامج دعم متقدم | 2 | `SUBSCRIPTION_BASED` |
| 7 | `ACCESS-SMARTSAVE` | خصم ذكي | 7 | `FEATURE_BASED` |
| 8 | `ACCESS-COMPASSIONATE` | علاج مجاني | 8 | `FEATURE_BASED` |
| 9 | `ACCESS-CONTINUITY` | مكافآت الاستمرارية | 9 | `FEATURE_BASED` |
| 10 | `ACCESS-FLEXIPAY` | خطط تقسيط | 10 | `FEATURE_BASED` |

#### `ServiceCategory` — فئات الخدمات

**⚠️ 6 فئات رئيسية + 20 فئة فرعية:**

| ID | الاسم | ParentID | الوصف |
|:--:|:---|:---:|:---|
| 2 | برامج الوصول المالي | NULL | ACCESS Programs |
| 3 | برامج الالتزام العلاجي | NULL | ADHERENCE Programs |
| 4 | برامج التيسير والراحة | NULL | CONVENIENCE Programs |
| 5 | برامج قياس النتائج | NULL | OUTCOMES Programs |
| 6 | برامج الدعم المجتمعي | NULL | COMMUNITY Programs |
| 7-10 | (فرعيات ACCESS) | 2 | SmartSave, Compassionate, ... |
| 11-14 | (فرعيات ADHERENCE) | 3 | MyTherapy Academy, CareTrack, ... |
| 15-18 | (فرعيات CONVENIENCE) | 4 | MediDirect, HomeCare, ... |
| 19-22 | (فرعيات OUTCOMES) | 5 | Dashboard, RWE, Loyalty, ... |
| 23-26 | (فرعيات COMMUNITY) | 6 | Peer Support, Mental Wellness, ... |

#### `SubscriptionPlan` — خطة الاشتراك

```csharp
[Table("SubscriptionPlans")]
public class SubscriptionPlan
{
    public int PlanID { get; set; }
    public string PlanNameAr { get; set; }
    public string? PlanNameEn { get; set; }
    public string PlanCode { get; set; }
    public string PlanType { get; set; } = "CLINIC";  // CLINIC, HOSPITAL, PHARMACY, DOCTOR
    public decimal PriceInRubika { get; set; }
    public int DurationDays { get; set; } = 30;
    public string BillingCycle { get; set; } = "MONTHLY";
    public int MaxServices { get; set; } = 10;
    public int MaxStaff { get; set; } = 5;
    public bool HasPremiumSupport { get; set; }
    public bool HasAnalytics { get; set; }
    public bool HasCustomBranding { get; set; }
    // ⭐ M:N
    public virtual ICollection<PlanService> IncludedServices { get; set; }
}
```

**⚠️ الحالة الحالية:** **0 سجل** — الجدول فارغ.

#### `PlanService` — M:N

```csharp
[Table("PlanServices")]
public class PlanService
{
    [Key, Column(Order = 1)]
    public int PlanID { get; set; }

    [Key, Column(Order = 2)]
    public int ServiceProductID { get; set; }

    public int? MaxUnits { get; set; }
    public bool IsUnlimited { get; set; } = false;
    public decimal? ExtraUnitPrice_RUB { get; set; }
    public bool IsIncluded { get; set; } = true;
    public int DisplayOrder { get; set; }
}
```

**⚠️ الحالة الحالية:** **0 سجل**.

#### `ServicePricingOption` — خيارات التسعير

```csharp
[Table("ServicePricingOptions")]
public class ServicePricingOption
{
    public int PricingOptionID { get; set; }
    public int ServiceProductID { get; set; }
    public string PlanNameAr { get; set; }
    public string? PlanCode { get; set; }
    public string PricingType { get; set; }
    public string TargetUserType { get; set; } = "ALL";
    public decimal? PriceInRubika { get; set; }
    public decimal? PriceInEGP { get; set; }
    public decimal? CommissionRate { get; set; }
    public string PricingModel { get; set; } = "PER_UNIT";
    public string Currency { get; set; } = "RUBIKA";
    public int? MaxUsers { get; set; }
    public int? MaxClinics { get; set; }
    public int? MaxStorageGB { get; set; }
    public decimal? RewardAmount { get; set; }
    public string? TriggerCondition { get; set; }
    public DateTime ValidFrom { get; set; }
    public DateTime? ValidTo { get; set; }
}
```

**⚠️ الحالة الحالية:** **0 سجل**.

#### `ServiceTarget` — الجمهور المستهدف

```csharp
[Table("ServiceTargets")]
public class ServiceTarget
{
    public int ServiceTargetID { get; set; }
    public int ServiceProductID { get; set; }
    public string TargetType { get; set; } = "ORGANIZATION";  // ORGANIZATION | USER_ROLE
    public int TargetID { get; set; }
    public bool IsActive { get; set; } = true;
    // ⭐ العلاقات
    public virtual ServiceProduct ServiceProduct { get; set; }
}
```

**⚠️ الاستخدام:** يربط خدمة بـ:
- `TargetType = "ORGANIZATION"` → `TargetID = OrganizationTypeID`
- `TargetType = "USER_ROLE"` → `TargetID = RoleID`

#### `PremiumService` — الخدمات المميزة

```csharp
[Table("PremiumServices")]
public class PremiumService
{
    public int PremiumServiceID { get; set; }
    public string ServiceCode { get; set; }
    public string ServiceNameAr { get; set; }
    public decimal MonthlyPrice_InRubika { get; set; }
    public int? DurationMonths { get; set; }
    public string? FeaturesAr { get; set; }
}
```

### 2.9 ⭐ نظام جروبات المنظمات (OrganizationGroup)

**⚠️ موجود (9 أكتوبر 2026) — غير مُستخدَم.**

```csharp
// OrganizationGroup.cs
public class OrganizationGroup
{
    public int GroupID { get; set; }
    public string GroupCode { get; set; }
    public string GroupNameAr { get; set; }
    public string? GroupNameEn { get; set; }
    public int? ParentGroupID { get; set; }     // للتداخل
    public bool IsActive { get; set; }
    // ... + Members
}

// OrganizationGroupMember.cs
public class OrganizationGroupMember
{
    public int GroupID { get; set; }
    public int OrganizationID { get; set; }
    public string MemberRole { get; set; }      // Main, Branch, Department
    public bool IsActive { get; set; }
    // ...
}
```

**⚠️ الفلسفة:**

> **"المستشفى = مجموعة من المنظمات (عيادات + صيدلية + معمل + غرفة عمليات + عناية مركزة + ...)."**

**⚠️ الحالة:** **تأسيس فقط** — لا UI، لا Backend.

### 2.10 ⭐ نظام Rubika (Currency)

**⚠️ موجود — Engine غير مُنفَّذ.**

```csharp
// RubikaWallet.cs
public class RubikaWallet
{
    public int WalletID { get; set; }
    public int? UserProfileID { get; set; }
    public int? OrganizationID { get; set; }
    public decimal Balance { get; set; }
    // ...
}

// RubikaTransaction.cs
public class RubikaTransaction
{
    public int TransactionID { get; set; }
    public int WalletID { get; set; }
    public string TransactionType { get; set; }  // Credit, Debit
    public decimal Amount { get; set; }
    // ...
}

// RubikaValueSetting.cs
public class RubikaValueSetting
{
    public int SettingID { get; set; }
    public decimal RubikaToEGP_Rate { get; set; }
    // ...
}
```

**⚠️ الاستخدام:**
- عملة داخلية للمنصة.
- تُستخدم للاشتراكات (PriceInRubika).
- تُستخدم للـ Redeem (PSP Points).

### 2.11 ERD — علاقات النظام الكامل

```
SystemMenus
    ├── MenuItems
    │   ├── MenuVisibilityRules
    │   └── (⭐ ServiceProductID — مقترح)
    └── MenuAssignments
        └── Organizations
            ├── OrgMemberships
            │   └── CustomJobTitles
            ├── OrganizationGroups
            │   └── OrganizationGroupMember
            ├── OrganizationSubscriptions ⭐ (جديد — مقترح)
            │   └── SubscriptionPlan
            │       └── PlanService
            │           └── ServiceProduct
            │               ├── ServiceCategory
            │               ├── ServicePricingOption
            │               ├── ServiceTarget
            │               └── PremiumService
            └── RubikaWallet
                └── RubikaTransaction
```

---

## 🎯 الجزء الثالث: تدفق الفلترة (Algorithm — مُنفَّذ)

### 3.1 المخطط العام

```
User (ClaimsPrincipal)
    │
    ├── UserProfileID ────────► OrgMemberships ────────► OrganizationID
    │                                                         │
    │                                                         ▼
    │                                             OrganizationTypeID
    │
    ├── ActiveOrganizationID ────────────────────► MenuAssignments
    │                                                         │
    │                                                         ▼
    │                                            SystemMenus + MenuItems
    │                                                         │
    │                                                         ▼
    │                                            MenuVisibilityRules
    │                                                         │
    │                                                         ▼
    │                                              القائمة النهائية
    │
    └── CustomJobTitleID ────────────────────────► (الطبقة 4 — 🟡 مُخطَّطة)
```

### 3.2 الخطوات التفصيلية

#### 🔹 Step 1: جلب المنظمة النشطة

```csharp
var activeOrgId = userSession.CurrentOrganizationId;
var membership = userSession.Memberships.FirstOrDefault(m => m.OrganizationId == activeOrgId);
var orgTypeId = membership.OrganizationTypeInfo.TypeId;
```

#### 🔹 Step 2: جلب القوائم المتاحة للـ (OrgType + Org)

```csharp
var assignments = await db.MenuAssignments
    .Where(a => a.IsActive && 
                (a.OrganizationTypeID == null || a.OrganizationTypeID == orgTypeId) &&
                (a.OrganizationID == null || a.OrganizationID == activeOrgId))
    .ToListAsync();
```

#### 🔹 Step 3: جلب الـ MenuItems + فلترة `IsVisible`

```csharp
var menuIds = assignments.Select(a => a.MenuID).Distinct().ToList();

var items = await db.MenuItems
    .Where(i => menuIds.Contains(i.MenuID) && i.IsVisible)
    .ToListAsync();
```

#### 🔹 Step 4: تطبيق `MenuVisibilityRules`

```csharp
var itemIds = items.Select(i => i.ItemID).ToList();

var rules = await _menuVisibilityService.GetActiveRulesForItemsAsync(itemIds);

var filteredItems = _menuFilteringService.ApplyRules(
    items: items,
    rules: rules,
    context: new MenuFilterContextDto
    {
        OrganizationId = activeOrgId,
        OrganizationTypeId = orgTypeId,
        SubscriptionPlanId = session.CurrentSubscriptionPlanId,    // 🟡 مُخطَّط
        UserProfileId = userProfileId,
        // CustomJobTitleIds = ... — 🟡 مُخطَّط
    });
```

### 3.3 تطبيق `Priority` في Rules

**⚠️ القاعدة:** **`Priority` الأصغر = الأولوية الأعلى.**

**الخوارزمية المُطبَّقة (من `MenuFilteringService`):**

```csharp
foreach (var item in items)
{
    var matchingRules = rules
        .Where(r => r.ItemID == item.ItemID 
                 && r.IsActive
                 && IsMatchingContext(r, context))
        .OrderBy(r => r.Priority)     // الأصغر أولاً
        .ToList();

    if (!matchingRules.Any())
    {
        item.IsVisible = true;        // لا Rules → مرئي
        continue;
    }

    var winningRule = matchingRules.First();
    item.IsVisible = winningRule.IsVisible;
}
```

### 3.4 أمثلة حية

#### مثال 1: إخفاء Item لمنظمة محددة

**السياق:**
- Item: `LOYALTY_SETTINGS`.
- المنظمة: `OrganizationID = 5`.

**القواعد:**
- Rule 1: `RuleType=All`, `IsVisible=true`, `Priority=100`.
- Rule 2: `RuleType=Organization`, `OrganizationID=5`, `IsVisible=false`, `Priority=50`.

**النتيجة:** مخفي.

#### مثال 2: إظهار Item في اشتراك مميز

**السياق:**
- Item: `LOYALTY_SCHEMES`.
- المنظمة: `SubscriptionPlanID = 3`.

**القواعد:**
- Rule 1: `RuleType=All`, `IsVisible=false`, `Priority=100`.
- Rule 2: `RuleType=Subscription`, `SubscriptionPlanID=3`, `IsVisible=true`, `Priority=50`.

**النتيجة:** مرئي.

---

## 🏗️ الجزء الرابع: الطبقات التقنية (مُنفَّذ)

### 4.1 المبدأ

> **"منطق واحد في Service، عرض واحد في Shared.UI، ثلاثة أغلفة رقيقة."**

### 4.2 التوزيع حسب الطبقة

| # | الطبقة | المكوّن | الموقع |
|:-:|--------|--------|--------|
| 1 | **Domain** | `MenuVisibilityRule` | `RubikCare.Domain/Entities/Navigation/MenuSystem/` |
| 2 | **Domain** | `SystemMenu`, `MenuItem`, `MenuAssignment` | نفس المجلد |
| 3 | **Domain** | `ServiceProduct`, `SubscriptionPlan`, `PlanService`, `ServicePricingOption`, `ServiceTarget` | `RubikCare.Domain/Entities/` |
| 4 | **Application** | `IMenuVisibilityService` | `RubikCare.Application/Interfaces/Navigation/` |
| 5 | **Application** | `IMenuFilteringService` | نفس المجلد |
| 6 | **Application** | `MenuFilterContextDto` | `RubikCare.Application/DTOs/Navigation/` |
| 7 | **Application** | `MenuFilteringService` (منطق خالص) | `RubikCare.Application/Services/Navigation/` |
| 8 | **Infrastructure** | `MenuVisibilityService` (DB) | `RubikCare.Infrastructure/Services/Navigation/` |
| 9 | **Web** | `DynamicMenuService` | `Rubikcare.Web/Data/Services/Navigation/` |

### 4.3 `IMenuVisibilityService` — المسؤوليات

```csharp
public interface IMenuVisibilityService
{
    Task<List<MenuVisibilityRule>> GetActiveRulesForItemsAsync(
        List<int> itemIds,
        CancellationToken cancellationToken = default);

    Task<List<MenuVisibilityRule>> GetAllActiveRulesAsync(
        CancellationToken cancellationToken = default);
}
```

**المسؤولية:** قراءة القواعد من DB.

### 4.4 `IMenuFilteringService` — المسؤوليات

```csharp
public interface IMenuFilteringService
{
    List<T> ApplyRules<T>(
        List<T> items,
        List<MenuVisibilityRule> rules,
        MenuFilterContextDto context)
        where T : IMenuFilterableItem;
}
```

**المسؤولية:** تطبيق القواعد — **منطق خالص (لا DB)**.

### 4.5 `MenuFilterContextDto` — السياق

```csharp
public class MenuFilterContextDto
{
    public int? OrganizationId { get; set; }
    public int? OrganizationTypeId { get; set; }
    public int? SubscriptionPlanId { get; set; }       // 🟡 مُخطَّط
    public int? UserProfileId { get; set; }
    public List<string> RoleIds { get; set; } = new(); // ⚠️ غير مستخدم
    public bool IsPlatformAdmin { get; set; }
    // public List<int> CustomJobTitleIds { get; set; } = new();  // 🟡 مُخطَّط
}
```

### 4.6 `DynamicMenuService` — التدفق

```
LoadMenuDataFromDatabaseAsync(user)
    │
    ├── GetPersonalMenuAsync(userProfileId)
    │   ├── جلب Items من SystemMenu (USER_BASE)
    │   ├── IMenuVisibilityService.GetActiveRulesForItemsAsync(itemIds)
    │   └── IMenuFilteringService.ApplyRules(items, rules, context)
    │
    ├── GetAdminMenuAsync(user)
    │   └── نفس النمط
    │
    └── LoadUserOrganizationsAsync(user)
        └── لكل Org:
            └── GetOrganizationMenuFromDatabaseAsync(orgId, orgTypeId, userProfileId)
```

---

## 🎨 الجزء الخامس: UI — ما هو موجود

### 5.1 `AdminMenuManager.razor`

**المسار:** `Rubikcare.Web/Components/Pages/Admin/GeneralSettings/AdminMenuManager.razor`

**الوظائف:**
- إدارة `SystemMenus` (إضافة/تعديل).
- إدارة `MenuItems` (إضافة/تعديل).
- إدارة الأيقونات (IconPicker).
- عرض القوائم حسب `MenuID`.

**⚠️ ما هو مفقود في UI:**
- ❌ إدارة `MenuAssignments`.
- ❌ إدارة `MenuVisibilityRules`.
- ❌ إدارة `SubscriptionPlans` للقوائم.

### 5.2 `InteractiveMenu.razor`

**المسار:** `Rubikcare.Web/Components/Layout/InteractiveMenu.razor`

**الوظائف:**
- عرض القوائم الديناميكية للمستخدم.
- يستخدم `DynamicMenuService`.

### 5.3 `ServicesHub.razor` (Admin/Services)

**المسار:** `Rubikcare.Web/Components/Pages/Admin/Services/ServicesHub.razor`

**الوظائف:**
- مركز إدارة الخدمات (UI موجود).
- يفتح على: ServiceProducts, ServicePricingOptions, ServiceTargets, SubscriptionPlans.

### 5.4 `SubscriptionPlans.razor` (Admin/Services + Admin/Pricing)

**⚠️ مكرر في مكانين:**
- `Admin/Services/SubscriptionPlans.razor`
- `Admin/Pricing/SubscriptionPlans.razor`

**⚠️ يحتاج توحيد.**

---

## 🔴 الجزء السادس: الربط المفقود (تحليل دقيق)

### 6.1 `ServiceProduct` ↔ `MenuItem`

**⚠️ الحالة:** **لا يوجد ربط مباشر.**

**⚠️ المقترح:** **`ServiceProductID INT NULL` في `MenuItem`.**

**الفلسفة:**
- `MenuItem` مع `ServiceProductID = NULL` → عنصر عام (مثل: الملف الشخصي).
- `MenuItem` مع `ServiceProductID = X` → عنصر مرتبط بميزة (مثل: نظام النقاط).

**الفائدة:**
- ربط واضح.
- سهل في الفلترة.
- `MenuVisibilityRule.SubscriptionPlanID` يقرأ منه.

### 6.2 `Organization` ↔ `SubscriptionPlan`

**⚠️ الحالة:** **لا يوجد جدول ربط.**

**⚠️ المقترح:** **`OrganizationSubscription` (جدول جديد).**

```csharp
[Table("OrganizationSubscriptions")]
public class OrganizationSubscription
{
    public int SubscriptionID { get; set; }
    public int OrganizationID { get; set; }
    public int SubscriptionPlanID { get; set; }
    public DateTime StartDate { get; set; }
    public DateTime? EndDate { get; set; }
    public bool IsActive { get; set; }
    public string BillingCycle { get; set; }
    public decimal AmountPaid { get; set; }
    public string Currency { get; set; }             // RUBIKA | EGP
    public DateTime? NextBillingDate { get; set; }
    // ...
}
```

**الفائدة:**
- منظمة قد تُشترك في أكثر من خطة.
- تواريخ واضحة.
- دورة فوترة.
- تتبع الدفع.

### 6.3 `MenuVisibilityRules` — التفعيل

**⚠️ الحالة:** **فارغة (0 سجل).**

**⚠️ المطلوب:**
1. **Migration:** `AddCustomJobTitleIdToMenuVisibilityRules`.
2. **UI لإدارة Rules.**
3. **Rules فعلية في DB** (تُملأ بناءً على الخطة).

### 6.4 `CustomJobTitleId` — Migration

**⚠️ مطلوب:**

```sql
ALTER TABLE MenuVisibilityRules
ADD CustomJobTitleId INT NULL;

ALTER TABLE MenuVisibilityRules
ADD CONSTRAINT FK_MenuVisibilityRules_CustomJobTitles
FOREIGN KEY (CustomJobTitleId) REFERENCES CustomJobTitles(CustomJobTitleId);
```

**⚠️ + إزالة `RoleId`** — بعد الانتقال الكامل.

### 6.5 `DynamicMenuComponent` — تعميم

**⚠️ الحالة:** **غير موجود.**

**⚠️ المقترح:**

```razor
@* Shared.UI/Components/Navigation/DynamicMenuComponent.razor *@
@using RubikCare.Shared.UI.Services

@code {
    [Parameter] public IApiService? ApiService { get; set; }
    [Parameter] public ISharedTranslationService? TranslationService { get; set; }
    // ...
}
```

**⚠️ الاستخدام:**
- **Web:** `DynamicMenuComponent` مباشرة (لأنها على نفس الخادم).
- **PWA:** `DynamicMenuComponent` + `WebApiService`.
- **MAUI:** `DynamicMenuComponent` + `MauiApiService`.

---

## ⚖️ الجزء السابع: القرارات المطلوبة (6 قرارات)

### قرار 1: `ServiceProductID` في `MenuItem`

**الخيارات:**

| # | الخيار | التوصية |
|:-:|:---|:---:|
| A | `ServiceProductID INT NULL` في `MenuItem` | ⭐ **موصى** |
| B | جدول `ServiceMenuItem` (M:N) | — |
| C | عبر `ServiceTarget` فقط | — |

**🎯 التوصية: A** — بسيط، واضح، مرن.

### قرار 2: `OrganizationSubscription`

**الخيارات:**

| # | الخيار | التوصية |
|:-:|:---|:---:|
| A | جدول جديد `OrganizationSubscription` | ⭐ **موصى** |
| B | `SubscriptionPlanID` في `Organization` | — |
| C | لا ربط | — |

**🎯 التوصية: A** — يدعم خطط متعددة + تواريخ.

### قرار 3: `CustomJobTitleId` في `MenuVisibilityRule`

**🎯 التوصية: إضافة `CustomJobTitleId INT NULL` الآن (Migration).**

### قرار 4: `ServiceNature` — القيم

**🎯 التوصية: تأجيل — ليس ضرورياً للربط الحالي.**

### قرار 5: `DynamicMenuComponent` — متى؟

**🎯 التوصية: توثيق فقط الآن — التنفيذ لاحقاً.**

### قرار 6: `OrganizationGroup` — متى؟

**🎯 التوصية: توثيق فقط الآن — التنفيذ لاحقاً.**

---

## 🗄️ الجزء الثامن: إعداد قاعدة البيانات

### 8.1 تصحيح بيانات SQL

**⚠️ تصحيح `HOSPITAL_MENU` → `DRUG_WAREHOUSE_MENU`:**

```sql
-- ⚠️ استخدم N'' للعربية
MERGE SystemMenus AS target
USING (VALUES
    (11, N'DRUG_WAREHOUSE_MENU', N'مخزن الأدوية', N'Drug Warehouse Menu', N'ORG_TYPE_BASED')
) AS source (MenuID, MenuCode, MenuNameAr, MenuNameEn, MenuType)
ON target.MenuID = source.MenuID
WHEN MATCHED THEN
    UPDATE SET
        MenuCode = source.MenuCode,
        MenuNameAr = source.MenuNameAr,
        MenuNameEn = source.MenuNameEn,
        MenuType = source.MenuType,
        LastModifiedDate = GETDATE();
```

### 8.2 Migrations المطلوبة الآن

**⚠️ 3 Migrations — قبل أي بيانات تشغيلية:**

**1. `AddCustomJobTitleIdToMenuVisibilityRules`:**
```csharp
migrationBuilder.AddColumn<int>(
    name: "CustomJobTitleId",
    table: "MenuVisibilityRules",
    type: "int",
    nullable: true);

migrationBuilder.CreateIndex(
    name: "IX_MenuVisibilityRules_CustomJobTitleId",
    table: "MenuVisibilityRules",
    column: "CustomJobTitleId");

migrationBuilder.AddForeignKey(
    name: "FK_MenuVisibilityRules_CustomJobTitles_CustomJobTitleId",
    table: "MenuVisibilityRules",
    column: "CustomJobTitleId",
    principalTable: "CustomJobTitles",
    principalColumn: "CustomJobTitleId",
    onDelete: ReferentialAction.SetNull);
```

**2. `CreateOrganizationSubscriptionTable`:**
```csharp
migrationBuilder.CreateTable(
    name: "OrganizationSubscriptions",
    columns: table => new
    {
        SubscriptionID = table.Column<int>(nullable: false).Annotation("SqlServer:Identity", "1, 1"),
        OrganizationID = table.Column<int>(nullable: false),
        SubscriptionPlanID = table.Column<int>(nullable: false),
        StartDate = table.Column<DateTime>(nullable: false),
        EndDate = table.Column<DateTime>(nullable: true),
        IsActive = table.Column<bool>(nullable: false, defaultValue: true),
        BillingCycle = table.Column<string>(maxLength: 50, nullable: false),
        AmountPaid = table.Column<decimal>(type: "decimal(18,2)", nullable: false),
        Currency = table.Column<string>(maxLength: 10, nullable: false, defaultValue: "RUBIKA"),
        NextBillingDate = table.Column<DateTime>(nullable: true),
        CreatedDate = table.Column<DateTime>(nullable: false, defaultValueSql: "GETUTCDATE()"),
        LastModifiedDate = table.Column<DateTime>(nullable: true)
    },
    constraints: table =>
    {
        table.PrimaryKey("PK_OrganizationSubscriptions", x => x.SubscriptionID);
        table.ForeignKey("FK_OrganizationSubscriptions_Organizations_OrganizationID",
            x => x.OrganizationID, "Organizations", "OrganizationID", onDelete: ReferentialAction.Cascade);
        table.ForeignKey("FK_OrganizationSubscriptions_SubscriptionPlans_SubscriptionPlanID",
            x => x.SubscriptionPlanID, "SubscriptionPlans", "PlanID", onDelete: ReferentialAction.Restrict);
    });
```

**3. `AddServiceProductIdToMenuItems`:**
```csharp
migrationBuilder.AddColumn<int>(
    name: "ServiceProductID",
    table: "MenuItems",
    type: "int",
    nullable: true);

migrationBuilder.CreateIndex(
    name: "IX_MenuItems_ServiceProductID",
    table: "MenuItems",
    column: "ServiceProductID");

migrationBuilder.AddForeignKey(
    name: "FK_MenuItems_ServiceProducts_ServiceProductID",
    table: "MenuItems",
    column: "ServiceProductID",
    principalTable: "ServiceProducts",
    principalColumn: "ServiceProductID",
    onDelete: ReferentialAction.SetNull);
```

### 8.3 خطوات التنفيذ

```powershell
# 1. Backup DB
# 2. تصحيح HOSPITAL_MENU (SQL مباشر — Data Migration)
# 3. Add-Migration AddCustomJobTitleIdToMenuVisibilityRules
# 4. Update-Database
# 5. Add-Migration CreateOrganizationSubscriptionTable
# 6. Update-Database
# 7. Add-Migration AddServiceProductIdToMenuItems
# 8. Update-Database
# 9. التحقق من __EFMigrationsHistory
```

---

## 🔧 الجزء التاسع: Sidebar Direction Stability (دمج 17)

### 9.1 المشكلة الموثقة

**⚠️ الأعراض (بالعربية):**
- القائمة الجانبية تقفز لليسار عند التنقل.
- تظل عالقة حتى يتم تغيير اللغة يدوياً مرتين.

### 9.2 السبب الجذري

**⚠️:** `MainLayout.razor` يرسم `<div class="main-wrapper">` بدون سمة `dir`. عند إعادة الرسم، يفقد المتصفح السياق.

### 9.3 الحل النهائي

**⚠️ في `MainLayout.razor`:**

```razor
<div class="main-wrapper" dir="@(TranslationState.CurrentLanguage == "ar" ? "rtl" : "ltr")">
```

**⚠️ في `direction-protector.js`:**
- يقرأ `localStorage` فوراً عند التحميل.
- يضبط `html[dir]` و `body[dir]` قبل Blazor.

### 9.4 الملفات المحظور تعديلها

| الملف | القاعدة |
|:---|:---|
| `MainLayout.razor` | **ممنوع** إزالة `dir` من `.main-wrapper` |
| `_sidebar.css` | **ممنوع** استبدال `.main-wrapper[dir="rtl"]` بـ `html[dir="rtl"]` |
| `InteractiveMenu.razor` | **ممنوع** `Navigation.NavigateTo(url, forceLoad: true)` |
| `direction-protector.js` | **ممنوع** `MutationObserver` عدواني |

### 9.5 Testing Checklist

- [ ] تسجيل دخول عربي: القائمة يمين — بدون وميض.
- [ ] التنقل الداخلي: لا قفزة.
- [ ] تبديل الأوضاع: يحافظ على اللغة والاتجاه.
- [ ] تحديث الصفحة: يعيد اللغة والاتجاه.
- [ ] التبديل للإنجليزية: القائمة يسار.

---

## 🌐 الجزء العاشر: Web Localization Direction (دمج 19)

### 10.1 المشكلة

**⚠️:** `TranslationStateService` (Scoped) — في كل مرحلة (SSR/Interactive) ينشئ instance جديد.

### 10.2 التسلسل الزمني

```
1. المتصفح يطلب GET /settings
2. SSR Phase: TranslationStateService (Instance #1) → يقرأ cookie
3. المتصفح يستلم HTML: script مبكر يضبط html[dir]
4. Interactive Phase: TranslationStateService (Instance #2) → httpContext = null → افتراضي "ar"
5. بعد الاتصال: InteractiveMenu.OnAfterRenderAsync → SyncFromLocalStorageAsync
```

### 10.3 الحل — 4 طبقات

#### الطبقة 1: السكربت المبكر في `App.razor`

```html
<script>
    (function() {
        try {
            var lang = 'ar';
            var cookieMatch = document.cookie.match(/(?:^|;\s*)\.RubikCare\.Language=([^;]*)/);
            if (cookieMatch) {
                var decoded = decodeURIComponent(cookieMatch[1]);
                var langMatch = decoded.match(/c=([a-z]{2})/);
                if (langMatch) lang = langMatch[1];
            } else {
                var stored = localStorage.getItem('RubikCare:Language');
                if (stored === 'ar' || stored === 'en') lang = stored;
            }
            var dir = lang === 'ar' ? 'rtl' : 'ltr';
            document.documentElement.setAttribute('dir', dir);
            document.documentElement.setAttribute('lang', lang);
            document.body.setAttribute('dir', dir);
        } catch (e) {
            document.documentElement.setAttribute('dir', 'rtl');
        }
    })();
</script>
```

#### الطبقة 2: `direction-protector.js`

**⚠️:** `MutationObserver` يراقب `html[dir]` ويمنع التغيير الخاطئ.

#### الطبقة 3: `TranslationStateService.InitializeAsync` (SSR)

**⚠️:** يقرأ cookie فقط — **لا JavaScript**.

#### الطبقة 4: `SyncFromLocalStorageAsync` في `InteractiveMenu`

**⚠️:** في `OnAfterRenderAsync` — **أول شيء** قبل أي عملية.

### 10.4 قواعد إلزامية

1. **لا تضع `dir` على أي عنصر Blazor SSR** — فقط `<html>` عبر JavaScript.
2. **`SyncFromLocalStorageAsync` فقط في `@rendermode InteractiveServer`.**
3. **`InitializeAsync` لا تستدعي JavaScript.**
4. **CSS يقرأ `html[dir]` فقط.**
5. **اسم الـ cookie: `.RubikCare.Language`** (بنقطة).
6. **`localStorage` هو المصدر الموثوق في Interactive mode.**

---

## 🔧 الجزء الحادي عشر: EF Core Migration — ملاحظات حرجة

### 11.1 المبدأ

> **"Migration المُولَّد صحيح — فقط عندما يكون Snapshot متزامناً مع DB."**

### 11.2 القاعدة الصارمة

> **"قبل أي `Add-Migration` — تأكد أن Snapshot = DB."**

### 11.3 الممنوعات

| # | الممنوع | السبب |
|:-:|:---|:---|
| 1 | تعديل Migration files يدوياً | تُلغى عند أول `Add-Migration` |
| 2 | تعديل `ModelSnapshot.cs` كروتين | يفقد الاتساق |
| 3 | SQL مباشر لتعديل DB — كبديل | يخلق فجوة |
| 4 | حذف Migration file من DB بدون `Remove-Migration` | يكسر `__EFMigrationsHistory` |
| 5 | تعديل DB دون Migration — كروتين | يُفسد التوازن |

### 11.4 الاستثناءات

| # | الاستثناء | الشرط |
|:-:|:---|:---|
| 1 | SQL مباشر للبيانات (Data Migration) | عند نقل بيانات |
| 2 | تعديل Snapshot يدوياً | لمرة واحدة — مع تحقق فوري |
| 3 | `Remove-Migration` | لحذف Migration غير مُطبَّق |
| 4 | Backup DB | قبل أي Migration حساس |

### 11.5 Checklist

- [ ] Backup DB كامل.
- [ ] Snapshot = DB؟
- [ ] `OnModelCreating` يُطابق Entity؟
- [ ] `Add-Migration` — مراجعة Migration المُولَّد.
- [ ] `Update-Database` — مراقبة الأخطاء.

---

## 🎯 الجزء الثاني عشر: Roadmap المُتبقي

### 12.1 ما هو مطلوب

| # | المهمة | الأولوية | الحالة |
|:-:|:---|:---:|:---:|
| 1 | **تصحيح `HOSPITAL_MENU` → `DRUG_WAREHOUSE_MENU`** | 🔴 عالية | 🟡 SQL جاهز |
| 2 | **Migration: `CustomJobTitleId` في MenuVisibilityRules** | 🔴 عالية | 🟡 جاهز |
| 3 | **Migration: `OrganizationSubscription` table** | 🔴 عالية | 🟡 جاهز |
| 4 | **Migration: `ServiceProductID` في MenuItems** | 🔴 عالية | 🟡 جاهز |
| 5 | **UI لإدارة `MenuVisibilityRules`** | 🟠 متوسطة | ❌ |
| 6 | **UI لإدارة `OrganizationSubscription`** | 🟠 متوسطة | ❌ |
| 7 | **UI لربط `SubscriptionPlan` بـ `MenuItem`** | 🟠 متوسطة | ❌ |
| 8 | **إزالة `RoleId` من `MenuVisibilityRules`** | 🟠 متوسطة | ❌ |
| 9 | **إزالة `SystemMenuMenuID` (Shadow)** | 🟠 متوسطة | ❌ |
| 10 | **`DynamicMenuComponent` في Shared.UI** | 🟠 متوسطة | ❌ |
| 11 | **تعميم Menus على PWA + MAUI** | 🟠 متوسطة | ❌ |
| 12 | **`OrganizationGroup` UI + Backend** | 🟡 منخفضة | ❌ |

### 12.2 الاعتمادات

```
1 → لا يعتمد على شيء
2 → لا يعتمد على شيء
3 → لا يعتمد على شيء (يحتاج SubscriptionPlans مملوء)
4 → لا يعتمد على شيء
5 → يعتمد على 2
6 → يعتمد على 3
7 → يعتمد على 4 + 5 + 6
8 → يعتمد على 2
9 → مستقل
10 → يعتمد على 5
11 → يعتمد على 10
12 → مستقل (تأسيس)
```

---

## 📝 الجزء الثالث عشر: سجل التغييرات

| الإصدار | التاريخ | التغييرات |
|:---:|:---|:---|
| **1.0** | 18 يوليو 2026 | الإصدار الأولي — `04-dynamic-menus.md` |
| **2.0** | 24 مايو 2026 | إزالة `IMemoryCache` — كاش مركزي |
| **3.0** | 13 أكتوبر 2026 | `04-dynamic-menus.md` v3.0 — دمج `27` |
| **4.0** | 16 أكتوبر 2026 | **`03-MENUS` v4.0** — تصحيح `ServiceProduct` + تصحيح `HOSPITAL_MENU` + نظام الخدمات والميزات + نظام جروبات المنظمات + دمج `17` + `19` + القرارات الـ 6 + Migrations المُقترَحة |

---

## 🔗 روابط ذات صلة

### 📚 وثائق أساسية

- [00 — Architecture Overview](./00-architecture-overview.md)
- [01 — Program.cs Foundation](./01-program-cs-foundation.md)
- [02 — Identity System](./02-identity-system.md)
- [05 — Page Creation](./05-page-creation-checklist.md)
- [08 — Strategic Roadmap](./08-Strategic-Roadmap.md)
- [14 — Caching System](./14-caching-system.md)

### 📚 وثائق مرتبطة بالمنصات

- [10 — MAUI Development](./10-maui-development-guide.md)
- [11 — BlazorWebView](./11-blazor-webview-guide.md)
- [22 — PWA](./22-RubikCare.PWA.md)

### 📚 وثائق مرتبطة بالتنفيذ

- [15 — Translation System](./15-translation-system.md)
- [26 — Rubika Points System](./26-rubika-points-system.md)
- [27 — Reports System](./27-reports-system.md)

---

**© 2026 RubikCare — للاستخدام الداخلي**

**نهاية الوثيقة ✅**
```
