# 00B — PROJECT-BACKLOG

الاسم: PROJECT-BACKLOG — سجل التطوير الحي  
الإصدار: 6.1 (Production Blocker Resolved Edition)  
آخر تحديث: 29 أكتوبر 2026  
Sprint الحالي: Sprint 3.5 (In Progress)  
الحالة: 🟢 حي — يُحدَّث كل جلسة  
الجمهور: فريق التطوير + AI Assistants  

---

## 📌 مقدمة

هذه الوثيقة هي المصدر الوحيد للحقيقة لكل ما يتعلق بمهام التطوير في RubikCare.

تدمج:

- 18-RELEASE-ROADMAP-DASHBOARD (مهام P1-P4).
- 23-TECHNICAL-DEBT (الديون التقنية).
- Changelog (سجل التغييرات).
- TASK-LOG (سجل الأحداث — Sprint Events).

الاستخدام:

- قبل كل جلسة: تُشارَك مع AI ليعرف السياق.
- بعد كل مهمة: تُحدَّث الحالة.

---

## 🎯 الرؤية الحالية

RubikCare في مرحلة Pre-Launch. الهدف الحالي:

- إكمال Backend + Frontend الأساسي.
- إصدار Production مستقر.
- بناء الأساس للسجلات الطبية (PHR) والتكامل الخارجي.

---

## 📊 Sprint 1 — Points-PSP Integration (22-23 أكتوبر 2026) — ✅ مكتمل

الهدف: إغلاق الفجوة بين نظام النقاط و PSP (9 عقود تكامل — IC-01 → IC-09).

### ✅ Done (12 مهمة)

| # | المهمة | SP | الحالة |
|---|---|---|---|
| P1 | توثيق Integration Contracts | 2 | ✅ |
| P2.1 | Migration: REDEEM_DISPENSATION (ID=9) | 1 | ✅ |
| P2.2 | إعادة كتابة GenerateNextToken (ذرية) | 5 | ✅ |
| P2.3 | إصلاح ConfirmDispense (احترام الخصم) | 3 | ✅ |
| P2.4 | إصلاح PspController.patientdetails | 2 | ✅ |
| P2.5 | إضافة MonetaryValuePerPoint إلى DTO | 1 | ✅ |
| P3.1 | PspDetailComponent.razor (Dialog + Success) | 3 | ✅ |
| P3.2 | CSS: _shared/dialogs.css | 2 | ✅ |
| P3.3 | إصلاح PspStatsBar | 1 | ✅ |
| P4.1 | Idempotency في TryAwardPointsViaOrchestrator | 2 | ✅ |
| P4.2 | Idempotency في AcknowledgeReminderHandler | 1 | ✅ |
| P6.1 | إشعار عند كسب/خصم نقاط | 2 | ✅ |

### ✅ Done (إضافي)

| # | المهمة | SP |
|---|---|---|
| D1 | إصلاح IRefreshService في PWA | 1 |
| D2 | إصلاح فقدان ربط PSP بعد Migration (22 حالة) | 3 |
| D3 | إصلاح حساب النقاط (PatientProfileID بدلاً من CaseID) | 2 |
| D4 | إصلاح عرض الخصم للصيدلي (PointsDiscount) | 2 |
| D5 | إصلاح بطاقة "لا يوجد توكن" + عرض التوكن الجديد | 1 |
| D6 | إصلاح تجاوب BarcodeScanner (CSS) | 1 |

المجموع: 36 SP (Sprint 1) + 10 SP (إضافي) = 46 SP

### ⏳ Pending (المتبقي)

| # | المهمة | SP | الأولوية |
|---|---|---|---|
| P5.1 | GetAdherenceHandler — إضافة Points | 2 | 🟠 |
| P5.2 | GetPharmacyFinancialHandler — فصل PointsDiscount | 3 | 🟠 |
| P5.3 | GetProgramLifecycleHandler — إضافة Points | 2 | 🟠 |
| P7.1 | الترجمة (SQL MERGE) | 1 | 🟡 |
| P7.2 | اختبار E2E كامل | 3 | 🟠 |

المجموع المتبقي: 11 SP

---

## 📊 Sprint 2 — MAUI Stabilization & Integration (24-25 أكتوبر 2026) — ✅ مكتمل

الهدف: حل مشاكل MAUI (JavaProxyThrowable، QueryProperty، InvalidCastException، patientId=0).

### ✅ Done (13 مهمة)

| # | المهمة | SP |
|---|---|---|
| M1 | إنشاء PatientPointsFlow.xaml + .xaml.cs | 2 |
| M2 | تعديل PspDetailComponent.razor — استخدام Action | 2 |
| M3 | تعديل PatientPointsDashboard.razor — استخدام Bridge | 2 |
| M4 | إصلاح InvalidCastException (userId.ToString()) | 1 |
| M5 | ربط كل دوال PspDetailNavigationBridge | 3 |
| M6 | استخدام Dispatcher.DispatchAsync | 1 |
| M7 | إصلاح MedicationScheduleFlow.OnNavigateToPsp | 2 |
| M8 | إصلاح MedicationSchedulePage.OnRefillRowClick | 1 |
| M9 | إضافة CaseID إلى DispensationScheduleDto | 2 |
| M10 | إصلاح PspSchedulePageFlow | 2 |
| M11 | تحديث 13-maui-development-guide.md | 3 |
| M12 | إصلاح PspController.patientDetails | 1 |
| M13 | إصلاح DispensationScheduleDto | 1 |

