# 🔐 Google Sign-In - الدليل الشامل لكل المنصات

**الإصدار: 3.0** | **آخر تحديث: أكتوبر 2026**

---

## 📌 نظرة عامة

هذا الدليل يوثّق الإعداد الصحيح الكامل لتسجيل الدخول عبر Google في **مشروع RubikCare بكل منصاته**:

| المنصة | التقنية | الحالة |
|--------|---------|--------|
| **Web (Blazor Server)** | OAuth 2.0 + ASP.NET Core Identity | ✅ يعمل |
| **PWA** | نفس Web | ✅ يعمل |
| **Mobile (MAUI)** | WebAuthenticator + PKCE | ✅ يعمل |

كما يوثّق:
- إعدادات الإيميلات الاحترافية (Zoho + Gmail Failover)
- حلول جميع المشاكل التي واجهناها
- خطوات المراقبة والتوثيق

---

## 🗺️ جدول المحتويات

1. [قسم Web (Blazor Server)](#1-web-blazor-server)
2. [قسم Mobile (MAUI)](#2-mobile-maui)
3. [قسم الإيميلات (Failover)](#3-الإيميلات-failover)
4. [المشاكل والحلول](#4-المشاكل-والحلول)
5. [Logging والمراقبة](#5-logging-والمراقبة)
6. [إعدادات VS Debugging](#6-إعدادات-vs-debugging)
7. [مراجعة دورية](#7-مراجعة-دورية)

---

## 1. Web (Blazor Server)

### 1.1 الملفات المتعلقة

| الملف | المسار | الدور |
|-------|--------|-------|
| `Program.cs` | `Rubikcare.Web/` | تسجيل Google OAuth |
| `IGoogleRegistrationService.cs` | `RubikCare.Application/Interfaces/Auth/` | واجهة الخدمة الموحدة |
| `GoogleRegistrationService.cs` | `RubikCare.Application/Services/Auth/` | تنفيذ الخدمة |
| `ExternalLogin.razor` | `Rubikcare.Web/Components/Account/Pages/` | استقبال Callback من Google |
| `Login.razor` | `Rubikcare.Web/Components/Account/Pages/` | زر Google |

### 1.2 إعدادات `Program.cs`

```csharp
var googleClientId = builder.Configuration["Authentication:Google:ClientId"]
    ?? throw new InvalidOperationException("Google ClientId not found");

var googleClientSecret = builder.Configuration["Authentication:Google:ClientSecret"]
    ?? throw new InvalidOperationException("Google ClientSecret not found");

builder.Services.AddAuthentication()
    .AddGoogle(options =>
    {
        options.ClientId = googleClientId;
        options.ClientSecret = googleClientSecret;
        options.CallbackPath = "/signin-google";
        options.CorrelationCookie.SameSite = SameSiteMode.Unspecified;
        options.CorrelationCookie.SecurePolicy = CookieSecurePolicy.Always;
        options.CorrelationCookie.HttpOnly = true;
    });

builder.Services.ConfigureExternalCookie(options =>
{
    options.Cookie.SameSite = SameSiteMode.Unspecified;
    options.Cookie.SecurePolicy = CookieSecurePolicy.Always;
});

builder.Services.ConfigureApplicationCookie(options =>
{
    options.Cookie.SameSite = SameSiteMode.Unspecified;
    options.Cookie.SecurePolicy = CookieSecurePolicy.Always;
});
```

### 1.3 زر Google في `Login.razor`

```html
<form method="post" action="/Account/PerformExternalLogin" class="login-google-form">
    <AntiforgeryToken />
    <input type="hidden" name="provider" value="Google" />
    <input type="hidden" name="returnUrl" value="@ReturnUrl" />
    <button type="submit" class="login-google-btn">
        <svg class="login-google-icon" viewBox="0 0 24 24">
            <!-- Google SVG paths -->
        </svg>
        <span data-translate="LOGIN.BUTTON.GOOGLE">
            <LocalizedText ResourceKey="LOGIN.BUTTON.GOOGLE" />
        </span>
    </button>
</form>
```

### 1.4 المنطق في `ExternalLogin.razor`

```csharp
protected override async Task OnInitializedAsync()
{
    try
    {
        Input ??= new();

        var info = await SignInManager.GetExternalLoginInfoAsync();
        if (info is null)
        {
            message = T("Error.LoadingInfo");
            return;
        }

        externalLoginInfo = info;
        var email = info.Principal.FindFirstValue(ClaimTypes.Email);

        // 1. محاولة تسجيل دخول مباشر
        var signInResult = await SignInManager.ExternalLoginSignInAsync(
            info.LoginProvider, info.ProviderKey,
            isPersistent: false, bypassTwoFactor: false);

        if (signInResult.Succeeded)
        {
            Logger.LogInformation("✅ Google login succeeded: {Email} at {Time}",
                email, DateTime.Now);
            NavigationManager.NavigateTo(ReturnUrl ?? "/", forceLoad: true);
            return;
        }

        // 2. البريد غير مُفعَّل → تفعيل تلقائي
        if (signInResult.IsNotAllowed)
        {
            var userByLogin = await UserManager.FindByLoginAsync(info.LoginProvider, info.ProviderKey);
            if (userByLogin != null
                && string.Equals(userByLogin.Email, email, StringComparison.OrdinalIgnoreCase))
            {
                if (await GoogleService.AutoConfirmEmailAsync(userByLogin))
                {
                    await SignInManager.SignInAsync(userByLogin, isPersistent: false);
                    Logger.LogInformation("✅ Auto-confirmed and signed in {Email}", email);
                    NavigationManager.NavigateTo(ReturnUrl ?? "/", forceLoad: true);
                    return;
                }
            }
        }

        // 3. ربط Google Login بمستخدم موجود
        var existingUser = await UserManager.FindByEmailAsync(email);
        if (existingUser != null)
        {
            var linked = await GoogleService.LinkGoogleLoginAsync(
                existingUser, info.LoginProvider, info.ProviderKey);

            if (linked)
            {
                await GoogleService.AutoConfirmEmailAsync(existingUser);
                await SignInManager.SignInAsync(existingUser, isPersistent: false);
                Logger.LogInformation("✅ Linked and signed in {Email}", email);
                NavigationManager.NavigateTo(ReturnUrl ?? "/", forceLoad: true);
                return;
            }
        }

        // 4. مستخدم جديد → نموذج إكمال البيانات
        _showRegistrationForm = true;

        var name = info.Principal.FindFirstValue(ClaimTypes.Name) ?? "";
        var nameParts = name.Split(' ');
        Input.FirstName = nameParts.Length > 0 ? nameParts[0] : "";
        Input.LastName = nameParts.Length > 1 ? string.Join(" ", nameParts.Skip(1)) : "";
        Input.Email = email;
    }
    finally
    {
        _isLoading = false;
    }
}
```

### 1.5 الخدمة الموحدة `IGoogleRegistrationService`

```csharp
public interface IGoogleRegistrationService
{
    // ⭐ الحل الجذري: معالجة كل السيناريوهات
    Task<ExternalAuthResult> ProcessGoogleLoginAsync(
        ExternalLoginInfo info,
        string? phoneNumber = null,
        string? firstName = null,
        string? lastName = null);

    Task<bool> AutoConfirmEmailAsync(ApplicationUser user);

    Task<bool> LinkGoogleLoginAsync(
        ApplicationUser user,
        string loginProvider,
        string providerKey);
}
```

### 1.6 الحل الجذري لـ `EmailConfirmed`

في `GoogleRegistrationService.CreateNewUserAsync`:

```csharp
var user = new ApplicationUser
{
    UserName = email,
    Email = email,
    PhoneNumber = phoneNumber,
    EmailConfirmed = true  // ⭐ Google أكد البريد
};

// تأكيد البريد بشكل صريح
var token = await _userManager.GenerateEmailConfirmationTokenAsync(user);
await _userManager.ConfirmEmailAsync(user, token);
```

---

## 2. Mobile (MAUI)

### 2.1 الملفات المتعلقة

| الملف | المسار |
|-------|--------|
| `WebAuthenticatorActivity.cs` | `Mobile/Platforms/Android/` |
| `AndroidManifest.xml` | `Mobile/Platforms/Android/` |
| `LoginViewModel.cs` | `Mobile/Features/Auth/ViewModels/` |
| `RubikCare.Mobile.csproj` | `Mobile/` |

### 2.2 `WebAuthenticatorActivity.cs`

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

### 2.3 إعدادات Google Console للموبايل

- **نوع الـ Client:** Android OAuth Client
- **Package name:** `Rubikcare.com`
- **SHA-1:** `1E:EB:53:0B:C0:49:76:DB:02:F3:28:70:80:93:10:9A:C6:40:D5:88`
- **تفعيل "Custom URI Scheme"** يدوياً

### 2.4 الـ Redirect URI في `LoginViewModel.cs`

```csharp
private readonly string _googleClientId =
    "369163319733-8229skagomnn7t0j74uuj3ilj5h1635g.apps.googleusercontent.com";

private readonly string _googleRedirectUri =
    "com.googleusercontent.apps.369163319733-8229skagomnn7t0j74uuj3ilj5h1635g:/oauth2redirect";
```

---

## 3. الإيميلات (Failover)

### 3.1 كلاسات الإعدادات

```csharp
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

### 3.2 `FailoverEmailService`

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

### 3.3 `appsettings.json`

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
    "EnableSsl": true
  },
  "Gmail": {
    "SmtpServer": "smtp.gmail.com",
    "SmtpPort": 587,
    "SenderEmail": "shadyelzaher@gmail.com",
    "SenderName": "RubikCare",
    "Username": "shadyelzaher@gmail.com",
    "EnableSsl": true
  }
}
```

### 3.4 `appsettings.Secrets.json`

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

## 4. المشاكل والحلول

### 4.1 مشاكل Google Login

| # | المشكلة | السبب | الحل |
|---|---------|-------|------|
| 1 | زر Google لا يستجيب | `<a href>` داخل `EditForm` | استخدم `<form method="post">` خارج `EditForm` |
| 2 | `AntiforgeryValidationException` | لا يوجد Anti-forgery token | أضف `<AntiforgeryToken />` |
| 3 | `CryptographicException` | Data Protection keys غير ثابتة | `PersistKeysToFileSystem` |
| 4 | `TypeError: Failed to execute 'query' on 'Permissions': Illegal invocation` | خطأ في Google GIS Library | تعطيل VS JavaScript Debugging |
| 5 | تسجيل دخول يفتح صفحة بحث | `<data scheme>` مفقود | أضف `DataScheme` في `WebAuthenticatorActivity.cs` |
| 6 | Google Login لا يعود للتطبيق | Manifest + Code مسجلان | احذف من Manifest، ابقِ الكود |
| 7 | `Error 400: invalid_request` | Custom URI Scheme غير مفعّل | فعّله في Google Console |
| 8 | APK لا يُثبَّت | `Exported = false` | غيّر لـ `Exported = true` |

### 4.2 مشاكل `EmailConfirmed = 0`

**الأعراض:**
- المستخدم موجود لكن `ExternalLoginSignInAsync` يفشل
- النظام يعرض نموذج "إنشاء حساب جديد"
- يفشل لأن البريد مكرر

**السبب:**
- `RequireConfirmedAccount = true` + `EmailConfirmed = 0`

**الحل (للمستخدمين الحاليين):**

```sql
-- إصلاح الحسابات المتأثرة
UPDATE AspNetUsers
SET EmailConfirmed = 1
WHERE Id IN (
    SELECT u.Id
    FROM AspNetUsers u
    INNER JOIN AspNetUserLogins l ON l.UserId = u.Id
    WHERE l.LoginProvider = 'Google'
      AND u.EmailConfirmed = 0
);
```

**الحل (للمستخدمين الجدد):**
- الكود الجديد يُفعِّل البريد تلقائياً في `CreateNewGoogleUserAsync`

### 4.3 مشاكل VS Debugger

| المشكلة | السبب | الحل |
|---------|-------|------|
| VS يتوقف عند خطأ Google | JavaScript Debugging مفعّل | `Tools → Options → Debugging → General` → إلغاء `Enable JavaScript debugging for ASP.NET` |
| SignalR Circuit ينقطع | توقف VS | نفس الحل |
| Google Login يفشل | Circuit مقطوع | نفس الحل |

---

## 5. Logging والمراقبة

### 5.1 Logging في `ExternalLogin.razor`

```csharp
// بعد نجاح تسجيل الدخول
Logger.LogInformation("✅ Google login succeeded: {Email} at {Time}",
    email, DateTime.Now);

// بعد التفعيل التلقائي
Logger.LogInformation("✅ Auto-confirmed and signed in {Email}", email);

// بعد الربط
Logger.LogInformation("✅ Linked and signed in {Email}", email);

// عند إنشاء حساب جديد
Logger.LogInformation("✅ New Google user created: {Email}", email);
```

### 5.2 مراقبة دورية

**استعلام للتحقق الأسبوعي:**

```sql
-- عدد حسابات Google غير المُفعَّلة
SELECT COUNT(*) AS UnconfirmedGoogleUsers
FROM AspNetUsers u
INNER JOIN AspNetUserLogins l ON l.UserId = u.Id
WHERE l.LoginProvider = 'Google'
  AND u.EmailConfirmed = 0;
```

**النتيجة المتوقعة:** **0** — إذا كان الكود يعمل بشكل صحيح.

**في حالة النتيجة > 0:**

```sql
-- إصلاح فوري
UPDATE AspNetUsers
SET EmailConfirmed = 1
WHERE Id IN (
    SELECT u.Id
    FROM AspNetUsers u
    INNER JOIN AspNetUserLogins l ON l.UserId = u.Id
    WHERE l.LoginProvider = 'Google'
      AND u.EmailConfirmed = 0
);
```

### 5.3 إحصائيات Google Login

```sql
-- عدد تسجيلات الدخول عبر Google (آخر 30 يوماً)
SELECT
    COUNT(*) AS TotalGoogleLogins,
    COUNT(DISTINCT UserId) AS UniqueUsers
FROM AspNetUserLogins
WHERE LoginProvider = 'Google'
  AND UserId IN (
    SELECT Id FROM AspNetUsers
    WHERE LastActivityDate >= DATEADD(day, -30, GETDATE())
  );
```

---

## 6. إعدادات VS Debugging

### 6.1 تعطيل JavaScript Debugging

**الخطوات:**

1. `Tools` → `Options`
2. `Debugging` → `General`
3. **إلغاء:** `Enable JavaScript debugging for ASP.NET (Chrome, Edge and IE)`
4. **OK**

**لماذا؟**
- VS يتوقف عند أخطاء JavaScript من Google
- التوقف يقطع SignalR Circuit
- Circuit المقطوع يُفشل Google Login

### 6.2 تعطيل Exception Settings (احتياطي)

**الخطوات:**

1. `Debug` → `Windows` → `Exception Settings` (`Ctrl+Alt+E`)
2. ابحث عن `JavaScript Exceptions`
3. **أزل `Thrown`** من `TypeError` (أو من `JavaScript Exceptions` بالكامل)

### 6.3 تفعيل Just My Code

**الخطوات:**

1. `Tools` → `Options` → `Debugging` → `General`
2. **فعّل:** `Enable Just My Code`
3. **أزل:** `Warn if no user code on launch`

**النتيجة:**
- VS يتجاهل الأكواد الخارجية (Google)
- يتوقف فقط عند كود مشروعك

---

## 7. مراجعة دورية

### 7.1 المراجعة الأسبوعية

| # | الفحص | الأداة |
|---|-------|--------|
| 1 | `UnconfirmedGoogleUsers` = 0؟ | SQL |
| 2 | Google Login يعمل في المتصفح العادي؟ | اختبار يدوي |
| 3 | Google Login يعمل في التصفح الخفي؟ | اختبار يدوي |
| 4 | Google Login يعمل في الموبايل؟ | اختبار يدوي |

### 7.2 المراجعة الشهرية

| # | الفحص |
|---|-------|
| 1 | تحديثات Google OAuth API |
| 2 | تحديثات ASP.NET Core Identity |
| 3 | إحصائيات Google Login (نجاح/فشل) |
| 4 | مراجعة `Authorized JavaScript Origins` في Google Console |

### 7.3 المراجعة الربع سنوية

| # | الفحص |
|---|-------|
| 1 | مراجعة شاملة لنظام المصادقة |
| 2 | تحديث مكتبات NuGet |
| 3 | مراجعة الصلاحيات والأمان |
| 4 | اختبار Failover للإيميلات |

---

## 🔗 الوثائق ذات الصلة

- [Project-Dashboard](./Project-Dashboard.md) — حالة المشروع
- [Identity-Guide](./identity-Guide.md) — نظام الهوية
- [Localization-Guide](./Localization-Guide.md) — نظام الترجمة
- [MAUI-Development-Guide](./10-maui-development-guide.md) — تطوير MAUI
- [Deployment-Guide](./12-deployment-guide.md) — النشر

---

## 📝 ملاحظات مهمة

### قاعدة ذهبية

**لا تعدل `IGoogleRegistrationService` بدون فهم كامل للسيناريوهات الأربعة:**

1. **تسجيل دخول مباشر** (EmailConfirmed = 1)
2. **تفعيل تلقائي** (EmailConfirmed = 0)
3. **ربط Google Login** (مستخدم موجود بدون Google)
4. **إنشاء حساب جديد** (يحتاج رقم هاتف)

### قاعدة ثانية

**بعد أي تعديل على Google Login:**

1. اختبر في المتصفح العادي
2. اختبر في التصفح الخفي
3. اختبر في الموبايل
4. تحقق من `UnconfirmedGoogleUsers` = 0

---

© 2026 RubikCare — للاستخدام الداخلي
```
