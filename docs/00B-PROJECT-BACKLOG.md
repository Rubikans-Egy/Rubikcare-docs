
# 00B — PROJECT-BACKLOG (Realigned Edition)

**الاسم:** PROJECT-BACKLOG — سجل التطوير الحي 

**الإصدار:** 8.0 (Realigned Edition — Sprint Plan v8.0) 

**آخر تحديث:** 25 نوفمبر 2026 

**Sprint الحالي:** S09 — Notification Hub — Phase 2 (25 نوفمبر — 10 ديسمبر 2026) 
**الحالة:** 🟢 حي — يُحدَّث كل جلسة 

**الجمهور:** فريق التطوير + AI Assistants 

**الوثائق الشقيقة:** `00C-CHANGELOG.md` (ما تم إنجازه) + `00D-ROADMAP.md` (الجدول الزمني الكلي)


---

## 📖 منهجية هذه الوثيقة (Methodology)

### القاعدة 1: الحالة الحية (Live State Only)
هذه الوثيقة تُجيب على سؤال واحد: **"ما الذي نعمل عليه الآن، وما الذي سيأتي، وما الذي تأجَّل؟"** لا تحتوي على تفاصيل ما تم إنجازه — ذلك في `00C-CHANGELOG.md`.

### القاعدة 2: الوثيقة تطابق الكود (Doc = Code)
أي تعارض بين هذه الوثيقة والكود الفعلي يُحلّ **لصالح الكود**. الوثيقة تُحدَّث لتطابق الواقع، وليس العكس.

### القاعدة 3: كل فجوة موثقة (All Gaps Documented)
كل فجوة بين "ما هو مخطط" و"ما هو منفّذ" تُوثَّق في **§7 Gaps Register** برقم `GAP-NN` وحالة واضحة.

### القاعدة 4: المصدر الوحيد (Single Source of Truth)
- **هنا فقط:** كل ما هو "قيد التنفيذ" أو "متبقٍّ" أو "مؤجل" أو "مُخطط".
- **في `00C` فقط:** كل ما هو "مكتمل" (Done).
- **في `00D` فقط:** الجدول الزمني الكلي (Sprint-by-Sprint) والمعالم.

### القاعدة 5: Sprint = وحدة قابلة للقياس
كل Sprint له: هدف واضح، مخرجات، SP، حالة، تاريخ. لا Sprint بدون هذه العناصر.

**قاعدة ذهبية للـ AI Assistant:**
> عند قراءة هذه الوثيقة، ابدأ من **§3 (Sprint Current)**. لا تقرأ §8 (Sprint History) إلا إذا احتجت حساب Velocity. لا تفترض أن مهمة "مكتملة" لأنها غير موجودة هنا — ابحث عنها في `00C`.

---

## 🎯 الرؤية الحالية (Current Vision — 25 نوفمبر 2026)

RubikCare في مرحلة **Pre-Launch — MVP Completion**. الهدف:

1. **إكمال Notification Hub** (S09 — نشط الآن) — Hub متطور مستقل يخدم كل وحدات الأعمال.
2. **إكمال واجهات Clinic Integration** (S10) — `Join.razor`, `PrescriptionComposer`, `DoctorNotes`, `RequiredDataEntry`.
3. **تحسين Commerce B2B** (S11) — البونص UI، إدارة الحالة، Tiers، Logistics.
4. **إغلاق فجوات PSP Cycle** (S12) — Reports Integration، زر "ابدأ رحلتك".
5. **صيانة Messaging** (S13) — فصل الرسائل الشخصية عن رسائل المنظمات.
6. **تفعيل Ledger** (S14) — ربطه بـ `SupplyOrderService` + UI.
7. **الاستعداد لـ Production Launch** (S15).

---

## 🟢 §1: Sprint Plan Overview (v8.0)

### 1.1 Sprint History (ما تم فعلاً)

