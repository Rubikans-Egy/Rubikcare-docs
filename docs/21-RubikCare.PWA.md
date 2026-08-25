# 📘 وثيقة مشروع RubikCare.PWA

**آخر تحديث:** 25 أغسطس 2026  
**الحالة:** مرحلة التطوير النشط (Alpha)  
**الإصدار:** 0.1.0

---

## 🎯 1. المقدمة والهدف

### ما هو مشروع RubikCare.PWA؟

**RubikCare.PWA** هو تطبيق ويب تقدمي (Progressive Web App) مبني بتقنية **Blazor WebAssembly** في إطار عمل **.NET 10**. يمثل هذا المشروع **العميل الثالث** في منظومة RubikCare، إلى جانب:

| المشروع | التقنية | الجمهور المستهدف |
|---------|---------|------------------|
| **RubikCare.Web** | Blazor Server | الإدارة والموظفون (داخلي) |
| **RubikCare.Mobile** | .NET MAUI + BlazorWebView | المرضى والمهنيون (موبايل أصلي) |
| **RubikCare.PWA** ⭐ | Blazor WebAssembly | المرضى والمهنيون (ويب متقدم) |

### الأهداف الاستراتيجية

1. **تجربة موحدة:** تقديم نفس تجربة المستخدم الموجودة في تطبيق الموبايل (MAUI) ولكن على الويب
2. **قابلية التثبيت:** إمكانية تثبيت التطبيق على الشاشة الرئيسية للأجهزة (iOS/Android/Desktop)
3. **العمل دون اتصال:** استخدام Service Worker للعمل في ظروف الشبكة الضعيفة
4. **أداء عالي:** تحميل سريع وتفاعل فوري بفضل WebAssembly
5. **إعادة استخدام المنطق:** مشاركة DTOs وخدمات الترجمة مع المشاريع الأخرى عبر `Shared.UI`

---

## 🏗️ 2. البنية المعمارية والربط مع الحل

### الموقع في Clean Architecture

```
┌─────────────────────────────────────────────────────────┐
│                    RubikCare.Api.Web                     │
│              (نقطة الدخول الوحيدة للبيانات)              │
─────────────────────┬───────────────────────────────────┘
                      │ HTTP/REST + JWT
        ┌─────────────┼─────────────┐
        ▼             ▼             ▼
   ┌────────   ┌──────────┐   ──────────┐
   │  Web   │   │  Mobile  │   │   PWA ⭐ │
   │Server  │   │  (MAUI)  │   │ (WASM)   │
   ────────┘   └──────────┘   └──────────┘
        │             │             │
        └─────────────┼─────────────┘
                      ▼
              ┌───────────────┐
              │  Shared.UI    │
              │  (RCL)        │
              └───────────────┘
```

### التبعيات المسموحة (Dependency Rules)

```
RubikCare.PWA
├── ✅ يعتمد على: RubikCare.Shared.UI (مكونات فقط)
├── ✅ يعتمد على: RubikCare.Application.DTOs (عبر Shared.UI)
── ❌ لا يعتمد على: Infrastructure
├── ❌ لا يعتمد على: Domain مباشرة
└── ❌ لا يعتمد على: Api.Web (يتواصل عبر HTTP فقط)
```

### ملفات الربط الحرجة

#### `Program.cs` - نقطة التسجيل المركزية

```csharp
// 1. تسجيل HttpClient للاتصال بـ Api.Web
builder.Services.AddScoped(sp => new HttpClient
{
    BaseAddress = new Uri(builder.Configuration["ApiBaseUrl"] 
        ?? "https://localhost:5235/")
});

// 2. تسجيل خدمات PWA المخصصة
builder.Services.AddScoped<IApiService, WebApiService>();
builder.Services.AddScoped<ISharedTranslationService, WebTranslationService>();
builder.Services.AddSingleton<ISharedTranslationState, SharedTranslationState>();

// 3. تسجيل الخدمات المشتركة من Shared.UI
builder.Services.AddScoped<CurrentOrganizationState>();
```

