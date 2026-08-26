# 📘 وثيقة مشروع RubikCare.PWA

**آخر تحديث:** 26 أغسطس 2026
**الحالة:** مرحلة التطوير النشط (Alpha) — تشغيل مكونات Shared.UI
**الإصدار:** 0.2.0

---

## 🎯 1. المقدمة والهدف

### ما هو مشروع RubikCare.PWA؟

**RubikCare.PWA** هو تطبيق ويب تقدمي (Progressive Web App) مبني بتقنية **Blazor WebAssembly** في إطار **.NET 10**. يمثل **العميل الثالث** في منظومة RubikCare:

| المشروع | التقنية | الجمهور | الحالة |
|---------|---------|---------|--------|
| **RubikCare.Web** | Blazor Server | الإدارة والموظفون | ✅ منشور |
| **RubikCare.Mobile** | .NET MAUI + BlazorWebView | المرضى والمهنيون | ✅ منشور على Google Play |
| **RubikCare.PWA** ⭐ | Blazor WebAssembly | المرضى والمهنيون (ويب متقدم) | 🚧 قيد التطوير |

### الأهداف الاستراتيجية

1. **تجربة موحدة:** نفس تجربة الموبايل (تصميم بطاقات على تدرج لوني) وليس نمط الويب (Split-Panel)
2. **إعادة الاستخدام القصوى:** تشغيل مكونات `Shared.UI` الموجودة بالفعل دون إعادة بنائها
3. **قابلية التثبيت:** تثبيت التطبيق على الشاشة الرئيسية (iOS/Android/Desktop)
4. **مصدر حقيقة واحد:** بيانات الجلسة تُجلب مرة واحدة وتُشارك عبر جميع الصفحات

---

## 🏗️ 2. البنية المعمارية والربط مع الحل

### الموقع في Clean Architecture

```
┌─────────────────────────────────────────────────────────┐
│                    RubikCare.Api.Web                     │
│              (نقطة الدخول الوحيدة للبيانات)              │
└─────────────────────┬───────────────────────────────────┘
                      │ HTTP/REST + JWT
        ┌─────────────┼─────────────┐
        ▼             ▼             ▼
   ┌────────┐   ┌──────────┐   ┌──────────┐
   │  Web   │   │  Mobile  │   │  PWA ⭐  │
   │ Server │   │  (MAUI)  │   │  (WASM)  │
   └────────┘   └──────────┘   └──────────┘
        │             │             │
        └─────────────┼─────────────┘
                      ▼
              ┌───────────────┐
              │  Shared.UI    │
              │    (RCL)      │
              └───────────────┘
```

### التبعيات المسموحة

```
RubikCare.PWA
├── ✅ يعتمد على: RubikCare.Shared.UI (مكونات + خدمات)
├── ✅ يتواصل مع: Api.Web عبر HTTP فقط
├── ❌ لا يعتمد على: Infrastructure
├── ❌ لا يعتمد على: Domain مباشرة
└── ❌ لا يعتمد على: DbContext
```

### ملفات الربط الحرجة (الحالية)

| الملف | الدور | الحالة |
|-------|-------|--------|
| `PWA/Program.cs` | تسجيل الخدمات (`IApiService`, `WebApiService`, `WebTranslationService`, `UserSessionState`) | ✅ مكتمل |
| `PWA/Layout/MainLayout.razor` | الهيكل الرئيسي + تحميل الجلسة مرة واحدة | ✅ مكتمل |
| `PWA/Layout/NavMenu.razor` | القائمة الجانبية الديناميكية بالترجمات | ✅ مكتمل |
| `PWA/Services/UserSessionState.cs` | حالة الجلسة المشتركة (تستدعي `api/user/session-bootstrap`) | ✅ مكتمل |
| `PWA/Services/WebApiService.cs` | تنفيذ `IApiService` عبر HTTP مع التوكن | ✅ مكتمل |
| `PWA/Services/WebTranslationService.cs` | تنفيذ `ISharedTranslationService` عبر API | ✅ مكتمل |

---

## 📦 3. البنية الحقيقية لـ Shared.UI (من الاستكشاف الفعلي)

