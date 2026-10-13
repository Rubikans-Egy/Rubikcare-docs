## `27-multi-layer-menu-control.md`

**الإصدار:** 1.0 | **آخر تحديث:** 6 أكتوبر 2026
**الحالة:** مقترح للاعتماد
**المؤلف:** RubikCare Team

---

## 📌 ملخص تنفيذي

هذه الوثيقة تُعرّف نظامًا معماريًا موحّدًا للتحكم في القوائم والصلاحيات والميزات عبر المنصات الثلاث (Blazor Web + PWA + MAUI)، من خلال **أربع طبقات تحكم** مستقلة ومتكاملة:

1. **طبقة المنصة** (Platform Control) — ما هو متاح لنوع المنظمة.
2. **طبقة الاشتراكات** (Subscriptions Control) — ما هو مفعّل في اشتراك المنظمة.
3. **طبقة المنظمة** (Organization Control) — ما سمحت به المنظمة لوظائفها.
4. **طبقة الوظيفة** (JobTitle Control) — ما تراه كل وظيفة داخل المنظمة.

**الهدف:** توحيد مصدر الحقيقة، إزالة الفوضى، تمكين المنظمات من التحكم الذاتي، وتفعيل نظام الاشتراكات والصلاحيات في حل واحد.

**المدة المقدرة للتنفيذ:** 7-8 جلسات عمل مركزة (~أسبوع إلى 10 أيام).

---

## 🎯 الجزء 0: الوثيقة المعمارية (هذا المستند)

### 0.1 الهدف من هذا الجزء
وضع **حجر الأساس** للنظام الجديد، وتوثيق:
- الفلسفة المعمارية.
- الطبقات الأربع.
- نموذج البيانات الموسّع.
- توزيع المسؤوليات بين الطبقات التقنية.
- خارطة التنفيذ على أربع مراحل.
- معايير القبول.

### 0.2 الفلسفة المعمارية

#### المبدأ الأساسي
> **"لا نبني نظامًا جديدًا، بل نُوسّع ونُوحّد الموجود."**

#### القواعد الذهبية
1. **التطوير التراكمي الآمن:** لا تغييرات جذرية مفاجئة.
2. **مصدر حقيقة واحد:** كل بيانات القوائم في جداول مركزية.
3. **فصل الطبقات:** كل طبقة تحكم لها مسؤولية واضحة.
4. **إعادة الاستخدام:** 80% مما نحتاجه موجود.
5. **التوحيد عبر المنصات:** كود واحد، ثلاثة أغلفة رقيقة.
6. **الأولوية للأدق:** الطبقة الأدق تتجاوز الأعلى عمومية.

---

## 🏛️ الجزء 1: الطبقات الأربع للتحكم

### 1.1 نظرة عامة

```
┌─────────────────────────────────────────────────────────┐
│  الطبقة 1: المنصة (Platform)                             │
│  من يتحكم؟ Super Admin                                   │
│  ماذا؟ القوائم الرئيسية لكل نوع منظمة                    │
│  المعيار: OrganizationType + MenuAssignments             │
└─────────────────────────────────────────────────────────┘
                          ↓
┌─────────────────────────────────────────────────────────┐
│  الطبقة 2: الاشتراكات (Subscriptions)                    │
│  من يتحكم؟ Super Admin                                   │
│  ماذا؟ الميزات المفعّلة في اشتراك المنظمة                 │
│  المعيار: Features + Plans + Subscriptions               │
└─────────────────────────────────────────────────────────┘
                          ↓
┌─────────────────────────────────────────────────────────┐
│  الطبقة 3: المنظمة (Organization)                        │
│  من يتحكم؟ Admin المنظمة                                 │
│  ماذا؟ ما سمحت به المنظمة لعناصر قوائمها                 │
│  المعيار: MenuItemVisibility (Org-level)                │
└─────────────────────────────────────────────────────────┘
                          ↓
┌─────────────────────────────────────────────────────────┐
│  الطبقة 4: الوظيفة (JobTitle)                            │
│  من يتحكم؟ Admin المنظمة                                 │
│  ماذا؟ ما تراه كل وظيفة داخل المنظمة                     │
│  المعيار: CustomJobTitle + MenuItemVisibility            │
└─────────────────────────────────────────────────────────┘
                          ↓
              ┌───────────────────────┐
              │   القائمة النهائية    │
              │   للمستخدم الحالي     │
              └───────────────────────┘
```

### 1.2 الطبقة 1: المنصة (Platform Control)

#### الوصف
- **من يتحكم؟** Super Admin (RubikCare).
- **ماذا يتحكم؟** القوائم الرئيسية + عناصرها لكل `OrganizationType`.
- **المعيار:** `OrganizationType` + `MenuAssignments`.
- **النطاق:** كل المنصات.
- **الحالة:** ✅ موجودة وتعمل.

#### الوظائف
- إضافة قائمة رئيسية.
- إضافة عناصر للقائمة.
- إخفاء/إظهار عنصر.
- نقل عنصر بين قوائم.
- إظهار عنصر في عدة قوائم.

#### الجداول المستخدمة
- `SystemMenus`
- `MenuItems`
- `MenuAssignments` (بمعيار `OrganizationType`)

### 1.3 الطبقة 2: الاشتراكات (Subscriptions Control)

#### الوصف
- **من يتحكم؟** Super Admin.
- **ماذا يتحكم؟** الميزات المفعّلة في اشتراك كل منظمة.
- **المعيار:** `Features` + `Plans` + `Subscriptions`.
- **النطاق:** كل المنصات.
- **الحالة:** ⚠️ البنية موجودة، غير مفعّلة.

