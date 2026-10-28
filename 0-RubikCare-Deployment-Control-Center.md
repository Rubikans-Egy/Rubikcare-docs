# 🚀 RubikCare - Deployment Control Center (v2.0)

مرحباً بك في مركز التحكم في النشر. هذا الدليل هو واجهتك الوحيدة لإدارة عمليات النشر (Deployment) والتراجع (Rollback) على السيرفر لجميع المشاريع (Web, API, PWA).

> 🛑 **قاعدة ذهبية:** لا تقم بتعديل السكريبتات الموجودة في هذا المجلد. استخدم هذا الدليل فقط لتنفيذ الأوامر.

---

## 📂 هيكل بيئات النشر (Environments)

1. **جهازك المحلي (Local):** تقوم بعمل Build/Publish إلى مجلد.
2. **البيئة الانتقالية (Staging):** تنسخ الملفات يدوياً إلى `C:\WebSite\_Staging\[ProjectName]`.
3. **بيئة الاختبار (Stage):** السكريبت ينقل الملفات من `_Staging` إلى بيئة الاختبار.
4. **بيئة الإنتاج (Production):** بعد التأكد من أن Stage يعمل، السكريبت ينقل الملفات إلى الإنتاج.

---

## 🛠️ 1. لوحة التحكم في العمليات (Web Project)

| # | المهمة | السكريبت | كيف تستخدمه؟ | ماذا يفعل؟ |
| :--- | :--- | :--- | :--- | :--- |
| 1 | **النشر إلى Stage** | `Deploy-ToStage-Web.ps1` | 1. ضع الملفات في `_Staging\Web`<br>2. اكتب `.\Deploy-ToStage-Web.ps1` | ينشئ نسخة احتياطية، ويوقف App Pool، وينسخ الملفات إلى `RubikCareNew`. |
| 2 | **التراجع عن Stage** | `Rollback-WebStage.ps1` | اكتب `.\Rollback-WebStage.ps1` | يعيد بيئة الاختبار إلى نسخة سابقة. |
| 3 | **النشر إلى الإنتاج** | `Deploy-ToProd-Web.ps1` | اكتب `.\Deploy-ToProd-Web.ps1` | ينقل الملفات من `RubikCareNew` إلى `RubikCareLive`. |
| 4 | **التراجع عن الإنتاج** | `Rollback-WebProd.ps1` | اكتب `.\Rollback-WebProd.ps1` | يعيد بيئة الإنتاج إلى نسخة سابقة. |

---

## 🛠️ 2. لوحة التحكم في العمليات (API Project)

| # | المهمة | السكريبت | كيف تستخدمه؟ | ماذا يفعل؟ |
| :--- | :--- | :--- | :--- | :--- |
| 1 | **النشر إلى Stage** | `Deploy-ToStage-Api.ps1` | 1. ضع الملفات في `_Staging\API`<br>2. اكتب `.\Deploy-ToStage-Api.ps1` | ينشئ نسخة احتياطية، ويوقف App Pool، وينسخ الملفات إلى `RubikCareUat`. |
| 2 | **التراجع عن Stage** | `Rollback-ApiStage.ps1` | اكتب `.\Rollback-ApiStage.ps1` | يعيد بيئة الاختبار إلى نسخة سابقة. |
| 3 | **النشر إلى الإنتاج** | `Deploy-ToProd-Api.ps1` | اكتب `.\Deploy-ToProd-Api.ps1` | ينقل الملفات من `RubikCareUat` إلى `RubicCareApi`. |
| 4 | **التراجع عن الإنتاج** | `Rollback-ApiProd.ps1` | اكتب `.\Rollback-ApiProd.ps1` | يعيد بيئة الإنتاج إلى نسخة سابقة. |

---

## 🛠️ 3. لوحة التحكم في العمليات (PWA Project)

| # | المهمة | السكريبت | كيف تستخدمه؟ | ماذا يفعل؟ |
| :--- | :--- | :--- | :--- | :--- |
| 1 | **النشر إلى Stage** | `Deploy-ToStage-PWA.ps1` | 1. ضع الملفات في `_Staging\PWA`<br>2. اكتب `.\Deploy-ToStage-PWA.ps1` | ينشئ نسخة احتياطية، ويوقف App Pool، وينسخ الملفات إلى `PU_RubicCareStage`. |
| 2 | **التراجع عن Stage** | `Rollback-PWAStage.ps1` | اكتب `.\Rollback-PWAStage.ps1` | يعيد بيئة PWA Stage إلى نسخة سابقة. |
| 3 | **النشر إلى الإنتاج** | `Deploy-ToProd-PWA.ps1` | اكتب `.\Deploy-ToProd-PWA.ps1` | ينقل الملفات من `PU_RubicCareStage` إلى `RubicCareStage`. |
| 4 | **التراجع عن الإنتاج** | `Rollback-PWAProd.ps1` | اكتب `.\Rollback-PWAProd.ps1` | يعيد بيئة PWA Production إلى نسخة سابقة. |

---

## 📋 4. سيناريوهات العمل الشائعة

### السيناريو الأول: نشر تحديث جديد لمشروع الويب (Web)
1. قم بعمل Publish من Visual Studio إلى مجلد على جهازك.
2. انسخ محتويات المجلد إلى السيرفر في: `C:\WebSite\_Staging\Web`.
3. في PowerShell: `cd C:\WebSite\_Docs\Scripts`
4. انشر للاختبار: `.\Deploy-ToStage-Web.ps1`
5. اختبر الموقع. إذا كان يعمل، انشر للإنتاج: `.\Deploy-ToProd-Web.ps1`.

### السيناريو الثاني: حدث خطأ في الإنتاج (Rollback Web)
1. اكتب `.\Rollback-WebProd.ps1`
2. اختر النسخة السابقة، واكتب `y`.

### السيناريو الثالث: نشر تحديث لمشروع PWA
1. انسخ الملفات إلى `_Staging\PWA`.
2. انشر للاختبار: `.\Deploy-ToStage-PWA.ps1`
3. اختبر PWA (تأكد من عمل `_framework` و MIME Types).
4. انشر للإنتاج: `.\Deploy-ToProd-PWA.ps1`.

---

## ⚙️ 5. معلومات تقنية هامة

* **مجلد النسخ الاحتياطي:** `C:\WebSite\_Docs\Backups` (يتم الاحتفاظ بآخر 10 نسخ فقط تلقائياً).
* **ملفات الإعدادات:** السكريبتات **لا تقوم بنسخ أو حذف** `appsettings.json` أو `web.config`. الإعدادات الموجودة على السيرفر لكل بيئة محمية.
* **الملفات المشتركة (SharedStorage):** لا تحذفها أو تنقلها. جميع الصور والملفات المرفوعة تُحفظ هناك.

---
© 2026 RubikCare — Internal DevOps.
