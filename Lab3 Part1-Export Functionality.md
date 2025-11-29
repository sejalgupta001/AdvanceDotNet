# Lab3-Excel Export Functionality Using ClosedXML

# 1. Install ClosedXML

```
Install-Package ClosedXML
```

---

# 2. SQL Stored Procedure

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
---

# 3. Add Excel Export in Controller

### **Controller Code — State Excel Export**

```csharp
public IActionResult ExportToExcel()
{
    try
    {
        // Step 1: Fetch Data from Stored Procedure into DataTable
        DataTable dt = new DataTable();
        string connStr = _configuration.GetConnectionString("ConnectionString");

        using (SqlConnection conn = new SqlConnection(connStr))
        {
            conn.Open();

            using (SqlCommand cmd = conn.CreateCommand())
            {
                cmd.CommandType = CommandType.StoredProcedure;
                cmd.CommandText = "PR_State_SelectAll";

                using (SqlDataReader dr = cmd.ExecuteReader())
                {
                    dt.Load(dr);
                }
            }
        }

        // Step 2: Generate Excel File Using ClosedXML
        using (var workbook = new XLWorkbook())
        {
            var worksheet = workbook.Worksheets.Add("States");

            // Add Header Row Dynamically
            for (int i = 0; i < dt.Columns.Count; i++)
            {
                worksheet.Cell(1, i + 1).Value = dt.Columns[i].ColumnName;
                worksheet.Cell(1, i + 1).Style.Font.Bold = true;
            }

            // Add Data Rows
            for (int row = 0; row < dt.Rows.Count; row++)
            {
                for (int col = 0; col < dt.Columns.Count; col++)
                {
                    worksheet.Cell(row + 2, col + 1).Value = dt.Rows[row][col]?.ToString();
                }
            }

            worksheet.Columns().AdjustToContents();

            using (var stream = new MemoryStream())
            {
                workbook.SaveAs(stream);
                var content = stream.ToArray();

                return File(
                    content,
                    "application/vnd.openxmlformats-officedocument.spreadsheetml.sheet",
                    "States.xlsx"
                );
            }
        }
    }
    catch (Exception ex)
    {
        TempData["ErrorMessage"] = "Error exporting state data: " + ex.Message;
        return RedirectToAction("Index");
    }
}
```

---

# 4. Add Download Button in View

```html
<a href="/State/ExportToExcel" class="btn btn-primary">
  Download State Excel
</a>
```

---

# Task To do:

Create Excel Export for **Employee** and **Department**
