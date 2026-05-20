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

