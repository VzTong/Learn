# ⚡ **CQRS - TÁCH BIỆT ĐỌC-GHI**
*"Như ngân hàng: quầy gửi tiền khác quầy rút tiền"*

---

## 🎯 **TẠI SAO CẦN CQRS?**

### 🏦 **Ví dụ thực tế: Ngân hàng**
```
🏪 Truyền thống (1 quầy làm tất cả):
Customer → Teller → Check Balance ✅
Customer → Teller → Deposit Money ⏳ (phải đợi)
Customer → Teller → Withdraw Money ⏳ (phải đợi)
Customer → Teller → Transfer Money ⏳ (phải đợi)

😵 Kết quả: Khách hàng xếp hàng dài, chậm chạp!
```

```
🏦 CQRS (Tách quầy chuyên biệt):
👁️ QUERY (Xem số dư):
Customer → Inquiry Counter → Quick Check ⚡ (Rất nhanh)

✍️ COMMAND (Giao dịch):  
Customer → Transaction Counter → Process ⏳ (Cần thời gian)

😍 Kết quả: Xem nhanh, giao dịch an toàn!
```

### 🤔 **Vấn đề kiến trúc truyền thống:**
```csharp
❌ Traditional CRUD:
public class ProductService
{
    // Đọc và ghi dùng chung 1 model
    public async Task<Product> GetProduct(int id) { ... }      // READ
    public async Task UpdateProduct(Product product) { ... }   // WRITE  
    public async Task<List<Product>> SearchProducts() { ... }  // READ
    public async Task DeleteProduct(int id) { ... }            // WRITE
}

🚨 Vấn đề:
- Model quá phức tạp (vừa đọc vừa ghi)
- Không tối ưu được hiệu suất riêng
- Khó scale riêng từng phần
- Business logic lẫn lộn
```

---

## ⚡ **CQRS PATTERN CHI TIẾT**

### 🎭 **Kiến trúc CQRS:**
```
📝 COMMANDS (Thay đổi dữ liệu)
    ↓
CommandHandler
    ↓  
Business Logic + Validation
    ↓
Write Database (Tối ưu cho ghi)
    ↓
Domain Events (Thông báo thay đổi)

👁️ QUERIES (Lấy dữ liệu)  
    ↓
QueryHandler
    ↓
Simple Data Access (Không validation)
    ↓
Read Database (Tối ưu cho đọc)
    ↓
DTOs/ViewModels
```

### 🏗️ **Implementation với MediatR:**

```csharp
// 📁 Application/Commands/CreateProduct/CreateProductCommand.cs
public class CreateProductCommand : IRequest<Guid>
{
    public string Name { get; set; }
    public string Description { get; set; }
    public decimal Price { get; set; }
    public int CategoryId { get; set; }
}

// 📁 Application/Commands/CreateProduct/CreateProductHandler.cs
public class CreateProductHandler : IRequestHandler<CreateProductCommand, Guid>
{
    private readonly IProductRepository _repository;
    private readonly IUnitOfWork _unitOfWork;
    private readonly IDomainEventDispatcher _eventDispatcher;
    
    public CreateProductHandler(
        IProductRepository repository,
        IUnitOfWork unitOfWork, 
        IDomainEventDispatcher eventDispatcher)
    {
        _repository = repository;
        _unitOfWork = unitOfWork;
        _eventDispatcher = eventDispatcher;
    }
    
    public async Task<Guid> Handle(CreateProductCommand request, CancellationToken ct)
    {
        // 1️⃣ VALIDATION - Business Rules
        await ValidateBusinessRules(request);
        
        // 2️⃣ CREATE DOMAIN OBJECT
        var product = Product.Create(
            request.Name,
            request.Description, 
            request.Price,
            CategoryId.From(request.CategoryId)
        );
        
        // 3️⃣ SAVE TO WRITE DATABASE
        await _repository.AddAsync(product);
        await _unitOfWork.CommitAsync();
        
        // 4️⃣ PUBLISH DOMAIN EVENTS
        await _eventDispatcher.DispatchAsync(product.DomainEvents);
        
        return product.Id.Value;
    }
    
    private async Task ValidateBusinessRules(CreateProductCommand request)
    {
        // Business validation
        if (request.Price <= 0)
            throw new BusinessRuleException("Price must be positive");
            
        var existingProduct = await _repository.GetByNameAsync(request.Name);
        if (existingProduct != null)
            throw new BusinessRuleException("Product name already exists");
    }
}

// 📁 Application/Commands/UpdateProduct/UpdateProductCommand.cs
public class UpdateProductCommand : IRequest
{
    public Guid Id { get; set; }
    public string Name { get; set; }
    public string Description { get; set; }
    public decimal Price { get; set; }
}

public class UpdateProductHandler : IRequestHandler<UpdateProductCommand>
{
    private readonly IProductRepository _repository;
    private readonly IUnitOfWork _unitOfWork;
    
    public async Task Handle(UpdateProductCommand request, CancellationToken ct)
    {
        // 1️⃣ LOAD AGGREGATE
        var product = await _repository.GetByIdAsync(ProductId.From(request.Id));
        if (product == null)
            throw new NotFoundException("Product not found");
        
        // 2️⃣ BUSINESS LOGIC
        product.UpdateDetails(request.Name, request.Description, request.Price);
        
        // 3️⃣ SAVE CHANGES
        await _repository.UpdateAsync(product);
        await _unitOfWork.CommitAsync();
        
        // 4️⃣ PUBLISH EVENTS
        await PublishDomainEventsAsync(product);
    }
}
```

