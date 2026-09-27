## 📄 وثيقة `07-psp-system.md` المحدّثة

**انسخ الكود ده كامل والصقه في الملف:**

```markdown
# 07 — نظام PSP (Patient Support Programs)

## 📌 نظرة عامة

نظام **PSP** هو القلب الأساسي لمنصة RubikCare — بيدير **برامج دعم المرضى** اللي بتقدمها شركات الأدوية.

**المكونات الأساسية:**
- **Program** — البرنامج (علاج السكري، علاج الأنيميا...)
- **ProgramMedication** — الأدوية المرتبطة بالبرنامج
- **DispensationPlan** — خطة الصرف
- **DispensationPlanPhase** — ⭐ مراحل خطة الصرف
- **Dispensation** — عمليات الصرف الفعلية
- **Patient** — المرضى المشاركين
- **Invitation** — الدعوات
- **Participation** — المشاركات
- **eRX** — الروشتات الإلكترونية

---

## 🧱 هيكل الكيانات (Domain Entities)

### 📂 المسار:
```
RubikCare.Domain\Entities\PSP\
├── Config\          ← إعدادات
│   ├── PSPRequiredDataEntry.cs
│   ├── PSPRequiredFollowUp.cs
│   └── PSPRequiredTest.cs
│
├── Core\            ← القلب
│   ├── LinkTargetAudience.cs
│   ├── PSPDispensation.cs
│   ├── PSPDispensationPlan.cs
│   ├── PSPDispensationPlanMedication.cs
│   ├── PSPDispensationPlanPhase.cs               ⭐ جديد
│   ├── PSPDispensationPlanPhaseMedication.cs     ⭐ جديد
│   ├── PSPeRX.cs
│   ├── PSPInvitation.cs
│   ├── PSPParticipation.cs
│   ├── PSPPatient.cs
│   ├── PSPProgram.cs
│   ├── PSPProgramLink.cs
│   ├── PSPProgramMedication.cs
│   └── PSPProgramSpeciality.cs
│
├── Doctor\
│   └── DoctorNote.cs
│
├── Execution\
│   ├── PSPFollowUpRecord.cs
│   └── PSPTestResult.cs
│
└── Patient\
    └── PSPAdverseEvent.cs
```

---

## 🔗 خريطة العلاقات

```
PSPProgram (البرنامج)
    │
    ├── ProgramID ──┬──> PSPProgramMedications (أدوية البرنامج)
    │               │         │
    │               │         └── ProgramMedicationID ──┐
    │               │                                   │
    │               └──> PSPDispensationPlans (خطط الصرف)
    │                         │
    │                         └── PlanID ──┐
    │                                      │
    │                                      ▼
    │                          PSPDispensationPlanMedications
    │                          (Legacy — دواء في الخطة مباشرة)
    │
    └── ProgramID ──> PSPRequiredTests / PSPRequiredFollowUps / PSPRequiredDataEntries
```

### ⭐ مع Phase System الجديد:

```
PSPProgram
  └── PSPDispensationPlan
        ├── PSPDispensationPlanMedication (Legacy — للتوافق)
        └── PSPDispensationPlanPhase ⭐ جديد
              └── PSPDispensationPlanPhaseMedication ⭐ جديد
                    └── PSPProgramMedication
```

---

## 🎯 نظام مراحل خطة الصرف (Phase System) ⭐ جديد

### 📌 الهدف

كل **خطة صرف** ممكن تحتوي على **مراحل متعددة** — كل مرحلة ليها:
- **مدة زمنية** (3 شهور، 6 شهور...)
- **تبدأ بعد كام شهر** من بداية البرنامج
- **أدوية خاصة بيها**
- **تفاصيل كاملة**: كام علبة/شهر، التقسيمة، الخصم

### 🎬 السيناريو

```
برنامج: علاج السكري
  └── خطة الصرف: "الخطة الأساسية"
        ├── المرحلة الأولى (0-3 شهور)
        │     ├── Metformin — 2 علبة/شهر — 3 مرات يومياً
        │     └── Insulin — 1 علبة/شهر — 2 مرات يومياً
        └── المرحلة الثانية (3-6 شهور)
              ├── Metformin — 3 علبة/شهر — 4 مرات يومياً
              └── Glimepiride — 1 علبة/شهر — 1 مرة يومياً
