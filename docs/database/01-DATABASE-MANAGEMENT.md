
# 01— DATABASE MANAGEMENT (إدارة قواعد البيانات)

**الإصدار:** 1.0
**آخر تحديث:** 2026-11-07
**الحالة:** 🟢 حي — يُحدَّث كل جلسة
**الجمهور:** فريق التطوير + DevOps + AI Assistants
**المرجع:** يُقرأ قبل أي Migration أو تغيير في Schema/Seed

---

## 📌 مقدمة

هذه الوثيقة هي المرجع الكامل لكل ما يتعلق بإدارة قواعد البيانات في RubikCare:

- كيف نُطبّق Migrations بأمان.
- كيف نتتبع Seed Data.
- كيف نتعامل مع الطوارئ (Rollback).
- كيف نُدير البيئات (Dev/ProdCopy/Production).
- ما تم إنجازه (2026-11-06/07) + ما هو مخطط.

**المرجع المرتبط:** `15-DEPLOYMENT.md` (نشر IIS/Web/API/PWA) — هذه الوثيقة مكملة، لا بديلة.

---

## 🎯 الجزء صفر: الفلسفة والمبادئ

### 0.1 المبادئ الخمسة (إلزامية)

| # | المبدأ | التطبيق |
|---|---|---|
| **1** | **Three-Tier Schema Management** | فصل Schema (Migration) عن Data Migration (SQL) عن Seed (MERGE). |
| **2** | **Idempotency by Default** | كل سكريبت آمن للتشغيل مرتين. |
| **3** | **Pre-Flight Validation** | `Pre-Migration-Check.ps1` يفحص 12 نقطة قبل أي Migration. |
| **4** | **Versioned Seed Scripts** | `V001__*`, `V002__*`, مع تسجيل في `__SeedHistory`. |
| **5** | **Staging Mirror + Automated Diff** | اختبار كل Migration على `RubikCare_ProdCopy` قبل `RubikCare_Production`. |

### 0.2 قاعدة "الاستنتاج من السياق"

**قبل افتراض عطل في البنية التحتية (API, CSS, DB, Services) — راجع الأدلة الفعلية في السياق.**
- إذا كان هناك دليل على عمل شيء، لا تفترض عطلاً فيه.
- عند الشك، اسأل قبل اقتراح حل.
- إذا كان الكود يعمل في سياق ويفشل في آخر، ابحث في الفارق بين السياقين أولاً.

### 0.3 قاعدة "الصريح دائماً" (Explicit Always)

- ❌ **لا اعتماد على `appsettings.json` أو أي ملف إعدادات** عند تطبيق Migrations.
- ✅ **`--connection` صريح** (param `-ConnectionString` إلزامي).
- ✅ **`-DatabaseName` صريح** + تحقق أنه يطابق `Database=` في الـ connection string.

**السبب:** `appsettings.json` قد يُستبدل بملف بيئة غير متوقع. `ASPNETCORE_ENVIRONMENT` فارغ → .NET يعامله كـ `Production`. `appsettings.Production.json` يشير إلى `RubikCare_Production`. `appsettings.Secrets.json` يشير إلى `RubikCare_Dev_V2`. أي خطأ في ترتيب التحميل = Migration على Production بلا قصد.

### 0.4 قاعدة "الحماية المزدوجة"

- ✅ **`-DryRun` افتراضي** (يطبع ما سيفعله، لا ينفّذ).
- ✅ **`-Execute` + `-Confirm`** مطلوبان للتنفيذ الفعلي.
- ✅ **`-IAmSureThisIsProduction`** مطلوب إذا `-DatabaseName` يحتوي "Production".
- ✅ **تأكيد تفاعلي** — كتابة اسم DB يدوياً (لـ Production فقط).

### 0.5 قاعدة "النسخة الاحتياطية الإلزامية"

- ❌ **لا `-Execute` بدون Backup ناجح** (مُتحقَّق منه في `msdb.dbo.backupset`).
- ✅ **`Pre-Migration-Check.ps1`** يتحقق من وجود Backup أقل من 24 ساعة.
- ✅ **`Apply-Migrations.ps1`** يأخذ Backup تلقائياً قبل التطبيق (إذا `-AutoBackup`، افتراضي `$true`).

### 0.6 قاعدة "التحقق المسبق"

- ✅ **`Pre-Migration-Check.ps1`** يفحص 12 نقطة قبل أي Migration.
- ✅ **أي فحص فاشل = توقف كامل** (لا `-Execute` ممكن).

### 0.7 قاعدة "التسجيل الكامل"

