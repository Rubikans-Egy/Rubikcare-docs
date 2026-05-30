# 09 - دليل API

**آخر تحديث: 17 مايو 2026**

---

## مقدمة

هذا المرجع هو الدليل الكامل لجميع APIs في منصة RubikCare. يتم استخدام هذه الـ APIs من قبل:
- تطبيق الويب (Blazor Server) - عبر HTTP/SignalR
- تطبيق الموبايل (MAUI) - عبر HTTP/REST

---

## الهيكل العام

```
RubikCare.Api.Web/
├── Controllers/
│   ├── AuthController.cs        # المصادقة وتسجيل الدخول
│   ├── UserController.cs        # إدارة المستخدمين والملفات الشخصية
│   ├── PspController.cs         # برامج دعم المرضى
│   ├── DispenseController.cs    # صرف الأدوية
│   ├── PharmacyController.cs    # إدارة الصيدليات
│   ├── OrganizationController.cs # إدارة المنظمات
│   └── AdminController.cs       # لوحة التحكم والإدارة
├── Program.cs                   # تسجيل الخدمات و Middleware
└── appsettings.json             # إعدادات التطبيق
```

---

## المصادقة (Authentication)

### تسجيل الدخول
```
POST /api/auth/login
```
**Request:**
```json
{
    "email": "user@example.com",
    "password": "Password123!"
}
```
**Response (نجاح):**
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

### تسجيل مستخدم جديد
```
POST /api/auth/register
```
**Request:**
```json
{
    "firstName": "أحمد",
    "lastName": "محمد",
    "email": "ahmed@example.com",
    "phoneNumber": "0123456789",
    "password": "Password123!",
    "role": "Doctor"
}
```

---

## المستخدمون (Users)

### جلب الملف الشخصي
```
GET /api/user/profile
Authorization: Bearer {token}
```
**Response:**
```json
{
    "userProfileId": 12,
    "firstName": "أحمد",
    "lastName": "محمد",
    "email": "ahmed@example.com",
    "phoneNumber": "0123456789",
    "roles": ["Doctor"],
    "organizations": [
        {
            "organizationId": 5,
            "organizationName": "شركة ديفارت لاب",
            "role": "Owner"
        }
    ]
}
```

### تحديث الملف الشخصي
```
PUT /api/user/profile
Authorization: Bearer {token}
```
**Request:**
```json
{
    "firstName": "أحمد",
    "lastName": "محمد",
    "phoneNumber": "0123456789",
    "profilePictureUrl": "/uploads/profiles/12.jpg"
}
```

---

## برامج دعم المرضى (PSP)

### قائمة البرامج
```
GET /api/psp/programs
Authorization: Bearer {token}
```
**Query Parameters:**
- `page` (int, default: 1)
- `pageSize` (int, default: 10)
- `search` (string, optional)

### تفاصيل برنامج
```
GET /api/psp/programs/{programId}
Authorization: Bearer {token}
```

### الاشتراك في برنامج
```
POST /api/psp/participate
Authorization: Bearer {token}
```
**Request:**
```json
{
    "programId": 2,
    "organizationId": 1245
}
```

### إنشاء دعوة لمريض
```
POST /api/psp/create-invitation
Authorization: Bearer {token}
```
**Request:**
```json
{
    "programId": 2,
    "clinicId": 1245
}
```
**Response:**
```json
{
    "success": true,
    "invitationId": 12,
    "invitationToken": "INV-5CWEFHEJ",
    "message": "تم إنشاء الدعوة بنجاح"
}
```

### نقطة دخول المريض
```
POST /api/psp/entry
Authorization: Bearer {token}
```
**Request:**
```json
{
    "invitationCode": "INV-5CWEFHEJ"
}
```
**حالات الاستجابة:**

| الحالة | الكود | المعنى |
|--------|-------|--------|
| NEW_ENROLLMENT | 200 | تم الاشتراك بنجاح |
| ACTIVE_PROGRAMS | 200 | للمستخدم برامج نشطة بالفعل |
| ALREADY_ENROLLED | 200 | مسجل مسبقاً في هذا البرنامج |
| CODE_INVALID | 400 | كود غير صالح |
| CODE_EXPIRED | 400 | كود منتهي الصلاحية |

### تفاصيل برنامج المريض
```
GET /api/psp/patient-details?patientId={patientId}
Authorization: Bearer {token}
```

