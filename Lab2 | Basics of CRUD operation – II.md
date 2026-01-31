# **Lab 2 – Insert & Update Operation (State)**


# **1. SQL Stored Procedures for INSERT & UPDATE (STATE)**

## **1.1 PR_State_SelectByPK**
```sql
CREATE PROCEDURE [dbo].[PR_State_SelectByPK]
    @StateID INT
AS
BEGIN
    SELECT 
        S.StateID,
        S.StateName,
        S.StateCode,
        S.CountryID
    FROM State S
    WHERE S.StateID = @StateID;
END
```


---

## **1.2 PR_State_Insert**

```sql
CREATE PROCEDURE PR_State_Insert
    @StateName VARCHAR(100),
    @StateCode VARCHAR(10),
    @CountryID INT
AS
BEGIN
    INSERT INTO State (StateName, StateCode, CountryID)
    VALUES (@StateName, @StateCode, @CountryID);
END
GO
```

---

## **1.3 PR_State_Update**

```sql
CREATE PROCEDURE PR_State_Update
    @StateID INT,
    @StateName VARCHAR(100),
    @StateCode VARCHAR(10),
    @CountryID INT
AS
BEGIN
    UPDATE State
    SET 
        StateName = @StateName,
        StateCode = @StateCode,
        CountryID = @CountryID
    WHERE StateID = @StateID;
END
GO
```

---

# **2. Controller – StateController.cs**

---

## **2.1 AddEdit Method (Single View for Insert + Update)**

```csharp
public IActionResult AddEdit(int? StateID)
{
    StateModel model = new StateModel();

    // Load Country Dropdown
    LoadCountryDropdown();

    if (StateID != null)
    {
        try
        {
            string conn = _configuration.GetConnectionString("ConnectionString");

            using (SqlConnection sql = new SqlConnection(conn))
            {
                sql.Open();

                using (SqlCommand cmd = sql.CreateCommand())
                {
                    cmd.CommandType = CommandType.StoredProcedure;
                    cmd.CommandText = "PR_State_SelectByPK";
                    cmd.Parameters.AddWithValue("@StateID", StateID);

                    SqlDataReader dr = cmd.ExecuteReader();
                    if (dr.Read())
                    {
                        model.StateID = Convert.ToInt32(dr["StateID"]);
                        model.StateName = dr["StateName"].ToString();
                        model.StateCode = dr["StateCode"].ToString();
                        model.CountryID = Convert.ToInt32(dr["CountryID"]);
                    }
                }
            }
        }
        catch (Exception ex)
        {
            TempData["ErrorMessage"] = "Error loading state: " + ex.Message;
        }
    }

    return View("AddEdit", model);
}
```

---

### 🔹 **Load Country Dropdown Helper**

```csharp
private void LoadCountryDropdown()
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
}
```

---

## **2.2 Save Method (Insert + Update combined)**

```csharp
[HttpPost]
 public IActionResult Save(StateModel model)
 {
     try
     {
         if (!ModelState.IsValid)
         {
             LoadCountryDropdown();
             return View("AddEdit", model);
         }

         string conn = _configuration.GetConnectionString("ConnectionString");

         using (SqlConnection sql = new SqlConnection(conn))
         {
             sql.Open();
             using (SqlCommand cmd = sql.CreateCommand())
             {
                 cmd.CommandType = CommandType.StoredProcedure;

                 if (model.StateID == 0)
                 {
                     cmd.CommandText = "PR_State_Insert";
                 }
                 else
                 {
                     cmd.CommandText = "PR_State_Update";
                     cmd.Parameters.AddWithValue("@StateID", model.StateID);
                 }

                 cmd.Parameters.AddWithValue("@StateName", model.StateName);
                 cmd.Parameters.AddWithValue("@StateCode", model.StateCode);
                 cmd.Parameters.AddWithValue("@CountryID", model.CountryID);

                 cmd.ExecuteNonQuery();
             }
         }

         TempData["SuccessMessage"] = model.StateID == 0 ?
             "State added successfully!" :
             "State updated successfully!";

         return RedirectToAction("Index");
     }
     catch (Exception ex)
     {
         TempData["ErrorMessage"] = "Error saving: " + ex.Message;
         return RedirectToAction("AddEdit");
     }
 }
```

---

# **3. View – AddEdit.cshtml (State)**

```html
@model StateModel

<form asp-action="Save" method="post">

    <input type="hidden" asp-for="StateID" />

    <div class="form-group">
        <label>State Name</label>
        <input asp-for="StateName" class="form-control" />
        <span asp-validation-for="StateName" class="text-danger"></span>
    </div>

    <div class="form-group">
        <label>State Code</label>
        <input asp-for="StateCode" class="form-control" />
        <span asp-validation-for="StateCode" class="text-danger"></span>
    </div>

    <div class="form-group">
        <label>Select Country</label>
        <select asp-for="CountryID" class="form-control" asp-items="@(new SelectList(ViewBag.CountryList, "CountryID", "CountryName"))">
            <option value="0">Select Country</option>
            </select>
        <span asp-validation-for="CountryID" class="text-danger"></span>
    </div>

    <br />

    <button type="submit" class="btn btn-success">Save</button>
</form>

```

---

# **4. Model – StateModel.cs**

```csharp
using System.ComponentModel.DataAnnotations;
namespace add.Models
{
    public class StateModel
    {
        public int StateID { get; set; }

        [Required(ErrorMessage = "State Name is required")]
        [StringLength(100, ErrorMessage = "State Name cannot exceed 100 characters")]
        public string StateName { get; set; }

        [Required(ErrorMessage = "State Code is required")]
        [StringLength(10, ErrorMessage = "State Code cannot exceed 10 characters")]
        public string StateCode { get; set; }

        [Required(ErrorMessage = "Please select a country")]
        [Range(1, int.MaxValue, ErrorMessage = "Invalid Country")]
        public int CountryID { get; set; }
    }

    public class CountryDropDownModel
    {
        public int CountryID { get; set; }
        public string CountryName { get; set; }
    }

}

```
---

# **TASK FOR YOU**
1. Add Insert and Update functionality for Department. 
2. Add Insert and Update functionality for Employee. 
