# 🏗️ MVC, MVP, MVVM - "BỘ BA KIẾN TRÚC UI KINH ĐIỂN"
*Chi tiết từ A-Z: So sánh 3 patterns, ứng dụng thực tế và trả lời phỏng vấn*

---

## 🎯 **MVC LÀ GÌ? TẠI SAO QUAN TRỌNG?**

### 🏠 **Ví dụ đời thực: Nhà hàng**
```
🏚️ Nhà hàng không tổ chức:
- Chef vừa nấu ăn, vừa ra nhận order, vừa tính tiền → Loạn xì
- Khách hàng phải vào bếp để gọi món → Bẩn thỉu
- Không có ai quản lý → Khó mở rộng

🏛️ Nhà hàng có tổ chức (MVC):
- Chef (Model): Chỉ lo nấu ăn, không lo gì khác
- Waiter (Controller): Nhận order, giao tiếp với Chef
- Menu/Bàn ăn (View): Khách hàng nhìn và tương tác
```

**🎯 MVC giúp code:**
- **Phân tách trách nhiệm**: Mỗi thành phần có vai trò riêng biệt
- **Dễ bảo trì**: Thay đổi UI không ảnh hưởng business logic
- **Dễ test**: Có thể test từng phần độc lập
- **Teamwork tốt**: Người làm UI, người làm logic có thể work song song

---

## 📖 **KIẾN THỨC CƠ BẢN**

### 🤔 **MVC thực chất là gì?**
MVC (Model-View-Controller) là **mẫu kiến trúc** chia ứng dụng thành 3 phần riêng biệt:

## 🏗️ **Kiến trúc MVC - Data Flow Diagram**

```
┌─────────────────────────────────────────────────────────────┐
│                    🌐 MVC ARCHITECTURE                      │
└─────────────────────────────────────────────────────────────┘

     👤 USER
       │
       │ (1) Request/Action
       ▼
┌─────────────┐    (2) User Input     ┌─────────────────┐
│    👁️ VIEW    │ ──────────────────→ │  🎮 CONTROLLER   │
│   (UI Layer) │                     │  (Orchestrator)  │
│              │ ←────────────────── │                 │
│ - Display UI │    (6) Render       │ - Handle Request│
│ - Show Data  │        Response     │ - Route Actions │
│ - User Input │                     │ - Select View   │
└─────────────┘                     └─────────────────┘
       ▲                                       │
       │                                       │
       │ (5) Return                           │ (3) Get/Update
       │     Data                             │     Data
       │                                      ▼
       │                             ┌─────────────────┐
       │                             │   🏭 MODEL       │
       └─────────────────────────────│  (Data + Logic) │
              (4) Processed Data     │                 │
                                     │ - Business Logic│
                                     │ - Data Access   │
                                     │ - Validation    │
                                     │ - Database      │
                                     └─────────────────┘
                                              │
                                              │ (Database Call)
                                              ▼
                                     ┌─────────────────┐
                                     │   💾 DATABASE    │
                                     │   (Storage)     │
                                     └─────────────────┘
```

### 📊 **Flow giải thích chi tiết:**
```
1️⃣ USER tương tác với VIEW (click button, submit form...)
2️⃣ VIEW gửi user input đến CONTROLLER
3️⃣ CONTROLLER xử lý request, gọi MODEL để get/update data
4️⃣ MODEL thực hiện business logic, truy xuất database, trả về data
5️⃣ CONTROLLER nhận data từ Model
6️⃣ CONTROLLER chọn VIEW thích hợp và gửi data để render response
```

### 🧩 **Chi tiết từng thành phần:**

```
🏭 MODEL (Dữ liệu & Logic)
├── Quản lý dữ liệu
├── Business rules & validation
├── Database interaction
└── Không biết gì về UI

👁️ VIEW (Giao diện người dùng)
├── Hiển thị dữ liệu
├── Nhận input từ user
├── UI components (forms, buttons...)
└── Không chứa business logic

🎮 CONTROLLER (Điều phối viên)
├── Nhận request từ View
├── Gọi Model để xử lý
├── Chọn View phù hợp để response
└── Là cầu nối giữa Model và View
```

### 🆚 **Tại sao không viết hết trong View?**

