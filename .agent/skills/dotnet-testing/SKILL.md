---
name: dotnet-testing
description: Testing strategies for .NET applications using xUnit, Moq/NSubstitute, and Testcontainers.
---

# .NET Testing Strategy

## 📚 Knowledge Source (MANDATORY)

**Rule:** Before implementing or recommending any pattern, you MUST query the `microsoft-learn` MCP to verify the latest versions (.NET 8/9/10) and best practices.
**Citation:** You MUST cite the MS Learn URL for any major architectural decision or API usage.


## 1. Testing Pyramid
- **Unit Tests**: Fast, isolated, mocks external deps. (xUnit + FluentAssertions)
- **Integration Tests**: Real database, real HTTP stack. (Testcontainers + interactions)
- **E2E Tests**: Playwright.

## 2. Stack Recommendation
- **Framework**: `xUnit` (Standard)
- **Assertions**: `FluentAssertions` (Readable)
- **Mocking**: `NSubstitute` (Simpler syntax than Moq)
- **Data Generation**: `Bogus`
- **Containers**: `Testcontainers` (For Docker dependencies)

## 3. Implementation Patterns

### Unit Test (Application Layer)
```csharp
public class CreateUserHandlerTests
{
    private readonly IUserRepository _repo = Substitute.For<IUserRepository>();
    
    [Fact]
    public async Task Handle_ShouldReturnId_WhenValid()
    {
        // Arrange
        var command = new CreateUserCommand("test@test.com");
        _repo.AddAsync(Arg.Any<User>()).Returns(Task.CompletedTask);
        
        var handler = new CreateUserHandler(_repo);
        
        // Act
        var result = await handler.Handle(command, CancellationToken.None);
        
        // Assert
        result.Should().BePositive();
        await _repo.Received(1).AddAsync(Arg.Is<User>(u => u.Email == command.Email));
    }
}
```

### Integration Test (Using WebApplicationFactory)
```csharp
public class HealthCheckTests : IClassFixture<WebApplicationFactory<Program>>
{
    private readonly HttpClient _client;

    public HealthCheckTests(WebApplicationFactory<Program> factory)
    {
        _client = factory.CreateClient();
    }

    [Fact]
    public async Task Health_ReturnsOk()
    {
        var response = await _client.GetAsync("/health");
        response.EnsureSuccessStatusCode();
    }
}
```

## 4. Best Practices
- **Naming**: `Method_Scenario_ExpectedResult`
- **AAA**: Arrange, Act, Assert comments.
- **Do Not Logic**: Test code should be flat. No branches/loops in tests usually.
- **One Assert**: ideally one logical assertion per test.
