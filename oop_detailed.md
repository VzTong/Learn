# 🎯 OOP - Lập Trình Hướng Đối Tượng
*"Biến thế giới thực thành code - Nghệ thuật mô phỏng cuộc sống"*

---

## 🌟 **TẠI SAO OOP LÀI QUAN TRỌNG ĐẾN VẬY?**

### 🧠 **Bộ não con người và OOP**
```
🧭 Bộ não con người tự nhiên suy nghĩ theo OOP:
- Nhìn thấy "xe hơi" → biết ngay nó có bánh, có cửa, chạy được
- Thấy "con chó" → biết nó có 4 chân, biết sủa, biết chạy
- OOP mô phỏng cách tư duy tự nhiên này!
```

### 🏗️ **Code không OOP vs Code có OOP**

**❌ Code kiểu "tát nước theo mưa":**
```csharp
// Code hỗn loạn - Nightmare của developer
void ProcessOrder()
{
    // 200 dòng code lộn xộn
    var customerName = "";
    var productPrice = 0;
    var discount = 0;
    // Tính toán rải rác khắp nơi
    // Ai đọc cũng muốn khóc 😭
}
```

**✅ Code OOP - Sạch sẽ như căn hộ cao cấp:**
```csharp
// Mỗi thứ đều có chỗ của nó
public class Order
{
    public Customer Customer { get; set; }
    public List<Product> Products { get; set; }
    
    public decimal CalculateTotal()
    {
        return Products.Sum(p => p.GetPriceAfterDiscount());
    }
}

// Đọc như đọc truyện: "Đơn hàng tính tổng tiền"
```

---

## 🏛️ **4 TRỤ CỘT VÀNG CỦA OOP**

### 🔒 **1. ENCAPSULATION - Đóng Gói**
*"Giấu đi những gì không cần thiết, chỉ để lộ cái quan trọng"*

#### 🏠 **Ví dụ: Ngôi nhà thông minh**
```csharp
public class SmartHouse
{
    // PRIVATE - Bí mật gia đình, không ai được biết
    private string _securityCode = "123456";
    private bool _alarmSystem = false;
    private int _temperature = 25;
    
    // PUBLIC - Cửa chính, ai cũng có thể dùng
    public void EnterHouse(string code)
    {
        if (code == _securityCode)
        {
            Console.WriteLine("🏠 Chào mừng về nhà!");
            _alarmSystem = false;
        }
        else
        {
            Console.WriteLine("🚨 Mật khẩu sai! Báo động!");
            _alarmSystem = true;
        }
    }
    
    public void SetTemperature(int temp)
    {
        if (temp >= 16 && temp <= 30)  // Validation logic ẩn bên trong
        {
            _temperature = temp;
            Console.WriteLine($"🌡️ Nhiệt độ được đặt: {temp}°C");
        }
        else
        {
            Console.WriteLine("❄️🔥 Nhiệt độ không hợp lệ!");
        }
    }
    
    // PROTECTED - Chỉ con cái trong gia đình biết
    protected void CallMaintenance()
    {
        Console.WriteLine("🔧 Gọi thợ sửa chữa...");
    }
}

// Sử dụng:
var myHouse = new SmartHouse();
myHouse.EnterHouse("123456");  // ✅ OK
myHouse.SetTemperature(22);    // ✅ OK
// myHouse._securityCode;      // ❌ Không được! Compiler báo lỗi
```

#### 🎯 **Tại sao Encapsulation quan trọng?**
```
🛡️ BẢO MẬT: Không ai sửa được dữ liệu quan trọng
🔧 DỄ SỬA: Thay đổi logic bên trong mà không ảnh hưởng bên ngoài
🐛 ÍT BUG: Validate dữ liệu ngay từ đầu
📚 DỄ ĐỌC: Interface rõ ràng, không phải đoán
```

### 👪 **2. INHERITANCE - Kế Thừa**
*"Con cái thừa hưởng từ bố mẹ, nhưng có thể có tài năng riêng"*

