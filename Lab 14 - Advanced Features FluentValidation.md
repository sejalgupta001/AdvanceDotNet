## Lab 14 - Advanced Features FluentValidation(Both Approaches)

### 1. Async Business Rules (Database Checks)

```csharp
public class LeaveRequestValidator : AbstractValidator<LeaveRequestCreateDto>
{
    private readonly AppDbContext _db;

    public LeaveRequestValidator(AppDbContext db)
    {
        _db = db;

        RuleFor(x => x)
            .MustAsync(async (model, ct) =>
            {
                return !await _db.LeaveRequests.AnyAsync(l =>
                    l.UserId == model.UserId &&
                    model.StartDate <= l.EndDate &&
                    model.EndDate >= l.StartDate, ct);
            })
            .WithMessage("Overlapping leave exists.");
    }
}
```

### 2. Conditional Rules

```csharp
RuleFor(x => x.ManagerComments)
    .NotEmpty()
    .When(x => x.IsRejected)
    .WithMessage("Comments required when rejected.");
```

### 3. Collection Validation

```csharp
RuleFor(x => x.Items)
    .NotEmpty()
    .ForEach(item =>
    {
        item.RuleFor(i => i.Quantity).GreaterThan(0);
        item.RuleFor(i => i.Price).GreaterThan(0);
    });
```

### 4. Cascade Mode (Stop on First Failure)

```csharp
RuleFor(x => x.Email)
    .Cascade(CascadeMode.Stop)
    .NotEmpty()
    .EmailAddress();
```

---
