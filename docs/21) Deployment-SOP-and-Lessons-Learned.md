# 🚀 RubikCare — وثيقة الدرس المستفاد وإجراءات النشر الآمن (Post-Mortem & Deployment SOP)

**تاريخ التوثيق:** 20 أغسطس 2026  
**المطور:** يوسف شادي  
**الحالة:** تم حل مشكلة "الفشل الصامت" وترتيب بيئات النشر بشكل نهائي.

---

## 📌 1. ملخص المشكلة الجذرية (Root Cause Analysis)
كانت تظهر مشكلة "فشل صامت" (Silent Failure) عند إضافة روابط البرامج في صفحة التعديل على سيرفر الـ Test، دون ظهور أي أخطاء في الواجهة أو في سجلات المتصفح. بعد تشخيص معمق، تبين أن السبب كان مزيجاً من 3 عوامل:
1. **ابتلاع الاستثناءات (Swallowed Exceptions):** كانت دوال `GetAsync` و `PostAsync` في `WebApiService.cs` تعيد `null` بصمت عند فشل الطلب (مثل 404 أو 500) بدلاً من رمي `Exception`، مما منع واجهة Blazor من عرض رسائل الخطأ.
2. **خطأ في توجيه البيئة (Misconfigured BaseUrl):** كان ملف `appsettings.Test.json` على السيرفر (ومحلياً) يشير إلى `https://api.rubikcare.com` (وهو الـ API الخاص بالـ Live)، بينما كان المفروض أن يشير إلى بيئة الـ Test/UAT.
3. **تضارب بيئات الـ API:** وجود نسخ متعددة من الـ API على السيرفر (Live و UAT) دون توثيق واضح، مما أدى إلى نشر كود الـ Web الجديد بينما كان الـ API المستهدف هو النسخة القديمة (Live) التي لا تحتوي على Endpoint الخاص بـ `/api/psp/links`.

---

## 🗺️ 2. خريطة البيئات المعتمدة (Environment Map)
يجب الالتزام بهذه الخريطة بدقة عند أي عملية نشر أو تعديل:

| البيئة | الغرض | رابط الـ Web | رابط الـ API | مسار السيرفر (Physical Path) |
| :--- | :--- | :--- | :--- | :--- |
| **Local** | التطوير والاختبار المبدئي | `https://localhost:xxxx` | `https://localhost:yyyy` | `C:\Users\shady\source\repos\...\` |
| **Test / UAT** | الاختبار الشامل قبل النشر | `https://test.rubikcare.com` | `https://uat.rubikcare.com` | Web: `C:\WebSite\RubikCareNew`<br>API: `C:\WebSite\RubikCareUat` |
| **Production (Live)** | البيئة الفعلية للمستخدمين | `https://rubikcare.com` | `https://api.rubikcare.com` | Web: `C:\WebSite\RubikCareLive`<br>API: `C:\WebSite\RubicCareApi` |

⚠️ **قاعدة ذهبية:** تطبيق الـ Web في بيئة الـ Test **يجب** أن يكون `ApiSettings:BaseUrl` فيه مساوياً لـ `https://uat.rubikcare.com` وليس الـ Live.

---

## 🛠️ 3. الإصلاحات البرمجية التي تم تطبيقها (Code Fixes)
1. **في `WebApiService.cs` (مشروع Web):** 
   - تم تعديل دوال `GetAsync` و `PostAsync` لرمي `Exception` يحتوي على `StatusCode` ومحتوى الخطأ الفعلي من الـ API بدلاً من العودة بـ `default` بصمت.
2. **في `PSPStep_Links.razor`:**
   - تم إضافة حماية (Guard Clause) للتأكد من أن `ProgramID` لا يساوي `0` قبل محاولة الإرسال، مع توجيه المستخدم لحفظ البرنامج الأساسي أولاً.
3. **في `web.config` (على سيرفر الـ Test):**
   - تم إضافة `<environmentVariable name="ASPNETCORE_ENVIRONMENT" value="Test" />` لضمان قراءة ملف `appsettings.Test.json`.
   - تم تفعيل `stdoutLogEnabled="true"` للمساعدة في تتبع أخطاء المصادقة (Tokens) مستقبلاً.

---

## 🚀 4. إجراءات النشر الآمن (Standard Operating Procedure - SOP)

عند الحاجة لنشر تحديث جديد، **يجب** اتباع هذا الترتيب بدقة لتجنب تخريب الإعدادات:

### أ. نشر تحديث الـ API (يتم أولاً دائماً)
1. من Visual Studio: Publish لمشروع `Api.Web` إلى مجلد محلي (مثلاً `C:\PublishApi`).
2. على سيرفر الـ Test: 
   - أوقف الـ App Pool الخاص بالـ Test API: `Stop-WebAppPool -Name "RubikCareUat"`
   - انسخ ملفات المجلد المحلي إلى `C:\WebSite\RubikCareUat` (اختر Replace All).
   - شغل الـ App Pool مجدداً: `Start-WebAppPool -Name "RubikCareUat"`

### ب. نشر تحديث الـ Web
1. من Visual Studio: Publish لمشروع `Rubikcare.Web` إلى مجلد محلي (مثلاً `C:\PublishWeb`).
2. على سيرفر الـ Test:
   - أوقف الـ App Pool الخاص بالـ Web: `Stop-WebAppPool -Name "rubikcarenew"`
   - انسخ الملفات إلى `C:\WebSite\RubikCareNew`.
   - ⚠️ **تحذير هام:** إذا ظهر تحذير لاستبدال `web.config` أو `appsettings.Test.json`، تأكد أن النسخة المحلية محدثة (تشير إلى `uat.rubikcare.com`)، أو اختر **Skip** لهما للحفاظ على إعدادات السيرفر اليدوية.
   - شغل الـ App Pool مجدداً: `Start-WebAppPool -Name "rubikcarenew"`

---

## 🔍 5. أوامر تشخيص سريعة (Troubleshooting Cheatsheet)
في حالة تكرار أي مشكلة، نفذ هذه الأوامر على سيرفر الـ Test فوراً:

```powershell
# 1. التأكد من أن الـ Web يقرأ رابط الـ API الصحيح
Get-Content "C:\WebSite\RubikCareNew\appsettings.Test.json" | Select-String "BaseUrl"

# 2. التأكد من أن بيئة الـ Test مفعلة في web.config
Get-Content "C:\WebSite\RubikCareNew\web.config" | Select-String "ASPNETCORE_ENVIRONMENT"

# 3. فحص آخر أخطاء الـ Blazor/Server في السجلات
Get-ChildItem -Path "C:\WebSite\RubikCareNew\logs\stdout" -Filter "*.log" | Sort-Object LastWriteTime -Descending | Select-Object -First 1 | Get-Content -Tail 30
