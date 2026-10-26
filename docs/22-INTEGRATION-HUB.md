
# 22 — INTEGRATION-HUB (مركز التكامل الخارجي)

الإصدار: 1.0 (Draft — Foundations Edition) 
آخر تحديث: 26 أكتوبر 2026 
الحالة: 🟡 مسودة — بانتظار المراجعة والاعتماد 
الجمهور: فريق التطوير + AI Assistants + القيادة التقنية 
المرجع الإلزامي: يُقرأ قبل أي تطوير يتعلق بالتكامل مع الأنظمة الخارجية.

---

## 📌 الجزء صفر: الفلسفة والرؤية

### 0.1 ما هو Integration Hub؟
Integration Hub هو **الطبقة الوسيطة (Middleware Layer)** التي تسمح لمنصة RubikCare بتبادل البيانات بشكل آمن ومنظم مع أي نظام خارجي (عيادات، مستشفيات، معامل، شركات أدوية، مخازن، صيدليات).

**ليس** مجرد `HttpClient` يرسل طلبات. هو **نظام بيئي (Ecosystem)** كامل يتضمن:
- التوثيق (Authentication).
- المرونة (Retry & Resilience).
- التتبع (Audit & Logging).
- التوحيد (Standardized DTOs).
- العقود (Contracts & Webhooks).

### 0.2 الرؤية الاستراتيجية (من 17-STRATEGIC-ROADMAP)
> "ما أفكر فيه هو تصميم المشروع كله بهذا النمط، بحيث يمكن إعادة استخدام أي ميزة لاحقاً... كذلك يمكن من خلال هذا الفكر تطوير مكتبة API client لإعادة استخدام هذه القطع الخدمية المستقلة داخلياً وخارجياً."
> (نص المؤلف الحرفي — وثيقة 17)

الترجمة التقنية:
- Integration Hub هو **الخدمة القابلة لإعادة الاستخدام** (Reusable Service) التي تربط كل شيء.
- لا نبني "تكاملاً مخصصاً" لكل نظام. نبني **Hub** واحد، ثم نضيف **Adapters** لكل نظام.
- هذا يمنع الدين التقني (ازدواجية الخدمات) قبل أن ينشأ.

### 0.3 المبادئ التصميمية (مستوحاة من 17)
1. **مرونة كاملة (Nothing Hardcoded)**: كل Adapter هو مكون مستقل، يمكن إضافته أو إزالته دون تعديل Core.
2. **التوحيد (Standardization)**: كل تكامل يمر عبر `IIntegrationClient` (Interface موحد)، بغض النظر عن التقنية (REST, SOAP, FHIR, Webhook).
3. **الأمان أولاً (Security First)**: OAuth 2.0، API Keys، Rate Limiting، Encryption، Audit Log.
4. **المرونة (Resilience)**: Retry Policy، Circuit Breaker، Timeout، Fallback.
5. **الشفافية (Transparency)**: كل استدعاء يُسجل (Audit Log). لا "صندوق أسود".
6. **التوثيق قبل التنفيذ**: هذه الوثيقة.

---

## 🗄️ الجزء الأول: نموذج البيانات (Data Model)

### 1.1 الطبقات الثلاث
Integration Hub يتكون من 3 طبقات:

```
┌─────────────────────────────────────────────────────────────┐
│  Layer 1: Core (القلب)                                     │
│  IntegrationClient, IntegrationRequest, IntegrationResult   │
├─────────────────────────────────────────────────────────────┤
│  Layer 2: Adapters (المحولات)                              │
│  ClinicOsAdapter, LabLisAdapter, HospitalHisAdapter, ...    │
├─────────────────────────────────────────────────────────────┤
│  Layer 3: Contracts (العقود)                               │
│  DTOs, OpenAPI Spec, Webhook Events                         │
└─────────────────────────────────────────────────────────────┘
```

### 1.2 الطبقة 1: Core (القلب)

#### 1.2.1 `IntegrationClient` (كيان التكامل)
| العمود | النوع | الوصف |
| :--- | :--- | :--- |
| `ClientID` | PK, int | |
| `Name` | nvarchar(200) | اسم النظام الخارجي (مثلاً "ClinicOS - Al-Amal") |
| `AdapterType` | nvarchar(50) | `CLINIC_OS`, `LAB_LIS`, `HOSPITAL_HIS`, `PHARMA_ERP`, `PHARMACY_POS` |
| `BaseUrl` | nvarchar(500) | رابط النظام |
| `AuthType` | nvarchar(50) | `OAUTH2`, `API_KEY`, `BASIC`, `JWT` |
| `AuthConfigJson` | nvarchar(max) | إعدادات التوثيق (مشفرة) |
| `OrganizationID` | FK → Organizations | المنظمة المرتبطة |
| `IsActive` | bit | |
| `RateLimitPerMinute` | int | الحد الأقصى للطلبات |
| `CreatedDate`, `LastModifiedDate` | datetime2 | |