- ✅ **كل سكريبت يكتب Log** إلى `C:\WebSite\_Docs\Logs\Database\`.
- ✅ **كل Migration تُسجَّل** في `__EFMigrationsHistory` (EF Core — تلقائي).
- ✅ **كل Seed يُسجَّل** في `__SeedHistory` (جديد — 2026-11-06).

### 0.8 قاعدة "التراجع الجاهز"

- ✅ **كل Migration لها `Down()` كامل** (نتحقق منه عند التوليد).
- ✅ **كل Backup يبقى 30 يوماً على الأقل** قبل أي حذف.

### 0.9 قاعدة "فصل الإنتاج"

- ✅ **`-DatabaseName` = `RubikCare_Production`** → يتطلب `-IAmSureThisIsProduction` + كتابة اسم DB يدوياً.
- ✅ **أي محاولة `RESTORE` على Production** → ترفض من السكريبت (يُطلب SSMS يدوي).

---

## 🗺️ الجزء الأول: البيئات

### 1.1 خريطة قواعد البيانات

| DB | الغرض | مالك | متصلة بـ |
|---|---|---|---|
| `RubikCare_Dev_V2` | التطوير الرئيسي | فريق التطوير | Dev Environment (جهازك + السيرفر) |
| `RubikCare_ProdCopy` | نسخة من Production للاختبار | فريق التطوير | **غير متصلة بأي بيئة** |
| `RubikCare_Production` | الإنتاج | فريق DevOps | Production Environment |
| `RubikCare_Test` | اختبارات | فريق QA | Test Environment |
| `RubikCare_Uat` | اختبار القبول | فريق QA | UAT Environment |
| `RubikCare_Pre` | Pre-Production | فريق DevOps | Pre-Prod Environment |
| `RubikCare_Backup` | نسخ احتياطية | فريق DevOps | غير متصلة |
| `RubikCare_v2` | نسخة قديمة (Legacy) | — | غير متصلة |
| `RubikCare_Master` | Master (Legacy) | — | غير متصلة |

**ملاحظة مهمة:** كل DBs على **نفس SQL Server** (`161.97.105.83\RUBICKCARE`). هذا يُسهّل النسخ الاحتياطي/الاستعادة، لكنه يُصعّب اختبار "بيئة معزولة تماماً". عيب صغير مقابل ميزة كبيرة (تطابق بيئات).

### 1.2 Single SQL Server Architecture

**البنية:**
```
SQL Server Instance: 161.97.105.83\RUBICKCARE
├── RubikCare_Dev_V2         (Development)
├── RubikCare_ProdCopy       (Production Copy — للاختبار)
├── RubikCare_Production     (Production)
├── RubikCare_Test           (Testing)
├── RubikCare_Uat            (UAT)
├── RubikCare_Pre            (Pre-Production)
├── RubikCare_Backup         (Backups)
├── msdb                     (System — backup history)
└── ... (Legacy DBs)
```

**المزايا:**
- ✅ نسخ/استعادة مباشر بين DBs.
- ✅ `BACKUP DATABASE ... WITH MOVE` سهل.
- ✅ `Compare-Databases.ps1` يعمل من نفس الاتصال.

**العيوب:**
- ⚠️ I/O مشترك — Backup كبير على Dev قد يؤثر على Production.
- ⚠️ Backup History مشترك في `msdb.dbo.backupset` — يجب التمييز بـ `database_name`.
- ⚠️ حجم القرص مشترك — Backup لـ Production + Backup لـ Dev = مساحة × 2.

### 1.3 أي DB لأي غرض؟

| العملية | DB المستهدفة | سبب |
|---|---|---|
| **تطوير ميزة جديدة** | `RubikCare_Dev_V2` | Development |
| **اختبار Migration (أول مرة)** | `RubikCare_Dev_V2` | Development |
| **اختبار Migration (ثاني مرة)** | `RubikCare_ProdCopy` | Production-like data |
| **نشر Migration (الإنتاج)** | `RubikCare_Production` | في جلسة مع AI، مع تأكيدات |
| **اختبار Rollback** | `RubikCare_ProdCopy` | Production-like data |
| **اختبار Compare** | `RubikCare_ProdCopy` + `RubikCare_Production` | مقارنة |
| **اختبار Switch-DevToProdCopy** | `RubikCare_Dev_V2` + `RubikCare_ProdCopy` | تبديل |

### 1.4 متى تُنشأ نسخة من Production؟

**السيناريو:** قبل اختبار Migration كبيرة على Production (مثل إعادة هيكلة جدول).

**الخطوات:**
1. **Backup** من `RubikCare_Production` (`WITH COMPRESSION`).
2. **Restore** كـ `RubikCare_ProdCopy` (`WITH MOVE` — لأن الاسم مختلف).
3. **اختبار Migration** على `RubikCare_ProdCopy`.
4. **بعد النجاح** → Migration على `RubikCare_Production`.

**ملاحظة:** `RubikCare_ProdCopy` غير متصلة بأي بيئة — آمنة للاختبار الحر.

---

## 🗄️ الجزء الثاني: الكيانات (Entities)

### 2.1 `__EFMigrationsHistory` (EF Core — موجود)

**الغرض:** تتبع كل Migrations EF Core المطبقة على DB.

**البنية:**
```sql
CREATE TABLE __EFMigrationsHistory (
    MigrationId    NVARCHAR(150) NOT NULL PRIMARY KEY,
    ProductVersion NVARCHAR(32)  NOT NULL
);
```

**ملاحظات:**
- جدول `__` (system table) — لا يظهر في `INFORMATION_SCHEMA.TABLES`، استخدم `sys.tables`.
- يُحدَّث تلقائياً عند كل `dotnet ef database update`.
- مصدر الحقيقة لـ "ما هو مطبق على DB".

**الاستخدام:**
```sql
SELECT TOP 5 MigrationId, ProductVersion 
FROM __EFMigrationsHistory 
ORDER BY MigrationId DESC;
```

### 2.2 `__SeedHistory` (جديد — 2026-11-06)

**الغرض:** تتبع كل Seed Scripts المطبقة على DB (Idempotency).

**البنية:**
```sql
CREATE TABLE __SeedHistory (
    SeedVersion     NVARCHAR(50)  NOT NULL PRIMARY KEY,  -- "V001", "V002", ...
    SeedName        NVARCHAR(200) NOT NULL,             -- "PaymentMethods"
    Description     NVARCHAR(500) NULL,
    FileHash        NVARCHAR(64)  NULL,                  -- SHA256
    AppliedDate     DATETIME2     NOT NULL,              -- UTC
    AppliedBy       NVARCHAR(100) NULL,
    DatabaseName    NVARCHAR(128) NULL,
    ExecutionTimeMs INT           NULL,
    Notes           NVARCHAR(1000) NULL
);
```

**التسجيل:** `Apply-Migrations.ps1` (أو `ISeedHistoryService` من C#).

**الاستخدام:**
```sql
SELECT * FROM __SeedHistory ORDER BY SeedVersion;
-- مثال: V001__PaymentMethods, V002__PointSystemTypeOrganizationTypes_B2B
```

**لماذا `SeedHistory` منفصل عن `EFMigrationsHistory`؟**
- Migrations = تغييرات Schema (EF Core يديرها).
- Seeds = تغييرات Data (مرجعية) — لا علاقة بـ EF Core.
- خلطهما = فقدان التحكم في الـ Idempotency.

### 2.3 `BackupHistory` (موجود)

**الغرض:** تتبع كل Backups المُنشأة من التطبيق (C# `BackupService`).

**البنية:**
```sql
CREATE TABLE BackupHistory (
    BackupID          INT IDENTITY PRIMARY KEY,
    BackupPath        NVARCHAR(500) NOT NULL,
    BackupName        NVARCHAR(200) NOT NULL,
    BackupSizeMB      DECIMAL(18,2) NOT NULL,
    DatabaseName      NVARCHAR(128) NOT NULL,
    CreatedDate       DATETIME2     NOT NULL,
    CreatedByUserId   NVARCHAR(450) NULL,
    Description       NVARCHAR(500) NULL,
    IsActive          BIT           NOT NULL DEFAULT 1,
    IsDeleted         BIT           NOT NULL DEFAULT 0,
    DeletedDate       DATETIME2     NULL,
    DeletedByUserId   NVARCHAR(450) NULL
);
```

**ملاحظة:** `BackupService` (C#) هو الذي يُنشئ السجلات — من داخل التطبيق. **لا يعمل في Production** (يحتاج RDP أو PowerShell).

**الاستخدام:**
```sql
SELECT TOP 10 BackupID, BackupName, BackupSizeMB, CreatedDate
FROM BackupHistory
WHERE DatabaseName = 'RubikCare_Production' AND IsDeleted = 0
ORDER BY CreatedDate DESC;
```

### 2.4 `msdb.dbo.backupset` (System — موجود)

**الغرض:** سجل SQL Server لكل Backups (بما فيها التي أُنشئت خارج التطبيق).

**الاستخدام:**
```sql
SELECT TOP 10 
    database_name, backup_start_date, backup_finish_date,
    CAST(backup_size / 1048576.0 AS DECIMAL(10,2)) AS SizeMB,
    physical_device_name
