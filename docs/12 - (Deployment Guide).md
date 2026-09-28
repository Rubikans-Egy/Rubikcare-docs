
---

# 📄 12-deployment-guide.md — النسخة الموحدة الكاملة

```markdown
# 📖 12 - دليل النشر والإنتاج الشامل (Deployment Guide & SOP)

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

### 🚀 3.1 إجراءات ما قبل النشر

```powershell
# 1. تأكد من عدم وجود أخطاء بناء
cd C:\RC\Rubikcare.Full.Migration
dotnet build RubikCare.Api.Web -c Release

# 2. تحقق من Migrations غير مطبقة
dotnet ef migrations list --project RubikCare.Infrastructure --startup-project RubikCare.Api.Web

# 3. اختبر محلياً
dotnet run --project RubikCare.Api.Web
```

### 📦 3.2 خطوات النشر

#### الخطوة 1: إيقاف الـ App Pool

```powershell
C:\Windows\System32\inetsrv\appcmd stop apppool "RubikCareUat"
```

#### الخطوة 2: نسخة احتياطية من appsettings

```powershell
Copy-Item "C:\WebSite\RubikCareUat\appsettings.Test.json" `
    "C:\WebSite\RubikCareUat\appsettings.Test.json.backup" -Force
```

#### الخطوة 3: النشر

```powershell
cd C:\RC\Rubikcare.Full.Migration
dotnet publish RubikCare.Api.Web -c Release -o E:\rubikans\Publish\ApiUat -r win-x64 --self-contained false
```

#### الخطوة 4: نسخ الملفات (باستثناء appsettings)

```powershell
Get-ChildItem "E:\rubikans\Publish\ApiUat" -Exclude "appsettings.*.json" | 
    Copy-Item -Destination "C:\WebSite\RubikCareUat" -Recurse -Force
```

#### الخطوة 5: تشغيل الـ App Pool

```powershell
C:\Windows\System32\inetsrv\appcmd start apppool "RubikCareUat"
```

### 🧪 3.3 التحقق بعد النشر

```powershell
try {
    $r = Invoke-WebRequest -Uri "https://uat.rubikcare.com/health" -UseBasicParsing -TimeoutSec 10
    "✅ UAT API: $($r.StatusCode) - $($r.Content)"
} catch {
    "❌ UAT API: $($_.Exception.Message)"
}
```

### 🐛 3.4 الأخطاء الشائعة

#### ❌ الخطأ 1: `502.5 - Process Failure`

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

#### ❌ الخطأ 2: `401 Unauthorized`

**الحل:** تحقق من JWT Key و CORS (راجع القسم 7)

#### ❌ الخطأ 3: `404 Not Found`

**الحل:** تحقق من نشر الـ Controllers الجديدة

### 🚀 3.5 سكربت النشر الأوتوماتيكي

```powershell
# deploy-api-uat.ps1
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

---

## 🌐 القسم 4: نشر Web (Blazor Server)

### 📋 نظرة عامة

تطبيق `RubikCare.Web` هو تطبيق **Blazor Server** يعمل عبر SignalR.

> ⚠️ **ملاحظة معمارية:** يعتمد `RubikCare.Web` على `appsettings.{Environment}.json` لتحديد رابط الـ API المستهدف.

### 🚀 4.1 إجراءات ما قبل النشر

```powershell
# تحقق من رابط الـ API
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

### 📦 4.2 خطوات النشر

#### الخطوة 1: إيقاف الـ App Pool

```powershell
C:\Windows\System32\inetsrv\appcmd stop apppool "rubikcarenew"
```

#### الخطوة 2: نسخة احتياطية

```powershell
Copy-Item "C:\WebSite\RubikCareNew\web.config" "C:\WebSite\RubikCareNew\web.config.backup" -Force
Copy-Item "C:\WebSite\RubikCareNew\appsettings.Test.json" "C:\WebSite\RubikCareNew\appsettings.Test.json.backup" -Force
```

#### الخطوة 3: النشر والنسخ

```powershell
cd C:\RC\Rubikcare.Full.Migration
dotnet publish RubikCare.Web -c Release -o E:\rubikans\Publish\WebTest

Get-ChildItem "E:\rubikans\Publish\WebTest" -Exclude "web.config","appsettings.*.json" | 
    Copy-Item -Destination "C:\WebSite\RubikCareNew" -Recurse -Force
```

#### الخطوة 4: تشغيل الـ App Pool

```powershell
C:\Windows\System32\inetsrv\appcmd start apppool "rubikcarenew"
```

