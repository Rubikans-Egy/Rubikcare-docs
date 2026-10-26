# 21 — PHR-SYSTEM (نظام السجلات الطبية الشخصية)

الإصدار: 1.0 (Draft — Foundations Edition) 
آخر تحديث: 26 أكتوبر 2026 
الحالة: 🟡 مسودة — بانتظار المراجعة والاعتماد 
الجمهور: فريق التطوير + AI Assistants + القيادة التقنية 
المرجع الإلزامي: يُقرأ قبل أي تطوير يتعلق بالسجلات الطبية.

---

## 📌 الجزء صفر: الفلسفة والرؤية

### 0.1 ما هو PHR؟
PHR = Personal Health Record = السجل الطبي الشخصي.

**ليس** مجرد جدول لنتائج التحاليل. هو **الذاكرة الطبية للمستخدم** (UserProfile)، ويمثل **حجر الزاوية (Cornerstone)** الذي يربط كل الخدمات في منصة RubikCare.

### 0.2 الرؤية الاستراتيجية (من 17-STRATEGIC-ROADMAP)
> "كل مستخدم حالي هو مريض محتمل... المرض في المنصة ليس دوراً ولكنه حالة مؤقتة قد يمر بها الطبيب والمندوب والصيدلي."
> (نص المؤلف الحرفي — وثيقة 17)

الترجمة التقنية:
- `PHR` مرتبط بـ `UserProfileID` (الشخص)، وليس بـ `PatientProfileID` أو `CaseID` (الحالة).
- المستخدم يملك سجله الطبي بغض النظر عن اشتراكه في أي برنامج (PSP أو غيره).
- PHR هو **المصدر الوحيد للحقيقة (Single Source of Truth)** لكل ما يتعلق بالصحة.

### 0.3 المبادئ التصميمية (مستوحاة من 17)
1. **مرونة كاملة (Nothing Hardcoded)**: أنواع التحاليل، القواعد، والقياسات كلها في جداول مرجعية (Reference Tables).
2. **قابلية التوسع (Extensibility)**: إضافة نوع سجل جديد (عملية، وصفة، قياس) = إضافة صف في جدول مرجعي + كيان تفصيلي.
3. **التحكم الكامل للمستخدم (User Consent)**: المريض هو **المالك** (Data Owner). لا أحد يرى بياناته إلا بموافقته الصريحة والمحددة زمنياً.
4. **التكامل المخطط (Planned Integration)**: PHR ليس جزيرة معزولة. هو **مستهلك ومزوّد** للبيانات من/إلى:
   - **PSP**: استقبال نتائج التحاليل المطلوبة.
   - **Clinic OS** (مستقبلاً): استقبال السجلات من العيادات وإرسالها.
   - **Labs** (مستقبلاً): استقبال النتائج مباشرة من المعامل.
   - **B2C/B2B** (مستقبلاً): ربط الوصفات والعلاجات.
5. **التوثيق قبل التنفيذ**: هذه الوثيقة.

---

## 🗄️ الجزء الأول: نموذج البيانات (Data Model)

### 1.1 الطبقات الثلاث
PHR يتكون من 3 طبقات مترابطة:

```
┌─────────────────────────────────────────────────────────────┐
│  Layer 1: Reference Data (جداول مرجعية)                     │
│  LabTestType, MedicalRecordType, AssessmentRule             │
├─────────────────────────────────────────────────────────────┤
│  Layer 2: Core Entities (كيانات أساسية)                     │
│  HealthRecord, LabTestResult, Prescription, Measurement     │
├─────────────────────────────────────────────────────────────┤
│  Layer 3: Consent & Sharing (المشاركة والتحكم)              │
│  SharingConsent, RecordAccessLog                            │
└─────────────────────────────────────────────────────────────┘
```

### 1.2 الطبقة 1: الجداول المرجعية (Reference Data)

#### 1.2.1 `MedicalRecordType` (أنواع السجلات)
| العمود | النوع | الوصف |
| :--- | :--- | :--- |
| `RecordTypeID` | PK, int | |
| `Code` | nvarchar(50) | `LAB_RESULT`, `PRESCRIPTION`, `SURGERY`, `MEASUREMENT`, `VACCINATION` |
| `NameAr`, `NameEn` | nvarchar(200) | |
| `IconName` | nvarchar(100) | للعرض في الـ Timeline |
| `IsActive` | bit | |

