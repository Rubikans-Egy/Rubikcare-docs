
# 📄 24 - جرد هيكلة المشاريع وحالة النقل

**آخر تحديث:** 3 أكتوبر 2026
**الهدف:** مرجع دائم لتجنب إعادة الجرد والفحص في كل جلسة
**الحالة:** ✅ محدّث بعد إضافة نظام النقاط + PWA

---

## 📌 جدول المحتويات

1. [نظرة عامة على المشاريع](#1-نظرة-عامة-على-المشاريع)
2. [جرد شامل: Shared.UI](#2-جرد-شامل-sharedui)
3. [جرد شامل: Mobile (MAUI)](#3-جرد-شامل-mobile-maui)
4. [حالة الـ PWA](#4-حالة-الـ-pwa)
5. [🆕 جرد نظام النقاط](#5-جرد-نظام-النقاط)
6. [جدول المقارنة الكامل](#6-جدول-المقارنة-الكامل)
7. [خريطة النقل وحالة الإنجاز](#7-خريطة-النقل-وحالة-الإنجاز)
8. [الخدمات: ما هو موجود وما ينقص](#8-الخدمات-ما-هو-موجود-وما-ينقص)

---

## 1. نظرة عامة على المشاريع

| المشروع | التقنية | الغرض | عدد المكونات/الصفحات |
|---------|---------|-------|----------------------|
| **Shared.UI** | Razor Class Library (RCL) | مكونات UI مشتركة بين جميع المنصات | 68 مكون Razor + 120+ ملف CSS + 25 خدمة |
| **Mobile** | .NET MAUI + BlazorWebView | تطبيق الموبايل (Android + iOS) | 57 صفحة XAML + 16 ViewModel + 20+ خدمة |
| **PWA** | Blazor WebAssembly | نسخة كربونية من MAUI للمتصفح | 25+ صفحة Razor |

### العلاقة بين المشاريع الثلاثة

```
┌─────────────────────────────────────────────────────────────────┐
│                    RubikCare.Shared.UI                          │
│              (مكتبة المكونات المشتركة - RCL)                    │
│  • 68 مكون Razor جاهز                                            │
│  • 120+ ملف CSS منظم حسب الأدوار                                │
│  • 25 خدمة/واجهة                                                │
└─────────────────────────────────────────────────────────────────┘
                                      │
        ┌─────────────────────────────┼─────────────────────────────┐
        │                             │                             │
        ▼                             ▼                             ▼
┌─────────────────┐          ┌─────────────────┐          ┌─────────────────┐
│   Mobile (MAUI)  │          │   Web (Blazor)   │          │      PWA         │
│  57 صفحة XAML    │          │   (Blazor Server)│          │  Blazor WASM     │
│  BlazorWebView   │          │                  │          │  نسخة كربونية    │
└─────────────────┘          └─────────────────┘          └─────────────────┘
```

---

## 2. جرد شامل: Shared.UI

### 2.1 المكونات حسب الفئة (68 مكون)

#### 🏥 فئة العيادة (Clinic) — 4 مكونات

| المكون | الوظيفة |
|--------|---------|
| `ClinicDashboard.razor` | لوحة تحكم الطبيب |
| `ClinicPatientDetails.razor` | تفاصيل مريض |
| `ClinicPatientsList.razor` | قائمة المرضى |
| `MyPatientInvitations.razor` | دعوات المرضى |

#### 💊 فئة الصيدلية (Pharmacy) — 12 مكون

| المكون | الوظيفة |
|--------|---------|
| `PharmacyDashboard.razor` | لوحة تحكم الصيدلية |
| `PharmacyDetailPage.razor` | تفاصيل صيدلية |
| `PharmacySearchPage.razor` | بحث صيدليات |
| `NearbyPharmacies.razor` | الصيدليات القريبة |
| `PharmacyGateway.razor` | بوابة الصيدلية |
| `PharmacyReportsPage.razor` | تقارير الصيدلية |
| `PatientOrders.razor` | طلبات المرضى |
| `MedicationRequestCard.razor` | بطاقة طلب دواء |
| `TokenRedemptionCard.razor` | بطاقة استرداد رمز |
| `PharmacyCard.razor` | بطاقة صيدلية |
| `PharmacyGrid.razor` | شبكة صيدليات |
| `PharmacyFilter.razor` | فلتر البحث |

#### 👔 فئة المندوب (Rep) — 5 مكونات

| المكون | الوظيفة |
|--------|---------|
| `RepDashboard.razor` | لوحة المندوب |
| `PharmaCompanyDashboard.razor` | لوحة شركة الأدوية |
| `InviteDoctor.razor` | دعوة طبيب |
| `MyInvitations.razor` | دعواتي |
| `MyNetwork.razor` | شبكتي |

#### 👤 فئة المريض (Patient) — 16 مكون

| المكون | الوظيفة |
|--------|---------|
| `SettingsPage.razor` | الإعدادات الرئيسية |
| `MyProfilePage.razor` | ملفي الشخصي |
| `EditProfilePage.razor` | تعديل الملف |
| `PatientOrdersPage.razor` | طلبات المريض |
| `OrderTracker.razor` | تتبع الطلب |
| `NotificationsPage.razor` | الإشعارات |
| `MedicationSchedulePage.razor` | جدول الأدوية |
| `AddMedicationPage.razor` | إضافة دواء |
| `PspSchedulePage.razor` | جدولة PSP |
| `DailyDosesTab.razor` | تبويب الجرعات اليومية |
| `RefillDatesTab.razor` | تبويب مواعيد الصرف |
| `ProfessionalStatusPage.razor` | الحالة المهنية |
| `ProfessionalLicensePage.razor` | الترخيص المهني |
| `SecuritySection.razor` | قسم الأمان |
| `LanguageSection.razor` | قسم اللغة |
| `NotificationsSection.razor` | قسم الإشعارات |
| `PrivacySection.razor` | قسم الخصوصية |

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

#### 🌿 فئة برامج الدعم (PSP) — 8 مكونات

| المكون | الوظيفة |
|--------|---------|
| `PspAboutPage.razor` | عن برامج الدعم |
| `PspGateway.razor` | بوابة PSP |
| `PspSearch.razor` | بحث برامج |
| `PspEntry.razor` | دخول برنامج |
| `PspScheduleSetup.razor` | إعداد جدولة |
| `PspDetailComponent.razor` | تفاصيل برنامج |
| `ProgramDetailsComponent.razor` | تفاصيل برنامج (طبيب) |
| **`PointDashboard.razor`** ⭐ | **محفظة النقاط (جديد)** |

#### 🏢 إدارة المنظمات (OrganizationManagement) — 4 مكونات

| المكون | الوظيفة |
|--------|---------|
| `CreateOrganizationPage.razor` | إنشاء منظمة |
| `MembersTab.razor` | تبويب الأعضاء |
| `MemberTitlesTab.razor` | ألقاب الأعضاء |
| `CustomJobTitles.razor` | المسميات الوظيفية |

#### ⚖️ الفئة القانونية (Legal) — 2 مكونات

| المكون | الوظيفة |
|--------|---------|
| `PrivacyPolicy.razor` | سياسة الخصوصية |
| `TermsOfService.razor` | شروط الاستخدام |

#### 🛠️ مكونات مساعدة مشتركة — 7 مكونات

| المكون | الوظيفة |
|--------|---------|
| `LoaderOverlay.razor` | شاشة تحميل |
| `RubikButton.razor` | زر موحد |
| `TestPage.razor` | صفحة اختبار |
| `Pagination.razor` | ترقيم الصفحات |
| `SearchBar.razor` | شريط بحث |
| `RubikSmartTable.razor` | جدول ذكي |
| `SupportPage.razor` | صفحة دعم |

---

### 2.2 خدمات وواجهات Shared.UI (25 ملف)

| الخدمة/الواجهة | النوع | الحالة |
|-----------------|-------|--------|
| `IApiService.cs` | واجهة | ✅ |
| `ITranslationService.cs` | واجهة | ✅ |
| `ITranslationState.cs` | واجهة | ✅ |
| `CurrentOrganizationState.cs` | حالة | ✅ |
| `IAppStateService.cs` | واجهة | ⚠️ |
| `IPatientSessionService.cs` | واجهة | ⚠️ |
| `PatientSessionService.cs` | تنفيذ | ⚠️ |
| `SharedApiService.cs` | تنفيذ | ✅ |
| `ILocalNotificationService.cs` | واجهة | ✅ |
| `IMobileNavigationService.cs` | واجهة | ⚠️ |
| `INearbyPharmacyService.cs` | واجهة | ⚠️ |
| `INotificationNavigationService.cs` | واجهة | ✅ |
| `IPspUiService.cs` | واجهة | ✅ |
| `IPublicApiService.cs` | واجهة | ⚠️ |
| `InviteNavigationBridge.cs` | جسر | ✅ |
| `RepNavigationBridge.cs` | جسر | ✅ |
| `PspGatewayNavigationBridge.cs` | جسر | ✅ |
| **`PointDashboardNavigationBridge.cs`** ⭐ | **جسر (جديد)** | ✅ |
| `OsmPharmacyService.cs` | تنفيذ | ⚠️ |
| `PendingRequestService.cs` | خدمة | ⚠️ |
| `Models/NotificationItem.cs` | نموذج | ✅ |
| `Models/PspModels.cs` | نموذج | ✅ |
| `Models/LicenseSubmitData.cs` | نموذج | ✅ |
| `Models/MedicationRequestStatus.cs` | نموذج | ✅ |
| **`Models/PointModels.cs`** ⭐ | **نموذج (جديد)** | ✅ |

---

## 3. جرد شامل: Mobile (MAUI)

### 3.1 الصفحات حسب الـ Feature (57 صفحة)

**⚠️ ملاحظة:** جميع صفحات MAUI (57) لها نظير في PWA. للحصول على قائمة تفصيلية، راجع وثيقة 22-PWA.

---

## 4. حالة الـ PWA

### 4.1 نسبة الإنجاز

| العنصر | النسبة |
|--------|:------:|
| **البنية التحتية** | ✅ 100% |
| **صفحات المصادقة** | ✅ 100% |
| **صفحات PSP** | ✅ 100% |
| **صفحات المريض** | ✅ 100% |
| **صفحات الطبيب** | ✅ 100% |
| **صفحات الصيدلية** | ✅ 100% |
| **صفحات المندوب** | ✅ 100% |
| **نظام النقاط** | ✅ 100% |
| **نظام المحادثات** | ✅ 90% |
| **الإشعارات Push** | ✅ 100% |
| **Google Login** | ✅ 100% |
| **الإجمالي** | **~95%** |

### 4.2 الخدمات المسجلة في `Program.cs`

| الخدمة | الحالة |
|--------|--------|
| `IApiService` → `WebApiService` | ✅ |
| `ITranslationService` → `WebTranslationService` | ✅ |
| `ITranslationState` → `SharedTranslationState` | ✅ |
| `ILocalNotificationService` → `WebLocalNotificationService` | ✅ |
| `UserSessionState` | ✅ |
| `CurrentOrganizationState` | ✅ |
| `AuthenticationStateProvider` → `CustomAuthenticationStateProvider` | ✅ |

---

## 5. 🆕 جرد نظام النقاط

### 5.1 الموقع في المشاريع الثلاثة

| الطبقة | المكوّن | العدد | الموقع |
|--------|---------|:-----:|--------|
| **Domain** | كيانات | 8 | `Entities/PSP/Points/` |
| **Domain** | Lookups | 3 | `Entities/PSP/Points/Lookups/` |
| **Domain** | Constants | 3 | `Constants/PSP/Points/` |
| **Application** | Services | 2 | `Services/PSP/` |
| **Infrastructure** | Handlers | 19 | `UseCases/PSP/Points/` |
| **Api.Web** | Controllers | 5 | `Controllers/PSP/Points/` |
| **Shared.UI** | Components | 1 | `Components/PSP/Points/` |
| **Shared.UI** | Bridge | 1 | `Components/PSP/Points/` |
| **Shared.UI** | Models | 1 | `Components/PSP/Points/` |
| **Web** | Razor | 1 | `Components/Pages/Professional/PSPSteps/` |
| **PWA** | Page | 1 | `Pages/PSP/Points/` |
| **MAUI** | Container | 1 | `Features/PSP/Doctor/Views/` |

### 5.2 الكيانات (Domain) — 8

| # | الكيان | الغرض |
|---|--------|-------|
| 1 | `PSPActivityType` | Lookup — أنواع الأحداث (7) |
| 2 | `PSPProgramPointTransactionType` | Lookup — أنواع المعاملات (4) |
| 3 | `PSPProgramPointRedeemStatus` | Lookup — حالات Redeem (4) |
| 4 | `PSPProgramPointScheme` | نظام النقاط لكل برنامج |
| 5 | `PSPProgramPointRule` | قواعد المنح |
| 6 | `PSPProgramPointBalance` | أرصدة العيادات |
| 7 | `PSPProgramPointTransaction` | سجل المعاملات |
| 8 | `PSPProgramPointRedeemRequest` | طلبات الـ Redeem |

### 5.3 Handlers (Infrastructure) — 19

| المجموعة | العدد | الملفات |
|----------|:-----:|---------|
| Schemes | 4 | `CreatePointSchemeHandler`, `UpdatePointSchemeHandler`, `GetPointSchemeByProgramHandler`, `DeactivatePointSchemeHandler` |
| Rules | 5 | `CreatePointRuleHandler`, `UpdatePointRuleHandler`, `DeletePointRuleHandler`, `GetPointRulesBySchemeHandler`, `TogglePointRuleHandler` |
| Balances | 3 | `GetBalanceByOrganizationHandler`, `GetAllBalancesBySchemeHandler`, `GetOrganizationPointSummaryHandler` |
| Earning | 2 | **`EarnPointsHandler`** ⭐, `GetPointTransactionsHandler` |
| Redeem | 5 | `CreateRedeemRequestHandler` ⭐, `AcceptRedeemRequestHandler` ⭐, `RejectRedeemRequestHandler`, `GetRedeemRequestsHandler`, `GetRedeemRequestDetailsHandler` |

**⚠️ الـ Transaction مُطبَّق في:** `EarnPointsHandler`, `CreateRedeemRequestHandler`, `AcceptRedeemRequestHandler`.

### 5.4 Controllers (Api.Web) — 5

| Controller | Endpoints |
|-----------|:---------:|
| `PointSchemesController` | 4 |
| `PointRulesController` | 5 |
| `PointBalancesController` | 3 |
| `PointEarningController` | 1 |
| `PointRedeemController` | 5 |

### 5.5 المكونات في Shared.UI

| الملف | الغرض |
|-------|-------|
| `PointDashboard.razor` | لوحة النقاط الكاملة |
| `PointDashboardNavigationBridge.cs` | جسر التنقل |
| `PointModels.cs` | DTOs |

### 5.6 ربط الأحداث — 5 من 7

| # | الحدث | الموقع | الحالة |
|---|-------|--------|:------:|
| 1 | `PATIENT_INVITED` | `PSPEnrollmentService` | ✅ |
| 2 | `PATIENT_ENROLLED` | `PSPEnrollmentService` | ✅ |
| 3 | `ERX_CREATED` | `PSPEnrollmentService` | ✅ |
| 4 | `DISPENSATION_COMPLETED` | `DispenseController` | ✅ |
| 5 | `REFILL_COMPLETED` | `DispenseController` | ✅ |
| 6 | `PROGRAM_COMPLETED` | مؤجل | ⏸️ |
| 7 | `LAB_TEST_UPLOADED` | مؤجل | ⏸️ |

---

## 6. جدول المقارنة الكامل

### ملخص حسب الفئة

| الفئة | Mobile | Shared.UI | PWA | النسبة |
|-------|:---:|:---:|:---:|:---:|
| **المصادقة** | 3 | — | 2 | ✅ 100% |
| **العيادة** | 5 | 4 | 4 | ✅ 100% |
| **الصيدلية** | 4 | 12 | 4 | ✅ 100% |
| **المندوب** | 5 | 5 | 5 | ✅ 100% |
| **برامج الدعم** | 7 | 8 | 7 | ✅ 100% |
| **المستخدم العام** | 7 | 16 | 12 | ✅ 100% |
| **المحادثات** | 2 | 7 | 5 | ✅ 90% |
| **المنظمات** | 1 | 4 | 4 | ✅ 100% |
| **القانونية** | 3 | 2 | 3 | ✅ 100% |
| **نظام النقاط** | 1 | 1 | 1 | ✅ 100% |

---

## 7. خريطة النقل وحالة الإنجاز

### ✅ ما تم نقله إلى PWA

**⚠️ PWA مكتمل بنسبة عالية — يحوي نفس صفحات MAUI الأساسية.**

- **المصادقة**: Login, Register, ExternalLogin
- **PSP**: Gateway, Detail, Entry, Search, Points Dashboard
- **المستخدم العام**: Dashboard, Profile, Settings, Orders, Notifications
- **الأطباء**: Search, Profile, Booking
- **الصيدليات**: Search, Detail, Nearby
- **المحادثات**: ChatPage, ConversationsList, MessagingSettings
- **نظام النقاط**: PointsDashboard

### 🔄 ما لم يكتمل بعد

- **MessagingHubPage** — قيد التطوير.
- **تجربة شاملة (E2E)** — لم تُنفَّذ بعد.

---

## 8. الخدمات: ما هو موجود وما ينقص

### الخدمات المشتركة المسجلة في PWA

| الخدمة | الواجهة | التنفيذ في PWA |
|--------|:-------:|:--------------:|
| `IApiService` | ✅ | `WebApiService` |
| `ITranslationService` | ✅ | `WebTranslationService` |
| `ITranslationState` | ✅ | `SharedTranslationState` |
| `ILocalNotificationService` | ✅ | `WebLocalNotificationService` |
| `AuthenticationStateProvider` | ✅ | `CustomAuthenticationStateProvider` |

### الخدمات التي تحتاج تنفيذ في PWA

| الخدمة | السبب | الأولوية |
|--------|-------|:--------:|
| `INearbyPharmacyService` | يحتاج GPS | 🟡 |
| `IPspUiService` | لواجهات PSP | 🟢 |
| `IAppStateService` | لإدارة حالة التطبيق | 🟢 |

---

## 📝 ملاحظات مهمة

1. **PWA هو نسخة كربونية من Mobile** — يحوي نفس الصفحات والمكونات.
2. **نظام النقاط متعدد المنصات** — يعمل في Web و PWA و MAUI.
3. **الـ `PointDashboard` هو المكوّن الوحيد الجديد** في `Shared.UI` المرتبط بنظام النقاط.
4. **الأحداث الـ 5 من 7** مربوطة — `PROGRAM_COMPLETED` و `LAB_TEST_UPLOADED` مؤجلان.

---

## 🔗 روابط ذات صلة

- [00 - الهيكل المعماري](./00-architecture-overview.md)
- [08 - خطة التطوير](./08-roadmap.md)
- [11 - دليل BlazorWebView](./11-blazor-webview-guide.md)
- [22 - RubikCare.PWA](./22-RubikCare.PWA.md)
- [26 - نظام النقاط](./26-rubika-points-system.md)
- [27 - الديون التقنية](./27-technical-debt.md)

---

## 📝 سجل التحديثات

| التاريخ | التغيير | السبب |
| :--- | :--- | :--- |
| 3 أكتوبر 2026 | **إضافة قسم نظام النقاط (5)** | توثيق النظام الجديد. |
| 3 أكتوبر 2026 | **تحديث حالة PWA** إلى ~95% | بناءً على وثيقة 22. |
| 3 أكتوبر 2026 | **تحديث جرد Shared.UI** (66 → 68 مكون). | إضافة مكونات النقاط. |
| 29 أغسطس 2026 | **إعادة كتابة شاملة**. | دمج PWA. |

---

**آخر تحديث:** 3 أكتوبر 2026
**الملف:** `24-project-structure-inventory.md`
````

---

## ✅ ما تم في هذا التحديث

| # | الإضافة | التفاصيل |
|---|---------|----------|
| 1 | **قسم كامل لنظام النقاط** | 8 كيانات + 19 Handler + 5 Controllers |
| 2 | **تحديث إحصائيات Shared.UI** | 66 → 68 مكون، 23 → 25 خدمة |
| 3 | **تحديث حالة PWA** | ~95% (بدلاً من 35%) |
| 4 | **إضافة `PointDashboard`** | في قائمة مكونات PSP |
| 5 | **إضافة `PointDashboardNavigationBridge`** | في قائمة الخدمات |
| 6 | **إضافة `PointModels.cs`** | في قائمة النماذج |
| 7 | **ربط الأحداث الـ 5** | `PATIENT_INVITED` → `REFILL_COMPLETED` |
| 8 | **سجل التحديثات** | 3 أكتوبر 2026 |

