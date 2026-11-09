# 00B — PROJECT-BACKLOG

الاسم: PROJECT-BACKLOG — سجل التطوير الحي  
الإصدار: 6.2 (Production Blocker Resolved Edition)  
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
|S3.5-H7 | ✅ تثبيت بيئة نشر PWA (إعدادات البيئة appsettings، حل حلقة cache-reset اللانهائية، وحماية web.config)| 5 | مكتمل (31 أكتوبر): تم اعتماد سكريبت نشر آمن (v4.0)، إصلاح منطق MAX_ATTEMPTS في صفحة التنظيف، وضبط اتصال الـ PWA بـ API الإنتاج بنجاح. |

### 🟢 مكتمل في هذه الجلسة (4 نوفمبر 2026)

| # | المهمة | SP | ملاحظات |
|---|---|---|---|
| S3.5-H14 | إعادة تصميم `DashboardPage.razor` (المشترك) | 8 | ✅ مكتمل في PWA (Header + Stats + Program Cards + Adherence + Reminders). |
| S3.5-H15 | ربط `DashboardPage.razor` بـ API الحقيقية | 5 | ✅ `psp/entry` + `patient-details` + `patient-dashboard` + `schedules`. |
| S3.5-H16 | إصلاح RTL/LTR في `DashboardPage.razor` (PWA) | 2 | ✅ `TranslationState` + `localStorage`. |
| S3.5-H17 | إضافة `ProgramCardDto` (`CaseID` + `ParticipationId`) | 2 | ✅ Backend + PWA (يُستخدم في التوجيه). |
| S3.5-H18 | ربط `PatientPointsDashboard` (`/patient/points`) | 3 | ✅ PWA + MAUI (يعمل). |
| S3.5-H19 | إصلاح `MedicineGuidePage` (استقبال `caseId` + `programId`) | 2 | ✅ PWA (يعمل). |
| S3.5-H20 | إنشاء `CurrentUserProfileState` (Singleton) | 1 | ✅ مُنشأ (لكن غير مستخدم في MAUI — بسبب نسختين). |
| S3.5-H21 | إنشاء `DashboardNavigationBridge` | 1 | ✅ مستخدم في PWA + MAUI. |

**المجموع (مكتمل):** 24 SP
### ✅ مكتمل في 5 نوفمبر 2026 (الجزء الثاني)

| # | المهمة | SP | ملاحظات |
|---|---|---|---|
| S3.5-H27 | **`DashboardFlowPage` — الصفحة الرئيسية الجديدة** | 8 | `CurrentUserProfileState` + `<RootComponent>` في XAML. يعمل. |
| S3.5-H28 | **`DashboardPage.razor` — استخدام `CurrentUserProfileState`** | 3 | بدلاً من `[Parameter] UserProfileId`. يعمل. |
| S3.5-H29 | **`OnboardingPage` — قبل تسجيل الدخول** | 5 | يظهر مرة واحدة (`has_seen_onboarding`). يعمل. |
| S3.5-H30 | **`MedicineGuidePage` — `CaseIdOverride` + `ProgramIdOverride`** | 3 | يعمل (بعد تمرير `caseId` + `programId` في `ConvertToMauiRoute`). |
| S3.5-H31 | **`PatientPointsFlow` — تمرير `userProfileId`** | 2 | يعمل (بعد `ConvertToMauiRoute`). |
| S3.5-H32 | **`DashboardNavigationBridge` — `Navigate` + `OpenMenu`** | 2 | يعمل. |
| S3.5-H33 | **`13-MAUI.md` v4.0 — إعادة كتابة شاملة** | 5 | 5 أنماط ناجحة + 4 فاشلة + 7 دروس جديدة. |
| S3.5-H34 | **إصلاح `MainPage = shell` قبل `InitializeAsync`** | 2 | `App.xaml.cs` — يمنع ومضة `LoginPage`. |
| S3.5-H35 | **إصلاح `has_seen_onboarding` = `true` (تم تعيينه مسبقاً)** | 1 | `Preferences.Remove("has_seen_onboarding")` مؤقتاً للتجربة. |

**المجموع (مكتمل — الجزء الثاني):** 31 SP
### 🟡 مؤجل في هذه الجلسة

