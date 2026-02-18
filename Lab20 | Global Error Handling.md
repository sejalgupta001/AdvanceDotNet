# Global Error Handling – Middleware & HTTP Interceptor (Client-Side MVC)

## Overview

When your MVC project consumes Web APIs, many things can go wrong:

- API server is **down** → Connection error
- Token **expired** or missing → `401 Unauthorized`
- User doesn't have **permission** → `403 Forbidden`
- Wrong URL or deleted record → `404 Not Found`
- API crashes internally → `500 Internal Server Error`

Without proper handling, users see ugly **developer error pages** or the app just breaks silently.

### What We Will Build

| Component | Purpose |
|---|---|
| **Global Exception Middleware** | Catches any unhandled exception in MVC and shows a friendly error page |
| **API Response Interceptor (DelegatingHandler)** | Intercepts every API call and handles 401, 403, 404, 500 responses automatically |
| **Program.cs Registration** | Wires everything together |

---

## Overall Flow

1. MVC Controller calls the API using `HttpClient`
2. **DelegatingHandler** intercepts the API response before it reaches the controller
3. If API returns `401` → Redirect to Login
4. If API returns `404` → Redirect to Not Found page
5. If API returns `500` → Redirect to Server Error page
6. If any **unhandled exception** is thrown → **Global Middleware** catches it and shows error page

---

## 1. Global Exception Middleware

This middleware wraps the **entire MVC request pipeline**. If any controller or service throws an exception that is not caught, this middleware catches it and redirects to an error page.

### Middleware/GlobalExceptionMiddleware.cs

```csharp
namespace YourMVCProject.Middleware
{
    public class GlobalExceptionMiddleware
    {
        private readonly RequestDelegate _next;

        public GlobalExceptionMiddleware(RequestDelegate next)
        {
            _next = next;
        }

        public async Task InvokeAsync(HttpContext context)
        {
            try
            {
                await _next(context); // Continue to the next middleware/controller
            }
            catch (HttpRequestException ex)
            {
                // API connection failed (server down, network issue)
                context.Response.Redirect("/Error/ServerError");
            }
            catch (UnauthorizedAccessException ex)
            {
                // Custom thrown exception for auth issues
                context.Response.Redirect("/Account/Login");
            }
            catch (Exception ex)
            {
                // Any other unhandled exception
                context.Response.Redirect("/Error/ServerError");
            }
        }
    }
}
```

**Key Points:**

- `RequestDelegate _next` → Represents the next step in the pipeline
- `InvokeAsync` → Runs for every incoming HTTP request
- We wrap `_next(context)` in try-catch to **catch any exception globally**
- Different exception types → Different redirect targets

---

## 2. API Response Interceptor (DelegatingHandler)

A `DelegatingHandler` is like a **middleware for HttpClient**. Every API call made through `IHttpClientFactory` passes through this handler.

### Why Use This?

Without this, **every controller action** needs to manually check:

```csharp
if (response.StatusCode == HttpStatusCode.Unauthorized)
{
    return RedirectToAction("Login", "Account");
}
```

That's **repetitive and messy**. The `DelegatingHandler` does it **once for all API calls**.

---

### Handlers/ApiResponseHandler.cs

```csharp
using System.Net;

namespace YourMVCProject.Handlers
{
    public class ApiResponseHandler : DelegatingHandler
    {
        private readonly IHttpContextAccessor _httpContextAccessor;

        public ApiResponseHandler(IHttpContextAccessor httpContextAccessor)
        {
            _httpContextAccessor = httpContextAccessor;
        }

        protected override async Task<HttpResponseMessage> SendAsync(
            HttpRequestMessage request, CancellationToken cancellationToken)
        {
            // Step 1: Let the API call happen
            var response = await base.SendAsync(request, cancellationToken);

            var context = _httpContextAccessor.HttpContext;

            if (context == null)
                return response;

            // Step 2: Check the API response status code
            switch (response.StatusCode)
            {
                case HttpStatusCode.Unauthorized: // 401
                    // Token expired or missing → clear session and go to login
                    context.Session.Clear();
                    context.Response.Redirect("/Account/Login");
                    break;

                case HttpStatusCode.Forbidden: // 403
                    // User doesn't have permission
                    context.Response.Redirect("/Error/Forbidden");
                    break;

                case HttpStatusCode.NotFound: // 404
                    // API endpoint or resource not found
                    context.Response.Redirect("/Error/NotFound");
                    break;

                case HttpStatusCode.InternalServerError: // 500
                    // API crashed
                    context.Response.Redirect("/Error/ServerError");
                    break;
            }

            return response;
        }
    }
}
```

