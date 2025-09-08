# 🗄️ Entity Framework Core - "Phiên Dịch Viên Siêu Đẳng"
*"Nói chuyện với Database bằng C# - Không cần biết SQL!"*

---

## 🌟 **TẠI SAO ENTITY FRAMEWORK LÀ "SIÊU PHẨM"?**

### 🧠 **Cuộc sống trước và sau EF**

**😭 Thời đại đồ đá (ADO.NET thuần):**
```csharp
// Nightmare của mọi developer - 50+ dòng code cho 1 query đơn giản
public List<Customer> GetCustomers()
{
    var customers = new List<Customer>();
    using var connection = new SqlConnection(connectionString);
    connection.Open();
    
    var command = new SqlCommand(@"
        SELECT c.Id, c.Name, c.Email, c.City, 
               o.Id as OrderId, o.OrderDate, o.Total
        FROM Customers c
        LEFT JOIN Orders o ON c.Id = o.CustomerId
        WHERE c.IsActive = 1", connection);
    
    using var reader = command.ExecuteReader();
    var customerDict = new Dictionary<int, Customer>();
    
    while (reader.Read())
    {
        var customerId = reader.GetInt32("Id");
        if (!customerDict.ContainsKey(customerId))
        {
            customerDict[customerId] = new Customer
            {
                Id = customerId,
                Name = reader.GetString("Name"),
                Email = reader.GetString("Email"),
                City = reader.GetString("City"),
                Orders = new List<Order>()
            };
        }
        
        if (!reader.IsDBNull("OrderId"))
        {
            customerDict[customerId].Orders.Add(new Order
            {
                Id = reader.GetInt32("OrderId"),
                OrderDate = reader.GetDateTime("OrderDate"),
                Total = reader.GetDecimal("Total")
            });
        }
    }
    
    return customerDict.Values.ToList();
}
// 😵‍💫 Phức tạp, dễ bug, khó maintain!
```

**🚀 Thời đại vàng son (Entity Framework):**
```csharp
// Magic! Chỉ 1 dòng code
public async Task<List<Customer>> GetCustomers()
{
    return await _context.Customers
        .Include(c => c.Orders)
        .Where(c => c.IsActive)
        .ToListAsync();
}
// 🎉 Clean, safe, maintainable!
```

### 🎭 **EF như một phiên dịch viên tài ba:**
```
👨‍💻 Developer: "Tôi muốn tất cả khách hàng ở Hà Nội có đơn hàng > 1 triệu"

🤖 EF: "OK, để tôi dịch sang SQL..."

💾 Database: 
SELECT c.* FROM Customers c 
INNER JOIN Orders o ON c.Id = o.CustomerId 
WHERE c.City = 'Hà Nội' AND o.Total > 1000000

🎯 Kết quả: List<Customer> với đầy đủ Orders
```

---

## 🏗️ **THIẾT LẬP EF CORE TỪNG BƯỚC**

### 📦 **1. Cài đặt Packages**
```bash
# Core package
dotnet add package Microsoft.EntityFrameworkCore

# SQL Server provider
dotnet add package Microsoft.EntityFrameworkCore.SqlServer

# Migration tools
dotnet add package Microsoft.EntityFrameworkCore.Tools

# Design package (cho migrations)
dotnet add package Microsoft.EntityFrameworkCore.Design
```

### 🎯 **2. Tạo Models (Entities)**
```csharp
// Customer.cs - Khách hàng
public class Customer
{
    public int Id { get; set; }                    // Primary Key (tự động nhận diện)
    public string Name { get; set; } = string.Empty;
    public string Email { get; set; } = string.Empty;
    public string? Phone { get; set; }             // Nullable
    public string City { get; set; } = string.Empty;
    public DateTime CreatedDate { get; set; } = DateTime.Now;
    public bool IsActive { get; set; } = true;
    
    // Navigation Properties - Mối quan hệ
    public List<Order> Orders { get; set; } = new();           // 1-nhiều
    public CustomerProfile? Profile { get; set; }              // 1-1
}

// Order.cs - Đơn hàng
public class Order
{
    public int Id { get; set; }
    public int CustomerId { get; set; }            // Foreign Key
    public DateTime OrderDate { get; set; } = DateTime.Now;
    public decimal Total { get; set; }
    public string Status { get; set; } = "Pending";
    
    // Navigation Properties
    public Customer Customer { get; set; } = null!;           // Thuộc về khách hàng nào
    public List<OrderItem> Items { get; set; } = new();       // Chứa sản phẩm gì
}

// OrderItem.cs - Chi tiết đơn hàng
public class OrderItem
{
    public int Id { get; set; }
    public int OrderId { get; set; }               // FK to Order
    public int ProductId { get; set; }             // FK to Product
    public int Quantity { get; set; }
    public decimal UnitPrice { get; set; }
    
    // Navigation Properties
    public Order Order { get; set; } = null!;
    public Product Product { get; set; } = null!;
}

// Product.cs - Sản phẩm
public class Product
{
    public int Id { get; set; }
    public string Name { get; set; } = string.Empty;
    public string Description { get; set; } = string.Empty;
    public decimal Price { get; set; }
    public int Stock { get; set; }
    public string Category { get; set; } = string.Empty;
    
    // Navigation Properties
    public List<OrderItem> OrderItems { get; set; } = new();
}

// CustomerProfile.cs - Hồ sơ khách hàng (1-1 relationship)
public class CustomerProfile
{
    public int Id { get; set; }
    public int CustomerId { get; set; }            // FK unique
    public DateTime DateOfBirth { get; set; }
    public string Gender { get; set; } = string.Empty;
    public string Address { get; set; } = string.Empty;
    public string Preferences { get; set; } = string.Empty;
    
    public Customer Customer { get; set; } = null!;
}
```