> ⚠️ **ملاحظة:** هذا القسم مبني على استكشاف فعلي للمشروع بتاريخ 26 أغسطس 2026، وليس على الوثائق القديمة.

### 3.1 قائمة المكونات الكاملة (66 مكون)

#### 🏥 فئة العيادة (Clinic) — 4 مكونات
| المكون | النوع | الوظيفة |
|--------|------|---------|
| `ClinicDashboard.razor` | لوحة تحكم كاملة | لوحة تحكم الطبيب |
| `ClinicPatientDetails.razor` | صفحة تفاصيل | تفاصيل مريض |
| `ClinicPatientsList.razor` | قائمة | قائمة المرضى |
| `MyPatientInvitations.razor` | صفحة | دعوات المرضى |

#### ⚖️ الفئة القانونية (Legal) — 2 مكونات ✅ لها @page
| المكون | المسار | الوظيفة |
|--------|--------|---------|
| `PrivacyPolicy.razor` | `/privacy`, `/legal/privacy` | سياسة الخصوصية |
| `TermsOfService.razor` | `/terms`, `/legal/terms` | شروط الاستخدام |

#### 💬 فئة المحادثات (Messaging) — 7 مكونات
| المكون | الوظيفة |
|--------|---------|
| `MessagingHubPage.razor` | مركز المحادثات |
| `ChatPage.razor` | صفحة الدردشة |
| `ChatMessage.razor` | مكون رسالة |
| `ConversationsListPage.razor` | قائمة المحادثات |
| `DoctorSearchPage.razor` | بحث عن طبيب |
| `DoctorProfilePage.razor` | ملف طبيب |
| `MessagingSettingsPage.razor` | إعدادات المحادثات |

#### 🏢 إدارة المنظمات (OrganizationManagement) — 4 مكونات ✅ لها @page
| المكون | المسار | الوظيفة |
|--------|--------|---------|
| `CreateOrganizationPage.razor` | `/create-organization` | إنشاء منظمة |
| `MembersTab.razor` | `/organization/members` | تبويب الأعضاء |
| `MemberTitlesTab.razor` | `/organization/member-titles` | ألقاب الأعضاء |
| `CustomJobTitles.razor` | `/organization/job-titles` | المسميات الوظيفية |

#### 👤 فئة المريض (Patient) — 11 مكون
| المكون | الوظيفة |
|--------|---------|
| `SettingsPage.razor` | الإعدادات الرئيسية |
| `MyProfilePage.razor` | ملفي الشخصي |
| `EditProfilePage.razor` | تعديل الملف |
| `PatientOrdersPage.razor` | طلبات المريض |
| `OrderTracker.razor` | تتبع الطلب |
| `NotificationsPage.razor` | الإشعارات |

##### ⏰ جدولة الأدوية (Patient/SchedualMedication) — 5 مكونات
| المكون | الوظيفة |
|--------|---------|
| `MedicationSchedulePage.razor` | جدول الأدوية الرئيسي |
| `AddMedicationPage.razor` | إضافة دواء |
| `PspSchedulePage.razor` | جدولة PSP |
| `DailyDosesTab.razor` | تبويب الجرعات اليومية |
| `RefillDatesTab.razor` | تبويب مواعيد الصرف |

##### ⚙️ إعدادات المريض (Patient/Settings) — 6 مكونات (2 منها @page)
| المكون | المسار | الوظيفة |
|--------|--------|---------|
| `ProfessionalStatusPage.razor` | `/professional-status` ✅ | الحالة المهنية |
| `ProfessionalLicensePage.razor` | `/professional-license` ✅ | الترخيص المهني |
| `SecuritySection.razor` | — | قسم الأمان |
| `LanguageSection.razor` | — | قسم اللغة |
| `NotificationsSection.razor` | — | قسم الإشعارات |
| `PrivacySection.razor` | — | قسم الخصوصية |