#### الوظائف
- تعريف الميزات (`Features`).
- تعريف الخطط (`Plans`).
- ربط الميزات بالخطط (`PlanFeatures`).
- ربط المنظمات بالخطط (`OrganizationSubscriptions`).
- تفعيل/تعطيل ميزة لمنظمة معينة.

#### الجداول المستخدمة
- `Features` (موجود)
- `Plans` (موجود)
- `Pricing` (موجود)
- `PlanFeatures` (يُضاف إن لم يكن موجودًا)
- `OrganizationSubscriptions` (يُضاف إن لم يكن موجودًا)
- `FeatureMenuItems` (جديد — يربط الميزة بعناصر القائمة)

#### آلية العمل
عند بناء القائمة لمستخدم:
1. يُحدد اشتراك المنظمة النشطة.
2. تُحدد الميزات المفعّلة.
3. تُفلتر `MenuItems` بحيث تظهر فقط العناصر المرتبطة بميزات مفعّلة.

### 1.4 الطبقة 3: المنظمة (Organization Control)

#### الوصف
- **من يتحكم؟** Admin المنظمة.
- **ماذا يتحكم؟** ما سمحت به المنظمة لعناصر قوائمها (إظهار/إخفاء).
- **المعيار:** `MenuItemVisibility` على مستوى المنظمة.
- **النطاق:** عناصر قوائم المنظمة فقط.
- **الحالة:** 🆕 جديدة.

#### الوظائف
- عرض عناصر قوائم المنظمة (المتاحة وفق اشتراكها).
- إظهار/إخفاء عنصر على مستوى المنظمة (لكل الوظائف).
- **لا يضيف** عناصر جديدة.
- **لا ينقل** عناصر بين قوائم.

#### الجداول المستخدمة
- `MenuItemVisibility` (جديد)

#### الفرق بين الطبقة 1 والطبقة 3
| البند | الطبقة 1 (المنصة) | الطبقة 3 (المنظمة) |
|-------|---------------------|----------------------|
| **من يتحكم؟** | Super Admin | Admin المنظمة |
| **النطاق** | كل المنظمات | منظمة واحدة |
| **الوظائف** | إضافة/نقل/إخفاء | إظهار/إخفاء فقط |
| **المستوى** | `MenuItem` عام | `MenuItem` داخل منظمة |

### 1.5 الطبقة 4: الوظيفة (JobTitle Control)

#### الوصف
- **من يتحكم؟** Admin المنظمة.
- **ماذا يتحكم؟** ما تراه كل وظيفة (`CustomJobTitle`) داخل المنظمة.
- **المعيار:** `CustomJobTitle` + `MenuItemVisibility`.
- **النطاق:** وظائف المنظمة فقط.
- **الحالة:** 🆕 جديدة.

#### الوظائف
- عرض وظائف المنظمة (`CustomJobTitles`).
- ربط كل وظيفة بعناصر مرئية.
- معاينة القائمة كما سيراها المستخدم بهذه الوظيفة.

#### الجداول المستخدمة
- `CustomJobTitles` (موجود)
- `MenuItemVisibility` (جديد — بمعيار `CustomJobTitleId`)

### 1.6 ترتيب الأولويات بين الطبقات

عند بناء القائمة لمستخدم، تُطبّق الطبقات بالترتيب التالي:

```
1. الطبقة 1: تحديد القوائم المتاحة لنوع المنظمة
   ↓
2. الطبقة 2: فلترة القوائم وفق الاشتراك
   ↓
3. الطبقة 3: فلترة العناصر وفق إعدادات المنظمة
   ↓
4. الطبقة 4: فلترة العناصر وفق وظيفة المستخدم
   ↓
5. القائمة النهائية
```

#### قاعدة الأولوية
> **"الطبقة الأدق تتجاوز الأعلى عمومية."**

- إذا أظهرت الطبقة 3 عنصرًا، لكن الطبقة 4 حجبته عن وظيفة المستخدم → **لا يظهر**.
- إذا حجبت الطبقة 3 عنصرًا، لكن الطبقة 4 سمحت به → **لا يظهر** (لأن الطبقة 3 أعلى).
- **الاستثناء:** `UserProfileID` في `MenuAssignment` (تخصيص مباشر) يتجاوز كل الطبقات.

---

## 🗄️ الجزء 2: نموذج البيانات الموسّع (ERD)

### 2.1 الجداول الموجودة (لا تُعدّل)

| الجدول | الوظيفة |
|--------|---------|
| `SystemMenus` | القوائم الرئيسية |
| `MenuItems` | عناصر القوائم |
| `MenuAssignments` | تخصيص القوائم |
| `CustomJobTitles` | وظائف الأعضاء |
| `OrgMemberships` | عضويات المستخدمين |
| `Organizations` | المنظمات |
| `OrganizationTypes` | أنواع المنظمات |
| `Features` | الميزات/الخدمات |
| `Plans` | الخطط |
| `Pricing` | التسعير |
| `AspNetRoles` | الأدوار النظامية |
| `UserProfiles` | ملفات المستخدمين |

### 2.2 الجداول الجديدة

#### الجدول 1: `MenuItemVisibility`
**الوظيفة:** يربط `MenuItem` بـ `CustomJobTitleId` داخل منظمة محددة.

