# 09 — API GUIDE v3.0 (Developer-First Edition)

**آخر تحديث:** 25 نوفمبر 2026
**الإصدار:** 3.0 (Developer-First Edition)
**الحالة:** ✅ مرجع شامل — موثّق من الكود الفعلي
**الجمهور:** فريق التطوير، تكامل العملاء (Web, PWA, Mobile)، AI Assistants

---

## 📌 مقدمة

هذا المرجع هو **الدليل الكامل لجميع APIs** في منصة RubikCare، مع تركيز على **كيفية الاستخدام** (وليس فقط الجرد).

### 🎯 ما الجديد في v3.0؟

1. **توثيق ذكي** — كل Endpoint له: وصف، مصادقة، مدخلات (Path/Query/Body)، استجابة ناجحة، أخطاء محتملة.
2. **إضافة 17 Endpoint** لم تكن موثقة من قبل (ClinicReferral 4 + Prescription 8 + PharmacyOrders 4 + PharmacyPsp 1).
3. **حذف التفاصيل الزائدة** — إزالة التكرار، وإزالة قائمة `AllowAnonymous` (نُقلت إلى `00B` كـ TD-C4).
4. **الأرقام الفعلية** — 45+ Controller، ~337 Endpoint (بعد التصحيح).

### 📊 الأرقام الفعلية (25 نوفمبر 2026)

| المقياس | القيمة |
|---|---|
| عدد Controllers | **45+** |
| عدد Endpoints | **~337** |
| عدد المجموعات | 20 |

### 🧭 كيف تقرأ هذه الوثيقة

- **§0 المنهجية** — اقرأها أولاً لفهم الهيكل الموحد.
- **§1 الفهرس** — للتنقل السريع.
- **§2–§20 المجموعات** — كل مجموعة لها Controllers وEndpoints.
- **§21 جدول ملخص** — للمراجعة السريعة.

---

## 📖 §0: المنهجية (كيف نوثّق كل Endpoint)

كل Endpoint في هذه الوثيقة يتبع الهيكل التالي:

### 🏷️ [اسم Endpoint]
**`METHOD`** `Route`

[وصف مختصر — سطر أو سطران]

**المصادقة:** [مطلوبة (JWT) / اختيارية / مجهولة]

**المدخلات:**

| النوع | الحقل | النوع | مطلوب | الوصف |
|---|---|---|---|---|
| Path | `id` | `int` | نعم | معرف المورد |
| Query | `page` | `int` | لا | رقم الصفحة (افتراضي 1) |
| Body | `name` | `string` | نعم | اسم المورد |

**الاستجابة الناجحة (`200 OK`):**
```json
{
  "success": true,
  "data": { ... }
}
```

**أخطاء محتملة:**

| الكود | السبب |
|---|---|
| `400` | بيانات غير صالحة |
| `401` | توكن مفقود أو منتهي |
| `404` | المورد غير موجود |

---

## 📚 §1: الفهرس (Table of Contents)

| # | المجموعة | Controllers | Endpoints | § |
|---|---|---|---|---|
| 2 | Auth & Identity | 2 | 20 | §2 |
| 3 | User & Profile | 2 | 10 | §3 |
| 4 | Organizations | 4 | 13 | §4 |
| 5 | Clinical & Geo | 5 | 11 | §5 |
| 6 | Messaging | 2 | 16 | §6 |
| 7 | Notifications | 1 | 9 | §7 |
| 8 | Clinic | 1 | 15 | §8 |
| 9 | Professional | 2 | 4 | §9 |
| 10 | Patient | 2 | 13 | §10 |
| 11 | Pharmacy | 5 | 32 | §11 |
| 12 | PSP | 15 | 68 | §12 |
| 13 | Points | 6 | 38 | §13 |
| 14 | Rep | 1 | 12 | §14 |
| 15 | Reports | 1 | 8 | §15 |
| 16 | Dashboard & Misc | 5 | 7 | §16 |
| 17 | **ClinicReferral** ⭐ | 1 | 4 | §17 |
| 18 | **Prescription** ⭐ | 1 | 8 | §18 |
| 19 | **PharmacyOrders** ⭐ | 1 | 4 | §19 |
| 20 | **PharmacyPsp** ⭐ | 1 | 1 | §20 |
| 21 | **Commerce** | 6 | 28 | §21 |
| 22 | جدول ملخص | — | — | §22 |

⭐ = جديد في v3.0

---

## 🏥 §17: ClinicReferral (جديد — 4 Endpoints)

**المسار:** `Api.Web/Controllers/Clinic/ClinicReferralController.cs`
**Route:** `[Route("api/clinic/referrals")]`
**الغرض:** كود إحالة ثابت (Clinic+Program) — بديل عن توليد QR جديد لكل مريض.

### 🏷️ 17.1 توليد أو جلب كود الإحالة الثابت
**`POST`** `/api/clinic/referrals/generate`

يولّد كود إحالة ثابت لعيادة-برنامج، أو يعيد الكود الموجود.

**المصادقة:** مطلوبة (JWT) — الطبيب/العيادة.

**المدخلات (Body):**

| الحقل | النوع | مطلوب | الوصف |
|---|---|---|---|
| `ClinicId` | `int` | نعم | معرف العيادة |
| `ProgramId` | `int` | نعم | معرف برنامج الدعم |

**الاستجابة الناجحة (`200 OK`):**
```json
{
  "success": true,
  "data": {
    "referralId": 1,
    "referralCode": "REF-A1B2C3",
    "qrData": "https://rubikcare.com/join?code=REF-A1B2C3",
    "clinicId": 5,
    "programId": 2,
    "isActive": true,
    "usageCount": 0,
    "createdDate": "2026-11-25T10:00:00Z"
  }
}
```

**أخطاء محتملة:**

| الكود | السبب |
|---|---|
| `400` | العيادة/البرنامج غير موجود |
| `401` | توكن مفقود |

---

### 🏷️ 17.2 جلب جميع أكواد العيادة
**`GET`** `/api/clinic/referrals/clinic/{clinicId}`

يعيد كل أكواد الإحالة الخاصة بعيادة.

**المصادقة:** مطلوبة (JWT).

**المدخلات (Path):**

| الحقل | النوع | مطلوب | الوصف |
|---|---|---|---|
| `clinicId` | `int` | نعم | معرف العيادة |

**الاستجابة الناجحة (`200 OK`):**
```json
{
  "success": true,
  "data": [
    {
      "referralId": 1,
      "referralCode": "REF-A1B2C3",
      "programId": 2,
      "programName": "برنامج دعم السكري",
      "isActive": true,
      "usageCount": 45,
      "lastUsedDate": "2026-11-24T15:30:00Z"
    }
  ]
}
```

---

### 🏷️ 17.3 مسح الكود (من قبل المريض)
**`GET`** `/api/clinic/referrals/scan?code={code}`

يتحقق من كود الإحالة ويعيد تفاصيل البرنامج والعيادة (قبل الاشتراك).

**المصادقة:** **مجهولة** (`[AllowAnonymous]`) — يمكن للزائر مسح الكود.

**المدخلات (Query):**

| الحقل | النوع | مطلوب | الوصف |
|---|---|---|---|
| `code` | `string` | نعم | كود الإحالة |

**الاستجابة الناجحة (`200 OK`):**
```json
{
  "success": true,
  "data": {
    "isValid": true,
    "clinicName": "عيادة النور",
    "programName": "برنامج دعم السكري",
    "programDescription": "...",
    "companyName": "شركة ديفارت لاب",
    "patientAlreadyEnrolled": false
  }
}
```

**أخطاء محتملة:**

| الكود | السبب |
|---|---|
| `200` مع `isValid: false` | كود غير صالح أو منتهي |
| `500` | خطأ داخلي |

---

### 🏷️ 17.4 الاشتراك عبر الكود (من قبل المريض)
**`POST`** `/api/clinic/referrals/enroll`

يسجّل المريض في البرنامج عبر كود الإحالة (ينشئ `PSPPatient` + `PSPeRX` + `PSPDispensation` تلقائياً).

**المصادقة:** مطلوبة (JWT) — المريض.

**المدخلات (Body):**

| الحقل | النوع | مطلوب | الوصف |
|---|---|---|---|
| `ReferralCode` | `string` | نعم | كود الإحالة |

**الاستجابة الناجحة (`200 OK`):**
```json
{
  "success": true,
  "message": "تم الاشتراك بنجاح",
  "data": {
    "caseId": 1083,
    "programId": 2,
    "tokenCode": "RC-20261125-0001",
    "erxCode": "PSP-ERX-20261125-ABCD"
  }
}
```

**أخطاء محتملة:**

| الكود | السبب |
|---|---|
| `400` | كود غير صالح / المريض مشترك بالفعل |
| `401` | توكن مفقود |

---

## 📋 §18: Prescription (جديد — 8 Endpoints)

**المسار:** `Api.Web/Controllers/Clinic/PrescriptionController.cs`
**Route:** `[Route("api/clinic/prescriptions")]`
**الغرض:** وصفة طبية متعددة الأدوية (عادية + مدعومة) في سجل واحد.

### 🏷️ 18.1 إنشاء وصفة طبية جديدة
**`POST`** `/api/clinic/prescriptions`

ينشئ وصفة متعددة الأدوية. الأدوية المدعومة تُنشئ `PSPeRXItem` + `PrescriptionItem` مرتبط.

**المصادقة:** مطلوبة (JWT) — الطبيب.

**المدخلات (Body):**

| الحقل | النوع | مطلوب | الوصف |
|---|---|---|---|
| `CaseID` | `int` | نعم | معرف الحالة (المريض) |
| `ClinicID` | `int` | نعم | معرف العيادة |
| `ProgramID` | `int?` | لا | معرف برنامج الدعم (إن وُجد) |
| `Notes` | `string` | لا | ملاحظات |
| `Items` | `array` | نعم | قائمة الأدوية |
| `Items[].MedicationID` | `int` | نعم | معرف الدواء |
| `Items[].IsSupported` | `bool` | نعم | هل الدواء مدعوم؟ |
| `Items[].ProgramMedicationID` | `int?` | لا | معرف دواء البرنامج (إن كان مدعوماً) |
| `Items[].Dosage` | `string` | نعم | الجرعة |
| `Items[].Frequency` | `string` | نعم | التكرار |
| `Items[].Quantity` | `int` | نعم | الكمية |
| `Items[].OverrideTimesPerDay` | `int?` | لا | تعديل عدد المرات/يوم |
| `Items[].OverrideQuantityPerDispense` | `int?` | لا | تعديل الكمية/صرف |

**الاستجابة الناجحة (`201 Created`):**
```json
{
  "prescriptionId": 123,
  "prescriptionCode": "RX-20261125-0001",
  "status": "Active",
  "createdDate": "2026-11-25T10:00:00Z"
}
```

**أخطاء محتملة:**

