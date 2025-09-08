# 🔍 LINQ - Language Integrated Query
*"Google Search cho dữ liệu trong code - Tìm gì cũng có!"*

---

## 🌟 **LINQ LÀ GÌ VÀ TẠI SAO NÓ "THẦN THÁNH"?**

### 🧠 **Ví dụ đời thường**
```
🛒 Đi siêu thị:
- "Cho tôi tất cả sản phẩm táo có giá < 50k"
- "Sắp xếp theo giá từ thấp đến cao"
- "Chỉ lấy 5 cái đầu tiên"

🔍 LINQ:
var cheapApples = from product in supermarket
                  where product.Name.Contains("táo") && product.Price < 50000
                  orderby product.Price
                  select product
                  take 5;
```

### ⚡ **Cuộc sống trước và sau LINQ**

**😭 Thời kỳ đồ đá (Không có LINQ):**
```csharp
// Tìm học sinh có điểm > 8 - Code dài loằng ngoằng
List<Student> goodStudents = new List<Student>();
foreach (Student student in allStudents)
{
    if (student.Math > 8 && student.English > 8 && student.Physics > 8)
    {
        goodStudents.Add(student);
    }
}

// Sắp xếp theo điểm trung bình
goodStudents.Sort((s1, s2) => s2.AverageScore.CompareTo(s1.AverageScore));

// Lấy 10 học sinh đầu tiên
List<Student> top10 = new List<Student>();
for (int i = 0; i < Math.Min(10, goodStudents.Count); i++)
{
    top10.Add(goodStudents[i]);
}

// 😵‍💫 15+ dòng code, dễ bug, khó đọc
```

**🚀 Thời kỳ hiện đại (Có LINQ):**
```csharp
// Cùng logic, chỉ 1 dòng - Clean như tờ giấy trắng!
var top10GoodStudents = allStudents
    .Where(s => s.Math > 8 && s.English > 8 && s.Physics > 8)
    .OrderByDescending(s => s.AverageScore)
    .Take(10)
    .ToList();

// 🎉 1 câu lệnh, đọc như tiếng Anh bình thường!
```

---

## 🎯 **CÚ PHÁP LINQ - 2 PHONG CÁCH**

### 📝 **1. Query Syntax (Cú pháp SQL-like)**
```csharp
// Giống SQL, dễ đọc với developer biết Database
var result = from student in students
             where student.Age >= 18
             orderby student.Name
             select new { student.Name, student.Age };
```

### ⚡ **2. Method Syntax (Cú pháp Method Chain)**
```csharp
// Giống JavaScript, modern hơn
var result = students
    .Where(s => s.Age >= 18)
    .OrderBy(s => s.Name)
    .Select(s => new { s.Name, s.Age });
```

### 🤔 **Nên dùng phong cách nào?**
```
🏢 Query Syntax:
✅ Dễ đọc cho developer có background SQL
✅ Tốt cho query phức tạp với nhiều join
❌ Không support hết methods của LINQ

⚡ Method Syntax:
✅ Powerful hơn, support đầy đủ tính năng
✅ Dễ debug (F11 từng step)
✅ Trendy, modern style
❌ Hơi khó đọc với newbie

🎯 KẾT LUẬN: Học cả hai, dùng Method Syntax chủ yếu!
```

---

## 🛠️ **CÁC PHÉP THUẬT LINQ CƠ BẢN**

### 🔍 **1. WHERE - Lọc dữ liệu**
*"Tìm kim trong đống cỏ khô"*

