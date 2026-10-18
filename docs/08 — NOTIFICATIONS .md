## 📘 `08-NOTIFICATIONS` — نظام الإشعارات (وثيقة مستقلة)

```markdown
# 08 — NOTIFICATIONS (نظام الإشعارات والتذكيرات)

**الإصدار:** 2.0 (Unified Edition)
**آخر تحديث:** 17 أكتوبر 2026
**الحالة:** 🟡 شغال — نقاط مفتوحة
**الجمهور:** فريق التطوير + AI Assistants
**المرجع الإلزامي:** يُقرأ قبل أي تعديل على الإشعارات

---

## 📌 الجزء صفر: نظرة عامة

### 0.1 نظامان منفصلان تماماً

**⚠️ مهم جداً: عدم الخلط بينهما.**

| # | النظام | الآلية | الحالة |
|:-:|:---|:---|:---:|
| **1** | **إشعارات الطلبات المهنية** (Push/FCM) | Firebase (FCM) — يحتاج إنترنت + سيرفر | ✅ شغال |
| **2** | **تذكيرات مواعيد الأدوية** (Local Alarms) | `Plugin.LocalNotification` — محلي على الجهاز | ✅ شغال — ملاحظتان مفتوحتان |
| **3** | **Web Push Notifications** (PWA) | Web Push + VAPID — Service Worker | ✅ شغال — (ملاحظة) |

**⚠️ 3 أنظمة منفصلة — ليس 2.**

---

## 📌 الجزء الأول: نظام إشعارات الطلبات المهنية (Push/FCM)

### 1.1 الحالة

✅ **شغال بالكامل.**

### 1.2 الوظيفة

- **إشعار للأدمن** — عند "طلب مهني" (فتح عيادة/صيدلية).
- **إشعار لصاحب الطلب** — عند الموافقة.
- **قابل للضغط** — التوجيه للصفحة المناسبة.

### 1.3 الملفات المسؤولة

```
Mobile\Services\MauiNotificationNavigationService.cs        ← تنفيذ التوجيه على الموبايل
Rubikcare.Web\Services\WebNotificationNavigationService.cs  ← تنفيذ التوجيه على الويب
Shared.UI\Services\INotificationNavigationService.cs        ← Interface مشترك
```

### 1.4 Commits ذات صلة

```
16129b9 — 2026-07-28 — ProfessionalMode Notification for Admin & user
a864b8b — 2026-07-26 — Push Notification After Add Member to Organization
b71c635 — 2026-07-26 — Push Notification From Fire BASE
```

**⚠️ لا حاجة للعمل على هذا النظام حاليًا — مغلق ومستقر.**

---

## 📌 الجزء الثاني: نظام تذكيرات مواعيد الأدوية (Local Alarms)

### 2.1 الحالة

✅ **الأساسيات شغالة — نقطتان مفتوحتان.**

### 2.2 البنية المعمارية

```
Shared.UI\Services\ILocalNotificationService.cs          ← Interface مشترك
Mobile\Infrastructure\Services\MauiLocalNotificationService.cs  ← التنفيذ (Mobile فقط)
Rubikcare.Web\Services\WebLocalNotificationService.cs     ← No-Op (الويب)
Mobile\MauiProgram.cs                                      ← Plugin + Channel
Mobile\Platforms\Android\MainActivity.cs                   ← الصلاحيات + Exact Alarm
Shared.UI\Components\Patient\SchedulePage.razor            ← نقطة الاستدعاء
```

### 2.3 آلية العمل

1. **أي تغيير في المواعيد** → `LoadAllData()`.
2. **`NotificationService.SyncReminders(_schedules)`** — في نهاية `LoadAllData()`.
3. **`SyncReminders`** — لكل موعد:
   - لو `Status == "Pending"` + في المستقبل → `ScheduleDoseReminder`.
   - غير كده → `CancelReminder`.
4. **`ScheduleDoseReminder`** — `NotifyTime = ScheduledTime - reminderMinutesBefore`.

### 2.4 الكود (مختصر)

**`ILocalNotificationService.cs`:**

```csharp
public interface ILocalNotificationService
{
    Task ScheduleDoseReminder(int scheduleId, string medicationName, string dosage, DateTime scheduledTime, int reminderMinutesBefore = 0);
    Task CancelReminder(int scheduleId);
    Task SyncReminders(IEnumerable<MedicationScheduleDto> schedules, int reminderMinutesBefore = 0);
}
```

**`MauiLocalNotificationService.cs`:**

```csharp
public class MauiLocalNotificationService : ILocalNotificationService
{
    public async Task ScheduleDoseReminder(...)
    {
        var notifyTime = scheduledTime.AddMinutes(-reminderMinutesBefore);
        if (notifyTime <= DateTime.Now) return;
        
        var notification = new NotificationRequest
        {
            NotificationId = scheduleId,
            Title = "💊 تذكير دواء",
            Description = body,
            CategoryType = NotificationCategoryType.Alarm,
            Schedule = new NotificationRequestSchedule { NotifyTime = notifyTime },
            Android = new AndroidOptions
            {
                ChannelId = "rubikcare_medication",
                Priority = AndroidPriority.High
            }
        };
        await LocalNotificationCenter.Current.Show(notification);
    }
    
