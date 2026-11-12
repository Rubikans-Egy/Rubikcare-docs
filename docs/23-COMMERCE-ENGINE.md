
# 23 — COMMERCE ENGINE (محرك التجارة)

الإصدار: 1.1 (Implementation — Sprint 1-4 Completed)
آخر تحديث: 2026-11-12
الحالة: 🟢 B2B MVP مُنفَّذ (Backend + UI للصيدلي والشركة) + نظام البونص + الحساب المالي
**الجمهور:** فريق التطوير + AI Assistants
**المرجع:** يُقرأ قبل أي عمل على B2B / Commerce
**الوثائق المرتبطة:** `01-DATABASE-MANAGEMENT.md`, `06-POINTS.md`, `17-STRATEGIC-ROADMAP.md`

---

## 📌 مقدمة

### 0.1 ما هو Commerce Engine؟

**Commerce Engine** هو الطبقة الأساسية التي تُدير **العمليات التجارية** في RubikCare — الطلبات، الدفع، الشحن، الفواتير، التسويات المالية — بين **منظمتين** على المنصة.

**الرؤية:** ليس نظاماً خاصاً بـ B2B فقط، بل **محرك عام** يمكن استخدامه في:
- **B2B Supply** (صيدلية ← شركة أدوية) — **هذا هو البداية**.
- **Clinic OS** (مريض ← عيادة) — لاحقاً.
- **Pharmacy Retail** (مريض ← صيدلية) — لاحقاً.
- **Booking** (مريض ← عيادة) — لاحقاً.
- **Marketplace** (أي كيان ← أي كيان) — لاحقاً.

### 0.2 لماذا Commerce Engine منفصل؟

**المشكلة الحالية:** `MedicationRequest` موجود (مريض ← صيدلية)، لكنه:
- ❌ ليس Generic — مغلق على نوع واحد.
- ❌ لا يدعم الشحن/الفواتير/الدفع.
- ❌ لا يمكن إعادة استخدامه.

**الحل:** `Commerce Engine` = طبقة عامة + `Module Layer` لكل حالة استخدام.

### 0.3 الفرق بين ERP و Commerce Engine

| ERP (Odoo/SAP) | Commerce Engine (RubikCare) |
|---|---|
| يدير **الشركة نفسها** (موارد، موظفين، محاسبة) | يدير **العلاقة بين منظمتين** (عبر المنصة) |
| Accounting مزدوج كامل (GL, Trial Balance) | Ledger بسيط بين طرفين |
| Inventory متعدد + Costing (FIFO/LIFO) | Stock بسيط (كمية + سعر) |
| Manufacturing + MRP | لا |
| HR + Payroll | لا |
| Multi-currency + Tax Engine كامل | EGP فقط الآن، Tax بسيط |

**ما نبنيه = Marketplace + Settlement Layer** — لا ERP.

### 0.4 ما لا نبنيه (الحد الصريح)

- ❌ General Ledger / Journal Entries / Trial Balance.
- ❌ Financial Statements.
- ❌ Cost Accounting (FIFO/LIFO).
- ❌ Multi-Warehouse Transfers / Adjustments.
- ❌ Depreciation / Assets.
- ❌ Manufacturing / MRP.

**لماذا؟** لأننا لسنا ERP. نحن نُدير **دورة الطلب والدفع بين منظمتين** — لا محاسبة كاملة.

### 0.5 التكامل مع ERP خارجي (لاحقاً)

**الرؤية:** عند نمو المنصة (> 50 شركة)، يمكن لكل شركة ربط ERP خاص بها عبر `Integration Hub` (موجود في `22-INTEGRATION-HUB.md`).

**مثال:**
- شركة A تستخدم Odoo → `OdooAdapter` يقرأ المخزون/الأسعار/الطلبات.
- شركة B تستخدم SAP → `SapAdapter`.
- Platform يبقى المصدر الوحيد للحقيقة (Source of Truth).

**لماذا لا نبدأ بـ ERP الآن؟**
- تعقيد كبير (Odoo API معقد، SAP مستحيل).
- التكامل يستهلك 3× وقت البناء البسيط.
- ما نبنيه الآن **بسيط جداً** (~5 كيانات)، فتكامل ERP مبالغة.

---
## 📊 0.6 حالة التنفيذ الفعلية (Implementation Status)

آخر تحديث: 2026-11-12.

### ✅ ما تم إنجازه (Completed)

| المرحلة | المدة | تاريخ البدء | تاريخ الانتهاء | الحالة |
|---|---|---|---|---|
| **Domain Entities + Migration (Engine + B2B)** | ~2 يوم | 2026-11-06 | 2026-11-08 | ✅ مكتمل |
| **Application Services (SupplyOrder + Catalog + Ledger)** | ~3 أيام | 2026-11-08 | 2026-11-10 | ✅ مكتمل |
| **API Controllers (SupplyOrders + Catalog + Ledger)** | ~2 يوم | 2026-11-09 | 2026-11-10 | ✅ مكتمل |
| **B2B Future Features (Tiers + Bundles + Campaigns + Bonuses + Logistics)** | ~2 يوم | 2026-11-10 | 2026-11-11 | ✅ مكتمل (Entities فقط + Campaigns CRUD) |
| **UI Phase 1 (Pharmacy — 5 صفحات)** | ~2 يوم | 2026-11-09 | 2026-11-11 | ✅ مكتمل |
| **UI Phase 2 (Company — Orders + Details)** | ~1 يوم | 2026-11-10 | 2026-11-11 | ✅ مكتمل |
| **Commerce Settings Hub (Company)** | ~1 يوم | 2026-11-11 | 2026-11-11 | ✅ مكتمل |
| **Campaigns Page (Full CRUD)** | ~1 يوم | 2026-11-11 | 2026-11-11 | ✅ مكتمل |
| **Bonus System (Tiers + Auto-Calc)** | ~1 يوم | 2026-11-11 | 2026-11-12 | ✅ مكتمل |
| **ProductDetailsPage (Pharmacy)** | ~1 يوم | 2026-11-11 | 2026-11-12 | ✅ مكتمل (مع BONUS CALCULATOR) |
| **Price & Tax Model (Simplified)** | — | 2026-11-11 | 2026-11-11 | ✅ مكتمل |

### 🟡 ما هو قيد التطوير (In Progress)

| الميزة | الحالة | ملاحظات |
|---|---|---|
| **Ledger UI (Pharmacy + Company)** | ⏳ لم يبدأ | الكيانات جاهزة (ServiceLedger, LedgerTransaction) |
| **Checkout Page (طريقة الدفع + رصيد)** | ⏳ لم يبدأ | يحتاج تعديل `SupplyOrderService` + `LedgerService` |
| **Bonus Page (Full CRUD للشركة)** | 🟡 جزئي | موجود (في صفحة المنتج للشركة) |
| **Tiers Page (الخصومات المؤجلة)** | ⏳ لم يبدأ | الكيان موجود، يحتاج API + UI |
| **Logistics Page (الشحن)** | ⏳ لم يبدأ | الكيان موجود، يحتاج API + UI |
| **Payments API** | ⏳ لم يبدأ | الكيانات جاهزة (ServicePayment) |
| **Shipments API** | ⏳ لم يبدأ | الكيانات جاهزة (ServiceShipment) |
| **Invoices API** | ⏳ لم يبدأ | الكيانات جاهزة (ServiceInvoice) |
| **Agreements API** | ⏳ لم يبدأ | الكيان جاهز (PharmacyCompanyAgreement) |
| **Stock API** | ⏳ لم يبدأ | الكيان جاهز (PharmaCompanyMedicationStock) |
| **Status History** | ⏳ لم يبدأ | الكيان جاهز (PharmacySupplyOrderStatusHistory) |

### 🔴 ما تم تأجيله (Deferred)

- **Warehouse Layer** (FulfillmentMode = WAREHOUSE)
- **B2B Points Integration** (Seed Data جاهز في الكود لكن غير مطبق)
- **Pharmacy Stock Page** (بعد استلام الطلبات)
- **Online Payments** (Paymob/Fawry)
- **Campaigns Auto-Apply in Orders** (تم بناء CRUD فقط، لم يُطبق تلقائياً في `SupplyOrderService`)
  ---
  
## 🏗️ الجزء الأول: البنية المعمارية

### 1.1 المبادئ الأساسية (Feature-as-a-Service)

**مبدأ المنصة:** كل ميزة تُصمم كـ **"Engine"** مستقل، والوحدات الإنتاجية تُنشئ **"Connections"** فقط.

**تطبيق على Commerce:**

| الميزة | Engine | مثال استخدام |
|---|---|---|
| **Orders** | `ServiceOrders` + `ServiceOrderItems` | B2B SupplyOrder, Clinic Booking, Retail Order |
| **Payments** | `ServicePayments` | دفع SupplyOrder, دفع Booking, دفع Retail |
| **Ledger** | `ServiceLedger` + `LedgerTransactions` | رصيد صيدلية-شركة, رصيد مريض-عيادة |
| **Shipments** | `ServiceShipments` | شحن SupplyOrder, توصيل Booking |
| **Invoices** | `ServiceInvoices` | فاتورة SupplyOrder, فاتورة Booking |
| **Returns** | `ServiceReturns` (مؤجل) | مرتجع SupplyOrder, إلغاء Booking |

**النتيجة:** كل هذه الكيانات **عامة** — تستخدم `SourceEntityType` + `SourceEntityID` للربط بأي وحدة إنتاجية.

### 1.2 Engine Layer + Module Layer

```
┌─────────────────────────────────────────────────────────┐
│                    MODULE LAYER                          │
│  ┌──────────────┐  ┌──────────────┐  ┌──────────────┐  │
│  │  B2B Supply  │  │  Clinic OS   │  │  Retail      │  │
│  │  (اليوم)     │  │  (لاحقاً)    │  │  (لاحقاً)    │  │
│  └──────┬───────┘  └──────┬───────┘  └──────┬───────┘  │
└─────────┼─────────────────┼─────────────────┼──────────┘
          │                 │                 │
          ▼                 ▼                 ▼
┌─────────────────────────────────────────────────────────┐
│                    ENGINE LAYER                          │
│  ┌──────────────┐  ┌──────────────┐  ┌──────────────┐  │
│  │ ServiceOrders│  │ServicePayments│ │ ServiceLedger│  │
│  └──────────────┘  └──────────────┘  └──────────────┘  │
│  ┌──────────────┐  ┌──────────────┐  ┌──────────────┐  │
│  │ServiceShipments││ServiceInvoices│ │ServiceReturns│  │
│  └──────────────┘  └──────────────┘  └──────────────┘  │
└─────────────────────────────────────────────────────────┘
          │
          ▼
┌─────────────────────────────────────────────────────────┐
│                    SHARED LAYER                          │
│  ┌──────────────┐  ┌──────────────┐  ┌──────────────┐  │
│  │ Organizations│  │  UserProfiles │ │  Payments     │  │
│  │              │  │              │  │  Methods      │  │
│  └──────────────┘  └──────────────┘  └──────────────┘  │
└─────────────────────────────────────────────────────────┘
```

### 1.3 `SourceEntityType` — القيم المعتمدة

**كل كيان في Engine Layer يستخدم `SourceEntityType` (nvarchar) + `SourceEntityID` (int):**

| `SourceEntityType` | `SourceEntityID` يُشير إلى | الاستخدام |
|---|---|---|
| `B2B_SUPPLY_ORDER` | `PharmacySupplyOrder.SupplyOrderID` | B2B (اليوم) |
| `CLINIC_BOOKING` | `ClinicBooking.BookingID` | Clinic OS (لاحقاً) |
| `PHARMACY_RETAIL_ORDER` | `RetailOrder.OrderID` | Retail (لاحقاً) |
| `LAB_TEST_ORDER` | `LabTestOrder.OrderID` | Labs (لاحقاً) |
| `PSP_ENROLLMENT` | `PSPParticipation.ParticipationID` | PSP (لاحقاً — دفع اشتراك) |

**المرونة:** أي وحدة جديدة تُضيف قيمتها إلى `SourceEntityType` — بدون تعديل بنية.

### 1.4 العلاقة بـ `MedicationRequest` (الموجود)

**القرار:** **لا نُعيد كتابة `MedicationRequest`**. يبقى كما هو (مريض ← صيدلية regular).

**السبب:**
- يعمل، مستقر، لا سبب لكسره.
- إعادة كتابته ستستغرق وقتاً، وتُخاطر بـ Regression bugs.

**لاحقاً (عند الحاجة):** يمكن ترحيل `MedicationRequest` إلى `ServiceOrder` (SourceEntityType = `PHARMACY_RETAIL_ORDER`) في Migration منفصل، بدون كسر الواجهة.

**الآن:** `MedicationRequest` يبقى، و`ServiceOrder` يُستخدم للـ B2B فقط.

### 1.5 العلاقة بـ `PSPDispensation` (الموجود)

**القرار:** **لا نلمس `PSPDispensation`**. يبقى كما هو (صرف PSP).

**السبب:**
- هو جزء من دورة PSP، وليس Commerce.
- يستخدم `PSPProgramPointScheme` (نظام نقاط PSP).

**التكامل المستقبلي (اختياري):** عند نمو المنصة، يمكن إنشاء `ServiceOrder` لكل `PSPDispensation` (SourceEntityType = `PSP_DISPENSATION`) لتحليل موحد. **مؤجل.**

### 1.6 العلاقة بنظام النقاط (B2B_SUPPLY)

**`PointSystemTypeID = 2 (B2B_SUPPLY)`** موجود في `06-POINTS`، ومهيأ لاستقبال حملات B2B.

**التكامل:**
- `PharmacySupplyOrder` = Campaign → `SourceEntityID = SupplyOrder.SupplyOrderID` + `PointSystemTypeID = 2`.
- عند تسليم الطلب (`DELIVERED`) → استدعاء `IEarnPointsOrchestrator.AwardByCampaignAsync` (نفس نمط `DispenseController`).
- النقاط للمستفيد: الصيدلية (`BeneficiaryTypeID = 1`) أو الموظف (`BeneficiaryTypeID = 2`).

**التفاصيل الكاملة:** الجزء 5.

### 1.7 العلاقة بـ `PSPParticipation` (الموجود)