```csharp
var students = new List<Student>
{
    new Student { Name = "An", Age = 20, Grade = 8.5, City = "Hà Nội" },
    new Student { Name = "Bình", Age = 19, Grade = 7.2, City = "TP.HCM" },
    new Student { Name = "Chi", Age = 21, Grade = 9.1, City = "Đà Nẵng" },
    new Student { Name = "Dung", Age = 18, Grade = 6.8, City = "Hà Nội" },
    new Student { Name = "Em", Age = 22, Grade = 8.9, City = "TP.HCM" }
};

// Tìm học sinh giỏi ở Hà Nội
var excellentHanoiStudents = students
    .Where(s => s.Grade >= 8.0 && s.City == "Hà Nội")
    .ToList();
// Kết quả: An (8.5, Hà Nội)

// Tìm học sinh tên bắt đầu bằng "A" hoặc "E"
var aOrEStudents = students
    .Where(s => s.Name.StartsWith("A") || s.Name.StartsWith("E"))
    .ToList();
// Kết quả: An, Em

// Where với logic phức tạp
var premiumStudents = students
    .Where(s => s.Age >= 20 && s.Grade > 8.0 && s.City.Contains("TP"))
    .ToList();
```

### 📊 **2. SELECT - Chọn dữ liệu**
*"Photographer chọn góc chụp đẹp nhất"*

```csharp
// Select đơn giản - lấy 1 field
var studentNames = students
    .Select(s => s.Name)
    .ToList();
// Kết quả: ["An", "Bình", "Chi", "Dung", "Em"]

// Select với transformation
var upperNames = students
    .Select(s => s.Name.ToUpper())
    .ToList();
// Kết quả: ["AN", "BÌNH", "CHI", "DUNG", "EM"]

// Select Anonymous Object (Đối tượng ẩn danh)
var studentSummary = students
    .Select(s => new 
    { 
        FullName = s.Name,
        IsAdult = s.Age >= 18,
        GradeLevel = s.Grade >= 8.5 ? "Xuất sắc" : 
                     s.Grade >= 7.0 ? "Khá" : "Trung bình",
        Location = s.City
    })
    .ToList();

// Select với index
var numberedStudents = students
    .Select((student, index) => new 
    { 
        Number = index + 1,
        Name = student.Name,
        Info = $"#{index + 1}: {student.Name} - {student.Grade}"
    })
    .ToList();
```

### 🔀 **3. ORDER BY - Sắp xếp**
*"Xếp hàng từ cao đến thấp, từ A đến Z"*

```csharp
// Sắp xếp đơn giản
var byGradeAsc = students
    .OrderBy(s => s.Grade)  // Từ thấp đến cao
    .ToList();

var byGradeDesc = students
    .OrderByDescending(s => s.Grade)  // Từ cao đến thấp
    .ToList();

// Sắp xếp đa cấp (Multiple levels)
var complexSort = students
    .OrderBy(s => s.City)           // Theo thành phố trước
    .ThenByDescending(s => s.Grade) // Rồi theo điểm (cao→thấp)
    .ThenBy(s => s.Age)             // Cuối cùng theo tuổi (thấp→cao)
    .ToList();

// Sắp xếp với custom logic
var customSort = students
    .OrderBy(s => s.City == "Hà Nội" ? 0 : 1)  // Hà Nội lên đầu
    .ThenByDescending(s => s.Grade)
    .ToList();
```

### 🎯 **4. TAKE/SKIP - Phân trang**
*"Lấy một ít, bỏ qua một ít"*

```csharp
// Top 3 học sinh giỏi nhất
var top3 = students
    .OrderByDescending(s => s.Grade)
    .Take(3)
    .ToList();

// Bỏ qua 2 học sinh đầu, lấy 2 học sinh tiếp theo
var page2 = students
    .OrderBy(s => s.Name)
    .Skip(2)
    .Take(2)
    .ToList();

// Phân trang professional
int pageSize = 2;
int pageNumber = 2; // Page thứ 2 (0-based thì là 1)

var pagedResult = students
    .OrderBy(s => s.Name)
    .Skip((pageNumber - 1) * pageSize)
    .Take(pageSize)
    .ToList();

// TakeWhile / SkipWhile - Conditional
var takeWhileGood = students
    .OrderByDescending(s => s.Grade)
    .TakeWhile(s => s.Grade >= 8.0)  // Lấy cho đến khi gặp điểm < 8
    .ToList();
```

