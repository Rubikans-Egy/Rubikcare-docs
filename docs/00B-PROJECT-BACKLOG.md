# 00B — PROJECT-BACKLOG (Live State Edition)

الاسم: PROJECT-BACKLOG — سجل التطوير الحي
الإصدار: 7.1 (Live State Edition)
آخر تحديث: 17 نوفمبر 2026
Sprint الحالي: Sprint 6 — PSP Cycle Completion (25-30 نوفمبر)
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
> عند قراءة هذه الوثيقة، ابدأ من **§3 (Sprint 6 الحالي)**. لا تقرأ §8 (SP التاريخي) إلا إذا احتجت حساب Velocity. لا تفترض أن مهمة "مكتملة" لأنها غير موجودة هنا — ابحث عنها في `00C`.

---

## 🎯 الرؤية الحالية (Current Vision — 17 نوفمبر 2026)

RubikCare في مرحلة **Pre-Launch — MVP Completion**. الهدف:

1. **إغلاق فجوات PSP Cycle** (Sprint 6 — ديسمبر).
2. **الاستعداد لـ Production Launch** (Sprint 7+).

**ملاحظة مهمة:** Sprint 6P1-1 و P1-2 اكتملوا (خلال الجلسة الحالية). P1-3 (زر "ابدأ رحلتك") هو المهمة القادمة.

---

## 🟢 §1: Sprint 4 — B2B Commerce MVP Completion (11-17 نوفمبر 2026) — ✅ مُغلق

**الحالة:** مُغلق (اكتمل قبل Sprint 6 — التفاصيل في `00C`).

**المرجع:** `23-COMMERCE-ENGINE.md` §0.6 (حالة التنفيذ) + §15 (Roadmap المتبقي).

### 🔴 P1 — حرجة (MVP Completion) — ✅ مكتمل

| # | المهمة | SP | الحالة | المرجع |
|---|---|---|---|---|
| S4-P1-1 | إنهاء منطق حساب الباقات (`BundleService` + UI) | 2 | ✅ | 23-COMMERCE §15 |
| S4-P1-2 | إنشاء صفحة Bonus (UI + API) | 3 | ✅ | 23-COMMERCE §15 |
| S4-P1-3 | إنشاء صفحة Tiers (UI + API) | 3 | ✅ | 23-COMMERCE §15 |
| S4-P1-4 | إنشاء صفحة Logistics (UI + API) | 2 | ✅ | 23-COMMERCE §15 |
| S4-P1-5 | اختبار إنشاء طلب جديد بعد تعديلات Price & Tax | 1 | ✅ | 23-COMMERCE §14 |

**المجموع P1:** 11 SP — **مكتمل**

### 🟠 P2 — متوسطة (مؤجل إلى Sprint 7+)

| # | المهمة | SP | الحالة | المرجع |
|---|---|---|---|---|
| S4-P2-1 | Payments API (ServicePayment) | 2 | ⏳ | 23-COMMERCE §15 |
| S4-P2-2 | Shipments API (ServiceShipment) | 2 | ⏳ | 23-COMMERCE §15 |
| S4-P2-3 | Invoices API (ServiceInvoice) | 2 | ⏳ | 23-COMMERCE §15 |
| S4-P2-4 | Agreements API (PharmacyCompanyAgreement) | 2 | ⏳ | 23-COMMERCE §15 |
| S4-P2-5 | Stock API (PharmaCompanyMedicationStock) | 2 | ⏳ | 23-COMMERCE §15 |
| S4-P2-6 | Ledger UI (صفحة الأرصدة) | 2 | ⏳ | 23-COMMERCE §15 |
| S4-P2-7 | Status History (PharmacySupplyOrderStatusHistory) | 1 | ⏳ | 23-COMMERCE §15 |

**المجموع P2:** 13 SP — **مؤجل**

### 🟡 P3 — منخفضة (مؤجل إلى Sprint 8+)

| # | المهمة | SP | الحالة |
|---|---|---|---|
| S4-P3-1 | Warehouse Layer (FulfillmentMode = WAREHOUSE) | 8 | ⏳ |
| S4-P3-2 | B2B Points Integration (Seed + Orchestrator) | 5 | ⏳ |
| S4-P3-3 | Pharmacy Stock Page | 2 | ⏳ |
| S4-P3-4 | Online Payments (Paymob/Fawry) | 8 | ⏳ |

