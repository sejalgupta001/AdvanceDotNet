# Lab 8: Create API – I (Roles API & User Management API)

## Task 1: Understanding HTTP Verbs

### What Are HTTP Verbs?

HTTP verbs (also called HTTP methods) tell the server what action you want to perform on a resource. Think of them as commands you send to the server.

| HTTP Verb  | What It Does                     | Real-World Example              | Database Operation    |
| ---------- | -------------------------------- | ------------------------------- | --------------------- |
| **GET**    | Gets/reads data from server      | View a list of products         | SELECT (read)         |
| **POST**   | Creates new data on server       | Add a new product               | INSERT (create)       |
| **PUT**    | Updates existing data completely | Update all details of a product | UPDATE (edit)         |
| **DELETE** | Removes data from server         | Delete a product                | DELETE (remove)       |
| **PATCH**  | Updates part of existing data    | Update only product price       | UPDATE (partial edit) |

---

## Task 2: Routing Basics

### Controller-Level Routing

```csharp
[ApiController]
[Route("api/[controller]")]
public class RolesController : ControllerBase
{
}
```

- `[controller]` is replaced with controller name (without _Controller_)
- `RolesController` → `api/roles`

### Method-Level Routing

```csharp
[HttpGet]
public IActionResult GetAll()
```

```csharp
[HttpGet("{id}")]
public IActionResult GetById(int id)
```

> **Note:** Route constraints are intentionally **not used** in Lab 8. They are covered in **Lab 11 – Routing and Constraints**.

---

## Task 3: Parameter Binding

These attributes tell ASP.NET where to get the data from:

| Attribute      | Where Data Comes From    | Example URL/Data                      | Method Usage                             |
| -------------- | ------------------------ | ------------------------------------- | ---------------------------------------- |
| `[FromRoute]`  | From URL path            | `api/roles/5` → id=5                  | `GetById([FromRoute] int id)`            |
| `[FromQuery]`  | From URL query string    | `api/roles?name=admin` → name="admin" | `Search([FromQuery] string name)`        |
| `[FromBody]`   | From request body (JSON) | `{ "name": "Admin" }`                 | `Create([FromBody] Role role)`           |
| `[FromHeader]` | From HTTP headers        | `Authorization: Bearer token`         | `Get([FromHeader] string authorization)` |
| `[FromForm]`   | From form data           | Form submission                       | `Upload([FromForm] IFormFile file)`      |

**Note:** ASP.NET Core automatically binds parameters, so these attributes are optional unless explicitly needed.

---

## Task 4: HTTP Status Codes and Response Methods

### Common Status Codes in APIs

| Status Code | Name                  | When to Use                    | .NET Method                                  |
| ----------- | --------------------- | ------------------------------ | -------------------------------------------- |
| **200**     | OK                    | Success - operation completed  | `Ok(data)`                                   |
| **201**     | Created               | Resource created successfully  | `CreatedAtAction(action, routeValues, data)` |
| **204**     | No Content            | Success - no data to return    | `NoContent()`                                |
| **400**     | Bad Request           | Validation failed or bad input | `BadRequest(message)`                        |
| **404**     | Not Found             | Resource not found             | `NotFound(message)`                          |
| **500**     | Internal Server Error | Server-side error              | `StatusCode(500, message)`                   |

### Response Method Examples

| Method                  | Status Code | Example Usage                                            |
| ----------------------- | ----------- | -------------------------------------------------------- |
| `Ok(data)`              | 200         | `return Ok(users);`                                      |
| `Created(uri, data)`    | 201         | `return Created("", new { message = "Created", data });` |
| `NoContent()`           | 204         | `return NoContent();`                                    |
| `BadRequest(msg)`       | 400         | `return BadRequest(new { message = "Email exists" });`   |
| `NotFound(msg)`         | 404         | `return NotFound(new { message = "Not found" });`        |
| `StatusCode(code, msg)` | Custom      | `return StatusCode(500, new { message = "Error" });`     |

**Note:** We use `Created("", data)` as a simple alternative to `CreatedAtAction()` for easier implementation.

## Task 5: Async Method Comparison

