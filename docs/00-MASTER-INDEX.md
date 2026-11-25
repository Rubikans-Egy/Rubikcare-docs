
# 00 — MASTER INDEX (فهرس روبيك كير الشامل)

**الإصدار:** 3.0 (Realigned Edition — Sprint Plan v8.0)
**آخر تحديث:** 25 نوفمبر 2026
**الحالة:** 🟢 مرجع حي — يُحدَّث كل جلسة
**الجمهور:** فريق التطوير، AI Assistants، القيادة التقنية

---

## 📌 مقدمة

هذه الوثيقة هي **البوصلة المعمارية** لمشروع RubikCare. الغرض منها:

- فهرس كامل لكل الوثائق (22 وثيقة أساسية + ملاحق).
- حالة المشروع — ما هو موجود / مخطط / قيد التنفيذ.
- خريطة الجلسات — ما تم إنجازه في كل جلسة.
- مناطق مظلمة (Dark Areas) — ما هو منفَّذ لكن غير موثَّق.
- قواعد صارمة للـ AI Assistant.
- **خريطة الفجوات (Gaps Register)** — ما هو مخطط لكن غير منفّذ.

🎯 **القاعدة الذهبية:** "قبل أي جلسة تطوير — أرسل هذه الوثيقة للـ AI. عندها سيعرف ما هو موجود — وما لا ينبغي أن يفترض أنه غير موجود."

🎯 **القاعدة الجديدة:** "**الوثيقة تطابق الكود.** أي تعارض بين الوثيقة والكود يُحلّ لصالح الكود، والوثيقة تُحدَّث لتطابق الواقع."

---

## 📚 الجزء الأول: فهرس الوثائق الكامل

### الوثائق الأساسية (00-24)

| # | الوثيقة | الموضوع | آخر تحديث | الحالة |
|---|---|---|---|---|
| 00 | `00-MASTER-INDEX.md` | الفهرس الشامل (هذه الوثيقة) | 25 نوفمبر 2026 | ✅ |
| 00B | `00B-PROJECT-BACKLOG.md` | سجل المهام الحية (v8.0) | 25 نوفمبر 2026 | ✅ |
| 00C | `00C-CHANGELOG.md` | كل ما تم إنجازه | — | ⏳ يحتاج إنشاء |
| 00D | `00D-ROADMAP.md` | الجدول الزمني الكلي | — | ⏳ يحتاج إنشاء |
| 01 | `01-ARCHITECTURE.md` | الهيكل المعماري + Program.cs (v2.2) | 24 نوفمبر 2026 | ✅ |
| 02 | `02-IDENTITY.md` | الهوية والمصادقة | 17 أكتوبر 2026 | ✅ |
| 03 | `03-MENUS.md` | نظام القوائم الديناميكية | 16 أكتوبر 2026 | ✅ |
| 04 | `04-TRANSLATION.md` | الترجمة والاتجاه | 16 أكتوبر 2026 | ✅ |
| 05 | `05-psp-system.md` | نظام دعم المرضى (v2.1) | 24 نوفمبر 2026 | ✅ |
| 06 | `06-POINTS.md` | نظام النقاط وروبيكا | 17 أكتوبر 2026 | ✅ |
| 07 | `07-REPORTS.md` | نظام التقارير | 17 أكتوبر 2026 | ✅ |
| 08 | `08-NOTIFICATIONS.md` | الإشعارات والتذكيرات | 17 أكتوبر 2026 | 🟡 (S09 نشط) |
| 09 | `09-API-GUIDE.md` | دليل API الشامل (~337 Endpoint) | 25 نوفمبر 2026 | 🟡 (يحتاج تحديث +17) |
| 10 | `10-UI-STYLE-GUIDE.md` | دليل الأنماط | 17 أكتوبر 2026 | ✅ |
| 11 | `11-PAGE-CREATION.md` | إنشاء الصفحات | 17 أكتوبر 2026 | ✅ |
| 12 | `12-CACHING.md` | نظام الكاش | 17 أكتوبر 2026 | ✅ |
| 13 | `13-MAUI.md` | تطوير MAUI | 17 أكتوبر 2026 | ✅ |
| 14 | `14-PWA.md` | تطبيق الويب التقدمي | 17 أكتوبر 2026 | ✅ |
| 15 | `15-DEPLOYMENT.md` | دليل النشر | 17 أكتوبر 2026 | ✅ |
| 16 | `16-TROUBLESHOOTING.md` | المنهجية + قواعد AI | 17 أكتوبر 2026 | ✅ |
| 17 | `17-STRATEGIC-ROADMAP.md` | الدستور التطويري | 17 أكتوبر 2026 | ✅ |
| 18 | `18-MARKETING-WEBSITE.md` | الموقع التسويقي | 17 أكتوبر 2026 | ✅ |
| 19 | `19-PERMISSIONS.md` | نظام الصلاحيات الكامل | 16 أكتوبر 2026 | ✅ |
| 20 | `20-USER JOURNEY.md` | رحلة المريض | 21 أكتوبر 2026 | ✅ |
| 21 | `21-PHR-SYSTEM.md` | نظام السجلات الطبية الشخصية | 26 أكتوبر 2026 | 🟡 Draft |
| 22 | `22-INTEGRATION-HUB.md` | مركز التكامل الخارجي | 26 أكتوبر 2026 | 🟡 Draft (رؤية مستقبلية) |
| 23 | `23-COMMERCE-ENGINE.md` | محرك التجارة (v1.1) | 12 نوفمبر 2026 | ✅ |
| 24 | `24-pharmacy-B2C-Orders.md` | طلبات B2C | 22 نوفمبر 2026 | 🟡 (يحتاج تحديث CaseID) |

