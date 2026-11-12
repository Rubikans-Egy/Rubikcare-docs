
# 13 — MAUI (تطوير MAUI + BlazorWebView)

**الإصدار:** 4.0 (Proven Patterns Edition)
**آخر تحديث:** 5 نوفمبر 2026
**الحالة:** ✅ محدّث بالأنماط الناجحة (بعد جلسة 4-5 نوفمبر)
**الجمهور:** فريق التطوير + AI Assistants
**المرجع الإلزامي:** يُقرأ قبل أي تطوير لصفحة MAUI

---

## 📌 الجزء صفر: الفلسفة

### 0.1 لماذا BlazorWebView؟

| الميزة | XAML التقليدي | BlazorWebView |
|---|---|---|
| لغة التطوير | لغة مختلفة عن Web | نفس HTML/CSS المستخدم في Web |
| ربط البيانات | مشاكل Binding متكررة | لا Binding — استخدام Razor |
| مشاركة المكونات | صعبة | مشاركة مباشرة عبر RCL |
| منحنى التعلم | منفصل تماماً | نفس مهارات Blazor |

### 0.2 المبادئ الأساسية

- **مشاركة المكونات** — `Shared.UI` مشترك بين Web + PWA + MAUI.
- **`CustomBlazorWebView`** — استخدمه بحذر (انظر § 10).
- **`Action` بدلاً من `EventCallback`** — نمط MAUI (انظر § 9).
- **لا `@inject` للخدمات MAUI** — استخدم `[Parameter]` (انظر § 2).
- **`SafeAsyncAction`** — لكل `Action<T>` مع async.
- **`Singleton` للتواصل** — `CurrentOrganizationState` + `CurrentUserProfileState` (انظر § 6.5).

---

## 🚨 الجزء الأول: النمط الناجح (Proven Patterns — 5 نوفمبر 2026)

### 1.0 مقدمة: لماذا هذا القسم؟

**هذا القسم هو "الدرس المستفاد" من جلسة 4-5 نوفمبر 2026.** في تلك الجلسة، فشلت **كل المحاولات** لاستخدام `DashboardPage.razor` (المشترك) كصفحة رئيسية (`ShellContent`) في MAUI — **لأن الوثيقة السابقة (v3.3) قدمت أنماطاً لم تنجح** (`RootComponent` في XAML + `Parameters` ديناميكية + `CurrentUserProfileState`).

**الآن، بعد النجاح، نُوثّق النمط الناجح بدقة.** هذا القسم **إلزامي** قبل أي تطوير لصفحة MAUI.

### 1.1 القاعدة الذهبية (The Golden Rule)

> **"الصفحات الرئيسية (`ShellContent`) = XAML محلي (وليس `BlazorWebView`). الصفحات الفرعية (`Routing.RegisterRoute`) = `BlazorWebView` + `RootComponents.Add` في `OnAppearing` مع `Parameters`. التواصل بين MAUI و Blazor = `Singleton Services` (مع `Set...` في `Constructor`)."**

### 1.2 جدول القرار السريع

| نوع الصفحة | `ShellContent` (Root Page) | `Routing.RegisterRoute` (Detail Page) |
|---|---|---|
| **XAML محلي** (مثل `DashboardPage.xaml` القديمة) | ✅ **موصى به** (لا مشاكل). | ✅ ممكن (لكن نادر). |
| **`BlazorWebView` + `<RootComponent>` في XAML** | ⚠️ **فقط** إذا كانت لا تحتاج `Parameters` ديناميكية (مثل `ClinicDashboardPage`). | ❌ لا (استخدم `RootComponents.Add`). |
| **`BlazorWebView` + `RootComponents.Add` في `OnAppearing`** | ❌ **فاشل** (BlazorWebView لا يعيد تحميل المكون). | ✅ **موصى به** (مثل `PatientPointsFlow`). |
| **`BlazorWebView` + `RootComponents.Add` في `Constructor`** | ⚠️ يحتاج بيانات متزامنة (من `Preferences`). | ✅ ممكن. |
| **`BlazorWebView` + `CurrentUserProfileState` + `<RootComponent>` في XAML** | ✅ **النمط الناجح** (مثل `DashboardFlowPage`). | ✅ ممكن. |

### 1.3 الأنماط الناجحة (5 أنماط — مع أمثلة)

#### 🌟 النمط 1: XAML محلي (للصفحات الرئيسية البسيطة)

**متى:** الصفحة الرئيسية لا تحتاج `BlazorWebView` (أو تحتاج XAML بسيط).

**مثال:** `DashboardPage.xaml` (القديمة — قبل التحويل).

**البنية:**
```xml
<ContentPage x:Class="...">
    <ScrollView>
        <VerticalStackLayout>
            <!-- عناصر XAML -->
        </VerticalStackLayout>
    </ScrollView>
</ContentPage>
```

**الـ ViewModel:** `DashboardPageViewModel` (مع `ICommand` + `INotifyPropertyChanged`).

**المزايا:** ✅ لا مشاكل `BlazorWebView`. ✅ أداء عالي. ✅ تحكم كامل.
**العيوب:** ❌ لا مشاركة كود مع PWA. ❌ صيانة مزدوجة.

---

#### 🌟 النمط 2: `BlazorWebView` + `<RootComponent>` في XAML (للصفحات الرئيسية التي تعتمد على Singleton)

**متى:** الصفحة الرئيسية تحتاج `BlazorWebView`، **لكن** لا تحتاج `Parameters` ديناميكية (تعتمد على `Singleton Service`).

**أمثلة ناجحة:**
- `ClinicDashboardPage.xaml` (يعتمد على `CurrentOrganizationState`).
- `PharmacyDashboardPage.xaml` (يعتمد على `CurrentOrganizationState`).
- `PharmaCompanyDashboardPage.xaml` (يعتمد على `PharmaCompanyNavigationBridge`).
- **`DashboardFlowPage.xaml`** (يعتمد على `CurrentUserProfileState` — بعد النجاح).

**البنية (`ClinicDashboardPage.xaml`):**
```xml
<ContentPage x:Class="RubikCare.Mobile.Features.ClinicManagement.Views.ClinicDashboardPage">
    <b:BlazorWebView x:Name="blazorWebView" HostPage="wwwroot/index.html">
        <b:BlazorWebView.RootComponents>
            <b:RootComponent Selector="#app" ComponentType="{x:Type clinic:ClinicDashboard}" />
        </b:BlazorWebView.RootComponents>
    </b:BlazorWebView>
</ContentPage>
```

**البنية (`ClinicDashboardPage.xaml.cs`):**
```csharp
public partial class ClinicDashboardPage : ContentPage
{
    [QueryProperty(nameof(ClinicId), "ClinicId")]
    public int ClinicId { get; set; }

    public ClinicDashboardPage()
    {
        InitializeComponent();
        SetupNavigation();
    }

    protected override void OnAppearing()
    {
        base.OnAppearing();
        if (ClinicId > 0)
        {
            var clinicState = IPlatformApplication.Current?.Services?.GetService<CurrentOrganizationState>();
            clinicState?.SetClinicId(ClinicId);  // ⭐ Set قبل OnInitializedAsync
        }
    }
}
```

**المكون المشترك (`ClinicDashboard.razor`):**
```csharp
@inject CurrentOrganizationState ClinicState

protected override void OnInitialized()
{
    _currentClinicId = ClinicState.ClinicId;  // ⭐ يقرأ من Singleton
    ClinicState.OnChange += OnStateChange;
}

protected override async Task OnInitializedAsync()
{
    if (_currentClinicId > 0)
        await LoadDataAsync();
    else
        _isLoading = false;
}

private void OnStateChange()
{
    _currentClinicId = ClinicState.ClinicId;
    InvokeAsync(async () => { if (_currentClinicId > 0) await LoadDataAsync(); });
}
```

**المزايا:** ✅ يعمل. ✅ مشاركة كود. ✅ لا مشاكل `Parameters`.
**العيوب:** ❌ يحتاج `Singleton Service` + `Set...` في `Constructor` (أو `OnAppearing` — قبل `OnInitializedAsync`).

---

#### 🌟 النمط 3: `BlazorWebView` + `RootComponents.Add` في `OnAppearing` (للصفحات الفرعية)

**متى:** الصفحة **فرعية** (`Routing.RegisterRoute`) تحتاج `Parameters` ديناميكية.

**أمثلة ناجحة:**
- `PatientPointsFlow.xaml` (يستقبل `UserProfileId` من `Query`).
- `MedicineGuideFlow.xaml` (يستقبل `caseId` + `programId` من `Query`).
- `PspDetailFlow.xaml` (يستقبل `patientId` + `participationId` + `programId` من `Query`).

**البنية (`PatientPointsFlow.xaml`):**
```xml
<ContentPage x:Class="RubikCare.Mobile.Features.PSP.Patient.Views.PatientPointsFlow">
    <BlazorWebView x:Name="blazorWebView" HostPage="wwwroot/index.html" />
</ContentPage>
```

**البنية (`PatientPointsFlow.xaml.cs`):**
```csharp
[QueryProperty(nameof(UserProfileIdRaw), "userProfileId")]
public partial class PatientPointsFlow : ContentPage
{
    public string? UserProfileIdRaw { get; set; }
    private int UserProfileId => int.TryParse(UserProfileIdRaw, out var id) ? id : 0;

    public PatientPointsFlow()
    {
        InitializeComponent();
    }

    protected override void OnAppearing()
    {
        base.OnAppearing();
        LoadBlazorComponent();
    }

    private void LoadBlazorComponent()
    {
        var effectiveUserId = UserProfileId > 0 ? UserProfileId : _sessionService.PatientId;
        
        blazorWebView.RootComponents.Clear();
        blazorWebView.RootComponents.Add(new RootComponent
        {
            Selector = "#app",
            ComponentType = typeof(PatientPointsDashboard),
            Parameters = new Dictionary<string, object?>
            {
                { "UserProfileId", effectiveUserId },
                { "ApiService", _apiService },
                { "ShowBackButton", true }
            }
        });
    }
}
```

**المزايا:** ✅ يعمل. ✅ `Parameters` ديناميكية. ✅ `RootComponents.Add` في `OnAppearing` يعمل (لأن المكون يبدأ بعد `OnAppearing`).
**العيوب:** ❌ لا يمكن استخدامه كصفحة رئيسية (`ShellContent`).

---

#### 🌟 النمط 4: `BlazorWebView` + `RootComponents.Add` في `Constructor` (للصفحات الرئيسية التي تحتاج `Parameters`)

**متى:** الصفحة **رئيسية** (`ShellContent`) تحتاج `Parameters` (لكن `Parameters` غير متزامنة — من `Preferences`).

**مثال:** `DashboardFlowPage.xaml` — بعد النجاح (4-5 نوفمبر).

**البنية (`DashboardFlowPage.xaml`):**
```xml
<ContentPage x:Class="RubikCare.Mobile.Features.PublicUser.Views.DashboardFlowPage">
    <b:BlazorWebView x:Name="blazorWebView" HostPage="wwwroot/index.html">
        <b:BlazorWebView.RootComponents>
            <b:RootComponent Selector="#app" ComponentType="{x:Type sharedPages:DashboardPage}" />
        </b:BlazorWebView.RootComponents>
    </b:BlazorWebView>
</ContentPage>
```

**البنية (`DashboardFlowPage.xaml.cs`):**
```csharp
public partial class DashboardFlowPage : ContentPage
{
    private readonly CurrentUserProfileState _userState;

    public DashboardFlowPage()
    {
        InitializeComponent();
        _userState = IPlatformApplication.Current!.Services.GetRequiredService<CurrentUserProfileState>();

        // ⭐ قراءة UserProfileId من Preferences (متزامن — قبل OnInitializedAsync)
        var userId = Preferences.Get("user_profile_id", 0);
        var userName = Preferences.Get("user_full_name", "مستخدم");

        if (userId > 0)
        {
            _userState.SetUserProfile(userId, userName);  // ⭐ قبل OnInitializedAsync
        }
    }

    protected override async void OnAppearing()
    {
        base.OnAppearing();

        if (_userState.UserProfileId <= 0)
        {
            var session = await _sessionService.GetSessionAsync();
            if (session != null)
            {
                _userState.SetUserProfile(session.UserProfileId, session.FullNameAr);
                Preferences.Set("user_profile_id", session.UserProfileId);
                Preferences.Set("user_full_name", session.FullNameAr);
            }
        }
    }
}
```

**المكون المشترك (`DashboardPage.razor`):**
```csharp
@inject CurrentUserProfileState UserState

protected override void OnInitialized()
{
    UserState.OnChange += OnUserStateChanged;
}

protected override async Task OnInitializedAsync()
{
    await LoadTranslationsAsync();
    _translationsLoaded = true;

    if (UserState.UserProfileId > 0)
    {
        _dataLoaded = true;
        await LoadAllDataAsync();
    }
    else
    {
        _isLoading = false;
    }
}

private async void OnUserStateChanged()
{
    if (UserState.UserProfileId > 0 && !_dataLoaded)
    {
        _dataLoaded = true;
        await InvokeAsync(async () => { await LoadAllDataAsync(); StateHasChanged(); });
    }
}
```

**المزايا:** ✅ يعمل كصفحة رئيسية. ✅ لا `Parameters` ديناميكية. ✅ `Preferences` (متزامن).
**العيوب:** ❌ يحتاج `Preferences.Set` (في `AppShell.InitializeAsync`).

**⚠️ هذا هو النمط الناجح الذي يجب استخدامه للصفحات الرئيسية التي تحتاج بيانات مستخدم.**

---

#### 🌟 النمط 5: `CurrentUserProfileState` (Singleton) — للتواصل بين MAUI و Blazor

**متى:** تحتاج تمرير بيانات (مثل `UserProfileId`) من MAUI إلى `BlazorWebView` **دون** `Parameters` ديناميكية.

**⚠️ هذا النمط مرتبط بالنمط 4.**

**البنية (`Shared.UI/Services/CurrentUserProfileState.cs`):**
```csharp
public class CurrentUserProfileState
{
    private int _userProfileId;
    private string _userFullNameAr = string.Empty;

    public int UserProfileId => _userProfileId;
    public string UserFullNameAr => _userFullNameAr;
    public string UserInitials => GetInitials(_userFullNameAr);

    public event Action? OnChange;

    public void SetUserProfile(int userProfileId, string userFullNameAr)
    {
        if (_userProfileId != userProfileId)
        {
            _userProfileId = userProfileId;
            _userFullNameAr = userFullNameAr ?? string.Empty;
            OnChange?.Invoke();
        }
        else
        {
            _userFullNameAr = userFullNameAr ?? string.Empty;
        }
    }

    // ...
}
```

