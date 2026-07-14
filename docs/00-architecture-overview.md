# 00 - الهيكل المعماري للمشاريع الثمانية (Clean Architecture)

**آخر تحديث:** 14 يوليو 2026  
**الإصدار:** 1.2

---

## نظرة عامة

يتبع مشروع RubikCare نمط Clean Architecture مقسماً إلى 8 مشاريع منفصلة، كل منها له مسؤولية محددة وعلاقات واضحة مع المشاريع الأخرى.

---

## هيكل المشاريع والعلاقات

```
┌─────────────────────────────────────────────────────────────────┐
│                      RubikCare.Api.Web                          │
│              (Presentation Layer - API)                         │
│         المسؤول: استقبال طلبات HTTP                            │
│         يعتمد على: Application, Infrastructure                 │
└─────────────────────────────────────────────────────────────────┘
                              │
                              ▼
┌─────────────────────────────────────────────────────────────────┐
│                    RubikCare.Application                        │
│              (Application Layer - Use Cases)                    │
│         المسؤول: منطق الأعمال والتنسيق                         │
│         يعتمد على: Domain فقط                                  │
│         يحتوي: Services, Interfaces, DTOs                      │
└─────────────────────────────────────────────────────────────────┘
                              │
                              ▼
┌─────────────────────────────────────────────────────────────────┐
│                  RubikCare.Infrastructure                       │
│              (Infrastructure Layer - Data Access)               │
│         المسؤول: التواصل مع قاعدة البيانات                     │
│         يعتمد على: Application, Domain                         │
│         يحتوي: DbContext, Repositories, Migrations             │
└─────────────────────────────────────────────────────────────────┘
                              │
                              ▼
┌─────────────────────────────────────────────────────────────────┐
│                      RubikCare.Domain                           │
│                    (Domain Layer - Core)                        │
│         المسؤول: الكيانات الأساسية وقواعد العمل                │
│         لا يعتمد على أي مشروع آخر                              │
│         يحتوي: Entities, Enums, Interfaces                     │
└─────────────────────────────────────────────────────────────────┘

┌─────────────────────────────────────────────────────────────────┐
│                      RubikCare.Mobile                           │
│                    (MAUI Application)                           │
│         المسؤول: تطبيق الموبايل                                │
│         يتصل بـ: Api.Web فقط (عبر HTTP)                        │
│         لا يعتمد على أي مشروع آخر                              │
└─────────────────────────────────────────────────────────────────┘

┌─────────────────────────────────────────────────────────────────┐
│                        RubikCare.Web                            │
│                (Blazor Server Application)                      │
│         المسؤول: تطبيق الويب                                   │
│         يعتمد على: Api.Web (ضمنياً)                            │
└─────────────────────────────────────────────────────────────────┘

┌─────────────────────────────────────────────────────────────────┐
│                    RubikCare.Shared.UI                          │
│              (Razor Class Library)                              │
│         المسؤول: مكونات UI مشتركة بين Web و Mobile             │
│         يحتوي: مكونات Razor قابلة لإعادة الاستخدام             │
└─────────────────────────────────────────────────────────────────┘

┌─────────────────────────────────────────────────────────────────┐
│                      RubikCare.Tests                            │
│                    (Test Project)                               │
│         المسؤول: اختبارات الوحدة والتكامل                      │
└─────────────────────────────────────────────────────────────────┘
```

---

## قواعد إلزامية للعلاقات بين المشاريع

| # | القاعدة | التفاصيل |
|---|---------|----------|
| 1 | **Domain لا يعتمد على أحد** | أنقى طبقة - تحتوي فقط على Entities, Enums, Value Objects, Interfaces |
| 2 | **Application يعتمد فقط على Domain** | يحتوي على Use Cases, Services, DTOs - لا يعرف شيئاً عن قاعدة البيانات أو بيئة التشغيل |
| 3 | **Infrastructure يعتمد على Application و Domain** | ينفذ الواجهات المعرفة في Application - يحتوي على DbContext, Repositories, Migrations |
| 4 | **Api.Web يعتمد على Application و Infrastructure** | نقطة الدخول الوحيدة للبيانات - يحتوي على Controllers, Middleware, Program.cs |
| 5 | **Web (Blazor) يعتمد على Api.Web** | يتواصل مع الخادم عبر SignalR (Blazor Server) |
| 6 | **Mobile (MAUI) لا يعتمد على أي مشروع آخر** | يتواصل مع Api.Web عبر HTTP/REST فقط |
| 7 | **Shared.UI يمكن استخدامه في Web و Mobile** | مكتبة مكونات Razor مشتركة (RCL) |
| 8 | **Tests يمكنه الاعتماد على أي مشروع** | لاختبار الوحدات والتكامل |

