## 📝 ملف `10-maui-development-guide.md` بعد التحديث:

# 10 - دليل تطوير MAUI

**آخر تحديث: 24 مايو 2026**

---

## مقدمة

هذا المرجع هو الدليل الكامل لتطوير تطبيق RubikCare للموبايل باستخدام .NET MAUI. يغطي هيكل المشروع، الأنماط المتبعة، التحديات المعروفة، وحلولها.

---

## هيكل مشروع MAUI

```
RubikCare.Mobile/
├── App.xaml / App.xaml.cs              # نقطة بداية التطبيق
├── AppShell.xaml / AppShell.xaml.cs    # التنقل الرئيسي (Shell)
├── MauiProgram.cs                      # تسجيل الخدمات
│
├── Features/                           # الميزات مقسمة حسب المجال
│   ├── Auth/
│   │   ├── Views/
│   │   │   ├── LoginView.xaml
│   │   │   └── RegisterPage.xaml
│   │   └── ViewModels/
│   │       ├── LoginViewModel.cs
│   │       └── RegisterViewModel.cs
│   │
│   ├── PSP/
│   │   ├── Doctor/
│   │   │   ├── Views/
│   │   │   │   ├── PspSearchPage.xaml
│   │   │   │   ├── ProgramDetailsPage.xaml
│   │   │   │   └── InvitePatientPage.xaml
│   │   │   └── ViewModels/
│   │   ├── Patient/
│   │   │   ├── Views/
│   │   │   │   ├── PspEntryPage.xaml
│   │   │   │   └── PspDetailPage.xaml
│   │   │   └── ViewModels/
│   │   └── Pharmacist/
│   │       ├── Views/
│   │       │   ├── ScanTokenPage.xaml
│   │       │   └── VerifyTokenPage.xaml
│   │       └── ViewModels/
│   │
│   ├── PublicUser/
│   │   ├── Views/
│   │   │   ├── DashboardPage.xaml
│   │   │   └── ProfilePage.xaml
│   │   └── ViewModels/
│   │       ├── DashboardPageViewModel.cs
│   │       └── AppShellViewModel.cs
│   │
│   └── Shared/
│       └── Views/ (صفحات مشتركة)
│
├── Infrastructure/
│   └── Services/
│       ├── ApiService.cs               # التواصل مع Api.Web
│       ├── AuthService.cs              # المصادقة وإدارة التوكن
│       ├── CachedUserSessionService.cs # ⭐ كاش الجلسة المحلي
│       └── AppStateService.cs          # حالة التطبيق المشتركة
│
├── ViewModels/
│   └── AppShellViewModel.cs            # ⭐ ViewModel القائمة الجانبية
│
├── Converters/                         # محولات XAML
├── Resources/                          # موارد التطبيق
│   ├── Fonts/
│   ├── Images/
│   └── Styles/
│
└── Platforms/                          # كود خاص بكل منصة
    ├── Android/
    └── iOS/
```

---

## إدارة الجلسات والكاش في الموبايل (⭐ جديد - 24 مايو 2026)

### نظرة عامة

إدارة الجلسات في الموبايل تعتمد على **ثلاث طبقات**:

| الطبقة | المسؤول | الموقع |
|--------|---------|--------|
| **الخادم** | `UserSessionService` (Application) | Api.Web |
| **محلي** | `CachedUserSessionService` | الموبايل |
| **تخزين آمن** | `SecureStorage` | الموبايل |

### CachedUserSessionService

**المسار:** `RubikCare.Mobile/Infrastructure/Services/CachedUserSessionService.cs`

**الوظائف الرئيسية:**
- `GetSessionAsync()` - جلب الجلسة من الكاش المحلي أو من API
- `RefreshSessionAsync()` - تحديث الجلسة من `POST api/user/session-bootstrap`
- `ClearSessionAsync()` - **⭐ مسح الكاش المحلي (يجب استدعاؤها عند Logout)**
- `IsSessionValidAsync()` - التحقق من صلاحية الجلسة