### ⚙️ 4.3 ملف `web.config` للـ Blazor Server

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
          <environmentVariable name="ASPNETCORE_ENVIRONMENT" value="Test" />
        </environmentVariables>
      </aspNetCore>
    </system.webServer>
  </location>
</configuration>
```

### 🧪 4.4 التحقق بعد النشر

```powershell
try {
    $r = Invoke-WebRequest -Uri "https://test.rubikcare.com" -UseBasicParsing -TimeoutSec 15
    "✅ Web: $($r.StatusCode)"
} catch {
    "❌ Web: $($_.Exception.Message)"
}

Get-Content "C:\WebSite\RubikCareNew\appsettings.Test.json" | Select-String "BaseUrl"
Get-Content "C:\WebSite\RubikCareNew\web.config" | Select-String "ASPNETCORE_ENVIRONMENT"
```

---
**ممتاز — سأحدّث وثيقة النشر بإضافة قسم كامل عن هذه المشكلة + السكربت الصحيح.**

---

## 📄 التحديثات المطلوبة في `12-deployment-guide.md`

### 1. إضافة قسم جديد بعد "القسم 4: نشر Web (Blazor Server)"

**أضف هذا القسم:**

```markdown
---

## 🔴 القسم 4.5: نشر Web — تحديث حرج (28 سبتمبر 2026)

### ⚠️ مشكلة "الصفحة فارغة + تحميل بطيء" — Root Cause

**التاريخ:** 28 سبتمبر 2026  
**البيئة:** `test.rubikcare.com` (RubikCare.Web)  
**الأعراض:** 
- صفحة Dashboard والتقارير تفتح ببطء شديد (> دقيقتين)
- ثم تفتح فارغة (بدون KPIs، Charts، Quick Access Data)
- الأزرار تعمل، لكن لا تُحمَّل أي بيانات من API

### 🎯 الأسباب الجذرية (3 مشاكل متتالية)

| # | السبب | التفصيل |
|---|-------|---------|
| 1 | **`ASPNETCORE_ENVIRONMENT` غير محدد** | `web.config` لا يحتوي على `<environmentVariables>` → التطبيق يعمل بـ `Production` افتراضياً → يقرأ `appsettings.Production.json` (BaseUrl خاطئ) بدلاً من `appsettings.Test.json` |
| 2 | **`appcmd recycle` لا يُعيد تحميل `web.config`** | `recycle` يعيد تشغيل عملية `w3wp` **بدون** قطع الاتصالات النشطة → الإعدادات القديمة تبقى |
| 3 | **`IUserSessionService` يُخزّن الجلسة في `IMemoryCache`** | `IMemoryCache` يبدأ فارغاً عند restart، لكن **الجلسة القديمة** تبقى في الكاش حتى Logout → `CurrentOrganizationId = null` |

### 🔧 الحل الكامل

#### الخطوة 1: تعديل `web.config` (إضافة `ASPNETCORE_ENVIRONMENT`)

**القالب الصحيح لـ `web.config`:**

```xml
<?xml version="1.0" encoding="utf-8"?>
<configuration>
  <location path="." inheritInChildApplications="false">
    <system.webServer>
      <handlers>
        <add name="aspNetCore" path="*" verb="*" modules="AspNetCoreModuleV2" resourceType="Unspecified" />
      </handlers>
      <aspNetCore processPath="dotnet" 
                  arguments=".\Rubikcare.Web.dll" 
                  stdoutLogEnabled="true" 
                  stdoutLogFile=".\logs\stdout" 
                  hostingModel="inprocess">
        <environmentVariables>
          <environmentVariable name="ASPNETCORE_ENVIRONMENT" value="Test" />
        </environmentVariables>
      </aspNetCore>
    </system.webServer>
  </location>
</configuration>
```

**⚠️ القيم المسموحة لـ `value`:**
- `Test` → لبيئة `test.rubikcare.com`
- `Production` → لبيئة `rubikcare.com`
- `Development` → للتطوير المحلي فقط

#### الخطوة 2: إعادة تشغيل نظيفة (Stop + Start)

**❌ خطأ:**
```powershell
C:\Windows\System32\inetsrv\appcmd recycle apppool "rubikcarenew"
```

**✅ صحيح:**
```powershell
C:\Windows\System32\inetsrv\appcmd stop apppool "rubikcarenew"
Start-Sleep -Seconds 5
C:\Windows\System32\inetsrv\appcmd start apppool "rubikcarenew"
```

**السبب:** `recycle` لا يُطبّق `web.config` الجديد بشكل موثوق.

#### الخطوة 3: مسح الجلسة القديمة

