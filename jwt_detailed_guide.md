# 🔐 JWT - "THẺ CĂN CƯỚC ĐIỆN TỬ"
*Chi tiết từ A-Z: Hiểu sâu, làm chắc, áp dụng thành thạo*

---

## 🎯 **JWT LÀ GÌ? TẠI SAO QUAN TRỌNG?**

### 🎫 **Ví dụ đời thực: Vé xem phim**
```
🎬 Rạp chiếu phim truyền thống:
1. Mua vé → Nhận vé giấy có mã QR
2. Vào rạp → Quét mã QR 
3. Nhân viên kiểm tra → Cho vào xem phim
4. Vé có thời hạn → Hết giờ thì không dùng được

🔐 JWT hoạt động tương tự:
1. Đăng nhập → Nhận JWT token
2. Gọi API → Gửi kèm JWT
3. Server kiểm tra → Cho phép truy cập
4. Token có expiry → Hết hạn thì phải đăng nhập lại
```

### ⚡ **So sánh với Session truyền thống:**

| **Session Cookie** 🍪 | **JWT Token** 🎫 |
|---|---|
| Server lưu thông tin session | Client lưu thông tin trong token |
| Cần database/cache để lưu | Không cần lưu gì ở server |
| Khó scale với nhiều server | Dễ scale, stateless |
| An toàn hơn (server control) | Linh hoạt hơn (client control) |

---

## 🏗️ **CẤU TRÚC JWT: BA PHẦN QUAN TRỌNG**

### 📋 **Cấu trúc tổng quan:**
```
eyJhbGciOiJIUzI1NiJ9.eyJzdWIiOiIxMjM0NTY3ODkwIiwibmFtZSI6IkpvaG4gRG9lIn0.Gfx6VO9tcxwk6xqx9yYzSN5_7mZbHcmuV
     HEADER           .           PAYLOAD            .              SIGNATURE
```

### 🔍 **PHẦN 1: HEADER**
*"Chứng minh thư của token"*

```json
{
  "alg": "HS256",      // Thuật toán mã hóa
  "typ": "JWT"         // Loại token
}
```

**🎯 Các thuật toán phổ biến:**
- **HS256** (HMAC SHA256): Dùng secret key, nhanh, phù hợp app nhỏ
- **RS256** (RSA SHA256): Dùng public/private key, an toàn hơn, phù hợp microservice
- **ES256** (ECDSA SHA256): Nhanh + an toàn, trending hiện tại

### 📦 **PHẦN 2: PAYLOAD**
*"Vali chứa thông tin quan trọng"*

```json
{
  // Standard Claims (Chuẩn quốc tế)
  "iss": "myapp.com",           // Issuer - Ai tạo token
  "sub": "user123",             // Subject - Token của ai
  "aud": "mobile-app",          // Audience - Dùng cho app nào
  "exp": 1640995200,            // Expiry - Hết hạn khi nào
  "iat": 1640908800,            // Issued At - Tạo lúc nào
  "jti": "abc123",              // JWT ID - ID duy nhất
  
  // Custom Claims (Tự định nghĩa)
  "name": "Nguyễn Văn A",       // Tên user
  "email": "user@email.com",    // Email
  "role": "admin",              // Vai trò
  "permissions": ["read", "write"] // Quyền hạn
}
```

**⚠️ Lưu ý quan trọng:**
```
✅ NÊN lưu: user_id, role, permissions, expiry
❌ KHÔNG lưu: password, credit_card, sensitive_data

Lý do: JWT chỉ mã hóa, KHÔNG bảo mật 100%
Ai cũng có thể decode và đọc được payload!
```

### 🔒 **PHẦN 3: SIGNATURE**
*"Chữ ký xác thực"*

```javascript
// Công thức tạo signature
HMACSHA256(
  base64UrlEncode(header) + "." + base64UrlEncode(payload),
  your-256-bit-secret
)
```

