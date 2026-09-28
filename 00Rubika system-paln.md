# خطة عمل نظام RUbikca
## 🔍 الفهم الجديد

### الفكرة:

```
❌ الاعتقاد السابق:
   ServiceProducts = خدمات ثابتة بتسعير ثابت
   ServicePricingOptions = تسعير مسبق
   ServiceTargets = أهداف محددة

✅ الفهم الصحيح:
   ServiceProducts = وصف مرن لخدمة (يمكن أن تكون أي شيء)
   ServicePricingOptions = وصف مرن للتسعير (يمكن أن يكون أي شيء)
   ServiceTargets = وصف مرن للهدف (يمكن أن يكون أي شيء)
```

### لماذا هذا مهم؟

**طبيعة PSP:**
- نظام **تعاقدات وتفاوض** (وليس تسعير ثابت)
- البرنامج الواحد قد يحتوي **عدة أنواع** من الدعم
- كل برنامج **فريد** (يُتفاوض عليه مع شركة الأدوية)

**إذن:**

| المفهوم | التطبيق على PSP |
|---------|-----------------|
| `ServiceProduct` | وصف البرنامج (اسم، وصف، تصنيف) |
| `ServicePricingOption` | شروط التعاقد (ميزانية، عمولة، نقاط) |
| `ServiceTarget` | الأطراف المستهدفة (أطباء، مرضى، صيدليات) |
| `ServiceCategory` | تصنيف البرنامج (ACCESS, ADHERENCE, ...) |

**🎯 كل هذه حقول وصفية — يمكن أن تحوي أي شيء!**

---

## 🎯 إعادة الهيكلة المقترحة

### 1. `ServiceProducts` — وصف البرنامج

**الاستخدام الجديد:**

```sql
-- كل PSPProgram يمكن أن يكون ServiceProduct
INSERT INTO ServiceProducts (
    SKU_Code, ServiceNameAr, ServiceNameEn, ServiceDescription,
    CategoryID, ServiceNature, IsActive, IsRewardService, 
    IsIncludedInFreePlan, DisplayOrder, CreatedDate
)
VALUES 
    -- برنامج دعم (وصف فقط)
    (N'PSP-PROGRAM-001', N'برنامج دعم حالات الحمل', N'Pregnancy Support Program',
     N'برنامج دعم للحالات الحرجة', 2, N'PSP_PROGRAM', 1, 0, 1, 100, GETDATE());
```

**🎯 `ServiceNature` = `PSP_PROGRAM`** — نوع جديد.

### 2. `ServicePricingOptions` — شروط التعاقد

**الاستخدام الجديد:**

```sql
-- شروط التعاقد لكل برنامج
INSERT INTO ServicePricingOptions (
    ServiceProductID, PlanNameAr, PlanNameEn, PricingType, TargetUserType,
    PriceInRubika, CommissionRate, RewardAmount, TriggerCondition,
    BillingCycle, IsActive, CreatedDate
)
VALUES 
    -- للطبيب
    (1, N'نقاط الاشتراك', N'Enrollment Points', N'REWARD', N'DOCTOR',
     50, 0, 50, N'PER_PATIENT_ENROLLED', N'PER_USE', 1, GETDATE()),
    
    -- للمريض
    (1, N'نقاط الصرف', N'Dispensation Points', N'REWARD', N'PATIENT',
     10, 0, 10, N'PER_DISPENSATION', N'PER_USE', 1, GETDATE());
```

**🎯 `PricingType` = `REWARD`** — نوع جديد (نقاط).

### 3. `ServiceTargets` — الأطراف المستهدفة

**الاستخدام الجديد:**

```sql
-- ربط البرنامج بالأطراف
INSERT INTO ServiceTargets (ServiceProductID, TargetType, TargetID, IsActive, CreatedDate)
VALUES 
    (1, N'DOCTOR', 0, 1, GETDATE()),    -- كل الأطباء
    (1, N'PATIENT', 0, 1, GETDATE()),   -- كل المرضى
    (1, N'PHARMACY', 0, 1, GETDATE());  -- كل الصيدليات
```

