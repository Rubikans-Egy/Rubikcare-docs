# 05 — PUBLIC USER JOURNEY (رحلة المستخدم العام)

الإصدار: 1.0
آخر تحديث: 15 نوفمبر 2026
الحالة: ✅ موثق
الجمهور: فريق التطوير + AI Assistants + فريق دعم المستخدمين + صانعو الفيديوهات

## 🎯 نظرة سريعة

| العنصر | القيمة |
|---|---|
| **Persona** | Public User (المستخدم العام) |
| **الحساب** | `UserProfile` (بدون عضوية مؤسسة) |
| **Dashboard** | `DashboardFlowPage` (Personal) |
| **المنصات** | Web + PWA + MAUI |
| **الاستخدام الأساسي** | البحث + الحجز + المحادثات |

## 📊 الرحلة الكاملة (5 Phases)

```
PHASE 1: REGISTRATION
    ↓
PHASE 2: DOCTOR SEARCH + BOOK
    ↓
PHASE 3: PHARMACY SEARCH + DETAIL
    ↓
PHASE 4: NEARBY PHARMACIES
    ↓
PHASE 5: MESSAGING
```

---

## 🚀 PHASE 1: REGISTRATION

### 📱 الخطوات

**⚠️ نفس رحلة المريض — انظر [01-patient.md](01-patient.md) Phase 1.**

**الفرق:** المستخدم العام **لا ينضم لبرنامج PSP** — يستخدم المنصة للبحث والحجز فقط.

---

## 🔍 PHASE 2: DOCTOR SEARCH + BOOK

### 📱 الخطوات

1. **فتح البحث عن طبيب**
   - **من Personal Dashboard:** Quick Action "البحث عن طبيب".
   - **من Flyout:** "بحث عن طبيب".
   - **الصفحة:** `DoctorSearchFlow.xaml` → `DoctorSearchPage.razor` (Shared.UI).

2. **البحث + الفلترة**
   - **حقل البحث:** `searchTerm`.
   - **فلتر التخصص الرئيسي:** `selectedMainSpeciality`.
   - **فلتر التخصص الفرعي:** `selectedSubSpeciality`.
   - **زر البحث:** `SearchClinics`.

3. **قائمة النتائج**
   - Grid of cards (10 per page).
   - كل عيادة → `UserClinicDto`:
     - صورة العيادة (Base64).
     - اسم العيادة.
     - التخصص الرئيسي.
     - قيمة الكشف (`ExaminationFee`).
     - المنطقة.
   - **Pagination** (إذا `totalPages > 1`).

4. **عرض تفاصيل الطبيب**
   - **الصفحة:** `DoctorProfileFlow` → `DoctorProfilePage.razor`.
   - **المحتوى:**
     - Hero: صورة + اسم + تخصص.
     - Contact Info: عنوان، هاتف، بريد، منطقة.
     - قيمة الكشف (`ExaminationFee`).
     - **بطاقة حجز موعد:**
       - اختيار اليوم.
       - سبب التواصل (فحص جديد / متابعة / تقارير).
       - ملاحظات.
       - زر "إرسال حجز".

5. **إرسال الحجز**
   - **`SendBookingRequest`:**
     - `POST /api/messaging/conversations` (organizationId = ClinicId).
     - `POST /api/messaging/messages` (رسالة الحجز).
   - **محتوى الرسالة:**
     - 🏥 طلب حجز موعد.
     - 📅 اليوم.
     - 🩺 السبب.
     - 📝 ملاحظات.
   - **بعد الإرسال:**
     - `_bookingSent = true` (زر معطل).
     - `OnStartChat.Invoke(conversationId)` — يفتح `ChatPage`.

### 🔌 Endpoints

| Endpoint | Method | الغرض |
|---|---|---|
| `/api/specialities/main` | GET | التخصصات الرئيسية |
| `/api/specialities/sub/{mainId}` | GET | التخصصات الفرعية |
| `/api/organizations/doctors/search?page={p}&pageSize={s}` | GET | البحث عن أطباء |
| `/api/Clinic/{id}` | GET | تفاصيل العيادة |
| `/api/clinic/{id}/messaging-settings` | GET | إعدادات المحادثة |
| `/api/messaging/conversations` | POST | إنشاء محادثة |
| `/api/messaging/messages` | POST | إرسال رسالة |

### 📂 الملفات المتأثرة

- `DoctorSearchFlow.xaml` + `.xaml.cs`
- `DoctorSearchPage.razor` (Shared.UI)
- `DoctorProfileFlow.xaml` + `.xaml.cs`
- `DoctorProfilePage.razor` (Shared.UI)

---

## 💊 PHASE 3: PHARMACY SEARCH + DETAIL

### 📱 الخطوات

1. **فتح البحث عن صيدلية**
   - **من Personal Dashboard:** Quick Action "البحث عن صيدلية".
   - **من Flyout:** "بحث عن صيدلية".
   - **الصفحة:** `PharmacySearchFlow.xaml` → `PharmacySearchPage.razor`.

