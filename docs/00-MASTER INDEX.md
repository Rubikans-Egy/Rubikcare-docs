# 00 — MASTER INDEX (فهرس روبيك كير الشامل)

الإصدار: 2.1 (Final Architecture Edition)
آخر تحديث: 25 أكتوبر 2026
الحالة: 🟢 مرجع حي — يُحدَّث كل جلسة
الجمهور: فريق التطوير، AI Assistants، القيادة التقنية

---

## 📌 مقدمة

هذه الوثيقة هي البوصلة المعمارية لمشروع RubikCare. الغرض منها:
- فهرس كامل لكل الوثائق (20 وثيقة أساسية + ملاحق).
- حالة المشروع — ما هو موجود / مخطط / قيد التنفيذ.
- خريطة الجلسات — ما تم إنجازه في كل جلسة.
- مناطق مظلمة (Dark Areas) — ما هو منفَّذ لكن غير موثَّق.
- قواعد صارمة للـ AI Assistant.

🎯 القاعدة الذهبية:
"قبل أي جلسة تطوير — أرسل هذه الوثيقة للـ AI. عندها سيعرف ما هو موجود — وما لا ينبغي أن يفترض أنه غير موجود."

---

## 📚 الجزء الأول: فهرس الوثائق الكامل (الهيكل النهائي المُوحَّد)

### الوثائق الأساسية (00-19)

| # | الوثيقة | الموضوع | آخر تحديث | الحالة |
|---|---------|---------|-----------|--------|
| 00 | 00-MASTER-INDEX.md | الفهرس الشامل (هذه الوثيقة) | 25 أكتوبر 2026 | ✅ |
| 00B | 00B-PROJECT-BACKLOG.md | سجل المهام الحية (يدمج 18, 23) | 25 أكتوبر 2026 | ✅ |
| 01 | 01-ARCHITECTURE.md | الهيكل المعماري + Program.cs (يدمج 00, 01, 24) | 17 أكتوبر 2026 | ✅ |
| 02 | 02-IDENTITY.md | الهوية والمصادقة (يدمج 02, 16) | 17 أكتوبر 2026 | ✅ |
| 03 | 03-MENUS.md | نظام القوائم الديناميكية (يدمج 04, 17, 19) | 16 أكتوبر 2026 | ✅ |
| 04 | 04-TRANSLATION.md | الترجمة والاتجاه (يدمج 15, 17, 19) | 16 أكتوبر 2026 | ✅ |
| 05 | 05-PSP.md | نظام دعم المرضى (يدمج 07, User Flow) | 17 أكتوبر 2026 | ✅ |
| 06 | 06-POINTS.md | نظام النقاط وروبيكا (يدمج 26, Changelog) | 17 أكتوبر 2026 | ✅ |
| 07 | 07-REPORTS.md | نظام التقارير (يدمج 27) | 17 أكتوبر 2026 | ✅ |
| 08 | 08-NOTIFICATIONS.md | الإشعارات والتذكيرات (يدمج 20, 21) | 17 أكتوبر 2026 | ✅ |
| 09 | 09-API-GUIDE.md | دليل API الشامل (~290 Endpoint) | 16 أكتوبر 2026 | ✅ |
| 10 | 10-UI-STYLE-GUIDE.md | دليل الأنماط (يدمج 03, 25) | 17 أكتوبر 2026 | ✅ |
| 11 | 11-PAGE-CREATION.md | إنشاء الصفحات (يدمج 05, 13) | 17 أكتوبر 2026 | ✅ |
| 12 | 12-CACHING.md | نظام الكاش (يدمج 14) | 17 أكتوبر 2026 | ✅ |
| 13 | 13-MAUI.md | تطوير MAUI (يدمج 10, 11) | **25 أكتوبر 2026** | ✅ (الجزء 20 — جديد) |
| 14 | 14-PWA.md | تطبيق الويب التقدمي (يدمج 22, 24) | 17 أكتوبر 2026 | ✅ |
| 15 | 15-DEPLOYMENT.md | دليل النشر (يدمج 12) | 17 أكتوبر 2026 | ✅ |
| 16 | 16-TROUBLESHOOTING.md | المنهجية + قواعد AI (يدمج 06, 18) | 17 أكتوبر 2026 | ✅ |
| 17 | 17-STRATEGIC-ROADMAP.md | الدستور التطويري (يدمج 08) | 17 أكتوبر 2026 | ✅ |
| 18 | 18-MARKETING-WEBSITE.md | الموقع التسويقي (يدمج 25) | 17 أكتوبر 2026 | ✅ |
| 19 | 19-PERMISSIONS.md | نظام الصلاحيات الكامل (جديد) | 16 أكتوبر 2026 | ✅ |

