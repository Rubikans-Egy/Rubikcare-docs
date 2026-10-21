
# 00B — PROJECT-BACKLOG

**الاسم:** PROJECT-BACKLOG — سجل التطوير الحي
**الإصدار:** 1.0
**آخر تحديث:** 20 أكتوبر 2026
**Sprint الحالي:** Sprint 0 — Foundation
**الحالة:** 🟢 حي — يُحدَّث كل جلسة
**الجمهور:** فريق التطوير + AI Assistants

---

## 📌 مقدمة

هذه الوثيقة هي **المصدر الوحيد للحقيقة** لكل ما يتعلق بمهام التطوير في RubikCare.

**تدمج:**
- **`18-RELEASE-ROADMAP-DASHBOARD`** (مهام P1-P4).
- **`23-TECHNICAL-DEBT`** (الديون التقنية).
- **`Changelog`** (سجل التغييرات).
- **`TASK-LOG`** (سجل الأحداث — Sprint Events).

**الاستخدام:**
1. **قبل كل جلسة:** تُشارَك مع AI ليعرف السياق.
2. **بعد كل مهمة:** تُحدَّث الحالة.
3. **في Sprint Planning:** تُخطَّط المهام الجديدة.
4. **في Sprint Review:** تُراجَع الإنجازات.

**الأساس المنهجي:**
- **Scrum Methodology** (Sprint أسبوعي).
- **Story Points** (1, 2, 3, 5, 8, 13, 21).
- **User Stories** (كـ [دور]، أريد [هدف]، لكي [قيمة]).
- **Board Columns** (9 حالات).

---

## 🎯 الرؤية الحالية

RubikCare في مرحلة **Pre-Launch**. الهدف الحالي:
- إكمال Backend + Frontend الأساسي.
- إصدار Production مستقر.
- تأسيس Schema للأنظمة الكبرى (Permissions, Subscriptions).

---

## 📊 Sprint 0 — Foundation

**الهدف:** تأسيس الأساسات للتنفيذ المُنظَّم
**المدة:** 16 - 22 أكتوبر 2026 (7 أيام)
**المُخطَّط:** 22 Story Point
**الفِعلي:** (سيُحدَّث)

---

### ✅ Done

| # | المهمة | SP | المُسند | تاريخ الإنجاز | ملاحظات |
|:-:|:---|:---:|:---:|:---:|:---|
| 1 | تحديث 09-API-Guide v2.0 | 5 | AI | 16 أكتوبر | 39 Controller + 229 Endpoint |
| 2 | كتابة 03-MENUS v4.0 | 5 | AI | 16 أكتوبر | مع النصوص الحرفية + القرارات |
| 3 | تجربة السكريبت التوليدي | 1 | User + AI | 16 أكتوبر | نجح — 3 ملفات |

**المجموع:** 11 SP

---

### 🟡 In Progress

| # | المهمة | SP | المُسند | البداية | ETA |
|:-:|:---|:---:|:---:|:---:|:---:|
| 4 | إنشاء 00B-PROJECT-BACKLOG | 2 | AI | 16 أكتوبر | 16 أكتوبر |

**المجموع:** 2 SP

---

### ⏳ Pending

| # | المهمة | SP | المُسند | ETA | الاعتماد |
|:-:|:---|:---:|:---:|:---:|:---|
| 5 | تحسين السكريبت v2 | 2 | AI | 17 أكتوبر | — |
| 6 | كتابة 19-PERMISSIONS | 5 | AI | 18 أكتوبر | — |
| 7 | Migration 1: CustomJobTitleId في MenuVisibilityRules | 1 | AI | 19 أكتوبر | #6 |
| 8 | Migration 2: OrganizationSubscription table | 1 | AI | 19 أكتوبر | #6 |
| 9 | Migration 3: ServiceProductID في MenuItems | 1 | AI | 19 أكتوبر | #6 |
| 10 | Migration 4: ControlLevel + Permission | 2 | AI | 20 أكتوبر | #6 |
| 11 | Migration 5: UserControlLevel + JobTitleControlLevel | 2 | AI | 20 أكتوبر | #6 |

**المجموع:** 14 SP

---

### 🚫 Blocked

| # | المهمة | السبب |
|:-:|:---|:---|
| — | — | — |

---

### 🔄 Rework

| # | المهمة | السبب |
|:-:|:---|:---|
| — | — | — |

