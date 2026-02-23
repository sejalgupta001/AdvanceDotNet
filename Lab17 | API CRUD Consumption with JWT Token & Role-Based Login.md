# ASP.NET MVC – API CRUD Consumption with JWT Token & Role-Based Login

## Overall Flow

1. User logs in from MVC UI
2. MVC calls Web API Login endpoint
3. API returns JWT Token + Role
4. MVC stores Token & Role in Session
5. MVC uses Token to call secured API CRUD endpoints
6. API validates token & role
7. MVC displays data (Staff, Department, Dashboard)

---

## 1. Configuration Setup (Before Coding)

### appsettings.json

```csharp
{
 "Logging": {
   "LogLevel": {
     "Default": "Information",
     "Microsoft.AspNetCore": "Warning"
   }
 },
"AllowedHosts": "*",
"ApiSettings": {
  "BaseUrl": "https://localhost:7005/"
}

}
```

---

### Program.cs Configuration

**Why this is required:**

- MVC needs sessions to store JWT token
- `HttpClientFactory` is required for API calls

**Key concepts:**

- `AddSession()` → Enables session storage
- `UseSession()` → Activates session middleware
- `AddHttpClient()` → Safe and reusable HttpClient

**Program.cs**

```csharp
using Microsoft.AspNetCore.Authentication.Cookies; // only if you later decide to use SignInAsync
using System.Text.Json;

var builder = WebApplication.CreateBuilder(args);

builder.Services.AddControllersWithViews();
builder.Services.AddHttpContextAccessor();
builder.Services.AddSession();
builder.Services.AddHttpClient(); // Registers IHttpClientFactory

var app = builder.Build();

app.UseSession();
app.UseStaticFiles();
app.UseRouting();
app.UseAuthorization();

app.MapControllerRoute(
    name: "default",
    pattern: "{controller=Account}/{action=Login}/{id?}");

app.Run();
```

### Models
```csharp
  // LoginRequest.cs
  public class LoginRequest
  {
      public string Email { get; set; } = string.Empty;
      public string Password { get; set; } = string.Empty;
  }

  // LoginResponse.cs (matches your API return)
  public class LoginResponse
  {
      public string Token { get; set; } = string.Empty;
      public string Role { get; set; } = string.Empty;
      public string User { get; set; } = string.Empty;
  }

  public class ServiceResponse<T>
  {
      public bool Success { get; set; }
      public string Message { get; set; }
      public T Data { get; set; } // This will hold your List<Staff>
  }
  // Department.cs
  public class Department
  {
      public int? DepartmentID { get; set; }
      public string DepartmentName { get; set; } = string.Empty;
  }

  // Staff.cs
  public class Staff
  {
      public int? StaffID { get; set; }
      public int DepartmentID { get; set; }
      public int RoleID { get; set; }
      public string DepartmentName { get; set; } = string.Empty;

      public string StaffName { get; set; } = string.Empty;
      public string MobileNo { get; set; } = string.Empty;
      public string Password { get; set; } = string.Empty;
      public string EmailAddress { get; set; } = string.Empty;
      public bool IsActive { get; set; }
      public string? Remarks { get; set; }
  }

  // DashboardViewModel.cs (optional but recommended)
  public class DashboardViewModel
  {
      public List<Department> Departments { get; set; } = new();
      public List<Staff> Staff { get; set; } = new();
  }

```


---

## 2. Login Module (AccountController)

### Login View (GET)

```html
<div class="container mt-5">
  <div class="row justify-content-center">
    <div class="col-md-4 card shadow p-4">
      <h2 class="text-center mb-4">Minutes of Meeting</h2>

      <form asp-action="Login" method="post">
        <div class="mb-3">
          <label>Email Address</label>
          <input type="email" name="email" class="form-control" required />
        </div>
        <div class="mb-3">
          <label>Password</label>
          <input
            type="password"
            name="password"
            class="form-control"
            required
          />
        </div>
        <button type="submit" class="btn btn-primary w-100">Sign In</button>
      </form>

      @if (ViewBag.Error != null) {
      <div class="alert alert-danger mt-3">@ViewBag.Error</div>
      }
    </div>
  </div>
</div>
```

### Login Action (POST)

**What happens step-by-step:**