| الكود | السبب |
|---|---|
| `400` | بيانات غير صالحة / دواء غير موجود |
| `401` | توكن مفقود |

---

### 🏷️ 18.2 جلب وصفة بالمعرف
**`GET`** `/api/clinic/prescriptions/{id}`

يعيد تفاصيل وصفة (رأس + بنود).

**المصادقة:** مطلوبة (JWT).

**الاستجابة الناجحة (`200 OK`):**
```json
{
  "prescriptionId": 123,
  "prescriptionCode": "RX-20261125-0001",
  "caseId": 1083,
  "doctorId": "abc123",
  "clinicId": 5,
  "prescriptionDate": "2026-11-25T10:00:00Z",
  "status": "Active",
  "items": [
    {
      "prescriptionItemId": 456,
      "medicationId": 5,
      "medicationName": "Oxy Free",
      "dosage": "500mg",
      "frequency": "3 مرات يومياً",
      "quantity": 20,
      "isSupported": true,
      "linkedPspERxItemId": 789
    }
  ]
}
```

---

### 🏷️ 18.3 جلب وصفة بالكود (للمشاركة)
**`GET`** `/api/clinic/prescriptions/code/{code}`

يُستخدم للمشاركة (واتساب/QR) — الصيدلي أو المريض يفتح الوصفة بالكود.

**المصادقة:** **مجهولة** (`[AllowAnonymous]`).

**الاستجابة الناجحة (`200 OK`):** (نفس 18.2)

**أخطاء محتملة:**

| الكود | السبب |
|---|---|
| `404` | وصفة غير موجودة / كود غير صحيح |

---

### 🏷️ 18.4 جلب وصفات مريض معين
**`GET`** `/api/clinic/prescriptions/patient/{caseId}?page=1&pageSize=20`

**المصادقة:** مطلوبة (JWT).

**الاستجابة الناجحة (`200 OK`):**
```json
{
  "items": [ { ... } ],
  "totalCount": 5,
  "page": 1,
  "pageSize": 20
}
```

---

### 🏷️ 18.5 جلب وصفات طبيب معين في عيادة معينة
**`GET`** `/api/clinic/prescriptions/doctor/{doctorId}/clinic/{clinicId}?page=1&pageSize=20`

**المصادقة:** مطلوبة (JWT).

---

### 🏷️ 18.6 جلب الأدوية المتاحة في برنامج دعم
**`GET`** `/api/clinic/prescriptions/program/{programId}/medications?clinicId={clinicId}`

يعيد قائمة `AvailableProgramMedicationDto` — الأدوية المدعومة في البرنامج (مع تفاصيل الصرف).

**المصادقة:** مطلوبة (JWT) — الطبيب.

**الاستجابة الناجحة (`200 OK`):**
```json
[
  {
    "programMedicationId": 789,
    "medicationId": 5,
    "medicationName": "Oxy Free",
    "dosage": "500mg",
    "timesPerDay": 3,
    "quantityPerDispense": 20,
    "discountPercentage": 20,
    "canOverride": true
  }
]
```

---

### 🏷️ 18.7 إلغاء وصفة طبية
**`PUT`** `/api/clinic/prescriptions/{id}/cancel`

**المصادقة:** مطلوبة (JWT) — الطبيب الذي أنشأها فقط.

**الاستجابة الناجحة (`200 OK`):**
```json
{ "message": "تم إلغاء الوصفة بنجاح" }
```

**أخطاء محتملة:**

| الكود | السبب |
|---|---|
| `400` | وصفة غير موجودة / لا تملك صلاحية |

---

### 🏷️ 18.8 توليد بيانات المشاركة (واتساب / QR)
**`GET`** `/api/clinic/prescriptions/{id}/share`

يعيد `PrescriptionShareDto` — رابط المشاركة + بيانات QR.

**المصادقة:** مطلوبة (JWT).

**الاستجابة الناجحة (`200 OK`):**
```json
{
  "shareUrl": "https://rubikcare.com/rx/RX-20261125-0001",
  "whatsAppUrl": "https://wa.me/?text=...",
  "qrData": "RX-20261125-0001"
}
```

---

## 🏥 §19: PharmacyOrders (جديد — 4 Endpoints)

**المسار:** `Api.Web/Controllers/Pharmacy/PharmacyOrdersController.cs`
**Route:** `[Route("api/pharmacy/orders")]`
**الغرض:** قائمة موحدة لطلبات الصيدلية (Regular + PSP + Legacy) + إدارة الحالة.

### 🏷️ 19.1 جلب الطلبات الموحدة
**`GET`** `/api/pharmacy/orders/unified?pharmacyId={id}&status={status}&orderType={type}&page=1&pageSize=20`

**المصادقة:** مطلوبة (JWT).

**المدخلات (Query):**

| الحقل | النوع | مطلوب | الوصف |
|---|---|---|---|
| `pharmacyId` | `int` | نعم | معرف الصيدلية |
| `status` | `string` | لا | `ACTIVE` (افتراضي) / `COMPLETED` / `ALL` / قيمة محددة |
| `orderType` | `string` | لا | `ALL` (افتراضي) / `REGULAR_RETAIL` / `PSP_DISPENSATION` |
| `page` | `int` | لا | افتراضي 1 |
| `pageSize` | `int` | لا | افتراضي 20 |

**الاستجابة الناجحة (`200 OK`):**
```json
{
  "orders": [
    {
      "id": 52,
      "orderNumber": "ORD-20261125-0001",
      "orderType": "REGULAR_RETAIL",
      "patientName": "أحمد محمد",
      "medicationName": "بانادول",
      "quantity": 2,
      "totalAmount": 45.50,
      "status": "SUBMITTED",
      "requestDate": "2026-11-25T10:00:00Z",
      "items": [ { ... } ]
    }
  ],
  "totalCount": 15,
  "totalPages": 1,
  "currentPage": 1,
  "pageSize": 20
}
```

---

### 🏷️ 19.2 تحديث حالة الطلب
**`PUT`** `/api/pharmacy/orders/{id}/status?pharmacyId={id}&orderType={type}`

**المصادقة:** مطلوبة (JWT).

**المدخلات (Body):**

| الحقل | النوع | مطلوب | الوصف |
|---|---|---|---|
| `Status` | `string` | نعم | الحالة الجديدة |
| `Notes` | `string?` | لا | ملاحظات |

**ملاحظة:** التحقق من صحة تتابع الحالات (State Machine) — لا يمكن الانتقال من `DRAFT` إلى `DELIVERED` مباشرة.

**الاستجابة الناجحة (`200 OK`):**
```json
{ "success": true, "message": "تم تحديث حالة الطلب بنجاح" }
```

---

### 🏷️ 19.3 معالجة الطلب (موافقة جزئية / تعديل كميات)
**`PUT`** `/api/pharmacy/orders/{id}/fulfill`

**المصادقة:** مطلوبة (JWT).

**المدخلات (Body):**

| الحقل | النوع | مطلوب | الوصف |
|---|---|---|---|
| `Items` | `array` | نعم | قائمة البنود |
| `Items[].ItemReferenceID` | `int` | نعم | معرف الدواء |
| `Items[].IsApproved` | `bool` | نعم | هل تمت الموافقة؟ |
| `Items[].ApprovedQuantity` | `int` | نعم | الكمية الموافق عليها |
| `Items[].FinalUnitPrice` | `decimal?` | لا | السعر النهائي (إن عُدّل) |
| `Items[].RejectionReason` | `string?` | لا | سبب الرفض |

**الاستجابة الناجحة (`200 OK`):**
```json
{ "success": true, "message": "تم تحديث حالة الطلب والأصناف بنجاح" }
```

---

### 🏷️ 19.4 جلب تفاصيل طلب واحد
**`GET`** `/api/pharmacy/orders/{id}?pharmacyId={id}`

**المصادقة:** مطلوبة (JWT).

**الاستجابة الناجحة (`200 OK`):** (نفس شكل 19.1 لعنصر واحد)

---

## 🏥 §20: PharmacyPsp (جديد — 1 Endpoint)

**المسار:** `Api.Web/Controllers/Pharmacy/PharmacyPspController.cs`
**Route:** `[Route("api/pharmacy-psp")]`
**الغرض:** البرامج المشتركة بين المريض والصيدلية.

### 🏷️ 20.1 جلب البرامج المشتركة
**`GET`** `/api/pharmacy-psp/matching-programs/{pharmacyId}`

يعيد البرامج التي يشترك فيها **كل من** المريض الحالي والصيدلية.

**المصادقة:** مطلوبة (JWT) — المريض.

**الاستجابة الناجحة (`200 OK`):**
```json
{
  "hasMatchingPrograms": true,
  "matchingPrograms": [
    {
      "programId": 2,
      "programName": "برنامج دعم السكري"
    }
  ]
}
```

**أخطاء محتملة:**

| الكود | السبب |
|---|---|
| `401` | توكن مفقود |

---

## 📊 §22: جدول ملخص (Summary)

| # | المجموعة | Controllers | Endpoints | الحالة |
|---|---|---|---|---|
| 2 | Auth & Identity | 2 | 20 | ✅ |
| 3 | User & Profile | 2 | 10 | ✅ |
| 4 | Organizations | 4 | 13 | ✅ |
| 5 | Clinical & Geo | 5 | 11 | ✅ |
| 6 | Messaging | 2 | 16 | 🟡 (يحتاج صيانة) |
| 7 | Notifications | 1 | 9 | 🟡 (S09 نشط) |
| 8 | Clinic | 1 | 15 | ✅ |
| 9 | Professional | 2 | 4 | ✅ |
| 10 | Patient | 2 | 13 | ✅ |
| 11 | Pharmacy | 5 | 32 | ✅ |
| 12 | PSP | 15 | 68 | ✅ |
| 13 | Points | 6 | 38 | ✅ |
| 14 | Rep | 1 | 12 | ✅ |
| 15 | Reports | 1 | 8 | ✅ |
| 16 | Dashboard & Misc | 5 | 7 | ✅ |
| **17** | **ClinicReferral** ⭐ | **1** | **4** | ✅ |
| **18** | **Prescription** ⭐ | **1** | **8** | ✅ |
| **19** | **PharmacyOrders** ⭐ | **1** | **4** | ✅ |
| **20** | **PharmacyPsp** ⭐ | **1** | **1** | ✅ |
| 21 | Commerce | 6 | 28 | ✅ |
| **المجموع** | | **~45+** | **~337** | |

---

## 🔗 روابط ذات صلة

- `00-MASTER-INDEX.md` — الفهرس الشامل
- `00B-PROJECT-BACKLOG.md` — الحالة الحية
- `01-ARCHITECTURE.md` — الهيكل المعماري
- `05-psp-system.md` — نظام PSP
- `23-COMMERCE-ENGINE.md` — محرك التجارة

---

## 📝 سجل التغييرات