### 👁️ **QUERIES - Đọc Dữ Liệu:**

```csharp
// 📁 Application/Queries/GetProduct/GetProductQuery.cs
public class GetProductQuery : IRequest<ProductDto>
{
    public Guid Id { get; set; }
}

// 📁 Application/Queries/GetProduct/GetProductHandler.cs
public class GetProductHandler : IRequestHandler<GetProductQuery, ProductDto>
{
    private readonly IProductReadRepository _readRepository;
    
    public GetProductHandler(IProductReadRepository readRepository)
    {
        _readRepository = readRepository;
    }
    
    public async Task<ProductDto> Handle(GetProductQuery request, CancellationToken ct)
    {
        // 🚀 SIMPLE DATA ACCESS - Không business logic
        var product = await _readRepository.GetProductByIdAsync(request.Id);
        
        if (product == null)
            return null;
            
        // 📦 Trả về DTO đơn giản
        return new ProductDto
        {
            Id = product.Id,
            Name = product.Name,
            Description = product.Description,
            Price = product.Price,
            CategoryName = product.CategoryName,
            IsAvailable = product.Stock > 0,
            CreatedAt = product.CreatedAt
        };
    }
}

// 📁 Application/Queries/SearchProducts/SearchProductsQuery.cs
public class SearchProductsQuery : IRequest<PagedResult<ProductListDto>>
{
    public string SearchTerm { get; set; }
    public int? CategoryId { get; set; }
    public decimal? MinPrice { get; set; }
    public decimal? MaxPrice { get; set; }
    public int PageNumber { get; set; } = 1;
    public int PageSize { get; set; } = 10;
    public string SortBy { get; set; } = "Name";
    public bool SortDescending { get; set; } = false;
}

public class SearchProductsHandler : IRequestHandler<SearchProductsQuery, PagedResult<ProductListDto>>
{
    private readonly IProductReadRepository _readRepository;
    
    public SearchProductsHandler(IProductReadRepository readRepository)
    {
        _readRepository = readRepository;
    }
    
    public async Task<PagedResult<ProductListDto>> Handle(SearchProductsQuery request, CancellationToken ct)
    {
        // 🔍 COMPLEX QUERY - Tối ưu cho đọc
        var result = await _readRepository.SearchProductsAsync(
            searchTerm: request.SearchTerm,
            categoryId: request.CategoryId,
            minPrice: request.MinPrice,
            maxPrice: request.MaxPrice,
            pageNumber: request.PageNumber,
            pageSize: request.PageSize,
            sortBy: request.SortBy,
            sortDescending: request.SortDescending
        );
        
        return new PagedResult<ProductListDto>
        {
            Items = result.Items.Select(p => new ProductListDto
            {
                Id = p.Id,
                Name = p.Name,
                Price = p.Price,
                CategoryName = p.CategoryName,
                ImageUrl = p.ImageUrl,
                Rating = p.AverageRating,
                IsAvailable = p.Stock > 0
            }).ToList(),
            TotalItems = result.TotalItems,
            PageNumber = request.PageNumber,
            PageSize = request.PageSize
        };
    }
}

// 📁 Application/DTOs/ProductDto.cs  
public class ProductDto
{
    public Guid Id { get; set; }
    public string Name { get; set; }
    public string Description { get; set; }
    public decimal Price { get; set; }
    public string CategoryName { get; set; }
    public bool IsAvailable { get; set; }
    public DateTime CreatedAt { get; set; }
    public List<ProductImageDto> Images { get; set; } = new();
    public List<ProductReviewDto> Reviews { get; set; } = new();
    public decimal AverageRating { get; set; }
}

public class ProductListDto
{
    public Guid Id { get; set; }
    public string Name { get; set; }
    public decimal Price { get; set; }
    public string CategoryName { get; set; }
    public string ImageUrl { get; set; }
    public decimal Rating { get; set; }
    public bool IsAvailable { get; set; }
}
```

