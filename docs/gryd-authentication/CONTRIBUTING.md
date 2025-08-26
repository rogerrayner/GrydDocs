# Contributing to Gryd Authentication Framework

Thank you for your interest in contributing to the Gryd Authentication Framework! This document provides guidelines and information for contributors.

## 🤝 How to Contribute

### Reporting Issues

Before creating an issue, please:

1. **Search existing issues** to avoid duplicates
2. **Use the issue template** when available
3. **Provide detailed information** including:
   - .NET version
   - Framework version
   - Steps to reproduce
   - Expected vs actual behavior
   - Error messages and stack traces

### Suggesting Features

For new features:

1. **Check the roadmap** to see if it's already planned
2. **Open a discussion** before implementing large changes
3. **Provide use cases** and examples
4. **Consider backwards compatibility**

### Code Contributions

#### Prerequisites

- .NET 9.0 SDK
- Git
- Your favorite IDE (Visual Studio, VS Code, Rider)

#### Setup Development Environment

```bash
# Clone the repository
git clone https://github.com/your-org/GrydAuthenticationFramework.git
cd GrydAuthenticationFramework

# Restore dependencies
dotnet restore

# Run tests
dotnet test

# Build solution
dotnet build
```

#### Development Workflow

1. **Fork the repository**
2. **Create a feature branch**
   ```bash
   git checkout -b feature/my-new-feature
   ```
3. **Make your changes**
4. **Add tests** for new functionality
5. **Run tests** to ensure everything works
6. **Commit with descriptive messages**
7. **Push to your fork**
8. **Create a Pull Request**

## 📝 Coding Standards

### General Guidelines

- Follow **Clean Architecture** principles
- Adhere to **SOLID** principles
- Use **meaningful names** for classes, methods, and variables
- Write **self-documenting code**
- Add **XML documentation** for public APIs

### Code Style

We follow the standard .NET coding conventions:

```csharp
// ✅ Good
public class UserService : IUserService
{
    private readonly IUserRepository _userRepository;
    private readonly ILogger<UserService> _logger;

    public UserService(
        IUserRepository userRepository,
        ILogger<UserService> logger)
    {
        _userRepository = userRepository ?? throw new ArgumentNullException(nameof(userRepository));
        _logger = logger ?? throw new ArgumentNullException(nameof(logger));
    }

    public async Task<User> GetUserByIdAsync(Guid userId)
    {
        _logger.LogInformation("Getting user with ID {UserId}", userId);
        
        var user = await _userRepository.GetByIdAsync(userId);
        if (user == null)
        {
            throw new UserNotFoundException($"User with ID {userId} not found");
        }

        return user;
    }
}
```

### Naming Conventions

- **Classes**: PascalCase (`UserService`, `AuthenticationController`)
- **Methods**: PascalCase (`GetUserAsync`, `ValidatePassword`)
- **Properties**: PascalCase (`Email`, `CreatedAt`)
- **Fields**: camelCase with underscore prefix (`_userRepository`, `_logger`)
- **Parameters**: camelCase (`userId`, `email`)
- **Constants**: PascalCase (`MaxRetryAttempts`)

### File Organization

```
src/
├── Domain/
│   ├── Entities/
│   ├── Events/
│   ├── Exceptions/
│   └── Repositories/
├── Application/
│   ├── Commands/
│   ├── Queries/
│   ├── Handlers/
│   ├── DTOs/
│   └── Behaviors/
├── Infrastructure/
│   ├── Data/
│   ├── Services/
│   └── Repositories/
└── Presentation/
    ├── Controllers/
    ├── Middleware/
    └── Attributes/
```

## 🧪 Testing Guidelines

### Test Structure

We use the **Arrange-Act-Assert** pattern:

```csharp
[Fact]
public async Task GetUserByIdAsync_WithValidId_ReturnsUser()
{
    // Arrange
    var userId = Guid.NewGuid();
    var expectedUser = new User { Id = userId, Email = "test@example.com" };
    
    _mockUserRepository
        .Setup(x => x.GetByIdAsync(userId))
        .ReturnsAsync(expectedUser);

    // Act
    var result = await _userService.GetUserByIdAsync(userId);

    // Assert
    result.Should().NotBeNull();
    result.Id.Should().Be(userId);
    result.Email.Should().Be("test@example.com");
}
```

### Test Categories

- **Unit Tests**: Test individual components in isolation
- **Integration Tests**: Test component interactions
- **End-to-End Tests**: Test complete user scenarios

### Test Naming

Use descriptive names that explain:
- What is being tested
- Under what conditions
- What the expected outcome is

```csharp
// ✅ Good
[Fact]
public async Task LoginAsync_WithInvalidCredentials_ThrowsInvalidCredentialsException()

// ❌ Bad
[Fact]
public async Task LoginTest()
```

### Mocking Guidelines

- Use **Moq** for mocking dependencies
- Mock external dependencies (databases, HTTP clients, etc.)
- Don't mock the system under test
- Verify important interactions

