# 🏛️ 00 - الهيكل المعماري الشامل لنظام RubikCare

**لمحة سريعة**: وثيقة مرجعية للبنية التحتية للمشروع.
**آخر تحديث**: 3 أكتوبر 2026
**المؤلف**: فريق RubikCare
**الحالة**: ✅ محدثة لتشمل جميع المشاريع التسعة (بما فيها PWA) + نظام النقاط

---

## 📌 مقدمة: فلسفة البناء

نظام RubikCare هو منصة صحية متكاملة، صُممت لتوحيد إدارة المرضى، ومقدمي الرعاية (أطباء، صيادلة، مندوبين)، وبرامج الدعم (PSP) في منظومة رقمية واحدة. يعتمد النظام على **Clean Architecture** لضمان فصل واضح للمسؤوليات، وقابلية عالية للتطوير، وسهولة في الصيانة والاختبار.

تمتد المنصة عبر **تسعة مشاريع برمجية**، تنقسم إلى طبقات أساسية (Core) وتطبيقات طرفية (Clients)، جميعها تشترك في مكونات UI موحدة وتتواصل عبر واجهة برمجية (API) مركزية.

---

## 🗺️ الخريطة المعمارية للمشاريع التسعة

```mermaid
graph TD
    subgraph "Core Layers"
        Domain["RubikCare.Domain<br/>Entities, Enums, Interfaces"]
        App["RubikCare.Application<br/>Use Cases, DTOs, Services"]
        Infra["RubikCare.Infrastructure<br/>DbContext, Repositories"]
    end

    subgraph "Presentation & API"
        Api["RubikCare.Api.Web<br/>Central REST API Gateway"]
    end

    subgraph "Client Applications"
        Web["RubikCare.Web<br/>Blazor Server App"]
        PWA["RubikCare.PWA<br/>Blazor WebAssembly App"]
        Mobile["RubikCare.Mobile<br/>MAUI App"]
    end

    subgraph "Shared & Testing"
        SharedUI["RubikCare.Shared.UI<br/>Razor Component Library"]
        Tests["RubikCare.Tests<br/>Unit & Integration Tests"]
    end

    App --> Domain
    Infra --> App & Domain
    Api --> App & Infra

    Web -- "SignalR / HTTP" --> Api
    PWA -- "HTTP / REST" --> Api
    Mobile -- "HTTP / REST" --> Api

    SharedUI -.-> Web & PWA & Mobile
    Tests -.-> Api & App & Infra & Domain & Web & PWA & Mobile & SharedUI
```

---

## 📊 الجدول التنفيذي للمشاريع

| المشروع | التقنية الأساسية | المسؤولية الرئيسية | يعتمد على (مباشر) |
| :--- | :--- | :--- | :--- |
| **Domain** | .NET 10 Class Library | الكيانات الأساسية، التعدادات، وواجهات النطاق المجردة. | لا يعتمد على أي مشروع. |
| **Application** | .NET 10 Class Library | منطق الأعمال، حالات الاستخدام (Use Cases)، وخدمات التطبيق. | `Domain` فقط. |
| **Infrastructure** | .NET 10, EF Core 10, SQL Server | تنفيذ واجهات الـ `Application`، الوصول للبيانات، الترحيلات (Migrations). | `Application` و `Domain`. |
| **Api.Web** | ASP.NET Core 10 | نقطة النهاية الموحدة (REST API) لجميع العملاء. | `Application` و `Infrastructure`. |
| **Web** | Blazor Server 10 | تطبيق الويب الرئيسي (تفاعلي مع SignalR). | `Api.Web` (ضمنياً) و `Shared.UI`. |
| **PWA** | Blazor WebAssembly 10 | تطبيق ويب تقدمي (نسخة كربونية من Mobile). | `Api.Web` (عبر HTTP) و `Shared.UI`. |
| **Mobile** | .NET MAUI 10 | تطبيق الهواتف الذكية (Android, iOS). | `Api.Web` (عبر HTTP) و `Shared.UI`. |
| **Shared.UI** | Razor Class Library (RCL) | مكونات Razor قابلة لإعادة الاستخدام + خدمات مشتركة. | لا يعتمد على تطبيقات محددة. |
| **Tests** | xUnit, Moq | اختبارات الوحدة والتكامل والمعمارية. | جميع المشاريع (حسب الحاجة). |

---

## 🧩 مسؤوليات كل مشروع بالتفصيل

### 1. `RubikCare.Domain` - القلب النابض 🫀

**أكثر طبقة استقراراً ونقاءً.**

