
# 🌐 Marketing Website — Architecture & Operations Guide

**آخر تحديث:** 23 سبتمبر 2026
**الأولوية:** 🔴 حرج
**النطاق:** الموقع التسويقي العام (rubikcare.com)

---

## 📌 مقدمة

هذا المرجع يوثّق **الموقع التسويقي لـ RubikCare** — بشكل مستقل عن التطبيق الداخلي وعن تطبيق الموبايل. يغطي:

- **الفلسفة التصميمية** وأهداف الموقع
- **البنية المعمارية** الفعلية
- **أساليب العمل** الإلزامية
- **قائمة الصفحات** والمكوّنات
- **نظام الترجمة** الخاص بالموقع
- **ما تم إنجازه** و**ما لم يتم بعد**

**الجمهور المستهدف من هذه الوثيقة:**
- المطورون العاملون على الموقع
- المصممون المساهمون في التوسعة
- المهتمون بفهم البنية قبل إضافة صفحات جديدة

---

## 🎯 الجزء الأول: فلسفة الموقع

### 1.1 الموقع ليس تطبيقًا

**الموقع التسويقي يختلف جذريًا عن التطبيق الداخلي:**

| الجانب | التطبيق الداخلي | الموقع التسويقي |
|--------|------------------|-----------------|
| **الجمهور** | مستخدمون مسجّلون | زوار غير مسجّلين |
| **الهدف** | إنجاز مهام | إقناع + تحويل |
| **التفاعل** | عالٍ (نماذج، لوحات تحكم) | منخفض (قراءة، تصفح) |
| **الأداء** | Circuit Interactive | Static SSR + Interactive محدود |
| **التخطيط** | `MainLayout` | `MarketingLayout` |
| **الألوان** | `--rubik-*` | `--rc-*` + `--hp-*` |
| **التخطيط** | Sidebar + Header | Header + Footer فقط |

**النتيجة:** لا يُخلط بين الاثنين — لا في الكود ولا في الأصول.

### 1.2 الجمهور الأساسي — شركات الأدوية

**الموقع موجّه بالأساس إلى:**
1. **شركات الأدوية** — صنّاع القرار التجاري
2. **الفرق القانونية / Compliance** — لصفحة الامتثال
3. **الأطباء والصيدليات** — للانضمام
4. **المرضى** — للتثقيف

**التركيز:**
- **PSP كخدمة أساسية** — وليس منتجًا عامًا
- **الأرقام والنتائج** — بدلًا من الشعارات
- **الامتثال** — كعنصر أساسي، وليس كإضافة

### 1.3 المبادئ البصرية

| # | المبدأ | التفصيل |
|---|--------|---------|
| **1** | **هوية موحّدة** | أزرق `#1B5A7A` + أخضر `#2E7D5E` + بيج `#F1F0E9` |
| **2** | **Serif للعناوين** | `Fraunces` + `Markazi Text` للعربية |
| **3** | **Sans للنصوص** | `IBM Plex Sans` + `Cairo` للعربية |
| **4** | **Hero بصورة متلاشية** | صورة على يمين/يسار — تتلاشى نحو النص |
| **5** | **Zigzag layouts** | للقصص المتعددة — كـ How It Works |
| **6** | **بطاقات بنمط Dashboard** | للأرقام والإحصائيات |
| **7** | **RTL/LTR حقيقي** | ليس ترجمة سطحية |

---

## 🏛️ الجزء الثاني: البنية المعمارية

### 2.1 بنية المشروع

**الموقع التسويقي جزء من `Rubikcare.Web`** — لكنه معزول عن التطبيق الداخلي:

