# 03 - دليل الأنماط (Style Guide)

**آخر تحديث:** 18 يوليو 2026

## 📌 مقدمة

هذا الدليل يوثق نظام الألوان والتصميم في منصة RubikCare، مع التركيز على الأنماط المخصصة لكل دور (عيادة، صيدلية، مندوب)، **وأنماط صفحات المصادقة، ودليل التجاوبية للموبايل**. الهدف هو ضمان تناسق بصري مع احترام هوية كل دور، وتوفير تجربة مستخدم ممتازة على جميع الأجهزة.

---

## 🎨 نظام الألوان العام (المتغيرات العالمية)

جميع المتغيرات العالمية معرفة في `_CoreBundle.css` وتستخدم كقاعدة للمتغيرات الخاصة بكل دور.

### المتغيرات الأساسية

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

### 🆕 المتغيرات الدلالية المتقدمة (⭐ جديد - 18 يوليو 2026)

| المتغير | القيمة | الاستخدام |
|---------|--------|-----------|
| `--rubik-danger` | `#B33F40` | رسائل الخطأ، الأزرار الخطرة |
| `--rubik-danger-light` | `#FBEAEA` | خلفيات رسائل الخطأ |
| `--rubik-success` | `#2E7D5E` | رسائل النجاح، التأكيدات |
| `--rubik-success-light` | `#E8F3E9` | خلفيات رسائل النجاح |
| `--rubik-warning` | `#B88B4A` | رسائل التحذير |
| `--rubik-warning-light` | `#FFF3E0` | خلفيات رسائل التحذير |
| `--rubik-info` | `#4A7B8C` | رسائل المعلومات |
| `--rubik-info-light` | `#E6F3F7` | خلفيات رسائل المعلومات |
| `--rubik-card-bg` | `#FFFFFF` | خلفية البطاقات |
| `--rubik-card-alt` | `#F9FAFC` | خلفية البطاقات البديلة |
| `--rubik-body-bg` | `#F5F7FA` | خلفية الصفحة الرئيسية |
| `--rubik-border-light` | `#EFF1F5` | حدود فاتحة |
| `--rubik-border-dark` | `#C5C9D0` | حدود داكنة |
| `--rubik-text-primary` | `#1A2B3C` | النصوص الأساسية |
| `--rubik-text-secondary` | `#546E7A` | النصوص الثانوية |
| `--rubik-text-muted` | `#8A9BA8` | النصوص الخافتة |

---

## 🏥 نمط العيادة (Clinic) - أزرق بحري + أزرق-أخضر

### الألوان الأساسية

| المتغير | القيمة | الاستخدام |
|---------|--------|-----------|
| `--clinic-hero-start` | `#0A3D5C` | بداية تدرج الـ Hero |
| `--clinic-hero-end` | `#1A7A7A` | نهاية تدرج الـ Hero |
| `--clinic-primary` | `#1A7A7A` | اللون الأساسي للعيادة |
| `--clinic-primary-light` | `#D1ECF1` | خلفيات فاتحة للعيادة |
| `--clinic-primary-dark` | `#0A3D5C` | تدرج غامق للعيادة |
| `--clinic-secondary` | `#2D8B6E` | اللون الثانوي للعيادة |
| `--clinic-secondary-light` | `#D4EDDA` | خلفيات ثانوية فاتحة |
| `--clinic-gold` | `#B88B4A` | لون مميز (تنبيهات) |
| `--clinic-gold-light` | `#F7F0E4` | خلفيات ذهبية فاتحة |

### تدرج الـ Hero

```css
background: linear-gradient(135deg, #0A3D5C 0%, #1A7A7A 100%);
```

### أيقونات العيادة

| الأيقونة | اللون | الاستخدام |
|---------|-------|-----------|
| `clinic-action-icon-clinic` | `#D1ECF1` | أيقونة "عيادتي" |
| `clinic-action-icon-psp` | `#D4EDDA` | أيقونة "برامج الدعم" |
| `clinic-action-icon-patients` | `#D1ECF1` | أيقونة "مرضاي" |
| `clinic-action-icon-invite` | `#F7F0E4` | أيقونة "دعواتي" |