#### 💊 فئة الصيدلية (Pharmacy) — 13 مكون (1 له @page)
| المكون | النوع | الوظيفة |
|--------|------|---------|
| `PharmacyDashboard.razor` | لوحة تحكم | لوحة الصيدلية |
| `PharmacyDetailPage.razor` | صفحة @page ✅ | تفاصيل صيدلية |
| `PharmacySearchPage.razor` | بحث | بحث صيدليات |
| `NearbyPharmacies.razor` | قائمة | الصيدليات القريبة |
| `PharmacyGateway.razor` | بوابة | بوابة الصيدلية |
| `PharmacyReportsPage.razor` | تقارير | تقارير الصيدلية |
| `PatientOrders.razor` | قائمة | طلبات المرضى |
| `MedicationRequestCard.razor` | بطاقة | بطاقة طلب دواء |
| `TokenRedemptionCard.razor` | بطاقة | بطاقة استرداد رمز |
| `PharmacyCard.razor` | بطاقة | بطاقة صيدلية |
| `PharmacyGrid.razor` | شبكة | شبكة صيدليات |
| `PharmacyFilter.razor` | فلتر | فلتر البحث |

#### 🌿 فئة برامج الدعم (PSP) — 5 مكونات (1 له @page)
| المكون | النوع | الوظيفة |
|--------|------|---------|
| `PspAboutPage.razor` | صفحة @page ✅ | عن برامج الدعم |
| `PspGateway.razor` | بوابة | بوابة PSP |
| `PspSearch.razor` | بحث | بحث برامج |
| `PspEntry.razor` (Patient) | إدخال | دخول برنامج |
| `PspScheduleSetup.razor` (Patient) | إعداد | جدولة برنامج |

#### 👔 فئة المندوب (Rep) — 5 مكونات
| المكون | الوظيفة |
|--------|---------|
| `RepDashboard.razor` | لوحة المندوب |
| `PharmaCompanyDashboard.razor` | لوحة شركة الأدوية |
| `InviteDoctor.razor` | دعوة طبيب |
| `MyInvitations.razor` | دعواتي |
| `MyNetwork.razor` | شبكتي |

#### 🛠️ مكونات مساعدة مشتركة — 7 مكونات
| المكون | الوظيفة | الحالة في PWA |
|--------|---------|---------------|
| `LoaderOverlay.razor` | شاشة تحميل | ✅ مستخدم بالفعل |
| `RubikButton.razor` | زر موحد | جاهز |
| `TestPage.razor` | صفحة اختبار | جاهز |
| `Pagination.razor` | ترقيم الصفحات | جاهز |
| `SearchBar.razor` | شريط بحث | جاهز |
| `RubikSmartTable.razor` | جدول ذكي | جاهز |
| `SupportPage.razor` | صفحة دعم @page ✅ | جاهز |

### 3.2 خدمات Shared.UI (20 ملف)

| الخدمة | النوع | الاستخدام في PWA |
|--------|------|------------------|
| `IApiService.cs` | واجهة | ✅ مسجلة كـ `WebApiService` |
| `ITranslationService.cs` | واجهة | ✅ مسجلة كـ `WebTranslationService` |
| `ITranslationState.cs` | واجهة | ✅ مسجلة كـ `SharedTranslationState` |
| `CurrentOrganizationState.cs` | حالة | ✅ مسجلة |
| `IAppStateService.cs` | واجهة | ⚠️ تحتاج تنفيذ في PWA |
| `IPatientSessionService.cs` | واجهة | ⚠️ تحتاج تنفيذ في PWA |
| `PatientSessionService.cs` | تنفيذ | ⚠️ قد تحتاج بديل |
| `SharedApiService.cs` | تنفيذ | بديل محتمل |
| `ILocalNotificationService.cs` | واجهة | ✅ مسجلة كـ `WebLocalNotificationService` |
| `IMobileNavigationService.cs` | واجهة | ❌ خاص بـ MAUI — يحتاج بديل |
| `INearbyPharmacyService.cs` | واجهة | ⚠️ يحتاج تنفيذ (GPS) |
| `INotificationNavigationService.cs` | واجهة | ⚠️ يحتاج تنفيذ |
| `IPspUiService.cs` | واجهة | ⚠️ يحتاج تنفيذ |
| `IPublicApiService.cs` | واجهة | ⚠️ يحتاج تنفيذ |
| `InviteNavigationBridge.cs` | جسر | ⚠️ خاص بـ MAUI |
| `RepNavigationBridge.cs` | جسر | ⚠️ خاص بـ MAUI |
| `OsmPharmacyService.cs` | تنفيذ | ⚠️ يحتاج تقييم |
| `PendingRequestService.cs` | خدمة | ⚠️ يحتاج تقييم |
| `Models/NotificationItem.cs` | نموذج | ✅ جاهز |
| `Models/PspModels.cs` | نموذج | ✅ جاهز |