**`PSPParticipation`** يربط الصيدلية بالبرنامج (شركة ← صيدلية). هذا **ليس** عقد B2B.

**الفرق:**

| `PSPParticipation` | `PharmacyCompanyAgreement` (جديد) |
|---|---|
| شركة → صيدلية (برنامج دعم) | شركة → صيدلية (توريد) |
| للمرضى (PSP) | للصيدلية نفسها |
| `ProgramID` | `CompanyOrgId` + `PharmacyOrgId` |
| يُنشأ عبر `PSPInvitation` | يُنشأ عبر `ServiceOrder` الأول |

**التكامل:**
- `PSPParticipation` موجود → يستخدمه `DispenseController` (صرف PSP).
- `PharmacyCompanyAgreement` جديد → يستخدمه `SupplyOrdersController` (توريد B2B).

**السؤال المطروح في [#20]:** "هل نحتاج `PharmacyCompanyAgreement`؟"

**الإجابة:** **نعم** — لأن:
- `PSPParticipation` محدد بـ `ProgramID` (واحد لكل برنامج).
- B2B يحتاج عقد تجاري بين الشركة والصيدلية (شروط، خصم، حد ائتمان) — مستقل عن أي برنامج.

**التفاصيل:** الجزء 2 (Entities).

### 1.8 مسار الترقية (Migration Path)

**المرحلة 1 (هذه الجلسة — B2B Direct):**
- 6 كيانات Engine + 2 Module.
- `FulfillmentMode = DIRECT_COMPANY` (الشركة تشحن مباشرة).
- `PharmacyCompanyAgreement` (شركة-صيدلية).
- `PharmaCompanyMedicationStock` (مخزون + سعر جملة).
- Ledger (صيدلية ↔ شركة).

**المرحلة 2 (لاحقاً — Warehouse Layer):**
- `WarehouseProfile` (مخزن + نطاق جغرافي).
- `PharmacySupplyOrder.WarehouseOrgId` (nullable).
- `FulfillmentMode = WAREHOUSE`.
- `WarehouseCommission` (رسوم لوجستية).

**المرحلة 3 (لاحقاً — Loyalty + Marketing):**
- `PharmacyLoyaltyPoint` (منفصل عن PSPProgramPointScheme).
- حملات تسويقية للصيدليات.

**المرحلة 4 (لاحقاً — Clinic OS):**
- `ClinicBooking` (SourceEntityType = `CLINIC_BOOKING`).
- يستخدم نفس Engine Layer.

**المرحلة 5 (لاحقاً — Retail):**
- `RetailOrder` (SourceEntityType = `PHARMACY_RETAIL_ORDER`).
- يستخدم نفس Engine Layer.

**الفلسفة:** كل مرحلة تُضيف Module Layer فقط، دون تعديل Engine Layer. **هذا هو جمال التصميم.**

### 1.9 المكونات التفصيلية (ملخص)

| المكوّن | النوع | الموقع | الحالة |
|---|---|---|---|
| `ServiceOrders` | Engine | `RubikCare.Domain/Commerce/` | جديد |
| `ServiceOrderItems` | Engine | نفس | جديد |
| `ServicePayments` | Engine | نفس | جديد |
| `ServiceLedger` | Engine | نفس | جديد |
| `LedgerTransactions` | Engine | نفس | جديد |
| `ServiceShipments` | Engine | نفس | جديد |
| `ServiceInvoices` | Engine | نفس | جديد |
| `PharmacySupplyOrder` | Module | `RubikCare.Domain/B2B/` | جديد |
| `PharmacySupplyOrderItem` | Module | نفس | جديد |
| `PharmacySupplyOrderStatusHistory` | Module | نفس | جديد |
| `PharmacyCompanyAgreement` | Module | نفس | جديد |
| `PharmaCompanyMedicationStock` | Module | نفس | جديد |
| `PaymentMethods` | Shared | موجود | استخدام |
| `PointSystemTypes` | Shared | موجود | B2B_SUPPLY = 2 |
| `PointSystemTypeOrganizationTypes` | Shared | موجود | إضافة Seed |

**التفاصيل الكاملة لكل كيان:** الجزء 2.

---


## 🗄️ الجزء الثاني: الكيانات (Entities)

### 2.0 هيكل المجلدات

```
### 2.0 هيكل المجلدات (Implementation Reality)

RubikCare.Domain/
└── Commerce/
    ├── Engine/
    │   ├── ServiceOrder.cs                       ✅ (migration 20261107123244)
    │   ├── ServiceOrderItem.cs                   ✅ (migration 20261107123244)
    │   ├── ServiceLedger.cs                      ✅ (migration 20261107123244)
    │   ├── LedgerTransaction.cs                  ✅ (migration 20261107123244)
    │   ├── OrganizationSubscription.cs           ✅ (migration 20261107123244)
    │   └── Enums/
    │       ├── ServiceOrderStatus.cs             ✅
    │       ├── FulfillmentMode.cs                ✅
    │
    └── B2B/
        ├── PharmacySupplyOrder.cs                ✅ (migration 20261107123244)
        ├── PharmacySupplyOrderItem.cs            ✅ (migration 20261107123244)
        ├── SupplierIncentiveTier.cs              ✅ (migration 20261109090826)
        ├── PharmacyTierProgress.cs               ✅ (migration 20261109090826)
        ├── ProductBundle.cs                      ✅ (migration 20261109090826 + AddBundleDescription)
        ├── ProductBundleItem.cs                  ✅ (migration 20261109090826)
        ├── CommerceCampaign.cs                   ✅ (migration 20261110133611)
        ├── ProductBonus.cs                       ✅ (migration AddBundlesBonusesAndLogistics + UpdateBonusWithTiers)
        ├── ProductBonusTier.cs                   ✅ (migration UpdateBonusWithTiers)
        ├── LogisticsProvider.cs                  ✅ (migration AddBundlesBonusesAndLogistics)
        │
        │   ⏳ غير منفذة بعد:
        ├── PharmacyCompanyAgreement.cs           ❌
        ├── PharmaCompanyMedicationStock.cs       ❌
        └── PharmacySupplyOrderStatusHistory.cs   ❌
```
ملاحظة: أضفنا ProductBonusTier (بعد إعادة تصميم البونص ليدعم شرائح متعددة)، و BundleDiscountPercent (للباقات).


**الفلسفة:** `Engine/` = كيانات عامة (تستخدمها أي وحدة). `B2B/` = كيانات خاصة بـ B2B. عند إضافة `Clinic/` و`Retail/` لاحقاً، تُضاف كمجلدات شقيقة لـ `B2B/`.

### 2.1 `ServiceOrder` (Engine — Order Header)

**الغرض:** رأس الطلب — أي طلب في أي وحدة.

**الحقول:**
```csharp
[Table("ServiceOrders")]
public class ServiceOrder
{
    [Key]
    public int ServiceOrderID { get; set; }
    
    // ─── التصنيف (SourceEntityType) ───
    [Required]
    [MaxLength(50)]
    public string SourceEntityType { get; set; } = string.Empty;  // "B2B_SUPPLY_ORDER"
    
    [Required]
    [MaxLength(50)]
    public string OrderNumber { get; set; } = string.Empty;  // "ORD-20261107-0001" (unique)
    
    // ─── الأطراف ───
    [Required]
    [MaxLength(50)]
    public string OrderingPartyType { get; set; } = "ORGANIZATION";  // "ORGANIZATION" | "USER"
    
    [Required]
    public int OrderingPartyId { get; set; }  // PharmacyOrgId
    
    [Required]
    [MaxLength(50)]
    public string FulfillingPartyType { get; set; } = "ORGANIZATION";  // "ORGANIZATION" | "USER"
    
    [Required]
    public int FulfillingPartyId { get; set; }  // CompanyOrgId
    
    // ─── الحالة ───
    [Required]
    [MaxLength(30)]
    public string Status { get; set; } = "DRAFT";  // DRAFT|SUBMITTED|CONFIRMED|PREPARING|SHIPPED|DELIVERED|COMPLETED|CANCELLED|REJECTED
    
    // ─── المبالغ ───
    [Column(TypeName = "decimal(18,2)")]
    public decimal SubTotal { get; set; }
    
    [Column(TypeName = "decimal(18,2)")]
    public decimal TaxAmount { get; set; }
    
    [Column(TypeName = "decimal(18,2)")]
    public decimal ShippingAmount { get; set; }
    
    [Column(TypeName = "decimal(18,2)")]
    public decimal TotalAmount { get; set; }
    
    [MaxLength(3)]
    public string CurrencyCode { get; set; } = "EGP";
    
    // ─── التوصيل ───
    [MaxLength(30)]
    public string? FulfillmentMode { get; set; }  // "DIRECT_COMPANY" | "WAREHOUSE" | null
    
    public DateTime? ExpectedDeliveryDate { get; set; }
    
    public DateTime? ActualDeliveryDate { get; set; }
    
    // ─── Metadata ───
    public string? Notes { get; set; }
    
    public string? MetadataJson { get; set; }  // حقول إضافية حسب SourceEntityType (اختياري)
    
    // ─── Audit ───
    public int? CreatedBy { get; set; }
    public DateTime CreatedDate { get; set; } = DateTime.UtcNow;
    public int? LastModifiedBy { get; set; }
    public DateTime? LastModifiedDate { get; set; }
    
    // ─── Navigation ───
    public virtual ICollection<ServiceOrderItem> Items { get; set; } = new List<ServiceOrderItem>();
    public virtual ICollection<ServicePayment> Payments { get; set; } = new List<ServicePayment>();
    public virtual ICollection<ServiceShipment> Shipments { get; set; } = new List<ServiceShipment>();
    public virtual ICollection<ServiceInvoice> Invoices { get; set; } = new List<ServiceInvoice>();
}
```

**Indexes:**
- `IX_ServiceOrders_SourceEntityType` (nvarchar(50))
- `IX_ServiceOrders_OrderNumber` — UNIQUE.
- `IX_ServiceOrders_Status`
- `IX_ServiceOrders_OrderingPartyId` (pharmacy)
- `IX_ServiceOrders_FulfillingPartyId` (company)
- `IX_ServiceOrders_CreatedDate`

### 2.2 `ServiceOrderItem` (Engine — Order Line)

**الغرض:** بند في الطلب — أي منتج/خدمة.

**الحقول:**
```csharp
[Table("ServiceOrderItems")]
public class ServiceOrderItem
{
    [Key]
    public int ServiceOrderItemID { get; set; }
    
    [Required]
    public int ServiceOrderID { get; set; }
    public virtual ServiceOrder ServiceOrder { get; set; } = null!;
    
    // ─── التصنيف ───
    [Required]
    [MaxLength(50)]
    public string ItemType { get; set; } = "MEDICATION";  // "MEDICATION" | "SERVICE" | "LAB_TEST"
    
    [Required]
    public int ItemReferenceID { get; set; }  // PharmaCompanyMedicationID | ServiceProductID
    
    // ─── الكمية ───
    [Required]
    public int Quantity { get; set; }
    
    [Column(TypeName = "decimal(18,2)")]
    public decimal UnitPrice { get; set; }
    
    [Column(TypeName = "decimal(5,2)")]
    public decimal DiscountPercent { get; set; }
    
    [Column(TypeName = "decimal(18,2)")]
    public decimal DiscountAmount { get; set; }
    
    [Column(TypeName = "decimal(18,2)")]
    public decimal LineTotal { get; set; }
    
    // ─── Fulfillment (لاحقاً — Warehouse) ───
    public int? FulfilledByOrgId { get; set; }  // null = الشركة (DIRECT), X = Warehouse
    
    // ─── Metadata ───
    public string? Notes { get; set; }
    
    public string? MetadataJson { get; set; }  // خصائص حسب ItemType
    
    // ─── Audit ───
    public DateTime CreatedDate { get; set; } = DateTime.UtcNow;
}
```

**Indexes:**
- `IX_ServiceOrderItems_ServiceOrderID`
- `IX_ServiceOrderItems_ItemType`
- `IX_ServiceOrderItems_ItemReferenceID`

### 2.3 `ServicePayment` (Engine — Payment)

**الغرض:** دفعة على طلب (قد تكون جزئية أو كاملة).

**الحقول:**
```csharp
[Table("ServicePayments")]
public class ServicePayment
{
    [Key]
    public int ServicePaymentID { get; set; }
    
    [Required]
    public int ServiceOrderID { get; set; }
    public virtual ServiceOrder ServiceOrder { get; set; } = null!;
    
    // ─── الدافع والمستلم ───
    [Required]
    [MaxLength(50)]
    public string PayerPartyType { get; set; } = "ORGANIZATION";
    
    [Required]
    public int PayerPartyId { get; set; }
    
    [Required]
    [MaxLength(50)]
    public string PayeePartyType { get; set; } = "ORGANIZATION";
    
    [Required]
    public int PayeePartyId { get; set; }
    
    // ─── المبلغ ───
    [Required]
    [Column(TypeName = "decimal(18,2)")]
    public decimal Amount { get; set; }
    
    [MaxLength(3)]
    public string CurrencyCode { get; set; } = "EGP";
    
    // ─── طريقة الدفع ───
    [Required]
    public int PaymentMethodID { get; set; }
    public virtual PaymentMethod PaymentMethod { get; set; } = null!;
    
    // ─── الحالة ───
    [Required]
    [MaxLength(30)]
    public string Status { get; set; } = "PENDING";  // PENDING|COMPLETED|FAILED|REFUNDED
    
    // ─── المرجع الخارجي ───
    [MaxLength(100)]
    public string? ExternalReference { get; set; }  // بوابة الدفع
    
    [MaxLength(100)]
    public string? IdempotencyKey { get; set; }  // لمنع الدفع المزدوج
    
    // ─── التواريخ ───
    public DateTime CreatedDate { get; set; } = DateTime.UtcNow;
    public DateTime? CompletedDate { get; set; }
    public DateTime? FailedDate { get; set; }
    
    // ─── Audit ───
    public int? CreatedBy { get; set; }
    public string? Notes { get; set; }
}
```

**Indexes:**
- `IX_ServicePayments_ServiceOrderID`
- `IX_ServicePayments_Status`
- `IX_ServicePayments_IdempotencyKey` — UNIQUE (nullable).

### 2.4 `ServiceLedger` (Engine — Ledger Account)

**الغرض:** حساب مالي بين طرفين (دائن/مدين).

**الحقول:**
```csharp
[Table("ServiceLedgers")]
public class ServiceLedger
{
    [Key]
    public int ServiceLedgerID { get; set; }
    
    // ─── الطرفان ───
    [Required]
    [MaxLength(50)]
    public string PartyAType { get; set; } = "ORGANIZATION";
    
    [Required]
    public int PartyAId { get; set; }
    
    [Required]
    [MaxLength(50)]
    public string PartyBType { get; set; } = "ORGANIZATION";
    
    [Required]
    public int PartyBId { get; set; }
    
    // ─── العملة ───
    [MaxLength(3)]
    public string CurrencyCode { get; set; } = "EGP";
    
    // ─── الرصيد ───
    [Column(TypeName = "decimal(18,2)")]
    public decimal Balance { get; set; }  // + = لصالح A، - = على A
    
    // ─── الحالة ───
    public bool IsActive { get; set; } = true;
    
    // ─── التواريخ ───
    public DateTime CreatedDate { get; set; } = DateTime.UtcNow;
    public DateTime? LastSettlementDate { get; set; }
    public DateTime? LastUpdatedDate { get; set; }
    
    // ─── Navigation ───
    public virtual ICollection<LedgerTransaction> Transactions { get; set; } = new List<LedgerTransaction>();
}
```

**Indexes:**
- `IX_ServiceLedgers_Parties` — UNIQUE (PartyAType, PartyAId, PartyBType, PartyBId, CurrencyCode).

**ملاحظة:** `PartyA` هو صاحب الرصيد (Balance + = له، - = عليه). `PartyB` هو الطرف الآخر.

### 2.5 `LedgerTransaction` (Engine — Ledger Entry)

**الغرض:** حركة في Ledger (Charge, Credit, Settlement, Adjustment).

**الحقول:**
```csharp
[Table("LedgerTransactions")]
public class LedgerTransaction
{
    [Key]
    public int LedgerTransactionID { get; set; }
    
    [Required]
    public int ServiceLedgerID { get; set; }
    public virtual ServiceLedger ServiceLedger { get; set; } = null!;
    
    // ─── النوع ───
    [Required]
    [MaxLength(30)]
    public string TransactionType { get; set; } = "CHARGE";  // CHARGE|CREDIT|SETTLEMENT|ADJUSTMENT
    
    // ─── المرجع ───
    [Required]
    [MaxLength(50)]
    public string SourceEntityType { get; set; } = string.Empty;  // "SUPPLY_ORDER" | "PSP_DISPENSATION" | "PAYMENT"
    
    [Required]
    public int SourceEntityID { get; set; }
    
    // ─── المبلغ ───
    [Required]
    [Column(TypeName = "decimal(18,2)")]
    public decimal Amount { get; set; }  // + = Credit (يزيد رصيد A), - = Charge (ينقص رصيد A)
    
    // ─── الرصيد قبل/بعد ───
    [Column(TypeName = "decimal(18,2)")]
    public decimal BalanceBefore { get; set; }
    
    [Column(TypeName = "decimal(18,2)")]
    public decimal BalanceAfter { get; set; }
    
    // ─── الوصف ───
    [MaxLength(500)]
    public string? Description { get; set; }
    
    // ─── Audit ───
    public int? CreatedBy { get; set; }
    public DateTime CreatedDate { get; set; } = DateTime.UtcNow;
}
```

**Indexes:**
- `IX_LedgerTransactions_ServiceLedgerID`
- `IX_LedgerTransactions_TransactionType`
- `IX_LedgerTransactions_SourceEntityType_SourceEntityID` — composite.

### 2.6 `ServiceShipment` (Engine — Shipment)

**الغرض:** شحنة على طلب.

**الحقول:**
```csharp
[Table("ServiceShipments")]
public class ServiceShipment
{
    [Key]
    public int ServiceShipmentID { get; set; }
    
    [Required]
    public int ServiceOrderID { get; set; }
    public virtual ServiceOrder ServiceOrder { get; set; } = null!;
    
    // ─── الناقل ───
    [Required]
    [MaxLength(50)]
    public string CarrierType { get; set; } = "DIRECT";  // DIRECT|THIRD_PARTY|WAREHOUSE
    
    public int? CarrierOrgId { get; set; }  // org للناقل
    
    [MaxLength(100)]
    public string? TrackingNumber { get; set; }
    
    // ─── الحالة ───
    [Required]
    [MaxLength(30)]
    public string Status { get; set; } = "PENDING";  // PENDING|PICKED|SHIPPED|IN_TRANSIT|DELIVERED|FAILED
    
    // ─── التواريخ ───
    public DateTime? ShippedDate { get; set; }
    public DateTime? DeliveredDate { get; set; }
    
    // ─── Metadata ───
    public string? Notes { get; set; }
    public string? MetadataJson { get; set; }
    
    // ─── Audit ───
    public DateTime CreatedDate { get; set; } = DateTime.UtcNow;
    public int? CreatedBy { get; set; }
}
```

**Indexes:**
- `IX_ServiceShipments_ServiceOrderID`
- `IX_ServiceShipments_Status`
- `IX_ServiceShipments_TrackingNumber`

### 2.7 `ServiceInvoice` (Engine — Invoice)

**الغرض:** فاتورة على طلب.

**الحقول:**
```csharp
[Table("ServiceInvoices")]
public class ServiceInvoice
{
    [Key]
    public int ServiceInvoiceID { get; set; }
    
    [Required]
    public int ServiceOrderID { get; set; }
    public virtual ServiceOrder ServiceOrder { get; set; } = null!;
    
    // ─── رقم الفاتورة ───
    [Required]
    [MaxLength(50)]
    public string InvoiceNumber { get; set; } = string.Empty;  // "INV-20261107-0001" (unique)
    
    // ─── المُصدر والمستلم ───
    [Required]
    [MaxLength(50)]
    public string IssuerPartyType { get; set; } = "ORGANIZATION";
    
    [Required]
    public int IssuerPartyId { get; set; }
    
    [Required]
    [MaxLength(50)]
    public string ReceiverPartyType { get; set; } = "ORGANIZATION";
    
    [Required]
    public int ReceiverPartyId { get; set; }
    
    // ─── المبالغ ───
    [Column(TypeName = "decimal(18,2)")]
    public decimal SubTotal { get; set; }
    
    [Column(TypeName = "decimal(18,2)")]
    public decimal TaxAmount { get; set; }
    
    [Column(TypeName = "decimal(18,2)")]
    public decimal TotalAmount { get; set; }
    
    [MaxLength(3)]
    public string CurrencyCode { get; set; } = "EGP";
    
    // ─── الحالة ───
    [Required]
    [MaxLength(30)]
    public string Status { get; set; } = "DRAFT";  // DRAFT|ISSUED|PAID|VOID|REFUNDED
    
    // ─── التواريخ ───
    public DateTime IssuedDate { get; set; } = DateTime.UtcNow;
    public DateTime? DueDate { get; set; }
    public DateTime? PaidDate { get; set; }
    
    // ─── Metadata ───
    public string? Notes { get; set; }
    public string? MetadataJson { get; set; }
    
    // ─── Audit ───
    public DateTime CreatedDate { get; set; } = DateTime.UtcNow;
    public int? CreatedBy { get; set; }
}
```

**Indexes:**
- `IX_ServiceInvoices_ServiceOrderID`
- `IX_ServiceInvoices_InvoiceNumber` — UNIQUE.
- `IX_ServiceInvoices_Status`

### 2.8 `PharmacySupplyOrder` (Module B2B — Header)

**الغرض:** رأس طلب توريد B2B — يرتبط بـ `ServiceOrder` عبر `SourceEntityType = "B2B_SUPPLY_ORDER"`.

**الفلسفة:** `PharmacySupplyOrder` = بيانات خاصة بـ B2B. `ServiceOrder` = بيانات عامة (الحالة، المبالغ، إلخ). **مترابطان عبر `ServiceOrderID`.**

**الحقول:**
```csharp
[Table("PharmacySupplyOrders")]
public class PharmacySupplyOrder
{
    [Key]
    public int SupplyOrderID { get; set; }
    
    // ─── الربط بـ ServiceOrder ───
    [Required]
    public int ServiceOrderID { get; set; }
    public virtual ServiceOrder ServiceOrder { get; set; } = null!;
    
    // ─── الأطراف ───
    [Required]
    public int PharmacyOrgId { get; set; }
    public virtual Organization PharmacyOrg { get; set; } = null!;
    
    [Required]
    public int CompanyOrgId { get; set; }
    public virtual Organization CompanyOrg { get; set; } = null!;
    
    // ─── العقد ───
    public int? AgreementID { get; set; }
    public virtual PharmacyCompanyAgreement? Agreement { get; set; }
    
    // ─── الحالة (مكررة عن ServiceOrder للسرعة) ───
    [Required]
    [MaxLength(30)]
    public string Status { get; set; } = "DRAFT";
    
    // ─── Fulfillment ───
    [MaxLength(30)]
    public string FulfillmentMode { get; set; } = "DIRECT_COMPANY";  // DIRECT_COMPANY|WAREHOUSE
    
    public int? WarehouseOrgId { get; set; }  // لاحقاً
    
    // ─── Metadata ───
    public string? Notes { get; set; }
    
    // ─── Audit ───
    public int? CreatedBy { get; set; }
    public DateTime CreatedDate { get; set; } = DateTime.UtcNow;
    public int? LastModifiedBy { get; set; }
    public DateTime? LastModifiedDate { get; set; }
    
    // ─── Navigation ───
    public virtual ICollection<PharmacySupplyOrderItem> Items { get; set; } = new List<PharmacySupplyOrderItem>();
    public virtual ICollection<PharmacySupplyOrderStatusHistory> StatusHistory { get; set; } = new List<PharmacySupplyOrderStatusHistory>();
}
```

**Indexes:**
- `IX_PharmacySupplyOrders_ServiceOrderID` — UNIQUE (one-to-one مع ServiceOrder).
- `IX_PharmacySupplyOrders_PharmacyOrgId`
- `IX_PharmacySupplyOrders_CompanyOrgId`
- `IX_PharmacySupplyOrders_Status`
- `IX_PharmacySupplyOrders_CreatedDate`

### 2.9 `PharmacySupplyOrderItem` (Module B2B — Line)

**الغرض:** بند في طلب التوريد — يرتبط بـ `ServiceOrderItem`.

**الحقول:**
```csharp
[Table("PharmacySupplyOrderItems")]
public class PharmacySupplyOrderItem
{
    [Key]
    public int SupplyOrderItemID { get; set; }
    
    // ─── الربط بـ ServiceOrderItem ───
    [Required]
    public int ServiceOrderItemID { get; set; }
    public virtual ServiceOrderItem ServiceOrderItem { get; set; } = null!;
    
    [Required]
    public int SupplyOrderID { get; set; }
    public virtual PharmacySupplyOrder SupplyOrder { get; set; } = null!;
    
    // ─── المنتج ───
    [Required]
    public int PharmaCompanyMedicationID { get; set; }
    public virtual PharmaCompanyMedication Medication { get; set; } = null!;
    
    // ─── الكمية (مكررة عن ServiceOrderItem للسرعة) ───
    [Required]
    public int Quantity { get; set; }
    
    // ─── Metadata ───
    public string? Notes { get; set; }
    
    // ─── Audit ───
    public DateTime CreatedDate { get; set; } = DateTime.UtcNow;
}
```

**Indexes:**
- `IX_PharmacySupplyOrderItems_SupplyOrderID`
- `IX_PharmacySupplyOrderItems_PharmaCompanyMedicationID`
- `IX_PharmacySupplyOrderItems_ServiceOrderItemID` — UNIQUE (one-to-one).

### 2.10 `PharmacySupplyOrderStatusHistory` (Module B2B — Audit)

**الغرض:** سجل تغييرات الحالة (State Machine Audit).

**الحقول:**
```csharp
[Table("PharmacySupplyOrderStatusHistory")]
public class PharmacySupplyOrderStatusHistory
{
    [Key]
    public int StatusHistoryID { get; set; }
    
    [Required]
    public int SupplyOrderID { get; set; }
    public virtual PharmacySupplyOrder SupplyOrder { get; set; } = null!;
    
    // ─── الحالة ───
    [Required]
    [MaxLength(30)]
    public string FromStatus { get; set; } = string.Empty;
    
    [Required]
    [MaxLength(30)]
    public string ToStatus { get; set; } = string.Empty;
    
    // ─── من غيّر؟ ───
    public int? ChangedByUserId { get; set; }
    public virtual UserProfile? ChangedByUser { get; set; }
    
    // ─── لماذا؟ ───
    [MaxLength(500)]
    public string? Reason { get; set; }
    
    // ─── متى؟ ───
    public DateTime ChangedDate { get; set; } = DateTime.UtcNow;
    
    // ─── Metadata ───
    public string? MetadataJson { get; set; }
}
```

**Indexes:**
- `IX_PharmacySupplyOrderStatusHistory_SupplyOrderID`
- `IX_PharmacySupplyOrderStatusHistory_ChangedDate`

### 2.11 `PharmacyCompanyAgreement` (Module B2B — Contract)

**الغرض:** عقد توريد بين صيدلية وشركة أدوية.

**الحقول:**
```csharp
[Table("PharmacyCompanyAgreements")]
public class PharmacyCompanyAgreement
{
    [Key]
    public int AgreementID { get; set; }
    
    // ─── الأطراف ───
    [Required]
    public int PharmacyOrgId { get; set; }
    public virtual Organization PharmacyOrg { get; set; } = null!;
    
    [Required]
    public int CompanyOrgId { get; set; }
    public virtual Organization CompanyOrg { get; set; } = null!;
    
    // ─── الشروط ───
    [Column(TypeName = "decimal(5,2)")]
    public decimal DefaultDiscountPercent { get; set; } = 0;
    
    [Column(TypeName = "decimal(18,2)")]
    public decimal? CreditLimit { get; set; }  // حد الائتمان
    
    [Required]
    public int PaymentTermsDays { get; set; } = 0;  // 0 = فوري
    
    // ─── الحالة ───
    [Required]
    [MaxLength(30)]
    public string Status { get; set; } = "ACTIVE";  // ACTIVE|SUSPENDED|EXPIRED
    
    // ─── التواريخ ───
    public DateTime StartDate { get; set; } = DateTime.UtcNow;
    public DateTime? EndDate { get; set; }
    
    // ─── Metadata ───
    public string? Notes { get; set; }
    
    // ─── Audit ───
    public int? CreatedBy { get; set; }
    public DateTime CreatedDate { get; set; } = DateTime.UtcNow;
    public int? LastModifiedBy { get; set; }
    public DateTime? LastModifiedDate { get; set; }
}
```

**Indexes:**
- `IX_PharmacyCompanyAgreements_PharmacyOrgId`
- `IX_PharmacyCompanyAgreements_CompanyOrgId`
- `IX_PharmacyCompanyAgreements_Status`
- `IX_PharmacyCompanyAgreements_Unique` — UNIQUE (PharmacyOrgId, CompanyOrgId, Status) WHERE Status = 'ACTIVE' (Filtered Unique).

### 2.12 `PharmaCompanyMedicationStock` (Module B2B — Stock)

**الغرض:** مخزون شركة الأدوية من كل دواء + سعر الجملة.

**الحقول:**
```csharp
[Table("PharmaCompanyMedicationStock")]
public class PharmaCompanyMedicationStock
{
    [Key]
    public int StockID { get; set; }
    
    // ─── الربط بالدواء ───
    [Required]
    public int PharmaCompanyMedicationID { get; set; }
    public virtual PharmaCompanyMedication Medication { get; set; } = null!;
    
    [Required]
    public int CompanyOrgId { get; set; }  // للتحقق السريع
    public virtual Organization CompanyOrg { get; set; } = null!;
    
    // ─── المخزون ───
    [Required]
    public int AvailableQuantity { get; set; } = 0;
    
    [Required]
    public int ReservedQuantity { get; set; } = 0;  // محجوز لأوامر مؤكدة لم تُشحن
    
    public int? ReorderLevel { get; set; }  // حد التنبيه
    
    // ─── سعر الجملة ───
    [Column(TypeName = "decimal(18,2)")]
    public decimal? WholesalePrice { get; set; }  // سعر الصيدلية (مستقل عن Price للجمهور)
    
    // ─── حدود الطلب ───
    public int? MinOrderQuantity { get; set; }
    public int? MaxOrderQuantity { get; set; }
    
    // ─── الحالة ───
    public bool IsActive { get; set; } = true;
    
    // ─── التواريخ ───
    public DateTime? LastRestockDate { get; set; }
    public DateTime? LastUpdatedDate { get; set; }
    
    // ─── Concurrency ───
    [Timestamp]
    public byte[]? RowVersion { get; set; }  // للـ Concurrency الآمن
}
```

**Indexes:**
- `IX_PharmaCompanyMedicationStock_PharmaCompanyMedicationID` — UNIQUE.
- `IX_PharmaCompanyMedicationStock_CompanyOrgId`
- `IX_PharmaCompanyMedicationStock_IsActive`

---

## 📊 ملخص الكيانات الـ 12

| # | الكيان | الطبقة | الغرض |
|---|---|---|---|
| 1 | `ServiceOrder` | Engine | رأس الطلب (عام) |
| 2 | `ServiceOrderItem` | Engine | بند الطلب (عام) |
| 3 | `ServicePayment` | Engine | دفعة على طلب |
| 4 | `ServiceLedger` | Engine | حساب مالي بين طرفين |
| 5 | `LedgerTransaction` | Engine | حركة في Ledger |
| 6 | `ServiceShipment` | Engine | شحنة على طلب |
| 7 | `ServiceInvoice` | Engine | فاتورة على طلب |
| 8 | `PharmacySupplyOrder` | Module B2B | رأس طلب توريد |
| 9 | `PharmacySupplyOrderItem` | Module B2B | بند في طلب التوريد |
| 10 | `PharmacySupplyOrderStatusHistory` | Module B2B | سجل تغييرات الحالة |
| 11 | `PharmacyCompanyAgreement` | Module B2B | عقد توريد |
| 12 | `PharmaCompanyMedicationStock` | Module B2B | مخزون + سعر جملة |

**المجموع:** 7 Engine + 5 B2B = 12 كيان.

---

## 🔗 العلاقات الرئيسية

```
Organization (Pharmacy) ←── PharmacySupplyOrder ──→ Organization (Company)
                                    │
                                    ▼
                            ServiceOrder (1:1)
                                    │
                    ┌───────────────┼───────────────┐
                    ▼               ▼               ▼
            ServiceOrderItem  ServicePayment  ServiceInvoice
                    │
                    ▼
            PharmacySupplyOrderItem ──→ PharmaCompanyMedication

Organization (Pharmacy) ←── PharmacyCompanyAgreement ──→ Organization (Company)
                                    │
                                    ▼
                          (يستخدم في PharmacySupplyOrder)

Organization (A) ←── ServiceLedger ──→ Organization (B)
                        │
                        ▼
                  LedgerTransaction
```

## 🎬 الجزء الثالث: State Machines + Business Rules

### 3.1 State Machine: `PharmacySupplyOrder`

**الرسم:**
```
                    ┌─────────┐
                    │  DRAFT  │ ← الصيدلي يُنشئ
                    └────┬────┘
                         │ submit
                         ▼
                    ┌─────────────┐
                    │  SUBMITTED  │ ← في انتظار الشركة
                    └────┬────────┘
                         │
              ┌──────────┼──────────┐
              │ confirm  │          │ reject
              ▼          │          ▼
        ┌──────────┐     │    ┌──────────┐
        │CONFIRMED │     │    │ REJECTED │ (نهائي)
        └────┬─────┘     │    └──────────┘
             │           │
             │ prepare   │
             ▼           │
        ┌──────────┐     │
        │PREPARING │     │
        └────┬─────┘     │
             │           │
             │ ship      │
             ▼           │
        ┌──────────┐     │
        │ SHIPPED  │     │
        └────┬─────┘     │
             │           │
             │ deliver   │
             ▼           │
        ┌───────────┐    │
        │ DELIVERED │    │ ← الصيدلي يؤكد الاستلام
        └────┬──────┘    │
             │           │
             │ complete  │
             ▼           │
        ┌──────────┐     │
        │COMPLETED │     │ (نهائي)
        └──────────┘     │
                         │
                    ┌────▼─────┐
                    │CANCELLED │ (نهائي)
                    └──────────┘
```

**الجدول الرسمي للتحولات:**

| From | To | من يستطيع؟ | شرط |
|---|---|---|---|
| `DRAFT` | `SUBMITTED` | الصيدلية (`CanManageFinance`) | لا يوجد |
| `DRAFT` | `CANCELLED` | الصيدلية | لا يوجد |
| `SUBMITTED` | `CONFIRMED` | الشركة (`CanManageFinance`) | يوجد مخزون كافٍ |
| `SUBMITTED` | `REJECTED` | الشركة | لا يوجد مخزون / سبب آخر |
| `SUBMITTED` | `CANCELLED` | الصيدلية | لا يوجد |
| `CONFIRMED` | `PREPARING` | الشركة | لا يوجد |
| `CONFIRMED` | `CANCELLED` | الشركة | رسوم إلغاء (إن وُجدت) |
| `PREPARING` | `SHIPPED` | الشركة | لا يوجد |
| `PREPARING` | `CANCELLED` | الشركة | رسوم إلغاء |
| `SHIPPED` | `DELIVERED` | الصيدلية (تأكيد استلام) | لا يوجد |
| `SHIPPED` | `CANCELLED` | — | لا يمكن بعد الشحن |
| `DELIVERED` | `COMPLETED` | النظام (auto) | بعد أيام `ReturnsWindow` |

**الأحداث (Events):**
- عند `SUBMITTED`: إشعار الشركة.
- عند `CONFIRMED`: إشعار الصيدلية + Ledger (CHARGE).
- عند `SHIPPED`: إشعار الصيدلية + Shipment.Status = SHIPPED.
- عند `DELIVERED`: إشعار الشركة + Ledger (CREDIT إن لم يُدفع).
- عند `COMPLETED`: منح نقاط B2B (Points Engine).

### 3.2 State Machine: `ServiceOrder`

**الحالات (عامة):** `DRAFT | SUBMITTED | CONFIRMED | PREPARING | SHIPPED | DELIVERED | COMPLETED | CANCELLED | REJECTED`.

**العلاقة بـ `PharmacySupplyOrder.Status`:**
- `ServiceOrder.Status` = مصدر الحقيقة العام.
- `PharmacySupplyOrder.Status` = مرآة (للسرعة في الاستعلام).

**التزامن:** عند أي تغيير في `ServiceOrder.Status`، يُحدَّث `PharmacySupplyOrder.Status` في نفس Transaction.

### 3.3 State Machine: `ServicePayment`

```
PENDING ──[confirm]──> COMPLETED ──[refund]──> REFUNDED
   │
   └──[fail]──> FAILED
```

**ملاحظة:** `PENDING` يُنشأ عند طلب الدفع. `COMPLETED` بعد تأكيد بوابة الدفع (أو تأكيد يدوي).

### 3.4 State Machine: `ServiceShipment`

```
PENDING ──[pick]──> PICKED ──[ship]──> SHIPPED ──[transit]──> IN_TRANSIT ──[deliver]──> DELIVERED
                                                                                      
                                                                                      
                                                                              FAILED ──[retry]──> PENDING
```

### 3.5 State Machine: `ServiceInvoice`

```
DRAFT ──[issue]──> ISSUED ──[pay]──> PAID
                       │
                       └──[void]──> VOID
                                              
PAID ──[refund]──> REFUNDED
```

### 3.6 Business Rules — `PharmacySupplyOrder`

#### Rule 1: العقد نشط
**الشرط:** `PharmacyCompanyAgreement.Status = "ACTIVE"` قبل إنشاء طلب.
**الخطأ:** `NO_ACTIVE_AGREEMENT` — "لا يوجد عقد نشط مع هذه الشركة."

#### Rule 2: المخزون كافٍ
**الشرط:** لكل بند، `PharmaCompanyMedicationStock.AvailableQuantity - ReservedQuantity >= Quantity`.
**الخطأ:** `INSUFFICIENT_STOCK` — "الكمية المتاحة من <medication> أقل من المطلوب."

#### Rule 3: حد الائتمان
**الشرط:** إذا `Agreement.CreditLimit.HasValue`، `(ServiceLedger.Balance + OrderTotal) <= CreditLimit` (Balance سالب = على الصيدلية).
**الخطأ:** `CREDIT_LIMIT_EXCEEDED` — "تجاوزت حد الائتمان."

**ملاحظة:** إذا `CreditLimit` = 0/null، يُسمح بحد لا نهائي.

#### Rule 4: حدود الطلب
**الشرط:** لكل بند، `MinOrderQuantity <= Quantity <= MaxOrderQuantity`.
**الخطأ:** `QUANTITY_OUT_OF_RANGE`.

#### Rule 5: صلاحية العضو
**الشرط:** `OrgMembership.CanManageFinance = true` (للصيدلية) — لإرسال الطلب.
**الخطأ:** `INSUFFICIENT_PERMISSIONS`.

#### Rule 6: لا حذف بعد SUBMITTED
**الشرط:** `ServiceOrder.Status = "DRAFT"` فقط يمكن حذفه.
**الخطأ:** `CANNOT_DELETE_AFTER_SUBMIT`.

#### Rule 7: رسوم الإلغاء
**الشرط:** إذا `CONFIRMED` أو `PREPARING`، إلغاء يُنشئ `ServicePayment` (refund) + `LedgerTransaction` (ADJUSTMENT).
**الخطأ:** لا شيء — يُنفّذ تلقائياً.

### 3.7 Business Rules — `ServiceLedger`

#### Rule 1: طرفان مختلفان
**الشرط:** `PartyAType != PartyBType` أو `PartyAId != PartyBId`.
**الخطأ:** `SAME_PARTIES`.

#### Rule 2: حساب فريد
**الشرط:** `UNIQUE (PartyAType, PartyAId, PartyBType, PartyBId, CurrencyCode)`.
**الخطأ:** `DUPLICATE_LEDGER`.

#### Rule 3: تحديث ذري
**الشرط:** `Balance` يُحدَّث في Transaction (`UPDLOCK, ROWLOCK`).
**الخطأ:** `CONCURRENCY_ERROR` — "الرصيد تغيّر، حاول مجدداً."

### 3.8 Business Rules — `PharmaCompanyMedicationStock`

#### Rule 1: حجز عند التأكيد
**الشرط:** عند `ServiceOrder` → `CONFIRMED`، `ReservedQuantity += OrderItem.Quantity` لكل بند.
**الخطأ:** `INSUFFICIENT_STOCK` (نفس Rule 2 أعلاه).

#### Rule 2: خصم عند الشحن
**الشرط:** عند `SHIPPED`، `AvailableQuantity -= Quantity` و `ReservedQuantity -= Quantity`.

#### Rule 3: تحرير الحجز عند الإلغاء
**الشرط:** عند `CANCELLED`، `ReservedQuantity -= Quantity` (بدون خصم `AvailableQuantity`).

#### Rule 4: Concurrency
**الشرط:** `[Timestamp] RowVersion` يمنع التعارض. أي تعديل يُحدّث `RowVersion`.

### 3.9 Business Rules — `PharmacyCompanyAgreement`

#### Rule 1: عقد نشط واحد
**الشرط:** `UNIQUE (PharmacyOrgId, CompanyOrgId)` WHERE `Status = 'ACTIVE'`.
**الخطأ:** `DUPLICATE_ACTIVE_AGREEMENT`.

#### Rule 2: حد الائتمان سالب = لا نهائي
**الشرط:** إذا `CreditLimit <= 0` → لا حد.
**الخطأ:** لا شيء.

#### Rule 3: EndDate > StartDate
**الشرط:** `EndDate > StartDate` (إن وُجد).
**الخطأ:** `INVALID_DATE_RANGE`.

---

## 🌐 الجزء الرابع: API Endpoints (Design)

### 4.1 Controllers المخططة

| Controller | الموقع | الغرض |
|---|---|---|
| `CatalogController` | `Api.Web/Controllers/Commerce/` | الصيدلي يتصفح كتالوج الشركة |
| `SupplyOrdersController` | نفسه | CRUD + State Transitions |
| `SupplyOrderPaymentsController` | نفسه | دفع + تأكيد |
| `SupplyOrderShipmentsController` | نفسه | شحن + تتبع |
| `SupplyOrderInvoicesController` | نفسه | فواتير |
| `LedgerController` | نفسه | رصيد + تاريخ |
| `CompanyStockController` | نفسه | إدارة المخزون (للشركة) |
| `CompanyAgreementsController` | نفسه | إدارة العقود (للشركة + الصيدلية) |

### 4.2 `CatalogController` (الصيدلي)

**الغرض:** الصيدلي يتصفح كتالوج الشركة (الأدوية المتاحة + السعر + المخزون).

| Method | Endpoint | الوصف |
|---|---|---|
| `GET` | `/api/commerce/catalog/{companyOrgId}` | كل الأدوية المتاحة للشركة |
| `GET` | `/api/commerce/catalog/{companyOrgId}/medications` | بحث بالاسم (pagination) |
| `GET` | `/api/commerce/catalog/{companyOrgId}/medications/{medicationId}` | تفاصيل دواء + سعر جملة + مخزون |
| `GET` | `/api/commerce/catalog/companies` | الشركات المتاحة (لها عقود نشطة) |

**Response Example:**
```json
{
  "companyOrgId": 74,
  "companyName": "BIOTECH EGYPT",
  "medications": [
    {
      "pharmaCompanyMedicationId": 123,
      "medicationNameAr": "باراسيتامول 500 مجم",
      "medicationNameEn": "Paracetamol 500mg",
      "wholesalePrice": 15.50,
      "availableQuantity": 1000,
      "minOrderQuantity": 10,
      "maxOrderQuantity": 500,
      "medicationImageUrl": "/uploads/medications/123.jpg"
    }
  ],
  "totalCount": 401,
  "page": 1,
  "pageSize": 30
}
```

**الصلاحيات:** `[Authorize]` + `CanManageFinance` (للصيدلية).

### 4.3 `SupplyOrdersController` (الصيدلي + الشركة)

| Method | Endpoint | الوصف | من؟ |
|---|---|---|---|
| `GET` | `/api/commerce/supply-orders` | قائمة الطلبات | الصيدلية + الشركة |
| `GET` | `/api/commerce/supply-orders/{id}` | تفاصيل طلب | الصيدلية + الشركة |
| `POST` | `/api/commerce/supply-orders` | إنشاء طلب (DRAFT) | الصيدلية |
| `PUT` | `/api/commerce/supply-orders/{id}` | تعديل طلب (DRAFT فقط) | الصيدلية |
| `POST` | `/api/commerce/supply-orders/{id}/submit` | إرسال (DRAFT → SUBMITTED) | الصيدلية |
| `POST` | `/api/commerce/supply-orders/{id}/confirm` | تأكيد (SUBMITTED → CONFIRMED) | الشركة |
| `POST` | `/api/commerce/supply-orders/{id}/reject` | رفض (SUBMITTED → REJECTED) | الشركة |
| `POST` | `/api/commerce/supply-orders/{id}/prepare` | تجهيز (CONFIRMED → PREPARING) | الشركة |
| `POST` | `/api/commerce/supply-orders/{id}/ship` | شحن (PREPARING → SHIPPED) | الشركة |
| `POST` | `/api/commerce/supply-orders/{id}/deliver` | تأكيد استلام (SHIPPED → DELIVERED) | الصيدلية |
| `POST` | `/api/commerce/supply-orders/{id}/complete` | إكمال (DELIVERED → COMPLETED) | النظام (auto) |
| `POST` | `/api/commerce/supply-orders/{id}/cancel` | إلغاء | الصيدلية / الشركة |
| `GET` | `/api/commerce/supply-orders/{id}/history` | سجل الحالات | الصيدلية + الشركة |

**Request Example — Create:**
```json
POST /api/commerce/supply-orders
{
  "companyOrgId": 74,
  "items": [
    {
      "pharmaCompanyMedicationId": 123,
      "quantity": 100
    },
    {
      "pharmaCompanyMedicationId": 456,
      "quantity": 50
    }
  ],
  "notes": "طلب توريد شهري"
}
```

**Response Example:**
```json
{
  "supplyOrderId": 1001,
  "serviceOrderId": 5001,
  "orderNumber": "ORD-20261107-0001",
  "status": "DRAFT",
  "pharmacyOrgId": 1258,
  "companyOrgId": 74,
  "subTotal": 2000.00,
  "taxAmount": 0,
  "shippingAmount": 0,
  "totalAmount": 2000.00,
  "currencyCode": "EGP",
  "fulfillmentMode": "DIRECT_COMPANY",
  "createdDate": "2026-11-07T10:00:00Z"
}
```

**الصلاحيات:**
- `POST` / `PUT` / `submit` / `cancel` — `CanManageFinance` (للصيدلية).
- `confirm` / `reject` / `prepare` / `ship` — `CanManageFinance` (للشركة).
- `deliver` — `CanManageFinance` (للصيدلية).

### 4.4 `SupplyOrderPaymentsController`

| Method | Endpoint | الوصف |
|---|---|---|
| `POST` | `/api/commerce/supply-orders/{id}/payments` | إنشاء دفعة (PENDING) |
| `POST` | `/api/commerce/supply-orders/{id}/payments/{paymentId}/confirm` | تأكيد الدفع (PENDING → COMPLETED) |
| `GET` | `/api/commerce/supply-orders/{id}/payments` | دفعات الطلب |
| `POST` | `/api/commerce/supply-orders/{id}/payments/{paymentId}/refund` | استرداد |

**Request Example — Create Payment:**
```json
POST /api/commerce/supply-orders/1001/payments
{
  "amount": 2000.00,
  "paymentMethodId": 1,  // CASH_ON_DELIVERY
  "externalReference": null
}
```

### 4.5 `SupplyOrderShipmentsController`

| Method | Endpoint | الوصف |
|---|---|---|
| `POST` | `/api/commerce/supply-orders/{id}/shipments` | إنشاء شحنة (PENDING) |
| `POST` | `/api/commerce/supply-orders/{id}/shipments/{shipmentId}/pick` | تم الاستلام من المخزن (PICKED) |
| `POST` | `/api/commerce/supply-orders/{id}/shipments/{shipmentId}/ship` | تم الشحن (SHIPPED) |
| `POST` | `/api/commerce/supply-orders/{id}/shipments/{shipmentId}/transit` | في الطريق (IN_TRANSIT) |
| `POST` | `/api/commerce/supply-orders/{id}/shipments/{shipmentId}/deliver` | تم التوصيل (DELIVERED) |
| `GET` | `/api/commerce/supply-orders/{id}/shipments` | شحنات الطلب |

### 4.6 `SupplyOrderInvoicesController`

| Method | Endpoint | الوصف |
|---|---|---|
| `POST` | `/api/commerce/supply-orders/{id}/invoices` | إنشاء فاتورة (DRAFT) |
| `POST` | `/api/commerce/supply-orders/{id}/invoices/{invoiceId}/issue` | إصدار (ISSUED) |
| `POST` | `/api/commerce/supply-orders/{id}/invoices/{invoiceId}/pay` | دفع (PAID) |
| `GET` | `/api/commerce/supply-orders/{id}/invoices` | فواتير الطلب |

### 4.7 `LedgerController`

| Method | Endpoint | الوصف |
|---|---|---|
| `GET` | `/api/commerce/ledger/{partyType}/{partyId}` | أرصدة الطرف (مع كل الأطراف الأخرى) |
| `GET` | `/api/commerce/ledger/{partyType}/{partyId}/{otherPartyType}/{otherPartyId}` | تفاصيل حساب محدد |
| `GET` | `/api/commerce/ledger/{partyType}/{partyId}/{otherPartyType}/{otherPartyId}/transactions` | حركات الحساب |
| `POST` | `/api/commerce/ledger/settle` | تسوية (settlement) |

**Response Example — Balances:**
```json
GET /api/commerce/ledger/ORGANIZATION/1258
{
  "partyType": "ORGANIZATION",
  "partyId": 1258,
  "partyName": "صيدلية روبيك كير",
  "balances": [
    {
      "otherPartyType": "ORGANIZATION",
      "otherPartyId": 74,
      "otherPartyName": "BIOTECH EGYPT",
      "balance": -1500.00,  // على الصيدلية 1500
      "currencyCode": "EGP",
      "lastSettlementDate": null
    },
    {
      "otherPartyType": "ORGANIZATION",
      "otherPartyId": 129,
      "otherPartyName": "EVA PHARMA",
      "balance": 300.00,  // للصيدلية 300
      "currencyCode": "EGP",
      "lastSettlementDate": "2026-10-15"
    }
  ],
  "totalReceivable": 300.00,  // مستحق للصيدلية
  "totalPayable": 1500.00      // مستحق على الصيدلية
}
```

### 4.8 `CompanyStockController` (الشركة)

| Method | Endpoint | الوصف |
|---|---|---|
| `GET` | `/api/commerce/company/{companyOrgId}/stock` | كل مخزون الشركة |
| `GET` | `/api/commerce/company/{companyOrgId}/stock/{medicationId}` | تفاصيل مخزون دواء |
| `PUT` | `/api/commerce/company/{companyOrgId}/stock/{medicationId}` | تعديل مخزون (كمية + سعر) |
| `POST` | `/api/commerce/company/{companyOrgId}/stock/{medicationId}/restock` | إعادة تخزين (زيادة) |

### 4.9 `CompanyAgreementsController`

| Method | Endpoint | الوصف |
|---|---|---|
| `GET` | `/api/commerce/company/{companyOrgId}/agreements` | عقود الشركة |
| `POST` | `/api/commerce/company/{companyOrgId}/agreements` | إنشاء عقد (الشركة) |
| `PUT` | `/api/commerce/agreements/{agreementId}` | تعديل عقد |
| `POST` | `/api/commerce/agreements/{agreementId}/suspend` | تعليق |
| `POST` | `/api/commerce/agreements/{agreementId}/activate` | إعادة تفعيل |
| `GET` | `/api/commerce/pharmacy/{pharmacyOrgId}/agreements` | عقود الصيدلية |

### 4.10 Authentication & Authorization

**كل Endpoints تحتاج `[Authorize]`.**

**Authorization Checklist:**
- `CanManageFinance` — لإنشاء/تعديل/إرسال الطلبات، الدفع، إدارة العقود، إدارة المخزون.
- `CanViewReports` — لعرض الطلبات، الفواتير، الـ Ledger.
- `CanManageMembers` — لا شيء في Commerce (لاحقاً، لتعيين المندوبين).

**Middleware (لاحقاً — 19-PERMISSIONS):**
```csharp
[RequirePermission("PHARMACY.SUPPLY_ORDER.CREATE")]
public async Task<IActionResult> Create(...)
```

**الآن:** `[Authorize]` + تحقق يدوي داخل Controller.

---

## 🎯 الجزء الخامس: تكامل نظام النقاط (B2B_SUPPLY)

### 5.1 نظرة عامة

**المرجع:** `06-POINTS.md` (v7.0).

**الحالة الحالية:**
- ✅ `PointSystemTypeID = 2 (B2B_SUPPLY)` موجود في `PointSystemTypes`.
- ✅ `PointSystemTypeOrganizationTypes` يحتوي: `B2B_SUPPLY → PHARMACY (3)` + `DRUG_WAREHOUSE (5)`.
- ⚠️ **ينقص:** `B2B_SUPPLY → PHARMA_COMPANY (2)` (الشركة المصنعة).
- ⚠️ **ينقص:** `PointSystemTypeBeneficiaryTypes` لـ `B2B_SUPPLY → ORG_MEMBER_INTERNAL (2)`.
- ⚠️ **ينقص:** `PSPActivityTypes` لأحداث B2B (`SUPPLY_ORDER_CREATED`, `SUPPLY_ORDER_DELIVERED`, `SUPPLY_ORDER_PAID`).

### 5.2 Seed Data المطلوب

**`V004__PointSystemTypeOrganizationTypes_B2B.sql`:**
```sql
-- إضافة B2B_SUPPLY → PHARMA_COMPANY
IF NOT EXISTS (
    SELECT 1 FROM PointSystemTypeOrganizationTypes 
    WHERE PointSystemTypeID = 2 AND OrganizationTypeID = 2
)
INSERT INTO PointSystemTypeOrganizationTypes (PointSystemTypeID, OrganizationTypeID, IsActive, CreatedDate)
VALUES (2, 2, 1, GETUTCDATE());
```

**`V005__PointSystemTypeBeneficiaryTypes_B2B.sql`:**
```sql
-- إضافة B2B_SUPPLY → ORG_MEMBER_INTERNAL (مندوب الشركة)
IF NOT EXISTS (
    SELECT 1 FROM PointSystemTypeBeneficiaryTypes 
    WHERE PointSystemTypeID = 2 AND BeneficiaryTypeID = 2
)
INSERT INTO PointSystemTypeBeneficiaryTypes (PointSystemTypeID, BeneficiaryTypeID, CreatedDate)
VALUES (2, 2, GETUTCDATE());
```

**`V006__PSPActivityTypes_B2B.sql`:**
```sql
-- إضافة أحداث B2B
MERGE INTO PSPActivityTypes AS target
USING (VALUES 
    ('SUPPLY_ORDER_CREATED', 'إنشاء طلب توريد', 'Supply Order Created', 'عند إنشاء طلب توريد B2B', NULL, 1, 1, 12),
    ('SUPPLY_ORDER_DELIVERED', 'تسليم طلب توريد', 'Supply Order Delivered', 'عند تسليم طلب توريد B2B', NULL, 1, 1, 13),
    ('SUPPLY_ORDER_PAID', 'دفع طلب توريد', 'Supply Order Paid', 'عند دفع طلب توريد B2B', NULL, 1, 1, 14)
) AS source (ActivityTypeCode, ActivityNameAr, ActivityNameEn, Description, DefaultPoints, IsSystem, IsActive, DisplayOrder)
ON target.ActivityTypeCode = source.ActivityTypeCode
WHEN NOT MATCHED THEN
    INSERT (ActivityTypeCode, ActivityNameAr, ActivityNameEn, Description, DefaultPoints, IsSystem, IsActive, DisplayOrder, CreatedDate)
    VALUES (source.ActivityTypeCode, source.ActivityNameAr, source.ActivityNameEn, source.Description, source.DefaultPoints, source.IsSystem, source.IsActive, source.DisplayOrder, GETUTCDATE());
```

### 5.3 كيف يعمل منح النقاط؟

**السيناريو:** شركة أنشأت `PSPProgramPointScheme` (SystemType=B2B_SUPPLY, SourceEntityID=SupplyOrderID).

**عند أحداث معينة:**

| الحدث | `ActivityTypeID` | ماذا يحدث؟ |
|---|---|---|
| **طلب توريد جديد** | `SUPPLY_ORDER_CREATED` | نقاط للمندوب (إن وُجد Rule) |
| **تسليم الطلب** | `SUPPLY_ORDER_DELIVERED` | نقاط للصيدلية + المندوب |
| **دفع الطلب** | `SUPPLY_ORDER_PAID` | نقاط للصيدلية |

**Flow:**
```
1. PharmacySupplyOrder.Status = "DELIVERED"
   ↓
2. SupplyOrderService.CompleteAsync(orderId)
   ↓
3. TryAwardPointsViaOrchestratorAsync(
       campaignId: orderId,
       pointSystemTypeID: 2 (B2B_SUPPLY),
       activityTypeId: SUPPLY_ORDER_DELIVERED,
       context: new PointsContext {
           OrganizationID = companyOrgId,
           CaseID = null,  // ليس مريض
           UserID = pharmacyAdminUserId
       },
       referenceType: "SUPPLY_ORDER",
       referenceId: orderId
   )
   ↓
4. IEarnPointsOrchestrator.AwardByCampaignAsync(...)
   ↓
5. لكل Rule نشط:
       - BeneficiaryTypeID = 1 (ORGANIZATION) → الصيدلية
       - BeneficiaryTypeID = 2 (ORG_MEMBER_INTERNAL) → مندوب الشركة
   ↓
6. EarnPointsHandler.HandleAsync(...)
   ↓
7. Balance + Transaction
```

### 5.4 PointsContext لـ B2B

**الفرق بين PSP و B2B:**

| Field | PSP | B2B |
|---|---|---|
| `OrganizationID` | `patient.InvitedByOrganizationID` (الشركة) | `supplyOrder.CompanyOrgId` (الشركة) |
| `CaseID` | `PSPPatient.CaseID` | `null` (لا مريض) |
| `UserID` | `PSPPatient.PatientProfileID` | `pharmacyAdmin.UserProfileID` |

**ملاحظة:** `PointsContext` يجب أن يدعم `null` لـ `CaseID` (B2B ليس له مريض).

### 5.5 Idempotency

**نفس نمط PSP:**
```csharp
// قبل منح النقاط
var alreadyAwarded = await _db.Set<PSPProgramPointTransaction>()
    .AnyAsync(t => t.ReferenceType == "SUPPLY_ORDER"
                && t.ReferenceID == orderId
                && t.ActivityTypeID == activityTypeId
                && t.TransactionTypeID == PointTransactionTypeIds.Earn);

if (alreadyAwarded) return;
```

### 5.6 التكامل مع `IEarnPointsOrchestrator`

**موجود في `06-POINTS.md`، لكن يحتاج:**

**(1)** `PointsContext` — دعم `CaseID = null` (لأن B2B ليس له مريض).
- **التعديل:** `public int? CaseID { get; set; }` (من `int` إلى `int?`).

**(2)** `AwardByCampaignAsync` — يقبل `pointSystemTypeID = 2` (B2B_SUPPLY).
- **موجود:** يقبل أي `pointSystemTypeID`.

**(3)** `BeneficiaryTypeID = 1 (ORGANIZATION)` — يستخدم `OrganizationID` (من `PointsContext`).
- **موجود:** `EarnPointsHandler` يعرف كيف يتعامل مع `ORGANIZATION`.

**(4)** `BeneficiaryTypeID = 2 (ORG_MEMBER_INTERNAL)` — يستخدم `UserID` (من `PointsContext`).
- **موجود:** `EarnPointsHandler` يعرف كيف يتعامل مع `ORG_MEMBER_INTERNAL`.

**الخلاصة:** لا تعديلات جوهرية — فقط `CaseID` nullable في `PointsContext`.

### 5.7 Loyalty للصيدلية (مؤجل)

**ملاحظة:** `PSPProgramPointScheme` مع `SystemType=2 (B2B_SUPPLY)` و`BeneficiaryTypeID=1 (ORGANIZATION)` = **نقاط للصيدلية**. هذا يكفي الآن.

**ما لا نبنيه الآن:**
- `PharmacyLoyaltyPoint` منفصل — لا حاجة (نستخدم النظام الموجود).
- Cashback على حجم المشتريات — لاحقاً (كيان منفصل عند الحاجة).
- حملات تسويقية معقدة — لاحقاً.

---

## 🎨 الجزء السادس: UI Plan (Pharmacy + Company)

### 6.1 مبادئ التصميم

**المرجع:** `10-UI-STYLE-GUIDE.md` + `01-ARCHITECTURE.md` (Blazor WebView constraints).

**القواعد:**
- ✅ استخدام `Shared.UI` Components (PspGateway, PspDetail إلخ كأنماط).
- ✅ RTL/LTR support (TranslationState + localStorage).
- ✅ Loading/Empty/Error states.
- ✅ Responsive (mobile-first).
- ❌ لا `BlazorWebView` مع Parameters ديناميكية في ShellContent (انظر 13-MAUI § 6.5).

### 6.2 صفحات الصيدلية (Pharmacy)

#### (أ) `PharmacyCatalogPage.razor`
**الغرض:** تصفح كتالوج الشركة.

**المكونات:**
- **Header:** اسم الشركة + رصيد Ledger (دائن/مدين).
- **Filter Bar:** بحث بالاسم + Filter (Category).
- **Grid:** كروت الأدوية (صورة، اسم، سعر جملة، مخزون).
- **Add to Cart:** زر لكل دواء (يفتح Drawer للكمية).

**مصدر البيانات:** `GET /api/commerce/catalog/{companyOrgId}`.

**المسار:** `/commerce/catalog/{companyOrgId}`.

#### (ب) `PharmacyCartPage.razor`
**الغرض:** سلة المشتريات + Checkout.

**المكونات:**
- **Cart Items:** قائمة (دواء، كمية، سعر، إجمالي).
- **Edit Quantity:** + / -.
- **Remove Item:** زر.
- **Totals:** SubTotal + Tax + Shipping + Total.
- **Notes:** textarea.
- **Checkout Button:** → `ServiceOrder` (DRAFT → SUBMITTED).

**مصدر البيانات:** Local state + `POST /api/commerce/supply-orders`.

**المسار:** `/commerce/cart/{companyOrgId}`.

#### (ج) `PharmacyOrdersPage.razor`
**الغرض:** قائمة الطلبات (صيدلية).

**المكونات:**
- **Tabs:** All | DRAFT | SUBMITTED | CONFIRMED | SHIPPED | DELIVERED | COMPLETED | CANCELLED.
- **List:** كروت (رقم الطلب، التاريخ، الشركة، الحالة، الإجمالي).
- **Click:** → `PharmacyOrderDetailsPage`.

**مصدر البيانات:** `GET /api/commerce/supply-orders?pharmacyOrgId=X`.

**المسار:** `/commerce/orders`.

#### (د) `PharmacyOrderDetailsPage.razor`
**الغرض:** تفاصيل طلب + Actions.

**المكونات:**
- **Header:** رقم الطلب + الحالة (Badge) + الشركة.
- **Timeline:** State History.
- **Items:** جدول (دواء، كمية، سعر، إجمالي).
- **Totals:** SubTotal + Tax + Shipping + Total.
- **Actions:**
  - `DRAFT` → Submit / Delete.
  - `SUBMITTED` → Cancel.
  - `SHIPPED` → Confirm Delivery.
  - `DELIVERED` → View Invoice / Pay.
- **Sidebar:** Payment + Shipment + Invoice.

**مصدر البيانات:** `GET /api/commerce/supply-orders/{id}` + `/history` + `/payments` + `/shipments`.

**المسار:** `/commerce/orders/{id}`.

#### (هـ) `PharmacyLedgerPage.razor`
**الغرض:** أرصدة الصيدلية مع كل الشركات.

**المكونات:**
- **Summary Cards:** Total Receivable + Total Payable.
- **List:** الشركات (اسم، رصيد، آخر تسوية).
- **Click:** → `LedgerDetailsPage`.

**مصدر البيانات:** `GET /api/commerce/ledger/ORGANIZATION/{pharmacyOrgId}`.

**المسار:** `/commerce/ledger`.

#### (و) `PharmacyStockPage.razor` (اختياري — لاحقاً)
**الغرض:** مخزون الصيدلية (بعد استلام الطلبات).

**مؤجل — لا نبنيها الآن.**

### 6.3 صفحات الشركة (Company)

#### (أ) `CompanyOrdersPage.razor`
**الغرض:** قائمة الطلبات الواردة.

**المكونات:**
- **Tabs:** All | SUBMITTED | CONFIRMED | PREPARING | SHIPPED.
- **List:** كروت (رقم الطلب، التاريخ، الصيدلية، الحالة، الإجمالي).
- **Click:** → `CompanyOrderDetailsPage`.

**مصدر البيانات:** `GET /api/commerce/supply-orders?companyOrgId=X`.

**المسار:** `/commerce/company/orders`.

#### (ب) `CompanyOrderDetailsPage.razor`
**الغرض:** تفاصيل طلب + Actions.

**المكونات:**
- **Header:** رقم الطلب + الحالة + الصيدلية.
- **Timeline:** State History.
- **Items:** جدول.
- **Actions:**
  - `SUBMITTED` → Confirm / Reject.
  - `CONFIRMED` → Prepare / Cancel.
  - `PREPARING` → Ship / Cancel.
- **Sidebar:** Payment + Shipment + Invoice.

**مصدر البيانات:** `GET /api/commerce/supply-orders/{id}` + `/history`.

**المسار:** `/commerce/company/orders/{id}`.

#### (ج) `CompanyStockPage.razor`
**الغرض:** إدارة مخزون الشركة.

**المكونات:**
- **Search:** بحث دواء.
- **Table:** (دواء، مخزون متاح، محجوز، سعر جملة، حد أدنى).
- **Edit:** → Dialog (تعديل كمية/سعر).
- **Restock:** → Dialog (إضافة كمية).

**مصدر البيانات:** `GET /api/commerce/company/{companyOrgId}/stock` + `PUT` + `POST /restock`.

**المسار:** `/commerce/company/stock`.

#### (د) `CompanyAgreementsPage.razor`
**الغرض:** إدارة العقود.

**المكونات:**
- **List:** عقود (صيدلية، خصم، حد ائتمان، حالة).
- **Create:** → Dialog (صيدلية + شروط).
- **Actions:** Suspend / Activate / Edit.

**مصدر البيانات:** `GET /api/commerce/company/{companyOrgId}/agreements` + `POST` + `PUT`.

**المسار:** `/commerce/company/agreements`.

#### (هـ) `CompanyLedgerPage.razor`
**الغرض:** أرصدة الشركة مع كل الصيدليات.

**نفس نمط `PharmacyLedgerPage`** — لكن من منظور الشركة.

**المسار:** `/commerce/company/ledger`.

### 6.4 التنقل (Navigation)

**في `PharmacyDashboard` (موجود):**
- إضافة كارت جديد: "طلبات التوريد" → `/commerce/orders`.
- إضافة كارت: "المستحقات" → `/commerce/ledger`.

**في `PharmaCompanyDashboard` (موجود):**
- إضافة كارت: "طلبات التوريد الواردة" → `/commerce/company/orders`.
- إضافة كارت: "المخزون" → `/commerce/company/stock`.
- إضافة كارت: "العقود" → `/commerce/company/agreements`.
- إضافة كارت: "المستحقات" → `/commerce/company/ledger`.

**التعديل على AppShell (MAUI) + NavMenu (PWA):**
- إضافة الروابط الجديدة (ضمن `MOBILE.SHELL` translation keys).

### 6.5 Shared.UI Components المخططة

| المكون | الغرض | الحالة |
|---|---|---|
| `CommerceOrderCard.razor` | كارت طلب (في القوائم) | جديد |
| `CommerceOrderTimeline.razor` | Timeline الحالات | جديد |
| `CommerceLedgerCard.razor` | كارت رصيد | جديد |
| `CommerceStockTable.razor` | جدول مخزون | جديد |
| `CommercePaymentDialog.razor` | Dialog دفع | جديد |
| `CommerceShipmentDialog.razor` | Dialog شحن | جديد |
| `CommerceAgreementDialog.razor` | Dialog عقد | جديد |
| `CommerceQuantityInput.razor` | Input كمية + validation | جديد |

**كل هذه في `Shared.UI/Components/Commerce/`.**

### 6.6 أنماط التصميم

**المرجع:** `10-UI-STYLE-GUIDE.md`.

**الألوان:**
- `--rubik-primary: #1B5A7A` (B2B).
- `--rubik-success: #2E7D5E` (تم التسليم).
- `--rubik-warning: #B88B4A` (قيد الانتظار).
- `--rubik-danger: #B33F40` (ملغي).

**المكونات:**
- **Cards:** مثل `PspGateway` (Header + Body + Actions).
- **Timeline:** عمودي مع icons.
- **Dialogs:** `GenericModal` (موجود).

### 6.7 أولوية الصفحات (للتنفيذ)

**المرحلة 1 (MVP — للاختبار):**
1. `PharmacyCatalogPage` — تصفح.
2. `PharmacyCartPage` — سلة + Submit.
3. `PharmacyOrdersPage` — قائمة.
4. `PharmacyOrderDetailsPage` — تفاصيل + Actions (deliver).
5. `CompanyOrdersPage` — قائمة.
6. `CompanyOrderDetailsPage` — تفاصيل + Actions (confirm/prepare/ship).

**المرحلة 2 (إثراء):**
7. `PharmacyLedgerPage`.
8. `CompanyLedgerPage`.
9. `CompanyStockPage`.
10. `CompanyAgreementsPage`.

**المرحلة 3 (تحسينات):**
11. `PharmacyStockPage` (لاحقاً).
12. Payments online (Paymob/Fawry).

### 6.8 التكامل مع Dashboard الموجود

**`PharmacyDashboard.razor` (موجود):**
```razor
<!-- في pharmacy-actions-grid -->
<div class="pharmacy-action-card" @onclick="() => OnNavigateToSupplyOrders()">
    <div class="pharmacy-action-icon pharmacy-action-icon-orders"><span>📦</span></div>
    <span class="pharmacy-action-title">@T("SHARED.PHARMACY_DASHBOARD.SUPPLY_ORDERS")</span>
    <span class="pharmacy-action-subtitle">@T("SHARED.PHARMACY_DASHBOARD.SUPPLY_ORDERS_DESC")</span>
</div>
```

**`PharmaCompanyDashboard.razor` (موجود):**
```razor
<!-- في الشركة dashboard -->
<div class="company-action-card" @onclick="() => OnNavigateToIncomingOrders()">
    <div class="company-action-icon"><span>📥</span></div>
    <span class="company-action-title">@T("SHARED.COMPANY_DASHBOARD.INCOMING_ORDERS")</span>
</div>
```

**Translation Keys الجديدة:**
- `SHARED.PHARMACY_DASHBOARD.SUPPLY_ORDERS` = "طلبات التوريد"
- `SHARED.PHARMACY_DASHBOARD.SUPPLY_ORDERS_DESC` = "تصفح وإدارة طلبات الشراء"
- `SHARED.COMPANY_DASHBOARD.INCOMING_ORDERS` = "الطلبات الواردة"
- ... (باقي keys لكل صفحة)

### 6.9 Routes (MAUI + PWA)

**MAUI (`AppShell.xaml.cs`):**
```csharp
Routing.RegisterRoute("commerce/catalog", typeof(CommerceCatalogFlow));
Routing.RegisterRoute("commerce/cart", typeof(CommerceCartFlow));
Routing.RegisterRoute("commerce/orders", typeof(CommerceOrdersFlow));
Routing.RegisterRoute("commerce/orders/{id}", typeof(CommerceOrderDetailsFlow));
Routing.RegisterRoute("commerce/ledger", typeof(CommerceLedgerFlow));
Routing.RegisterRoute("commerce/company/orders", typeof(CompanyOrdersFlow));
Routing.RegisterRoute("commerce/company/orders/{id}", typeof(CompanyOrderDetailsFlow));
Routing.RegisterRoute("commerce/company/stock", typeof(CompanyStockFlow));
Routing.RegisterRoute("commerce/company/agreements", typeof(CompanyAgreementsFlow));
Routing.RegisterRoute("commerce/company/ledger", typeof(CompanyLedgerFlow));
```

**PWA (`@page` directives):**
- `/commerce/catalog/{companyOrgId:int}`
- `/commerce/cart/{companyOrgId:int}`
- `/commerce/orders`
- `/commerce/orders/{id:int}`
- `/commerce/ledger`
- `/commerce/company/orders`
- `/commerce/company/orders/{id:int}`
- `/commerce/company/stock`
- `/commerce/company/agreements`
- `/commerce/company/ledger`

---
## 🗓️ الجزء السابع: خطة العمل (Sprints)

### 7.1 نظرة عامة على Sprints

**الفلسفة:** تقسيم العمل إلى Sprints أسبوعية، كل Sprint يُسلّم **شيئاً قابلاً للاختبار**.

| Sprint | الموضوع | المدة | الحالة |
|---|---|---|---|
| **S1** | Domain Entities + Migration | 3-4 أيام | ⏳ القادم |
| **S2** | Application Layer (Services) | 3-4 أيام | ⏳ |
| **S3** | API Controllers (Orders + Catalog) | 4-5 أيام | ⏳ |
| **S4** | API Controllers (Payments + Shipments + Invoices + Ledger) | 4-5 أيام | ⏳ |
| **S5** | UI Phase 1 (Pharmacy — 5 صفحات) | 5-7 أيام | ⏳ |
| **S6** | UI Phase 2 (Company — 4 صفحات) | 4-5 أيام | ⏳ |
| **S7** | Integration + E2E Testing | 3-4 أيام | ⏳ |
| **S8** | Seed Data + Polish + Deployment | 2-3 أيام | ⏳ |

**المجموع:** 8 Sprints، ~30-37 يوم عمل (~6-7 أسابيع مع جداول واقعية).

### 7.2 Sprint 1 — Domain Entities + Migration

**الهدف:** بناء 12 كيان + Migration + تطبيق على Dev.

**المهام:**

| # | المهمة | التقدير | المخرج |
|---|---|---|---|
| 1.1 | إنشاء `RubikCare.Domain/Commerce/Engine/` + 7 كيانات | 4 ساعات | كود C# |
| 1.2 | إنشاء `RubikCare.Domain/Commerce/B2B/` + 5 كيانات | 3 ساعات | كود C# |
| 1.3 | إنشاء Enums (4 Engine + 2 B2B) | 1 ساعة | كود C# |
| 1.4 | إضافة `DbSet<>` في `BusinessDbContext` | 1 ساعة | Patch |
| 1.5 | Fluent Config في `OnModelCreating` (Indexes + Relations) | 2 ساعات | Patch |
| 1.6 | Migration `AddCommerceEngineTables` (dotnet ef) | 1 ساعة | Migration |
| 1.7 | مراجعة Migration المولّد | 1 ساعة | — |
| 1.8 | Pre-Check + Apply على Dev | 1 ساعة | مطبّق |
| 1.9 | التحقق (12 جدول موجود) | 30 دقيقة | — |
| 1.10 | Pre-Check + Apply على ProdCopy | 1 ساعة | مطبّق |
| 1.11 | تحديث `23-COMMERCE-ENGINE.md` (سجل) | 30 دقيقة | — |

**المجموع:** ~16 ساعة (~2 يوم عمل مكثف، أو 3-4 أيام مع جداول).

**المخاطر:**
- ⚠️ Migration قد يفشل (Indexes متعارضة).
- **الاحتياطي:** اختبار على Dev أولاً.

**Deliverable:** 12 جدول على `RubikCare_Dev_V2` + `RubikCare_ProdCopy`.

### 7.3 Sprint 2 — Application Layer (Services)

**الهدف:** بناء Services للعمليات الأساسية.

**المهام:**

| # | المهمة | التقدير |
|---|---|---|
| 2.1 | `IServiceOrderService` + `ServiceOrderService` (Create, Get, Update) | 3 ساعات |
| 2.2 | `ISupplyOrderService` + `SupplyOrderService` (State Machine) | 4 ساعات |
| 2.3 | `ICatalogService` + `CatalogService` (Get Catalog by Company) | 2 ساعات |
| 2.4 | `ILedgerService` + `LedgerService` (Get Balance, Charge, Credit) | 3 ساعات |
| 2.5 | `IPaymentService` + `PaymentService` | 2 ساعات |
| 2.6 | `IShipmentService` + `ShipmentService` | 2 ساعات |
| 2.7 | `IStockService` + `StockService` (Reserve, Deduct, Release) | 2 ساعات |
| 2.8 | `IAgreementService` + `AgreementService` | 2 ساعات |
| 2.9 | Unit Tests (Business Rules: stock, credit, state) | 4 ساعات |
| 2.10 | التسجيل في `Program.cs` | 30 دقيقة |

**المجموع:** ~25 ساعة (~3-4 أيام).

**المخاطر:**
- ⚠️ Transaction boundaries في `SupplyOrderService` (Confirm + Stock Reserve + Ledger Charge — كلها في Transaction واحد).
- **الاحتياطي:** استخدام `ExecuteWithNewContextAsync` (موجود).

**Deliverable:** 8 Services + Unit Tests.

### 7.4 Sprint 3 — API Controllers (Orders + Catalog)

**الهدف:** 4 Controllers (Catalog, SupplyOrders, CompanyStock, CompanyAgreements).

**المهام:**

| # | المهمة | التقدير |
|---|---|---|
| 3.1 | `CatalogController` (4 endpoints) | 3 ساعات |
| 3.2 | `SupplyOrdersController` (12 endpoints) | 6 ساعات |
| 3.3 | `CompanyStockController` (4 endpoints) | 2 ساعات |
| 3.4 | `CompanyAgreementsController` (6 endpoints) | 3 ساعات |
| 3.5 | Authorization (`CanManageFinance` + تحقق) | 1 ساعة |
| 3.6 | Swagger + OpenAPI annotations | 1 ساعة |
| 3.7 | Postman Collection | 1 ساعة |
| 3.8 | اختبار يدوي (Postman) | 2 ساعات |

**المجموع:** ~19 ساعة (~2-3 أيام).

**المخاطر:**
- ⚠️ Endpoints Authorization قد يسبب 403 غير متوقع.
- **الاحتياطي:** اختبار بـ `RubikCare_ProdCopy`.

**Deliverable:** 26 endpoints جاهزة.

### 7.5 Sprint 4 — API Controllers (Payments + Shipments + Invoices + Ledger)

**الهدف:** 4 Controllers إضافية.

**المهام:**

| # | المهمة | التقدير |
|---|---|---|
| 4.1 | `SupplyOrderPaymentsController` (4 endpoints) | 3 ساعات |
| 4.2 | `SupplyOrderShipmentsController` (6 endpoints) | 4 ساعات |
| 4.3 | `SupplyOrderInvoicesController` (4 endpoints) | 3 ساعات |
| 4.4 | `LedgerController` (4 endpoints) | 3 ساعات |
| 4.5 | اختبار Integration (Order → Payment → Shipment → Invoice) | 4 ساعات |
| 4.6 | Postman Collection محدّث | 1 ساعة |

**المجموع:** ~18 ساعة (~2-3 أيام).

**Deliverable:** 18 endpoints + اختبار E2E للدورة الكاملة.

### 7.6 Sprint 5 — UI Phase 1 (Pharmacy)

**الهدف:** 5 صفحات للصيدلية.

**المهام:**

| # | المهمة | التقدير |
|---|---|---|
| 5.1 | `CommerceOrderCard.razor` + `CommerceOrderTimeline.razor` | 3 ساعات |
| 5.2 | `CommerceQuantityInput.razor` | 1 ساعة |
| 5.3 | `PharmacyCatalogPage.razor` | 4 ساعات |
| 5.4 | `PharmacyCartPage.razor` | 3 ساعات |
| 5.5 | `PharmacyOrdersPage.razor` | 3 ساعات |
| 5.6 | `PharmacyOrderDetailsPage.razor` | 4 ساعات |
| 5.7 | `PharmacyLedgerPage.razor` | 3 ساعات |
| 5.8 | Translation keys (bilingual) | 1 ساعة |
| 5.9 | CSS (`commerce.css`) | 2 ساعات |
| 5.10 | Navigation (`AppShell.xaml.cs` + PWA routes) | 1 ساعة |
| 5.11 | Dashboard integration (`PharmacyDashboard.razor`) | 1 ساعة |

**المجموع:** ~26 ساعة (~4 أيام).

**المخاطر:**
- ⚠️ BlazorWebView Parameters (إن استُخدمت في ShellContent).
- **الاحتياطي:** استخدام `Routing.RegisterRoute` للصفحات الفرعية.

**Deliverable:** 5 صفحات جاهزة (PWA + MAUI).

### 7.7 Sprint 6 — UI Phase 2 (Company)

**الهدف:** 4 صفحات للشركة.

**المهام:**

| # | المهمة | التقدير |
|---|---|---|
| 6.1 | `CommerceStockTable.razor` | 2 ساعات |
| 6.2 | `CommerceAgreementDialog.razor` + `CommercePaymentDialog.razor` | 3 ساعات |
| 6.3 | `CompanyOrdersPage.razor` | 3 ساعات |
| 6.4 | `CompanyOrderDetailsPage.razor` | 4 ساعات |
| 6.5 | `CompanyStockPage.razor` | 3 ساعات |
| 6.6 | `CompanyAgreementsPage.razor` | 3 ساعات |
| 6.7 | `CompanyLedgerPage.razor` | 2 ساعات |
| 6.8 | Translation keys | 1 ساعة |
| 6.9 | Navigation + Dashboard integration | 1 ساعة |

**المجموع:** ~22 ساعة (~3-4 أيام).

**Deliverable:** 4 صفحات جاهزة.

### 7.8 Sprint 7 — Integration + E2E Testing

**الهدف:** اختبار الدورة الكاملة.

**المهام:**

| # | المهمة | التقدير |
|---|---|---|
| 7.1 | اختبار كامل: صيدلية تطلب → شركة تؤكد → تشحن → الصيدلية تستلم | 4 ساعات |
| 7.2 | اختبار Payment + Ledger + Settlement | 3 ساعات |
| 7.3 | اختبار State Machine (كل التحولات) | 2 ساعات |
| 7.4 | اختبار Authorization (403 للحالات غير المصرح) | 2 ساعات |
| 7.5 | اختبار Edge Cases (CreditLimit, Stock, Reject) | 3 ساعات |
| 7.6 | اختبار PWA (browser) | 2 ساعات |
| 7.7 | اختبار MAUI (device/emulator) | 2 ساعات |
| 7.8 | Bug Fixes | 4 ساعات |

**المجموع:** ~22 ساعة (~3-4 أيام).

### 7.9 Sprint 8 — Seed Data + Polish + Deployment

**الهدف:** تجهيز النشر.

**المهام:**

| # | المهمة | التقدير |
|---|---|---|
| 8.1 | Seed Scripts (V004, V005, V006 — B2B Points) | 2 ساعات |
| 8.2 | Seed Scripts (V007 — PaymentMethods) | 1 ساعة |
| 8.3 | Translations (كل المفاتيح الجديدة) | 2 ساعات |
| 8.4 | Documentation (تحديث `23-COMMERCE-ENGINE.md`) | 2 ساعات |
| 8.5 | Pre-Check + Apply على `RubikCare_Production` | 1 ساعة |
| 8.6 | Deployment (Web/API/PWA) | 2 ساعات |
| 8.7 | Smoke Test على Production | 2 ساعات |
| 8.8 | Rollback Plan موثّق | 1 ساعة |

**المجموع:** ~13 ساعة (~2 أيام).

### 7.10 Sprint 0 (تمهيدي — اختياري)

**الهدف:** تجهيز البيئة.

**المهام:**
- ✅ `__SeedHistory` (مطبّق).
- ✅ `Pre-Migration-Check.ps1` (مطبّق).
- ✅ `Apply-Migrations.ps1` (مطبّق).
- ⏳ `Backup-Before-Deploy.ps1` (اختياري — قبل Sprint 8).

### 7.11 إجمالي التقديرات

| Sprint | الساعات | الأيام (8h) |
|---|---|---|
| S1 | 16 | 2 |
| S2 | 25 | 3-4 |
| S3 | 19 | 2-3 |
| S4 | 18 | 2-3 |
| S5 | 26 | 3-4 |
| S6 | 22 | 3-4 |
| S7 | 22 | 3-4 |
| S8 | 13 | 2 |
| **المجموع** | **161** | **~20-26 يوم** |

**معامل AI (0.3-0.5):** قد يُقلّص إلى **~10-15 يوم** مع AI Assistants.

**الجدول الواقعي:** **6-7 أسابيع** (مع الاختبار والمراجعة).

---

## ⚠️ الجزء الثامن: المخاطر + الدروس المتوقعة

### 8.1 المخاطر الحرجة (High)

| # | الخطر | الاحتمال | التأثير | التخفيف |
|---|---|---|---|---|
| 1 | Migration فاشلة على Production | 🟡 متوسط | 🔴 عالي | اختبار كامل على Dev + ProdCopy أولاً. Backup إلزامي. |
| 2 | تعارض Schema مع `PSPParticipation` | 🟢 منخفض | 🔴 عالي | `PharmacyCompanyAgreement` منفصل. اختبار على ProdCopy. |
| 3 | Ledger Balance غير متسق | 🟡 متوسط | 🔴 عالي | Transaction + `UPDLOCK`. Idempotency. اختبار E2E. |
| 4 | Stock overselling | 🟡 متوسط | 🟠 متوسط | `ReservedQuantity` + `[Timestamp]`. اختبار Concurrency. |
| 5 | Authorization خطأ (403 غير متوقع) | 🟡 متوسط | 🟠 متوسط | اختبار كل Role. `CanManageFinance` واضح. |
| 6 | نقاط B2B مضاعفة | 🟡 متوسط | 🟠 متوسط | Idempotency (`ReferenceType + ReferenceID + ActivityTypeID`). |

### 8.2 المخاطر المتوسطة (Medium)

| # | الخطر | الاحتمال | التأثير | التخفيف |
|---|---|---|---|---|
| 7 | UI BlazorWebView parameters conflict | 🟡 متوسط | 🟠 متوسط | استخدام `Routing.RegisterRoute` (كما في 13-MAUI § 6.5). |
| 8 | Performance (قوائم كبيرة) | 🟡 متوسط | 🟠 متوسط | Pagination. Indexes. |
| 9 | Translation keys ناقصة | 🟡 متوسط | 🟢 منخفض | Pre-Check لـ Resources (SQL MERGE). |
| 10 | BlazorWebView قديم في PWA | 🟢 منخفض | 🟠 متوسط | Test في InPrivate. Clear Service Worker. |

### 8.3 الدروس المتوقعة (Lessons to be Learned)

#### الدرس 14: تعارض Schema بين B2B و PSP

**التوقع:** `PharmacyCompanyAgreement` قد يتعارض مع `PSPParticipation`.
**الحل:** فصل واضح (B2B = عقد تجاري، PSP = مشاركة برنامج).

#### الدرس 15: Ledger Balance غير متسق

**التوقع:** `ServiceLedger.Balance` قد لا يتطابق مع مجموع `LedgerTransactions`.
**الحل:** Transaction + `BalanceBefore`/`BalanceAfter`. Reconciliation script لاحقاً.

#### الدرس 16: Stock Overselling

**التوقع:** صيدليتان تطلبان نفس الدواء في نفس الوقت → كلاهما يحصل على تأكيد.
**الحل:** `ReservedQuantity` + `[Timestamp] RowVersion` + Transaction.

#### الدرس 17: نقاط B2B مكررة

**التوقع:** إعادة تشغيل `TryAwardPointsViaOrchestratorAsync` قد تُضاعف النقاط.
**الحل:** Idempotency check (نفس نمط PSP).

#### الدرس 18: Authorization 403 غير متوقع

**التوقع:** `CanManageFinance` غير مضبوط لبعض الأعضاء → 403.
**الحل:** عرض رسالة واضحة. التحقق من `OrgMembership` قبل المحاولة.

### 8.4 ما هو مخطط لاحقاً (Post-B2B)

| # | الميزة | الأولوية | Sprint متوقع |
|---|---|---|---|
| 1 | Warehouse Layer (مخازن) | 🟠 عالية | Sprint 9-10 |
| 2 | Pharmacy Loyalty Points | 🟡 متوسطة | Sprint 11 |
| 3 | Online Payments (Paymob/Fawry) | 🟠 عالية | Sprint 12 |
| 4 | Clinic OS (Booking) | 🟡 متوسطة | Sprint 13-14 |
| 5 | Retail (Pharmacy → Patient) | 🟡 متوسطة | Sprint 15 |
| 6 | `/admin/database` (Razor) | 🟢 منخفضة | Sprint 16 |
| 7 | Hangfire + Windows Service | 🟢 منخفضة | Sprint 17 |
| 8 | CI/CD (GitHub Actions) | 🟡 متوسطة | Sprint 18 |

---

## 📝 الجزء التاسع: سجل التغييرات

### 9.1 الإصدار 1.0 (2026-11-07) — Draft

**الإنجازات:**
- ✅ التصميم الكامل (Engine + Module Layer).
- ✅ 12 كيان موثّق.
- ✅ 5 State Machines.
- ✅ 9 Business Rules.
- ✅ 9 Controllers + ~40 Endpoint.
- ✅ 9 صفحات UI.
- ✅ 8 Sprints + تقديرات.
- ✅ 6 مخاطر حرجة + 4 متوسطة.
- ✅ 5 دروس متوقعة.

**الملفات المخططة:**
| الملف | الحالة |
|---|---|
| `23-COMMERCE-ENGINE.md` (هذه الوثيقة) | ✅ مطبّق في `docs/commerce/` |
| 12 Domain Entities | ⏳ Sprint 1 |
| Migration `AddCommerceEngineTables` | ⏳ Sprint 1 |
| 8 Application Services | ⏳ Sprint 2 |
| 9 API Controllers | ⏳ Sprint 3-4 |
| 9 UI Pages + 8 Components | ⏳ Sprint 5-6 |
| Seed Scripts (V004-V007) | ⏳ Sprint 8 |

### 9.2 ما هو مخطط (2026-11/12)

| # | البند | Sprint | التقدير |
|---|---|---|---|
| 1 | Domain Entities + Migration | S1 | 2-4 أيام |
| 2 | Application Services | S2 | 3-4 أيام |
| 3 | API Controllers (Orders + Catalog) | S3 | 2-3 أيام |
| 4 | API Controllers (Payments + Shipments + Invoices + Ledger) | S4 | 2-3 أيام |
| 5 | UI Phase 1 (Pharmacy) | S5 | 3-4 أيام |
| 6 | UI Phase 2 (Company) | S6 | 3-4 أيام |
| 7 | Integration + E2E Testing | S7 | 3-4 أيام |
| 8 | Seed Data + Polish + Deployment | S8 | 2 أيام |

### 9.3 سجل التغييرات المستقبلي

**عند كل تغيير في هذه الوثيقة — أضف سطراً هنا:**

| الإصدار | التاريخ | التغيير |
|---|---|---|
| 1.0 | 2026-11-07 | الإصدار الأولي — التصميم الكامل (Engine + Module + State Machines + APIs + UI + Sprints) |

---

## 🔗 الجزء العاشر: مراجع ذات صلة

### 10.1 وثائق داخلية

| الوثيقة | العلاقة |
|---|---|
| `01-DATABASE-MANAGEMENT.md` | إدارة DB (Migration, Backup, Pre-Check) |
| `06-POINTS.md` | نظام النقاط (B2B_SUPPLY = 2) |
| `17-STRATEGIC-ROADMAP.md` | استراتيجية B2B + Warehouse + Retail |
| `19-PERMISSIONS.md` | نظام الصلاحيات (لاحقاً) |
| `22-INTEGRATION-HUB.md` | تكامل ERP (لاحقاً) |
| `15-DEPLOYMENT.md` | نشر IIS/PWA/API |

### 10.2 السكريبتات

| السكريبت | الاستخدام |
|---|---|
| `Pre-Migration-Check.ps1` | قبل كل Migration |
| `Apply-Migrations.ps1` | تطبيق Migrations |
| `Migrations_Exceptions.md` | سجل استثناءات |

### 10.3 البيئات

| DB | الغرض |
|---|---|
| `RubikCare_Dev_V2` | Development (اختبار أول) |
| `RubikCare_ProdCopy` | Production-like (اختبار ثاني) |
| `RubikCare_Production` | Production (بحذر) |

---
## 🎁 الجزء الثالث عشر: نظام البونص (Bonus System)

### 13.1 نظرة عامة

نظام البونص يسمح للشركة بتقديم "هدايا" (كميات مجانية) على كميات محددة من المنتجات. مثال:
- اشترِ 10، احصل على 1 مجاناً.
- اشترِ 20، احصل على 3 مجاناً.

### 13.2 الكيانات

| الكيان | الغرض |
|---|---|
| `ProductBonus` | رأس البونص (منتج، فترة، حالة). |
| `ProductBonusTier` | شرائح البونص (MinQuantity → BonusQuantity). |

### 13.3 منطق الحساب (Greedy Auto-Calc)

عند إضافة منتج للطلب، يُحسب البونص تلقائياً:
1. ترتيب الشرائح تنازلياً (`MinQuantity`).
2. لكل شريحة: `times = remainingQty / MinQuantity`، `bonus += times × BonusQuantity`، `remainingQty -= times × MinQuantity`.
3. البونص النهائي = مجموع `bonus` لكل الشرائح.

**مثال:** الكمية = 35، الشرائح = [10+1, 20+3].
- 35 / 20 = 1 → bonus = 3، remainingQty = 15.
- 15 / 10 = 1 → bonus = 1، remainingQty = 5.
- 5 < 10 → توقف.
- **البونص = 4.**

### 13.4 الملفات التي تطبق هذا المنطق

| الملف | الغرض |
|---|---|
| `SupplyOrderService.CreateDraftAsync` | حساب البونص في الطلب. |
| `ProductDetailsPage.razor` | عرض حساب البونص (Greedy) للصيدلي. |
| `MedicationDetail.razor` | إدارة شرائح البونص (للشركة). |

### 13.5 حالة التنفيذ

- ✅ الكيانات (ProductBonus + ProductBonusTier).
- ✅ API (Create + Get + Delete).
- ✅ UI (إدارة البونص للشركة + عرض البونص للصيدلي).
- ✅ تطبيق في الطلب (Greedy Auto-Calc).
---
4. **الضريبة لا تُضاف فوق سعر الجمهور** — هي "مضمنة" فيه.
5. **الخصومات تُحسب على سعر الجمهور مباشرةً** (لأنها "تخفيضات" على ما يدفعه المريض).
6. **الأسعار التي تظهر للصيدلي:**
- السعر المشطوب = PublicPrice
- الخصم = PublicPrice × Pharm% × Add%
- السعر النهائي = NetUnitPrice
- الضريبة المضمنة = NetUnitPrice × Tax% / (1 + Tax%)
---
## 💰 الجزء الرابع عشر: Price & Tax Model (القاعدة الذهبية)

### 14.1 التعريفات المرجعية

| المصطلح | المعنى | المعادلة |
|---|---|---|
| **PublicPrice** | ما يُخزَّن في `PharmaCompanyMedications.Price` — شاملاً الضريبة | مدخل من الشركة |
| **TaxPercent** | `PharmaCompanyMedications.SalesTax` | مدخل من الشركة |
| **BasePrice** | السعر قبل الضريبة (يُشتَق، لا يُخزَّن) | `PublicPrice / (1 + TaxPercent / 100)` |
| **PharmacistDiscountPercent** | خصم الصيدلي | مدخل من الشركة |
| **AdditionalDiscountPercent** | خصم إضافي | مدخل من الشركة |
| **NetUnitPrice** | ما يدفعه الصيدلي فعلياً | `PublicPrice × (1 − Pharm%) × (1 − Add%)` |

### 14.2 القواعد الذهبية

1. **`PharmaCompanyMedications.Price` = سعر الجمهور شاملاً الضريبة.**
2. **`SalesTax` = نسبة الضريبة (للعرض فقط).**
3. **معادلة السعر الصافي للصيدلي:**
NetUnitPrice = PublicPrice × (1 − Pharm%) × (1 − Add%)
### 14.3 الملفات التي تطبق هذه القاعدة

| الملف | الغرض |
|---|---|
| `PharmacyCatalogPage.razor` | عرض السعر للصيدلي |
| `PharmacyCartPage.razor` + `CommerceCartItem.cs` | حساب إجماليات السلة |
| `SupplyOrderService.CreateDraftAsync` | حفظ SubTotal / TotalAmount |
| `OrderDetailsPage.razor` | عرض تفاصيل الفاتورة |
| `ProductDetailsPage.razor` | عرض تفاصيل المنتج |
| `BundlesPage.razor` + `BundleService` | حساب سعر الباقة |
| `PharmacyOrdersPage.razor` | عرض الإجمالي في القائمة |

### 14.4 مثال عددي

**الدواء 13169 (بريستو 20 قطعة):**
- BasePrice (قبل الضريبة) = 56.00 EGP
- Tax = 14%
- PublicPrice (سعر الجمهور) = 56 × 1.14 = **63.84 EGP**

**بعد خصم صيدلي 20% + خصم إضافي 5%:**
- بعد خصم الصيدلي = 56 × 0.80 = 44.80
- بعد الخصم الإضافي = 44.80 × 0.95 = 42.56
- الضريبة = 42.56 × 0.14 = 5.96
- **NetUnitPrice = 48.52 EGP** (ما يدفعه الصيدلي فعلياً)

**في الفاتورة:**
- السعر قبل الخصم = 63.84 EGP
- الخصم = 15.32 EGP
- السعر النهائي = 48.52 EGP
- الضريبة المضمنة = 5.96 EGP