FROM msdb.dbo.backupset bs
JOIN msdb.dbo.backupmediafamily bmf ON bs.media_set_id = bmf.media_set_id
WHERE database_name LIKE 'RubikCare%'
ORDER BY backup_finish_date DESC;
```

**مهم:** يُستخدم في `Pre-Migration-Check.ps1` للتحقق من "آخر Backup أقل من 24 ساعة".

### 2.5 `Migrations_Exceptions.md` (جديد — 2026-11-06)

**الغرض:** سجل Migration تم حذفها أو تعطيلها، مع السبب والبديل.

**البنية:** Markdown (ليس DB).

**المحتوى الحالي:**
- `20260425124724_UpdateMedicationRequest` — حُذفت (استُبدلت بـ `MakePatientIdNullable`).

**الاستخدام:** عند فحص "لماذا Migration X غير موجودة؟" — راجع هذا الملف أولاً.

### 2.6 جدول مقارن للكيانات

| الكيان | النوع | الموقع | الغرض |
|---|---|---|---|
| `__EFMigrationsHistory` | DB Table | `sys.tables` | Migrations مطبقة |
| `__SeedHistory` | DB Table | `sys.tables` | Seeds مطبقة |
| `BackupHistory` | DB Table | `sys.tables` | Backups (من C#) |
| `msdb.dbo.backupset` | System Table | `msdb` | Backups (كل) |
| `Migrations_Exceptions.md` | Markdown | `Scripts/Database/` | استثناءات |

---

## 🔧 الجزء الثالث: السكريبتات (ما تم + ما هو مخطط)

### 3.1 `Pre-Migration-Check.ps1` ✅ (v2.1 — 2026-11-06)

**الغرض:** 12 فحصاً أمانياً قبل أي Migration (قراءة فقط).

**المسار:**
- **جهازك:** `C:\RC\Rubikcare.Full.Migration\Scripts\Database\_Infrastructure\Pre-Migration-Check.ps1`
- **السيرفر:** `C:\WebSite\_Docs\Scripts\database\_Infrastructure\Pre-Migration-Check.ps1`

**الفحوصات الـ 12:**

| # | الفحص | ماذا يكشف؟ |
|---|---|---|
| 1 | الاتصال بـ SQL Server | DB موجودة + ONLINE |
| 2 | صلاحيات المستخدم | sysadmin أو dbcreator |
| 3 | آخر Backup حديث | Backup أقل من 24 ساعة |
| 4 | لا Pending Model Changes | الكود و Snapshot متزامنان |
| 5 | مطابقة Migrations | DB ↔ الكود |
| 6 | لا Orphans | PSPeRX → PSPPatients |
| 7 | حجم DB | < 10 GB |
| 8 | عدد الاتصالات | < 10 |
| 9 | Applied Migrations | من `__EFMigrationsHistory` |
| 10 | Available Migrations | من الكود |
| 11 | Pending Migrations | الفرق — يطبع الأسماء |
| 12 | Seeds | Applied + Pending |

**الاستخدام:**
```powershell
cd C:\RC\Rubikcare.Full.Migration\Scripts\Database\_Infrastructure
powershell -ExecutionPolicy Bypass -File ".\Pre-Migration-Check.ps1" `
    -DatabaseName "RubikCare_ProdCopy" `
    -Password "Rubikans@123"
```

**النتيجة:**
- `exit 0` — كل شيء سليم.
- `exit 1` — يوجد فشل، راجع الـ Log.

**الـ Log:** `C:\WebSite\_Docs\Logs\Database\PreMigrationCheck_<DB>_<timestamp>.log`.

**ما تم إنجازه:**
- ✅ v2.0 (8 فحوصات) → v2.1 (12 فحصاً + عرض Pending).
- ✅ اختُبر على `RubikCare_ProdCopy` و`RubikCare_Production` — 12/12 نجح.

### 3.2 `Apply-Migrations.ps1` ✅ (v1.0 — 2026-11-06)

**الغرض:** تطبيق Migrations بأمان، واحدة تلو الأخرى.

**المسار:** نفسه (بجانب `Pre-Migration-Check.ps1`).

**الميزات:**
- `-DryRun` افتراضي (لا ينفّذ).
- `-Execute` + `-Confirm` مطلوبان للتنفيذ.
- `-IAmSureThisIsProduction` مطلوب لـ `RubikCare_Production`.
- تأكيد تفاعلي (كتابة اسم DB يدوياً).
- `-AutoBackup` (افتراضي `$true`).
- `-AutoPreCheck` (افتراضي `$true`).
- `-OneByOne` (افتراضي `$true`).
- `-TargetMigration` (اختياري — Migration محددة).
- `-SkipAutoBackup` / `-SkipPreCheck` (للحالات الحرجة).

**الاستخدام — DryRun:**
```powershell
.\Apply-Migrations.ps1 -DatabaseName "RubikCare_ProdCopy" -Password "Rubikans@123"
```

**الاستخدام — Execute على Dev:**
```powershell
.\Apply-Migrations.ps1 -DatabaseName "RubikCare_Dev_V2" -Password "Rubikans@123" -Execute -Confirm
```

**الاستخدام — Execute على ProdCopy (Migration محددة):**
```powershell
.\Apply-Migrations.ps1 -DatabaseName "RubikCare_ProdCopy" -Password "Rubikans@123" `
    -TargetMigration "20261106171444_AddSeedHistory" `
    -Execute -Confirm
```

**الاستخدام — Execute على Production:**
```powershell
.\Apply-Migrations.ps1 -DatabaseName "RubikCare_Production" -Password "Rubikans@123" `
    -Execute -Confirm -IAmSureThisIsProduction
# ثم اكتب: RubikCare_Production (للتأكيد)
```

**ما تم إنجازه:**
- ✅ v1.0 — 7 خطوات (PreCheck → قراءة Pending → DryRun/Execute → تأكيد Production → Backup → تطبيق → تحقق).
- ✅ اختُبر DryRun على `RubikCare_ProdCopy` — "لا توجد Migrations معلقة".
- ⏳ **Execute لم يُختبر بعد** — سيكون مع أول Migration B2B.

### 3.3 `Backup-Before-Deploy.ps1` ⏳ (مخطط)

**الغرض:** نسخة احتياطية شاملة قبل أي نشر.

**المحتوى المخطط:**
- Full Backup لـ DB (`.bak`).
- نسخة من `appsettings.json` + `web.config` لكل موقع.
- نسخة من `__EFMigrationsHistory` (`.csv`).
- نسخة من `__SeedHistory` (`.csv`).
- نسخة من `SharedStorage` (اختياري).

**المتى؟** قبل أي نشر على Production.

### 3.4 `Restore-Database.ps1` ⏳ (مخطط)

**الغرض:** استعادة DB من Backup — خارج IIS.

**المحتوى المخطط:**
- إيقاف App Pool.
- `RESTORE DATABASE` من Backup.
- إعادة تشغيل App Pool.
- التحقق.
- **رفض الاستعادة على `RubikCare_Production`** (يُطلب SSMS يدوي).

**المتى؟** عند Rollback أو اختبار.

### 3.5 `Compare-Databases.ps1` ⏳ (مخطط)

**الغرض:** مقارنة Schema + Seed Data بين DBs.

**المحتوى المخطط:**
- `Dev_V2` ↔ `ProdCopy` ↔ `Production`.
- Migration History.
- Seed History.
- عدد الجداول/الأعمدة.
- Orphans.

**المتى؟** عند فحص "هل البيئات متطابقة؟".