**🛡️ Vai trò:**
- **Đảm bảo tính toàn vẹn:** Token không bị sửa đổi
- **Xác thực nguồn gốc:** Chỉ server có secret key mới tạo được
- **Phát hiện giả mạo:** Thay đổi 1 ký tự → signature sai → từ chối

---

## ⚡ **FLOW HOẠT ĐỘNG CHI TIẾT**

### 🎬 **Kịch bản hoàn chỉnh:**

```mermaid
sequenceDiagram
    participant U as User (Browser/Mobile)
    participant A as Auth Server
    participant R as Resource Server

    Note over U,R: 1. ĐĂNG NHẬP
    U->>A: POST /login (username, password)
    A->>A: Validate credentials
    A->>A: Create JWT token
    A->>U: Return JWT + Refresh Token

    Note over U,R: 2. TRUY CẬP TÀI NGUYÊN
    U->>R: GET /api/users (Authorization: Bearer JWT)
    R->>R: Verify JWT signature
    R->>R: Check expiry time
    R->>R: Extract user info from payload
    R->>U: Return user data

    Note over U,R: 3. LÀM MỚI TOKEN
    U->>A: POST /refresh (refresh_token)
    A->>A: Validate refresh token
    A->>A: Create new JWT
    A->>U: Return new JWT
```

### 🚀 **Code Implementation từng bước:**

**BƯỚC 1: Cấu hình JWT Service**
```csharp
// JwtService.cs
public class JwtService
{
    private readonly IConfiguration _config;
    private readonly SymmetricSecurityKey _key;

    public JwtService(IConfiguration config)
    {
        _config = config;
        _key = new SymmetricSecurityKey(Encoding.UTF8.GetBytes(_config["JWT:SecretKey"]));
    }

    public string GenerateToken(User user)
    {
        var claims = new List<Claim>
        {
            new Claim(ClaimTypes.NameIdentifier, user.Id.ToString()),
            new Claim(ClaimTypes.Name, user.Username),
            new Claim(ClaimTypes.Email, user.Email),
            new Claim(ClaimTypes.Role, user.Role),
            new Claim("permissions", string.Join(",", user.Permissions))
        };

        var creds = new SigningCredentials(_key, SecurityAlgorithms.HmacSha256);
        
        var tokenDescriptor = new SecurityTokenDescriptor
        {
            Subject = new ClaimsIdentity(claims),
            Expires = DateTime.UtcNow.AddMinutes(_config.GetValue<int>("JWT:ExpiryMinutes")),
            SigningCredentials = creds,
            Issuer = _config["JWT:Issuer"],
            Audience = _config["JWT:Audience"]
        };

        var tokenHandler = new JwtSecurityTokenHandler();
        var token = tokenHandler.CreateToken(tokenDescriptor);
        
        return tokenHandler.WriteToken(token);
    }

    public ClaimsPrincipal ValidateToken(string token)
    {
        var tokenHandler = new JwtSecurityTokenHandler();
        
        var validationParameters = new TokenValidationParameters
        {
            ValidateIssuerSigningKey = true,
            IssuerSigningKey = _key,
            ValidateIssuer = true,
            ValidIssuer = _config["JWT:Issuer"],
            ValidateAudience = true,
            ValidAudience = _config["JWT:Audience"],
            ValidateLifetime = true,
            ClockSkew = TimeSpan.Zero // Không cho phép lệch thời gian
        };

        var principal = tokenHandler.ValidateToken(token, validationParameters, out _);
        return principal;
    }
}
```