**المزايا:** ✅ يعمل. ✅ لا `Parameters` ديناميكية.
**العيوب:** ❌ يحتاج `SetUserProfile` في `Constructor` (قبل `OnInitializedAsync`).

**⚠️ تحذير:** `CurrentUserProfileState` **Singleton** — **لا يعمل** إذا كان `BlazorWebView` له حاوية DI مستقلة (§ 2.1). **لكن** — في `DashboardFlowPage`، **يعمل** لأن `RootComponent` في XAML + `CurrentUserProfileState` Singleton = **نفس النسخة** (لأن `BlazorWebView` في `DashboardFlowPage` يحقن `CurrentUserProfileState` من حاوية MAUI — وليس من حاوية Blazor). **⚠️ هذا ليس موثقاً في `13-MAUI` السابق — لكنه يعمل.**

---

---

#### 🌟 النمط 8: NavigationBridge + PushAsync + RootComponents.Add (للصفحات الفرعية المعقدة)

**متى:** الصفحة الفرعية تحتاج `Parameters` ديناميكية، وتحتاج أن تظهر كصفحة كاملة (مع Header / Breadcrumb / زر رجوع)، وليس كـ Modal.

**المثال الناجح:** `CommerceProductDetailsFlow` (جلسة Commerce، 11-12 نوفمبر 2026).

**البنية (Shared.UI):**
- واجهة `ICommerceNavigationBridge` (في `Shared.UI/Services/`).
- المكون `ProductDetailsPage.razor` (مع `[Parameter] MedicationId`, `CompanyOrgId`).

**البنية (Mobile):**
- تطبيق `CommerceNavigationBridge` (في `Mobile/Services/`).
- صفحة `CommerceProductDetailsFlow.xaml` (مع `Shell.NavBarIsVisible="True"`).
- تسجيل في `MauiProgram.cs`.

**البنية (CommerceProductDetailsFlow.xaml.cs):**
```csharp
public partial class CommerceProductDetailsFlow : ContentPage
{
    private readonly int _medicationId;
    private readonly int _companyOrgId;

    public CommerceProductDetailsFlow(int medicationId, int companyOrgId)
    {
        InitializeComponent();
        _medicationId = medicationId;
        _companyOrgId = companyOrgId;

        BlazorView.RootComponents.Add(new RootComponent
        {
            Selector = "#app",
            ComponentType = typeof(ProductDetailsPage),
            Parameters = new Dictionary<string, object>
            {
                { "MedicationId", _medicationId },
                { "CompanyOrgId", _companyOrgId },
                { "OnBack", EventCallback.Factory.Create(this, async () => await Navigation.PopAsync()) }
            }
        });
    }
}
```
لمزايا:

✅ يعمل بدون Reload (لأن PushAsync يحتفظ بـ Navigation Stack).

✅ يظهر Shell.NavBar (بفضل Shell.NavBarIsVisible="True").

✅ Shared.UI لا تعتمد على MAUI (بفضل Bridge).

✅ Parameters تُمرر بشكل صحيح (بفضل RootComponents.Add).

العيوب:

⚠️ يحتاج واجهة (Bridge) + تطبيق + تسجيل.

⚠️ OnBack يجب أن يكون EventCallback (وليس Action)، لتجنب JavaProxyThrowable.

---
### 1.4 الأنماط الفاشلة (Failed Patterns — لا تستخدمها)

#### ❌ النمط 1: `BlazorWebView` + `<RootComponent>` في XAML مع `Parameters` ديناميكية

**المشكلة:** `RootComponent` في XAML **لا يدعم `Parameters` ديناميكية** (فقط `ComponentType`).

**النتيجة:** `UserProfileId` = 0 دائماً.

**البديل:** استخدم النمط 3 (`RootComponents.Add`) للصفحات الفرعية، أو النمط 4 (`CurrentUserProfileState`) للصفحات الرئيسية.

---

#### ❌ النمط 2: `BlazorWebView` + `RootComponents.Add` في `OnAppearing` كصفحة رئيسية (`ShellContent`)

**المشكلة:** `BlazorWebView` **لا يعيد تحميل المكون** بعد `RootComponents.Add` (إذا كان `ShellContent`).

**النتيجة:** المكون لا يُحمَّل (الصفحة بيضاء).

**البديل:** استخدم النمط 4 (`CurrentUserProfileState` + `<RootComponent>` في XAML) للصفحات الرئيسية.

---

#### ❌ النمط 3: `CustomBlazorWebView` + `RootComponents.Add` في `OnAppearing` كصفحة رئيسية

**المشكلة:** `CustomBlazorWebView` (مع `ForceRender`) **يعيد تعيين `RootComponents`** بعد `Add` — مما يسبب **اختفاء المكون**.

**النتيجة:** الصفحة بيضاء (أو المكون يختفي).

**البديل:** استخدم `BlazorWebView` العادي (وليس `CustomBlazorWebView`) للصفحات الرئيسية.

---

#### ❌ النمط 4: `OnComponentReady` (لتمرير `Parameters` بعد `OnAfterRender`)

**المشكلة:** `OnAfterRender` **لا يُستدعى** إذا لم يُحمَّل المكون (بسبب `RootComponents.Add` في `OnAppearing` كصفحة رئيسية).

**النتيجة:** `OnComponentReady` لا يُستدعى.

**البديل:** استخدم `CurrentUserProfileState` (النمط 5) — لا حاجة لـ `OnComponentReady`.

---
#### ❌ النمط 5: BlazorWebView + <RootComponent> في XAML بدون Parameters (لصفحة فرعية تحتاج Parameters)

**المشكلة:** الصفحة الفرعية (`CommerceProductDetailsFlow`) تستخدم `<RootComponent>` في XAML، لكنها تحتاج `MedicationId` و `CompanyOrgId`.

**النتيجة:** `Parameters` = `null`، والصفحة تظهر فارغة (أو "Product not found").

**البديل:** استخدم `RootComponents.Add` في `Constructor` (مع `Parameters`) — **للصفحات الفرعية فقط**.

**مثال من جلسة Commerce:**
```csharp
// ❌ خطأ — ProductDetailsPage لن تستقبل Parameters
<BlazorWebView>
    <BlazorWebView.RootComponents>
        <RootComponent Selector="#app" ComponentType="{x:Type shared:ProductDetailsPage}" />
    </BlazorWebView.RootComponents>
</BlazorWebView>

// ✅ صحيح — RootComponents.Add مع Parameters
public CommerceProductDetailsFlow(int medicationId, int companyOrgId)
{
    InitializeComponent();
    BlazorView.RootComponents.Add(new RootComponent
    {
        Selector = "#app",
        ComponentType = typeof(ProductDetailsPage),
        Parameters = new Dictionary<string, object>
        {
            { "MedicationId", medicationId },
            { "CompanyOrgId", companyOrgId },
            { "OnBack", EventCallback.Factory.Create(this, async () => await Navigation.PopModalAsync()) }
        }
    });
}
```

---

#### ❌ النمط 6: Shared.UI تحاول الوصول إلى MAUI (MainThread، Application.Current)

**المشكلة:** `CommerceRootPage.razor` (في Shared.UI) حاولت استدعاء `MainThread.BeginInvokeOnMainThread` و `Application.Current.MainPage.Navigation.PushModalAsync` لفتح صفحة تفاصيل المنتج.

**النتيجة:** `CS0103` (MainThread not found) — لأن `Shared.UI` مشروع مشترك (RCL)، لا يعرف `MAUI`.

**البديل:** استخدم **Bridge Pattern**:
1. واجهة (`ICommerceNavigationBridge`) في `Shared.UI`.
2. تطبيق (`CommerceNavigationBridge`) في `Mobile`.
3. تسجيل في `MauiProgram.cs`.
4. حقن في `CommerceRootPage.razor`.

**مثال من جلسة Commerce:**
```csharp
// 1. Shared.UI/Services/ICommerceNavigationBridge.cs
public interface ICommerceNavigationBridge
{
    Task OpenProductDetailsAsync(int medicationId, int companyOrgId);
}

// 2. Mobile/Services/CommerceNavigationBridge.cs
public class CommerceNavigationBridge : ICommerceNavigationBridge
{
    public async Task OpenProductDetailsAsync(int medicationId, int companyOrgId)
    {
        await MainThread.InvokeOnMainThreadAsync(async () =>
        {
            if (Shell.Current != null)
                await Shell.Current.Navigation.PushAsync(
                    new CommerceProductDetailsFlow(medicationId, companyOrgId));
        });
    }
}

// 3. MauiProgram.cs
builder.Services.AddSingleton<ICommerceNavigationBridge, CommerceNavigationBridge>();

// 4. CommerceRootPage.razor
@inject ICommerceNavigationBridge NavigationBridge

private async Task GoToMedicationDetails((int MedicationId, int CompanyOrgId) payload)
{
    await NavigationBridge.OpenProductDetailsAsync(payload.MedicationId, payload.CompanyOrgId);
}
```

---

#### ❌ النمط 7: PushModalAsync بدون Shell.NavBarIsVisible="True"

**المشكلة:** `CommerceProductDetailsFlow` استخدمت `PushModalAsync` مع `Shell.NavBarIsVisible="False"` (أو بدون `Shell.NavBar`)، فظهرت "صفحة بيضاء" (بدون Header / Breadcrumb / زر رجوع).

**النتيجة:** المستخدم لا يستطيع الرجوع، والصفحة تبدو "مبتورة".

**البديل:**
1. `Shell.NavBarIsVisible="True"` في XAML (أو `PushAsync` بدلاً من `PushModalAsync`).
2. **أو** إضافة `Header` + `Breadcrumb` داخل المكون نفسه (`ProductDetailsPage.razor`)، لتصبح مستقلة.

**مثال من جلسة Commerce:**
```xml
<!-- ✅ صحيح -->
<ContentPage ...
             Shell.NavBarIsVisible="True">
    <BlazorWebView x:Name="BlazorView" HostPage="wwwroot/index.html" />
</ContentPage>

```

---
### 1.5 ملخص الأنماط (Quick Reference)

| # | النمط | `ShellContent` (Root) | `Routing.RegisterRoute` (Detail) |
|---|---|---|---|
| 1 | XAML محلي | ✅ | ✅ |
| 2 | `BlazorWebView` + `<RootComponent>` في XAML (بدون `Parameters`) | ✅ (Singleton) | ✅ |
| 3 | `BlazorWebView` + `RootComponents.Add` في `OnAppearing` | ❌ | ✅ |
| 4 | `BlazorWebView` + `RootComponents.Add` في `Constructor` | ⚠️ (يحتاج `Preferences`) | ✅ |
| 5 | `CurrentUserProfileState` (Singleton) | ✅ (مع النمط 2) | ✅ |

---

## 🚨 الجزء الثاني: مشكلة Dependency Injection (DI)

### 2.1 المشكلة الجذرية (اكتشاف: 13 يونيو 2026)

`BlazorWebView` في MAUI يستخدم **حاوية DI مستقلة** عن حاوية MAUI الرئيسية. هذا يعني أن الخدمات المسجلة في `MauiProgram.cs` ليست متاحة تلقائياً لمكونات Blazor الداخلية.

**⚠️ لكن — في النمط 5 (الناجح)، `CurrentUserProfileState` يعمل — لأنه Singleton في حاوية MAUI، و`BlazorWebView` في `DashboardFlowPage` يحقن منه (وليس من حاوية Blazor).**

### 2.2 ❌ العرض: فشل صامت للمكون

`@inject` لخدمة غير مسجلة في حاوية Blazor الداخلية، يفشل المكون في التحميل تماماً بدون أي خطأ في Debug Output. الصفحة تظهر فارغة أو لا تظهر على الإطلاق.

**⚠️ تحذير:** هذا الفشل صامت تماماً — لا exception، لا خطأ في الـ Console، لا أي مؤشر. `OnInitializedAsync` لا تُستدعى أبداً.

### 2.3 📊 الخدمات التي تعمل وتفشل مع `@inject`

| الخدمة | `@inject` في Blazor | التفسير |
|---|---|---|
| `ISharedTranslationService` | ✅ يعمل | مسجلة في حاوية Blazor |
| `ISharedTranslationState` | ✅ يعمل | مسجلة في حاوية Blazor |
| `NavigationManager` | ✅ يعمل | مسجلة تلقائياً من Blazor runtime |
| `CurrentOrganizationState` | ✅ يعمل | مسجلة في حاوية MAUI (Singleton) — و`BlazorWebView` يحقن منها (في النمط 2). |
| `CurrentUserProfileState` | ✅ يعمل | مسجلة في حاوية MAUI (Singleton) — و`BlazorWebView` يحقن منها (في النمط 4). |
| `HttpClient` | ❌ يفشل بصمت | غير مسجلة في حاوية Blazor |
| `ApiService` | ❌ يفشل بصمت | غير مسجلة في حاوية Blazor |

### 2.4 ✅ الحل: تمرير الخدمات كـ Parameters (أو Singleton)

**في مكون Blazor (.razor):**
```razor
@* ❌ خطأ — سيفشل بصمت في MAUI *@
@inject HttpClient Http
@inject ApiService ApiService

@* ✅ صحيح — استخدم Parameter *@
@code {
    [Parameter] public IApiService? ApiService { get; set; }
}

@* ✅ صحيح — استخدم Singleton (إذا كان مسجلاً في حاوية MAUI) *@
@inject CurrentUserProfileState UserState
```

**في صفحة MAUI المضيفة (.xaml.cs):**
```csharp
private readonly ApiService _apiService;

public MyFlowPage()
{
    InitializeComponent();
    _apiService = IPlatformApplication.Current!.Services
        .GetRequiredService<ApiService>();

    var parameters = new Dictionary<string, object?>
    {
        { "ApiService", (IApiService)_apiService }
    };

    blazorWebView.RootComponents.Add(new RootComponent
    {
        Selector = "#app",
        ComponentType = typeof(MyBlazorPage),
        Parameters = parameters
    });
}
```

### 2.5 🔍 Checklist تشخيص مشكلة DI

عندما يفشل مكون Blazor في التحميل داخل MAUI:
- هل المكون يعمل في Web Blazor لكن ليس في MAUI؟
- هل تستخدم `@inject HttpClient`؟ → احذفه فوراً.
- هل تستخدم `@inject` لأي خدمة غير `ISharedTranslation*` أو `NavigationManager` أو `Current...State` (Singleton)؟
- هل `OnInitializedAsync` لا تُستدعى (لا يظهر Debug Output)؟
- هل الصفحة فارغة تماماً بدون أي خطأ؟

إذا أجبت بنعم على أي سؤال، استبدل `@inject` بـ `[Parameter]` أو `Singleton`.

