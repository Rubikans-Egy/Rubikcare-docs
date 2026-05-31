# 🎨 دليل الأنماط - RubikCare Style Guide

> **الإصدار:** 2.0  
> **آخر تحديث:** 31 مايو 2026  
> **الغرض:** توثيق نظام التنسيقات والألوان والمكونات البصرية لمشروع RubikCare

---

## 📋 جدول المحتويات

1. [نظرة عامة](#1-نظرة-عامة)
2. [نظام المتغيرات](#2-نظام-المتغيرات)
3. [هيكل الملفات](#3-هيكل-الملفات)
4. [مكتبة المكونات](#4-مكتبة-المكونات)
5. [أنماط التخطيط](#5-أنماط-التخطيط)
6. [أنماط الصفحات](#6-أنماط-الصفحات)
7. [أفضل الممارسات](#7-أفضل-الممارسات)
8. [أمثلة عملية](#8-أمثلة-عملية)

---

## 1. نظرة عامة

### 📍 المسار الأساسي
```
RubikCare.Shared.UI/wwwroot/css/
```

### 🏗️ فلسفة التنظيم

يعتمد نظام الأنماط على هيكل هرمي من 4 مستويات:

```
main.css                    ← نقطة الدخول الوحيدة (imports فقط)
  ├── _CoreBundle.css       ← المتغيرات + الأساسيات + الخطوط
  ├── _ComponentsBundle.css ← المكونات المستقلة
  ├── _LayoutBundle.css     ← تخطيطات الصفحات
  └── _PagesBundle.css      ← أنماط الصفحات
       ├── _AdminBundle.css
       ├── _Auth_Public_Bundle.css
       └── _OrgBundle.css
```

### 🎯 مبادئ التصميم

| المبدأ | الوصف |
|--------|-------|
| **الفصل التام** | Core لا يعتمد على شيء، Components تعتمد على Core، Layout تعتمد على Components |
| **BEM معدل** | نستخدم بادئات مختصرة لكل صفحة/مكون (مثال: `ps-` لـ Pharmacy Search) |
| **RTL أولاً** | كل الأنماط تدعم العربية بشكل افتراضي مع `[dir="rtl"]` |
| **Desktop-First** | Media Queries للشاشات الأصغر فقط |

---

## 2. نظام المتغيرات

### 📁 `_core/_variables.css`

```css
:root {
    /* ═══════════════════════════════════
       🎨 لوحة الألوان الرئيسية
       ═══════════════════════════════════ */

    /* Primary - الأزرق الطبي */
    --rubik-primary: #1B5A7A;
    --rubik-primary-dark: #0F3D55;
    --rubik-primary-light: #2C7AA0;

    /* Secondary - البنفسجي */
    --rubik-secondary: #6B4E9B;
    --rubik-secondary-light: #8B6FBB;

    /* Accent - البرتقالي الدافئ */
    --rubik-accent: #E87722;
    --rubik-accent-light: #F0A050;

    /* Success - الأخضر */
    --rubik-success: #2E7D5E;
    --rubik-success-light: rgba(46,125,94,0.12);

    /* Warning - الكهرماني */
    --rubik-warning: #B88B4A;
    --rubik-warning-light: rgba(184,139,74,0.12);

    /* Danger - الأحمر */
    --rubik-danger: #B33F40;
    --rubik-danger-light: rgba(179,63,64,0.1);

    /* Info - السماوي */
    --rubik-info: #3A7CA5;
    --rubik-info-light: rgba(58,124,165,0.12);

    /* ═══════════════════════════════════
       📝 ألوان النص
       ═══════════════════════════════════ */
    --rubik-text-primary: #1A2332;
    --rubik-text-secondary: #4A5568;
    --rubik-text-muted: #8A94A6;

    /* ═══════════════════════════════════
       🏠 ألوان الخلفية
       ═══════════════════════════════════ */
    --rubik-body-bg: #F5F7FA;
    --rubik-card-bg: #FFFFFF;
    --rubik-card-alt: #F8FAFC;

    /* ═══════════════════════════════════
       📏 الحدود والظلال
       ═══════════════════════════════════ */
    --rubik-border: #E2E8F0;
    --rubik-border-light: #EDF2F7;
    --rubik-radius: 12px;
    --rubik-radius-lg: 18px;
    --shadow-card: 0 2px 12px rgba(0,0,0,0.06);
}
```

---

## 3. هيكل الملفات

### 📂 الشجرة الكاملة

```
css/
├── main.css                          ← نقطة الدخول
├── webviewbundle.css                 ← خاص بـ MAUI WebView
│
├── _CoreBundle.css                   ← أساسيات + متغيرات + خطوط
│   └── _core/
│       ├── _variables.css
│       ├── _reset.css
│       ├── _fonts.css
│       ├── _base.css
│       └── _typography.css
│   └── _themes/
│       └── _rubik.css
│
├── _ComponentsBundle.css             ← مكونات UI مستقلة
│   └── _components/
│       ├── _cards.css
│       ├── _tables.css
│       ├── _buttons.css
│       ├── _forms.css
│       ├── _wizard.css
│       ├── _GenericModal.css
│       ├── _RubikDropdown.css
│       ├── _slider.css
│       └── _bottom-nav.css
│
├── _LayoutBundle.css                 ← تخطيطات + هيكل الصفحات
│   └── _layout/
│       ├── _containers.css
│       ├── _grid.css
│       ├── _header.css
│       ├── _sidebar.css
│       ├── _dashboard.css
│       ├── _footer.css
│       ├── _sidebar_orgs.css
│       ├── gallery.css
│       └── legal.css
│
├── _PagesBundle.css                  ← أنماط الصفحات
│   └── _pages/
│       ├── _homepage.css
│       ├── _centerhub.css
│       ├── _psp-about.css
│       ├── _PSPpresentation.css
│       ├── _doctor-presentation.css
│       ├── _contentswitch_flow.css
│       │
│       ├── Auth/
│       │   ├── Login.css
│       │   ├── Register.css
│       │   └── _forget_password.css
│       │
│       ├── PublicUser/
│       │   ├── Userprofile.css
│       │   ├── Settings.css
│       │   ├── Notifications.css
│       │   ├── CreateOrganization.css
│       │   ├── InvitationAcceptPage.css
│       │   └── PharmacySearch/
│       │       ├── pharmacy-card.css
│       │       ├── pharmacy-filter.css
│       │       ├── pharmacy-grid.css
│       │       ├── pharmacy-search.css
│       │       └── pharmacy-detail.css
│       │
│       ├── Organization/
│       │   ├── AllOrganizations.css
│       │   ├── MYOrganizations.css
│       │   ├── OrgSettings/
│       │   │   ├── OrgSettings.css
│       │   │   ├── MembersTab.css
│       │   │   ├── EditOrganization.css
│       │   │   ├── CustomJobTitles.css
│       │   │   └── MemberTitlestab.css
│       │   ├── Medications/
│       │   │   ├── CompanyMedicationsSpecialities.css
│       │   │   ├── MedicationForm.css
│       │   │   └── _medication-detail.css
│       │   └── PSP/
│       │       ├── PSPProgramsList.css
│       │       ├── PSPProgramDetails.css
│       │       ├── PSPProgramsEdit.css
│       │       └── PSPEditSteps/
│       │           ├── PSPStep0_BasicInfo.css
│       │           ├── PSPStep1_SpecsMeds.css
│       │           ├── PSPStep2_Budget.css
│       │           ├── PSPStep2_DispensationPlans.css
│       │           └── PSPStep3_Review.css
│       │
│       ├── Admin/
│       │   ├── MedicalSettings/
│       │   ├── Pricing/
│       │   ├── Services/
│       │   ├── GeneralSettings/
│       │   └── Users/
│       │
│       ├── Shared/
│       │   └── Shared_web_Mobile.css
│       │
│       └── LandingPages/
│           └── _publicuserslandingpage.css
│
├── _Auth_Public_Bundle.css           ← مصادقة + صفحات عامة
├── _AdminBundle.css                  ← لوحة التحكم
├── _OrgBundle.css                    ← المنظمات + PSP
└── _UtilitiesBundle.css              ← أدوات مساعدة
    └── _utilities/
        ├── _spacing.css
        ├── _colors.css
        └── observability.css
```

### 🔗 قاعدة تسجيل الملفات الجديدة

**عند إنشاء ملف CSS جديد:**

1. ضع الملف في المسار المناسب تحت `_pages/`
2. سجله في أقرب Bundle:
   - صفحات المنظمات ← `_OrgBundle.css`
   - صفحات المستخدم العادي ← `_Auth_Public_Bundle.css`
   - صفحات المشرف ← `_AdminBundle.css`

---

## 4. مكتبة المكونات

### 4.1 البطاقات (Cards)

#### `.ps-card` - بطاقة قابلة للنقر (قوائم البحث)

```css
.ps-card {
    display: flex;
    align-items: center;
    gap: 16px;
    padding: 16px;
    background: var(--rubik-card-bg);
    border-radius: 16px;
    border: 1.5px solid var(--rubik-border);
    transition: all 0.18s;
    cursor: pointer;
}
.ps-card:hover {
    border-color: rgba(27,90,122,0.2);
    box-shadow: 0 6px 20px rgba(27,90,122,0.12);
    transform: translateY(-2px);
}
```

**الاستخدام:** قوائم الصيدليات، قوائم المرضى، نتائج البحث

**البنية الداخلية:**
```
.ps-card
  ├── .ps-card-icon       ← أيقونة/صورة
  ├── .ps-card-body       ← المحتوى الرئيسي
  │   ├── .ps-card-name   ← الاسم
  │   └── .ps-card-alt    ← نص ثانوي
  └── .ps-card-arrow      ← سهم التنقل
```

---

### 4.2 الأزرار (Buttons)

| الكلاس | الاستخدام |
|--------|----------|
| `.trc-btn-primary` | زر رئيسي - إجراء أساسي |
| `.trc-btn-success` | زر نجاح - تأكيد/إكمال |
| `.trc-btn-secondary` | زر ثانوي - إجراء بديل |
| `.trc-btn-outline` | زر مخطط - أقل أهمية |

---

### 4.3 شريط الفلترة (Filter Bar)

#### `.ps-filter-bar` - شريط بحث وفلترة مثبت

```css
.ps-filter-bar {
    position: sticky;
    top: 60px;
    z-index: 10;
    background: var(--rubik-card-bg);
    border-radius: 0 0 18px 18px;
    box-shadow: 0 4px 20px rgba(27,90,122,0.1);
}
```

**المكونات الداخلية:**
```
.ps-filter-inner
  ├── .ps-search-wrap        ← حقل البحث
  │   ├── .ps-search-ico     ← أيقونة البحث
  │   └── .ps-search-input   ← إدخال النص
  ├── .ps-filter-select-wrap ← قائمة منسدلة للفلترة
  ├── .ps-search-btn         ← زر البحث
  └── .ps-view-toggle        ← تبديل عرض (شبكة/قائمة)
```

---

### 4.4 عرض البيانات (Grid/List)

```css
/* عرض الشبكة */
.ps-grid {
    display: grid;
    grid-template-columns: repeat(auto-fill, minmax(320px, 1fr));
    gap: 16px;
}

/* عرض القائمة */
.ps-grid.list-view {
    display: flex;
    flex-direction: column;
    gap: 12px;
}
```

---

## 5. أنماط التخطيط

### 5.1 ترويسة الصفحة (Hero)

```css
.ps-hero {
    background: linear-gradient(135deg,
        var(--rubik-primary-dark) 0%,
        var(--rubik-primary) 55%,
        var(--rubik-primary-light) 100%);
    padding: 2rem 2.5rem 0;
    overflow: hidden;
    position: relative;
}
```

**العناصر الزخرفية:**
- `.ps-hero-overlay` - تدرجات شفافة
- `.ps-hero-dots` - نمط نقاط

---

### 5.2 الحالات الخاصة

| الكلاس | الاستخدام |
|--------|----------|
| `.ps-loading-bar` | شريط تحميل متحرك أعلى الصفحة |
| `.ps-spinner` | دائرة تحميل دائرية |
| `.ps-skeleton` | هيكل تحميل وهمي |
| `.ps-empty` | حالة عدم وجود بيانات |
| `.trc-error-box` | مربع خطأ مع اهتزاز |
| `.trc-success-state` | حالة نجاح مع أنيميشن |

---

## 6. أنماط الصفحات

### 6.1 بادئات الصفحات (Namespacing)

| البادئة | الصفحة |
|---------|--------|
| `ps-` | Pharmacy Search (البحث عن صيدلية) |
| `pdp-` | Pharmacy Detail Page (تفاصيل الصيدلية) |
| `trc-` | Token Redemption Card (كارت صرف التوكن) |
| `mrc-` | Medication Request Card (كارت طلب دواء) |

---

### 6.2 مثال: صفحة البحث عن صيدلية

**الملفات:**
- `pharmacy-search.css` ← `.ps-root`, `.ps-hero`, `.ps-body`
- `pharmacy-filter.css` ← `.ps-filter-bar`, `.ps-search-wrap`
- `pharmacy-card.css` ← `.ps-card` ومكوناته
- `pharmacy-grid.css` ← `.ps-grid`, `.ps-pagination`
- `pharmacy-detail.css` ← `.pdp-root`, `.pdp-hero`

**الهيكل:**
```
ps-root
  ├── ps-loading-bar
  ├── ps-hero (ترويسة + إحصائيات)
  │   ├── ps-hero-overlay
  │   └── ps-hero-stats
  ├── ps-body
  │   ├── ps-filter-bar (بحث + فلترة)
  │   ├── ps-grid (عرض النتائج)
  │   │   └── ps-card × N
  │   └── ps-pagination
  └── ps-empty (عند عدم وجود نتائج)
```

---

## 7. أفضل الممارسات

### ✅ افعل

- استخدم متغيرات `--rubik-*` دائماً، لا تكتب ألوان ثابتة
- استخدم بادئة فريدة لكل صفحة جديدة (3-4 حروف)
- اختبر على الشاشات الصغيرة: `@media (max-width: 768px)`
- ادعم RTL: `[dir="rtl"]` للهوامش والأسهم
- سجل الملف في الـ Bundle المناسب

### ❌ لا تفعل

- لا تستخدم `!important` إلا للضرورة القصوى
- لا تكرر الأنماط - استخدم المكونات الموجودة
- لا تنشئ ملف CSS خارج نظام `_pages/`
- لا تستخدم ألوان ثابتة (مثل `#ff0000`) - استخدم المتغيرات
- لا تنسَ دعم RTL للعناصر التي تتأثر بالاتجاه

---

### 📝 قالب صفحة جديدة

```css
/* ================================================================
   [page-name].css — RubikCare
   الوصف: [وصف مختصر للصفحة]
   يعتمد على: _CoreBundle.css, _ComponentsBundle.css
================================================================ */

/* ── Root ── */
.xxx-root {
    min-height: 100vh;
    background: var(--rubik-body-bg);
    font-family: 'Tajawal', 'Cairo', sans-serif;
}

/* ── Hero ── */
.xxx-hero { }

/* ── Body ── */
.xxx-body { }

/* ── Responsive ── */
@media (max-width: 768px) { }

@media (max-width: 480px) { }
```

---

## 8. أمثلة عملية

### 8.1 إضافة صفحة جديدة - مثال كامل

**السيناريو:** إنشاء صفحة "قائمة المرضى" للطبيب

**الخطوة 1:** إنشاء ملف CSS
```
المسار: Shared.UI/wwwroot/css/_pages/Clinic/clinic-patients.css
البادئة: cpt- (Clinic PaTients)
```

**الخطوة 2:** تسجيل الملف
```css
/* في _Auth_Public_Bundle.css أو bundle مناسب */
@import url('_pages/Clinic/clinic-patients.css');
```

**الخطوة 3:** استخدام المكونات الموجودة
```html
<div class="cpt-root">
    <!-- استخدام ps-hero لنفس الترويسة -->
    <div class="ps-hero">...</div>
    
    <!-- استخدام ps-filter-bar للفلترة -->
    <div class="ps-filter-bar">...</div>
    
    <!-- استخدام ps-grid + ps-card للقائمة -->
    <div class="ps-grid list-view">
        <div class="ps-card cpt-patient-card">...</div>
    </div>
</div>
```

---

## 📚 المراجع

- [وثيقة Clean Architecture](./13-clean-architecture-enforcement.md)
- [دليل تطوير MAUI](./10-maui-development-guide.md)
- [نظرة عامة على المشروع](./00-architecture-overview.md)

---

> **آخر تحديث:** 31 مايو 2026 