| # | المهمة | SP | السبب |
|---|---|---|---|
| S3.5-H22 | استخدام `DashboardPage.razor` (المشترك) في MAUI | — | ❌ فشل (تعارض `BlazorWebView` + `ShellContent` — انظر الدروس 12-16). |
| S3.5-H23 | تعديل `DashboardPage.xaml` (المحلية) لتطابق PWA (XAML) | 8 | ⏳ مؤجل (الخيار B). |
| S3.5-H24 | قوائم ديناميكية في `AppShell` (من DB) | 13 | ⏳ مؤجل (أسهل من `BlazorWebView`). |
| S3.5-H25 | تحويل `AppShell` إلى صفحة بسيطة + قائمة مشتركة | — | ⏳ مؤجل (مخاطرة عالية). |
| S3.5-H26 | إصلاح ظهور مفاتيح الترجمة في بعض صفحات MAUI | 5 | ⏳ مؤجل (`04-TRANSLATION` § 4.6). |

**المجموع (مؤجل):** 26 SP (بدون H22/H25 — لأنها فشلت/مؤجلة بدون SP محدد).
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
### 🎉 ملخص جلسة 5 نوفمبر 2026 — نجاح `DashboardFlowPage`

**الهدف:** استبدال `DashboardPage` (القديمة) بـ `DashboardPage.razor` (المشترك) في MAUI — كصفحة رئيسية.

**النتيجة:** ✅ **نجاح كامل** (بعد فشل متكرر في محاولات سابقة).

**الحل الناجح:**
1. `DashboardFlowPage.xaml` — `<BlazorWebView>` + `<RootComponent>` في XAML (بدون `Parameters`).
2. `DashboardFlowPage.xaml.cs` — `CurrentUserProfileState.SetUserProfile` في `Constructor` (من `Preferences`).
3. `DashboardPage.razor` — `@inject CurrentUserProfileState UserState` (بدلاً من `[Parameter] UserProfileId`).
4. `App.xaml.cs` — `MainPage = shell` **قبل** `InitializeAsync`.
5. `AppShell.InitializeAsync` — `GoToAsync` أولاً (بدون `Task.Delay`)، `RefreshUnreadNotificationsCount` في الخلفية.
6. `AppShell.xaml` — `Onboarding` أول `ShellContent` (مرة واحدة — بدون تكرار).

**الصفحات التي تم اختبارها (وتعمل):**
- ✅ `DashboardFlowPage` (الصفحة الرئيسية الجديدة).
- ✅ `OnboardingPage` (قبل تسجيل الدخول).
- ✅ `PatientPointsFlow` (من Dashboard + PspDetail).
- ✅ `MedicineGuideFlow` (من Dashboard — بعد تمرير `caseId` + `programId`).
- ✅ `PspDetailFlow` (من Dashboard).
- ✅ `MedicationScheduleFlow` (من Dashboard).
- ✅ `PspAboutFlow` (من Dashboard — إذا لم تكن هناك نقاط).

**الصفحات التي تم التراجع عنها (مؤقتاً):**
- ❌ `DashboardPage` (القديمة — أصبحت `LegacyDashboard` — مخفية).
- ❌ `Onboarding` (الموسّع — في جلسة منفصلة).

**ما تم توثيقه:**
- ✅ `13-MAUI.md` v4.0 — 5 أنماط ناجحة + 4 فاشلة + 7 دروس جديدة.
- ✅ `01-ARCHITECTURE.md` v2.1 — قيود `BlazorWebView` + `Shared.UI` (Page Components).
- ✅ `00-MASTER-INDEX.md` v2.3 — تحديث `Modules Snapshot` + `Dark Areas` + `Epic 14`.

**ما تبقى (لجلسات منفصلة):**
- ⏳ `OnboardingPage` (موسّع — صور + روابط + شرح مفصل).
- ⏳ القوائم الديناميكية (Epic 8 — 13 SP).
- ⏳ `OrganizationSubscription` (خطط الاشتراك).
- ⏳ `MenuVisibilityRules` (التحكم في الميزات).
- ⏳ `DashboardPage.xaml` (MAUI) — تطابق PWA (XAML).
- ⏳ إصلاح مفاتيح الترجمة في بعض صفحات MAUI (`S3.5-H26`).
## 🗓️ Sprint 3.6 — Post-Deployment Fixes (Round 2) — ✅ مكتمل (9 نوفمبر 2026)

الهدف: إغلاق المهام الناتجة عن اختبار UAT (Round 2) — مشاكل Workflow، UI، و Notifications.

### ✅ High Priority (مكتمل)