### الملاحق

| # | الوثيقة | الحالة |
|---|---------|--------|
| A | appendix-a-glossary.md | ✅ (يحتاج تحديث طفيف) |
| B | appendix-b-service-index.md | ✅ (يحتاج تحديث طفيف) |

---

## 🏗️ الجزء الثاني: بنية المشروع (9 مشاريع)

| # | المشروع | التقنية | المسؤولية | الحالة |
|---|---------|---------|-----------|--------|
| 1 | RubikCare.Domain | .NET 10 Class Library | الكيانات + Enums + Constants | ✅ |
| 2 | RubikCare.Application | .NET 10 Class Library | Use Cases + DTOs + Interfaces | ✅ |
| 3 | RubikCare.Infrastructure | .NET 10 + EF Core 10 | DbContext + Repositories + Migrations | ✅ |
| 4 | Api.Web | ASP.NET Core 10 | REST API + Controllers (~290 Endpoint) | ✅ |
| 5 | Rubikcare.Web | Blazor Server 10 | تطبيق الويب (للشركة + المنصة) | ✅ |
| 6 | RubikCare.PWA | Blazor WebAssembly 10 | نسخة كربونية من Mobile للمتصفح | 🟠 ~95% |
| 7 | Mobile | .NET MAUI 10 | تطبيق الموبايل (Android + iOS) | ✅ |
| 8 | Shared.UI | Razor Class Library | 68 مكون مشترك + 25 خدمة | ✅ |
| 9 | RubikCare.Tests | xUnit + Moq | 33 اختبار (معماري + وحدة) | ✅ |

---

## 🎯 الجزء الثالث: الوضع الحالي (Modules Snapshot)

### ✅ Modules مكتملة ومستقرة

| # | Module | Backend | Frontend | الوثيقة |
|---|--------|---------|----------|---------|
| 1 | الهوية والمصادقة | ✅ | ✅ | 02 |
| 2 | القوائم الديناميكية (4 طبقات) | ✅ | ✅ | 03 |
| 3 | الترجمة والاتجاه (Web + Marketing) | ✅ | ✅ | 04 |
| 4 | نظام PSP + Phase System | ✅ | ✅ | 05 |
| 5 | نظام التقارير (6 + Dashboard) | ✅ | ✅ | 07 |
| **6** | **نظام النقاط (Points)** | **✅** | **✅** | **06** |
| 7 | Google Sign-In | ✅ | ✅ | 02 |
| 8 | الكاش الموحد | ✅ | ✅ | 12 |
| 9 | الإشعارات (FCM + Local + Web Push) | ✅ | ✅ | 08 |
| 10 | الموقع التسويقي (8 صفحات) | ✅ | ✅ | 18 |
| 11 | نظام الصلاحيات (Schema) | ✅ | ❌ | 19 |
| **12** | **Patient Mobile (MAUI)** | **✅** | **✅** | **13** |

### 🟠 Modules مكتملة جزئياً

| # | Module | ما ينقص |
|---|--------|---------|
| 1 | PWA | MessagingHubPage + 3 لوحات مهنية |
| 2 | نظام الصلاحيات | الـ Middleware + الـ UI (Backend جاهز) |
| 3 | طلبات شراء الأدوية | تكامل مع نظام النقاط |

### ⏳ Modules مُخطَّطة (لم تبدأ)

| # | Module | الرؤية | الوثيقة |
|---|--------|--------|---------|
| 1 | B2B Commerce | Supply Chain كامل | 17 |
| 2 | B2C Pharmacy | تجارة التجزئة | 17 |
| 3 | Medical CRM | KPI's للمندوبين | 17 |
| 4 | Clinic OS | إدارة العيادات (SaaS) | 17 |
| 5 | CME | التعليم الطبي المستمر | 17 |
| 6 | Rubika Currency Engine | عملة داخلية موازية | 06 |
| 7 | Medical Tourism | السياحة العلاجية | 17 |

---

## 🔴 الجزء الرابع: المناطق المظلمة (Dark Areas)

🎯 هذه ميزات مُنفَّذة لكن غير موثَّقة أو موثَّقة جزئياً.