#### 🚗 **Ví dụ: Gia đình xe hơi**
```csharp
// Bố mẹ - Class cha
public class Vehicle
{
    public string Brand { get; set; }
    public int Year { get; set; }
    public bool IsRunning { get; private set; }
    
    // Tất cả xe đều biết nổ máy
    public virtual void Start()
    {
        IsRunning = true;
        Console.WriteLine($"🚗 {Brand} đã nổ máy!");
    }
    
    public virtual void Stop()
    {
        IsRunning = false;
        Console.WriteLine($"🛑 {Brand} đã tắt máy!");
    }
}

// Con trai - Kế thừa + mở rộng
public class Car : Vehicle
{
    public int NumberOfDoors { get; set; }
    
    // Tính năng riêng của ô tô
    public void OpenTrunk()
    {
        Console.WriteLine("📦 Cốp xe đã mở!");
    }
    
    // Override - Làm theo cách riêng
    public override void Start()
    {
        Console.WriteLine("🔑 Kiểm tra dây an toàn...");
        base.Start();  // Vẫn gọi cách của bố mẹ
        Console.WriteLine("🎵 Bật nhạc trong xe!");
    }
}

// Con gái - Cũng kế thừa nhưng khác anh trai
public class Motorcycle : Vehicle
{
    public bool HasSidecar { get; set; }
    
    // Tính năng riêng của xe máy
    public void PopWheelie()
    {
        if (IsRunning)
            Console.WriteLine("🏍️ Bốc đầu xe máy!");
        else
            Console.WriteLine("❌ Phải nổ máy trước!");
    }
    
    public override void Start()
    {
        Console.WriteLine("⚡ Khởi động điện...");
        base.Start();
        Console.WriteLine("🏍️ Sẵn sàng phiêu lưu!");
    }
}

// Sử dụng:
var myCar = new Car 
{ 
    Brand = "Toyota", 
    Year = 2023, 
    NumberOfDoors = 4 
};

var myBike = new Motorcycle 
{ 
    Brand = "Honda", 
    Year = 2022, 
    HasSidecar = false 
};

myCar.Start();     // 🔑 Kiểm tra dây an toàn...
                   // 🚗 Toyota đã nổ máy!
                   // 🎵 Bật nhạc trong xe!

myBike.Start();    // ⚡ Khởi động điện...
                   // 🚗 Honda đã nổ máy!
                   // 🏍️ Sẵn sàng phiêu lưu!

myCar.OpenTrunk(); // 📦 Cốp xe đã mở!
myBike.PopWheelie(); // 🏍️ Bốc đầu xe máy!
```

### 🎭 **3. POLYMORPHISM - Đa Hình**
*"Một diễn viên, nhiều vai diễn - Cùng hành động, khác cách thực hiện"*