**BƯỚC 2: Controller đăng nhập**
```csharp
[ApiController]
[Route("api/[controller]")]
public class AuthController : ControllerBase
{
    private readonly JwtService _jwtService;
    private readonly IUserService _userService;

    public AuthController(JwtService jwtService, IUserService userService)
    {
        _jwtService = jwtService;
        _userService = userService;
    }

    [HttpPost("login")]
    public async Task<IActionResult> Login([FromBody] LoginRequest request)
    {
        // 1. Validate input
        if (string.IsNullOrEmpty(request.Username) || string.IsNullOrEmpty(request.Password))
            return BadRequest("Username và Password không được rỗng");

        // 2. Verify credentials
        var user = await _userService.ValidateUserAsync(request.Username, request.Password);
        if (user == null)
            return Unauthorized("Sai username hoặc password");

        // 3. Generate tokens
        var accessToken = _jwtService.GenerateToken(user);
        var refreshToken = _jwtService.GenerateRefreshToken();

        // 4. Save refresh token to database
        await _userService.SaveRefreshTokenAsync(user.Id, refreshToken);

        // 5. Return tokens
        return Ok(new LoginResponse
        {
            AccessToken = accessToken,
            RefreshToken = refreshToken,
            ExpiresIn = 900, // 15 phút
            TokenType = "Bearer",
            User = new UserInfo
            {
                Id = user.Id,
                Username = user.Username,
                Email = user.Email,
                Role = user.Role
            }
        });
    }

    [HttpPost("refresh")]
    public async Task<IActionResult> RefreshToken([FromBody] RefreshTokenRequest request)
    {
        var user = await _userService.ValidateRefreshTokenAsync(request.RefreshToken);
        if (user == null)
            return Unauthorized("Refresh token không hợp lệ");

        var newAccessToken = _jwtService.GenerateToken(user);
        var newRefreshToken = _jwtService.GenerateRefreshToken();

        await _userService.SaveRefreshTokenAsync(user.Id, newRefreshToken);

        return Ok(new LoginResponse
        {
            AccessToken = newAccessToken,
            RefreshToken = newRefreshToken,
            ExpiresIn = 900
        });
    }
}
```

**BƯỚC 3: Middleware xác thực**
```csharp
public class JwtMiddleware
{
    private readonly RequestDelegate _next;
    private readonly JwtService _jwtService;

    public JwtMiddleware(RequestDelegate next, JwtService jwtService)
    {
        _next = next;
        _jwtService = jwtService;
    }

    public async Task InvokeAsync(HttpContext context)
    {
        var token = ExtractTokenFromHeader(context.Request);
        
        if (!string.IsNullOrEmpty(token))
        {
            try
            {
                var principal = _jwtService.ValidateToken(token);
                context.User = principal;
            }
            catch (SecurityTokenExpiredException)
            {
                context.Response.StatusCode = 401;
                await context.Response.WriteAsync("Token đã hết hạn");
                return;
            }
            catch (Exception)
            {
                context.Response.StatusCode = 401;
                await context.Response.WriteAsync("Token không hợp lệ");
                return;
            }
        }

        await _next(context);
    }

    private string ExtractTokenFromHeader(HttpRequest request)
    {
        var authHeader = request.Headers["Authorization"].FirstOrDefault();
        if (authHeader != null && authHeader.StartsWith("Bearer "))
        {
            return authHeader.Substring(7); // Bỏ "Bearer "
        }
        return null;
    }
}
```

---

## 🛡️ **BẢO MẬT VÀ BEST PRACTICES**

### ⚠️ **Các lỗ hổng bảo mật phổ biến:**

**1. 🔓 Token không có expiry time**
```csharp
❌ SAI:
var tokenDescriptor = new SecurityTokenDescriptor
{
    Subject = new ClaimsIdentity(claims),
    // Không set Expires → Token sống mãi!
    SigningCredentials = creds
};

✅ ĐÚNG:
var tokenDescriptor = new SecurityTokenDescriptor
{
    Subject = new ClaimsIdentity(claims),
    Expires = DateTime.UtcNow.AddMinutes(15), // 15 phút
    SigningCredentials = creds
};
```

**2. 🔑 Secret key yếu**
```csharp
❌ SAI:
"JWT:SecretKey": "123456"  // Quá ngắn, dễ bị crack

✅ ĐÚNG:
"JWT:SecretKey": "MyVeryLongAndSecureSecretKeyWithAtLeast256Bits2024!"
// Ít nhất 32 ký tự (256 bits)
```