| الحقل | النوع | الوصف |
|-------|------|-------|
| `VisibilityId` | INT PK | المعرف |
| `OrganizationId` | INT FK | المنظمة |
| `CustomJobTitleId` | INT FK NULL | الوظيفة (NULL = على مستوى المنظمة) |
| `MenuItemId` | INT FK | عنصر القائمة |
| `IsVisible` | BIT | مرئي/مخفي |
| `CreatedDate` | DATETIME2 | تاريخ الإنشاء |
| `CreatedBy` | INT FK | من أنشأ |
| `ModifiedDate` | DATETIME2 | تاريخ التعديل |

**الفهارس:**
- `IX_MenuItemVisibility_Org_JobTitle` (`OrganizationId`, `CustomJobTitleId`)
- `IX_MenuItemVisibility_MenuItem` (`MenuItemId`)

**الاستخدام:**
- `CustomJobTitleId = NULL` → إعداد على مستوى المنظمة (الطبقة 3).
- `CustomJobTitleId = X` → إعداد لوظيفة محددة (الطبقة 4).

#### الجدول 2: `FeatureMenuItems`
**الوظيفة:** يربط `Feature` بـ `MenuItem`.

| الحقل | النوع | الوصف |
|-------|------|-------|
| `Id` | INT PK | المعرف |
| `FeatureId` | INT FK | الميزة |
| `MenuItemId` | INT FK | عنصر القائمة |
| `IsActive` | BIT | مفعّل/معطّل |

**الاستخدام:** عند تفعيل ميزة لمنظمة، تظهر عناصرها المرتبطة.

#### الجدول 3: `PlanFeatures` (إن لم يكن موجودًا)
**الوظيفة:** يربط `Plan` بـ `Feature`.

| الحقل | النوع | الوصف |
|-------|------|-------|
| `Id` | INT PK | المعرف |
| `PlanId` | INT FK | الخطة |
| `FeatureId` | INT FK | الميزة |
| `IsIncluded` | BIT | مضمنة/غير مضمنة |

#### الجدول 4: `OrganizationSubscriptions` (إن لم يكن موجودًا)
**الوظيفة:** يربط `Organization` بـ `Plan`.

| الحقل | النوع | الوصف |
|-------|------|-------|
| `SubscriptionId` | INT PK | المعرف |
| `OrganizationId` | INT FK | المنظمة |
| `PlanId` | INT FK | الخطة |
| `StartDate` | DATETIME2 | تاريخ البدء |
| `EndDate` | DATETIME2 | تاريخ الانتهاء |
| `IsActive` | BIT | نشط/غير نشط |

### 2.3 التوسيعات على الجداول الموجودة

#### `MenuItems` — إضافة أعمدة
| العمود | النوع | الوصف |
|--------|------|-------|
| `Platform` | NVARCHAR(20) | `Web` / `Mobile` / `Both` |
| `IsMobileVisible` | BIT | مرئي في الموبايل |
| `FeatureId` | INT FK NULL | الميزة المرتبطة (اختياري) |

#### `MenuAssignments` — إضافة عمود
| العمود | النوع | الوصف |
|--------|------|-------|
| `CustomJobTitleId` | INT FK NULL | الوظيفة (لمن تظهر القائمة) |

### 2.4 مخطط العلاقات (مبسّط)

```
Organizations ──┬── OrganizationSubscriptions ── Plans ── PlanFeatures ── Features
                │                                                            │
                │                                                            ↓
                ├── OrgMemberships ── CustomJobTitles              FeatureMenuItems
                │                                                            │
                │                                                            ↓
                └── MenuItemVisibility ──────────────────────────── MenuItems
                                                                             │
                                                                             ↓
                                                                       SystemMenus
                                                                             │
                                                                             ↓
                                                                     MenuAssignments
```

---

## 🏗️ الجزء 3: توزيع المسؤوليات التقنية

### 3.1 المبدأ
> **"منطق واحد في UseCase، عرض واحد في Shared.UI، ثلاثة أغلفة رقيقة."**

### 3.2 التوزيع

| الطبقة | النوع | الموقع | المسؤولية |
|--------|------|--------|-----------|
| **Domain** | منطق نقي | `Rubikcare.Domain` | قواعد الأولوية، تقييم الصلاحية |
| **Application** | UseCase | `Rubikcare.Application` | `GetVisibleMenuItemsUseCase` |
| **Infrastructure** | Handler + Repositories | `Rubikcare.Infrastructure` | التنفيذ + الوصول للبيانات |
| **Shared.UI** | Service + Components | `Rubikcare.Shared.UI` | `MenuStateService` + `DynamicMenuComponent` + `RouteGuard` |
| **Web** | Thin Wrapper | `Rubikcare.Web` | استدعاء `Shared.UI` |
| **MAUI** | Thin Wrapper | `Rubikcare.Mobile` | استدعاء `Shared.UI` داخل `BlazorWebView` |
| **PWA** | Thin Wrapper | `Rubikcare.PWA` | استدعاء `Shared.UI` |

### 3.3 UseCase + Handler

#### `GetVisibleMenuItemsUseCase`
- **الموقع:** `Rubikcare.Application`
- **الوظيفة:** يحدد القائمة النهائية للمستخدم.
- **المدخلات:** `UserProfileId`, `ActiveOrganizationId`, `Platform`
- **المخرجات:** `List<VisibleMenuItem>`

#### `GetVisibleMenuItemsHandler`
- **الموقع:** `Rubikcare.Infrastructure`
- **الوظيفة:** تنفيذ الاستعلامات على قاعدة البيانات.
- **يعتمد على:** `DbContextFactory` + Repositories.

#### لماذا هذا النمط؟
- **فصل تام** بين المنطق والتنفيذ.
- **قابلية اختبار** عالية.
- **إعادة استخدام** عبر المنصات الثلاثة.
- **متوافق** مع النمط الحالي في المشروع.

