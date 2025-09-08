# 🌟 **DOMAIN DRIVEN DESIGN (DDD)**
*"Thiết Kế Theo Nghiệp Vụ - Code phải nói chuyện như chuyên gia"*

---

## 🎯 **TẠI SAO CẦN DDD?**

### 🤔 **Vấn đề của thiết kế truyền thống:**
```
❌ Technical-Driven Design:
Database → Entities → Services → Controllers
    ↓
Code nói "ngôn ngữ kỹ thuật":
- CreateUser(), UpdateStatus(), DeleteRecord()
- UserTable, OrderTable, CustomerTable
- GetById(), SaveToDatabase()

😵 Kết quả: 
- Business Expert không hiểu code
- Developer không hiểu nghiệp vụ  
- Miscommunication → Bugs → Failed projects
```

### ✨ **DDD Solution:**
```
✅ Domain-Driven Design:
Business Domain → Ubiquitous Language → Code
    ↓
Code nói "ngôn ngữ nghiệp vụ":
- RegisterPatient(), DischargePatient()
- Patient, Doctor, Prescription
- ScheduleAppointment(), PrescribeMedication()

🎉 Kết quả:
- Mọi người cùng "ngôn ngữ"
- Code phản ánh thực tế
- Ít bugs, delivery nhanh hơn
```

---

## 🗣️ **UBIQUITOUS LANGUAGE - NGÔN NGỮ CHUNG**

### 🏥 **Ví dụ: Hệ thống bệnh viện**

```csharp
❌ TECHNICAL LANGUAGE (Sai):
public class User
{
    public void UpdateStatus(int statusId) { ... }
    public void SetRole(string role) { ... }
}

public class UserService  
{
    public void CreateUser(UserDto dto) { ... }
    public void ChangeUserData(int id, object data) { ... }
}

// 🤯 Bác sĩ đọc code: "User? Status? Là gì vậy?"
```

```csharp
✅ UBIQUITOUS LANGUAGE (Đúng):
public class Patient
{
    public void Admit(Doctor attendingDoctor, Department department) { ... }
    public void Discharge(DischargeReason reason) { ... }
    public void TransferToDepartment(Department newDepartment) { ... }
}

public class DoctorService
{
    public void RegisterPatient(PatientRegistration registration) { ... }
    public void PrescribeMedication(PatientId patientId, Prescription prescription) { ... }
}

// 😍 Bác sĩ đọc code: "À, đúng rồi! Thế này mới đúng!"
```

### 📚 **Xây dựng Ubiquitous Language:**

```csharp
// 📁 Domain/ValueObjects/PatientId.cs
public class PatientId : ValueObject
{
    public string Value { get; private set; }
    
    private PatientId(string value) => Value = value;
    
    public static PatientId Create(string value)
    {
        if (string.IsNullOrWhiteSpace(value))
            throw new DomainException("Patient ID cannot be empty");
            
        if (!IsValidFormat(value))
            throw new DomainException("Patient ID must follow format: PT-YYYYMMDD-XXXX");
            
        return new PatientId(value);
    }
    
    public static PatientId Generate()
    {
        var dateStr = DateTime.Now.ToString("yyyyMMdd");
        var sequence = GenerateSequence();
        return new PatientId($"PT-{dateStr}-{sequence:D4}");
    }
    
    private static bool IsValidFormat(string value)
    {
        // PT-20241230-0001
        return Regex.IsMatch(value, @"^PT-\d{8}-\d{4}$");
    }
}

// 📁 Domain/ValueObjects/MedicalRecord.cs
public class MedicalRecord : ValueObject
{
    public PatientId PatientId { get; private set; }
    public string ChiefComplaint { get; private set; } // Lý do khám
    public string Diagnosis { get; private set; } // Chẩn đoán
    public List<Symptom> Symptoms { get; private set; } // Triệu chứng
    public DateTime RecordedAt { get; private set; }
    public DoctorId RecordedBy { get; private set; }
    
    public static MedicalRecord Create(
        PatientId patientId,
        string chiefComplaint,
        DoctorId doctorId)
    {
        if (string.IsNullOrWhiteSpace(chiefComplaint))
            throw new DomainException("Chief complaint is required");
            
        return new MedicalRecord
        {
            PatientId = patientId,
            ChiefComplaint = chiefComplaint,
            Symptoms = new List<Symptom>(),
            RecordedAt = DateTime.UtcNow,
            RecordedBy = doctorId
        };
    }
    
    public void AddSymptom(string description, SeverityLevel severity)
    {
        var symptom = Symptom.Create(description, severity);
        Symptoms.Add(symptom);
    }
    
    public void SetDiagnosis(string diagnosis, DoctorId doctorId)
    {
        if (RecordedBy != doctorId)
            throw new DomainException("Only the attending doctor can set diagnosis");
            
        Diagnosis = diagnosis;
    }
}

// 📁 Domain/Enums/PatientStatus.cs
public enum PatientStatus
{
    /// <summary>Đã đăng ký, chờ khám</summary>
    Registered,
    
    /// <summary>Đang khám bệnh</summary>
    UnderExamination,
    
    /// <summary>Chờ xét nghiệm</summary>
    AwaitingTestResults,
    
    /// <summary>Đang điều trị nội trú</summary>
    Hospitalized,
    
    /// <summary>Đã xuất viện</summary>
    Discharged,
    
    /// <summary>Chuyển viện</summary>
    Transferred
}
```

---

## 🏘️ **BOUNDED CONTEXT - NGỮ CẢNH GIỚI HẠN**

### 🎯 **Khái niệm:**
*"Cùng 1 từ nhưng ý nghĩa khác nhau trong từng ngữ cảnh"*

```
🏥 Bệnh viện có nhiều Bounded Context:

👩‍⚕️ MEDICAL CONTEXT (Khám chữa bệnh):
- Patient = Bệnh nhân cần điều trị
- Doctor = Bác sĩ chẩn đoán, kê đơn
- Appointment = Cuộc hẹn khám bệnh

💰 BILLING CONTEXT (Thanh toán):  
- Patient = Khách hàng phải trả tiền
- Doctor = Nhà cung cấp dịch vụ
- Appointment = Dịch vụ cần thanh toán

💊 PHARMACY CONTEXT (Nhà thuốc):
- Patient = Người mua thuốc  
- Doctor = Người kê đơn thuốc
- Prescription = Đơn thuốc cần bán

🗂️ HR CONTEXT (Nhân sự):
- Doctor = Nhân viên có lương, ca trực
- Department = Phòng ban
- Schedule = Lịch làm việc
```