### 3.3 الصفحات القابلة للتوجيه مباشرة (@page)

هذه الصفحات يمكن الوصول إليها مباشرة عبر الروابط دون الحاجة لـ Wrapper:

| الصفحة | المسار(ات) |
|--------|-----------|
| `TermsOfService` | `/terms`, `/legal/terms` |
| `PrivacyPolicy` | `/privacy`, `/legal/privacy` |
| `SupportPage` | `/support` |
| `CreateOrganizationPage` | `/create-organization` |
| `MembersTab` | `/organization/members` |
| `MemberTitlesTab` | `/organization/member-titles` |
| `CustomJobTitles` | `/organization/job-titles` |
| `ProfessionalStatusPage` | `/professional-status` |
| `ProfessionalLicensePage` | `/professional-license` |
| `PharmacyDetailPage` | `/pharmacy/{id}` |
| `PspAboutPage` | `/psp/about` |

---

## ⚠️ 4. التناقضات المعمارية المكتشفة والحلول

### التناقض 1: "لا صفحات كاملة في Shared.UI"

**ما تقوله الوثيقة `00-architecture-overview.md`:**
> ❌ ما لا يوضع في Shared.UI: صفحات كاملة (Dashboard, PSP, Admin)

**الواقع الفعلي:**
تحتوي Shared.UI على لوحات تحكم كاملة (`ClinicDashboard`, `PharmacyDashboard`, `RepDashboard`) وصفحات متعددة.

**التفسير:**
هذه الصفحات وُضعت في Shared.UI لأنها **مشتركة بين الويب والموبايل** (تُستضاف في الموبايل عبر `BlazorWebView`). هذا يتوافق مع **المستوى 4** من شجرة القرار المعماري (عمليات مشتركة بين المنصات).

**القرار العملي:**
> ✅ نقبل الواقع الحالي. هذه الصفحات موجودة وتعمل في الموبايل والويب. مهمتنا في PWA هي **استضافتها** وليس إعادة بنائها.

### التناقض 2: نمط التصميم (MAUI مقابل Web)

**ما تقوله الوثيقة `03-style-guide.md`:**
صفحات المصادقة تستخدم نمط **Split-Panel** (لوحة بصرية + لوحة نموذج).

**ما اتفقنا عليه في هذه الجلسة:**
صفحات المصادقة في PWA تتبع نمط **الموبايل** (بطاقة بيضاء على تدرج لوني) وليس نمط الويب.

**القرار العملي:**
> ✅ صفحات المصادقة (Login, Register) في PWA تتبع نمط الموبايل.
> ✅ الصفحات الداخلية تتبع نمط الدور (Clinic/Pharmacy/Rep) حسب وثيقة `03`.

### التناقض 3: حقن الخدمات في مكونات Shared.UI

**ما تقوله الوثيقة `11-blazor-webview-guide.md`:**
مكونات Shared.UI تستقبل `IApiService` كـ `[Parameter]` وليس `@inject` (بسبب عزل حاوية DI في BlazorWebView).

**الأثر على PWA:**
عند استضافة هذه المكونات في PWA، يجب تمرير `IApiService` كـ Parameter:

```razor
<SettingsPage ApiService="ApiService" />
```

حيث `ApiService` هو `IApiService` المحقون في صفحة PWA المضيفة.

---

## 🗺️ 5. خطة تشغيل مكونات Shared.UI في PWA

