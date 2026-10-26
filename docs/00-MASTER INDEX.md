# 00 — MASTER INDEX (فهرس روبيك كير الشامل)

الإصدار: 2.2 (PHR + Integration Hub Edition) 
آخر تحديث: 26 أكتوبر 2026 
الحالة: 🟢 مرجع حي — يُحدَّث كل جلسة 
الجمهور: فريق التطوير، AI Assistants، القيادة التقنية

## 📌 مقدمة

هذه الوثيقة هي البوصلة المعمارية لمشروع RubikCare. الغرض منها:

- فهرس كامل لكل الوثائق (22 وثيقة أساسية + ملاحق).
- حالة المشروع — ما هو موجود / مخطط / قيد التنفيذ.
- خريطة الجلسات — ما تم إنجازه في كل جلسة.
- مناطق مظلمة (Dark Areas) — ما هو منفَّذ لكن غير موثَّق.
- قواعد صارمة للـ AI Assistant.

🎯 القاعدة الذهبية:
"قبل أي جلسة تطوير — أرسل هذه الوثيقة للـ AI. عندها سيعرف ما هو موجود — وما لا ينبغي أن يفترض أنه غير موجود."

---

## 📚 الجزء الأول: فهرس الوثائق الكامل (الهيكل النهائي المُوحَّد)

### الوثائق الأساسية (00-22)

| # | الوثيقة | الموضوع | آخر تحديث | الحالة |
| :--- | :--- | :--- | :--- | :--- |
| 00 | 00-MASTER-INDEX.md | الفهرس الشامل (هذه الوثيقة) | 26 أكتوبر 2026 | ✅ |
| 00B | 00B-PROJECT-BACKLOG.md | سجل المهام الحية | 26 أكتوبر 2026 | ✅ |
| 01 | 01-ARCHITECTURE.md | الهيكل المعماري + Program.cs | 17 أكتوبر 2026 | ✅ |
| 02 | 02-IDENTITY.md | الهوية والمصادقة | 17 أكتوبر 2026 | ✅ |
| 03 | 03-MENUS.md | نظام القوائم الديناميكية | 16 أكتوبر 2026 | ✅ |
| 04 | 04-TRANSLATION.md | الترجمة والاتجاه | 16 أكتوبر 2026 | ✅ |
| 05 | 05-PSP.md | نظام دعم المرضى | 17 أكتوبر 2026 | ✅ |
| 06 | 06-POINTS.md | نظام النقاط وروبيكا | 17 أكتوبر 2026 | ✅ |
| 07 | 07-REPORTS.md | نظام التقارير | 17 أكتوبر 2026 | ✅ |
| 08 | 08-NOTIFICATIONS.md | الإشعارات والتذكيرات | 17 أكتوبر 2026 | ✅ |
| 09 | 09-API-GUIDE.md | دليل API الشامل (~290 Endpoint) | 16 أكتوبر 2026 | ✅ |
| 10 | 10-UI-STYLE-GUIDE.md | دليل الأنماط | 17 أكتوبر 2026 | ✅ |
| 11 | 11-PAGE-CREATION.md | إنشاء الصفحات | 17 أكتوبر 2026 | ✅ |
| 12 | 12-CACHING.md | نظام الكاش | 17 أكتوبر 2026 | ✅ |
| 13 | 13-MAUI.md | تطوير MAUI | 17 أكتوبر 2026 | ✅ |
| 14 | 14-PWA.md | تطبيق الويب التقدمي | 17 أكتوبر 2026 | ✅ |
| 15 | 15-DEPLOYMENT.md | دليل النشر | 17 أكتوبر 2026 | ✅ |
| 16 | 16-TROUBLESHOOTING.md | المنهجية + قواعد AI | 17 أكتوبر 2026 | ✅ |
| 17 | 17-STRATEGIC-ROADMAP.md | الدستور التطويري | 17 أكتوبر 2026 | ✅ |
| 18 | 18-MARKETING-WEBSITE.md | الموقع التسويقي | 17 أكتوبر 2026 | ✅ |
| 19 | 19-PERMISSIONS.md | نظام الصلاحيات الكامل | 16 أكتوبر 2026 | ✅ |
| 20 | 20-USER JOURNEY.md | رحلة المريض | 21 أكتوبر 2026 | ✅ |
| 21 | 21-PHR-SYSTEM.md | نظام السجلات الطبية الشخصية | 26 أكتوبر 2026 | 🟡 Draft |
| **22** | **22-INTEGRATION-HUB.md** | **مركز التكامل الخارجي** | **26 أكتوبر 2026** | **🟡 Draft** |