### ✅ Done (إضافي)

| # | المهمة | SP |
|---|---|---|
| M14 | توحيد PspDetailFlow | 2 |
| M15 | حذف ScanTokenPage.razor (القديم) | 1 |
| M16 | إصلاح PharmacyGateway | 1 |

المجموع الكلي: 27 SP

---

## 🎯 Sprint 0 — Foundation (16-22 أكتوبر 2026) — ⏳ In Progress

### ✅ Done

| # | المهمة | SP |
|---|---|---|
| 1 | تحديث 09-API-Guide v2.0 | 5 |
| 2 | كتابة 03-MENUS v4.0 | 5 |
| 3 | تجربة السكريبت التوليدي | 1 |

### ⏳ Pending

| # | المهمة | SP |
|---|---|---|
| 5 | تحسين السكريبت v2 | 2 |
| 6 | كتابة 19-PERMISSIONS | 5 |
| 7-11 | 5 Migrations | 7 |

المجموع المتبقي: 14 SP

---

## 📊 Sprint 3 — PSP Dispensation & Medicine Guide (26-28 أكتوبر 2026) — ✅ مكتمل

الهدف: إكمال دورة الصرف بناءً على الـ Phases + صفحة Medicine Guide منفصلة + تحسين الإشعارات.

### ✅ Done (24 مهمة)

| # | المهمة | SP | ملاحظات |
|---|---|---|---|
| S3-Done-1 | تنظيف بيئة النشر (حذف المواقع القديمة، أرشفة _Archive) | 2 | — |
| S3-Done-2 | إعداد SharedStorage وأذوناته (Web + API) | 3 | تم حل مشكلة رفع الملفات. |
| S3-Done-3 | ترقية قاعدة البيانات (UAT → Production) | 5 | تم تطبيق Migration RenamePatientIdToCaseId مع تنظيف السجلات اليتيمة. |
| S3-Done-4 | مزامنة Seed Data الناقصة (Menus, ActivityTypes, ...) | 3 | تم حل مشكلة القوائم المفقودة. |
| S3-Done-5 | إنشاء نظام سكريبتات النشر (Web, API, PWA) + Rollback | 5 | 12 سكريبت جاهز في _Docs\Scripts. |
| S3-Done-6 | توثيق 15-DEPLOYMENT.md (v3.0) مع قسم قاعدة البيانات | 2 | — |
| S3-Done-7 | Backend: Phases support في PSPEnrollmentService (RefillIntervalDays + StartAfterMonths) | 3 | توليد التوكنات بناءً على المراحل. |
| S3-Done-8 | Backend: GenerateSchedulesFromERxAsync — استخدام PhaseMedication بدل PlanMedication | 3 | Daily Doses صحيحة (1-2 يومياً بدل 7). |
| S3-Done-9 | Backend: GetDispensationSchedulesAsync — إزالة Include(Plan) + planMap + PENDING→PROJECTED | 2 | 11 صف بدل 9 + PROJECTED ظاهر. |
| S3-Done-10 | Backend: Lazy Activation — ActivateEligiblePendingTokensAsync | 2 | PENDING → GENERATED تلقائياً. |
| S3-Done-11 | Backend: ConfirmDispense — DispenseDate + ActivateNextPendingTokenAsync | 2 | التوكن التالي يتفعل بعد RefillIntervalDays. |
| S3-Done-12 | Backend: ReminderProcessingService — التحقق من ScheduleStatus (تجنب الإشعار المتكرر) | 2 | مش بيبعت بعد "تناول". |
| S3-Done-13 | Backend: GetMedicationDetailsAsync — endpoint جديد (CaseID) + Fallback للبرامج بدون Phases | 3 | Medicine Guide page data. |
| S3-Done-14 | Web: PSPStep2_DispensationPlans — إعادة هيكلة Plan Settings + Phase Medication | 3 | حقول جديدة (RefillIntervalDays, QuantityPerDispense, UnitsPerPack) + Auto-calc. |
| S3-Done-15 | Web: CanRequestEarly + EarlyRequestDays + "Coming Soon" لـ Single/AllowRefill | 2 | — |
| S3-Done-16 | Mobile: MedicineGuidePage (NEW) + MedicineGuideFlow + Route | 3 | صفحة منفصلة /medicine-guide. |
| S3-Done-17 | Mobile: PspSchedulePage (NEW component) — برامج + توليد + FirstDoseTime لكل برنامج | 3 | بدل duplicate of RefillDatesTab. |
| S3-Done-18 | Mobile: AddMedicationPage validation + Toast + PspDetailComponent زرار "دليل الأدوية" | 2 | — |
| S3-Done-19 | Mobile: Navigation Bridges (GoBack, GoToTests, GoToMedicineGuide, CopyToken, ShareToken) | 2 | — |
| S3-Done-20 | Mobile: PatientTestsFlow + Route + PatientSession update قبل GoToTests | 2 | — |
| S3-Done-21 | DB: Migration AddRefillFieldsToPhaseMedications | 2 | RefillIntervalDays, QuantityPerDispense, UnitsPerPack. |
| S3-Done-22 | DB: MERGE Plans (TokenValidityHours=720) + MERGE Patient 1081 (15 dispensations) | 2 | — |
| S3-Done-23 | DB: MERGE Resources (translations — Medicine Guide, Dispense Status, Phase fallback) | 1 | — |
| S3-Done-24 | Build: PublishTrimmed=false + AndroidEnableProfiledAot=false (IL Trimmer fix) | 1 | — |

