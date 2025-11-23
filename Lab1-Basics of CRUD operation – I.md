# Lab 1 – Basics of CRUD Operation – I

## Country and State Example (Reference for Department and Employee)

This help file guides on how to perform basic CRUD operations (Select All and Delete) in ASP.NET Core MVC using SQL Server Stored Procedures.
We use the Country and State tables as an example.

---

# 1. SQL Stored Procedures

## Country Procedures

```sql
CREATE PROCEDURE PR_Country_SelectAll
AS
BEGIN
    SELECT CountryID, CountryName
    FROM Country
END
GO
```

```sql
CREATE PROCEDURE PR_Country_Delete
    @CountryID INT
AS
BEGIN
    DELETE FROM Country WHERE CountryID = @CountryID
END
GO
```

---

## State Procedures

```sql
CREATE PROCEDURE PR_State_SelectAll
AS
BEGIN
    SELECT 
        S.StateID,
        S.StateName,
        S.StateCode,
        S.CountryID,
        C.CountryName
    FROM State S
    LEFT JOIN Country C ON C.CountryID = S.CountryID
END
GO
```

```sql
CREATE PROCEDURE PR_State_Delete
    @StateID INT
AS
BEGIN
    DELETE FROM State WHERE StateID = @StateID
END
GO
```

---

# 2. MVC Models

## Models/CountryModel.cs

```csharp
namespace YourProject.Models
{
    public class CountryModel
    {
        public int CountryID { get; set; }
        public string CountryName { get; set; }
    }
}
```

---

## Models/StateModel.cs

```csharp
namespace YourProject.Models
{
    public class StateModel
    {
        public int StateID { get; set; }
        public string StateName { get; set; }
        public string StateCode { get; set; }

        // Foreign Key
        public int CountryID { get; set; }

        // For display only
        public string CountryName { get; set; }
    }
}
```

---

# 3. appsettings.json Connection String

```json
  "ConnectionStrings": {
    "ConnectionString": "Server=YOUR_SERVER_NAME;Database=YOUR_DATABASE_NAME;Trusted_Connection=True;Encrypt=False;TrustServerCertificate=True"
  }
```


---

# 4. Controllers
# Required Packages to Install

Before starting, install the following NuGet package in your MVC project:

### ✔ **System.Data.SqlClient**

Used for SQL Connection, Command, and DataReader operations.

Install using Package Manager Console:

```
Install-Package System.Data.SqlClient
```

Or search in NuGet UI:
**System.Data.SqlClient → Install**

---

## Controllers/CountryController.cs

```csharp
using Microsoft.AspNetCore.Mvc;
using System.Data.SqlClient;
using System.Data;

namespace YourProjectName.Controllers
{
    public class CountryController : Controller
    {
        private readonly IConfiguration _configuration;

        public CountryController(IConfiguration configuration)
        {
            _configuration = configuration;
        }

        // -------------------------
        // SELECT ALL
        // -------------------------
        public IActionResult Index()
        {
            DataTable dt = new DataTable();

            try
            {
                string connStr = _configuration.GetConnectionString("ConnectionString");

                using (SqlConnection conn = new SqlConnection(connStr))
                {
                    conn.Open();

                    using (SqlCommand cmd = conn.CreateCommand())
                    {
                        cmd.CommandType = CommandType.StoredProcedure;
                        cmd.CommandText = "PR_Country_SelectAll";

                        SqlDataReader reader = cmd.ExecuteReader();
                        dt.Load(reader);
                    }
                }
            }
            catch (Exception ex)
            {
                TempData["ErrorMessage"] = "Error loading country data: " + ex.Message;
            }

            return View("Index", dt);
        }

        // -------------------------
        // DELETE
        // -------------------------
        public IActionResult Delete(int CountryID)
        {
            try
            {
                string connStr = _configuration.GetConnectionString("ConnectionString");

                using (SqlConnection conn = new SqlConnection(connStr))
                {
                    conn.Open();

                    using (SqlCommand cmd = conn.CreateCommand())
                    {
                        cmd.CommandType = CommandType.StoredProcedure;
                        cmd.CommandText = "PR_Country_Delete";
                        cmd.Parameters.AddWithValue("@CountryID", CountryID);

                        cmd.ExecuteNonQuery();
                    }
                }

                TempData["SuccessMessage"] = "Country deleted successfully!";
            }
            catch (Exception ex)
            {
                TempData["ErrorMessage"] = "Error deleting country: " + ex.Message;
            }

            return RedirectToAction("Index");
        }
    }
}

```

