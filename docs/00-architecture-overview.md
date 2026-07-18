# 00 - الهيكل المعماري للمشاريع الثمانية (Clean Architecture)

**آخر تحديث:** 18 يوليو 2026

## نظرة عامة

يتبع مشروع RubikCare نمط Clean Architecture مقسماً إلى 8 مشاريع منفصلة، كل منها له مسؤولية محددة وعلاقات واضحة مع المشاريع الأخرى.

## هيكل المشاريع والعلاقات

```
┌─────────────────────────────────────────────────────────────────┐
│                        RubikCare.Api.Web                        │
│                   (Presentation Layer - API)                    │
│                  المسؤول: استقبال طلبات HTTP                    │
│                  يعتمد على: Application, Infrastructure         │
└─────────────────────────────────────────────────────────────────┘
                                     │
                                     ▼
┌─────────────────────────────────────────────────────────────────┐
│                     RubikCare.Application                       │
│                  (Application Layer - Use Cases)                │
│                  المسؤول: منطق الأعمال والتنسيق                │
│                  يعتمد على: Domain فقط                          │
│                  يحتوي: Services, Interfaces, DTOs              │
└─────────────────────────────────────────────────────────────────┘
                                     │
                                     ▼
┌─────────────────────────────────────────────────────────────────┐
│                   RubikCare.Infrastructure                      │
│                  (Infrastructure Layer - Data Access)           │
│                  المسؤول: التواصل مع قاعدة البيانات             │
│                  يعتمد على: Application, Domain                 │
│                  يحتوي: DbContext, Repositories, Migrations     │
└─────────────────────────────────────────────────────────────────┘
                                     │
                                     ▼
┌─────────────────────────────────────────────────────────────────┐
│                       RubikCare.Domain                          │
│                       (Domain Layer - Core)                     │
│                  المسؤول: الكيانات الأساسية وقواعد العمل         │
│                  لا يعتمد على أي مشروع آخر                      │
│                  يحتوي: Entities, Enums, Interfaces             │
└─────────────────────────────────────────────────────────────────┘

┌─────────────────────────────────────────────────────────────────┐
│                       RubikCare.Mobile                          │
│                       (MAUI Application)                        │
│                  المسؤول: تطبيق الموبايل                        │
│                  يتصل بـ: Api.Web فقط (عبر HTTP)                │
│                  لا يعتمد على أي مشروع آخر                      │
└─────────────────────────────────────────────────────────────────┘

┌─────────────────────────────────────────────────────────────────┐
│                        RubikCare.Web                            │
│                   (Blazor Server Application)                   │
│                  المسؤول: تطبيق الويب                           │
│                  يعتمد على: Api.Web (ضمنياً)                    │
└─────────────────────────────────────────────────────────────────┘

┌─────────────────────────────────────────────────────────────────┐
│                     RubikCare.Shared.UI                         │
│                   (Razor Class Library)                         │
│                  المسؤول: مكونات UI مشتركة بين Web و Mobile     │
│                  يحتوي: مكونات Razor قابلة لإعادة الاستخدام     │
└─────────────────────────────────────────────────────────────────┘

┌─────────────────────────────────────────────────────────────────┐
│                       RubikCare.Tests                           │
│                       (Test Project)                            │
│                  المسؤول: اختبارات الوحدة والتكامل              │
└─────────────────────────────────────────────────────────────────┘
```

## قواعد إلزامية للعلاقات بين المشاريع