| # | المهمة | SP | ملاحظات | الحالة |
|---|---|---|---|---|
| S3.6-H1 | تفعيل دعوة مريض من دكتور لبرنامج دعم بشكل صحيح | 5 | ✅ اتحل: `EnrollUserInProgramAsync` بيفلتر بـ `ProgramID` (مش بس `PatientProfileID`). `ScanTokenPage` بقى بيستخدم `NewProgram` بدل `ActivePrograms.FirstOrDefault()`. `ScanTokenViewModel` بقى يعرض رسالة نجاح واضحة قبل الـ Navigation. | ✅ مكتمل |
| S3.6-H2 | مشكلة صفحة تفاصيل البرنامج — تجديد توكن + تأكيد استلام | 5 | ✅ اتحل: `ShowRenewCard` بقى بيعتمد على `CanRequestEarly` + `DaysBetweenDispense` + `EarlyRequestDays`. زرار "تأكيد الاستلام" اتشال من `PspDetailComponent` (بقى في "طلباتي"). زرار "متابعة الطلب" بيظهر لما `TokenStatus = APPROVED` / `DISPENSED`. زرار "تجديد" بيظهر رسالة لو المريض لسه ما أكدش الاستلام. | ✅ مكتمل |
| S3.6-H3 | إصلاح Error بدل صفحة النقاط (`points-dashboard`) | 3 | ✅ اتحل ضمن تعديلات `PatientPointsFlow` (route + tokenCode). | ✅ مكتمل |
| S3.6-H4 | توليد مواعيد الأدوية — أكتر من دواء في نفس المعاد | 5 | ✅ اتحل: `GenerateSchedulesFromERxAsync` بقى بيولّد جدول مستقل لكل دواء (من `PhaseMedication` بدل `PlanMedication`). | ✅ مكتمل |
| S3.6-H5 | تعديل منطق زر "ابدأ رحلتك العلاجية" | 3 | ✅ اتحل: `StartedJourney` بقى بيعتمد على `CanStartTreatment` + `MandatoryBeforePending`. `SaveResult` بيحدّث `StartDate` أوتوماتيك بعد رفع التحاليل الإلزامية. | ✅ مكتمل |

### ✅ Medium Priority (مكتمل)

| # | المهمة | SP | ملاحظات | الحالة |
|---|---|---|---|---|
| S3.6-M1 | تظبيط تاريخ الأوردر في "طلباتي" (المريض والصيدلي) | 2 | ✅ اتحل: `OrderTracker` + `PatientOrdersPage` بيستخدموا `yyyy/MM/dd HH:mm` بشكل موحد. | ✅ مكتمل |
| S3.6-M2 | صفحة "دعواتي" من عند الدكتور | 3 | ✅ اتحل: `MyPatientInvitations.razor` بقت بتعرض `InvitationToken` + `CanResend` + إعادة إرسال الدعوة. | ✅ مكتمل |
| S3.6-M3 | الإشعارات | 3 | ✅ اتحل جزئياً: `SubmitForApproval` بيبعت إشعار للصيدلية. `ConfirmReceipt` بيبعت إشعار للمريض. `MedicineGuidePage` بقى يفتح من `PspDetailComponent` + `MedicationSchedulePage`. | ✅ مكتمل |

### 🟡 Low Priority (مؤجل)

| # | المهمة | SP | ملاحظات | الحالة |
|---|---|---|---|---|
| S3.6-L1 | تعديل صفحات المقدمة (Landing Pages) + ترتيبها | 5 | UI/UX — تحسين الترتيب والتنسيق. | ⏳ مؤجل |

**المجموع (Sprint 3.6):** 34 SP — **29 SP مكتمل** (85%) — **5 SP مؤجل**.

---
## 🗓️ Sprint 3.6 — Post-Deployment Fixes (Round 2) — ⏳ In Progress

الهدف: إغلاق المهام الناتجة عن اختبار UAT (Round 2) — مشاكل Workflow، UI، و Notifications.

### ✅ High Priority

| # | المهمة | SP | الحالة | ملاحظات |
|---|---|---|---|---|
| S3.6-H1 | تفعيل دعوة مريض من دكتور لبرنامج دعم بشكل صحيح | 5 | ✅ مكتمل (9 نوفمبر) | Fix `EnrollUserInProgramAsync` (filter by `ProgramID`) + `ScanTokenPage` (`NewProgram` بدل `ActivePrograms.FirstOrDefault()`). |
| S3.6-H2 | مشكلة صفحة تفاصيل البرنامج — تجديد توكن + متابعة الطلب | 5 | ✅ مكتمل (9 نوفمبر) | Fix `ShowRenewCard` (`CanRequestEarly` + `DaysBetweenDispense`) + `ShowFollowUpButton` + `ShowFollowUpBeforeApproval` + `RenewToken` (يمنع لو لسه ما أكدش). |
| S3.6-H3 | إصلاح Error بدل صفحة النقاط (`points-dashboard`) | 3 | ⏳ لسه | `PatientPointsFlow` / `points-dashboard` — خطأ بيظهر بدل الصفحة. |
| S3.6-H4 | توليد مواعيد الأدوية — أكتر من دواء في نفس المعاد | 5 | ⏳ لسه | لو برنامج فيه أكتر من دواء، التوليد بيجبرهم كلهم على نفس الوقت بدل ما كل دواء ليه جدوله المستقل. |
| S3.6-H5 | تعديل منطق زر "ابدأ رحلتك العلاجية" | 3 | ⏳ لسه | زر `StartTreatment` — منطق التفعيل/الظهور. (ملاحظة: إصلاح رفع الملفات في `StartedJourney` تم، لكن الـ logic بتاع الزر نفسه لسه). |

