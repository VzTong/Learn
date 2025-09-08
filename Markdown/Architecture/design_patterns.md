# 🧩 DESIGN PATTERNS - "CÔNG THỨC BÍ MẬT CỦA CÁC CỐ VẤN PHẦN MỀM"
*Chi tiết từ A-Z: Từ hiểu bản chất đến ứng dụng thực tế và trả lời phỏng vấn*

---

## 🎯 **DESIGN PATTERNS LÀ GÌ? TẠI SAO QUAN TRỌNG?**

### 🏗️ **Ví dụ đời thực: Xây dựng**
```
🏚️ Xây nhà không có thiết kế:
- Mỗi ngôi nhà đều phát minh từ đầu → Tốn thời gian, dễ sai
- Không có quy chuẩn → Khó bảo trì, mở rộng
- Thợ mới không hiểu → Khó làm việc nhóm

🏛️ Xây nhà theo bản thiết kế có sẵn:
- Sử dụng thiết kế đã được kiểm chứng → Nhanh, ít lỗi
- Có chuẩn chung → Dễ hiểu, dễ bảo trì
- Ai cũng hiểu thiết kế → Teamwork hiệu quả
```

**🎯 Design Patterns giúp code:**
- **Dễ hiểu**: Ai cũng biết "Factory tạo object", "Observer theo dõi sự kiện"
- **Tái sử dụng**: Giải pháp đã có, chỉ cần áp dụng
- **Bảo trì**: Code theo chuẩn, dễ sửa chữa và mở rộng
- **Giao tiếp**: Team dùng ngôn ngữ chung, hiểu nhau ngay

---

## 📖 **KIẾN THỨC CƠ BẢN**

### 🤔 **Design Pattern thực chất là gì?**
Design Pattern là **công thức đã được chuẩn hóa** để giải quyết các vấn đề thường gặp trong thiết kế phần mềm.
- **Không phải code cụ thể** → Là ý tưởng, template
- **Đã được kiểm chứng** → Qua nhiều dự án, nhiều dev sử dụng
- **Tái sử dụng được** → Áp dụng trong nhiều ngữ cảnh khác nhau

### 🏗️ **3 Nhóm chính - Gang of Four (GoF)**

![Design Patterns Overview](img/image.png)

```
┌─────────────────────────────────────────────────────────────┐
│                    🧩 DESIGN PATTERNS                       │
│                  Gang of Four (GoF) - 1994                 │
└─────────────────────────────────────────────────────────────┘

🏭 CREATIONAL PATTERNS          🔗 STRUCTURAL PATTERNS         🎭 BEHAVIORAL PATTERNS
   (Create Objects)                (Organize Classes)            (Manage Interactions)
┌─────────────────────┐        ┌─────────────────────┐        ┌─────────────────────┐
│  📦 SINGLETON       │        │  🔌 ADAPTER         │        │  👁️ OBSERVER        │
│  🏗️ FACTORY METHOD  │        │  🎨 DECORATOR       │        │  🎯 STRATEGY        │
│  👷 BUILDER         │        │  🏢 FACADE          │        │  🔄 STATE           │
│  🏭 ABSTRACT FACTORY│        │  🌉 PROXY           │        │  📋 COMMAND         │
│  📋 PROTOTYPE       │        │  🔗 COMPOSITE       │        │  📨 MEDIATOR        │
└─────────────────────┘        │  ⚖️ BRIDGE          │        │  💾 MEMENTO         │
                               │  🪶 FLYWEIGHT       │        │  🔧 TEMPLATE METHOD │
   Purpose:                    └─────────────────────┘        │  🔍 VISITOR         │
   • Control object creation                                  │  🔄 ITERATOR        │
   • Manage instantiation      Purpose:                       │  🏛️ INTERPRETER     │
   • Hide complexity          • Compose objects              │  ⛓️ CHAIN OF RESP.   │
                              • Structure relationships       └─────────────────────┘
   Examples:                  • Simplify interfaces
   • Database Connection                                      Purpose:
     (Kết nối cơ sở dữ liệu)                                 • Define communication
   • Logger Instance          Examples:                       • Manage algorithms
     (Thể hiện ghi log)       • API Wrapper (Facade)         • Handle state changes
   • Configuration Manager      (Bao bọc API - Mặt tiền)
     (Quản lý cấu hình)       • UI Components (Decorator)    Examples:
                               (Thành phần UI - Trang trí)   • Event System
                                                               (Hệ thống sự kiện)
                                                             • Algorithm Switching
                                                               (Chuyển đổi thuật toán)
                                                             • State Machines
                                                               (Máy trạng thái)
```

