# Gryd Authentication Framework

Um framework robusto de autenticação e autorização para aplicações .NET, construído seguindo os princípios da Clean Architecture e integrado com Auth0.

## 📋 Características

- **Clean Architecture**: Implementação rigorosa dos princípios de Clean Architecture
- **Princípios SOLID**: Código limpo e manutenível seguindo todos os princípios SOLID
- **Integração Auth0**: Suporte completo para autenticação via Auth0
- **JWT Tokens**: Geração e validação de tokens JWT para autorização
- **Login Social**: Suporte para login com Google, Facebook, GitHub, etc.
- **Sistema de Permissões**: Sistema granular de roles e permissões
- **Entity Framework Core**: Persistência de dados com EF Core
- **MediatR**: Implementação do padrão CQRS com MediatR
- **FluentValidation**: Validação robusta de dados de entrada
- **Logging**: Sistema de logging estruturado com Serilog
- **Testes**: Cobertura de testes unitários e de integração

## 🏗️ Arquitetura

O framework é organizado em camadas seguindo Clean Architecture:

```
src/
├── Core/
│   ├── Domain/                 # Entidades, Value Objects, Domain Events
│   └── Application/            # Use Cases, DTOs, Interfaces
├── Infrastructure/
│   ├── Infrastructure/         # Data Access, Services
│   └── Infrastructure.Auth0/   # Integração específica com Auth0
└── Presentation/
    └── API/                    # Controllers REST, Middleware
```

## 🚀 Instalação

### Via NuGet (Recomendado)

```bash
# Instalar o pacote principal
Install-Package GrydAuthenticationFramework.Infrastructure

# Instalar integração com Auth0
Install-Package GrydAuthenticationFramework.Infrastructure.Auth0

# Instalar API (se necessário)
Install-Package GrydAuthenticationFramework.API
```

### Via Source Code

1. Clone o repositório:
```bash
git clone https://github.com/gryd/authentication-framework.git
```

2. Compile a solução:
```bash
dotnet build
```

## ⚙️ Configuração

### 1. Configuração do appsettings.json

```json
{
  "ConnectionStrings": {
    "DefaultConnection": "Server=.;Database=YourApp;Trusted_Connection=true;MultipleActiveResultSets=true"
  },
  "JwtSettings": {
    "SecretKey": "your-super-secret-key-at-least-32-characters-long",
    "Issuer": "YourApp",
    "Audience": "YourApp",
    "ExpirationMinutes": 60
  },
  "Auth0": {
    "Domain": "your-domain.auth0.com",
    "ClientId": "your-auth0-client-id",
    "ClientSecret": "your-auth0-client-secret",
    "Audience": "your-auth0-api-audience",
    "ManagementApiAudience": "https://your-domain.auth0.com/api/v2/",
    "ConnectionName": "Username-Password-Authentication",
    "SocialProviders": {
      "google": {
        "Name": "Google",
        "ConnectionName": "google-oauth2",
        "Enabled": true
      },
      "facebook": {
        "Name": "Facebook",
        "ConnectionName": "facebook",
        "Enabled": true
      }
    }
  }
}
```

### 2. Configuração no Program.cs

```csharp
using GrydAuthenticationFramework.Application;
using GrydAuthenticationFramework.Infrastructure;
using GrydAuthenticationFramework.Infrastructure.Auth0;

var builder = WebApplication.CreateBuilder(args);

// Adicionar serviços do framework
builder.Services.AddApplicationServices();
builder.Services.AddInfrastructureServices(builder.Configuration);
builder.Services.AddAuth0Services(builder.Configuration);

// Adicionar controllers
builder.Services.AddControllers();

var app = builder.Build();

// Configurar pipeline
app.UseAuthentication();
app.UseAuthorization();
app.MapControllers();

app.Run();
```

### 3. Executar Migrations

```bash
dotnet ef migrations add InitialCreate
dotnet ef database update
```

## 📖 Como Usar

### Autenticação

#### Login com Email/Senha

```csharp
POST /api/auth/login
{
  "email": "user@example.com",
  "password": "password123"
}
```

#### Login Social

```csharp
POST /api/auth/social-login
{
  "provider": "google",
  "accessToken": "social-provider-access-token"
}
```

### Autorização

#### Proteger Controllers

```csharp
[ApiController]
[Route("api/[controller]")]
[Authorize]
public class ProductsController : ControllerBase
{
    [HttpGet]
    [RequirePermission("products.read")]
    public async Task<IActionResult> GetProducts()
    {
        // Só usuários com permissão "products.read" podem acessar
        return Ok();
    }

    [HttpPost]
    [RequirePermission("products.create")]
    public async Task<IActionResult> CreateProduct()
    {
        // Só usuários com permissão "products.create" podem acessar
        return Ok();
    }
}
```

## 🔐 Sistema de Permissões

O framework usa um sistema hierárquico de permissões:

1. **Admin Role**: Tem acesso a todas as funcionalidades automaticamente
2. **Roles**: Agrupam permissões relacionadas
3. **Permissões Diretas**: Podem ser atribuídas diretamente aos usuários

### Permissões Padrão

```
users.read, users.create, users.update, users.delete
roles.read, roles.create, roles.update, roles.delete
permissions.read, permissions.create, permissions.update, permissions.delete
admin.full
```

## 📄 Licença

Este projeto está licenciado sob a Licença MIT.
