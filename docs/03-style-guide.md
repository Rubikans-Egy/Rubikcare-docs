# 03 - دليل التصميم والنمط البصري (Style Guide)

**آخر تحديث: 17 مايو 2026**

---

## مقدمة

هذا المرجع هو **الهوية البصرية** لروبيك كير. أي لون، خط، مسافة، أو ظل تراه في المنصة مصدره الوحيد هو المتغيرات الموثقة هنا. **لا تستخدم قيماً ثابتة أبداً** (مثل `#1B5A7A` مباشرة في CSS). كل شيء يجب أن يمر عبر المتغيرات.

---

## نظام الألوان - المصدر الوحيد (`_core/_variables.css`)

### الألوان الأساسية

```css
:root {
    /* ====== الألوان الأساسية ====== */
    --rubik-primary:       #1B5A7A;   /* الأزرق الداكن - اللون الرئيسي */
    --rubik-primary-light: #2C7AA0;   /* أزرق فاتح - hover وتفاصيل */
    --rubik-primary-dark:  #0F3B52;   /* أزرق غامق - تدرجات وخلفيات */
    --rubik-secondary:     #5F9EA0;   /* أزرق رمادي - عناصر ثانوية */
    --rubik-accent:        #6D8A8D;   /* رمادي مزرق - تفاصيل دقيقة */

    /* ====== ألوان الحالات ====== */
    --rubik-success:       #2E7D5E;   /* أخضر - نجاح */
    --rubik-success-light: #E8F3E9;   /* خلفية النجاح الفاتحة */
    --rubik-danger:        #B33F40;   /* أحمر - خطأ */
    --rubik-danger-light:  #FBEAEA;   /* خلفية الخطأ الفاتحة */
    --rubik-warning:       #B88B4A;   /* برتقالي - تحذير */
    --rubik-warning-light: #FFF3E0;   /* خلفية التحذير الفاتحة */
    --rubik-info:          #4A7B8C;   /* أزرق معلومات */
    --rubik-info-light:    #E6F3F7;   /* خلفية المعلومات الفاتحة */

    /* ====== الخلفيات ====== */
    --rubik-body-bg:  #F5F7FA;   /* خلفية الصفحة الرئيسية */
    --rubik-card-bg:  #FFFFFF;   /* خلفية الكروت */
    --rubik-card-alt: #F9FAFC;   /* خلفية بديلة للكروت */

    /* ====== الحدود ====== */
    --rubik-border-light: #EFF1F5;
    --rubik-border:       #E2E6EA;
    --rubik-border-dark:  #C5C9D0;

    /* ====== النصوص ====== */
    --rubik-text-primary:   #1A2B3C;  /* النص الرئيسي - على الخلفيات الفاتحة */
    --rubik-text-secondary: #546E7A;  /* النص الثانوي */
    --rubik-text-muted:     #8A9BA8;  /* النص الخافت */
    --rubik-text-light:     #FFFFFF;  /* النص الأبيض - على الخلفيات الداكنة */

    /* ====== الظلال ====== */
    --shadow-sm:   0 2px 8px  rgba(26,43,60,0.05);
    --shadow-md:   0 4px 12px rgba(26,43,60,0.08);
    --shadow-lg:   0 8px 24px rgba(26,43,60,0.12);
    --shadow-card: 0 2px 12px rgba(27,90,122,0.08);
}
```

### دعم الاتجاه (RTL/LTR)

```css
:root[dir="rtl"] {
    --rubik-direction:  rtl;
    --rubik-text-align: right;
    --rubik-float:      right;
}

:root[dir="ltr"] {
    --rubik-direction:  ltr;
    --rubik-text-align: left;
    --rubik-float:      left;
}
```

---

## المقاييس والمسافات

```css
:root {
    /* ====== المسافات ====== */
    --space-xs:  4px;
    --space-sm:  8px;
    --space-md:  16px;
    --space-lg:  24px;
    --space-xl:  32px;
    --space-xxl: 48px;

    /* ====== الزوايا ====== */
    --radius-sm:   4px;
    --radius-md:   8px;   /* الافتراضي */
    --radius-lg:   12px;
    --radius-xl:   16px;
    --radius-full: 9999px;

    /* ====== الأبعاد الثابتة ====== */
    --header-height:        60px;
    --sidebar-width:        250px;
    --container-max-width:  1400px;
}
```

---

## الخطوط والنصوص

### تعريف الخطوط

