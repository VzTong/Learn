# ⚖️ SOLID PRINCIPLES - "5 NGUYÊN TẮC VÀNG"
*Chi tiết từ A-Z: Code như kiến trúc sư, không như thợ xây*

---

## 🎯 **SOLID LÀ GÌ? TẠI SAO QUAN TRỌNG?**

### 🏠 **Ví dụ đời thực: Xây nhà**
```
🏚️ Nhà xây bừa bãi:
- Phòng bếp kiêm phòng ngủ → Khó sống
- Tường không có móng → Dễ sập
- Không có thiết kế → Khó sửa chữa, mở rộng

🏛️ Nhà xây theo quy chuẩn:
- Mỗi phòng một chức năng → Dễ sử dụng
- Móng vững chắc → Bền lâu
- Thiết kế rõ ràng → Dễ sửa, mở rộng
```

**🎯 SOLID giúp code:**
- **Dễ đọc**: Như đọc một cuốn sách hay
- **Dễ sửa**: Thay đổi 1 chỗ, không ảnh hưởng chỗ khác  
- **Dễ test**: Từng phần độc lập, test riêng biệt
- **Dễ mở rộng**: Thêm tính năng mới không phá code cũ

---

## 📖 **S - SINGLE RESPONSIBILITY PRINCIPLE**
*"Một class chỉ nên có một lý do để thay đổi"*

### 🎭 **Ví dụ sai:**
```csharp
❌ SVI PHẠM SRP:
public class User
{
    public string Name { get; set; }
    public string Email { get; set; }
    
    // Quản lý thông tin user
    public void UpdateProfile(string name, string email) { }
    
    // Gửi email - KHÔNG LIÊN QUAN đến User!
    public void SendWelcomeEmail() 
    {
        // Logic gửi email ở đây - SAI!
        var smtpClient = new SmtpClient();
        // ...
    }
    
    // Validate email - CŨNG KHÔNG LIÊN QUAN!
    public bool IsEmailValid(string email)
    {
        return email.Contains("@"); // Logic validation
    }
    
    // Lưu database - VẪN KHÔNG LIÊN QUAN!
    public void SaveToDatabase()
    {
        var connectionString = "...";
        // Logic database ở đây
    }
}

// Vấn đề: 1 class làm 4 việc khác nhau!
// - Quản lý thông tin user
// - Gửi email
// - Validate dữ liệu  
// - Truy cập database
```

### ✅ **Ví dụ đúng:**
```csharp
✅ TUÂN THỦ SRP:

// 1. Class User chỉ quản lý thông tin
public class User
{
    public int Id { get; set; }
    public string Name { get; set; }
    public string Email { get; set; }
    public DateTime CreatedAt { get; set; }
    
    // Chỉ có logic liên quan đến User
    public void UpdateProfile(string name, string email)
    {
        Name = name;
        Email = email;
    }
    
    public string GetDisplayName()
    {
        return string.IsNullOrEmpty(Name) ? "Anonymous" : Name;
    }
}

// 2. Class riêng cho Email
public class EmailService
{
    private readonly IConfiguration _config;
    
    public EmailService(IConfiguration config)
    {
        _config = config;
    }
    
    public async Task SendWelcomeEmailAsync(User user)
    {
        var subject = "Chào mừng bạn đến với hệ thống!";
        var body = $"Xin chào {user.GetDisplayName()}...";
        
        await SendEmailAsync(user.Email, subject, body);
    }
    
    private async Task SendEmailAsync(string to, string subject, string body)
    {
        // Logic gửi email ở đây
    }
}

// 3. Class riêng cho Validation
public class UserValidator
{
    public ValidationResult ValidateUser(User user)
    {
        var result = new ValidationResult();
        
        if (string.IsNullOrEmpty(user.Name))
            result.AddError("Tên không được rỗng");
            
        if (!IsEmailValid(user.Email))
            result.AddError("Email không hợp lệ");
            
        return result;
    }
    
    private bool IsEmailValid(string email)
    {
        return !string.IsNullOrEmpty(email) && 
               email.Contains("@") && 
               email.Contains(".");
    }
}

// 4. Class riêng cho Database
public class UserRepository
{
    private readonly IDbContext _context;
    
    public UserRepository(IDbContext context)
    {
        _context = context;
    }
    
    public async Task<User> GetByIdAsync(int id)
    {
        return await _context.Users.FindAsync(id);
    }
    
    public async Task SaveAsync(User user)
    {
        _context.Users.Add(user);
        await _context.SaveChangesAsync();
    }
}
```

### 🎯 **Khi nào vi phạm SRP?**
```
🚨 Dấu hiệu nhận biết:
- Class có quá nhiều method (>10-15 methods)
- Tên class có "And", "Manager", "Helper", "Utility"
- Khi thay đổi requirement, phải sửa nhiều method trong 1 class
- Class import quá nhiều namespace khác nhau

💡 Cách khắc phục:
- Tách class thành nhiều class nhỏ
- Mỗi class chỉ làm 1 việc cụ thể
- Sử dụng Dependency Injection để kết nối các class
```

---

## 🔓 **O - OPEN/CLOSED PRINCIPLE**
*"Mở để mở rộng, đóng để sửa đổi"*

### 🏢 **Ví dụ đời thực: Tòa nhà**
```
🏗️ Xây tòa nhà:
- Muốn thêm tầng → Xây thêm lên trên (OPEN for extension)
- Không phá nền móng cũ → Giữ nguyên cấu trúc (CLOSED for modification)

💻 Code cũng tương tự:
- Thêm tính năng mới → Tạo class mới (OPEN)
- Không sửa code cũ → Tránh bug (CLOSED)
```

