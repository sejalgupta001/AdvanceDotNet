# Connecting the .NET 8 API with the React (Vite) Frontend

This repository contains:

- **Backend API**: `cardprintingAPI/` (ASP.NET Core **.NET 8**)
- **Frontend**: `card printing/` (React + TypeScript + **Vite**)

The React app calls the API under `/api/*` endpoints.

## Packages used

### Frontend packages

- `axios`
- `cors`
- `react`
- `react-dom`
- `react-router-dom`

### Backend packages

- `Microsoft.EntityFrameworkCore`
- `Microsoft.EntityFrameworkCore.Design`
- `Microsoft.EntityFrameworkCore.SqlServer`
- `Microsoft.EntityFrameworkCore.Tools`
- `Newtonsoft.Json`
- `Swashbuckle.AspNetCore`

---

## Backend (ASP.NET Core 8) API

### How to run the API

- Open the solution: `cardprintingAPI/cardprintingAPI.sln`
- Run the API project.

From `cardprintingAPI/Properties/launchSettings.json` the development URLs are:

- **HTTPS**: `https://localhost:7090`
- **HTTP**: `http://localhost:5222`

Swagger is enabled in development:

- `https://localhost:7090/swagger`

### API base path

Controllers use the route:

- `api/[controller]`

Examples:

- `GET https://localhost:7090/api/UserDetail`
- `GET https://localhost:7090/api/CardTemplate`
- `GET https://localhost:7090/api/Payment`

---

## Examples

### Example: What is our database?

The backend API uses:

- **Database**: Microsoft SQL Server (Local SQL Express)
- **Database name**: `CardPrintingDB`

The connection string is configured in:

- `cardprintingAPI/appsettings.json`

Example:

```json
"ConnectionStrings": {
  "DefaultConnection": "Server=localhost\\SQLEXPRESS;Database=CardPrintingDB;Trusted_Connection=True;TrustServerCertificate=True;"
}
```

The API connects via Entity Framework Core in:

- `cardprintingAPI/Program.cs`

Example:

```csharp
builder.Services.AddDbContext<CardPrintingDbContext>(options =>
    options.UseSqlServer(builder.Configuration.GetConnectionString("DefaultConnection")));
```

This means:

- Run SQL Server Express locally
- Ensure the `CardPrintingDB` database exists
- Then start the API and it will connect using `DefaultConnection`

### Example: How we connect API with a React page

Below is a simple end-to-end example:

1. A small service function that calls the API
2. A React page component that loads data from the API and renders it

#### 1) Service file example

Create (or follow the same pattern as) a service like this:

```ts
// Example: src/services/userDetailService.ts

const API_BASE_URL = import.meta.env.VITE_API_BASE_URL;

export interface UserDetail {
  userId: number;
  username: string;
  email: string;
  isPremium: boolean;
  isAdmin: boolean;
}

export async function getAllUsers(): Promise<UserDetail[]> {
  const response = await fetch(`${API_BASE_URL}/UserDetail`, {
    method: 'GET',
    headers: {
      'Content-Type': 'application/json',
    },
  });

  if (!response.ok) {
    const errorText = await response.text();
    throw new Error(`HTTP ${response.status}: ${errorText}`);
  }

  return response.json();
}
```

#### 2) React page/component example

```tsx
import { useEffect, useState } from 'react';
import type { UserDetail } from '../services/userDetailService';
import { getAllUsers } from '../services/userDetailService';

export default function UsersPage() {
  const [users, setUsers] = useState<UserDetail[]>([]);
  const [loading, setLoading] = useState(true);
  const [error, setError] = useState<string | null>(null);

  useEffect(() => {
    let mounted = true;

    (async () => {
      try {
        const data = await getAllUsers();
        if (mounted) setUsers(data);
      } catch (e) {
        if (mounted) setError(e instanceof Error ? e.message : 'Unknown error');
      } finally {
        if (mounted) setLoading(false);
      }
    })();

    return () => {
      mounted = false;
    };
  }, []);

  if (loading) return <div>Loading...</div>;
  if (error) return <div style={{ color: 'red' }}>{error}</div>;

  return (
    <div>
      <h1>Users</h1>
      <ul>
        {users.map((u) => (
          <li key={u.userId}>
            {u.username} ({u.email})
          </li>
        ))}
      </ul>
    </div>
  );
}
```

For this to work locally:

- Ensure the API is running on `https://localhost:7090`
- Set `VITE_API_BASE_URL=https://localhost:7090/api` in `card printing/.env.local`
- Then the React page will call `GET /api/UserDetail`

### CORS (important for React)

**What is CORS?**

CORS (Cross-Origin Resource Sharing) is a browser security feature. When your React app (`http://localhost:5173`) tries to call the API (`https://localhost:7090`), the browser blocks it by default because they are different origins. CORS headers tell the browser it's okay to allow the request.

**Current setup in `cardprintingAPI/Program.cs`:**

