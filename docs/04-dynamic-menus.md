# 04 - نظام القوائم الديناميكية (Dynamic Menu System)

**آخر تحديث: 17 مايو 2026**

---

## مقدمة

نظام القوائم الديناميكية هو **الوجهة التي يراها المستخدم** عند دخوله إلى المنصة. واجهته ليست مجرد قائمة ثابتة، بل نظام ذكي يتكيف مع:
- من هو المستخدم؟ (UserProfile)
- ماذا يفعل؟ (الأدوار النظامية)
- أين يعمل؟ (العضويات في المؤسسات)

هذا المرجع يشرح كيفية بناء هذا النظام، وكيف نضمن أنه سريع، آمن، وسهل الصيانة.

---

## فلسفة النظام: "التطوير التراكمي الآمن"

### المبدأ الأساسي

في RubikCare، لا نؤمن بـ **"التغيير الجذري المفاجئ"**. بدلاً من ذلك، نتبع منهجية **"التطوير التراكمي الآمن"**:

```mermaid
graph TD
    A[1. فهم النظام الحالي] --> B[2. إضافة الجديد بجانب القديم]
    B --> C[3. تحويل المستخدمين تدريجياً]
    C --> D[4. جمع الملاحظات والتحسين]
    D --> E[5. إزالة القديم بعد الاستقرار]
```

**النتيجة:**
- **لا توقف في الخدمة** أبداً
- **اكتشاف المشاكل مبكراً** على نطاق محدود
- **قابلية التراجع** الفوري إذا ظهرت مشكلة

---

## الهيكل المعماري لجداول القوائم

### مخطط العلاقات

```mermaid
erDiagram
    SystemMenus ||--o{ MenuItems : "One-to-Many"
    SystemMenus ||--o{ MenuAssignments : "One-to-Many"
    MenuItems ||--o{ MenuItems : "Parent-Child (Self-Referencing)"
    MenuAssignments }o--|| UserProfiles : "Many-to-One (Nullable)"
    MenuAssignments }o--|| AspNetRoles : "Many-to-One (Nullable)"
    MenuAssignments }o--|| Organizations : "Many-to-One (Nullable)"
```

### SystemMenu - القوائم الرئيسية

```sql
-- جدول القوائم الرئيسية (مثل: USER_BASE, PHARMA_PSP, ADMIN_MENU)
CREATE TABLE SystemMenus (
    MenuID INT PRIMARY KEY IDENTITY(1,1),
    MenuCode NVARCHAR(50) NOT NULL,           -- USER_BASE, PHARMA_PSP, ADMIN_MENU
    MenuNameAr NVARCHAR(100) NOT NULL,         -- القائمة الشخصية، برامج دعم المرضى
    MenuNameEn NVARCHAR(100) NULL,             -- Personal Menu, PSP Programs
    MenuType NVARCHAR(20) NOT NULL,             -- BASE, ROLE_BASED, ORG_BASED
    SortOrder INT NOT NULL DEFAULT 100,
    Description NVARCHAR(500) NULL,
    IsActive BIT NOT NULL DEFAULT 1,
    CreatedDate DATETIME2 NOT NULL DEFAULT GETDATE()
);
```

**البيانات الحالية في النظام:**

| MenuID | MenuCode | MenuNameAr | MenuType |
|--------|----------|------------|----------|
| 6 | USER_BASE | القائمة الشخصية | BASE |
| 7 | PHARMA_PSP | برامج دعم المرضى | ROLE_BASED |
| 12 | ADMIN_MENU | لوحة تحكم الإدارة | ROLE_BASED |

### MenuItem - عناصر القوائم (مع التسلسل الهرمي)

```sql
-- جدول عناصر القوائم (يدعم Parent-Child للتسلسل الهرمي)
CREATE TABLE MenuItems (
    ItemID INT PRIMARY KEY IDENTITY(1,1),
    MenuID INT NOT NULL FOREIGN KEY REFERENCES SystemMenus(MenuID),
    ParentItemID INT NULL FOREIGN KEY REFERENCES MenuItems(ItemID),  -- ⭐ للتسلسل الهرمي
    ItemNameAr NVARCHAR(100) NOT NULL,
    ItemNameEn NVARCHAR(100) NULL,
    Url NVARCHAR(200) NOT NULL,
    IconClass NVARCHAR(50) NULL,              -- أيقونات Bootstrap: bi-house, bi-gear
    Description NVARCHAR(500) NULL,
    SortOrder INT NOT NULL DEFAULT 100,
    RequiredPermission NVARCHAR(50) NULL,      -- صلاحية مطلوبة (اختياري)
    IsActive BIT NOT NULL DEFAULT 1,
    IsExternalLink BIT NOT NULL DEFAULT 0,
    CreatedDate DATETIME2 NOT NULL DEFAULT GETDATE()
);
```

