
# 15 — DEPLOYMENT (دليل النشر والإنتاج)

**الإصدار:** 2.0 (Unified Edition)
**آخر تحديث:** 17 أكتوبر 2026
**الحالة:** ✅ مرجع شامل — يدمج 12 + دروس مُتراكمة
**الجمهور:** فريق التطوير + DevOps + AI Assistants
**المرجع الإلزامي:** يُقرأ قبل أي نشر

---

## 📌 الجزء صفر: مقدمة

### 0.1 نطاق هذه الوثيقة

هذا المرجع الشامل يغطي **كل ما يتعلق بنشر منصة RubikCare** على جميع البيئات:

- **Web (IIS)** — Blazor Server.
- **PWA (IIS)** — Blazor WebAssembly.
- **API (ASP.NET Core)** — Backend REST API.
- **Mobile (MAUI)** — Android APK/AAB + iOS IPA.

### 0.2 الأهداف

1. **توحيد إجراءات النشر** عبر جميع البيئات.
2. **توثيق الدروس المستفادة** من المشاكل الحقيقية.
3. **توفير سكربتات أتمتة** لتقليل الأخطاء اليدوية.
4. **إنشاء قوائم تحقق** لضمان الجودة.

### 0.3 الوثائق المرجعية

- **[01 — ARCHITECTURE](./01-ARCHITECTURE.md)** — لفهم بنية المشاريع.
- **[02 — IDENTITY](./02-IDENTITY.md)** — لفهم إدارة الجلسات.
- **[12 — CACHING](./12-CACHING.md)** — لفهم `IMemoryCache` + `IUserSessionService`.
- **[14 — PWA](./14-PWA.md)** — لتفاصيل PWA.
- **[13 — MAUI](./13-MAUI.md)** — لتفاصيل MAUI.

---

## 🗺️ الجزء الأول: خريطة البيئات

### 1.1 البيئات المعتمدة

| البيئة | الغرض | رابط Web | رابط API | مسار السيرفر |
|--------|-------|----------|----------|--------------|
| **Local** | التطوير والاختبار المبدئي | `https://localhost:xxxx` | `https://localhost:yyyy` | `C:\Users\{user}\source\repos\...` |
| **Test / UAT** | الاختبار الشامل قبل النشر | `https://test.rubikcare.com` | `https://uat.rubikcare.com` | Web: `C:\WebSite\RubikCareNew`<br>API: `C:\WebSite\RubikCareUat` |
| **Stage (PWA)** | بيئة PWA للاختبار | `https://stagepu.rubikcare.com` | `https://uat.rubikcare.com` | PWA: `C:\WebSite\PU_RubicCareStage`<br>API: `C:\WebSite\RubikCareUat` |
| **Production** | البيئة الفعلية | `https://rubikcare.com` | `https://api.rubikcare.com` | Web: `C:\WebSite\RubikCareLive`<br>API: `C:\WebSite\RubikCareApi` |

### 1.2 ⚠️ قاعدة ذهبية

> **تطبيق الـ Web/PWA في بيئة الـ Test/Stage يجب أن يكون `ApiSettings:BaseUrl` فيه مساوياً لـ `https://uat.rubikcare.com` وليس الـ Live.**

### 1.3 IIS Sites على VPS

| الموقع | الدومين | App Pool |
|:---|:---|:---|
| `RubikCareNew` | `test.rubikcare.com` | `rubikcarenew` |
| `RubikCareUat` | `uat.rubikcare.com` | `RubikCareUat` |
| `PU_RubicCareStage` | `stagepu.rubikcare.com` | `PU_RubicCareStage` |
| `RubikCareLive` | `rubikcare.com` | `RubikCareLive` |
| `RubikCareApi` | `api.rubikcare.com` | `RubikCareApi` |

---

## 🏗️ الجزء الثاني: البنية التحتية المشتركة

### 2.1 المتطلبات الأساسية

| المكون | الإصدار الأدنى | الغرض |
|--------|----------------|-------|
| Windows Server | 2019+ | نظام التشغيل |
| IIS | 10+ | خادم الويب |
| .NET Runtime | 10.0+ | تشغيل التطبيقات |
| SQL Server | 2019+ | قاعدة البيانات |
| URL Rewrite Module | 2.1+ | إعادة كتابة الروابط (ضروري لـ SPA) |

### 2.2 🔧 إعداد IIS الأساسي

#### تثبيت URL Rewrite Module

```powershell
# التحقق من وجود الوحدة
Get-WebGlobalModule | Where-Object { $_.Name -like "*Rewrite*" }

# إذا لم تظهر، ثبّت من:
# https://www.iis.net/downloads/microsoft/url-rewrite

# التحقق بعد التثبيت
Test-Path "C:\Windows\System32\inetsrv\rewrite.dll"
```

#### إنشاء Application Pool

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

**⚠️ ملاحظة مهمة:** `managedRuntimeVersion:""` يعني "No Managed Code" — ضروري لـ ASP.NET Core.

#### منح الصلاحيات

```powershell
# منح صلاحيات القراءة للمجلد
icacls "C:\WebSite\RubikCareApi" /grant "IIS_IUSRS:(OI)(CI)RX" /T
icacls "C:\WebSite\PU_RubicCareStage" /grant "IIS_IUSRS:(OI)(CI)RX" /T

# منح صلاحيات الكتابة لمجلد الرفع (إذا لزم)
icacls "C:\WebSite\RubikCareApi\wwwroot\uploads" /grant "IIS_IUSRS:(OI)(CI)M" /T
```

---

## 🔌 الجزء الثالث: نشر API (ASP.NET Core)

### 3.1 نظرة عامة

**API هو نقطة الدخول الوحيدة للبيانات** في المنصة. جميع العملاء (Web, Mobile, PWA) يتواصلون معه عبر HTTP/REST.

### 3.2 🚀 إجراءات ما قبل النشر

```powershell
# 1. تأكد من عدم وجود أخطاء بناء
cd C:\RC\Rubikcare.Full.Migration
dotnet build RubikCare.Api.Web -c Release

# 2. تحقق من Migrations غير مطبقة
dotnet ef migrations list --project RubikCare.Infrastructure --startup-project RubikCare.Api.Web

# 3. اختبر محلياً
dotnet run --project RubikCare.Api.Web
```

