# 26 — نظام النقاط وروبيكا (Rubika Points & Currency System)

**آخر تحديث:** 9 أكتوبر 2026
**الإصدار:** 4.0 (تحديث ما بعد Stage 3 — BeneficiaryTypeID FK + OrganizationGroups مكتملة)
**الحالة:** ✅ Backend مكتمل — Stage 1-3 منفّذة — UI قيد الانتظار
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
- 🆔 **BeneficiaryTypeID (FK)** — بديل نهائي عن النص — لا Magic Strings

---

## 🗺️ الخريطة المعمارية

### المستويات الأربعة

```
┌─────────────────────────────────────────────────────────────────┐
│  Layer 1: Reference Layer (ديناميكي بالكامل)                     │
│  ├── OrganizationTypes (أنواع المؤسسات — قابل للتوسع)            │
│  ├── PointSystemTypes (أنواع أنظمة النقاط — 5 قيم)               │
│  ├── PointActivityTypes (أنواع الأحداث — 7 قيم)                  │
│  ├── PointBeneficiaryTypes (أنواع المستفيدين — 6 قيم) ✅         │
│  └── MembershipTypes (أنواع العضويات — قابل للتوسع)              │
└─────────────────────────────────────────────────────────────────┘
                              ▲
                              │
┌─────────────────────────────────────────────────────────────────┐
│  Layer 2: Beneficiary Layer (ديناميكي)                           │
│  ├── PointBeneficiaryTypes (6 قيم) ✅                            │
│  ├── PointSchemeBeneficiaryTypes (M:N) ✅                        │
│  └── PointSystemTypeBeneficiaryTypes (M:N) ✅                    │
└─────────────────────────────────────────────────────────────────┘
                              ▲
                              │
┌─────────────────────────────────────────────────────────────────┐
│  Layer 3: Organization Structure                                 │
│  ├── Organizations (موجود)                                       │
│  ├── OrganizationGroups (جديد — ✅ منفّذ)                        │
│  ├── OrganizationGroupMembers (جديد — ✅ منفّذ)                  │
│  ├── UserProfiles (موجود — كل إنسان)                             │
│  └── OrgMemberships (موجود — عضوية الشخص في مؤسسة)               │
└─────────────────────────────────────────────────────────────────┘
                              ▲
                              │
┌─────────────────────────────────────────────────────────────────┐
│  Layer 4: Value Engines (محركان — منفصلان في البيانات)           │
│  ├── Program Points Engine (مكافآت الشركات)                      │
│  │   ├── PSPProgramPointSchemes ✅                                │
│  │   ├── PSPProgramPointRules ✅                                  │
│  │   ├── PSPProgramPointBalances ✅                               │
│  │   ├── PSPProgramPointTransactions ✅                           │
│  │   └── PSPProgramPointRedeemRequests ✅                         │
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

### 2.4 Constants (`PointBeneficiaryTypeIds`)

**الموقع:** `RubikCare.Domain/Constants/PSP/Points/PointBeneficiaryTypeIds.cs`

```csharp
namespace RubikCare.Domain.Constants.PSP.Points;

/// <summary>
/// ثوابت أنواع المستفيدين من نظام النقاط
/// 
/// ⚠️ هذه القيم ثابتة — مُعرَّفة في Seed Data (PointBeneficiaryTypes)
/// ⚠️ لا تُعدّل هذه القيم دون Migration
/// </summary>
public static class PointBeneficiaryTypeIds
{
    public const int Organization = 1;
    public const int OrgMemberInternal = 2;
    public const int OrgMemberExternal = 3;
    public const int RegularUser = 4;
    public const int Patient = 5;
    public const int Generic = 6;
}
```

**الاستخدام:**
```csharp
// بدلاً من "ORG" (نصي) — نستخدم:
BeneficiaryTypeID = PointBeneficiaryTypeIds.Organization
```

### 2.5 جدول M:N (الربط مع Schemes)

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
    SchemeID, 
    BeneficiaryTypeID, 
    ISNULL(OrganizationTypeID, 0), 
    ISNULL(MembershipType, '')
);
```