---

## 🚨 الجزء الثالث: تسجيل صفحات BlazorWebView — قاعدة ذهبية

### 3.1 الخطوة 1: `MauiProgram.cs`

**⚠️ `AddTransient` للصفحات الفرعية (`Routing.RegisterRoute`) — وليس للصفحات الرئيسية (`ShellContent`) — قد يسبب انهيار Android عند Splash.**

**لصفحات `ShellContent`:** يمكن `AddTransient<PageName>()` (آمن).
**لصفحات `Routing.RegisterRoute`:** يمكن `AddTransient<PageName>()` (آمن — لأنها تُنشأ عند الطلب).

### 3.2 الخطوة 2: `AppShell.xaml.cs`

```csharp
Routing.RegisterRoute("PageName", typeof(PageName));
```

### 3.3 نمط XAML الصحيح

**للصفحات الرئيسية (النمط 2):**
```xml
<b:BlazorWebView x:Name="blazorWebView" HostPage="wwwroot/index.html">
    <b:BlazorWebView.RootComponents>
        <b:RootComponent Selector="#app" ComponentType="{x:Type shared:MyPage}" />
    </b:BlazorWebView.RootComponents>
</b:BlazorWebView>
```

**للصفحات الفرعية (النمط 3):**
```xml
<b:BlazorWebView x:Name="blazorWebView" HostPage="wwwroot/index.html" />
```
(بدون `<RootComponents>` — نضيفها في `OnAppearing`).

### 3.4 ⚠️ الخطأ الشائع: `jumpToEnd`

إذا ظهر خطأ `No view found for id 0x7f0800ff (jumpToEnd)`:
- احذف `AddTransient<PageName>()` من `MauiProgram.cs` (إذا كان `ShellContent`).
- قم بـ Clean + حذف `bin/obj` + إعادة بناء.

---

## 🚨 الجزء الرابع: قيود RootComponents في BlazorWebView

### 4.1 ⚠️ السبب الجذري

`BlazorWebView` على Windows يستخدم WebView2 كمحرك. هذا المحرك يربط الـ DOM بمكونات Blazor فقط أثناء دورة حياة الكونستركتور للـ `ContentPage` المضيفة.

`RootComponents.Clear()` ثم `Add` يعمل بنجاح في **بعض** السيناريوهات — خاصة عند استخدام `RootComponents.Add` في `OnAppearing` (للصفحات الفرعية) أو `Constructor` (للصفحات الرئيسية).

**⚠️ لكن** — `RootComponents.Add` في `OnAppearing` **لا يعمل** للصفحات الرئيسية (`ShellContent`) — لأن `BlazorWebView` بدأ بالفعل (المكون لا يُعاد تحميله).

### 4.2 ❌ أنماط فاشلة موثقة

```csharp
// ❌ النمط 1: async في الكونستركتور
public MyFlowPage()
{
    InitializeComponent();
    _ = LoadAndShowAsync(); // Task تنتهي بعد الكونستركتور — لن يعمل
}

// ❌ النمط 2: RootComponents.Add في OnAppearing للصفحات الرئيسية
protected override void OnAppearing()
{
    base.OnAppearing();
    blazorWebView.RootComponents.Add(...); // ← لن يُعاد تحميل المكون
}

// ❌ النمط 3: Clear ثم Add بعد تحميل البيانات (بدون CustomBlazorWebView)
private async Task RefreshWithData()
{
    var data = await apiService.GetDataAsync();
    blazorWebView.RootComponents.Clear();
    blazorWebView.RootComponents.Add(...); // ← لن يُعاد تحميل المكون
}
```

### 4.3 ✅ النمط الصحيح (للصفحات الفرعية)

```csharp
public MyFlowPage()
{
    InitializeComponent();
    _apiService = IPlatformApplication.Current!.Services
        .GetRequiredService<ApiService>();

    LoadBlazorComponent(); // ⭐ في Constructor (للصفحات الفرعية)
}

protected override void OnAppearing()
{
    base.OnAppearing();
    // ⭐ لا نضيف RootComponents هنا (للصفحات الرئيسية)
}
```

**⚠️ للصفحات الرئيسية:** استخدم النمط 4 (`CurrentUserProfileState` + `<RootComponent>` في XAML).

---

## 🚨 الجزء الخامس: فخ `[QueryProperty]` مع Nullable Types

### 5.1 ⚠️ المشكلة (اكتشاف: 8 أكتوبر 2026)

عند استخدام `[QueryProperty]` مع خاصية من نوع `int?` (أو أي `Nullable<T>`)، يفشل MAUI Shell في تحويل القيمة القادمة (String) إلى `Nullable<int>` — ويظهر الخطأ:
```
System.InvalidCastException: Invalid cast from 'System.String' to 
'System.Nullable`1[[System.Int32, System.Private.CoreLib, ...]]'.
```

### 5.2 🎯 السبب الجذري

`[QueryProperty]` يستخدم `Convert.ChangeType` داخلياً، وهذا لا يدعم `Nullable<T>`. عندما يمرر Shell القيمة "1" (String) إلى خاصية `int?`، يفشل التحويل.

### 5.3 ❌ الكود الفاشل
```csharp
[QueryProperty(nameof(ProgramId), "programId")]
public partial class PointsDashboardPage : ContentPage
{
    public int? ProgramId { get; set; }   // ❌ يفشل بـ InvalidCastException
}
```

### 5.4 ✅ الحل: استقبل كـ String ثم حوّل يدوياً
```csharp
[QueryProperty(nameof(ProgramIdRaw), "programId")]
public partial class PointsDashboardPage : ContentPage
{
    public string? ProgramIdRaw { get; set; }
    private int? ProgramId => int.TryParse(ProgramIdRaw, out var id) ? id : null;
}
```

### 5.5 📌 القاعدة الذهبية

> `[QueryProperty]` يستخدم `Convert.ChangeType` داخلياً، وهذا لا يدعم `Nullable<T>`. **دائماً استقبل QueryProperty كـ `string`، ثم حوّل يدوياً.**

### 5.6 📊 جدول: أنواع Property ونتائجها

| النوع | يعمل؟ | ملاحظات |
|---|---|---|
| `string` | ✅ | الأكثر أماناً — استخدمه دائماً |
| `string?` | ✅ | يعمل (مع nullable reference types) |
| `int` (non-nullable) | ✅ | يعمل، لكن يفشل إذا كانت القيمة فارغة |
| `int?` (nullable) | ❌ | يفشل بـ InvalidCastException |
| `DateTime?` | ❌ | نفس المشكلة |
| `bool` (non-nullable) | ✅ | يعمل |
| `bool?` (nullable) | ❌ | نفس المشكلة |

### 5.7 ⚠️ فخ إضافي: `[QueryProperty]` يُعيَّن بعد Constructor

**المشكلة:**
```
Constructor (LoadBlazorComponent) → QueryProperty setter → OnAppearing
                                       ↑
                          ⚠️ يُعيَّن هنا، بعد أن يكون المكون قد بُني
```

**الحل:** انقل `LoadBlazorComponent()` إلى `OnAppearing` (مع flag `_componentLoaded`):
```csharp
private bool _componentLoaded = false;

protected override void OnAppearing()
{
    base.OnAppearing();
    if (!_componentLoaded)
    {
        LoadBlazorComponent();
        _componentLoaded = true;
    }
}
```

### 5.8 📊 متى تستخدم `IQueryAttributable` بدلاً من `[QueryProperty]`؟

| الحالة | `[QueryProperty]` | `IQueryAttributable` |
|---|---|---|
| 1-2 معاملات | ✅ مُوصى | يعمل |
| 3+ معاملات | ❌ وميض | ✅ مُوصى |
| `Nullable<T>` | ❌ يحتاج حيلة (string) | ✅ يدعم أصلاً |
| إعادة إنشاء المكون | لا | لا |
| LoadBlazorComponent في Constructor | ✅ | يحتاج `_queryReady` flag |
| ترتيب الاستدعاء | Setter في Constructor | ApplyQueryAttributes (بعد Constructor، قبل OnAppearing) |

**القاعدة:**
- `[QueryProperty]` للبساطة (1-2 معاملات، string).
- `IQueryAttributable` للمرونة (3+ معاملات، Nullable<T>، تحكم كامل).

---

## 🎯 الجزء السادس: الأنماط الخمسة الفعلية في المشروع (Proven)

### 6.1 🌟 النمط 1: Constructor + LoadComponent (الأساسي — مستقر)

**متى يُستخدم:**
- عند عدم وجود معاملات أو وجود معاملات بسيطة جداً.
- عند الحاجة لتحميل المكون مرة واحدة عند فتح الصفحة.

**المثال:** `PharmacySearchFlow.xaml.cs`

```csharp
public partial class MyFlowPage : ContentPage
{
    public MyFlowPage()
    {
        InitializeComponent();
        _apiService = IPlatformApplication.Current!.Services
            .GetRequiredService<ApiService>();

        LoadBlazorComponent();  // ⭐ في الـ Constructor
    }

    private void LoadBlazorComponent()
    {
        blazorWebView.RootComponents.Clear();
        blazorWebView.RootComponents.Add(new RootComponent
        {
            Selector = "#app",
            ComponentType = typeof(MyBlazorComponent),
            Parameters = new Dictionary<string, object?>
            {
                { "ApiService", (IApiService)_apiService }
            }
        });
    }

    protected override void OnDisappearing()
    {
        base.OnDisappearing();
        try { blazorWebView.RootComponents.Clear(); } catch { }
    }
}
```

**المزايا:** ✅ الأكثر استقراراً وبساطة. ✅ لا وميض. ✅ الأداء الأمثل.
**العيوب:** ⚠️ لا يستقبل `QueryProperty` تلقائياً. ⚠️ لا يُحدَّث عند تغيير الصفحة (يحتاج `OnAppearing` مع `_isLoaded` flag).

### 6.2 🌟 النمط 2: QueryProperty + Setter → LoadComponent (للمعاملات القليلة)

**متى يُستخدم:**
- عند وجود 1-2 معاملات فقط (QueryProperty).
- عندما لا تكون البيانات ثقيلة جداً.
- ⚠️ لا يُنصح به لأكثر من 2-3 معاملات (يسبب وميض).
- ⚠️ لا تستخدمه مع `Nullable<T>` — استخدم `string` + تحويل يدوي.

**المثال:** `PharmacyDetailFlow.xaml.cs`

```csharp
[QueryProperty(nameof(PharmacyId), "pharmacyId")]
[QueryProperty(nameof(TokenCode), "token")]
public partial class PharmacyDetailFlow : ContentPage, IBackButtonHandler
{
    private int _pharmacyId;
    public int PharmacyId
    {
        get => _pharmacyId;
        set
        {
            _pharmacyId = value;
            LoadComponent();   // ⭐ إعادة تحميل المكون في الـ Setter
        }
    }

    private string? _tokenCode;
    public string? TokenCode
    {
        get => _tokenCode;
        set
        {
            _tokenCode = value;
            LoadComponent();   // ⭐ إعادة تحميل المكون في الـ Setter
        }
    }

    public PharmacyDetailFlow()
    {
        InitializeComponent();  // لا شيء آخر هنا
    }

    private void LoadComponent()
    {
        blazorWebView.RootComponents.Clear();
        blazorWebView.RootComponents.Add(new RootComponent
        {
            Selector = "#app",
            ComponentType = typeof(PharmacyDetailPage),
            Parameters = new Dictionary<string, object?>
            {
                { "PharmacyId", _pharmacyId },
                { "TokenCode", _tokenCode ?? "" },
                { "OnGoBackAction", new Action(() => GoBack()) }
            }
        });
    }
}
```

**المزايا:** ✅ يتعامل مع `QueryProperty` تلقائياً. ✅ يعمل بثبات في المشروع — خاصة مع `CustomBlazorWebView`.
**العيوب:** ⚠️ يسبب وميضاً عند إعادة التحميل. ⚠️ مع 3+ معاملات: `LoadComponent()` تُستدعى 3+ مرات → وميض شديد + طلبات API متكررة.

### 6.3 🌟 النمط 3: Constructor + Refresh in Background (مع Cache)

**متى يُستخدم:**
- عند وجود بيانات محلية (Preferences/SecureStorage) يمكن عرضها فوراً.
- عند الحاجة لتحديث شفاف في الخلفية.

**المثال:** `ProfessionalStatusFlow.xaml.cs`

```csharp
public partial class ProfessionalStatusFlow : ContentPage
{
    private static string _statusCode = "";
    private static string _professionalRole = "";

    public ProfessionalStatusFlow()
    {
        InitializeComponent();
        _apiService = IPlatformApplication.Current!.Services
            .GetRequiredService<ApiService>();

        // 1. تحميل البيانات من الذاكرة المحلية
        LoadCachedData();

        // 2. تحميل UI فوراً
        LoadProfessionalStatusPage();

        // 3. تحديث من API في الخلفية
        _ = RefreshFromApiAsync();
    }

    private void LoadCachedData()
    {
        _statusCode = Preferences.Get("license_status_code", "");
        _professionalRole = Preferences.Get("license_role_type", "");
    }

    private async Task RefreshFromApiAsync()
    {
        var session = await _cachedSessionService.GetSessionAsync();
        Preferences.Set("license_status_code", newStatusCode);

        if (changed)
        {
            await MainThread.InvokeOnMainThreadAsync(() =>
            {
                LoadProfessionalStatusPage();  // ← Clear + Add
            });
        }
    }
}
```

**المزايا:** ✅ لا وميض — UI يظهر فوراً من الـ Cache. ✅ تحديث شفاف. ✅ تجربة مستخدم ممتازة.
**العيوب:** ⚠️ يستخدم static variables (ليس نظيفاً معمارياً). ⚠️ يعيد إنشاء `RootComponent` عند التحديث (قد يسبب وميضاً بسيطاً). ⚠️ يحتاج `OnDisappearing` للتنظيف.

### 6.4 🌟 النمط 4: Constructor + OnComponentReady (للتفاعل MAUI ↔ Component)

**متى يُستخدم:**
- عندما يحتاج MAUI لاستدعاء methods على المكون (مثل FilePicker).
- عند وجود تفاعل مستمر بين MAUI والمكوّن.

**المثال:** `ProfessionalLicenseFlow.xaml.cs`

```csharp
public partial class ProfessionalLicenseFlow : ContentPage
{
    private ProfessionalLicensePage? _licensePageRef;

    public ProfessionalLicenseFlow()
    {
        InitializeComponent();
        _apiService = IPlatformApplication.Current!.Services
            .GetRequiredService<ApiService>();

        LoadLicensePage();  // في Constructor
    }