**🎯 Giải thích từng nhóm:**

```
1️⃣ 🏭 CREATIONAL PATTERNS (Tạo Objects)
   ├── SINGLETON → "Chỉ một thôi!" - Đảm bảo 1 instance duy nhất
   ├── FACTORY → "Xưởng sản xuất" - Tạo objects qua interface
   ├── BUILDER → "Xây từng bước" - Tạo objects phức tạp từng phần
   └── Purpose: Kiểm soát cách tạo objects, giấu complexity

2️⃣ 🔗 STRUCTURAL PATTERNS (Tổ chức Classes)
   ├── ADAPTER → "Bộ chuyển đổi" - Kết nối interfaces khác nhau
   ├── DECORATOR → "Trang trí" - Thêm tính năng không sửa code gốc
   ├── FACADE → "Mặt tiền" - Interface đơn giản cho hệ thống phức tạp
   └── Purpose: Tổ chức cấu trúc, kết nối objects hiệu quả

3️⃣ 🎭 BEHAVIORAL PATTERNS (Quản lý Tương tác)
   ├── OBSERVER → "Theo dõi" - Pub/Sub, notification system
   ├── STRATEGY → "Chiến lược" - Nhiều thuật toán, switch runtime
   ├── STATE → "Trạng thái" - Object thay đổi hành vi theo state
   └── Purpose: Quản lý giao tiếp, algorithms, responsibilities
```

## 🧠 **CÁCH NHỚ NHANH DESIGN PATTERNS**

### 🎯 **Nhóm CREATIONAL - "Tạo Object" (Nhớ: Xưởng sản xuất)**
```
🏭 Factory → Xưởng làm xe: Gọi "làm car" → ra Car, gọi "làm bike" → ra Bike
📦 Singleton → "Chỉ một thôi!": Như CEO công ty, chỉ có 1 người
👷 Builder → Xây nhà: Từng bước - móng → tường → mái → hoàn thành
🏭 Abstract Factory → Xưởng lớn: BMW factory làm BMW car + BMW bike
📋 Prototype → Photocopy: Copy từ bản gốc, không tạo từ đầu
```

### 🔗 **Nhóm STRUCTURAL - "Kết nối Object" (Nhớ: Lắp ráp)**
```
🔌 Adapter → Adapter điện: Chuyển đổi từ 3 chân sang 2 chân
🎨 Decorator → Trang trí bánh: Bánh + kem + cherry + chocolate...
🏢 Facade → Lễ tân khách sạn: 1 người lo hết, khách không cần biết bên trong
🌉 Proxy → Người đại diện: Thay mặt xử lý, có thể kiểm soát quyền truy cập
🔗 Composite → Cây thư mục: Folder chứa file, folder chứa folder khác
⚖️ Bridge → Cầu nối: Tách abstract và implementation
🪶 Flyweight → Chia sẻ: Dùng chung data để tiết kiệm memory
```

### 🎭 **Nhóm BEHAVIORAL - "Hành vi Object" (Nhớ: Giao tiếp)**
```
👁️ Observer → Đăng ký YouTube: Có video mới → thông báo tất cả subscriber
🎯 Strategy → Phương án: Thanh toán bằng thẻ/PayPal/tiền mặt - chọn 1
🔄 State → Trạng thái máy ATM: Chờ thẻ → Nhập PIN → Chọn giao dịch
📋 Command → Remote TV: Mỗi nút là 1 command, có thể undo
📨 Mediator → Trung gian: Air traffic control điều khiển máy bay
💾 Memento → Ctrl+Z: Lưu trạng thái để undo sau này
🔧 Template Method → Công thức nấu ăn: Khung chung, chi tiết tùy món
🔍 Visitor → Du khách: Thăm từng nơi, mỗi nơi có cách đón khác
🔄 Iterator → Duyệt danh sách: for each item in list
🏛️ Interpreter → Dịch thuật: Hiểu và thực thi ngôn ngữ tự định nghĩa
⛓️ Chain of Responsibility → Chuỗi xử lý: Đơn từ cấp thấp lên cấp cao
```

