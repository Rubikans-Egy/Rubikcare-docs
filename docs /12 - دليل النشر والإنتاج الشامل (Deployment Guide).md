
# 📖 12 - دليل النشر والإنتاج الشامل (Deployment Guide)

**آخر تحديث:** 29 أغسطس 2026  
**الحالة:** ✅ محدّث ليشمل Blazor WebAssembly PWA  
**المؤلفون:** فريق RubikCare

---

## 📌 مقدمة

هذا المرجع الشامل يغطي **كل ما يتعلق بنشر منصة RubikCare** على جميع البيئات:

- **Web (IIS)** - Blazor Server + Blazor WebAssembly PWA
- **Mobile (MAUI)** - Android APK/AAB + iOS IPA
- **API (ASP.NET Core)** - Backend REST API

### 🎯 الأهداف

1. **توحيد إجراءات النشر** عبر جميع البيئات
2. **توثيق الدروس المستفادة** من المشاكل الحقيقية
3. **توفير سكربتات أتمتة** لتقليل الأخطاء اليدوية
4. **إنشاء قوائم تحقق** لضمان الجودة

### 📚 الوثائق المرجعية

قبل البدء، يُفضل مراجعة:
- [00 - الهيكل المعماري](00-architecture-overview.md) - لفهم بنية المشاريع
- [01 - Program.cs والتسجيلات الأساسية](01-program-cs-foundation.md) - لفهم الخدمات
- [02 - نظام الهوية والمصادقة](02-identity-system.md) - لفهم إدارة الجلسات

---

## 🗺️ القسم 1: خريطة البيئات (Environment Map)

### 🌍 البيئات المعتمدة

| البيئة | الغرض | رابط Web | رابط API | مسار السيرفر |
|--------|-------|----------|----------|--------------|
| **Local** | التطوير والاختبار المبدئي | `https://localhost:xxxx` | `https://localhost:yyyy` | `C:\Users\{user}\source\repos\...` |
| **Test / UAT** | الاختبار الشامل قبل النشر | `https://test.rubikcare.com` | `https://uat.rubikcare.com` | Web: `C:\WebSite\RubikCareNew`<br>API: `C:\WebSite\RubikCareUat` |
| **Stage (PWA)** | بيئة PWA للاختبار | `https://stagepu.rubikcare.com` | `https://uat.rubikcare.com` | PWA: `C:\WebSite\PU_RubicCareStage`<br>API: `C:\WebSite\RubikCareUat` |
| **Production** | البيئة الفعلية للمستخدمين | `https://rubikcare.com` | `https://api.rubikcare.com` | Web: `C:\WebSite\RubikCareLive`<br>API: `C:\WebSite\RubikCareApi` |

### ⚠️ قاعدة ذهبية

> **تطبيق الـ Web/PWA في بيئة الـ Test/Stage يجب أن يكون `ApiSettings:BaseUrl` فيه مساوياً لـ `https://uat.rubikcare.com` وليس الـ Live.**

---

## 🏗️ القسم 2: البنية التحتية المشتركة

### 📋 المتطلبات الأساسية

| المكون | الإصدار الأدنى | الغرض |
|--------|----------------|-------|
| Windows Server | 2019+ | نظام التشغيل |
| IIS | 10+ | خادم الويب |
| .NET Runtime | 10.0+ | تشغيل التطبيقات |
| SQL Server | 2019+ | قاعدة البيانات |
| URL Rewrite Module | 2.1+ | إعادة كتابة الروابط (ضروري لـ SPA) |

### 🔧 إعداد IIS الأساسي

#### 1. تثبيت URL Rewrite Module

```powershell
# التحقق من وجود الوحدة
Get-WebGlobalModule | Where-Object { $_.Name -like "*Rewrite*" }

# إذا لم تظهر، ثبّت من:
# https://www.iis.net/downloads/microsoft/url-rewrite

# التحقق بعد التثبيت
Test-Path "C:\Windows\System32\inetsrv\rewrite.dll"
```

#### 2. إنشاء Application Pool

```powershell
# Blazor Server / API (ASP.NET Core)
C:\Windows\System32\inetsrv\appcmd add apppool `
    /name:"RubikCareApi" `
    /managedRuntimeVersion:"" `
    /managedPipelineMode:"Integrated"

# Blazor WebAssembly PWA (Static Files)
C:\Windows\System32\inetsrv\appcmd add apppool `
    /name:"PU_RubicCareStage" `
    /managedRuntimeVersion:"" `
    /managedPipelineMode:"Integrated"
```

**ملاحظة مهمة:** `managedRuntimeVersion:""` يعني "No Managed Code" - ضروري لـ ASP.NET Core.

#### 3. منح الصلاحيات

```powershell
# منح صلاحيات القراءة للمجلد
icacls "C:\WebSite\RubikCareApi" /grant "IIS_IUSRS:(OI)(CI)RX" /T
icacls "C:\WebSite\PU_RubicCareStage" /grant "IIS_IUSRS:(OI)(CI)RX" /T

# منح صلاحيات الكتابة لمجلد الرفع (إذا لزم)
icacls "C:\WebSite\RubikCareApi\wwwroot\uploads" /grant "IIS_IUSRS:(OI)(CI)M" /T

```
---


## 🔌 القسم 3: نشر API (ASP.NET Core)

### 📋 نظرة عامة

API هو **نقطة الدخول الوحيدة للبيانات** في المنصة. جميع العملاء (Web, Mobile, PWA) يتواصلون معه عبر HTTP/REST.

