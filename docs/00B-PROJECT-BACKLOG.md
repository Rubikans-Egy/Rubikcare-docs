# 00B — PROJECT-BACKLOG (Live State Edition)

الاسم: PROJECT-BACKLOG — سجل التطوير الحي
الإصدار: 7.0 (Live State Edition)
آخر تحديث: 11 نوفمبر 2026
Sprint الحالي: Sprint 4 — B2B Commerce MVP Completion (11-17 نوفمبر)
الحالة: 🟢 حي — يُحدَّث كل جلسة
الجمهور: فريق التطوير + AI Assistants
الوثيقة الشقيقة: `00C-CHANGELOG.md` (كل ما تم إنجازه)

---

## 📖 منهجية هذه الوثيقة (Methodology)

هذه الوثيقة تتبع 3 قواعد صارمة:

### القاعدة 1: الحالة الحية (Live State Only)
هذه الوثيقة تُجيب على سؤال واحد فقط: **"ما الذي نعمل عليه الآن، وما الذي سيأتي، وما الذي تأجَّل؟"**
لا تحتوي على تفاصيل ما تم إنجازه — ذلك في `00C-CHANGELOG.md`.

### القاعدة 2: SP التاريخي (Historical SP Preserved)
نُبقي على **إجمالي SP لكل Sprint** في جدول واحد مختصر في النهاية (§8) — لمرجعية Velocity فقط. التفاصيل الكاملة لكل مهمة مكتملة موجودة في `00C`.

### القاعدة 3: المصدر الوحيد (Single Source of Truth)
- **هنا فقط:** كل ما هو "قيد التنفيذ" أو "متبقٍّ" أو "مؤجل" أو "مُخطط".
- **في `00C` فقط:** كل ما هو "مكتمل" (Done).
- **في `00D` فقط:** الجدول الزمني الكلي (Sprint-by-Sprint) والمعالم.

### قاعدة ذهبية للـ AI Assistant
> عند قراءة هذه الوثيقة، ابدأ من **§2 (Sprint الحالي)**. لا تقرأ §8 (SP التاريخي) إلا إذا احتجت حساب Velocity. لا تفترض أن مهمة "مكتملة" لأنها غير موجودة هنا — ابحث عنها في `00C`.

---

## 🎯 الرؤية الحالية (Current Vision — 11 نوفمبر 2026)

RubikCare في مرحلة **Pre-Launch — MVP Completion**. الهدف:

1. **إغلاق B2B Commerce MVP** (Sprint 4 — الأسبوع الحالي).
2. **تأسيس PHR Phase 1 (Hb Navigator)** (Sprint 5 — الأسبوع القادم).
3. **إغلاق فجوات PSP Cycle** (Sprint 6 — ديسمبر).
4. **الاستعداد لـ Production Launch** (Sprint 7+).

**ملاحظة مهمة:** B2B Commerce كان مخططاً في Sprint 8+ (بعد PHR وIntegration Hub)، لكن تم تقديمه إلى Sprint 4 لأنه **حلقة مفقودة في PSP Cycle** (الصيدلية تحتاج طلب الأدوية من الشركة قبل صرفها للمريض). هذا التقديم أدى إلى تأجيل PHR من Sprint 4 إلى Sprint 5.

---

## 🟢 §1: Sprint 4 — B2B Commerce MVP Completion (11-17 نوفمبر 2026)

**الهدف:** إغلاق كل ما بدأ في B2B Commerce MVP (Domain + Application + API + UI Phase 1+2 منجز — انظر `00C`).

**المرجع:** `23-COMMERCE-ENGINE.md` §0.6 (حالة التنفيذ) + §15 (Roadmap المتبقي).

### 🔴 P1 — حرجة (MVP Completion) — هذا الأسبوع

| # | المهمة | SP | الوقت المقدر | الحالة | المرجع |
|---|---|---|---|---|---|
| S4-P1-1 | إنهاء منطق حساب الباقات (`BundleService` + UI) | 2 | 2-3 ساعات | 🟡 قيد التنفيذ | 23-COMMERCE §15 |
| S4-P1-2 | إنشاء صفحة Bonus (UI + API) | 3 | 4 ساعات | ⏳ | 23-COMMERCE §15 |
| S4-P1-3 | إنشاء صفحة Tiers (UI + API) | 3 | 4 ساعات | ⏳ | 23-COMMERCE §15 |
| S4-P1-4 | إنشاء صفحة Logistics (UI + API) | 2 | 3 ساعات | ⏳ | 23-COMMERCE §15 |
| S4-P1-5 | اختبار إنشاء طلب جديد بعد تعديلات Price & Tax | 1 | 1 ساعة | ⏳ | 23-COMMERCE §14 |

