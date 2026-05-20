# 11 - دليل BlazorWebView والأنماط الناجحة

**آخر تحديث: 17 مايو 2026**

---

## مقدمة

هذا المرجع يوثق **الأنماط الناجحة** وما يجب تجنبه عند استخدام BlazorWebView داخل تطبيق MAUI. الهدف هو مشاركة المكونات بين Web و Mobile مع تجنب المشاكل المعروفة.

---

## لماذا BlazorWebView؟

| XAML التقليدي | BlazorWebView |
|---------------|---------------|
| لغة مختلفة عن Web | نفس HTML/CSS المستخدم في Web |
| مشاكل Binding متكررة | لا Binding - استخدام Razor |
| صعوبة مشاركة المكونات | مشاركة مباشرة عبر RCL |
| منحنى تعلم منفصل | نفس مهارات Blazor |

---

## هيكل المشروع مع BlazorWebView

```
RubikCare.Mobile/
├── Features/
│   ├── BlazorPages/                     # صفحات Blazor الجديدة
│   │   ├── wwwroot/
│   │   │   ├── index.html               # صفحة مضيف Blazor
│   │   │   └── css/
│   │   │       └── app.css
│   │   ├── _Imports.razor               # استيرادات مشتركة
│   │   ├── Routes.razor                 # توجيه Blazor الداخلي
│   │   └── Pages/
│   │       ├── PspDashboard.razor
│   │       ├── PatientList.razor
│   │       └── Reports.razor
│   │
│   └── XamlPages/                       # صفحات XAML الموجودة
│       └── ... (تبقى كما هي)
│
└── BlazorHostPage.xaml                  # صفحة XAML تستضيف BlazorWebView
```

---

## الأنماط الناجحة (يجب اتباعها)

### 1. استخدام RCL للمكونات المشتركة

```razor
@* في RubikCare.Shared.UI - مكون مشترك *@
@* Components/RubikButton.razor *@

<button class="btn @ColorClass" @onclick="OnClick">
    @if (!string.IsNullOrEmpty(Icon))
    {
        <i class="@Icon me-1"></i>
    }
    @Text
</button>

@code {
    [Parameter] public string Text { get; set; }
    [Parameter] public string Icon { get; set; }
    [Parameter] public string ColorClass { get; set; } = "btn-primary";
    [Parameter] public EventCallback OnClick { get; set; }
}
```

**الاستخدام في Web و Mobile:**
```razor
@* نفس المكون يعمل في المنصتين *@
<RubikButton Text="حفظ" Icon="bi-save" ColorClass="btn-success" OnClick="SaveData" />
```

### 2. حقن الخدمات عبر DI

```csharp
// في MauiProgram.cs
builder.Services.AddScoped<IApiService, ApiService>();
builder.Services.AddScoped<IAuthService, AuthService>();

// في مكون Blazor
@inject IApiService ApiService
@inject IAuthService AuthService
```

### 3. استخدام HttpClient موحد

```csharp
// في MauiProgram.cs
builder.Services.AddScoped(sp =>
{
    var handler = new HttpClientHandler();
    
    // ⭐ تجاهل أخطاء SSL في التطوير فقط
#if DEBUG
    handler.ServerCertificateCustomValidationCallback = (message, cert, chain, errors) => true;
#endif
    
    return new HttpClient(handler)
    {
        BaseAddress = new Uri(AppConfig.ApiBaseUrl)
    };
});
```

### 4. التعامل مع الترجمة

```csharp
// ⭐ استخدام نفس نمط T() من الويب
private Dictionary<string, string> _translations = new();
private string T(string key) =>
    _translations.TryGetValue(key, out var value) ? value : key;

protected override async Task OnInitializedAsync()
{
    var lang = Preferences.Get("language", "ar");
    var translations = await ApiService.GetAsync<Dictionary<string, string>>(
        $"/api/localization/page/MOBILE?lang={lang}");
    var common = await ApiService.GetAsync<Dictionary<string, string>>(
        $"/api/localization/page/COMMON?lang={lang}");
    
    _translations = common.Concat(translations)
        .ToDictionary(kvp => kvp.Key, kvp => kvp.Value);
}
```

---

## ما يجب تجنبه (Anti-Patterns)

### ❌ 1. استدعاء Native APIs مباشرة من Blazor

```csharp
// ❌ خطأ - سيسبب استثناء
await SecureStorage.SetAsync("key", "value");

// ✅ صحيح - استخدم خدمة وسيطة
await _storageService.SetAsync("key", "value");
```