### 2.6 جدول M:N (الربط مع PointSystemTypes) ✅ جديد

**يحدد أنواع المستفيدين المسموحين لكل نظام نقاط (PSP, B2B, B2C, ...):**

```sql
CREATE TABLE PointSystemTypeBeneficiaryTypes (
    Id INT IDENTITY PRIMARY KEY,
    PointSystemTypeID INT NOT NULL 
        FOREIGN KEY REFERENCES PointSystemTypes(PointSystemTypeID) ON DELETE CASCADE,
    BeneficiaryTypeID INT NOT NULL 
        FOREIGN KEY REFERENCES PointBeneficiaryTypes(BeneficiaryTypeID),
    CreatedDate DATETIME2 NOT NULL DEFAULT GETUTCDATE()
);

CREATE UNIQUE INDEX IX_PointSystemTypeBeneficiaryTypes_Unique 
ON PointSystemTypeBeneficiaryTypes(PointSystemTypeID, BeneficiaryTypeID);
```

### 2.7 Seed Data — `PointSystemTypeBeneficiaryTypes` (12 صف)

| # | PointSystemTypeID | BeneficiaryTypeID |
|:---:|:---:|:---:|
| **PSP** (1) | 1 | 1 (Organization) |
| | 1 | 2 (OrgMemberInternal) |
| | 1 | 5 (Patient) |
| **B2B_SUPPLY** (2) | 2 | 1 (Organization) |
| **B2C_PHARMACY** (3) | 3 | 4 (RegularUser) |
| | 3 | 5 (Patient) |
| **PLATFORM_MKT** (4) | 4 | 1 (Organization) |
| | 4 | 2 (OrgMemberInternal) |
| | 4 | 5 (Patient) |
| | 4 | 4 (RegularUser) |
| **RUBIKA** (5) | 5 | 4 (RegularUser) |
| | 5 | 1 (Organization) |

### 2.8 أمثلة تطبيقية

| السيناريو | BeneficiaryTypeID | OrganizationTypeID | MembershipType |
|:---|:---|:---|:---|
| مكافأة العيادات | ORGANIZATION (1) | CLINIC | NULL |
| مكافأة الأطباء الداخليين | ORG_MEMBER_INTERNAL (2) | PHARMA_COMPANY | DOCTOR |
| مكافأة مندوبي الصيدليات | ORG_MEMBER_EXTERNAL (3) | PHARMACY | REP |
| مكافأة المرضى | PATIENT (5) | NULL | NULL |
| مكافأة المستخدمين العاديين | REGULAR_USER (4) | NULL | NULL |

---

## 🎯 الجزء 3: مجموعات المؤسسات (OrganizationGroups) ✅ منفّذ

### 3.1 الفلسفة

**مجموعة مؤسسات** = كيان واحد يضم عدة مؤسسات (متجانسة أو غير متجانسة).

**أمثلة:**
- **مستشفى** = عيادات + صيدلية + معمل + غرفة عمليات
- **سلسلة صيدليات** = صيدلية 1 + صيدلية 2 + صيدلية 3
- **Polyclinic** = عيادة باطنة + عيادة أطفال + معمل
- **مجموعة مستشفيات** = مستشفى القاهرة + مستشفى الجيزة

### 3.2 البنية

**جدولان فقط** — بلا `GroupTypes` معقدة.

```sql
CREATE TABLE OrganizationGroups (
    GroupID INT IDENTITY PRIMARY KEY,
    GroupCode NVARCHAR(100) NOT NULL UNIQUE,
    GroupNameAr NVARCHAR(400) NOT NULL,
    GroupNameEn NVARCHAR(400) NULL,
    Description NVARCHAR(2000) NULL,
    
    ParentGroupID INT NULL 
        FOREIGN KEY REFERENCES OrganizationGroups(GroupID),
    
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

CREATE UNIQUE INDEX IX_OrganizationGroupMembers_Unique 
ON OrganizationGroupMembers(GroupID, OrganizationID);
CREATE INDEX IX_OrganizationGroupMembers_Org 
ON OrganizationGroupMembers(OrganizationID);
```

