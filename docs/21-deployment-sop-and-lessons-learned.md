
# 📄 21-deployment-sop-and-lessons-learned.md

```markdown
# 🚀 RubikCare — Deployment SOP & Lessons Learned

**Document Date:** August 29, 2026  
**Status:** ✅ Updated with PWA deployment lessons  
**Maintainers:** RubikCare Team

---

## 📌 Table of Contents

1. [Root Cause Analysis: Silent Failure](#1-root-cause-analysis-silent-failure)
2. [Environment Map](#2-environment-map)
3. [Code Fixes Applied](#3-code-fixes-applied)
4. [Standard Operating Procedure (SOP)](#4-standard-operating-procedure-sop)
5. [Troubleshooting Cheatsheet](#5-troubleshooting-cheatsheet)
6. [Lessons Learned from PWA Deployment](#6-lessons-learned-from-pwa-deployment)
7. [Critical Prohibitions](#7-critical-prohibitions)

---

## 1. Root Cause Analysis: Silent Failure

**Date:** August 20, 2026  
**Developer:** Youssef Shady  
**Status:** Resolved

### Problem Description

A "Silent Failure" issue appeared when adding program links on the Test server's edit page, with no error messages displayed in the UI or browser console.

### Root Cause (3 Factors Combined)

| # | Factor | Details |
|---|--------|---------|
| 1 | **Swallowed Exceptions** | `GetAsync` and `PostAsync` methods in `WebApiService.cs` were returning `null` silently on request failure (404/500) instead of throwing an `Exception`, preventing Blazor from displaying error messages. |
| 2 | **Misconfigured BaseUrl** | `appsettings.Test.json` on the server (and locally) was pointing to `https://api.rubikcare.com` (Live API) instead of the Test/UAT environment. |
| 3 | **API Environment Conflict** | Multiple API versions existed on the server (Live and UAT) without clear documentation, causing new Web code to be deployed while targeting the old Live API that didn't have the `/api/psp/links` endpoint. |

### Solution Applied

```csharp
// ❌ BEFORE: Silent failure
public async Task<T> GetAsync<T>(string endpoint)
{
    var response = await _httpClient.GetAsync(endpoint);
    if (!response.IsSuccessStatusCode)
        return default; // Silent failure!
    return await response.Content.ReadFromJsonAsync<T>();
}

// ✅ AFTER: Explicit exception
public async Task<T> GetAsync<T>(string endpoint)
{
    var response = await _httpClient.GetAsync(endpoint);
    
    if (!response.IsSuccessStatusCode)
    {
        var errorContent = await response.Content.ReadAsStringAsync();
        throw new ApiException($"Request failed: {(int)response.StatusCode} - {errorContent}");
    }
    
    return await response.Content.ReadFromJsonAsync<T>();
}
```

---

## 2. Environment Map

**Critical Rule:** Always verify which environment you're deploying to before any deployment.

| Environment | Purpose | Web URL | API URL | Server Path |
|-------------|---------|---------|---------|-------------|
| **Local** | Development & initial testing | `https://localhost:xxxx` | `https://localhost:yyyy` | `C:\Users\{user}\source\repos\...` |
| **Test / UAT** | Comprehensive testing before production | `https://test.rubikcare.com` | `https://uat.rubikcare.com` | Web: `C:\WebSite\RubikCareNew`<br>API: `C:\WebSite\RubikCareUat` |
| **Stage (PWA)** | PWA testing environment | `https://stagepu.rubikcare.com` | `https://uat.rubikcare.com` | PWA: `C:\WebSite\PU_RubicCareStage`<br>API: `C:\WebSite\RubikCareUat` |
| **Production (Live)** | Actual production environment | `https://rubikcare.com` | `https://api.rubikcare.com` | Web: `C:\WebSite\RubikCareLive`<br>API: `C:\WebSite\RubikCareApi` |

### ⚠️ Golden Rule

> **The Web/PWA app in Test/Stage environment MUST have `ApiSettings:BaseUrl` equal to `https://uat.rubikcare.com`, NOT the Live API.**

---

## 3. Code Fixes Applied

