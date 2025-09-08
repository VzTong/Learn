# MVVM (Model – View – ViewModel) - Hướng dẫn chi tiết

## 1. Khái niệm

* **MVVM** là một mẫu kiến trúc (architecture pattern), thường dùng trong **ứng dụng desktop (WPF, WinUI), mobile (Xamarin, MAUI), và web frontend (Angular, Vue, React với state management)**.
* Nó chia ứng dụng thành 3 phần:

  * **Model** → Dữ liệu & business logic (giống MVC).
  * **View** → Giao diện người dùng (UI).
  * **ViewModel** → "Cầu nối" giữa View và Model. ViewModel chứa logic hiển thị (presentation logic), binding dữ liệu 2 chiều (two-way data binding).

👉 Điểm khác biệt so với MVC: **Controller được thay bằng ViewModel**.
ViewModel không trực tiếp điều khiển View, mà View **bind** dữ liệu vào ViewModel.

## 2. Kiến trúc MVVM

```
┌─────────────┐    Data Binding    ┌─────────────┐
│    View     │ ←─────────────────→ │ ViewModel   │
│    (UI)     │                    │ (Logic UI)  │
└─────────────┘                    └─────────────┘
                                          │
                                          │ Update/Get Data
                                          ▼
                                   ┌─────────────┐
                                   │    Model    │
                                   │ (Data/API)  │
                                   └─────────────┘
```

---

## 3. Khi nào dùng MVVM?

* Khi ứng dụng có **UI phức tạp, nhiều state (trạng thái)**.
* Cần **data binding tự động** (View tự update khi Model thay đổi).
* Thường dùng trong **WPF, Xamarin, MAUI, Angular, Vue, React + Redux/Pinia/MobX**.
* Khi muốn tách biệt rõ UI với logic hiển thị, để **test logic mà không cần UI**.

---

## 4. Ví dụ thực tế

### 4.1. WPF – C#

#### Model

```csharp
public class User
{
    public int Id { get; set; }
    public string Name { get; set; }
    public string Email { get; set; }
    public bool IsActive { get; set; }
}

public class UserService
{
    public List<User> GetUsers()
    {
        // Call API or Database
        return new List<User>
        {
            new User { Id = 1, Name = "Alice", Email = "alice@email.com", IsActive = true }
        };
    }
}
```

#### ViewModel

```csharp
using System.ComponentModel;
using System.Collections.ObjectModel;
using System.Windows.Input;

public class UserViewModel : INotifyPropertyChanged
{
    private readonly UserService _userService;

    public UserViewModel()
    {
        _userService = new UserService();
        Users = new ObservableCollection<User>();
        LoadUsersCommand = new RelayCommand(LoadUsers);
        LoadUsers();
    }

    // Properties với Data Binding
    private string _name;
    public string Name
    {
        get => _name;
        set { _name = value; OnPropertyChanged(nameof(Name)); }
    }

    public ObservableCollection<User> Users { get; set; }

    // Commands
    public ICommand LoadUsersCommand { get; }
    public ICommand AddUserCommand => new RelayCommand(AddUser);

    // Methods
    private void LoadUsers()
    {
        var users = _userService.GetUsers();
        Users.Clear();
        foreach (var user in users)
            Users.Add(user);
    }

    private void AddUser()
    {
        if (!string.IsNullOrEmpty(Name))
        {
            Users.Add(new User { Name = Name, Email = $"{Name}@email.com" });
            Name = string.Empty; // Clear input
        }
    }

    // INotifyPropertyChanged Implementation
    public event PropertyChangedEventHandler PropertyChanged;
    protected void OnPropertyChanged(string propertyName) =>
        PropertyChanged?.Invoke(this, new PropertyChangedEventArgs(propertyName));
}
```

#### View (XAML)

```xml
<Window x:Class="MVVMExample.MainWindow"
        xmlns="http://schemas.microsoft.com/winfx/2006/xaml/presentation"
        xmlns:x="http://schemas.microsoft.com/winfx/2006/xaml">
    <Window.DataContext>
        <local:UserViewModel />
    </Window.DataContext>

    <Grid>
        <Grid.RowDefinitions>
            <RowDefinition Height="Auto" />
            <RowDefinition Height="*" />
        </Grid.RowDefinitions>

        <!-- Input Form -->
        <StackPanel Grid.Row="0" Orientation="Horizontal" Margin="10">
            <TextBox Text="{Binding Name, UpdateSourceTrigger=PropertyChanged}"
                     Width="200" PlaceholderText="Enter name..." />
            <Button Command="{Binding AddUserCommand}" Content="Add User" Margin="10,0" />
        </StackPanel>

        <!-- User List -->
        <ListBox Grid.Row="1" ItemsSource="{Binding Users}" Margin="10">
            <ListBox.ItemTemplate>
                <DataTemplate>
                    <StackPanel Orientation="Horizontal">
                        <TextBlock Text="{Binding Name}" FontWeight="Bold" />
                        <TextBlock Text="{Binding Email}" Margin="10,0" />
                    </StackPanel>
                </DataTemplate>
            </ListBox.ItemTemplate>
        </ListBox>
    </Grid>
</Window>
```