### 🎭 **Ví dụ sai:**
```csharp
❌ VI PHẠM OCP:

public class PaymentProcessor
{
    public void ProcessPayment(Payment payment)
    {
        if (payment.Type == "CreditCard")
        {
            // Logic xử lý thẻ tín dụng
            Console.WriteLine("Processing credit card payment");
            ValidateCreditCard();
            ChargeCreditCard();
        }
        else if (payment.Type == "PayPal")
        {
            // Logic xử lý PayPal
            Console.WriteLine("Processing PayPal payment");
            ValidatePayPal();
            ChargePayPal();
        }
        else if (payment.Type == "BankTransfer")  // THÊM LOẠI MỚI
        {
            // Phải SỬA CODE CŨ để thêm tính năng → VI PHẠM OCP!
            Console.WriteLine("Processing bank transfer");
            ValidateBankTransfer();
            ProcessBankTransfer();
        }
    }
    
    // Mỗi lần thêm payment method mới → phải sửa method này
    // Nguy cơ: Làm hỏng logic cũ, khó test, khó maintain
}
```

### ✅ **Ví dụ đúng:**
```csharp
✅ TUÂN THỦ OCP:

// 1. Định nghĩa interface chung
public interface IPaymentMethod
{
    Task<PaymentResult> ProcessAsync(decimal amount, PaymentDetails details);
    bool CanProcess(PaymentType type);
}

// 2. Implement cho từng loại payment
public class CreditCardPayment : IPaymentMethod
{
    public async Task<PaymentResult> ProcessAsync(decimal amount, PaymentDetails details)
    {
        // Logic riêng cho Credit Card
        Console.WriteLine("Processing credit card payment");
        
        if (!ValidateCreditCard(details.CardNumber))
            return PaymentResult.Failed("Invalid card number");
            
        var result = await ChargeCreditCardAsync(amount, details);
        return result;
    }
    
    public bool CanProcess(PaymentType type) => type == PaymentType.CreditCard;
    
    private bool ValidateCreditCard(string cardNumber) { /* ... */ return true; }
    private async Task<PaymentResult> ChargeCreditCardAsync(decimal amount, PaymentDetails details) 
    { 
        /* ... */ 
        return PaymentResult.Success();
    }
}

public class PayPalPayment : IPaymentMethod
{
    public async Task<PaymentResult> ProcessAsync(decimal amount, PaymentDetails details)
    {
        Console.WriteLine("Processing PayPal payment");
        
        if (!ValidatePayPalAccount(details.PayPalEmail))
            return PaymentResult.Failed("Invalid PayPal account");
            
        var result = await ChargePayPalAsync(amount, details);
        return result;
    }
    
    public bool CanProcess(PaymentType type) => type == PaymentType.PayPal;
}

// 3. THÊM PAYMENT METHOD MỚI mà KHÔNG SỬA CODE CŨ
public class BankTransferPayment : IPaymentMethod  // CLASS MỚI!
{
    public async Task<PaymentResult> ProcessAsync(decimal amount, PaymentDetails details)
    {
        Console.WriteLine("Processing bank transfer");
        
        if (!ValidateBankAccount(details.BankAccount))
            return PaymentResult.Failed("Invalid bank account");
            
        var result = await ProcessBankTransferAsync(amount, details);
        return result;
    }
    
    public bool CanProcess(PaymentType type) => type == PaymentType.BankTransfer;
}

// 4. PaymentProcessor KHÔNG CẦN SỬA
public class PaymentProcessor
{
    private readonly IEnumerable<IPaymentMethod> _paymentMethods;
    
    public PaymentProcessor(IEnumerable<IPaymentMethod> paymentMethods)
    {
        _paymentMethods = paymentMethods;
    }
    
    public async Task<PaymentResult> ProcessPaymentAsync(Payment payment)
    {
        var paymentMethod = _paymentMethods
            .FirstOrDefault(p => p.CanProcess(payment.Type));
            
        if (paymentMethod == null)
            return PaymentResult.Failed("Unsupported payment method");
            
        return await paymentMethod.ProcessAsync(payment.Amount, payment.Details);
    }
}

// 5. Đăng ký trong DI Container
services.AddScoped<IPaymentMethod, CreditCardPayment>();
services.AddScoped<IPaymentMethod, PayPalPayment>();
services.AddScoped<IPaymentMethod, BankTransferPayment>(); // MỚI!
services.AddScoped<PaymentProcessor>();
```

### 🎯 **Lợi ích của OCP:**
```
✅ Thêm tính năng mới mà không sửa code cũ
✅ Giảm nguy cơ tạo bug cho tính năng đã hoạt động
✅ Dễ unit test (test riêng từng payment method)
✅ Team có thể làm việc song song (mỗi người làm 1 payment method)
✅ Dễ maintain và debug
```

---

## 🔄 **L - LISKOV SUBSTITUTION PRINCIPLE**
*"Con có thể thay thế cha mà không làm hỏng chương trình"*

### 🚗 **Ví dụ đời thực: Bánh xe ô tô**
```
🚗 Xe ô tô có 4 bánh xe:
- Bánh Michelin thay Bridgestone → OK (cùng kích thước, chức năng)
- Bánh xe đạp thay bánh ô tô → KHÔNG OK (khác kích thước, chức năng)

💻 Code cũng vậy:
- Object con thay object cha → phải hoạt động bình thường
- Không được thay đổi hành vi cơ bản của cha
```