المجموع المنجز: 58 SP.

---

## 🗓️ Sprint 3.5 — Post-Deployment Fixes & Polish (28-31 أكتوبر 2026) — ⏳ In Progress

الهدف: إغلاق المهام الناتجة عن اختبار UAT + تحسينات UX المطلوبة.

### 🔴 High Priority

| # | المهمة | SP | ملاحظات |
|---|---|---|---|
| S3.5-H1 | إصلاح Crash في صفحة البرامج — وضع المندوب (Rep) | 3 | الصفحة تعمل Crash عند فتحها في وضع Rep — يحتاج فحص Stack Trace + إصلاح NullReference. |
| S3.5-H2 | إظهار اسم الصيدلية/العيادة في "شبكتي" أو "دعواتي" بعد قبول الدعوة | 5 | حالياً المندوب لا يرى اسم الجهة بعد اشتراكها — يحتاج ربط الدعوة بـ PSPParticipation + إرجاع OrganizationName. |
| S3.5-H3 | إظهار اسم برنامج الدعم بجانب كل دواء في تاب "مواعيد الأدوية" | 2 | حالياً يعرض MedicationName فقط — يجب إضافة ProgramName. |
| S3.5-H4 | الضغط على الدواء في "مواعيد الأدوية" يفتح MedicineGuidePage | 2 | Navigation من MedicationSchedulePage → /medicine-guide?medicationId=X. |
| S3.5-H5 | إصلاح الإشعارات المتكررة — بعد "تناول" مفيش إشعار، لو ما ضغطش تجيله مرتين فقط | 3 | بعد إصلاح ReminderProcessingService — يحتاج اختبار + ضبط MaxRepeats=2. |
| S3.5-H6 | ✅ إصلاح Blocker: بطء Pharma Dashboard (42s) + اختفاء الأشكال البيانية | 8 | **مكتمل (29 أكتوبر):** السبب الجذري كان `ApiSettings.BaseUrl` في `appsettings.json` يشير إلى منفذ تطوير مغلق (5235). تم تصحيحه إلى `https://api.rubikcare.com` + ضبط `ASPNETCORE_ENVIRONMENT` إلى `Production`. الصفحات تعمل الآن بسرعة وبدون أخطاء. |

المجموع (High): 23 SP (15 SP منها مكتملة)

### 🟠 Medium Priority

| # | المهمة | SP | ملاحظات |
|---|---|---|---|
| S3.5-M1 | تحسين CSS في MedicineGuidePage — Program Header + Hero + Empty State | 2 | تم جزئياً — يحتاج مراجعة نهائية. |
| S3.5-M2 | إضافة قسم "حالة الصرف" في MedicineGuidePage (TotalDispensed + LastDispensedDate) | 2 | البيانات موجودة في DTO — يحتاج عرض UI. |
| S3.5-M3 | عرض اسم البرنامج + Phase في كارت الدواء (MedicineGuidePage) | 1 | تم — يحتاج تأكيد. |
| S3.5-M4 | تحسين رسالة "No Medications" للبرامج بدون Phases | 1 | "البرنامج مش متوفر — تواصل مع الدعم" بدل "اشترك في برنامج". |

المجموع (Medium): 6 SP

### 🟡 Low Priority

| # | المهمة | SP | ملاحظات |
|---|---|---|---|
| S3.5-L1 | إضافة "Medicine Guide" card في PspDetailComponent | 1 | تم — يحتاج تأكيد. |
| S3.5-L2 | تحسين ترتيب الـ Bridges في PspDetailFlow | 1 | تنظيم الكود فقط. |
| S3.5-L3 | حذف MedicineGuideFlow المكرر إذا وجد | 1 | تنظيف. |

المجموع (Low): 3 SP

المجموع (Sprint 3.5): 32 SP (8 منها مكتملة في 29 أكتوبر)

---

## 📈 Velocity Chart

| Sprint | Planned | Actual | Ratio |
|---|---|---|---|
| Sprint 1 | 36 | 46 | 128% ✅ (مع إضافي) |
| Sprint 2 | 27 | 27 | 100% ✅ |
| Sprint 0 | 22 | 11 | 50% 🟡 |
| Sprint 3 | 53 | 58 | 109% ✅ (مع إضافي) |
| Sprint 3.5 | 24 | 8 (H6) | ⏳ In Progress |