### تطبيق النمط

في Razor Component:

```razor
<div class="clinic-root @LangClass">
    <div class="clinic-hero">...</div>
    <div class="clinic-actions-section">...</div>
    <div class="clinic-section">...</div>
</div>
```

ملف CSS: `_pages/Clinic/clinic-dashboard.css`

---

## 💊 نمط الصيدلية (Pharmacy) - أخضر غامق + زمردي

### الألوان الأساسية

| المتغير | القيمة | الاستخدام |
|---------|--------|-----------|
| `--pharmacy-hero-start` | `#1A3A3A` | بداية تدرج الـ Hero |
| `--pharmacy-hero-end` | `#2D8B6E` | نهاية تدرج الـ Hero |
| `--pharmacy-primary` | `#2D8B6E` | اللون الأساسي للصيدلية |
| `--pharmacy-primary-light` | `#D4EDDA` | خلفيات فاتحة للصيدلية |
| `--pharmacy-primary-dark` | `#1A3A3A` | تدرج غامق للصيدلية |
| `--pharmacy-secondary` | `#1A7A7A` | اللون الثانوي للصيدلية |
| `--pharmacy-secondary-light` | `#D1ECF1` | خلفيات ثانوية فاتحة |
| `--pharmacy-gold` | `#B88B4A` | لون مميز (تنبيهات) |
| `--pharmacy-gold-light` | `#F7F0E4` | خلفيات ذهبية فاتحة |

### تدرج الـ Hero

```css
background: linear-gradient(135deg, #1A3A3A 0%, #2D8B6E 100%);
```

### أيقونات الصيدلية

| الأيقونة | اللون | الاستخدام |
|---------|-------|-----------|
| `pharmacy-action-icon-pharmacy` | `#D4EDDA` | أيقونة "صيدليتي" |
| `pharmacy-action-icon-psp` | `#D4EDDA` | أيقونة "برامج الدعم" |
| `pharmacy-action-icon-products` | `#F7F0E4` | أيقونة "المنتجات" |
| `pharmacy-action-icon-orders` | `#D1ECF1` | أيقونة "طلبات المرضى" |

### تطبيق النمط

في Razor Component:

```razor
<div class="pharmacy-root @LangClass">
    <div class="pharmacy-hero">...</div>
    <div class="pharmacy-actions-section">...</div>
    <div class="pharmacy-section">...</div>
</div>
```

ملف CSS: `_pages/Pharmacy/pharmacy-dashboard.css`

---

## 🏢 نمط المندوب (Rep) - ذهبي

### الألوان الأساسية

| المتغير | القيمة | الاستخدام |
|---------|--------|-----------|
| `--rep-hero-start` | `#3D2E1A` | بداية تدرج الـ Hero |
| `--rep-hero-end` | `#B88B4A` | نهاية تدرج الـ Hero |
| `--rep-primary` | `#B88B4A` | اللون الأساسي للمندوب |
| `--rep-primary-light` | `#F7F0E4` | خلفيات فاتحة للمندوب |
| `--rep-primary-dark` | `#3D2E1A` | تدرج غامق للمندوب |
| `--rep-secondary` | `#1A7A7A` | اللون الثانوي للمندوب |
| `--rep-secondary-light` | `#D1ECF1` | خلفيات ثانوية فاتحة |

### تدرج الـ Hero

```css
background: linear-gradient(135deg, #3D2E1A 0%, #B88B4A 100%);
```

### تطبيق النمط

في Razor Component:

```razor
<div class="rep-root @LangClass">
    <div class="rep-hero">...</div>
    <div class="rep-actions-section">...</div>
    <div class="rep-section">...</div>
</div>
```

ملف CSS: `_pages/Rep/rep-dashboard.css`

---

## 📋 الصفحات الداخلية (Sub-pages)

### مبدأ التطبيق

يجب أن تتبع الصفحات الداخلية نفس نظام ألوان الدور الرئيسي.

**مثال:** إذا كان المستخدم طبيباً (دور Clinic)، فإن جميع الصفحات الداخلية (مثل `PspGateway`, `MyPatientInvitations`) يجب أن تستخدم ألوان العيادة (الأزرق البحري + الأزرق-الأخضر).

