# 26 — نظام النقاط وروبيكا (Rubika Points & Currency System)

**آخر تحديث:** 8 أكتوبر 2026
**الإصدار:** 3.0 (تحديث شامل — الرؤية الكاملة + Beneficiary Types + Organization Groups + Rubika Currency)
**الحالة:** ✅ Backend مكتمل جزئياً — UI قيد الانتظار — Migration جديدة مطلوبة
**المؤلف:** فريق RubikCare

---

## 📌 نظرة عامة

وثيقة **نظام النقاط وروبيكا** هي المرجع الشامل لجميع أنظمة القيمة داخل منصة RubikCare. تشمل:

1. **نظام النقاط (Points System)** — نظام مكافآت تحفيزية تُصدره شركات الأدوية والمؤسسات
2. **عملة روبيكا (Rubika Currency)** — عملة داخلية موحّدة تُصدرها المنصة للاستخدام في الخدمات
3. **البنية التحتية المشتركة** — المستفيدون، المجموعات، الأنظمة المرجعية

**الفلسفة الأساسية:**
- 🎯 **مرونة كاملة** — كل شيء ديناميكي (أنواع، مستفيدون، مجموعات)
- 🔄 **نظامان، محرك واحد** — Points و Rubika منفصلان في البيانات، متكاملان في الخدمة
- 🏢 **المؤسسات أولاً** — المؤسسة هي الفاعل الأساسي، والأفراد يعملون من خلالها
- 👤 **كل فرد مريض محتمل** — لا يوجد "دور مريض" منفصل
- 📈 **قابل للتوسع لعشر سنوات** — إضافة أي خدمة = INSERT، ليس Migration

---

## 🗺️ الخريطة المعمارية

### المستويات الأربعة

```
┌─────────────────────────────────────────────────────────────────┐
│  Layer 1: Reference Layer (ديناميكي بالكامل)                     │
│  ├── OrganizationTypes (أنواع المؤسسات — قابل للتوسع)            │
│  ├── PointSystemTypes (أنواع أنظمة النقاط — 5 قيم)               │
│  ├── PointActivityTypes (أنواع الأحداث — 7 قيم)                  │
│  └── MembershipTypes (أنواع العضويات — قابل للتوسع)              │
└─────────────────────────────────────────────────────────────────┘
                              ▲
                              │
┌─────────────────────────────────────────────────────────────────┐
│  Layer 2: Beneficiary Layer (ديناميكي)                           │
│  ├── PointBeneficiaryTypes (6 قيم — المستوى الأعلى)              │
│  └── PointSchemeBeneficiaryTypes (M:N + تفصيل)                   │
└─────────────────────────────────────────────────────────────────┘
                              ▲
                              │
┌─────────────────────────────────────────────────────────────────┐
│  Layer 3: Organization Structure                                 │
│  ├── Organizations (موجود)                                       │
│  ├── OrganizationGroups (جديد — مجموعات المؤسسات)                │
│  ├── OrganizationGroupMembers (جديد — أعضاء المجموعة)            │
│  ├── UserProfiles (موجود — كل إنسان)                             │
│  └── OrgMemberships (موجود — عضوية الشخص في مؤسسة)               │
└─────────────────────────────────────────────────────────────────┘
                              ▲
                              │
┌─────────────────────────────────────────────────────────────────┐
│  Layer 4: Value Engines (محركان — منفصلان في البيانات)           │
│  ├── Program Points Engine (مكافآت الشركات)                      │
│  │   ├── PSPProgramPointSchemes                                  │
│  │   ├── PSPProgramPointRules                                    │
│  │   ├── PSPProgramPointBalances                                 │
│  │   ├── PSPProgramPointTransactions                             │
│  │   └── PSPProgramPointRedeemRequests                           │
│  │                                                               │
│  └── Rubika Currency Engine (عملة المنصة)                        │
│      ├── RubikaWallet                                            │
│      ├── RubikaTransactions                                      │
│      └── RubikaValueSettings                                     │
└─────────────────────────────────────────────────────────────────┘
```