| الإصدار | التاريخ | التغييرات |
|---|---|---|
| 1.0 | 17 مايو 2026 | الإصدار الأولي |
| 2.0 | 16 أكتوبر 2026 | شامل — ~290 endpoint |
| 2.1 | 12 نوفمبر 2026 | إضافة Commerce (28 Endpoint) |
| **3.0** | **25 نوفمبر 2026** | **إعادة كتابة كاملة (Developer-First Edition): إضافة 17 Endpoint (ClinicReferral 4 + Prescription 8 + PharmacyOrders 4 + PharmacyPsp 1)؛ توثيق ذكي لكل Endpoint (Input/Output/Auth/Errors)؛ حذف التكرار؛ نقل `AllowAnonymous` إلى `00B TD-C4`؛ تحديث الأرقام إلى ~337 Endpoint.** |

---

© 2026 RubikCare — للاستخدام الداخلي

**نهاية الجزء 1** ✅


## 🛒 §21: Commerce (28 Endpoints)

**المسار:** `Api.Web/Controllers/Commerce/`
**Route:** `[Route("api/commerce/...")]`
**الغرض:** محرك التجارة B2B (طلبات التوريد، الـ Ledger، الحملات، البونص، الباقات).

### 📁 21.1 CatalogController (3 Endpoints)

#### 🏷️ 21.1.1 الشركات النشطة
**`GET`** `/api/commerce/catalog/companies`

يعيد الشركات النشطة في B2B (لها اشتراك نشط).

**المصادقة:** مطلوبة (JWT).

**الاستجابة الناجحة (`200 OK`):**
```json
{
  "success": true,
  "data": [
    {
      "companyOrgId": 91,
      "companyName": "شركة ديفارت لاب",
      "medicationCount": 45
    }
  ]
}
```

---

#### 🏷️ 21.1.2 كتالوج أدوية شركة
**`GET`** `/api/commerce/catalog/{companyOrgId}`

**المصادقة:** مطلوبة (JWT).

**الاستجابة الناجحة (`200 OK`):**
```json
{
  "success": true,
  "data": [
    {
      "pharmaCompanyMedicationId": 11465,
      "medicationName": "اوكسي فري 20 كبسولة",
      "publicPrice": 63.84,
      "salesTax": 14,
      "pharmacistDiscount": 20,
      "additionalDiscount": 5,
      "netUnitPrice": 48.52
    }
  ]
}
```

---

#### 🏷️ 21.1.3 بحث في الكتالوج
**`GET`** `/api/commerce/catalog/{companyOrgId}/search?term={term}`

**المصادقة:** مطلوبة (JWT).

**أخطاء محتملة:**

| الكود | السبب |
|---|---|
| `400` | `term` فارغ |

---

### 📁 21.2 SupplyOrdersController (11 Endpoints)

#### 🏷️ 21.2.1 تفاصيل طلب
**`GET`** `/api/commerce/supply-orders/{id}`

**المصادقة:** مطلوبة (JWT) — الصيدلية أو الشركة.

**الاستجابة الناجحة (`200 OK`):**
```json
{
  "success": true,
  "data": {
    "supplyOrderId": 1001,
    "serviceOrderId": 5001,
    "orderNumber": "ORD-20261125-0001",
    "status": "CONFIRMED",
    "pharmacyOrgId": 1258,
    "companyOrgId": 91,
    "subTotal": 4650.00,
    "taxAmount": 0,
    "totalAmount": 2553.60,
    "items": [ { ... } ]
  }
}
```

---

#### 🏷️ 21.2.2 طلبات الصيدلية
**`GET`** `/api/commerce/supply-orders/pharmacy/{pharmacyOrgId}?status={status}`

**المصادقة:** مطلوبة (JWT).

---

#### 🏷️ 21.2.3 طلبات الشركة
**`GET`** `/api/commerce/supply-orders/company/{companyOrgId}?status={status}`

**المصادقة:** مطلوبة (JWT).

---

#### 🏷️ 21.2.4 إنشاء طلب (DRAFT)
**`POST`** `/api/commerce/supply-orders`

**المصادقة:** مطلوبة (JWT) — الصيدلية.

**المدخلات (Body):**

| الحقل | النوع | مطلوب | الوصف |
|---|---|---|---|
| `PharmacyOrgId` | `int` | نعم | معرف الصيدلية |
| `CompanyOrgId` | `int` | نعم | معرف الشركة |
| `Items` | `array` | نعم | قائمة الأدوية |
| `Items[].PharmaCompanyMedicationID` | `int` | نعم | معرف الدواء |
| `Items[].Quantity` | `int` | نعم | الكمية |
| `Notes` | `string?` | لا | ملاحظات |

**الاستجابة الناجحة (`200 OK`):** (نفس 21.2.1)

**ملاحظة:** يحسب البونص تلقائياً (Greedy).

---

#### 🏷️ 21.2.5 إرسال (DRAFT → SUBMITTED)
**`POST`** `/api/commerce/supply-orders/{id}/submit`

**المصادقة:** مطلوبة (JWT) — الصيدلية.

---

#### 🏷️ 21.2.6 تأكيد (SUBMITTED → CONFIRMED)
**`POST`** `/api/commerce/supply-orders/{id}/confirm`

**المصادقة:** مطلوبة (JWT) — الشركة.

**ملاحظة:** يتحقق من المخزون، ويحجز الكميات (`ReservedQuantity`).

---

#### 🏷️ 21.2.7 رفض (SUBMITTED → REJECTED)
**`POST`** `/api/commerce/supply-orders/{id}/reject`

**المدخلات (Body):** `{ "Reason": "string?" }`

**المصادقة:** مطلوبة (JWT) — الشركة.

---

#### 🏷️ 21.2.8 تجهيز (CONFIRMED → PREPARING)
**`POST`** `/api/commerce/supply-orders/{id}/prepare`

**المصادقة:** مطلوبة (JWT) — الشركة.

---

#### 🏷️ 21.2.9 شحن (PREPARING → SHIPPED)
**`POST`** `/api/commerce/supply-orders/{id}/ship`

**المصادقة:** مطلوبة (JWT) — الشركة.

---

#### 🏷️ 21.2.10 تأكيد استلام (SHIPPED → DELIVERED)
**`POST`** `/api/commerce/supply-orders/{id}/deliver`

**المصادقة:** مطلوبة (JWT) — الصيدلية.

**ملاحظة:** يخصم المخزون (`AvailableQuantity`)، ويُحدّث الـ Ledger (إن كان مفعّلاً — GAP-01).

---

#### 🏷️ 21.2.11 إلغاء
**`POST`** `/api/commerce/supply-orders/{id}/cancel`

**المدخلات (Body):** `{ "Reason": "string?" }`

**المصادقة:** مطلوبة (JWT) — الصيدلية أو الشركة.

---

### 📁 21.3 LedgerController (3 Endpoints)

> ⚠️ **ملاحظة:** `Ledger` موجود كـ API لكن **غير مفعّل وظيفياً** (GAP-01 — لا يُستدعى من `SupplyOrderService`). التوثيق أدناه للـ API فقط.

#### 🏷️ 21.3.1 أرصدة طرف مع كل الأطراف
**`GET`** `/api/commerce/ledger/{partyType}/{partyId}`

**المصادقة:** مطلوبة (JWT).

**الاستجابة الناجحة (`200 OK`):**
```json
{
  "success": true,
  "data": [
    {
      "serviceLedgerId": 1,
      "partyAType": "ORGANIZATION",
      "partyAId": 1258,
      "partyBType": "ORGANIZATION",
      "partyBId": 91,
      "balance": 0,
      "currencyCode": "EGP"
    }
  ]
}
```

---

#### 🏷️ 21.3.2 رصيد محدد بين طرفين
**`GET`** `/api/commerce/ledger/{partyAType}/{partyAId}/{partyBType}/{partyBId}`

**المصادقة:** مطلوبة (JWT).

---

#### 🏷️ 21.3.3 التأكد من وجود Ledger
**`POST`** `/api/commerce/ledger/{partyAType}/{partyAId}/{partyBType}/{partyBId}/ensure?currency=EGP`

**المصادقة:** مطلوبة (JWT).

**الاستجابة الناجحة (`200 OK`):**
```json
{ "success": true, "serviceLedgerId": 1 }
```

---

### 📁 21.4 CampaignsController (5 Endpoints)

#### 🏷️ 21.4.1 حملات الشركة
**`GET`** `/api/commerce/campaigns/company/{companyOrgId}`

**المصادقة:** مطلوبة (JWT).

---

#### 🏷️ 21.4.2 تفاصيل حملة
**`GET`** `/api/commerce/campaigns/{id}`

**المصادقة:** مطلوبة (JWT).

---

#### 🏷️ 21.4.3 إنشاء حملة
**`POST`** `/api/commerce/campaigns`

**المدخلات (Body):**

| الحقل | النوع | مطلوب |
|---|---|---|
| `CompanyOrgId` | `int` | نعم |
| `CampaignName` | `string` | نعم |
| `Description` | `string?` | لا |
| `StartDate` | `DateTime` | نعم |
| `EndDate` | `DateTime` | نعم |
| `DiscountType` | `string` | نعم (`PERCENTAGE` / `FIXED`) |
| `DiscountValue` | `decimal` | نعم |

**المصادقة:** مطلوبة (JWT).

---

#### 🏷️ 21.4.4 تعديل حملة
**`PUT`** `/api/commerce/campaigns/{id}`

**المصادقة:** مطلوبة (JWT).

---

#### 🏷️ 21.4.5 حذف حملة
**`DELETE`** `/api/commerce/campaigns/{id}`

**المصادقة:** مطلوبة (JWT).

---

### 📁 21.5 BonusesController (3 Endpoints)

#### 🏷️ 21.5.1 بونصات الشركة
**`GET`** `/api/commerce/bonuses/company/{companyOrgId}`

**المصادقة:** مطلوبة (JWT).

**الاستجابة الناجحة (`200 OK`):**
```json
{
  "success": true,
  "data": [
    {
      "bonusId": 1,
      "pharmaCompanyMedicationId": 13487,
      "bonusName": "Bonus for ارثينيا كريم",
      "tiers": [
        { "minQuantity": 10, "bonusQuantity": 1 },
        { "minQuantity": 20, "bonusQuantity": 3 }
      ]
    }
  ]
}
```

---

#### 🏷️ 21.5.2 إنشاء/تحديث بونص (مع Tiers)
**`POST`** `/api/commerce/bonuses`

**المدخلات (Body):**

| الحقل | النوع | مطلوب |
|---|---|---|
| `CompanyOrgId` | `int` | نعم |
| `PharmaCompanyMedicationId` | `int` | نعم |
| `BonusName` | `string` | نعم |
| `StartDate` | `DateTime` | نعم |
| `EndDate` | `DateTime` | نعم |
| `Tiers` | `array` | نعم |
| `Tiers[].MinQuantity` | `int` | نعم |
| `Tiers[].BonusQuantity` | `int` | نعم |

