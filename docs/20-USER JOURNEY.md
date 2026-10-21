
# 20 — USER JOURNEY (رحلة المستخدم)

**الإصدار:** 1.0
**آخر تحديث:** 21 أكتوبر 2026
**الحالة:** ✅ موثق
**الجمهور:** فريق التطوير + AI Assistants

---

## 🎯 نظرة عامة

هذه الوثيقة توثق **رحلة المريض** (Patient Journey) في RubikCare، من التسجيل إلى الصرف إلى النقاط. كل مرحلة موثقة بالأحداث، الـ Endpoints، والملفات المتأثرة.

---

## 📊 الرحلة الكاملة (8 Phases)

```
┌─────────────────────────────────────────────────────────────────────┐
│  PHASE 1: ONBOARDING (ما قبل المريض)                                │
└─────────────────────────────────────────────────────────────────────┘

[1] المندوب (Rep)
    ├── ينشئ حساب منظمة (عيادة/صيدلية)
    ├── يرسل دعوة للعيادة/الصيدلية للانضمام لبرنامج دعم
    └── العيادة/الصيدلية توافق وتُفعّل العضوية

[2] الطبيب (Doctor)
    ├── يسجل دخول من حسابه في العيادة
    ├── يفتح قائمة برامج PSP المتاحة
    ├── يولّد كود دعوة (InvitationCode) لكل مريض
    └── يُسلّم الكود للمريض

┌─────────────────────────────────────────────────────────────────────┐
│  PHASE 2: ENROLLMENT (المريض)                                       │
└─────────────────────────────────────────────────────────────────────┘

[3] المريض (Patient)
    ├── ينشئ حساباً على المنصة
    ├── يفتح صفحة "برامج الدعم" (/psp/entry)
    ├── يُدخل كود الدعوة
    ├── النظام يتحقق ويُنشئ PSPPatient (CaseID) + PSPParticipation
    ├── النظام يُنشئ PSPeRX لكل دواء
    └── المريض يُصبح "مشاركاً" في البرنامج

┌─────────────────────────────────────────────────────────────────────┐
│  PHASE 3: INITIAL TOKEN (التوكن الأول)                              │
└─────────────────────────────────────────────────────────────────────┘

[4] النظام (System)
    ├── يجلب DispensationPlan المرتبط بالبرنامج
    ├── يحسب عدد التوكنات المتاحة
    ├── يولّد التوكن الأول (RefillNumber = 0)
    ├── يُنشئ PSPDispensation (Status = "GENERATED")
    └── يُرسل إشعار للمريض

[5] المريض (Patient)
    ├── يفتح PspDetail → يرى التوكن + QR Code
    ├── يضغط "إرسال للصيدلية"
    └── الصيدلية تستلم التوكن

┌─────────────────────────────────────────────────────────────────────┐
│  PHASE 4: DISPENSATION (الصرف)                                     │
└─────────────────────────────────────────────────────────────────────┘

[6] الصيدلي (Pharmacist)
    ├── يستلم التوكن من المريض
    ├── يفتح صفحة "التحقق من التوكن"
    ├── يُدخل TokenCode أو يمسح QR
    ├── يضغط "تأكيد الصرف"
    └── النظام يُحدّث PSPDispensation.Status = "USED"

[7] النظام (System) — عند الصرف
    ├── يحسب: TotalAmount, PSPCoverage, PatientPayment
    ├── 🎁 يمنح نقاط الصرف (DISPENSATION_COMPLETED / REFILL_COMPLETED)
    │   └── عبر IEarnPointsOrchestrator.AwardByCampaignAsync()
    ├── يُحدّث PSPProgramPointBalance
    ├── يُنشئ PSPProgramPointTransaction
    └── يُرسل إشعار للمريض

┌─────────────────────────────────────────────────────────────────────┐
│  PHASE 5: CONFIRMATION (تأكيد الاستلام)                             │
└─────────────────────────────────────────────────────────────────────┘

[8] المريض
    ├── يستلم إشعار → يفتح PspDetail
    ├── يضغط "تأكيد الاستلام"
    └── النظام يُحدّث PSPDispensation.Status = "RECEIVED"

[9] النظام — تلقائياً
    ├── GenerateNextTokenIfNeeded()
    ├── يولّد توكن جديد
    └── المريض يرى توكن جديد

┌─────────────────────────────────────────────────────────────────────┐
│  PHASE 6: DAILY ADHERENCE (الالتزام اليومي)                         │
└─────────────────────────────────────────────────────────────────────┘

[10] المريض (يومياً)
     ├── يستلم تذكير (PSPReminder)
     ├── يفتح صفحة "جدول الأدوية"
     ├── يضغط "تناولت الجرعة"
     └── النظام:
         ├── يُحدّث PatientMedicationSchedule.Status = "Taken"
         ├── يُحدّث PSPReminder.Status = "Completed"
         └── 🎁 يمنح نقاط التناول (DAILY_DOSE_TAKEN)
             └── عبر AcknowledgeReminderHandler / UpdateScheduleStatusAsync

┌─────────────────────────────────────────────────────────────────────┐
│  PHASE 7: POINTS REDEMPTION (استخدام النقاط)                        │
└─────────────────────────────────────────────────────────────────────┘

[11] المريض (عند تجديد التوكن أو صرفه)
     ├── يفتح PspDetail → يرى زر "استخدم نقاطي"
     ├── يُدخل عدد النقاط
     ├── يضغط "تطبيق وتجديد التوكن"
     └── النظام (ApplyPointsToCurrentToken / GenerateNextToken):
         ├── يخصم X نقطة من PSPProgramPointBalance
         ├── يُنشئ PSPProgramPointTransaction (Points = -X, Type = 9)
         ├── يُحدّث PatientPayment (ينخفض)
         └── يُحدّث PSPCoverage (يرتفع)

┌─────────────────────────────────────────────────────────────────────┐
│  PHASE 8: POINTS DASHBOARD (لوحة النقاط)                            │
└─────────────────────────────────────────────────────────────────────┘

[12] المريض (أي وقت)
     ├── يفتح /patient/points
     ├── يرى:
     │   ├── TotalBalance
     │   ├── TotalEarned
     │   ├── TotalRedeemed
     │   ├── Adherence
     │   ├── Streak
     │   ├── Badges
     │   └── DynamicDiscount
     └── يرى برامجه النشطة (كل برنامج برصيده)
```

