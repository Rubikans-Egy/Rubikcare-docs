# 03 — PHARMACIST JOURNEY (رحلة الصيدلي)

الإصدار: 1.0
آخر تحديث: 15 نوفمبر 2026
الحالة: ✅ موثق
الجمهور: فريق التطوير + AI Assistants + فريق دعم الصيادلة + صانعو الفيديوهات

## 🎯 نظرة سريعة

| العنصر | القيمة |
|---|---|
| **Persona** | Pharmacist (الصيدلي) |
| **الحساب** | `UserProfile` + دور `Pharmacist` |
| **العضوية** | `OrgMembership` (OrganizationTypeID = 3 = Pharmacy) |
| **Dashboard** | `PharmacyDashboardPage` |
| **المنصات** | Web + PWA + MAUI |
| **البرامج المدعومة** | PSP + B2B Commerce + Pharmacy OS (مستقبلاً) |

## 📊 الرحلة الكاملة (6 Phases)

```
PHASE 1: REGISTRATION + PROFESSIONAL LICENSE
    ↓
PHASE 2: PHARMACY MEMBERSHIP
    ↓
PHASE 3: PHARMACY DASHBOARD
    ↓
PHASE 4: TOKEN REDEMPTION (صرف توكنات)
    ↓
PHASE 5: MEDICATION REQUESTS (طلبات أدوية)
    ↓
PHASE 6: B2B COMMERCE + REPORTS
```

---

## 🚀 PHASE 1: REGISTRATION + PROFESSIONAL LICENSE

**⚠️ نفس رحلة الطبيب تماماً — انظر [02-doctor.md](02-doctor.md).**

**الفرق:** بدلاً من "Doctor" — يختار "Pharmacist".

---

## 🏥 PHASE 2: PHARMACY MEMBERSHIP

### 📱 الخطوات

1. **الحصول على عضوية صيدلية**
   - **السيناريو 1 — دعوة من Rep:**
     - مندوب يرسل دعوة لـ "صيدلية".
   - **السيناريو 2 — إنشاء صيدلية جديدة:**
     - `create-organization` → `CreateOrganizationFlow`.

2. **التبديل إلى Pharmacy Mode**
   - **من Flyout:** "صيدليتي".
   - **`AppShellViewModel.SwitchToProfessionalMode("Pharmacist")`:**
     - جلب `session.Pharmacies.FirstOrDefault()`.
     - `CurrentOrganizationState.SetOrganizationId()`.
     - `GoToAsync("//PharmacyDashboardPage?PharmacyId=...")`.

3. **بعد التبديل**
   - `CurrentMode = AppMode.Pharmacist`.
   - Flyout يعرض "الوضع المهني - صيدلي".
   - زر "العودة للوضع الشخصي".

### 🔌 Endpoints

| Endpoint | Method | الغرض |
|---|---|---|
| `/api/clinic/my-pharmacies` | GET | صيدليات المستخدم |
| `/api/pharmacy/details/{id}` | GET | تفاصيل الصيدلية |

---

## 🏥 PHASE 3: PHARMACY DASHBOARD

### 📱 الصفحة الرئيسية

**`PharmacyDashboardPage` → `PharmacyDashboard.razor`:**

1. **Hero Section**
   - اسم الصيدلية.
   - اسم الصيدلي.
   - **Pills:**
     - صرف اليوم (`TodayDispenses`).
     - البرامج النشطة (`ActivePrograms`).
   - **Stats Row:**
     - صرف اليوم.
     - طلبات معلقة.
     - المرضى.

2. **Quick Actions**
   - **صيدليتي** → `MyPharmacyPage`.
   - **برامج PSP** → `PharmacyGatewayFlow`.
   - **التقارير** → `PharmacyReportsFlow`.
   - **طلبات المرضى** → `PatientOrdersFlow`.
   - **طلبات التوريد (B2B)** → `CommerceFlowPage`.

3. **Location Section (جديد)**
   - Latitude + Longitude.
   - زر "تحديد موقعي" (GPS).
   - زر "حفظ".

4. **Recent Dispenses**
   - آخر صرف.
   - زر "عرض الكل".