### الملاحق

| # | الوثيقة | الحالة |
| :--- | :--- | :--- |
| A | appendix-a-glossary.md | ✅ |
| B | appendix-b-service-index.md | ✅ |

---

## 🏗️ الجزء الثاني: بنية المشروع (9 مشاريع)

| # | المشروع | التقنية | المسؤولية | الحالة |
| :--- | :--- | :--- | :--- | :--- |
| 1 | RubikCare.Domain | .NET 10 Class Library | الكيانات + Enums + Constants | ✅ |
| 2 | RubikCare.Application | .NET 10 Class Library | Use Cases + DTOs + Interfaces | ✅ |
| 3 | RubikCare.Infrastructure | .NET 10 + EF Core 10 | DbContext + Repositories + Migrations | ✅ |
| 4 | Api.Web | ASP.NET Core 10 | REST API + Controllers (~290 Endpoint) | ✅ |
| 5 | Rubikcare.Web | Blazor Server 10 | تطبيق الويب | ✅ |
| 6 | RubikCare.PWA | Blazor WebAssembly 10 | نسخة كربونية من Mobile | 🟠 ~95% |
| 7 | Mobile | .NET MAUI 10 | تطبيق الموبايل | ✅ |
| 8 | Shared.UI | Razor Class Library | 68 مكون مشترك + 25 خدمة | ✅ |
| 9 | RubikCare.Tests | xUnit + Moq | 33 اختبار | ✅ |

---

## 🎯 الجزء الثالث: الوضع الحالي (Modules Snapshot)

### ✅ Modules مكتملة ومستقرة

| # | Module | Backend | Frontend | الوثيقة |
| :--- | :--- | :--- | :--- | :--- |
| 1 | الهوية والمصادقة | ✅ | ✅ | 02 |
| 2 | القوائم الديناميكية | ✅ | ✅ | 03 |
| 3 | الترجمة والاتجاه | ✅ | ✅ | 04 |
| 4 | نظام PSP + Phase System | ✅ | ✅ | 05 |
| 5 | نظام التقارير | ✅ | ✅ | 07 |
| 6 | نظام النقاط (Backend) | ✅ | 🟠 | 06 |
| 7 | Google Sign-In | ✅ | ✅ | 02 |
| 8 | الكاش الموحد | ✅ | ✅ | 12 |
| 9 | الإشعارات | ✅ | ✅ | 08 |
| 10 | الموقع التسويقي | ✅ | ✅ | 18 |
| 11 | نظام الصلاحيات (Schema) | ✅ | ❌ | 19 |

### 🟠 Modules مكتملة جزئياً

| # | Module | ما ينقص |
| :--- | :--- | :--- |
| 1 | نظام النقاط | واجهة المريض (PointDashboard) |
| 2 | PWA | MessagingHubPage + 3 لوحات مهنية |
| 3 | نظام الصلاحيات | الـ Middleware + الـ UI |
| 4 | طلبات شراء الأدوية | تكامل مع نظام النقاط |

### ⏳ Modules مُخطَّطة (لم تبدأ)

| # | Module | الرؤية | الوثيقة |
| :--- | :--- | :--- | :--- |
| 1 | B2B Commerce | Supply Chain كامل | 17 |
| 2 | B2C Pharmacy | تجارة التجزئة | 17 |
| 3 | Medical CRM | KPI's للمندوبين | 17 |
| 4 | Clinic OS | إدارة العيادات | 17 |
| 5 | CME | التعليم الطبي المستمر | 17 |
| 6 | Rubika Currency Engine | عملة داخلية | 06 |
| 7 | Medical Tourism | السياحة العلاجية | 17 |
| 8 | Personal Health Records (PHR) | السجل الطبي الشخصي | 21 |
| **9** | **Integration Hub** | **مركز التكامل الخارجي** | **22** |

