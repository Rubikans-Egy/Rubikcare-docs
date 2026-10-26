
# 15 — DEPLOYMENT (دليل النشر والإنتاج)

الإصدار: 3.0 (After Real-World Deployment) 
آخر تحديث: 27 أكتوبر 2026 
الحالة: ✅ مرجع شامل — يدمج 12 + دروس مُتراكمة + دروس من النشر الفعلي (26-27 أكتوبر) 
الجمهور: فريق التطوير + DevOps + AI Assistants 
المرجع الإلزامي: يُقرأ قبل أي نشر

---

## 📌 الجزء صفر: مقدمة

### 0.1 نطاق هذه الوثيقة
هذا المرجع يغطي كل ما يتعلق بنشر منصة RubikCare على جميع البيئات:
- **Web (IIS)** — Blazor Server.
- **PWA (IIS)** — Blazor WebAssembly.
- **API (ASP.NET Core)** — Backend REST API.
- **Mobile (MAUI)** — Android APK/AAB + iOS IPA.
- **قاعدة البيانات (Database)** — تحديث الإنتاج بأمان.

### 0.2 الأهداف
- توحيد إجراءات النشر.
- توثيق الدروس المستفادة من مشاكل حقيقية.
- توفير سكريبتات أتمتة (PowerShell) لتقليل الأخطاء.
- حماية البيانات (عدم المساس بـ appsettings.json و web.config و SharedStorage).

### 0.3 الوثائق المرجعية
- `01-ARCHITECTURE` — بنية المشاريع.
- `02-IDENTITY` — إدارة الجلسات.
- `04-TRANSLATION` — الترجمة و MERGE.
- `05-PSP` — PatientID vs CaseID (الدرس المستفاد الأهم).
- `06-POINTS` — نظام النقاط.
- `13-MAUI` — تفاصيل MAUI.
- `14-PWA` — تفاصيل PWA.

---

## 🗺️ الجزء الأول: خريطة البيئات

### 1.1 البيئات المعتمدة

| البيئة | الغرض | رابط Web | رابط API | مسار السيرفر |
| :--- | :--- | :--- | :--- | :--- |
| Local | تطوير واختبار | https://localhost:xxxx | https://localhost:yyyy | C:\RC\... (جهازك) |
| **Stage (Web)** | اختبار Web | https://test.rubikcare.com | https://uat.rubikcare.com | `C:\WebSite\RubikCareNew` |
| **Stage (API)** | اختبار API | — | https://uat.rubikcare.com | `C:\WebSite\RubikCareUat` |
| **Stage (PWA)** | اختبار PWA | https://stagepu.rubikcare.com | https://uat.rubikcare.com | `C:\WebSite\PU_RubicCareStage` |
| **Production (Web)** | الإنتاج | https://rubikcare.com | https://api.rubikcare.com | `C:\WebSite\RubikCareLive` |
| **Production (API)** | الإنتاج | — | https://api.rubikcare.com | `C:\WebSite\RubicCareApi` |
| **Production (PWA)** | الإنتاج | https://stage.rubikcare.com | https://api.rubikcare.com | `C:\WebSite\RubicCareStage` (مخطط) |

### 1.2 الهيكل الكامل لمجلد `C:\WebSite`

```
C:\WebSite\
├── _Archive\               ← المواقع القديمة (لا تستخدم)
├── _Docs\                  ← التوثيق والسكريبتات
│   ├── README.md
│   ├── Scripts\
│   ├── Backups\
│   └── Logs\
├── _Staging\               ← منطقة الانتقال (لنشر الملفات يدوياً)
│   ├── Web\
│   ├── API\
│   └── PWA\
├── SharedStorage\          ← تخزين الملفات المشترك
│   └── uploads\
├── RubikCareNew\           ← Web Stage
├── RubikCareLive\          ← Web Prod
├── RubikCareUat\           ← API Stage
├── RubicCareApi\           ← API Prod
├── PU_RubicCareStage\      ← PWA Stage
└── RubicCareStage\         ← PWA Prod
```

