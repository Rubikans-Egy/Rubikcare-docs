# 00B — PROJECT-BACKLOG

الاسم: PROJECT-BACKLOG — سجل التطوير الحي الإصدار: 6.1 (Production Blocker Resolved Edition) آخر تحديث: 29 أكتوبر 2026 Sprint الحالي: Sprint 3.5 (In Progress) الحالة: 🟢 حي — يُحدَّث كل جلسة الجمهور: فريق التطوير + AI Assistants

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

## 🎯 الرؤية الحالية

RubikCare في مرحلة Pre-Launch. الهدف الحالي:

- إكمال Backend + Frontend الأساسي.
- إصدار Production مستقر.
- بناء الأساس للسجلات الطبية (PHR) والتكامل الخارجي.

## 📊 Sprint 1 — Points-PSP Integration (22-23 أكتوبر 2026) — ✅ مكتمل

الهدف: إغلاق الفجوة بين نظام النقاط و PSP (9 عقود تكامل — IC-01 → IC-09).

### ✅ Done (12 مهمة)

# | المهمة | SP | الحالة 
---|---|---|---
P1 | توثيق Integration Contracts | 2 | ✅ 
P2.1 | Migration: REDEEM_DISPENSATION (ID=9) | 1 | ✅ 
P2.2 | إعادة كتابة GenerateNextToken (ذرية) | 5 | ✅ 
P2.3 | إصلاح ConfirmDispense (احترام الخصم) | 3 | ✅ 
P2.4 | إصلاح PspController.patientdetails | 2 | ✅ 
P2.5 | إضافة MonetaryValuePerPoint إلى DTO | 1 | ✅ 
P3.1 | PspDetailComponent.razor (Dialog + Success) | 3 | ✅ 
P3.2 | CSS: _shared/dialogs.css | 2 | ✅ 
P3.3 | إصلاح PspStatsBar | 1 | ✅ 
P4.1 | Idempotency في TryAwardPointsViaOrchestrator | 2 | ✅ 
P4.2 | Idempotency في AcknowledgeReminderHandler | 1 | ✅ 
P6.1 | إشعار عند كسب/خصم نقاط | 2 | ✅ 

### ✅ Done (إضافي)

# | المهمة | SP 
---|---|---
D1 | إصلاح IRefreshService في PWA | 1 
D2 | إصلاح فقدان ربط PSP بعد Migration (22 حالة) | 3 
D3 | إصلاح حساب النقاط (PatientProfileID بدلاً من CaseID) | 2 
D4 | إصلاح عرض الخصم للصيدلي (PointsDiscount) | 2 
D5 | إصلاح بطاقة "لا يوجد توكن" + عرض التوكن الجديد | 1 
D6 | إصلاح تجاوب BarcodeScanner (CSS) | 1 

المجموع: 36 SP (Sprint 1) + 10 SP (إضافي) = 46 SP

### ⏳ Pending (المتبقي)

# | المهمة | SP | الأولوية 
---|---|---|---
P5.1 | GetAdherenceHandler — إضافة Points | 2 | 🟠 
P5.2 | GetPharmacyFinancialHandler — فصل PointsDiscount | 3 | 🟠 
P5.3 | GetProgramLifecycleHandler — إضافة Points | 2 | 🟠 
P7.1 | الترجمة (SQL MERGE) | 1 | 🟡 
P7.2 | اختبار E2E كامل | 3 | 🟠 

المجموع المتبقي: 11 SP

## 📊 Sprint 2 — MAUI Stabilization & Integration (24-25 أكتوبر 2026) — ✅ مكتمل

الهدف: حل مشاكل MAUI (JavaProxyThrowable، QueryProperty، InvalidCastException، patientId=0).

### ✅ Done (13 مهمة)

