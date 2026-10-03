# 08 — خريطة التطوير (Roadmap)

## 📌 نظرة عامة

وثيقة **خريطة التطوير** توضح:
- ✅ المهام المنجزة
- 🔄 المهام الحالية
- 🎯 المهام القادمة
- ⏸️ المهام المؤجلة
- ⚠️ الديون التقنية

**آخر تحديث:** 3 أكتوبر 2026

**الإصدار الحالي:** `V2.0`

---

## 📊 لوحة المعلومات السريعة

| الفئة | العدد |
|-------|-------|
| ✅ مكتمل | 32 |
| 🔄 جاري | 1 |
| 🎯 مخطط | 10 |
| ⏸️ مؤجل | 5 |
| ⚠️ دين تقني | 8 |

---

## 🗂️ المشاريع الحالية (9 مشاريع)

| # | المشروع | التقنية | الحالة |
|---|---------|---------|:------:|
| 1 | `RubikCare.Domain` | .NET 10 Class Library | ✅ مستقر |
| 2 | `RubikCare.Application` | .NET 10 Class Library | ✅ مستقر |
| 3 | `RubikCare.Infrastructure` | .NET 10 + EF Core 10 | ✅ مستقر |
| 4 | `RubikCare.Api.Web` | ASP.NET Core 10 | ✅ مستقر |
| 5 | `RubikCare.Web` | Blazor Server 10 | ✅ مستقر |
| 6 | `RubikCare.PWA` | Blazor WebAssembly 10 | 🔄 75% |
| 7 | `RubikCare.Mobile` | .NET MAUI 10 | 🔄 في التطوير |
| 8 | `RubikCare.Shared.UI` | Razor Class Library | ✅ مستقر |
| 9 | `RubikCare.Tests` | xUnit + Moq | ✅ 33 اختبار |

---

## ✅ المهام المكتملة (Sprint سبتمبر — أكتوبر 2026)

### 🎯 نظام نقاط برامج الدعم (PSP Program Points) — Backend مكتمل

| # | المهمة | الحالة | التاريخ |
|---|--------|:------:|:-------:|
| 1 | **Migration: AddPSPProgramPointsTables** (8 جداول) | ✅ | 29/09/2026 |
| 2 | **Seed Data (15 صفاً)** | ✅ | 29/09/2026 |
| 3 | **Domain — 8 كيانات + 3 Constants** | ✅ | 29/09/2026 |
| 4 | **19 Handler** (Schemes/Rules/Balances/Earning/Redeem) | ✅ | 29/09/2026 |
| 5 | **5 Controllers — 18 Endpoint** | ✅ | 29/09/2026 |
| 6 | **`PSPEnrollmentService`** | ✅ | 30/09/2026 |
| 7 | **`IPSPPointSchemeService` + `PSPPointSchemeService`** | ✅ | 30/09/2026 |
| 8 | **ربط `PATIENT_INVITED`** | ✅ | 30/09/2026 |
| 9 | **ربط `PATIENT_ENROLLED`** | ✅ | 30/09/2026 |
| 10 | **ربط `ERX_CREATED`** | ✅ | 30/09/2026 |
| 11 | **ربط `DISPENSATION_COMPLETED`** | ✅ | 01/10/2026 |
| 12 | **ربط `REFILL_COMPLETED`** | ✅ | 01/10/2026 |

### 🎯 نظام النقاط — Frontend

| # | المهمة | الحالة | التاريخ |
|---|--------|:------:|:-------:|
| 13 | **`PSPStep2b_PointScheme.razor`** (واجهة إدارة الشركة) | ✅ | 30/09/2026 |
| 14 | **عرض نظام النقاط في `PSPProgramsDetails`** | ✅ | 01/10/2026 |
| 15 | **`PointDashboard` (Shared.UI — متعدد المنصات)** | ✅ | 01/10/2026 |
| 16 | **تكامل PWA** (صفحة + بطاقة) | ✅ | 01/10/2026 |
| 17 | **تكامل MAUI** (Container + بطاقة) | ✅ | 01/10/2026 |
| 18 | **مزامنة `access_token` Claim في `Login.razor`** | ✅ | 01/10/2026 |
| 19 | **`IsTokenValid` في `PointSchemeClientService`** | ✅ | 01/10/2026 |
| 20 | **حل مشكلة Google Login** | ✅ | 03/10/2026 |

