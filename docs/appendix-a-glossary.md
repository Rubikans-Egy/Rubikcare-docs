
# الملحق أ - مسرد المصطلحات (Glossary)

**آخر تحديث: 17 مايو 2026**

---

## الأساسيات

| المصطلح | الشرح |
|---------|-------|
| **Clean Architecture** | بنية نظيفة تفصل التطبيق إلى طبقات (Domain, Application, Infrastructure, Presentation) |
| **Domain Layer** | طبقة الكيانات الأساسية - لا تعتمد على أي طبقة أخرى |
| **Application Layer** | طبقة منطق الأعمال (Use Cases) - تعتمد فقط على Domain |
| **Infrastructure Layer** | طبقة الوصول للبيانات - تعتمد على Application و Domain |
| **Presentation Layer** | طبقة واجهة المستخدم (Api.Web, Web, Mobile) |
| **GenericService\<T\>** | خدمة عامة لعمليات CRUD على أي كيان |
| **DbContextFactory** | مصنع لإنشاء سياقات قاعدة البيانات - يمنع مشاكل السياق المتعدد |

---

## الهوية والمصادقة

| المصطلح | الشرح |
|---------|-------|
| **Identity** | نظام Microsoft.Identity لإدارة المستخدمين والأدوار |
| **ApplicationUser** | كيان المستخدم الممتد من IdentityUser |
| **UserProfile** | كيان البيانات الموسعة للمستخدم |
| **UserRoleService** | خدمة للتعامل مع جدول AspNetUserRoles الموسع |
| **JWT** | JSON Web Token - للمصادقة في API |
| **OrgMembership** | عضوية مستخدم في منظمة (عيادة، شركة، صيدلية) |

---

## القوائم الديناميكية

| المصطلح | الشرح |
|---------|-------|
| **SystemMenu** | قائمة رئيسية (مثل USER_BASE, ADMIN_MENU) |
| **MenuItem** | عنصر داخل قائمة رئيسية |
| **MenuAssignment** | تخصيص قائمة لمستخدم/دور/منظمة |
| **DynamicMenuService** | خدمة جلب القوائم المناسبة للمستخدم |

---

## نظام PSP

| المصطلح | الشرح |
|---------|-------|
| **PSP** | Patient Support Program - برنامج دعم المرضى |
| **PSPProgram** | برنامج الدعم (الاسم، الشركة المالكة) |
| **PSPProgramMedication** | دواء داخل برنامج دعم |
| **PSPParticipation** | مشاركة عيادة/طبيب في برنامج |
| **PSPInvitation** | دعوة من طبيب لمريض |
| **PSPPatient** | مريض مسجل في برنامج |
| **PSPeRX** | وصفة إلكترونية |
| **PSPDispensationPlan** | خطة صرف الأدوية |
| **PSPDispensation** | رمز صرف يستخدمه المريض في الصيدلية |
| **InvitationToken** | رمز دعوة فريد (مثل INV-5CWEFHEJ) |
| **TokenCode** | رمز صرف (مثل RC-20260323-1234) |

---

## الموبايل

| المصطلح | الشرح |
|---------|-------|
| **MAUI** | .NET Multi-platform App UI |
| **XAML** | لغة تعريف واجهات MAUI |
| **BlazorWebView** | مكون لدمج Blazor داخل تطبيق MAUI |
| **Shell** | نظام التنقل في MAUI |
| **SecureStorage** | تخزين آمن للبيانات الحساسة |
| **MVVM** | Model-View-ViewModel - نمط معماري |

---

## الويب

| المصطلح | الشرح |
|---------|-------|
| **Blazor Server** | تطبيق ويب تفاعلي يعمل على الخادم |
| **Razor** | لغة قوالب .NET |
| **RCL** | Razor Class Library - مكتبة مكونات مشتركة |
| **SignalR** | تقنية اتصال حي بين الخادم والمتصفح |

---

## النشر

| المصطلح | الشرح |
|---------|-------|
| **IIS** | Internet Information Services - خادم ويب |
| **APK** | Android Package Kit - ملف تثبيت Android |
| **AAB** | Android App Bundle - صيغة Google Play |
| **SSL** | Secure Sockets Layer - تشفير الاتصال |

---

## عام

| المصطلح | الشرح |
|---------|-------|
| **CRUD** | Create, Read, Update, Delete |
| **DTO** | Data Transfer Object |
| **DI** | Dependency Injection |
| **EF Core** | Entity Framework Core - ORM |
| **Middleware** | وسيط في pipeline الطلبات |
| **RWE** | Real-World Evidence - أدلة العالم الحقيقي |