### 🎭 **Ví dụ sai:**
```csharp
❌ VI PHẠM LSP:

public class Bird
{
    public virtual void Fly()
    {
        Console.WriteLine("Bird is flying");
    }
}

public class Eagle : Bird
{
    public override void Fly()
    {
        Console.WriteLine("Eagle soars high in the sky");
        // OK - Eagle vẫn bay được
    }
}

public class Penguin : Bird  // VẤN ĐỀ Ở ĐÂY!
{
    public override void Fly()
    {
        throw new NotSupportedException("Penguins cannot fly!");
        // VI PHẠM LSP: Penguin không thể thay thế Bird
    }
}

// Code sử dụng
public class BirdWatcher
{
    public void WatchBird(Bird bird)
    {
        Console.WriteLine("Watching bird...");
        bird.Fly(); // CRASH nếu bird là Penguin!
    }
}

// Test
var birds = new List<Bird> 
{ 
    new Eagle(),   // OK
    new Penguin()  // CRASH!
};

foreach (var bird in birds)
{
    birdWatcher.WatchBird(bird); // Exception khi đến Penguin
}
```

### ✅ **Ví dụ đúng:**
```csharp
✅ TUÂN THỦ LSP:

// 1. Tái thiết kế hierarchy
public abstract class Bird
{
    public string Name { get; set; }
    public abstract void Move();
    public abstract void MakeSound();
}

// 2. Tách riêng birds có thể bay
public abstract class FlyingBird : Bird
{
    public abstract void Fly();
    
    public override void Move()
    {
        Fly(); // Flying birds di chuyển bằng cách bay
    }
}

// 3. Tách riêng birds không bay được
public abstract class FlightlessBird : Bird
{
    public abstract void Walk();
    
    public override void Move()
    {
        Walk(); // Flightless birds di chuyển bằng cách đi bộ
    }
}

// 4. Implement cụ thể
public class Eagle : FlyingBird
{
    public override void Fly()
    {
        Console.WriteLine("Eagle soars high in the sky");
    }
    
    public override void MakeSound()
    {
        Console.WriteLine("Eagle: Screech!");
    }
}

public class Penguin : FlightlessBird
{
    public override void Walk()
    {
        Console.WriteLine("Penguin waddles on ice");
    }
    
    public override void MakeSound()
    {
        Console.WriteLine("Penguin: Squawk!");
    }
}

public class Sparrow : FlyingBird
{
    public override void Fly()
    {
        Console.WriteLine("Sparrow flies quickly");
    }
    
    public override void MakeSound()
    {
        Console.WriteLine("Sparrow: Tweet!");
    }
}

// 5. Code sử dụng - AN TOÀN với LSP
public class BirdWatcher
{
    public void WatchBird(Bird bird)
    {
        Console.WriteLine($"Watching {bird.Name}...");
        bird.MakeSound();
        bird.Move(); // Gọi Move(), không care fly hay walk
    }
    
    public void WatchFlyingBird(FlyingBird bird)
    {
        Console.WriteLine($"Watching flying bird {bird.Name}...");
        bird.Fly(); // An toàn vì chỉ accept FlyingBird
    }
}

// Test - Không crash
var allBirds = new List<Bird>
{
    new Eagle { Name = "Golden Eagle" },
    new Penguin { Name = "Emperor Penguin" },
    new Sparrow { Name = "House Sparrow" }
};

var birdWatcher = new BirdWatcher();
foreach (var bird in allBirds)
{
    birdWatcher.WatchBird(bird); // Tất cả đều OK!
}

var flyingBirds = new List<FlyingBird>
{
    new Eagle { Name = "Bald Eagle" },
    new Sparrow { Name = "Robin" }
    // Không thể add Penguin vào đây → Compiler error
};

foreach (var bird in flyingBirds)
{
    birdWatcher.WatchFlyingBird(bird); // An toàn 100%
}
```

### 🎯 **Nguyên tắc LSP:**
```csharp
// Preconditions không được mạnh hơn ở subclass
public class Rectangle
{
    public virtual int Width { get; set; }
    public virtual int Height { get; set; }
    
    public virtual void SetDimensions(int width, int height)
    {
        // Base: Chấp nhận mọi width, height > 0
        if (width <= 0 || height <= 0)
            throw new ArgumentException("Dimensions must be positive");
            
        Width = width;
        Height = height;
    }
}

public class Square : Rectangle
{
    public override void SetDimensions(int width, int height)
    {
        // ❌ SAI: Precondition mạnh hơn (width phải = height)
        if (width != height)
            throw new ArgumentException("Square must have equal dimensions");
            
        base.SetDimensions(width, height);
    }
}

// ✅ ĐÚNG: Postcondition có thể mạnh hơn
public class Square : Rectangle
{
    public override void SetDimensions(int width, int height)
    {
        // Chấp nhận bất kỳ width, height nào (không mạnh hóa precondition)
        var dimension = Math.Max(width, height); // Lấy giá trị lớn hơn
        
        Width = dimension;
        Height = dimension; // Postcondition: Width = Height (mạnh hơn base)
    }
}
```

---

## 🧩 **I - INTERFACE SEGREGATION PRINCIPLE**
*"Không ép client phụ thuộc vào interface không cần thiết"*

### 🎮 **Ví dụ đời thực: Tay cầm game**
```
🎮 Game khác nhau, cần control khác nhau:
- Tetris: Chỉ cần 4 nút (Trái, Phải, Xoay, Xuống)
- Street Fighter: Cần 6 nút đấm + 6 nút đá + joystick
- Racing: Cần ga, phanh, vô lăng

❌ SAI: Làm 1 tay cầm có 20 nút cho tất cả game
→ Chơi Tetris phải bỏ qua 16 nút không dùng

✅ ĐÚNG: Mỗi game có interface riêng
→ Tetris chỉ cần implement 4 method cần thiết
```