---

## 🚀 **LINQ NÂNG CAO - NHỮNG PHÉP THUẬT MẠNH**

### 🔗 **1. JOIN - Kết nối dữ liệu**
*"Kết nối các mảnh ghép puzzle"*

```csharp
// Data setup
var students = new List<Student>
{
    new Student { Id = 1, Name = "An", ClassId = 101 },
    new Student { Id = 2, Name = "Bình", ClassId = 102 },
    new Student { Id = 3, Name = "Chi", ClassId = 101 },
    new Student { Id = 4, Name = "Dung", ClassId = 103 }
};

var classes = new List<Class>
{
    new Class { Id = 101, Name = "Toán A1", Teacher = "Thầy Hùng" },
    new Class { Id = 102, Name = "Lý A2", Teacher = "Cô Lan" },
    new Class { Id = 103, Name = "Hóa B1", Teacher = "Thầy Nam" }
};

// INNER JOIN - Chỉ lấy dữ liệu có match
var innerJoin = students
    .Join(classes,
          student => student.ClassId,     // Key từ students
          cls => cls.Id,                  // Key từ classes  
          (student, cls) => new           // Kết quả
          {
              StudentName = student.Name,
              ClassName = cls.Name,
              Teacher = cls.Teacher
          })
    .ToList();

// LEFT JOIN - Lấy tất cả students, kể cả không có class
var leftJoin = students
    .GroupJoin(classes,
               s => s.ClassId,
               c => c.Id,
               (student, classGroup) => new
               {
                   StudentName = student.Name,
                   ClassName = classGroup.FirstOrDefault()?.Name ?? "Chưa có lớp",
                   Teacher = classGroup.FirstOrDefault()?.Teacher ?? "Chưa có GV"
               })
    .ToList();

// Multiple JOIN
var subjects = new List<Subject>
{
    new Subject { ClassId = 101, Name = "Đại số", Credits = 3 },
    new Subject { ClassId = 102, Name = "Vật lý", Credits = 4 },
    new Subject { ClassId = 103, Name = "Hóa học", Credits = 3 }
};

var tripleJoin = students
    .Join(classes, s => s.ClassId, c => c.Id, (s, c) => new { Student = s, Class = c })
    .Join(subjects, sc => sc.Class.Id, sub => sub.ClassId, (sc, sub) => new
    {
        StudentName = sc.Student.Name,
        ClassName = sc.Class.Name,
        SubjectName = sub.Name,
        Credits = sub.Credits,
        Teacher = sc.Class.Teacher
    })
    .ToList();
```

### 📊 **2. GROUP BY - Nhóm dữ liệu**
*"Phân loại đồ chơi vào từng hộp"*

```csharp
var sales = new List<Sale>
{
    new Sale { Product = "iPhone", Category = "Phone", Amount = 1000, Month = "Jan" },
    new Sale { Product = "Samsung", Category = "Phone", Amount = 800, Month = "Jan" },
    new Sale { Product = "MacBook", Category = "Laptop", Amount = 2000, Month = "Jan" },
    new Sale { Product = "Dell", Category = "Laptop", Amount = 1200, Month = "Feb" },
    new Sale { Product = "iPhone", Category = "Phone", Amount = 1000, Month = "Feb" }
};

// Group đơn giản
var byCategory = sales
    .GroupBy(s => s.Category)
    .Select(g => new
    {
        Category = g.Key,
        TotalSales = g.Sum(s => s.Amount),
        Count = g.Count(),
        AverageAmount = g.Average(s => s.Amount)
    })
    .ToList();

// Group đa cấp
var byCategoryAndMonth = sales
    .GroupBy(s => new { s.Category, s.Month })
    .Select(g => new
    {
        Category = g.Key.Category,
        Month = g.Key.Month,
        TotalSales = g.Sum(s => s.Amount),
        Products = g.Select(s => s.Product).ToList()
    })
    .ToList();

// Group với điều kiện phức tạp
var salesAnalysis = sales
    .GroupBy(s => s.Category)
    .Where(g => g.Sum(s => s.Amount) > 1500)  // Chỉ lấy category có tổng > 1500
    .OrderByDescending(g => g.Sum(s => s.Amount))
    .Select(g => new
    {
        Category = g.Key,
        TotalRevenue = g.Sum(s => s.Amount),
        BestSellingProduct = g.OrderByDescending(s => s.Amount).First().Product,
        MonthlyBreakdown = g.GroupBy(s => s.Month)
                            .ToDictionary(mg => mg.Key, mg => mg.Sum(s => s.Amount))
    })
    .ToList();
```

