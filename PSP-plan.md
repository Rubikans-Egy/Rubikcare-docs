# 📊 مرجع بيانات PSP - النسخة الموحدة

## 📅 تاريخ الإنشاء: 20 يناير 2026

## 🔄 طريقة التحديث: عند أي تغيير في النماذج أو العلاقات

---

## 🏗️ **الجداول التسعة المكتملة:**

### **1. PSPPrograms** - برامج دعم المرضى
```sql
الغرض: البرامج الرئيسية التي تقدمها شركات الأدوية

الحقول الأساسية:
- ProgramID (PK, int) ← معرف البرنامج
- ProgramCode (nvarchar(40)) ← كود البرنامج
- ProgramNameAr (nvarchar(400)) ← الاسم العربي
- PharmaCompanyID (FK → Organizations) ← شركة الأدوية
- ServiceID (FK → Services, nullable) ⭐ للتسعير
- StartDate (datetime2) ← تاريخ البدء
- EndDate (datetime2) ← تاريخ الانتهاء
- Budget (decimal) ← الميزانية
- IsActive (bit) ← مفعل/معطل
```

### **2. PSPProgramMedications** - أدوية البرامج  
```sql
الغرض: ربط البرامج بالأدوية المدعومة

الحقول الأساسية:
- ProgramMedicationID (PK, int) ← المعرف
- ProgramID (FK → PSPPrograms) ← البرنامج
- MedicationID (FK → Medications) ← الدواء
- SupportType (nvarchar(40)) ← نوع الدعم (نسبة، مبلغ)
- SupportValue (decimal) ← قيمة الدعم
- MaxQuantityPerPatient (int) ← الحد الأقصى لكل مريض
```

### **3. PSPEnrollments** - انضمام الأطباء للبرامج
```sql
الغرض: تسجيل الأطباء في برامج PSP

الحقول الأساسية:
- EnrollmentID (PK, int) ← المعرف
- ProgramID (FK → PSPPrograms) ← البرنامج
- DoctorID (FK → UserProfiles) ← الطبيب
- Status (nvarchar(40)) ← الحالة (معلق، مقبول، مرفوض)
- SubmittedDate (datetime2) ← تاريخ التقديم
- ReviewedBy (FK → UserProfiles, nullable) ← المراجع
- IsActive (bit) ← نشط/غير نشط
```

### **4. PSPPatients** - مرضى مسجلين في البرامج
```sql
الغرض: المرضى الذين يستفيدون من برامج PSP

الحقول الأساسية:
- PatientID (PK, int) ← معرف المريض في PSP
- EnrollmentID (FK → PSPEnrollments) ← الانضمام
- PatientProfileID (FK → UserProfiles) ← ملف المريض
- DiagnosisDetails (nvarchar(2000)) ← تفاصيل التشخيص
- Status (nvarchar(40)) ← حالة المريض (نشط، مكتمل، متوقف)
- EnrollmentDate (datetime2) ← تاريخ التسجيل
- CaseManagerID (FK → UserProfiles, nullable) ← مدير الحالة
```

### **5. PSPeRX** - وصفات PSP الإلكترونية
```sql
الغرض: وصفات طبية إلكترونية مع نظام OTP

الحقول الأساسية:
- eRxID (PK, int) ← معرف الوصفة
- PatientID (FK → PSPPatients) ← المريض
- DoctorID (FK → UserProfiles) ← الطبيب الواصف
- ProgramID (FK → PSPPrograms) ← البرنامج
- MedicationID (FK → Medications) ← الدواء
- eRxCode (nvarchar(50)) ← كود الوصفة (مثال: PSP-RX-2026-001)
- Quantity (int) ← الكمية
- OTPCode (nvarchar(10), nullable) ← كود OTP لمرة واحدة
- OTPExpiryDate (datetime2, nullable) ← صلاحية الـOTP
- Status (nvarchar(20)) ← الحالة (ACTIVE, DISPENSED, EXPIRED)
```

### **6. PSPDispensation** - خطط صرف الأدوية
```sql
الغرض: تحديد كيف ومتى يصرف الدواء للمريض

الحقول الأساسية:
- DispensationID (PK, int) ← معرف الخطة
- eRxID (FK → PSPeRX) ← الوصفة
- PlanType (nvarchar(20)) ← نوع الخطة (SINGLE, MULTIPLE)
- AllowRefills (bit) ← يسمح بالتكرر؟
- DaysBetweenRefills (int, nullable) ← الأيام بين كل صرف
- TokenValidityHours (int, nullable) ← صلاحية التوكن (ساعات)
- MaxQuantityPerDispense (int, nullable) ← الحد الأقصى لكل صرف
```

