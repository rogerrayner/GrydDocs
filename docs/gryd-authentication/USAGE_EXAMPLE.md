# Exemplo de Uso - Gryd Authentication Framework

Este arquivo demonstra como integrar e usar o Gryd Authentication Framework em uma aplicação .NET.

## 1. Configuração Inicial

### Program.cs

```csharp
using GrydAuthenticationFramework.Application;
using GrydAuthenticationFramework.Infrastructure;
using GrydAuthenticationFramework.Infrastructure.Auth0;

var builder = WebApplication.CreateBuilder(args);

// Adicionar serviços do framework
builder.Services.AddApplicationServices();
builder.Services.AddInfrastructureServices(builder.Configuration);
builder.Services.AddAuth0Services(builder.Configuration);

// Configurar CORS
builder.Services.AddCors(options =>
{
    options.AddPolicy("AllowFrontend", policy =>
    {
        policy.WithOrigins("http://localhost:3000", "https://meuapp.com")
              .AllowAnyMethod()
              .AllowAnyHeader()
              .AllowCredentials();
    });
});

var app = builder.Build();

app.UseCors("AllowFrontend");
app.UseAuthentication();
app.UseAuthorization();

app.MapControllers();
app.Run();
```

### appsettings.json

```json
{
  "ConnectionStrings": {
    "DefaultConnection": "Server=localhost;Database=MeuApp;Integrated Security=true;"
  },
  "JwtSettings": {
    "SecretKey": "MinhaChaveSecretaSuperSeguraComPeloMenos32Caracteres",
    "Issuer": "MeuApp",
    "Audience": "MeuApp",
    "ExpirationMinutes": 60
  },
  "Auth0": {
    "Domain": "meudominio.auth0.com",
    "ClientId": "seu_client_id_aqui",
    "ClientSecret": "seu_client_secret_aqui",
    "Audience": "https://api.meuapp.com",
    "ManagementApiAudience": "https://meudominio.auth0.com/api/v2/"
  }
}
```

## 2. Controller Personalizado

```csharp
using Microsoft.AspNetCore.Mvc;
using Microsoft.AspNetCore.Authorization;
using MediatR;
using GrydAuthenticationFramework.API.Attributes;

[ApiController]
[Route("api/[controller]")]
[Authorize]
public class ProdutosController : ControllerBase
{
    private readonly IMediator _mediator;

    public ProdutosController(IMediator mediator)
    {
        _mediator = mediator;
    }

    [HttpGet]
    [RequirePermission("produtos.read")]
    public async Task<ActionResult> GetProdutos()
    {
        // Apenas usuários com permissão "produtos.read" podem acessar
        return Ok("Lista de produtos");
    }

    [HttpPost]
    [RequirePermission("produtos.create")]
    public async Task<ActionResult> CreateProduto([FromBody] CreateProdutoCommand command)
    {
        // Apenas usuários com permissão "produtos.create" podem acessar
        return Ok("Produto criado");
    }

    [HttpDelete("{id}")]
    [RequirePermission("produtos.delete")]
    public async Task<ActionResult> DeleteProduto(int id)
    {
        // Apenas usuários com permissão "produtos.delete" podem acessar
        return Ok("Produto excluído");
    }

    [HttpGet("admin-only")]
    [Authorize(Roles = "Admin")]
    public ActionResult AdminOnly()
    {
        // Apenas usuários com role "Admin" podem acessar
        return Ok("Área restrita para administradores");
    }
}
```

## 3. Middleware Personalizado para Auditoria

```csharp
public class AuditoriaMiddleware
{
    private readonly RequestDelegate _next;
    private readonly ICurrentUserService _currentUserService;

    public AuditoriaMiddleware(RequestDelegate next, ICurrentUserService currentUserService)
    {
        _next = next;
        _currentUserService = currentUserService;
    }

    public async Task InvokeAsync(HttpContext context)
    {
        if (_currentUserService.IsAuthenticated)
        {
            // Log da ação do usuário
            Console.WriteLine($"Usuário {_currentUserService.Email} acessou {context.Request.Path}");
        }

        await _next(context);
    }
}

// Registrar no Program.cs
app.UseMiddleware<AuditoriaMiddleware>();
```

## 4. Uso no Frontend (React/JavaScript)

### Login

```javascript
const login = async (email, password) => {
    try {
        const response = await fetch('/api/auth/login', {
            method: 'POST',
            headers: {
                'Content-Type': 'application/json'
            },
            body: JSON.stringify({ email, password })
        });

        const result = await response.json();
        
        if (result.isSuccess) {
            // Salvar token no localStorage
            localStorage.setItem('token', result.data.token);
            localStorage.setItem('permissions', JSON.stringify(result.data.permissions));
            
            return result.data;
        } else {
            throw new Error(result.errorMessage);
        }
    } catch (error) {
        console.error('Erro no login:', error);
        throw error;
    }
};
```

### Interceptor para Requisições

```javascript
// Axios interceptor para adicionar token automaticamente
axios.interceptors.request.use(
    (config) => {
        const token = localStorage.getItem('token');
        if (token) {
            config.headers.Authorization = `Bearer ${token}`;
        }
        return config;
    },
    (error) => {
        return Promise.reject(error);
    }
);

// Interceptor para lidar com token expirado
axios.interceptors.response.use(
    (response) => response,
    async (error) => {
        if (error.response?.status === 401) {
            localStorage.removeItem('token');
            localStorage.removeItem('permissions');
            window.location.href = '/login';
        }
        return Promise.reject(error);
    }
);
```

