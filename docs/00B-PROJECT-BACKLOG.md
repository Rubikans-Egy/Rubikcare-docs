# 00B — PROJECT-BACKLOG (Live State Edition)

**الاسم:** PROJECT-BACKLOG — سجل التطوير الحي  
**الإصدار:** 7.2 (Live State Edition)  
**آخر تحديث:** 23 نوفمبر 2026  
**Sprint الحالي:** Sprint 6 — PSP Cycle Completion (25-30 نوفمبر)  
**الحالة:** 🟢 حي — يُحدَّث كل جلسة  
**الجمهور:** فريق التطوير + AI Assistants  
**الوثيقة الشقيقة:** [00C-CHANGELOG.md](00C-CHANGELOG.md) (كل ما تم إنجازه)

---

## 📖 منهجية هذه الوثيقة (Methodology)

هذه الوثيقة تتبع 3 قواعد صارمة:

### القاعدة 1: الحالة الحية (Live State Only)
هذه الوثيقة تُجيب على سؤال واحد فقط: *"ما الذي نعمل عليه الآن، وما الذي سيأتي، وما الذي تأجَّل؟"* لا تحتوي على تفاصيل ما تم إنجازه — ذلك في `00C-CHANGELOG.md`.

### القاعدة 2: SP التاريخي (Historical SP Preserved)
نُبقي على إجمالي SP لكل Sprint في جدول واحد مختصر في النهاية (§8) — لمرجعية Velocity فقط. التفاصيل الكاملة لكل مهمة مكتملة موجودة في `00C`.

### القاعدة 3: المصدر الوحيد (Single Source of Truth)
- **هنا فقط:** كل ما هو "قيد التنفيذ" أو "متبقٍّ" أو "مؤجل" أو "مُخطط".
- **في 00C فقط:** كل ما هو "مكتمل" (Done).
- **في 00D فقط:** الجدول الزمني الكلي (Sprint-by-Sprint) والمعالم.

> **قاعدة ذهبية للـ AI Assistant:**  
> عند قراءة هذه الوثيقة، ابدأ من §3 (Sprint 6 الحالي). لا تقرأ §8 (SP التاريخي) إلا إذا احتجت حساب Velocity. لا تفترض أن مهمة "مكتملة" لأنها غير موجودة هنا — ابحث عنها في 00C.

---

## 🎯 الرؤية الحالية (Current Vision — 23 نوفمبر 2026)

RubikCare في مرحلة **Pre-Launch — MVP Completion**. الهدف:

1. إغلاق فجوات PSP Cycle (Sprint 6 — نوفمبر).
2. إنجاز نظام الإشعارات الكامل (Notification Hub).
3. الاستعداد لـ Production Launch (Sprint 7+).

---

## 🟢 §1: Sprint 4 — B2B Commerce MVP Completion (11-17 نوفمبر 2026) — ✅ مُغلق

**الحالة:** مُغلق (اكتمل قبل Sprint 6 — التفاصيل في 00C).

**المرجع:** [23-COMMERCE-ENGINE.md](23-COMMERCE-ENGINE.md) §0.6 (حالة التنفيذ) + §15 (Roadmap المتبقي).

### 🔴 P1 — حرجة (MVP Completion) — ✅ مكتمل

| # | المهمة | SP | الحالة | المرجع |
|---|--------|-----|--------|--------|
| S4-P1-1 | إنهاء منطق حساب الباقات (BundleService + UI) | 2 | ✅ | 23-COMMERCE §15 |
| S4-P1-2 | إنشاء صفحة Bonus (UI + API) | 3 | ✅ | 23-COMMERCE §15 |
| S4-P1-3 | إنشاء صفحة Tiers (UI + API) | 3 | ✅ | 23-COMMERCE §15 |
| S4-P1-4 | إنشاء صفحة Logistics (UI + API) | 2 | ✅ | 23-COMMERCE §15 |
| S4-P1-5 | اختبار إنشاء طلب جديد بعد تعديلات Price & Tax | 1 | ✅ | 23-COMMERCE §14 |

**المجموع P1:** 11 SP — مكتمل

### 🟠 P2 — متوسطة (مؤجل إلى Sprint 7+)