لا تعرف هذه الطبقة شيئاً عن قواعد البيانات، أو واجهات المستخدم، أو حتى كيفية نقل البيانات. همها الوحيد هو تمثيل مفاهيم العمل (Business Concepts).

- **المحتويات**: الكيانات (Entities) مثل `ApplicationUser`, `Patient`, `Medication`، التعدادات (Enums)، كائنات القيمة (Value Objects) كـ `Address`، وأحداث النطاق (Domain Events).
- **إضافة حديثة**: كيانات نظام النقاط (`PSPProgramPointScheme`, `PSPProgramPointRule`, إلخ).
- **قاعدة ذهبية**: **ممنوع** إضافة أي مرجعيات لـ Entity Framework أو مكتبات خارجية متعلقة بالبيانات.

### 2. `RubikCare.Application` - عقل النظام 🧠

**هنا يُتخذ القرار.**

تحتوي على حالات الاستخدام (Use Cases) التي تمثل متطلبات المستخدم الفعلية (مثل: `EnrollPatientInPspUseCase`). تقوم بتنسيق تدفق البيانات بين الـ `Api.Web` والـ `Infrastructure`، وتطبق منطق الأعمال.

- **المحتويات**: الخدمات (Services)، واجهات للبنية التحتية (Interfaces)، كائنات نقل البيانات (DTOs)، محولات (Mappers)، ومُحققّات (Validators) باستخدام FluentValidation.
- **إضافات حديثة**:
  - `PSPEnrollmentService` — تسجيل المرضى في برامج الدعم.
  - `PSPPointSchemeService` — قراءة نظام النقاط.
- **قاعدة ذهبية**: لا تعتمد إلا على `Domain` ولا تعرف كيف أو من أين تأتي البيانات (مستودع، خدمة خارجية، إلخ).

### 3. `RubikCare.Infrastructure` - الجسر إلى العالم الخارجي 🌉

**التفاصيل التقنية هنا.**

هذا هو مكان تنفيذ الواجهات المجردة التي عرّفها الـ `Application`. مسؤول عن التواصل مع قواعد البيانات، وإرسال البريد الإلكتروني، والخدمات السحابية.

- **المحتويات**: `BusinessDbContext` (EF Core)، تنفيذ المستودعات (Repositories)، ملفات الترحيل (Migrations)، وخدمات خارجية (مثل `EmailService`).
- **إضافات حديثة**:
  - **19 Handler** لنظام النقاط في `UseCases/PSP/Points/`.
  - **`PSPEnrollmentService`** في `Services/PSP/` (يستخدم `IGenericService<T>`).
- **قاعدة ذهبية**: يُسمح لها بالاعتماد على `Application` و `Domain` فقط، ولا يُستدعى مباشرة من قبل التطبيقات العميلة (Web, PWA, Mobile).

### 4. `RubikCare.Api.Web` - بوابة النظام الموحدة 🚪

**نقطة الدخول الوحيدة للبيانات.**

هي واجهة برمجية (REST API) مركزية تخدم جميع العملاء (Web, PWA, Mobile). تتولى المصادقة (JWT)، والتحقق من الصلاحيات، وتنسيق الطلبات.

- **المحتويات**: وحدات التحكم (Controllers)، `Program.cs`، وMiddleware (معالجة الأخطاء، التسجيل).
- **Controllers حديثة**:
  - `PointSchemesController` (4 endpoints)
  - `PointRulesController` (5 endpoints)
  - `PointBalancesController` (3 endpoints)
  - `PointEarningController` (1 endpoint)
  - `PointRedeemController` (5 endpoints)
- **قاعدة ذهبية**: وحدات التحكم يجب أن تكون **رفيعة جداً** (Thin Controllers)؛ كل المنطق يُفوض إلى الخدمات في طبقة `Application` أو إلى Handlers في `Infrastructure`.

### 5. `RubikCare.Web` - التطبيق التفاعلي الغني 💻

**تطبيق Blazor Server.**

يُستخدم في بيئات الإدارة واللوحات التي تتطلب تفاعلاً فورياً. يحافظ على اتصال مستمر (SignalR) مع الخادم، مما يوفر تجربة مستخدم سلسة.

- **المحتويات**: صفحات (Pages)، مكونات خاصة بالويب، وخدمات محلية لإدارة الحالة.
- **إضافات حديثة**:
  - **`PSPStep2b_PointScheme.razor`** — واجهة إدارة نظام النقاط للشركة.
  - **قسم "نظام النقاط"** في `PSPProgramsDetails.razor`.
- **قاعدة ذهبية**: يتواصل مع الـ `Api.Web` عبر HTTP، ولا يتفاعل مباشرة مع قواعد البيانات.

