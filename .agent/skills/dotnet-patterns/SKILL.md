---
name: dotnet-patterns
description: Core C# and .NET development patterns, idioms, and CLI mastery.
---

# .NET Patterns & Practices

## 📚 Knowledge Source (MANDATORY)

**Rule:** Before implementing or recommending any pattern, you MUST query the `microsoft-learn` MCP to verify the latest versions (.NET 8/9/10) and best practices.
**Citation:** You MUST cite the MS Learn URL for any major architectural decision or API usage.


This skill dictates how to write idiomatic C# and manage .NET projects effectively.

## 1. C# Coding Standards

### Modern Syntax (C# 12+)
- **File Scoped Namespaces**: Always use implementation.
  ```csharp
  namespace MyProject.Services; // Good
  namespace MyProject.Services { ... } // Avoid
  ```
- **Top-level Statements**: Limit to `Program.cs` only.
- **Implicit Usings**: Enable in `.sproj`.

### Data Types
- **Records**: Use `record` for DTOs, events, and value objects.
  ```csharp
  public record CreateUserRequest(string Email, string Password);
  ```
- **Var**: Use `var` when type is obvious from the right-hand side (e.g., `new`, `Cast`).
  ```csharp
  var user = new User(); // Good
  int count = GetCount(); // Good
  var data = GetData(); // Avoid - what is data?
  ```

### Null Handling
- **Nullable Context**: Must be enabled `<Nullable>enable</Nullable>`.
- **Parameters**: Don't check for null if typed as non-null. Trust the compiler mostly, validate at boundaries.
- **Defaults**: `string?` for optional, `string` for required.

## 2. CLI Mastery

### Project Management
```powershell
# Create new solution
dotnet new sln -n MySolution

# Create projects
dotnet new webapi -n MyApi
dotnet new classlib -n MyCore

# Add to solution
dotnet sln add MyApi/MyApi.csproj
dotnet sln add MyCore/MyCore.csproj

# Add references
dotnet add MyApi/MyApi.csproj reference MyCore/MyCore.csproj
```

### Tooling
```powershell
# Install tools
dotnet tool install --global dotnet-ef
dotnet tool install --global dotnet-outdated

# EF Migrations
dotnet ef migrations add InitialCreate -p Infrastructure -s Api
dotnet ef database update -p Infrastructure -s Api
```

## 3. Dependency Injection Patterns

### Service Lifetimes
- **Transient**: Lightweight, stateless services.
- **Scoped**: Database contexts, User contexts (Request scope).
- **Singleton**: Caches, configuration, stateless computations.

### Registration
- Use extension methods on `IServiceCollection` to organize registrations.
  ```csharp
  // Infrastructure/DependencyInjection.cs
  public static IServiceCollection AddInfrastructure(this IServiceCollection services, IConfiguration config)
  {
      services.AddDbContext<AppDbContext>(...);
      return services;
  }
  ```

## 4. Configuration Options Diagram
Bind configuration to strongly typed objects.

```csharp
// Definition
public class EmailSettings
{
    public const string SectionName = "Email";
    public string SmtpServer { get; init; } = string.Empty;
}

// Registration
builder.Services.Configure<EmailSettings>(builder.Configuration.GetSection(EmailSettings.SectionName));

// Usage
public class EmailService(IOptions<EmailSettings> options)
{
    private readonly EmailSettings _settings = options.Value;
}
```