### 🏗️ **Implementation Bounded Context:**

```csharp
// 📁 Medical.Domain/Entities/Patient.cs (Medical Context)
namespace Medical.Domain.Entities
{
    public class Patient : AggregateRoot
    {
        public PatientId Id { get; private set; }
        public PersonalInfo PersonalInfo { get; private set; }
        public MedicalHistory MedicalHistory { get; private set; }
        public PatientStatus Status { get; private set; }
        public List<Appointment> Appointments { get; private set; }
        
        public void ScheduleAppointment(DoctorId doctorId, DateTime scheduledTime, AppointmentType type)
        {
            // Business Rule: Không được đặt lịch trùng
            var existingAppointment = Appointments
                .FirstOrDefault(a => a.ScheduledTime.Date == scheduledTime.Date && 
                                   a.Status == AppointmentStatus.Scheduled);
                                   
            if (existingAppointment != null)
                throw new DomainException("Patient already has appointment on this date");
                
            var appointment = Appointment.Create(Id, doctorId, scheduledTime, type);
            Appointments.Add(appointment);
            
            // Domain Event
            AddDomainEvent(new AppointmentScheduledEvent(Id, appointment.Id, scheduledTime));
        }
        
        public void Admit(DoctorId attendingDoctor, DepartmentId department, string reason)
        {
            if (Status != PatientStatus.Registered)
                throw new DomainException("Patient must be registered before admission");
                
            Status = PatientStatus.Hospitalized;
            
            AddDomainEvent(new PatientAdmittedEvent(Id, attendingDoctor, department, reason));
        }
    }
}

// 📁 Billing.Domain/Entities/Patient.cs (Billing Context)  
namespace Billing.Domain.Entities
{
    public class Patient : AggregateRoot
    {
        public PatientId Id { get; private set; }
        public BillingInfo BillingInfo { get; private set; }
        public List<Invoice> Invoices { get; private set; }
        public PaymentStatus PaymentStatus { get; private set; }
        
        public Invoice CreateInvoice(List<BillableService> services)
        {
            var invoice = Invoice.Create(Id, services);
            Invoices.Add(invoice);
            
            UpdatePaymentStatus();
            
            AddDomainEvent(new InvoiceCreatedEvent(Id, invoice.Id, invoice.TotalAmount));
            
            return invoice;
        }
        
        public void ProcessPayment(decimal amount, PaymentMethod method)
        {
            var pendingInvoice = Invoices
                .FirstOrDefault(i => i.Status == InvoiceStatus.Pending);
                
            if (pendingInvoice == null)
                throw new DomainException("No pending invoice found");
                
            pendingInvoice.ProcessPayment(amount, method);
            UpdatePaymentStatus();
            
            AddDomainEvent(new PaymentProcessedEvent(Id, amount, method));
        }
    }
}

// 📁 Pharmacy.Domain/Entities/Patient.cs (Pharmacy Context)
namespace Pharmacy.Domain.Entities
{
    public class Patient : AggregateRoot
    {
        public PatientId Id { get; private set; }
        public ContactInfo ContactInfo { get; private set; }
        public List<PrescriptionFill> PrescriptionHistory { get; private set; }
        public List<DrugAllergy> KnownAllergies { get; private set; }
        
        public PrescriptionFill FillPrescription(Prescription prescription, PharmacistId pharmacistId)
        {
            // Business Rule: Kiểm tra dị ứng thuốc
            foreach (var medication in prescription.Medications)
            {
                var allergy = KnownAllergies
                    .FirstOrDefault(a => a.DrugName.Equals(medication.DrugName, StringComparison.OrdinalIgnoreCase));
                    
                if (allergy != null)
                    throw new DomainException($"Patient is allergic to {medication.DrugName}");
            }
            
            var fill = PrescriptionFill.Create(Id, prescription, pharmacistId);
            PrescriptionHistory.Add(fill);
            
            AddDomainEvent(new PrescriptionFilledEvent(Id, prescription.Id, pharmacistId));
            
            return fill;
        }
        
        public void AddDrugAllergy(string drugName, AllergySeverity severity, string notes)
        {
            var allergy = DrugAllergy.Create(drugName, severity, notes);
            KnownAllergies.Add(allergy);
            
            AddDomainEvent(new DrugAllergyAddedEvent(Id, drugName, severity));
        }
    }
}
```

---

## 🎯 **DDD BUILDING BLOCKS**

### 👑 **ENTITIES - Đối Tượng Có Danh Tính**
*"Quan trọng là AI, không phải gì"*

```csharp
// 📁 Domain/Entities/Doctor.cs
public class Doctor : AggregateRoot
{
    public DoctorId Id { get; private set; } // Identity
    public string Name { get; private set; }
    public Specialty Specialty { get; private set; }
    public LicenseNumber LicenseNumber { get; private set; }
    public List<Availability> Availabilities { get; private set; }
    
    // Behavior, not just data!
    public bool IsAvailableAt(DateTime dateTime)
    {
        var dayOfWeek = dateTime.DayOfWeek;
        var timeOfDay = dateTime.TimeOfDay;
        
        return Availabilities.Any(a => 
            a.DayOfWeek == dayOfWeek &&
            a.StartTime <= timeOfDay &&
            a.EndTime >= timeOfDay &&
            a.IsActive);
    }
    
    public Appointment ScheduleAppointment(PatientId patientId, DateTime scheduledTime, AppointmentType type)
    {
        // Business Rules
        if (!IsAvailableAt(scheduledTime))
            throw new DomainException("Doctor is not available at requested time");
            
        if (scheduledTime < DateTime.Now.AddHours(1))
            throw new DomainException("Appointment must be scheduled at least 1 hour in advance");
        
        var appointment = Appointment.Create(patientId, Id, scheduledTime, type);
        
        // Domain Event
        AddDomainEvent(new AppointmentScheduledEvent(patientId, Id, appointment.Id));
        
        return appointment;
    }
    
    public void UpdateSpecialty(Specialty newSpecialty, LicenseVerificationService verificationService)
    {
        // Domain Service để kiểm tra license
        if (!verificationService.IsValidForSpecialty(LicenseNumber, newSpecialty))
            throw new DomainException("Doctor license is not valid for this specialty");
            
        Specialty = newSpecialty;
        
        AddDomainEvent(new DoctorSpecialtyChangedEvent(Id, newSpecialty));
    }
}

// 📁 Domain/Entities/Appointment.cs
public class Appointment : Entity
{
    public AppointmentId Id { get; private set; }
    public PatientId PatientId { get; private set; }
    public DoctorId DoctorId { get; private set; }
    public DateTime ScheduledTime { get; private set; }
    public AppointmentType Type { get; private set; }
    public AppointmentStatus Status { get; private set; }
    public string Notes { get; private set; }
    
    public static Appointment Create(PatientId patientId, DoctorId doctorId, DateTime scheduledTime, AppointmentType type)
    {
        return new Appointment
        {
            Id = AppointmentId.Generate(),
            PatientId = patientId,
            DoctorId = doctorId,
            ScheduledTime = scheduledTime,
            Type = type,
            Status = AppointmentStatus.Scheduled,
            Notes = string.Empty
        };
    }
    
    public void Cancel(string reason)
    {
        if (Status == AppointmentStatus.Completed)
            throw new DomainException("Cannot cancel completed appointment");
            
        if (ScheduledTime < DateTime.Now.AddHours(2))
            throw new DomainException("Cannot cancel appointment less than 2 hours before scheduled time");
            
        Status = AppointmentStatus.Cancelled;
        Notes = $"Cancelled: {reason}";
    }
    
    public void Complete(string notes)
    {
        if (Status != AppointmentStatus.InProgress)
            throw new DomainException("Only in-progress appointments can be completed");
            
        Status = AppointmentStatus.Completed;
        Notes = notes;
    }
    
    public void Reschedule(DateTime newTime)
    {
        if (Status != AppointmentStatus.Scheduled)
            throw new DomainException("Only scheduled appointments can be rescheduled");
            
        ScheduledTime = newTime;
        Notes += $"\nRescheduled from {ScheduledTime} to {newTime}";
    }
}
```