### 3.1 WebApiService.cs (Web Project)

Modified `GetAsync` and `PostAsync` methods to throw exceptions with StatusCode and actual error content instead of returning `default` silently.

### 3.2 PSPStep_Links.razor

Added guard clause to ensure `ProgramID` is not `0` before submission, with user guidance to save the base program first.

### 3.3 web.config (Test Server)

```xml
<environmentVariables>
  <environmentVariable name="ASPNETCORE_ENVIRONMENT" value="Test" />
</environmentVariables>
```

Also enabled `stdoutLogEnabled="true"` to help track authentication errors (Tokens) in the future.

---

## 4. Standard Operating Procedure (SOP)

### 4.1 Deploying API Updates (Always First)

```powershell
# Step 1: Stop the App Pool
C:\Windows\System32\inetsrv\appcmd stop apppool "RubikCareUat"

# Step 2: Backup current appsettings
Copy-Item "C:\WebSite\RubikCareUat\appsettings.Test.json" `
    "C:\WebSite\RubikCareUat\appsettings.Test.json.backup" -Force

# Step 3: Copy new files (excluding environment files)
Get-ChildItem "E:\Publish\ApiUat" -Exclude "appsettings.*.json" | 
    Copy-Item -Destination "C:\WebSite\RubikCareUat" -Recurse -Force

# Step 4: Start the App Pool
C:\Windows\System32\inetsrv\appcmd start apppool "RubikCareUat"
```

### 4.2 Deploying Web Updates

```powershell
# Step 1: Stop the App Pool
C:\Windows\System32\inetsrv\appcmd stop apppool "rubikcarenew"

# Step 2: Backup environment files
Copy-Item "C:\WebSite\RubikCareNew\web.config" `
    "C:\WebSite\RubikCareNew\web.config.backup" -Force
Copy-Item "C:\WebSite\RubikCareNew\appsettings.Test.json" `
    "C:\WebSite\RubikCareNew\appsettings.Test.json.backup" -Force

# Step 3: Copy new files (excluding environment files)
Get-ChildItem "E:\Publish\WebTest" -Exclude "web.config","appsettings.*.json" | 
    Copy-Item -Destination "C:\WebSite\RubikCareNew" -Recurse -Force

# Step 4: Start the App Pool
C:\Windows\System32\inetsrv\appcmd start apppool "rubikcarenew"
```

> ⚠️ **Critical Warning:** If prompted to replace `web.config` or `appsettings.Test.json`, verify the local version is updated (points to `uat.rubikcare.com`), or choose **Skip** to preserve server's manual settings.

---

## 5. Troubleshooting Cheatsheet

```powershell
# 1. Verify Web app is reading correct API URL
Get-Content "C:\WebSite\RubikCareNew\appsettings.Test.json" | Select-String "BaseUrl"

# 2. Verify Test environment is active in web.config
Get-Content "C:\WebSite\RubikCareNew\web.config" | Select-String "ASPNETCORE_ENVIRONMENT"

# 3. Check latest Blazor/Server errors in logs
Get-ChildItem -Path "C:\WebSite\RubikCareNew\logs\stdout" -Filter "*.log" | 
    Sort-Object LastWriteTime -Descending | 
    Select-Object -First 1 | 
    Get-Content -Tail 30

# 4. Check Application Event Log
Get-EventLog -LogName Application -Source "*ASP.NET*" -Newest 10 | Format-List
```

---

## 6. Lessons Learned from PWA Deployment

### 6.1 Lesson: Hashed Filenames Cause SRI Failures

**Problem:** After deploying PWA, blue screen appeared with `SRI integrity checks failed` error.

**Root Cause:** Blazor files have hashed filenames that change with each publish. Copying new files over old ones without clearing the folder left old files with old hashes.

**Solution:**
```powershell
# Always clear the folder before copying (except web.config)
Get-ChildItem "C:\WebSite\PU_RubicCareStage" -Exclude "web.config" | 
    Remove-Item -Recurse -Force

# Then copy new files
Copy-Item -Path "E:\rubikans\Publish\PWA\*" `
    -Destination "C:\WebSite\PU_RubicCareStage\" -Recurse -Force
```

