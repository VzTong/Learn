# MVP (Model – View – Presenter) - Hướng dẫn chi tiết

## 1. Khái niệm

* **MVP** là một mẫu kiến trúc (architecture pattern), được dùng nhiều trong **ứng dụng desktop, mobile (Android trước khi phổ biến MVVM), hoặc các app UI phức tạp**.
* Nó chia ứng dụng thành 3 phần:

  * **Model** → Quản lý dữ liệu & business logic.
  * **View** → UI, chỉ tập trung hiển thị, không chứa logic.
  * **Presenter** → "Người điều phối" giữa View và Model.

👉 **Điểm khác biệt chính:**

* Trong **MVC**, View có thể gọi Model trực tiếp.
* Trong **MVP**, View **không gọi Model**, mà phải đi qua **Presenter**.
* View trong MVP là **hoàn toàn passive** - chỉ hiển thị, không có logic.

## 2. Kiến trúc MVP

```
┌─────────────────────────────────────────────────────────────┐
│                   🎪 MVP ARCHITECTURE                       │
└─────────────────────────────────────────────────────────────┘

     👤 USER
       │
       │ (1) UI Events
       ▼
┌─────────────┐    (2) Events Only    ┌─────────────────┐
│   👁️ VIEW    │ ──────────────────→ │  🎪 PRESENTER    │
│ (Passive UI) │                     │  (UI Controller) │
│              │ ←────────────────── │                 │
│ - Show Data  │   (5) Update UI     │ - Handle Events │
│ - Raise Event│       Commands      │ - UI Logic      │
│ - NO Logic   │                     │ - Format Data   │
│ - Interface  │                     │ - Validation    │
└─────────────┘                     └─────────────────┘
                                               │
                                               │ (3) Get/Update
                                               │     Data
                                               ▼
                                     ┌─────────────────┐
                                     │   🏭 MODEL       │
                                     │  (Business Core)│
                                     │                 │
                                     │ - Domain Logic  │◄─────────
                                     │ - Data Access   │          │
                                     │ - Business Rules│          │
                                     │ - Validation    │          │
                                     └─────────────────┘          │
                                               │                  │
                                               │ (4) Return       │
                                               │     Data         │
                                               └──────────────────┘
```

### 📊 **Flow giải thích chi tiết:**
```
1️⃣ USER tương tác với VIEW (click button, input text...)
2️⃣ VIEW chỉ raise events cho PRESENTER (không xử lý logic)
3️⃣ PRESENTER nhận events, gọi MODEL để get/update data
4️⃣ MODEL thực hiện business logic, trả về data
5️⃣ PRESENTER format data và cập nhật VIEW thông qua interface methods
```

---

## 3. Khi nào dùng MVP?

* Khi muốn **tách biệt hoàn toàn View và Model**.
* Khi View (UI) phức tạp, cần test logic xử lý nhưng vẫn muốn UI gọn nhẹ.
* **Unit testing** quan trọng - có thể test Presenter mà không cần UI.
* Thường dùng trong **Android (trước khi MVVM + LiveData phổ biến)**, WinForms, hoặc các UI app lớn.
* Khi platform **không hỗ trợ data binding** mạnh mẽ.

---

## 4. Ví dụ thực tế

### 4.1. Android Example (Java/Kotlin Style)

#### Model

```java
// User Entity
public class User {
    private int id;
    private String name;
    private String email;
    private boolean isActive;

    public User(int id, String name, String email) {
        this.id = id;
        this.name = name;
        this.email = email;
        this.isActive = true;
    }

    // Getters and Setters
    public String getName() { return name; }
    public String getEmail() { return email; }
    public boolean isActive() { return isActive; }

    // Business logic
    public boolean isValidEmail() {
        return email != null && email.contains("@");
    }
}

// Repository/Service
public class UserRepository {
    private List<User> users = new ArrayList<>();

    public void addUser(User user) {
        if (user.isValidEmail()) {
            users.add(user);
        } else {
            throw new IllegalArgumentException("Invalid email format");
        }
    }

    public List<User> getAllUsers() {
        return new ArrayList<>(users);
    }

    public User getUserById(int id) {
        return users.stream()
                .filter(user -> user.getId() == id)
                .findFirst()
                .orElse(null);
    }

    public void deleteUser(int id) {
        users.removeIf(user -> user.getId() == id);
    }
}
```

#### View Interface (Contract)