### 3.3 الفوائد

| الميزة | التفسير |
|--------|---------|
| **مرونة كاملة** | أي عدد، أي نوع |
| **متداخل** | `ParentGroupID` |
| **مجموعة = كيان** | للمحاسبة، التقارير، النقاط |
| **لا Migration** | إضافة مجموعة = INSERT |
| **متوافق مع `PointBeneficiaryTypes`** | المجموعة مستفيد عبر `ORGANIZATION` |

---

## 🎯 الجزء 4: Rubika Currency Engine

### 4.1 البنية (موجودة بالفعل — 3 جداول)

```sql
-- ═══ RubikaWallet ═══
CREATE TABLE RubikaWallet (
    WalletID INT IDENTITY PRIMARY KEY,
    UserProfileID INT NULL,
    OrganizationID INT NULL,
    WalletType NVARCHAR(40) NOT NULL,
    Balance DECIMAL(18,2) NOT NULL DEFAULT 0,
    Currency NVARCHAR(6) NOT NULL DEFAULT 'RUB',
    IsActive BIT NOT NULL DEFAULT 1,
    IsLocked BIT NOT NULL DEFAULT 0,
    LockReason NVARCHAR(500) NULL,
    CreatedDate DATETIME2 NOT NULL DEFAULT GETUTCDATE(),
    LastUpdatedDate DATETIME2 NOT NULL DEFAULT GETUTCDATE(),
    PointsType NVARCHAR(40) NOT NULL,
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
    TransactionType NVARCHAR(40) NOT NULL,
    TransactionStatus NVARCHAR(40) NOT NULL,
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
    DataType NVARCHAR(40) NOT NULL,
    IsActive BIT NOT NULL DEFAULT 1,
    LastModifiedBy INT NULL,
    LastModifiedDate DATETIME2 NULL,
    Category NVARCHAR(100) NULL,
    DisplayOrder INT NULL,
    ValidFrom DATETIME2 NULL,
    ValidTo DATETIME2 NULL
);
```

---

## 🎯 الجزء 5: Migration المُنفَّذة (Stage 1-3) ✅

### 5.1 نظرة عامة — 3 Stages

| Stage | النوع | الطريقة | الحالة |
|:---:|:---|:---|:---:|
| **Stage 1** | Schema | Migration (`AddPointBeneficiaryAndOrganizationGroups`) | ✅ |
| **Stage 2** | Data | SQL Script (`UPDATE` 8 صفوف) | ✅ |
| **Stage 3** | Cleanup | Migration (`RemoveLegacyBeneficiaryColumns`) | ✅ |

### 5.2 Stage 1: Schema Migration

**الاسم:** `20261009151558_AddPointBeneficiaryAndOrganizationGroups`

**ما تم إنشاؤه:**

| # | العنصر | النوع |
|:---:|:---|:---|
| **1** | `PointBeneficiaryTypes` (+ 6 Seed) | جدول جديد |
| **2** | `PointSchemeBeneficiaryTypes` | جدول جديد (M:N) |
| **3** | `PointSystemTypeBeneficiaryTypes` (+ 12 Seed) | جدول جديد (M:N) |
| **4** | `OrganizationGroups` | جدول جديد |
| **5** | `OrganizationGroupMembers` | جدول جديد |
| **6** | `BeneficiaryTypeID` في `PSPProgramPointRules` | عمود جديد (NULL) |
| **7** | `BeneficiaryTypeID` في `PSPProgramPointBalances` | عمود جديد (NULL) |
| **8** | `BeneficiaryTypeID` في `PSPProgramPointRedeemRequests` | عمود جديد (NULL) |
| **9** | 3 FKs نظيفة | FK |
| **10** | 3 Indexes | Index |

### 5.3 Stage 2: Data Migration

**الطريقة:** SQL Script مباشر (وفق قاعدة "البيانات عبر SQL").

**ما تم:**