**🎯 `TargetID = 0`** = الكل.

### 4. `ServiceCategories` — تصنيفات البرامج

**موجود بالفعل** (25 تصنيف).

### 5. `SubscriptionPlans` — مؤجل

**لا نحتاجه الآن.**

### 6. `PaymentMethods` — مؤجل

**لا نحتاجه الآن.**

---

## 🎯 الربط بين `PSPPrograms` و `ServiceProducts`

### الطريقة الصحيحة:

**1. `PSPPrograms.ServiceProductID`** — موجود بالفعل ✅

**2. إعادة هيكلة:**

```
لكل PSPProgram:
├── ServiceProductID → ServiceProducts (وصف البرنامج)
├── ServicePricingOptions (شروط التعاقد: نقاط، عمولة)
└── ServiceTargets (الأطراف المستهدفة)
```

### مثال عملي:

**البرنامج 1: `pregnant case support`**

```sql
-- 1. ServiceProduct
INSERT INTO ServiceProducts (SKU_Code, ServiceNameAr, ServiceNameEn, CategoryID, ServiceNature, IsActive, CreatedDate)
VALUES (N'PSP-PREGNANT-001', N'برنامج دعم حالات الحمل', N'Pregnancy Support Program', 2, N'PSP_PROGRAM', 1, GETDATE());

-- 2. ربط PSPProgram
UPDATE PSPPrograms SET ServiceProductID = (SELECT SCOPE_IDENTITY()) WHERE ProgramID = 1;

-- 3. ServicePricingOptions (شروط التعاقد)
INSERT INTO ServicePricingOptions (ServiceProductID, PlanNameAr, PricingType, TargetUserType, RewardAmount, TriggerCondition, IsActive, CreatedDate)
VALUES 
    (@ServiceProductID, N'نقاط اشتراك الطبيب', N'REWARD', N'DOCTOR', 50, N'PER_PATIENT_ENROLLED', 1, GETDATE()),
    (@ServiceProductID, N'نقاط صرف الطبيب', N'REWARD', N'DOCTOR', 10, N'PER_DISPENSATION', 1, GETDATE()),
    (@ServiceProductID, N'نقاط صرف المريض', N'REWARD', N'PATIENT', 10, N'PER_DISPENSATION', 1, GETDATE());

-- 4. ServiceTargets
INSERT INTO ServiceTargets (ServiceProductID, TargetType, TargetID, IsActive, CreatedDate)
VALUES 
    (@ServiceProductID, N'DOCTOR', 0, 1, GETDATE()),
    (@ServiceProductID, N'PATIENT', 0, 1, GETDATE());
```

---

## 🎯 ما نحتاجه الآن

### 1. إعادة هيكلة `ServiceProducts` — إضافة `ServiceNature`

**القيم الجديدة:**
- `PSP_PROGRAM` — برنامج دعم
- `PLATFORM_SERVICE` — خدمة منصة (مؤجل)
- `REWARD` — مكافأة (مؤجل)
- `SUBSCRIPTION` — اشتراك (مؤجل)
- `COMMISSION` — عمولة (مؤجل)

**⚠️ لكن `ServiceNature` موجود بالفعل كـ `nvarchar(MAX)`.**

**إذن:** لا نحتاج تعديل Schema — فقط نستخدم قيم جديدة.

### 2. إعادة هيكلة `ServicePricingOptions` — إضافة `PricingType`

**القيم الجديدة:**
- `REWARD` — نقاط
- `FIXED` — سعر ثابت
- `PERCENTAGE` — نسبة
- `TIERED` — متدرج

**⚠️ لكن `PricingType` موجود بالفعل كـ `nvarchar(50)`.**

**إذن:** لا نحتاج تعديل Schema.

### 3. إعادة هيكلة `ServiceTargets` — إضافة `TargetType`

**القيم الجديدة:**
- `DOCTOR` — طبيب
- `PATIENT` — مريض
- `PHARMACY` — صيدلية
- `ORGANIZATION` — منظمة
- `ALL` — الكل