# | المهمة | SP 
---|---|---
M1 | إنشاء PatientPointsFlow.xaml + .xaml.cs | 2 
M2 | تعديل PspDetailComponent.razor — استخدام Action | 2 
M3 | تعديل PatientPointsDashboard.razor — استخدام Bridge | 2 
M4 | إصلاح InvalidCastException (userId.ToString()) | 1 
M5 | ربط كل دوال PspDetailNavigationBridge | 3 
M6 | استخدام Dispatcher.DispatchAsync | 1 
M7 | إصلاح MedicationScheduleFlow.OnNavigateToPsp | 2 
M8 | إصلاح MedicationSchedulePage.OnRefillRowClick | 1 
M9 | إضافة CaseID إلى DispensationScheduleDto | 2 
M10 | إصلاح PspSchedulePageFlow | 2 
M11 | تحديث 13-maui-development-guide.md | 3 
M12 | إصلاح PspController.patientDetails | 1 
M13 | إصلاح DispensationScheduleDto | 1 

### ✅ Done (إضافي)

# | المهمة | SP 
---|---|---
M14 | توحيد PspDetailFlow | 2 
M15 | حذف ScanTokenPage.razor (القديم) | 1 
M16 | إصلاح PharmacyGateway | 1 

المجموع الكلي: 27 SP

## 🎯 Sprint 0 — Foundation (16-22 أكتوبر 2026) — ⏳ In Progress

### ✅ Done

# | المهمة | SP 
---|---|---
1 | تحديث 09-API-Guide v2.0 | 5 
2 | كتابة 03-MENUS v4.0 | 5 
3 | تجربة السكريبت التوليدي | 1 

### ⏳ Pending

# | المهمة | SP 
---|---|---
5 | تحسين السكريبت v2 | 2 
6 | كتابة 19-PERMISSIONS | 5 
7-11 | 5 Migrations | 7 

المجموع المتبقي: 14 SP

## 📊 Sprint 3 — PSP Dispensation & Medicine Guide (26-28 أكتوبر 2026) — ✅ مكتمل

الهدف: إكمال دورة الصرف بناءً على الـ Phases + صفحة Medicine Guide منفصلة + تحسين الإشعارات.

### ✅ Done (24 مهمة)

# | المهمة | SP | ملاحظات 
---|---|---|---
S3-Done-1 | تنظيف بيئة النشر (حذف المواقع القديمة، أرشفة _Archive) | 2 | — 
S3-Done-2 | إعداد SharedStorage وأذوناته (Web + API) | 3 | تم حل مشكلة رفع الملفات. 
S3-Done-3 | ترقية قاعدة البيانات (UAT → Production) | 5 | تم تطبيق Migration RenamePatientIdToCaseId مع تنظيف السجلات اليتيمة. 
S3-Done-4 | مزامنة Seed Data الناقصة (Menus, ActivityTypes, ...) | 3 | تم حل مشكلة القوائم المفقودة. 
S3-Done-5 | إنشاء نظام سكريبتات النشر (Web, API, PWA) + Rollback | 5 | 12 سكريبت جاهز في _Docs\Scripts. 
S3-Done-6 | توثيق 15-DEPLOYMENT.md (v3.0) مع قسم قاعدة البيانات | 2 | — 
S3-Done-7 | Backend: Phases support في PSPEnrollmentService (RefillIntervalDays + StartAfterMonths) | 3 | توليد التوكنات بناءً على المراحل. 
S3-Done-8 | Backend: GenerateSchedulesFromERxAsync — استخدام PhaseMedication بدل PlanMedication | 3 | Daily Doses صحيحة (1-2 يومياً بدل 7). 
S3-Done-9 | Backend: GetDispensationSchedulesAsync — إزالة Include(Plan) + planMap + PENDING→PROJECTED | 2 | 11 صف بدل 9 + PROJECTED ظاهر. 
S3-Done-10 | Backend: Lazy Activation — ActivateEligiblePendingTokensAsync | 2 | PENDING → GENERATED تلقائياً. 
S3-Done-11 | Backend: ConfirmDispense — DispenseDate + ActivateNextPendingTokenAsync | 2 | التوكن التالي يتفعل بعد RefillIntervalDays. 
S3-Done-12 | Backend: ReminderProcessingService — التحقق من ScheduleStatus (تجنب الإشعار المتكرر) | 2 | مش بيبعت بعد "تناول". 
S3-Done-13 | Backend: GetMedicationDetailsAsync — endpoint جديد (CaseID) + Fallback للبرامج بدون Phases | 3 | Medicine Guide page data. 
S3-Done-14 | Web: PSPStep2_DispensationPlans — إعادة هيكلة Plan Settings + Phase Medication | 3 | حقول جديدة (RefillIntervalDays, QuantityPerDispense, UnitsPerPack) + Auto-calc. 
S3-Done-15 | Web: CanRequestEarly + EarlyRequestDays + "Coming Soon" لـ Single/AllowRefill | 2 | — 
S3-Done-16 | Mobile: MedicineGuidePage (NEW) + MedicineGuideFlow + Route | 3 | صفحة منفصلة /medicine-guide. 
S3-Done-17 | Mobile: PspSchedulePage (NEW component) — برامج + توليد + FirstDoseTime لكل برنامج | 3 | بدل duplicate of RefillDatesTab. 
S3-Done-18 | Mobile: AddMedicationPage validation + Toast + PspDetailComponent زرار "دليل الأدوية" | 2 | — 
S3-Done-19 | Mobile: Navigation Bridges (GoBack, GoToTests, GoToMedicineGuide, CopyToken, ShareToken) | 2 | — 
S3-Done-20 | Mobile: PatientTestsFlow + Route + PatientSession update قبل GoToTests | 2 | — 
S3-Done-21 | DB: Migration AddRefillFieldsToPhaseMedications | 2 | RefillIntervalDays, QuantityPerDispense, UnitsPerPack. 
S3-Done-22 | DB: MERGE Plans (TokenValidityHours=720) + MERGE Patient 1081 (15 dispensations) | 2 | — 
S3-Done-23 | DB: MERGE Resources (translations — Medicine Guide, Dispense Status, Phase fallback) | 1 | — 
S3-Done-24 | Build: PublishTrimmed=false + AndroidEnableProfiledAot=false (IL Trimmer fix) | 1 | — 

