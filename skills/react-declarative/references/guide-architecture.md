# Guide: Project Architecture with react-declarative

Based on the reference CRM application: `git clone https://github.com/react-declarative/react-pocketbase-crm.git`

---

## Folder structure

```
src/
├── api/                    # React hooks that wrap service paginate functions
├── assets/                 # Declarative UI configs — TypedField[] and IColumn[]
│   ├── employee_fields/    # Form schema for create/edit forms
│   └── employee_columns/   # Column config for data grids
├── components/
│   ├── common/             # App shell (App.tsx, AppHeader, Logo, Version)
│   ├── hoc/                # heavy.tsx (lazy load), redirect.tsx (auth guard)
│   ├── provider/           # AlertProvider, LayoutProvider
│   └── slot/               # OneSlotFactory overrides (DateSlot, TimeSlot)
├── config/
│   ├── routes.tsx          # ISwitchItem[] — all app routes + sideMenu path hints
│   ├── sidemenu.ts         # IScaffold2Group[] — left sidebar menu tree
│   ├── scaffoldmenu.ts     # Top-bar action buttons
│   ├── theme.ts            # MUI createTheme (dark mode)
│   ├── params.ts           # process.env variables with defaults
│   └── dayjs.ts            # dayjs locale init
├── hooks/                  # Custom hooks composing modals + list actions
├── i18n/                   # Internationalization
├── lib/
│   ├── config.ts           # provide() registrations — the DI wiring file
│   ├── ioc.ts              # inject() calls — the singleton accessor object
│   ├── types.ts            # Symbol keys for all services
│   ├── services/
│   │   ├── base/           # Infrastructure: Pocketbase, Router, Layout, Alert, Error, Logger
│   │   ├── db/             # DAL: one service per collection, CRUD + paginate + Subjects
│   │   ├── view/           # App layer: delegates to DB, adds logging + side effects
│   │   └── global/         # Cross-cutting: PermissionService
│   ├── types/              # Shared TypeScript types (Paginator, etc.)
│   └── utils/              # readTransform, writeTransform, listTransform
├── pages/
│   ├── base/               # ErrorPage, LoginPage
│   └── view/               # Feature pages (EmployeePage, KanbanPage, SettingsPage)
├── styles/                 # makeStyles re-export from tss-react
├── utils/                  # UI utils: getRouteParams, hasRouteMatch, hasAny, hasAll
├── view/                   # Modal hooks: useEmployeeCreateModal, useEmployeePreviewModal
└── index.tsx               # Entry point: providers stack, OneConfig, OneSlotFactory
```

---

## Layered architecture

```
Pages & Components (UI)
        ↓
View Modals / Custom Hooks (Presentation)
        ↓
View Services (Application logic — logging, side effects, caching)
        ↓
DB Services (DAL — CRUD + pagination + real-time Subjects)
        ↓
Base Services (Infrastructure — Pocketbase, Router, Layout, Alert)
```

Each layer only calls the layer directly below it. UI never talks to DB services directly.

---

## DI container wiring

Three files work together:

**`lib/types.ts`** — define Symbol keys:
```ts
export const TYPES = {
  alertService:        Symbol.for('alertService'),
  layoutService:       Symbol.for('layoutService'),
  routerService:       Symbol.for('routerService'),
  pocketbaseService:   Symbol.for('pocketbaseService'),
  employeeDbService:   Symbol.for('employeeDbService'),
  employeeViewService: Symbol.for('employeeViewService'),
  permissionService:   Symbol.for('permissionService'),
} as const;
```

**`lib/config.ts`** — register factories with `provide()` (lazy singletons):
```ts
import { provide } from 'react-declarative';

// Infrastructure
provide(TYPES.alertService,      () => new AlertService());
provide(TYPES.layoutService,     () => new LayoutService());
provide(TYPES.routerService,     () => new RouterService());
provide(TYPES.pocketbaseService, () => new PocketbaseService());

// DAL
provide(TYPES.employeeDbService, () => new EmployeeDbService());

// Application layer
provide(TYPES.employeeViewService, () => new EmployeeViewService());

// Cross-cutting
provide(TYPES.permissionService, () => new PermissionService());
```

