
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

### 📋 5.5 إجراءات ما قبل النشر (Pre-Deployment Checklist)

#### ✅ التحقق من الكود

```powershell
# 1. تأكد من عدم وجود أخطاء بناء
cd C:\RC\Rubikcare.Full.Migration
dotnet build RubikCare.PWA -c Release

# 2. اختبر محلياً (يفتح على localhost)
dotnet run --project RubikCare.PWA
```

#### ✅ التحقق من وجود `web.config` في المشروع

```powershell
Test-Path "C:\RC\Rubikcare.Full.Migration\RubikCare.PWA\web.config"
```

**النتيجة المطلوبة:** `True`

> ⚠️ إذا كان `False`، فستفقد إعدادات IIS عند كل نشر. أنشئ `web.config` في مجلد المشروع وأضفه إلى `.csproj` (راجع القسم 5.3).

#### ✅ التحقق من `CopyBlazorAssets` Target في `.csproj`

```powershell
Select-String -Path "C:\RC\Rubikcare.Full.Migration\RubikCare.PWA\RubikCare.PWA.csproj" -Pattern "CopyBlazorAssets"
```

**النتيجة المطلوبة:** يجب أن يظهر `<Target Name="CopyBlazorAssets" ...>`

#### ✅ التحقق من رابط الـ API في الكود

تأكد أن `RubikCare.PWA` يشير إلى الـ API الصحيح لبيئة النشر. في بناء Release، يجب أن يكون:

```csharp
// في ApiService أو ملف الإعدادات
apiUrl = "https://uat.rubikcare.com/";  // لبيئة Stage
```

> 🔴 **قاعدة ذهبية:** تطبيق الـ PWA في بيئة Stage **يجب** أن يتصل بـ `https://uat.rubikcare.com` وليس الـ Live.

---

### 📦 5.6 خطوات النشر (كل مرة)

#### الخطوة 1: إيقاف الـ App Pool (لمنع قفل الملفات)

```powershell
C:\Windows\System32\inetsrv\appcmd stop apppool "PU_RubicCareStage"
Start-Sleep -Seconds 2
```

> ⚠️ **لماذا؟** ملفات `.wasm` و `.js` قد تكون مقفولة من قبل عملية `w3wp.exe`. النسخ أثناء التشغيل قد يفشل أو ينتج ملفات ناقصة.

#### الخطوة 2: مسح محتوى المجلد (باستثناء `web.config`)

```powershell
Get-ChildItem "C:\WebSite\PU_RubicCareStage" -Exclude "web.config" | 
    Remove-Item -Recurse -Force
```

> 🔴 **هذا إلزامي!** ملفات الـ Blazor لها أسماء مُجزّأة (مثل `dotnet.runtime.zbexyp8zrs.js`) تتغير مع كل نشر. إذا لم تمسح المجلد، ستبقى ملفات قديمة بـ hash قديم، ويطلبها المتصفح ثم يحصل على **خطأ `SRI integrity checks failed`**.

#### الخطوة 3: نشر المشروع

```powershell
cd C:\RC\Rubikcare.Full.Migration
dotnet publish RubikCare.PWA -c Release -o E:\rubikans\Publish\PWA
```

**تحقق من نجاح النشر:**
```powershell
if ($LASTEXITCODE -eq 0) { "✅ النشر نجح" } else { "❌ النشر فشل" }
```

> 💡 إذا أضفت الـ `CopyBlazorAssets` Target (القسم 5.4)، فستظهر رسالة `✅ تم نسخ ملفات Blazor بالأسماء الثابتة` في نهاية النشر.

#### الخطوة 4: نسخ الملفات إلى IIS

```powershell
Copy-Item -Path "E:\rubikans\Publish\PWA\*" -Destination "C:\WebSite\PU_RubicCareStage\" -Recurse -Force
```

#### الخطوة 5: تشغيل الـ App Pool

```powershell
C:\Windows\System32\inetsrv\appcmd start apppool "PU_RubicCareStage"
```

#### الخطوة 6: الاختبار من نافذة InPrivate

> 🔴 **إلزامي:** اختبر دائماً من نافذة **InPrivate** (`Ctrl + Shift + N`). المتصفح العادي يحتوي على **Service Worker** مخزّن من النشر السابق، وسيخدم ملفات قديمة حتى بعد النشر الجديد.

---

### 🤖 5.7 سكربت النشر الأوتوماتيكي (`deploy-stage.ps1`)

احفظ هذا السكربت في جذر المشروع. ينفّذ كل الخطوات أعلاه تلقائياً مع التحقق:

