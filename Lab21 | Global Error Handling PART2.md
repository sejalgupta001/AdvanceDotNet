# Global Error Handling – User-Friendly Error Pages & Auto-Redirect

## Overview

In **Part 1**, we built the middleware and interceptor to **catch** API errors. Now we need to:

1. Create an **ErrorController** to handle different error codes
2. Build **user-friendly error views** (404, 403, 500)
3. Auto-redirect **unauthorized users** (expired/missing token) to Login page
4. Add a **shared layout** for error pages

---

## 1. Error Controller

This controller serves the error views. Each action handles a specific error scenario.

### Controllers/ErrorController.cs

```csharp
using Microsoft.AspNetCore.Mvc;

namespace YourMVCProject.Controllers
{
    public class ErrorController : Controller
    {
        // Handles status codes from UseStatusCodePagesWithReExecute
        [Route("Error/HandleError/{statusCode}")]
        public IActionResult HandleError(int statusCode)
        {
            switch (statusCode)
            {
                case 401:
                    return RedirectToAction("Login", "Account");
                case 403:
                    return View("Forbidden");
                case 404:
                    return View("NotFound");
                case 500:
                    return View("ServerError");
                default:
                    ViewBag.ErrorCode = statusCode;
                    ViewBag.ErrorMessage = "Something unexpected happened.";
                    return View("GenericError");
            }
        }

        public IActionResult NotFound()
        {
            return View();
        }

        public IActionResult Forbidden()
        {
            return View();
        }

        public IActionResult ServerError()
        {
            return View();
        }

        public IActionResult GenericError()
        {
            return View();
        }
    }
}
```

**Key Points:**

- `HandleError(int statusCode)` → Called by `UseStatusCodePagesWithReExecute` from Program.cs
- `401` → Directly redirects to Login (no error page needed)
- Other status codes → Show their respective views
- Individual actions (`NotFound()`, `Forbidden()`, etc.) → Called by our `ApiResponseHandler` from Part 1

---

## 2. Error Views

Create these views inside `Views/Error/` folder.

### Views/Error/NotFound.cshtml (404)

```html
@{
    ViewData["Title"] = "Page Not Found";
    Layout = "_Layout";
}

<div class="container text-center py-5">
    <div class="row justify-content-center">
        <div class="col-md-6">
            <h1 class="display-1 text-warning fw-bold">404</h1>
            <h3 class="mb-3">Page Not Found</h3>
            <p class="text-muted mb-4">
                The page or resource you are looking for does not exist or has been removed.
            </p>
            <a asp-controller="Dashboard" asp-action="Index" class="btn btn-primary">
                Go to Dashboard
            </a>
            <a asp-controller="Account" asp-action="Login" class="btn btn-outline-secondary ms-2">
                Back to Login
            </a>
        </div>
    </div>
</div>
```

---

### Views/Error/Forbidden.cshtml (403)

```html
@{
    ViewData["Title"] = "Access Denied";
    Layout = "_Layout";
}

<div class="container text-center py-5">
    <div class="row justify-content-center">
        <div class="col-md-6">
            <h1 class="display-1 text-danger fw-bold">403</h1>
            <h3 class="mb-3">Access Denied</h3>
            <p class="text-muted mb-4">
                You do not have permission to access this resource. 
                Please contact your administrator if you believe this is a mistake.
            </p>
            <a asp-controller="Dashboard" asp-action="Index" class="btn btn-primary">
                Go to Dashboard
            </a>
        </div>
    </div>
</div>
```

---

### Views/Error/ServerError.cshtml (500)

```html
@{
    ViewData["Title"] = "Server Error";
    Layout = "_Layout";
}

<div class="container text-center py-5">
    <div class="row justify-content-center">
        <div class="col-md-6">
            <h1 class="display-1 text-danger fw-bold">500</h1>
            <h3 class="mb-3">Internal Server Error</h3>
            <p class="text-muted mb-4">
                Something went wrong on the server. This could mean the API is 
                down or experiencing issues. Please try again later.
            </p>
            <a asp-controller="Dashboard" asp-action="Index" class="btn btn-primary">
                Go to Dashboard
            </a>
            <a href="javascript:location.reload();" class="btn btn-outline-warning ms-2">
                Retry
            </a>
        </div>
    </div>
</div>
```

---

### Views/Error/GenericError.cshtml (Any other error)

```html
@{
    ViewData["Title"] = "Error";
    Layout = "_Layout";
}

<div class="container text-center py-5">
    <div class="row justify-content-center">
        <div class="col-md-6">
            <h1 class="display-1 text-secondary fw-bold">@(ViewBag.ErrorCode ?? "Error")</h1>
            <h3 class="mb-3">Something Went Wrong</h3>
            <p class="text-muted mb-4">
                @(ViewBag.ErrorMessage ?? "An unexpected error occurred. Please try again.")
            </p>
            <a asp-controller="Dashboard" asp-action="Index" class="btn btn-primary">
                Go to Dashboard
            </a>
        </div>
    </div>
</div>
```

---

## 3. Auto-Redirect Unauthorized Users to Login

There are **two levels** where unauthorized users are caught:

### Level 1: API Returns 401 (Handled by ApiResponseHandler – Part 1)

When the API returns `401 Unauthorized`, our `ApiResponseHandler` from Part 1 automatically:

1. Clears the session
2. Redirects to `/Account/Login`

This happens **before** the controller even sees the response.

### Level 2: Session Expired / No Token (Handled by a Session Check Filter)

If a user tries to access a page **without logging in** (no token in session), we need to block them. Instead of checking in every controller, create an **Action Filter**.

### Filters/SessionAuthFilter.cs