---

## 📅 Sprint History

### Sprint 1 — Points-PSP Integration (20-22 أكتوبر 2026) 🔴 Critical

**الهدف:** سد الفجوات بين نظام النقاط و PSP والتقارير — معالجة 9 عقود تكامل (IC-01 → IC-09).

**السياق:**
جلسة #136 كشفت أن:
- نظام النقاط Backend كامل (40 Handler) لكن غير مُستدعى بشكل صحيح من `DispenseController`.
- التقارير لا تعرف عن نظام النقاط أصلاً.
- `GenerateNextToken` به 6 فجوات حرجة.
- `ConfirmDispense` يتجاهل الخصم.
- زر "استخدم نقاطي" لا يخصم فعلياً (0 معاملات REDEEM_DISPENSATION).

**المهام:**

| # | المهمة | SP | الحالة |
|---|--------|----|----|
| P1 | توثيق Integration Contracts في `01-ARCHITECTURE.md` | 2 | 🟡 In Progress |
| P2.1 | Migration: `REDEEM_DISPENSATION` (ID=9) | 1 | ⏳ Pending |
| P2.2 | إعادة كتابة `GenerateNextToken` (ذرية + Idempotency + ReferenceType) | 5 | ⏳ Pending |
| P2.3 | إصلاح `ConfirmDispense` (احترام الخصم) | 3 | ⏳ Pending |
| P2.4 | إصلاح `PspController.patientdetails.cs` | 2 | ⏳ Pending |
| P2.5 | إضافة `MonetaryValuePerPoint` إلى DTO | 1 | ⏳ Pending |
| P3.1 | `PspDetailComponent.razor` (Disabled + Dialog + Success) | 3 | ⏳ Pending |
| P3.2 | CSS: `_shared/dialogs.css` + `.psp-redeem-card` | 2 | ⏳ Pending |
| P3.3 | إصلاح `PspStatsBar` | 1 | ⏳ Pending |
| P4.1 | Idempotency في `TryAwardPointsViaOrchestratorAsync` | 2 | ⏳ Pending |
| P4.2 | Idempotency في `AcknowledgeReminderHandler` | 1 | ⏳ Pending |
| P5.1 | `GetAdherenceHandler` — إضافة Points | 2 | ⏳ Pending |
| P5.2 | `GetPharmacyFinancialHandler` — فصل PointsDiscount | 3 | ⏳ Pending |
| P5.3 | `GetProgramLifecycleHandler` — إضافة Points | 2 | ⏳ Pending |
| P6.1 | إشعار عند كسب/خصم نقاط | 2 | ⏳ Pending |
| P7.1 | الترجمة (SQL MERGE) | 1 | ⏳ Pending |
| P7.2 | اختبار E2E | 3 | ⏳ Pending |

**المجموع:** 36 SP

**Acceptance Criteria:**
- [ ] كل عقد (IC-01 → IC-09) موثق في `01-ARCHITECTURE.md`.
- [ ] `GenerateNextToken` يُنشئ `PSPProgramPointTransaction` صحيحة مع `ReferenceType="DISPENSATION"` + `ReferenceID=DispensationID`.
- [ ] `ConfirmDispense` يحتفظ بالخصم (لا يعيد حسابه).
- [ ] زر "استخدم نقاطي" يعمل: Disabled إذا توكن جديد، Dialog إذا مصروف، Success message بعد النجاح.
- [ ] التقارير تُظهر النقاط (Adherence + Financial + Lifecycle).
- [ ] 0 معاملات `REDEEM_DISPENSATION` → بعد الاختبار: N معاملة (بحسب السيناريوهات).
- [ ] `ProgramLifecycle` لا يحتوي على "لا شيء عن Points".


### Sprint 0 — Foundation (16-22 أكتوبر 2026)

- **الهدف:** تأسيس الأساسات
- **المُخطَّط:** 22 SP
- **الفِعلي:** (سيُحدَّث)

---

## 📈 Velocity Chart

| Sprint | Planned | Actual | Ratio |
|:---:|:---:|:---:|:---:|
| 0 | 22 | — | — |

**Velocity المتوسط:** (سيُحسب بعد Sprint 0)

---

## 🎯 Epic Overview

### Epic 1: Foundation & Tooling
- **User Stories:** 5
- **Done:** 2
- **In Progress:** 1
- **Backlog:** 2
- **Completion:** ~40%