```powershell
# ============================================================
# سكربت نشر RubikCare PWA إلى بيئة Stage
# ============================================================

param(
    [string]$ProjectPath = "C:\RC\Rubikcare.Full.Migration",
    [string]$PublishPath = "E:\rubikans\Publish\PWA",
    [string]$IISPath = "C:\WebSite\PU_RubicCareStage",
    [string]$AppPoolName = "PU_RubicCareStage",
    [string]$SiteUrl = "https://stagepu.rubikcare.com"
)

Write-Host "================================================" -ForegroundColor Cyan
Write-Host "  RubikCare PWA - النشر إلى بيئة Stage" -ForegroundColor Cyan
Write-Host "================================================`n" -ForegroundColor Cyan

# الخطوة 1: إيقاف الـ App Pool
Write-Host "⏸️  الخطوة 1: إيقاف الـ App Pool..." -ForegroundColor Yellow
C:\Windows\System32\inetsrv\appcmd stop apppool $AppPoolName
Start-Sleep -Seconds 2

# الخطوة 2: مسح المجلد (باستثناء web.config)
Write-Host "`n🗑️  الخطوة 2: مسح المجلد القديم..." -ForegroundColor Yellow
Get-ChildItem "$IISPath" -Exclude "web.config" | Remove-Item -Recurse -Force
Write-Host "✅ تم المسح (احتفظنا بـ web.config)" -ForegroundColor Green

# الخطوة 3: نشر المشروع
Write-Host "`n📦 الخطوة 3: نشر المشروع..." -ForegroundColor Yellow
Set-Location $ProjectPath
dotnet publish RubikCare.PWA -c Release -o $PublishPath

if ($LASTEXITCODE -ne 0) {
    Write-Host "❌ فشل النشر!" -ForegroundColor Red
    C:\Windows\System32\inetsrv\appcmd start apppool $AppPoolName
    exit 1
}
Write-Host "✅ النشر نجح" -ForegroundColor Green

# الخطوة 4: نسخ الملفات إلى IIS
Write-Host "`n📋 الخطوة 4: نسخ الملفات إلى IIS..." -ForegroundColor Yellow
Copy-Item -Path "$PublishPath\*" -Destination $IISPath -Recurse -Force
Write-Host "✅ تم النسخ" -ForegroundColor Green

# الخطوة 5: التحقق من الملفات الحرجة
Write-Host "`n🔍 الخطوة 5: التحقق من الملفات الحرجة..." -ForegroundColor Yellow

$criticalFiles = @(
    "$IISPath\wwwroot\index.html",
    "$IISPath\wwwroot\manifest.webmanifest",
    "$IISPath\wwwroot\service-worker.js",
    "$IISPath\web.config"
)

$allFound = $true
foreach ($file in $criticalFiles) {
    if (Test-Path $file) {
        Write-Host "  ✅ $(Split-Path $file -Leaf)" -ForegroundColor Green
    } else {
        Write-Host "  ❌ $(Split-Path $file -Leaf) - مفقود!" -ForegroundColor Red
        $allFound = $false
    }
}

# التحقق من ملفات الـ Runtime
$wasmFiles = Get-ChildItem "$IISPath\wwwroot\_framework" -Filter "*.wasm" -ErrorAction SilentlyContinue
$datFiles  = Get-ChildItem "$IISPath\wwwroot\_framework" -Filter "icudt_*.dat" -ErrorAction SilentlyContinue

if ($wasmFiles.Count -gt 0) { Write-Host "  ✅ ملفات .wasm: $($wasmFiles.Count)" -ForegroundColor Green }
else { Write-Host "  ❌ لا توجد ملفات .wasm!" -ForegroundColor Red; $allFound = $false }

if ($datFiles.Count -gt 0) { Write-Host "  ✅ ملفات ICU (.dat): $($datFiles.Count)" -ForegroundColor Green }
else { Write-Host "  ❌ لا توجد ملفات ICU!" -ForegroundColor Red; $allFound = $false }

# الخطوة 6: تشغيل الـ App Pool
Write-Host "`n▶️  الخطوة 6: تشغيل الـ App Pool..." -ForegroundColor Yellow
C:\Windows\System32\inetsrv\appcmd start apppool $AppPoolName

# الخطوة 7: اختبار الروابط
Write-Host "`n🧪 الخطوة 7: اختبار الروابط..." -ForegroundColor Yellow
Start-Sleep -Seconds 3

$tests = @(
    @{ Name = "الصفحة الرئيسية";        Url = "$SiteUrl/" },
    @{ Name = "blazor.webassembly.js";  Url = "$SiteUrl/_framework/blazor.webassembly.js" },
    @{ Name = "manifest.webmanifest";   Url = "$SiteUrl/manifest.webmanifest" },
    @{ Name = "service-worker.js";      Url = "$SiteUrl/service-worker.js" }
)

