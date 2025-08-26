# Gryd Authentication Framework

Um framework de autenticação e autorização completo para aplicações .NET, seguindo os princípios do Clean Architecture e integrado com Auth0.

## 🚀 Características

- **Clean Architecture**: Estrutura bem organizada seguindo os princípios do Clean Architecture
- **Princípios SOLID**: Implementação respeitando todos os princípios SOLID
- **Integração Auth0**: Suporte completo para autenticação via Auth0
- **Login Social**: Suporte para login com Google, Facebook, GitHub, etc.
- **JWT Tokens**: Geração e validação de tokens JWT
- **Sistema de Permissões**: Sistema robusto de roles e permissões
- **Entity Framework**: Persistência de dados com EF Core
- **Soft Delete**: Exclusão lógica de dados
- **Auditoria**: Rastreamento automático de criação, atualização e exclusão
- **Logs Estruturados**: Logging com Serilog
- **Documentação API**: Swagger/OpenAPI integrado

## 📦 Pacotes NuGet

O framework é distribuído em pacotes separados para máxima flexibilidade:

- `GrydAuthenticationFramework.Domain` - Entidades de domínio e lógica de negócio
- `GrydAuthenticationFramework.Application` - Casos de uso e lógica de aplicação
- `GrydAuthenticationFramework.Infrastructure` - Implementações de infraestrutura
- `GrydAuthenticationFramework.Infrastructure.Auth0` - Integração específica com Auth0
- `GrydAuthenticationFramework.API` - Controladores e middleware para APIs

## 🛠️ Instalação

### Via Package Manager Console
```powershell
Install-Package GrydAuthenticationFramework.Domain
Install-Package GrydAuthenticationFramework.Application
Install-Package GrydAuthenticationFramework.Infrastructure
Install-Package GrydAuthenticationFramework.Infrastructure.Auth0
```

### Via .NET CLI
```bash
dotnet add package GrydAuthenticationFramework.Domain
dotnet add package GrydAuthenticationFramework.Application
dotnet add package GrydAuthenticationFramework.Infrastructure
dotnet add package GrydAuthenticationFramework.Infrastructure.Auth0
```

## ⚙️ Configuração

### 1. appsettings.json

```json
{
  "ConnectionStrings": {
    "DefaultConnection": "Server=(localdb)\\mssqllocaldb;Database=SuaAplicacao;Trusted_Connection=true"
  },
  "JwtSettings": {
    "SecretKey": "SuaChaveSecretaDeMinimo32Caracteres",
    "Issuer": "SuaAplicacao",
    "Audience": "SuaAplicacao",
    "ExpirationMinutes": 60
  },
  "Auth0": {
    "Domain": "seu-dominio.auth0.com",
    "ClientId": "seu-client-id",
    "ClientSecret": "seu-client-secret",
    "Audience": "sua-api-audience",
    "ManagementApiAudience": "https://seu-dominio.auth0.com/api/v2/",
    "ConnectionName": "Username-Password-Authentication"
  }
}
```

### 2. Program.cs

```csharp
using GrydAuthenticationFramework.Application;
using GrydAuthenticationFramework.Infrastructure;
using GrydAuthenticationFramework.Infrastructure.Auth0;

var builder = WebApplication.CreateBuilder(args);

// Adicionar serviços do framework
builder.Services.AddApplicationServices();
builder.Services.AddInfrastructureServices(builder.Configuration);
builder.Services.AddAuth0Services(builder.Configuration);

// Seus outros serviços...
builder.Services.AddControllers();

var app = builder.Build();

// Configurar pipeline
app.UseAuthentication();
app.UseAuthorization();

app.MapControllers();
app.Run();
```

## 🔐 Uso Básico

### Autenticação

```csharp
[ApiController]
[Route("api/[controller]")]
public class AuthController : ControllerBase
{
    private readonly IMediator _mediator;

    [HttpPost("login")]
    public async Task<ActionResult> Login([FromBody] LoginCommand command)
    {
        var result = await _mediator.Send(command);
        return result.IsSuccess ? Ok(result) : BadRequest(result);
    }
}
```

### Controle de Acesso

```csharp
[ApiController]
[Route("api/[controller]")]
[Authorize]
public class UsuariosController : ControllerBase
{
    [HttpGet]
    [RequirePermission("users.read")]
    public async Task<ActionResult> GetUsuarios()
    {
        // Apenas usuários com permissão 'users.read' podem acessar
    }
}
```

## 📄 Licença

Este projeto está licenciado sob a Licença MIT.