**`lib/ioc.ts`** — retrieve singletons with `inject()`:
```ts
import { inject } from 'react-declarative';

const ioc = {
  alertService:        inject<AlertService>(TYPES.alertService),
  layoutService:       inject<LayoutService>(TYPES.layoutService),
  routerService:       inject<RouterService>(TYPES.routerService),
  pocketbaseService:   inject<PocketbaseService>(TYPES.pocketbaseService),
  employeeDbService:   inject<EmployeeDbService>(TYPES.employeeDbService),
  employeeViewService: inject<EmployeeViewService>(TYPES.employeeViewService),
  permissionService:   inject<PermissionService>(TYPES.permissionService),
};

export default ioc;
```

Usage everywhere: `ioc.employeeViewService.create(data)` — no constructor arguments, no prop drilling.

---

## DB service pattern

One service per collection. Provides CRUD + paginate + real-time Subjects:

```ts
export class EmployeeDbService {
  // Real-time subjects — emit on WebSocket events
  public readonly reloadSubject  = new Subject<void>();
  public readonly createSubject  = new Subject<IEmployeeRow>();
  public readonly updateSubject  = new Subject<[string, IEmployeeRow]>();
  public readonly deleteSubject  = new Subject<string>();

  // Pagination handler for ListTyped
  paginate: Paginator = async (filterData, pagination, sort, chips, search, payload) => {
    // filter, search, sort, paginate against Pocketbase
    // return { rows, total }
  };

  async create(dto: IEmployeeDto): Promise<IEmployeeModel> { /* ... */ }
  async read(id: string): Promise<IEmployeeModel> { /* ... */ }
  async update(id: string, dto: IEmployeeDto): Promise<IEmployeeModel> { /* ... */ }
  async remove(id: string): Promise<boolean> { /* ... */ }

  // Called automatically by Switch router via prefetch
  protected async prefetch() {
    // subscribe to Pocketbase WebSocket, emit to subjects on changes
  }
}
```

Data normalization helpers:
- `readTransform` — empty arrays → null on read
- `writeTransform` — strips `id`, `collectionId`, `created`, `updated` on write
- `listTransform` — applies `readTransform` to an array

---

## View service pattern

Wraps a DB service. Adds logging, side effects (history records), caching:

```ts
export class EmployeeViewService {
  // Forward subjects from DB service directly
  public readonly createSubject = this.employeeDbService.createSubject;
  public readonly reloadSubject = this.employeeDbService.reloadSubject;
  public readonly updateSubject = this.employeeDbService.updateSubject;

  async create(dto: IEmployeeDto) {
    const result = await ioc.employeeDbService.create(dto);
    await ioc.historyViewService.create({ type: 'created', employee_id: result.id, ... });
    ioc.loggerService.log('employee created', { id: result.id });
    return result;
  }

  // paginate delegates directly, read/remove also delegate directly
  paginate = ioc.employeeDbService.paginate;
  async read(id: string) { return ioc.employeeDbService.read(id); }
  async remove(id: string) { return ioc.employeeDbService.remove(id); }
}
```

---

## Declarative UI assets

Keep `TypedField[]` and `IColumn[]` in `assets/`, not inside components:

```ts
// assets/employee_fields/employee_fields.tsx
export const employee_fields: TypedField<IEmployeeDto, IPayload>[] = [
  {
    type: FieldType.Combo,
    name: 'status',
    title: 'Status',
    itemList: ['active', 'pending', 'on_leave', 'inactive'],
    isVisible: (_, { visibility }) => visibility.has('status'),
  },
  {
    type: FieldType.Group,
    fields: [
      { type: FieldType.Text, name: 'first_name', title: 'First name',
        columns: '6', validation: { required: true } },
      { type: FieldType.Text, name: 'last_name',  title: 'Last name',
        columns: '6', validation: { required: true } },
    ],
  },
  { type: FieldType.Text, name: 'email', title: 'Email',
    isInvalid: ({ email }) => /^[\w-.]+@[\w-.]+\.\w+$/.test(email) ? null : 'Invalid email',
    isVisible: (_, { visibility }) => visibility.has('email') },
];
```