### 🎲 **3. DISTINCT, UNION, INTERSECT, EXCEPT**
*"Loại bỏ trùng lặp và so sánh tập hợp"*

```csharp
var list1 = new[] { 1, 2, 3, 4, 5, 3, 2 };
var list2 = new[] { 4, 5, 6, 7, 8 };

// DISTINCT - Loại bỏ trùng lặp
var unique = list1.Distinct().ToList();
// Kết quả: [1, 2, 3, 4, 5]

// UNION - Hợp 2 tập hợp (tự động distinct)
var union = list1.Union(list2).ToList();
// Kết quả: [1, 2, 3, 4, 5, 6, 7, 8]

// INTERSECT - Giao 2 tập hợp
var intersect = list1.Intersect(list2).ToList();
// Kết quả: [4, 5]

// EXCEPT - Hiệu 2 tập hợp (list1 - list2)
var except = list1.Except(list2).ToList();
// Kết quả: [1, 2, 3]

// Ví dụ thực tế với objects
var currentUsers = new[] { "An", "Bình", "Chi" };
var newUsers = new[] { "Chi", "Dung", "Em" };

var allUsers = currentUsers.Union(newUsers);           // Tất cả user
var existingNewUsers = newUsers.Intersect(currentUsers); // User đã tồn tại
var reallyNewUsers = newUsers.Except(currentUsers);      // User thực sự mới
```

---

## 🎯 **LINQ VỚI CÁC DATA SOURCE KHÁC NHAU**

### 📁 **1. LINQ to Objects (Collection, Array, List)**
```csharp
// Với Array
int[] numbers = { 1, 2, 3, 4, 5, 6, 7, 8, 9, 10 };
var evenNumbers = numbers.Where(n => n % 2 == 0).ToArray();

// Với Dictionary
var phoneBook = new Dictionary<string, string>
{
    {"An", "0901234567"},
    {"Bình", "0907654321"},
    {"Chi", "0912345678"}
};

var longNumbers = phoneBook
    .Where(kvp => kvp.Value.Length > 10)
    .ToDictionary(kvp => kvp.Key, kvp => kvp.Value);
```

### 🗄️ **2. LINQ to SQL / Entity Framework**
```csharp
using var context = new SchoolContext();

// Query database như query collection
var topStudents = context.Students
    .Where(s => s.Grade > 8.5)
    .OrderByDescending(s => s.Grade)
    .Take(10)
    .Include(s => s.Class)  // Eager loading
    .ToListAsync();         // Async cho performance

// Complex query
var classStatistics = context.Classes
    .Select(c => new
    {
        ClassName = c.Name,
        StudentCount = c.Students.Count(),
        AverageGrade = c.Students.Average(s => s.Grade),
        TopStudent = c.Students.OrderByDescending(s => s.Grade).First().Name
    })
    .ToListAsync();
```