### 6.2 Lesson: Physical Path Must Be Root, Not wwwroot

**Problem:** Framework files returning 404 despite existing on server.

**Root Cause:** Physical path was pointing to `wwwroot`, causing the `Serve subdir` rule in `web.config` to look for `wwwroot\wwwroot\...`.

**Solution:**
```powershell
# Physical path MUST be root, not wwwroot
C:\Windows\System32\inetsrv\appcmd set vdir "PU_RubicCareStage/" `
    /physicalPath:"C:\WebSite\PU_RubicCareStage"
```

### 6.3 Lesson: URL Rewrite Module is Mandatory

**Problem:** Site returns 500 error immediately after deployment.

**Root Cause:** `web.config` contains `<rewrite>` section that IIS doesn't understand without URL Rewrite Module.

**Solution:**
```powershell
# Check if module is installed
Get-WebGlobalModule | Where-Object { $_.Name -like "*Rewrite*" }

# If not present, install from:
# https://www.iis.net/downloads/microsoft/url-rewrite

# Then recycle the App Pool
C:\Windows\System32\inetsrv\appcmd recycle apppool "PU_RubicCareStage"
```

### 6.4 Lesson: Service Worker Caching

**Problem:** App works in regular browser but not in InPrivate (or vice versa).

**Root Cause:** Service Worker stored in regular browser is serving old files.

**Solution:**
1. **Always test in InPrivate window:** `Ctrl + Shift + N`
2. Or manually clear Service Worker:
   - `F12` → **Application** → **Service Workers** → **Unregister**
   - `F12` → **Application** → **Storage** → **Clear site data**

---

## 7. Critical Prohibitions

| # | Prohibition | Reason | Alternative |
|---|-------------|--------|-------------|
| 1 | ❌ Delete `web.config` during deployment | Contains IIS settings (MIME + Rewrite) | Always keep it |
| 2 | ❌ Copy files over old ones without clearing | Hashed filename conflicts → `SRI failed` | Clear folder first, then copy |
| 3 | ❌ Manually edit `web.config` on server | Will be lost in next deployment | Make it part of the project |
| 4 | ❌ Change Physical Path to `wwwroot` | Conflicts with `Serve subdir` rule | Physical Path = Root always |
| 5 | ❌ Test in regular browser after new deployment | Service Worker serves old files | Always test in InPrivate |
| 6 | ❌ Deploy without stopping App Pool | File locks and copy failures | Stop App Pool first |
| 7 | ❌ Deploy API to PWA folder | They are completely separate apps | Each has its own folder |

---

## 📋 Master Deployment Checklist

### Before Any Deployment

- [ ] All code changes saved?
- [ ] Application tested locally?
- [ ] Database backup taken?
- [ ] Environment files (`appsettings.{Environment}.json`) point to correct URLs?

### During Deployment

- [ ] App Pool stopped before copying files?
- [ ] Backup taken of `web.config` and `appsettings`?
- [ ] Files copied successfully?

### After Deployment

- [ ] App Pool started?
- [ ] Health Check returns `200`?
- [ ] Tested in InPrivate window (for PWA)?
- [ ] Logs free of errors?
- [ ] Core operations working (login, save, read)?

---

## 🔗 Related Documents

- [12 - Deployment Guide](12-deployment-guide.md)
- [22 - RubikCare.PWA](22-RubikCare.PWA.md)
- [00 - Architecture Overview](00-architecture-overview.md)

---

**Last Updated:** August 29, 2026 | **File:** `21-deployment-sop-and-lessons-learned.md`
```

---

## ✅ الخلاصة

تم إنشاء الوثيقة الجديدة بالإنجليزية:
- **الاسم:** `21-deployment-sop-and-lessons-learned.md`
- **المحتوى:** 7 أقسام رئيسية تغطي:
  1. تحليل السبب الجذري للفشل الصامت
  2. خريطة البيئات
  3. الإصلاحات البرمجية
  4. إجراءات النشر الآمنة (SOP)
  5. أوامر التشخيص السريع
  6. الدروس المستفادة من نشر PWA
  7. المحاذير الحرجة
