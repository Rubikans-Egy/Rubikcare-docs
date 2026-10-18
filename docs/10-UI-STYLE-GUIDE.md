
# 10 — UI-STYLE-GUIDE (دليل الأنماط والتصميم)

**الإصدار:** 2.0 (Unified Edition)
**آخر تحديث:** 16 أكتوبر 2026
**الحالة:** ✅ مرجع شامل — يدمج 03 + 25 (CSS) + التنظيم الفعلي
**الجمهور:** فريق التطوير + المصممون + AI Assistants
**المرجع الإلزامي:** يُقرأ قبل أي تعديل على CSS أو الأنماط

---

## 📌 الجزء صفر: الفلسفة والرؤية

### 0.1 الهدف

> **"نظام أنماط موحّد لكل المنصات — بلا تكرار، بلا فوضى، بقابلية توسّع لا نهائية."**

### 0.2 المبادئ التصميمية

| # | المبدأ | التطبيق |
|:-:|:---|:---|
| **1** | **هوية موحّدة** | متغيرات `--rubik-*` عالمية |
| **2** | **تنظيم هرمي** | من الأعمق إلى الأعلى (Core → Components → Layout → Pages) |
| **3** | **Bundle Files** | ملفات `_*Bundle.css` للتجميع |
| **4** | **CSS Variables** | لا قيم Hardcoded |
| **5** | **Mobile-First** | تصميم للموبايل أولاً |
| **6** | **RTL/LTR** | دعم كامل للاتجاهين |
| **7** | **Touch-Friendly** | الحد الأدنى 48px |
| **8** | **Performance** | Lazy Loading + Bundle |

---

## 🎨 الجزء الأول: نظام الألوان العام

### 1.1 المتغيرات الأساسية (`--rubik-*`)

**الموقع:** `Shared.UI/wwwroot/css/_core/` (مُصدَّر عبر `_CoreBundle.css`)

| المتغير | القيمة | الاستخدام |
|---------|--------|-----------|
| `--rubik-primary` | `#1B5A7A` | اللون الأساسي |
| `--rubik-primary-light` | `#E8F3F9` | خلفيات فاتحة |
| `--rubik-primary-dark` | `#0D3D5A` | تدرج غامق |
| `--rubik-secondary` | `#2E7D5E` | اللون الثانوي |
| `--rubik-secondary-light` | `#EAF4EF` | خلفيات فاتحة ثانوية |
| `--rubik-gold` | `#B88B4A` | لون مميز (مندوب) |
| `--rubik-gold-light` | `#F7F0E4` | خلفيات ذهبية فاتحة |
| `--rubik-text` | `#1A2B38` | النصوص الرئيسية |
| `--rubik-text-muted` | `#7A9BB0` | النصوص الثانوية |
| `--rubik-bg` | `#F4F7F9` | خلفية الصفحات |
| `--rubik-white` | `#FFFFFF` | خلفية البطاقات |
| `--rubik-border` | `#E1E8EE` | حدود الفواصل |
| `--rubik-shadow` | `0 2px 12px rgba(0,0,0,0.08)` | الظلال |
| `--rubik-radius` | `16px` | الزوايا الأساسية |
| `--rubik-radius-sm` | `10px` | الزوايا الصغيرة |

### 1.2 المتغيرات الدلالية المتقدمة

| المتغير | القيمة | الاستخدام |
|---------|--------|-----------|
| `--rubik-danger` | `#B33F40` | رسائل الخطأ |
| `--rubik-danger-light` | `#FBEAEA` | خلفيات رسائل الخطأ |
| `--rubik-success` | `#2E7D5E` | رسائل النجاح |
| `--rubik-success-light` | `#E8F3E9` | خلفيات رسائل النجاح |
| `--rubik-warning` | `#B88B4A` | رسائل التحذير |
| `--rubik-warning-light` | `#FFF3E0` | خلفيات رسائل التحذير |
| `--rubik-info` | `#4A7B8C` | رسائل المعلومات |
| `--rubik-info-light` | `#E6F3F7` | خلفيات رسائل المعلومات |
| `--rubik-card-bg` | `#FFFFFF` | خلفية البطاقات |
| `--rubik-card-alt` | `#F9FAFC` | خلفية البطاقات البديلة |
| `--rubik-body-bg` | `#F5F7FA` | خلفية الصفحة |
| `--rubik-border-light` | `#EFF1F5` | حدود فاتحة |
| `--rubik-border-dark` | `#C5C9D0` | حدود داكنة |
| `--rubik-text-primary` | `#1A2B3C` | النصوص الأساسية |
| `--rubik-text-secondary` | `#546E7A` | النصوص الثانوية |
| `--rubik-text-muted` | `#8A9BA8` | النصوص الخافتة |

### 1.3 🆕 متغيرات الموقع التسويقي (`--rc-*` + `--hp-*`)

