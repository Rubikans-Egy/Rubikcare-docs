# 04 — نظام القوائم الديناميكية والصلاحيات
# (Dynamic Menu & Permissions System)

**الإصدار:** 3.0 (Unified Edition)
**آخر تحديث:** 13 أكتوبر 2026
**الحالة:** ✅ مرجع معماري إلزامي — يقرأه كل مطور قبل العمل على القوائم
**المؤلف:** RubikCare Team

---

## 📌 الجزء صفر: الرؤية والفلسفة (نص المؤلف الحرفي)

> **⚠️ ملاحظة:**
> هذا القسم يحتوي على **نص الكاتب الحرفي** — دون إعادة صياغة.
> الهدف: حفظ الرؤية الكاملة كما صيغت.

---

### 0.1 الرؤية الكاملة للنظام

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

### 0.2 سبب الحاجة للنظام (الربط بنظام النقاط)

> **نص المؤلف الحرفي:**
>
> "لماذا أخبرك بهذه الجزئية الآن؟
> أخبرك بها لأننا سنحتاج بعد الانتهاء من backend ما ننجزه الآن أن نفكر في مراعاة هذه الجزئية المذكورة في تصميم الواجهات، وتعديل الوضع الحالي ليسمح بمرونة أكثر، بدلاً من كونها ميزة مدمجة في داخل برامج الدعم.
>
> فلا نحتاج مستقبلاً لتطوير واجهات مخصصة لكل خدمة نريد إضافة نظام نقاط لها."

**الترجمة التقنية:**

**المشكلة الحالية:**
- نظام النقاط **مدمج داخل PSP**.
- لو أردنا نظام نقاط في B2B → نسخ كود + تعديلات.

**بعد الوثيقة:**
- نظام النقاط يصبح **ميزة (Feature)**.
- المنظمة تشترك في **Plan** يحتوي على `POINTS_SYSTEM`.
- **لا كود مخصص لكل خدمة.**
- **نفس الميزة، نفس الواجهة، نفس المحرك.**

---

### 0.3 تعميم النظام على كل المنصات

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

### 0.4 فلسفة "التطوير التراكمي الآمن"

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

هذه الوثيقة تُعرّف نظامًا معماريًا موحّدًا للتحكم في القوائم والصلاحيات والميزات عبر المنصات الثلاث (Blazor Web + PWA + MAUI)، من خلال **أربع طبقات تحكم** مستقلة ومتكاملة، مع **آلية Rules** ديناميكية.

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
│  الطبقة 2: الاشتراكات (Subscriptions) — 🟡 مُخطَّطة      │
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

**⚠️ ملاحظة:**
- **الطبقة 1** — ✅ مُنفَّذة بالكامل.
- **الطبقة 2** — 🟡 مُخطَّطة — تحتاج `SubscriptionPlans` مُفعَّلة + `MenuVisibilityRules`.
- **الطبقة 3** — ⚠️ `MenuVisibilityRules` **موجود** في DB — لكن **فارغ** — يحتاج UI لإدارته.
- **الطبقة 4** — 🟡 مُخطَّطة — يحتاج إضافة `CustomJobTitleId` إلى `MenuVisibilityRules` (Migration).

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

**⚠️ مثال من الواقع (Pharma Company):**

```
OrganizationTypeID = 2 → PHARMA_PSP (ORG_TYPE_BASED)
OrganizationTypeID = 2 → LOYALTY (FEATURE_BASED) — يحتاج OrganizationID محدد؟ لا — عام لـ Pharma

OrganizationTypeID = 4 → CLINIC_MENU (ORG_TYPE_BASED)
OrganizationTypeID = 3 → PHARMACY_MENU (ORG_TYPE_BASED)
OrganizationTypeID = 5 → HOSPITAL_MENU (ORG_TYPE_BASED)
```

**⚠️ التطبيق:** عند بناء القائمة لمستخدم في Pharma:
- **المرحلة الأولى:** جلب `MenuAssignments` بـ `OrganizationTypeID = 2`.
- **النتيجة:** `PHARMA_PSP` + `LOYALTY`.
- **المرحلة الثانية:** فلترة `MenuItems` بـ `IsVisible = 1`.

### 1.5 الطبقة 2: الاشتراكات (Subscriptions) — 🟡 مُخطَّطة

#### الوصف
- **من يتحكم؟** Super Admin.
- **ماذا يتحكم؟** الميزات المفعّلة في اشتراك كل منظمة.
- **المعيار:** `MenuVisibilityRules` — مع `SubscriptionPlanID`.
- **الحالة:** 🟡 **مُخطَّطة** — البنية موجودة (`MenuVisibilityRules.SubscriptionPlanID`)، لكن **UI إدارة الاشتراكات غير موجود**.

#### الوظائف المُخطَّطة
- تعريف الميزات (`Features`) — **⚠️ الجدول غير موجود حالياً**.
- تعريف الخطط (`Plans`) — **⚠️ الجدول غير موجود — لكن `SubscriptionPlans` موجود**.
- ربط الميزات بالخطط (`PlanFeatures`) — **⚠️ الجدول غير موجود حالياً**.
- ربط المنظمات بالخطط (`OrganizationSubscriptions`) — **⚠️ الجدول غير موجود حالياً**.
- تفعيل/تعطيل ميزة لمنظمة معينة.

#### الجداول المُخطَّطة
- `Features` — 🟡 **مطلوب إنشاؤه**.
- `Plans` — 🟡 **مطلوب إنشاؤه** — أو استخدام `SubscriptionPlans` الموجود.
- `PlanFeatures` — 🟡 **مطلوب إنشاؤه**.
- `OrganizationSubscriptions` — 🟡 **مطلوب إنشاؤه**.
- `FeatureMenuItems` — 🟡 **مطلوب إنشاؤه** (يربط الميزة بعناصر القائمة).

#### آلية العمل — التصميم
عند بناء القائمة لمستخدم:
1. يُحدد اشتراك المنظمة النشطة.
2. تُحدد الميزات المفعّلة.
3. تُفلتر `MenuItems` بحيث تظهر فقط العناصر المرتبطة بميزات مفعّلة — عبر `MenuVisibilityRules (SubscriptionPlanID)`.