### الملاحق

| # | الوثيقة | الحالة |
|---|---|---|
| A | `appendix-a-glossary.md` | ✅ |
| B | `appendix-b-service-index.md` | ✅ |

---

## 🏗️ الجزء الثاني: بنية المشروع (9 مشاريع)

| # | المشروع | التقنية | المسؤولية | الحالة |
|---|---|---|---|---|
| 1 | `RubikCare.Domain` | .NET 10 Class Library | الكيانات + Enums + Constants | ✅ |
| 2 | `RubikCare.Application` | .NET 10 Class Library | Use Cases + DTOs + Interfaces | ✅ |
| 3 | `RubikCare.Infrastructure` | .NET 10 + EF Core 10 | DbContext + Repositories + Migrations | ✅ |
| 4 | `Api.Web` | ASP.NET Core 10 | REST API + Controllers (**45+ Controller، ~337 Endpoint**) | ✅ |
| 5 | `Rubikcare.Web` | Blazor Server 10 | تطبيق الويب | ✅ |
| 6 | `RubikCare.PWA` | Blazor WebAssembly 10 | نسخة كربونية من Mobile | 🟠 ~95% |
| 7 | `Mobile` | .NET MAUI 10 | تطبيق الموبايل | ✅ |
| 8 | `Shared.UI` | Razor Class Library | **~70 مكون** مشترك + 25 خدمة | ✅ |
| 9 | `RubikCare.Tests` | xUnit + Moq | 33 اختبار | ✅ |

---

## 🎯 الجزء الثالث: الوضع الحالي (Modules Snapshot)

### ✅ Modules مكتملة ومستقرة

| # | Module | Backend | Frontend | الوثيقة |
|---|---|---|---|---|
| 1 | الهوية والمصادقة | ✅ | ✅ | 02 |
| 2 | القوائم الديناميكية | ✅ | ✅ | 03 |
| 3 | الترجمة والاتجاه | ✅ | ✅ | 04 |
| 4 | نظام PSP + Phase System | ✅ | ✅ | 05 |
| 5 | نظام التقارير | ✅ | ✅ | 07 |
| 6 | **نظام النقاط (كامل للمريض)** | ✅ | ✅ | 06 |
| 7 | Google Sign-In | ✅ | ✅ | 02 |
| 8 | الكاش الموحد | ✅ | ✅ | 12 |
| 9 | الإشعارات (المرحلة 1 — NH-1→NH-20) | ✅ | ✅ | 08 |
| 10 | الموقع التسويقي | ✅ | ✅ | 18 |
| 11 | نظام الصلاحيات (Schema) | ✅ | ❌ | 19 |
| 12 | Dashboard (PWA) — DashboardPage.razor | ✅ | ✅ | 00B (S3.5-H14) |
| 13 | **B2C Orders (3 Endpoints)** | ✅ | ✅ | 09 §9.2 |
| 14 | **ClinicReferral (4 Endpoints)** | ✅ | 🟡 (CodeView موجود، Join.razor 0%) | 09 (يحتاج إضافة) |
| 15 | **Prescription (8 Endpoints)** | ✅ | 🟡 (Controller موجود، Composer 0%) | 09 (يحتاج إضافة) |
| 16 | **Commerce B2B (28 Endpoints)** | ✅ | 🟡 (تحسينات مطلوبة) | 09 §21, 23 |

### 🟠 Modules مكتملة جزئياً