---

## 🎯 الجزء 1: الرؤية الكاملة — نظامان لا واحد

### 1.1 التمييز الجوهري

| البُعد | **Program Points** | **Rubika Currency** |
|--------|--------------------|---------------------|
| **الطبيعة** | نقاط تحفيزية (Rewards) | عملة داخلية (Currency) |
| **القيمة** | متغيرة (حسب الشركة) | ثابتة (1 Rubika = X EGP) |
| **المُصدر** | شركة أدوية / مؤسسة | **RubikCare** (المنصة) |
| **الغرض** | تحفيز سلوكي لحملة تسويقية | تشغيل خدمات + ولاء |
| **الدورة** | كسب → صرف (Linear) | شحن → استخدام → خصم (Circular) |
| **المحاسبة** | Marketing Cost | Prepaid Balance |
| **الاستخدام** | مكافآت مالية | دفع مقابل خدمات |
| **الجدول** | `PSPProgramPoint*` | `RubikaWallet` + `RubikaTransactions` |

### 1.2 أمثلة توضيحية

**مثال 1: Program Points (من شركة أدوية)**
```
شركة أدوية "X" تطلق برنامج دعم مرضى السكر:
- الطبيب يسجّل مريضًا → 100 نقطة للعيادة
- المريض يصرف الدواء → 50 نقطة للصيدلية
- المريض يعمل تحليل → 30 نقطة للمعمل

العيادة تجمع 1000 نقطة → تستبدلها بـ 1000 EGP نقدًا (تدفعها شركة X)
```

**مثال 2: Rubika Currency (من المنصة)**
```
عيادة "Y" تشحن محفظتها بـ 500 Rubika (= 500 EGP)
- كل حجز أونلاين = 5 Rubika (تُخصم من الرصيد)
- العيادة تستقبل 100 حجز → رصيدها = 0
- تُعيد الشحن بـ 500 Rubika

المريض "A" يكسب 50 Rubika (عند إكمال ملفه)
→ يستبدلها بخصم 50 EGP على الكشف
→ الخصم يُضاف إلى محفظة العيادة
```

### 1.3 لماذا الفصل ضروري؟

- **محاسبيًا**: `Program Points` = مصروف تسويقي. `Rubika` = رصيد مدفوع مقدمًا
- **ضريبيًا**: كلاهما يخضع لمعالجة مختلفة
- **قانونيًا**: `Rubika` = عملة داخلية (تحتاج تنظيم). `Points` = مكافأة تجارية
- **تقنيًا**: `Rubika` يحتاج تدقيقًا ماليًا صارمًا. `Points` أكثر مرونة

**لكن**: **نفس المحرك** على مستوى Service Layer (`IPointEngineService`).

---

## 🎯 الجزء 2: طبقة المستفيدين (PointBeneficiaryTypes)

### 2.1 الفلسفة

**6 قيم فقط** — المستوى الأعلى. التفصيل يأتي من `PointSchemeBeneficiaryTypes`.

```
PointBeneficiaryTypes
├── ORGANIZATION            (مؤسسة — أي نوع)
├── ORG_MEMBER_INTERNAL     (عضو داخلي — يعمل في الشركة المصممة)
├── ORG_MEMBER_EXTERNAL     (عضو خارجي — يعمل في مؤسسة أخرى)
├── REGULAR_USER            (مستخدم عادي)
├── PATIENT                 (مريض مُسجَّل في برنامج)
└── GENERIC                 (للاستخدامات المستقبلية)
```

### 2.2 البنية

