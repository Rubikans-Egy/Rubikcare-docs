# 13 - إصلاح وتطبيق Clean Architecture بشكل صارم

**آخر تحديث: 25 مايو 2026** | **الأولوية: 🔴 حرجة**

---

## حالة الإصلاحات الحالية

| الإصلاح | الحالة | التاريخ |
|---------|--------|---------|
| إنشاء أول Use Case (`ClearUserCacheUseCase`) | ✅ تم | 24 مايو 2026 |
| توحيد نظام الكاش | ✅ تم | 24 مايو 2026 |
| إزالة الكاش من `DynamicMenuService` | ✅ تم | 24 مايو 2026 |
| دمج `UserBootstrapService` ← `UserSessionService` | ✅ تم | 24 مايو 2026 |
| نقل `IUserBootstrapRepository` إلى Application | ✅ تم | 24 مايو 2026 |
| **إزالة References غير الصحيحة من Api.Web** | ✅ تم | 25 مايو 2026 |
| **إنشاء `InfrastructureExtensions.cs` موحد** | ✅ تم | 25 مايو 2026 |
| **إصلاح 16 Controller** | ✅ تم | 25 مايو 2026 |
| **Architecture Tests (33 اختبار ناجح)** | ✅ تم | 25 مايو 2026 |
| تحويل Controllers لاستخدام Use Cases | ⬜ مستقبلاً | - |
| حقن DbContext المباشر في الخدمات | ⬜ مستقبلاً | - |

---

## ما تم إنجازه (25 مايو 2026)

### ✅ إزالة References غير الصحيحة

**قبل:**
```xml
<!-- Api.Web.csproj -->
<ProjectReference Include="..\RubikCare.Domain\RubikCare.Domain.csproj" />
<ProjectReference Include="..\RubikCare.Infrastructure\RubikCare.Infrastructure.csproj" />
```

**بعد:**
```xml
<!-- Api.Web.csproj -->
<ProjectReference Include="..\RubikCare.Application\RubikCare.Application.csproj" />
<ProjectReference Include="..\RubikCare.Infrastructure\RubikCare.Infrastructure.csproj" />
```

**ملاحظة:** Reference Infrastructure بقي لضرورة `Program.cs` فقط. الحماية عبر Architecture Tests.

---

### ✅ إنشاء `InfrastructureExtensions.cs`

**المسار:** `RubikCare.Infrastructure/InfrastructureExtensions.cs`

Extension Method وحيد يسجل **كل خدمات** التطبيق:

```csharp
public static IServiceCollection AddAllInfrastructureServices(
    this IServiceCollection services, string connectionString)
{
    // Identity + Database + Services + Repositories + Use Cases
    // (كل التسجيلات في مكان واحد)
}
```

**الاستخدام في Program.cs (سطر واحد):**
```csharp
builder.Services.AddAllInfrastructureServices(connectionString);
```

---

### ✅ إصلاح 16 Controller

كل Controller كان يستخدم `BusinessDbContext` مباشرة تم تحويله إلى `IGenericService<T>`:

| قبل | بعد |
|------|------|
| `BusinessDbContext _context` | `IGenericService<Entity> _service` |
| `_context.Entities.Where(...)` | `_service.GetAllAsync(...)` |
| `_context.SaveChangesAsync()` | `_service.SaveChangesAsync()` |

**قائمة Controllers التي تم إصلاحها:**
- AuthController
- NotificationsController
- DispenseController
- MedicationRequestController
- OrganizationJobTitlesController
- OrganizationMembersController
- OrganizationMemberTitlesController
- OrganizationsController
- + 8 Controllers أخرى

---

### ✅ Architecture Tests

**المسار:** `RubikCare.Tests/Architecture/CleanArchitectureTests.cs`

اختبارات تمنع كسر المعمارية:

```csharp
[Fact]
public void Application_ShouldNot_DependOn_Infrastructure()
{
    var result = Types
        .InAssembly(applicationAssembly)
        .ShouldNot()
        .HaveDependencyOn("RubikCare.Infrastructure")
        .GetResult();

    Assert.True(result.IsSuccessful);
}
```

**النتيجة:** 33 اختبار ناجح ✅

---

## قواعد إلزامية (تصبح سارية فوراً)

### 🔴 ممنوعات مطلقة

1. **لا Controllers تستخدم `BusinessDbContext` مباشرة** - استخدم `IGenericService<T>`
2. **لا تستخدم `using RubikCare.Infrastructure` في أي Controller**
3. **لا حقن DbContext مباشرة في Use Cases - استخدم Repositories**
4. **أي مخالفة = فشل في Architecture Tests = لا يمكن بناء المشروع**

### 🟢 أنماط إلزامية للكود الجديد

| النمط | التطبيق |
|--------|---------|
| **كل API جديد** | `IGenericService<T>` أو خدمة متخصصة من Application |
| **كل خدمة جديدة** | واجهة في Application، تنفيذ في Infrastructure |
| **كل DTO** | معرف في Application |
| **كل تسجيل** | في `InfrastructureExtensions.cs` فقط |

---

## CHECKLIST: عند إنشاء API جديد

- [ ] هل استخدمت `IGenericService<T>` بدل `BusinessDbContext`؟
- [ ] هل التسجيل مضاف في `InfrastructureExtensions.cs`؟
- [ ] هل Architecture Tests ما زالت ناجحة؟
- [ ] هل `Program.cs` لم يتغير (سطر واحد فقط)؟

---
## ✅ الأنماط المسموح بها للوصول للبيانات

### النمط 1: IGenericService<T> (المفضل)
- للـ CRUD البسيطة
- يستخدم في معظم الـ Controllers

### النمط 2: IDbContextFactoryService (للاستعلامات المعقدة)
- مسموح في: Controllers + Services في Infrastructure فقط
- النمط: `_dbFactory.ExecuteReadOnlyAsync(async context => { var db = (DbContext)context; ... })`

### النمط 3: Service متخصص في Infrastructure
- للاستعلامات المعقدة التي تحتاج إعادة استخدام
- يحقن فيه `IDbContextFactoryService`
- مثال: `ClinicReportService`

## 📦 تسجيل الخدمات
- جميع الخدمات تسجل في `InfrastructureExtensions.cs`
- لا يوجد ملف `DependencyInjection.cs` منفصل
## 🔗 روابط ذات صلة

- [00 - الهيكل المعماري](00-architecture-overview.md)
- [09 - دليل API](09-api-guide.md)
- [14 - نظام الكاش الموحد](14-caching-system.md)
```
