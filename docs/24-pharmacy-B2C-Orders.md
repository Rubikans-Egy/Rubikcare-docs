# 📋 وثيقة B2C - عملية طلب الدواء من الصيدليات

**الإصدار:** 1.0  
**التاريخ:** 22 نوفمبر 2026  
**الحالة:** ✅ موثق بالكامل  
**الجمهور:** فريق التطوير + AI Assistants  
**المرجع الإلزامي:** يُقرأ قبل أي تطوير لصفحات B2C

---

## 📌 الجزء الأول: نظرة عامة (Overview)

### 1.1 ما هو نموذج B2C في RubikCare؟

نموذج **B2C (Business-to-Consumer)** في RubikCare يمثل التدفق الكامل للمريض (Patient) من البحث عن صيدلية حتى استلام الدواء. هذا النموذج يخدم **المريض النهائي** الذي يحتاج إلى:
- البحث عن صيدليات قريبة أو في منطقة معينة
- طلب أدوية عادية (Non-PSP) أو أدوية مدعومة (PSP)
- تتبع حالة الطلبات
- استلام الأدوية من الصيدلية

### 1.2 التدفق الكامل (End-to-End Flow)

```
┌─────────────────────────────────────────────────────────────────┐
│                    B2C Patient Journey                          │
└─────────────────────────────────────────────────────────────────┘

1️⃣  البحث عن صيدلية
    └─ PharmacySearchPage → PharmacyFilter → PharmacyCard
    
2️⃣  اختيار الصيدلية
    └─ PharmacyDetailPage (Tabs: PSP / Regular)
    
3️⃣  طلب الدواء
    ├─ Regular Tab → MedicationRequestCard (بحث + سلة + تأكيد)
    └─ PSP Tab → TokenRedemptionCard (صرف بالتوكن)
    
4️⃣  تأكيد الطلب
    └─ شاشة تأكيد العنوان والهاتف → إرسال للـ API
    
5️⃣  تتبع الطلب
    └─ PatientMyOrdersComponent → OrderTracker
    
6️⃣  استلام الدواء
    └─ الصيدلي يوافق → يحضر → يسلم → المريض يستلم
```

### 1.3 الأنواع الثلاثة للطلبات

| النوع | الوصف | الـ API | الجدول |
|-------|-------|---------|--------|
| **Regular** | طلب دواء عادي بدون دعم | `api/patient/orders` | `MedicationRequests` |
| **PSP Dispensation** | صرف دواء مدعوم بتوكن | `api/dispense/*` | `PSPDispensations` |
| **PSP eRX** | روشتة إلكترونية مدعومة | `api/psp/erx/*` | `PSPeRX` |

---

## 📌 الجزء الثاني: البنية المعمارية (Architecture)

### 2.1 بنية المجلدات (Folder Structure)

```
RubikCare.Full.Migration/
├── Shared.UI/
│   ├── Components/
│   │   ├── Patient/
│   │   │   └── B2CCommerce/
│   │   │       ├── PatientMyOrdersComponent.razor      # قائمة طلبات المريض
│   │   │       ├── PatientOrderTracking.razor           # تتبع طلب محدد
│   │   │       ├── OrderTracker.razor                   # مكون التتبع البصري
│   │   │       └── MedicationRequestCard.razor          # كارت طلب الدواء
│   │   └── Pharmacy/
│   │       ├── PharmacyDetailPage.razor                 # تفاصيل الصيدلية
│   │       └── PharmacySearch/
│   │           ├── PharmacySearchPage.razor             # صفحة البحث
│   │           ├── PharmacyFilter.razor                 # الفلتر
│   │           ├── PharmacyCard.razor                   # كارت الصيدلية
│   │           └── PharmacyGrid.razor                   # شبكة العرض
│   └── wwwroot/css/_pages/Patient/
│       ├── B2cCommerce/
│       │   ├── _med-request-cart.css
│       │   └── _order-tracker.css
│       └── PharmacySearch/
│           ├── _pharmacy-search-page.css
│           ├── _pharmacy-filter.css
│           ├── _pharmacy-card.css
│           ├── _pharmacy-grid.css
│           ├── _pharmacy-detail.css
│           └── _pharmacy-detail-tabs.css
│
├── Api.Web/
│   └── Controllers/
│       ├── Patient/
│       │   └── PatientOrdersController.cs               # طلبات المريض
│       ├── OrganizationsController.cs                   # الصيدليات
│       └── PharmacyController.cs                        # عمليات الصيدلية
│
└── Mobile/
    └── Features/
        └── Shared/
            └── Views/
                ├── PatientOrdersFlow.xaml.cs            # قائمة الطلبات (MAUI)
                ├── PatientOrdersBlazorPage.xaml.cs      # تتبع الطلب (MAUI)
                ├── PharmacyDetailFlow.xaml.cs           # تفاصيل الصيدلية (MAUI)
                └── PharmacySearchFlow.xaml.cs           # البحث (MAUI)
```

