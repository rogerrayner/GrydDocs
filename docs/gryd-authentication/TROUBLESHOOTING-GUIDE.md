# Guia de Solução de Problemas - Gryd Authentication Framework

## 🚨 Problemas Comuns e Soluções

### 1. Problemas de Autenticação

#### ❌ "401 Unauthorized" mesmo com token válido

**Sintomas:**
- Token JWT válido mas API retorna 401
- Headers de autorização estão corretos

**Possíveis Causas e Soluções:**

```csharp
// ✅ Verificar ordem no Program.cs
app.UseAuthentication(); // DEVE vir antes
app.UseAuthorization();  // DEVE vir depois
app.MapControllers();
```

```csharp
// ✅ Verificar configuração do JWT
builder.Services.AddAuthentication(JwtBearerDefaults.AuthenticationScheme)
    .AddJwtBearer(options =>
    {
        options.TokenValidationParameters = new TokenValidationParameters
        {
            ValidateIssuer = true,
            ValidateAudience = true,
            ValidateLifetime = true,
            ValidateIssuerSigningKey = true,
            ValidIssuer = jwtSettings.Issuer,
            ValidAudience = jwtSettings.Audience,
            IssuerSigningKey = new SymmetricSecurityKey(Encoding.UTF8.GetBytes(jwtSettings.SecretKey)),
            ClockSkew = TimeSpan.Zero // Importante para tokens expirados
        };
    });
```

#### ❌ "Invalid signature" no token JWT

**Solução:**
```json
// Verificar se a chave secreta é a mesma na geração e validação
{
  "JwtSettings": {
    "SecretKey": "A-MESMA-CHAVE-EM-TODOS-OS-AMBIENTES"
  }
}
```

#### ❌ Token expira muito rápido

**Solução:**
```json
{
  "JwtSettings": {
    "ExpirationMinutes": 60, // Ajustar conforme necessário
    "ClockSkew": 5 // Tolerância para diferenças de relógio
  }
}
```

### 2. Problemas de Autorização

#### ❌ "403 Forbidden" para usuário autenticado

**Debugging:**
```csharp
// No seu controller, adicione logs para debug
[HttpGet]
[RequirePermission("produtos.read")]
public async Task<ActionResult> GetProdutos()
{
    var userClaims = User.Claims.Select(c => $"{c.Type}: {c.Value}");
    _logger.LogInformation("User claims: {Claims}", string.Join(", ", userClaims));
    
    return Ok();
}
```

**Verificar no banco:**
```sql
-- Verificar permissões do usuário
SELECT u.Email, r.Name as Role, p.Name as Permission
FROM Users u
JOIN UserRoles ur ON u.Id = ur.UserId
JOIN Roles r ON ur.RoleId = r.Id
JOIN RolePermissions rp ON r.Id = rp.RoleId
JOIN Permissions p ON rp.PermissionId = p.Id
WHERE u.Email = 'usuario@exemplo.com';
```

#### ❌ Permissões não aparecem no token

**Solução:**
```csharp
// Verificar se o JwtTokenService está incluindo permissões
private string GenerateAccessToken(User user)
{
    var claims = new List<Claim>
    {
        new(ClaimTypes.NameIdentifier, user.Id.ToString()),
        new(ClaimTypes.Email, user.Email),
        new(ClaimTypes.Name, user.Name)
    };

    // ✅ IMPORTANTE: Incluir permissões
    foreach (var permission in user.GetAllPermissions())
    {
        claims.Add(new Claim("permission", permission));
    }

    // ... resto do código
}
```

### 3. Problemas com Auth0

#### ❌ "Invalid audience" ao usar Auth0

**Solução:**
```json
{
  "Auth0": {
    "Domain": "meu-dominio.auth0.com",
    "Audience": "https://minha-api.com", // DEVE ser o mesmo da API no Auth0
    "ClientId": "client-id-correto"
  }
}
```

#### ❌ Social login não funciona

**Verificar configuração:**
```csharp
// No Auth0Service, verificar se está passando a connection correta
var request = new SignupUserRequest
{
    ClientId = _auth0Settings.ClientId,
    Connection = _auth0Settings.ConnectionName, // ✅ Verificar nome da connection
    Email = email,
    Password = password
};
```

**No Auth0 Dashboard:**
1. Applications > Sua App > Connections
2. Verificar se as Social Connections estão habilitadas
3. Testar callback URLs

### 4. Problemas de Banco de Dados

#### ❌ "Table doesn't exist" após migration

**Solução:**
```bash
# Verificar migrations
dotnet ef migrations list

# Aplicar migrations
dotnet ef database update

# Se necessário, resetar banco (CUIDADO EM PRODUÇÃO!)
dotnet ef database drop
dotnet ef database update
```

#### ❌ Foreign key constraint na inserção de usuário

**Verificar seed data:**
```csharp
protected override void OnModelCreating(ModelBuilder modelBuilder)
{
    base.OnModelCreating(modelBuilder);

    // ✅ Certificar que IDs das permissões/roles batem
    var adminRole = new Role(Guid.Parse("11111111-1111-1111-1111-111111111111"), "Admin", "Administrator");
    var readPermission = new Permission(Guid.Parse("22222222-2222-2222-2222-222222222222"), "users.read", "Read users", "Users");
    
    modelBuilder.Entity<Role>().HasData(adminRole);
    modelBuilder.Entity<Permission>().HasData(readPermission);
    
    // ✅ Usar os mesmos IDs na relação
    modelBuilder.Entity<RolePermission>().HasData(
        new RolePermission(adminRole.Id, readPermission.Id)
    );
}
```