---

## Controllers/StateController.cs

```csharp
using Microsoft.AspNetCore.Mvc;
using System.Data.SqlClient;
using System.Data;

namespace YourProjectName.Controllers
{
    public class StateController : Controller
    {
        private readonly IConfiguration _configuration;

        public StateController(IConfiguration configuration)
        {
            _configuration = configuration;
        }

        // -------------------------
        // SELECT ALL
        // -------------------------
        public IActionResult Index()
        {
            DataTable dt = new DataTable();

            try
            {
                string connStr = _configuration.GetConnectionString("ConnectionString");

                using (SqlConnection conn = new SqlConnection(connStr))
                {
                    conn.Open();

                    using (SqlCommand cmd = conn.CreateCommand())
                    {
                        cmd.CommandType = CommandType.StoredProcedure;
                        cmd.CommandText = "PR_State_SelectAll";

                        SqlDataReader dr = cmd.ExecuteReader();
                        dt.Load(dr);
                    }
                }
            }
            catch (Exception ex)
            {
                TempData["ErrorMessage"] = "Error loading state data: " + ex.Message;
            }

            return View("Index", dt);
        }

        // -------------------------
        // DELETE
        // -------------------------
        public IActionResult Delete(int StateID)
        {
            try
            {
                string connStr = _configuration.GetConnectionString("ConnectionString");

                using (SqlConnection conn = new SqlConnection(connStr))
                {
                    conn.Open();

                    using (SqlCommand cmd = conn.CreateCommand())
                    {
                        cmd.CommandType = CommandType.StoredProcedure;
                        cmd.CommandText = "PR_State_Delete";
                        cmd.Parameters.AddWithValue("@StateID", StateID);

                        cmd.ExecuteNonQuery();
                    }
                }

                TempData["SuccessMessage"] = "State deleted successfully!";
            }
            catch (Exception ex)
            {
                TempData["ErrorMessage"] = "Error deleting state: " + ex.Message;
            }

            return RedirectToAction("Index");
        }
    }
}

```

---

# 5. Views

## Views/Country/Index.cshtml

```html
@using System.Data
@{
    ViewData["Title"] = "Country List";
    Layout = "~/Views/Shared/_Layout.cshtml";
}

<main id="main" class="main">

    <div class="pagetitle">
        <h1>Country</h1>
        <nav>
            <ol class="breadcrumb">
                <li class="breadcrumb-item">
                    <a asp-controller="HomeMaster" asp-action="Index">Home</a>
                </li>
                <li class="breadcrumb-item active">Country List</li>
            </ol>
        </nav>

        <div class="d-flex justify-content-end mb-2">
            <a class="btn btn-outline-primary" asp-controller="Country" asp-action="AddEdit">
                Add Country
            </a>
        </div>
    </div>

    <input type="text" class="form-control mb-3" id="searchBox" placeholder="Search">

    @if (TempData["SuccessMessage"] != null)
    {
        <div class="alert alert-success">@TempData["SuccessMessage"]</div>
    }
    
    @if (TempData["ErrorMessage"] != null)
    {
        <div class="alert alert-danger">@TempData["ErrorMessage"]</div>
    }

    <table class="table table-hover">
        <thead>
            <tr>
                <th>Country Name</th>
                <th class="text-center">Actions</th>
            </tr>
        </thead>

        <tbody id="tableRows">
            @foreach (DataRow row in Model.Rows)
            {
                <tr>
                    <td>@row["CountryName"]</td>

                    <td class="text-center">
                        <a class="btn btn-outline-success btn-sm"
                            asp-controller="Country" asp-action="AddEdit"
                            asp-route-CountryID="@row["CountryID"]">Edit</a>

                        <a class="btn btn-outline-danger btn-sm"
                            asp-controller="Country" asp-action="Delete"
                            asp-route-CountryID="@row["CountryID"]"
                            onclick="return confirm('Delete this country?');">
                            Delete
                        </a>
                    </td>
                </tr>
            }
        </tbody>
    </table>
</main>

@section Scripts {
<script>
$("#searchBox").on("keyup", function () {
    var value = $(this).val().toLowerCase();
    $("#tableRows tr").filter(function () {
        $(this).toggle($(this).text().toLowerCase().indexOf(value) > -1);
    });
});
</script>
}
```

