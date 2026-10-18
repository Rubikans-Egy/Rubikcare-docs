# 08 — NOTIFICATIONS (نظام الإشعارات والتذكيرات)

**الإصدار:** 3.1 (Full Merge Edition)
**آخر تحديث:** 17 أكتوبر 2026
**الحالة:** 🟡 شغال — نقاط مفتوحة
**الجمهور:** فريق التطوير + AI Assistants
**المرجع الإلزامي:** يُقرأ قبل أي تعديل على الإشعارات

---

## 🗂️ نظرة عامة — عندنا 3 أنظمة منفصلة تمامًا

المشروع فيه أنظمة إشعارات مختلفة في الآلية والغرض، ومهم عدم الخلط بينهم:

| # | النظام | الآلية | الحالة |
|---|--------|--------|--------|
| 1 | **إشعارات الطلبات المهنية** (Professional Requests) | Push Notification عبر Firebase (FCM) — يحتاج اتصال إنترنت وسيرفر | ✅ شغال بالكامل |
| 2 | **تذكيرات مواعيد الأدوية** (Medication Schedule Reminders) | Local Notification/Alarm عبر `Plugin.LocalNotification` — مجدول محليًا على الجهاز | ✅ شغال بس فيه ملاحظتين مفتوحتين |
| 3 | **إشعارات الويب (Web Push)** | Web Push + VAPID عبر Service Worker | ✅ شغال (مشكلة ترميز عربي) |

---

## 1️⃣ نظام إشعارات الطلبات المهنية (Push / Firebase)

### الحالة: ✅ شغال بالكامل

**الوظيفة:**
- بيتبعت إشعار للأدمن لما مستخدم يبعت "طلب مهني" (Professional Request) — مثلاً طلب فتح عيادة/صيدلية
- بيتبعت إشعار لصاحب الطلب لما الأدمن يوافق عليه
- الاتنين قابلين للضغط (Tappable) من صفحة الإشعارات جوه التطبيق، وكل واحد بيوجّه (Navigate) للصفحة المناسبة:
  - النوع الأول → بيفتح المتصفح (Browser) ✅
  - النوع الثاني → بيوجّه لصفحة الطلب جوه التطبيق نفسه، واللي بيكون بقى معتمد يقدر ينشئ العيادة/الصيدلية بتاعته من نفس المكان ✅

**الملفات المسؤولة:**
```
Mobile\Services\MauiNotificationNavigationService.cs        ← تنفيذ التوجيه على الموبايل
Rubikcare.Web\Services\WebNotificationNavigationService.cs  ← تنفيذ التوجيه على الويب
Shared.UI\Services\INotificationNavigationService.cs        ← الـ Interface المشترك
```

**Commits ذات الصلة:**
```
16129b9 — 2026-07-28 — ProfesstionalMode Notification for Admin & user
a864b8b — 2026-07-26 — Push Notification After Add Member to Organization
b71c635 — 2026-07-26 — Push Notification From Fire BASE
```

**⚠️ لا حاجة للعمل على هذا النظام حاليًا — مغلق ومستقر.**

---

## 2️⃣ نظام تذكيرات مواعيد الأدوية (Local Alarm Reminders)

### الحالة: ✅ الأساسيات شغالة — فيه نقطتين مفتوحتين

### 🏗️ البنية المعمارية

```
Shared.UI\Services\ILocalNotificationService.cs          ← الـ Interface المشترك (Web + Mobile)
Mobile\Infrastructure\Services\MauiLocalNotificationService.cs  ← التنفيذ الحقيقي (Mobile فقط)
Rubikcare.Web\Services\WebLocalNotificationService.cs     ← تنفيذ No-Op (الويب مش محتاج Local Alarms)
Mobile\MauiProgram.cs                                      ← تسجيل الـ Plugin + Notification Channel
Mobile\Platforms\Android\MainActivity.cs                   ← طلب صلاحيات الإشعارات + Exact Alarm
Shared.UI\Components\Patient\SchedulePage.razor            ← نقطة الاستدعاء (بعد كل LoadAllData)
```

### 🔄 آلية العمل