### 6. `RubikCare.PWA` - التطبيق الخفيف متعدد المنصات 📱

**تطبيق Blazor WebAssembly — نسخة كربونية من MAUI.**

تمت إضافته ليكون بديلاً لتطبيق الويب (Blazor Server) على الأجهزة التي لا تتعامل بكفاءة مع SignalR (مثل أجهزة iPhone)، وأيضاً ليكون حلاً سريعاً ومستقراً للوصول إلى النظام من أي متصفح.

- **طبيعة التشغيل**: يعمل كلياً داخل متصفح المستخدم بعد تحميل ملفات `WebAssembly`. يُنشر كملفات **ثابتة** (Static Files) على خادم ويب (مثل IIS).
- **المحتويات**:
  - صفحات Razor مكافئة لـ `RubikCare.Mobile`.
  - خدمات للتواصل مع الـ API (مثل `WebApiService`).
  - `googleAuth.js` — تكامل Google Identity Services.
  - `cache-reset.html` — لتنظيف Service Worker تلقائياً.
- **إضافات حديثة**:
  - **`PointDashboard.razor`** — محفظة النقاط.
- **الرابط المرجعي**: [دليل PWA](./22-RubikCare.PWA.md).

### 7. `RubikCare.Mobile` - تجربة الهواتف الذكية 📲

**تطبيق MAUI الأصلي.**

يستخدم `BlazorWebView` لعرض مكونات `Shared.UI`، مما يسمح بإعادة استخدام كبير للكود مع تطبيقات الويب. يوفر إمكانية الوصول إلى ميزات الجهاز (الكاميرا، GPS، الإشعارات المحلية).

- **المحتويات**: صفحات XAML، ViewModels، وخدمات خاصة بالمنصة (مثل `MauiLocalNotificationService`).
- **إضافات حديثة**:
  - **`PointsDashboardPage.xaml`** — Container لعرض `PointDashboard.razor`.
- **قاعدة ذهبية**: لا يعتمد على أي مشروع آخر باستثناء `Shared.UI`؛ يتواصل مع النظام حصراً عبر `Api.Web` باستخدام HTTP.

### 8. `RubikCare.Shared.UI` - مستودع المكونات المشتركة 🧩

**اللبنة الأساسية لواجهات المستخدم.**

مكتبة تحتوي على كل المكونات البصرية والخدمات المساعدة التي تستخدمها التطبيقات العميلة الثلاثة (Web, PWA, Mobile)، مما يضمن توحيد الشكل والأداء.

- **المحتويات**:
  - 66+ مكون Razor (مثل `RubikSmartTable`, `RubikButton`).
  - 120+ ملف CSS منظم.
  - 23+ خدمة/واجهة مشتركة (مثل `ITranslationService`).
- **إضافات حديثة**:
  - **`PointDashboard.razor`** + `PointDashboardNavigationBridge.cs` + `PointModels.cs`.
  - **`point-dashboard.css`**.
- **قاعدة ذهبية**: تحتوي على **مكونات** قابلة لإعادة الاستخدام، وليس صفحات كاملة، لتجنب تعقيد التبعيات.

### 9. `RubikCare.Tests` - شبكة الأمان الجودة 🛡️

**ضمان استقرار المعمارية.**

يحتوي على اختبارات الوحدة (Unit Tests) لكل طبقة، واختبارات تكامل (Integration Tests) بين الطبقات، بالإضافة إلى اختبارات معماريّة باستخدام `NetArchTest` للتأكد من عدم كسر قواعد التبعية.

- **عدد الاختبارات**: 33 اختبار ناجح (حتى الآن).
- **قاعدة ذهبية**: جزء لا يتجزأ من عملية التطوير.

---

## 🎯 نظام نقاط برامج الدعم (PSP Program Points) — نظرة معمارية

### الموقع في Clean Architecture