**بعد إعادة التشغيل:**
1. افتح المتصفح على `https://test.rubikcare.com`
2. **سجّل خروج** (Logout) — لمسح `IMemoryCache` للجلسة القديمة
3. **سجّل دخول** من جديد — لبناء جلسة جديدة بالـ `BaseUrl` الصحيح
4. اختبر صفحة التقارير

### 📋 Checklist النشر الآمن للـ Web

```
قبل النشر:
  [ ] appsettings.Test.json يحتوي BaseUrl = "https://uat.rubikcare.com"
  [ ] web.config يحتوي ASPNETCORE_ENVIRONMENT = "Test"

أثناء النشر:
  [ ] appcmd stop apppool "rubikcarenew"
  [ ] نسخ الملفات (باستثناء web.config و appsettings.*.json)
  [ ] appcmd start apppool "rubikcarenew"

بعد النشر:
  [ ] افتح المتصفح وتأكد أن التطبيق يقرأ appsettings.Test.json
  [ ] سجّل خروج ودخول
  [ ] اختبر صفحة Dashboard (يجب أن تُحمَّل KPIs)
  [ ] اختبر صفحة Reports (يجب أن تُحمَّل البيانات)
```

### 🚀 سكربت النشر الصحيح (`deploy-web-test.ps1`)

```powershell
# deploy-web-test.ps1 — النشر الآمن لـ RubikCare.Web إلى Test
param(
    [string]$ProjectPath = "C:\RC\Rubikcare.Full.Migration",
    [string]$PublishPath = "E:\rubikans\Publish\WebTest",
    [string]$IISPath = "C:\WebSite\RubikCareNew",
    [string]$AppPoolName = "rubikcarenew",
    [string]$SiteUrl = "https://test.rubikcare.com"
)

Write-Host "================================================" -ForegroundColor Cyan
Write-Host "  RubikCare Web - نشر آمن إلى Test" -ForegroundColor Cyan
Write-Host "================================================`n" -ForegroundColor Cyan

# ─── 1. إيقاف كامل (وليس recycle) ───
Write-Host "⏸️  الخطوة 1: إيقاف App Pool (Stop كامل)..." -ForegroundColor Yellow
C:\Windows\System32\inetsrv\appcmd stop apppool $AppPoolName
Start-Sleep -Seconds 5
Write-Host "✅ تم الإيقاف" -ForegroundColor Green

# ─── 2. التحقق من الإعدادات ───
Write-Host "`n🔍 الخطوة 2: التحقق من appsettings.Test.json..." -ForegroundColor Yellow
$testSettings = Get-Content "$IISPath\appsettings.Test.json" -Raw | ConvertFrom-Json
$baseUrl = $testSettings.ApiSettings.BaseUrl
if ($baseUrl -ne "https://uat.rubikcare.com") {
    Write-Host "❌ BaseUrl خاطئ: $baseUrl" -ForegroundColor Red
    Write-Host "⚠️  يجب أن يكون: https://uat.rubikcare.com" -ForegroundColor Yellow
    C:\Windows\System32\inetsrv\appcmd start apppool $AppPoolName
    exit 1
}
Write-Host "✅ BaseUrl = $baseUrl" -ForegroundColor Green

# ─── 3. التحقق من ASPNETCORE_ENVIRONMENT ───
Write-Host "`n🔍 الخطوة 3: التحقق من ASPNETCORE_ENVIRONMENT..." -ForegroundColor Yellow
$webConfig = Get-Content "$IISPath\web.config" -Raw
if ($webConfig -notmatch 'value="Test"') {
    Write-Host "❌ ASPNETCORE_ENVIRONMENT غير محدد أو ليس Test" -ForegroundColor Red
    C:\Windows\System32\inetsrv\appcmd start apppool $AppPoolName
    exit 1
}
Write-Host "✅ ASPNETCORE_ENVIRONMENT = Test" -ForegroundColor Green

# ─── 4. النشر ───
Write-Host "`n📦 الخطوة 4: نشر المشروع..." -ForegroundColor Yellow
Set-Location $ProjectPath
dotnet publish RubikCare.Web -c Release -o $PublishPath
if ($LASTEXITCODE -ne 0) {
    Write-Host "❌ فشل النشر!" -ForegroundColor Red
    C:\Windows\System32\inetsrv\appcmd start apppool $AppPoolName
    exit 1
}
Write-Host "✅ النشر نجح" -ForegroundColor Green

# ─── 5. نسخ الملفات (باستثناء web.config و appsettings) ───
Write-Host "`n📋 الخطوة 5: نسخ الملفات إلى IIS..." -ForegroundColor Yellow
Get-ChildItem $PublishPath -Exclude "web.config","appsettings.*.json" | 
    Copy-Item -Destination $IISPath -Recurse -Force