1. أي تغيير في بيانات المواعيد (تحميل الصفحة / إضافة دواء / تعديل / تغيير حالة / توليد PSP) بينادي `LoadAllData()`
2. في نهاية `LoadAllData()` بتتنادى `NotificationService.SyncReminders(_schedules)`
3. `SyncReminders` بتعمل الآتي لكل موعد (Idempotent — باستخدام `NotificationId = ScheduleID` كمفتاح فريد **يستبدل القديم تلقائيًا**):
   - لو `Status == "Pending"` والموعد في المستقبل → `ScheduleDoseReminder` (يجدول/يحدّث التنبيه)
   - غير كده (Taken/Skipped أو الموعد فات) → `CancelReminder` (يلغي أي تنبيه قديم)
4. `ScheduleDoseReminder` بتحسب `NotifyTime = ScheduledTime - reminderMinutesBefore` وتستخدم `Plugin.LocalNotification` لجدولة Notification بـ:
   - `CategoryType = Alarm`
   - `ChannelId = "rubikcare_medication"` (channel بـ Importance = High معرّف في `MauiProgram.cs`)

### ✅ الكود الحالي الكامل (آخر نسخة موثقة)

**`ILocalNotificationService.cs`:**
```csharp
namespace RubikCare.Shared.UI.Services;

public interface ILocalNotificationService
{
    Task ScheduleDoseReminder(int scheduleId, string medicationName, string dosage, DateTime scheduledTime, int reminderMinutesBefore = 0);
    Task CancelReminder(int scheduleId);
    Task SyncReminders(IEnumerable<RubikCare.Application.DTOs.Patient.MedicationScheduleDto> schedules, int reminderMinutesBefore = 0);
}
```

**`MauiLocalNotificationService.cs`:**
```csharp
using Plugin.LocalNotification;
using RubikCare.Application.DTOs.Patient;
using RubikCare.Shared.UI.Services;
using System.Diagnostics;

namespace RubikCare.Mobile.Services;

public class MauiLocalNotificationService : ILocalNotificationService
{
    public async Task ScheduleDoseReminder(
        int scheduleId, string medicationName, string dosage,
        DateTime scheduledTime, int reminderMinutesBefore = 0)
    {
        var notifyTime = scheduledTime.AddMinutes(-reminderMinutesBefore);
        if (notifyTime <= DateTime.Now)
        {
            Debug.WriteLine($"❌ SKIPPED: notifyTime already passed for {medicationName}");
            return;
        }
        var body = string.IsNullOrEmpty(dosage) ? medicationName : $"{medicationName} - {dosage}";
        var notification = new NotificationRequest
        {
            NotificationId = scheduleId,
            Title = "💊 تذكير دواء",
            Description = body,
            CategoryType = NotificationCategoryType.Alarm,
            Schedule = new NotificationRequestSchedule { NotifyTime = notifyTime },
            Android = new Plugin.LocalNotification.AndroidOption.AndroidOptions
            {
                ChannelId = "rubikcare_medication",
                Priority = Plugin.LocalNotification.AndroidOption.AndroidPriority.High
            }
        };
        await LocalNotificationCenter.Current.Show(notification);
    }

    public Task CancelReminder(int scheduleId)
    {
        LocalNotificationCenter.Current.Cancel(scheduleId);
        return Task.CompletedTask;
    }

    public async Task SyncReminders(IEnumerable<MedicationScheduleDto> schedules, int reminderMinutesBefore = 0)
    {
        var list = schedules?.ToList() ?? new List<MedicationScheduleDto>();
        foreach (var s in list)
        {
            var isPendingAndFuture = s.Status == "Pending" && s.ScheduledDateTime > DateTime.Now;
            if (isPendingAndFuture)
                await ScheduleDoseReminder(s.ScheduleID, s.MedicationName, s.Dosage, s.ScheduledDateTime, reminderMinutesBefore);
            else
                await CancelReminder(s.ScheduleID);
        }
    }
}
```

**تسجيل الـ Plugin في `MauiProgram.cs` (داخل `MauiApp.CreateBuilder()`):**
```csharp
.UseMauiApp<App>()
.UseBarcodeReader()
.UseLocalNotification(config =>
{
    config.AddAndroid(android =>
    {
        android.AddChannel(new Plugin.LocalNotification.AndroidOption.NotificationChannelRequest
        {
            Id = "rubikcare_medication",
            Name = "تذكيرات الأدوية",
            Description = "تنبيهات مواعيد جرعات الأدوية",
            Importance = Plugin.LocalNotification.AndroidOption.AndroidImportance.High,
            Sound = "custom_sound.wav", // ⚠️ ملحوظة: لا يوجد ملف صوت مخصص فعليًا حاليًا — راجع القسم أدناه
            LockScreenVisibility = Plugin.LocalNotification.AndroidOption.AndroidVisibilityType.Public,
            VibrationPattern = new long[] { 0, 250, 250, 250 }
        });
    });
}).ConfigureFonts(fonts => ...)
```