#### 1.2.2 `IntegrationRequest` (كائن الطلب الموحد)
| العمود | النوع | الوصف |
| :--- | :--- | :--- |
| `RequestID` | PK, int | |
| `ClientID` | FK → `IntegrationClient` | |
| `Operation` | nvarchar(100) | `GetPatient`, `PushLabResult`, `ShareRecord` |
| `PayloadJson` | nvarchar(max) | البيانات |
| `IdempotencyKey` | nvarchar(100) | لمنع التكرار |
| `Status` | nvarchar(50) | `PENDING`, `IN_PROGRESS`, `SUCCESS`, `FAILED` |
| `RetryCount` | int | |
| `CreatedDate`, `CompletedDate` | datetime2 | |

#### 1.2.3 `IntegrationResult` (كائن النتيجة الموحد)
| العمود | النوع | الوصف |
| :--- | :--- | :--- |
| `ResultID` | PK, int | |
| `RequestID` | FK → `IntegrationRequest` | |
| `Success` | bit | |
| `ResponseJson` | nvarchar(max) | البيانات المُرجعة |
| `ErrorCode` | nvarchar(50) | |
| `ErrorMessage` | nvarchar(500) | |
| `DurationMs` | int | |
| `Timestamp` | datetime2 | |

#### 1.2.4 `IntegrationAuditLog` (سجل التتبع)
| العمود | النوع | الوصف |
| :--- | :--- | :--- |
| `LogID` | PK, int | |
| `ClientID` | FK → `IntegrationClient` | |
| `RequestID` | FK → `IntegrationRequest` (nullable) | |
| `Action` | nvarchar(100) | `SEND`, `RECEIVE`, `AUTH_FAILED`, `RATE_LIMITED` |
| `Details` | nvarchar(max) | تفاصيل إضافية |
| `UserProfileID` | FK → UserProfiles (nullable) | من قام بالعملية |
| `Timestamp` | datetime2 | |

### 1.3 الطبقة 2: Adapters (المحولات)

كل Adapter هو **تنفيذ** لـ `IIntegrationClient`، ويتعامل مع تقنية مختلفة:

| Adapter | التقنية | الاستخدام |
| :--- | :--- | :--- |
| `ClinicOsAdapter` | REST | العيادات الداخلية |
| `HospitalHisAdapter` | SOAP | المستشفيات القديمة |
| `LabLisAdapter` | HL7 FHIR | المعامل |
| `PharmaErpAdapter` | REST | شركات الأدوية |
| `PharmacyPosAdapter` | Webhook | الصيدليات |
| `WarehouseWmsAdapter` | REST | المخازن |

**قاعدة صارمة:** لا يوجد Adapter يعرف عن الآخر. كلهم يتحدثون عبر `IIntegrationClient`.

### 1.4 الطبقة 3: Contracts (العقود)

#### 1.4.1 DTOs موحدة (Unified DTOs)
- `PatientDto`: (UserProfileID, FullName, DateOfBirth, Gender, ...).
- `LabResultDto`: (ResultID, LabTestType, Value, Unit, ReadingDate, ...).
- `PrescriptionDto`: (PrescriptionID, Medications, Dosage, ...).
- `ConsentDto`: (ConsentID, Scope, ValidFrom, ValidTo, ...).
- `AppointmentDto`: (AppointmentID, DoctorID, Date, ...).

#### 1.4.2 OpenAPI Spec (Swagger)
- كل Adapter يجب أن يُنتج `openapi.json` خاصاً به.
- هذا يسمح للمطورين الخارجيين بفهم كيف يتكاملون مع RubikCare.

#### 1.4.3 Webhook Events (الأحداث)
| الحدث | متى يُطلق | المستهلك |
| :--- | :--- | :--- |
| `record.created` | عند إضافة سجل طبي جديد | ClinicOS |
| `consent.given` | عند مشاركة سجل | ClinicOS, Labs |
| `consent.revoked` | عند إلغاء مشاركة | ClinicOS, Labs |
| `lab_result.received` | عند استلام نتيجة من معمل | PHR |
| `prescription.created` | عند إنشاء وصفة | Pharmacy |

