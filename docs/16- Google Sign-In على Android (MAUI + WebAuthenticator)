# 16 - Google Sign-In وإعدادات الإيميلات الاحترافية على Android (MAUI + WebAuthenticator)

**الإصدار: 2.0** | **آخر تحديث: يونيو 2026**

---

## 📌 نظرة عامة

هذا الدليل يوثّق الإعداد الصحيح الكامل لـ:
1. **تسجيل الدخول عبر Google** في تطبيق RubikCare MAUI على Android، باستخدام `WebAuthenticator` مع PKCE.
2. **إعدادات الإيميلات الاحترافية** مع `Failover` تلقائي بين Zoho (Primary) و Gmail (Backup) باستخدام `Keyed Services`.

---

## 🗂️ الملفات المتعلقة

| الملف | المسار | الدور |
|-------|--------|-------|
| `WebAuthenticatorActivity.cs` | `Mobile/Platforms/Android/` | تسجيل الـ callback للـ Intent |
| `AndroidManifest.xml` | `Mobile/Platforms/Android/` | صلاحيات وإعدادات التطبيق |
| `LoginViewModel.cs` | `Mobile/Features/Auth/ViewModels/` | منطق تسجيل الدخول عبر Google |
| `RubikCare.Mobile.csproj` | `Mobile/` | إعدادات التوقيع (Signing) |
| `ZohoEmailSettings.cs` | `RubikCare.Infrastructure/Settings/` | إعدادات Zoho |
| `GmailEmailSettings.cs` | `RubikCare.Infrastructure/Settings/` | إعدادات Gmail |
| `ZohoEmailService.cs` | `RubikCare.Infrastructure/Services/` | خدمة Zoho SMTP |
| `GmailEmailService.cs` | `RubikCare.Infrastructure/Services/` | خدمة Gmail SMTP |
| `FailoverEmailService.cs` | `RubikCare.Infrastructure/Services/` | التبديل التلقائي بين الخدمات |
| `InfrastructureExtensions.cs` | `RubikCare.Infrastructure/` | تسجيل الخدمات في Dependency Injection |

---

## ✅ 1. Google Sign-In (الإعدادات التي نجحت)

### 1.1 `Platforms/Android/WebAuthenticatorActivity.cs`

```csharp
using Android.App;
using Android.Content.PM;
using Microsoft.Maui.Authentication;

namespace RubikCare.Mobile;

[Activity(
    NoHistory = true,
    LaunchMode = LaunchMode.SingleTop,
    Exported = true)]
[IntentFilter(
    new[] { Android.Content.Intent.ActionView },
    Categories = new[]
    {
        Android.Content.Intent.CategoryDefault,
        Android.Content.Intent.CategoryBrowsable
    },
    DataScheme = "com.googleusercontent.apps.369163319733-8229skagomnn7t0j74uuj3ilj5h1635g")]
public class WebAuthenticatorActivity : WebAuthenticatorCallbackActivity
{
}
```

> ⚠️ **القواعد الثابتة:**
> - `Exported = true` إلزامي — Android 12+ يرفض `Exported = false` مع IntentFilter.
> - لا تضع `<activity>` في `AndroidManifest.xml` إذا كان مسجلاً في الكود — وجود الاثنين معاً يسبب تعارضاً.

---

### 1.2 `Platforms/Android/AndroidManifest.xml`

```xml
<?xml version="1.0" encoding="utf-8"?>
<manifest xmlns:android="http://schemas.android.com/apk/res/android" package="Rubikcare.com">

    <uses-permission android:name="android.permission.INTERNET" />

    <application android:usesCleartextTraffic="true">
        <!-- ❌ لا تضع WebAuthenticatorCallbackActivity هنا -->
    </application>

</manifest>
```

---

### 1.3 إعدادات التوقيع في `RubikCare.Mobile.csproj`

```xml
<PropertyGroup Condition="'$(Configuration)|$(TargetFramework)|$(Platform)'=='Release|net10.0-android|AnyCPU'">
    <AndroidKeyStore>True</AndroidKeyStore>
    <AndroidPackageFormat>apk</AndroidPackageFormat>
    <AndroidSigningKeyStore>E:\Keys\APK Rubikcare keystore\Rubikcare.keystore</AndroidSigningKeyStore>
    <AndroidSigningStorePass>*****</AndroidSigningStorePass>
    <AndroidSigningKeyAlias>rubikcare</AndroidSigningKeyAlias>
    <AndroidSigningKeyPass>*****</AndroidSigningKeyPass>
    <AndroidApkSignerAdditionalArguments></AndroidApkSignerAdditionalArguments>
</PropertyGroup>
```

