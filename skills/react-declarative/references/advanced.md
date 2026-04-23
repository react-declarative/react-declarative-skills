# Advanced Topics Reference

---

## Routing — Switch Component

`<Switch />` is react-declarative's built-in router. Routes are `ISwitchItem[]` objects — no JSX route declarations.

```tsx
import { Switch, ISwitchItem } from 'react-declarative';
import { createBrowserHistory } from 'history';

const history = createBrowserHistory();

const routes: ISwitchItem[] = [
  { path: '/', redirect: () => ioc.authService.isAuthorized ? '/dashboard' : '/login' },
  { path: '/login', element: () => <LoginPage /> },
  { path: '/dashboard',
    guard: () => ioc.authService.hasRole('user'),
    element: () => <DashboardPage /> },
  { path: '/users/:id',
    guard: async () => await ioc.authService.hasRole('admin'),
    prefetch: async () => await ioc.apiService.prefetch(),
    unload: async () => await ioc.apiService.unload(),
    redirect: ({ id }) => `/users/${id}/card`,
  },
  { path: '/users/:id/card', element: () => <UserDetailPage /> },
];

export const App = () => (
  <Switch
    history={history}
    items={routes}
    Loader={() => <CircularProgress />}
    NotFound={() => <NotFoundPage />}
  />
);
```

### ISwitchItem fields

| Field | Type | Notes |
|---|---|---|
| `path` | `string` | path-to-regexp pattern |
| `element` | `() => ReactElement` | Factory function (not component reference!) |
| `guard` | `() => boolean \| Promise<boolean>` | Block route if returns false |
| `prefetch` | `() => Promise<void>` | Runs before element mounts |
| `unload` | `() => Promise<void>` | Runs when route is left |
| `redirect` | `string \| ((params) => string \| null)` | String = always; function = conditional |

### Nested routing with OutletView

```tsx
import { OutletView } from 'react-declarative';

const UserDetailPage = ({ history }) => (
  <OutletView
    history={history}
    routes={[
      { id: 'card',  isActive: (p) => !!parseRouteUrl('/users/:id/card', p),  element: UserCardView },
      { id: 'files', isActive: (p) => !!parseRouteUrl('/users/:id/files', p), element: UserFilesView },
    ]}
  />
);
// Navigate: history.replace(`/users/${id}/files`)
```

---

## Reactive Programming — Subject, BehaviorSubject, Source

### Subject
Push-based event emitter. Subscribers receive only future emissions.

```ts
import { Subject } from 'react-declarative';

const clickSubject = new Subject<MouseEvent>();
const unsub = clickSubject.subscribe((e) => console.log(e.clientX));
clickSubject.next(event);
unsub();
```

### BehaviorSubject
Like Subject but stores current value. New subscribers get it immediately.

```ts
import { BehaviorSubject } from 'react-declarative';

const userSubject = new BehaviorSubject<string | null>(null);
userSubject.next('alice');
console.log(userSubject.data);  // 'alice'
userSubject.subscribe((user) => ...);  // called immediately with 'alice'
```

### EventEmitter
Named-event pub/sub for multiple channels on one object.

```ts
import { EventEmitter } from 'react-declarative';

const emitter = new EventEmitter();
emitter.subscribe('login', (user) => ...);
emitter.emit('login', { name: 'alice' });
```

### Source — composable MapReduce pipelines

Use when joining multiple streams with stateful logic.

```ts
import { Source } from 'react-declarative';

const verifyEmitter = Source.multicast(() =>
  Source
    .join([captureStateEmitter, Source.fromInterval(1_000)])
    .reduce((acm, [{ state: isValid }]) => isValid ? acm + 1 : 0, 0)
    .tap((ticker) => { if (ticker === 1) recorder.beginCapture(); })
    .filter((ticker) => ticker === 3)
    .tap(() => recorder.endCapture())
);
```

| Operator | Purpose |
|---|---|
| `Source.join(observers[])` | Combine streams; emit when any emits |
| `Source.multicast(factory)` | Share single upstream among multiple consumers |
| `Source.fromInterval(ms)` | Emit void on fixed interval |
| `.reduce(fn, init)` | Accumulate across emissions |
| `.filter(fn)` | Drop non-matching emissions |
| `.tap(fn)` | Side effect without changing value |
| `.map(fn)` | Transform each emission |

---

## Dependency Injection — provide / inject

Lightweight IoC container inspired by Angular DI.