**الوصف:** تأسيس الأدوات (Scripts) + الوثائق الأساسية + Backlog.

---

### Epic 2: Permissions System
- **User Stories:** 4
- **Done:** 0
- **In Progress:** 0
- **Backlog:** 4
- **Completion:** 0%

**الوصف:** نظام صلاحيات كامل (RBAC + ABAC) — Vertical Slice مستقل.

---

### Epic 3: PSP System
- **User Stories:** 15
- **Done:** 13
- **In Progress:** 1
- **Backlog:** 1
- **Completion:** ~85%

**الوصف:** نظام برامج دعم المرضى — Backend + Frontend.

---

### Epic 4: Points System
- **User Stories:** 12
- **Done:** 10
- **In Progress:** 1
- **Backlog:** 1
- **Completion:** ~80%

**الوصف:** نظام النقاط + Rubika Currency.

---

### Epic 5: Reports System
- **User Stories:** 8
- **Done:** 7
- **In Progress:** 0
- **Backlog:** 1
- **Completion:** ~85%

**الوصف:** نظام التقارير (6 تقارير + Dashboard).

---

### Epic 6: Notifications System
- **User Stories:** 6
- **Done:** 5
- **In Progress:** 0
- **Backlog:** 1
- **Completion:** ~80%

**الوصف:** 3 طبقات إشعارات (FCM + WebPush + Local).

---

### Epic 7: PWA
- **User Stories:** 10
- **Done:** 8
- **In Progress:** 1
- **Backlog:** 1
- **Completion:** ~85%

**الوصف:** نسخة كربونية من Mobile.

---

### Epic 8: Menus & Services Integration
- **User Stories:** 6
- **Done:** 0
- **In Progress:** 0
- **Backlog:** 6
- **Completion:** 0%

**الوصف:** الربط بين Services + Menus + Subscriptions.

---

### Epic 9: Patient Mobile
- **User Stories:** 3
- **Done:** 2
- **In Progress:** 0
- **Backlog:** 1
- **Completion:** ~65%

**الوصف:** إكمال واجهة المريض (Mobile + Token).

---

### Epic 10: Deployment & Release
- **User Stories:** 4
- **Done:** 1
- **In Progress:** 0
- **Backlog:** 3
- **Completion:** ~25%

**الوصف:** إصدار Production + سكريبتات النشر.

---

## 📝 Sprint Events Log

### Sprint Planning — Sprint 0 (16 أكتوبر 2026)

- **الحضور:** User + AI
- **المُنجَز:** تحديد 7 مهام أساسية — 22 SP.
- **المخاطر:**
  - مساحة C: محدودة → الحل: `E:\Docs`.
  - المستودع يحتاج تنظيماً جديداً.

### Sprint Review — Sprint 0

- **التاريخ:** (سيُحدَّث بعد 22 أكتوبر)
- **المُنجَز:** (سيُحدَّث)
- **التغذية الراجعة:** (سيُحدَّث)

### Sprint Retrospective — Sprint 0

- **التاريخ:** (سيُحدَّث)
- **ما نجح:** (سيُحدَّث)
- **ما يحتاج تحسيناً:** (سيُحدَّث)
- **الإجراءات:** (سيُحدَّث)

---

## 🎯 Sprint 1 — Planned

**الهدف:** إكمال واجهة المريض + الإصدار
**المدة:** 23 - 29 أكتوبر 2026
**المُخطَّط المُتوقَّع:** 20 SP

**المهام المتوقعة:**

| # | المهمة | SP |
|:-:|:---|:---:|
| 1 | إكمال Patient Mobile (Mobile + Token) | 5 |
| 2 | اختبار E2E لنظام النقاط | 3 |
| 3 | إصلاح أخطاء P1 (من Technical Debt) | 5 |
| 4 | اختبار Production Readiness | 3 |
| 5 | إصدار Production | 4 |

---

## 📋 Product Backlog (المستقبلي)

### 🔴 High Priority

