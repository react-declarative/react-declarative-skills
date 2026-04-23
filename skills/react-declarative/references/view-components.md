# View Components Reference

Multi-step and tabbed view components imported from `react-declarative`.

---

## `<OutletView />` — multi-route view container

The primary page-level layout component. Renders the matching route from an `IOutlet[]` array. Manages shared `initialData`, `payload`, `onChange`, `onSubmit` across all child outlet components. Wraps `<FetchView />` externally to load data before mounting.

### Props

| Prop | Type | Description |
|---|---|---|
| `routes` | `IOutlet[]` | **Required.** Route definitions |
| `history` | `History` | **Required.** Browser/router history |
| `payload` | `any \| (() => any)` | Forwarded to all outlet components |
| `initialData` | `any \| (() => any)` | Initial form data shared across outlets |
| `params` | `Record<string, string>` | URL params (e.g. `{ id }`) available in outlets |
| `changed` | `() => boolean` | Returns `true` if unsaved draft exists |
| `onChange` | `(data, initial) => void` | Form change handler (e.g. push to draft service) |
| `onSubmit` | `async (data) => boolean` | Save handler — return `false` to keep form dirty |
| `onLeave` | `() => void` | Called when user navigates away |
| `waitForChangesDelay` | `number` | Debounce ms before `onChange` fires |
| `changeSubject` | `Subject<void>` | Emit to trigger external re-render |
| `onLoadStart` | `() => void` | Loading indicator start |
| `onLoadEnd` | `() => void` | Loading indicator end |

### `IOutlet` shape

```ts
{
  id: string,
  element: React.ComponentType<IOutletProps>,
  isActive: (pathname: string) => boolean,
  isAvailable?: () => boolean,   // false = skip this route (list pages that share OutletView)
}
```

### Outlet component props (`IOutletProps`)

```ts
{
  history,       // navigate between outlets
  payload,       // from OutletView.payload
  formState,     // { data, initialData, changed }
  data,          // current form data
  onChange,      // (data, initial) => void
  onSubmit,      // () => void — trigger save
  params,        // URL params
  loading,
  setLoading,
}
```

### Usage pattern (entity edit pages)

```tsx
// ApartmentPages.tsx
const routes: IOutlet[] = [
  // List page — marked isAvailable: false so it shares same OutletView without data
  {
    id: "list",
    element: ListPage,
    isActive: (p) => hasRouteMatch(["/apartment", "/apartment_rent"], p),
    isAvailable: () => false,
  },
  // Edit routes — share the same initialData / payload
  {
    id: "object",
    element: ObjectEdit,
    isActive: (p) => hasRouteMatch(["/apartment_edit/:id", "/apartment_edit/:id/object"], p),
  },
  {
    id: "documents",
    element: DocumentsEdit,
    isActive: (p) => hasRouteMatch(["/apartment_edit/:id/documents"], p),
  },
  {
    id: "owner_contact",
    element: OwnerContactEdit,
    isActive: (p) => hasRouteMatch(["/apartment_edit/:id/owner_contact"], p),
  },
  // View routes (readonly)
  {
    id: "object",
    element: ObjectView,
    isActive: (p) => hasRouteMatch(["/apartment_view/:id", "/apartment_view/:id/object"], p),
  },
];

export const ApartmentPages = ({ id = "create" }) => {
  const changeSubject = useSubject();

  return (
    <FetchView
      state={async () => [
        await ioc.apartmentViewService.tryRead(id),
        await ioc.permissionService.getPermissions({ apartmentId: id }),
        await ioc.featureService.getAccessLevel(),
      ]}
      reloadSubject={ioc.layoutService.reloadOutletSubject}
      fallback={ioc.errorService.handleGlobalError}
    >
      {async ([object, permissions, accessLevel]) => (
        <OutletView
          history={ioc.routerService}
          changeSubject={changeSubject}
          waitForChangesDelay={CC_WAIT_FOR_CHANGES_DELAY}
          routes={routes}
          params={{ id }}
          changed={() => ioc.draftService.apartmentId === id}
          initialData={() =>
            ioc.draftService.apartmentId === id
              ? ioc.draftService.readLastApartment()
              : { object, documents: object?.apartment_files || [] }
          }
          payload={() => ({
            permissions,
            accessLevel,
            id,
            userId: ioc.appwriteService.currentUser.$id,
          })}
          onChange={(data, initial) =>
            ioc.draftService.pushApartment({ id, data }, initial)
          }
          onLeave={ioc.draftService.dropApartment}
          onSubmit={handleSubmit}
          onLoadStart={() => ioc.layoutService.setAppbarLoader(true)}
          onLoadEnd={() => ioc.layoutService.setAppbarLoader(false)}
        />
      )}
    </FetchView>
  );
};
```