---

## 🏗️ **READ/WRITE DATABASE SEPARATION**

### 💾 **Write Database (Command Side):**
*"Tối ưu cho ghi dữ liệu, bảo đảm tính nhất quán"*

```csharp
// 📁 Infrastructure/Persistence/Write/WriteDbContext.cs
public class WriteDbContext : DbContext
{
    public DbSet<Product> Products { get; set; }
    public DbSet<Category> Categories { get; set; }
    public DbSet<Order> Orders { get; set; }
    
    protected override void OnModelCreating(ModelBuilder builder)
    {
        // 🎯 Configuration tập trung vào WRITE operations
        
        // Products - Normalized structure
        builder.Entity<Product>(entity =>
        {
            entity.HasKey(e => e.Id);
            entity.Property(e => e.Name).IsRequired().HasMaxLength(200);
            entity.Property(e => e.Price).HasPrecision(18, 2);
            
            // Complex relationships for data consistency
            entity.HasOne(p => p.Category)
                  .WithMany(c => c.Products)  
                  .HasForeignKey(p => p.CategoryId);
                  
            // Audit fields
            entity.Property(e => e.CreatedAt).IsRequired();
            entity.Property(e => e.UpdatedAt);
            entity.Property(e => e.Version).IsRowVersion(); // Optimistic locking
        });
        
        // Domain Events table
        builder.Entity<DomainEvent>(entity =>
        {
            entity.HasKey(e => e.Id);
            entity.Property(e => e.EventType).IsRequired();
            entity.Property(e => e.EventData).IsRequired();
            entity.Property(e => e.OccurredAt).IsRequired();
        });
    }
}

// 📁 Infrastructure/Repositories/Write/ProductRepository.cs
public class ProductRepository : IProductRepository
{
    private readonly WriteDbContext _context;
    
    public ProductRepository(WriteDbContext context)
    {
        _context = context;
    }
    
    public async Task<Product> GetByIdAsync(ProductId id)
    {
        // 🎯 Load full aggregate with all related data
        return await _context.Products
            .Include(p => p.Category)
            .Include(p => p.ProductImages)
            .Include(p => p.ProductReviews)
            .FirstOrDefaultAsync(p => p.Id == id.Value);
    }
    
    public async Task AddAsync(Product product)
    {
        _context.Products.Add(product);
    }
    
    public async Task UpdateAsync(Product product)
    {
        _context.Products.Update(product);
    }
    
    // 🔒 Optimistic Concurrency Control
    public async Task<Product> GetForUpdateAsync(ProductId id)
    {
        return await _context.Products
            .Where(p => p.Id == id.Value)
            .FirstOrDefaultAsync();
            // EF sẽ track entity để detect changes
    }
}
```

### 👁️ **Read Database (Query Side):**
*"Tối ưu cho đọc dữ liệu, denormalized để truy vấn nhanh"*