| # | المهمة | SP | الحالة | المرجع |
|---|--------|-----|--------|--------|
| S4-P2-1 | Payments API (ServicePayment) | 2 | ⏳ | 23-COMMERCE §15 |
| S4-P2-2 | Shipments API (ServiceShipment) | 2 | ⏳ | 23-COMMERCE §15 |
| S4-P2-3 | Invoices API (ServiceInvoice) | 2 | ⏳ | 23-COMMERCE §15 |
| S4-P2-4 | Agreements API (PharmacyCompanyAgreement) | 2 | ⏳ | 23-COMMERCE §15 |
| S4-P2-5 | Stock API (PharmaCompanyMedicationStock) | 2 | ⏳ | 23-COMMERCE §15 |
| S4-P2-6 | Ledger UI (صفحة الأرصدة) | 2 | ⏳ | 23-COMMERCE §15 |
| S4-P2-7 | Status History (PharmacySupplyOrderStatusHistory) | 1 | ⏳ | 23-COMMERCE §15 |

**المجموع P2:** 13 SP — مؤجل

### 🟡 P3 — منخفضة (مؤجل إلى Sprint 8+)

| # | المهمة | SP | الحالة |
|---|--------|-----|--------|
| S4-P3-1 | Warehouse Layer (FulfillmentMode = WAREHOUSE) | 8 | ⏳ |
| S4-P3-2 | B2B Points Integration (Seed + Orchestrator) | 5 | ⏳ |
| S4-P3-3 | Pharmacy Stock Page | 2 | ⏳ |
| S4-P3-4 | Online Payments (Paymob/Fawry) | 8 | ⏳ |

**المجموع P3:** 23 SP — مؤجل

---

## 🟠 §2: Sprint 5 — PHR Phase 1 (Hb Navigator) (18-24 نوفمبر 2026) — 🔴 مؤجل

**الحالة:** مؤجل (بسبب تأجيل B2B Commerce — انظر 00D).

**المرجع:** [21-PHR-SYSTEM.md](21-PHR-SYSTEM.md) §1 + §2 + [01-ARCHITECTURE.md](01-ARCHITECTURE.md) §3.

### 🔴 P1 — حرجة (Foundation) — 🔴 مؤجل

| # | المهمة | SP | الحالة | المرجع |
|---|--------|-----|--------|--------|
| S5-P1-1 | Domain: MedicalRecordType, LabTestType, AssessmentRule, AssessmentRuleType | 3 | ⏳ | 21-PHR §1.2 |
| S5-P1-2 | Domain: HealthRecord, LabTestResult | 2 | ⏳ | 21-PHR §1.3 |
| S5-P1-3 | Domain: SharingConsent | 1 | ⏳ | 21-PHR §1.4 |
| S5-P1-4 | Seeds: MedicalRecordType, LabTestType, AssessmentRuleType, AssessmentRule | 2 | ⏳ | 21-PHR §1.2 |
| S5-P1-5 | Migration + EF Configurations | 2 | ⏳ | 01-ARCH §15 |

**المجموع P1:** 10 SP — مؤجل

### 🟠 P2 — متوسطة (منطق التقييم) — 🔴 مؤجل

| # | المهمة | SP | الحالة | المرجع |
|---|--------|-----|--------|--------|
| S5-P2-1 | IHbClinicalRulesEngine + Implementation | 3 | ⏳ | 21-PHR §1.2.3 |
| S5-P2-2 | SaveLabResultUseCase + GetLabResultsUseCase | 2 | ⏳ | 21-PHR §2.1 |
| S5-P2-3 | SaveLabResultValidator (FluentValidation) | 1 | ⏳ | 21-PHR §1.2.2 |
| S5-P2-4 | API: HealthRecordsController (POST + GET) | 2 | ⏳ | 21-PHR §2.1 |

**المجموع P2:** 8 SP — مؤجل

### 🟡 P3 — منخفضة (الواجهة) — 🔴 مؤجل

