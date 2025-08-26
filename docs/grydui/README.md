# GrydUI Framework Documentation

Welcome to GrydUI - a comprehensive React framework designed for enterprise-grade microfrontend applications using Module Federation.

## Table of Contents

1. [Overview](#overview)
2. [Architecture](#architecture)
3. [Quick Start](#quick-start)
4. [Packages](#packages)
5. [Module Federation](#module-federation)
6. [Development Guide](#development-guide)
7. [Deployment](#deployment)
8. [API Reference](#api-reference)
9. [Examples](#examples)
10. [Contributing](#contributing)

## Overview

GrydUI is a monorepo framework that provides:

- **Module Federation Architecture**: Independent microfrontends with shared dependencies
- **Container Shell Pattern**: Main application that dynamically loads microfrontends
- **Component Library**: Reusable UI components with Tailwind CSS
- **API Client**: Unified HTTP client with authentication and error handling
- **CRUD Operations**: Full-stack CRUD functionality with backend integration
- **Layout System**: Responsive layouts with dynamic header updates
- **CLI Tools**: Automated project generation and scaffolding

## Architecture

### High-Level Architecture

```
┌─────────────────────────────────────────────────────────┐
│                Container Shell (Port 3000)              │
│  ┌─────────────────────────────────────────────────────┐│
│  │              MaterialHeader                         ││
│  │         (Dynamic title, breadcrumbs)               ││
│  └─────────────────────────────────────────────────────┘│
│  ┌─────────────────────────────────────────────────────┐│
│  │              MicrofrontendLoader                   ││
│  │         (Dynamic loading & error boundaries)       ││
│  └─────────────────────────────────────────────────────┘│
└─────────────────────────────────────────────────────────┘
                             │
                             │ Module Federation
                             ▼
┌─────────────────┐ ┌─────────────────┐ ┌─────────────────┐
│  Microfrontend  │ │  Microfrontend  │ │  Microfrontend  │
│   Users (3001)  │ │ Products (3002) │ │ Orders (3003)   │
│                 │ │                 │ │                 │
│ ┌─────────────┐ │ │ ┌─────────────┐ │ │ ┌─────────────┐ │
│ │ GrydCrud    │ │ │ │ GrydForms   │ │ │ │ GrydCharts  │ │
│ │ Components  │ │ │ │ Components  │ │ │ │ Components  │ │
│ └─────────────┘ │ │ └─────────────┘ │ │ └─────────────┘ │
└─────────────────┘ └─────────────────┘ └─────────────────┘
```

### Module Federation Flow

1. **Container Shell** loads and initializes
2. **Shared Dependencies** (React, @gryd packages) are provided
3. **Microfrontends** are dynamically loaded via `remoteEntry.js`
4. **EventBus Communication** enables cross-microfrontend messaging
5. **Header Updates** happen automatically via event system

### Package Dependencies

```
@gryd/api-client (Core HTTP client, EventBus)
      ↑
@gryd/components (Base UI components)
      ↑
@gryd/default-layout (Layout components, MaterialHeader)
      ↑
@gryd/crud (CRUD operations)
@gryd/forms (Form components)
@gryd/charts (Data visualization)
@gryd/auth (Authentication)
```

## Quick Start

### 1. Create Container Application

```bash
# Install CLI globally
npm install -g create-gryd-app

# Create container shell
create-gryd-app my-app create
# Choose: Container Shell
# Port: 3000
# Microfrontends: users,products,orders
# Additional packages: @gryd/crud,@gryd/forms
```

### 2. Create Microfrontends

```bash
# Create first microfrontend
create-gryd-app users create
# Choose: Microfrontend
# Port: 3001

# Create second microfrontend
create-gryd-app products create
# Choose: Microfrontend  
# Port: 3002
```

### 3. Development

```bash
# Terminal 1: Start container
cd my-app
npm install
npm start  # Runs on http://localhost:3000

# Terminal 2: Start users microfrontend
cd users
npm install
npm start  # Runs on http://localhost:3001

# Terminal 3: Start products microfrontend
cd products
npm install
npm start  # Runs on http://localhost:3002
```

### 4. Access Application

- Open `http://localhost:3000`
- Navigate to `/users` to see the users microfrontend
- Navigate to `/products` to see the products microfrontend

## Packages

### Core Packages

#### @gryd/api-client
HTTP client with authentication, error handling, and EventBus communication.

```typescript
import { HttpClient, MicroFrontendEventBus } from '@gryd/api-client';

// HTTP requests
const users = await HttpClient.get('/api/users');

// EventBus communication
MicroFrontendEventBus.emit('page:title-changed', 'Users Management');
```

#### @gryd/components
Base UI components built with Tailwind CSS and accessibility in mind.

```typescript
import { Button, DataTable, Modal } from '@gryd/components';

<Button variant="primary" onClick={handleClick}>
  Save Changes
</Button>
```

#### @gryd/default-layout
Layout components including the responsive MaterialHeader.

```typescript
import { MaterialHeader } from '@gryd/default-layout';

// Automatically listens to EventBus for title/breadcrumb updates
<MaterialHeader />
```

### Feature Packages

#### @gryd/crud
Complete CRUD operations with backend integration.

```typescript
import { GrydCrudProvider, useGrydCrudApi } from '@gryd/crud';

function UsersPage() {
  const { data, create, update, delete: remove } = useGrydCrudApi({
    endpoint: '/api/users',
    entityName: 'User'
  });

  return (
    <GrydCrudProvider>
      {/* CRUD operations automatically available */}
    </GrydCrudProvider>
  );
}
```

#### @gryd/forms
Advanced form components with validation and accessibility.

```typescript
import { FormBuilder, FormField } from '@gryd/forms';

<FormBuilder onSubmit={handleSubmit}>
  <FormField name="email" type="email" required />
  <FormField name="password" type="password" required />
</FormBuilder>
```

#### @gryd/charts
Data visualization components using Chart.js and D3.

```typescript
import { LineChart, BarChart, PieChart } from '@gryd/charts';

<LineChart 
  data={chartData} 
  options={{ responsive: true }}
/>
```

#### @gryd/auth
Authentication helpers and components.

```typescript
import { AuthProvider, useAuth, ProtectedRoute } from '@gryd/auth';

function App() {
  return (
    <AuthProvider>
      <ProtectedRoute component={Dashboard} />
    </AuthProvider>
  );
}
```

## Module Federation

### Configuration

#### Container Shell (webpack.config.js)
```javascript
new ModuleFederationPlugin({
  name: 'container',
  remotes: {
    users: 'http://localhost:3001',
    products: 'http://localhost:3002'
  },
  shared: {
    react: { singleton: true, eager: true },
    'react-dom': { singleton: true, eager: true },
    '@gryd/api-client': { singleton: true },
    '@gryd/components': { singleton: true }
  }
})
```

#### Microfrontend (webpack.config.js)
```javascript
new ModuleFederationPlugin({
  name: 'users',
  filename: 'remoteEntry.js',
  exposes: {
    './App': './src/App'
  },
  shared: {
    react: { singleton: true },
    'react-dom': { singleton: true },
    '@gryd/api-client': { singleton: true },
    '@gryd/components': { singleton: true }
  }
})
```

### Dynamic Loading

The `MicrofrontendLoader` component handles dynamic loading:

```typescript
<MicrofrontendLoader 
  name="users"
  url="http://localhost:3001"
  module="./App"
/>
```

### Error Boundaries

Automatic error handling for failed microfrontend loads:

```typescript
// Automatic retry mechanisms
// Fallback UI for errors
// Network failure handling
```

## Development Guide

### Setting Up Development Environment

1. **Clone Repository**
```bash
git clone https://github.com/gryd/grydui.git
cd grydui
```

2. **Install Dependencies**
```bash
npm install -g pnpm
pnpm install
```

3. **Build Packages**
```bash
pnpm run build
```

4. **Start Development**
```bash
pnpm run dev
```

### Creating New Components

1. **Add to @gryd/components**
```bash
cd packages/components/src
mkdir NewComponent
```

2. **Component Structure**
```
NewComponent/
├── index.ts
├── NewComponent.tsx
├── NewComponent.test.tsx
├── NewComponent.stories.tsx
└── types.ts
```

3. **Export Component**
```typescript
// packages/components/src/index.ts
export { NewComponent } from './NewComponent';
```

### Testing

```bash
# Unit tests
pnpm run test

# E2E tests
pnpm run test:e2e

# Visual regression tests
pnpm run test:visual
```

### Building

```bash
# Build all packages
pnpm run build

# Build specific package
pnpm run --filter "@gryd/components" build
```

## Deployment

### CI/CD Pipeline

Our GitHub Actions pipeline handles:

1. **Linting & Type Checking**
2. **Unit Testing**
3. **Building Packages**
4. **E2E Testing**
5. **Security Auditing**
6. **NPM Publishing**
7. **Documentation Deployment**

### Environment Strategy

#### Development
- Local development with Module Federation
- Hot reloading for all microfrontends
- Shared local packages

#### Staging
- Containerized deployment
- Integration testing
- Performance monitoring

#### Production
- CDN-hosted microfrontends
- Blue-green deployment
- Zero-downtime updates

### Deployment Commands

```bash
# Build for production
pnpm run build:prod

# Deploy to staging
pnpm run deploy:staging

# Deploy to production
pnpm run deploy:prod
```

## API Reference

### HttpClient

```typescript
class HttpClient {
  static get<T>(url: string, options?: RequestOptions): Promise<T>
  static post<T>(url: string, data?: any, options?: RequestOptions): Promise<T>
  static put<T>(url: string, data?: any, options?: RequestOptions): Promise<T>
  static delete<T>(url: string, options?: RequestOptions): Promise<T>
  static setBaseURL(url: string): void
  static setAuthToken(token: string): void
}
```

### MicroFrontendEventBus

```typescript
class MicroFrontendEventBus {
  static emit(event: string, data: any): void
  static on(event: string, callback: (data: any) => void): () => void
  static off(event: string, callback: (data: any) => void): void
  static once(event: string, callback: (data: any) => void): void
}

// Available Events
'page:title-changed': string
'page:subtitle-changed': string
'page:breadcrumb-changed': BreadcrumbItem[]
'layout:message': MessageConfig
'layout:loading': boolean
'navigation:change': string
```

### useGrydCrudApi

```typescript
function useGrydCrudApi<T>(config: CrudConfig) {
  return {
    data: T[],
    loading: boolean,
    error: Error | null,
    create: (item: Partial<T>) => Promise<T>,
    update: (id: string, item: Partial<T>) => Promise<T>,
    delete: (id: string) => Promise<void>,
    refresh: () => Promise<void>,
    pagination: PaginationState,
    filtering: FilterState,
    sorting: SortState
  }
}
```

## Examples

### Basic Microfrontend

```typescript
import React, { useEffect } from 'react';
import { MicroFrontendEventBus } from '@gryd/api-client';
import { Button } from '@gryd/components';

const UserManagement: React.FC = () => {
  useEffect(() => {
    // Update header when component loads
    MicroFrontendEventBus.emit('page:title-changed', 'User Management');
    MicroFrontendEventBus.emit('page:breadcrumb-changed', [
      { text: 'Home', path: '/' },
      { text: 'Users' }
    ]);
  }, []);

  return (
    <div className="p-6">
      <h1 className="text-2xl font-bold mb-4">Users</h1>
      <Button 
        onClick={() => console.log('Add user')}
        variant="primary"
      >
        Add User
      </Button>
    </div>
  );
};

export default UserManagement;
```

### CRUD Integration

```typescript
import { GrydCrudProvider, useGrydCrudApi } from '@gryd/crud';
import { DataTable } from '@gryd/components';

function ProductsPage() {
  const { 
    data, 
    loading, 
    create, 
    update, 
    delete: remove 
  } = useGrydCrudApi({
    endpoint: '/api/products',
    entityName: 'Product'
  });

  return (
    <GrydCrudProvider>
      <DataTable
        data={data}
        loading={loading}
        onEdit={update}
        onDelete={remove}
        columns={[
          { key: 'name', label: 'Name' },
          { key: 'price', label: 'Price' },
          { key: 'category', label: 'Category' }
        ]}
      />
    </GrydCrudProvider>
  );
}
```

### Form with Validation

```typescript
import { FormBuilder, FormField } from '@gryd/forms';
import { useGrydCrudApi } from '@gryd/crud';

function UserForm({ userId }: { userId?: string }) {
  const { create, update } = useGrydCrudApi({
    endpoint: '/api/users',
    entityName: 'User'
  });

  const handleSubmit = async (data: any) => {
    if (userId) {
      await update(userId, data);
    } else {
      await create(data);
    }
  };

  return (
    <FormBuilder onSubmit={handleSubmit}>
      <FormField 
        name="name" 
        label="Full Name"
        required 
        validation={{ minLength: 2 }}
      />
      <FormField 
        name="email" 
        label="Email"
        type="email" 
        required 
      />
      <FormField 
        name="role" 
        label="Role"
        type="select"
        options={[
          { value: 'admin', label: 'Administrator' },
          { value: 'user', label: 'User' }
        ]}
      />
    </FormBuilder>
  );
}
```

## Contributing

### Development Workflow

1. **Fork the repository**
2. **Create feature branch** (`git checkout -b feature/amazing-feature`)
3. **Make changes** and add tests
4. **Add changeset** (`pnpm exec changeset`)
5. **Commit changes** (`git commit -m 'feat: add amazing feature'`)
6. **Push to branch** (`git push origin feature/amazing-feature`)
7. **Open Pull Request**

### Code Standards

- **TypeScript**: Strict mode enabled
- **ESLint**: Airbnb configuration
- **Prettier**: Automatic formatting
- **Testing**: Jest + React Testing Library
- **Documentation**: JSDoc comments required

### Package Guidelines

1. **Follow semver** for version bumps
2. **Add tests** for new features
3. **Update documentation** 
4. **Use TypeScript** strictly
5. **Follow naming conventions**

### Release Process

We use Changesets for automated releases:

1. **Add changeset** for your changes
2. **CI validates** all checks pass
3. **Merge to main** triggers release
4. **NPM publishing** happens automatically

---

## Support

- **Documentation**: [docs.gryd.com](https://docs.gryd.com)
- **Issues**: [GitHub Issues](https://github.com/gryd/grydui/issues)
- **Discussions**: [GitHub Discussions](https://github.com/gryd/grydui/discussions)
- **Discord**: [GrydUI Community](https://discord.gg/grydui)

## License

MIT © GrydUI Team