```csharp
// 📁 Infrastructure/Persistence/Read/ReadDbContext.cs
public class ReadDbContext : DbContext
{
    public DbSet<ProductReadModel> Products { get; set; }
    public DbSet<ProductListReadModel> ProductList { get; set; }
    public DbSet<CategoryReadModel> Categories { get; set; }
    
    protected override void OnModelCreating(ModelBuilder builder)
    {
        // 🚀 Configuration tập trung vào READ performance
        
        // Denormalized Product view
        builder.Entity<ProductReadModel>(entity =>
        {
            entity.HasKey(e => e.Id);
            entity.ToTable("vw_ProductDetails"); // Database View
            
            // Flat structure - no joins needed
            entity.Property(e => e.CategoryName); // Denormalized
            entity.Property(e => e.AverageRating); // Pre-calculated
            entity.Property(e => e.ReviewCount); // Pre-calculated
            entity.Property(e => e.ImageUrls); // JSON array
        });
        
        // Optimized for list queries
        builder.Entity<ProductListReadModel>(entity =>
        {
            entity.HasKey(e => e.Id);
            entity.ToTable("vw_ProductList");
            
            // Indexes for fast searching
            entity.HasIndex(e => e.Name);
            entity.HasIndex(e => e.CategoryId);
            entity.HasIndex(e => e.Price);
            entity.HasIndex(e => new { e.CategoryId, e.Price });
        });
    }
}

// 📁 Infrastructure/Repositories/Read/ProductReadRepository.cs
public interface IProductReadRepository
{
    Task<ProductReadModel> GetProductByIdAsync(Guid id);
    Task<PagedResult<ProductListReadModel>> SearchProductsAsync(
        string searchTerm = null,
        int? categoryId = null,
        decimal? minPrice = null,
        decimal? maxPrice = null,
        int pageNumber = 1,
        int pageSize = 10,
        string sortBy = "Name",
        bool sortDescending = false);
}

public class ProductReadRepository : IProductReadRepository
{
    private readonly ReadDbContext _context;
    private readonly IMemoryCache _cache;
    
    public ProductReadRepository(ReadDbContext context, IMemoryCache cache)
    {
        _context = context;
        _cache = cache;
    }
    
    public async Task<ProductReadModel> GetProductByIdAsync(Guid id)
    {
        // 🚀 Cache frequently accessed data
        var cacheKey = $"product-{id}";
        
        if (_cache.TryGetValue(cacheKey, out ProductReadModel cached))
            return cached;
            
        var product = await _context.Products
            .AsNoTracking() // Read-only, no change tracking
            .FirstOrDefaultAsync(p => p.Id == id);
            
        if (product != null)
        {
            _cache.Set(cacheKey, product, TimeSpan.FromMinutes(10));
        }
        
        return product;
    }
    
    public async Task<PagedResult<ProductListReadModel>> SearchProductsAsync(
        string searchTerm = null,
        int? categoryId = null, 
        decimal? minPrice = null,
        decimal? maxPrice = null,
        int pageNumber = 1,
        int pageSize = 10,
        string sortBy = "Name",
        bool sortDescending = false)
    {
        var query = _context.ProductList.AsNoTracking();
        
        // 🔍 Dynamic filtering
        if (!string.IsNullOrEmpty(searchTerm))
        {
            query = query.Where(p => p.Name.Contains(searchTerm) || 
                                   p.Description.Contains(searchTerm));
        }
        
        if (categoryId.HasValue)
            query = query.Where(p => p.CategoryId == categoryId.Value);
            
        if (minPrice.HasValue)
            query = query.Where(p => p.Price >= minPrice.Value);
            
        if (maxPrice.HasValue)
            query = query.Where(p => p.Price <= maxPrice.Value);
        
        // 📊 Dynamic sorting
        query = sortBy.ToLower() switch
        {
            "price" => sortDescending ? query.OrderByDescending(p => p.Price) 
                                      : query.OrderBy(p => p.Price),
            "rating" => sortDescending ? query.OrderByDescending(p => p.AverageRating)
                                       : query.OrderBy(p => p.AverageRating),
            "date" => sortDescending ? query.OrderByDescending(p => p.CreatedAt)
                                     : query.OrderBy(p => p.CreatedAt),
            _ => sortDescending ? query.OrderByDescending(p => p.Name)
                                : query.OrderBy(p => p.Name)
        };
        
        var totalItems = await query.CountAsync();
        
        var items = await query
            .Skip((pageNumber - 1) * pageSize)
            .Take(pageSize)
            .ToListAsync();
            
        return new PagedResult<ProductListReadModel>
        {
            Items = items,
            TotalItems = totalItems,
            PageNumber = pageNumber,
            PageSize = pageSize
        };
    }
}

// 📁 Infrastructure/ReadModels/ProductReadModel.cs
public class ProductReadModel
{
    public Guid Id { get; set; }
    public string Name { get; set; }
    public string Description { get; set; }
    public decimal Price { get; set; }
    public int Stock { get; set; }
    
    // Denormalized fields - no joins needed!
    public string CategoryName { get; set; }
    public decimal AverageRating { get; set; }
    public int ReviewCount { get; set; }
    public string ImageUrls { get; set; } // JSON: ["url1", "url2"]
    
    public DateTime CreatedAt { get; set; }
    
    // Computed properties
    public bool IsAvailable => Stock > 0;
    public List<string> Images => JsonSerializer.Deserialize<List<string>>(ImageUrls ?? "[]");
}
```