**⚠️ ملاحظة:** هذه الطبقة **ستُفعَّل لاحقاً** — بالتزامن مع بناء نظام الاشتراكات.

### 1.6 الطبقة 3: المنظمة (Organization Control) — ⚠️ قيد الإعداد

#### الوصف
- **من يتحكم؟** Admin المنظمة.
- **ماذا يتحكم؟** ما سمحت به المنظمة لعناصر قوائمها (إظهار/إخفاء).
- **المعيار:** `MenuVisibilityRules` — مع `OrganizationID`.
- **الحالة:** ⚠️ `MenuVisibilityRules` **موجود** — لكن **فارغ** — يحتاج UI لإدارته.

#### الوظائف المُخطَّطة
- عرض عناصر قوائم المنظمة (المتاحة وفق اشتراكها).
- إظهار/إخفاء عنصر على مستوى المنظمة (لكل الوظائف).
- **لا يضيف** عناصر جديدة.
- **لا ينقل** عناصر بين قوائم.

#### الجداول المستخدمة
- `MenuVisibilityRules` — مع `RuleType = Organization`.

#### الفرق بين الطبقة 1 والطبقة 3

| البند | الطبقة 1 (المنصة) | الطبقة 3 (المنظمة) |
|-------|---------------------|----------------------|
| **من يتحكم؟** | Super Admin | Admin المنظمة |
| **النطاق** | كل المنظمات | منظمة واحدة |
| **الوظائف** | إضافة/نقل/إخفاء | إظهار/إخفاء فقط |
| **المستوى** | `MenuAssignment` + `MenuItems.IsVisible` | `MenuVisibilityRules (OrganizationID)` |

### 1.7 الطبقة 4: الوظيفة (JobTitle) — 🟡 مُخطَّطة

#### الوصف
- **من يتحكم؟** Admin المنظمة.
- **ماذا يتحكم؟** ما تراه كل وظيفة داخل المنظمة.
- **المعيار:** `MenuVisibilityRules` — مع `CustomJobTitleId`.
- **الحالة:** 🟡 **مُخطَّطة** — تحتاج:
  - إضافة `CustomJobTitleId` إلى `MenuVisibilityRules` — **Migration مطلوبة**.
  - ربط `CustomJobTitle` بـ `OrgMemberships`.
  - UI لإدارة صلاحيات الوظائف.

#### الوظائف المُخطَّطة
- عرض وظائف المنظمة (`CustomJobTitles`).
- ربط كل وظيفة بعناصر مرئية.
- معاينة القائمة كما سيراها المستخدم بهذه الوظيفة.

#### الجداول المُخطَّطة
- `CustomJobTitles` — ✅ موجود.
- `OrgMemberships` — ✅ موجود (يربط `UserProfile` بـ `CustomJobTitle`).
- `MenuVisibilityRules.CustomJobTitleId` — 🟡 **مطلوب إضافته (Migration)**.

**⚠️ ملاحظة مهمة:**
- **`RoleId`** (في `AspNetRoles`) — **لا يُستخدم لإدارة القوائم** — لأنه للأدوار المهنية (`Doctor`, `Pharmacist`).
- **`CustomJobTitleId`** — هو المعيار الصحيح — لأنه يحدد **الوظيفة داخل المنظمة**.

### 1.8 ترتيب الأولويات بين الطبقات

عند بناء القائمة لمستخدم، تُطبّق الطبقات بالترتيب التالي:

```
1. الطبقة 1: تحديد القوائم المتاحة (MenuAssignments)
   ↓
2. الطبقة 2: فلترة القوائم وفق الاشتراك (MenuVisibilityRules — SubscriptionPlanID) 🟡
   ↓
3. الطبقة 3: فلترة العناصر وفق إعدادات المنظمة (MenuVisibilityRules — OrganizationID) ⚠️
   ↓
4. الطبقة 4: فلترة العناصر وفق وظيفة المستخدم (MenuVisibilityRules — CustomJobTitleId) 🟡
   ↓
5. القائمة النهائية
```

#### قاعدة الأولوية
> **"الطبقة الأدق تتجاوز الأعلى عمومية."**

- إذا أظهرت الطبقة 3 عنصرًا، لكن الطبقة 4 حجبته عن وظيفة المستخدم → **لا يظهر**.
- إذا حجبت الطبقة 3 عنصرًا، لكن الطبقة 4 سمحت به → **لا يظهر** (لأن الطبقة 3 أعلى).
- **الاستثناء:** `UserProfileID` في `MenuAssignment` (تخصيص مباشر) يتجاوز كل الطبقات — **إذا أُضيف مستقبلاً**.

---

## 🗄️ الجزء الثاني: نموذج البيانات (ERD)

### 2.1 الجداول الأساسية — ✅ موجودة

| # | الجدول | الدور |
|:-:|--------|-------|
| 1 | `SystemMenus` | القوائم الرئيسية (USER_BASE, ADMIN_MENU, LOYALTY, ...) |
| 2 | `MenuItems` | عناصر القوائم (Items) |
| 3 | `MenuAssignments` | ربط القوائم بالمنظمات وأنواعها |
| 4 | `MenuVisibilityRules` | قواعد الإظهار/الإخفاء الديناميكية |
| 5 | `CustomJobTitles` | وظائف الأعضاء |
| 6 | `OrgMemberships` | عضويات المستخدمين — تربط المستخدم بـ `CustomJobTitle` |

### 2.2 `SystemMenus` — البنية الفعلية

```sql
CREATE TABLE SystemMenus (
    MenuID INT PRIMARY KEY IDENTITY(1,1),
    MenuCode NVARCHAR(50) NOT NULL,
    MenuNameAr NVARCHAR(100) NOT NULL,
    MenuNameEn NVARCHAR(100) NULL,
    MenuType NVARCHAR(20) NOT NULL,   -- BASE | ROLE_BASED | ORG_TYPE_BASED | FEATURE_BASED
    SortOrder INT NOT NULL DEFAULT 100,
    Description NVARCHAR(500) NULL,
    IsActive BIT NOT NULL DEFAULT 1,
    CreatedDate DATETIME2 NOT NULL DEFAULT GETDATE()
);
```