```java
// Contract định nghĩa giao tiếp giữa View và Presenter
public interface UserContract {

    interface View {
        // Methods để Presenter cập nhật UI
        void showUsers(List<User> users);
        void showUserAdded(String message);
        void showError(String error);
        void clearForm();
        void showLoading();
        void hideLoading();

        // Getter methods cho Presenter lấy input
        String getUserName();
        String getUserEmail();
    }

    interface Presenter {
        // Methods để View gọi khi có events
        void loadUsers();
        void addUser();
        void deleteUser(int userId);
        void onDestroy();
    }
}
```

#### Presenter

```java
public class UserPresenter implements UserContract.Presenter {
    private UserContract.View view;
    private UserRepository repository;

    public UserPresenter(UserContract.View view) {
        this.view = view;
        this.repository = new UserRepository();
    }

    @Override
    public void loadUsers() {
        if (view != null) {
            view.showLoading();

            try {
                // Simulate async operation
                List<User> users = repository.getAllUsers();
                view.hideLoading();
                view.showUsers(users);
            } catch (Exception e) {
                view.hideLoading();
                view.showError("Failed to load users: " + e.getMessage());
            }
        }
    }

    @Override
    public void addUser() {
        if (view != null) {
            String name = view.getUserName();
            String email = view.getUserEmail();

            // Validation logic in Presenter
            if (name == null || name.trim().isEmpty()) {
                view.showError("Name cannot be empty");
                return;
            }

            if (email == null || email.trim().isEmpty()) {
                view.showError("Email cannot be empty");
                return;
            }

            try {
                User newUser = new User(System.currentTimeMillis(), name, email);
                repository.addUser(newUser);

                view.showUserAdded("User added successfully!");
                view.clearForm();
                loadUsers(); // Refresh list
            } catch (IllegalArgumentException e) {
                view.showError(e.getMessage());
            } catch (Exception e) {
                view.showError("Failed to add user: " + e.getMessage());
            }
        }
    }

    @Override
    public void deleteUser(int userId) {
        if (view != null) {
            try {
                repository.deleteUser(userId);
                view.showUserAdded("User deleted successfully!");
                loadUsers(); // Refresh list
            } catch (Exception e) {
                view.showError("Failed to delete user: " + e.getMessage());
            }
        }
    }

    @Override
    public void onDestroy() {
        view = null; // Prevent memory leaks
    }
}
```

#### View Implementation (Android Activity)

```java
public class MainActivity extends AppCompatActivity implements UserContract.View {
    private UserContract.Presenter presenter;

    private EditText etName;
    private EditText etEmail;
    private Button btnAdd;
    private RecyclerView recyclerView;
    private ProgressBar progressBar;
    private UserAdapter adapter;

    @Override
    protected void onCreate(Bundle savedInstanceState) {
        super.onCreate(savedInstanceState);
        setContentView(R.layout.activity_main);

        initViews();
        setupPresenter();
        setupListeners();
    }

    private void initViews() {
        etName = findViewById(R.id.etName);
        etEmail = findViewById(R.id.etEmail);
        btnAdd = findViewById(R.id.btnAdd);
        recyclerView = findViewById(R.id.recyclerView);
        progressBar = findViewById(R.id.progressBar);

        adapter = new UserAdapter(this::onDeleteClicked);
        recyclerView.setAdapter(adapter);
    }

    private void setupPresenter() {
        presenter = new UserPresenter(this);
        presenter.loadUsers(); // Load initial data
    }

    private void setupListeners() {
        btnAdd.setOnClickListener(v -> presenter.addUser());
    }

    // View Contract Implementation - Presenter calls these methods
    @Override
    public void showUsers(List<User> users) {
        adapter.updateUsers(users);
    }

    @Override
    public void showUserAdded(String message) {
        Toast.makeText(this, message, Toast.LENGTH_SHORT).show();
    }

    @Override
    public void showError(String error) {
        Toast.makeText(this, "Error: " + error, Toast.LENGTH_LONG).show();
    }

    @Override
    public void clearForm() {
        etName.setText("");
        etEmail.setText("");
    }

    @Override
    public void showLoading() {
        progressBar.setVisibility(View.VISIBLE);
    }

    @Override
    public void hideLoading() {
        progressBar.setVisibility(View.GONE);
    }

    // Getter methods for Presenter
    @Override
    public String getUserName() {
        return etName.getText().toString().trim();
    }

    @Override
    public String getUserEmail() {
        return etEmail.getText().toString().trim();
    }

    // Event handlers - just delegate to Presenter
    private void onDeleteClicked(int userId) {
        presenter.deleteUser(userId);
    }

    @Override
    protected void onDestroy() {
        super.onDestroy();
        presenter.onDestroy();
    }
}
```