    private void LoadLicensePage()
    {
        blazorWebView.RootComponents.Clear();
        blazorWebView.RootComponents.Add(new RootComponent
        {
            Selector = "#app",
            ComponentType = typeof(ProfessionalLicensePage),
            Parameters = new Dictionary<string, object?>
            {
                { "ApiService", (IApiService)_apiService },
                { "OnPickFileRequested", new Action(async () => await PickFileAndUpdateAsync()) },
                { "OnComponentReady", new Action<ProfessionalLicensePage>(page =>
                    {
                        _licensePageRef = page;
                        Debug.WriteLine("✅ LicensePage ref captured");
                    })
                }
            }
        });
    }

    private async Task PickFileAndUpdateAsync()
    {
        var result = await FilePicker.PickAsync(...);
        if (result == null) return;

        using var stream = await result.OpenReadAsync();
        using var ms = new MemoryStream();
        await stream.CopyToAsync(ms);
        var bytes = ms.ToArray();
        var fileName = result.FileName;

        await MainThread.InvokeOnMainThreadAsync(() =>
        {
            _licensePageRef?.SetFileData(fileName, bytes);
        });
    }

    protected override void OnDisappearing()
    {
        base.OnDisappearing();
        _licensePageRef = null;
        try { blazorWebView.RootComponents.Clear(); } catch { }
    }
}
```

**في المكون (.razor):**
```csharp
@code {
    [Parameter] public Action<MyComponent>? OnComponentReady { get; set; }

    protected override void OnAfterRender(bool firstRender)
    {
        if (firstRender)
        {
            OnComponentReady?.Invoke(this);
        }
    }

    public void SetFileData(string fileName, byte[] fileBytes)
    {
        _fileName = fileName;
        _selectedFileBytes = fileBytes;
        InvokeAsync(StateHasChanged);
    }
}
```

**المزايا:** ✅ يسمح بـ التفاعل MAUI ↔ Component بدون إعادة تحميل. ✅ لا وميض. ✅ يحافظ على State المكون.
**العيوب:** ⚠️ يتطلب public methods على المكون. ⚠️ يحتاج Dispose لتنظيف المرجع. ⚠️ `OnComponentReady` يُستدعى مرة واحدة فقط عند `firstRender`.

### 6.5 🌟 النمط 5: هجين (IQueryAttributable + OnComponentReady + OnAppearing)

**متى يُستخدم:**
- عند وجود 3+ معاملات (تجنب النمط 2).
- عند الحاجة لتحديث المكون بعد وصول المعاملات.
- عند الرغبة في تجنب الوميض.

**المثال:** `ProgramDetailsPage.xaml.cs`

```csharp
public partial class ProgramDetailsPage : ContentPage, IQueryAttributable
{
    private ProgramDetailsWrapper? _wrapperRef;
    private readonly IApiService _apiService;
    private bool _queryReady = false;

    public ProgramDetailsPage()
    {
        InitializeComponent();
        _apiService = IPlatformApplication.Current!.Services
            .GetRequiredService<IApiService>();

        LoadBlazorComponent();  // ⭐ بدون QueryProperties (بقيم افتراضية)
    }

    private void LoadBlazorComponent()
    {
        blazorWebView.RootComponents.Clear();
        blazorWebView.RootComponents.Add(new RootComponent
        {
            Selector = "#app",
            ComponentType = typeof(ProgramDetailsWrapper),
            Parameters = new Dictionary<string, object?>
            {
                { "ApiService", (IApiService)_apiService },
                { "OnComponentReady", new Action<ProgramDetailsWrapper>(w =>
                    {
                        _wrapperRef = w;
                        Debug.WriteLine("✅ ProgramDetailsWrapper ref captured");
                        TryLoadProgram();   // ⭐ محاولة فورية
                    })
                },
                { "OnGoBack", SafeAsyncAction(GoBackAsync) },
                { "OnGoToPointsDashboard", SafeAsyncAction<int>(GoToPointsDashboardAsync) },
            }
        });
    }

    public void ApplyQueryAttributes(IDictionary<string, object> query)
    {
        if (query.TryGetValue("ProgramId", out var pid))
            _programId = Convert.ToInt32(pid);
        if (query.TryGetValue("Source", out var src))
            _source = src?.ToString() ?? "Search";

        _queryReady = true;
        TryLoadProgram();   // ⭐ محاولة فورية
    }

    private void TryLoadProgram()
    {
        if (!_queryReady || _wrapperRef == null)
        {
            Debug.WriteLine($"⏸️ [TryLoadProgram] Waiting: QueryReady={_queryReady}, WrapperRef={_wrapperRef != null}");
            return;
        }

        Debug.WriteLine("🚀 [TryLoadProgram] Both ready — invoking LoadProgram");
        MainThread.BeginInvokeOnMainThread(async () =>
        {
            try
            {
                await _wrapperRef.LoadProgram(_programId, _source, _isSubscribed,
                                              _organizationId, _invitationToken);
            }
            catch (Exception ex)
            {
                Debug.WriteLine($"❌ [TryLoadProgram] LoadProgram error: {ex}");
            }
        });
    }

    private Action<T> SafeAsyncAction<T>(Func<T, Task> asyncAction) =>
        (arg) => MainThread.BeginInvokeOnMainThread(async () =>
        {
            try { await asyncAction(arg); }
            catch (Exception ex) { Debug.WriteLine($"❌ [SafeAsyncAction] {ex}"); }
        });

    private Action SafeAsyncAction(Func<Task> asyncAction) =>
        () => MainThread.BeginInvokeOnMainThread(async () =>
        {
            try { await asyncAction(); }
            catch (Exception ex) { Debug.WriteLine($"❌ [SafeAsyncAction] {ex}"); }
        });

    protected override void OnDisappearing()
    {
        base.OnDisappearing();
        _wrapperRef = null;
        _queryReady = false;
        try { blazorWebView.RootComponents.Clear(); } catch { }
    }
}
```

**المزايا:** ✅ يستقبل كل المعاملات مرة واحدة (لا وميض). ✅ لا يعيد تحميل المكون (لا طلبات API متكررة). ✅ يحافظ على State المكون. ✅ تم التحقق منه في المشروع (8 أكتوبر 2026 — `ProgramDetailsPage`).
**العيوب:** ⚠️ يحتاج public method على المكون (`LoadProgram`). ⚠️ يحتاج flag `_queryReady` لمنع الاستدعاء قبل وصول الـ Query.

---

## 🌟 الجزء السادس-ب: النمط الناجح (6.6 — بعد نجاح 4-5 نوفمبر)

### 6.6.1 🌟 النمط 6: `CurrentUserProfileState` (Singleton) + `<RootComponent>` في XAML (للصفحات الرئيسية)

**متى يُستخدم:**
- الصفحة **رئيسية** (`ShellContent`) وتحتاج `BlazorWebView`.
- تحتاج بيانات مستخدم (`UserProfileId`) — بدون `Parameters` ديناميكية.

**المثال الناجح:** `DashboardFlowPage.xaml` (4-5 نوفمبر 2026).

**البنية (`DashboardFlowPage.xaml`):**
```xml
<ContentPage x:Class="RubikCare.Mobile.Features.PublicUser.Views.DashboardFlowPage">
    <b:BlazorWebView x:Name="blazorWebView" HostPage="wwwroot/index.html">
        <b:BlazorWebView.RootComponents>
            <b:RootComponent Selector="#app" ComponentType="{x:Type sharedPages:DashboardPage}" />
        </b:BlazorWebView.RootComponents>
    </b:BlazorWebView>
</ContentPage>
```

**البنية (`DashboardFlowPage.xaml.cs`):**
```csharp
public partial class DashboardFlowPage : ContentPage
{
    private readonly CurrentUserProfileState _userState;

    public DashboardFlowPage()
    {
        InitializeComponent();
        _userState = IPlatformApplication.Current!.Services.GetRequiredService<CurrentUserProfileState>();

        // ⭐ قراءة UserProfileId من Preferences (متزامن — قبل OnInitializedAsync)
        var userId = Preferences.Get("user_profile_id", 0);
        var userName = Preferences.Get("user_full_name", "مستخدم");

        if (userId > 0)
        {
            _userState.SetUserProfile(userId, userName);  // ⭐ قبل OnInitializedAsync
        }
    }

    protected override async void OnAppearing()
    {
        base.OnAppearing();

        if (_userState.UserProfileId <= 0)
        {
            var session = await _sessionService.GetSessionAsync();
            if (session != null)
            {
                _userState.SetUserProfile(session.UserProfileId, session.FullNameAr);
                Preferences.Set("user_profile_id", session.UserProfileId);
                Preferences.Set("user_full_name", session.FullNameAr);
            }
        }
    }
}
```

**المكون المشترك (`DashboardPage.razor`):**
```csharp
@inject CurrentUserProfileState UserState

protected override void OnInitialized()
{
    UserState.OnChange += OnUserStateChanged;
}

protected override async Task OnInitializedAsync()
{
    await LoadTranslationsAsync();
    _translationsLoaded = true;

    if (UserState.UserProfileId > 0)
    {
        _dataLoaded = true;
        await LoadAllDataAsync();
    }
    else
    {
        _isLoading = false;
    }
}

private async void OnUserStateChanged()
{
    if (UserState.UserProfileId > 0 && !_dataLoaded)
    {
        _dataLoaded = true;
        await InvokeAsync(async () => { await LoadAllDataAsync(); StateHasChanged(); });
    }
}
```

**المزايا:**
- ✅ يعمل كصفحة رئيسية (`ShellContent`).
- ✅ لا `Parameters` ديناميكية (فقط `Singleton`).
- ✅ `Preferences` (متزامن — سريع).

**العيوب:**
- ❌ يحتاج `Preferences.Set` (في `AppShell.InitializeAsync`).
- ❌ يحتاج `SetUserProfile` في `Constructor` (قبل `OnInitializedAsync`).

### 6.6.2 🌟 النمط 7: `RootComponents.Add` في `Constructor` (للصفحات الرئيسية التي تحتاج `Parameters`)

**متى يُستخدم:**
- الصفحة **رئيسية** (`ShellContent`) وتحتاج `Parameters` ديناميكية (لكن `Parameters` متزامنة من `Preferences`).

**مثال:** `MedicineGuideFlow.xaml.cs` (بعد التعديل — 5 نوفمبر).

**البنية (`MedicineGuideFlow.xaml`):**
```xml
<ContentPage x:Class="RubikCare.Mobile.Features.PSP.Patient.Views.MedicineGuideFlow">
    <BlazorWebView x:Name="blazorWebView" HostPage="wwwroot/index.html" />
</ContentPage>
```

**البنية (`MedicineGuideFlow.xaml.cs`):**
```csharp
public partial class MedicineGuideFlow : ContentPage, IQueryAttributable
{
    private int _caseId;
    private int _programId;

    public MedicineGuideFlow()
    {
        InitializeComponent();
    }

    public void ApplyQueryAttributes(IDictionary<string, object> query)
    {
        if (query.TryGetValue("caseId", out var cid))
            int.TryParse(cid?.ToString(), out _caseId);
        if (query.TryGetValue("programId", out var pid))
            int.TryParse(pid?.ToString(), out _programId);

        LoadBlazorComponent();
    }

    private void LoadBlazorComponent()
    {
        blazorWebView.RootComponents.Clear();
        blazorWebView.RootComponents.Add(new RootComponent
        {
            Selector = "#app",
            ComponentType = typeof(MedicineGuidePage),
            Parameters = new Dictionary<string, object?>
            {
                { "CaseIdOverride", _caseId > 0 ? _caseId : (int?)null },
                { "ProgramIdOverride", _programId > 0 ? _programId : (int?)null }
            }
        });
    }
}
```

**⚠️ لاحظ:** `MedicineGuideFlow` **صفحة فرعية** (`Routing.RegisterRoute`)، وليس `ShellContent`. **لذا `RootComponents.Add` في `ApplyQueryAttributes` يعمل.**

---

## 🌟 الجزء السادس-ج: ملخص الأنماط الناجحة (Quick Reference)

| # | النمط | `ShellContent` (Root) | `Routing.RegisterRoute` (Detail) | يعمل؟ |
|---|---|---|---|---|
| 1 | XAML محلي | ✅ | ✅ | ✅ |
| 2 | `BlazorWebView` + `<RootComponent>` في XAML (بدون `Parameters`) | ✅ (Singleton) | ✅ | ✅ |
| 3 | `RootComponents.Add` في `OnAppearing` | ❌ | ✅ | ✅ (Detail فقط) |
| 4 | `RootComponents.Add` في `Constructor` | ⚠️ (يحتاج `Preferences`) | ✅ | ✅ (Detail) |
| 5 | `OnComponentReady` | ❌ | ✅ | ✅ (Detail فقط) |
| 6 | `IQueryAttributable` + `OnComponentReady` | ❌ | ✅ | ✅ (Detail فقط) |
| **7** | **`CurrentUserProfileState` (Singleton) + `<RootComponent>` في XAML** | **✅** | ✅ | **✅ (Root + Detail)** |
| **8** | **`RootComponents.Add` في `ApplyQueryAttributes`** (مع `Parameters`) | ❌ | ✅ | **✅ (Detail فقط)** |

**⚠️ القاعدة:** `ShellContent` (Root) → النمط 1 (XAML محلي) أو النمط 7 (`CurrentUserProfileState` + `<RootComponent>` في XAML). **لا `RootComponents.Add` في `OnAppearing` لـ `ShellContent`.**

---

## 🎯 الجزء السابع: شجرة القرار — أي نمط أستخدم؟

### 7.0 شجرة القرار المحدثة (Proven — 5 نوفمبر 2026)

```
هل الصفحة رئيسية (ShellContent)؟
├── نعم
│   ├── هل تحتاج Parameters ديناميكية (UserProfileId, etc.)؟
│   │   ├── نعم → ❌ لا تستخدم BlazorWebView.
│   │   │         → ✅ استخدم XAML محلي (النمط 1).
│   │   │         → أو ✅ استخدم CurrentUserProfileState (Singleton) + <RootComponent> في XAML (النمط 7).
│   │   └── لا → ✅ يمكن استخدام BlazorWebView + <RootComponent> في XAML (النمط 2).
│   └── (في كل الحالات) → الأفضل: XAML محلي أو النمط 7.
│
└── لا (Routing.RegisterRoute)
    ├── هل تحتاج Parameters ديناميكية؟
    │   ├── نعم
    │   │   ├── هل Parameter واحد أو اثنان؟ (string) → النمط 2 (QueryProperty + Setter).
    │   │   ├── هل 3+ Parameters؟ → النمط 5 (IQueryAttributable + OnComponentReady).
    │   │   └── (أو) → النمط 8 (RootComponents.Add في ApplyQueryAttributes).
    │   └── لا
    │       ├── هل تحتاج Cache؟ → النمط 3 (Constructor + Refresh).
    │       ├── هل تحتاج تفاعل MAUI ↔ Component؟ → النمط 4 (Constructor + OnComponentReady).
    │       └── لا → النمط 1 (Constructor + LoadComponent).
    └── (في كل الحالات) → لا تستخدم <RootComponent> في XAML (بدون Parameters) للصفحات الفرعية.