**الصلاحيات في `AndroidManifest.xml`:**
```xml
<uses-permission android:name="android.permission.POST_NOTIFICATIONS" />
<uses-permission android:name="android.permission.SCHEDULE_EXACT_ALARM" />
<uses-permission android:name="android.permission.USE_EXACT_ALARM" />
<uses-permission android:name="android.permission.FOREGROUND_SERVICE" />
<uses-permission android:name="android.permission.FOREGROUND_SERVICE_SPECIAL_USE" />
<uses-permission android:name="android.permission.RECEIVE_BOOT_COMPLETED" />
<uses-permission android:name="android.permission.VIBRATE" />
<uses-permission android:name="android.permission.WAKE_LOCK" />
```

**طلب الصلاحيات في `MainActivity.cs` → `OnCreate`:**
```csharp
RequestPermissions(new string[] { Manifest.Permission.PostNotifications }, 0);

if (OperatingSystem.IsAndroidVersionAtLeast(31))
{
    var alarmManager = (AlarmManager)GetSystemService(AlarmService)!;
    if (!alarmManager.CanScheduleExactAlarms())
    {
        var intent = new Intent(Settings.ActionRequestScheduleExactAlarm);
        StartActivity(intent);
    }
}
```
> ملحوظة: بسبب وجود `USE_EXACT_ALARM` في الـ Manifest (متاحة من Android 13)، النظام بيمنح صلاحية الـ Exact Alarm تلقائيًا وقت التثبيت للتطبيقات اللي جوهر وظيفتها تنبيهات/تذكيرات — فـ`CanScheduleExactAlarms()` بترجع `true` من البداية ومفيش شاشة موافقة إضافية بتظهر. **هذا سلوك متوقع وليس نقصًا.**

**نقطة الاستدعاء في `SchedulePage.razor`:**
```razor
@inject ILocalNotificationService NotificationService
```
```csharp
public async Task LoadAllData()
{
    ...
    try
    {
        await NotificationService.SyncReminders(_schedules);
    }
    catch (Exception ex)
    {
        Debug.WriteLine($"⚠️ SyncReminders ERROR: {ex.Message}");
    }
    ...
}
```

---

## ⚠️ النقاط المفتوحة (Pending) — للـ Local Alarms

### 🟡 نقطة 1: فرق التوقيت — التذكير بيتأخر عن الميعاد الفعلي

**الوصف:**
حاليًا `reminderMinutesBefore` الافتراضية = `0` (اتغيرت من `10` إلى `0` في آخر نسخة من الكود — **تأكد من هذا قبل البدء**، لأن أول نسخة كانت بـ 10 دقايق قبل الميعاد، وده اللي سبب اللبس الأول ("التذكير جه قبل الميعاد بـ 10 دقايق")).

**الحالة الحالية (من الكود المعروض فوق):**
```csharp
Task ScheduleDoseReminder(..., int reminderMinutesBefore = 0);
Task SyncReminders(..., int reminderMinutesBefore = 0);
```
والاستدعاء في `SchedulePage.razor` بيستخدم القيمة الافتراضية (0) — يعني حاليًا **من المفروض التنبيه يظهر في نفس لحظة الميعاد بالظبط**، بدون أي تأخير.

**المطلوب في الشات الجديد:**
- ✅ تأكيد فعلي (باختبار حقيقي على الجهاز) إن التنبيه فعلاً بيظهر في نفس الميعاد المحدد بدون فرق الـ 10 دقايق القديم
- لو لسه فيه فرق، نراجع منطق `Schedule.NotifyTime` وهل فيه أي طبقة تانية (مثلاً كاش قديم أو نسخة قديمة من DLL) بتفرض الـ 10 دقايق دي

### 🔴 نقطة 2: عدم استقرار ظهور التنبيهات على أجهزة ColorOS (Oppo/Realme/OnePlus)

