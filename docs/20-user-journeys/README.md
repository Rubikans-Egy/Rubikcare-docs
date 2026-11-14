# 20 — USER JOURNEYS (رحلات المستخدمين)

الإصدار: 1.0 (Live State Edition)
آخر تحديث: 15 نوفمبر 2026
الحالة: 🟢 حي — يُحدَّث كل جلسة تمس رحلة مستخدم
الجمهور: فريق التطوير + AI Assistants + فريق دعم المرضى + صانعو المحتوى (فيديوهات)
الوثيقة الأم: [20-USER JOURNEY.md](../20-USER%20JOURNEY.md) — النسخة القديمة (Patient-only)

## 🎯 الغرض

هذا المجلد يوثّق **رحلة كل نوع من مستخدمي RubikCare** — بدقة كافية لتكون:

1. **دليل مرجعي للمطورين** — لفهم تدفق العمليات والـ Endpoints.
2. **قاعدة معرفة لـ AI Support Bot** — للإجابة على "كيف أفعل X؟".
3. **سكريبت لفيديوهات شرح التطبيق** — لكل Persona.

## 📚 فهرس الوثائق

| # | الملف | Persona | الأولوية | الحالة |
|---|---|---|---|---|
| 00 | [00-overview.md](00-overview.md) | نظرة عامة (5 Personas) | 🔴 عالية | ✅ |
| 01 | [01-patient.md](01-patient.md) | المريض | 🔴 عالية | ✅ |
| 02 | [02-doctor.md](02-doctor.md) | الطبيب | 🔴 عالية | ✅ |
| 03 | [03-pharmacist.md](03-pharmacist.md) | الصيدلي | 🔴 عالية | ✅ |
| 04 | [04-rep.md](04-rep.md) | المندوب | 🟠 متوسطة | ✅ |
| 05 | [05-public-user.md](05-public-user.md) | المستخدم العام | 🟠 متوسطة | ✅ |
| 06 | [06-cross-cutting.md](06-cross-cutting.md) | ميزات مشتركة (Offline, Translation, Navigation, Messaging, Commerce) | 🔴 عالية | ✅ |

## 🎯 5 Personas (نظرة سريعة)

| Persona | الحساب | Dashboards | رحلته الأساسية |
|---|---|---|---|
| **Patient** | مستخدم عام (UserProfile) | `DashboardFlowPage` (Personal) | تسجيل → PSP → صرف → نقاط |
| **Doctor** | Doctor role | `ClinicDashboardPage` | تسجيل → عضوية عيادة → دعوة مرضى → متابعة |
| **Pharmacist** | Pharmacist role | `PharmacyDashboardPage` | تسجيل → عضوية صيدلية → صرف توكنات → B2B |
| **Rep** | Rep flag | `PharmaCompanyDashboardPage` | تسجيل → عضوية شركة → دعوة أطباء/صيادلة → شبكة |
| **Public User** | UserProfile (بدون عضوية) | `DashboardFlowPage` (Personal) | تسجيل → بحث طبيب/صيدلية → حجز → محادثة |

## 🔗 الوثائق المرتبطة

- **[00-MASTER-INDEX.md](../00-MASTER-INDEX.md)** — الفهرس الشامل.
- **[01-ARCHITECTURE.md](../01-ARCHITECTURE.md)** — الهيكل المعماري.
- **[02-IDENTITY.md](../02-IDENTITY.md)** — المصادقة والأدوار.
- **[03-MENUS.md](../03-MENUS.md)** — القوائم الديناميكية.
- **[04-TRANSLATION.md](../04-TRANSLATION.md)** — الترجمة + RTL + Offline.
- **[05-PSP.md](../05-PSP.md)** — نظام دعم المرضى.
- **[06-POINTS.md](../06-POINTS.md)** — نظام النقاط.
- **[08-NOTIFICATIONS.md](../08-NOTIFICATIONS.md)** — الإشعارات.
- **[13-MAUI.md](../13-MAUI.md)** — تطوير MAUI.
- **[19-PERMISSIONS.md](../19-PERMISSIONS.md)** — الصلاحيات (مستقبلاً).
- **[23-COMMERCE-ENGINE.md](../23-COMMERCE-ENGINE.md)** — B2B Commerce.

## 📋 ملاحظات مهمة

### ⚠️ المنصات الثلاث
كل رحلة موثقة لـ **3 منصات**:
- **Web** (`Rubikcare.Web` — Blazor Server) — للمسؤولين والموظفين.
- **PWA** (`RubikCare.PWA` — Blazor WASM) — نسخة كربونية من Mobile.
- **MAUI** (`RubikCare.Mobile` — .NET MAUI) — Android + iOS.

### ⚠️ Offline Mode
كل رحلة موثقة **بافتراضين**:
- **Online** — كل شيء يعمل.
- **Offline** — ما يعمل، وما لا يعمل، وما يظهر للمستخدم.

### ⚠️ الترجمة (AR/EN) + RTL
- **العربية** هي اللغة الافتراضية.
- **الإنجليزية** مدعومة بالكامل.
- **RTL** يعمل تلقائياً حسب اللغة.

## 🔄 الاختلاف عن `20-USER JOURNEY.md` القديمة

| الجانب | القديمة (v1.0) | الجديدة (v2.0) |
|---|---|---|
| **النطاق** | المريض فقط | 5 Personas |
| **المنصات** | لم تُذكر | 3 منصات (Web, PWA, MAUI) |
| **Offline** | لم تُذكر | موثق لكل شخصية |
| **الترجمة** | لم تُذكر | موثقة |
| **Cross-Cutting** | لا | نعم (`06-cross-cutting.md`) |
| **الأداء** | لا | نعم (After Session #107) |
| **Messaging** | لا | نعم |
| **Commerce (B2B)** | لا | نعم |

© 2026 RubikCare — للاستخدام الداخلي
