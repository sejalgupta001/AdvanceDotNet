# Consuming FluentValidation API in ASP.NET Core MVC
The goal is to correctly:
- Handle HTTP failures (server/network errors)
- Handle business validation errors (FluentValidation)
- Display errors properly inside MVC Views

---

## API Response Contract
The API does NOT return default ModelState errors. Instead, it returns a structured wrapper.

### Example Validation Error Response (400 Bad Request)

```json
{
  "success": false,
  "message": "Validation Error",
  "data": null,
  "errors": [
    "The DepartmentName field is required.",
    "Department name is required"
  ]
}
```

### Meaning of Fields

| Field | Purpose |
|------|------|
| success | Indicates business success/failure |
| message | General description |
| data | Returned object (null when validation fails) |
| errors | List of validation messages |

Important: Errors are NOT mapped to specific properties, so they must be shown in ValidationSummary.

---

## Step 1 — Create Response Wrapper Class

```csharp
public class ApiResponse<T>
{
    public bool Success { get; set; }
    public string Message { get; set; }
    public T Data { get; set; }
    public List<string> Errors { get; set; }
}
```

---

## Step 2 — Controller Implementation (BEST PRACTICE)

We must handle errors in this order:
1. HTTP error
2. Deserialize response
3. Business validation error
4. Success

```csharp
[HttpPost]
public async Task<IActionResult> Create(DepartmentViewModel model)
{
    var json = JsonSerializer.Serialize(model);
    var content = new StringContent(json, Encoding.UTF8, "application/json");

    var response = await _httpClient.PostAsync("api/Department/Create", content);

    // STEP 1: Handle HTTP Errors (Server Crash / Unauthorized / Not Found)
    if (!response.IsSuccessStatusCode)
    {
        var serverError = await response.Content.ReadAsStringAsync();
        ModelState.AddModelError("", $"Server Error ({(int)response.StatusCode}): {response.ReasonPhrase}");
        return View(model);
    }

    // STEP 2: Read API Response
    var responseBody = await response.Content.ReadAsStringAsync();

    var result = JsonSerializer.Deserialize<ApiResponse<object>>(responseBody,
        new JsonSerializerOptions { PropertyNameCaseInsensitive = true });

    // STEP 3: Handle Business Validation Errors
    if (result != null && !result.Success)
    {
        if (result.Errors != null)
        {
            foreach (var error in result.Errors)
            {
                ModelState.AddModelError("", error);
            }
        }
        return View(model);
    }

    // STEP 4: Success
    return RedirectToAction("Index");
}
```

---

## Step 3 — Display Errors in View

Because errors are not tied to specific fields, use ValidationSummary.

```html
@model DepartmentViewModel

<h2>Create Department</h2>

<div asp-validation-summary="All" class="alert alert-danger"></div>

<form asp-action="Create" method="post">
    <div class="form-group">
        <label asp-for="DepartmentName"></label>
        <input asp-for="DepartmentName" class="form-control" />
        <span asp-validation-for="DepartmentName" class="text-danger"></span>
    </div>

    <button type="submit" class="btn btn-primary">Submit</button>
</form>
```

---

## How Error Flow Works

### Case 1 — Server Failure
API returns: 500 Internal Server Error

Result:
Displayed as: Server Error (500)

### Case 2 — Validation Failure
API returns: success = false + errors[]

Result:
Displayed inside validation summary

### Case 3 — Success
API returns: success = true

Result:
Redirect to Index page

---

## Why Both Checks Are Required

| Type | Example | Checked By |
|----|----|----|
| Technical Error | API crashed | StatusCode |
| Authentication | Token expired | StatusCode |
| Validation Error | Name required | result.Success |
| Business Rule | Duplicate record | result.Success |

---

## Final Flow Diagram (Conceptual)

Request → HTTP Status Check → Deserialize → Success Flag Check → Show Errors or Redirect