### 🎛️ **3. Tạo DbContext - "Trung tâm điều khiển"**
```csharp
// ShopDbContext.cs
public class ShopDbContext : DbContext
{
    public ShopDbContext(DbContextOptions<ShopDbContext> options) : base(options)
    {
    }
    
    // DbSet - Đại diện cho các bảng
    public DbSet<Customer> Customers { get; set; }
    public DbSet<Order> Orders { get; set; }
    public DbSet<OrderItem> OrderItems { get; set; }
    public DbSet<Product> Products { get; set; }
    public DbSet<CustomerProfile> CustomerProfiles { get; set; }
    
    // Fluent API - Cấu hình chi tiết
    protected override void OnModelCreating(ModelBuilder modelBuilder)
    {
        base.OnModelCreating(modelBuilder);
        
        // Cấu hình Customer
        modelBuilder.Entity<Customer>(entity =>
        {
            entity.HasKey(e => e.Id);
            entity.Property(e => e.Name).HasMaxLength(100).IsRequired();
            entity.Property(e => e.Email).HasMaxLength(150).IsRequired();
            entity.Property(e => e.Phone).HasMaxLength(20);
            entity.Property(e => e.City).HasMaxLength(50).IsRequired();
            
            // Index cho tìm kiếm nhanh
            entity.HasIndex(e => e.Email).IsUnique();
            entity.HasIndex(e => new { e.City, e.IsActive });
        });
        
        // Cấu hình Order
        modelBuilder.Entity<Order>(entity =>
        {
            entity.HasKey(e => e.Id);
            entity.Property(e => e.Total).HasColumnType("decimal(18,2)");
            entity.Property(e => e.Status).HasMaxLength(20).HasDefaultValue("Pending");
            
            // Relationship: Order belongs to Customer
            entity.HasOne(e => e.Customer)
                  .WithMany(c => c.Orders)
                  .HasForeignKey(e => e.CustomerId)
                  .OnDelete(DeleteBehavior.Cascade);  // Xóa customer → xóa orders
        });
        
        // Cấu hình OrderItem
        modelBuilder.Entity<OrderItem>(entity =>
        {
            entity.HasKey(e => e.Id);
            entity.Property(e => e.UnitPrice).HasColumnType("decimal(18,2)");
            
            // Relationship: OrderItem belongs to Order
            entity.HasOne(e => e.Order)
                  .WithMany(o => o.Items)
                  .HasForeignKey(e => e.OrderId)
                  .OnDelete(DeleteBehavior.Cascade);
            
            // Relationship: OrderItem references Product
            entity.HasOne(e => e.Product)
                  .WithMany(p => p.OrderItems)
                  .HasForeignKey(e => e.ProductId)
                  .OnDelete(DeleteBehavior.Restrict);  // Không cho xóa product đang có order
        });
        
        // Cấu hình Product
        modelBuilder.Entity<Product>(entity =>
        {
            entity.HasKey(e => e.Id);
            entity.Property(e => e.Name).HasMaxLength(200).IsRequired();
            entity.Property(e => e.Price).HasColumnType("decimal(18,2)");
            entity.Property(e => e.Category).HasMaxLength(50);
            
            entity.HasIndex(e => e.Category);
            entity.HasIndex(e => e.Name);
        });
        
        // Cấu hình CustomerProfile (1-1 relationship)
        modelBuilder.Entity<CustomerProfile>(entity =>
        {
            entity.HasKey(e => e.Id);
            entity.Property(e => e.Gender).HasMaxLength(10);
            entity.Property(e => e.Address).HasMaxLength(500);
            
            // One-to-One relationship
            entity.HasOne(e => e.Customer)
                  .WithOne(c => c.Profile)
                  .HasForeignKey<CustomerProfile>(e => e.CustomerId)
                  .OnDelete(DeleteBehavior.Cascade);
        });
        
        // Seed Data - Dữ liệu mẫu
        SeedData(modelBuilder);
    }
    
    private void SeedData(ModelBuilder modelBuilder)
    {
        // Seed Products
        modelBuilder.Entity<Product>().HasData(
            new Product { Id = 1, Name = "iPhone 15", Description = "Apple smartphone", Price = 25000000, Stock = 50, Category = "Phone" },
            new Product { Id = 2, Name = "Samsung S24", Description = "Samsung smartphone", Price = 22000000, Stock = 30, Category = "Phone" },
            new Product { Id = 3, Name = "MacBook Pro", Description = "Apple laptop", Price = 45000000, Stock = 20, Category = "Laptop" },
            new Product { Id = 4, Name = "Dell XPS", Description = "Dell laptop", Price = 35000000, Stock = 25, Category = "Laptop" }
        );
        
        // Seed Customers
        modelBuilder.Entity<Customer>().HasData(
            new Customer { Id = 1, Name = "Nguyễn Văn An", Email = "an@email.com", Phone = "0901234567", City = "Hà Nội", CreatedDate = DateTime.Now.AddMonths(-6) },
            new Customer { Id = 2, Name = "Trần Thị Bình", Email = "binh@email.com", Phone = "0907654321", City = "TP.HCM", CreatedDate = DateTime.Now.AddMonths(-3) },
            new Customer { Id = 3, Name = "Lê Văn Chi", Email = "chi@email.com", Phone = "0912345678", City = "Đà Nẵng", CreatedDate = DateTime.Now.AddMonths(-1) }
        );
    }
}
```

### ⚙️ **4. Đăng ký trong Program.cs**
```csharp
// Program.cs
var builder = WebApplication.CreateBuilder(args);

// Đăng ký DbContext
builder.Services.AddDbContext<ShopDbContext>(options =>
    options.UseSqlServer(builder.Configuration.GetConnectionString("DefaultConnection")));

// Services khác...
builder.Services.AddControllers();

var app = builder.Build();

// Configure pipeline...
app.MapControllers();
app.Run();
```

