# Scaffold2, KanbanView, WizardView — Component Reference

---

## Scaffold2 (App Shell)

Renders a Material Design app shell: collapsible sidebar, top app bar, content area. Generic over `Payload`.

`Scaffold3` is a drop-in visual refresh of Scaffold2 — identical API, just import differently.

### Core props

| Prop | Type | Notes |
|---|---|---|
| `options` | `IScaffold2Group[]` | **Required.** Navigation tree. |
| `activeOptionPath` | `string` | **Required.** Dot-separated path of active item, e.g. `"main.users"`. |
| `children` | `ReactNode` | **Required.** Page content area. |
| `appName` | `string` | App name in top bar. |
| `payload` | `T` | Forwarded to all `isVisible`/`isDisabled` callbacks. |
| `actions` | `IScaffold2Action[]` | Toolbar action buttons (top right). |
| `activeTabPath` | `string` | Active tab within current option. |
| `loading` | `boolean \| number` | Shows loading indicator in sidebar. |
| `noSearch` | `boolean` | Hide sidebar search. |
| `noAppName` | `boolean` | Hide app name. |
| `fixedHeader` | `boolean` | Fixed top bar while content scrolls. |
| `dense` | `boolean` | Compact sidebar item padding. |
| `deps` | `any[]` | Re-evaluate all async guards when changed. |
| `onOptionClick` | `(path, id) => void \| boolean` | Return `false` to prevent highlight update. |
| `onTabChange` | `(path, tab, id) => void` | Tab switch in top bar. |
| `onAction` | `(name) => void` | Toolbar action clicked. |
| `onInit` | `() => void \| Promise<void>` | Called once on mount. |
| `fallback` | `(e: Error) => void` | Async guard error handler. |

### Slot components

| Prop | Position |
|---|---|
| `AfterAppName` | After app name in sidebar header |
| `BeforeSearch` / `AfterSearch` | Around sidebar search |
| `BeforeMenuContent` / `AfterMenuContent` | Around menu item list |
| `BeforeContent` / `AfterContent` | Around main content area |
| `Copyright` | Footer of sidebar |

### IScaffold2Group shape

```ts
{
  id: string,           // required — used in path strings
  label: string,
  icon: React.ComponentType,
  noHeader: boolean,    // suppress group heading
  isVisible: async (payload?) => boolean,
  isDisabled: async (payload?) => boolean,
  children: IScaffold2Option[],  // required
}
```

### IScaffold2Option shape

```ts
{
  id: string,           // contributes to path e.g. "main" + "users" → "main.users"
  label: ReactNode,
  icon: React.ComponentType,
  isVisible: (payload) => boolean | Promise<boolean>,
  isDisabled: (payload) => boolean | Promise<boolean>,
  tabs: IScaffold2Tab[],       // sub-tabs in top bar when this option is active
  options: IScaffold2Option[], // nested sub-items in sidebar
}
```

### Minimal example

```tsx
import { Scaffold2 } from 'react-declarative';
import PeopleIcon from '@mui/icons-material/People';

const options = [{
  id: 'main', label: 'Main',
  children: [
    { id: 'users', label: 'Users', icon: PeopleIcon,
      isVisible: async (payload) => payload.isAdmin },
    { id: 'settings', label: 'Settings' },
  ],
}];

export const App = () => {
  const [path, setPath] = React.useState('main.users');
  return (
    <Scaffold2
      appName="My App"
      options={options}
      activeOptionPath={path}
      payload={{ isAdmin: true }}
      onOptionClick={(path, id) => { setPath(path); router.push(`/${id}`); }}
    >
      <Outlet />
    </Scaffold2>
  );
};
```

---

## KanbanView

Drag-and-drop kanban board. Generic: `KanbanView<Data, Payload, ColumnType>`.

### Core props

| Prop | Type | Notes |
|---|---|---|
| `columns` | `IBoardColumn[]` | **Required.** Lane definitions. |
| `items` | `IBoardItem[]` | **Required.** Flat list of all cards. |
| `onChangeColumn` | `(id, column, data, payload) => void \| Promise<void>` | Card moved to new lane. |
| `payload` | `Payload` | Forwarded to all row callbacks. |
| `reloadSubject` | `TSubject<void>` | Emit to clear cache and re-render all cards. |
| `onDataRequest` | `(initial: boolean) => void` | Called on mount and on visibility change. |
| `cardLabel` | `ReactNode \| ((id, data, payload) => ReactNode \| Promise<ReactNode>)` | Default card header. |
| `disabled` | `boolean` | Disable drag interactions. |
| `sx` | `SxProps` | Set `height` to enable column scrolling. |
| `withGoBack` | `boolean` | Allow dragging to earlier columns. |
| `withUpdateOrder` | `boolean` | Sort cards by `updatedAt` descending. |
| `bufferSize` | `number` | Virtual list buffer (default 15). |
| `rowTtl` | `number` | Row value cache TTL ms (default 500). |
| `filterFn` | `(item) => boolean` | Client-side card filter. |