### 🎭 **Ví dụ sai:**
```csharp
❌ VI PHẠM ISP:

// Interface "béo" - chứa quá nhiều method
public interface IWorker
{
    void Work();
    void Eat();
    void Sleep();
    void TakeBreak();
    
    // Methods cho human worker
    void GetSalary();
    void TakeVacation();
    void AttendMeeting();
    
    // Methods cho robot worker  
    void Charge();
    void RunDiagnostics();
    void UpdateSoftware();
}

// Human worker phải implement methods không cần thiết
public class HumanWorker : IWorker
{
    public void Work() => Console.WriteLine("Human working");
    public void Eat() => Console.WriteLine("Human eating");
    public void Sleep() => Console.WriteLine("Human sleeping");
    public void TakeBreak() => Console.WriteLine("Human taking break");
    public void GetSalary() => Console.WriteLine("Human getting salary");
    public void TakeVacation() => Console.WriteLine("Human on vacation");
    public void AttendMeeting() => Console.WriteLine("Human in meeting");
    
    // Phải implement nhưng không có ý nghĩa!
    public void Charge() => throw new NotSupportedException("Humans don't charge");
    public void RunDiagnostics() => throw new NotSupportedException("Humans don't run diagnostics");
    public void UpdateSoftware() => throw new NotSupportedException("Humans don't update software");
}

// Robot worker cũng vậy
public class RobotWorker : IWorker
{
    public void Work() => Console.WriteLine("Robot working");
    public void Charge() => Console.WriteLine("Robot charging");
    public void RunDiagnostics() => Console.WriteLine("Robot running diagnostics");
    public void UpdateSoftware() => Console.WriteLine("Robot updating software");
    
    // Robot không cần những này!
    public void Eat() => throw new NotSupportedException("Robots don't eat");
    public void Sleep() => throw new NotSupportedException("Robots don't sleep");
    public void TakeBreak() => throw new NotSupportedException("Robots don't take breaks");
    public void GetSalary() => throw new NotSupportedException("Robots don't get salary");
    public void TakeVacation() => throw new NotSupportedException("Robots don't take vacation");
    public void AttendMeeting() => throw new NotSupportedException("Robots don't attend meetings");
}
```

### ✅ **Ví dụ đúng:**
```csharp
✅ TUÂN THỦ ISP:

// 1. Tách thành nhiều interface nhỏ, chuyên biệt
public interface IWorkable
{
    void Work();
}

public interface IBiological
{
    void Eat();
    void Sleep();
    void TakeBreak();
}

public interface IEmployable
{
    void GetSalary();
    void TakeVacation();
    void AttendMeeting();
}

public interface IMaintainable
{
    void Charge();
    void RunDiagnostics();
    void UpdateSoftware();
}

// 2. Human chỉ implement interfaces cần thiết
public class HumanWorker : IWorkable, IBiological, IEmployable
{
    // IWorkable
    public void Work() => Console.WriteLine("Human working efficiently");
    
    // IBiological
    public void Eat() => Console.WriteLine("Human eating lunch");
    public void Sleep() => Console.WriteLine("Human sleeping 8 hours");
    public void TakeBreak() => Console.WriteLine("Human taking coffee break");
    
    // IEmployable
    public void GetSalary() => Console.WriteLine("Human receiving monthly salary");
    public void TakeVacation() => Console.WriteLine("Human going on vacation");
    public void AttendMeeting() => Console.WriteLine("Human attending team meeting");
}

// 3. Robot chỉ implement interfaces phù hợp
public class RobotWorker : IWorkable, IMaintainable
{
    // IWorkable
    public void Work() => Console.WriteLine("Robot working 24/7");
    
    // IMaintainable
    public void Charge() => Console.WriteLine("Robot charging battery");
    public void RunDiagnostics() => Console.WriteLine("Robot running system diagnostics");
    public void UpdateSoftware() => Console.WriteLine("Robot updating to latest version");
}

// 4. AI Worker - kết hợp đặc điểm của cả hai
public class AIWorker : IWorkable, IMaintainable, IEmployable
{
    // IWorkable
    public void Work() => Console.WriteLine("AI processing complex tasks");
    
    // IMaintainable
    public void Charge() => Console.WriteLine("AI optimizing power consumption");
    public void RunDiagnostics() => Console.WriteLine("AI self-analyzing performance");
    public void UpdateSoftware() => Console.WriteLine("AI learning new algorithms");
    
    // IEmployable (AI có thể có "contract")
    public void GetSalary() => Console.WriteLine("AI receiving resource allocation");
    public void TakeVacation() => Console.WriteLine("AI entering maintenance mode");
    public void AttendMeeting() => Console.WriteLine("AI joining virtual conference");
}

// 5. Services chỉ phụ thuộc vào interface cần thiết
public class WorkManager
{
    public void ManageWork(IWorkable worker)
    {
        worker.Work(); // Chỉ cần IWorkable, không care loại worker
    }
}

public class HRManager
{
    public void ManageEmployee(IEmployable employee)
    {
        employee.GetSalary();
        employee.AttendMeeting();
        // Chỉ làm việc với những ai có thể "được tuyển dụng"
    }
}

public class MaintenanceManager
{
    public void PerformMaintenance(IMaintainable device)
    {
        device.RunDiagnostics();
        device.UpdateSoftware();
        device.Charge();
        // Chỉ làm việc với những thứ cần maintenance
    }
}

// 6. Usage
var workers = new List<IWorkable>
{
    new HumanWorker(),
    new RobotWorker(),
    new AIWorker()
};

var workManager = new WorkManager();
foreach (var worker in workers)
{
    workManager.ManageWork(worker); // Tất cả đều OK
}

// HR chỉ quản lý "employees"
var employees = new List<IEmployable>
{
    new HumanWorker(),
    new AIWorker()
    // Không thể add RobotWorker vì nó không implement IEmployable
};

var hrManager = new HRManager();
foreach (var employee in employees)
{
    hrManager.ManageEmployee(employee);
}
```