---

## 🎯 Epic Overview

| Epic | User Stories | Done | Completion |
|---|---|---|---|
| 1: Foundation & Tooling | 5 | 2 | ~40% |
| 2: Permissions System | 4 | 0 | 0% |
| 3: PSP System | 15 | 15 | ~100% ✅ |
| 4: Points System | 12 | 12 | ~100% ✅ |
| 5: Reports System | 8 | 7 | ~85% |
| 6: Notifications System | 6 | 6 | ~100% ✅ |
| 7: PWA | 10 | 9 | ~90% |
| 8: Menus & Services | 6 | 0 | 0% |
| 9: Patient Mobile (MAUI) | 3 | 3 | ~100% ✅ |
| 10: Deployment & Release | 4 | 1 | ~25% |
| 11: Personal Health Records (PHR) | 3 (Phases) | 0 | 0% |
| 12: Integration Hub | 3 (Phases) | 0 | 0% |
| 13: Rep (Medical Representative) | 4 (Phases) | 0 | 0% |

---

## 📋 Product Backlog (المستقبلي)

### 🔴 High Priority

| # | User Story | Epic | SP |
|---|---|---|---|
| 20 | كـ مريض، أريد رؤية رصيد النقاط في Mobile | Points | 3 |
| 21 | كـ Admin، أريد إدارة Control Levels | Permissions | 5 |
| 22 | كـ Admin، أريد ربط Users بـ Control Levels | Permissions | 3 |
| 23 | كـ Super Admin، أريد إدارة Permissions | Permissions | 5 |
| 24 | كـ Admin، أريد إدارة اشتراك المنظمة | Subscriptions | 5 |
| 25 | كـ Admin، أريد ربط SubscriptionPlan بـ Menus | Menus | 5 |
| 26 | كـ طبيب/مريضة، أريد أداة Hb Navigator | PHR | 13 |
| 27 | كـ Rep، أريد رؤية اسم الصيدلية/العيادة بعد قبول الدعوة في "شبكتي" | Rep | 5 |

### 🟠 Medium Priority

| # | User Story | Epic | SP |
|---|---|---|---|
| 30 | Menus ديناميكية في PWA | Menus | 5 |
| 31 | Menus ديناميكية في MAUI | Menus | 5 |
| 32 | ربط ServiceProduct بـ MenuItems | Menus | 3 |
| 33 | ApiClient موحّد (Integration Hub) | Integration Hub | 8 |
| 34 | API-Contracts (OpenAPI) | Integration Hub | 5 |
| 35 | كـ Rep، أريد إشعار عند قبول الدعوة من صيدلية/عيادة | Rep | 3 |

### 🟡 Low Priority

| # | User Story | Epic | SP |
|---|---|---|---|
| 40 | إدارة OrganizationGroups | Organization | 5 |
| 41 | Rubika Currency Engine | Rubika | 8 |
| 42 | Deep Linking من QR | Mobile | 3 |

---

## 🏗️ Technical Debt Register

### 🔴 Critical Debt

| # | الدين | الموقع |
|---|---|---|
| 1 | انتهاك معماري: @page في Shared.UI | PharmacyDetailPage.razor |
| 2 | خلط Namespaces | PharmacySearchPage.razor |
| 3 | 404 في /admin/pending-licenses | SystemManagment vs SystemManagement |

### 🟡 Medium Debt

| # | الدين | الموقع |
|---|---|---|
| 4 | زر البحث في Dashboard غير مربوط | Pages/Dashboard.razor |
| 5 | ازدواجية FirebaseNotificationService | Api.Web + Infrastructure |
| 6 | ازدواجية IPSPPointSchemeService | Application + Web |
| 7 | عدم تحديث PROJECT-BACKLOG أسبوعياً | عملية |
| 8 | ✅ **ازدواجية BaseUrl في appsettings (Web)** — تم تصحيح `RubikCareLive` و `RubikCareNew` في 29 أكتوبر. | RubikCareLive و RubikCareNew |
| 9 | ازدواجية MedicineGuideFlow (إذا وجد) | Mobile/Features/PSP/Patient/Views/ |
| 18 | 🟡 **TD-001: الطلبات المكررة (Double API Calls)** — بعض الصفحات (Dashboard, Loyalty, Reports) ترسل نفس طلب الـ API مرتين في نفس الثانية. | `Rubikcare.Web` (Blazor Server) + `Api.Web` (السجلات) |

#### تفاصيل TD-001

**الوصف:** في سجلات `stdout` (أكتوبر 2026)، لوحظ أن بعض الصفحات ترسل نفس طلب الـ API مرتين في نفس الثانية (مثلاً: `GET /api/psp/points/schemes/by-organization/91`). السبب الأرجح: Prerendering + Interactive (تشغيل مزدوج في Blazor)، أو استدعاءات في `OnParametersSetAsync` أو `OnAfterRenderAsync` دون شرط `firstRender`.

**الأثر الحالي:** ضئيل جداً (المنصة لم تبدأ الإنتاج الفعلي بعد، والطلبات صغيرة الحجم 2-100ms).

