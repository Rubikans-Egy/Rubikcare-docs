
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
## 🔴 Sprint 1 — Points-PSP Integration (Planned)

### Epic: Points-PSP Integration Gap

**السياق:**
نظام النقاط مُصمم كـ Reusable Service. تكامل "استخدام النقاط" مع `PspDetail` غير مكتمل:
- لا تُنشأ `PSPProgramPointTransaction` عند الخصم.
- لا يوجد `ReferenceType` / `ReferenceID` يربط المعاملة بالـ Dispensation.
- `PspController.patientdetails.cs` يعتمد على `Description.Contains(TokenCode)` — هشّ.

**User Story #101 — تكامل نقاط الصرف (Backend)**
- **كـ** مريض، **أريد** استخدام نقاطي كخصم على الدواء عند تجديد التوكن، **لكي** أوفّر تكلفة الدواء.
- **Acceptance Criteria:**
  1. عند `POST /api/dispense/generate-next-token` مع `PointsToRedeem > 0`:
     - تُنشأ `PSPProgramPointTransaction` واحدة بـ `Points = -X`، `TransactionTypeID = REDEEM_DISPENSATION`، `ReferenceType = "DISPENSATION"`، `ReferenceID = newDispensation.DispensationID`.
     - يُحدّث `PSPProgramPointBalance.Balance -= X` و `TotalRedeemed += X`.
     - يُحدّث `PSPDispensation.PatientPayment` و `PSPCoverage` بناءً على الخصم.
     - يُمنع الخصم المزدوج لنفس Dispensation (Idempotency).
  2. `PspController.patientdetails.cs` يحسب `PointsUsedInCurrentToken` من `PSPProgramPointTransaction` (وليس من `Description`).
- **SP:** 5

**User Story #102 — زر "استخدم نقاطي" (Frontend)**
- **كـ** مريض، **أريد** زراً واضحاً **لكي** أعرف متى وكيف أستخدم نقاطي.
- **Acceptance Criteria:**
  1. الزر يظهر فقط إذا `TotalPoints > 0 && !PointsUsedInCurrentToken`.
  2. الزر **Disabled** إذا كان التوكن نشطاً ولم يُصرف (GENERATED/ACTIVE)، مع tooltip "اصرف التوكن الحالي أولاً".
  3. عند الضغط: Dialog يعرض الرصيد + قيمة الخصم المتوقعة.
  4. بعد النجاح: رسالة توضح "تم استخدام X نقطة، خصم Y ج.م".
  5. لا يوجد زر مكرر (حذف الزر من `psp-renew-card`).
- **SP:** 3

**User Story #103 — إصلاح `PspStatsBar` (Frontend)**
- **كـ** مريض، **أريد** رؤية إحصائيات حقيقية، **لكي** أعرف وضعي الفعلي.
- **Acceptance Criteria:** ربط `TotalPoints`، `TakenDosesToday`، `TotalDosesToday` من `_program` بدلاً من قيم ثابتة.
- **SP:** 1

**User Story #104 — الترجمة (Backend/DB)**
- **كـ** مطور، **أريد** مفاتيح ترجمة كاملة، **لكي** لا تظهر مفاتيح خام.
- **Acceptance Criteria:** إضافة `PSP.POINTS.REDEEM.*`، `PSP.STATS.*`، `PSP.POINTS.DASHBOARD.*`، `COMMON.CANCEL`، `COMMON.OK`، `COMMON.PLEASE_WAIT`، `PSP.STATUS.*`، `PSP.FORM.EGP` عبر `MERGE`.
- **SP:** 1

**User Story #105 — توحيد CSS (Refactoring)**
- **كـ** مطور، **أريد** تنظيم CSS، **لكي** يسهل الصيانة.
- **Acceptance Criteria:**
  1. إنشاء `_shared/dialogs.css` ببادئة `rubik-` (نقل `.psp-dialog-*` و `.pdash-dialog-*` إليها).
  2. إضافة تعليق مسار CSS في رأس كل `.razor` (نمط موحّد).
  3. لا تكرار بين `psp-detail.css` و `point-dashboard.css`.
- **SP:** 2

### Epic: Technical Debt (مضافة)

- **TD-01**: `PspController.patientdetails.cs` — إزالة الاعتماد على `Description.Contains(TokenCode)`.
- **TD-02**: `PspDetailComponent.razor` — إزالة `ToString("N0").100.00` (خطأ الصيغة).
- **TD-03**: `Resources` — توحيد حالة الأحرف (`Common.*` vs `COMMON.*`).
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