### 2.2 نمط التواصل بين MAUI و Blazor (Bridge Pattern)

```csharp
// ⭐ النمط الصحيح: EventCallback + Parameters
// في PharmacyDetailFlow.xaml.cs (MAUI Host)

blazorWebView.RootComponents.Add(new RootComponent
{
    Selector = "#app",
    ComponentType = typeof(PharmacyDetailPage),
    Parameters = new Dictionary<string, object?>
    {
        { "PharmacyId", _pharmacyId },
        { "TokenCode", _tokenCode ?? "" },
        { "OnGoBackAction", new Action(() => GoBack()) },
        { "OnTrackOrder", EventCallback.Factory.Create(this, async () =>
            {
                await Shell.Current.GoToAsync("my-orders");
            })
        }
    }
});
```

```razor
@* في PharmacyDetailPage.razor (Blazor Component) *@

@code {
    [Parameter] public int PharmacyId { get; set; }
    [Parameter] public string? TokenCode { get; set; }
    [Parameter] public Action? OnGoBackAction { get; set; }
    [Parameter] public EventCallback OnTrackOrder { get; set; }  ⭐

    private async Task GoToPatientOrders()
    {
        if (OnTrackOrder.HasDelegate)
            await OnTrackOrder.InvokeAsync();  ⭐
        else
            Navigation.NavigateTo("/patient/orders");
    }
}
```

---

## 📌 الجزء الثالث: المكونات الأمامية (Frontend Components)

### 3.1 PharmacySearchPage.razor

**المسار:** `Shared.UI/Components/PharmacySearch/PharmacySearchPage.razor`

**المسؤولية:** عرض صفحة البحث عن الصيدليات مع Hero Section وإحصائيات.

**الـ Parameters:**
```csharp
[Parameter] public EventCallback<int> OnPharmacySelected { get; set; }
[Parameter] public EventCallback<(int PharmacyId, string TokenCode)> OnResumePendingRequest { get; set; }
```

**الـ API Calls:**
```csharp
// جلب المدن
GET api/cities/by-country/1

// جلب الصيدليات (مع الفلاتر)
GET api/organizations/pharmacies?page=1&pageSize=9&areaId={id}&searchTerm={term}&onlyPspContracted=true

// جلب الإحصائيات
GET api/organizations/pharmacies?page=1&pageSize=1
```

**منطق الفلترة:**
```csharp
// فلتر الصيدليات المتعاقدة مع برامج المريض
if (onlyPspContracted && patientProfileId.HasValue)
{
    var patientProgramIds = await db.Set<PSPPatient>()
        .Where(p => p.PatientProfileID == patientProfileId.Value && p.Status == "ACTIVE")
        .Join(db.Set<PSPParticipation>(), ...)
        .Select(p => part.ProgramID)
        .Distinct()
        .ToListAsync();

    var contractedPharmacyIds = await db.Set<PSPParticipation>()
        .Where(p => patientProgramIds.Contains(p.ProgramID) && p.Status == "ACTIVE")
        .Select(p => p.ParticipantOrganizationID)
        .Distinct()
        .ToListAsync();

    query = query.Where(o => contractedPharmacyIds.Contains(o.OrganizationID));
}
```

### 3.2 PharmacyFilter.razor

**المسار:** `Shared.UI/Components/PharmacySearch/PharmacyFilter.razor`

**المسؤولية:** شريط الفلتر (بحث، مدينة، منطقة، PSP Contracted).

**الـ Parameters:**
```csharp
[Parameter] public EventCallback<(int? AreaId, string? SearchTerm, bool OnlyPspContracted)> OnSearch { get; set; }
[Parameter] public EventCallback<string> OnViewModeChanged { get; set; }
[Parameter] public bool IsSearching { get; set; }
```

**الحالة المحلية:**
```csharp
private int? _selectedCityId;
private int? _selectedAreaId;
private string? _searchTerm;
private bool _onlyPspContracted = false;  ⭐ فلتر PSP
private string _viewMode = "grid";
```

### 3.3 PharmacyCard.razor

**المسار:** `Shared.UI/Components/PharmacySearch/PharmacyCard.razor`

**المسؤولية:** عرض كارت الصيدلية مع البرامج المدعومة.

**الـ Parameters:**
```csharp
[Parameter] public PharmacyDto Pharmacy { get; set; }
[Parameter] public EventCallback<PharmacyDto> OnClick { get; set; }
```

**PharmacyDto:**
```csharp
public class PharmacyDto
{
    public int OrganizationID { get; set; }
    public string? OrganizationNameAr { get; set; }
    public string? OrganizationNameEn { get; set; }
    public string? FullAddressAr { get; set; }
    public string? Phone { get; set; }
    public int? AreaID { get; set; }
    public string? AreaNameAr { get; set; }
    public string? CityNameAr { get; set; }
    public List<string> SupportedProgramNames { get; set; } = new();  ⭐
}
```