```csharp
using Microsoft.AspNetCore.Mvc;
using Microsoft.AspNetCore.Mvc.Filters;

namespace YourMVCProject.Filters
{
    public class SessionAuthFilter : ActionFilterAttribute
    {
        public override void OnActionExecuting(ActionExecutingContext context)
        {
            var session = context.HttpContext.Session;
            var token = session.GetString("JWToken");

            if (string.IsNullOrEmpty(token))
            {
                // No token = not logged in → redirect to login
                context.Result = new RedirectToActionResult("Login", "Account", null);
            }

            base.OnActionExecuting(context);
        }
    }
}
```

### How to Use the Filter

**Option A: Apply on specific controllers** (Recommended)

```csharp
[SessionAuthFilter]  // ← Add this attribute
public class DashboardController : Controller
{
    // All actions in this controller now require login
}

[SessionAuthFilter]
public class StaffController : Controller
{
    // Protected
}
```

**Option B: Apply globally in Program.cs** (All controllers protected)

```csharp
builder.Services.AddControllersWithViews(options =>
{
    options.Filters.Add<SessionAuthFilter>();
});
```

> **Note:** If you apply globally, the `AccountController` and `ErrorController` should be excluded. Add `[SkipSessionAuth]` or check the controller name inside the filter:

### Updated Filter with Exclusion

```csharp
using Microsoft.AspNetCore.Mvc;
using Microsoft.AspNetCore.Mvc.Filters;

namespace YourMVCProject.Filters
{
    public class SessionAuthFilter : ActionFilterAttribute
    {
        public override void OnActionExecuting(ActionExecutingContext context)
        {
            // Skip auth check for Account and Error controllers
            var controllerName = context.RouteData.Values["controller"]?.ToString();

            if (controllerName == "Account" || controllerName == "Error")
            {
                base.OnActionExecuting(context);
                return;
            }

            var session = context.HttpContext.Session;
            var token = session.GetString("JWToken");

            if (string.IsNullOrEmpty(token))
            {
                context.Result = new RedirectToActionResult("Login", "Account", null);
            }

            base.OnActionExecuting(context);
        }
    }
}
```

---

## 4. Final Program.cs (Complete)

Combining everything from Part 1 and Part 2:

```csharp
using YourMVCProject.Middleware;
using YourMVCProject.Handlers;
using YourMVCProject.Filters;

var builder = WebApplication.CreateBuilder(args);

// Add MVC with global session filter
builder.Services.AddControllersWithViews(options =>
{
    options.Filters.Add<SessionAuthFilter>(); // Auto-redirect if not logged in
});

builder.Services.AddHttpContextAccessor();
builder.Services.AddSession();

// Register API response handler
builder.Services.AddTransient<ApiResponseHandler>();

// Register named HttpClient with handler
builder.Services.AddHttpClient("ApiClient")
    .AddHttpMessageHandler<ApiResponseHandler>();

var app = builder.Build();

// Global exception handler (must be first)
app.UseMiddleware<GlobalExceptionMiddleware>();

// Handle MVC route errors (404 for missing pages, etc.)
app.UseStatusCodePagesWithReExecute("/Error/HandleError/{0}");

app.UseSession();
app.UseStaticFiles();
app.UseRouting();
app.UseAuthorization();

app.MapControllerRoute(
    name: "default",
    pattern: "{controller=Account}/{action=Login}/{id?}");

app.Run();
```

---

## 5. Final Folder Structure

```
YourMVCProject/
├── Controllers/
│   ├── AccountController.cs
│   ├── DashboardController.cs
│   ├── ErrorController.cs          ← New
│   └── StaffController.cs
├── Filters/
│   └── SessionAuthFilter.cs        ← New
├── Handlers/
│   └── ApiResponseHandler.cs       ← From Part 1
├── Middleware/
│   └── GlobalExceptionMiddleware.cs ← From Part 1
├── Models/
│   └── ...
├── Views/
│   ├── Error/
│   │   ├── NotFound.cshtml          ← New
│   │   ├── Forbidden.cshtml         ← New
│   │   ├── ServerError.cshtml       ← New
│   │   └── GenericError.cshtml      ← New
│   ├── Account/
│   ├── Dashboard/
│   └── Shared/
│       └── _Layout.cshtml
├── Program.cs                       ← Updated
└── appsettings.json
```

---

## 6. Testing Your Error Handling

| Test Case | How to Test | Expected Result |
|---|---|---|
| Expired Token | Login → Wait for token to expire → Click any page | Auto-redirect to Login page |
| No Token | Open `/Dashboard/Index` directly without login | Redirect to Login page |
| Wrong API URL | Change API base URL to wrong value | Server Error page (500) |
| Missing Record | Try to edit a deleted staff record | Not Found page (404) |
| No Permission | Login as User role → Try Admin-only page | Forbidden page (403) |
| API Server Down | Stop the API project → Try any action | Server Error page |

---

## Summary

| Component | What It Does |
|---|---|
| **ErrorController** | Serves user-friendly error views for different status codes |
| **Error Views** (404, 403, 500) | Clean, styled pages instead of ugly developer errors |
| **SessionAuthFilter** | Auto-redirects to Login if session token is missing |
| **ApiResponseHandler** (Part 1) | Auto-redirects on 401 from API responses |
| **GlobalExceptionMiddleware** (Part 1) | Catches unhandled exceptions and shows error page |

### Complete Error Handling Flow

```
User Request
    │
    ▼
GlobalExceptionMiddleware (catches unhandled exceptions)
    │
    ▼
SessionAuthFilter (checks if user is logged in)
    │
    ▼
Controller calls API using HttpClient("ApiClient")
    │
    ▼
ApiResponseHandler (checks API response: 401, 403, 404, 500)
    │
    ▼
ErrorController → Error View (user-friendly page)
```