```csharp
// ✅ Good - Verify important interactions
_mockUserRepository.Verify(
    x => x.UpdateAsync(It.Is<User>(u => u.LastLoginAt != null)), 
    Times.Once);

// ❌ Avoid over-verification
_mockLogger.Verify(
    x => x.LogInformation(It.IsAny<string>(), It.IsAny<object[]>()), 
    Times.Once);
```

## 📚 Documentation

### XML Documentation

All public APIs must have XML documentation:

```csharp
/// <summary>
/// Authenticates a user with email and password.
/// </summary>
/// <param name="email">The user's email address.</param>
/// <param name="password">The user's password.</param>
/// <returns>A JWT token if authentication is successful.</returns>
/// <exception cref="InvalidCredentialsException">
/// Thrown when the email or password is invalid.
/// </exception>
public async Task<string> LoginAsync(string email, string password)
{
    // Implementation
}
```

### README Updates

When adding new features:

1. Update the main README.md
2. Add examples to the documentation
3. Update the CHANGELOG.md
4. Consider adding to the troubleshooting guide

## 🚀 Pull Request Guidelines

### Before Submitting

- [ ] All tests pass
- [ ] Code follows style guidelines
- [ ] New code has tests
- [ ] Documentation is updated
- [ ] No breaking changes (or clearly documented)

### PR Description Template

```markdown
## Description
Brief description of changes

## Type of Change
- [ ] Bug fix
- [ ] New feature
- [ ] Breaking change
- [ ] Documentation update

## Testing
- [ ] Unit tests added/updated
- [ ] Integration tests added/updated
- [ ] Manual testing completed

## Checklist
- [ ] Code follows style guidelines
- [ ] Self-review completed
- [ ] Documentation updated
- [ ] No new warnings or errors
```

### Review Process

1. **Automated checks** must pass (build, tests, linting)
2. **Peer review** by at least one maintainer
3. **Address feedback** promptly and professionally
4. **Squash commits** before merging (if requested)

## 🏗️ Architecture Guidelines

### Layer Dependencies

```
Presentation → Application → Domain
Infrastructure → Application → Domain
```

**Rules:**
- Domain layer has no dependencies
- Application layer only depends on Domain
- Infrastructure implements Application interfaces
- Presentation coordinates everything

### Adding New Features

#### 1. Domain First
Start by modeling the domain:

```csharp
// Domain/Entities/NewEntity.cs
public class NewEntity : BaseEntity
{
    public string Name { get; private set; }
    
    public NewEntity(string name)
    {
        Name = Guard.Against.NullOrEmpty(name, nameof(name));
    }
    
    public void UpdateName(string name)
    {
        Name = Guard.Against.NullOrEmpty(name, nameof(name));
        AddDomainEvent(new EntityNameUpdatedEvent(Id, name));
    }
}
```

#### 2. Application Layer
Add commands, queries, and handlers:

```csharp
// Application/Commands/CreateEntityCommand.cs
public record CreateEntityCommand(string Name) : IRequest<Guid>;

// Application/Handlers/CreateEntityCommandHandler.cs
public class CreateEntityCommandHandler : IRequestHandler<CreateEntityCommand, Guid>
{
    // Implementation
}
```

#### 3. Infrastructure Layer
Implement repositories and services:

```csharp
// Infrastructure/Repositories/EntityRepository.cs
public class EntityRepository : Repository<NewEntity>, IEntityRepository
{
    // Implementation
}
```

#### 4. Presentation Layer
Add controllers and DTOs:

```csharp
// API/Controllers/EntitiesController.cs
[ApiController]
[Route("api/[controller]")]
public class EntitiesController : ControllerBase
{
    // Implementation
}
```

## 🏷️ Versioning

We follow [Semantic Versioning](https://semver.org/):

- **MAJOR**: Breaking changes
- **MINOR**: New features (backwards compatible)
- **PATCH**: Bug fixes (backwards compatible)

### Breaking Changes

When introducing breaking changes:

1. **Document clearly** in PR description
2. **Update migration guide**
3. **Consider deprecation** before removal
4. **Provide examples** of migration path

## 📋 Release Process

1. **Update version** in project files
2. **Update CHANGELOG.md**
3. **Create release notes**
4. **Tag the release**
5. **Publish NuGet package**

## 🎯 Code Review Checklist

### For Reviewers

- [ ] Code follows architecture principles
- [ ] Tests are comprehensive
- [ ] Performance considerations addressed
- [ ] Security implications considered
- [ ] Documentation is accurate
- [ ] Breaking changes are justified

### For Contributors

- [ ] Self-reviewed the code
- [ ] Tested on multiple scenarios
- [ ] Considered edge cases
- [ ] Updated relevant documentation
- [ ] Followed coding standards

## ❓ Questions?

- **General questions**: Open a discussion
- **Bug reports**: Create an issue
- **Feature requests**: Open a discussion first
- **Security issues**: Email directly (never create public issues)

## 📞 Contact

- **Maintainer**: [Your Name]
- **Email**: [your-email@example.com]
- **Discord**: [Discord Server Link]

Thank you for contributing to Gryd Authentication Framework! 🙏