1. User submits email & password
2. MVC creates **Form URL Encoded data**
3. MVC sends POST request to API `/api/Auth/login`
4. API validates credentials
5. API returns:
   - JWT Token
   - User Role
   - User Name
6. MVC stores data in **Session**
7. Redirects to Dashboard

**Why `[FromForm]` matters:**

- API expects form-data, not JSON

**Session Keys Used:**

- `JWToken` → Used for Authorization header
- `UserRole` → Used for role-based UI
- `UserName` → Display purpose

---

### Logout

**Purpose:**

- Clear all session data
- Force re-login
- JWT logout is client-side (token removal)

**AccountController**

```csharp
using Consume_202526.Models;
using Microsoft.AspNetCore.Mvc;
using System.Net.Http.Headers;

namespace Consume_202526.Controllers
{
    public class AccountController : Controller
    {
        private readonly IHttpClientFactory _factory;
        private readonly string _baseUrl;

        public AccountController(IHttpClientFactory factory, IConfiguration config)
        {
            _factory = factory;
            _baseUrl = config["ApiSettings:BaseUrl"];
        }

        public IActionResult Login()
        {
            return View();
        }

        [HttpPost]
        public async Task<IActionResult> Login(string email, string password)
        {
            try
            {
                var client = _factory.CreateClient();

                // IMPORTANT: This creates the "Form" data that [FromForm] expects
                var dict = new Dictionary<string, string>();
                dict.Add("email", email);
                dict.Add("password", password);

                var content = new FormUrlEncodedContent(dict);

                // Call the API
                var response = await client.PostAsync(_baseUrl + "api/Auth/login", content);

                if (response.IsSuccessStatusCode)
                {
                    // Map the JSON response to our model
                    var result = await response.Content.ReadFromJsonAsync<LoginResponse>();

                    if (result != null)
                    {
                        // Save to Session
                        HttpContext.Session.SetString("JWToken", result.Token);
                        HttpContext.Session.SetString("UserRole", result.Role);
                        HttpContext.Session.SetString("UserName", result.User);

                        return RedirectToAction("Index", "Dashboard");
                    }
                }

                ViewBag.Error = "Invalid Login - API returned " + response.StatusCode;
                return View();
            }
            catch (Exception ex)
            {
                ViewBag.Error = "Exception: " + ex.Message;
                return View();
            }
        }

        public IActionResult Logout()
        {
            HttpContext.Session.Clear();
            return RedirectToAction("Login");
        }
    }
}
```

---

## 3. Token Handling Concept (Very Important)

### Why Session is Used?

- MVC is **stateful**
- Token must be reused for every API request

### How Token is Sent to API?

```
Authorization: Bearer <JWT_TOKEN>
```

### Where this happens?

- In a **common helper method** (`GetClient()`)

```csharp
 private HttpClient GetClient()
 {
     var client = _factory.CreateClient();
     var token = HttpContext.Session.GetString("JWToken");
     client.DefaultRequestHeaders.Authorization = new AuthenticationHeaderValue("Bearer", token);
     return client;
 }
```

---

## 4. Dashboard Module

**DashboardController**

```csharp
using Consume_202526.Models;
using Microsoft.AspNetCore.Mvc;

using System.Net.Http.Headers;

namespace Consume_202526.Controllers
{
    public class DashboardController : Controller
    {
        private readonly IHttpClientFactory _factory;
        private readonly string _baseUrl;

        public DashboardController(IHttpClientFactory factory, IConfiguration config)
        {
            _factory = factory;
            _baseUrl = config["ApiSettings:BaseUrl"];
        }

        // Helper method to setup the client with the Session Token
        private HttpClient GetClient()
        {
            var client = _factory.CreateClient();
            var token = HttpContext.Session.GetString("JWToken");

            // Redirect to login if token is missing
            if (string.IsNullOrEmpty(token))
            {
                return null;
            }

            client.DefaultRequestHeaders.Authorization = new AuthenticationHeaderValue("Bearer", token);
            return client;
        }

        public async Task<IActionResult> Index()
        {
            var client = GetClient();
            var role = HttpContext.Session.GetString("UserRole");
            var token = HttpContext.Session.GetString("JWToken");

            if (string.IsNullOrEmpty(token))
            {
                return RedirectToAction("Login", "Account");
            }

            ViewBag.Role = role; // Pass role to View

            // Protection: If no token, kick back to login
            if (client == null)
            {
                return RedirectToAction("Login", "Account");
            }

            try
            {
                // Optional: Fetch counts to show on Dashboard cards
                var departments = await client.GetFromJsonAsync<List<Department>>(_baseUrl + "api/Department/GetAll");
                var staff = await client.GetFromJsonAsync<List<Staff>>(_baseUrl + "api/MOM_Staff/GetAll");

                ViewBag.DeptCount = departments?.Count ?? 0;
                ViewBag.StaffCount = staff?.Count ?? 0;
            }
            catch (Exception ex)
            {
                // If API fails, we still show the dashboard but with 0 counts
                ViewBag.DeptCount = 0;
                ViewBag.StaffCount = 0;
                ViewBag.Error = "System could not sync data: " + ex.Message;
            }

            return View();
        }
    }
}
```

