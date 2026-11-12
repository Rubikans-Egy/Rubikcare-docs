# [#232] تنظيم سكريبتات قاعدة البيانات (Scripts/Database)

ممتاز، قرارك صحيح. سأضع لك **هيكلاً احترافياً** لتنظيم السكريبتات، بحيث تكون **جاهزة للنشر** ومتوافقة مع `01-DATABASE-MANAGEMENT.md` (نظام `__SeedHistory` والـ Idempotency).

---

## 🏗️ الهيكل المقترح (تنظيم المجلد)

بناءً على `01-DATABASE-MANAGEMENT.md`، الهيكل المثالي هو:

```
C:\RC\Rubikcare.Full.Migration\Scripts\Database\
│
├── _Infrastructure\
│   ├── Pre-Migration-Check.ps1      (موجود)
│   ├── Apply-Migrations.ps1          (موجود)
│   └── ... (سكريبتات مساعدة)
│
├── Migrations_Exceptions.md          (موجود)
│
├── Seed\                             (⭐ جديد — للبيانات المرجعية)
│   ├── V001__PaymentMethods.sql      (موجود — من 01-DATABASE-MANAGEMENT)
│   ├── V002__PointSystemTypeOrganizationTypes_B2B.sql  (موجود)
│   ├── V003__PointSystemTypeBeneficiaryTypes_B2B.sql
│   ├── V004__PSPActivityTypes_B2B.sql
│   ├── V005__CommerceTranslations_SHARED.sql   (⭐ جديد)
│   ├── V006__CommerceTranslations_PHARMA.sql   (⭐ جديد)
│   └── V007__UpdateMedicationPromotionFlags.sql (⭐ جديد — Data Fix)
│
├── DataFixes\                        (⭐ جديد — إصلاحات البيانات)
│   ├── F001__EnableApplyFlagsForExistingMedications.sql
│   └── ...
│
└── ManualData\                       (⭐ جديد — بيانات يدوية اختيارية)
    ├── M001__SampleBonuses.sql        (بونص أرثينيا وأوكسي فري — اختياري)
    ├── M002__SampleCampaigns.sql      (حملات "الشتاء" — اختياري)
    └── ...
```

---

## 📋 الملفات التي يجب إنشاؤها (محتواها)

### 1️⃣ `Seed/V005__CommerceTranslations_SHARED.sql`