```
Rubikcare.Web/
├── Components/
│   ├── Layout/
│   │   ├── MainLayout.razor              ← للتطبيق الداخلي
│   │   ├── InteractiveMenu.razor         ← تفاعلي للتطبيق
│   │   ├── MarketingLayout.razor         ← للموقع التسويقي (Static)
│   │   ├── MarketingHeader.razor         ← تفاعلي
│   │   ├── MarketingFooter.razor         ← تفاعلي
│   │   └── LanguageSwitcher.razor        ← تفاعلي
│   │
│   ├── Pages/
│   │   ├── Website/                      ← صفحات الموقع التسويقي
│   │   │   ├── Home.razor
│   │   │   ├── PharmaceuticalCompanies.razor
│   │   │   ├── Physicians.razor
│   │   │   ├── Pharmacies.razor
│   │   │   ├── Patients.razor
│   │   │   ├── Compliance.razor
│   │   │   ├── Demo.razor
│   │   │   └── About.razor
│   │   │
│   │   └── ... (صفحات التطبيق الداخلي)
│   │
│   └── Base/
│       └── BasePage.cs                   ← للصفحات المترجمة
│
└── wwwroot/
    └── Images/
        └── own/                          ← كل صور الموقع
            ├── hero-pharmacy.png
            ├── patient-hero.jpg
            ├── pharmacy-hero.jpg
            ├── physicians-hero.png
            ├── pharma-hero.jpg
            ├── data-protection.jpg
            ├── rubikcare-office.jpg
            ├── vision-patient.jpg
            ├── purpose-doctor.jpg
            ├── today-cairo.jpg
            ├── dashboard-meeting.jpg
            ├── manufacturer.jpg
            ├── doctor.jpg
            ├── patient.jpg
            ├── pharmacy.jpg
            ├── step-1-invite.png
            ├── step-2-token.png
            ├── step-3-schedule.png
            └── step-4-dispensed.png
```

### 2.2 بنية CSS

**كل التنسيقات في `Shared.UI/wwwroot/css/_pages/website/`:**

```
Shared.UI/wwwroot/css/
├── _website-bundle.css                   ← Bundle رئيسي للموقع
│
└── _pages/website/
    ├── sitevariables.css                 ← متغيرات التصميم
    ├── sitelayout.css                    ← Header/Footer/Nav
    ├── _homepage.css                     ← Home
    ├── pharmaceutical-companies.css      ← Pharma
    ├── physicians.css                    ← Physicians
    ├── pharmacies.css                    ← Pharmacies
    ├── patients.css                      ← Patients
    ├── compliance.css                    ← Compliance
    ├── demo.css                          ← Demo
    └── about.css                         ← About
```

**`_website-bundle.css`:**

```css
@import url('_pages/website/sitevariables.css');
@import url('_pages/website/sitelayout.css');
@import url('_pages/website/_homepage.css');
@import url('_pages/website/pharmaceutical-companies.css');
@import url('_pages/website/physicians.css');
@import url('_pages/website/pharmacies.css');
@import url('_pages/website/patients.css');
@import url('_pages/website/compliance.css');
@import url('_pages/website/demo.css');
@import url('_pages/website/about.css');
```

**تسجيل في `main.css`:**

```css
@import url('_website-bundle.css');
```

### 2.3 نمط المكوّنات

**الموقع يعتمد على نمط "Static Layout + Interactive Children":**

```
MarketingLayout.razor (Static)
├── <MarketingHeader />        ← Interactive (rendermode InteractiveServer)
├── <main>@Body</main>          ← حسب الصفحة
└── <MarketingFooter />        ← Interactive
```

**السبب:**
- `LayoutComponentBase` لا يقبل `@rendermode` (لأنه يستقبل `RenderFragment`)
- الحل: نقل التفاعل إلى مكوّنات فرعية

**نفس النمط في `MainLayout`:** `InteractiveMenu` مكوّن تفاعلي.

---

## 🔧 الجزء الثالث: أساليب العمل الإلزامية

### 3.1 قواعد الصفحات

