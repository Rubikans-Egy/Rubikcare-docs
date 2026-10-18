
# 11 — PAGE-CREATION (إنشاء الصفحات + Clean Architecture)

**الإصدار:** 2.0 (Unified Edition)
**آخر تحديث:** 17 أكتوبر 2026
**الحالة:** ✅ مرجع شامل — يدمج 05 + 13
**الجمهور:** فريق التطوير + AI Assistants
**المرجع الإلزامي:** يُقرأ قبل إنشاء أي صفحة جديدة

---

## 📌 الجزء صفر: الفلسفة

### 0.1 المبدأ الأساسي

> **"اختر المستوى المناسب لتعقيد الصفحة — لا تبالغ، لا تُبسّط."**

### 0.2 المشكلة التي تحلها هذه الوثيقة

في المشاريع الحقيقية، لا يكفي أن تعرف **"ما هي المعمارية"**، بل يجب أن تعرف **"متى تستخدم ماذا"**.

**⚠️ هذا القسم يحل التناقض بين الوثائق ويوفر دليلاً عملياً للمطور لاتخاذ القرارات الصحيحة.**

---

## 🧭 الجزء الأول: شجرة القرار المعماري

### 1.1 مستويات "النقاء المعماري" (من الأسوأ إلى الأفضل)

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

### 1.2 القاعدة الذهبية: "اختر المستوى المناسب لتعقيد الصفحة"

| نوع الصفحة | النهج الموصى به | السبب |
|-----------|----------------|-------|
| **CRUD بسيط** | `IGenericService<T>` | سريع، بسيط، لا يحتاج تعقيد UseCase |
| **صفحة معقدة** | **UseCase** | يعبر عن "نية العمل" بوضوح |
| **صفحة مشتركة بين Web و Mobile** | **API** | إعادة استخدام المنطق |
| **عمليات حساسة** | **UseCase + API** | أمان + اختبارية |

### 1.3 شجرة القرار التفصيلية

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

---

## 🎯 الجزء الثاني: أنواع الصفحات الخمسة

| النوع | الوصف | الأمثلة | النهج |
|------|-------|---------|-------|
| **Type 1: CRUD بسيطة** | عرض/إضافة/تعديل/حذف كيان واحد | قائمة الأدوية، تعديل اسم | `IGenericService<T>` |
| **Type 2: عمليات معقدة** | منطق أعمال متعدد الخطوات | إنشاء PSP، تسجيل مريض | UseCase |
| **Type 3: مشتركة** | تُستخدم في Web و Mobile | المزامنة، الإشعارات | UseCase + API |
| **Type 4: حساسة** | عمليات مالية/أمنية | الدفع، تغيير كلمة المرور | UseCase + API + Validation |
| **Type 5: تقارير** | استعلامات قراءة معقدة | لوحة التحكم، الإحصائيات | `DbContextFactoryService` |

---

## 📊 الجزء الثالث: مصفوفة اختيار الخدمة

### 3.1 خدمات أساسية (ضرورية لكل صفحة)

| الخدمة | الواجهة | متى تستخدمها؟ | مثال |
|--------|---------|---------------|------|
| `GenericService<T>` | `IGenericService<T>` | **CRUD بسيط** على كيان واحد | قائمة الأدوية، تعديل اسم |
| `DbContextFactoryService` | - | **استعلامات قراءة مخصصة** معقدة | لوحة التحكم، التقارير |
| `UserContextService` | `IUserContextService` | **معلومات المستخدم الحالي** | التحقق من صلاحية الوصول |

### 3.2 خدمات الترجمة (لأي صفحة تدعم اللغتين)

| الخدمة | الواجهة | متى تستخدمها؟ |
|--------|---------|---------------|
| `TranslationStateService` | - | **اللغة الحالية** + الاشتراك |
| `ILocalizationService` | `ILocalizationService` | **الترجمات من DB** |
| `LayoutDirectionService` | `ILayoutDirectionService` | **اتجاه الصفحة** (RTL/LTR) |

### 3.3 خدمات المستخدم والسياق

| الخدمة | الواجهة | متى تستخدمها؟ |
|--------|---------|---------------|
| `UserContextService` | `IUserContextService` | ID المستخدم، الأدوار، الصلاحيات |
| `IUserSessionService` | `IUserSessionService` | localStorage + إدارة الجلسات |
| `IUserRoleService` | `IUserRoleService` | إدارة الأدوار الموسعة |

