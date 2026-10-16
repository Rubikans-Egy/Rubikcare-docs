# [#44] 📘 `19-PERMISSIONS` — Part 1 (الفلسفة + التصميم + نموذج البيانات)

**شكراً على التوضيح الكامل. الآن الصورة واضحة 100%. سأكتب الوثيقة بدقة.**

---

```markdown
# 19 — PERMISSIONS (نظام الصلاحيات الكامل)

**الإصدار:** 1.0
**آخر تحديث:** 16 أكتوبر 2026
**الحالة:** ✅ تصميم كامل — جاهز للتنفيذ
**الجمهور:** فريق التطوير + AI Assistants

---

## 📌 الجزء صفر: الفلسفة والرؤية (نص المؤلف الحرفي)

> **⚠️ ملاحظة:**
> هذا القسم يحتوي على **نص الكاتب الحرفي** — دون إعادة صياغة.
> الهدف: حفظ الرؤية الكاملة كما صيغت.

---

### 0.1 الرؤية الأساسية للنظام

> **نص المؤلف الحرفي:**
>
> "ما أريده أن يكون هناك جدول يتم فيه تسجيل مستويات التحكم داخل كل منظمة.
> ثم يتم ربط كل وظيفة مع مستوى معين.
> ويمكن فصل الوظيفة صاحبة الصلاحيات عن المسمى الوظيفي المخصص للمستخدم، بحيث يمكن أن يكون لنفس المسمى الوظيفي صلاحيات مختلفة.
> ونحن لدينا جدولان يسمحان بهذا فعلاً، ولكن نعيد توظيفهما بما يخدم ما أريده:
> - `CustomJobTitleId`
> - `MemberTitle`."

**الترجمة التقنية:**

| # | المفهوم | التنفيذ |
|:-:|:---|:---|
| 1 | **مستويات التحكم** | `ControlLevel` (كيان جديد) |
| 2 | **ربط الوظيفة بالمستوى** | `CustomJobTitle` ← `ControlLevel` |
| 3 | **فصل الوظيفة عن الصلاحيات** | `MemberTitle` = M:N بين `OrgMembership` و `CustomJobTitle` |
| 4 | **نفس المسمى، صلاحيات مختلفة** | `MemberTitle` يُتيح تخصيص الصلاحيات لكل عضو |

---

### 0.2 سبب الحاجة للنظام (مثال حي)

> **نص المؤلف الحرفي:**
>
> "مثلاً ممكن يكون لدى الشركة عدد من الأعضاء في قسم التسويق لهم نفس المسمى الوظيفي، ولكن في الصلاحيات يتم منح أحدهما صلاحية إضافية داخل منصة RubikCare كإعداد أو الإشراف على برامج الدعم وتعديل أسعار المنتجات وغير ذلك."

**الترجمة التقنية:**

**⚠️ المثال:**

```
قسم التسويق في شركة X:
├── عضو A: المسمى = "أخصائي تسويق"
│         الصلاحيات = [عرض التقارير]
│
└── عضو B: المسمى = "أخصائي تسويق"
          الصلاحيات = [عرض التقارير + إعداد برامج الدعم + تعديل أسعار المنتجات]
```

**⚠️ نفس المسمى — صلاحيات مختلفة.**

**⚠️ هذا يُحل بـ:**

- **`CustomJobTitle`** = المسمى (مشترك).
- **`MemberTitle`** = ربط العضو بالمسمى (M:N).
- **`ControlLevel`** = الصلاحيات المرفقة.
- **`MemberTitleOverride`** (جديد) = صلاحيات إضافية فردية.

---

### 0.3 فلسفة "نظام الصلاحيات المرن"

> **نص المؤلف الحرفي:**
>
> "لا أريد اعتماد هذه القيم (MembershipType — 8 قيم)، ولكن كما قلت لك، نريد تصميم نظام مرن احترافي، ويمكن إعادة تدوير واستخدام الجداول الزائدة الحالية لتخدم الغرض الفعلي لنظام الصلاحيات."

**الترجمة التقنية:**

**⚠️ المشكلة:** `OrgMembership.MembershipType` — 8 قيم ثابتة (`OWNER`, `ADMIN`, `MANAGER`, `DOCTOR`, `PHARMACIST`, `ASSISTANT`, `STAFF`, `CONTRACTOR`).

**⚠️ الحل:** `ControlLevel` — **مستويات ديناميكية** تُعرِّفها كل منظمة.

**⚠️ إعادة التدوير:**

| العنصر القديم | يُعاد توظيفه كـ |
|:---|:---|
| `OrgMembership.MembershipType` | → `Deprecated` (يُبقى للتوافق) |
| `OrgMembership.CustomJobTitleID` | → يبقى — للربط بالمسمى الأساسي |
| `MemberTitle` (غير مستخدم) | → **يُفعَّل** — M:N بين `OrgMembership` و `CustomJobTitle` |
| `UserRole.RoleType` | → يبقى — `Professional` للأطباء/الصيادلة فقط |

---

### 0.4 فلسفة "العضوية المتعددة"

> **نص المؤلف الحرفي:**
>
> "النظام يسمح بأن يكون للمستخدم عدة عضويات في عدة منظمات.
> المسموح له بإنشاء عيادة: الطبيب فقط.
> المسموح له بإنشاء صيدلية: الصيدلي فقط، بعد التحقق من تراخيصهم عبر المنصة.
> يمكن للطبيب أن يكون عضواً عاملاً في صيدلية، والعكس: يمكن لصيدلي أن يكون عضواً في عيادة ولكن لا ينشئها.
> يمكن أن يكون مندوب دعاية أو تسويق في شركة أدوية ولديه صيدلية أو عيادة أو عضو فيها."

**الترجمة التقنية:**

**⚠️ نموذج `User` — `OrgMembership` = M:N:**

```
User (طبيب)
├── OrgMembership 1: Clinic #5 (بصفة OWNER)
├── OrgMembership 2: Pharmacy #10 (بصفة PHARMACIST عضو عامل)
└── OrgMembership 3: Pharma Company #3 (بصفة Rep)