| Sprint | الاسم | التاريخ | SP Planned | SP Actual | الحالة | Epic |
|---|---|---|---|---|---|---|
| **S01** | Foundation | 16-22 أكتوبر 2026 | 22 | 11 | ✅ مُغلق | E00 |
| **S02** | Points-PSP Integration | 22-23 أكتوبر 2026 | 36 | 46 | ✅ مُغلق | E01 |
| **S03** | MAUI Stabilization | 24-25 أكتوبر 2026 | 27 | 27 | ✅ مُغلق | E01 |
| **S04** | PSP Dispensation | 26-28 أكتوبر 2026 | 53 | 58 | ✅ مُغلق | E01 |
| **S05** | Post-Deploy Fixes 1 | 28-31 أكتوبر 2026 | 24 | 24 | ✅ مُغلق | E01 |
| **S06** | Post-Deploy Fixes 2 | 1-9 نوفمبر 2026 | 34 | 29 | 🟡 مُغلق جزئياً (5 SP → S08) | E01 |
| **S07** | B2B Commerce MVP | 6-11 نوفمبر 2026 | ~50 | ~55 | 🟡 مُغلق جزئياً (تحسينات → S11) | E03 |
| **S08** | PSP Cycle + NH Part 1 | 25-30 نوفمبر 2026 | 103 | 78 | 🟡 مفتوح (25 SP متبقٍ) | E01, E02 |

### 1.2 Sprint Current (النشط الآن)

| Sprint | الاسم | التاريخ | الحالة | Epic |
|---|---|---|---|---|
| **S09** | **Notification Hub — Phase 2** | **25 نوفمبر — 10 ديسمبر 2026** | 🟢 **نشط** | E02 |

### 1.3 Sprint Future (المخطط القريب والمتوسط)

| Sprint | الاسم | التاريخ | الحالة | Epic |
|---|---|---|---|---|
| **S10** | Clinic Integration — UI Completion | 11-20 ديسمبر 2026 | ⏳ مخطط | E04 |
| **S11** | Commerce Enhancements | 21-31 ديسمبر 2026 | ⏳ مخطط | E03 |
| **S12** | PSP Cycle — Final Polish | 1-10 يناير 2027 | ⏳ مخطط | E01 |
| **S13** | Messaging Maintenance | 11-20 يناير 2027 | ⏳ مخطط | E05 |
| **S14** | Ledger Activation | 21-31 يناير 2027 | ⏳ مخطط | E03 |
| **S15** | Production Launch Prep | فبراير 2027 | ⏳ مخطط | E06 |

### 1.4 Epic Map

| Epic | الاسم | Sprint(s) | الحالة |
|---|---|---|---|
| **E00** | Foundation | S01 | ✅ |
| **E01** | PSP Cycle | S02, S03, S04, S05, S06, S08, S12 | 🟡 (S08, S12 متبقيان) |
| **E02** | Notification Hub | S08 (Part 1), S09 (Part 2) | 🟢 (S09 نشط) |
| **E03** | Commerce B2B | S07, S11, S14 | 🟡 (S11, S14 متبقيان) |
| **E04** | Clinic Integration | S10 | ⏳ مخطط |
| **E05** | Messaging | S13 | ⏳ مخطط |
| **E06** | Production Launch | S15 | ⏳ مخطط |

---

## 🟢 §2: Sprint History — التفاصيل الموجزة

> **ملاحظة:** التفاصيل الكاملة لكل مهمة مكتملة موجودة في `00C-CHANGELOG.md`. هنا ملخص فقط.

### S01 — Foundation (16-22 أكتوبر 2026) — ✅ مُغلق
- **الهدف:** بناء الأساس (Identity, Menus, Translation, Caching).
- **SP:** 22 مخطط / 11 فعلي.
- **المتبقي:** 11 SP (مؤجلة إلى ما بعد MVP — غير حرجة).

### S02 — Points-PSP Integration (22-23 أكتوبر 2026) — ✅ مُغلق
- **الهدف:** ربط نظام النقاط بـ PSP.
- **SP:** 36 مخطط / 46 فعلي.
- **المتبقي:** 11 SP (P5.1-P5.3 → S12).