### 3.4 خدمات التنقل والقوائم

| الخدمة | الواجهة | متى تستخدمها؟ |
|--------|---------|---------------|
| `DynamicMenuService` | - | القوائم الديناميكية |

### 3.5 خدمات مساعدة

| الخدمة | الواجهة | متى تستخدمها؟ |
|--------|---------|---------------|
| `ExcelService` | `IExcelService<T>` | استيراد/تصدير Excel |
| `IImageService` | `IImageService` | رفع ومعالجة الصور |

---

## 💡 الجزء الرابع: أمثلة عملية

### 4.1 ✅ مثال 1: صفحة CRUD بسيطة

**القرار:** `IGenericService<T>`

```csharp
// في Web/Pages/Admin/Medications.razor
@inject IGenericService<Medication> MedicationService

@code {
    private List<Medication> medications = new();
    
    protected override async Task OnInitializedAsync()
    {
        medications = await MedicationService.GetAllAsync().ToListAsync();
    }
    
    private async Task DeleteMedication(int id)
    {
        await MedicationService.DeleteAsync(id);
    }
}
```

**السبب:** عملية قراءة/حذف بسيطة.

### 4.2 ✅ مثال 2: عملية معقدة

**القرار:** UseCase

```csharp
// في Application/UseCases/Medication/SyncCompanyMedicationsUseCase.cs
public class SyncCompanyMedicationsUseCase
{
    private readonly IMedicationRepository _repository;
    
    public async Task<Result> ExecuteAsync(int companyId)
    {
        var companyMeds = await _repository.GetCompanyMedicationsAsync(companyId);
        // ... منطق معقد ...
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
    }
}
```

**السبب:** عملية معقدة متعددة الخطوات.

### 4.3 ✅ مثال 3: استعلام قراءة معقد

**القرار:** `DbContextFactoryService`

```csharp
// في Web/Pages/Dashboard.razor
@inject DbContextFactoryService DbFactory

@code {
    private DashboardStats? stats;
    
    protected override async Task OnInitializedAsync()
    {
        await using var context = await DbFactory.CreateDbContextAsync();
        
        stats = new DashboardStats
        {
            TotalPatients = await context.UserProfiles.CountAsync(up => up.IsActive),
            ActivePrograms = await context.PSPs.CountAsync(p => p.IsActive),
            RecentOrders = await context.Orders
                .OrderByDescending(o => o.CreatedDate)
                .Take(10)
                .ToListAsync()
        };
    }
}
```

**السبب:** استعلام مخصص معقد.

### 4.4 ✅ مثال 4: عملية مشتركة

**القرار:** UseCase + API

```csharp
// 1. UseCase
public class SyncCompanyMedicationsUseCase { ... }

// 2. Api.Web/Controllers/MedicationController.cs
[HttpPost("sync")]
public async Task<IActionResult> Sync([FromBody] SyncRequest request)
{
    var result = await _syncUseCase.ExecuteAsync(request.CompanyId);
    return Ok(result);
}

// 3. Mobile - يستدعي API
var result = await ApiService.PostAsync<SyncResult>(
    "api/medication/sync", 
    new { CompanyId = 123 });

// 4. Web - يستدعي UseCase مباشرة
var result = await SyncUseCase.ExecuteAsync(123);
```

**السبب:** العملية مشتركة.

---

## 🏛️ الجزء الخامس: قواعد Shared.UI — مكونات vs صفحات

### 5.1 ✅ ما يوضع في `Shared.UI`

- **المكونات القابلة لإعادة الاستخدام** (`RubikSmartTable`, `RubikDropdown`).
- **الأقسام المشتركة** (`SettingsPage`, `SecuritySection`, `ProfileHeader`).
- **الخدمات المشتركة** (`TranslationState`, `LayoutDirection`).

### 5.2 ❌ ما لا يوضع في `Shared.UI`

- **صفحات كاملة** (`Dashboard`, `PSP`, `Admin`).
- **منطق أعمال** (يجب أن يكون في `Application`).
- **اتصالات API** (يجب أن تكون في `ApiService` الخاص بكل منصة).