### 3.3 📦 خطوات النشر

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

### 3.4 🧪 التحقق بعد النشر

```powershell
try {
    $r = Invoke-WebRequest -Uri "https://uat.rubikcare.com/health" -UseBasicParsing -TimeoutSec 10
    "✅ UAT API: $($r.StatusCode) - $($r.Content)"
} catch {
    "❌ UAT API: $($_.Exception.Message)"
}
```

### 3.5 🐛 الأخطاء الشائعة

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

**الحل:** تحقق من JWT Key و CORS.

#### ❌ الخطأ 3: `404 Not Found`

**الحل:** تحقق من نشر الـ Controllers الجديدة.

### 3.6 🚀 سكربت النشر الأوتوماتيكي (`deploy-api-uat.ps1`)

```powershell
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
Write-Host "`n💾 الخطوة 2: نسخة احتياطية..." -ForegroundColor Yellow
Copy-Item "$IISPath\appsettings.Test.json" "$IISPath\appsettings.Test.json.backup" -Force

# الخطوة 3: نشر المشروع
Write-Host "`n📦 الخطوة 3: نشر المشروع..." -ForegroundColor Yellow
Set-Location $ProjectPath
dotnet publish RubikCare.Api.Web -c Release -o $PublishPath -r win-x64 --self-contained false

if ($LASTEXITCODE -ne 0) {
    Write-Host "❌ فشل النشر!" -ForegroundColor Red
    C:\Windows\System32\inetsrv\appcmd start apppool $AppPoolName
    exit 1
}

# الخطوة 4: نسخ الملفات
Write-Host "`n📋 الخطوة 4: نسخ الملفات..." -ForegroundColor Yellow
Get-ChildItem $PublishPath -Exclude "appsettings.*.json" | 
    Copy-Item -Destination $IISPath -Recurse -Force

# الخطوة 5: تشغيل الـ App Pool
Write-Host "`n▶️  الخطوة 5: تشغيل الـ App Pool..." -ForegroundColor Yellow
C:\Windows\System32\inetsrv\appcmd start apppool $AppPoolName

# الخطوة 6: اختبار Health Check
Write-Host "`n🧪 الخطوة 6: اختبار Health Check..." -ForegroundColor Yellow
Start-Sleep -Seconds 5

try {
    $r = Invoke-WebRequest -Uri "$ApiUrl/health" -UseBasicParsing -TimeoutSec 15
    Write-Host "✅ Health Check: $($r.StatusCode)" -ForegroundColor Green
} catch {
    Write-Host "❌ Health Check: $($_.Exception.Message)" -ForegroundColor Red
}

Write-Host "`n================================================" -ForegroundColor Cyan
Write-Host "  🎉 تم نشر API بنجاح!" -ForegroundColor Green
Write-Host "  🔗 الرابط: $ApiUrl" -ForegroundColor Green
Write-Host "================================================`n" -ForegroundColor Cyan
```

---

## 🌐 الجزء الرابع: نشر Web (Blazor Server)

### 4.1 نظرة عامة

**تطبيق `RubikCare.Web` هو تطبيق Blazor Server** يعمل عبر SignalR.

> ⚠️ **ملاحظة معمارية:** يعتمد `RubikCare.Web` على `appsettings.{Environment}.json` لتحديد رابط الـ API المستهدف.

### 4.2 🚀 إجراءات ما قبل النشر

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

### 4.3 📦 خطوات النشر

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

### 4.4 ⚙️ ملف `web.config` للـ Blazor Server

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

### 4.5 🧪 التحقق بعد النشر

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

### 4.6 🔴 القسم 4.5: نشر Web — تحديث حرج (28 سبتمبر 2026)

#### ⚠️ مشكلة "الصفحة فارغة + تحميل بطيء" — Root Cause

**التاريخ:** 28 سبتمبر 2026
**البيئة:** `test.rubikcare.com` (RubikCare.Web)
**الأعراض:**
- صفحة Dashboard والتقارير تفتح ببطء شديد (> دقيقتين).
- ثم تفتح فارغة (بدون KPIs، Charts، Quick Access Data).
- الأزرار تعمل، لكن لا تُحمَّل أي بيانات من API.

#### 🎯 الأسباب الجذرية (3 مشاكل متتالية)

| # | السبب | التفصيل |
|---|-------|---------|
| 1 | **`ASPNETCORE_ENVIRONMENT` غير محدد** | `web.config` لا يحتوي على `<environmentVariables>` → التطبيق يعمل بـ `Production` افتراضياً → يقرأ `appsettings.Production.json` (BaseUrl خاطئ) بدلاً من `appsettings.Test.json` |
| 2 | **`appcmd recycle` لا يُعيد تحميل `web.config`** | `recycle` يعيد تشغيل عملية `w3wp` **بدون** قطع الاتصالات النشطة → الإعدادات القديمة تبقى |
| 3 | **`IUserSessionService` يُخزّن الجلسة في `IMemoryCache`** | `IMemoryCache` يبدأ فارغاً عند restart، لكن **الجلسة القديمة** تبقى في الكاش حتى Logout → `CurrentOrganizationId = null` |

#### 🔧 الحل الكامل

##### الخطوة 1: تعديل `web.config` (إضافة `ASPNETCORE_ENVIRONMENT`)

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
- `Test` → لبيئة `test.rubikcare.com`.
- `Production` → لبيئة `rubikcare.com`.
- `Development` → للتطوير المحلي فقط.

##### الخطوة 2: إعادة تشغيل نظيفة (Stop + Start)

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

##### الخطوة 3: مسح الجلسة القديمة

**بعد إعادة التشغيل:**
1. افتح المتصفح على `https://test.rubikcare.com`.
2. **سجّل خروج** (Logout) — لمسح `IMemoryCache` للجلسة القديمة.
3. **سجّل دخول** من جديد — لبناء جلسة جديدة بالـ `BaseUrl` الصحيح.
4. اختبر صفحة التقارير.