| # | Module | ما ينقص | Sprint |
|---|---|---|---|
| 1 | **Notification Hub — Phase 2** | Hub متطور مستقل | S09 (نشط) |
| 2 | **Clinic Integration UI** | Join.razor, PrescriptionComposer, DoctorNotes | S10 |
| 3 | **Commerce Enhancements** | البونص UI، إدارة الحالة، Tiers، Logistics | S11 |
| 4 | **PSP Cycle Final Polish** | Reports Integration، زر "ابدأ رحلتك" | S12 |
| 5 | **Messaging** | فصل الرسائل الشخصية/المنظمات | S13 |
| 6 | **Ledger** | غير مفعّل (API موجود) | S14 |
| 7 | نظام الصلاحيات | الـ Middleware + الـ UI | لاحقاً |
| 8 | طلبات شراء الأدوية | تكامل مع نظام النقاط | لاحقاً |
| 9 | **PWA** | MessagingHubPage + 3 لوحات مهنية | S13 |
| 10 | **Dashboard (MAUI)** | DashboardPage.xaml — تطابق PWA | لاحقاً |

### ⏳ Modules مُخطَّطة (لم تبدأ)

| # | Module | الرؤية | الوثيقة |
|---|---|---|---|
| 1 | B2B Commerce (Warehouse Layer) | Supply Chain كامل | 17 |
| 2 | Medical CRM | KPI's للمندوبين | 17 |
| 3 | Clinic OS | إدارة العيادات | 17 |
| 4 | CME | التعليم الطبي المستمر | 17 |
| 5 | Rubika Currency Engine | عملة داخلية | 06 |
| 6 | Medical Tourism | السياحة العلاجية | 17 |
| 7 | Personal Health Records (PHR) | السجل الطبي الشخصي | 21 |
| 8 | Integration Hub | مركز التكامل الخارجي | 22 |
| 9 | Dashboard Unification (PWA + MAUI) | — | 00B (Epic 14) |
| 10 | Dynamic Menus (MAUI) | — | 03-MENUS |

---

## 🔴 الجزء الرابع: المناطق المظلمة (Dark Areas)

| # | الميزة | الموقع | التوثيق |
|---|---|---|---|
| 1 | PSPEnrollmentService | `Application/Services/PSP/` | ✅ 05 |
| 2 | DispenseController | `Api.Web/Controllers/Pharmacy/` | ✅ 09 (⚠️ كله AllowAnonymous — TD-C4) |
| 3 | MenuVisibilityRules UI | مفقود | 03 |
| 4 | OrganizationGroups | تأسيس فقط | ✅ 03 |
| 5 | Rubika Currency Schema | DB فقط | ✅ 06 |
| 6 | **Messaging System** | ✅ كامل | 14 (⚠️ **يحتاج صيانة — GAP-11**) |
| 7 | File Uploads | ✅ كامل | — |
| 8 | Booking System | ✅ كامل | 14 |
| 9 | Nearby Pharmacies | ✅ كامل | 14 |
| 10 | CustomBlazorWebView | ✅ كامل | ✅ 13 |
| 11 | Marketing Website — Header/Footer | ✅ كامل | ✅ 18 |
| 12 | DashboardPage.xaml (MAUI) — النسخة المحلية | ✅ كامل | — |
| 13 | DashboardPage.razor (Shared.UI) | ✅ كامل (PWA) | 00B (S3.5-H14) |
| 14 | CurrentUserProfileState (Singleton) | ✅ مُنشأ | 13-MAUI § 11.5 |
| 15 | DashboardNavigationBridge | ✅ مُنشأ | — |
| 16 | **ClinicReferralController (4 Endpoints)** | ✅ كامل | ⏳ 09 (يحتاج إضافة) |
| 17 | **PrescriptionController (8 Endpoints)** | ✅ كامل | ⏳ 09 (يحتاج إضافة) |
| 18 | **PharmacyOrdersController (4 Endpoints)** | ✅ كامل | ⏳ 09 (يحتاج إضافة) |
| 19 | **PharmacyPspController (1 Endpoint)** | ✅ كامل | ⏳ 09 (يحتاج إضافة) |
| 20 | **LedgerController (3 Endpoints)** | ✅ كامل (API) | ⏳ 09 (⚠️ غير مفعّل — GAP-01) |

---

## 📅 الجزء الخامس: خريطة الجلسات (Sessions Map)