| # | الميزة | الموقع | التوثيق |
|---|--------|--------|---------|
| 1 | PSPEnrollmentService | Application/Services/PSP/ | ✅ 05 |
| 2 | DispenseController | Api.Web/Controllers/ | ✅ 09 (v2.0) |
| 3 | MenuVisibilityRules UI | مفقود | 03 يذكر الجدول فقط |
| 4 | OrganizationGroups | تأسيس فقط | ✅ 03 |
| 5 | Rubika Currency Schema | DB فقط | ✅ 06 |
| 6 | Messaging System | ✅ كامل | 14 PWA فقط |
| 7 | File Uploads | ✅ كامل | — |
| 8 | Booking System | ✅ كامل | 14 |
| 9 | Nearby Pharmacies | ✅ كامل | 14 |
| 10 | CustomBlazorWebView | ✅ كامل | ✅ 13 |
| 11 | Marketing Website — Header/Footer | ✅ كامل | ✅ 18 |
| **12** | **`PspDetailNavigationBridge`** | **✅ كامل** | **✅ 13 (الجزء 20)** |
| **13** | **`MedicationScheduleFlow` (MAUI)** | **✅ كامل** | **✅ 13 (الجزء 20)** |
| **14** | **`PatientPointsFlow` (MAUI)** | **✅ كامل** | **✅ 13 (الجزء 20)** |

🎯 الإجراء المطلوب: عند الجلسات القادمة — اسأل عن الموقع الفعلي لكل ميزة قبل الافتراض.

---

## 📅 الجزء الخامس: خريطة الجلسات

| الجلسة | التاريخ | ما تم | الوثائق المُحدَّثة |
|--------|---------|-------|-------------------|
| #104-108 | 10-13 أكتوبر 2026 | Campaign Reference + SourceEntityID | 06 |
| #109-115 | 14-15 أكتوبر 2026 | Loyalty UI + Reports + Beneficiaries | 06 + 07 |
| #116-121 | 16 أكتوبر 2026 | جرد شامل + الفهرس v1.0 | 00 |
| #122-130 | 16-17 أكتوبر 2026 | إعادة هيكلة الوثائق (من 34 إلى 20) | جميع الوثائق |
| #131 | 17 أكتوبر 2026 | دمج 04-TRANSLATION (v3.1) + تصحيح 08 | 04 + 08 |
| #132 | 18 أكتوبر 2026 | إصدار 00-MASTER-INDEX v2.0 (هذه الوثيقة) | 00 |
| **#136** | **22-23 أكتوبر 2026** | **إغلاق Sprint 1 (Points-PSP Integration)** | **00 + 00B + 01 + 05 + 06 + 20** |
| **#137-138** | **24-25 أكتوبر 2026** | **إغلاق Sprint 2 (MAUI Stabilization + Integration)** | **00 + 00B + 13** |

---

## 🚨 الجزء السادس: الأولويات الحالية

### 🔴 P1 — حرجة

| # | المهمة | السبب | الوثيقة |
|---|--------|-------|---------|
| 1 | اختبار E2E كامل (نظام النقاط + MAUI) | قبل التوسع | 00B (Sprint 3) |
| 2 | إكمال `PspDetailFlow` — ربط `GoHome` + `ShareToken` + `CopyToken` | بعض الأزرار لا تعمل في MAUI | 00B (Sprint 3) |
| 3 | إكمال `PspSchedulePageFlow` — تمرير `PatientId` | `PspSchedulePageFlow` لا يمرر `PatientId` | 00B (Sprint 3) |
| 4 | حل مشكلة mscorlib.dll (إذا لا تزال) | توقف Debugger | 16 |

### 🟠 P2 — عالية

| # | المهمة | الوثيقة |
|---|--------|---------|
| 1 | ربط النقاط بالتقارير (ProgramLifecycle + PharmaDashboard) | 06 + 07 |
| 2 | تقرير النقاط المخصص (PointsReport) | 07 |
| 3 | مستويات الولاء (Loyalty Tiers) | 06 |
| 4 | إكمال P5.1 → P5.3 (التقارير) | 00B (Sprint 3) |
| 5 | الترجمة المتبقية (P7.1) | 00B (Sprint 3) |

### 🟡 P3 — متوسطة

| # | المهمة | الوثيقة |
|---|--------|---------|
| 1 | تنفيذ 19-PERMISSIONS (Backend + UI) | 19 |
| 2 | إعادة هيكلة Admin* (LVL 1 → LVL 3) | 11 |
| 3 | MenuVisibilityRules UI | 03 |
| 4 | MessagingHubPage في PWA | 14 |
| 5 | لوحات PWA المهنية (Clinic, Pharmacy, Rep) | 14 |