### 3.4 Shared.UI

#### `MenuStateService`
- **الوظيفة:** يحتفظ بالقوائم المفلترة للمستخدم.
- **يدعم:** تعدد المنظمات (كاش لكل منظمة).
- **يعتمد على:** `GetVisibleMenuItemsUseCase`.

#### `DynamicMenuComponent`
- **الوظيفة:** عرض القائمة.
- **يُستدعى في:** Web + PWA + MAUI.
- **يعتمد على:** `MenuStateService`.

#### `RouteGuard`
- **الوظيفة:** منع الوصول المباشر للصفحات المحجوبة.
- **يعتمد على:** `MenuStateService`.

---

## 🗺️ الجزء 4: خارطة التنفيذ (4 مراحل)

### 4.1 نظرة عامة

| المرحلة | الوصف | الجلسات |
|---------|-------|---------|
| **المرحلة 1** | إدارة الخدمات والميزات | 2 |
| **المرحلة 2** | إدارة الصلاحيات | 2 |
| **المرحلة 3** | تحويل القوائم الثلاثة لديناميكية | 2 |
| **المرحلة 4** | الاختبار الشامل | 2 |
| **الإجمالي** | | **8 جلسات** |

### 4.2 المرحلة 1: إدارة الخدمات والميزات

#### الهدف
تفعيل نظام `Features` + `Plans` + `Subscriptions` وربطه بالقوائم.

#### المهام
1. مراجعة الجداول الموجودة (`Features`, `Plans`, `Pricing`).
2. إضافة `PlanFeatures` (إن لم يكن موجودًا).
3. إضافة `OrganizationSubscriptions` (إن لم يكن موجودًا).
4. إضافة `FeatureMenuItems` (جديد).
5. إضافة `FeatureId` إلى `MenuItems`.
6. Migration آمن.
7. بناء `GetSubscribedFeaturesUseCase`.
8. بناء Handler.
9. تحديث شاشة تفاصيل المنظمة (إدارة الميزات).

#### المخرجات
- جداول مفعّلة.
- UseCase + Handler.
- شاشة إدارة الميزات.

### 4.3 المرحلة 2: إدارة الصلاحيات

#### الهدف
إضافة طبقتين: المنظمة (Org) والوظيفة (JobTitle).

#### المهام
1. إضافة `MenuItemVisibility`.
2. إضافة `CustomJobTitleId` إلى `MenuAssignments`.
3. Migration آمن.
4. بناء `GetVisibleMenuItemsUseCase` (يدمج الطبقات الأربع).
5. بناء Handler.
6. بناء شاشة تحكم المنظمة (مشتركة، ديناميكية).
7. بناء منطق تقييم `MenuItem.RequiredPermission`.

#### المخرجات
- جداول جديدة.
- UseCase موحّد للطبقات الأربع.
- شاشة تحكم المنظمة.

### 4.4 المرحلة 3: تحويل القوائم الثلاثة لديناميكية

#### الهدف
توحيد القوائم عبر Web + PWA + MAUI.

#### المهام
1. نقل قائمة PWA الحالية إلى `SystemMenus` + `MenuItems`.
2. إضافة عمود `Platform` / `IsMobileVisible` إلى `MenuItems`.
3. بناء `MenuStateService` (كاش متعدد المنظمات).
4. بناء `DynamicMenuComponent`.
5. دمج `DynamicMenuComponent` في Web.
6. دمج `DynamicMenuComponent` في PWA.
7. استبدال `AppShell` في MAUI:
   - الاحتفاظ بـ Shell Host بسيط.
   - استدعاء `DynamicMenuComponent` في `BlazorWebView`.
8. بناء `RouteGuard`.

#### المخرجات
- قائمة موحّدة عبر المنصات.
- `AppShell` مبسّط.
- `RouteGuard` فعّال.

### 4.5 المرحلة 4: الاختبار الشامل

#### الهدف
التحقق من سلامة النظام عبر المنصات الثلاث.

#### المهام
1. اختبار Web:
   - تسجيل دخول.
   - تبديل منظمات.
   - رؤية القوائم الصحيحة.
   - `RouteGuard` يمنع الوصول المباشر.
2. اختبار PWA:
   - نفس السيناريوهات.
3. اختبار MAUI:
   - نفس السيناريوهات.
   - `BlazorWebView` يعمل.
4. اختبار السيناريوهات الحدّية:
   - مستخدم في 3 منظمات.
   - وظائف مختلفة.
   - ميزات معطّلة.
5. إصلاح Bugs.
6. توثيق الميزة الجديدة.

#### المخرجات
- نظام مستقر.
- توثيق محدّث.
- تقرير اختبار.

---

## ⚠️ الجزء 5: المخاطر والتخفيف

| المخاطرة | الاحتمال | الأثر | التخفيف |
|----------|-----------|--------|----------|
| مشاكل `BlazorWebView` في MAUI | متوسط | مرتفع | اختبار مبكر + وثيقة `11-blazor-webview-guide.md` |
| Migration خاطئ | منخفض | مرتفع | نسخ احتياطي + اختبار على بيئة تجريبية |
| تضارب في الكاش (تعدد المنظمات) | متوسط | متوسط | `MenuStateService` مصمم لهذا |
| RouteGuard لا يعمل في PWA | منخفض | متوسط | اختبار مبكر + fallback |
| تعطيل `AppShell` يكسر التنقل | مرتفع | مرتفع | الاحتفاظ بـ Shell Host بسيط |
| بطء الأداء | منخفض | متوسط | `UserSessionService` كاش مركزي |
| تضارب الأولويات بين الطبقات | متوسط | مرتفع | وثيقة الأولويات + اختبارات |