### 💡 **Mnemonic Tricks - Thủ thuật ghi nhớ:**
```
🎨 "CỚ SẮP BẤP" → Creational, Structural, Behavioral Patterns
🏭 "SFBAP" → Singleton, Factory, Builder, Abstract Factory, Prototype
🔗 "ADFPCBF" → Adapter, Decorator, Facade, Proxy, Composite, Bridge, Flyweight
🎭 "OSSC MTV CI" → Observer, Strategy, State, Command, Mediator, Template, Visitor, Chain, Iterator
```

### 🚀 **Câu thần chú nhớ nhanh:**
> **"Xưởng tạo (Creational), Lắp ráp (Structural), Giao tiếp (Behavioral)"**
> **"Factory tạo, Decorator trang trí, Observer theo dõi"**

---

## 🎤 **CÁCH TRẢ LỜI KHI PHỎNG VẤN**

### 📋 **Template trả lời chuyên nghiệp**
```
🎯 Bước 1: Định nghĩa ngắn gọn
"Design Pattern là tập hợp các giải pháp chuẩn hóa cho các vấn đề
thường gặp trong phát triển phần mềm."

💡 Bước 2: Lợi ích cốt lõi
"Nó giúp tăng khả năng tái sử dụng, giảm độ phức tạp và làm
hệ thống dễ mở rộng hơn."

🔧 Bước 3: Ví dụ cụ thể
"Ví dụ như Singleton đảm bảo chỉ có 1 instance của Database connection..."
```

### 🚀 **Flow vàng khi trả lời**
1. **Định nghĩa pattern** → Ngắn gọn, đúng trọng tâm
2. **Khi nào sử dụng** → Use case cụ thể, thực tế
3. **Code example** → Ngắn gọn, dễ hiểu
4. **Ưu/nhược điểm** → Thể hiện hiểu sâu

---

## 🏭 **CREATIONAL PATTERNS - TẠO OBJECT THÔNG MINH**

### 1️⃣ **SINGLETON - "CHỈ MỘT MÀ THÔI"**
*"Đảm bảo chỉ có duy nhất một instance trong suốt lifecycle ứng dụng"*

#### 🎯 **Khi nào sử dụng:**
- **Database Connection**: Chỉ cần 1 kết nối duy nhất
- **Logger**: Ghi log tập trung
- **Configuration**: Cài đặt toàn ứng dụng
- **Cache Manager**: Quản lý bộ nhớ đệm

#### ✅ **Code mẫu - Thread Safe:**
```csharp
public class DatabaseConnection
{
    private static DatabaseConnection _instance;
    private static readonly object _lockObject = new object();

    // Constructor private → Không ai tạo từ bên ngoài được
    private DatabaseConnection()
    {
        Console.WriteLine("🗄️ Database connection initialized!");
    }

    public static DatabaseConnection Instance
    {
        get
        {
            // Double-checked locking pattern
            if (_instance == null)
            {
                lock (_lockObject)
                {
                    if (_instance == null)
                        _instance = new DatabaseConnection();
                }
            }
            return _instance;
        }
    }

    public void ExecuteQuery(string sql)
    {
        Console.WriteLine($"📝 Executing: {sql}");
    }
}

// Sử dụng:
var db1 = DatabaseConnection.Instance;
var db2 = DatabaseConnection.Instance;
Console.WriteLine(db1 == db2); // True - Cùng 1 instance!
```

#### ⚠️ **Lưu ý quan trọng:**
```csharp
// ❌ Cách này không an toàn với multi-threading
public static DatabaseConnection Instance
{
    get
    {
        return _instance ??= new DatabaseConnection(); // Race condition!
    }
}

// ✅ Cách modern với Lazy<T>
public class ModernSingleton
{
    private static readonly Lazy<ModernSingleton> _lazy =
        new(() => new ModernSingleton());

    public static ModernSingleton Instance => _lazy.Value;

    private ModernSingleton() { }
}
```

### 2️⃣ **FACTORY METHOD - "NHẬN ĐẶT HÀNG"**
*"Cung cấp interface để tạo object, ẩn đi logic khởi tạo phức tạp"*

