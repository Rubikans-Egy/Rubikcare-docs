[#32]

## 📊 ملخص إنجازات الجلسة الحالية

### 🎯 هدف الجلسة
تمكين "تقارير المرضى في العيادة" + إدخال بيانات المريض المرن (للإشعارات الأسبوعية) + تتبع الزيارات عبر QR.

---

### ✅ ما تم إنجازه (مكتمل 100%)

#### 1️⃣ تحليل معماري وقرارات تقنية
| القرار | السبب |
|--------|-------|
| إضافة `ScopeType` إلى `PSPRequiredDataEntry` (1=Program, 2=Clinic, 3=PHR) | تجنب دين تقني عند توسعة Clinic OS / PHR |
| جعل `ProgramID` nullable + CHECK CONSTRAINT | دعم ScopeType≠1 دون كسر العلاقة مع البرنامج |
| `PSPPatientDataEntry` يستخدم `CaseID` (وليس PatientProfileID) | اتساق مع `PSPTestResult` و`PSPFollowUpRecord` |
| `ParticipationID` nullable في `PSPPatientDataEntry` | دعم PHR مستقبلاً |
| `InvitationMethod = 'STATIC_REFERRAL'` كعلامة canonical | يميز مرضى كود الإحالة عن الدعوات الفردية (بدون حقل ReferralID جديد) |
| `ClinicVisit` منفصل عن `PSPFollowUpRecord` | الأول زيارة عيادة، الثاني متابعة برنامج |

#### 2️⃣ قاعدة البيانات (Migrations مطبّقة)
| Migration | المحتوى |
|-----------|---------|
| `AddClinicVisitsAndPatientDataEntries` | جدول `ClinicVisits` + جدول `PSPPatientDataEntries` + `ScopeType` + `ProgramID nullable` + الفهارس |
| `FixScopeTypeCheckConstraint` | إصلاح CHECK CONSTRAINT (أقواس مزدوجة) |

**تم التحقق:**
- ✅ `ClinicVisits` موجود + 3 FKs + 2 Indexes
- ✅ `PSPPatientDataEntries` موجود + 3 FKs + Unique Index (filtered)
- ✅ CHECK CONSTRAINT يعمل (اختبار `INSERT` مرفوض فعلياً)

#### 3️⃣ Backend — إدخال بيانات المريض (المهمة العاجلة 1)
| الملف | الوظيفة |
|-------|---------|
| `GetRequiredDataQuery.cs` | Query + Response DTOs |
| `IGetRequiredDataHandler.cs` | Interface |
| `GetRequiredDataHandler.cs` | جلب البيانات المطلوبة + القيم الحالية |
| `SubmitRequiredDataCommand.cs` | Command + DTOs |
| `ISubmitRequiredDataHandler.cs` | Interface |
| `SubmitRequiredDataHandler.cs` | Upsert + تحقق (IsMandatory, Min/Max) |
| `PSPPatientDataController.cs` | `GET/POST api/psp/patient/{participationId}/required-data` |

**✅ البناء ناجح.**

#### 4️⃣ Backend — تقرير مرضى العيادة (المهمة العاجلة 2)
| الملف | الوظيفة |
|-------|---------|
| `GetClinicPatientsQuery.cs` | Query + `ClinicPatientDto` + Response |
| `IGetClinicPatientsHandler.cs` | Interface |
| `GetClinicPatientsHandler.cs` | استعلام `InvitationMethod='STATIC_REFERRAL'` + `InvitedByOrganizationID=ClinicId` + تجميع الصرف |
| `ClinicPatientsController.cs` | `GET api/clinic/{clinicId}/patients` |

**✅ الكود مكتوب (بانتظار تأكيد البناء).**

#### 5️⃣ DI Registration
- `IGetRequiredDataHandler` + `ISubmitRequiredDataHandler` → مسجلان.
- `IGetClinicPatientsHandler` → **بانتظار التسجيل** (الملف 5 في الرد السابق).

#### 6️⃣ التوثيق
- ✅ Commit message شامل للجلستين.

---
[#33]
### 📊 ما تغيّر
- **المرحلة أ** → من 🟡 80% إلى 🟢 **95%** (A1 + A2 مكتملان: DI مسجّل + البناء ناجح في 3 مشاريع).
- **المتبقي في أ**: فقط **A3** (تعديل `ClinicPatientsView.razor`) + **A4** (اختبار Endpoint).

---

## 📋 الملخص المحدّث (المرحلة أ)

| # | المهمة | الحالة |
|---|--------|--------|
| A1 | تسجيل `IGetClinicPatientsHandler` في DI | ✅ مكتمل |
| A2 | `dotnet build` (API + Infrastructure + Application) | ✅ ناجح |
| A3 | تعديل `ClinicPatientsView.razor` ليقرأ `response.Patients` | ⏳ متبقٍ |
| A4 | اختبار Endpoint عبر Swagger | ⏳ متبقٍ |

---

## ⏭️ خطوتي التالية الواحدة: **A3** (تعديل `ClinicPatientsView.razor`)

**قبل الكتابة، أحتاج تأكيد نقطة واحدة فقط:**

في `ClinicPatientsView.razor` الحالي (من الرد السابق)، الكود يستدعي:

```csharp
_allPatients = await ApiService.GetAsync<List<ClinicPatientDto>>(
    $"api/clinic/{actualClinicId}/patients") ?? new();
```

والـ Endpoint الجديد يُرجع `GetClinicPatientsResponse` (كائن يحوي `Patients` + `ActiveCount` + `CompletedCount` + `TotalCount`).

**تعديل الواجهة سيكون:**

```csharp
var response = await ApiService.GetAsync<GetClinicPatientsResponse>(
    $"api/clinic/{actualClinicId}/patients?pageSize=1000") ?? new();

_allPatients = response.Patients ?? new();
_activeCount = response.ActiveCount;
_completedCount = response.CompletedCount;
_totalCount = response.TotalCount;
```

> 💡 **ملاحظة**: `pageSize=1000` لجلب كل المرضى (لأن الواجهة الحالية تُطبّق الفلترة والترقيم **محلياً** في `ApplyFilters()`). هذا يحافظ على سلوك الواجهة دون تغيير جوهري.

**هل توافق على هذا التعديل؟** (نعم/لا — وسأكتب الكود الكامل للتعديل في الرد التالي مباشرة، مع تحديد الأسطر التي تُستبدل بدقة).

## 📋 ما تبقى من أهداف الجلسة (خطة دقيقة)

### 🟢 المرحلة أ: استكمال تقرير المرضى (30 دقيقة)

| # | المهمة | المخرج | الاعتماد |
|---|--------|--------|----------|
| **A1** | تسجيل `IGetClinicPatientsHandler` في `InfrastructureExtensions.cs` | DI يعمل | — |
| **A2** | `dotnet build` + إصلاح أي أخطاء | بناء ناجح | A1 |
| **A3** | تعديل `ClinicPatientsView.razor` ليقرأ `response.Patients` (بدلاً من `List<ClinicPatientDto>` مباشرة) | الواجهة تعمل مع الـ Endpoint الجديد | A2 |
| **A4** | اختبار Endpoint عبر Swagger/Postman | تأكيد النتائج | A2 |

---

### 🟡 المرحلة ب: تسجيل الزيارة عبر QR (المهمة العاجلة 3) — 60-90 دقيقة

| # | المهمة | المخرج | الاعتماد |
|---|--------|--------|----------|
| **B1** | `ConfirmVisitCommand` + Handler (يُنشئ `ClinicVisit` بـ `Source='QR_MANUAL'` + `ConfirmedByUserID`) | Backend جاهز | A2 |
| **B2** | `GetPatientVisitsQuery` + Handler (لعرض سجل زيارات المريض) | Backend جاهز | B1 |
| **B3** | `ClinicVisitsController` (`POST /api/clinic/visits/confirm`, `GET /api/clinic/{clinicId}/patients/{patientId}/visits`) | Endpoints جاهزة | B1, B2 |
| **B4** | تعديل `ClinicPatientDetailsView.razor`: إضافة زر "QR الزيارة" + Modal تأكيد الزيارة (نوع الزيارة + ملاحظات) | واجهة المريض | B3 |
| **B5** | شاشة تأكيد الزيارة لموظف العيادة (بعد مسح QR): تختار `VisitType` (REGULAR/FOLLOW_UP) + ملاحظات | واجهة الموظف | B3 |
| **B6** | اختبار E2E: مسح QR → تأكيد → ظهور السجل في `GetPatientVisits` | تأكيد كامل | B4, B5 |

---

### 🟠 المرحلة ج: ربط الإشعارات الأسبوعية (اختياري — إن سمح الوقت) — 30-45 دقيقة

| # | المهمة | المخرج |
|---|--------|--------|
| **C1** | `GetPregnancyWeekQuery` (يقرأ `PSPPatientDataEntry` حيث `RequiredDataEntryID` = "أسبوع الحمل") | API جاهز |
| **C2** | تحليل `PSPReminders.ReminderType=1` (هل يصلح لإشعارات أسبوعية؟) — **يحتاج فحصاً** | قرار: نستخدم `PSPReminders` أم نُنشئ كياناً جديداً |
| **C3** | محرك الإشعارات الأسبوعية (Background Job) — **قد يُؤجل للجلسة القادمة** | — |

---

### 🔵 المرحلة د: تنظيف وتوثيق نهائي — 15 دقيقة

| # | المهمة |
|---|--------|
| **D1** | تحديث `docs/05-psp-system.md` (قسم جديد: "Clinic Reports + Patient Data Entries + Visits") |
| **D2** | تحديث `23-TECHNICAL_DEBT.md` (إزالة "POST api/psp/entry لا يربط الدعوة" — لأنه أصبح يعمل) |
| **D3** | Commit نهائي للجلسة |

---

## 📊 ملخص الحالة

| المرحلة | الحالة | المدة المتوقعة |
|---------|--------|----------------|
| **أ: تقرير المرضى** | 🟡 80% (كود جاهز، بانتظار DI + بناء + تعديل الواجهة) | 30 دقيقة |
| **ب: تسجيل الزيارة QR** | 🔴 0% (لم يبدأ) | 60-90 دقيقة |
| **ج: الإشعارات الأسبوعية** | 🔴 0% (اختياري) | 30-45 دقيقة |
| **د: تنظيف وتوثيق** | 🔴 0% | 15 دقيقة |

**الأولوية القصوى**: إكمال **المرحلة أ** (لتأكيد أن تقرير المرضى يعمل فعلاً) ثم **المرحلة ب** (تسجيل الزيارة عبر QR).

---A3** (تعديل `ClinicPatientsView.razor`)؟

**توصيتي: (1)** — لأن البناء سيكشف أي أخطاء في `GetClinicPatientsHandler` قبل تعديل الواجهة.