| # | القاعدة | التفاصيل |
|---|---------|----------|
| 1 | **Domain لا يعتمد على أحد** | أنقى طبقة - تحتوي فقط على Entities, Enums, Value Objects, Interfaces |
| 2 | **Application يعتمد فقط على Domain** | يحتوي على Use Cases, Services, DTOs - لا يعرف شيئاً عن قاعدة البيانات |
| 3 | **Infrastructure يعتمد على Application و Domain** | ينفذ الواجهات المعرفة في Application - يحتوي على DbContext, Repositories, Migrations |
| 4 | **Api.Web يعتمد على Application و Infrastructure** | نقطة الدخول الوحيدة للبيانات - يحتوي على Controllers, Middleware, Program.cs |
| 5 | **Web (Blazor) يعتمد على Api.Web** | يتواصل مع الخادم عبر SignalR (Blazor Server) |
| 6 | **Mobile (MAUI) لا يعتمد على أي مشروع آخر** | يتواصل مع Api.Web عبر HTTP/REST فقط |
| 7 | **Shared.UI يمكن استخدامه في Web و Mobile** | مكتبة مكونات Razor مشتركة (RCL) |
| 8 | **Tests يمكنه الاعتماد على أي مشروع** | لاختبار الوحدات والتكامل |

## مسؤوليات كل طبقة

### RubikCare.Domain (الطبقة الأساسية)
- **Entities:** الكيانات الأساسية (ApplicationUser, UserProfile, Organization, Medication, إلخ)
- **Enums:** التعدادات (UserStatus, ProgramType, RoleType)
- **Interfaces:** تعريف العقود (IRepository<T>, IUnitOfWork)
- **Value Objects:** كائنات القيمة (Address, Money)
- **Domain Events:** أحداث النطاق
- **لا يحتوي على:** أي إشارة لقاعدة البيانات أو EF Core أو APIs خارجية

### RubikCare.Application (طبقة التطبيق)
- **Services:** خدمات منطق الأعمال (UserService, PspService, PharmacyService)
- **DTOs:** كائنات نقل البيانات (UserDto, ProgramDto, InvitationDto)
- **Interfaces:** تعريف عقود البنية التحتية (IUserRepository, IPspRepository)
- **Validators:** قواعد التحقق (FluentValidation)
- **Mappings:** تحويل Entities ↔ DTOs (AutoMapper أو يدوي)
- **Use Cases:** حالات استخدام محددة (CreateInvitationUseCase, EnrollPatientUseCase)

### RubikCare.Infrastructure (طبقة البنية التحتية)
- **DbContext:** BusinessDbContext (EF Core)
- **Repositories:** تنفيذ IRepository<T>
- **Migrations:** Code-First Migrations
- **Services:** خدمات خارجية (EmailService, SmsService, FileStorageService)
- **Configuration:** إعدادات Entity Framework (Fluent API, IEntityTypeConfiguration)

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

### RubikCare.Mobile (تطبيق MAUI)
- **Pages:** صفحات XAML + BlazorWebView
- **ViewModels:** MVVM ViewModels
- **Services:** ApiService, AuthService, NavigationService
- **Platform-specific:** كاميرا، مسح، إشعارات

### RubikCare.Shared.UI (مكتبة المكونات المشتركة)
- **Components:** RubikSmartTable, RubikDropdown, RubikButton
- **Layouts:** تخطيطات مشتركة
- **Utilities:** دوال مساعدة مشتركة

### RubikCare.Tests (مشروع الاختبارات)
- **Unit Tests:** اختبارات الوحدات لكل طبقة
- **Integration Tests:** اختبارات التكامل بين الطبقات
- **Architecture Tests:** اختبارات حماية المعمارية (NetArchTest) ⭐

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
│                   Application - Service                         │
│  - ينفذ منطق الأعمال                                            │
│  - يستخدم Repository Interface                                  │
│  - يحول Entities ↔ DTOs                                         │
│  - يعيد DTO أو Result                                           │
└─────────────────────────────────────────────────────────────────┘
                                     │
                                     ▼
┌─────────────────────────────────────────────────────────────────┐
│                 Infrastructure - Repository                     │
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
| Tests | xUnit, Moq, FluentAssertions, NetArchTest ⭐ |

---

## 🧭 شجرة القرار المعماري: كيف تختار النهج الصحيح (⭐ جديد - 18 يوليو 2026)

### لماذا هذا القسم مهم؟
في المشاريع الحقيقية، لا يكفي أن تعرف **"ما هي المعمارية"**، بل يجب أن تعرف **"متى تستخدم ماذا"**. هذا القسم يحل التناقض بين الوثائق ويوفر دليلاً عملياً للمطور لاتخاذ القرارات الصحيحة.