#### ❌ **Code "tự làm hết" - Spaghetti Code:**
```csharp
// UI vừa hiển thị vừa xử lý logic - NIGHTMARE!
public class ProductPage
{
    public void ShowProducts()
    {
        // UI code
        Console.WriteLine("=== PRODUCT LIST ===");

        // Database code - SAI!
        var conn = new SqlConnection("Server=.;Database=Shop;");
        var cmd = new SqlCommand("SELECT Name, Price FROM Product", conn);

        // Business logic - SAI!
        conn.Open();
        var reader = cmd.ExecuteReader();
        decimal totalValue = 0;

        while (reader.Read())
        {
            var name = reader["Name"].ToString();
            var price = (decimal)reader["Price"];

            // Validation logic - SAI!
            if (price < 0) price = 0;

            // Display logic
            Console.WriteLine($"📦 {name}: ${price}");
            totalValue += price;
        }

        // More UI code
        Console.WriteLine($"💰 Total: ${totalValue}");
        conn.Close();
    }
}

// Vấn đề: 1 method làm 5 việc!
// - Hiển thị UI
// - Kết nối Database
// - Xử lý business logic
// - Validation
// - Tính toán
```

#### ✅ **Code MVC - Clean & Maintainable:**
```csharp
// MODEL - Chỉ lo dữ liệu & logic
public class Product
{
    public int Id { get; set; }
    public string Name { get; set; }
    public decimal Price { get; set; }

    // Business logic thuộc về Model
    public decimal GetDiscountedPrice(decimal discountPercent)
    {
        if (discountPercent < 0 || discountPercent > 100)
            throw new ArgumentException("Invalid discount");

        return Price * (1 - discountPercent / 100);
    }

    public bool IsExpensive() => Price > 1000;
}

// MODEL - Repository pattern
public class ProductRepository
{
    public List<Product> GetAllProducts()
    {
        // Database logic tách riêng
        using var conn = new SqlConnection("Server=.;Database=Shop;");
        var products = new List<Product>();

        var cmd = new SqlCommand("SELECT Id, Name, Price FROM Product", conn);
        conn.Open();

        using var reader = cmd.ExecuteReader();
        while (reader.Read())
        {
            products.Add(new Product
            {
                Id = (int)reader["Id"],
                Name = reader["Name"].ToString(),
                Price = (decimal)reader["Price"]
            });
        }

        return products;
    }

    public decimal GetTotalValue()
    {
        return GetAllProducts().Sum(p => p.Price);
    }
}

// CONTROLLER - Điều phối
public class ProductController
{
    private readonly ProductRepository _repository;
    private readonly ProductView _view;

    public ProductController()
    {
        _repository = new ProductRepository();
        _view = new ProductView();
    }

    public void ShowAllProducts()
    {
        try
        {
            // Lấy data từ Model
            var products = _repository.GetAllProducts();
            var totalValue = _repository.GetTotalValue();

            // Gửi data cho View để hiển thị
            _view.DisplayProducts(products, totalValue);
        }
        catch (Exception ex)
        {
            _view.DisplayError($"Error: {ex.Message}");
        }
    }

    public void ShowDiscountedProducts(decimal discount)
    {
        var products = _repository.GetAllProducts();

        // Business logic xử lý discount
        var discountedProducts = products.Select(p => new
        {
            p.Name,
            OriginalPrice = p.Price,
            DiscountedPrice = p.GetDiscountedPrice(discount)
        }).ToList();

        _view.DisplayDiscountedProducts(discountedProducts, discount);
    }
}

// VIEW - Chỉ lo hiển thị
public class ProductView
{
    public void DisplayProducts(List<Product> products, decimal totalValue)
    {
        Console.WriteLine("🛍️  === PRODUCT CATALOG ===");
        Console.WriteLine();

        foreach (var product in products)
        {
            var priceIcon = product.IsExpensive() ? "💎" : "📦";
            Console.WriteLine($"{priceIcon} {product.Name}: ${product.Price:F2}");
        }

        Console.WriteLine();
        Console.WriteLine($"💰 Total Inventory Value: ${totalValue:F2}");
    }

    public void DisplayDiscountedProducts(dynamic products, decimal discount)
    {
        Console.WriteLine($"🎉 === {discount}% DISCOUNT SALE ===");

        foreach (var product in products)
        {
            Console.WriteLine($"📦 {product.Name}");
            Console.WriteLine($"   Original: ${product.OriginalPrice:F2}");
            Console.WriteLine($"   Sale: ${product.DiscountedPrice:F2}");
            Console.WriteLine();
        }
    }

    public void DisplayError(string message)
    {
        Console.WriteLine($"❌ {message}");
    }
}

// SỬ DỤNG - Clean & Simple
var controller = new ProductController();
controller.ShowAllProducts();
controller.ShowDiscountedProducts(20);
```

---

## 🎤 **CÁCH TRẢ LỜI KHI PHỎNG VẤN**

### 📋 **Template trả lời chuyên nghiệp**

#### **Q1: MVC là gì? Tại sao cần MVC?**
```
🎯 Trả lời gọn:
"MVC là mẫu kiến trúc tách ứng dụng thành 3 phần: Model (dữ liệu & logic),
View (UI), Controller (điều phối). Nó giúp phân tách trách nhiệm, code dễ
bảo trì, dễ mở rộng và dễ test. Nếu không tách, code sẽ nhanh chóng rối
và khó quản lý."
```