**المجموع P3:** 23 SP — **مؤجل**

---

## 🟠 §2: Sprint 5 — PHR Phase 1 (Hb Navigator) (18-24 نوفمبر 2026) — 🔴 مؤجل

**الحالة:** مؤجل (بسبب تأجيل B2B Commerce — انظر 00D).

**المرجع:** `21-PHR-SYSTEM.md` §1 + §2 + `01-ARCHITECTURE.md` §3.

### 🔴 P1 — حرجة (Foundation) — 🔴 مؤجل

| # | المهمة | SP | الحالة | المرجع |
|---|---|---|---|---|
| S5-P1-1 | Domain: `MedicalRecordType`, `LabTestType`, `AssessmentRule`, `AssessmentRuleType` | 3 | ⏳ | 21-PHR §1.2 |
| S5-P1-2 | Domain: `HealthRecord`, `LabTestResult` | 2 | ⏳ | 21-PHR §1.3 |
| S5-P1-3 | Domain: `SharingConsent` | 1 | ⏳ | 21-PHR §1.4 |
| S5-P1-4 | Seeds: `MedicalRecordType`, `LabTestType`, `AssessmentRuleType`, `AssessmentRule` | 2 | ⏳ | 21-PHR §1.2 |
| S5-P1-5 | Migration + EF Configurations | 2 | ⏳ | 01-ARCH §15 |

**المجموع P1:** 10 SP — **مؤجل**

### 🟠 P2 — متوسطة (منطق التقييم) — 🔴 مؤجل

| # | المهمة | SP | الحالة | المرجع |
|---|---|---|---|---|
| S5-P2-1 | `IHbClinicalRulesEngine` + Implementation | 3 | ⏳ | 21-PHR §1.2.3 |
| S5-P2-2 | `SaveLabResultUseCase` + `GetLabResultsUseCase` | 2 | ⏳ | 21-PHR §2.1 |
| S5-P2-3 | `SaveLabResultValidator` (FluentValidation) | 1 | ⏳ | 21-PHR §1.2.2 |
| S5-P2-4 | API: `HealthRecordsController` (POST + GET) | 2 | ⏳ | 21-PHR §2.1 |

**المجموع P2:** 8 SP — **مؤجل**

### 🟡 P3 — منخفضة (الواجهة) — 🔴 مؤجل

| # | المهمة | SP | الحالة | المرجع |
|---|---|---|---|---|
| S5-P3-1 | `HbNavigator.razor` + `.razor.cs` | 4 | ⏳ | 21-PHR §2.1 |
| S5-P3-2 | `hb-navigator.css` | 1 | ⏳ | 01-ARCH §2.8 |
| S5-P3-3 | Chart Component (Blazor أو JSInterop) | 2 | ⏳ | 21-PHR §2.1 |
| S5-P3-4 | Translation Keys AR/EN (SQL MERGE) | 1 | ⏳ | 04-TRANSLATION |
| S5-P3-5 | RTL/LTR + Responsive Design | 2 | ⏳ | 01-ARCH §2.8 |
| S5-P3-6 | Cross-Platform Testing | 2 | ⏳ | 01-ARCH §3.4 |

**المجموع P3:** 12 SP — **مؤجل**

**المجموع الكلي Sprint 5:** 30 SP — **مؤجل**

---

## 🟡 §3: Sprint 6 — PSP Cycle Completion (25-30 نوفمبر 2026)

**الهدف:** إغلاق كل ما تبقى من Sprint 3.6 (Post-Deployment Fixes) + ربط B2B Points بـ PSP Cycle + PHR Phase 2 الجزئي.

**المرجع:** `05-PSP.md` + `06-POINTS.md` + `00B v6.4` (المهام المؤجلة).

**آخر تحديث:** 17 نوفمبر 2026 — بعد إغلاق S6-P1-1 و S6-P1-2.

### 🔴 P1 — حرجة (Sprint 3.6 المتبقي)