Write-Host "✅ تم النسخ (بدون المساس بـ web.config و appsettings)" -ForegroundColor Green

# ─── 6. تشغيل App Pool ───
Write-Host "`n▶️  الخطوة 6: تشغيل App Pool (Start نظيف)..." -ForegroundColor Yellow
C:\Windows\System32\inetsrv\appcmd start apppool $AppPoolName
Write-Host "✅ تم التشغيل" -ForegroundColor Green

# ─── 7. اختبار Health Check ───
Write-Host "`n🧪 الخطوة 7: اختبار الموقع..." -ForegroundColor Yellow
Start-Sleep -Seconds 8
try {
    $r = Invoke-WebRequest -Uri "$SiteUrl" -UseBasicParsing -TimeoutSec 20
    Write-Host "✅ الموقع يعمل: $($r.StatusCode)" -ForegroundColor Green
} catch {
    Write-Host "❌ الموقع لا يعمل: $($_.Exception.Message)" -ForegroundColor Red
}

# ─── 8. اختبار API ───
Write-Host "`n🧪 الخطوة 8: اختبار الاتصال بـ API..." -ForegroundColor Yellow
try {
    $r = Invoke-WebRequest -Uri "https://uat.rubikcare.com/api/ping" -UseBasicParsing -TimeoutSec 15
    Write-Host "✅ API يعمل: $($r.StatusCode)" -ForegroundColor Green
} catch {
    Write-Host "❌ API لا يعمل: $($_.Exception.Message)" -ForegroundColor Red
}

# ─── النتيجة النهائية ───
Write-Host "`n================================================" -ForegroundColor Cyan
Write-Host "  🎉 تم النشر بنجاح!" -ForegroundColor Green
Write-Host "  🔗 الرابط: $SiteUrl" -ForegroundColor Green
Write-Host "" -ForegroundColor White
Write-Host "  ⚠️  تذكير:" -ForegroundColor Yellow
Write-Host "  1. سجّل خروج ثم دخول في المتصفح (لمسح IMemoryCache)" -ForegroundColor Yellow
Write-Host "  2. اختبر صفحة Dashboard والتقارير" -ForegroundColor Yellow
Write-Host "================================================`n" -ForegroundColor Cyan
```

### 🎯 القاعدة الذهبية الجديدة

> **بعد أي تعديل في `web.config` أو `appsettings.*.json`:**
> 1. **`appcmd stop`** (وليس `recycle`)
> 2. **`Start-Sleep -Seconds 5`**
> 3. **`appcmd start`**
> 4. **تسجيل خروج ودخول** في المتصفح
>
> **بدون هذه الخطوات:** التعديلات **لن تُطبَّق**، وستستمر الصفحات فارغة.

---
```

---

### 2. إضافة تحذير في قسم "الأخطاء الشائعة" (4.4 أو 5.7)

**أضف تحت `❌ الخطأ 4`:**

```markdown
#### ❌ الخطأ 5: الصفحة تفتح فارغة + تحميل بطيء (Test/UAT)

**الأعراض:**
- الصفحة تستغرق > دقيقتين للفتح
- تفتح بدون بيانات (KPIs، Charts، Tables فارغة)
- الأزرار تعمل (Quick Access) لكن لا تُحمَّل البيانات

**السبب:**
1. `ASPNETCORE_ENVIRONMENT` غير محدد في `web.config`
2. `appcmd recycle` بدل `stop` + `start`
3. الجلسة القديمة في `IMemoryCache` لم تُمسح

**الحل السريع:**
```powershell
# 1. تحقق من البيئة
Get-Content "C:\WebSite\RubikCareNew\web.config" | Select-String "ASPNETCORE_ENVIRONMENT"

# 2. تحقق من BaseUrl
Get-Content "C:\WebSite\RubikCareNew\appsettings.Test.json" | Select-String "BaseUrl"
# يجب أن يكون: https://uat.rubikcare.com

# 3. إعادة تشغيل نظيفة
C:\Windows\System32\inetsrv\appcmd stop apppool "rubikcarenew"
Start-Sleep -Seconds 5
C:\Windows\System32\inetsrv\appcmd start apppool "rubikcarenew"

# 4. في المتصفح: سجّل خروج ثم دخول
```
```

---

### 3. إضافة قسم كامل في نهاية الوثيقة

**أضف قبل "Lessons Learned" (القسم 11):**