```

### 7.1 جدول ملخص لاختيار النمط

| السيناريو | النمط المُوصى به | ملاحظات |
|---|---|---|
| صفحة رئيسية (`ShellContent`) بدون Parameters | النمط 2 (`BlazorWebView` + `<RootComponent>` في XAML) | يعتمد على Singleton. |
| صفحة رئيسية (`ShellContent`) مع Singleton (UserProfileId) | النمط 7 (`CurrentUserProfileState` + `<RootComponent>` في XAML) | **النمط الناجح** (4-5 نوفمبر). |
| صفحة رئيسية (`ShellContent`) بدون BlazorWebView | النمط 1 (XAML محلي) | الأبسط. |
| صفحة فرعية بمعامل واحد أو اثنين (string) | النمط 2 (QueryProperty + Setter) | يعمل بثبات. |
| صفحة فرعية ببيانات Cache محلية | النمط 3 (Constructor + Refresh) | تجربة مستخدم ممتازة. |
| صفحة فرعية بتفاعل MAUI → Component | النمط 4 (Constructor + OnComponentReady) | يحتاج public methods. |
| صفحة فرعية بـ 3+ Parameters | النمط 5 (IQueryAttributable + OnComponentReady) | انظر § 8. |
| صفحة فرعية بـ Nullable QueryProperty | النمط 2 مع string | استقبل كـ string، حوّل يدوياً. |
| صفحة فرعية تحتاج `Parameters` ديناميكية | النمط 8 (`RootComponents.Add` في `ApplyQueryAttributes`) | **النمط الناجح** (MedicineGuideFlow). |

---

## 🔀 الجزء الثامن: النمط 5 (هجين) — `IQueryAttributable` + `OnComponentReady` + `OnAppearing`

*(هذا الجزء يحتوي على تفاصيل النمط 5 — كما في الإصدار السابق. لم يتغير.)*

### 8.1 متى يُستخدم
- عند وجود 3+ معاملات (تجنب النمط 2).
- عند الحاجة لتحديث المكون بعد وصول المعاملات.
- عند الرغبة في تجنب الوميض.

### 8.2 البنية الكاملة
*(كما في النمط 5 — § 6.5)*

### 8.3 المزايا
- ✅ يستقبل كل المعاملات مرة واحدة (لا وميض).
- ✅ لا يعيد تحميل المكون (لا طلبات API متكررة).
- ✅ يحافظ على State المكون.
- ✅ تم التحقق منه في المشروع (8 أكتوبر 2026 — `ProgramDetailsPage`).

### 8.4 العيوب
- ⚠️ يحتاج public method على المكون (`LoadProgram`).
- ⚠️ يحتاج flag `_queryReady` لمنع الاستدعاء قبل وصول الـ Query.

---

## 🚨 الجزء التاسع: تجنّب `Action<T>` مع async

*(هذا الجزء يحتوي على تفاصيل `Action<T>` مع async — كما في الإصدار السابق. لم يتغير.)*

### 9.1 ⚠️ المشكلة (اكتشاف: 8 أكتوبر 2026)

تمرير `async lambda` كـ `Action<int>` يُخفي الـ Exception الحقيقي ويحوّله إلى `JavaProxyThrowable` المبهم على Android — مما يجعل التشخيص شبه مستحيل.

### 9.2 ❌ النمط الخاطئ
```csharp
{ "OnGoToPointsDashboard", new Action<int>(async (programId) =>
    await GoToPointsDashboardAsync(programId)) },
```

**النتيجة:** عند حدوث Exception في `GoToPointsDashboardAsync`، يظهر:
```
Android.Runtime.JavaProxyThrowable
  HResult=0x80131500
  Source=<Cannot evaluate the exception source>
  StackTrace:
<Cannot evaluate the exception stack trace>
```
لا يخبرك بشيء.

### 9.3 ✅ النمط الصحيح

#### الحل 1: لفّ مباشر
```csharp
{ "OnGoToPointsDashboard", new Action<int>((programId) =>
{
    MainThread.BeginInvokeOnMainThread(async () =>
    {
        try
        {
            await GoToPointsDashboardAsync(programId);
        }
        catch (Exception ex)
        {
            Debug.WriteLine($"❌ [OnGoToPointsDashboard] {ex}");
        }
    });
}) },
```

#### الحل 2: Helper Method (مُوصى به)
```csharp
private Action<T> SafeAsyncAction<T>(Func<T, Task> asyncAction) =>
    (arg) => MainThread.BeginInvokeOnMainThread(async () =>
    {
        try { await asyncAction(arg); }
        catch (Exception ex) { Debug.WriteLine($"❌ [SafeAsyncAction] {ex}"); }
    });

private Action SafeAsyncAction(Func<Task> asyncAction) =>
    () => MainThread.BeginInvokeOnMainThread(async () =>
    {
        try { await asyncAction(); }
        catch (Exception ex) { Debug.WriteLine($"❌ [SafeAsyncAction] {ex}"); }
    });

// الاستخدام:
{ "OnGoToPointsDashboard", SafeAsyncAction<int>(GoToPointsDashboardAsync) },
{ "OnGoBack", SafeAsyncAction(GoBackAsync) },
```

### 9.4 📌 القاعدة الذهبية

> كل `Action<T>` يحتوي على `async` يجب أن:
> 1. يُلفّ في `MainThread.BeginInvokeOnMainThread`.
> 2. يحتوي على `try/catch` صريح.
> 3. يطبع الـ Exception الكامل (`ex.ToString()`).

هذا يحوّل `JavaProxyThrowable` المبهم إلى Exception واضح.

### 9.5 📊 مقارنة: قبل/بعد

| قبل | بعد |
|---|---|
| `JavaProxyThrowable` فقط | Exception حقيقي مع Stack Trace كامل |
| لا نعرف السبب | نعرف السطر والسبب |
| ساعات من التشخيص | دقائق من التشخيص |

---

## 🚨 الجزء العاشر: `CustomBlazorWebView` (بعد التعديل)

### 10.1 📌 ما هو؟

`CustomBlazorWebView` هو Wrapper مخصص حول `BlazorWebView` مُسجَّل في `RubikCare.Mobile.Controls`.

### 10.2 📌 لماذا؟

- يحل مشكلة `jumpToEnd` على Android.
- يوفر `OnPageAppearing()` method.
- يعالج `IBackButtonHandler` تلقائياً.
- **⚠️ لكنه يعيد تعيين `RootComponents` (`ForceRender`) — مما يسبب مشاكل للصفحات الرئيسية.**

### 10.3 📌 الاستخدام القياسي (بعد التعديل)

**⚠️ `CustomBlazorWebView` — استخدمه فقط للصفحات الفرعية (التي تعمل مع `RootComponents.Add` في `OnAppearing`).**

```xml
<ContentPage xmlns="http://schemas.microsoft.com/dotnet/2021/maui"
             xmlns:x="http://schemas.microsoft.com/winfx/2009/xaml"
             xmlns:controls="clr-namespace:RubikCare.Mobile.Controls"
             x:Class="RubikCare.Mobile.Features.MyFlow">
    <controls:CustomBlazorWebView x:Name="blazorWebView" HostPage="wwwroot/index.html" />
</ContentPage>
```

### 10.4 ⚠️ القاعدة المحدثة (بعد 4-5 نوفمبر 2026)

- **للصفحات الرئيسية (`ShellContent`):** استخدم `BlazorWebView` العادي (وليس `CustomBlazorWebView`) + `<RootComponent>` في XAML. **أو** استخدم `CustomBlazorWebView` مع `CurrentUserProfileState` (النمط 7) — **لكن `CustomBlazorWebView` قد يسبب مشاكل مع `RootComponent` في XAML.**
- **للصفحات الفرعية (`Routing.RegisterRoute`):** استخدم `CustomBlazorWebView` (كما كان) — يعمل.
- **⚠️ `CustomBlazorWebView` + `<RootComponent>` في XAML = خطر** (ForceRender قد يمسح المكون).

**توصيتي:** **للصفحات الرئيسية:** `BlazorWebView` العادي (وليس `Custom`). **للصفحات الفرعية:** `CustomBlazorWebView` (كما كان).

---

## 🔴 الجزء الحادي عشر: تشخيص الصفحة الفارغة (بعد التعديل)

### 11.0 مقدمة: الفشل الصامت في `ShellContent`

**بعد جلسة 4-5 نوفمبر 2026، أصبح التشخيص أكثر دقة.** إذا كانت الصفحة الرئيسية (`ShellContent`) بيضاء (بدون أي خطأ)، فالمشكلة على الأرجح في **أحد هذه الأسباب**:

| # | السبب | الدليل |
|---|---|---|
| 1 | `RootComponents.Add` في `OnAppearing` (لصفحة `ShellContent`) — لا يعيد تحميل المكون. | `Console.WriteLine` في `OnInitializedAsync` لا يظهر. |
| 2 | `CurrentUserProfileState` غير مملوء (نسختان أو `SetUserProfile` لم يُستدعى). | `Console.WriteLine` يظهر `UserProfileId=0`. |
| 3 | `@inject` لخدمة غير مسجلة → فشل صامت. | `OnInitializedAsync` لا تُستدعى. |
| 4 | `CustomBlazorWebView` (ForceRender) يمسح `RootComponents`. | المكون يظهر لثم يختفي. |

### 11.1 الخطوة 1: تأكد أن MAUI layer يعمل

```csharp
public MyFlowPage()
{
    InitializeComponent();
    Debug.WriteLine("🔵 Constructor START");
    Debug.WriteLine($"🟢 RootComponents: {blazorWebView.RootComponents.Count}");
}
```

| النتيجة | المشكلة |
|---|---|
| لا يظهر "Constructor START" | مشكلة في تسجيل الصفحة أو الـ Routing. |
| يظهر لكن `Count = 0` | مشكلة في `RootComponents.Add`. |
| `Count = 1` | انتقل للخطوة 2. |

### 11.2 الخطوة 2: تأكد أن Blazor runtime يعمل

**استبدل مكونك مؤقتاً بـ `TestPage` بسيطة بدون أي `@inject`:**
```razor
@namespace RubikCare.Shared.UI.Components

<h1 style="color:red;font-size:48px;">✅ BLAZOR WORKS</h1>

@code {
    protected override void OnInitialized()
    {
        Debug.WriteLine("🔴 TestPage: OnInitialized CALLED");
    }
}
```

| النتيجة | المشكلة |
|---|---|
| لم تظهر TestPage | مشكلة في WebView2 أو `index.html` أو تسجيل الخدمات. |
| ظهرت TestPage | انتقل للخطوة 3. |

### 11.3 الخطوة 3: عزّل مشكلة `@inject`

**أضف `@inject` الخاصة بمكونك واحداً واحداً لـ `TestPage` حتى تجد المشكلة.**

**توقفت عند خدمة معينة** → هذه الخدمة غير مسجلة في حاوية Blazor → حوّلها إلى `[Parameter]` أو `Singleton`.

### 11.4 الخطوة 4: تأكد من `OnInitializedAsync`

```csharp
protected override async Task OnInitializedAsync()
{
    Debug.WriteLine("🔵 OnInitializedAsync START");
    Debug.WriteLine("🟢 OnInitializedAsync END");
}
```

| النتيجة | المشكلة |
|---|---|
| لا يظهر START | مشكلة DI (الخطوة 3). |
| يظهر START لكن لا يظهر END | Exception في المنتصف. |

### 11.5 الخطوة 5: إذا ظهر `JavaProxyThrowable`

**السبب المحتمل:** `Action<T>` مع `async` — راجع § 9.

**الحل:** لفّ الـ `Action` في `MainThread.BeginInvokeOnMainThread` + `try/catch` (أو `SafeAsyncAction`).

### 11.6 الخطوة 6 (جديد): فحص `CurrentUserProfileState`

**إذا كانت الصفحة الرئيسية تستخدم `CurrentUserProfileState`:**

```csharp
protected override void OnInitialized()
{
    System.Diagnostics.Debug.WriteLine($"🟢 [Razor] OnInitialized: UserProfileId={UserState.UserProfileId}");
    UserState.OnChange += OnUserStateChanged;
}
```

| النتيجة | المشكلة |
|---|---|
| `UserProfileId=0` | `SetUserProfile` لم يُستدعى (في `Constructor` أو `OnAppearing`). |
| `UserProfileId=15` | ✅ (المشكلة في مكان آخر — افحص `LoadAllDataAsync`). |

### 11.7 الخطوة 7 (جديد): فحص `Preferences`

**إذا كانت الصفحة الرئيسية تستخدم `Preferences` (في `Constructor`):**

```csharp
var userId = Preferences.Get("user_profile_id", 0);
Console.WriteLine($"🟢 [Constructor] userId={userId}");
```

| النتيجة | المشكلة |
|---|---|
| `userId=0` | `Preferences.Set("user_profile_id", ...)` لم يُستدعى (في `AppShell.InitializeAsync`). |
| `userId=15` | ✅ (المشكلة في مكان آخر). |

---

## 🔗 الجزء الثاني عشر: نظام الملاحة (Navigation) في MAUI Shell

### 12.1 📌 القواعد الذهبية لتسجيل الصفحات

#### 1. الصفحات الرئيسية (Root Pages) — 6 صفحات فقط

| الصفحة | Route | الملف |
|---|---|---|
| تسجيل الدخول | LoginPage | AppShell.xaml |
| إنشاء حساب | RegisterPage | AppShell.xaml |
| الرئيسية (Personal) | MainDashboard | AppShell.xaml |
| لوحة العيادة | ClinicDashboardPage | AppShell.xaml |
| لوحة الصيدلية | PharmacyDashboardPage | AppShell.xaml |
| لوحة شركة الأدوية | PharmaCompanyDashboardPage | AppShell.xaml |

**تسجل في:** `AppShell.xaml` فقط كـ `<ShellContent>`.
**تستدعى بـ:** `//Route` (شرطتان مائلتان).

**⚠️ بعد 4-5 نوفمبر 2026:** `MainDashboard` يشير إلى `DashboardFlowPage` (الجديدة — النمط 7). `DashboardPage` (القديمة) قد تبقى كـ `ShellContent` مخفي (`Route="LegacyDashboard"`, `FlyoutItemIsVisible="False"`).

