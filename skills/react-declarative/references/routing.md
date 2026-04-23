# Routing

react-declarative ships its own router. Works with any `history` object — no React Router required.

## Basic setup

```tsx
import { Switch, ISwitchItem } from 'react-declarative';
import { createBrowserHistory } from 'history';

const history = createBrowserHistory();

const routes: ISwitchItem[] = [
  { path: '/',        redirect: '/profile' },
  { path: '/profile', element: () => <ProfilePage /> },
  { path: '/login',   element: () => <LoginPage /> },
];

const App = () => (
  <Switch history={history} items={routes}
    Loader={() => <p>Loading...</p>}
    NotFound={() => <p>404</p>} />
);
```

Pass `element` as `() => <MyPage />`, not as `MyPage` — enables lazy instantiation.

## Guards

```tsx
{ path: '/admin', guard: async () => await ioc.authService.hasRole('admin'), element: () => <AdminPage /> }
```

If `guard` returns `false`, route simply doesn't render — no auto-redirect. Handle navigation inside guard if needed.

## prefetch / unload

```tsx
{
  path: '/mint',
  guard:    async () => await ioc.roleService.has('whitelist'),
  prefetch: async () => await ioc.ethersService.init(),
  unload:   async () => await ioc.ethersService.dispose(),
  element: () => <MintPage />,
}
```

## Redirects

```tsx
{ path: '/', redirect: '/profile' }
{ path: '/', redirect: () => ioc.authService.isAuthorized ? '/profile' : '/login' }
{ path: '/ticket/:id', redirect: ({ id }) => `/ticket/${id}/card` }
```

## Full production example

```tsx
const routes: ISwitchItem[] = [
  { path: '/',               redirect: () => ioc.authService.isAuthorized ? '/dashboard' : '/login' },
  { path: '/login',          element: () => <LoginPage /> },
  { path: '/dashboard',      guard: () => ioc.authService.hasFeature('dashboard_read'), element: () => <DashboardPage /> },
  { path: '/users',          guard: () => ioc.authService.hasRole('admin'), element: () => <UserListPage /> },
  { path: '/users/:id/card', guard: () => ioc.authService.hasRole('admin'), element: () => <UserDetailPage /> },
  { path: '/users/:id',      redirect: ({ id }) => `/users/${id}/card` },
];
```

## Route hooks

```tsx
import { useRouteParams, useRouteItem } from 'react-declarative';

// URL params for current route
const params = useRouteParams<{ id: string }>(routes, history);
// params.id === '42' when URL is /users/42/card

// Full ISwitchItem object (for custom route properties)
interface IAppRoute extends ISwitchItem { sideMenu?: string; }
const currentRoute = useRouteItem<IAppRoute>(routes, history);
```

## Utility functions (outside React)

```tsx
import { getRouteParams, getRouteItem, parseRouteUrl } from 'react-declarative';

const params = getRouteParams(routes, window.location.pathname);
const route  = getRouteItem(routes, window.location.pathname);

parseRouteUrl('/ticket/:id/card', '/ticket/42/card'); // { id: '42' }
parseRouteUrl('/ticket/:id/card', '/dashboard');      // null
```

## Nested routing — OutletView

```tsx
import { OutletView } from 'react-declarative';

const UserDetailPage = ({ history }) => (
  <OutletView history={history} routes={[
    { id: 'card',  isActive: (p) => !!parseRouteUrl('/users/:id/card',  p), element: UserCardView  },
    { id: 'files', isActive: (p) => !!parseRouteUrl('/users/:id/files', p), element: UserFilesView },
  ]} />
);

// Navigate:
history.replace(`/users/${id}/files`);
```
