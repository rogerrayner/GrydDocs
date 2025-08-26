# Gryd Authentication Framework - Migração para v2.0

## 🚀 Visão Geral da v2.0

A versão 2.0 do Gryd Authentication Framework traz melhorias significativas em performance, segurança e funcionalidades, mantendo a compatibilidade com .NET 9.

### 🆕 Principais Novidades

- **Multi-tenancy** nativo
- **OAuth 2.0 / OpenID Connect** completo
- **Refresh tokens** automáticos
- **Rate limiting** integrado
- **Audit logging** avançado
- **Políticas de senha** configuráveis
- **Two-Factor Authentication (2FA)**
- **Session management** melhorado

## 🔄 Guia de Migração

### 1. Atualizando Pacotes

```xml
<!-- Antes (v1.x) -->
<PackageReference Include="GrydAuthenticationFramework" Version="1.0.0" />

<!-- Depois (v2.x) -->
<PackageReference Include="GrydAuthenticationFramework" Version="2.0.0" />
```

### 2. Mudanças na Configuração

#### appsettings.json

```json
{
  // ✅ NOVO: Configuração de multi-tenancy
  "MultiTenancy": {
    "Enabled": true,
    "DefaultTenant": "default",
    "TenantResolution": "Header" // Header, Subdomain, Route
  },
  
  // ✅ EXPANDIDO: Configurações JWT
  "JwtSettings": {
    "SecretKey": "sua-chave-secreta",
    "Issuer": "sua-aplicacao",
    "Audience": "sua-aplicacao",
    "ExpirationMinutes": 15, // ⚠️ MUDANÇA: Padrão reduzido de 60 para 15
    "RefreshTokenExpirationDays": 7, // ✅ NOVO
    "ClockSkew": 5
  },
  
  // ✅ NOVO: Políticas de senha
  "PasswordPolicy": {
    "MinLength": 8,
    "RequireUppercase": true,
    "RequireLowercase": true,
    "RequireDigit": true,
    "RequireSpecialChar": true,
    "MaxRepeatingChars": 3,
    "PreventCommonPasswords": true
  },
  
  // ✅ NOVO: Rate limiting
  "RateLimiting": {
    "LoginAttempts": {
      "Limit": 5,
      "Window": "00:15:00", // 15 minutos
      "RetryAfter": "00:15:00"
    },
    "ApiCalls": {
      "Limit": 1000,
      "Window": "01:00:00" // 1 hora
    }
  },
  
  // ✅ EXPANDIDO: Auth0 com 2FA
  "Auth0": {
    "Domain": "seu-dominio.auth0.com",
    "ClientId": "seu-client-id",
    "ClientSecret": "seu-client-secret",
    "Audience": "https://sua-api.com",
    "EnableMFA": true, // ✅ NOVO
    "MFAProvider": "auth0" // ✅ NOVO: auth0, sms, email
  }
}
```

### 3. Mudanças no Program.cs

```csharp
var builder = WebApplication.CreateBuilder(args);

// ✅ NOVO: Multi-tenancy (opcional)
builder.Services.AddMultiTenancy<Tenant>(builder.Configuration);

// ✅ ATUALIZADO: Serviços com novas configurações
builder.Services.AddApplicationServices();
builder.Services.AddInfrastructureServices(builder.Configuration);
builder.Services.AddAuth0Services(builder.Configuration);

// ✅ NOVO: Rate limiting
builder.Services.AddRateLimiting(builder.Configuration);

// ✅ NOVO: 2FA services
builder.Services.AddTwoFactorAuthentication(builder.Configuration);

var app = builder.Build();

// ✅ NOVO: Rate limiting middleware
app.UseRateLimiter();

// ✅ NOVO: Multi-tenancy middleware (se habilitado)
app.UseMultiTenancy();

// Ordem mantida
app.UseAuthentication();
app.UseAuthorization();

app.MapControllers();
app.Run();
```

### 4. Mudanças nas Entities