#### 2. الصفحات الفرعية (Detail Pages) — كل ما عدا الستة

**تسجل في:** `AppShell.xaml.cs` فقط باستخدام `Routing.RegisterRoute`.
**تستدعى بـ:** `Route` (بدون شرطات).

#### 3. جدول أنواع التنقل الكامل

| الرمز | النوع | التأثير | يستخدم مع | مثال |
|---|---|---|---|---|
| `//Route` | Absolute | يستبدل الصفحة الحالية في المكدس | الصفحات الرئيسية | `//MainDashboard` |
| `///Route` | Absolute كامل | يمسح المكدس بالكامل | ❌ ممنوع الاستخدام | - |
| `Route` | Relative | يضيف صفحة فوق المكدس | الصفحات الفرعية | `doctorprofile` |
| `..` | Back | يرجع صفحة للخلف | أي صفحة | `..` |
| `Route?param=value` | Relative + معاملات | يضيف صفحة مع بيانات | الصفحات الفرعية | `doctorprofile?clinicId=5` |

### 12.2 🔴 أخطاء التنقل الشائعة

#### الخطأ 1: `Ambiguous routes matched for...`

**الرسالة:**
```
System.ArgumentException: Ambiguous routes matched for: 
//.../pageName matches found: //.../pageName, //.../pageName
```

**ماذا يعني:** الصفحة مسجلة مرتين — في `AppShell.xaml` و `AppShell.xaml.cs` معاً.

**الحل:** احذف التسجيل من أحد الملفين.

#### الخطأ 2: `JavaProxyThrowable`

**متى يظهر:** عند العودة من صفحة إلى أخرى (زر الرجوع).

**الأسباب المحتملة:**
- تعارض إصدارات `WebView.Maui` مع `MAUI.Controls` (الأكثر شيوعاً).
- Ambiguous Routes (يظهر معه في نفس الوقت).

**الحل:** تأكد من تطابق إصدارات MAUI في `.csproj`:
```xml
<PackageReference Include="Microsoft.Maui.Controls" Version="10.0.20" />
<PackageReference Include="Microsoft.AspNetCore.Components.WebView.Maui" Version="10.0.20" />
```

#### الخطأ 3: `No view found for id 0x7f0800ff (jumpToEnd)`

**متى يظهر:** بعد Splash Screen مباشرة، قبل تحميل أي صفحة. التطبيق ينهار فوراً.

**ماذا يعني:** استخدام `///` مع صفحة رئيسية أدى إلى تدمير Fragment الداخلي لـ Shell نفسه.

**الأسباب المحتملة:**

| # | السبب | مثال خاطئ |
|---|---|---|
| 1 | `///` مع صفحة رئيسية | `GoToAsync("///ClinicDashboardPage")` |
| 2 | مسار نسبي مع صفحة رئيسية | `GoToAsync("MainDashboard")` بدون `//` |
| 3 | حذف كود XAML معلق دون تنظيف | ترك xmlns references مكسورة في الملف |
| 4 | تسجيل رئيسية في AppShell.xaml.cs | `Routing.RegisterRoute("MainDashboard", ...)` |

**الحل:**
- استبدل كل `///Route` بـ `//Route` للصفحات الرئيسية.
- استبدل كل مسار نسبي لصفحة رئيسية بـ `//Route`.
- نفذ تنظيف عميق:
```bash
rmdir /s /q bin
rmdir /s /q obj
# احذف التطبيق من الهاتف
dotnet restore
dotnet build -f net10.0-android -c Debug -t:Install
```

#### الخطأ 4: `Global routes currently cannot be the only page on the stack`

**ماذا يعني:** تحاول استخدام `//` مع صفحة فرعية (مسجلة في AppShell.xaml.cs).

**الحل:** احذف `//` واستخدم المسار النسبي فقط.

#### الخطأ 5: `unable to figure out route for...`

**ماذا يعني:** الصفحة غير مسجلة في `AppShell.xaml.cs` أو أن المسار غير معروف.

**الحل:** تحقق من `Routing.RegisterRoute` موجود.

**⚠️ مثال حقيقي (5 نوفمبر 2026):** `//DashboardFlowPage` — لأن `Route="DashboardFlow"` (وليس `DashboardFlowPage`). **تحقق من اسم `Route` في `AppShell.xaml`.**

### 12.3 📊 مصفوفة التشخيص السريع

| الخطأ | أول ملف تراجعه | ثاني ملف | ثالث ملف |
|---|---|---|---|
| Ambiguous routes | AppShell.xaml | AppShell.xaml.cs | - |
| JavaProxyThrowable | AppShell.xaml.cs | .csproj | AppShell.xaml |
| jumpToEnd | AppShellViewModel.cs | AppShell.xaml | AppShell.xaml.cs |
| Global routes... | ViewModel المعني | AppShell.xaml.cs | - |
| unable to figure out | AppShell.xaml.cs | ViewModel المعني | AppShell.xaml (Route) |

### 12.4 🛡️ إجراءات الوقاية

- قبل أي Commit: تأكد من عدم وجود `///` في المشروع كله.
- بعد حذف كود XAML معلق: تنظيف عميق إجباري.
- عند إضافة صفحة جديدة: اسأل: "هل هي من الـ 6 الرئيسية؟"
- القاعدة النهائية:
```
رئيسية؟ → AppShell.xaml → //Route
فرعية؟  → AppShell.xaml.cs → Route
رجوع؟   → ..
///     → ممنوع نهائياً
```

---

## 📖 الجزء الثالث عشر: قصة نجاح `PointsDashboardPage`

### 13.1 السياق

صفحة `PointsDashboardPage` كانت لا تفتح في MAUI (تعمل في PWA). الخطأ الظاهر: `JavaProxyThrowable` المبهم.

### 13.2 التشخيص (بعد ساعات من التحقيق)

3 مشاكل متراكبة:
1. `ProgramId` لم يُستقبل أصلاً — كان `[QueryProperty]` يُعيَّن بعد `LoadBlazorComponent()`.
2. `[QueryProperty]` مع `int?` — فشل التحويل.
3. `Action<int>` + async — بدون try/catch.

### 13.3 الحل النهائي

**في `PointsDashboardPage.xaml.cs`:**
```csharp
[QueryProperty(nameof(ProgramIdRaw), "programId")]
public partial class PointsDashboardPage : ContentPage
{
    public string? ProgramIdRaw { get; set; }
    private int? ProgramId => int.TryParse(ProgramIdRaw, out var id) ? id : null;
    private bool _componentLoaded = false;

    protected override void OnAppearing()
    {
        base.OnAppearing();
        if (!_componentLoaded)
        {
            LoadBlazorComponent();
            _componentLoaded = true;
        }
    }
}
```

**في `ProgramDetailsPage.xaml.cs`:**
```csharp
{ "OnGoToPointsDashboard", SafeAsyncAction<int>(GoToPointsDashboardAsync) },
```

### 13.4 النتيجة

✅ الصفحة تفتح بنجاح في MAUI. ✅ الـ Log يظهر القيم الصحيحة (OrgId=1247, ProgramId=1). ✅ لا `JavaProxyThrowable`.

### 13.5 الدروس المستفادة

- `[QueryProperty]` ≠ `Nullable<T>` — استخدم `string`.
- `[QueryProperty]` يُعيَّن بعد `Constructor` — استخدم `OnAppearing` مع flag.
- `Action<T>` + async = `JavaProxyThrowable` — لفّها بـ `try/catch`.
- "الفشل الصامت" في MAUI — قد يكون DI، قد يكون QueryProperty، قد يكون Action.
- `SafeAsyncAction` Helper — يوحد النمط، يقلل التكرار.

---

## 📖 الجزء الثالث عشر-ب: قصة نجاح `DashboardFlowPage` (4-5 نوفمبر 2026)

### 13ب.1 السياق

**محاولات فاشلة متعددة** لاستخدام `DashboardPage.razor` (المشترك) كصفحة رئيسية (`ShellContent`). كل المحاولات فشلت:
- `<RootComponent>` في XAML → `UserProfileId=0`.
- `RootComponents.Add` في `OnAppearing` → لا يعيد تحميل المكون.
- `OnComponentReady` → `OnAfterRender` لا يُستدعى.
- `CurrentUserProfileState` في `OnAppearing` → بعد `OnInitializedAsync`.

### 13ب.2 الحل الناجح

**3 تغييرات حاسمة:**

1. **`DashboardFlowPage.xaml`** — `<RootComponent>` في XAML (بدون `Parameters`).
2. **`DashboardFlowPage.xaml.cs`** — `SetUserProfile` في **`Constructor`** (من `Preferences`).
3. **`DashboardPage.razor`** — `@inject CurrentUserProfileState UserState` (بدلاً من `[Parameter] UserProfileId`).

### 13ب.3 الكود الحاسم (3 أسطر)

**في `DashboardFlowPage.xaml.cs` (Constructor):**
```csharp
var userId = Preferences.Get("user_profile_id", 0);
if (userId > 0)
    _userState.SetUserProfile(userId, Preferences.Get("user_full_name", "مستخدم"));
```

**في `AppShell.InitializeAsync`:**
```csharp
Preferences.Set("user_profile_id", session.UserProfileId);
Preferences.Set("user_full_name", session.FullNameAr);
```

**في `DashboardPage.razor`:**
```csharp
@inject CurrentUserProfileState UserState
```

### 13ب.4 لماذا نجح هذا (بينما فشل غيره)

| السبب | التفسير |
|---|---|
| `SetUserProfile` في `Constructor` | يحدث **قبل** `OnInitializedAsync` (لأن `Constructor` يسبق دورة حياة المكون). |
| `Preferences` (متزامن) | لا يحتاج `await` (بخلاف `_sessionService.GetSessionAsync()`). |
| `<RootComponent>` في XAML | يضيف المكون **قبل** `InitializeComponent` (لا مشكلة ForceRender). |
| `CurrentUserProfileState` (Singleton) | يشارك الحالة بين MAUI و BlazorWebView (لأن `SetUserProfile` في MAUI يعدل **نفس النسخة** التي يقرأها BlazorWebView — **لأن `BlazorWebView` في `DashboardFlowPage` يحقن من حاوية MAUI، وليس من حاوية Blazor**). |

**⚠️ ملاحظة:** `CurrentUserProfileState` (Singleton) يعمل في `DashboardFlowPage` **لأن `RootComponent` في XAML**. في محاولات سابقة (مع `RootComponents.Add`)، لم يعمل — لأن `BlazorWebView` كان يحقن من حاوية مختلفة.

### 13ب.5 الدروس المستفادة

- **`SetUserProfile` في `Constructor`** (وليس `OnAppearing`) = المفتاح.
- **`Preferences` (متزامن)** = الحل لجلب `UserProfileId` قبل `OnInitializedAsync`.
- **`<RootComponent>` في XAML** = يبدأ المكون في الوقت المناسب.
- **`CurrentUserProfileState` (Singleton)** = يعمل مع `<RootComponent>` في XAML (لأن `BlazorWebView` يحقن من حاوية MAUI).
- **`CustomBlazorWebView`** = عدو `RootComponent` (ForceRender يمسح المكون).

---

## 🔧 الجزء الرابع عشر: التحديات المعروفة وحلولها

### 14.1 مشكلة Binding في XAML

**المشكلة:** `InvalidCastException` عند binding لخصائص معقدة.

**الحل:** استخدام خصائص بسيطة في ViewModel.
```csharp
// ❌ خطأ
public PspProgram SelectedProgram { get; set; }

// ✅ صحيح
public string ProgramName { get; set; }
public string ProgramDescription { get; set; }
```

### 14.2 مشكلة UI Thread

**المشكلة:** تحديث UI من خيط غير رئيسي يسبب استثناء.

**الحل:** استخدام `MainThread.BeginInvokeOnMainThread`.
```csharp
MainThread.BeginInvokeOnMainThread(() =>
{
    IsLoading = false;
    Programs = new ObservableCollection<PspProgram>(data);
});
```

### 14.3 مشكلة SecureStorage

**المشكلة:** `SecureStorage` يفشل أحياناً على بعض أجهزة Android.

**الحل:** استخدام `Preferences` كبديل مع تشفير القيمة.
```csharp
public async Task SaveTokenAsync(string token)
{
    try
    {
        await SecureStorage.SetAsync("auth_token", token);
    }
    catch
    {
        // fallback لـ Preferences
        Preferences.Set("auth_token", token);
    }
}
```

### 14.4 مشكلة الاتصال بالإنترنت

**المشكلة:** التطبيق يتجمد عند فقدان الاتصال.

**الحل:** التحقق من الاتصال قبل كل استدعاء API.
```csharp
public async Task<T> SafeApiCallAsync<T>(Func<Task<T>> apiCall)
{
    if (Connectivity.Current.NetworkAccess != NetworkAccess.Internet)
    {
        await ShowError("لا يوجد اتصال بالإنترنت");
        return default;
    }

    try
    {
        return await apiCall();
    }
    catch (HttpRequestException)
    {
        await ShowError("خطأ في الاتصال بالخادم");
        return default;
    }
}
```

### 14.5 ⭐ مشكلة: منظمات المستخدم لا تظهر في القائمة الجانبية

**السبب:** `AppShellViewModel.LoadUserDataAsync()` لم تكن تُستدعى من `DashboardPage`.

**الحل:** إضافة استدعاء `appShell.RefreshUserDataAsync()` في `DashboardPage.OnAppearing`.

### 14.6 ⭐ مشكلة: ظهور بيانات المستخدم السابق بعد Logout

**السبب:** الكاش المحلي (`_cachedSession`) لم يكن يُمسح عند Logout.

**الحل:** استدعاء `_cachedSessionService.ClearSessionAsync()` في `AppShellViewModel.LogoutAsync`.

### 14.7 ⭐ مشكلة: `Console.WriteLine` في BlazorWebView → Logcat

**المشكلة:** `Console.WriteLine` في `.razor` (BlazorWebView) يظهر في **Logcat** (تحت وسم `DOTNET`)، وليس في **Output Window** (Visual Studio).

**الحل:**
- استخدم **Logcat** (Visual Studio: `Tools → Android → Device Log`، أو `adb logcat | findstr "DOTNET"`).
- `Debug.WriteLine` في `.razor` **قد لا يظهر** (لأنه لا يمر عبر `Debug` في MAUI). استخدم `Console.WriteLine`.

---

---

### 14.8 ⭐ مشكلة: "Reload" في BlazorWebView عند تغيير الصفحات

