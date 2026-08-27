# الديون التقنية - RubikCare PWA

> آخر تحديث: 28 أغسطس 2026
> الحالة: قيد الترحيل التدريجي

---

## 🔴 حرجة (قبل الإنتاج)

### 1. صفحات كاملة في Shared.UI تحتوي على @page
**الملفات المتأثرة:**
- `Shared.UI/Components/Pharmacy/PharmacyDetailPage.razor` → `@page "/pharmacy/{PharmacyId:int}"`
- `Shared.UI/Components/PSP/PspAboutPage.razor` → `@page "/psp/about"`

**المشكلة:** يخالف وثيقة `00-architecture-overview.md`:
> "❌ ما لا يوضع في Shared.UI: صفحات كاملة"

**الحل المقترح:** نقل `@page` إلى PWA، وإبقاء المكون في Shared.UI بدون توجيه.

---

### 2. خلط namespace غير متسق
**الملفات المتأثرة:**
- `PharmacySearchPage.razor` → namespace `PharmacySearch` (المفترض `Pharmacy`)
- `PharmacyDetailPage.razor` → namespace `PharmacySearch` (المفترض `Pharmacy`)

**المشكلة:** صعوبات في الاستيراد والصيانة.

**الحل المقترح:** توحيد الـ namespaces في `Shared.UI/Components/Pharmacy`.

---

## 🟡 متوسطة (خلال شهر)

### 3. كاميرا مسح QR غير مفعلة في PWA
**الملف:** `Shared.UI/Components/PSP/Patient/PspDetailComponent.razor`
**الحالة:** زر `ScanQr` يعرض رسالة "الكاميرا غير متوفرة"
**الحل:** مكتبة `html5-qrcode` أو `@zxing/browser`

### 4. OnStartChat غير مفعل في صفحة الطبيب
**الملف:** `Shared.UI/Components/Messaging/DoctorProfilePage.razor`
**الحالة:** الـ Parameter موجود لكن لم يمرر من PWA
**الحل:** ربط بصفحة المحادثات

### 5. زر البحث في Dashboard غير مربوط بمسار فعلي
**الملف:** `Pages/Dashboard.razor`
**الحالة:** كروت البحث عن طبيب/صيدلية تعمل، لكن لم تتحقق من مسار البحث الفعلي في القائمة الجانبية

---

## 🟢 منخفضة (تحسينات مستقبلية)

### 6. اسم خدمة مضلل
**الملف:** `Shared.UI/Services/IMobileNavigationService.cs`
**المشكلة:** الاسم "Mobile" رغم استخدامها في PWA
**الحل:** إعادة تسمية إلى `IAppNavigationService`

### 7. استخدام HttpUtility في Blazor WASM
**الملف:** `Pages/PublicUser/PharmacySearch.razor`
**المشكلة:** `System.Web.HttpUtility` غير موصى به في WASM
**الحل:** استخدام `System.Web.HttpUtility` من `Microsoft.AspNetCore.WebUtilities`

### 8. alert() بدلاً من Toast
**الملفات المتأثرة:** `PspSearch.razor`
**المشكلة:** تجربة مستخدم ضعيفة
**الحل:** نظام Toast موحد

### 9. صفحات الموبايل الملحقة
**الحالة:** مؤجلة بقرار - كاميرا، مسح، إشعارات أصلية
**الحل:** لاحقاً عند دعم MAUI للـ Shared.UI الجديد

---

## 📊 إحصائيات

| الفئة | العدد | الحالة |
|-------|-------|--------|
| 🔴 حرجة | 2 | تحتاج معالجة قبل الإنتاج |
| 🟡 متوسطة | 3 | خلال شهر |
| 🟢 منخفضة | 4 | تحسينات مستقبلية |
| **المجموع** | **9** | - |

---

## 📝 ملاحظات

- **قاعدة معتمدة:** المكون في Shared.UI بدون `@page`، والصفحة في PWA (نمط `PspDetailComponent`)
- **قرار معلق:** ترحيل `PharmacyDetailPage` و `PspAboutPage` إلى النمط الصحيح