### 3.6 `Switch-DevToProdCopy.ps1` ⏳ (مخطط)

**الغرض:** استعادة `ProdCopy` كـ `Dev_V2` (اختبار على بيانات Production).

**المحتوى المخطط:**
- Backup `Dev_V2` الحالي.
- `RESTORE` `ProdCopy` as `Dev_V2` (`WITH MOVE`).
- التحقق.

**المتى؟** عند اختبار Migration كبيرة على بيانات Production.

### 3.7 جدول مقارن للسكريبتات

| السكريبت | الحالة | الاستخدام | الموقع |
|---|---|---|---|
| `Pre-Migration-Check.ps1` | ✅ v2.1 | قبل أي Migration | `_Infrastructure/` |
| `Apply-Migrations.ps1` | ✅ v1.0 | تطبيق Migrations | `_Infrastructure/` |
| `Backup-Before-Deploy.ps1` | ⏳ | قبل النشر | `_Infrastructure/` |
| `Restore-Database.ps1` | ⏳ | Rollback | `_Infrastructure/` |
| `Compare-Databases.ps1` | ⏳ | فحص التطابق | `_Infrastructure/` |
| `Switch-DevToProdCopy.ps1` | ⏳ | اختبار على Prod data | `_Infrastructure/` |

---

## 🎬 الجزء الرابع: سيناريوهات الاستخدام

### 4.1 سيناريو 1: إضافة Migration جديدة (B2B SupplyOrder)

**السياق:** مطوّر أضاف كيانات جديدة (`PharmacySupplyOrder`, `PharmacySupplyOrderItem`).

**الخطوات:**

**الخطوة 1 — إنشاء Migration:**
```powershell
cd C:\RC\Rubikcare.Full.Migration
dotnet ef migrations add AddPharmacySupplyOrder `
    --project RubikCare.Infrastructure `
    --startup-project Api.Web
```

**الخطوة 2 — مراجعة Migration المولّد:**
- افتح `RubikCare.Infrastructure/Migrations/<timestamp>_AddPharmacySupplyOrder.cs`.
- تحقق من: `Up()` صحيح، `Down()` كامل، لا `Sql()` غير متوقع.

**الخطوة 3 — Pre-Check على Dev:**
```powershell
cd Scripts\Database\_Infrastructure
powershell -ExecutionPolicy Bypass -File ".\Pre-Migration-Check.ps1" `
    -DatabaseName "RubikCare_Dev_V2" -Password "Rubikans@123"
```
**النتيجة المتوقعة:** 12/12 نجح، Pending = 1 (هذه Migration).

**الخطوة 4 — DryRun على Dev:**
```powershell
.\Apply-Migrations.ps1 -DatabaseName "RubikCare_Dev_V2" -Password "Rubikans@123"
```
**النتيجة المتوقعة:** "سيتم تطبيق: <timestamp>_AddPharmacySupplyOrder".

**الخطوة 5 — Execute على Dev:**
```powershell
.\Apply-Migrations.ps1 -DatabaseName "RubikCare_Dev_V2" -Password "Rubikans@123" -Execute -Confirm
```
**ما سيحدث:** PreCheck → Backup → تطبيق → تحقق.

**الخطوة 6 — اختبار الميزة على Dev** (من التطبيق).

**الخطوة 7 — Pre-Check + DryRun على ProdCopy:**
```powershell
.\Pre-Migration-Check.ps1 -DatabaseName "RubikCare_ProdCopy" -Password "Rubikans@123"
.\Apply-Migrations.ps1 -DatabaseName "RubikCare_ProdCopy" -Password "Rubikans@123"
```

**الخطوة 8 — Execute على ProdCopy:**
```powershell
.\Apply-Migrations.ps1 -DatabaseName "RubikCare_ProdCopy" -Password "Rubikans@123" -Execute -Confirm
```

**الخطوة 9 — اختبار الميزة على ProdCopy** (من التطبيق المتصل بـ ProdCopy — إذا وُجد).

**الخطوة 10 — Pre-Check + DryRun على Production:**
```powershell
.\Pre-Migration-Check.ps1 -DatabaseName "RubikCare_Production" -Password "Rubikans@123"
.\Apply-Migrations.ps1 -DatabaseName "RubikCare_Production" -Password "Rubikans@123"
```

**الخطوة 11 — Execute على Production** (في جلسة مع AI، مع تأكيدات):
```powershell
.\Apply-Migrations.ps1 -DatabaseName "RubikCare_Production" -Password "Rubikans@123" `
    -Execute -Confirm -IAmSureThisIsProduction
# ثم اكتب: RubikCare_Production
```

**الخطوة 12 — تحديث `15B-DATABASE-MANAGEMENT.md`** (سجل التغييرات).

### 4.2 سيناريو 2: إضافة Seed Data جديدة (V003__PaymentMethods)

**السياق:** أضفت `PaymentMethods` (`CASH_ON_DELIVERY`, `BANK_TRANSFER`, `LEDGER_OFFSET`).

**الخطوة 1 — إنشاء ملف Seed:**
```
Scripts/Database/Seed/V003__PaymentMethods.sql
```

**البنية:**
```sql
-- ============================================================
-- V003__PaymentMethods
-- الغرض: إضافة طرق الدفع الأساسية
-- التاريخ: 2026-11-XX
-- ============================================================

-- فحص Idempotency (إن كان __SeedHistory موجوداً)
IF EXISTS (SELECT 1 FROM __SeedHistory WHERE SeedVersion = 'V003')
BEGIN
    PRINT '⚠️ V003 مطبق مسبقاً — تخطي.';
    RETURN;
END

-- البيانات (MERGE — Idempotent)
MERGE INTO PaymentMethods AS target
USING (VALUES 
    ('CASH_ON_DELIVERY', 'الدفع عند الاستلام', 'Cash on Delivery', 'CASH'),
    ('BANK_TRANSFER', 'تحويل بنكي', 'Bank Transfer', 'TRANSFER'),
    ('LEDGER_OFFSET', 'خصم من الرصيد', 'Ledger Offset', 'LEDGER')
) AS source (Code, NameAr, NameEn, PaymentType)
ON target.Code = source.Code
WHEN NOT MATCHED THEN
    INSERT (Code, NameAr, NameEn, PaymentType, IsActive, DisplayOrder, CreatedDate)
    VALUES (source.Code, source.NameAr, source.NameEn, source.PaymentType, 1, 100, GETUTCDATE());

-- تسجيل في __SeedHistory
INSERT INTO __SeedHistory (SeedVersion, SeedName, AppliedDate, AppliedBy, DatabaseName)
VALUES ('V003', 'PaymentMethods', GETUTCDATE(), SYSTEM_USER, DB_NAME());

PRINT '✅ V003__PaymentMethods مطبّق بنجاح.';
```

**الخطوة 2 — تنفيذ على Dev:**
```powershell
Invoke-Sqlcmd -ConnectionString "Server=161.97.105.83\RUBICKCARE;Database=RubikCare_Dev_V2;User Id=RubikansAdmin;Password=***;TrustServerCertificate=True" `
    -InputFile "Scripts\Database\Seed\V003__PaymentMethods.sql"