### S03 — MAUI Stabilization (24-25 أكتوبر 2026) — ✅ مُغلق
- **الهدف:** استقرار تطبيق MAUI.
- **SP:** 27 مخطط / 27 فعلي.

### S04 — PSP Dispensation (26-28 أكتوبر 2026) — ✅ مُغلق
- **الهدف:** دورة الصرف الكاملة.
- **SP:** 53 مخطط / 58 فعلي.

### S05 — Post-Deploy Fixes 1 (28-31 أكتوبر 2026) — ✅ مُغلق
- **الهدف:** إصلاحات ما بعد النشر.
- **SP:** 24 مخطط / 24 فعلي.

### S06 — Post-Deploy Fixes 2 (1-9 نوفمبر 2026) — 🟡 مُغلق جزئياً
- **الهدف:** إصلاحات إضافية.
- **SP:** 34 مخطط / 29 فعلي.
- **المتبقي:** 5 SP (S3.6-H3/H4/H5 → S08).

### S07 — B2B Commerce MVP (6-11 نوفمبر 2026) — 🟡 مُغلق جزئياً
- **الهدف:** B2B Commerce MVP (Backend + UI).
- **SP:** ~50 مخطط / ~55 فعلي.
- **المتبقي:** تحسينات (البونص UI، إدارة الحالة) → S11.

### S08 — PSP Cycle + NH Part 1 (25-30 نوفمبر 2026) — 🟡 مفتوح
- **الهدف:** إغلاق فجوات PSP Cycle + Notification Hub (المرحلة الأولى).
- **SP:** 103 مخطط / 78 فعلي.
- **المتبقي:** 25 SP (S6-P1-3, P2, P3, BLOCK-1 → S12؛ NH-1→NH-20 → S09).

---

## 🟢 §3: Sprint Current — S09 (Notification Hub — Phase 2)

### 3.1 الهدف

بناء **Notification Hub متطور مستقل** يخدم كل وحدات الأعمال (نقاط، وصفات، تجارة، PSP) — كخدمة قابلة لإعادة الاستخدام (Reusable Service).

### 3.2 المهام (S09)

| # | المهمة | SP | الحالة | المرجع |
|---|---|---|---|---|
| **S09-P1-01** | Notification Hub — Core (Entities + Migrations) | 8 | ⏳ | `08-NOTIFICATIONS` |
| **S09-P1-02** | Notification Hub — Services (Orchestrator + TargetResolver) | 8 | ⏳ | `08-NOTIFICATIONS` |
| **S09-P1-03** | Notification Hub — API (Controllers + Endpoints) | 5 | ⏳ | `09-API-GUIDE §6` |
| **S09-P1-04** | Notification Hub — UI (Hub Page + Filters) | 8 | ⏳ | `10-UI-STYLE-GUIDE` |
| **S09-P1-05** | Notification Hub — ربط بالنقاط (PointsEarned/PointsRedeemed) | 3 | ⏳ | `01-ARCHITECTURE §6ب (IC-05, IC-06)` |
| **S09-P2-01** | Notification Hub — ربط بالوصفات (PrescriptionCreated) | 3 | ⏳ | `05-PSP §v2` |
| **S09-P2-02** | Notification Hub — ربط بالتجارة (SupplyOrderStatusChanged) | 3 | ⏳ | `23-COMMERCE-ENGINE §3.1` |
| **S09-P2-03** | Notification Hub — ربط بـ PSP (DispensationCompleted) | 2 | ⏳ | `01-ARCHITECTURE §6ب (IC-01, IC-02)` |
| **S09-P3-01** | Notification Hub — Push (FCM + Web Push) | 5 | ⏳ | `08-NOTIFICATIONS` |
| **S09-P3-02** | Notification Hub — Preferences (User Settings) | 3 | ⏳ | `08-NOTIFICATIONS` |
| **المجموع** | | **48** | | |

### 3.3 المخرجات المتوقعة (Deliverables)