```sql
CREATE TABLE PointBeneficiaryTypes (
    BeneficiaryTypeID INT IDENTITY PRIMARY KEY,
    BeneficiaryTypeCode NVARCHAR(40) NOT NULL UNIQUE,
    NameAr NVARCHAR(200) NOT NULL,
    NameEn NVARCHAR(200) NOT NULL,
    Description NVARCHAR(1000) NULL,
    IsOrganization BIT NOT NULL,
    IsActive BIT NOT NULL DEFAULT 1,
    DisplayOrder INT NOT NULL DEFAULT 0,
    CreatedDate DATETIME2 NOT NULL DEFAULT GETUTCDATE()
);
```

### 2.3 Seed Data

| ID | Code | NameAr | NameEn | IsOrg |
|:---:|:---|:---|:---|:---:|
| 1 | ORGANIZATION | مؤسسة | Organization | ✅ |
| 2 | ORG_MEMBER_INTERNAL | عضو داخلي | Internal Member | ❌ |
| 3 | ORG_MEMBER_EXTERNAL | عضو خارجي | External Member | ❌ |
| 4 | REGULAR_USER | مستخدم عادي | Regular User | ❌ |
| 5 | PATIENT | مريض | Patient | ❌ |
| 6 | GENERIC | عام | Generic | ❌ |

### 2.4 منطق التصميم

**عندما تصمم الشركة نظام نقاط، تسأل:**

```
من أريد أن أكافئه؟
    │
    ├── 🏢 مؤسسة (ORGANIZATION)
    │   └── تُحدَّد نوعها لاحقًا (CLINIC, PHARMACY, LAB, ...)
    │
    └── 👤 فرد
        │
        ├── عضو في مؤسسة
        │   │
        │   ├── داخلي (ORG_MEMBER_INTERNAL) — يعمل في الشركة المصممة
        │   └── خارجي (ORG_MEMBER_EXTERNAL) — يعمل في مؤسسة أخرى
        │
        ├── مستخدم عادي (REGULAR_USER) — مريض محتمل
        │
        └── مريض مُسجَّل (PATIENT) — مُسجَّل في برنامج
```

### 2.5 جدول M:N (الربط مع Schemes)

```sql
CREATE TABLE PointSchemeBeneficiaryTypes (
    Id INT IDENTITY PRIMARY KEY,
    SchemeID INT NOT NULL 
        FOREIGN KEY REFERENCES PSPProgramPointSchemes(SchemeID) ON DELETE CASCADE,
    BeneficiaryTypeID INT NOT NULL 
        FOREIGN KEY REFERENCES PointBeneficiaryTypes(BeneficiaryTypeID),
    
    -- تفصيل اختياري
    OrganizationTypeID INT NULL 
        FOREIGN KEY REFERENCES OrganizationTypes(OrganizationTypeID),
    MembershipType NVARCHAR(40) NULL,
    
    CreatedDate DATETIME2 NOT NULL DEFAULT GETUTCDATE()
);

CREATE UNIQUE INDEX IX_SchemeBeneficiaryTypes_Unique 
ON PointSchemeBeneficiaryTypes(
    SchemeID, 
    BeneficiaryTypeID, 
    ISNULL(OrganizationTypeID, 0), 
    ISNULL(MembershipType, '')
);
```

### 2.6 أمثلة تطبيقية

| السيناريو | BeneficiaryTypeID | OrganizationTypeID | MembershipType |
|:---|:---|:---|:---|
| مكافأة العيادات | ORGANIZATION | CLINIC | NULL |
| مكافأة الأطباء الداخليين | ORG_MEMBER_INTERNAL | PHARMA_COMPANY | DOCTOR |
| مكافأة مندوبي الصيدليات | ORG_MEMBER_EXTERNAL | PHARMACY | REP |
| مكافأة المرضى | PATIENT | NULL | NULL |
| مكافأة المستخدمين العاديين | REGULAR_USER | NULL | NULL |

**الفائدة**: إضافة نوع مؤسسة جديدة (مثل `WAREHOUSE`) = **INSERT في `OrganizationTypes`** — لا Migration.

---