User (صيدلي)
├── OrgMembership 1: Pharmacy #7 (بصفة OWNER)
└── OrgMembership 2: Clinic #12 (بصفة Staff)
```

**⚠️ القيود:**

| # | القاعدة |
|:-:|:---|
| 1 | **إنشاء عيادة** — Doctor فقط (بعد التحقق). |
| 2 | **إنشاء صيدلية** — Pharmacist فقط (بعد التحقق). |
| 3 | **العضوية** — أي دور في أي منظمة. |
| 4 | **التمييز** — عبر `OrgMembership` (كل مؤسسات المستخدم). |

---

### 0.5 طبقات النظام (3 طبقات)

```
1. طبقة المستخدم (User)
   └── التمييز المهني عبر UserRole (Doctor/Pharmacist)

2. طبقة العضوية (OrgMembership)
   └── كل مؤسسات المستخدم

3. طبقة الصلاحيات داخل المؤسسة (ControlLevel + MemberTitle)
   └── ما يفعله المستخدم داخل المؤسسة
```

**⚠️ الترتيب:**

```
1. المستخدم يدخل → النظام يُحدد أدواره المهنية (UserRole)
   ↓
2. المستخدم يختار منظمة → النظام يعرض OrgMemberships
   ↓
3. داخل المنظمة → النظام يقرأ ControlLevel + MemberTitle
   ↓
4. النتيجة: قائمة الصلاحيات الفعلية
```

---

## 📌 الجزء الأول: البنية المعمارية

### 1.1 المبدأ الأساسي

> **"لا نبني نظامًا جديدًا، بل نُوسّع ونُوحّد الموجود."**

**⚠️ مع إعادة تدوير:**

- `CustomJobTitle` — موجود → يُوسَّع.
- `MemberTitle` — موجود → يُفعَّل.
- `OrgMembership` — موجود → يُنظَّف.
- `UserRole` — موجود → يبقى للمهنة.

### 1.2 الطبقات الثلاث للتحكم

```
┌─────────────────────────────────────────────────────────┐
│  الطبقة 1: المستخدم (User)                               │
│  من يتحكم؟ Super Admin                                   │
│  ماذا؟ الأدوار المهنية (UserRole)                        │
│  المعيار: RoleType = Professional                        │
└─────────────────────────────────────────────────────────┘
                          ↓
┌─────────────────────────────────────────────────────────┐
│  الطبقة 2: العضوية (Organization Membership)             │
│  من يتحكم؟ Super Admin + Organization Admin              │
│  ماذا؟ كل مؤسسات المستخدم                                │
│  المعيار: OrgMembership (M:N بين User و Organization)   │
└─────────────────────────────────────────────────────────┘
                          ↓
┌─────────────────────────────────────────────────────────┐
│  الطبقة 3: الصلاحيات (Control Level + MemberTitle)      │
│  من يتحكم؟ Organization Admin                            │
│  ماذا؟ ما يفعله المستخدم داخل المؤسسة                    │
│  المعيار: ControlLevel + MemberTitle                     │
└─────────────────────────────────────────────────────────┘
```

### 1.3 العلاقة بنظام القوائم (`03-MENUS`)

**⚠️ التكامل:**

| النظام | السؤال | المستوى |
|:---|:---|:---|
| **`03-MENUS`** | "هل يظهر العنصر؟" | UI (Presentation) |
| **`19-PERMISSIONS`** | "هل يُسمح بالفعل؟" | Domain + Application |

**⚠️ الفرق:**

- **`MenuVisibilityRule`** — يفلتر القوائم في UI.
- **`Permission`** — يمنع/يسمح في Backend.

**⚠️ التكامل:**

- **`MenuVisibilityRule.CustomJobTitleId`** — يُوسَّع.
- **`MenuVisibilityRule.ControlLevelId`** — يُضاف لاحقاً.
- **`Permission`** — يُستخدم في Middleware.

---

## 📌 الجزء الثاني: نموذج البيانات (ERD)

### 2.1 الكيانات الجديدة

#### 2.1.1 `ControlLevel` — مستوى التحكم

```csharp
[Table("ControlLevels")]
public class ControlLevel
{
    [Key]
    public int ControlLevelID { get; set; }

    // ⭐ من يُعرِّف؟
    [Required]
    public int OrganizationID { get; set; }         // المستوى خاص بمنظمة
    
    [Required]
    [MaxLength(100)]
    public string LevelNameAr { get; set; }          // مثال: "مدير قسم التسويق"

    [MaxLength(100)]
    public string? LevelNameEn { get; set; }

    [MaxLength(500)]
    public string? Description { get; set; }

    // ⭐ المستوى الافتراضي؟ (للمستخدمين الجدد)
    public bool IsDefault { get; set; } = false;

    // ⭐ ترتيب المستوى (للتصنيف)
    public int LevelOrder { get; set; } = 100;

    // ⭐ الحالة
    public bool IsActive { get; set; } = true;

    // ⭐ التواريخ
    public DateTime CreatedDate { get; set; } = DateTime.UtcNow;
    public int? CreatedByUserID { get; set; }
    public DateTime? LastModifiedDate { get; set; }

    // ⭐ العلاقات
    [ForeignKey("OrganizationID")]
    public virtual Organization Organization { get; set; } = null!;

    [ForeignKey("CreatedByUserID")]
    public virtual UserProfile? CreatedByUser { get; set; }

    // M:N
    public virtual ICollection<ControlLevelPermission> Permissions { get; set; } = new List<ControlLevelPermission>();
    public virtual ICollection<MemberTitle> MemberTitles { get; set; } = new List<MemberTitle>();
}
```

**⚠️ ملاحظة:** `OrganizationID` مطلوب — كل منظمة تُعرِّف مستوياتها.

---

#### 2.1.2 `Permission` — الصلاحية

```csharp
[Table("Permissions")]
public class Permission
{
    [Key]
    public int PermissionID { get; set; }

    // ⭐ الكود الفريد (يُستخدم في Middleware)
    [Required]
    [MaxLength(100)]
    public string PermissionCode { get; set; }       // مثال: "PSP.PROGRAM.CREATE"

    [Required]
    [MaxLength(200)]
    public string PermissionNameAr { get; set; }

    [MaxLength(200)]
    public string? PermissionNameEn { get; set; }