---

## مسؤوليات كل طبقة

### RubikCare.Domain (الطبقة الأساسية)
- **Entities:** الكيانات الأساسية (ApplicationUser, UserProfile, Organization, Medication، إلخ)
- **Enums:** التعدادات (UserStatus, ProgramType, RoleType)
- **Interfaces:** تعريف العقود (IRepository<T>, IUnitOfWork)
- **Value Objects:** كائنات القيمة (Address, Money)
- **Domain Events:** أحداث النطاق
- ❌ **لا يحتوي على:** أي إشارة لقاعدة البيانات أو EF Core أو APIs خارجية أو بيئة تشغيل

### RubikCare.Application (طبقة التطبيق)
- **Services:** خدمات منطق الأعمال (UserService, PspService, PharmacyService)
- **DTOs:** كائنات نقل البيانات (UserDto, ProgramDto, InvitationDto)
- **Interfaces:** تعريف عقود البنية التحتية (IUserRepository, IPspRepository)
- **Validators:** قواعد التحقق (FluentValidation)
- **Mappings:** تحويل Entities ↔ DTOs (AutoMapper أو يدوي)
- **Use Cases:** حالات استخدام محددة (CreateInvitationUseCase, EnrollPatientUseCase)
- ❌ **لا يحتوي على:** أي إشارة لـ IJSRuntime, IHttpContextAccessor, localStorage, أو أي اعتماد على Web/Mobile

### RubikCare.Infrastructure (طبقة البنية التحتية)
- **DbContext:** BusinessDbContext (EF Core)
- **Repositories:** تنفيذ IRepository<T>
- **Migrations:** Code-First Migrations
- **Services:** خدمات خارجية (EmailService, SmsService, FileStorageService)
- **Configuration:** إعدادات Entity Framework (Fluent API, IEntityTypeConfiguration)
- ⚠️ **تحذير:** يجب ألا تحتوي على IHttpContextAccessor (هذا من طبقة العرض)

### RubikCare.Api.Web (طبقة العرض - API)
- **Controllers:** ApiControllers (AuthController, PspController, UserController)
- **Middleware:** Authentication, Exception Handling, Logging
- **Program.cs:** تسجيل الخدمات، إعداد pipeline
- **Filters:** Action Filters, Exception Filters
- **Hubs:** SignalR Hubs (للتواصل الحي مع Blazor Server)

### RubikCare.Web (تطبيق Blazor Server)
- **Pages:** صفحات Razor (Dashboard, PSP, Admin, Profile)
- **Components:** مكونات قابلة لإعادة الاستخدام
- **Layouts:** التخطيطات الرئيسية
- **Services:** خدمات محلية (TranslationState, MenuService)
- **هنا يجب أن تكون:** أي خدمة تعتمد على IJSRuntime أو IHttpContextAccessor

### RubikCare.Mobile (تطبيق MAUI)
- **Pages:** صفحات XAML + BlazorWebView
- **ViewModels:** MVVM ViewModels
- **Services:** ApiService, AuthService, NavigationService
- **Platform-specific:** كاميرا، مسح، إشعارات
- **هنا يجب أن تكون:** أي خدمة تعتمد على Preferences أو SecureStorage

### RubikCare.Shared.UI (مكتبة المكونات المشتركة)
- **Components:** RubikSmartTable, RubikDropdown, RubikButton
- **Layouts:** تخطيطات مشتركة
- **Utilities:** دوال مساعدة مشتركة
- **Translation Services:** ISharedTranslationService, ISharedTranslationState