### مستويات "النقاء المعماري" (من الأسوأ إلى الأفضل)

```
المستوى 1 ❌ (مرفوض):
Blazor Page → DbContext مباشرة

المستوى 2 ⚠️ (مقبول حالياً - للصفحات القديمة):
Blazor Page → IGenericService<T> → DbContext

المستوى 3 ✅ (الموصى به للصفحات المعقدة):
Blazor Page → UseCase (Application) → Repository (Infrastructure) → DbContext

المستوى 4 🏆 (الأنقى نظرياً - للعمليات المشتركة):
Blazor Page → HTTP → Api.Web → UseCase → Repository → DbContext
```

### 🎯 القاعدة الذهبية: "اختر المستوى المناسب لتعقيد الصفحة"

| نوع الصفحة | النهج الموصى به | السبب |
|-----------|----------------|-------|
| **CRUD بسيط** (إضافة/تعديل/حذف) | `IGenericService<T>` | سريع، بسيط، لا يحتاج تعقيد UseCase |
| **صفحة معقدة** (تقارير، عمليات تجارية) | **UseCase** | يعبر عن "نية العمل" بوضوح |
| **صفحة مشتركة بين Web و Mobile** | **API** | إعادة استخدام المنطق |
| **عمليات حساسة** (دفع، مصادقة) | **UseCase + API** | أمان + اختبارية |

### 📊 شجرة القرار التفصيلية

```
هل الصفحة جديدة؟
│
├── نعم → هل العملية مشتركة بين Web و Mobile؟
│         │
│         ├── نعم → استخدم UseCase في Application + API في Api.Web
│         │         (مثال: SyncCompanyMedications, CreatePsp)
│         │
│         └── لا → هل العملية معقدة (منطق أعمال، تحويلات، تحقق متعدد)؟
│                   │
│                   ├── نعم → استخدم UseCase في Application
│                   │         (مثال: EnrollPatientUseCase, ProcessOrderUseCase)
│                   │
│                   └── لا → هل العملية CRUD بسيطة؟
│                             │
│                             ├── نعم → استخدم IGenericService<T>
│                             │         (مثال: عرض قائمة أدوية، تعديل اسم)
│                             │
│                             └── لا → استخدم DbContextFactoryService
│                                       (للاستعلامات المخصصة المعقدة)
│
└── لا (صفحة قديمة) → هل تستخدم @inject BusinessDbContext مباشرة؟
                      │
                      ├── نعم → خطط لترحيلها تدريجياً
                      │         (ابدأ بـ IGenericService ثم UseCase)
                      │
                      └── لا → اتركها كما هي إذا كانت تعمل
```

### 🏛️ أنواع الصفحات الخمسة

| النوع | الوصف | الأمثلة | النهج |
|------|-------|---------|-------|
| **Type 1: CRUD بسيطة** | عرض/إضافة/تعديل/حذف كيان واحد | قائمة الأدوية، تعديل اسم | `IGenericService<T>` |
| **Type 2: عمليات معقدة** | منطق أعمال متعدد الخطوات | إنشاء PSP، تسجيل مريض | UseCase |
| **Type 3: مشتركة** | تُستخدم في Web و Mobile | المزامنة، الإشعارات | UseCase + API |
| **Type 4: حساسة** | عمليات مالية/أمنية | الدفع، تغيير كلمة المرور | UseCase + API + Validation |
| **Type 5: تقارير** | استعلامات قراءة معقدة | لوحة التحكم، الإحصائيات | DbContextFactoryService |

### 🎨 قواعد Shared.UI: مكونات vs صفحات

#### ✅ ما يوضع في Shared.UI:
- **المكونات القابلة لإعادة الاستخدام** (RubikSmartTable, RubikDropdown)
- **الأقسام المشتركة** (SettingsPage, SecuritySection, ProfileHeader)
- **الخدمات المشتركة** (TranslationState, LayoutDirection)