### ❌ 2. استخدام JavaScript Interop بكثرة

```javascript
// ❌ تجنب استدعاء JavaScript المتكرر
// كل استدعاء JSInterop له تكلفة أداء
```

```csharp
// ✅ بدلاً من ذلك، استخدم C# مباشرة
// معظم وظائف MAUI متاحة في C#
```

### ❌ 3. تخزين البيانات الحساسة في localStorage

```javascript
// ❌ خطأ - localStorage غير آمن
localStorage.setItem("token", "eyJhbGci...");
```

```csharp
// ✅ صحيح - استخدم SecureStorage عبر خدمة
await _storageService.SecureSetAsync("token", token);
```

### ❌ 4. Hardcoding المسارات

```csharp
// ❌ خطأ
Navigation.NavigateTo("/psp/patient/5/details");

// ✅ صحيح - استخدم Constants
Navigation.NavigateTo($"{Routes.PspPatientDetails}/{patientId}");
```

---

## التكامل مع XAML

### فتح صفحة XAML من Blazor

```csharp
// في مكون Blazor
@inject NavigationManager Navigation

private void OpenCamera()
{
    // الانتقال لصفحة XAML للمسح
    Navigation.NavigateTo("/xaml/scan-token");
}
```

### فتح صفحة Blazor من XAML

```csharp
// في XAML ViewModel
await Shell.Current.GoToAsync("blazorHost", new Dictionary<string, object>
{
    { "Route", "/psp/dashboard" }
});
```

---

## إعدادات BlazorWebView

### في XAML

```xml
<!-- BlazorHostPage.xaml -->
<ContentPage>
    <BlazorWebView HostPage="wwwroot/index.html">
        <BlazorWebView.RootComponents>
            <RootComponent Selector="#app" 
                         ComponentType="{x:Type blazor:Routes}" />
        </BlazorWebView.RootComponents>
    </BlazorWebView>
</ContentPage>
```

### ملف index.html

```html
<!DOCTYPE html>
<html lang="ar" dir="rtl">
<head>
    <meta charset="UTF-8">
    <meta name="viewport" content="width=device-width, initial-scale=1.0">
    <link rel="stylesheet" href="css/app.css">
</head>
<body>
    <div id="app">جاري التحميل...</div>
    <script src="_framework/blazor.webview.js"></script>
</body>
</html>
```

---

## تحسين الأداء

### 1. تجنب StateHasChanged المتكرر

```csharp
// ❌ تجنب
await Task.Delay(100);
StateHasChanged();

// ✅ استخدم InvokeAsync عند الحاجة فقط
await InvokeAsync(StateHasChanged);
```

### 2. استخدام ShouldRender

```csharp
protected override bool ShouldRender()
{
    // ⭐ لا تعيد رسم إذا لم تتغير البيانات
    return _hasChanges;
}
```

### 3. تحميل البيانات عند الحاجة

```csharp
// ❌ تحميل كل البيانات دفعة واحدة
protected override async Task OnInitializedAsync()
{
    Programs = await LoadAllPrograms();        // 1000+ برنامج
    Patients = await LoadAllPatients();        // 10000+ مريض
    Pharmacies = await LoadAllPharmacies();    // 500+ صيدلية
}

// ✅ تحميل البيانات المطلوبة فقط مع Pagination
private async Task LoadPage(int page)
{
    Programs = await LoadProgramsAsync(page, 20);  // 20 برنامج فقط
}
```

---

## CHECKLIST: عند إنشاء صفحة BlazorWebView جديدة

- [ ] هل المكون موجود في `RubikCare.Shared.UI` بدلاً من تكراره؟
- [ ] هل استخدمت DI لحقن الخدمات بدلاً من استدعاء Native APIs؟
- [ ] هل تعاملت مع الترجمة باستخدام `T()` المساعدة؟
- [ ] هل تجنبت JavaScript Interop غير الضروري؟
- [ ] هل اختبرت الصفحة على Android و iOS؟
- [ ] هل الأداء مقبول (تحميل < 2 ثانية)؟
- [ ] هل الصفحة متجاوبة مع أحجام الشاشات المختلفة؟

---

## 🔗 روابط ذات صلة

- [00 - الهيكل المعماري](00-architecture-overview.md)
- [05 - إنشاء الصفحات والمكونات](05-page-creation-checklist.md)
- [10 - دليل تطوير MAUI](10-maui-development-guide.md)
```