**Index.cshtml**

```html
<div class="container py-5">
  <div class="text-center mb-5">
    <h1>Welcome, @Context.Session.GetString("UserName")</h1>
    <span class="badge bg-info text-dark">Role: @ViewBag.Role</span>
  </div>

  <div class="row">
    @if (ViewBag.Role == "Admin") {
    <div class="col-md-6 mb-4">
      <div class="card shadow border-primary h-100">
        <div class="card-body text-center">
          <h3 class="card-title">Department Management</h3>
          <p>Full access to Create, Update, and Delete Departments.</p>
          <a
            asp-controller="Department"
            asp-action="Index"
            class="btn btn-primary"
            >Enter</a
          >
        </div>
      </div>
    </div>
    } @if (ViewBag.Role == "Admin" || ViewBag.Role == "Manager") {
    <div class="col-md-6 mb-4">
      <div class="card shadow border-success h-100">
        <div class="card-body text-center">
          <h3 class="card-title">Staff Management</h3>
          <p>Manage employee records and department assignments.</p>
          <a asp-controller="Staff" asp-action="Index" class="btn btn-success"
            >Enter</a
          >
        </div>
      </div>
    </div>
    }
  </div>
</div>
```

---

## 5. CRUD Operations – Staff Module

### StaffController

```csharp
using Consume_202526.Models;
using Microsoft.AspNetCore.Mvc;
using Newtonsoft.Json; // Ensure this is present
using System.Net.Http.Headers;
using System.Text;

namespace Consume_202526.Controllers
{
    public class StaffController : Controller
    {
        private readonly IHttpClientFactory _factory;
        private readonly string _baseUrl;

        public StaffController(IHttpClientFactory factory, IConfiguration config)
        {
            _factory = factory;
            _baseUrl = config["ApiSettings:BaseUrl"];
        }

        private HttpClient GetClient()
        {
            var client = _factory.CreateClient();
            var token = HttpContext.Session.GetString("JWToken");
            client.DefaultRequestHeaders.Authorization = new AuthenticationHeaderValue("Bearer", token);
            return client;
        }

        public async Task<IActionResult> Index()
        {
            try
            {
                var client = GetClient();
                // 1. Fetch raw string
                string jsonString = await client.GetStringAsync(_baseUrl + "api/MOM_Staff/GetAll");

                // 2. Use Newtonsoft to Deserialize the wrapped list
                var result = JsonConvert.DeserializeObject<ServiceResponse<List<Staff>>>(jsonString);

                if (result != null && result.Data != null)
                {
                    return View(result.Data);
                }
                return View(new List<Staff>());
            }
            catch (Exception ex)
            {
                ViewBag.Error = "Error: " + ex.Message;
                return View(new List<Staff>());
            }
        }

        public async Task<IActionResult> AddEdit(int id = 0)
        {
            await LoadDepartments();

            if (id == 0)
            {
                return View(new Staff { IsActive = true, RoleID = 2 });
            }
            else
            {
                try
                {
                    var client = GetClient();
                    // 1. Fetch raw string
                    string jsonString = await client.GetStringAsync(_baseUrl + "api/MOM_Staff/GetById/" + id);
                    
                    // 2. Deserialize single object
                    var result = JsonConvert.DeserializeObject<ServiceResponse<Staff>>(jsonString);

                    if (result != null && result.Data != null)
                    {
                        return View(result.Data);
                    }
                    return RedirectToAction("Index");
                }
                catch
                {
                    return RedirectToAction("Index");
                }
            }
        }

        [HttpPost]
        public async Task<IActionResult> AddEdit(Staff model)
        {
            try
            {
                var client = GetClient();
                HttpResponseMessage response;

                // 1. Manually serialize the model using Newtonsoft
                string jsonModel = JsonConvert.SerializeObject(model);
                
                // 2. Prepare content (must specify media type as application/json)
                var content = new StringContent(jsonModel, Encoding.UTF8, "application/json");

                if (model.StaffID == 0 || model.StaffID == null)
                {
                    response = await client.PostAsync(_baseUrl + "api/MOM_Staff/Create", content);
                }
                else
                {
                    response = await client.PutAsync(_baseUrl + "api/MOM_Staff/Update/" + model.StaffID, content);
                }

                if (response.IsSuccessStatusCode)
                {
                    return RedirectToAction("Index");
                }
            }
            catch (Exception ex) 
            { 
                ViewBag.Error = "Save Error: " + ex.Message; 
            }

            await LoadDepartments();
            return View(model);
        }

        private async Task LoadDepartments()
        {
            try
            {
                var client = GetClient();
                // 1. Fetch raw string
                string jsonString = await client.GetStringAsync(_baseUrl + "api/Department/GetAll");
                
                // 2. Deserialize (check if your Dept API also uses the ServiceResponse wrapper)
                var result = JsonConvert.DeserializeObject<List<Department>>(jsonString);
                
                ViewBag.Departments = result ?? new List<Department>();
            }
            catch
            {
                ViewBag.Departments = new List<Department>();
            }
        }

        public async Task<IActionResult> Delete(int id)
        {
            try
            {
                var client = GetClient();
                await client.DeleteAsync(_baseUrl + "api/MOM_Staff/Delete/" + id);
            }
            catch (Exception ex)
            {
                TempData["Error"] = "Delete failed: " + ex.Message;
            }
            return RedirectToAction("Index");
        }
    }
}
```