**⚠️ لكن `TargetType` موجود بالفعل كـ `nvarchar(20)`.**

**إذن:** لا نحتاج تعديل Schema.

---

## 🎯 الخلاصة

### ✅ لا نحتاج تعديل Schema

**كل الحقول موجودة — فقط نحتاج:**
1. **استخدام قيم جديدة** في الحقول الموجودة
2. **إعادة هيكلة البيانات** الموجودة
3. **إكمال الربط** بين `PSPPrograms` و `ServiceProducts`

### ✅ لا نحتاج جداول جديدة

**كل الجداول موجودة:**
- `ServiceProducts` ✅
- `ServicePricingOptions` ✅
- `ServiceTargets` ✅
- `ServiceCategories` ✅

### ✅ لا نحتاج إعادة هيكلة API

**كل الـ API موجودة** (CRUD).

**⚠️ لكن:** قد تحتاج **إعادة هيكلة الصفحات** لتتوافق مع المعمارية.

---

## 🎯 خطة العمل المقترحة

### المرحلة 1: إعادة هيكلة البيانات (SQL)

| # | المهمة | الوصف |
|---|--------|-------|
| 1 | **إضافة `ServiceNature = 'PSP_PROGRAM'`** | لكل `ServiceProducts` المرتبط بـ PSP |
| 2 | **ربط `PSPPrograms` بـ `ServiceProducts`** | للبرامج الـ 11 |
| 3 | **إضافة `ServicePricingOptions`** | شروط التعاقد لكل برنامج |
| 4 | **إضافة `ServiceTargets`** | الأطراف المستهدفة |
| 5 | **إضافة محفظة SYSTEM** | في `RubikaWallet` |
| 6 | **إضافة `RubikaValueSettings`** | القيم الافتراضية |

### المرحلة 2: إعادة هيكلة API (إذا لزم)

| # | المهمة | الوصف |
|---|--------|-------|
| 1 | **فحص API الموجود** | `ServiceProductsController`, إلخ. |
| 2 | **إعادة هيكلة إذا لزم** | لتتوافق مع Clean Architecture |

### المرحلة 3: Rubika Service (Backend)

| # | المهمة | الوصف |
|---|--------|-------|
| 1 | `IRubikaService` | Interface |
| 2 | `RubikaService` | Implementation |
| 3 | `EarnRubikaHandler` | للكسب |
| 4 | Triggers | في الأحداث |

### المرحلة 4: UI

| # | المهمة | الوصف |
|---|--------|-------|
| 1 | `RubikaBalanceCard.razor` | بطاقة الرصيد |
| 2 | `RubikaTransactionsPage.razor` | سجل المعاملات |
| 3 | `DoctorPointsDashboard.razor` | لوحة نقاط الطبيب |
| 4 | `PatientPointsDashboard.razor` | لوحة نقاط المريض |

---

**سؤال ممتاز — وهو جوهر التصميم. دعني أحلل الوضع الحالي وأجيب بدقة.**

---

## 🔍 الإجابة المباشرة

**نعم، البنية الحالية تستطيع دعم رؤيتك — لكنها تحتاج تعديلات.**

### ما هو موجود:

| # | الجدول | الوصف | يدعم رؤيتك؟ |
|---|--------|-------|-------------|
| 1 | `RubikaWallet` | محفظة لكل مستخدم | ✅ نعم |
| 2 | `RubikaTransactions` | سجل المعاملات | ✅ نعم |
| 3 | `RubikaValueSettings` | إعدادات القيم | ⚠️ **جزئياً** |
| 4 | `ServiceProducts` | خدمات PSP | ⚠️ **يحتاج تعديل** |
| 5 | `ServicePricingOptions` | خيارات التسعير | ⚠️ **يحتاج تعديل** |
| 6 | `ServiceTargets` | أهداف الخدمات | ✅ نعم |
| 7 | `ServiceCategories` | تصنيفات | ✅ نعم |

### ما ينقص:

| # | ما ينقص | الوصف |
|---|---------|-------|
| 1 | **جدول `PSPProgramPoints`** | قواعد النقاط لكل برنامج |
| 2 | **جدول `PSPProgramPointActivities`** | الأنشطة القابلة للنقاط |
| 3 | **`PointsType` في `RubikaWallet`** | لدعم أنواع متعددة من النقاط |
| 4 | **`ProgramID` في `RubikaTransaction`** | لربط المعاملة بالبرنامج |
| 5 | **`PointsCategory` في `RubikaWallet`** | لتمييز النقاط الخاصة بكل برنامج |

---

## 🎯 تحليل عميق: نوعان من النقاط

### 1. نقاط المنصة (Platform Points) — موجودة جزئياً

**تعريف:** نقاط موحدة لكل المنصة، تُمنح لكل المستخدمين.

**أمثلة:**
- استكمال البروفايل → 10 Rubika
- دعوة صديق → 5 Rubika
- تقييم خدمة → 2 Rubika

**المصدر:** `RubikaValueSettings` (القيم الموحدة).

**الحالة:** ✅ مدعومة.

---

### 2. نقاط البرنامج (Program Points) — **جديدة**

**تعريف:** نقاط تُنشئها كل شركة أدوية لكل برنامج دعم على حدة.

**المميزات:**
- **قيمة مادية محددة من الشركة** (مثلاً: 1 ProgramPoint = 10 جنيه)
- **أنشطة محددة من الشركة** (مثلاً: تسجيل مريض = 50 نقطة، صرف = 10 نقاط)
- **قابلة للاستبدال** بمكافآت (خصومات، هدايا، إلخ)
- **مختلفة من برنامج لآخر** (حتى لو كانت نفس الشركة)

**الحالة:** ❌ **غير مدعومة حالياً.**

---

## 🎯 ما نحتاجه (تصميم البنية)

### 1. جدول `PSPProgramPoints` — تعريف النقاط لكل برنامج

```sql
CREATE TABLE PSPProgramPoints (
    ProgramPointID INT PRIMARY KEY IDENTITY(1,1),
    ProgramID INT NOT NULL FOREIGN KEY REFERENCES PSPPrograms(ProgramID),
    PharmaCompanyID INT NOT NULL FOREIGN KEY REFERENCES Organizations(OrganizationID),
    
    -- ⭐ تعريف النقاط
    PointsName NVARCHAR(200) NOT NULL,           -- "نقاط برنامج العقم"
    PointsNameEn NVARCHAR(200) NULL,
    PointsCode NVARCHAR(50) NOT NULL,            -- "INFERTILITY_POINTS"
    Description NVARCHAR(1000) NULL,
    
    -- ⭐ القيمة المادية
    MonetaryValue DECIMAL(18,2) NOT NULL,        -- قيمة النقطة بالجنيه
    Currency NVARCHAR(10) DEFAULT 'EGP',
    
    -- ⭐ فترة السريان
    StartDate DATETIME2 NULL,
    EndDate DATETIME2 NULL,
    
    -- ⭐ الحالة
    IsActive BIT NOT NULL DEFAULT 1,
    CreatedDate DATETIME2 NOT NULL DEFAULT GETDATE(),
    CreatedBy INT NULL,
    
    INDEX IX_PSPProgramPoints_Program (ProgramID, IsActive)
);
```

---

### 2. جدول `PSPProgramPointRules` — قواعد النقاط