#### ❌ ما لا يوضع في Shared.UI:
- **صفحات كاملة** (Dashboard, PSP, Admin)
- **منطق أعمال** (يجب أن يكون في Application)
- **اتصالات API** (يجب أن تكون في ApiService الخاص بكل منصة)

#### 💡 لماذا هذا التمييز مهم؟

1. **كسر مبدأ "Single Responsibility":** Shared.UI يصبح "مصغراً" لكل شيء
2. **مشاكل الأداء في MAUI:** BlazorWebView أبطأ، والصفحات الكبيرة تزيد الحمل
3. **صعوبة الاختبار:** كيف تختبر صفحة تعتمد على `NavigationManager` في Shared.UI؟
4. **تضارب التبعيات:** Web يحتاج `DbContextFactory`، Mobile يحتاج `ApiService`

### 📋 المعمارية الصحيحة (نموذج مرجعي)

```
┌─────────────────────────────────────────────────────────────────┐
│                    RubikCare.Web (Blazor Server)                │
│  ├── Pages/                                                     │
│  │   ├── Dashboard.razor              ← صفحة كاملة             │
│  │   ├── PSP.razor                    ← صفحة كاملة             │
│  │   └── Admin/Medications.razor      ← صفحة كاملة             │
│  │                                                              │
│  └── يستخدم:                                                    │
│      ├── Shared.UI Components         (مكونات فقط)             │
│      ├── DbContextFactory             (للصفحات البسيطة)         │
│      └── UseCases                     (للصفحات المعقدة)         │
└─────────────────────────────────────────────────────────────────┘

┌─────────────────────────────────────────────────────────────────┐
│                    RubikCare.Mobile (MAUI)                      │
│  ├── Pages/                                                     │
│  │   ├── DashboardPage.xaml           ← صفحة كاملة             │
│  │   └── PSPPage.xaml                 ← صفحة كاملة             │
│  │                                                              │
│  └── يستخدم:                                                    │
│      ├── Shared.UI Components         (مكونات فقط)             │
│      └── ApiService                   (HTTP calls)              │
└─────────────────────────────────────────────────────────────────┘

┌─────────────────────────────────────────────────────────────────┐
│                    RubikCare.Shared.UI (RCL)                    │
│  ├── Components/                                                │
│  │   ├── RubikSmartTable.razor        ← مكون                   │
│  │   ├── Patient/                                               │
│  │   │   ├── SettingsPage.razor       ← مكون (ليس صفحة!)       │
│  │   │   └── SecuritySection.razor    ← مكون                   │
│  │   └── Shared/DashboardCard.razor   ← مكون                   │
│  │                                                              │
│  └── لا يستخدم:                                                 │
│      ❌ DbContext                                               │
│      ❌ ApiService                                              │
│      ❌ NavigationManager (إلا عبر Parameters)                  │
└─────────────────────────────────────────────────────────────────┘
```

### 💡 أمثلة عملية على القرارات الصحيحة

#### ✅ مثال 1: صفحة CRUD بسيطة (عرض قائمة أدوية)
```csharp
// في Web/Pages/Admin/Medications.razor
@inject IGenericService<Medication> MedicationService

@code {
    private List<Medication> medications = new();
    
    protected override async Task OnInitializedAsync()
    {
        medications = await MedicationService.GetAllAsync().ToListAsync();
    }
}
```
**السبب:** عملية قراءة بسيطة، لا تحتاج UseCase.

#### ✅ مثال 2: عملية معقدة (مزامنة أدوية الشركة)
```csharp
// في Application/UseCases/Medication/SyncCompanyMedicationsUseCase.cs
public class SyncCompanyMedicationsUseCase
{
    private readonly IMedicationRepository _repository;
    
    public async Task<Result> ExecuteAsync(int companyId)
    {
        // منطق معقد: مزامنة، تحقق، تحويل
        var companyMeds = await _repository.GetCompanyMedicationsAsync(companyId);
        // ... منطق الأعمال ...
        await _repository.SyncToMainTableAsync(companyMeds);
        return Result.Success();
    }
}

// في Web/Pages/Admin/SyncMedications.razor
@inject SyncCompanyMedicationsUseCase SyncUseCase

@code {
    private async Task SyncNow()
    {
        var result = await SyncUseCase.ExecuteAsync(CurrentCompanyId);
        // ...
    }
}
```
**السبب:** عملية معقدة متعددة الخطوات، تحتاج UseCase.