**الأثر المتوقع عند التوسع:** زيادة الضغط على الـ API عند زيادة عدد المستخدمين + استهلاك غير ضروري للنطاق الترددي + بطء محتمل عند الـ Scaling.

**خطة المعالجة (مؤجلة):**
1. تفعيل `DuplicateRequestDetectionMiddleware` في `Api.Web` (للتسجيل فقط).
2. مراجعة كود الصفحات المتأثرة (استخدام `OnInitializedAsync` بدلاً من `OnParametersSetAsync` حيث أمكن).
3. إضافة `if (firstRender)` في `OnAfterRenderAsync`.
4. إضافة Caching على مستوى الـ API (اختياري).
5. Load Test بعد المعالجة.

**الحالة:** ⏸️ مؤجلة حتى إنهاء UAT وبدء الإنتاج الفعلي ومراقبة الأداء.

**المرجع:** `C:\WebSite\RubikCareLive\logs\stdout_*.log` (أكتوبر 2026).

### 🟢 Low Debt

| # | الدين | الموقع |
|---|---|---|
| 10 | IMobileNavigationService (اسم مضلل) | Shared.UI/Services/ |
| 11 | HttpUtility في Blazor WASM | PharmacySearch.razor |
| 12 | alert() بدلاً من Toast | PspSearch.razor |
| 13 | SystemMenuMenuID Shadow Property | MenuAssignments |
| 14 | RoleId في MenuVisibilityRule | بعد Migration |
| 15 | PspDetailFlow — لا يربط GoHome | MAUI (تم إصلاحه في Sprint 3) |
| 16 | PspSchedulePageFlow — لا يمرر PatientId | MAUI (تم إصلاحه في Sprint 3) |
| 17 | MedicineGuidePage — LoadTranslations (PSP + SHARED.PSP_DETAIL) | تم إصلاحه في Sprint 3.5 |

---

## ✅ Blocker — Pharma Dashboard Performance (تم الحل في 29 أكتوبر 2026)

**الحالة:** ✅ **مغلق** (تم حل المشكلة نهائياً).

**الوصف الأصلي:** صفحات `dashboard/organization` و `reports/*` في واجهة شركات الأدوية (Pharma Company) تستغرق 42 ثانية للتحميل في الإنتاج، بينما تفتح في UAT في أجزاء من الثانية. الصفحات الأخرى (مثل loyalty و psp) تعمل بشكل طبيعي.

**السبب الجذري (تم اكتشافه في 29 أكتوبر):**

- ملف `appsettings.json` في مجلد `RubikCareLive` (بيئة الإنتاج) كان يحتوي على:
  ```json
  "ApiSettings": {
      "BaseUrl": "http://161.97.105.83:5235"
  }
  ```
- المنفذ `5235` هو منفذ التطوير المحلي (Visual Studio)، ولا يوجد خادم يستمع عليه في بيئة الإنتاج.
- الـ Web كان يحاول الاتصال بالـ API، يفشل، يعيد المحاولة، ثم ينتظر انتهاء SignalR Timeout (42 ثانية) قبل أن يظهر الصفحة (بدون أشكال بيانية).
- إعداد `ASPNETCORE_ENVIRONMENT = Production` في `web.config` كان يخفي الأخطاء ويجعل البطء يظهر كـ Timeout كامل.

**الحل المطبق:**

1. تصحيح `BaseUrl` في `appsettings.json` (و`appsettings.Production.json` إن وجد) ليصبح:
   ```json
   "ApiSettings": {
      "BaseUrl": "https://api.rubikcare.com"
   }
   ```
2. إعادة `ASPNETCORE_ENVIRONMENT` إلى `Production` في `web.config` (بعد التحقق من عمل الصفحة).
3. إعادة تشغيل App Pool في IIS.

**النتيجة:**

- ✅ الصفحات تفتح بسرعة (أجزاء من الثانية).
- ✅ الأشكال البيانية والأرقام تظهر بشكل صحيح.
- ✅ لا توجد أخطاء في السجلات.

**الإجراء الوقائي (مستقبلاً):**

- مراجعة `BaseUrl` في كل ملفات `appsettings` قبل أي نشر (كجزء من Master Deployment Checklist).
- إضافة فحص آلي (`grep` لـ `5235` أو `161.97` في `appsettings.json`) في سكريبت النشر.
- استخدام `appsettings.Production.json` فقط في الإنتاج (وليس `appsettings.json`).

---

## 🔴 Sprint 3 — Pending (المتبقي)

### P0 — النشر الآلي (متبقي)

| # | المهمة | SP |
|---|---|---|
| S3-P0.1 | إنشاء Deploy-FromStage.ps1 | 3 |
| S3-P0.2 | إنشاء Promote-ToProduction.ps1 | 2 |
| S3-P0.3 | إنشاء Deploy-All.ps1 | 1 |
| S3-P0.4 | إضافة Health Check Endpoint | 1 |
| S3-P0.5 | تحديث 15-DEPLOYMENT.md | 2 |
| S3-P0.6 | اختبار E2E للنشر الآلي | 2 |

المجموع (P0): 11 SP

### Sprint 3 — المهام الأخرى

