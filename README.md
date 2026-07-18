**الإصدار:** 1.3 | **آخر تحديث:** 18 يوليو 2026

---

## 📌 عن هذا المستودع

هذا المستودع هو المصدر الوحيد والمرجع الرسمي لكل ما يتعلق بتطوير منصة RubikCare. يحتوي على:

- المعمارية (Clean Architecture - 8 مشاريع)
- التصميم (UI/UX - Web + Mobile)
- التنفيذ (قواعد وأنماط إلزامية)
- APIs (التعريفات والاستخدام)
- أدلة المطورين (MAUI, Blazor, BlazorWebView)
- النشر (IIS, APK)

---

## 🆕 آخر التحديثات (18 يوليو 2026)

### 🔥 التحديثات الرئيسية

| الوثيقة | التحديث | الأهمية |
|---------|---------|---------|
| **00-architecture-overview.md** | ✅ إضافة شجرة القرار المعماري، أنواع الصفحات الخمسة، قواعد Shared.UI | 🔴 حرج |
| **01-program-cs-foundation.md** | ✅ إضافة مصفوفة اختيار الخدمة، أمثلة عملية، CHECKLIST محدّث | 🔴 حرج |
| **02-identity-system.md** | ✅ إضافة شجرة قرار الهوية، أمثلة UseCase، ربط بالوثيقتين السابقتين | 🟠 مهم |
| **03-style-guide.md** | ✅ إضافة أنماط المصادقة، دليل التجاوبية، متغيرات دلالية | 🔴 حرج |
| **05-page-creation-checklist.md** | ✅ تحديث شامل مع شجرة القرار، مصفوفة الخدمات، Shared.UI | 🔴 حرج |

### 📊 ملخص التحديثات

- ✅ **شجرة القرار المعماري:** دليل عملي لاختيار النهج الصحيح (CRUD vs UseCase vs API)
- ✅ **أنماط صفحات المصادقة:** توثيق `.forgot-*` classes ونمط Split-Panel
- ✅ **دليل التجاوبية:** Breakpoints, Touch Targets, iOS Zoom Prevention
- ✅ **مصفوفة اختيار الخدمة:** متى تستخدم `IGenericService<T>` vs UseCase vs API
- ✅ **قواعد Shared.UI:** مكونات vs صفحات + رسم توضيحي

---

## 🧭 خريطة التوثيق

### 🏗️ الأساسيات والمعمارية

| الملف | المحتوى | التحديث الأخير |
|-------|---------|----------------|
| [00-architecture-overview.md](docs/00-architecture-overview.md) | Clean Architecture، هيكل المشاريع الثمانية، علاقاتها، **شجرة القرار المعماري** ⭐ | 18 يوليو 2026 |
| [01-program-cs-foundation.md](docs/01-program-cs-foundation.md) | Program.cs، التسجيلات الأساسية، Middleware، App.razor، **مصفوفة اختيار الخدمة** ⭐ | 18 يوليو 2026 |
| [02-identity-system.md](docs/02-identity-system.md) | نظام الهوية، AspNetUsers، UserProfile، الأدوار، نظام إدارة الجلسات والكاش ⭐ | 18 يوليو 2026 |

### 🎨 التصميم والواجهة

| الملف | المحتوى | التحديث الأخير |
|-------|---------|----------------|
| [03-style-guide.md](docs/03-style-guide.md) | الألوان، الخطوط، `.rubik-dark`، هيكل CSS، **أنماط المصادقة** ⭐، **دليل التجاوبية** ⭐ | 18 يوليو 2026 |
| [04-dynamic-menus.md](docs/04-dynamic-menus.md) | SystemMenu، MenuItem، MenuAssignment، DynamicMenuService (بدون كاش) ⭐ | 24 مايو 2026 |
| [05-page-creation-checklist.md](docs/05-page-creation-checklist.md) | إنشاء الصفحات، الترجمة، المكونات الذكية، **شجرة القرار المعماري** ⭐، **قواعد Shared.UI** ⭐ | 18 يوليو 2026 |

### 💊 أنظمة الأعمال

| الملف | المحتوى | التحديث الأخير |
|-------|---------|----------------|
| [07-psp-system.md](docs/07-psp-system.md) | برامج دعم المرضى، APIs، التدفقات | 17 مايو 2026 |