### 5.3 💡 لماذا هذا التمييز مهم؟

1. **كسر مبدأ "Single Responsibility":** `Shared.UI` يصبح "مصغراً" لكل شيء.
2. **مشاكل الأداء في MAUI:** `BlazorWebView` أبطأ، والصفحات الكبيرة تزيد الحمل.
3. **صعوبة الاختبار:** كيف تختبر صفحة تعتمد على `NavigationManager` في `Shared.UI`؟
4. **تضارب التبعيات:** Web يحتاج `DbContextFactory`، Mobile يحتاج `ApiService`.

### 5.4 📋 المعمارية الصحيحة (نموذج مرجعي)

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

---

## 🔴 الجزء السادس: Clean Architecture — Enforcement

### 6.1 حالة الإصلاحات الحالية

| الإصلاح | الحالة | التاريخ |
|---------|--------|---------|
| إنشاء أول Use Case (`ClearUserCacheUseCase`) | ✅ تم | 24 مايو 2026 |
| توحيد نظام الكاش | ✅ تم | 24 مايو 2026 |
| إزالة الكاش من `DynamicMenuService` | ✅ تم | 24 مايو 2026 |
| دمج `UserBootstrapService` ← `UserSessionService` | ✅ تم | 24 مايو 2026 |
| نقل `IUserBootstrapRepository` إلى Application | ✅ تم | 24 مايو 2026 |
| **إزالة References غير الصحيحة من Api.Web** | ✅ تم | 25 مايو 2026 |
| **إنشاء `InfrastructureExtensions.cs` موحد** | ✅ تم | 25 مايو 2026 |
| **إصلاح 16 Controller** | ✅ تم | 25 مايو 2026 |
| **Architecture Tests (33 اختبار ناجح)** | ✅ تم | 25 مايو 2026 |
| تحويل Controllers لاستخدام Use Cases | ⬜ مستقبلاً | - |
| حقن DbContext المباشر في الخدمات | ⬜ مستقبلاً | - |

### 6.2 ✅ إزالة References غير الصحيحة

**قبل:**

```xml
<!-- Api.Web.csproj -->
<ProjectReference Include="..\RubikCare.Domain\RubikCare.Domain.csproj" />
<ProjectReference Include="..\RubikCare.Infrastructure\RubikCare.Infrastructure.csproj" />
```

**بعد:**

```xml
<!-- Api.Web.csproj -->
<ProjectReference Include="..\RubikCare.Application\RubikCare.Application.csproj" />
<ProjectReference Include="..\RubikCare.Infrastructure\RubikCare.Infrastructure.csproj" />
```

**ملاحظة:** Reference `Infrastructure` بقي لضرورة `Program.cs` فقط. الحماية عبر Architecture Tests.

### 6.3 ✅ `InfrastructureExtensions.cs` — التسجيل الموحد

**المسار:** `RubikCare.Infrastructure/InfrastructureExtensions.cs`

**Extension Method وحيد يسجل كل خدمات التطبيق:**

```csharp
public static IServiceCollection AddAllInfrastructureServices(
    this IServiceCollection services, string connectionString)
{
    // Identity + Database + Services + Repositories + Use Cases
    // (كل التسجيلات في مكان واحد)
}
```

**الاستخدام في Program.cs (سطر واحد):**

```csharp
builder.Services.AddAllInfrastructureServices(connectionString);
```

### 6.4 ✅ إصلاح 16 Controller

كل Controller كان يستخدم `BusinessDbContext` مباشرة تم تحويله إلى `IGenericService<T>`:

| قبل | بعد |
|------|------|
| `BusinessDbContext _context` | `IGenericService<Entity> _service` |
| `_context.Entities.Where(...)` | `_service.GetAllAsync(...)` |
| `_context.SaveChangesAsync()` | `_service.SaveChangesAsync()` |

**قائمة Controllers التي تم إصلاحها:**
- `AuthController`
- `NotificationsController`
- `DispenseController`
- `MedicationRequestController`
- `OrganizationJobTitlesController`
- `OrganizationMembersController`
- `OrganizationMemberTitlesController`
- `OrganizationsController`
- + 8 Controllers أخرى

### 6.5 ✅ Architecture Tests

