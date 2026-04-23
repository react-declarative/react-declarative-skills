# Dependency Injection

Lightweight IoC container inspired by Angular DI.

| Function | Purpose |
|---|---|
| `provide(key, factory)` | Register a factory (lazy singleton) |
| `inject(key)` | Get singleton (created on first call) |
| `createServiceManager(name)` | Isolated scope with fallback to global |

## Define a service

```typescript
export class ApiService {
  async getUser(id: string) {
    return fetch(`/api/v1/users/${id}`).then(r => r.json());
  }

  async prefetch() { await this.getUser('me'); }  // called when entering a route
  async unload()   {}                              // called when leaving a route
}
```

## Registration

```typescript
// ioc/TYPES.ts
export const TYPES = {
  apiService:  Symbol('apiService'),
  authService: Symbol('authService'),
} as const;

// ioc/config.ts
import { provide } from 'react-declarative';
provide(TYPES.apiService,  () => new ApiService());
provide(TYPES.authService, () => new AuthService());
```

## Inject

```typescript
// ioc/ioc.ts
import { inject } from 'react-declarative';
export const ioc = {
  apiService:  inject<ApiService>(TYPES.apiService),
  authService: inject<AuthService>(TYPES.authService),
};

// In a component:
import { ioc } from './ioc/ioc';
useEffect(() => { ioc.apiService.getUser(id).then(setUser); }, [id]);
```

## Scoped container

```typescript
import { createServiceManager } from 'react-declarative';
const checkoutContainer = createServiceManager('checkout');
checkoutContainer.provide(TYPES.checkoutService, () => new CheckoutService());
const svc = checkoutContainer.inject<CheckoutService>(TYPES.checkoutService);
// inject looks in the local container first, then falls back to global
```

## Route lifecycle

```tsx
const routes = [{
  path: '/dashboard',
  guard:    async () => await ioc.authService.isAuthenticated(),
  prefetch: async () => await ioc.apiService.prefetch(),
  unload:   async () => await ioc.apiService.unload(),
  redirect: () => !ioc.authService.hasRole('user') ? '/login' : null,
}];

export const App = () => <Switch history={history} items={routes} />;
```

> Do not call `serviceManager.clear()` in production — it resets all singletons.