---

## 🔄 **EVENT-DRIVEN SYNCHRONIZATION**

### 📢 **Domain Events:**
*"Khi có thay đổi bên Write, thông báo cho Read side"*

```csharp
// 📁 Domain/Events/ProductCreatedEvent.cs
public class ProductCreatedEvent : IDomainEvent
{
    public Guid ProductId { get; }
    public string Name { get; }
    public decimal Price { get; }
    public int CategoryId { get; }
    public DateTime OccurredAt { get; }
    
    public ProductCreatedEvent(Guid productId, string name, decimal price, int categoryId)
    {
        ProductId = productId;
        Name = name;
        Price = price;
        CategoryId = categoryId;
        OccurredAt = DateTime.UtcNow;
    }
}

// 📁 Application/EventHandlers/ProductCreatedEventHandler.cs
public class ProductCreatedEventHandler : INotificationHandler<ProductCreatedEvent>
{
    private readonly IProductReadModelUpdater _readModelUpdater;
    private readonly ILogger<ProductCreatedEventHandler> _logger;
    
    public ProductCreatedEventHandler(
        IProductReadModelUpdater readModelUpdater,
        ILogger<ProductCreatedEventHandler> logger)
    {
        _readModelUpdater = readModelUpdater;
        _logger = logger;
    }
    
    public async Task Handle(ProductCreatedEvent notification, CancellationToken ct)
    {
        try
        {
            // 🔄 Update Read Model
            await _readModelUpdater.HandleProductCreatedAsync(
                notification.ProductId,
                notification.Name,
                notification.Price,
                notification.CategoryId
            );
            
            _logger.LogInformation("Read model updated for product {ProductId}", 
                notification.ProductId);
        }
        catch (Exception ex)
        {
            _logger.LogError(ex, "Failed to update read model for product {ProductId}", 
                notification.ProductId);
            
            // 🔄 Có thể retry hoặc đưa vào dead letter queue
            throw;
        }
    }
}

// 📁 Infrastructure/ReadModelUpdaters/ProductReadModelUpdater.cs
public interface IProductReadModelUpdater
{
    Task HandleProductCreatedAsync(Guid productId, string name, decimal price, int categoryId);
    Task HandleProductUpdatedAsync(Guid productId, string name, decimal price);
    Task HandleProductDeletedAsync(Guid productId);
}

public class ProductReadModelUpdater : IProductReadModelUpdater
{
    private readonly ReadDbContext _readDbContext;
    private readonly WriteDbContext _writeDbContext;
    private readonly IMemoryCache _cache;
    
    public ProductReadModelUpdater(
        ReadDbContext readDbContext,
        WriteDbContext writeDbContext,
        IMemoryCache cache)
    {
        _readDbContext = readDbContext;
        _writeDbContext = writeDbContext;
        _cache = cache;
    }
    
    public async Task HandleProductCreatedAsync(Guid productId, string name, decimal price, int categoryId)
    {
        // 1️⃣ Get additional data from write side
        var category = await _writeDbContext.Categories
            .FirstOrDefaultAsync(c => c.Id == categoryId);
            
        // 2️⃣ Create read model
        var readModel = new ProductReadModel
        {
            Id = productId,
            Name = name,
            Price = price,
            CategoryName = category?.Name ?? "Unknown",
            Stock = 0, // Default
            AverageRating = 0,
            ReviewCount = 0,
            ImageUrls = "[]",
            CreatedAt = DateTime.UtcNow
        };
        
        // 3️⃣ Save to read database
        _readDbContext.Products.Add(readModel);
        
        // Also add to list view
        var listModel = new ProductListReadModel
        {
            Id = productId,
            Name = name,
            Price = price,
            CategoryId = categoryId,
            CategoryName = category?.Name ?? "Unknown",
            AverageRating = 0,
            CreatedAt = DateTime.UtcNow
        };
        
        _readDbContext.ProductList.Add(listModel);
        
        await _readDbContext.SaveChangesAsync();
        
        // 4️⃣ Invalidate cache
        _cache.Remove($"product-{productId}");
    }
    
    public async Task HandleProductUpdatedAsync(Guid productId, string name, decimal price)
    {
        // Update both detailed and list read models
        var product = await _readDbContext.Products.FindAsync(productId);
        var listProduct = await _readDbContext.ProductList.FindAsync(productId);
        
        if (product != null)
        {
            product.Name = name;
            product.Price = price;
        }
        
        if (listProduct != null)
        {
            listProduct.Name = name;
            listProduct.Price = price;
        }
        
        await _readDbContext.SaveChangesAsync();
        
        // Invalidate cache
        _cache.Remove($"product-{productId}");
    }
}
```