---

## ✅ الجزء 6: معايير القبول (Definition of Done)

### 6.1 معايير وظيفية
- [ ] المستخدم يرى القوائم الصحيحة وفق الطبقات الأربع.
- [ ] تبديل المنظمات يعمل ويُحدّث القوائم.
- [ ] `RouteGuard` يمنع الوصول المباشر للصفحات المحجوبة.
- [ ] المنصة تتحكم في الميزات من شاشة تفاصيل المنظمة.
- [ ] المنظمة تتحكم في صلاحيات الوظائف من شاشتها.
- [ ] `CustomJobTitle` يُستخدم كمعيار للصلاحيات.

### 6.2 معايير تقنية
- [ ] `GetVisibleMenuItemsUseCase` موجود في Application.
- [ ] Handler موجود في Infrastructure.
- [ ] `MenuStateService` يدعم تعدد المنظمات.
- [ ] `DynamicMenuComponent` يعمل في Web + PWA + MAUI.
- [ ] `AppShell` مبسّط ولا يكسر التنقل.
- [ ] جداول جديدة موثّقة ومفهرسة.

### 6.3 معايير تجربة المستخدم
- [ ] القائمة تظهر بسرعة (< 1 ثانية).
- [ ] تبديل المنظمات سلس.
- [ ] لا وميض في الاتجاه (RTL/LTR).
- [ ] `RouteGuard` يعطي رسالة واضحة.

### 6.4 معايير التوثيق
- [ ] `15-multi-layer-menu-control.md` محدّثة.
- [ ] `04-dynamic-menus.md` محدّثة.
- [ ] `02-identity-system.md` محدّثة.
- [ ] `14-caching-system.md` محدّثة.

---


## 📄 الوثيقة الموحدة `15-multi-layer-menu-control.md
```markdown
# 15 - نظام القوائم والصلاحيات الشامل

**الإصدار:** 3.0 (Unified Edition — Menu & Permissions System)
**آخر تحديث:** 13 أكتوبر 2026
**الحالة:** ✅ مرجع معماري إلزامي — يقرأه كل مطور قبل العمل على القوائم
**المؤلف:** RubikCare Team

---

## 📌 الجزء صفر: الرؤية والفلسفة (نص المؤلف الحرفي)

> **⚠️ ملاحظة:** هذا القسم يحتوي على **نص الكاتب الحرفي** — دون إعادة صياغة.
> الهدف: حفظ الرؤية الكاملة كما صيغت.

---

### 0.1 الرؤية الكاملة للنظام

> **نص المؤلف الحرفي:**
> 
> "راجع هذه الوثيقة جيداً والخاصة بتحويل جذري لإدارة القوائم في المنصة.
> وذلك بهدف إظهار أو إخفاء هذه الميزات بشكل ديناميكي من لوحة التحكم في المنصة وليس عبر الكود، وفقاً لنظام اشتراكات أو خطط تسويق مختلفة ومرنة.
> 
> لاحظ أن النظام موجود في الويب ولكن مطلوب تعميمه ليكون في منصات العرض كلها مع تحكم تام مركزي، مرن أيضاً بنفس نمط تفكيري."

**الترجمة التقنية:**
- **مصدر الحقيقة** = قاعدة البيانات (لا كود).
- **التعميم** = Web + PWA + MAUI (نفس النظام).
- **المرونة** = Subscriptions + Plans + Dynamic Visibility.
- **التحكم المركزي** = لوحة تحكم Super Admin.

---

### 0.2 سبب الحاجة للنظام (الربط بنظام النقاط)

> **نص المؤلف الحرفي:**
> 
> "لماذا أخبرك بهذه الجزئية الآن؟
> أخبرك بها لأننا سنحتاج بعد الانتهاء من backend ما ننجزه الآن أن نفكر في مراعاة هذه الجزئية المذكورة في تصميم الواجهات، وتعديل الوضع الحالي ليسمح بمرونة أكثر، بدلاً من كونها ميزة مدمجة في داخل برامج الدعم.
> 
> فلا نحتاج مستقبلاً لتطوير واجهات مخصصة لكل خدمة نريد إضافة نظام نقاط لها."

**الترجمة التقنية:**

**المشكلة الحالية:**
- نظام النقاط **مدمج داخل PSP**.
- لو أردنا نظام نقاط في B2B → نسخ كود + تعديلات.

**بعد وثيقة 15:**
- نظام النقاط يصبح **ميزة (Feature)**.
- المنظمة تشترك في **Plan** يحتوي على `POINTS_SYSTEM`.
- **لا كود مخصص لكل خدمة.**
- **نفس الميزة، نفس الواجهة، نفس المحرك.**

---

### 0.3 تعميم النظام على كل المنصات

> **نص المؤلف الحرفي:**
> 
> "لاحظ أن النظام موجود في الويب ولكن مطلوب تعميمه ليكون في منصات العرض كلها مع تحكم تام مركزي."

**الترجمة التقنية:**

| المنصة | الحالة الحالية | الهدف |
|:------:|:--------------:|:-----:|
| **Web** | ✅ النظام مُطبَّق | يبقى |
| **PWA** | 🟡 قائمة ثابتة | تُطبَّق 4 طبقات |
| **MAUI** | 🟡 AppShell ثابت | يُستبدل بـ `DynamicMenuComponent` |

**الحل:** `DynamicMenuComponent` في `Shared.UI` — يُستخدم في **الثلاث منصات**.

---

## 📌 الملخص التنفيذي

هذه الوثيقة تُعرّف نظامًا معماريًا موحّدًا للتحكم في القوائم والصلاحيات والميزات عبر المنصات الثلاث (Blazor Web + PWA + MAUI)، من خلال **أربع طبقات تحكم** مستقلة ومتكاملة، مع **آلية Rules** ديناميكية.

**الهدف:** توحيد مصدر الحقيقة، إزالة الفوضى، تمكين المنظمات من التحكم الذاتي، وتفعيل نظام الاشتراكات والصلاحيات في حل واحد.

---

## 🏛️ الجزء الأول: البنية المعمارية

### 1.1 المبدأ الأساسي

> **"لا نبني نظامًا جديدًا، بل نُوسّع ونُوحّد الموجود."**

### 1.2 الطبقات الأربع للتحكم

```
┌─────────────────────────────────────────────────────────┐
│  الطبقة 1: المنصة (Platform)                             │
│  من يتحكم؟ Super Admin                                   │
│  ماذا؟ القوائم الرئيسية لكل نوع منظمة                    │
│  المعيار: OrganizationType + MenuAssignments             │
└─────────────────────────────────────────────────────────┘
                          ↓