**Outlet component** (child of OutletView):
```tsx
export const ObjectEdit = ({ data, onChange, onSubmit, payload, formState }: IOutletProps) => (
  <>
    <Breadcrumbs2 payload={formState} items={breadcrumbOptions} onAction={handleAction} />
    <One
      fields={object_fields}
      handler={() => data}
      payload={payload}
      onChange={onChange}
    />
  </>
);
```

---

## `<TabsView />` — tabbed view with internal routing

A lighter alternative to `OutletView` for simple two-tab layouts (e.g. List / Calendar). Uses internal memory history to switch between tabs.

### Props

| Prop | Type | Description |
|---|---|---|
| `tabs` | `ITabsStep[]` | **Required.** Tab header definitions |
| `routes` | `ITabsOutlet[]` | **Required.** Tab content components |
| `pathname` | `string` | Initial active route |
| `payload` | `any` | Forwarded to all outlet components |
| `onTabChange` | `(id: string, history: History) => void` | Called when tab is clicked — navigate with `history.replace` |

### `ITabsStep` shape

```ts
{ id: string, label: string }
```

### `ITabsOutlet` shape

```ts
{
  id: string,
  element: React.ComponentType<ITabsOutletProps>,
  isActive: (pathname: string) => boolean,
}
```

### `ITabsOutletProps`

```ts
{
  payload,
  history,
  loading,
  setLoading,
}
```

### Usage

```tsx
// ReviewPages/BidPage/BidPage.tsx
import { ITabsOutlet, ITabsStep, TabsView, parseRouteUrl } from "react-declarative";

const tabs: ITabsStep[] = [
  { id: "list",     label: "Список"   },
  { id: "calendar", label: "Календарь" },
];

const routes: ITabsOutlet[] = [
  {
    id: "list",
    element: ReviewList,
    isActive: (p) => !!parseRouteUrl("/list", p),
  },
  {
    id: "calendar",
    element: ReviewCalendar,
    isActive: (p) => !!parseRouteUrl("/calendar", p),
  },
];

export const BidPage = ({ payload }: IOutletProps) => (
  <Container>
    <TabsView
      pathname="/list"
      payload={payload}
      tabs={tabs}
      routes={routes}
      onTabChange={(id, history) => {
        history.replace(`/${id}`);
      }}
    />
  </Container>
);
```

**Tab outlet component:**
```tsx
export const ReviewList = ({ payload, setLoading }: ITabsOutletProps) => {
  // use payload for permissions, setLoading for appbar indicator
};
```

---

## `<WizardView />` — multi-step wizard

Linear step-by-step wizard with a MUI Stepper header. Each step is a route with its own component. Navigation between steps is done via `history.replace` inside step components.

### Props

| Prop | Type | Description |
|---|---|---|
| `steps` | `IWizardStep[]` | **Required.** Stepper header items |
| `routes` | `IWizardOutlet[]` | **Required.** Step components |
| `pathname` | `string` | Initial step route |
| `payload` | `any` | Forwarded to all step components |
| `initialData` | `any` | Shared data object passed to all steps and mutated via `onChange` |
| `history` | `History` | External history (optional — uses internal if omitted) |
| `onNavigate` | `(update) => void` | Sync internal path with browser URL |
| `withScroll` | `boolean` | Scrollable content area |
| `outlinePaper` | `boolean` | Outlined paper style |
| `transparentPaper` | `boolean` | Transparent background |
| `fullScreen` | `boolean` | Full-viewport |

### `IWizardStep` shape