```sql
CREATE TABLE PSPProgramPointRules (
    RuleID INT PRIMARY KEY IDENTITY(1,1),
    ProgramPointID INT NOT NULL FOREIGN KEY REFERENCES PSPProgramPoints(ProgramPointID),
    
    -- ⭐ نوع النشاط
    ActivityType NVARCHAR(50) NOT NULL,           -- PATIENT_ENROLLED, DISPENSATION, ADHERENCE_HIGH, ...
    ActivityNameAr NVARCHAR(200) NOT NULL,
    ActivityNameEn NVARCHAR(200) NULL,
    
    -- ⭐ الفئة المستهدفة
    TargetRole NVARCHAR(20) NOT NULL,             -- DOCTOR, PHARMACIST, PATIENT
    
    -- ⭐ قيمة النقاط
    Points INT NOT NULL,                          -- عدد النقاط
    MaxPointsPerMonth INT NULL,                   -- حد أقصى شهري (اختياري)
    MaxPointsPerPatient INT NULL,                 -- حد أقصى لكل مريض (اختياري)
    
    -- ⭐ الشروط
    Conditions NVARCHAR(MAX) NULL,                -- JSON للشروط الإضافية
    
    -- ⭐ الحالة
    IsActive BIT NOT NULL DEFAULT 1,
    CreatedDate DATETIME2 NOT NULL DEFAULT GETDATE(),
    
    INDEX IX_PSPProgramPointRules_Program (ProgramPointID, TargetRole, IsActive)
);
```

**قيم `ActivityType` المقترحة:**

| النشاط | TargetRole | الوصف |
|--------|------------|-------|
| `PATIENT_ENROLLED` | DOCTOR | تسجيل مريض جديد |
| `DISPENSATION_COMPLETED` | DOCTOR | صرف دواء لمريضه |
| `DISPENSATION_COMPLETED` | PATIENT | صرف دواء |
| `ADHERENCE_HIGH` | DOCTOR | التزام مرضاه > 80% |
| `ADHERENCE_WEEK` | PATIENT | أسبوع كامل التزام |
| `ADHERENCE_MONTH` | PATIENT | شهر كامل التزام |
| `PHARMACY_DISPENSE_FAST` | PHARMACIST | صرف سريع (< 24 ساعة) |
| `PHARMACY_DISPENSE_COUNT` | PHARMACIST | عدد صرفات |

---

### 3. تعديل `RubikaWallet` — دعم أنواع النقاط

**الحالي:**
```sql
PointsType NVARCHAR(50) NOT NULL              -- SERVICE_POINTS
```

**المطلوب:**
```sql
PointsType NVARCHAR(50) NOT NULL              -- SERVICE_POINTS, PLATFORM_POINTS
ProgramPointID INT NULL                       -- ⭐ جديد: النقاط الخاصة بالبرنامج
    FOREIGN KEY REFERENCES PSPProgramPoints(ProgramPointID)
```

**النتيجة:** كل مستخدم قد يكون لديه **محافظ متعددة:**
- محفظة `SERVICE_POINTS` (نقاط المنصة)
- محفظة `PROGRAM_POINTS` لكل برنامج

---

### 4. تعديل `RubikaTransaction` — ربط بالبرنامج

**الحالي:**
```sql
ServiceProductID INT NULL
OrganizationID INT NULL
```

**المطلوب:**
```sql
ProgramID INT NULL                            -- ⭐ جديد: البرنامج المرتبط
ProgramPointID INT NULL                       -- ⭐ جديد: نوع النقاط
ActivityType NVARCHAR(50) NULL                -- ⭐ جديد: نوع النشاط
ReferenceID INT NULL                          -- ⭐ جديد: معرف الكيان المرجعي (PatientID, DispensationID)
```

---

## 🎯 الرؤية الكاملة للنظام

### المخطط:

```
┌─────────────────────────────────────────────────────────────┐
│                    Rubika System                              │
│                                                               │
│  ┌─────────────────────────────────────────────────────┐    │
│  │  RubikaWallet (محفظة لكل مستخدم × كل برنامج)        │    │
│  │  ├── SERVICE_POINTS (نقاط المنصة)                  │    │
│  │  ├── PROGRAM_POINTS_1 (نقاط البرنامج 1)            │    │
│  │  ├── PROGRAM_POINTS_2 (نقاط البرنامج 2)            │    │
│  │  └── ...                                            │    │
│  └─────────────────────────────────────────────────────┘    │
│                                                               │
│  ┌─────────────────────────────────────────────────────┐    │
│  │  RubikaTransaction (سجل المعاملات)                  │    │
│  │  ├── EARN (من SYSTEM إلى User)                      │    │
│  │  ├── SPEND (من User إلى SYSTEM)                     │    │
│  │  └── TRANSFER (بين المستخدمين)                      │    │
│  └─────────────────────────────────────────────────────┘    │
│                                                               │
│  ┌─────────────────────────────────────────────────────┐    │
│  │  PSPProgramPoints (تعريف النقاط لكل برنامج)         │    │
│  │  ├── ProgramPointID                                  │    │
│  │  ├── ProgramID                                       │    │
│  │  ├── MonetaryValue                                   │    │
│  │  └── PSPProgramPointRules (قواعد النقاط)            │    │
│  └─────────────────────────────────────────────────────┘    │
└─────────────────────────────────────────────────────────────┘
```