### 💎 **VALUE OBJECTS - Đối Tượng Giá Trị**
*"Quan trọng là GÌ, không phải ai"*

```csharp
// 📁 Domain/ValueObjects/Address.cs
public class Address : ValueObject
{
    public string Street { get; private set; }
    public string City { get; private set; }
    public string Province { get; private set; }
    public string PostalCode { get; private set; }
    public string Country { get; private set; }
    
    private Address(string street, string city, string province, string postalCode, string country)
    {
        Street = street;
        City = city;
        Province = province;
        PostalCode = postalCode;
        Country = country;
    }
    
    public static Address Create(string street, string city, string province, string postalCode, string country = "Vietnam")
    {
        if (string.IsNullOrWhiteSpace(street))
            throw new DomainException("Street is required");
        if (string.IsNullOrWhiteSpace(city))
            throw new DomainException("City is required");
        if (string.IsNullOrWhiteSpace(province))
            throw new DomainException("Province is required");
            
        return new Address(street, city, province, postalCode, country);
    }
    
    public string GetFullAddress()
    {
        return $"{Street}, {City}, {Province}, {Country}";
    }
    
    public bool IsInSameCity(Address other)
    {
        return City.Equals(other.City, StringComparison.OrdinalIgnoreCase) &&
               Province.Equals(other.Province, StringComparison.OrdinalIgnoreCase);
    }
    
    // Value Objects so sánh bằng giá trị, không phải identity
    protected override IEnumerable<object> GetEqualityComponents()
    {
        yield return Street;
        yield return City;
        yield return Province;
        yield return PostalCode;
        yield return Country;
    }
}

// 📁 Domain/ValueObjects/Money.cs
public class Money : ValueObject
{
    public decimal Amount { get; private set; }
    public Currency Currency { get; private set; }
    
    private Money(decimal amount, Currency currency)
    {
        Amount = amount;
        Currency = currency;
    }
    
    public static Money Create(decimal amount, Currency currency)
    {
        if (amount < 0)
            throw new DomainException("Money amount cannot be negative");
            
        return new Money(amount, currency);
    }
    
    public static Money VND(decimal amount) => Create(amount, Currency.VND);
    public static Money USD(decimal amount) => Create(amount, Currency.USD);
    
    public Money Add(Money other)
    {
        if (Currency != other.Currency)
            throw new DomainException("Cannot add money with different currencies");
            
        return new Money(Amount + other.Amount, Currency);
    }
    
    public Money Subtract(Money other)
    {
        if (Currency != other.Currency)
            throw new DomainException("Cannot subtract money with different currencies");
            
        if (Amount < other.Amount)
            throw new DomainException("Insufficient funds");
            
        return new Money(Amount - other.Amount, Currency);
    }
    
    public Money MultiplyBy(decimal multiplier)
    {
        return new Money(Amount * multiplier, Currency);
    }
    
    public bool IsGreaterThan(Money other)
    {
        if (Currency != other.Currency)
            throw new DomainException("Cannot compare money with different currencies");
            
        return Amount > other.Amount;
    }
    
    protected override IEnumerable<object> GetEqualityComponents()
    {
        yield return Amount;
        yield return Currency;
    }
    
    public override string ToString()
    {
        return $"{Amount:N0} {Currency}";
    }
}

// 📁 Domain/ValueObjects/PhoneNumber.cs
public class PhoneNumber : ValueObject
{
    public string Value { get; private set; }
    public PhoneNumberType Type { get; private set; }
    
    private PhoneNumber(string value, PhoneNumberType type)
    {
        Value = value;
        Type = type;
    }
    
    public static PhoneNumber Create(string phoneNumber, PhoneNumberType type = PhoneNumberType.Mobile)
    {
        if (string.IsNullOrWhiteSpace(phoneNumber))
            throw new DomainException("Phone number is required");
            
        var normalized = NormalizePhoneNumber(phoneNumber);
        
        if (!IsValidVietnamesePhoneNumber(normalized))
            throw new DomainException("Invalid Vietnamese phone number format");
            
        return new PhoneNumber(normalized, type);
    }
    
    private static string NormalizePhoneNumber(string phoneNumber)
    {
        // Remove spaces, dashes, parentheses
        var digits = Regex.Replace(phoneNumber, @"[^\d+]", "");
        
        // Convert to international format
        if (digits.StartsWith("0"))
            digits = "+84" + digits.Substring(1);
        else if (!digits.StartsWith("+84"))
            digits = "+84" + digits;
            
        return digits;
    }
    
    private static bool IsValidVietnamesePhoneNumber(string phoneNumber)
    {
        // +84 followed by 9 digits (mobile) or 8-10 digits (landline)
        return Regex.IsMatch(phoneNumber, @"^\+84[0-9]{8,10}$");
    }
    
    public bool IsMobile()
    {
        return Type == PhoneNumberType.Mobile;
    }
    
    public string GetDisplayFormat()
    {
        // +84912345678 → 0912 345 678
        if (Value.StartsWith("+84"))
        {
            var domestic = "0" + Value.Substring(3);
            return FormatDomesticNumber(domestic);
        }
        
        return Value;
    }
    
    private static string FormatDomesticNumber(string number)
    {
        // 0912345678 → 0912 345 678
        if (number.Length == 10)
            return $"{number.Substring(0, 4)} {number.Substring(4, 3)} {number.Substring(7)}";
            
        return number;
    }
    
    protected override IEnumerable<object> GetEqualityComponents()
    {
        yield return Value;
        yield return Type;
    }
}
```