```

### 🏗️ الكيانات

#### 1️⃣ `PSPDispensationPlanPhase`

**المسار:** `RubikCare.Domain\Entities\PSP\Core\PSPDispensationPlanPhase.cs`

```csharp
[Table("PSPDispensationPlanPhases")]
public class PSPDispensationPlanPhase
{
    [Key]
    public int PhaseID { get; set; }
    
    public int PlanID { get; set; }  // FK → PSPDispensationPlans (CASCADE)
    
    public string PhaseName { get; set; }   // "المرحلة الأولى"
    public int PhaseOrder { get; set; }     // 1, 2, 3...
    public int DurationMonths { get; set; } // 3, 6...
    public int StartAfterMonths { get; set; } // 0 = من بداية البرنامج
    public string? Description { get; set; }
    
    public bool IsActive { get; set; } = true;
    public DateTime CreatedDate { get; set; }
    public DateTime? LastModifiedDate { get; set; }
    
    // Navigation
    public virtual PSPDispensationPlan? Plan { get; set; }
    public virtual ICollection<PSPDispensationPlanPhaseMedication> PhaseMedications { get; set; }
}
```

#### 2️⃣ `PSPDispensationPlanPhaseMedication`

**المسار:** `RubikCare.Domain\Entities\PSP\Core\PSPDispensationPlanPhaseMedication.cs`

```csharp
[Table("PSPDispensationPlanPhaseMedications")]
public class PSPDispensationPlanPhaseMedication
{
    [Key]
    public int PhaseMedicationID { get; set; }
    
    public int PhaseID { get; set; }              // FK → PSPDispensationPlanPhases (CASCADE)
    public int ProgramMedicationID { get; set; }  // FK → PSPProgramMedications (NO_ACTION)
    
    public int QuantityPerMonth { get; set; }     // كام علبة/شهر
    public int? TimesPerDay { get; set; }         // 3 = ثلاث مرات يومياً
    public int? HoursBetweenDoses { get; set; }   // 8 = كل 8 ساعات
    public decimal DiscountPercentage { get; set; } // نسبة الخصم
    
    public bool IsActive { get; set; } = true;
    public DateTime CreatedDate { get; set; }
    public DateTime? LastModifiedDate { get; set; }
    
    // Navigation
    public virtual PSPDispensationPlanPhase? Phase { get; set; }
    public virtual PSPProgramMedication? ProgramMedication { get; set; }
}
```

### ⚠️ قواعد الـ Foreign Keys

| العلاقة | OnDelete | السبب |
|---------|----------|-------|
| `Phase.PlanID → Plan` | **CASCADE** | لو الخطة اتمسحت، المراحل تتمسح |
| `PhaseMedication.PhaseID → Phase` | **CASCADE** | لو المرحلة اتمسحت، الأدوية تتمسح |
| `PhaseMedication.ProgramMedicationID → ProgramMedication` | **NO_ACTION** | يمنع مسح الدواء لو فيه مراحل بتشير ليه |

**السبب:** `ProgramMedication` هو **بيانات مرجعية** (Master Data)، مش **بيانات تشغيلية**.

### 📊 الـ Migration

**ID:** `20260923150028_AddPSPDispensationPlanPhases`

**الـ Tables الجديدة:**
- `PSPDispensationPlanPhases`
- `PSPDispensationPlanPhaseMedications`

**الـ Columns الجديدة:**
- `PSPDispensationPlanMedications.PhaseID` (nullable) — للتوافق

---

## 📚 الـ DTOs

### المسار: `RubikCare.Application\DTOs\PSP\`

| # | الملف | الاستخدام |
|---|-------|-----------|
| 1 | `DispensationPlanPhaseDto.cs` | Read |
| 2 | `DispensationPlanPhaseMedicationDto.cs` | Read |
| 3 | `CreateDispensationPlanPhaseDto.cs` | Write |
| 4 | `CreateDispensationPlanPhaseMedicationDto.cs` | Write |

**تعديل على:** `DispensationPlanDto.cs` — إضافة `List<DispensationPlanPhaseDto> Phases`

---

## 🌐 الـ API Endpoints

### المسار: `Api.Web\Controllers\PSP\PspController.DispensationPlans.cs`

| # | Method | Route | الوظيفة |
|---|--------|-------|---------|
| 1 | `GET` | `api/psp/plans/{planId}/phases` | جلب كل مراحل خطة |
| 2 | `GET` | `api/psp/phases/{phaseId}` | جلب مرحلة واحدة |
| 3 | `POST` | `api/psp/plans/{planId}/phases` | إضافة مرحلة |
| 4 | `PUT` | `api/psp/phases/{phaseId}` | تعديل مرحلة |
| 5 | `DELETE` | `api/psp/phases/{phaseId}` | حذف مرحلة |
| 6 | `GET` | `api/psp/plans/{planId}/available-medications` | جلب الأدوية المتاحة |

---

## 🎨 الـ UI

### 1️⃣ صفحة تعديل البرنامج

**الملف:** `Rubikcare.Web\Components\Pages\Professional\PSPSteps\PSPStep2_DispensationPlans.razor`

**النمط:** **Inline Forms** (مش Modals)

**البنية:**
```
قائمة الخطط
    └── [+ إضافة خطة]
            └── Inline Form (تابين):
                ├── تاب 1: الإعدادات
                └── تاب 2: المراحل
                    ├── [+ إضافة مرحلة]
                    │       └── Inline Form (فرعي)
                    │           ├── بيانات المرحلة
                    │           └── قسم الأدوية
                    │               ├── [+ إضافة دواء]
                    │               │       └── Inline Form (فرعي فرعي)
                    │               └── قائمة الأدوية
                    └── قائمة المراحل