**⚠️ من `25-marketing-website`:**

| المتغير | القيمة | الاستخدام |
|---------|--------|-----------|
| `--rc-primary` | `#1B5A7A` | اللون الأساسي |
| `--rc-secondary` | `#2E7D5E` | اللون الثانوي |
| `--rc-beige` | `#F1F0E9` | البيج |
| `--hp-primary` | `#1B5A7A` | Hero Primary |
| `--hp-text` | `#1A2B38` | Hero Text |
| `--rc-font-serif` | `Fraunces`, `Markazi Text` | Serif للعناوين |
| `--rc-font-sans` | `IBM Plex Sans`, `Cairo` | Sans للنصوص |

**⚠️ الموقع التسويقي = هوية منفصلة عن التطبيق.**

---

## 🏥 الجزء الثاني: أنماط الأدوار

### 2.1 نمط العيادة (Clinic) — أزرق بحري + أزرق-أخضر

| المتغير | القيمة | الاستخدام |
|---------|--------|-----------|
| `--clinic-hero-start` | `#0A3D5C` | بداية تدرج الـ Hero |
| `--clinic-hero-end` | `#1A7A7A` | نهاية تدرج الـ Hero |
| `--clinic-primary` | `#1A7A7A` | اللون الأساسي |
| `--clinic-primary-light` | `#D1ECF1` | خلفيات فاتحة |
| `--clinic-primary-dark` | `#0A3D5C` | تدرج غامق |
| `--clinic-secondary` | `#2D8B6E` | اللون الثانوي |
| `--clinic-secondary-light` | `#D4EDDA` | خلفيات ثانوية فاتحة |
| `--clinic-gold` | `#B88B4A` | لون مميز |
| `--clinic-gold-light` | `#F7F0E4` | خلفيات ذهبية فاتحة |

**تدرج الـ Hero:**

```css
background: linear-gradient(135deg, #0A3D5C 0%, #1A7A7A 100%);
```

**أيقونات العيادة:**

| الأيقونة | اللون | الاستخدام |
|---------|-------|-----------|
| `clinic-action-icon-clinic` | `#D1ECF1` | أيقونة "عيادتي" |
| `clinic-action-icon-psp` | `#D4EDDA` | أيقونة "برامج الدعم" |
| `clinic-action-icon-patients` | `#D1ECF1` | أيقونة "مرضاي" |
| `clinic-action-icon-invite` | `#F7F0E4` | أيقونة "دعواتي" |

**ملف CSS:** `_pages/Clinic/clinic-dashboard.css`

### 2.2 نمط الصيدلية (Pharmacy) — أخضر غامق + زمردي

| المتغير | القيمة | الاستخدام |
|---------|--------|-----------|
| `--pharmacy-hero-start` | `#1A3A3A` | بداية تدرج الـ Hero |
| `--pharmacy-hero-end` | `#2D8B6E` | نهاية تدرج الـ Hero |
| `--pharmacy-primary` | `#2D8B6E` | اللون الأساسي |
| `--pharmacy-primary-light` | `#D4EDDA` | خلفيات فاتحة |
| `--pharmacy-primary-dark` | `#1A3A3A` | تدرج غامق |
| `--pharmacy-secondary` | `#1A7A7A` | اللون الثانوي |
| `--pharmacy-secondary-light` | `#D1ECF1` | خلفيات ثانوية فاتحة |
| `--pharmacy-gold` | `#B88B4A` | لون مميز |
| `--pharmacy-gold-light` | `#F7F0E4` | خلفيات ذهبية فاتحة |

**تدرج الـ Hero:**

```css
background: linear-gradient(135deg, #1A3A3A 0%, #2D8B6E 100%);
```

**ملف CSS:** `_pages/Pharmacy/pharmacy-dashboard.css`

### 2.3 نمط المندوب (Rep) — ذهبي

| المتغير | القيمة | الاستخدام |
|---------|--------|-----------|
| `--rep-hero-start` | `#3D2E1A` | بداية تدرج الـ Hero |
| `--rep-hero-end` | `#B88B4A` | نهاية تدرج الـ Hero |
| `--rep-primary` | `#B88B4A` | اللون الأساسي |
| `--rep-primary-light` | `#F7F0E4` | خلفيات فاتحة |
| `--rep-primary-dark` | `#3D2E1A` | تدرج غامق |
| `--rep-secondary` | `#1A7A7A` | اللون الثانوي |
| `--rep-secondary-light` | `#D1ECF1` | خلفيات ثانوية فاتحة |

**تدرج الـ Hero:**

```css
background: linear-gradient(135deg, #3D2E1A 0%, #B88B4A 100%);
```

**ملف CSS:** `_pages/Rep/rep-dashboard.css`

### 2.4 الصفحات الداخلية (Sub-pages)

**مبدأ التطبيق:** يجب أن تتبع الصفحات الداخلية نفس نظام ألوان الدور الرئيسي.