#### 📋 Checklist النشر الآمن للـ Web

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

#### 🎯 القاعدة الذهبية الجديدة

> **بعد أي تعديل في `web.config` أو `appsettings.*.json`:**
> 1. **`appcmd stop`** (وليس `recycle`).
> 2. **`Start-Sleep -Seconds 5`**.
> 3. **`appcmd start`**.
> 4. **تسجيل خروج ودخول** في المتصفح.
>
> **بدون هذه الخطوات:** التعديلات **لن تُطبَّق**، وستستمر الصفحات فارغة.

#### 🚀 سكربت النشر الصحيح (`deploy-web-test.ps1`)

```powershell
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

# ─── 2. التحقق من الإعدادات ───
Write-Host "`n🔍 الخطوة 2: التحقق من appsettings.Test.json..." -ForegroundColor Yellow
$testSettings = Get-Content "$IISPath\appsettings.Test.json" -Raw | ConvertFrom-Json
$baseUrl = $testSettings.ApiSettings.BaseUrl
if ($baseUrl -ne "https://uat.rubikcare.com") {
    Write-Host "❌ BaseUrl خاطئ: $baseUrl" -ForegroundColor Red
    C:\Windows\System32\inetsrv\appcmd start apppool $AppPoolName
    exit 1
}

# ─── 3. التحقق من ASPNETCORE_ENVIRONMENT ───
Write-Host "`n🔍 الخطوة 3: التحقق من ASPNETCORE_ENVIRONMENT..." -ForegroundColor Yellow
$webConfig = Get-Content "$IISPath\web.config" -Raw
if ($webConfig -notmatch 'value="Test"') {
    Write-Host "❌ ASPNETCORE_ENVIRONMENT غير محدد أو ليس Test" -ForegroundColor Red
    C:\Windows\System32\inetsrv\appcmd start apppool $AppPoolName
    exit 1
}

# ─── 4. النشر ───
Write-Host "`n📦 الخطوة 4: نشر المشروع..." -ForegroundColor Yellow
Set-Location $ProjectPath
dotnet publish RubikCare.Web -c Release -o $PublishPath
if ($LASTEXITCODE -ne 0) {
    C:\Windows\System32\inetsrv\appcmd start apppool $AppPoolName
    exit 1
}

# ─── 5. نسخ الملفات ───
Write-Host "`n📋 الخطوة 5: نسخ الملفات..." -ForegroundColor Yellow
Get-ChildItem $PublishPath -Exclude "web.config","appsettings.*.json" | 
    Copy-Item -Destination $IISPath -Recurse -Force

# ─── 6. تشغيل App Pool ───
Write-Host "`n▶️  الخطوة 6: تشغيل App Pool..." -ForegroundColor Yellow
C:\Windows\System32\inetsrv\appcmd start apppool $AppPoolName

# ─── 7. اختبار ───
Write-Host "`n🧪 الخطوة 7: اختبار الموقع..." -ForegroundColor Yellow
Start-Sleep -Seconds 8
try {
    $r = Invoke-WebRequest -Uri "$SiteUrl" -UseBasicParsing -TimeoutSec 20
    Write-Host "✅ الموقع يعمل: $($r.StatusCode)" -ForegroundColor Green
} catch {
    Write-Host "❌ الموقع لا يعمل: $($_.Exception.Message)" -ForegroundColor Red
}

Write-Host "`n================================================" -ForegroundColor Cyan
Write-Host "  🎉 تم النشر بنجاح!" -ForegroundColor Green
Write-Host "  ⚠️  تذكير: سجّل خروج ثم دخول في المتصفح (لمسح IMemoryCache)" -ForegroundColor Yellow
Write-Host "================================================`n" -ForegroundColor Cyan
```

---

## 📱 الجزء الخامس: نشر PWA (Blazor WebAssembly)

### 5.1 نظرة عامة

**تطبيق `RubikCare.PWA` هو Blazor WebAssembly Standalone.** يختلف نشره كلياً عن الـ Web والـ API:

| الجانب | Web / API | PWA (Blazor WASM) |
|--------|-----------|-------------------|
| **طبيعة التشغيل** | خادم .NET يعمل فعلياً | ملفات ثابتة تُخدم من IIS |
| **الـ App Pool** | يحتاج .NET Runtime | **No Managed Code** |
| **`web.config`** | ASP.NET Core Module | **URL Rewrite + MIME types** |
| **منطق التطبيق** | يعمل على الخادم | يعمل في المتصفح |

> 🔴 **القاعدة المحورية:** تطبيق الـ PWA لا يحتاج خادم .NET، لكنه يحتاج **إعداد IIS دقيقاً** ليخدم ملفات `.wasm` و `.dat` و `.js` بشكل صحيح.

### 5.2 🏗️ المتطلبات الأساسية

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

### 5.3 🎯 القاعدة الذهبية: المسار الفيزيائي

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

### 5.4 📄 ملف `web.config` الكامل للـ PWA

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

### 5.5 🔢 مشكلة الأسماء المُجزّأة (Hashed Filenames)

**أضف هذا الـ Target في `RubikCare.PWA.csproj`:**

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

### 5.6 📦 خطوات النشر

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

### 5.7 🤖 سكربت النشر الأوتوماتيكي (`deploy-stage.ps1`)

```powershell
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

# الخطوة 3: نشر المشروع
Write-Host "`n📦 الخطوة 3: نشر المشروع..." -ForegroundColor Yellow
Set-Location $ProjectPath
dotnet publish RubikCare.PWA -c Release -o $PublishPath

if ($LASTEXITCODE -ne 0) {
    C:\Windows\System32\inetsrv\appcmd start apppool $AppPoolName
    exit 1
}

# الخطوة 4: نسخ الملفات
Write-Host "`n📋 الخطوة 4: نسخ الملفات..." -ForegroundColor Yellow
Copy-Item -Path "$PublishPath\*" -Destination $IISPath -Recurse -Force