    // ... Cancel + Sync
}
```

**`MauiProgram.cs` — التسجيل:**

```csharp
.UseLocalNotification(config =>
{
    config.AddAndroid(android =>
    {
        android.AddChannel(new NotificationChannelRequest
        {
            Id = "rubikcare_medication",
            Name = "تذكيرات الأدوية",
            Description = "تنبيهات مواعيد جرعات الأدوية",
            Importance = AndroidImportance.High,
            Sound = "custom_sound.wav", // ⚠️ ملاحظة
            LockScreenVisibility = AndroidVisibilityType.Public,
            VibrationPattern = new long[] { 0, 250, 250, 250 }
        });
    });
})
```

**الصلاحيات (AndroidManifest.xml):**

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

### 2.5 ⚠️ النقاط المفتوحة

#### 🟡 نقطة 1: فرق التوقيت

**الوصف:** `reminderMinutesBefore = 0` افتراضي (من 10 سابقاً).

**المطلوب:**
- ✅ تأكيد فعلي (باختبار حقيقي) إن التنبيه يظهر في نفس الميعاد.
- لو لسه فيه فرق — مراجعة `Schedule.NotifyTime` + Cache.

#### 🔴 نقطة 2: عدم استقرار على ColorOS

**الوصف:**
- التنبيه اشتغل في أول اختبار.
- توقف في اختبارات لاحقة — نفس الكود.

**التشخيص:** ColorOS (Oppo/Realme/OnePlus) يقيّد Background Alarms.

**الحل المُقترَح:**

**1. تحقق يدوي أولاً:**
- `Settings → Battery → App Battery Management → RubikCare → No restrictions`.
- `Settings → App Management → Auto-launch → تفعيل RubikCare يدويًا`.

**2. لو المشكلة اتحلت → إضافة شاشة إرشادية داخل التطبيق.**

**3. لو لم تُحل → مراجعة `WakeLock` + `AlarmManager.setExactAndAllowWhileIdle` مباشرة.**

#### 🟢 ملاحظة إضافية

`Sound = "custom_sound.wav"` — ملف غير موجود فعليًا.

**الحل:** نسخ ملف صوت لـ `Mobile\Platforms\Android\Resources\raw\`.

### 2.6 خطة العمل

```
1. تأكيد التوقيت (نقطة 1)
2. التحقق اليدوي من إعدادات ColorOS (نقطة 2)
3. بناءً على النتيجة:
   - لو اتحل → إضافة شاشة إرشادية
   - لو لم يُحل → مراجعة أعمق