**المتطلبات الأساسية:**
- مشروع `RubikCare.Api.Web` (ASP.NET Core 10)
- Application Pool بـ "No Managed Code"
- `appsettings.{Environment}.json` صحيح لكل بيئة

---

### 🚀 3.1 إجراءات ما قبل النشر (Pre-Deployment Checklist)

#### ✅ التحقق من الكود

```powershell
# 1. تأكد من عدم وجود أخطاء بناء
cd C:\RC\Rubikcare.Full.Migration
dotnet build RubikCare.Api.Web -c Release

# 2. تحقق من Migrations غير مطبقة
dotnet ef migrations list --project RubikCare.Infrastructure --startup-project RubikCare.Api.Web

# 3. اختبر محلياً
dotnet run --project RubikCare.Api.Web
```

#### ✅ التحقق من قاعدة البيانات

```sql
-- هل هناك Migrations معلقة؟
SELECT * FROM __EFMigrationsHistory 
ORDER BY MigrationId DESC;

-- نسخة احتياطية قبل النشر (إلزامي!)
BACKUP DATABASE RubikCare_UAT 
TO DISK = 'C:\Backups\RubikCare_UAT_PreDeploy.bak' 
WITH INIT, COMPRESSION;
```

#### ✅ التحقق من ملفات البيئة

```powershell
# تأكد من وجود ملفات البيئة الصحيحة
Get-ChildItem "C:\RC\Rubikcare.Full.Migration\RubikCare.Api.Web" -Filter "appsettings.*.json"
```

**الملفات المطلوبة:**
- `appsettings.json` (الإعدادات العامة)
- `appsettings.Development.json` (للتطوير)
- `appsettings.Test.json` (لبيئة UAT)
- `appsettings.Production.json` (للإنتاج)

---

### 📦 3.2 خطوات النشر

#### الخطوة 1: إيقاف الـ App Pool

```powershell
# UAT API
C:\Windows\System32\inetsrv\appcmd stop apppool "RubikCareUat"

# Production API
C:\Windows\System32\inetsrv\appcmd stop apppool "RubikCareApi"
```

#### الخطوة 2: النشر من Visual Studio أو CLI

**الخيار أ: من Visual Studio**
1. كليك يمين على `RubikCare.Api.Web` → **Publish**
2. اختر **Folder** → `E:\rubikans\Publish\Api`
3. اضغط **Publish**

**الخيار ب: من CLI (الأفضل للأتمتة)**

```powershell
cd C:\RC\Rubikcare.Full.Migration

# UAT Environment
dotnet publish RubikCare.Api.Web -c Release -o E:\rubikans\Publish\ApiUat -r win-x64 --self-contained false

# Production Environment
dotnet publish RubikCare.Api.Web -c Release -o E:\rubikans\Publish\ApiLive -r win-x64 --self-contained false
```

#### الخطوة 3: نسخ الملفات إلى السيرفر

```powershell
# UAT API
$source = "E:\rubikans\Publish\ApiUat"
$destination = "C:\WebSite\RubikCareUat"

# ⚠️ احفظ ملفات البيئة الحالية
Copy-Item "$destination\appsettings.Test.json" "$destination\appsettings.Test.json.backup" -Force

# انسخ الملفات الجديدة (باستثناء ملفات البيئة)
Get-ChildItem $source -Exclude "appsettings.*.json" | Copy-Item -Destination $destination -Recurse -Force

# Production API
$source = "E:\rubikans\Publish\ApiLive"
$destination = "C:\WebSite\RubikCareApi"

Copy-Item "$destination\appsettings.Production.json" "$destination\appsettings.Production.json.backup" -Force
Get-ChildItem $source -Exclude "appsettings.*.json" | Copy-Item -Destination $destination -Recurse -Force
```

**⚠️ تحذير حرج:**
> **لا تستبدل `appsettings.{Environment}.json`** إلا إذا كنت متأكداً 100% أن الإعدادات المحلية مطابقة للسيرفر (خاصة Connection Strings و JWT Keys).

#### الخطوة 4: تطبيق Migrations (إذا لزم الأمر)

```powershell
# من جهاز التطوير - تطبيق Migrations على قاعدة بيانات UAT
dotnet ef database update --project RubikCare.Infrastructure --startup-project RubikCare.Api.Web -- --environment Test

# من جهاز التطوير - تطبيق Migrations على قاعدة بيانات Production
dotnet ef database update --project RubikCare.Infrastructure --startup-project RubikCare.Api.Web -- --environment Production
```

**⚠️ تحذير:**
> **لا تطبّق Migrations على Production مباشرة** دون اختبارها على UAT أولاً.

#### الخطوة 5: تشغيل الـ App Pool

```powershell
# UAT API
C:\Windows\System32\inetsrv\appcmd start apppool "RubikCareUat"

# Production API
C:\Windows\System32\inetsrv\appcmd start apppool "RubikCareApi"
```

---

### 🧪 3.3 التحقق بعد النشر (Post-Deployment Verification)

#### ✅ اختبار Health Check

```powershell
# UAT
try {
    $r = Invoke-WebRequest -Uri "https://uat.rubikcare.com/health" -UseBasicParsing -TimeoutSec 10
    "✅ UAT API: $($r.StatusCode) - $($r.Content)"
} catch {
    "❌ UAT API: $($_.Exception.Message)"
}

# Production
try {
    $r = Invoke-WebRequest -Uri "https://api.rubikcare.com/health" -UseBasicParsing -TimeoutSec 10
    "✅ Production API: $($r.StatusCode) - $($r.Content)"
} catch {
    "❌ Production API: $($_.Exception.Message)"
}
```