**الوصف:**
- التنبيه اشتغل ناجح في أول اختبار (يوم 30-07)
- في اختبارات لاحقة (نفس اليوم وبعده)، نفس الكود بالظبط — التنبيه معملش Trigger خالص رغم إن اللوج بيأكد إن `ScheduleDoseReminder` اتنفذت بنجاح وطبعت `⏰ Reminder set` بدون أي خطأ في الكود

**التشخيص الحالي (لم يُتحقق منه ميدانيًا بعد):**
الجهاز المستخدم في الاختبار يظهر في اللوج أنه جهاز يعمل بنظام **ColorOS** (Oppo/Realme/OnePlus) بناءً على وجود سطور مثل:
```
OplusScrollToTopManager / OplusViewDragTouchViewHelper / OplusCursorFeedback
```
أنظمة ColorOS معروفة بأنها تقيّد Background Alarms بشدة عبر آليتين منفصلتين عن أذونات Android القياسية:
1. **Battery Optimization لكل تطبيق** (Settings → Battery → App Battery Management)
2. **Auto-launch / Startup Manager** (Settings → App Management → Auto-launch أو Privacy Permissions → Startup)

النمط الملاحظ (اشتغل أول مرة، وقف بعد كده) يتفق مع سلوك ColorOS المعروف: بيسمح بفترة سماح قصيرة بعد التثبيت، وبعدين يبدأ يقيّد التطبيق تلقائيًا لو مفيش استثناء يدوي من المستخدم.

**المطلوب في الشات الجديد (بالترتيب):**
1. **خطوة تحقق يدوية أولاً** (قبل أي تعديل كود) — التأكد من إعدادات الجهاز:
   - `Settings → Battery → App Battery Management → RubikCare → No restrictions` (مش Optimized)
   - `Settings → App Management → Auto-launch → تفعيل RubikCare يدويًا`
   - تكرار اختبار التنبيه بعد التفعيل اليدوي ده ومراقبة النتيجة
2. لو المشكلة اتحلت بعد الخطوة اليدوية → المطلوب برمجيًا: **إضافة شاشة إرشادية داخل التطبيق** (تظهر مرة واحدة أو من صفحة الإعدادات) توجّه المستخدمين (خصوصًا على أجهزة Oppo/Realme/Vivo/Xiaomi/OnePlus) لتفعيل الاستثناءات دي يدويًا — ده مهم بشكل خاص لأن التطبيق طبي (تذكير أدوية) ومينفعش نسيبه عرضة لقيود النظام الصامتة
3. لو المشكلة لم تُحل حتى بعد الاستثناء اليدوي → نحتاج نراجع تفعيل `WakeLock` وقت الجدولة، واحتمال استخدام `AlarmManager.setExactAndAllowWhileIdle` مباشرة (native) كبديل أعمق من طبقة الـ Plugin لو المشكلة أعمق من مجرد Battery Optimization

### 🟢 ملاحظة إضافية (أولوية منخفضة)
سطر `Sound = "custom_sound.wav"` في `MauiProgram.cs` **يشير لملف غير موجود فعليًا** كـ Android Raw Resource (تم التأكد إن مفيش أي `.wav`/`.mp3` جوه `Mobile\Platforms\Android\Resources`). فيه ملف `Shared.UI\wwwroot\sounds\notification.mp3` لكنه Web asset مش Android Raw Resource ومش هيشتغل كصوت Native للـ Channel. حاليًا هذا السطر على الأرجح بيتجاهله النظام أو بيستخدم الصوت الافتراضي — **لازم تأكيد إن ده مش بيسبب Exception صامت أو مشكلة في تسجيل الـ Channel نفسه.** لو عايزين صوت مخصص لاحقًا، محتاجين ننسخ ملف صوت لمجلد `Mobile\Platforms\Android\Resources\raw\` ونربطه بشكل صحيح كـ Android Resource.

---

## 3️⃣ نظام Web Push Notifications (PWA)

### 3.1 الحالة
- ✅ الإشعارات تعمل بنجاح على اللابتوب والموبايل (Android).
- 🟡 مشكلة علامات الاستفهام في النص العربي لسه قيد الفحص.

### 3.2 🗺️ الخريطة الزمنية الكاملة للمشكلة (7 مراحل)

#### المرحلة 1: نظاما إشعارات منفصلان
```
❌ الأصل: الـ Controller كان ينادي INotificationService (FCM) فقط
❌ WebPushSenderService كان مسجّل في DI لكن Dead Code — محدش يناديه
✅ الحل: مناداة النظامين معاً (FCM + WebPush) في try/catch منفصلين
```
**النتيجة:** الإشعارات بدأت تصل على اللابتوب (Desktop Chrome) بنجاح.

#### المرحلة 2: السيرفر الخطأ (uat.rubikcare.com كان متوقف)
```
❌ الـ PWA المنشور دايماً بيتصل بـ https://uat.rubikcare.com (مش localhost)
❌ موقع IIS الخاص بـ uat.rubikcare.com كان Stopped
✅ الحل: تشغيل الموقع + فحص Idle Timeout بتاع الـ Application Pool
```
**درس مهم:** أي اختبار لازم يتم بالرابط الكامل الصحيح (`https://uat.rubikcare.com/...`)، مش مسار نسبي (`/api/...`)، لأن الـ PWA والـ API منشورين على **دومينين مختلفين تماماً**.

