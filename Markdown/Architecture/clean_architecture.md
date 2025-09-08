# 🏛️ **CLEAN ARCHITECTURE - KIẾN TRÚC HOÀNG GIA**
*"Xây dựng hệ thống như cung điện: tầng bậc rõ ràng, bền vững ngàn năm"*

---

## 🎯 **TẠI SAO CẦN CLEAN ARCHITECTURE?**

### 🏚️ **Vấn đề của kiến trúc truyền thống:**
```
❌ Spaghetti Code - Như mớ mì tôm:
Controller → Service → Repository → Database
     ↓         ↓          ↓
  Business   Database   HTTP Request
   Logic     Logic      Logic
   
🤯 Kết quả: Tất cả lẫn lộn, sửa 1 chỗ hỏng 10 chỗ!
```

### ✨ **Clean Architecture giải quyết:**
```
✅ Như cung điện có tầng bậc:
👑 Domain (Hoàng gia) - Business Rules
🤵 Application (Quản gia) - Use Cases  
🌐 Infrastructure (Hạ tầng) - Database, API
🎭 Presentation (Giao diện) - Controllers, Views

🎯 Nguyên tắc: Tầng trong KHÔNG phụ thuộc tầng ngoài!
```

---

## 🏰 **4 TẦNG LÂUĐÀI CHI TIẾT**

### 👑 **DOMAIN LAYER - Tầng Hoàng Gia**
*"Quyền lực tối cao, không phụ thuộc ai"*

**🎪 Vai trò:** Chứa nghiệp vụ cốt lõi, quy tắc kinh doanh
**🛡️ Đặc điểm:** Không import gì từ tầng khác!

```csharp
// 📁 Domain/Entities/Customer.cs
public class Customer
{
    public CustomerId Id { get; private set; }
    public string Name { get; private set; }
    public Email Email { get; private set; }
    public CustomerType Type { get; private set; }
    
    // 👑 BUSINESS RULE: Khách VIP được giảm giá
    public decimal CalculateDiscount(decimal amount)
    {
        return Type == CustomerType.VIP ? amount * 0.2m : 0;
    }
    
    // 👑 BUSINESS RULE: Email phải hợp lệ khi tạo
    public static Customer Create(string name, string email)
    {
        if (string.IsNullOrEmpty(name))
            throw new DomainException("Customer name is required");
            
        return new Customer
        {
            Id = CustomerId.NewId(),
            Name = name,
            Email = Email.Create(email), // Value Object validation
            Type = CustomerType.Regular
        };
    }
}

// 📁 Domain/ValueObjects/Email.cs  
public class Email : ValueObject
{
    public string Value { get; private set; }
    
    private Email(string value) => Value = value;
    
    public static Email Create(string email)
    {
        if (!IsValidEmail(email))
            throw new DomainException("Invalid email format");
            
        return new Email(email);
    }
    
    private static bool IsValidEmail(string email)
    {
        // Regex validation logic
        return email.Contains("@") && email.Contains(".");
    }
}

// 📁 Domain/Interfaces/ICustomerRepository.cs
public interface ICustomerRepository
{
    Task<Customer> GetByIdAsync(CustomerId id);
    Task<Customer> GetByEmailAsync(Email email);
    Task SaveAsync(Customer customer);
}
```

**🔥 Điểm mạnh Domain Layer:**
- ✅ **Pure Business Logic:** Chỉ chứa quy tắc kinh doanh
- ✅ **Testable:** Không phụ thuộc database, API
- ✅ **Reusable:** Dùng được cho Web, Mobile, Console
- ✅ **Stable:** Ít thay đổi nhất

---

### 🤵 **APPLICATION LAYER - Tầng Quản Gia**
*"Điều phối mọi hoạt động, nhưng không làm việc nặng"*

**🎪 Vai trò:** Orchestration - điều phối các Use Cases
**🔗 Phụ thuộc:** Chỉ Domain Layer