#### `wwwroot/index.html` - نقطة دخول المتصفح

```html
<!-- ⚠️ ترتيب التحميل مهم جداً -->
<link href="_content/RubikCare.Shared.UI/css/main.css" rel="stylesheet" />
<link href="RubikCare.PWA.styles.css" rel="stylesheet" />

<!-- Blazor WebAssembly Engine -->
<script src="_framework/blazor.webassembly.js"></script>

<!-- Service Worker (للقدرات التقدمية) -->
<script>navigator.serviceWorker.register('service-worker.js');</script>
```

---

## 🔧 3. الإصلاحات المعمارية الرئيسية

### 3.1 فصل Layouts حسب السياق

**المشكلة السابقة:** استخدام `MainLayout` (الذي يحتوي على Sidebar و TopBar) لصفحات المصادقة.

**الحل:** إنشاء `EmptyLayout` للصفحات التي لا تحتاج شريط تنقل:

```razor
// EmptyLayout.razor
@inherits LayoutComponentBase
@Body  // يعرض المحتوى فقط بدون قوائم
```

**التطبيق:**
```razor
@page "/login"
@layout EmptyLayout  // ← يضمن ملء الشاشة بالكامل
```

### 3.2 معالجة الترجمات بشكل آمن

**المشكلة السابقة:** استخدام `.ToDictionary()` الذي يفشل عند تكرار المفاتيح.

**الحل:** استخدام حلقة `foreach` مع `Task.WhenAll` للأداء:

```csharp
private async Task LoadTranslations()
{
    _translations.Clear();
    
    var tasks = new[]
    {
        TranslationService.GetPageTranslationsAsync("LOGIN", _currentLang),
        TranslationService.GetPageTranslationsAsync("COMMON", _currentLang)
    };
    
    var results = await Task.WhenAll(tasks);
    
    foreach (var dict in results)
    {
        foreach (var kvp in dict)
            _translations[kvp.Key] = kvp.Value;  // آمن من التكرار
    }
}
```

### 3.3 Auth Guard في MainLayout

**المشكلة السابقة:** الوصول إلى الصفحات المحمية بدون توكن.

**الحل:** التحقق من `localStorage` في `OnInitializedAsync`:

```csharp
protected override async Task OnInitializedAsync()
{
    var token = await JS.InvokeAsync<string>("localStorage.getItem", "auth_token");
    
    if (string.IsNullOrEmpty(token))
    {
        Navigation.NavigateTo("/login", true);
        return;
    }
}
```

---

## ✅ 4. الأنماط الناجحة (Best Practices)

### 4.1 إدارة حالة Razor Expressions

**النمط الصحيح:** استخدام دوال منفصلة بدلاً من Lambda expressions المعقدة

```razor
<!-- ✅ صحيح -->
<button @onclick="GoToLogin">تسجيل الدخول</button>

@code {
    private void GoToLogin() => Navigation.NavigateTo("/login");
}

<!-- ❌ خطأ -->
<button @onclick="() => Navigation.NavigateTo("/login")">
<!-- السبب: تعارض علامات التنصيص -->
```

### 4.2 Scoped CSS للتنسيقات الخاصة

**النمط الصحيح:** استخدام `*.razor.css` للتنسيقات الفريدة

```
Pages/
├── Auth/
│   ├── Login.razor
│   └── Login.razor.css  ← تنسيقات خاصة بـ Login فقط
└── Dashboard.razor
```

**القاعدة:** 
- ✅ استخدم Scoped CSS للتنسيقات الفريدة
- ✅ استخدم `Shared.UI` للتنسيقات المشتركة (مثل `.btn-primary`)
- ❌ لا تستخدم inline styles (`style="..."`)

### 4.3 تحميل الترجمات بالتوازي