### AppShellViewModel

**المسار:** `RubikCare.Mobile/ViewModels/AppShellViewModel.cs`

**الوظائف الرئيسية:**
- `LoadUserDataAsync()` - تحميل بيانات المستخدم ومنظماته
- `LogoutAsync()` - تسجيل الخروج ومسح جميع البيانات
- `NavigateToOrganization(org)` - التنقل لمنظمة محددة

**⚠️ نقطة حرجة:** `AppShellViewModel.LoadUserDataAsync()` يجب أن تُستدعى صراحة من `DashboardPage.OnAppearing`:

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

### AuthService - المصادقة والخروج

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
}
```

---

## الأنماط المعمارية في MAUI

### نمط MVVM (Model-View-ViewModel)

```csharp
// ViewModel مثال:
public class LoginViewModel : BaseViewModel
{
    private readonly IAuthService _authService;
    private readonly INavigationService _navigationService;

    private string _email;
    public string Email
    {
        get => _email;
        set { _email = value; OnPropertyChanged(); }
    }

    private string _password;
    public string Password
    {
        get => _password;
        set { _password = value; OnPropertyChanged(); }
    }

    private bool _isLoading;
    public bool IsLoading
    {
        get => _isLoading;
        set { _isLoading = value; OnPropertyChanged(); }
    }

    public ICommand LoginCommand { get; }

    public LoginViewModel(IAuthService authService, INavigationService navigationService)
    {
        _authService = authService;
        _navigationService = navigationService;
        LoginCommand = new Command(async () => await LoginAsync());
    }

    private async Task LoginAsync()
    {
        if (IsLoading) return;
        IsLoading = true;

        try
        {
            var result = await _authService.LoginAsync(Email, Password);
            if (result.Success)
            {
                await _navigationService.GoToMainPage();
            }
            else
            {
                await ShowError(result.Message);
            }
        }
        finally
        {
            IsLoading = false;
        }
    }
}
```

### BaseViewModel

```csharp
public class BaseViewModel : INotifyPropertyChanged
{
    public event PropertyChangedEventHandler PropertyChanged;

    protected void OnPropertyChanged([CallerMemberName] string propertyName = null)
    {
        PropertyChanged?.Invoke(this, new PropertyChangedEventArgs(propertyName));
    }

    protected async Task ShowError(string message)
    {
        await Application.Current.MainPage.DisplayAlert("خطأ", message, "موافق");
    }

    protected async Task ShowSuccess(string message)
    {
        await Application.Current.MainPage.DisplayAlert("نجاح", message, "موافق");
    }
}
```

---

## ApiService - التواصل مع الخادم

```csharp
public class ApiService : IApiService
{
    private readonly HttpClient _httpClient;
    private readonly IAuthService _authService;

    public ApiService(IAuthService authService)
    {
        _httpClient = new HttpClient
        {
            BaseAddress = new Uri(AppConfig.ApiBaseUrl)
        };
        _authService = authService;
    }

    public async Task<T> GetAsync<T>(string endpoint)
    {
        await SetAuthHeader();
        var response = await _httpClient.GetAsync(endpoint);
        response.EnsureSuccessStatusCode();
        var json = await response.Content.ReadAsStringAsync();
        return JsonSerializer.Deserialize<T>(json);
    }

    public async Task<T> PostAsync<T>(string endpoint, object data)
    {
        await SetAuthHeader();
        var content = new StringContent(
            JsonSerializer.Serialize(data),
            Encoding.UTF8,
            "application/json");
        var response = await _httpClient.PostAsync(endpoint, content);
        response.EnsureSuccessStatusCode();
        var json = await response.Content.ReadAsStringAsync();
        return JsonSerializer.Deserialize<T>(json);
    }