### 🔧 **5. Connection String trong appsettings.json**
```json
{
  "ConnectionStrings": {
    "DefaultConnection": "Server=(localdb)\\mssqllocaldb;Database=ShopDB;Trusted_Connection=true;MultipleActiveResultSets=true"
  },
  "Logging": {
    "LogLevel": {
      "Default": "Information",
      "Microsoft.EntityFrameworkCore": "Information"
    }
  }
}
```

---

## 🚀 **MIGRATIONS - "Kiến Trúc Sư Database"**

### 📐 **Tạo và chạy Migration**
```bash
# 1. Tạo migration đầu tiên
dotnet ef migrations add InitialCreate

# 2. Xem SQL sẽ được generate
dotnet ef migrations script

# 3. Apply migration vào database
dotnet ef database update

# 4. Thêm migration mới (sau khi sửa model)
dotnet ef migrations add AddCustomerProfile

# 5. Rollback migration
dotnet ef database update PreviousMigrationName

# 6. Remove migration chưa apply
dotnet ef migrations remove
```

### 🎯 **Migration thực tế**
```csharp
// 20241201120000_AddCustomerProfile.cs
public partial class AddCustomerProfile : Migration
{
    protected override void Up(MigrationBuilder migrationBuilder)
    {
        migrationBuilder.CreateTable(
            name: "CustomerProfiles",
            columns: table => new
            {
                Id = table.Column<int>(type: "int", nullable: false)
                    .Annotation("SqlServer:Identity", "1, 1"),
                CustomerId = table.Column<int>(type: "int", nullable: false),
                DateOfBirth = table.Column<DateTime>(type: "datetime2", nullable: false),
                Gender = table.Column<string>(type: "nvarchar(10)", maxLength: 10, nullable: false),
                Address = table.Column<string>(type: "nvarchar(500)", maxLength: 500, nullable: false),
                Preferences = table.Column<string>(type: "nvarchar(max)", nullable: false)
            },
            constraints: table =>
            {
                table.PrimaryKey("PK_CustomerProfiles", x => x.Id);
                table.ForeignKey(
                    name: "FK_CustomerProfiles_Customers_CustomerId",
                    column: x => x.CustomerId,
                    principalTable: "Customers",
                    principalColumn: "Id",
                    onDelete: ReferentialAction.Cascade);
            });
    }

    protected override void Down(MigrationBuilder migrationBuilder)
    {
        migrationBuilder.DropTable(name: "CustomerProfiles");
    }
}
```

---

## 💎 **EF CORE QUERIES - LINQ TO SQL MAGIC**

### 🔍 **1. Basic Queries**
```csharp
public class CustomerService
{
    private readonly ShopDbContext _context;
    
    public CustomerService(ShopDbContext context)
    {
        _context = context;
    }
    
    // Lấy tất cả customers
    public async Task<List<Customer>> GetAllCustomersAsync()
    {
        return await _context.Customers
            .Where(c => c.IsActive)
            .OrderBy(c => c.Name)
            .ToListAsync();
    }
    
    // Tìm customer theo ID
    public async Task<Customer?> GetCustomerByIdAsync(int id)
    {
        return await _context.Customers
            .Include(c => c.Orders)           // Load orders cùng lúc
            .Include(c => c.Profile)          // Load profile cùng lúc
            .FirstOrDefaultAsync(c => c.Id == id);
    }
    
    // Tìm customer theo email
    public async Task<Customer?> GetCustomerByEmailAsync(string email)
    {
        return await _context.Customers
            .FirstOrDefaultAsync(c => c.Email == email);
    }
    
    // Tìm customers theo thành phố
    public async Task<List<Customer>> GetCustomersByCityAsync(string city)
    {
        return await _context.Customers
            .Where(c => c.City == city && c.IsActive)
            .Include(c => c.Orders.Where(o => o.Status == "Completed"))  // Filtered include
            .ToListAsync();
    }
}
```

### 🔗 **2. Complex Queries với Join**
```csharp
public class OrderService
{
    private readonly ShopDbContext _context;
    
    public OrderService(ShopDbContext context)
    {
        _context = context;
    }
    
    // Customer với orders và order items
    public async Task<List<CustomerOrderSummary>> GetCustomerOrderSummaryAsync()
    {
        return await _context.Customers
            .Include(c => c.Orders)
                .ThenInclude(o => o.Items)      // Include nested relationship
                    .ThenInclude(oi => oi.Product)
            .Where(c => c.IsActive)
            .Select(c => new CustomerOrderSummary
            {
                CustomerName = c.Name,
                Email = c.Email,
                TotalOrders = c.Orders.Count(),
                TotalSpent = c.Orders.Sum(o => o.Total),
                LastOrderDate = c.Orders.Max(o => o.OrderDate),
                FavoriteProduct = c.Orders
                    .SelectMany(o => o.Items)
                    .GroupBy(oi => oi.Product.Name)
                    .OrderByDescending(g => g.Sum(oi => oi.Quantity))
                    .First().Key
            })
            .ToListAsync();
    }
    
    // Top selling products
    public async Task<List<ProductSalesReport>> GetTopSellingProductsAsync(int topCount = 10)
    {
        return await _context.OrderItems
            .Include(oi => oi.Product)
            .GroupBy(oi => new { oi.ProductId, oi.Product.Name, oi.Product.Category })
            .Select(g => new ProductSalesReport
            {
                ProductId = g.Key.ProductId,
                ProductName = g.Key.Name,
                Category = g.Key.Category,
                TotalQuantitySold = g.Sum(oi => oi.Quantity),
                TotalRevenue = g.Sum(oi => oi.Quantity * oi.UnitPrice),
                OrderCount = g.Count()
            })
            .OrderByDescending(p => p.TotalQuantitySold)
            .Take(topCount)
            .ToListAsync();
    }
    
    // Revenue by month
    public async Task<List<MonthlyRevenue>> GetMonthlyRevenueAsync(int year)
    {
        return await _context.Orders
            .Where(o => o.OrderDate.Year == year && o.Status == "Completed")
            .GroupBy(o => o.OrderDate.Month)
            .Select(g => new MonthlyRevenue
            {
                Month = g.Key,
                Year = year,
                TotalRevenue = g.Sum(o => o.Total),
                OrderCount = g.Count(),
                AverageOrderValue = g.Average(o => o.Total)
            })
            .OrderBy(mr => mr.Month)
            .ToListAsync();
    }
}
```