```

**الخطوة 3 — Pre-Check على Dev** (يجب أن يكتشف V003 كمطبق).

**الخطوة 4 — كرر على ProdCopy** ثم **Production**.

### 4.3 سيناريو 3: Migration اختبارية على ProdCopy

**السياق:** تريد اختبار `Apply-Migrations.ps1` قبل أن تُطبّق Migration كبيرة على Production.

**الخطوة 1 — إنشاء Migration اختبارية (على Dev):**
```powershell
cd C:\RC\Rubikcare.Full.Migration
dotnet ef migrations add __TestMigration `
    --project RubikCare.Infrastructure `
    --startup-project Api.Web
```

**الخطوة 2 — افتح الملف وأضف جدولاً بسيطاً:**
```csharp
protected override void Up(MigrationBuilder migrationBuilder)
{
    migrationBuilder.CreateTable(
        name: "__TestMigration",
        columns: table => new
        {
            Id = table.Column<int>(nullable: false)
                .Annotation("SqlServer:Identity", "1, 1"),
            TestName = table.Column<string>(maxLength: 100, nullable: true),
            CreatedDate = table.Column<DateTime>(nullable: false, defaultValueSql: "GETUTCDATE()")
        },
        constraints: table => table.PrimaryKey("PK___TestMigration", x => x.Id));
}

protected override void Down(MigrationBuilder migrationBuilder)
{
    migrationBuilder.DropTable(name: "__TestMigration");
}
```

**الخطوة 3 — Pre-Check على ProdCopy:**
```powershell
.\Pre-Migration-Check.ps1 -DatabaseName "RubikCare_ProdCopy" -Password "Rubikans@123"
```

**الخطوة 4 — DryRun على ProdCopy:**
```powershell
.\Apply-Migrations.ps1 -DatabaseName "RubikCare_ProdCopy" -Password "Rubikans@123"
```

**الخطوة 5 — Execute على ProdCopy:**
```powershell
.\Apply-Migrations.ps1 -DatabaseName "RubikCare_ProdCopy" -Password "Rubikans@123" -Execute -Confirm
```

**الخطوة 6 — تحقق:**
```sql
USE RubikCare_ProdCopy;
SELECT * FROM __TestMigration;         -- جدول موجود
SELECT TOP 1 MigrationId FROM __EFMigrationsHistory ORDER BY MigrationId DESC;  -- __TestMigration
```

**الخطوة 7 — حذف Migration الاختبارية (Rollback):**
```powershell
cd C:\RC\Rubikcare.Full.Migration
dotnet ef database update <PreviousMigration> --project RubikCare.Infrastructure --startup-project Api.Web --connection "..."
dotnet ef migrations remove --project RubikCare.Infrastructure --startup-project Api.Web
```

**الخطوة 8 — تحقق من الحذف:**
```sql
USE RubikCare_ProdCopy;
SELECT * FROM INFORMATION_SCHEMA.TABLES WHERE TABLE_NAME = '__TestMigration';  -- لا نتيجة
```

### 4.4 سيناريو 4: نشر على Production (خطوة بخطوة)

**السياق:** Migration اختُبرت على Dev و ProdCopy، جاهزة للإنتاج.

**قبل النشر:**
- [ ] Migration نجحت على Dev.
- [ ] Migration نجحت على ProdCopy.
- [ ] الميزة تعمل من التطبيق على Dev.
- [ ] الميزة تعمل من التطبيق على ProdCopy (إذا متاح).
- [ ] `Migrations_Exceptions.md` محدّث.
- [ ] `15B-DATABASE-MANAGEMENT.md` محدّث.

**خلال النشر (في جلسة مع AI):**

**الخطوة 1 — Pre-Check:**
```powershell
.\Pre-Migration-Check.ps1 -DatabaseName "RubikCare_Production" -Password "Rubikans@123"
```
**يجب أن ينجح 12/12.**

**الخطوة 2 — DryRun:**
```powershell
.\Apply-Migrations.ps1 -DatabaseName "RubikCare_Production" -Password "Rubikans@123"
```
**راجع قائمة Pending.**

**الخطوة 3 — Execute (مع تأكيدات):**
```powershell
.\Apply-Migrations.ps1 -DatabaseName "RubikCare_Production" -Password "Rubikans@123" `
    -Execute -Confirm -IAmSureThisIsProduction
```
**اكتب:** `RubikCare_Production` عند الطلب.

**الخطوة 4 — تحقق:**
```sql
USE RubikCare_Production;
SELECT TOP 1 MigrationId FROM __EFMigrationsHistory ORDER BY MigrationId DESC;
-- يجب أن يكون: <timestamp>_<NewMigration>
```

**الخطوة 5 — اختبار الميزة على Production** (من التطبيق).

**الخطوة 6 — تحديث السجلات:**
- `15B-DATABASE-MANAGEMENT.md` (سجل التغييرات).
- `Migrations_Exceptions.md` (إن وُجد).

### 4.5 سيناريو 5: Rollback (الطوارئ)

**السياق:** Migration نجحت على Production لكن الميزة تُسبب أخطاء.

**الخيارات (بالترتيب):**

**الخيار أ — Rollback Migration (إن كان `Down()` كامل):**
```powershell
cd C:\RC\Rubikcare.Full.Migration
dotnet ef database update <PreviousMigration> `
    --project RubikCare.Infrastructure `
    --startup-project Api.Web `
    --connection "Server=...;Database=RubikCare_Production;..."