- [ ] كيانات `NotificationHub`, `NotificationTemplate`, `NotificationPreference`, `NotificationTargetRule`.
- [ ] Migrations مطبقة.
- [ ] `INotificationOrchestrator` + `ITargetResolverService`.
- [ ] `NotificationHubController` (Endpoints: Send, Get, MarkRead, Preferences).
- [ ] `NotificationHub.razor` (UI مع Filters + Preferences).
- [ ] ربط بـ 4 أحداث (PointsEarned, PointsRedeemed, PrescriptionCreated, SupplyOrderStatusChanged, DispensationCompleted).
- [ ] FCM + Web Push integration.
- [ ] اختبارات E2E.

### 3.4 الاعتمادات (Dependencies)

- `08-NOTIFICATIONS` (المرحلة الأولى — NH-1→NH-20 مكتملة في S08).
- `06-POINTS` (IC-05, IC-06).
- `05-PSP` (IC-01, IC-02).
- `23-COMMERCE-ENGINE` (IC-10, IC-11).

---

## 🟠 §4: Sprint Future — S10 (Clinic Integration — UI Completion)

### 4.1 الهدف

إكمال الواجهات الناقصة لـ Clinic Integration (`Join.razor`, `PrescriptionComposer`, `DoctorNotes`, `RequiredDataEntry`).

### 4.2 المهام (S10)

| # | المهمة | SP | الحالة | GAP |
|---|---|---|---|---|
| **S10-P1-01** | تعديل `Join.razor` لدعم كود الإحالة + استدعاء `enroll` | 5 | ❌ 0% | GAP-02 |
| **S10-P1-02** | إنشاء `PrescriptionComposerView.razor` | 8 | ❌ 0% | GAP-12, GAP-14 |
| **S10-P1-03** | ربط `PrescriptionComposerView` بـ `PrescriptionController` | 3 | ❌ 0% | GAP-12 |
| **S10-P2-01** | إنشاء `DoctorNotesView.razor` + `DoctorNoteController` | 5 | ❌ 0% | GAP-03 |
| **S10-P2-02** | إنشاء `PSPDataEntryResult` + `PatientRequiredDataEntryPage` | 8 | ❌ 0% | GAP-13, GAP-15 |
| **S10-P3-01** | مفاتيح الترجمة (`CLINIC_REFERRAL.*`) | 2 | ❌ 0% | — |
| **S10-P3-02** | اختبار E2E (Clinic Referral + Prescription) | 5 | ❌ 0% | — |
| **المجموع** | | **36** | | |

### 4.3 الاعتمادات

- `ClinicReferralController` (4 Endpoints) — ✅ موجود.
- `PrescriptionController` (8 Endpoints) — ✅ موجود.
- `[#29]` (ملخص جلسة تطوير الوصفات).

---

## 🟠 §5: Sprint Future — S11 (Commerce Enhancements)

### 5.1 الهدف

تحسين Commerce B2B (البونص UI، إدارة الحالة، Tiers، Logistics).

### 5.2 المهام (S11)

| # | المهمة | SP | GAP |
|---|---|---|---|
| **S11-P1-01** | إصلاح نظام البونص في الواجهة | 5 | GAP-04 |
| **S11-P1-02** | إصلاح إدارة حالة الطلب من الشركة | 5 | GAP-05 |
| **S11-P2-01** | Campaigns Auto-Apply في `SupplyOrderService` | 3 | GAP-06 |
| **S11-P2-02** | Tiers Page (UI + API) | 5 | GAP-07 |
| **S11-P2-03** | Logistics Page (UI + API) | 5 | GAP-08 |
| **S11-P3-01** | CompanyStockController (4 Endpoints) | 3 | GAP-09 |
| **S11-P3-02** | CompanyAgreementsController (6 Endpoints) | 3 | GAP-10 |
| **المجموع** | | **29** | |

---

## 🟡 §6: Sprint Future — S12–S15

### S12 — PSP Cycle — Final Polish (1-10 يناير 2027)

