# 📊 ملخص الجلسة — [التاريخ: 28 نوفمبر 2026]

## 🎯 هدف الجلسة
استكمال تدفق "تسجيل زيارة العيادة عبر QR" من البداية للنهاية، وإصلاح المشاكل التقنية التي ظهرت أثناء الاختبار (الصفحة البيضاء، 401 Unauthorized، تسجيل الزيارات المتتالية، ظهور المرضى في القائمة)، ثم توثيق النتائج.

---

## ✅ ما تم إنجازه (مكتمل 100%)

### 1️⃣ إصلاح مشكلة "الصفحة البيضاء" في `ClinicVisitConfirmPage`
**الأعراض:** صفحة بيضاء فارغة عند مسح QR لزيارة العيادة.
**السبب الجذري:**
- `ClinicVisitConfirmView.razor` كان يعتمد على `@inject IApiService`، بينما الحاوية `ClinicVisitConfirmPage.xaml.cs` كانت تمرر `ApiService` كـ `Parameter` — وهذا تناقض يسبب استثناءً صامتاً في `BlazorWebView`.
- الحاوية كانت تستخدم `[QueryProperty]` بدلاً من `IQueryAttributable`، مما قد يؤدي لاستدعاء `LoadBlazorComponent()` قبل اكتمال التهيئة.

**الحل المطبق (مطابق للنمط الناجح `EnrollClinicReferralFlow`):**
- إزالة `@inject IApiService` من المكون، وإضافة `[Parameter] public IApiService? ApiService { get; set; }`.
- تعديل الحاوية لتنفيذ `IQueryAttributable` واستدعاء `LoadBlazorComponent()` داخل `ApplyQueryAttributes`.
- إضافة `SafeAsyncAction` للـ `OnBack`.

**الملفات المتأثرة:**
- `Shared.UI/Components/Clinic/ClinicVisitConfirmView.razor`
- `Mobile/Features/Clinic/Views/ClinicVisitConfirmPage.xaml.cs`

---

### 2️⃣ إصلاح مشكلة 401 Unauthorized في `ConfirmVisit`
**الأعراض:** عند الضغط على "تأكيد الزيارة"، كان الـ Endpoint يرجع `401` برسالة "يجب تسجيل الدخول لتأكيد الزيارة".
**السبب الجذري:**
- `ClaimTypes.NameIdentifier` في التوكن يحتوي على **`AspNetUserID` (GUID)** وليس `UserProfileID` (int).
- الكود كان يحاول `int.TryParse` على GUID → يفشل → يرجع `Unauthorized`.
- الـ `UserProfileID` الفعلي موجود في claim باسم **`userProfileId`** (بقيمة `15`).

**الحل المطبق:**
- تعديل `ClinicVisitsController.cs` ليبحث عن `userProfileId` (وأسماء بديلة) بدلاً من `ClaimTypes.NameIdentifier`.
- حذف `[AllowAnonymous]` المؤقت واستعادة `[Authorize]`.

**الملفات المتأثرة:**
- `Api.Web/Controllers/Clinic/ClinicVisitsController.cs`

---

### 3️⃣ إضافة "عداد الزيارات" (Visits Count) لكارت المريض
**الهدف:** عرض عدد زيارات المريض في العيادة بجانب بيانات الصرف.

**التعديلات:**
- `ClinicPatientDto.cs`: إضافة حقل `public int VisitsCount { get; set; }`.
- `GetClinicPatientsHandler.cs`: إضافة استعلام `GroupBy` على `ClinicVisits` لجلب عدد الزيارات لكل مريض (بناءً على `PatientProfileID` و `ClinicID`).
- `ClinicPatientsList.razor`: إضافة سطر لعرض 🗓️ `@patient.VisitsCount` بجانب 💊 `TotalDispensations`.
- إضافة مفتاح الترجمة `CLINIC_PATIENTS.VISITS` (عربي: "زيارات" / إنجليزي: "Visits") في جدول `Resources` (Module: `CLINIC_PATIENTS`, ResourceType: `Label`).

**الملفات المتأثرة:**
- `Application/DTOs/Clinic/ClinicPatientDto.cs`
- `Infrastructure/UseCases/Clinic/Patients/GetClinicPatientsHandler.cs`
- `Shared.UI/Components/Clinic/ClinicPatientsList.razor`
- جدول `Resources` (SQL)