---

## تدفق البيانات بين الطبقات

```
طلب HTTP (من Web أو Mobile)
│
▼
┌─────────────────────────────────────────────────────────────────┐
│                    Api.Web - Controller                         │
│  - يستقبل الطلب                                                 │
│  - يحقق من صحة البيانات                                         │
│  - يحقق من الصلاحيات (JWT)                                      │
│  - يستدعي Application Service                                   │
└─────────────────────────────────────────────────────────────────┘
│
▼
┌─────────────────────────────────────────────────────────────────┐
│                  Application - Service                          │
│  - ينفذ منطق الأعمال                                            │
│  - يستخدم Repository Interface                                  │
│  - يحول Entities ↔ DTOs                                         │
│  - يعيد DTO أو Result                                           │
└─────────────────────────────────────────────────────────────────┘
│
▼
┌─────────────────────────────────────────────────────────────────┐
│                Infrastructure - Repository                      │
│  - ينفذ IRepository                                             │
│  - يتواصل مع قاعدة البيانات عبر DbContext                       │
│  - يعيد Entities                                                │
└─────────────────────────────────────────────────────────────────┘
│
▼
┌─────────────────────────────────────────────────────────────────┐
│                    Database - SQL Server                        │
└─────────────────────────────────────────────────────────────────┘
```

---

## التقنيات المستخدمة في كل مشروع

| المشروع | التقنيات الرئيسية |
|---------|-------------------|
| Domain | .NET 8 Class Library, C# |
| Application | .NET 8 Class Library, AutoMapper, FluentValidation |
| Infrastructure | .NET 8, Entity Framework Core, SQL Server |
| Api.Web | ASP.NET Core Minimal API, JWT, Swagger |
| Web | Blazor Server, Bootstrap 5, Custom CSS |
| Mobile | .NET MAUI, BlazorWebView, XAML |
| Shared.UI | Razor Class Library (RCL) |
| Tests | xUnit, Moq, FluentAssertions |

---

## ⚠️ تحذيرات معمارية عامة

1. **لا تكسر تبعية الطبقات:** لا تستدعي Infrastructure مباشرة من Web أو Mobile
2. **لا تضع منطق أعمال في Controllers:** Controllers للتنسيق فقط، المنطق في Application
3. **لا تضع استعلامات قاعدة بيانات في Domain:** Domain لا يعرف شيئاً عن EF Core
4. **استخدم DTOs:** لا تُرجع Entities مباشرة من API
5. **Mobile لا يعتمد على أي مشروع:** التواصل عبر HTTP فقط
6. **لا تستخدم IJSRuntime في Application Layer:** هذا انتهاك صريح لـ Clean Architecture
7. **لا تستخدم IHttpContextAccessor في Application أو Infrastructure:** هذا من طبقة العرض

---

## 🔴 انتهاكات Clean Architecture المكتشفة (يوليو 2026)

خلال عملية التطوير، تم اكتشاف عدة انتهاكات للمعمارية يجب توثيقها ومعالجتها تدريجياً:

### الانتهاك 1: استخدام IJSRuntime في Application Layer

**الملفات المتأثرة:**
- `RubikCare.Application/Services/TranslationStateService.cs`
- `RubikCare.Application/Services/LayoutDirectionService.cs`

**المشكلة:**
```csharp
// ❌ خطأ - في Application Layer
namespace RubikCare.Application.Services
{
    public class TranslationStateService
    {
        private readonly IJSRuntime _jsRuntime; // ⚠️ انتهاك!
        
        public TranslationStateService(IJSRuntime jsRuntime)
        {
            _jsRuntime = jsRuntime;
        }
        
        public async Task InitializeAsync()
        {
            // استخدام IJSRuntime هنا ينتهك قاعدة "Application يعتمد فقط على Domain"
            var savedLang = await _jsRuntime.InvokeAsync<string>(
                "localStorage.getItem", "RubikCare:Language");
        }
    }
}
```