---

## 🎯 كيف تعمل الدورة؟

### 1. شركة الأدوية تُنشئ برنامج + نقاط

```
شركة ديفارت لاب
    ↓
برنامج Infertility Support
    ↓
PSPProgramPoints:
  - PointsName: "نقاط برنامج العقم"
  - MonetaryValue: 10 ج.م لكل نقطة
  ↓
PSPProgramPointRules:
  - PATIENT_ENROLLED (DOCTOR): 50 نقطة
  - DISPENSATION (DOCTOR): 10 نقاط
  - ADHERENCE_HIGH (DOCTOR): 100 نقطة
```

### 2. الطبيب يكسب نقاط عند الأحداث

```
الطبيب يسجل مريض جديد
    ↓
EarnRubikaHandler:
  - UserProfileID: 2 (الطبيب)
  - ProgramPointID: 1 (نقاط العقم)
  - ActivityType: PATIENT_ENROLLED
  - Points: 50
    ↓
RubikaWallet (محفظة الطبيب للبرنامج):
  - Balance += 50
    ↓
RubikaTransaction:
  - FromWalletID: SystemWallet
  - ToWalletID: DoctorProgramWallet
  - AmountInRubika: 50
```

### 3. الطبيب يصرف النقاط

```
الطبيب يستبدل 100 نقطة بمكافأة
    ↓
SpendRubikaHandler:
  - UserProfileID: 2 (الطبيب)
  - ProgramPointID: 1
  - Points: 100
  - Reward: خصم 1000 ج.م على اشتراكه
    ↓
RubikaWallet.Balance -= 100
    ↓
RubikaTransaction:
  - FromWalletID: DoctorProgramWallet
  - ToWalletID: SystemWallet
```

---

## ⚠️ التحديات

### 1. `RubikaWallet` — تعارض Unique Index

**من الخطأ السابق:**
```
Cannot insert duplicate key row in object 'dbo.RubikaWallet' 
with unique index 'IX_RubikaWallet_UserProfileID'. 
The duplicate key value is (1).
```

**السبب:** `IX_RubikaWallet_UserProfileID` **فريد** — يعني **محفظة واحدة لكل مستخدم**.

**الحل:** إزالة الـ Unique Index، واستبداله بـ:
```sql
CREATE UNIQUE INDEX IX_RubikaWallet_User_Type_Program
ON RubikaWallet (UserProfileID, PointsType, ProgramPointID)
WHERE ProgramPointID IS NOT NULL;
```

**هذا يسمح بـ:**
- محفظة واحدة لكل نوع نقاط لكل مستخدم لكل برنامج

---

### 2. تعديلات Schema

**نحتاج Migration جديدة لـ:**
- إضافة `ProgramPointID` لـ `RubikaWallet`
- إضافة `ProgramID`, `ProgramPointID`, `ActivityType`, `ReferenceID` لـ `RubikaTransaction`
- إزالة `IX_RubikaWallet_UserProfileID` الفريد
- إضافة `PSPProgramPoints` + `PSPProgramPointRules`

---

### 3. إدارة الأنشطة

**شركة الأدوية تحتاج UI لإدارة:**
- إنشاء نقاط للبرنامج
- تحديد قيمة النقطة
- تحديد الأنشطة وعدد النقاط لكل نشاط
- تفعيل/تعطيل كل نشاط