### 🎯 **Lợi ích của ISP:**
```
✅ Classes chỉ implement methods thực sự cần
✅ Giảm coupling giữa các components
✅ Dễ testing (mock chỉ interface cần thiết)
✅ Dễ maintain (thay đổi 1 interface không ảnh hưởng khác)
✅ Code rõ ràng hơn (interface nhỏ, mục đích rõ ràng)
```

---

## 🔄 **D - DEPENDENCY INVERSION PRINCIPLE**
*"Phụ thuộc vào abstraction, không phụ thuộc vào concrete"*

### 🔌 **Ví dụ đời thực: Ổ cắm điện**
```
🏠 Thiết bị điện trong nhà:
- Tivi, tủ lạnh, máy giặt... cắm vào ổ điện (interface)
- Không cắm trực tiếp vào nhà máy điện (concrete implementation)
- Thay đổi nhà cung cấp điện → thiết bị vẫn hoạt động bình thường

💻 Code cũng vậy:
- High-level modules phụ thuộc vào interfaces
- Low-level modules implement interfaces đó
- Thay đổi implementation → không ảnh hưởng business logic
```

### 🎭 **Ví dụ sai:**
```csharp
❌ VI PHẠM DIP:

// Low-level modules (concrete implementations)
public class SqlServerRepository
{
    private readonly string _connectionString;
    
    public SqlServerRepository(string connectionString)
    {
        _connectionString = connectionString;
    }
    
    public void SaveUser(User user)
    {
        // SQL Server specific logic
        using var connection = new SqlConnection(_connectionString);
        // ... SQL Server code
    }
    
    public User GetUser(int id)
    {
        // SQL Server specific logic
        using var connection = new SqlConnection(_connectionString);
        // ... return user
        return new User();
    }
}

public class EmailService
{
    public void SendEmail(string to, string subject, string body)
    {
        // SMTP specific logic
        var smtpClient = new SmtpClient("smtp.gmail.com", 587);
        // ... email sending logic
    }
}

// High-level module PHỤ THUỘC TRỰC TIẾP vào concrete classes
public class UserService  // ❌ VI PHẠM DIP
{
    private readonly SqlServerRepository _repository;  // Phụ thuộc concrete!
    private readonly EmailService _emailService;       // Phụ thuộc concrete!
    
    public UserService()
    {
        // Hard-coded dependencies - RẤT KHÓ TEST!
        _repository = new SqlServerRepository("Server=...;Database=...");
        _emailService = new EmailService();
    }
    
    public async Task RegisterUserAsync(User user)
    {
        // Business logic
        _repository.SaveUser(user);  // Nếu đổi từ SQL Server sang MongoDB?
        _emailService.SendEmail(     // Nếu đổi từ SMTP sang SendGrid?
            user.Email, 
            "Welcome", 
            "Welcome to our system"
        );
    }
}

// Vấn đề:
// 1. Khó test: Không thể mock SqlServerRepository, EmailService
// 2. Tight coupling: Đổi database → phải sửa UserService
// 3. Khó mở rộng: Thêm cách gửi email mới → phải sửa code
// 4. Khó maintain: Thay đổi 1 chỗ ảnh hưởng nhiều chỗ
```