| Operation        | Sync Method                  | Async Method                            |
| ---------------- | ---------------------------- | --------------------------------------- |
| Get all          | `_db.Users.ToList()`         | `await _db.Users.ToListAsync()`         |
| Find by ID       | `_db.Users.Find(id)`         | `await _db.Users.FindAsync(id)`         |
| First or default | `_db.Users.FirstOrDefault()` | `await _db.Users.FirstOrDefaultAsync()` |
| Any check        | `_db.Users.Any(x => ...)`    | `await _db.Users.AnyAsync(x => ...)`    |
| Save changes     | `_db.SaveChanges()`          | `await _db.SaveChangesAsync()`          |

### Example: Sync vs Async

**Sync (Not Recommended):**

```csharp
[HttpGet]
public IActionResult GetAll()
{
    var users = _db.Users.ToList(); // Thread waits here
    return Ok(users);
}
```

**Async (Recommended):**

```csharp
[HttpGet]
public async Task<IActionResult> GetAll()
{
    var users = await _db.Users.ToListAsync(); // Thread is freed during DB operation
    return Ok(users);
}
```

### Key Points

✅ Always use `async/await` for database operations in production

✅ Add `async` keyword to method signature

✅ Return `Task<IActionResult>` instead of `IActionResult`

✅ Use `await` keyword before async methods

✅ Add `Async` suffix to EF Core methods (`ToListAsync`, `FindAsync`, etc.)

---

## Task 6: Roles API (CRUD)

### RolesController.cs

```csharp
using Microsoft.AspNetCore.Mvc;
using Microsoft.EntityFrameworkCore;

[ApiController]
[Route("api/[controller]")]
public class RolesController : ControllerBase
{
    private readonly AppDbContext _db;

    public RolesController(AppDbContext db)
    {
        _db = db;
    }

    // GET: api/roles
    [HttpGet]
    public async Task<IActionResult> GetAll()
    {
        try
        {
            var roles = await _db.Roles.ToListAsync();
            return Ok(roles);
        }
        catch (Exception ex)
        {
            return StatusCode(500, new { message = "Error getting roles", error = ex.Message });
        }
    }

    // GET: api/roles/5
    [HttpGet("{id}")]
    public async Task<IActionResult> GetById(int id)
    {
        try
        {
            var role = await _db.Roles.FindAsync(id);
            if (role == null)
                return NotFound(new { message = "Role not found" });

            return Ok(role);
        }
        catch (Exception ex)
        {
            return StatusCode(500, new { message = "Error getting role", error = ex.Message });
        }
    }

    // POST: api/roles
    [HttpPost]
    public async Task<IActionResult> Create(Role role)
    {
        try
        {
            if (!ModelState.IsValid)
                return BadRequest(ModelState);

            bool exists = await _db.Roles.AnyAsync(r => r.Name == role.Name);
            if (exists)
                return BadRequest(new { message = "Role already exists" });

            _db.Roles.Add(role);
            await _db.SaveChangesAsync();

            return Created("", role);
        }
        catch (Exception ex)
        {
            return StatusCode(500, new { message = "Error creating role", error = ex.Message });
        }
    }

    // PUT: api/roles/5
    [HttpPut("{id}")]
    public async Task<IActionResult> Update(int id, Role role)
    {
        try
        {
            if (id != role.Id)
                return BadRequest(new { message = "ID mismatch" });

            var existing = await _db.Roles.FindAsync(id);
            if (existing == null)
                return NotFound(new { message = "Role not found" });

            existing.Name = role.Name;
            await _db.SaveChangesAsync();

            return Ok(existing);
        }
        catch (Exception ex)
        {
            return StatusCode(500, new { message = "Error updating role", error = ex.Message });
        }
    }

    // DELETE: api/roles/5
    [HttpDelete("{id}")]
    public async Task<IActionResult> Delete(int id)
    {
        try
        {
            var role = await _db.Roles.FindAsync(id);
            if (role == null)
                return NotFound(new { message = "Role not found" });

            _db.Roles.Remove(role);
            await _db.SaveChangesAsync();

            return Ok(new { message = "Role deleted successfully" });
        }
        catch (Exception ex)
        {
            return StatusCode(500, new { message = "Error deleting role", error = ex.Message });
        }
    }
}
```

---

## Task 7: User Management API

### UsersController.cs