**AddEdit.cshtml**

```html
@model Staff

<div class="container mt-4">
    <div class="card shadow-sm col-md-8 mx-auto">
        <div class="card-header bg-dark text-white">
            <h4 class="mb-0">@(Model.StaffID > 0 ? "Update" : "Add") Staff Member</h4>
        </div>
        <div class="card-body">
            <form asp-action="AddEdit" method="post">
                <input type="hidden" asp-for="StaffID" />
                <input type="hidden" asp-for="RoleID" />

                <div class="row mb-3">
                    <div class="col-md-6">
                        <label class="fw-bold">Staff Name</label>
                        <input asp-for="StaffName" class="form-control" required />
                    </div>
                    <div class="col-md-6">
                        <label class="fw-bold">Department</label>
                        <select asp-for="DepartmentID" class="form-control" required
                                asp-items="@(new SelectList(ViewBag.Departments, "DepartmentID", "DepartmentName"))">
                            <option value="">-- Choose Department --</option>
                        </select>
                    </div>
                </div>

                <div class="row mb-3">
                    <div class="col-md-6">
                        <label class="fw-bold">Email</label>
                        <input asp-for="EmailAddress" type="email" class="form-control" required />
                    </div>
                    <div class="col-md-6">
                        <label class="fw-bold">Mobile</label>
                        <input asp-for="MobileNo" class="form-control" required />
                    </div>
                </div>

                <div class="row mb-3">
                    <div class="col-md-6">
                        <label class="fw-bold">Password</label>
                        <input asp-for="Password" type="password" class="form-control" required />
                    </div>
                    <div class="col-md-6">
                        <label class="fw-bold">Status</label>
                        <select asp-for="IsActive" class="form-select">
                            <option value="true">Active</option>
                            <option value="false">Inactive</option>
                        </select>
                    </div>
                </div>

                <div class="border-top pt-3">
                    <button type="submit" class="btn btn-primary px-5">Submit</button>
                    <a asp-action="Index" class="btn btn-outline-secondary ms-2">Back</a>
                </div>
            </form>
        </div>
    </div>
</div>
```

