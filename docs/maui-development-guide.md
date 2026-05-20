# 10 - دليل تطوير MAUI

**آخر تحديث: 17 مايو 2026**

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
│   │   │   ├── LoginPage.xaml
│   │   │   ├── RegisterPage.xaml
│   │   │   └── ProRolePage.xaml
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
│   └── Profile/
│       ├── Views/
│       │   └── ProfilePage.xaml
│       └── ViewModels/
│
├── Services/                           # خدمات التطبيق
│   ├── ApiService.cs                   # التواصل مع Api.Web
│   ├── AuthService.cs                  # المصادقة وإدارة التوكن
│   ├── NavigationService.cs            # التنقل بين الصفحات
│   └── AppStateService.cs              # حالة التطبيق المشتركة
│
├── Models/                             # نماذج محلية
│   ├── User.cs
│   ├── PspProgram.cs
│   └── ApiResponse.cs
│
├── Helpers/                            # دوال مساعدة
│   ├── SecureStorageHelper.cs
│   └── ConnectivityHelper.cs
│
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

## التنقل بين الصفحات

### AppShell - التنقل الرئيسي

```xml
<!-- AppShell.xaml -->
<Shell x:Class="RubikCare.Mobile.AppShell">
    <!-- صفحات عامة (بدون مصادقة) -->
    <ShellContent Route="login" ContentTemplate="{DataTemplate views:LoginPage}" />
    <ShellContent Route="register" ContentTemplate="{DataTemplate views:RegisterPage}" />
    
    <!-- صفحات محمية (بعد المصادقة) -->
    <TabBar Route="main">
        <ShellContent Route="psp" Title="برامج الدعم" 
                      ContentTemplate="{DataTemplate psp:PspSearchPage}" />
        <ShellContent Route="profile" Title="الملف الشخصي" 
                      ContentTemplate="{DataTemplate profile:ProfilePage}" />
    </TabBar>
</Shell>
```

### التنقل البرمجي

```csharp
// الانتقال لصفحة مع تمرير معاملات
await Shell.Current.GoToAsync("programDetails", new Dictionary<string, object>
{
    { "ProgramId", selectedProgram.ProgramId }
});

// استقبال المعاملات في الصفحة المستقبلة
[QueryProperty(nameof(ProgramId), "ProgramId")]
public partial class ProgramDetailsPage : ContentPage
{
    private int _programId;
    public int ProgramId
    {
        get => _programId;
        set
        {
            _programId = value;
            LoadProgramAsync();
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

---

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
    public static string ApiBaseUrl = "https://localhost:5001";
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

---

## 🔗 روابط ذات صلة

- [00 - الهيكل المعماري](00-architecture-overview.md)
- [07 - نظام PSP](07-psp-system.md)
- [09 - دليل API](09-api-guide.md)
- [11 - دليل BlazorWebView](11-blazor-webview-guide.md)
```
