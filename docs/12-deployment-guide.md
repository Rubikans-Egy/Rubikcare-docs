# 12 - دليل النشر والإنتاج (Deployment Guide)

**آخر تحديث: 17 مايو 2026**

---

## مقدمة

هذا المرجع يغطي كل ما يتعلق بنشر منصة RubikCare على بيئات الإنتاج: Web (IIS)، Mobile (APK/AAB)، وإعدادات الأمان.

---

## نشر تطبيق الويب (IIS)

### المتطلبات

- Windows Server 2019+
- IIS 10+
- .NET 10 Hosting Bundle
- SQL Server 2019+

### خطوات النشر

#### 1. نشر الملفات

```bash
# انشر من مجلد Api.Web
dotnet publish -c Release -o ./publish
```

#### 2. إعداد IIS

1. أنشئ Application Pool جديد:
   - Name: `RubikCare`
   - .NET CLR Version: `No Managed Code`
   - Identity: `ApplicationPoolIdentity`

2. أضف موقع جديد:
   - Site Name: `RubikCare`
   - Physical Path: `C:\inetpub\wwwroot\rubikcare`
   - Binding: `https://rubikcare.com` (Port 443)

3. امنح صلاحيات المجلد:
   ```
   IIS_IUSRS → Read & Execute
   NETWORK SERVICE → Modify (لرفع الملفات)
   ```

#### 3. إعدادات appsettings.json

```json
{
  "ConnectionStrings": {
    "DefaultConnection": "Server=.;Database=RubikCare;Trusted_Connection=True;TrustServerCertificate=True;"
  },
  "Jwt": {
    "Key": "YOUR_PRODUCTION_KEY_32_CHARS_MIN",
    "Issuer": "https://rubikcare.com",
    "ExpireDays": 7
  },
  "Google": {
    "ClientId": "YOUR_GOOGLE_CLIENT_ID",
    "ClientSecret": "YOUR_GOOGLE_CLIENT_SECRET"
  }
}
```

#### 4. شهادة SSL

```powershell
# باستخدام Let's Encrypt (مجاني)
# أو اشتر شهادة من موفر معتمد
```

---

## نشر تطبيق الموبايل (Android)

### إنشاء APK للتوزيع

```bash
# APK عادي
dotnet publish -f net10.0-android -c Release /p:AndroidPackageFormat=apk

# AAB لـ Google Play
dotnet publish -f net10.0-android -c Release /p:AndroidPackageFormat=aab
```

### توقيع التطبيق

```bash
# إنشاء keystore (مرة واحدة)
keytool -genkey -v -keystore rubikcare.keystore -alias rubikcare -keyalg RSA -keysize 2048 -validity 10000
```

### رفع لـ Google Play Console

1. أنشئ تطبيق جديد
2. ارفع AAB
3. املأ بيانات المتجر
4. أضف سياسة الخصوصية
5. قدم للمراجعة

---

## نشر تطبيق الموبايل (iOS)

### المتطلبات

- Mac مع Xcode 16+
- Apple Developer Account ($99/سنة)

### خطوات النشر

```bash
# بناء من Windows (تحتاج Mac متصل)
dotnet publish -f net10.0-ios -c Release

# أو من Mac مباشرة
dotnet publish -f net10.0-ios -c Release /p:ArchiveOnBuild=true
```

---

## إعدادات الأمان

### 1. مفاتيح الإنتاج

```bash
# استخدم User Secrets للتطوير
dotnet user-secrets set "Jwt:Key" "dev-key-12345678"

# استخدم Environment Variables للإنتاج
setx Jwt__Key "production-key-32-chars-minimum"
```

### 2. حماية البيانات

```csharp
// في Program.cs
builder.Services.AddDataProtection()
    .PersistKeysToFileSystem(new DirectoryInfo(@"C:\Keys\RubikCare"))
    .SetApplicationName("RubikCare");
```

### 3. CORS (إذا لزم)