**لماذا هذا خطأ؟**
- `IJSRuntime` هو مفهوم خاص ببيئة التشغيل (Web/Mobile)
- طبقة Application يجب أن تكون مستقلة عن بيئة التشغيل
- هذا يجعل اختبار الوحدة مستحيلاً بدون Mocking معقد
- يكسر إمكانية إعادة استخدام Application Layer في بيئات أخرى (CLI, Background Services)

**✅ الحل الصحيح (خطة الترحيل):**

```csharp
// ✅ الخطوة 1: جعل TranslationStateService "منطقية بحتة" في Application Layer
namespace RubikCare.Application.Services
{
    public class TranslationStateService
    {
        private string? _currentLanguage;
        public event Action? OnLanguageChanged;
        
        public string CurrentLanguage
        {
            get => _currentLanguage ?? CultureInfo.CurrentCulture.TwoLetterISOLanguageName;
            private set
            {
                if (_currentLanguage != value)
                {
                    _currentLanguage = value;
                    OnLanguageChanged?.Invoke();
                }
            }
        }
        
        // لا IJSRuntime - فقط منطق الحالة
        public void SetLanguage(string language)
        {
            CurrentLanguage = language;
        }
    }
}

// ✅ الخطوة 2: إنشاء "Bridge" في Web Layer يربط بين IJSRuntime والخدمة المنطقية
namespace Rubikcare.Web.Services
{
    public class WebTranslationState : ISharedTranslationState
    {
        private readonly TranslationStateService _translationStateService;
        private readonly IJSRuntime _jsRuntime; // ✅ آمن هنا في Web Layer
        
        public WebTranslationState(
            TranslationStateService translationStateService,
            IJSRuntime jsRuntime)
        {
            _translationStateService = translationStateService;
            _jsRuntime = jsRuntime;
        }
        
        public async Task InitializeAsync()
        {
            // قراءة من localStorage (Web-specific)
            var savedLang = await _jsRuntime.InvokeAsync<string>(
                "localStorage.getItem", "RubikCare:Language");
            
            // تحديث الخدمة المنطقية (Application Layer)
            _translationStateService.SetLanguage(savedLang ?? "ar");
        }
        
        public async Task SetLanguageAsync(string language)
        {
            // حفظ في localStorage (Web-specific)
            await _jsRuntime.InvokeVoidAsync(
                "localStorage.setItem", "RubikCare:Language", language);
            
            // تحديث الخدمة المنطقية
            _translationStateService.SetLanguage(language);
        }
    }
}

// ✅ الخطوة 3: نفس الشيء في Mobile Layer
namespace RubikCare.Mobile.Services
{
    public class MobileTranslationState : ISharedTranslationState
    {
        private readonly TranslationStateService _translationStateService;
        
        public MobileTranslationState(TranslationStateService translationStateService)
        {
            _translationStateService = translationStateService;
        }
        
        public void Initialize()
        {
            // قراءة من Preferences (Mobile-specific)
            var savedLang = Preferences.Get("RubikCare:Language", "ar");
            _translationStateService.SetLanguage(savedLang);
        }
        
        public void SetLanguage(string language)
        {
            // حفظ في Preferences (Mobile-specific)
            Preferences.Set("RubikCare:Language", language);
            _translationStateService.SetLanguage(language);
        }
    }
}
```

**حالة المعالجة:** ⏳ مخطط للترحيل - الأولوية المتوسطة

---

### الانتهاك 2: ازدواجية أنظمة الكاش

**الملفات المتأثرة:**
- `RubikCare.Infrastructure/Caching/LocalizationCacheService.cs`
- `RubikCare.Application/Services/LocalizationService.cs`