### 🟠 Medium Priority

| # | المهمة | SP | الحالة | ملاحظات |
|---|---|---|---|---|
| S3.6-M1 | تظبيط تاريخ الأوردر في "طلباتي" (المريض والصيدلي) | 2 | ⏳ لسه | التاريخ بيظهر بشكل غلط — يحتاج format موحد. |
| S3.6-M2 | صفحة "دعواتي" من عند الدكتور | 3 | ⏳ لسه | `MyInvitations` (Doctor) — مش شغالة صح / مش ظاهرة. |
| S3.6-M3 | الإشعارات | 3 | ⏳ لسه | مراجعة عامة — إشعارات مش بتظهر / متأخرة / مكررة. |

### 🟡 Low Priority

| # | المهمة | SP | الحالة | ملاحظات |
|---|---|---|---|---|
| S3.6-L1 | تعديل صفحات المقدمة (Landing Pages) + ترتيبها | 5 | ⏳ لسه | UI/UX — تحسين الترتيب والتنسيق. |

**المجموع (Sprint 3.6):** 34 SP — **10 SP مكتمل** (29%) — **24 SP متبقي**.

### 📝 تفاصيل المهام المنجزة (Sprint 3.6)

#### S3.6-H1: تفعيل دعوة مريض من دكتور
- **Backend:** `EnrollUserInProgramAsync` (في `PSPEnrollmentService.cs`) بقى بيفلتر `PSPPatient` بـ `Participation.ProgramID` (مش بس `PatientProfileID`). ده يضمن إن المريض بيتعمله `INSERT PSPPatient` جديد لكل برنامج.
- **Frontend (Mobile):** `ScanTokenPage.xaml.cs` — شيلنا `finally` block اللي كان بيعيد الـ Scan. ضفنا `PropertyChanged` listener لإعادة تفعيل الـ Scan بعد `ResetScan`.
- **Frontend (Shared.UI):** `ScanTokenViewModel.cs` — ضفنا `SetSuccess` + `SetInfo` + `ScanPageState.Info`. `ProcessInvitationCode` بقى بيستخدم `NewProgram` (مش `ActivePrograms.FirstOrDefault()`) + بيعرض رسالة نجاح قبل الـ Navigation.
- **Frontend (DTOs):** `PspEntryApiResponse` — ضفنا `NewProgram`.

#### S3.6-H2: مشكلة صفحة تفاصيل البرنامج
- **Backend:** `Pspcontroller.patientdetails.cs` — ضفنا `HasPendingPharmacyRequest` + `PharmacyId` + `PharmacyName` + `CanRequestEarly` + `EarlyRequestDays` + `TokenGeneratedDate` + `LastDispensedDate`. إصلاح `TokenStatus` (يستخدم `lastDispensation` لو `currentDispensation = null`). إصلاح `TokenCode` + `TokenGeneratedDate` + `TokenExpiryDate` (يستخدموا `referenceDispensation`).
- **Frontend (Shared.UI):** `PspDetailComponent.razor` — `ShowRenewCard` بيعتمد على `CanRequestEarly` + `DaysBetweenDispense` + `EarlyRequestDays`. `IsRenewButtonDisabled` + `GetRenewHint` جديدين. `ShowConfirmButton` اتحول لـ `ShowFollowUpButton` (يودي على "طلباتي"). `ShowFollowUpBeforeApproval` (يظهر لما الطلب يتبعت للصيدلية، لسه ما وافقتش). `RenewToken` بقى بيمنع التجديد لو `TokenStatus = APPROVED` / `DISPENSED` (لازم يأكد الاستلام الأول).
- **Frontend (Shared.UI):** `PspDetailNavigationBridge.cs` — `GoToMyOrders` بقى `Action<string?>` (بيمرر `tokenCode`).
- **Frontend (Shared.UI):** `PatientOrdersPage.razor` — `HighlightTokenCode` بيستخدم `SelectPspOrder` (مش `_orders` + `OpenOrderDetails`).
- **Frontend (Mobile):** `PspDetailFlow.xaml.cs` — `GoToMyOrders` بقى بيروح `my-orders` (مش `patient-orders` — الصيدلي) + بيمرر `tokenCode`.
- **Frontend (Mobile):** `PatientOrdersBlazorPage.xaml.cs` — استقبال `tokenCode` عبر `IQueryAttributable` + تمريره `HighlightTokenCode`.
- **Frontend (Mobile):** `PatientOrdersBlazorPage.xaml` — ضفنا `x:Name="blazorWebView"`.
- **Frontend (PWA):** `MainLayout.razor` — `GoToMyOrders` بيمرر `tokenCode` في الـ URL.

