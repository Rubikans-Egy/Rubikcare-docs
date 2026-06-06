```markdown
# 06 - منهجية موحدة لتتبع وحل المشاكل (Troubleshooting)

آخر تحديث: 6 يونيو 2026

## مقدمة

في رحلة تطوير روبيك كير، واجهنا العديد من المشاكل التقنية. من مشاكل الترجمة في `ExternalLogin` إلى مشاكل الأداء في الصفحات المترجمة، تعلمنا أن المشكلة الحقيقية ليست في الخطأ نفسه، بل في كيفية تتبعه وحله.

هذا المرجع يقدم منهجية موحدة من 7 خطوات يمكن تطبيقها على أي مشكلة في المشروع.

## 🎯 فلسفة حل المشاكل: "Console First"

### المبدأ الأساسي
قبل أن تغير أي كود، قبل أن تضيف أي سطر، قبل أن تفكر في الحل:
                ⭐ افتح Console المتصفح (F12) ⭐

لماذا؟

- 90% من المشاكل تظهر أخطاءها في Console
- الـ Network Tab يظهر كل طلبات API
- الـ Elements Tab يظهر بنية HTML الفعلية

## 📋 الخطوات السبع الذهبية لتتبع المشاكل

```
graph TD
    A[1. Console First] --> B[2. اعزل المشكلة]
    B --> C[3. تحقق من المصدر]
    C --> D[4. تتبع التدفق]
    D --> E[5. اختبر المتغيرات]
    E --> F[6. جرّب الحل في معزل]
    F --> G[7. وثّق الحل]
```

### الخطوة 1: Console First - ابدأ هنا دائماً
```javascript
// في أي صفحة تواجه مشكلة، ابدأ بهذه الأوامر في Console

// 1. هل هناك أخطاء حمراء؟
// انظر إلى Console مباشرة - أي خطأ أحمر هو دليل مباشر

// 2. هل الدوال المطلوبة موجودة؟
console.log('toggleLanguageSimple:', typeof window.toggleLanguageSimple);
console.log('loginLocalization:', typeof window.loginLocalization);
console.log('updateTranslations:', typeof window.loginLocalization?.updateTranslations);

// 3. هل البيانات المطلوبة موجودة؟
console.log('__initialTranslations:', window.__initialTranslations);
console.log('__currentLang:', window.__currentLang);

// 4. تحقق من Network Tab
// انتقل إلى Network tab -> XHR -> شاهد الطلبات والاستجابات
```

### الخطوة 2: اعزل المشكلة - هل هي في Server أم Client؟

#### اختبار API مباشرة
```javascript
// بدلاً من افتراض أن المشكلة في HTML، اختبر API مباشرة

// لصفحات الترجمة
fetch('/api/localization/page/LOGIN?lang=en')
    .then(r => r.json())
    .then(d => console.log('LOGIN API response:', d));

fetch('/api/localization/page/COMMON?lang=en')  
    .then(r => r.json())
    .then(d => console.log('COMMON API response:', d));
```

### الخطوة 3: تحقق من البيانات في المصدر (قاعدة البيانات)
```sql
-- 1. هل البيانات موجودة أصلاً؟
SELECT ResourceKey, ResourceValueAr, ResourceValueEn, Module, IsActive
FROM Resources
WHERE ResourceKey IN ('COMMON.SUCCESS', 'LOGIN.TITLE', 'EXTERNAL_LOGIN.VerifiedVia');
```

### الخطوة 4: تتبع التدفق - أضف logs في كل خطوة
```csharp
// في الـ Server (C#)
public async Task<Dictionary<string, string>> GetPageTranslationsAsync(string pageDomain, string language)
{
    Console.WriteLine($"📥 GetPageTranslationsAsync called for {pageDomain} in {language}");
    // ... باقي الكود
}
```

### الخطوة 5: اختبر المتغيرات المشتركة
```javascript
// في Console
console.log('🔄 localStorage contents:');
console.log('- Language:', localStorage.getItem('RubikCare:Language'));
console.log('- Direction:', localStorage.getItem('RubikCare:Direction'));
console.log('🍪 Cookies:', document.cookie);
console.log('📄 HTML attributes:');
console.log('- lang:', document.documentElement.lang);
console.log('- dir:', document.documentElement.dir);
```

### الخطوة 6: جرّب الحل في معزل (Isolate and Test)
```javascript
// جرّب تحديث الترجمة يدوياً في Console
await window.loginLocalization.updateTranslations('en');
```

### الخطوة 7: وثّق الحل (Document the Fix)
```
## 📝 سجل المشاكل والحلول