```ts
import { provide, inject, IService } from 'react-declarative';

// Define a service
export class ApiService implements IService {
  async getUser(id: string) { return fetch(`/api/users/${id}`).then(r => r.json()); }
  async prefetch() { /* warm caches */ }
  async unload() { /* cleanup */ }
}

// Register (lazy singleton — factory runs on first inject)
export const TYPES = { apiService: Symbol('apiService') };
provide(TYPES.apiService, () => new ApiService());

// Retrieve (singleton)
import { inject } from 'react-declarative';
export const ioc = {
  apiService: inject<ApiService>(TYPES.apiService),
};

// Use in components
const UserProfile = ({ id }) => {
  const [user, setUser] = useState(null);
  useEffect(() => { ioc.apiService.getUser(id).then(setUser); }, [id]);
  return user ? <div>{user.name}</div> : null;
};
```

### Scoped containers

```ts
import { createServiceManager } from 'react-declarative';

const checkoutContainer = createServiceManager('checkout');
checkoutContainer.provide(CHECKOUT_TYPES.checkoutService, () => new CheckoutService());
// inject checks local container first, falls back to global
const svc = checkoutContainer.inject<CheckoutService>(CHECKOUT_TYPES.checkoutService);
```

### Route lifecycle hooks

```ts
// Switch router calls prefetch/unload automatically:
{
  path: '/dashboard',
  guard: async () => await ioc.authService.isAuthenticated(),
  prefetch: async () => await ioc.apiService.prefetch(),
  unload: async () => await ioc.apiService.unload(),
}
```

---

## SlotFactory — Custom Field Renderers

### OneSlotFactory
Replace built-in field renderers globally within a subtree.

```tsx
import { OneSlotFactory, One } from 'react-declarative';
import { MyTextInput } from './MyTextInput';

export const App = () => (
  <OneSlotFactory Text={MyTextInput} CheckBox={MyCheckBox}>
    <One fields={fields} handler={handler} />
  </OneSlotFactory>
);
```

Available slots: `Text`, `CheckBox`, `Radio`, `Combo`, `Items`, `Choose`, `Date`, `Time`, `Switch`, `YesNo`, `Slider`, `Rating`, `Progress`, `File`, `Dict`, `Tree`, `Complete`, `Typography`, `Button`, `Icon`, `Line`

Each slot component receives a typed interface (e.g. `ITextSlot`, `ICheckBoxSlot`) from `react-declarative`.

```tsx
import { ITextSlot } from 'react-declarative';

export const MyTextInput = ({
  value, onChange, title, placeholder, disabled, readonly, invalid, dirty, description
}: ITextSlot) => (
  <div>
    <label>{title}</label>
    <input
      value={String(value ?? '')}
      disabled={disabled || readonly}
      onChange={(e) => onChange(e.target.value)}
    />
    {dirty && invalid && <span className="error">{invalid}</span>}
  </div>
);
```

Inner `OneSlotFactory` overrides outer ones — layer for section-specific overrides.

### ListSlotFactory
Same concept for `<List />` columns.

```tsx
<ListSlotFactory BodyRow={MyBodyRow}>
  <List columns={columns} handler={fetchRows} />
</ListSlotFactory>
```

### react-declarative-mantine
Pre-built `OneSlotFactory` with all fields redesigned for Mantine UI.

```bash
npm install react-declarative-mantine
```

```tsx
import { MantineSlotFactory } from 'react-declarative-mantine';
<MantineSlotFactory>
  <One fields={fields} handler={handler} />
</MantineSlotFactory>
```

---

## AI-Assisted Form Generation

`react-declarative` schemas are JSON arrays — ideal for LLM generation.

### Prompt workflow

1. Pick a reference sample from [docs/sample/](https://github.com/react-declarative/react-declarative/tree/master/docs)
2. Paste + describe what you want:

```text
Read the code below and generate a new form schema in the same format.
The form should collect: [your description here].
Use FieldType.Paper to group related fields.

[paste reference code here]
```

3. Verify in playground: https://react-declarative-playground.github.io/
4. Drop `fields` array into `<One fields={fields} handler={() => ({})} onChange={...} />`

### Key tips for LLM prompting

- Always include `import { TypedField, FieldType } from "react-declarative"` in the sample
- Ask for validation inside `isInvalid` callbacks (not external)
- Request `desktopColumns`/`tabletColumns`/`phoneColumns` explicitly for responsive layouts
- Use `gallery_of_controls.md` sample when you need less common field types

### Available reference samples (in docs/sample/)

- `order_info.md` — multi-section form with Paper, Date, Combo
- `login_form.md` — centered card with action feedback
- `profile_card.md` — avatar layout, nested groups, Component injection
- `settings_page.md` — Switch, Slider, Rating, Progress
- `gallery_of_controls.md` — every FieldType in one form
- `variant_form.md` — conditional visibility with `isVisible`
- `adaptive_form.md` — responsive breakpoints on every field
- `dashboard.md` — KPI cards, read-only display panels