#### **Q2: Nếu viết tất cả trong View có được không?**
```
💡 Trả lời khiêm tốn:
"Về mặt kỹ thuật thì được, ứng dụng vẫn chạy. Nhưng khi dự án lớn lên,
code khó bảo trì, logic dính vào UI, khó test. MVC giúp tách biệt rõ ràng,
nên thường được áp dụng để đảm bảo chất lượng dài hạn."
```

#### **Q3: Vai trò cụ thể của từng thành phần?**
```
📝 Trả lời ngắn gọn:
- Model: Dữ liệu & business logic, không biết gì về UI
- View: Hiển thị UI, nhận input, không chứa business logic
- Controller: Nhận request, gọi Model xử lý, chọn View response
```

#### **Q4: Ưu điểm và nhược điểm của MVC?**
```
⚖️ Trả lời cân bằng:
Ưu điểm:
- Tách biệt trách nhiệm rõ ràng
- Dễ bảo trì và mở rộng
- Dễ test từng component
- Team có thể work song song

Nhược điểm:
- Với ứng dụng nhỏ có thể hơi "over-engineering"
- Cần tạo nhiều file/class hơn
- Learning curve đối với beginner
```

#### **Q5: So sánh MVC với MVVM/MVP?**
```
🤝 Trả lời khiêm tốn:
"MVC tập trung tách Model-View-Controller với Controller làm trung gian.
MVVM và MVP cũng tương tự nhưng thay đổi cách kết nối View và logic.
Em có kinh nghiệm nhiều nhất với MVC vì nó phổ biến trong web frameworks,
nhưng em sẵn sàng học các pattern khác khi cần thiết."
```

### 🚀 **Flow vàng khi trả lời**
1. **Định nghĩa ngắn** → MVC là gì
2. **Lý do sử dụng** → Tại sao cần tách
3. **Ví dụ cụ thể** → Code hoặc real-world example
4. **Ưu/nhược điểm** → Thể hiện hiểu sâu

---

## 🌐 **MVC TRONG THỰC TẾ**

### 🎯 **ASP.NET Core MVC Example**
```csharp
// MODEL
public class User
{
    public int Id { get; set; }
    public string Name { get; set; }
    public string Email { get; set; }

    [JsonIgnore]
    public bool IsActive => !string.IsNullOrEmpty(Email);
}

public class UserService
{
    public async Task<List<User>> GetUsersAsync()
    {
        // Database call hoặc API call
        return await _context.Users.ToListAsync();
    }

    public async Task<User> CreateUserAsync(User user)
    {
        // Validation & business logic
        if (string.IsNullOrEmpty(user.Email))
            throw new ArgumentException("Email is required");

        _context.Users.Add(user);
        await _context.SaveChangesAsync();
        return user;
    }
}

// CONTROLLER
[ApiController]
[Route("api/[controller]")]
public class UsersController : ControllerBase
{
    private readonly UserService _userService;

    public UsersController(UserService userService)
    {
        _userService = userService;
    }

    [HttpGet]
    public async Task<IActionResult> GetUsers()
    {
        try
        {
            var users = await _userService.GetUsersAsync();
            return Ok(users);
        }
        catch (Exception ex)
        {
            return BadRequest(ex.Message);
        }
    }

    [HttpPost]
    public async Task<IActionResult> CreateUser([FromBody] User user)
    {
        if (!ModelState.IsValid)
            return BadRequest(ModelState);

        try
        {
            var createdUser = await _userService.CreateUserAsync(user);
            return CreatedAtAction(nameof(GetUsers), new { id = createdUser.Id }, createdUser);
        }
        catch (ArgumentException ex)
        {
            return BadRequest(ex.Message);
        }
    }
}

// VIEW (React/Angular Frontend)
// UserList.tsx
const UserList = () => {
    const [users, setUsers] = useState<User[]>([]);

    useEffect(() => {
        fetchUsers();
    }, []);

    const fetchUsers = async () => {
        try {
            const response = await fetch('/api/users');
            const userData = await response.json();
            setUsers(userData);
        } catch (error) {
            console.error('Error fetching users:', error);
        }
    };

    return (
        <div className="user-list">
            <h2>👥 User Management</h2>
            {users.map(user => (
                <div key={user.id} className="user-card">
                    <span>{user.name}</span>
                    <span>{user.email}</span>
                    <span className={user.isActive ? 'active' : 'inactive'}>
                        {user.isActive ? '✅' : '❌'}
                    </span>
                </div>
            ))}
        </div>
    );
};
```