👉 Khi user nhập vào `TextBox`, property `Name` trong ViewModel sẽ update, và khi click button, item mới sẽ thêm vào `ListBox` tự động → **two-way binding**.

### 4.2. Angular – TypeScript

#### Model (Service)

```typescript
// user.model.ts
export interface User {
  id: number;
  name: string;
  email: string;
}

// user.service.ts
@Injectable({
  providedIn: 'root'
})
export class UserService {
  private users: User[] = [];

  getUsers(): Observable<User[]> {
    return of(this.users);
  }

  addUser(user: User): void {
    this.users.push(user);
  }
}
```

#### ViewModel (Component)

```typescript
// user.component.ts
@Component({
  selector: 'app-user',
  templateUrl: './user.component.html'
})
export class UserComponent implements OnInit {
  users: User[] = [];
  newUserName = '';
  newUserEmail = '';

  constructor(private userService: UserService) {}

  ngOnInit(): void {
    this.loadUsers();
  }

  loadUsers(): void {
    this.userService.getUsers().subscribe(users => {
      this.users = users;
    });
  }

  addUser(): void {
    if (this.newUserName && this.newUserEmail) {
      const user: User = {
        id: Date.now(),
        name: this.newUserName,
        email: this.newUserEmail
      };
      this.userService.addUser(user);
      this.loadUsers();
      this.clearForm();
    }
  }

  clearForm(): void {
    this.newUserName = '';
    this.newUserEmail = '';
  }
}
```

#### View (Template)

```html
<!-- user.component.html -->
<div class="user-management">
  <h2>User Management</h2>

  <!-- Form -->
  <div class="form-group">
    <input [(ngModel)]="newUserName" placeholder="Name" />
    <input [(ngModel)]="newUserEmail" placeholder="Email" />
    <button (click)="addUser()">Add User</button>
  </div>

  <!-- User List -->
  <ul class="user-list">
    <li *ngFor="let user of users">
      <strong>{{ user.name }}</strong> - {{ user.email }}
    </li>
  </ul>
</div>
```

---

## 5. Ưu & Nhược điểm

### ✅ Ưu điểm

* **Binding dữ liệu tự động** → ít code glue giữa UI và logic.
* **Dễ unit test ViewModel** mà không cần UI (testable).
* **Tách biệt rõ ràng** giữa UI (View) và logic hiển thị (ViewModel).
* **Reusable ViewModel** - có thể dùng cho nhiều View khác nhau.
* **Declarative UI** - View chỉ khai báo binding, không chứa logic.

### ❌ Nhược điểm

* **Hơi phức tạp** cho ứng dụng nhỏ (over-engineering).
* **Learning curve** - dev cần học data binding, commands.
* **Performance** - việc binding nhiều có thể ảnh hưởng hiệu năng.
* **Memory leaks** - nếu không dispose event handlers đúng cách.
* **Cần framework hỗ trợ** binding (WPF, Angular, Vue...).

---

## 6. So sánh MVC vs MVVM

| **Tiêu chí** | **MVC** | **MVVM** |
|-------------|---------|----------|
| **🎯 Use Case** | Web applications, API | Desktop, Mobile, SPA |
| **🔄 Data Flow** | Request → Controller → Model → View | View ↔ ViewModel ↔ Model |
| **🔗 View-Logic Coupling** | Controller điều khiển View | View bind trực tiếp với ViewModel |
| **📱 Platform** | Server-side (ASP.NET, Spring) | Client-side (WPF, Xamarin, Angular) |
| **⚡ Data Updates** | Manual (re-render page/partial) | Automatic (two-way binding) |
| **🧪 Testability** | Test Controller logic dễ | Test ViewModel logic dễ (không cần UI) |
| **📊 State Management** | Stateless (HTTP request-response) | Stateful (client-side state) |
| **🎨 UI Complexity** | Simple to moderate | Complex, interactive UI |
| **🔧 Setup Complexity** | Đơn giản, ít config | Phức tạp hơn (binding, commands) |
| **💾 Memory Usage** | Ít (server renders) | Nhiều hơn (client-side state) |
| **🌐 Real-time Updates** | Cần WebSocket/SignalR | Native support qua binding |
| **👥 Team Structure** | Frontend/Backend tách biệt | Full-stack hoặc client-focused |

---

## 7. Khi nào dùng MVC vs MVVM?

### 🎯 Dùng **MVC** khi:
* **Web applications** truyền thống
* **RESTful APIs**, microservices
* **Server-side rendering** (SSR)
* **Simple UI** không cần real-time updates
* **Team** có frontend/backend developers riêng biệt