    [MaxLength(500)]
    public string? Description { get; set; }

    // ⭐ التصنيف (للتنظيم)
    [MaxLength(100)]
    public string? Category { get; set; }            // مثال: "PSP", "Loyalty", "Admin"

    // ⭐ هل هي صلاحية عامة أم خاصة بخدمة؟
    [MaxLength(50)]
    public string Scope { get; set; } = "PLATFORM";  // PLATFORM | PSP | LOYALTY | ...

    // ⭐ الحالة
    public bool IsActive { get; set; } = true;

    // ⭐ التواريخ
    public DateTime CreatedDate { get; set; } = DateTime.UtcNow;

    // M:N
    public virtual ICollection<ControlLevelPermission> ControlLevels { get; set; } = new List<ControlLevelPermission>();
}
```

**⚠️ أمثلة على `PermissionCode`:**

| الكود | المعنى | Scope |
|:---|:---|:---|
| `PSP.PROGRAM.VIEW` | عرض برامج الدعم | PSP |
| `PSP.PROGRAM.CREATE` | إنشاء برنامج | PSP |
| `PSP.PROGRAM.EDIT` | تعديل برنامج | PSP |
| `PSP.PROGRAM.DELETE` | حذف برنامج | PSP |
| `PSP.PROGRAM.APPROVE` | اعتماد برنامج | PSP |
| `PSP.MEDICATION.EDIT_PRICE` | تعديل أسعار الأدوية | PSP |
| `LOYALTY.SCHEME.VIEW` | عرض أنظمة النقاط | LOYALTY |
| `LOYALTY.SCHEME.CREATE` | إنشاء نظام نقاط | LOYALTY |
| `PHARMACY.DISPENSE.CONFIRM` | تأكيد الصرف | PHARMACY |
| `ADMIN.MEMBERS.ADD` | إضافة أعضاء | ADMIN |

---

#### 2.1.3 `ControlLevelPermission` — M:N

```csharp
[Table("ControlLevelPermissions")]
public class ControlLevelPermission
{
    [Key, Column(Order = 1)]
    public int ControlLevelID { get; set; }

    [Key, Column(Order = 2)]
    public int PermissionID { get; set; }

    // ⭐ هل الصلاحية مُمنوحة أم ممنوعة؟ (نادراً ما نستخدم ممنوعة)
    public bool IsGranted { get; set; } = true;

    // ⭐ التواريخ
    public DateTime CreatedDate { get; set; } = DateTime.UtcNow;

    // ⭐ العلاقات
    [ForeignKey("ControlLevelID")]
    public virtual ControlLevel ControlLevel { get; set; } = null!;

    [ForeignKey("PermissionID")]
    public virtual Permission Permission { get; set; } = null!;
}
```

---

#### 2.1.4 `MemberTitleOverride` — صلاحيات فردية إضافية

**⚠️ هذا الكيان الجديد يحل مشكلة "نفس المسمى، صلاحيات مختلفة".**

```csharp
[Table("MemberTitleOverrides")]
public class MemberTitleOverride
{
    [Key]
    public int MemberTitleOverrideID { get; set; }

    // ⭐ يربط بـ MemberTitle (العضو + المسمى)
    [Required]
    public int MemberTitleID { get; set; }

    // ⭐ الصلاحية الإضافية/الممنوعة
    [Required]
    public int PermissionID { get; set; }

    // ⭐ IsGranted = true (إضافة) أو false (منع)
    public bool IsGranted { get; set; } = true;

    // ⭐ السبب
    [MaxLength(500)]
    public string? Reason { get; set; }

    // ⭐ التواريخ
    public DateTime CreatedDate { get; set; } = DateTime.UtcNow;
    public int? CreatedByUserID { get; set; }

    // ⭐ العلاقات
    [ForeignKey("MemberTitleID")]
    public virtual MemberTitle MemberTitle { get; set; } = null!;

    [ForeignKey("PermissionID")]
    public virtual Permission Permission { get; set; } = null!;

    [ForeignKey("CreatedByUserID")]
    public virtual UserProfile? CreatedByUser { get; set; }
}
```

**⚠️ المثال الحي:**

```
قسم التسويق:
├── عضو A: المسمى = "أخصائي تسويق"
│         ControlLevel = "تسويق - أساسي"
│         الصلاحيات = [عرض التقارير]
│
└── عضو B: المسمى = "أخصائي تسويق"
          ControlLevel = "تسويق - أساسي"
          + MemberTitleOverride:
            ├── PSP.PROGRAM.CREATE (IsGranted = true)
            ├── PSP.PROGRAM.APPROVE (IsGranted = true)
            └── PSP.MEDICATION.EDIT_PRICE (IsGranted = true)
          الصلاحيات الفعلية = [عرض التقارير + إعداد PSP + تعديل الأسعار]
```

---

### 2.2 الكيانات المُعدَّلة

#### 2.2.1 `CustomJobTitle` — يُوسَّع

```csharp
[Table("CustomJobTitles")]
public class CustomJobTitle
{
    [Key]
    public int CustomJobTitleID { get; set; }

    [Required]
    public int OrganizationID { get; set; }

    [Required]
    [MaxLength(100)]
    public string TitleNameAr { get; set; } = string.Empty;

    [MaxLength(100)]
    public string? TitleNameEn { get; set; }

    [MaxLength(500)]
    public string? DescriptionAr { get; set; }

    // ⭐⭐ جديد: ControlLevel الافتراضي
    public int? DefaultControlLevelID { get; set; }

    public bool IsActive { get; set; } = true;
    public DateTime CreatedDate { get; set; } = DateTime.Now;
    public int? CreatedByUserID { get; set; }

    // ⭐ العلاقات
    [ForeignKey("OrganizationID")]
    public virtual Organization Organization { get; set; }

    [ForeignKey("DefaultControlLevelID")]
    public virtual ControlLevel? DefaultControlLevel { get; set; }

    [ForeignKey("CreatedByUserID")]
    public virtual UserProfile? CreatedByUser { get; set; }
}
```

**⚠️ التغييرات:**

- **`DefaultControlLevelID`** (جديد) — المستوى الافتراضي للمسمى.
- **يبقى كما هو** — لا كسر للتوافق.

---

#### 2.2.2 `MemberTitle` — يُفعَّل

```csharp
[Table("MemberTitles")]
public class MemberTitle
{
    [Key]
    public int MemberTitleID { get; set; }