```

**الخيار ب — Restore Backup (إن كان Rollback كامل مستحيل):**
```powershell
# عبر SSMS (يدوي — لأن Restore على Production ممنوع من PowerShell)
RESTORE DATABASE [RubikCare_Production]
FROM DISK = N'C:\Program Files\Microsoft SQL Server\MSSQL15.RUBICKCARE\MSSQL\Backup\PreMigration_<DB>_<timestamp>.bak'
WITH REPLACE, RECOVERY, STATS = 10;
```

**الخيار ج — تصحيح أمامي (الأفضل):**
- Migration جديدة `Fix<Issue>` تُصلح المشكلة.
- أسرع من Rollback، وأأمن.

**توصيتي:** **الخيار ج** — لأن Rollback قد يُسبب مشاكل إضافية (فقدان بيانات جديدة).

---

## 🛡️ الجزء الخامس: قواعد الأمان الصارمة

### 5.1 لا اعتماد على `appsettings.json`

**القاعدة:**
- ❌ **لا** `dotnet ef database update` بدون `--connection` صريح.
- ❌ **لا** `Apply-Migrations.ps1` بدون `-Password` + `-DatabaseName` صريح.
- ✅ **دائماً** `--connection` صريح + `-DatabaseName` صريح.

**السبب:** `appsettings.json` قد يُستبدل بملف بيئة. `ASPNETCORE_ENVIRONMENT` فارغ = `Production`.

### 5.2 DryRun افتراضي

**القاعدة:**
- ✅ **`-DryRun`** افتراضي (لا ينفّذ).
- ✅ **`-Execute`** مطلوب للتنفيذ.
- ✅ **`-Confirm`** مطلوب مع `-Execute`.

**السبب:** منع التنفيذ غير المقصود.

### 5.3 Backup إلزامي

**القاعدة:**
- ✅ **`Apply-Migrations.ps1`** يأخذ Backup تلقائياً.
- ✅ **`-SkipAutoBackup`** للاستخدام فقط عند وجود Backup خارجي حديث.
- ❌ **لا `-Execute` بدون Backup** (إلا بـ `-SkipAutoBackup` مع تأكيد).

**السبب:** Rollback يعتمد على Backup.

### 5.4 Pre-Check إلزامي

**القاعدة:**
- ✅ **`Apply-Migrations.ps1`** يستدعي `Pre-Migration-Check.ps1` تلقائياً.
- ✅ **`-SkipPreCheck`** للاستخدام فقط في حالات نادرة.
- ❌ **لا `-Execute` بدون Pre-Check**.

**السبب:** 12 فحصاً تكشف مشاكل قبل التطبيق.

### 5.5 Production — تأكيد مزدوج

**القاعدة:**
- ✅ **`-IAmSureThisIsProduction`** مطلوب لـ `RubikCare_Production`.
- ✅ **تأكيد تفاعلي** (كتابة اسم DB يدوياً).
- ✅ **في جلسة مع AI** (للمراجعة البشرية).

**السبب:** Production = بيانات حقيقية. لا مجال للخطأ.

### 5.6 Log كامل

**القاعدة:**
- ✅ **كل سكريبت** يكتب Log في `C:\WebSite\_Docs\Logs\Database\`.
- ✅ **كل Migration** تُسجَّل في `__EFMigrationsHistory` (تلقائي).
- ✅ **كل Seed** يُسجَّل في `__SeedHistory`.
- ✅ **كل Backup** يُسجَّل في `BackupHistory` (C#) + `msdb.dbo.backupset`.

**السبب:** تحليل الأخطاء + Compliance.

### 5.7 Idempotency

**القاعدة:**
- ✅ **كل Seed Script** يستخدم `MERGE` (Idempotent).
- ✅ **كل Migration** يستخدم `IF NOT EXISTS` (إن لزم).
- ✅ **`-idempotent`** عند توليد Scripts من `dotnet ef migrations script`.

**السبب:** تشغيل السكريبت مرتين = لا تأثير.

### 5.8 قاعدة "الحجم"

**القاعدة:**
- ✅ **`Pre-Migration-Check.ps1`** يُنبّه إذا `SizeGB > 5`.
- ✅ **`AutoBackup`** يبقى مفيداً حتى `10 GB`.
- ✅ **بعد 10 GB:** `Differential Backup` أو `-SkipAutoBackup` مع Backup خارجي.

**السبب:** Backup كامل على DB كبيرة = بطء + مساحة.

### 5.9 قاعدة "منع Rollback التلقائي على Production"

**القاعدة:**
- ❌ **`Restore-Database.ps1`** يرفض `RubikCare_Production` (يُطلب SSMS يدوي).
- ✅ **`dotnet ef database update <Previous>`** مسموح (Rollback Migration).
- ✅ **Rollback عبر SSMS** مسموح (يدوي، مع وعي كامل).

**السبب:** `RESTORE` = فقدان كل التغييرات بعد Backup. قرار بشري، لا آلي.

---

## 📋 الجزء السادس: Migrations Exceptions

### 6.1 سجل الاستثناءات

**الملف:** `Scripts/Database/Migrations_Exceptions.md`

**المحتوى الحالي:**

| # | Migration | التاريخ | السبب | البديل |
|---|---|---|---|---|
| 1 | `20260425124724_UpdateMedicationRequest` | 2026-11-06 | Migration زائدة — لم تُطبَّق | `20260425140827_MakePatientIdNullable` |

### 6.2 كيف نتعامل مع Migration زائدة؟

**السيناريو:** Migration موجودة في الكود، لكن لم تُطبَّق على أي DB، ولن تحتاج تطبيقها (لأن Migration أخرى فعلت نفس الشيء).

**الخطوات:**

**الخطوة 1 — التأكيد:**
- افحص `__EFMigrationsHistory` على كل DBs — هل مطبّقة؟
- افحص Migration الأخرى (البديل) — هل فعلت نفس الشيء؟

**الخطوة 2 — الحذف من Git:**
```powershell
cd C:\RC\Rubikcare.Full.Migration
git rm "RubikCare.Infrastructure/Migrations/<timestamp>_<Name>.cs"
git rm "RubikCare.Infrastructure/Migrations/<timestamp>_<Name>.Designer.cs"  # إذا كان موجوداً
git commit -m "Remove <Name> (superseded by <Alternative>)"
```

**الخطوة 3 — التحقق من `BusinessDbContextModelSnapshot.cs`:**
```powershell
Select-String -Path "RubikCare.Infrastructure\Migrations\BusinessDbContextModelSnapshot.cs" -Pattern "<Name>" -SimpleMatch
```
- إذا ظهر سطر → احذفه يدوياً.

**الخطوة 4 — التوثيق:**
- أضف سطراً في `Migrations_Exceptions.md`.

**الخطوة 5 — التحقق النهائي:**
```powershell
dotnet ef migrations list --project RubikCare.Infrastructure --startup-project Api.Web
# يجب أن لا تُظهر <Name>
```

### 6.3 سجل كامل (تاريخي)

**الجدول الحالي:**

| التاريخ | Migration | السبب | البديل | Commit |
|---|---|---|---|---|
| 2026-11-06 | `20260425124724_UpdateMedicationRequest` | لم تُطبَّق أبداً | `20260425140827_MakePatientIdNullable` | `06bb375` |

**ملاحظة:** `20260425134518_SyncMedicationRequestModel` فارغة (`Up()` و`Down()` لا يفعلان شيئاً) — لكنها مطبقة على كل DBs، فلا حاجة لحذفها.

---


## 🚀 الجزء السابع: خطة التوسع المستقبلية

### 7.1 عند نمو DB > 10 GB

**المشكلة:** `AutoBackup` (Full) يصبح بطيئاً (5-15 دقيقة)، ويستهلك مساحة كبيرة.

**الحل:**
- **`Differential Backup`** — Backup التغييرات منذ آخر Full (أسرع 10×).
- **`Transaction Log Backup`** — كل 15-30 دقيقة (Recovery Model = FULL).
- **`AutoBackup`** في `Apply-Migrations.ps1` → `-SkipAutoBackup` (الاعتماد على Backup خارجي مجدول).

**ما يُعدّل:**
- `Apply-Migrations.ps1` — إضافة `-UseDifferentialBackup` param.
- Windows Task Scheduler — Full يومي (ليلاً)، Diff كل 6 ساعات.

**ما يبقى:**
- `Pre-Migration-Check.ps1` — يعمل كما هو (12 فحصاً سريعاً).
- `Apply-Migrations.ps1` — `-AutoPreCheck` يبقى مفعّلاً.

### 7.2 عند نمو DB > 50 GB

**المشكلة:** حتى `Differential` يصبح كبيراً. `RESTORE` يستغرق ساعات.

**الحل:**
- **`Always On Availability Groups`** — نسخة ثانوية (للقراءة فقط + Failover).
- **`Database Snapshots`** — نسخة لحظية للاختبار.
- **`Stretch Database`** (Azure) — بيانات قديمة على السحابة.
- **`Partitioning`** — تقسيم الجداول الكبيرة (PSPTransactions, Backups).

**ما يُعدّل:**
- `Restore-Database.ps1` — يستخدم `WITH MOVE` + `PARTIAL`.
- `Compare-Databases.ps1` — يقارن Schema فقط (لا Data).

### 7.3 لوحة Admin `/admin/database`

**الرؤية:** صفحة Razor لعرض حالة DBs + تشغيل السكريبتات (بشكل آمن).

**المكونات:**
- **Dashboard** — حالة كل DB (آخر Migration، آخر Backup، الحجم).
- **Migrations** — عرض Applied + Pending لكل DB.
- **Seeds** — عرض Applied + Pending لكل DB.
- **Backups** — عرض Backups (مع Retention).
- **Actions** — أزرار: Create Backup, Compare, Pre-Check, Switch.

**التقييد:**
- **`Restore`** — معطّل على `RubikCare_Production` (يُطلب SSMS).
- **`Execute Migration`** — معطّل (يُطلب PowerShell).
- **`Compare`** — مسموح (قراءة فقط).

**متى؟** بعد B2B (عند نمو الحاجة للوحة).

### 7.4 Hangfire + Windows Service

**الرؤية:** تشغيل السكريبتات من واجهة Admin بأمان.

**البنية:**
```
[Admin UI] → [API endpoint] → [Hangfire Queue] 
                              ↓
                        [Windows Service "RubikCare-Jobs"]
                              ↓
                        [PowerShell Scripts]