```markdown
---

## 🔬 القسم 10.5: تشخيص متقدم — صفحة فارغة في Test

### 🔍 خطوات التشخيص السريع (5 دقائق)

```powershell
# ═══════════════════════════════════════════════════════
# 1. هل البيئة محددة؟
# ═══════════════════════════════════════════════════════
Get-Content "C:\WebSite\RubikCareNew\web.config" | Select-String "ASPNETCORE_ENVIRONMENT"
# النتيجة المتوقعة: value="Test"

# ═══════════════════════════════════════════════════════
# 2. ما هو BaseUrl الفعلي المستخدم؟
# ═══════════════════════════════════════════════════════
Get-Content "C:\WebSite\RubikCareNew\appsettings.Test.json" | Select-String "BaseUrl"
# النتيجة المتوقعة: "BaseUrl": "https://uat.rubikcare.com"

# ═══════════════════════════════════════════════════════
# 3. هل كل ملفات appsettings لها نفس BaseUrl؟
# ═══════════════════════════════════════════════════════
Get-ChildItem "C:\WebSite\RubikCareNew\appsettings*.json" | ForEach-Object { 
    "--- $($_.Name) ---"
    (Get-Content $_.FullName -Raw) -split "`n" | Select-String "BaseUrl"
}

# ═══════════════════════════════════════════════════════
# 4. هل API يعمل؟
# ═══════════════════════════════════════════════════════
Invoke-WebRequest -Uri "https://uat.rubikcare.com/api/ping" -UseBasicParsing
# النتيجة المتوقعة: {"success":true,...}

# ═══════════════════════════════════════════════════════
# 5. هل App Pool يعمل؟
# ═══════════════════════════════════════════════════════
Get-IISAppPool | Where-Object { $_.Name -eq "rubikcarenew" } | Select-Object Name, State, ProcessId
# State يجب أن تكون Started
```

### 🎯 شجرة القرار

```
الصفحة فارغة؟
│
├── هل ASPNETCORE_ENVIRONMENT = Test?
│   ├── لا → عدّل web.config + stop/start
│   └── نعم → 
│       │
│       ├── هل BaseUrl في appsettings.Test.json = https://uat.rubikcare.com?
│       │   ├── لا → عدّل الملف + stop/start
│       │   └── نعم → 
│       │       │
│       │       ├── هل API يعمل؟ (api/ping)
│       │       │   ├── لا → افحص App Pool + Event Log
│       │       │   └── نعم → 
│       │       │       │
│       │       │       └── ✅ المشكلة في IMemoryCache
│       │       │           → سجّل خروج ودخول
```

---
```


## 📱 القسم 5: نشر PWA (Blazor WebAssembly) ⭐

### 📋 نظرة عامة

تطبيق `RubikCare.PWA` هو **Blazor WebAssembly Standalone**. يختلف نشره كلياً عن الـ Web والـ API:

| الجانب | Web / API | PWA (Blazor WASM) |
|--------|-----------|-------------------|
| **طبيعة التشغيل** | خادم .NET يعمل فعلياً | ملفات ثابتة تُخدم من IIS |
| **الـ App Pool** | يحتاج .NET Runtime | **No Managed Code** |
| **`web.config`** | ASP.NET Core Module | **URL Rewrite + MIME types** |
| **منطق التطبيق** | يعمل على الخادم | يعمل في المتصفح |

> 🔴 **القاعدة المحورية:** تطبيق الـ PWA لا يحتاج خادم .NET، لكنه يحتاج **إعداد IIS دقيقاً** ليخدم ملفات `.wasm` و `.dat` و `.js` بشكل صحيح.

### 🏗️ 5.1 المتطلبات الأساسية

```powershell
# التحقق من URL Rewrite Module
Get-WebGlobalModule | Where-Object { $_.Name -like "*Rewrite*" }
Test-Path "C:\Windows\System32\inetsrv\rewrite.dll"

# إنشاء App Pool بوضع No Managed Code
C:\Windows\System32\inetsrv\appcmd add apppool `
    /name:"PU_RubicCareStage" `
    /managedRuntimeVersion:"" `
    /managedPipelineMode:"Integrated"
```

### 🎯 5.2 القاعدة الذهبية: المسار الفيزيائي

> 🔴 **أكبر خطأ:** المسار الفيزيائي يشير إلى `wwwroot` بدلاً من الجذر.

#### ❌ الخطأ (يسبب 404 لملفات الـ framework)
```
C:\WebSite\PU_RubicCareStage\wwwroot
```

#### ✅ الصحيح (المسار الفيزيائي هو الجذر)
```
C:\WebSite\PU_RubicCareStage
```

```powershell
# للتحقق
C:\Windows\System32\inetsrv\appcmd list vdir "PU_RubicCareStage/" /text:physicalPath