```csharp
// Add CORS policy
builder.Services.AddCors(options =>
{
    options.AddPolicy("AllowReactApp", policy =>
    {
        policy.AllowAnyOrigin()
              .AllowAnyHeader()
              .AllowAnyMethod();
    });
});

// ... after building the app

// Apply CORS (must be before MapControllers)
app.UseCors("AllowReactApp");
app.MapControllers();
```

This means the React dev server can call the API without browser CORS blocking.

**Production CORS (recommended):**

For production, restrict to your frontend domain:

```csharp
policy.WithOrigins("https://app.yourdomain.com")
      .AllowAnyHeader()
      .AllowAnyMethod();
```

### Common CORS Errors

| Error | Cause | Fix |
|-------|-------|-----|
| `No 'Access-Control-Allow-Origin' header present` | CORS not enabled on API | Add `app.UseCors()` in Program.cs |
| `Origin 'http://localhost:5173' is not allowed` | Origin not in allowed list | Add your frontend URL to `WithOrigins()` or use `AllowAnyOrigin()` for dev |
| `Request header field content-type is not allowed` | Missing `AllowAnyHeader()` | Add `.AllowAnyHeader()` to policy |
| `Method PUT is not allowed` | Missing `AllowAnyMethod()` | Add `.AllowAnyMethod()` to policy |
| `The value of the 'Access-Control-Allow-Credentials' header is ''` | Using credentials without proper config | Add `.AllowCredentials()` and use `WithOrigins()` (not `AllowAnyOrigin()`) |
| Preflight (OPTIONS) request fails | CORS middleware not applied correctly | Ensure `UseCors()` is called before `MapControllers()` |

**Important:** Middleware order matters! `UseCors()` must come after `UseRouting()` and before `UseAuthorization()` and `MapControllers()`.

---

## Frontend (React + Vite)

### How to run the React app

In the `card printing/` folder:

- Install dependencies: `npm install`
- Run dev server: `npm run dev`

Vite will print the local URL (commonly `http://localhost:5173`).

### Where the API URL is configured (current code)

Some frontend services hardcode the API base URL as:

- `https://localhost:7090/api`

Examples:

- `card printing/src/services/cardTemplateService.ts`
- `card printing/src/services/paymentService.ts`

So, to connect React to the API locally:

- Make sure the API is running on `https://localhost:7090`
- Then start the React dev server

---

## Recommended setup (use environment variables)

Hardcoding `https://localhost:7090/api` works locally, but it is harder to change for production.

A cleaner setup is:

1. Create a Vite env variable
2. Read it in your services

### 1) Create `.env.local` in the React project

Create this file:

- `card printing/.env.local`

With content:

```env
VITE_API_BASE_URL=https://localhost:7090/api
```

> In Vite, only variables starting with `VITE_` are exposed to the browser.

### 2) Use the env variable in service files

Replace hardcoded lines like:

```ts
const API_BASE_URL = 'https://localhost:7090/api';
```

With:

```ts
const API_BASE_URL = import.meta.env.VITE_API_BASE_URL;
```

Now switching API environments is just changing `.env.*` files.

---

## HTTPS certificate note (common issue)

Because the API runs on HTTPS (`https://localhost:7090`), the browser must trust the development certificate.

If your React app shows errors like:

- `NET::ERR_CERT_AUTHORITY_INVALID`
- `Failed to fetch`

Then:

- Trust the ASP.NET Core development certificate (Visual Studio usually does this automatically).

---

## Option: Use a Vite dev proxy (avoids CORS + avoids hardcoding full URLs)

Instead of calling `https://localhost:7090/api/...` from the browser, you can proxy `/api` requests via Vite.

### 1) Update `card printing/vite.config.ts`

Add a `server.proxy` section like:

```ts
import { defineConfig } from 'vite';
import react from '@vitejs/plugin-react';

export default defineConfig({
  plugins: [react()],
  optimizeDeps: {
    exclude: ['lucide-react'],
  },
  server: {
    proxy: {
      '/api': {
        target: 'https://localhost:7090',
        changeOrigin: true,
        secure: false,
      },
    },
  },
});
```

### 2) Call the API using relative URLs

Then your frontend can call:

- `/api/CardTemplate`

Instead of:

- `https://localhost:7090/api/CardTemplate`

This is especially convenient for local development.

---

## Production deployment notes

### Recommended approach

- **Backend** deployed as: `https://api.yourdomain.com`
- **Frontend** deployed as: `https://app.yourdomain.com`

Then configure:

- React: `VITE_API_BASE_URL=https://api.yourdomain.com/api`
- API CORS: allow origin `https://app.yourdomain.com`

### CORS in production

Current API CORS uses `AllowAnyOrigin()`. For production you should restrict it to your frontend domain for better security.

---

## Quick checklist

- **API running?** `https://localhost:7090/swagger`
- **React running?** `npm run dev`
- **Frontend API URL correct?** points to `https://localhost:7090/api`
- **CORS enabled?** Already enabled in `Program.cs`
- **HTTPS trusted?** Browser trusts ASP.NET dev certificate