---

## 🎮 **CONTROLLER LAYER:**

```csharp
// 📁 WebAPI/Controllers/ProductsController.cs
[ApiController]
[Route("api/[controller]")]
public class ProductsController : ControllerBase
{
    private readonly IMediator _mediator;
    
    public ProductsController(IMediator mediator)
    {
        _mediator = mediator;
    }
    
    /// <summary>
    /// 📝 COMMAND: Tạo sản phẩm mới
    /// </summary>
    [HttpPost]
    [ProducesResponseType(typeof(Guid), 201)]
    public async Task<IActionResult> CreateProduct([FromBody] CreateProductRequest request)
    {
        var command = new CreateProductCommand
        {
            Name = request.Name,
            Description = request.Description,
            Price = request.Price,
            CategoryId = request.CategoryId
        };
        
        var productId = await _mediator.Send(command);
        
        return CreatedAtAction(nameof(GetProduct), new { id = productId }, productId);
    }
    
    /// <summary>
    /// 📝 COMMAND: Cập nhật sản phẩm
    /// </summary>
    [HttpPut("{id:guid}")]
    [ProducesResponseType(204)]
    public async Task<IActionResult> UpdateProduct(Guid id, [FromBody] UpdateProductRequest request)
    {
        var command = new UpdateProductCommand
        {
            Id = id,
            Name = request.Name,
            Description = request.Description,
            Price = request.Price
        };
        
        await _mediator.Send(command);
        
        return NoContent();
    }
    
    /// <summary>
    /// 👁️ QUERY: Lấy chi tiết sản phẩm
    /// </summary>
    [HttpGet("{id:guid}")]
    [ProducesResponseType(typeof(ProductDto), 200)]
    public async Task<IActionResult> GetProduct(Guid id)
    {
        var query = new GetProductQuery { Id = id };
        var result = await _mediator.Send(query);
        
        if (result == null)
            return NotFound();
            
        return Ok(result);
    }
    
    /// <summary>
    /// 👁️ QUERY: Tìm kiếm sản phẩm với phân trang
    /// </summary>
    [HttpGet]
    [ProducesResponseType(typeof(PagedResult<ProductListDto>), 200)]
    public async Task<IActionResult> SearchProducts([FromQuery] SearchProductsRequest request)
    {
        var query = new SearchProductsQuery
        {
            SearchTerm = request.SearchTerm,
            CategoryId = request.CategoryId,
            MinPrice = request.MinPrice,
            MaxPrice = request.MaxPrice,
            PageNumber = request.PageNumber,
            PageSize = request.PageSize,
            SortBy = request.SortBy,
            SortDescending = request.SortDescending
        };
        
        var result = await _mediator.Send(query);
        
        return Ok(result);
    }
    
    /// <summary>
    /// 📝 COMMAND: Xóa sản phẩm
    /// </summary>
    [HttpDelete("{id:guid}")]
    [ProducesResponseType(204)]
    public async Task<IActionResult> DeleteProduct(Guid id)
    {
        var command = new DeleteProductCommand { Id = id };
        await _mediator.Send(command);
        
        return NoContent();
    }
}

// 📁 WebAPI/Requests/CreateProductRequest.cs
public class CreateProductRequest
{
    [Required]
    [StringLength(200, MinimumLength = 3)]
    public string Name { get; set; }
    
    [StringLength(1000)]
    public string Description { get; set; }
    
    [Required]
    [Range(0.01, double.MaxValue)]
    public decimal Price { get; set; }
    
    [Required]
    public int CategoryId { get; set; }
}

public class SearchProductsRequest
{
    public string SearchTerm { get; set; }
    public int? CategoryId { get; set; }
    public decimal? MinPrice { get; set; }
    public decimal? MaxPrice { get; set; }
    
    [Range(1, int.MaxValue)]
    public int PageNumber { get; set; } = 1;
    
    [Range(1, 100)]
    public int PageSize { get; set; } = 10;
    
    public string SortBy { get; set; } = "Name";
    public bool SortDescending { get; set; } = false;
}
```