**المسار:** `RubikCare.Tests/Architecture/CleanArchitectureTests.cs`

**اختبارات تمنع كسر المعمارية:**

```csharp
[Fact]
public void Application_ShouldNot_DependOn_Infrastructure()
{
    var result = Types
        .InAssembly(applicationAssembly)
        .ShouldNot()
        .HaveDependencyOn("RubikCare.Infrastructure")
        .GetResult();

    Assert.True(result.IsSuccessful);
}
```

**النتيجة:** 33 اختبار ناجح ✅.

### 6.6 🔴 قواعد إلزامية (سارية فوراً)

#### ممنوعات مطلقة

1. **لا Controllers تستخدم `BusinessDbContext` مباشرة** — استخدم `IGenericService<T>`.
2. **لا تستخدم `using RubikCare.Infrastructure` في أي Controller**.
3. **لا حقن DbContext مباشرة في Use Cases** — استخدم Repositories.
4. **أي مخالفة = فشل في Architecture Tests = لا يمكن بناء المشروع.**

#### 🟢 أنماط إلزامية للكود الجديد

| النمط | التطبيق |
|--------|---------|
| **كل API جديد** | `IGenericService<T>` أو خدمة متخصصة من Application |
| **كل خدمة جديدة** | واجهة في Application، تنفيذ في Infrastructure |
| **كل DTO** | معرف في Application |
| **كل تسجيل** | في `InfrastructureExtensions.cs` فقط |

### 6.7 ✅ الأنماط المسموح بها للوصول للبيانات

#### النمط 1: `IGenericService<T>` (المفضل)

- للـ CRUD البسيطة.
- يستخدم في معظم الـ Controllers.

#### النمط 2: `IDbContextFactoryService` (للاستعلامات المعقدة)

- مسموح في: Controllers + Services في Infrastructure فقط.
- النمط: `_dbFactory.ExecuteReadOnlyAsync(async context => { var db = (DbContext)context; ... })`.

#### النمط 3: Service متخصص في Infrastructure

- للاستعلامات المعقدة التي تحتاج إعادة استخدام.
- يحقن فيه `IDbContextFactoryService`.
- مثال: `ClinicReportService`.

### 6.8 📦 تسجيل الخدمات

- جميع الخدمات تسجل في `InfrastructureExtensions.cs`.
- لا يوجد ملف `DependencyInjection.cs` منفصل.

### 6.9 ✅ Checklist: عند إنشاء API جديد

- [ ] هل استخدمت `IGenericService<T>` بدل `BusinessDbContext`؟
- [ ] هل التسجيل مضاف في `InfrastructureExtensions.cs`؟
- [ ] هل Architecture Tests ما زالت ناجحة؟
- [ ] هل `Program.cs` لم يتغير (سطر واحد فقط)؟

---

## 🛠️ الجزء السابع: كيفية إنشاء صفحة جديدة (خطوة بخطوة)

### 7.1 الخطوة 1: تحديد نوع الصفحة

راجع **شجرة القرار المعماري** (الجزء الأول) وحدد:
- هل هي CRUD بسيطة؟ → `IGenericService<T>`.
- هل هي عملية معقدة؟ → UseCase.
- هل هي مشتركة؟ → UseCase + API.

### 7.2 الخطوة 2: إنشاء الملفات المطلوبة

#### للصفحات البسيطة (CRUD)

```
Web/Pages/
└── Admin/
    └── Medications.razor          ← الصفحة
```

#### للصفحات المعقدة (UseCase)

```
Application/UseCases/
└── Medication/
    └── SyncCompanyMedicationsUseCase.cs  ← UseCase

Web/Pages/
└── Admin/
    └── SyncMedications.razor      ← الصفحة
```

#### للصفحات المشتركة (Web + Mobile)

```
Application/UseCases/
└── Medication/
    └── SyncCompanyMedicationsUseCase.cs  ← UseCase

Api.Web/Controllers/
└── MedicationController.cs        ← API Endpoint

Web/Pages/
└── Admin/
    └── SyncMedications.razor      ← صفحة الويب

Mobile/Pages/
└── SyncMedicationsPage.xaml       ← صفحة الموبايل
```

### 7.3 الخطوة 3: إضافة CSS (إذا لزم الأمر)