### 🎉 ملخص جلسة 9 نوفمبر 2026

**Sprint 3.6 اكتمل بنسبة 29%** (10/34 SP). المهام المؤجلة: `S3.6-H3`, `H4`, `H5`, `M1-M3`, `L1`.

**الإنجازات الرئيسية (في الجلسة دي):**
- ✅ دعوة المريض من الدكتور بقت بتشتغل (Fixed `EnrollUserInProgramAsync` + `ScanTokenPage`).
- ✅ تجديد التوكن + تأكيد الاستلام (Fixed `ShowRenewCard` + `ShowFollowUpButton` + `RenewToken`).
- ✅ "متابعة الطلب" بتفتح "طلباتي" (المريض) + تفاصيل الطلب مباشرة (Fixed `GoToMyOrders` + `HighlightTokenCode`).
- ✅ "متابعة الطلب" (قبل الموافقة) — `ShowFollowUpBeforeApproval`.
- ✅ Fix medicine guide navigation (Fixed `GoToMedicineGuide` — `Action<int, int>`).
- ✅ Fix `DailyDosesTab` (Row click + Info button).
- ✅ Fix `StartedJourney` (Android file upload + Validation).

**Lessons Learned:**
- **Lesson 19:** `PSPPatient` لازم يتعمل `INSERT` جديد لكل برنامج (مش `UPDATE` للقديم). الفلترة بـ `Participation.ProgramID` ضرورية.
- **Lesson 20:** `ScanTokenPage` — لازم نوقف الـ Scan بعد أول `ProcessCode` (مش `finally` — `PropertyChanged` listener).
- **Lesson 21:** `TokenStatus` — `APPROVED` و `DISPENSED` و `USED` كلهم محتاجين "متابعة الطلب" (مش بس `DISPENSED`).
- **Lesson 22:** `PatientOrdersPage` — `HighlightTokenCode` بقى يفتح تفاصيل الطلب مباشرة (لو جاي من `PspDetailComponent`).
- **Lesson 23:** `RenewToken` — لازم يمنع التجديد لو `TokenStatus = APPROVED` / `DISPENSED` (المريض لازم يأكد الاستلام الأول).
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
| 7: PWA | 10 | 10 | ~100% ✅ |
| 8: Menus & Services | 6 | 0 | 0% |
| 9: Patient Mobile (MAUI) | 8 | 8 | ~100% ✅ |
| 10: Deployment & Release | 4 | 1 | ~25% |
| 11: Personal Health Records (PHR) | 3 (Phases) | 0 | 0% |
| 12: Integration Hub | 3 (Phases) | 0 | 0% |
| 13: Rep (Medical Representative) | 4 (Phases) | 0 | 0% |
| 14: Dashboard Unification (PWA + MAUI) | 5 | 5 | ~100% ✅ |
---
تفاصيل Epic 14 (مكتمل):

✅ DashboardPage.razor (PWA) — مكتمل.

✅ PatientPointsDashboard (PWA + MAUI) — مكتمل.

✅ MedicineGuidePage (PWA) — مكتمل.

✅ DashboardFlowPage (MAUI) — مكتمل (النمط 7).

✅ DashboardPage.razor (MAUI) — مكتمل (بعد CurrentUserProfileState).

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

###🟡 Medium Debt ( تم حله)
| # | الدين | الموقع |
|---|---|---|
| TD-002 | ✅ حلقة إعادة التوجيه اللانهائية في cache-reset.html وخدمة Service Worker "الزومبي"|RubikCare.PWA |تم الحل: إضافة عداد محاولات (MAX_ATTEMPTS) مع استعادته بعد sessionStorage.clear()، واعتماد استراتيجية نشر تحافظ على ملفات .br و .gz المتطابقة. |

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
### v6.4 — 5 نوفمبر 2026

