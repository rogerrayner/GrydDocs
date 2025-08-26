# Segurança Multiaplicação com GrydAuth + Auth0

## Alterações no GrydAuth

1. **Campo `AppIds` no modelo User**
   - Usuário pode estar vinculado a uma ou mais aplicações.
   - Todas consultas e criação de usuário devem considerar o(s) appId(s).

2. **Validação de appId nos endpoints protegidos**
   - Ao receber o JWT, a API deve validar se o usuário local está vinculado ao appId da requisição/token.
   - Middleware recomendado para centralizar essa validação.

3. **Sincronização de appId com Auth0**
   - O appId deve ser incluído no `user_metadata` do Auth0.
   - Use Connections separadas para cada aplicação, se necessário.

4. **Scopes e claims customizadas no JWT**
   - Configure o Auth0 para adicionar claims como appId, roles, permissions no token.
   - A API deve validar essas claims antes de liberar acesso.

5. **Usuário pode estar vinculado a múltiplos appIds**
   - O campo `AppIds` é uma coleção.
   - Permite cenários multiapp e maior flexibilidade.

## Como configurar o Auth0 para multiaplicação

### 1. Criar uma Connection separada para cada aplicação

1. **Acesse o Dashboard do Auth0**: https://manage.auth0.com/
2. **No menu lateral**, clique em "Authentication" > "Database"
3. **Clique em "+ Create Database"**
4. **Escolha um nome para a Connection** (ex: `app1-users`, `requisition-control-users`)
5. **Configure as opções**:
   - Enable sign ups (se permitir cadastro)
   - Username/Email login
   - Password policy
6. **Salve a Connection**
7. **Vincule a Connection à sua aplicação**:
   - Vá em "Applications" > [Sua aplicação] > "Connections"
   - Marque a Connection que você acabou de criar
   - Desmarque outras connections desnecessárias

### 2. Adicionar o campo `appId` no `user_metadata` ao criar o usuário

**Via dashboard do Auth0:**
```json
{
  "user_metadata": {
    "appId": "requisition-control"
  }
}
```

**Para múltiplos apps:**
```json
{
  "user_metadata": {
    "appIds": ["requisition-control", "financeiro"]
  }
}
```

**Via Management API durante onboarding:**
```javascript
// Exemplo de chamada para atualizar user_metadata
const updateUserMetadata = async (userId, appId) => {
  const response = await fetch(`https://YOUR_DOMAIN.auth0.com/api/v2/users/${userId}`, {
    method: 'PATCH',
    headers: {
      'Authorization': `Bearer ${managementApiToken}`,
      'Content-Type': 'application/json'
    },
    body: JSON.stringify({
      user_metadata: {
        appId: appId,
        // ou appIds: [appId] para múltiplas apps
      }
    })
  });
};
```

### 3. Configurar Actions para incluir claims customizadas no JWT

1. **Acesse "Actions" > "Flows" > "Login"**
2. **Clique em "+" para adicionar uma Action**
3. **Crie uma nova Action customizada**:

```javascript
exports.onExecutePostLogin = async (event, api) => {
  // Adiciona appId do user_metadata como claim
  const appId = event.user.user_metadata && event.user.user_metadata.appId;
  const appIds = event.user.user_metadata && event.user.user_metadata.appIds;
  
  if (appId) {
    api.idToken.setCustomClaim("appId", appId);
    api.accessToken.setCustomClaim("appId", appId);
  }
  
  if (appIds && Array.isArray(appIds)) {
    api.idToken.setCustomClaim("appIds", appIds);
    api.accessToken.setCustomClaim("appIds", appIds);
  }
  
  // Adicione roles/permissions se estiverem no Auth0
  const roles = event.authorization && event.authorization.roles;
  if (roles && roles.length > 0) {
    api.idToken.setCustomClaim("roles", roles);
    api.accessToken.setCustomClaim("roles", roles);
  }
};
```

4. **Deploy da Action** e adicione ao flow de Login

### 4. Garantir que cada aplicação use seu próprio ClientId e Connection

- **ClientId**: Cada aplicação deve ter seu próprio ClientId no Auth0
- **Connection**: Configure cada app para usar apenas sua Connection específica
- **Callback URLs**: Configure URLs de callback específicas para cada app
- **Allowed Origins**: Configure domínios permitidos para cada aplicação

### 5. Exemplo de configuração completa

**Aplicação 1 - RequisitionControl:**
- Connection: `requisition-control-users`
- ClientId: `abc123...`
- AppId: `requisition-control`

**Aplicação 2 - Financeiro:**
- Connection: `financeiro-users`  
- ClientId: `def456...`
- AppId: `financeiro`

**Usuário multiapp:**
```json
{
  "user_metadata": {
    "appIds": ["requisition-control", "financeiro"]
  }
}
```

## Clean Architecture & SOLID
- Todas alterações seguem rigorosamente Clean Architecture.
- Nenhuma dependência circular, separação clara de camadas.
- Princípios SOLID respeitados: responsabilidade única, injeção de dependência, abstração, etc.

## Referências
- [Documentação oficial Auth0](https://auth0.com/docs/)
- [Auth0 Actions Documentation](https://auth0.com/docs/customize/actions)
- [Auth0 Management API](https://auth0.com/docs/api/management/v2)
- [Documentação GrydAuth](./README.md)

## Exemplo de fluxo completo

1. **Setup inicial**: Crie Connections e configure Actions no Auth0
2. **Onboarding**: Vincule usuários ao appId correto durante cadastro/primeiro login
3. **Autenticação**: JWT contém claims com appId/appIds
4. **Autorização**: Middleware valida se usuário local está vinculado ao appId da requisição
5. **Operações**: Todas operações filtram dados pelo appId do usuário

---

**Checklist para segurança multiapp:**
- [x] Campo AppIds no User
- [x] Validação de appId nos endpoints  
- [x] Claims customizadas no JWT
- [x] Sincronização com Auth0
- [x] Clean Architecture & SOLID
- [x] Connections separadas por aplicação
- [x] Actions configuradas para claims
- [x] User metadata com appId