### 🎯 إصلاحات عامة

| # | المهمة | الحالة | التاريخ |
|---|--------|:------:|:-------:|
| 21 | **قفزة القائمة الجانبية (RTL/LTR)** | ✅ | 30/09/2026 |
| 22 | **`_pspBundle.css` → `_pspbundle.css`** (case-sensitive) | ✅ | 01/10/2026 |
| 23 | **Phase System (PSP)** | ✅ | 27/09/2026 |
| 24 | **Required Tests (PSP)** | ✅ | 27/09/2026 |
| 25 | **Rep Dashboard** | ✅ | سبتمبر 2026 |
| 26 | **Pharma Company Dashboard** | ✅ | سبتمبر 2026 |
| 27 | **Default Language = "en"** | ✅ | 27/09/2026 |
| 28 | **إصلاح `default-avatar.png` 404** | ✅ | 27/09/2026 |
| 29 | **إصلاح صفحة الروابط (ترجمة + CSS)** | ✅ | 27/09/2026 |
| 30 | **Reports System** (Pharma Company) | ✅ | سبتمبر 2026 |
| 31 | **Notifications** (Push + Local) | ✅ | سبتمبر 2026 |
| 32 | **Translate System** (3 مسارات) | ✅ | 17/09/2026 |

---

## 🔄 المهام الجارية

| # | المهمة | التقدم | الملاحظات |
|---|--------|:------:|-----------|
| 1 | **PWA — نقل باقي الصفحات** | 🔄 75% | 45 صفحة متبقية |
| 2 | **Toast Notification** | 🔄 في شات منفصل | مشكلة: يظهر في آخر الصفحة |

---

## 🎯 المهام القادمة (Sprint أكتوبر 2026)

### 🔴 أولوية عالية — إكمال نظام النقاط

| # | المهمة | التقدير | يعتمد على |
|---|--------|:-------:|:---------:|
| 1 | **بطاقة مصغّرة للنقاط** في `ProgramDetailsPage` (Web + MAUI) | ~2 ساعات | ✅ Backend |
| 2 | **لوحة الشركة — إدارة الأرصدة + Redeem** | ~4-5 ساعات | ✅ Backend |
| 3 | **واجهة العيادة (Web) — Dashboard النقاط** | ~5-6 ساعات | ✅ Backend |
| 4 | **اختبار End-to-End شامل** لنظام النقاط | ~2-3 ساعات | ✅ Backend |

### 🔴 أولوية عالية — PSP

| # | المهمة | التقدير | يعتمد على |
|---|--------|:-------:|:---------:|
| 5 | **تحديد التحاليل الطبية** (Required Tests) — التحسين | ~6-8 ساعات | — |
| 6 | **خصوصية التحاليل** | ~4-6 ساعات | مهمة 5 |
| 7 | **رفع التحاليل (PDF) + التذكيرات** | ~10-12 ساعة | مهمة 5+6 |

### 🟡 أولوية متوسطة

| # | المهمة | التقدير | يعتمد على |
|---|--------|:-------:|:---------:|
| 8 | **ربط `PROGRAM_COMPLETED`** | ~1-2 ساعة | آلية الإكمال |
| 9 | **ربط `LAB_TEST_UPLOADED`** | ~2-3 ساعات | Handler رفع التحاليل |
| 10 | **تقرير نقاط لكل مريض** | ~2 ساعات | API جديد |

### 🟢 أولوية منخفضة

| # | المهمة | التقدير |
|---|--------|:-------:|
| 11 | **إعادة تسمية `IMobileNavigationService` → `IAppNavigationService`** | ~1 ساعة |
| 12 | **`WebApiService` — فحص `exp`** (مثل `PointSchemeClientService`) | ~30 دقيقة |

---

## ⏸️ المهام المؤجلة