---

## 🔴 الجزء الرابع: المناطق المظلمة (Dark Areas)

| # | الميزة | الموقع | التوثيق |
| :--- | :--- | :--- | :--- |
| 1 | PSPEnrollmentService | Application/Services/PSP/ | ✅ 05 |
| 2 | DispenseController | Api.Web/Controllers/ | ✅ 09 |
| 3 | MenuVisibilityRules UI | مفقود | 03 |
| 4 | OrganizationGroups | تأسيس فقط | ✅ 03 |
| 5 | Rubika Currency Schema | DB فقط | ✅ 06 |
| 6 | Messaging System | ✅ كامل | 14 |
| 7 | File Uploads | ✅ كامل | — |
| 8 | Booking System | ✅ كامل | 14 |
| 9 | Nearby Pharmacies | ✅ كامل | 14 |
| 10 | CustomBlazorWebView | ✅ كامل | ✅ 13 |
| 11 | Marketing Website — Header/Footer | ✅ كامل | ✅ 18 |

---

## 📅 الجزء الخامس: خريطة الجلسات

| الجلسة | التاريخ | ما تم | الوثائق المُحدَّثة |
| :--- | :--- | :--- | :--- |
| #104-108 | 10-13 أكتوبر 2026 | Campaign Reference + SourceEntityID | 06 |
| #109-115 | 14-15 أكتوبر 2026 | Loyalty UI + Reports + Beneficiaries | 06 + 07 |
| #116-121 | 16 أكتوبر 2026 | جرد شامل + الفهرس v1.0 | 00 |
| #122-130 | 16-17 أكتوبر 2026 | إعادة هيكلة الوثائق (34 → 20) | جميع الوثائق |
| #131 | 17 أكتوبر 2026 | دمج 04-TRANSLATION (v3.1) | 04 + 08 |
| #132 | 18 أكتوبر 2026 | إصدار 00-MASTER-INDEX v2.0 | 00 |
| #133 | 26 أكتوبر 2026 | إصدار 00-MASTER-INDEX v2.1 + PHR (21) + Sprint 4-6 | 00 + 00B + 21 |
| **#134** | **26 أكتوبر 2026** | **إصدار 00-MASTER-INDEX v2.2 + Integration Hub (22) + Sprint 7-9** | **00 + 00B + 22** |

---

## 🚨 الجزء السادس: الأولويات الحالية

### 🔴 P1 — حرجة

| # | المهمة | الوثيقة |
| :--- | :--- | :--- |
| 1 | إصلاح بيئة VS (wasm-tools) | 13 |
| 2 | اختبار E2E لنظام النقاط | 00B |
| 3 | واجهة المريض (PointDashboard) | 06 |
| 4 | حل مشكلة mscorlib.dll | 16 |

### 🟠 P2 — عالية

| # | المهمة | الوثيقة |
| :--- | :--- | :--- |
| 1 | ربط النقاط بالتقارير | 06 + 07 |
| 2 | تقرير النقاط المخصص | 07 |
| 3 | مستويات الولاء | 06 |
| 4 | إصلاح المفاتيح المترجمة | 04 |
| 5 | إصلاح الترميز العربي في الإشعارات | 08 |

### 🟡 P3 — متوسطة

| # | المهمة | الوثيقة |
| :--- | :--- | :--- |
| 1 | تنفيذ 19-PERMISSIONS | 19 |
| 2 | إعادة هيكلة Admin* | 11 |
| 3 | MenuVisibilityRules UI | 03 |
| 4 | MessagingHubPage في PWA | 14 |
| 5 | لوحات PWA المهنية | 14 |
| 6 | تنفيذ PHR Phase 1 (Sprint 4) | 21 |
| **7** | **تنفيذ Integration Hub Core (Sprint 7)** | **22** |

---

## 🗺️ الجزء السابع: خريطة الجلسات المستقبلية