```sql
-- تحويل "ORG" → BeneficiaryTypeID = 1 (ORGANIZATION)
UPDATE PSPProgramPointRules 
SET BeneficiaryTypeID = 1 
WHERE BeneficiaryType = N'ORG' AND BeneficiaryTypeID IS NULL;

UPDATE PSPProgramPointBalances 
SET BeneficiaryTypeID = 1 
WHERE BeneficiaryType = N'ORG' AND BeneficiaryTypeID IS NULL;

UPDATE PSPProgramPointRedeemRequests 
SET BeneficiaryTypeID = 1 
WHERE BeneficiaryType = N'ORG' AND BeneficiaryTypeID IS NULL;
```

**النتيجة:**
- Rules: 4 صفوف → `BeneficiaryTypeID = 1`
- Balances: 4 صفوف → `BeneficiaryTypeID = 1`
- RedeemRequests: 1 صف → `BeneficiaryTypeID = 1`

### 5.4 Stage 3: Cleanup Migration

**الاسم:** `202610091XXXXX_RemoveLegacyBeneficiaryColumns`

**ما تم:**

#### حذف الأعمدة النصية (5 أعمدة):

| # | الجدول | العمود |
|:---:|:---|:---|
| **1** | `PSPProgramPointRules` | `BeneficiaryType` |
| **2** | `PSPProgramPointBalances` | `BeneficiaryType` |
| **3** | `PSPProgramPointRedeemRequests` | `BeneficiaryType` |
| **4** | `PSPProgramPointSchemes` | `TargetBeneficiaryTypes` |
| **5** | `PointSystemTypes` | `AllowedBeneficiaryTypes` |

#### تحويل `BeneficiaryTypeID` إلى `NOT NULL`:

```sql
ALTER TABLE PSPProgramPointRules 
ALTER COLUMN BeneficiaryTypeID INT NOT NULL;

ALTER TABLE PSPProgramPointBalances 
ALTER COLUMN BeneficiaryTypeID INT NOT NULL;

ALTER TABLE PSPProgramPointRedeemRequests 
ALTER COLUMN BeneficiaryTypeID INT NOT NULL;
```

#### إعادة بناء الفهارس:

```sql
-- حذف الفهارس القديمة
DROP INDEX IX_PSPProgramPointRules_Scheme_Activity_Beneficiary;
DROP INDEX IX_PointBalances_Scheme_Beneficiary;

-- إنشاء الفهارس الجديدة (بـ BeneficiaryTypeID)
CREATE UNIQUE INDEX IX_PSPProgramPointRules_Scheme_Activity_Beneficiary 
ON PSPProgramPointRules(SchemeID, ActivityTypeID, BeneficiaryTypeID);

CREATE UNIQUE INDEX IX_PointBalances_Scheme_Beneficiary 
ON PSPProgramPointBalances(SchemeID, BeneficiaryTypeID, BeneficiaryID);
```

---

## 🎯 الجزء 6: تحديثات Handlers ✅

### 6.1 ما تم تحديثه — 11 Handler

| # | Handler | التعديل |
|:---:|:---|:---|
| **1** | `EarnPointsHandler` | `"ORG"` → `PointBeneficiaryTypeIds.Organization` |
| **2** | `CheckPointsVisibilityHandler` | `b.BeneficiaryType == "ORG"` → `b.BeneficiaryTypeID == 1` |
| **3** | `GetAllBalancesBySchemeHandler` | Select + Filter |
| **4** | `GetBalanceByOrganizationHandler` | Filter |
| **5** | `GetOrganizationPointSummaryHandler` | Filter |
| **6** | `SetPointsVisibilityHandler` | Filter + Create |
| **7** | `GetPointTransactionsHandler` | Filter (Subquery) |
| **8** | `GetSchemeClinicsHandler` | Filter |
| **9** | `GetRedeemRequestsHandler` | Filter (Subquery) |
| **10** | `CreatePointRuleHandler` | `BeneficiaryTypeID = 1` عند الإنشاء |
| **11** | `CreateRedeemRequestHandler` | `BeneficiaryTypeID = balance.BeneficiaryTypeID` |

### 6.2 Handlers نظيفة (لا تحتاج تعديل)

| المجموعة | العدد |
|:---|:---:|
| Schemes Handlers | 4 |
| Rules Handlers (الباقي) | 4 |
| Redeem Actions Handlers (الباقي) | 4 |

