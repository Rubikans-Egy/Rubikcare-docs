# 🔔 RubikCare — وثيقة حالة نظام الإشعارات والتذكيرات

**ترسل هذه الوثيقة في بداية أي شات جديد يخص موضوع الإشعارات/التذكيرات**

📅 آخر تحديث: 2026-07-31
🎯 آخر Commit ذو صلة: `84a0bf2 — 2026-07-29 — Handle PSP Create Program Problem`

---

## 🗂️ نظرة عامة — عندنا نظامين منفصلين تمامًا

المشروع فيه نظامين إشعارات مختلفين في الآلية والغرض، ومهم عدم الخلط بينهم:

| # | النظام | الآلية | الحالة |
|---|--------|--------|--------|
| 1 | **إشعارات الطلبات المهنية** (Professional Requests) | Push Notification عبر Firebase (FCM) — يحتاج اتصال إنترنت وسيرفر | ✅ شغال بالكامل |
| 2 | **تذكيرات مواعيد الأدوية** (Medication Schedule Reminders) | Local Notification/Alarm عبر `Plugin.LocalNotification` — مجدول محليًا على الجهاز، مش محتاج سيرفر وقت الإطلاق | ✅ شغال بس فيه ملاحظتين مفتوحتين (تفصيل تحت) |

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
3. `SyncReminders` بتعمل الآتي لكل موعد (Idempotent — باستخدام `NotificationId = ScheduleID` كمفتاح فريد يستبدل القديم تلقائيًا):
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

## ⚠️ النقاط المفتوحة (Pending) — دي اللي محتاجين نشتغل عليها في الشات الجاي

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

## 📋 خطة العمل المقترحة للشات الجديد (بالترتيب)

```
1. تأكيد التوقيت (نقطة 1): اختبار مباشر إن reminderMinutesBefore=0 بيشتغل صح من غير تأخير
2. التحقق اليدوي من إعدادات ColorOS (Battery + Auto-launch) وإعادة الاختبار
3. بناءً على نتيجة الخطوة 2:
   - لو اتحل → إضافة شاشة/رسالة إرشادية داخل التطبيق لباقي المستخدمين
   - لو لسه فيه مشكلة → مراجعة أعمق لطبقة الـ AlarmManager الأصلية
4. (اختياري - أولوية منخفضة) حسم موضوع ملف الصوت المخصص للـ Channel
5. إزالة/تنظيف الزرار التشخيصي المؤقت (Test Notification) من SchedulePage.razor بعد انتهاء الاختبارات
```

---

## 🔧 أداة التشخيص المؤقتة الموجودة حاليًا في الكود

**تنويه: هذا الزرار للاختبار فقط ولازم يتشال قبل الإصدار النهائي (Release).**

في `SchedulePage.razor` تم إضافة زرار أحمر مؤقت "اختبار تذكير (30 ثانية)" بينادي مباشرة:
```csharp
private async Task TestNotificationNow()
{
    await NotificationService.ScheduleDoseReminder(
        scheduleId: 999999,
        medicationName: "🧪 اختبار مباشر",
        dosage: "Test Dose",
        scheduledTime: DateTime.Now.AddSeconds(30),
        reminderMinutesBefore: 0);
}
```
ده بيتخطى منطق `SyncReminders` بالكامل وبيختبر جدولة الإشعار الخام مباشرة — مفيد جدًا للتشخيص السريع، سيبه لحد ما تتأكد إن كل حاجة شغالة 100%.

---

**نهاية وثيقة الحالة 🚀**