### المرحلة 1: الصفحات المستقلة ذات @page (منخفضة المخاطر) ⭐ الحالية

**الهدف:** إثبات أن البنية تعمل عبر تشغيل أبسط الصفحات.

| الصفحة | المسار في NavMenu | الأولوية |
|--------|-------------------|----------|
| `SupportPage` | `/support` | 🔴 عالية |
| `TermsOfService` | `/terms` | 🔴 عالية |
| `PrivacyPolicy` | `/privacy` | 🔴 عالية |
| `ProfessionalStatusPage` | `/professional-status` | 🟡 متوسطة |
| `ProfessionalLicensePage` | `/professional-license` | 🟡 متوسطة |

### المرحلة 2: صفحات المريض الشخصية

| الصفحة | المسار | المتطلبات |
|--------|--------|-----------|
| `MyProfilePage` | `/profile` | `UserSessionState`, `IApiService` |
| `EditProfilePage` | `/profile/edit` | رفع صور (قد يحتاج `localStorage`) |
| `SettingsPage` | `/settings` | أقسام متعددة (لغة، إشعارات، أمان، خصوصية) |
| `NotificationsPage` | `/notifications` | `INotificationNavigationService` |

### المرحلة 3: لوحات التحكم حسب الدور (الأعلى قيمة)

| اللوحة | الدور | المتطلبات |
|--------|------|-----------|
| `ClinicDashboard` | 👨‍⚕️ طبيب | `CurrentOrganizationState`, `IApiService` |
| `PharmacyDashboard` | 💊 صيدلي | `CurrentOrganizationState`, `IApiService` |
| `PharmaCompanyDashboard` | 👔 مندوب | `CurrentOrganizationState`, `IApiService` |

### المرحلة 4: الميزات المتخصصة

| الفئة | المكونات | التعقيد |
|------|---------|---------|
| جدولة الأدوية | `MedicationSchedulePage`, `AddMedicationPage`, `PspSchedulePage` | 🔴 عالية |
| المحادثات | `MessagingHubPage`, `ChatPage`, `DoctorSearchPage` | 🔴 عالية (قد تحتاج SignalR) |
| PSP | `PspGateway`, `PspSearch`, `PspEntry` | 🟠 متوسطة-عالية |
| الصيدليات القريبة | `NearbyPharmacies`, `PharmacySearchPage` | 🟠 متوسطة (تحتاج GPS) |
| إدارة المنظمة | `MembersTab`, `MemberTitlesTab`, `CustomJobTitles` | 🟡 متوسطة |

### المرحلة 5: الميزات المتقدمة

| الميزة | المتطلبات |
|--------|-----------|
| تتبّع الطلبات (`OrderTracker`) | ربط مع نظام الطلبات |
| دعوات المرضى (`MyPatientInvitations`) | ربط مع نظام الدعوات |
| شبكة المندوب (`MyNetwork`) | ربط مع بيانات المندوب |

---

## 🔑 6. الأنماط الناجحة والموثقة

### 6.1 استضافة مكون Shared.UI في PWA

```razor
@* صفحة PWA المضيفة *@
@page "/settings"
@inject IApiService ApiService

<SettingsPage ApiService="ApiService" />
```

### 6.2 تمرير بيانات الجلسة

```razor
@* استخدام UserSessionState في أي صفحة *@
@inject UserSessionState SessionState

<h1>@SessionState.CurrentSession?.FullNameAr</h1>
```

### 6.3 الترجمة الآمنة (مع مفتاح احتياطي)

```csharp
private string T(string key)
{
    if (_translations.TryGetValue(key, out var value))
        return value;

    // مفتاح مختصر (بدون البادئة) كاحتياطي
    var shortKey = key.Contains('.') ? key[(key.LastIndexOf('.') + 1)..] : key;
    if (_translations.TryGetValue(shortKey, out var shortValue))
        return shortValue;

    return key;
}
```

### 6.4 إعادة الرسم عند تغيير البيانات

```csharp
protected override async Task OnInitializedAsync()
{
    SessionState.OnChange += OnSessionChanged;
}

private async void OnSessionChanged() => await InvokeAsync(StateHasChanged);

public void Dispose() => SessionState.OnChange -= OnSessionChanged;
```