### ✅ **Ví dụ đúng:**
```csharp
✅ TUÂN THỦ DIP:

// 1. Định nghĩa abstractions (interfaces)
public interface IUserRepository
{
    Task SaveUserAsync(User user);
    Task<User> GetUserAsync(int id);
    Task<User> GetUserByEmailAsync(string email);
}

public interface IEmailService
{
    Task SendEmailAsync(string to, string subject, string body);
    Task SendWelcomeEmailAsync(User user);
}

public interface INotificationService
{
    Task SendNotificationAsync(string userId, string message);
}

// 2. High-level module phụ thuộc vào abstractions
public class UserService  // ✅ TUÂN THỦ DIP
{
    private readonly IUserRepository _repository;
    private readonly IEmailService _emailService;
    private readonly INotificationService _notificationService;
    private readonly ILogger<UserService> _logger;
    
    // Dependency Injection thông qua constructor
    public UserService(
        IUserRepository repository,
        IEmailService emailService,
        INotificationService notificationService,
        ILogger<UserService> logger)
    {
        _repository = repository;
        _emailService = emailService;
        _notificationService = notificationService;
        _logger = logger;
    }
    
    public async Task<UserRegistrationResult> RegisterUserAsync(User user)
    {
        try
        {
            // Business logic - KHÔNG THAY ĐỔI dù đổi implementation
            
            // 1. Validate
            var existingUser = await _repository.GetUserByEmailAsync(user.Email);
            if (existingUser != null)
                return UserRegistrationResult.Failed("Email already exists");
            
            // 2. Save user
            await _repository.SaveUserAsync(user);
            _logger.LogInformation($"User {user.Email} registered successfully");
            
            // 3. Send welcome email
            await _emailService.SendWelcomeEmailAsync(user);
            
            // 4. Send notification
            await _notificationService.SendNotificationAsync(
                user.Id.ToString(), 
                "Welcome to our platform!"
            );
            
            return UserRegistrationResult.Success(user);
        }
        catch (Exception ex)
        {
            _logger.LogError(ex, $"Failed to register user {user.Email}");
            return UserRegistrationResult.Failed("Registration failed");
        }
    }
}

// 3. Low-level modules implement abstractions
public class SqlServerUserRepository : IUserRepository
{
    private readonly IDbContext _context;
    
    public SqlServerUserRepository(IDbContext context)
    {
        _context = context;
    }
    
    public async Task SaveUserAsync(User user)
    {
        _context.Users.Add(user);
        await _context.SaveChangesAsync();
    }
    
    public async Task<User> GetUserAsync(int id)
    {
        return await _context.Users.FindAsync(id);
    }
    
    public async Task<User> GetUserByEmailAsync(string email)
    {
        return await _context.Users
            .FirstOrDefaultAsync(u => u.Email == email);
    }
}

public class MongoUserRepository : IUserRepository  // Dễ dàng thay thế!
{
    private readonly IMongoCollection<User> _users;
    
    public MongoUserRepository(IMongoDatabase database)
    {
        _users = database.GetCollection<User>("users");
    }
    
    public async Task SaveUserAsync(User user)
    {
        await _users.InsertOneAsync(user);
    }
    
    public async Task<User> GetUserAsync(int id)
    {
        return await _users.Find(u => u.Id == id).FirstOrDefaultAsync();
    }
    
    public async Task<User> GetUserByEmailAsync(string email)
    {
        return await _users.Find(u => u.Email == email).FirstOrDefaultAsync();
    }
}

public class SmtpEmailService : IEmailService
{
    private readonly IConfiguration _config;
    
    public SmtpEmailService(IConfiguration config)
    {
        _config = config;
    }
    
    public async Task SendEmailAsync(string to, string subject, string body)
    {
        using var client = new SmtpClient(_config["Smtp:Host"]);
        var message = new MailMessage("noreply@company.com", to, subject, body);
        await client.SendMailAsync(message);
    }
    
    public async Task SendWelcomeEmailAsync(User user)
    {
        var subject = "Welcome to our platform!";
        var body = $"Hello {user.Name}, welcome to our awesome platform!";
        await SendEmailAsync(user.Email, subject, body);
    }
}

public class SendGridEmailService : IEmailService  // Dễ dàng thay thế!
{
    private readonly ISendGridClient _client;
    
    public SendGridEmailService(ISendGridClient client)
    {
        _client = client;
    }
    
    public async Task SendEmailAsync(string to, string subject, string body)
    {
        var from = new EmailAddress("noreply@company.com", "Company");
        var toEmail = new EmailAddress(to);
        var msg = MailHelper.CreateSingleEmail(from, toEmail, subject, body, body);
        await _client.SendEmailAsync(msg);
    }
    
    public async Task SendWelcomeEmailAsync(User user)
    {
        // Có thể dùng template fancy của SendGrid
        var templateId = "d-welcome-template-id";
        var dynamicData = new { user_name = user.Name, company_name = "Our Company" };
        
        var msg = MailHelper.CreateSingleTemplateEmail(
            new EmailAddress("noreply@company.com"),
            new EmailAddress(user.Email),
            templateId,
            dynamicData
        );
        
        await _client.SendEmailAsync(msg);
    }
}

// 4. Đăng ký dependencies trong DI Container
public void ConfigureServices(IServiceCollection services)
{
    // Có thể dễ dàng thay đổi implementation
    services.AddScoped<IUserRepository, SqlServerUserRepository>(); // Hoặc MongoUserRepository
    services.AddScoped<IEmailService, SmtpEmailService>();          // Hoặc SendGridEmailService
    services.AddScoped<INotificationService, PushNotificationService>();
    
    services.AddScoped<UserService>(); // UserService không thay đổi!
}

// 5. Testing trở nên dễ dàng
[TestClass]
public class UserServiceTests
{
    private Mock<IUserRepository> _mockRepository;
    private Mock<IEmailService> _mockEmailService;
    private Mock<INotificationService> _mockNotificationService;
    private Mock<ILogger<UserService>> _mockLogger;
    private UserService _userService;
    
    [TestInitialize]
    public void Setup()
    {
        _mockRepository = new Mock<IUserRepository>();
        _mockEmailService = new Mock<IEmailService>();
        _mockNotificationService = new Mock<INotificationService>();
        _mockLogger = new Mock<ILogger<UserService>>();
        
        _userService = new UserService(
            _mockRepository.Object,
            _mockEmailService.Object,
            _mockNotificationService.Object,
            _mockLogger.Object
        );
    }
    
    [TestMethod]
    public async Task RegisterUserAsync_WithNewEmail_ShouldSucceed()
    {
        // Arrange
        var user = new User { Email = "test@example.com", Name = "Test User" };
        _mockRepository.Setup(r => r.GetUserByEmailAsync(user.Email))
                      .ReturnsAsync((User)null); // Email chưa tồn tại
        
        // Act
        var result = await _userService.RegisterUserAsync(user);
        
        // Assert
        Assert.IsTrue(result.IsSuccess);
        _mockRepository.Verify(r => r.SaveUserAsync(user), Times.Once);
        _mockEmailService.Verify(e => e.SendWelcomeEmailAsync(user), Times.Once);
    }
    
    [TestMethod]
    public async Task RegisterUserAsync_WithExistingEmail_ShouldFail()
    {
        // Arrange
        var user = new User { Email = "existing@example.com", Name = "Test User" };
        var existingUser = new User { Email = "existing@example.com" };
        _mockRepository.Setup(r => r.GetUserByEmailAsync(user.Email))
                      .ReturnsAsync(existingUser); // Email đã tồn tại
        
        // Act
        var result = await _userService.RegisterUserAsync(user);
        
        // Assert
        Assert.IsFalse(result.IsSuccess);
        Assert.AreEqual("Email already exists", result.ErrorMessage);
        _mockRepository.Verify(r => r.SaveUserAsync(It.IsAny<User>()), Times.Never);
    }
}
```