```csharp
// فقط إذا كان Web و API على domains مختلفة
builder.Services.AddCors(options =>
{
    options.AddPolicy("AllowWebApp", builder =>
    {
        builder.WithOrigins("https://app.rubikcare.com")
               .AllowAnyHeader()
               .AllowAnyMethod();
    });
});
```

---

## النسخ الاحتياطي

### قاعدة البيانات

```sql
-- نسخ احتياطي كامل
BACKUP DATABASE RubikCare TO DISK = 'C:\Backups\RubikCare_Full.bak' WITH INIT;

-- نسخ احتياطي تفاضلي
BACKUP DATABASE RubikCare TO DISK = 'C:\Backups\RubikCare_Diff.bak' WITH DIFFERENTIAL;
```

### جدولة تلقائية

```sql
-- إنشاء Job في SQL Server Agent
-- النسخ الاحتياطي اليومي الساعة 2 صباحاً
EXEC sp_add_job @job_name = 'RubikCare_Daily_Backup';
EXEC sp_add_jobstep @job_name = 'RubikCare_Daily_Backup',
    @step_name = 'Backup',
    @command = 'BACKUP DATABASE RubikCare TO DISK = ''C:\Backups\RubikCare.bak'' WITH INIT';
EXEC sp_add_schedule @job_name = 'RubikCare_Daily_Backup',
    @name = 'Daily',
    @freq_type = 4,
    @freq_interval = 1,
    @active_start_time = 20000;
```

---

## متغيرات البيئة

| المتغير | الوصف | مثال |
|---------|-------|-------|
| `ASPNETCORE_ENVIRONMENT` | البيئة | `Production` |
| `ConnectionStrings__DefaultConnection` | اتصال قاعدة البيانات | `Server=...` |
| `Jwt__Key` | مفتاح JWT السري | `key-32-chars...` |
| `Jwt__Issuer` | مصدر التوكن | `https://rubikcare.com` |
| `Google__ClientId` | Google OAuth Client ID | `xxx.apps.google...` |
| `Google__ClientSecret` | Google OAuth Secret | `GOCSPX-...` |

---

## فحص الصحة (Health Checks)

### إضافة Health Check

```csharp
// في Program.cs
builder.Services.AddHealthChecks()
    .AddDbContextCheck<BusinessDbContext>();

app.MapHealthChecks("/health");
```

### مراقبة

```bash
# فحص أن API يعمل
curl https://api.rubikcare.com/health

# استجابة متوقعة:
# Healthy
```

---

## استكشاف الأخطاء في الإنتاج

### 1. التطبيق لا يستجيب

```bash
# تحقق من Application Pool في IIS
# تأكد أنه Started وليس Stopped

# تحقق من Event Viewer
eventvwr.msc → Windows Logs → Application
```

### 2. خطأ في قاعدة البيانات

```bash
# تحقق من Connection String
# تأكد من أن SQL Server يعمل
# تأكد من صلاحيات المستخدم
```

### 3. أخطاء 500

```bash
# فعّل logging التفصيلي
"Logging": {
    "LogLevel": {
        "Default": "Information",
        "Microsoft.AspNetCore": "Warning",
        "RubikCare": "Debug"
    }
}
```

---

## 🔗 روابط ذات صلة

- [01 - Program.cs والتسجيلات الأساسية](01-program-cs-foundation.md)
- [09 - دليل API](09-api-guide.md)
- [10 - دليل تطوير MAUI](10-maui-development-guide.md)
```

---

## الملف الثاني: `appendix-a-glossary.md`

**المسار:** `docs/appendix-a-glossary.md`

```markdown
# الملحق أ - مسرد المصطلحات (Glossary)

**آخر تحديث: 17 مايو 2026**

---

## الأساسيات