### 🌐 **3. LINQ to XML**
```csharp
var xml = XDocument.Parse(@"
<students>
    <student id='1' name='An' grade='8.5' />
    <student id='2' name='Bình' grade='7.2' />
    <student id='3' name='Chi' grade='9.1' />
</students>");

var goodStudents = xml.Descendants("student")
    .Where(x => double.Parse(x.Attribute("grade").Value) >= 8.0)
    .Select(x => new
    {
        Name = x.Attribute("name").Value,
        Grade = double.Parse(x.Attribute("grade").Value)
    })
    .ToList();
```

---

## 🔥 **LINQ TRICKS & PERFORMANCE TIPS**

### ⚡ **1. Deferred Execution (Lazy Loading)**
```csharp
var bigList = Enumerable.Range(1, 1000000);

// Query này CHƯA chạy! Chỉ định nghĩa thôi
var query = bigList
    .Where(x => x % 2 == 0)
    .Select(x => x * x);

Console.WriteLine("Query defined"); // In ngay lập tức

// Query mới thực sự chạy khi enumerate
foreach (var item in query.Take(5)) // Chỉ xử lý 5 items đầu
{
    Console.WriteLine(item); // 4, 16, 36, 64, 100
}
```

### 🎯 **2. ToList() vs AsEnumerable()**
```csharp
var data = GetBigData(); // 1 triệu records

// ❌ BAD - Load hết vào memory
var badQuery = data
    .Where(x => x.IsActive)
    .ToList()  // Tất cả vào RAM!
    .Where(x => x.CreatedDate > DateTime.Now.AddDays(-30))
    .Take(10);

// ✅ GOOD - Lazy evaluation
var goodQuery = data
    .Where(x => x.IsActive)
    .Where(x => x.CreatedDate > DateTime.Now.AddDays(-30))
    .Take(10)
    .ToList(); // Chỉ load 10 items
```

### 🚀 **3. Parallel LINQ (PLINQ)**
```csharp
var bigData = Enumerable.Range(1, 10000000);

// Sequential LINQ
var sequential = bigData
    .Where(x => IsPrime(x))
    .ToList();

// Parallel LINQ - Tận dụng multi-core
var parallel = bigData
    .AsParallel()
    .Where(x => IsPrime(x))
    .ToList();

// Với ordering
var orderedParallel = bigData
    .AsParallel()
    .AsOrdered()  // Giữ thứ tự
    .Where(x => IsPrime(x))
    .Take(100)
    .ToList();
```

---

## 🛠️ **DỰ ÁN THỰC HÀNH: HỆ THỐNG QUẢN LÝ BÁN HÀNG**