| الجلسة | الموضوع | المتوقع | الوثيقة |
| :--- | :--- | :--- | :--- |
| القادمة | 19-PERMISSIONS (Phase 2 + 3) | Middleware + UI | 19 |
| +2 | ربط Menus + Services + Subscriptions | Super Admin UI | 03 |
| +3 | B2B Commerce — MVP | ربط النقاط بالتوريد | 17 |
| +4 | B2C Pharmacy | تجارة التجزئة | 17 |
| +5 | Medical CRM | KPI's للمندوبين | 17 |
| +6 | Clinic OS | إدارة العيادات | 17 |
| +7 | PHR Phase 1 (Sprint 4) | Hb Navigator + HealthRecord | 21 |
| +8 | PHR Phase 2 (Sprint 5) | تحاليل جديدة + مشاركة | 21 |
| +9 | PHR Phase 3 (Sprint 6) | تكامل كامل + تحكم | 21 |
| **+10** | **Integration Hub Core (Sprint 7)** | **IIntegrationClient + Auth** | **22** |
| **+11** | **ClinicOS Adapter (Sprint 8)** | **أول Adapter** | **22** |
| **+12** | **Webhooks + OpenAPI (Sprint 9)** | **بوابة مطورين** | **22** |

---

## 📋 الجزء الثامن: خريطة النشر

### البيئات

| البيئة | Web | API | PWA |
| :--- | :--- | :--- | :--- |
| Local | https://localhost | https://localhost | https://localhost |
| Test/UAT | test.rubikcare.com | uat.rubikcare.com | stagepu.rubikcare.com |
| Production | rubikcare.com | api.rubikcare.com | (تلقائي) |

### IIS Sites على VPS

| الموقع | الدومين | App Pool |
| :--- | :--- | :--- |
| RubikCareNew | test.rubikcare.com | rubikcarenew |
| RubikCareUat | uat.rubikcare.com | RubikCareUat |
| PU_RubicCareStage | stagepu.rubikcare.com | PU_RubicCareStage |
| RubikCareLive | rubikcare.com | — |
| RubikCareApi | api.rubikcare.com | — |

---

## 🧠 الجزء التاسع: قواعد صارمة للـ AI Assistant

### قبل أي جلسة — اطلب:

- هذه الوثيقة (00-MASTER-INDEX.md).
- الوثيقة المتعلقة بالمهمة (03, 05, 06, 07, 19, 21, 22 — حسب الحالة).

### 🎯 القواعد الإلزامية (14 قاعدة):