| # | القاعدة | السبب |
|---|---------|-------|
| **1** | كل صفحة ترث `BasePage` | للترجمة + `@T()` |
| **2** | كل صفحة تستخدم `@layout MarketingLayout` | للتناسق |
| **3** | كل صفحة تحدد `GetPageDomain()` | لنطاق الترجمة |
| **4** | كل صفحة تحمل `@attribute [AllowAnonymous]` | للجمهور العام |
| **5** | كل صفحة تحمل `@rendermode InteractiveServer` | للتفاعل |

**مثال:**

```razor
@page "/about"
@inherits BasePage
@layout MarketingLayout
@rendermode InteractiveServer
@attribute [AllowAnonymous]

<PageTitle>@T("ABOUT.PAGE_TITLE")</PageTitle>

@code {
    protected override string GetPageDomain() => "ABOUT";
}
```

### 3.2 قواعد CSS

| # | القاعدة | السبب |
|---|---------|-------|
| **1** | **لا `<style>` داخل `.razor`** | مخالف للمعمارية |
| **2** | كل صفحة لها ملف CSS في `_pages/website/` | تنظيم |
| **3** | Namespace واضح لكل صفحة (`rc-phys-*`, `rc-phar-*`, ...) | تجنّب التعارض |
| **4** | `@import url('...')` في `_website-bundle.css` | تسجيل مركزي |
| **5** | استخدام `html[dir="rtl"]` — وليس `[dir="rtl"]` | يعمل دائمًا |
| **6** | استخدام `inset-inline-*` — وليس `left/right` | RTL/LTR تلقائي |
| **7** | استخدام `var(--rc-*)` — وليس قيم ثابتة | اتساق |

### 3.3 قواعد الترجمة

| # | القاعدة | السبب |
|---|---------|-------|
| **1** | **لا نصوص Hardcoded** — كل شيء `@T("...")` | الترجمة |
| **2** | **بعد إضافة مفاتيح جديدة** — أعد تشغيل التطبيق | مسح Cache |
| **3** | **`N''` في SQL** — للعربية | ترميز |
| **4** | **`MERGE` — وليس `INSERT`** | تجنّب التكرار |
| **5** | **`ResourceKey + Module` في `MERGE`** | للفهرس |
| **6** | **بعد التعديلات — امسح cache المتصفح** | ضمان الظهور |

### 3.4 قواعد الصور

| # | القاعدة | السبب |
|---|---------|-------|
| **1** | **كل الصور في `wwwroot/Images/own/`** | تنظيم |
| **2** | **صيغة `.jpg` أو `.png`** | لا `.gif` أو `.bmp` |
| **3** | **`loading="lazy"`** — للصور غير Hero | الأداء |
| **4** | **`loading="eager"`** — لصور Hero فقط | الأولوية |
| **5** | **حجم أقصى 500KB** | الأداء |
| **6** | **نسبة 4:3 أو 16:9** — للأفقي | التناسق |
| **7** | **نسبة 9:19.5** — للموبايل | التناسق |

### 3.5 قواعد Hero

**كل صفحة لها Hero بالمعايير التالية:**

| # | العنصر | القيمة |
|---|--------|--------|
| **1** | **`min-height`** | 560-600px |
| **2** | **الصورة** | 48-55% من العرض |
| **3** | **`object-fit`** | `cover` |
| **4** | **التلاشي** | `mask-image` على الحافة الداخلية |
| **5** | **اتجاه التلاشي** | يتغير مع اللغة (`html[dir="rtl"]` vs `ltr`) |
| **6** | **عكس الصورة** | ❌ **لا** — إلا إذا كانت بلا نصوص |

**⚠️ قاعدة حرجة:** الصور التي تحتوي **نصوصًا عربية** — **لا تُعكس**.

---

## 📋 الجزء الرابع: الصفحات

### 4.1 قائمة الصفحات

