# Guia de Configuração - Gryd Authentication Framework

## 🔧 Configuração Inicial

### 1. Configuração do Auth0

#### Criando uma Application no Auth0:

1. Acesse o Dashboard do Auth0
2. Vá em **Applications** > **Create Application**
3. Escolha **Regular Web Application**
4. Anote o **Domain**, **Client ID** e **Client Secret**

#### Configurando APIs no Auth0:

1. Vá em **APIs** > **Create API**
2. Defina um **Identifier** (será o `Audience`)
3. Habilite **Enable RBAC** e **Add Permissions in the Access Token**

#### Configurando Social Logins:

1. Vá em **Authentication** > **Social**
2. Configure os provedores desejados (Google, Facebook, etc.)
3. Anote os **Connection Names**

### 2. Configuração do Banco de Dados

#### SQL Server:
```json
{
  "ConnectionStrings": {
    "DefaultConnection": "Server=localhost;Database=MinhaApp;Trusted_Connection=true;MultipleActiveResultSets=true"
  }
}
```

#### PostgreSQL:
```json
{
  "ConnectionStrings": {
    "DefaultConnection": "Host=localhost;Database=MinhaApp;Username=user;Password=password"
  }
}
```

#### SQLite (Desenvolvimento):
```json
{
  "ConnectionStrings": {
    "DefaultConnection": "Data Source=MinhaApp.db"
  }
}
```

### 3. Configuração Completa do appsettings.json

```json
{
  "Logging": {
    "LogLevel": {
      "Default": "Information",
      "Microsoft.AspNetCore": "Warning",
      "GrydAuthenticationFramework": "Debug"
    }
  },
  "AllowedHosts": "*",
  
  "ConnectionStrings": {
    "DefaultConnection": "Server=localhost;Database=MinhaApp;Trusted_Connection=true;MultipleActiveResultSets=true"
  },
  
  "JwtSettings": {
    "SecretKey": "SuaChaveSecretaMuitoSeguraDeMinimo32Caracteres",
    "Issuer": "MinhaAplicacao",
    "Audience": "MinhaAplicacao",
    "ExpirationMinutes": 60
  },
  
  "Auth0": {
    "Domain": "meu-dominio.auth0.com",
    "ClientId": "meu-client-id-do-auth0",
    "ClientSecret": "meu-client-secret-do-auth0",
    "Audience": "https://minha-api.com",
    "ManagementApiAudience": "https://meu-dominio.auth0.com/api/v2/",
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
      },
      "github": {
        "Name": "GitHub",
        "ConnectionName": "github",
        "Enabled": false
      }
    }
  },
  
  "Serilog": {
    "MinimumLevel": {
      "Default": "Information",
      "Override": {
        "Microsoft": "Warning",
        "System": "Warning"
      }
    },
    "WriteTo": [
      {
        "Name": "Console"
      },
      {
        "Name": "File",
        "Args": {
          "path": "logs/app-.txt",
          "rollingInterval": "Day",
          "rollOnFileSizeLimit": true,
          "fileSizeLimitBytes": 10485760
        }
      }
    ]
  }
}
```

## 🚀 Configuração do Program.cs

### Configuração Básica:

```csharp
using GrydAuthenticationFramework.Application;
using GrydAuthenticationFramework.Infrastructure;
using GrydAuthenticationFramework.Infrastructure.Auth0;
using Serilog;

var builder = WebApplication.CreateBuilder(args);

// Configurar Serilog
Log.Logger = new LoggerConfiguration()
    .ReadFrom.Configuration(builder.Configuration)
    .CreateLogger();

builder.Host.UseSerilog();

// Adicionar serviços básicos
builder.Services.AddControllers();
builder.Services.AddEndpointsApiExplorer();
builder.Services.AddSwaggerGen();

// Adicionar serviços do Framework
builder.Services.AddApplicationServices();
builder.Services.AddInfrastructureServices(builder.Configuration);
builder.Services.AddAuth0Services(builder.Configuration);

// CORS para frontend
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

// Pipeline de desenvolvimento
if (app.Environment.IsDevelopment())
{
    app.UseSwagger();
    app.UseSwaggerUI(c =>
    {
        c.SwaggerEndpoint("/swagger/v1/swagger.json", "Minha API v1");
        c.RoutePrefix = string.Empty; // Swagger na raiz
    });
}

app.UseHttpsRedirection();
app.UseCors("AllowFrontend");

// Ordem importante!
app.UseAuthentication();
app.UseAuthorization();

app.MapControllers();

app.Run();
```

### Configuração Avançada com Custom DbContext:

```csharp
using Microsoft.EntityFrameworkCore;
using GrydAuthenticationFramework.Infrastructure.Data;

// Custom DbContext
public class MinhaAppDbContext : AuthenticationDbContext
{
    public MinhaAppDbContext(DbContextOptions<MinhaAppDbContext> options) 
        : base(options)
    {
    }

    // Suas entidades personalizadas
    public DbSet<Produto> Produtos => Set<Produto>();
    public DbSet<Pedido> Pedidos => Set<Pedido>();

    protected override void OnModelCreating(ModelBuilder modelBuilder)
    {
        base.OnModelCreating(modelBuilder); // Importante!

        // Suas configurações personalizadas
        modelBuilder.Entity<Produto>(entity =>
        {
            entity.ToTable("Produtos");
            entity.Property(e => e.Nome).HasMaxLength(100);
        });
    }
}

// No Program.cs
builder.Services.AddDbContext<MinhaAppDbContext>(options =>
{
    options.UseSqlServer(builder.Configuration.GetConnectionString("DefaultConnection"));
});
```

## 🗃️ Migrations e Seed Data