**3. 💾 Lưu sensitive data trong payload**
```json
❌ SAI:
{
  "user_id": 123,
  "password": "123456",        // NGUY HIỂM!
  "credit_card": "1234-5678",  // NGUY HIỂM!
  "salary": 50000              // NGUY HIỂM!
}

✅ ĐÚNG:
{
  "user_id": 123,
  "username": "john_doe",
  "role": "user",
  "permissions": ["read_profile", "update_profile"]
}
```

### 🔐 **Security Best Practices:**

**1. Refresh Token Strategy**
```csharp
public class TokenPair
{
    public string AccessToken { get; set; }    // 15 phút
    public string RefreshToken { get; set; }   // 7 ngày
}

// Tại sao cần 2 token?
// - Access Token: Ngắn hạn, dùng cho API calls
// - Refresh Token: Dài hạn, chỉ dùng để lấy access token mới
// - Nếu access token bị đánh cắp → Chỉ dùng được 15 phút
// - Nếu refresh token bị đánh cắp → Có thể revoke ngay
```

**2. Token Blacklist**
```csharp
public class TokenBlacklistService
{
    private readonly IMemoryCache _cache;
    
    public async Task BlacklistTokenAsync(string tokenId, DateTime expiry)
    {
        // Lưu vào cache/redis với TTL = thời gian còn lại của token
        var ttl = expiry - DateTime.UtcNow;
        _cache.Set($"blacklist:{tokenId}", true, ttl);
    }
    
    public async Task<bool> IsTokenBlacklistedAsync(string tokenId)
    {
        return _cache.TryGetValue($"blacklist:{tokenId}", out _);
    }
}

// Dùng khi:
// - User logout → Blacklist access token
// - Phát hiện token bị đánh cắp → Blacklist ngay
// - User đổi password → Blacklist tất cả token cũ
```

**3. HTTPS và Secure Headers**
```csharp
// Program.cs
app.UseHttpsRedirection();
app.UseHsts(); // HTTP Strict Transport Security

// Custom middleware
app.Use((context, next) =>
{
    context.Response.Headers.Add("X-Content-Type-Options", "nosniff");
    context.Response.Headers.Add("X-Frame-Options", "DENY");
    context.Response.Headers.Add("X-XSS-Protection", "1; mode=block");
    return next();
});
```

---

## 📱 **CLIENT-SIDE IMPLEMENTATION**

### 🌐 **JavaScript/React Example:**

