# LAB 7 – Database Design with EF Core (Code-First)

# Entity Framework Core (EF Core) Setup Guide

This guide walks through EF Core basics and a practical setup using a leave management system with `User`, `Role`, `LeaveType`, `LeaveRequest`, and `LeaveBalance` models.

## What Is EF Core and Why Use It?

EF Core is Microsoft's modern object-relational mapper (ORM) for .NET. It connects your C\# classes to database tables so you can work with data using LINQ and strongly typed models instead of writing raw SQL.

**Why use EF Core:**

- Productivity: Query with LINQ and track changes automatically.
- Easy to maintain: Keep data access logic in one place; models are strongly typed.
- Works with many databases: SQL Server, SQLite, PostgreSQL, MySQL, etc.
- Version control your database: Track schema changes alongside code.

## What is ORM (Object-Relational Mapping)

ORM is a technique that maps object-oriented classes to relational database structures.

| Database Concept | Object-Oriented Concept |
| :--------------- | :---------------------- |
| Table            | Class                   |
| Row              | Object                  |
| Column           | Property                |
| Primary Key      | Object Identifier       |
| Foreign Key      | Object Reference        |

ORM enables developers to work with data using C\# objects instead of SQL queries.

## EF Core Development Approaches

### Code-First vs DB-First

| Criteria       | Code-First                                                | DB-First                                                                    |
| :------------- | :-------------------------------------------------------- | :-------------------------------------------------------------------------- |
| Starting point | Write C\# models and set up schema via code.              | Start from an existing database schema.                                     |
| Best for       | New projects or projects where schema changes with code.  | Working with existing or shared databases.                                  |
| How it works   | Create/update models → add migrations → update database.  | Create entities and context from the database → keep schema as main source. |
| Main tool      | Uses migration commands (`dotnet ef migrations add ...`). | Uses `dotnet ef dbcontext scaffold` (or PMC `Scaffold-DbContext`).          |

## EF Core (Code-First) Project Steps:

1. Create ASP.NET Core Web API Project
2. Install EF Core Packages
3. Create Entity Models (`User`, `Role`, `LeaveType`, `LeaveRequest`, `LeaveBalance`)
4. Create `AppDbContext`
5. Configure Connection String
6. Register DbContext in `Program.cs`
7. Create Initial Migration
8. Apply Migration to Database
9. Modify Models and Update Schema
10. Use `Up()`/`Down()` in migrations to understand changes and rollbacks.
11. Seed Initial Data via `HasData` or add data via `DbContext`.
12. Create API Controllers and Endpoints -- NEXT LAB

## Project Setup

### 1. Create ASP.NET Core Web API Project

1. Open Visual Studio
2. Create **ASP.NET Core Web API** project
3. Project Name: `LeaveManagementAPI`
4. Framework: **.NET 8.0**

**Folder Structure After Creation:**

```
LeaveManagementAPI/
├── Controllers/
├── Data/
├── Models/
├── Program.cs
├── appsettings.json
└── LeaveManagementAPI.csproj
```

### 2. Install EF Core Packages

#### Required NuGet Packages

| Package Name                              | Purpose                      | Why It Is Needed                                                                      |
| :---------------------------------------- | :--------------------------- | :------------------------------------------------------------------------------------ |
| `Microsoft.EntityFrameworkCore`           | Core EF Core runtime library | Enables ORM features like DbContext, DbSet, LINQ queries, and LINQ-based data access  |
| `Microsoft.EntityFrameworkCore.Design`    | Design-time support          | Required for migrations, scaffolding, and other design-time services                  |
| `Microsoft.EntityFrameworkCore.SqlServer` | SQL Server provider          | Allows EF Core to communicate with SQL Server databases (replace if using another DB) |
| `Microsoft.EntityFrameworkCore.Tools`     | EF Core CLI tools            | Enables EF Core commands for migrations and database updates                          |

**Install via CLI:**

```powershell
# From your project directory
dotnet add package Microsoft.EntityFrameworkCore
dotnet add package Microsoft.EntityFrameworkCore.Design
dotnet add package Microsoft.EntityFrameworkCore.SqlServer
dotnet add package Microsoft.EntityFrameworkCore.Tools
```

**Install via Visual Studio Package Manager UI:**

1. Right-click project → **Manage NuGet Packages**
2. Browse tab → Search package name → **Install**

**Install via Package Manager Console:**

```
Tools → NuGet Package Manager → Package Manager Console
Install-Package Microsoft.EntityFrameworkCore
Install-Package Microsoft.EntityFrameworkCore.SqlServer
```

### 3. Create Entity Models