---

## 🗺️ الجزء السابع: خريطة الجلسات المستقبلية

| الجلسة | الموضوع | المتوقع | الوثيقة |
|--------|---------|---------|---------|
| القادمة | 19-PERMISSIONS (Phase 2 + 3) | Middleware + UI | 19 |
| +2 | ربط Menus + Services + Subscriptions | Super Admin UI | 03 |
| +3 | B2B Commerce — MVP | ربط النقاط بالتوريد | 17 |
| +4 | B2C Pharmacy | تجارة التجزئة | 17 |
| +5 | Medical CRM | KPI's للمندوبين | 17 |
| +6 | Clinic OS | إدارة العيادات | 17 |

---

## 📋 الجزء الثامن: خريطة النشر

### البيئات

| البيئة | Web | API | PWA |
|--------|-----|-----|-----|
| Local | https://localhost | https://localhost | https://localhost |
| Test/UAT | test.rubikcare.com | uat.rubikcare.com | stagepu.rubikcare.com |
| Production | rubikcare.com | api.rubikcare.com | (منشور تلقائياً) |

### IIS Sites على VPS

| الموقع | الدومين | App Pool |
|--------|---------|----------|
| RubikCareNew | test.rubikcare.com | rubikcarenew |
| RubikCareUat | uat.rubikcare.com | RubikCareUat |
| PU_RubicCareStage | stagepu.rubikcare.com | PU_RubicCareStage |
| RubikCareLive | rubikcare.com | — |
| RubikCareApi | api.rubikcare.com | — |

🎯 القاعدة الذهبية: راجع 15-DEPLOYMENT.md قبل أي نشر.

---

## 🧠 الجزء التاسع: قواعد صارمة للـ AI Assistant

### قبل أي جلسة — اطلب:
1. هذه الوثيقة (00-MASTER-INDEX.md).
2. الوثيقة المتعلقة بالمهمة (03, 05, 06, 07, 19 — حسب الحالة).

### 🎯 القواعد الإلزامية (16 قاعدة):

| # | القاعدة |
|---|---------|
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
| **15** | **🔴 قاعدة `JavaProxyThrowable`: ابحث عن `Exception` الحقيقي في `Output Window` — وليس `JavaProxyThrowable` نفسه.** |
| **16** | **🔴 قاعدة `[QueryProperty]`: استقبل `string` من الطرفين (المرسل + المستقبل) — لتفادي `InvalidCastException`.** |

### 🎯 عند الغموض:

| السؤال | الإجراء |
|--------|---------|
| هل هذه الميزة موجودة؟ | اسأل + راجع الـ Dark Areas |
| ما هو الملف الفعلي؟ | اطلب الرؤية |
| ما هي الحقول؟ | اطلب SELECT TOP 1 * FROM Table |
| ما هو النمط المعماري؟ | راجع 01 + 11 + 03 |

---

## 📊 الجزء العاشر: إحصائيات المشروع

| المقياس | القيمة |
|---------|--------|
| عدد المشاريع | 9 |
| عدد وثائق docs/ | 20 + ملاحق (بعد الدمج) |
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
| نظام التقارير — Pages | 7 |

---

## 🕰️ الجزء العاشر-ب: سجل التقديرات التاريخية (Historical Estimates)
⚠️ ملاحظة: هذا الجدول يُوثّق الوقت الفعلي الذي استغرقته الأنظمة الرئيسية، بناءً على تواريخ الـ Migrations والملفات. الهدف: بناء نموذج تقديري دقيق للجلسات المستقبلية.