#### User Entity - Novos Campos

```csharp
// ✅ NOVOS campos adicionados automaticamente via migration
public class User : BaseEntity
{
    // Campos existentes mantidos...
    
    // ✅ NOVOS campos
    public string? TenantId { get; private set; } // Multi-tenancy
    public DateTime? LastPasswordChangeAt { get; private set; }
    public int FailedLoginAttempts { get; private set; }
    public DateTime? LockedUntil { get; private set; }
    public bool IsTwoFactorEnabled { get; private set; }
    public string? TwoFactorSecret { get; private set; }
    public string? PhoneNumber { get; private set; }
    public bool PhoneNumberConfirmed { get; private set; }
    public List<string> RecoveryCodes { get; private set; } = new();
    
    // ✅ NOVOS métodos
    public void EnableTwoFactor(string secret)
    {
        IsTwoFactorEnabled = true;
        TwoFactorSecret = secret;
        AddDomainEvent(new UserTwoFactorEnabledEvent(Id));
    }
    
    public void LockAccount(DateTime until)
    {
        LockedUntil = until;
        AddDomainEvent(new UserAccountLockedEvent(Id, until));
    }
}
```

### 5. Migration do Banco de Dados

```bash
# Criar nova migration para v2.0
dotnet ef migrations add UpgradeToV2 -p src/Infrastructure/GrydAuthenticationFramework.Infrastructure

# Aplicar migration
dotnet ef database update
```

**⚠️ IMPORTANTE**: Faça backup do banco antes da migration!

### 6. Mudanças na API

#### Novos Endpoints

```http
# ✅ NOVOS endpoints v2.0
POST /api/v2/auth/refresh-token
POST /api/v2/auth/enable-2fa
POST /api/v2/auth/verify-2fa
POST /api/v2/auth/disable-2fa
GET  /api/v2/auth/qr-code
POST /api/v2/auth/forgot-password
POST /api/v2/auth/reset-password
POST /api/v2/auth/change-password

# Multi-tenancy
GET  /api/v2/tenants
POST /api/v2/tenants
PUT  /api/v2/tenants/{id}
```

#### Mudanças em Endpoints Existentes

```http
# ⚠️ MUDANÇA: Login agora retorna refresh token
POST /api/auth/login
{
  "email": "user@example.com",
  "password": "password123",
  "twoFactorCode": "123456" // ✅ NOVO: opcional para 2FA
}

# Resposta atualizada
{
  "success": true,
  "data": {
    "accessToken": "jwt-token",
    "refreshToken": "refresh-token", // ✅ NOVO
    "expiresIn": 900, // ✅ NOVO: em segundos
    "user": {
      "id": "guid",
      "email": "user@example.com",
      "name": "User Name",
      "isTwoFactorEnabled": false, // ✅ NOVO
      "tenantId": "tenant-id" // ✅ NOVO: se multi-tenancy habilitado
    }
  }
}
```

### 7. Mudanças no Frontend

#### Refresh Token Automático

```typescript
// ✅ NOVO: Interceptor para refresh token automático
let isRefreshing = false;
let failedQueue: any[] = [];

axios.interceptors.response.use(
  (response) => response,
  async (error) => {
    const originalRequest = error.config;

    if (error.response?.status === 401 && !originalRequest._retry) {
      if (isRefreshing) {
        return new Promise((resolve, reject) => {
          failedQueue.push({ resolve, reject });
        }).then(token => {
          originalRequest.headers.Authorization = `Bearer ${token}`;
          return axios(originalRequest);
        });
      }

      originalRequest._retry = true;
      isRefreshing = true;

      try {
        const refreshToken = localStorage.getItem('refreshToken');
        const response = await axios.post('/api/v2/auth/refresh-token', {
          refreshToken
        });

        const { accessToken } = response.data.data;
        localStorage.setItem('token', accessToken);
        
        failedQueue.forEach(({ resolve }) => resolve(accessToken));
        failedQueue = [];

        originalRequest.headers.Authorization = `Bearer ${accessToken}`;
        return axios(originalRequest);
      } catch (refreshError) {
        localStorage.removeItem('token');
        localStorage.removeItem('refreshToken');
        window.location.href = '/login';
        return Promise.reject(refreshError);
      } finally {
        isRefreshing = false;
      }
    }

    return Promise.reject(error);
  }
);
```