```javascript
class AuthService {
    constructor() {
        this.baseURL = 'https://api.example.com';
        this.accessToken = localStorage.getItem('accessToken');
        this.refreshToken = localStorage.getItem('refreshToken');
    }

    async login(username, password) {
        try {
            const response = await fetch(`${this.baseURL}/auth/login`, {
                method: 'POST',
                headers: { 'Content-Type': 'application/json' },
                body: JSON.stringify({ username, password })
            });

            if (response.ok) {
                const data = await response.json();
                this.saveTokens(data.accessToken, data.refreshToken);
                return { success: true, user: data.user };
            } else {
                return { success: false, error: 'Đăng nhập thất bại' };
            }
        } catch (error) {
            return { success: false, error: error.message };
        }
    }

    saveTokens(accessToken, refreshToken) {
        this.accessToken = accessToken;
        this.refreshToken = refreshToken;
        localStorage.setItem('accessToken', accessToken);
        localStorage.setItem('refreshToken', refreshToken);
    }

    async apiCall(url, options = {}) {
        let token = this.accessToken;
        
        // Kiểm tra token có hết hạn không
        if (this.isTokenExpired(token)) {
            const refreshed = await this.refreshAccessToken();
            if (!refreshed) {
                this.logout();
                throw new Error('Session expired. Please login again.');
            }
            token = this.accessToken;
        }

        const headers = {
            'Authorization': `Bearer ${token}`,
            'Content-Type': 'application/json',
            ...options.headers
        };

        const response = await fetch(url, { ...options, headers });
        
        if (response.status === 401) {
            // Token có thể bị blacklist, thử refresh
            const refreshed = await this.refreshAccessToken();
            if (refreshed) {
                // Retry với token mới
                headers.Authorization = `Bearer ${this.accessToken}`;
                return fetch(url, { ...options, headers });
            } else {
                this.logout();
                throw new Error('Authentication failed');
            }
        }

        return response;
    }

    isTokenExpired(token) {
        if (!token) return true;
        
        try {
            const payload = JSON.parse(atob(token.split('.')[1]));
            const currentTime = Date.now() / 1000;
            return payload.exp < currentTime;
        } catch {
            return true;
        }
    }

    async refreshAccessToken() {
        try {
            const response = await fetch(`${this.baseURL}/auth/refresh`, {
                method: 'POST',
                headers: { 'Content-Type': 'application/json' },
                body: JSON.stringify({ refreshToken: this.refreshToken })
            });

            if (response.ok) {
                const data = await response.json();
                this.saveTokens(data.accessToken, data.refreshToken);
                return true;
            }
        } catch (error) {
            console.error('Refresh token failed:', error);
        }
        
        return false;
    }

    logout() {
        this.accessToken = null;
        this.refreshToken = null;
        localStorage.removeItem('accessToken');
        localStorage.removeItem('refreshToken');
        window.location.href = '/login';
    }

    getCurrentUser() {
        if (!this.accessToken) return null;
        
        try {
            const payload = JSON.parse(atob(this.accessToken.split('.')[1]));
            return {
                id: payload.nameid,
                username: payload.unique_name,
                email: payload.email,
                role: payload.role,
                permissions: payload.permissions?.split(',') || []
            };
        } catch {
            return null;
        }
    }
}

// Sử dụng
const authService = new AuthService();

// Đăng nhập
const loginResult = await authService.login('username', 'password');
if (loginResult.success) {
    console.log('Đăng nhập thành công:', loginResult.user);
}

// Gọi API
try {
    const response = await authService.apiCall('/api/users/profile');
    const profile = await response.json();
    console.log('Profile:', profile);
} catch (error) {
    console.error('API call failed:', error.message);
}
```

---

## 🚀 **TESTING JWT**

### 🧪 **Unit Tests:**

```csharp
[TestClass]
public class JwtServiceTests
{
    private JwtService _jwtService;
    private IConfiguration _config;

    [TestInitialize]
    public void Setup()
    {
        var configData = new Dictionary<string, string>
        {
            {"JWT:SecretKey", "MyTestSecretKeyForUnitTestingPurposes123!"},
            {"JWT:Issuer", "TestIssuer"},
            {"JWT:Audience", "TestAudience"},
            {"JWT:ExpiryMinutes", "15"}
        };
        
        _config = new ConfigurationBuilder()
            .AddInMemoryCollection(configData)
            .Build();
            
        _jwtService = new JwtService(_config);
    }

    [TestMethod]
    public void GenerateToken_ShouldReturnValidToken()
    {
        // Arrange
        var user = new User 
        { 
            Id = 1, 
            Username = "testuser", 
            Email = "test@email.com",
            Role = "user" 
        };

        // Act
        var token = _jwtService.GenerateToken(user);

        // Assert
        Assert.IsNotNull(token);
        Assert.IsTrue(token.Split('.').Length == 3); // Header.Payload.Signature
        
        // Decode và kiểm tra payload
        var handler = new JwtSecurityTokenHandler();
        var jsonToken = handler.ReadJwtToken(token);
        
        Assert.AreEqual("1", jsonToken.Claims.FirstOrDefault(x => x.Type == ClaimTypes.NameIdentifier)?.Value);
        Assert.AreEqual("testuser", jsonToken.Claims.FirstOrDefault(x => x.Type == ClaimTypes.Name)?.Value);
    }

    [TestMethod]
    public void ValidateToken_WithValidToken_ShouldReturnPrincipal()
    {
        // Arrange
        var user = new User { Id = 1, Username = "testuser" };
        var token = _jwtService.GenerateToken(user);

        // Act
        var principal = _jwtService.ValidateToken(token);

        // Assert
        Assert.IsNotNull(principal);
        Assert.AreEqual("1", principal.FindFirst(ClaimTypes.NameIdentifier)?.Value);
    }

    [TestMethod]
    [ExpectedException(typeof(SecurityTokenExpiredException))]
    public void ValidateToken_WithExpiredToken_ShouldThrowException()
    {
        // Arrange - Tạo token đã hết hạn
        var expiredToken = CreateExpiredToken();

        // Act & Assert
        _jwtService.ValidateToken(expiredToken);
    }
}
```