### 🏗️ **AGGREGATES - Nhóm Đối Tượng**
*"Ranh giới nhất quán, Root duy nhất"*

```csharp
// 📁 Domain/Aggregates/OrderAggregate/Order.cs
public class Order : AggregateRoot
{
    public OrderId Id { get; private set; }
    public PatientId PatientId { get; private set; }
    public OrderStatus Status { get; private set; }
    public DateTime OrderDate { get; private set; }
    public Money TotalAmount { get; private set; }
    
    // Aggregate chứa các Entity và Value Objects liên quan
    private readonly List<OrderItem> _items = new();
    public IReadOnlyList<OrderItem> Items => _items.AsReadOnly();
    
    public static Order Create(PatientId patientId)
    {
        var order = new Order
        {
            Id = OrderId.Generate(),
            PatientId = patientId,
            Status = OrderStatus.Draft,
            OrderDate = DateTime.UtcNow,
            TotalAmount = Money.VND(0)
        };
        
        order.AddDomainEvent(new OrderCreatedEvent(order.Id, patientId));
        
        return order;
    }
    
    // Business Logic: Thêm item vào order
    public void AddItem(ServiceId serviceId, string serviceName, Money price, int quantity = 1)
    {
        if (Status != OrderStatus.Draft)
            throw new DomainException("Cannot add items to non-draft order");
            
        if (quantity <= 0)
            throw new DomainException("Quantity must be positive");
        
        // Business Rule: Không duplicate service
        var existingItem = _items.FirstOrDefault(i => i.ServiceId == serviceId);
        if (existingItem != null)
        {
            existingItem.UpdateQuantity(existingItem.Quantity + quantity);
        }
        else
        {
            var item = OrderItem.Create(serviceId, serviceName, price, quantity);
            _items.Add(item);
        }
        
        RecalculateTotal();
        
        AddDomainEvent(new OrderItemAddedEvent(Id, serviceId, quantity));
    }
    
    public void RemoveItem(ServiceId serviceId)
    {
        if (Status != OrderStatus.Draft)
            throw new DomainException("Cannot remove items from non-draft order");
            
        var item = _items.FirstOrDefault(i => i.ServiceId == serviceId);
        if (item == null)
            throw new DomainException("Item not found in order");
            
        _items.Remove(item);
        RecalculateTotal();
        
        AddDomainEvent(new OrderItemRemovedEvent(Id, serviceId));
    }
    
    public void Submit()
    {
        if (Status != OrderStatus.Draft)
            throw new DomainException("Only draft orders can be submitted");
            
        if (!_items.Any())
            throw new DomainException("Cannot submit empty order");
            
        Status = OrderStatus.Submitted;
        
        AddDomainEvent(new OrderSubmittedEvent(Id, PatientId, TotalAmount));
    }
    
    public void Confirm()
    {
        if (Status != OrderStatus.Submitted)
            throw new DomainException("Only submitted orders can be confirmed");
            
        Status = OrderStatus.Confirmed;
        
        AddDomainEvent(new OrderConfirmedEvent(Id, PatientId));
    }
    
    public void Cancel(string reason)
    {
        if (Status == OrderStatus.Completed)
            throw new DomainException("Cannot cancel completed order");
            
        Status = OrderStatus.Cancelled;
        
        AddDomainEvent(new OrderCancelledEvent(Id, reason));
    }
    
    private void RecalculateTotal()
    {
        var total = Money.VND(0);
        
        foreach (var item in _items)
        {
            total = total.Add(item.GetTotalPrice());
        }
        
        TotalAmount = total;
    }
    
    // Aggregate Root là điểm duy nhất để truy cập
    public OrderItem GetItem(ServiceId serviceId)
    {
        return _items.FirstOrDefault(i => i.ServiceId == serviceId);
    }
    
    public bool HasService(ServiceId serviceId)
    {
        return _items.Any(i => i.ServiceId == serviceId);
    }
}

// 📁 Domain/Aggregates/OrderAggregate/OrderItem.cs
public class OrderItem : Entity
{
    public OrderItemId Id { get; private set; }
    public ServiceId ServiceId { get; private set; }
    public string ServiceName { get; private set; }
    public Money UnitPrice { get; private set; }
    public int Quantity { get; private set; }
    
    internal static OrderItem Create(ServiceId serviceId, string serviceName, Money unitPrice, int quantity)
    {
        return new OrderItem
        {
            Id = OrderItemId.Generate(),
            ServiceId = serviceId,
            ServiceName = serviceName,
            UnitPrice = unitPrice,
            Quantity = quantity
        };
    }
    
    internal void UpdateQuantity(int newQuantity)
    {
        if (newQuantity <= 0)
            throw new DomainException("Quantity must be positive");
            
        Quantity = newQuantity;
    }
    
    public Money GetTotalPrice()
    {
        return UnitPrice.MultiplyBy(Quantity);
    }
}
```

### 🏭 **DOMAIN SERVICES - Dịch Vụ Nghiệp Vụ**
*"Logic không thuộc về Entity nào cả"*