# للتصحيح
C:\Windows\System32\inetsrv\appcmd set vdir "PU_RubicCareStage/" /physicalPath:"C:\WebSite\PU_RubicCareStage"
```

### 📄 5.3 ملف `web.config` الكامل للـ PWA

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

### 🔢 5.4 مشكلة الأسماء المُجزّأة (Hashed Filenames)

أضف هذا الـ Target في `RubikCare.PWA.csproj`:

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

### 📦 5.5 خطوات النشر

```powershell
# 1. إيقاف الـ App Pool
C:\Windows\System32\inetsrv\appcmd stop apppool "PU_RubicCareStage"

# 2. مسح المجلد (باستثناء web.config)
Get-ChildItem "C:\WebSite\PU_RubicCareStage" -Exclude "web.config" | 
    Remove-Item -Recurse -Force

# 3. نشر المشروع
cd C:\RC\Rubikcare.Full.Migration
dotnet publish RubikCare.PWA -c Release -o E:\rubikans\Publish\PWA

# 4. نسخ الملفات
Copy-Item -Path "E:\rubikans\Publish\PWA\*" -Destination "C:\WebSite\PU_RubicCareStage\" -Recurse -Force

# 5. تشغيل الـ App Pool
C:\Windows\System32\inetsrv\appcmd start apppool "PU_RubicCareStage"
```

### 🤖 5.6 سكربت النشر الأوتوماتيكي (`deploy-stage.ps1`)

```powershell
# deploy-stage.ps1
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

# الخطوة 5: تشغيل الـ App Pool
Write-Host "`n▶️  الخطوة 5: تشغيل الـ App Pool..." -ForegroundColor Yellow
C:\Windows\System32\inetsrv\appcmd start apppool $AppPoolName

# الخطوة 6: اختبار الروابط
Write-Host "`n🧪 الخطوة 6: اختبار الروابط..." -ForegroundColor Yellow
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

if ($allPassed) {
    Write-Host "`n  🎉 تم النشر بنجاح!" -ForegroundColor Green
    Write-Host "  📱 الرابط: $SiteUrl" -ForegroundColor Green
    Write-Host "  ⚠️  تذكر: اختبر من نافذة InPrivate" -ForegroundColor Yellow
} else {
    Write-Host "`n  ⚠️  بعض الفحوصات فشلت" -ForegroundColor Yellow
}
```

### 🐛 5.7 الأخطاء الشائعة في PWA

#### ❌ الخطأ 1: شاشة زرقاء + `Unexpected token '<'`

**السبب:** المتصفح يتلقى `index.html` بدلاً من ملف JavaScript.

**الحل:**
```powershell
# تحقق من وجود الملف
Test-Path "C:\WebSite\PU_RubicCareStage\wwwroot\_framework\blazor.webassembly.js"

# تحقق من المسار الفيزيائي
C:\Windows\System32\inetsrv\appcmd list vdir "PU_RubicCareStage/" /text:physicalPath
```

#### ❌ الخطأ 2: `SRI integrity checks failed`

**السبب:** ملفات قديمة بـ hash مختلف، أو المسار الفيزيائي خاطئ.

**الحل:**
```powershell
# امسح المجلد وأعد النشر
C:\Windows\System32\inetsrv\appcmd stop apppool "PU_RubicCareStage"
Get-ChildItem "C:\WebSite\PU_RubicCareStage" -Exclude "web.config" | Remove-Item -Recurse -Force
Copy-Item -Path "E:\rubikans\Publish\PWA\*" -Destination "C:\WebSite\PU_RubicCareStage\" -Recurse -Force
C:\Windows\System32\inetsrv\appcmd start apppool "PU_RubicCareStage"
```

#### ❌ الخطأ 3: `manifest.webmanifest` يرجع 404

**الحل:** تأكد من MIME type في `web.config`:
```xml
<mimeMap fileExtension=".webmanifest" mimeType="application/manifest+json" />
```

#### ❌ الخطأ 4: الموقع لا يفتح (خطأ 500)

**الحل:** تحقق من URL Rewrite Module:
```powershell
Get-WebGlobalModule | Where-Object { $_.Name -like "*Rewrite*" }
```

#### ❌ الخطأ 5: Service Worker يخدم ملفات قديمة

**الحل:** اختبر دائماً في **InPrivate** (`Ctrl + Shift + N`)

#### ❌ الخطأ 6: صورة البروفايل لا تظهر + CORS

**الحل:** أضف CORS في الـ API (راجع القسم 7)

---

## 📲 القسم 6: نشر Mobile (MAUI)

### 🤖 6.1 نشر Android (APK / AAB)

```bash
# إنشاء Keystore (مرة واحدة)
keytool -genkey -v -keystore rubikcare.keystore -alias rubikcare -keyalg RSA -keysize 2048 -validity 10000

