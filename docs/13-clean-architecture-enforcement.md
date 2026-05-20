# 13 - إصلاح وتطبيق Clean Architecture بشكل صارم

**آخر تحديث: 17 مايو 2026** | **الأولوية: 🔴 حرجة**

---

## المشاكل المكتشفة

### 1. خرق تبعية Api.Web
```
❌ الوضع الحالي:
Api.Web → Domain (مباشر - خطأ)
Api.Web → Infrastructure (مباشر - خطأ)

✅ الوضع الصحيح:
Api.Web → Application فقط
```

### 2. غياب Use Cases الصريحة
```
❌ الوضع الحالي:
Controllers تستدعي Services متعددة وتنسق بينها يدوياً
نفس المنطق يتكرر في Controller وmobile service

✅ الوضع الصحيح:
Controllers تستدعي Use Case واحد فقط
Use Case ينسق بين الخدمات ويعيد نتيجة موحدة
```

### 3. DTOs غير مترابطة
```
❌ الوضع الحالي:
كل Service تعيد DTO مختلف لنفس الكيان
تحويلات متكررة في أماكن متعددة

✅ الوضع الصحيح:
Use Case يعيد DTO موحد
التحويل يحدث مرة واحدة في Application layer
```

---

## خطة الإصلاح التدريجي (بدون توقف الخدمة)

### المرحلة 1: إعداد البنية الجديدة (أسبوع 1)

#### 1.1 إنشاء مجلدات Use Cases في Application

```
RubikCare.Application/
├── UseCases/
│   ├── Auth/
│   │   ├── LoginUseCase.cs
│   │   ├── RegisterUseCase.cs
│   │   └── GoogleLoginUseCase.cs
│   ├── PSP/
│   │   ├── CreateInvitationUseCase.cs
│   │   ├── EnrollPatientUseCase.cs
│   │   ├── ValidateDispenseTokenUseCase.cs
│   │   └── ConfirmDispenseUseCase.cs
│   ├── User/
│   │   ├── GetUserProfileUseCase.cs
│   │   └── UpdateUserProfileUseCase.cs
│   └── Organization/
│       ├── CreateOrganizationUseCase.cs
│       └── ManageMembershipUseCase.cs
```

#### 1.2 نموذج Use Case القياسي

```csharp
// RubikCare.Application/UseCases/PSP/CreateInvitationUseCase.cs

namespace RubikCare.Application.UseCases.PSP
{
    // 1. Request DTO (مدخلات محددة)
    public class CreateInvitationRequest
    {
        public int ProgramId { get; set; }
        public int ClinicId { get; set; }
        public int DoctorUserId { get; set; }
    }

    // 2. Response DTO (مخرجات محددة)
    public class CreateInvitationResponse
    {
        public bool Success { get; set; }
        public int InvitationId { get; set; }
        public string InvitationToken { get; set; }
        public string Message { get; set; }
    }

    // 3. واجهة Use Case
    public interface ICreateInvitationUseCase
    {
        Task<CreateInvitationResponse> ExecuteAsync(CreateInvitationRequest request);
    }

    // 4. تنفيذ Use Case
    public class CreateInvitationUseCase : ICreateInvitationUseCase
    {
        private readonly IPspProgramRepository _programRepository;
        private readonly IPspInvitationRepository _invitationRepository;
        private readonly ITokenGenerator _tokenGenerator;

        // ⭐ حقن Repository Interfaces فقط (وليس DbContext مباشرة)
        public CreateInvitationUseCase(
            IPspProgramRepository programRepository,
            IPspInvitationRepository invitationRepository,
            ITokenGenerator tokenGenerator)
        {
            _programRepository = programRepository;
            _invitationRepository = invitationRepository;
            _tokenGenerator = tokenGenerator;
        }

        public async Task<CreateInvitationResponse> ExecuteAsync(CreateInvitationRequest request)
        {
            // 1. التحقق من اشتراك الطبيب في البرنامج
            var participation = await _programRepository
                .GetActiveParticipationAsync(request.ProgramId, request.DoctorUserId);
            
            if (participation == null)
                return new CreateInvitationResponse 
                { 
                    Success = false, 
                    Message = "الطبيب غير مشترك في هذا البرنامج" 
                };

            // 2. توليد رمز فريد
            var token = _tokenGenerator.GenerateInvitationToken();

            // 3. إنشاء الدعوة
            var invitation = new PspInvitation
            {
                ProgramId = request.ProgramId,
                InvitedByUserId = request.DoctorUserId,
                InvitedOrganizationId = request.ClinicId,
                InvitationToken = token,
                Status = "PENDING",
                InvitationDate = DateTime.UtcNow,
                ExpiryDate = DateTime.UtcNow.AddDays(30)
            };

            await _invitationRepository.AddAsync(invitation);

            // 4. إرجاع النتيجة
            return new CreateInvitationResponse
            {
                Success = true,
                InvitationId = invitation.InvitationId,
                InvitationToken = token,
                Message = "تم إنشاء الدعوة بنجاح"
            };
        }
    }
}
```

---

### المرحلة 2: إصلاح التبعيات (أسبوع 1-2)

#### 2.1 إزالة References غير الصحيحة من Api.Web