5. **Today Reminders**
   - طلبات Refills معلقة.
   - أو "لا توجد تذكيرات".

### 🔌 Endpoints

| Endpoint | Method | الغرض |
|---|---|---|
| `/api/pharmacy/dashboard/stats?pharmacyId={id}` | GET | إحصائيات الصيدلية |
| `/api/pharmacy/details/{id}` | GET | تفاصيل الصيدلية (بما فيها Location) |
| `/api/pharmacy/update-location` | PUT | حفظ الموقع |

### 📂 الملفات المتأثرة

- `PharmacyDashboard.razor`
- `PharmacyDashboardPage.xaml` + `.xaml.cs`
- `NearbyPharmacyService.cs` — `GetCurrentLocationAsync`

---

## 💊 PHASE 4: TOKEN REDEMPTION (صرف توكنات)

### 📱 الخطوات

1. **استلام التوكن من المريض**
   - **QR Code** — يمسحه الصيدلي.
   - **TokenCode** — يدخله يدوياً.

2. **فتح صفحة الصرف**
   - **المسار:** المريض → "إرسال للصيدلية" → `PharmacyDetailFlow?pharmacyId=X&token=Y`.
   - **أو:** الصيدلي يفتح `PharmacySearchPage` → يختار صيدلية → `PharmacyDetailFlow`.
   - **الصفحة:** `PharmacyDetailPage.razor` (Shared.UI).

3. **صرف التوكن (`TokenRedemptionCard`)**
   - **الخطوة 1 — إدخال التوكن:**
     - إدخال `TokenCode` أو مسح QR.
     - `POST /api/dispense/validate-token`.
     - إذا `Status = "APPROVED"` → `ShowDetails`.
     - إذا `Status = "PENDING_APPROVAL"` → `WaitingApproval`.
   - **الخطوة 2 — الانتظار (Auto-Refresh كل 5s):**
     - `GET /api/dispense/status/{dispensationId}`.
     - إذا `APPROVED` → `ShowDetails`.
     - إذا `REJECTED` → `EnterToken` + رسالة رفض.
   - **الخطوة 3 — عرض التفاصيل + تأكيد:**
     - تفاصيل الوصفة (المريض، الدواء، الكمية).
     - زر "تأكيد الاستلام".
     - `POST /api/dispense/confirm`.
   - **الخطوة 4 — النجاح:**
     - `Status = "DISPENSED"`.
     - Push Notification للمريض.

4. **التحقق من التعاقد**
   - إذا `TokenCode` غير موجود — `can-dispense-token`.
   - إذا الصيدلية غير متعاقدة → تحذير ("غير متعاقدة").

### 🔌 Endpoints

| Endpoint | Method | الغرض |
|---|---|---|
| `/api/dispense/can-dispense-token` | GET | التحقق من التعاقد |
| `/api/dispense/validate-token` | POST | التحقق من التوكن |
| `/api/dispense/submit-for-approval` | POST | إرسال الموافقة |
| `/api/dispense/status/{id}` | GET | حالة الموافقة |
| `/api/dispense/confirm` | POST | تأكيد الصرف |
| `/api/dispense/apply-points` | POST | تطبيق نقاط المريض |

### 📂 الملفات المتأثرة

- `PharmacyDetailPage.razor`
- `PharmacyDetailFlow.xaml` + `.xaml.cs`
- `TokenRedemptionCard.razor`

---

## 💊 PHASE 5: MEDICATION REQUESTS (طلبات أدوية)

### 📱 الخطوات

1. **إنشاء طلب دواء**
   - **الصفحة:** `MedicationRequestCard.razor` (Shared.UI).
   - **الخطوات:**
     - البحث عن دواء (`SearchMedications`).
     - اختيار الدواء (Checkbox).
     - تحديد الكمية + ملاحظات.
     - `SubmitRequest` → `POST /api/pharmacy/medication-request`.

2. **عرض طلباتي**
   - **من Quick Actions:** "طلبات المرضى".
   - **الصفحة:** `PatientOrdersFlow` → `PatientOrdersBlazorPage`.
   - **المحتوى:**
     - قائمة الطلبات.
     - حالة كل طلب (Pending / Approved / Delivered).
     - تفاصيل الطلب.

