# GrydFrameworkDocs

Documentação centralizada para todos os frameworks Gryd

## 🚀 Visão Geral

Este repositório contém a documentação unificada para todos os frameworks da família Gryd:

- **GrydUI Framework** - Framework React com Module Federation para microfrontends
- **GrydAuthentication Framework** - Framework de autenticação e autorização enterprise
- **GrydCrud Framework** - Framework completo para operações CRUD com Clean Architecture
- **GrydUserTenant Framework** - Framework para gestão de usuários e multi-tenancy

## 📁 Estrutura do Projeto

```
GrydFrameworkDocs/
├── index.html                           # Página principal com navegação
├── README.md                           # Este arquivo
├── .github/
│   └── workflows/
│       └── deploy.yml                  # GitHub Actions para deploy automático
└── docs/
    ├── grydui/
    │   ├── index.html                  # Página principal do GrydUI
    │   ├── README.html                 # Documentação convertida
    │   ├── MODULE-FEDERATION-GUIDE.html
    │   ├── GITFLOW-STRATEGY.html
    │   └── ...                         # Outras documentações
    ├── gryd-authentication/
    │   ├── index.html                  # Página principal do GrydAuthentication
    │   ├── README.html
    │   ├── CONFIGURATION-GUIDE.html
    │   ├── SECURE-MULTIAPP-SETUP.html
    │   ├── TROUBLESHOOTING-GUIDE.html
    │   └── ...                         # Outras documentações
    ├── gryd-crud/
    │   ├── index.html                  # Página principal do GrydCrud
    │   ├── README.html
    │   ├── README-IMPLEMENTACAO.html
    │   └── ...                         # Outras documentações
    └── gryd-user-tenant/
        ├── index.html                  # Página principal do GrydUserTenant
        ├── README.html
        └── ...                         # Outras documentações
```

## 🌐 GitHub Pages Setup

### 1. Configuração do Repositório

1. **Criar Repositório no GitHub:**
   ```bash
   # No diretório GrydFrameworkDocs
   git init
   git add .
   git commit -m "Initial commit: Centralized Gryd Framework Documentation"
   git branch -M main
   git remote add origin https://github.com/YOUR_USERNAME/GrydFrameworkDocs.git
   git push -u origin main
   ```

2. **Configurar GitHub Pages:**
   - Vá para Settings → Pages
   - Source: GitHub Actions
   - O workflow `.github/workflows/deploy.yml` será executado automaticamente

### 2. Workflow de Deploy Automático

O arquivo `.github/workflows/deploy.yml` contém:

```yaml
name: Deploy to GitHub Pages

on:
  push:
    branches: [ main ]
  pull_request:
    branches: [ main ]

jobs:
  deploy:
    runs-on: ubuntu-latest
    steps:
    - uses: actions/checkout@v3
    
    - name: Setup Pages
      uses: actions/configure-pages@v3
      
    - name: Upload artifact
      uses: actions/upload-pages-artifact@v2
      with:
        path: '.'
        
    - name: Deploy to GitHub Pages
      id: deployment
      uses: actions/deploy-pages@v2
```

### 3. Configuração de Domínio (Opcional)

Para usar um domínio customizado:

1. Adicionar arquivo `CNAME` na raiz:
   ```
   docs.gryd.com
   ```

2. Configurar DNS para apontar para GitHub Pages:
   ```
   CNAME: YOUR_USERNAME.github.io
   ```

## 🔄 Atualização da Documentação

### Processo Manual

1. **Copiar arquivos dos repositórios fonte:**
   ```bash
   # Copiar do GrydUI
   cp ../GrydUI/README.md docs/grydui/
   cp ../GrydUI/docs/* docs/grydui/
   
   # Copiar do GrydAuthentication
   cp ../GrydAuthenticationFramework/README.md docs/gryd-authentication/
   cp ../GrydAuthenticationFramework/docs/* docs/gryd-authentication/
   
   # Etc...
   ```

2. **Converter Markdown para HTML** (se necessário):
   ```bash
   # Usar pandoc ou similar para converter .md para .html
   pandoc README.md -o README.html --standalone --css=../styles.css
   ```

3. **Commit e Push:**
   ```bash
   git add .
   git commit -m "docs: Update documentation from source repositories"
   git push
   ```

### Processo Automatizado (Futuro)

Considerações para automação:

1. **GitHub Actions com Submodules:**
   - Adicionar repositórios como submodules
   - Workflow para sync automático

2. **API GitHub para sync:**
   - Script que usa GitHub API para baixar arquivos
   - Execução agendada (cron)

3. **Webhooks:**
   - Configurar webhooks nos repos fonte
   - Trigger automático quando documentação é atualizada

## 🎨 Customização

### Temas e Estilos

- **Tailwind CSS**: Framework CSS usado para styling
- **Highlight.js**: Syntax highlighting para código
- **Cores por Framework**:
  - GrydUI: Azul (#3b82f6)
  - GrydAuthentication: Verde (#059669)
  - GrydCrud: Roxo (#7c3aed)
  - GrydUserTenant: Vermelho (#dc2626)

### Adicionando Novo Framework

1. **Criar pasta em `/docs/`:**
   ```bash
   mkdir docs/novo-framework
   ```

2. **Criar index.html baseado nos existentes**

3. **Adicionar card na página principal** (`index.html`)

4. **Copiar documentações do framework**

## 🔧 Desenvolvimento Local

Para testar localmente:

```bash
# Servir arquivos estáticos
python -m http.server 8000
# ou
npx serve .

# Acessar: http://localhost:8000
```

## 📝 Contributing

1. Fork o repositório
2. Crie uma branch para sua feature
3. Faça commit das mudanças
4. Abra um Pull Request

## 📄 License

Este projeto está sob a licença MIT. Veja o arquivo LICENSE para detalhes.

## 🔗 Links Úteis

- [GitHub Pages Documentation](https://docs.github.com/en/pages)
- [GitHub Actions Documentation](https://docs.github.com/en/actions)
- [Tailwind CSS](https://tailwindcss.com/)
- [Highlight.js](https://highlightjs.org/)

---

**Desenvolvido pela equipe Gryd** 🚀