#### ✅ اختبار Endpoint حرج

```powershell
# اختبار endpoint بسيط (مثل جلب الإعدادات العامة)
try {
    $r = Invoke-WebRequest -Uri "https://uat.rubikcare.com/api/settings/public" -UseBasicParsing -TimeoutSec 10
    "✅ Settings API: $($r.StatusCode)"
} catch {
    "❌ Settings API: $($_.Exception.Message)"
}
```

#### ✅ التحقق من السجلات

```powershell
# آخر 50 سطر من سجل stdout
Get-Content "C:\WebSite\RubikCareUat\logs\stdout_*.log" -Tail 50

# Event Viewer (للأخطاء الحرجة)
Get-EventLog -LogName Application -Source "ASP.NET*" -Newest 10 | Format-List
```

---

### 🐛 3.4 الأخطاء الشائعة وحلولها

#### ❌ الخطأ 1: `502.5 - Process Failure`

**العرض:**
```
HTTP Error 502.5 - Process Failure
Common causes of this issue:
- The application process failed to start
```

**السبب:**
- .NET Runtime غير مثبت أو إصدار خاطئ
- خطأ في `Program.cs` يمنع بدء التطبيق
- Connection String خاطئ

**الحل:**

```powershell
# 1. تحقق من .NET Runtime
dotnet --list-runtimes

# 2. فعّل stdout logging في web.config
# <aspNetCore stdoutLogEnabled="true" stdoutLogFile=".\logs\stdout" />

# 3. افحص السجلات
Get-ChildItem "C:\WebSite\RubikCareUat\logs" -Filter "stdout_*.log" | 
    Sort-Object LastWriteTime -Descending | 
    Select-Object -First 1 | 
    Get-Content -Tail 100
```

---

#### ❌ الخطأ 2: `401 Unauthorized` على endpoints صحيحة

**السبب:**
- JWT Key مختلف بين environments
- Token منتهي الصلاحية
- CORS يمنع الطلب

**الحل:**

```powershell
# 1. تحقق من JWT Key في appsettings
Get-Content "C:\WebSite\RubikCareUat\appsettings.Test.json" | Select-String "Jwt:Key"

# 2. تحقق من CORS في Program.cs
Get-Content "C:\RC\Rubikcare.Full.Migration\RubikCare.Api.Web\Program.cs" | Select-String "AddCors|UseCors"

# 3. أضف CORS policy صحيح (إذا كان مفقوداً)
```

**CORS policy الصحيح في `Program.cs`:**

```csharp
builder.Services.AddCors(options =>
{
    options.AddPolicy("AllowPWA", policy =>
    {
        policy.WithOrigins(
            "https://stagepu.rubikcare.com",
            "https://test.rubikcare.com",
            "https://rubikcare.com"
        )
        .AllowAnyMethod()
        .AllowAnyHeader()
        .AllowCredentials();
    });
});

// بعد builder.Build()
app.UseCors("AllowPWA");
```

---

#### ❌ الخطأ 3: `404 Not Found` على endpoints جديدة

**السبب:**
- لم تُنشر الـ Controllers الجديدة
- Routing خاطئ
- Environment مختلف (API قديم)

**الحل:**

```powershell
# 1. تحقق من وجود الـ Controller
Get-ChildItem "C:\WebSite\RubikCareUat" -Filter "*Controller.dll" -Recurse

# 2. اختبر الـ endpoint مباشرة
Invoke-WebRequest -Uri "https://uat.rubikcare.com/api/new-endpoint" -UseBasicParsing

# 3. تحقق من Environment
Get-Content "C:\WebSite\RubikCareUat\web.config" | Select-String "ASPNETCORE_ENVIRONMENT"
```

---

### 📊 3.5 Checklist النشر النهائي

#### قبل النشر
- [ ] هل تم حفظ جميع التعديلات في الكود؟
- [ ] هل `dotnet build` ينجح بدون أخطاء؟
- [ ] هل تم أخذ نسخة احتياطية من قاعدة البيانات؟
- [ ] هل تم اختبار Migrations محلياً؟
- [ ] هل `appsettings.{Environment}.json` جاهز؟

#### أثناء النشر
- [ ] هل تم إيقاف الـ App Pool؟
- [ ] هل تم حفظ نسخة احتياطية من `appsettings`؟
- [ ] هل تم نسخ الملفات بنجاح؟
- [ ] هل تم تطبيق Migrations (إذا لزم)؟
- [ ] هل تم تشغيل الـ App Pool؟

#### بعد النشر
- [ ] هل Health Check يعيد `200`؟
- [ ] هل endpoints الحرجة تعمل؟
- [ ] هل السجلات خالية من الأخطاء؟
- [ ] هل CORS يعمل للعملاء (Web, Mobile, PWA)؟
- [ ] هل تم اختبار من عميل حقيقي؟

---

### 🚀 3.6 سكربت النشر الأوتوماتيكي

احفظ هذا السكربت كـ `deploy-api-uat.ps1`:

```powershell
# ============================================================
# سكربت نشر RubikCare API إلى بيئة UAT
# ============================================================

param(
    [string]$ProjectPath = "C:\RC\Rubikcare.Full.Migration",
    [string]$PublishPath = "E:\rubikans\Publish\ApiUat",
    [string]$IISPath = "C:\WebSite\RubikCareUat",
    [string]$AppPoolName = "RubikCareUat",
    [string]$ApiUrl = "https://uat.rubikcare.com"
)

Write-Host "================================================" -ForegroundColor Cyan
Write-Host "  RubikCare API - النشر إلى بيئة UAT" -ForegroundColor Cyan
Write-Host "================================================" -ForegroundColor Cyan

# الخطوة 1: إيقاف الـ App Pool
Write-Host "`n⏸️  الخطوة 1: إيقاف الـ App Pool..." -ForegroundColor Yellow
C:\Windows\System32\inetsrv\appcmd stop apppool $AppPoolName
Start-Sleep -Seconds 2

# الخطوة 2: نسخة احتياطية من appsettings
Write-Host "`n💾 الخطوة 2: نسخة احتياطية من appsettings..." -ForegroundColor Yellow
Copy-Item "$IISPath\appsettings.Test.json" "$IISPath\appsettings.Test.json.backup" -Force
Write-Host "✅ تم الحفظ" -ForegroundColor Green

# الخطوة 3: نشر المشروع
Write-Host "`n📦 الخطوة 3: نشر المشروع..." -ForegroundColor Yellow
Set-Location $ProjectPath
dotnet publish RubikCare.Api.Web -c Release -o $PublishPath -r win-x64 --self-contained false

if ($LASTEXITCODE -ne 0) {
    Write-Host "❌ فشل النشر!" -ForegroundColor Red
    C:\Windows\System32\inetsrv\appcmd start apppool $AppPoolName
    exit 1
}
Write-Host "✅ النشر نجح" -ForegroundColor Green

# الخطوة 4: نسخ الملفات (باستثناء appsettings)
Write-Host "`n📋 الخطوة 4: نسخ الملفات إلى IIS..." -ForegroundColor Yellow
Get-ChildItem $PublishPath -Exclude "appsettings.*.json" | Copy-Item -Destination $IISPath -Recurse -Force
Write-Host "✅ تم النسخ" -ForegroundColor Green

# الخطوة 5: تشغيل الـ App Pool
Write-Host "`n▶️  الخطوة 5: تشغيل الـ App Pool..." -ForegroundColor Yellow
C:\Windows\System32\inetsrv\appcmd start apppool $AppPoolName

# الخطوة 6: اختبار Health Check
Write-Host "`n🧪 الخطوة 6: اختبار Health Check..." -ForegroundColor Yellow
Start-Sleep -Seconds 5

try {
    $r = Invoke-WebRequest -Uri "$ApiUrl/health" -UseBasicParsing -TimeoutSec 15
    Write-Host "✅ Health Check: $($r.StatusCode) - $($r.Content)" -ForegroundColor Green
} catch {
    Write-Host "❌ Health Check: $($_.Exception.Message)" -ForegroundColor Red
}

# النتيجة النهائية
Write-Host "`n================================================" -ForegroundColor Cyan
Write-Host "  🎉 تم نشر API بنجاح!" -ForegroundColor Green
Write-Host "  🔗 الرابط: $ApiUrl" -ForegroundColor Green
Write-Host "================================================`n" -ForegroundColor Cyan
```


## 🌐 القسم 4: نشر Web (Blazor Server)

### 📋 نظرة عامة

تطبيق `RubikCare.Web` هو تطبيق **Blazor Server** يعمل عبر SignalR. يختلف نشره جوهرياً عن الـ PWA لأنه يحتاج **خادم .NET يعمل فعلياً** وليس مجرد ملفات ثابتة.

> ⚠️ **ملاحظة معمارية:** في النسخة الحالية، يعتمد `RubikCare.Web` على `appsettings.{Environment}.json` لتحديد رابط الـ API المستهدف. هذا المصدر كان سبباً لأخطر مشكلة موثقة في المنصة (الفشل الصامت) — راجع القسم 4.5.

---

### 🚀 4.1 إجراءات ما قبل النشر

#### ✅ التحقق من ملف البيئة (الأهم!)

قبل أي نشر، تأكد أن `appsettings.{Environment}.json` يشير إلى الـ API الصحيح:

```powershell
# تحقق من رابط الـ API في ملف البيئة
Get-Content "C:\RC\Rubikcare.Full.Migration\RubikCare.Web\appsettings.Test.json" | Select-String "BaseUrl"
```

**النتيجة الصحيحة لبيئة Test:**
```json
{
  "ApiSettings": {
    "BaseUrl": "https://uat.rubikcare.com"
  }
}
```

> 🔴 **قاعدة ذهبية:** تطبيق الـ Web في بيئة الـ Test **يجب** أن يكون `BaseUrl` فيه مساوياً لـ `https://uat.rubikcare.com` وليس الـ Live.

#### ✅ التحقق من الكود

```powershell
cd C:\RC\Rubikcare.Full.Migration
dotnet build RubikCare.Web -c Release
```

---

### 📦 4.2 خطوات النشر

#### الخطوة 1: إيقاف الـ App Pool

```powershell
# Test / UAT Web
C:\Windows\System32\inetsrv\appcmd stop apppool "rubikcarenew"

# Production Web
C:\Windows\System32\inetsrv\appcmd stop apppool "RubikCareLive"
```

#### الخطوة 2: النشر من Visual Studio أو CLI

**الخيار أ: من Visual Studio**
1. كليك يمين على `RubikCare.Web` → **Publish**
2. اختر **Folder** → `E:\rubikans\Publish\Web`
3. اضغط **Publish**

