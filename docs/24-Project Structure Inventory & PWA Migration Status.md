# 📄 24 - جرد هيكلة المشاريع وحالة نقل الـ PWA (Project Structure Inventory & PWA Migration Status)

**آخر تحديث:** 29 أغسطس 2026  
**الهدف:** مرجع دائم لتجنب إعادة الجرد والفحص في كل جلسة  
**الحالة:** ✅ محدّث بعد الجرد الشامل

---

## 📌 جدول المحتويات

1. [نظرة عامة على المشاريع الثلاثة](#1-نظرة-عامة-على-المشاريع-الثلاثة)
2. [جرد شامل: Shared.UI](#2-جرد-شامل-sharedui)
3. [جرد شامل: Mobile (MAUI)](#3-جرد-شامل-mobile-maui)
4. [حالة الـ PWA الحالي](#4-حالة-الـ-pwa-الحالي)
5. [جدول المقارنة الكامل](#5-جدول-المقارنة-الكامل)
6. [خريطة النقل وحالة الإنجاز](#6-خريطة-النقل-وحالة-الإنجاز)
7. [الخدمات: ما هو موجود وما ينقص](#7-الخدمات-ما-هو-موجود-وما-ينقص)

---

## 1. نظرة عامة على المشاريع الثلاثة

| المشروع | التقنية | الغرض | عدد المكونات/الصفحات |
|---------|---------|-------|----------------------|
| **Shared.UI** | Razor Class Library (RCL) | مكونات UI مشتركة بين جميع المنصات | 66 مكون Razor + 120+ ملف CSS + 23 خدمة |
| **Mobile** | .NET MAUI + BlazorWebView | تطبيق الموبايل (Android + iOS) | 57 صفحة XAML + 16 ViewModel + 20+ خدمة |
| **RubikCare.PWA** | Blazor WebAssembly | بديل تطبيق الويب للموبايل (هدف: دعم iPhone) | 25 صفحة Razor (حالة جزئية) |

### العلاقة بين المشاريع الثلاثة

```
┌─────────────────────────────────────────────────────────────────┐
│                    RubikCare.Shared.UI                          │
│              (مكتبة المكونات المشتركة - RCL)                    │
│  • 66 مكون Razor جاهز                                            │
│  • 120+ ملف CSS منظم حسب الأدوار                                │
│  • 23 خدمة/واجهة                                                │
└─────────────────────────────────────────────────────────────────┘
                                      │
        ┌─────────────────────────────┼─────────────────────────────┐
        │                             │                             │
        ▼                             ▼                             ▼
┌─────────────────┐          ┌─────────────────┐          ┌─────────────────┐
│   Mobile (MAUI)  │          │   Web (Blazor)   │          │      PWA         │
│  57 صفحة XAML    │          │   (Blazor Server)│          │  Blazor WASM     │
│  BlazorWebView   │          │                  │          │  (هدف: iPhone)   │
└─────────────────┘          └─────────────────┘          └─────────────────┘
```

---

## 2. جرد شامل: Shared.UI

### 2.1 المكونات حسب الفئة (66 مكون)

#### 🏥 فئة العيادة (Clinic) — 4 مكونات

| المكون | الوظيفة | حالة النقل للـ PWA |
|--------|---------|-------------------|
| `ClinicDashboard.razor` | لوحة تحكم الطبيب | ✅ منقول (صفحة موجودة) |
| `ClinicPatientDetails.razor` | تفاصيل مريض | ❌ غير منقول |
| `ClinicPatientsList.razor` | قائمة المرضى | ❌ غير منقول |
| `MyPatientInvitations.razor` | دعوات المرضى | ❌ غير منقول |

#### 💊 فئة الصيدلية (Pharmacy) — 12 مكون

| المكون | الوظيفة | حالة النقل للـ PWA |
|--------|---------|-------------------|
| `PharmacyDashboard.razor` | لوحة تحكم الصيدلية | ❌ غير منقول |
| `PharmacyDetailPage.razor` | تفاصيل صيدلية | ❌ غير منقول |
| `PharmacySearchPage.razor` | بحث صيدليات | ❌ غير منقول |
| `NearbyPharmacies.razor` | الصيدليات القريبة | ⚠️ موجود في PublicUser |
| `PharmacyGateway.razor` | بوابة الصيدلية | ❌ غير منقول |
| `PharmacyReportsPage.razor` | تقارير الصيدلية | ❌ غير منقول |
| `PatientOrders.razor` | طلبات المرضى | ⚠️ موجود في PublicUser |
| `MedicationRequestCard.razor` | بطاقة طلب دواء | ❌ غير منقول |
| `TokenRedemptionCard.razor` | بطاقة استرداد رمز | ❌ غير منقول |
| `PharmacyCard.razor` | بطاقة صيدلية | ❌ غير منقول |
| `PharmacyGrid.razor` | شبكة صيدليات | ❌ غير منقول |
| `PharmacyFilter.razor` | فلتر البحث | ❌ غير منقول |

#### 👔 فئة المندوب (Rep) — 5 مكونات

| المكون | الوظيفة | حالة النقل للـ PWA |
|--------|---------|-------------------|
| `RepDashboard.razor` | لوحة المندوب | ❌ غير منقول |
| `PharmaCompanyDashboard.razor` | لوحة شركة الأدوية | ❌ غير منقول |
| `InviteDoctor.razor` | دعوة طبيب | ❌ غير منقول |
| `MyInvitations.razor` | دعواتي | ❌ غير منقول |
| `MyNetwork.razor` | شبكتي | ❌ غير منقول |

#### 👤 فئة المريض (Patient) — 16 مكون

| المكون | الوظيفة | حالة النقل للـ PWA |
|--------|---------|-------------------|
| `SettingsPage.razor` | الإعدادات الرئيسية | ⚠️ موجود في PublicUser |
| `MyProfilePage.razor` | ملفي الشخصي | ⚠️ موجود في PublicUser |
| `EditProfilePage.razor` | تعديل الملف | ❌ غير منقول |
| `PatientOrdersPage.razor` | طلبات المريض | ⚠️ موجود في PublicUser |
| `OrderTracker.razor` | تتبع الطلب | ❌ غير منقول |
| `NotificationsPage.razor` | الإشعارات | ⚠️ موجود في PublicUser |
| `MedicationSchedulePage.razor` | جدول الأدوية | ⚠️ موجود في PublicUser |
| `AddMedicationPage.razor` | إضافة دواء | ❌ غير منقول |
| `PspSchedulePage.razor` | جدولة PSP | ❌ غير منقول |
| `DailyDosesTab.razor` | تبويب الجرعات اليومية | ❌ غير منقول |
| `RefillDatesTab.razor` | تبويب مواعيد الصرف | ❌ غير منقول |
| `ProfessionalStatusPage.razor` | الحالة المهنية | ❌ غير منقول |
| `ProfessionalLicensePage.razor` | الترخيص المهني | ❌ غير منقول |
| `SecuritySection.razor` | قسم الأمان | ❌ غير منقول |
| `LanguageSection.razor` | قسم اللغة | ❌ غير منقول |
| `NotificationsSection.razor` | قسم الإشعارات | ❌ غير منقول |
| `PrivacySection.razor` | قسم الخصوصية | ❌ غير منقول |

#### 💬 فئة المحادثات (Messaging) — 7 مكونات

| المكون | الوظيفة | حالة النقل للـ PWA |
|--------|---------|-------------------|
| `MessagingHubPage.razor` | مركز المحادثات | ❌ غير منقول |
| `ChatPage.razor` | صفحة الدردشة | ❌ غير منقول |
| `ChatMessage.razor` | مكون رسالة | ❌ غير منقول |
| `ConversationsListPage.razor` | قائمة المحادثات | ❌ غير منقول |
| `DoctorSearchPage.razor` | بحث عن طبيب | ⚠️ موجود في PublicUser |
| `DoctorProfilePage.razor` | ملف طبيب | ⚠️ موجود في PublicUser |
| `MessagingSettingsPage.razor` | إعدادات المحادثات | ⚠️ موجود في PublicUser |

#### 🌿 فئة برامج الدعم (PSP) — 6 مكونات

| المكون | الوظيفة | حالة النقل للـ PWA |
|--------|---------|-------------------|
| `PspAboutPage.razor` | عن برامج الدعم | ❌ غير منقول |
| `PspGateway.razor` | بوابة PSP | ✅ منقول |
| `PspSearch.razor` | بحث برامج | ❌ غير منقول |
| `PspEntry.razor` | دخول برنامج | ✅ منقول |
| `PspScheduleSetup.razor` | إعداد جدولة | ❌ غير منقول |
| `PspDetailComponent.razor` | تفاصيل برنامج | ✅ منقول |

#### 🏢 إدارة المنظمات (OrganizationManagement) — 4 مكونات

| المكون | الوظيفة | حالة النقل للـ PWA |
|--------|---------|-------------------|
| `CreateOrganizationPage.razor` | إنشاء منظمة | ❌ غير منقول |
| `MembersTab.razor` | تبويب الأعضاء | ❌ غير منقول |
| `MemberTitlesTab.razor` | ألقاب الأعضاء | ❌ غير منقول |
| `CustomJobTitles.razor` | المسميات الوظيفية | ❌ غير منقول |

#### ⚖️ الفئة القانونية (Legal) — 2 مكونات

| المكون | الوظيفة | حالة النقل للـ PWA |
|--------|---------|-------------------|
| `PrivacyPolicy.razor` | سياسة الخصوصية | ❌ غير منقول |
| `TermsOfService.razor` | شروط الاستخدام | ❌ غير منقول |

#### 🛠️ مكونات مساعدة مشتركة — 7 مكونات

| المكون | الوظيفة | حالة النقل للـ PWA |
|--------|---------|-------------------|
| `LoaderOverlay.razor` | شاشة تحميل | ✅ مستخدم بالفعل |
| `RubikButton.razor` | زر موحد | ❌ غير منقول |
| `TestPage.razor` | صفحة اختبار | ❌ غير منقول |
| `Pagination.razor` | ترقيم الصفحات | ❌ غير منقول |
| `SearchBar.razor` | شريط بحث | ❌ غير منقول |
| `RubikSmartTable.razor` | جدول ذكي | ❌ غير منقول |
| `SupportPage.razor` | صفحة دعم | ❌ غير منقول |

---

### 2.2 خدمات وواجهات Shared.UI (23 ملف)

| الخدمة/الواجهة | النوع | الحالة في الـ PWA |
|---------------|------|------------------|
| `IApiService.cs` | واجهة | ✅ مسجلة كـ `WebApiService` |
| `ITranslationService.cs` | واجهة | ✅ مسجلة كـ `WebTranslationService` |
| `ITranslationState.cs` | واجهة | ✅ مسجلة كـ `SharedTranslationState` |
| `CurrentOrganizationState.cs` | حالة | ✅ مسجلة |
| `IAppStateService.cs` | واجهة | ❌ تحتاج تنفيذ في PWA |
| `IPatientSessionService.cs` | واجهة | ❌ تحتاج تنفيذ في PWA |
| `PatientSessionService.cs` | تنفيذ | ❌ تحتاج تقييم |
| `SharedApiService.cs` | تنفيذ | ❌ بديل محتمل |
| `ILocalNotificationService.cs` | واجهة | ✅ مسجلة كـ `WebLocalNotificationService` |
| `IMobileNavigationService.cs` | واجهة | ❌ خاص بـ MAUI — يحتاج بديل |
| `INearbyPharmacyService.cs` | واجهة | ❌ يحتاج تنفيذ (GPS) |
| `INotificationNavigationService.cs` | واجهة | ❌ يحتاج تنفيذ |
| `IPspUiService.cs` | واجهة | ❌ يحتاج تنفيذ |
| `IPublicApiService.cs` | واجهة | ❌ يحتاج تنفيذ |
| `InviteNavigationBridge.cs` | جسر | ❌ خاص بـ MAUI |
| `RepNavigationBridge.cs` | جسر | ❌ خاص بـ MAUI |
| `OsmPharmacyService.cs` | تنفيذ | ❌ يحتاج تقييم |
| `PendingRequestService.cs` | خدمة | ❌ يحتاج تقييم |
| `Models/NotificationItem.cs` | نموذج | ✅ جاهز |
| `Models/PspModels.cs` | نموذج | ✅ جاهز |
| `Models/LicenseSubmitData.cs` | نموذج | ✅ جاهز |
| `Models/MedicationRequestStatus.cs` | نموذج | ✅ جاهز |

---

### 2.3 ملفات الأنماط (CSS) — أهم الملفات حسب الدور

#### ملفات الـ Hero والتدرجات (لكل دور)

| الدور | ملف CSS | التدرج |
|------|---------|--------|
| 🏥 العيادة | `_pages/Clinic/clinic-dashboard.css` | `#0A3D5C → #1A7A7A` |
| 💊 الصيدلية | `_pages/Pharmacy/pharmacy-dashboard.css` | `#1A3A3A → #2D8B6E` |
| 👔 المندوب | `_pages/Rep/RepDashboard.css` | `#3D2E1A → #B88B4A` |

#### ملفات الأنماط حسب الفئة

| الفئة | عدد الملفات | أهم الملفات |
|-------|-------------|-------------|
| **Auth** | 3 | `Login.css`, `Register.css`, `_forget_password.css` |
| **Clinic** | 7 | `clinic-dashboard.css`, `clinic-patients.css`, `clinic-patient-details.css` |
| **Pharmacy** | 3 | `pharmacy-dashboard.css`, `pharmacy-gateway.css` |
| **Rep** | 4 | `RepDashboard.css`, `InviteDoctor.css`, `MyInvitations.css`, `MyNetwork.css` |
| **PSP** | 8 | `psp-dashboard.css`, `psp-gateway.css`, `PSPProgramsList.css` |
| **Patient** | 5 | `SchedulePage.css`, `PspEntry.css`, `order-tracker.css` |
| **Admin** | 20+ | ملفات متعددة حسب القسم |
| **Organization** | 10 | `CreateOrganization.css`, `MYOrganizations.css` |
| **PublicUser** | 8 | `Userprofile.css`, `Settings.css`, `ProfessionalLicensePage.css` |
| **Shared** | 2 | `Shared_web_Mobile.css`, `Pharmacist/PatientOrders.css` |

---

## 3. جرد شامل: Mobile (MAUI)

### 3.1 الصفحات حسب الـ Feature (57 صفحة)

#### 🔐 المصادقة (Auth) — 3 صفحات

| الصفحة | النوع | الحالة |
|--------|-------|--------|
| `LoginView.xaml` | XAML | ✅ موجودة في الـ PWA (`Login.razor`) |
| `RegisterPage.xaml` | XAML | ✅ موجودة في الـ PWA (`Register.razor`) |
| `EmailVerificationPage.xaml` | XAML | ❌ غير موجودة في الـ PWA |

#### 🏥 إدارة العيادة (ClinicManagement) — 5 صفحات

| الصفحة | النوع | الحالة |
|--------|-------|--------|
| `ClinicDashboardPage.xaml` | XAML | ✅ موجودة في الـ PWA (`ClinicDashboard.razor`) |
| `ClinicPatientDetailsPage.xaml` | XAML | ❌ غير موجودة في الـ PWA |
| `ClinicPatientsPage.xaml` | XAML | ❌ غير موجودة في الـ PWA |
| `MyClinicPage.xaml` | XAML | ❌ غير موجودة في الـ PWA |
| `MyPatientInvitationsFlow.xaml` | XAML | ❌ غير موجودة في الـ PWA |

#### 💊 الصيدلية (Pharmacist) — 4 صفحات

| الصفحة | النوع | الحالة |
|--------|-------|--------|
| `MyPharmacyPage.xaml` | XAML | ❌ غير موجودة في الـ PWA |
| `PharmacyDashboardPage.xaml` | XAML | ❌ غير موجودة في الـ PWA |
| `PharmacyGatewayFlow.xaml` | XAML | ❌ غير موجودة في الـ PWA |
| `PharmacyReportsFlow.xaml` | XAML | ❌ غير موجودة في الـ PWA |

#### 🎓 التسجيل المهني (ProfessionalOnboarding) — 3 صفحات

| الصفحة | النوع | الحالة |
|--------|-------|--------|
| `CreateOrganizationFlow.xaml` | XAML | ❌ غير موجودة في الـ PWA |
| `ProfessionalLicenseFlow.xaml` | XAML | ❌ غير موجودة في الـ PWA |
| `ProfessionalStatusFlow.xaml` | XAML | ❌ غير موجودة في الـ PWA |

#### 🌿 برامج الدعم (PSP) — 7 صفحات

| الصفحة | النوع | الحالة |
|--------|-------|--------|
| `InvitePatientPage.xaml` | XAML | ❌ غير موجودة في الـ PWA |
| `ProgramDetailsPage.xaml` | XAML | ❌ غير موجودة في الـ PWA |
| `PspAboutFlow.xaml` | XAML | ❌ غير موجودة في الـ PWA |
| `PspGatewayFlow.xaml` | XAML | ✅ موجودة في الـ PWA |
| `PspSearchPage.xaml` | XAML | ❌ غير موجودة في الـ PWA |
| `PspDetailPage.xaml` | XAML | ✅ موجودة في الـ PWA |
| `PspScheduleSetupHostPage.xaml` | XAML | ❌ غير موجودة في الـ PWA |

#### 👤 المستخدم العام (PublicUser) — 7 صفحات

| الصفحة | النوع | الحالة |
|--------|-------|--------|
| `DashboardPage.xaml` | XAML | ⚠️ موجودة في الـ PWA (بشكل مختلف) |
| `EditProfilePage.xaml` | XAML | ❌ غير موجودة في الـ PWA |
| `NearbyPharmaciesPage.xaml` | XAML | ⚠️ موجودة في الـ PWA |
| `PatientOrdersBlazorPage.xaml` | XAML | ⚠️ موجودة في الـ PWA |
| `ProfilePage.xaml` | XAML | ⚠️ موجودة في الـ PWA |
| `NearbyPharmaciesHost.razor` | Razor | ⚠️ موجودة في الـ PWA |

#### 👔 المندوب (Rep) — 5 صفحات

| الصفحة | النوع | الحالة |
|--------|-------|--------|
| `InviteDoctorPage.xaml` | XAML | ❌ غير موجودة في الـ PWA |
| `MyInvitationsPage.xaml` | XAML | ❌ غير موجودة في الـ PWA |
| `MyNetworkPage.xaml` | XAML | ❌ غير موجودة في الـ PWA |
| `PharmaCompanyDashboardPage.xaml` | XAML | ❌ غير موجودة في الـ PWA |
| `RepDashboardPage.xaml` | XAML | ❌ غير موجودة في الـ PWA |

#### 🛠️ المشتركة (Shared) — 23 صفحة

| الصفحة | النوع | الحالة |
|--------|-------|--------|
| `AddMedicationPageFlow.xaml` | XAML | ❌ غير موجودة في الـ PWA |
| `MedicationScheduleFlow.xaml` | XAML | ⚠️ موجودة في الـ PWA |
| `PspSchedulePageFlow.xaml` | XAML | ❌ غير موجودة في الـ PWA |
| `BlazorPage.xaml` | XAML | ✅ (هيكل عام) |
| `DoctorProfileFlow.xaml` | XAML | ⚠️ موجودة في الـ PWA |
| `DoctorSearchFlow.xaml` | XAML | ⚠️ موجودة في الـ PWA |
| `JobTitlesManagementFlow.xaml` | XAML | ❌ غير موجودة في الـ PWA |
| `MembersManagementFlow.xaml` | XAML | ❌ غير موجودة في الـ PWA |
| `MemberTitlesManagementFlow.xaml` | XAML | ❌ غير موجودة في الـ PWA |
| `MessagingHubFlow.xaml` | XAML | ❌ غير موجودة في الـ PWA |
| `MessagingSettingsFlow.xaml` | XAML | ⚠️ موجودة في الـ PWA |
| `NotificationsBlazorPage.xaml` | XAML | ⚠️ موجودة في الـ PWA |
| `OrganizationManagementFlow.xaml` | XAML | ❌ غير موجودة في الـ PWA |
| `PatientOrdersFlow.xaml` | XAML | ⚠️ موجودة في الـ PWA |
| `PharmacyDetailFlow.xaml` | XAML | ❌ غير موجودة في الـ PWA |
| `PharmacySearchFlow.xaml` | XAML | ⚠️ موجودة في الـ PWA |
| `PrivacyPage.xaml` | XAML | ❌ غير موجودة في الـ PWA |
| `ScanTokenPage.xaml` | XAML | ❌ غير موجودة في الـ PWA |
| `SettingsFlow.xaml` | XAML | ⚠️ موجودة في الـ PWA |
| `SupportFlow.xaml` | XAML | ❌ غير موجودة في الـ PWA |
| `TermsPage.xaml` | XAML | ❌ غير موجودة في الـ PWA |
| `LoadingPage.xaml` | XAML | ✅ موجودة في الـ PWA (`LoaderOverlay`) |

---

### 3.2 خدمات الموبايل (20+ خدمة)

| الخدمة | المسار | الحالة في الـ PWA |
|--------|--------|------------------|
| `ApiService.cs` | `Infrastructure/Services/` | ✅ بديل: `WebApiService` |
| `AuthService.cs` | `Infrastructure/Services/` | ❌ يحتاج بديل |
| `AppStateService.cs` | `Infrastructure/Services/` | ❌ يحتاج تنفيذ |
| `CachedUserSessionService.cs` | `Infrastructure/Services/` | ❌ خاص بـ MAUI |
| `DevIpService.cs` | `Infrastructure/Services/` | ❌ خاص بـ MAUI |
| `INavigationService.cs` | `Infrastructure/Services/` | ❌ خاص بـ MAUI |
| `MauiLocalNotificationService.cs` | `Infrastructure/Services/` | ❌ خاص بـ MAUI |
| `NavigationService.cs` | `Infrastructure/Services/` | ❌ خاص بـ MAUI |
| `NetworkDiscoveryService.cs` | `Infrastructure/Services/` | ❌ خاص بـ MAUI |
| `GeolocationService.cs` | `Services/` | ❌ يحتاج بديل (GPS) |
| `MauiNotificationNavigationService.cs` | `Services/` | ❌ خاص بـ MAUI |
| `MobileNavigationService.cs` | `Services/` | ❌ خاص بـ MAUI |
| `MobileNearbyPharmacyService.cs` | `Services/` | ❌ يحتاج بديل |
| `MobilePendingRequestService.cs` | `Services/` | ❌ يحتاج تقييم |
| `MobileTranslationService.cs` | `Services/` | ✅ بديل: `WebTranslationService` |
| `MobileTranslationState.cs` | `Services/` | ✅ بديل: `SharedTranslationState` |
| `PharmacyService.cs` | `Services/` | ❌ يحتاج تقييم |
| `PspUiService.cs` | `Services/` | ❌ يحتاج تقييم |
| `TranslationCacheService.cs` | `Services/` | ❌ خاص بـ MAUI |
| `AppShellViewModel.cs` | `ViewModels/` | ❌ خاص بـ MAUI |
| `BaseViewModel.cs` | `ViewModels/` | ❌ خاص بـ MAUI |

### 3.3 الـ ViewModels حسب الـ Feature (16 ملف)

| الـ Feature | الـ ViewModels |
|-------------|---------------|
| **Auth** | `EmailVerificationViewModel`, `LoginViewModel`, `RegisterViewModel` |
| **ClinicManagement** | `MyClinicViewModel` + 4 نماذج |
| **Pharmacist** | `MyPharmacyViewModel` |
| **PSP Doctor** | `InvitePatientViewModel`, `ProgramDetailsViewModel`, `PspSearchViewModel` |
| **PSP Patient** | `PspDetailViewModel` |
| **PublicUser** | `DashboardPageViewModel` |
| **Rep** | `InviteDoctorViewModel`, `RepDashboardViewModel` |

---

## 4. حالة الـ PWA الحالي

### 4.1 الهيكل الحالي

```
RubikCare.PWA/
├── Layout/
│   ├── EmptyLayout.razor          ✅
│   ├── MainLayout.razor           ✅
│   └── NavMenu.razor              ✅
├── Pages/
│   ├── Counter.razor              ❌ (صفحة تجريبية - يمكن حذفها)
│   ├── Dashboard.razor            ✅
│   ├── NotFound.razor             ✅
│   ├── Weather.razor              ❌ (صفحة تجريبية - يمكن حذفها)
│   ├── Auth/
│   │   ├── Login.razor            ✅
│   │   └── Register.razor         ✅
│   ├── Clinic/
│   │   └── ClinicDashboard.razor  ✅
│   ├── PSP/
│   │   ├── Psp-Detail.razor       ✅
│   │   ├── PspEntry.razor         ✅
│   │   └── PspGateway.razor       ✅
│   └── PublicUser/
│       ├── DoctorProfile.razor    ✅
│       ├── DoctorSearch.razor     ✅
│       ├── MedicationSchedulePage.razor ✅
│       ├── NearbyPharmacies.razor ✅
│       ├── Notifications.razor    ✅
│       ├── PatientOrders.razor    ✅
│       ├── PharmacySearch.razor   ✅
│       ├── Profile.razor          ✅
│       └── Setting/
│           ├── MessagingSettings.razor ✅
│           └── Settings.razor     ✅
├── Services/
├── wwwroot/
│   ├── css/
│   ├── images/
│   ├── js/
│   └── lib/
└── Program.cs                     ✅
```

### 4.2 الخدمات المسجلة في `Program.cs`

| الخدمة | الحالة |
|--------|--------|
| `IApiService` → `WebApiService` | ✅ مسجلة |
| `ITranslationService` → `WebTranslationService` | ✅ مسجلة |
| `ITranslationState` → `SharedTranslationState` | ✅ مسجلة |
| `ILocalNotificationService` → `WebLocalNotificationService` | ✅ مسجلة |
| `UserSessionState` | ✅ مسجلة |
| `CurrentOrganizationState` | ✅ مسجلة |

### 4.3 الخدمات غير المسجلة (تحتاج إضافة)

| الخدمة | السبب | الأولوية |
|--------|-------|----------|
| `AuthenticationStateProvider` | مطلوب لتسجيل الدخول عبر Google | 🔴 عالية |
| `IAppStateService` | مطلوب لإدارة حالة التطبيق | 🔴 عالية |
| `INearbyPharmacyService` | مطلوب للصيدليات القريبة | 🟡 متوسطة |
| `IPspUiService` | مطلوب لواجهات PSP | 🟡 متوسطة |
| `IPatientSessionService` | مطلوب لجلسات المريض | 🟢 منخفضة |

---

## 5. جدول المقارنة الكامل

### ملخص حسب الفئة

| الفئة | Mobile | Shared.UI | PWA الحالي | الناقص | نسبة الإنجاز |
|-------|:---:|:---:|:---:|:---:|:---:|
| **المصادقة** | 3 | — | 2 | 1 | 67% |
| **العيادة** | 5 | 4 | 1 | 3 | 20% |
| **الصيدلية** | 4 | 12 | 0 | 12 | 0% |
| **المندوب** | 5 | 5 | 0 | 5 | 0% |
| **برامج الدعم** | 7 | 6 | 3 | 3 | 43% |
| **المستخدم العام** | 7 | 16 | 10 | 6 | 63% |
| **المحادثات** | 2 | 7 | 1 | 6 | 14% |
| **المنظمات** | 1 | 4 | 0 | 4 | 0% |
| **القانونية** | 3 | 2 | 0 | 2 | 0% |
| **المساعدة** | 2 | 7 | 1 | 6 | 14% |
| **الإجمالي** | **39** | **63** | **18** | **45** | **~29%** |

---

## 6. خريطة النقل وحالة الإنجاز

### ✅ ما تم نقله (18 صفحة)

| الصفحة | الموقع في الـ PWA |
|--------|-------------------|
| `Login` | `Pages/Auth/Login.razor` |
| `Register` | `Pages/Auth/Register.razor` |
| `ClinicDashboard` | `Pages/Clinic/ClinicDashboard.razor` |
| `Psp-Detail` | `Pages/PSP/Psp-Detail.razor` |
| `PspEntry` | `Pages/PSP/PspEntry.razor` |
| `PspGateway` | `Pages/PSP/PspGateway.razor` |
| `DoctorProfile` | `Pages/PublicUser/DoctorProfile.razor` |
| `DoctorSearch` | `Pages/PublicUser/DoctorSearch.razor` |
| `MedicationSchedulePage` | `Pages/PublicUser/MedicationSchedulePage.razor` |
| `NearbyPharmacies` | `Pages/PublicUser/NearbyPharmacies.razor` |
| `Notifications` | `Pages/PublicUser/Notifications.razor` |
| `PatientOrders` | `Pages/PublicUser/PatientOrders.razor` |
| `PharmacySearch` | `Pages/PublicUser/PharmacySearch.razor` |
| `Profile` | `Pages/PublicUser/Profile.razor` |
| `MessagingSettings` | `Pages/PublicUser/Setting/MessagingSettings.razor` |
| `Settings` | `Pages/PublicUser/Setting/Settings.razor` |
| `Dashboard` | `Pages/Dashboard.razor` |
| `NotFound` | `Pages/NotFound.razor` |

### 🔴 ما لم يُنقل بعد (45 صفحة)

#### أولوية عالية (لوحات التحكم الرئيسية)
- [ ] `PharmacyDashboard` — لوحة الصيدلية
- [ ] `RepDashboard` — لوحة المندوب
- [ ] `PharmaCompanyDashboard` — لوحة شركة الأدوية
- [ ] `ClinicPatientDetails` — تفاصيل المريض
- [ ] `ClinicPatientsList` — قائمة المرضى

#### أولوية متوسطة (صفحات الميزات)
- [ ] `PspAboutPage` — عن برامج الدعم
- [ ] `PspSearch` — بحث برامج
- [ ] `MessagingHubPage` — مركز المحادثات
- [ ] `ChatPage` — الدردشة
- [ ] `ConversationsListPage` — قائمة المحادثات
- [ ] `OrderTracker` — تتبع الطلب
- [ ] `MyNetwork` — شبكة المندوب
- [ ] `MyInvitations` — دعواتي
- [ ] `InviteDoctor` — دعوة طبيب

#### أولوية منخفضة (صفحات ثانوية)
- [ ] `PrivacyPolicy` — سياسة الخصوصية
- [ ] `TermsOfService` — شروط الاستخدام
- [ ] `SupportPage` — صفحة الدعم
- [ ] `ProfessionalLicensePage` — الترخيص المهني
- [ ] `ProfessionalStatusPage` — الحالة المهنية
- [ ] `CreateOrganizationPage` — إنشاء منظمة
- [ ] `MembersTab` — تبويب الأعضاء
- [ ] `MemberTitlesTab` — ألقاب الأعضاء
- [ ] `CustomJobTitles` — المسميات الوظيفية

---

## 7. الخدمات: ما هو موجود وما ينقص

### الخدمات المشتركة المسجلة في الـ PWA

| الخدمة | الواجهة | التنفيذ في الـ PWA |
|--------|---------|-------------------|
| `IApiService` | ✅ | `WebApiService` |
| `ITranslationService` | ✅ | `WebTranslationService` |
| `ITranslationState` | ✅ | `SharedTranslationState` |
| `ILocalNotificationService` | ✅ | `WebLocalNotificationService` |

### الخدمات التي تحتاج تنفيذ أو بديل

| الخدمة | السبب | الحل المقترح | الأولوية |
|--------|-------|--------------|----------|
| `IMobileNavigationService` | خاص بـ MAUI | إنشاء `WebNavigationService` | 🔴 عالية |
| `INearbyPharmacyService` | يحتاج GPS | استخدام Geolocation API في المتصفح | 🟡 متوسطة |
| `IPspUiService` | غير مسجل | تنفيذ في الـ PWA | 🟡 متوسطة |
| `IAppStateService` | غير مسجل | تنفيذ في الـ PWA | 🟡 متوسطة |
| `IPatientSessionService` | خاص بـ MAUI | استخدام `UserSessionState` الحالي | 🟢 منخفضة |
| `AuthenticationStateProvider` | مطلوب لـ Google | تسجيل في `Program.cs` | 🔴 عالية |

---

## 📝 ملاحظات وتوصيات

### ملاحظات مهمة
1. **تجنب التكرار:** بعض الصفحات في `PublicUser` قد تكون نسخاً من مكونات `Patient` في `Shared.UI` — يجب التحقق من التكرار قبل النقل
2. **الخدمات الخاصة بـ MAUI:** مثل `NavigationService` و `GeolocationService` تحتاج بدائل في الـ PWA
3. **ملفات CSS جاهزة:** جميع ملفات الأنماط جاهزة في `Shared.UI` ولا تحتاج تعديل

### توصيات للنقل
1. **ابدأ بلوحات التحكم الرئيسية** (Pharmacy, Rep) لأنها الأكثر استخداماً
2. **انقل الخدمات أولاً** قبل الصفحات التي تعتمد عليها
3. **اختبر كل صفحة بعد النقل** للتأكد من عملها في بيئة WASM

---

## 🔗 روابط ذات صلة

- [00 - الهيكل المعماري](00-architecture-overview.md)
- [03 - دليل الأنماط](03-style-guide.md)
- [22 - RubikCare.PWA](22-RubikCare.PWA.md)
- [الملحق ب - فهرس الخدمات](appendix-b-service-index.md)

---

**آخر تحديث:** 29 أغسطس 2026 | **الملف:** `24-project-structure-inventory.md`

