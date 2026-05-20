# 📘 **دليل نظام الترجمة والاتجاه - RubikCare**
**الإصدار:** 1.0.0  
**آخر تحديث:** 14 فبراير 2026  
**الغرض:** توثيق تفاعل الملفات وآلية العمل للتطوير المستقبلي

---

## 🧩 **1. ملفات JavaScript (الأساسية)**

### **1.1 direction-protector.js**
| الخاصية | القيمة |
|---------|--------|
| **المسار** | `wwwroot/Assets/js/direction-protector.js` |
| **الوظيفة** | حماية اتجاه الصفحة من المسح أثناء التنقل |
| **نوع التشغيل** | تلقائي عند تحميل الصفحة |
| **التفاصيل** | يراقب التغييرات على `dir` ويعيد تعيينه من `localStorage` |

### **1.2 accessibility.js**
| الخاصية | القيمة |
|---------|--------|
| **المسار** | `wwwroot/Assets/js/accessibility.js` |
| **الوظيفة** | إعدادات الوصولية (تم تعطيل الثيم الداكن) |
| **التعديل** | تم تعطيل `settings.theme` لفرض الوضع الفاتح |

### **1.3 observability.js**
| الخاصية | القيمة |
|---------|--------|
| **المسار** | `wwwroot/Assets/js/observability.js` |
| **الوظيفة** | مراقبة الأداء والأخطاء (للمطورين) |
| **التفعيل** | عبر Console المتصفح |

---

## 🗄️ **2. ملفات C# Services**

### **2.1 TranslationStateService.cs**
| الخاصية | القيمة |
|---------|--------|
| **المسار** | `Data/Services/Localization/TranslationStateService.cs` |
| **المسؤولية** | مصدر الحقيقة الوحيد للغة |
| **المفاتيح** | `RubikCare:Language` في `localStorage` |
| **الأحداث** | `OnLanguageChanged` |

### **2.2 LayoutDirectionService.cs**
| الخاصية | القيمة |
|---------|--------|
| **المسار** | `Data/Services/Localization/LayoutDirectionService.cs` |
| **المسؤولية** | حساب الاتجاه من اللغة (`ar` → `rtl`, `en` → `ltr`) |
| **التبعية** | يعتمد على `TranslationStateService` |

### **2.3 UserSessionService.cs**
| الخاصية | القيمة |
|---------|--------|
| **المسار** | `Data/Services/Session/UserSessionService.cs` |
| **المسؤولية** | حفظ/تحميل تفضيلات المستخدم (اللغة، الاتجاه) |

---

## 🎨 **3. ملفات CSS المؤثرة في الاتجاه**

### **3.1 _reset.css**
| الخاصية | القيمة |
|---------|--------|
| **المسار** | `wwwroot/css/_core/_reset.css` |
| **التعديل الرئيسي** | **تم حذف** كل قواعد فرض الاتجاه |
| **الوضع الحالي** | لا يوجد `direction` أو `[dir]` rules |

### **3.2 _sidebar.css**
| الخاصية | القيمة |
|---------|--------|
| **المسار** | `wwwroot/css/_layout/_sidebar.css` |
| **الخصائص** | تستخدم `inset-inline-start` (منطقية) |
| **الاستثناء** | نصوص القائمة بيضاء (`color: white !important`) |

### **3.3 _variables.css**
| الخاصية | القيمة |
|---------|--------|
| **المسار** | `wwwroot/css/_core/_variables.css` |
| **دعم الاتجاه** | `:root[dir="rtl"]` و `:root[dir="ltr"]` لتغيير المتغيرات |

---

## 🔄 **4. ملفات Razor Components**

### **4.1 InteractiveMenu.razor**
| الخاصية | القيمة |
|---------|--------|
| **المسار** | `Components/Layout/InteractiveMenu.razor` |
| **الدوال الرئيسية** | `HandleLanguageChanged`، `ApplyDirectionToBodyAsync` |
| **التفاعل** | يستمع لحدث `OnLanguageChanged` من `TranslationStateService` |