**الخيار ب: من CLI**

```powershell
cd C:\RC\Rubikcare.Full.Migration

# Test Environment
dotnet publish RubikCare.Web -c Release -o E:\rubikans\Publish\WebTest

# Production Environment
dotnet publish RubikCare.Web -c Release -o E:\rubikans\Publish\WebLive
```

#### الخطوة 3: نسخ الملفات إلى السيرفر (مع حماية ملفات البيئة)

```powershell
$source = "E:\rubikans\Publish\WebTest"
$destination = "C:\WebSite\RubikCareNew"

# ⚠️ احفظ ملفات البيئة الحالية (إلزامي!)
Copy-Item "$destination\web.config" "$destination\web.config.backup" -Force
Copy-Item "$destination\appsettings.Test.json" "$destination\appsettings.Test.json.backup" -Force

# انسخ الملفات الجديدة (باستثناء ملفات البيئة)
Get-ChildItem $source -Exclude "web.config","appsettings.*.json" | 
    Copy-Item -Destination $destination -Recurse -Force
```

> ⚠️ **تحذير هام:** إذا ظهر تحذير لاستبدال `web.config` أو `appsettings.Test.json`، تأكد أن النسخة المحلية محدثة (تشير إلى `uat.rubikcare.com`)، أو اختر **Skip** لهما للحفاظ على إعدادات السيرفر اليدوية.

#### الخطوة 4: تشغيل الـ App Pool

```powershell
C:\Windows\System32\inetsrv\appcmd start apppool "rubikcarenew"
```

---

### ⚙️ 4.3 ملف `web.config` للـ Blazor Server

يختلف `web.config` هنا عن الـ PWA — لأنه يستخدم **ASP.NET Core Module**:

```xml
<?xml version="1.0" encoding="utf-8"?>
<configuration>
  <location path="." inheritInChildApplications="false">
    <system.webServer>
      <handlers>
        <add name="aspNetCore" path="*" verb="*" 
             modules="AspNetCoreModuleV2" resourceType="Unspecified" />
      </handlers>
      <aspNetCore processPath="dotnet" 
                  arguments=".\RubikCare.Web.dll" 
                  stdoutLogEnabled="true" 
                  stdoutLogFile=".\logs\stdout" 
                  hostingModel="inprocess">
        <environmentVariables>
          <!-- ⭐ ضمان قراءة ملف appsettings.Test.json -->
          <environmentVariable name="ASPNETCORE_ENVIRONMENT" value="Test" />
        </environmentVariables>
      </aspNetCore>
    </system.webServer>
  </location>
</configuration>
```

**النقاط الحرجة:**

| النقطة | القيمة الصحيحة | الغرض |
|--------|---------------|-------|
| `ASPNETCORE_ENVIRONMENT` | `Test` / `Production` | ضمان قراءة ملف البيئة الصحيح |
| `stdoutLogEnabled` | `true` | تتبع أخطاء المصادقة والتوكنات |
| `hostingModel` | `inprocess` | أداء أفضل |

---

### 🧪 4.4 التحقق بعد النشر

```powershell
# 1. اختبار الصفحة الرئيسية
try {
    $r = Invoke-WebRequest -Uri "https://test.rubikcare.com" -UseBasicParsing -TimeoutSec 15
    "✅ Web: $($r.StatusCode) | Length: $($r.RawContentLength)"
} catch {
    "❌ Web: $($_.Exception.Message)"
}

# 2. التأكد من أن الـ Web يقرأ رابط الـ API الصحيح
Get-Content "C:\WebSite\RubikCareNew\appsettings.Test.json" | Select-String "BaseUrl"

# 3. التأكد من أن بيئة الـ Test مفعلة في web.config
Get-Content "C:\WebSite\RubikCareNew\web.config" | Select-String "ASPNETCORE_ENVIRONMENT"
```

---

### 🔥 4.5 الأخطاء الشائعة وحلولها (من الدروس المستفادة)

#### ❌ الخطأ 1: الفشل الصامت (Silent Failure) — أخطر مشكلة موثقة

**العرض:**
- عمليات الحفظ/الإضافة تفشل بدون أي رسالة خطأ
- لا أخطاء في الواجهة
- لا أخطاء في Console المتصفح

**السبب الجذري:** مزيج من 3 عوامل:

| # | العامل | التفاصيل |
|---|--------|----------|
| 1 | **ابتلاع الاستثناءات** | دوال `GetAsync` و `PostAsync` تعيد `null` بصمت عند فشل الطلب (404/500) بدلاً من رمي `Exception` |
| 2 | **خطأ في توجيه البيئة** | `appsettings.Test.json` يشير إلى `https://api.rubikcare.com` (Live) بدلاً من `https://uat.rubikcare.com` |
| 3 | **تضارب بيئات الـ API** | نشر كود الـ Web الجديد بينما الـ API المستهدف هو النسخة القديمة (Live) التي لا تحتوي على الـ Endpoint الجديد |

**الحل:**

```csharp
// في WebApiService.cs — رمي Exception بدلاً من العودة بـ default بصمت
public async Task<T> GetAsync<T>(string endpoint)
{
    var response = await _httpClient.GetAsync(endpoint);
    
    if (!response.IsSuccessStatusCode)
    {
        var errorContent = await response.Content.ReadAsStringAsync();
        throw new ApiException($"Request failed: {(int)response.StatusCode} - {errorContent}");
    }
    
    return await response.Content.ReadFromJsonAsync<T>();
}
```

