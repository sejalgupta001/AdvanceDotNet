# Lab 4 – Data Filtering : Implement Various Filters on the State List Screen

# 1. Stored Procedure: PR_State_Search

```sql
CREATE PROCEDURE PR_State_Search
    @StateName VARCHAR(100) = '',
    @StateCode VARCHAR(50) = '',
    @CountryID INT = 0
AS
BEGIN
    SELECT
        S.StateID,
        S.StateName,
        S.StateCode,
        C.CountryName
    FROM State S
    INNER JOIN Country C ON S.CountryID = C.CountryID
    WHERE
        (@StateName = '' OR S.StateName LIKE '%' + @StateName + '%')
        AND (@StateCode = '' OR S.StateCode LIKE '%' + @StateCode + '%')
        AND (@CountryID = 0 OR S.CountryID = @CountryID)
    ORDER BY S.StateName
END
```

Explanation:

• If StateName is empty, that condition is skipped.
• If StateCode is empty, that condition is skipped.
• If CountryID is 0, the country filter is skipped.
• This approach is called Optional Filtering.

---

# 2. Controller Code: StateController.cs

```csharp
public IActionResult Index(string StateName = "", string StateCode = "", int CountryID = 0)
{
    // Load dropdown list for Country
    LoadCountryDropdown(CountryID);

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
                cmd.CommandText = "PR_State_Search";

                // Passing Search Values to Stored Procedure
                cmd.Parameters.Add("@StateName", SqlDbType.VarChar).Value = StateName ?? "";
                cmd.Parameters.Add("@StateCode", SqlDbType.VarChar).Value = StateCode ?? "";
                cmd.Parameters.Add("@CountryID", SqlDbType.Int).Value = CountryID;

                SqlDataAdapter da = new SqlDataAdapter(cmd);
                da.Fill(dt);
            }
        }
    }
    catch (Exception ex)
    {
        TempData["ErrorMessage"] = "Error loading state list: " + ex.Message;
    }

    // Preserve search values
    ViewBag.StateName = StateName;
    ViewBag.StateCode = StateCode;
    ViewBag.SelectedCountryID = CountryID;

    return View(dt);
}

// Method to load Country dropdown list
private void LoadCountryDropdown(int selectedCountryID = 0)
{
    List<CountryDropDownModel> countryList = new List<CountryDropDownModel>();

    string conn = _configuration.GetConnectionString("ConnectionString");

    using (SqlConnection sql = new SqlConnection(conn))
    {
        sql.Open();
        using (SqlCommand cmd = sql.CreateCommand())
        {
            cmd.CommandType = CommandType.StoredProcedure;
            cmd.CommandText = "PR_Country_SelectAll";

            SqlDataReader dr = cmd.ExecuteReader();

            while (dr.Read())
            {
                countryList.Add(new CountryDropDownModel
                {
                    CountryID = Convert.ToInt32(dr["CountryID"]),
                    CountryName = dr["CountryName"].ToString()
                });
            }
        }
    }

    ViewBag.CountryList = countryList;
    ViewBag.CountryID = selectedCountryID;
}
```

---

# 3. View Page: Index.cshtml

```cshtml
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

    <a href="/State/ExportToExcel" class="btn btn-primary mb-3">
        Download State Excel
    </a>

    @if (TempData["SuccessMessage"] != null)
    {
        <div class="alert alert-success">@TempData["SuccessMessage"]</div>
    }

    @if (TempData["ErrorMessage"] != null)
    {
        <div class="alert alert-danger">@TempData["ErrorMessage"]</div>
    }

    <!-- Search Form -->
    <form method="post" asp-action="Index" class="mb-4">
        <div class="row">
            <div class="col-md-3">
                <input type="text" name="StateName" class="form-control" placeholder="State Name"
                       value="@ViewBag.StateName" />
            </div>

            <div class="col-md-3">
                <input type="text" name="StateCode" class="form-control" placeholder="State Code"
                       value="@ViewBag.StateCode" />
            </div>

            <div class="col-md-3">
                <select name="CountryID" class="form-control"
                        asp-items="@(new SelectList(ViewBag.CountryList, "CountryID", "CountryName", ViewBag.SelectedCountryID))">
                    <option value="0">Select Country</option>
                </select>
            </div>

            <div class="col-md-2">
                <button type="submit" class="btn btn-primary w-100">Search</button>
            </div>

            <div class="col-md-1">
                <a href="@Url.Action("Index")" class="btn btn-secondary w-100">Clear</a>
            </div>
        </div>
    </form>

    <!-- Table -->
    <table class="table table-hover">
        <thead>
            <tr>
                <th>State Name</th>
                <th>State Code</th>
                <th>Country</th>
                <th class="text-center">Actions</th>
            </tr>
        </thead>

        <tbody>
            @if (Model.Rows.Count > 0)
            {
                foreach (DataRow row in Model.Rows)
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
            }
            else
            {
                <tr>
                    <td colspan="4" class="text-center">No records found</td>
                </tr>
            }
        </tbody>
    </table>
</main>
```

---

# Task To Do

Implement Similar Data Filtering For:

## Employee List Screen

• Filter by Employee Name
• Filter by Salary
• Filter by Department (Dropdown)
• Filter by JoiningDate
• Filter by City

Follow the same pattern used in this lab:

• Create Stored Procedure
• Modify Controller
• Add Search Form in the View
• Preserve Search Values
• Display Filtered Results
