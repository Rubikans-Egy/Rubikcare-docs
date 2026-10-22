
# 00B — PROJECT-BACKLOG

الاسم: PROJECT-BACKLOG — سجل التطوير الحي
الإصدار: 2.0 (نهاية Sprint 1)
آخر تحديث: 23 أكتوبر 2026
Sprint الحالي: مكتمل — Sprint 1 (Points-PSP Integration)
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

---

## 📊 Sprint 1 — Points-PSP Integration (22-23 أكتوبر 2026) — ✅ مكتمل

**الهدف:** إغلاق الفجوة بين نظام النقاط و PSP (9 عقود تكامل — IC-01 → IC-09).

**السياق:** جلسة #136 كشفت أن نظام النقاط Backend كامل لكن غير مُستدعى بشكل صحيح من `DispenseController`.

### ✅ Done (12 مهمة)

| # | المهمة | SP | الحالة | ملاحظات |
|---|--------|----|--------|---------|
| P1 | توثيق Integration Contracts | 2 | ✅ | موثق في 01-ARCHITECTURE |
| P2.1 | Migration: REDEEM_DISPENSATION (ID=9) | 1 | ✅ | مطبق في DB |
| P2.2 | إعادة كتابة GenerateNextToken (ذرية) | 5 | ✅ | Transaction + Idempotency |
| P2.3 | إصلاح ConfirmDispense (احترام الخصم) | 3 | ✅ | يقرأ PointsDiscount |
| P2.4 | إصلاح PspController.patientdetails | 2 | ✅ | + MonetaryValuePerPoint |
| P2.5 | إضافة MonetaryValuePerPoint إلى DTO | 1 | ✅ | في PatientProgramDetailsResponse |
| P3.1 | PspDetailComponent.razor (Dialog + Success) | 3 | ✅ | يعمل |
| P3.2 | CSS: _shared/dialogs.css | 2 | ✅ | .psp-redeem-card |
| P3.3 | إصلاح PspStatsBar | 1 | ✅ | بيانات حقيقية |
| P4.1 | Idempotency في TryAwardPointsViaOrchestrator | 2 | ✅ | ReferenceType + ReferenceID |
| P4.2 | Idempotency في AcknowledgeReminderHandler | 1 | ✅ | — |
| P6.1 | إشعار عند كسب/خصم نقاط | 2 | ✅ | WebPush + Notification |

### ✅ Done (إضافي — خارج Sprint 1)

| # | المهمة | SP | ملاحظات |
|---|--------|----|---------|
| D1 | إصلاح `IRefreshService` في PWA | 1 | إنشاء Shared.UI/Services/RefreshService.cs |
| D2 | إصلاح فقدان ربط PSP بعد Migration (22 حالة) | 3 | إعادة إدراج PSPPatients + تحديث PSPeRX |
| D3 | إصلاح حساب النقاط (PatientProfileID بدلاً من CaseID) | 2 | PointsContext + GenerateNextToken |
| D4 | إصلاح عرض الخصم للصيدلي (PointsDiscount) | 2 | ValidateTokenResponse + PharmacyScanTokenPage |
| D5 | إصلاح بطاقة "لا يوجد توكن" + عرض التوكن الجديد | 1 | PspDetailComponent + PspController |
| D6 | إصلاح تجاوب BarcodeScanner (CSS) | 1 | .manual-input-row |

**المجموع:** 36 SP (Sprint 1) + 10 SP (إضافي) = **46 SP**

### ⏳ Pending (المتبقي — Sprint 1)

| # | المهمة | SP | الأولوية |
|---|--------|----|----------|
| P5.1 | GetAdherenceHandler — إضافة Points | 2 | 🟠 |
| P5.2 | GetPharmacyFinancialHandler — فصل PointsDiscount | 3 | 🟠 |
| P5.3 | GetProgramLifecycleHandler — إضافة Points | 2 | 🟠 |
| P7.1 | الترجمة (SQL MERGE) — بعض المفاتيح | 1 | 🟡 |
| P7.2 | اختبار E2E كامل (سيناريوهات متعددة) | 3 | 🟠 |

**المجموع المتبقي:** 11 SP

---

## 🎯 Sprint 0 — Foundation (16-22 أكتوبر 2026) — ⏳ In Progress

**الهدف:** تأسيس الأساسات للتنفيذ المُنظَّم.
**المُخطَّط:** 22 SP
**الفِعلي:** 11 SP

### ✅ Done