```css
/* العربية RTL */
:root[dir="rtl"] {
    --rubik-font-family:    'Cairo', sans-serif;
    --rubik-line-height:    1.6;
    --rubik-letter-spacing: normal;
}

/* الإنجليزية LTR */
:root[dir="ltr"] {
    --rubik-font-family:    'Poppins', 'Cairo', sans-serif;
    --rubik-line-height:    1.5;
    --rubik-letter-spacing: -0.011em;
}

/* تطبيق الخط على الجسم */
body {
    font-family: var(--rubik-font-family);
    line-height: var(--rubik-line-height);
    letter-spacing: var(--rubik-letter-spacing);
    color: var(--rubik-text-primary);
    background-color: var(--rubik-body-bg);
}
```

### أحجام الخطوط

```css
/* عناوين رئيسية */
h1 { font-size: 2.5rem; font-weight: 700; }
h2 { font-size: 2rem;   font-weight: 600; }
h3 { font-size: 1.75rem; font-weight: 600; }
h4 { font-size: 1.5rem;  font-weight: 500; }
h5 { font-size: 1.25rem; font-weight: 500; }
h6 { font-size: 1rem;    font-weight: 500; }

/* نصوص عادية */
p, span, div, label, input, textarea {
    font-size: var(--rubik-font-size-base, 0.9375rem);
}

/* نصوص مساعدة */
small, .text-muted {
    font-size: 0.875rem;
    color: var(--rubik-text-muted);
}
```

---

## نظام `.rubik-dark` - حل مشكلة النصوص السوداء (الابتكار الرئيسي)

### المشكلة التي حُلّت

```css
/* ❌ خاطئ - هذا كان موجوداً ويسبب مشاكل */
body, p, span, div:not([class*="btn"]) {
    color: var(--rubik-text-primary);  /* يجبر اللون الداكن على كل شيء */
}
/* النتيجة: حتى على الخلفيات الداكنة، النصوص تصبح داكنة وغير مقروءة */
```

### الحل المعتمد

```css
/* ✅ صحيح - body فقط، والباقي يرث */
body {
    color: var(--rubik-text-primary);
    background-color: var(--rubik-body-bg);
}

/* ⭐⭐ الحل السحري: .rubik-dark */
.rubik-dark,
.rubik-dark * {
    color: var(--rubik-text-light) !important;  /* يجبر كل النصوص الداخلية على اللون الأبيض */
}

/* استثناءات مهمة */
.rubik-dark .btn { color: unset; }  /* الأزرار تحافظ على ألوانها */
.rubik-dark .text-warning { color: var(--rubik-warning) !important; }
.rubik-dark .text-success { color: var(--rubik-success) !important; }
.rubik-dark .text-danger  { color: var(--rubik-danger) !important; }
.rubik-dark .text-muted   { color: rgba(255,255,255,.65) !important; }
```

### كيفية الاستخدام

```html
<!-- ✅ Hero sections -->
<section class="dashboard-hero rubik-dark">
    <h1>مرحباً بك في روبيك كير</h1>
    <p>منصة صحية متكاملة</p>
</section>

<!-- ✅ Card headers الداكنة -->
<div class="card">
    <div class="card-header rubik-dark">
        <h6><i class="fas fa-chart-bar me-2"></i>إحصائيات طبية</h6>
    </div>
    <div class="card-body">
        <!-- محتوى فاتح عادي -->
    </div>
</div>

<!-- ✅ أي عنصر بخلفية داكنة -->
<div class="bg-primary rubik-dark">
    هذا النص سيظهر أبيض تلقائياً
</div>
```

---

## هيكل ملفات CSS

### الهيكل الكامل