```
قبل الإصلاح (Api.Web.csproj):
<ProjectReference Include="..\RubikCare.Domain\RubikCare.Domain.csproj" />        ← احذف
<ProjectReference Include="..\RubikCare.Infrastructure\RubikCare.Infrastructure.csproj" /> ← احذف
<ProjectReference Include="..\RubikCare.Application\RubikCare.Application.csproj" /> ← أبقِ فقط

بعد الإصلاح (Api.Web.csproj):
<ProjectReference Include="..\RubikCare.Application\RubikCare.Application.csproj" />
```

#### 2.2 نقل تسجيلات Infrastructure إلى Application

```csharp
// ملف جديد: RubikCare.Application/DependencyInjection.cs

public static class ApplicationDependencyInjection
{
    public static IServiceCollection AddApplication(this IServiceCollection services)
    {
        // سجل كل Use Cases
        services.AddScoped<ICreateInvitationUseCase, CreateInvitationUseCase>();
        services.AddScoped<IEnrollPatientUseCase, EnrollPatientUseCase>();
        services.AddScoped<ILoginUseCase, LoginUseCase>();
        // ... إلخ

        return services;
    }
}
```

```csharp
// في Program.cs (Api.Web):
builder.Services.AddApplication(); // ⭐ استدعاء واحد بدلاً من تسجيلات متفرقة
```

---

### المرحلة 3: تحويل Controllers لاستخدام Use Cases (أسبوع 2-3)

#### 3.1 مثال: تحويل PspController

```csharp
// ❌ قبل الإصلاح - Controller يعتمد على Services متعددة
[ApiController]
[Route("api/psp")]
public class PspController : ControllerBase
{
    private readonly IPspProgramService _programService;
    private readonly IPspInvitationService _invitationService;
    private readonly IUserService _userService;
    private readonly ITokenService _tokenService;
    // 4 خدمات مختلفة - تنسيق يدوي في كل action

    [HttpPost("create-invitation")]
    public async Task<IActionResult> CreateInvitation([FromBody] CreateInvitationDto dto)
    {
        // منطق منسق يدوياً
        var user = await _userService.GetUserAsync(dto.DoctorId);
        var program = await _programService.GetProgramAsync(dto.ProgramId);
        // ...
    }
}
```

```csharp
// ✅ بعد الإصلاح - Controller يعتمد على Use Case واحد فقط
[ApiController]
[Route("api/psp")]
public class PspController : ControllerBase
{
    private readonly ICreateInvitationUseCase _createInvitationUseCase;

    public PspController(ICreateInvitationUseCase createInvitationUseCase)
    {
        _createInvitationUseCase = createInvitationUseCase;
    }

    [HttpPost("create-invitation")]
    public async Task<IActionResult> CreateInvitation([FromBody] CreateInvitationRequest request)
    {
        var result = await _createInvitationUseCase.ExecuteAsync(request);
        
        if (!result.Success)
            return BadRequest(result);
            
        return Ok(result);
    }
}
```

---

### المرحلة 4: نقل Repositories إلى Infrastructure (أسبوع 2-3)

#### 4.1 واجهات Repository في Application

```csharp
// RubikCare.Application/Contracts/Repositories/IPspProgramRepository.cs

public interface IPspProgramRepository
{
    Task<PspParticipation> GetActiveParticipationAsync(int programId, int userId);
    Task<List<PspProgram>> GetActiveProgramsAsync();
    Task<PspProgram> GetProgramByIdAsync(int programId);
}
```

#### 4.2 تنفيذ Repository في Infrastructure

```csharp
// RubikCare.Infrastructure/Repositories/PspProgramRepository.cs

public class PspProgramRepository : IPspProgramRepository
{
    private readonly BusinessDbContext _context;

    public PspProgramRepository(BusinessDbContext context)
    {
        _context = context;
    }

    public async Task<PspParticipation> GetActiveParticipationAsync(int programId, int userId)
    {
        return await _context.PspParticipations
            .Where(p => p.ProgramId == programId && 
                       p.UserProfileId == userId && 
                       p.IsActive)
            .FirstOrDefaultAsync();
    }

    // ... باقي التنفيذات
}
```

---

## قواعد إلزامية جديدة (تصبح سارية فوراً)

### 🔴 ممنوعات مطلقة

1. **Api.Web لا يشير إلى Domain أو Infrastructure أبداً**
2. **لا Controllers تستدعي أكثر من Use Case واحد**
3. **لا Use Case يعيد Entity مباشرة - دائماً DTO**
4. **لا منطق أعمال في Controllers - تنسيق فقط**
5. **لا حقن DbContext مباشرة في Use Cases - استخدم Repositories**

### 🟢 أنماط إلزامية للكود الجديد

| النمط | التطبيق |
|--------|---------|
| **كل API جديد** | Use Case واحد في Application |
| **كل خدمة جديدة** | واجهة في Application، تنفيذ في Infrastructure |
| **كل DTO** | معرف في Application مع Use Case الخاص به |
| **كل تحقق** | في Use Case، وليس في Controller |

---

## CHECKLIST: عند إنشاء API جديد

- [ ] هل أنشأت Use Case في `Application/UseCases/`؟
- [ ] هل عرفت Request و Response DTOs؟
- [ ] هل عرفت واجهة Repository في Application؟
- [ ] هل نفذت Repository في Infrastructure؟
- [ ] هل سجلت Use Case و Repository في DI؟
- [ ] هل يستخدم Controller Use Case واحد فقط؟
- [ ] هل Api.Web لا يشير إلى Domain أو Infrastructure؟

---

## 🔗 روابط ذات صلة

- [00 - الهيكل المعماري](00-architecture-overview.md)
- [09 - دليل API](09-api-guide.md)
```

---
