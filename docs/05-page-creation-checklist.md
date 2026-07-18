# 05 - إنشاء الصفحات والمكونات (Page Creation Checklist)

**آخر تحديث:** 18 يوليو 2026

## مقدمة

هذا الدليل يشرح كيفية إنشاء صفحات ومكونات جديدة في مشروع RubikCare، مع التركيز على **القرارات المعمارية الصحيحة** و**اختيار الخدمة المناسبة** لكل نوع من العمليات.

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

---

## 🎯 أنواع الصفحات الخمسة (⭐ جديد - 18 يوليو 2026)

| النوع | الوصف | الأمثلة | النهج |
|------|-------|---------|-------|
| **Type 1: CRUD بسيطة** | عرض/إضافة/تعديل/حذف كيان واحد | قائمة الأدوية، تعديل اسم | `IGenericService<T>` |
| **Type 2: عمليات معقدة** | منطق أعمال متعدد الخطوات | إنشاء PSP، تسجيل مريض | UseCase |
| **Type 3: مشتركة** | تُستخدم في Web و Mobile | المزامنة، الإشعارات | UseCase + API |
| **Type 4: حساسة** | عمليات مالية/أمنية | الدفع، تغيير كلمة المرور | UseCase + API + Validation |
| **Type 5: تقارير** | استعلامات قراءة معقدة | لوحة التحكم، الإحصائيات | DbContextFactoryService |

---

## 📊 مصفوفة اختيار الخدمة - مرجع اتخاذ القرار (⭐ جديد - 18 يوليو 2026)

### خدمات أساسية (ضرورية لكل صفحة)

| الخدمة | الواجهة | متى تستخدمها؟ | مثال |
|--------|---------|---------------|------|
| `GenericService<T>` | `IGenericService<T>` | **CRUD بسيط** على كيان واحد | قائمة الأدوية، تعديل اسم منظمة |
| `DbContextFactoryService` | - | **استعلامات قراءة مخصصة** معقدة | لوحة التحكم، تقارير المبيعات |
| `UserContextService` | `IUserContextService` | **معلومات المستخدم الحالي** | أي صفحة تحتاج معرفة من هو المسجل |

### خدمات الترجمة (لأي صفحة تدعم اللغتين)

| الخدمة | الواجهة | متى تستخدمها؟ | مثال |
|--------|---------|---------------|------|
| `TranslationStateService` | - | معرفة **اللغة الحالية** | أي صفحة تدعم AR/EN |
| `ILocalizationService` | `ILocalizationService` | جلب **الترجمات من قاعدة البيانات** | عرض النصوص المترجمة |
| `LayoutDirectionService` | `ILayoutDirectionService` | إدارة **اتجاه الصفحة** (RTL/LTR) | ضبط `dir` attribute |

### خدمات المستخدم والسياق

| الخدمة | الواجهة | متى تستخدمها؟ | مثال |
|--------|---------|---------------|------|
| `UserContextService` | `IUserContextService` | ID المستخدم، الأدوار، الصلاحيات | التحقق من صلاحية الوصول |
| `IUserSessionService` | `IUserSessionService` | localStorage + إدارة الجلسات | حفظ تفضيلات الجلسة |
| `IUserRoleService` | `IUserRoleService` | **إدارة الأدوار الموسعة** | إضافة/تعديل/حذف أدوار المستخدمين |

### خدمات التنقل والقوائم

| الخدمة | الواجهة | متى تستخدمها؟ | مثال |
|--------|---------|---------------|------|
| `DynamicMenuService` | - | جلب **القوائم الديناميكية** | عرض القائمة الجانبية |

### خدمات مساعدة

| الخدمة | الواجهة | متى تستخدمها؟ | مثال |
|--------|---------|---------------|------|
| `ExcelService` | `IExcelService<T>` | استيراد/تصدير Excel | تصدير قائمة مرضى |
| `IImageService` | `IImageService` | رفع ومعالجة الصور | رفع صورة البروفايل |

---

## 💡 أمثلة عملية على اختيار الخدمة الصحيحة (⭐ جديد - 18 يوليو 2026)

### ✅ مثال 1: صفحة CRUD بسيطة (عرض قائمة أدوية)

**القرار:** استخدم `IGenericService<T>`

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

**السبب:** عملية قراءة/حذف بسيطة، لا تحتاج UseCase.

---

### ✅ مثال 2: عملية معقدة (مزامنة أدوية الشركة)

**القرار:** استخدم UseCase

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

---

### ✅ مثال 3: استعلام قراءة معقد (لوحة التحكم)