**كيفية التطبيق — استخدام المتغيرات العالمية مع الحفاظ على تناسق الألوان:**

**مثال:** `psp-gateway.css`

```css
.pg-root {
    --pg-primary: var(--clinic-primary, #1A7A7A);
    --pg-primary-light: var(--clinic-primary-light, #D1ECF1);
    --pg-primary-dark: var(--clinic-primary-dark, #0A3D5C);
    --pg-secondary: var(--clinic-secondary, #2D8B6E);
    --pg-secondary-light: var(--clinic-secondary-light, #D4EDDA);
    --pg-gold: var(--clinic-gold, #B88B4A);
    --pg-gold-light: var(--clinic-gold-light, #F7F0E4);
}
```

**بهذه الطريقة، الصفحات الداخلية ترث ألوان الدور الرئيسي تلقائياً.**

---

## 🌐 الجزء الثالث: أنماط الموقع التسويقي

### 3.1 الفرق عن التطبيق

| الجانب | التطبيق الداخلي | الموقع التسويقي |
|--------|------------------|-----------------|
| **الجمهور** | مستخدمون مسجّلون | زوار غير مسجّلين |
| **الهدف** | إنجاز مهام | إقناع + تحويل |
| **التخطيط** | `MainLayout` | `MarketingLayout` |
| **الألوان** | `--rubik-*` | `--rc-*` + `--hp-*` |
| **الخطوط** | Cairo | Cairo + Fraunces + IBM Plex |
| **التفاعل** | عالٍ | منخفض |

**⚠️ لا يُخلط بين الاثنين — لا في الكود ولا في الأصول.**

### 3.2 المتغيرات والخطوط

| العنصر | القيمة |
|--------|--------|
| **Serif للعناوين** | `Fraunces` + `Markazi Text` للعربية |
| **Sans للنصوص** | `IBM Plex Sans` + `Cairo` للعربية |
| **الأزرق الأساسي** | `#1B5A7A` |
| **الأخضر الثانوي** | `#2E7D5E` |
| **البيج** | `#F1F0E9` |

### 3.3 المبادئ البصرية للموقع

| # | المبدأ | التفصيل |
|:-:|:---|:---|
| **1** | **هوية موحّدة** | أزرق + أخضر + بيج |
| **2** | **Hero بصورة متلاشية** | صورة على يمين/يسار — تتلاشى نحو النص |
| **3** | **Zigzag layouts** | للقصص المتعددة |
| **4** | **بطاقات بنمط Dashboard** | للأرقام والإحصائيات |
| **5** | **RTL/LTR حقيقي** | ليس ترجمة سطحية |

### 3.4 بنية CSS للموقع

**المسار:** `Shared.UI/wwwroot/css/_pages/website/`