**المجموع P1:** 11 SP — **~14-15 ساعة عمل**

### 🟠 P2 — متوسطة (يمكن أن تنجز هذا الأسبوع أو القادم)

| # | المهمة | SP | الوقت المقدر | المرجع |
|---|---|---|---|---|
| S4-P2-1 | Payments API (ServicePayment — الكيان جاهز) | 2 | 4 ساعات | 23-COMMERCE §15 |
| S4-P2-2 | Shipments API (ServiceShipment — الكيان جاهز) | 2 | 3 ساعات | 23-COMMERCE §15 |
| S4-P2-3 | Invoices API (ServiceInvoice — الكيان جاهز) | 2 | 3 ساعات | 23-COMMERCE §15 |
| S4-P2-4 | Agreements API (PharmacyCompanyAgreement) | 2 | 3 ساعات | 23-COMMERCE §15 |
| S4-P2-5 | Stock API (PharmaCompanyMedicationStock) | 2 | 3 ساعات | 23-COMMERCE §15 |
| S4-P2-6 | Ledger UI (صفحة الأرصدة — API جاهز) | 2 | 4 ساعات | 23-COMMERCE §15 |
| S4-P2-7 | Status History (PharmacySupplyOrderStatusHistory) | 1 | 2 ساعات | 23-COMMERCE §15 |

**المجموع P2:** 13 SP — **~22 ساعة عمل**

### 🟡 P3 — منخفضة (ما بعد MVP B2B — Sprint 6+)

| # | المهمة | SP | الوقت المقدر |
|---|---|---|---|
| S4-P3-1 | Warehouse Layer (FulfillmentMode = WAREHOUSE) | 8 | 1-2 يوم |
| S4-P3-2 | B2B Points Integration (Seed + Orchestrator) | 5 | 1 يوم |
| S4-P3-3 | Pharmacy Stock Page (بعد استلام الطلبات) | 2 | 3 ساعات |
| S4-P3-4 | Online Payments (Paymob/Fawry) | 8 | 2 أيام |

**المجموع P3:** 23 SP — **~4 أيام عمل** (مؤجل)

### 📌 ملاحظة: Sprint 4 في `00D` = 11-17 نوفمبر، لكن P2 قد تمتد إلى الأسبوع القادم إذا لم يكتمل P1 في الوقت. **قاعدة صارمة:** لا نبدأ Sprint 5 (PHR) قبل إغلاق كل P1 من Sprint 4.

---

## 🟠 §2: Sprint 5 — PHR Phase 1 (Hb Navigator) (18-24 نوفمبر 2026)

**الهدف:** تأسيس البنية التحتية للسجلات الطبية + تسجيل أول تحليل (Hb) كـ MVP.

**المرجع:** `21-PHR-SYSTEM.md` §1 (نموذج البيانات) + §2 (السيناريوهات) + `01-ARCHITECTURE.md` §3 (شجرة القرار).

### 🔴 P1 — حرجة (Foundation)

| # | المهمة | SP | الوقت المقدر | الحالة | المرجع |
|---|---|---|---|---|---|
| S5-P1-1 | Domain: `MedicalRecordType`, `LabTestType`, `AssessmentRule`, `AssessmentRuleType` | 3 | 1 يوم | ⏳ | 21-PHR §1.2 |
| S5-P1-2 | Domain: `HealthRecord`, `LabTestResult` | 2 | 0.5 يوم | ⏳ | 21-PHR §1.3 |
| S5-P1-3 | Domain: `SharingConsent` (basic — للتصميم فقط) | 1 | 0.5 يوم | ⏳ | 21-PHR §1.4 |
| S5-P1-4 | Seeds: `MedicalRecordType`, `LabTestType` (HB), `AssessmentRuleType`, `AssessmentRule` (كل قواعد classify) | 2 | 0.5 يوم | ⏳ | 21-PHR §1.2 |
| S5-P1-5 | Migration + EF Configurations (6 ملفات) | 2 | 0.5 يوم | ⏳ | 01-ARCH §15 |

**المجموع P1:** 10 SP — **~3 أيام عمل**