| # | المهمة | السبب |
|---|--------|-------|
| 1 | **صفحة المحادثات (Messaging Hub)** | غير مكتملة — مخفية من الـ UI |
| 2 | **QR Code للانضمام من MAUI (Deep Linking)** | مؤجل — مش أولوية |
| 3 | **Android Deep Linking في BlazorWebView** | مؤجل |
| 4 | **إعادة هيكلة `PspController`** (Refactoring) | مؤجل — دَين تقني |
| 5 | **الانتقال من `DbContext` direct إلى `API` في UI** | مؤجل — فرض تغيير جذري |

---

## ⚠️ الديون التقنية (Technical Debt)

### 🔴 حرجة (قبل الإنتاج):

| # | المشكلة | الملفات | الأولوية |
|---|---------|---------|:--------:|
| 1 | صفحات كاملة فيها `@page` داخل `Shared.UI` | `PharmacyDetailPage.razor`, `PspAboutPage.razor` | 🔴 |
| 2 | خلط Namespaces | `PharmacySearchPage.razor`, `PharmacyDetailPage.razor` | 🔴 |
| 3 | خطأ 404 في `/admin/pending-licenses` | `Pages/Admin/SystemManagment/` | 🔴 |

### 🟡 متوسطة (خلال شهر):

| # | المشكلة | الحالة |
|---|---------|--------|
| 4 | `_homepage.css` 404 (warning) | ⏸️ مؤجل |
| 5 | `Toast` مش بيشتغل بشكل صح | 🔄 في شات منفصل |
| 6 | `Set-Content` في PowerShell يتلف العربي | ⚠️ تحذيري |
| 7 | `[Parameter]` في Blazor مع MAUI مش موثوق | ⚠️ تحذيري |
| 8 | `pspdashboard.css` مفقود (خطأ MSB4018) | 🟢 |

### 🟢 منخفضة (تحسينات):

| # | المشكلة | الحالة |
|---|---------|--------|
| 9 | استخدام `alert()` بدل Toast موحد | ⏸️ |
| 10 | `HttpUtility` في Blazor WASM | ⏸️ |
| 11 | `IMobileNavigationService` اسم مضلل | ⏸️ |

---

## 📅 الجدول الزمني

### Sprint سبتمبر 2026 (مكتمل):
- ✅ Phase System
- ✅ Required Tests
- ✅ Rep Dashboard
- ✅ Pharma Company Dashboard
- ✅ Reports System
- ✅ Notifications

### Sprint أكتوبر 2026 (مكتمل جزئياً):
- ✅ Backend نظام النقاط
- ✅ واجهة إدارة نظام النقاط (Web)
- ✅ `PointDashboard` متعدد المنصات
- ✅ تكامل PWA + MAUI
- ✅ Google Login (في الجلسة الأخرى)

### Sprint أكتوبر 2026 (قادم):

**الأسبوع 1:**
- 🔴 اختبار End-to-End شامل
- 🔴 بطاقة مصغّرة للنقاط

**الأسبوع 2:**
- 🔴 لوحة الشركة (Redeem)
- 🟡 واجهة العيادة (Dashboard)

**الأسبوع 3-4:**
- 🟡 Required Tests (تحسينات)
- 🟡 ربط `PROGRAM_COMPLETED` + `LAB_TEST_UPLOADED`

---

## 🎯 خريطة PSP التفصيلية

### ✅ مكتمل:

| # | الميزة | التاريخ |
|---|--------|---------|
| 1 | Program Management (CRUD) | Q1 2026 |
| 2 | ProgramMedication | Q1 2026 |
| 3 | ProgramSpeciality | Q1 2026 |
| 4 | DispensationPlan (خطة صرف أساسية) | Q2 2026 |
| 5 | Invitations System | Q2 2026 |
| 6 | Participations | Q2 2026 |
| 7 | Patients Management | Q2 2026 |
| 8 | eRX (الروشتات الإلكترونية) | Q3 2026 |
| 9 | Dispensations (عمليات الصرف) | Q3 2026 |
| 10 | **Phases (مراحل خطة الصرف)** | Q3 2026 |
| 11 | **Required Tests** | Q3 2026 |
| 12 | **نظام النقاط (Backend)** | Q3 2026 |
| 13 | **نظام النقاط (Frontend)** | Q4 2026 |

### 🎯 قادم:

| # | الميزة | التاريخ المتوقع |
|---|--------|:---------------:|
| 14 | بطاقة مصغّرة للنقاط | Q4 2026 |
| 15 | لوحة الشركة (Redeem) | Q4 2026 |
| 16 | واجهة العيادة (Dashboard) | Q4 2026 |
| 17 | TestUpload (رفع التحاليل) | Q4 2026 |
| 18 | TestReminders (تذكيرات) | Q4 2026 |
| 19 | PharmaPrivacy (خصوصية التحاليل) | Q4 2026 |
| 20 | FollowUps (المتابعات) | Q1 2027 |
| 21 | AdverseEvents (الآثار الجانبية) | Q1 2027 |
| 22 | Reports & Analytics | Q1 2027 |

---

## 🎯 خريطة التطبيقات الأخرى

### 📱 Mobile (MAUI):
- ✅ التسجيل والدخول
- ✅ PSP Program Search
- ✅ Pharmacy Search
- ✅ QR Scanner
- ✅ نظام النقاط (PointDashboard)
- 🎯 Push Notifications (في التطوير)
- 🎯 Deep Linking
- 🎯 Offline Mode

### 🌐 PWA (75%):
- ✅ المسح الضوئي QR
- ✅ إشعارات Push
- ✅ نظام النقاط
- 🔄 45 صفحة متبقية
- 🎯 Service Worker محسّن
- 🎯 Offline First

### 💊 Pharmacy Portal:
- ✅ تسجيل الدخول
- ✅ عرض البرامج
- ✅ صرف الأدوية
- 🎯 تقارير الصيدلية
- 🎯 إحصائيات

### 👨‍⚕️ Doctor Portal:
- ✅ تسجيل الدخول
- ✅ إدارة المرضى
- ✅ الروشتات الإلكترونية
- ✅ نظام النقاط (محفظة)
- 🎯 تقارير الطبيب

### 🏢 Pharma Company Portal:
- ✅ إدارة البرامج
- ✅ إدارة الأدوية
- ✅ إدارة المندوبين
- ✅ لوحة المندوب
- ✅ نظام النقاط (إدارة النظام)
- 🎯 التقارير المتقدمة
- 🎯 إدارة Redeem

### 👤 Patient Portal:
- ✅ تسجيل الدخول
- ✅ عرض البرامج
- ✅ طلب الصرف
- ✅ متابعة الطلبات
- 🎯 رفع التحاليل
- 🎯 التذكيرات

---

## 📊 إحصائيات المشروع

### المشاريع (9):
| # | المشروع | الحالة |
|---|---------|--------|
| 1 | `RubikCare.Domain` | ✅ مستقر |
| 2 | `RubikCare.Application` | ✅ مستقر |
| 3 | `RubikCare.Infrastructure` | ✅ مستقر |
| 4 | `RubikCare.Api.Web` | ✅ مستقر |
| 5 | `RubikCare.Web` | ✅ مستقر |
| 6 | `RubikCare.PWA` | 🔄 75% |
| 7 | `RubikCare.Mobile` | 🔄 في التطوير |
| 8 | `RubikCare.Shared.UI` | ✅ مستقر |
| 9 | `RubikCare.Tests` | ✅ 33 اختبار |

### الجداول في قاعدة البيانات:
- **إجمالي:** ~68 جدول
- **PSP:** 25 جدول (17 + 8 نظام النقاط)
- **Identity:** ~10 جداول
- **Resources:** 1 جدول (~5400 مفتاح)

### الـ Migrations:
- **إجمالي:** ~51 Migration
- **آخر Migration:** `20260929163106_AddPSPProgramPointsTables`

### الكود:
- **إجمالي الأسطر:** ~155,000 سطر
- **الملفات:** ~1,550 ملف

---

## 🎯 مؤشرات الأداء (KPIs)

### Sprint سبتمبر — أكتوبر 2026:
| المقياس | القيمة |
|---------|--------|
| المهام المكتملة | 32 مهمة |
| الأخطاء المُصلحة | 30+ خطأ |
| Migrations جديدة | 1 |
| ملفات جديدة | 60+ ملف |
| جداول جديدة | 8 |
| Handlers جديدة | 19 |
| Controllers جديدة | 5 |
| API Endpoints جديدة | 18 |
| مفاتيح ترجمة جديدة | 60+ مفتاح |