### 3.4 PharmacyDetailPage.razor

**المسار:** `Shared.UI/Components/Pharmacy/PharmacyDetailPage.razor`

**المسؤولية:** عرض تفاصيل الصيدلية مع تبويبين (PSP / Regular).

**الـ Parameters:**
```csharp
[Parameter] public int PharmacyId { get; set; }
[Parameter] public string? TokenCode { get; set; }
[Parameter] public Action? OnGoBackAction { get; set; }
[Parameter] public EventCallback OnTrackOrder { get; set; }  ⭐
```

**منطق التبويبات:**
```csharp
private string _activeTab = "regular";
private PharmacyPspIntersectionDto? _matchingPrograms;

// الذكاء التلقائي في اختيار التبويب
if (_matchingPrograms?.HasMatchingPrograms == true)
    _activeTab = "psp";
else
    _activeTab = "regular";
```

**الـ API Calls:**
```csharp
// جلب تفاصيل الصيدلية
GET api/organizations/pharmacy/{PharmacyId}

// جلب البرامج المتطابقة
GET api/pharmacy-psp/matching-programs/{PharmacyId}

// التحقق من التوكن (اختياري)
GET api/dispense/can-dispense-token?pharmacyId={id}&tokenCode={code}
```

### 3.5 MedicationRequestCard.razor

**المسار:** `Shared.UI/Components/PharmacySearch/MedicationRequestCard.razor`

**المسؤولية:** كارت طلب الدواء (بحث + سلة + تأكيد + نجاح).

**الـ Parameters:**
```csharp
[Parameter] public int PharmacyId { get; set; }
[Parameter] public EventCallback OnTrackOrder { get; set; }  ⭐
```

**خطوات الـ Checkout:**
```csharp
private int _checkoutStep = 1; // 1: Search/Cart, 2: Confirm Details, 3: Success

// الخطوة 1: البحث والسلة
private async Task SearchMedications()
{
    var url = $"api/PharmaCompanyMedications/search?term={term}&pageSize=10";
    var response = await ApiService.GetAsync<MedicationSearchResponse>(url);
    _filteredMeds = response?.Data?.Where(m => m.IsActive).ToList() ?? new();
}

// الخطوة 2: تأكيد البيانات
private async Task GoToConfirmStep()
{
    _checkoutStep = 2;
    _userProfile = await ApiService.GetAsync<UserProfileDto>("api/user/profile");
    _confirmPhone = _userProfile.PhoneNumber ?? "";
    _confirmAddress = _userProfile.Address ?? "";
}

// الخطوة 3: الإرسال
private async Task SubmitFinalOrderAsync()
{
    // 1. تحديث ملف المستخدم
    await ApiService.PutAsync<object>("api/user/profile", new { 
        PhoneNumber = _confirmPhone, 
        Address = _confirmAddress 
    });

    // 2. إرسال الطلب
    var payload = new { 
        PharmacyId = PharmacyId, 
        Items = _selectedMeds.Select(m => new { 
            m.PharmaCompanyMedicationID, 
            m.Quantity 
        }).ToList(), 
        OrderNotes = _orderNotes 
    };
    var result = await ApiService.PostAsync<CheckoutResponse>("api/patient/checkout", payload);

    if (result?.Success == true)
    {
        _createdOrderNumber = result.OrderNumber ?? "";
        _checkoutStep = 3;
    }
}
```

### 3.6 PatientMyOrdersComponent.razor

**المسار:** `Shared.UI/Components/Patient/B2CCommerce/PatientMyOrdersComponent.razor`

**المسؤولية:** عرض قائمة طلبات المريض.

**الـ Parameters:**
```csharp
[Parameter] public EventCallback<PatientOrderDto> OnTrackOrder { get; set; }
```

**الـ API Call:**
```csharp
private async Task LoadPatientOrders()
{
    _isLoading = true;
    StateHasChanged();
    
    try
    {
        var response = await ApiService.GetAsync<List<PatientOrderDto>>("api/patient/orders");
        _orders = response ?? new List<PatientOrderDto>();
    }
    catch (Exception ex)
    {
        System.Diagnostics.Debug.WriteLine($"❌ LoadPatientOrders error: {ex.Message}");
        _orders = new List<PatientOrderDto>();
    }
    finally
    {
        _isLoading = false;
        StateHasChanged();
    }
}
```

**PatientOrderDto:**
```csharp
public class PatientOrderDto
{
    public int ServiceOrderID { get; set; }
    public string OrderNumber { get; set; } = "";
    public string PharmacyName { get; set; } = "";
    public decimal TotalAmount { get; set; }
    public string Status { get; set; } = "";
    public DateTime CreatedDate { get; set; }
}
```

### 3.7 OrderTracker.razor

**المسار:** `Shared.UI/Components/Patient/OrderTracker.razor`

**المسؤولية:** عرض شريط تتبع حالة الطلب بصرياً.