### 🎯 **3. Raw SQL khi cần thiết**
```csharp
public class AdvancedQueryService
{
    private readonly ShopDbContext _context;
    
    public AdvancedQueryService(ShopDbContext context)
    {
        _context = context;
    }
    
    // Raw SQL cho query phức tạp
    public async Task<List<CustomerAnalytics>> GetCustomerAnalyticsAsync()
    {
        return await _context.Database
            .SqlQueryRaw<CustomerAnalytics>(@"
                SELECT 
                    c.Id as CustomerId,
                    c.Name as CustomerName,
                    c.City,
                    COUNT(o.Id) as TotalOrders,
                    ISNULL(SUM(o.Total), 0) as TotalSpent,
                    AVG(o.Total) as AverageOrderValue,
                    MAX(o.OrderDate) as LastOrderDate,
                    DATEDIFF(day, MAX(o.OrderDate), GETDATE()) as DaysSinceLastOrder
                FROM Customers c
                LEFT JOIN Orders o ON c.Id = o.CustomerId AND o.Status = 'Completed'
                WHERE c.IsActive = 1
                GROUP BY c.Id, c.Name, c.City
                HAVING COUNT(o.Id) > 0 OR DATEDIFF(day, c.CreatedDate, GETDATE()) < 30
                ORDER BY TotalSpent DESC")
            .ToListAsync();
    }
    
    // Stored Procedure
    public async Task<List<Customer>> GetVIPCustomersAsync(decimal minSpent)
    {
        return await _context.Customers
            .FromSqlInterpolated($"EXEC GetVIPCustomers {minSpent}")
            .ToListAsync();
    }
    
    // Bulk operations với raw SQL
    public async Task<int> BulkUpdateCustomerTierAsync()
    {
        return await _context.Database.ExecuteSqlRawAsync(@"
            UPDATE Customers 
            SET Tier = CASE 
                WHEN (SELECT SUM(Total) FROM Orders WHERE CustomerId = Customers.Id) >= 100000000 THEN 'Platinum'
                WHEN (SELECT SUM(Total) FROM Orders WHERE CustomerId = Customers.Id) >= 50000000 THEN 'Gold'
                WHEN (SELECT SUM(Total) FROM Orders WHERE CustomerId = Customers.Id) >= 20000000 THEN 'Silver'
                ELSE 'Bronze'
            END
            WHERE IsActive = 1");
    }
}
```

---

## 🔥 **EF CORE PERFORMANCE OPTIMIZATION**

### ⚡ **1. Eager Loading vs Lazy Loading vs Explicit Loading**
```csharp
public class LoadingStrategiesService
{
    private readonly ShopDbContext _context;
    
    public LoadingStrategiesService(ShopDbContext context)
    {
        _context = context;
    }
    
    // ❌ N+1 Problem - Performance killer!
    public async Task<List<CustomerDto>> BadApproach()
    {
        var customers = await _context.Customers.ToListAsync();
        var result = new List<CustomerDto>();
        
        foreach (var customer in customers)  // N+1: 1 query + N queries
        {
            var orders = await _context.Orders
                .Where(o => o.CustomerId == customer.Id)
                .ToListAsync();  // Mỗi customer = 1 query riêng!
            
            result.Add(new CustomerDto
            {
                Name = customer.Name,
                OrderCount = orders.Count
            });
        }
        return result;
    }
    
    // ✅ Eager Loading - Load tất cả 1 lúc
    public async Task<List<CustomerDto>> EagerLoadingApproach()
    {
        return await _context.Customers
            .Include(c => c.Orders)           // Load orders cùng lúc
                .ThenInclude(o => o.Items)    // Load order items luôn
                    .ThenInclude(oi => oi.Product)  // Load products luôn
            .Select(c => new CustomerDto
            {
                Name = c.Name,
                OrderCount = c.Orders.Count,
                TotalSpent = c.Orders.Sum(o => o.Total)
            })
            .ToListAsync();  // Chỉ 1 query duy nhất!
    }
    
    // 🎯 Selective Loading - Chỉ load cái cần
    public async Task<List<CustomerOrderSummary>> SelectiveLoading()
    {
        return await _context.Customers
            .Include(c => c.Orders.Where(o => o.Status == "Completed"))  // Filtered include
            .Where(c => c.IsActive)
            .Select(c => new CustomerOrderSummary  // Projection - chỉ lấy field cần thiết
            {
                CustomerName = c.Name,
                CompletedOrderCount = c.Orders.Count(),
                LastOrderDate = c.Orders.Max(o => o.OrderDate)
            })
            .ToListAsync();
    }
    
    // 🚀 Split Queries cho data lớn
    public async Task<List<Customer>> SplitQueryApproach()
    {
        return await _context.Customers
            .AsSplitQuery()  // Tách thành nhiều query riêng biệt
            .Include(c => c.Orders)
            .Include(c => c.Profile)
            .ToListAsync();
    }
}
```