### IBoardColumn shape

```ts
{
  column: ColumnType,   // unique lane ID
  label: string,        // header label
  color: string,        // CSS color for header accent
  rows: IBoardRow[],    // card content definition
}
```

### IBoardItem shape

```ts
{
  id: string,
  column: ColumnType,   // current lane
  data: Data,
  label?: ReactNode | ((id, data, payload) => ReactNode | Promise<ReactNode>),
  updatedAt?: string,   // ISO 8601, used with withUpdateOrder
}
```

### IBoardRow shape

```ts
{
  label: ReactNode,
  value: (id, data, payload) => ReactNode | Promise<ReactNode>,
  visible?: boolean | ((id, data, payload) => boolean | Promise<boolean>),
  click?: (id, data, payload) => void | Promise<void>,
}
```

### Slot components

| Prop | Position |
|---|---|
| `AfterCardContent` | Below rows inside each card |
| `BeforeColumnTitle` | Left of column header |
| `AfterColumnTitle` | Right of column header (e.g. card count badge) |

### Real-time support

```tsx
import { KanbanView, useSubject } from 'react-declarative';

export const RealtimeBoard = () => {
  const reloadSubject = useSubject<void>();
  useEffect(() => {
    const ws = new WebSocket('wss://api.example.com/stream');
    ws.onmessage = () => reloadSubject.next();
    return () => ws.close();
  }, []);
  return <KanbanView columns={columns} items={items} reloadSubject={reloadSubject} onChangeColumn={handleMove} />;
};
```

### Auto-scroll on drag

```tsx
const boardRef = useRef<HTMLDivElement>();
useEffect(KanbanView.enableScrollOnDrag(boardRef, { threshold: 200, speed: 15 }), []);
return <KanbanView ref={boardRef} ... />;
```

---

## WizardView

Multi-step wizard with MUI Stepper + nested router. Generic: `WizardView<Data, Payload>`.

### Core props

| Prop | Type | Notes |
|---|---|---|
| `steps` | `IWizardStep[]` | **Required.** Stepper header items. |
| `routes` | `IWizardOutlet[]` | **Required.** Path-to-component mapping. |
| `pathname` | `string` | Current path (defaults to `"/"`). |
| `history` | `History` | Pass your router's history to integrate. |
| `payload` | `Payload` | Forwarded to all outlet props and step `isVisible`. |
| `onNavigate` | `(update: Update) => void` | Sync internal path with browser URL. |
| `withScroll` | `boolean` | Scrollable content area. |
| `outlinePaper` | `boolean` | Outlined paper instead of elevation. |
| `transparentPaper` | `boolean` | Transparent paper background. |
| `fullScreen` | `boolean` | Full-viewport rendering. |

### IWizardStep shape

```ts
{
  id: string,
  label: string,
  icon: React.ComponentType,
  isMatch: (routeId: string) => boolean,   // custom matching for multi-route steps
  isVisible: (payload) => boolean,          // hide from stepper header
  passthrough: boolean,                     // hide chrome entirely for loading screens
}
```

### IWizardOutlet shape

```ts
{
  id: string,                               // matched against step ids
  isActive: (pathname: string) => boolean,  // which outlet renders for current path
  element: (props: IWizardOutletProps) => ReactElement,
}
```

### IWizardOutletProps

```ts
{
  history,          // navigate between steps
  payload,
  onSubmit,
  data,
  size,             // { width, height } of content area
  loading,          // global loading state
  setLoading,       // show/hide progress bar
  progress,         // 0-100 for determinate progress
  setProgress,
}
```

### WizardNavigation component

```tsx
<WizardNavigation
  hasPrev                            // enable Prev button
  hasNext                            // enable Next button
  labelPrev="Back"                   // default "Prev"
  labelNext="Continue"               // default "Next"
  onPrev={() => history.replace('/step-1')}
  onNext={async () => {              // async: buttons disable while pending
    await validateStep();
    history.replace('/step-3');
  }}
  AfterPrev={<CustomWidget />}
  BeforeNext={<CustomWidget />}
/>
```

Place `WizardNavigation` in the `Navigation` slot of `WizardContainer`:

```tsx
export const Step1 = ({ history }: IWizardOutletProps) => (
  <WizardContainer
    Navigation={
      <WizardNavigation hasNext onNext={() => history.replace('/step-2')} />
    }
  >
    <YourStepContent />
  </WizardContainer>
);
```

### Outlet context timing: setLoading / setProgress

```tsx
const Step2 = ({ history, setLoading, setProgress }: IWizardOutletProps) => {
  const handleNext = async () => {
    setLoading(true);
    for (let i = 0; i <= 100; i += 20) {
      setProgress(i);
      await sleep(200);
    }
    setLoading(false);
    history.replace('/step-3');
  };
  return <WizardContainer Navigation={<WizardNavigation hasNext onNext={handleNext} />}>...</WizardContainer>;
};
```