### 🎯 Dùng **MVVM** khi:
* **Desktop applications** (WPF, WinUI)
* **Mobile apps** (Xamarin, MAUI, Flutter)
* **Single Page Applications** (Angular, Vue, React)
* **Rich interactive UI** với nhiều binding
* **Real-time updates** cần thiết
* **Client-side state management** phức tạp

---

## 8. MVVM Frameworks/Libraries

### Desktop:
* **WPF** (Windows Presentation Foundation)
* **WinUI 3** - Modern Windows apps
* **Avalonia** - Cross-platform .NET UI
* **Electron** với frameworks như Vue/Angular

### Mobile:
* **Xamarin.Forms** / **MAUI** - Cross-platform
* **Flutter** với Provider/BLoC pattern
* **React Native** với Redux/MobX

### Web Frontend:
* **Angular** - Built-in MVVM support
* **Vue.js** - Reactive data binding
* **React** + state management (Redux, MobX, Zustand)
* **Svelte** - Compile-time reactivity

---

## 9. Best Practices cho MVVM

### ✅ DOs:
```csharp
// 1. Implement INotifyPropertyChanged properly
public class BaseViewModel : INotifyPropertyChanged
{
    protected void SetProperty<T>(ref T backingField, T value, [CallerMemberName] string propertyName = null)
    {
        if (EqualityComparer<T>.Default.Equals(backingField, value))
            return;

        backingField = value;
        OnPropertyChanged(propertyName);
    }
}

// 2. Use Commands thay vì code-behind events
public ICommand SaveCommand => new RelayCommand(Save, CanSave);

// 3. Use ObservableCollection cho collections
public ObservableCollection<User> Users { get; set; }

// 4. Validation
public class UserViewModel : IDataErrorInfo
{
    public string this[string columnName]
    {
        get
        {
            if (columnName == nameof(Name) && string.IsNullOrEmpty(Name))
                return "Name is required";
            return null;
        }
    }
}
```

### ❌ DON'Ts:
* **Không reference View** trong ViewModel
* **Không put business logic** trong ViewModel (để trong Model/Service)
* **Không forget dispose** event subscriptions
* **Không bind** quá nhiều properties nếu không cần thiết

---

## 10. Câu hỏi phỏng vấn & Cách trả lời

### Q1. MVVM là gì?

**Trả lời gọn:**

> **"MVVM là một mẫu kiến trúc chia ứng dụng thành 3 phần: Model (dữ liệu), View (UI), và ViewModel (logic hiển thị, binding dữ liệu).**
> **Điểm mạnh là hỗ trợ data binding tự động, giúp tách biệt UI và logic, dễ test và dễ bảo trì."**

---

### Q2. MVVM khác gì với MVC?

**Trả lời ngắn gọn, chuyên nghiệp:**

> **"MVC có Controller điều phối giữa Model và View theo kiểu request-response.**
> **Còn MVVM thay Controller bằng ViewModel, View liên kết trực tiếp với ViewModel qua data binding.**
> **MVC thường dùng cho web, MVVM thích hợp cho desktop/mobile với UI phức tạp cần real-time updates."**

---

### Q3. Khi nào dùng MVVM?

**Trả lời khiêm tốn:**

> **"MVVM phù hợp với ứng dụng có UI phức tạp và cần binding dữ liệu, ví dụ như WPF, mobile app với Xamarin/MAUI, hoặc SPA với Angular/Vue.**
> **Nếu ứng dụng web đơn giản hoặc API backend, MVC sẽ phù hợp hơn vì đơn giản và ít overhead."**

---

### Q4. Ưu và nhược điểm của MVVM?

**Trả lời đúng trọng tâm:**

> **"Ưu điểm: Two-way data binding tự động, tách biệt UI và logic, dễ unit test ViewModel, reusable.**
> **Nhược điểm: Phức tạp cho ứng dụng nhỏ, learning curve cao hơn, binding nhiều có thể ảnh hưởng performance, cần framework hỗ trợ."**

---

### Q5. Data Binding trong MVVM hoạt động như thế nào?

**Trả lời technical:**

> **"Data binding dựa trên INotifyPropertyChanged interface. Khi property trong ViewModel thay đổi, nó trigger PropertyChanged event.**
> **View đăng ký nghe event này và tự động update UI. Ngược lại, khi user thay đổi UI, View update lại property trong ViewModel.**
> **Framework như WPF, Angular sử dụng reflection và change detection để implement binding này."**

---

## 11. Tổng kết

* **MVC**: tốt cho **web applications**, request/response rõ ràng, server-side rendering.
* **MVVM**: tốt cho **UI phức tạp**, nhiều state, cần binding dữ liệu (WPF, mobile, SPA).
* **Khi phỏng vấn**: trả lời **ngắn gọn – đúng trọng tâm – khiêm tốn**, không lan man ví dụ ngoài lề.
* **Key takeaway**: Chọn pattern phù hợp với platform và yêu cầu của dự án! 🎯

**MVVM = MVC + Data Binding + Client-side State Management** 🚀