---

## صرف الأدوية (Dispensation)

### التحقق من رمز الصرف
```
POST /api/dispense/validate-token
Authorization: Bearer {token}
```
**Request:**
```json
{
    "tokenCode": "RC-20260323-1234"
}
```
**Response (صالحة):**
```json
{
    "isValid": true,
    "tokenStatus": "ACTIVE",
    "patientName": "أحمد محمد",
    "programName": "Infertility Support Program",
    "medicationName": "Oxy Free",
    "quantity": 20,
    "expiryDate": "2026-04-22T00:00:00",
    "dispensationId": 42
}
```

### تأكيد صرف الدواء
```
POST /api/dispense/confirm
Authorization: Bearer {token}
```
**Request:**
```json
{
    "dispensationId": 42,
    "pharmacyId": 1260
}
```
**Response:**
```json
{
    "success": true,
    "message": "تم صرف الدواء بنجاح",
    "dispensedQuantity": 20,
    "remainingDispenses": 2
}
```

---

## الصيدليات (Pharmacy)

### تسجيل صيدلية جديدة
```
POST /api/pharmacy/register
Authorization: Bearer {token}
```
**Request:**
```json
{
    "pharmacyName": "صيدلية الشفاء",
    "licenseNumber": "PH-2026-001",
    "address": "شارع الرئيسي، القاهرة",
    "phoneNumber": "0123456789"
}
```

### جلب بيانات الصيدلية
```
GET /api/pharmacy/my-pharmacy
Authorization: Bearer {token}
```

---

## الترجمة (Localization)

### جلب ترجمات صفحة
```
GET /api/localization/page/{page}?lang={lang}
```
**Parameters:**
- `page`: اسم domain (مثل LOGIN, COMMON, PSP)
- `lang`: `ar` أو `en`

**Response:**
```json
{
    "LOGIN.TITLE": "تسجيل الدخول",
    "LOGIN.EMAIL": "البريد الإلكتروني",
    "LOGIN.PASSWORD": "كلمة المرور",
    "LOGIN.SUBMIT": "دخول"
}
```

---

## المصادقة (JWT)

جميع الـ APIs (ما عدا Login و Register) تتطلب توكن JWT صالح في الهيدر:

```
Authorization: Bearer eyJhbGciOiJIUzI1NiIs...
```

التوكن يحتوي على:
- `userId` - معرف المستخدم
- `userProfileId` - معرف الملف الشخصي
- `roles` - الأدوار النظامية
- `exp` - تاريخ انتهاء الصلاحية

---

## رموز الحالة (HTTP Status Codes)

| الكود | المعنى |
|--------|--------|
| 200 | نجاح |
| 201 | تم الإنشاء |
| 400 | خطأ في البيانات المدخلة |
| 401 | غير مصرح (تسجيل الدخول مطلوب) |
| 403 | ممنوع (صلاحيات غير كافية) |
| 404 | غير موجود |
| 500 | خطأ داخلي في الخادم |

---

## تنسيق الأخطاء

كل الأخطاء تُرجع بنفس التنسيق:

```json
{
    "success": false,
    "message": "وصف الخطأ بالعربية",
    "errors": {
        "Email": ["البريد الإلكتروني مطلوب"],
        "Password": ["كلمة المرور قصيرة جداً"]
    }
}
```
📝 بخصوص AllowAnonymous
متفق معك تماماً. نتركها كما هي الآن مع تدوين الملاحظة التالية:

⚠️ ملاحظة مؤجلة: الـ Endpoints التالية تستخدم [AllowAnonymous] وتحتاج مراجعة أمنية لاحقاً:

DispenseController.ValidateToken + ConfirmDispense

MedicationRequestController.CreateMedicationRequest

PharmaCompanyMedicationsController.GetAll + Search

بعض Endpoints في PspController و DispensationPlansController

نعود لها بعد إنجاز مهام الجلسة الحالية ✅
---

## 🔗 روابط ذات صلة

- [00 - الهيكل المعماري](00-architecture-overview.md)
- [07 - نظام PSP](07-psp-system.md)
- [10 - دليل تطوير MAUI](10-maui-development-guide.md)
- [13 - إصلاح Clean Architecture](13-clean-architecture-enforcement.md)
```

---