**المصادقة:** مطلوبة (JWT).

**ملاحظة:** إذا كان هناك بونص نشط، يضيف شرائح جديدة (لا ينشئ بونصاً جديداً).

---

#### 🏷️ 21.5.3 حذف بونص
**`DELETE`** `/api/commerce/bonuses/{id}`

**المصادقة:** مطلوبة (JWT).

---

### 📁 21.6 BundlesController (3 Endpoints)

#### 🏷️ 21.6.1 باقات الشركة
**`GET`** `/api/commerce/bundles/company/{companyOrgId}`

**المصادقة:** مطلوبة (JWT).

---

#### 🏷️ 21.6.2 إنشاء باقة
**`POST`** `/api/commerce/bundles`

**المدخلات (Body):**

| الحقل | النوع | مطلوب |
|---|---|---|
| `CompanyOrgId` | `int` | نعم |
| `BundleName` | `string` | نعم |
| `Description` | `string?` | لا |
| `BundlePrice` | `decimal` | نعم |
| `Items` | `array` | نعم |
| `Items[].PharmaCompanyMedicationId` | `int` | نعم |
| `Items[].Quantity` | `int` | نعم |

**المصادقة:** مطلوبة (JWT).

---

#### 🏷️ 21.6.3 حذف باقة
**`DELETE`** `/api/commerce/bundles/{id}`

**المصادقة:** مطلوبة (JWT).

---

## 🎯 §13: Points (38 Endpoints)

**المسار:** `Api.Web/Controllers/PSP/Points/`
**Route:** `[Route("api/psp/points/...")]`
**الغرض:** نظام النقاط (Schemes, Rules, Balances, Earning, Redeem, Lookups).

### 📁 13.1 PointSchemesController (13 Endpoints)

#### 🏷️ 13.1.1 إنشاء Scheme
**`POST`** `/api/psp/points/schemes`

**المدخلات (Body):**

| الحقل | النوع | مطلوب |
|---|---|---|
| `CampaignID` | `int` | نعم |
| `PointSystemTypeID` | `int` | نعم (1=PSP, 2=B2B) |
| `Name` | `string` | نعم |

**المصادقة:** مطلوبة (JWT) — الشركة.

---

#### 🏷️ 13.1.2 تحديث Scheme
**`PUT`** `/api/psp/points/schemes/{schemeId}`

**المصادقة:** مطلوبة (JWT).

---

#### 🏷️ 13.1.3 Scheme واحد (PSP)
**`GET`** `/api/psp/points/schemes/by-program/{programId}`

**المصادقة:** مطلوبة (JWT).

---

#### 🏷️ 13.1.4 تعطيل Scheme
**`POST`** `/api/psp/points/schemes/{schemeId}/deactivate`

**المصادقة:** مطلوبة (JWT).

---

#### 🏷️ 13.1.5 قائمة Schemes
**`GET`** `/api/psp/points/schemes/list-by-program/{programId}`

**المصادقة:** مطلوبة (JWT).

---

#### 🏷️ 13.1.6 نظرة عامة
**`GET`** `/api/psp/points/schemes/programs/{programId}/overview`

**المصادقة:** مطلوبة (JWT).

---

#### 🏷️ 13.1.7 عيادات البرنامج
**`GET`** `/api/psp/points/schemes/programs/{programId}/clinics`

**المصادقة:** مطلوبة (JWT).

---

#### 🏷️ 13.1.8 حسب المنظمة
**`GET`** `/api/psp/points/schemes/by-organization/{organizationId}`

**المصادقة:** مطلوبة (JWT).

---

#### 🏷️ 13.1.9 تفاصيل Scheme
**`GET`** `/api/psp/points/schemes/{schemeId:int}`

**المصادقة:** مطلوبة (JWT).

---

#### 🏷️ 13.1.10 عيادات Scheme
**`GET`** `/api/psp/points/schemes/{schemeId:int}/clinics`

**المصادقة:** مطلوبة (JWT).

---

#### 🏷️ 13.1.11 أنواع المستفيدين
**`GET`** `/api/psp/points/schemes/{schemeId:int}/beneficiary-types`

**المصادقة:** مطلوبة (JWT).

---

#### 🏷️ 13.1.12 إضافة نوع مستفيد
**`POST`** `/api/psp/points/schemes/{schemeId:int}/beneficiary-types`

**المدخلات (Body):** `{ "BeneficiaryTypeID": "int" }`

**المصادقة:** مطلوبة (JWT).

---

#### 🏷️ 13.1.13 حذف نوع مستفيد
**`DELETE`** `/api/psp/points/schemes/{schemeId:int}/beneficiary-types/{id:int}`

**المصادقة:** مطلوبة (JWT).

---

### 📁 13.2 PointRulesController (5 Endpoints)

| # | Method | Route | الوصف |
|---|---|---|---|
| 13.2.1 | `POST` | `/api/psp/points/rules` | إنشاء قاعدة |
| 13.2.2 | `PUT` | `/api/psp/points/rules/{ruleId}` | تحديث |
| 13.2.3 | `DELETE` | `/api/psp/points/rules/{ruleId}` | حذف |
| 13.2.4 | `GET` | `/api/psp/points/rules/by-scheme/{schemeId}` | قواعد Scheme |
| 13.2.5 | `POST` | `/api/psp/points/rules/{ruleId}/toggle` | تفعيل/تعطيل |

**المصادقة:** كلها مطلوبة (JWT).

---

### 📁 13.3 PointBalancesController (7 Endpoints)

| # | Method | Route | الوصف |
|---|---|---|---|
| 13.3.1 | `GET` | `/api/psp/points/balances/by-organization` | حسب المنظمة |
| 13.3.2 | `GET` | `/api/psp/points/balances/by-scheme/{schemeId}` | حسب Scheme |
| 13.3.3 | `GET` | `/api/psp/points/balances/organization-summary/{organizationId}` | ملخص |
| 13.3.4 | `PUT` | `/api/psp/points/balances/visibility` | إعدادات الظهور |
| 13.3.5 | `GET` | `/api/psp/points/balances/visibility/check` | فحص |
| 13.3.6 | `PUT` | `/api/psp/points/balances/visibility/all/{schemeId}` | الكل |
| 13.3.7 | `GET` | `/api/psp/points/balances/by-company/{companyOrganizationId:int}` | أرصدة الشركة |

**المصادقة:** كلها مطلوبة (JWT).

---

### 📁 13.4 PointEarningController (1 Endpoint)

#### 🏷️ 13.4.1 سجل المعاملات
**`GET`** `/api/psp/points/transactions`

**المصادقة:** مطلوبة (JWT).

**الاستجابة الناجحة (`200 OK`):**
```json
{
  "success": true,
  "data": [
    {
      "transactionId": 1,
      "transactionTypeId": 1,
      "points": 5,
      "balanceBefore": 10,
      "balanceAfter": 15,
      "referenceType": "DISPENSATION",
      "referenceId": 42,
      "createdDate": "2026-11-25T10:00:00Z"
    }
  ]
}
```

---

### 📁 13.5 PointRedeemController (8 Endpoints)

| # | Method | Route | الوصف | من؟ |
|---|---|---|---|---|
| 13.5.1 | `POST` | `/api/psp/points/redeem-requests` | إنشاء طلب | الطبيب |
| 13.5.2 | `POST` | `/api/psp/points/redeem-requests/{id}/accept` | قبول | الشركة |
| 13.5.3 | `POST` | `/api/psp/points/redeem-requests/{id}/reject` | رفض | الشركة |
| 13.5.4 | `POST` | `/api/psp/points/redeem-requests/{id}/confirm` | تأكيد | الطبيب |
| 13.5.5 | `POST` | `/api/psp/points/redeem-requests/{id}/dispute` | اعتراض | الطبيب |
| 13.5.6 | `POST` | `/api/psp/points/redeem-requests/{id}/resolve` | حل نزاع | — |
| 13.5.7 | `GET` | `/api/psp/points/redeem-requests` | قائمة | — |
| 13.5.8 | `GET` | `/api/psp/points/redeem-requests/{id}` | تفاصيل | — |

**المصادقة:** كلها مطلوبة (JWT).

---

### 📁 13.6 PointLookupsController (4 Endpoints)

| # | Method | Route | الوصف |
|---|---|---|---|
| 13.6.1 | `GET` | `/api/psp/points/lookups/organization-types` | أنواع المنظمات |
| 13.6.2 | `GET` | `/api/psp/points/lookups/beneficiary-types` | أنواع المستفيدين |
| 13.6.3 | `GET` | `/api/psp/points/lookups/activity-types` | أنواع الأنشطة |
| 13.6.4 | `GET` | `/api/psp/points/lookups/programs` | برامج الشركة |

**المصادقة:** كلها مطلوبة (JWT).

**ملاحظة:** `organization-types` و`beneficiary-types` يدعمان فلترة حسب `pointSystemTypeID`.

---

## 💚 §12: PSP (62 Endpoints)

**المسار:** `Api.Web/Controllers/PSP/`
**Route:** `[Route("api/psp/...")]`
**الغرض:** نظام دعم المرضى (برامج، دعوات، اشتراكات، خطط صرف، تحاليل).

> **ملاحظة:** `PspController` مقسم إلى 12 ملف. الـ 38 Endpoint الخاصة بالـ Tests موثقة في جدول مختصر (لأنها متشابهة البنية).

### 📁 12.1 PspController.Programs.cs (4 Endpoints)

| # | Method | Route | الوصف |
|---|---|---|---|
| 12.1.1 | `GET` | `/api/psp/programs` | قائمة البرامج (Pagination) |
| 12.1.2 | `GET` | `/api/psp/programs-temp` | ⚠️ مؤقت (للاختبار) |
| 12.1.3 | `GET` | `/api/psp/programs/{id}` | تفاصيل برنامج |
| 12.1.4 | `GET` | `/api/psp/program-details-temp/{id}` | ⚠️ مؤقت |

**المصادقة:** 12.1.1 و12.1.3 مطلوبة (JWT). 12.1.2 و12.1.4 مجهولة (⚠️ مؤقتة).

---

### 📁 12.2 PspController.Invitations.cs (2 Endpoints)

#### 🏷️ 12.2.1 إنشاء دعوة مريض (Doctor → Patient)
**`POST`** `/api/psp/create-invitation`

**المدخلات (Body):**

| الحقل | النوع | مطلوب |
|---|---|---|
| `ProgramID` | `int` | نعم |
| `PatientProfileID` | `int` | نعم |

**المصادقة:** مطلوبة (JWT) — الطبيب.

**الاستجابة الناجحة (`200 OK`):**
```json
{
  "success": true,
  "invitationToken": "INV-XXXXXXXX",
  "expiryDate": "2026-12-25T10:00:00Z"
}
```

---

#### 🏷️ 12.2.2 قبول دعوة المريض
**`POST`** `/api/psp/accept-invitation`