$allPassed = $true
foreach ($test in $tests) {
    try {
        $r = Invoke-WebRequest -Uri $test.Url -UseBasicParsing -TimeoutSec 15
        Write-Host "  ✅ $($test.Name): $($r.StatusCode)" -ForegroundColor Green
    } catch {
        Write-Host "  ❌ $($test.Name): $($_.Exception.Message)" -ForegroundColor Red
        $allPassed = $false
    }
}

# النتيجة النهائية
Write-Host "`n================================================" -ForegroundColor Cyan
if ($allFound -and $allPassed) {
    Write-Host "  🎉 تم النشر بنجاح!" -ForegroundColor Green
    Write-Host "  📱 الرابط: $SiteUrl" -ForegroundColor Green
    Write-Host "  ⚠️  تذكر: اختبر من نافذة InPrivate" -ForegroundColor Yellow
} else {
    Write-Host "  ⚠️  بعض الفحوصات فشلت - راجع الأخطاء أعلاه" -ForegroundColor Yellow
}
Write-Host "================================================`n" -ForegroundColor Cyan
```

**التشغيل:**
```powershell
powershell -ExecutionPolicy Bypass -File ".\deploy-stage.ps1"
```

---

### 🧪 5.8 التحقق بعد النشر (Post-Deployment Verification)

#### الفحص 1: حالة الموقع والـ App Pool

```powershell
C:\Windows\System32\inetsrv\appcmd list site "PU_RubicCareStage"
C:\Windows\System32\inetsrv\appcmd list apppool "PU_RubicCareStage"
```

#### الفحص 2: المسار الفيزيائي (الأهم)

```powershell
C:\Windows\System32\inetsrv\appcmd list vdir "PU_RubicCareStage/" /text:physicalPath
```

**النتيجة الصحيحة:** `C:\WebSite\PU_RubicCareStage` (بدون `\wwwroot`)

#### الفحص 3: ملفات الـ Runtime الحرجة

```powershell
# ملفات .wasm
Get-ChildItem "C:\WebSite\PU_RubicCareStage\wwwroot\_framework" -Filter "*.wasm" | Select-Object Name, Length

# ملفات ICU (.dat)
Get-ChildItem "C:\WebSite\PU_RubicCareStage\wwwroot\_framework" -Filter "icudt_*.dat" | Select-Object Name, Length
```

#### الفحص 4: اختبار الروابط من السيرفر مباشرة

```powershell
$urls = @(
    "https://stagepu.rubikcare.com/",
    "https://stagepu.rubikcare.com/_framework/blazor.webassembly.js",
    "https://stagepu.rubikcare.com/_framework/dotnet.js",
    "https://stagepu.rubikcare.com/manifest.webmanifest",
    "https://stagepu.rubikcare.com/service-worker.js",
    "https://stagepu.rubikcare.com/icon-192.png"
)

foreach ($url in $urls) {
    try {
        $r = Invoke-WebRequest -Uri $url -UseBasicParsing -TimeoutSec 10
        Write-Host "✅ $url → $($r.StatusCode)" -ForegroundColor Green
    } catch {
        Write-Host "❌ $url → $($_.Exception.Message)" -ForegroundColor Red
    }
}
```

#### الفحص 5: محتوى `web.config`

```powershell
Get-Content "C:\WebSite\PU_RubicCareStage\web.config" | Select-String "stopProcessing|mimeMap|rule name"
```

**النتيجة المطلوبة:** يجب أن تظهر قاعدتا `Serve subdir` و `SPA fallback routing` مع `stopProcessing="true"`.

---

### 🐛 5.9 الأخطاء الشائعة وحلولها (القسم الأهم)

#### ❌ الخطأ 1: شاشة زرقاء + `Unexpected token '<'`

**العرض في Console:**
```
blazor.webassembly.js:1 Uncaught SyntaxError: Unexpected token '<'
```

**السبب:** المتصفح يتلقى `index.html` (يبدأ بـ `<!DOCTYPE`) بدلاً من ملف JavaScript. هذا يحدث عندما تفشل قاعدة `Serve subdir` في إيجاد الملف، فيُعاد `index.html` عبر `SPA fallback`.

**التشخيص:**
```powershell
# هل ملف بلوزر موجود فعلاً؟
Test-Path "C:\WebSite\PU_RubicCareStage\wwwroot\_framework\blazor.webassembly.js"
```

**الحل:**
1. إذا كان الملف **غير موجود** → أعد النشر (تأكد من وجود الـ `CopyBlazorAssets` Target).
2. إذا كان **موجوداً** → تحقق من `web.config` يحتوي على `stopProcessing="true"` في قاعدة `Serve subdir`.
3. تحقق من المسار الفيزيائي (القسم 5.2).

---

#### ❌ الخطأ 2: `SRI integrity checks failed` + ملفات `.dat` أو `.wasm` ترجع 404

**العرض في Console:**
```
Failed to find a valid digest in the 'integrity' attribute for resource '...RubikCare.PWA.xxx.wasm'
Failed to fetch. SRI's integrity checks failed.
```

**السبب:** أحد أمرين:
1. **المسار الفيزيائي خاطئ** (يشير إلى `wwwroot` بدلاً من الجذر) → قاعدة `Serve subdir` تبحث في `wwwroot\wwwroot\...`.
2. **ملفات قديمة بـ hash مختلف** ما زالت موجودة من نشر سابق.

**الحل:**
```powershell
# 1. تحقق من المسار الفيزيائي
C:\Windows\System32\inetsrv\appcmd list vdir "PU_RubicCareStage/" /text:physicalPath
# يجب أن يكون: C:\WebSite\PU_RubicCareStage (بدون \wwwroot)