### 🏗️ **MVC trong các Framework phổ biến**
```
🌐 Web Frameworks:
├── ASP.NET Core MVC (C#)
├── Spring MVC (Java)
├── Django (Python)
├── Ruby on Rails (Ruby)
└── Express.js (Node.js)

📱 Mobile Frameworks:
├── Android (Activity = Controller, Layout = View, Model = Data)
├── iOS (UIViewController = Controller, UIView = View)
└── Flutter (Widget = View, Controller = BLoC/Provider)

🖥️ Desktop Applications:
├── WPF với MVVM (tương tự MVC)
├── WinForms với MVP pattern
└── Electron apps
```

---

## 🔄 **MVVM & MVP - "ANH EM SINH BA CỦA MVC"**

### 🤔 **Tại sao có MVVM và MVP?**
MVC tốt nhưng có một số hạn chế:
- **Tight coupling**: View và Controller dính chặt nhau
- **Testing khó**: Khó test Controller mà không có View
- **UI complexity**: Với UI phức tạp, Controller dễ bị "fat"

👉 MVVM và MVP sinh ra để giải quyết những vấn đề này!

---

## 🎭 **MVP - MODEL VIEW PRESENTER**

### 🏗️ **Cấu trúc MVP**
```
🏭 MODEL (Dữ liệu & Logic)
├── Giống MVC hoàn toàn
├── Business rules & validation
└── Không biết gì về UI

👁️ VIEW (Passive View)
├── Hiển thị dữ liệu
├── Nhận user input
├── KHÔNG xử lý logic
└── Chỉ thông báo Presenter về events

🎪 PRESENTER (Trung gian thông minh)
├── Nhận events từ View
├── Gọi Model để xử lý
├── Cập nhật View với kết quả
└── Chứa toàn bộ UI logic
```

## �️ **Kiến trúc MVP - Data Flow Diagram**

```
┌─────────────────────────────────────────────────────────────┐
│                    🎭 MVP ARCHITECTURE                      │
└─────────────────────────────────────────────────────────────┘

     👤 USER
       │
       │ (1) UI Events
       ▼
┌─────────────┐    (2) Events Only    ┌─────────────────┐
│   👁️ VIEW    │ ──────────────────→ │  🎪 PRESENTER    │
│ (Passive UI) │                     │  (UI Controller) │
│              │ ←────────────────── │                 │
│ - Show Data  │   (4) Update UI     │ - Handle Events │
│ - Raise Event│       Commands      │ - UI Logic      │
│ - NO Logic   │                     │ - Update View   │
└─────────────┘                     └─────────────────┘
                                               │
                                               │ (3) Business
                                               │     Operations
                                               ▼
                                     ┌─────────────────┐
                                     │   🏭 MODEL       │
                                     │  (Pure Logic)   │
                                     │                 │
                                     │ - Business Rules│
                                     │ - Data Access   │
                                     │ - Validation    │
                                     └─────────────────┘
```

### 🆚 **So sánh MVC vs MVP Flow:**
```
📊 MVC Flow:
User → View → Controller → Model → Controller → View

📊 MVP Flow:
User → View → Presenter → Model → Presenter → View

🔑 Khác biệt chính:
- MVC: View biết Controller, có thể có logic
- MVP: View không biết gì, hoàn toàn passive
- MVP: Presenter điều khiển 100% UI logic
```

