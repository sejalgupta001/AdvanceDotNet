# JWT Authentication and Authorization in ASP.NET Core Web API

## 1. Introduction

### What is Authentication?

Authentication verifies **who the user is** (login with username/password).

### What is Authorization?

Authorization decides **what the user is allowed to do** (Admin, User, Manager roles, etc.).

### What is JWT (JSON Web Token)?

JWT is a secure, compact token used to transmit user identity and roles between client and server.

Structure of JWT:

- Header – algorithm & token type
- Payload – user data (claims like username, role)
- Signature – ensures token is not tampered

Example:

```
xxxxx.yyyyy.zzzzz
```
<img width="1238" height="598" alt="image" src="https://github.com/user-attachments/assets/679269e6-60f6-45d6-828d-59eeff255da5" />
<img width="1286" height="707" alt="image" src="https://github.com/user-attachments/assets/bb105e4f-ae8c-4972-9c60-c2337cc34923" />

---

## 2. Why Use JWT in Web API?

- Stateless (no session stored on server)
- Scalable for distributed systems
- Secure and widely supported
- Perfect for Web, Mobile, and SPA apps

---

## 3. User Table Columns:

- UserId
- UserName
- Email
- Password
- Role

---

## 4. Install Required NuGet Packages

Install the following packages:

- Microsoft.AspNetCore.Authentication.JwtBearer
- Microsoft.IdentityModel.Tokens
- System.IdentityModel.Tokens.Jwt

---

## 5. Configure appsettings.json

Add JWT configuration section:

```json
"Jwt": {
  "Key": "ThisIsASecretKeyForJwtTokenGeneration",
  "Issuer": "https://localhost:7093",
  "Audience": "*",
  "TokenExpiryMinutes": 60
}
```

### Explanation:

- Key → Secret key used to sign token (keep it private)
- Issuer → Who created the token
- Audience → Who can use the token
- TokenExpiryMinutes → Token validity time

---

## 6. Configure JWT in Program.cs

```csharp
using Microsoft.AspNetCore.Authentication.JwtBearer;

builder.Services.AddScoped<IAuthService, AuthService>();

// Configure Authentication with JWT
builder.Services.AddAuthentication(JwtBearerDefaults.AuthenticationScheme)
    .AddJwtBearer(options =>
    {
        options.TokenValidationParameters = new TokenValidationParameters
        {
            ValidateIssuer = true,               // Validate token issuer
            ValidateAudience = true,             // Validate token audience
            ValidateLifetime = true,             // Validate token expiry
            ValidateIssuerSigningKey = true,     // Validate signing key

            ValidIssuer = builder.Configuration["Jwt:Issuer"],
            ValidAudience = builder.Configuration["Jwt:Audience"],

            // Secret key used to verify token
            IssuerSigningKey = new SymmetricSecurityKey(
                Encoding.UTF8.GetBytes(builder.Configuration["Jwt:Key"])
            )
        };
    });

builder.Services.AddAuthorization();

var app = builder.Build();

// Middleware order is IMPORTANT
app.UseAuthentication();   // First authenticate
app.UseAuthorization();    // Then authorize

app.MapControllers();
app.Run();
```

---

## 7. Create Auth Service

### Interface – IAuthService.cs

```csharp
public interface IAuthService
{
    Task<object> LoginAsync(string username, string password);
    string GenerateJwtToken(User user);
}
```

---

### Implementation – AuthService.cs