> 🔑 **Keystore المستخدم:**
> - المسار: `E:\Keys\APK Rubikcare keystore\Rubikcare.keystore`
> - Alias: `rubikcare`
> - SHA-1: `1E:EB:53:0B:C0:49:76:DB:02:F3:28:70:80:93:10:9A:C6:40:D5:88`

---

### 1.4 إعداد Google Cloud Console

- **نوع الـ Client:** Android OAuth Client
- **Package name:** `Rubikcare.com`
- **SHA-1:** `1E:EB:53:0B:C0:49:76:DB:02:F3:28:70:80:93:10:9A:C6:40:D5:88`
- **⚠️ تفعيل "Custom URI Scheme"** يدوياً في إعدادات الـ Android Client.

> بدون تفعيل Custom URI Scheme يظهر خطأ: `Error 400: Custom URI scheme is not enabled for your Android client`

---

### 1.5 الـ Redirect URI في `LoginViewModel.cs`

```csharp
private readonly string _googleClientId =
    "369163319733-8229skagomnn7t0j74uuj3ilj5h1635g.apps.googleusercontent.com";

private readonly string _googleRedirectUri =
    "com.googleusercontent.apps.369163319733-8229skagomnn7t0j74uuj3ilj5h1635g:/oauth2redirect";
```

> الـ `DataScheme` في `WebAuthenticatorActivity.cs` يجب أن يطابق بداية الـ `_googleRedirectUri` بالضبط.

---

## ✅ 2. إعدادات الإيميلات الاحترافية (Zoho + Gmail Failover)

### 2.1 كلاسات الإعدادات (`Settings/`)

#### `ZohoEmailSettings.cs`

```csharp
namespace RubikCare.Infrastructure.Settings;

public class ZohoEmailSettings
{
    public string SmtpServer { get; set; } = "smtp.zoho.com";
    public int SmtpPort { get; set; } = 587;
    public string SenderEmail { get; set; } = string.Empty;
    public string SenderName { get; set; } = string.Empty;
    public string Username { get; set; } = string.Empty;
    public string Password { get; set; } = string.Empty;
    public bool EnableSsl { get; set; } = true;
}
```

#### `GmailEmailSettings.cs`

```csharp
namespace RubikCare.Infrastructure.Settings;

public class GmailEmailSettings
{
    public string SmtpServer { get; set; } = "smtp.gmail.com";
    public int SmtpPort { get; set; } = 587;
    public string SenderEmail { get; set; } = string.Empty;
    public string SenderName { get; set; } = string.Empty;
    public string Username { get; set; } = string.Empty;
    public string Password { get; set; } = string.Empty;
    public bool EnableSsl { get; set; } = true;
}
```

---

### 2.2 خدمات الإيميل (`Services/`)

#### `ZohoEmailService.cs` (مختصر)

```csharp
public class ZohoEmailService : IEmailService
{
    private readonly ZohoEmailSettings _settings;
    private readonly SmtpClient _smtpClient;

    public ZohoEmailService(IOptions<ZohoEmailSettings> options)
    {
        _settings = options.Value;
        _smtpClient = new SmtpClient(_settings.SmtpServer, _settings.SmtpPort)
        {
            Credentials = new NetworkCredential(_settings.Username, _settings.Password),
            EnableSsl = true,
            UseDefaultCredentials = false,
            Timeout = 10000
        };
    }

    public async Task<bool> SendVerificationCodeAsync(string toEmail, string userName, string verificationCode)
    {
        // ... إرسال الإيميل مع Encoding.UTF8
    }
}
```

> **لحل مشكلة اللغة العربية:** أضف `SubjectEncoding = Encoding.UTF8`, `BodyEncoding = Encoding.UTF8`, `HeadersEncoding = Encoding.UTF8`.

#### `FailoverEmailService.cs`

```csharp
public class FailoverEmailService : IEmailService
{
    private readonly IEmailService _primaryService;
    private readonly IEmailService _backupService;

    public FailoverEmailService(
        [FromKeyedServices("Primary")] IEmailService primary,
        [FromKeyedServices("Backup")] IEmailService backup)
    {
        _primaryService = primary;
        _backupService = backup;
    }

    public async Task<bool> SendVerificationCodeAsync(...)
    {
        var result = await _primaryService.SendVerificationCodeAsync(...);
        if (result) return true;
        return await _backupService.SendVerificationCodeAsync(...);
    }
}
```

---

### 2.3 تسجيل الخدمات (`InfrastructureExtensions.cs`)

