# 📄 الملحق ب - فهرس الخدمات (Service Index)

**آخر تحديث:** 18 يوليو 2026

## 📌 مقدمة

هذا الملف هو المرجع السريع والشامل لجميع الخدمات المسجلة في `Program.cs` والمسموح باستخدامها في طبقة العرض (Web/Mobile). 

> ⚠️ **قبل اختيار أي خدمة، يرجى مراجعة:**
> - 🧭 [شجرة القرار المعماري](00-architecture-overview.md#-شجرة-القرار-المعماري-كيف-تختار-النهج-الصحيح)
> - 📊 [مصفوفة اختيار الخدمة](01-program-cs-foundation.md#-مصفوفة-اختيار-الخدمة---مرجع-اتخاذ-القرار)

---

## 📊 1. الخدمات الأساسية (Basic Services)

| الخدمة | الواجهة | متى تستخدمها؟ | المرجع |
|--------|---------|---------------|--------|
| `GenericService<T>` | `IGenericService<T>` | **CRUD بسيط** على كيان واحد (عرض، إضافة، تعديل، حذف) بدون منطق أعمال معقد. | [01-Program.cs](01-program-cs-foundation.md) |
| `DbContextFactoryService` | - | **استعلامات قراءة مخصصة ومعقدة** (تقارير، فلترة متعددة، إحصائيات) لا تناسب `IGenericService`. | [01-Program.cs](01-program-cs-foundation.md) |

---

## 🌍 2. خدمات الترجمة (Translation Services)

| الخدمة | الواجهة | متى تستخدمها؟ | المرجع |
|--------|---------|---------------|--------|
| `TranslationStateService` | - | لمعرفة **اللغة الحالية** والاشتراك في حدث تغيير اللغة (`OnLanguageChanged`). | [01-Program.cs](01-program-cs-foundation.md) |
| `ILocalizationService` | `ILocalizationService` | لجلب **النصوص المترجمة** من قاعدة البيانات بناءً على المفتاح (`ResourceKey`) واللغة. | [01-Program.cs](01-program-cs-foundation.md) |
| `ILayoutDirectionService` | `ILayoutDirectionService` | لإدارة **اتجاه الصفحة** (RTL للعربية، LTR للإنجليزية) وضبط سمة `dir`. | [01-Program.cs](01-program-cs-foundation.md) |

---

## 👤 3. خدمات المستخدم والسياق (User & Context Services)

| الخدمة | الواجهة | متى تستخدمها؟ | المرجع |
|--------|---------|---------------|--------|
| `UserContextService` | `IUserContextService` | للحصول على **معلومات المستخدم الحالي** المسجل دخوله (ID, الأدوار، الصلاحيات) بسرعة. | [01-Program.cs](01-program-cs-foundation.md) |
| `UserSessionService` | `IUserSessionService` | لإدارة **الجلسات والكاش الموحد** للمستخدم (تفضيلات، بيانات أساسية). المصدر الوحيد للكاش. | [02-Identity](02-identity-system.md), [14-Caching](14-caching-system.md) |
| `UserRoleService` | `IUserRoleService` | للتعامل مع جدول `AspNetUserRoles` **الموسع** (إضافة، تعديل، حذف أدوار مع بيانات مهنية وزمنية). | [02-Identity](02-identity-system.md) |

---

## 🧭 4. خدمات التنقل والقوائم (Navigation & Menu Services)

| الخدمة | الواجهة | متى تستخدمها؟ | المرجع |
|--------|---------|---------------|--------|
| `DynamicMenuService` | - | لجلب **القوائم الديناميكية** المخصصة للمستخدم بناءً على أدواره وعضويته في المنظمة. | [04-Dynamic Menus](04-dynamic-menus.md) |

---

## 🔐 5. خدمات الهوية والمصادقة (Identity Services)

| الخدمة | الواجهة | متى تستخدمها؟ | المرجع |
|--------|---------|---------------|--------|
| `IdentityRedirectManager` | - | لإعادة التوجيه الآمن بعد عمليات المصادقة (تسجيل الدخول، تسجيل الخروج) مع الحفاظ على حالة المصادقة. | [01-Program.cs](01-program-cs-foundation.md) |
| `AuthenticationStateProvider` | `AuthenticationStateProvider` | للتحقق من حالة المصادقة الحالية للمستخدم (مسجل دخول أم لا) في مكونات Blazor. | [01-Program.cs](01-program-cs-foundation.md) |

---

## 🛠️ 6. خدمات المساعدة (Helper Services)

| الخدمة | الواجهة | متى تستخدمها؟ | المرجع |
|--------|---------|---------------|--------|
| `ExcelService` | `IExcelService<T>` | لاستيراد أو تصدير البيانات بصيغة Excel. | [01-Program.cs](01-program-cs-foundation.md) |
| `ImageService` | `IImageService` | لرفع، التحقق من صحة، ومعالجة الصور (مثل صور البروفايل أو المنتجات). | [01-Program.cs](01-program-cs-foundation.md) |

---

## ⚙️ 7. حالات الاستخدام الشائعة (Common Use Cases)

> 💡 **ملاحظة:** يجب استدعاء الـ UseCases من طبقة `Application`، ولا يجب كتابة منطق الأعمال في Controllers أو صفحات Blazor مباشرة.

| حالة الاستخدام (Use Case) | الغرض | متى تستخدمها؟ | المرجع |
|---------------------------|-------|---------------|--------|
| `ClearUserCacheUseCase` | مسح جميع كاشات المستخدم عند تسجيل الخروج لضمان الأمان. | عند تنفيذ عملية `Logout`. | [02-Identity](02-identity-system.md) |
| `SyncCompanyMedicationsUseCase` | مزامنة أدوية شركة معينة مع الجدول الرئيسي بعد المراجعة. | عند تنفيذ عملية المزامنة في لوحة تحكم الشركة. | [05-Page Creation](05-page-creation-checklist.md) |
| `RegisterDoctorUseCase` | تسجيل طبيب جديد مع إنشاء `UserProfile` وربطه بـ `ApplicationUser` وإضافة الدور. | عند إضافة مستخدم جديد بصلاحيات معقدة. | [02-Identity](02-identity-system.md) |

---

## 🚫 8. ⚠️ الخدمات والأنماط المحظورة (Forbidden Patterns)

ممنوع تماماً استخدام الأنماط التالية في أي صفحة أو خدمة **جديدة**:

| النمط المحظور | السبب | البديل الصحيح |
|---------------|-------|----------------|
| `@inject BusinessDbContext` مباشرة في صفحة Blazor | يكسر مبدأ فصل الاهتمامات، ويسبب أخطاء "A second operation started". | استخدم `DbContextFactoryService` أو `IGenericService<T>`. |
| كتابة منطق أعمال (Business Logic) داخل `Controllers` | الـ Controllers مخصصة للتنسيق (Orchestration) فقط. | انقل المنطق إلى `UseCase` في طبقة `Application`. |
| استدعاء `Infrastructure` (مثل Repositories) مباشرة من `Web` أو `Mobile` | يكسر تبعية الطبقات (Dependency Rule). | استخدم `UseCase` أو `ApiService` (للموبايل). |
| وضع **صفحات كاملة** (مثل `Dashboard.razor`) في `Shared.UI` | يسبب مشاكل أداء في MAUI ويكسر مبدأ المسؤولية الواحدة. | ضع **المكونات فقط** (Components) في `Shared.UI`. |
| استخدام `IMemoryCache` بشكل مباشر ومتفرق في الخدمات | يؤدي إلى بيانات قديمة (Stale Data) وصعوبة في المسح. | استخدم `UserSessionService` كمصدر وحيد للكاش. |
| إرجاع `Entity` مباشرة من API | يعرض بيانات حساسة غير مقصودة ويكسر التجريد. | استخدم `DTO` (Data Transfer Object) دائماً. |

---

## ✅ CHECKLIST: قبل استخدام أي خدمة

- [ ] هل راجعت **شجرة القرار المعماري** للتأكد من أن هذه هي الخدمة الصحيحة لنوع صفحتي؟
- [ ] هل هذه الخدمة مسجلة بالفعل في `Program.cs`؟
- [ ] هل أفهم الفرق بين استخدام `IGenericService<T>` (للبسيط) و `UseCase` (للمعقد)؟
- [ ] هل أتجنب تماماً الأنماط المحظورة المذكورة أعلاه؟

---

## 🔗 روابط ذات صلة

- [00 - الهيكل المعماري](00-architecture-overview.md)
- [01 - Program.cs والتسجيلات الأساسية](01-program-cs-foundation.md)
- [02 - نظام الهوية والمصادقة](02-identity-system.md)
- [05 - إنشاء الصفحات والمكونات](05-page-creation-checklist.md)
- [14 - نظام الكاش الموحد](14-caching-system.md)

---

**آخر تحديث:** 18 يوليو 2026 | **الملف:** `appendix-b-service-index.md`

---

### 📊 ملخص ما تم إضافته في هذا الملف:
1. ✅ **روابط مباشرة** لشجرة القرار ومصفوفة الخدمات في المقدمة.
2. ✅ **7 فئات منظمة** للخدمات مع عمود "متى تستخدمها؟" لكل خدمة.
3. ✅ **قسم UseCases** لتوثيق حالات الاستخدام الشائعة كمثال عملي.
4. ✅ **قسم الخدمات المحظورة** كتحذير واضح للمطورين الجدد.
5. ✅ **CHECKLIST** سريع قبل الاستخدام.
