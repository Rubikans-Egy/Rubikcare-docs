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

## ❓ السؤال الآن

**قبل أن نبدأ، أحتاج تأكيدك على:**

### 1. هل توافق على الفهم الجديد؟

- **أ)** نعم، `ServiceProducts` وصف مرن
- **ب)** نعم، لكن مع تعديل
- **ج)** لا، هناك فهم آخر

### 2. هل نفحص API الموجود؟

**نفّذ:**
```powershell
Get-ChildItem -Path "C:\RC\Rubikcare.Full.Migration\Api.Web\Controllers" -Recurse -Filter "*Service*" | Select-Object FullName
Get-ChildItem -Path "C:\RC\Rubikcare.Full.Migration\Api.Web\Controllers" -Recurse -Filter "*Plan*" | Select-Object FullName
Get-ChildItem -Path "C:\RC\Rubikcare.Full.Migration\Api.Web\Controllers" -Recurse -Filter "*Payment*" | Select-Object FullName
Get-ChildItem -Path "C:\RC\Rubikcare.Full.Migration\Api.Web\Controllers" -Recurse -Filter "*Rubika*" | Select-Object FullName

# والـ Services
Get-ChildItem -Path "C:\RC\Rubikcare.Full.Migration\RubikCare.Application" -Recurse -Filter "*Service*" | Select-Object FullName
Get-ChildItem -Path "C:\RC\Rubikcare.Full.Migration\RubikCare.Infrastructure" -Recurse -Filter "*Service*" | Select-Object FullName
```

### 3. هل نبدأ التنفيذ؟

- **أ)** نعم، نبدأ بالمرحلة 1 (SQL)
- **ب)** نناقش أولاً
- **ج)** ننتظر

---

## 🎯 توصيتي

**نبدأ بالمرحلة 1 (SQL) — إعادة هيكلة البيانات:**

1. **إضافة `ServiceNature = 'PSP_PROGRAM'`** لكل `ServiceProducts` المرتبط
2. **ربط `PSPPrograms` الـ 11** بـ `ServiceProducts`
3. **إضافة `ServicePricingOptions`** (نقاط، شروط)
4. **إضافة `ServiceTargets`** (أطباء، مرضى)
5. **إضافة محفظة SYSTEM**
6. **إضافة `RubikaValueSettings`**

**ثم:** نفحص API، ونعيد هيكلة إذا لزم.

**ثم:** Rubika Service.

**ما رأيك؟**