    private async Task SetAuthHeader()
    {
        var token = await _authService.GetTokenAsync();
        if (!string.IsNullOrEmpty(token))
        {
            _httpClient.DefaultRequestHeaders.Authorization = 
                new AuthenticationHeaderValue("Bearer", token);
        }
    }
}
```

---

## التحديات المعروفة وحلولها

### 1. مشكلة Binding في XAML

**المشكلة:** InvalidCastException عند binding لخصائص معقدة.

**الحل:** استخدام خصائص بسيطة في ViewModel وتجنب binding المباشر للكائنات المعقدة.

```csharp
// ❌ خطأ
public PspProgram SelectedProgram { get; set; }

// ✅ صحيح
public string ProgramName { get; set; }
public string ProgramDescription { get; set; }
```

### 2. مشكلة UI Thread

**المشكلة:** تحديث UI من خيط غير رئيسي يسبب استثناء.

**الحل:** استخدام `MainThread.BeginInvokeOnMainThread`.

```csharp
MainThread.BeginInvokeOnMainThread(() =>
{
    IsLoading = false;
    Programs = new ObservableCollection<PspProgram>(data);
});
```

### 3. مشكلة SecureStorage

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

### 4. مشكلة الاتصال بالإنترنت

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

### 5. ⭐ مشكلة: منظمات المستخدم لا تظهر في القائمة الجانبية (تم حلها - 24 مايو 2026)

**السبب:** `AppShellViewModel.LoadUserDataAsync()` لم تكن تُستدعى من `DashboardPage`.

**الحل:** إضافة استدعاء `appShell.RefreshUserDataAsync()` في `DashboardPage.OnAppearing`.

### 6. ⭐ مشكلة: ظهور بيانات المستخدم السابق بعد Logout (تم حلها - 24 مايو 2026)

**السبب:** الكاش المحلي (`_cachedSession`) لم يكن يُمسح عند Logout.

**الحل:** استدعاء `_cachedSessionService.ClearSessionAsync()` في `AppShellViewModel.LogoutAsync`.

---### 7. ⭐ مشكلة: نظام الملاحة (Navigation) - الحل النهائي (تم حلها - 26 مايو 2026)

**الأعراض:**
*   ضغط زر الرجوع في Android يغلق التطبيق مع ظهور استثناء `JavaProxyThrowable`.
*   عدم ظهور زر الرجوع (⬅️) في بعض الصفحات، وظهور زر القائمة (☰) بدلاً منه.
*   ظهور خطأ `Ambiguous routes matched for...` عند محاولة التنقل بين الصفحات.
*   ظهور خطأ `Global routes currently cannot be the only page on the stack`.

**السبب الجذري:**
1.  **تضارب المسارات (Ambiguous Routes):** تم تسجيل نفس الصفحة في كل من `AppShell.xaml` (كـ `<ShellContent>`) و `AppShell.xaml.cs` (باستخدام `Routing.RegisterRoute`)، مما أربك نظام التوجيه.
2.  **إساءة استخدام المسارات المطلقة (`///`):** استخدام `///` يمسح مكدس التنقل (Navigation Stack) مما يمنع العودة للخلف. كان مرتبطًا بخاصية `Shell.NavBarIsVisible`.

**الحل النهائي: إعادة هيكلة نظام الملاحة**
تم اعتماد القواعد التالية بشكل صارم:

1.  **`AppShell.xaml` للصفحات الرئيسية (Root Pages) فقط:**
    *   يحتوي على 6 صفحات رئيسية: `LoginPage`, `RegisterPage`, `MainDashboard`, `ClinicDashboardPage`, `PharmacyDashboardPage`, `PharmaCompanyDashboardPage`.
    *   هذه الصفحات هي الوحيدة التي يتم التنقل إليها باستخدام المسار المطلق `//Route`.

2.  **`AppShell.xaml.cs` لجميع الصفحات الفرعية:**
    *   يتم تسجيل جميع الصفحات الأخرى هنا باستخدام `Routing.RegisterRoute("routeName", typeof(PageType))`.
    *   يتم التنقل إليها باستخدام **المسار النسبي** فقط (بدون `//`).

3.  **حذف `///` من كل أوامر التنقل:**
    *   تم استبدال كل استدعاءات `GoToAsync("///route")` بـ `GoToAsync("route")` للصفحات الفرعية.