**السبب:** عند تغيير `_currentView` (أو `Navigation.NavigateTo`) داخل `BlazorWebView` (الذي يعمل كـ `ShellContent`)، يفشل `BlazorWebView` في إعادة تهيئة `RootComponent`، فيظهر "Reload" (أو صفحة بيضاء).

**الحل:** 
1. **لا تغيّر `_currentView`** داخل `BlazorWebView` (إذا كان `ShellContent`).
2. **استخدم `NavigationBridge`** لفتح صفحة XAML جديدة (`PushAsync` + `RootComponents.Add` + `Parameters`).
3. **أو** اجعل `BlazorWebView` يعرض **مكوناً واحداً فقط** (الكتالوج، مثلاً)، وكل صفحة أخرى = صفحة XAML جديدة.

**مثال من جلسة Commerce:**
- ❌ `CommerceRootPage` (ShellContent) حاول تغيير `_currentView` إلى `product-details` → `Reload`.
- ✅ الحل: `ICommerceNavigationBridge.OpenProductDetailsAsync` → `PushAsync(CommerceProductDetailsFlow)` → `RootComponents.Add(ProductDetailsPage, Parameters)`.

### 14.9 ⭐ مشكلة: `AppShell.xaml.cs` يشير إلى صفحات غير موجودة

**السبب:** إضافة `Routing.RegisterRoute("commerce/cart", typeof(CommerceCartFlow))` قبل إنشاء `CommerceCartFlow`.

**النتيجة:** `CS0246` (Type or namespace not found).

**الحل:**
1. أنشئ الصفحة (`CommerceCartFlow.xaml` + `.xaml.cs`).
2. سجّلها في `AppShell.xaml.cs`.
3. **لا تسجّل صفحة قبل إنشائها.**

## 📱 الجزء الخامس عشر: إدارة الجلسات والكاش في الموبايل
---
### 15.1 نظرة عامة

إدارة الجلسات في الموبايل تعتمد على 3 طبقات:

| الطبقة | المسؤول | الموقع |
|---|---|---|
| الخادم | `UserSessionService` (Application) | Api.Web |
| محلي | `CachedUserSessionService` | الموبايل |
| تخزين آمن | `SecureStorage` | الموبايل |


### 15.2 `CachedUserSessionService`

**المسار:** `RubikCare.Mobile/Infrastructure/Services/CachedUserSessionService.cs`

**الوظائف الرئيسية:**
- `GetSessionAsync()` — جلب الجلسة من الكاش المحلي أو من API.
- `RefreshSessionAsync()` — تحديث الجلسة من `POST api/user/session-bootstrap`.
- `ClearSessionAsync()` — ⭐ مسح الكاش المحلي (يجب استدعاؤها عند Logout).
- `IsSessionValidAsync()` — التحقق من صلاحية الجلسة.

### 15.3 `AppShellViewModel`

**المسار:** `RubikCare.Mobile/ViewModels/AppShellViewModel.cs`

**الوظائف الرئيسية:**
- `LoadUserDataAsync()` — تحميل بيانات المستخدم ومنظماته.
- `LogoutAsync()` — تسجيل الخروج ومسح جميع البيانات.
- `NavigateToOrganization(org)` — التنقل لمنظمة محددة.

**`AppShellViewModel.LoadUserDataAsync()` يجب أن تُستدعى صراحة من `DashboardPage.OnAppearing`:**
```csharp
// في DashboardPage.xaml.cs
protected override async void OnAppearing()
{
    base.OnAppearing();

    // ⭐ تحديث القائمة الجانبية
    if (Shell.Current is AppShell appShell)
    {
        await appShell.RefreshUserDataAsync();
    }

    // تحميل بيانات Dashboard
    if (BindingContext is DashboardPageViewModel vm)
    {
        await vm.LoadUserDataAsync();
    }
}
```

### 15.4 `AuthService` — المصادقة والخروج

**المسار:** `RubikCare.Mobile/Infrastructure/Services/AuthService.cs`

**تسلسل Logout الصحيح:**
```csharp
public async Task LogoutAsync()
{
    // 1. إعلام الخادم (يمسح كاش UserSessionService)
    try
    {
        await _apiService.PostAsync<object>("api/auth/logout", null);
    }
    catch (Exception ex)
    {
        Debug.WriteLine($"❌ Logout API error: {ex.Message}");
    }

    // 2. مسح التخزين المحلي
    SecureStorage.Remove("auth_token");
    SecureStorage.Remove("user_profile");
    SecureStorage.Remove("userProfileId");
    Preferences.Remove("user_profile_id");  // ⭐ جديد
    Preferences.Remove("user_full_name");   // ⭐ جديد
}
```

### 15.5 (جديد) `CurrentUserProfileState` — Singleton

**المسار:** `Shared.UI/Services/CurrentUserProfileState.cs`

**الوظيفة:** يحمل `UserProfileId` + `UserFullNameAr` بين MAUI و BlazorWebView (Singleton).

**التسجيل في `MauiProgram.cs`:**
```csharp
builder.Services.AddSingleton<CurrentUserProfileState>();
```

**الاستخدام:**
- في `DashboardFlowPage.Constructor`: `_userState.SetUserProfile(userId, userName)`.
- في `DashboardPage.razor`: `@inject CurrentUserProfileState UserState`.
- في `AppShell.InitializeAsync`: `Preferences.Set("user_profile_id", ...)`.

### 15.6 (جديد) `DashboardNavigationBridge` — Static Actions

**المسار:** `Shared.UI/Components/Dashboard/DashboardNavigationBridge.cs`

**الوظيفة:** جسر للتنقل بين Blazor و MAUI (بدلاً من `EventCallback`).

**البنية:**
```csharp
public static class DashboardNavigationBridge
{
    public static Action<string>? Navigate { get; set; }
    public static Action? OpenMenu { get; set; }
}
```

**الاستخدام:**
- في `DashboardFlowPage.Constructor`: `DashboardNavigationBridge.Navigate = async (path) => { ... }`.
- في `DashboardPage.razor`: `DashboardNavigationBridge.Navigate?.Invoke("/medication-schedule")`.

---

## 🎨 الجزء السادس عشر: الأنماط المعمارية في MAUI

### 16.1 نمط MVVM

*(كما في الإصدار السابق — لم يتغير.)*

### 16.2 `BaseViewModel`

*(كما في الإصدار السابق — لم يتغير.)*

---

## 🔗 الجزء السابع عشر: Bridges (التواصل بين Blazor و MAUI)

### 17.1 إنشاء Bridge

**في `Shared.UI/Components/Rep/RepNavigationBridge.cs`:**
```csharp
public static class RepNavigationBridge
{
    public static Action<int>? OnProgramSelected { get; set; }
    public static Func<string, string, Task>? ShareToken { get; set; }
}
```

### 17.2 تسجيل Bridge في XAML.cs
```csharp
RepNavigationBridge.ShareToken = async (token, programName) =>
{
    var message = $"🎫 كود الدعوة: {token}";
    await Share.Default.RequestAsync(new ShareTextRequest
    {
        Text = message,
        Title = $"دعوة لبرنامج {programName}"
    });
};
```

### 17.3 استخدام Bridge في Razor
```csharp
private async Task ShareToken()
{
    if (RepNavigationBridge.ShareToken != null)
        await RepNavigationBridge.ShareToken(_token, _programName);
}
```

### 17.4 ✅ ربط جميع دوال الـ Bridge في `OnAppearing`

**القاعدة الذهبية:** أي دالة في `PspDetailNavigationBridge` (أو أي Bridge آخر) يجب ربطها من الصفحة المضيفة (`.xaml.cs`) **قبل** استخدامها في المكون. ربط بعض الدوال ونسيان البعض الآخر هو سبب شائع لـ "زر لا يعمل".

**Checklist سريعة:**
- افتح ملف الـ Bridge (`PspDetailNavigationBridge.cs`).
- لكل `Action` أو `Func` في الـ Bridge، تأكد من وجود سطر ربط مقابل في `OnAppearing` في الصفحة المضيفة.
- استخدم `Dispatcher.DispatchAsync` أو `MainThread.BeginInvokeOnMainThread` داخل كل `Action` لتجنب `JavaProxyThrowable`.
- تأكد من أن المسارات (`medication-schedule`, `pharmacy-search`) مسجلة في `AppShell.xaml.cs`.

### 17.5 (جديد) `DashboardNavigationBridge` — مثال حي

**في `DashboardFlowPage.Constructor`:**
```csharp
DashboardNavigationBridge.Navigate = async (path) =>
{
    await Dispatcher.DispatchAsync(async () =>
    {
        try
        {
            var mauiRoute = ConvertToMauiRoute(path);
            if (mauiRoute != null) await Shell.Current.GoToAsync(mauiRoute);
        }
        catch (Exception ex) { Debug.WriteLine($"❌ [Navigate] {ex}"); }
    });
};
```

**في `DashboardPage.razor`:**
```csharp
private async Task HandleStatsCardClick(string card)
{
    switch (card)
    {
        case "schedule": DashboardNavigationBridge.Navigate?.Invoke("/medication-schedule"); break;
        // ...
    }
    await Task.CompletedTask;
}
```

### 17.6 (جديد) `ConvertToMauiRoute` — تحويل مسارات PWA إلى MAUI

**في `DashboardFlowPage.xaml.cs` (أو أي صفحة host):**
```csharp
private string? ConvertToMauiRoute(string path)
{
    if (path.StartsWith("/psp-detail/"))
    {
        var parts = path.Trim('/').Split('/');
        if (parts.Length >= 4)
            return $"PspDetailFlow?patientId={parts[2]}&participationId={parts[3]}&programId={parts[1]}";
    }
    if (path == "/psp-entry") return "PspEntryPage";
    if (path.StartsWith("/medication-schedule")) return "medication-schedule";
    if (path == "/doctor-search") return "search-doctor";
    if (path.StartsWith("/pharmacy-search")) return "pharmacy-search";
    if (path.StartsWith("/patient/points")) return $"patient-points?userProfileId={_userState.UserProfileId}";
    
    // ⭐ medicine-guide: استخراج caseId و programId من path
    if (path.StartsWith("/medicine-guide"))
    {
        var queryIndex = path.IndexOf('?');
        if (queryIndex >= 0)
        {
            var query = path[(queryIndex + 1)..];
            var queryParams = query.Split('&')
                .Select(p => p.Split('='))
                .Where(p => p.Length == 2)
                .ToDictionary(p => p[0], p => p[1]);
            
            var caseId = queryParams.TryGetValue("caseId", out var c) ? c : "0";
            var programId = queryParams.TryGetValue("programId", out var p) ? p : "0";
            return $"medicine-guide?caseId={caseId}&programId={programId}";
        }
        return "medicine-guide";
    }
    
    if (path == "/psp/about") return "PspAboutFlow";
    if (path == "/notifications") return "notifications";
    return null;
}
```

**⚠️ تحذير:** `ConvertToMauiRoute` يجب أن يمرر `QueryString` كاملاً (مثل `caseId` + `programId`). **نسيان ذلك يسبب `caseId=0`** (كما حدث في 5 نوفمبر).

### 17.7 📊 الخلاصة

| العنصر | الحالة |
|---|---|
| صفحة النقاط | ✅ تعمل (بعد حل `InvalidCastException` بتحويل `userId` إلى `string`) |
| زر جدول الأدوية | ✅ يعمل (بعد ربط `GoToSchedule` في `DashboardFlowPage`) |
| زر البحث عن صيدلية | ✅ يعمل (بعد ربط `GoToPharmacySearch` في `DashboardFlowPage`) |
| `DashboardFlowPage` كصفحة رئيسية | ✅ يعمل (النمط 7 — `CurrentUserProfileState` + `<RootComponent>` في XAML). |
| `DashboardPage.razor` في MAUI | ✅ يعمل (بعد تعديل `DashboardPage.razor` لاستخدام `CurrentUserProfileState`). |

---
### 17.8 (جديد) ICommerceNavigationBridge — مثال حي (Commerce)

**الغرض:** فتح صفحة تفاصيل المنتج (`ProductDetailsPage`) من `CommerceRootPage` (الذي يعمل في `BlazorWebView`).

**البنية (Shared.UI):**
```csharp
// Shared.UI/Services/ICommerceNavigationBridge.cs
public interface ICommerceNavigationBridge
{
    Task OpenProductDetailsAsync(int medicationId, int companyOrgId);
}
البنية (Mobile):

csharp
// Mobile/Services/CommerceNavigationBridge.cs
public class CommerceNavigationBridge : ICommerceNavigationBridge
{
    public async Task OpenProductDetailsAsync(int medicationId, int companyOrgId)
    {
        await MainThread.InvokeOnMainThreadAsync(async () =>
        {
            if (Shell.Current != null)
                await Shell.Current.Navigation.PushAsync(
                    new CommerceProductDetailsFlow(medicationId, companyOrgId));
        });
    }
}
```
التسجيل (MauiProgram.cs):

```csharp
builder.Services.AddSingleton<ICommerceNavigationBridge, CommerceNavigationBridge>();
الاستخدام (CommerceRootPage.razor):

razor
@inject ICommerceNavigationBridge NavigationBridge

private async Task GoToMedicationDetails((int MedicationId, int CompanyOrgId) payload)
{
    await NavigationBridge.OpenProductDetailsAsync(payload.MedicationId, payload.CompanyOrgId);
}
```
النتيجة: صفحة تفاصيل المنتج تفتح بشكل صحيح، بدون Reload، مع Shell.NavBar و Breadcrumb.

---

## 🚀 الجزء الثامن عشر: النشر (Publishing)

### 18.1 Android (APK/AAB)
```bash
# إنشاء APK
dotnet publish -f net10.0-android -c Release /p:AndroidPackageFormat=apk

# إنشاء AAB (Google Play)
dotnet publish -f net10.0-android -c Release /p:AndroidPackageFormat=aab
```

### 18.2 iOS
```bash
# إنشاء IPA
dotnet publish -f net10.0-ios -c Release
```

---

## ✅ الجزء التاسع عشر: CHECKLIST

### 19.1 عند إضافة صفحة BlazorWebView جديدة

#### MAUI Layer (صفحة الحاوية)

- هل الصفحة رئيسية (`ShellContent`) أم فرعية (`Routing.RegisterRoute`)؟
- **إذا رئيسية:**
  - هل تستخدم `BlazorWebView` + `<RootComponent>` في XAML (النمط 2 أو 7)؟
  - إذا كانت تحتاج `UserProfileId` → هل تستخدم `CurrentUserProfileState` (النمط 7)؟
  - هل `SetUserProfile` يُستدعى في `Constructor` (قبل `OnInitializedAsync`)؟
  - هل `Preferences.Set("user_profile_id", ...)` موجود في `AppShell.InitializeAsync`؟
