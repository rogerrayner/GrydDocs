# Module Federation Guide

This guide covers everything you need to know about implementing Module Federation with GrydUI.

## Table of Contents

1. [Overview](#overview)
2. [Architecture](#architecture)
3. [Setup](#setup)
4. [Configuration](#configuration)
5. [Communication](#communication)
6. [Deployment](#deployment)
7. [Best Practices](#best-practices)
8. [Troubleshooting](#troubleshooting)

## Overview

Module Federation allows you to:
- **Split large applications** into manageable microfrontends
- **Independent deployments** for different teams
- **Shared dependencies** to reduce bundle size
- **Runtime integration** without build-time coupling

## Architecture

### Container Shell Pattern

```typescript
// Container Shell (Main App)
const App: React.FC = () => {
  return (
    <BrowserRouter>
      <MaterialHeader />
      <Routes>
        <Route path="/users/*" element={
          <MicrofrontendLoader 
            name="users"
            url="http://localhost:3001"
            module="./App"
          />
        } />
      </Routes>
    </BrowserRouter>
  );
};
```

### Microfrontend Structure

```typescript
// Microfrontend App
const App: React.FC = () => {
  useEffect(() => {
    // Communicate with container
    MicroFrontendEventBus.emit('page:title-changed', 'Users');
  }, []);

  return (
    <div>
      {/* Microfrontend content */}
    </div>
  );
};

export default App;
```

## Setup

### 1. Create Container Shell

```bash
create-gryd-app my-container create
# Choose: Container Shell
# Microfrontends: users,products,orders
```

This generates:

```
my-container/
├── webpack.config.js     # Module Federation config
├── src/
│   ├── App.tsx          # Main router
│   └── components/
│       └── MicrofrontendLoader.tsx
└── @types/
    ├── users-federation.d.ts
    ├── products-federation.d.ts
    └── orders-federation.d.ts
```

### 2. Create Microfrontends

```bash
create-gryd-app users create
# Choose: Microfrontend
# Port: 3001

create-gryd-app products create  
# Choose: Microfrontend
# Port: 3002
```

## Configuration

### Container Webpack Config

```javascript
// webpack.config.js
const ModuleFederationPlugin = require('@module-federation/webpack');

module.exports = {
  plugins: [
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
  ]
};
```

### Microfrontend Webpack Config

```javascript
// webpack.config.js
const ModuleFederationPlugin = require('@module-federation/webpack');

module.exports = {
  plugins: [
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
  ]
};
```

### TypeScript Declarations

```typescript
// @types/users-federation.d.ts
declare module 'users/App' {
  import { ComponentType } from 'react';
  const App: ComponentType;
  export default App;
}
```

## Communication

### EventBus System

The `MicroFrontendEventBus` enables communication between microfrontends and the container:

```typescript
import { MicroFrontendEventBus } from '@gryd/api-client';

// Send events from microfrontend
MicroFrontendEventBus.emit('page:title-changed', 'New Title');
MicroFrontendEventBus.emit('page:breadcrumb-changed', [
  { text: 'Home', path: '/' },
  { text: 'Users' }
]);

// Listen in container or other microfrontends
MicroFrontendEventBus.on('page:title-changed', (title) => {
  setHeaderTitle(title);
});
```

### Available Events

#### Header Updates
```typescript
// Update page title
MicroFrontendEventBus.emit('page:title-changed', 'User Management');

// Update subtitle
MicroFrontendEventBus.emit('page:subtitle-changed', 'Manage system users');

// Update breadcrumbs
MicroFrontendEventBus.emit('page:breadcrumb-changed', [
  { text: 'Dashboard', path: '/' },
  { text: 'Users', path: '/users' },
  { text: 'Edit User' }
]);
```

#### Layout Messages
```typescript
// Show success message
MicroFrontendEventBus.emit('layout:message', {
  type: 'success',
  text: 'User created successfully!'
});

// Show loading state
MicroFrontendEventBus.emit('layout:loading', true);
```

#### Navigation
```typescript
// Navigate to different route
MicroFrontendEventBus.emit('navigation:change', '/products');
```

### Cross-Microfrontend Communication

```typescript
// Microfrontend A sends data
MicroFrontendEventBus.emit('user:selected', { userId: 123 });

// Microfrontend B receives data
MicroFrontendEventBus.on('user:selected', (data) => {
  loadUserDetails(data.userId);
});
```

## Deployment

### Development Environment

Each microfrontend runs independently:

```bash
# Terminal 1: Container
cd container && npm start  # Port 3000

# Terminal 2: Users microfrontend  
cd users && npm start      # Port 3001

# Terminal 3: Products microfrontend
cd products && npm start   # Port 3002
```

### Production Environment

#### Option 1: Single Domain with Proxying

```nginx
# nginx.conf
location /users/ {
  proxy_pass http://users-service:3001/;
}

location /products/ {
  proxy_pass http://products-service:3002/;
}

location / {
  proxy_pass http://container-service:3000/;
}
```

#### Option 2: Subdomain Strategy

```
https://app.company.com          → Container Shell
https://users.app.company.com    → Users Microfrontend  
https://products.app.company.com → Products Microfrontend
```

Container config:
```javascript
remotes: {
  users: 'https://users.app.company.com',
  products: 'https://products.app.company.com'
}
```

### Docker Deployment

```dockerfile
# Dockerfile for microfrontend
FROM node:18-alpine
WORKDIR /app
COPY package*.json ./
RUN npm ci --only=production
COPY dist ./dist
EXPOSE 3001
CMD ["npm", "start"]
```

```yaml
# docker-compose.yml
version: '3.8'
services:
  container:
    build: ./container
    ports:
      - "3000:3000"
    environment:
      - USERS_URL=http://users:3001
      - PRODUCTS_URL=http://products:3002
      
  users:
    build: ./users
    ports:
      - "3001:3001"
      
  products:
    build: ./products
    ports:
      - "3002:3002"
```

## Best Practices

### 1. Shared Dependencies

**Do share:**
- React, React DOM
- @gryd packages (api-client, components)
- Major libraries (react-router, styled-components)

**Don't share:**
- Business logic libraries
- Microfrontend-specific dependencies
- Frequently changing packages

```javascript
shared: {
  react: { singleton: true, eager: true },
  'react-dom': { singleton: true, eager: true },
  '@gryd/components': { singleton: true },
  // ❌ Don't share microfrontend-specific deps
  // 'user-service-client': false
}
```

### 2. Error Boundaries

Always wrap microfrontends in error boundaries:

```typescript
const MicrofrontendLoader: React.FC = ({ name, url, module }) => {
  return (
    <ErrorBoundary fallback={<ErrorFallback />}>
      <Suspense fallback={<Loading />}>
        <RemoteComponent name={name} url={url} module={module} />
      </Suspense>
    </ErrorBoundary>
  );
};
```

### 3. Loading States

Provide feedback during microfrontend loading:

```typescript
const Loading: React.FC = () => (
  <div className="flex items-center justify-center p-8">
    <div className="animate-spin rounded-full h-8 w-8 border-b-2 border-blue-600"></div>
    <span className="ml-2">Loading microfrontend...</span>
  </div>
);
```

### 4. Routing Strategy

Use nested routing for microfrontends:

```typescript
// Container routing
<Route path="/users/*" element={<UsersApp />} />

// Microfrontend internal routing
const UsersApp = () => (
  <Routes>
    <Route path="/" element={<UsersList />} />
    <Route path="/create" element={<CreateUser />} />
    <Route path="/:id" element={<UserDetails />} />
  </Routes>
);
```

### 5. State Management

Keep state isolated within microfrontends:

```typescript
// ✅ Good: Isolated state
const UsersApp = () => {
  const [users, setUsers] = useState([]);
  // Handle state locally
};

// ❌ Bad: Shared global state
// Don't share Redux stores across microfrontends
```

## Troubleshooting

### Common Issues

#### 1. Module Not Found Error

```bash
Error: Cannot resolve 'users/App'
```

**Solutions:**
- Check microfrontend is running on correct port
- Verify `exposes` configuration in microfrontend
- Ensure `remotes` URL is correct in container

#### 2. React Version Conflicts

```bash
Error: Multiple versions of React detected
```

**Solutions:**
- Ensure React is marked as singleton in all configs
- Use same React version across all microfrontends
- Check shared dependencies configuration

#### 3. CORS Issues

```bash
Access to fetch at 'http://localhost:3001/remoteEntry.js' from origin 'http://localhost:3000' has been blocked by CORS policy
```

**Solutions:**
```javascript
// webpack.config.js - devServer
devServer: {
  headers: {
    'Access-Control-Allow-Origin': '*',
  },
}
```

#### 4. TypeScript Errors

```bash
Module '"users/App"' has no exported member 'default'
```

**Solutions:**
- Add federation type declarations
- Use proper module resolution
- Check export/import statements

### Debugging Tools

#### 1. Module Federation DevTools

```bash
# Install devtools
npm install @module-federation/devtools
```

#### 2. Network Tab Analysis

Check browser network tab for:
- `remoteEntry.js` loading successfully
- Shared chunks being loaded
- Error responses from microfrontend URLs

#### 3. Console Logging

```typescript
// Debug microfrontend loading
console.log('Loading microfrontend:', name, url);

// Debug EventBus communication  
MicroFrontendEventBus.on('*', (event, data) => {
  console.log('EventBus:', event, data);
});
```

### Performance Optimization

#### 1. Lazy Loading

```typescript
const LazyMicrofrontend = lazy(() => import('users/App'));
```

#### 2. Preloading

```typescript
// Preload microfrontends
useEffect(() => {
  const link = document.createElement('link');
  link.rel = 'prefetch';
  link.href = 'http://localhost:3001/remoteEntry.js';
  document.head.appendChild(link);
}, []);
```

#### 3. Bundle Analysis

```bash
# Analyze webpack bundles
npx webpack-bundle-analyzer dist/static/js/*.js
```

## Advanced Patterns

### 1. Dynamic Remote Discovery

```typescript
// Load remote configuration dynamically
const loadRemoteConfig = async () => {
  const config = await fetch('/api/microfrontends').then(r => r.json());
  return config.remotes;
};
```

### 2. A/B Testing with Microfrontends

```typescript
// Load different versions based on feature flags
const getUsersApp = () => {
  return featureFlags.newUsersUI 
    ? import('users-v2/App')
    : import('users/App');
};
```

### 3. Micro-Application Shell

```typescript
// Shell that can load any microfrontend
const MicroAppShell = ({ appName, appUrl }) => (
  <MicrofrontendLoader 
    name={appName}
    url={appUrl}
    module="./App"
  />
);
```

---

This guide should help you implement Module Federation successfully with GrydUI. For more advanced use cases, refer to the [Webpack Module Federation documentation](https://webpack.js.org/concepts/module-federation/).