المجموع المنجز: 58 SP.

## 🗓️ Sprint 3.5 — Post-Deployment Fixes & Polish (28-31 أكتوبر 2026) — ⏳ In Progress

الهدف: إغلاق المهام الناتجة عن اختبار UAT + تحسينات UX المطلوبة.

### 🔴 High Priority

# | المهمة | SP | ملاحظات 
---|---|---|---
S3.5-H1 | إصلاح Crash في صفحة البرامج — وضع المندوب (Rep) | 3 | الصفحة تعمل Crash عند فتحها في وضع Rep — يحتاج فحص Stack Trace + إصلاح NullReference. 
S3.5-H2 | إظهار اسم الصيدلية/العيادة في "شبكتي" أو "دعواتي" بعد قبول الدعوة | 5 | حالياً المندوب لا يرى اسم الجهة بعد اشتراكها — يحتاج ربط الدعوة بـ PSPParticipation + إرجاع OrganizationName. 
S3.5-H3 | إظهار اسم برنامج الدعم بجانب كل دواء في تاب "مواعيد الأدوية" | 2 | حالياً يعرض MedicationName فقط — يجب إضافة ProgramName. 
S3.5-H4 | الضغط على الدواء في "مواعيد الأدوية" يفتح MedicineGuidePage | 2 | Navigation من MedicationSchedulePage → /medicine-guide?medicationId=X. 
S3.5-H5 | إصلاح الإشعارات المتكررة — بعد "تناول" مفيش إشعار، لو ما ضغطش تجيله مرتين فقط | 3 | بعد إصلاح ReminderProcessingService — يحتاج اختبار + ضبط MaxRepeats=2. 
S3.5-H6 | ✅ إصلاح Blocker: بطء Pharma Dashboard (42s) + اختفاء الأشكال البيانية | 8 | **مكتمل (29 أكتوبر):** السبب الجذري كان `ApiSettings.BaseUrl` في `appsettings.json` يشير إلى منفذ تطوير مغلق (5235). تم تصحيحه إلى `https://api.rubikcare.com` + ضبط `ASPNETCORE_ENVIRONMENT` إلى `Production`. الصفحات تعمل الآن بسرعة وبدون أخطاء. |