```csharp
// 📁 Application/UseCases/CreateCustomer/CreateCustomerCommand.cs
public class CreateCustomerCommand : IRequest<CustomerDto>
{
    public string Name { get; set; }
    public string Email { get; set; }
}

// 📁 Application/UseCases/CreateCustomer/CreateCustomerHandler.cs
public class CreateCustomerHandler : IRequestHandler<CreateCustomerCommand, CustomerDto>
{
    private readonly ICustomerRepository _customerRepository;
    private readonly IEmailService _emailService; // Interface từ Domain
    private readonly IUnitOfWork _unitOfWork;
    
    public CreateCustomerHandler(
        ICustomerRepository customerRepository,
        IEmailService emailService,
        IUnitOfWork unitOfWork)
    {
        _customerRepository = customerRepository;
        _emailService = emailService;
        _unitOfWork = unitOfWork;
    }
    
    public async Task<CustomerDto> Handle(CreateCustomerCommand request, CancellationToken ct)
    {
        // 1️⃣ Validation
        var email = Email.Create(request.Email);
        var existingCustomer = await _customerRepository.GetByEmailAsync(email);
        if (existingCustomer != null)
            throw new BusinessException("Email already exists");
        
        // 2️⃣ Create Domain Object
        var customer = Customer.Create(request.Name, request.Email);
        
        // 3️⃣ Save to Repository
        await _customerRepository.SaveAsync(customer);
        await _unitOfWork.CommitAsync();
        
        // 4️⃣ Send Welcome Email (Side Effect)
        await _emailService.SendWelcomeEmailAsync(customer.Email.Value);
        
        // 5️⃣ Return DTO
        return new CustomerDto
        {
            Id = customer.Id.Value,
            Name = customer.Name,
            Email = customer.Email.Value
        };
    }
}

// 📁 Application/DTOs/CustomerDto.cs
public class CustomerDto
{
    public Guid Id { get; set; }
    public string Name { get; set; }
    public string Email { get; set; }
    public string Type { get; set; }
}

// 📁 Application/Interfaces/IEmailService.cs
public interface IEmailService
{
    Task SendWelcomeEmailAsync(string email);
    Task SendOrderConfirmationAsync(string email, OrderDto order);
}
```

**🎯 Application Layer Patterns:**
- **📋 Commands:** Thay đổi dữ liệu (Create, Update, Delete)
- **🔍 Queries:** Lấy dữ liệu (Get, List, Search)
- **🎭 DTOs:** Data Transfer Objects - chuyển đổi dữ liệu
- **⚡ Events:** Domain Events, Integration Events

---

### 🌐 **INFRASTRUCTURE LAYER - Tầng Hạ Tầng**
*"Thợ kỹ thuật - Kết nối với thế giới bên ngoài"*

**🎪 Vai trò:** Implement các Interface từ Domain/Application
**🔧 Công việc:** Database, File, Email, HTTP calls...