| # | المهمة | SP | تاريخ الإنجاز |
|---|--------|----|---------------|
| 1 | تحديث 09-API-Guide v2.0 | 5 | 16 أكتوبر |
| 2 | كتابة 03-MENUS v4.0 | 5 | 16 أكتوبر |
| 3 | تجربة السكريبت التوليدي | 1 | 16 أكتوبر |

### ⏳ Pending

| # | المهمة | SP | ETA |
|---|--------|----|-----|
| 5 | تحسين السكريبت v2 | 2 | — |
| 6 | كتابة 19-PERMISSIONS | 5 | — |
| 7-11 | 5 Migrations (ControlLevel, Permissions, ...) | 7 | — |

**المجموع المتبقي:** 14 SP

---

## 📈 Velocity Chart

| Sprint | Planned | Actual | Ratio |
|--------|---------|--------|-------|
| Sprint 1 | 36 | 36 | 100% ✅ |
| Sprint 0 | 22 | 11 | 50% 🟡 |

---

## 🎯 Epic Overview

| Epic | User Stories | Done | Completion |
|------|--------------|------|------------|
| 1: Foundation & Tooling | 5 | 2 | ~40% |
| 2: Permissions System | 4 | 0 | 0% |
| 3: PSP System | 15 | 14 | ~93% |
| **4: Points System** | **12** | **12** | **~100% ✅** |
| 5: Reports System | 8 | 7 | ~85% |
| 6: Notifications System | 6 | 6 | ~100% ✅ |
| 7: PWA | 10 | 9 | ~90% |
| 8: Menus & Services | 6 | 0 | 0% |
| 9: Patient Mobile | 3 | 2 | ~65% |
| 10: Deployment & Release | 4 | 1 | ~25% |

---

## 📋 Product Backlog (المستقبلي)

### 🔴 High Priority

| # | User Story | Epic | SP |
|---|------------|------|----|
| 20 | كـ مريض، أريد رؤية رصيد النقاط في Mobile | Points | 3 |
| 21 | كـ Admin، أريد إدارة Control Levels | Permissions | 5 |
| 22 | كـ Admin، أريد ربط Users بـ Control Levels | Permissions | 3 |
| 23 | كـ Super Admin، أريد إدارة Permissions | Permissions | 5 |
| 24 | كـ Admin، أريد إدارة اشتراك المنظمة | Subscriptions | 5 |
| 25 | كـ Admin، أريد ربط SubscriptionPlan بـ Menus | Menus | 5 |

### 🟠 Medium Priority

| # | User Story | Epic | SP |
|---|------------|------|----|
| 30 | Menus ديناميكية في PWA | Menus | 5 |
| 31 | Menus ديناميكية في MAUI | Menus | 5 |
| 32 | ربط ServiceProduct بـ MenuItems | Menus | 3 |
| 33 | ApiClient موحّد | Architecture | 8 |
| 34 | API-Contracts | Architecture | 5 |

### 🟡 Low Priority

| # | User Story | Epic | SP |
|---|------------|------|----|
| 40 | إدارة OrganizationGroups | Organization | 5 |
| 41 | Rubika Currency Engine | Rubika | 8 |
| 42 | Deep Linking من QR | Mobile | 3 |

---

## 🏗️ Technical Debt Register

### 🔴 Critical Debt (Before Production)

| # | الدين | الموقع | الأولوية |
|---|-------|--------|----------|
| 1 | انتهاك معماري: @page في Shared.UI | PharmacyDetailPage.razor + PspAboutPage.razor | 🔴 |
| 2 | خلط Namespaces | PharmacySearchPage.razor | 🔴 |
| 3 | 404 في /admin/pending-licenses | SystemManagment vs SystemManagement | 🔴 |

### 🟡 Medium Debt

| # | الدين | الموقع |
|---|-------|--------|
| 4 | زر البحث في Dashboard غير مربوط | Pages/Dashboard.razor |
| 5 | ازدواجية FirebaseNotificationService | Api.Web + Infrastructure |
| 6 | ازدواجية IPSPPointSchemeService | Application + Web |
| 7 | عدم تحديث PROJECT-BACKLOG أسبوعياً | عملية |

### 🟢 Low Debt

| # | الدين | الموقع |
|---|-------|--------|
| 8 | IMobileNavigationService (اسم مضلل) | Shared.UI/Services/ |
| 9 | HttpUtility في Blazor WASM | PharmacySearch.razor |
| 10 | alert() بدلاً من Toast | PspSearch.razor |
| 11 | SystemMenuMenuID Shadow Property | MenuAssignments |
| 12 | RoleId في MenuVisibilityRule | بعد Migration |