### 4.2. WinForms Example (C#)

#### View Interface

```csharp
public interface IUserView
{
    string UserName { get; set; }
    string UserEmail { get; set; }

    void ShowUsers(List<User> users);
    void ShowMessage(string message);
    void ShowError(string error);
    void ClearForm();

    event Action AddUserRequested;
    event Action LoadUsersRequested;
    event Action<int> DeleteUserRequested;
}
```

#### Presenter

```csharp
public class UserPresenter
{
    private readonly IUserView _view;
    private readonly UserRepository _repository;

    public UserPresenter(IUserView view)
    {
        _view = view;
        _repository = new UserRepository();

        // Subscribe to View events
        _view.AddUserRequested += OnAddUserRequested;
        _view.LoadUsersRequested += OnLoadUsersRequested;
        _view.DeleteUserRequested += OnDeleteUserRequested;
    }

    private void OnAddUserRequested()
    {
        try
        {
            if (string.IsNullOrWhiteSpace(_view.UserName))
            {
                _view.ShowError("Name is required");
                return;
            }

            if (string.IsNullOrWhiteSpace(_view.UserEmail))
            {
                _view.ShowError("Email is required");
                return;
            }

            var user = new User(_view.UserName, _view.UserEmail);
            _repository.AddUser(user);

            _view.ShowMessage("User added successfully!");
            _view.ClearForm();
            LoadUsers();
        }
        catch (Exception ex)
        {
            _view.ShowError($"Error adding user: {ex.Message}");
        }
    }

    private void OnLoadUsersRequested()
    {
        LoadUsers();
    }

    private void OnDeleteUserRequested(int userId)
    {
        try
        {
            _repository.DeleteUser(userId);
            _view.ShowMessage("User deleted successfully!");
            LoadUsers();
        }
        catch (Exception ex)
        {
            _view.ShowError($"Error deleting user: {ex.Message}");
        }
    }

    private void LoadUsers()
    {
        try
        {
            var users = _repository.GetAllUsers();
            _view.ShowUsers(users);
        }
        catch (Exception ex)
        {
            _view.ShowError($"Error loading users: {ex.Message}");
        }
    }
}
```

#### View Implementation

```csharp
public partial class UserForm : Form, IUserView
{
    private UserPresenter _presenter;

    public UserForm()
    {
        InitializeComponent();
        _presenter = new UserPresenter(this);
    }

    // Properties for Presenter
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
    public event Action AddUserRequested;
    public event Action LoadUsersRequested;
    public event Action<int> DeleteUserRequested;

    // View methods called by Presenter
    public void ShowUsers(List<User> users)
    {
        listBoxUsers.DataSource = users.ToList();
        listBoxUsers.DisplayMember = "Name";
    }

    public void ShowMessage(string message)
    {
        MessageBox.Show(message, "Success", MessageBoxButtons.OK, MessageBoxIcon.Information);
    }

    public void ShowError(string error)
    {
        MessageBox.Show(error, "Error", MessageBoxButtons.OK, MessageBoxIcon.Error);
    }

    public void ClearForm()
    {
        txtName.Text = "";
        txtEmail.Text = "";
    }

    // Event handlers - just raise events for Presenter
    private void btnAdd_Click(object sender, EventArgs e)
    {
        AddUserRequested?.Invoke();
    }

    private void btnDelete_Click(object sender, EventArgs e)
    {
        if (listBoxUsers.SelectedItem is User selectedUser)
        {
            DeleteUserRequested?.Invoke(selectedUser.Id);
        }
    }

    private void UserForm_Load(object sender, EventArgs e)
    {
        LoadUsersRequested?.Invoke();
    }
}
```

---

## 5. Ưu & Nhược điểm

### ✅ Ưu điểm

* **Tách biệt hoàn toàn**: View không biết gì về Model, loose coupling cao.
* **Dễ unit test**: Có thể test Presenter độc lập mà không cần UI.
* **View gọn nhẹ**: View chỉ tập trung render, không chứa business logic.
* **Presenter reusable**: Có thể dùng cho nhiều View implementations khác nhau.
* **Clear responsibility**: Mỗi component có vai trò rõ ràng.
* **Maintainable**: Dễ maintain và mở rộng.

### ❌ Nhược điểm

