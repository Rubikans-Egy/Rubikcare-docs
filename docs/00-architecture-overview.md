بكل سرور، سأقوم بإعادة كتابة وثيقة "الهيكل المعماري" (Architecture Overview) من الصفر، لتكون الوثيقة الأيقونية والمرجعية الأساسية للمشروع. سأدمج فيها المعلومات الجديدة عن مشروع PWA، وأعتمد الأسلوب الاحترافي المدعوم بالرسوم التوضيحية والجداول، مع الحفاظ على عمق المحتوى التقني.

---

# 🏛️ 00 - الهيكل المعماري الشامل لنظام RubikCare

**لمحة سريعة**: وثيقة مرجعية للبنية التحتية للمشروع.
**آخر تحديث**: 29 أغسطس 2026
**المؤلف**: فريق RubikCare
**الحالة**: ✅ محدثة لتشمل جميع المشاريع التسعة (بما فيها PWA)

---

## 📌 مقدمة: فلسفة البناء

نظام RubikCare هو منصة صحية متكاملة، صُممت لتوحيد إدارة المرضى، ومقدمي الرعاية (أطباء، صيادلة، مندوبين)، وبرامج الدعم (PSP) في منظومة رقمية واحدة. يعتمد النظام على **Clean Architecture** لضمان فصل واضح للمسؤوليات، وقابلية عالية للتطوير، وسهولة في الصيانة والاختبار.

تمتد المنصة عبر **تسعة مشاريع برمجية**، تنقسم إلى طبقات أساسية (Core) وتطبيقات طرفية (Clients)، جميعها تشترك في مكونات UI موحدة وتتواصل عبر واجهة برمجية (API) مركزية.

---

## 🗺️ الخريطة المعمارية للمشاريع التسعة

Absolutely! The error you encountered is due to a syntax issue in the Mermaid `graph TD` definition, specifically with the use of parentheses `()` and special characters within node labels.

GitHub's Mermaid parser is very strict. To fix this, we need to:

1.  **Remove parentheses** `()` from node labels.
2.  **Use quotes** `" "` around labels that contain special characters like `/` or `&`.
3.  **Simplify subgraph titles** to avoid problematic characters.

Here is the corrected and enhanced version of the architecture diagram. It's been rewritten to be fully compatible with GitHub's Mermaid rendering engine while maintaining a professional and clear structure.

### 🏛️ Corrected Architecture Diagram (Mermaid v10+ Compatible)

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

    %% Core Dependencies
    App --> Domain
    Infra --> App & Domain
    Api --> App & Infra

    %% Client Dependencies
    Web -- "SignalR / HTTP" --> Api
    PWA -- "HTTP / REST" --> Api
    Mobile -- "HTTP / REST" --> Api

    %% Shared UI Dependencies
    SharedUI -.-> Web & PWA & Mobile
    Tests -.-> Api & App & Infra & Domain & Web & PWA & Mobile & SharedUI