```csharp
// 📁 Infrastructure/Persistence/CustomerRepository.cs
public class CustomerRepository : ICustomerRepository
{
    private readonly AppDbContext _context;
    
    public CustomerRepository(AppDbContext context)
    {
        _context = context;
    }
    
    public async Task<Customer> GetByIdAsync(CustomerId id)
    {
        var entity = await _context.Customers
            .FirstOrDefaultAsync(c => c.Id == id.Value);
            
        return entity?.ToDomainModel(); // Convert Entity → Domain
    }
    
    public async Task<Customer> GetByEmailAsync(Email email)
    {
        var entity = await _context.Customers
            .FirstOrDefaultAsync(c => c.Email == email.Value);
            
        return entity?.ToDomainModel();
    }
    
    public async Task SaveAsync(Customer customer)
    {
        var entity = CustomerEntity.FromDomain(customer); // Domain → Entity
        
        var existing = await _context.Customers
            .FirstOrDefaultAsync(c => c.Id == customer.Id.Value);
            
        if (existing == null)
            _context.Customers.Add(entity);
        else
            existing.UpdateFrom(entity);
    }
}

// 📁 Infrastructure/Persistence/Entities/CustomerEntity.cs
[Table("Customers")]
public class CustomerEntity
{
    public Guid Id { get; set; }
    public string Name { get; set; }
    public string Email { get; set; }
    public string Type { get; set; }
    public DateTime CreatedAt { get; set; }
    public DateTime? UpdatedAt { get; set; }
    
    // Conversion methods
    public Customer ToDomainModel()
    {
        return Customer.Restore(
            CustomerId.From(Id),
            Name,
            Email.Create(Email),
            Enum.Parse<CustomerType>(Type)
        );
    }
    
    public static CustomerEntity FromDomain(Customer customer)
    {
        return new CustomerEntity
        {
            Id = customer.Id.Value,
            Name = customer.Name,
            Email = customer.Email.Value,
            Type = customer.Type.ToString(),
            CreatedAt = DateTime.UtcNow
        };
    }
}

// 📁 Infrastructure/Services/EmailService.cs
public class EmailService : IEmailService
{
    private readonly IConfiguration _config;
    private readonly ILogger<EmailService> _logger;
    
    public async Task SendWelcomeEmailAsync(string email)
    {
        try
        {
            // SMTP logic, SendGrid API, etc.
            var message = new MailMessage
            {
                To = email,
                Subject = "Welcome to our platform!",
                Body = GetWelcomeTemplate()
            };
            
            await SendEmailAsync(message);
            _logger.LogInformation("Welcome email sent to {Email}", email);
        }
        catch (Exception ex)
        {
            _logger.LogError(ex, "Failed to send welcome email to {Email}", email);
            throw;
        }
    }
}

// 📁 Infrastructure/Persistence/AppDbContext.cs
public class AppDbContext : DbContext
{
    public DbSet<CustomerEntity> Customers { get; set; }
    
    protected override void OnModelCreating(ModelBuilder builder)
    {
        // 🎯 Configuration cho từng Entity
        builder.Entity<CustomerEntity>(entity =>
        {
            entity.HasKey(e => e.Id);
            entity.Property(e => e.Name).IsRequired().HasMaxLength(100);
            entity.Property(e => e.Email).IsRequired().HasMaxLength(255);
            entity.HasIndex(e => e.Email).IsUnique();
        });
    }
}
```

---

### 🎭 **PRESENTATION LAYER - Tầng Giao Diện**
*"Diễn viên - Giao tiếp với khán giả (người dùng)"*

**🎪 Vai trò:** Nhận input, gọi Application, trả output
**🎨 Dạng:** Web API, MVC, Blazor, Console, Mobile...

```csharp
// 📁 WebAPI/Controllers/CustomersController.cs
[ApiController]
[Route("api/[controller]")]
public class CustomersController : ControllerBase
{
    private readonly IMediator _mediator; // MediatR pattern
    
    public CustomersController(IMediator mediator)
    {
        _mediator = mediator;
    }
    
    /// <summary>
    /// Tạo khách hàng mới
    /// </summary>
    [HttpPost]
    [ProducesResponseType(typeof(CustomerDto), 201)]
    [ProducesResponseType(typeof(ErrorResponse), 400)]
    public async Task<IActionResult> CreateCustomer([FromBody] CreateCustomerRequest request)
    {
        try
        {
            var command = new CreateCustomerCommand
            {
                Name = request.Name,
                Email = request.Email
            };
            
            var result = await _mediator.Send(command);
            
            return CreatedAtAction(
                nameof(GetCustomer), 
                new { id = result.Id }, 
                result
            );
        }
        catch (DomainException ex)
        {
            return BadRequest(new ErrorResponse { Message = ex.Message });
        }
        catch (BusinessException ex)
        {
            return Conflict(new ErrorResponse { Message = ex.Message });
        }
    }
    
    /// <summary>
    /// Lấy thông tin khách hàng theo ID
    /// </summary>
    [HttpGet("{id:guid}")]
    [ProducesResponseType(typeof(CustomerDto), 200)]
    [ProducesResponseType(404)]
    public async Task<IActionResult> GetCustomer(Guid id)
    {
        var query = new GetCustomerByIdQuery { Id = id };
        var result = await _mediator.Send(query);
        
        if (result == null)
            return NotFound();
            
        return Ok(result);
    }
}

// 📁 WebAPI/Requests/CreateCustomerRequest.cs
public class CreateCustomerRequest
{
    [Required]
    [StringLength(100, MinimumLength = 2)]
    public string Name { get; set; }
    
    [Required]
    [EmailAddress]
    public string Email { get; set; }
}

// 📁 WebAPI/Program.cs - Dependency Injection Setup
var builder = WebApplication.CreateBuilder(args);

// 🏛️ Register layers từ trong ra ngoài
builder.Services.AddDomain(); // Domain layer
builder.Services.AddApplication(); // Application layer  
builder.Services.AddInfrastructure(builder.Configuration); // Infrastructure layer

// 🎭 Presentation layer
builder.Services.AddControllers();
builder.Services.AddSwaggerGen();

var app = builder.Build();

// 🚀 Configure pipeline
if (app.Environment.IsDevelopment())
{
    app.UseSwagger();
    app.UseSwaggerUI();
}

app.UseHttpsRedirection();
app.UseAuthorization();
app.MapControllers();

app.Run();
```