| المصطلح | الشرح |
|---------|-------|
| **Clean Architecture** | بنية نظيفة تفصل التطبيق إلى طبقات (Domain, Application, Infrastructure, Presentation) |
| **Domain Layer** | طبقة الكيانات الأساسية - لا تعتمد على أي طبقة أخرى |
| **Application Layer** | طبقة منطق الأعمال (Use Cases) - تعتمد فقط على Domain |
| **Infrastructure Layer** | طبقة الوصول للبيانات - تعتمد على Application و Domain |
| **Presentation Layer** | طبقة واجهة المستخدم (Api.Web, Web, Mobile) |
| **GenericService\<T\>** | خدمة عامة لعمليات CRUD على أي كيان |
| **DbContextFactory** | مصنع لإنشاء سياقات قاعدة البيانات - يمنع مشاكل السياق المتعدد |

---

## الهوية والمصادقة

| المصطلح | الشرح |
|---------|-------|
| **Identity** | نظام Microsoft.Identity لإدارة المستخدمين والأدوار |
| **ApplicationUser** | كيان المستخدم الممتد من IdentityUser |
| **UserProfile** | كيان البيانات الموسعة للمستخدم |
| **UserRoleService** | خدمة للتعامل مع جدول AspNetUserRoles الموسع |
| **JWT** | JSON Web Token - للمصادقة في API |
| **OrgMembership** | عضوية مستخدم في منظمة (عيادة، شركة، صيدلية) |

---

## القوائم الديناميكية

| المصطلح | الشرح |
|---------|-------|
| **SystemMenu** | قائمة رئيسية (مثل USER_BASE, ADMIN_MENU) |
| **MenuItem** | عنصر داخل قائمة رئيسية |
| **MenuAssignment** | تخصيص قائمة لمستخدم/دور/منظمة |
| **DynamicMenuService** | خدمة جلب القوائم المناسبة للمستخدم |

---

## نظام PSP

| المصطلح | الشرح |
|---------|-------|
| **PSP** | Patient Support Program - برنامج دعم المرضى |
| **PSPProgram** | برنامج الدعم (الاسم، الشركة المالكة) |
| **PSPProgramMedication** | دواء داخل برنامج دعم |
| **PSPParticipation** | مشاركة عيادة/طبيب في برنامج |
| **PSPInvitation** | دعوة من طبيب لمريض |
| **PSPPatient** | مريض مسجل في برنامج |
| **PSPeRX** | وصفة إلكترونية |
| **PSPDispensationPlan** | خطة صرف الأدوية |
| **PSPDispensation** | رمز صرف يستخدمه المريض في الصيدلية |
| **InvitationToken** | رمز دعوة فريد (مثل INV-5CWEFHEJ) |
| **TokenCode** | رمز صرف (مثل RC-20260323-1234) |

---

## الموبايل

| المصطلح | الشرح |
|---------|-------|
| **MAUI** | .NET Multi-platform App UI |
| **XAML** | لغة تعريف واجهات MAUI |
| **BlazorWebView** | مكون لدمج Blazor داخل تطبيق MAUI |
| **Shell** | نظام التنقل في MAUI |
| **SecureStorage** | تخزين آمن للبيانات الحساسة |
| **MVVM** | Model-View-ViewModel - نمط معماري |

---

## الويب

| المصطلح | الشرح |
|---------|-------|
| **Blazor Server** | تطبيق ويب تفاعلي يعمل على الخادم |
| **Razor** | لغة قوالب .NET |
| **RCL** | Razor Class Library - مكتبة مكونات مشتركة |
| **SignalR** | تقنية اتصال حي بين الخادم والمتصفح |

---

## النشر

| المصطلح | الشرح |
|---------|-------|
| **IIS** | Internet Information Services - خادم ويب |
| **APK** | Android Package Kit - ملف تثبيت Android |
| **AAB** | Android App Bundle - صيغة Google Play |
| **SSL** | Secure Sockets Layer - تشفير الاتصال |

---

## عام