```css
/* في Shared.UI/wwwroot/css/_pages/Role/page-name.css */
.page-root {
    background: var(--rubik-body-bg);
}

@media (max-width: 992px) {
    /* تنسيقات الموبايل */
}
```

### 7.4 الخطوة 4: إضافة الاستيراد في الباندل

```css
/* في _PagesBundle.css أو الباندل المناسب */
@import url('_pages/Role/page-name.css');
```

### 7.5 الخطوة 5: اختبار الصفحة

- [ ] اختبر على Desktop (1920px).
- [ ] اختبر على Tablet (768px).
- [ ] اختبر على Mobile (375px).
- [ ] اختبر الترجمة (AR/EN).
- [ ] اختبر RTL/LTR.

---

## 📱 الجزء الثامن: دليل التجاوبية

### 8.1 المبادئ الأساسية

1. **Mobile-First:** صمم للموبايل أولاً.
2. **Touch-Friendly:** الحد الأدنى `48px × 48px`.
3. **Readable:** الحد الأدنى `14px` (Body) — `16px` (Inputs).
4. **Fast:** تجنب الصور الكبيرة.

### 8.2 Breakpoints المعيارية

| الاسم | العرض |
|-------|-------|
| **Mobile (Portrait)** | ≤ 480px |
| **Mobile (Landscape)** | 481px - 767px |
| **Tablet** | 768px - 991px |
| **Desktop** | 992px - 1199px |
| **Large Desktop** | ≥ 1200px |

### 8.3 قواعد التجاوبية

#### Touch Targets

```css
/* ❌ خطأ */
.btn { padding: 8px 16px; font-size: 12px; }

/* ✅ صحيح */
.btn { min-height: 48px; min-width: 48px; padding: 12px 24px; font-size: 14px; }
```

#### iOS Zoom Prevention

```css
/* ❌ خطأ: iOS سيكبر الصفحة */
input { font-size: 14px; }

/* ✅ صحيح */
input { font-size: 16px; }
```

---

## 🔐 الجزء التاسع: أنماط صفحات المصادقة (Auth Pages)

### 9.1 نظرة عامة

**تستخدم صفحات المصادقة (Login, Register, ForgotPassword, ResetPassword) نظام تصميم موحد يعتمد على نمط Split-Panel.**

### 9.2 Split-Panel

```
┌─────────────────────────────────────────────────────────┐
│  Visual Panel (60%)  │  Form Panel (40%)               │
│  ─────────────────   │  ──────────────                 │
│  • تدرج لوني جذاب    │  • نموذج الإدخال                │
│  • أيقونات عائمة     │  • شعار الشركة                  │
│  • خطوات العملية     │  • أزرار الإجراء                │
│  • موجات زخرفية      │  • روابط مساعدة                 │
└─────────────────────────────────────────────────────────┘
```

### 9.3 الألوان والتدرجات

**التدرج الأساسي (Visual Panel):**

```css
background: linear-gradient(135deg, #0F3B52 0%, #1B5A7A 50%, #2C7AA0 100%);
```

### 9.4 الفئات (Classes) المشتركة

- `.forgot-root` — الحاوية الرئيسية.
- `.forgot-visual` — اللوحة البصرية.
- `.forgot-form-panel` — لوحة النموذج.
- `.forgot-input-wrap` — حاوية حقل الإدخال.
- `.forgot-submit-btn` — زر الإرسال.

### 9.5 التجاوبية

```css
@media (max-width: 992px) {
    .forgot-visual { display: none; }
    
    .forgot-form-panel {
        flex: 1;
        width: 100%;
        padding: 24px 16px;
    }
    
    .forgot-input { font-size: 16px; }
}
```

### 9.6 أفضل الممارسات

1. **Touch Targets:** الحد الأدنى `48px`.
2. **iOS Zoom Prevention:** `font-size: 16px`.
3. **RTL Support:** `[dir="rtl"]` + `[dir="ltr"]`.
4. **Focus States:** إضافة `.focused` class.

---

## ✅ الجزء العاشر: CHECKLIST — قبل كتابة أي صفحة جديدة

### 10.1 التحقق من القرارات المعمارية