#### 1.2.2 `LabTestType` (أنواع التحاليل)
| العمود | النوع | الوصف |
| :--- | :--- | :--- |
| `LabTestTypeID` | PK, int | |
| `Code` | nvarchar(50) | `HB`, `FERRITIN`, `VITAMIN_D` |
| `NameAr`, `NameEn` | nvarchar(200) | |
| `DefaultUnit` | nvarchar(50) | `g/dL`, `ng/mL` |
| `NormalRangeMin`, `NormalRangeMax` | decimal | |
| `PlausibleRangeMin`, `PlausibleRangeMax` | decimal | للتحقق من صحة الإدخال |
| `ExtraFieldsJson` | nvarchar(max) | **JSON** يحدد الحقول الإضافية المطلوبة (مثل `Trimester`, `Month` لـ Hb) |
| `AssessmentRuleTypeID` | FK → `AssessmentRuleType` | يحدد أي محرك تقييم يُستخدم |

#### 1.2.3 `AssessmentRule` (قواعد التقييم)
| العمود | النوع | الوصف |
| :--- | :--- | :--- |
| `RuleID` | PK, int | |
| `LabTestTypeID` | FK | |
| `SeverityCode` | nvarchar(50) | `NORMAL`, `BORDERLINE`, `MILD`, `MODERATE`, `SEVERE` |
| `MinValue`, `MaxValue` | decimal | |
| `TrimesterConstraint` | int? | (1, 2, 3, null = any) |
| `RecommendationText` | nvarchar(max) | |
| `SafetyNote` | nvarchar(max) | |
| `CssClass` | nvarchar(50) | للتلوين (`severity-normal`, `severity-severe`) |

### 1.3 الطبقة 2: الكيانات الأساسية (Core Entities)

#### 1.3.1 `HealthRecord` (السجل الموحد — Aggregation Layer)
هذا هو الجدول المركزي الذي يمثل "التايم لاين" الطبي للمستخدم.

| العمود | النوع | الوصف |
| :--- | :--- | :--- |
| `RecordID` | PK, int | |
| `UserProfileID` | FK → UserProfiles | **الشخص** (وليس الحالة) |
| `RecordTypeID` | FK → `MedicalRecordType` | |
| `RecordDate` | datetime2 | تاريخ الحدث الطبي |
| `SourceEntityID` | int | **مثل نظام النقاط**: يشير إلى `LabTestResultID` أو `PrescriptionID` |
| `Summary` | nvarchar(500) | وصف مختصر للعرض |
| `CreatedBy` | int | من أدخل السجل (المريض / الطبيب) |
| `CreatedDate` | datetime2 | |
| `IsDeleted` | bit | Soft Delete |

**قاعدة صارمة:** لا يتم حذف أي سجل طبي فعلياً (Soft Delete فقط) — لأسباب قانونية وأخلاقية.

#### 1.3.2 `LabTestResult` (نتيجة تحليل)
| العمود | النوع | الوصف |
| :--- | :--- | :--- |
| `ResultID` | PK, int | |
| `HealthRecordID` | FK → `HealthRecord` | |
| `LabTestTypeID` | FK → `LabTestType` | |
| `Value` | decimal | |
| `Unit` | nvarchar(50) | |
| `ReadingDate` | datetime2 | |
| `AssessedSeverity` | nvarchar(50) | من `AssessmentRule` |
| `AssessmentRuleID` | FK → `AssessmentRule` | |
| `ExtraDataJson` | nvarchar(max) | JSON (مثلاً `{"trimester": 2, "month": 6}`) |
| `AttachmentUrl` | nvarchar(500) | (اختياري) ملف PDF |

#### 1.3.3 كيانات مستقبلية (Planned — لا تُبنى الآن)
- `Prescription` (وصفة طبية).
- `Surgery` (عملية جراحية).
- `Measurement` (قياس طبي: ضغط، سكر، وزن).
- `Vaccination` (تطعيم).

**قاعدة ذهبية:** لا نبنيها الآن. نبني البنية التي تسمح بإضافتها لاحقاً عبر `HealthRecord` + `SourceEntityID`.

### 1.4 الطبقة 3: المشاركة والتحكم (Consent & Sharing)

