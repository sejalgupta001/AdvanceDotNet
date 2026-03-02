# Making ASP.NET Core Web API Live

This document explains the **complete process of hosting database + deploying ASP.NET Core Web API live using Docker and Render**.

---

# 🌐 Part 1 — Host Database on Live Server (Somee.com)

Before deploying your API, you must host your **SQL Server database online**, so your API can access it from anywhere.

## Step 1 — Watch Complete Database Hosting Tutorial

Follow the complete video tutorial below to host SQL Server database on Somee:

👉 **Database Hosting Video Guide (Must Watch)**

🔗 [https://www.youtube.com/watch?v=XS2uTMzKhtI](https://www.youtube.com/watch?v=XS2uTMzKhtI)

---

## Step 2 — Create Account on Somee

1. Visit **Somee.com**.
2. Register using Email ID.
3. Verify your account.

---

## Step 3 — Create Database

Inside Somee Dashboard:

1. Go to **SQL Server Section**.
2. Click **Create Database**.
3. Enter Database Name.
4. Save credentials.

You will receive:

- Server Name
- Database Name
- Username
- Password

---

## Step 4 — Import Database

Using SQL Server Management Studio (SSMS):

1. Connect using Somee credentials.
2. Right Click Database → Tasks.
3. Import `.bak` OR Run SQL Script.

---

## Step 5 — Update Connection String

Update inside:

`appsettings.json`

Example:

```json
"ConnectionStrings": {
 "DefaultConnection": "Server=SQLXXXX.somee.com;Database=YourDB;
 User Id=YourUser;Password=YourPassword;
 TrustServerCertificate=True;"
}
```

✅ Now database is live and accessible publicly.

---

# 🚀 Part 2 — Deploy ASP.NET Core API Live Using Docker + Render

Now deploy your Web API.

---

# Step-by-Step Guide to Getting Your App Live

---

# ✅ 1. Prepare your Dockerfile

Create a file named:

```
Dockerfile
```

(no extension)

Place it inside your **Project Root Folder**.

> Replace `YourProjectName` with your actual project name.

```dockerfile
# Stage 1: Build
FROM mcr.microsoft.com/dotnet/sdk:8.0 AS build
WORKDIR /src

COPY ["YourProjectName/YourProjectName.csproj", "YourProjectName/"]
RUN dotnet restore "YourProjectName/YourProjectName.csproj"

COPY . .
WORKDIR "/src/YourProjectName"
RUN dotnet build "YourProjectName.csproj" -c Release -o /app/build

# Stage 2: Publish
FROM build AS publish
RUN dotnet publish "YourProjectName.csproj" -c Release -o /app/publish /p:UseAppHost=false

# Stage 3: Final Runtime
FROM mcr.microsoft.com/dotnet/aspnet:8.0 AS final
WORKDIR /app
COPY --from=publish /app/publish .

ENV ASPNETCORE_URLS=http://+:10000
EXPOSE 10000

ENTRYPOINT ["dotnet", "YourProjectName.dll"]
```

---

# ✅ 2. Add .dockerignore

Create:

```
.dockerignore
```

Add:

```text
**/.git
**/.vs
**/bin
**/obj
```

This keeps Docker image clean and fast.

---

# ✅ 3. Enable Swagger in Production

Open:

```
Program.cs
```

Move Swagger outside Development environment block.

Example:

```csharp
app.UseSwagger();

app.UseSwaggerUI(options =>
{
 options.SwaggerEndpoint("/swagger/v1/swagger.json", "v1");
 options.RoutePrefix = string.Empty;
});
```

✅ Swagger becomes public landing page.

---

# ✅ 4. Publishing via Visual Studio (GitHub Upload)

## Prerequisites

Create GitHub account if not available.

---

## Steps

Inside Visual Studio:

1. Click:

```
Git → Create Git Repository
```

---

2. Sign In to GitHub.

---

3. Enter:

- Repository Name.

---

4. Select:

```
Private Repository
```

(Optional)

---

5. Click:

```
Create and Push
```

Visual Studio automatically:

- Initializes Git.
- Commits project.
- Uploads code.

---

## Verification

Login GitHub dashboard.

Confirm repository is visible.

Copy Repository URL.

---

# ✅ 5. Deploy on Render

Now deploy API.

Platform used:

👉 **Render**

---

## Steps

1. Login:

[https://dashboard.render.com/](https://dashboard.render.com/)

---

2. Click:

```
New +
```

Select:

```
Web Service
```

---

3. Connect GitHub Repository.

---

## Configuration

Fill:

- Name → API Name.

- Language → Docker.

- Region → Closest to users.

- Instance Type →

```
Free (Testing Purpose)
```

(Note: Sleeps after 15 min inactivity.)

---

### Dockerfile Path

Example:

```
YourProjectName/Dockerfile
```

---

4. Click:

```
Create Web Service
```

---

## Deployment Process

Render will automatically:

- Build Docker Image.
- Install .NET Runtime.
- Publish API.
- Generate Live URL.

Example:

```
https://yourapi.onrender.com
```

---

# ✅ Final Testing

Open:

```
https://yourapi.onrender.com
```

Swagger UI should appear.

Test APIs.