### 🎯 **Lợi ích của DIP:**
```
✅ Loose coupling: High-level không phụ thuộc low-level
✅ Dễ test: Mock dependencies dễ dàng
✅ Linh hoạt: Thay đổi implementation không ảnh hưởng business logic
✅ Extensible: Thêm implementation mới mà không sửa code cũ
✅ Maintainable: Separation of concerns rõ ràng
```

---

## 🎯 **TỔNG KẾT: ÁP DỤNG SOLID TRONG DỰ ÁN THỰC TẾ**

### 🛒 **Case Study: E-Commerce System**

```csharp
// 1. SINGLE RESPONSIBILITY - Mỗi class một trách nhiệm
public class Product  // Chỉ quản lý thông tin sản phẩm
{
    public int Id { get; set; }
    public string Name { get; set; }
    public decimal Price { get; set; }
    public int Stock { get; set; }
    
    public bool IsAvailable() => Stock > 0;
    public void UpdateStock(int quantity) => Stock = Math.Max(0, Stock + quantity);
}

public class Order  // Chỉ quản lý đơn hàng
{
    public int Id { get; set; }
    public int CustomerId { get; set; }
    public List<OrderItem> Items { get; set; } = new();
    public OrderStatus Status { get; set; }
    
    public decimal GetTotal() => Items.Sum(i => i.Price * i.Quantity);
    public void AddItem(int productId, int quantity, decimal price)
    {
        Items.Add(new OrderItem(productId, quantity, price));
    }
}

// 2. OPEN/CLOSED - Mở rộng không sửa đổi
public interface IDiscountStrategy
{
    decimal ApplyDiscount(decimal originalPrice, Customer customer);
}

public class RegularCustomerDiscount : IDiscountStrategy
{
    public decimal ApplyDiscount(decimal originalPrice, Customer customer)
    {
        return originalPrice * 0.95m; // 5% discount
    }
}

public class VIPCustomerDiscount : IDiscountStrategy
{
    public decimal ApplyDiscount(decimal originalPrice, Customer customer)
    {
        return originalPrice * 0.85m; // 15% discount
    }
}

public class SeasonalDiscount : IDiscountStrategy  // Thêm mới không sửa code cũ
{
    public decimal ApplyDiscount(decimal originalPrice, Customer customer)
    {
        var now = DateTime.Now;
        if (now.Month == 12) // Tháng 12
            return originalPrice * 0.8m; // 20% discount
        return originalPrice;
    }
}

// 3. LISKOV SUBSTITUTION - Con thay thế cha an toàn
public abstract class PaymentProcessor
{
    public abstract Task<PaymentResult> ProcessAsync(decimal amount);
    
    // Template method - common behavior
    public async Task<PaymentResult> ProcessPaymentWithLogging(decimal amount)
    {
        LogPaymentAttempt(amount);
        var result = await ProcessAsync(amount);
        LogPaymentResult(result);
        return result;
    }
    
    protected virtual void LogPaymentAttempt(decimal amount) { }
    protected virtual void LogPaymentResult(PaymentResult result) { }
}

public class CreditCardProcessor : PaymentProcessor
{
    public override async Task<PaymentResult> ProcessAsync(decimal amount)
    {
        // Credit card specific logic
        return await ChargeCreditCard(amount);
    }
    
    private async Task<PaymentResult> ChargeCreditCard(decimal amount)
    {
        // Implementation
        return PaymentResult.Success();
    }
}

public class PayPalProcessor : PaymentProcessor  // Có thể thay thế CreditCardProcessor
{
    public override async Task<PaymentResult> ProcessAsync(decimal amount)
    {
        // PayPal specific logic
        return await ChargePayPal(amount);
    }
    
    private async Task<PaymentResult> ChargePayPal(decimal amount)
    {
        // Implementation
        return PaymentResult.Success();
    }
}

// 4. INTERFACE SEGREGATION - Interface nhỏ, chuyên biệt
public interface IOrderReader
{
    Task<Order> GetOrderAsync(int orderId);
    Task<List<Order>> GetOrdersByCustomerAsync(int customerId);
}

public interface IOrderWriter
{
    Task<int> CreateOrderAsync(Order order);
    Task UpdateOrderStatusAsync(int orderId, OrderStatus status);
}

public interface IOrderNotifier
{
    Task NotifyOrderCreatedAsync(Order order);
    Task NotifyOrderStatusChangedAsync(Order order, OrderStatus oldStatus);
}

// Services chỉ phụ thuộc interface cần thiết
public class OrderDisplayService
{
    private readonly IOrderReader _orderReader;  // Chỉ cần đọc
    
    public OrderDisplayService(IOrderReader orderReader)
    {
        _orderReader = orderReader;
    }
    
    public async Task<OrderViewModel> GetOrderDetailsAsync(int orderId)
    {
        var order = await _orderReader.GetOrderAsync(orderId);
        return MapToViewModel(order);
    }
}

public class OrderProcessingService
{
    private readonly IOrderWriter _orderWriter;      // Chỉ cần ghi
    private readonly IOrderNotifier _orderNotifier;  // Chỉ cần notify
    
    public OrderProcessingService(IOrderWriter orderWriter, IOrderNotifier orderNotifier)
    {
        _orderWriter = orderWriter;
        _orderNotifier = orderNotifier;
    }
    
    public async Task ProcessOrderAsync(CreateOrderRequest request)
    {
        var order = MapToOrder(request);
        var orderId = await _orderWriter.CreateOrderAsync(order);
        order.Id = orderId;
        
        await _orderNotifier.NotifyOrderCreatedAsync(order);
    }
}

// 5. DEPENDENCY INVERSION - Phụ thuộc abstraction
public class OrderService  // High-level module
{
    private readonly IOrderRepository _repository;      // Abstraction
    private readonly IPaymentProcessor _paymentProcessor; // Abstraction
    private readonly IInventoryService _inventoryService; // Abstraction
    private readonly IDiscountStrategy _discountStrategy; // Abstraction
    
    public OrderService(
        IOrderRepository repository,
        IPaymentProcessor paymentProcessor,
        IInventoryService inventoryService,
        IDiscountStrategy discountStrategy)
    {
        _repository = repository;
        _paymentProcessor = paymentProcessor;
        _inventoryService = inventoryService;
        _discountStrategy = discountStrategy;
    }
    
    public async Task<OrderResult> PlaceOrderAsync(PlaceOrderRequest request)
    {
        // Business logic không thay đổi dù đổi implementation
        
        // 1. Validate inventory
        var available = await _inventoryService.CheckAvailabilityAsync(request.Items);
        if (!available)
            return OrderResult.Failed("Some items are not available");
        
        // 2. Calculate price with discount
        var originalPrice = request.Items.Sum(i => i.Price * i.Quantity);
        var finalPrice = _discountStrategy.ApplyDiscount(originalPrice, request.Customer);
        
        // 3. Process payment
        var paymentResult = await _paymentProcessor.ProcessAsync(finalPrice);
        if (!paymentResult.IsSuccess)
            return OrderResult.Failed("Payment failed");
        
        // 4. Create order
        var order = new Order
        {
            CustomerId = request.CustomerId,
            Items = request.Items,
            Status = OrderStatus.Confirmed
        };
        
        await _repository.SaveOrderAsync(order);
        
        // 5. Update inventory
        await _inventoryService.ReserveItemsAsync(request.Items);
        
        return OrderResult.Success(order);
    }
}

// DI Configuration
public void ConfigureServices(IServiceCollection services)
{
    // Repository pattern
    services.AddScoped<IOrderRepository, SqlOrderRepository>();
    
    // Strategy pattern for discounts
    services.AddScoped<IDiscountStrategy>(provider => 
    {
        // Logic to choose strategy based on customer type
        var httpContext = provider.GetService<IHttpContextAccessor>()?.HttpContext;
        var customerType = httpContext?.User?.FindFirst("CustomerType")?.Value;
        
        return customerType switch
        {
            "VIP" => new VIPCustomerDiscount(),
            "Regular" => new RegularCustomerDiscount(),
            _ => new RegularCustomerDiscount()
        };
    });
    
    // Payment processing
    services.AddScoped<IPaymentProcessor, CreditCardProcessor>();
    
    // Other services
    services.AddScoped<IInventoryService, InventoryService>();
    services.AddScoped<OrderService>();
}
```