| # | المهمة | SP |
|---|---|---|
| 1 | إكمال P5.1 → P5.3 (التقارير) | 7 |
| 2 | اختبار E2E كامل | 3 |
| 3 | كتابة 19-PERMISSIONS | 5 |
| 4 | Migration 4: ControlLevel + Permission | 2 |
| 5 | Migration 5: UserControlLevel + JobTitleControlLevel | 2 |
| 6 | الترجمة المتبقية (P7.1) | 1 |
| 7 | إكمال PspDetailFlow | 1 |
| 8 | إكمال PspSchedulePageFlow | 1 |

المجموع (Sprint 3 كامل): 58 SP (Done) + 11 SP (P0) + 22 SP (Pending) = 91 SP

---

## 🗓️ Sprint 4 — Planned (1-5 نوفمبر 2026) — PHR Phase 1

الهدف: تأسيس البنية التحتية للسجلات الطبية الشخصية (PHR) + تسجيل Hb Navigator.

المرجع: 21-PHR-SYSTEM.md.

| # | المهمة | SP |
|---|---|---|
| PHR-1 | Domain: LabTestType, AssessmentRule, LabTestResult, HealthRecord | 3 |
| PHR-2 | Domain: SharingConsent (النسخة الأساسية) | 1 |
| PHR-3 | Application: HbClinicalRulesEngine + DTOs | 3 |
| PHR-4 | Application: SaveLabResultUseCase + GetLabResultsUseCase | 2 |
| PHR-5 | API: HealthRecordsController | 2 |
| PHR-6 | Shared.UI: HbNavigator.razor | 4 |
| PHR-7 | اختبارات E2E + ربط بـ PSP | 2 |

المجموع: 17 SP

---

## 🗓️ Sprint 5 — Planned (8-12 نوفمبر 2026) — PHR Phase 2

| # | المهمة | SP |
|---|---|---|
| PHR-8 | إضافة LabTestType جديد (Ferritin) | 1 |
| PHR-9 | واجهة ديناميكية (Dynamic Form) | 3 |
| PHR-10 | SharingConsent (Scope: ALL, LAB_RESULTS) | 2 |
| PHR-11 | API: ShareRecordEndpoint + RevokeShareEndpoint | 2 |
| PHR-12 | UI: شاشة "مشاركة السجل" | 3 |
| PHR-13 | اختبارات E2E للمشاركة | 2 |

المجموع: 13 SP

---

## 🗓️ Sprint 6 — Planned (15-19 نوفمبر 2026) — PHR Phase 3

| # | المهمة | SP |
|---|---|---|
| PHR-14 | HealthRecord يدعم PRESCRIPTION, SURGERY, MEASUREMENT | 3 |
| PHR-15 | ربط PHR بـ PSP | 3 |
| PHR-16 | SharingConsent (Scope: JSON، صلاحية زمنية) | 3 |
| PHR-17 | RecordAccessLog + تقارير الوصول | 2 |
| PHR-18 | UI: لوحة "السجل الطبي" (Timeline View) | 4 |
| PHR-19 | اختبارات E2E شاملة + توثيق | 2 |

المجموع: 17 SP

---

## 🗓️ Sprint 7 — Planned (22-26 نوفمبر 2026) — Integration Hub Core

الهدف: تأسيس البنية التحتية لمركز التكامل الخارجي.

المرجع: 22-INTEGRATION-HUB.md.

| # | المهمة | SP |
|---|---|---|
| IH-1 | Domain: IntegrationClient, IntegrationRequest, IntegrationResult | 3 |
| IH-2 | Application: IIntegrationClient Interface + IntegrationOrchestrator | 3 |
| IH-3 | Infrastructure: AuthenticationService (OAuth 2.0 + API Keys) | 3 |
| IH-4 | Infrastructure: RetryPolicy (Polly) + AuditLogger | 2 |
| IH-5 | Infrastructure: RateLimiter + IdempotencyService | 2 |
| IH-6 | API: IntegrationsController | 2 |

المجموع: 15 SP

---

## 🗓️ Sprint 8 — Planned (29 نوفمبر - 3 ديسمبر 2026) — ClinicOS Adapter

| # | المهمة | SP |
|---|---|---|
| IH-7 | ClinicOsAdapter : IIntegrationClient (REST) | 4 |
| IH-8 | ClinicOsDtoMapper | 3 |
| IH-9 | اختبارات E2E: PHR → ClinicOS | 3 |
| IH-10 | توثيق Adapter + OpenAPI Spec | 3 |

المجموع: 13 SP

---

## 🗓️ Sprint 9 — Planned (6-10 ديسمبر 2026) — Webhooks + OpenAPI

| # | المهمة | SP |
|---|---|---|
| IH-11 | Webhook Events (record.created, consent.given, ...) | 4 |
| IH-12 | OpenAPI Spec (Swagger) لكل Adapter | 3 |
| IH-13 | بوابة مطورين (Developer Portal) | 3 |
| IH-14 | اختبارات E2E شاملة | 2 |

المجموع: 12 SP

---

## 📝 Changelog (مختصر)