#### 🎯 **Khi nào sử dụng:**
- **Nhiều loại object**: Car, Bike, Truck... cùng interface Vehicle
- **Logic tạo phức tạp**: Cần validation, config trước khi tạo
- **Tách rời việc tạo**: Client không cần biết cách tạo object

#### ✅ **Code mẫu:**
```csharp
// Base class
public abstract class Vehicle
{
    public string Brand { get; set; }
    public abstract void Drive();
    public abstract double GetFuelConsumption();
}

// Concrete classes
public class Car : Vehicle
{
    public Car(string brand) => Brand = brand;
    public override void Drive() => Console.WriteLine($"🚗 {Brand} car driving smoothly");
    public override double GetFuelConsumption() => 8.5;
}

public class Motorcycle : Vehicle
{
    public Motorcycle(string brand) => Brand = brand;
    public override void Drive() => Console.WriteLine($"🏍️ {Brand} motorcycle speeding!");
    public override double GetFuelConsumption() => 3.2;
}

// Factory
public class VehicleFactory
{
    public static Vehicle CreateVehicle(string type, string brand)
    {
        return type.ToLower() switch
        {
            "car" => new Car(brand),
            "motorcycle" => new Motorcycle(brand),
            _ => throw new ArgumentException($"Unknown vehicle type: {type}")
        };
    }
}

// Sử dụng:
var car = VehicleFactory.CreateVehicle("car", "Toyota");
var bike = VehicleFactory.CreateVehicle("motorcycle", "Honda");
car.Drive();  // 🚗 Toyota car driving smoothly
```

---

## 🔗 **STRUCTURAL PATTERNS - KẾT NỐI THÔNG MINH**

### 3️⃣ **DECORATOR - "TRANG TRẠÍ LINH HOẠT"**
*"Thêm chức năng cho object mà không sửa code gốc"*

#### 🎯 **Khi nào sử dụng:**
- **Mở rộng tính năng**: Thêm tính năng mà không sửa class gốc
- **Kết hợp linh hoạt**: Coffee + Milk + Sugar + Cream...
- **Runtime decoration**: Thay đổi hành vi lúc chạy

#### ✅ **Code mẫu - Coffee Shop:**
```csharp
// Interface chung
public interface ICoffee
{
    string GetDescription();
    decimal GetCost();
}

// Coffee cơ bản
public class SimpleCoffee : ICoffee
{
    public string GetDescription() => "Simple Coffee";
    public decimal GetCost() => 20000;
}

// Base Decorator
public abstract class CoffeeDecorator : ICoffee
{
    protected ICoffee _coffee;

    public CoffeeDecorator(ICoffee coffee)
    {
        _coffee = coffee;
    }

    public virtual string GetDescription() => _coffee.GetDescription();
    public virtual decimal GetCost() => _coffee.GetCost();
}

// Concrete Decorators
public class MilkDecorator : CoffeeDecorator
{
    public MilkDecorator(ICoffee coffee) : base(coffee) { }

    public override string GetDescription() => _coffee.GetDescription() + " + Milk";
    public override decimal GetCost() => _coffee.GetCost() + 5000;
}

public class SugarDecorator : CoffeeDecorator
{
    public SugarDecorator(ICoffee coffee) : base(coffee) { }

    public override string GetDescription() => _coffee.GetDescription() + " + Sugar";
    public override decimal GetCost() => _coffee.GetCost() + 2000;
}

// Sử dụng:
ICoffee coffee = new SimpleCoffee();
coffee = new MilkDecorator(coffee);
coffee = new SugarDecorator(coffee);

Console.WriteLine($"{coffee.GetDescription()}: {coffee.GetCost()} VND");
// Output: Simple Coffee + Milk + Sugar: 27000 VND
```

### 4️⃣ **FACADE - "GIAO DIỆN ĐƠN GIẢN"**
*"Cung cấp interface đơn giản cho hệ thống phức tạp bên trong"*

#### 🎯 **Khi nào sử dụng:**
- **Hệ thống phức tạp**: Nhiều class, nhiều bước thực hiện
- **API đơn giản**: Client chỉ cần gọi 1 method
- **Giấu complexity**: Ẩn chi tiết implementation