## 🎯 الجزء 3: مجموعات المؤسسات (OrganizationGroups)

### 3.1 الفلسفة

**مجموعة مؤسسات** = كيان واحد يضم عدة مؤسسات (متجانسة أو غير متجانسة).

**أمثلة:**
- **مستشفى** = عيادات + صيدلية + معمل + غرفة عمليات
- **سلسلة صيدليات** = صيدلية 1 + صيدلية 2 + صيدلية 3
- **Polyclinic** = عيادة باطنة + عيادة أطفال + معمل
- **مجموعة مستشفيات** = مستشفى القاهرة + مستشفى الجيزة

### 3.2 البنية المُبسَّطة

**جدولان فقط** — بلا `GroupTypes` معقدة.

```sql
CREATE TABLE OrganizationGroups (
    GroupID INT IDENTITY PRIMARY KEY,
    GroupCode NVARCHAR(100) NOT NULL UNIQUE,
    GroupNameAr NVARCHAR(400) NOT NULL,
    GroupNameEn NVARCHAR(400) NULL,
    
    Description NVARCHAR(2000) NULL,
    
    -- للتداخل (مجموعة داخل مجموعة)
    ParentGroupID INT NULL 
        FOREIGN KEY REFERENCES OrganizationGroups(GroupID),
    
    -- المالك
    OwnerUserProfileID INT NULL 
        FOREIGN KEY REFERENCES UserProfiles(UserProfileID),
    AdminOrganizationID INT NULL 
        FOREIGN KEY REFERENCES Organizations(OrganizationID),
    
    LogoUrl NVARCHAR(500) NULL,
    IsActive BIT NOT NULL DEFAULT 1,
    CreatedDate DATETIME2 NOT NULL DEFAULT GETUTCDATE(),
    LastModifiedDate DATETIME2 NULL
);

CREATE TABLE OrganizationGroupMembers (
    Id INT IDENTITY PRIMARY KEY,
    GroupID INT NOT NULL 
        FOREIGN KEY REFERENCES OrganizationGroups(GroupID) ON DELETE CASCADE,
    OrganizationID INT NOT NULL 
        FOREIGN KEY REFERENCES Organizations(OrganizationID),
    
    MembershipRole NVARCHAR(40) NULL,
    IsPrimary BIT NOT NULL DEFAULT 0,
    
    JoinedDate DATETIME2 NOT NULL DEFAULT GETUTCDATE(),
    LeftDate DATETIME2 NULL,
    IsActive BIT NOT NULL DEFAULT 1
);

CREATE UNIQUE INDEX IX_OrgGroupMembers_Unique 
ON OrganizationGroupMembers(GroupID, OrganizationID);
CREATE INDEX IX_OrgGroupMembers_Org 
ON OrganizationGroupMembers(OrganizationID);
```

### 3.3 أمثلة الاستخدام

**مثال 1: مستشفى متكامل**
```
Group: "مستشفى الأمل"
├── عيادة قلب
├── عيادة باطنة
├── صيدلية داخلية
├── معمل تحاليل
└── غرفة عمليات
```

**مثال 2: مجموعة مستشفيات (تداخل)**
```
Group: "مجموعة الشفاء الطبية"
├── Group: "مستشفى الشفاء - القاهرة"
│   ├── عيادة
│   ├── صيدلية
│   └── معمل
├── Group: "مستشفى الشفاء - الجيزة"
│   ├── عيادة
│   └── معمل
└── Group: "مستشفى الشفاء - الإسكندرية"
    └── عيادة
```

**التداخل يعمل بشكل طبيعي عبر `ParentGroupID`.**

### 3.4 الفوائد

| الميزة | التفسير |
|--------|---------|
| **مرونة كاملة** | أي عدد، أي نوع |
| **متداخل** | `ParentGroupID` |
| **مجموعة = كيان** | للمحاسبة، التقارير، النقاط |
| **لا Migration** | إضافة مجموعة = INSERT |
| **متوافق مع `PointBeneficiaryTypes`** | المجموعة مستفيد عبر `ORGANIZATION` |