# الخطوة 5: تشغيل الـ App Pool
Write-Host "`n▶️  الخطوة 5: تشغيل الـ App Pool..." -ForegroundColor Yellow
C:\Windows\System32\inetsrv\appcmd start apppool $AppPoolName

# الخطوة 6: اختبار
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

### 5.8 🐛 الأخطاء الشائعة في PWA

#### ❌ الخطأ 1: شاشة زرقاء + `Unexpected token '<'`

**السبب:** المتصفح يتلقى `index.html` بدلاً من ملف JavaScript.

**الحل:**
```powershell
Test-Path "C:\WebSite\PU_RubicCareStage\wwwroot\_framework\blazor.webassembly.js"
C:\Windows\System32\inetsrv\appcmd list vdir "PU_RubicCareStage/" /text:physicalPath
```

#### ❌ الخطأ 2: `SRI integrity checks failed`

**السبب:** ملفات قديمة بـ hash مختلف، أو المسار الفيزيائي خاطئ.

**الحل:** امسح المجلد وأعد النشر.

#### ❌ الخطأ 3: `manifest.webmanifest` يرجع 404

**الحل:** تأكد من MIME type في `web.config`.

#### ❌ الخطأ 4: الموقع لا يفتح (خطأ 500)

**الحل:** تحقق من URL Rewrite Module.

#### ❌ الخطأ 5: Service Worker يخدم ملفات قديمة

**الحل:** اختبر دائماً في **InPrivate** (`Ctrl + Shift + N`).

#### ❌ الخطأ 6: صورة البروفايل لا تظهر + CORS

**الحل:** أضف CORS في الـ API.

**⚠️ التفاصيل الكاملة في `14-PWA`.**

---

## 📲 الجزء السادس: نشر Mobile (MAUI)

### 6.1 🤖 نشر Android (APK / AAB)

```bash
# إنشاء Keystore (مرة واحدة)
keytool -genkey -v -keystore rubikcare.keystore -alias rubikcare -keyalg RSA -keysize 2048 -validity 10000

# APK للتوزيع المباشر
cd C:\RC\Rubikcare.Full.Migration
dotnet publish RubikCare.Mobile -f net10.0-android -c Release /p:AndroidPackageFormat=apk

# AAB لـ Google Play
dotnet publish RubikCare.Mobile -f net10.0-android -c Release /p:AndroidPackageFormat=aab
```

### 6.2 🍎 نشر iOS (IPA)

```bash
# من Mac
dotnet publish RubikCare.Mobile -f net10.0-ios -c Release /p:ArchiveOnBuild=true
```

### 6.3 🐛 أخطاء شائعة في Mobile

#### ❌ `Ambiguous routes matched`

**الحل:** الصفحات الرئيسية في `AppShell.xaml` فقط، الفرعية في `AppShell.xaml.cs`.

**⚠️ التفاصيل في `13-MAUI`.**

#### ❌ `JavaProxyThrowable`

**الحل:** وحّد إصدارات MAUI:
```xml
<PackageReference Include="Microsoft.Maui.Controls" Version="10.0.20" />
<PackageReference Include="Microsoft.AspNetCore.Components.WebView.Maui" Version="10.0.20" />
```

---

## 🔐 الجزء السابع: إعدادات الأمان

### 7.1 مفاتيح الإنتاج

```bash
# للتطوير
dotnet user-secrets set "Jwt:Key" "dev-key-12345678" --project RubikCare.Api.Web

# للإنتاج
setx Jwt__Key "production-key-32-chars-minimum" /M
```

### 7.2 حماية البيانات (Data Protection)

```csharp
builder.Services.AddDataProtection()
    .PersistKeysToFileSystem(new DirectoryInfo(@"C:\Keys\RubikCare"))
    .SetApplicationName("RubikCare");
```

### 7.3 سياسات CORS

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

## 💾 الجزء الثامن: النسخ الاحتياطي

### 8.1 قاعدة البيانات

```sql
BACKUP DATABASE RubikCare 
TO DISK = 'C:\Backups\RubikCare_Full.bak' 
WITH INIT, COMPRESSION;
```

### 8.2 نسخ ملفات الموقع

```powershell
$timestamp = Get-Date -Format "yyyyMMdd_HHmmss"
Compress-Archive -Path "C:\WebSite\PU_RubicCareStage\*" `
    -DestinationPath "C:\Backups\PWA_$timestamp.zip" -Force
```

---

## 🔍 الجزء التاسع: Health Checks

```csharp
builder.Services.AddHealthChecks()
    .AddDbContextCheck<BusinessDbContext>();