| # | المهمة | SP | الحالة | المرجع |
|---|---|---|---|---|
| S6-P1-1 | S3.6-H3: إصلاح Error بدل صفحة النقاط (`points-dashboard`) | 3 | ✅ **مكتمل** | 00C |
| S6-P1-2 | S3.6-H4: توليد مواعيد الأدوية — أكتر من دواء في نفس المعاد | 5 | ✅ **مكتمل** | 00C |
| S6-P1-3 | S3.6-H5: تعديل منطق زر "ابدأ رحلتك العلاجية" | 3 | ⏳ **لم يبدأ** | 00C |

**المجموع P1:** 11 SP — **المكتمل: 8 SP** / **المتبقي: 3 SP**

### 🟠 P2 — متوسطة (Sprint 1 المتبقي — Reports Integration)

| # | المهمة | SP | الحالة | المرجع |
|---|---|---|---|---|
| S6-P2-1 | P5.1: `GetAdherenceHandler` — إضافة Points | 2 | ⏳ | 01-ARCH §6ب (IC-07) |
| S6-P2-2 | P5.2: `GetPharmacyFinancialHandler` — فصل PointsDiscount | 3 | ⏳ | 01-ARCH §6ب (IC-08) |
| S6-P2-3 | P5.3: `GetProgramLifecycleHandler` — إضافة Points | 2 | ⏳ | 01-ARCH §6ب (IC-09) |

**المجموع P2:** 7 SP

### 🟡 P3 — منخفضة (Sprint 3.6 Medium + Low)

| # | المهمة | SP | الحالة |
|---|---|---|---|
| S6-P3-1 | S3.6-M1: تظبيط تاريخ الأوردر في "طلباتي" | 2 | ⏳ |
| S6-P3-2 | S3.6-M2: صفحة "دعواتي" من عند الدكتور | 3 | ⏳ |
| S6-P3-3 | S3.6-M3: الإشعارات (مراجعة عامة) | 3 | ⏳ |
| S6-P3-4 | S3.6-L1: تعديل صفحات المقدمة (Landing Pages) | 5 | ⏳ |

**المجموع P3:** 13 SP

---

### ✅ §3.1: مهام إضافية مكتملة (Sprint 6 — أثناء S6-P1-1 و S6-P1-2)

*(جدول مختصر — التفاصيل في `00C`)*

| # | المهمة | SP | الحالة |
|---|---|---|---|
| ADD-1 | `my-psp-orders` — إصلاح (كل participations) | 3 | ✅ |
| ADD-2 | `psp-order-by-token` endpoint | 2 | ✅ |
| ADD-3 | `PspPatientOrderDto` — إضافة `GroupCategory` + `CanConfirmReceipt` + `IsOverdue` | 2 | ✅ |
| ADD-4 | `PatientOrdersPage` — 5 مجموعات + Badge "متأخر" | 3 | ✅ |
| ADD-5 | `PspController.GetPatientProgramDetails` — `ThenByDescending(CreatedDate)` | 1 | ✅ |
| ADD-6 | ترجمة `SHARED.ORDER_TRACKER.*` (10 مفاتيح SQL MERGE) | 1 | ✅ |
| ADD-7 | `GetProgramsNeedingScheduleAsync` — إصلاح `ProgramID` من `ParticipationID` | 2 | ✅ |
| ADD-8 | `PspSchedulePage` — توليد على مستوى الدواء | 3 | ✅ |
| ADD-9 | `MedicationSchedulePage` — زر PSP + `_showAllProgramsInModal` | 2 | ✅ |
| ADD-10 | `PspSchedulePage.OnInitializedAsync` — `LoadTranslationsAsync` + `_isTranslationsLoaded` | 1 | ✅ |

**المجموع الإضافي:** 20 SP — **مكتمل**

---

### 🔴 §3.2: مشاكل معلقة (Blocked Issues)

| # | المشكلة | الأولوية | السبب | الحل المقترح |
|---|---|---|---|---|
| BLOCK-1 | `patient-details` endpoint بيرجع `403 Forbidden` لو `patientId=15` (PatientProfileID) بدل `CaseID` | 🟡 متوسطة | `PspController.GetPatientProgramDetails` بيدور بـ `PatientProfileID` بدل `CaseID` | تعديل الـ filter ليدور بـ `PatientProfileID == currentUserId && CaseID == patientId` |