### المشكلة: [تاريخ] [وصف مختصر]
**الخطأ:** [ما الذي كان يحدث؟]
**التشخيص:** [كيف تتبعت المشكلة؟]
**السبب الجذري:** [لماذا حدثت المشكلة؟]
**الحل:** [كيف حُلّت؟]
**الملفات المتأثرة:** [أسماء الملفات]
**الدروس المستفادة:** [كيف نمنع حدوثها مستقبلاً؟]
```

---

## ⭐ دليل شامل: نظام الملاحة والتوجيه في MAUI Shell

**تاريخ التحديث:** 6 يونيو 2026

### 📌 القواعد الذهبية لتسجيل الصفحات

#### 1. الصفحات الرئيسية (Root Pages) - 6 صفحات فقط

| الصفحة | Route | الملف |
|--------|-------|-------|
| تسجيل الدخول | `LoginPage` | `AppShell.xaml` |
| إنشاء حساب | `RegisterPage` | `AppShell.xaml` |
| الرئيسية (Personal) | `MainDashboard` | `AppShell.xaml` |
| لوحة العيادة | `ClinicDashboardPage` | `AppShell.xaml` |
| لوحة الصيدلية | `PharmacyDashboardPage` | `AppShell.xaml` |
| لوحة شركة الأدوية | `PharmaCompanyDashboardPage` | `AppShell.xaml` |

**تسجل في:** `AppShell.xaml` فقط كـ `<ShellContent>`
**تستدعى بـ:** `//Route` (شرطتان مائلتان)

#### 2. الصفحات الفرعية (Detail Pages) - كل ما عدا الستة

**تسجل في:** `AppShell.xaml.cs` فقط باستخدام `Routing.RegisterRoute`
**تستدعى بـ:** `Route` (بدون شرطات)

#### 3. جدول أنواع التنقل الكامل

| الرمز | النوع | التأثير | يستخدم مع | مثال |
|-------|-------|---------|-----------|------|
| `//Route` | Absolute | يستبدل الصفحة الحالية في المكدس | الصفحات الرئيسية | `//MainDashboard` |
| `///Route` | Absolute كامل | يمسح المكدس بالكامل | ❌ ممنوع الاستخدام | - |
| `Route` | Relative | يضيف صفحة فوق المكدس | الصفحات الفرعية | `doctorprofile` |
| `..` | Back | يرجع صفحة للخلف | أي صفحة | `..` |
| `Route?param=value` | Relative + معاملات | يضيف صفحة مع بيانات | الصفحات الفرعية | `doctorprofile?clinicId=5` |

---

### 🔴 أخطاء التنقل الشائعة - الدليل الكامل

#### الخطأ 1: `Ambiguous routes matched for...`

**الرسالة:**
```
System.ArgumentException: Ambiguous routes matched for: 
//.../pageName matches found: //.../pageName, //.../pageName
```

**ماذا يعني:**
الصفحة مسجلة مرتين - في `AppShell.xaml` و `AppShell.xaml.cs` معاً.

**أين تبحث:**
- `AppShell.xaml` ← ابحث عن `<ShellContent Route="اسم_الصفحة">`
- `AppShell.xaml.cs` ← ابحث عن `Routing.RegisterRoute("اسم_الصفحة", ...)`

**الحل:**
احذف التسجيل من أحد الملفين. إذا كانت الصفحة رئيسية → احذف من `xaml.cs`. إذا كانت فرعية → احذف من `xaml`.

---

#### الخطأ 2: `JavaProxyThrowable`

**الرسالة:**
```
Android.Runtime.JavaProxyThrowable: Exception of type 
'Android.Runtime.JavaProxyThrowable' was thrown.
```

**متى يظهر:**
عند العودة من صفحة إلى أخرى (زر الرجوع).