```csharp
// Models
public class Product
{
    public int Id { get; set; }
    public string Name { get; set; }
    public string Category { get; set; }
    public decimal Price { get; set; }
    public int Stock { get; set; }
    public DateTime ReleaseDate { get; set; }
    public string Manufacturer { get; set; }
}

public class Order
{
    public int Id { get; set; }
    public int CustomerId { get; set; }
    public DateTime OrderDate { get; set; }
    public List<OrderItem> Items { get; set; } = new();
    public string Status { get; set; }
}

public class OrderItem
{
    public int ProductId { get; set; }
    public int Quantity { get; set; }
    public decimal UnitPrice { get; set; }
}

public class Customer
{
    public int Id { get; set; }
    public string Name { get; set; }
    public string Email { get; set; }
    public string City { get; set; }
    public DateTime JoinDate { get; set; }
    public string Tier { get; set; } // Bronze, Silver, Gold, Platinum
}

// Data
var products = new List<Product>
{
    new Product { Id = 1, Name = "iPhone 15", Category = "Phone", Price = 25000000, Stock = 50, ReleaseDate = new DateTime(2023, 9, 15), Manufacturer = "Apple" },
    new Product { Id = 2, Name = "Samsung S24", Category = "Phone", Price = 22000000, Stock = 30, ReleaseDate = new DateTime(2024, 1, 20), Manufacturer = "Samsung" },
    new Product { Id = 3, Name = "MacBook Pro", Category = "Laptop", Price = 45000000, Stock = 20, ReleaseDate = new DateTime(2023, 10, 1), Manufacturer = "Apple" },
    new Product { Id = 4, Name = "Dell XPS", Category = "Laptop", Price = 35000000, Stock = 25, ReleaseDate = new DateTime(2023, 8, 15), Manufacturer = "Dell" }
};

// Complex LINQ Queries
public class SalesAnalytics
{
    // 1. Top sản phẩm bán chạy theo từng tháng
    public static var GetTopSellingProducts(List<Order> orders, List<Product> products, int month, int year)
    {
        return orders
            .Where(o => o.OrderDate.Month == month && o.OrderDate.Year == year)
            .SelectMany(o => o.Items)  // Flatten order items
            .GroupBy(item => item.ProductId)
            .Select(g => new
            {
                ProductId = g.Key,
                ProductName = products.First(p => p.Id == g.Key).Name,
                TotalQuantitySold = g.Sum(item => item.Quantity),
                TotalRevenue = g.Sum(item => item.Quantity * item.UnitPrice),
                OrderCount = g.Count()
            })
            .OrderByDescending(x => x.TotalQuantitySold)
            .Take(10)
            .ToList();
    }

    // 2. Phân tích khách hàng VIP
    public static var GetVIPCustomerAnalysis(List<Customer> customers, List<Order> orders)
    {
        return customers
            .GroupJoin(orders,
                      c => c.Id,
                      o => o.CustomerId,
                      (customer, customerOrders) => new
                      {
                          Customer = customer,
                          Orders = customerOrders.ToList()
                      })
            .Select(x => new
            {
                CustomerInfo = x.Customer,
                TotalOrders = x.Orders.Count(),
                TotalSpent = x.Orders.SelectMany(o => o.Items)
                                   .Sum(item => item.Quantity * item.UnitPrice),
                AverageOrderValue = x.Orders.Any() ? 
                    x.Orders.SelectMany(o => o.Items)
                           .Sum(item => item.Quantity * item.UnitPrice) / x.Orders.Count() : 0,
                LastOrderDate = x.Orders.Any() ? x.Orders.Max(o => o.OrderDate) : (DateTime?)null,
                FavoriteCategory = x.Orders
                    .SelectMany(o => o.Items)
                    .Join(products, item => item.ProductId, p => p.Id, (item, product) => product.Category)
                    .GroupBy(cat => cat)
                    .OrderByDescending(g => g.Count())
                    .FirstOrDefault()?.Key ?? "None"
            })
            .Where(x => x.TotalSpent > 50000000) // VIP threshold: 50M VND
            .OrderByDescending(x => x.TotalSpent)
            .ToList();
    }

    // 3. Dự báo xu hướng theo mùa
    public static var GetSeasonalTrends(List<Order> orders, List<Product> products)
    {
        return orders
            .GroupBy(o => new { Season = GetSeason(o.OrderDate), Year = o.OrderDate.Year })
            .Select(seasonGroup => new
            {
                Season = seasonGroup.Key.Season,
                Year = seasonGroup.Key.Year,
                TotalRevenue = seasonGroup.SelectMany(o => o.Items)
                                        .Sum(item => item.Quantity * item.UnitPrice),
                TopCategories = seasonGroup
                    .SelectMany(o => o.Items)
                    .Join(products, item => item.ProductId, p => p.Id, (item, product) => new { product.Category, Revenue = item.Quantity * item.UnitPrice })
                    .GroupBy(x => x.Category)
                    .OrderByDescending(g => g.Sum(x => x.Revenue))
                    .Take(3)
                    .Select(g => new { Category = g.Key, Revenue = g.Sum(x => x.Revenue) })
                    .ToList()
            })
            .OrderBy(x => x.Year)
            .ThenBy(x => x.Season)
            .ToList();
    }

    private static string GetSeason(DateTime date)
    {
        return date.Month switch
        {
            12 or 1 or 2 => "Winter",
            3 or 4 or 5 => "Spring", 
            6 or 7 or 8 => "Summer",
            _ => "Autumn"
        };
    }

    // 4. Inventory optimization
    public static var GetInventoryInsights(List<Product> products, List<Order> orders, int days = 30)
    {
        var recentOrders = orders.Where(o => o.OrderDate >= DateTime.Now.AddDays(-days));
        
        return products
            .GroupJoin(recentOrders.SelectMany(o => o.Items),
                      p => p.Id,
                      item => item.ProductId,
                      (product, items) => new
                      {
                          Product = product,
                          RecentSales = items.Sum(i => i.Quantity),
                          SalesVelocity = items.Sum(i => i.Quantity) / (double)days,
                          DaysUntilStockout = items.Any() ? product.Stock / (items.Sum(i => i.Quantity) / (double)days) : double.MaxValue
                      })
            .Select(x => new
            {
                ProductName = x.Product.Name,
                Category = x.Product.Category,
                CurrentStock = x.Product.Stock,
                RecentSales = x.RecentSales,
                DailyAverageSales = Math.Round(x.SalesVelocity, 2),
                DaysUntilStockout = Math.Round(x.DaysUntilStockout, 1),
                ReorderRecommendation = x.DaysUntilStockout < 30 ? "URGENT - Reorder now" :
                                      x.DaysUntilStockout < 60 ? "Consider reordering" : "Stock sufficient",
                StockStatus = x.Product.Stock == 0 ? "Out of stock" :
                            x.Product.Stock < 10 ? "Low stock" :
                            x.Product.Stock < 50 ? "Medium stock" : "High stock"
            })
            .OrderBy(x => x.DaysUntilStockout)
            .ToList();
    }
}
```