---

## 🔗 الجزء الثاني: عقود التكامل (Integration Contracts)

على غرار `01-ARCHITECTURE` (الجزء السادس-ب)، كل تكامل يجب أن يكون موثقاً:

| # | From → To | Event | Reference | Idempotency |
| :--- | :--- | :--- | :--- | :--- |
| IC-IH-01 | PHR → ClinicOS | RecordShared | `CONSENT/{ConsentID}` | ✅ مفتاح Idempotency |
| IC-IH-02 | PHR → LabLIS | LabResultRequest | `LAB_TEST_RESULT/{ResultID}` | ✅ |
| IC-IH-03 | LabLIS → PHR | LabResultReceived | `LAB_RESULT/{ResultID}` | ✅ |
| IC-IH-04 | PHR → PharmaERP | PrescriptionShared | `PRESCRIPTION/{PrescriptionID}` | ✅ |
| IC-IH-05 | ClinicOS → PHR | PatientRecordCreated | `HEALTH_RECORD/{RecordID}` | ✅ |
| IC-IH-06 | Integration Hub → Notifications | IntegrationFailed | `INTEGRATION_REQUEST/{RequestID}` | N/A |

**القاعدة الذهبية:** لا تكامل بدون عقد موثق هنا.

---

## 🎯 الجزء الثالث: سيناريوهات التكامل (Integration Scenarios)

### 3.1 السيناريو 1: مشاركة سجل مع عيادة (ClinicOS)
```
المريض يشارك سجله مع عيادة (SharingConsent)
    ↓
PHR يستدعي Integration Hub: ShareRecord
    ↓
ClinicOsAdapter يحول DTO إلى صيغة ClinicOS
    ↓
إرسال HTTP POST إلى ClinicOS
    ↓
ClinicOS يؤكد الاستلام
    ↓
Integration Hub يسجل النجاح في Audit Log
    ↓
إشعار للمريض (Notification)
```

### 3.2 السيناريو 2: استلام نتيجة تحليل من معمل (LabLIS)
```
معمل يرسل نتيجة تحليل (Webhook) إلى Integration Hub
    ↓
LabLisAdapter يستقبل الـ Webhook ويحوله إلى `LabResultDto`
    ↓
Integration Hub يستدعي PHR: SaveLabResult
    ↓
PHR ينشئ HealthRecord + LabTestResult
    ↓
إشعار للمريض
    ↓
Integration Hub يسجل النجاح
```

### 3.3 السيناريو 3: جلب بيانات مريض من مستشفى (HospitalHIS)
```
عيادة تطلب بيانات مريض من مستشفى
    ↓
Integration Hub يستدعي HospitalHisAdapter: GetPatient
    ↓
HospitalHisAdapter يحول الطلب إلى SOAP
    ↓
يرسل SOAP Request إلى HospitalHIS
    ↓
يستقبل SOAP Response
    ↓
يحوله إلى PatientDto
    ↓
يعيده إلى العيادة
```

### 3.4 السيناريو 4: إرسال وصفة إلى صيدلية (PharmacyPOS)
```
الطبيب ينشئ وصفة في PHR
    ↓
PHR يستدعي Integration Hub: PushPrescription
    ↓
PharmacyPosAdapter يحول الوصفة إلى Webhook
    ↓
يرسل Webhook إلى الصيدلية
    ↓
الصيدلية تؤكد الاستلام
    ↓
PHR يُحدث حالة الوصفة
```

---

## 🔐 الجزء الرابع: الأمان والخصوصية (Security & Privacy)

### 4.1 المبادئ
1. **Zero Trust**: لا تثق بأي نظام خارجي. كل طلب يجب أن يُصادق عليه.
2. **Least Privilege**: كل Adapter له صلاحيات محدودة (فقط ما يحتاجه).
3. **Encryption**: كل البيانات الحساسة مشفرة (TLS 1.3 + AES-256).
4. **Audit**: كل استدعاء يُسجل (من، ماذا، متى، نتيجة).
5. **Rate Limiting**: حماية من الإغراق (DDoS).

### 4.2 آليات المصادقة (Authentication)
| النوع | الاستخدام |
| :--- | :--- |
| **OAuth 2.0** | للأنظمة الحديثة (ClinicOS, PharmaERP) |
| **API Keys** | للأنظمة البسيطة (PharmacyPOS) |
| **JWT** | للأنظمة الداخلية (بين خدمات RubikCare) |
| **mTLS** | للمعامل (بيانات حساسة) |