---

## 🚫 7. الأنماط المحظورة

| النمط | السبب | البديل |
|-------|-------|--------|
| اختراع مفاتيح ترجمة | المفاتيح يجب أن تكون موجودة في قاعدة البيانات | تحقق بـ `SELECT` قبل الاستخدام |
| استخدام `@inject HttpClient` في مكونات Shared.UI | يفشل بصمت في BlazorWebView | استخدم `[Parameter] IApiService` |
| تعديل ملفات Migration يدوياً | يكسر تتبع EF Core | استخدم `Add-Migration` فقط |
| استخدام ألوان ثابتة بدلاً من المتغيرات | صعب الصيانة | استخدم `var(--rubik-primary)` |
| Inline styles في ملفات كبيرة | غير قابل للصيانة | استخدم ملفات `.razor.css` |
| `@keyframes` بدون هروب | يفسرها Razor كـ C# | استخدم `@@keyframes` |
| `?.` مع `EventCallback` | هو struct وليس nullable | استخدم `.InvokeAsync()` |

---

## 📋 8. الدروس المستفادة من هذه الجلسة

### 8.1 لا تفترض، بل تحقق
- **الخطأ:** افتراض أن مفاتيح الترجمة غير موجودة
- **الصحيح:** فتح Console وقراءة رسالة الخطأ الفعلية

### 8.2 استخدم المفاتيح الموجودة فعلاً
- **الخطأ:** اختراع مفاتيح مثل `DASHBOARD.WELCOME`
- **الصحيح:** استخدام المفاتيح الفعلية `SHARED.DASHBOARD.WELCOME` من قاعدة البيانات

### 8.3 افهم دورة حياة المكونات
- **الخطأ:** توقع أن المكون الفرعي يُعاد رسمه تلقائياً
- **الصحيح:** الاشتراك في أحداث التغيير (`OnChange`) واستدعاء `StateHasChanged`

### 8.4 ابدأ بالبسيط ثم تعقّد
- **الخطأ:** بناء لوحة تحكم كاملة من البداية
- **الصحيح:** صفحة مؤقتة → بيانات حقيقية → ميزات متقدمة

### 8.5 وثّق قبل أن تنسى
- كل اكتشاف جديد يجب أن يُضاف لهذه الوثيقة فوراً

---

## 🎯 9. الحالة الحالية والخطوات التالية

### ✅ ما تم إنجازه (0.2.0)

- [x] البنية التحتية للـ PWA (Program.cs, Services)
- [x] صفحات المصادقة (Login, Register) بنمط الموبايل
- [x] الهيكل الرئيسي (MainLayout, NavMenu, EmptyLayout)
- [x] نظام الترجمة الكامل (عربي/إنجليزي)
- [x] نظام الجلسة الموحدة (UserSessionState)
- [x] لوحة تحكم أساسية (Dashboard) ببيانات حقيقية
- [x] القائمة الجانبية الديناميكية بالبيانات الحقيقية

### 🔄 الخطوة التالية

**المرحلة 1:** تشغيل الصفحات القانونية والدعم (Support, Terms, Privacy)

### 📊 إحصائيات المشروع

| المقياس | القيمة |
|---------|--------|
| مكونات Shared.UI المتاحة | 66 |
| صفحات @page جاهزة للتوجيه | 11 |
| خدمات تحتاج تنفيذ في PWA | ~8 |
| نسبة الإنجاز الحالية | ~20% |

---

**ملاحظة:** هذه وثيقة حية (Living Document) — يتم تحديثها مع تقدم المشروع.

**آخر مراجعة:** 26 أغسطس 2026
**المراجعة التالية:** بعد إكمال المرحلة 1 (الصفحات القانونية والدعم)

---

هل هذه الوثيقة المحدثة تلبي احتياجاتك؟ إذا كانت الإجابة نعم، يمكننا الانتقال فوراً لتنفيذ **المرحلة 1: تشغيل الصفحات القانونية والدعم**. 🚀