**How It Works:**

1. `base.SendAsync()` → Sends the actual HTTP request to the API
2. We read `response.StatusCode` → Check what API returned
3. Based on status code → Redirect to appropriate error page or login
4. For `401` → We also **clear the session** so stale tokens are removed

---

## 3. Program.cs – Register Everything

### Updated Program.cs

```csharp
var builder = WebApplication.CreateBuilder(args);

builder.Services.AddControllersWithViews();
builder.Services.AddHttpContextAccessor();
builder.Services.AddSession();

// Register the DelegatingHandler
builder.Services.AddTransient<ApiResponseHandler>();

// Register HttpClient WITH the handler attached
builder.Services.AddHttpClient("ApiClient")
    .AddHttpMessageHandler<ApiResponseHandler>();

var app = builder.Build();

// Register the Global Exception Middleware (MUST be first)
app.UseMiddleware<GlobalExceptionMiddleware>();

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

**Important Notes:**

- `AddTransient<ApiResponseHandler>()` → Registers the handler in DI
- `AddHttpClient("ApiClient").AddHttpMessageHandler<>()` → Attaches handler to named HttpClient
- `UseMiddleware<GlobalExceptionMiddleware>()` → Must be **before** other middleware to catch all errors
- `UseStatusCodePagesWithReExecute` → Handles status codes like 404 for MVC routes (not API calls)

---

## 4. Update Controllers to Use Named HttpClient

Since we registered a **named HttpClient** (`"ApiClient"`), update your controllers:

### Before (without interceptor)

```csharp
private HttpClient GetClient()
{
    var client = _factory.CreateClient(); // default client, no handler
    var token = HttpContext.Session.GetString("JWToken");
    client.DefaultRequestHeaders.Authorization = new AuthenticationHeaderValue("Bearer", token);
    return client;
}
```

### After (with interceptor)

```csharp
private HttpClient GetClient()
{
    var client = _factory.CreateClient("ApiClient"); // uses our handler!
    var token = HttpContext.Session.GetString("JWToken");
    client.DefaultRequestHeaders.Authorization = new AuthenticationHeaderValue("Bearer", token);
    return client;
}
```

**Only change:** `CreateClient()` → `CreateClient("ApiClient")`

Now **every API call** automatically goes through our `ApiResponseHandler`.

---

## 5. Folder Structure

After adding the middleware and handler, your project structure should look like:

```
YourMVCProject/
├── Controllers/
│   ├── AccountController.cs
│   ├── DashboardController.cs
│   ├── ErrorController.cs         ← New (Part 2)
│   └── StaffController.cs
├── Handlers/
│   └── ApiResponseHandler.cs      ← New
├── Middleware/
│   └── GlobalExceptionMiddleware.cs ← New
├── Models/
│   └── ...
├── Views/
│   ├── Error/                      ← New (Part 2)
│   └── ...
├── Program.cs                      ← Updated
└── appsettings.json
```

---

## Quick Reference

| Status Code | Meaning | What Happens |
|---|---|---|
| `200` | Success | Normal flow continues |
| `401` | Unauthorized | Session cleared → Redirect to Login |
| `403` | Forbidden | Redirect to Forbidden page |
| `404` | Not Found | Redirect to Not Found page |
| `500` | Server Error | Redirect to Server Error page |
| Exception | Unhandled error | Global Middleware catches → Error page |

---

## Summary

- **GlobalExceptionMiddleware** → Catches **any unhandled exception** in MVC
- **ApiResponseHandler** → Intercepts **every API call** and handles error status codes
- Named HttpClient (`"ApiClient"`) → Makes sure the handler is used everywhere
- No need to write error-checking code in **every controller action**
- **Part 2** covers creating the actual Error pages and views
