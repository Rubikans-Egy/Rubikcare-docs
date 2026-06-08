**الإصدار: 1.2** | **آخر تحديث: 25 مايو 2026**

---

## 📌 عن هذا المستودع

هذا المستودع هو **المصدر الوحيد والمرجع الرسمي** لكل ما يتعلق بتطوير منصة RubikCare. يحتوي على:

- المعمارية (Clean Architecture - 8 مشاريع)
- التصميم (UI/UX - Web + Mobile)
- التنفيذ (قواعد وأنماط إلزامية)
- APIs (التعريفات والاستخدام)
- أدلة المطورين (MAUI, Blazor, BlazorWebView)
- النشر (IIS, APK)

---

## 🧭 خريطة التوثيق

### 🏗️ الأساسيات والمعمارية

| الملف | المحتوى |
|-------|---------|
| [00-architecture-overview.md](docs/00-architecture-overview.md) | Clean Architecture، هيكل المشاريع الثمانية، علاقاتها |
| [01-program-cs-foundation.md](docs/01-program-cs-foundation.md) | Program.cs، التسجيلات الأساسية، Middleware، App.razor |
| [02-identity-system.md](docs/02-identity-system.md) | نظام الهوية، AspNetUsers، UserProfile، الأدوار، **نظام إدارة الجلسات والكاش** ⭐ |

### 🎨 التصميم والواجهة

| الملف | المحتوى |
|-------|---------|
| [03-style-guide.md](docs/03-style-guide.md) | الألوان، الخطوط، `.rubik-dark`، هيكل CSS |
| [04-dynamic-menus.md](docs/04-dynamic-menus.md) | SystemMenu، MenuItem، MenuAssignment، **DynamicMenuService (بدون كاش)** ⭐ |
| [05-page-creation-checklist.md](docs/05-page-creation-checklist.md) | إنشاء الصفحات، الترجمة، المكونات الذكية |

### 💊 أنظمة الأعمال

| الملف | المحتوى |
|-------|---------|
| [07-psp-system.md](docs/07-psp-system.md) | برامج دعم المرضى، APIs، التدفقات |

### 📱 أدلة المنصات

| الملف | المحتوى |
|-------|---------|
| [09-api-guide.md](docs/09-api-guide.md) | دليل API الكامل |
| 10-maui-development-guide.md | دليل تطوير MAUI، إدارة الجلسات والكاش في الموبايل + ⭐ حل مشكلة JavaProxyThrowable |
| [11-blazor-webview-guide.md](docs/11-blazor-webview-guide.md) | دليل BlazorWebView والأنماط الناجحة |

### 🚀 التطوير والنشر

| الملف | المحتوى |
|-------|---------|
| [06-troubleshooting-methodology.md](docs/06-troubleshooting-methodology.md) | منهجية حل المشاكل (الخطوات السبع) + زر الرجوع للخلف|
| [08-roadmap.md](docs/08-roadmap.md) | خريطة التطوير والوضع الحالي |
| [12-deployment-guide.md](docs/12-deployment-guide.md) | النشر على IIS و APK |
| [13-clean-architecture-enforcement.md](docs/13-clean-architecture-enforcement.md) | إصلاح وتطبيق Clean Architecture، **✅ تم: إزالة References، إصلاح 16 Controller، Architecture Tests** ⭐ |

### 🆕 أنظمة جديدة

| الملف | المحتوى |
|-------|---------|
| [14-caching-system.md](docs/14-caching-system.md) | **نظام الكاش الموحد** - هيكل، طبقات، قواعد الاستخدام ⭐ |
| [15-mobile-translation-system.md](docs/15-mobile-translation-system.md) | نظام الترجمة في الموبايل |
| [14-google-signin-android.md](docs/14-google-signin-android.md) | Google Sign-In على Android — WebAuthenticator + PKCE + APK Signing |
### 📚 المراجع

| الملف | المحتوى |
|-------|---------|
| [appendix-a-glossary.md](docs/appendix-a-glossary.md) | مسرد المصطلحات |

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

---

## ⚠️ ملاحظات هامة

- هذا المستودع يحتوي على **توثيق تقني** فقط - الكود المصدري في مستودع منفصل
- المحتوى التسويقي ومنهجية العمل في مستودع `rubikcare-methodology`
- جميع الملفات بصيغة Markdown (`.md`) لسهولة القراءة على GitHub
- **أي مخالفة لـ Clean Architecture ستفشل Architecture Tests (33 اختبار)** 🔒

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
| 19 | 16 -google-signin-android.md](docs/14-google-signin-android.md) | Google Sign-In على Android — WebAuthenticator + PKCE + APK Signing |
---

© 2026 RubikCare - للاستخدام الداخلي

---