2. **البحث + الفلترة**
   - **حقل البحث:** `_searchTerm`.
   - **فلتر المدينة:** `_selectedCityId`.
   - **فلتر المنطقة:** `_selectedAreaId`.
   - **زر البحث:** `LoadPharmacies`.

3. **قائمة النتائج**
   - Grid of cards (9 per page).
   - كل صيدلية → `PharmacyDto`:
     - اسم الصيدلية.
     - صورة (Base64).
     - العنوان.
     - الهاتف.
   - **Pagination**.

4. **عرض تفاصيل الصيدلية**
   - **الصفحة:** `PharmacyDetailFlow` → `PharmacyDetailPage.razor`.
   - **المحتوى:**
     - Hero: صورة + اسم + عنوان.
     - تفاصيل: هاتف، بريد، منطقة.
     - **`TokenRedemptionCard`** (إذا `_isPharmacyContracted`).
     - **`MedicationRequestCard`** (طلب دواء).

5. **طلب دواء (إذا لم يكن المريض)**
   - **`MedicationRequestCard`:**
     - البحث عن دواء.
     - اختيار + كمية + ملاحظات.
     - `SubmitRequest` → `POST /api/pharmacy/medication-request`.

### 🔌 Endpoints

| Endpoint | Method | الغرض |
|---|---|---|
| `/api/cities/by-country/1` | GET | المدن |
| `/api/areas/by-city/{cityId}` | GET | المناطق |
| `/api/organizations/pharmacies?page={p}&pageSize={s}` | GET | البحث عن صيدليات |
| `/api/organizations/pharmacy/{id}` | GET | تفاصيل الصيدلية |
| `/api/pharmacy/medication-request` | POST | طلب دواء |

### 📂 الملفات المتأثرة

- `PharmacySearchFlow.xaml` + `.xaml.cs`
- `PharmacySearchPage.razor`
- `PharmacyFilter.razor`
- `PharmacyDetailFlow.xaml` + `.xaml.cs`
- `PharmacyDetailPage.razor`
- `MedicationRequestCard.razor`

---

## 📍 PHASE 4: NEARBY PHARMACIES

### 📱 الخطوات

1. **فتح "أقرب الصيدليات"**
   - **من Flyout:** "أقرب الصيدليات".
   - **الصفحة:** `NearbyPharmaciesPage.xaml` + `.xaml.cs`.

2. **المحتوى**
   - **خريطة** (Map control في MAUI).
   - **قائمة الصيدليات القريبة** (بناءً على GPS).
   - **المسافة** لكل صيدلية.
   - **زر "اتجاهات"** — يفتح Google Maps.

3. **التفاعل**
   - ضغط على صيدلية → `PharmacyDetailFlow` (نفس Phase 3).

### 🔌 Endpoints

| Endpoint | Method | الغرض |
|---|---|---|
| `/api/pharmacies/nearby?lat={lat}&lng={lng}&radius={km}` | GET | الصيدليات القريبة |

### 📂 الملفات المتأثرة

- `NearbyPharmaciesPage.xaml` + `.xaml.cs`

---

## 💬 PHASE 5: MESSAGING

### 📱 الخطوات

1. **فتح Messaging Hub**
   - **من Flyout:** "المحادثات".
   - **من Dashboard:** Quick Action "المحادثات".
   - **الصفحة:** `MessagingHubPage.razor` (Shared.UI).

2. **المحتوى (بعد Phase 5 fixes)**
   - **Tab "Conversations"** (افتراضي).
   - **Tab "Chat"** — يظهر فقط إذا `selectedConversationId > 0`.

3. **عرض قائمة المحادثات**
   - **`ConversationsListPage.razor`:**
     - Load patient conversations (`/my-patient-conversations`).
     - Load org conversations (`/organization/{orgId}` × عدد المنظمات — **متوازي** بعد Phase 5).
     - Union + Order by LastMessageDate.
   - **كل محادثة:**
     - `DisplayIcon` (🏥 / 👤).
     - `DisplayName` (اسم الطرف الآخر).
     - `LastMessage`.
     - `UnreadCount`.

4. **فتح محادثة**
   - `OnConversationSelected((ConversationId, DisplayName))`.
   - `MessagingHubPage` يمرر `InitialDisplayName` إلى `ChatPage`.
   - **`ChatPage.razor`:**
     - Header: `_displayName` (من `InitialDisplayName` أو `DeriveDisplayNameFromMessages`).
     - Messages area.
     - Input area.

5. **إرسال رسالة**
   - **`POST /api/messaging/messages`.**
   - **Payload:**
     ```json
     {
       "ConversationId": 46,
       "SenderProfileId": 15,
       "MessageType": "TEXT",
       "Content": "..."
     }
     ```

6. **حذف رسالة**
   - `DELETE /api/messaging/messages/{id}/for-patient`.
   - **للمريض فقط.**

7. **حذف محادثة**
   - `POST /api/messaging/conversations/{id}/close`.
   - يظهر فقط للمريض.

### 🔌 Endpoints