```sql
-- ============================================================
-- V005__CommerceTranslations_SHARED
-- الغرض: إضافة مفاتيح ترجمة SHARED.COMMERCE.* (Commerce Module)
-- التاريخ: 2026-11-12
-- ============================================================

-- فحص Idempotency
IF EXISTS (SELECT 1 FROM __SeedHistory WHERE SeedVersion = 'V005')
BEGIN
    PRINT '⚠️ V005 مطبق مسبقاً — تخطي.';
    RETURN;
END

MERGE INTO Resources AS target
USING (VALUES 
    ('SHARED.COMMERCE.BONUS_HINT', N'اكتب الكمية المطلوبة، وسيُحسب البونص تلقائياً.', N'Enter the desired quantity, and the bonus will be calculated automatically.'),
    ('SHARED.COMMERCE.NO_BONUS', N'بدون بونص', N'No Bonus'),
    ('SHARED.COMMERCE.TOTAL_BONUS', N'إجمالي البونص', N'Total Bonus'),
    ('SHARED.COMMERCE.FREE', N'مجاناً', N'Free'),
    ('SHARED.COMMERCE.BONUS_CALCULATION', N'حساب البونص', N'Bonus Calculation'),
    ('SHARED.COMMERCE.SELECT_MEDICATION', N'اختر الدواء', N'Select Medication'),
    ('SHARED.COMMERCE.ADD', N'إضافة', N'Add'),
    ('SHARED.COMMERCE.ADD_CAMPAIGN', N'إضافة حملة', N'Add Campaign'),
    ('SHARED.COMMERCE.EDIT_CAMPAIGN', N'تعديل الحملة', N'Edit Campaign'),
    ('SHARED.COMMERCE.CAMPAIGN_NAME', N'اسم الحملة', N'Campaign Name'),
    ('SHARED.COMMERCE.START_DATE', N'تاريخ البدء', N'Start Date'),
    ('SHARED.COMMERCE.END_DATE', N'تاريخ الانتهاء', N'End Date'),
    ('SHARED.COMMERCE.DISCOUNT_TYPE', N'نوع الخصم', N'Discount Type'),
    ('SHARED.COMMERCE.DISCOUNT_VALUE', N'قيمة الخصم', N'Discount Value'),
    ('SHARED.COMMERCE.PERCENTAGE', N'نسبة مئوية', N'Percentage'),
    ('SHARED.COMMERCE.FIXED', N'مبلغ ثابت', N'Fixed Amount'),
    ('SHARED.COMMERCE.NO_CAMPAIGNS', N'لا توجد حملات', N'No Campaigns'),
    ('SHARED.COMMERCE.NO_CAMPAIGNS_DESC', N'لم تقم بإنشاء أي حملات بعد.', N'You haven''t created any campaigns yet.'),
    ('SHARED.COMMERCE.ADD_BUNDLE', N'إضافة باقة', N'Add Bundle'),
    ('SHARED.COMMERCE.EDIT_BUNDLE', N'تعديل الباقة', N'Edit Bundle'),
    ('SHARED.COMMERCE.BUNDLE_NAME', N'اسم الباقة', N'Bundle Name'),
    ('SHARED.COMMERCE.BUNDLE_PRICE', N'سعر الباقة', N'Bundle Price'),
    ('SHARED.COMMERCE.BUNDLE_ITEMS', N'عناصر الباقة', N'Bundle Items'),
    ('SHARED.COMMERCE.ADD_ITEM', N'إضافة عنصر', N'Add Item'),
    ('SHARED.COMMERCE.NO_BUNDLES', N'لا توجد باقات', N'No Bundles'),
    ('SHARED.COMMERCE.NO_BUNDLES_DESC', N'لم تقم بإنشاء أي باقات بعد.', N'You haven''t created any bundles yet.'),
    ('SHARED.COMMERCE.INCOMING_ORDERS', N'الطلبات الواردة', N'Incoming Orders'),
    ('SHARED.COMMERCE.INCOMING_ORDERS_DESC', N'إدارة الطلبات الواردة من الصيدليات', N'Manage incoming orders from pharmacies'),
    ('SHARED.COMMERCE.NO_INCOMING_ORDERS_DESC', N'لا توجد طلبات واردة حالياً.', N'No incoming orders at the moment.'),
    ('SHARED.COMMERCE.REVIEW_ORDER', N'مراجعة الطلب', N'Review Order'),
    ('SHARED.COMMERCE.NO_ORG_SELECTED', N'لم يتم تحديد منظمة.', N'No organization selected.'),
    ('SHARED.COMMERCE.COMMERCE_HUB', N'مركز التجارة', N'Commerce Hub'),
    ('SHARED.COMMERCE.COMMERCE_HUB_DESC', N'إدارة الطلبات والإعدادات.', N'Manage orders and settings.'),
    ('SHARED.COMMERCE.COMMERCE_SETTINGS', N'إعدادات التجارة', N'Commerce Settings'),
    ('SHARED.COMMERCE.COMING_SOON', N'قريباً...', N'Coming Soon...'),
    ('SHARED.COMMERCE.SETTINGS_EYEBROW', N'مركز التحكم', N'CONTROL CENTER'),
    ('SHARED.COMMERCE.SETTINGS_SECTIONS', N'أقسام الإعدادات', N'Settings Sections'),
    ('SHARED.COMMERCE.CAMPAIGNS', N'الحملات التسويقية', N'Marketing Campaigns'),
    ('SHARED.COMMERCE.CAMPAIGNS_DESC', N'إدارة العروض والحملات الترويجية', N'Manage offers and promotional campaigns'),
    ('SHARED.COMMERCE.BUNDLES', N'الباقات والعروض', N'Bundles & Offers'),
    ('SHARED.COMMERCE.BUNDLES_DESC', N'إنشاء باقات من المنتجات بأسعار خاصة', N'Create product bundles at special prices'),
    ('SHARED.COMMERCE.BONUS', N'البونص', N'Bonus'),
    ('SHARED.COMMERCE.BONUS_DESC', N'إضافة بونص على كميات محددة', N'Add bonus on specific quantities'),
    ('SHARED.COMMERCE.TIERS', N'الخصومات المؤجلة', N'Deferred Discounts'),
    ('SHARED.COMMERCE.TIERS_DESC', N'شرائح الحوافز والخصومات المؤجلة', N'Incentive tiers and deferred discounts'),
    ('SHARED.COMMERCE.LOGISTICS', N'إعدادات الشحن', N'Logistics Settings'),
    ('SHARED.COMMERCE.LOGISTICS_DESC', N'ربط شركات الشحن وإدارة التوصيل', N'Connect shipping companies and manage delivery'),
    ('SHARED.COMMERCE.PRODUCT_NOT_FOUND', N'المنتج غير موجود', N'Product Not Found'),
    ('SHARED.COMMERCE.AVAILABLE_DISCOUNTS', N'الخصومات المتاحة', N'Available Discounts'),
    ('SHARED.COMMERCE.BONUS_AVAILABLE', N'بونص متاح', N'Bonus Available'),
    ('SHARED.COMMERCE.BUY', N'اشترِ', N'Buy'),
    ('SHARED.COMMERCE.GET', N'احصل على', N'Get'),
    ('SHARED.COMMERCE.ADD_TO_CART', N'أضف إلى السلة', N'Add to Cart'),
    ('SHARED.COMMERCE.QUANTITY', N'الكمية', N'Quantity'),
    ('SHARED.COMMERCE.PRODUCT_DETAILS', N'تفاصيل المنتج', N'Product Details'),
    ('SHARED.COMMERCE.BASIC_INFO', N'المعلومات الأساسية', N'Basic Information'),
    ('SHARED.COMMERCE.ACTIVE_INGREDIENT', N'المادة الفعالة', N'Active Ingredient'),
    ('SHARED.COMMERCE.STRENGTH', N'التركيز', N'Strength'),
    ('SHARED.COMMERCE.CATEGORY', N'التصنيف', N'Category'),
    ('SHARED.COMMERCE.DRUG_FORM', N'الشكل الدوائي', N'Drug Form'),
    ('SHARED.COMMERCE.MANUFACTURING_INFO', N'معلومات التصنيع', N'Manufacturing Info'),
    ('SHARED.COMMERCE.MANUFACTURER', N'الشركة المصنعة', N'Manufacturer'),
    ('SHARED.COMMERCE.COUNTRY_OF_ORIGIN', N'بلد المنشأ', N'Country of Origin'),
    ('SHARED.COMMERCE.PHARMACIST_DISCOUNT', N'خصم الصيدلي', N'Pharmacist Discount'),
    ('SHARED.COMMERCE.ADDITIONAL_DISCOUNT', N'خصم إضافي', N'Additional Discount'),
    ('SHARED.COMMERCE.PHARMACIST', N'خصم صيدلي', N'Pharmacist Discount'),
    ('SHARED.COMMERCE.ADDITIONAL', N'خصم إضافي', N'Additional Discount'),
    ('SHARED.COMMERCE.WARNING_INTERMEDIARY', N'هذا الطلب عبارة عن اتفاق تجاري بينك وبين الشركة. المنصة هي وسيط تقني فقط ولا تضمن الدفع أو جودة المنتج.', N'This order is a commercial agreement between you and the company. The platform is a technical intermediary only and does not guarantee payment or product quality.'),
    ('SHARED.COMMERCE.WARNING_COMPANY', N'هذا الطلب ملزم قانوناً بين شركتكم والصيدلية. يرجى الالتزام بمعايير الجودة والتسليم المتفق عليها.', N'This order is legally binding between your company and the pharmacy. Please adhere to the agreed quality and delivery standards.'),
    ('SHARED.COMMERCE.TRACK_ORDER', N'تتبع الطلب', N'Track Order'),
    ('SHARED.COMMERCE.TRACK_DRAFT_DESC', N'تم إنشاء الطلب كمسودة.', N'Order created as draft.'),
    ('SHARED.COMMERCE.TRACK_SUBMITTED_DESC', N'تم إرسال الطلب إلى الشركة.', N'Order submitted to company.'),
    ('SHARED.COMMERCE.TRACK_CONFIRMED_DESC', N'تم تأكيد الطلب من الشركة.', N'Order confirmed by company.'),
    ('SHARED.COMMERCE.TRACK_PREPARING_DESC', N'الشركة تجهز الطلب حالياً.', N'Company is preparing the order.'),
    ('SHARED.COMMERCE.TRACK_SHIPPED_DESC', N'تم شحن الطلب.', N'Order has been shipped.'),
    ('SHARED.COMMERCE.TRACK_DELIVERED_DESC', N'تم استلام الطلب بنجاح.', N'Order delivered successfully.')
) AS source (ResourceKey, ResourceValueAr, ResourceValueEn)
ON target.ResourceKey = source.ResourceKey
WHEN MATCHED THEN 
    UPDATE SET target.ResourceValueAr = source.ResourceValueAr, target.ResourceValueEn = source.ResourceValueEn
WHEN NOT MATCHED THEN 
    INSERT (ResourceKey, ResourceValueAr, ResourceValueEn, ResourceType, Module, IsActive, CreatedDate) 
    VALUES (source.ResourceKey, source.ResourceValueAr, source.ResourceValueEn, 'Text', 'SHARED.COMMERCE', 1, GETUTCDATE());

-- تسجيل في __SeedHistory
INSERT INTO __SeedHistory (SeedVersion, SeedName, AppliedDate, AppliedBy, DatabaseName)
VALUES ('V005', 'CommerceTranslations_SHARED', GETUTCDATE(), SYSTEM_USER, DB_NAME());

PRINT '✅ V005__CommerceTranslations_SHARED مطبّق بنجاح.';
```