3. **عرض تفاصيل طلب**
   - **الصفحة:** `OrderDetailsPage.razor`.
   - **المحتوى:**
     - رقم الطلب.
     - بيانات المريض.
     - الأدوية.
     - زر "تتبع" → `OrderTrackingPage`.
     - زر "قبول" / "رفض" (حسب الصلاحية).

### 🔌 Endpoints

| Endpoint | Method | الغرض |
|---|---|---|
| `/api/pharmacy/medication-request` | POST | إنشاء طلب |
| `/api/commerce/supply-orders/pharmacy/{id}` | GET | طلبات الصيدلية |
| `/api/commerce/supply-orders/{id}` | GET | تفاصيل طلب |

### 📂 الملفات المتأثرة

- `MedicationRequestCard.razor`
- `PharmacyOrdersPage.razor`
- `PatientOrdersFlow.xaml` + `.xaml.cs`
- `OrderDetailsPage.razor`
- `OrderTrackingPage.razor`

---

## 🛒 PHASE 6: B2B COMMERCE + REPORTS

### 📱 الخطوات

1. **فتح Commerce**
   - **من Quick Actions:** "طلبات التوريد (B2B)".
   - **الصفحة:** `CommerceFlowPage.razor` (Shared.UI).

2. **تصفح الكتالوج**
   - **الصفحة:** `PharmacyCatalogPage.razor`.
   - **المحتوى:**
     - اختيار شركة الأدوية (`_selectedCompanyId`).
     - قائمة الأدوية.
     - بحث + Grid/List.
     - إضافة للسلة (`AddToCart`).

3. **عرض تفاصيل منتج**
   - **الصفحة:** `ProductDetailsPage.razor`.
   - **المحتوى:**
     - معلومات الدواء.
     - الأسعار.
     - الخصومات.
     - البونص (`BonusDto`).
     - زر "إضافة للسلة".

4. **عرض السلة**
   - **الصفحة:** `PharmacyCartPage.razor`.
   - **المحتوى:**
     - قائمة الأدوية.
     - تعديل الكميات.
     - حساب الإجمالي (SubTotal + Discount + Tax).
     - زر "تأكيد الطلب" → `SubmitOrderAsync`.
     - **Endpoint:** `POST /api/commerce/supply-orders`.
     - **ثم:** `POST /api/commerce/supply-orders/{id}/submit`.

5. **متابعة الطلبات**
   - **الصفحة:** `PharmacyOrdersPage.razor`.
   - **المحتوى:**
     - قائمة الطلبات.
     - فلترة حسب الحالة.
     - تفاصيل كل طلب.

6. **تتبع الطلب**
   - **الصفحة:** `OrderTrackingPage.razor`.
   - **المحتوى:**
     - Timeline (Draft → Submitted → Confirmed → Preparing → Shipped → Delivered).
     - زر "رجوع".

7. **التقارير**
   - **الصفحة:** `PharmacyReportsFlow` (مستقبلاً).
   - **المحتوى:**
     - صرف اليوم / الأسبوع / الشهر.
     - الأدوية الأكثر صرفاً.
     - الأرباح.
     - الالتزام.

### 🔌 Endpoints

| Endpoint | Method | الغرض |
|---|---|---|
| `/api/commerce/catalog/companies` | GET | شركات الأدوية المتاحة |
| `/api/commerce/catalog/{companyId}` | GET | كتالوج الشركة |
| `/api/commerce/catalog/{companyId}/search?term={term}` | GET | البحث |
| `/api/commerce/bonuses/company/{companyId}` | GET | البونص |
| `/api/commerce/supply-orders` | POST | إنشاء طلب |
| `/api/commerce/supply-orders/{id}/submit` | POST | إرسال الطلب |
| `/api/commerce/supply-orders/pharmacy/{id}` | GET | طلبات الصيدلية |

### 📂 الملفات المتأثرة