---

## 🎯 الجزء 7: الجرد الشامل ✅

### 7.1 ما تم فحصه

| # | الطبقة | النتيجة |
|:---:|:---|:---:|
| **1** | `Api.Web` (Controllers) | ✅ نظيف |
| **2** | `RubikCare.Application` (Commands/DTOs) | ✅ نظيف |
| **3** | `Rubikcare.Web` (Blazor Server) | ✅ نظيف |
| **4** | `Shared.UI` (Components) | ✅ نظيف |
| **5** | `RubikCare.PWA` | ✅ نظيف |
| **6** | `RubikCare.Tests` | ✅ نظيف |
| **7** | `Mobile` (MAUI) | ✅ نظيف |

### 7.2 الاستنتاج

**✅ لا يوجد أي استخدام لـ `BeneficiaryType` (النصي) خارج الـ 11 Handler التي تم تحديثها.**

**النتيجة:** Stage 3 آمن 100%.

---

## 🎯 الجزء 8: خطة التوسع المستقبلية

### 8.1 الرؤية (3-5 سنوات)

```
Phase 1: PSP (الحالي) ✅
Phase 2: B2B Supply Chain + Medical CRM
Phase 3: B2C + Labs + Pharmacy Network
Phase 4: Medical Tourism + Hotels + Airlines
Phase 5: Healthcare Professional Network + University
```

### 8.2 الخدمات القادمة + كيفية ربطها

| # | الخدمة | PointSystemType | Beneficiary Types |
|:---:|:---|:---|:---|
| **1** | B2B Supply Chain | `B2B_SUPPLY` | ORGANIZATION (Warehouse, Pharmacy) |
| **2** | B2C Store | `B2C_PHARMACY` | PATIENT, REGULAR_USER |
| **3** | Medical CRM | `PLATFORM_MKT` | ORG_MEMBER_INTERNAL (Rep) |
| **4** | Labs Integration | `PSP` (موسّع) | ORGANIZATION (Lab) |
| **5** | Medical Tourism | `PLATFORM_MKT` (جديد) | ORGANIZATION (Hospital, Hotel, Airline) |
| **6** | Professional Network | `PLATFORM_MKT` | REGULAR_USER |
| **7** | University Programs | `PLATFORM_MKT` (جديد) | REGULAR_USER (Students) |

### 8.3 إضافة خدمة جديدة — الخطوات

| # | الخطوة | الطريقة |
|:---:|:---|:---|
| **1** | إضافة `PointSystemType` جديد | SQL Script (INSERT) |
| **2** | ربط `BeneficiaryTypes` المسموحة | SQL Script (INSERT في M:N) |
| **3** | إنشاء `PointScheme` جديد | عبر Handler موجود |
| **4** | ربط الحملة | عبر Handler موجود |
| **5** | إضافة `Rules` | عبر Handler موجود |

**⚠️ لا Migration جديدة — فقط SQL Scripts.**

---

## 🎯 الجزء 9: خطة المرحلة القادمة 🎯

### 9.1 ما يتبقى للوصول إلى "نظام نقاط كامل"

| # | المهمة | الأولوية | الوقت المتوقع |
|:---:|:---|:---:|:---:|
| **1** | **`IPointEngineService`** (Service Layer مشترك) | 🔴 عالية | 2-3 ساعات |
| **2** | **تحديث `EarnPointsHandler`** لدعم `BeneficiaryTypeID` من Command | 🔴 عالية | 1-2 ساعة |
| **3** | **تحديث `Commands`/`DTOs`** لإضافة `BeneficiaryTypeID` | 🔴 عالية | 1-2 ساعة |
| **4** | **API Endpoints** — دعم `BeneficiaryType` في Requests | 🟠 متوسطة | 1 ساعة |
| **5** | **Frontend (Web)** — UI لاختيار نوع المستفيد | 🔴 عالية | 3-4 ساعات |
| **6** | **Frontend (PWA/MAUI)** — عرض النقاط حسب النوع | 🟠 متوسطة | 2-3 ساعات |
| **7** | **ربط `PROGRAM_COMPLETED`** | 🟡 متوسطة | 1-2 ساعة |
| **8** | **ربط `LAB_TEST_UPLOADED`** | 🟡 متوسطة | 2-3 ساعات |
| **9** | **واجهة إدارة `OrganizationGroups`** | 🟡 متوسطة | 3-4 ساعات |
| **10** | **اختبار شامل** | 🔴 عالية | 2-3 ساعات |

