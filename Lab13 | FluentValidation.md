# FluentValidation

## Overview

**FluentValidation** is a popular .NET library for building strongly‑typed, fluent, and expressive validation rules for models. It improves maintainability and clean separation between domain logic and validation concerns.

### Why Use FluentValidation

- Keeps validation logic **separate** from data models.
- Produces **readable and maintainable** rule syntax.
- Supports **advanced conditions, dependent rules, and async checks**.
- Integrates easily with ASP.NET Core APIs or MVC controllers.

---

## Installation

### ✅ **Modern Approach (.NET 6+ Recommended)**

```bash
dotnet add package FluentValidation
dotnet add package FluentValidation.DependencyInjectionExtensions
```

### ❌ **Deprecated Approach**

```bash
dotnet add package FluentValidation.AspNetCore
```

> ⚠️ **Important:** `FluentValidation.AspNetCore` is **deprecated and unsupported**. Use the modern manual validation approach instead.

---

## Built‑In Validation Rules (Reference Table)

| Category                       | Method / Syntax                                  | Example                                                              | Description                                                                         |
| ------------------------------ | ------------------------------------------------ | -------------------------------------------------------------------- | ----------------------------------------------------------------------------------- |
| **Null / Empty Checks**        | `NotNull()`                                      | `RuleFor(x => x.Name).NotNull()`                                     | Ensures property is not null.                                                       |
|                                | `NotEmpty()`                                     | `RuleFor(x => x.Name).NotEmpty()`                                    | Ensures property is not null/empty (`""` for strings, zero-length for collections). |
|                                | `Null()`                                         | `RuleFor(x => x.DeletedAt).Null()`                                   | Ensures value must be null.                                                         |
| **String Validation**          | `Length(min, max)`                               | `RuleFor(x => x.Name).Length(3, 50)`                                 | Validates that length is within range.                                              |
|                                | `MinimumLength(int)` / `MaximumLength(int)`      | `RuleFor(x => x.Password).MinimumLength(6)`                          | Checks minimum or maximum string length.                                            |
|                                | `Matches(regex)`                                 | `RuleFor(x => x.Phone).Matches(@"^\d{10}$")`                         | Validates format via regex.                                                         |
|                                | `EmailAddress()`                                 | `RuleFor(x => x.Email).EmailAddress()`                               | Validates that the string is a valid email address.                                 |
| **Numeric / Date Ranges**      | `GreaterThan(value)`                             | `RuleFor(x => x.Age).GreaterThan(18)`                                | Value must be greater than specified number.                                        |
|                                | `GreaterThanOrEqualTo(value)`                    | `RuleFor(x => x.StartDate).GreaterThanOrEqualTo(DateTime.Today)`     | Ensures date or number isn't less than given limit.                                 |
|                                | `LessThan(value)` / `LessThanOrEqualTo(value)`   | `RuleFor(x => x.EndDate).GreaterThan(x => x.StartDate)`              | Compares one property to another.                                                   |
| **Boolean Rules**              | `Equal(value)` / `NotEqual(value)`               | `RuleFor(x => x.IsActive).Equal(true)`                               | Matches specific values.                                                            |
| **Conditional Validation**     | `When(condition)` / `Unless(condition)`          | `When(x => x.IsApproved, () => RuleFor(x => x.Approver).NotEmpty())` | Applies rules conditionally.                                                        |
| **Collections**                | `ForEach(...)`                                   | `RuleForEach(x => x.Tags).NotEmpty()`                                | Applies rules per item in a collection.                                             |
| **Async or Custom Rules**      | `Must(predicate)` / `MustAsync(async predicate)` | `RuleFor(x => x).MustAsync(NoOverlapAsync)`                          | Allows custom or database‑backed validation logic.                                  |
| **Fluent Control**             | `Cascade(CascadeMode.Stop)`                      | Stops subsequent checks on first failure                             | Controls whether to continue after a failure.                                       |
| **Comparisons**                | `InclusiveBetween(min, max)`                     | `RuleFor(x => x.Quantity).InclusiveBetween(1, 100)`                  | Ensures value is within inclusive bounds.                                           |
| **Credit card / Regex / Enum** | `CreditCard()`, `Matches()`, `IsInEnum()`        | `RuleFor(x => x.Status).IsInEnum()`                                  | Specialized validators for format or enum validation.                               |
| **Custom Messages**            | `.WithMessage("text")`                           | `RuleFor(x => x.Name).NotEmpty().WithMessage("Name required")`       | Overrides the default system message.                                               |

---

## 🚀 **MODERN APPROACH (.NET 6+ - RECOMMENDED)**

### Model (DTO)

```csharp
public class UserDto
{
    public string Name { get; set; };
    public string Email { get; set; };
    public string Password { get; set; };
}
```

### Validator

```csharp
using FluentValidation;

public class UserValidator : AbstractValidator<UserDto>
{
    public UserValidator()
    {
        RuleFor(x => x.Name)
            .NotEmpty().WithMessage("Name is required.")
            .MaximumLength(50).WithMessage("Name cannot exceed 50 characters.");

        RuleFor(x => x.Email)
            .Cascade(CascadeMode.Stop)
            .NotEmpty().WithMessage("Email is required.")
            .EmailAddress().WithMessage("Invalid email format.");

        RuleFor(x => x.Password)
            .NotEmpty().WithMessage("Password is required.")
            .MinimumLength(6).WithMessage("Password must be at least 6 characters.");
    }
}
```

### Program.cs (Auto-register ALL validators)

```csharp
using FluentValidation.DependencyInjectionExtensions;

var builder = WebApplication.CreateBuilder(args);
builder.Services.AddControllers();

// ONE LINE registers ALL validators in the assembly
builder.Services.AddValidatorsFromAssemblyContaining<Program>();

var app = builder.Build();
app.MapControllers();
app.Run();
```

### Controller (Manual Validation)

```csharp
[ApiController]
[Route("api/[controller]")]
public class UsersController : ControllerBase
{
    private readonly IValidator<UserDto> _validator;

    public UsersController(IValidator<UserDto> validator)
    {
        _validator = validator;
    }

    [HttpPost]
    public async Task<IActionResult> Create([FromBody] UserDto dto)
    {
        var result = await _validator.ValidateAsync(dto);

        if (!result.IsValid)
        {
            return BadRequest(new
            {
                success = false,
                errors = result.Errors.Select(e => e.ErrorMessage)
            });
        }

        return Ok(new { success = true, message = "User created" });
    }
}
```

---

## 🗑️ **DEPRECATED APPROACH (FluentValidation.AspNetCore)**

### Program.cs (Legacy Auto-validation)

```csharp
using FluentValidation.AspNetCore;

var builder = WebApplication.CreateBuilder(args);
builder.Services.AddControllers()
    .AddFluentValidation(fv => fv.RegisterValidatorsFromAssemblyContaining<UserValidator>());
    // This enables AUTOMATIC validation (the "magic")
    // builder.Services.AddFluentValidationAutoValidation();

var app = builder.Build();
app.MapControllers();
app.Run();
```

### Controller (NO manual validation needed)

```csharp
[ApiController]
[Route("api/[controller]")]
public class UsersController : ControllerBase
{
    [HttpPost]
    public IActionResult Create([FromBody] UserDto dto)
    {
        // Validation happens AUTOMATICALLY before controller method
        // No need for IValidator injection or ValidateAsync calls

        return Ok(new { success = true, message = "User created" });
    }
}
```