#### ❌ الخطأ 2: `500.30 - ANCM In-Process Start Failure`

**السبب:** خطأ في `Program.cs` أو `appsettings` غير موجود

**الحل:**

```powershell
# فعّل السجلات في web.config
# <aspNetCore stdoutLogEnabled="true" stdoutLogFile=".\logs\stdout" />

# ثم افحص السجلات
Get-ChildItem -Path "C:\WebSite\RubikCareNew\logs\stdout" -Filter "*.log" | 
    Sort-Object LastWriteTime -Descending | 
    Select-Object -First 1 | 
    Get-Content -Tail 30
```

#### ❌ الخطأ 3: التطبيق يعمل لكن البيانات من بيئة خاطئة

**العرض:** البيانات المخزنة/المسترجعة من Live رغم أنك في بيئة Test

**السبب:** `appsettings.Test.json` غير موجود أو غير مقروء

**الحل:** تأكد من `ASPNETCORE_ENVIRONMENT=Test` في `web.config`

---

### 📊 4.6 Checklist النشر النهائي

#### قبل النشر
- [ ] هل تم حفظ جميع التعديلات في الكود؟
- [ ] هل `BaseUrl` في `appsettings.Test.json` يشير إلى `uat.rubikcare.com`؟
- [ ] هل تم اختبار التطبيق محلياً؟

#### أثناء النشر
- [ ] هل تم إيقاف الـ App Pool؟
- [ ] هل تم حفظ نسخة احتياطية من `web.config` و `appsettings`؟
- [ ] هل تم نسخ الملفات بنجاح؟
- [ ] هل تم تشغيل الـ App Pool؟

#### بعد النشر
- [ ] هل الصفحة الرئيسية تعمل؟
- [ ] هل `BaseUrl` صحيح على السيرفر؟
- [ ] هل `ASPNETCORE_ENVIRONMENT` صحيح؟
- [ ] هل السجلات خالية من الأخطاء؟
- [ ] هل العمليات (حفظ/إضافة) تعمل فعلياً (اختبار الفشل الصامت)؟

---

### 🔍 4.7 أوامر تشخيص سريعة (Troubleshooting Cheatsheet)

```powershell
# 1. التأكد من أن الـ Web يقرأ رابط الـ API الصحيح
Get-Content "C:\WebSite\RubikCareNew\appsettings.Test.json" | Select-String "BaseUrl"

# 2. التأكد من أن بيئة الـ Test مفعلة في web.config
Get-Content "C:\WebSite\RubikCareNew\web.config" | Select-String "ASPNETCORE_ENVIRONMENT"

# 3. فحص آخر أخطاء الـ Blazor/Server في السجلات
Get-ChildItem -Path "C:\WebSite\RubikCareNew\logs\stdout" -Filter "*.log" | 
    Sort-Object LastWriteTime -Descending | 
    Select-Object -First 1 | 
    Get-Content -Tail 30
```

---
## 📱 القسم 5: نشر PWA (Blazor WebAssembly) ⭐ الأهم

### 📋 نظرة عامة: لماذا نشر PWA مختلف جذرياً؟

تطبيق `RubikCare.PWA` هو تطبيق **Blazor WebAssembly Standalone**. يختلف نشره **كلياً** عن الـ Web والـ API:

| الجانب | Web / API | PWA (Blazor WASM) |
|--------|-----------|-------------------|
| **طبيعة التشغيل** | خادم .NET يعمل فعلياً | ملفات ثابتة تُخدم من IIS |
| **الـ App Pool** | يحتاج .NET Runtime | **No Managed Code** |
| **`web.config`** | ASP.NET Core Module | **URL Rewrite + MIME types** |
| **منطق التطبيق** | يعمل على الخادم | يعمل في المتصفح (يُحمَّل كـ `.wasm`) |
| **أكبر تحدٍّ** | إعدادات البيئة | **خدمة ملفات `_framework` بشكل صحيح** |

> 🔴 **القاعدة المحورية:** تطبيق الـ PWA لا يحتاج خادم .NET، لكنه يحتاج **إعداد IIS دقيقاً** ليخدم ملفات `.wasm` و `.dat` و `.js` بشكل صحيح. أي خطأ هنا = شاشة زرقاء (فشل إقلاع الـ Runtime).

---

### 🏗️ 5.1 المتطلبات الأساسية (قبل البدء)

#### 1. تثبيت URL Rewrite Module (إلزامي)

بدون هذه الوحدة، لن يعمل `web.config` الخاص بالـ PWA وستحصل على خطأ 500.

```powershell
# التحقق من وجود الوحدة
Get-WebGlobalModule | Where-Object { $_.Name -like "*Rewrite*" }

# التحقق من وجود ملف الـ DLL
Test-Path "C:\Windows\System32\inetsrv\rewrite.dll"
```

إذا لم تظهر، ثبّتها من:
```
https://www.iis.net/downloads/microsoft/url-rewrite
```

#### 2. إنشاء App Pool بوضع "No Managed Code"

```powershell
C:\Windows\System32\inetsrv\appcmd add apppool `
    /name:"PU_RubicCareStage" `
    /managedRuntimeVersion:"" `
    /managedPipelineMode:"Integrated"