```ts
// assets/employee_columns/employee_columns.tsx
export const employee_columns: IColumn<IPayload, IEmployeeRow>[] = [
  { type: ColumnType.Text,   field: 'first_name', headerName: 'Name',   width: '200px', sortable: true },
  { type: ColumnType.Text,   field: 'email',      headerName: 'Email',  width: '250px' },
  { type: ColumnType.Action, headerName: '',       width: '60px',
    actions: [
      { label: 'Edit',          action: 'open-preview' },
      { label: 'Toggle active', action: 'toggle-active',
        isDisabled: (row, { features }) => !features.has('toggle_active') },
    ]},
];
```

---

## Route configuration

Extend `ISwitchItem` to carry `sideMenu` for Scaffold active path sync:

```ts
// config/routes.tsx
interface IRouteItem extends ISwitchItem {
  sideMenu?: string;  // dot-path: "root.data.employee.active"
}

export const routes: IRouteItem[] = [
  { path: '/',                     sideMenu: 'root.data.employee.active', redirect: '/employee_active' },
  { path: '/employee_active',      sideMenu: 'root.data.employee.active', element: () => <EmployeePage /> },
  { path: '/employee/:id/employee',sideMenu: 'root.data.employee.active', element: () => <EmployeePage /> },
  { path: '/kanban',               sideMenu: 'root.data.kanban',          element: () => <KanbanPage /> },
  { path: '/settings',             sideMenu: 'root.application.settings', element: () => <SettingsPage /> },
];

export const baseRoutes: IRouteItem[] = [
  { path: '/login_page', element: () => <LoginPage /> },
  { path: '/error_page', element: () => <ErrorPage /> },
];
```

In `App.tsx`, read the current route to sync Scaffold's `activeOptionPath`:
```ts
const currentRoute = useRouteItem<IRouteItem>([...routes, ...baseRoutes], ioc.routerService);
const activeOptionPath = currentRoute?.sideMenu ?? '';
```

---

## App shell (App.tsx)

```tsx
export const App = () => {
  const currentRoute = useRouteItem<IRouteItem>([...routes, ...baseRoutes], ioc.routerService);

  return (
    <Scaffold3
      appName={CC_APP_NAME}
      options={sidemenu}
      activeOptionPath={currentRoute?.sideMenu ?? ''}
      payload={payload}           // features, user role — flows into isVisible guards
      onOptionClick={handleOptionClick}
      onTabChange={handleTabClick}
      onAction={(action) => {
        if (action === 'logout-action') ioc.pocketbaseService.logout();
      }}
    >
      <Switch
        history={ioc.routerService}
        items={[...routes, ...baseRoutes]}
        Loader={() => <LinearProgress />}
        NotFound={() => <ErrorPage />}
      />
    </Scaffold3>
  );
};
```

---

## Infrastructure services

### LayoutService — global loader state

```ts
export class LayoutService {
  public readonly appbarLoaderSubject = new Subject<boolean>();

  private _appbarCount = 0;

  get hasAppbarLoader() { return this._appbarCount > 0; }

  setAppbarLoader(loading: boolean) {
    this._appbarCount += loading ? 1 : -1;
    this.appbarLoaderSubject.next(this.hasAppbarLoader);
  }
}
```

Usage in every async action: `onLoadStart: () => ioc.layoutService.setAppbarLoader(true)` and `onLoadEnd`.

### RouterService — history wrapper

```ts
import { createBrowserHistory, createMemoryHistory } from 'history';

export class RouterService implements BrowserHistory {
  private _history = CC_FORCE_MEMORY_HISTORY
    ? createMemoryHistory()
    : createBrowserHistory();

  get location() { return this._history.location; }
  push(to: To)    { this._history.push(to); }
  replace(to: To) { this._history.replace(to); }
  // ... rest of BrowserHistory interface
}
```

Pass `ioc.routerService` directly to `<Switch history={...}>` and `useRouteItem(routes, ioc.routerService)`.

---

## Modal pattern