---

## 🔄 **DEPENDENCY INJECTION SETUP**

### 🎯 **Mỗi layer tự đăng ký dependencies:**

```csharp
// 📁 Domain/DependencyInjection.cs
public static class DependencyInjection
{
    public static IServiceCollection AddDomain(this IServiceCollection services)
    {
        // Domain layer không có dependencies!
        return services;
    }
}

// 📁 Application/DependencyInjection.cs
public static class DependencyInjection
{
    public static IServiceCollection AddApplication(this IServiceCollection services)
    {
        services.AddMediatR(cfg => cfg.RegisterServicesFromAssembly(Assembly.GetExecutingAssembly()));
        services.AddScoped<IValidator<CreateCustomerCommand>, CreateCustomerValidator>();
        
        return services;
    }
}

// 📁 Infrastructure/DependencyInjection.cs
public static class DependencyInjection
{
    public static IServiceCollection AddInfrastructure(
        this IServiceCollection services, 
        IConfiguration configuration)
    {
        // Database
        services.AddDbContext<AppDbContext>(options =>
            options.UseSqlServer(configuration.GetConnectionString("Default")));
            
        // Repositories
        services.AddScoped<ICustomerRepository, CustomerRepository>();
        services.AddScoped<IUnitOfWork, UnitOfWork>();
        
        // External Services
        services.AddScoped<IEmailService, EmailService>();
        services.Configure<EmailSettings>(configuration.GetSection("Email"));
        
        return services;
    }
}
```

---

## 🧪 **TESTING STRATEGY**

### 🎯 **Kiểm thử từng tầng:**

```csharp
// 📁 Tests/Domain.Tests/CustomerTests.cs
public class CustomerTests
{
    [Fact]
    public void Create_WithValidData_ShouldCreateCustomer()
    {
        // Arrange
        var name = "John Doe";
        var email = "john@example.com";
        
        // Act
        var customer = Customer.Create(name, email);
        
        // Assert
        Assert.NotNull(customer);
        Assert.Equal(name, customer.Name);
        Assert.Equal(email, customer.Email.Value);
        Assert.Equal(CustomerType.Regular, customer.Type);
    }
    
    [Fact]
    public void Create_WithInvalidEmail_ShouldThrowException()
    {
        // Arrange & Act & Assert
        Assert.Throws<DomainException>(() => 
            Customer.Create("John", "invalid-email"));
    }
}

// 📁 Tests/Application.Tests/CreateCustomerHandlerTests.cs
public class CreateCustomerHandlerTests
{
    private readonly Mock<ICustomerRepository> _customerRepo;
    private readonly Mock<IEmailService> _emailService;
    private readonly CreateCustomerHandler _handler;
    
    public CreateCustomerHandlerTests()
    {
        _customerRepo = new Mock<ICustomerRepository>();
        _emailService = new Mock<IEmailService>();
        _handler = new CreateCustomerHandler(_customerRepo.Object, _emailService.Object);
    }
    
    [Fact]
    public async Task Handle_WithNewEmail_ShouldCreateCustomer()
    {
        // Arrange
        var command = new CreateCustomerCommand 
        { 
            Name = "John", 
            Email = "john@test.com" 
        };
        
        _customerRepo.Setup(r => r.GetByEmailAsync(It.IsAny<Email>()))
                    .ReturnsAsync((Customer)null);
        
        // Act
        var result = await _handler.Handle(command, CancellationToken.None);
        
        // Assert
        Assert.NotNull(result);
        Assert.Equal("John", result.Name);
        _customerRepo.Verify(r => r.SaveAsync(It.IsAny<Customer>()), Times.Once);
        _emailService.Verify(e => e.SendWelcomeEmailAsync("john@test.com"), Times.Once);
    }
}
```