**المدخلات (Body):** `{ "InvitationCode": "INV-XXXXXXXX" }`

**المصادقة:** مطلوبة (JWT).

---

### 📁 12.3 PspController.RepInvitations.cs (2 Endpoints)

| # | Method | Route | الوصف |
|---|---|---|---|
| 12.3.1 | `GET` | `/api/psp/invitation/by-token?token=X` | البحث عن دعوة بالكود |
| 12.3.2 | `POST` | `/api/psp/accept-rep-invitation` | قبول دعوة المندوب |

**المصادقة:** مطلوبة (JWT).

---

### 📁 12.4 PspController.Enrollment.cs (2 Endpoints)

| # | Method | Route | الوصف |
|---|---|---|---|
| 12.4.1 | `POST` | `/api/psp/participate` | اشتراك/إعادة اشتراك |
| 12.4.2 | `POST` | `/api/psp/unsubscribe` | إلغاء اشتراك |

**المصادقة:** مطلوبة (JWT).

---

### 📁 12.5 PspController.Unsubscribe.cs (1 Endpoint)

| # | Method | Route | الوصف |
|---|---|---|---|
| 12.5.1 | `POST` | `/api/psp/unsubscribe` | إلغاء اشتراك (UseCase) |

**المصادقة:** مطلوبة (JWT).

**⚠️ ملاحظة:** مكرر مع 12.4.2.

---

### 📁 12.6 PspController.Entry.cs (1 Endpoint)

#### 🏷️ 12.6.1 نقطة دخول المريض
**`POST`** `/api/psp/entry`

**المدخلات (Body):** `{ "invitationCode": "INV-XXXXXXXX" }`

**المصادقة:** مطلوبة (JWT).

**الاستجابة الناجحة (`200 OK`):**
```json
{
  "status": "NEW_ENROLLMENT",
  "message": "تم الاشتراك بنجاح",
  "tokenCode": "RC-20261125-0001"
}
```

**الحالات المحتملة:** `NEW_ENROLLMENT`, `ACTIVE_PROGRAMS`, `ALREADY_ENROLLED`, `CODE_INVALID`, `CODE_EXPIRED`, `CODE_USED`, `NO_CODE`, `UNAUTHORIZED`.

---

### 📁 12.7 PspController.DispensationPlans.cs (6 Endpoints)

| # | Method | Route | الوصف |
|---|---|---|---|
| 12.7.1 | `GET` | `/api/psp/plans/{planId}/phases` | مراحل خطة |
| 12.7.2 | `GET` | `/api/psp/phases/{phaseId}` | مرحلة واحدة |
| 12.7.3 | `POST` | `/api/psp/plans/{planId}/phases` | إضافة مرحلة |
| 12.7.4 | `PUT` | `/api/psp/phases/{phaseId}` | تعديل مرحلة |
| 12.7.5 | `DELETE` | `/api/psp/phases/{phaseId}` | حذف مرحلة |
| 12.7.6 | `GET` | `/api/psp/plans/{planId}/available-medications` | أدوية متاحة |

**المصادقة:** مطلوبة (JWT).

---

### 📁 12.8 PspController.Links.cs (3 Endpoints)

| # | Method | Route | الوصف |
|---|---|---|---|
| 12.8.1 | `POST` | `/api/psp/links` | إضافة رابط |
| 12.8.2 | `PUT` | `/api/psp/links/{id}` | تحديث |
| 12.8.3 | `DELETE` | `/api/psp/links/{id}` | حذف |

**المصادقة:** مطلوبة (JWT).

**ملاحظة:** `MaxLinksPerProgram = 6`.

---

### 📁 12.9 PspController.patientdetails.cs (1 Endpoint)

#### 🏷️ 12.9.1 تفاصيل برنامج المريض
**`GET`** `/api/psp/patient-details?patientId={patientId}&participationId={participationId}`

**المصادقة:** مطلوبة (JWT).

**⚠️ ملاحظة:** `PharmaciesCount` + `NearestPharmacy` + `PharmacyDistance` — Hardcoded (تحتاج ربط بـ `NearbyPharmacyService`).

**⚠️ BLOCK-1:** 403 Forbidden لو `patientId=15` (PatientProfileID) بدل `CaseID` — يحتاج إصلاح.

---

### 📁 12.10 PspController.Tests.cs (38 Endpoints) — جدول مختصر

**8 مجموعات فرعية:**

| المجموعة | Endpoints | مثال |
|---|---|---|
| LabTest (مرجعية) | 1 | `GET /api/psp/lab-tests` |
| RequiredTest | 5 | `POST /api/psp/required-tests` |
| TestResult | 8 | `POST /api/psp/test-results` |
| Attachment | 5 | `POST /api/psp/test-results/{id}/attachments` |
| PharmaView (Privacy) | 1 | `GET /api/psp/test-results/pharma-view` |
| Reminder | 3 | `POST /api/psp/reminders` |
| PhaseRequiredTest | 3 | `POST /api/psp/phase-required-tests` |
| Patient Mobile | 5 | `GET /api/psp/test-results/mobile` |

**المصادقة:** مطلوبة (JWT) — ما عدا `pharma-view` (له منطق خاص).

**RequiredTiming (5 قيم):**
1. `Before` — قبل البدء
2. `During` — أثناء
3. `After N days` — بعد N يوم
4. `Periodic` — كل N يوم
5. `OnEvent` — عند حدث (⭐ جديد)

**EditWindow:**
- `Mandatory` → 24 ساعة
- غير `Mandatory` → 168 ساعة (7 أيام)

**Consent:** موافقة المريض مطلوبة لكل رفع.

**Privacy في PharmaView:** لا اسم، لا هاتف — فقط `PatientAnonymousId`.

---

### 📁 12.11 PspController.Helpers.cs (0 Endpoints)

**Helpers:**
- `GetCurrentUserId()` — من claim `userProfileId`
- `GenerateTokenCode()` — `RC-yyyyMMdd-NNNN`
- `GenerateERxCode()` — `PSP-ERX-yyyyMMdd-XXXXXX`
- `GenerateDispensationCode()` — `DISP-yyyyMMdd-XXXXXX`
- `GenerateInvitationToken()` — `INV-XXXXXXXX`
- `GetUserActivePrograms()` — `[NonAction]`
- `EnrollUserInProgram()` — `[NonAction]`

---

### 📁 12.12 RemindersController (8 Endpoints)

| # | Method | Route | الوصف |
|---|---|---|---|
| 12.12.1 | `POST` | `/api/psp/reminders` | إنشاء تذكير |
| 12.12.2 | `GET` | `/api/psp/reminders` | التذكيرات |
| 12.12.3 | `GET` | `/api/psp/reminders/{id}` | تذكير واحد |
| 12.12.4 | `PATCH` | `/api/psp/reminders/{id}/sent` | تعليم كـ Sent |
| 12.12.5 | `POST` | `/api/psp/reminders/{id}/take` | المريض ضغط "تناول" |
| 12.12.6 | `PATCH` | `/api/psp/reminders/{id}/complete` | إكمال |
| 12.12.7 | `PATCH` | `/api/psp/reminders/{id}/cancel` | إلغاء |
| 12.12.8 | `POST` | `/api/psp/reminders/{id}/repeat` | تكرار |

**المصادقة:** مطلوبة (JWT).

---

### 📁 12.13 DispensationPlansController (5 Endpoints)

| # | Method | Route | الوصف |
|---|---|---|---|
| 12.13.1 | `GET` | `/api/dispensationplans/program/{programId}` | خطط برنامج |
| 12.13.2 | `GET` | `/api/dispensationplans/{planId}` | تفاصيل خطة |
| 12.13.3 | `POST` | `/api/dispensationplans` | إنشاء خطة |
| 12.13.4 | `PUT` | `/api/dispensationplans/{planId}` | تحديث |
| 12.13.5 | `DELETE` | `/api/dispensationplans/{planId}` | حذف |

**المصادقة:** مطلوبة (JWT) — ما عدا 12.13.1 و12.13.2 (مجهولة — ⚠️ TD-C4).

---

### 📁 12.14 PharmaCompanyMedicationsController (18 Endpoints)

**Route:** `/api/pharmacompanymedications`

| # | Method | Route | الوصف |
|---|---|---|---|
| 12.14.1 | `GET` | `/api/pharmacompanymedications` | قائمة (Pagination) |
| 12.14.2 | `GET` | `/api/pharmacompanymedications/{id}` | تفاصيل |
| 12.14.3 | `POST` | `/api/pharmacompanymedications` | إنشاء |
| 12.14.4 | `PUT` | `/api/pharmacompanymedications/{id}` | تحديث |
| 12.14.5 | `DELETE` | `/api/pharmacompanymedications/{id}` | حذف (Soft) |
| 12.14.6 | `POST` | `/api/pharmacompanymedications/{id}/image/medication` | رفع صورة الدواء |
| 12.14.7 | `POST` | `/api/pharmacompanymedications/{id}/image/package` | رفع صورة العبوة |
| 12.14.8 | `GET` | `/api/pharmacompanymedications/search` | بحث |
| 12.14.9 | `GET` | `/api/pharmacompanymedications/by-category/{categoryId}` | حسب الفئة |
| 12.14.10 | `GET` | `/api/pharmacompanymedications/by-speciality/{specialityId}` | حسب التخصص |
| 12.14.11 | `GET` | `/api/pharmacompanymedications/{id}/specialities` | تخصصات الدواء |
| 12.14.12 | `POST` | `/api/pharmacompanymedications/{id}/specialities/{specialityId}` | إضافة تخصص |
| 12.14.13 | `DELETE` | `/api/pharmacompanymedications/{id}/specialities/{specialityId}` | حذف تخصص |
| 12.14.14 | `PUT` | `/api/pharmacompanymedications/{id}/specialities` | تحديث التخصصات |
| 12.14.15 | `POST` | `/api/pharmacompanymedications/{id}/approve` | موافقة |
| 12.14.16 | `POST` | `/api/pharmacompanymedications/{id}/reject` | رفض |
| 12.14.17 | `PATCH` | `/api/pharmacompanymedications/{id}/toggle-active` | تفعيل/تعطيل |
| 12.14.18 | `GET` | `/api/pharmacompanymedications/statistics/{companyId}` | إحصائيات |

**المصادقة:** مطلوبة (JWT) — ما عدا 12.14.1 و12.14.8 (مجهولة — ⚠️ TD-C4).

---

## 📊 §22: جدول ملخص (Summary) — محدّث