| # | User Story | Epic | SP |
|:-:|:---|:---:|:---:|
| 20 | كـ مريض، أريد رؤية رصيد النقاط في Mobile | Points | 3 |
| 21 | كـ Admin، أريد إدارة Control Levels | Permissions | 5 |
| 22 | كـ Admin، أريد ربط Users بـ Control Levels | Permissions | 3 |
| 23 | كـ Super Admin، أريد إدارة Permissions | Permissions | 5 |
| 24 | كـ Admin، أريد إدارة اشتراك المنظمة | Subscriptions | 5 |
| 25 | كـ Admin، أريد ربط SubscriptionPlan بـ Menus | Menus | 5 |

---

### 🟠 Medium Priority

| # | User Story | Epic | SP |
|:-:|:---|:---:|:---:|
| 30 | كـ مستخدم، أريد Menus ديناميكية في PWA | Menus | 5 |
| 31 | كـ مستخدم، أريد Menus ديناميكية في MAUI | Menus | 5 |
| 32 | كـ Admin، أريد ربط ServiceProduct بـ MenuItems | Menus | 3 |
| 33 | كـ مستخدم، أريد ApiClient موحّد | Architecture | 8 |
| 34 | كـ مطور، أريد API-Contracts | Architecture | 5 |

---

### 🟡 Low Priority

| # | User Story | Epic | SP |
|:-:|:---|:---:|:---:|
| 40 | كـ Super Admin، أريد إدارة OrganizationGroups | Organization | 5 |
| 41 | كـ منصة، أريد Rubika Currency Engine | Rubika | 8 |
| 42 | كـ مريض، أريد Deep Linking من QR | Mobile | 3 |

---

### ⏸️ Deferred

| # | المهمة | السبب |
|:-:|:---|:---|
| 50 | GitHub Project Board Integration | معقد — يؤجّل |
| 51 | Source Generator للـ ApiClient | بعد الأساسيات |
| 52 | B2B Supply Chain | Phase 4 |

---

## 🏗️ Technical Debt Register

> **ملاحظة:** هذا القسم يجمع الديون التقنية — التي هي مهام صغيرة لا تستحق Sprint منفصل.
> **المصدر:** دمج `23-TECHNICAL-DEBT`.

### 🔴 Critical Debt (Before Production)

| # | الدين | الموقع | الأولوية |
|:-:|:---|:---|:---:|
| 1 | انتهاك معماري: `@page` في Shared.UI | `Shared.UI/Components/Pharmacy/PharmacyDetailPage.razor` + `PSP/PspAboutPage.razor` | 🔴 |
| 2 | خلط Namespaces | `PharmacySearchPage.razor` + `PharmacyDetailPage.razor` | 🔴 |
| 3 | 404 في `/admin/pending-licenses` | `SystemManagment` vs `SystemManagement` | 🔴 |

### 🟡 Medium Debt

| # | الدين | الموقع |
|:-:|:---|:---|
| 4 | زر البحث في Dashboard غير مربوط | `Pages/Dashboard.razor` |
| 5 | ازدواجية `FirebaseNotificationService` | `Api.Web` + `Infrastructure` |
| 6 | ازدواجية `IPSPPointSchemeService` / `IPointSchemeClientService` | `Application` + `Web` |
| 7 | عدم تحديث `PROJECT-BACKLOG` أسبوعياً | عملية |

### 🟢 Low Debt

| # | الدين | الموقع |
|:-:|:---|:---|
| 8 | اسم خدمة مضلل `IMobileNavigationService` | `Shared.UI/Services/` |
| 9 | `HttpUtility` في Blazor WASM | `PharmacySearch.razor` |
| 10 | `alert()` بدلاً من Toast | `PspSearch.razor` |
| 11 | `SystemMenuMenuID` Shadow Property | `MenuAssignments` |
| 12 | `RoleId` في `MenuVisibilityRule` | بعد Migration |

### ⏸️ Deferred Debt

| # | الدين | السبب |
|:-:|:---|:---|
| 13 | Messaging Hub مخفي | غير مكتمل |
| 14 | Deep Linking من QR في MAUI | مؤجل |

---

## ✅ ما تم إنجازه (History)

> **ملاحظة:** هذا القسم يجمع كل المهام المُنجَزة في Sprints سابقة.
> **المصدر:** دمج `23-TECHNICAL-DEBT` (الجزء المُنجَز) + `Changelog`.

### Sprint سابق (8 سبتمبر 2026)