---

## 📋 جدول المراحل (ملخص)

| # | المرحلة | الحدث الرئيسي | الـ Endpoint | الحالة |
|---|---------|---------------|-------------|--------|
| 1 | Onboarding | Rep → Org | POST /api/rep/invitations/create | ✅ |
| 2 | Doctor → Patient | إنشاء دعوة | POST /api/psp/create-invitation | ✅ |
| 3 | Patient Enrollment | قبول دعوة | POST /api/psp/accept-invitation | ✅ |
| 4 | Initial Token | توليد توكن | (تلقائي) | ✅ |
| 5 | Dispensation | تأكيد صرف | POST /api/dispense/confirm | ✅ |
| 6 | Confirmation | تأكيد استلام | POST /api/dispense/confirm-receipt | ✅ |
| 7 | Daily Adherence | تناول جرعة | PUT /api/patient/medications/status | ✅ |
| 8 | Points Redemption | استخدام نقاط | POST /api/dispense/apply-points | ✅ |
| 9 | Points Dashboard | عرض نقاط | GET /api/psp/points/balances/patient-dashboard/{id} | ✅ |

---

## ⚠️ ملاحظات مهمة

### ملاحظة 1: المريض قد يكون مشتركاً في برامج متعددة
- كل برنامج له `PSPParticipation` منفصل.
- `PSPPatient` واحد (CaseID واحد).
- النقاط تتراكم في `PSPProgramPointBalance` (حسب `Scheme`).

### ملاحظة 2: النقاط مرتبطة بـ `PatientProfileID` (وليس `CaseID`)
- `PSPProgramPointBalance.BeneficiaryID = PatientProfileID`.
- لكل `Scheme` رصيد واحد فقط.

### ملاحظة 3: التوكنات مرتبطة بـ `CaseID` (وليس `PatientProfileID`)
- `PSPDispensation.ERxID → PSPeRX.CaseID → PSPPatient.CaseID`.

---

© 2026 RubikCare — للاستخدام الداخلي