# APK للتوزيع المباشر
cd C:\RC\Rubikcare.Full.Migration
dotnet publish RubikCare.Mobile -f net10.0-android -c Release /p:AndroidPackageFormat=apk

# AAB لـ Google Play
dotnet publish RubikCare.Mobile -f net10.0-android -c Release /p:AndroidPackageFormat=aab
```

### 🍎 6.2 نشر iOS (IPA)

```bash
# من Mac
dotnet publish RubikCare.Mobile -f net10.0-ios -c Release /p:ArchiveOnBuild=true
```

### 🐛 6.3 أخطاء شائعة في Mobile

#### ❌ `Ambiguous routes matched`

**الحل:** الصفحات الرئيسية في `AppShell.xaml` فقط، الفرعية في `AppShell.xaml.cs`

#### ❌ `JavaProxyThrowable`

**الحل:** وحّد إصدارات MAUI:
```xml
<PackageReference Include="Microsoft.Maui.Controls" Version="10.0.20" />
<PackageReference Include="Microsoft.AspNetCore.Components.WebView.Maui" Version="10.0.20" />
```

---

## 🔐 القسم 7: إعدادات الأمان

### 🔑 7.1 مفاتيح الإنتاج

```bash
# للتطوير
dotnet user-secrets set "Jwt:Key" "dev-key-12345678" --project RubikCare.Api.Web

# للإنتاج
setx Jwt__Key "production-key-32-chars-minimum" /M
```

### 🛡️ 7.2 حماية البيانات (Data Protection)

```csharp
builder.Services.AddDataProtection()
    .PersistKeysToFileSystem(new DirectoryInfo(@"C:\Keys\RubikCare"))
    .SetApplicationName("RubikCare");
```

### 🌐 7.3 سياسات CORS

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

app.UseCors("AllowWebApp");
```

> ⚠️ **لا تستخدم `AllowAnyOrigin`** مع `AllowCredentials` — غير آمن.

---

## 💾 القسم 8: النسخ الاحتياطي

### 🗄️ 8.1 قاعدة البيانات

```sql
BACKUP DATABASE RubikCare 
TO DISK = 'C:\Backups\RubikCare_Full.bak' 
WITH INIT, COMPRESSION;
```

### 📂 8.2 نسخ ملفات الموقع

```powershell
$timestamp = Get-Date -Format "yyyyMMdd_HHmmss"
Compress-Archive -Path "C:\WebSite\PU_RubicCareStage\*" `
    -DestinationPath "C:\Backups\PWA_$timestamp.zip" -Force
```

---

## 🔍 القسم 9: Health Checks

```csharp
builder.Services.AddHealthChecks()
    .AddDbContextCheck<BusinessDbContext>();

app.MapHealthChecks("/health");
```

```powershell
Invoke-WebRequest -Uri "https://api.rubikcare.com/health" -UseBasicParsing
```

---

## 🔴 القسم 10: Root Cause Analysis (الفشل الصامت)

**Date:** August 20, 2026  
**Developer:** Youssef Shady  
**Status:** Resolved

### وصف المشكلة

"Silent Failure" — عمليات الحفظ تفشل بدون أي رسالة خطأ في UI أو Console.

### السبب الجذري (3 عوامل)

| # | العامل | التفاصيل |
|---|--------|----------|
| 1 | **Swallowed Exceptions** | `GetAsync` و `PostAsync` تعيد `null` بصمت عند الفشل بدلاً من رمي Exception |
| 2 | **Misconfigured BaseUrl** | `appsettings.Test.json` يشير إلى Live API بدلاً من UAT |
| 3 | **API Environment Conflict** | كود Web جديد منشور بينما API المستهدف قديم |

### الحل

```csharp
// ❌ قبل: فشل صامت
public async Task<T> GetAsync<T>(string endpoint)
{
    var response = await _httpClient.GetAsync(endpoint);
    if (!response.IsSuccessStatusCode)
        return default;
    return await response.Content.ReadFromJsonAsync<T>();
}

// ✅ بعد: Exception واضح
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

---

## 📚 القسم 11: Lessons Learned (الدروس المستفادة)

### 🔴 الدرس 1: الفشل الصامت

**الدرس:** لا تبتلع الاستثناءات أبداً. ارمِ Exception واضحاً عند فشل أي طلب. تحقق من ملف البيئة قبل كل نشر.