**الـ Parameters:**
```csharp
[Parameter] public string Status { get; set; } = "Pending";
```

**الحالات:**
```csharp
private readonly List<(int Number, string LabelKey, MedicationRequestStatus Status)> _steps = new()
{
    (1, "SHARED.ORDER_TRACKER.PENDING", MedicationRequestStatus.Pending),
    (2, "SHARED.ORDER_TRACKER.APPROVED", MedicationRequestStatus.Approved),
    (3, "SHARED.ORDER_TRACKER.READY", MedicationRequestStatus.Ready),
    (4, "SHARED.ORDER_TRACKER.DELIVERED", MedicationRequestStatus.Delivered)
};
```

---

## 📌 الجزء الرابع: الـ APIs الخلفية (Backend APIs)

### 4.1 OrganizationsController.cs

**المسار:** `Api.Web/Controllers/Organizations/OrganizationsController.cs`

#### GET `api/organizations/pharmacies`

**الوصف:** جلب قائمة الصيدليات مع الفلاتر.

**Parameters:**
```csharp
[FromQuery] int? areaId = null
[FromQuery] string? searchTerm = null
[FromQuery] int? programId = null
[FromQuery] bool onlyPspContracted = false  ⭐
[FromQuery] int page = 1
[FromQuery] int pageSize = 10
```

**Response:**
```json
{
  "items": [
    {
      "organizationID": 1258,
      "organizationNameAr": "صيدلية روبيك كير",
      "fullAddressAr": "...",
      "phone": "...",
      "areaNameAr": "...",
      "cityNameAr": "...",
      "supportedProgramNames": ["برنامج دعم مرضى السكر", "علاج الأنيميا"]
    }
  ],
  "totalCount": 27,
  "page": 1,
  "pageSize": 9,
  "totalPages": 3
}
```

**منطق الفلترة:**
```csharp
var query = db.Set<Organization>()
    .AsNoTracking()
    .Where(o => o.OrganizationTypeID == 3 && o.IsActive && !o.IsDeleted);

if (onlyPspContracted && patientProfileId.HasValue)
{
    var patientProgramIds = await db.Set<PSPPatient>()
        .Where(p => p.PatientProfileID == patientProfileId.Value && p.Status == "ACTIVE")
        .Join(db.Set<PSPParticipation>(), p => p.ParticipationID, part => part.ParticipationID, (p, part) => part.ProgramID)
        .Distinct()
        .ToListAsync();

    var contractedPharmacyIds = await db.Set<PSPParticipation>()
        .Where(p => patientProgramIds.Contains(p.ProgramID) && p.Status == "ACTIVE")
        .Select(p => p.ParticipantOrganizationID)
        .Distinct()
        .ToListAsync();

    query = query.Where(o => contractedPharmacyIds.Contains(o.OrganizationID));
}
```

#### GET `api/organizations/pharmacy/{id}`

**الوصف:** جلب تفاصيل صيدلية محددة.

**Response:**
```json
{
  "success": true,
  "data": {
    "organizationID": 1258,
    "organizationNameAr": "صيدلية روبيك كير",
    "fullAddressAr": "...",
    "phone": "...",
    "email": "...",
    "areaID": 45
  }
}
```

### 4.2 PatientOrdersController.cs

**المسار:** `Api.Web/Controllers/Patient/PatientOrdersController.cs`

#### GET `api/patient/orders`

**الوصف:** جلب طلبات المريض الحالي.

**Authorization:** Required (JWT Token)

**Response:**
```json
[
  {
    "id": 52,
    "pharmacyName": "صيدلية روبيك كير",
    "pharmacyAddress": "...",
    "pharmacyPhone": "...",
    "medicationName": "بانادول ادفانس 500 مجم 24 قرص",
    "dosage": "1 قرص كل 8 ساعات",
    "quantity": 2,
    "price": 45.50,
    "notes": "...",
    "status": "Delivered",
    "requestDate": "2026-11-18T21:38:02",
    "reviewedDate": "2026-11-18T22:00:00",
    "completedDate": "2026-11-19T10:00:00",
    "rejectionReason": null
  }
]
```

**Implementation:**
```csharp
[HttpGet]
public async Task<ActionResult<List<PatientOrderTrackingDto>>> GetMyOrders()
{
    var userId = GetCurrentUserId();
    if (userId == 0) return Unauthorized(new { Message = "الرجاء تسجيل الدخول" });

    var requests = await _medicationRequestService.GetAllAsync(
        filter: r => r.PatientId == userId,  ⭐
        orderBy: q => q.OrderByDescending(r => r.RequestDate),
        includeProperties: "Pharmacy"
    );

    var result = requests.Select(r => new PatientOrderTrackingDto
    {
        Id = r.Id,
        PharmacyName = r.Pharmacy?.OrganizationNameAr ?? "صيدلية",
        MedicationName = r.MedicationName,
        Status = r.Status.ToString(),
        // ... باقي الحقول
    }).ToList();

    return Ok(result);
}

private int GetCurrentUserId()
{
    var userProfileIdClaim = User.FindFirst("userProfileId")?.Value;
    if (!string.IsNullOrEmpty(userProfileIdClaim) && int.TryParse(userProfileIdClaim, out int userId)) 
        return userId;
    return 0;
}
```