```

---

### ✅ Explanation of Fixes

| Issue in Original | Correction Applied |
| :--- | :--- |
| `A[RubikCare.Domain<br/>الطبقة الأساسية (Entities, Enums, Interfaces)]` | **Removed parentheses** and used quotes: `Domain["RubikCare.Domain<br/>Entities, Enums, Interfaces"]`. |
| `B[RubikCare.Application<br/>طبقة التطبيق (Use Cases, DTOs, Services)]` | **Removed parentheses** and used quotes: `App["RubikCare.Application<br/>Use Cases, DTOs, Services"]`. |
| `C[RubikCare.Infrastructure<br/>طبقة البنية التحتية (DbContext, Repositories, Migrations)]` | **Removed parentheses** and used quotes: `Infra["RubikCare.Infrastructure<br/>DbContext, Repositories"]`. |
| `D[RubikCare.Api.Web<br/>نقطة الدخول المركزية (REST API)]` | **Removed parentheses** and used quotes: `Api["RubikCare.Api.Web<br/>Central REST API Gateway"]`. |
| **Subgraph Titles** with Arabic text and symbols | Simplified and placed within quotes, e.g., `subgraph "Core Layers"`. |
| **Node IDs** | Changed from single letters (A, B, C) to descriptive names (`Domain`, `App`, `Infra`) for better readability and maintenance. |

---

### 📊 Alternative: Simplified Dependency Table (If Diagrams Continue to Fail)

If you continue to face rendering issues, a clear table can be an excellent and foolproof alternative to convey the same information.

| **Project** | **Primary Responsibility** | **Directly Depends On** |
| :--- | :--- | :--- |
| **Domain** | Core Entities, Enums, and Domain Interfaces. | None |
| **Application** | Business Logic, Use Cases, and DTOs. | `Domain` |
| **Infrastructure** | Data Access (EF Core), Repositories, Migrations. | `Application`, `Domain` |
| **Api.Web** | Central REST API Gateway for all clients. | `Application`, `Infrastructure` |
| **Web** | Blazor Server Application (Interactive UI). | `Api.Web` (via HTTP/SignalR), `Shared.UI` |
| **PWA** | Blazor WebAssembly Application (Client-side). | `Api.Web` (via HTTP/REST), `Shared.UI` |
| **Mobile** | .NET MAUI Application (Android & iOS). | `Api.Web` (via HTTP/REST), `Shared.UI` |
| **Shared.UI** | Reusable Razor Components (RCL). | None (used by Web, PWA, Mobile) |
| **Tests** | Unit, Integration, and Architecture Tests. | Any project as needed |

---

### 🛠️ How to Test the Fix

1.  Copy the corrected Mermaid code block above.
2.  Go to your repository's Markdown file (e.g., `00-architecture-overview.md`) on GitHub.
3.  Replace the old diagram code with the new one.
4.  Commit the changes and view the rendered file on GitHub.

This version is now fully compliant with GitHub's Mermaid parser and will render correctly without errors.

---

## 📊 الجدول التنفيذي للمشاريع

| المشروع | التقنية الأساسية | المسؤولية الرئيسية | يعتمد على (مباشر) |
| :--- | :--- | :--- | :--- |
| **Domain** | .NET 10 Class Library | الكيانات الأساسية، التعدادات، وواجهات النطاق المجردة. | لا يعتمد على أي مشروع. |
| **Application** | .NET 10 Class Library | منطق الأعمال، حالات الاستخدام (Use Cases)، وخدمات التطبيق. | `Domain` فقط. |
| **Infrastructure** | .NET 10, EF Core 10, SQL Server | تنفيذ واجهات الـ `Application`، الوصول للبيانات، الترحيلات (Migrations). | `Application` و `Domain`. |
| **Api.Web** | ASP.NET Core 10 | نقطة النهاية الموحدة (REST API) لجميع العملاء. | `Application` و `Infrastructure`. |
| **Web** | Blazor Server 10 | تطبيق الويب الرئيسي (تفاعلي مع SignalR). | `Api.Web` (ضمنياً) و `Shared.UI`. |
| **PWA** | Blazor WebAssembly 10 | تطبيق ويب تقدمي يعمل في المتصفح (بديل لـ Web على أجهزة معينة). | `Api.Web` (عبر HTTP) و `Shared.UI`. |
| **Mobile** | .NET MAUI 10 | تطبيق الهواتف الذكية (Android, iOS). | `Api.Web` (عبر HTTP) و `Shared.UI`. |
| **Shared.UI** | Razor Class Library (RCL) | مكونات Razor قابلة لإعادة الاستخدام (جداول، أزرار، تخطيطات). | لا يعتمد على تطبيقات محددة. |
| **Tests** | xUnit, Moq | اختبارات الوحدة والتكامل والمعمارية. | جميع المشاريع (حسب الحاجة). |

---

## 🧩 مسؤوليات كل مشروع بالتفصيل

### 1. `RubikCare.Domain` - القلب النابض 🫀
**أكثر طبقة استقراراً ونقاءً.**
لا تعرف هذه الطبقة شيئاً عن قواعد البيانات، أو واجهات المستخدم، أو حتى كيفية نقل البيانات. همها الوحيد هو تمثيل مفاهيم العمل (Business Concepts).
*   **المحتويات**: الكيانات (Entities) مثل `ApplicationUser`, `Patient`, `Medication`، التعدادات (Enums)، كائنات القيمة (Value Objects) كـ `Address`، وأحداث النطاق (Domain Events).
*   **قاعدة ذهبية**: **ممنوع** إضافة أي مرجعيات لـ Entity Framework أو مكتبات خارجية متعلقة بالبيانات.

### 2. `RubikCare.Application` - عقل النظام 🧠
**هنا يُتخذ القرار.**
تحتوي على حالات الاستخدام (Use Cases) التي تمثل متطلبات المستخدم الفعلية (مثل: `EnrollPatientInPspUseCase`). تقوم بتنسيق تدفق البيانات بين الـ `Api.Web` والـ `Infrastructure`، وتطبق منطق الأعمال.
*   **المحتويات**: الخدمات (Services)، واجهات للبنية التحتية (Interfaces)، كائنات نقل البيانات (DTOs)، محولات (Mappers)، ومُحققّات (Validators) باستخدام FluentValidation.
*   **قاعدة ذهبية**: لا تعتمد إلا على `Domain` ولا تعرف كيف أو من أين تأتي البيانات (مستودع، خدمة خارجية، إلخ).

### 3. `RubikCare.Infrastructure` - الجسر إلى العالم الخارجي 🌉
**التفاصيل التقنية هنا.**
هذا هو مكان تنفيذ الواجهات المجردة التي عرّفها الـ `Application`. مسؤول عن التواصل مع قواعد البيانات، وإرسال البريد الإلكتروني، والخدمات السحابية.
*   **المحتويات**: `BusinessDbContext` (EF Core)، تنفيذ المستودعات (Repositories)، ملفات الترحيل (Migrations)، وخدمات خارجية (مثل `EmailService`).
*   **قاعدة ذهبية**: يُسمح لها بالاعتماد على `Application` و `Domain` فقط، ولا يُستدعى مباشرة من قبل التطبيقات العميلة (Web, PWA, Mobile).

### 4. `RubikCare.Api.Web` - بوابة النظام الموحدة 🚪
**نقطة الدخول الوحيدة للبيانات.**
هي واجهة برمجية (REST API) مركزية تخدم جميع العملاء (Web, PWA, Mobile). تتولى المصادقة (JWT)، والتحقق من الصلاحيات، وتنسيق الطلبات.
*   **المحتويات**: وحدات التحكم (Controllers)، `Program.cs`، وMiddleware (معالجة الأخطاء، التسجيل).
*   **قاعدة ذهبية**: وحدات التحكم يجب أن تكون **رفيعة جداً** (Thin Controllers)؛ كل المنطق يُفوض إلى الخدمات في طبقة `Application`.

### 5. `RubikCare.Web` - التطبيق التفاعلي الغني 💻
**تطبيق Blazor Server.**
يُستخدم في بيئات الإدارة واللوحات التي تتطلب تفاعلاً فورياً. يحافظ على اتصال مستمر (SignalR) مع الخادم، مما يوفر تجربة مستخدم سلسة.
*   **المحتويات**: صفحات (Pages)، مكونات خاصة بالويب، وخدمات محلية لإدارة الحالة.
*   **قاعدة ذهبية**: يتواصل مع الـ `Api.Web`، ولا يتفاعل مباشرة مع قواعد البيانات.

### 6. `RubikCare.PWA` - التطبيق الخفيف متعدد المنصات 📱
**تطبيق Blazor WebAssembly (التطبيق الأحدث).**
تمت إضافته ليكون بديلاً لتطبيق الويب (Blazor Server) على الأجهزة التي لا تتعامل بكفاءة مع SignalR (مثل أجهزة iPhone)، وأيضاً ليكون حلّاً سريعاً ومستقراً للوصول إلى النظام من أي متصفح.
*   **طبيعة التشغيل**: يعمل كلياً داخل متصفح المستخدم بعد تحميل ملفات `WebAssembly`. يُنشر كملفات **ثابتة** (Static Files) على خادم ويب (مثل IIS).
*   **المحتويات**: صفحات Razor مكافئة لـ `RubikCare.Web`، وخدمات للتواصل مع الـ API (مثل `WebApiService`).
*   **الحالة**: قيد التطوير النشط، مع خارطة طريق واضحة لنقل المكونات من `Shared.UI`.
*   **الرابط المرجعي**: [دليل PWA](https://github.com/Rubikans-Egy/Rubikcare-docs/blob/main/docs/22-RubikCare.PWA.md).

### 7. `RubikCare.Mobile` - تجربة الهواتف الذكية 📲
**تطبيق MAUI الأصلي.**
يستخدم `BlazorWebView` لعرض مكونات `Shared.UI`، مما يسمح بإعادة استخدام كبير للكود مع تطبيقات الويب. يوفر إمكانية الوصول إلى ميزات الجهاز (الكاميرا، GPS، الإشعارات المحلية).
*   **المحتويات**: صفحات XAML، ViewModels، وخدمات خاصة بالمنصة (مثل `MauiLocalNotificationService`).
*   **قاعدة ذهبية**: لا يعتمد على أي مشروع آخر باستثناء `Shared.UI`؛ يتواصل مع النظام حصراً عبر `Api.Web` باستخدام HTTP.

### 8. `RubikCare.Shared.UI` - مستودع المكونات المشتركة 🧩
**اللبنة الأساسية لواجهات المستخدم.**
مكتبة تحتوي على كل المكونات البصرية والخدمات المساعدة التي تستخدمها التطبيقات العميلة الثلاثة (Web, PWA, Mobile)، مما يضمن توحيد الشكل والأداء.
*   **المحتويات**: 66+ مكون Razor (مثل `RubikSmartTable`، `RubikButton`)، أكثر من 120 ملف CSS، و23 خدمة/واجهة مشتركة (مثل `ITranslationService`).
*   **قاعدة ذهبية**: تحتوي على **مكونات** قابلة لإعادة الاستخدام، وليس صفحات كاملة، لتجنب تعقيد التبعيات.

### 9. `RubikCare.Tests` - شبكة الأمان الجودة 🛡️
**ضمان استقرار المعمارية.**
يحتوي على اختبارات الوحدة (Unit Tests) لكل طبقة، واختبارات تكامل (Integration Tests) بين الطبقات، بالإضافة إلى اختبارات معماريّة باستخدام `NetArchTest` للتأكد من عدم كسر قواعد التبعية.
*   **قاعدة ذهبية**: جزء لا يتجزأ من عملية التطوير.

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

*   **لا تكسر تبعية الطبقات**: ممنوع استدعاء `Infrastructure` أو `DbContext` مباشرة من `Web` أو `PWA` أو `Mobile`.
*   **الـ `API` للجميع**: تطبيقات `Mobile` و `PWA` **لا تعتمد** على أي مشروع آخر (`Domain`, `Application`...)، وتتواصل فقط مع `Api.Web` عبر HTTP.
*   **`Shared.UI` للمكونات**: لا تضع صفحات كاملة (مثل `Dashboard.razor`) في `Shared.UI`، بل ضعها في المشاريع العميلة نفسها (`Web`, `PWA`, `Mobile`).
*   **استخدم DTOs**: لا تُرجع كائنات `Domain` (Entities) مباشرة من الـ `API`، استخدم DTOs لمنع تسرب تفاصيل النطاق وتقليل حجم البيانات.

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

*   **[01 - Program.cs والتسجيلات الأساسية](./01-program-cs-foundation.md)**: لفهم كيفية تسجيل الخدمات والحقن.
*   **[02 - نظام الهوية والمصادقة](./02-identity-system.md)**: لإدارة المستخدمين والأدوار.
*   **[09 - دليل الـ API](./09-api-guide.md)**: لمعرفة كيفية استخدام وبناء نقاط النهاية.
*   **[10 - دليل تطوير MAUI](./10-maui-development-guide.md)**: لنشر وتطوير تطبيق الموبايل.
*   **[12 - دليل النشر الشامل](./12%20-%20%D8%AF%D9%84%D9%8A%D9%84%20%D8%A7%D9%84%D9%86%D8%B4%D8%B1%20%D9%88%D8%A7%D9%84%D8%A5%D9%86%D8%AA%D8%A7%D8%AC%20%D8%A7%D9%84%D8%B4%D8%A7%D9%85%D9%84%20(Deployment%20Guide).md)**: لإعداد البيئات والنشر على IIS.
*   **[22 - دليل RubikCare.PWA](./22-RubikCare.PWA.md)**: للنشر وحالة التطوير الخاصة بـ PWA.
*   **[24 - جرد هيكلة المشاريع وحالة PWA](./24-Project%20Structure%20Inventory%20%26%20PWA%20Migration%20Status.md)**: لمعرفة المكونات المنقولة والمتبقية في PWA.

---

## 📝 سجل التحديثات (Changelog)

| التاريخ | التغيير | السبب |
| :--- | :--- | :--- |
| 29 أغسطس 2026 | **إعادة كتابة شاملة للوثيقة**. | دمج مشروع PWA وهيكل المشاريع الجديد (9 مشاريع). |
| 29 أغسطس 2026 | إضافة **شجرة القرار المعماري** و **جداول المقارنة**. | توجيه المطورين لاختيار النهج الصحيح (UseCase vs GenericService). |
| 18 يوليو 2026 | إضافة قسم "شجرة القرار المعماري". | حل التناقض بين الوثائق وتوحيد الممارسات. |
| 18 يوليو 2026 | تحديث العلاقات وقواعد Shared.UI. | توضيح الفرق بين المكونات والصفحات في المكتبة المشتركة. |

---

**خاتمة**: هذه الوثيقة هي البوصلة المعمارية لـ RubikCare. أي تغيير جوهري في الهيكل أو إضافة مشروع جديد يجب أن ينعكس هنا أولاً، لتظل المرجعية الأساسية للفريق.