### ✅ **MVP Code Example - WinForms**
```csharp
// MODEL - Giống MVC
public class User
{
    public int Id { get; set; }
    public string Name { get; set; }
    public string Email { get; set; }

    public bool IsValid() => !string.IsNullOrEmpty(Name) && !string.IsNullOrEmpty(Email);
}

public class UserRepository
{
    private List<User> _users = new List<User>();

    public List<User> GetAllUsers() => _users.ToList();

    public void AddUser(User user)
    {
        user.Id = _users.Count + 1;
        _users.Add(user);
    }

    public void DeleteUser(int id)
    {
        _users.RemoveAll(u => u.Id == id);
    }
}

// VIEW Interface - Contract giữa View và Presenter
public interface IUserView
{
    string UserName { get; set; }
    string UserEmail { get; set; }

    void ShowUsers(List<User> users);
    void ShowError(string message);
    void ShowSuccess(string message);
    void ClearForm();

    // Events
    event Action AddUserClicked;
    event Action<int> DeleteUserClicked;
    event Action LoadUsersRequested;
}

// PRESENTER - Trung tâm điều khiển
public class UserPresenter
{
    private readonly IUserView _view;
    private readonly UserRepository _repository;

    public UserPresenter(IUserView view)
    {
        _view = view;
        _repository = new UserRepository();

        // Đăng ký events từ View
        _view.AddUserClicked += OnAddUserClicked;
        _view.DeleteUserClicked += OnDeleteUserClicked;
        _view.LoadUsersRequested += OnLoadUsersRequested;
    }

    private void OnAddUserClicked()
    {
        try
        {
            var user = new User
            {
                Name = _view.UserName,
                Email = _view.UserEmail
            };

            if (!user.IsValid())
            {
                _view.ShowError("Vui lòng nhập đầy đủ thông tin!");
                return;
            }

            _repository.AddUser(user);
            _view.ShowSuccess("Thêm user thành công!");
            _view.ClearForm();
            LoadUsers();
        }
        catch (Exception ex)
        {
            _view.ShowError($"Lỗi: {ex.Message}");
        }
    }

    private void OnDeleteUserClicked(int userId)
    {
        try
        {
            _repository.DeleteUser(userId);
            _view.ShowSuccess("Xóa user thành công!");
            LoadUsers();
        }
        catch (Exception ex)
        {
            _view.ShowError($"Lỗi: {ex.Message}");
        }
    }

    private void OnLoadUsersRequested()
    {
        LoadUsers();
    }

    private void LoadUsers()
    {
        var users = _repository.GetAllUsers();
        _view.ShowUsers(users);
    }
}

// VIEW Implementation - WinForms Form
public partial class UserForm : Form, IUserView
{
    private UserPresenter _presenter;

    public UserForm()
    {
        InitializeComponent();
        _presenter = new UserPresenter(this);
    }

    // Properties cho Presenter access
    public string UserName
    {
        get => txtName.Text;
        set => txtName.Text = value;
    }

    public string UserEmail
    {
        get => txtEmail.Text;
        set => txtEmail.Text = value;
    }

    // Events
    public event Action AddUserClicked;
    public event Action<int> DeleteUserClicked;
    public event Action LoadUsersRequested;

    // UI Methods
    public void ShowUsers(List<User> users)
    {
        listBoxUsers.DataSource = users;
        listBoxUsers.DisplayMember = "Name";
    }

    public void ShowError(string message)
    {
        MessageBox.Show(message, "Lỗi", MessageBoxButtons.OK, MessageBoxIcon.Error);
    }

    public void ShowSuccess(string message)
    {
        MessageBox.Show(message, "Thành công", MessageBoxButtons.OK, MessageBoxIcon.Information);
    }

    public void ClearForm()
    {
        txtName.Text = "";
        txtEmail.Text = "";
    }

    // Event Handlers - Chỉ trigger events
    private void btnAdd_Click(object sender, EventArgs e)
    {
        AddUserClicked?.Invoke();
    }

    private void btnDelete_Click(object sender, EventArgs e)
    {
        if (listBoxUsers.SelectedItem is User selectedUser)
        {
            DeleteUserClicked?.Invoke(selectedUser.Id);
        }
    }

    private void UserForm_Load(object sender, EventArgs e)
    {
        LoadUsersRequested?.Invoke();
    }
}
```

---

## 🎨 **MVVM - MODEL VIEW VIEWMODEL**

### 🏗️ **Cấu trúc MVVM**
```
🏭 MODEL (Dữ liệu & Logic)
├── Giống MVC và MVP
├── Business rules & validation
└── Không biết gì về UI

👁️ VIEW (Declarative UI)
├── XAML, HTML, hoặc declarative UI
├── Data binding với ViewModel
├── Không có code-behind logic
└── Tự động sync với ViewModel

🎭 VIEWMODEL (Binding Logic)
├── Expose data cho View binding
├── Commands cho user actions
├── Notify View khi data thay đổi
└── Không biết gì về View cụ thể
```

## 🏗️ **Kiến trúc MVVM - Data Flow Diagram**

```
┌─────────────────────────────────────────────────────────────┐
│                   🎨 MVVM ARCHITECTURE                      │
└─────────────────────────────────────────────────────────────┘

     👤 USER
       │
       │ (1) UI Interactions
       ▼
┌─────────────┐                        ┌─────────────────┐
│   👁️ VIEW    │ ←──────────────────→  │  🎭 VIEWMODEL   │
│(Declarative)│    (2) Two-Way         │ (Binding Logic) │
│             │        Data Binding    │                 │
│- XAML/HTML  │ ◄═══════════════════► │ - Properties    │
│- Auto Sync  │    (3) Property        │ - Commands      │
│- Binding    │        Changed         │ - Notifications │
│- Commands   │        Events          │ - UI State      │
└─────────────┘                        └─────────────────┘
                                                │
                                                │ (4) Business
                                                │     Calls
                                                ▼
                                      ┌─────────────────┐
                                      │   🏭 MODEL       │
                                      │ (Business Core) │
                                      │                 │
                                      │ - Domain Logic  │
                                      │ - Data Services │
                                      │ - Validation    │
                                      └─────────────────┘
```

