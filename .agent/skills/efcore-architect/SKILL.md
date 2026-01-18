---
name: efcore-architect
description: Advanced Entity Framework Core patterns, performance tuning, and database modeling.
---

# EF Core Architect

## 1. Performance Guidelines

### Query Optimization
- **AsNoTracking**: MANDATORY for read-only queries.
  ```csharp
  var products = await context.Products.AsNoTracking().ToListAsync();
  ```
- **Split Queries**: Use for queries with multiple `.Include()` to avoid cartesian explosion.
  ```csharp
  var order = await context.Orders
      .Include(x => x.Items)
      .Include(x => x.Customer)
      .AsSplitQuery()
      .FirstOrDefaultAsync(x => x.Id == id);
  ```
- **Projections**: Fetch only what you need.
  ```csharp
  var dto = await context.Users
      .Select(x => new UserDto(x.Id, x.Name)) // Only selects Id and Name columns
      .ToListAsync();
  ```

### Bulk Operations (Net 7+)
Stop fetching entities just to update/delete them.
```csharp
// Bad
var users = await ctx.Users.Where(u => u.Inactive).ToListAsync();
foreach(var u in users) ctx.Users.Remove(u);
await ctx.SaveChangesAsync();

// Good
await ctx.Users.Where(u => u.Inactive).ExecuteDeleteAsync();
```

## 2. Modeling Best Practices

### Configuration
Use `IEntityTypeConfiguration<T>` classes, DO NOT clutter `DbContext.OnModelCreating`.

```csharp
public class ProductConfiguration : IEntityTypeConfiguration<Product>
{
    public void Configure(EntityTypeBuilder<Product> builder)
    {
        builder.HasKey(x => x.Id);
        builder.Property(x => x.Sku).HasMaxLength(50).IsRequired();
        builder.HasIndex(x => x.Sku).IsUnique();
        
        // Value Objects
        builder.OwnsOne(x => x.Price, money => {
            money.Property(p => p.Amount).HasColumnName("PriceAmount");
            money.Property(p => p.Currency).HasColumnName("PriceCurrency");
        });
    }
}
```

### Enumerations
Convert Enums to string in DB for readability, or int for size.
```csharp
builder.Property(e => e.Status).HasConversion<string>();
```

## 3. Migrations & DevOps
- **Idempotent Scripts**: Generate scripts that can run safely on any DB version.
  ```powershell
  dotnet ef migrations script --idempotent --output migration.sql
  ```
- **Code-First**: Prefer Code-First, but respect the DB.
- **Seeds**: separate seeding logic from migration logic if possible, or use data migrations.

## 4. Diagnostics
- Enable sensitive data logging ONLY in Development.
- Use `TagWith` to trace queries in logs.
  ```csharp
  var data = await context.Users.TagWith("GetUserById").ToListAsync();
  ```