| # | الصفحة | المسار | الحالة | Domain |
|---|--------|--------|--------|--------|
| **1** | Home | `/` | ✅ مكتملة | `HOME` |
| **2** | Pharmaceutical Companies | `/pharmaceutical-companies` | ✅ مكتملة | `PHARMA` |
| **3** | Physicians | `/physicians` | ✅ مكتملة | `PHYSICIANS` |
| **4** | Pharmacies | `/pharmacies` | ✅ مكتملة | `PHARMACIES` |
| **5** | Patients | `/patients` | ✅ مكتملة | `PATIENTS` |
| **6** | Compliance | `/compliance` | ✅ مكتملة | `COMPLIANCE` |
| **7** | Book a Demo | `/demo` | ✅ مكتملة | `DEMO` |
| **8** | About | `/about` | ✅ مكتملة | `ABOUT` |

### 4.2 الصفحات المؤجلة

| # | الصفحة | السبب |
|---|--------|-------|
| **1** | Resources | تحوّل إلى About |
| **2** | Articles / Insights | يحتاج محتوى ضخم |
| **3** | Privacy / Terms | موجودتان في `Shared.UI/Components/Legal/` — تحتاجان ربطًا |
| **4** | Contact | مدموجة في Demo |
| **5** | Roadmap | سرّي — للمستقبل |
| **6** | Clinics | مستقبلي — إشارة خفيفة فقط |

### 4.3 المكوّنات المشتركة

| # | المكوّن | الاستخدام |
|---|---------|-----------|
| **1** | `MarketingHeader` | في كل صفحة تسويقية |
| **2** | `MarketingFooter` | في كل صفحة تسويقية |
| **3** | `LanguageSwitcher` | داخل Header |
| **4** | `BasePage` | ترثه كل صفحة |

---

## 🌐 الجزء الخامس: نظام الترجمة في الموقع

### 5.1 المبادئ

**راجع الوثيقة الشاملة:** `15-translation-system.md`

**الملخص للصفحات التسويقية:**

| # | المبدأ |
|---|--------|
| **1** | **الافتراضي `en`** — وليس `ar` |
| **2** | **اللغة محفوظة** في `localStorage` + `Cookie` |
| **3** | **`@T()`** — عبر `BasePage` |
| **4** | **`GetPageDomain()`** — يحدد نطاق الترجمة |
| **5** | **`COMMON`** — للعناصر المشتركة |

### 5.2 نطاقات الترجمة

| النطاق | الصفحة |
|--------|--------|
| `HOME.*` | Home |
| `PHARMA.*` | Pharmaceutical Companies |
| `PHYSICIANS.*` | Physicians |
| `PHARMACIES.*` | Pharmacies |
| `PATIENTS.*` | Patients |
| `COMPLIANCE.*` | Compliance |
| `DEMO.*` | Demo |
| `ABOUT.*` | About |
| `COMMON.NAV.*` | Header |
| `COMMON.FOOTER.*` | Footer |

### 5.3 إضافة مفاتيح جديدة

```sql
MERGE Resources AS target
USING (VALUES
    (N'ABOUT.VISION.TITLE',
     N'تمكين الناس من إدارة صحتهم لحياة أفضل.',
     N'Empowering people to manage their health for a better quality of life.',
     N'ABOUT', N'Title')
) AS source (ResourceKey, ResourceValueAr, ResourceValueEn, Module, ResourceType)
ON target.ResourceKey = source.ResourceKey
WHEN MATCHED THEN UPDATE SET ... 
WHEN NOT MATCHED THEN INSERT ...;
```

**⚠️ مهم:**
- `ResourceKey` — **فريد عالميًا** (الفهرس الحالي)
- `Module` — للمجموعة
- `N''` — إلزامي للعربية

---

## ✅ الجزء السادس: ما تم إنجازه

### 6.1 الصفحات المكتملة