┌─────────────────────────────────────────────────────────┐
│  الطبقة 2: الاشتراكات (Subscriptions)                    │
│  من يتحكم؟ Super Admin                                   │
│  ماذا؟ الميزات المفعّلة في اشتراك المنظمة                 │
│  المعيار: MenuVisibilityRules (SubscriptionPlanID)       │
└─────────────────────────────────────────────────────────┘
                          ↓
┌─────────────────────────────────────────────────────────┐
│  الطبقة 3: المنظمة (Organization)                        │
│  من يتحكم؟ Admin المنظمة                                 │
│  ماذا؟ ما سمحت به المنظمة لعناصر قوائمها                 │
│  المعيار: MenuVisibilityRules (OrganizationID)           │
└─────────────────────────────────────────────────────────┘
                          ↓
┌─────────────────────────────────────────────────────────┐
│  الطبقة 4: الوظيفة (JobTitle)                            │
│  من يتحكم؟ Admin المنظمة                                 │
│  ماذا؟ ما تراه كل وظيفة داخل المنظمة                     │
│  المعيار: MenuVisibilityRules (RoleId)                   │
└─────────────────────────────────────────────────────────┘
```

### 1.3 ترتيب الأولويات بين الطبقات

```
1. MenuAssignments → تحديد القوائم المتاحة لنوع المنظمة
   ↓
2. MenuVisibilityRules → تطبيق القواعد (RuleType + Priority)
   ↓
3. القائمة النهائية للمستخدم
```

**قاعدة الأولوية:**

> **"Priority الأصغر = الأولوية الأعلى."**

**مثال:**
- Rule 1: `RuleType=All`, `Priority=100`, `IsVisible=true` → يظهر للجميع.
- Rule 2: `RuleType=Organization`, `OrganizationID=5`, `Priority=50`, `IsVisible=false` → يخفي عن Org 5.
- **النتيجة لـ Org 5:** **الاثنان مطابقان** → **الأعلى أولوية (Rule 2, Priority=50) يفوز** → **مخفي**.

---

## 🗄️ الجزء الثاني: نموذج البيانات (ERD)

### 2.1 الجداول الأساسية

| # | الجدول | الدور |
|:-:|--------|-------|
| 1 | `SystemMenus` | القوائم الرئيسية (USER_BASE, ADMIN_MENU, LOYALTY, ...) |
| 2 | `MenuItems` | عناصر القوائم (Items) |
| 3 | `MenuAssignments` | ربط القوائم بأنواع المنظمات |
| 4 | **`MenuVisibilityRules`** | **قواعد الإظهار/الإخفاء الديناميكية** |
| 5 | `CustomJobTitles` | وظائف الأعضاء |
| 6 | `OrgMemberships` | عضويات المستخدمين |

### 2.2 `MenuVisibilityRules` — التفصيل

| العمود | النوع | الوصف |
|--------|------|-------|
| `RuleID` | `INT PK` | المعرف |
| `ItemID` | `INT FK` | العنصر المستهدف |
| `RuleType` | `INT` | نوع القاعدة (enum) |
| `OrganizationID` | `INT NULL` | منظمة محددة |
| `OrganizationTypeID` | `INT NULL` | نوع منظمة |
| `SubscriptionPlanID` | `INT NULL` | خطة اشتراك |
| `RoleId` | `NVARCHAR NULL` | دور مستخدم |
| `IsVisible` | `BIT` | مرئي/مخفي |
| `EffectiveFrom` | `DATETIME2` | بداية الصلاحية |
| `EffectiveTo` | `DATETIME2 NULL` | نهاية الصلاحية |
| **`Priority`** | `INT` | **الأصغر = الأعلى** |
| `IsActive` | `BIT` | نشط/معطّل |

### 2.3 `RuleType` Enum

```csharp
public enum RuleType
{
    All = 0,                // للجميع
    Organization = 1,       // منظمة محددة
    OrganizationType = 2,   // نوع منظمة
    Subscription = 3,       // خطة اشتراك
    Role = 4,               // دور مستخدم
    User = 5                // مستخدم محدد (مستقبلاً)
}
```

---

## 🏗️ الجزء الثالث: توزيع المسؤوليات التقنية

### 3.1 المبدأ

> **"منطق واحد في Service، عرض واحد في Shared.UI، ثلاثة أغلفة رقيقة."**

### 3.2 التوزيع حسب الطبقة

| # | الطبقة | المكوّن | الموقع |
|:-:|--------|--------|--------|
| 1 | **Domain** | `MenuVisibilityRule` (Entity) | `RubikCare.Domain/Entities/Navigation/MenuSystem/` |
| 2 | **Application** | `IMenuVisibilityService` | `RubikCare.Application/Interfaces/Navigation/` |
| 3 | **Application** | `IMenuFilteringService` | `RubikCare.Application/Interfaces/Navigation/` |
| 4 | **Application** | `IMenuFilterableItem` | `RubikCare.Application/Interfaces/Navigation/` |
| 5 | **Application** | `MenuFilterContextDto` | `RubikCare.Application/DTOs/Navigation/` |
| 6 | **Application** | `MenuFilteringService` (منطق خالص) | `RubikCare.Application/Services/Navigation/` |
| 7 | **Infrastructure** | `MenuVisibilityService` (DB) | `RubikCare.Infrastructure/Services/Navigation/` |
| 8 | **Web** | `DynamicMenuService` (يستخدم الخدمات) | `Rubikcare.Web/Data/Services/Navigation/` |
| 9 | **Web** | `UserMenuService` (Bridge) | `Rubikcare.Web/Data/Services/` |

### 3.3 `IMenuVisibilityService` — المسؤوليات

```csharp
public interface IMenuVisibilityService
{
    Task<List<MenuVisibilityRule>> GetActiveRulesForItemsAsync(
        List<int> itemIds,
        CancellationToken cancellationToken = default);