### 🔴 الدرس 2: نشر PWA — الأسماء المُجزّأة

**الدرس:** امسح مجلد النشر دائماً قبل نسخ الملفات الجديدة (مع الاحتفاظ بـ `web.config`).

### 🔴 الدرس 3: المسار الفيزيائي للـ PWA

**الدرس:** المسار الفيزيائي يجب أن يكون الجذر، وليس `wwwroot`.

### 🔴 الدرس 4: URL Rewrite Module

**الدرس:** بدون هذه الوحدة، `web.config` الخاص بالـ PWA لن يعمل.

### 🔴 الدرس 5: Service Worker

**الدرس:** اختبر دائماً في InPrivate بعد كل نشر.

---

## 🚫 القسم 12: Critical Prohibitions (الممنوعات المطلقة)

| # | المحظور | السبب | البديل |
|---|---------|-------|--------|
| 1 | ❌ حذف `web.config` عند النشر | يحتوي إعدادات IIS | احتفظ به دائماً |
| 2 | ❌ نسخ الملفات فوق القديمة بدون مسح | تضارب hash → `SRI failed` | امسح أولاً ثم انسخ |
| 3 | ❌ تعديل `web.config` يدوياً على السيرفر | ستفقده في النشر القادم | اجعله جزءاً من المشروع |
| 4 | ❌ تغيير المسار الفيزيائي إلى `wwwroot` | تضارب مع `Serve subdir` | المسار = الجذر دائماً |
| 5 | ❌ اختبار في المتصفح العادي بعد نشر | Service Worker قديم | InPrivate دائماً |
| 6 | ❌ نشر بدون إيقاف App Pool | قفل الملفات | أوقف أولاً |
| 7 | ❌ نشر API إلى مجلد PWA | تطبيقان منفصلان | لكلٍ مجلده |

---

## 📋 القسم 13: Master Deployment Checklist

### قبل أي نشر

- [ ] هل تم حفظ جميع التعديلات؟
- [ ] هل تم اختبار التطبيق محلياً؟
- [ ] هل تم أخذ نسخة احتياطية من قاعدة البيانات؟
- [ ] هل ملفات البيئة تشير إلى الروابط الصحيحة؟

### أثناء النشر

- [ ] هل تم إيقاف الـ App Pool؟
- [ ] هل تم حفظ نسخة احتياطية من `web.config` و `appsettings`؟
- [ ] هل تم نسخ الملفات بنجاح؟

### بعد النشر

- [ ] هل تم تشغيل الـ App Pool؟
- [ ] هل Health Check يعيد `200`؟
- [ ] هل تم الاختبار من InPrivate (للـ PWA)؟
- [ ] هل السجلات خالية من الأخطاء؟
- [ ] هل العمليات الرئيسية تعمل؟

---

## 🔍 القسم 14: Troubleshooting Cheatsheet

```powershell
# 1. Verify Web app is reading correct API URL
Get-Content "C:\WebSite\RubikCareNew\appsettings.Test.json" | Select-String "BaseUrl"

# 2. Verify Test environment is active in web.config
Get-Content "C:\WebSite\RubikCareNew\web.config" | Select-String "ASPNETCORE_ENVIRONMENT"

# 3. Check latest Blazor/Server errors in logs
Get-ChildItem -Path "C:\WebSite\RubikCareNew\logs\stdout" -Filter "*.log" | 
    Sort-Object LastWriteTime -Descending | 
    Select-Object -First 1 | 
    Get-Content -Tail 30

# 4. Check Application Event Log
Get-EventLog -LogName Application -Source "*ASP.NET*" -Newest 10 | Format-List

# 5. Verify PWA physical path
C:\Windows\System32\inetsrv\appcmd list vdir "PU_RubicCareStage/" /text:physicalPath

# 6. Check URL Rewrite Module
Get-WebGlobalModule | Where-Object { $_.Name -like "*Rewrite*" }
```

---

## 🔗 روابط ذات صلة

- [00 - الهيكل المعماري](00-architecture-overview.md)
- [01 - Program.cs والتسجيلات الأساسية](01-program-cs-foundation.md)
- [02 - نظام الهوية والمصادقة](02-identity-system.md)
- [14 - نظام الكاش الموحد](14-caching-system.md)

---

**آخر تحديث:** 29 أغسطس 2026 | **الملف:** `12-deployment-guide.md`
```

---

## ✅ تم!

ده المحتوى الكامل للوثيقة الموحدة. **انسخه كله** وحطه في `12-deployment-guide.md` في GitHub.


**قولي لما تخلص عشان نتأكد إن كل حاجة تمام!** 🚀