| الجلسة | التاريخ | ما تم | الوثائق المُحدَّثة |
|---|---|---|---|
| #104-108 | 10-13 أكتوبر 2026 | Campaign Reference + SourceEntityID | 06 |
| #109-115 | 14-15 أكتوبر 2026 | Loyalty UI + Reports + Beneficiaries | 06 + 07 |
| #116-121 | 16 أكتوبر 2026 | جرد شامل + الفهرس v1.0 | 00 |
| #122-130 | 16-17 أكتوبر 2026 | إعادة هيكلة الوثائق (34 → 20) | جميع الوثائق |
| #131 | 17 أكتوبر 2026 | دمج 04-TRANSLATION (v3.1) | 04 + 08 |
| #132 | 18 أكتوبر 2026 | إصدار 00-MASTER-INDEX v2.0 | 00 |
| #133 | 26 أكتوبر 2026 | إصدار 00-MASTER-INDEX v2.1 + PHR (21) + Sprint 4-6 | 00 + 00B + 21 |
| #134 | 26 أكتوبر 2026 | إصدار 00-MASTER-INDEX v2.2 + Integration Hub (22) + Sprint 7-9 | 00 + 00B + 22 |
| #135-136 | 4-5 نوفمبر 2026 | إعادة تصميم DashboardPage.razor (PWA) + فشل MAUI (BlazorWebView) + العودة لـ XAML المحلي | 00B (v6.3) + 13-MAUI (v3.3) + 01-ARCHITECTURE (v2.1) |
| **#137-150** | **6-23 نوفمبر 2026** | **Notification Hub (NH-1→NH-20، 50 SP) + B2B Commerce MVP + PSP Cycle fixes** | **00B (v7.2)** |
| **#151-153** | **24-25 نوفمبر 2026** | **Clinic Integration + Prescription + Referral + توثيق الفجوات + إعادة تشكيل Sprint Plan** | **00B (v8.0) + 00-MASTER-INDEX (v3.0) + 09 (محدث)** |

---

## 🚨 الجزء السادس: الأولويات الحالية

### 🔴 P1 — حرجة (S09 — Notification Hub — Phase 2)

| # | المهمة | Sprint | GAP |
|---|---|---|---|
| 1 | Notification Hub — Core (Entities + Migrations) | S09 | — |
| 2 | Notification Hub — Services | S09 | — |
| 3 | Notification Hub — API | S09 | — |
| 4 | Notification Hub — UI | S09 | — |
| 5 | ربط Hub بالنقاط | S09 | — |

### 🟠 P2 — عالية (S10 — Clinic Integration)

| # | المهمة | Sprint | GAP |
|---|---|---|---|
| 1 | تعديل `Join.razor` | S10 | GAP-02 |
| 2 | `PrescriptionComposerView` | S10 | GAP-12, GAP-14 |
| 3 | `DoctorNotesView` + `DoctorNoteController` | S10 | GAP-03 |
| 4 | `PSPDataEntryResult` + `PatientRequiredDataEntryPage` | S10 | GAP-13, GAP-15 |

### 🟡 P3 — متوسطة (S11–S14)

| # | المهمة | Sprint | GAP |
|---|---|---|---|
| 1 | إصلاح البونص UI | S11 | GAP-04 |
| 2 | إدارة حالة الطلب (الشركة) | S11 | GAP-05 |
| 3 | Tiers + Logistics Pages | S11 | GAP-07, GAP-08 |
| 4 | Reports Integration (Points) | S12 | — |
| 5 | Messaging صيانة | S13 | GAP-11 |
| 6 | Ledger Activation | S14 | GAP-01 |

---

## 🗺️ الجزء السابع: Sprint Plan v8.0 (خريطة التنفيذ)

### Sprint History

| Sprint | الاسم | التاريخ | الحالة |
|---|---|---|---|
| S01 | Foundation | 16-22 أكتوبر 2026 | ✅ |
| S02 | Points-PSP Integration | 22-23 أكتوبر 2026 | ✅ |
| S03 | MAUI Stabilization | 24-25 أكتوبر 2026 | ✅ |
| S04 | PSP Dispensation | 26-28 أكتوبر 2026 | ✅ |
| S05 | Post-Deploy Fixes 1 | 28-31 أكتوبر 2026 | ✅ |
| S06 | Post-Deploy Fixes 2 | 1-9 نوفمبر 2026 | 🟡 |
| S07 | B2B Commerce MVP | 6-11 نوفمبر 2026 | 🟡 |
| S08 | PSP Cycle + NH Part 1 | 25-30 نوفمبر 2026 | 🟡 |

### Sprint Current

| Sprint | الاسم | التاريخ | الحالة |
|---|---|---|---|
| **S09** | **Notification Hub — Phase 2** | **25 نوفمبر — 10 ديسمبر 2026** | 🟢 نشط |

### Sprint Future

| Sprint | الاسم | التاريخ | الحالة |
|---|---|---|---|
| S10 | Clinic Integration — UI Completion | 11-20 ديسمبر 2026 | ⏳ مخطط |
| S11 | Commerce Enhancements | 21-31 ديسمبر 2026 | ⏳ مخطط |
| S12 | PSP Cycle — Final Polish | 1-10 يناير 2027 | ⏳ مخطط |
| S13 | Messaging Maintenance | 11-20 يناير 2027 | ⏳ مخطط |
| S14 | Ledger Activation | 21-31 يناير 2027 | ⏳ مخطط |
| S15 | Production Launch Prep | فبراير 2027 | ⏳ مخطط |