```csharp
// 📁 Domain/Services/AppointmentSchedulingService.cs
public interface IAppointmentSchedulingService
{
    Task<bool> CanScheduleAppointmentAsync(DoctorId doctorId, DateTime scheduledTime);
    Task<List<DateTime>> GetAvailableTimeSlotsAsync(DoctorId doctorId, DateTime date);
    Task<Doctor> FindBestAvailableDoctorAsync(Specialty specialty, DateTime preferredTime);
}

public class AppointmentSchedulingService : IAppointmentSchedulingService
{
    private readonly IDoctorRepository _doctorRepository;
    private readonly IAppointmentRepository _appointmentRepository;
    
    public AppointmentSchedulingService(
        IDoctorRepository doctorRepository,
        IAppointmentRepository appointmentRepository)
    {
        _doctorRepository = doctorRepository;
        _appointmentRepository = appointmentRepository;
    }
    
    public async Task<bool> CanScheduleAppointmentAsync(DoctorId doctorId, DateTime scheduledTime)
    {
        // 1. Kiểm tra bác sĩ có làm việc không
        var doctor = await _doctorRepository.GetByIdAsync(doctorId);
        if (doctor == null || !doctor.IsAvailableAt(scheduledTime))
            return false;
        
        // 2. Kiểm tra có appointment trùng không
        var existingAppointments = await _appointmentRepository
            .GetByDoctorAndDateAsync(doctorId, scheduledTime.Date);
            
        var hasConflict = existingAppointments.Any(a => 
            Math.Abs((a.ScheduledTime - scheduledTime).TotalMinutes) < 30);
            
        return !hasConflict;
    }
    
    public async Task<List<DateTime>> GetAvailableTimeSlotsAsync(DoctorId doctorId, DateTime date)
    {
        var doctor = await _doctorRepository.GetByIdAsync(doctorId);
        if (doctor == null)
            return new List<DateTime>();
        
        var availability = doctor.GetAvailabilityForDate(date);
        if (availability == null)
            return new List<DateTime>();
        
        var existingAppointments = await _appointmentRepository
            .GetByDoctorAndDateAsync(doctorId, date);
        
        var availableSlots = new List<DateTime>();
        var currentTime = date.Date.Add(availability.StartTime);
        var endTime = date.Date.Add(availability.EndTime);
        
        while (currentTime < endTime)
        {
            // Kiểm tra slot 30 phút có trống không
            var hasAppointment = existingAppointments.Any(a =>
                Math.Abs((a.ScheduledTime - currentTime).TotalMinutes) < 30);
                
            if (!hasAppointment)
                availableSlots.Add(currentTime);
                
            currentTime = currentTime.AddMinutes(30);
        }
        
        return availableSlots;
    }
    
    public async Task<Doctor> FindBestAvailableDoctorAsync(Specialty specialty, DateTime preferredTime)
    {
        var doctors = await _doctorRepository.GetBySpecialtyAsync(specialty);
        
        foreach (var doctor in doctors.OrderBy(d => d.Rating).Reverse())
        {
            if (await CanScheduleAppointmentAsync(doctor.Id, preferredTime))
                return doctor;
        }
        
        return null; // Không có bác sĩ nào rảnh
    }
}

// 📁 Domain/Services/PrescriptionValidationService.cs
public interface IPrescriptionValidationService
{
    Task<ValidationResult> ValidatePrescriptionAsync(Prescription prescription, PatientId patientId);
}

public class PrescriptionValidationService : IPrescriptionValidationService
{
    private readonly IPatientRepository _patientRepository;
    private readonly IDrugInteractionService _drugInteractionService;
    
    public async Task<ValidationResult> ValidatePrescriptionAsync(Prescription prescription, PatientId patientId)
    {
        var result = new ValidationResult();
        
        // 1. Kiểm tra dị ứng thuốc
        var patient = await _patientRepository.GetByIdAsync(patientId);
        foreach (var medication in prescription.Medications)
        {
            if (patient.IsAllergicTo(medication.DrugName))
            {
                result.AddError($"Patient is allergic to {medication.DrugName}");
            }
        }
        
        // 2. Kiểm tra tương tác thuốc
        var interactions = await _drugInteractionService
            .CheckInteractionsAsync(prescription.Medications.Select(m => m.DrugName));
            
        foreach (var interaction in interactions.Where(i => i.Severity == InteractionSeverity.Major))
        {
            result.AddWarning($"Major drug interaction: {interaction.Description}");
        }
        
        // 3. Kiểm tra liều dùng
        foreach (var medication in prescription.Medications)
        {
            if (medication.Dosage.IsExcessive(patient.Age, patient.Weight))
            {
                result.AddWarning($"High dosage for {medication.DrugName}: {medication.Dosage}");
            }
        }
        
        return result;
    }
}

public class ValidationResult
{
    public List<string> Errors { get; } = new();
    public List<string> Warnings { get; } = new();
    
    public bool IsValid => !Errors.Any();
    
    public void AddError(string error) => Errors.Add(error);
    public void AddWarning(string warning) => Warnings.Add(warning);
}
```

---

## 🎭 **DOMAIN EVENTS - SỰ KIỆN NGHIỆP VỤ**

### 📢 **Khái niệm:**
*"Khi có gì quan trọng xảy ra trong domain, thông báo cho mọi người biết"*

```csharp
// 📁 Domain/Events/PatientAdmittedEvent.cs
public class PatientAdmittedEvent : IDomainEvent
{
    public PatientId PatientId { get; }
    public DoctorId AttendingDoctor { get; }
    public DepartmentId Department { get; }
    public string AdmissionReason { get; }
    public DateTime OccurredAt { get; }
    
    public PatientAdmittedEvent(
        PatientId patientId, 
        DoctorId attendingDoctor, 
        DepartmentId department, 
        string admissionReason)
    {
        PatientId = patientId;
        AttendingDoctor = attendingDoctor;
        Department = department;
        AdmissionReason = admissionReason;
        OccurredAt = DateTime.UtcNow;
    }
}

// 📁 Domain/Events/PrescriptionIssuedEvent.cs
public class PrescriptionIssuedEvent : IDomainEvent
{
    public PrescriptionId PrescriptionId { get; }
    public PatientId PatientId { get; }
    public DoctorId DoctorId { get; }
    public List<MedicationInfo> Medications { get; }
    public DateTime OccurredAt { get; }
    
    public PrescriptionIssuedEvent(
        PrescriptionId prescriptionId,
        PatientId patientId,
        DoctorId doctorId,
        List<MedicationInfo> medications)
    {
        PrescriptionId = prescriptionId;
        PatientId = patientId;
        DoctorId = doctorId;
        Medications = medications;
        OccurredAt = DateTime.UtcNow;
    }
}

// 📁 Application/EventHandlers/PatientAdmittedEventHandler.cs
public class PatientAdmittedEventHandler : INotificationHandler<PatientAdmittedEvent>
{
    private readonly IBillingService _billingService;
    private readonly INotificationService _notificationService;
    private readonly IBedAssignmentService _bedAssignmentService;
    
    public PatientAdmittedEventHandler(
        IBillingService billingService,
        INotificationService notificationService,
        IBedAssignmentService bedAssignmentService)
    {
        _billingService = billingService;
        _notificationService = notificationService;
        _bedAssignmentService = bedAssignmentService;
    }
    
    public async Task Handle(PatientAdmittedEvent domainEvent, CancellationToken ct)
    {
        // 1️⃣ Tạo hồ sơ thanh toán
        await _billingService.CreateAdmissionBillAsync(
            domainEvent.PatientId, 
            domainEvent.Department);
        
        // 2️⃣ Assign giường bệnh
        await _bedAssignmentService.AssignBedAsync(
            domainEvent.PatientId, 
            domainEvent.Department);
        
        // 3️⃣ Thông báo cho các bên liên quan
        await _notificationService.NotifyPatientFamilyAsync(
            domainEvent.PatientId, 
            "Patient has been admitted to hospital");
            
        await _notificationService.NotifyDepartmentAsync(
            domainEvent.Department,
            $"New patient admitted: {domainEvent.PatientId}");
    }
}

// 📁 Application/EventHandlers/PrescriptionIssuedEventHandler.cs
public class PrescriptionIssuedEventHandler : INotificationHandler<PrescriptionIssuedEvent>
{
    private readonly IPharmacyService _pharmacyService;
    private readonly IInventoryService _inventoryService;
    
    public async Task Handle(PrescriptionIssuedEvent domainEvent, CancellationToken ct)
    {
        // 1️⃣ Gửi đơn thuốc đến nhà thuốc
        await _pharmacyService.ReceivePrescriptionAsync(
            domainEvent.PrescriptionId,
            domainEvent.PatientId,
            domainEvent.Medications);
        
        // 2️⃣ Kiểm tra tồn kho thuốc
        foreach (var medication in domainEvent.Medications)
        {
            var availability = await _inventoryService
                .CheckMedicationAvailabilityAsync(medication.DrugName, medication.Quantity);
                
            if (!availability.IsAvailable)
            {
                await _pharmacyService.NotifyShortageAsync(
                    medication.DrugName, 
                    medication.Quantity,
                    availability.CurrentStock);
            }
        }
    }
}
```