### 1.3 ⚠️ القواعد الذهبية
1. **لا تحذف `SharedStorage` أبداً** (يحتوي على ملفات المستخدمين).
2. **لا تنسخ `appsettings.json` أو `web.config`** من Stage إلى Prod (تحتوي على إعدادات مختلفة).
3. **لا تنشر إلى Production مباشرة** — دائمًا Stage أولاً ثم اختبار ثم Production.

---

## 🏗️ الجزء الثاني: البنية التحتية المشتركة

### 2.1 المتطلبات الأساسية
| المكون | الإصدار الأدنى | الغرض |
| :--- | :--- | :--- |
| Windows Server | 2019+ | نظام التشغيل |
| IIS | 10+ | خادم الويب |
| .NET Runtime | 10.0+ | تشغيل التطبيقات |
| SQL Server | 2019+ | قاعدة البيانات |
| URL Rewrite | 2.1+ | ضروري لـ PWA |

### 2.2 إعداد IIS (باستخدام PowerShell)

#### إنشاء Application Pools
```powershell
# API Production
New-WebAppPool -Name "RubikCareApi"
Set-ItemProperty -Path "IIS:\AppPools\RubikCareApi" -Name "managedRuntimeVersion" -Value ""

# Web Stage
New-WebAppPool -Name "rubikcarenew"
Set-ItemProperty -Path "IIS:\AppPools\rubikcarenew" -Name "managedRuntimeVersion" -Value ""

# PWA Stage
New-WebAppPool -Name "PU_RubicCareStage"
Set-ItemProperty -Path "IIS:\AppPools\PU_RubicCareStage" -Name "managedRuntimeVersion" -Value ""
```

#### منح الصلاحيات (مهم جداً)
```powershell
# 1. صلاحيات Web على SharedStorage
icacls "C:\WebSite\SharedStorage" /grant "IIS AppPool\RubikCareLive:(OI)(CI)M" /T
icacls "C:\WebSite\SharedStorage" /grant "IIS AppPool\rubikcarenew:(OI)(CI)M" /T

# 2. صلاحيات API على SharedStorage
icacls "C:\WebSite\SharedStorage" /grant "IIS AppPool\RubikCareApi:(OI)(CI)M" /T
icacls "C:\WebSite\SharedStorage" /grant "IIS AppPool\RubikCareUat:(OI)(CI)M" /T
```

### 2.3 إعداد `SharedStorage` (تخزين الملفات المشترك)
**كل من Web (Stage/Prod) و API (Stage/Prod) يجب أن يشير إلى نفس المسار:**

في `appsettings.json` لكل موقع:
```json
"Storage": {
  "SharedPath": "C:\\WebSite\\SharedStorage\\uploads"
}
```

**السبب:** لضمان أن الملفات المرفوعة من Web يمكن قراءتها من API والعكس صحيح.

---

## 🔌 الجزء الثالث: نشر API (ASP.NET Core)

### 3.1 إجراءات ما قبل النشر
```powershell
# 1. تأكد من عدم وجود أخطاء بناء
cd C:\RC\Rubikcare.Full.Migration
dotnet build RubikCare.Api.Web -c Release

# 2. تحقق من Migrations غير مطبقة
dotnet ef migrations list --project RubikCare.Infrastructure --startup-project RubikCare.Api.Web
```