```csharp
using Microsoft.AspNetCore.Mvc;
using Microsoft.EntityFrameworkCore;

[ApiController]
[Route("api/[controller]")]
public class UsersController : ControllerBase
{
    private readonly AppDbContext _db;

    public UsersController(AppDbContext db)
    {
        _db = db;
    }

    // GET: api/users
    [HttpGet]
    public async Task<IActionResult> GetAll()
    {
        try
        {
            var users = await _db.Users
                .Include(u => u.Role)
                .Select(u => new
                {
                    u.Id,
                    u.Name,
                    u.Email,
                    Role = u.Role.Name
                })
                .ToListAsync();

            return Ok(users);
        }
        catch (Exception ex)
        {
            return StatusCode(500, new { message = "Error getting users", error = ex.Message });
        }
    }

    // POST: api/users
    [HttpPost]
    public async Task<IActionResult> Register(User user)
    {
        try
        {
            if (!ModelState.IsValid)
                return BadRequest(ModelState);

            if (string.IsNullOrWhiteSpace(user.Password) || user.Password.Length < 6)
                return BadRequest(new { message = "Password must be at least 6 characters" });

            bool emailExists = await _db.Users.AnyAsync(u => u.Email == user.Email);
            if (emailExists)
                return BadRequest(new { message = "Email already registered" });

            _db.Users.Add(user);
            await _db.SaveChangesAsync();

            return Created("", new { user.Id, user.Name, user.Email });
        }
        catch (Exception ex)
        {
            return StatusCode(500, new { message = "Error registering user", error = ex.Message });
        }
    }

    // PUT: api/users/5
    [HttpPut("{id}")]
    public async Task<IActionResult> Update(int id, User user)
    {
        try
        {
            var existing = await _db.Users.FindAsync(id);
            if (existing == null)
                return NotFound(new { message = "User not found" });

            existing.Name = user.Name;
            existing.Email = user.Email;
            existing.Password = user.Password;
            existing.RoleId = user.RoleId;

            await _db.SaveChangesAsync();
            return Ok(new { message = "User updated successfully" });
        }
        catch (Exception ex)
        {
            return StatusCode(500, new { message = "Error updating user", error = ex.Message });
        }
    }
}
```

---

## Common Errors Possibilities

### Error 1: Missing HTTP Verb Attribute

**Problem:** Method without an HTTP verb attribute won't be accessible as an API endpoint.

❌ **Example:**

```csharp
[Route("api/[controller]")]
public class RolesController : ControllerBase
{
    // Missing HTTP verb attribute - this won't work!
    public IActionResult GetAll()
    {
        return Ok("roles");
    }
}
```

**Error Message:**

```
HTTP Error 404 - Not Found
```

---

### Error 2: Route Conflicts - Same HTTP Verb with Same Route

**Problem:** Multiple methods with the same HTTP verb and same route pattern cause conflicts.

❌ **Example:**

```csharp
[Route("api/[controller]")]
public class RolesController : ControllerBase
{
    [HttpGet("{id}")]
    public IActionResult GetById(int id)
    {
        return Ok($"Role {id}");
    }

    [HttpGet("{roleId}")]  // Conflict! Same as above
    public IActionResult GetRoleDetails(int roleId)
    {
        return Ok($"Details {roleId}");
    }
}
```

**Error Message:**

```
AmbiguousMatchException: The request matched multiple endpoints.
```

---

### Error 3: Multiple Same HTTP Verbs Without Route Differentiation

**Problem:** Two or more methods with same HTTP verb on base route.

❌ **Example:**

```csharp
[Route("api/[controller]")]
public class RolesController : ControllerBase
{
    [HttpGet]  // Route: api/roles
    public IActionResult GetAll()
    {
        return Ok("All roles");
    }

    [HttpGet]  // Route: api/roles - CONFLICT!
    public IActionResult GetActive()
    {
        return Ok("Active roles");
    }
}
```

**Error Message:**

```
AmbiguousMatchException: The request matched multiple endpoints
```

---

### Error 4: Parameter Name Mismatch

**Problem:** Route parameter name doesn't match method parameter name.

❌ **Example:**

```csharp
[HttpGet("{id}")]
public IActionResult GetById(int roleId)  // Parameter name doesn't match route
{
    return Ok($"Role {roleId}");
}
```

**Error Message:**

```
HTTP 404 - Not Found (parameter won't bind correctly)
```

---