| # | النظام (Module) | تاريخ البداية | تاريخ النهاية | الوقت الفعلي | حجم العمل | ملاحظات |
|---|---|---|---|---|---|---|
| 1 | نظام PSP الأساسي | مارس 2026 | سبتمبر 2026 | ~6 أشهر | ~30 ملف (Domain + API) | شمل إعادة هيكلة شاملة + Phase System لاحقاً |
| 2 | نظام Phase System (PSP) | 23 سبتمبر 2026 | 23 سبتمبر 2026 | ~1 يوم | 2 كيان + 1 Migration + 6 Endpoints | تم بسرعة لأن الـ Backend جاهز |
| 3 | نظام التحاليل (Tests System) | 27 سبتمبر 2026 | 4 أكتوبر 2026 | ~7 أيام | 4 كيانات + 38 Endpoint + 8 UI | الأكبر في حجم الـ Endpoints |
| 4 | نظام النقاط (Points) | 29 سبتمبر 2026 | 15 أكتوبر 2026 | ~17 يوم | 8 كيانات + 19 Handler + 5 Controllers + 4 Migrations | شمل إعادة تصميم معماري (Campaign Ref) |
| 5 | نظام التقارير (Reports) | 16 أكتوبر 2026 | 16 أكتوبر 2026 | ~1 يوم | 8 Handlers + 1 Controller + 6 Pages + 30 DTO | بُني على DbContextFactory |
| 6 | نظام الإشعارات (Notifications) | 31 مارس 2026 | 16 سبتمبر 2026 | ~5.5 أشهر | 3 أنظمة (FCM + Local + Web Push) | تطوير متقطع (ليس متصلاً) |
| 7 | نظام القوائم (Menus) | 2 مارس 2026 | 13 أكتوبر 2026 | ~7 أشهر | 4 جداول + 3 خدمات + UI | تطوير متقطع (انتظار الاشتراكات) |
| 8 | نظام الترجمة (Translation) | مايو 2026 | سبتمبر 2026 | ~4 أشهر | 3 مسارات + 10 ملفات | تطوير متقطع (حل مشاكل SSR) |
| 9 | إعادة هيكلة الوثائق | 16 أكتوبر 2026 | 18 أكتوبر 2026 | ~2 أيام | 34 وثيقة → 20 وثيقة | جهد مكثف (AI + مراجعات) |
| **10** | **نظام النقاط + PSP (Sprint 1)** | **22 أكتوبر 2026** | **23 أكتوبر 2026** | **~2 أيام** | **12 مهمة (36 SP) + 6 إضافية (10 SP)** | **شمل إصلاح 9 عقود تكامل + حل 3 مشاكل جذرية** |
| **11** | **MAUI Stabilization (Sprint 2)** | **24 أكتوبر 2026** | **25 أكتوبر 2026** | **~2 أيام** | **13 مهمة (23 SP) + 4 إضافية (4 SP)** | **حل `JavaProxyThrowable` + `patientId=0` + `InvalidCastException`** |

### 🎯 كيف نستخدم هذا الجدول للتقديرات المستقبلية؟

المعادلة: التقدير = (معامل AI) × (المعدل الفعلي) × (التعقيد)
المعدل الفعلي = (الوقت الفعلي / عدد الملفات)
معامل AI = 0.3 (للأنظمة الجديدة) إلى 0.5 (للإضافات)

**أمثلة تطبيقية:**
- إغلاق النقاط (7 صفحات UI): الوقت الفعلي لنظام مشابه = 1 يوم / 8 صفحات → ~1.5 يوم.
- نظام الصلاحيات (Domain + 6 Migrations): الوقت الفعلي لنظام Phase = 1 يوم / 2 كيان → ~5 أيام.
- أتمتة النشر (Scripts): الوقت الفعلي = ~1 يوم (إذا استخدمنا السكريبتات الموجودة).

---

## 🎯 الجزء الحادي عشر: القرارات المعمارية الحاسمة

| # | القرار | الوثيقة |
|---|--------|---------|
| 1 | 9 مشاريع — Clean Architecture | 01 |
| 2 | IGenericService<T> للـ CRUD — UseCase للمعقد | 11 |
| 3 | MenuVisibilityRules — 4 طبقات | 03 |
| 4 | Reusable Services (Points, Notifications, Translation, Messaging) | 17 |
| 5 | SourceEntityID + PointSystemTypeID — لا جدول Campaigns | 06 |
| 6 | TargetBeneficiaryTypeID — لا يحذف. يُدار في LoyaltySchemeEdit | 06 |
| 7 | PointDashboard — واجهة واحدة لكل المستفيدين | 06 |
| 8 | Marketing Website — معزول عن التطبيق | 18 |
| 9 | 3 طبقات إشعارات (FCM + WebPush + Local) | 08 |
| 10 | PWA = نسخة كربونية من Mobile | 14 |
| 11 | Code-First Migration — لا تعديل يدوي | 11 |
| 12 | N'' للعربية + MERGE للبيانات | 04 |
| 13 | لا dir على div الجذر — على <html> فقط | 04 |
| 14 | SourceEntityID + OrganizationTypeFilterID (14 أكتوبر) | 06 |
| 15 | ControlLevel + Permission (نظام الصلاحيات الجديد) | 19 |
| **16** | **`JavaProxyThrowable` = غلاف — ابحث عن `Exception` الحقيقي في `Output Window`** | **13 (الجزء 20)** |
| **17** | **`[QueryProperty]` — `string` من الطرفين (المرسل + المستقبل)** | **13 (الجزء 20)** |
| **18** | **Bridge — ربط كل الدوال في `OnAppearing` + `Dispatcher.DispatchAsync`** | **13 (الجزء 20)** |