| # | المجموعة | Controllers | Endpoints | الحالة |
|---|---|---|---|---|
| 2 | Auth & Identity | 2 | 20 | ✅ |
| 3 | User & Profile | 2 | 10 | ✅ |
| 4 | Organizations | 4 | 13 | ✅ |
| 5 | Clinical & Geo | 5 | 11 | ✅ |
| 6 | Messaging | 2 | 16 | 🟡 |
| 7 | Notifications | 1 | 9 | 🟡 (S09) |
| 8 | Clinic | 1 | 15 | ✅ |
| 9 | Professional | 2 | 4 | ✅ |
| 10 | Patient | 2 | 13 | ✅ |
| 11 | Pharmacy | 5 | 32 | ✅ |
| 12 | PSP | 15 | 68 | ✅ |
| 13 | Points | 6 | 38 | ✅ |
| 14 | Rep | 1 | 12 | ✅ |
| 15 | Reports | 1 | 8 | ✅ |
| 16 | Dashboard & Misc | 5 | 7 | ✅ |
| 17 | ClinicReferral ⭐ | 1 | 4 | ✅ |
| 18 | Prescription ⭐ | 1 | 8 | ✅ |
| 19 | PharmacyOrders ⭐ | 1 | 4 | ✅ |
| 20 | PharmacyPsp ⭐ | 1 | 1 | ✅ |
| 21 | Commerce | 6 | 28 | ✅ |
| **المجموع** | | **~45+** | **~337** | |

---

© 2026 RubikCare — للاستخدام الداخلي

**نهاية الجزء 2*


## 🔐 §2: Auth & Identity (20 Endpoints)

**المسار:** `Api.Web/Controllers/`
**Route:** `[Route("api/auth")]`

### 📁 2.1 AuthController (13 Endpoints)

#### 🏷️ 2.1.1 تسجيل الدخول التقليدي
**`POST`** `/api/auth/login`

**المدخلات (Body):**
```json
{ "email": "user@example.com", "password": "Password123!" }
```

**الاستجابة الناجحة (`200 OK`):**
```json
{
  "success": true,
  "token": "eyJhbGciOiJIUzI1NiIs...",
  "user": {
    "id": "abc123",
    "email": "user@example.com",
    "displayName": "أحمد محمد",
    "userProfileId": 12,
    "roles": ["Doctor"]
  }
}
```

**أخطاء محتملة:**

| الكود | السبب |
|---|---|
| `400` | بريد/كلمة مرور غير صحيحة |
| `401` | الحساب غير مفعّل |

---

#### 🏷️ 2.1.2 تسجيل مستخدم جديد
**`POST`** `/api/auth/register`

**المدخلات (Body):** `{ "email", "password", "firstName", "lastName", "phoneNumber" }`

**الاستجابة الناجحة (`200 OK`):** `{ "success": true, "userId": "...", "requiresConfirmation": true }`

---

#### 🏷️ 2.1.3 تسجيل عبر Google
**`POST`** `/api/auth/google-login`

**المدخلات (Body):** `{ "idToken": "..." }`

**الاستجابة الناجحة (`200 OK`):** (نفس 2.1.1)

---

#### 🏷️ 2.1.4 تسجيل الخروج
**`POST`** `/api/auth/logout`

**المصادقة:** مطلوبة (JWT).

**الاستجابة الناجحة (`200 OK`):** `{ "success": true }`

---

#### 🏷️ 2.1.5 تغيير كلمة المرور
**`POST`** `/api/auth/change-password`

**المدخلات (Body):** `{ "currentPassword", "newPassword" }`

**المصادقة:** مطلوبة (JWT).

---

#### 🏷️ 2.1.6 نسيت كلمة المرور
**`POST`** `/api/auth/forgot-password`

**المدخلات (Body):** `{ "email" }`

---

#### 🏷️ 2.1.7 إعادة تعيين كلمة المرور
**`POST`** `/api/auth/reset-password`

**المدخلات (Body):** `{ "email", "token", "newPassword" }`

---

#### 🏷️ 2.1.8 تأكيد البريد
**`POST`** `/api/auth/confirm-email`

**المدخلات (Body):** `{ "userId", "token" }`

---

#### 🏷️ 2.1.9 إعادة إرسال التأكيد
**`POST`** `/api/auth/resend-confirmation`

**المدخلات (Body):** `{ "email" }`

---

#### 🏷️ 2.1.10 تجديد JWT
**`POST`** `/api/auth/refresh-token`

**المدخلات (Body):** `{ "refreshToken" }`

---

#### 🏷️ 2.1.11 المستخدم الحالي
**`GET`** `/api/auth/me`

**المصادقة:** مطلوبة (JWT).

**الاستجابة الناجحة (`200 OK`):** (نفس 2.1.1)

---

#### 🏷️ 2.1.12 إكمال الملف الشخصي
**`POST`** `/api/auth/complete-profile`

**المدخلات (Body):** `{ "firstName", "lastName", "phoneNumber", ... }`

**المصادقة:** مطلوبة (JWT).

---

#### 🏷️ 2.1.13 التحقق من الرمز
**`POST`** `/api/auth/verify-code`

**المدخلات (Body):** `{ "email", "code" }`

---

### 📁 2.2 Secure Cross-App SSO (2 Endpoints)

#### 🏷️ 2.2.1 توليد رمز الانتقال (Handoff Token)
**`POST`** `/api/auth/generate-handoff-token`

**المصادقة:** مطلوبة (JWT / Session).

**الاستجابة الناجحة (`200 OK`):**
```json
{ "success": true, "handoffToken": "aB3xY9...zQ1" }
```

**ملاحظة:** الرمز يُحفظ في `IMemoryCache` لمدة **60 ثانية**، مرتبط بـ `UserId` و`UserProfileId`.

---

#### 🏷️ 2.2.2 استبدال رمز الانتقال بـ JWT
**`POST`** `/api/auth/exchange-handoff-token`

**المصادقة:** **مجهولة** (`[AllowAnonymous]`).

**المدخلات (Body):** `{ "handoffToken" }`

**الاستجابة الناجحة (`200 OK`):** (نفس 2.1.1)

**ملاحظة:** الرمز يُحذف فوراً بعد الاستخدام (One-Time Use).

---

## 👤 §3: User & Profile (10 Endpoints)

### 📁 3.1 UserController (7 Endpoints)

| # | Method | Route | الوصف |
|---|---|---|---|
| 3.1.1 | `GET` | `/api/user/profile` | جلب الملف الشخصي |
| 3.1.2 | `PUT` | `/api/user/profile` | تحديث الملف الشخصي |
| 3.1.3 | `POST` | `/api/user/upload-picture` | رفع صورة البروفايل |
| 3.1.4 | `GET` | `/api/user/session-bootstrap` | تهيئة الجلسة (Mobile) |
| 3.1.5 | `GET` | `/api/user/memberships` | عضويات المستخدم |
| 3.1.6 | `GET` | `/api/user/preferences` | تفضيلات المستخدم |
| 3.1.7 | `PUT` | `/api/user/preferences` | تحديث التفضيلات |

**المصادقة:** كلها مطلوبة (JWT).

---

### 📁 3.2 ProfessionalController (3 Endpoints) — موثّق في §9

---

## 👥 §4: Organizations (13 Endpoints)

### 📁 4.1 OrganizationsController (3 Endpoints)

| # | Method | Route | الوصف |
|---|---|---|---|
| 4.1.1 | `GET` | `/api/organizations/{id}` | جلب منظمة |
| 4.1.2 | `POST` | `/api/organizations` | إنشاء منظمة |
| 4.1.3 | `PUT` | `/api/organizations/{id}` | تحديث منظمة |

**المصادقة:** مطلوبة (JWT).

---

### 📁 4.2 OrganizationMembersController (3 Endpoints)

| # | Method | Route | الوصف |
|---|---|---|---|
| 4.2.1 | `GET` | `/api/organizations/{orgId}/members` | أعضاء المنظمة |
| 4.2.2 | `POST` | `/api/organizations/{orgId}/members` | إضافة عضو |
| 4.2.3 | `DELETE` | `/api/organizations/{orgId}/members/{memberId}` | حذف عضو |

**المصادقة:** مطلوبة (JWT).

---

### 📁 4.3 OrganizationJobTitlesController (2 Endpoints)

| # | Method | Route | الوصف |
|---|---|---|---|
| 4.3.1 | `GET` | `/api/organizations/{orgId}/job-titles` | المسميات الوظيفية |
| 4.3.2 | `POST` | `/api/organizations/{orgId}/job-titles` | إضافة مسمى |

---

### 📁 4.4 OrganizationMemberTitlesController (5 Endpoints)

| # | Method | Route | الوصف |
|---|---|---|---|
| 4.4.1 | `GET` | `/api/organizations/{orgId}/member-titles` | ألقاب الأعضاء |
| 4.4.2 | `POST` | `/api/organizations/{orgId}/member-titles` | إضافة لقب |
| 4.4.3 | `PUT` | `/api/organizations/{orgId}/member-titles/{id}` | تحديث |
| 4.4.4 | `DELETE` | `/api/organizations/{orgId}/member-titles/{id}` | حذف |
| 4.4.5 | `POST` | `/api/organizations/{orgId}/member-titles/assign` | تعيين لقب لعضو |

---

## 🏥 §5: Clinical & Geo (11 Endpoints)

### 📁 5.1 OrganizationTypesController (1 Endpoint)

| # | Method | Route | الوصف |
|---|---|---|---|
| 5.1.1 | `GET` | `/api/clinical/organization-types` | أنواع المنظمات |

**ملاحظة:** `OrganizationTypeID`: 1=Personal, 2=PharmaCompany, 3=Pharmacy, 4=Clinic.

---

### 📁 5.2 SpecialitiesController (4 Endpoints)

| # | Method | Route | الوصف |
|---|---|---|---|
| 5.2.1 | `GET` | `/api/clinical/specialities` | التخصصات |
| 5.2.2 | `GET` | `/api/clinical/specialities/{id}` | تخصص واحد |
| 5.2.3 | `POST` | `/api/clinical/specialities` | إضافة تخصص |
| 5.2.4 | `DELETE` | `/api/clinical/specialities/{id}` | حذف تخصص |

---

### 📁 5.3 CountriesController (1 Endpoint)

| # | Method | Route | الوصف |
|---|---|---|---|
| 5.3.1 | `GET` | `/api/geo/countries` | قائمة الدول |

---

### 📁 5.4 CitiesController (3 Endpoints)

| # | Method | Route | الوصف |
|---|---|---|---|
| 5.4.1 | `GET` | `/api/geo/cities` | كل المدن |
| 5.4.2 | `GET` | `/api/geo/countries/{countryId}/cities` | مدن دولة |
| 5.4.3 | `GET` | `/api/geo/cities/{id}` | مدينة واحدة |

---

### 📁 5.5 AreasController (2 Endpoints)

| # | Method | Route | الوصف |
|---|---|---|---|
| 5.5.1 | `GET` | `/api/geo/cities/{cityId}/areas` | مناطق مدينة |
| 5.5.2 | `GET` | `/api/geo/areas/{id}` | منطقة واحدة |

---

## 💬 §6: Messaging (16 Endpoints)