# 2. إذا كان خاطئاً، صححه
C:\Windows\System32\inetsrv\appcmd set vdir "PU_RubicCareStage/" /physicalPath:"C:\WebSite\PU_RubicCareStage"

# 3. امسح المجلد وأعد النشر (لإزالة ملفات الـ hash القديم)
C:\Windows\System32\inetsrv\appcmd stop apppool "PU_RubicCareStage"
Get-ChildItem "C:\WebSite\PU_RubicCareStage" -Exclude "web.config" | Remove-Item -Recurse -Force
Copy-Item -Path "E:\rubikans\Publish\PWA\*" -Destination "C:\WebSite\PU_RubicCareStage\" -Recurse -Force
C:\Windows\System32\inetsrv\appcmd start apppool "PU_RubicCareStage"

# 4. اختبر من نافذة InPrivate
```

---

#### ❌ الخطأ 3: `manifest.webmanifest` يرجع 404

**السبب:** IIS لا يعرف MIME type للامتداد `.webmanifest` افتراضياً.

**الحل:** تأكد من وجود هذين السطرين في `<staticContent>` في `web.config`:
```xml
<remove fileExtension=".webmanifest" />
<mimeMap fileExtension=".webmanifest" mimeType="application/manifest+json" />
```

**التحقق:**
```powershell
try {
    $r = Invoke-WebRequest -Uri "https://stagepu.rubikcare.com/manifest.webmanifest" -UseBasicParsing
    "✅ manifest: $($r.StatusCode) | $($r.Headers.'Content-Type')"
} catch {
    "❌ manifest: $($_.Exception.Message)"
}
```

**النتيجة المطلوبة:** `✅ manifest: 200 | application/manifest+json`

---

#### ❌ الخطأ 4: الموقع لا يفتح أبداً (خطأ 500)

**السبب:** وحدة **URL Rewrite** غير مثبتة. `web.config` يحتوي على قسم `<rewrite>` الذي لا يفهمه IIS بدون هذه الوحدة.

**الحل:**
```powershell
# تحقق من وجود الوحدة
Get-WebGlobalModule | Where-Object { $_.Name -like "*Rewrite*" }

# إذا لم تظهر، ثبّتها من:
# https://www.iis.net/downloads/microsoft/url-rewrite

# ثم أعد تدوير الـ App Pool
C:\Windows\System32\inetsrv\appcmd recycle apppool "PU_RubicCareStage"
```

---

#### ❌ الخطأ 5: التطبيق يعمل في المتصفح العادي لكن ليس في InPrivate (أو العكس)

**السبب:** **Service Worker** مخزّن في المتصفح العادي من نشر سابق، ويخدم ملفات قديمة.

**الحل:**
1. **اختبر دائماً من نافذة InPrivate:** `Ctrl + Shift + N`
2. أو امسح Service Worker يدوياً من المتصفح العادي:
   - افتح `F12` → **Application** → **Service Workers** → **Unregister**
   - ثم `F12` → **Application** → **Storage** → **Clear site data**
   - أعد تحميل الصفحة بـ `Ctrl + Shift + R`

---

#### ❌ الخطأ 6: صورة البروفايل لا تظهر + خطأ CORS

**العرض في Console:**
```
Access to fetch at 'https://uat.rubikcare.com/uploads/...' from origin 'https://stagepu.rubikcare.com'
has been blocked by CORS policy: No 'Access-Control-Allow-Origin' header
```

**السبب:** الصور مخزنة على نطاق الـ API (`uat.rubikcare.com`)، والـ PWA على نطاق مختلف (`stagepu.rubikcare.com`). بدون سياسة CORS، يمنع المتصفح التحميل.

**الحل:** أضف سياسة CORS في **الـ API** (`RubikCare.Api.Web/Program.cs`):

```csharp
builder.Services.AddCors(options =>
{
    options.AddPolicy("AllowPWA", policy =>
    {
        policy.WithOrigins("https://stagepu.rubikcare.com")
              .AllowAnyMethod()
              .AllowAnyHeader()
              .AllowCredentials();
    });
});

