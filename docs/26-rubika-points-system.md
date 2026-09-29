# 26 — نظام نقاط برامج الدعم (PSP Program Points)

**آخر تحديث:** 29 سبتمبر 2026
**الحالة:** ✅ Backend مكتمل — UI قيد الانتظار
**المؤلف:** فريق RubikCare

---

## 📌 نظرة عامة

نظام **نقاط برامج الدعم (PSP Program Points)** هو نظام مكافآت تُصممه شركات الأدوية لتحفيز الأطباء والعيادات على إتمام أحداث محددة خلال سير برامج دعم المرضى (PSP).

**الفلسفة الأساسية:**
- 🎯 **النقاط خاصة بكل برنامج** — وليس بالنظام العام
- 🏥 **الفاعل هو المؤسسة (العيادة)** — وليس الفرد (الطبيب)
- 💰 **الشركة تحدد القيمة** — وتتحمل التكلفة
- 📊 **النقاط تُصفَّر بـ Redeem** — خارج المنصة (Off-Platform)

---

## 🗺️ الخريطة المعمارية

### المكوّنات الرئيسية
┌─────────────────────────────────────────────────────────────┐
│ Domain Layer │
│ ┌───────────────────────────────────────────────────────┐ │
│ │ 8 كيانات في Entities/PSP/Points/ │ │
│ │ 3 ملفات Constants في Constants/PSP/Points/ │ │
│ └───────────────────────────────────────────────────────┘ │
└─────────────────────────────────────────────────────────────┘
↓
┌─────────────────────────────────────────────────────────────┐
│ Infrastructure Layer │
│ ┌───────────────────────────────────────────────────────┐ │
│ │ 19 Handler في UseCases/PSP/Points/ │ │
│ │ (Schemes, Rules, Balances, Earning, Redeem) │ │
│ └───────────────────────────────────────────────────────┘ │
└─────────────────────────────────────────────────────────────┘
↓
┌─────────────────────────────────────────────────────────────┐
│ Api.Web Layer │
│ ┌───────────────────────────────────────────────────────┐ │
│ │ 5 Controllers (18 Endpoints) │ │
│ │ + IPSPEnrollmentService (مدمجة) │ │
│ └───────────────────────────────────────────────────────┘ │
└─────────────────────────────────────────────────────────────┘
↓
┌─────────────────────────────────────────────────────────────┐
│ Client Layer │
│ ┌───────────────────────────────────────────────────────┐ │
│ │ ⏸️ MAUI/PWA: عرض الرصيد في صفحة البرنامج │ │
│ │ ⏸️ Web: تقارير الشركة │ │
│ └───────────────────────────────────────────────────────┘ │
└─────────────────────────────────────────────────────────────┘

text

---

## 🗄️ بنية قاعدة البيانات

### 8 جداول جديدة

| # | الجدول | الغرض |
|---|--------|-------|
| 1 | `PSPActivityTypes` | جدول مرجعي — أنواع الأحداث (Seed: 7) |
| 2 | `PSPProgramPointTransactionTypes` | جدول مرجعي — أنواع المعاملات (Seed: 4) |
| 3 | `PSPProgramPointRedeemStatuses` | جدول مرجعي — حالات Redeem (Seed: 4) |
| 4 | `PSPProgramPointSchemes` | نظام النقاط لكل برنامج |
| 5 | `PSPProgramPointRules` | قواعد المنح |
| 6 | `PSPProgramPointBalances` | أرصدة العيادات |
| 7 | `PSPProgramPointTransactions` | سجل المعاملات |
| 8 | `PSPProgramPointRedeemRequests` | طلبات الـ Redeem |

### Migration

- **الاسم:** `20260929163106_AddPSPProgramPointsTables`
- **عدد الجداول:** 8
- **عدد Unique Indexes:** 9
- **عدد Foreign Keys:** 14

### Seed Data (15 صفاً)

| الجدول | عدد الصفوف | القيم |
|--------|:----------:|-------|
| `PSPActivityTypes` | 7 | 1-7 (PatientInvited, PatientEnrolled, ErxCreated, DispensationCompleted, RefillCompleted, ProgramCompleted, LabTestUploaded) |
| `PSPProgramPointTransactionTypes` | 4 | 1-4 (Earn, RedeemRequest, RedeemAccepted, RedeemRejected) |
| `PSPProgramPointRedeemStatuses` | 4 | 1-4 (Pending, Accepted, Rejected, Expired) |

**⚠️ حرج:** الـ IDs يجب أن تبقى ثابتة (تُستخدم في `Constants`).

---

## 📚 الثوابت (Constants)

### `PointActivityTypeIds` — أنواع الأحداث