> ⚠️ **ملاحظة:** `Messaging` يحتاج صيانة (GAP-11) — خلط بين الرسائل الشخصية والمنظمات.

### 📁 6.1 ConversationsController (7 Endpoints)

| # | Method | Route | الوصف |
|---|---|---|---|
| 6.1.1 | `GET` | `/api/messaging/conversations` | قائمة المحادثات |
| 6.1.2 | `POST` | `/api/messaging/conversations` | إنشاء محادثة |
| 6.1.3 | `GET` | `/api/messaging/conversations/{id}` | تفاصيل محادثة |
| 6.1.4 | `PUT` | `/api/messaging/conversations/{id}` | تحديث محادثة |
| 6.1.5 | `DELETE` | `/api/messaging/conversations/{id}` | حذف محادثة |
| 6.1.6 | `POST` | `/api/messaging/conversations/{id}/read` | تعليم كمقروءة |
| 6.1.7 | `POST` | `/api/messaging/conversations/{id}/archive` | أرشفة |

**المصادقة:** كلها مطلوبة (JWT).

---

### 📁 6.2 MessagesController (9 Endpoints)

| # | Method | Route | الوصف |
|---|---|---|---|
| 6.2.1 | `GET` | `/api/messaging/conversations/{convId}/messages` | رسائل محادثة |
| 6.2.2 | `POST` | `/api/messaging/conversations/{convId}/messages` | إرسال رسالة |
| 6.2.3 | `GET` | `/api/messaging/messages/{id}` | رسالة واحدة |
| 6.2.4 | `PUT` | `/api/messaging/messages/{id}` | تعديل رسالة |
| 6.2.5 | `DELETE` | `/api/messaging/messages/{id}` | حذف رسالة |
| 6.2.6 | `POST` | `/api/messaging/messages/{id}/read` | تعليم كمقروءة |
| 6.2.7 | `POST` | `/api/messaging/messages/{id}/attachment` | رفع مرفق |
| 6.2.8 | `GET` | `/api/messaging/messages/unread-count` | عدد غير المقروءة |
| 6.2.9 | `POST` | `/api/messaging/messages/mark-all-read` | تعليم الكل كمقروء |

---

## 🔔 §7: Notifications (9 Endpoints)

**المسار:** `Api.Web/Controllers/NotificationsController.cs`

| # | Method | Route | الوصف |
|---|---|---|---|
| 7.1 | `GET` | `/api/notifications` | قائمة الإشعارات |
| 7.2 | `GET` | `/api/notifications/unread-count` | عدد غير المقروءة |
| 7.3 | `POST` | `/api/notifications/{id}/read` | تعليم كمقروء |
| 7.4 | `POST` | `/api/notifications/mark-all-read` | تعليم الكل كمقروء |
| 7.5 | `DELETE` | `/api/notifications/{id}` | حذف إشعار |
| 7.6 | `POST` | `/api/notifications/register-device` | تسجيل FCM Token |
| 7.7 | `POST` | `/api/notifications/register-web-push` | تسجيل Web Push Subscription |
| 7.8 | `POST` | `/api/notifications/unregister-device` | إلغاء تسجيل FCM |
| 7.9 | `POST` | `/api/notifications/send-test` | إرسال إشعار تجريبي (Dev) |

**المصادقة:** مطلوبة (JWT) — ما عدا 7.9 (Dev فقط).

**ملاحظة:** `NH-1→NH-20` (S08) + `S09` (Notification Hub — Phase 2) — قيد التطوير.

---

## 🏥 §8: Clinic (15 Endpoints)

**المسار:** `Api.Web/Controllers/ClinicController.cs`

| # | Method | Route | الوصف |
|---|---|---|---|
| 8.1 | `GET` | `/api/clinic/dashboard` | لوحة تحكم العيادة |
| 8.2 | `GET` | `/api/clinic/patients` | مرضى العيادة |
| 8.3 | `GET` | `/api/clinic/patients/{id}` | تفاصيل مريض |
| 8.4 | `POST` | `/api/clinic/patients` | إضافة مريض |
| 8.5 | `PUT` | `/api/clinic/patients/{id}` | تحديث مريض |
| 8.6 | `DELETE` | `/api/clinic/patients/{id}` | حذف مريض |
| 8.7 | `GET` | `/api/clinic/invitations` | دعوات العيادة |
| 8.8 | `POST` | `/api/clinic/invitations` | إنشاء دعوة |
| 8.9 | `POST` | `/api/clinic/invitations/{id}/resend` | إعادة إرسال |
| 8.10 | `GET` | `/api/clinic/psp-programs` | برامج PSP للعيادة |
| 8.11 | `GET` | `/api/clinic/psp-participations` | مشاركات العيادة |
| 8.12 | `GET` | `/api/clinic/schedule` | جدول العيادة |
| 8.13 | `POST` | `/api/clinic/schedule` | إضافة موعد |
| 8.14 | `PUT` | `/api/clinic/schedule/{id}` | تحديث موعد |
| 8.15 | `DELETE` | `/api/clinic/schedule/{id}` | حذف موعد |

**المصادقة:** كلها مطلوبة (JWT) — الطبيب/العيادة.

---

## 👨‍⚕️ §9: Professional (4 Endpoints)

### 📁 9.1 ProfessionalController (3 Endpoints)

| # | Method | Route | الوصف |
|---|---|---|---|
| 9.1.1 | `GET` | `/api/professional/status` | حالة الترخيص |
| 9.1.2 | `POST` | `/api/professional/submit-license` | تقديم طلب ترخيص |
| 9.1.3 | `POST` | `/api/professional/upload-document` | رفع مستند |

**المصادقة:** مطلوبة (JWT).

---

### 📁 9.2 DoctorInvitationsController (1 Endpoint)

| # | Method | Route | الوصف |
|---|---|---|---|
| 9.2.1 | `GET` | `/api/doctor-invitations/pending` | دعوات معلقة |

**المصادقة:** مطلوبة (JWT).

---

## 👤 §10: Patient (13 Endpoints)

### 📁 10.1 PatientMedicationsController (10 Endpoints)

| # | Method | Route | الوصف |
|---|---|---|---|
| 10.1.1 | `GET` | `/api/patient/medications` | أدوية المريض |
| 10.1.2 | `GET` | `/api/patient/medications/{id}` | دواء واحد |
| 10.1.3 | `POST` | `/api/patient/medications` | إضافة دواء |
| 10.1.4 | `PUT` | `/api/patient/medications/{id}` | تحديث دواء |
| 10.1.5 | `DELETE` | `/api/patient/medications/{id}` | حذف دواء |
| 10.1.6 | `GET` | `/api/patient/schedules` | جدول الجرعات |
| 10.1.7 | `POST` | `/api/patient/schedules` | إضافة موعد جرعة |
| 10.1.8 | `PUT` | `/api/patient/schedules/{id}/status` | تحديث حالة الجرعة |
| 10.1.9 | `GET` | `/api/patient/reminders` | التذكيرات |
| 10.1.10 | `POST` | `/api/patient/reminders/sync` | مزامنة التذكيرات |

**المصادقة:** مطلوبة (JWT).

---

### 📁 10.2 PatientOrdersController (3 Endpoints)

| # | Method | Route | الوصف |
|---|---|---|---|
| 10.2.1 | `GET` | `/api/patient/orders` | طلبات المريض |
| 10.2.2 | `GET` | `/api/patient/orders/{id}` | تفاصيل طلب |
| 10.2.3 | `POST` | `/api/patient/orders/{id}/confirm-receipt` | تأكيد الاستلام |

**المصادقة:** مطلوبة (JWT).

**ملاحظة:** `MedicationRequests` — `PatientId` (FK to `UserProfile`).

---

## 💊 §11: Pharmacy (32 Endpoints)

### 📁 11.1 PharmacyController (7 Endpoints)

| # | Method | Route | الوصف |
|---|---|---|---|
| 11.1.1 | `GET` | `/api/pharmacy/dashboard/stats` | إحصائيات اللوحة |
| 11.1.2 | `GET` | `/api/pharmacy/reports` | تقارير الصيدلية |
| 11.1.3 | `GET` | `/api/pharmacy/nearby` | الصيدليات القريبة (GPS) |
| 11.1.4 | `PUT` | `/api/pharmacy/update-location` | تحديث موقع الصيدلية |
| 11.1.5 | `GET` | `/api/pharmacy/details/{id}` | تفاصيل صيدلية |
| 11.1.6 | `GET` | `/api/pharmacy/{pharmacyId}/programs` | برامج الصيدلية |
| 11.1.7 | `GET` | `/api/pharmacy/medication-requests` | طلبات الدواء |

**المصادقة:** 11.1.3 مجهولة (`[AllowAnonymous]`)، الباقي مطلوب (JWT).

---

### 📁 11.2 DispenseController (14 Endpoints)

| # | Method | Route | الوصف | Auth |
|---|---|---|---|---|
| 11.2.1 | `POST` | `/api/dispense/validate-token` | التحقق من رمز الصرف | مجهول |
| 11.2.2 | `POST` | `/api/dispense/confirm` | تأكيد الصرف (+ نقاط) | مجهول |
| 11.2.3 | `GET` | `/api/dispense/history` | سجل الصرف | JWT |
| 11.2.4 | `POST` | `/api/dispense/send-to-pharmacy` | إرسال توكن لصيدلية | JWT |
| 11.2.5 | `POST` | `/api/dispense/confirm-receipt` | تأكيد الاستلام | JWT |
| 11.2.6 | `POST` | `/api/dispense/generate-next-token` | توليد توكن جديد | JWT |
| 11.2.7 | `POST` | `/api/dispense/submit-for-approval` | المريض يرسل للصيدلي | JWT |
| 11.2.8 | `POST` | `/api/dispense/approve` | الصيدلي يوافق | JWT |
| 11.2.9 | `POST` | `/api/dispense/scan-and-dispense` | مسح QR → صرف (+ نقاط) | JWT |
| 11.2.10 | `GET` | `/api/dispense/pending-approvals` | طلبات معلقة | JWT |
| 11.2.11 | `GET` | `/api/dispense/status/{id}` | حالة الصرف | JWT |
| 11.2.12 | `GET` | `/api/dispense/status-by-token/{tokenCode}` | حالة بالتوكن | مجهول |
| 11.2.13 | `GET` | `/api/dispense/my-psp-orders` | طلبات PSP للمريض | JWT |
| 11.2.14 | `POST` | `/api/dispense/refill-completed` | إتمام Refill | JWT |

**⚠️ TD-C4:** 11.2.1، 11.2.2، 11.2.12 مجهولة — تحتاج مراجعة أمنية.

---

### 📁 11.3 MedicationRequestController (6 Endpoints)