### v6.1 — 29 أكتوبر 2026

- ✅ **حل Blocker:** إغلاق مشكلة بطء Pharma Dashboard (42s) — السبب: `BaseUrl` خاطئ في `appsettings.json`.
- ✅ إضافة `S3.5-H6` (حل مشكلة Blocker) — 8 SP.
- ✅ تحديث TD #8 (ازدواجية BaseUrl) — تم تصحيحه.
- ✅ إضافة TD-001 (الطلبات المكررة) — دين تقني جديد مؤجل.
- ✅ إضافة الدرس 9 (BaseUrl في Production) والدرس 10 (تأثير Environment على Blazor).
- تحديث Velocity Chart (Sprint 3.5: 8 SP مكتملة).
- تحديث سجل التحديثات.

### v6.0 — 28 أكتوبر 2026

- إغلاق Sprint 3 (58 SP) — PSP Dispensation + Medicine Guide.
- إضافة Sprint 3.5 (Post-Deployment Fixes & Polish) — 24 SP.
- إضافة Epic 13: Rep (Medical Representative).
- إضافة User Story 27 (Rep — رؤية اسم الصيدلية/العيادة).
- إضافة User Story 35 (Rep — إشعار عند قبول الدعوة).
- إضافة الدرس المستفاد رقم 8 (PSP Phases vs Plan Medications).
- تحديث Velocity Chart (Sprint 3 = 109%).
- تحديث Technical Debt (MedicineGuideFlow duplication).

### v5.1 — 27 أكتوبر 2026

- إغلاق مهام Sprint 3 المتعلقة بالبنية التحتية (20 SP).
- إضافة Blocker جديد: بطء لوحة تحكم شركات الأدوية (42 ثانية).
- إضافة الدرس المستفاد رقم 7 (مزامنة Seed Data).
- تحديث 15-DEPLOYMENT.md إلى v3.0 (قسم قاعدة البيانات).

### v5.0 — 26 أكتوبر 2026

- إضافة Epic 12: Integration Hub.
- إضافة Sprint 7-9 (Planned).
- إضافة الوثيقة 22-INTEGRATION-HUB.md.
- تحديث User Story 33, 34.

### v4.0 — 26 أكتوبر 2026

- إضافة Epic 11: PHR.
- إضافة User Story 26 (Hb Navigator).
- إضافة Sprint 4-6 (Planned).
- إنشاء وثيقة 21-PHR-SYSTEM.md.

### v3.0 — 25 أكتوبر 2026

- إغلاق Sprint 2: MAUI Stabilization & Integration (27 SP).

### v2.0 — 23 أكتوبر 2026

- إغلاق Sprint 1: Points-PSP Integration (36 SP).

### v1.0 — 16 أكتوبر 2026

- إنشاء 00B-PROJECT-BACKLOG.

---

## 🔴 Lessons Learned (الدروس المستفادة)

### درس 1: فقدان ربط PSP بعد Migration (22 أكتوبر 2026)

...

### درس 2: ازدواجية BeneficiaryID (22 أكتوبر 2026)

...

### درس 3: PENDING Token (23 أكتوبر 2026)

...

### درس 4: JavaProxyThrowable (25 أكتوبر 2026)

...

### درس 5: patientId=0 (25 أكتوبر 2026)

...

### درس 6: مشكلة الأداء في Pharma Dashboard (27 أكتوبر 2026) — ✅ تم الحل

**المشكلة:** صفحة `dashboard/organization` تستغرق 42 ثانية في الإنتاج (Web + API)، بينما في UAT أجزاء من الثانية.

**السبب الجذري (تم اكتشافه في 29 أكتوبر):** `ApiSettings.BaseUrl` في `appsettings.json` كان يشير إلى `http://161.97.105.83:5235` (منفذ تطوير مغلق). الـ Web كان يحاول الاتصال، يفشل، ينتظر SignalR Timeout (42 ثانية).

**الحل:** تصحيح `BaseUrl` إلى `https://api.rubikcare.com` + ضبط `ASPNETCORE_ENVIRONMENT` إلى `Production`. (التفاصيل الكاملة في قسم Blocker).

**الإجراء الوقائي:** مراجعة `appsettings` قبل كل نشر + إضافة فحص آلي في سكريبت النشر.

### درس 7: مزامنة البيانات (Seed Data) ضرورية قبل النشر

**المشكلة:** نشر كود جديد (Web/API) على بيانات قديمة (Seed Data ناقص) يسبب فشل الطلبات (مثل فشل القوائم، أو استعلامات بطيئة). الحل: قبل أي نشر، يجب التأكد من أن Seed Data متطابقة (عبر MERGE).

**الإجراء الوقائي:** إنشاء Sync-SeedData.ps1 (سيتم في Sprint القادم).

### درس 8: PSP Phases vs Plan Medications — تعارض جوهري (28 أكتوبر 2026)

**المشكلة:** PSPDispensationPlanMedication (القديم) كان يحتوي على NumberOfTokens + DaysBetweenDispense + TimesPerDay — بينما الـ PSPDispensationPlanPhaseMedication (الجديد) يحتوي على RefillIntervalDays + QuantityPerDispense + UnitsPerPack. التوليد كان بيستخدم القديم — فبيطلع Daily Doses غلط (7 في اليوم) و Dispensations غلط (3 بدل 5).