```csharp
// ✅ أداء عالي: 3 استدعاءات بالتوازي (~200ms)
var task1 = TranslationService.GetPageTranslationsAsync("LOGIN", lang);
var task2 = TranslationService.GetPageTranslationsAsync("COMMON", lang);
var task3 = TranslationService.GetPageTranslationsAsync("AUTH", lang);

await Task.WhenAll(task1, task2, task3);

// ❌ أداء منخفض: 3 استدعاءات متتالية (~600ms)
var t1 = await TranslationService.GetPageTranslationsAsync("LOGIN", lang);
var t2 = await TranslationService.GetPageTranslationsAsync("COMMON", lang);
var t3 = await TranslationService.GetPageTranslationsAsync("AUTH", lang);
```

### 4.4 استخدام المتغيرات العالمية للألوان

```css
/* ✅ صحيح */
.btn-primary {
    background: var(--rubik-primary);  /* #1B5A7A */
    color: white;
}

/* ❌ خطأ */
.btn-primary {
    background: #1B5A7A;  /* لون ثابت - صعب الصيانة */
}
```

### 4.5 التحقق من وجود الملفات قبل الاستخدام

```csharp
// ✅ آمن
<img src="/Images/logo.png" onerror="this.style.display='none'" />

// ❌ يسبب أخطاء لا نهائية
<img src="/Images/logo.png" onerror="this.src='https://external.com/fallback.png'" />
```

---

## 🚫 5. الأنماط المحظورة (Anti-Patterns)

### 5.1 خلط أنماط المشاريع الثلاثة

**المشكلة:** استخدام تصميم MAUI (بطاقة بيضاء على خلفية متدرجة) في Blazor Server (Split-Panel).

**القاعدة:** 
- ✅ PWA يستخدم نفس نمط MAUI (لأنه موبايل في المتصفح)
- ✅ Web (Blazor Server) يستخدم Split-Panel
- ❌ لا تخلط النمطين في نفس المشروع

### 5.2 استخدام `@keyframes` بدون هروب

```css
/* ❌ خطأ: Razor يفسر @ كمؤشر لكود C# */
@keyframes spin { to { transform: rotate(360deg); } }

/* ✅ صحيح: استخدام @@ للهروب */
@@keyframes spin { to { transform: rotate(360deg); } }
```

### 5.3 تعديل ملفات Migration يدوياً

**القاعدة الذهبية:**
```bash
# ✅ صحيح: استخدام أوامر EF Core
Add-Migration AddNewField
Update-Database

# ❌ ممنوع: تعديل ملفات Migration يدوياً
# السبب: يكسر تتبع EF Core ويسبب أخطاء في الإنتاج
```

### 5.4 تجاهل Service Worker في التطوير

**المشكلة:** المتصفح يخزن الملفات القديمة بقوة.

**الحل أثناء التطوير:**
```html
<!-- علّق هذا السطر في index.html -->
<!-- <script>navigator.serviceWorker.register('service-worker.js');</script> -->
```

**أو استخدم Hard Refresh:** `Ctrl + Shift + R` (Windows) / `Cmd + Shift + R` (Mac)

### 5.5 استخدام `?.` مع EventCallback

```csharp
// ❌ خطأ: EventCallback هو struct وليس nullable
OnNavigate?.Invoke();

// ✅ صحيح
await OnNavigate.InvokeAsync();
```

---

## 📋 6. Checklist عملي قبل Commit

### التحقق من البنية
- [ ] هل الصفحة تستخدم Layout الصحيح (EmptyLayout للمصادقة، MainLayout للباقي)؟
- [ ] هل ملفات CSS مسجلة في `index.html` أو كـ Scoped CSS؟
- [ ] هل الترجمات تُحمّل بالتوازي (`Task.WhenAll`)؟

### التحقق من الأداء
- [ ] هل عدد استدعاءات API في الصفحة الواحدة ≤ 5؟
- [ ] هل يتم استخدام `CancellationToken` للعمليات الطويلة؟
- [ ] هل Service Worker معطل في التطوير؟

### التحقق من الأمان
- [ ] هل Auth Guard موجود في MainLayout؟
- [ ] هل التوكن يُحفظ في `localStorage` وليس في Cookie؟
- [ ] هل الصفحات الحساسة تتحقق من الدور (Role)؟