app.MapHealthChecks("/health");
```

```powershell
Invoke-WebRequest -Uri "https://api.rubikcare.com/health" -UseBasicParsing
```

---

## 🔴 الجزء العاشر: Root Cause Analysis — الفشل الصامت

**Date:** August 20, 2026
**Developer:** Youssef Shady
**Status:** Resolved

### 10.1 وصف المشكلة

**"Silent Failure"** — عمليات الحفظ تفشل بدون أي رسالة خطأ في UI أو Console.

### 10.2 السبب الجذري (3 عوامل)

| # | العامل | التفاصيل |
|---|--------|----------|
| 1 | **Swallowed Exceptions** | `GetAsync` و `PostAsync` تعيد `null` بصمت عند الفشل بدلاً من رمي Exception |
| 2 | **Misconfigured BaseUrl** | `appsettings.Test.json` يشير إلى Live API بدلاً من UAT |
| 3 | **API Environment Conflict** | كود Web جديد منشور بينما API المستهدف قديم |

### 10.3 الحل

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

## 📚 الجزء الحادي عشر: Lessons Learned (الدروس المستفادة)

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

### 🔴 الدرس 6: `appcmd stop` بدل `recycle`

**الدرس:** `recycle` لا يُطبّق `web.config` الجديد بشكل موثوق. استخدم `stop` + `start`.

### 🔴 الدرس 7: `ASPNETCORE_ENVIRONMENT`

**الدرس:** بدون تحديد `ASPNETCORE_ENVIRONMENT`، التطبيق يقرأ `Production` افتراضياً — مما يسبب فشلاً صامتاً.

---

## 🚫 الجزء الثاني عشر: Critical Prohibitions (الممنوعات المطلقة)

| # | المحظور | السبب | البديل |
|---|---------|-------|--------|
| 1 | ❌ حذف `web.config` عند النشر | يحتوي إعدادات IIS | احتفظ به دائماً |
| 2 | ❌ نسخ الملفات فوق القديمة بدون مسح | تضارب hash → `SRI failed` | امسح أولاً ثم انسخ |
| 3 | ❌ تعديل `web.config` يدوياً على السيرفر | ستفقده في النشر القادم | اجعله جزءاً من المشروع |
| 4 | ❌ تغيير المسار الفيزيائي إلى `wwwroot` | تضارب مع `Serve subdir` | المسار = الجذر دائماً |
| 5 | ❌ اختبار في المتصفح العادي بعد نشر PWA | Service Worker قديم | InPrivate دائماً |
| 6 | ❌ نشر بدون إيقاف App Pool | قفل الملفات | أوقف أولاً |
| 7 | ❌ نشر API إلى مجلد PWA | تطبيقان منفصلان | لكلٍ مجلده |
| 8 | ⭐ ❌ `appcmd recycle` بعد تعديل `web.config` | لا يُعيد التحميل | `stop` + `start` |
| 9 | ⭐ ❌ الاعتماد على `appsettings.Production.json` في Test | BaseUrl خاطئ | `ASPNETCORE_ENVIRONMENT = "Test"` |
| 10 | ⭐ ❌ نسيان تسجيل خروج/دخول بعد النشر | `IMemoryCache` قديم | Logout + Login |

---

## 📋 الجزء الثالث عشر: Master Deployment Checklist

### قبل أي نشر

- [ ] هل تم حفظ جميع التعديلات؟
- [ ] هل تم اختبار التطبيق محلياً؟
- [ ] هل تم أخذ نسخة احتياطية من قاعدة البيانات؟
- [ ] هل ملفات البيئة تشير إلى الروابط الصحيحة؟
- [ ] ⭐ **هل `ASPNETCORE_ENVIRONMENT` مضبوط في `web.config`؟**

### أثناء النشر

- [ ] هل تم إيقاف الـ App Pool؟
- [ ] هل تم حفظ نسخة احتياطية من `web.config` و `appsettings`؟
- [ ] هل تم نسخ الملفات بنجاح؟
- [ ] ⭐ **هل استخدمت `stop` + `start` (وليس `recycle`)؟**

### بعد النشر

- [ ] هل تم تشغيل الـ App Pool؟
- [ ] هل Health Check يعيد `200`؟
- [ ] هل تم الاختبار من InPrivate (للـ PWA)؟
- [ ] هل السجلات خالية من الأخطاء؟
- [ ] هل العمليات الرئيسية تعمل؟
- [ ] ⭐ **هل تم تسجيل خروج ودخول (لمسح `IMemoryCache`)?**

---

## 🔍 الجزء الرابع عشر: Troubleshooting Cheatsheet

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

## 🔗 الجزء الخامس عشر: روابط ذات صلة

- [00 — MASTER-INDEX](./00-MASTER-INDEX.md)
- [00B — PROJECT-BACKLOG](./00B-PROJECT-BACKLOG.md)
- [01 — ARCHITECTURE](./01-ARCHITECTURE.md)
- [02 — IDENTITY](./02-IDENTITY.md)
- [12 — CACHING](./12-CACHING.md)
- [13 — MAUI](./13-MAUI.md)
- [14 — PWA](./14-PWA.md)
- [16 — TROUBLESHOOTING](./16-TROUBLESHOOTING.md)

---

## 📝 الجزء السادس عشر: سجل التغييرات

| الإصدار | التاريخ | التغييرات |
|:---:|:---|:---|
| **1.0** | **29 أغسطس 2026** | الإصدار الأولي — `12-deployment-guide.md` |
| **2.0** | **17 أكتوبر 2026** | **`15-DEPLOYMENT` v2.0** — دمج `12` + القسم 4.5 الحرج (`ASPNETCORE_ENVIRONMENT` + `appcmd stop`) + الدرس 6 + 7 + إشارات |

---
## 🚀 الجزء السابع عشر: النشر الآلي (Automated Deployment) — جديد

### 17.1 نظرة عامة

**الهدف:** تقليل الأخطاء البشرية عند النشر عبر:
- **الحالة** لم ينفذ بعد.
- **نسخ احتياطي تلقائي** قبل كل عملية.
- **استبعاد الإعدادات** (`appsettings.*.json` + `web.config`) تلقائياً.
- **Health Check** قبل وبعد النشر.
- **Logging كامل** لكل عملية.
- **Rollback سريع** في حالة الفشل.

### 17.2 السكريبتات الثلاثة

| السكريبت | الغرض | المصدر | الهدف |
|-----------|--------|--------|--------|
| `Deploy-FromStage.ps1` | نشر من Stage → Permanent | `C:\WebSite\_Stage\{Project}` | `C:\WebSite\{Permanent}` |
| `Promote-ToProduction.ps1` | ترقية من Dev → Prod | `C:\WebSite\{Dev}` | `C:\WebSite\{Prod}` |
| `Deploy-All.ps1` | نشر كل المشاريع | (تسلسل) | (تسلسل) |

### 17.3 `Deploy-FromStage.ps1` (الكود الكامل)

```powershell
<#
.SYNOPSIS
    Deploy-FromStage.ps1 — نشر مشروع من مجلد Stage إلى المجلد الدائم (مع الحفاظ على الإعدادات).
.DESCRIPTION
    - نسخ احتياطي تلقائي.
    - إيقاف/تشغيل App Pool.
    - استبعاد ملفات الإعدادات (appsettings, web.config).
    - Health Check (اختياري).
.PARAMETER ProjectName
    اسم المشروع: PWA, Web (Blazor Server), API
.PARAMETER AppPoolName
    اسم App Pool في IIS (مثال: PU_RubicCareStage).