---

## 🎯 الجزء الثاني عشر: "Stable vs Unstable" Modules

### ✅ Modules مستقرة (لا تلمس)

| # | Module | الوثيقة |
|---|--------|---------|
| 1 | Identity + Auth | 02 |
| 2 | Menus System | 03 |
| 3 | Translation System | 04 |
| 4 | Caching System | 12 |
| 5 | Google Sign-In | 02 |
| 6 | Deployment | 15 |
| 7 | PSP Core | 05 |
| 8 | Reports | 07 |
| 9 | Marketing Website | 18 |
| **10** | **Points System** | **06** |
| **11** | **Notifications System** | **08** |

### 🟡 Modules قابلة للتعديل (بإذن)

| # | Module | السبب |
|---|--------|-------|
| 1 | PWA | ~95% — قيد الإكمال |
| 2 | Admin* Pages | LVL 1 → LVL 3 |
| 3 | PSPStep2b | إعادة هيكلة |
| 4 | MessagingHubPage | قيد التطوير |
| 5 | نظام الصلاحيات | Phase 2 + 3 |
| **6** | **`PspDetailFlow` (MAUI)** | **ربط `GoHome` + `ShareToken` + `CopyToken`** |
| **7** | **`PspSchedulePageFlow` (MAUI)** | **تمرير `PatientId`** |

### 🔴 Modules غير مستقرة (قيد إعادة الهيكلة)

| # | Module | السبب |
|---|--------|-------|
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
- خريطة الجلسات (الجزء الخامس).
- الأولويات (الجزء السادس).
- Dark Areas (الجزء الرابع).
- Modules Snapshot (الجزء الثالث).

---

## 🔗 الجزء الرابع عشر: روابط ذات صلة

- [00B-PROJECT-BACKLOG](00B-PROJECT-BACKLOG.md)
- [01-ARCHITECTURE](01%20%E2%80%94%20ARCHITECTURE%20.md)
- [05-PSP-SYSTEM](05-psp-system.md)
- [06-POINTS](06-points.md)
- [13-MAUI](13-maui-development-guide.md)

---

## 📝 الجزء الخامس عشر: سجل التحديثات

| الإصدار | التاريخ | التغييرات |
|---------|---------|-----------|
| 1.0 | 16 أكتوبر 2026 | الإصدار الأولي — فهرس شامل + Snapshot + Dark Areas + خريطة الجلسات |
| 2.0 | 18 أكتوبر 2026 | 00-MASTER-INDEX v2.0 — تحديث الفهرس ليعكس الهيكل النهائي (20 وثيقة بدل 34) + إضافة 19-PERMISSIONS + تحديث القواعد للـ 14 + تحديث الأولويات + Modules Snapshot |
| **2.1** | **25 أكتوبر 2026** | **v2.1 — إغلاق Sprint 1 (Points-PSP) + Sprint 2 (MAUI) + 5 دروس مستفادة + 2 قاعدة جديدة (15, 16) + تحديث Dark Areas (3 ميزات MAUI) + تحديث Stable vs Unstable (3 موديولات) + تحديث خريطة الجلسات (جلستين جديدتين) + تحديث الأولويات + تحديث جدول التقديرات (Sprint 1 + 2)** |
---


## 📖 الجزء السادس عشر: قاموس الاختصارات (Glossary)

هذا القاموس يشرح كل الاختصارات المستخدمة في `00B-PROJECT-BACKLOG.md` والوثائق الأخرى.

### 🎯 اختصارات التخطيط (Planning)

| الاختصار | المعنى (عربي) | المعنى (إنجليزي) | مثال |
|-----------|----------------|-------------------|------|
| **SP** | **نقطة قصة** — وحدة قياس حجم المهمة. | **Story Point** | P5.1 = 2 SP |
| **SP (Sprint)** | **نقاط Sprint** — مجموع نقاط Sprint. | **Sprint Points** | Sprint 1 = 36 SP |
| **P0** | **أولوية قصوى** — حرجة (يجب أن تُنجز فوراً). | **Priority 0** | S3-P0.1 = 3 SP |
| **P1** | **أولوية عالية** — مهمة ولكن ليست حرجة. | **Priority 1** | P1 = 2 SP |
| **P2** | **أولوية متوسطة** — يمكن تأجيلها. | **Priority 2** | P2 = 5 SP |
| **P3** | **أولوية منخفضة** — تحسينات. | **Priority 3** | P7.1 = 1 SP |
| **ETA** | **الوقت المتوقع للإنجاز** — تاريخ التسليم. | **Estimated Time of Arrival** | ETA: 26 أكتوبر |
| **E2E** | **اختبار شامل** — من البداية للنهاية. | **End-to-End** | P7.2 = E2E |

