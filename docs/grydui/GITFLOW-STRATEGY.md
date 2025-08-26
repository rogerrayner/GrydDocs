# GrydUI Gitflow Strategy

This document outlines the Git workflow strategy for the GrydUI monorepo, designed for efficient package management and Module Federation deployment.

## Branch Structure

### Main Branches

- **`main`** - Production-ready code
  - All releases are tagged from this branch
  - Protected branch requiring PR reviews
  - Automatically triggers NPM publishing via CI/CD
  - Deploy to production environments

- **`develop`** - Integration branch for features
  - Latest development changes
  - All feature branches merge here first
  - Staging environment deployments
  - Pre-release testing

### Supporting Branches

- **`feature/*`** - New features or enhancements
  - Branch from: `develop`
  - Merge to: `develop`
  - Naming: `feature/package-description` (e.g., `feature/crud-pagination`)

- **`hotfix/*`** - Critical production fixes
  - Branch from: `main`
  - Merge to: `main` and `develop`
  - Naming: `hotfix/issue-description`

- **`release/*`** - Prepare releases
  - Branch from: `develop`
  - Merge to: `main` and `develop`
  - Naming: `release/v1.2.0`

- **`package/*`** - Package-specific changes
  - Branch from: `develop`
  - Merge to: `develop`
  - Naming: `package/components-redesign`

## Package Management Workflow

### 1. Feature Development

```bash
# Start new feature
git checkout develop
git pull origin develop
git checkout -b feature/crud-advanced-filters

# Development work...
git add .
git commit -m "feat(crud): add advanced filtering options"

# Push and create PR to develop
git push origin feature/crud-advanced-filters
```

### 2. Version Management with Changesets

We use Changesets for version management:

```bash
# Add a changeset for your changes
pnpm exec changeset

# Select packages that changed
# Choose version bump type (major, minor, patch)
# Write description of changes
```

Example changeset:
```yaml
---
"@gryd/crud": minor
"@gryd/components": patch
---

Add advanced filtering capabilities to CRUD operations
- New FilterBuilder component
- Enhanced search functionality
- Bug fix for table pagination
```

### 3. Release Process

#### Automated Release (Recommended)
1. Changes accumulate in `develop`
2. When ready for release, merge `develop` → `main`
3. CI/CD automatically:
   - Builds all packages
   - Runs tests and security audits
   - Creates release PR with changelog
   - Publishes to NPM when PR is merged

#### Manual Release
```bash
# Create release branch
git checkout develop
git checkout -b release/v1.2.0

# Update versions and generate changelog
pnpm exec changeset version

# Commit version changes
git add .
git commit -m "chore: release v1.2.0"

# Merge to main
git checkout main
git merge release/v1.2.0
git tag v1.2.0
git push origin main --tags

# Merge back to develop
git checkout develop
git merge main
git push origin develop
```

## Module Federation Deployment Strategy

### Development Environment
- Container Shell: `http://localhost:3000`
- Microfrontends: `http://localhost:3001+`
- Independent development and testing

### Staging Environment
- Container Shell: `https://staging-app.gryd.com`
- Microfrontends: `https://staging-{mf-name}.gryd.com`
- Integration testing across teams

### Production Environment
- Container Shell: `https://app.gryd.com`
- Microfrontends: `https://{mf-name}.gryd.com`
- Blue-green deployment for zero downtime

### Deployment Flow

1. **Feature Development**
   ```
   Developer → feature/* → develop → staging deployment
   ```

2. **Production Release**
   ```
   develop → main → production deployment
   ```

3. **Hotfix**
   ```
   main → hotfix/* → main → immediate production deployment
   ```

## Team Collaboration

### For Package Maintainers
- Work in `package/*` branches for major package changes
- Use `feature/*` for cross-package features
- Always update documentation and tests

### For Application Teams
- Use `feature/*` branches for application features
- Reference stable package versions in production
- Test with latest packages in development

### For DevOps Team
- Monitor CI/CD pipeline health
- Manage deployment configurations
- Handle production hotfixes

## Branch Protection Rules

### `main` Branch
- Require PR reviews (2 reviewers)
- Require status checks to pass
- Require branches to be up to date
- Restrict pushes to admins only
- Include administrators in restrictions

### `develop` Branch
- Require PR reviews (1 reviewer)
- Require status checks to pass
- Allow force pushes to admins

## Commit Message Convention

We follow Conventional Commits:

```
<type>(<scope>): <description>

[optional body]

[optional footer(s)]
```

### Types
- `feat`: New feature
- `fix`: Bug fix
- `docs`: Documentation changes
- `style`: Code style changes
- `refactor`: Code refactoring
- `test`: Adding tests
- `chore`: Maintenance tasks
- `ci`: CI/CD changes

### Scopes
- Package names: `crud`, `components`, `auth`, etc.
- Feature areas: `layout`, `forms`, `charts`
- Special: `workspace`, `cli`, `deps`

### Examples
```bash
feat(crud): add pagination support
fix(components): resolve button hover state
docs(readme): update installation instructions
chore(deps): update react to v18.2.0
ci(github): add security audit workflow
```

## Emergency Procedures

### Critical Production Bug
1. Create hotfix branch from `main`
2. Fix the issue with minimal changes
3. Test thoroughly
4. Create emergency PR to `main`
5. Deploy immediately after approval
6. Merge hotfix to `develop`

### Rollback Strategy
1. Use GitHub releases to identify stable version
2. Revert problematic changes
3. Deploy previous stable version
4. Investigate and fix in development

## Tools and Scripts

### Useful Commands
```bash
# Check which packages changed
pnpm exec changeset status

# Preview version changes
pnpm exec changeset version --snapshot

# Build all packages
pnpm run build

# Run tests across all packages
pnpm run test

# Lint all packages
pnpm run lint

# Create new package
pnpm exec create-gryd-package

# Create new microfrontend
pnpm exec create-gryd-app my-app --template microfrontend
```

### Git Aliases (Optional)
```bash
git config alias.co checkout
git config alias.br branch
git config alias.ci commit
git config alias.st status
git config alias.unstage 'reset HEAD --'
git config alias.last 'log -1 HEAD'
git config alias.visual '!gitk'
```

## Best Practices

1. **Keep PRs Small**: Focus on single features or fixes
2. **Write Clear Commits**: Use conventional commit format
3. **Update Documentation**: Keep docs in sync with code
4. **Add Tests**: Ensure new features have proper test coverage
5. **Review Code**: Provide constructive feedback in PRs
6. **Sync Regularly**: Keep your branches up to date
7. **Use Changesets**: Always add changesets for package changes
8. **Test Integration**: Verify Module Federation works across packages

## Troubleshooting

### Common Issues
- **Merge Conflicts**: Resolve locally, test, then push
- **Failed CI**: Check logs, fix issues, force-push if needed
- **Missing Changesets**: Add changeset before merging to main
- **Version Conflicts**: Use changeset CLI to resolve

### Getting Help
- Check existing issues and documentation
- Ask in team chat channels
- Create issue with detailed reproduction steps
- Contact package maintainers for specific packages