**Create `Models/` folder** and add these complete models:
# Schema view
## **Roles**

| Column Name | Data Type    | Constraints      |
| ----------- | ------------ | ---------------- |
| Id          | int          | PK, Identity     |
| Name        | nvarchar(50) | Not Null, Unique |

---

## **Users**

| Column Name | Data Type     | Constraints              |
| ----------- | ------------- | ------------------------ |
| Id          | int           | PK, Identity             |
| Name        | nvarchar(100) | Not Null                 |
| Email       | nvarchar(100) | Not Null, Unique         |
| Password    | nvarchar(255) | Not Null                 |
| PhoneNumber | nvarchar(20)  | Null                     |
| RoleId      | int           | FK → Roles(Id), Not Null |

---

## **LeaveTypes**

| Column Name | Data Type     | Constraints      |
| ----------- | ------------- | ---------------- |
| Id          | int           | PK, Identity     |
| Name        | nvarchar(100) | Not Null, Unique |
| Description | nvarchar(500) | Null             |

---

## **LeaveRequests**

| Column Name | Data Type      | Constraints                              |
| ----------- | -------------- | ---------------------------------------- |
| Id          | int            | PK, Identity                             |
| UserId      | int            | FK → Users(Id), Not Null                 |
| LeaveTypeId | int            | FK → LeaveTypes(Id), Not Null            |
| StartDate   | date           | Not Null                                 |
| EndDate     | date           | Not Null                                 |
| Days        | int            | Not Null                                 |
| Reason      | nvarchar(1000) | Null                                     |
| Status      | nvarchar(20)   | Not Null (Pending / Approved / Rejected) |

---

## **LeaveBalances**

| Column Name   | Data Type | Constraints                        |
| ------------- | --------- | ---------------------------------- |
| Id            | int       | PK, Identity                       |
| UserId        | int       | FK → Users(Id), Not Null           |
| LeaveTypeId   | int       | FK → LeaveTypes(Id), Not Null      |
| TotalDays     | int       | Not Null                           |
| UsedDays      | int       | Not Null                           |
| RemainingDays | int       | Not Null                           |
| Year          | int       | Not Null                           |
| *(Composite)* | —         | Unique (UserId, LeaveTypeId, Year) |

---

## **Relationship Summary**

| Relationship              | Type        |
| ------------------------- | ----------- |
| Role → Users              | One-to-Many |
| User → LeaveRequests      | One-to-Many |
| LeaveType → LeaveRequests | One-to-Many |
| User → LeaveBalances      | One-to-Many |
| LeaveType → LeaveBalances | One-to-Many |
---

#### Relationship Types (How Tables Connect)

**One-to-many (`Role` ↔ `User`)**

- One role can have many users
- Foreign key on User table

How it looks in models:

```csharp
// Role (Parent - One side)
public class Role
{
    public int Id { get; set; }
    public string Name { get; set; } = string.Empty;

    // Navigation property: Collection of related Users
    public ICollection<User> Users { get; set; } = new List<User>();
}

// User (Child - Many side)
public class User
{
    public int Id { get; set; }
    public string Name { get; set; } = string.Empty;

    // Foreign Key property
    public int RoleId { get; set; }

    // Navigation property: Reference to parent Role
    public Role? Role { get; set; }
}
```

**Key Pattern Summary:**

| Side             | Contains                                                                 | Example                                                                |
| :--------------- | :----------------------------------------------------------------------- | :--------------------------------------------------------------------- |
| **Parent (One)** | `ICollection<ChildEntity>` property                                      | `public ICollection<User> Users { get; set; }`                         |
| **Child (Many)** | Foreign Key property (`int ParentId`) + Link property (`Parent? Parent`) | `public int RoleId { get; set; }`<br>`public Role? Role { get; set; }` |

Create your models and context.

Make sure to import `System.ComponentModel.DataAnnotations` for validation attributes. Add `System.ComponentModel.DataAnnotations.Schema` when you use `[ForeignKey]`.

### Data Annotations for Keys and Links

**Primary Key Annotations:**

- `[Key]` - Marks a property as the primary key
- By default, properties named `Id` or `<TypeName>Id` are automatically treated as primary keys

**Foreign Key Annotations:**

- `[ForeignKey("NavigationPropertyName")]` - Clearly defines a foreign key link
- Can be placed on either the FK property or the link property

**Unique Key Annotations:**

- `[Index(IsUnique = true)]` - Creates a unique index (EF Core 5.0+)
- For older versions, set up via Fluent API in `OnModelCreating`

#### Complete Models with Data Annotations

**Add to top of each model file:**

```csharp
using System.ComponentModel.DataAnnotations;
using System.ComponentModel.DataAnnotations.Schema;
```