#### 1.4.1 `SharingConsent` (موافقة المشاركة)
| العمود | النوع | الوصف |
| :--- | :--- | :--- |
| `ConsentID` | PK, int | |
| `PatientUserProfileID` | FK → UserProfiles | المالك |
| `SharedWithTypeID` | FK → OrganizationTypes | (عيادة / طبيب / معمل) |
| `SharedWithOrgID` | FK → Organizations | |
| `SharedWithUserProfileID` | FK → UserProfiles (nullable) | لطبيب محدد |
| `ScopeJson` | nvarchar(max) | `{"recordTypes": ["LAB_RESULT"], "labTestTypes": ["HB"]}` |
| `ValidFrom`, `ValidTo` | datetime2 | صلاحية زمنية |
| `RevokedAt` | datetime2 (nullable) | تاريخ الإلغاء |
| `ConsentText` | nvarchar(max) | النص الذي وافق عليه المريض |

#### 1.4.2 `RecordAccessLog` (سجل الوصول)
| العمود | النوع | الوصف |
| :--- | :--- | :--- |
| `LogID` | PK, int | |
| `HealthRecordID` | FK → `HealthRecord` | |
| `AccessedByUserProfileID` | FK → UserProfiles | |
| `AccessedDate` | datetime2 | |
| `Action` | nvarchar(50) | `VIEW`, `DOWNLOAD`, `SHARE` |

---

## 🎯 الجزء الثاني: سيناريوهات التكامل (Integration Scenarios)

### 2.1 السيناريو 1: Hb Navigator (Phase 1)
```
المريض (UserProfileID = 15) يدخل Hb = 7, Trimester = 2, Month = 6
    ↓
HbClinicalRulesEngine يقيّم → Severe
    ↓
INSERT INTO HealthRecord (UserProfileID=15, RecordTypeID=LAB_RESULT, RecordDate=...)
    ↓
INSERT INTO LabTestResult (HealthRecordID=..., LabTestTypeID=HB, Value=7, ...)
    ↓
(لاحقاً) مشاركة مع العيادة عبر SharingConsent
```

### 2.2 السيناريو 2: التكامل مع PSP (Phase 3)
```
المريض مشترك في PSP (PSPRequiredTest: Hb قبل البدء)
    ↓
PSPEnrollmentService يطلب من المريض رفع التحليل
    ↓
المريض يرفع القراءة عبر Hb Navigator
    ↓
INSERT INTO HealthRecord + LabTestResult
    ↓
PSPTestResult يُنشأ تلقائياً (ربط PSP بـ PHR)
    ↓
الشركة ترى النتيجة (مع احترام Privacy — PatientAnonymousId)
```

### 2.3 السيناريو 3: التكامل مع Clinic OS (مستقبلاً)
```
العيادة (Organizations) لديها نظام إدارة (Clinic OS)
    ↓
المريض يشارك سجله مع العيادة (SharingConsent)
    ↓
Clinic OS يستدعي API: GET /api/phr/records?consentId=...
    ↓
PHR يُرجع السجلات المصرح بها فقط
    ↓
RecordAccessLog يُسجل الوصول
    ↓
الطبيب يُضيف ملاحظة (DoctorNote) تُحفظ في PHR (كـ RecordType = NOTE)
```

### 2.4 السيناريو 4: استقبال السجلات من المعامل (مستقبلاً)
```
معمل (OrganizationType = LAB) يرفع نتيجة تحليل
    ↓
Webhook / API Call إلى PHR
    ↓
INSERT INTO HealthRecord + LabTestResult (CreatedBy = Lab)
    ↓
إشعار للمريض (Notification Service)
    ↓
المريض يرى النتيجة في سجله
```

---

## 🔗 الجزء الثالث: التكامل مع النظم الحالية والمستقبلية

### 3.1 مصفوفة التكامل

| النظام | الاتجاه | الآلية | الحالة |
| :--- | :--- | :--- | :--- |
| **Identity** | ← | `UserProfileID` | ✅ موجود |
| **PSP** | ↔ | `PSPTestResult` ↔ `LabTestResult` | 🟡 Phase 3 |
| **Notifications** | ← | `INotificationService` | 🟡 Phase 1 |
| **Permissions** | ← | `SharingConsent` + `ControlLevel` | 🟡 Phase 2 |
| **Clinic OS** | ↔ | API (REST) | 🎯 مستقبلاً |
| **Labs** | ← | Webhook | 🎯 مستقبلاً |
| **B2C/B2B** | ← | `Prescription` ↔ `HealthRecord` | 🎯 مستقبلاً |