```csharp
public static class PointActivityTypeIds
{
    public const int PatientInvited = 1;
    public const int PatientEnrolled = 2;
    public const int ErxCreated = 3;
    public const int DispensationCompleted = 4;
    public const int RefillCompleted = 5;
    public const int ProgramCompleted = 6;
    public const int LabTestUploaded = 7;
}
PointTransactionTypeIds
csharp
public static class PointTransactionTypeIds
{
    public const int Earn = 1;
    public const int RedeemRequest = 2;
    public const int RedeemAccepted = 3;
    public const int RedeemRejected = 4;
}
RedeemStatusIds
csharp
public static class RedeemStatusIds
{
    public const int Pending = 1;
    public const int Accepted = 2;
    public const int Rejected = 3;
    public const int Expired = 4;
}
🎯 UseCases (19 Handler)
1️⃣ Schemes (4)
#	Handler	الوظيفة
1	CreatePointSchemeHandler	إنشاء نظام نقاط
2	UpdatePointSchemeHandler	تعديل نظام النقاط
3	GetPointSchemeByProgramHandler	جلب نظام النقاط
4	DeactivatePointSchemeHandler	تعطيل نظام النقاط
2️⃣ Rules (5)
#	Handler	الوظيفة
5	CreatePointRuleHandler	إنشاء قاعدة
6	UpdatePointRuleHandler	تعديل قاعدة
7	DeletePointRuleHandler	حذف قاعدة
8	GetPointRulesBySchemeHandler	جلب القواعد
9	TogglePointRuleHandler	تفعيل/تعطيل
3️⃣ Balances (3)
#	Handler	الوظيفة
10	GetBalanceByOrganizationHandler	رصيد عيادة
11	GetAllBalancesBySchemeHandler	كل الأرصدة
12	GetOrganizationPointSummaryHandler	ملخص عيادة
4️⃣ Earning (2)
#	Handler	Transaction	الوظيفة
13	EarnPointsHandler ⭐	✅	منح النقاط
14	GetPointTransactionsHandler	❌	سجل المعاملات
5️⃣ Redeem (5)
#	Handler	Transaction	الوظيفة
15	CreateRedeemRequestHandler	✅	إنشاء طلب
16	AcceptRedeemRequestHandler	✅	قبول الطلب
17	RejectRedeemRequestHandler	❌	رفض الطلب
18	GetRedeemRequestsHandler	❌	قائمة الطلبات
19	GetRedeemRequestDetailsHandler	❌	تفاصيل طلب
🌐 API Endpoints (18)
PointSchemesController (4)
POST /api/psp/points/schemes

PUT /api/psp/points/schemes/{id}

GET /api/psp/points/schemes/by-program/{programId}

POST /api/psp/points/schemes/{id}/deactivate

PointRulesController (5)
POST /api/psp/points/rules

PUT /api/psp/points/rules/{id}

DELETE /api/psp/points/rules/{id}

GET /api/psp/points/rules/by-scheme/{schemeId}

POST /api/psp/points/rules/{id}/toggle

PointBalancesController (3)
GET /api/psp/points/balances/by-organization

GET /api/psp/points/balances/by-scheme/{schemeId}

GET /api/psp/points/balances/organization-summary/{orgId}

PointEarningController (1)
GET /api/psp/points/transactions

⚠️ ملاحظة: لا يوجد Endpoint لـ POST /earn — EarnPointsHandler داخلي فقط (يُستدعى من Handlers أخرى عبر DI).

PointRedeemController (5)
POST /api/psp/points/redeem-requests

POST /api/psp/points/redeem-requests/{id}/accept

POST /api/psp/points/redeem-requests/{id}/reject

GET /api/psp/points/redeem-requests

GET /api/psp/points/redeem-requests/{id}

🔗 ربط النقاط بالأحداث
✅ تم ربطها (3 أحداث)
#	الحدث	الموقع	الحالة
1	PATIENT_INVITED	PSPEnrollmentService.CreateInvitationAsync	✅
2	PATIENT_ENROLLED	PSPEnrollmentService.EnrollUserInProgramAsync	✅
3	ERX_CREATED	PSPEnrollmentService.EnrollUserInProgramAsync	✅
⏸️ بحاجة لربط (2 أحداث)
#	الحدث	الموقع المُخطَّط	الأداة
4	DISPENSATION_COMPLETED	DispenseController.ConfirmDispense + ScanAndDispense	Helper
5	REFILL_COMPLETED	DispenseController.GenerateNextToken	Helper
❌ مؤجلة للمرحلة الثانية (2 أحداث)
#	الحدث	السبب
6	PROGRAM_COMPLETED	يحتاج تحديد آلية الإكمال
7	LAB_TEST_UPLOADED	يحتاج Handler لرفع التحاليل
🏗️ البنية المعمارية المُتبعة
نمط UseCase vs Service
النوع	الاستخدام	الموقع
Handler	تقارير Read-Only + عمليات معقدة	Application/UseCases/ + Infrastructure/UseCases/
Service	عمليات CRUD متعددة	Application/Services/
Helper	استدعاءات متكررة معزولة	داخل Controller
PSPEnrollmentService — نموذج الخدمة المتخصصة
الموقع: Application/Services/PSP/PSPEnrollmentService.cs

المسؤوليات:

CreateInvitationAsync — إنشاء دعوة + نقاط PATIENT_INVITED

EnrollUserInProgramAsync — تسجيل مريض + نقاط PATIENT_ENROLLED + ERX_CREATED

AcceptInvitationAsync — قبول دعوة + نفس النقاط

AcceptRepInvitationAsync — اشتراك مؤسسة (بدون نقاط)

النمط المُتبع:

csharp
private async Task TryAwardPointsAsync(...)
{
    try
    {
        await _earnPointsHandler.HandleAsync(...);
    }
    catch (Exception ex)
    {
        // ⚠️ لا نُفشل العملية الأصلية — نسجّل فقط
        _logger.LogWarning(ex, "...");
    }
}
🚧 الحالة الحالية
✅ ما تم إنجازه
#	المكوّن	الحالة
1	Domain Entities (8)	✅
2	Constants (3)	✅
3	Migration + Seed	✅
4	Handlers (19)	✅
5	Controllers (5 × 18 Endpoints)	✅
6	PSPEnrollmentService	✅
7	ربط PATIENT_INVITED	✅
8	ربط PATIENT_ENROLLED	✅
9	ربط ERX_CREATED	✅
⏸️ ما لم يتم إنجازه
#	المكوّن	الأولوية
1	ربط DISPENSATION_COMPLETED	🔴 عالية
2	ربط REFILL_COMPLETED	🔴 عالية
3	ربط PROGRAM_COMPLETED	🟡 متوسطة
4	ربط LAB_TEST_UPLOADED	🟡 متوسطة
5	واجهة MAUI: عرض الرصيد	🔴 عالية
6	واجهة Web: تقارير الشركة	🔴 عالية
7	واجهة Web: إدارة الـ Schemes	🟡 متوسطة
8	واجهة Web: إدارة الـ Redeem	🟡 متوسطة
9	اختبار شامل (End-to-End)	🔴 عالية
10	رسائل الإشعارات (Push)	🟢 منخفضة
📝 خطة الإتمام
المرحلة القادمة (يوم 1-2): ربط الأحداث المتبقية
المهمة 1: ربط DISPENSATION_COMPLETED

الموقع: DispenseController.ConfirmDispense

الأداة: Helper TryAwardPointsAsync

الحدث: بعد SaveChangesAsync بنجاح

OrganizationID: من PSPPatient.Participation.ParticipantOrganizationID

المهمة 2: ربط REFILL_COMPLETED

الموقع: DispenseController.GenerateNextToken

الأداة: نفس Helper

الشرط: refillNumber > 0

المرحلة 2 (يوم 3-4): واجهة MAUI/PWA
المهمة 3: عرض الرصيد في صفحة البرنامج

الملف: PSPProgramDetailsPage.xaml (MAUI) + .razor (PWA)

API: GET /api/psp/points/balances/by-organization

العرض: بطاقة تعرض الرصيد + آخر 5 معاملات

المهمة 4: عرض سجل المعاملات

API: GET /api/psp/points/transactions?organizationId=X&schemeId=Y&take=20

المرحلة 3 (يوم 5-7): لوحة تحكم الشركة
المهمة 5: إدارة Schemes + Rules

الصفحة: PointSchemesManagement.razor

APIs:

POST /api/psp/points/schemes

POST /api/psp/points/rules

GET /api/psp/points/schemes/by-program/{programId}

المهمة 6: تقارير الأرصدة

الصفحة: PointBalancesReport.razor

API: GET /api/psp/points/balances/by-scheme/{schemeId}

المهمة 7: إدارة الـ Redeem

الصفحة: RedeemRequestsManagement.razor

APIs: POST /api/psp/points/redeem-requests + GET /api/psp/points/redeem-requests

المرحلة 4 (يوم 8-10): اختبار + تحسينات
المهمة 8: اختبار شامل

السيناريو: إنشاء Scheme + Rules → دعوة مريض → اشتراك → صرف → Redeem

التحقق: من كل نقطة يدوياً في قاعدة البيانات

المهمة 9: رسائل الإشعارات

Push notification عند:

منح النقاط

إنشاء Redeem

قبول Redeem

⚠️ قواعد حرجة
1️⃣ أرقام الـ Constants لا تتغير
csharp
// ⚠️ لا تغيّر هذه الأرقام — فهي مرتبطة بـ Seed Data
public const int PatientEnrolled = 2;
2️⃣ IDENTITY_INSERT عند Seed
sql
SET IDENTITY_INSERT PSPActivityTypes ON;
-- إدراج صريح بـ IDs
SET IDENTITY_INSERT PSPActivityTypes OFF;
3️⃣ TryAwardPointsAsync معزول
csharp
// ⚠️ لا نرمي استثناء — العملية الأصلية نجحت
catch (Exception ex)
{
    _logger.LogWarning(ex, "فشل منح النقاط — العملية الأصلية نجحت");
}
4️⃣ فحص PointsEarned قبل العرض
csharp
if (result.IsSuccess && result.PointsEarned)
{
    // النقاط مُنحت فعلاً
}
🔗 روابط ذات صلة
00 - الهيكل المعماري

02 - نظام الهوية

07 - نظام PSP

08 - خطة التطوير

27 - الديون التقنية

نهاية الوثيقة 🚀
