# 🚀 CI/CD Pipeline - Guia Completo

## Visão Geral

Este projeto utiliza um sistema avançado de CI/CD com **branch protection** que garante que apenas código de alta qualidade seja mergeado nas branches principais.

## 🛡️ Branch Protection

### Como Funciona

1. **Checks Obrigatórios**: Todos os jobs marcados como obrigatórios devem passar antes do merge
2. **Quality Gates**: Sistema de validação em múltiplas camadas
3. **Automated Blocking**: PRs são automaticamente bloqueados até todos os checks passarem

### Jobs Obrigatórios

| Job | Descrição | Obrigatório |
|-----|-----------|-------------|
| Quality Gate | Validação inicial e cálculo de versão | ✅ |
| Unit Tests | Testes unitários em múltiplas versões .NET | ✅ |
| Security Scan | Análise de vulnerabilidades | ✅ |
| Integration Tests | Testes com múltiplos bancos de dados | ✅ |
| Build and Package | Criação dos pacotes NuGet | ✅ |
| CI Success | Status consolidado final | ✅ |

## 📋 Fluxo de Trabalho

### 1. Criação de Pull Request

```bash
# Criar nova feature branch
git checkout develop
git pull origin develop
git checkout -b feature/nova-funcionalidade

# Fazer alterações e commit
git add .
git commit -m "feat: adicionar nova funcionalidade"
git push origin feature/nova-funcionalidade

# Criar PR via GitHub UI
```

### 2. Validação Automática

Assim que o PR é criado, o sistema executa:

1. **Quality Gate** - Validação inicial
2. **Unit Tests** - Testes em .NET 8.0 e 9.0
3. **Security Scan** - Verificação de vulnerabilidades
4. **Integration Tests** - Testes com PostgreSQL e SQL Server
5. **Build and Package** - Criação dos pacotes

### 3. Branch Protection em Ação

- ❌ **Merge bloqueado** enquanto algum check falhar
- ⏳ **Status "Pending"** durante execução dos testes
- ✅ **Merge liberado** apenas quando todos passarem

## 🔄 Automatic Publishing

### Estratégia de Versionamento

| Branch/Tag | Versão | Publicar | Exemplo |
|------------|--------|----------|---------|
| `master` | Patch increment | ✅ | `1.2.4` |
| `develop` | Alpha pre-release | ❌ | `1.2.4-alpha.abc1234` |
| `v*` tags | Tag version | ✅ | `1.3.0` |
| Feature branches | Alpha pre-release | ❌ | `1.2.4-alpha.xyz5678` |

### Publicação Automática

```yaml
# Apenas para master e tags
if: needs.quality-gate.outputs.should-publish == 'true'
```

- **Master**: Publica versão estável
- **Tags**: Publica versão específica
- **Develop/Feature**: Apenas build, sem publicação

## 🧪 Testes Multi-Ambiente

### Unit Tests Matrix

```yaml
strategy:
  matrix:
    dotnet-version: ['8.0.x', '9.0.x']
```

### Integration Tests

- **PostgreSQL 15**: Testes de persistência
- **SQL Server 2022**: Testes de compatibilidade
- **Multi-database**: Validação cross-platform

## 🔒 Security Scanning

### Ferramentas Utilizadas

1. **dotnet audit**: Verificação de vulnerabilidades conhecidas
2. **Snyk**: Análise avançada de segurança
3. **Dependency Check**: Validação de dependências

### Políticas de Segurança

- ❌ **Build falha** se vulnerabilidades HIGH forem encontradas
- ⚠️ **Warning** para vulnerabilidades MEDIUM/LOW
- 📊 **Reports** enviados para ferramentas de monitoramento

## 📦 Package Management

### Pacotes Gerados

1. `GrydCrudFramework.Domain`
2. `GrydCrudFramework.Application`
3. `GrydCrudFramework.Infrastructure`
4. `GrydCrudFramework.API`

### Versionamento Sincronizado

Todos os pacotes recebem a mesma versão calculada automaticamente:

```bash
# Exemplo para master
VERSION="1.2.4"

# Exemplo para develop
VERSION="1.2.4-alpha.abc1234"
```