**Index.cshtml**

```html
@model IEnumerable<Staff>
  <div class="container mt-4">
    <div class="card shadow-sm">
      <div
        class="card-header bg-dark text-white d-flex justify-content-between align-items-center"
      >
        <h4 class="mb-0">Staff Directory</h4>
        <a asp-action="AddEdit" class="btn btn-success btn-sm fw-bold">
          <i class="bi bi-person-plus"></i> Register New Staff
        </a>
      </div>
      <div class="card-body">
        @if (Model != null && Model.Any()) {
        <div class="table-responsive">
          <table class="table table-striped align-middle">
            <thead class="table-secondary">
              <tr>
                <th>Staff Name</th>
                <th>Department</th>
                <th>Contact Info</th>
                <th>Status</th>
                <th class="text-center">Actions</th>
              </tr>
            </thead>
            <tbody>
              @foreach (var s in Model) {
              <tr>
                <td>
                  <div class="fw-bold">@s.StaffName</div>
                  <small class="text-muted">ID: @s.StaffID</small>
                </td>
                <td>@s.DepartmentName</td>
                <td>
                  <div>@s.EmailAddress</div>
                  <div class="small text-muted">@s.MobileNo</div>
                </td>
                <td>
                  @if (s.IsActive) {
                  <span class="badge bg-success">Active</span>
                  } else {
                  <span class="badge bg-danger">Inactive</span>
                  }
                </td>
                <td class="text-center">
                  <a
                    asp-action="AddEdit"
                    asp-route-id="@s.StaffID"
                    class="btn btn-info btn-sm text-white"
                    >Edit</a
                  >

                  <a
                    asp-action="Delete"
                    asp-route-id="@s.StaffID"
                    class="btn btn-danger btn-sm"
                    onclick="return confirm('Delete this staff member?')"
                    >Delete</a
                  >
                </td>
              </tr>
              }
            </tbody>
          </table>
        </div>
        } else {
        <div class="text-center py-4">
          <p class="text-muted">No staff records found.</p>
        </div>
        }
      </div>
    </div>
  </div></Staff
>
```
## 6. CRUD Operations – Department Module

### DepartmentController

```csharp
using Consume_202526.Models;
using Microsoft.AspNetCore.Mvc;

using System.Net.Http.Headers;

namespace Consume_202526.Controllers
{
    public class DepartmentController : Controller
    {
        private readonly IHttpClientFactory _factory;
        private readonly string _baseUrl;

        public DepartmentController(IHttpClientFactory factory, IConfiguration config)
        {
            _factory = factory;
            _baseUrl = config["ApiSettings:BaseUrl"];
        }

        private HttpClient GetClient()
        {
            var client = _factory.CreateClient();
            var token = HttpContext.Session.GetString("JWToken");
            client.DefaultRequestHeaders.Authorization = new AuthenticationHeaderValue("Bearer", token);
            return client;
        }

        public async Task<IActionResult> Index()
        {
            try
            {
                var client = GetClient();
                var list = await client.GetFromJsonAsync<List<Department>>(_baseUrl + "api/Department/GetAll");
                return View(list);
            }
            catch (Exception ex)
            {
                return View(new List<Department>());
            }
        }

        // Single method to handle both Add and Edit View
        public async Task<IActionResult> AddEdit(int id = 0)
        {
            if (id == 0)
            {
                return View(new Department());
            }
            else
            {
                try
                {
                    var client = GetClient();
                    // Call API with the wrapper ServiceResponse<Department>
                    var response = await client.GetFromJsonAsync<ServiceResponse<Department>>(_baseUrl + "api/Department/GetById/" + id);

                    if (response != null && response.Data != null)
                    {
                        return View(response.Data); // Pass the inner Data to the View
                    }
                    return RedirectToAction("Index");
                }
                catch (Exception ex)
                {
                    return RedirectToAction("Index");
                }
            }
        }

        [HttpPost]
        public async Task<IActionResult> AddEdit(Department model)
        {
            try
            {
                var client = GetClient();
                HttpResponseMessage response;

                // If ID is 0, it's a new record (Create)
                if (model.DepartmentID == 0 || model.DepartmentID == null)
                {
                    response = await client.PostAsJsonAsync(_baseUrl + "api/Department/Create", model);
                }
                // If ID > 0, it's an existing record (Update)
                else
                {
                    response = await client.PutAsJsonAsync(_baseUrl + "api/Department/Update/" + model.DepartmentID, model);
                }

                if (response.IsSuccessStatusCode)
                {
                    return RedirectToAction("Index");
                }
            }
            catch (Exception ex)
            {
                ViewBag.Error = "Operation failed: " + ex.Message;
            }
            return View(model);
        }

        public async Task<IActionResult> Delete(int id)
        {
            try
            {
                var client = GetClient();
                await client.DeleteAsync(_baseUrl + "api/Department/Delete/" + id);
            }
            catch { }
            return RedirectToAction("Index");
        }
    }
}
```
### Index.cshtml