* **Presenter có thể phình to**: Gánh quá nhiều UI logic, trở thành "God object".
* **Boilerplate code**: Nhiều interfaces, events, methods → code dài dòng.
* **Memory leaks**: Nếu không dispose events đúng cách.
* **Over-engineering**: Với ứng dụng đơn giản có thể thừa thãi.
* **Learning curve**: Phức tạp hơn MVC truyền thống.
* **Performance**: Thêm một layer abstraction.

---

## 6. So sánh MVP với MVC và MVVM

| **Tiêu chí** | **MVC** | **MVP** | **MVVM** |
|-------------|---------|---------|----------|
| **🎯 View Intelligence** | Active (có logic) | Passive (không logic) | Declarative (binding) |
| **🔄 View-Model Communication** | Direct access | Via Presenter only | Via ViewModel binding |
| **🧪 Testability** | Khó test Controller | Dễ test Presenter | Dễ test ViewModel |
| **📱 Platform** | Web, Server-side | Desktop, Mobile | Rich UI, Desktop, SPA |
| **⚡ Data Updates** | Manual refresh | Manual via Presenter | Automatic binding |
| **🔗 Coupling** | Medium | Low | Very Low |
| **🛠️ Setup Complexity** | Simple | Medium | Complex |
| **📊 UI Logic Location** | Controller + View | Presenter | ViewModel |
| **🔄 Data Binding** | No | No | Yes |
| **💾 State Management** | Stateless | Stateful in Presenter | Stateful in ViewModel |

---

## 7. MVP Variants

### 7.1. **Passive View MVP** (Recommended)
```
- View hoàn toàn passive, không có logic
- Presenter điều khiển mọi UI operations
- Dễ test nhất, coupling thấp nhất
```

### 7.2. **Supervising Controller MVP**
```
- View có thể binding trực tiếp với Model cho simple operations
- Presenter chỉ handle complex logic
- Hybrid approach giữa MVC và MVP
```

---

## 8. Best Practices cho MVP

### ✅ DOs:
```csharp
// 1. Use interfaces cho View contract
public interface IUserView {
    void ShowUsers(List<User> users);
    void ShowError(string error);
}

// 2. Handle memory leaks
public void OnDestroy() {
    _view = null; // Prevent memory leaks
}

// 3. Keep View dumb - chỉ UI operations
public void ShowUsers(List<User> users) {
    listBox.DataSource = users; // Chỉ hiển thị, không logic
}

// 4. Put validation in Presenter
private void OnAddUser() {
    if (string.IsNullOrEmpty(_view.UserName)) {
        _view.ShowError("Name required");
        return;
    }
    // Business logic here
}

// 5. Use dependency injection
public UserPresenter(IUserView view, IUserRepository repository) {
    _view = view;
    _repository = repository;
}
```

### ❌ DON'Ts:
```csharp
// ❌ Don't put business logic in View
private void btnAdd_Click(object sender, EventArgs e) {
    if (txtName.Text.Length < 3) { // Logic in View - BAD!
        MessageBox.Show("Name too short");
    }
}

// ❌ Don't let View access Model directly
private void LoadUsers() {
    var users = userRepository.GetUsers(); // View calling Model - BAD!
    listBox.DataSource = users;
}

// ❌ Don't make Presenter too fat
public class UserPresenter {
    // 500+ lines of code - God object anti-pattern!
}
```

---

## 9. MVP trong các Framework