```
wwwroot/css/
├── main.css                    ← الملف الرئيسي (imports فقط)
│
├── _core/                      ← الأساسيات (تُستورد أولاً)
│   ├── _variables.css          ← ⭐ متغيرات التصميم الموحدة
│   ├── _reset.css              ← إعادة ضبط المتصفحات
│   ├── _fonts.css              ← تعريفات الخطوط
│   ├── _base.css               ← الأنماط الأساسية
│   └── _typography.css         ← ⭐ أنماط النصوص (معدلة)
│
├── _components/                 ← المكونات المستقلة
│   ├── _cards.css              ← ⭐ البطاقات
│   ├── _tables.css             ← الجداول
│   ├── _buttons.css            ← الأزرار
│   ├── _forms.css              ← النماذج
│   ├── _wizard.css             ← معالج الخطوات
│   ├── _GenericModal.css       ← النوافذ المنبثقة
│   ├── _RubikDropdown.css      ← ⭐ القوائم المنسدلة المخصصة
│   └── _slider.css             ← السلايدر
│
├── _layout/                     ← التخطيطات
│   ├── _containers.css         ← الحاويات
│   ├── _grid.css               ← الشبكة
│   ├── _header.css             ← الهيدر
│   ├── _sidebar.css            ← ⭐ القائمة الجانبية
│   ├── _dashboard.css          ← لوحة التحكم العامة
│   └── _footer.css             ← الفوتر
│
├── _pages/                      ← ⭐ صفحات لها تنسيق خاص
│   └── _centerhub.css          ← الصفحة الرئيسية
│
├── _themes/                     ← السمات
│   └── _rubik.css              ← السمة الرئيسية
│
├── _utilities/                  ← الأدوات المساعدة
│   ├── _spacing.css            ← المسافات
│   ├── _colors.css             ← ألوان مساعدة
│   └── observability.css       ← أدوات المطور
│
└── auth-pages.css                ← صفحات تسجيل الدخول
```

### ترتيب الاستيراد في `main.css` (حرج)

```css
/* ====== main.css ====== */

/* 1. الأساسيات - من الأعمق للأعم */
@import url('_core/_variables.css');
@import url('_core/_reset.css');
@import url('_core/_fonts.css');
@import url('_core/_base.css');
@import url('_core/_typography.css');

/* 2. المكونات المستقلة */
@import url('_components/_cards.css');
@import url('_components/_tables.css');
@import url('_components/_buttons.css');
@import url('_components/_forms.css');
@import url('_components/_wizard.css');
@import url('_components/_GenericModal.css');
@import url('_components/_RubikDropdown.css');
@import url('_components/_slider.css');

/* 3. التخطيط */
@import url('_layout/_containers.css');
@import url('_layout/_grid.css');
@import url('_layout/_header.css');
@import url('_layout/_sidebar.css');
@import url('_layout/_dashboard.css');
@import url('_layout/_footer.css');

/* 4. صفحات تسجيل الدخول */
@import url('auth-pages.css');

/* 5. السمات */
@import url('_themes/_rubik.css');

/* 6. الأدوات */
@import url('_utilities/_spacing.css');
@import url('_utilities/_colors.css');
@import url('_utilities/observability.css');

/* 7. صفحات خاصة */
@import url('_pages/_centerhub.css');
```

### قاعدة إضافة ملف CSS جديد

| الموقف | أين تضعه |
|--------|----------|
| **صفحة جديدة بتصميم فريد** (> 100 سطر) | `_pages/_pagename.css` |
| **مكون يُستخدم في أكثر من صفحة** | `_components/_componentname.css` |
| **تعديلات بسيطة** (< 50 سطر) | أضفها لملف موجود |
| **تنسيقات Bootstrap أصلية** | لا تلمسها، استخدم المتغيرات |

---

## البطاقات (Cards)

### البطاقات الأساسية Bootstrap (المعدلة)

```css
/* _components/_cards.css */

/* توريث الألوان - حل مشكلة النصوص القادمة من قاعدة البيانات */
.card *,
.card-header *,
.card-body *,
.card-footer * {
    color: inherit;
}

/* تخصيص رأس البطاقة */
.card-header {
    padding: 1rem 1.5rem;
    background-color: var(--rubik-card-alt);
    border-bottom: 1px solid var(--rubik-border);
    font-weight: 600;
}

.card-header.rubik-dark {
    background-color: var(--rubik-primary);
    border-bottom: none;
}
```

### بطاقات Rubik المخصصة (`.rubik-card`)

```html
<!-- بطاقة مع حد علوي ملون -->
<div class="rubik-card primary">
    <div class="rubik-card-header">
        <div class="rubik-card-title">
            <div class="rubik-card-icon primary">
                <i class="bi bi-capsule-pill"></i>
            </div>
            <div>
                <h5>عنوان البطاقة</h5>
                <div class="rubik-card-subtitle">وصف فرعي</div>
            </div>
        </div>
    </div>
    <div class="rubik-card-body">
        محتوى البطاقة
    </div>
    <div class="rubik-card-footer">
        <RubikButton Text="إجراء" ColorClass="btn-primary" />
    </div>
</div>
```