### كيفية التطبيق

استخدم نفس `--clinic-*` أو `--pharmacy-*` أو `--rep-*` المتغيرات في ملفات CSS الخاصة بالصفحات الداخلية.

أو استخدم المتغيرات العالمية (`--rubik-*`) مع الحفاظ على تناسق الألوان.

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
    /* ... باقي المتغيرات ... */
}
```

بهذه الطريقة، الصفحات الداخلية ترث ألوان الدور الرئيسي تلقائياً.

---

## 🔐 أنماط صفحات المصادقة (Auth Pages) (⭐ جديد - 18 يوليو 2026)

### نظرة عامة

تستخدم صفحات المصادقة (Login, Register, ForgotPassword, ResetPassword) نظام تصميم موحد يعتمد على نمط **Split-Panel** (لوحة بصرية + لوحة نموذج).

### 🏗️ نمط Split-Panel

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

### 🎨 الألوان والتدرجات

#### التدرج الأساسي (Visual Panel)

```css
background: linear-gradient(135deg, #0F3B52 0%, #1B5A7A 50%, #2C7AA0 100%);
```

#### الألوان الثانوية

| العنصر | اللون | الاستخدام |
|--------|-------|-----------|
| خلفية النموذج | `#FFFFFF` | Form Panel |
| خلفية الصفحة | `#F0F4F7` | Root Container |
| النصوص الأساسية | `#1A2B38` | العناوين |
| النصوص الثانوية | `#7A9BB0` | الوصف |
| الحدود | `#E1E8EE` | حقول الإدخال |
| الأزرار | `linear-gradient(135deg, #1B5A7A 0%, #2C7AA0 100%)` | Submit Button |

### 📐 الفئات (Classes) المشتركة

#### `.forgot-root` - الحاوية الرئيسية

```css
.forgot-root {
    display: flex;
    min-height: 100vh;
    background-color: #F0F4F7;
    font-family: 'Cairo', sans-serif;
}
```

#### `.forgot-visual` - اللوحة البصرية

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

#### `.forgot-form-panel` - لوحة النموذج

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

#### `.forgot-input-wrap` - حاوية حقل الإدخال

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

#### `.forgot-submit-btn` - زر الإرسال

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

### 📱 التجاوبية (Responsive Design)

#### Breakpoints

| النقطة | العرض | السلوك |
|--------|-------|--------|
| **Desktop** | > 992px | Split-Panel كامل (Visual + Form) |
| **Tablet/Mobile** | ≤ 992px | إخفاء Visual Panel، Form يملأ الشاشة |
| **Small Mobile** | ≤ 480px | تصغير إضافي للخطوط والمسافات |

#### Media Queries

```css
/* على الشاشات المتوسطة والصغيرة (تابلت وهاتف) */
@media (max-width: 992px) {
    .forgot-visual {
        display: none; /* إخفاء اللوحة البصرية */
    }
    
    .forgot-form-panel {
        flex: 1;
        width: 100%;
        padding: 24px 16px;
        align-items: flex-start;
        padding-top: 40px;
    }
    
    .forgot-form-container {
        max-width: 100%;
        padding: 0 8px;
    }
    
    /* تصغير العناوين */
    .forgot-title {
        font-size: 22px;
        line-height: 1.3;
    }
    
    /* تحسين الحقول للمس (Touch Targets) */
    .forgot-input-wrap {
        min-height: 48px; /* حجم لمس مريح */
        border-radius: 10px;
    }
    
    .forgot-input {
        padding: 12px 44px 12px 14px;
        font-size: 16px; /* ⭐ يمنع التكبير التلقائي في iOS */
    }
    
    .forgot-submit-btn {
        min-height: 48px;
        font-size: 15px;
    }
}

/* على الشاشات الصغيرة جداً (هاتف عمودي) */
@media (max-width: 480px) {
    .forgot-form-panel {
        padding: 16px 12px;
        padding-top: 30px;
    }
    
    .forgot-title {
        font-size: 20px;
    }
    
    .forgot-input {
        font-size: 16px; /* منع التكبير التلقائي في iOS */
    }
}
```

### 💡 أفضل الممارسات

1. **Touch Targets:** الحد الأدنى `48px` (معيار Google Material Design)
2. **iOS Zoom Prevention:** `font-size: 16px` في حقول الإدخال يمنع التكبير التلقائي
3. **RTL Support:** استخدام `[dir="rtl"]` و `[dir="ltr"]` للعناصر الاتجاهية
4. **Focus States:** إضافة `.focused` class عند التركيز على الحقل

### 📂 ملفات CSS

| الملف | المحتوى |
|-------|---------|
| `_pages/Auth/_forget_password.css` | أنماط `.forgot-*` المشتركة |
| `_pages/Auth/Login.css` | أنماط `.login-*` الخاصة بصفحة الدخول |
| `_pages/Auth/Register.css` | أنماط `.register-*` الخاصة بصفحة التسجيل |
| `_Auth_Public_Bundle.css` | استيراد جميع ملفات المصادقة |

---

## 📱 دليل التجاوبية (Responsive Design Guidelines) (⭐ جديد - 18 يوليو 2026)

### 🎯 المبادئ الأساسية

1. **Mobile-First:** صمم للموبايل أولاً، ثم وسّع للشاشات الأكبر
2. **Touch-Friendly:** الحد الأدنى للعناصر القابلة للنقر: `48px × 48px`
3. **Readable:** الحد الأدنى لحجم الخط: `14px` (Body), `16px` (Inputs)
4. **Fast:** تجنب الصور الكبيرة، استخدم `lazy loading`

### 📏 Breakpoints المعيارية

| الاسم | العرض | الاستخدام |
|-------|-------|-----------|
| **Mobile (Portrait)** | ≤ 480px | هواتف صغيرة |
| **Mobile (Landscape)** | 481px - 767px | هواتف كبيرة / تابلت صغير |
| **Tablet** | 768px - 991px | تابلت |
| **Desktop** | 992px - 1199px | شاشات متوسطة |
| **Large Desktop** | ≥ 1200px | شاشات كبيرة |

### 🎨 قواعد التجاوبية

#### 1. Touch Targets (أهداف اللمس)

```css
/* ❌ خطأ: صغير جداً */
.btn {
    padding: 8px 16px;
    font-size: 12px;
}

/* ✅ صحيح: حجم مريح */
.btn {
    min-height: 48px;
    min-width: 48px;
    padding: 12px 24px;
    font-size: 14px;
}
```

#### 2. iOS Zoom Prevention (منع التكبير التلقائي)

```css
/* ❌ خطأ: iOS سيكبر الصفحة عند التركيز */
input {
    font-size: 14px;
}

/* ✅ صحيح: يمنع التكبير */
input {
    font-size: 16px; /* الحد الأدنى لمنع التكبير في iOS */
}
```

#### 3. Responsive Typography (خطوط متجاوبة)

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

#### 4. Responsive Spacing (مسافات متجاوبة)

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

### 🧪 اختبار التجاوبية

1. **Chrome DevTools:** `F12` ➔ `Toggle Device Toolbar` (Ctrl+Shift+M)
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

## 🗂️ هيكل ملفات الأنماط

```
Shared.UI/wwwroot/css/
 ├── _CoreBundle.css              (المتغيرات العالمية)
 ├── _ComponentsBundle.css        (مكونات مشتركة)
 ├── _LayoutBundle.css            (تخطيطات)
 ├── _PagesBundle.css             (استيراد جميع الصفحات)
 ├── _Auth_Public_Bundle.css      (⭐ صفحات المصادقة)
 │
 ├── _pages/
 │   ├── Auth/
 │   │   ├── _forget_password.css    (⭐ أنماط .forgot-* المشتركة)
 │   │   ├── Login.css               (⭐ أنماط .login-*)
 │   │   └── Register.css            (⭐ أنماط .register-*)
 │   │
 │   ├── Clinic/
 │   │   └── clinic-dashboard.css    (نمط العيادة)
 │   │
 │   ├── Pharmacy/
 │   │   └── pharmacy-dashboard.css  (نمط الصيدلية)
 │   │
 │   ├── Rep/
 │   │   └── rep-dashboard.css       (نمط المندوب)
 │   │
 │   ├── PSP/
 │   │   └── psp-gateway.css         (بوابة برامج الدعم)
 │   │
 │   └── Shared/
 │       └── dashboard-common.css    (مشترك - للصفحات القديمة)
```

---

## 🛠️ كيفية إضافة نمط جديد

1. إنشاء ملف CSS جديد في المجلد المناسب (مثل `_pages/Role/page-name.css`)
2. تعريف المتغيرات باستخدام `--role-*` prefix
3. استخدام المتغيرات في جميع التنسيقات
4. إضافة الاستيراد في `_PagesBundle.css` أو الباندل المناسب
5. تطبيق الـ Class الرئيسي (`role-root`) في Razor Component
6. ⭐ **إضافة Media Queries** للتجاوبية مع الموبايل
7. ⭐ **اختبار على 3 أجهزة:** Mobile, Tablet, Desktop

---

## ✅ CHECKLIST: عند إنشاء صفحة جديدة

### التحقق من النمط

- [ ] هل الصفحة تتبع نمط الدور الرئيسي (Clinic/Pharmacy/Rep)؟
- [ ] هل تستخدم المتغيرات الصحيحة (`--clinic-*` / `--pharmacy-*` / `--rep-*`)?
- [ ] هل الـ Hero Section يستخدم التدرج الصحيح؟
- [ ] هل أيقونات الأزرار تستخدم الألوان الصحيحة؟
- [ ] هل تم إضافة ملف CSS إلى الباندل المناسب؟

### ⭐ التحقق من التجاوبية (جديد - 18 يوليو 2026)

- [ ] هل الصفحة تعمل على الموبايل (≤ 480px)؟
- [ ] هل الأزرار بحجم `48px` على الأقل؟
- [ ] هل حقول الإدخال `font-size: 16px` (لمنع تكبير iOS)؟
- [ ] هل النصوص مقروءة بدون تكبير؟
- [ ] هل تم اختبار الصفحة على 3 أجهزة (Mobile, Tablet, Desktop)؟

### ⭐ التحقق من صفحات المصادقة (جديد - 18 يوليو 2026)

- [ ] هل تستخدم `.forgot-*` classes المشتركة؟
- [ ] هل Visual Panel يُخفى على الموبايل (`@media (max-width: 992px)`)?
- [ ] هل Form Panel يملأ الشاشة على الموبايل؟
- [ ] هل تم إضافة Media Queries للتجاوبية؟

### التحقق من الترجمة

- [ ] هل أضفت مفاتيح الترجمة في قاعدة البيانات مع `N'' prefix` للعربية؟
- [ ] هل تستخدم `T()` المساعدة بدلاً من `<LocalizedText />`؟

---

## 🔗 روابط ذات صلة

- [00 - الهيكل المعماري](00-architecture-overview.md)
- [05 - إنشاء الصفحات والمكونات](05-page-creation-checklist.md)
- [11 - دليل BlazorWebView](11-blazor-webview-guide.md)

---

**آخر تحديث:** 18 يوليو 2026 | **الملف:** `03-style-guide.md`
```

---

## 📊 ملخص التحديثات في هذه الوثيقة

| القسم | التغيير |
|-------|---------|
| **المحتوى الأصلي** | ✅ محفوظ بالكامل (نظام الألوان، أنماط الأدوار، الصفحات الداخلية، هيكل الملفات) |
| **المتغيرات الدلالية المتقدمة** | ✅ إضافة 16 متغير جديد (`--rubik-danger`, `--rubik-success`, إلخ) |
| **🔐 أنماط صفحات المصادقة** | ✅ إضافة كاملة: Split-Panel, `.forgot-*` classes, الألوان, التدرجات |
| **📱 دليل التجاوبية** | ✅ إضافة كاملة: Breakpoints, Touch Targets, iOS Zoom Prevention, Media Queries |
| **هيكل الملفات** | ✅ تحديث ليشمل ملفات Auth الجديدة |
| **CHECKLIST** | ✅ إضافة 8 نقاط تحقق جديدة (التجاوبية + المصادقة) |