---

## 🧠 الجزء الثامن: قواعد صارمة للـ AI Assistant

### قبل أي جلسة — اطلب:

- هذه الوثيقة (`00-MASTER-INDEX.md`).
- الوثيقة المتعلقة بالمهمة (03, 05, 06, 07, 08, 19, 21, 22 — حسب الحالة).
- `00B-PROJECT-BACKLOG.md` (للحالة الحية).

### 🎯 القواعد الإلزامية (18 قاعدة):

| # | القاعدة |
|---|---|
| 1 | لا تفترض أن ميزة غير موجودة — راجع الـ Dark Areas + الوثيقة المرتبطة. |
| 2 | لا تخمّن حقولاً — اطلب رؤية Entity أو `SELECT TOP 1 *`. |
| 3 | ارجع لـ `17-STRATEGIC-ROADMAP` قبل أي قرار معماري كبير. |
| 4 | ارجع لـ `03-MENUS` قبل أي تعديل على القوائم. |
| 5 | ارجع لـ `06-POINTS` قبل أي تعديل على النقاط. |
| 6 | ارجع لـ `11-PAGE-CREATION` قبل أي كود جديد. |
| 7 | قاعدة "Stable vs Unstable" — لا تعيد هيكلة Stable Modules بدون إذن. |
| 8 | EF Core Code-First — لا تعديل يدوي على Migration files. |
| 9 | استخدم `N''` في SQL — للعربية. |
| 10 | استخدم `MERGE` بدلاً من `INSERT/UPDATE` — منفصلة. |
| 11 | قبل كتابة أي كود، اذكر صراحةً: "أحتاج لرؤية ملف [الاسم] للتأكد من الحقول والعلاقات." |
| 12 | 🔴 قاعدة "الاستنتاج من السياق": قبل افتراض عطل في البنية التحتية، راجع الأدلة الفعلية في السياق. |
| 13 | 🔴 قواعد العمل خلال الجلسة: ابدأ كل رد بـ `[#رقم]` متسلسل، نفّذ خطوة واحدة واضحة في كل رد. |
| 14 | 🔴 قاعدة التقديرات: التقديرات تُبنى على نموذج v2.0. |
| 15 | 🔴 لا تستخدم `BlazorWebView` كصفحة رئيسية (`ShellContent`) في MAUI مع Parameters ديناميكية. |
| 16 | 🔴 لا تعتمد على Singleton Services للتواصل بين MAUI و BlazorWebView. |
| 17 | 🔴 **الوثيقة تطابق الكود** — أي تعارض يُحلّ لصالح الكود، والوثيقة تُحدَّث. |
| 18 | 🔴 **كل فجوة موثقة** — أي فجوة بين "مخطط" و"منفّذ" تُوثَّق في `00B §7 Gaps Register` برقم `GAP-NN`. |

### 🎯 عند الغموض:

| السؤال | الإجراء |
|---|---|
| هل هذه الميزة موجودة؟ | اسأل + راجع الـ Dark Areas |
| ما هو الملف الفعلي؟ | اطلب الرؤية |
| ما هي الحقول؟ | اطلب `SELECT TOP 1 * FROM Table` |
| ما هو النمط المعماري؟ | راجع `01` + `11` + `03` |

---

## 📊 الجزء التاسع: إحصائيات المشروع (بعد التصحيح)

| المقياس | القيمة القديمة | **القيمة الفعلية** | المصدر |
|---|---|---|---|
| عدد المشاريع | 9 | **9** | — |
| عدد وثائق `docs/` | 22 + ملاحق | **24 + ملاحق** | — |
| `Shared.UI` — مكونات | 68 | **~70** | `[#29]` + `ClinicReferralCodeView` |
| `Shared.UI` — ملفات CSS | 120+ | **~122** | + `ClinicReferralCodeView.css` |
| `Shared.UI` — خدمات | 25 | **25** | — |
| `Mobile` — صفحات XAML | 57 | **57** | — |
| `PWA` — صفحات Razor | 25+ | **25+** | — |
| **`Api.Web` — Controllers** | **~35** | **45+** | `09-API-GUIDE` + `[#29]` + Controllers المرفقة |
| **`Api.Web` — Endpoints** | **~290** | **~337** | `09-API-GUIDE` (320) + 17 جديد |
| `Domain` — Entities | 150+ | **150+** | — |
| `Tests` — Architecture | 33 | **33** | — |
| نظام النقاط — Handlers | 19 | **19** | — |
| نظام التقارير — Handlers | 8 | **8** | — |
| **Commerce — Controllers** | — | **6** | `09-API-GUIDE §21` |
| **Commerce — Endpoints** | — | **28** | `09-API-GUIDE §21` |
| **Clinic Integration — Controllers** | — | **2** | `[#29]` + الكود |
| **Clinic Integration — Endpoints** | — | **12** | 4 (Referral) + 8 (Prescription) |
| **Pharmacy — Controllers** | — | **5** | `09-API-GUIDE §10` + `PharmacyOrders` + `PharmacyPsp` |
| **Pharmacy — Endpoints** | — | **32** | 14 (Dispense) + 6 (MedicationRequest) + 7 (Pharmacy) + 4 (PharmacyOrders) + 1 (PharmacyPsp) |