### 3.2 عقود التكامل (Integration Contracts)
على غرار `01-ARCHITECTURE` (الجزء السادس-ب)، كل تكامل يجب أن يكون موثقاً:

| # | From → To | Event | Reference | Idempotency |
| :--- | :--- | :--- | :--- | :--- |
| IC-PHR-01 | PHR → Notifications | RecordCreated | `HEALTH_RECORD/{RecordID}` | N/A |
| IC-PHR-02 | PHR → PSP | TestResultUploaded | `LAB_RESULT/{ResultID}` | ✅ فحص مسبق |
| IC-PHR-03 | Clinic OS → PHR | RecordShared | `CONSENT/{ConsentID}` | N/A |
| IC-PHR-04 | Labs → PHR | LabResultReceived | `LAB_RESULT/{ResultID}` | ✅ فحص مسبق |

---

## 🔐 الجزء الرابع: الأمان والخصوصية (Security & Privacy)

### 4.1 المبادئ
1. **المريض هو المالك (Data Owner)**: لا وصول بدون `SharingConsent` نشط.
2. **الحد الأدنى من البيانات (Least Privilege)**: `ScopeJson` يحدد بدقة ما هو مصرح به.
3. **الشفافية (Transparency)**: `RecordAccessLog` يُسجل كل وصول.
4. **قابلية الإلغاء (Revocability)**: `RevokedAt` يلغي الوصول فوراً.

### 4.2 الصلاحيات (Permissions)
- `PHR.READ.OWN`: قراءة سجلاته.
- `PHR.WRITE.OWN`: إضافة سجلات.
- `PHR.READ.SHARED`: قراءة السجلات المشتركة (للأطباء).
- `PHR.SHARE`: مشاركة السجلات (للمريض).
- `PHR.ADMIN`: إدارة (للمسؤول).

**يتكامل مع `19-PERMISSIONS.md`.**

---

## 📅 الجزء الخامس: خارطة التنفيذ (Roadmap)

### Phase 1: Foundation (Sprint 4 — 1-5 نوفمبر 2026)
- Domain: `LabTestType`, `AssessmentRule`, `HealthRecord`, `LabTestResult`.
- Application: `HbClinicalRulesEngine`, `SaveLabResultUseCase`.
- API: `HealthRecordsController` (Endpoints أساسية).
- UI: `HbNavigator.razor` (Shared.UI).

### Phase 2: Expansion (Sprint 5 — 8-12 نوفمبر 2026)
- إضافة `LabTestType` جديد (Ferritin).
- واجهة ديناميكية (Dynamic Form).
- `SharingConsent` (Scope: ALL, LAB_RESULTS).
- UI: شاشة المشاركة.

### Phase 3: Full Integration (Sprint 6 — 15-19 نوفمبر 2026)
- `HealthRecord` يدعم `PRESCRIPTION`, `SURGERY`, `MEASUREMENT`.
- ربط PHR بـ PSP.
- `SharingConsent` (Scope: JSON، صلاحية زمنية).
- `RecordAccessLog`.
- UI: لوحة "السجل الطبي" (Timeline View).

---

## 🔗 روابط ذات صلة

- `01-ARCHITECTURE.md` — Integration Contracts.
- `05-PSP.md` — نظام التحاليل الحالي.
- `17-STRATEGIC-ROADMAP.md` — المبادئ التصميمية.
- `19-PERMISSIONS.md` — الصلاحيات.
- `20-USER JOURNEY.md` — رحلة المريض.

---

## 📝 الجزء السادس: سجل التحديثات

| الإصدار | التاريخ | التغييرات |
| :--- | :--- | :--- |
| 1.0 (Draft) | 26 أكتوبر 2026 | إنشاء الوثيقة — الفلسفة + نموذج البيانات + التكامل + الأمان + خارطة التنفيذ |

---

© 2026 RubikCare — للاستخدام الداخلي
نهاية الوثيقة ✅
```