### Desktop Applications:
* **WinForms** (C#) - Classic MVP implementation
* **Swing** (Java) - MVC/MVP hybrid
* **Tkinter** (Python) - với custom MVP pattern

### Mobile Applications:
* **Android** (pre-Architecture Components era)
* **iOS** với custom MVP implementation
* **Xamarin.Forms** (trước khi MVVM phổ biến)

### Web Frontend:
* **ASP.NET Web Forms** - Code-behind pattern tương tự MVP
* **GWT** (Google Web Toolkit) - MVP for complex web UIs
* **Angular.js** (version 1.x) - với Controller as Presenter

---

## 10. Unit Testing MVP

### Testing Presenter (Main benefit of MVP):

```csharp
[Test]
public void AddUser_WithValidData_ShouldAddUserAndShowMessage()
{
    // Arrange
    var mockView = new Mock<IUserView>();
    var mockRepository = new Mock<IUserRepository>();

    mockView.Setup(v => v.UserName).Returns("John Doe");
    mockView.Setup(v => v.UserEmail).Returns("john@email.com");

    var presenter = new UserPresenter(mockView.Object, mockRepository.Object);

    // Act
    presenter.AddUser();

    // Assert
    mockRepository.Verify(r => r.AddUser(It.IsAny<User>()), Times.Once);
    mockView.Verify(v => v.ShowMessage("User added successfully!"), Times.Once);
    mockView.Verify(v => v.ClearForm(), Times.Once);
}

[Test]
public void AddUser_WithEmptyName_ShouldShowError()
{
    // Arrange
    var mockView = new Mock<IUserView>();
    mockView.Setup(v => v.UserName).Returns("");
    mockView.Setup(v => v.UserEmail).Returns("john@email.com");

    var presenter = new UserPresenter(mockView.Object);

    // Act
    presenter.AddUser();

    // Assert
    mockView.Verify(v => v.ShowError("Name is required"), Times.Once);
}
```

---

## 11. Câu hỏi phỏng vấn & Cách trả lời

### Q1. MVP là gì?

**Trả lời ngắn gọn:**

> **"MVP là một mẫu kiến trúc tách ứng dụng thành 3 phần: Model (dữ liệu), View (UI passive), và Presenter (điều phối logic).**
> **View chỉ hiển thị UI và raise events, Presenter xử lý tất cả logic và cập nhật View.**
> **Điểm mạnh là tách biệt hoàn toàn View với Model, dễ test Presenter độc lập."**

---

### Q2. MVP khác gì với MVC?

**Trả lời chuyên nghiệp:**

> **"Trong MVC, View có thể truy cập Model trực tiếp, và Controller biết về View.**
> **Trong MVP, View hoàn toàn passive, chỉ giao tiếp với Presenter qua interfaces.**
> **MVP có coupling thấp hơn và dễ unit test hơn, nhưng phức tạp hơn trong implementation."**

---

### Q3. Khi nào dùng MVP thay vì MVC/MVVM?

**Trả lời khiêm tốn:**

> **"MVP phù hợp khi cần unit testing mạnh mẽ và platform không hỗ trợ data binding.**
> **Ví dụ Android trước khi có LiveData/ViewModel, hoặc WinForms applications.**
> **Hiện tại với modern frameworks hỗ trợ binding (WPF, Angular, Vue), MVVM thường được ưa chuộng hơn vì ít boilerplate code."**

---

### Q4. Presenter có thể bị "fat" không? Cách giải quyết?

**Trả lời technical:**

> **"Có, Presenter dễ bị phình to khi gánh quá nhiều UI logic (God object anti-pattern).**
> **Giải quyết bằng cách:**
> - **Tách Presenter thành nhiều Presenter nhỏ**
> - **Extract business logic vào Services**
> - **Dùng Composition pattern**
> - **Apply Single Responsibility Principle"**

---

### Q5. Làm sao test MVP effectively?

**Trả lời practical:**

> **"MVP dễ test vì Presenter không depend vào UI framework.**
> **Em sử dụng Mock objects cho IView interface và Repository.**
> **Test các scenarios: valid input, invalid input, error handling.**
> **Key là test Presenter logic mà không cần tạo UI thật."**

---

## 12. MVP vs Modern Patterns

### 🆚 **MVP vs MVVM:**
```
MVP (2000s era):
+ Dễ implement trên platforms không có binding
+ Explicit control flow
+ Dễ debug
- Nhiều boilerplate code
- Manual UI updates

MVVM (Modern era):
+ Automatic data binding
+ Less boilerplate
+ Reactive programming
- Cần framework support
- Learning curve với binding
```

### 🆚 **MVP vs Redux/State Management:**
```
MVP:
+ Simple state trong Presenter
+ Direct UI updates
+ Easy to understand

Redux/State Management:
+ Global state management
+ Time travel debugging
+ Predictable state changes
+ Complex setup
```

---

## 13. Tổng kết

* **MVC**: đơn giản, hay dùng cho **web applications** (request/response).
* **MVP**: tách biệt mạnh hơn, hợp với **desktop/mobile apps** cần unit testing.
* **MVVM**: hiện đại hơn, tận dụng **data binding**, hợp với rich UI applications.

### 🎯 **Khi nào chọn MVP:**
✅ **Platform không hỗ trợ data binding mạnh**
✅ **Unit testing là priority**
✅ **UI phức tạp nhưng không cần real-time binding**
✅ **Team quen với event-driven programming**

### ❌ **Khi nào KHÔNG chọn MVP:**
❌ **Platform có binding mạnh (WPF, Angular, React)**
❌ **Ứng dụng đơn giản (over-engineering)**
❌ **Cần real-time data synchronization**
❌ **Performance critical (thêm abstraction layer)**

**MVP = MVC + Passive View + Better Testability** 🎯