| # | المهمة | SP |
|---|---|---|
| S12-P1-01 | S3.6-H5: تعديل زر "ابدأ رحلتك" | 3 |
| S12-P1-02 | S6-P2-1: `GetAdherenceHandler` — إضافة Points | 2 |
| S12-P1-03 | S6-P2-2: `GetPharmacyFinancialHandler` — فصل PointsDiscount | 3 |
| S12-P1-04 | S6-P2-3: `GetProgramLifecycleHandler` — إضافة Points | 2 |
| S12-P2-01 | S6-P3-1: تظبيط تاريخ الأوردر في "طلباتي" | 2 |
| S12-P2-02 | S6-P3-2: صفحة "دعواتي" من عند الدكتور | 3 |
| S12-P2-03 | BLOCK-1: حل 403 في `patient-details` | 2 |
| **المجموع** | | **17** |

### S13 — Messaging Maintenance (11-20 يناير 2027)

| # | المهمة | SP | GAP |
|---|---|---|---|
| S13-P1-01 | فصل الرسائل الشخصية عن رسائل المنظمات | 8 | GAP-11 |
| S13-P1-02 | إصلاح واجهة MessagingHubPage | 5 | GAP-11 |
| S13-P2-01 | اختبار E2E للرسائل | 3 | GAP-11 |
| **المجموع** | | **16** | |

### S14 — Ledger Activation (21-31 يناير 2027)

| # | المهمة | SP | GAP |
|---|---|---|---|
| S14-P1-01 | ربط `LedgerService` بـ `SupplyOrderService` (Confirm/Deliver/Cancel) | 5 | GAP-01 |
| S14-P1-02 | Ledger UI (Pharmacy + Company) | 8 | GAP-01 |
| S14-P2-01 | اختبار E2E للـ Ledger | 3 | GAP-01 |
| **المجموع** | | **16** | |

### S15 — Production Launch Prep (فبراير 2027)

| # | المهمة | SP |
|---|---|---|
| S15-P1-01 | Security Audit | 8 |
| S15-P1-02 | Performance Optimization | 8 |
| S15-P1-03 | Load Testing (1000+ users) | 5 |
| S15-P2-01 | Documentation + User Manuals | 5 |
| S15-P2-02 | Training Material | 3 |
| **المجموع** | | **29** |

---

## 🔴 §7: Gaps Register (الفجوات الموثقة)

| # | الفجوة | الحالة | Sprint | الأولوية | المرجع |
|---|---|---|---|---|---|
| **GAP-01** | `Ledger` غير مفعّل في الواجهة | 🟡 موجود، غير مفعّل | S14 | 🟠 | `23-COMMERCE-ENGINE §0.6` |
| **GAP-02** | `Join.razor` لا يكمل الاشتراك | ❌ 0% | S10 | 🔴 | `[#29]` |
| **GAP-03** | `DoctorNoteController` غير موجود | ❌ | S10 | 🟠 | `[#29]` |
| **GAP-04** | نظام البونص — إصلاحات UI | 🟡 | S11 | 🟠 | `23-COMMERCE-ENGINE §13` |
| **GAP-05** | إدارة حالة الطلب من الشركة — مراجعة | 🟡 | S11 | 🟠 | `23-COMMERCE-ENGINE §3.1` |
| **GAP-06** | Campaigns Auto-Apply غير مطبق | 🟡 CRUD فقط | S11 | 🟡 | `23-COMMERCE-ENGINE §0.6` |
| **GAP-07** | Tiers Page غير موجودة | ❌ | S11 | 🟡 | `23-COMMERCE-ENGINE §6.3` |
| **GAP-08** | Logistics Page غير موجودة | ❌ | S11 | 🟡 | `23-COMMERCE-ENGINE §6.3` |
| **GAP-09** | CompanyStockController غير موجود | ❌ | S11 | 🟡 | `23-COMMERCE-ENGINE §4.8` |
| **GAP-10** | CompanyAgreementsController غير موجود | ❌ | S11 | 🟡 | `23-COMMERCE-ENGINE §4.9` |
| **GAP-11** | Messaging — خلط الرسائل الشخصية/المنظمات | 🟡 | S13 | 🟠 | `09-API-GUIDE §5` |
| **GAP-12** | `PrescriptionComposerView` غير موجود | ❌ 0% | S10 | 🔴 | `[#29]` |
| **GAP-13** | `PSPDataEntryResult` غير موجود | ❌ | S10 | 🟠 | `[#29]` |
| **GAP-14** | `WritePrescriptionPage` غير موجود | ❌ 0% | S10 | 🔴 | `05-PSP §v2` |
| **GAP-15** | `PatientRequiredDataEntryPage` غير موجود | ❌ 0% | S10 | 🟠 | `05-PSP §v2` |