### 🎯 **2. Query Optimization Techniques**
```csharp
public class OptimizedQueryService
{
    private readonly ShopDbContext _context;
    
    public OptimizedQueryService(ShopDbContext context)
    {
        _context = context;
    }
    
    // AsNoTracking cho read-only queries
    public async Task<List<ProductDto>> GetProductsForDisplayAsync()
    {
        return await _context.Products
            .AsNoTracking()  // Không track changes - Nhanh hơn 30-50%
            .Where(p => p.Stock > 0)
            .Select(p => new ProductDto  // Projection - chỉ lấy cần thiết
            {
                Id = p.Id,
                Name = p.Name,
                Price = p.Price,
                Category = p.Category
            })
            .ToListAsync();
    }
    
    // Compiled Queries cho queries hay dùng
    private static readonly Func<ShopDbContext, string, IAsyncEnumerable<Customer>> 
        GetCustomersByCity = EF.CompileAsyncQuery(
            (ShopDbContext context, string city) =>
                context.Customers
                    .Where(c => c.City == city && c.IsActive)
                    .OrderBy(c => c.Name));
    
    public async Task<List<Customer>> GetCustomersByCityOptimizedAsync(string city)
    {
        return await GetCustomersByCity(_context, city).ToListAsync();
    }
    
    // Batch queries thay vì multiple round trips
    public async Task<CustomerDashboard> GetCustomerDashboardAsync(int customerId)
    {
        // Thay vì 4 queries riêng, gộp thành 1 query phức tạp
        var result = await _context.Customers
            .Where(c => c.Id == customerId)
            .Select(c => new CustomerDashboard
            {
                CustomerInfo = new CustomerInfo
                {
                    Name = c.Name,
                    Email = c.Email,
                    City = c.City
                },
                OrderStatistics = new OrderStats
                {
                    TotalOrders = c.Orders.Count(),
                    CompletedOrders = c.Orders.Count(o => o.Status == "Completed"),
                    TotalSpent = c.Orders.Where(o => o.Status == "Completed").Sum(o => o.Total),
                    LastOrderDate = c.Orders.Max(o => o.OrderDate)
                },
                RecentOrders = c.Orders
                    .OrderByDescending(o => o.OrderDate)
                    .Take(5)
                    .Select(o => new RecentOrderDto
                    {
                        Id = o.Id,
                        OrderDate = o.OrderDate,
                        Total = o.Total,
                        Status = o.Status
                    }).ToList(),
                FavoriteProducts = c.Orders
                    .SelectMany(o => o.Items)
                    .GroupBy(oi => new { oi.ProductId, oi.Product.Name })
                    .OrderByDescending(g => g.Sum(oi => oi.Quantity))
                    .Take(3)
                    .Select(g => new FavoriteProductDto
                    {
                        ProductName = g.Key.Name,
                        TotalQuantity = g.Sum(oi => oi.Quantity)
                    }).ToList()
            })
            .FirstOrDefaultAsync();
        
        return result ?? new CustomerDashboard();
    }
}
```

### 🗄️ **3. Database Indexing Strategy**
```csharp
// Trong OnModelCreating method
protected override void OnModelCreating(ModelBuilder modelBuilder)
{
    // Single column indexes
    modelBuilder.Entity<Customer>()
        .HasIndex(c => c.Email)
        .IsUnique()
        .HasDatabaseName("IX_Customer_Email");
    
    modelBuilder.Entity<Customer>()
        .HasIndex(c => c.City)
        .HasDatabaseName("IX_Customer_City");
    
    // Composite indexes cho queries phức tạp
    modelBuilder.Entity<Order>()
        .HasIndex(o => new { o.CustomerId, o.Status, o.OrderDate })
        .HasDatabaseName("IX_Order_Customer_Status_Date");
    
    // Filtered index
    modelBuilder.Entity<Customer>()
        .HasIndex(c => c.CreatedDate)
        .HasDatabaseName("IX_Customer_CreatedDate_Active")
        .HasFilter("[IsActive] = 1");
    
    // Include columns for covering index
    modelBuilder.Entity<Product>()
        .HasIndex(p => p.Category)
        .IncludeProperties(p => new { p.Name, p.Price })
        .HasDatabaseName("IX_Product_Category_Covering");
}
```

---

## 🛠️ **CRUD OPERATIONS - TẤT CẢ THAO TÁC CƠ BẢN**

### ➕ **1. CREATE - Thêm dữ liệu**
```csharp
public class CustomerRepository
{
    private readonly ShopDbContext _context;
    
    public CustomerRepository(ShopDbContext context)
    {
        _context = context;
    }
    
    // Thêm 1 customer đơn giản
    public async Task<Customer> CreateCustomerAsync(CreateCustomerDto dto)
    {
        var customer = new Customer
        {
            Name = dto.Name,
            Email = dto.Email,
            Phone = dto.Phone,
            City = dto.City,
            CreatedDate = DateTime.Now,
            IsActive = true
        };
        
        _context.Customers.Add(customer);
        await _context.SaveChangesAsync();
        
        return customer;
    }
    
    // Thêm customer với profile và orders
    public async Task<Customer> CreateCustomerWithDetailsAsync(CreateCustomerDetailDto dto)
    {
        using var transaction = await _context.Database.BeginTransactionAsync();
        
        try
        {
            // Tạo customer
            var customer = new Customer
            {
                Name = dto.Name,
                Email = dto.Email,
                Phone = dto.Phone,
                City = dto.City
            };
            
            // Tạo profile
            customer.Profile = new CustomerProfile
            {
                DateOfBirth = dto.DateOfBirth,
                Gender = dto.Gender,
                Address = dto.Address,
                Preferences = dto.Preferences
            };
            
            // Tạo order đầu tiên (nếu có)
            if (dto.InitialOrder != null)
            {
                var order = new Order
                {
                    OrderDate = DateTime.Now,
                    Status = "Pending",
                    Items = dto.InitialOrder.Items.Select(item => new OrderItem
                    {
                        ProductId = item.ProductId,
                        Quantity = item.Quantity,
                        UnitPrice = item.UnitPrice
                    }).ToList()
                };
                
                order.Total = order.Items.Sum(i => i.Quantity * i.UnitPrice);
                customer.Orders.Add(order);
            }
            
            _context.Customers.Add(customer);
            await _context.SaveChangesAsync();
            
            await transaction.CommitAsync();
            return customer;
        }
        catch
        {
            await transaction.RollbackAsync();
            throw;
        }
    }
    
    // Bulk insert cho performance
    public async Task<List<Customer>> CreateCustomersBulkAsync(List<CreateCustomerDto> dtos)
    {
        var customers = dtos.Select(dto => new Customer
        {
            Name = dto.Name,
            Email = dto.Email,
            Phone = dto.Phone,
            City = dto.City,
            CreatedDate = DateTime.Now,
            IsActive = true
        }).ToList();
        
        _context.Customers.AddRange(customers);  // Bulk add
        await _context.SaveChangesAsync();
        
        return customers;
    }
}
```

