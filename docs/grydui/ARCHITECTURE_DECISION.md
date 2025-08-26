/**
 * Micro-frontend Architecture Decision Document (ADR)
 * 
 * DECISION: Hybrid Approach - Monorepo + Dynamic Loading
 * 
 * CONTEXT:
 * - Performance issues with iframe approach
 * - Infrastructure complexity with multiple ports
 * - Need for component sharing and consistent theming
 * 
 * DECISION:
 * We will implement a hybrid approach that combines:
 * 1. Monorepo structure for development
 * 2. Dynamic component loading for runtime flexibility
 * 3. Shared dependency optimization
 * 4. Single deployment unit
 * 
 * CONSEQUENCES:
 * + Better performance (no iframe overhead)
 * + Simplified infrastructure (single port in production)
 * + Better component sharing
 * + Improved SEO and accessibility
 * + Easier state management
 * - Slightly more complex build process
 * - Runtime loading complexity
 */

// New Architecture Implementation
export interface MicrofrontendConfig {
  name: string;
  path: string;
  loader: () => Promise<React.ComponentType>;
  fallback?: React.ComponentType;
  permissions?: string[];
  metadata?: {
    version: string;
    author: string;
    description: string;
  };
}

export interface MicrofrontendRegistry {
  register(config: MicrofrontendConfig): void;
  load(name: string): Promise<React.ComponentType>;
  unload(name: string): void;
  list(): MicrofrontendConfig[];
}

// Implementation will follow...