- **إذا فرعية:**
  - هل تستخدم `RootComponents.Add` في `OnAppearing` أو `ApplyQueryAttributes` (النمط 3 أو 8)؟
  - هل `[QueryProperty]` من نوع `string` (وليس `Nullable<T>`)؟
  - هل `LoadBlazorComponent` في `OnAppearing` (أو `ApplyQueryAttributes`) وليس `Constructor`؟
- هل `RootComponents.Add` مع `Parameters` (بما فيها `EventCallback` و `Action`)؟
- هل `Action<T>` مع `async` ملفوف في `MainThread.BeginInvokeOnMainThread` + `try/catch` (أو `SafeAsyncAction`)؟

#### Blazor Component (المكون)

- هل المكون لا يستخدم `@inject HttpClient` أو `@inject ApiService`؟
- هل كل خدمة MAUI تُستخدم عبر `[Parameter]` وليس `@inject` (ما عدا الترجمة و `NavigationManager` و `Current...State` Singleton)؟
- هل `OnInitializedAsync` تُستدعى؟ (تأكد من `Console.WriteLine`).
- هل المكون يعرض Skeleton/Loading أثناء جلب البيانات؟
- هل توجد `public methods` للتحديث من الحاوية عند الحاجة (النمط 4/5)؟
- هل الـ `Parameters` من نوع `Action<T>` (وليس `EventCallback`)؟
- [ ] هل الصفحة `ShellContent` (رئيسية) أم `Routing.RegisterRoute` (فرعية)؟
- [ ] إذا كانت `ShellContent`: هل تحتاج `Parameters` ديناميكية؟
  - [ ] نعم → استخدم `XAML محلي` أو `CurrentUserProfileState (Singleton) + <RootComponent> في XAML`.
  - [ ] لا → يمكن استخدام `BlazorWebView + <RootComponent> في XAML` (النمط 2).
- [ ] إذا كانت `Routing.RegisterRoute`:
  - [ ] هل تحتاج `Parameters`؟ → `RootComponents.Add` (في Constructor أو `ApplyQueryAttributes`).
  - [ ] هل تحتاج `Shell.NavBar`؟ → `PushAsync` (بدلاً من `PushModalAsync`) + `Shell.NavBarIsVisible="True"`.
  - [ ] هل تحتاج `Shared.UI` للوصول إلى `Mobile`؟ → استخدم **Bridge Pattern** (`ICommerceNavigationBridge`).
- [ ] هل `AppShell.xaml.cs` يشير إلى صفحة موجودة؟ (لا تسجّل صفحة قبل إنشائها).
- [ ] هل `OnBack` هو `EventCallback` (وليس `Action`)، لتجنب `JavaProxyThrowable`؟

### 19.2 عند إضافة صفحة MAUI جديدة

- هل أنشأت مجلداً في `Features/{Domain}/Views/`؟
- هل أنشأت ViewModel مطابقاً في `Features/{Domain}/ViewModels/`؟
- هل ورثت من `BaseViewModel`؟
- هل استخدمت `SafeApiCallAsync` لاستدعاءات API؟
- هل تعاملت مع حالة عدم الاتصال بالإنترنت؟
- هل اختبرت على Android و iOS؟
- هل تعاملت مع UI Thread بشكل صحيح؟

### 19.3 عند التعامل مع الجلسات والكاش

- هل `AppShellViewModel.LoadUserDataAsync()` تُستدعى من `DashboardPage.OnAppearing`؟
- هل `LogoutAsync` يمسح `_cachedSession` + `SecureStorage` + `Preferences` + يستدعي `api/auth/logout`؟
- هل `HasClinic`/`HasPharmacy` تستخدمان `Memberships` وليس `Clinics`/`Pharmacies`؟

### 19.4 (جديد) عند استخدام `CurrentUserProfileState`

- هل `CurrentUserProfileState` مسجل في `MauiProgram.cs` كـ `AddSingleton`؟
- هل `SetUserProfile` يُستدعى في `Constructor` (قبل `OnInitializedAsync`)؟
- هل `Preferences.Set("user_profile_id", ...)` موجود في `AppShell.InitializeAsync`؟
- هل `DashboardPage.razor` يستخدم `@inject CurrentUserProfileState UserState`؟
- هل `OnUserStateChanged` يستدعي `LoadAllDataAsync` (إذا كانت `UserProfileId` موجودة)؟

---

## الجزء 20 — دروس مستفادة (Lessons Learned)

### 20.1 🎯 قاعدة ذهبية: `JavaProxyThrowable` ليس الخطأ — إنه الغلاف

`JavaProxyThrowable` هو غلاف يلف أي Exception يحدث داخل الكود المُدار (C#) عند عبوره إلى بيئة Android. الخطأ الحقيقي موجود في Output Window (Visual Studio) — في السطور التي تسبق `JavaProxyThrowable` مباشرة.

**⚠️ لكن في BlazorWebView:** `Console.WriteLine` في `.razor` يظهر في **Logcat** (تحت وسم `DOTNET`)، وليس في Output Window.

**القاعدة:** قبل أي تخمين، افتح Logcat (وليس Output Window) وابحث عن أول Exception (وليس `JavaProxyThrowable`). هذا هو السبب الحقيقي.

### 20.2 ⚠️ فخ `[QueryProperty]` — التحويل يجب أن يكون من الطرفين

*(كما في الإصدار السابق — لم يتغير.)*

### 20.3 ⚠️ `GoToAsync` — يجب أن يكون داخل `Dispatcher.DispatchAsync`

*(كما في الإصدار السابق — لم يتغير.)*

### 20.4 ⚠️ Bridge — يجب ربط كل الدوال في `OnAppearing` (وليس بعضها)

*(كما في الإصدار السابق — لم يتغير.)*

### 20.5 📋 Checklist "زر لا يعمل في MAUI"

*(كما في الإصدار السابق — لم يتغير.)*

### 20.6 🆕 جدول: الفرق بين "زر لا يعمل" و "زر لا يفتح"

*(كما في الإصدار السابق — لم يتغير.)*

### 20.7 ✅ Checklist "قبل تشغيل التطبيق" (لأي صفحة MAUI جديدة)

*(كما في الإصدار السابق — لم يتغير.)*

### 20.8 ⚠️ `patientId=0` — CaseID مفقود في DispensationScheduleDto

*(كما في الإصدار السابق — لم يتغير.)*

### 20.9 ⚠️ `PspDetailFlow` — تحويل PatientIdRaw (string) + OnAppearing

*(كما في الإصدار السابق — لم يتغير.)*

### 20.10 ⚠️ `MedicationScheduleFlow` — ربط OnNavigateToPsp مع Dispatcher + CaseID

*(كما في الإصدار السابق — لم يتغير.)*

### 20.11 ⚠️ `PatientPointsFlow` (MAUI) — Wrapper + PspDetailNavigationBridge

*(كما في الإصدار السابق — لم يتغير. لكن `PatientPointsFlow` الآن يعمل — بعد تمرير `userProfileId` في `ConvertToMauiRoute`.)*

### 20.12 ⚠️ `ScanTokenPage.razor` (القديم) — حذفه واستخدام `PharmacyScanTokenPage`

*(كما في الإصدار السابق — لم يتغير.)*

### 20.13 ⚠️ `PspSchedulePageFlow` — تمرير PatientId

*(كما في الإصدار السابق — لم يتغير.)*

### 20.14 🆕 ⚠️ `CurrentUserProfileState` (Singleton) — نسختان (MAUI + BlazorWebView) — **لكن يعمل في `DashboardFlowPage`**

**المشكلة (في المحاولات السابقة):** `CurrentUserProfileState` مسجل في `MauiProgram.cs` كـ `AddSingleton`. `DashboardPage.xaml.cs` (MAUI) يحقن نسخة من حاوية MAUI، بينما `DashboardPage.razor` (BlazorWebView) يحقن نسخة من حاوية Blazor. `SetUserProfile(15, ...)` في MAUI لا يؤثر على `DashboardPage.razor`.

**الحل (الناجح — 4-5 نوفمبر 2026):** `DashboardFlowPage.xaml` — `<RootComponent>` في XAML (بدون `Parameters`). `DashboardFlowPage.xaml.cs` — `SetUserProfile` في `Constructor` (من `Preferences`). **`BlazorWebView` في `DashboardFlowPage` يحقن `CurrentUserProfileState` من حاوية MAUI** (لأن `RootComponent` في XAML — وليس `RootComponents.Add`). **لذا، `SetUserProfile` في MAUI يعدل نفس النسخة التي يقرأها `DashboardPage.razor`.**

**⚠️ القاعدة:** `CurrentUserProfileState` (Singleton) **يعمل فقط** إذا كان `RootComponent` في XAML (وليس `RootComponents.Add`). **مع `RootComponents.Add`، نسختان.**

### 20.15 🆕 ⚠️ `OnAfterRender` في `DashboardPage.razor` لا يُستدعى عندما لا يُحمَّل المكون

**المشكلة:** `OnAfterRender(firstRender)` (الذي يستدعي `OnComponentReady`) **لم يُستدعى** في Logcat — لأن `DashboardPage.razor` لم يكمل دورة حياته (بسبب `RootComponents.Add` بعد بدء `BlazorWebView`).

**الإجراء الوقائي:**
- **للصفحات الفرعية:** `OnAfterRender` يعمل (لأن `RootComponents.Add` يبدأ المكون قبل `OnAfterRender`).
- **للصفحات الرئيسية:** لا تعتمد على `OnAfterRender` (المكون قد لا يكتمل تحميله).

**المرجع:** `13-MAUI` § 6.5.

### 20.16 🆕 ⚠️ `Console.WriteLine` في `BlazorWebView` يظهر في Logcat (وليس Output Window)

**المشكلة:** `Console.WriteLine` في `DashboardPage.razor` (BlazorWebView) ظهر في **Logcat** (تحت وسم `DOTNET`)، وليس في **Output Window** (Visual Studio). هذا جعل التشخيص صعباً.

**الإجراء الوقائي:**
- **لتصحيح `BlazorWebView`:** استخدم **Logcat** (Visual Studio: `Tools → Android → Device Log`، أو `adb logcat | findstr "DOTNET"`).
- **`Debug.WriteLine`** في BlazorWebView **قد لا يظهر** (لأنه لا يمر عبر `Debug` في MAUI). استخدم `Console.WriteLine`.

### 20.17 🆕 ⚠️ `Route` في `ShellContent` — تحقق من الاسم بدقة

**المشكلة (5 نوفمبر 2026):** `GoToAsync("//DashboardFlowPage")` — لكن `Route="DashboardFlow"` (وليس `DashboardFlowPage`). **`Shell` لا يعرف مساراً باسم `DashboardFlowPage`.** النتيجة: `unable to figure out route for: //DashboardFlowPage`.

**الإجراء الوقائي:**
- تحقق من اسم `Route` في `AppShell.xaml` قبل استخدام `GoToAsync("//...")`.
- استخدم `nameof(Page)` إذا كان `Route` = `nameof(Page)`.

---

## 🔗 الجزء الحادي والعشرون: روابط ذات صلة

- [00 — MASTER-INDEX](./00-MASTER-INDEX.md)
- [00B — PROJECT-BACKLOG](./00B-PROJECT-BACKLOG.md)
- [01 — ARCHITECTURE](./01-ARCHITECTURE.md)
- [04 — TRANSLATION](./04-TRANSLATION.md)
- [10 — UI-STYLE-GUIDE](./10-UI-STYLE-GUIDE.md)
- [11 — PAGE-CREATION](./11-PAGE-CREATION.md)
- [12 — CACHING](./12-CACHING.md)
- [14 — PWA](./14-PWA.md)
- [16 — TROUBLESHOOTING](./16-TROUBLESHOOTING.md)

---

## 📝 الجزء الثاني والعشرون: سجل التغييرات

| الإصدار | التاريخ | التغييرات |
|---|---|---|
| 1.0 | 24 مايو 2026 | الإصدار الأولي — `10-maui-development-guide.md` |
| 1.0 | 8 يوليو 2026 | الإصدار الأولي — `11-blazor-webview-guide.md` v1.0 |
| 2.0 | 8 أكتوبر 2026 | `[QueryProperty]` + Nullable + `Action<T>` + async + قصة نجاح `PointsDashboardPage` |
| 3.0 | 10 أكتوبر 2026 | `SafeAsyncAction` Helper + جدول مقارنة `[QueryProperty]` vs `IQueryAttributable` |
| 2.0 | 17 أكتوبر 2026 | 13-MAUI v2.0 — دمج 10 + 11 + قاعدة تسجيل الصفحات + Bridges + إشارات |
| 3.1 | 23 أكتوبر 2026 | الجزء 20 — إضافة 7 أقسام (20.1 → 20.7): `JavaProxyThrowable` + `[QueryProperty]` من الطرفين + `Dispatcher.DispatchAsync` + ربط Bridge الشامل + Checklist "زر لا يعمل" + جدول "زر لا يعمل vs زر لا يفتح" + Checklist "قبل التشغيل" |
| 3.2 | 25 أكتوبر 2026 | الجزء 20 — إضافة 6 أقسام (20.8 → 20.13): `patientId=0` (CaseID مفقود) + `PspDetailFlow` (QueryProperty + OnAppearing) + `MedicationScheduleFlow` (Dispatcher + CaseID) + `PatientPointsFlow` (Wrapper + Bridge بدلاً من EventCallback) + `ScanTokenPage` (حذف + استخدام PharmacyScanTokenPage) + `PspSchedulePageFlow` (تمرير PatientId) |
| **4.0** | **5 نوفمبر 2026** | **إعادة كتابة شاملة (Proven Patterns Edition). إضافة الجزء الأول (النمط الناجح — 5 أنماط ناجحة + 4 أنماط فاشلة + جدول القرار السريع). إعادة تنظيم § 6 (الأنماط الخمسة الفعلية) + § 6.6 (النمط 6: `CurrentUserProfileState` + `<RootComponent>` في XAML + النمط 7: `RootComponents.Add` في `ApplyQueryAttributes`). تحديث § 10 (`CustomBlazorWebView` — للصفحات الفرعية فقط). تحديث § 11 (تشخيص الصفحة الفارغة — 7 خطوات). تحديث § 12 (Route — تحقق من الاسم). تحديث § 13ب (قصة نجاح `DashboardFlowPage`). تحديث § 17 (Bridges — `ConvertToMauiRoute` + `DashboardNavigationBridge`). تحديث § 19 (Checklist — إضافة 19.4). إضافة § 20.14-20.17 (4 دروس جديدة).** |

---

**© 2026 RubikCare — للاستخدام الداخلي**

**نهاية الوثيقة ✅**