#### Two-Factor Authentication

```typescript
// ✅ NOVO: Componente para 2FA
interface TwoFactorSetupResponse {
  qrCodeUrl: string;
  manualEntryKey: string;
  recoveryCodes: string[];
}

const enableTwoFactor = async (): Promise<TwoFactorSetupResponse> => {
  const response = await axios.post('/api/v2/auth/enable-2fa');
  return response.data.data;
};

const verifyTwoFactor = async (code: string): Promise<void> => {
  await axios.post('/api/v2/auth/verify-2fa', { code });
};
```

### 8. Multi-Tenancy (Opcional)

Se você quiser usar multi-tenancy:

```csharp
// ✅ NOVO: Tenant entity
public class Tenant : BaseEntity
{
    public string Name { get; private set; }
    public string Subdomain { get; private set; }
    public bool IsActive { get; private set; }
    public Dictionary<string, object> Settings { get; private set; }
}

// ✅ NOVO: Tenant service
public interface ITenantService
{
    Task<Tenant> GetCurrentTenantAsync();
    Task<Tenant> GetTenantByIdAsync(string tenantId);
}
```

## ⚠️ Breaking Changes

### 1. JWT Token Expiration
- **Antes**: 60 minutos
- **Depois**: 15 minutos (com refresh token)

### 2. User Entity
- Novos campos obrigatórios adicionados
- Migration automática aplicará valores padrão

### 3. API Responses
- Algumas respostas agora incluem campos adicionais
- Códigos de status podem ter mudado em cenários específicos

### 4. Dependency Injection
- Novos serviços registrados automaticamente
- Configurações adicionais podem ser necessárias

## 🔍 Validação da Migração

### Checklist de Validação

- [ ] ✅ Aplicação compila sem erros
- [ ] ✅ Migrations aplicadas com sucesso
- [ ] ✅ Login funciona normalmente
- [ ] ✅ Refresh token funciona
- [ ] ✅ Rate limiting não bloqueia uso normal
- [ ] ✅ Permissões existentes ainda funcionam
- [ ] ✅ Frontend continua funcionando
- [ ] ✅ Logs não mostram erros

### Scripts de Teste

```bash
# Testar login
curl -X POST "https://localhost:5001/api/auth/login" \
  -H "Content-Type: application/json" \
  -d '{
    "email": "admin@test.com",
    "password": "Admin123!"
  }'

# Testar refresh token
curl -X POST "https://localhost:5001/api/v2/auth/refresh-token" \
  -H "Content-Type: application/json" \
  -d '{
    "refreshToken": "seu-refresh-token"
  }'
```

## 🚨 Rollback

Se precisar fazer rollback:

```bash
# Voltar migration
dotnet ef migrations remove

# Restaurar versão anterior
dotnet restore --force -s https://api.nuget.org/v3/index.json
```

## 📞 Suporte

Se encontrar problemas durante a migração:

1. **Consulte o troubleshooting guide**: `TROUBLESHOOTING-GUIDE.md`
2. **Verifique os logs** da aplicação
3. **Teste em ambiente de desenvolvimento** primeiro
4. **Abra uma issue** no GitHub com detalhes completos

## 🎯 Próximos Passos

Após a migração, considere:

1. **Habilitar 2FA** para usuários administrativos
2. **Configurar rate limiting** apropriado para seu caso
3. **Implementar multi-tenancy** se necessário
4. **Atualizar documentação** interna
5. **Treinar equipe** nas novas funcionalidades

---

**Boa migração! 🚀**