```csharp
public class AuthService : IAuthService
{
    private readonly AddressDemoDbContext _context;
    private readonly IConfiguration _configuration;

    public AuthService(AddressDemoDbContext context, IConfiguration configuration)
    {
        _context = context;
        _configuration = configuration;
    }

    // Generate JWT Token
    public string GenerateJwtToken(User user)
    {
        var jwtSettings = _configuration.GetSection("Jwt");

        // Create secret key
        var key = new SymmetricSecurityKey(
            Encoding.UTF8.GetBytes(jwtSettings["Key"])
        );

        // Signing credentials uses Hash-based Message Authentication Code
        var creds = new SigningCredentials(key, SecurityAlgorithms.HmacSha256);

        // Add claims (user info inside token)
        var claims = new[]
        {
            new Claim(ClaimTypes.Name, user.UserName),
            new Claim(ClaimTypes.Email, user.Email),
            new Claim(ClaimTypes.Role, user.Role),
            new Claim("UserId", user.UserId.ToString())
        };

        // Token expiry time
        var expiryMinutes = Convert.ToDouble(jwtSettings["TokenExpiryMinutes"]);

        // Create token
        var token = new JwtSecurityToken(
            issuer: jwtSettings["Issuer"],
            audience: jwtSettings["Audience"],
            claims: claims,
            expires: DateTime.Now.AddMinutes(expiryMinutes),
            signingCredentials: creds
        );

        // Return token string
        return new JwtSecurityTokenHandler().WriteToken(token);
    }

    // Login Logic
    public async Task<object> LoginAsync(string username, string password)
    {
        var user = await _context.Users
            .FirstOrDefaultAsync(u => u.UserName == username && u.Password == password);

        if (user == null)
            return null;

        var token = GenerateJwtToken(user);

        return new
        {
            token,
            user = new
            {
                user.UserId,
                user.UserName,
                user.Email,
                user.Role
            }
        };
    }
}
```

---

## 8. Register AuthService in Program.cs

```csharp
builder.Services.AddScoped<IAuthService, AuthService>();
```

---

## 9. Create Login API – UserController

```csharp
[ApiController]
[Route("api/[controller]")]
public class UserController : ControllerBase
{
    private readonly IAuthService _authService;

    public UserController(IAuthService authService)
    {
        _authService = authService;
    }

    // Login Endpoint
    [AllowAnonymous]
    [HttpPost("login")]
    public async Task<IActionResult> Login([FromBody] User loginUser)
    {
        var result = await _authService.LoginAsync(
            loginUser.UserName,
            loginUser.Password
        );

        if (result == null)
            return Unauthorized(new { message = "Invalid username or password" });

        return Ok(result);
    }
}
```

---

## 10. Apply Authorization Using [Authorize]

### Any Authenticated User

```csharp
[Authorize]
[HttpGet("profile")]
public IActionResult Profile()
{
    return Ok(new
    {
        userName = User.Identity.Name,
        role = User.FindFirst(ClaimTypes.Role)?.Value
    });
}
```

---

### Role‑Based Authorization

```csharp
[Authorize(Roles = "Admin")]
[HttpGet("admin-data")]
public IActionResult AdminData()
{
    return Ok("Only Admin can access this");
}
```
---
### Multiple Role‑Based Authorization

```csharp
[Authorize(Roles = "Admin,Staff")]
[HttpGet("admin-data")]
public IActionResult UserData()
{
    return Ok("Only Admin and Staff can access this");
}
```
---

### Allow Public Access

```csharp
[AllowAnonymous]
[HttpGet("public")]
public IActionResult Public()
{
    return Ok("No authentication required");
}
```

---

## 11. Enable JWT in Swagger

Add in Program.cs:

```csharp
builder.Services.AddSwaggerGen(options =>
{
    options.AddSecurityDefinition("Bearer", new OpenApiSecurityScheme
    {
        Name = "Authorization",
        Type = SecuritySchemeType.Http,
        Scheme = "Bearer",
        BearerFormat = "JWT",
        In = ParameterLocation.Header,
        Description = "Enter 'Bearer {token}'"
    });

    options.AddSecurityRequirement(new OpenApiSecurityRequirement
    {
        {
            new OpenApiSecurityScheme
            {
                Reference = new OpenApiReference
                {
                    Type = ReferenceType.SecurityScheme,
                    Id = "Bearer"
                }
            },
            new string[] {}
        }
    });
});
```  