```ts
{
  id: string,
  label: string,
  icon?: React.ComponentType,
  isVisible?: (payload) => boolean,
  isMatch?: (routeId: string) => boolean,   // custom matching for multi-route steps
  passthrough?: boolean,                     // hide chrome (loading screens)
}
```

### `IWizardOutlet` shape

```ts
{
  id: string,
  element: React.ComponentType<IWizardOutletProps>,
  isActive: (pathname: string) => boolean,
}
```

### `IWizardOutletProps`

```ts
{
  history,       // navigate to next/prev step
  payload,
  data,          // shared wizard data
  onChange,      // (data, initial) => void
  onSubmit,
  loading,
  setLoading,
  progress,      // 0–100 for determinate progress bar
  setProgress,
  size,          // { width, height } of content area
}
```

### Usage

```tsx
// ImportPages/MainPage.tsx
import { IWizardOutlet, IWizardStep, WizardView, parseRouteUrl } from "react-declarative";

const steps: IWizardStep[] = [
  { id: "template", label: "Выбор шаблона" },
  { id: "price",    label: "Стоимость"     },
  { id: "preview",  label: "Предпросмотр"  },
  { id: "import",   label: "Импорт"        },
  { id: "report",   label: "Отчет"         },
];

const routes: IWizardOutlet[] = [
  {
    id: "template",
    element: ChooseTemplateView,
    isActive: (p) => !!parseRouteUrl("/choose-template", p),
  },
  {
    id: "price",
    element: ChoosePriceView,
    isActive: (p) => !!parseRouteUrl("/choose-price", p),
  },
  {
    id: "preview",
    element: PreviewImportView,
    isActive: (p) => !!parseRouteUrl("/preview-import", p),
  },
  {
    id: "import",
    element: ImportView,
    isActive: (p) => !!parseRouteUrl("/import", p),
  },
  {
    id: "report",
    element: ReportView,
    isActive: (p) => !!parseRouteUrl("/report", p),
  },
];

export const MainPage = ({ payload, formState }: IOutletProps) => (
  <Container>
    <Breadcrumbs2 payload={formState} items={options} />
    <WizardView
      pathname="/choose-template"
      initialData={{ template: {}, price: {}, import: { errors: [] } }}
      payload={payload}
      steps={steps}
      routes={routes}
    />
  </Container>
);
```

### Step component pattern — `WizardContainer` + `WizardNavigation`

Every step component should use `WizardContainer` to place navigation buttons consistently, and `WizardNavigation` for Prev/Next controls:

```tsx
// view/ChooseTemplateView.tsx
import { IWizardOutletProps, WizardContainer, WizardNavigation, One } from "react-declarative";

export const ChooseTemplateView = ({ history, data, onChange }: IWizardOutletProps) => (
  <WizardContainer
    Navigation={
      <WizardNavigation
        hasNext={!!data.id}                         // enable Next only when a template is selected
        onNext={() => history.replace("/choose-price")}
      />
    }
  >
    <One
      fields={fields}
      sx={{ mb: 3 }}
      handler={() => data}
      onChange={onChange}
    />
  </WizardContainer>
);
```

**Step with async navigation and progress:**
```tsx
export const ImportView = ({ history, data, setLoading, setProgress }: IWizardOutletProps) => {
  const handleImport = async () => {
    setLoading(true);
    for (let i = 0; i < data.rows.length; i++) {
      await importRow(data.rows[i]);
      setProgress(Math.round((i + 1) / data.rows.length * 100));
    }
    setLoading(false);
    history.replace("/report");
  };

  return (
    <WizardContainer
      Navigation={
        <WizardNavigation
          hasPrev
          hasNext
          labelNext="Начать импорт"
          onPrev={() => history.replace("/preview-import")}
          onNext={handleImport}
        />
      }
    >
      <Typography>Готово к импорту: {data.rows.length} записей</Typography>
    </WizardContainer>
  );
};
```

### WizardNavigation props

| Prop | Type | Description |
|---|---|---|
| `hasPrev` | `boolean` | Show / enable Prev button |
| `hasNext` | `boolean` | Show / enable Next button |
| `labelPrev` | `string` | Override "Prev" label |
| `labelNext` | `string` | Override "Next" label |
| `onPrev` | `() => void \| Promise<void>` | Prev click handler |
| `onNext` | `() => void \| Promise<void>` | Next click handler — async disables buttons while pending |
| `AfterPrev` | `ReactNode` | Extra content after Prev button |
| `BeforeNext` | `ReactNode` | Extra content before Next button |