```

**الحجم:** ~1349 سطر

### 2️⃣ صفحة تفاصيل البرنامج

**الملف:** `Rubikcare.Web\Components\Pages\Professional\PharmaCompany\PSP\PSPProgramsDetails.razor`

**التحديث:** عرض المراحل تحت كل خطة صرف

**النمط:** Phases Mini (بطاقات صغيرة)

---

## 🎨 الـ CSS

### الملفات:

| # | الملف | الحجم |
|---|-------|-------|
| 1 | `Shared.UI\wwwroot\css\_pages\Organization\PSP\PSPEditSteps\PSPStep2_DispensationPlans.css` | ~1213 سطر |

### الـ Classes الأساسية:

| # | القسم | Classes |
|---|-------|---------|
| 1 | **Inline Form** | `.s2dp-inline-form`, `.s2dp-inline-hdr`, `.s2dp-inline-body`, `.s2dp-inline-footer` |
| 2 | **Variants** | `.s2dp-inline-form--nested`, `.s2dp-inline-form--deep` |
| 3 | **Tabs** | `.s2dp-inline-tabs`, `.s2dp-inline-tab` |
| 4 | **Buttons** | `.s2dp-btn-outline`, `.s2dp-btn-primary` |
| 5 | **Phases** | `.s2dp-phase-card`, `.s2dp-phase-info` |
| 6 | **Meds** | `.s2dp-med-card`, `.s2dp-med-details` |

---

## 🌐 مفاتيح الترجمة

### Module: `PSP`

**المفاتيح الجديدة** (~41 مفتاح):

| البادئة | الوصف |
|---------|-------|
| `PSP.DP.MODAL.TAB_*` | تابين الـ Modal |
| `PSP.DP.PHASES.*` | قسم المراحل |
| `PSP.DP.PHASE_MODAL.*` | Modal المرحلة |
| `PSP.DP.MED_MODAL.*` | Modal الدواء |
| `PSP.DP.MSG.*` | رسائل |
| `PSP.DP.UNIT.*` | وحدات |

---

## 🎯 الخطوات المنجزة

| # | الخطوة | الحالة |
|---|--------|--------|
| 1 | إنشاء `PSPDispensationPlanPhase` + `PhaseMedication` | ✅ |
| 2 | تعديل `PSPDispensationPlan` + `PlanMedication` | ✅ |
| 3 | Migration جديدة | ✅ |
| 4 | DTOs + Endpoints في الـ API | ✅ |
| 5 | إعادة بناء `PSPStep2_DispensationPlans.razor` | ✅ |
| 6 | اختبار + ربط | ✅ |
| 7 | صفحة تفاصيل البرنامج | ✅ |
| 8 | مفاتيح الترجمة | ✅ |

---

## 🔍 ملاحظات مهمة

### 1️⃣ الفرق بين Phase والـ Plan:

| العنصر | الوصف | مثال |
|--------|-------|------|
| **Plan** | خطة الصرف الكاملة | "خطة العلاج الأساسية" |
| **Phase** | مرحلة داخل الخطة | "المرحلة الأولى — 3 شهور" |
| **PhaseMedication** | دواء داخل المرحلة | "Dispirin 100mg — علبة/شهر — 3 مرات يومياً" |

### 2️⃣ Legacy Support:

**`PSPDispensationPlanMedication`** لسه موجود للتوافق مع:
- الخطط القديمة
- الخطط بدون Phases

**`PSPDispensationPlanMedication.PhaseID`** ← nullable للربط بالمراحل

### 3️⃣ Inline Forms vs Modals:

**السبب في التحويل لـ Inline Forms:**
- مساحة أكبر
- تفاصيل أوضح
- مش محتاج Modals فوق بعض
- Deep Linking (URL مباشر)

---

## 🎯 الملفات المتأثرة

### Domain:
| الملف | الإجراء |
|-------|---------|
| `PSPDispensationPlanPhase.cs` | ⭐ جديد |
| `PSPDispensationPlanPhaseMedication.cs` | ⭐ جديد |
| `PSPDispensationPlan.cs` | ✏️ إضافة `Phases` |
| `PSPDispensationPlanMedication.cs` | ✏️ إضافة `PhaseID?` |

### Application:
| الملف | الإجراء |
|-------|---------|
| `DispensationPlanPhaseDto.cs` | ⭐ جديد |
| `DispensationPlanPhaseMedicationDto.cs` | ⭐ جديد |
| `CreateDispensationPlanPhaseDto.cs` | ⭐ جديد |
| `CreateDispensationPlanPhaseMedicationDto.cs` | ⭐ جديد |
| `DispensationPlanDto.cs` | ✏️ إضافة `Phases` |

### Infrastructure:
| الملف | الإجراء |
|-------|---------|
| Migrations | ⭐ `AddPSPDispensationPlanPhases` |

### API:
| الملف | الإجراء |
|-------|---------|
| `PspController.DispensationPlans.cs` | ⭐ جديد |

### Web:
| الملف | الإجراء |
|-------|---------|
| `PSPStep2_DispensationPlans.razor` | ✏️ إعادة بناء |
| `PSPStep2_DispensationPlans.css` | ✏️ تحديث |
| `PSPProgramsDetails.razor` | ✏️ إضافة قسم المراحل |

---

**نهاية الوثيقة 🚀**
```