### 🟠 P2 — متوسطة (منطق التقييم)

| # | المهمة | SP | الوقت المقدر | المرجع |
|---|---|---|---|---|
| S5-P2-1 | `IHbClinicalRulesEngine` + Implementation | 3 | 1 يوم | 21-PHR §1.2.3 |
| S5-P2-2 | `SaveLabResultUseCase` + `GetLabResultsUseCase` | 2 | 0.5 يوم | 21-PHR §2.1 |
| S5-P2-3 | `SaveLabResultValidator` (FluentValidation + PlausibleRange من DB) | 1 | 0.5 يوم | 21-PHR §1.2.2 |
| S5-P2-4 | API: `HealthRecordsController` (POST + GET) | 2 | 0.5 يوم | 21-PHR §2.1 |

**المجموع P2:** 8 SP — **~2.5 يوم عمل**

### 🟡 P3 — منخفضة (الواجهة)

| # | المهمة | SP | الوقت المقدر | المرجع |
|---|---|---|---|---|
| S5-P3-1 | `HbNavigator.razor` + `.razor.cs` (من HTML الحالي) | 4 | 1.5 يوم | 21-PHR §2.1 |
| S5-P3-2 | `hb-navigator.css` (بدون Tailwind CDN) | 1 | 0.5 يوم | 01-ARCH §2.8 |
| S5-P3-3 | Chart Component (Blazor أو JSInterop) | 2 | 1 يوم | 21-PHR §2.1 |
| S5-P3-4 | Translation Keys AR/EN (SQL MERGE) | 1 | 0.5 يوم | 04-TRANSLATION |
| S5-P3-5 | RTL/LTR + Responsive Design | 2 | 0.5 يوم | 01-ARCH §2.8 |
| S5-P3-6 | Cross-Platform Testing (Web + PWA + MAUI) | 2 | 0.5 يوم | 01-ARCH §3.4 |

**المجموع P3:** 12 SP — **~4.5 يوم عمل**

**المجموع الكلي Sprint 5:** 30 SP — **~10 أيام عمل** (يحتاج Buffer 20% → **~12 يوم** — قد يمتد إلى الأسبوع التالي)

### ⚠️ تحذيرات حرجة قبل البدء

1. **قاعدة MAUI Golden Rule (01-ARCH §3.4):** لا تستخدم `HbNavigator.razor` كصفحة رئيسية في MAUI. استخدم XAML محلي + BlazorWebView فرعي (نمط S3.5-H27).
2. **قاعدة Decimal Comparison:** `h === 11` في JS ≠ `h == 11` في C# (EF Core). استخدم `Math.Round(h, 1) == 11`.
3. **AssessmentRule Overlap:** يجب إضافة عمود `Priority` (int) لتفادي اختيار Normal بدلاً من Borderline.
4. **UserProfileID وليس PatientProfileID:** حسب 21-PHR §0.2.
5. **Soft Delete إلزامي:** `HealthRecord.IsDeleted` (حسب 21-PHR §1.3.1).

---

## 🟡 §3: Sprint 6 — PSP Cycle Completion (25-30 نوفمبر 2026)

**الهدف:** إغلاق كل ما تبقى من Sprint 3.6 (Post-Deployment Fixes) + ربط B2B Points بـ PSP Cycle + PHR Phase 2 الجزئي.

**المرجع:** `05-PSP.md` + `06-POINTS.md` + `00B v6.4` (المهام المؤجلة).

### 🔴 P1 — حرجة (Sprint 3.6 المتبقي)

| # | المهمة | SP | الحالة | المرجع |
|---|---|---|---|---|
| S6-P1-1 | S3.6-H3: إصلاح Error بدل صفحة النقاط (points-dashboard) | 3 | ⏳ | 00C (المتبقي) |
| S6-P1-2 | S3.6-H4: توليد مواعيد الأدوية — أكتر من دواء في نفس المعاد | 5 | ⏳ | 00C (المتبقي) |
| S6-P1-3 | S3.6-H5: تعديل منطق زر "ابدأ رحلتك العلاجية" | 3 | ⏳ | 00C (المتبقي) |

**المجموع P1:** 11 SP

### 🟠 P2 — متوسطة (Sprint 1 المتبقي — Reports Integration)