**المجموع:** 1 مشكلة معلقة — ~2 SP

---

### 📊 ملخص Sprint 6 (محدث)

| الفئة | SP | الحالة |
|---|---|---|
| P1 المكتمل (S6-P1-1 + S6-P1-2) | 8 | ✅ مكتمل |
| إضافي (ADD-1 → ADD-10) | 20 | ✅ مكتمل |
| P1 المتبقي (S6-P1-3) | 3 | ⏳ لم يبدأ |
| P2 (Reports Integration) | 7 | ⏳ |
| P3 (Medium + Low) | 13 | ⏳ |
| Blocked (BLOCK-1) | 2 | ⏳ |
| **المجموع الكلي** | **53 SP** | |
| **المكتمل** | **28 SP** | ✅ |
| **المتبقي** | **25 SP** | ⏳ |

**المجموع الكلي Sprint 6 (الأصلي):** 31 SP → **المحدث:** 53 SP (بسبب المشاكل الإضافية).

---

### 📌 ملاحظات على Sprint 6

1. **S6-P1-1 و S6-P1-2 اكتملوا** — والـ SP الحقيقي كان أكبر من المتوقع (بسبب المشاكل الإضافية).
2. **S6-P1-3** هو المهمة القادمة (زر "ابدأ رحلتك العلاجية").
3. **BLOCK-1** (403 في `patient-details`) — مشكلة قديمة، لسه محتاجة حل.
4. **التقدير الأولي لـ Sprint 6 (31 SP)** كان **أقل من الواقع** — الـ SP الفعلي 53 SP.

---

## 📅 §4: Sprint 7 — Production Launch Preparation (ديسمبر 2026 — يناير 2027)

**الحالة:** ⏳ مُخطط

**الهدف:** الاستعداد لـ Production Launch.

**المهام المقترحة:**
- Security Audit + Penetration Testing.
- Performance Optimization (Frontend + Backend).
- Load Testing (1000+ concurrent users).
- Documentation + User Manuals.
- Training Material.

---

## 📅 §5: Sprint 8+ — Post-Launch Features (2027)

**الحالة:** ⏳ مُخطط

**المهام المقترحة:**
- Online Payments (Paymob/Fawry).
- Warehouse Layer (FulfillmentMode = WAREHOUSE).
- Advanced Analytics + Reports.
- Mobile App Enhancements.
- Integration Hub (External APIs).

---

## 📊 §8: SP التاريخي (Historical SP — Velocity Reference)

| Sprint | SP المخطط | SP الفعلي | الملاحظات |
|---|---|---|---|
| Sprint 1 | 25 | 28 | +3 (مهام إضافية) |
| Sprint 2 | 30 | 32 | +2 |
| Sprint 3 | 35 | 38 | +3 |
| Sprint 3.5 | 20 | 22 | +2 |
| Sprint 3.6 | 25 | 28 | +3 |
| Sprint 4 | 30 | 47 | +17 (B2B Commerce MVP) |
| Sprint 5 | 30 | 0 | مؤجل |
| **Sprint 6** | **31** | **53** | **+22 (مهام إضافية + BLOCK-1)** |

**متوسط Velocity:** ~30-35 SP/Sprint.

---

## 📋 قواعد المشروع الذهبية

- ❌ لا تعدل Migration قديم — Migration جديد فقط.
- ✅ استخدم `IGenericService<T>` بدل `DbContext` مباشرة.
- ✅ `N''` prefix للعربية في SQL.
- ✅ `MERGE` بدل `INSERT/UPDATE`.
- ✅ استخدم `T("KEY")` للترجمة.
- ❌ لا `Set-Content` لملفات `.razor`/`.cshtml` (تتلف العربية).
- ✅ كل رد يبدأ بـ `[#رقم]` (ترقيم تصاعدي).
- ✅ خطوة واحدة لكل رد.
- ✅ مناقشة قبل التنفيذ.

---

**نهاية `00B — PROJECT-BACKLOG` (v7.1) — 17 نوفمبر 2026**

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