**القيم الفعلية:**

| MenuID | MenuCode | MenuNameAr | MenuType | SortOrder |
|:------:|----------|------------|----------|:---------:|
| 2 | USER_BASE | القائمة الشخصية | `BASE` | 10 |
| 13 | LOYALTY | نظام النقاط | `FEATURE_BASED` | 15 |
| 3 | ADMIN_MENU | لوحة تحكم الإدارة | `ROLE_BASED` | 20 |
| 4 | PHARMA_PSP | شركات الأدوية | `ORG_TYPE_BASED` | 30 |
| 9 | CLINIC_MENU | العيادة | `ORG_TYPE_BASED` | 40 |
| 10 | PHARMACY_MENU | الصيدلية | `ORG_TYPE_BASED` | 45 |
| 11 | HOSPITAL_MENU | مستشفى | `ORG_TYPE_BASED` | 50 |

### 2.3 `MenuType` — القيم المُعتمَدة

| القيمة | الغرض | مثال |
|:---:|:---|:---|
| **`BASE`** | قوائم عامة لكل المستخدمين | `USER_BASE` |
| **`ROLE_BASED`** | قوائم مرتبطة بالدور النظامي | `ADMIN_MENU` |
| **`ORG_TYPE_BASED`** | قوائم مرتبطة بنوع المنظمة | `PHARMA_PSP`, `CLINIC_MENU` |
| **`FEATURE_BASED`** | قوائم مرتبطة بميزة (اشتراك/خطة) | `LOYALTY` |

**⚠️ ملاحظة:**
- هذه القيم **4 نهائية**.
- **قيم مستقبلية محتملة:** `USER_SPECIFIC`, `HYBRID` — تُضاف عند الحاجة.

### 2.4 `MenuItems` — البنية الفعلية

```sql
CREATE TABLE MenuItems (
    ItemID INT PRIMARY KEY IDENTITY(1,1),
    MenuID INT NOT NULL FOREIGN KEY REFERENCES SystemMenus(MenuID),
    ParentItemID INT NULL FOREIGN KEY REFERENCES MenuItems(ItemID),
    ItemCode NVARCHAR(100) NOT NULL UNIQUE,
    ItemNameAr NVARCHAR(100) NOT NULL,
    ItemNameEn NVARCHAR(100) NULL,
    Url NVARCHAR(200) NOT NULL,
    IconClass NVARCHAR(50) NULL,
    SortOrder INT NOT NULL DEFAULT 100,
    IsVisible BIT NOT NULL DEFAULT 1,
    RequiresPermission BIT NOT NULL DEFAULT 0,
    PermissionCode NVARCHAR(50) NULL,
    OpenInNewTab BIT NOT NULL DEFAULT 0,
    Tooltip NVARCHAR(500) NULL,
    CreatedDate DATETIME2 NOT NULL DEFAULT GETDATE(),
    LastModifiedDate DATETIME2 NULL
);
```

**⚠️ ملاحظات مهمة:**
- **`ItemCode`** — كود فريد لكل عنصر.
- **`IsVisible`** — بديل عن `IsActive` (غير موجود).
- **`RequiresPermission`** — bit — مع `PermissionCode` (منفصل).
- **`OpenInNewTab`** — فتح الرابط في تبويب جديد.
- **`Tooltip`** — نص تلميح.
- **`ParentItemID`** — يدعم Hierarchy نظرياً — لكن **غير مستخدم حالياً** (كل Items root).

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
- **لا يوجد `AssignmentType`** — عكس ما ذكرته النسخ السابقة.
- **لا يوجد `UserProfileID`** ولا **`RoleId`** — التخصيص عبر `OrganizationTypeID` و `OrganizationID` فقط.
- **⚠️ دين تقني:** عمود `SystemMenuMenuID` (Shadow Property) موجود — **يجب حذفه** (Migration مستقبلي).

**أمثلة حية:**

| AssignmentID | MenuID | OrganizationTypeID | OrganizationID | ملاحظة |
|:---:|:---:|:---:|:---:|:---|
| 5 | 2 | NULL | NULL | USER_BASE — عام |
| 2 | 3 | NULL | 1 | ADMIN_MENU — لمؤسسة رقم 1 |
| 4 | 4 | 2 | NULL | PHARMA_PSP — لكل شركات الأدوية |
| 14 | 13 | 2 | NULL | LOYALTY — لكل شركات الأدوية |

**⚠️ التطبيق:**
- **`OrganizationTypeID`** — الأولوية العليا — لضمان ظهور القائمة الصحيحة وفق نوع المنظمة.
- **`OrganizationID`** — الأولوية الثانية — لظهور التفاصيل الخاصة وفق المنظمة المحددة.

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

**الفهارس (6):**
- `IX_MenuVisibilityRules_ItemID_OrganizationID_IsActive`
- `IX_MenuVisibilityRules_ItemID_OrganizationTypeID_IsActive`
- `IX_MenuVisibilityRules_ItemID_SubscriptionPlanID_IsActive`
- `IX_MenuVisibilityRules_ItemID_RoleId_IsActive`
- `IX_MenuVisibilityRules_ItemID_Priority_IsActive`
- `PK_MenuVisibilityRules` (Clustered على `RuleID`)

**FK واحد:**
- `FK_MenuVisibilityRules_MenuItems_ItemID` → `MenuItems.ItemID`

**⚠️ الحالة الحالية:** **0 سجلات** — الجدول فارغ.

**⚠️ التغييرات المُخطَّطة:**
- **إضافة `CustomJobTitleId INT NULL`** — لدعم الطبقة 4.
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
```

[#100] 📄 `04-dynamic-menus.md` — الإصدار 3.0 (الجزء الثاني)

---

```markdown
## 🎯 الجزء الثالث: تدفق الفلترة (Algorithm)

### 3.1 المخطط العام