### 📱 أدلة المنصات

| الملف | المحتوى | التحديث الأخير |
|-------|---------|----------------|
| [09-api-guide.md](docs/09-api-guide.md) | دليل API الكامل | 17 مايو 2026 |
| [10-maui-development-guide.md](docs/10-maui-development-guide.md) | دليل تطوير MAUI، إدارة الجلسات والكاش في الموبايل + ⭐ حل مشكلة JavaProxyThrowable | 24 مايو 2026 |
| [11-blazor-webview-guide.md](docs/11-blazor-webview-guide.md) | دليل BlazorWebView والأنماط الناجحة | 17 مايو 2026 |

### 🚀 التطوير والنشر

| الملف | المحتوى | التحديث الأخير |
|-------|---------|----------------|
| [06-troubleshooting-methodology.md](docs/06-troubleshooting-methodology.md) | منهجية حل المشاكل (الخطوات السبع) + زر الرجوع للخلف | 17 مايو 2026 |
| [08-roadmap.md](docs/08-roadmap.md) | خريطة التطوير والوضع الحالي | 17 مايو 2026 |
| [12-deployment-guide.md](docs/12-deployment-guide.md) | النشر على IIS و APK | 17 مايو 2026 |
| [13-clean-architecture-enforcement.md](docs/13-clean-architecture-enforcement.md) | إصلاح وتطبيق Clean Architecture، ✅ تم: إزالة References، إصلاح 16 Controller، Architecture Tests ⭐ | 17 مايو 2026 |

### 🆕 أنظمة جديدة

| الملف | المحتوى | التحديث الأخير |
|-------|---------|----------------|
| [14-caching-system.md](docs/14-caching-system.md) | نظام الكاش الموحد - هيكل، طبقات، قواعد الاستخدام ⭐ | 24 مايو 2026 |
| [15-mobile-translation-system.md](docs/15-mobile-translation-system.md) | نظام الترجمة في الموبايل | 17 مايو 2026 |
| [14-google-signin-android.md](docs/14-google-signin-android.md) | Google Sign-In على Android — WebAuthenticator + PKCE + APK Signing | 24 مايو 2026 |

### 📚 المراجع

| الملف | المحتوى | التحديث الأخير |
|-------|---------|----------------|
| [appendix-a-glossary.md](docs/appendix-a-glossary.md) | مسرد المصطلحات | 17 مايو 2026 |
| [appendix-b-service-index.md](docs/appendix-b-service-index.md) | فهرس الخدمات | 17 مايو 2026 |

---

## 👥 الجمهور المستهدف

- مطورو Backend (Api.Web, Application, Infrastructure, Domain)
- مطورو Frontend (Blazor Server, MAUI, BlazorWebView)
- مطورو Shared.UI (Razor Class Library)
- أي مطور جديد ينضم للمشروع

---

## 🛠️ التقنيات المستخدمة

| التقنية | الاستخدام |
|---------|-----------|
| .NET 10 | الإطار العام |
| Blazor Server | تطبيق الويب |
| MAUI + BlazorWebView | تطبيق الموبايل |
| Clean Architecture | المعمارية (Domain, Application, Infrastructure, API, Web, Mobile, Shared.UI) |
| Entity Framework Core | ORM |
| SQL Server | قاعدة البيانات |
| Bootstrap 5 + Custom CSS | واجهة المستخدم |
| JWT | المصادقة في API |
| IMemoryCache | الكاش الموحد (في UserSessionService فقط) |
| NetArchTest.Rules | Architecture Tests (حماية المعمارية) ⭐ |
| InfrastructureExtensions | تسجيل جميع الخدمات في Extension Method واحد ⭐ |

---

## 📝 كيفية استخدام هذا الدليل