#### المرحلة 3: الإرسال ناجح من السيرفر لكن مفيش إشعار على الموبايل
تم التأكد عبر الـ stdout logs إن الطلب بيوصل للـ API الصحيح وبينجح فعلياً (FCM + Web Push الاتنين بيرجعوا نجاح)، لكن الموبايل لسه مفيش إشعار. هنا بدأ التشخيص الأعمق.

#### المرحلة 4: 🔑 الاكتشاف الأهم — ملفين Service Worker مش ملف واحد

**السبب الجذري الحقيقي للمشكلة كلها:**

| الملف | الاستخدام |
|-------|-----------|
| `wwwroot/service-worker.js` | يُستخدم في وضع **Development** فقط |
| `wwwroot/service-worker.published.js` | القالب المصدري اللي بيحل محل `service-worker.js` وقت `dotnet publish` |

**السلوك الحقيقي وقت الـ publish:**
عند تشغيل `dotnet publish`، أداة البناء بتاخد **محتوى** `service-worker.published.js`، بتحقنله الـ assets manifest، وبتكتب الناتج في ملف اسمه **`service-worker.js`** في مجلد النشر (`publish/wwwroot/`) — يعني بيستبدل تماماً أي كود كان موجود في `service-worker.js` الأصلي (بتاع الـ Dev).

**النتيجة:** كل تعديلاتنا الأولى (كود push notifications، الـ logging) كانت بتتعمل في `wwwroot/service-worker.js` — الملف الغلط بالنسبة للإنتاج! أي `dotnet publish` كان بيمسحها ويحط مكانها القالب الافتراضي من مايكروسوفت (اللي بيعمل offline caching بس، ومفيهوش أي push handler خالص).

**علامة التشخيص:** الفرق بين:
```javascript
console.log('[SW] Activating new service worker...');   // ✅ كودنا (dev فقط)
console.info('Service worker: Activate');                 // ⚠️ القالب الافتراضي (بيوصل للإنتاج)
```

#### المرحلة 5: نقل الملف لوحده مش كفاية — لازم المجلد كله

بعد نقل تعديلات الـ push handler لـ `service-worker.published.js` الصح، وعمل publish، طلع إن نسخ ملف `service-worker.js` **لوحده** على السيرفر مش كفاية.

**السبب:**
```javascript
const assetsRequests = self.assetsManifest.assets
    .map(asset => new Request(asset.url, { integrity: asset.hash, cache: 'no-cache' }));
await caches.open(cacheName).then(cache => cache.addAll(assetsRequests));
```
الـ `onInstall` بيعمل fetch لكل الملفات (`.dll`, `.wasm`, إلخ) **مع تحقق integrity hash صارم** من `service-worker-assets.js` (ملف مربوط بنفس نسخة الـ build). لو نقلت `service-worker.js` بس من غير باقي ملفات نفس الـ build، بيحصل **integrity mismatch**، فـ `onInstall` بيفشل صامت، والـ SW الجديد بيفضل "waiting" ومش بيتفعّل خالص.

**القاعدة الذهبية:** أي تحديث لـ Service Worker لازم ينتقل **مع كل مجلد `wwwroot` الناتج من نفس عملية الـ publish**، مش ملف لوحده أبداً.