    Task<List<MenuVisibilityRule>> GetAllActiveRulesAsync(
        CancellationToken cancellationToken = default);
}
```

**المسؤولية:** قراءة القواعد من DB.

### 3.4 `IMenuFilteringService` — المسؤوليات

```csharp
public interface IMenuFilteringService
{
    List<T> ApplyRules<T>(
        List<T> items,
        List<MenuVisibilityRule> rules,
        MenuFilterContextDto context)
        where T : IMenuFilterableItem;
}
```

**المسؤولية:** تطبيق القواعد على قائمة عناصر — **منطق خالص (لا DB).**

### 3.5 `MenuFilterContextDto` — السياق

```csharp
public class MenuFilterContextDto
{
    public int? OrganizationId { get; set; }
    public int? OrganizationTypeId { get; set; }
    public int? SubscriptionPlanId { get; set; }
    public int? UserProfileId { get; set; }
    public List<string> RoleIds { get; set; } = new();
    public bool IsPlatformAdmin { get; set; }
}
```

### 3.6 تدفق البيانات في `DynamicMenuService`

```
LoadMenuDataFromDatabaseAsync(user)
    │
    ├── GetPersonalMenuAsync(userProfileId)
    │   ├── جلب Items من SystemMenu (USER_BASE)
    │   ├── IMenuVisibilityService.GetActiveRulesForItemsAsync(itemIds)
    │   └── IMenuFilteringService.ApplyRules(items, rules, context)
    │
    ├── GetAdminMenuAsync(user)
    │   ├── جلب Items من SystemMenu (ADMIN_MENU)
    │   ├── جلب Rules
    │   └── ApplyRules
    │
    └── LoadUserOrganizationsAsync(user)
        └── لكل Org:
            └── GetOrganizationMenuFromDatabaseAsync(orgId, orgTypeId, userProfileId)
                ├── جلب MenuAssignments + Items
                ├── جلب Rules
                └── ApplyRules