---

## `<CalendarView<T> />` — month calendar with items

Generic calendar component. Renders a month grid with event dots per day. The handler is called once per visible month range and returns a flat list of items with timestamps.

### Props

| Prop | Type | Description |
|---|---|---|
| `handler` | `async ({ fromStamp, toStamp, payload }) => { data: T, stamp: number, payload?: any }[]` | **Required.** Load items for the visible month range |
| `renderItem` | `({ data: T }) => ReactNode` | **Required.** Render one item in the day detail list |
| `date` | `Dayjs` | Controlled selected date (pass `dayjs()` for today) |
| `payload` | `any` | Forwarded to `handler` and all callbacks; changing it triggers a reload |
| `sx` | `SxProps` | MUI sx on the root element |
| `reloadSubject` | `TSubject<void>` | Emit to force a reload — pass `useChangeSubject(filterData) as TSubject<void>` to auto-reload on filter change |
| `onItemClick` | `({ data: T }) => void` | Called when the user clicks a rendered item |
| `BeforeDayHeader` | `React.ComponentType<{ items: { stamp: number }[] }>` | Slot rendered before each day header cell — used for navigation to a day-detail page |
| `dotSide` | `number` | Controls dot indicator size/visibility. `0` = hidden, `6` = visible (default) |
| `onLoadStart` | `() => void` | Loading start callback |
| `onLoadEnd` | `() => void` | Loading end callback |

### Handler signature

```ts
handler: async ({
  fromStamp: number,   // start of visible range (unix ms)
  toStamp:   number,   // end of visible range (unix ms)
  payload,             // from CalendarView.payload
}) => Array<{
  data:    T,          // domain object
  stamp:   number,     // unix ms — determines which day cell the item appears in
  payload?: any,
}>
```

### Usage

```tsx
// ReviewPages/BidPage/ReviewCalendar.tsx
import { CalendarView, useChangeSubject } from "react-declarative";
import dayjs from "dayjs";

export const ReviewCalendar = ({ payload }: ITabsOutletProps) => {
  const reloadSubject = useChangeSubject(payload) as TSubject<void>;

  return (
    <CalendarView<IReviewDto>
      date={dayjs()}
      payload={payload}
      reloadSubject={reloadSubject}
      handler={async ({ fromStamp, toStamp, payload }) => {
        const reviews = await ioc.reviewViewService.findByRange({
          fromStamp,
          toStamp,
          ...payload,
        });
        return reviews.map((data) => ({
          data,
          stamp: data.review_date,
        }));
      }}
      renderItem={({ data }) => (
        <ReviewCard key={data.$id} data={data} payload={payload} />
      )}
      onItemClick={({ data }) => {
        ioc.routerService.push(`/review_edit/${data.$id}`);
      }}
    />
  );
};
```

### `BeforeDayHeader` slot — navigate to day detail

Use `BeforeDayHeader` to add a click target on each day header that navigates to a day-scoped page. The slot receives the list of items falling on that day:

```tsx
<CalendarView<ISmsDto>
  date={dayjs()}
  payload={payload}
  handler={smsHandler}
  renderItem={({ data }) => <SmsCard data={data} />}
  BeforeDayHeader={({ items }) => {
    const stamp = items[0]?.stamp;
    if (!stamp) return null;
    return (
      <IconButton
        size="small"
        onClick={() =>
          ioc.routerService.push(
            `/sms_calendar/${dayjs(stamp).format("YYYY-MM-DD")}`
          )
        }
      >
        <ArrowForward fontSize="small" />
      </IconButton>
    );
  }}
  dotSide={6}
/>
```

### Reactive reload on filter change

To reload the calendar whenever external filter data changes, cast `useChangeSubject` to `TSubject<void>`:

```tsx
const reloadSubject = useChangeSubject(filterData) as TSubject<void>;

<CalendarView
  reloadSubject={reloadSubject}
  handler={...}
  ...
/>
```
