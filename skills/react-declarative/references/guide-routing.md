# Guide: Client-Side Routing with Switch

React-declarative ships its own router. Routes are plain `ISwitchItem[]` arrays — no JSX route declarations. Works with any `history` object from the `history` package.

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
  <Switch
    history={history}
    items={routes}
    Loader={() => <p>Loading...</p>}
    NotFound={() => <p>Page not found</p>}
  />
);
```

> Pass `element` as `() => <MyPage />`, not as `MyPage` — enables lazy instantiation.

## Guards — async auth checks

If `guard` returns `false`, the route simply does not render — no auto-redirect. Handle navigation inside guard if needed.

```tsx
{ path: '/mint-page', guard: async () => await ioc.roleService.has('whitelist'), element: () => <MintPage /> }
{ path: '/admin',     guard: () => ioc.authService.hasRole('admin'),             element: () => <AdminPage /> }
```

## `prefetch` / `unload` lifecycle hooks

`prefetch` runs before element mounts; `unload` runs when the route is left.

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

// Conditional
{ path: '/', redirect: () => ioc.authService.isAuthorized ? '/profile' : '/login' }

// With params
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
  {
    path: '/mint',
    guard: async () => await ioc.roleService.has('whitelist'),
    prefetch: async () => await ioc.ethersService.init(),
    unload:   async () => await ioc.ethersService.dispose(),
    redirect: () => {
      const { isMetamaskAvailable, isProviderConnected } = ioc.ethersService;
      return (!isMetamaskAvailable || !isProviderConnected) ? '/connect' : null;
    },
    element: () => <MintPage />,
  },
];
```

## Route hooks

```tsx
// URL params — reactively updates on location change
const params = useRouteParams<{ id: string }>(routes, history);
// params.id === '42' when URL is /users/42/card

// Full ISwitchItem object (for custom route properties)
interface IAppRoute extends ISwitchItem { sideMenu?: string; }
const currentRoute = useRouteItem<IAppRoute>(routes, history);
```

## Utility functions (outside React)

```tsx
import { getRouteParams, getRouteItem, parseRouteUrl } from 'react-declarative';

const params = getRouteParams(routes, window.location.pathname);  // { id: '42' }
const route  = getRouteItem(routes, window.location.pathname);   // ISwitchItem

parseRouteUrl('/ticket/:id/card', '/ticket/42/card');  // { id: '42' }
parseRouteUrl('/ticket/:id/card', '/dashboard');       // null
```

## Nested routing — `OutletView`

For views with sub-navigation (tabs: Card, Files, Settings):

```tsx
import { OutletView } from 'react-declarative';

const UserDetailPage = ({ history }) => (
  <OutletView
    history={history}
    routes={[
      { id: 'card',  isActive: (p) => !!parseRouteUrl('/users/:id/card',  p), element: UserCardView  },
      { id: 'files', isActive: (p) => !!parseRouteUrl('/users/:id/files', p), element: UserFilesView },
    ]}
  />
);

// Navigate:
history.replace(`/users/${id}/files`);
```