---

## Views/State/Index.cshtml

```html
@using System.Data
@{
    ViewData["Title"] = "State List";
    Layout = "~/Views/Shared/_Layout.cshtml";
}

<main id="main" class="main">

    <div class="pagetitle">
        <h1>State</h1>

        <nav>
            <ol class="breadcrumb">
                <li class="breadcrumb-item">
                    <a asp-controller="HomeMaster" asp-action="Index">Home</a>
                </li>
                <li class="breadcrumb-item active">State List</li>
            </ol>
        </nav>

        <div class="d-flex justify-content-end mb-2">
            <a class="btn btn-outline-primary" asp-controller="State" asp-action="AddEdit">
                Add State
            </a>
        </div>
    </div>

    <input type="text" class="form-control mb-3" id="searchBox" placeholder="Search">

    @if (TempData["SuccessMessage"] != null)
    {
        <div class="alert alert-success">@TempData["SuccessMessage"]</div>
    }
    
    @if (TempData["ErrorMessage"] != null)
    {
        <div class="alert alert-danger">@TempData["ErrorMessage"]</div>
    }

    <table class="table table-hover">
        <thead>
            <tr>
                <th>State Name</th>
                <th>State Code</th>
                <th>Country</th>
                <th class="text-center">Actions</th>
            </tr>
        </thead>

        <tbody id="tableRows">
            @foreach (DataRow row in Model.Rows)
            {
                <tr>
                    <td>@row["StateName"]</td>
                    <td>@row["StateCode"]</td>
                    <td>@row["CountryName"]</td>

                    <td class="text-center">
                        <a class="btn btn-outline-success btn-sm"
                           asp-controller="State"
                           asp-action="AddEdit"
                           asp-route-StateID="@row["StateID"]">
                            Edit
                        </a>

                        <a class="btn btn-outline-danger btn-sm"
                           asp-controller="State"
                           asp-action="Delete"
                           asp-route-StateID="@row["StateID"]"
                           onclick="return confirm('Delete this state?');">
                            Delete
                        </a>
                    </td>
                </tr>
            }
        </tbody>
    </table>
</main>

@section Scripts {
<script>
$("#searchBox").on("keyup", function () {
    var value = $(this).val().toLowerCase();
    $("#tableRows tr").filter(function () {
        $(this).toggle($(this).text().toLowerCase().indexOf(value) > -1);
    });
});
</script>
}
```

---

# ✔ **Task to do**

### **Basics of CRUD Operation – I**

Using the Country–State example, perform Select All and Delete for the following:

---

## 1️⃣ **Tables to Create**

### **Department**

* DeptID (PK, Identity)
* DepartmentName

### **Employee**

* EmpID (PK, Identity)
* EmpName
* Salary
* JoiningDate
* City
* DeptID (FK → Department)

---

## 2️⃣ **Stored Procedures**

Create:

### Department

✔ `PR_Department_SelectAll`
✔ `PR_Department_Delete`

### Employee

✔ `PR_Employee_SelectAll`
✔ `PR_Employee_Delete`

---

## 3️⃣ **Controller Tasks**

Create:

* `DepartmentController → Index(), Delete()`
* `EmployeeController → Index(), Delete()`

---

## 4️⃣ **Views**

Create:

* Department/Index.cshtml
* Employee/Index.cshtml

Include:

✔ Table
✔ Delete button

---

## 5️⃣ **Test**

Ensure all Select All + Delete operations work.