- ✅ **نجاح `DashboardFlowPage`** (الصفحة الرئيسية الجديدة) — `CurrentUserProfileState` + `<RootComponent>` في XAML (S3.5-H27).
- ✅ **`DashboardPage.razor` (المشترك)** — استخدام `CurrentUserProfileState` (بدلاً من `[Parameter] UserProfileId`) (S3.5-H28).
- ✅ **`OnboardingPage`** — قبل تسجيل الدخول (S3.5-H29).
- ✅ **`MedicineGuidePage`** — `CaseIdOverride` + `ProgramIdOverride` (S3.5-H30).
- ✅ **`PatientPointsFlow`** — تمرير `userProfileId` (S3.5-H31).
- ✅ **`DashboardNavigationBridge`** — `Navigate` + `OpenMenu` (S3.5-H32).
- ✅ **`13-MAUI.md` v4.0** — إعادة كتابة شاملة (S3.5-H33).
- ✅ **إصلاح `MainPage = shell` قبل `InitializeAsync`** (S3.5-H34).
- ✅ **إصلاح `has_seen_onboarding`** (S3.5-H35).
- ✅ **إضافة `Epic 14: Dashboard Unification`** (100% مكتمل).
- ✅ **تحديث `Epic 9: Patient Mobile`** (100% مكتمل).
- ✅ **إضافة "ملخص جلسة 5 نوفمبر 2026"** (نجاح `DashboardFlowPage`).
- ✅ **تحديث `13-MAUI.md` v4.0** + `01-ARCHITECTURE.md` v2.1 + `00-MASTER-INDEX.md` v2.3.
### v6.3 — 4 نوفمبر 2026

- ✅ **إعادة تصميم `DashboardPage.razor` (المشترك) — PWA:** Header بسيط + Stats Cards + Program Cards + Adherence Bar + Reminders (S3.5-H14).
- ✅ **ربط `DashboardPage.razor` بـ API الحقيقية (PWA):** `psp/entry` + `patient-details` + `patient-dashboard` + `schedules` (S3.5-H15).
- ✅ **إصلاح RTL/LTR في `DashboardPage.razor` (PWA):** `TranslationState` + `localStorage` (S3.5-H16).
- ✅ **إضافة `ProgramCardDto` (`CaseID` + `ParticipationId`):** Backend + PWA (S3.5-H17).
- ✅ **ربط `PatientPointsDashboard` (`/patient/points`):** PWA + MAUI (S3.5-H18).
- ✅ **إصلاح `MedicineGuidePage`:** استقبال `caseId` + `programId` (PWA) (S3.5-H19).
- ✅ **إنشاء `CurrentUserProfileState` (Singleton)** — غير مستخدم في MAUI (S3.5-H20).
- ✅ **إنشاء `DashboardNavigationBridge`** — مستخدم في PWA + MAUI (S3.5-H21).
- ✅ **العودة إلى `DashboardPage.xaml` المحلية (MAUI)** — بعد فشل استخدام `DashboardPage.razor` (المشترك).
- ❌ **استخدام `DashboardPage.razor` (المشترك) في MAUI:** فشل (تعارض `BlazorWebView` + `ShellContent`) — انظر الدروس 12-16.
- ⏳ **مؤجل:** تعديل `DashboardPage.xaml` لتطابق PWA (S3.5-H23)، القوائم الديناميكية (S3.5-H24)، تحويل `AppShell` (S3.5-H25)، إصلاح مفاتيح الترجمة (S3.5-H26).
- ✅ **إضافة الدروس 12-16** (Lessons Learned).
- ✅ **إضافة `S3.5-H22/H23/H24/H25/H26`** إلى Sprint 3.5.
###v6.2 — 31 أكتوبر 2026

- ✅ إغلاق مشاكل نشر PWA: تثبيت إعدادات البيئة (appsettings.Staging/Production)، حل حلقة cache-reset اللانهائية، وحماية web.config أثناء النشر.
- ✅ إضافة مهمة S3.5-H7 (تثبيت بيئة نشر PWA) كمكتملة.
- ✅ إضافة دين تقني جديد تم حله: TD-002 (حلقة الـ Cache Reset و Service Worker الزومبي).
- ✅ إضافة الدرس المستفاد رقم 11 (إدارة نشر PWA وحلقة الـ Cache Reset).
- ✅ تحديث حالة Epic 7 (PWA) إلى 100% مكتمل.

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