#### 🎪 **Ví dụ: Rạp xiếc động vật**
```csharp
// Interface chung - Kịch bản chung
public abstract class Animal
{
    public string Name { get; set; }
    
    // Tất cả động vật đều biết "biểu diễn", nhưng mỗi con một kiểu
    public abstract void Perform();
    public abstract void MakeSound();
}

// Mỗi con vật có cách biểu diễn riêng
public class Lion : Animal
{
    public override void Perform()
    {
        Console.WriteLine($"🦁 {Name} nhảy qua vòng lửa!");
    }
    
    public override void MakeSound()
    {
        Console.WriteLine("🦁 ROAAAAAR!");
    }
}

public class Elephant : Animal
{
    public override void Perform()
    {
        Console.WriteLine($"🐘 {Name} đứng bằng 2 chân sau!");
    }
    
    public override void MakeSound()
    {
        Console.WriteLine("🐘 Pawoooo!");
    }
}

public class Monkey : Animal
{
    public override void Perform()
    {
        Console.WriteLine($"🐵 {Name} đu dây qua lại!");
    }
    
    public override void MakeSound()
    {
        Console.WriteLine("🐵 Oook ook ook!");
    }
}

// Quản lý rạp xiếc - Polymorphism thể hiện ở đây!
public class CircusManager
{
    public void StartShow(List<Animal> animals)
    {
        Console.WriteLine("🎪 === CHƯƠNG TRÌNH BIỂU DIỄN BẮT ĐẦU ===");
        
        foreach (Animal animal in animals)
        {
            // Polymorphism: Không cần biết con vật gì
            // Chỉ cần gọi Perform(), mỗi con tự biết làm gì!
            animal.MakeSound();
            animal.Perform();
            Console.WriteLine("👏 Tràng pháo tay!");
            Console.WriteLine();
        }
        
        Console.WriteLine("🎊 === CHƯƠNG TRÌNH KẾT THÚC ===");
    }
}

// Sử dụng:
var animals = new List<Animal>
{
    new Lion { Name = "Simba" },
    new Elephant { Name = "Dumbo" },
    new Monkey { Name = "King Kong" }
};

var manager = new CircusManager();
manager.StartShow(animals);

// Kết quả:
// 🎪 === CHƯƠNG TRÌNH BIỂU DIỄN BẮT ĐẦU ===
// 🦁 ROAAAAAR!
// 🦁 Simba nhảy qua vòng lửa!
// 👏 Tràng pháo tay!
//
// 🐘 Pawoooo!
// 🐘 Dumbo đứng bằng 2 chân sau!
// 👏 Tràng pháo tay!
//
// 🐵 Oook ook ook!
// 🐵 King Kong đu dây qua lại!
// 👏 Tràng pháo tay!
//
// 🎊 === CHƯƠNG TRÌNH KẾT THÚC ===
```

#### 💡 **Polymorphism có gì hay?**
```
🔄 LINH HOẠT: Thêm động vật mới không cần sửa CircusManager
🎯 CLEAN CODE: Không cần if-else dài loằng ngoằng
🚀 MỞ RỘNG: Dễ dàng scale up ứng dụng
🧩 MODULE: Mỗi class độc lập, dễ test
```

### 🎨 **4. ABSTRACTION - Trừu Tượng**
*"Tập trung vào cái quan trọng, bỏ qua chi tiết phức tạp"*

#### 🚗 **Ví dụ: Lái xe ô tô**
```csharp
// Interface đơn giản cho người dùng
public interface ICar
{
    void StartEngine();
    void Accelerate();
    void Brake();
    void TurnLeft();
    void TurnRight();
}

// Implementation phức tạp ẩn bên trong
public class ModernCar : ICar
{
    // Hàng trăm chi tiết phức tạp ẩn bên trong
    private Engine _engine;
    private TransmissionSystem _transmission;
    private BrakeSystem _brakes;
    private SteeringSystem _steering;
    private FuelInjectionSystem _fuelSystem;
    private IgnitionSystem _ignition;
    
    public void StartEngine()
    {
        // Ẩn đi 50+ bước phức tạp:
        _ignition.CheckBattery();
        _fuelSystem.PrimeFuelPump();
        _engine.CheckOilPressure();
        _transmission.SetToNeutral();
        _ignition.StartSequence();
        // ... và nhiều thứ khác
        
        Console.WriteLine("🚗 Xe đã khởi động!");
    }
    
    public void Accelerate()
    {
        // Ẩn đi logic phức tạp:
        _fuelSystem.IncreaseFuelFlow();
        _transmission.ShiftGear();
        _engine.IncreaseThrottle();
        
        Console.WriteLine("🚀 Xe tăng tốc!");
    }
    
    public void Brake()
    {
        // Ẩn đi hệ thống phanh phức tạp:
        _brakes.ApplyDiscBrakes();
        _brakes.ApplyABS();
        _transmission.EngineBreaking();
        
        Console.WriteLine("🛑 Xe giảm tốc!");
    }
    
    public void TurnLeft()
    {
        _steering.CalculateAngle(-15);
        _steering.AdjustWheels();
        Console.WriteLine("⬅️ Xe rẽ trái!");
    }
    
    public void TurnRight()
    {
        _steering.CalculateAngle(15);
        _steering.AdjustWheels();
        Console.WriteLine("➡️ Xe rẽ phải!");
    }
}

// Người lái xe chỉ cần biết interface đơn giản
public class Driver
{
    public void DriveToWork(ICar car)
    {
        Console.WriteLine("👨‍💼 Đi làm thôi!");
        
        car.StartEngine();    // Không cần biết 50+ bước bên trong
        car.Accelerate();     // Không cần biết về fuel injection
        car.TurnLeft();       // Không cần biết công thức tính góc
        car.Accelerate();
        car.TurnRight();
        car.Brake();          // Không cần biết về ABS
        
        Console.WriteLine("🏢 Đã đến công ty!");
    }
}

// Sử dụng:
var myCar = new ModernCar();
var me = new Driver();
me.DriveToWork(myCar);
```