### التحقق من الترجمة
- [ ] هل جميع النصوص تستخدم `T("KEY")` بدلاً من نص ثابت؟
- [ ] هل المفاتيح موجودة في جدول `Resources`؟
- [ ] هل يدعم التطبيق تغيير اللغة ديناميكياً؟

### التحقق من التجاوبية
- [ ] هل الصفحة تعمل على عرض 375px (iPhone SE)؟
- [ ] هل الأزرار بحجم ≥ 48px؟
- [ ] هل حقول الإدخال `font-size: 16px` (لمنع تكبير iOS)؟

---

## 🔗 7. روابط مرجعية

| الوثيقة | المحتوى |
|---------|---------|
| `00-architecture-overview.md` | الهيكل المعماري وشجرة القرار |
| `01-program-cs-foundation.md` | التسجيلات الأساسية ومصفوفة الخدمات |
| `02-identity-system.md` | نظام الهوية والمصادقة |
| `03-style-guide.md` | دليل الأنماط والألوان |
| `appendix-b-service-index.md` | فهرس الخدمات المتاحة |

---

## 📝 8. الدروس المستفادة (Lessons Learned)

### 8.1 لا تفترض، بل تحقق

**الخطأ:** افتراض أن مفاتيح الترجمة غير موجودة لأنها لم تظهر.

**الصحيح:** فتح Console المتصفح وقراءة رسالة الخطأ الفعلية.

### 8.2 ابدأ بالبسيط، ثم تعقّد

**الخطأ:** بناء Dashboard كاملة مع كل الإحصائيات من البداية.

**الصحيح:** بناء Dashboard مؤقتة بسيطة، ثم إضافة الميزات تدريجياً.

### 8.3 اختبر على الجهاز الحقيقي

**الخطأ:** الاعتماد فقط على Chrome DevTools للمحاكاة.

**الصحيح:** الاختبار على iPhone و Android حقيقيين لاكتشاف مشاكل اللمس والأداء.

### 8.4 وثّق قبل أن تنسى

**الخطأ:** حل المشكلة ونسيان سببها.

**الصحيح:** كتابة ملاحظة في الكود أو تحديث هذه الوثيقة فوراً.

---

## 🎯 9. الخطوات التالية (Roadmap)

### المرحلة 2: Dashboard الكاملة (الأسبوع القادم)
- [ ] تحويل `DashboardPage.xaml` من MAUI إلى `Dashboard.razor`
- [ ] عرض الإحصائيات الحقيقية (TodayDoses, PendingDoses)
- [ ] Quick Actions (بحث طبيب، صيدلية، PSP)

### المرحلة 3: ربط القائمة الجانبية بالبيانات
- [ ] استبدال البيانات الثابتة بـ `UserSessionService`
- [ ] عرض المنظمات الحقيقية للمستخدم
- [ ] تبديل الأوضاع (Personal, Doctor, Pharmacist, Rep)

### المرحلة 4: القدرات التقدمية (PWA Features)
- [ ] تثبيت التطبيق على الشاشة الرئيسية
- [ ] العمل دون اتصال (Offline Mode)
- [ ] إشعارات Push Notifications

### المرحلة 5: الاختبار والنشر
- [ ] اختبارات وحدة (Unit Tests) للخدمات
- [ ] اختبارات تكامل (Integration Tests) للـ API
- [ ] نشر على بيئة UAT

---

## ✍️ المساهمون

| الاسم | الدور | المساهمة |
|-------|-------|----------|
| [اسمك] | Lead Developer | البنية التحتية، المصادقة، الترجمة |
| [قريباً] | UI/UX Designer | تصميم Dashboard |
| [قريباً] | QA Engineer | اختبارات الأداء والأمان |

---

**ملاحظة:** هذه وثيقة حية (Living Document) - يتم تحديثها باستمرار مع تقدم المشروع.

**آخر مراجعة:** 25 أغسطس 2026  
**المراجعة التالية:** بعد إكمال Dashboard الكاملة