**الحل:** تحويل كل التوليد لـ PhaseMedication + StartAfterMonths + RefillIntervalDays.

**الإجراء الوقائي:** Deprecate PSPDispensationPlanMedication نهائياً في Sprint 4.

### درس 9: إعدادات BaseUrl في appsettings.json — أخطر من Cold Start (29 أكتوبر 2026)

**المشكلة:** ملف `appsettings.json` في بيئة الإنتاج (`RubikCareLive`) كان يحتوي على `BaseUrl = http://161.97.105.83:5235` (منفذ تطوير مغلق). الـ Web كان يحاول الاتصال، يفشل، يعيد المحاولة، ثم ينتظر SignalR Timeout (42 ثانية) قبل أن يظهر الصفحة (بدون بيانات). هذا يسبب بطءً شديداً يبدو وكأنه مشكلة في قاعدة البيانات أو الـ API، لكنه في الحقيقة مشكلة في إعدادات الاتصال.

**الدرس:** لا تفترض أبداً أن البطء بسبب Cold Start أو قاعدة البيانات قبل التحقق من `BaseUrl` في `appsettings.json`. هذا الإعداد البسيط يمكن أن يخفي وراءه مشكلة كبيرة.

**الإجراء الوقائي:**

1. قبل أي نشر، تأكد من أن `BaseUrl` في `appsettings.json` (و`appsettings.Production.json`) يشير إلى الدومين الصحيح (`https://api.rubikcare.com`).
2. لا تستخدم `161.97.x.x` أو منافذ تطوير (`5235`) في ملفات الإنتاج.
3. استخدم سكريبت نشر يتحقق من عدم وجود `5235` أو `161.97` في أي `appsettings` قبل النشر.
4. استخدم `appsettings.Production.json` فقط في الإنتاج (وليس `appsettings.json`).

### درس 10: فارق ASPNETCORE_ENVIRONMENT بين Production و Test في Blazor Server (29 أكتوبر 2026)

**المشكلة:** عندما كان `ASPNETCORE_ENVIRONMENT = Production`، كانت الصفحات تعلق 42 ثانية (بسبب فشل الاتصال بالـ API). عندما تم تغييره إلى `Test`، فتحت الصفحات بسرعة (لكن بدون بيانات). هذا يثبت أن وضع `Production` كان يخفي الأخطاء ويعليق الصفحة، بينما وضع `Test` كان يظهرها (أو يتخطاها بسرعة).

**الدرس:** `ASPNETCORE_ENVIRONMENT` ليس مجرد "إعداد شكلي" — إنه يؤثر على سلوك التطبيق بشكل جذري (Exception Handling, HSTS, SignalR Timeout). لا تغيره لمجرد "تجربة"، ولا تتركه على `Development` في الإنتاج (خطر أمني).

**الإجراء الوقائي:**

1. استخدم `Production` دائماً في بيئة الإنتاج.
2. إذا كنت تشك في وجود خطأ صامت، استخدم `Development` مؤقتاً للتشخيص (مع تفعيل `DetailedErrors`)، ثم أعد `Production` فوراً.
3. راقب سجلات `stdout` بانتظام — الأخطاء لا تظهر في وضع `Production` على الشاشة، لكنها تُسجَّل في الملفات.

---

## 📝 سجل التحديثات

| التاريخ | التحديث |
|---|---|
| 16 أكتوبر 2026 | إنشاء الوثيقة |
| 20 أكتوبر 2026 | Points-PSP Integration (Planned) |
| 23 أكتوبر 2026 | v2.0 — إغلاق Sprint 1 |
| 25 أكتوبر 2026 | v3.0 — إغلاق Sprint 2 |
| 26 أكتوبر 2026 | v4.0 — إضافة PHR (Epic 11) + Sprint 4-6. |
| 26 أكتوبر 2026 | v5.0 — إضافة Integration Hub (Epic 12) + Sprint 7-9. |
| 27 أكتوبر 2026 | v5.1 — إغلاق 20 SP من Sprint 3 + إضافة Blocker جديد (Pharma Dashboard) + الدرس 7. |
| 28 أكتوبر 2026 | v6.0 — إغلاق Sprint 3 (58 SP) + إضافة Sprint 3.5 (24 SP) + Epic 13 (Rep) + الدرس 8. |
| 29 أكتوبر 2026 | v6.1 — حل Blocker (Pharma Dashboard) + TD-001 + الدرسان 9, 10 + تحديث TD #8. |

---

## 🔗 روابط ذات صلة

- 00-MASTER-INDEX.md
- 17-STRATEGIC-ROADMAP.md
- 19-PERMISSIONS.md
- 21-PHR-SYSTEM.md
- 22-INTEGRATION-HUB.md (جديد)
- 15-DEPLOYMENT.md (v3.0)

---

© 2026 RubikCare — للاستخدام الداخلي  
نهاية الوثيقة ✅
```

---