#### 🎯 **Abstraction giúp gì?**
```
🧠 DỄ HỌC: Chỉ học interface, không cần hiểu hết implementation
🔧 DỄ SỬA: Thay đổi bên trong không ảnh hưởng người dùng
🚀 HIỆU QUẢ: Tập trung vào logic business, không sa lầy vào chi tiết
🧩 MODULARITY: Mỗi phần độc lập, dễ test và maintain
```

---

## 🎯 **BÀI TẬP THỰC HÀNH SIÊU THỰC TẾ**

### 🏥 **Dự án: Hệ thống Quản lý Bệnh viện**

```csharp
// 1. ABSTRACTION - Interface chung
public interface IMedicalStaff
{
    string Name { get; set; }
    string ID { get; set; }
    void StartWorkDay();
    void TreatPatient(Patient patient);
    void EndWorkDay();
}

// 2. INHERITANCE + POLYMORPHISM
public abstract class MedicalStaff : IMedicalStaff
{
    public string Name { get; set; }
    public string ID { get; set; }
    
    public virtual void StartWorkDay()
    {
        Console.WriteLine($"👨‍⚕️ {Name} bắt đầu ca làm việc");
    }
    
    public abstract void TreatPatient(Patient patient);
    
    public virtual void EndWorkDay()
    {
        Console.WriteLine($"👨‍⚕️ {Name} kết thúc ca làm việc");
    }
}

public class Doctor : MedicalStaff
{
    public string Specialty { get; set; }
    
    public override void TreatPatient(Patient patient)
    {
        Console.WriteLine($"🩺 Bác sĩ {Name} chuyên khoa {Specialty} khám cho {patient.Name}");
        patient.Diagnose($"Chẩn đoán bởi BS {Name}");
    }
    
    public void PerformSurgery(Patient patient)
    {
        Console.WriteLine($"🏥 BS {Name} thực hiện phẫu thuật cho {patient.Name}");
    }
}

public class Nurse : MedicalStaff
{
    public override void TreatPatient(Patient patient)
    {
        Console.WriteLine($"💉 Y tá {Name} chăm sóc {patient.Name}");
        patient.GiveMedicine("Thuốc theo đơn bác sĩ");
    }
    
    public void TakeVitalSigns(Patient patient)
    {
        Console.WriteLine($"🌡️ Y tá {Name} đo sinh hiệu cho {patient.Name}");
    }
}

// 3. ENCAPSULATION
public class Patient
{
    private string _medicalRecord = "";  // Bí mật y tế
    private List<string> _medications = new();
    
    public string Name { get; set; }
    public int Age { get; set; }
    public string PatientID { get; private set; }  // Chỉ đọc từ bên ngoài
    
    public Patient(string name, int age)
    {
        Name = name;
        Age = age;
        PatientID = GeneratePatientID();
    }
    
    // PUBLIC methods - Interface cho bên ngoài
    public void Diagnose(string diagnosis)
    {
        _medicalRecord += $"\n📋 {DateTime.Now}: {diagnosis}";
        Console.WriteLine($"📝 Đã ghi chẩn đoán cho {Name}");
    }
    
    public void GiveMedicine(string medicine)
    {
        _medications.Add($"{DateTime.Now}: {medicine}");
        Console.WriteLine($"💊 {Name} đã được cho thuốc: {medicine}");
    }
    
    public void GetMedicalSummary()  // Chỉ hiện thông tin cần thiết
    {
        Console.WriteLine($"📊 Tóm tắt bệnh án {Name}:");
        Console.WriteLine($"   - Tuổi: {Age}");
        Console.WriteLine($"   - Số lượng chẩn đoán: {CountDiagnoses()}");
        Console.WriteLine($"   - Số loại thuốc: {_medications.Count}");
    }
    
    // PRIVATE methods - Logic nội bộ
    private string GeneratePatientID()
    {
        return $"BN{DateTime.Now.Year}{Random.Shared.Next(1000, 9999)}";
    }
    
    private int CountDiagnoses()
    {
        return _medicalRecord.Split('\n', StringSplitOptions.RemoveEmptyEntries).Length;
    }
}

// 4. POLYMORPHISM trong thực tế
public class Hospital
{
    private List<IMedicalStaff> _staff = new();
    private List<Patient> _patients = new();
    
    public void AddStaff(IMedicalStaff staff)
    {
        _staff.Add(staff);
        Console.WriteLine($"➕ Đã thêm nhân viên: {staff.Name}");
    }
    
    public void AdmitPatient(Patient patient)
    {
        _patients.Add(patient);
        Console.WriteLine($"🏥 Đã tiếp nhận bệnh nhân: {patient.Name}");
    }
    
    public void StartDailyOperation()
    {
        Console.WriteLine("🌅 === BẮT ĐẦU NGÀY LÀM VIỆC ===");
        
        // Polymorphism: Mỗi nhân viên tự biết cách bắt đầu ca
        foreach (var staff in _staff)
        {
            staff.StartWorkDay();
        }
        
        // Chăm sóc bệnh nhân
        for (int i = 0; i < _patients.Count && i < _staff.Count; i++)
        {
            _staff[i].TreatPatient(_patients[i]);
        }
        
        Console.WriteLine("🌇 === KẾT THÚC NGÀY LÀM VIỆC ===");
        foreach (var staff in _staff)
        {
            staff.EndWorkDay();
        }
    }
}

// Sử dụng tất cả concepts:
var hospital = new Hospital();

// Thêm nhân viên (Polymorphism)
hospital.AddStaff(new Doctor { Name = "Dr. Hòa", ID = "D001", Specialty = "Tim mạch" });
hospital.AddStaff(new Nurse { Name = "Y tá Lan", ID = "N001" });

// Thêm bệnh nhân (Encapsulation)
hospital.AdmitPatient(new Patient("Anh Minh", 35));
hospital.AdmitPatient(new Patient("Chị Hường", 28));

// Hoạt động hàng ngày
hospital.StartDailyOperation();
```