### 🔗 **Data Binding - Trái tim của MVVM:**
```
📡 Two-way Data Binding Flow:
┌─────────┐    PropertyChanged    ┌─────────────┐    Data Access    ┌─────────┐
│  VIEW   │ ←──────────────────── │ VIEWMODEL   │ ←───────────────→ │ MODEL   │
│         │                      │             │                   │         │
│TextBox  │ ────────────────────→ │ Properties  │ ───────────────→  │Services │
│Button   │    User Input         │ Commands    │   Update Data     │Database │
│List     │                      │ Collections │                   │ APIs    │
└─────────┘                      └─────────────┘                   └─────────┘

🔄 Automatic Sync:
- User nhập data → ViewModel property update → Model update
- Model thay đổi → ViewModel notify → View tự động re-render
- Commands execute → Business logic → UI updates automatically
```

### 🆚 **So sánh 3 Patterns Flow:**
```
📊 MVC:    User → View → Controller → Model → View
📊 MVP:    User → View → Presenter → Model → View
📊 MVVM:   User → View ←→ ViewModel → Model ←→ ViewModel

🔑 Key Differences:
┌─────────────┬────────────────┬────────────────┬────────────────┐
│   Pattern   │   Coupling     │  Data Updates  │  Best For      │
├─────────────┼────────────────┼────────────────┼────────────────┤
│    MVC      │ Medium         │ Manual         │ Web Apps       │
│    MVP      │ Loose          │ Manual         │ Desktop Apps   │
│   MVVM      │ Very Loose     │ Automatic      │ Rich UI Apps   │
└─────────────┴────────────────┴────────────────┴────────────────┘
```

### ✅ **MVVM Code Example - WPF**
```csharp
// MODEL - Giống MVP
public class User
{
    public int Id { get; set; }
    public string Name { get; set; }
    public string Email { get; set; }

    public bool IsValid() => !string.IsNullOrEmpty(Name) && !string.IsNullOrEmpty(Email);
}

public class UserService
{
    private List<User> _users = new List<User>();

    public List<User> GetAllUsers() => _users.ToList();

    public void AddUser(User user)
    {
        user.Id = _users.Count + 1;
        _users.Add(user);
    }
}

// VIEWMODEL - Binding logic với INotifyPropertyChanged
public class UserViewModel : INotifyPropertyChanged
{
    private readonly UserService _userService;
    private ObservableCollection<User> _users;
    private string _userName;
    private string _userEmail;
    private string _statusMessage;

    public UserViewModel()
    {
        _userService = new UserService();
        Users = new ObservableCollection<User>();

        // Commands
        AddUserCommand = new RelayCommand(AddUser, CanAddUser);
        LoadUsersCommand = new RelayCommand(LoadUsers);

        LoadUsers();
    }

    // Properties với PropertyChanged notification
    public ObservableCollection<User> Users
    {
        get => _users;
        set
        {
            _users = value;
            OnPropertyChanged();
        }
    }

    public string UserName
    {
        get => _userName;
        set
        {
            _userName = value;
            OnPropertyChanged();
            AddUserCommand.RaiseCanExecuteChanged(); // Refresh command state
        }
    }

    public string UserEmail
    {
        get => _userEmail;
        set
        {
            _userEmail = value;
            OnPropertyChanged();
            AddUserCommand.RaiseCanExecuteChanged();
        }
    }

    public string StatusMessage
    {
        get => _statusMessage;
        set
        {
            _statusMessage = value;
            OnPropertyChanged();
        }
    }

    // Commands for View binding
    public RelayCommand AddUserCommand { get; }
    public RelayCommand LoadUsersCommand { get; }

    private void AddUser()
    {
        try
        {
            var user = new User { Name = UserName, Email = UserEmail };

            if (!user.IsValid())
            {
                StatusMessage = "⚠️ Vui lòng nhập đầy đủ thông tin!";
                return;
            }

            _userService.AddUser(user);
            Users.Add(user); // ObservableCollection tự động notify View

            // Clear form
            UserName = "";
            UserEmail = "";
            StatusMessage = "✅ Thêm user thành công!";
        }
        catch (Exception ex)
        {
            StatusMessage = $"❌ Lỗi: {ex.Message}";
        }
    }

    private bool CanAddUser()
    {
        return !string.IsNullOrWhiteSpace(UserName) && !string.IsNullOrWhiteSpace(UserEmail);
    }

    private void LoadUsers()
    {
        var users = _userService.GetAllUsers();
        Users.Clear();
        foreach (var user in users)
        {
            Users.Add(user);
        }
        StatusMessage = "📋 Users loaded successfully!";
    }

    // INotifyPropertyChanged implementation
    public event PropertyChangedEventHandler PropertyChanged;

    protected virtual void OnPropertyChanged([CallerMemberName] string propertyName = null)
    {
        PropertyChanged?.Invoke(this, new PropertyChangedEventArgs(propertyName));
    }
}

// RelayCommand helper class
public class RelayCommand : ICommand
{
    private readonly Action _execute;
    private readonly Func<bool> _canExecute;

    public RelayCommand(Action execute, Func<bool> canExecute = null)
    {
        _execute = execute;
        _canExecute = canExecute;
    }

    public bool CanExecute(object parameter) => _canExecute?.Invoke() ?? true;

    public void Execute(object parameter) => _execute();

    public event EventHandler CanExecuteChanged;

    public void RaiseCanExecuteChanged()
    {
        CanExecuteChanged?.Invoke(this, EventArgs.Empty);
    }
}
```