    [Required]
    public int OrgMembershipID { get; set; }
    public virtual OrgMembership OrgMembership { get; set; } = null!;

    [Required]
    public int CustomJobTitleID { get; set; }
    public virtual CustomJobTitle CustomJobTitle { get; set; } = null!;

    // ⭐⭐ جديد: ControlLevel الفعلي للعضو
    public int? ControlLevelID { get; set; }

    // ⭐⭐ جديد: صفة العضوية في هذا المسمى
    // OWNER, MANAGER, MEMBER, TRAINEE
    [MaxLength(50)]
    public string? MemberRole { get; set; }

    [MaxLength(500)]
    public string? Description { get; set; }

    [Required]
    public bool IsActive { get; set; } = true;

    public DateTime? EffectiveFrom { get; set; }
    public DateTime? EffectiveTo { get; set; }

    [Required]
    public DateTime CreatedDate { get; set; } = DateTime.UtcNow;
    public DateTime? LastModifiedDate { get; set; }

    public int? CreatedByUserID { get; set; }
    public virtual UserProfile? CreatedByUser { get; set; }

    // ⭐ العلاقات الجديدة
    [ForeignKey("ControlLevelID")]
    public virtual ControlLevel? ControlLevel { get; set; }

    // M:N — Overrides
    public virtual ICollection<MemberTitleOverride> Overrides { get; set; } = new List<MemberTitleOverride>();
}
```

**⚠️ التغييرات:**

- **`ControlLevelID`** (جديد) — المستوى الفعلي للعضو.
- **`MemberRole`** (جديد) — الصفة داخل المسمى (`OWNER`, `MANAGER`, `MEMBER`).
- **`Overrides`** (جديد) — صلاحيات فردية.

---

#### 2.2.3 `OrgMembership` — تنظيف

```csharp
public class OrgMembership
{
    // ⚠️ لا تغيير في المفاتيح
    public int OrgMembershipID { get; set; }
    public int UserProfileID { get; set; }
    public int OrganizationID { get; set; }

    // ⚠️ MembershipType — يُبقى للتوافق (Deprecated)
    [Required]
    [MaxLength(50)]
    public string MembershipType { get; set; } = "EMPLOYEE";

    // ⚠️ 5 Can* Booleans — Deprecated (تُبقى للتوافق)
    public bool CanManageOrganization { get; set; } = false;
    public bool CanManageMembers { get; set; } = false;
    public bool CanManageFinance { get; set; } = false;
    public bool CanManageServices { get; set; } = false;
    public bool CanViewReports { get; set; } = false;

    // فترة الصلاحية (تبقى)
    public DateTime EffectiveFrom { get; set; } = DateTime.Now;
    public DateTime? EffectiveTo { get; set; }

    [MaxLength(500)]
    public string? AssignmentReason { get; set; }

    public bool IsActive { get; set; } = true;
    public DateTime CreatedDate { get; set; } = DateTime.Now;
    public DateTime? LastModifiedDate { get; set; }
    public int? AssignedByUserID { get; set; }

    // ⭐ العلاقات
    [ForeignKey("UserProfileID")]
    public virtual UserProfile UserProfile { get; set; }

    [ForeignKey("OrganizationID")]
    public virtual Organization Organization { get; set; }

    [ForeignKey("AssignedByUserID")]
    public virtual UserProfile? AssignedByUser { get; set; }

    // ⚠️ SystemJobTitleID + CustomJobTitleID
    // نُبقيها للتوافق، لكن المنطق الجديد يعتمد على MemberTitle
    public int? SystemJobTitleID { get; set; }
    public int? CustomJobTitleID { get; set; }

    [ForeignKey("SystemJobTitleID")]
    public virtual SystemJobTitle? SystemJobTitle { get; set; }

    [ForeignKey("CustomJobTitleID")]
    public virtual CustomJobTitle? CustomJobTitle { get; set; }

    // ⭐⭐⭐ المنطق الجديد: M:N مع CustomJobTitle (عبر MemberTitle)
    public virtual ICollection<MemberTitle> MemberTitles { get; set; } = new HashSet<MemberTitle>();
}
```

**⚠️ التغييرات:**

- **لا تعديل على البنية** — فقط `Deprecated` للحقول القديمة.
- **`MemberTitles`** — يُفعَّل (M:N).

---

#### 2.2.4 `UserRole` — لا تغيير

**⚠️ يبقى كما هو:**

- **`RoleType = Professional`** — للأطباء والصيادلة فقط.
- **`OrganizationID`** — للسجل النشط.
- **`CustomJobTitleID`** — يمكن استخدامه.

**⚠️ السبب:** الأدوار المهنية = تراخيص — تحتاج جدولاً منفصلاً.

---

### 2.3 مخطط ERD الكامل

```
User (ApplicationUser)
    │
    └── UserProfile
            │
            ├── UserRole (M:N)
            │   ├── RoleType = Professional (Doctor/Pharmacist)
            │   └── ProfessionalLicense
            │
            └── OrgMembership (M:N)
                    │
                    ├── Organization
                    │       └── OrganizationType
                    │
                    └── MemberTitles (M:N)
                            │
                            ├── CustomJobTitle
                            │       │
                            │       └── DefaultControlLevel
                            │               │
                            │               └── ControlLevelPermission (M:N)
                            │                       │
                            │                       └── Permission
                            │
                            ├── ControlLevel (Override)
                            │       └── ControlLevelPermission (M:N)
                            │               └── Permission
                            │
                            └── MemberTitleOverride (M:N)
                                    └── Permission