**مثال من البيانات الحالية:**

| ItemID | MenuID | ItemNameAr | Url | IconClass | SortOrder |
|--------|--------|------------|-----|-----------|-----------|
| 17 | 12 | لوحة التحكم المركزية | /admin/dashboard | bi-speedometer2 | 1 |
| 19 | 12 | إعدادات طبية | /admin/medical | bi-heart-pulse | 2 |
| 20 | 6 | ملفي الشخصي | /profile | bi-person | 1 |

### MenuAssignment - نظام التخصيص الذكي (⭐ القلب الحقيقي)

```sql
-- جدول التخصيصات: يحدد من يرى أي قائمة
CREATE TABLE MenuAssignments (
    AssignmentID INT PRIMARY KEY IDENTITY(1,1),
    MenuID INT NOT NULL FOREIGN KEY REFERENCES SystemMenus(MenuID),
    
    -- ⭐ أربع طرق للتخصيص (واحد منها على الأقل مطلوب)
    UserProfileID INT NULL FOREIGN KEY REFERENCES UserProfiles(UserProfileID),
    RoleId NVARCHAR(450) NULL FOREIGN KEY REFERENCES AspNetRoles(Id),
    OrganizationID INT NULL FOREIGN KEY REFERENCES Organizations(OrganizationID),
    OrganizationTypeID INT NULL FOREIGN KEY REFERENCES OrganizationTypes(OrganizationTypeID),
    
    AssignmentType NVARCHAR(20) NOT NULL,      -- USER_SPECIFIC, ROLE_BASED, ORGANIZATION_BASED
    AssignedDate DATETIME2 NOT NULL DEFAULT GETDATE(),
    AssignedBy INT NULL FOREIGN KEY REFERENCES UserProfiles(UserProfileID),
    ExpiryDate DATETIME2 NULL,
    IsActive BIT NOT NULL DEFAULT 1,
    
    -- ⭐ فهرس لتحسين أداء البحث
    INDEX IX_MenuAssignments_UserProfile (UserProfileID, IsActive),
    INDEX IX_MenuAssignments_Role (RoleId, IsActive),
    INDEX IX_MenuAssignments_Organization (OrganizationID, IsActive)
);
```

**مثال حي من النظام:**

| AssignmentID | MenuID | UserProfileID | RoleId | OrganizationID | OrganizationTypeID |
|--------------|--------|---------------|--------|----------------|--------------------|
| 6 | 12 | NULL | Admin | NULL | NULL |
| 7 | 7 | 12 | NULL | NULL | NULL |
| 8 | 7 | NULL | NULL | 5 | NULL |

---

## 🔑 الاكتشاف الحاسم: فصل القوائم الثلاثة

**المشكلة:** كيف نفصل بين القائمة الشخصية، قوائم المؤسسات، وقائمة الإدارة؟

**الحل:** جعل `OrganizationID` في `MenuAssignment` **قابلاً للـ Null**:

```csharp
public class MenuAssignment
{
    // ...
    public int? OrganizationID { get; set; }  // ⭐ من int إلى int?
    public virtual Organization? Organization { get; set; }
}
```

**النتيجة:**
- **القائمة الشخصية (USER_BASE):** `OrganizationID = NULL` ← تظهر للجميع
- **قائمة المؤسسات:** مرتبطة بـ `OrgMemberships.IsActive` للمستخدم
- **قائمة الإدارة (ADMIN_MENU):** `OrganizationID = 1` (مؤسسة روبيك كير) ← تظهر فقط لمن في هذه المؤسسة

---

## DynamicMenuService - قلب النظام

### لماذا هذه الخدمة؟