```

**لماذا Windows Service؟**
- `IIS App Pool` لا يجب أن يُشغّل PowerShell بصلاحيات sysadmin.
- Service منفصل = أمان أعلى + استقلالية.

**متى؟** بعد B2B + `/admin/database`.

### 7.5 Retention Policy

**الرؤية:** إدارة دورية للنسخ الاحتياطية (حذف القديم).

**السياسة المقترحة:**
- **يومي:** احتفظ بـ 7 أيام.
- **أسبوعي:** احتفظ بـ 4 أسابيع (كل جمعة).
- **شهري:** احتفظ بـ 12 شهراً (أول كل شهر).
- **المجموع:** ~23 نسخة لكل DB.

**السكريبت:** `Retention-Manager.ps1` (مخطط).

**المنطق:**
```powershell
# احذف Backups الأقدم من 7 أيام (إذا لم تكن "أسبوعية" أو "شهرية")
# احذف Backups الأقدم من 4 أسابيع (إذا لم تكن "شهرية")
# احذف Backups الأقدم من 12 شهراً
```

**متى؟** بعد أول 20 Migration على Production.

### 7.6 Development Workflow (مستقبلاً)

**الرؤية:** تحويل كل ما نفعله يدوياً إلى CI/CD.

**البنية:**
```
[Git Push] → [GitHub Actions] → [Build] → [Test] → [Deploy to Dev]
                                              ↓
                                       [Deploy to ProdCopy]
                                              ↓
                                       [Manual Approval]
                                              ↓
                                       [Deploy to Production]
```

**الأدوات:**
- **GitHub Actions** (أو Azure DevOps).
- **Octopus Deploy** (اختياري — Production).
- **Sentry** (مراقبة الأخطاء).

**متى؟** بعد B2B + استقرار الفريق.

---

## 📚 الجزء الثامن: الدروس المستفادة

### 8.1 دروس الجلسة الأولى (2026-11-06/07)

#### الدرس 1: `INFORMATION_SCHEMA.TABLES` لا تُظهر `__` tables

**المشكلة:** `SELECT COUNT(*) FROM INFORMATION_SCHEMA.TABLES WHERE TABLE_NAME = '__SeedHistory'` = 0 (خطأ).
**السبب:** SQL Server يعامل الجداول التي تبدأ بـ `__` كـ System Tables.
**الحل:** استخدم `sys.tables`:
```sql
SELECT COUNT(*) FROM sys.tables WHERE name = '__SeedHistory';
```

#### الدرس 2: `--connection` صريح في `dotnet ef`

**المشكلة:** `dotnet ef database update` قرأ `appsettings.Secrets.json` (Dev_V2)، لكن عند `ASPNETCORE_ENVIRONMENT=Production` يقرأ `appsettings.Production.json` (Production).
**الخطر:** Migration قد تُطبَّق على Production بالخطأ.
**الحل:** استخدم `--connection` صريح دائماً.

#### الدرس 3: `ExecutionPolicy LocalMachine = Restricted`

**المشكلة:** `.\Pre-Migration-Check.ps1` فشل: "not digitally signed".
**الحل:** `powershell -ExecutionPolicy Bypass -File ".\Pre-Migration-Check.ps1"`.
**بديل:** `Set-ExecutionPolicy -Scope Process -ExecutionPolicy Bypass` (مؤقت).

#### الدرس 4: PowerShell backtick (`` ` ``) في بداية السطر