| # | المهمة | SP | الحالة | المرجع |
|---|--------|-----|--------|--------|
| S5-P3-1 | HbNavigator.razor + .razor.cs | 4 | ⏳ | 21-PHR §2.1 |
| S5-P3-2 | hb-navigator.css | 1 | ⏳ | 01-ARCH §2.8 |
| S5-P3-3 | Chart Component (Blazor أو JSInterop) | 2 | ⏳ | 21-PHR §2.1 |
| S5-P3-4 | Translation Keys AR/EN (SQL MERGE) | 1 | ⏳ | 04-TRANSLATION |
| S5-P3-5 | RTL/LTR + Responsive Design | 2 | ⏳ | 01-ARCH §2.8 |
| S5-P3-6 | Cross-Platform Testing | 2 | ⏳ | 01-ARCH §3.4 |

**المجموع P3:** 12 SP — مؤجل

**المجموع الكلي Sprint 5:** 30 SP — مؤجل

---

## 🟡 §3: Sprint 6 — PSP Cycle Completion (25-30 نوفمبر 2026)

**الهدف:** إغلاق كل ما تبقى من Sprint 3.6 (Post-Deployment Fixes) + ربط B2B Points بـ PSP Cycle + PHR Phase 2 الجزئي + **Notification Hub كامل**.

**المرجع:** [05-PSP.md](05-PSP.md) + [06-POINTS.md](06-POINTS.md) + 00B v6.4 (المهام المؤجلة).

**آخر تحديث:** 23 نوفمبر 2026 — بعد إغلاق Notification Hub (NH-1 → NH-20).

### 🔴 P1 — حرجة (Sprint 3.6 المتبقي)

| # | المهمة | SP | الحالة | المرجع |
|---|--------|-----|--------|--------|
| S6-P1-1 | S3.6-H3: إصلاح Error بدل صفحة النقاط (points-dashboard) | 3 | ✅ مكتمل | 00C |
| S6-P1-2 | S3.6-H4: توليد مواعيد الأدوية — أكتر من دواء في نفس المعاد | 5 | ✅ مكتمل | 00C |
| S6-P1-3 | S3.6-H5: تعديل منطق زر "ابدأ رحلتك العلاجية" | 3 | ⏳ لم يبدأ | 00C |

**المجموع P1:** 11 SP — المكتمل: 8 SP / المتبقي: 3 SP

### 🟠 P2 — متوسطة (Sprint 1 المتبقي — Reports Integration)

| # | المهمة | SP | الحالة | المرجع |
|---|--------|-----|--------|--------|
| S6-P2-1 | P5.1: GetAdherenceHandler — إضافة Points | 2 | ⏳ | 01-ARCH §6ب (IC-07) |
| S6-P2-2 | P5.2: GetPharmacyFinancialHandler — فصل PointsDiscount | 3 | ⏳ | 01-ARCH §6ب (IC-08) |
| S6-P2-3 | P5.3: GetProgramLifecycleHandler — إضافة Points | 2 | ⏳ | 01-ARCH §6ب (IC-09) |

**المجموع P2:** 7 SP

### 🟡 P3 — منخفضة (Sprint 3.6 Medium + Low)

| # | المهمة | SP | الحالة |
|---|--------|-----|--------|
| S6-P3-1 | S3.6-M1: تظبيط تاريخ الأوردر في "طلباتي" | 2 | ⏳ |
| S6-P3-2 | S3.6-M2: صفحة "دعواتي" من عند الدكتور | 3 | ⏳ |
| S6-P3-3 | S3.6-M3: الإشعارات (مراجعة عامة) | 3 | ⏳ |
| S6-P3-4 | S3.6-L1: تعديل صفحات المقدمة (Landing Pages) | 5 | ⏳ |

**المجموع P3:** 13 SP

---

### ✅ §3.1: مهام إضافية مكتملة (Sprint 6 — أثناء S6-P1-1 و S6-P1-2)

*(جدول مختصر — التفاصيل في 00C)*

| # | المهمة | SP | الحالة |
|---|--------|-----|--------|
| ADD-1 | my-psp-orders — إصلاح (كل participations) | 3 | ✅ |
| ADD-2 | psp-order-by-token endpoint | 2 | ✅ |
| ADD-3 | PspPatientOrderDto — إضافة GroupCategory + CanConfirmReceipt + IsOverdue | 2 | ✅ |
| ADD-4 | PatientOrdersPage — 5 مجموعات + Badge "متأخر" | 3 | ✅ |
| ADD-5 | PspController.GetPatientProgramDetails — ThenByDescending(CreatedDate) | 1 | ✅ |
| ADD-6 | ترجمة SHARED.ORDER_TRACKER.* (10 مفاتيح SQL MERGE) | 1 | ✅ |
| ADD-7 | GetProgramsNeedingScheduleAsync — إصلاح ProgramID من ParticipationID | 2 | ✅ |
| ADD-8 | PspSchedulePage — توليد على مستوى الدواء | 3 | ✅ |
| ADD-9 | MedicationSchedulePage — زر PSP + _showAllProgramsInModal | 2 | ✅ |
| ADD-10 | PspSchedulePage.OnInitializedAsync — LoadTranslationsAsync + _isTranslationsLoaded | 1 | ✅ |