```
┌─────────────────────────────────────────────────────────────┐
│                    RubikCare.Domain                          │
│  ┌───────────────────────────────────────────────────────┐  │
│  │ 8 كيانات في Entities/PSP/Points/                     │  │
│  │ 3 ملفات Constants في Constants/PSP/Points/           │  │
│  └───────────────────────────────────────────────────────┘  │
└─────────────────────────────────────────────────────────────┘
                              ↓
┌─────────────────────────────────────────────────────────────┐
│                  Infrastructure Layer                        │
│  ┌───────────────────────────────────────────────────────┐  │
│  │ 19 Handler في UseCases/PSP/Points/                   │  │
│  │ (Schemes, Rules, Balances, Earning, Redeem)          │  │
│  └───────────────────────────────────────────────────────┘  │
└─────────────────────────────────────────────────────────────┘
                              ↓
┌─────────────────────────────────────────────────────────────┐
│                    Api.Web Layer                             │
│  ┌───────────────────────────────────────────────────────┐  │
│  │ 5 Controllers (18 Endpoints)                          │  │
│  └───────────────────────────────────────────────────────┘  │
└─────────────────────────────────────────────────────────────┘
                              ↓
┌─────────────────────────────────────────────────────────────┐
│                   Client Layer                               │
│  ┌───────────────────────────────────────────────────────┐  │
│  │  Web: PSPStep2b_PointScheme.razor                     │  │
│  │  PWA: PointsDashboardPage.razor                       │  │
│  │  MAUI: PointsDashboardPage.xaml                       │  │
│  │  Shared: PointDashboard.razor                         │  │
│  └───────────────────────────────────────────────────────┘  │
└─────────────────────────────────────────────────────────────┘
```

### المكونات الأساسية

| # | المكوّن | العدد | الموقع |
|---|---------|:-----:|--------|
| 1 | **كيانات Domain** | 8 | `Domain/Entities/PSP/Points/` |
| 2 | **Constants** | 3 | `Domain/Constants/PSP/Points/` |
| 3 | **Handlers** | 19 | `Infrastructure/UseCases/PSP/Points/` |
| 4 | **Controllers** | 5 | `Api.Web/Controllers/PSP/Points/` |
| 5 | **Application Services** | 2 | `Application/Services/PSP/` |
| 6 | **Shared UI Components** | 2 | `Shared.UI/Components/PSP/Points/` |
| 7 | **جداول قاعدة البيانات** | 8 | — |
| 8 | **API Endpoints** | 18 | — |

### قاعدة ذهبية (جديدة)

```
❌ تعديل IDs في PSPActivityTypes
✅ استخدام PointActivityTypeIds

❌ استدعاء EarnPointsHandler من External
✅ استدعاؤه من Handlers أخرى

✅ IsTokenValid قبل إرسال التوكن
```

---

## 🧭 شجرة القرار المعماري (دليل المطور)

مع وجود ثلاثة تطبيقات عميلة وطبقات متعددة، قد يحتار المطور في النهج الصحيح لتطوير صفحة جديدة. هذا القسم يوفر إرشادات عملية.

### مستويات "النقاء المعماري" (من العملي إلى النظري)

| المستوى | الوصف | النهج |
| :--- | :--- | :--- |
| **LVL 1** | **مقبول للصفحات القديمة جداً** (يُنصح بالترقية). | `Blazor Page` ← `DbContext` مباشرة. |
| **LVL 2** | **مناسب للعمليات البسيطة (CRUD)**. | `Blazor Page` ← `IGenericService<T>` ← `DbContext`. |
| **LVL 3** | **الموصى به للعمليات المعقدة والجديدة**. | `Blazor/MAUI/PWA Page` ← `UseCase` (في `Application`) ← `Repository` (في `Infrastructure`). |
| **LVL 4** | **الأنقى للعمليات المشتركة بين العملاء**. | `Client (Web/PWA/Mobile)` → (HTTP) → `Api.Web` → `UseCase`. |

### القاعدة الذهبية: اختر النهج حسب التعقيد والمشاركة

| نوع الصفحة/الوظيفة | النهج الأمثل | مثال تطبيقي |
| :--- | :--- | :--- |
| **CRUD بسيط** (إضافة/تعديل/حذف لكيان واحد) | **LVL 2**: `IGenericService<T>` | عرض قائمة الأدوية، تعديل ملف المستخدم. |
| **منطق أعمال معقد** (متعدد الخطوات والشروط) | **LVL 3**: `UseCase` (في `Application`) | تسجيل مريض في برنامج دعم (PSP)، عملية مزامنة مع نظام خارجي. |
| **وظيفة مشتركة بين Web و PWA و Mobile** | **LVL 4**: `UseCase` (في `Application`) + `API` | إرسال إشعار، إنشاء تقرير، البحث المتقدم. |
| **عمليات حساسة جداً** (مالية، مصادقة) | **LVL 4**: `UseCase` + `API` مع تحقق إضافي (Validation). | معالجة الدفع، تغيير كلمة المرور. |
| **استعلامات تقارير معقدة** (قراءة فقط) | **استخدام** `DbContextFactoryService` مباشرة في الصفحة (مع الحذر). | لوحة التحكم (Dashboard) بإحصائيات متعددة. |

### ⚠️ تحذيرات معمارية صارمة

