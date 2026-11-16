
# 16 — MAUI Release Deployment & R8 Troubleshooting

**الإصدار:** 1.0
**آخر تحديث:** 16 نوفمبر 2026
**الحالة:** ✅ معتمد — يحل مشاكل بناء الـ Release ونشر Google Play
**الجمهور:** فريق التطوير + AI Assistants
**المرجع الإلزامي:** يُقرأ قبل أي بناء لنسخة `Release` أو نشر على Google Play.

## 📌 الجزء صفر: مقدمة

هذه الوثيقة هي خلاصة "معركة" بناء نسخة الـ `Release` لتطبيق `RubikCare.Mobile`. واجهنا فيها انهيارات متتالية (SIGABRT، ClassNotFoundException، icall.c Assertion) عند بناء التطبيق في وضع Release بعد تفعيل أدوات التحسين (R8) لتلبية متطلبات Google Play. 

**القاعدة الذهبية:** Debug يعمل ≠ Release سيعمل. أدوات R8 والـ Linker تتخلص من الكود الذي يبدو "غير مستخدم"، وهذا يكسر مكتبات MAUI التي تعتمد على Reflection (مثل BlazorWebView و DI).

## 🚨 الجزء الأول: الأخطاء التي واجهناها وأسبابها الجذرية

واجهنا 3 أخطاء متتالية عند محاولة تشغيل نسخة Release على هاتف Samsung SM-S9080:

| # | الخطأ (في Logcat) | السبب الجذري |
|---|---|---|
| 1 | `java.lang.ClassNotFoundException: androidx.core.app.CoreComponentFactory` | R8 قام بحذف هذا الكلاس لأنه غير مستدعى مباشرة في كود Java، لكن نظام Android يبحث عنه في `AndroidManifest.xml` لحظة إقلاع التطبيق. |
| 2 | `monodroid: * Assertion at icall.c:6261` | تعارض بين `<PublishTrimmed>false</PublishTrimmed>` و `<AndroidLinkMode>SdkOnly</AndroidLinkMode>`. محرك Mono في .NET 10 يحتاج `PublishTrimmed=true` لتهيئة دوال `[UnmanagedCallersOnly]`. |
| 3 | `Fatal signal 6 (SIGABRT)` | الانهيار النهائي نتيجة تراكم الأخطاء أعلاه، أو بسبب `AndroidR8ObfuscationMode private-members` الذي يغير أسماء كلاسات JNI التي تربط C# بـ Java. |

## 🛠️ الجزء الثاني: الإعدادات الصحيحة (الحل النهائي)

لضمان بناء نسخة Release تعمل بسلاسة وتحقق نسبة Obfuscation تتجاوز 25% (متطلبات Google Play)، يجب أن يكون ملف `RubikCare.Mobile.csproj` كالتالي:

### 1️⃣ قسم إعدادات Release (داخل `<PropertyGroup>` الخاص بـ Release)
```xml
<PropertyGroup Condition="'$(Configuration)|$(TargetFramework)|$(Platform)'=='Release|net10.0-android|AnyCPU'">
    <!-- أدوات R8 -->
    <AndroidLinkTool>r8</AndroidLinkTool>
    <AndroidUseFastDeployment>false</AndroidUseFastDeployment>
    <EmbedAssembliesIntoApk>true</EmbedAssembliesIntoApk>
    
    <!-- ⚠️ يجب أن تكون true لحل مشكلة SIGABRT في MAUI -->
    <AndroidUseAssemblyStore>true</AndroidUseAssemblyStore>
    
    <!-- ملف حماية R8 -->
    <AndroidProguardConfiguration>Platforms\Android\proguard-rules.pro</AndroidProguardConfiguration>
    <AndroidCreateProguardMappingFile>true</AndroidCreateProguardMappingFile>

    <!-- صيغة النشر (اجعلها apk للتجربة، و aab للرفع النهائي) -->
    <AndroidPackageFormat>aab</AndroidPackageFormat>
    
    <!-- إعدادات التوقيع (احتفظ ببياناتك الخاصة) -->
    <AndroidKeyStore>True</AndroidKeyStore>
    <AndroidSigningKeyStore>E:\Keys\APK Rubikcare keystore\Rubikcare.keystore</AndroidSigningKeyStore>
    <AndroidSigningStorePass>xxxxxxxx</AndroidSigningStorePass>
    <AndroidSigningKeyAlias>rubikcare</AndroidSigningKeyAlias>
    <AndroidSigningKeyPass>xxxxxxx</AndroidSigningKeyPass>
</PropertyGroup>
```

### 2️⃣ قسم إعدادات Trimming (خارج قسم Release، أو مشروط بـ Release)
*(هذا القسم هو الذي حل مشكلة `icall.c` نهائياً)*
```xml
<PropertyGroup Condition="'$(Configuration)' == 'Release'">
    <PublishTrimmed>true</PublishTrimmed>
    <TrimMode>partial</TrimMode>
    <AndroidEnableProfiledAot>false</AndroidEnableProfiledAot>
    <RunAOTCompilation>false</RunAOTCompilation>
</PropertyGroup>
```
*(⚠️ هام: احذف أي سطر `AndroidLinkMode` من ملف الـ `.csproj` بالكامل، ولا تستخدم `AndroidR8ObfuscationMode` مع MAUI لأنه يكسر الربط).*