### تفصيل الـ Endpoints (بعد التصحيح)

| المجموعة | Controllers | Endpoints | الحالة |
|---|---|---|---|
| Auth & User | 2 | 20 | ✅ |
| Organizations | 4 | 13 | ✅ |
| Clinical | 2 | 5 | ✅ |
| GeoLocations | 3 | 6 | ✅ |
| Messaging | 2 | 16 | 🟡 (يحتاج صيانة) |
| Notifications | 1 | 9 | 🟡 (S09 نشط) |
| Clinic | 1 | 15 | ✅ |
| Professional | 1 | 3 | ✅ |
| Patient | 2 | 13 | ✅ |
| **Pharmacy** | **5** | **32** | ✅ |
| **PSP** | **15** | **68** (62 + 6 Points) | ✅ |
| **Points** | **6** | **38** | ✅ |
| Rep | 1 | 12 | ✅ |
| Reports | 1 | 8 | ✅ |
| Dashboard | 1 | 1 | ✅ |
| Misc | 3 | 3 | ✅ |
| **Commerce** | **6** | **28** | ✅ |
| **ClinicReferral** | **1** | **4** | ✅ (جديد) |
| **Prescription** | **1** | **8** | ✅ (جديد) |
| Base | 1 | 8 | — |
| **المجموع** | **~45+** | **~337** | — |

---

## 🕰️ الجزء العاشر: سجل التقديرات التاريخية (Historical Estimates)

| # | النظام (Module) | تاريخ البداية | تاريخ النهاية | الوقت الفعلي | حجم العمل |
|---|---|---|---|---|---|
| 1 | نظام PSP الأساسي | مارس 2026 | سبتمبر 2026 | ~6 أشهر | ~30 ملف |
| 2 | نظام Phase System (PSP) | 23 سبتمبر 2026 | 23 سبتمبر 2026 | ~1 يوم | 2 كيان + 1 Migration |
| 3 | نظام التحاليل (Tests) | 27 سبتمبر 2026 | 4 أكتوبر 2026 | ~7 أيام | 4 كيانات + 38 Endpoint |
| 4 | نظام النقاط (Points) | 29 سبتمبر 2026 | 15 أكتوبر 2026 | ~17 يوم | 8 كيانات + 19 Handler |
| 5 | نظام التقارير (Reports) | 16 أكتوبر 2026 | 16 أكتوبر 2026 | ~1 يوم | 8 Handlers |
| 6 | نظام الإشعارات (Notifications — Part 1) | 31 مارس 2026 | 16 سبتمبر 2026 | ~5.5 أشهر | 3 أنظمة |
| 7 | نظام القوائم (Menus) | 2 مارس 2026 | 13 أكتوبر 2026 | ~7 أشهر | 4 جداول + 3 خدمات |
| 8 | نظام الترجمة (Translation) | مايو 2026 | سبتمبر 2026 | ~4 أشهر | 3 مسارات + 10 ملفات |
| 9 | إعادة هيكلة الوثائق | 16 أكتوبر 2026 | 18 أكتوبر 2026 | ~2 أيام | 34 → 22 وثيقة |
| **10** | **B2B Commerce MVP** | **6 نوفمبر 2026** | **12 نوفمبر 2026** | **~6 أيام** | **12 كيان + 28 Endpoint** |
| **11** | **Notification Hub (NH-1→NH-20)** | **22 نوفمبر 2026** | **23 نوفمبر 2026** | **~2 يوم** | **20 مهمة، 50 SP** |
| **12** | **Clinic Integration (Referral + Prescription)** | **24 نوفمبر 2026** | **24 نوفمبر 2026** | **~1 يوم** | **4 كيانات + 12 Endpoint** |

### 🎯 معادلة التقدير (v2.0):

**التقدير = (المعدل البشري × معامل AI) × (1 + هامش المخاطر)**