```

---

## 🎨 الجزء الرابع: الأنماط المُتبعة (Best Practices)

### 4.1 نمط DTOs: مصدر واحد للحقيقة

**⚠️ قاعدة صارمة:** كل DTO يُعرَّف **مرة واحدة فقط** في `Application.DTOs`.

**ممنوع:**
- ❌ DTO في `Web.Data.DTOs` + DTO في `Application.DTOs.Navigation`.
- ❌ استخدام Aliases (`@using X = Y`) — إلا في حالات نادرة.

**الحل الصحيح:**
- ✅ `Application.DTOs.Navigation.MenuItemDto` — للاستخدام المُشترك.
- ✅ لا `@using RubikCare.Shared.UI.Components.PSP.Points` في `_Imports.razor`.
- ✅ إضافة `@using` محلية في الملفات التي تحتاجها.

### 4.2 نمط Rule-Based Filtering

**المبدأ:**
1. **جلب العناصر** من DB (بدون فلترة).
2. **جلب القواعد** من `MenuVisibilityRules`.
3. **تطبيق القواعد** في `MenuFilteringService` (منطق خالص).
4. **الإرجاع** للـ Caller.

**المزايا:**
- **قابلية اختبار** عالية (منطق منفصل).
- **إعادة استخدام** في MAUI/PWA.
- **فصل تام** بين DB و Business Logic.

### 4.3 نمط Cache Invalidation

**⚠️ مهم:** عند تعديل `SystemMenus`, `MenuItems`, `MenuAssignments`, أو `MenuVisibilityRules`:

**يجب استدعاء:**
```csharp
await _menuService.ClearCacheAsync(userId);
```

**⚠️ مكان الاستدعاء:** `AdminMenuManager.razor` — بعد كل حفظ.

**⚠️ السبب:** لضمان أن القائمة تُحدَّث فوراً.

### 4.4 نمط Priority في Rules

**⚠️ تذكير:** **`Priority` الأصغر = الأعلى.**

**الترتيب في الاستعلام:**
```csharp
.OrderBy(r => r.Priority)  // ✅ الأصغر أولاً
```

---

## 🗺️ الجزء الخامس: خارطة التنفيذ

### 5.1 ما تم إنجازه ✅

| # | المكوّن | الحالة |
|:-:|--------|:------:|
| 1 | `MenuVisibilityRule` Entity | ✅ |
| 2 | `RuleType` Enum | ✅ |
| 3 | `IMenuVisibilityService` + Impl | ✅ |
| 4 | `IMenuFilteringService` + Impl | ✅ |
| 5 | `IMenuFilterableItem` | ✅ |
| 6 | `MenuFilterContextDto` | ✅ |
| 7 | `DynamicMenuService` المُحدَّث | ✅ |
| 8 | `MenuRepository` — `using` صحيح | ✅ |
| 9 | `LOYALTY_MENU` + 5 Items (SQL) | ✅ |
| 10 | توحيد DTOs على `Application.DTOs.Navigation` | ✅ |

### 5.2 ما تبقّى ⏳

| # | المكوّن | الأولوية |
|:-:|--------|:--------:|
| 1 | `LoyaltySchemesList.razor` | 🔴 |
| 2 | `LoyaltySchemeDetails.razor` | 🔴 |
| 3 | `LoyaltySchemeEdit.razor` | 🔴 |
| 4 | `GetSchemesByOrganizationHandler` + Endpoint | 🔴 |
| 5 | `MenuVisibilityRules` Seed للمنظمات | 🟠 |
| 6 | `LoyaltyProgram.razor` (إن وُجد) | 🟡 |
| 7 | واجهة إدارة `MenuVisibilityRules` | 🟠 |
| 8 | توسعة MAUI/PWA للقوائم الديناميكية | 🟠 |

### 5.3 المخاطر

| # | المخاطرة | الاحتمال | التخفيف |
|:-:|----------|:--------:|---------|
| 1 | تعارض DTOs (`Shared.UI` vs `Application`) | متوسط | **حذف `@using Shared.UI.Components.PSP.Points` من `_Imports.razor`** |
| 2 | Cache قديم | منخفض | `ClearCacheAsync` بعد كل تعديل |
| 3 | Priority مقلوب | متوسط | توثيق واضح + اختبار |
| 4 | Rules متناقضة | متوسط | ترتيب `Priority` يضمن الحسم |

---

## 📚 الجزء السادس: مراجع سريعة

### 6.1 قاموس المصطلحات

| المصطلح | المعنى |
|---------|--------|
| `MenuAssignment` | ربط `SystemMenu` بـ `OrganizationType` |
| `MenuVisibilityRule` | قاعدة إظهار/إخفاء عنصر |
| `RuleType` | نوع القاعدة (All, Organization, ...) |
| `Priority` | أولوية القاعدة — **الأصغر = الأعلى** |
| `IMenuFilterableItem` | واجهة موحدة لكل عنصر قابل للتصفية |
| `MenuFilterContextDto` | سياق المستخدم للتصفية |

### 6.2 حل المشاكل الشائعة

| # | المشكلة | الحل |
|:-:|---------|------|
| 1 | **`CS0104` Ambiguous reference** | احذف `@using RubikCare.Shared.UI.Components.PSP.Points` من `_Imports.razor` |
| 2 | **`CS0246` Type not found** | تحقق من `@using` في الملف |
| 3 | **القائمة لا تظهر** | تحقق من `MenuAssignment` + `IsActive` |
| 4 | **عنصر لا يظهر** | تحقق من `MenuVisibilityRule` + `Priority` |
| 5 | **Cache قديم** | `ClearCacheAsync(userId)` |

### 6.3 روابط ذات صلة

- [00 - الهيكل المعماري](./00-architecture-overview.md)
- [02 - نظام الهوية والمصادقة](./02-identity-system.md)
- [08 - Strategic Roadmap](./08-Strategic-Roadmap.md)
- [11 - دليل BlazorWebView](./11-blazor-webview-guide.md)
- [14 - نظام الكاش الموحد](./14-caching-system.md)
- [26 - نظام النقاط وروبيكا](./26-rubika-points-system.md)

---

## 📝 الجزء السابع: سجل التغييرات

| الإصدار | التاريخ | التغييرات |
|---------|---------|-----------|
| 1.0 | 6 أكتوبر 2026 | الإصدار الأولي |
| 2.0 | 10 أكتوبر 2026 | إضافة نص المؤلف الحرفي |
| **3.0** | **13 أكتوبر 2026** | **دمج مع `04-dynamic-menus` + `MenuVisibilityRules` + `IMenuFilteringService` + خارطة تنفيذ شاملة** |

---

**© 2026 RubikCare — للاستخدام الداخلي**
```


## 🔗 الجزء 7: روابط ذات صلة

- [00 - الهيكل المعماري](00-architecture-overview.md)
- [02 - نظام الهوية والمصادقة](02-identity-system.md)
- [04 - نظام القوائم الديناميكية](04-dynamic-menus.md)
- [05 - إنشاء الصفحات والمكونات](05-page-creation-checklist.md)
- [10 - دليل تطوير MAUI](10-maui-development-guide.md)
- [11 - دليل BlazorWebView](11-blazor-webview-guide.md)
- [14 - نظام الكاش الموحد](14-caching-system.md)
- [17 - استقرار اتجاه القائمة الجانبية واللغة](17-sidebar-direction-stability.md)

---

## 📝 الجزء 8: سجل التغييرات

| الإصدار | التاريخ | التغييرات |
|---------|---------|------------|
| 1.0 | 6 أكتوبر 2026 | الإصدار الأولي |

---

**© 2026 RubikCare — للاستخدام الداخلي**

---