---

## 🎯 الجزء 4: Rubika Currency Engine

### 4.1 البنية الحالية (موجودة بالفعل)

**3 جداول ناضجة:**

```sql
-- ═══ RubikaWallet ═══
CREATE TABLE RubikaWallet (
    WalletID INT IDENTITY PRIMARY KEY,
    UserProfileID INT NULL,              -- لمستخدم
    OrganizationID INT NULL,              -- أو لمؤسسة
    WalletType NVARCHAR(40) NOT NULL,     -- USER, ORGANIZATION, SYSTEM
    Balance DECIMAL(18,2) NOT NULL DEFAULT 0,
    Currency NVARCHAR(6) NOT NULL DEFAULT 'RUB',
    IsActive BIT NOT NULL DEFAULT 1,
    IsLocked BIT NOT NULL DEFAULT 0,
    LockReason NVARCHAR(500) NULL,
    CreatedDate DATETIME2 NOT NULL DEFAULT GETUTCDATE(),
    LastUpdatedDate DATETIME2 NOT NULL DEFAULT GETUTCDATE(),
    PointsType NVARCHAR(40) NOT NULL,     -- RUBIKA, PROMO, ...
    EquivalentEGP DECIMAL(18,2) NULL,
    ExchangeRate DECIMAL(10,4) NULL
);

-- ═══ RubikaTransactions ═══
CREATE TABLE RubikaTransactions (
    TransactionID INT IDENTITY PRIMARY KEY,
    TransactionReference NVARCHAR(100) NOT NULL UNIQUE,
    FromWalletID INT NOT NULL,
    ToWalletID INT NOT NULL,
    ServiceProductID INT NULL,
    OrganizationID INT NULL,
    AmountInEGP DECIMAL(18,2) NOT NULL,
    AmountInRubika DECIMAL(18,2) NOT NULL,
    ExchangeRate DECIMAL(10,4) NOT NULL,
    TransactionType NVARCHAR(40) NOT NULL,  -- TOPUP, CONSUME, TRANSFER, REFUND
    TransactionStatus NVARCHAR(40) NOT NULL, -- PENDING, COMPLETED, FAILED
    Description NVARCHAR(1000) NULL,
    PaymentGatewayReference NVARCHAR(200) NULL,
    IsAutomated BIT NOT NULL DEFAULT 0,
    CreatedDate DATETIME2 NOT NULL DEFAULT GETUTCDATE(),
    CompletedDate DATETIME2 NULL
);

-- ═══ RubikaValueSettings ═══
CREATE TABLE RubikaValueSettings (
    SettingID INT IDENTITY PRIMARY KEY,
    SettingKey NVARCHAR(100) NOT NULL UNIQUE,
    SettingValue NVARCHAR(500) NOT NULL,
    SettingDescription NVARCHAR(1000) NULL,
    DataType NVARCHAR(40) NOT NULL,       -- DECIMAL, STRING, BOOLEAN
    IsActive BIT NOT NULL DEFAULT 1,
    LastModifiedBy INT NULL,
    LastModifiedDate DATETIME2 NULL,
    Category NVARCHAR(100) NULL,
    DisplayOrder INT NULL,
    ValidFrom DATETIME2 NULL,
    ValidTo DATETIME2 NULL
);
```

### 4.2 الاستخدامات

| # | الاستخدام | الوصف |
|:---:|:---|:---|
| **1** | **شحن رصيد** | العيادة/الصيدلية تشحن محفظتها بـ Rubika |
| **2** | **استهلاك خدمة** | كل حجز/طلب يخصم Rubika من الرصيد |
| **3** | **مكافأة المنصة** | المنصة تمنح Rubika مقابل سلوك مرغوب |
| **4** | **تحويل بين محافظ** | من محفظة المنصة → محفظة المستخدم |
| **5** | **خصم عند الدفع** | المريض يستبدل Rubika بخصم على الكشف |