### Componente de Proteção

```jsx
import React from 'react';

const ProtectedComponent = ({ permission, children }) => {
    const permissions = JSON.parse(localStorage.getItem('permissions') || '[]');
    
    if (!permissions.includes(permission)) {
        return <div>Você não tem permissão para ver este conteúdo.</div>;
    }
    
    return children;
};

// Uso
<ProtectedComponent permission="produtos.create">
    <button onClick={createProduto}>Criar Produto</button>
</ProtectedComponent>
```

## 5. Seeds Personalizados

```csharp
// Adicionar no DbContext
protected override void OnModelCreating(ModelBuilder modelBuilder)
{
    base.OnModelCreating(modelBuilder);
    
    // Permissões personalizadas da aplicação
    var produtoPermissions = new[]
    {
        new Permission("produtos.read", "Visualizar produtos", "Produtos", false),
        new Permission("produtos.create", "Criar produtos", "Produtos", false),
        new Permission("produtos.update", "Editar produtos", "Produtos", false),
        new Permission("produtos.delete", "Excluir produtos", "Produtos", false),
        new Permission("vendas.read", "Visualizar vendas", "Vendas", false),
        new Permission("vendas.create", "Criar vendas", "Vendas", false),
        new Permission("relatorios.read", "Visualizar relatórios", "Relatórios", false)
    };

    modelBuilder.Entity<Permission>().HasData(produtoPermissions);
    
    // Role personalizada
    var vendedorRole = new Role("Vendedor", "Usuário que pode gerenciar vendas", false);
    modelBuilder.Entity<Role>().HasData(vendedorRole);
    
    // Atribuir permissões ao role Vendedor
    var vendedorPermissions = new[]
    {
        new RolePermission(vendedorRole.Id, produtoPermissions[0].Id), // produtos.read
        new RolePermission(vendedorRole.Id, produtoPermissions[4].Id), // vendas.read
        new RolePermission(vendedorRole.Id, produtoPermissions[5].Id)  // vendas.create
    };
    
    modelBuilder.Entity<RolePermission>().HasData(vendedorPermissions);
}
```

## 6. Validação Personalizada

```csharp
public class CreateProdutoCommandValidator : AbstractValidator<CreateProdutoCommand>
{
    public CreateProdutoCommandValidator()
    {
        RuleFor(x => x.Nome)
            .NotEmpty().WithMessage("Nome é obrigatório")
            .MaximumLength(100).WithMessage("Nome não pode exceder 100 caracteres");

        RuleFor(x => x.Preco)
            .GreaterThan(0).WithMessage("Preço deve ser maior que zero");
    }
}
```

## 7. Event Handlers Personalizados

```csharp
public class UserCreatedEventHandler : INotificationHandler<UserCreatedEvent>
{
    private readonly IEmailService _emailService;

    public UserCreatedEventHandler(IEmailService emailService)
    {
        _emailService = emailService;
    }

    public async Task Handle(UserCreatedEvent notification, CancellationToken cancellationToken)
    {
        // Enviar email de boas-vindas
        await _emailService.SendWelcomeEmailAsync(
            notification.Email, 
            "Bem-vindo ao nosso sistema!", 
            cancellationToken);
    }
}
```

## 8. Testando a API

### Postman/Insomnia

```bash
# 1. Login
POST /api/auth/login
Content-Type: application/json

{
    "email": "admin@exemplo.com",
    "password": "senha123"
}

# 2. Usar token retornado nas próximas requisições
GET /api/users
Authorization: Bearer SEU_TOKEN_AQUI

# 3. Criar usuário
POST /api/users
Authorization: Bearer SEU_TOKEN_AQUI
Content-Type: application/json

{
    "email": "novo@exemplo.com",
    "firstName": "João",
    "lastName": "Silva",
    "roleIds": ["GUID_DO_ROLE_USER"]
}
```

## 9. Docker (Opcional)

```dockerfile
# Dockerfile
FROM mcr.microsoft.com/dotnet/aspnet:9.0 AS base
WORKDIR /app
EXPOSE 80

FROM mcr.microsoft.com/dotnet/sdk:9.0 AS build
WORKDIR /src
COPY ["MeuApp.csproj", "."]
RUN dotnet restore "MeuApp.csproj"
COPY . .
RUN dotnet build "MeuApp.csproj" -c Release -o /app/build

FROM build AS publish
RUN dotnet publish "MeuApp.csproj" -c Release -o /app/publish

FROM base AS final
WORKDIR /app
COPY --from=publish /app/publish .
ENTRYPOINT ["dotnet", "MeuApp.dll"]
```

```yaml
# docker-compose.yml
version: '3.8'
services:
  app:
    build: .
    ports:
      - "5000:80"
    environment:
      - ConnectionStrings__DefaultConnection=Server=sqlserver;Database=MeuApp;User Id=sa;Password=YourPassword123!
    depends_on:
      - sqlserver

  sqlserver:
    image: mcr.microsoft.com/mssql/server:2022-latest
    environment:
      SA_PASSWORD: "YourPassword123!"
      ACCEPT_EULA: "Y"
    ports:
      - "1433:1433"
```

Este exemplo mostra como integrar completamente o framework em uma aplicação real, desde a configuração até o uso no frontend e testes.