**الإجمالي:** 18-26 ساعة (3-4 أيام).

### 9.2 ما يدعمه النظام الآن

| # | الخدمة | جاهز؟ |
|:---:|:---|:---:|
| **1** | نظام نقاط للعيادات (ORGANIZATION) | ✅ |
| **2** | نظام نقاط للصيادلة (ORG_MEMBER_INTERNAL + PHARMACY) | ✅ (Schema جاهز) |
| **3** | نظام نقاط للمرضى (PATIENT) | ✅ (Schema جاهز) |
| **4** | نظام نقاط للمندوبين (ORG_MEMBER_INTERNAL + REP) | ✅ (Schema جاهز) |
| **5** | نظام نقاط لأي خدمة (B2B, B2C, إلخ) | ✅ (Schema جاهز) |

**⚠️ لكن:** `EarnPointsHandler` لا يزال يدعم `ORGANIZATION` فقط (مُثبَّت).
**التحديث مطلوب** لدعم الأنواع الأخرى.

---

## 📊 الجزء 10: ملخص الحالة

### 10.1 ما هو مكتمل (✅)

| # | المكون | الحالة |
|:---:|:---|:---:|
| 1 | 8 جداول PSP Program Points | ✅ |
| 2 | 5 جداول جديدة (Beneficiary + Groups) | ✅ |
| 3 | 19 Handler (+ 11 مُحدَّث) | ✅ |
| 4 | 5 Controllers | ✅ |
| 5 | `PSPEnrollmentService` | ✅ |
| 6 | 3 جداول Rubika | ✅ |
| 7 | `SubscriptionPlans` + `PlanServices` | ✅ |
| 8 | `PointBeneficiaryTypeIds` Constants | ✅ |
| 9 | Migration Stage 1-3 | ✅ |
| 10 | الجرد الشامل | ✅ |

### 10.2 ما يحتاج عملاً (🎯)

| # | المهمة | الأولوية |
|:---:|:---|:---:|
| 1 | `IPointEngineService` | 🔴 |
| 2 | Frontend UI (اختيار BeneficiaryType) | 🔴 |
| 3 | تحديث `EarnPointsHandler` لدعم أنواع متعددة | 🔴 |
| 4 | ربط `PROGRAM_COMPLETED` | 🟡 |
| 5 | ربط `LAB_TEST_UPLOADED` | 🟡 |
| 6 | واجهة إدارة `OrganizationGroups` | 🟡 |

---

## 🔗 الجزء 11: روابط ذات صلة

- [00 - الهيكل المعماري](00-architecture-overview.md)
- [07 - نظام PSP](07-psp-system.md)
- [08 - خطة التطوير](08-roadmap.md)
- [15 - نظام التحكم المركزي للقوائم](15-multi-layer-menu-control.md)
- [27 - الديون التقنية](27-technical-debt.md)

---

## 📝 الجزء 12: سجل التغييرات

| الإصدار | التاريخ | التغييرات |
|---------|---------|-----------|
| 1.0 | 29 سبتمبر 2026 | الإصدار الأولي — Backend |
| 2.0 | 3 أكتوبر 2026 | إضافة التفاصيل الكاملة |
| 3.0 | 8 أكتوبر 2026 | الرؤية الكاملة + Beneficiary Types + Organization Groups + Rubika Currency |
| **4.0** | **9 أكتوبر 2026** | **Stage 1-3 منفّذة — BeneficiaryTypeID FK + OrganizationGroups + تحديث 11 Handler + الجرد الشامل** |

---

**© 2026 RubikCare — للاستخدام الداخلي**

**نهاية الوثيقة 🚀**