#### GET `api/patient/orders/{id}`

**الوصف:** جلب تفاصيل طلب محدد.

#### PUT `api/patient/orders/{id}/status`

**الوصف:** تحديث حالة الطلب (يستخدمه الصيدلي).

**Request Body:**
```json
{
  "status": "Ready"
}
```

**Valid Transitions:**
```csharp
private bool IsValidStatusTransition(MedicationRequestStatus current, MedicationRequestStatus next)
{
    return current switch
    {
        MedicationRequestStatus.Approved => next == MedicationRequestStatus.Ready,
        MedicationRequestStatus.Ready => next == MedicationRequestStatus.Delivered,
        _ => false
    };
}
```

**Notifications:**
```csharp
// 1. حفظ الإشعار في قاعدة البيانات
await _dbNotificationService.CreateNotificationAsync(request.PatientId.Value, title, message, targetUrl, "MEDICATION_STATUS");

// 2. إرسال FCM (للموبايل)
await _pushNotificationService.SendToUserAsync(request.PatientId.Value, title, message);

// 3. إرسال Web Push (للـ PWA)
await _webPushSenderService.SendToUserAsync(request.PatientId.Value, title, message, targetUrl);
```

### 4.3 PharmacyController.cs

**المسار:** `Api.Web/Controllers/Pharmacy/PharmacyController.cs`

#### GET `api/pharmacy/dashboard/stats`

**الوصف:** إحصائيات لوحة تحكم الصيدلي.

#### GET `api/pharmacy/reports`

**الوصف:** تقارير الصيدلية (PSP + Regular).

#### GET `api/pharmacy/{pharmacyId}/programs`

**الوصف:** جلب برامج PSP التي تتعاقد معها الصيدلية.

**Response:**
```json
[
  {
    "programId": 2,
    "programName": "Infertility Support Program",
    "companyName": "شركة دواء",
    "status": "ACCEPTED",
    "joinDate": "2026-10-01T00:00:00",
    "invitationId": 123,
    "canInvitePatients": true
  }
]
```

---

## 📌 الجزء الخامس: نموذج البيانات (Data Models)

### 5.1 جدول Organizations

```sql
CREATE TABLE Organizations (
    OrganizationID INT PRIMARY KEY IDENTITY,
    OrganizationNameAr NVARCHAR(200),
    OrganizationNameEn NVARCHAR(200),
    OrganizationTypeID INT,  -- 1: Personal, 2: PharmaCompany, 3: Pharmacy, 4: Clinic
    FullAddressAr NVARCHAR(500),
    Phone NVARCHAR(50),
    Email NVARCHAR(100),
    AreaID INT,
    Latitude NVARCHAR(50),
    Longitude NVARCHAR(50),
    IsActive BIT DEFAULT 1,
    IsDeleted BIT DEFAULT 0,
    CreateDate DATETIME DEFAULT GETDATE()
);
```

### 5.2 جدول MedicationRequests

```sql
CREATE TABLE MedicationRequests (
    Id INT PRIMARY KEY IDENTITY,
    PatientId INT,  ⭐ FK to UserProfile
    PharmacyId INT,  ⭐ FK to Organizations
    MedicationName NVARCHAR(200),
    Dosage NVARCHAR(100),
    Quantity INT,
    Price DECIMAL(10,2),
    Notes NVARCHAR(1000),
    Status INT,  -- 0: Pending, 1: Approved, 2: Rejected, 3: Cancelled, 4: Ready, 5: Delivered
    RequestDate DATETIME DEFAULT GETDATE(),
    ReviewedDate DATETIME,
    CompletedDate DATETIME,
    RejectionReason NVARCHAR(500)
);
```

### 5.3 جدول PSPDispensations

```sql
CREATE TABLE PSPDispensations (
    DispensationID INT PRIMARY KEY IDENTITY,
    DispensationCode NVARCHAR(50),
    ERxID INT,  ⭐ FK to PSPeRX
    PlanID INT,  ⭐ FK to PSPDispensationPlan
    QuantityDispensed INT,
    UnitPrice DECIMAL(10,2),
    TotalAmount DECIMAL(10,2),
    PatientPayment DECIMAL(10,2),
    PSPCoverage DECIMAL(10,2),
    TokenCode NVARCHAR(50),
    TokenGeneratedDate DATETIME,
    TokenExpiryDate DATETIME,
    TokenUsed BIT DEFAULT 0,
    TokenUsedDate DATETIME,
    PharmacyID INT,  ⭐ FK to Organizations
    PharmacistID INT,  ⭐ FK to UserProfile
    Status NVARCHAR(20),  -- PENDING, GENERATED, PENDING_APPROVAL, APPROVED, REJECTED, DISPENSED, RECEIVED
    DispenseDate DATETIME,
    RefillNumber INT DEFAULT 0,
    IsEarlyRequest BIT DEFAULT 0,
    DispensationNotes NVARCHAR(1000),
    RejectionReason NVARCHAR(500),
    CreatedDate DATETIME DEFAULT GETDATE()
);
```