**القرار:** استخدم `DbContextFactoryService`

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
            TotalPatients = await context.UserProfiles
                .CountAsync(up => up.IsActive),
            ActivePrograms = await context.PSPs
                .CountAsync(p => p.IsActive),
            RecentOrders = await context.Orders
                .OrderByDescending(o => o.CreatedDate)
                .Take(10)
                .ToListAsync()
        };
    }
}
```

**السبب:** استعلام مخصص معقد، لا يناسب `IGenericService<T>`.

---

### ✅ مثال 4: عملية مشتركة (Web + Mobile)

**القرار:** استخدم UseCase + API

```csharp
// 1. في Application/UseCases/Medication/SyncCompanyMedicationsUseCase.cs
// (نفس UseCase أعلاه)

// 2. في Api.Web/Controllers/MedicationController.cs
[HttpPost("sync")]
public async Task<IActionResult> Sync([FromBody] SyncRequest request)
{
    var result = await _syncUseCase.ExecuteAsync(request.CompanyId);
    return Ok(result);
}

// 3. في Mobile - يستدعي API
var result = await ApiService.PostAsync<SyncResult>(
    "api/medication/sync", 
    new { CompanyId = 123 });

// 4. في Web - يستدعي UseCase مباشرة (لأنه على نفس الخادم)
var result = await SyncUseCase.ExecuteAsync(123);
```

**السبب:** العملية مشتركة، نضع UseCase في Application ونستدعيها بطرق مختلفة.

---

## 🏛️ قواعد Shared.UI: مكونات vs صفحات (⭐ جديد - 18 يوليو 2026)

### ✅ ما يوضع في Shared.UI:
- **المكونات القابلة لإعادة الاستخدام** (RubikSmartTable, RubikDropdown)
- **الأقسام المشتركة** (SettingsPage, SecuritySection, ProfileHeader)
- **الخدمات المشتركة** (TranslationState, LayoutDirection)

### ❌ ما لا يوضع في Shared.UI:
- **صفحات كاملة** (Dashboard, PSP, Admin)
- **منطق أعمال** (يجب أن يكون في Application)
- **اتصالات API** (يجب أن تكون في ApiService الخاص بكل منصة)

### 💡 لماذا هذا التمييز مهم؟

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

---

## 📱 دليل التجاوبية (Responsive Design Guidelines) (⭐ جديد - 18 يوليو 2026)

### 🎯 المبادئ الأساسية

1. **Mobile-First:** صمم للموبايل أولاً، ثم وسّع للشاشات الأكبر
2. **Touch-Friendly:** الحد الأدنى للعناصر القابلة للنقر: `48px × 48px`
3. **Readable:** الحد الأدنى لحجم الخط: `14px` (Body), `16px` (Inputs)
4. **Fast:** تجنب الصور الكبيرة، استخدم `lazy loading`

### 📏 Breakpoints المعيارية

| الاسم | العرض | الاستخدام |
|-------|-------|-----------|
| **Mobile (Portrait)** | ≤ 480px | هواتف صغيرة |
| **Mobile (Landscape)** | 481px - 767px | هواتف كبيرة / تابلت صغير |
| **Tablet** | 768px - 991px | تابلت |
| **Desktop** | 992px - 1199px | شاشات متوسطة |
| **Large Desktop** | ≥ 1200px | شاشات كبيرة |

### 🎨 قواعد التجاوبية

#### 1. Touch Targets (أهداف اللمس)

```css
/* ❌ خطأ: صغير جداً */
.btn {
    padding: 8px 16px;
    font-size: 12px;
}

/* ✅ صحيح: حجم مريح */
.btn {
    min-height: 48px;
    min-width: 48px;
    padding: 12px 24px;
    font-size: 14px;
}
```

#### 2. iOS Zoom Prevention (منع التكبير التلقائي)

```css
/* ❌ خطأ: iOS سيكبر الصفحة عند التركيز */
input {
    font-size: 14px;
}

/* ✅ صحيح: يمنع التكبير */
input {
    font-size: 16px; /* الحد الأدنى لمنع التكبير في iOS */
}
```

#### 3. Responsive Typography (خطوط متجاوبة)

```css
/* Desktop */
h1 { font-size: 32px; }
h2 { font-size: 24px; }
p { font-size: 16px; }

/* Tablet */
@media (max-width: 992px) {
    h1 { font-size: 28px; }
    h2 { font-size: 20px; }
    p { font-size: 15px; }
}