```

> ⚠️ **`managedRuntimeVersion:""` تعني "No Managed Code"** — وهذا إلزامي للـ PWA لأنه مجرد ملفات ثابتة.

#### 3. إنشاء الموقع وربطه

```powershell
# إنشاء الموقع
C:\Windows\System32\inetsrv\appcmd add site `
    /name:"PU_RubicCareStage" `
    /physicalPath:"C:\WebSite\PU_RubicCareStage" `
    /bindings:https/*:443:stagepu.rubikcare.com

# ربط الموقع بالـ App Pool
C:\Windows\System32\inetsrv\appcmd set app "PU_RubicCareStage/" `
    /applicationPool:"PU_RubicCareStage"
```

#### 4. تثبيت شهادة SSL

عبر **IIS Manager**:
1. افتح **Server Certificates**
2. استورد شهادة `stagepu.rubikcare.com`
3. اربطها بالموقع في **Bindings** (Port 443)

---

### 🎯 5.2 القاعدة الذهبية: المسار الفيزيائي

> 🔴 **أكبر خطأ واجهناه:** كان المسار الفيزيائي يشير إلى `wwwroot` بدلاً من الجذر.

#### ❌ الخطأ (يسبب 404 لملفات الـ framework)

```powershell
# خاطئ — لا تفعل هذا
C:\WebSite\PU_RubicCareStage\wwwroot
```

**لماذا؟** لأن `web.config` يحتوي على قاعدة `Serve subdir` التي تعيد كتابة الطلبات إلى `wwwroot\{R:0}`. إذا كان المسار الفيزيائي أصلاً `wwwroot`، فإن القاعدة ستبحث في `wwwroot\wwwroot\...` وهو غير موجود → **404**.

#### ✅ الصحيح

```powershell
# صحيح — المسار الفيزيائي هو الجذر
C:\WebSite\PU_RubicCareStage
```

**للتحقق من المسار الحالي:**
```powershell
C:\Windows\System32\inetsrv\appcmd list vdir "PU_RubicCareStage/" /text:physicalPath
```
**النتيجة الصحيحة:** `C:\WebSite\PU_RubicCareStage` (بدون `\wwwroot`)

**للتصحيح إذا كان خاطئاً:**
```powershell
C:\Windows\System32\inetsrv\appcmd set vdir "PU_RubicCareStage/" /physicalPath:"C:\WebSite\PU_RubicCareStage"
```

#### هيكل المجلد الصحيح على السيرفر

```
C:\WebSite\PU_RubicCareStage\          ← المسار الفيزيائي (الجذر)
│
├── web.config                         ← إعدادات IIS (لا يُحذف أبداً)
│
└── wwwroot\
    ├── index.html                     ← نقطة الدخول
    ├── manifest.webmanifest           ← PWA manifest
    ├── service-worker.js              ← Service Worker
    ├── icon-192.png, icon-512.png     ← أيقونات PWA
    │
    ├── _framework\                    ← ملفات Blazor Runtime (الحرجة)
    │   ├── blazor.webassembly.js
    │   ├── dotnet.js
    │   ├── dotnet.runtime.{hash}.js
    │   ├── dotnet.native.{hash}.js
    │   ├── RubikCare.PWA.{hash}.wasm  ← كود التطبيق المُجمَّع
    │   ├── icudt_EFIGS.{hash}.dat     ← بيانات ICU للترميز
    │   ├── icudt_CJK.{hash}.dat
    │   └── icudt_no_CJK.{hash}.dat
    │
    └── _content\RubikCare.Shared.UI\  ← ملفات المكتبة المشتركة
        ├── css\
        ├── plugins\
        └── Images\
```

---

### 📄 5.3 ملف `web.config` الكامل والصحيح

> 🔴 **هذا الملف هو قلب النشر.** احفظ هذه النسخة واجعلها جزءاً من المشروع.

```xml
<?xml version="1.0" encoding="UTF-8"?>
<configuration>
  <system.webServer>
    <staticContent>
      <remove fileExtension=".blat" />
      <remove fileExtension=".dat" />
      <remove fileExtension=".dll" />
      <remove fileExtension=".webcil" />
      <remove fileExtension=".json" />
      <remove fileExtension=".wasm" />
      <remove fileExtension=".woff" />
      <remove fileExtension=".woff2" />
      <remove fileExtension=".webmanifest" />
      <mimeMap fileExtension=".blat" mimeType="application/octet-stream" />
      <mimeMap fileExtension=".dll" mimeType="application/octet-stream" />
      <mimeMap fileExtension=".webcil" mimeType="application/octet-stream" />
      <mimeMap fileExtension=".dat" mimeType="application/octet-stream" />
      <mimeMap fileExtension=".json" mimeType="application/json" />
      <mimeMap fileExtension=".wasm" mimeType="application/wasm" />
      <mimeMap fileExtension=".woff" mimeType="application/font-woff" />
      <mimeMap fileExtension=".woff2" mimeType="application/font-woff" />
      <mimeMap fileExtension=".webmanifest" mimeType="application/manifest+json" />
    </staticContent>
    <httpCompression>
      <dynamicTypes>
        <add mimeType="application/octet-stream" enabled="true" />
        <add mimeType="application/wasm" enabled="true" />
      </dynamicTypes>
    </httpCompression>
    <rewrite>
      <rules>
        <rule name="Serve subdir" stopProcessing="true">
          <match url=".*" />
          <action type="Rewrite" url="wwwroot\{R:0}" />
        </rule>
        <rule name="SPA fallback routing" stopProcessing="true">
          <match url=".*" />
          <conditions logicalGrouping="MatchAll">
            <add input="{REQUEST_FILENAME}" matchType="IsFile" negate="true" />
          </conditions>
          <action type="Rewrite" url="wwwroot\" />
        </rule>
      </rules>
    </rewrite>
  </system.webServer>