// بعد builder.Build() — مهم أن يكون قبل UseAuthorization
app.UseCors("AllowPWA");
```

> ⚠️ **تذكر:** هذا التعديل في الـ **API**، وليس في الـ PWA. ويجب إعادة نشر الـ API بعد التعديل.

---

### 🚫 5.10 المحاذير الحرجة (الممنوعات المطلقة)

| # | المحظور | السبب | البديل |
|---|---------|-------|--------|
| 1 | ❌ حذف `web.config` عند النشر | يحتوي إعدادات IIS (MIME + Rewrite) | احتفظ به دائماً |
| 2 | ❌ نسخ الملفات فوق القديمة بدون مسح | تضارب الأسماء المُجزّأة (hash) → `SRI failed` | امسح المجلد أولاً ثم انسخ |
| 3 | ❌ تعديل `web.config` يدوياً على السيرفر | ستفقده في النشر القادم | اجعله جزءاً من المشروع |
| 4 | ❌ تغيير المسار الفيزيائي إلى `wwwroot` | تضارب مع قاعدة `Serve subdir` | المسار الفيزيائي = الجذر دائماً |
| 5 | ❌ اختبار في المتصفح العادي بعد نشر جديد | Service Worker يخدم ملفات قديمة | اختبر في InPrivate دائماً |
| 6 | ❌ نشر بدون إيقاف الـ App Pool | قفل الملفات وفشل النسخ | أوقف الـ App Pool أولاً |
| 7 | ❌ نشر الـ API إلى مجلد الـ PWA | هما تطبيقان منفصلان تماماً | لكلٍ مجلده الخاص |

#### 🟡 إجراءات تتطلب حذراً

| الإجراء | الطريقة الصحيحة | الطريقة الخاطئة |
|---------|-----------------|-----------------|
| إعادة النشر | إيقاف → مسح → نشر → نسخ → تشغيل | نسخ فوق الملفات مباشرة |
| تعديل `web.config` | عدّل النسخة في المشروع ثم انشر | عدّل النسخة على السيرفر يدوياً |
| تحديث الـ API | انشر في مجلده الخاص (`RubikCareUat`) | لا تنسخ ملفات API إلى مجلد PWA |

#### 🟢 أفضل الممارسات

| الممارسة | السبب |
|----------|-------|
| احتفظ بنسخة احتياطية من `web.config` | للرجوع إليها عند الحاجة |
| وثّق أي تغيير في `Program.cs` أو `MainLayout.razor` | لتجنب فقدان التعديلات |
| اختبر في InPrivate بعد كل نشر | لتجنب كاش Service Worker |
| استخدم سكربت نشر موحد (`deploy-stage.ps1`) | لتجنب الأخطاء اليدوية |

---

### ✅ 5.11 قائمة المراجعة النهائية (CHECKLIST)

#### قبل النشر
- [ ] هل تم حفظ جميع التعديلات في الكود؟
- [ ] هل تم اختبار التطبيق محلياً؟
- [ ] هل `web.config` موجود في مجلد المشروع؟
- [ ] هل `CopyBlazorAssets` Target موجود في `.csproj`؟
- [ ] هل رابط الـ API في الكود يشير إلى البيئة الصحيحة (UAT)؟

#### أثناء النشر
- [ ] هل تم إيقاف الـ App Pool؟
- [ ] هل تم مسح المجلد القديم (باستثناء `web.config`)؟
- [ ] هل نجح `dotnet publish` بدون أخطاء؟
- [ ] هل تم نسخ الملفات إلى IIS؟

#### بعد النشر
- [ ] هل تم تشغيل الـ App Pool؟
- [ ] هل المسار الفيزيائي = الجذر (بدون `\wwwroot`)؟
- [ ] هل جميع فحوصات الروابط تعيد `200`؟
- [ ] هل اختبرت من نافذة InPrivate؟
- [ ] هل Console خالي من الأخطاء الحمراء؟
- [ ] هل تسجيل الدخول يعمل؟
- [ ] هل الوظائف الرئيسية تعمل؟

---

### 🎯 ملخص سريع للنشر

```
1. أوقف الـ App Pool
2. امسح المجلد (احتفظ بـ web.config)
3. dotnet publish
4. انسخ الملفات
5. شغّل الـ App Pool
6. اختبر في InPrivate
```
---

## 📲 القسم 6: نشر Mobile (MAUI)

### 📋 نظرة عامة

تطبيق `RubikCare.Mobile` مبني بـ **.NET MAUI** مع **BlazorWebView** لعرض مكونات `Shared.UI`. يُنشر كـ **APK** أو **AAB** لـ Android و **IPA** لـ iOS.

> ⚠️ **ملاحظة معمارية:** تطبيق الـ Mobile **لا يعتمد على أي مشروع آخر** مباشرة — يتواصل مع الـ API عبر HTTP فقط. لذلك، قبل نشر أي إصدار جديد من الموبايل، تأكد أن الـ API يدعم كل الـ Endpoints التي يستدعيها التطبيق.

---

### 🤖 6.1 نشر Android (APK / AAB)

#### متطلبات البناء

| المكوّن | الإصدار الأدنى | الغرض |
|---------|----------------|-------|
| Android SDK | API 34+ | البناء والتوقيع |
| Java JDK | 17+ | أدوات التوقيع |
| .NET MAUI Workload | 10.0+ | `dotnet workload install maui-android` |

#### الخطوة 1: إنشاء ملف التوقيع (Keystore) — مرة واحدة فقط

```bash
keytool -genkey -v -keystore rubikcare.keystore -alias rubikcare -keyalg RSA -keysize 2048 -validity 10000
```

> 🔴 **احفظ ملف `rubikcare.keystore` وكلمة المرور في مكان آمن.** لا يمكن تحديث التطبيق على Google Play بدونه.

#### الخطوة 2: إنشاء APK للتوزيع المباشر

```bash
cd C:\RC\Rubikcare.Full.Migration
dotnet publish RubikCare.Mobile -f net10.0-android -c Release /p:AndroidPackageFormat=apk
```

#### الخطوة 3: إنشاء AAB لـ Google Play

```bash
dotnet publish RubikCare.Mobile -f net10.0-android -c Release /p:AndroidPackageFormat=aab
```

#### الخطوة 4: التحقق من رابط الـ API

تأكد أن `AppConfig.cs` في بناء Release يشير إلى البيئة الصحيحة:

```csharp
// في RubikCare.Mobile/Infrastructure/AppConfig.cs
#if DEBUG
    public static string ApiBaseUrl = "http://localhost:5235";