```
User (ClaimsPrincipal)
    │
    ├── UserProfileID ────────► OrgMemberships ────────► OrganizationID
    │                                                         │
    │                                                         │
    │                                                         ▼
    │                                             OrganizationTypeID
    │
    ├── ActiveOrganizationID ────────────────────► MenuAssignments
    │                                                         │
    │                                                         │
    │                                                         ▼
    │                                            SystemMenus + MenuItems
    │                                                         │
    │                                                         │
    │                                                         ▼
    │                                            MenuVisibilityRules
    │                                                         │
    │                                                         │
    │                                                         ▼
    │                                              القائمة النهائية
    │
    └── CustomJobTitleID ────────────────────────► (الطبقة 4 — 🟡 مُخطَّطة)
```

### 3.2 الخطوات التفصيلية

#### 🔹 Step 1: **جلب المنظمة النشطة**

```csharp
var activeOrgId = userSession.CurrentOrganizationId;
var membership = userSession.Memberships.FirstOrDefault(m => m.OrganizationId == activeOrgId);
var orgTypeId = membership.OrganizationTypeInfo.TypeId;
```

#### 🔹 Step 2: **جلب القوائم المتاحة للـ (OrgType + Org)**

```csharp
var assignments = await db.MenuAssignments
    .Where(a => a.IsActive && 
                (a.OrganizationTypeID == null || a.OrganizationTypeID == orgTypeId) &&
                (a.OrganizationID == null || a.OrganizationID == activeOrgId))
    .ToListAsync();
```

**⚠️ ملاحظة مهمة:**
- **الترتيب:** `OrganizationTypeID` أولاً — لضمان **القوائم العامة للنوع**.
- **ثم:** `OrganizationID` — لإضافة **قوائم خاصة بالمنظمة**.

#### 🔹 Step 3: **جلب الـ MenuItems + فلترة `IsVisible`**

```csharp
var menuIds = assignments.Select(a => a.MenuID).Distinct().ToList();

var items = await db.MenuItems
    .Where(i => menuIds.Contains(i.MenuID) && i.IsVisible)
    .ToListAsync();
```

#### 🔹 Step 4: **تطبيق `MenuVisibilityRules`**

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
        // RoleIds = ... — ❌ غير مستخدم لإدارة القوائم
        // CustomJobTitleIds = ... — 🟡 مُخطَّط
    });
```

### 3.3 تطبيق `Priority` في Rules

**⚠️ القاعدة الأساسية:**

> **`Priority` الأصغر = الأولوية الأعلى.**

**خوارزمية التطبيق:**

```csharp
foreach (var item in items)
{
    // جلب كل Rules التي تطابق هذا الـ Item + السياق
    var matchingRules = rules
        .Where(r => r.ItemID == item.ItemID 
                 && r.IsActive
                 && IsMatchingContext(r, context))
        .OrderBy(r => r.Priority)     // الأصغر أولاً
        .ToList();

    if (!matchingRules.Any())
    {
        // لا Rules → الافتراضي: مرئي
        item.IsVisible = true;
        continue;
    }

    // الأعلى أولوية = الأول في القائمة
    var winningRule = matchingRules.First();
    item.IsVisible = winningRule.IsVisible;
}
```

### 3.4 أمثلة حية

#### مثال 1: **إخفاء Item لمنظمة محددة**

**السياق:**
- Item: `LOYALTY_SETTINGS` — من `LOYALTY_MENU`.
- المنظمة: `OrganizationID = 5`.

**القواعد:**
- Rule 1: `RuleType=All`, `IsVisible=true`, `Priority=100`.
- Rule 2: `RuleType=Organization`, `OrganizationID=5`, `IsVisible=false`, `Priority=50`.

**النتيجة لـ Org 5:**
- Rule 1 + Rule 2 كلاهما يطابق.
- الأعلى أولوية: **Rule 2** (Priority=50).
- **القرار:** `IsVisible = false` → **مخفي**.

#### مثال 2: **إظهار Item في اشتراك مميز** 🟡 مُخطَّط

**السياق:**
- Item: `LOYALTY_SCHEMES`.
- المنظمة: `SubscriptionPlanID = 3` (Premium).

**القواعد:**
- Rule 1: `RuleType=All`, `IsVisible=false`, `Priority=100`.
- Rule 2: `RuleType=Subscription`, `SubscriptionPlanID=3`, `IsVisible=true`, `Priority=50`.

**النتيجة لـ Plan 3:**
- الأعلى أولوية: **Rule 2**.
- **القرار:** `IsVisible = true` → **مرئي**.

#### مثال 3: **إخفاء Item عن وظيفة محددة** 🟡 مُخطَّط

**السياق:**
- Item: `LOYALTY_SETTINGS`.
- الوظيفة: `CustomJobTitleId = 12` (مندوب).

**القواعد:**
- Rule 1: `RuleType=All`, `IsVisible=true`, `Priority=100`.
- Rule 2: `RuleType=JobTitle`, `CustomJobTitleId=12`, `IsVisible=false`, `Priority=50`.

**النتيجة لـ JobTitle 12:**
- الأعلى أولوية: **Rule 2**.
- **القرار:** `IsVisible = false` → **مخفي**.

---

## 🏗️ الجزء الرابع: الطبقات التقنية

### 4.1 المبدأ

> **"منطق واحد في Service، عرض واحد في Shared.UI، ثلاثة أغلفة رقيقة."**

### 4.2 التوزيع حسب الطبقة

| # | الطبقة | المكوّن | الموقع |
|:-:|--------|--------|--------|
| 1 | **Domain** | `MenuVisibilityRule` (Entity) | `RubikCare.Domain/Entities/Navigation/MenuSystem/` |
| 2 | **Application** | `IMenuVisibilityService` | `RubikCare.Application/Interfaces/Navigation/` |
| 3 | **Application** | `IMenuFilteringService` | `RubikCare.Application/Interfaces/Navigation/` |
| 4 | **Application** | `IMenuFilterableItem` | `RubikCare.Application/Interfaces/Navigation/` |
| 5 | **Application** | `MenuFilterContextDto` | `RubikCare.Application/DTOs/Navigation/` |
| 6 | **Application** | `MenuFilteringService` (منطق خالص) | `RubikCare.Application/Services/Navigation/` |
| 7 | **Infrastructure** | `MenuVisibilityService` (DB) | `RubikCare.Infrastructure/Services/Navigation/` |
| 8 | **Web** | `DynamicMenuService` (يستخدم الخدمات) | `Rubikcare.Web/Data/Services/Navigation/` |
| 9 | **Web** | `UserMenuService` (Bridge) | `Rubikcare.Web/Data/Services/` |

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

**المسؤولية:** تطبيق القواعد على قائمة عناصر — **منطق خالص (لا DB).**

### 4.5 `MenuFilterContextDto` — السياق

```csharp
public class MenuFilterContextDto
{
    public int? OrganizationId { get; set; }
    public int? OrganizationTypeId { get; set; }
    public int? SubscriptionPlanId { get; set; }       // 🟡 مُخطَّط
    public int? UserProfileId { get; set; }
    public List<string> RoleIds { get; set; } = new(); // ⚠️ غير مستخدم لإدارة القوائم
    public bool IsPlatformAdmin { get; set; }
    // public List<int> CustomJobTitleIds { get; set; } = new();  // 🟡 مُخطَّط
}
```

### 4.6 تدفق البيانات في `DynamicMenuService`

```
LoadMenuDataFromDatabaseAsync(user)
    │
    ├── GetPersonalMenuAsync(userProfileId)
    │   ├── جلب Items من SystemMenu (USER_BASE)
    │   ├── IMenuVisibilityService.GetActiveRulesForItemsAsync(itemIds)
    │   └── IMenuFilteringService.ApplyRules(items, rules, context)
    │
    ├── GetAdminMenuAsync(user)
    │   ├── جلب Items من SystemMenu (ADMIN_MENU)
    │   ├── جلب Rules
    │   └── ApplyRules
    │
    └── LoadUserOrganizationsAsync(user)
        └── لكل Org:
            └── GetOrganizationMenuFromDatabaseAsync(orgId, orgTypeId, userProfileId)
                ├── جلب MenuAssignments + Items
                ├── جلب Rules
                └── ApplyRules