| Endpoint | Method | الغرض |
|---|---|---|
| `/api/messaging/conversations/my-patient-conversations?page={p}&pageSize={s}` | GET | محادثات المريض |
| `/api/messaging/conversations/organization/{orgId}?page={p}&pageSize={s}` | GET | محادثات المؤسسة |
| `/api/messaging/conversations` | POST | إنشاء محادثة |
| `/api/messaging/messages/conversation/{convId}?page={p}&pageSize={s}` | GET | رسائل المحادثة |
| `/api/messaging/messages` | POST | إرسال رسالة |
| `/api/messaging/messages/{id}/for-patient` | DELETE | حذف رسالة (للمريض) |
| `/api/messaging/conversations/{id}/close` | POST | إغلاق محادثة |

### 📂 الملفات المتأثرة

- `MessagingHubPage.razor`
- `ConversationsListPage.razor`
- `ChatPage.razor`

---

## 🌐 الميزات المشتركة في رحلة المستخدم العام

### 1. Offline Mode
- **يعمل:**
  - الترجمات.
  - جلسة المستخدم.
  - بيانات آخر تحميل (قائمة المحادثات، تفاصيل طبيب/صيدلية).
- **لا يعمل:**
  - البحث الجديد.
  - إرسال رسائل.
  - حجز موعد.

### 2. Central Back Button
- **في `DashboardFlowPage`** — `AppShell.OnBackButtonPressed`.

### 3. Translation + RTL
- **الترجمة:** كاملة (AR/EN).
- **RTL:** تلقائي.

### 4. Notifications
- **Push:**
  - عند رد الطبيب/الصيدلية على الحجز.
- **In-App:**
  - زر الإشعارات.

---

## 🎯 السيناريوهات الشائعة

### سيناريو 1: مستخدم جديد يحجز عند طبيب
```
1. Onboarding (3 شرائح)
2. Register
3. Login → DashboardFlow
4. Doctor Search → فلترة "Cardiology"
5. اختيار طبيب → DoctorProfilePage
6. اختيار اليوم + سبب → إرسال حجز
7. MessagingHub → ChatPage (رسالة الحجز)
8. الطبيب يرد → Push Notification
```

**الوقت المتوقع:** ~3 دقائق.

### سيناريو 2: مستخدم يبحث عن صيدلية
```
1. Login → DashboardFlow
2. Pharmacy Search → فلترة حسب المدينة
3. اختيار صيدلية → PharmacyDetailPage
4. عرض التفاصيل → MedicationRequestCard (إن لم يكن مريض)
```

### سيناريو 3: مستخدم في وضع Offline
- **يفتح التطبيق:** Spinner → Dashboard (من Cache).
- **Banner:** "أنت غير متصل".
- **محاولة بحث:** فشل — "تأكد من الاتصال".

---

## 📊 جدول المراحل (ملخص)

| # | المرحلة | Endpoint | الحالة |
|---|---|---|---|
| 1 | Onboarding | (شاشات) | ✅ |
| 2 | Registration | `POST /api/auth/register` | ✅ |
| 3 | Session Bootstrap | `POST /api/user/session-bootstrap` | ✅ |
| 4 | Doctor Search | `GET /api/organizations/doctors/search` | ✅ |
| 5 | Doctor Profile | `GET /api/Clinic/{id}` | ✅ |
| 6 | Book Appointment | `POST /api/messaging/conversations` + `messages` | ✅ |
| 7 | Pharmacy Search | `GET /api/organizations/pharmacies` | ✅ |
| 8 | Pharmacy Detail | `GET /api/organizations/pharmacy/{id}` | ✅ |
| 9 | Nearby Pharmacies | `GET /api/pharmacies/nearby` | ✅ |
| 10 | Messaging | `GET/POST /api/messaging/*` | ✅ |

---

## 📂 الملفات المتأثرة (كاملة)

### Mobile (MAUI)
- `DoctorSearchFlow.xaml` + `.xaml.cs`
- `DoctorProfileFlow.xaml` + `.xaml.cs`
- `PharmacySearchFlow.xaml` + `.xaml.cs`
- `PharmacyDetailFlow.xaml` + `.xaml.cs`
- `NearbyPharmaciesPage.xaml` + `.xaml.cs`
- `MessagingHubFlow.xaml` + `.xaml.cs`
- `ChatFlow.xaml` + `.xaml.cs`

### Shared.UI
- `DoctorSearchPage.razor`
- `DoctorProfilePage.razor`
- `PharmacySearchPage.razor`
- `PharmacyFilter.razor`
- `PharmacyDetailPage.razor`
- `MedicationRequestCard.razor`
- `TokenRedemptionCard.razor`
- `MessagingHubPage.razor`
- `ConversationsListPage.razor`
- `ChatPage.razor`

### Backend (للمرجعية)
- `DoctorController.cs`
- `PharmacyController.cs`
- `MessagingController.cs`

---

## 🔗 روابط سريعة

- **[00-overview.md](00-overview.md)**
- **[01-patient.md](01-patient.md)**
- **[06-cross-cutting.md](06-cross-cutting.md)**

© 2026 RubikCare — للاستخدام الداخلي
```

---