```css
/* _components/_cards.css - Rubik Cards */
.rubik-card {
    background: var(--rubik-card-bg);
    border-radius: var(--radius-md);
    box-shadow: var(--shadow-card);
    overflow: hidden;
    transition: all 0.3s ease;
}

.rubik-card:hover {
    box-shadow: var(--shadow-md);
    transform: translateY(-2px);
}

/* الحدود العلوية الملونة */
.rubik-card.primary { border-top: 4px solid var(--rubik-primary); }
.rubik-card.success { border-top: 4px solid var(--rubik-success); }
.rubik-card.warning { border-top: 4px solid var(--rubik-warning); }
.rubik-card.danger  { border-top: 4px solid var(--rubik-danger); }
.rubik-card.info    { border-top: 4px solid var(--rubik-info); }

/* الأيقونات الملونة */
.rubik-card-icon {
    width: 48px;
    height: 48px;
    border-radius: var(--radius-full);
    display: flex;
    align-items: center;
    justify-content: center;
    font-size: 1.5rem;
}

.rubik-card-icon.primary { background-color: var(--rubik-primary-light); color: white; }
.rubik-card-icon.success { background-color: var(--rubik-success); color: white; }
.rubik-card-icon.warning { background-color: var(--rubik-warning); color: white; }
```

### بطاقات الإحصائيات (`.stat-card-modern`)

```html
<div class="stat-card-modern">
    <div class="stat-icon">💊</div>
    <div class="stat-value">128</div>
    <div class="stat-label">دواء مسجل</div>
</div>
```

```css
.stat-card-modern {
    background: var(--rubik-card-bg);
    border-radius: var(--radius-md);
    padding: var(--space-lg);
    box-shadow: var(--shadow-sm);
    text-align: center;
    transition: all 0.3s ease;
}

.stat-card-modern:hover {
    box-shadow: var(--shadow-md);
    transform: translateY(-2px);
}

.stat-icon {
    font-size: 2rem;
    margin-bottom: var(--space-sm);
}

.stat-value {
    font-size: 2rem;
    font-weight: 700;
    color: var(--rubik-primary);
    line-height: 1.2;
}

.stat-label {
    color: var(--rubik-text-secondary);
    font-size: 0.9rem;
}
```

---

## القائمة الجانبية (`_sidebar.css`)

```css
/* _layout/_sidebar.css */

/* القاعدة العامة - RTL افتراضي */
.interactive-sidebar {
    position: fixed;
    top: 0;
    right: 0;  /* RTL */
    left: auto;
    width: var(--sidebar-width);
    height: 100vh;
    background-color: var(--rubik-card-bg);
    border-left: 1px solid var(--rubik-border);  /* RTL */
    border-right: none;
    box-shadow: var(--shadow-md);
    overflow-y: auto;
    transition: all 0.3s ease;
    z-index: 1000;
}

/* دعم LTR */
[dir="ltr"] .interactive-sidebar {
    right: auto;
    left: 0;
    border-left: none;
    border-right: 1px solid var(--rubik-border);
}

/* تبويبات القائمة */
.menu-tab {
    padding: 12px 16px;
    cursor: pointer;
    border-radius: var(--radius-sm);
    margin-bottom: 4px;
    transition: all 0.3s ease;
    display: flex;
    align-items: center;
    gap: 10px;
    color: var(--rubik-text-primary);
}

.menu-tab:hover {
    background-color: var(--rubik-border-light);
    color: var(--rubik-primary);
}

.menu-tab.active {
    background-color: var(--rubik-primary);
    color: white;
    font-weight: 600;
}

/* عناصر القائمة */
.nav-link {
    padding: 10px 16px;
    color: var(--rubik-text-secondary);
    border-radius: var(--radius-sm);
    display: flex;
    align-items: center;
    gap: 10px;
    transition: all 0.2s;
}

.nav-link:hover {
    background-color: var(--rubik-border-light);
    color: var(--rubik-primary);
}

.nav-link.active {
    background-color: var(--rubik-primary-light);
    color: white;
}

.nav-link i {
    width: 20px;
    text-align: center;
}

/* زر الرجوع للقائمة الشخصية */
.btn-back-to-personal {
    padding: 10px 16px;
    background-color: var(--rubik-card-alt);
    border: 1px solid var(--rubik-border);
    border-radius: var(--radius-sm);
    cursor: pointer;
    display: flex;
    align-items: center;
    justify-content: center;
    gap: 8px;
    transition: all 0.3s;
    color: var(--rubik-text-secondary);
    margin-top: var(--space-md);
}

.btn-back-to-personal:hover {
    background-color: var(--rubik-border-light);
    border-color: var(--rubik-border-dark);
    color: var(--rubik-text-primary);
}
```