**المشكلة:** `.\Pre-Migration-Check.ps1 `-DatabaseName "..."` → PowerShell قرأ `-DatabaseName` كاسم السكريبت.
**الحل:** backtick في نهاية السطر (متابعة)، أو لا يوجد.

#### الدرس 5: `MigrationId` = `<Timestamp>_<Name>`

**المشكلة:** السكريبت قارن `MigrationName` (`AddSeedHistory`) بـ `MigrationId` (`20261106171444_AddSeedHistory`) — لم يتطابقا.
**الحل:** قارن `MigrationId` الكامل (اسم الملف بدون `.cs`).

#### الدرس 6: `appsettings.Secrets.json` هو مصدر `DefaultConnection` في Dev

**السبب:** `appsettings.json` فارغ، `appsettings.Secrets.json` يشير إلى `RubikCare_Dev_V2`.
**الخطر:** عند `ASPNETCORE_ENVIRONMENT=Development` — يقرأ `Secrets`. عند `Production` — يقرأ `Production`.
**الحل:** `--connection` صريح.

#### الدرس 7: `git rm` لملف `.Designer.cs`

**المشكلة:** `git rm "*.Designer.cs"` فشل: "did not match any files".
**السبب:** الملف لم يكن موجوداً على القرص (ربما لم يُنشأ أصلاً).
**الحل:** تحقق من `Test-Path` أولاً.

#### الدرس 8: `git add` لا يُنشئ الملفات

**المشكلة:** `git add "Scripts/Database/Migrations_Exceptions.md"` فشل: "did not match any files".
**السبب:** الملف لم يُنشأ بعد.
**الحل:** أنشئ الملف أولاً (`Out-File` أو Notepad).

#### الدرس 9: `ASPNETCORE_ENVIRONMENT` فارغ = `Production`

**السبب:** .NET يعامل `EnvironmentName` الفارغ كـ `Production` (في `WebApplication.CreateBuilder`).
**الخطر:** `dotnet ef` من المجلد الجذر بدون `ASPNETCORE_ENVIRONMENT` → يقرأ `appsettings.Production.json`.
**الحل:** `--connection` صريح + `-DatabaseName` صريح.

#### الدرس 10: `RubikCare_ProdCopy` = نسخة من Production للاختبار

**الاستخدام:** اختبار Migration قبل Production.
**ملاحظة:** `RubikCare_ProdCopy` **غير متصلة بأي بيئة** — آمنة للاختبار الحر.
**الحماية:** `Apply-Migrations.ps1` لا يحتاج `-IAmSureThisIsProduction` لـ ProdCopy (فقط لـ Production).

### 8.2 دروس مستقبلية (متوقعة)

#### الدرس 11: عند نمو DB (> 10 GB)

**التوقع:** `AutoBackup` يصبح بطيئاً.
**الحل:** `Differential Backup` + `-SkipAutoBackup`.

#### الدرس 12: عند نمو الفريق (2+ مطورين)

**التوقع:** تعارض Migrations بين الفروع.
**الحل:** `EF Core` يكتشف التعارض. `Migrations_Exceptions.md` يُوثّق.

#### الدرس 13: عند إضافة Seed Scripts كثيرة

**التوقع:** إدارة `V001, V002, ...` تصبح صعبة.
**الحل:** `__SeedHistory` يتتبع. `Apply-Migrations.ps1` يكتشف Pending.

---

## 📝 الجزء التاسع: سجل التغييرات

### 9.1 الإصدار 1.0 (2026-11-07)

**الإنجازات:**
- ✅ `__SeedHistory` — كيان + خدمة + Migration مطبّق على 3 DBs.
- ✅ `Pre-Migration-Check.ps1` v2.1 — 12 فحصاً، مُختبر على ProdCopy و Production.
- ✅ `Apply-Migrations.ps1` v1.0 — DryRun مُختبر، Execute بانتظار أول Migration B2B.
- ✅ حذف `20260425124724_UpdateMedicationRequest` (Migration زائدة).
- ✅ `Migrations_Exceptions.md` — سجل الاستثناءات.
- ✅ 3 نسخ احتياطية — Dev_V2, ProdCopy, Production.

**الملفات المنشأة:**
| الملف | المسار |
|---|---|
| `SeedHistory.cs` | `RubikCare.Domain/Entities/System/` |
| `ISeedHistoryService.cs` | `RubikCare.Application/Interfaces/MyBackup/` |
| `SeedHistoryService.cs` | `RubikCare.Infrastructure/Services/MyBackup/` |
| Migration `20261106171444_AddSeedHistory` | `RubikCare.Infrastructure/Migrations/` |
| `Pre-Migration-Check.ps1` v2.1 | `Scripts/Database/_Infrastructure/` |
| `Apply-Migrations.ps1` v1.0 | `Scripts/Database/_Infrastructure/` |
| `Migrations_Exceptions.md` | `Scripts/Database/` |
| `15B-DATABASE-MANAGEMENT.md` (هذه الوثيقة) | `Scripts/Database/` |

**Git commits:**
- `237ee56` — WIP: SeedHistory + Pre-Migration-Check + Dashboard fixes + Onboarding
- `06bb375` — Remove UpdateMedicationRequest (superseded by MakePatientIdNullable)
- `36254b9` — Document exception: removed UpdateMedicationRequest

### 9.2 ما هو مخطط (2026-11/12)

| # | البند | الأولوية | التقدير |
|---|---|---|---|
| 1 | `Backup-Before-Deploy.ps1` | 🟠 عالية | 2-3 ساعات |
| 2 | `Restore-Database.ps1` | 🟠 عالية | 3-4 ساعات |
| 3 | `Compare-Databases.ps1` | 🟡 متوسطة | 4-6 ساعات |
| 4 | `Switch-DevToProdCopy.ps1` | 🟡 متوسطة | 2-3 ساعات |
| 5 | Seed Scripts (V001, V002, V003) | 🟠 عالية | 2-3 ساعات |
| 6 | `/admin/database` (Razor) | 🟡 متوسطة | 8-10 ساعات |
| 7 | `Retention-Manager.ps1` | 🟢 منخفضة | 2-3 ساعات |
| 8 | Hangfire + Windows Service | 🟢 منخفضة | 15-20 ساعة |

### 9.3 سجل التغييرات المستقبلي

**عند كل تغيير في هذه الوثيقة — أضف سطراً هنا:**

| الإصدار | التاريخ | التغيير |
|---|---|---|
| 1.0 | 2026-11-07 | الإصدار الأولي — `__SeedHistory` + `Pre-Migration-Check.ps1` v2.1 + `Apply-Migrations.ps1` v1.0 + `Migrations_Exceptions.md` |

---

## 🔗 الجزء العاشر: مراجع ذات صلة

### 10.1 وثائق داخلية

| الوثيقة | الموضوع | علاقة |
|---|---|---|
| `15-DEPLOYMENT.md` | نشر IIS/Web/API/PWA | مكمل (نشر التطبيقات) |
| `06-POINTS.md` | نظام النقاط | يستخدم `__SeedHistory` (B2B) |
| `19-PERMISSIONS.md` | نظام الصلاحيات | يستخدم `Pre-Migration-Check` |
| `00-MASTER-INDEX.md` | الفهرس الشامل | يُحدَّث مع كل جلسة |

### 10.2 السكريبتات

| السكريبت | المسار |
|---|---|
| `Pre-Migration-Check.ps1` | `Scripts/Database/_Infrastructure/` |
| `Apply-Migrations.ps1` | `Scripts/Database/_Infrastructure/` |
| `Migrations_Exceptions.md` | `Scripts/Database/` |

### 10.3 اللوجات

| Log | المسار |
|---|---|
| `PreMigrationCheck_<DB>_<timestamp>.log` | `C:\WebSite\_Docs\Logs\Database\` |
| `ApplyMigrations_<DB>_<timestamp>.log` | `C:\WebSite\_Docs\Logs\Database\` |

### 10.4 النسخ الاحتياطية

| Backup | المسار |
|---|---|
| `PreSeedHistory_<DB>_<timestamp>.bak` | `C:\Program Files\Microsoft SQL Server\MSSQL15.RUBICKCARE\MSSQL\Backup\` |
| `PreMigration_<DB>_<timestamp>.bak` (مستقبلاً) | نفس المسار |

---

## ✅ الخلاصة

**هذه الوثيقة هي المرجع الكامل لإدارة قواعد البيانات في RubikCare.**

**قبل أي Migration:**
1. اقرأ هذه الوثيقة (خاصة الجزء 0 + 5).
2. اتبع سيناريو 4.1 (Migration جديدة).
3. وثّق النتائج هنا (الجزء 9).

**عند أي طارئ:**
1. اقرأ الجزء 6 (Migrations Exceptions).
2. استخدم الجزء 4.5 (Rollback).

**عند نمو DB:**
1. اقرأ الجزء 7 (خطة التوسع).
2. طبّق `Differential Backup` أو `Retention`.

---