### **7. PSPTransactions** - معاملات مالية
```sql
الغرض: تسجيل الحركات المالية المرتبطة بصرف الأدوية

الحقول الأساسية:
- TransactionID (PK, int) ← المعاملة
- DispensationID (FK → PSPDispensation) ← الصرف
- TransactionType (nvarchar(20)) ← النوع (PATIENT_PAYMENT, PSP_COVERAGE)
- Amount (decimal) ← المبلغ
- PaymentMethod (nvarchar(20)) ← طريقة الدفع (CASH, CARD, MADA)
- TransactionStatus (nvarchar(20)) ← الحالة (PENDING, COMPLETED, FAILED)
- TransactionDate (datetime2) ← تاريخ المعاملة
```
### **8. pspspecialities** التخصصات المستهدفة 
```sql
الغرض:الربط بجدول التخصصات لتحديد التخصصات المستهدفة في كل برنامج دعم 

- PSPProgramSpecialityID	(PK, int) 
ProgramID	int	 (FK → pspprogram)
SpecialityID	int	
AddedDate	datetime2(7)	
AddedBy	int	Checked
Notes	nvarchar(500)	
IsActive	bit	
```
### **9.PSPInvitations** التخصصات المستهدفة 
```sql
الغرض:الربط بجدول التخصصات لتحديد التخصصات المستهدفة في كل برنامج دعم 

- InvitationID	(PK, int) 	
- InvitedOrganizationID	int (FK → Organizations)
- ProgramID	int	(FK → PSPPrograms)
- InvitedByUserID	int	(FK → Userprofiles)
- Status	nvarchar(20)	
- InvitationDate	datetime2(7)	
- ResponseDate	datetime2(7)	
- ResponseNotes	nvarchar(1000)	
- SourceType	nvarchar(20)	
- ReferralOrganizationID	int	
- ReferralUserID	int	
- InvitedOrganizationType	nvarchar(20)	
- InvitationMessage	nvarchar(2000)	
- ExpiryDate	datetime2(7)	
- NotificationSent	bit	
- NotificationSentDate	datetime2(7)	
- Notes	nvarchar(1000)	
- ResultingParticipationParticipationID	int	
		
```
---

## 🔗 **العلاقات مع النظام الحالي:**

### **✅ علاقات مؤكدة (موجودة وتعمل):**

1. `PSPPrograms` → `Organizations` ✓
2. `PSPPrograms` → `UserProfiles` ✓  
3. `PSPProgramMedications` → `Medications` ✓
4. `PSPEnrollments` → `UserProfiles` ✓
5. `PSPPatients` → `UserProfiles` ✓
6. `PSPeRX` → `Medications` ✓
7. `PSPDispensations` → `Organizations` ✓
8. `PSPPrograms` → `ServiceProducts`✓

### **🔗 العلاقات الحالية:**
```sql
-- العلاقات الموجودة (كلها صحيحة):
✅ PSPPrograms.PharmaCompanyID → Organizations.OrganizationID
✅ PSPProgramMedications.MedicationID → Medications.MedicationID
✅ PSPEnrollments.DoctorID → UserProfiles.UserProfileID
✅ PSPPatients.PatientProfileID → UserProfiles.UserProfileID
✅ PSPeRX.MedicationID → Medications.MedicationID
✅ PSPDispensations.PharmacyOrganizationID → Organizations.OrganizationID
✅ PSPPrograms.ServiceProductID → ServiceProducts.ServiceProductID


### **📋  ما تم عمله مؤخرا:**

✅-- 1. إضافة حقول الربط
ALTER TABLE PSPPrograms 
ADD ServiceProductID INT NULL,
    MonthlyFee DECIMAL(18,2) NULL,
    BillingCycle NVARCHAR(20) NULL;

✅-- 2. إضافة العلاقة
ALTER TABLE PSPPrograms
ADD CONSTRAINT FK_PSPPrograms_ServiceProducts
FOREIGN KEY (ServiceProductID) REFERENCES ServiceProducts(ServiceProductID);


✅### **تأثير على DbContext:**


public DbSet<ServiceProduct> ServiceProducts { get; set; }

✅// في PSPProgram.cs إضافة:
public int? ServiceProductID { get; set; }
public decimal? MonthlyFee { get; set; }
public string? BillingCycle { get; set; }

[ForeignKey("ServiceProductID")]
✅public virtual ServiceProduct LinkedService { get; set; }
```

## 🔍 **استعلامات سريعة للتحقق:**

### **1. التحقق من وجود الجداول:**
```sql
SELECT name AS TableName 
FROM sys.tables 
WHERE name LIKE 'PSP%' 
ORDER BY name;
```

### **2. عينة بيانات (آخر 3 سجلات):**
```sql
-- لكل جدول
SELECT TOP 3 * FROM PSPPrograms ORDER BY ProgramID DESC;
SELECT TOP 3 * FROM PSPProgramMedications ORDER BY ProgramMedicationID DESC;
-- وهكذا لباقي الجداول...
```

### **3. التحقق من العلاقات:**
```sql
SELECT 
    fk.name AS ForeignKeyName,
    OBJECT_NAME(fk.parent_object_id) AS ChildTable,
    COL_NAME(fkc.parent_object_id, fkc.parent_column_id) AS ChildColumn,
    OBJECT_NAME(fk.referenced_object_id) AS ParentTable,
    COL_NAME(fkc.referenced_object_id, fkc.referenced_column_id) AS ParentColumn
FROM sys.foreign_keys fk
INNER JOIN sys.foreign_key_columns fkc ON fk.object_id = fkc.constraint_object_id
WHERE OBJECT_NAME(fk.parent_object_id) LIKE 'PSP%'
   OR OBJECT_NAME(fk.referenced_object_id) LIKE 'PSP%'
ORDER BY ChildTable;
```