1. ابدأ بـ [00-architecture-overview.md](docs/00-architecture-overview.md) لفهم الهيكل العام
2. انتقل للملف الذي يهمك حسب تخصصك
3. عند مواجهة مشكلة، راجع [06-troubleshooting-methodology.md](docs/06-troubleshooting-methodology.md)
4. لفهم نظام الكاش، راجع [14-caching-system.md](docs/14-caching-system.md) ⭐
5. لفهم قواعد Clean Architecture، راجع [13-clean-architecture-enforcement.md](docs/13-clean-architecture-enforcement.md) ⭐
6. **⭐ جديد:** لاختيار الخدمة الصحيحة، راجع **مصفوفة اختيار الخدمة** في [01-program-cs-foundation.md](docs/01-program-cs-foundation.md)
7. **⭐ جديد:** لفهم أنماط المصادقة والتجاوبية، راجع [03-style-guide.md](docs/03-style-guide.md)

---

## ⚠️ ملاحظات هامة

- هذا المستودع يحتوي على توثيق تقني فقط - الكود المصدري في مستودع منفصل
- المحتوى التسويقي ومنهجية العمل في مستودع `rubikcare-methodology`
- جميع الملفات بصيغة Markdown (`.md`) لسهولة القراءة على GitHub
- أي مخالفة لـ Clean Architecture ستفشل Architecture Tests (33 اختبار) 🔒

---

## 🔗 روابط سريعة لجميع الملفات (للنسخ والمشاركة)

| # | الملف | الرابط المباشر |
|---|-------|---------------|
| 1 | README.md | https://raw.githubusercontent.com/Rubikans-Egy/Rubikcare-docs/main/README.md |
| 2 | 00-architecture-overview.md | https://raw.githubusercontent.com/Rubikans-Egy/Rubikcare-docs/main/docs/00-architecture-overview.md |
| 3 | 01-program-cs-foundation.md | https://raw.githubusercontent.com/Rubikans-Egy/Rubikcare-docs/main/docs/01-program-cs-foundation.md |
| 4 | 02-identity-system.md | https://raw.githubusercontent.com/Rubikans-Egy/Rubikcare-docs/main/docs/02-identity-system.md |
| 5 | 03-style-guide.md | https://raw.githubusercontent.com/Rubikans-Egy/Rubikcare-docs/main/docs/03-style-guide.md |
| 6 | 04-dynamic-menus.md | https://raw.githubusercontent.com/Rubikans-Egy/Rubikcare-docs/main/docs/04-dynamic-menus.md |
| 7 | 05-page-creation-checklist.md | https://raw.githubusercontent.com/Rubikans-Egy/Rubikcare-docs/main/docs/05-page-creation-checklist.md |
| 8 | 06-troubleshooting-methodology.md | https://raw.githubusercontent.com/Rubikans-Egy/Rubikcare-docs/main/docs/06-troubleshooting-methodology.md |
| 9 | 07-psp-system.md | https://raw.githubusercontent.com/Rubikans-Egy/Rubikcare-docs/main/docs/07-psp-system.md |
| 10 | 08-roadmap.md | https://raw.githubusercontent.com/Rubikans-Egy/Rubikcare-docs/main/docs/08-roadmap.md |
| 11 | 09-api-guide.md | https://raw.githubusercontent.com/Rubikans-Egy/Rubikcare-docs/main/docs/09-api-guide.md |
| 12 | 10-maui-development-guide.md | https://raw.githubusercontent.com/Rubikans-Egy/Rubikcare-docs/main/docs/10-maui-development-guide.md |
| 13 | 11-blazor-webview-guide.md | https://raw.githubusercontent.com/Rubikans-Egy/Rubikcare-docs/main/docs/11-blazor-webview-guide.md |
| 14 | 12-deployment-guide.md | https://raw.githubusercontent.com/Rubikans-Egy/Rubikcare-docs/main/docs/12-deployment-guide.md |
| 15 | 13-clean-architecture-enforcement.md | https://raw.githubusercontent.com/Rubikans-Egy/Rubikcare-docs/main/docs/13-clean-architecture-enforcement.md |
| 16 | 14-caching-system.md ⭐ | https://raw.githubusercontent.com/Rubikans-Egy/Rubikcare-docs/main/docs/14-caching-system.md |
| 17 | 15-mobile-translation-system.md | https://github.com/Rubikans-Egy/Rubikcare-docs/blob/main/docs/15-mobile-translation-system.md |
| 18 | appendix-a-glossary.md | https://raw.githubusercontent.com/Rubikans-Egy/Rubikcare-docs/main/docs/appendix-a-glossary.md |
| 19 | appendix-b-service-index.md | https://raw.githubusercontent.com/Rubikans-Egy/Rubikcare-docs/main/docs/appendix-b-service-index.md |
| 20 | 14-google-signin-android.md | https://raw.githubusercontent.com/Rubikans-Egy/Rubikcare-docs/main/docs/14-google-signin-android.md |