| # | المهمة | SP | الحالة | المرجع |
|---|---|---|---|---|
| S6-P2-1 | P5.1: GetAdherenceHandler — إضافة Points | 2 | ⏳ | 01-ARCH §6ب (IC-07) |
| S6-P2-2 | P5.2: GetPharmacyFinancialHandler — فصل PointsDiscount | 3 | ⏳ | 01-ARCH §6ب (IC-08) |
| S6-P2-3 | P5.3: GetProgramLifecycleHandler — إضافة Points | 2 | ⏳ | 01-ARCH §6ب (IC-09) |

**المجموع P2:** 7 SP

### 🟡 P3 — منخفضة (Sprint 3.6 Medium + Low)

| # | المهمة | SP | الحالة |
|---|---|---|---|
| S6-P3-1 | S3.6-M1: تظبيط تاريخ الأوردر في "طلباتي" | 2 | ⏳ |
| S6-P3-2 | S3.6-M2: صفحة "دعواتي" من عند الدكتور | 3 | ⏳ |
| S6-P3-3 | S3.6-M3: الإشعارات (مراجعة عامة) | 3 | ⏳ |
| S6-P3-4 | S3.6-L1: تعديل صفحات المقدمة (Landing Pages) | 5 | ⏳ |

**المجموع P3:** 13 SP

**المجموع الكلي Sprint 6:** 31 SP — **~10 أيام عمل**

---

## 🔵 §4: Sprint 7+ — Planned (ديسمبر 2026 وما بعده)

### Sprint 7: PHR Phase 2 (1-7 ديسمبر)

| # | المهمة | SP | المرجع |
|---|---|---|---|
| S7-1 | إضافة `LabTestType` جديد (Ferritin) | 1 | 21-PHR §5 |
| S7-2 | واجهة ديناميكية (Dynamic Form) | 3 | 21-PHR §5 |
| S7-3 | `SharingConsent` (Scope: ALL, LAB_RESULTS) | 2 | 21-PHR §1.4 |
| S7-4 | API: `ShareRecordEndpoint` + `RevokeShareEndpoint` | 2 | 21-PHR §5 |
| S7-5 | UI: شاشة "مشاركة السجل" | 3 | 21-PHR §5 |
| S7-6 | اختبارات E2E للمشاركة | 2 | 21-PHR §5 |

**المجموع:** 13 SP

### Sprint 8: PHR Phase 3 + Integration Hub Core (8-21 ديسمبر)

| # | المهمة | SP | المرجع |
|---|---|---|---|
| S8-1 | `HealthRecord` يدعم PRESCRIPTION, SURGERY, MEASUREMENT | 3 | 21-PHR §5 |
| S8-2 | ربط PHR بـ PSP | 3 | 21-PHR §2.2 |
| S8-3 | `SharingConsent` (Scope: JSON، صلاحية زمنية) | 3 | 21-PHR §1.4 |
| S8-4 | `RecordAccessLog` + تقارير الوصول | 2 | 21-PHR §1.4 |
| S8-5 | UI: لوحة "السجل الطبي" (Timeline View) | 4 | 21-PHR §5 |
| S8-6 | Integration Hub Core (Domain + Application + Infrastructure) | 15 | 22-INTEGRATION-HUB |

**المجموع:** 30 SP

### Sprint 9+: Production Launch Preparation (يناير 2027)

| # | المهمة | SP |
|---|---|---|
| S9-1 | ClinicOS Adapter | 13 |
| S9-2 | Webhooks + OpenAPI | 12 |
| S9-3 | E2E Testing شامل | 8 |
| S9-4 | Performance Testing (Load) | 5 |
| S9-5 | Security Audit | 5 |
| S9-6 | Production Deployment | 3 |

**المجموع:** 46 SP

---

## 📋 §5: Backlog المستقبلي (Product Backlog — Priorities)

### 🔴 High Priority (بعد Sprint 9)

| # | User Story | Epic | SP | المرجع |
|---|---|---|---|---|
| B-01 | كـ Admin، أريد إدارة Control Levels | Permissions | 5 | 19-PERMISSIONS |
| B-02 | كـ Admin، أريد ربط Users بـ Control Levels | Permissions | 3 | 19-PERMISSIONS |
| B-03 | كـ Super Admin، أريد إدارة Permissions | Permissions | 5 | 19-PERMISSIONS |
| B-04 | كـ Admin، أريد إدارة اشتراك المنظمة | Subscriptions | 5 | 03-MENUS |
| B-05 | كـ Admin، أريد ربط SubscriptionPlan بـ Menus | Menus | 5 | 03-MENUS |
| B-06 | كـ Rep، أريد رؤية اسم الصيدلية/العيادة بعد قبول الدعوة | Rep | 5 | 13-Rep Epic |
| B-07 | كـ Rep، أريد إشعار عند قبول الدعوة | Rep | 3 | 13-Rep Epic |

