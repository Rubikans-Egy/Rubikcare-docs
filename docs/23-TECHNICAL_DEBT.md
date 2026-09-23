---

# 🏗️ الديون التقنية وحالة المشروع - RubikCare PWA

**آخر تحديث:** 23 سبتمبر 2026  
**الحالة:** قيد المعالجة والتحسين المستمر  
**المسؤول:** فريق تطوير RubikCare (يوسف شادي + المساعد التقني)

---

## 📊 لوحة المعلومات السريعة (Dashboard)

| الفئة | العدد | الحالة |
|-------|-------|--------|
| ✅ تم الإنجاز مؤخراً | 13 | مغلق ومُختبر |
| 🔴 حرجة (قبل الإنتاج) | 3 | تتطلب تدخلاً فورياً |
| 🟡 متوسطة (خلال شهر) | 6 | مجدولة للسبرنت القادم |
| 🟢 منخفضة (تحسينات) | 3 | تراكمية، تُحل عند التعديل |
| ⏸️ مؤجلة / مخفية | 2 | قرار إداري بتأجيلها حالياً |
| **المجموع النشط** | **14** | - |

---

## ✅ ما تم إنجازه حديثاً (Recently Resolved)

### من Sprint سابق (8 سبتمبر 2026):
| # | المهمة / المشكلة | الحل المطبق | الحالة |
|---|------------------|-------------|--------|
| 1 | توجيه "Next Refill" الخاطئ | كان يوجه لبرامج الطبيب. تم تعديله ليوجه المريض لصفحة "تفاصيل البرنامج" الخاصة به. | ✅ مغلق |
| 2 | حالة الطلب تظهر كـ ??? ????? | إنشاء دالة `GetStatusTextArabic` في `OrderTracker.razor`. | ✅ مغلق |
| 3 | صفحة "دعواتي" (وضع العيادة) | إعادة تصميم كامل بـ CSS احترافي + PeriodicTimer. | ✅ مغلق |
| 4 | ميزة "أقرب الصيدليات" | دمج OsmPharmacyService مع بيانات قاعدة البيانات. | ✅ مغلق |
| 5 | كاميرا مسح QR غير مفعلة في PWA | دمج `html5-qrcode` مع JSInterop لإنشاء `QrScannerComponent`. | ✅ مغلق |
| 6 | بوابة الصيدلية تفتح برامج العيادة | إضافة Endpoint جديد في `PharmacyController`. | ✅ مغلق |

### من Sprint سبتمبر 2026 (Rep Dashboard & PSP):
| # | المهمة / المشكلة | الحل المطبق | الحالة |
|---|------------------|-------------|--------|
| 7 | المندوب لا يرى برامج شركته الصحيحة | إضافة `companyId` parameter في `GetCurrentRepInfo` + تمريره في كل endpoints `RepController`. | ✅ مغلق |
| 8 | خصومات الميزانية لا تظهر في PSP Step 2 | تعديل `PSPStep2_Budget.razor` لتحميل الأدوية من `Program.Medications` + `PharmaCompanyMedications`. | ✅ مغلق |
| 9 | حفظ الأدوية بـ `MedicationID = 0` | تعديل `SaveMeds` في `PSPStep1_SpecsMeds.razor` لاستخدام `OriginalMedicationID ?? PharmaCompanyMedicationID`. | ✅ مغلق |
| 10 | `Unsubscribe` endpoint غير موجود | إنشاء `PspController.Unsubscribe.cs` + `UnsubscribeDto`. | ✅ مغلق |
| 11 | `PharmaCompanyDashboard` لا يتحدث عند تبديل الشركة | استخدام `///` (3 slashes) في `GoToAsync` لعمل instance جديدة. | ✅ مغلق |
| 12 | `GetDashboardStats` يعرض أرقام كل الشركات | فلترة بـ `companyProgramIds` في كل استعلامات `RepController`. | ✅ مغلق |
| 13 | `MyInvitations` و `MyNetwork` بدون فلترة | إضافة `companyId` لكل الطلبات + فلترة في `GetMyInvitations` و `GetMyDoctors` و `GetMyPharmacies`. | ✅ مغلق |

---

## 🔴 حرجة (يجب حلها قبل الانتقال للإنتاج Production)

### 1. انتهاك معماري: صفحات كاملة تحتوي على @page داخل Shared.UI
**الملفات المتأثرة:**
- `Shared.UI/Components/Pharmacy/PharmacyDetailPage.razor`
- `Shared.UI/Components/PSP/PspAboutPage.razor`