### 3.2 خطوات النشر (يدوياً)
```powershell
# 1. إيقاف App Pool
Stop-WebAppPool -Name "RubikCareApi"

# 2. نسخ احتياطي
Copy-Item "C:\WebSite\RubicCareApi\appsettings.json" "C:\WebSite\RubicCareApi\appsettings.json.bak" -Force
Copy-Item "C:\WebSite\RubicCareApi\web.config" "C:\WebSite\RubicCareApi\web.config.bak" -Force

# 3. نشر المشروع
cd C:\RC\Rubikcare.Full.Migration
dotnet publish RubikCare.Api.Web -c Release -o "C:\WebSite\_Staging\API"

# 4. نسخ الملفات (مع استبعاد الإعدادات)
robocopy "C:\WebSite\_Staging\API" "C:\WebSite\RubicCareApi" /MIR /XF "appsettings.json" "appsettings.*.json" "web.config" /XD "logs"

# 5. تشغيل App Pool
Start-WebAppPool -Name "RubikCareApi"
```

### 3.3 التحقق بعد النشر
```powershell
Invoke-WebRequest -Uri "https://api.rubikcare.com/health" -UseBasicParsing
```

---

## 🌐 الجزء الرابع: نشر Web (Blazor Server)

### 4.1 إعدادات `appsettings.json` في Stage و Prod
**Stage (`RubikCareNew`):**
```json
{
  "ApiSettings": { "BaseUrl": "https://uat.rubikcare.com" },
  "Storage": { "SharedPath": "C:\\WebSite\\SharedStorage\\uploads" }
}
```

**Prod (`RubikCareLive`):**
```json
{
  "ApiSettings": { "BaseUrl": "https://api.rubikcare.com" },
  "Storage": { "SharedPath": "C:\\WebSite\\SharedStorage\\uploads" }
}
```

### 4.2 خطوات النشر
```powershell
# 1. إيقاف App Pool
Stop-WebAppPool -Name "rubikcarenew"

# 2. نسخ احتياطي
$timestamp = Get-Date -Format "yyyyMMdd_HHmm"
Compress-Archive -Path "C:\WebSite\RubikCareNew\*" -DestinationPath "C:\WebSite\_Docs\Backups\WebStage_$timestamp.zip" -Force

# 3. نشر المشروع إلى Staging
cd C:\RC\Rubikcare.Full.Migration
dotnet publish RubikCare.Web -c Release -o "C:\WebSite\_Staging\Web"

# 4. نسخ الملفات (مع استبعاد الإعدادات)
robocopy "C:\WebSite\_Staging\Web" "C:\WebSite\RubikCareNew" /MIR /XF "appsettings.json" "appsettings.*.json" "web.config" /XD "logs"

# 5. تشغيل App Pool
Start-WebAppPool -Name "rubikcarenew"
```

---
## 📱 الجزء الخامس: نشر PWA (Blazor WebAssembly)

### 5.1 نظرة عامة
تطبيق `RubikCare.PWA` هو Blazor WebAssembly. يختلف نشره كلياً عن Web/API:
- **يحتاج** URL Rewrite + MIME Types (لأنه يخدم ملفات `.wasm` و `.dat`).
- **لا يحتاج** .NET Runtime (لأنه ملفات ثابتة).
- **المسار الفيزيائي** في IIS يجب أن يكون الجذر (`C:\WebSite\PU_RubicCareStage`)، وليس `wwwroot`.

### 5.2 خطوات النشر (Stage)
```powershell
# 1. إيقاف App Pool
Stop-WebAppPool -Name "PU_RubicCareStage"

# 2. نسخ احتياطي
$timestamp = Get-Date -Format "yyyyMMdd_HHmm"
Compress-Archive -Path "C:\WebSite\PU_RubicCareStage\*" -DestinationPath "C:\WebSite\_Docs\Backups\PWAStage_$timestamp.zip" -Force

# 3. نشر المشروع إلى Staging
cd C:\RC\Rubikcare.Full.Migration
dotnet publish RubikCare.PWA -c Release -o "C:\WebSite\_Staging\PWA"

# 4. نسخ الملفات (مع استبعاد web.config فقط)
robocopy "C:\WebSite\_Staging\PWA" "C:\WebSite\PU_RubicCareStage" /MIR /XF "web.config"

# 5. تشغيل App Pool
Start-WebAppPool -Name "PU_RubicCareStage"
```