بدون `DynamicMenuService`، كنا سنضطر لكتابة استعلامات معقدة في كل صفحة لجلب القوائم المناسبة لكل مستخدم. هذه الخدمة:
- **توحد** منطق جلب القوائم في مكان واحد
- **تحسن الأداء** باستخدام الـ Caching
- **تضمن الأمان** بعدم كشف قوائم لا تخص المستخدم

### الكود الكامل

```csharp
using Microsoft.EntityFrameworkCore;
using Microsoft.Extensions.Caching.Memory;
using Rubikcare.Web.Data.Models;

namespace Rubikcare.Web.Data.Services
{
    public class DynamicMenuService
    {
        private readonly IDbContextFactory<BusinessDbContext> _dbContextFactory;
        private readonly UserContextService _userContext;
        private readonly IMemoryCache _cache;
        private readonly ILogger<DynamicMenuService> _logger;

        public DynamicMenuService(
            IDbContextFactory<BusinessDbContext> dbContextFactory,
            UserContextService userContext,
            IMemoryCache cache,
            ILogger<DynamicMenuService> logger)
        {
            _dbContextFactory = dbContextFactory;
            _userContext = userContext;
            _cache = cache;
            _logger = logger;
        }

        /// <summary>
        /// ⭐ الوظيفة الرئيسية: جلب القوائم المناسبة للمستخدم الحالي
        /// </summary>
        public async Task<List<MenuItem>> GetUserMenuAsync(ClaimsPrincipal user)
        {
            var cacheKey = $"UserMenu_{user.Identity?.Name}";
            
            // ⭐ محاولة الاسترجاع من الكاش أولاً (أداء أفضل)
            if (_cache.TryGetValue(cacheKey, out List<MenuItem> cachedMenu))
            {
                _logger.LogDebug("Returning menu from cache for user: {User}", user.Identity?.Name);
                return cachedMenu;
            }

            await using var context = await _dbContextFactory.CreateDbContextAsync();
            
            try
            {
                // 1. الحصول على معلومات المستخدم
                var userProfileId = await _userContext.GetCurrentUserProfileIdAsync(user);
                var roles = await _userContext.GetUserRolesAsync(user);
                var organizationIds = await _userContext.GetUserOrganizationIdsAsync(user);

                // 2. البحث في MenuAssignments بجميع المعايير الأربعة
                var assignments = await context.MenuAssignments
                    .Include(ma => ma.SystemMenu)
                        .ThenInclude(m => m.MenuItems)
                    .Where(ma => ma.IsActive && 
                        (ma.ExpiryDate == null || ma.ExpiryDate > DateTime.UtcNow) &&
                        (
                            (ma.UserProfileID == userProfileId) ||
                            (ma.RoleId != null && roles.Contains(ma.RoleId)) ||
                            (ma.OrganizationID != null && organizationIds.Contains(ma.OrganizationID.Value)) ||
                            (ma.OrganizationTypeID != null && 
                             await _userContext.HasOrganizationTypeAsync(user, ma.OrganizationTypeID.Value))
                        ))
                    .ToListAsync();

                // 3. تجميع عناصر القوائم (مع إزالة التكرار)
                var menuItems = assignments
                    .SelectMany(a => a.SystemMenu.MenuItems.Where(mi => mi.IsActive))
                    .OrderBy(mi => mi.SortOrder)
                    .DistinctBy(mi => mi.ItemID)
                    .ToList();

                // 4. تخزين في الكاش (لمدة 5 دقائق)
                _cache.Set(cacheKey, menuItems, TimeSpan.FromMinutes(5));
                
                _logger.LogInformation("Menu loaded for user {User}: {Count} items", 
                    user.Identity?.Name, menuItems.Count);
                    
                return menuItems;
            }
            catch (Exception ex)
            {
                _logger.LogError(ex, "Error loading dynamic menu for user {User}", 
                    user.Identity?.Name);
                throw;
            }
        }

        /// <summary>
        /// ⭐ جلب قوائم مؤسسة محددة (عند التبديل بين التبويبات)
        /// </summary>
        public async Task<List<MenuItem>> GetOrganizationMenuAsync(int organizationId)
        {
            await using var context = await _dbContextFactory.CreateDbContextAsync();
            
            return await context.MenuItems
                .Include(mi => mi.SystemMenu)
                .Where(mi => mi.IsActive && 
                       mi.SystemMenu.MenuAssignments.Any(ma => 
                           ma.IsActive && 
                           ma.OrganizationID == organizationId))
                .OrderBy(mi => mi.SortOrder)
                .ToListAsync();
        }

        /// <summary>
        /// ⭐ جلب جميع المؤسسات التي للمستخدم فيها قوائم
        /// </summary>
        public async Task<List<Organization>> GetUserOrganizationsWithMenusAsync(int userProfileId)
        {
            await using var context = await _dbContextFactory.CreateDbContextAsync();
            
            var userOrgs = await context.OrgMemberships
                .Where(om => om.UserProfileID == userProfileId && 
                       om.IsActive && 
                       (om.EndDate == null || om.EndDate > DateTime.UtcNow))
                .Select(om => om.Organization)
                .ToListAsync();

            var orgsWithMenus = userOrgs
                .Where(o => context.MenuAssignments.Any(ma => 
                    ma.IsActive && ma.OrganizationID == o.OrganizationID))
                .ToList();

            return orgsWithMenus;
        }
    }
}
```