.PARAMETER HealthCheckUrl
    (اختياري) رابط للتحقق من صحة النشر (مثال: https://stagepu.rubikcare.com).
.EXAMPLE
    .\Deploy-FromStage.ps1 -ProjectName PWA -AppPoolName "PU_RubicCareStage" -HealthCheckUrl "https://stagepu.rubikcare.com"
#>

param(
    [Parameter(Mandatory=$true)]
    [ValidateSet("PWA", "Web", "API")]
    [string]$ProjectName,

    [Parameter(Mandatory=$true)]
    [string]$AppPoolName,

    [string]$HealthCheckUrl = ""
)

# ═══════════════════════════════════════════════════════════
# 1. الإعدادات
# ═══════════════════════════════════════════════════════════
$BasePath = "C:\WebSite"
$StagePath = Join-Path $BasePath "_Stage\$ProjectName"
$PermanentPath = switch ($ProjectName) {
    "PWA" { Join-Path $BasePath "PU_RubicCareStage" }
    "Web" { Join-Path $BasePath "RubikCareNew" }
    "API" { Join-Path $BasePath "RubikCareApi" }
}

# الملفات التي يجب استبعادها (لأنها تحتوي على إعدادات البيئة)
$ExcludeFiles = @(
    "appsettings.json",
    "appsettings.Development.json",
    "appsettings.Production.json",
    "appsettings.Staging.json",
    "appsettings.UAT.json",
    "appsettings.Test.json",
    "appsettings.Secrets.json",
    "web.config"
)

# مجلدات يجب استبعادها (تحتوي على بيانات يجب أن تبقى)
$ExcludeFolders = @(
    "Logs",
    "logs",
    "uploads",
    "Uploads",
    "wwwroot\uploads"
)

$Timestamp = Get-Date -Format "yyyyMMdd_HHmmss"
$BackupPath = Join-Path $BasePath "_Backups\$ProjectName`_$Timestamp"
$LogFile = Join-Path $BasePath "_Logs\Deploy-$ProjectName-$Timestamp.log"

# ═══════════════════════════════════════════════════════════
# 2. دوال مساعدة
# ═══════════════════════════════════════════════════════════
function Write-Log {
    param([string]$Message, [string]$Level = "INFO")
    $Time = Get-Date -Format "yyyy-MM-dd HH:mm:ss"
    $Line = "[$Time] [$Level] $Message"
    Write-Host $Line -ForegroundColor $(switch ($Level) {
        "ERROR" { "Red" }
        "WARN" { "Yellow" }
        "SUCCESS" { "Green" }
        default { "White" }
    })
    Add-Content -Path $LogFile -Value $Line -ErrorAction SilentlyContinue
}

function Test-Prerequisites {
    Write-Log "🔍 التحقق من المتطلبات..." -Level "INFO"

    if (-not (Test-Path $StagePath)) {
        Write-Log "❌ المجلد المرحلي غير موجود: $StagePath" -Level "ERROR"
        return $false
    }

    if (-not (Test-Path $PermanentPath)) {
        Write-Log "❌ المجلد الدائم غير موجود: $PermanentPath" -Level "ERROR"
        return $false
    }

    $permanentAppSettings = Join-Path $PermanentPath "appsettings.json"
    if (-not (Test-Path $permanentAppSettings)) {
        Write-Log "⚠️ تحذير: appsettings.json غير موجود في المجلد الدائم — قد يفشل النشر." -Level "WARN"
    }

    return $true
}

function Invoke-Backup {
    Write-Log "📦 إنشاء نسخة احتياطية: $BackupPath" -Level "INFO"
    New-Item -ItemType Directory -Path $BackupPath -Force | Out-Null

    Copy-Item -Path "$PermanentPath\*" -Destination $BackupPath -Recurse -Force -ErrorAction SilentlyContinue

    $backupSize = (Get-ChildItem $BackupPath -Recurse | Measure-Object -Property Length -Sum).Sum / 1MB
    Write-Log "✅ تم إنشاء النسخة الاحتياطية (الحجم: $([math]::Round($backupSize, 2)) MB)" -Level "SUCCESS"
}

function Stop-AppPool {
    Write-Log "⏸️ إيقاف App Pool: $AppPoolName" -Level "INFO"
    & "$env:SystemRoot\System32\inetsrv\appcmd.exe" stop apppool /apppool.name:"$AppPoolName" 2>&1 | Out-Null
    Start-Sleep -Seconds 2
}

function Start-AppPool {
    Write-Log "▶️ تشغيل App Pool: $AppPoolName" -Level "INFO"
    & "$env:SystemRoot\System32\inetsrv\appcmd.exe" start apppool /apppool.name:"$AppPoolName" 2>&1 | Out-Null
}

function Invoke-CopyWithExclusions {
    Write-Log "📋 نسخ الملفات (مع استبعاد الإعدادات)..." -Level "INFO"

    $robocopyArgs = @(
        "`"$StagePath`"",
        "`"$PermanentPath`"",
        "/MIR",
        "/XF " + ($ExcludeFiles -join " "),
        "/XD " + ($ExcludeFolders -join " "),
        "/R:2",
        "/W:5",
        "/NP",
        "/NFL",
        "/NDL",
        "/LOG+:`"$LogFile`""
    )

    $robocopyCmd = "robocopy " + ($robocopyArgs -join " ")
    Invoke-Expression $robocopyCmd

    $exitCode = $LASTEXITCODE

    if ($exitCode -ge 8) {
        Write-Log "❌ فشل النسخ (robocopy exit code: $exitCode)" -Level "ERROR"
        return $false
    }

    Write-Log "✅ تم نسخ الملفات بنجاح (robocopy exit code: $exitCode)" -Level "SUCCESS"
    return $true
}

function Test-Health {
    if ([string]::IsNullOrEmpty($HealthCheckUrl)) {
        Write-Log "⏭️ تخطي Health Check (لم يُحدد URL)." -Level "INFO"
        return $true
    }

    Write-Log "🩺 Health Check: $HealthCheckUrl" -Level "INFO"
    Start-Sleep -Seconds 5

    try {
        $response = Invoke-WebRequest -Uri $HealthCheckUrl -UseBasicParsing -TimeoutSec 30
        if ($response.StatusCode -eq 200) {
            Write-Log "✅ Health Check نجح (HTTP $($response.StatusCode))" -Level "SUCCESS"
            return $true
        } else {
            Write-Log "⚠️ Health Check أعاد HTTP $($response.StatusCode)" -Level "WARN"
            return $true
        }
    } catch {
        Write-Log "❌ Health Check فشل: $($_.Exception.Message)" -Level "ERROR"
        return $false
    }
}

# ═══════════════════════════════════════════════════════════
# 3. التنفيذ
# ═══════════════════════════════════════════════════════════
Write-Log "═══════════════════════════════════════════════════════════" -Level "INFO"
Write-Log "🚀 بدء النشر: $ProjectName" -Level "INFO"
Write-Log "   Stage:     $StagePath" -Level "INFO"
Write-Log "   Permanent: $PermanentPath" -Level "INFO"
Write-Log "   App Pool:  $AppPoolName" -Level "INFO"
Write-Log "═══════════════════════════════════════════════════════════" -Level "INFO"

if (-not (Test-Prerequisites)) {
    Write-Log "❌ فشل النشر (المتطلبات غير مكتملة)." -Level "ERROR"
    exit 1
}

Invoke-Backup
Stop-AppPool
$copySuccess = Invoke-CopyWithExclusions
Start-AppPool

if (-not $copySuccess) {
    Write-Log "❌ فشل النشر (نسخ الملفات)." -Level "ERROR"
    Write-Log "💡 يمكنك استعادة النسخة الاحتياطية من: $BackupPath" -Level "INFO"
    exit 1
}

$healthSuccess = Test-Health

if ($healthSuccess) {
    Write-Log "🎉 تم النشر بنجاح!" -Level "SUCCESS"
    Write-Log "📁 النسخة الاحتياطية: $BackupPath" -Level "INFO"
    Write-Log "📄 السجل: $LogFile" -Level "INFO"
    exit 0
} else {
    Write-Log "⚠️ النشر اكتمل لكن Health Check فشل — راجع السجل." -Level "WARN"
    Write-Log "💡 يمكنك استعادة النسخة الاحتياطية من: $BackupPath" -Level "INFO"
    exit 2
}
```

### 17.4 `Promote-ToProduction.ps1` (الكود الكامل)

```powershell
<#
.SYNOPSIS
    Promote-ToProduction.ps1 — نقل مشروع من Dev إلى Production (مع الحفاظ على إعدادات الإنتاج).
.DESCRIPTION
    - التحقق من استقرار Dev (Health Check).
    - نسخ احتياطي للإنتاج.
    - استبعاد إعدادات الإنتاج.
    - Health Check إنتاجي.
.PARAMETER ProjectName
    اسم المشروع: PWA, Web, API
.PARAMETER DevAppPool
    اسم App Pool في Dev (مثال: rubikcarenew).
.PARAMETER ProdAppPool
    اسم App Pool في Prod (مثال: RubikCareLive).
.PARAMETER DevHealthCheckUrl
    رابط Health Check في Dev.
.PARAMETER ProdHealthCheckUrl
    رابط Health Check في Prod.
.EXAMPLE
    .\Promote-ToProduction.ps1 -ProjectName Web -DevAppPool "rubikcarenew" -ProdAppPool "RubikCareLive" -DevHealthCheckUrl "https://test.rubikcare.com" -ProdHealthCheckUrl "https://rubikcare.com"
#>

param(
    [Parameter(Mandatory=$true)]
    [ValidateSet("PWA", "Web", "API")]
    [string]$ProjectName,

    [Parameter(Mandatory=$true)]
    [string]$DevAppPool,

    [Parameter(Mandatory=$true)]
    [string]$ProdAppPool,

    [string]$DevHealthCheckUrl = "",
    [string]$ProdHealthCheckUrl = ""
)

$BasePath = "C:\WebSite"
$DevPath = switch ($ProjectName) {
    "PWA" { Join-Path $BasePath "PU_RubicCareStage" }
    "Web" { Join-Path $BasePath "RubikCareNew" }
    "API" { Join-Path $BasePath "RubikCareApi" }
}
$ProdPath = switch ($ProjectName) {
    "PWA" { Join-Path $BasePath "PU_RubicCareLive" }
    "Web" { Join-Path $BasePath "RubikCareLive" }
    "API" { Join-Path $BasePath "RubikCareApiLive" }
}

$ExcludeFiles = @(
    "appsettings.json",
    "appsettings.Production.json",
    "appsettings.Secrets.json",
    "web.config"
)

$ExcludeFolders = @(
    "Logs",
    "logs",
    "uploads",
    "Uploads"
)

$Timestamp = Get-Date -Format "yyyyMMdd_HHmmss"
$BackupPath = Join-Path $BasePath "_Backups\Prod_$ProjectName`_$Timestamp"
$LogFile = Join-Path $BasePath "_Logs\Promote-$ProjectName-$Timestamp.log"

# ... (نفس دوال Write-Log, Test-Prerequisites, Invoke-Backup, Stop-AppPool, Start-AppPool, Invoke-CopyWithExclusions, Test-Health — مع تمرير Source/Destination كـ Parameters)

Write-Log "═══════════════════════════════════════════════════════════" -Level "INFO"
Write-Log "🚀 بدء الترقية: $ProjectName" -Level "INFO"
Write-Log "   Dev:  $DevPath" -Level "INFO"
Write-Log "   Prod: $ProdPath" -Level "INFO"
Write-Log "═══════════════════════════════════════════════════════════" -Level "INFO"

if (-not [string]::IsNullOrEmpty($DevHealthCheckUrl)) {
    Write-Log "🩺 التحقق من استقرار Dev..." -Level "INFO"
    if (-not (Test-Health -HealthCheckUrl $DevHealthCheckUrl)) {
        Write-Log "❌ Dev غير مستقر — إلغاء الترقية." -Level "ERROR"
        exit 1
    }
}

Invoke-Backup
Stop-AppPool -AppPoolName $ProdAppPool
$copySuccess = Invoke-CopyWithExclusions -Source $DevPath -Destination $ProdPath -ExcludeFiles $ExcludeFiles -ExcludeFolders $ExcludeFolders
Start-AppPool -AppPoolName $ProdAppPool

if (-not $copySuccess) {
    Write-Log "❌ فشل الترقية (نسخ الملفات)." -Level "ERROR"
    Write-Log "💡 يمكنك استعادة النسخة الاحتياطية من: $BackupPath" -Level "INFO"
    exit 1
}

$healthSuccess = Test-Health -HealthCheckUrl $ProdHealthCheckUrl

if ($healthSuccess) {
    Write-Log "🎉 تمت الترقية إلى Production بنجاح!" -Level "SUCCESS"
    exit 0
} else {
    Write-Log "⚠️ الترقية اكتملت لكن Health Check فشل — راجع السجل." -Level "WARN"
    Write-Log "💡 يمكنك استعادة النسخة الاحتياطية من: $BackupPath" -Level "INFO"
    exit 2
}
```

### 17.5 `Deploy-All.ps1` (الكود الكامل)

```powershell
<#
.SYNOPSIS
    Deploy-All.ps1 — نشر كل المشاريع (PWA + Web + API) من Stage إلى Permanent.
#>

$ErrorActionPreference = "Stop"

$Projects = @(
    @{ Name = "API"; AppPool = "RubikCareApi"; HealthCheck = "https://uat.rubikcare.com/api/ping" },
    @{ Name = "Web"; AppPool = "RubikCareNew"; HealthCheck = "https://test.rubikcare.com" },
    @{ Name = "PWA"; AppPool = "PU_RubicCareStage"; HealthCheck = "https://stagepu.rubikcare.com" }
)

$Failed = @()

foreach ($project in $Projects) {
    Write-Host "`n═══════════════════════════════════════════════════════════" -ForegroundColor Cyan
    Write-Host "🚀 نشر: $($project.Name)" -ForegroundColor Cyan
    Write-Host "═══════════════════════════════════════════════════════════" -ForegroundColor Cyan

    try {
        & ".\Deploy-FromStage.ps1" `
            -ProjectName $project.Name `
            -AppPoolName $project.AppPool `
            -HealthCheckUrl $project.HealthCheck

        if ($LASTEXITCODE -ne 0) {
            $Failed += $project.Name
        }
    } catch {
        Write-Host "❌ فشل نشر $($project.Name): $($_.Exception.Message)" -ForegroundColor Red
        $Failed += $project.Name
    }
}

Write-Host "`n═══════════════════════════════════════════════════════════" -ForegroundColor Cyan
if ($Failed.Count -eq 0) {
    Write-Host "🎉 تم نشر كل المشاريع بنجاح!" -ForegroundColor Green
    exit 0
} else {
    Write-Host "⚠️ فشل نشر: $($Failed -join ', ')" -ForegroundColor Yellow
    exit 1
}
```

### 17.6 خطوات الاستخدام (يومياً)

#### 1. النشر من المحلي إلى Stage (على السيرفر):
- انسخ `publish` إلى `C:\WebSite\_Stage\PWA` (أو `Web` / `API`).
- **⚠️ لا تنسخ `appsettings.json` أو `web.config`** (السكريبت سيحتفظ بها من المجلد الدائم).

#### 2. النشر من Stage إلى Permanent:
```powershell
.\Deploy-FromStage.ps1 -ProjectName PWA -AppPoolName "PU_RubicCareStage" -HealthCheckUrl "https://stagepu.rubikcare.com"
```

#### 3. النشر من Dev إلى Production (بعد التأكد من استقرار Dev):
```powershell
.\Promote-ToProduction.ps1 -ProjectName Web -DevAppPool "rubikcarenew" -ProdAppPool "RubikCareLive" -DevHealthCheckUrl "https://test.rubikcare.com" -ProdHealthCheckUrl "https://rubikcare.com"
```

### 17.7 Health Check Endpoint

**الـ API يحتوي على `/api/ping`:**

```csharp
app.MapGet("/api/ping", () => Results.Ok(new
{
    success = true,
    message = "✅ API is working!",
    time = DateTime.Now,
    environment = app.Environment.EnvironmentName
}));
```

**الاستخدام في السكريبتات:**
```powershell
-HealthCheckUrl "https://uat.rubikcare.com/api/ping"
```

**النتيجة المتوقعة:**
```json
{
  "success": true,
  "message": "✅ API is working!",
  "time": "2026-10-26T...",
  "environment": "Staging"
}
```

### 17.8 Checklist النشر الآلي

- [ ] المجلد `_Stage\{Project}` موجود (نسخ الملفات إليه أولاً).
- [ ] `appsettings.json` + `web.config` **موجودان** في المجلد الدائم (لن يُستبدلا).
- [ ] `AppPoolName` صحيح (من خريطة البيئات).
- [ ] `HealthCheckUrl` صحيح (`/api/ping` للـ API — أو الدومين للـ Web/PWA).
- [ ] السكريبت يعمل كـ Administrator.

### 17.9 Rollback (في حالة الفشل)

**كل سكريبت ينشئ نسخة احتياطية في:**
```
C:\WebSite\_Backups\{Project}_{Timestamp}\
```

**للاستعادة:**
```powershell
# 1. إيقاف App Pool
C:\Windows\System32\inetsrv\appcmd stop apppool "PU_RubicCareStage"

# 2. حذف الملفات الجديدة (باستثناء web.config)
Get-ChildItem "C:\WebSite\PU_RubicCareStage" -Exclude "web.config","appsettings.*.json" | 
    Remove-Item -Recurse -Force

# 3. استعادة النسخة الاحتياطية
Copy-Item -Path "C:\WebSite\_Backups\PWA_20261026_120000\*" `
    -Destination "C:\WebSite\PU_RubicCareStage\" -Recurse -Force

# 4. تشغيل App Pool
C:\Windows\System32\inetsrv\appcmd start apppool "PU_RubicCareStage"
```

---

**© 2026 RubikCare — للاستخدام الداخلي**

**نهاية الوثيقة ✅**
```