| نوع المهمة | معامل AI | مثال |
|---|---|---|
| Backend — كيانات + Migration | 0.2 | إنشاء 12 كيان + Migration |
| Backend — Services + API | 0.3 | SupplyOrderService + Controllers |
| Frontend — UI بسيطة (CRUD) | 0.5 | CampaignsPage, BundlesPage |
| Frontend — UI معقدة | 0.7 | ProductDetailsPage (Bonus Calculator) |
| Integration — ربط UI بـ API | 0.8 | ربط الشاشات بالـ Endpoints |
| Testing — اختبارات E2E | 1.0 | اختبار دورة الطلب الكاملة |
| DevOps / Deployment | 0.9 | نشر على Production |

---

## 🎯 الجزء الحادي عشر: القرارات المعمارية الحاسمة

| # | القرار | الوثيقة |
|---|---|---|
| 1 | 9 مشاريع — Clean Architecture | 01 |
| 2 | `IGenericService` للـ CRUD — UseCase للمعقد | 11 |
| 3 | MenuVisibilityRules — 4 طبقات | 03 |
| 4 | Reusable Services (Points, Notifications, Translation, Messaging) | 17 |
| 5 | SourceEntityID + PointSystemTypeID — لا جدول Campaigns | 06 |
| 6 | TargetBeneficiaryTypeID — لا يحذف | 06 |
| 7 | PointDashboard — واجهة واحدة | 06 |
| 8 | Marketing Website — معزول | 18 |
| 9 | 3 طبقات إشعارات | 08 |
| 10 | PWA = نسخة كربونية من Mobile | 14 |
| 11 | Code-First Migration | 11 |
| 12 | `N''` للعربية + MERGE للبيانات | 04 |
| 13 | لا `dir` على `div` الجذر | 04 |
| 14 | SourceEntityID + OrganizationTypeFilterID | 06 |
| 15 | ControlLevel + Permission | 19 |
| 16 | PHR: HealthRecord (Aggregation) + SourceEntityID | 21 |
| 17 | PHR: `UserProfileID` (الشخص) وليس `PatientProfileID` | 21 |
| 18 | PHR: المريض هو المالك — SharingConsent | 21 |
| 19 | Integration Hub: IIntegrationClient موحد لكل التكاملات | 22 |
| 20 | Integration Hub: Adapters مستقلة (REST, SOAP, FHIR, Webhook) | 22 |
| 21 | Integration Hub: كل استدعاء له Audit Log + Idempotency | 22 |
| 22 | الصفحات الرئيسية في MAUI (ShellContent) = XAML محلي | 13-MAUI § 6.5 |
| 23 | الصفحات الفرعية في MAUI = BlazorWebView + RootComponents.Add | 13-MAUI § 6.1 |
| 24 | لا Singleton Services للتواصل بين MAUI و BlazorWebView | 13-MAUI § 11.5 |
| 25 | `Shared.UI` للمكونات (بما فيها Page Components) — لكن ليس كصفحات رئيسية مع Parameters | 01-ARCHITECTURE § 2.8 |
| **26** | **`CaseID` هو المعرّف الرسمي (وليس `PatientProfileID`)** | 05 §1.5, 01 §3ب |
| **27** | **`PSPParticipation` = مشاركة منظمة (عيادة/صيدلية) — وليست اشتراك مريض** | 05 §1.5.3 R6 |
| **28** | **`ClinicProgramReferral` ≠ `PSPInvitation`** | 05 §1.5.3 R7 |
| **29** | **`Ledger` موجود كـ API لكن غير مفعّل — يحتاج ربط بـ `SupplyOrderService`** | 23 §0.6, GAP-01 |
| **30** | **كل `Endpoint` يجب أن يكون موثقاً في `09-API-GUIDE` (Input/Output/Auth)** | 09 (محدث) |

---

## 🎯 الجزء الثاني عشر: "Stable vs Unstable" Modules

### ✅ Modules مستقرة (لا تلمس)

| # | Module | الوثيقة |
|---|---|---|
| 1 | Identity + Auth | 02 |
| 2 | Menus System | 03 |
| 3 | Translation System | 04 |
| 4 | Caching System | 12 |
| 5 | Google Sign-In | 02 |
| 6 | Deployment | 15 |
| 7 | PSP Core | 05 |
| 8 | Reports | 07 |
| 9 | Marketing Website | 18 |
| 10 | Dashboard (PWA) — DashboardPage.razor | 00B (S3.5-H14) |
| **11** | **Points (Backend + Frontend — كامل للمريض)** | 06 |
| **12** | **B2C Orders (3 Endpoints)** | 09 §9.2 |