#else
    public static string ApiBaseUrl = "https://api.rubikcare.com"; // للإنتاج
#endif
```

#### الخطوة 5: رفع إلى Google Play Console

1. أنشئ إصداراً جديداً في **Production Track**
2. ارفع ملف **AAB**
3. املأ بيانات الإصدار (ملاحظات الإصدار)
4. راجع بيانات المتجر (إن كانت أول مرة)
5. قدّم للمراجعة

---

### 🍎 6.2 نشر iOS (IPA)

#### المتطلبات

| المكوّن | التفاصيل |
|---------|----------|
| Mac | Xcode 16+ مثبت |
| Apple Developer Account | $99/سنة |
| .NET MAUI Workload | `dotnet workload install maui-ios` |

#### البناء من Windows (يتطلب Mac متصل)

```bash
dotnet publish RubikCare.Mobile -f net10.0-ios -c Release
```

#### البناء من Mac مباشرة

```bash
dotnet publish RubikCare.Mobile -f net10.0-ios -c Release /p:ArchiveOnBuild=true
```

#### الرفع إلى App Store

1. افتح **Xcode → Organizer**
2. اختر الأرشيف → **Distribute App**
3. اختر **App Store Connect**
4. اتبع خطوات الرفع
5. في **App Store Connect**: أنشئ إصداراً جديداً وقدّمه للمراجعة

---

### 🐛 6.3 الأخطاء الشائعة في نشر الموبايل

#### ❌ الخطأ 1: `Ambiguous routes matched`

**العرض:**
```
System.ArgumentException: Ambiguous routes matched for: //.../pageName
```

**السبب:** نفس الصفحة مسجلة مرتين — في `AppShell.xaml` و `AppShell.xaml.cs` معاً.

**الحل:**
- الصفحات الرئيسية (Root) → تسجل في `AppShell.xaml` فقط
- الصفحات الفرعية (Detail) → تسجل في `AppShell.xaml.cs` فقط عبر `Routing.RegisterRoute`

#### ❌ الخطأ 2: `JavaProxyThrowable` عند زر الرجوع

**السبب:** تعارض إصدارات حزم MAUI.

**الحل:** وحّد الإصدارات في `.csproj`:
```xml
<PackageReference Include="Microsoft.Maui.Controls" Version="10.0.20" />
<PackageReference Include="Microsoft.AspNetCore.Components.WebView.Maui" Version="10.0.20" />
```

> 🔴 **القاعدة الذهبية:** `Microsoft.AspNetCore.Components.WebView.Maui` يجب أن يكون دائماً على نفس إصدار `Microsoft.Maui.Controls`.

---

## 🔐 القسم 7: إعدادات الأمان

### 🔑 7.1 مفاتيح الإنتاج

#### للتطوير (User Secrets)

```bash
dotnet user-secrets set "Jwt:Key" "dev-key-12345678" --project RubikCare.Api.Web
```

#### للإنتاج (Environment Variables)

```powershell
# ⚠️ لا تضع مفاتيح الإنتاج في الكود أو في appsettings المنشور
setx Jwt__Key "production-key-32-chars-minimum" /M
```

### 🛡️ 7.2 حماية البيانات (Data Protection)

```csharp
// في Program.cs
builder.Services.AddDataProtection()
    .PersistKeysToFileSystem(new DirectoryInfo(@"C:\Keys\RubikCare"))
    .SetApplicationName("RubikCare");