### **4.2 LanguageSwitcher.razor**
| الخاصية | القيمة |
|---------|--------|
| **المسار** | `Components/Layout/LanguageSwitcher.razor` |
| **الوظيفة** | واجهة تبديل اللغة |
| **الآلية** | يستدعي `TranslationState.SetLanguageAsync()` |

### **4.3 App.razor**
| الخاصية | القيمة |
|---------|--------|
| **المسار** | `App.razor` |
| **المحتوى** | يحتوي على `<script>` لتحميل ملفات JS بالترتيب |
| **الترتيب** | `direction-protector.js` ← `observability.js` |

---

## 📦 **5. ملفات التخزين (localStorage)**

| المفتاح | النوع | المثال | المصدر |
|---------|------|--------|--------|
| `RubikCare:Language` | `string` | `"ar"`, `"en"` | `TranslationStateService` |
| `RubikCare:Direction` | `string` | `"rtl"`, `"ltr"` | احتياطي (نادر الاستخدام) |
| `RubikCare:MenuMode` | `string` | `"Personal"`, `"Admin"` | `UserSessionService` |
| `RubikCare:OrganizationId` | `string` | `"1"` | `UserSessionService` |

---
[#98]

أهلاً بك شادي. **سؤال ممتاز!** الآن بعد أن أصبح نظام الترجمة والاتجاه مستقراً، حان وقت **ترحيل النصوص الثابتة**.

---

## 📋 **6. خطة ترحيل النصوص الثابتة إلى نظام الترجمة**

### **الملفات المؤثرة:**
1. **`LocalizationService.cs`** - يقرأ الترجمات من قاعدة البيانات
2. **`TranslationStateService.cs`** - يدير اللغة الحالية
3. **`LocalizedText.razor`** - المكون الذي يعرض النص المترجم
4. **`Resources`** - جدول قاعدة البيانات (2,434 ترجمة)

---

## ✅ **7. الخطوة الأولى: فهم أنواع النصوص الثابتة**

### **النوع أ: نصوص في عناوين الصفحات**
```razor
<h3>قائمة المدن</h3>  ← يجب استبدالها
```

### **النوع ب: نصوص في الأزرار**
```razor
<button class="btn btn-primary">حفظ</button>  ← يجب استبدالها
```

### **النوع ج: نصوص في الجداول (Headers)**
```razor
<th>الاسم</th>  ← يجب استبدالها
```

### **النوع د: نصوص في التسميات (Labels)**
```razor
<label>البريد الإلكتروني</label>  ← يجب استبدالها
```

---

## 🛠️ **8. الخطوة الثانية: المكونات الجاهزة للاستخدام**

### **2.1 LocalizedText.razor (الأساسي)**
```razor
@* Components/Shared/RubikCare/LocalizedText.razor *@
@inject LocalizationService Localizer

@Localizer.Get(ResourceKey)

@code {
    [Parameter] public string ResourceKey { get; set; } = "";
    [Parameter] public string? DefaultText { get; set; }
}
```

**الاستخدام:**
```razor
<LocalizedText ResourceKey="Cities.List.Title" />
```

### **2.2 LocalizedLabel.razor (للتسميات)**
```razor
@* Components/Shared/RubikCare/LocalizedLabel.razor *@
<label for="@For">
    <LocalizedText ResourceKey="@ResourceKey" DefaultText="@DefaultText" />
</label>

@code {
    [Parameter] public string For { get; set; } = "";
    [Parameter] public string ResourceKey { get; set; } = "";
    [Parameter] public string? DefaultText { get; set; }
}
```

### **2.3 LocalizedButton.razor (للأزرار)**
```razor
@* Components/Shared/RubikCare/LocalizedButton.razor *@
<button class="@ButtonClass" @onclick="OnClick">
    <LocalizedText ResourceKey="@ResourceKey" DefaultText="@DefaultText" />
</button>

@code {
    [Parameter] public string ResourceKey { get; set; } = "";
    [Parameter] public string? DefaultText { get; set; }
    [Parameter] public string ButtonClass { get; set; } = "btn btn-primary";
    [Parameter] public EventCallback OnClick { get; set; }
}
```

---

## 📝 **9. الخطوة الثالثة: قائمة المفاتيح (Resource Keys) المطلوبة**

### **للصفحات العامة:**
```
Common.Save = "حفظ"
Common.Cancel = "إلغاء"
Common.Search = "بحث"
Common.Add = "إضافة جديد"
Common.Edit = "تعديل"
Common.Delete = "حذف"
Common.Export = "تصدير"
Common.Import = "استيراد"
```

### **لصفحة المدن (مثال):**
```
Cities.Title = "قائمة المدن"
Cities.Add = "إضافة مدينة"
Cities.Edit = "تعديل مدينة"
Cities.Delete = "حذف مدينة"
Cities.NameAr = "اسم المدينة (عربي)"
Cities.NameEn = "اسم المدينة (إنجليزي)"
Cities.Country = "الدولة"
Cities.Status = "الحالة"
Cities.Active = "نشط"
Cities.Inactive = "غير نشط"
```

### **للصفحات الطبية:**
```
Medical.Specialities = "التخصصات"
Medical.Diseases = "الأمراض"
Medical.Medications = "الأدوية"
Medical.LabTests = "التحاليل"
```

---

## 🔄 **10. الخطوة الرابعة: خطة الترحيل التدريجي**

### **اليوم 1: الصفحات الأساسية (الأكثر استخداماً)**
| الصفحة | عدد النصوص | الأولوية |
|--------|-----------|----------|
| `/admin/cities/list` | 15-20 نص | ⭐⭐⭐⭐⭐ |
| `/admin/areas/list` | 15-20 نص | ⭐⭐⭐⭐⭐ |
| `/admin/countries/list` | 15-20 نص | ⭐⭐⭐⭐⭐ |

### **اليوم 2: الصفحات الطبية**
| الصفحة | عدد النصوص | الأولوية |
|--------|-----------|----------|
| `/admin/medical/specialities/list` | 20-25 نص | ⭐⭐⭐⭐ |
| `/admin/medical/diseases/list` | 20-25 نص | ⭐⭐⭐⭐ |
| `/admin/medical/medications/list` | 25-30 نص | ⭐⭐⭐⭐ |

### **اليوم 3: الصفحات الإدارية**
| الصفحة | عدد النصوص | الأولوية |
|--------|-----------|----------|
| `/admin/users/list` | 20-25 نص | ⭐⭐⭐ |
| `/admin/organizations/list` | 20-25 نص | ⭐⭐⭐ |
| `/admin/system/translations/list` | 15-20 نص | ⭐⭐ |

---

## 🛠️ **11. الخطوة الخامسة: أداة مساعدة للكشف عن النصوص الثابتة**

### **في observability.js، أضف هذه الدالة:**
```javascript
// كشف النصوص الثابتة في الصفحة
window.rubikcare.findHardcodedTexts = function() {
    const elements = document.querySelectorAll('h1, h2, h3, h4, h5, h6, p, span, label, button, th, td, a:not([href*="/"])');
    const arabicTexts = [];
    
    elements.forEach(el => {
        const text = el.innerText?.trim();
        if (text && /[\u0600-\u06FF]/.test(text)) { // يحتوي على عربي
            // تجاهل العناصر التي تستخدم LocalizedText
            if (!el.closest('[data-localized="true"]')) {
                arabicTexts.push({
                    tag: el.tagName,
                    text: text,
                    class: el.className,
                    html: el.outerHTML.substring(0, 100)
                });
            }
        }
    });
    
    console.table(arabicTexts);
    return arabicTexts;
};
```

**الاستخدام:**
```javascript
// في Console المتصفح
window.rubikcare.findHardcodedTexts()
```

---

## 📌 **12. الخطوة السادسة: مثال عملي لتحويل صفحة**

### **قبل:**
```razor
@page "/admin/cities/list"

<h3>قائمة المدن</h3>

<table class="table">
    <thead>
        <tr>
            <th>الاسم (عربي)</th>
            <th>الاسم (إنجليزي)</th>
            <th>الدولة</th>
            <th>الحالة</th>
            <th>إجراءات</th>
        </tr>
    </thead>
    <tbody>
        @foreach (var city in cities)
        {
            <tr>
                <td>@city.CityNameAr</td>
                <td>@city.CityNameEn</td>
                <td>@city.Country.CountryNameAr</td>
                <td>
                    <span class="badge @(city.IsActive ? "bg-success" : "bg-secondary")">
                        @(city.IsActive ? "نشط" : "غير نشط")
                    </span>
                </td>
                <td>
                    <button class="btn btn-sm btn-primary" @onclick="() => Edit(city)">تعديل</button>
                    <button class="btn btn-sm btn-danger" @onclick="() => Delete(city)">حذف</button>
                </td>
            </tr>
        }
    </tbody>
</table>
```

### **بعد:**
```razor
@page "/admin/cities/list"
@inject LocalizationService Localizer

<h3><LocalizedText ResourceKey="Cities.Title" /></h3>

<table class="table">
    <thead>
        <tr>
            <th><LocalizedText ResourceKey="Cities.NameAr" /></th>
            <th><LocalizedText ResourceKey="Cities.NameEn" /></th>
            <th><LocalizedText ResourceKey="Cities.Country" /></th>
            <th><LocalizedText ResourceKey="Common.Status" /></th>
            <th><LocalizedText ResourceKey="Common.Actions" /></th>
        </tr>
    </thead>
    <tbody>
        @foreach (var city in cities)
        {
            <tr>
                <td>@city.CityNameAr</td>
                <td>@city.CityNameEn</td>
                <td>@city.Country.CountryNameAr</td>
                <td>
                    <span class="badge @(city.IsActive ? "bg-success" : "bg-secondary")">
                        @(city.IsActive ? Localizer.Get("Common.Active") : Localizer.Get("Common.Inactive"))
                    </span>
                </td>
                <td>
                    <button class="btn btn-sm btn-primary" @onclick="() => Edit(city)">
                        <LocalizedText ResourceKey="Common.Edit" />
                    </button>
                    <button class="btn btn-sm btn-danger" @onclick="() => Delete(city)">
                        <LocalizedText ResourceKey="Common.Delete" />
                    </button>
                </td>
            </tr>
        }
    </tbody>
</table>
```

---

## 📊 **13. الخطوة السابعة: التحقق من اكتمال الترجمة**

### **في observability.js، أضف:**
```javascript
// التحقق من اكتمال الترجمة في الصفحة
window.rubikcare.checkTranslationCompleteness = function() {
    const elements = document.querySelectorAll('h1, h2, h3, h4, h5, h6, p, span, label, button, th, td');
    let total = 0;
    let untranslated = [];
    
    elements.forEach(el => {
        const text = el.innerText?.trim();
        if (text && /[\u0600-\u06FF]/.test(text)) {
            total++;
            // تحقق من وجود سمة data-localized أو استخدام LocalizedText
            if (!el.closest('[data-localized="true"]') && 
                !el.className.includes('localized') &&
                !el.hasAttribute('data-localized')) {
                untranslated.push({
                    tag: el.tagName,
                    text: text,
                    class: el.className
                });
            }
        }
    });
    
    console.log(`📊 إجمالي النصوص: ${total}`);
    console.log(`❌ نصوص غير مترجمة: ${untranslated.length}`);
    console.table(untranslated);
    
    return { total, untranslated };
};
```

---

## 🎯 **14. الخطوة الثامنة: خطة العمل النهائية**

| اليوم | المهمة | المخرجات |
|-------|--------|----------|
| 1 | إضافة أداة الكشف عن النصوص | `window.rubikcare.findHardcodedTexts()` |
| 2 | ترجمة الصفحات الأساسية (مدن، مناطق، دول) | 3 صفحات كاملة |
| 3 | ترجمة الصفحات الطبية (تخصصات، أمراض) | صفحتان |
| 4 | ترجمة الأدوية (أكبر صفحة) | صفحة واحدة |
| 5 | ترجمة صفحات المستخدمين والمؤسسات | صفحتان |
| 6 | ترجمة الصفحات الإدارية المتبقية | 3-4 صفحات |
| 7 | اختبار شامل وتصحيح | جميع الصفحات |

---



---

**ابدأ بصفحة المدن (`/admin/cities/list`) - هي الأسهل والأوضح.**


## ⚙️ **15. آلية التفاعل (Data Flow)**

### **6.1 عند تحميل الصفحة**
```
1. App.razor → تحميل direction-protector.js
2. direction-protector.js → يبدأ المراقبة (MutationObserver)
3. accessibility.js → يحمل إعدادات الوصولية (بدون ثيم)
4. InteractiveMenu.OnInitializedAsync → يحمل الجلسة
5. TranslationStateService.InitializeAsync → يقرأ RubikCare:Language
6. InteractiveMenu.OnAfterRenderAsync → يستدعي ApplyDirectionToBodyAsync
7. direction-protector.js → يؤكد صحة dir (شبكة أمان)
```

### **6.2 عند تغيير اللغة**
```
1. LanguageSwitcher.razor → المستخدم يضغط
2. TranslationStateService.SetLanguageAsync(lang) ← يحفظ في localStorage
3. TranslationStateService.OnLanguageChanged → يحدث
4. InteractiveMenu.HandleLanguageChanged ← يستمع
   - يقرأ اللغة الجديدة
   - يحسب الاتجاه (ar→rtl, en→ltr)
   - يستدعي ApplyDirectionToBodyAsync(direction)
5. direction-protector.js → يلتقط التغيير ويؤكده
```

### **6.3 عند التنقل بين الصفحات**
```
1. المستخدم يضغط رابط
2. Blazor يبدأ التنقل (SPA Navigation)
3. Blazor يحاول مسح dir (مشكلة داخلية)
4. direction-protector.js (MutationObserver) يلتقط التغيير فوراً
5. direction-protector.js يقرأ RubikCare:Language من localStorage
6. direction-protector.js يعيد تعيين dir الصحيح
7. المستخدم لا يرى أي قفزة أو تغيير
```

---

## 🛠️ **16. أدوات التصحيح (Debugging)**

### **7.1 في Console المتصفح**
```javascript
// التحقق من اللغة الحالية
localStorage.getItem('RubikCare:Language')

// التحقق من اتجاه HTML
document.documentElement.dir

// تشغيل/إيقاف المراقبة
rubikcare.observability.status()
rubikcare.observability.disable()
rubikcare.observability.enable()
```

### **7.2 في Elements (F12)**
- راقب عنصر `<html>` وتغيرات `dir`
- راقب عنصر `<aside class="interactive-sidebar">` وخاصية `inset-inline-start`

---

## ⚠️ **17. قواعد صارمة للتطوير المستقبلي**

| القاعدة | السبب |
|--------|-------|
| **لا تعدل `direction-protector.js`** | هو خط الدفاع الأخير ضد مسح الاتجاه |
| **لا تضف `dir` في HTML static** | `dir` يجب أن يدار عبر JS فقط |
| **لا تستخدم `!important` في CSS إلا للضرورة** | يسبب صعوبة في التصحيح |
| **لا تضع قواعد اتجاه في `_reset.css`** | هذا كان سبب المشكلة الأصلية |
| **استخدم الخصائص المنطقية** `inset-inline-start` بدل `left`/`right` | تدعم RTL/LTR تلقائياً |

### ⚠️ **تنبيهات مهمة**

1. **لا تترجم كل شيء دفعة واحدة** - اختر صفحة واحدة يومياً
2. **استخدم أداة الكشف** قبل وبعد الترجمة للتأكد
3. **اختبر اللغة بعد كل ترجمة** (عربي/إنجليزي)
4. **تأكد من وجود المفاتيح في قاعدة البيانات** قبل استخدامها
5. **احتفظ بنسخة احتياطية** من الصفحة قبل التعديل
---

## 📋 **18. قائمة التحقق (Checklist) عند إضافة صفحة جديدة**

- [ ] هل تستخدم `LocalizedText` للنصوص؟
- [ ] هل اختبرت الصفحة في وضع RTL (العربية)؟
- [ ] هل اختبرت الصفحة في وضع LTR (الإنجليزية)؟
- [ ] هل تأكدت من عدم وجود `dir` ثابت في HTML؟
- [ ] هل استخدمت خصائص CSS منطقية؟
- [ ] هل راجعت Console للتأكد من عدم وجود أخطاء؟

---

## 🔄 **20. تاريخ التعديلات (Changelog)**

| التاريخ | الإصدار | التعديل |
|---------|---------|---------|
| 13 فبراير 2026 | v0.1 | إضافة `direction-protector.js` |
| 13 فبراير 2026 | v0.2 | تعطيل الثيم الداكن في `accessibility.js` |
| 14 فبراير 2026 | v0.3 | تنظيف `_reset.css` من قواعد الاتجاه |
| 14 فبراير 2026 | v0.4 | إضافة `observability.js` |
| 14 فبراير 2026 | v1.0.0 | **إصدار مستقر** - كل المشاكل محلولة |

---

## 📞 **21. للدعم الفني**

عند مواجهة مشكلة:
1. افتح Console (F12)
2. تحقق من رسائل `🛡️ DIR` في direction-protector.js
3. تأكد من قيمة `RubikCare:Language` في localStorage
4. استخدم `rubikcare.observability.status()` لمعرفة حالة النظام
5. إذا استمرت المشكلة، راجع `_reset.css` أولاً



# 📘 **ملحق دليل الترجمة: نظام مسح النصوص التلقائي (Page Scanner)**
**الإصدار:** 1.0.0  
**آخر تحديث:** 14 فبراير 2026  
**الغرض:** أتمتة اكتشاف النصوص العربية في الصفحات وتسجيلها في قاعدة البيانات

---

## 🎯 **1. نظرة عامة على النظام**

صفحة **Page Translation Scanner** (`/admin/system/page-scanner`) تتيح لك:

- مسح أي صفحة من صفحات المشروع تلقائياً
- اكتشاف جميع النصوص العربية فيها
- عرضها في جدول منظم
- إضافة النصوص الجديدة إلى قاعدة بيانات الترجمة
- توليد مفاتيح (Keys) مقترحة لكل نص
- تصنيف النصوص حسب نوعها (زر، تسمية، عنوان...)

---

## 🧩 **2. الملفات المطلوبة**

| الملف | المسار | الوظيفة |
|-------|--------|---------|
| `PageTranslationScanner.razor` | `Components/Shared/Localization/` | واجهة المستخدم ومنطق الصفحة |
| `utils.js` أو `observability.js` | `wwwroot/Assets/js/` | دوال JavaScript لمسح النصوص |
| `Screens` | قاعدة البيانات | جدول الصفحات (موجود مسبقاً) |
| `Resources` | قاعدة البيانات | جدول الترجمات |

---

## ⚙️ **3. آلية العمل خطوة بخطوة**

### **3.1 تحميل قائمة الصفحات**
```
1. الصفحة تستخدم DbFactory.ExecuteWithNewContextAsync
2. تجلب البيانات من جدول Screens (حيث IsActive = 1 و ScreenPath موجود)
3. تعرضها في قائمة منسدلة مرتبة حسب Module ثم Name
```

### **3.2 مسح الصفحة المحددة**
```
1. المستخدم يختار صفحة ويضغط "مسح الصفحة"
2. C# تستدعي دالة JavaScript: rubikcare.scanPageForTexts(pageUrl)
3. JavaScript:
   - تجلب محتوى الصفحة باستخدام fetch
   - تحلل HTML وتستخرج النصوص العربية
   - تصنف النصوص حسب نوع العنصر (H1, Button, Label...)
   - ترجع مصفوفة من الكائنات إلى C#
```

### **3.3 معالجة النصوص المكتشفة**
```
1. لكل نص، تتحقق C# من وجوده في قاعدة البيانات
2. إذا كان موجوداً: تظهر علامة "مسجل" والمفتاح الحالي
3. إذا كان جديداً:
   - تولد مفتاحاً مقترحاً
   - تسمح بتعديل التصنيف والوحدة
   - تظهر علامة "جديد"
```

### **3.4 حفظ الترجمات**
```
1. حفظ فردي: زر Save بجانب كل نص جديد
2. حفظ جماعي: زر "حفظ جميع الجديدة"
3. عند الحفظ:
   - تُنشأ سجل جديد في جدول Resources
   - تتغير حالة النص إلى "مسجل"
```

---

## 📊 **4. هيكل البيانات**

### **4.1 DetectedText (كائن النص المكتشف)**
```csharp
class DetectedText
{
    string ArabicText;      // النص العربي
    string ElementType;     // نوع العنصر (H1, Button, Label...)
    int LineNumber;         // رقم السطر التقريبي
    string Category;        // التصنيف (UI, Button, Label...)
    string Module;          // الوحدة (Medical, Admin, PSP...)
    bool ExistsInDb;        // هل موجود في قاعدة البيانات؟
    string? ExistingKey;    // المفتاح الحالي (إن وجد)
    string? SuggestedKey;   // المفتاح المقترح
    string? FilePath;       // مسار الملف (اختياري)
}
```

### **4.2 PageInfo (معلومات الصفحة)**
```csharp
class PageInfo
{
    string Name;    // اسم الصفحة (عربي)
    string Url;     // مسار الصفحة
    string Module;  // الوحدة
}
```

---

## 🛠️ **5. كيفية الإعداد والتشغيل**

### **الخطوة 1: إضافة دوال JavaScript**

في `utils.js` أو `observability.js`، أضف:

```javascript
window.rubikcare = window.rubikcare || {};

window.rubikcare.scanPageForTexts = async function(pageUrl) {
    console.log(`🔍 Scanning: ${pageUrl}`);
    
    // 1. جلب الصفحة
    const response = await fetch(pageUrl);
    const html = await response.text();
    
    // 2. تحليل HTML
    const parser = new DOMParser();
    const doc = parser.parseFromString(html, 'text/html');
    
    // 3. استخراج النصوص
    const texts = [];
    const elements = doc.querySelectorAll('h1,h2,h3,h4,h5,h6,p,span,label,button,a,th,td');
    
    elements.forEach(el => {
        const text = el.innerText?.trim();
        if (text && text.length > 2 && /[\u0600-\u06FF]/.test(text)) {
            texts.push({
                arabicText: text,
                elementType: el.tagName,
                lineNumber: 0,
                category: 'General',
                module: 'Common'
            });
        }
    });
    
    return texts;
};

window.rubikcare.log = function(type, message) {
    console.log(`[${type}] ${message}`);
};
```

### **الخطوة 2: التأكد من الخدمات في Program.cs**
```csharp
builder.Services.AddScoped<ILocalizationService, LocalizationService>();
builder.Services.AddScoped<TranslationStateService>();
builder.Services.AddScoped<DbContextFactoryService>();
```

### **الخطوة 3: إضافة الصفحة إلى القائمة**
أضف رابط الصفحة في القائمة الجانبية أو في `GeneralSettingsHub`:
```html
<a href="/admin/system/page-scanner" class="btn btn-primary">
    <i class="bi bi-search-heart"></i> ماسح النصوص
</a>
```

---

## 🔍 **6. استكشاف الأخطاء وإصلاحها**

| المشكلة | السبب المحتمل | الحل |
|---------|---------------|------|
| القائمة المنسدلة فارغة | استعلام خاطئ أو جدول Screens فارغ | تحقق من بيانات الجدول يدوياً |
| زر المسح لا يعمل | دالة JavaScript غير مسجلة | تأكد من وجود `window.rubikcare.scanPageForTexts` |
| النصوص لا تظهر بعد المسح | خطأ في ربط البيانات | تأكد من استدعاء `StateHasChanged()` |
| أخطاء في حفظ الترجمات | مشكلة في DbContext | استخدم `ExecuteWithNewContextAsync` |

---

## 📝 **7. خطوات التفعيل الكاملة**

1. **تأكد من وجود جدول Screens** بالبيانات الصحيحة
2. **أضف دوال JavaScript** إلى `utils.js`
3. **أضف الصفحة** `PageTranslationScanner.razor` في المسار الصحيح
4. **أضف الخدمات** في `Program.cs` إن لم تكن موجودة
5. **اختبر** باختيار صفحة والضغط على Scan
6. **راجع Console** في المتصفح للتأكد من عدم وجود أخطاء

---

## 🚀 **8. تطويرات مستقبلية مقترحة**

- [ ] إضافة تصدير Excel لجميع النصوص المكتشفة
- [ ] إمكانية تعديل التصنيف والوحدة بشكل جماعي
- [ ] ربط مباشر مع صفحة إدارة الترجمات
- [ ] إضافة فلترة حسب الحالة (مسجل/جديد)
- [ ] إمكانية مسح جميع الصفحات دفعة واحدة

---

[#27] ممتاز! دعنا أولاً نفهم هيكل جدول `Resources` الحالي.

## 📋 **هيكل جدول Resources:**

```sql
-- استعلام لمعرفة أعمدة الجدول
SELECT 
    COLUMN_NAME,
    DATA_TYPE,
    IS_NULLABLE,
    CHARACTER_MAXIMUM_LENGTH
FROM INFORMATION_SCHEMA.COLUMNS
WHERE TABLE_NAME = 'Resources'
ORDER BY ORDINAL_POSITION;
```

**الرجاء تشغيل هذا الاستعلام وإرسال النتيجة لي** لأعرف بالضبط الأعمدة الموجودة.

# 🎯 **طريقة التسمية resourcekey :**



### **المستوى 1: المجال (Domain)**
```
PSP          ← برامج دعم المرضى
PHARMA       ← بوابة شركات الأدوية
CLINIC       ← بوابة العيادات
PHARMACY     ← بوابة الصيدليات
PATIENT      ← بوابة المرضى
COMMON       ← مشترك بين الكل
ADMIN        ← لوحة التحكم
```

### **المستوى 2: الصفحة (Page)**
```
PROGRAM_EDIT
PROGRAM_LIST
PROGRAM_VIEW
DASHBOARD
PROFILE
SETTINGS
```

### **المستوى 3: نوع النص (Text Type)**
```
TITLE        ← عناوين رئيسية
SUBTITLE     ← عناوين فرعية
BUTTON       ← أزرار
LABEL        ← تسميات الحقول
PLACEHOLDER  ← نصوص إرشادية في الحقول
MESSAGE      ← رسائل (نجاح، خطأ، تحذير)
TOOLTIP      ← تلميحات
TABLE_HEADER ← عناوين الجداول
EMPTY_STATE  ← حالة عدم وجود بيانات
LOADING      ← حالة التحميل
CONFIRM      ← رسائل تأكيد
BREADCRUMB   ← مسار التنقل
TAB          ← عناوين التبويبات
```

### **المستوى 4: المفتاح المحدد (Specific Key)**
```
NAME
DESCRIPTION
SAVE
CANCEL
DELETE
EDIT
VIEW
ACTIVE
INACTIVE
```

## 📝 **مثال لتكوين ResourceKey كامل:**
```
PSP.PROGRAM_EDIT.TITLE.NEW
PSP.PROGRAM_EDIT.BUTTON.SAVE
PSP.PROGRAM_EDIT.LABEL.PROGRAM_NAME
PSP.PROGRAM_EDIT.PLACEHOLDER.SEARCH_SPECIALTY
PSP.PROGRAM_EDIT.MESSAGE.SAVE_SUCCESS
PSP.PROGRAM_EDIT.EMPTY_STATE.NO_SPECIALTIES
```

## ✅ **مزايا هذا التصنيف:**

1. **منظم وهرمي** - يسهل فهم مكان استخدام كل نص
2. **قابل للتوسع** - يمكن إضافة مجالات وصفحات جديدة بسهولة
3. **موحد** - نفس النمط لكل الصفحات
4. **قابل للبحث** - يسهل العثور على النصوص في قاعدة البيانات

**بعد أن ترسل لي نتيجة استعلام هيكل الجدول، سأعدل الخطة حسب الأعمدة المتوفرة.**