```

### 4.7 `DynamicMenuService` — تغييرات مهمة

**⚠️ من 24 مايو 2026:**
- **إزالة `IMemoryCache`** — إدارة الكاش مركزية في `UserSessionService`.

**⚠️ الوظائف الرئيسية:**
- `GetUserMenuAsync(ClaimsPrincipal user)` — جلب قوائم المستخدم (بدون كاش).
- `GetMenuDataAsync(ClaimsPrincipal user)` — جلب بيانات القوائم المنفصلة.
- `GetOrganizationMenuFromDatabaseAsync(int organizationId)` — جلب قوائم مؤسسة.
- `ClearCacheAsync(string userId)` — **no-op** — تم نقل مسؤولية الكاش لـ `UserSessionService`.

**⚠️ الاعتماديات:**
- `IDbContextFactoryService` — للوصول لقاعدة البيانات.
- `UserContextService` — لجلب بيانات المستخدم.
- `ILogger<DynamicMenuService>` — للتسجيل.

**⚠️ ملاحظة:** `DynamicMenuService` تُستخدم فقط في تطبيق Blazor Web. تطبيق الموبايل يستخدم `UserSessionService` مباشرة.

---

## 📌 الجزء الخامس: القيم المُعتمَدة (Constants)

### 5.1 `MenuType` — القيم النهائية

| القيمة | الغرض | الثابت (Constants) |
|:---:|:---|:---|
| `BASE` | قوائم عامة لكل المستخدمين | `MenuTypes.Base` |
| `ROLE_BASED` | قوائم مرتبطة بالدور النظامي | `MenuTypes.RoleBased` |
| `ORG_TYPE_BASED` | قوائم مرتبطة بنوع المنظمة | `MenuTypes.OrgTypeBased` |
| `FEATURE_BASED` | قوائم مرتبطة بميزة (اشتراك/خطة) | `MenuTypes.FeatureBased` |

**⚠️ الموقع المُقترَح:** `RubikCare.Domain/Constants/Navigation/MenuTypes.cs`.

**⚠️ الحالة:** 🟡 **مطلوب إنشاؤه** — حالياً القيم كنصوص مباشرة.

### 5.2 `RuleType` — Enum

```csharp
public enum RuleType
{
    All = 0,                // للجميع
    Organization = 1,       // منظمة محددة
    OrganizationType = 2,   // نوع منظمة
    Subscription = 3,       // خطة اشتراك
    JobTitle = 4,           // وظيفة (بدلاً من Role — بعد Migration)
    User = 5                // مستخدم محدد (مستقبلاً)
}
```

**⚠️ الموقع:** `RubikCare.Domain/Enums/Navigation/RuleType.cs`.

**⚠️ الحالة:** ✅ **موجود** (باسم `Role` — سيُعاد تسميته إلى `JobTitle` — Migration مطلوبة).

### 5.3 `MenuCodes` — أكواد القوائم

**⚠️ الموقع المُقترَح:** `RubikCare.Domain/Constants/Navigation/MenuCodes.cs`.

```csharp
public static class MenuCodes
{
    public const string UserBase = "USER_BASE";
    public const string AdminMenu = "ADMIN_MENU";
    public const string PharmaPsp = "PHARMA_PSP";
    public const string ClinicMenu = "CLINIC_MENU";
    public const string PharmacyMenu = "PHARMACY_MENU";
    public const string HospitalMenu = "HOSPITAL_MENU";
    public const string Loyalty = "LOYALTY";
}
```

**⚠️ الحالة:** 🟡 **مطلوب إنشاؤه**.

### 5.4 `ItemCodes` — أكواد عناصر القوائم

**⚠️ الموقع المُقترَح:** `RubikCare.Domain/Constants/Navigation/ItemCodes.cs`.

```csharp
public static class ItemCodes
{
    // LOYALTY
    public const string LoyaltyHub = "LOYALTY_HUB";
    public const string LoyaltySchemes = "LOYALTY_SCHEMES";
    public const string LoyaltyBalances = "LOYALTY_BALANCES";
    public const string LoyaltyRedeem = "LOYALTY_REDEEM";
    public const string LoyaltySettings = "LOYALTY_SETTINGS";