</configuration>
```

#### شرح القواعد الحرجة

| القاعدة | الوظيفة | لماذا حرجة؟ |
|---------|---------|-------------|
| `Serve subdir` | تعيد كتابة كل طلب إلى `wwwroot\{R:0}` | بدونها، لن يجد IIS الملفات داخل `wwwroot` |
| `SPA fallback routing` | للروابط غير الملفات (مثل `/psp-entry`) تعيد `index.html` | بدونها، تفشل التنقلات الداخلية (Routing) |
| `stopProcessing="true"` | يوقف معالجة القواعد اللاحقة | **إلزامي** لمنع تضارب القاعدتين |

#### ⚠️ نقاط حرجة في `web.config`

| النقطة | القيمة الصحيحة | إذا كانت خاطئة |
|--------|---------------|-----------------|
| `stopProcessing="true"` في `Serve subdir` | ✅ يجب أن تكون موجودة | تضارب مع `SPA fallback` → سلوك غير متوقع |
| MIME types لـ `.wasm`, `.dat`, `.webmanifest` | ✅ يجب أن تكون موجودة | **404** على ملفات الـ Runtime |
| ترتيب القواعد | `Serve subdir` أولاً، ثم `SPA fallback` | فشل خدمة الملفات |

#### 💡 اجعل `web.config` جزءاً من المشروع (الحل الدائم)

لتجنب فقدان `web.config` أو الحاجة لتعديله يدوياً على السيرفر، ضعه داخل مجلد المشروع `RubikCare.PWA/` وأضفه إلى `.csproj`:

```xml
<ItemGroup>
  <Content Include="web.config">
    <CopyToPublishDirectory>PreserveNewest</CopyToPublishDirectory>
  </Content>
</ItemGroup>
```

بهذه الطريقة، `dotnet publish` سينسخه تلقائياً مع كل نشر.

---

### 🔢 5.4 مشكلة الأسماء المُجزّأة (Hashed Filenames) والحل

#### المشكلة

عند النشر، يولّد .NET ملفات بأسماء مُجزّأة (بصمة) مثل:
```
blazor.webassembly.w3qd1tpl0e.js
dotnet.f17erswj0l.js
dotnet.runtime.zbexyp8zrs.js
RubikCare.PWA.s8e7likdzs.wasm
```

لكن `index.html` يطلب الاسم **الثابت** `blazor.webassembly.js` (بدون hash). إذا لم يوجد هذا الملف الثابت، يفشل الإقلاع.

#### الحل: إضافة Target في `.csproj` لنسخ الأسماء الثابتة

أضف هذا الـ Target في `RubikCare.PWA.csproj` (قبل `</Project>`):

```xml
<Target Name="CopyBlazorAssets" AfterTargets="Publish">
    <ItemGroup>
      <_BlazorJsFiles Include="$(PublishDir)wwwroot\_framework\blazor.webassembly.*.js"
                      Exclude="$(PublishDir)wwwroot\_framework\blazor.webassembly.js" />
      <_DotnetJsFiles Include="$(PublishDir)wwwroot\_framework\dotnet.*.js"
                      Exclude="$(PublishDir)wwwroot\_framework\dotnet.js;$(PublishDir)wwwroot\_framework\dotnet.runtime.*.js;$(PublishDir)wwwroot\_framework\dotnet.native.*.js" />
      <_DotnetRuntimeFiles Include="$(PublishDir)wwwroot\_framework\dotnet.runtime.*.js"
                           Exclude="$(PublishDir)wwwroot\_framework\dotnet.runtime.js" />
      <_DotnetNativeFiles Include="$(PublishDir)wwwroot\_framework\dotnet.native.*.js"
                          Exclude="$(PublishDir)wwwroot\_framework\dotnet.native.js" />
    </ItemGroup>

    <Copy SourceFiles="@(_BlazorJsFiles)"
          DestinationFiles="$(PublishDir)wwwroot\_framework\blazor.webassembly.js"
          Condition="@(_BlazorJsFiles->Count()) == 1" />

    <Copy SourceFiles="@(_DotnetJsFiles)"
          DestinationFiles="$(PublishDir)wwwroot\_framework\dotnet.js"
          Condition="@(_DotnetJsFiles->Count()) == 1" />

    <Copy SourceFiles="@(_DotnetRuntimeFiles)"
          DestinationFiles="$(PublishDir)wwwroot\_framework\dotnet.runtime.js"
          Condition="@(_DotnetRuntimeFiles->Count()) == 1" />

    <Copy SourceFiles="@(_DotnetNativeFiles)"
          DestinationFiles="$(PublishDir)wwwroot\_framework\dotnet.native.js"
          Condition="@(_DotnetNativeFiles->Count()) == 1" />

    <Message Text="✅ تم نسخ ملفات Blazor بالأسماء الثابتة" Importance="high" />
</Target>
```

> ⚠️ **ملاحظة:** هذا الـ Target يجب أن يكون ابناً مباشراً لعنصر `<Project>`، وليس داخل `<ItemGroup>` أو `<PropertyGroup>` — وإلا ستحصل على خطأ `The element <Target> is unrecognized`.

---

هل تريد أن أكمل بقية القسم 5 (خطوات النشر + سكربت الأتمتة + الأخطاء الشائعة + المحاذير)؟