**ماذا يعني:**
يحاول Android تدمير `WebView` أو `Fragment` لكن كائن C# المرتبط به تم جمعه مسبقاً.

**الأسباب المحتملة:**
1. تعارض إصدارات `WebView.Maui` مع `MAUI.Controls` (الأكثر شيوعاً)
2. `Ambiguous Routes` (يظهر معه في نفس الوقت)

**أين تبحث:**
1. `RubikCare.Mobile.csproj` ← تأكد من تطابق إصدارات MAUI
2. `AppShell.xaml` + `AppShell.xaml.cs` ← ابحث عن تسجيل مزدوج

---

#### الخطأ 3: `No view found for id 0x7f0800ff (jumpToEnd)`

**الرسالة:**
```
Java.Lang.IllegalArgumentException: No view found for id 0x7f0800ff 
(jumpToEnd) for fragment NavigationRootManager_ElementBasedFragment
```

**متى يظهر:**
بعد Splash Screen مباشرة، قبل تحميل أي صفحة. التطبيق ينهار فوراً.

**ماذا يعني:**
استخدام `///` مع صفحة رئيسية أدى إلى تدمير `Fragment` الداخلي لـ Shell نفسه. Android لم يعد يجد الـ View الخاص بـ Shell.

**الأسباب المحتملة (مرتبة حسب الأولوية):**

| # | السبب | مثال خاطئ |
|---|-------|-----------|
| 1 | `///` مع صفحة رئيسية | `GoToAsync("///ClinicDashboardPage")` |
| 2 | مسار نسبي مع صفحة رئيسية | `GoToAsync("MainDashboard")` بدون `//` |
| 3 | حذف كود XAML معلق دون تنظيف | ترك `xmlns` references مكسورة في الملف |
| 4 | تسجيل رئيسية في `AppShell.xaml.cs` | `Routing.RegisterRoute("MainDashboard", ...)` |

**أين تبحث (الملفات المرشحة):**

| الملف | ماذا تبحث عنه |
|-------|---------------|
| `AppShellViewModel.cs` | `GoToAsync("///...")` أو `GoToAsync("//...")` خطأ |
| `AppShell.xaml` | `<ShellContent>` زائد أو ناقص |
| `AppShell.xaml.cs` | `Routing.RegisterRoute` لصفحة رئيسية |
| أي `ViewModel` جديد | `GoToAsync` بـ `///` |

**الحل:**
1. استبدل كل `///Route` بـ `//Route` للصفحات الرئيسية
2. استبدل كل مسار نسبي لصفحة رئيسية بـ `//Route`
3. نفذ تنظيف عميق:
```bash
rmdir /s /q bin
rmdir /s /q obj
# احذف التطبيق من الهاتف
dotnet restore
dotnet build -f net10.0-android -c Debug -t:Install
```

---

#### الخطأ 4: `Global routes currently cannot be the only page on the stack`

**الرسالة:**
```
System.Exception: Global routes currently cannot be the only page on the stack, 
so absolute routing to global routes is not supported. 
For now, just navigate to: /PageName
```

**ماذا يعني:**
تحاول استخدام `//` مع صفحة **فرعية** (مسجلة في `AppShell.xaml.cs`). الصفحات الفرعية لا تدعم `//`.

**أين تبحث:**
- أي `ViewModel` فيه `GoToAsync("//صفحة_فرعية")`

**الحل:**
احذف `//` واستخدم المسار النسبي فقط.

---

#### الخطأ 5: `unable to figure out route for...`

**الرسالة:**
```
System.ArgumentException: unable to figure out route for: 
doctorprofile?clinicId=1245
```

**ماذا يعني:**
الصفحة غير مسجلة في `AppShell.xaml.cs` أو أن المسار غير معروف في السياق الحالي.

**أين تبحث:**
- `AppShell.xaml.cs` ← هل `Routing.RegisterRoute` موجود؟
- طريقة الاستدعاء ← هل تستخدم `//` بالخطأ؟

---

### 📊 مصفوفة التشخيص السريع