---

## 🏭 **REPOSITORIES - KHO DỮ LIỆU**

### 📚 **Repository Pattern trong DDD:**

```csharp
// 📁 Domain/Repositories/IPatientRepository.cs
public interface IPatientRepository
{
    // Aggregate Root repository methods
    Task<Patient> GetByIdAsync(PatientId id);
    Task<Patient> GetByPatientNumberAsync(string patientNumber);
    Task<List<Patient>> GetByDoctorAsync(DoctorId doctorId);
    
    Task SaveAsync(Patient patient);
    Task DeleteAsync(PatientId id);
    
    // Domain-specific queries
    Task<List<Patient>> FindPatientsAwaitingDischargeAsync();
    Task<List<Patient>> FindPatientsByDiagnosisAsync(string diagnosis);
    Task<bool> ExistsWithEmailAsync(Email email);
    
    // Specification pattern for complex queries
    Task<List<Patient>> FindAsync(ISpecification<Patient> specification);
    Task<PagedResult<Patient>> FindPagedAsync(ISpecification<Patient> specification, int page, int size);
}

// 📁 Infrastructure/Repositories/PatientRepository.cs
public class PatientRepository : IPatientRepository
{
    private readonly HospitalDbContext _context;
    
    public PatientRepository(HospitalDbContext context)
    {
        _context = context;
    }
    
    public async Task<Patient> GetByIdAsync(PatientId id)
    {
        var entity = await _context.Patients
            .Include(p => p.MedicalHistory)
            .Include(p => p.Appointments)
            .Include(p => p.Prescriptions)
            .FirstOrDefaultAsync(p => p.Id == id.Value);
            
        return entity?.ToDomainModel();
    }
    
    public async Task<Patient> GetByPatientNumberAsync(string patientNumber)
    {
        var entity = await _context.Patients
            .Include(p => p.MedicalHistory)
            .FirstOrDefaultAsync(p => p.PatientNumber == patientNumber);
            
        return entity?.ToDomainModel();
    }
    
    public async Task SaveAsync(Patient patient)
    {
        var entity = PatientEntity.FromDomainModel(patient);
        
        var existingEntity = await _context.Patients
            .FirstOrDefaultAsync(p => p.Id == patient.Id.Value);
            
        if (existingEntity == null)
        {
            _context.Patients.Add(entity);
        }
        else
        {
            _context.Entry(existingEntity).CurrentValues.SetValues(entity);
            
            // Handle child collections
            await UpdateAppointmentsAsync(existingEntity, patient.Appointments);
            await UpdatePrescriptionsAsync(existingEntity, patient.Prescriptions);
        }
    }
    
    public async Task<List<Patient>> FindPatientsAwaitingDischargeAsync()
    {
        var entities = await _context.Patients
            .Where(p => p.Status == PatientStatus.AwaitingDischarge)
            .Include(p => p.MedicalHistory)
            .ToListAsync();
            
        return entities.Select(e => e.ToDomainModel()).ToList();
    }
    
    public async Task<List<Patient>> FindAsync(ISpecification<Patient> specification)
    {
        var query = _context.Patients.AsQueryable();
        
        // Apply specification
        query = specification.Apply(query);
        
        var entities = await query
            .Include(p => p.MedicalHistory)
            .Include(p => p.Appointments)
            .ToListAsync();
            
        return entities.Select(e => e.ToDomainModel()).ToList();
    }
}

// 📁 Domain/Specifications/PatientSpecifications.cs
public static class PatientSpecifications
{
    public static ISpecification<Patient> ByAge(int minAge, int maxAge)
    {
        return new PatientByAgeSpecification(minAge, maxAge);
    }
    
    public static ISpecification<Patient> ByDiagnosis(string diagnosis)
    {
        return new PatientByDiagnosisSpecification(diagnosis);
    }
    
    public static ISpecification<Patient> AdmittedBetween(DateTime from, DateTime to)
    {
        return new PatientAdmittedBetweenSpecification(from, to);
    }
}

public class PatientByAgeSpecification : Specification<Patient>
{
    private readonly int _minAge;
    private readonly int _maxAge;
    
    public PatientByAgeSpecification(int minAge, int maxAge)
    {
        _minAge = minAge;
        _maxAge = maxAge;
    }
    
    public override Expression<Func<Patient, bool>> ToExpression()
    {
        return patient => patient.Age >= _minAge && patient.Age <= _maxAge;
    }
}
```

---

## 🧪 **TESTING DDD**

### 🎯 **Domain Testing:**