---

## 🎯 **التحديثات اليومية:**

### ملخص جلسة تطوير منصة روبيك كير** -  📅 5 فبراير 2026**
 #### 🎯 تحليل الوضع الحالي والإنجازات:
 ✅ ما أنجزناه بنجاح:
 ✅ نظام ServiceTargets (جدول وسيط + علاقات)
 ✅ تصحيح البيانات الهرمية (ServiceCategories)
 ✅ واجهة PSPProgramEdit مع النظام الهرمي
 ✅ نظام فلترة الخدمات حسب الأهداف

 #### 📋 الأجزاء المتبقية:
 **1. الصفحات التي تحتاج ترحيل للنظام الجديد:**

 1. ServiceCategories.razor    ← إدارة التصنيفات الهرمية
 2. ServiceProducts.razor      ← إدارة الخدمات
 3. ServiceTargets.razor       ← تخصيص الخدمات (جديد - لم ننشئه بعد)
 
 **2. التحسينات المطلوبة على PSPProgramEdit:**
 1. مراجعة نظام حفظ الأدوية والتخصصات الصة بكل برنامج دعم
 2. تحسين رسائل الخطأ والتوجيه
 3. إضافة تحقق من الميزانية


### 📅 تاريخ التحديث: 4 فبراير 2026

 #### 1. 🔧 التعديلات الحديثة:

 ✅ تحويل من ملف واحد 2100+ سطر إلى مناطق #region منظمة
 ✅ فصل المنطق إلى 9 أقسام منطقية
 ✅ الحفاظ على كل الوظائف أثناء التنظيم

 #### 2. 🎯 إكمال نظام إدارة الأدوية
 ✅ ربط أدوات الواجهة مع PSPProgramService
 ✅ إضافة دوال الحذف الفردي والجماعي مع تأكيد
 ✅ تحسين RubikListView للأدوية (مع إصلاح مشكلة 2 checkboxes)
 ✅ إضافة البحث والفلترة المحلية

 #### 3. 🔄 تحسين الأداء والفهم
 ✅ اكتشاف أن الأدوية لا تُحمَّل تلقائياً
 ✅ إضافة تحميل تلقائي عند اختيار الشركة
 ✅ تحسين رسائل الأخطاء والتشخيص

### 📋 **تلخيص جلسة اليوم 3 فبراير 2026**

 ✅ **ما تم إنجازه:**

 #### **1. المهمة الأولى: تصحيح بيانات شركة ديفارت لاب ✓**
 - نقل 26 دواء من 4 شركات مكررة إلى الشركة الرئيسية (ID: 5)
 - تعطيل الشركات المكررة (IsActive=0, IsDeleted=1)

 #### **2. المهمة الثانية: إضافة نظام الأدوية لبرامج PSP ✓**
 - استخدام `RubikListView` لاختيار الأدوية
 - تصميم واجهة عرض الأدوية مع التصنيف والشكل الدوائي
 - إضافة جدول عرض الأدوية المختارة

 #### **3. المهمة الثالثة: تحسين نظام الحفظ ✓**
 - تصحيح `SaveSelectedMedications()` بإضافة Transaction
 - فهم نظام حفظ التخصصات (في الذاكرة أولاً، ثم حفظ نهائي)
---


## 📞 **رابط سريع مع الوثيقة الرئيسية:**

```markdown
### **في `PSP-MASTER-PLAN-DASHBOARD.md`:**
## 📋 مرجع البيانات:
✅ 9 جداول PSP مكتملة  
✅ 6 علاقات مؤكدة مع النظام  


🔗 [تفاصيل كاملة في PSP-DATA-REFERENCE.md](PSP-DATA-REFERENCE.md)
```

## 🎯 **الخلاصة النهائية الواضحة:**

### **ما اكتشفناه من اللائحة الكاملة:**
1. ✅ **نظام كامل:** 41 جدول (كل شيء موجود)
2. ✅ **PSP مكتمل:** 9 جداول + علاقات صحيحة

---

**✅ تم تحديث كل الوثائق بناءً على التحليل الشامل.**  

**✅ تم إنشاء `PSP-DATA-REFERENCE.md` في: 20 يناير 2026 - 17:20**

الوثيقة جاهزة للاستخدام:
- ✅ واضحة ومباشرة
- ✅ سهلة التحديث
- ✅ مرتبطة بـ Dashboard الرئيسي
- ✅ تحتوي على كل ما تحتاجه للبدء غداً


## 📊 **النتيجة النهائية:**
- **✅ USER_BASE:** 7 عناصر (القائمة الشخصية)
- **✅ PHARMA_PSP:** 3 عناصر (برامج دعم المرضى)
- **✅ ADMIN_MENU:** 1 عنصر (لوحة التحكم المركزية) - **تمت الإضافة!**

---

---

**هل تريد إضافة شيء للتلخيص أو تعديل الأولويات؟**