### 📖 **2. READ - Đọc dữ liệu (đã có ở trên)**

### ✏️ **3. UPDATE - Cập nhật dữ liệu**
```csharp
public class CustomerUpdateService
{
    private readonly ShopDbContext _context;
    
    public CustomerUpdateService(ShopDbContext context)
    {
        _context = context;
    }
    
    // Update tracked entity
    public async Task<Customer?> UpdateCustomerAsync(int id, UpdateCustomerDto dto)
    {
        var customer = await _context.Customers.FindAsync(id);
        if (customer == null) return null;
        
        // EF tự động track changes
        customer.Name = dto.Name;
        customer.Email = dto.Email;
        customer.Phone = dto.Phone;
        customer.City = dto.City;
        
        await _context.SaveChangesAsync();  // Chỉ update fields thay đổi
        return customer;
    }
    
    // Update với navigation properties
    public async Task<Customer?> UpdateCustomerWithProfileAsync(int id, UpdateCustomerProfileDto dto)
    {
        var customer = await _context.Customers
            .Include(c => c.Profile)
            .FirstOrDefaultAsync(c => c.Id == id);
        
        if (customer == null) return null;
        
        // Update customer info
        customer.Name = dto.Name;
        customer.Email = dto.Email;
        
        // Update or create profile
        if (customer.Profile == null)
        {
            customer.Profile = new CustomerProfile();
        }
        
        customer.Profile.DateOfBirth = dto.DateOfBirth;
        customer.Profile.Gender = dto.Gender;
        customer.Profile.Address = dto.Address;
        customer.Profile.Preferences = dto.Preferences;
        
        await _context.SaveChangesAsync();
        return customer;
    }
    
    // Bulk update với raw SQL
    public async Task<int> BulkUpdateCustomerCityAsync(string oldCity, string newCity)
    {
        return await _context.Database.ExecuteSqlInterpolatedAsync(
            $"UPDATE Customers SET City = {newCity} WHERE City = {oldCity} AND IsActive = 1");
    }
    
    // Update với concurrency check
    public async Task<bool> UpdateCustomerConcurrentSafeAsync(int id, UpdateCustomerDto dto, byte[] rowVersion)
    {
        var customer = await _context.Customers.FindAsync(id);
        if (customer == null) return false;
        
        // Set original row version for concurrency check
        _context.Entry(customer).Property("RowVersion").OriginalValue = rowVersion;
        
        customer.Name = dto.Name;
        customer.Email = dto.Email;
        customer.Phone = dto.Phone;
        customer.City = dto.City;
        
        try
        {
            await _context.SaveChangesAsync();
            return true;
        }
        catch (DbUpdateConcurrencyException)
        {
            // Someone else modified the record
            return false;
        }
    }
}
```

### 🗑️ **4. DELETE - Xóa dữ liệu**
```csharp
public class CustomerDeleteService  
{
    private readonly ShopDbContext _context;
    
    public CustomerDeleteService(ShopDbContext context)
    {
        _context = context;
    }
    
    // Soft delete - Đánh dấu xóa
    public async Task<bool> SoftDeleteCustomerAsync(int id)
    {
        var customer = await _context.Customers.FindAsync(id);
        if (customer == null) return false;
        
        customer.IsActive = false;  // Soft delete
        await _context.SaveChangesAsync();
        return true;
    }
    
    // Hard delete - Xóa thật
    public async Task<bool> HardDeleteCustomerAsync(int id)
    {
        var customer = await _context.Customers
            .Include(c => c.Orders)
                .ThenInclude(o => o.Items)
            .Include(c => c.Profile)
            .FirstOrDefaultAsync(c => c.Id == id);
        
        if (customer == null) return false;
        
        // Xóa theo thứ tự (vì foreign key constraints)
        foreach (var order in customer.Orders)
        {
            _context.OrderItems.RemoveRange(order.Items);
        }
        _context.Orders.RemoveRange(customer.Orders);
        
        if (customer.Profile != null)
        {
            _context.CustomerProfiles.Remove(customer.Profile);
        }
        
        _context.Customers.Remove(customer);
        await _context.SaveChangesAsync();
        return true;
    }
    
    // Bulk delete
    public async Task<int> BulkDeleteInactiveCustomersAsync()
    {
        return await _context.Database.ExecuteSqlRawAsync(
            "DELETE FROM Customers WHERE IsActive = 0 AND CreatedDate < DATEADD(year, -2, GETDATE())");
    }
    
    // Cascade delete với transaction
    public async Task<bool> DeleteCustomerCascadeAsync(int id)
    {
        using var transaction = await _context.Database.BeginTransactionAsync();
        
        try
        {
            // Delete order items first
            await _context.Database.ExecuteSqlInterpolatedAsync(
                $@"DELETE oi FROM OrderItems oi 
                   INNER JOIN Orders o ON oi.OrderId = o.Id 
                   WHERE o.CustomerId = {id}");
            
            // Delete orders
            await _context.Database.ExecuteSqlInterpolatedAsync(
                $"DELETE FROM Orders WHERE CustomerId = {id}");
            
            // Delete profile
            await _context.Database.ExecuteSqlInterpolatedAsync(
                $"DELETE FROM CustomerProfiles WHERE CustomerId = {id}");
            
            // Delete customer
            await _context.Database.ExecuteSqlInterpolatedAsync(
                $"DELETE FROM Customers WHERE Id = {id}");
            
            await transaction.CommitAsync();
            return true;
        }
        catch
        {
            await transaction.RollbackAsync();
            return false;
        }
    }
}
```