**VIEW - XAML với Data Binding:**
```xml
<Window x:Class="UserApp.UserWindow"
        xmlns="http://schemas.microsoft.com/winfx/2006/xaml/presentation"
        xmlns:x="http://schemas.microsoft.com/winfx/2006/xaml">

    <Window.DataContext>
        <local:UserViewModel />
    </Window.DataContext>

    <Grid Margin="20">
        <Grid.RowDefinitions>
            <RowDefinition Height="Auto"/>
            <RowDefinition Height="Auto"/>
            <RowDefinition Height="*"/>
            <RowDefinition Height="Auto"/>
        </Grid.RowDefinitions>

        <!-- Input Form -->
        <StackPanel Grid.Row="0" Orientation="Vertical">
            <TextBox Text="{Binding UserName, UpdateSourceTrigger=PropertyChanged}"
                     PlaceholderText="👤 Tên người dùng" Margin="0,5"/>
            <TextBox Text="{Binding UserEmail, UpdateSourceTrigger=PropertyChanged}"
                     PlaceholderText="📧 Email" Margin="0,5"/>
            <Button Content="➕ Thêm User"
                    Command="{Binding AddUserCommand}"
                    Margin="0,10"/>
        </StackPanel>

        <!-- User List -->
        <ListBox Grid.Row="2"
                 ItemsSource="{Binding Users}"
                 DisplayMemberPath="Name"
                 Margin="0,20"/>

        <!-- Status -->
        <TextBlock Grid.Row="3"
                   Text="{Binding StatusMessage}"
                   FontWeight="Bold"
                   Margin="0,10"/>
    </Grid>
</Window>
```

---

## ⚖️ **SO SÁNH MVC vs MVP vs MVVM**

### 📊 **Bảng so sánh chi tiết**
```
┌─────────────────┬─────────────────┬─────────────────┬─────────────────┐
│     KHÍA CẠNH   │      MVC        │      MVP        │      MVVM       │
├─────────────────┼─────────────────┼─────────────────┼─────────────────┤
│ View-Logic      │ Controller biết │ Presenter điều  │ ViewModel bind  │
│ Communication   │ View            │ khiển hoàn toàn │ với View        │
├─────────────────┼─────────────────┼─────────────────┼─────────────────┤
│ View            │ Active View     │ Passive View    │ Declarative     │
│ Intelligence    │ (có logic)      │ (không logic)   │ (data binding)  │
├─────────────────┼─────────────────┼─────────────────┼─────────────────┤
│ Testing         │ Khó test        │ Dễ test         │ Rất dễ test     │
│                 │ Controller      │ Presenter       │ ViewModel       │
├─────────────────┼─────────────────┼─────────────────┼─────────────────┤
│ Data Binding    │ Manual          │ Manual          │ Automatic       │
├─────────────────┼─────────────────┼─────────────────┼─────────────────┤
│ Coupling        │ Tight           │ Loose           │ Very Loose      │
├─────────────────┼─────────────────┼─────────────────┼─────────────────┤
│ Best For        │ Web Apps        │ Desktop Apps    │ WPF, Xamarin    │
│                 │ Server-side     │ WinForms        │ Angular, Vue    │
└─────────────────┴─────────────────┴─────────────────┴─────────────────┘
```

### 🎯 **Khi nào dùng pattern nào?**

#### 🌐 **MVC - Cho Web Applications**
```
✅ Phù hợp khi:
- Làm web application (ASP.NET MVC, Spring MVC)
- Server-side rendering
- RESTful APIs
- Team quen với web development

❌ Không phù hợp:
- Desktop applications với UI phức tạp
- Real-time data binding requirements
- Heavy client-side interactions
```

#### 🖥️ **MVP - Cho Desktop Applications**
```
✅ Phù hợp khi:
- WinForms, Console applications
- Cần test UI logic mà không có UI thật
- View đơn giản, không cần data binding
- Team muốn control hoàn toàn UI flow

❌ Không phù hợp:
- Data binding intensive applications
- Real-time UI updates
- Complex UI với many-to-many relationships
```