### درس 11: إدارة نشر PWA وحلقة الـ Cache Reset (31 أكتوبر 2026)
المشكلة:

1. عند نشر تحديثات PWA، كان المتصفح يتمسك بنسخة قديمة من service-worker.js مما يسبب أخطاء SRI ويدخل التطبيق في حلقة لا نهائية من صفحة cache-reset.html.
2. التعديل اليدوي لملفات .html أو .js مباشرة على السيرفر دون إعادة dotnet publish يكسر التطابق مع ملفات الضغط المضغوطة مسبقاً (.br و .gz) مما يسبب أخطاء 500.19.183 في IIS.
الحل المطبق:
1. إصلاح منطق cache-reset.html بإضافة MAX_ATTEMPTS واستعادة قيمة العداد فوراً بعد تنفيذ sessionStorage.clear().
2. اعتماد سكريبت نشر (Deploy-ToStage-PWA.ps1 v4.0) يقوم بـ:
- أخذ نسخة احتياطية كاملة قبل أي تغيير في C:\WebSite\_Docs\Backups\.
- استثناء ملف web.config وملفات appsettings.*.json من الحذف أو الاستبدال أثناء النسخ.
3.  التأكيد على أن أي تعديل على ملفات wwwroot يجب أن يتم محلياً ثم إعادة publish لضمان توليد ملفات الـ Hash والضغط بشكل متطابق.
  
الإجراء الوقائي:
- استخدام سكريبت النشر الآمن حصرياً.
- مسح بيانات الموقع (Clear site data) وإلغاء تسجيل Service Worker يدوياً من المتصفح عند اختبار إصدار جديد لأول مرة.

---
### درس 12: `BlazorWebView` + `ShellContent` = تعارض معماري في MAUI 10 (4 نوفمبر 2026)

**المشكلة:** محاولات متعددة لاستخدام `DashboardPage.razor` (المشترك) كصفحة رئيسية (`ShellContent` في `AppShell.xaml`) في MAUI فشلت كلها:
- `<RootComponent>` في XAML: `Parameters` ديناميكية لا تُمرَّر (`UserProfileId` = 0).
- `RootComponents.Add` في `OnAppearing`: `BlazorWebView` لا يعيد تحميل المكون بعد `Add`.
- `OnComponentReady`: `OnAfterRender` لا يُستدعى (لأن المكون لا يُحمَّل).
- `CurrentUserProfileState` (Singleton): نسختان (MAUI + BlazorWebView) — لا تتشاركان.
- `LoadingPage` → `DashboardPageFlow`: `Shell` لا يعيد توجيه `LoadingPage` (صفحة فرعية).

**السبب الجذري:** `BlazorWebView` في MAUI 10 يستخدم حاوية DI مستقلة (`13-MAUI` § 2.1)، ولا يدعم `Parameters` ديناميكية في `ShellContent`.

**الحل المعتمد:** العودة إلى `DashboardPage.xaml` المحلية (XAML) + `DashboardPageViewModel` (التي كانت تعمل).

**الإجراء الوقائي:**
- **الصفحات الرئيسية (`ShellContent`):** استخدم **XAML محلي** (وليس `BlazorWebView`).
- **الصفحات الفرعية (`Routing.RegisterRoute`):** يمكن استخدام `BlazorWebView` + `RootComponents.Add` في `Constructor` (مثل `PatientPointsFlow` — الذي يعمل).

**المرجع:** `13-MAUI` § 3.4، § 4.3، § 6.1.

---

### درس 13: Singleton Services لا تتشارك بين MAUI و `BlazorWebView` (4 نوفمبر 2026)

**المشكلة:** `CurrentUserProfileState` مسجل في `MauiProgram.cs` كـ `AddSingleton`. لكن `DashboardPage.xaml.cs` (MAUI) يحقن نسخة من حاوية MAUI، بينما `DashboardPage.razor` (BlazorWebView) يحقن نسخة من حاوية Blazor. `SetUserProfile(15, ...)` في MAUI لا يؤثر على `DashboardPage.razor` (الذي يرى `UserProfileId=0`).

**الإجراء الوقائي:** لا تعتمد على Singleton Services للتواصل بين MAUI و BlazorWebView. استخدم `RootComponent.Parameters` (للصفحات الفرعية) — أو `static` (لكن غير موصى به).

**المرجع:** `13-MAUI` § 2.1.

---

### درس 14: `OnAfterRender` في `DashboardPage.razor` لا يُستدعى عندما لا يُحمَّل المكون (4 نوفمبر 2026)