### Sprint أكتوبر 2026 (المستهدف):
| المقياس | الهدف |
|---------|-------|
| المهام المكتملة | 8-10 مهام |
| الأخطاء المُصلحة | 10+ أخطاء |
| Migrations جديدة | 0-1 |
| مفاتيح ترجمة جديدة | 30+ مفتاح |

---

## 🔧 الأدوات المستخدمة

### التطوير:
- **.NET 10** — Framework
- **Visual Studio 2026** — IDE
- **SQL Server 2022** — Database
- **Package Manager Console** — Migrations

### الإدارة:
- **GitHub** — Version Control
- **GitHub Docs Repo** — Documentation
- **PowerShell** — Scripts

### الـ Testing:
- **xUnit** — Unit Tests
- **Architecture Tests** — 33 اختبار

---

## 📝 قواعد التطوير

### 1️⃣ Code-First:
```
❌ لا تعدل Migration قديم
✅ أي تغيير عبر Migration جديدة
✅ Database: RubikCare_Dev_V2
✅ N'' prefix للعربي
```

### 2️⃣ Clean Architecture:
```
❌ Controllers → DbContext مباشر
✅ Controllers → IGenericService<T>
✅ Mobile → HTTP فقط
```

### 3️⃣ UI/UX:
```
❌ Hardcoded Colors
✅ var(--rubik-primary)
❌ Set-Content لملفات .razor
✅ Visual Studio للتحرير
```

### 4️⃣ Translation:
```
❌ نصوص عربية مباشرة
✅ T("KEY") / LocalizedText
✅ MERGE في Resources
✅ Restart بعد إضافة مفاتيح جديدة
```

### 5️⃣ نظام النقاط (جديد):
```
❌ تعديل IDs في PSPActivityTypes
✅ استخدام PointActivityTypeIds
❌ استدعاء EarnPointsHandler من External
✅ استدعاؤه من Handlers أخرى
✅ IsTokenValid قبل إرسال التوكن
```

---

## 🎯 الخطوات القادمة

### 🔴 فوري:
1. **اختبار End-to-End شامل** لنظام النقاط
2. **بطاقة مصغّرة للنقاط** في `ProgramDetailsPage`

### 🟡 قريب:
3. **لوحة الشركة (Redeem)** — إدارة الأرصدة + الـ Redeem
4. **واجهة العيادة (Dashboard)** — عرض شامل
5. **Required Tests (تحسينات)**

### 🟢 متوسط المدى:
6. **ربط `PROGRAM_COMPLETED`**
7. **ربط `LAB_TEST_UPLOADED`**
8. **Reports & Analytics**

---

## 📞 للتواصل

**الفريق:**
- **يوسف شادي** — المطور
- **د. شادي عبداللطيف** — صاحب الفكرة

**المستودع:**
- **Code:** `Rubikans-Egy/Rubikcare.Full.Migration`
- **Docs:** `Rubikans-Egy/Rubikcare-docs`

---

**آخر تحديث:** 3 أكتوبر 2026
**الإصدار:** V2.0

**نهاية الوثيقة 🚀**
```

---

## ✅ ما تم في هذا التحديث

| # | الإضافة | التفاصيل |
|---|---------|----------|
| 1 | **إضافة PWA** | 9 مشاريع بدلاً من 8 |
| 2 | **Sprint سبتمبر — أكتوبر 2026** | 32 مهمة مكتملة |
| 3 | **نظام النقاط** | Backend + Frontend |
| 4 | **Google Login** | ✅ تم حله |
| 5 | **إصلاحات حرجة** | قفزة القائمة + `_pspbundle.css` |
| 6 | **حالة PWA** | 75% |
| 7 | **مكونات النقاط** | `PointDashboard` + `PSPStep2b` |
| 8 | **الجداول** | 68 (بدلاً من 60) |
| 9 | **KPIs محدَّثة** | 60+ ملف + 18 Endpoint |
| 10 | **قاعدة نظام النقاط** | جديدة في قواعد التطوير |