**User.cs:**

```csharp
public class User
{
    [Key]
    public int Id { get; set; }

    [Required, MaxLength(100)]
    public string Name { get; set; } = string.Empty;

    [Required, MaxLength(100), EmailAddress]
    [Index(IsUnique = true)]
    public string Email { get; set; } = string.Empty;

    [Required, MaxLength(255)]
    public string Password { get; set; } = string.Empty;

    [MaxLength(20)]
    public string? PhoneNumber { get; set; }

    [Required]
    [ForeignKey("Role")]
    public int RoleId { get; set; }
    public Role? Role { get; set; }

    public ICollection<LeaveRequest> LeaveRequests { get; set; } = new List<LeaveRequest>();
    public ICollection<LeaveBalance> LeaveBalances { get; set; } = new List<LeaveBalance>();
}
```

**Role.cs:**

```csharp
public class Role
{
    [Key]
    public int Id { get; set; }

    [Required, MaxLength(50)]
    [Index(IsUnique = true)]
    public string Name { get; set; } = string.Empty;

    public ICollection<User> Users { get; set; } = new List<User>();
}
```

**LeaveType.cs:**

```csharp
public class LeaveType
{
    [Key]
    public int Id { get; set; }

    [Required, MaxLength(100)]
    [Index(IsUnique = true)]
    public string Name { get; set; } = string.Empty;

    [MaxLength(500)]
    public string? Description { get; set; }

    public ICollection<LeaveRequest> LeaveRequests { get; set; } = new List<LeaveRequest>();
    public ICollection<LeaveBalance> LeaveBalances { get; set; } = new List<LeaveBalance>();
}
```

**LeaveRequest.cs:**

```csharp
public class LeaveRequest
{
    [Key]
    public int Id { get; set; }

    [Required]
    [ForeignKey("User")]
    public int UserId { get; set; }
    public User? User { get; set; }

    [Required]
    [ForeignKey("LeaveType")]
    public int LeaveTypeId { get; set; }
    public LeaveType? LeaveType { get; set; }

    [Required, DataType(DataType.Date)]
    public DateTime StartDate { get; set; }

    [Required, DataType(DataType.Date)]
    public DateTime EndDate { get; set; }

    [Required]
    public int Days { get; set; }

    [MaxLength(1000)]
    public string? Reason { get; set; }

    [Required, MaxLength(20)]
    public string Status { get; set; } = "Pending"; // Pending, Approved, Rejected
}
```

**LeaveBalance.cs:**

```csharp
public class LeaveBalance
{
    [Key]
    public int Id { get; set; }

    [Required]
    [ForeignKey("User")]
    public int UserId { get; set; }
    public User? User { get; set; }

    [Required]
    [ForeignKey("LeaveType")]
    public int LeaveTypeId { get; set; }
    public LeaveType? LeaveType { get; set; }

    [Required]
    public int TotalDays { get; set; }

    [Required]
    public int UsedDays { get; set; }

    [Required]
    public int RemainingDays { get; set; }

    [Required]
    public int Year { get; set; }
}
```

#### Primary Key and Foreign Key Mappings

| Entity       | Primary Key | Foreign Keys                                           | Unique Keys |
| :----------- | :---------- | :----------------------------------------------------- | :---------- |
| User         | `Id`        | `RoleId` → `Role.Id`                                   | `Email`     |
| Role         | `Id`        | None                                                   | `Name`      |
| LeaveType    | `Id`        | None                                                   | `Name`      |
| LeaveRequest | `Id`        | `UserId` → `User.Id`<br>`LeaveTypeId` → `LeaveType.Id` | None        |
| LeaveBalance | `Id`        | `UserId` → `User.Id`<br>`LeaveTypeId` → `LeaveType.Id` | None        |

### 4. Create `AppDbContext`

**Create `Data/AppDbContext.cs`:**