---

## 📊 §8: Technical Debt Register

### 🔴 Critical Debt

| # | الدين | الموقع | المرجع |
|---|---|---|---|
| **TD-C1** | انتهاك معماري: `@page` في `Shared.UI` | `PharmacyDetailPage.razor` | `01-ARCHITECTURE §2.8` |
| **TD-C2** | خلط Namespaces | `PharmacySearchPage.razor` | `01-ARCHITECTURE §2.8` |
| **TD-C3** | 404 في `/admin/pending-licenses` | `SystemManagment` vs `SystemManagement` | — |
| **TD-C4** | `DispenseController` كله `[AllowAnonymous]` (14 Endpoint) | `Api.Web/Controllers/Pharmacy/` | `09-API-GUIDE §17` |
| **TD-C5** | `Ledger` موجود كـ API لكن غير مفعّل | `23-COMMERCE-ENGINE §0.6` | GAP-01 |

### 🟠 Medium Debt

| # | الدين | الموقع | المرجع |
|---|---|---|---|
| **TD-M1** | زر البحث في `Dashboard` غير مربوط | `Pages/Dashboard.razor` | — |
| **TD-M2** | ازدواجية `FirebaseNotificationService` | `Api.Web` + `Infrastructure` | — |
| **TD-M3** | ازدواجية `IPSPPointSchemeService` | `Application` + `Web` | — |
| **TD-M4** | ازدواجية `MedicineGuideFlow` | `Mobile/Features/PSP/Patient/Views/` | — |
| **TD-M5** | `TD-001`: الطلبات المكررة (Double API Calls) | `Rubikcare.Web` + `Api.Web` | `00C` |
| **TD-M6** | `Messaging` — خلط الرسائل الشخصية/المنظمات | `MessagingHubPage` | GAP-11 |

### 🟢 Low Debt

| # | الدين | الموقع |
|---|---|---|
| **TD-L1** | `IMobileNavigationService` (اسم مضلل) | `Shared.UI/Services/` |
| **TD-L2** | `HttpUtility` في Blazor WASM | `PharmacySearch.razor` |
| **TD-L3** | `alert()` بدلاً من Toast | `PspSearch.razor` |
| **TD-L4** | `SystemMenuMenuID` Shadow Property | `MenuAssignments` |
| **TD-L5** | `RoleId` في `MenuVisibilityRule` | بعد Migration |

---

## 📅 §9: Sprint Velocity (آخر 4 Sprints)

| Sprint | Planned | Actual | Ratio | الحالة |
|---|---|---|---|---|
| S04 (PSP Dispensation) | 53 SP | 58 SP | 109% ✅ | مكتمل |
| S05 (Post-Deploy Fixes 1) | 24 SP | 24 SP | 100% ✅ | مكتمل |
| S06 (Post-Deploy Fixes 2) | 34 SP | 29 SP | 85% 🟡 | مكتمل جزئياً |
| S07 (B2B Commerce) | ~50 SP | ~55 SP | 110% ✅ | مكتمل جزئياً |
| S08 (PSP Cycle + NH Part 1) | 103 SP | 78 SP | 76% 🟡 | مفتوح |

**المعدل الفعلي للفريق:** ~45-50 SP/Sprint (بعد استقرار S02–S04 وإنجاز NH Part 1).

---

## 🎯 §10: الأولويات الفورية (Next 3 Days — 25-27 نوفمبر 2026)