| المصطلح | الشرح |
|---------|-------|
| **CRUD** | Create, Read, Update, Delete |
| **DTO** | Data Transfer Object |
| **DI** | Dependency Injection |
| **EF Core** | Entity Framework Core - ORM |
| **Middleware** | وسيط في pipeline الطلبات |
| **RWE** | Real-World Evidence - أدلة العالم الحقيقي |
```

---
# 📋 دليل النشر والتشغيل على IIS — RubikCare PWA

**آخر تحديث:** أغسطس 2026
**البيئة:** Windows Server + IIS + Blazor WebAssembly PWA

---

## 🎯 الإجابة المباشرة على سؤالك

### هل أحذف `web.config` مع باقي الملفات عند النشر؟

**لا! ملف `web.config` يجب أن يبقى دائماً.**

| الملف | الإجراء عند النشر | السبب |
|-------|-------------------|-------|
| `web.config` | ✅ **احتفظ به** | يحتوي إعدادات IIS (URL Rewrite + MIME types) |
| `wwwroot/*` | 🔄 **امسح واستبدل** | ملفات التطبيق تتغير مع كل نشر |
| `_framework/*` | 🔄 **امسح واستبدل** | ملفات Blazor تتغير مع كل نشر (أسماء مُجزّأة) |

### الحل الصحيح: اجعل `web.config` جزءاً من المشروع

ضع `web.config` داخل مجلد المشروع `RubikCare.PWA/` وأضفه إلى `.csproj`:

```xml
<ItemGroup>
  <Content Include="web.config">
    <CopyToPublishDirectory>PreserveNewest</CopyToPublishDirectory>
  </Content>
</ItemGroup>
```

بهذه الطريقة، `dotnet publish` سينسخه تلقائياً مع كل نشر، ولن تحتاج للتدخل اليدوي أبداً.

---

## 📁 محتويات `web.config` الصحيحة (احفظ هذه النسخة)

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

### ⚠️ نقاط حرجة في `web.config`

| النقطة | القيمة الصحيحة | إذا كانت خاطئة |
|--------|---------------|-----------------|
| `stopProcessing="true"` في `Serve subdir` | ✅ يجب أن تكون موجودة | تضارب مع `SPA fallback` |
| MIME types لـ `.wasm`, `.dat`, `.webmanifest` | ✅ يجب أن تكون موجودة | 404 على ملفات Blazor |
| `rule name="Serve subdir"` | ✅ يجب أن يكون الأول | ترتيب القواعد مهم |

---

## 🏗️ إعداد السيرفر لأول مرة (مرة واحدة فقط)

### 1. تثبيت URL Rewrite Module

```powershell
# التحقق من وجود الوحدة
Get-WebGlobalModule | Where-Object { $_.Name -like "*Rewrite*" }
```

إذا لم تظهر، ثبّتها من:
```
https://www.iis.net/downloads/microsoft/url-rewrite
```

ثم تحقق:
```powershell
Test-Path "C:\Windows\System32\inetsrv\rewrite.dll"
```

### 2. إنشاء الموقع في IIS

```powershell
# إنشاء App Pool
C:\Windows\System32\inetsrv\appcmd add apppool /name:"PU_RubicCareStage" /managedRuntimeVersion:"" /managedPipelineMode:"Integrated"

# إنشاء الموقع
C:\Windows\System32\inetsrv\appcmd add site /name:"PU_RubicCareStage" /physicalPath:"C:\WebSite\PU_RubicCareStage" /bindings:https/*:443:stagepu.rubikcare.com

# ربط الموقع بالـ App Pool
C:\Windows\System32\inetsrv\appcmd set app "PU_RubicCareStage/" /applicationPool:"PU_RubicCareStage"
```

### 3. تثبيت شهادة SSL

عبر **IIS Manager**:
1. افتح **Server Certificates**
2. استورد شهادة `stagepu.rubikcare.com`
3. اربطها بالموقع في **Bindings**

### 4. تعيين المسار الفيزيائي الصحيح

```powershell
# ⚠️ مهم جداً: المسار الفيزيائي يجب أن يكون الجذر وليس wwwroot
C:\Windows\System32\inetsrv\appcmd set vdir "PU_RubicCareStage/" /physicalPath:"C:\WebSite\PU_RubicCareStage"
```

**تحذير:** إذا كان المسار الفيزيائي `C:\WebSite\PU_RubicCareStage\wwwroot`، ستحدث مشكلة تضارب مع قاعدة `Serve subdir` في `web.config`.

---

## 🚀 خطوات النشر الصحيحة (كل مرة)

### السكربت الكامل `deploy-stage.ps1`

احفظ هذا السكربت في جذر المشروع:

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
Write-Host "================================================" -ForegroundColor Cyan

# الخطوة 1: إيقاف الـ App Pool (لمنع قفل الملفات)
Write-Host "`n⏸️  الخطوة 1: إيقاف الـ App Pool..." -ForegroundColor Yellow
C:\Windows\System32\inetsrv\appcmd stop apppool $AppPoolName
Start-Sleep -Seconds 2

# الخطوة 2: مسح محتوى المجلد (باستثناء web.config)
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

# الخطوة 5: التأكد من وجود الملفات الحرجة
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

# التحقق من ملفات _framework
$wasmFiles = Get-ChildItem "$IISPath\wwwroot\_framework" -Filter "*.wasm" -ErrorAction SilentlyContinue
$datFiles = Get-ChildItem "$IISPath\wwwroot\_framework" -Filter "icudt_*.dat" -ErrorAction SilentlyContinue

if ($wasmFiles.Count -gt 0) {
    Write-Host "  ✅ ملفات .wasm: $($wasmFiles.Count)" -ForegroundColor Green
} else {
    Write-Host "  ❌ لا توجد ملفات .wasm!" -ForegroundColor Red
    $allFound = $false
}

if ($datFiles.Count -gt 0) {
    Write-Host "  ✅ ملفات ICU (.dat): $($datFiles.Count)" -ForegroundColor Green
} else {
    Write-Host "  ❌ لا توجد ملفات ICU!" -ForegroundColor Red
    $allFound = $false
}

# الخطوة 6: تشغيل الـ App Pool
Write-Host "`n▶️  الخطوة 6: تشغيل الـ App Pool..." -ForegroundColor Yellow
C:\Windows\System32\inetsrv\appcmd start apppool $AppPoolName

# الخطوة 7: اختبار سريع
Write-Host "`n🧪 الخطوة 7: اختبار الروابط..." -ForegroundColor Yellow
Start-Sleep -Seconds 3

$tests = @(
    @{ Name = "الصفحة الرئيسية"; Url = "$SiteUrl/" },
    @{ Name = "blazor.webassembly.js"; Url = "$SiteUrl/_framework/blazor.webassembly.js" },
    @{ Name = "manifest.webmanifest"; Url = "$SiteUrl/manifest.webmanifest" },
    @{ Name = "service-worker.js"; Url = "$SiteUrl/service-worker.js" }
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

---

## 🔍 الفحوصات التشخيصية عند حدوث مشكلة

### الفحص 1: حالة الموقع والـ App Pool

```powershell
C:\Windows\System32\inetsrv\appcmd list site "PU_RubicCareStage"
C:\Windows\System32\inetsrv\appcmd list apppool "PU_RubicCareStage"
```

### الفحص 2: المسار الفيزيائي

```powershell
C:\Windows\System32\inetsrv\appcmd list vdir "PU_RubicCareStage/" /text:physicalPath
```

**النتيجة الصحيحة:** `C:\WebSite\PU_RubicCareStage` (بدون `\wwwroot`)

### الفحص 3: ملفات Blazor الحرجة

```powershell
# ملفات .wasm
Get-ChildItem "C:\WebSite\PU_RubicCareStage\wwwroot\_framework" -Filter "*.wasm" | Select-Object Name, Length

# ملفات ICU
Get-ChildItem "C:\WebSite\PU_RubicCareStage\wwwroot\_framework" -Filter "icudt_*.dat" | Select-Object Name, Length

# ملفات JavaScript
Get-ChildItem "C:\WebSite\PU_RubicCareStage\wwwroot\_framework" -Filter "*.js" | Where-Object { $_.Name -notlike "*.br" -and $_.Name -notlike "*.gz" } | Select-Object Name
```

### الفحص 4: اختبار الروابط من السيرفر

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

### الفحص 5: محتوى `web.config`

```powershell
Get-Content "C:\WebSite\PU_RubicCareStage\web.config" | Select-String "stopProcessing|mimeMap|rule name"
```

---

## 🐛 الأخطاء الشائعة وحلولها

### ❌ الخطأ 1: شاشة زرقاء + `Unexpected token '<'`

**العرض في Console:**
```
blazor.webassembly.js:1 Uncaught SyntaxError: Unexpected token '<'
```

**السبب:** المتصفح يتلقى HTML بدلاً من JavaScript (عادة `index.html` عبر SPA fallback)

**الحل:**
```powershell
# 1. تأكد من وجود blazor.webassembly.js
Test-Path "C:\WebSite\PU_RubicCareStage\wwwroot\_framework\blazor.webassembly.js"

# 2. إذا كان غير موجود، أعد النشر
# 3. تأكد من أن web.config يحتوي على stopProcessing="true" في قاعدة Serve subdir
```

---

### ❌ الخطأ 2: `SRI integrity checks failed` + ملفات `.dat` أو `.wasm` ترجع 404

**العرض في Console:**
```
Failed to find a valid digest in the 'integrity' attribute
Failed to fetch. SRI's integrity checks failed.
```

**السبب:** ملفات الـ framework غير موجودة أو المسار الفيزيائي خاطئ

**الحل:**
```powershell
# 1. تحقق من المسار الفيزيائي
C:\Windows\System32\inetsrv\appcmd list vdir "PU_RubicCareStage/" /text:physicalPath
# يجب أن يكون: C:\WebSite\PU_RubicCareStage (بدون \wwwroot)

# 2. تحقق من وجود الملفات
Get-ChildItem "C:\WebSite\PU_RubicCareStage\wwwroot\_framework" -Filter "icudt_*.dat"

# 3. صحح المسار إذا كان خاطئاً
C:\Windows\System32\inetsrv\appcmd set vdir "PU_RubicCareStage/" /physicalPath:"C:\WebSite\PU_RubicCareStage"

# 4. أعد تدوير الـ App Pool
C:\Windows\System32\inetsrv\appcmd recycle apppool "PU_RubicCareStage"
```

---

### ❌ الخطأ 3: `manifest.webmanifest` يرجع 404

**السبب:** IIS لا يعرف MIME type للامتداد `.webmanifest`

**الحل:** أضف في `web.config`:
```xml
<remove fileExtension=".webmanifest" />
<mimeMap fileExtension=".webmanifest" mimeType="application/manifest+json" />
```

---

### ❌ الخطأ 4: الموقع لا يفتح أبداً (خطأ 500)

**السبب:** وحدة URL Rewrite غير مثبتة

**الحل:**
```powershell
# تحقق
Get-WebGlobalModule | Where-Object { $_.Name -like "*Rewrite*" }

# إذا لم تظهر، ثبّت من:
# https://www.iis.net/downloads/microsoft/url-rewrite
```

---

### ❌ الخطأ 5: التطبيق يعمل في المتصفح العادي لكن ليس في InPrivate

**السبب:** Service Worker مخزّن في المتصفح العادي

**الحل:**
1. افتح نافذة InPrivate دائماً للاختبار: `Ctrl + Shift + N`
2. أو امسح Service Worker من المتصفح العادي:
   - `F12` → **Application** → **Service Workers** → **Unregister**
   - `F12` → **Application** → **Storage** → **Clear site data**

---

### ❌ الخطأ 6: صورة البروفايل لا تظهر + خطأ CORS

**العرض:**
```
Access to fetch at 'https://uat.rubikcare.com/uploads/...' from origin 'https://stagepu.rubikcare.com' 
has been blocked by CORS policy
```

**الحل:** أضف CORS policy في API UAT (`Program.cs`):
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

// بعد builder.Build()
app.UseCors("AllowPWA");
```

---

## ⚠️ المحاذير الحرجة

### 🔴 ممنوعات مطلقة

| # | المحظور | السبب | البديل |
|---|---------|-------|--------|
| 1 | ❌ حذف `web.config` عند النشر | يحتوي إعدادات IIS الضرورية | احتفظ به دائماً |
| 2 | ❌ نسخ الملفات فوق الملفات القديمة بدون مسح | تضارب الأسماء المُجزّأة (hash) | امسح المجلد أولاً ثم انسخ |
| 3 | ❌ تعديل `web.config` يدوياً بعد النشر | قد تفقد التعديلات في النشر القادم | اجعل `web.config` جزءاً من المشروع |
| 4 | ❌ تغيير المسار الفيزيائي إلى `wwwroot` | تضارب مع قاعدة `Serve subdir` | المسار الفيزيائي = الجذر دائماً |
| 5 | ❌ اختبار في المتصفح العادي بعد نشر جديد | Service Worker يخدم ملفات قديمة | اختبر في InPrivate دائماً |
| 6 | ❌ نشر بدون إيقاف الـ App Pool | قفل الملفات وفشل النسخ | أوقف الـ App Pool أولاً |

### 🟡 إجراءات تتطلب حذراً

| الإجراء | الطريقة الصحيحة | الطريقة الخاطئة |
|---------|-----------------|-----------------|
| إعادة النشر | إيقاف → مسح → نشر → نسخ → تشغيل | نسخ فوق الملفات مباشرة |
| تعديل `web.config` | عدّل النسخة في المشروع ثم انشر | عدّل النسخة على السيرفر يدوياً |
| تحديث API UAT | انشر في مجلد منفصل (`RubikCareUat`) | لا تنسخ ملفات API إلى مجلد PWA |

### 🟢 أفضل الممارسات

| الممارسة | السبب |
|----------|-------|
| احتفظ بنسخة احتياطية من `web.config` | للرجوع إليها عند الحاجة |
| وثّق أي تغيير في `Program.cs` أو `MainLayout.razor` | لتجنب فقدان التعديلات |
| اختبر في InPrivate بعد كل نشر | لتجنب كاش Service Worker |
| استخدم سكربت نشر موحد | لتجنب الأخطاء اليدوية |

---

## 📊 قائمة المراجعة النهائية (CHECKLIST)

### قبل النشر
- [ ] هل تم حفظ جميع التعديلات في الكود؟
- [ ] هل تم اختبار التطبيق محلياً (`Ctrl+F5`)؟
- [ ] هل `web.config` موجود في مجلد المشروع؟
- [ ] هل `CopyBlazorAssets` Target موجود في `.csproj`؟

### أثناء النشر
- [ ] هل تم إيقاف الـ App Pool؟
- [ ] هل تم مسح المجلد القديم (باستثناء `web.config`)؟
- [ ] هل نجح `dotnet publish` بدون أخطاء؟
- [ ] هل تم نسخ الملفات إلى IIS؟

### بعد النشر
- [ ] هل تم تشغيل الـ App Pool؟
- [ ] هل جميع الفحوصات تظهر ✅؟
- [ ] هل اختبرت من نافذة InPrivate؟
- [ ] هل Console خالي من الأخطاء الحمراء؟
- [ ] هل تسجيل الدخول يعمل؟
- [ ] هل الوظائف الرئيسية تعمل؟

---

## 🎯 ملخص سريع للنشر

```
1. أوقف الـ App Pool
2. امسح المجلد (احتفظ بـ web.config)
3. dotnet publish
4. انسخ الملفات
5. شغّل الـ App Pool
6. اختبر في InPrivate
```

**هذا هو!** 🚀

---

**هل تريد أن أساعدك في:**
1. إنشاء ملف `web.config` داخل المشروع وتفعيله في `.csproj`؟
2. إنشاء سكربت `deploy-stage.ps1` فعلياً على جهازك؟
3. إضافة `CopyBlazorAssets` Target إلى `.csproj` (إذا لم يكن موجوداً بعد)؟