```csharp
using Microsoft.EntityFrameworkCore;
using LeaveManagementAPI.Models; // Adjust namespace as needed

namespace LeaveManagementAPI.Data
{
    public class AppDbContext : DbContext
    {
        public AppDbContext(DbContextOptions<AppDbContext> options)
            : base(options) { }

        public DbSet<User> Users { get; set; }
        public DbSet<Role> Roles { get; set; }
        public DbSet<LeaveType> LeaveTypes { get; set; }
        public DbSet<LeaveRequest> LeaveRequests { get; set; }
        public DbSet<LeaveBalance> LeaveBalances { get; set; }

        protected override void OnModelCreating(ModelBuilder modelBuilder)
        {
            // Fluent API examples (optional - Data Annotations already handle most)
            modelBuilder.Entity<User>()
                .HasIndex(u => u.Email)
                .IsUnique();

            modelBuilder.Entity<LeaveBalance>()
                .HasIndex(lb => new { lb.UserId, lb.LeaveTypeId, lb.Year })
                .IsUnique();

            // Data Seeding (Section 11)
            SeedData(modelBuilder);
        }

        private static void SeedData(ModelBuilder modelBuilder)
        {
            // Seed Roles
            modelBuilder.Entity<Role>().HasData(
                new Role { Id = 1, Name = "Admin" },
                new Role { Id = 2, Name = "Employee" }
            );

            // Seed Leave Types
            modelBuilder.Entity<LeaveType>().HasData(
                new LeaveType { Id = 1, Name = "Vacation", Description = "Annual paid vacation" },
                new LeaveType { Id = 2, Name = "Sick", Description = "Paid sick leave" }
            );

            // Seed Users (requires Roles to exist)
            modelBuilder.Entity<User>().HasData(
                new User
                {
                    Id = 1, Name = "John Doe", Email = "john@company.com",
                    Password = "hashedpassword123", RoleId = 2, PhoneNumber = "123-456-7890"
                }
            );
        }
    }
}
```

#### DbContext vs DbSet

| **Aspect**               | **DbContext**                                                                                          | **DbSet<T>**                                                           |
| ------------------------ | ------------------------------------------------------------------------------------------------------ | ---------------------------------------------------------------------- |
| **Role**                 | Main EF Core manager that handles database connection and tracks changes                               | Represents a table (or view) for a specific entity type                |
| **Main Jobs**            | Configures models, tracks entity states, manages transactions, and saves changes using `SaveChanges()` | Executes LINQ queries and performs CRUD operations on a specific table |
| **Scope**                | Represents the entire database                                                                         | Represents a single table                                              |
| **Contains**             | One or more `DbSet<T>` properties                                                                      | Collection of entities of type `T`                                     |
| **Common Usage**         | Create a custom context class (e.g., `AppDbContext`) inheriting from `DbContext`                       | Access tables using `context.Users`, `context.LeaveRequests`, etc.     |
| **One-Liner** | Acts as a bridge between the application and the database and manages all entities                     | Represents a collection of entities corresponding to a database table  |


### 5. Configure Connection String

**appsettings.json:**

```json
{
  "ConnectionStrings": {
    "Default": "Server=.;Database=LeaveManagementDb;Trusted_Connection=True;TrustServerCertificate=True;MultipleActiveResultSets=true"
  },
  "Logging": {
    "LogLevel": {
      "Default": "Information",
      "Microsoft.AspNetCore": "Warning"
    }
  },
  "AllowedHosts": "*"
}
```

**Note:** update for your SQL Server instance.

### 6. Register DbContext in `Program.cs`

```csharp
using Microsoft.EntityFrameworkCore;
using LeaveManagementAPI.Data;

var builder = WebApplication.CreateBuilder(args);

// Add services to the container
builder.Services.AddControllers();
builder.Services.AddEndpointsApiExplorer();
builder.Services.AddSwaggerGen();

// Register DbContext
builder.Services.AddDbContext<AppDbContext>(options =>
    options.UseSqlServer(builder.Configuration.GetConnectionString("Default")));

var app = builder.Build();

// Configure the HTTP request pipeline
if (app.Environment.IsDevelopment())
{
    app.UseSwagger();
    app.UseSwaggerUI();
}

app.UseHttpsRedirection();
app.UseAuthorization();
app.MapControllers();

app.Run();
```

Here’s a clear, **step-by-step version** for the `### 7. Create Initial Migration` section:

---

### 7. Create Initial Migration

#### Steps to Run Migration

1. **Open terminal/command prompt** in the **project root folder** (where `.csproj` is located).
   OR
   **Open Package Manager Console (PMC)** in Visual Studio:

   * Go to **Tools → NuGet Package Manager → Package Manager Console**
   * Make sure **Default Project** is set to your API project.

2. **Create a new migration** with a descriptive name:

```powershell
dotnet ef migrations add InitialCreate
```

OR in PMC:

```powershell
Add-Migration InitialCreate
```

### 8. Apply Migration to Database

1. **Apply the migration to the database**:

```powershell
dotnet ef database update
```

OR in PMC:

```powershell
Update-Database
```

2. **Check generated migration code** (in `Migrations/` folder) to review `Up()` and `Down()` methods.

3. **Verify database**: Open SQL Server Management Studio (SSMS) or your DB tool to see the created tables and columns.

---

#### Migrations Commands Reference