### 📊 **Metrics đánh giá SOLID:**

```csharp
public class CodeQualityMetrics
{
    // 1. SRP Metrics
    public static int CountResponsibilities(Type classType)
    {
        var methods = classType.GetMethods(BindingFlags.Public | BindingFlags.Instance);
        var responsibilities = methods
            .GroupBy(m => GetResponsibilityGroup(m.Name))
            .Count();
            
        return responsibilities; // Nên < 3
    }
    
    // 2. OCP Metrics  
    public static bool IsOpenForExtension(Type classType)
    {
        return classType.IsAbstract || 
               classType.IsInterface ||
               classType.GetMethods().Any(m => m.IsVirtual);
    }
    
    // 3. LSP Metrics
    public static bool ViolatesLSP(Type baseType, Type derivedType)
    {
        // Check if derived class throws exceptions not thrown by base
        // Check if derived class has stronger preconditions
        // Implementation depends on specific analysis
        return false;
    }
    
    // 4. ISP Metrics
    public static double CalculateInterfaceCohesion(Type interfaceType)
    {
        var methods = interfaceType.GetMethods();
        if (methods.Length <= 3) return 1.0; // Small interface is good
        
        // Calculate cohesion based on method relationships
        return 0.8; // Placeholder
    }
    
    // 5. DIP Metrics
    public static double CalculateAbstractionLevel(Type classType)
    {
        var dependencies = GetDependencies(classType);
        var abstractDependencies = dependencies.Count(d => d.IsInterface || d.IsAbstract);
        
        return dependencies.Count == 0 ? 1.0 : 
               (double)abstractDependencies / dependencies.Count;
    }
    
    private static List<Type> GetDependencies(Type classType)
    {
        // Analyze constructor parameters, field types, etc.
        var constructors = classType.GetConstructors();
        var dependencies = new List<Type>();
        
        foreach (var ctor in constructors)
        {
            dependencies.AddRange(ctor.GetParameters().Select(p => p.ParameterType));
        }
        
        return dependencies.Distinct().ToList();
    }
}
```

### 🏆 **Roadmap thành thạo SOLID:**

**📚 Tuần 1-2: Hiểu concept**
- [ ] Đọc hiểu 5 nguyên tắc với ví dụ đơn giản
- [ ] Nhận biết được code vi phạm SOLID
- [ ] Làm bài tập nhỏ cho từng nguyên tắc

**🔨 Tuần 3-4: Thực hành refactoring**
- [ ] Lấy 1 project cũ, identify vi phạm SOLID
- [ ] Refactor từng nguyên tắc một
- [ ] So sánh before/after

**🏗️ Tuần 5-6: Áp dụng vào dự án mới**
- [ ] Thiết kế architecture tuân thủ SOLID từ đầu
- [ ] Implement với Dependency Injection
- [ ] Viết unit tests

**🚀 Tuần 7-8: Nâng cao**
- [ ] Kết hợp SOLID với Design Patterns
- [ ] Áp dụng trong Microservices
- [ ] Code review với focus vào SOLID

---

*"SOLID không phỉ là quy tắc cứng nhắc, mà là nghệ thuật cân bằng giữa tính linh hoạt và đơn giản!"* ⚖️✨