---

## 📊 إحصائيات المستودع

| الفئة | العدد |
|-------|------|
| **إجمالي الوثائق** | 20 |
| **وثائق محدّثة (18 يوليو 2026)** | 5 |
| **وثائق جديدة (2026)** | 4 |
| **أقسام جديدة مضافة** | 12 |

---

## 🎯 الأقسام الأكثر أهمية (⭐ جديد)

### 1. شجرة القرار المعماري
**الموقع:** [00-architecture-overview.md](docs/00-architecture-overview.md) + [05-page-creation-checklist.md](docs/05-page-creation-checklist.md)

**الفائدة:** دليل عملي لاختيار النهج الصحيح لكل نوع صفحة (CRUD vs UseCase vs API)

### 2. مصفوفة اختيار الخدمة
**الموقع:** [01-program-cs-foundation.md](docs/01-program-cs-foundation.md)

**الفائدة:** جدول شامل يوضح متى تستخدم كل خدمة (`IGenericService<T>`, UseCase, API, DbContextFactory)

### 3. أنماط صفحات المصادقة
**الموقع:** [03-style-guide.md](docs/03-style-guide.md)

**الفائدة:** توثيق كامل لنمط Split-Panel، `.forgot-*` classes، والتجاوبية

### 4. دليل التجاوبية
**الموقع:** [03-style-guide.md](docs/03-style-guide.md)

**الفائدة:** Breakpoints, Touch Targets (48px), iOS Zoom Prevention

### 5. قواعد Shared.UI
**الموقع:** [00-architecture-overview.md](docs/00-architecture-overview.md) + [05-page-creation-checklist.md](docs/05-page-creation-checklist.md)

**الفائدة:** متى تضع مكوناً في Shared.UI vs صفحة كاملة

---

© 2026 RubikCare - للاستخدام الداخلي
```

---

## 📊 ملخص التحديثات في README.md

| القسم | التغيير |
|-------|---------|
| **الإصدار والتاريخ** | ✅ تحديث إلى 1.3 و 18 يوليو 2026 |
| **قسم "آخر التحديثات"** | ✅ إضافة جديدة بالكامل توضح ما تم تحديثه |
| **جداول الوثائق** | ✅ إضافة عمود "التحديث الأخير" لكل وثيقة |
| **أوصاف الوثائق المحدّثة** | ✅ إضافة ⭐ للمحتوى الجديد (شجرة القرار، مصفوفة الخدمات، أنماط المصادقة) |
| **قسم "الأقسام الأكثر أهمية"** | ✅ إضافة جديدة تبرز 5 أقسام حرجة |
| **قسم "إحصائيات المستودع"** | ✅ إضافة جديدة |
| **كيفية الاستخدام** | ✅ إضافة 3 نقاط جديدة للوثائق المحدّثة |
| **الروابط** | ✅ إضافة `appendix-b-service-index.md` |

---

## ✅ ما تم إنجازه في تحديث الوثائق

| الوثيقة | الحالة | التحديثات الرئيسية |
|---------|--------|-------------------|
| **00-architecture-overview.md** | ✅ محدّث | شجرة القرار المعماري، أنواع الصفحات، قواعد Shared.UI |
| **01-program-cs-foundation.md** | ✅ محدّث | مصفوفة اختيار الخدمة، أمثلة عملية، CHECKLIST محدّث |
| **02-identity-system.md** | ✅ محدّث | شجرة قرار الهوية، أمثلة UseCase |
| **03-style-guide.md** | ✅ محدّث | أنماط المصادقة، دليل التجاوبية، متغيرات دلالية |
| **05-page-creation-checklist.md** | ✅ محدّث | شجرة القرار، مصفوفة الخدمات، Shared.UI، CHECKLIST شامل |
| **README.md** | ✅ محدّث | قسم التحديثات، إحصائيات، روابط محدّثة |