### 4.3 الصلاحيات (Permissions) — تتكامل مع `19-PERMISSIONS.md`
- `INTEGRATION.READ`: قراءة بيانات من نظام خارجي.
- `INTEGRATION.WRITE`: إرسال بيانات إلى نظام خارجي.
- `INTEGRATION.ADMIN`: إدارة IntegrationClients.
- `INTEGRATION.AUDIT`: عرض Audit Log.

### 4.4 الامتثال (Compliance)
- **PDPL** (قانون حماية البيانات المصري): كل مشاركة تتطلب `SharingConsent`.
- **GDPR** (إذا توسعنا لأوروبا): Right to be Forgotten (Soft Delete + Anonymization).
- **HIPAA** (إذا توسعنا لأمريكا): Encryption + Audit + Access Control.

---

## 📅 الجزء الخامس: خارطة التنفيذ (Roadmap)

### Phase 1: Core (Sprint 7 — 22-26 نوفمبر 2026)
- Domain: `IntegrationClient`, `IntegrationRequest`, `IntegrationResult`, `IntegrationAuditLog`.
- Application: `IIntegrationClient` Interface + `IntegrationOrchestrator`.
- Infrastructure: `AuthenticationService`, `RetryPolicy`, `AuditLogger`, `RateLimiter`.
- API: `IntegrationsController` (Endpoints أساسية).

### Phase 2: First Adapter (Sprint 8 — 29 نوفمبر - 3 ديسمبر 2026)
- `ClinicOsAdapter` (REST).
- `ClinicOsDtoMapper`.
- اختبارات E2E: PHR → ClinicOS.

### Phase 3: Webhooks + OpenAPI (Sprint 9 — 6-10 ديسمبر 2026)
- Webhook Events (`record.created`, `consent.given`, ...).
- OpenAPI Spec (Swagger) لكل Adapter.
- بوابة مطورين (Developer Portal).

### Phase 4: More Adapters (Q1 2027)
- `LabLisAdapter` (HL7 FHIR).
- `PharmacyPosAdapter` (Webhook).
- `PharmaErpAdapter` (REST).

### Phase 5: Hospital + Warehouse (Q2 2027)
- `HospitalHisAdapter` (SOAP).
- `WarehouseWmsAdapter` (REST).

### Phase 6: API Marketplace (Q3 2027)
- فتح API للجمهور.
- تسعير (Pay-per-call).
- Rubika كعملة للدفع.

---

## 🚨 الجزء السادس: القواعد الصارمة (Rules)

| # | القاعدة | السبب |
| :--- | :--- | :--- |
| R1 | لا تبنِ Adapter قبل Core. | ستعيد كتابته. |
| R2 | كل Adapter يجب أن يكون `IIntegrationClient`. | التوحيد. |
| R3 | كل استدعاء يجب أن يُسجل في `IntegrationAuditLog`. | الشفافية. |
| R4 | كل تكامل يجب أن يكون له `IdempotencyKey`. | منع التكرار. |
| R5 | لا تخزن بيانات حساسة في `AuthConfigJson` بدون تشفير. | الأمان. |
| R6 | كل Webhook يجب أن يكون موقّعاً (HMAC). | منع التزوير. |
| R7 | لا تتجاهل SOAP (المستشفيات القديمة). | الواقعية. |
| R8 | لا تتجاهل HL7 FHIR (المعامل). | المعيار العالمي. |
| R9 | كل Adapter يجب أن يكون له OpenAPI Spec. | التوثيق. |
| R10 | لا تكامل بدون عقد موثق في هذه الوثيقة. | الحوكمة. |

---

## 🔗 روابط ذات صلة

- `01-ARCHITECTURE.md` — Integration Contracts.
- `17-STRATEGIC-ROADMAP.md` — المبادئ التصميمية.
- `19-PERMISSIONS.md` — الصلاحيات.
- `21-PHR-SYSTEM.md` — مصدر البيانات.
- `00B-PROJECT-BACKLOG.md` — Sprint 7-9.

---

## 📝 الجزء السابع: سجل التحديثات

| الإصدار | التاريخ | التغييرات |
| :--- | :--- | :--- |
| 1.0 (Draft) | 26 أكتوبر 2026 | إنشاء الوثيقة — الفلسفة + نموذج البيانات + العقود + السيناريوهات + الأمان + خارطة التنفيذ |

---

© 2026 RubikCare — للاستخدام الداخلي
نهاية الوثيقة ✅
```
*Sprint 4 (PHR Phase 1)** ونترك تحديث الوثائق لاحقاً؟