```
Shared.UI/wwwroot/css/
├── _website-bundle.css                   ← Bundle رئيسي
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

### 3.5 قواعد Hero في الموقع

| # | العنصر | القيمة |
|:-:|:---|:---|
| **1** | `min-height` | 560-600px |
| **2** | الصورة | 48-55% من العرض |
| **3** | `object-fit` | `cover` |
| **4** | التلاشي | `mask-image` على الحافة الداخلية |
| **5** | اتجاه التلاشي | يتغير مع اللغة (`html[dir="rtl"]` vs `ltr`) |
| **6** | عكس الصورة | ❌ **لا** — إلا إذا كانت بلا نصوص |

**⚠️ قاعدة حرجة:** الصور التي تحتوي **نصوصًا عربية** — **لا تُعكس**.

### 3.6 قواعد النشر للموقع

**عند إضافة صفحة تسويقية:**

| # | الخطوة |
|:-:|:---|
| **1** | إنشاء `PageName.razor` في `Components/Pages/Website/` |
| **2** | الوراثة — `@inherits BasePage` |
| **3** | Layout — `@layout MarketingLayout` |
| **4** | `@rendermode InteractiveServer` |
| **5** | `@attribute [AllowAnonymous]` |
| **6** | `GetPageDomain()` — نطاق الترجمة |
| **7** | إنشاء `pagename.css` في `_pages/website/` |
| **8** | إضافة `@import` في `_website-bundle.css` |
| **9** | إضافة مفاتيح الترجمة — عبر SQL `MERGE` |
| **10** | إضافة روابط — في `MarketingHeader` و `MarketingFooter` |
| **11** | إعادة تشغيل التطبيق |
| **12** | مسح cache المتصفح |

---

## 🔐 الجزء الرابع: أنماط صفحات المصادقة (Auth Pages)

### 4.1 نظرة عامة

**تستخدم صفحات المصادقة (Login, Register, ForgotPassword, ResetPassword) نظام تصميم موحد يعتمد على نمط Split-Panel (لوحة بصرية + لوحة نموذج).**

### 4.2 نمط Split-Panel

```
┌─────────────────────────────────────────────────────────┐
│  Visual Panel (60%)  │  Form Panel (40%)               │
│  ─────────────────   │  ──────────────                 │
│  • تدرج لوني جذاب    │  • نموذج الإدخال                │
│  • أيقونات عائمة     │  • شعار الشركة                  │
│  • خطوات العملية     │  • أزرار الإجراء                │
│  • موجات زخرفية      │  • روابط مساعدة                 │
└─────────────────────────────────────────────────────────┘
```

### 4.3 الألوان والتدرجات

**التدرج الأساسي (Visual Panel):**

```css
background: linear-gradient(135deg, #0F3B52 0%, #1B5A7A 50%, #2C7AA0 100%);
```

**الألوان الثانوية:**

| العنصر | اللون | الاستخدام |
|--------|-------|-----------|
| خلفية النموذج | `#FFFFFF` | Form Panel |
| خلفية الصفحة | `#F0F4F7` | Root Container |
| النصوص الأساسية | `#1A2B38` | العناوين |
| النصوص الثانوية | `#7A9BB0` | الوصف |
| الحدود | `#E1E8EE` | حقول الإدخال |
| الأزرار | `linear-gradient(135deg, #1B5A7A 0%, #2C7AA0 100%)` | Submit Button |

### 4.4 الفئات (Classes) المشتركة

#### `.forgot-root` — الحاوية الرئيسية

```css
.forgot-root {
    display: flex;
    min-height: 100vh;
    background-color: #F0F4F7;
    font-family: 'Cairo', sans-serif;
}
```

#### `.forgot-visual` — اللوحة البصرية

```css
.forgot-visual {
    flex: 1.2;
    position: relative;
    background: linear-gradient(135deg, #0F3B52 0%, #1B5A7A 50%, #2C7AA0 100%);
    overflow: hidden;
    display: flex;
    align-items: center;
    justify-content: center;
    padding: 40px;
}
```

#### `.forgot-form-panel` — لوحة النموذج

```css
.forgot-form-panel {
    flex: 0.8;
    background: white;
    display: flex;
    align-items: center;
    justify-content: center;
    padding: 40px;
}
```

#### `.forgot-input-wrap` — حاوية حقل الإدخال

```css
.forgot-input-wrap {
    position: relative;
    display: flex;
    align-items: center;
    border: 1px solid #E1E8EE;
    border-radius: 12px;
    transition: all 0.2s;
    background: white;
}

.forgot-input-wrap.focused {
    border-color: #1B5A7A;
    box-shadow: 0 0 0 3px rgba(27,90,122,0.1);
}
```

#### `.forgot-submit-btn` — زر الإرسال

```css
.forgot-submit-btn {
    width: 100%;
    background: linear-gradient(135deg, #1B5A7A 0%, #2C7AA0 100%);
    border: none;
    border-radius: 12px;
    padding: 14px;
    color: white;
    font-weight: 600;
    font-size: 16px;
    display: flex;
    align-items: center;
    justify-content: center;
    gap: 8px;
    cursor: pointer;
    transition: all 0.2s;
    margin-top: 8px;
}

.forgot-submit-btn:hover {
    transform: translateY(-2px);
    box-shadow: 0 4px 12px rgba(27,90,122,0.3);
}
```

### 4.5 التجاوبية

**Breakpoints:**

| النقطة | العرض | السلوك |
|--------|-------|--------|
| **Desktop** | > 992px | Split-Panel كامل |
| **Tablet/Mobile** | ≤ 992px | إخفاء Visual Panel، Form يملأ الشاشة |
| **Small Mobile** | ≤ 480px | تصغير إضافي |

**Media Queries:**

```css
@media (max-width: 992px) {
    .forgot-visual {
        display: none;
    }
    
    .forgot-form-panel {
        flex: 1;
        width: 100%;
        padding: 24px 16px;
        align-items: flex-start;
        padding-top: 40px;
    }
    
    .forgot-input {
        font-size: 16px; /* يمنع التكبير التلقائي في iOS */
    }
}

@media (max-width: 480px) {
    .forgot-form-panel {
        padding: 16px 12px;
        padding-top: 30px;
    }
    
    .forgot-title {
        font-size: 20px;
    }
}
```

### 4.6 ملفات CSS

| الملف | المحتوى |
|-------|---------|
| `_pages/Auth/_forget_password.css` | أنماط `.forgot-*` المشتركة |
| `_pages/Auth/Login.css` | أنماط `.login-*` |
| `_pages/Auth/Register.css` | أنماط `.register-*` |
| `_Auth_Public_Bundle.css` | استيراد جميع ملفات المصادقة |

### 4.7 أفضل الممارسات

1. **Touch Targets:** الحد الأدنى `48px`.
2. **iOS Zoom Prevention:** `font-size: 16px` في حقول الإدخال.
3. **RTL Support:** استخدام `[dir="rtl"]` و `[dir="ltr"]`.
4. **Focus States:** إضافة `.focused` class.

---

## 🎯 الجزء الخامس: أنماط الأنظمة الخاصة

### 5.1 نظام النقاط (Point Dashboard)

**المسار:** `_pages/Organization/PSP/Points/point-dashboard.css`

**الملفات المرتبطة:**

| الملف | الوصف |
|-------|-------|
| `point-dashboard.css` | لوحة النقاط الكاملة |
| `ppm-program-manage.css` | إدارة البرامج |
| `beneficiary-type-selector.css` | اختيار نوع المستفيد |

**الاستيراد (في `_pspbundle.css`):**

```css
/* point managment style pages */
@import url('_pages/Organization/PSP/Points/point-dashboard.css');
@import url('_pages/Organization/PSP/Points/ppm-program-manage.css');
@import url('_pages/Organization/PSP/Points/beneficiary-type-selector.css');
```

### 5.2 نظام التقارير (Reports)

**المسار:** `_pages/Reports/`

**الملفات:**

| الملف | الوصف |
|-------|-------|
| `provider-funnel.css` | تقرير المندوبين |
| `patient-funnel.css` | تقرير المرضى |
| `pharmacy-dispensation.css` | صرف الصيدليات |
| `pharmacy-financial.css` | التقرير المالي |
| `adherence.css` | الالتزام |
| `program-lifecycle.css` | دورة الحياة |

**⚠️ الاستيراد:** من `_PagesBundle.css`.

### 5.3 نظام القوائم الديناميكية

**المسار:** `_layout/`

**⚠️ لا يعتمد على `dir` على `.main-wrapper` — يعتمد على `html[dir]` فقط.**

**مثال من `_sidebar.css`:**

```css
/* ✅ الصحيح: يقرأ من html مباشرة */
html[dir="rtl"] .interactive-sidebar {
    right: 0 !important;
    left: auto !important;
}

html:not([dir="rtl"]) .interactive-sidebar {
    left: 0 !important;
    right: auto !important;
}

/* ❌ الخاطئ */
.main-wrapper[dir="rtl"] .interactive-sidebar { ... }
```

### 5.4 نظام PSP (PSP Styles)

**المسار:** `_pages/Organization/PSP/`

**بنية الملفات:**

```
_pages/Organization/PSP/
├── psp-detail.css
├── PSPProgramsList.css
├── PSPProgramDetails.css
├── PSPProgramsEdit.css
├── psp-search.css
├── PSPEditSteps/
│   ├── PSPStep0_BasicInfo.css
│   ├── PSPStep1_SpecsMeds.css
│   ├── PSPStep2_Budget.css
│   ├── PSPStep2_DispensationPlans.css
│   ├── PSPStep2b_PointScheme.css
│   ├── PSPStep_Links.css
│   ├── PSPStep3_Review.css
│   └── PSPRequiredTestsSection.css
├── program-details.css
├── invite-patient.css
├── join.css
└── Points/
    ├── point-dashboard.css
    ├── ppm-program-manage.css
    └── beneficiary-type-selector.css
```

**⚠️ ملاحظة:** `_pspbundle.css` مخصص لـ PSP — يُستورد من `main.css`.

---
# [#57] 📘 `10-UI-STYLE-GUIDE` v2.0 (Part 2)

---

## 📌 الجزء السادس: دليل التجاوبية (Responsive Design)

### 6.1 المبادئ الأساسية

| # | المبدأ | التطبيق |
|:-:|:---|:---|
| **1** | **Mobile-First** | صمم للموبايل أولاً، ثم وسّع |
| **2** | **Touch-Friendly** | الحد الأدنى `48px × 48px` |
| **3** | **Readable** | الحد الأدنى: `14px` (Body) — `16px` (Inputs) |
| **4** | **Fast** | تجنب الصور الكبيرة + Lazy Loading |

### 6.2 Breakpoints المعيارية

| الاسم | العرض | الاستخدام |
|-------|-------|-----------|
| **Mobile (Portrait)** | ≤ 480px | هواتف صغيرة |
| **Mobile (Landscape)** | 481px - 767px | هواتف كبيرة / تابلت صغير |
| **Tablet** | 768px - 991px | تابلت |
| **Desktop** | 992px - 1199px | شاشات متوسطة |
| **Large Desktop** | ≥ 1200px | شاشات كبيرة |

### 6.3 قواعد التجاوبية

#### 1. Touch Targets

```css
/* ❌ خطأ */
.btn {
    padding: 8px 16px;
    font-size: 12px;
}

/* ✅ صحيح */
.btn {
    min-height: 48px;
    min-width: 48px;
    padding: 12px 24px;
    font-size: 14px;
}
```

#### 2. iOS Zoom Prevention

```css
/* ❌ خطأ: iOS سيكبر الصفحة */
input { font-size: 14px; }

/* ✅ صحيح */
input { font-size: 16px; }
```

#### 3. Responsive Typography

```css
/* Desktop */
h1 { font-size: 32px; }
h2 { font-size: 24px; }
p { font-size: 16px; }

/* Tablet */
@media (max-width: 992px) {
    h1 { font-size: 28px; }
    h2 { font-size: 20px; }
    p { font-size: 15px; }
}

/* Mobile */
@media (max-width: 480px) {
    h1 { font-size: 24px; }
    h2 { font-size: 18px; }
    p { font-size: 14px; }
}
```

#### 4. Responsive Spacing

```css
/* Desktop */
.section {
    padding: 40px;
    margin-bottom: 32px;
}

/* Tablet */
@media (max-width: 992px) {
    .section {
        padding: 24px;
        margin-bottom: 20px;
    }
}

/* Mobile */
@media (max-width: 480px) {
    .section {
        padding: 16px;
        margin-bottom: 16px;
    }
}
```

### 6.4 اختبار التجاوبية

1. **Chrome DevTools:** `F12` → `Toggle Device Toolbar` (Ctrl+Shift+M).
2. **الأجهزة المختبرة:**
   - iPhone SE (375px)
   - iPhone 12 Pro (390px)
   - iPad (768px)
   - Desktop (1920px)
3. **النقاط الحرجة:**
   - [ ] هل الأزرار قابلة للنقر بسهولة؟
   - [ ] هل النصوص مقروءة بدون تكبير؟
   - [ ] هل الصور تظهر بشكل صحيح؟
   - [ ] هل النماذج تعمل على الموبايل؟

---

## 🗂️ الجزء السابع: نظام الباندلات المُتدرَّج

### 7.1 بنية المجلدات الفعلية

**المسار:** `Shared.UI/wwwroot/css/`

```
css/
├── _components/                    ← مكونات مشتركة
├── _core/                          ← المتغيرات الأساسية
├── _layout/                        ← التخطيطات
├── _pages/                         ← صفحات (مُقسَّمة حسب الدور)
│   ├── Admin/
│   ├── Auth/
│   ├── Clinic/
│   ├── LandingPages/
│   ├── Loyalty/
│   ├── Organization/
│   │   └── PSP/
│   │       ├── PSPEditSteps/
│   │       └── Points/
│   ├── Patient/
│   ├── Pharmacy/
│   ├── Public/
│   ├── PublicUser/
│   ├── Rep/
│   ├── Reports/
│   ├── Shared/
│   └── website/
├── _themes/                        ← الثيمات
├── _utilities/                     ← أدوات مساعدة
│
├── auth-pages.css                  ← صفحات المصادقة
├── main.css                        ← الملف الرئيسي (Imports)
├── RubikComponents.css             ← مكونات Rubik
├── style.css                       ← (فارغ)
├── webviewbundle.css               ← bundle لـ MAUI WebView
│
├── _AdminBundle.css                ← Bundle للـ Admin
├── _Auth_Public_Bundle.css         ← Bundle للـ Auth
├── _ComponentsBundle.css           ← Bundle للمكونات
├── _CoreBundle.css                 ← Bundle للـ Core
├── _LayoutBundle.css               ← Bundle للـ Layout
├── _PagesBundle.css                ← Bundle للصفحات
├── _pspbundle.css                  ← Bundle لـ PSP
├── _UtilitiesBundle.css            ← Bundle للـ Utilities
└── _website-bundle.css             ← Bundle للموقع التسويقي
```

### 7.2 الملف الرئيسي `main.css`

**⚠️ الترتيب مهم جداً — من الأعمق إلى الأعلى:**

```css
/* 
   main.css - الملف الرئيسي (imports فقط)
   الترتيب مهم جداً: من الأعمق إلى الأعلى
*/

@import url('_CoreBundle.css');          /* 1. الأساسيات */
@import url('_ComponentsBundle.css');    /* 2. المكونات */
@import url('_LayoutBundle.css');        /* 3. التخطيطات */
@import url('_PagesBundle.css');         /* 4. الصفحات */
@import url('_pspbundle.css');           /* 5. PSP */
@import url('_Auth_Public_Bundle.css');  /* 6. المصادقة */
@import url('_AdminBundle.css');         /* 7. Admin */
@import url('_UtilitiesBundle.css');     /* 8. الأدوات */
@import url('_website-bundle.css');      /* 9. الموقع التسويقي */
@import url('webviewbundle.css');        /* 10. MAUI WebView */
```

**⚠️ قاعدة ذهبية:** لا تكسر الترتيب — لأن CSS يعتمد على Cascade.

### 7.3 `_pspbundle.css` — مثال حي

**⚠️ هذا الملف مخصص لـ PSP — يُستورد من `main.css`:**

```css
/* PSP Edit Steps */
@import url('_pages/Organization/PSP/psp-detail.css');
@import url('_pages/Organization/PSP/PSPProgramsList.css');
@import url('_pages/Organization/PSP/PSPProgramDetails.css');
@import url('_pages/Organization/PSP/PSPProgramsEdit.css');
@import url('_pages/Organization/PSP/psp-search.css');

/* PSP Edit Steps - Steps */
@import url('_pages/Organization/PSP/PSPEditSteps/PSPStep0_BasicInfo.css');
@import url('_pages/Organization/PSP/PSPEditSteps/PSPStep1_SpecsMeds.css');
@import url('_pages/Organization/PSP/PSPEditSteps/PSPStep2_Budget.css');
@import url('_pages/Organization/PSP/PSPEditSteps/PSPStep2_DispensationPlans.css');
@import url('_pages/Organization/PSP/PSPEditSteps/PSPStep2b_PointScheme.css');
@import url('_pages/Organization/PSP/PSPEditSteps/PSPStep_Links.css');
@import url('_pages/Organization/PSP/PSPEditSteps/PSPStep3_Review.css');
@import url('_pages/Organization/PSP/PSPEditSteps/PSPRequiredTestsSection.css');

/* PSP Related */
@import url('_pages/Organization/PSP/program-details.css');
@import url('_pages/Organization/PSP/invite-patient.css');
@import url('_pages/Organization/PSP/join.css');

/* Point Management Style Pages */
@import url('_pages/Organization/PSP/Points/point-dashboard.css');
@import url('_pages/Organization/PSP/Points/ppm-program-manage.css');
@import url('_pages/Organization/PSP/Points/beneficiary-type-selector.css');

/* PSP About */
@import url('_pages/psp-about.css');
```

### 7.4 `_website-bundle.css` — مثال حي

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

### 7.5 نظام الباندلات المُتدرَّج — الفوائد

| # | الفائدة | التفصيل |
|:-:|:---|:---|
| **1** | **تنظيم واضح** | كل دور/نظام له bundle خاص |
| **2** | **سهولة الصيانة** | تعديل ملف واحد = تحديث النظام كله |
| **3** | **Tree-Shaking** | يمكن تحميل bundle واحد فقط |
| **4** | **Performance** | Browser يحمّل فقط ما يحتاجه |
| **5** | **Cascade صحيح** | الترتيب مضمون |
| **6** | **فصل الموقع عن التطبيق** | `_website-bundle` منفصل |

### 7.6 الباندلات الحالية (10 Bundles)

| # | الـ Bundle | المسؤولية |
|:-:|:---|:---|
| **1** | `_CoreBundle.css` | المتغيرات الأساسية |
| **2** | `_ComponentsBundle.css` | المكونات المشتركة |
| **3** | `_LayoutBundle.css` | التخطيطات |
| **4** | `_PagesBundle.css` | كل الصفحات |
| **5** | `_pspbundle.css` | PSP خاصة |
| **6** | `_Auth_Public_Bundle.css` | المصادقة |
| **7** | `_AdminBundle.css` | Admin |
| **8** | `_UtilitiesBundle.css` | أدوات |
| **9** | `_website-bundle.css` | الموقع التسويقي |
| **10** | `webviewbundle.css` | MAUI WebView |

---

## 🛠️ الجزء الثامن: كيفية إضافة نمط جديد

### 8.1 الخطوات

| # | الخطوة |
|:-:|:---|
| **1** | إنشاء ملف CSS جديد في المجلد المناسب: `_pages/Role/page-name.css` |
| **2** | تعريف المتغيرات باستخدام `--role-*` prefix |
| **3** | استخدام المتغيرات في جميع التنسيقات |
| **4** | إضافة `@import` في الـ Bundle المناسب |
| **5** | تطبيق الـ Class الرئيسي (`role-root`) في Razor Component |
| **6** | إضافة Media Queries للتجاوبية |
| **7** | اختبار على 3 أجهزة |

### 8.2 مثال: إضافة نمط لصفحة جديدة

**السكانيرو:** إضافة صفحة `ClinicReports`

**الخطوة 1:** إنشاء `_pages/Clinic/clinic-reports.css`:

```css
/* Clinic Reports - نمط الصفحة */
.clinic-reports-root {
    --cr-primary: var(--clinic-primary, #1A7A7A);
    --cr-primary-light: var(--clinic-primary-light, #D1ECF1);
    
    background: var(--rubik-body-bg);
    padding: 24px;
}

/* ... باقي الأنماط ... */

@media (max-width: 992px) {
    .clinic-reports-root {
        padding: 16px;
    }
}
```

**الخطوة 2:** إضافة `@import` في `_PagesBundle.css`:

```css
@import url('_pages/Clinic/clinic-reports.css');
```

**الخطوة 3:** تطبيق الـ Class في Razor:

```razor
<div class="clinic-reports-root">
    @* ... *@
</div>
```

### 8.3 Checklist عند إنشاء صفحة جديدة

#### التحقق من النمط

- [ ] هل الصفحة تتبع نمط الدور الرئيسي (Clinic/Pharmacy/Rep)؟
- [ ] هل تستخدم المتغيرات الصحيحة (`--clinic-*` / `--pharmacy-*` / `--rep-*`)؟
- [ ] هل الـ Hero Section يستخدم التدرج الصحيح؟
- [ ] هل أيقونات الأزرار تستخدم الألوان الصحيحة؟
- [ ] هل تم إضافة ملف CSS إلى الـ Bundle المناسب؟

#### التحقق من التجاوبية

- [ ] هل الصفحة تعمل على الموبايل (≤ 480px)؟
- [ ] هل الأزرار بحجم `48px` على الأقل؟
- [ ] هل حقول الإدخال `font-size: 16px`؟
- [ ] هل النصوص مقروءة بدون تكبير؟
- [ ] هل تم اختبار الصفحة على 3 أجهزة؟

#### التحقق من الترجمة

- [ ] هل أضفت مفاتيح الترجمة مع `N'' prefix`؟
- [ ] هل تستخدم `T()` المساعدة؟

#### التحقق من الاتجاه

- [ ] هل الصفحة **لا تحتوي** على `dir` ثابت؟
- [ ] هل الصفحة **ترث** `dir` من `<html>`؟
- [ ] هل CSS يعتمد على `html[dir]` فقط؟

---

## ⚠️ الجزء التاسع: تحذيرات ومحاذير

### 9.1 🔴 ممنوعات مطلقة

| # | الممنوع | البديل |
|---|---------|--------|
| **1** | ألوان Hardcoded (`#1B5A7A`) | `var(--rubik-primary)` |
| **2** | `left/right` في CSS | `inset-inline-start/end` |
| **3** | `@import` من داخل صفحة Razor | من الـ Bundle |
| **4** | ملف CSS بدون `@import` في Bundle | أضِفه |
| **5** | كسر ترتيب `main.css` | احترم Cascade |
| **6** | `.main-wrapper[dir]` في CSS | `html[dir]` |
| **7** | `dir` على عنصر Blazor SSR | `html[dir]` فقط |
| **8** | `font-size < 16px` في Inputs | يسبب Zoom في iOS |
| **9** | `Touch Target < 48px` | صعوبة في اللمس |
| **10** | وضع صفحات كاملة في Shared.UI | مكونات فقط |

### 9.2 🟡 تنبيهات مهمة

- **`main.css`** هو الملف الرئيسي — **لا تعدّل ملفات `_Bundle` بدون سبب**.
- **`_pspbundle.css`** مخصص — يمكن أن يكون مستقلاً.
- **`_website-bundle.css`** منفصل تماماً — لا يُخلط.
- **`webviewbundle.css`** خاص بـ MAUI WebView.
- **CSS Variables** — استخدمها دائماً.

### 9.3 ⚠️ الفرق بين Web و Mobile في الأنماط

| المنصة | المصدر | الملاحظة |
|--------|--------|----------|
| **Web** | `main.css` (كامل) | كل الـ Bundles |
| **PWA** | `main.css` (كامل) | نفس الشيء |
| **MAUI** | `webviewbundle.css` + بعض الـ Bundles | Bundle مخصص |

---

## 🔗 الجزء العاشر: روابط + سجل التغييرات

### 10.1 روابط ذات صلة

- [00 — MASTER-INDEX](./00-MASTER-INDEX.md)
- [00B — PROJECT-BACKLOG](./00B-PROJECT-BACKLOG.md)
- [01 — ARCHITECTURE](./01-ARCHITECTURE.md)
- [03 — MENUS](./03-MENUS.md)
- [04 — TRANSLATION](./04-TRANSLATION.md)
- [11 — PAGE-CREATION](./11-PAGE-CREATION.md)
- [13 — MAUI](./13-MAUI.md)
- [14 — PWA](./14-PWA.md)
- [18 — MARKETING-WEBSITE](./18-MARKETING-WEBSITE.md)

### 10.2 سجل التغييرات

| الإصدار | التاريخ | التغييرات |
|:---:|:---|:---|
| **1.0** | **18 يوليو 2026** | الإصدار الأولي — `03-style-guide.md` |
| **1.0** | **23 سبتمبر 2026** | الإصدار الأولي — `25-marketing-website.md` (جزء CSS) |
| **2.0** | **16 أكتوبر 2026** | **`10-UI-STYLE-GUIDE` v2.0** — دمج `03` + `25` (CSS) + نظام الباندلات المُتدرَّج من الفحص الفعلي |

---

**© 2026 RubikCare — للاستخدام الداخلي**

**نهاية الوثيقة ✅**
```

---