**المجموع الإضافي:** 20 SP — مكتمل

---

### ✅ §3.3: مهام Notification Hub مكتملة (الجلسة الحالية — 22-23 نوفمبر 2026)

| # | المهمة | SP | الحالة |
|---|--------|-----|--------|
| NH-1 | إصلاح أخطاء Build (PspProgramDto, OrganizationTypeId, _targetProgramId) | 2 | ✅ |
| NH-2 | إنشاء Entity: TemplateCategoryRule + TemplateSubCategory | 3 | ✅ |
| NH-3 | إنشاء Migration: AddTemplateClassificationSystem | 2 | ✅ |
| NH-4 | إنشاء TemplateAuthorizationService (Backend Validation) | 3 | ✅ |
| NH-5 | تحديث NotificationTemplatesController (صلاحيات + Endpoints جديدة) | 3 | ✅ |
| NH-6 | تحديث DTOs (CreateTemplateDto, UpdateTemplateDto, TemplateDetailDto) | 2 | ✅ |
| NH-7 | تحديث INotificationHubClientService + Implementation | 2 | ✅ |
| NH-8 | تحديث NotificationHub.razor (Dynamic Categories + SubCategories) | 5 | ✅ |
| NH-9 | إضافة Priority, ExpiryDays, RequiresAcknowledgment, MedicalDisclaimer | 2 | ✅ |
| NH-10 | إصلاح Lag المودال (Open First, Load Later Pattern) | 3 | ✅ |
| NH-11 | إضافة Caching للبرامج (تحميل مرة واحدة فقط) | 2 | ✅ |
| NH-12 | إضافة Skeleton Loader + زر تحديث يدوي | 2 | ✅ |
| NH-13 | تحديث Dashboard الإحصائيات (Runs Tab → KPI Cards + Bar Chart) | 4 | ✅ |
| NH-14 | ربط القوالب ببرامج PSP (ProgramId + Migration) | 2 | ✅ |
| NH-15 | تطبيق مصفوفة الصلاحيات (6 فئات × 4 أنواع منظمات) | 3 | ✅ |
| NH-16 | تطبيق مصفوفة الاستهداف (Campaign Target Users) | 3 | ✅ |
| NH-17 | تحديث TargetResolverService (عزل صارم حسب orgType) | 4 | ✅ |
| NH-18 | إضافة مفاتيح ترجمة جديدة (SQL MERGE — 30+ مفتاح) | 1 | ✅ |
| NH-19 | إصلاح خطأ Circuit Terminated (try-catch شامل) | 2 | ✅ |
| NH-20 | إضافة CSS احترافي (Skeleton, Alerts, Badges, Dashboard) | 2 | ✅ |

**المجموع:** 50 SP — ✅ مكتمل

---

### 🔴 §3.2: مشاكل معلقة (Blocked Issues)

| # | المشكلة | الأولوية | السبب | الحل المقترح |
|---|---------|----------|-------|--------------|
| BLOCK-1 | `patient-details` endpoint بيرجع `403 Forbidden` لو `patientId=15` (PatientProfileID) بدل `CaseID` | 🟡 متوسطة | `PspController.GetPatientProgramDetails` بيدور بـ `PatientProfileID` بدل `CaseID` | تعديل الـ filter ليدور بـ `PatientProfileID == currentUserId && CaseID == patientId` |

**المجموع:** 1 مشكلة معلقة — ~2 SP

---

### 📊 ملخص Sprint 6 (محدث)