---

## 🎯 **ADVANCED EF PATTERNS**

### 🏭 **1. Repository Pattern**
```csharp
// Generic Repository Interface
public interface IRepository<T> where T : class
{
    Task<T?> GetByIdAsync(int id);
    Task<IEnumerable<T>> GetAllAsync();
    Task<IEnumerable<T>> FindAsync(Expression<Func<T, bool>> predicate);
    Task<T> AddAsync(T entity);
    Task<IEnumerable<T>> AddRangeAsync(IEnumerable<T> entities);
    Task UpdateAsync(T entity);
    Task DeleteAsync(T entity);
    Task DeleteRangeAsync(IEnumerable<T> entities);
}

// Generic Repository Implementation
public class Repository<T> : IRepository<T> where T : class
{
    protected readonly ShopDbContext _context;
    protected readonly DbSet<T> _dbSet;
    
    public Repository(ShopDbContext context)
    {
        _context = context;
        _dbSet = context.Set<T>();
    }
    
    public async Task<T?> GetByIdAsync(int id)
    {
        return await _dbSet.FindAsync(id);
    }
    
    public async Task<IEnumerable<T>> GetAllAsync()
    {
        return await _dbSet.ToListAsync();
    }
    
    public async Task<IEnumerable<T>> FindAsync(Expression<Func<T, bool>> predicate)
    {
        return await _dbSet.Where(predicate).ToListAsync();
    }
    
    public async Task<T> AddAsync(T entity)
    {
        await _dbSet.AddAsync(entity);
        return entity;
    }
    
    public async Task<IEnumerable<T>> AddRangeAsync(IEnumerable<T> entities)
    {
        await _dbSet.AddRangeAsync(entities);
        return entities;
    }
    
    public Task UpdateAsync(T entity)
    {
        _dbSet.Update(entity);
        return Task.CompletedTask;
    }
    
    public Task DeleteAsync(T entity)
    {
        _dbSet.Remove(entity);
        return Task.CompletedTask;
    }
    
    public Task DeleteRangeAsync(IEnumerable<T> entities)
    {
        _dbSet.RemoveRange(entities);
        return Task.CompletedTask;
    }
}

// Specific Repository với business logic
public interface ICustomerRepository : IRepository<Customer>
{
    Task<Customer?> GetByEmailAsync(string email);
    Task<IEnumerable<Customer>> GetVIPCustomersAsync();
    Task<IEnumerable<Customer>> GetCustomersByCityAsync(string city);
    Task<CustomerStatistics> GetCustomerStatisticsAsync(int customerId);
}

public class CustomerRepository : Repository<Customer>, ICustomerRepository
{
    public CustomerRepository(ShopDbContext context) : base(context)
    {
    }
    
    public async Task<Customer?> GetByEmailAsync(string email)
    {
        return await _dbSet
            .Include(c => c.Profile)
            .Include(c => c.Orders)
            .FirstOrDefaultAsync(c => c.Email == email);
    }
    
    public async Task<IEnumerable<Customer>> GetVIPCustomersAsync()
    {
        return await _dbSet
            .Include(c => c.Orders)
            .Where(c => c.Orders.Sum(o => o.Total) >= 50000000)
            .ToListAsync();
    }
    
    public async Task<IEnumerable<Customer>> GetCustomersByCityAsync(string city)
    {
        return await _dbSet
            .Where(c => c.City == city && c.IsActive)
            .OrderBy(c => c.Name)
            .ToListAsync();
    }
    
    public async Task<CustomerStatistics> GetCustomerStatisticsAsync(int customerId)
    {
        return await _dbSet
            .Where(c => c.Id == customerId)
            .Select(c => new CustomerStatistics
            {
                TotalOrders = c.Orders.Count(),
                TotalSpent = c.Orders.Sum(o => o.Total),
                AverageOrderValue = c.Orders.Average(o => o.Total),
                LastOrderDate = c.Orders.Max(o => o.OrderDate)
            })
            .FirstOrDefaultAsync() ?? new CustomerStatistics();
    }
}
```

### 🏗️ **2. Unit of Work Pattern**
```csharp
public interface IUnitOfWork : IDisposable
{
    ICustomerRepository Customers { get; }
    IOrderRepository Orders { get; }
    IProductRepository Products { get; }
    
    Task<int> SaveChangesAsync();
    Task BeginTransactionAsync();
    Task CommitTransactionAsync();
    Task RollbackTransactionAsync();
}

public class UnitOfWork : IUnitOfWork
{
    private readonly ShopDbContext _context;
    private IDbContextTransaction? _transaction;
    
    public UnitOfWork(ShopDbContext context)
    {
        _context = context;
        Customers = new CustomerRepository(_context);
        Orders = new OrderRepository(_context);
        Products = new ProductRepository(_context);
    }
    
    public ICustomerRepository Customers { get; }
    public IOrderRepository Orders { get; }
    public IProductRepository Products { get; }
    
    public async Task<int> SaveChangesAsync()
    {
        return await _context.SaveChangesAsync();
    }
    
    public async Task BeginTransactionAsync()
    {
        _transaction = await _context.Database.BeginTransactionAsync();
    }
    
    public async Task CommitTransactionAsync()
    {
        if (_transaction != null)
        {
            await _transaction.CommitAsync();
            await _transaction.DisposeAsync();
            _transaction = null;
        }
    }
    
    public async Task RollbackTransactionAsync()
    {
        if (_transaction != null)
        {
            await _transaction.RollbackAsync();
            await _transaction.DisposeAsync();
            _transaction = null;
        }
    }
    
    public void Dispose()
    {
        _transaction?.Dispose();
        _context.Dispose();
    }
}

// Service sử dụng UoW
public class OrderProcessingService
{
    private readonly IUnitOfWork _unitOfWork;
    
    public OrderProcessingService(IUnitOfWork unitOfWork)
    {
        _unitOfWork = unitOfWork;
    }
    
    public async Task<bool> ProcessOrderAsync(CreateOrderDto orderDto)
    {
        await _unitOfWork.BeginTransactionAsync();
        
        try
        {
            // Get customer
            var customer = await _unitOfWork.Customers.GetByIdAsync(orderDto.CustomerId);
            if (customer == null) return false;
            
            // Create order
            var order = new Order
            {
                CustomerId = orderDto.CustomerId,
                OrderDate = DateTime.Now,
                Status = "Processing"
            };
            
            // Add order items và update stock
            foreach (var itemDto in orderDto.Items)
            {
                var product = await _unitOfWork.Products.GetByIdAsync(itemDto.ProductId);
                if (product == null || product.Stock < itemDto.Quantity)
                {
                    await _unitOfWork.RollbackTransactionAsync();
                    return false;
                }
                
                // Update stock
                product.Stock -= itemDto.Quantity;
                await _unitOfWork.Products.UpdateAsync(product);
                
                // Add order item
                order.Items.Add(new OrderItem
                {
                    ProductId = itemDto.ProductId,
                    Quantity = itemDto.Quantity,
                    UnitPrice = product.Price
                });
            }
            
            order.Total = order.Items.Sum(i => i.Quantity * i.UnitPrice);
            await _unitOfWork.Orders.AddAsync(order);
            
            await _unitOfWork.SaveChangesAsync();
            await _unitOfWork.CommitTransactionAsync();
            
            return true;
        }
        catch
        {
            await _unitOfWork.RollbackTransactionAsync();
            return false;
        }
    }
}
```