#### المرحلة 6: skipWaiting و clients.claim مفقودين
القالب الافتراضي من مايكروسوفت (`service-worker.published.js`) **مفيهوش** `self.skipWaiting()` ولا `self.clients.claim()`. ده معناه إن أي SW جديد بيفضل "waiting" لحد ما **كل التابات المفتوحة على الموقع تتقفل تماماً** (مش refresh عادي)، خصوصاً على الموبايل. تمت إضافتهم لضمان إن أي تحديث مستقبلي ياخد أثره أسرع.

#### المرحلة 7 (مفتوحة): الإشعار الافتراضي "This site has been updated in the background"
لو الـ push event وصل للـ Service Worker لكن الـ SW مانداش `showNotification()` (لأنه لسه قديم أو بيفشل صامت)، **Chrome بيجبر نفسه** يعرض إشعار افتراضي عام بدل الموقع (userVisibleOnly contract). لو شفت الرسالة دي، معناها **الموبايل لسه شغال بنسخة SW قديمة** — الحل: قفل Chrome بالكامل من الـ recent apps (مش بس التاب)، أو مسح بيانات الموقع بالكامل من إعدادات Chrome.

### 3.3 🏗️ معمارية النظام الكاملة (End-to-End)

```
1. المستخدم يفتح الـ PWA → MainLayout.razor → InitializePushSafelyAsync()
2. push-notifications.js → subscribeToPush(vapidPublicKey)
3. POST /api/notifications/register-web-push → يُحفظ في UserWebPushSubscriptions
   ══════════════ (انتهى التسجيل) ══════════════
4. Trigger (send-test / حدث حقيقي في التطبيق)
5. NotificationsController → يحفظ Notification في DB (للعرض داخل التطبيق)
6. _pushNotificationService.SendToUserAsync()  → FCM (DeviceTokens، لتطبيق الموبايل الأصلي MAUI)
7. _webPushSenderService.SendToUserAsync()     → Web Push (UserWebPushSubscriptions، للـ PWA)
8. WebPushClient.SendNotificationAsync() مع VAPID → fcm.googleapis.com (Push Service)
9. service-worker.js (النسخة المنشورة الفعلية) → addEventListener('push')
10. self.registration.showNotification() → 🔔 الإشعار يظهر فعلياً
```

### 3.4 نظاما الإشعارات المنفصلان (لازم الاتنين يفضلوا موجودين)

| النظام | الجدول | الاستخدام | ملاحظة |
|--------|--------|-----------|--------|
| **FCM Native** | `DeviceTokens` | تطبيق الموبايل الأصلي (MAUI) | عبر `FirebaseNotificationService` |
| **Web Push** | `UserWebPushSubscriptions` | متصفحات الويب / الـ PWA | عبر `WebPushSenderService` + VAPID |

⚠️ **لا تستبدل نظام بالتاني** — كل واحد بيخدم منصة مختلفة تماماً.

⚠️ **دَين تقني معروف:** يوجد كلاسان بنفس الاسم `FirebaseNotificationService`:
- `Api.Web\Services\FirebaseNotificationService.cs` — المُستخدم فعلياً
- `RubikCare.Infrastructure\Services\FirebaseNotificationService.cs` — دالة `SendToUserAsync` فيه **فارغة تماماً** (stub خطر)

يتم حل التعارض حالياً بـ `using` alias صريح في `Program.cs`. يُفضّل تنظيف ده لاحقاً بحذف النسخة الفارغة أو دمج الاتنين.

### 3.5 المسارات الكاملة للملفات المهمة

```
RubikCare.PWA\
├── wwwroot\
│   ├── service-worker.js              ← Dev فقط! لا يصل للإنتاج أبداً
│   ├── service-worker.published.js    ← ⭐⭐⭐ هذا هو الملف الفعلي في الإنتاج
│   ├── service-worker-assets.js       ← يتولد تلقائياً كل build (asset hashes)
│   ├── manifest.webmanifest
│   ├── js\push-notifications.js       ← subscribeToPush()
│   └── index.html
├── Layout\MainLayout.razor            ← InitializePushSafelyAsync + VAPID Public Key
└── Program.cs                         ← تحديد رابط الـ API (uat.rubikcare.com في الإنتاج)

Api.Web\
├── Services\
│   ├── WebPushSenderService.cs        ← إرسال Web Push (VAPID)
│   └── FirebaseNotificationService.cs ← إرسال FCM (DeviceTokens)
├── Controllers\NotificationsController.cs ← نقطة الدخول لكل الإشعارات
└── Program.cs                         ← DI registration

RubikCare.Domain\Entities\UserWebPushSubscription.cs
```