/* Mobile */
@media (max-width: 480px) {
    h1 { font-size: 24px; }
    h2 { font-size: 18px; }
    p { font-size: 14px; }
}
```

---

## 🔐 أنماط صفحات المصادقة (Auth Pages) (⭐ جديد - 18 يوليو 2026)

### نظرة عامة

تستخدم صفحات المصادقة (Login, Register, ForgotPassword, ResetPassword) نظام تصميم موحد يعتمد على نمط **Split-Panel** (لوحة بصرية + لوحة نموذج).

### 🏗️ نمط Split-Panel

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

### 🎨 الألوان والتدرجات

#### التدرج الأساسي (Visual Panel)

```css
background: linear-gradient(135deg, #0F3B52 0%, #1B5A7A 50%, #2C7AA0 100%);
```

### 📐 الفئات (Classes) المشتركة

- `.forgot-root` - الحاوية الرئيسية
- `.forgot-visual` - اللوحة البصرية
- `.forgot-form-panel` - لوحة النموذج
- `.forgot-input-wrap` - حاوية حقل الإدخال
- `.forgot-submit-btn` - زر الإرسال

### 📱 التجاوبية (Responsive Design)

```css
@media (max-width: 992px) {
    .forgot-visual {
        display: none; /* إخفاء اللوحة البصرية */
    }
    
    .forgot-form-panel {
        flex: 1;
        width: 100%;
        padding: 24px 16px;
    }
    
    .forgot-input {
        font-size: 16px; /* يمنع التكبير التلقائي في iOS */
    }
}
```

---

## ✅ CHECKLIST: قبل كتابة أي صفحة جديدة (⭐ محدّث - 18 يوليو 2026)

### التحقق من القرارات المعمارية (جديد)

- [ ] **هل الصفحة جديدة؟** → استخدم `IGenericService<T>` و `DbContextFactory`
- [ ] **هل العملية معقدة؟** → استخدم UseCase في Application
- [ ] **هل العملية مشتركة بين Web و Mobile؟** → استخدم UseCase + API
- [ ] **هل الصفحة قديمة وتستخدم `@inject BusinessDbContext` مباشرة؟** → خطط لترحيلها
- [ ] **هل راجعت شجرة القرار أعلاه لاختيار الخدمة الصحيحة؟**

### التحقق من التسجيلات

- [ ] هل الخدمة التي سأستخدمها مسجلة في `Program.cs`؟
- [ ] إذا كانت خدمة جديدة، هل أضفتها باستخدام `AddScoped` أو `AddSingleton`؟
- [ ] هل تأكدت من عدم وجود تسجيل مزدوج لنفس الخدمة؟

### التحقق من قاعدة البيانات

- [ ] هل النموذج (Model) موجود في مجلد `Data/Models`؟
- [ ] هل تم إضافة `DbSet<MyEntity>` في `BusinessDbContext`؟
- [ ] هل تم إنشاء `Migration` جديدة بعد إضافة النموذج؟

### التحقق من نمط الصفحة

- [ ] هل الصفحة تتبع نمط الدور الرئيسي (Clinic/Pharmacy/Rep)؟
- [ ] هل تستخدم المتغيرات الصحيحة (`--clinic-*` / `--pharmacy-*` / `--rep-*`)?
- [ ] هل الـ Hero Section يستخدم التدرج الصحيح؟
- [ ] هل أيقونات الأزرار تستخدم الألوان الصحيحة؟
- [ ] هل تم إضافة ملف CSS إلى الباندل المناسب؟

### ⭐ التحقق من التجاوبية (جديد - 18 يوليو 2026)

- [ ] هل الصفحة تعمل على الموبايل (≤ 480px)؟
- [ ] هل الأزرار بحجم `48px` على الأقل؟
- [ ] هل حقول الإدخال `font-size: 16px` (لمنع تكبير iOS)؟
- [ ] هل النصوص مقروءة بدون تكبير؟
- [ ] هل تم اختبار الصفحة على 3 أجهزة (Mobile, Tablet, Desktop)؟

### ⭐ التحقق من صفحات المصادقة (جديد - 18 يوليو 2026)

- [ ] هل تستخدم `.forgot-*` classes المشتركة؟
- [ ] هل Visual Panel يُخفى على الموبايل (`@media (max-width: 992px)`)?
- [ ] هل Form Panel يملأ الشاشة على الموبايل؟
- [ ] هل تم إضافة Media Queries للتجاوبية؟

### التحقق من Shared.UI (جديد)

- [ ] هل أضع **مكوناً** في Shared.UI وليس صفحة كاملة؟
- [ ] هل المكون لا يستخدم `DbContext` أو `ApiService` مباشرة؟
- [ ] هل المكون يستخدم `Parameters` بدلاً من `NavigationManager`؟

### التحقق من الترجمة

- [ ] هل أضفت مفاتيح الترجمة في قاعدة البيانات مع `N'' prefix` للعربية؟
- [ ] هل تستخدم `T()` المساعدة بدلاً من `<LocalizedText />`؟

---

## 🛠️ كيفية إنشاء صفحة جديدة (خطوة بخطوة)

### الخطوة 1: تحديد نوع الصفحة

راجع **شجرة القرار المعماري** أعلاه وحدد:
- هل هي CRUD بسيطة؟ → `IGenericService<T>`
- هل هي عملية معقدة؟ → UseCase
- هل هي مشتركة؟ → UseCase + API

### الخطوة 2: إنشاء الملفات المطلوبة

#### للصفحات البسيطة (CRUD):
```
Web/Pages/
└── Admin/
    └── Medications.razor          ← الصفحة
```

#### للصفحات المعقدة (UseCase):
```
Application/UseCases/
└── Medication/
    └── SyncCompanyMedicationsUseCase.cs  ← UseCase

Web/Pages/
└── Admin/
    └── SyncMedications.razor      ← الصفحة
```

#### للصفحات المشتركة (Web + Mobile):
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

### الخطوة 3: إضافة CSS (إذا لزم الأمر)

```css
/* في Shared.UI/wwwroot/css/_pages/Role/page-name.css */
.page-root {
    /* استخدم المتغيرات */
    background: var(--rubik-body-bg);
}

/* أضف Media Queries للتجاوبية */
@media (max-width: 992px) {
    /* تنسيقات الموبايل */
}
```

### الخطوة 4: إضافة الاستيراد في الباندل

```css
/* في _PagesBundle.css أو الباندل المناسب */
@import url('_pages/Role/page-name.css');
```

### الخطوة 5: اختبار الصفحة

- [ ] اختبر على Desktop (1920px)
- [ ] اختبر على Tablet (768px)
- [ ] اختبر على Mobile (375px)
- [ ] اختبر الترجمة (AR/EN)
- [ ] اختبر RTL/LTR

---

## ⚠️ تحذيرات ومحاذير (⭐ محدّث - 18 يوليو 2026)

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

---

## 🔗 روابط ذات صلة

- [00 - الهيكل المعماري](00-architecture-overview.md) - يحتوي على شجرة القرار المعماري الكاملة ⭐
- [01 - Program.cs والتسجيلات الأساسية](01-program-cs-foundation.md) - يحتوي على مصفوفة اختيار الخدمة ⭐
- [02 - نظام الهوية والمصادقة](02-identity-system.md)
- [03 - دليل الأنماط](03-style-guide.md) - يحتوي على أنماط المصادقة والتجاوبية ⭐
- [09 - دليل API](09-api-guide.md)
- [الملحق ب - فهرس الخدمات](../appendix-b-service-index.md)

---

**آخر تحديث:** 18 يوليو 2026 | **الملف:** `05-page-creation-checklist.md`
```

---

## 📊 ملخص التحديثات في هذه الوثيقة

| القسم | التغيير |
|-------|---------|
| **المحتوى الأصلي** | ✅ محفوظ بالكامل (إنشاء الصفحات، الترجمة، المكونات الذكية) |
| **🧭 شجرة القرار المعماري** | ✅ إضافة كاملة مع Mermaid Diagram |
| **🎯 أنواع الصفحات الخمسة** | ✅ تصنيف واضح مع أمثلة |
| **📊 مصفوفة اختيار الخدمة** | ✅ تحديث شامل مع عمود "متى تستخدمها؟" |
| **💡 أمثلة عملية** | ✅ 4 أمثلة كود جاهزة لكل حالة |
| **🏛️ قواعد Shared.UI** | ✅ مكونات vs صفحات + رسم توضيحي |
| **📱 دليل التجاوبية** | ✅ Breakpoints, Touch Targets, iOS Zoom Prevention |
| **🔐 أنماط المصادقة** | ✅ Split-Panel, `.forgot-*` classes, Media Queries |
| **✅ CHECKLIST محدّث** | ✅ إضافة 15 نقطة تحقق جديدة |
| **🛠️ كيفية إنشاء صفحة** | ✅ خطوات عملية مفصلة |
| **⚠️ تحذيرات** | ✅ إضافة 5 تحذيرات جديدة |

---