---

### 4️⃣ إضافة منطق "الحد الأدنى 7 أيام بين الزيارات"
**الهدف:** منع تسجيل زيارات متتالية لنفس المريض في نفس العيادة خلال أقل من 7 أيام.

**التعديلات:**
- `ConfirmVisitByPatientHandler.cs`: إضافة استعلام لجلب آخر زيارة `CHECKED_IN` للمريض في العيادة، ومقارنة `ActualVisitDateTime` بالتاريخ الحالي.
- إضافة ثابت `MinDaysBetweenVisits = 7` (قابل للتعديل مستقبلاً من إعدادات العيادة).
- عند الفشل: يرجع `Success = false` مع رسالة تفصيلية (تاريخ آخر زيارة + عدد الأيام المتبقية).

**الملفات المتأثرة:**
- `Infrastructure/UseCases/Clinic/Visits/ConfirmVisitByPatientHandler.cs`

---

### 5️⃣ إظهار رسالة الفشل في واجهة المريض
**الهدف:** عرض رسالة منع الزيارة المتتالية (400 BadRequest) للمريض بدلاً من بقائها في اللوج.

**التعديلات:**
- `ClinicVisitConfirmView.razor`: إضافة بلوك `<div class="dp-booking-error">` يظهر عند `!_visitConfirmed && !string.IsNullOrEmpty(_visitConfirmMessage)` — باللون الأحمر مع أيقونة تحذير.

**الملفات المتأثرة:**
- `Shared.UI/Components/Clinic/ClinicVisitConfirmView.razor`

---

### 6️⃣ إصلاح ظهور المريض في قائمة `ClinicPatientsList`
**الأعراض:** المريض (PatientProfileID = 15) سجل زيارة في العيادة (ClinicID = 1261)، لكنه لا يظهر في قائمة المرضى.
**السبب الجذري:**
- `GetClinicPatientsHandler` كان يشترط `InvitationMethod == "STATIC_REFERRAL"` — بينما المريض الحقيقي دخل عبر `InvitationMethod = 'CODE'` (دعوة فردية).
- كلاهما يجب أن يظهر في القائمة.

**الحل المطبق:**
- إزالة شرط `InvitationMethod == "STATIC_REFERRAL"` من الاستعلام.
- الاعتماد فقط على `InvitedByOrganizationID == query.ClinicId` (المريض مرتبط بالعيادة).

**الملفات المتأثرة:**
- `Infrastructure/UseCases/Clinic/Patients/GetClinicPatientsHandler.cs`

---

## 📋 المهام المتبقية (بالترتيب)

### 🟡 المرحلة أ: استكمال دورة الزيارة (مكتملة ✅)
- [x] إصلاح الصفحة البيضاء.
- [x] إصلاح 401 Unauthorized.
- [x] إضافة VisitsCount للكارت.
- [x] منع الزيارات المتتالية (7 أيام).
- [x] إظهار رسالة الفشل.
- [x] إصلاح ظهور المرضى في القائمة.

### 🔴 المرحلة ب: البيانات المرنة (PatientRequiredDataEntry) — لم تبدأ
**الهدف:** نموذج يعرض للمريض البيانات المطلوبة من شركة الأدوية عند التسجيل في برنامج دعم (مثل: أسبوع الحمل، السن، عدد مرات الحمل السابق، هل سبق الإجهاض).

**ما تم إنجازه في الـ Backend (من الجلسة السابقة):**
- `GetRequiredDataQuery` + Handler: جلب البيانات المطلوبة + القيم الحالية.
- `SubmitRequiredDataCommand` + Handler: Upsert + تحقق (IsMandatory, Min/Max).
- `PSPPatientDataController`: `GET/POST api/psp/patient/{participationId}/required-data`.