## 🎯 Como Configurar Branch Protection no GitHub

### 1. Acessar Repository Settings

1. Vá para **Settings** → **Branches**
2. Clique em **Add rule** ou edite regra existente

### 2. Configurar Protection Rule

```yaml
Branch name pattern: master
☑️ Restrict pushes that create files larger than 100 MB
☑️ Require a pull request before merging
  ☑️ Require approvals: 1
  ☑️ Dismiss stale PR approvals when new commits are pushed
  ☑️ Require review from code owners
☑️ Require status checks to pass before merging
  ☑️ Require branches to be up to date before merging
  Required status checks:
    ☑️ Quality Gate
    ☑️ Unit Tests
    ☑️ Security Scan  
    ☑️ Integration Tests
    ☑️ Build and Package
    ☑️ CI Success
☑️ Require conversation resolution before merging
☑️ Include administrators
```

### 3. Configurar para Develop

Repetir o processo para a branch `develop` com as mesmas configurações.

## 🔧 Secrets Necessários

Configure os seguintes secrets no GitHub:

| Secret | Descrição | Necessário Para |
|--------|-----------|-----------------|
| `NUGET_API_KEY` | Chave da API do NuGet | Publicação de pacotes |
| `CODECOV_TOKEN` | Token do Codecov | Upload de coverage |
| `SNYK_TOKEN` | Token do Snyk | Security scanning |

## 🚨 Troubleshooting

### PR Bloqueado

**Problema**: PR não pode ser mergeado
**Causa**: Algum check obrigatório falhou
**Solução**:

1. Verificar qual job falhou no GitHub Actions
2. Corrigir o problema na branch
3. Push das correções
4. Aguardar nova execução dos checks

### Testes Falhando

**Problema**: Unit ou Integration tests falhando
**Causa**: Código com bug ou testes quebrados
**Solução**:

```bash
# Executar testes localmente
dotnet test --configuration Release

# Para testes de integração
docker-compose up -d postgres sqlserver
dotnet test --filter "Category=Integration"
```

### Security Scan Falhou

**Problema**: Vulnerabilidades detectadas
**Causa**: Dependências com problemas de segurança
**Solução**:

```bash
# Verificar vulnerabilidades
dotnet list package --vulnerable --include-transitive

# Atualizar pacotes
dotnet outdated
dotnet add package PackageName --version NewVersion
```

### Build Failure

**Problema**: Falha na compilação
**Causa**: Erros de código
**Solução**:

```bash
# Build local
dotnet build --configuration Release

# Verificar warnings
dotnet build --verbosity normal
```

## 📈 Monitoring & Metrics

### Code Coverage

- **Target**: Mínimo 80% de cobertura
- **Tool**: Codecov
- **Reports**: Automáticos em cada PR

### Performance Metrics

- **Build Time**: ~5-10 minutos
- **Test Execution**: ~3-5 minutos
- **Security Scan**: ~2-3 minutos

### Success Metrics

- **Build Success Rate**: >95%
- **Test Pass Rate**: >98%
- **Security Issues**: 0 HIGH/CRITICAL

## 🎉 Benefits

### Para Desenvolvedores

- ✅ **Confidence**: Código sempre testado antes do merge
- ✅ **Quality**: Multiple validation layers
- ✅ **Security**: Automatic vulnerability detection
- ✅ **Documentation**: Automated release notes

### Para o Projeto

- 🚀 **Automated Publishing**: Zero-touch releases
- 🔒 **Security First**: Continuous security monitoring
- 📊 **Visibility**: Complete CI/CD transparency
- ⚡ **Fast Feedback**: Quick validation cycles

## 🔗 Links Úteis

- [GitHub Actions Documentation](https://docs.github.com/en/actions)
- [Branch Protection Rules](https://docs.github.com/en/repositories/configuring-branches-and-merges-in-your-repository/defining-the-mergeability-of-pull-requests/about-protected-branches)
- [NuGet Publishing](https://docs.microsoft.com/en-us/nuget/nuget-org/publish-a-package)
- [Codecov Integration](https://docs.codecov.com/docs/quick-start)
- [Snyk Security](https://docs.snyk.io/integrations/ci-cd-integrations/github-actions-integration)