### 4.3 التكامل مع Program Points

**الفكرة**: **Service Layer مشترك** بين النظامين.

```csharp
public interface IPointEngineService
{
    Task<EarnResult> EarnAsync(EarnContext context);
    Task<RedeemResult> RedeemAsync(RedeemContext context);
    Task<BalanceResult> GetBalanceAsync(BalanceQuery query);
    Task<List<TransactionDto>> GetTransactionsAsync(TransactionQuery query);
}

// تنفيذ لـ Program Points
public class ProgramPointsEngine : IPointEngineService { ... }

// تنفيذ لـ Rubika Points
public class RubikaPointsEngine : IPointEngineService { ... }
```

**الفائدة**:
- نفس الواجهة (`IPointEngineService`)
- تنفيذ مختلف لكل نظام
- **الاستخدام** من نفس المكان (`PointDashboard`)

---

## 🎯 الجزء 5: خطة التوسع المستقبلية

### 5.1 الرؤية (3-5 سنوات)

```
┌─────────────────────────────────────────────────────────────┐
│  RubikCare Platform — Digital Health Infrastructure         │
│                                                              │
│  Phase 1: PSP (الحالي)                                       │
│  Phase 2: B2B Supply Chain + Medical CRM                     │
│  Phase 3: B2C + Labs + Pharmacy Network                      │
│  Phase 4: Medical Tourism + Hotels + Airlines                │
│  Phase 5: Healthcare Professional Network + University       │
└─────────────────────────────────────────────────────────────┘
```

### 5.2 الخدمات القادمة + كيفية ربطها بنظام النقاط

| # | الخدمة | PointSystemType | Beneficiary Types |
|:---:|:---|:---|:---|
| **1** | **B2B Supply Chain** | `B2B_SUPPLY` | ORGANIZATION (Warehouse, Pharmacy) |
| **2** | **B2C Store** | `B2C_PHARMACY` | PATIENT, REGULAR_USER |
| **3** | **Medical CRM** | `PLATFORM_MKT` | ORG_MEMBER_INTERNAL (Rep) |
| **4** | **Labs Integration** | `PSP` (موسّع) | ORGANIZATION (Lab) |
| **5** | **Medical Tourism** | `PLATFORM_MKT` (جديد) | ORGANIZATION (Hospital, Hotel, Airline) |
| **6** | **Professional Network** | `PLATFORM_MKT` | REGULAR_USER |
| **7** | **University Programs** | `PLATFORM_MKT` (جديد) | REGULAR_USER (Students) |

### 5.3 شبكة التوزيع اللوجستي (Logistics Network)

**الرؤية**:
- 1800 شركة أدوية → 1000 مخزن جملة → مخازن تجزئة → 50,000 صيدلية
- تقسيم إلى 10 Lines × 180 شركة
- كل مخزن جملة يخدم 100 مخزن تجزئة في نطاقه الجغرافي

**الربط بنظام النقاط**:
- `PointSystemType = B2B_SUPPLY`
- `BeneficiaryType = ORGANIZATION` (Warehouse)
- كل عملية توريد ناجحة = نقاط للمخزن والصيدلية

### 5.4 Medical CRM

**الرؤية**: نظام لإدارة مندوبي الدعاية والبيع للأطباء والصيادلة.

**الربط**:
- `PointSystemType = PLATFORM_MKT`
- `BeneficiaryType = ORG_MEMBER_INTERNAL` (Rep)
- كل زيارة ناجحة = نقاط للمندوب

### 5.5 السياحة العلاجية

**الرؤية**: مستشفيات + فنادق + شركات طيران.

**الربط**:
- `PointSystemType = PLATFORM_MKT` (جديد)
- `BeneficiaryType = ORGANIZATION` (Hospital, Hotel, Airline)
- كل حزمة سياحية ناجحة = نقاط لكل طرف