### تسجيل الخدمة في Program.cs

```csharp
builder.Services.AddScoped<DynamicMenuService>();
builder.Services.AddMemoryCache();  // ⭐ ضروري للـ Caching
```

---

## كيف يعمل فصل القوائم الثلاثة؟ (الابتكار الرئيسي)

### القاعدة الذهبية

```sql
-- في جدول MenuAssignments، القاعدة هي:
-- OrganizationID = NULL    ← قائمة عامة (شخصية)
-- OrganizationID = 1       ← قائمة إدارة (لمؤسسة روبيك كير)
-- OrganizationID = 5       ← قائمة مؤسسة (لشركة ديفارت لاب)
```

### مثال حي: المستخدم 12 (shadyelzaher@devartlab.com)

```sql
-- هذا المستخدم له:
-- الدور: Admin
-- العضويات: 
--   - مالك في "شركة ديفارت لاب" (OrganizationID: 5)
--   - عضو في "منصة روبيك كير" (OrganizationID: 1)
```

**النتيجة عند تسجيل الدخول:**

```mermaid
graph LR
    A[تسجيل الدخول] --> B[التبويب 1: القائمة الشخصية]
    A --> C[التبويب 2: إدارة منصة RubikCare]
    A --> D[التبويب 3: شركة ديفارت لاب للادوية]
```

---

## InteractiveMenu.razor - واجهة المستخدم

### تدفق المكون

```mermaid
sequenceDiagram
    actor User
    participant IM as InteractiveMenu
    participant DMS as DynamicMenuService
    participant UC as UserContextService

    User->>IM: فتح الصفحة
    IM->>UC: GetCurrentUserAsync
    UC-->>IM: User + UserProfile
    IM->>DMS: GetUserMenuAsync
    DMS-->>IM: كل القوائم
    IM->>IM: فصل القوائم حسب المصدر
    IM-->>User: عرض التبويبات
    User->>IM: تبديل التبويب
    IM->>DMS: GetOrganizationMenuAsync
    DMS-->>IM: قائمة المؤسسة
    IM-->>User: عرض القائمة الجديدة
```

---

## استعلامات مفيدة للتحقق والمراقبة

```sql
-- 1. عرض جميع القوائم مع أنواعها
SELECT MenuCode, MenuNameAr, MenuType, IsActive
FROM SystemMenus
ORDER BY SortOrder;

-- 2. عرض عناصر قائمة محددة (مثال: ADMIN_MENU)
SELECT mi.ItemNameAr, mi.Url, mi.SortOrder
FROM MenuItems mi
INNER JOIN SystemMenus sm ON mi.MenuID = sm.MenuID
WHERE sm.MenuCode = 'ADMIN_MENU' AND mi.IsActive = 1
ORDER BY mi.SortOrder;

-- 3. معرفة من يرى أي قائمة
SELECT 
    sm.MenuNameAr AS القائمة,
    CASE 
        WHEN ma.UserProfileID IS NOT NULL THEN 'مستخدم محدد'
        WHEN ma.RoleId IS NOT NULL THEN 'دور: ' + r.Name
        WHEN ma.OrganizationID IS NOT NULL THEN 'منظمة: ' + o.OrganizationNameAr
        WHEN ma.OrganizationTypeID IS NOT NULL THEN 'نوع منظمة'
        ELSE 'عامة'
    END AS مخصصة_لـ,
    ma.IsActive,
    ma.ExpiryDate
FROM MenuAssignments ma
INNER JOIN SystemMenus sm ON ma.MenuID = sm.MenuID
LEFT JOIN AspNetRoles r ON ma.RoleId = r.Id
LEFT JOIN Organizations o ON ma.OrganizationID = o.OrganizationID
WHERE ma.IsActive = 1;

-- 4. التحقق من عدم وجود تكرار في التخصيصات
SELECT UserProfileID, RoleId, OrganizationID, COUNT(*) as DuplicateCount
FROM MenuAssignments
WHERE IsActive = 1
GROUP BY UserProfileID, RoleId, OrganizationID
HAVING COUNT(*) > 1;
```