### 🟡 Modules قابلة للتعديل (بإذن)

| # | Module | السبب |
|---|---|---|
| 1 | Notification Hub | S09 نشط |
| 2 | PWA | ~95% |
| 3 | Admin* Pages | LVL 1 → LVL 3 |
| 4 | PSPStep2b | إعادة هيكلة |
| 5 | Messaging | يحتاج صيانة (S13) |
| 6 | نظام الصلاحيات | Phase 2 + 3 |
| 7 | PHR (جديد) | Phase 1-3 |
| 8 | Integration Hub (جديد) | Phase 1-3 |
| 9 | Dashboard (MAUI) | تطابق PWA |
| **10** | **Commerce B2B** | تحسينات (S11, S14) |
| **11** | **Clinic Integration UI** | S10 |

### 🔴 Modules غير مستقرة (قيد إعادة الهيكلة)

| # | Module | السبب |
|---|---|---|
| 1 | AdminOrganizationsList | LVL 1 → LVL 3 |
| 2 | AdminIdentity | LVL 1 → LVL 3 |
| 3 | PSPProgramsList | LVL 1 → LVL 3 |
| 4 | إزالة ProgramID المتبقية | جزئياً |
| 5 | DashboardPage (المشترك في MAUI) | مؤجل (S3.5-H22) |
| **6** | **Ledger** | غير مفعّل (GAP-01) |

---

## 🎯 الجزء الثالث عشر: كيف تستخدم هذه الوثيقة؟

### سيناريو (1): بداية جلسة جديدة
1. أرسل هذه الوثيقة للـ AI.
2. أرسل `00B-PROJECT-BACKLOG.md` (الحالة الحية).
3. أرسل الوثيقة المتعلقة بالمهمة.
4. أرسل الملفات المطلوبة.

### سيناريو (2): AI يفترض ميزة غير موجودة
🎯 أرجع للـ Dark Areas + الوثيقة المرتبطة.

### سيناريو (3): تحتاج تحديث الوثيقة
🎯 بعد كل جلسة — حدّث:
- خريطة الجلسات.
- الأولويات.
- Dark Areas.
- Modules Snapshot.
- **Gaps Register** (جديد).

---

## 🔗 الجزء الرابع عشر: روابط ذات صلة

- `01-ARCHITECTURE.md`
- `05-psp-system.md`
- `17-STRATEGIC-ROADMAP.md`
- `19-PERMISSIONS.md`
- `20-USER JOURNEY.md`
- `21-PHR-SYSTEM.md`
- `22-INTEGRATION-HUB.md`
- `23-COMMERCE-ENGINE.md`
- `24-pharmacy-B2C-Orders.md`
- `00B-PROJECT-BACKLOG.md`
- `13-MAUI.md`

---

## 📝 الجزء الخامس عشر: سجل التحديثات

| الإصدار | التاريخ | التغييرات |
|---|---|---|
| 1.0 | 16 أكتوبر 2026 | الإصدار الأولي |
| 2.0 | 18 أكتوبر 2026 | v2.0 — تحديث الفهرس (20 وثيقة) + 19-PERMISSIONS |
| 2.1 | 26 أكتوبر 2026 | v2.1 — إضافة 21-PHR-SYSTEM + Epic 11 |
| 2.2 | 26 أكتوبر 2026 | v2.2 — إضافة 22-INTEGRATION-HUB + Epic 12 + Sprint 7-9 |
| 2.3 | 4 نوفمبر 2026 | تحديث Modules Snapshot + Dark Areas (12-15) + خريطة الجلسات (#135-136) + قواعد AI (15-16) |
| 2.4 | 12 نوفمبر 2026 | إضافة "الجزء العاشر-ج: نموذج تقدير السرعة (v2.0)" |
| **3.0** | **25 نوفمبر 2026** | **إعادة كتابة كاملة (Realigned Edition) وفق Sprint Plan v8.0: تحديث الإصدارات؛ تحديث `Api.Web` (`~290` → `~337` Endpoint، `~35` → `45+` Controller)؛ تحديث `Shared.UI` (68 → ~70)؛ إضافة Modules جديدة (B2C, ClinicReferral, Prescription, Commerce)؛ تحديث Dark Areas (16-20)؛ تحديث خريطة الجلسات (#137-153)؛ تحديث Sprint Plan (S01-S15)؛ إضافة Gaps Register؛ تحديث القرارات المعمارية (26-30)؛ تحديث Stable vs Unstable؛ إضافة قاعدتين جديدتين (17-18)؛ تحديث إحصائيات المشروع (بعد التصحيح).** |

---

© 2026 RubikCare — للاستخدام الداخلي

**نهاية الوثيقة** ✅
```