### 🎯 اختصارات Sprint (Sprint Markers)

| الاختصار | المعنى (عربي) | المعنى (إنجليزي) | مثال |
|-----------|----------------|-------------------|------|
| **M1, M2, ...** | **مرحلة/مهمة** — ترتيب المهمة في Sprint 2. | **Milestone** | M1 = إنشاء `PatientPointsFlow` |
| **D1, D2, ...** | **مهمة منجزة** — إضافية (خارج Sprint). | **Done** | D1 = IRefreshService |
| **P5.1, P5.2** | **المرحلة 5 — المهمة 1** — مهام فرعية. | **Phase 5 — Task 1** | P5.1 = GetAdherenceHandler |
| **S3-P0.1** | **Sprint 3 — الأولوية 0 — المهمة 1** | **Sprint 3 — P0 — Task 1** | S3-P0.1 = Deploy-FromStage.ps1 |

### 🎯 اختصارات التكامل (Integration)

| الاختصار | المعنى (عربي) | المعنى (إنجليزي) | مثال |
|-----------|----------------|-------------------|------|
| **IC-01 → IC-09** | **عقود التكامل** — 9 عقود بين نظام النقاط و PSP. | **Integration Contracts** | IC-04 = خصم النقاط على الصرف |
| **PSP** | **برامج دعم المرضى** | **Patient Support Programs** | PSPProgram |
| **PSPeRX** | **وصفة إلكترونية** | **PSP Electronic Prescription** | PSPeRX |
| **eRx** | **وصفة إلكترونية** | **Electronic Prescription** | eRxID |
| **DTO** | **كائن نقل البيانات** | **Data Transfer Object** | DispensationScheduleDto |
| **DI** | **حقن التبعية** | **Dependency Injection** | `builder.Services.Add...` |
| **DbContext** | **سياق قاعدة البيانات** | **Database Context** | BusinessDbContext |

### 🎯 اختصارات النظام (System)

| الاختصار | المعنى (عربي) | المعنى (إنجليزي) | مثال |
|-----------|----------------|-------------------|------|
| **PWA** | **تطبيق ويب تقدمي** | **Progressive Web App** | RubikCare.PWA |
| **MAUI** | **واجهة تطبيقات متعددة المنصات** | **Multi-platform App UI** | RubikCare.Mobile |
| **RCL** | **مكتبة فئات Razor** | **Razor Class Library** | Shared.UI |
| **EF Core** | **إطار كيان** | **Entity Framework Core** | BusinessDbContext |
| **SQL** | **لغة الاستعلامات** | **Structured Query Language** | `SELECT * FROM ...` |
| **API** | **واجهة برمجة التطبيقات** | **Application Programming Interface** | Api.Web |

### 🎯 اختصارات التطوير (Development)

| الاختصار | المعنى (عربي) | المعنى (إنجليزي) | مثال |
|-----------|----------------|-------------------|------|
| **UI** | **واجهة المستخدم** | **User Interface** | Shared.UI |
| **UX** | **تجربة المستخدم** | **User Experience** | PspDetailComponent |
| **DB** | **قاعدة البيانات** | **Database** | RubikCare_Dev_V2 |
| **IIS** | **خادم معلومات الإنترنت** | **Internet Information Services** | App Pools |
| **VPS** | **خادم افتراضي خاص** | **Virtual Private Server** | 161.97.105.83 |
| **CI/CD** | **التكامل/النشر المستمر** | **Continuous Integration/Deployment** | (مستقبلاً) |
| **SSR** | **العرض من الخادم** | **Server-Side Rendering** | Blazor Server |
| **WASM** | **WebAssembly** | **WebAssembly** | PWA |

### 🎯 اختصارات المجال (Domain)