### 5.4 جدول PSPeRX

```sql
CREATE TABLE PSPeRX (
    ERxID INT PRIMARY KEY IDENTITY,
    ERxCode NVARCHAR(50),
    CaseID INT,  ⭐ FK to PSPPatients
    DoctorID INT,  ⭐ FK to UserProfile
    ProgramID INT,  ⭐ FK to PSPPrograms
    MedicationID INT,  ⭐ FK to Medications
    Dosage NVARCHAR(100),
    Frequency NVARCHAR(100),
    Instructions NVARCHAR(200),
    TotalQuantity INT,
    Status NVARCHAR(20) DEFAULT 'ACTIVE',
    PrescriptionDate DATETIME DEFAULT GETDATE(),
    CompletionDate DATETIME,
    Notes NVARCHAR(1000)
);
```

### 5.5 جدول PSPPatients

```sql
CREATE TABLE PSPPatients (
    CaseID INT PRIMARY KEY IDENTITY,
    ParticipationID INT,  ⭐ FK to PSPParticipations
    PatientProfileID INT,  ⭐ FK to UserProfile
    DiagnosisDetails NVARCHAR(2000),
    DiagnosisCode NVARCHAR(100),
    Status NVARCHAR(20) DEFAULT 'ACTIVE',  -- ACTIVE, COMPLETED, WITHDRAWN, SUSPENDED
    EnrollmentDate DATETIME DEFAULT GETDATE(),
    StartDate DATETIME,
    EndDate DATETIME,
    CaseManagerID INT,
    NextFollowUpDate DATETIME,
    AdherenceRate DECIMAL(5,2),
    Notes NVARCHAR(4000),
    ApprovedByClinicID INT,
    ClinicApprovalDate DATETIME,
    InvitedByOrganizationID INT,
    InvitationMethod NVARCHAR(20) DEFAULT 'CLINIC_INVITE',
    InvitedByUserID INT,
    InvitationDate DATETIME,
    InvitationAcceptedDate DATETIME
);
```

### 5.6 جدول PSPParticipations

```sql
CREATE TABLE PSPParticipations (
    ParticipationID INT PRIMARY KEY IDENTITY,
    ParticipantOrganizationID INT,  ⭐ FK to Organizations (Pharmacy or Clinic)
    ProgramID INT,  ⭐ FK to PSPPrograms
    InvitationID INT,  ⭐ FK to PSPInvitations
    JoinMethod NVARCHAR(20) DEFAULT 'INVITATION',  -- INVITATION, SELF_ENROLL, SYSTEM
    Status NVARCHAR(20) DEFAULT 'ACTIVE',  -- ACTIVE, SUSPENDED, WITHDRAWN, PENDING
    JoinDate DATETIME DEFAULT GETDATE(),
    LastUpdatedDate DATETIME,
    CanInvitePatients BIT DEFAULT 1,
    CanInviteOrganizations BIT DEFAULT 0,
    Notes NVARCHAR(1000)
);
```

### 5.7 جدول PSPPrograms

```sql
CREATE TABLE PSPPrograms (
    ProgramID INT PRIMARY KEY IDENTITY,
    ProgramCode NVARCHAR(50),
    ProgramNameAr NVARCHAR(200),
    ProgramNameEn NVARCHAR(200),
    PharmaCompanyID INT,  ⭐ FK to Organizations
    Category NVARCHAR(100),
    Budget DECIMAL(15,2),
    BudgetUsed DECIMAL(15,2) DEFAULT 0,
    IsActive BIT DEFAULT 1,
    StartDate DATETIME,
    EndDate DATETIME,
    CreatedDate DATETIME DEFAULT GETDATE()
);
```

---

## 📌 الجزء السادس: حالات الاستخدام (Use Cases)

### 6.1 UC-01: البحث عن صيدلية

**Actor:** مريض  
**Precondition:** مسجل دخول  
**Flow:**
1. المريض يفتح صفحة البحث عن الصيدليات
2. النظام يعرض Hero Section مع إحصائيات (عدد الصيدليات والمدن)
3. المريض يدخل كلمة بحث أو يختار مدينة/منطقة
4. النظام يستدعي `GET api/organizations/pharmacies` مع الفلاتر
5. النظام يعرض النتائج في Grid/List View
6. المريض يضغط على صيدلية

**Postcondition:** الانتقال إلى صفحة تفاصيل الصيدلية