| الفئة | SP | الحالة |
|-------|-----|--------|
| P1 المكتمل (S6-P1-1 + S6-P1-2) | 8 | ✅ مكتمل |
| إضافي (ADD-1 → ADD-10) | 20 | ✅ مكتمل |
| **Notification Hub (NH-1 → NH-20)** | **50** | **✅ مكتمل** |
| P1 المتبقي (S6-P1-3) | 3 | ⏳ لم يبدأ |
| P2 (Reports Integration) | 7 | ⏳ |
| P3 (Medium + Low) | 13 | ⏳ |
| Blocked (BLOCK-1) | 2 | ⏳ |
| **المجموع الكلي** | **103 SP** | |
| **المكتمل** | **78 SP** | ✅ |
| **المتبقي** | **25 SP** | ⏳ |

**المجموع الكلي Sprint 6 (الأصلي):** 31 SP → **المحدث:** 103 SP (بسبب المشاكل الإضافية + Notification Hub).

---

### 📌 ملاحظات على Sprint 6

1. **S6-P1-1 و S6-P1-2 اكتملوا** — والـ SP الحقيقي كان أكبر من المتوقع (بسبب المشاكل الإضافية).
2. **Notification Hub (NH-1 → NH-20) اكتمل بالكامل** — 50 SP من العمل المكثف.
3. **S6-P1-3** هو المهمة القادمة (زر "ابدأ رحلتك العلاجية").
4. **BLOCK-1** (403 في `patient-details`) — مشكلة قديمة، لسه محتاجة حل.
5. **التقدير الأولي لـ Sprint 6 (31 SP)** كان **أقل بكثير من الواقع** — الـ SP الفعلي 103 SP.
6. **Velocity الفريق ارتفع** من ~30 SP/Sprint إلى ~45 SP/Sprint.

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

## 📊 §6: Velocity Chart (مختصر — آخر 4 Sprint فقط)

| Sprint | Planned | Actual | Ratio | الحالة |
|--------|---------|--------|-------|--------|
| Sprint 3 | 53 SP | 58 SP | 109% ✅ | مكتمل (في 00C) |
| Sprint 3.5 | 24 SP | 24 SP | 100% ✅ | مكتمل (في 00C) |
| Sprint 3.6 | 34 SP | 29 SP | 85% 🟡 | مكتمل جزئياً (5 SP مؤجلة → Sprint 6) |
| Sprint 4 (Commerce) | 11 SP (P1) | ~55 SP | — | 🟡 قيد التنفيذ |

**المعدل الفعلي للفريق:** ~40-45 SP/Sprint (بعد استقرار Sprint 2-3 وإنجاز Notification Hub).

---

## 🕰️ §7: SP التاريخي لكل Sprint (مرجعي فقط)

هذا الجدول للمرجعية فقط. التفاصيل الكاملة لكل مهمة مكتملة موجودة في `00C-CHANGELOG.md`.

| Sprint | الفترة | SP Planned | SP Actual | SP Done | SP متبقٍّ |
|--------|--------|------------|-----------|---------|-----------|
| Sprint 0 (Foundation) | 16-22 أكتوبر | 22 | 11 | 11 | 11 (مؤجلة إلى ما بعد MVP) |
| Sprint 1 (Points-PSP Integration) | 22-23 أكتوبر | 36 | 46 | 46 | 11 (P5.1-P5.3 → Sprint 6) |
| Sprint 2 (MAUI Stabilization) | 24-25 أكتوبر | 27 | 27 | 27 | 0 |
| Sprint 3 (PSP Dispensation) | 26-28 أكتوبر | 53 | 58 | 58 | 0 |
| Sprint 3.5 (Post-Deploy Fixes 1) | 28-31 أكتوبر | 24 | 24 | 24 | 0 |
| Sprint 3.6 (Post-Deploy Fixes 2) | 1-9 نوفمبر | 34 | 29 | 29 | 5 (→ Sprint 6) |
| Sprint 4 (B2B Commerce MVP) | 6-11 نوفمبر | ~50 | ~55 | ~50 | 11 (P1 المتبقي) |
| **Sprint 6 (PSP Cycle + Notifications)** | **25-30 نوفمبر** | **31** | **103** | **78** | **25** |
| **المجموع** | — | **277** | **349** | **323** | **63** |

**ملاحظة:** Sprint 4 (Commerce) و Sprint 6 (Notifications) لا يظهران في 00B v6.4 الأصلي — أُضيفا بناءً على التحديثات الأخيرة.