### Criando a primeira Migration:

```bash
# Se usando o projeto diretamente
dotnet ef migrations add InitialCreate -p src/Infrastructure/GrydAuthenticationFramework.Infrastructure

# Se usando DbContext customizado
dotnet ef migrations add InitialCreate -c MinhaAppDbContext
dotnet ef database update
```

### Seed Data Personalizado:

```csharp
// No seu DbContext customizado
protected override void OnModelCreating(ModelBuilder modelBuilder)
{
    base.OnModelCreating(modelBuilder);

    // Permissões personalizadas
    var permissoesProdutos = new[]
    {
        new Permission("produtos.read", "Visualizar produtos", "Produtos"),
        new Permission("produtos.create", "Criar produtos", "Produtos"),
        new Permission("produtos.update", "Atualizar produtos", "Produtos"),
        new Permission("produtos.delete", "Deletar produtos", "Produtos")
    };

    modelBuilder.Entity<Permission>().HasData(permissoesProdutos);

    // Role personalizada
    var roleVendedor = new Role("Vendedor", "Vendedor da loja");
    modelBuilder.Entity<Role>().HasData(roleVendedor);

    // Atribuir permissões à role
    var rolePermissions = permissoesProdutos.Select(p => 
        new RolePermission(roleVendedor.Id, p.Id)).ToArray();
    
    modelBuilder.Entity<RolePermission>().HasData(rolePermissions);
}
```

## 🔐 Configuração de Segurança

### Configuração de JWT para Produção:

```json
{
  "JwtSettings": {
    "SecretKey": "Uma-Chave-Muito-Segura-De-Pelo-Menos-64-Caracteres-Para-Producao",
    "Issuer": "https://api.meuapp.com",
    "Audience": "https://meuapp.com",
    "ExpirationMinutes": 30,
    "ClockSkew": 5
  }
}
```

### Configuração HTTPS em Produção:

```csharp
// No Program.cs para produção
if (!app.Environment.IsDevelopment())
{
    app.UseHsts();
    app.UseHttpsRedirection();
}

// Headers de segurança
app.Use(async (context, next) =>
{
    context.Response.Headers.Add("X-Frame-Options", "DENY");
    context.Response.Headers.Add("X-Content-Type-Options", "nosniff");
    context.Response.Headers.Add("X-XSS-Protection", "1; mode=block");
    await next();
});
```

## 🎯 Configuração de Permissões Personalizadas

### Criando Controller com Permissões:

```csharp
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
        // Apenas usuários com permissão "produtos.read"
        return Ok();
    }

    [HttpPost]
    [RequirePermission("produtos.create")]
    public async Task<ActionResult> CreateProduto()
    {
        // Apenas usuários com permissão "produtos.create"
        return Ok();
    }

    [HttpGet("relatorio")]
    [RequirePermission("produtos.read")]
    [RequirePermission("relatorios.read")] // Múltiplas permissões
    public async Task<ActionResult> GetRelatorio()
    {
        return Ok();
    }
}
```

### Verificação Programática:

```csharp
public class ProdutoService
{
    private readonly ICurrentUserService _currentUser;

    public ProdutoService(ICurrentUserService currentUser)
    {
        _currentUser = currentUser;
    }

    public async Task<List<Produto>> GetProdutos()
    {
        var produtos = await GetAllProdutos();

        // Filtrar baseado em permissões
        if (!_currentUser.HasPermission("produtos.read.all"))
        {
            // Mostrar apenas produtos do usuário
            produtos = produtos.Where(p => p.CriadoPor == _currentUser.UserId).ToList();
        }

        return produtos;
    }
}
```

## 🌐 Configuração para Diferentes Ambientes

### appsettings.Development.json:

```json
{
  "ConnectionStrings": {
    "DefaultConnection": "Data Source=dev.db"
  },
  "JwtSettings": {
    "SecretKey": "chave-de-desenvolvimento-nao-segura"
  },
  "Auth0": {
    "Domain": "dev-domain.auth0.com"
  }
}
```

### appsettings.Production.json:

```json
{
  "ConnectionStrings": {
    "DefaultConnection": "#{ConnectionString}#" // Variável do pipeline
  },
  "JwtSettings": {
    "SecretKey": "#{JwtSecretKey}#"
  },
  "Auth0": {
    "Domain": "#{Auth0Domain}#",
    "ClientId": "#{Auth0ClientId}#",
    "ClientSecret": "#{Auth0ClientSecret}#"
  }
}
```

## 📱 Configuração para Frontend

### Exemplo de login no frontend:

```typescript
// React/TypeScript
interface LoginRequest {
  email: string;
  password: string;
}

interface AuthResponse {
  token: string;
  user: {
    id: string;
    email: string;
    permissions: string[];
  };
}

const login = async (credentials: LoginRequest): Promise<AuthResponse> => {
  const response = await fetch('/api/auth/login', {
    method: 'POST',
    headers: {
      'Content-Type': 'application/json',
    },
    body: JSON.stringify(credentials),
  });

  if (!response.ok) {
    throw new Error('Login failed');
  }

  const result = await response.json();
  
  // Salvar token
  localStorage.setItem('token', result.data.token);
  
  return result.data;
};
```

### Interceptor para incluir token:

```typescript
// Axios interceptor
axios.interceptors.request.use(
  (config) => {
    const token = localStorage.getItem('token');
    if (token) {
      config.headers.Authorization = `Bearer ${token}`;
    }
    return config;
  },
  (error) => Promise.reject(error)
);
```

---

Este guia deve cobrir a maioria dos cenários de configuração. Para casos específicos, consulte o código de exemplo em `examples/ExampleWebApi`.