| # | الصفحة | المحتوى |
|---|--------|---------|
| **1** | **Home** | Hero + 4 Parties (بصور) + How It Works (Zigzag) + Why RubikCare + MFG Teaser + CTA |
| **2** | **Pharma Companies** | Hero + Comparison Table + ROI Calculator (Adherence Impact) + Physician Uplift (68%) + Adherence Stats (4) + Timeline + FAQ + CTA |
| **3** | **Physicians** | Hero + 3-Step Flow + 4-Stat Adherence Dashboard (3 أشرطة + Donut) + CTA |
| **4** | **Pharmacies** | Hero + Verification Flow (3 خطوات) + 4 Benefits + CTA |
| **5** | **Patients** | Hero + 4-Step App Flow + More Than a Code (3 بطاقات بصور) + Privacy (بصورة) + CTA |
| **6** | **Compliance** | Hero + 8 PDPL Requirements + Pharmacovigilance (4 خطوات) + Security (4 مبادئ) + FAQ (5 أسئلة) + Downloads |
| **7** | **Demo** | Hero + What Happens Next (3 خطوات) + Form (5 حقول + 2 اختياري) + Contacts (3) + Not a Manufacturer (3 مسارات) |
| **8** | **About** | Hero + Vision + Purpose + Core Values (RUBIK) + Targets (5) + Inside RubikCare + Today (background) |

### 6.2 المكوّنات المكتملة

| # | المكوّن | الميزة |
|---|---------|--------|
| **1** | **MarketingHeader** | تفاعلي + مترجم + Dropdown + Mobile Menu |
| **2** | **MarketingFooter** | تفاعلي + مترجم |
| **3** | **LanguageSwitcher** | تبديل فوري + يحفظ الحالة |
| **4** | **BasePage** | `@T()` + `OnLanguageChanged` |

### 6.3 نظام الترجمة

| # | الميزة |
|---|--------|
| **1** | **3 مصادر متزامنة** — localStorage + Cookie + html lang |
| **2** | **لا قفزات** — SSR و Interactive متسقان |
| **3** | **`ILocalizationService`** — للترجمات |
| **4** | **`PersistentCache`** — للأداء |

### 6.4 توثيق

| # | الوثيقة |
|---|---------|
| **1** | `15-translation-system.md` — شامل |
| **2** | `16-marketing-website.md` — هذه الوثيقة |

---

## ⏳ الجزء السابع: ما لم يتم بعد

### 7.1 صفحات مؤجلة

| # | الصفحة | الأولوية |
|---|--------|----------|
| **1** | **Privacy** | 🔴 عاجل — ربط في Footer |
| **2** | **Terms** | 🔴 عاجل — ربط في Footer |
| **3** | **Articles / Insights** | 🟡 متوسط — يحتاج محتوى |
| **4** | **Contact (مستقلة)** | 🟢 مدموجة في Demo |

### 7.2 ميزات ناقصة

| # | الميزة | الأولوية |
|---|--------|----------|
| **1** | **Demo Form → API** | 🔴 عاجل — حاليًا `Task.Delay` |
| **2** | **Pharmacy Registration Form** | 🟡 متوسط |
| **3** | **Analytics** | 🟡 متوسط |
| **4** | **Cookie Banner (GDPR)** | 🟡 متوسط |
| **5** | **Blog / News** | 🟢 منخفض |

### 7.3 تحسينات تقنية

| # | التحسين | الأولوية |
|---|---------|----------|
| **1** | **`[email protected]`** — حل Cloudflare | 🔴 عاجل |
| **2** | **فهرس `ResourceKey + Module`** | 🟡 متوسط |
| **3** | **اختبارات وحدة** | 🟡 متوسط |
| **4** | **قياس أداء Lighthouse** | 🟢 متوسط |
| **5** | **ADRs (Architecture Decision Records)** | 🟢 منخفض |

---

## 🚀 الجزء الثامن: النشر والصيانة

### 8.1 عملية النشر

| # | الإجراء |
|---|---------|
| **1** | **نفّذ SQL** — كل المفاتيح الجديدة |
| **2** | **تأكد من الصور** — رفعت في `wwwroot/Images/own/` |
| **3** | **`git commit`** — رسالة واضحة |
| **4** | **`git push`** — للمستودع |
| **5** | **انتظر CI/CD** — نشر تلقائي |
| **6** | **امسح cache الخادم** — للأداء |
| **7** | **امسح cache المتصفح** — للصور |
| **8** | **تحقق** — كل صفحة في اللغتين |