**المطلوب في الـ Frontend:**
| # | المهمة | المخرج | الاعتماد |
|---|---|---|---|
| B1 | إنشاء `PatientRequiredDataEntryPage.razor` في Shared.UI (باستخدام `IApiService` كـ Parameter — النمط الناجح). | مكون Blazor | Backend جاهز |
| B2 | إنشاء صفحة MAUI `PatientRequiredDataEntryPage.xaml` (IQueryAttributable + RootComponents.Add). | حاوية MAUI | B1 |
| B3 | تعديل `Join.razor` أو `PspEntry.razor` لتوجيه المريض إلى هذه الصفحة بعد نجاح الاشتراك (إذا كان البرنامج يحتوي على بيانات مطلوبة). | ربط التدفق | B2 |
| B4 | إضافة `RequiresDataEntry` في استجابة `POST /api/psp/entry` (يُرجع `true` إذا كان البرنامج يحتوي على `PSPRequiredDataEntry`). | تعديل API | — |
| B5 | اختبار E2E: اشتراك → توجيه → ملء البيانات → حفظها في `PSPPatientDataEntries`. | تأكيد | B1-B4 |

### 🟠 المرحلة ج: الإشعارات الأسبوعية (اختياري — مؤجلة)
| # | المهمة | المخرج |
|---|---|---|
| C1 | `GetPregnancyWeekQuery` (يقرأ `PSPPatientDataEntry` حيث `RequiredDataEntryID = "أسبوع الحمل"`). | API جاهز |
| C2 | تحليل `PSPReminders.ReminderType = 1` — هل يصلح للإشعارات الأسبوعية؟ | قرار |
| C3 | محرك الإشعارات الأسبوعية (Background Job). | مؤجل |

### 🔵 المرحلة د: تنظيف وتوثيق نهائي
| # | المهمة | الحالة |
|---|---|---|
| D1 | تحديث `docs/05-psp-system.md` (قسم "Clinic Reports + Patient Data Entries + Visits"). | ⏳ |
| D2 | تحديث `docs/13-maui-development-guide.md` (قسم "حل مشكلة الصفحة البيضاء"). | ⏳ |
| D3 | تحديث `23-TECHNICAL_DEBT.md` (إزالة "POST api/psp/entry لا يربط الدعوة"). | ⏳ |
| D4 | Commit نهائي للجلسة. | ⏳ |

---

## 🗂️ جدول الملفات المرتبطة (للمراجعة السريعة)

| الملف | الغرض | ملاحظة |
|---|---|---|
| `ClinicVisitConfirmView.razor` | واجهة تأكيد الزيارة | يعتمد على `[Parameter]` IApiService |
| `ClinicVisitConfirmPage.xaml.cs` | حاوية MAUI | `IQueryAttributable` |
| `ClinicVisitsController.cs` | Endpoint التأكيد | يبحث عن `userProfileId` |
| `ConfirmVisitByPatientHandler.cs` | منطق التأكيد | قاعدة 7 أيام |
| `ClinicPatientDto.cs` | DTO | يحتوي على `VisitsCount` |
| `GetClinicPatientsHandler.cs` | استعلام القائمة | بدون شرط `InvitationMethod` |
| `ClinicPatientsList.razor` | قائمة المرضى | يعرض VisitsCount |
| جدول `Resources` | التراجم | `CLINIC_PATIENTS.VISITS` |

---

## 📌 دروس مستفادة (Lessons Learned)
1. **BlazorWebView في MAUI:** لا تستخدم `@inject` لخدمات MAUI (مثل `IApiService`) — مررها كـ `Parameter` (13-MAUI §2.3).
2. **`[QueryProperty]` vs `IQueryAttributable`:** استخدم `IQueryAttributable` في الصفحات التي تعتمد على `BlazorWebView` — لضمان التهيئة بعد اكتمال `InitializeComponent()`.
3. **`ClaimTypes.NameIdentifier` ≠ `UserProfileID`:** في نظام RubikCare، `NameIdentifier` هو `AspNetUserID` (GUID)، و`UserProfileID` (int) في claim منفصل اسمه `userProfileId`.
4. **أخطاء 401 من الـ Controller:** قد تأتي الرسالة العربية من داخل الـ Controller (وليس من `[Authorize]`) — راجع الـ claims دائماً.
5. **استعلامات القوائم:** لا تقيدها بشروط `InvitationMethod` الصارمة — استخدم معايير الربط الأساسية (مثل `InvitedByOrganizationID`).

---

## ⏭️ الخطوة التالية
البدء في المرحلة ب: إنشاء `PatientRequiredDataEntryPage.razor` (Frontend) — بعد تأكيد أن الـ Backend جاهز.

---