---

## 🎯 **CHECKLIST MASTER LINQ**

### 📝 **Beginner Level:**
```
□ Hiểu được Where, Select, OrderBy
□ Phân biệt được Query vs Method syntax
□ Sử dụng được Take, Skip cho phân trang
□ Làm việc với List, Array cơ bản
```

### 🚀 **Intermediate Level:**
```
□ Thành thạo GroupBy và Aggregate functions
□ Sử dụng Join để kết nối dữ liệu
□ Hiểu Deferred Execution
□ Optimize performance với ToList() đúng chỗ
□ Làm việc với Anonymous objects
```

### 🏆 **Advanced Level:**
```
□ Master PLINQ cho big data
□ Custom IEqualityComparer cho Distinct/Union
□ LINQ to SQL/EF với Include, ThenInclude
□ Nested queries và SelectMany
□ Performance tuning và memory management
```

---

## 🎯 **BÀI TẬP VỀ NHÀ**

### 🗓️ **Tuần 1: Cơ bản**
```csharp
// Cho list này, thực hiện các yêu cầu:
var employees = new List<Employee>
{
    new Employee { Name = "An", Department = "IT", Salary = 15000000, Age = 28 },
    new Employee { Name = "Bình", Department = "HR", Salary = 12000000, Age = 32 },
    new Employee { Name = "Chi", Department = "IT", Salary = 18000000, Age = 26 },
    new Employee { Name = "Dung", Department = "Finance", Salary = 14000000, Age = 35 }
};

// TODO:
// 1. Tìm nhân viên lương > 15M
// 2. Sắp xếp theo lương giảm dần
// 3. Lấy 2 nhân viên đầu tiên
// 4. Tính lương trung bình theo phòng ban
```

### 📊 **Tuần 2: Nâng cao**
```csharp
// TODO: Tạo report phức tạp
// 1. Top 3 phòng ban có lương cao nhất
// 2. Nhân viên trẻ nhất và già nhất mỗi phòng
// 3. Phòng ban nào có độ chênh lệch lương lớn nhất
// 4. Tạo bảng pivot: Department vs Age Range
```

