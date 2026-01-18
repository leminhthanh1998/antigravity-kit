---
name: dotnet-architecture
description: High-level architectural patterns for .NET systems.
---

# .NET Architecture

## 1. Clean Architecture (The Standard)

### Layers
1. **Domain (Core)**
   - Enterprise logic, Entities, Value Objects, Domain Events.
   - NO dependencies.
   - "Pure C#".

2. **Application (Use Cases)**
   - Orchestration logic (Commands/Queries).
   - Interfaces (Abstractions) for Infrastructure.
   - Depends only on Domain.

3. **Infrastructure (Adapters)**
   - Implementations of Application interfaces (EF Core Repos, Email Senders).
   - Depends on Application and external libs.

4. **Presentation (API/Web)**
   - Entry point.
   - Depends on Application and Infrastructure (for DI registration).

### Folder Structure
```
Solution.sln
├── src
│   ├── Solution.Domain
│   ├── Solution.Application
│   ├── Solution.Infrastructure
│   └── Solution.Web
└── tests
    ├── Solution.Domain.Tests
    └── ...
```

## 2. Vertical Slice Architecture (The Alternative)
Best for rapid development and non-complex domains. Orgasnize by FEATURE, not layer.

### Folder Structure
```
src/Solution.Api/
  Features/
    Products/
      CreateProduct.cs (Contains Request, Handler, Validator, Endpoint)
      GetProduct.cs
      DeleteProduct.cs
    Orders/
      PlaceOrder.cs
```
**Benefits**: High cohesion. Changing a feature only touches one folder.

## 3. Microservices vs Modular Monolith
- Default to **Modular Monolith** first.
- Enforce module boundaries using stricter namespaces or independent projects.
- Extract to Microservices only when scaling requirements dictate it (independent deployment/scaling needed).

## 4. Cross-Cutting Concerns
- **Validation**: MediatR Pipeline Behavior.
- **Logging**: Serilog Request Logging middleware.
- **Auth**: Centralized Identity Provider (Keycloak/Duende/Auth0).