| الاختصار | المعنى (عربي) | المعنى (إنجليزي) | مثال |
|-----------|----------------|-------------------|------|
| **PSP** | **برامج دعم المرضى** | **Patient Support Programs** | PSPProgram |
| **Loyalty** | **الولاء** — نظام النقاط والمكافآت. | **Loyalty** | LoyaltyScheme |
| **Scheme** | **نظام نقاط** — قواعد كسب/استبدال النقاط. | **Point Scheme** | PSPProgramPointScheme |
| **Balance** | **رصيد** — رصيد نقاط المستفيد. | **Balance** | PSPProgramPointBalance |
| **Rule** | **قاعدة** — قاعدة منح نقاط (حدث + مستفيد + نقاط). | **Rule** | PSPProgramPointRule |
| **Beneficiary** | **مستفيد** — من يحصل على النقاط (مريض، طبيب، منظمة). | **Beneficiary** | BeneficiaryTypeID = 5 (Patient) |

### 🎯 اختصارات الاختبار (Testing)

| الاختصار | المعنى (عربي) | المعنى (إنجليزي) | مثال |
|-----------|----------------|-------------------|------|
| **E2E** | **اختبار شامل** | **End-to-End** | P7.2 = E2E |
| **Unit Test** | **اختبار وحدة** | **Unit Test** | RubikCare.Tests |
| **Moq** | **مكتبة محاكاة** | **Mocking Library** | `new Mock<IApiService>()` |
| **xUnit** | **إطار اختبار** | **xUnit Framework** | `[Fact]` |

### 🎯 اختصارات مكافحة الأخطاء (Troubleshooting)

| الاختصار | المعنى (عربي) | المعنى (إنجليزي) | مثال |
|-----------|----------------|-------------------|------|
| **JavaProxyThrowable** | **استثناء Java** — غلاف يخفي `Exception` الحقيقي. | **JavaProxyThrowable** | الجزء 20.1 |
| **NullReferenceException** | **استثناء مرجع فارغ** | **Null Reference Exception** | — |
| **InvalidCastException** | **استثناء تحويل غير صالح** | **Invalid Cast Exception** | QueryProperty |
| **Forbidden (403)** | **ممنوع** — المستخدم لا يملك صلاحية. | **Forbidden** | `PspController` |
| **BadRequest (400)** | **طلب غير صالح** — البيانات ناقصة. | **Bad Request** | `ValidateToken` |
| **InternalServerError (500)** | **خطأ داخلي** — خطأ في الكود. | **Internal Server Error** | `try/catch` |

### 🎯 اختصارات الأنساب (Naming)

| الاختصار | المعنى (عربي) | المعنى (إنجليزي) | مثال |
|-----------|----------------|-------------------|------|
| **Entity** | **كيان** — جدول قاعدة بيانات. | **Entity** | `PSPPatient` |
| **DTO** | **كائن نقل البيانات** | **Data Transfer Object** | `DispensationScheduleDto` |
| **Service** | **خدمة** — منطق أعمال. | **Service** | `PatientMedicationService` |
| **Handler** | **معالج** — حالة استخدام. | **Handler** | `GetRemindersHandler` |
| **Controller** | **متحكم** — Endpoint API. | **Controller** | `DispenseController` |
| **Repository** | **مستودع** — طبقة الوصول للبيانات. | **Repository** | `PatientMedicationScheduleRepository` |
| **Migration** | **هجرة** — تغيير قاعدة البيانات. | **Migration** | `RenamePatientIdToCaseId` |

### 🎯 اختصارات Bridges (MAUI)

| الاختصار | المعنى (عربي) | المعنى (إنجليزي) | مثال |
|-----------|----------------|-------------------|------|
| **Bridge** | **جسر** — `static Action` للتنقل بين MAUI و Blazor. | **Bridge** | `PspDetailNavigationBridge` |
| **RootComponent** | **مكون جذري** — المكون الذي يُحمَّل في BlazorWebView. | **Root Component** | `PspDetailComponent` |
| **Wrapper** | **غلاف** — صفحة MAUI تستدعي مكون Blazor. | **Wrapper** | `PspDetailFlow` |
| **QueryProperty** | **خاصية استعلام** — معامل يُمرَّر عبر URL. | **Query Property** | `[QueryProperty(nameof(PatientIdRaw), "patientId")]` |

### 📌 ملاحظة أخيرة

**هذا القاموس مرجع حي** — إذا ظهر اختصار جديد في `00B` أو أي وثيقة، **أضفه هنا** (مع الحفاظ على الترتيب الهجائي أو الموضوعي).
© 2026 RubikCare — للاستخدام الداخلي

نهاية الوثيقة ✅