### 6.2 UC-02: طلب دواء عادي (Regular)

**Actor:** مريض  
**Precondition:** في صفحة تفاصيل الصيدلية، تبويب Regular نشط  
**Flow:**
1. المريض يبحث عن دواء بالاسم
2. النظام يستدعي `GET api/PharmaCompanyMedications/search`
3. المريض يختار أدوية ويحدد الكميات
4. النظام يحسب الإجمالي التقديري
5. المريض يضغط "مراجعة البيانات وإرسال"
6. النظام يجلب بيانات المريض (الهاتف والعنوان) من `GET api/user/profile`
7. المريض يؤكد/يعدل البيانات
8. النظام يستدعي `PUT api/user/profile` لتحديث البيانات
9. النظام يستدعي `POST api/patient/checkout` لإرسال الطلب
10. النظام يعرض شاشة النجاح مع رقم الطلب
11. المريض يضغط "متابعة الطلب"

**Postcondition:** الانتقال إلى صفحة قائمة الطلبات

### 6.3 UC-03: صرف دواء مدعوم (PSP)

**Actor:** مريض  
**Precondition:** في صفحة تفاصيل الصيدلية، تبويب PSP نشط، الصيدلية متعاقدة مع برنامج المريض  
**Flow:**
1. النظام يعرض TokenRedemptionCard
2. المريض يدخل كود التوكن أو يمسحه بـ QR
3. النظام يستدعي `GET api/dispense/can-dispense-token` للتحقق
4. إذا صالح، النظام يعرض تفاصيل الصرف
5. المريض يؤكد الصرف
6. النظام يستدعي `POST api/dispense/use-token`
7. النظام يعرض شاشة النجاح

**Postcondition:** تحديث حالة التوكن إلى DISPENSED

### 6.4 UC-04: تتبع طلب

**Actor:** مريض  
**Precondition:** في صفحة قائمة الطلبات  
**Flow:**
1. المريض يضغط زر "تتبع" بجانب طلب
2. النظام يستدعي `EventCallback OnTrackOrder`
3. MAUI Shell ينتقل إلى `order-tracking?orderId={id}&status={status}`
4. النظام يعرض OrderTracker مع الحالة الحالية
5. النظام يستدعي `GET api/patient/orders/{id}` للتفاصيل

**Postcondition:** عرض شريط التقدم البصري

### 6.5 UC-05: الصيدلي يوافق على طلب

**Actor:** صيدلي  
**Precondition:** في لوحة تحكم الصيدلية، طلب في حالة Pending  
**Flow:**
1. الصيدلي يرى الطلب في قائمة "طلبات التوريد"
2. الصيدلي يحدد السعر ويضغط "موافقة"
3. النظام يستدعي `PUT api/pharmacy/medication-request/{id}/approve`
4. النظام يرسل إشعار للمريض (FCM + Web Push)
5. حالة الطلب تتغير إلى Approved

**Postcondition:** المريض يستلم إشعاراً، الطلب يظهر في حالة "تمت الموافقة"

---

## 📌 الجزء السابع: معالجة الأخطاء (Error Handling)

### 7.1 أنماط معالجة الأخطاء في Blazor Components

```csharp
private async Task LoadDataAsync()
{
    _isLoading = true;
    StateHasChanged();  ⭐ إجبار الرسم
    
    try
    {
        var response = await ApiService.GetAsync<List<T>>("api/...");
        _data = response ?? new List<T>();  ⭐ Null-safety
    }
    catch (Exception ex)
    {
        System.Diagnostics.Debug.WriteLine($"❌ LoadDataAsync error: {ex.Message}");
        _data = new List<T>();  ⭐ ضمان عدم انهيار الواجهة
    }
    finally
    {
        _isLoading = false;
        StateHasChanged();  ⭐ إجبار الرسم دائماً
    }
}
```

### 7.2 أنماط معالجة الأخطاء في MAUI

```csharp
private async void GoBack()
{
    try
    {
        await Shell.Current.GoToAsync("pharmacy-search");
    }
    catch (Exception ex)
    {
        System.Diagnostics.Debug.WriteLine($"GoBack error: {ex.Message}");
        await Shell.Current.GoToAsync("pharmacy-search");  ⭐ Fallback
    }
}
```

### 7.3 حالات الخطأ الشائعة

| الحالة | السبب | الحل |
|--------|-------|------|
| شاشة بيضاء في MAUI | استثناء صامت في Blazor | إضافة `try/catch/finally` + `StateHasChanged()` |
| 404 NotFound | مسار API خاطئ | التحقق من `[Route]` في Controller |
| 0 items | `PatientId` لا يطابق | التحقق من حفظ `PatientId` في Checkout |
| زر لا يعمل | EventCallback غير مربوط | التحقق من `Parameters` في `RootComponents.Add` |
| JavaProxyThrowable | Action async بدون MainThread | استخدام `MainThread.BeginInvokeOnMainThread` |