| اليوم | المهمة | المرجع |
|---|---|---|
| **25 نوفمبر** | S09-P1-01: Notification Hub — Core (Entities) | `08-NOTIFICATIONS` |
| **26 نوفمبر** | S09-P1-02: Notification Hub — Services | `08-NOTIFICATIONS` |
| **27 نوفمبر** | S09-P1-03: Notification Hub — API | `09-API-GUIDE §6` |

**قاعدة صارمة:** لا نبدأ S10 قبل إغلاق S09 (Notification Hub — Phase 2).

---

## 🔗 §11: الروابط المرجعية

### الوثائق الشقيقة (يجب قراءتها)

- `00C-CHANGELOG.md` — كل ما تم إنجازه (S01 → S08).
- `00D-ROADMAP.md` — الجدول الزمني الكلي + المعالم.
- `00-MASTER-INDEX.md` — الفهرس الشامل + الحالة الفنية لكل Module.

### الوثائق المرتبطة بالمهام الحالية

- `08-NOTIFICATIONS.md` — مرجع Notification Hub (S09).
- `05-PSP.md` — مرجع PSP Cycle (S10, S12).
- `06-POINTS.md` — مرجع Points (S09, S12).
- `23-COMMERCE-ENGINE.md` — مرجع Commerce (S11, S14).
- `09-API-GUIDE.md` — مرجع API (كل Sprint).
- `24-pharmacy-B2C-Orders.md` — مرجع B2C (S10).

### وثائق المعمارية

- `01-ARCHITECTURE.md` — الهيكل المعماري (يُقرأ قبل أي كود).
- `11-PAGE-CREATION.md` — إنشاء الصفحات.
- `04-TRANSLATION.md` — الترجمة.

---

## 📝 §12: سجل تحديثات هذه الوثيقة

| الإصدار | التاريخ | التغييرات |
|---|---|---|
| 6.4 | 5 نوفمبر 2026 | نجاح DashboardFlowPage + OnboardingPage + MedicineGuidePage |
| 7.0 | 11 نوفمبر 2026 | إعادة كتابة كاملة (Live State Edition) — فصل التاريخ إلى `00C` + إعادة ترتيب Sprint 4-9 حسب الواقع |
| 7.2 | 23 نوفمبر 2026 | إضافة §3.3 (Notification Hub — 20 مهمة، 50 SP) + تحديث ملخص Sprint 6 |
| **8.0** | **25 نوفمبر 2026** | **إعادة كتابة كاملة (Realigned Edition) وفق Sprint Plan v8.0 (S01–S15): إعادة ترقيم Sprints (S01–S08 History، S09 Current، S10–S15 Future)؛ إضافة Epic Map (E00–E06)؛ إضافة Gaps Register (GAP-01→GAP-15)؛ تحديث Technical Debt Register (TD-C4, TD-C5, TD-M6 جديدة)؛ تحديث Velocity Chart (S04–S08)؛ تحديث الأولويات الفورية (S09)؛ إضافة قاعدة "الوثيقة تطابق الكود" (القاعدة 2)؛ إضافة قاعدة "كل فجوة موثقة" (القاعدة 3).** |

---

## 📋 قواعد المشروع الذهبية (Golden Rules)

- ❌ لا تعدل Migration قديم — Migration جديد فقط.
- ✅ استخدم `IGenericService<T>` بدل `DbContext` مباشرة.
- ✅ `N''` prefix للعربية في SQL.
- ✅ `MERGE` بدل `INSERT/UPDATE`.
- ✅ استخدم `T("KEY")` للترجمة.
- ❌ لا `Set-Content` لملفات `.razor`/`.cshtml` (تتلف العربية).
- ✅ كل رد يبدأ بـ `[#رقم]` (ترقيم تصاعدي).
- ✅ خطوة واحدة لكل رد.
- ✅ مناقشة قبل التنفيذ.
- ✅ **الوثيقة تطابق الكود** (القاعدة الجديدة).
- ✅ **كل فجوة موثقة** (القاعدة الجديدة).

---

**نهاية 00B — PROJECT-BACKLOG (v8.0) — 25 نوفمبر 2026** 
```