---

## 🗓️ Sprint 2 — Planned (23-29 أكتوبر 2026)

**الهدف:** إغلاق Sprint 1 المتبقي + بدء Sprint 0.

| # | المهمة | SP | الأولوية |
|---|--------|----|----------|
| 1 | إكمال P5.1 → P5.3 (التقارير) | 7 | 🔴 |
| 2 | اختبار E2E كامل | 3 | 🔴 |
| 3 | كتابة 19-PERMISSIONS | 5 | 🟠 |
| 4 | Migration 4: ControlLevel + Permission | 2 | 🟠 |
| 5 | Migration 5: UserControlLevel + JobTitleControlLevel | 2 | 🟠 |
| 6 | الترجمة المتبقية (P7.1) | 1 | 🟡 |

**المجموع:** 20 SP

---

## 📝 Changelog (مختصر)

### v2.0 — 23 أكتوبر 2026
- **إغلاق Sprint 1:** Points-PSP Integration (36 SP).
- **إصلاحات إضافية:** 6 مهام (10 SP) — IRefreshService + فقدان PSP + حساب النقاط + عرض الخصم.
- **تحديث الإحصائيات:** Points System 100%، Notifications 100%.

### v1.0 — 16 أكتوبر 2026
- إنشاء 00B-PROJECT-BACKLOG (دمج 18 + 23 + Changelog).
- إعادة تنظيم الوثائق (34 → 20).

### v0.9 — 13 أكتوبر 2026
- نظام النقاط v6.0 (Campaign Reference Architecture).

### v0.8 — 10 أكتوبر 2026
- تحديث 08-Strategic-Roadmap v5.0.

---

## 🔴 Lessons Learned (الدروس المستفادة)

### درس 1: فقدان ربط PSP بعد Migration (22 أكتوبر 2026)
**المشكلة:** Migration `RenamePatientIdToCaseId` أعادت تسمية العمود لكن لم تُنشئ سجلات `PSPPatients` للحالات التي كانت موجودة قبل التعديل. النتيجة: 22 حالة مفقودة، 15 مستخدماً فقدوا الربط ببرامجهم.

**الحل:**
1. إعادة إدراج 22 سجلاً من النسخة الاحتياطية #19 (2026-10-21).
2. تحديث 70 `eRx.CaseID` من القديم إلى الجديد.

**الإجراء الوقائي:**
- أي Migration تعيد تسمية FK يجب أن تتحقق من البيانات المرتبطة.
- اختبار E2E لكل Migration تعيد تسمية أعمدة FK.

---

### درس 2: ازدواجية `BeneficiaryID` في نظام النقاط (22 أكتوبر 2026)
**المشكلة:** `EarnPointsHandler` يستخدم `CaseID` (18) بينما `PspController` يستخدم `PatientProfileID` (15). النتيجة: النقاط المكتسبة لا تظهر في الواجهة.

**الحل:**
1. تعديل `PointsContext.GetBeneficiaryIdFor(5)` ليستخدم `UserID` (PatientProfileID).
2. تعديل `GenerateNextToken` لاستخراج `PatientProfileID` من `PSPPatients`.

---

### درس 3: `PENDING` Token يمنع `generate-next-token` (23 أكتوبر 2026)
**المشكلة:** توكن `PENDING` نشط يمنع توليد توكن جديد (رغم أنه لم يُعرض في الواجهة).

**الحل:**
1. استبعاد `PENDING` من `activeTokens` في `GenerateNextToken`.
2. إعطاء أولوية `GENERATED` و `ACTIVE` في `PspController.patientDetails`.

---

## 📝 سجل التحديثات

| التاريخ | التحديث |
|---------|---------|
| 16 أكتوبر 2026 | إنشاء الوثيقة — دمج 18 + 23 + Changelog |
| 20 أكتوبر 2026 | Points-PSP Integration (Planned) |
| 23 أكتوبر 2026 | **v2.0 — إغلاق Sprint 1 + 3 دروس مستفادة + إحصائيات محدثة** |

---

## 🔗 روابط ذات صلة

- [00-MASTER-INDEX](00-MASTER%20INDEX.md)
- [01-ARCHITECTURE](01%20%E2%80%94%20ARCHITECTURE%20.md)
- [05-PSP-SYSTEM](05-psp-system.md)
- [06-POINTS](06-points.md)

© 2026 RubikCare — للاستخدام الداخلي

نهاية الوثيقة ✅
```

---