### 🟠 Medium Priority

| # | User Story | Epic | SP | المرجع |
|---|---|---|---|---|
| B-10 | Menus ديناميكية في PWA | Menus | 5 | 03-MENUS |
| B-11 | Menus ديناميكية في MAUI | Menus | 5 | 03-MENUS |
| B-12 | ربط ServiceProduct بـ MenuItems | Menus | 3 | 03-MENUS |
| B-13 | ApiClient موحّد (Integration Hub) | Integration Hub | 8 | 22-INTEGRATION-HUB |
| B-14 | API-Contracts (OpenAPI) | Integration Hub | 5 | 22-INTEGRATION-HUB |

### 🟡 Low Priority

| # | User Story | Epic | SP | المرجع |
|---|---|---|---|---|
| B-20 | إدارة OrganizationGroups | Organization | 5 | 01-ARCH |
| B-21 | Rubika Currency Engine | Rubika | 8 | 06-POINTS |
| B-22 | Deep Linking من QR | Mobile | 3 | 13-MAUI |
| B-23 | Warehouse Layer (B2B Phase 2) | Commerce | 8 | 23-COMMERCE |
| B-24 | B2B Points Integration | Commerce | 5 | 23-COMMERCE |
| B-25 | Online Payments (Paymob/Fawry) | Commerce | 8 | 23-COMMERCE |

---

## 🏗️ §6: Technical Debt Register (المتبقي فقط)

### 🔴 Critical Debt

| # | الدين | الموقع | المرجع |
|---|---|---|---|
| TD-C1 | انتهاك معماري: `@page` في `Shared.UI` | `PharmacyDetailPage.razor` | 01-ARCH §2.8 |
| TD-C2 | خلط Namespaces | `PharmacySearchPage.razor` | 01-ARCH §2.8 |
| TD-C3 | 404 في `/admin/pending-licenses` | `SystemManagment` vs `SystemManagement` | — |

### 🟠 Medium Debt

| # | الدين | الموقع | المرجع |
|---|---|---|---|
| TD-M1 | زر البحث في Dashboard غير مربوط | `Pages/Dashboard.razor` | — |
| TD-M2 | ازدواجية FirebaseNotificationService | `Api.Web` + `Infrastructure` | — |
| TD-M3 | ازدواجية `IPSPPointSchemeService` | `Application` + `Web` | — |
| TD-M4 | ازدواجية `MedicineGuideFlow` (إذا وُجد) | `Mobile/Features/PSP/Patient/Views/` | — |
| TD-M5 | TD-001: الطلبات المكررة (Double API Calls) | `Rubikcare.Web` + `Api.Web` | 00C (تفاصيل) |

### 🟢 Low Debt

| # | الدين | الموقع |
|---|---|---|
| TD-L1 | `IMobileNavigationService` (اسم مضلل) | `Shared.UI/Services/` |
| TD-L2 | `HttpUtility` في Blazor WASM | `PharmacySearch.razor` |
| TD-L3 | `alert()` بدلاً من Toast | `PspSearch.razor` |
| TD-L4 | SystemMenuMenuID Shadow Property | `MenuAssignments` |
| TD-L5 | RoleId في MenuVisibilityRule | بعد Migration |

---

## 📊 §7: Velocity Chart (مختصر — آخر 4 Sprint فقط)

| Sprint | Planned | Actual | Ratio | الحالة |
|---|---|---|---|---|
| Sprint 3 | 53 SP | 58 SP | 109% ✅ | مكتمل (في 00C) |
| Sprint 3.5 | 24 SP | 24 SP | 100% ✅ | مكتمل (في 00C) |
| Sprint 3.6 | 34 SP | 29 SP | 85% 🟡 | مكتمل جزئياً (5 SP مؤجلة → Sprint 6) |
| Sprint 4 (Commerce) | 11 SP (P1) | — | — | 🟡 قيد التنفيذ |

**المعدل الفعلي للفريق:** ~ **1.0-1.2× SP المتوقع** (بعد استقرار Sprint 2-3).

---