---

## 🚀 **ADVANCED CQRS PATTERNS**

### 📊 **Event Sourcing Integration:**
*"Lưu trữ tất cả events, rebuild state khi cần"*

```csharp
// 📁 Infrastructure/EventStore/IEventStore.cs
public interface IEventStore
{
    Task SaveEventsAsync(Guid aggregateId, IEnumerable<IDomainEvent> events, int expectedVersion);
    Task<List<IDomainEvent>> GetEventsAsync(Guid aggregateId);
    Task<List<IDomainEvent>> GetEventsAsync(Guid aggregateId, int fromVersion);
}

// 📁 Domain/Aggregates/Product.cs (Event Sourced)
public class Product : AggregateRoot
{
    public ProductId Id { get; private set; }
    public string Name { get; private set; }
    public decimal Price { get; private set; }
    
    // Event Sourcing: Rebuild từ events
    public static Product FromHistory(IEnumerable<IDomainEvent> events)
    {
        var product = new Product();
        
        foreach (var @event in events)
        {
            product.Apply(@event);
        }
        
        return product;
    }
    
    private void Apply(IDomainEvent @event)
    {
        switch (@event)
        {
            case ProductCreatedEvent e:
                Id = ProductId.From(e.ProductId);
                Name = e.Name;
                Price = e.Price;
                break;
                
            case ProductNameChangedEvent e:
                Name = e.NewName;
                break;
                
            case ProductPriceChangedEvent e:
                Price = e.NewPrice;
                break;
        }
    }
}
```

### 🔄 **Eventual Consistency với Saga Pattern:**

```csharp
// 📁 Application/Sagas/OrderProcessingSaga.cs
public class OrderProcessingSaga : ISaga<OrderCreatedEvent>,
                                   ISaga<PaymentProcessedEvent>,
                                   ISaga<InventoryReservedEvent>
{
    public async Task Handle(OrderCreatedEvent @event)
    {
        // 1️⃣ Process payment
        await SendCommand(new ProcessPaymentCommand 
        { 
            OrderId = @event.OrderId,
            Amount = @event.TotalAmount 
        });
        
        // 2️⃣ Reserve inventory
        await SendCommand(new ReserveInventoryCommand
        {
            OrderId = @event.OrderId,
            Items = @event.Items
        });
    }
    
    public async Task Handle(PaymentProcessedEvent @event)
    {
        if (@event.Success)
        {
            // Continue workflow
            await SendCommand(new ConfirmOrderCommand { OrderId = @event.OrderId });
        }
        else
        {
            // Compensate
            await SendCommand(new CancelOrderCommand { OrderId = @event.OrderId });
        }
    }
}
```

---

## 🎯 **KHI NÀO DÙNG CQRS?**

### ✅ **Nên dùng CQRS khi:**
1. **📊 Performance Requirements:**
   - Đọc nhiều hơn ghi (10:1 ratio trở lên)
   - Cần tối ưu query phức tạp
   - Scale đọc/ghi độc lập

2. **🏢 Complex Business Logic:**
   - Business rules phức tạp ở write side
   - Cần audit trail đầy đủ
   - Multiple bounded contexts

3. **👥 Team Structure:**
   - Team lớn, có thể tách read/write team
   - Khác nhau về skillset (DBA vs Developer)

### ❌ **KHÔNG nên dùng CQRS khi:**
1. **🚀 Simple CRUD Applications**
2. **👤 Small Team (< 5 people)**
3. **⏰ Tight Deadlines**
4. **📈 Uncertain Requirements**

---

## 🎊 **KẾT LUẬN**

### ✨ **Lợi ích của CQRS:**
- 🚀 **Performance:** Tối ưu riêng cho đọc/ghi
- 📈 **Scalability:** Scale độc lập từng side
- 🔒 **Security:** Phân quyền riêng biệt
- 🧹 **Clean Code:** Tách biệt concerns rõ ràng
- 🧪 **Testability:** Test riêng từng use case

### ⚠️ **Trade-offs:**
- 🔄 **Complexity:** Eventual consistency
- 💾 **Storage:** Cần nhiều database hơn
- 🔀 **Synchronization:** Phải handle data sync
- 📚 **Learning Curve:** Pattern phức tạp hơn

---

*"CQRS không phải silver bullet, nhưng là công cụ mạnh mẽ khi dùng đúng chỗ!"* ⚡