#### ❌ Connection string inválida

**Para SQL Server:**
```json
{
  "ConnectionStrings": {
    "DefaultConnection": "Server=localhost;Database=MinhaApp;Trusted_Connection=true;MultipleActiveResultSets=true;TrustServerCertificate=true"
  }
}
```

**Para PostgreSQL:**
```json
{
  "ConnectionStrings": {
    "DefaultConnection": "Host=localhost;Database=MinhaApp;Username=user;Password=pass;Trust Server Certificate=true"
  }
}
```

### 5. Problemas de CORS

#### ❌ "CORS policy" error no frontend

**Solução:**
```csharp
// Program.cs
builder.Services.AddCors(options =>
{
    options.AddPolicy("AllowFrontend", policy =>
    {
        policy.WithOrigins("http://localhost:3000", "https://meuapp.com")
              .AllowAnyMethod()
              .AllowAnyHeader()
              .AllowCredentials(); // ✅ Importante para cookies/auth
    });
});

var app = builder.Build();

// ✅ CORS deve vir ANTES de Authentication
app.UseCors("AllowFrontend");
app.UseAuthentication();
app.UseAuthorization();
```

### 6. Problemas de Performance

#### ❌ Queries N+1 na busca de usuários

**Problema:**
```csharp
// ❌ Isso causa N+1 queries
var users = await _context.Users.ToListAsync();
foreach (var user in users)
{
    var roles = user.UserRoles; // Query separada para cada usuário
}
```

**Solução:**
```csharp
// ✅ Usar Include para eager loading
var users = await _context.Users
    .Include(u => u.UserRoles)
        .ThenInclude(ur => ur.Role)
            .ThenInclude(r => r.RolePermissions)
                .ThenInclude(rp => rp.Permission)
    .ToListAsync();
```

#### ❌ Token muito grande

**Problema:** Token JWT com muitas permissões fica muito grande

**Solução:**
```csharp
// Incluir apenas permissões essenciais no token
private string GenerateAccessToken(User user)
{
    var claims = new List<Claim>
    {
        new(ClaimTypes.NameIdentifier, user.Id.ToString()),
        new(ClaimTypes.Email, user.Email),
        new(ClaimTypes.Name, user.Name)
    };

    // ✅ Incluir apenas roles, não todas as permissões
    foreach (var role in user.UserRoles.Select(ur => ur.Role.Name))
    {
        claims.Add(new Claim(ClaimTypes.Role, role));
    }

    // Buscar permissões via ICurrentUserService quando necessário
    return GenerateToken(claims);
}
```

### 7. Problemas de Logging

#### ❌ Logs não aparecem

**Configuração Serilog:**
```csharp
// Program.cs
Log.Logger = new LoggerConfiguration()
    .MinimumLevel.Debug()
    .MinimumLevel.Override("Microsoft", LogEventLevel.Information)
    .Enrich.FromLogContext()
    .WriteTo.Console()
    .WriteTo.File("logs/app-.txt", 
        rollingInterval: RollingInterval.Day,
        retainedFileCountLimit: 7)
    .CreateLogger();

builder.Host.UseSerilog();
```

**Logging estruturado:**
```csharp
// ✅ Usar logging estruturado
_logger.LogInformation("User {UserId} logged in with {Email}", userId, email);

// ❌ Evitar concatenação
_logger.LogInformation($"User {userId} logged in with {email}");
```

### 8. Problemas de Deployment

#### ❌ Connection string não funciona em produção

**Usar variáveis de ambiente:**
```bash
# No servidor
export ConnectionStrings__DefaultConnection="Server=prod-server;Database=ProdDB;..."
export JwtSettings__SecretKey="chave-super-secreta-de-producao"
```

**Ou usar user secrets em desenvolvimento:**
```bash
dotnet user-secrets init
dotnet user-secrets set "ConnectionStrings:DefaultConnection" "sua-connection-string"
dotnet user-secrets set "JwtSettings:SecretKey" "sua-chave-secreta"
```

#### ❌ HTTPS não funciona

**Configuração para produção:**
```csharp
// Program.cs
if (!app.Environment.IsDevelopment())
{
    app.UseHttpsRedirection();
    app.UseHsts();
}
```

## 🔍 Ferramentas de Debug

### JWT Decoder
Use https://jwt.io para decodificar e verificar seus tokens JWT.

### SQL Profiler
Para identificar queries problemáticas:
```csharp
// Habilitar logging de queries em desenvolvimento
protected override void OnConfiguring(DbContextOptionsBuilder optionsBuilder)
{
    if (Environment.GetEnvironmentVariable("ASPNETCORE_ENVIRONMENT") == "Development")
    {
        optionsBuilder.LogTo(Console.WriteLine, LogLevel.Information);
    }
}
```

### Health Checks
```csharp
// Program.cs
builder.Services.AddHealthChecks()
    .AddDbContextCheck<AuthenticationDbContext>();

app.MapHealthChecks("/health");
```

## 📞 Quando Buscar Ajuda

Se após seguir este guia você ainda tiver problemas:

1. **Verifique os logs** - sempre ative o logging detalhado primeiro
2. **Reproduza em ambiente limpo** - teste com a configuração mínima
3. **Documente o problema** - inclua configurações, logs e passos para reproduzir
4. **Verifique versões** - certifique-se de estar usando .NET 9 e versões compatíveis

---

**Dica:** Mantenha sempre backups do banco em produção antes de aplicar migrations!