---

## 🎯 الجزء 6: Migration المطلوبة

### 6.1 نظرة عامة

**6 خطوات** — لا أكثر:

| # | المهمة | الأثر |
|:---:|:---|:---|
| **1** | إنشاء `PointBeneficiaryTypes` + Seed (6 قيم) | جديد |
| **2** | إنشاء `PointSchemeBeneficiaryTypes` (M:N) | جديد |
| **3** | إنشاء `OrganizationGroups` + `OrganizationGroupMembers` | جديد |
| **4** | إضافة `BeneficiaryTypeID` FK في 3 جداول | تعديل |
| **5** | تحويل CSV → M:N | بيانات |
| **6** | حذف الأعمدة النصية | تنظيف |

### 6.2 التفاصيل

#### 6.2.1 Step 1: `PointBeneficiaryTypes`

```sql
CREATE TABLE PointBeneficiaryTypes (
    BeneficiaryTypeID INT IDENTITY PRIMARY KEY,
    BeneficiaryTypeCode NVARCHAR(40) NOT NULL UNIQUE,
    NameAr NVARCHAR(200) NOT NULL,
    NameEn NVARCHAR(200) NOT NULL,
    Description NVARCHAR(1000) NULL,
    IsOrganization BIT NOT NULL,
    IsActive BIT NOT NULL DEFAULT 1,
    DisplayOrder INT NOT NULL DEFAULT 0,
    CreatedDate DATETIME2 NOT NULL DEFAULT GETUTCDATE()
);

INSERT INTO PointBeneficiaryTypes VALUES
(N'ORGANIZATION', N'مؤسسة', N'Organization', N'أي مؤسسة — يُحدد نوعها لاحقًا', 1, 1, 1, GETUTCDATE()),
(N'ORG_MEMBER_INTERNAL', N'عضو داخلي', N'Internal Member', N'يعمل في الشركة المصممة للنظام', 0, 1, 2, GETUTCDATE()),
(N'ORG_MEMBER_EXTERNAL', N'عضو خارجي', N'External Member', N'يعمل في مؤسسة أخرى (شريك)', 0, 1, 3, GETUTCDATE()),
(N'REGULAR_USER', N'مستخدم عادي', N'Regular User', N'مستخدم عادي (مريض محتمل)', 0, 1, 4, GETUTCDATE()),
(N'PATIENT', N'مريض', N'Patient', N'مريض مُسجَّل في برنامج', 0, 1, 5, GETUTCDATE()),
(N'GENERIC', N'عام', N'Generic', N'للاستخدامات المستقبلية', 0, 1, 6, GETUTCDATE());
```

#### 6.2.2 Step 2: `PointSchemeBeneficiaryTypes`

```sql
CREATE TABLE PointSchemeBeneficiaryTypes (
    Id INT IDENTITY PRIMARY KEY,
    SchemeID INT NOT NULL 
        FOREIGN KEY REFERENCES PSPProgramPointSchemes(SchemeID) ON DELETE CASCADE,
    BeneficiaryTypeID INT NOT NULL 
        FOREIGN KEY REFERENCES PointBeneficiaryTypes(BeneficiaryTypeID),
    OrganizationTypeID INT NULL 
        FOREIGN KEY REFERENCES OrganizationTypes(OrganizationTypeID),
    MembershipType NVARCHAR(40) NULL,
    CreatedDate DATETIME2 NOT NULL DEFAULT GETUTCDATE()
);

CREATE UNIQUE INDEX IX_SchemeBeneficiaryTypes_Unique 
ON PointSchemeBeneficiaryTypes(
    SchemeID, BeneficiaryTypeID, 
    ISNULL(OrganizationTypeID, 0), 
    ISNULL(MembershipType, '')
);
```

#### 6.2.3 Step 3: `OrganizationGroups`