---

## معايير الجودة والإلزاميات

### قواعد إلزامية

```css
/* 1. استخدم المتغيرات دائماً */
color: var(--rubik-primary);          /* ✅ صحيح */
color: #1B5A7A;                        /* ❌ خطأ - ممنوع */

/* 2. استخدم خصائص منطقية للاتجاه */
padding-inline-start: var(--space-md); /* ✅ RTL/LTR */
padding-right: var(--space-md);        /* ❌ RTL فقط */

margin-inline-end: auto;               /* ✅ */
margin-left: auto;                     /* ❌ */

/* 3. لا تستخدم !important إلا في حالات استثنائية موثقة */
color: white !important; /* ✅ مقبول في .rubik-dark */
padding: 0 !important;   /* ❌ غير مقبول - مشكلة هيكلية */

/* 4. لا تضع CSS داخل ملفات Razor */
<style> ... </style>    /* ❌ ممنوع - استخدم ملفات CSS فقط */
```

### بنية CSS المقبولة

```css
/* ====================================================
   اسم القسم - وصف واضح
   ==================================================== */

/* --- القسم الفرعي --- */
.component-name {
    /* Layout */
    display: flex;
    align-items: center;

    /* Box Model */
    padding: var(--space-md) var(--space-lg);
    border-radius: var(--radius-md);

    /* Visual */
    background: var(--rubik-card-bg);
    color: var(--rubik-text-primary);
    box-shadow: var(--shadow-sm);

    /* Animation */
    transition: all .25s ease;
}

.component-name:hover {
    box-shadow: var(--shadow-md);
    transform: translateY(-2px);
}
```

### إمكانية الوصول

```css
/* دعم Screen Readers */
.sr-only {
    position: absolute;
    width: 1px; height: 1px;
    overflow: hidden;
    clip: rect(0,0,0,0);
}

/* تباين كافٍ - نسبة 4.5:1 على الأقل */
/* rubik-primary #1B5A7A على أبيض = 7.2:1 ✅ */
/* rubik-text-primary #1A2B3C على أبيض = 13.8:1 ✅ */
```

---

## تشخيص مشاكل CSS - دليل سريع

### عند تعارض الألوان

1. افتح DevTools (F12) ← انقر على العنصر
2. تبويب Computed ← ابحث عن `color`
3. شاهد من أين تأتي القاعدة المطبقة
4. إذا كانت من `_typography.css` القديمة، تأكد من حذف قاعدة `span/div`
5. إذا كانت الخلفية داكنة، أضف `.rubik-dark` على العنصر

### عند مشاكل الاتجاه (RTL/LTR)

```javascript
// في Console
console.log('Current dir:', document.documentElement.dir);
console.log('Body dir:', document.body.dir);
console.log('Stored language:', localStorage.getItem('RubikCare:Language'));
```

### عند اختفاء الأنماط

1. تأكد من استيراد الملف في `main.css`
2. تأكد من ترتيب الاستيراد (الأساسيات أولاً)
3. افحص Network Tab في DevTools هل الملف محمل؟

---

## CHECKLIST: عند إضافة تنسيقات جديدة

- [ ] هل استخدمت المتغيرات بدلاً من القيم الثابتة؟
- [ ] هل أضفت `dir` selectors لدعم RTL/LTR؟
- [ ] هل اختبرت في كلا الاتجاهين (عربي وإنجليزي)؟
- [ ] إذا أضفت خلفية داكنة، هل أضفت `.rubik-dark`؟
- [ ] هل تجنبت استخدام `!important` إلا للضرورة القصوى؟
- [ ] هل وضعت الملف في المكان الصحيح (`_components/` أم `_pages/` أم `_layout/` )؟
- [ ] هل أضفت الملف الجديد في `main.css` بالترتيب الصحيح؟

---

## 🔗 روابط ذات صلة

- [00 - الهيكل المعماري](00-architecture-overview.md)
- [04 - نظام القوائم الديناميكية](04-dynamic-menus.md)
- [05 - إنشاء الصفحات والمكونات](05-page-creation-checklist.md)
```