- [ ] **هل الصفحة جديدة؟** → استخدم `IGenericService<T>` و `DbContextFactory`.
- [ ] **هل العملية معقدة؟** → استخدم UseCase في Application.
- [ ] **هل العملية مشتركة بين Web و Mobile؟** → استخدم UseCase + API.
- [ ] **هل الصفحة قديمة وتستخدم `@inject BusinessDbContext` مباشرة؟** → خطط لترحيلها.
- [ ] **هل راجعت شجرة القرار أعلاه؟**

### 10.2 التحقق من التسجيلات

- [ ] هل الخدمة التي سأستخدمها مسجلة في `Program.cs`؟
- [ ] إذا كانت خدمة جديدة، هل أضفتها باستخدام `AddScoped` أو `AddSingleton`؟
- [ ] هل تأكدت من عدم وجود تسجيل مزدوج لنفس الخدمة؟

### 10.3 التحقق من قاعدة البيانات

- [ ] هل النموذج (Model) موجود في مجلد `Data/Models`؟
- [ ] هل تم إضافة `DbSet<MyEntity>` في `BusinessDbContext`؟
- [ ] هل تم إنشاء `Migration` جديدة بعد إضافة النموذج؟

### 10.4 التحقق من نمط الصفحة

- [ ] هل الصفحة تتبع نمط الدور الرئيسي (Clinic/Pharmacy/Rep)؟
- [ ] هل تستخدم المتغيرات الصحيحة (`--clinic-*` / `--pharmacy-*` / `--rep-*`)؟
- [ ] هل الـ Hero Section يستخدم التدرج الصحيح؟
- [ ] هل أيقونات الأزرار تستخدم الألوان الصحيحة؟
- [ ] هل تم إضافة ملف CSS إلى الباندل المناسب؟

### 10.5 التحقق من التجاوبية

- [ ] هل الصفحة تعمل على الموبايل (≤ 480px)؟
- [ ] هل الأزرار بحجم `48px` على الأقل؟
- [ ] هل حقول الإدخال `font-size: 16px`؟
- [ ] هل النصوص مقروءة بدون تكبير؟
- [ ] هل تم اختبار الصفحة على 3 أجهزة؟

### 10.6 التحقق من Shared.UI

- [ ] هل أضع **مكوناً** في `Shared.UI` وليس صفحة كاملة؟
- [ ] هل المكون لا يستخدم `DbContext` أو `ApiService` مباشرة؟
- [ ] هل المكون يستخدم `Parameters` بدلاً من `NavigationManager`؟

### 10.7 التحقق من الترجمة

- [ ] هل أضفت مفاتيح الترجمة في قاعدة البيانات مع `N'' prefix` للعربية؟
- [ ] هل تستخدم `T()` المساعدة بدلاً من `<LocalizedText />`؟

### 10.8 فحص الترجمة والاتجاه

- [ ] الصفحة **لا تحتوي** على `dir` ثابت (لا `dir="rtl"` ولا `dir="ltr"`).
- [ ] الصفحة **ترث** `dir` من `<html>`.
- [ ] عند التنقل **داخل** الصفحة → لا قفزة.
- [ ] عند **تبديل اللغة** → إما Reload كامل، أو تحديث ناجح.
- [ ] النصوص تستخدم `<LocalizedText>` أو `T("KEY")`.
- [ ] لا يوجد نص Hardcoded بالعربي/الإنجليزي.

---

## ⚠️ الجزء الحادي عشر: تحذيرات ومحاذير

| # | التحذير |
|---|---------|
| 1 | لا تغير ترتيب Middleware - `UseAuthentication` قبل `UseAuthorization` دائماً |
| 2 | لا تنسى `ServiceLifetime.Scoped` في `AddDbContextFactory` |
| 3 | استخدم `DbContextFactory` للصفحات الجديدة - لا تحقن `DbContext` مباشرة |
| 4 | سجل خدماتك قبل `builder.Build()` - أي تسجيل بعد البناء لن يعمل |
| 5 | تأكد من وجود `AddMemoryCache()` - ضروري للترجمة والقوائم |
| 6 | ⭐ **لا تستخدم `IGenericService<T>` للعمليات المعقدة** - استخدم UseCase |
| 7 | ⭐ **لا تضع منطق أعمال في Controllers** - Controllers للتنسيق فقط |
| 8 | ⭐ **Mobile لا يعتمد على أي مشروع** - التواصل عبر HTTP فقط |
| 9 | ⭐ **لا تضع صفحات كاملة في Shared.UI** - مكونات فقط |
| 10 | ⭐ **استخدم `font-size: 16px` في حقول الإدخال** - لمنع تكبير iOS |
| 11 | ⭐ **لا تعدل `AspNetUserRoles` يدوياً** - استخدم `UserRoleService` |
| 12 | ⭐ **كل خدمة جديدة في `InfrastructureExtensions.cs` فقط** |