المجموع (High): 23 SP (15 SP منها مكتملة)

### 🟠 Medium Priority

# | المهمة | SP | ملاحظات 
---|---|---|---
S3.5-M1 | تحسين CSS في MedicineGuidePage — Program Header + Hero + Empty State | 2 | تم جزئياً — يحتاج مراجعة نهائية. 
S3.5-M2 | إضافة قسم "حالة الصرف" في MedicineGuidePage (TotalDispensed + LastDispensedDate) | 2 | البيانات موجودة في DTO — يحتاج عرض UI. 
S3.5-M3 | عرض اسم البرنامج + Phase في كارت الدواء (MedicineGuidePage) | 1 | تم — يحتاج تأكيد. 
S3.5-M4 | تحسين رسالة "No Medications" للبرامج بدون Phases | 1 | "البرنامج مش متوفر — تواصل مع الدعم" بدل "اشترك في برنامج". 

المجموع (Medium): 6 SP

### 🟡 Low Priority

# | المهمة | SP | ملاحظات 
---|---|---|---
S3.5-L1 | إضافة "Medicine Guide" card في PspDetailComponent | 1 | تم — يحتاج تأكيد. 
S3.5-L2 | تحسين ترتيب الـ Bridges في PspDetailFlow | 1 | تنظيم الكود فقط. 
S3.5-L3 | حذف MedicineGuideFlow المكرر إذا وجد | 1 | تنظيف. 

المجموع (Low): 3 SP

المجموع (Sprint 3.5): 32 SP (8 منها مكتملة في 29 أكتوبر)

## 📈 Velocity Chart

Sprint | Planned | Actual | Ratio 
---|---|---|---
Sprint 1 | 36 | 46 | 128% ✅ (مع إضافي) 
Sprint 2 | 27 | 27 | 100% ✅ 
Sprint 0 | 22 | 11 | 50% 🟡 
Sprint 3 | 53 | 58 | 109% ✅ (مع إضافي) 
Sprint 3.5 | 24 | 8 (H6) | ⏳ In Progress 

## 🎯 Epic Overview

Epic | User Stories | Done | Completion 
---|---|---|---
1: Foundation & Tooling | 5 | 2 | ~40% 
2: Permissions System | 4 | 0 | 0% 
3: PSP System | 15 | 15 | ~100% ✅ 
4: Points System | 12 | 12 | ~100% ✅ 
5: Reports System | 8 | 7 | ~85% 
6: Notifications System | 6 | 6 | ~100% ✅ 
7: PWA | 10 | 9 | ~90% 
8: Menus & Services | 6 | 0 | 0% 
9: Patient Mobile (MAUI) | 3 | 3 | ~100% ✅ 
10: Deployment & Release | 4 | 1 | ~25% 
11: Personal Health Records (PHR) | 3 (Phases) | 0 | 0% 
12: Integration Hub | 3 (Phases) | 0 | 0% 
13: Rep (Medical Representative) | 4 (Phases) | 0 | 0% 

## 📋 Product Backlog (المستقبلي)

### 🔴 High Priority

# | User Story | Epic | SP 
---|---|---|---
20 | كـ مريض، أريد رؤية رصيد النقاط في Mobile | Points | 3 
21 | كـ Admin، أريد إدارة Control Levels | Permissions | 5 
22 | كـ Admin، أريد ربط Users بـ Control Levels | Permissions | 3 
23 | كـ Super Admin، أريد إدارة Permissions | Permissions | 5 
24 | كـ Admin، أريد إدارة اشتراك المنظمة | Subscriptions | 5 
25 | كـ Admin، أريد ربط SubscriptionPlan بـ Menus | Menus | 5 
26 | كـ طبيب/مريضة، أريد أداة Hb Navigator | PHR | 13 
27 | كـ Rep، أريد رؤية اسم الصيدلية/العيادة بعد قبول الدعوة في "شبكتي" | Rep | 5 