```html
@model IEnumerable<Department>

<div class="container mt-4">
    <div class="card shadow-sm">
        <div class="card-header bg-primary text-white d-flex justify-content-between align-items-center">
            <h4 class="mb-0">Department List</h4>
            <a asp-action="AddEdit" class="btn btn-light btn-sm fw-bold">
                <i class="bi bi-plus-circle"></i> Add New Department
            </a>
        </div>
        <div class="card-body">
            @if (Model != null && Model.Any())
            {
                <table class="table table-hover align-middle">
                    <thead class="table-light">
                        <tr>
                            <th style="width: 10%">ID</th>
                            <th style="width: 65%">Department Name</th>
                            <th style="width: 25%" class="text-center">Actions</th>
                        </tr>
                    </thead>
                    <tbody>
                        @foreach (var item in Model)
                        {
                            <tr>
                                <td>@item.DepartmentID</td>
                                <td><span class="fw-bold">@item.DepartmentName</span></td>
                                <td class="text-center">
                                    <div class="btn-group">
                                        <a asp-action="AddEdit" asp-route-id="@item.DepartmentID"
                                           class="btn btn-outline-warning btn-sm">
                                            Edit
                                        </a>
                                        <a asp-action="Delete" asp-route-id="@item.DepartmentID"
                                           class="btn btn-outline-danger btn-sm"
                                           onclick="return confirm('Are you sure you want to delete this department?')">
                                            Delete
                                        </a>
                                    </div>
                                </td>
                            </tr>
                        }
                    </tbody>
                </table>
            }
            else
            {
                <div class="alert alert-info mb-0">No departments found. Click "Add New" to create one.</div>
            }
        </div>
    </div>
</div>
```

### AddEdit.cshtml
```html
@model IEnumerable<Department>

<div class="container mt-4">
    <div class="card shadow-sm">
        <div class="card-header bg-primary text-white d-flex justify-content-between align-items-center">
            <h4 class="mb-0">Department List</h4>
            <a asp-action="AddEdit" class="btn btn-light btn-sm fw-bold">
                <i class="bi bi-plus-circle"></i> Add New Department
            </a>
        </div>
        <div class="card-body">
            @if (Model != null && Model.Any())
            {
                <table class="table table-hover align-middle">
                    <thead class="table-light">
                        <tr>
                            <th style="width: 10%">ID</th>
                            <th style="width: 65%">Department Name</th>
                            <th style="width: 25%" class="text-center">Actions</th>
                        </tr>
                    </thead>
                    <tbody>
                        @foreach (var item in Model)
                        {
                            <tr>
                                <td>@item.DepartmentID</td>
                                <td><span class="fw-bold">@item.DepartmentName</span></td>
                                <td class="text-center">
                                    <div class="btn-group">
                                        <a asp-action="AddEdit" asp-route-id="@item.DepartmentID"
                                           class="btn btn-outline-warning btn-sm">
                                            Edit
                                        </a>
                                        <a asp-action="Delete" asp-route-id="@item.DepartmentID"
                                           class="btn btn-outline-danger btn-sm"
                                           onclick="return confirm('Are you sure you want to delete this department?')">
                                            Delete
                                        </a>
                                    </div>
                                </td>
                            </tr>
                        }
                    </tbody>
                </table>
            }
            else
            {
                <div class="alert alert-info mb-0">No departments found. Click "Add New" to create one.</div>
            }
        </div>
    </div>
</div>
```