| # | Method | Route | الوصف |
|---|---|---|---|
| 11.3.1 | `POST` | `/api/pharmacy/medication-request` | إنشاء طلب دواء |
| 11.3.2 | `GET` | `/api/pharmacy/medication-requests` | طلبات الصيدلية |
| 11.3.3 | `PUT` | `/api/pharmacy/medication-request/{id}/approve` | موافقة |
| 11.3.4 | `PUT` | `/api/pharmacy/medication-request/{id}/reject` | رفض |
| 11.3.5 | `GET` | `/api/pharmacy/my-requests` | طلبات المريض |
| 11.3.6 | `PUT` | `/api/pharmacy/medication-request/{id}/cancel` | إلغاء |

**المصادقة:** كلها مطلوبة (JWT).

---

### 📁 11.4 PharmacyOrdersController (4 Endpoints) — موثّق في §19

---

### 📁 11.5 PharmacyPspController (1 Endpoint) — موثّق في §20

---

## 👔 §14: Rep (12 Endpoints)

**المسار:** `Api.Web/Controllers/Rep/RepController.cs`
**Route:** `[Route("api/rep")]`

| # | Method | Route | الوصف |
|---|---|---|---|
| 14.1 | `GET` | `/api/rep/programs` | برامج المندوب |
| 14.2 | `GET` | `/api/rep/my-doctors` | أطباء المندوب (USED) |
| 14.3 | `GET` | `/api/rep/my-pharmacies` | صيدليات المندوب (USED) |
| 14.4 | `GET` | `/api/rep/my-invitations` | سجل الدعوات |
| 14.5 | `POST` | `/api/rep/invitations/create` | إنشاء دعوة |
| 14.6 | `POST` | `/api/rep/invitations/{id}/resend` | إعادة إرسال |
| 14.7 | `GET` | `/api/rep/dashboard/stats` | إحصائيات اللوحة |
| 14.8 | `GET` | `/api/rep/my-network` | شبكتي |
| 14.9 | `GET` | `/api/rep/current-info` | معلومات المندوب |
| 14.10 | `GET` | `/api/rep/psp-programs` | برامج PSP للشركة |
| 14.11 | `GET` | `/api/rep/company-programs` | برامج شركة المندوب |
| 14.12 | `GET` | `/api/rep/invitations/{id}` | تفاصيل دعوة |

**المصادقة:** كلها مطلوبة (JWT).

---

## 📊 §15: Reports (8 Endpoints)

**المسار:** `Api.Web/Controllers/Reports/ReportsController.cs`

| # | Method | Route | الوصف |
|---|---|---|---|
| 15.1 | `GET` | `/api/reports/provider-funnel` | معدل تحويل الدعوات |
| 15.2 | `GET` | `/api/reports/patient-funnel` | قمع تحويل المريض |
| 15.3 | `GET` | `/api/reports/pharmacy-dispensation` | صرف الصيدليات |
| 15.4 | `GET` | `/api/reports/pharmacy-financial` | التقرير المالي |
| 15.5 | `GET` | `/api/reports/adherence` | الالتزام بالجرعات |
| 15.6 | `GET` | `/api/reports/program-lifecycle` | دورة حياة البرنامج |
| 15.7 | `GET` | `/api/reports/dashboard-summary` | لوحة التحكم |
| 15.8 | `GET` | `/api/reports/filters` | الفلاتر المتاحة |

**المصادقة:** كلها مطلوبة (JWT).

**⚠️ S12:** `GetAdherenceHandler`، `GetPharmacyFinancialHandler`، `GetProgramLifecycleHandler` — تحتاج ربط Points (IC-07, IC-08, IC-09).

---

## 📈 §16: Dashboard & Misc (7 Endpoints)

### 📁 16.1 DashboardController (1 Endpoint)

| # | Method | Route | الوصف |
|---|---|---|---|
| 16.1.1 | `GET` | `/api/dashboard` | لوحة التحكم العامة |

---

### 📁 16.2 HealthController (1 Endpoint)

| # | Method | Route | الوصف |
|---|---|---|---|
| 16.2.1 | `GET` | `/health` | Health Check |

---

### 📁 16.3 DrugFormsController (1 Endpoint)

| # | Method | Route | الوصف |
|---|---|---|---|
| 16.3.1 | `GET` | `/api/drug-forms` | الأشكال الدوائية |

---

### 📁 16.4 MedicationCategoriesController (1 Endpoint)

| # | Method | Route | الوصف |
|---|---|---|---|
| 16.4.1 | `GET` | `/api/medication-categories` | فئات الأدوية |

---

### 📁 16.5 LocalizationController (2 Endpoints)

| # | Method | Route | الوصف |
|---|---|---|---|
| 16.5.1 | `GET` | `/api/localization/page/{page}` | ترجمات صفحة |
| 16.5.2 | `GET` | `/api/localization/key/{key}` | ترجمة واحدة |

---

## 📊 §22: جدول ملخص (Summary) — نهائي

| # | المجموعة | Controllers | Endpoints | الحالة |
|---|---|---|---|---|
| 2 | Auth & Identity | 2 | 20 | ✅ |
| 3 | User & Profile | 2 | 10 | ✅ |
| 4 | Organizations | 4 | 13 | ✅ |
| 5 | Clinical & Geo | 5 | 11 | ✅ |
| 6 | Messaging | 2 | 16 | 🟡 (GAP-11) |
| 7 | Notifications | 1 | 9 | 🟡 (S09) |
| 8 | Clinic | 1 | 15 | ✅ |
| 9 | Professional | 2 | 4 | ✅ |
| 10 | Patient | 2 | 13 | ✅ |
| 11 | Pharmacy | 5 | 32 | ✅ |
| 12 | PSP | 15 | 68 | ✅ |
| 13 | Points | 6 | 38 | ✅ |
| 14 | Rep | 1 | 12 | ✅ |
| 15 | Reports | 1 | 8 | ✅ |
| 16 | Dashboard & Misc | 5 | 7 | ✅ |
| 17 | ClinicReferral ⭐ | 1 | 4 | ✅ |
| 18 | Prescription ⭐ | 1 | 8 | ✅ |
| 19 | PharmacyOrders ⭐ | 1 | 4 | ✅ |
| 20 | PharmacyPsp ⭐ | 1 | 1 | ✅ |
| 21 | Commerce | 6 | 28 | ✅ |
| **المجموع** | | **~45+** | **~337** | |

---

## 🚨 §23: ملاحظات حرجة (Critical Notes)

| # | الملاحظة | المرجع |
|---|---|---|
| 1 | **`TD-C4`**: 10 Endpoints `[AllowAnonymous]` تحتاج مراجعة أمنية (DispenseController 3، PharmaCompanyMedications 2، DispensationPlans 2، Programs 2، Pharmacy 1) | `00B §8` |
| 2 | **`GAP-01`**: `Ledger` موجود كـ API (3 Endpoints) لكن **غير مفعّل** وظيفياً — لا يُستدعى من `SupplyOrderService` | `00B §7` |
| 3 | **`BLOCK-1`**: 403 في `patient-details` لو `patientId=15` (PatientProfileID) بدل `CaseID` | `00B §3` |
| 4 | **`GAP-11`**: Messaging — خلط بين الرسائل الشخصية والمنظمات | `00B §7` |
| 5 | **`GAP-02`**: `Join.razor` لا يكمل الاشتراك (0%) | `00B §7` |
| 6 | **`GAP-12`**: `PrescriptionComposerView` غير موجود (0%) | `00B §7` |
| 7 | **`GAP-03`**: `DoctorNoteController` غير موجود | `00B §7` |
| 8 | **`GAP-13`**: `PSPDataEntryResult` غير موجود | `00B §7` |
| 9 | **`12.1.2` + `12.1.4`**: `programs-temp` + `program-details-temp` — مؤقتة | `09 §12` |
| 10 | **`12.4.2` + `12.5.1`**: `unsubscribe` مكرر | `09 §12` |

---

## 🔗 §24: روابط ذات صلة

- `00-MASTER-INDEX.md` — الفهرس الشامل
- `00B-PROJECT-BACKLOG.md` — الحالة الحية
- `01-ARCHITECTURE.md` — الهيكل المعماري
- `05-psp-system.md` — نظام PSP
- `06-POINTS.md` — نظام النقاط
- `23-COMMERCE-ENGINE.md` — محرك التجارة
- `24-pharmacy-B2C-Orders.md` — طلبات B2C

---

## 📝 §25: سجل التغييرات

| الإصدار | التاريخ | التغييرات |
|---|---|---|
| 1.0 | 17 مايو 2026 | الإصدار الأولي |
| 2.0 | 16 أكتوبر 2026 | شامل — ~290 endpoint |
| 2.1 | 12 نوفمبر 2026 | إضافة Commerce (28 Endpoint) |
| **3.0** | **25 نوفمبر 2026** | **إعادة كتابة كاملة (Developer-First Edition): إضافة 17 Endpoint (ClinicReferral 4 + Prescription 8 + PharmacyOrders 4 + PharmacyPsp 1)؛ توثيق ذكي لكل Endpoint (Input/Output/Auth/Errors)؛ حذف التكرار؛ نقل `AllowAnonymous` إلى `00B TD-C4`؛ تحديث الأرقام إلى ~337 Endpoint؛ إضافة §23 ملاحظات حرجة.** |

---

© 2026 RubikCare — للاستخدام الداخلي

**نهاية الوثيقة** ✅
```

---

## 📋 ملخص اكتمال `09-API-GUIDE v3.0`

| الجزء | المحتوى | الحجم |
|---|---|---|
| **الجزء 1** | المنهجية + الفهرس + ClinicReferral (4) + Prescription (8) + PharmacyOrders (4) + PharmacyPsp (1) | 17 Endpoint |
| **الجزء 2** | Commerce (28) + Points (38) + PSP (62) | 128 Endpoint |
| **الجزء 3** | Auth (20) + User (10) + Organizations (13) + Clinical (11) + Messaging (16) + Notifications (9) + Clinic (15) + Professional (4) + Patient (13) + Pharmacy (32) + Rep (12) + Reports (8) + Dashboard (7) | 170 Endpoint |
| **الإجمالي** | **~337 Endpoint** | **~45+ Controller** |

### ✅ ما تم إنجازه في v3.0

| # | الإنجاز |
|---|---|
| 1 | **توثيق ذكي** — كل Endpoint له: وصف، مصادقة، مدخلات (Path/Query/Body)، استجابة ناجحة، أخطاء محتملة |
| 2 | **إضافة 17 Endpoint** جديدة (ClinicReferral + Prescription + PharmacyOrders + PharmacyPsp) |
| 3 | **حذف التكرار** — لا تكرار بين الجداول |
| 4 | **نقل `AllowAnonymous`** إلى `00B TD-C4` (مع الإشارة في §23) |
| 5 | **§23 ملاحظات حرجة** — 10 ملاحظات (TD-C4، GAP-01، BLOCK-1، إلخ) |
| 6 | **الأرقام الفعلية** — ~337 Endpoint، ~45+ Controller |