**CLI Commands:**

| Action                | CLI Command                               |
| :-------------------- | :---------------------------------------- |
| Create migration      | `dotnet ef migrations add InitialCreate`  |
| Apply migrations      | `dotnet ef database update`               |
| List migrations       | `dotnet ef migrations list`               |
| Remove last migration | `dotnet ef migrations remove`             |
| Revert to migration   | `dotnet ef database update InitialCreate` |
| Generate SQL script   | `dotnet ef migrations script`             |

**Package Manager Console (Visual Studio):**

| Action                | PMC Command                   |
| :-------------------- | :---------------------------- |
| Create migration      | `Add-Migration InitialCreate` |
| Apply migrations      | `Update-Database`             |
| List migrations       | `Get-Migration`               |
| Remove last migration | `Remove-Migration`            |

#### Up/Down Methods Explained

| Method   | When Called               | Content Example                              | CLI Trigger                            |
| :------- | :------------------------ | :------------------------------------------- | :------------------------------------- |
| `Up()`   | Migration applied forward | Create tables, add columns, insert seed data | `dotnet ef database update`            |
| `Down()` | Migration rolled back     | Drop tables, remove columns                  | `dotnet ef database update <Previous>` |

**Example Generated Migration:**

```csharp
public partial class InitialCreate : Migration
{
    protected override void Up(MigrationBuilder migrationBuilder)
    {
        migrationBuilder.CreateTable(
            name: "Roles",
            columns: table => new
            {
                Id = table.Column<int>(type: "int", nullable: false)
                    .Annotation("SqlServer:Identity", "1, 1"),
                Name = table.Column<string>(type: "nvarchar(50)", maxLength: 50, nullable: false)
            },
            constraints: table =>
            {
                table.PrimaryKey("PK_Roles", x => x.Id);
            });
        // ... more tables
    }

    protected override void Down(MigrationBuilder migrationBuilder)
    {
        migrationBuilder.DropTable(name: "Users");
        migrationBuilder.DropTable(name: "LeaveTypes");
        migrationBuilder.DropTable(name: "Roles");
    }
}
```

### 9. Modify Models and Update Schema

**Example: Add PhoneNumber to User**

1. **Add property to User model:**

```csharp
[MaxLength(20)]
public string? PhoneNumber { get; set; }
```

2. **Create and apply migration:**

```powershell
dotnet ef migrations add AddUserPhoneNumber
dotnet ef database update
```

**Generated Migration:**

```csharp
protected override void Up(MigrationBuilder migrationBuilder)
{
    migrationBuilder.AddColumn<string>(
        name: "PhoneNumber",
        table: "Users",
        type: "nvarchar(20)",
        maxLength: 20,
        nullable: true);
}

protected override void Down(MigrationBuilder migrationBuilder)
{
    migrationBuilder.DropColumn(name: "PhoneNumber", table: "Users");
}
```

### 10. Understanding Up()/Down() Methods

**Workflow:**

1. Change models → `dotnet ef migrations add <Name>`
2. **Review** `Up()` (forward changes) and `Down()` (rollback changes)
3. **Edit** if needed (ensure `Down()` perfectly reverses `Up()`)
4. `dotnet ef database update`

### 11. Seed Initial Data

**Method 1: HasData() in OnModelCreating** (Already added above):

```csharp
protected override void OnModelCreating(ModelBuilder modelBuilder)
{
    modelBuilder.Entity<Role>().HasData(
        new Role { Id = 1, Name = "Admin" },
        new Role { Id = 2, Name = "Employee" }
    );
}
```

**Method 2: Seed via DbContext (Create Seeder Class):**

```csharp
public static class DbSeeder
{
    public static void Seed(AppDbContext context)
    {
        if (!context.Roles.Any())
        {
            context.Roles.AddRange(
                new Role { Name = "Admin" },
                new Role { Name = "Employee" }
            );
            context.SaveChanges();
        }
    }
}
```

**Call in Program.cs:**

```csharp
using (var scope = app.Services.CreateScope())
{
    var context = scope.ServiceProvider.GetRequiredService<AppDbContext>();
    DbSeeder.Seed(context);
}
```

## Common Troubleshooting

| Issue                          | Solution                                                            |
| :----------------------------- | :------------------------------------------------------------------ |
| `No DbContext found`           | Ensure `Microsoft.EntityFrameworkCore.Design` installed, restart VS |
| `Unable to create migration`   | Set startup project correctly, check connection string              |
| `Foreign key constraint error` | Check `OnDelete(DeleteBehavior.Restrict)` in Fluent API             |
| `Index already exists`         | Remove migration, update model, recreate migration                  |