4. حسم موضوع ملف الصوت (اختياري)
5. إزالة زرار "Test Notification" قبل الإصدار
```

### 2.7 أداة التشخيص المؤقتة

**⚠️ `TestNotificationNow()` — يجب إزالتها قبل الإصدار.**

---

## 📌 الجزء الثالث: نظام Web Push (PWA)

### 3.1 نظرة عامة

**⚠️ من `21-Web-Push`:**

**نظام Web Push** = إشعارات المتصفح (Desktop + Mobile) عبر **Service Worker** + **VAPID**.

### 3.2 نظامان منفصلان

| النظام | الجدول | الاستخدام |
|:---|:---|:---|
| **FCM Native** | `DeviceTokens` | تطبيق MAUI |
| **Web Push** | `UserWebPushSubscriptions` | PWA |

### 3.3 ⚠️ المشكلة الحرجة: ملفان Service Worker

**⚠️ مشاريع Blazor WASM PWA فيها ملفان:**

| الملف | الاستخدام |
|:---|:---|
| `wwwroot/service-worker.js` | **Dev فقط** |
| `wwwroot/service-worker.published.js` | **القالب الفعلي** |

**⚠️ عند `dotnet publish` — يُحل `service-worker.published.js` محل `service-worker.js`.**

**⚠️ الخطأ الشائع:** التعديل في `service-worker.js` — يضيع مع `publish`.

### 3.4 الحل: `service-worker.published.js`

**⚠️ يجب التعديل هنا.**

### 3.5 الحالة

**✅ الإشعارات تعمل على اللابتوب والموبايل.**

**🟡 مشكلة علامات الاستفهام في النص العربي — قيد الفحص.**

### 3.6 التشخيص

**⚠️ تفاصيل كاملة في `21-Web-Push`.**

---

## 📌 الجزء الرابع: القواعد الذهبية

### 4.1 🔴 ممنوعات مطلقة

| # | الممنوع |
|:-:|:---|
| 1 | الخلط بين الأنظمة الثلاثة (FCM/Local/Web Push). |
| 2 | تعديل `service-worker.js` — عدّل `service-worker.published.js`. |
| 3 | نقل ملف الـ Service Worker وحده (لازم مجلد `wwwroot` كامل). |
| 4 | اختبار في المتصفح العادي بعد النشر — استخدم InPrivate. |
| 5 | افتراض `success: true` معناها الإشعار وصل — اعتمد على logs. |
| 6 | اختبار بمسار نسبي (`/api/...`) — استخدم URL كامل. |
| 7 | افتراض نجاح اللابتوب = نجاح الموبايل — إذن + اشتراك منفصلان. |

### 4.2 🟡 قواعد صارمة

| # | القاعدة |
|:-:|:---|
| 1 | `N''` + `MERGE` في SQL. |
| 2 | `@rendermode InteractiveServer` في صفحات Blazor. |
| 3 | Cache 30 دقيقة — تعديل DB يحتاج Restart. |
| 4 | `IsTokenValid` قبل إرسال التوكن. |

---

## 📌 الجزء الخامس: Troubleshooting

### 5.1 نظام Local Alarms

#### مشكلة: التنبيه لا يظهر

**الأسباب المحتملة:**
- **إعدادات الجهاز** (ColorOS/Xiaomi) تقيّد Background Alarms.
- **الصلاحيات** — `POST_NOTIFICATIONS` غير ممنوحة.
- **`USE_EXACT_ALARM`** غير موجود في Manifest.

**الحل:** تحقق من الإعدادات اليدوية.

#### مشكلة: التنبيه يظهر متأخراً

**الأسباب:**
- `reminderMinutesBefore = 10` (قيمة قديمة).
- Cache قديم.
- AlarmManager مقيد.

### 5.2 نظام Web Push

#### مشكلة: الإشعار الافتراضي "This site has been updated in the background"

**السبب:** الـ Service Worker الحالي لا ينادي `showNotification()`.

**الحل:**
1. تحديث Service Worker فعلياً.
2. إعادة تشغيل Chrome بالكامل.

#### مشكلة: علامات استفهام في النص العربي

**الأسباب المحتملة:**
- Encoding.ASCII في pipeline.
- Middleware مخصص.

**⚠️ الحالة:** قيد الفحص.

---

## 📌 الجزء السادس: Roadmap

### 6.1 ما هو مطلوب

| # | المهمة | الأولوية | الحالة |
|:-:|:---|:---:|:---:|
| 1 | **تأكيد توقيت Local Alarms** | 🔴 عالية | ⏳ |
| 2 | **حل مشكلة ColorOS** | 🔴 عالية | ⏳ |
| 3 | **حل مشكلة النص العربي في Web Push** | 🟠 متوسطة | ⏳ |
| 4 | **إضافة شاشة إرشادية للمستخدمين** | 🟠 متوسطة | ❌ |
| 5 | **حسم ملف الصوت المخصص** | 🟢 منخفضة | ❌ |
| 6 | **إزالة Test Notification** | 🟠 متوسطة | ❌ |
| 7 | **حل مشكلة Mobile translation** (من `04-TRANSLATION`) | 🟠 متوسطة | ⏳ |

### 6.2 الاعتمادات

- **نقطة 1** — مستقلة.
- **نقطة 2** — تحتاج اختبار ميداني.
- **نقطة 3** — تحتاج فحص pipeline.
- **نقطة 4** — تحتاج UI.
- **نقطة 7** — تحتاج جلسة منفصلة.

---

## 📌 سجل التغييرات

| الإصدار | التاريخ | التغييرات |
|:---:|:---|:---|
| **1.0** | **31 يوليو 2026** | الإصدار الأولي — `20-notifications-status.md` |
| **1.0** | **16 سبتمبر 2026** | الإصدار الأولي — `21-web-push.md` |
| **2.0** | **17 أكتوبر 2026** | **`08-NOTIFICATIONS` v2.0** — دمج `20` + `21` + إشارات |

---

## 📌 روابط ذات صلة

- [00 — MASTER-INDEX](./00-MASTER-INDEX.md)
- [00B — PROJECT-BACKLOG](./00B-PROJECT-BACKLOG.md)
- [01 — ARCHITECTURE](./01-ARCHITECTURE.md)
- [04 — TRANSLATION](./04-TRANSLATION.md)
- [06 — POINTS](./06-POINTS.md)
- [07 — REPORTS](./07-REPORTS.md)
- [14 — PWA](./14-PWA.md)

---

**© 2026 RubikCare — للاستخدام الداخلي**

**نهاية وثيقة `08-NOTIFICATIONS` ✅**
```

---