### 5.3 ⚠️ قاعدة حرجة: `web.config` الخاص بـ PWA
ملف `web.config` للـ PWA **يجب أن يظل موجوداً** على السيرفر. لا تمسحه أبداً. استخدام `robocopy /XF "web.config"` هو الحل الآمن. (الوثيقة القديمة كانت تطلب مسح كل شيء، وهذا خطأ فادح).

---

## 🗄️ الجزء السادس: قاعدة البيانات (Database)

هذا هو **القسم الأكثر حساسية**. تم توثيقه بناءً على تجربة 26-27 أكتوبر.

### 6.1 قاعدة ذهبية: لا تنشر قاعدة بيانات مباشرة
لا تستخدم `dotnet ef database update` مباشرة على Production. بدلاً من ذلك:
1. **ولّد سكريبت SQL** من جهازك المحلي.
2. **راجعه** بعناية.
3. **نفذه** يدوياً في SSMS.

### 6.2 توليد سكريبت الترقية (من جهازك المحلي)
```powershell
# 1. معرفة آخر Migration مطبقة في Production (من SSMS)
# SELECT TOP 1 MigrationId FROM __EFMigrationsHistory ORDER BY MigrationId DESC;

# 2. توليد سكريبت للـ Migrations الناقصة
dotnet ef migrations script 20260721095751_AddPharmaCompanyMedicationIdToPSPProgramMedication 20261021151545_RenamePatientIdToCaseId -o "C:\RC\Production_Update.sql" --project RubikCare.Infrastructure --startup-project Rubikcare.Web --idempotent
```

**الخيار `--idempotent`**: يضمن أن السكريبت آمن للتشغيل أكثر من مرة (يتحقق من `__EFMigrationsHistory` قبل التطبيق).

### 6.3 فحص "السجلات اليتيمة" (Orphaned Records) قبل التطبيق
**الدرس المستفاد الأهم:** Migration `RenamePatientIdToCaseId` تفشل إذا كانت هناك سجلات في `PSPeRX` تشير إلى `PatientID` غير موجود في `PSPPatients`. **قبل التنفيذ، افحص:**

```sql
-- فحص كل الجداول التي ستعيد التسمية
SELECT 'PSPeRX' AS TableName, COUNT(*) AS Orphaned_Count
FROM PSPeRX e LEFT JOIN PSPPatients p ON e.PatientID = p.PatientID WHERE p.PatientID IS NULL
UNION ALL
SELECT 'PSPReminders', COUNT(*)
FROM PSPReminders r LEFT JOIN PSPPatients p ON r.PatientID = p.PatientID WHERE p.PatientID IS NULL
UNION ALL
SELECT 'PSPTestResults', COUNT(*)
FROM PSPTestResults t LEFT JOIN PSPPatients p ON t.PatientID = p.PatientID WHERE p.PatientID IS NULL
UNION ALL
SELECT 'PSPFollowUpRecords', COUNT(*)
FROM PSPFollowUpRecords f LEFT JOIN PSPPatients p ON f.PatientID = p.PatientID WHERE p.PatientID IS NULL
UNION ALL
SELECT 'PSPAdverseEvents', COUNT(*)
FROM PSPAdverseEvents a LEFT JOIN PSPPatients p ON a.PatientID = p.PatientID WHERE p.PatientID IS NULL
UNION ALL
SELECT 'DoctorNotes', COUNT(*)
FROM DoctorNotes d LEFT JOIN PSPPatients p ON d.PatientID = p.PatientID WHERE p.PatientID IS NULL;
```

**إذا كانت النتيجة > 0:** احذف السجلات اليتيمة (أو اربطها بمريض موجود) قبل تنفيذ السكريبت.