### 🎮 **Tuần 3: Thử thách**
```csharp
// TODO: Xây dựng query engine đơn giản
// Cho phép user nhập: "salary > 15000000 AND department = IT"
// Parse thành LINQ expression và execute
```

---

## 💡 **TROUBLESHOOTING - LỖI THƯỜNG GẶP**

### ❌ **1. Multiple enumeration**
```csharp
// WRONG - Query chạy 2 lần!
var expensiveQuery = data.Where(x => ComplexCalculation(x));
var count = expensiveQuery.Count();     // Chạy lần 1
var list = expensiveQuery.ToList();     // Chạy lần 2

// RIGHT - Chỉ chạy 1 lần
var result = data.Where(x => ComplexCalculation(x)).ToList();
var count = result.Count;
```

### ❌ **2. Null Reference Exception**
```csharp
// WRONG - Crash khi có null
var result = customers.Where(c => c.Address.City == "Hà Nội");

// RIGHT - Null-safe
var result = customers.Where(c => c.Address?.City == "Hà Nội");
```

### ❌ **3. Memory leak với big data**
```csharp
// WRONG - Load hết vào RAM
var bigResult = hugeDataset.ToList().Where(x => x.IsActive);

// RIGHT - Lazy evaluation
var bigResult = hugeDataset.Where(x => x.IsActive);
```

---

## 🚀 **NEXT LEVEL: CUSTOM LINQ EXTENSIONS**

```csharp
public static class LinqExtensions
{
    // Extension method tự tạo
    public static IEnumerable<T> WhereNotNull<T>(this IEnumerable<T?> source) 
        where T : class
    {
        return source.Where(x => x != null)!;
    }
    
    // Paging extension
    public static IEnumerable<T> Page<T>(this IEnumerable<T> source, int page, int size)
    {
        return source.Skip((page - 1) * size).Take(size);
    }
    
    // Batch processing
    public static IEnumerable<IEnumerable<T>> Batch<T>(this IEnumerable<T> source, int batchSize)
    {
        var batch = new List<T>(batchSize);
        foreach (var item in source)
        {
            batch.Add(item);
            if (batch.Count == batchSize)
            {
                yield return batch;
                batch = new List<T>(batchSize);
            }
        }
        if (batch.Count > 0)
            yield return batch;
    }
    
    // Safe aggregation
    public static decimal SafeSum<T>(this IEnumerable<T> source, Func<T, decimal?> selector)
    {
        return source.Select(selector).Where(x => x.HasValue).Sum(x => x.Value);
    }
}

// Sử dụng:
var result = data
    .WhereNotNull()
    .Page(1, 10)
    .Batch(5)
    .SelectMany(batch => ProcessBatch(batch));
```

---

## 🎊 **KẾT LUẬN**

### 🌟 **LINQ là siêu ngôn ngữ vì:**
```
🎯 UNIVERSAL: Dùng được cho mọi data source
⚡ POWERFUL: Từ simple filter đến complex analytics
📚 READABLE: Code như đọc tiếng Anh
🚀 PERFORMANCE: Lazy evaluation + parallel processing
🔧 EXTENSIBLE: Tự tạo extensions theo nhu cầu
```

### 🏆 **Roadmap chinh phục LINQ:**
```
Tuần 1-2: Where, Select, OrderBy, Take/Skip
Tuần 3-4: GroupBy, Join, Aggregate functions
Tuần 5-6: Advanced scenarios, Performance tuning
Tuần 7-8: Custom extensions, PLINQ, Real projects
```

### 💪 **Thử thách bản thân:**
- **30 ngày:** Solve 1 LINQ problem mỗi ngày
- **Dự án thực tế:** Apply LINQ vào 1 project có sẵn
- **Teaching:** Dạy lại LINQ cho junior developer

---

*"LINQ không chỉ là tool để query data, mà là cách suy nghĩ functional programming!"* 🔥

**Tiếp theo:** Entity Framework - Phiên dịch viên C# ↔ SQL 🗄️