**المشكلة:** يخالف وثيقة `00-architecture-overview.md`.  
**الحل المقترح:** نقل `@page` إلى Wrapper في `RubikCare.PWA`.

---

### 2. خلط الـ Namespaces بشكل غير متسق
**الملفات المتأثرة:** `PharmacySearchPage.razor` و `PharmacyDetailPage.razor`.  
**المشكلة:** namespace مكتوب `PharmacySearch` بدلاً من `Pharmacy`.  
**الحل المقترح:** توحيد namespaces لتطابق بنية المجلدات.

---

### 3. خطأ 404 في صفحة التراخيص المعلقة
**المسار:** `/admin/pending-licenses`  
**المشكلة:** الصفحة غير موجودة أو الـ Route غير معرف.  
**الحل المقترح:** فحص اسم المجلد (`SystemManagment` vs `SystemManagement`) وتصحيح الـ `@page`.

---

## 🟡 متوسطة (مجدولة للحل خلال الشهر القادم)

### 4. 🆕 مراحل خطة الصرف (Multi-Stage Dispensation)
**الأولوية:** 🟡 متوسطة — لكنها **مهمة جدًا** للمنتج  
**الوصف:**
- إنشاء نظام مراحل لخطة الصرف
- كل مرحلة ليها: مدة زمنية + أدوية خاصة + تفاصيل (كام علبة/شهر، التقسيمة)
- المستخدم يحدد عدد المراحل في الأول

**الملفات المتأثرة:**
- `Domain\Entities\PSP\PSPDispensationPlanPhase.cs` ⭐ جديد
- `Domain\Entities\PSP\PSPDispensationPlanPhaseMedication.cs` ⭐ جديد
- `Domain\Entities\PSP\PSPDispensationPlan.cs` ← إضافة `Phases`
- `Application\DTOs\PSP\DispensationPlanPhaseDto.cs` ⭐ جديد
- `Api.Web\Controllers\PSP\PspController.DispensationPlans.cs` ⭐ جديد
- `Rubikcare.Web\Components\Pages\Professional\PSPSteps\PSPStep2_DispensationPlans.razor` ← إعادة بناء

**التقدير:** ~12-15 ساعة (يومين عمل)

---

### 5. 🆕 تحديد التحاليل الطبية من شركة الدواء (Required Tests)
**الأولوية:** 🔴 حرجة (أساس لـ 6 و 7)  
**الوصف:**
- شركة الدواء تختار تحاليل معينة في البرنامج
- الطبيب والمريض يشوفوها بشكل مختلف

**الوضع الحالي:** `PSPRequiredTest` + `PSPTestResult` موجودين في Domain لكن مش موصولين بالـ UI.

**الملفات المتأثرة:**
- `Rubikcare.Web\Components\Pages\Professional\PSPSteps\PSPStep3_RequiredTests.razor` ⭐ جديد
- `Shared.UI\Components\PSP\PatientTests.razor` ⭐ جديد
- `Shared.UI\Components\PSP\DoctorTests.razor` ⭐ جديد
- `Api.Web\Controllers\PSP\PspController.Tests.cs` ⭐ جديد
- `Application\DTOs\PSP\PSPRequiredTestDto.cs` ⭐ جديد
- `Application\DTOs\PSP\PSPTestResultDto.cs` ⭐ جديد

**التقدير:** ~6-8 ساعات (يوم عمل)

---

### 6. 🆕 رفع التحاليل (PDF) + التذكيرات
**الأولوية:** 🟡 متوسطة  
**الوصف:**
- مكان لرفع التحاليل بصيغة PDF
- تسجيل مواعيد التذكيرات

**الملفات المتأثرة:**
- `Domain\Entities\PSP\Execution\PSPTestResult.cs` ← إضافة `FilePath` + `UploadedDate`
- `Api.Web\Controllers\PSP\PspController.Tests.cs` ← endpoint رفع
- `Shared.UI\Services\IFileUploadService.cs` ⭐ جديد
- `Shared.UI\Components\PSP\TestUploadDialog.razor` ⭐ جديد
- خدمة التذكيرات ⭐ جديد

**التقدير:** ~10-12 ساعة (يومين عمل)

---

### 7. 🆕 إظهار التحاليل مع مراعاة الخصوصية
**الأولوية:** 🟡 متوسطة  
**الوصف:**
- شركة الدواء: تشوف التحاليل **بدون** بيانات المريض الشخصية
- الطبيب: يشوف كل البيانات