```

> ⚠️ **مهم:** يجب أن يكون مجلد `C:\Keys\RubikCare` موجوداً على جميع الخوادم التي تشغل التطبيق، وإلا ستفشل المصادقة عند إعادة التشغيل.

### 🌐 7.3 سياسات CORS

#### للـ Web و PWA (على الـ API)

```csharp
builder.Services.AddCors(options =>
{
    options.AddPolicy("AllowWebApp", policy =>
    {
        policy.WithOrigins(
            "https://rubikcare.com",
            "https://stagepu.rubikcare.com",
            "https://test.rubikcare.com"
        )
        .AllowAnyMethod()
        .AllowAnyHeader()
        .AllowCredentials();
    });
});
```

> ⚠️ **لا تستخدم `AllowAnyOrigin`** مع `AllowCredentials` — هذا غير آمن وغير مدعوم.

---

## 💾 القسم 8: النسخ الاحتياطي

### 🗄️ 8.1 قاعدة البيانات

#### نسخ احتياطي كامل

```sql
BACKUP DATABASE RubikCare 
TO DISK = 'C:\Backups\RubikCare_Full.bak' 
WITH INIT, COMPRESSION;
```

#### نسخ احتياطي تفاضلي

```sql
BACKUP DATABASE RubikCare 
TO DISK = 'C:\Backups\RubikCare_Diff.bak' 
WITH DIFFERENTIAL, COMPRESSION;
```

### ⏰ 8.2 جدولة تلقائية (SQL Server Agent)

```sql
-- إنشاء Job للنسخ الاحتياطي اليومي الساعة 2 صباحاً
EXEC sp_add_job @job_name = 'RubikCare_Daily_Backup';
EXEC sp_add_jobstep @job_name = 'RubikCare_Daily_Backup',
    @step_name = 'Backup',
    @command = 'BACKUP DATABASE RubikCare TO DISK = ''C:\Backups\RubikCare.bak'' WITH INIT';
EXEC sp_add_schedule @job_name = 'RubikCare_Daily_Backup',
    @name = 'Daily',
    @freq_type = 4,
    @freq_interval = 1,
    @active_start_time = 020000;
EXEC sp_add_jobserver @job_name = 'RubikCare_Daily_Backup';
```

### 📂 8.3 نسخ ملفات الموقع

```powershell
# نسخة احتياطية من ملفات الـ PWA قبل النشر
$timestamp = Get-Date -Format "yyyyMMdd_HHmmss"
Compress-Archive -Path "C:\WebSite\PU_RubicCareStage\*" `
    -DestinationPath "C:\Backups\PWA_$timestamp.zip" -Force
```

> 🔴 **قاعدة ذهبية:** خذ نسخة احتياطية من قاعدة البيانات **قبل أي نشر** يتضمن Migrations.

---

## 🔍 القسم 9: فحص الصحة والمراقبة (Health Checks & Monitoring)

### 🏥 9.1 إضافة Health Check

```csharp
// في Program.cs
builder.Services.AddHealthChecks()
    .AddDbContextCheck<BusinessDbContext>()
    .AddCheck("api-connectivity", () => HealthCheckResult.Healthy());

app.MapHealthChecks("/health");
```

### 📊 9.2 المراقبة

```powershell
# فحص أن API يعمل
Invoke-WebRequest -Uri "https://api.rubikcare.com/health" -UseBasicParsing
# استجابة متوقعة: Healthy
```

### 📋 9.3 استكشاف الأخطاء في الإنتاج

#### التطبيق لا يستجيب