| الخطأ | أول ملف تراجعه | ثاني ملف | ثالث ملف |
|-------|---------------|---------|----------|
| Ambiguous routes | `AppShell.xaml` | `AppShell.xaml.cs` | - |
| JavaProxyThrowable | `AppShell.xaml.cs` | `.csproj` | `AppShell.xaml` |
| jumpToEnd | `AppShellViewModel.cs` | `AppShell.xaml` | `AppShell.xaml.cs` |
| Global routes... | `ViewModel` المعني | `AppShell.xaml.cs` | - |
| unable to figure out | `AppShell.xaml.cs` | `ViewModel` المعني | - |

---

### 🛡️ إجراءات الوقاية

1. **قبل أي Commit:** تأكد من عدم وجود `///` في المشروع كله
2. **بعد حذف كود XAML معلق:** تنظيف عميق إجباري
3. **عند إضافة صفحة جديدة:** اسأل: "هل هي من الـ 6 الرئيسية؟"
4. **القاعدة النهائية:**
   ```
   رئيسية؟ → AppShell.xaml → //Route
   فرعية؟  → AppShell.xaml.cs → Route
   رجوع؟   → ..
   ///     → ممنوع نهائياً
   ```

---

## 🚀 أمثلة تطبيق المنهجية على مشاكل حقيقية

### مثال 1: مشكلة الترجمة في صفحة ExternalLogin

| الخطوة | التطبيق |
|---------|----------|
| **1. Console First** | `Uncaught SyntaxError: Invalid or unexpected token` |
| **2. اعزل المشكلة** | `fetch('/api/localization/page/COMMON?lang=en')` ← يعود ببيانات صحيحة |
| **3. تحقق من المصدر** | `SELECT * FROM Resources WHERE ResourceKey LIKE '%SUCCESS%'` ← موجود |
| **4. تتبع التدفق** | أضف logs في `login-localization.js` ← وجدت نسختين متعارفتين |
| **5. اختبر المتغيرات** | `window.__initialTranslations` ← undefined بسبب SyntaxError |
| **6. جرّب الحل في معزل** | أنشئ صفحة HTML تجريبية بنفس الكود ← نفس المشكلة |
| **7. وثّق الحل** | سجلت الحل في سجل المشاكل |

---

## ⚠️ تحذيرات ومحاذير

### 🔴 ممنوعات مطلقة أثناء حل المشاكل

1. **لا تغير الكود عشوائياً** - كل تغيير يجب أن يكون مبنيًا على تشخيص
2. **لا تحذف ملفات الـ Migration** - استخدم `Add-Migration` جديد
3. **لا تعدل قاعدة البيانات مباشرة** - استخدم Code-First
4. **لا تهمل Console** - 90% من المشاكل تظهر هناك أولاً

### 🟡 ممارسات سيئة

| الممارسة السيئة | لماذا؟ | الأفضل |
|-----------------|--------|--------|
| "أظن أن المشكلة في كذا" | بدون دليل | اختبر الفرضية أولاً |
| تغيير 10 أسطر دفعة واحدة | لا تعرف أي سطر حل المشكلة | غيّر سطراً واحداً واختبر |
| إضافة `!important` عشوائياً | يسبب مشاكل توريث لاحقاً | ابحث عن السبب الحقيقي |
| تجاهل Cache المتصفح | قد تحل المشكلة وحدها | `Ctrl+Shift+R` للتحديث الكامل |

---

## 📚 أدوات التشخيص الموصى بها

| الأداة | الاستخدام | اختصار |
|--------|-----------|--------|
| **Console (F12)** | الأخطاء، logs، اختبار الدوال | `F12` ← Console |
| **Network Tab** | مراقبة طلبات API، وقت الاستجابة | `F12` ← Network |
| **Elements Tab** | فحص بنية HTML، الـ CSS المطبق | `F12` ← Elements |
| **Application Tab** | localStorage، cookies، session | `F12` ← Application |
| **SQL Server Profiler** | تتبع استعلامات قاعدة البيانات | أدوات SQL Server |
| **Postman** | اختبار APIs بمعزل عن التطبيق | تطبيق منفصل |

---

## ✅ CHECKLIST: عند مواجهة أي مشكلة