**المشكلة:**
```csharp
// ❌ خطأ - نظام كاش مخصص غير ضروري
public class LocalizationCacheService : ICacheService
{
    private readonly Dictionary<string, string> _cache = new();
    private readonly SemaphoreSlim _lock = new(1, 1);
    
    public bool TryGet(string key, out string? value)
        => _cache.TryGetValue(key, out value);
    
    public void Set(string key, string value)
        => _cache.TryAdd(key, value);
}

// وفي LocalizationService يتم استخدام كلا النظامين!
public class LocalizationService : ILocalizationService
{
    private readonly IMemoryCache _cache;           // ⚠️ نظام 1
    private readonly ICacheService _persistentCache; // ⚠️ نظام 2
    
    public async Task<string?> GetTranslationAsync(string key, string lang)
    {
        // استخدام IMemoryCache
        if (_cache.TryGetValue(cacheKey, out string? cachedValue))
            return cachedValue;
        
        // استخدام ICacheService أيضاً!
        if (_persistentCache.TryGet(cacheKey, out string? persistentValue))
        {
            _cache.Set(cacheKey, persistentValue, _cacheDuration);
            return persistentValue;
        }
        
        // ...
    }
}
```

**لماذا هذا خطأ؟**
- ازدواجية غير مبررة في استهلاك الذاكرة
- تعقيد غير ضروري في منطق الكاش
- `IMemoryCache` القياسي في .NET كافي تماماً لهذا الغرض
- `LocalizationCacheService` لا يضيف قيمة فوق `IMemoryCache`

**✅ الحل الصحيح:**

```csharp
// ✅ الاعتماد حصرياً على IMemoryCache
public class LocalizationService : ILocalizationService
{
    private readonly IMemoryCache _cache;
    private readonly TimeSpan _cacheDuration = TimeSpan.FromMinutes(30);
    
    public LocalizationService(IMemoryCache cache, ...)
    {
        _cache = cache;
    }
    
    public async Task<string?> GetTranslationAsync(string key, string lang)
    {
        var cacheKey = $"translation_{lang}_{key}";
        
        if (_cache.TryGetValue(cacheKey, out string? cachedValue))
            return cachedValue;
        
        // جلب من قاعدة البيانات
        var translation = await _dbFactory.ExecuteWithNewContextAsync(async context =>
        {
            // ... استعلام قاعدة البيانات
        });
        
        // حفظ في IMemoryCache فقط
        _cache.Set(cacheKey, translation, _cacheDuration);
        return translation;
    }
}

// ✅ حذف LocalizationCacheService تماماً
// ❌ public class LocalizationCacheService : ICacheService { ... }
```

**حالة المعالجة:** ⏳ مخطط للترحيل - الأولوية المنخفضة (لا يؤثر على الأداء بشكل ملحوظ)

---

### الانتهاك 3: تكرار تسجيل الخدمات في Program.cs

**الملفات المتأثرة:**
- `Rubikcare.Web/Program.cs`

**المشكلة:**
```csharp
// ❌ خطأ - تسجيلات مكررة
builder.Services.AddScoped<TranslationStateService>(); // السطر 120
// ... 50 سطر آخر ...
builder.Services.AddScoped<TranslationStateService>(); // السطر 170

builder.Services.AddScoped<IUserMenuService, UserMenuService>(); // السطر 130
// ... 30 سطر آخر ...
builder.Services.AddScoped<IUserMenuService, UserMenuService>(); // السطر 160

builder.Services.AddScoped<IUserRepository, UserRepository>(); // السطر 135
// ... 20 سطر آخر ...
builder.Services.AddScoped<IUserRepository, UserRepository>(); // السطر 155
```

**لماذا هذا خطأ؟**
- استهلاك غير ضروري للذاكرة (يتم إنشاء service descriptor مكرر)
- ارتباك للمطورين الجدد
- صعوبة في الصيانة

**✅ الحل الصحيح:**
```csharp
// ✅ تسجيل كل خدمة مرة واحدة فقط، مع تنظيمها في مناطق واضحة

// =====================================================
// منطقة 1: خدمات الترجمة
// =====================================================
builder.Services.AddScoped<TranslationStateService>();
builder.Services.AddScoped<ILayoutDirectionService, LayoutDirectionService>();
builder.Services.AddScoped<ILocalizationService, LocalizationService>();

// =====================================================
// منطقة 2: خدمات المستخدم والجلسات
// =====================================================
builder.Services.AddScoped<IUserRepository, UserRepository>();
builder.Services.AddScoped<IUserMenuService, UserMenuService>();
builder.Services.AddScoped<IUserSessionService, UserSessionService>();

// =====================================================
// منطقة 3: خدمات Shared UI
// =====================================================
builder.Services.AddScoped<ISharedTranslationState, WebTranslationState>();
builder.Services.AddScoped<ISharedTranslationService, WebTranslationService>();
```