```powershell
# تحقق من حالة الـ App Pool
C:\Windows\System32\inetsrv\appcmd list apppool "RubikCareApi"

# تحقق من سجلات النظام
Get-EventLog -LogName Application -Source "*ASP.NET*" -Newest 20
```

#### أخطاء 500

```powershell
# فعّل السجلات في web.config للـ Blazor Server / API
# <aspNetCore stdoutLogEnabled="true" stdoutLogFile=".\logs\stdout" />

# ثم افحص السجلات
Get-ChildItem "C:\WebSite\RubikCareApi\logs" -Filter "stdout_*.log" |
    Sort-Object LastWriteTime -Descending |
    Select-Object -First 1 |
    Get-Content -Tail 50
```

#### خطأ في قاعدة البيانات

```sql
-- تحقق من اتصال قاعدة البيانات
SELECT @@SERVERNAME AS ServerName, DB_NAME() AS CurrentDB, GETDATE() AS ServerTime;
```

---

## 📚 الملحق: الدروس المستفادة (Lessons Learned)

### 🔴 الدرس 1: الفشل الصامت (Silent Failure)

**القصة:** كانت عمليات الحفظ في صفحة التعديل على سيرفر الـ Test تفشل بدون أي رسالة خطأ.

**السبب الجذري:** مزيج من 3 عوامل:
1. دوال `GetAsync` و `PostAsync` تعيد `null` بصمت عند الفشل
2. `appsettings.Test.json` يشير إلى الـ API الخطأ (Live بدلاً من UAT)
3. نشر كود الـ Web الجديد بينما الـ API المستهدف هو النسخة القديمة

**الدرس:**
> ⭐ **لا تبتلع الاستثناءات أبداً.** ارمِ `Exception` واضحاً عند فشل أي طلب.
> ⭐ **تحقق من ملف البيئة قبل كل نشر.**

---

### 🔴 الدرس 2: نشر PWA — الأسماء المُجزّأة (Hashed Filenames)

**القصة:** بعد نشر الـ PWA، ظهرت شاشة زرقاء مع خطأ `SRI integrity checks failed`.

**السبب الجذري:** ملفات الـ Blazor لها أسماء مُجزّأة تتغير مع كل نشر. نسخ الملفات الجديدة فوق القديمة دون مسح المجلد ترك ملفات بـ hash قديم.

**الدرس:**
> ⭐ **امسح مجلد النشر دائماً قبل نسخ الملفات الجديدة** (مع الاحتفاظ بـ `web.config`).

---

### 🔴 الدرس 3: المسار الفيزيائي للـ PWA

**القصة:** ملفات الـ framework ترجع 404 رغم وجودها على السيرفر.

**السبب الجذري:** المسار الفيزيائي كان يشير إلى `wwwroot`، مما جعل قاعدة `Serve subdir` في `web.config` تبحث في `wwwroot\wwwroot\...`.

**الدرس:**
> ⭐ **المسار الفيزيائي للـ PWA يجب أن يكون الجذر**، وليس `wwwroot`.

---

## 📋 قائمة المراجعة الشاملة للنشر (Master Checklist)

### قبل أي نشر

- [ ] هل تم حفظ جميع التعديلات في الكود؟
- [ ] هل تم اختبار التطبيق محلياً؟
- [ ] هل تم أخذ نسخة احتياطية من قاعدة البيانات؟
- [ ] هل ملفات البيئة (`appsettings.{Environment}.json`) تشير إلى الروابط الصحيحة؟

### أثناء النشر

- [ ] هل تم إيقاف الـ App Pool قبل نسخ الملفات؟
- [ ] هل تم حفظ نسخة احتياطية من `web.config` و `appsettings`؟
- [ ] هل تم نسخ الملفات بنجاح؟

### بعد النشر

- [ ] هل تم تشغيل الـ App Pool؟
- [ ] هل الـ Health Check يعيد `200`؟
- [ ] هل تم الاختبار من نافذة InPrivate (للـ PWA)؟
- [ ] هل السجلات خالية من الأخطاء؟
- [ ] هل العمليات الرئيسية تعمل (تسجيل الدخول، الحفظ، القراءة)؟

---

## 🔗 روابط ذات صلة

- [00 - الهيكل المعماري](00-architecture-overview.md)
- [01 - Program.cs والتسجيلات الأساسية](01-program-cs-foundation.md)
- [02 - نظام الهوية والمصادقة](02-identity-system.md)
- [03 - دليل الأنماط](03-style-guide.md)
- [14 - نظام الكاش الموحد](14-caching-system.md)
- [الملحق ب - فهرس الخدمات](appendix-b-service-index.md)

---

**آخر تحديث:** 29 أغسطس 2026 | **الملف:** `12-deployment-guide.md`