**المشكلة:** `OnAfterRender(firstRender)` (الذي يستدعي `OnComponentReady`) لم يُستدعى في Logcat — لأن `DashboardPage.razor` لم يكمل دورة حياته (بسبب `RootComponents.Add` بعد بدء `BlazorWebView`).

**الإجراء الوقائي:** لا تعتمد على `OnAfterRender` لاستدعاء `OnComponentReady` في `ShellContent`. استخدم `OnAppearing` في MAUI (إذا كان المكون قد تم تحميله).

**المرجع:** `13-MAUI` § 4.3.

---

### درس 15: `Console.WriteLine` في `BlazorWebView` يظهر في Logcat (وليس Output Window) (4 نوفمبر 2026)

**المشكلة:** `Console.WriteLine` في `DashboardPage.razor` (BlazorWebView) ظهر في **Logcat** (تحت وسم `DOTNET`)، وليس في **Output Window** (Visual Studio). هذا جعل التشخيص صعباً (كنا نبحث في Output Window دون جدوى).

**الإجراء الوقائي:**
- لتصحيح BlazorWebView، استخدم **Logcat** (`adb logcat | findstr "DOTNET"` أو Tools → Android → Device Log).
- استخدم `Console.WriteLine` (وليس `Debug.WriteLine`) — لأن `Debug.WriteLine` قد لا يظهر في BlazorWebView.

---

### درس 16: حالة الأحرف في DTOs (`ProgramID` وليس `PROGRAMID`) (4 نوفمبر 2026)

**المشكلة:** استخدمت `s.PROGRAMID` في `DashboardPage.razor`، لكن `MedicationScheduleDto` يحتوي على `ProgramID` (حالة أحرف مختلفة). هذا سبب خطأ `CS1061`.

**الإجراء الوقائي:** قبل استخدام أي DTO، تحقق من أسماء الحقول (بـ `Ctrl+Space` في Visual Studio، أو من ملف DTO نفسه).
---
### درس 17: `MainPage = shell` يجب أن يسبق `InitializeAsync` (5 نوفمبر 2026)

**المشكلة:** `App.xaml.cs` كان يستدعي `shell.InitializeAsync()` **قبل** `MainPage = shell`. `GoToAsync("//Onboarding")` تفشل (لأن `Shell` ليس `MainPage` بعد) → `catch` → `GoToAsync("//LoginPage")` (يفشل أيضاً) → `MainPage = shell` → `Shell` يعرض **أول `ShellContent`** (LoginPage) → ومضة `LoginPage` قبل `Onboarding`.

**الحل:** `MainPage = shell` **قبل** `InitializeAsync`:
```csharp
MainPage = shell;
await shell.InitializeAsync();
---
الإجراء الوقائي:
- في App.xaml.cs، عيّن MainPag = shell قبل InitializeAsync
- في InitializeAsync، استدعِ GoToAsync أولاً (بدون Task.Delay).
- Task.Delay (للتحديثات الخلفية) — بعد GoToAsync (في Task.Run).
---
### درس 18: has_seen_onboarding قد يكون true من محاولات سابقة (5 نوفمبر 2026)
**المشكلة**: InitializeAsync تفحص has_seen_onboarding — وإذا كان true (من محاولات سابقة)، تذهب إلى //LoginPage (بدلاً من //Onboarding).

الحل: Preferences.Remove("has_seen_onboarding") — مؤقتاً (للتجربة). أو — Preferences.Get("has_seen_onboarding", false) (القيمة الافتراضية false).

الإجراء الوقائي:
- في بيئة التطوير، استخدم Preferences.Remove("has_seen_onboarding") قبل InitializeAsync (للتجربة).
- في الإنتاج، لا تمسحها (لأنها تُعيَّن عند "تخطي" أو "ابدأ الآن").
- تأكد من أن has_seen_onboarding ليست true (من محاولات سابقة).
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
| 31 أكتوبر 2026 | v6.2 — حل  PWA + TD-002 تثبيت بيئة نشر + الدرس  11+ تحديث 7: PWA. |
| 5 نوفمبر 2026 | v6.4 — نجاح `DashboardFlowPage` (الصفحة الرئيسية الجديدة) + `DashboardPage.razor` + `OnboardingPage` + `MedicineGuidePage` + `PatientPointsFlow` + `13-MAUI.md` v4.0 + `01-ARCHITECTURE.md` v2.1 + `00-MASTER-INDEX.md` v2.3 + الدرسان 17, 18 + `Epic 14` (100%) + `Epic 9` (100%) |

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