### 3️⃣ ملف `Platforms/Android/proguard-rules.pro`
يجب إنشاء هذا الملف (أو تعديله) ليكون كالتالي بالضبط:

```proguard
# 1. حماية مصنع مكونات AndroidX المطلوب لحظة الإقلاع (يحل خطأ CoreComponentFactory)
-keep class androidx.core.app.CoreComponentFactory { *; }
-keep class androidx.core.app.** { *; }
-keep class androidx.core.content.** { *; }
-keep class androidx.lifecycle.** { *; }
-keep class androidx.startup.** { *; }

# حماية جميع مكونات AndroidManifest (Activities, Receivers, Services, Providers)
-keep public class * extends android.app.Activity
-keep public class * extends android.app.Application
-keep public class * extends android.app.Service
-keep public class * extends android.content.BroadcastReceiver
-keep public class * extends android.content.ContentProvider

# 2. حماية مكونات .NET MAUI الأساسية وعمليات الربط (JNI & Mono)
-keep class mono.** { *; }
-keep class crc64** { *; }
-keep class microsoft.maui.** { *; }
-keep class com.microsoft.maui.** { *; }

-keepclasseswithmembernames class * {
    native <methods>;
}

# 3. حماية BlazorWebView و JavaScript Interop
-keep class android.webkit.** { *; }
-keepclassmembers class * {
    @android.webkit.JavascriptInterface <methods>;
}

# 4. حماية AndroidX والبيانات المشفرة (SecureStorage) و Parcelable
-keep class androidx.security.crypto.** { *; }
-keep class * implements android.os.Parcelable {
    public static final ** CREATOR;
}

# 5. حماية Firebase Messaging (الإشعارات) و LocalNotification
-keep class com.google.firebase.** { *; }
-keep class plugin.LocalNotification.** { *; }

# 6. حماية قارئ الباركود ZXing والكاميرا
-keep class com.google.zxing.** { *; }
-keep class androidx.camera.** { *; }

# 7. إعدادات تحسين وتعتيم R8 (لرفع النسبة فوق 25% وتجاوز تحذير Google Play)
-repackageclasses 'o'
-allowaccessmodification
-optimizations !code/simplification/arithmetic,!code/simplification/cast,!field/*,!class/merging/*
```

*(⚠️ هام: في ملف الـ `.csproj`، تأكد أن خصائص الملف `proguard-rules.pro` مضبوطة على `ProguardConfiguration`).*

## 🧹 الجزء الثالث: خطوات البناء الإلزامية (Checklist)

عند الانتقال من Debug إلى Release، لا تعتمد على الـ Build العادي. اتبع الآتي حرفياً:
- [ ] احذف الحزم التي لا تنتمي للموبايل من مشروع `RubikCare.Mobile` (مثل `FirebaseAdmin` — فهي للسيرفر، وحذفها يقلل حجم الكود).
- [ ] أغلق Visual Studio تماماً.
- [ ] احذف مجلدات `bin` و `obj` يدوياً من مشروع الموبايل (لإجبار R8 على إعادة توليد الملفات بالقواعد الجديدة).
- [ ] امسح التطبيق القديم من الهاتف (Uninstall).
- [ ] أعد فتح Visual Studio وقم بعمل `Rebuild` ثم `Publish`.
- [ ] **للتجربة المحلية:** غيّر `<AndroidPackageFormat>` إلى `apk` وثبت الملف على الهاتف.
- [ ] **للرفع النهائي:** غيّرها إلى `aab`، ثم ارفع الملف مع ملف `mapping.txt` إلى Google Play Console (في خانة Deobfuscation file).

## 💡 الجزء الرابع: دروس مستفادة (Lessons Learned)

1. **R8 والـ Reflection أعداء:** أي كلاس يُستدعى عبر `Reflection` أو `JNI` (مثل `CoreComponentFactory` أو `maui` أو `firebase`) يجب أن يُضاف إلى `proguard-rules.pro` وإلا سيختفي.
2. **لا تخلط بين `PublishTrimmed` و `AndroidLinkMode`:** في .NET 10 MAUI، الاعتماد على `PublishTrimmed=true` مع `TrimMode=partial` هو المسار الآمن الوحيد.
3. **`AndroidR8ObfuscationMode` ممنوع:** لا تستخدمه مع MAUI لأنه يكسر الربط بين C# و Java.
4. **`AndroidUseAssemblyStore=true` ضروري:** قيمته `false` في بعض الأجهزة (مثل Samsung) تسبب انهيار `SIGABRT` عند فك ضغط DLLs.
5. **حذف `FirebaseAdmin` و `System.Drawing.Common`:** هاتان الحزمتان مخصصتان للسيرفر/ويندوز، ووجودهما في مشروع الموبايل يضخم الحجم ويسبب مشاكل في الـ DEX.

---