```

---

### 2.4 مثال تطبيقي شامل

**⚠️ السيناريو:**

```
شركة RubikCare (Organization #1 — OrganizationType = Platform)
│
├── ControlLevels:
│   ├── #1 — "مالك" (Default)
│   ├── #2 — "مدير عام"
│   ├── #3 — "مدير قسم التسويق"
│   ├── #4 — "أخصائي تسويق"
│   └── #5 — "مشرف"
│
├── Permissions:
│   ├── PSP.PROGRAM.VIEW
│   ├── PSP.PROGRAM.CREATE
│   ├── PSP.PROGRAM.EDIT
│   ├── PSP.PROGRAM.DELETE
│   ├── PSP.PROGRAM.APPROVE
│   ├── PSP.MEDICATION.EDIT_PRICE
│   └── ...
│
├── ControlLevelPermissions:
│   ├── #4 (أخصائي تسويق) → [PSP.PROGRAM.VIEW]
│   ├── #3 (مدير قسم) → [PSP.PROGRAM.VIEW, PSP.PROGRAM.CREATE, PSP.PROGRAM.EDIT]
│   ├── #2 (مدير عام) → [كل الصلاحيات]
│   └── #1 (مالك) → [كل الصلاحيات]
│
├── CustomJobTitles:
│   ├── #10 — "أخصائي تسويق" (DefaultControlLevel = #4)
│   └── #11 — "مدير تسويق" (DefaultControlLevel = #3)
│
└── Users:
    ├── User A: المسمى = #10 (أخصائي تسويق)
    │   └── OrgMembership
    │       └── MemberTitle #1 (CustomJobTitle = #10, ControlLevel = #4)
    │           الصلاحيات الفعلية = [PSP.PROGRAM.VIEW]
    │
    └── User B: المسمى = #10 (أخصائي تسويق)
        └── OrgMembership
            └── MemberTitle #2 (CustomJobTitle = #10, ControlLevel = #4)
                + MemberTitleOverride:
                    ├── PSP.PROGRAM.CREATE (IsGranted = true)
                    ├── PSP.PROGRAM.APPROVE (IsGranted = true)
                    └── PSP.MEDICATION.EDIT_PRICE (IsGranted = true)
                الصلاحيات الفعلية = [PSP.PROGRAM.VIEW + إعدادات إضافية]
```

**⚠️ النتيجة:**

- **User A** و **User B** — **نفس المسمى** (`أخصائي تسويق`).
- **صلاحيات مختلفة** — بفضل `MemberTitleOverride`.

---
# [#45] 📘 `19-PERMISSIONS` — Part 2 (التنفيذ + Migrations + Roadmap)

---

## 📌 الجزء الثالث: تدفق الفلترة (Algorithm)

### 3.1 المستويات الثلاثة — نظرة شاملة

```
عند فتح المستخدم للتطبيق:

Step 1: جلب المستخدم + UserProfile
   ↓
Step 2: جلب كل OrgMemberships (كل مؤسسات المستخدم)
   ↓
Step 3: المستخدم يختار منظمة نشطة
   ↓
Step 4: جلب MemberTitles (للعضو في المنظمة)
   ↓
Step 5: لكل MemberTitle:
   ├── ControlLevel (من MemberTitle أو DefaultControlLevel من CustomJobTitle)
   ├── ControlLevelPermissions (M:N)
   ├── MemberTitleOverrides (M:N)
   └── Merge → قائمة الصلاحيات الفعلية
   ↓
Step 6: تخزين في Session + JWT Claims
   ↓
Step 7: الاستخدام في كل Endpoint
```

### 3.2 الخطوات التفصيلية

#### 🔹 Step 1: جلب المستخدم

```csharp
var user = await _userManager.GetUserAsync(principal);
var userProfile = await _userProfileService.GetByApplicationUserIdAsync(user.Id);
```

#### 🔹 Step 2: جلب OrgMemberships

```csharp
var memberships = await _dbFactory.ExecuteReadOnlyAsync(async ctx => {
    var db = (DbContext)ctx;
    return await db.Set<OrgMembership>()
        .AsNoTracking()
        .Where(m => m.UserProfileID == userProfile.UserProfileID && m.IsActive)
        .Include(m => m.Organization)
            .ThenInclude(o => o.Type)
        .Include(m => m.MemberTitles)
            .ThenInclude(mt => mt.CustomJobTitle)
        .Include(m => m.MemberTitles)
            .ThenInclude(mt => mt.ControlLevel)
        .ToListAsync();
});
```

#### 🔹 Step 3: المستخدم يختار منظمة (أو النظام يختار الأولى)

```csharp
var activeMembership = memberships.FirstOrDefault(m => m.OrganizationID == currentOrgId)
    ?? memberships.First();
```

#### 🔹 Step 4: جلب MemberTitles + الصلاحيات

```csharp
var memberTitles = activeMembership.MemberTitles
    .Where(mt => mt.IsActive)
    .ToList();

var allPermissions = new HashSet<string>();

foreach (var mt in memberTitles)
{
    // 1. ControlLevel (من MemberTitle أو DefaultControlLevel)
    var controlLevelId = mt.ControlLevelID 
        ?? mt.CustomJobTitle?.DefaultControlLevelID;
    
    if (controlLevelId.HasValue)
    {
        // 2. ControlLevelPermissions
        var clPermissions = await _permissionResolver.GetControlLevelPermissionsAsync(controlLevelId.Value);
        foreach (var p in clPermissions)
        {
            if (p.IsGranted) allPermissions.Add(p.PermissionCode);
        }
    }
    
    // 3. MemberTitleOverrides (إضافات فردية)
    var overrides = await _permissionResolver.GetMemberTitleOverridesAsync(mt.MemberTitleID);
    foreach (var o in overrides)
    {
        if (o.IsGranted)
            allPermissions.Add(o.PermissionCode);
        else
            allPermissions.Remove(o.PermissionCode);  // منع صريح
    }
}
```

#### 🔹 Step 5: تخزين في JWT Claims

```csharp
var claims = new List<Claim>
{
    new Claim("userId", user.Id),
    new Claim("userProfileId", userProfile.UserProfileID.ToString()),
    new Claim("organizationId", activeMembership.OrganizationID.ToString()),
    new Claim("organizationTypeId", activeMembership.Organization.Type.TypeID.ToString()),
};

foreach (var perm in allPermissions)
{
    claims.Add(new Claim("permission", perm));
}
```

### 3.3 قاعدة الأولوية

> **"Overrides (فردية) > ControlLevel (جماعي)"**

**⚠️ المثال:**

```
User A في MemberTitle #1:
- ControlLevel = "أخصائي تسويق"
- ControlLevelPermissions = [VIEW, EDIT]
- MemberTitleOverride:
   - CREATE (IsGranted = true)
   - EDIT (IsGranted = false)

النتيجة:
- VIEW: ✅ (من ControlLevel)
- EDIT: ❌ (Override يمنع)
- CREATE: ✅ (Override يسمح)
```

---

## 📌 الجزء الرابع: الطبقات التقنية

### 4.1 التوزيع حسب الطبقة

| # | الطبقة | المكوّن | الموقع |
|:-:|:---|:---|:---|
| 1 | **Domain** | `ControlLevel` | `RubikCare.Domain/Entities/Permissions/` |
| 2 | **Domain** | `Permission` | نفس المجلد |
| 3 | **Domain** | `ControlLevelPermission` | نفس المجلد |
| 4 | **Domain** | `MemberTitleOverride` | نفس المجلد |
| 5 | **Domain** | `CustomJobTitle` (مُعدَّل) | `Entities/` |
| 6 | **Domain** | `MemberTitle` (مُعدَّل) | `Entities/Organization/` |
| 7 | **Application** | `IPermissionService` | `Application/Interfaces/Permissions/` |
| 8 | **Application** | `IPermissionResolver` | نفس المجلد |
| 9 | **Application** | `PermissionContextDto` | `Application/DTOs/Permissions/` |
| 10 | **Application** | `PermissionService` | `Application/Services/Permissions/` |
| 11 | **Infrastructure** | `PermissionResolver` | `Infrastructure/Services/Permissions/` |
| 12 | **Infrastructure** | Use Cases (CRUD) | `Infrastructure/UseCases/Permissions/` |
| 13 | **Api.Web** | `RequirePermissionAttribute` | `Api.Web/Attributes/` |
| 14 | **Api.Web** | `PermissionMiddleware` | `Api.Web/Middleware/` |
| 15 | **Shared.UI** | `IPermissionClient` | `Shared.UI/Services/Permissions/` |

### 4.2 `IPermissionService` — المسؤوليات

```csharp
public interface IPermissionService
{
    // فحص
    Task<bool> HasPermissionAsync(int userProfileId, int organizationId, string permissionCode);
    Task<List<string>> GetAllPermissionsAsync(int userProfileId, int organizationId);
    
    // إدارة ControlLevels
    Task<ControlLevelDto> CreateControlLevelAsync(CreateControlLevelDto dto, int currentUserId);
    Task<bool> UpdateControlLevelAsync(int controlLevelId, UpdateControlLevelDto dto, int currentUserId);
    Task<bool> DeleteControlLevelAsync(int controlLevelId, int currentUserId);
    Task<List<ControlLevelDto>> GetControlLevelsAsync(int organizationId);
    
    // إدارة Permissions على ControlLevel
    Task<bool> GrantPermissionAsync(int controlLevelId, int permissionId, int currentUserId);
    Task<bool> RevokePermissionAsync(int controlLevelId, int permissionId, int currentUserId);
    
    // إدارة Overrides
    Task<bool> GrantOverrideAsync(int memberTitleId, int permissionId, int currentUserId, string? reason);
    Task<bool> RevokeOverrideAsync(int memberTitleOverrideId, int currentUserId);
}
```

### 4.3 `PermissionContextDto`

```csharp
public class PermissionContextDto
{
    public int UserProfileId { get; set; }
    public int OrganizationId { get; set; }
    public int? ActiveMembershipId { get; set; }
    public List<int> MemberTitleIds { get; set; } = new();
    public HashSet<string> Permissions { get; set; } = new();
}
```

### 4.4 `PermissionResolver` — التدفق

```csharp
public class PermissionResolver : IPermissionResolver
{
    public async Task<HashSet<string>> ResolveUserPermissionsAsync(
        int userProfileId, int organizationId)
    {
        // 1. جلب MemberTitles
        var memberTitles = await GetActiveMemberTitlesAsync(userProfileId, organizationId);
        
        var permissions = new HashSet<string>();
        
        foreach (var mt in memberTitles)
        {
            // 2. ControlLevel
            var controlLevelId = mt.ControlLevelID ?? mt.CustomJobTitle?.DefaultControlLevelID;
            
            if (controlLevelId.HasValue)
            {
                var clPerms = await GetControlLevelPermissionsAsync(controlLevelId.Value);
                foreach (var p in clPerms.Where(p => p.IsGranted))
                    permissions.Add(p.PermissionCode);
            }
            
            // 3. Overrides (تتجاوز)
            var overrides = await GetMemberTitleOverridesAsync(mt.MemberTitleID);
            foreach (var o in overrides)
            {
                if (o.IsGranted)
                    permissions.Add(o.PermissionCode);
                else
                    permissions.Remove(o.PermissionCode);
            }
        }
        
        return permissions;
    }
}
```

---

## 📌 الجزء الخامس: JWT + Claims

### 5.1 Claims المُقترَحة

```csharp
public static class PermissionClaims
{
    public const string UserId = "userId";
    public const string UserProfileId = "userProfileId";
    public const string OrganizationId = "organizationId";
    public const string OrganizationTypeId = "organizationTypeId";
    public const string RoleType = "roleType";              // System | Professional
    public const string Permission = "permission";          // متعدد
}
```

### 5.2 توليد JWT

```csharp
public async Task<string> GenerateTokenAsync(ApplicationUser user, int activeOrganizationId)
{
    var userProfile = await _userProfileService.GetByApplicationUserIdAsync(user.Id);
    
    // الصلاحيات
    var permissions = await _permissionResolver.ResolveUserPermissionsAsync(
        userProfile.UserProfileID, activeOrganizationId);
    
    // الأدوار المهنية
    var professionalRoles = await _userRoleService.GetProfessionalRolesAsync(userProfile.UserProfileID);
    
    var claims = new List<Claim>
    {
        new Claim(PermissionClaims.UserId, user.Id),
        new Claim(PermissionClaims.UserProfileId, userProfile.UserProfileID.ToString()),
        new Claim(PermissionClaims.OrganizationId, activeOrganizationId.ToString()),
    };
    
    foreach (var role in professionalRoles)
    {
        claims.Add(new Claim(PermissionClaims.RoleType, role.RoleType));
    }
    
    foreach (var perm in permissions)
    {
        claims.Add(new Claim(PermissionClaims.Permission, perm));
    }
    
    var key = new SymmetricSecurityKey(Encoding.UTF8.GetBytes(_jwtSettings.Key));
    var creds = new SigningCredentials(key, SecurityAlgorithms.HmacSha256);
    
    var token = new JwtSecurityToken(
        issuer: _jwtSettings.Issuer,
        audience: _jwtSettings.Audience,
        claims: claims,
        expires: DateTime.UtcNow.AddHours(_jwtSettings.ExpiryHours),
        signingCredentials: creds);
    
    return new JwtSecurityTokenHandler().WriteToken(token);
}
```

### 5.3 حجم الـ Token

**⚠️ ملاحظة مهمة:**

- **مع 50-100 صلاحية** — JWT يصبح كبيراً (~5-10 KB).
- **الحد الأقصى للـ Header** — عادة 8-16 KB.
- **الحل:** تخزين الصلاحيات في **Memory Cache** بدلاً من JWT.

**⚠️ القرار:** 

- **A)** JWT (بسيط، لكن كبير) — للصلاحيات القليلة (<30).
- **B)** Memory Cache (معقد، لكن scalable) — للصلاحيات الكثيرة.

**⚠️ توصيتي:** **B** — Memory Cache.

---

## 📌 الجزء السادس: Middleware موحد

### 6.1 `RequirePermissionAttribute`

```csharp
[AttributeUsage(AttributeTargets.Method | AttributeTargets.Class, AllowMultiple = true)]
public class RequirePermissionAttribute : Attribute
{
    public string[] PermissionCodes { get; }
    public bool RequireAll { get; set; } = false;   // true = الكل، false = واحد

    public RequirePermissionAttribute(params string[] permissionCodes)
    {
        PermissionCodes = permissionCodes;
    }
}
```

### 6.2 الاستخدام في Controllers

```csharp
[ApiController]
[Route("api/psp/programs")]
[Authorize]
public class PspProgramsController : ControllerBase
{
    [HttpGet]
    [RequirePermission("PSP.PROGRAM.VIEW")]
    public async Task<IActionResult> GetPrograms() { ... }
    
    [HttpPost]
    [RequirePermission("PSP.PROGRAM.CREATE")]
    public async Task<IActionResult> CreateProgram(...) { ... }
    
    [HttpPost("{id}/approve")]
    [RequirePermission("PSP.PROGRAM.APPROVE")]
    public async Task<IActionResult> ApproveProgram(int id) { ... }
}
```

### 6.3 `PermissionMiddleware`

```csharp
public class PermissionMiddleware
{
    private readonly RequestDelegate _next;
    private readonly IMemoryCache _cache;
    
    public async Task InvokeAsync(HttpContext context, IPermissionService permissionService)
    {
        var endpoint = context.GetEndpoint();
        var requirement = endpoint?.Metadata.GetMetadata<RequirePermissionAttribute>();
        
        if (requirement != null)
        {
            var userProfileId = context.User.FindFirst("userProfileId")?.Value;
            var orgId = context.User.FindFirst("organizationId")?.Value;
            
            if (string.IsNullOrEmpty(userProfileId) || string.IsNullOrEmpty(orgId))
            {
                context.Response.StatusCode = 401;
                return;
            }
            
            var cacheKey = $"perms_{userProfileId}_{orgId}";
            var userPermissions = await _cache.GetOrCreateAsync(cacheKey, async entry =>
            {
                entry.AbsoluteExpirationRelativeToNow = TimeSpan.FromMinutes(15);
                return await permissionService.GetAllPermissionsAsync(
                    int.Parse(userProfileId), int.Parse(orgId));
            });
            
            var hasPermission = requirement.RequireAll
                ? requirement.PermissionCodes.All(p => userPermissions.Contains(p))
                : requirement.PermissionCodes.Any(p => userPermissions.Contains(p));
            
            if (!hasPermission)
            {
                context.Response.StatusCode = 403;
                await context.Response.WriteAsJsonAsync(new
                {
                    success = false,
                    message = "ليس لديك صلاحية للقيام بهذا الإجراء",
                    requiredPermissions = requirement.PermissionCodes
                });
                return;
            }
        }
        
        await _next(context);
    }
}
```

### 6.4 التسجيل في `Program.cs`

```csharp
app.UseAuthentication();
app.UseAuthorization();
app.UseMiddleware<PermissionMiddleware>();  // ⭐ بعد Authorization
```

**⚠️ الترتيب إلزامي:**

```
UseAuthentication → UseAuthorization → PermissionMiddleware
```

---

## 📌 الجزء السابع: UI

### 7.1 Super Admin UI

**⚠️ الصفحات المطلوبة:**

| # | الصفحة | الوظيفة |
|:-:|:---|:---|
| 1 | `/admin/permissions/list` | قائمة كل الـ Permissions |
| 2 | `/admin/permissions/create` | إضافة Permission |
| 3 | `/admin/permissions/edit/{id}` | تعديل |
| 4 | `/admin/control-levels/{orgId}` | مستويات التحكم لكل منظمة |
| 5 | `/admin/control-levels/{orgId}/edit/{id}` | تعديل مستوى |
| 6 | `/admin/control-levels/{orgId}/permissions/{id}` | ربط Permissions |

### 7.2 Organization Admin UI

**⚠️ الصفحات المطلوبة:**

| # | الصفحة | الوظيفة |
|:-:|:---|:---|
| 1 | `/org/{orgId}/control-levels` | مستويات التحكم |
| 2 | `/org/{orgId}/control-levels/create` | إضافة |
| 3 | `/org/{orgId}/control-levels/{id}/permissions` | ربط Permissions |
| 4 | `/org/{orgId}/members` | الأعضاء |
| 5 | `/org/{orgId}/members/{id}/titles` | ألقاب العضو |
| 6 | `/org/{orgId}/members/{id}/overrides` | صلاحيات فردية |
| 7 | `/org/{orgId}/job-titles` | المسميات |
| 8 | `/org/{orgId}/job-titles/{id}/control-level` | ربط المسمى بمستوى |

---

## 📌 الجزء الثامن: Migrations

### 8.1 Migrations المطلوبة

**⚠️ 6 Migrations — بترتيب:**

| # | Migration | الوصف |
|:-:|:---|:---|
| 1 | `AddControlLevelsTable` | إنشاء جدول `ControlLevels` |
| 2 | `AddPermissionsTable` | إنشاء جدول `Permissions` |
| 3 | `AddControlLevelPermissionsTable` | M:N |
| 4 | `AddMemberTitleOverridesTable` | إضافات فردية |
| 5 | `ExtendCustomJobTitleWithDefaultControlLevel` | إضافة عمود |
| 6 | `ExtendMemberTitleWithControlLevel` | إضافة عمودين |

### 8.2 مثال Migration 1

```csharp
migrationBuilder.CreateTable(
    name: "ControlLevels",
    columns: table => new
    {
        ControlLevelID = table.Column<int>(nullable: false)
            .Annotation("SqlServer:Identity", "1, 1"),
        OrganizationID = table.Column<int>(nullable: false),
        LevelNameAr = table.Column<string>(maxLength: 100, nullable: false),
        LevelNameEn = table.Column<string>(maxLength: 100, nullable: true),
        Description = table.Column<string>(maxLength: 500, nullable: true),
        IsDefault = table.Column<bool>(nullable: false, defaultValue: false),
        LevelOrder = table.Column<int>(nullable: false, defaultValue: 100),
        IsActive = table.Column<bool>(nullable: false, defaultValue: true),
        CreatedDate = table.Column<DateTime>(nullable: false, defaultValueSql: "GETUTCDATE()"),
        CreatedByUserID = table.Column<int>(nullable: true),
        LastModifiedDate = table.Column<DateTime>(nullable: true)
    },
    constraints: table =>
    {
        table.PrimaryKey("PK_ControlLevels", x => x.ControlLevelID);
        table.ForeignKey("FK_ControlLevels_Organizations_OrganizationID",
            x => x.OrganizationID, "Organizations", "OrganizationID", 
            onDelete: ReferentialAction.Restrict);
        table.ForeignKey("FK_ControlLevels_UserProfiles_CreatedByUserID",
            x => x.CreatedByUserID, "UserProfiles", "UserProfileID",
            onDelete: ReferentialAction.SetNull);
    });

migrationBuilder.CreateIndex("IX_ControlLevels_OrganizationID", 
    "ControlLevels", "OrganizationID");
migrationBuilder.CreateIndex("IX_ControlLevels_IsDefault", 
    "ControlLevels", "IsDefault", 
    filter: "[IsDefault] = 1");
```

### 8.3 Data Seeding

**⚠️ بعد Migrations — Seed:**

```csharp
// Seed Default Permissions
var permissions = new[]
{
    new Permission { PermissionCode = "PSP.PROGRAM.VIEW", PermissionNameAr = "عرض برامج الدعم", Scope = "PSP" },
    new Permission { PermissionCode = "PSP.PROGRAM.CREATE", PermissionNameAr = "إنشاء برنامج دعم", Scope = "PSP" },
    new Permission { PermissionCode = "PSP.PROGRAM.EDIT", PermissionNameAr = "تعديل برنامج دعم", Scope = "PSP" },
    new Permission { PermissionCode = "PSP.PROGRAM.DELETE", PermissionNameAr = "حذف برنامج دعم", Scope = "PSP" },
    new Permission { PermissionCode = "PSP.PROGRAM.APPROVE", PermissionNameAr = "اعتماد برنامج دعم", Scope = "PSP" },
    // ... إلخ
};
```

---

## 📌 الجزء التاسع: Checklist

### 9.1 عند إضافة Permission جديدة

- [ ] تحديد `PermissionCode` (مثال: `PSP.PROGRAM.VIEW`).
- [ ] تحديد `Scope` (`PSP`, `LOYALTY`, `PHARMACY`, إلخ).
- [ ] إضافة SQL (MERGE).
- [ ] ربط بـ `ControlLevels` (M:N).
- [ ] استخدامها في `Controller` بـ `[RequirePermission]`.
- [ ] إخفاء/إظهار الأزرار في UI حسب `HasPermission`.

### 9.2 عند إضافة Control Level جديد

- [ ] `OrganizationID` — أي منظمة؟
- [ ] `LevelNameAr` — الاسم.
- [ ] `LevelOrder` — الترتيب.
- [ ] ربط `Permissions` (M:N).
- [ ] (اختياري) ربط `CustomJobTitle` افتراضي.

### 9.3 عند إضافة Member Title Override

- [ ] `MemberTitleID` — أي عضو؟
- [ ] `PermissionID` — أي صلاحية؟
- [ ] `IsGranted` — منح أم منع؟
- [ ] `Reason` — السبب (للتدقيق).

---

## 📌 الجزء العاشر: Roadmap + Changelog

### 10.1 Roadmap

| Phase | الوصف | الأسبوع |
|:---:|:---|:---:|
| **Phase 1** | Domain + Migrations + Data Seeding | 1 |
| **Phase 2** | Application Service + Infrastructure | 2 |
| **Phase 3** | JWT Claims + Middleware | 3 |
| **Phase 4** | Super Admin UI | 4 |
| **Phase 5** | Organization Admin UI | 5 |
| **Phase 6** | Integration في Controllers | 6-8 |
| **Phase 7** | Integration في UI | 8-10 |

### 10.2 Changelog

| الإصدار | التاريخ | التغييرات |
|:---:|:---|:---|
| **1.0** | **16 أكتوبر 2026** | التصميم الكامل — Philosophy + ERD + Migrations + Roadmap |

### 10.3 روابط ذات صلة

- [00 - MASTER-INDEX](./00-MASTER-INDEX.md)
- [00B - PROJECT-BACKLOG](./00B-PROJECT-BACKLOG.md)
- [02 - IDENTITY](./02-IDENTITY.md)
- [03 - MENUS](./03-MENUS.md)
- [11 - PAGE-CREATION](./11-PAGE-CREATION.md)

---

**© 2026 RubikCare — للاستخدام الداخلي**

**نهاية الوثيقة ✅**
```