```csharp
// 📁 Tests/Domain.Tests/PatientTests.cs
public class PatientTests
{
    [Fact]
    public void Patient_Create_WithValidData_ShouldCreatePatient()
    {
        // Arrange
        var personalInfo = PersonalInfo.Create("John Doe", DateTime.Parse("1990-01-01"), Gender.Male);
        var contactInfo = ContactInfo.Create("john@example.com", "+84912345678");
        
        // Act
        var patient = Patient.Register(personalInfo, contactInfo);
        
        // Assert
        Assert.NotNull(patient);
        Assert.Equal("John Doe", patient.PersonalInfo.FullName);
        Assert.Equal(PatientStatus.Registered, patient.Status);
        
        // Domain Event should be raised
        var domainEvents = patient.GetDomainEvents();
        Assert.Single(domainEvents);
        Assert.IsType<PatientRegisteredEvent>(domainEvents.First());
    }
    
    [Fact]
    public void Patient_Admit_WhenNotRegistered_ShouldThrowException()
    {
        // Arrange
        var patient = CreateTestPatient();
        patient.Discharge(DischargeReason.Recovered); // Change status
        
        var doctorId = DoctorId.Generate();
        var departmentId = DepartmentId.Generate();
        
        // Act & Assert
        var exception = Assert.Throws<DomainException>(() =>
            patient.Admit(doctorId, departmentId, "Test reason"));
            
        Assert.Equal("Patient must be registered before admission", exception.Message);
    }
    
    [Fact]
    public void Patient_ScheduleAppointment_WhenAlreadyHasAppointmentOnSameDate_ShouldThrowException()
    {
        // Arrange
        var patient = CreateTestPatient();
        var doctorId = DoctorId.Generate();
        var appointmentTime = DateTime.Today.AddDays(1).AddHours(10);
        
        patient.ScheduleAppointment(doctorId, appointmentTime, AppointmentType.Consultation);
        
        // Act & Assert
        var exception = Assert.Throws<DomainException>(() =>
            patient.ScheduleAppointment(doctorId, appointmentTime.AddHours(2), AppointmentType.FollowUp));
            
        Assert.Contains("already has appointment on this date", exception.Message);
    }
    
    private Patient CreateTestPatient()
    {
        var personalInfo = PersonalInfo.Create("Test Patient", DateTime.Parse("1985-05-15"), Gender.Male);
        var contactInfo = ContactInfo.Create("test@example.com", "+84987654321");
        return Patient.Register(personalInfo, contactInfo);
    }
}

// 📁 Tests/Domain.Tests/ValueObjects/MoneyTests.cs
public class MoneyTests
{
    [Fact]
    public void Money_Add_WithSameCurrency_ShouldReturnCorrectSum()
    {
        // Arrange
        var money1 = Money.VND(100000);
        var money2 = Money.VND(50000);
        
        // Act
        var result = money1.Add(money2);
        
        // Assert
        Assert.Equal(150000, result.Amount);
        Assert.Equal(Currency.VND, result.Currency);
    }
    
    [Fact]
    public void Money_Add_WithDifferentCurrency_ShouldThrowException()
    {
        // Arrange
        var vnd = Money.VND(100000);
        var usd = Money.USD(100);
        
        // Act & Assert
        var exception = Assert.Throws<DomainException>(() => vnd.Add(usd));
        Assert.Equal("Cannot add money with different currencies", exception.Message);
    }
    
    [Theory]
    [InlineData(100000, 2, 200000)]
    [InlineData(75000, 1.5, 112500)]
    [InlineData(50000, 0.1, 5000)]
    public void Money_MultiplyBy_ShouldReturnCorrectResult(decimal amount, decimal multiplier, decimal expected)
    {
        // Arrange
        var money = Money.VND(amount);
        
        // Act
        var result = money.MultiplyBy(multiplier);
        
        // Assert
        Assert.Equal(expected, result.Amount);
    }
}

// 📁 Tests/Application.Tests/UseCases/RegisterPatientTests.cs
public class RegisterPatientTests
{
    private readonly Mock<IPatientRepository> _patientRepository;
    private readonly Mock<IUnitOfWork> _unitOfWork;
    private readonly RegisterPatientHandler _handler;
    
    public RegisterPatientTests()
    {
        _patientRepository = new Mock<IPatientRepository>();
        _unitOfWork = new Mock<IUnitOfWork>();
        _handler = new RegisterPatientHandler(_patientRepository.Object, _unitOfWork.Object);
    }
    
    [Fact]
    public async Task Handle_WithValidData_ShouldRegisterPatient()
    {
        // Arrange
        var command = new RegisterPatientCommand
        {
            FullName = "John Doe",
            DateOfBirth = DateTime.Parse("1990-01-01"),
            Gender = "Male",
            Email = "john@example.com",
            PhoneNumber = "0912345678"
        };
        
        _patientRepository.Setup(r => r.ExistsWithEmailAsync(It.IsAny<Email>()))
                         .ReturnsAsync(false);
        
        // Act
        var result = await _handler.Handle(command, CancellationToken.None);
        
        // Assert
        Assert.NotNull(result);
        Assert.Equal("John Doe", result.FullName);
        
        _patientRepository.Verify(r => r.SaveAsync(It.IsAny<Patient>()), Times.Once);
        _unitOfWork.Verify(u => u.CommitAsync(), Times.Once);
    }
    
    [Fact]
    public async Task Handle_WithExistingEmail_ShouldThrowException()
    {
        // Arrange
        var command = new RegisterPatientCommand
        {
            FullName = "John Doe",
            Email = "existing@example.com",
            // ... other properties
        };
        
        _patientRepository.Setup(r => r.ExistsWithEmailAsync(It.IsAny<Email>()))
                         .ReturnsAsync(true);
        
        // Act & Assert
        var exception = await Assert.ThrowsAsync<BusinessException>(() =>
            _handler.Handle(command, CancellationToken.None));
            
        Assert.Equal("Email already exists", exception.Message);
        
        _patientRepository.Verify(r => r.SaveAsync(It.IsAny<Patient>()), Times.Never);
    }
}
```

---

## 🎯 **ANTI-PATTERNS - TRÁNH SAI LẦM**

### ❌ **Anemic Domain Model:**
```csharp
❌ SAI - Anemic Model (Chỉ có data, không có behavior):
public class Patient
{
    public Guid Id { get; set; }
    public string Name { get; set; }
    public DateTime DateOfBirth { get; set; }
    public string Status { get; set; }
    // Chỉ có properties, không có methods!
}

public class PatientService
{
    public void AdmitPatient(Patient patient, Doctor doctor)
    {
        // Business logic ở Service, không ở Domain
        if (patient.Status != "Registered")
            throw new Exception("Invalid status");
            
        patient.Status = "Admitted";
        // Logic scattered everywhere!
    }
}
```