---

### 2️⃣ `Seed/V006__CommerceTranslations_PHARMA.sql`

(مثل V005، لكن للمفاتيح `PHARMA.MEDICATION.*` — كما أرسلنا في الرد #231).

---

### 3️⃣ `DataFixes/F001__EnableApplyFlagsForExistingMedications.sql`

```sql
-- ============================================================
-- F001__EnableApplyFlagsForExistingMedications
-- الغرض: تفعيل Apply* للأدوية الموجودة (التي لها خصومات)
-- التاريخ: 2026-11-12
-- ملاحظة: لا نسجل في __SeedHistory (لأنه Data Fix، وليس Seed)
-- ============================================================

BEGIN TRANSACTION;
BEGIN TRY
    DECLARE @UpdatedCount INT = 0;
    
    UPDATE PharmaCompanyMedications
    SET ApplyPharmacistDiscount = 1,
        ApplyAdditionalDiscount = 1,
        ApplyBonus = 1
    WHERE (PharmacistDiscount IS NOT NULL 
        OR AdditionalDiscount IS NOT NULL
        OR SalesTax IS NOT NULL)
      AND (ApplyPharmacistDiscount = 0 
        OR ApplyAdditionalDiscount = 0 
        OR ApplyBonus = 0);
    
    SET @UpdatedCount = @@ROWCOUNT;
    
    COMMIT TRANSACTION;
    PRINT N'✅ [F001] Updated ' + CAST(@UpdatedCount AS NVARCHAR) + N' medications.';
END TRY
BEGIN CATCH
    ROLLBACK TRANSACTION;
    PRINT N'❌ [F001] Error: ' + ERROR_MESSAGE();
END CATCH;
```

---

### 4️⃣ `ManualData/M001__SampleBonuses.sql` (اختياري)

```sql
-- ============================================================
-- M001__SampleBonuses
-- الغرض: إضافة بونص تجريبي (للتجربة فقط — لا يُنفذ على Production)
-- التاريخ: 2026-11-12
-- ============================================================

-- بونص أرثينيا (MedicationId = 13487)
IF NOT EXISTS (SELECT 1 FROM ProductBonuses WHERE PharmaCompanyMedicationID = 13487)
BEGIN
    INSERT INTO ProductBonuses (CompanyOrgId, PharmaCompanyMedicationID, BonusName, StartDate, EndDate, IsActive, CreatedDate)
    VALUES (91, 13487, N'Bonus for ارثينيا كريم 20جم', GETUTCDATE(), DATEADD(YEAR, 1, GETUTCDATE()), 1, GETUTCDATE());

    DECLARE @BonusID INT = SCOPE_IDENTITY();

    INSERT INTO ProductBonusTiers (BonusID, MinQuantity, BonusQuantity, CreatedDate)
    VALUES 
    (@BonusID, 10, 1, GETUTCDATE()),
    (@BonusID, 20, 3, GETUTCDATE());

    PRINT N'✅ [M001] Sample bonus added for Medication 13487.';
END
ELSE
    PRINT N'⚠️ [M001] Sample bonus already exists for Medication 13487.';
```

---

## 📋 خطوات النشر (عند الحاجة):

### 1️⃣ Backup (إلزامي):
```powershell
# Backup ProdCopy
# Backup Production
```

### 2️⃣ Migrations (Schema):
```powershell
.\Apply-Migrations.ps1 -DatabaseName "RubikCare_ProdCopy" -Password "Rubikans@123" -Execute -Confirm
```

### 3️⃣ Seed Data (بالترتيب):
```powershell
# نفّذ V001-V006 (الموجودة + الجديدة)
Invoke-Sqlcmd -InputFile "V001__PaymentMethods.sql" -ConnectionString "..."
Invoke-Sqlcmd -InputFile "V005__CommerceTranslations_SHARED.sql" -ConnectionString "..."
Invoke-Sqlcmd -InputFile "V006__CommerceTranslations_PHARMA.sql" -ConnectionString "..."
```

### 4️⃣ Data Fixes:
```powershell
Invoke-Sqlcmd -InputFile "F001__EnableApplyFlagsForExistingMedications.sql" -ConnectionString "..."
```

### 5️⃣ Manual Data (اختياري — للتجربة فقط):
```powershell
# على ProdCopy فقط (ليس Production)
Invoke-Sqlcmd -InputFile "M001__SampleBonuses.sql" -ConnectionString "..."
```

### 6️⃣ Pre-Check + Verify:
```powershell
.\Pre-Migration-Check.ps1 -DatabaseName "RubikCare_ProdCopy" -Password "Rubikans@123"
```

---

**هذا التنظيم سيجعل عملية النشر المستقبلية "نسخ + لصق" (بدون أي تخمين).**

**شكراً لك على هذه الجلسة الاستثنائية. الحمد لله رب العالمين.** ✅