**الملفات المتأثرة:**
- `Application\DTOs\PSP\PSPTestResultDto.cs` ← تقسيم لـ `PharmaViewDto` + `DoctorViewDto`
- `Api.Web\Controllers\PSP\PspController.Tests.cs` ← endpoint منفصل لكل role
- `Shared.UI\Components\PSP\PharmaTestsView.razor` ⭐ جديد
- `Shared.UI\Components\PSP\DoctorTestsView.razor` ⭐ جديد

**التقدير:** ~4-6 ساعات (نص يوم)

---

### 8. تجربة المستخدم لتعدد شركات الدواء (Pharma Company UX)
**المشكلة:** التنقل بين شركات الأدوية "Clunky" — تم حلها جزئيًا بـ `///`  
**المتبقي:** استخدام `CurrentOrganizationState` للتبديل الفوري (Client-side).  
**الحالة:** 🟡 قيد المراقبة (تم حل المشكلة الأساسية)

---

### 9. زر البحث في لوحة التحكم (Dashboard) غير مربوط
**الملف:** `Pages/Dashboard.razor`  
**المشكلة:** كروت البحث تعمل لكن المسار غير مربوط.  
**الحل المقترح:** ربط الزر بصفحة `PharmacySearch` أو `DoctorSearch`.

---

## 🟢 منخفضة (تحسينات تراكمية مستقبلية)

### 10. اسم خدمة مضلل
**الملف:** `Shared.UI/Services/IMobileNavigationService.cs`  
**الحل:** إعادة تسمية إلى `IAppNavigationService`.

---

### 11. استخدام HttpUtility في Blazor WASM
**الملف:** `Pages/PublicUser/PharmacySearch.razor`  
**الحل:** استخدام `Uri.EscapeDataString`.

---

### 12. استخدام alert() بدلاً من نظام Toast موحد
**الملفات:** `PspSearch.razor` ومكونات أخرى.  
**الحل:** استبدال بـ Toast Notification موحد.

---

## ⏸️ مؤجلة / مخفية حالياً (Deferred / Hidden)

### 13. صفحة المحادثات (Messaging Hub)
**الحالة:** مخفية من الـ UI.  
**السبب:** غير مكتملة.

---

### 14. دعم مسح QR Code للانضمام من داخل تطبيق MAUI (Deep Linking)
**الحالة:** مؤجل.  
**المهام المطلوبة لاحقاً:**
- إعداد Deep Linking في `AndroidManifest.xml` و `Info.plist`.
- توجيه BlazorWebView مباشرة إلى `/join?token=...`.

---

## 📅 خطة التنفيذ المقترحة للمهام الجديدة

| الترتيب | المهمة | الوقت المتوقع | يعتمد على |
|---------|--------|---------------|-----------|
| **1** | مراحل خطة الصرف (Multi-Stage) | يومين | — |
| **2** | تحديد التحاليل الطبية | يوم | — |
| **3** | خصوصية التحاليل | نص يوم | مهمة 2 |
| **4** | رفع PDF + تذكيرات | يومين | مهمة 2 + 3 |
| **المجموع** | | **~5-6 أيام** | |

---

## 📝 ملاحظات وقواعد معمارية ثابتة (Architectural Guardrails)

1. **قاعدة Shared.UI:** المكون في `Shared.UI` يجب أن يكون بدون `@page`. التوجيه (`@page`) يكون حصرياً في `PWA` أو `Web`.

2. **قاعدة الألوان:** يُمنع منعاً باتاً استخدام ألوان Hardcoded. استخدم `var(--rubik-primary)`.

3. **قاعدة الـ Prefix:** كل صفحة يجب أن تمتلك CSS Prefix فريد.

4. **قاعدة الـ Database:** `AsNoTracking()` في القراءة، `ExecuteWithNewContextAsync` في الكتابة.

5. **قاعدة الـ API Endpoints:** كل Controller يدير كياناته الخاصة.

6. **🆕 قاعدة Shell Navigation (`///`):** عند التبديل بين كيانات متعددة (شركات أدوية، صيدليات)، استخدم `///` (3 slashes) في `GoToAsync` لضمان إنشاء instance جديد وتحديث البيانات.

7. **🆕 قاعدة BlazorWebView + Parameters:** `[Parameter]` مع `RootComponent.Parameters` غير موثوق — استخدم `static Bridge` لنقل القيم.

8. **🆕 قاعدة `Set-Content`:** يُمنع استخدامه لتعديل ملفات `.razor` أو `.cshtml` (يتلف العربي).

---

**تم إنشاء هذا الملف ومراجعته لضمان توافق جميع التطورات المستقبلية مع معايير RubikCare.**

---
---

**عايز تعديل حاجة تانية؟ ولا أجهّز المدوّنة للشات الجديد؟ 🫡**