#### ✅ مثال 3: عملية مشتركة (Web + Mobile)
```csharp
// في Application/UseCases/Medication/SyncCompanyMedicationsUseCase.cs
// (نفس UseCase أعلاه)

// في Api.Web/Controllers/MedicationController.cs
[HttpPost("sync")]
public async Task<IActionResult> Sync([FromBody] SyncRequest request)
{
    var result = await _syncUseCase.ExecuteAsync(request.CompanyId);
    return Ok(result);
}

// في Mobile - يستدعي API
var result = await ApiService.PostAsync<SyncResult>("api/medication/sync", 
    new { CompanyId = 123 });

// في Web - يستدعي UseCase مباشرة (لأنه على نفس الخادم)
var result = await SyncUseCase.ExecuteAsync(123);
```
**السبب:** العملية مشتركة، نضع UseCase في Application ونستدعيها بطرق مختلفة.

### ⚠️ تحذيرات معمارية

1. **لا تكسر تبعية الطبقات:** لا تستدعي Infrastructure مباشرة من Web أو Mobile
2. **لا تضع منطق أعمال في Controllers:** Controllers للتنسيق فقط، المنطق في Application
3. **لا تضع استعلامات قاعدة بيانات في Domain:** Domain لا يعرف شيئاً عن EF Core
4. **استخدم DTOs:** لا تُرجع Entities مباشرة من API
5. **Mobile لا يعتمد على أي مشروع:** التواصل عبر HTTP فقط
6. **Shared.UI للمكونات فقط:** لا تضع صفحات كاملة فيه
7. **لا تستخدم `@inject BusinessDbContext` في صفحات جديدة:** استخدم `IDbContextFactory<BusinessDbContext>` أو `IGenericService<T>`

---

## ⚠️ تحذيرات معمارية

- لا تكسر تبعية الطبقات: لا تستدعي Infrastructure مباشرة من Web أو Mobile
- لا تضع منطق أعمال في Controllers: Controllers للتنسيق فقط، المنطق في Application
- لا تضع استعلامات قاعدة بيانات في Domain: Domain لا يعرف شيئاً عن EF Core
- استخدم DTOs: لا تُرجع Entities مباشرة من API
- Mobile لا يعتمد على أي مشروع: التواصل عبر HTTP فقط

## 🔗 روابط ذات صلة

- [01 - Program.cs والتسجيلات الأساسية](01-program-cs-foundation.md)
- [02 - نظام الهوية والمصادقة](02-identity-system.md)
- [05 - إنشاء الصفحات والمكونات](05-page-creation-checklist.md)
- [09 - دليل API](09-api-guide.md)
- [13 - تطبيق Clean Architecture](13-clean-architecture-enforcement.md)
- [الملحق ب - فهرس الخدمات](../appendix-b-service-index.md)
```

---

## 📊 ملخص التحديثات في هذه الوثيقة

| القسم | التغيير |
|-------|---------|
| **القسم الجديد: 🧭 شجرة القرار المعماري** | ✅ إضافة كاملة مع أمثلة عملية |
| **مستويات النقاء المعماري** | ✅ شرح 4 مستويات من الأسوأ للأفضل |
| **القاعدة الذهبية** | ✅ جدول يربط نوع الصفحة بالنهج |
| **شجرة القرار التفصيلية** | ✅ Mermaid Diagram للتوجيه |
| **أنواع الصفحات الخمسة** | ✅ تصنيف واضح مع أمثلة |
| **قواعد Shared.UI** | ✅ مكونات vs صفحات |
| **أمثلة عملية** | ✅ 3 أمثلة كود جاهزة |
| **المعمارية الصحيحة** | ✅ رسم توضيحي للمشاريع الثلاثة |
| **المحتوى الأصلي** | ✅ محفوظ بالكامل دون حذف |