---

## 🚀 **TIPS MASTER OOP NHANH CHÓNG**

### 💡 **Quy tắc 4-3-2-1:**
- **4 tuần:** Nắm vững 4 tính chất OOP
- **3 dự án nhỏ:** Áp dụng từng tính chất riêng biệt  
- **2 dự án lớn:** Kết hợp tất cả tính chất
- **1 dự án thực tế:** Làm cho công ty/khách hàng

### 🎯 **Checklist thành thạo OOP:**
```
□ Có thể giải thích OOP cho người không biết lập trình
□ Thiết kế được class diagram cho bài toán thực tế
□ Biết khi nào dùng inheritance, khi nào dùng composition
□ Code không có duplicated logic
□ Class có single responsibility rõ ràng
□ Interface design clean và intuitive
```

### 🏆 **Bài tập về nhà:**
1. **Tuần 1:** Mô hình hóa trường học (Student, Teacher, Course)
2. **Tuần 2:** Hệ thống ngân hàng (Account, Transaction, Customer)  
3. **Tuần 3:** Game đơn giản (Character, Weapon, Skill)
4. **Tuần 4:** E-commerce mini (Product, Order, User)

---

*"OOP không chỉ là cách viết code, mà là cách suy nghĩ về thế giới!"* 🌟

**Tiếp theo:** LINQ - Siêu ngôn ngữ truy vấn 🔍