---

## 🎯 خطوات التنفيذ

### 1️⃣ **انسخ الكود كامل**

### 2️⃣ **افتح الملف على GitHub:**

```
https://github.com/Rubikans-Egy/Rubikcare-docs/blob/main/docs/07-psp-system.md
```

### 3️⃣ **اضغط Edit (قلم)**

### 4️⃣ **الصق الكود الجديد كامل**

### 5️⃣ **Commit changes**

**Commit message:**
```
docs: add Phase System section to PSP docs
```

---

## 📋 ملخص التحديثات

| # | القسم | الوصف |
|---|-------|-------|
| 1 | **نظرة عامة** | محدّثة بالكيانات الجديدة |
| 2 | **هيكل الكيانات** | إضافة Phase + PhaseMedication |
| 3 | **خريطة العلاقات** | محدّثة |
| 4 | **Phase System** ⭐ | قسم كامل جديد |
| 5 | **الـ DTOs** | 4 ملفات جديدة |
| 6 | **الـ API Endpoints** | 6 endpoints |
| 7 | **الـ UI** | Inline Forms |
| 8 | **الـ CSS** | Classes جديدة |
| 9 | **مفاتيح الترجمة** | 41 مفتاح |
| 10 | **الخطوات المنجزة** | محدّثة |
| 11 | **ملاحظات مهمة** | إضافات |
| 12 | **الملفات المتأثرة** | محدّثة |

---