**القاعدة الذهبية:**
> `AppShell.xaml` للرئيسية فقط، `AppShell.xaml.cs` للفرعية، والتنقل يكون نسبيًا بدون `//`. `///` = مسح كامل للمكدس، لا يُستخدم أبدًا للتنقل الطبيعي.

```
```

## BlazorWebView في MAUI

### التكامل بين XAML و Blazor

```
RubikCare.Mobile/
├── Features/
│   ├── PSP/                     # صفحات XAML (موجودة)
│   │   ├── Doctor/Views/
│   │   ├── Patient/Views/
│   │   └── Pharmacist/Views/
│   │
│   └── BlazorPages/             # صفحات Blazor (جديدة)
│       ├── wwwroot/
│       │   └── index.html
│       ├── _Imports.razor
│       └── Components/
│           └── PspDashboard.razor
```

### صفحة XAML تستضيف BlazorWebView

```xml
<!-- BlazorHostPage.xaml -->
<ContentPage xmlns="http://schemas.microsoft.com/dotnet/2021/maui"
             xmlns:b="clr-namespace:Microsoft.AspNetCore.Components.WebView.Maui">
    
    <b:BlazorWebView HostPage="wwwroot/index.html">
        <b:BlazorWebView.RootComponents>
            <b:RootComponent Selector="#app" 
                           ComponentType="{x:Type local:Routes}" />
        </b:BlazorWebView.RootComponents>
    </b:BlazorWebView>
</ContentPage>
```

---

## إعدادات التطبيق

### AppConfig.cs

```csharp
public static class AppConfig
{
#if DEBUG
    public static string ApiBaseUrl = "http://localhost:5235";
#else
    public static string ApiBaseUrl = "https://api.rubikcare.com";
#endif

    public static int RequestTimeout = 30; // ثواني
    public static int MaxRetryAttempts = 3;
}
```

---

## النشر (Publishing)

### Android (APK/AAB)

```bash
# إنشاء APK
dotnet publish -f net10.0-android -c Release /p:AndroidPackageFormat=apk

# إنشاء AAB (Google Play)
dotnet publish -f net10.0-android -c Release /p:AndroidPackageFormat=aab
```

### iOS

```bash
# إنشاء IPA
dotnet publish -f net10.0-ios -c Release
```

---

## CHECKLIST: عند إضافة صفحة جديدة في MAUI

- [ ] هل أنشأت مجلداً في `Features/{Domain}/Views/`؟
- [ ] هل أنشأت ViewModel مطابقاً في `Features/{Domain}/ViewModels/`؟
- [ ] هل ورثت من `BaseViewModel`؟
- [ ] هل استخدمت `SafeApiCallAsync` لاستدعاءات API؟
- [ ] هل تعاملت مع حالة عدم الاتصال بالإنترنت؟
- [ ] هل اختبرت على Android و iOS؟
- [ ] هل تعاملت مع UI Thread بشكل صحيح؟

### عند التعامل مع الجلسات والكاش (جديد)
- [ ] هل `AppShellViewModel.LoadUserDataAsync()` تُستدعى من `DashboardPage.OnAppearing`؟
- [ ] هل `LogoutAsync` يمسح `_cachedSession` + `SecureStorage` + يستدعي `api/auth/logout`؟
- [ ] هل `HasClinic`/`HasPharmacy` تستخدمان `Memberships` وليس `Clinics`/`Pharmacies`؟

---

## 🔗 روابط ذات صلة

- [00 - الهيكل المعماري](00-architecture-overview.md)
- [07 - نظام PSP](07-psp-system.md)
- [09 - دليل API](09-api-guide.md)
- [11 - دليل BlazorWebView](11-blazor-webview-guide.md)
- [14 - نظام الكاش الموحد](14-caching-system.md) ⭐ جديد
```

---حد

**أرسل الملف التالي:** `13-clean-architecture-enforcement.md` 🎯