- **لا تكسر تبعية الطبقات**: ممنوع استدعاء `Infrastructure` أو `DbContext` مباشرة من `Web` أو `PWA` أو `Mobile`.
- **الـ `API` للجميع**: تطبيقات `Mobile` و `PWA` **لا تعتمد** على أي مشروع آخر (`Domain`, `Application`...)، وتتواصل فقط مع `Api.Web` عبر HTTP.
- **`Shared.UI` للمكونات**: لا تضع صفحات كاملة (مثل `Dashboard.razor`) في `Shared.UI`، بل ضعها في المشاريع العميلة نفسها (`Web`, `PWA`, `Mobile`).
- **استخدم DTOs**: لا تُرجع كائنات `Domain` (Entities) مباشرة من الـ `API`، استخدم DTOs لمنع تسرب تفاصيل النطاق وتقليل حجم البيانات.

---

## 📋 ملخص: العلاقات والتبعيات (بصيغة مبسطة)

```mermaid
flowchart LR
    subgraph Clients
        Web[Web - Blazor Server] -->|SignalR/HTTP| API
        PWA[PWA - Blazor WASM] -->|HTTP| API
        Mobile[Mobile - MAUI] -->|HTTP| API
    end

    subgraph Core
        API[Api.Web] --> App[Application]
        App --> Dom[Domain]
        Inf[Infrastructure] --> App & Dom
    end

    subgraph Shared
        UI[Shared.UI] -.-> Web & PWA & Mobile
    end
```

**خلاصة القاعدة**: السهم يشير إلى "يعتمد على". التطبيقات العميلة تعتمد على الـ API، والـ API يعتمد على الـ Application، والـ Application على الـ Domain، والـ Infrastructure على الـ Application والـ Domain. `Shared.UI` في المنتصف يستخدمه الجميع.

---

## 🔗 روابط ذات صلة (المرشد الشامل)

للتعمق في جوانب محددة، راجع الوثائق التالية:

- **[01 - Program.cs والتسجيلات الأساسية](./01-program-cs-foundation.md)**: لفهم كيفية تسجيل الخدمات والحقن.
- **[02 - نظام الهوية والمصادقة](./02-identity-system.md)**: لإدارة المستخدمين والأدوار.
- **[07 - نظام PSP](./07-psp-system.md)**: لوثائق برامج دعم المرضى.
- **[08 - خطة التطوير](./08-roadmap.md)**: لمتابعة المهام المنجزة والقادمة.
- **[09 - دليل الـ API](./09-api-guide.md)**: لمعرفة كيفية استخدام وبناء نقاط النهاية.
- **[10 - دليل تطوير MAUI](./10-maui-development-guide.md)**: لنشر وتطوير تطبيق الموبايل.
- **[11 - دليل BlazorWebView](./11-blazor-webview-guide.md)**: لأنماط MAUI.
- **[15 - نظام الترجمة](./15-translation-system.md)**: لوثائق الترجمة والاتجاه.
- **[16 - Google Sign-In](./16-google-sign-in.md)**: لإعدادات Google Login.
- **[22 - دليل RubikCare.PWA](./22-RubikCare.PWA.md)**: للنشر وحالة التطوير.
- **[24 - جرد هيكلة المشاريع](./24-project-structure-inventory.md)**: لمعرفة المكونات المنقولة والمتبقية.
- **[26 - نظام النقاط](./26-rubika-points-system.md)**: توثيق كامل لنظام النقاط.
- **[27 - الديون التقنية](./27-technical-debt.md)**: لمراجعة الديون التقنية.

---

## 📝 سجل التحديثات (Changelog)

| التاريخ | التغيير | السبب |
| :--- | :--- | :--- |
| 3 أكتوبر 2026 | **إضافة نظام النقاط (PSP Program Points)** كقسم كامل. | توثيق البنية الجديدة. |
| 3 أكتوبر 2026 | **إضافة PWA** بشكل رسمي (9 مشاريع). | التحديث من 8 إلى 9 مشاريع. |
| 3 أكتوبر 2026 | **إضافة Controllers جديدة** (5 Controllers + 18 Endpoint). | توثيق نظام النقاط. |
| 29 أغسطس 2026 | **إعادة كتابة شاملة للوثيقة**. | دمج مشروع PWA وهيكل المشاريع الجديد. |
| 18 يوليو 2026 | إضافة **شجرة القرار المعماري** و **جداول المقارنة**. | توجيه المطورين لاختيار النهج الصحيح. |

---

**خاتمة**: هذه الوثيقة هي البوصلة المعمارية لـ RubikCare. أي تغيير جوهري في الهيكل أو إضافة مشروع جديد يجب أن ينعكس هنا أولاً، لتظل المرجعية الأساسية للفريق.
````

---