- [ ] **الخطوة 1:** فتح Console (F12) ← هل هناك أخطاء حمراء؟
- [ ] **الخطوة 2:** Network Tab ← هل الطلبات تذهب للأماكن الصحيحة؟
- [ ] **الخطوة 3:** Application Tab ← هل localStorage و cookies صحيحة؟
- [ ] **الخطوة 4:** اختبر API مباشرة (fetch في Console)
- [ ] **الخطوة 5:** أضف logs في الكود (Server و Client)
- [ ] **الخطوة 6:** اختبر في صفحة معزولة (isolated)
- [ ] **الخطوة 7:** ابحث عن المشكلة في سجل المشاكل السابقة
- [ ] **الخطوة 8:** جرّب `Ctrl+Shift+R` (تحديث كامل مع مسح cache)
- [ ] **الخطوة 9:** إذا وجدت الحل، وثّقه فوراً

---

### ⭐ مشكلة: JavaProxyThrowable عند زر الرجوع للخلف (Android)

**تاريخ الحل:** 25 مايو 2026

**الأعراض:**
- ضغط زر الرجوع في Android يغلق التطبيق بدلاً من الرجوع للصفحة السابقة
- عند إعادة فتح التطبيق بعد الإغلاق يظهر الخطأ:
  ```
  Android.Runtime.JavaProxyThrowable
  Method not found: void Microsoft.Maui.LifecycleEvents.LifecycleEventService.RemoveEvent
  ```

**التشخيص:**
الخطأ `Method not found` يشير إلى أن حزمة `Microsoft.AspNetCore.Components.WebView.Maui` تحاول استدعاء دالة `RemoveEvent` في `LifecycleEventService` غير موجودة في إصدار `Microsoft.Maui.Controls` المثبت.

**السبب:** تعارض إصدارات حزم MAUI:
- `Microsoft.Maui.Controls` = 10.0.20
- `Microsoft.AspNetCore.Components.WebView.Maui` = 10.0.51 ← إصدار مختلف

**الحل:**
توحيد إصدار `WebView.Maui` مع `Microsoft.Maui.Controls` في ملف `.csproj`:
```xml
<PackageReference Include="Microsoft.Maui.Controls" Version="10.0.20" />
<PackageReference Include="Microsoft.AspNetCore.Components.WebView.Maui" Version="10.0.20" />
```

**القاعدة الذهبية:**
`Microsoft.AspNetCore.Components.WebView.Maui` يجب أن يكون دائماً على نفس إصدار `Microsoft.Maui.Controls`.

---

### ⭐ مشكلة: Ambiguous Routes (تضارب المسارات) في MAUI Shell

**تاريخ الحل:** 26 مايو 2026

**الخطأ:**
```
System.ArgumentException: Ambiguous routes matched for: 
//.../pageName matches found: //.../pageName, //.../pageName
```

**الأعراض:** يحدث هذا الخطأ غالباً مع `JavaProxyThrowable` عند محاولة العودة إلى صفحة سابقة.

**التشخيص:**
1. افحص `AppShell.xaml` و `AppShell.xaml.cs`
2. إذا وجدت نفس الصفحة مسجلة في كليهما، فهذا هو السبب

**السبب الجذري:** تسجيل نفس الصفحة (Route) في كل من `ShellContent` داخل `AppShell.xaml` و `Routing.RegisterRoute` في `AppShell.xaml.cs`.

**الحل:** اعتماد هيكلة صارمة للملاحة:
- الصفحات الرئيسية (Root) تسجل في `AppShell.xaml` فقط
- كل ما عداها يسجل في `AppShell.xaml.cs`

**الوقاية:** عند إضافة أي صفحة جديدة، اسأل نفسك: "هل هي صفحة رئيسية (Root)؟" إذا كانت الإجابة لا، سجلها في `AppShell.xaml.cs` فقط ولا تستخدم `//` للتنقل إليها.

---

## 🔗 روابط ذات صلة

- [05 - إنشاء الصفحات والمكونات](05-page-creation-checklist.md)
- [07 - نظام برامج دعم المرضى](07-psp-system.md)
- [10 - دليل تطوير MAUI](10-maui-development-guide.md)
- [Problem Log](../problem-log.md)
```

---

هذا هو الملف الكامل جاهز للنسخ والنشر. ارفعه مباشرة إلى:
`https://github.com/Rubikans-Egy/Rubikcare-docs/blob/main/docs/06-troubleshooting-methodology.md`