    // Admin
    public const string AdminDashboard = "ADMIN_DASHBOARD";
    // ...
}
```

**⚠️ الحالة:** 🟡 **مطلوب إنشاؤه**.

---

## ✅ الجزء السادس: CHECKLIST — إضافة قائمة جديدة

### الخطوة 1: إضافة القائمة الرئيسية

- [ ] هل القائمة جديدة كلياً؟ أضف سجلاً في `SystemMenus` مع `MenuCode` فريد.
- [ ] هل تحددت `MenuType` المناسب؟
    - [ ] `BASE` — للجميع.
    - [ ] `ROLE_BASED` — لدور نظامي.
    - [ ] `ORG_TYPE_BASED` — لنوع منظمة.
    - [ ] `FEATURE_BASED` — لميزة (اشتراك/خطة).
- [ ] هل تحدد `SortOrder` المناسب؟

### الخطوة 2: إضافة عناصر القائمة

- [ ] هل أضفت كل عنصر في `MenuItems` مع `MenuID` الصحيح؟
- [ ] هل `ItemCode` فريد لكل عنصر؟
- [ ] هل رتبت العناصر باستخدام `SortOrder`؟
- [ ] هل اخترت أيقونة مناسبة من Bootstrap Icons؟
- [ ] هل `IsVisible = 1` للعناصر التي يجب أن تظهر؟
- [ ] هل `RequiresPermission = 1` + `PermissionCode` للعناصر المحمية؟

### الخطوة 3: تخصيص الظهور (MenuAssignment)

- [ ] لمن تظهر هذه القائمة؟
    - [ ] للجميع؟ → `OrganizationTypeID = NULL` + `OrganizationID = NULL`.
    - [ ] لنوع منظمة؟ → `OrganizationTypeID` محدد.
    - [ ] لمنظمة محددة؟ → `OrganizationID` محدد.
    - [ ] لنوع + منظمة؟ → كليهما.
- [ ] هل للقائمة صلاحية زمنية؟ اضبط `ValidTo`.
- [ ] هل `IsActive = 1`؟

### الخطوة 4: قواعد الإظهار (MenuVisibilityRules) — إن لزم

- [ ] هل نحتاج Rule لإخفاء عنصر عن منظمة محددة؟ → `RuleType = Organization`.
- [ ] هل نحتاج Rule لإخفاء عنصر عن وظيفة؟ → `RuleType = JobTitle`.
- [ ] هل نحتاج Rule لإظهار عنصر فقط في اشتراك معين؟ → `RuleType = Subscription`.
- [ ] هل `Priority` صحيح؟ **الأصغر = الأعلى**.

### الخطوة 5: الاختبار

- [ ] اختبر مع مستخدمين مختلفين (عادي، Admin، عضو في المؤسسة).
- [ ] تأكد من أن القائمة تظهر فقط لمن يفترض أن تظهر له.
- [ ] تأكد من أن الروابط تعمل بشكل صحيح.
- [ ] اختبر تبديل المنظمات — القوائم تُحدَّث.
- [ ] اختبر Cache — `Logout` + `Login` — القائمة تُحدَّث.

---

## 🔧 الجزء السابع: حل المشاكل الشائعة

### 7.1 القائمة لا تظهر

| # | الفحص | الحل |
|:-:|:---|:---|
| 1 | `SystemMenus.IsActive = 1`؟ | تحقق من DB |
| 2 | `MenuAssignments.IsActive = 1`؟ | تحقق من DB |
| 3 | `MenuAssignments.OrganizationTypeID` صحيح؟ | يجب أن يُطابق نوع منظمة المستخدم |
| 4 | `MenuAssignments.OrganizationID` صحيح؟ | يجب أن يكون `NULL` للقوائم العامة |
| 5 | `MenuItems` موجودة؟ | تحقق من `MenuID` |
| 6 | `MenuItems.IsVisible = 1`؟ | يجب أن يكون `1` للظهور |
| 7 | `MenuVisibilityRules` لا تحجب؟ | تحقق من `IsVisible` + `Priority` |
| 8 | Cache قديم؟ | Logout + Login |

### 7.2 عنصر لا يظهر داخل القائمة

| # | الفحص | الحل |
|:-:|:---|:---|
| 1 | `MenuItems.IsVisible = 1`؟ | تحقق من DB |
| 2 | `MenuVisibilityRules` لا تحجب؟ | تحقق من `IsVisible` + `Priority` |
| 3 | `RuleType = All` موجود؟ | يجب أن يكون `IsVisible = 1` كقاعدة افتراضية |
| 4 | `RequiresPermission = 1`؟ | تحقق من صلاحيات المستخدم |

### 7.3 `CS0104` Ambiguous reference

**السبب:** تعارض في `@using` بين `Application.DTOs.Navigation.MenuItemDto` و `Shared.UI.Components.PSP.Points.MenuItemDto`.

**الحل:**
```razor
@* احذف @using RubikCare.Shared.UI.Components.PSP.Points من _Imports.razor *@

@* وأضف @using محلياً في الملفات التي تحتاجها *@
@using RubikCare.Application.DTOs.Navigation
```

### 7.4 `CS0246` Type not found

**السبب:** `@using` مفقود.

**الحل:** أضف `@using` المناسب في الملف.

### 7.5 Cache قديم — القائمة لا تُحدَّث

**السبب:** `UserSessionService` cache.

**الحل:**
```csharp
await _menuService.ClearCacheAsync(userId);
```

**⚠️ مكان الاستدعاء:** `AdminMenuManager.razor` — بعد كل حفظ. أو `AuthController.Logout`.

### 7.6 قاعدة `Priority` مقلوبة

**المشكلة:** القاعدة الأقل أولوية تفوز.

**السبب:** الترتيب معكوس في الكود.

**الحل:**
```csharp
.OrderBy(r => r.Priority)   // ✅ الأصغر أولاً
```

### 7.7 `SystemMenuMenuID` Shadow Property

**المشكلة:** عمود غريب في `MenuAssignments`.

**السبب:** Navigation Property غير مهيأة بشكل صحيح.

**الحل:** إصلاح `MenuAssignment.cs` — إضافة `[ForeignKey("MenuID")]` — ثم Migration.

**⚠️ الحالة:** **دين تقني** — يحتاج Migration مستقبلي.
```

