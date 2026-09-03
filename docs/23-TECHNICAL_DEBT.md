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
### 📌 [TD-00X] دعم مسح QR Code للانضمام من داخل تطبيق MAUI (Deep Linking)
- **الحالة:** مؤجل (Deferred)
- **الأولوية:** متوسطة
- **تاريخ الإضافة:** 2026-09-03
- **الوصف:** 
  حالياً، صفحة `JoinPage` تعمل بنجاح عند فتح الرابط عبر متصفح الهاتف الخارجي بعد مسح الـ QR Code. المطلوب لاحقاً هو ضمان أن يعمل نفس السيناريو بسلاسة إذا قام المريض بمسح الكود *من داخل* تطبيق MAUI نفسه (عبر ميزة الكاميرا المدمجة في التطبيق).
- **السبب في التأجيل:** 
  التركيز الحالي منصب على تشغيل وإكمال الصفحات الرئيسية المتبقية في المشروع (PWA & MAUI) لتسليم النواة الأساسية وتشغيل دورة العمل الرئيسية أولاً.
- **المهام المطلوبة لاحقاً عند العودة لهذا البند:**
  1. إعداد ودعم **Deep Linking / App Links** في مشروع MAUI للتعامل مع روابط النطاق (مثل `https://rubikcare.com/join?token=...`).
  2. عند مسح الكود من داخل التطبيق، يجب توجيه `BlazorWebView` مباشرة إلى مسار `/join?token=...` بدلاً من فتح المتصفح الخارجي.
  3. اختبار سيناريوهين عند الفتح من داخل التطبيق:
     - المستخدم غير مسجل: توجيهه لصفحة التسجيل مع حفظ الـ Token لاستخدامه بعد التسجيل.
     - المستخدم مسجل بالفعل: تطبيق كود الدعوة مباشرة على حسابه.
- **الملفات والمكونات ذات الصلة:** 
  - `RubikCare.PWA/Pages/PSP/JoinPage.razor`
  - إعدادات `AndroidManifest.xml` و `Info.plist` في مشروع `RubikCare.Mobile` لدعم Deep Linking.
  - خدمة التنقل (Navigation Service) في MAUI.

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