---

## 🏗️ §8: Technical Debt Register (المتبقي فقط)

### 🔴 Critical Debt

| # | الدين | الموقع | المرجع |
|---|-------|--------|--------|
| TD-C1 | انتهاك معماري: `@page` في Shared.UI | PharmacyDetailPage.razor | 01-ARCH §2.8 |
| TD-C2 | خلط Namespaces | PharmacySearchPage.razor | 01-ARCH §2.8 |
| TD-C3 | 404 في `/admin/pending-licenses` | SystemManagment vs SystemManagement | — |

### 🟠 Medium Debt

| # | الدين | الموقع | المرجع |
|---|-------|--------|--------|
| TD-M1 | زر البحث في Dashboard غير مربوط | Pages/Dashboard.razor | — |
| TD-M2 | ازدواجية FirebaseNotificationService | Api.Web + Infrastructure | — |
| TD-M3 | ازدواجية IPSPPointSchemeService | Application + Web | — |
| TD-M4 | ازدواجية MedicineGuideFlow (إذا وُجد) | Mobile/Features/PSP/Patient/Views/ | — |
| TD-M5 | TD-001: الطلبات المكررة (Double API Calls) | Rubikcare.Web + Api.Web | 00C (تفاصيل) |

### 🟢 Low Debt

| # | الدين | الموقع |
|---|-------|--------|
| TD-L1 | IMobileNavigationService (اسم مضلل) | Shared.UI/Services/ |
| TD-L2 | HttpUtility في Blazor WASM | PharmacySearch.razor |
| TD-L3 | alert() بدلاً من Toast | PspSearch.razor |
| TD-L4 | SystemMenuMenuID Shadow Property | MenuAssignments |
| TD-L5 | RoleId في MenuVisibilityRule | بعد Migration |

---

## 📋 §9: Backlog المستقبلي (Product Backlog — Priorities)

### 🔴 High Priority (بعد Sprint 9)

| # | User Story | Epic | SP | المرجع |
|---|------------|------|-----|--------|
| B-01 | كـ Admin، أريد إدارة Control Levels | Permissions | 5 | 19-PERMISSIONS |
| B-02 | كـ Admin، أريد ربط Users بـ Control Levels | Permissions | 3 | 19-PERMISSIONS |
| B-03 | كـ Super Admin، أريد إدارة Permissions | Permissions | 5 | 19-PERMISSIONS |
| B-04 | كـ Admin، أريد إدارة اشتراك المنظمة | Subscriptions | 5 | 03-MENUS |
| B-05 | كـ Admin، أريد ربط SubscriptionPlan بـ Menus | Menus | 5 | 03-MENUS |
| B-06 | كـ Rep، أريد رؤية اسم الصيدلية/العيادة بعد قبول الدعوة | Rep | 5 | 13-Rep Epic |
| B-07 | كـ Rep، أريد إشعار عند قبول الدعوة | Rep | 3 | 13-Rep Epic |

### 🟠 Medium Priority

| # | User Story | Epic | SP | المرجع |
|---|------------|------|-----|--------|
| B-10 | Menus ديناميكية في PWA | Menus | 5 | 03-MENUS |
| B-11 | Menus ديناميكية في MAUI | Menus | 5 | 03-MENUS |
| B-12 | ربط ServiceProduct بـ MenuItems | Menus | 3 | 03-MENUS |
| B-13 | ApiClient موحّد (Integration Hub) | Integration Hub | 8 | 22-INTEGRATION-HUB |
| B-14 | API-Contracts (OpenAPI) | Integration Hub | 5 | 22-INTEGRATION-HUB |

### 🟡 Low Priority

| # | User Story | Epic | SP | المرجع |
|---|------------|------|-----|--------|
| B-20 | إدارة OrganizationGroups | Organization | 5 | 01-ARCH |
| B-21 | Rubika Currency Engine | Rubika | 8 | 06-POINTS |
| B-22 | Deep Linking من QR | Mobile | 3 | 13-MAUI |
| B-23 | Warehouse Layer (B2B Phase 2) | Commerce | 8 | 23-COMMERCE |
| B-24 | B2B Points Integration | Commerce | 5 | 23-COMMERCE |
| B-25 | Online Payments (Paymob/Fawry) | Commerce | 8 | 23-COMMERCE |