### 6.4 تنفيذ السكريبت (في SSMS على السيرفر)
1. اتصل بـ SQL Server (عبر IP أو اسم السيرفر).
2. اختر قاعدة بيانات **`RubikCare_Production`**.
3. افتح الملف `Production_Update.sql`.
4. اضغط **Execute**.
5. راقب علامة التبويب **Messages**. يجب أن ترى `Successfully applied migrations`.

### 6.5 التحقق النهائي (Health Check لقاعدة البيانات)
بعد التنفيذ، نفذ هذا الاستعلام الشامل للتأكد من أن كل شيء متطابق:

```sql
-- 1. فحص الأعمدة (يجب أن يكون فارغاً)
SELECT TABLE_NAME, COLUMN_NAME 
FROM INFORMATION_SCHEMA.COLUMNS 
WHERE COLUMN_NAME = 'PatientID'
AND TABLE_NAME IN ('PSPPatients', 'PSPeRX', 'PSPReminders', 'PSPTestResults', 'PSPFollowUpRecords', 'PSPAdverseEvents', 'DoctorNotes');

-- 2. فحص السجلات اليتيمة (يجب أن تكون 0)
SELECT 'PSPeRX' AS TableName, COUNT(*) FROM PSPeRX e LEFT JOIN PSPPatients p ON e.CaseID = p.CaseID WHERE p.CaseID IS NULL;

-- 3. فحص الـ Migrations (يجب أن يظهر Migration الجديد)
SELECT MigrationId FROM __EFMigrationsHistory WHERE MigrationId = '20261021151545_RenamePatientIdToCaseId';
```

### 6.6 تحديث الترجمات (استثناء: بدون Migration)
الترجمات (`Resources`) يتم تحديثها عبر **MERGE** مباشر، وليس عبر Migration. انظر `04-TRANSLATION` للتفاصيل.

---

## 📂 الجزء السابع: تنظيف وأرشفة المجلدات القديمة

### 7.1 قبل التنظيف
- خذ نسخة احتياطية من `appsettings.json` و `web.config` لكل موقع.
- تأكد من أن المجلدات المراد أرشفتها **غير مرتبطة بأي موقع نشط في IIS**.

### 7.2 أرشفة المجلدات
```powershell
# إنشاء مجلد الأرشيف
New-Item -ItemType Directory -Path "C:\WebSite\_Archive" -Force

# أرشفة المجلدات القديمة
$oldFolders = @("Old 2023-02-02", "RubicCareStageAdmin", "RubikCareAdminProduction", "RubikCareAdminTest", "RubikCareUpdate", "TEST& backup", "SharedStorage")
foreach ($folder in $oldFolders) {
    $src = Join-Path "C:\WebSite" $folder
    $dst = Join-Path "C:\WebSite\_Archive" $folder
    if (Test-Path $src) {
        Move-Item -Path $src -Destination $dst -Force
    }
}
```

### 7.3 ⚠️ تحذير: `SharedStorage` لا يؤرشف
`SharedStorage` يحتوي على ملفات المستخدمين. **لا تنقله إلى `_Archive` أبداً.** (لقد حدث هذا خطأً سابقاً، وتم إصلاحه).

---

## 🔴 الجزء الثامن: دروس مستفادة (Lessons Learned) — من أكتوبر 2026

### الدرس 1: `PatientID` vs `CaseID` (25 سجلاً يتيماً)
**المشكلة:** Migration `RenamePatientIdToCaseId` فشلت بـ `FK_PSPeRX_PSPPatients_CaseID` conflict.
**السبب:** 25 سجلاً في `PSPeRX` تشير إلى `PatientID` غير موجود.
**الحل:** حذف السجلات اليتيمة قبل التنفيذ.
**الإجراء الوقائي:** أي Migration تعيد تسمية FK يجب أن تُفحص بـ `LEFT JOIN` قبل التنفيذ.