### 🟠 Medium Priority

# | User Story | Epic | SP 
---|---|---|---
30 | Menus ديناميكية في PWA | Menus | 5 
31 | Menus ديناميكية في MAUI | Menus | 5 
32 | ربط ServiceProduct بـ MenuItems | Menus | 3 
33 | ApiClient موحّد (Integration Hub) | Integration Hub | 8 
34 | API-Contracts (OpenAPI) | Integration Hub | 5 
35 | كـ Rep، أريد إشعار عند قبول الدعوة من صيدلية/عيادة | Rep | 3 

### 🟡 Low Priority

# | User Story | Epic | SP 
---|---|---|---
40 | إدارة OrganizationGroups | Organization | 5 
41 | Rubika Currency Engine | Rubika | 8 
42 | Deep Linking من QR | Mobile | 3 

## 🏗️ Technical Debt Register

### 🔴 Critical Debt

# | الدين | الموقع 
---|---|---
1 | انتهاك معماري: @page في Shared.UI | PharmacyDetailPage.razor 
2 | خلط Namespaces | PharmacySearchPage.razor 
3 | 404 في /admin/pending-licenses | SystemManagment vs SystemManagement 

### 🟡 Medium Debt

# | الدين | الموقع 
---|---|---
4 | زر البحث في Dashboard غير مربوط | Pages/Dashboard.razor 
5 | ازدواجية FirebaseNotificationService | Api.Web + Infrastructure 
6 | ازدواجية IPSPPointSchemeService | Application + Web 
7 | عدم تحديث PROJECT-BACKLOG أسبوعياً | عملية 
8 | ✅ **ازدواجية BaseUrl في appsettings (Web)** — تم تصحيح `RubikCareLive` و `RubikCareNew` في 29 أكتوبر. | RubikCareLive و RubikCareNew 
9 | ازدواجية MedicineGuideFlow (إذا وجد) | Mobile/Features/PSP/Patient/Views/ 
18 | 🟡 **TD-001: الطلبات المكررة (Double API Calls)** — بعض الصفحات (Dashboard, Loyalty, Reports) ترسل نفس طلب الـ API مرتين في نفس الثانية. | `Rubikcare.Web` (Blazor Server) + `Api.Web` (السجلات) 

#### تفاصيل TD-001:

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

# | الدين | الموقع 
---|---|---
10 | IMobileNavigationService (اسم مضلل) | Shared.UI/Services/ 
11 | HttpUtility في Blazor WASM | PharmacySearch.razor 
12 | alert() بدلاً من Toast | PspSearch.razor 
13 | SystemMenuMenuID Shadow Property | MenuAssignments 
14 | RoleId في MenuVisibilityRule | بعد Migration 
15 | PspDetailFlow — لا يربط GoHome | MAUI (تم إصلاحه في Sprint 3) 
16 | PspSchedulePageFlow — لا يمرر PatientId | MAUI (تم إصلاحه في Sprint 3) 
17 | MedicineGuidePage — LoadTranslations (PSP + SHARED.PSP_DETAIL) | تم إصلاحه في Sprint 3.5 

## ✅ Blocker — Pharma Dashboard Performance (تم الحل في 29 أكتوبر 2026)

**الحالة:** ✅ **مغلق** (تم حل المشكلة نهائياً).

**الوصف الأصلي:** صفحات `dashboard/organization` و `reports/*` في واجهة شركات الأدوية (Pharma Company) تستغرق 42 ثانية للتحميل في الإنتاج، بينما تفتح في UAT في أجزاء من الثانية. الصفحات الأخرى (مثل loyalty و psp) تعمل بشكل طبيعي.

**السبب الجذري (تم اكتشافه في 29 أكتوبر):**
- ملف `appsettings.json` في مجلد `RubikCareLive` (بيئة الإنتاج) كان يحتوي على:
  ```json
  "ApiSettings": {
      "BaseUrl": "http://161.97.105.83:5235"
  }
  