#### ✅ **Code mẫu - Home Theater:**
```csharp
// Complex subsystems
public class DVDPlayer
{
    public void TurnOn() => Console.WriteLine("📀 DVD Player ON");
    public void Play(string movie) => Console.WriteLine($"▶️ Playing: {movie}");
    public void TurnOff() => Console.WriteLine("📀 DVD Player OFF");
}

public class SoundSystem
{
    public void TurnOn() => Console.WriteLine("🔊 Sound System ON");
    public void SetVolume(int volume) => Console.WriteLine($"🔊 Volume: {volume}");
    public void TurnOff() => Console.WriteLine("🔊 Sound System OFF");
}

public class Lights
{
    public void DimLights() => Console.WriteLine("💡 Lights dimmed");
    public void TurnOnLights() => Console.WriteLine("💡 Lights ON");
}

// Facade - Simple interface
public class HomeTheaterFacade
{
    private DVDPlayer _dvdPlayer;
    private SoundSystem _soundSystem;
    private Lights _lights;

    public HomeTheaterFacade()
    {
        _dvdPlayer = new DVDPlayer();
        _soundSystem = new SoundSystem();
        _lights = new Lights();
    }

    public void WatchMovie(string movie)
    {
        Console.WriteLine("🎬 Getting ready to watch movie...");
        _lights.DimLights();
        _soundSystem.TurnOn();
        _soundSystem.SetVolume(20);
        _dvdPlayer.TurnOn();
        _dvdPlayer.Play(movie);
        Console.WriteLine("🍿 Enjoy the movie!");
    }

    public void EndMovie()
    {
        Console.WriteLine("🎬 Shutting down theater...");
        _dvdPlayer.TurnOff();
        _soundSystem.TurnOff();
        _lights.TurnOnLights();
        Console.WriteLine("✅ Theater shutdown complete!");
    }
}

// Sử dụng - Cực kỳ đơn giản!
var theater = new HomeTheaterFacade();
theater.WatchMovie("Avengers: Endgame");
theater.EndMovie();
```

---

## 🎭 **BEHAVIORAL PATTERNS - GIAO TIẾP THÔNG MINH**

### 5️⃣ **OBSERVER - "THEO DÕI TIN TỨC"**
*"Một object thay đổi, các object liên quan được thông báo tự động"*

#### 🎯 **Khi nào sử dụng:**
- **Event System**: Click button → Update UI
- **Pub/Sub**: YouTube notification, Email newsletter
- **MVC Architecture**: Model thay đổi → View update

#### ✅ **Code mẫu - YouTube Channel:**
```csharp
// Observer interface
public interface ISubscriber
{
    void Update(string channelName, string videoTitle);
}

// Concrete Observer
public class Subscriber : ISubscriber
{
    public string Name { get; }

    public Subscriber(string name) => Name = name;

    public void Update(string channelName, string videoTitle)
    {
        Console.WriteLine($"📧 {Name} received notification: {channelName} uploaded '{videoTitle}'");
    }
}

// Subject
public class YouTubeChannel
{
    private List<ISubscriber> _subscribers = new List<ISubscriber>();
    private string _channelName;

    public YouTubeChannel(string channelName) => _channelName = channelName;

    public void Subscribe(ISubscriber subscriber)
    {
        _subscribers.Add(subscriber);
        Console.WriteLine($"➕ New subscriber for {_channelName}");
    }

    public void Unsubscribe(ISubscriber subscriber)
    {
        _subscribers.Remove(subscriber);
        Console.WriteLine($"➖ Subscriber left {_channelName}");
    }

    public void UploadVideo(string videoTitle)
    {
        Console.WriteLine($"🎥 {_channelName} uploaded: {videoTitle}");
        NotifySubscribers(videoTitle);
    }

    private void NotifySubscribers(string videoTitle)
    {
        foreach (var subscriber in _subscribers)
        {
            subscriber.Update(_channelName, videoTitle);
        }
    }
}

// Sử dụng:
var channel = new YouTubeChannel("TechTips VN");
var user1 = new Subscriber("Minh");
var user2 = new Subscriber("Hoa");

channel.Subscribe(user1);
channel.Subscribe(user2);
channel.UploadVideo("Design Patterns cho Beginners");
```