**حالة المعالجة:** ✅ تم الحل جزئياً - يحتاج تنظيف نهائي

---

### الانتهاك 4: استخدام IHttpContextAccessor في Infrastructure Layer

**الملفات المتأثرة:**
- `RubikCare.Infrastructure/Services/UserContextService.cs`

**المشكلة:**
```csharp
// ❌ خطأ - في Infrastructure Layer
namespace RubikCare.Infrastructure.Services
{
    public class UserContextService : IUserContextService
    {
        private readonly IHttpContextAccessor _httpContextAccessor; // ⚠️ انتهاك!
        
        public UserContextService(IHttpContextAccessor httpContextAccessor, ...)
        {
            _httpContextAccessor = httpContextAccessor;
        }
        
        private string? GetCurrentUserId()
        {
            return _httpContextAccessor.HttpContext?.User?
                .FindFirstValue(ClaimTypes.NameIdentifier);
        }
    }
}
```

**لماذا هذا خطأ؟**
- `IHttpContextAccessor` هو مفهوم خاص بـ ASP.NET Core
- Infrastructure Layer يجب أن يكون مستقلاً عن إطار العمل
- هذا يمنع استخدام `UserContextService` في Background Jobs أو Console Apps

**✅ الحل الصحيح:**
```csharp
// ✅ تمرير userId كمعامل بدلاً من الاعتماد على HttpContext
public class UserContextService : IUserContextService
{
    // ❌ إزالة IHttpContextAccessor
    // private readonly IHttpContextAccessor _httpContextAccessor;
    
    // ✅ استقبال userId كمعامل
    public async Task<UserDashboardContextDto> GetUserDashboardContextAsync(string userId)
    {
        var user = await GetUserWithProfileAsync(userId);
        // ...
    }
}

// ✅ في Controller أو Page، نستخرج userId ونمرره
public class DashboardController : ControllerBase
{
    private readonly IUserContextService _userContextService;
    
    public async Task<IActionResult> GetDashboard()
    {
        var userId = User.FindFirstValue(ClaimTypes.NameIdentifier);
        var context = await _userContextService.GetUserDashboardContextAsync(userId);
        return Ok(context);
    }
}
```

**حالة المعالجة:** ⏳ مخطط للترحيل - الأولوية المنخفضة

---

### الانتهاك 5: الصفحات القديمة (Legacy Pages) تحقن DbContext مباشرة

**الملفات المتأثرة:**
- بعض صفحات `Rubikcare.Web/Components/Pages/` القديمة

**المشكلة:**
```razor
@* ❌ خطأ - صفحة قديمة تحقن DbContext مباشرة *@
@page "/old-page"
@inject BusinessDbContext DbContext

@code {
    protected override async Task OnInitializedAsync()
    {
        // استخدام DbContext مباشرة في Razor Page
        var users = await DbContext.Users.ToListAsync();
    }
}
```

**لماذا هذا خطأ؟**
- كسر لـ Clean Architecture (View Layer لا يجب أن يعرف DbContext)
- مشاكل التزامن (Concurrency) عند استخدام DbContext في Blazor Server
- صعوبة في الاختبار

**✅ الحل الصحيح:**
```razor
@* ✅ صحيح - استخدام Service Layer *@
@page "/new-page"
@inject IGenericService<User> UserService

@code {
    private List<User> _users = new();
    
    protected override async Task OnInitializedAsync()
    {
        _users = (await UserService.GetAllAsync()).ToList();
    }
}
```

**حالة المعالجة:** ⏳ ترحيل تدريجي مستمر - الأولوية المنخفضة

---

## 📋 خطة الترحيل التدريجي (يوليو 2026)