```csharp
var zohoSettings = configuration.GetSection("Zoho").Get<ZohoEmailSettings>();
var gmailSettings = configuration.GetSection("Gmail").Get<GmailEmailSettings>();
var failoverSettings = configuration.GetSection("EmailFailover").Get<EmailFailoverSettings>();

if (zohoSettings?.Password != null)
{
    services.Configure<ZohoEmailSettings>(configuration.GetSection("Zoho"));
    services.AddKeyedScoped<IEmailService, ZohoEmailService>("Primary");
}

if (gmailSettings?.Password != null)
{
    services.Configure<GmailEmailSettings>(configuration.GetSection("Gmail"));
    services.AddKeyedScoped<IEmailService, GmailEmailService>("Backup");
}

if (failoverSettings?.EnableFailover == true)
{
    services.AddScoped<IEmailService, FailoverEmailService>();
}
```

---

### 2.4 إعدادات `appsettings.json`

```json
{
  "EmailFailover": {
    "EnableFailover": true,
    "PrimaryProvider": "Zoho",
    "BackupProvider": "Gmail"
  },
  "Zoho": {
    "SmtpServer": "smtp.zoho.com",
    "SmtpPort": 587,
    "SenderEmail": "noreply@rubikcare.com",
    "SenderName": "RubikCare",
    "Username": "shadyelzaher@devartlab.com",
    "Password": "",
    "EnableSsl": true
  },
  "Gmail": {
    "SmtpServer": "smtp.gmail.com",
    "SmtpPort": 587,
    "SenderEmail": "shadyelzaher@gmail.com",
    "SenderName": "RubikCare",
    "Username": "shadyelzaher@gmail.com",
    "Password": "",
    "EnableSsl": true
  }
}
```

### 2.5 إعدادات `appsettings.Secrets.json`

```json
{
  "Zoho": {
    "Password": "your_zoho_app_password"
  },
  "Gmail": {
    "Password": "your_gmail_app_password"
  }
}
```

---

## ❌ أسباب الفشل الشائعة وحلولها (الدروس المستفادة)

| الخطأ | السبب | الحل |
|-------|-------|------|
| Google Login يفتح صفحة بحث | `<data scheme>` مفقود من IntentFilter | أضفه في `WebAuthenticatorActivity.cs` |
| Google Login لا يعود للتطبيق | الـ Manifest والكود مسجلَّان معاً | احذف من الـ Manifest، ابقِ الكود فقط |
| `Error 400: invalid_request` | Custom URI Scheme غير مفعّل | فعّله في Google Console |
| APK لا يُثبَّت | `Exported = false` مع IntentFilter | غيّر لـ `Exported = true` |
| Build فاشل `java.exe code 1` | `AndroidApkSignerAdditionalArguments = v1` | اتركه فارغاً |
| الإيميلات تظهر `???` | `Encoding` غير محدد | أضف `Encoding.UTF8` للـ `MailMessage` |
| Zoho يستخدم إعدادات Gmail | خلط بين `EmailSettings` لكل الخدمات | استخدم كلاس Settings منفصل لكل خدمة |
| Zoho لا يعمل رغم صحة الإعدادات | لم تُعد بناء المشروع بعد تغيير `InfrastructureExtensions` | **أعد بناء (Rebuild) الحل بالكامل** |

---

## 🔨 أوامر البناء والتثبيت

### بناء الـ APK (من PowerShell)

```powershell
dotnet restore "C:\RC\Rubikcare.Full.Migration\Mobile\RubikCare.Mobile.csproj"
dotnet build "C:\RC\Rubikcare.Full.Migration\Mobile\RubikCare.Mobile.csproj" -c Release -f net10.0-android
```

### التثبيت عبر ADB

```powershell
& "C:\Program Files (x86)\Android\android-sdk\platform-tools\adb.exe" install -r "C:\RC\Rubikcare.Full.Migration\Mobile\bin\Release\net10.0-android\Rubikcare.com-Signed.apk"
```

### التحقق من الـ Keystore

```cmd
"C:\Program Files\Android\openjdk\jdk-21.0.8\bin\keytool.exe" -list -v -keystore "E:\Keys\APK Rubikcare keystore\Rubikcare.keystore" -storepass Rubik@123
```

---

## 🔗 الوثائق ذات الصلة

- [10 - دليل تطوير MAUI](./10-maui-development-guide.md)
- [12 - دليل النشر (Deployment)](./12-deployment-guide.md)
- [02 - نظام الهوية (Identity System)](./02-identity-system.md)

---

© 2026 RubikCare — للاستخدام الداخلي