- `CommerceFlowPage.razor`
- `PharmacyCatalogPage.razor`
- `ProductDetailsPage.razor`
- `PharmacyCartPage.razor`
- `PharmacyOrdersPage.razor`
- `OrderTrackingPage.razor`

---

## 🌐 الميزات المشتركة في رحلة الصيدلي

### 1. Offline Mode
- **يعمل:**
  - الترجمات.
  - جلسة المستخدم + عضوية الصيدلية.
- **لا يعمل:**
  - الإحصائيات.
  - صرف توكنات جديدة.
  - طلبات B2B جديدة.

### 2. Central Back Button
- **في `PharmacyDashboardPage`** — `AppShell.OnBackButtonPressed`.
- **في `PharmacyDetailFlow`** — زر Back → `pharmacy-search` (عبر `IBackButtonHandler`).

### 3. Messaging
- **Pharmacist ←→ Patient** (استفسارات، توكن).
- **Pharmacist ←→ Rep** (طلبات توريد).

### 4. Notifications
- **Push:**
  - عند استلام طلب جديد.
  - عند صرف دواء بنجاح.
  - تذكيرات Refills.

---

## 📊 جدول المراحل (ملخص)

| # | المرحلة | Endpoint | الحالة |
|---|---|---|---|
| 1 | Onboarding | (شاشات) | ✅ |
| 2 | Registration | `POST /api/auth/register` | ✅ |
| 3 | Professional License | `POST /api/professional/register` | ✅ |
| 4 | Session Bootstrap | `POST /api/user/session-bootstrap` | ✅ |
| 5 | Pharmacy Membership | `GET /api/clinic/my-pharmacies` | ✅ |
| 6 | Pharmacy Dashboard | `GET /api/pharmacy/dashboard/stats` | ✅ |
| 7 | Validate Token | `POST /api/dispense/validate-token` | ✅ |
| 8 | Confirm Dispense | `POST /api/dispense/confirm` | ✅ |
| 9 | Medication Request | `POST /api/pharmacy/medication-request` | ✅ |
| 10 | B2B Order | `POST /api/commerce/supply-orders` | ✅ |
| 11 | Track Order | `GET /api/commerce/supply-orders/{id}` | ✅ |

---

## 📂 الملفات المتأثرة (كاملة)

### Mobile (MAUI)
- `ProfessionalStatusFlow.xaml` + `.xaml.cs`
- `ProfessionalLicenseFlow.xaml` + `.xaml.cs`
- `PharmacyDashboardPage.xaml` + `.xaml.cs`
- `MyPharmacyPage.xaml` + `.xaml.cs`
- `PharmacyGatewayFlow.xaml` + `.xaml.cs`
- `PharmacyReportsFlow.xaml` + `.xaml.cs`
- `PatientOrdersFlow.xaml` + `.xaml.cs`
- `PharmacySearchFlow.xaml` + `.xaml.cs`
- `PharmacyDetailFlow.xaml` + `.xaml.cs`
- `CommerceFlowPage.xaml` + `.xaml.cs`

### Shared.UI
- `PharmacyDashboard.razor`
- `PharmacySearchPage.razor`
- `PharmacyFilter.razor`
- `PharmacyDetailPage.razor`
- `TokenRedemptionCard.razor`
- `MedicationRequestCard.razor`
- `CommerceFlowPage.razor`
- `PharmacyCatalogPage.razor`
- `ProductDetailsPage.razor`
- `PharmacyCartPage.razor`
- `PharmacyOrdersPage.razor`
- `OrderTrackingPage.razor`

### Backend (للمرجعية)
- `DispenseController.cs`
- `PharmacyController.cs`
- `CommerceController.cs`
- `PharmacyMedicationRequestController.cs`

---

## 🔗 روابط سريعة

- **[00-overview.md](00-overview.md)**
- **[01-patient.md](01-patient.md)**
- **[02-doctor.md](02-doctor.md)**
- **[06-cross-cutting.md](06-cross-cutting.md)**
- **[05-PSP.md](../05-PSP.md)**
- **[23-COMMERCE-ENGINE.md](../23-COMMERCE-ENGINE.md)**

© 2026 RubikCare — للاستخدام الداخلي
```