### المرحلة 1: ✅ مكتملة (يوليو 2026)
- [x] إصلاح مشاكل SSR في `App.razor`
- [x] توحيد اتجاه الصفحة مع اللغة في `MainLayout.razor`
- [x] إصلاح مشكلة إعادة التوجيه في `InteractiveMenu.razor`
- [x] تحسين الأداء في `InteractiveMenu.razor` (منع التحميل المتكرر)

### المرحلة 2: ⏳ جارية (يوليو 2026)
- [ ] توثيق الانتهاكات المعمارية (هذا الملف)
- [ ] تحديث `01-program-cs-foundation.md` بالدروس المستفادة
- [ ] تنظيف التسجيلات المكررة في `Program.cs`

### المرحلة 3: 📅 مخططة (أغسطس 2026)
- [ ] نقل `TranslationStateService` من Application إلى Web/Mobile
- [ ] إنشاء `WebTranslationState` كـ Bridge في Web Layer
- [ ] إنشاء `MobileTranslationState` كـ Bridge في Mobile Layer
- [ ] إزالة `LocalizationCacheService` والاعتماد على `IMemoryCache` فقط

### المرحلة 4: 📅 مخططة (سبتمبر 2026)
- [ ] إزالة `IHttpContextAccessor` من `UserContextService`
- [ ] تمرير `userId` كمعامل بدلاً من الاعتماد على HttpContext
- [ ] ترحيل الصفحات القديمة (Legacy Pages) لاستخدام Service Layer

---

## 🧪 اختبارات حماية المعمارية (Architecture Tests)

يستخدم المشروع مكتبة `NetArchTest.Rules` لحماية المعمارية من الانتهاكات المستقبلية:

```csharp
// مثال من RubikCare.Tests/ArchitectureTests.cs
[Fact]
public void Application_Should_Not_Reference_AspNetCore()
{
    var result = Types.InAssembly(typeof(TranslationStateService).Assembly)
        .ShouldNot()
        .HaveDependencyOn("Microsoft.AspNetCore")
        .GetResult();
    
    Assert.True(result.IsSuccessful);
}

[Fact]
public void Domain_Should_Not_Reference_EntityFramework()
{
    var result = Types.InAssembly(typeof(UserProfile).Assembly)
        .ShouldNot()
        .HaveDependencyOn("Microsoft.EntityFrameworkCore")
        .GetResult();
    
    Assert.True(result.IsSuccessful);
}
```

**عدد الاختبارات المعمارية:** 33 اختبار  
**الحالة:** ✅ جميعها ناجحة (حتى مع الانتهاكات الحالية، لأن الاختبارات تركز على التبعيات بين المشاريع وليس داخل المشروع الواحد)

---

## 🔗 روابط ذات صلة

- [01 - Program.cs والتسجيلات الأساسية](01-program-cs-foundation.md)
- [02 - نظام الهوية والمصادقة](02-identity-system.md)
- [09 - دليل API](09-api-guide.md)
- [13 - تطبيق Clean Architecture](13-clean-architecture-enforcement.md)
- [الملحق ب - فهرس الخدمات](../appendix-b-service-index.md)

---

## 📝 سجل التحديثات

| التاريخ | الإصدار | التغييرات |
|---------|---------|-----------|
| 17 مايو 2026 | 1.0 | الإصدار الأولي |
| 14 يوليو 2026 | 1.2 | إضافة قسم "انتهاكات Clean Architecture المكتشفة" + "خطة الترحيل التدريجي" + "اختبارات حماية المعمارية" |
```

---

## ✅ ملخص التحديثات المضافة

1. **قسم جديد: "انتهاكات Clean Architecture المكتشفة"** - يغطي 5 انتهاكات حقيقية مع أمثلة عملية من المشروع
2. **قسم جديد: "خطة الترحيل التدريجي"** - 4 مراحل زمنية واضحة
3. **قسم جديد: "اختبارات حماية المعمارية"** - أمثلة على NetArchTest
4. **تحديث قسم "تحذيرات معمارية عامة"** - إضافة 2 تحذيرات جديدة
5. **تحديث "مسؤوليات كل طبقة"** - إضافة توضيحات لما يجب ألا تحتويه كل طبقة
6. **سجل التحديثات** - لتتبع التغييرات