### 6️⃣ **STRATEGY - "NHIỀU CÁCH LÀM"**
*"Cho phép thay đổi thuật toán lúc runtime mà không sửa code chính"*

#### 🎯 **Khi nào sử dụng:**
- **Nhiều thuật toán**: Sort (QuickSort, MergeSort, BubbleSort)
- **Payment methods**: Credit Card, PayPal, Bitcoin
- **Runtime switching**: Thay đổi strategy theo điều kiện

#### ✅ **Code mẫu - Payment System:**
```csharp
// Strategy interface
public interface IPaymentStrategy
{
    bool ProcessPayment(decimal amount);
    string GetPaymentMethod();
}

// Concrete Strategies
public class CreditCardPayment : IPaymentStrategy
{
    private string _cardNumber;

    public CreditCardPayment(string cardNumber) => _cardNumber = cardNumber;

    public bool ProcessPayment(decimal amount)
    {
        Console.WriteLine($"💳 Processing ${amount} via Credit Card ****{_cardNumber.Substring(_cardNumber.Length - 4)}");
        // Logic xử lý thanh toán thẻ tín dụng
        return true;
    }

    public string GetPaymentMethod() => "Credit Card";
}

public class PayPalPayment : IPaymentStrategy
{
    private string _email;

    public PayPalPayment(string email) => _email = email;

    public bool ProcessPayment(decimal amount)
    {
        Console.WriteLine($"🅿️ Processing ${amount} via PayPal ({_email})");
        // Logic xử lý PayPal
        return true;
    }

    public string GetPaymentMethod() => "PayPal";
}

// Context
public class ShoppingCart
{
    private IPaymentStrategy _paymentStrategy;
    private decimal _totalAmount;

    public void SetPaymentStrategy(IPaymentStrategy paymentStrategy)
    {
        _paymentStrategy = paymentStrategy;
    }

    public void AddAmount(decimal amount)
    {
        _totalAmount += amount;
        Console.WriteLine($"🛒 Cart total: ${_totalAmount}");
    }

    public bool Checkout()
    {
        if (_paymentStrategy == null)
        {
            Console.WriteLine("❌ Please select a payment method");
            return false;
        }

        Console.WriteLine($"💰 Checkout with {_paymentStrategy.GetPaymentMethod()}");
        return _paymentStrategy.ProcessPayment(_totalAmount);
    }
}

// Sử dụng:
var cart = new ShoppingCart();
cart.AddAmount(100);
cart.AddAmount(250);

// Thay đổi strategy lúc runtime
cart.SetPaymentStrategy(new CreditCardPayment("1234567812345678"));
cart.Checkout();

cart.SetPaymentStrategy(new PayPalPayment("user@example.com"));
cart.Checkout();
```

---

## 🏆 **TỔNG KẾT CHO PHỎNG VẤN**

### 📊 **Quick Reference**
```
🏭 CREATIONAL (Tạo object)
├── Singleton → Chỉ 1 instance (Database, Logger)
├── Factory → Tạo object qua interface (Vehicle factory)
└── Builder → Xây dựng object phức tạp từng bước

🔗 STRUCTURAL (Kết nối object)
├── Adapter → Chuyển đổi interface
├── Decorator → Thêm tính năng không sửa code gốc
└── Facade → Interface đơn giản cho hệ thống phức tạp

🎭 BEHAVIORAL (Object giao tiếp)
├── Observer → Pub/Sub, Event handling
├── Strategy → Nhiều thuật toán, thay đổi runtime
└── Command → Đóng gói request thành object
```

### 💡 **Câu trả lời mẫu cho phỏng vấn**
> **"Design Pattern là giải pháp được chuẩn hóa cho các vấn đề thường gặp trong thiết kế phần mềm. Ví dụ, khi cần đảm bảo chỉ có 1 instance của Database connection, tôi sử dụng Singleton pattern với thread-safe implementation. Điều này giúp tiết kiệm tài nguyên và tránh conflict trong môi trường multi-threading."**

### 🎯 **Lời khuyên quan trọng:**
- **Đừng overuse**: Không phải lúc nào cũng cần pattern
- **Hiểu bản chất**: Tại sao dùng > Làm sao dùng
- **Code example**: Chuẩn bị sẵn 2-3 pattern thành thạo nhất
- **Trade-offs**: Biết ưu/nhược điểm của từng pattern