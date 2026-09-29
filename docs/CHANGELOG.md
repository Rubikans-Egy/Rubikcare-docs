# Changelog

All notable changes to this project will be documented in this file.

The format is based on [Keep a Changelog](https://keepachangelog.com/en/1.0.0/),
and this project adheres to [Semantic Versioning](https://semver.org/spec/v2.0.0.html).

## [Unreleased]

### Added — نظام نقاط برامج الدعم (PSP Program Points)

**Backend كامل — 29 سبتمبر 2026**

#### Domain Layer
- 8 كيانات جديدة:
  - `PSPActivityType` (جدول مرجعي)
  - `PSPProgramPointTransactionType` (جدول مرجعي)
  - `PSPProgramPointRedeemStatus` (جدول مرجعي)
  - `PSPProgramPointScheme` (نظام النقاط)
  - `PSPProgramPointRule` (القواعد)
  - `PSPProgramPointBalance` (الأرصدة)
  - `PSPProgramPointTransaction` (المعاملات)
  - `PSPProgramPointRedeemRequest` (طلبات Redeem)
- 3 ملفات Constants:
  - `PointActivityTypeIds` (7 أحداث)
  - `PointTransactionTypeIds` (4 أنواع)
  - `RedeemStatusIds` (4 حالات)

#### Infrastructure Layer
- Migration: `AddPSPProgramPointsTables`
- Seed Data: 15 صفاً (7 + 4 + 4)
- 19 Handler في 5 مجلدات (Schemes, Rules, Balances, Earning, Redeem)

#### API Layer
- 5 Controllers جديدة:
  - `PointSchemesController` (4 Endpoints)
  - `PointRulesController` (5 Endpoints)
  - `PointBalancesController` (3 Endpoints)
  - `PointEarningController` (1 Endpoint — قراءة فقط)
  - `PointRedeemController` (5 Endpoints)

#### Application Layer
- `IPSPEnrollmentService` + `PSPEnrollmentService` (4 دوال)
  - `CreateInvitationAsync`
  - `EnrollUserInProgramAsync`
  - `AcceptInvitationAsync`
  - `AcceptRepInvitationAsync`

### Added — ربط النقاط بالأحداث
- `PATIENT_INVITED` — عند إنشاء دعوة مريض
- `PATIENT_ENROLLED` — عند تسجيل المريض
- `ERX_CREATED` — عند إنشاء روشتة إلكترونية

### Changed
- `PspController.Invitations.cs` — استبدال `CreateInvitation` + `AcceptInvitation` لاستخدام `PSPEnrollmentService`
- `PspController.RepInvitations.cs` — استبدال `AcceptRepInvitation` لاستخدام `PSPEnrollmentService`
- `PspController.Helpers.cs` — استبدال `EnrollUserInProgram` لاستخدام `PSPEnrollmentService`
- `PspController.cs` — إضافة `IPSPEnrollmentService` في Constructor

### Added — DTOs
- `AcceptRepInvitationRequest` — نقل من `PspController.RepInvitations.cs` إلى `Application/DTOs/PSP/`

### Documentation
- `docs/26-rubika-points-system.md` — توثيق كامل لنظام النقاط
- `docs/27-technical-debt.md` — الديون التقنية + خطة تفكيك Controllers
- `docs/08-roadmap.md` — تحديث Roadmap بنظام النقاط

### Pending
- ربط `DISPENSATION_COMPLETED` (Helper في `DispenseController`)
- ربط `REFILL_COMPLETED` (Helper في `DispenseController`)
- واجهة MAUI لعرض الرصيد
- لوحة تحكم الشركة (Web)