| # | المهمة | الحل |
|:-:|:---|:---|
| 1 | توجيه "Next Refill" الخاطئ | تعديل `OrderTracker` |
| 2 | حالة الطلب `???` | `GetStatusTextArabic` |
| 3 | صفحة "دعواتي" | إعادة تصميم |
| 4 | "أقرب الصيدليات" | دمج `OsmPharmacyService` |
| 5 | كاميرا QR في PWA | `QrScannerComponent` |
| 6 | بوابة الصيدلية | Endpoint جديد في `PharmacyController` |

### Sprint سبتمبر 2026

| # | المهمة | الحل |
|:-:|:---|:---|
| 7 | المندوب لا يرى برامج شركته | `companyId` parameter |
| 8 | خصومات الميزانية لا تظهر | `PSPStep2_Budget.razor` |
| 9 | حفظ الأدوية بـ `MedicationID = 0` | `OriginalMedicationID ?? PharmaCompanyMedicationID` |
| 10 | `Unsubscribe` endpoint مفقود | `PspController.Unsubscribe.cs` |
| 11 | `PharmaCompanyDashboard` لا يتحدث | `///` في `GoToAsync` |
| 12 | `GetDashboardStats` يعرض أرقام كل الشركات | فلترة بـ `companyProgramIds` |
| 13 | `MyInvitations` و `MyNetwork` بدون فلترة | إضافة `companyId` |

### Sprint أكتوبر 2026

| # | المهمة | الحل |
|:-:|:---|:---|
| 14 | نظام النقاط (كامل) | 17 يوم — Backend |
| 15 | توسعة النقاط (M:N) | 10 أيام |
| 16 | نظام التقارير (كامل) | يوم واحد |
| 17 | Web Push Notifications | Service Worker |
| 18 | تحديث 09-API-Guide v2.0 | 39 Controller + 229 Endpoint |
| 19 | كتابة 03-MENUS v4.0 | مع النصوص الحرفية |
| 20 | السكريبتات التوليدية | 3 سكريبتات |

---

## 📝 Changelog (مختصر)

> **ملاحظة:** هذا القسم يجمع أهم التغييرات.
> **المصدر:** دمج `Changelog`.

### v1.0 — 16 أكتوبر 2026

- إنشاء `00B-PROJECT-BACKLOG` (دمج 18 + 23 + Changelog).
- إعادة تنظيم الوثائق (34 → 20).
- تحديث `09-API-GUIDE` v2.0.
- كتابة `03-MENUS` v4.0.

### v0.9 — 13 أكتوبر 2026

- نظام النقاط v6.0 (Campaign Reference Architecture).
- Migration `UseSourceEntityIdInPointSchemes`.

### v0.8 — 10 أكتوبر 2026

- تحديث `08-Strategic-Roadmap` v5.0.
- إضافة Reusable Services.

### v0.7 — 3 أكتوبر 2026

- إضافة نظام النقاط (Backend).
- إضافة PWA رسمياً (9 مشاريع).

### v0.6 — 23 سبتمبر 2026

- نظام التقارير v1.0 (6 تقارير + Dashboard).
- Technical Debt (13 منجز).

### v0.5 — 29 أغسطس 2026

- Deploy PWA.
- تحديث Deployment Guide.

---
## 🔴 Sprint 1 — Points-PSP Integration Fix (Critical)

### Epic: Points-PSP Integration Gap