---
[#101] 📄 `04-dynamic-menus.md` — الإصدار 3.0 (الجزء الثالث والأخير)

---

```markdown
## 🔧 الجزء الثامن: EF Core Migration — ملاحظات حرجة

> **⚠️ درس مستخلص من جلسة 13 أكتوبر 2026 — يجب قراءته قبل أي Migration.**

### 8.1 المبدأ الأساسي

> **"Migration المُولَّد صحيح — فقط عندما يكون Snapshot متزامناً مع DB."**

**⚠️ عند `Add-Migration`:**

**EF Core يقارن:**
1. **Snapshot** — الحالة السابقة (cache).
2. **Model Current** — الكود الحالي.

**⚠️ EF Core **لا يفحص DB** — بل يثق أن Snapshot يعكس DB.**

**⚠️ إذا Snapshot ≠ DB:**
- Migration المُولَّد يحاول فرض حالة **خاطئة**.
- **يفشل عند `Update-Database`.**

### 8.2 القاعدة الصارمة

> **"قبل أي `Add-Migration` — تأكد أن Snapshot = DB."**

**⚠️ التحقق:**

```sql
-- 1. البنية
SELECT COLUMN_NAME, IS_NULLABLE, DATA_TYPE
FROM INFORMATION_SCHEMA.COLUMNS
WHERE TABLE_NAME = 'TableName'
ORDER BY ORDINAL_POSITION;

-- 2. الـ Indexes
SELECT i.name AS IndexName, STRING_AGG(c.name, ', ') AS Columns
FROM sys.indexes i
INNER JOIN sys.index_columns ic ON i.object_id = ic.object_id AND i.index_id = ic.index_id
INNER JOIN sys.columns c ON ic.object_id = c.object_id AND ic.column_id = c.column_id
WHERE i.object_id = OBJECT_ID('TableName')
GROUP BY i.name
ORDER BY i.name;

-- 3. الـ FKs
SELECT fk.name, COL_NAME(fkc.parent_object_id, fkc.parent_column_id) AS ColumnName
FROM sys.foreign_keys fk
INNER JOIN sys.foreign_key_columns fkc ON fk.object_id = fkc.constraint_object_id
WHERE fk.parent_object_id = OBJECT_ID('TableName');
```

**⚠️ قارن بالـ Snapshot:**
```
C:\RC\Rubikcare.Full.Migration\RubikCare.Infrastructure\Migrations\BusinessDbContextModelSnapshot.cs
```

### 8.3 عندما Snapshot ≠ DB

**⚠️ الأعراض:**
- `Add-Migration` يُولّد Migration يحتوي `DropIndex` لـ Index غير موجود.
- `Update-Database` يفشل بـ `Cannot drop the index ... because it does not exist`.

**⚠️ الحل — خطوات:**

#### 🔹 الخطوة 1: **تحديد الفرق**

**⚠️ قارن:**
- **Snapshot:** ما يعرفه EF Core.
- **DB:** الواقع الفعلي.
- **Model:** الكود الحالي.

#### 🔹 الخطوة 2: **تعديل Snapshot ليعكس DB** — لمرة واحدة

**⚠️ هذا **تعديل يدوي** — مرفوض كقاعدة عامة.**

**⚠️ لكن — عند الضرورة القصوى — مسموح — وفق شروط:**
- **لمرة واحدة فقط** — لإصلاح تعارض تاريخي.
- **لا يُعدّل Migration files.**
- **يُصحح Snapshot ليعكس DB — ثم EF Core يُولّد Snapshot جديداً من Model — تلقائياً.**

#### 🔹 الخطوة 3: **التحقق — `Add-Migration TestSync`**

**⚠️ بعد تعديل Snapshot:**

```powershell
cd C:\RC\Rubikcare.Full.Migration
$ErrorActionPreference = 'Continue'
dotnet ef migrations add TestSync --project RubikCare.Infrastructure --startup-project Rubikcare.Web
```

**⚠️ إذا Migration فارغ** → Snapshot = Model → **متزامن** ✅.
**⚠️ إذا Migration غير فارغ** → Snapshot ≠ Model → **يحتاج تعديلاً**.

#### 🔹 الخطوة 4: **حذف Migration الاختباري**

```powershell
dotnet ef migrations remove --project RubikCare.Infrastructure --startup-project Rubikcare.Web
```

#### 🔹 الخطوة 5: **إنشاء Migration الحقيقي**

```powershell
dotnet ef migrations add MigrationName --project RubikCare.Infrastructure --startup-project Rubikcare.Web
```

**⚠️ راجع Migration المُولَّد — تأكد أنه يعكس تغييرات Model فقط.**

#### 🔹 الخطوة 6: **`Update-Database`**

```powershell
dotnet ef database update --project RubikCare.Infrastructure --startup-project Rubikcare.Web
```

### 8.4 الممنوعات المطلقة

| # | الممنوع | السبب |
|:-:|:---|:---|
| 1 | **تعديل Migration files يدوياً** | تُلغى عند أول `Add-Migration` آخر — تفقد التاريخ |
| 2 | **تعديل `ModelSnapshot.cs` كروتين** | يُلغى عند أول `Add-Migration` — يفقد الاتساق |
| 3 | **SQL مباشر لتعديل DB** — كبديل عن Migration | يخلق فجوة بين History و DB |
| 4 | **حذف Migration file من DB دون `Remove-Migration`** | يكسر `__EFMigrationsHistory` |
| 5 | **تعديل DB دون Migration** — كروتين | يُفسد التوازن |

### 8.5 الاستثناءات المسموحة

| # | الاستثناء | الشرط |
|:-:|:---|:---|
| 1 | **SQL مباشر للبيانات** (Data Migration) | عند نقل بيانات — مثل `UPDATE SourceEntityID = ProgramID` — قبل `Add-Migration` |
| 2 | **تعديل Snapshot يدوياً** | لمرة واحدة — لإصلاح تعارض تاريخي — مع تحقق فوري |
| 3 | **`Remove-Migration`** | لحذف Migration **غير مُطبَّق** |
| 4 | **Backup DB** | قبل أي Migration حساس |

### 8.6 حالة جلسة 13 أكتوبر 2026

**⚠️ ما حدث:**

1. **تعديل `OnModelCreating`** — تحويل `ProgramID` → `SourceEntityID`.
2. **`Add-Migration UseSourceEntityIdInPointSchemes`** — Migration يحاول `DropIndex("IX_..._ProgramID")`.
3. **`Update-Database`** — يفشل — لأن DB يحتوي `IX_..._ProgramID_General` وليس `IX_..._ProgramID`.
4. **السبب:** Snapshot يعكس حالة **قبل** Migration سابق — لا يعكس DB.
5. **الحل المُطبَّق:**
   - **SQL مباشر:** `UPDATE PSPProgramPointSchemes SET SourceEntityID = ProgramID`.
   - **تعديل Snapshot يدوياً** — ليعكس DB (بأسماء Indexes الفعلية).
   - **`Add-Migration UseSourceEntityIdInPointSchemes`** — جديد — يعكس الفرق الحقيقي.
   - **`Update-Database`** — نجح.
6. **النتيجة:** Model = Snapshot = DB — متزامن.

### 8.7 Checklist قبل أي Migration

- [ ] Backup DB كامل.
- [ ] مراجعة Snapshot — هل يعكس DB؟
- [ ] مراجعة `OnModelCreating` — هل يُطابق Entity؟
- [ ] `Add-Migration` — **مراجعة Migration المُولَّد قبل `Update-Database`.**
- [ ] إذا Migration يحتوي `DropIndex` لـ Index غير موجود → **توقف**.
- [ ] تحقق من `__EFMigrationsHistory` — كل Migrations مُطبَّقة؟
- [ ] اختبار على بيئة تجريبية — إن أمكن.

---

## 🔗 الجزء التاسع: روابط ذات صلة

### 📚 وثائق أساسية

- [00 - الهيكل المعماري](./00-architecture-overview.md)
- [01 - Program.cs والتسجيلات الأساسية](./01-program-cs-foundation.md)
- [02 - نظام الهوية والمصادقة](./02-identity-system.md)
- [05 - إنشاء الصفحات والمكونات](./05-page-creation-checklist.md)
- [08 - Strategic Roadmap](./08-Strategic-Roadmap.md)
- [14 - نظام الكاش الموحد](./14-caching-system.md)
- [17 - استقرار اتجاه القائمة الجانبية واللغة](./17-sidebar-direction-stability.md)
- [26 - نظام النقاط وروبيكا](./26-rubika-points-system.md)
- [27 - نظام القوائم والصلاحيات الشامل](./27-multi-layer-menu-control.md) — **⚠️ سيُحذف — دُمج في `04`**

### 📚 وثائق مرتبطة بالمنصات

- [10 - دليل تطوير MAUI](./10-maui-development-guide.md)
- [11 - دليل BlazorWebView](./11-blazor-webview-guide.md)

### 📚 وثائق مرتبطة بالتنفيذ

- [15 - نظام الترجمة](./15-translation-system.md)
- [24 - جرد هيكلة المشاريع](./24-project-structure-inventory.md)
- [27 - الديون التقنية](./27-technical-debt.md)

---

## 📝 الجزء العاشر: سجل التغييرات

| الإصدار | التاريخ | التغييرات |
|---------|---------|-----------|
| **1.0** | **18 يوليو 2026** | الإصدار الأولي — `04-dynamic-menus.md` |
| **2.0** | **24 مايو 2026** | إزالة `IMemoryCache` — كاش مركزي في `UserSessionService` |
| **3.0 (27 v1)** | **6 أكتوبر 2026** | `27-multi-layer-menu-control.md` — الإصدار الأولي (4 طبقات + MenuItemVisibility + Features/Plans) |
| **3.0 (27 v3)** | **13 أكتوبر 2026** | `27` — النسخة الموحّدة (MenuVisibilityRules + RuleType Enum + IMenuFilteringService) |
| **3.0 (Unified)** | **13 أكتوبر 2026** | **`04-dynamic-menus.md` v3.0** — الوثيقة الموحّدة (دمج `04` v2 + `27` v1 + `27` v3) + تحديثات جلسة 13 أكتوبر + ملاحظات EF Core الحرجة |

---

## 🎯 ملاحظة أخيرة

### 📌 ما تبقى (Roadmap)

| # | المهمة | الحالة | الأولوية |
|:-:|:---|:---:|:---:|
| 1 | **إضافة `CustomJobTitleId` إلى `MenuVisibilityRules`** | 🟡 مُخطَّطة | 🔴 عالية |
| 2 | **إزالة `RoleId` من `MenuVisibilityRules`** | 🟡 مُخطَّطة | 🟠 متوسطة |
| 3 | **إزالة `SystemMenuMenuID` (Shadow Property)** | 🟡 مُخطَّطة | 🟠 متوسطة |
| 4 | **إنشاء `Features` + `Plans` + `PlanFeatures` + `OrganizationSubscriptions`** | 🟡 مُخطَّطة | 🟠 متوسطة |
| 5 | **إنشاء `FeatureMenuItems`** | 🟡 مُخطَّطة | 🟠 متوسطة |
| 6 | **إنشاء `MenuTypes.cs` + `MenuCodes.cs` + `ItemCodes.cs`** | 🟡 مُخطَّطة | 🟢 منخفضة |
| 7 | **UI لإدارة `MenuVisibilityRules`** (Super Admin) | 🟡 مُخطَّطة | 🟠 متوسطة |
| 8 | **تفعيل `Priority` في `MenuFilteringService`** | ❓ يحتاج تحقق | 🔴 عالية |
| 9 | **تعميم `DynamicMenuComponent` على PWA + MAUI** | 🟡 مُخطَّطة | 🟠 متوسطة |

### 📌 قاعدة ذهبية

> **"أي تعديل على `SystemMenus`, `MenuItems`, `MenuAssignments`, أو `MenuVisibilityRules` — يجب أن يتبعه `ClearCacheAsync(userId)`."**

### 📌 ملاحظة عن EF Core

> **"قبل أي `Add-Migration` — تأكد أن Snapshot = DB. Migration المُولَّد صحيح فقط عندما يكون Snapshot متزامناً."**

---

**© 2026 RubikCare — للاستخدام الداخلي**

**نهاية الوثيقة ✅**
```