### 3.6 خريطة السيرفرات (IIS Sites) على الـ VPS

| الموقع | الدومين | الدور |
|--------|---------|-------|
| PU_RubicCareStage | `stagepu.rubikcare.com` | الـ PWA قيد الاختبار |
| **RubikCareUat** | `uat.rubikcare.com` | **API الصحيح** اللي الـ PWA بيتصل بيه في الإنتاج |
| RubicCareApi | `api.rubikcare.com` | API الإنتاج الحقيقي (لا تستخدمه للاختبار) |

⚠️ افحص دايماً حالة تشغيل الموقع (`Get-Website -Name "RubikCareUat"`) قبل أي جلسة تشخيص جديدة — ممكن يكون متوقف بسبب Idle Timeout بتاع الـ Application Pool.

### 3.7 🧪 أدوات ووصفات التشخيص الجاهزة (Playbook)

**1. فحص محتوى SW الفعلي المنشور (مش الملف المحلي):**
`https://stagepu.rubikcare.com/service-worker.js`
دور جوه المحتوى عن الكود المتوقع. لو لقيت `Service worker: Activate` بس (من غير أي push handler)، يبقى النسخة المنشورة قديمة/افتراضية.

**2. تفعيل stdout logging على API الـ UAT:**
```powershell
$webConfigPath = "C:\WebSite\RubikCareUat\web.config"
$content = Get-Content $webConfigPath -Raw
$content = $content -replace 'stdoutLogEnabled="false"', 'stdoutLogEnabled="true"'
Set-Content -Path $webConfigPath -Value $content -Encoding UTF8
Restart-WebAppPool -Name "RubikCareUat"
```

**3. فلترة اللوج لأهم الأسطر بس:**
```powershell
Select-String -Path "C:\WebSite\RubikCareUat\logs\stdout_XXXXX.log" `
  -Pattern "send-test|FCM Push|Web Push|Failed to send|WebPushSenderService"
```

**4. فحص هل الاشتراك اتسجل فعلاً:**
```sql
SELECT * FROM UserWebPushSubscriptions ORDER BY CreatedAt DESC;
```

**5. اختبار الإرسال من Browser Console:**
```javascript
const token = localStorage.getItem('auth_token');
fetch('https://uat.rubikcare.com/api/notifications/send-test', {
    method: 'POST',
    headers: { 'Authorization': `Bearer ${token}`, 'Content-Type': 'application/json' }
}).then(r => r.json()).then(d => console.log('نتيجة:', d)).catch(e => console.error('خطأ:', e));
```
⚠️ لازم يتنفذ من **نفس أصل (origin)** الموقع اللي بتختبر عليه (`stagepu.rubikcare.com`)، مش من `localhost`، عشان تتفادى أخطاء CORS، ولازم الرابط يكون كامل لدومين الـ API الصحيح.

**6. مراقبة الموبايل عن بُعد:**
`chrome://inspect#devices` من اللابتوب مع تفعيل USB Debugging.

**7. فرض تحديث Service Worker فوراً:**
من تاب **Application → Service Workers** → Unregister ثم إعادة تحميل، أو تفعيل "Update on reload" أثناء التطوير.

### 3.8 ⚠️ أخطاء شائعة وقعنا فيها — تجنبها مستقبلاً

1. **لا تعدّل `service-worker.js` وتفترض إنه هيوصل للإنتاج** — عدّل `service-worker.published.js` دايماً.
2. **لا تنقل ملف الـ Service Worker لوحده بعد أي publish** — انقل مجلد `wwwroot` بالكامل.
3. **لا تفترض إن `{success: true}` من الـ API معناها الإشعار وصل فعلاً** — اعتمد دايماً على الـ logs الحقيقية.
4. **لا تختبر بمسار نسبي (`/api/...`) من دومين الـ PWA** — استخدم الرابط الكامل دايماً.
5. **لا تفترض نجاح اللابتوب = نجاح الموبايل** — الإذن والاشتراك منفصلان تماماً.
6. **لو شفت "This site has been updated in the background" على أندرويد** — الحل: تحديث الـ SW فعلياً + إعادة تشغيل Chrome بالكامل.
7. **احذر ترميز الملفات** — استخدم `-Encoding UTF8` عند القراءة بـ PowerShell، واحفظ أي ملف فيه نص عربي مباشر بترميز UTF-8 with BOM.