**السياق المكتشف (جلسة #136):**
بعد تحليل `DispenseController` والبيانات، تبيّن أن تكامل "استخدام النقاط" مع تجديد التوكن **يعمل جزئياً** لكن به 5 ثغرات خطيرة:

| # | الثغرة | التأثير |
|---|--------|---------|
| 1 | `TransactionTypeID = 2` (REDEEM_REQUEST) بدلاً من نوع مخصص | تقارير خاطئة، خلط مع طلبات الاسترداد |
| 2 | لا `ReferenceType` / `ReferenceID` | لا تتبع أي Dispensation |
| 3 | `ConfirmDispense` يعيد حساب `PatientPayment` ويتجاهل الخصم | **ضياع الخصم المالي** |
| 4 | `balance` بدون `SchemeID` | خصم من Scheme خاطئ |
| 5 | لا Idempotency | خصم متكرر ممكن |
| 6 | `SaveChangesAsync` مرتين (غير ذري) | قد تُخصم النقاط دون تحديث Dispensation |
| 7 | `GenerateNextToken` لا يُعيّن `TotalAmount` / `PatientPayment` | التوكن يظهر NULL |

### User Story #201 — إضافة نوع معاملة مخصص (Backend)
- **كـ** مسؤول مالي، **أريد** TransactionTypeID مخصص للخصم على الصرف، **لكي** تكون التقارير دقيقة.
- **Acceptance Criteria:**
  - إضافة `REDEEM_DISPENSATION` (ID=9) إلى `PSPProgramPointTransactionTypes`.
  - تحديث `DispenseController` لاستخدامه.
- **SP:** 1

### User Story #202 — ربط المعاملة بالـ Dispensation (Backend)
- **Acceptance Criteria:**
  - `PSPProgramPointTransaction.ReferenceType = "DISPENSATION"`.
  - `PSPProgramPointTransaction.ReferenceID = newDispensation.DispensationID`.
  - تحديث `PspController.patientdetails.cs` ليعتمد على `ReferenceType/ReferenceID` بدلاً من `Description.Contains`.
- **SP:** 2

### User Story #203 — إصلاح `ConfirmDispense` (Backend) 🔴
- **Acceptance Criteria:**
  - `ConfirmDispense` **لا يعيد حساب** `PatientPayment` إذا كانت `PointsToRedeem > 0` سابقاً.
  - أو: يقرأ `PatientPayment` المخزّن ويستخدمه.
- **SP:** 2

### User Story #204 — Idempotency (Backend) 🔴
- **Acceptance Criteria:**
  - قبل الخصم: `AnyAsync(t => t.ReferenceType == "DISPENSATION" && t.ReferenceID == newDispensation.DispensationID)`.
  - إذا وُجد → لا خصم جديد.
- **SP:** 1

### User Story #205 — تحديد `SchemeID` من `ProgramID` (Backend) 🔴
- **Acceptance Criteria:**
  - `balance` يُفلتر بـ `SchemeID` المشتق من `SourceEntityID = request.ProgramId && PointSystemTypeID = 1`.
  - إذا لا يوجد Scheme → لا خصم (رسالة واضحة).
- **SP:** 1

### User Story #206 — ذرية (Atomicity) — Transaction Scope (Backend)
- **Acceptance Criteria:**
  - خصم النقاط + تحديث Dispensation + إنشاء Transaction **في Context واحد** (`ExecuteWithNewContextAsync` فقط).
  - إذا فشل أي جزء → rollback.
- **SP:** 2

### User Story #207 — تعيين `TotalAmount` / `PatientPayment` عند التوليد
- **Acceptance Criteria:**
  - `GenerateNextToken` يحسب السعر المبدئي (`unitPrice × quantity`) ويملأ `TotalAmount`.
  - `PatientPayment` يُحسب بعد تطبيق `PSPCoverage` + خصم النقاط.
- **SP:** 2

### User Story #208 — زر "استخدم نقاطي" (Frontend)
- كما في الخطة السابقة (Disabled + Dialog + Success Message + إصلاح صيغة).
- **SP:** 3

### User Story #209 — إصلاح `PspStatsBar`
- ربط `TotalPoints` / `TakenDosesToday` / `TotalDosesToday`.
- **SP:** 1

### User Story #210 — الترجمة
- إضافة `PSP.POINTS.REDEEM.*`, `PSP.STATS.*`, `COMMON.CANCEL`, `COMMON.OK`, `PSP.FORM.EGP`, `PSP.STATUS.*`.
- **SP:** 1

### User Story #211 — توحيد CSS
- `_shared/dialogs.css`, تعليقات المسارات، لا تكرار.
- **SP:** 2

**المجموع: 18 SP**
- 
## 📝 سجل التحديثات

| التاريخ | التحديث |
|:---:|:---|
| 16 أكتوبر 2026 | إنشاء الوثيقة — دمج 18 + 23 + Changelog |
| 20 أكتوبر 2026 |oints-PSP Integration (Planned)  Changelog |

---

## 🔗 روابط ذات صلة

- [00 - MASTER-INDEX](./00-MASTER-INDEX.md)
- [01 - ARCHITECTURE](./01-ARCHITECTURE.md)
- [03 - MENUS](./03-MENUS.md)
- [09 - API-GUIDE](./09-API-GUIDE.md)
- [19 - PERMISSIONS](./19-PERMISSIONS.md)

---

**© 2026 RubikCare — للاستخدام الداخلي**

**نهاية الوثيقة ✅**
```