---

## المحاذير والأخطاء الشائعة

### 🔴 ممنوعات مطلقة

1. **لا تستخدم `DbContext` مباشرة في `DynamicMenuService`.** استخدم `DbContextFactory`.
2. **لا تهمل الـ Caching.** بدونها، كل طلب صفحة سيؤدي لاستعلامات قاعدة بيانات متعددة.
3. **لا تضع `OrganizationID = 0` للقوائم العامة.** استخدم `NULL` كما هو موثق.
4. **لا تكرر نفس القائمة لنفس المستخدم بطرق متعددة.**

### 🟡 أخطاء شائعة وحلولها

| المشكلة | السبب | الحل |
|----------|-------|-------|
| القائمة لا تظهر لبعض المستخدمين | `MenuAssignment` غير مضبوط | تحقق من `UserProfileID`, `RoleId`, `OrganizationID` |
| القائمة تظهر لمن لا يجب أن تظهر له | عدم تحديد `OrganizationID` بشكل صحيح | تأكد من أن `OrganizationID = NULL` للقوائم العامة فقط |
| أداء بطيء في تحميل القوائم | عدم استخدام الـ Caching | أضف `IMemoryCache` واستخدمه في `GetUserMenuAsync` |
| زر الرجوع لا يعمل | مشكلة في `StateHasChanged()` | تأكد من استدعاء `StateHasChanged()` بعد تغيير `_activeTab` |

---

## CHECKLIST: عند إضافة قائمة جديدة

### الخطوة 1: إضافة القائمة الرئيسية
- [ ] هل القائمة جديدة كلياً؟ أضف سجلاً في `SystemMenus` مع `MenuCode` فريد
- [ ] هل تحددت `MenuType` المناسب (BASE, ROLE_BASED, ORGANIZATION_BASED)؟

### الخطوة 2: إضافة عناصر القائمة
- [ ] هل أضفت كل عنصر في `MenuItems` مع `MenuID` الصحيح؟
- [ ] هل رتبت العناصر باستخدام `SortOrder`؟
- [ ] هل اخترت أيقونة مناسبة من Bootstrap Icons؟

### الخطوة 3: تخصيص الظهور (MenuAssignment)
- [ ] لمن تظهر هذه القائمة؟
    - [ ] للجميع؟ → `OrganizationID = NULL`
    - [ ] لدور محدد؟ → `RoleId` محدد
    - [ ] لمستخدم محدد؟ → `UserProfileID` محدد
    - [ ] لمؤسسة محددة؟ → `OrganizationID` محدد
- [ ] هل للقائمة صلاحية زمنية؟ اضبط `ExpiryDate`

### الخطوة 4: الاختبار
- [ ] اختبر مع مستخدمين مختلفين (عادي، Admin، عضو في المؤسسة)
- [ ] تأكد من أن القائمة تظهر فقط لمن يفترض أن تظهر له
- [ ] تأكد من أن الروابط تعمل بشكل صحيح

---

## 🔗 روابط ذات صلة

- [00 - الهيكل المعماري](00-architecture-overview.md)
- [01 - Program.cs والتسجيلات الأساسية](01-program-cs-foundation.md)
- [02 - نظام الهوية والمصادقة](02-identity-system.md)
- [05 - إنشاء الصفحات والمكونات](05-page-creation-checklist.md)
```

--
