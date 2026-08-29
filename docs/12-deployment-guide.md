
# 📖 12 - دليل النشر والإنتاج الشامل (Deployment Guide)

**آخر تحديث:** 29 أغسطس 2026  
**الحالة:** ✅ محدّث ليشمل Blazor WebAssembly PWA  
**المؤلفون:** فريق RubikCare

---

## 📌 مقدمة

هذا المرجع الشامل يغطي **كل ما يتعلق بنشر منصة RubikCare** على جميع البيئات:

- **Web (IIS)** - Blazor Server + Blazor WebAssembly PWA
- **Mobile (MAUI)** - Android APK/AAB + iOS IPA
- **API (ASP.NET Core)** - Backend REST API

### 🎯 الأهداف

1. **توحيد إجراءات النشر** عبر جميع البيئات
2. **توثيق الدروس المستفادة** من المشاكل الحقيقية
3. **توفير سكربتات أتمتة** لتقليل الأخطاء اليدوية
4. **إنشاء قوائم تحقق** لضمان الجودة

### 📚 الوثائق المرجعية

قبل البدء، يُفضل مراجعة:
- [00 - الهيكل المعماري](00-architecture-overview.md) - لفهم بنية المشاريع
- [01 - Program.cs والتسجيلات الأساسية](01-program-cs-foundation.md) - لفهم الخدمات
- [02 - نظام الهوية والمصادقة](02-identity-system.md) - لفهم إدارة الجلسات

---

## 🗺️ القسم 1: خريطة البيئات (Environment Map)

### 🌍 البيئات المعتمدة

| البيئة | الغرض | رابط Web | رابط API | مسار السيرفر |
|--------|-------|----------|----------|--------------|
| **Local** | التطوير والاختبار المبدئي | `https://localhost:xxxx` | `https://localhost:yyyy` | `C:\Users\{user}\source\repos\...` |
| **Test / UAT** | الاختبار الشامل قبل النشر | `https://test.rubikcare.com` | `https://uat.rubikcare.com` | Web: `C:\WebSite\RubikCareNew`<br>API: `C:\WebSite\RubikCareUat` |
| **Stage (PWA)** | بيئة PWA للاختبار | `https://stagepu.rubikcare.com` | `https://uat.rubikcare.com` | PWA: `C:\WebSite\PU_RubicCareStage`<br>API: `C:\WebSite\RubikCareUat` |
| **Production** | البيئة الفعلية للمستخدمين | `https://rubikcare.com` | `https://api.rubikcare.com` | Web: `C:\WebSite\RubikCareLive`<br>API: `C:\WebSite\RubikCareApi` |

### ⚠️ قاعدة ذهبية

> **تطبيق الـ Web/PWA في بيئة الـ Test/Stage يجب أن يكون `ApiSettings:BaseUrl` فيه مساوياً لـ `https://uat.rubikcare.com` وليس الـ Live.**

---

## 🏗️ القسم 2: البنية التحتية المشتركة

### 📋 المتطلبات الأساسية

| المكون | الإصدار الأدنى | الغرض |
|--------|----------------|-------|
| Windows Server | 2019+ | نظام التشغيل |
| IIS | 10+ | خادم الويب |
| .NET Runtime | 10.0+ | تشغيل التطبيقات |
| SQL Server | 2019+ | قاعدة البيانات |
| URL Rewrite Module | 2.1+ | إعادة كتابة الروابط (ضروري لـ SPA) |

### 🔧 إعداد IIS الأساسي

#### 1. تثبيت URL Rewrite Module

```powershell
# التحقق من وجود الوحدة
Get-WebGlobalModule | Where-Object { $_.Name -like "*Rewrite*" }

# إذا لم تظهر، ثبّت من:
# https://www.iis.net/downloads/microsoft/url-rewrite

# التحقق بعد التثبيت
Test-Path "C:\Windows\System32\inetsrv\rewrite.dll"
```

#### 2. إنشاء Application Pool

```powershell
# Blazor Server / API (ASP.NET Core)
C:\Windows\System32\inetsrv\appcmd add apppool `
    /name:"RubikCareApi" `
    /managedRuntimeVersion:"" `
    /managedPipelineMode:"Integrated"

# Blazor WebAssembly PWA (Static Files)
C:\Windows\System32\inetsrv\appcmd add apppool `
    /name:"PU_RubicCareStage" `
    /managedRuntimeVersion:"" `
    /managedPipelineMode:"Integrated"
```

**ملاحظة مهمة:** `managedRuntimeVersion:""` يعني "No Managed Code" - ضروري لـ ASP.NET Core.

#### 3. منح الصلاحيات

```powershell
# منح صلاحيات القراءة للمجلد
icacls "C:\WebSite\RubikCareApi" /grant "IIS_IUSRS:(OI)(CI)RX" /T
icacls "C:\WebSite\PU_RubicCareStage" /grant "IIS_IUSRS:(OI)(CI)RX" /T

# منح صلاحيات الكتابة لمجلد الرفع (إذا لزم)
icacls "C:\WebSite\RubikCareApi\wwwroot\uploads" /grant "IIS_IUSRS:(OI)(CI)M" /T
```

---