---

## 🚀 **MIGRATION STRATEGY**

### 📈 **Từ Monolith sang Clean Architecture:**

```csharp
// BƯỚC 1: Tách Domain Models
// ❌ Trước: Entity Framework Model trực tiếp
public class Customer // EF Entity
{
    public int Id { get; set; }
    public string Name { get; set; }
    // ... database concerns lẫn business logic
}

// ✅ Sau: Tách riêng Domain và Database
// Domain Model (thuần business logic)
public class Customer 
{
    // Pure business logic, no database concerns
}

// Database Entity (chỉ mapping)
public class CustomerEntity
{
    // Database mapping only
}

// BƯỚC 2: Tách Business Logic ra Use Cases
// ❌ Trước: Business Logic trong Controller
[HttpPost]
public async Task<IActionResult> CreateCustomer(CustomerDto dto)
{
    // Validation logic
    // Business logic  
    // Database logic
    // Email logic
    // All mixed together!
}

// ✅ Sau: Mỗi Use Case 1 Handler riêng
[HttpPost]  
public async Task<IActionResult> CreateCustomer(CreateCustomerRequest request)
{
    var command = new CreateCustomerCommand { ... };
    var result = await _mediator.Send(command);
    return Ok(result);
}

// BƯỚC 3: Dependency Inversion
// ❌ Trước: Phụ thuộc concrete classes
public class CustomerService
{
    private readonly SqlCustomerRepository _repo; // Concrete!
    private readonly SmtpEmailService _email; // Concrete!
}

// ✅ Sau: Phụ thuộc abstractions
public class CreateCustomerHandler  
{
    private readonly ICustomerRepository _repo; // Interface!
    private readonly IEmailService _email; // Interface!
}
```

---

## 🎊 **KẾT LUẬN**

### ✨ **Lợi ích của Clean Architecture:**

1. **🧪 Testability:** Dễ test từng layer riêng biệt
2. **🔄 Maintainability:** Thay đổi 1 layer không ảnh hưởng layer khác  
3. **📈 Scalability:** Dễ mở rộng và thêm tính năng mới
4. **🚀 Deployability:** Deploy từng phần độc lập
5. **👥 Team Work:** Nhiều dev làm việc song song không conflict

### 🎯 **Khi nào dùng Clean Architecture:**
- ✅ **Dự án lớn** (> 6 tháng phát triển)
- ✅ **Team lớn** (> 3 developers)  
- ✅ **Business logic phức tạp**
- ✅ **Cần testing nghiêm túc**
- ✅ **Muốn maintainable lâu dài**

### ⚡ **Khi nào KHÔNG dùng:**
- ❌ **Prototype nhanh**
- ❌ **Dự án nhỏ < 3 tháng**
- ❌ **CRUD đơn giản**
- ❌ **Team 1-2 người**

---

*"Clean Architecture không phải về việc viết nhiều code hơn, mà về việc viết code tốt hơn!"* ✨