(كما هو موضح في الجزء 3)

#### 6.2.4 Step 4: إضافة FK في 3 جداول

```sql
ALTER TABLE PSPProgramPointRules
ADD BeneficiaryTypeID INT NULL 
    FOREIGN KEY REFERENCES PointBeneficiaryTypes(BeneficiaryTypeID);

ALTER TABLE PSPProgramPointBalances
ADD BeneficiaryTypeID INT NULL 
    FOREIGN KEY REFERENCES PointBeneficiaryTypes(BeneficiaryTypeID);

ALTER TABLE PSPProgramPointRedeemRequests
ADD BeneficiaryTypeID INT NULL 
    FOREIGN KEY REFERENCES PointBeneficiaryTypes(BeneficiaryTypeID);
```

#### 6.2.5 Step 5: تحويل CSV → M:N

(SQL Script يحلل `"ORG,Patient,Rep"` → 3 صفوف في `PointSchemeBeneficiaryTypes`)

#### 6.2.6 Step 6: حذف الأعمدة النصية

```sql
ALTER TABLE PSPProgramPointRules DROP COLUMN BeneficiaryType;
ALTER TABLE PSPProgramPointBalances DROP COLUMN BeneficiaryType;
ALTER TABLE PSPProgramPointRedeemRequests DROP COLUMN BeneficiaryType;
ALTER TABLE PSPProgramPointSchemes DROP COLUMN TargetBeneficiaryTypes;
ALTER TABLE PointSystemTypes DROP COLUMN AllowedBeneficiaryTypes;
```

---

## 📊 الجزء 7: ملخص الحالة

### 7.1 ما هو موجود (✅)

| # | المكون | الحالة |
|:---:|:---|:---:|
| 1 | 8 جداول PSP Program Points | ✅ |
| 2 | 19 Handler | ✅ |
| 3 | 5 Controllers (18 Endpoint) | ✅ |
| 4 | `PSPEnrollmentService` | ✅ |
| 5 | ربط 5 أحداث (Invited, Enrolled, eRX, Dispense, Refill) | ✅ |
| 6 | 3 جداول Rubika | ✅ |
| 7 | `SubscriptionPlans` + `PlanServices` + `ServicePricingOptions` | ✅ |

### 7.2 ما يحتاج عملاً (🎯)

| # | المهمة | الأولوية |
|:---:|:---|:---:|
| 1 | Migration (6 خطوات) | 🔴 عالية |
| 2 | `IPointEngineService` (Service Layer مشترك) | 🔴 عالية |
| 3 | ربط `PROGRAM_COMPLETED` | 🟡 متوسطة |
| 4 | ربط `LAB_TEST_UPLOADED` | 🟡 متوسطة |
| 5 | واجهة إدارة `OrganizationGroups` | 🟡 متوسطة |
| 6 | ربط Rubika Points بـ `PointDashboard` | 🔴 عالية |

---

## 🔗 الجزء 8: روابط ذات صلة

- [00 - الهيكل المعماري](00-architecture-overview.md)
- [07 - نظام PSP](07-psp-system.md)
- [08 - خطة التطوير](08-roadmap.md)
- [15 - نظام التحكم المركزي للقوائم](15-multi-layer-menu-control.md)
- [27 - الديون التقنية](27-technical-debt.md)

---

## 📝 الجزء 9: سجل التغييرات

| الإصدار | التاريخ | التغييرات |
|---------|---------|-----------|
| 1.0 | 29 سبتمبر 2026 | الإصدار الأولي — Backend |
| 2.0 | 3 أكتوبر 2026 | إضافة التفاصيل الكاملة |
| **3.0** | **8 أكتوبر 2026** | **تحديث شامل: الرؤية الكاملة + Beneficiary Types + Organization Groups + Rubika Currency + خطة التوسع** |

---

**© 2026 RubikCare — للاستخدام الداخلي**

**نهاية الوثيقة 🚀**