#### 🎨 **MVVM - Cho Rich Client Applications**
```
✅ Phù hợp khi:
- WPF, UWP, Xamarin applications
- Angular, Vue.js (frontend)
- Cần two-way data binding
- Complex UI với real-time updates
- Rich user interactions

❌ Không phù hợp:
- Simple applications (overkill)
- Platforms không support data binding
- Performance critical applications
```

---

## 🎤 **PHỎNG VẤN: SO SÁNH CÁC PATTERNS**

### **Q6: Khác biệt chính giữa MVC, MVP và MVVM?**
```
🎯 Trả lời chuyên nghiệp:
"Ba pattern này đều tách biệt UI và business logic nhưng theo cách khác nhau:

- MVC: Controller làm trung gian, View có thể biết Controller
- MVP: Presenter điều khiển hoàn toàn, View hoàn toàn passive
- MVVM: ViewModel bind với View thông qua data binding, loose coupling nhất

Trong thực tế, tôi chọn pattern dựa trên platform:
- MVC cho web apps (ASP.NET MVC)
- MVP cho desktop apps (WinForms)
- MVVM cho rich client apps (WPF, Angular)"
```

### **Q7: MVVM có ưu điểm gì so với MVC?**
```
💡 Trả lời cân bằng:
"MVVM có data binding tự động, loose coupling hơn, và dễ test hơn.
Tuy nhiên MVC đơn giản hơn và phù hợp cho web applications.
MVVM shine nhất với rich client applications như WPF hoặc modern frontend frameworks."
```

### **Q8: Khi nào không nên dùng những patterns này?**
```
⚖️ Trả lời thực tế:
"Với applications rất đơn giản (vài screens, ít logic), việc áp dụng những
patterns này có thể là overkill. Đôi khi một class duy nhất handle UI và
logic là đủ. Quan trọng là biết khi nào cần scaling up architecture."
```

---

## 🏆 **TỔNG KẾT CHO PHỎNG VẤN**

### 📊 **Quick Reference - Ba Patterns So Sánh**
```
🏭 MODEL (Giống nhau ở cả 3 patterns)
├── Entities/Domain models
├── Business logic & validation
├── Data access (Repository pattern)
└── Services & utilities

👁️ VIEW
├── MVC: Active View (có logic, biết Controller)
├── MVP: Passive View (không logic, chỉ hiển thị)
└── MVVM: Declarative View (data binding)

🎮 CONTROLLER/PRESENTER/VIEWMODEL
├── MVC Controller: Nhận request, gọi Model, chọn View
├── MVP Presenter: Điều khiển hoàn toàn View, handle tất cả logic
└── MVVM ViewModel: Expose data/commands, notify View changes
```

### 💡 **Câu trả lời vàng cho phỏng vấn**
> **"MVC, MVP, MVVM đều giải quyết vấn đề separation of concerns nhưng theo cách khác nhau. MVC phù hợp web apps với Controller làm trung gian. MVP tốt cho desktop apps với Presenter kiểm soát hoàn toàn UI. MVVM shine với rich client apps nhờ data binding tự động. Tôi chọn pattern dựa trên platform và requirements cụ thể của dự án."**

### 🎯 **Framework & Platform Mapping**
```
🌐 MVC Frameworks:
├── ASP.NET Core MVC, Spring MVC
├── Django, Ruby on Rails
├── Express.js, Laravel
└── Angular (Component-based MVC)

🖥️ MVP Frameworks:
├── WinForms (C#)
├── Swing (Java)
├── Android Activities
└── iOS UIViewController (modified MVP)

🎨 MVVM Frameworks:
├── WPF, UWP, Xamarin (C#)
├── Angular, Vue.js (frontend)
├── Android Data Binding
└── iOS với RxSwift/Combine
```

### 🎯 **Lời khuyên quan trọng:**
- **Đừng over-engineer**: Với app đơn giản, MVC có thể thừa thãi
- **Hiểu bản chất**: Separation of concerns là key concept
- **Thực hành**: Build 1 project nhỏ để hiểu rõ flow
- **Biết khi nào dùng**: MVC phù hợp với medium-large applications

### ⚡ **Common Pitfalls & Solutions**
```
❌ Fat Controllers:
- Controller chứa quá nhiều business logic
✅ Solution: Move logic to Services/Models

❌ Anemic Models:
- Model chỉ có properties, không có behavior
✅ Solution: Thêm business methods vào Models

❌ View Logic:
- View chứa business logic
✅ Solution: Dùng ViewModels hoặc Helpers

❌ Tight Coupling:
- Controller phụ thuộc trực tiếp vào concrete classes
✅ Solution: Dependency Injection với Interfaces
```