---

## 🚨 **COMMON PITFALLS & SOLUTIONS**

### ❌ **1. N+1 Query Problem**
```csharp
// BAD - N+1 queries
public async Task<List<CustomerSummary>> GetCustomerSummariesBad()
{
    var customers = await _context.Customers.ToListAsync(); // 1 query
    var summaries = new List<CustomerSummary>();
    
    foreach (var customer in customers) // N queries
    {
        var orderCount = await _context.Orders
            .CountAsync(o => o.CustomerId == customer.Id);
        
        summaries.Add(new CustomerSummary
        {
            Name = customer.Name,
            OrderCount = orderCount
        });
    }
    
    return summaries;
}

// GOOD - Single query
public async Task<List<CustomerSummary>> GetCustomerSummariesGood()
{
    return await _context.Customers
        .Select(c => new CustomerSummary
        {
            Name = c.Name,
            OrderCount = c.Orders.Count()
        })
        .ToListAsync(); // Only 1 query!
}
```

### ❌ **2. Memory Leaks với Change Tracking**  
```csharp
// BAD - Memory leak với long-running operations
public async Task ProcessLargeDataBad()
{
    var allCustomers = await _context.Customers.ToListAsync(); // Load all into memory
    
    foreach (var customer in allCustomers)
    {
        // Process customer...
        // EF tracks all these entities in memory!
    }
    // Memory usage keeps growing...
}

// GOOD - Process in batches
public async Task ProcessLargeDataGood()
{
    const int batchSize = 1000;
    int skip = 0;
    
    while (true)
    {
        var batch = await _context.Customers
            .AsNoTracking() // No change tracking
            .Skip(skip)
            .Take(batchSize)
            .ToListAsync();
        
        if (!batch.Any()) break;
        
        foreach (var customer in batch)
        {
            // Process customer...
        }
        
        skip += batchSize;
        // Memory is freed after each batch
    }
}
```

### ❌ **3. Incorrect Async Usage**
```csharp
// BAD - Blocking async calls
public List<Customer> GetCustomersBad()
{
    return _context.Customers.ToListAsync().Result; // Deadlock risk!
}

// BAD - Mixing sync and async
public async Task<Customer> CreateCustomerBad(Customer customer)
{
    _context.Customers.Add(customer);
    _context.SaveChanges(); // Should be SaveChangesAsync()
    return customer;
}

// GOOD - Consistent async/await
public async Task<List<Customer>> GetCustomersGood()
{
    return await _context.Customers.ToListAsync();
}

public async Task<Customer> CreateCustomerGood(Customer customer)
{
    _context.Customers.Add(customer);
    await _context.SaveChangesAsync();
    return customer;
}
```

---

## 🎯 **MASTERY CHECKLIST**

### 🌱 **Beginner Level:**
```
□ Setup DbContext và connection string
□ Tạo models với basic attributes
□ Chạy migrations thành công
□ CRUD operations cơ bản
□ Hiểu Include() và ThenInclude()
```

### 🚀 **Intermediate Level:**
```
□ Fluent API configuration
□ Relationship mapping (1-1, 1-many, many-many)
□ Query optimization với AsNoTracking()
□ Transactions và error handling
□ Raw SQL khi cần thiết
```

### 🏆 **Advanced Level:**
```
□ Repository & Unit of Work patterns
□ Performance tuning và indexing
□ Complex projections và GroupBy
□ Concurrency handling
□ Custom conventions và value converters
```

---

## 🎊 **KẾT LUẬN**

Entity Framework Core là cầu nối hoàn hảo giữa thế giới OOP và Database. Nó không chỉ giúp developer viết code nhanh hơn mà còn an toàn và maintainable hơn.

### 🌟 **Key Takeaways:**
- **Productivity:** Giảm 80% code boilerplate
- **Safety:** Type-safe, compile-time checking  
- **Performance:** Khi dùng đúng cách, performance tuyệt vời
- **Maintainability:** Code clean, dễ đọc, dễ test

### 📚 **Next Steps:**
1. **Practice:** Tạo 1 project e-commerce hoàn chỉnh
2. **Advanced:** Học về Domain Events, Specification pattern
3. **Performance:** Master SQL profiling và query optimization

---

*"Entity Framework - Biến Database thành playground của developer!"* 🎮

**Tiếp theo:** JWT Authentication - Thẻ căn cước điện tử 🔐