### الدرس 2: `Compress-Archive` يفشل مع `logs`
**المشكلة:** `Compress-Archive` فشل لأن ملف `stdout_*.log` مفتوح من IIS.
**الحل:** إيقاف App Pool قبل النسخ الاحتياطي (أو استبعاد مجلد `logs`).
**الإجراء الوقائي:** استخدم `robocopy /XD "logs"` في السكريبتات.

### الدرس 3: Web Deploy و `401 Unauthorized`
**المشكلة:** Web Deploy يفشل في الاتصال بـ `msdeploy.axd` رغم صحة بيانات الدخول.
**الحل:** التخلي عن Web Deploy والاعتماد على PowerShell + `robocopy`.
**الإجراء الوقائي:** لا تعتمد على Web Deploy في بيئات معقدة.

### الدرس 4: ترميز العربية في PowerShell
**المشكلة:** ملفات `.ps1` و `.md` تظهر بالعربية المشوشة.
**الحل:** حفظ الملفات بترميز **UTF-8 with BOM**، وإضافة `[Console]::OutputEncoding = [System.Text.Encoding]::UTF8` في بداية السكريبتات.

### الدرس 5: `Delete existing files` في Visual Studio
**المشكلة:** خيار `Delete existing files` في Visual Studio قد يمسح `appsettings.json` من مجلد النشر.
**الحل:** استخدم `robocopy /XF` في السكريبتات بدلاً من الاعتماد على Visual Studio.

### الدرس 6: `SharedStorage` يجب أن يكون مشتركاً
**المشكلة:** Web و API كلاهما يجب أن يشير إلى `SharedStorage` في `appsettings.json`، وإلا ستكون الملفات المرفوعة غير مرئية للطرف الآخر.

---

## 📋 الجزء التاسع: Master Deployment Checklist

### قبل النشر
- [ ] تم أخذ نسخة احتياطية من قاعدة البيانات.
- [ ] تم أخذ نسخة احتياطية من `appsettings.json` و `web.config` لكل موقع.
- [ ] تم فحص السجلات اليتيمة (Orphans) في قاعدة البيانات.
- [ ] `_Staging` يحتوي على أحدث نسخة من الكود.

### أثناء النشر
- [ ] إيقاف App Pool.
- [ ] نسخ الملفات باستخدام `robocopy /MIR /XF appsettings.json web.config /XD logs`.
- [ ] تشغيل App Pool.
- [ ] تنفيذ سكريبت قاعدة البيانات (إن وجد).

### بعد النشر
- [ ] اختبار الموقع (Health Check).
- [ ] اختبار تسجيل الدخول.
- [ ] اختبار الصفحة الرئيسية.
- [ ] اختبار صفحة التقارير.
- [ ] اختبار رفع ملف (للتأكد من `SharedStorage`).

---

## 🔗 الجزء العاشر: روابط ذات صلة
- `01-ARCHITECTURE.md`
- `05-PSP.md` (PatientID vs CaseID)
- `06-POINTS.md`
- `13-MAUI.md`
- `14-PWA.md`
- `17-STRATEGIC-ROADMAP.md`

---

## 📝 الجزء الحادي عشر: سجل التغييرات

| الإصدار | التاريخ | التغييرات |
| :--- | :--- | :--- |
| 1.0 | 29 أغسطس 2026 | الإصدار الأولي |
| 2.0 | 17 أكتوبر 2026 | 15-DEPLOYMENT v2.0 — دمج 12 + القسم 4.5 الحرج |
| **3.0** | **27 أكتوبر 2026** | **v3.0 — إضافة خريطة البيئات الكاملة + `SharedStorage` + `_Staging` + `_Archive` + قسم قاعدة البيانات (6) + Lessons Learned (8) + إصلاح `robocopy` + إزالة `Delete existing files` + إضافة `_Docs/Scripts`** |

---

© 2026 RubikCare — للاستخدام الداخلي
نهاية الوثيقة ✅
```

---