---

## 🔗 الجزء الثاني عشر: روابط ذات صلة

- [00 — MASTER-INDEX](./00-MASTER-INDEX.md)
- [00B — PROJECT-BACKLOG](./00B-PROJECT-BACKLOG.md)
- [01 — ARCHITECTURE](./01-ARCHITECTURE.md)
- [02 — IDENTITY](./02-IDENTITY.md)
- [03 — MENUS](./03-MENUS.md)
- [04 — TRANSLATION](./04-TRANSLATION.md)
- [05 — PSP](./05-PSP.md)
- [06 — POINTS](./06-POINTS.md)
- [07 — REPORTS](./07-REPORTS.md)
- [08 — NOTIFICATIONS](./08-NOTIFICATIONS.md)
- [09 — API-GUIDE](./09-API-GUIDE.md)
- [10 — UI-STYLE-GUIDE](./10-UI-STYLE-GUIDE.md)
- [19 — PERMISSIONS](./19-PERMISSIONS.md)
- [appendix-a-glossary](./appendix-a-glossary.md)
- [appendix-b-service-index](./appendix-b-service-index.md)

---

## 📝 الجزء الثالث عشر: سجل التغييرات

| الإصدار | التاريخ | التغييرات |
|:---:|:---|:---|
| **1.0** | **18 يوليو 2026** | الإصدار الأولي — `05-page-creation-checklist.md` |
| **1.0** | **25 مايو 2026** | الإصدار الأولي — `13-clean-architecture-enforcement.md` |
| **2.0** | **17 أكتوبر 2026** | **`11-PAGE-CREATION` v2.0** — دمج `05` + `13` + إضافة architecture enforcement + إشارات |

---

**© 2026 RubikCare — للاستخدام الداخلي**

**نهاية الوثيقة ✅**
```

---

## 📌 ملاحظات على الدمج

### ✅ **ما تم الحفاظ عليه:**

| # | العنصر | من الوثيقة |
|---|--------|-----------|
| 1 | مستويات النقاء المعماري (LVL 1-4) | `05` |
| 2 | شجرة القرار المعماري | `05` |
| 3 | أنواع الصفحات الخمسة | `05` |
| 4 | مصفوفة اختيار الخدمة | `05` |
| 5 | 4 أمثلة كود | `05` |
| 6 | قواعد Shared.UI | `05` |
| 7 | دليل التجاوبية | `05` |
| 8 | أنماط Auth Pages | `05` |
| 9 | Checklist (15 نقطة) | `05` |
| 10 | 10 تحذيرات | `05` |
| 11 | حالة الإصلاحات (11 بند) | `13` |
| 12 | إزالة References | `13` |
| 13 | `InfrastructureExtensions.cs` | `13` |
| 14 | 16 Controller | `13` |
| 15 | Architecture Tests | `13` |
| 16 | 4 ممنوعات مطلقة | `13` |
| 17 | 4 أنماط إلزامية | `13` |
| 18 | 3 أنماط مسموح بها | `13` |
| 19 | Checklist API | `13` |

### ⚠️ **ما أُضيف:**

| # | الإضافة | المصدر |
|---|--------|--------|
| 1 | **قاعدة "لا تخمن أسماء الحقول"** | `10-MAUI` (من `13`) |
| 2 | **التحقق من البيانات قبل التعديل** | `10-MAUI` (من `13`) |
| 3 | **نمط `IDbContextFactoryService` في Services** | `10-MAUI` (من `13`) |
| 4 | **إشارات للوثائق الأخرى** | محدَّثة |
| 5 | **ترقيم الوثيقة `11`** | جديد |

---ACHING` صغير + مستقل.

**⏳ بانتظار قرارك.** 🚀
