---
name: aspnet-core-expert
description: Deep expertise in building Web APIs, middleware, and web applications with ASP.NET Core.
---

# ASP.NET Core Expertise

## 1. API Architecture

### Minimal APIs vs Controllers
- **Minimal APIs**: Default for microservices, simple APIs, and high performance.
- **Controllers**: Use for large, organized applications requiring complex filters/attributes or when migrating legacy apps.

### Project Structure (Clean Architecture)
```text
src/
  MyProject.Domain/       # Entities, Value Objects, Domain Exceptions (No Depends)
  MyProject.Application/  # Use Cases, Interfaces, DTOs (Depends on Domain)
  MyProject.Infrastructure/# DbContext, External Services (Depends on Application)
  MyProject.Api/          # Controllers, Program.cs (Depends on App & Infra)
```

## 2. Critical Middleware

### Global Exception Handling
Use `IExceptionHandler` in .NET 8+.

```csharp
public class GlobalExceptionHandler : IExceptionHandler
{
    public async ValueTask<bool> TryHandleAsync(HttpContext context, Exception exception, CancellationToken ct)
    {
        var problemDetails = new ProblemDetails { Status = 500, Title = "Server Error" };
        context.Response.StatusCode = 500;
        await context.Response.WriteAsJsonAsync(problemDetails, ct);
        return true;
    }
}

// Program.cs
builder.Services.AddExceptionHandler<GlobalExceptionHandler>();
builder.Services.AddProblemDetails();
app.UseExceptionHandler();
```

### Validation Pipeline (FluentValidation)
Don't validate in controllers. Use a pipeline behavior (if using MediatR) or filter.
```csharp
public class ValidationBehavior<TRequest, TResponse> : IPipelineBehavior<TRequest, TResponse>
{
    // ... logic to run validators and throw ValidationException
}
```

## 3. Performance & Security

### JSON Serialization (`System.Text.Json`)
- Use Source Generators for high performance.
- Configure `JsonSerializerOptions.Web` defaults.

### Security
- **CORS**: Be specific. Never allow `*` in production.
- **Authentication**: Prefer JWT Bearer for APIs.
- **Authorization**: Use Policy-based authorization.
  ```csharp
  services.AddAuthorization(options => 
      options.AddPolicy("AdminOnly", policy => policy.RequireClaim("role", "Admin")));
  ```

## 4. SignalR & Real-time
- Use `Hub<T>` for strongly typed clients.
- Scale out using Redis backplane.