### 8.2 الصيانة الدورية

| # | المهمة | الدورية |
|---|--------|---------|
| **1** | **مراجعة الأخطاء** — من Logs | أسبوعيًا |
| **2** | **فحص الأداء** — Lighthouse | شهريًا |
| **3** | **تحديث المحتوى** — حسب الحاجة | حسب المشروع |
| **4** | **إضافة مفاتيح ترجمة** — عند الحاجة | مستمر |

### 8.3 إضافة صفحة جديدة — الخطوات

**الترتيب الإلزامي:**

| # | الخطوة |
|---|--------|
| **1** | **إنشاء `PageName.razor`** في `Components/Pages/Website/` |
| **2** | **الوراثة** — `@inherits BasePage` |
| **3** | **Layout** — `@layout MarketingLayout` |
| **4** | **`@rendermode InteractiveServer`** |
| **5** | **`@attribute [AllowAnonymous]`** |
| **6** | **`GetPageDomain()`** — نطاق الترجمة |
| **7** | **إنشاء `pagename.css`** في `_pages/website/` |
| **8** | **إضافة `@import`** في `_website-bundle.css` |
| **9** | **إضافة مفاتيح الترجمة** — عبر SQL `MERGE` |
| **10** | **إضافة روابط** — في `MarketingHeader` و `MarketingFooter` |
| **11** | **إعادة تشغيل التطبيق** |
| **12** | **مسح cache المتصفح** |

---

## 📊 الجزء التاسع: إحصائيات

### 9.1 أرقام المشروع

| # | المقياس | القيمة |
|---|---------|--------|
| **1** | عدد الصفحات المكتملة | 8 |
| **2** | عدد المكوّنات المشتركة | 4 |
| **3** | عدد ملفات CSS | 10 |
| **4** | عدد مفاتيح الترجمة (تقدير) | ~400 |
| **5** | عدد الصور | ~20 |
| **6** | عدد Modules | 9 |

### 9.2 التغطية

| # | اللغة | التغطية |
|---|--------|---------|
| **1** | **الإنجليزية** | 100% |
| **2** | **العربية** | 100% |
| **3** | **RTL/LTR** | يعمل تلقائيًا |

---

## 🔗 روابط ذات صلة

- [00 - الهيكل المعماري العام](../00-architecture.md)
- [15 - نظام الترجمة الشامل](15-translation-system.md)
- [05 - إنشاء الصفحات والمكونات](05-page-creation-checklist.md)
- [02 - نظام الهوية والمصادقة](02-identity-system.md)

---

**آخر تحديث:** 23 سبتمبر 2026
**الملف:** `16-marketing-website.md`
**الحالة:** ✅ مرجع مُعتمد — يُحدَّث مع كل توسعة
```

---

## 📌 ملاحظات على الوثيقة

### 1. لماذا هذا الاسم؟

**`16-marketing-website.md`** — لأنه:
- **مستقل** — وليس مدمجًا مع 15
- **مرقّم** — في السلسلة
- **واضح** — Marketing Website

### 2. ما يميز هذه الوثيقة

| # | الميزة |
|---|--------|
| **1** | **شاملة** — من الفلسفة إلى الصيانة |
| **2** | **عملية** — قواعد إلزامية |
| **3** | **صادقة** — تُظهر ما لم يتم |
| **4** | **قابلة للتنفيذ** — خطوات واضحة |
| **5** | **قابلة للتحديث** — مع كل توسعة |

### 3. ما يُحدَّث لاحقًا

| # | القسم |
|---|--------|
| **1** | **قائمة الصفحات** — عند إضافة مقالات |
| **2** | **الأرقام** — إحصائيات محدثة |
| **3** | **"ما لم يتم"** — تصبح "ما تم" |
| **4** | **الروابط** — للوثائق المستقبلية |

---