## 🕰️ §8: SP التاريخي لكل Sprint (مرجعي فقط)

> هذا الجدول للمرجعية فقط. التفاصيل الكاملة لكل مهمة مكتملة موجودة في `00C-CHANGELOG.md`.

| Sprint | الفترة | SP Planned | SP Actual | SP Done | SP متبقٍّ |
|---|---|---|---|---|---|
| Sprint 0 (Foundation) | 16-22 أكتوبر | 22 | 11 | 11 | 11 (مؤجلة إلى ما بعد MVP) |
| Sprint 1 (Points-PSP Integration) | 22-23 أكتوبر | 36 | 46 | 46 | 11 (P5.1-P5.3 → Sprint 6) |
| Sprint 2 (MAUI Stabilization) | 24-25 أكتوبر | 27 | 27 | 27 | 0 |
| Sprint 3 (PSP Dispensation) | 26-28 أكتوبر | 53 | 58 | 58 | 0 |
| Sprint 3.5 (Post-Deploy Fixes 1) | 28-31 أكتوبر | 24 | 24 | 24 | 0 |
| Sprint 3.6 (Post-Deploy Fixes 2) | 1-9 نوفمبر | 34 | 29 | 29 | 5 (→ Sprint 6) |
| Sprint 4 (B2B Commerce MVP) | 6-11 نوفمبر | ~50 | ~55 | ~50 | 11 (P1 المتبقي) |
| **المجموع** | — | **246** | **250** | **245** | **27** |

**ملاحظة:** Sprint 4 (Commerce) لا يظهر في `00B v6.4` الأصلي — أُضيف بناءً على التحديث [#138].

---

## 🎯 §9: الأولويات الفورية (Next 3 Days — 11-13 نوفمبر)

| اليوم | المهمة | المرجع |
|---|---|---|
| **11-12 نوفمبر** | إغلاق Bundle Logic + Bonus UI (S4-P1-1 + S4-P1-2) | 23-COMMERCE §15 |
| **12-13 نوفمبر** | Tiers UI + Logistics UI (S4-P1-3 + S4-P1-4) | 23-COMMERCE §15 |
| **13 نوفمبر** | اختبار إنشاء طلب + مراجعة Price & Tax (S4-P1-5) | 23-COMMERCE §14 |

**قاعدة صارمة:** لا نبدأ Sprint 5 (PHR) قبل إغلاق كل P1 من Sprint 4.

---

## 🔗 §10: الروابط المرجعية

### الوثائق الشقيقة (يجب قراءتها)

- `00C-CHANGELOG.md` — كل ما تم إنجازه (Sprint 0 → Sprint 4).
- `00D-ROADMAP.md` — الجدول الزمني الكلي + المعالم.
- `00-MASTER-INDEX.md` — الفهرس الشامل + الحالة الفنية لكل Module.

### الوثائق المرتبطة بالمهام الحالية

- `23-COMMERCE-ENGINE.md` — مرجع B2B Commerce (Sprint 4).
- `21-PHR-SYSTEM.md` — مرجع PHR (Sprint 5).
- `05-PSP.md` — مرجع PSP Cycle (Sprint 6).
- `06-POINTS.md` — مرجع Points (Sprint 6).
- `19-PERMISSIONS.md` — مرجع Permissions (Backlog).
- `22-INTEGRATION-HUB.md` — مرجع Integration Hub (Sprint 8+).

### وثائق المعمارية

- `01-ARCHITECTURE.md` — الهيكل المعماري (يُقرأ قبل أي كود).
- `11-PAGE-CREATION.md` — إنشاء الصفحات.
- `04-TRANSLATION.md` — الترجمة.

---

## 📝 §11: سجل تحديثات هذه الوثيقة

| الإصدار | التاريخ | التغييرات |
|---|---|---|
| 6.4 | 5 نوفمبر 2026 | نجاح DashboardFlowPage + OnboardingPage + MedicineGuidePage |
| **7.0** | **11 نوفمبر 2026** | **إعادة كتابة كاملة (Live State Edition)** — فصل التاريخ إلى `00C` + إعادة ترتيب Sprint 4-9 حسب الواقع (B2B Commerce قبل PHR) + توثيق B2B Commerce MVP بناءً على التحديث [#138] + إضافة منهجية الوثيقة في الرأس |

---

© 2026 RubikCare — للاستخدام الداخلي
نهاية الوثيقة ✅
```