---

## 🎯 §10: الأولويات الفورية (Next 3 Days — 24-26 نوفمبر 2026)

| اليوم | المهمة | المرجع |
|-------|--------|--------|
| 24 نوفمبر | S6-P1-3: تعديل منطق زر "ابدأ رحلتك العلاجية" | 00C |
| 25 نوفمبر | S6-P2-1: GetAdherenceHandler — إضافة Points | 01-ARCH §6ب |
| 26 نوفمبر | S6-P2-2: GetPharmacyFinancialHandler — فصل PointsDiscount | 01-ARCH §6ب |

**قاعدة صارمة:** لا نبدأ Sprint 7 قبل إغلاق كل P1 من Sprint 6.

---

## 🔗 §11: الروابط المرجعية

### الوثائق الشقيقة (يجب قراءتها)
- [00C-CHANGELOG.md](00C-CHANGELOG.md) — كل ما تم إنجازه (Sprint 0 → Sprint 6).
- [00D-ROADMAP.md](00D-ROADMAP.md) — الجدول الزمني الكلي + المعالم.
- [00-MASTER-INDEX.md](00-MASTER-INDEX.md) — الفهرس الشامل + الحالة الفنية لكل Module.

### الوثائق المرتبطة بالمهام الحالية
- [23-COMMERCE-ENGINE.md](23-COMMERCE-ENGINE.md) — مرجع B2B Commerce (Sprint 4).
- [21-PHR-SYSTEM.md](21-PHR-SYSTEM.md) — مرجع PHR (Sprint 5).
- [05-PSP.md](05-PSP.md) — مرجع PSP Cycle (Sprint 6).
- [06-POINTS.md](06-POINTS.md) — مرجع Points (Sprint 6).
- [19-PERMISSIONS.md](19-PERMISSIONS.md) — مرجع Permissions (Backlog).
- [22-INTEGRATION-HUB.md](22-INTEGRATION-HUB.md) — مرجع Integration Hub (Sprint 8+).

### وثائق المعمارية
- [01-ARCHITECTURE.md](01-ARCHITECTURE.md) — الهيكل المعماري (يُقرأ قبل أي كود).
- [11-PAGE-CREATION.md](11-PAGE-CREATION.md) — إنشاء الصفحات.
- [04-TRANSLATION.md](04-TRANSLATION.md) — الترجمة.

---

## 📝 §12: سجل تحديثات هذه الوثيقة

| الإصدار | التاريخ | التغييرات |
|---------|---------|-----------|
| 6.4 | 5 نوفمبر 2026 | نجاح DashboardFlowPage + OnboardingPage + MedicineGuidePage |
| 7.0 | 11 نوفمبر 2026 | إعادة كتابة كاملة (Live State Edition) — فصل التاريخ إلى 00C + إعادة ترتيب Sprint 4-9 حسب الواقع (B2B Commerce قبل PHR) + توثيق B2B Commerce MVP بناءً على التحديث [#138] + إضافة منهجية الوثيقة في الرأس |
| **7.2** | **23 نوفمبر 2026** | **إضافة §3.3 (Notification Hub — 20 مهمة، 50 SP) + تحديث ملخص Sprint 6 (103 SP إجمالي) + تحديث Velocity Chart + تحديث Technical Debt Register + إضافة ملاحظات Sprint 6** |

---

## 📋 قواعد المشروع الذهبية

- ❌ لا تعدل Migration قديم — Migration جديد فقط.
- ✅ استخدم `IGenericService<T>` بدل DbContext مباشرة.
- ✅ `N''` prefix للعربية في SQL.
- ✅ `MERGE` بدل `INSERT`/`UPDATE`.
- ✅ استخدم `T("KEY")` للترجمة.
- ❌ لا `Set-Content` لملفات `.razor`/`.cshtml` (تتلف العربية).
- ✅ كل رد يبدأ بـ `[#رقم]` (ترقيم تصاعدي).
- ✅ خطوة واحدة لكل رد.
- ✅ مناقشة قبل التنفيذ.

---

**نهاية 00B — PROJECT-BACKLOG (v7.2) — 23 نوفمبر 2026**