| # | القاعدة |
| :--- | :--- |
| 1 | لا تفترض أن ميزة غير موجودة — راجع الـ Dark Areas + الوثيقة المرتبطة. |
| 2 | لا تخمّن حقولاً — اطلب رؤية Entity أو SELECT TOP 1 *. |
| 3 | ارجع لـ 17-STRATEGIC-ROADMAP قبل أي قرار معماري كبير. |
| 4 | ارجع لـ 03-MENUS قبل أي تعديل على القوائم. |
| 5 | ارجع لـ 06-POINTS قبل أي تعديل على النقاط. |
| 6 | ارجع لـ 11-PAGE-CREATION قبل أي كود جديد. |
| 7 | قاعدة "Stable vs Unstable" — لا تعيد هيكلة Stable Modules بدون إذن. |
| 8 | EF Core Code-First — لا تعديل يدوي على Migration files. |
| 9 | استخدم N'' في SQL — للعربية. |
| 10 | استخدم MERGE بدلاً من INSERT/UPDATE — منفصلة. |
| 11 | قبل كتابة أي كود، اذكر صراحةً: "أحتاج لرؤية ملف [الاسم] للتأكد من الحقول والعلاقات." |
| 12 | 🔴 قاعدة "الاستنتاج من السياق": قبل افتراض عطل في البنية التحتية، راجع الأدلة الفعلية في السياق. |
| 13 | 🔴 قواعد العمل خلال الجلسة: ابدأ كل رد بـ [#رقم] متسلسل، نفّذ خطوة واحدة واضحة في كل رد. |
| 14 | 🔴 قاعدة التقديرات: التقديرات تُبنى على معامل AI (0.3-0.5) + معدل التطوير الفعلي + التعقيد. |

### 🎯 عند الغموض:

| السؤال | الإجراء |
| :--- | :--- |
| هل هذه الميزة موجودة؟ | اسأل + راجع الـ Dark Areas |
| ما هو الملف الفعلي؟ | اطلب الرؤية |
| ما هي الحقول؟ | اطلب SELECT TOP 1 * FROM Table |
| ما هو النمط المعماري؟ | راجع 01 + 11 + 03 |

---

## 📊 الجزء العاشر: إحصائيات المشروع

| المقياس | القيمة |
| :--- | :--- |
| عدد المشاريع | 9 |
| عدد وثائق docs/ | 22 + ملاحق |
| Shared.UI — مكونات | 68 |
| Shared.UI — ملفات CSS | 120+ |
| Shared.UI — خدمات | 25 |
| Mobile — صفحات XAML | 57 |
| PWA — صفحات Razor | 25+ |
| Api.Web — Controllers | ~35 |
| Api.Web — Endpoints | ~290 |
| Domain — Entities | 150+ |
| Tests — Architecture | 33 |
| نظام النقاط — Handlers | 19 |
| نظام التقارير — Handlers | 8 |

---

## 🕰️ الجزء العاشر-ب: سجل التقديرات التاريخية (Historical Estimates)

| # | النظام (Module) | تاريخ البداية | تاريخ النهاية | الوقت الفعلي | حجم العمل |
| :--- | :--- | :--- | :--- | :--- | :--- |
| 1 | نظام PSP الأساسي | مارس 2026 | سبتمبر 2026 | ~6 أشهر | ~30 ملف |
| 2 | نظام Phase System (PSP) | 23 سبتمبر 2026 | 23 سبتمبر 2026 | ~1 يوم | 2 كيان + 1 Migration |
| 3 | نظام التحاليل (Tests) | 27 سبتمبر 2026 | 4 أكتوبر 2026 | ~7 أيام | 4 كيانات + 38 Endpoint |
| 4 | نظام النقاط (Points) | 29 سبتمبر 2026 | 15 أكتوبر 2026 | ~17 يوم | 8 كيانات + 19 Handler |
| 5 | نظام التقارير (Reports) | 16 أكتوبر 2026 | 16 أكتوبر 2026 | ~1 يوم | 8 Handlers |
| 6 | نظام الإشعارات (Notifications) | 31 مارس 2026 | 16 سبتمبر 2026 | ~5.5 أشهر | 3 أنظمة |
| 7 | نظام القوائم (Menus) | 2 مارس 2026 | 13 أكتوبر 2026 | ~7 أشهر | 4 جداول + 3 خدمات |
| 8 | نظام الترجمة (Translation) | مايو 2026 | سبتمبر 2026 | ~4 أشهر | 3 مسارات + 10 ملفات |
| 9 | إعادة هيكلة الوثائق | 16 أكتوبر 2026 | 18 أكتوبر 2026 | ~2 أيام | 34 → 22 وثيقة |

### 🎯 معادلة التقدير:
`التقدير = (معامل AI) × (المعدل الفعلي) × (التعقيد)`
- معامل AI = 0.3 (للأنظمة الجديدة) إلى 0.5 (للإضافات).

---

## 🎯 الجزء الحادي عشر: القرارات المعمارية الحاسمة

| # | القرار | الوثيقة |
| :--- | :--- | :--- |
| 1 | 9 مشاريع — Clean Architecture | 01 |
| 2 | IGenericService<T> للـ CRUD — UseCase للمعقد | 11 |
| 3 | MenuVisibilityRules — 4 طبقات | 03 |
| 4 | Reusable Services (Points, Notifications, Translation, Messaging) | 17 |
| 5 | SourceEntityID + PointSystemTypeID — لا جدول Campaigns | 06 |
| 6 | TargetBeneficiaryTypeID — لا يحذف | 06 |
| 7 | PointDashboard — واجهة واحدة | 06 |
| 8 | Marketing Website — معزول | 18 |
| 9 | 3 طبقات إشعارات | 08 |
| 10 | PWA = نسخة كربونية من Mobile | 14 |
| 11 | Code-First Migration | 11 |
| 12 | N'' للعربية + MERGE للبيانات | 04 |
| 13 | لا dir على div الجذر | 04 |
| 14 | SourceEntityID + OrganizationTypeFilterID | 06 |
| 15 | ControlLevel + Permission | 19 |
| 16 | PHR: HealthRecord (Aggregation) + SourceEntityID | 21 |
| 17 | PHR: UserProfileID (الشخص) وليس PatientProfileID | 21 |
| 18 | PHR: المريض هو المالك — SharingConsent | 21 |
| **19** | **Integration Hub: IIntegrationClient موحد لكل التكاملات** | **22** |
| **20** | **Integration Hub: Adapters مستقلة (REST, SOAP, FHIR, Webhook)** | **22** |
| **21** | **Integration Hub: كل استدعاء له Audit Log + Idempotency** | **22** |

---

## 🎯 الجزء الثاني عشر: "Stable vs Unstable" Modules

### ✅ Modules مستقرة (لا تلمس)

| # | Module | الوثيقة |
| :--- | :--- | :--- |
| 1 | Identity + Auth | 02 |
| 2 | Menus System | 03 |
| 3 | Translation System | 04 |
| 4 | Caching System | 12 |
| 5 | Google Sign-In | 02 |
| 6 | Deployment | 15 |
| 7 | PSP Core | 05 |
| 8 | Reports | 07 |
| 9 | Marketing Website | 18 |

### 🟡 Modules قابلة للتعديل (بإذن)

| # | Module | السبب |
| :--- | :--- | :--- |
| 1 | نظام النقاط | توسع مستمر |
| 2 | PWA | ~95% |
| 3 | Admin* Pages | LVL 1 → LVL 3 |
| 4 | PSPStep2b | إعادة هيكلة |
| 5 | MessagingHubPage | قيد التطوير |
| 6 | نظام الصلاحيات | Phase 2 + 3 |
| 7 | PHR (جديد) | Phase 1-3 |
| **8** | **Integration Hub (جديد)** | **Phase 1-3** |

### 🔴 Modules غير مستقرة (قيد إعادة الهيكلة)

| # | Module | السبب |
| :--- | :--- | :--- |
| 1 | AdminOrganizationsList | LVL 1 → LVL 3 |
| 2 | AdminIdentity | LVL 1 → LVL 3 |
| 3 | PSPProgramsList | LVL 1 → LVL 3 |
| 4 | إزالة ProgramID المتبقية | جزئياً |

---

## 🎯 الجزء الثالث عشر: كيف تستخدم هذه الوثيقة؟

### سيناريو (1): بداية جلسة جديدة
1. أرسل هذه الوثيقة للـ AI.
2. أرسل الوثيقة المتعلقة بالمهمة.
3. أرسل الملفات المطلوبة.

### سيناريو (2): AI يفترض ميزة غير موجودة
🎯 أرجع للـ Dark Areas + الوثيقة المرتبطة.

### سيناريو (3): تحتاج تحديث الوثيقة
🎯 بعد كل جلسة — حدّث:
- خريطة الجلسات.
- الأولويات.
- Dark Areas.
- Modules Snapshot.

---

## 🔗 الجزء الرابع عشر: روابط ذات صلة

- `01-ARCHITECTURE.md`
- `05-PSP.md`
- `17-STRATEGIC-ROADMAP.md`
- `19-PERMISSIONS.md`
- `20-USER JOURNEY.md`
- `21-PHR-SYSTEM.md` (جديد)
- `22-INTEGRATION-HUB.md` (جديد)
- `00B-PROJECT-BACKLOG.md`

---

## 📝 الجزء الخامس عشر: سجل التحديثات

| الإصدار | التاريخ | التغييرات |
| :--- | :--- | :--- |
| 1.0 | 16 أكتوبر 2026 | الإصدار الأولي |
| 2.0 | 18 أكتوبر 2026 | v2.0 — تحديث الفهرس (20 وثيقة) + 19-PERMISSIONS |
| 2.1 | 26 أكتوبر 2026 | v2.1 — إضافة 21-PHR-SYSTEM + Epic 11 |
| **2.2** | **26 أكتوبر 2026** | **v2.2 — إضافة 22-INTEGRATION-HUB + Epic 12 + Sprint 7-9 + تحديث القرارات المعمارية (19-21)** |

---

© 2026 RubikCare — للاستخدام الداخلي
نهاية الوثيقة ✅
```
