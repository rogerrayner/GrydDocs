# Gryd Authentication Framework

Um framework robusto e flexível de autenticação e autorização para aplicações .NET, seguindo os princípios do Clean Architecture e integração com Auth0.

## 🚀 Características

- **Clean Architecture**: Estrutura bem definida em camadas
- **Auth0 Integration**: Integração completa com Auth0 para autenticação
- **JWT Tokens**: Geração e validação de tokens JWT
- **Role-Based Access Control**: Sistema de roles e permissões
- **Social Login**: Suporte para login social (Google, Facebook, GitHub)
- **Extensível**: Fácil de estender com novas funcionalidades
- **NuGet Ready**: Pronto para ser publicado como pacotes NuGet

## 📦 Pacotes

O framework é composto por múltiplos pacotes NuGet:

- **GrydAuthenticationFramework.Domain**: Camada de domínio com entidades e lógica de negócio
- **GrydAuthenticationFramework.Application**: Camada de aplicação com casos de uso
- **GrydAuthenticationFramework.Infrastructure**: Implementações de infraestrutura
- **GrydAuthenticationFramework.Infrastructure.Auth0**: Integração específica com Auth0
- **GrydAuthenticationFramework.API**: Camada de apresentação com controllers REST

## 🏗️ Arquitetura

```
├── src/
│   ├── Core/
│   │   ├── Domain/           # Entidades, eventos, interfaces de repositório
│   │   └── Application/      # Casos de uso, DTOs, validações
│   ├── Infrastructure/
│   │   ├── Infrastructure/   # Repositórios, serviços, DbContext
│   │   └── Infrastructure.Auth0/ # Integração Auth0
│   └── Presentation/
│       └── API/             # Controllers REST, middleware
└── tests/                   # Testes unitários e de integração
```

## ⚡ Início Rápido

### 1. Instalação

```bash
# Instalar os pacotes necessários
Install-Package GrydAuthenticationFramework.Domain
Install-Package GrydAuthenticationFramework.Application
Install-Package GrydAuthenticationFramework.Infrastructure
Install-Package GrydAuthenticationFramework.Infrastructure.Auth0
```

### 2. Configuração

```csharp
// Program.cs
builder.Services.AddApplicationServices();
builder.Services.AddInfrastructureServices(builder.Configuration);
builder.Services.AddAuth0Services(builder.Configuration);
```

### 3. Configuração do appsettings.json

```json
{
  "ConnectionStrings": {
    "DefaultConnection": "sua-connection-string"
  },
  "JwtSettings": {
    "SecretKey": "sua-chave-secreta-de-pelo-menos-32-caracteres",
    "Issuer": "sua-aplicacao",
    "Audience": "sua-aplicacao",
    "ExpirationMinutes": 60
  },
  "Auth0": {
    "Domain": "seu-dominio.auth0.com",
    "ClientId": "seu-client-id",
    "ClientSecret": "seu-client-secret",
    "Audience": "sua-api-audience",
    "ManagementApiAudience": "https://seu-dominio.auth0.com/api/v2/"
  }
}
```

## 🔐 Uso

### Autenticação

```csharp
// Login com email/senha
[HttpPost("login")]
public async Task<ActionResult> Login([FromBody] LoginCommand command)
{
    var result = await _mediator.Send(command);
    return Ok(result);
}

// Login social
[HttpPost("social-login")]
public async Task<ActionResult> SocialLogin([FromBody] SocialLoginCommand command)
{
    var result = await _mediator.Send(command);
    return Ok(result);
}
```

### Autorização

```csharp
// Usando atributo de permissão
[HttpGet]
[RequirePermission("users.read")]
public async Task<ActionResult> GetUsers()
{
    // Apenas usuários com permissão "users.read" podem acessar
}

// Verificação manual de permissão
if (User.HasClaim("permission", "users.create"))
{
    // Usuário tem permissão para criar usuários
}
```

### Gerenciamento de Usuários

```csharp
// Criar usuário
var createCommand = new CreateUserCommand
{
    Email = "user@example.com",
    FirstName = "João",
    LastName = "Silva",
    RoleIds = new List<Guid> { adminRoleId }
};

var result = await _mediator.Send(createCommand);
```

## 🛡️ Segurança

### Roles Padrão

- **Admin**: Acesso total ao sistema
- **User**: Usuário padrão com permissões básicas

### Permissões do Sistema

- `users.read`, `users.create`, `users.update`, `users.delete`
- `roles.read`, `roles.create`, `roles.update`, `roles.delete`
- `permissions.read`, `permissions.create`, `permissions.update`, `permissions.delete`
- `admin.full`: Acesso administrativo completo

## 🔧 Configuração Avançada

### Middleware Personalizado

```csharp
app.UseMiddleware<ExceptionHandlingMiddleware>();
app.UseAuthentication();
app.UseAuthorization();
```

### Injeção de Dependência

O framework utiliza o container de DI nativo do .NET Core. Todos os serviços são registrados automaticamente através dos métodos de extensão.

## 📊 Banco de Dados

O framework utiliza Entity Framework Core com SQL Server por padrão. As migrações são aplicadas automaticamente.

### Entidades Principais

- **User**: Usuários do sistema
- **Role**: Grupos de usuários
- **Permission**: Permissões específicas
- **UserRole**: Relacionamento N:N entre usuários e roles
- **UserPermission**: Relacionamento N:N entre usuários e permissões
- **RolePermission**: Relacionamento N:N entre roles e permissões

## 🧪 Testes

Execute os testes usando:

```bash
dotnet test
```

## 📝 Exemplos de API

### Endpoints Principais

```
POST /api/auth/login
POST /api/auth/social-login
POST /api/auth/refresh
POST /api/auth/logout
GET  /api/auth/me
GET  /api/auth/validate

GET    /api/users
POST   /api/users
GET    /api/users/{id}
PUT    /api/users/{id}
DELETE /api/users/{id}
POST   /api/users/{userId}/roles/{roleId}
DELETE /api/users/{userId}/roles/{roleId}
```

## 🤝 Contribuindo

1. Fork o projeto
2. Crie uma branch para sua feature (`git checkout -b feature/MinhaFeature`)
3. Commit suas mudanças (`git commit -m 'Add MinhaFeature'`)
4. Push para a branch (`git push origin feature/MinhaFeature`)
5. Abra um Pull Request

## 📄 Licença

Este projeto está licenciado sob a Licença MIT - veja o arquivo [LICENSE](LICENSE) para detalhes.

## 🆘 Suporte

Para suporte, envie um email para support@gryd.com ou abra uma issue no GitHub.

---

⭐ Se este projeto foi útil para você, considere dar uma estrela!