---

## 📌 الجزء الثامن: الأمان (Security)

### 8.1 المصادقة (Authentication)

جميع APIs الخاصة بالمريض تتطلب JWT Token:

```csharp
[Authorize]
[ApiController]
[Route("api/patient/orders")]
public class PatientOrdersController : ControllerBase
{
    private int GetCurrentUserId()
    {
        var userProfileIdClaim = User.FindFirst("userProfileId")?.Value;
        if (!string.IsNullOrEmpty(userProfileIdClaim) && int.TryParse(userProfileIdClaim, out int userId)) 
            return userId;
        return 0;
    }
}
```

### 8.2 الصلاحيات (Authorization)

```csharp
// المريض يرى طلباته فقط
var requests = await _medicationRequestService.GetAllAsync(
    filter: r => r.PatientId == userId  ⭐
);

// الصيدلي يرى طلبات صيدليته فقط
var requests = await _medicationRequestService.GetAllAsync(
    filter: r => r.PharmacyId == pharmacyId  ⭐
);
```

### 8.3 حماية البيانات الحساسة

```csharp
// تشفير البيانات عند حذف الحساب
profile.FirstName = "[محذوف]";
profile.Email = $"[deleted_{userProfileId}@anonymous.com]";
profile.PhoneNumber = "[محذوف]";
```

---

## 📌 الجزء التاسع: الاختبار (Testing)

### 9.1 Checklist اختبار B2C Flow

- [ ] البحث عن صيدلية يعرض نتائج صحيحة
- [ ] فلتر المدينة/المنطقة يعمل
- [ ] فلتر "الصيدليات المتعاقدة مع برامجي" يعمل
- [ ] كارت الصيدلية يعرض البرامج المدعومة
- [ ] صفحة تفاصيل الصيدلية تفتح
- [ ] تبويب PSP يظهر إذا كانت الصيدلية متعاقدة
- [ ] تبويب Regular يعمل دائماً
- [ ] البحث عن دواء يعرض نتائج
- [ ] إضافة دواء للسلة يعمل
- [ ] تعديل الكمية يعمل
- [ ] حساب الإجمالي صحيح
- [ ] شاشة تأكيد البيانات تظهر
- [ ] تحديث بيانات المستخدم يعمل
- [ ] إرسال الطلب ينجح
- [ ] شاشة النجاح تظهر مع رقم الطلب
- [ ] زر "متابعة الطلب" ينقل للقائمة
- [ ] قائمة الطلبات تعرض الطلبات الصحيحة
- [ ] زر "تتبع" يفتح صفحة التتبع
- [ ] OrderTracker يعرض الحالة الصحيحة
- [ ] الإشعارات تصل عند تغيير الحالة

### 9.2 اختبار MAUI Specific

- [ ] `RootComponents.Add` يعمل في Constructor
- [ ] `EventCallback` يمرر الأحداث بشكل صحيح
- [ ] `Shell.Current.GoToAsync` ينتقل للمسار الصحيح
- [ ] `QueryProperty` يستقبل المعاملات
- [ ] `BlazorWebView` لا ينهار عند تغيير الصفحات
- [ ] SafeArea padding يعمل (bottom=48dp)

---

## 📌 الجزء العاشر: الدروس المستفادة (Lessons Learned)

### 10.1 قواعد ذهبية

1. **لا تستخدم `@inject ApiService` في MAUI BlazorWebView** → استخدم `[Parameter]` أو Singleton
2. **EventCallback يجب أن يُمرر في Parameters** → وإلا لن يعمل الزر
3. **StateHasChanged() في finally** → لضمان رسم الواجهة دائماً
4. **Null-safety في الـ DTOs** → `_data = response ?? new List<T>()`
5. **فصل المسؤوليات** → Shared.UI للمكونات، MAUI/PWA للاستضافة والتوجيه

### 10.2 أنماط فاشلة يجب تجنبها

```csharp
// ❌ خطأ - سي失败 بصمت في MAUI
@inject IApiService ApiService

// ✅ صحيح
[Parameter] public IApiService? ApiService { get; set; }

// ❌ خطأ - لن يعمل الزر
blazorWebView.RootComponents.Add(new RootComponent
{
    ComponentType = typeof(MyComponent)
    // Parameters ناقصة
});

// ✅ صحيح
blazorWebView.RootComponents.Add(new RootComponent
{
    ComponentType = typeof(MyComponent),
    Parameters = new Dictionary<string, object?>
    {
        { "OnButtonClick", EventCallback.Factory.Create(this, async () => { ... }) }
    }
});
```

---

## 📝 سجل التغييرات

| الإصدار | التاريخ | التغييرات |
|---------|---------|-----------|
| 1.0 | 22 نوفمبر 2026 | الإصدار الأولي - توثيق كامل لـ B2C Flow |

---

**© 2026 RubikCare — للاستخدام الداخلي**

**نهاية الوثيقة ✅**
