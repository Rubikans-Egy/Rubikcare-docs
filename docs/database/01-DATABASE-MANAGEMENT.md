
# 15B — DATABASE MANAGEMENT (إدارة قواعد البيانات)

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