### 3.9 🔴 المشكلة المفتوحة حالياً: علامات الاستفهام في النص العربي

**الأعراض:**
```json
{"success": true, "message": "? ?? ????? ??????? ?????!"}
```

**ما تم استبعاده:**
- ✅ ترميز `NotificationsController.cs` — عليه UTF-8 BOM سليم
- ✅ منطق `WebPushSenderService.cs` — بيستخدم `System.Text.Json.JsonSerializer.Serialize` بشكل صحيح

**الاتجاه القادم للتحقيق:**
علامات الاستفهام الحرفية (`?`) بدل الرموز التالفة (`Ø§Ù„`) بتدل تحديداً على **تحويل صريح عبر Encoding.ASCII** في مكان ما من الـ pipeline. المشتبه فيهم:
- الكود اللي بيبني الـ `message` string نفسها جوه action الـ `send-test` في `NotificationsController.cs` (لسه محتاجين نشوفه كامل)
- أي middleware مخصص بيتعامل مع الـ response body
- إعدادات الـ Content-Type / charset على الـ response

**الخطوة التالية:** فحص كود الـ `send-test` action كامل.

---

## 📌 القواعد الذهبية

### 🔴 ممنوعات مطلقة

| # | الممنوع |
|:-:|:---|
| 1 | الخلط بين الأنظمة الثلاثة (FCM/Local/Web Push). |
| 2 | تعديل `service-worker.js` — عدّل `service-worker.published.js`. |
| 3 | نقل ملف الـ Service Worker وحده (لازم مجلد `wwwroot` كامل). |
| 4 | اختبار في المتصفح العادي بعد النشر — استخدم `InPrivate`. |
| 5 | افتراض `success: true` معناها الإشعار وصل — اعتمد على logs. |
| 6 | اختبار بمسار نسبي (`/api/...`) — استخدم URL كامل. |
| 7 | افتراض نجاح اللابتوب = نجاح الموبايل — إذن + اشتراك منفصلان. |

### 🟡 قواعد صارمة

| # | القاعدة |
|:-:|:---|
| 1 | `N''` + `MERGE` في SQL. |
| 2 | `@rendermode InteractiveServer` في صفحات Blazor. |
| 3 | Cache 30 دقيقة — تعديل DB يحتاج Restart. |
| 4 | `IsTokenValid` قبل إرسال التوكن. |

---

## 📋 خطة العمل المقترحة للشات الجديد

1. تأكيد توقيت Local Alarms.
2. حل مشكلة ColorOS.
3. حل مشكلة النص العربي في Web Push.
4. إضافة شاشة إرشادية للمستخدمين.
5. حسم ملف الصوت المخصص.
6. إزالة Test Notification.
7. حل مشكلة Mobile translation (من `04-TRANSLATION`).

---

## 📝 سجل التغييرات

| الإصدار | التاريخ | التغييرات |
|:---:|:---|:---|
| **1.0** | **31 يوليو 2026** | الإصدار الأولي — `20-notifications-status.md` |
| **1.0** | **16 سبتمبر 2026** | الإصدار الأولي — `21-web-push.md` |
| **2.0** | **17 أكتوبر 2026** | `08-NOTIFICATIONS` v2.0 — دمج `20` + `21` (ملخص) |
| **3.0** | **17 أكتوبر 2026** | `08-NOTIFICATIONS` v3.0 — إعادة الدمج بالكامل (Playbook) |
| **3.1** | **17 أكتوبر 2026** | **`08-NOTIFICATIONS` v3.1** — استرجاع حرفي لكل تفاصيل `20-notifications-status` (التوقيت، ColorOS، الصلاحيات، ملف الصوت) بدون أي حذف. |

---

**© 2026 RubikCare — للاستخدام الداخلي**

**نهاية وثيقة `08-NOTIFICATIONS` ✅**
```

---