Modals are hooks in `view/`. Each hook:
1. Uses `useActionModal` (form modal) or `useOutletModal` (multi-tab modal)
2. Loads data via `fetchState`
3. Calls `ioc.*ViewService.*` on submit
4. Uses `useModalManager` push/pop for stack management

```ts
// view/useEmployeeCreateModal/useEmployeeCreateModal.tsx
export const useEmployeeCreateModal = (payload: IPayload) => {
  const { push, pop } = useModalManager();

  const { render, pickData } = useActionModal<IEmployeeDto>({
    fields: employee_fields,
    payload,
    title: 'Create employee',
    onSubmit: async (data) => {
      await ioc.employeeViewService.create(data);
      pop();
    },
    onClose: pop,
  });

  return () => {
    push({ render, onClose: pop });
    return pickData();
  };
};
```

---

## HOC patterns

**Lazy loading** — wrap any heavy page component:
```ts
// components/hoc/heavy.tsx
const heavy = <P extends {}>(factory: () => Promise<{ default: React.ComponentType<P> }>) => {
  const Component = React.lazy(factory);
  return (props: P) => (
    <Suspense fallback={<LinearProgress />}>
      <Component {...props} />
    </Suspense>
  );
};

// In routes.tsx:
const KanbanPage = heavy(() => import('../../pages/view/KanbanPage/KanbanPage'));
```

**Auth guard** — redirect if not authenticated:
```ts
// components/hoc/redirect.tsx
const redirect = (Component: React.ComponentType) => (props: any) => {
  if (!ioc.pocketbaseService.isAuthorized) {
    ioc.routerService.replace('/login_page');
    return null;
  }
  return <Component {...props} />;
};

// In routes.tsx:
{ path: '/employee_active', element: () => redirect(EmployeePage) }
```

---

## Entry point providers stack

```tsx
// index.tsx — order matters: outermost = most global
<ErrorBoundary history={ioc.routerService} onError={ioc.errorService.handleGlobalError}>
  <ThemeProvider theme={THEME_DARK}>
    <LocalizationProvider dateAdapter={AdapterDayjs} adapterLocale="en-gb">
      <OneSlotFactory Date={DateSlot} Time={TimeSlot}>   {/* custom field renderers */}
        <ListSlotFactory {...ListRules.denceFilterRule}>  {/* list row density */}
          <ModalProvider>
            <ModalManagerProvider>
              <LayoutProvider>
                <AlertProvider>
                  <App />
                </AlertProvider>
              </LayoutProvider>
            </ModalManagerProvider>
          </ModalProvider>
        </ListSlotFactory>
      </OneSlotFactory>
    </LocalizationProvider>
  </ThemeProvider>
</ErrorBoundary>
```

Set global `<One />` behavior once at the top:
```ts
OneConfig.setValue({
  WITH_DIRTY_CLICK_LISTENER: true,
  WITH_MOBILE_READONLY_FALLBACK: true,
  WITH_SYNC_COMPUTE: true,
  CUSTOM_FIELD_DEBOUNCE: 800,
  FIELD_BLUR_DEBOUNCE: 50,
});
```

---

## Adding a new entity — checklist

1. **Types** — `lib/services/db/NewEntityDbService.ts`: define `INewEntityDto` and `INewEntityModel`
2. **Symbol** — add `newEntityDbService` and `newEntityViewService` to `lib/types.ts`
3. **DB service** — create `NewEntityDbService` with CRUD + `paginate` + Subjects + `prefetch` WebSocket subscription
4. **View service** — create `NewEntityViewService`, delegate to DB service, add logging/history
5. **Register** — add both `provide()` calls to `lib/config.ts`
6. **Inject** — add both `inject()` calls to `lib/ioc.ts`
7. **Assets** — create `assets/new_entity_fields/` and `assets/new_entity_columns/`
8. **Route** — add entry to `config/routes.tsx` with `sideMenu` path
9. **Menu** — add option to `config/sidemenu.ts`
10. **Page** — create `pages/view/NewEntityPage/` with List + One subpages
11. **Modals** — create `view/useNewEntityCreateModal/` and `view/useNewEntityPreviewModal/`