```csharp
✅ ĐÚNG - Rich Domain Model (Có behavior):
public class Patient : AggregateRoot
{
    public PatientId Id { get; private set; }
    public string Name { get; private set; }
    public PatientStatus Status { get; private set; }
    
    // Business logic trong Domain Object
    public void Admit(DoctorId attendingDoctor, DepartmentId department, string reason)
    {
        if (Status != PatientStatus.Registered)
            throw new DomainException("Patient must be registered before admission");
            
        Status = PatientStatus.Hospitalized;
        
        AddDomainEvent(new PatientAdmittedEvent(Id, attendingDoctor, department, reason));
    }
}
```

### ❌ **Domain Logic trong Application Services:**
```csharp
❌ SAI - Business Logic trong Application Layer:
public class AdmitPatientHandler
{
    public async Task Handle(AdmitPatientCommand command)
    {
        var patient = await _repository.GetByIdAsync(command.PatientId);
        
        // ❌ Business logic không thuộc về đây!
        if (patient.Age < 18 && command.Department == "Cardiology")
            throw new BusinessException("Children cannot be admitted to Cardiology");
            
        if (patient.HasInsurance && command.RoomType == "VIP")
        {
            // Complex insurance calculation logic...
        }
        
        patient.Status = PatientStatus.Hospitalized;
    }
}
```

```csharp
✅ ĐÚNG - Business Logic trong Domain:
public class Patient : AggregateRoot
{
    public void Admit(DoctorId attendingDoctor, DepartmentId department, RoomType roomType)
    {
        // ✅ Business rules ở đây!
        if (Age < 18 && department.IsAdultOnly())
            throw new DomainException("Minors cannot be admitted to adult-only departments");
            
        if (HasInsurance())
        {
            var insurance = Insurance.Calculate(roomType, department);
            // Domain logic here
        }
        
        Status = PatientStatus.Hospitalized;
        AddDomainEvent(new PatientAdmittedEvent(Id, attendingDoctor, department));
    }
}

public class AdmitPatientHandler
{
    public async Task Handle(AdmitPatientCommand command)
    {
        var patient = await _repository.GetByIdAsync(command.PatientId);
        
        // ✅ Simply delegate to domain
        patient.Admit(command.DoctorId, command.Department, command.RoomType);
        
        await _repository.SaveAsync(patient);
    }
}
```

---

## 🚀 **DDD VỚI MODERN TECH STACK**

### 🔄 **DDD + Event Sourcing:**
```csharp
// 📁 Domain/Aggregates/PatientEventSourced.cs
public class Patient : EventSourcedAggregateRoot
{
    public PatientId Id { get; private set; }
    public string Name { get; private set; }
    public PatientStatus Status { get; private set; }
    
    // Rebuild từ events
    public static Patient FromHistory(IEnumerable<IDomainEvent> events)
    {
        var patient = new Patient();
        foreach (var @event in events)
        {
            patient.Apply(@event);
        }
        return patient;
    }
    
    public void Register(PersonalInfo personalInfo, ContactInfo contactInfo)
    {
        var @event = new PatientRegisteredEvent(
            PatientId.Generate(),
            personalInfo.FullName,
            personalInfo.DateOfBirth,
            personalInfo.Gender,
            contactInfo.Email,
            contactInfo.PhoneNumber);
            
        ApplyChange(@event);
    }
    
    private void Apply(PatientRegisteredEvent @event)
    {
        Id = @event.PatientId;
        Name = @event.FullName;
        Status = PatientStatus.Registered;
    }
    
    private void Apply(PatientAdmittedEvent @event)
    {
        Status = PatientStatus.Hospitalized;
    }
}
```

### 🌐 **DDD + Microservices:**
```csharp
// Medical Bounded Context - Microservice 1
namespace Medical.API
{
    [ApiController]
    [Route("api/medical/patients")]
    public class PatientsController : ControllerBase
    {
        [HttpPost("{id}/admit")]
        public async Task<IActionResult> AdmitPatient(Guid id, AdmitPatientRequest request)
        {
            var command = new AdmitPatientCommand
            {
                PatientId = PatientId.From(id),
                DoctorId = DoctorId.From(request.DoctorId),
                DepartmentId = DepartmentId.From(request.DepartmentId),
                Reason = request.Reason
            };
            
            await _mediator.Send(command);
            
            // Publish integration event to other microservices
            await _eventBus.PublishAsync(new PatientAdmittedIntegrationEvent
            {
                PatientId = id,
                DepartmentId = request.DepartmentId,
                AdmittedAt = DateTime.UtcNow
            });
            
            return Ok();
        }
    }
}

// Billing Bounded Context - Microservice 2
namespace Billing.API
{
    public class PatientAdmittedIntegrationEventHandler : IIntegrationEventHandler<PatientAdmittedIntegrationEvent>
    {
        public async Task Handle(PatientAdmittedIntegrationEvent @event)
        {
            // Tạo billing record cho patient vừa admit
            var command = new CreateAdmissionBillCommand
            {
                PatientId = @event.PatientId,
                DepartmentId = @event.DepartmentId,
                AdmissionDate = @event.AdmittedAt
            };
            
            await _mediator.Send(command);
        }
    }
}
```

---

## 🎊 **KẾT LUẬN**

### ✨ **Lợi ích của DDD:**
1. **🗣️ Ubiquitous Language:** Mọi người cùng "ngôn ngữ"
2. **🏘️ Bounded Context:** Tách biệt rõ ràng các nghiệp vụ
3. **👑 Rich Domain Model:** Business logic tập trung, rõ ràng
4. **🧪 Testability:** Test business logic dễ dàng
5. **🔄 Maintainability:** Code phản ánh thực tế, dễ maintain

### 🎯 **Khi nào dùng DDD:**
- ✅ **Complex Business Domain**
- ✅ **Long-term Projects** (> 1 year)
- ✅ **Large Teams** with domain experts
- ✅ **Critical Business Applications**

### ⚠️ **Khi nào KHÔNG dùng:**
- ❌ **Simple CRUD Applications**
- ❌ **Prototypes/MVPs**
- ❌ **Small Teams** without domain expertise
- ❌ **Technical-focused Projects**

### 🚀 **Roadmap học DDD:**
1. **📚 Week 1-2:** Hiểu concepts (Ubiquitous Language, Bounded Context)
2. **🏗️ Week 3-4:** Thực hành Building Blocks (Entity, Value Object, Aggregate)
3. **⚡ Week 5-6:** Domain Events, Domain Services
4. **🧪 Week 7-8:** Testing strategies
5. **🌐 Week 9-12:** Integration với Clean Architecture, CQRS

---

*"DDD không chỉ là pattern, mà là cách tư duy về thiết kế phần mềm!"* 🌟