# 03 - دليل الأنماط (Style Guide)

آخر تحديث: 22 يونيو 2026

## 📌 مقدمة

هذا الدليل يوثق نظام الألوان والتصميم في منصة RubikCare، مع التركيز على الأنماط المخصصة لكل دور (عيادة، صيدلية، مندوب). الهدف هو ضمان تناسق بصري مع احترام هوية كل دور.

## 🎨 نظام الألوان العام (المتغيرات العالمية)

جميع المتغيرات العالمية معرفة في `_CoreBundle.css` وتستخدم كقاعدة للمتغيرات الخاصة بكل دور.

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

---

## 🏥 **نمط العيادة (Clinic) - أزرق بحري + أزرق-أخضر**

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
|----------|-------|-----------|
| `clinic-action-icon-clinic` | `#D1ECF1` | أيقونة "عيادتي" |
| `clinic-action-icon-psp` | `#D4EDDA` | أيقونة "برامج الدعم" |
| `clinic-action-icon-patients` | `#D1ECF1` | أيقونة "مرضاي" |
| `clinic-action-icon-invite` | `#F7F0E4` | أيقونة "دعواتي" |

### تطبيق النمط

**في Razor Component:**

```razor
<div class="clinic-root @LangClass">
    <div class="clinic-hero">...</div>
    <div class="clinic-actions-section">...</div>
    <div class="clinic-section">...</div>
</div>
```

**ملف CSS:** `_pages/Clinic/clinic-dashboard.css`

---

## 💊 **نمط الصيدلية (Pharmacy) - أخضر غامق + زمردي**

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
|----------|-------|-----------|
| `pharmacy-action-icon-pharmacy` | `#D4EDDA` | أيقونة "صيدليتي" |
| `pharmacy-action-icon-psp` | `#D4EDDA` | أيقونة "برامج الدعم" |
| `pharmacy-action-icon-products` | `#F7F0E4` | أيقونة "المنتجات" |
| `pharmacy-action-icon-orders` | `#D1ECF1` | أيقونة "طلبات المرضى" |

### تطبيق النمط

**في Razor Component:**

```razor
<div class="pharmacy-root @LangClass">
    <div class="pharmacy-hero">...</div>
    <div class="pharmacy-actions-section">...</div>
    <div class="pharmacy-section">...</div>
</div>
```

**ملف CSS:** `_pages/Pharmacy/pharmacy-dashboard.css`

---

## 🏢 **نمط المندوب (Rep) - ذهبي**

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

**في Razor Component:**

```razor
<div class="rep-root @LangClass">
    <div class="rep-hero">...</div>
    <div class="rep-actions-section">...</div>
    <div class="rep-section">...</div>
</div>
```

**ملف CSS:** `_pages/Rep/rep-dashboard.css`

---

## 📋 **الصفحات الداخلية (Sub-pages)**

### مبدأ التطبيق

**يجب أن تتبع الصفحات الداخلية نفس نظام ألوان الدور الرئيسي.**

**مثال:** إذا كان المستخدم طبيباً (دور Clinic)، فإن جميع الصفحات الداخلية (مثل `PspGateway`, `MyPatientInvitations`) يجب أن تستخدم ألوان العيادة (الأزرق البحري + الأزرق-الأخضر).

### كيفية التطبيق

1. **استخدم نفس `--clinic-*` أو `--pharmacy-*` أو `--rep-*` المتغيرات** في ملفات CSS الخاصة بالصفحات الداخلية.
2. **أو استخدم المتغيرات العالمية** (`--rubik-*`) مع الحفاظ على تناسق الألوان.

### مثال: `psp-gateway.css`

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

**بهذه الطريقة، الصفحات الداخلية ترث ألوان الدور الرئيسي تلقائياً.**

---

## 🗂️ **هيكل ملفات الأنماط**

```
Shared.UI/wwwroot/css/
├── _CoreBundle.css          (المتغيرات العالمية)
├── _PagesBundle.css         (استيراد جميع الصفحات)
│
├── _pages/
│   ├── Clinic/
│   │   └── clinic-dashboard.css        (نمط العيادة)
│   │
│   ├── Pharmacy/
│   │   └── pharmacy-dashboard.css      (نمط الصيدلية)
│   │
│   ├── Rep/
│   │   └── rep-dashboard.css           (نمط المندوب)
│   │
│   ├── PSP/
│   │   └── psp-gateway.css             (بوابة برامج الدعم)
│   │
│   └── Shared/
│       └── dashboard-common.css        (مشترك - للصفحات القديمة)
```

---

## 🛠️ **كيفية إضافة نمط جديد**

1. **إنشاء ملف CSS جديد** في المجلد المناسب (مثل `_pages/Role/page-name.css`).
2. **تعريف المتغيرات** باستخدام `--role-*` prefix.
3. **استخدام المتغيرات** في جميع التنسيقات.
4. **إضافة الاستيراد** في `_PagesBundle.css`.
5. **تطبيق الـ Class الرئيسي** (`role-root`) في Razor Component.

---

## ✅ **CHECKLIST: عند إنشاء صفحة جديدة**

- [ ] هل الصفحة تتبع نمط الدور الرئيسي (Clinic/Pharmacy/Rep)؟
- [ ] هل تستخدم المتغيرات الصحيحة (`--clinic-*` / `--pharmacy-*` / `--rep-*`)?
- [ ] هل الـ Hero Section يستخدم التدرج الصحيح؟
- [ ] هل أيقونات الأزرار تستخدم الألوان الصحيحة؟
- [ ] هل تم إضافة ملف CSS إلى `_PagesBundle.css`؟
- [ ] هل تم اختبار الصفحة على كل من (Web + Mobile Android + Mobile Windows)؟

---

## 🔗 روابط ذات صلة

- [00 - الهيكل المعماري](https://github.com/Rubikans-Egy/Rubikcare-docs/blob/main/docs/00-architecture-overview.md)
- [11 - دليل BlazorWebView](https://github.com/Rubikans-Egy/Rubikcare-docs/blob/main/docs/11-blazor-webview-guide.md)
- [05 - إنشاء الصفحات والمكونات](https://github.com/Rubikans-Egy/Rubikcare-docs/blob/main/docs/05-page-creation-checklist.md)

---

**آخر تحديث:** 22 يونيو 2026 | **الملف:** `03-style-guide.md`
```

---

## 📋 **ملخص التغييرات في الوثيقة**

| القسم | التغيير |
|-------|---------|
| **نظام الألوان العام** | ✅ إضافة جدول المتغيرات العالمية |
| **نمط العيادة** | ✅ إضافة الألوان الأساسية، التدرج، الأيقونات، التطبيق |
| **نمط الصيدلية** | ✅ إضافة الألوان الأساسية، التدرج، الأيقونات، التطبيق |
| **نمط المندوب** | ✅ إضافة الألوان الأساسية، التدرج، التطبيق |
| **الصفحات الداخلية** | ✅ شرح كيفية توريث الألوان من الدور الرئيسي |
| **هيكل الملفات** | ✅ تحديث الهيكل ليعكس الملفات الجديدة |
| **CHECKLIST** | ✅ إضافة نقاط تحقق جديدة |