### 🔍 **Integration Tests:**

```csharp
[TestClass]
public class AuthControllerIntegrationTests
{
    private TestServer _server;
    private HttpClient _client;

    [TestInitialize]
    public void Setup()
    {
        var webHostBuilder = new WebHostBuilder()
            .UseEnvironment("Testing")
            .UseStartup<TestStartup>();
            
        _server = new TestServer(webHostBuilder);
        _client = _server.CreateClient();
    }

    [TestMethod]
    public async Task Login_WithValidCredentials_ShouldReturnToken()
    {
        // Arrange
        var loginRequest = new LoginRequest 
        { 
            Username = "testuser", 
            Password = "password123" 
        };
        var json = JsonConvert.SerializeObject(loginRequest);
        var content = new StringContent(json, Encoding.UTF8, "application/json");

        // Act
        var response = await _client.PostAsync("/api/auth/login", content);

        // Assert
        Assert.AreEqual(HttpStatusCode.OK, response.StatusCode);
        
        var responseContent = await response.Content.ReadAsStringAsync();
        var loginResponse = JsonConvert.DeserializeObject<LoginResponse>(responseContent);
        
        Assert.IsNotNull(loginResponse.AccessToken);
        Assert.IsNotNull(loginResponse.RefreshToken);
        Assert.AreEqual("Bearer", loginResponse.TokenType);
    }

    [TestMethod]
    public async Task AccessProtectedEndpoint_WithValidToken_ShouldReturnData()
    {
        // Arrange - Đăng nhập trước
        var token = await GetValidTokenAsync();
        _client.DefaultRequestHeaders.Authorization = 
            new AuthenticationHeaderValue("Bearer", token);

        // Act
        var response = await _client.GetAsync("/api/users/profile");

        // Assert
        Assert.AreEqual(HttpStatusCode.OK, response.StatusCode);
    }

    [TestMethod]
    public async Task AccessProtectedEndpoint_WithoutToken_ShouldReturn401()
    {
        // Act
        var response = await _client.GetAsync("/api/users/profile");

        // Assert
        Assert.AreEqual(HttpStatusCode.Unauthorized, response.StatusCode);
    }
}
```

---

## 🎯 **KẾT LUẬN VÀ ROADMAP**

### ✅ **Checklist thành thạo JWT:**

**Cấp độ Beginner:**
- [ ] Hiểu được JWT là gì, tại sao cần dùng
- [ ] Phân biệt được 3 phần của JWT
- [ ] Implement được login/logout cơ bản
- [ ] Sử dụng được JWT trong API calls

**Cấp độ Intermediate:**
- [ ] Implement refresh token mechanism
- [ ] Xử lý token expiry và auto-refresh
- [ ] Áp dụng được authorization (roles, permissions)
- [ ] Viết được unit tests cho JWT service

**Cấp độ Advanced:**
- [ ] Implement token blacklist
- [ ] Xử lý multiple device login
- [ ] Optimize performance (caching, Redis)
- [ ] Security hardening (rate limiting, monitoring)

### 🚀 **Bước tiếp theo:**
1. **Thực hành:** Tạo một mini project với JWT authentication
2. **Mở rộng:** Tích hợp với OAuth2, OpenID Connect
3. **Production:** Deploy với HTTPS, monitoring, logging
4. **Nâng cao:** Microservices authentication, API Gateway

---

*"JWT không chỉ là công nghệ, mà là nền tảng cho mọi ứng dụng hiện đại!"* 🔐✨