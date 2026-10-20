┌─────────────────────────────────────────────────────────────────────┐
│                    PHASE 1: ONBOARDING (ما قبل المريض)              │
└─────────────────────────────────────────────────────────────────────┘

[1] المندوب (Rep)
    │
    ├── ينشئ حساب منظمة (عيادة/صيدلية) في النظام
    ├── يرسل دعوة للعيادة/الصيدلية للانضمام لبرنامج دعم
    └── العيادة توافق وتُفعّل العضوية

[2] الطبيب (Doctor)
    │
    ├── يسجل دخول من حسابه في العيادة
    ├── يفتح قائمة برامج PSP المتاحة للعيادة
    ├── يولّد كود دعوة (InvitationCode) لكل مريض
    └── يُسلّم الكود للمريض (ورقياً أو رقمياً)

┌─────────────────────────────────────────────────────────────────────┐
│                    PHASE 2: ENROLLMENT (المريض)                     │
└─────────────────────────────────────────────────────────────────────┘

[3] المريض (Patient)
    │
    ├── ينشئ حساباً على المنصة (تسجيل تقليدي أو Google)
    ├── يفتح صفحة "برامج الدعم" (/patient/programs)
    ├── يُدخل كود الدعوة الذي حصل عليه من الطبيب
    ├── النظام يتحقق من الكود → يُنشئ PSPPatient + PSPParticipation
    └── المريض يُصبح "مشاركاً" في البرنامج

┌─────────────────────────────────────────────────────────────────────┐
│                    PHASE 3: INITIAL TOKEN (التوكن الأول)            │
└─────────────────────────────────────────────────────────────────────┘

[4] النظام (System)
    │
    ├── يجلب DispensationPlan المرتبط بالبرنامج
    ├── يحسب عدد التوكنات المتاحة (NumberOfTokens من PlanMedication)
    ├── يولّد التوكن الأول (RefillNumber = 0)
    ├── يُنشئ PSPDispensation (Status = "GENERATED")
    └── يُرسل إشعار للمريض

[5] المريض (Patient)
    │
    ├── يفتح PspDetail → يرى التوكن + QR Code + التفاصيل
    ├── يضغط "إرسال للصيدلية" أو "مشاركة" أو "نسخ"
    └── الصيدلية تستلم التوكن (عبر إشعار أو مسح QR)

┌─────────────────────────────────────────────────────────────────────┐
│                    PHASE 4: DISPENSATION (الصرف)                    │
└─────────────────────────────────────────────────────────────────────┘

[6] الصيدلي (Pharmacist)
    │
    ├── يستلم التوكن من المريض
    ├── يفتح صفحة "التحقق من التوكن"
    ├── يُدخل TokenCode أو يمسح QR
    ├── يضغط "تأكيد الصرف" (POST /api/dispense/confirm)
    └── النظام يُحدّث PSPDispensation.Status = "USED"

[7] النظام (System) — عند الصرف
    │
    ├── يحسب: TotalAmount = unitPrice × quantity
    ├── يحسب: PSPCoverage = TotalAmount × (supportPercentage / 100)
    ├── يحسب: PatientPayment = TotalAmount - PSPCoverage
    │
    ├── 🎁 يمنح نقاط الصرف (DispensationCompleted / RefillCompleted)
    │   └── عبر IEarnPointsOrchestrator.AwardByCampaignAsync()
    │       ├── يُحدّث PSPProgramPointBalance.Balance += points
    │       ├── TotalEarned += points
    │       └── يُنشئ PSPProgramPointTransaction (TransactionTypeID = EARN)
    │
    └── يُرسل إشعار للمريض: "تم صرف الدواء، يرجى تأكيد الاستلام"

┌─────────────────────────────────────────────────────────────────────┐
│               PHASE 5: CONFIRMATION (تأكيد الاستلام)                 │
└─────────────────────────────────────────────────────────────────────┘

[8] المريض
    │
    ├── يستلم إشعار → يفتح PspDetail
    ├── يضغط "تأكيد الاستلام" (POST /api/dispense/confirm-receipt)
    └── النظام يُحدّث PSPDispensation.Status = "RECEIVED"

[9] النظام — تلقائياً
    │
    ├── GenerateNextTokenIfNeeded() (private method)
    ├── يولّد توكن جديد (RefillNumber + 1)
    ├── يُنشئ PSPDispensation جديد (Status = "GENERATED")
    └── المريض يرى توكن جديد في PspDetail

┌─────────────────────────────────────────────────────────────────────┐
│            PHASE 6: DAILY ADHERENCE (الالتزام اليومي)               │
└─────────────────────────────────────────────────────────────────────┘

[10] المريض (يومياً)
     │
     ├── يستلم تذكير (PSPReminder) عبر Push/Web Push
     ├── يفتح صفحة "جدول الأدوية"
     ├── يضغط "تناولت الجرعة" (PATCH /api/patient-medication/update-status)
     └── النظام:
         ├── يُحدّث PatientMedicationSchedule.Status = "Taken"
         ├── يُحدّث PSPReminder.Status = "Completed"
         └── 🎁 يمنح نقاط التناول (DailyDose)
             └── عبر AcknowledgeReminderHandler

┌─────────────────────────────────────────────────────────────────────┐
│           PHASE 7: POINTS REDEMPTION (استخدام النقاط)               │
└─────────────────────────────────────────────────────────────────────┘

[11] المريض (عند تجديد التوكن)
     │
     ├── يفتح PspDetail → يرى زر "استخدم نقاطي" (إذا TotalPoints > 0)
     ├── يضغط الزر → Dialog: "كم نقطة تريد استخدامها؟"
     ├── يُدخل X نقطة → يضغط "تطبيق وتجديد التوكن"
     └── النظام (GenerateNextToken):
         │
         ├── [1] يولّد PSPDispensation جديد
         ├── [2] يخصم X نقطة من PSPProgramPointBalance.Balance
         ├── [3] يُنشئ PSPProgramPointTransaction (Points = -X)
         ├── [4] يُحدّث PatientPayment -= (X × MonetaryValuePerPoint)
         └── [5] يُحدّث PSPCoverage += (X × MonetaryValuePerPoint)

┌─────────────────────────────────────────────────────────────────────┐
│                  PHASE 8: POINTS DASHBOARD (لوحة النقاط)            │
└─────────────────────────────────────────────────────────────────────┘

[12] المريض (أي وقت)
     │
     ├── يفتح /patient/points
     ├── يرى: TotalBalance, TotalEarned, TotalRedeemed, Adherence, Streak
     ├── يرى: البرامج النشطة + رصيدها
     ├── يرى: الأوسمة (Badges) + التقدم
     └── يرى: الخصم الديناميكي (Dynamic Discount) إن وُجد
