# UI Components Reference

Components imported from `react-declarative` used across the real-estate-crm project.

---

## `<VirtualView />`

Virtualized list container. Renders only visible rows — safe for hundreds of items. Supports infinite-scroll via `onDataRequest`.

### Props

| Prop | Type | Description |
|---|---|---|
| `sx` | `SxProps` | MUI sx — set `height` (required for virtualization to work) |
| `className` | `string` | CSS class on the root element |
| `minRowHeight` / `minHeight` | `number` | Minimum row height hint for virtualization calculations |
| `bufferSize` | `number` | Number of rows to render outside visible viewport |
| `loading` | `boolean` | Show loading indicator, disables new `onDataRequest` calls |
| `hasMore` | `boolean` | When `false`, `onDataRequest` is no longer called |
| `onDataRequest` | `(initial: boolean) => void` | Called on mount (`initial=true`) and when user scrolls near the end |

### Critical: children must be wrapped in `forwardRef`

`VirtualView` passes a `ref` to each child to measure its height. If the child does not forward refs, virtualization breaks silently. Always define card components with `forwardRef`:

```tsx
// ✅ Correct — child forwards ref
export const TaskCard = forwardRef<HTMLDivElement, ITaskCard>(
  ({ id, title, ...props }, ref) => (
    <div ref={ref}>
      {/* card content */}
    </div>
  )
);

// ❌ Wrong — ref is dropped, heights are not measured
export const TaskCard = ({ id, title }: ITaskCard) => (
  <div>{/* ... */}</div>
);
```

### Usage examples

**Basic list (fixed height):**
```tsx
<VirtualView
  sx={{ height: "40vh", width: "258px" }}
  bufferSize={15}
  minRowHeight={72}
>
  {items.map((item) => (
    <DealCard key={item.id} entity={item} payload={payload} />
  ))}
</VirtualView>
```

**Infinite scroll with external data loading:**
```tsx
<VirtualView
  sx={{ height: "80vh" }}
  minHeight={72}
  loading={loading}
  hasMore={hasMore}
  bufferSize={CC_LIST_BUFFER_SIZE}
  onDataRequest={(initial) => void execute(initial)}
>
  {data.map((item) => (
    <CallCard key={item.$id} data={item} sx={{ mb: 1 }} />
  ))}
</VirtualView>
```

**Inside a horizontal `<ScrollView />`:**
```tsx
<ScrollView withScrollbar hideOverflowY>
  {COLUMNS.map((col) => (
    <Box key={col}>
      <VirtualView
        className={classes.list}
        minRowHeight={CARD_MIN_HEIGHT}
        bufferSize={CC_BOARD_BUFFERSIZE}
      >
        {itemMap.get(col)?.map((doc) => (
          <TaskCard key={doc.id} payload={payload} {...doc} />
        ))}
      </VirtualView>
    </Box>
  ))}
</ScrollView>
```

---

## `<ScrollView />`

Scrollable container with optional custom scrollbar. Use instead of `overflow: auto` on plain divs to get consistent cross-browser scrollbar styling.

### Props

| Prop | Type | Description |
|---|---|---|
| `sx` | `SxProps` | MUI sx styling (typically `height: "100%"`) |
| `className` | `string` | CSS class on root element |
| `withScrollbar` | `boolean` | Show a styled custom scrollbar |
| `hideOverflowY` | `boolean` | Hide vertical overflow (use for horizontal-scroll boards) |

### Usage examples

**Wrap a `<One />` form to make it scrollable in a modal:**
```tsx
<ScrollView withScrollbar sx={{ height: "100%" }}>
  <Container>
    <One fields={fields} handler={handler} payload={payload} onChange={onChange} />
  </Container>
</ScrollView>
```

**Horizontal kanban board — scroll horizontally, hide vertical overflow:**
```tsx
<ScrollView withScrollbar hideOverflowY className={classes.content}>
  {TASK_STATUS_LIST.map((status) => (
    <Box key={status}>
      <VirtualView minRowHeight={CARD_MIN_HEIGHT} bufferSize={10}>
        {itemMap.get(status)?.map((doc) => (
          <TaskCard key={doc.id} {...doc} />
        ))}
      </VirtualView>
    </Box>
  ))}
</ScrollView>
```

**Wrap mixed content (InfoLabel in a list header):**
```tsx
<ScrollView sx={{ height: "100%" }} hideOverflowY>
  <Box sx={{ display: "flex", alignItems: "center", gap: 1 }}>
    <Typography variant="subtitle2">Заявки:</Typography>
    <Async payload={null} Loader={Loader}>
      {async () => { /* fetch counts */ }}
    </Async>
  </Box>
</ScrollView>
```

---

## `<Async />`

Renders an async function as JSX. Shows `Loader` while the promise is pending, then mounts the resolved result. Re-runs when `payload` or `deps` change.

### Props

| Prop | Type | Description |
|---|---|---|
| `Loader` | `React.ComponentType` | Shown while the async child is running |
| `payload` | `any` | Passed as the first argument to the async child function; changing it re-runs the function |
| `deps` | `any[]` | Additional deps (like `useEffect`); re-run when any dep changes |

### Children

Children must be a single async function: `async (payload?) => ReactNode`.

### Usage examples

**No payload — fire-and-forget fetch:**
```tsx
<Async Loader={Loader}>
  {async () => {
    const count = await ioc.contactRequestService.getApartmentCount();
    return <Typography>{count} объектов</Typography>;
  }}
</Async>
```

**With payload — re-runs when `image` prop changes:**
```tsx
<Async Loader={Loader} payload={image}>
  {async (image) => {
    if (!image) return <PlaceholderIcon />;
    const src = await ioc.appwriteService.getImageURL(image);
    return (
      <img
        loading="lazy"
        src={src}
        alt={image}
        style={{ objectFit: "contain", height: "100%", width: "100%" }}
      />
    );
  }}
</Async>
```

**With payload + extra deps — re-runs when `period` changes OR `target`/`userId` change:**
```tsx
<Async payload={period} deps={[target, userId]} Loader={Loader}>
  {async (period) => {
    const stats = await ioc.dashboardService.getStats(period, target, userId);
    return <StatChart data={stats} />;
  }}
</Async>
```

**Inline Loader component:**
```tsx
<Async
  Loader={() => (
    <Typography p={1} variant="caption">Поиск дубликатов...</Typography>
  )}
  payload={telefon}
>
  {async (telefon) => {
    const { rows } = await ioc.contactRequestService.findDuplicates(telefon);
    if (!rows.length) {
      return <Typography p={1} variant="caption">Дубликаты не найдены</Typography>;
    }
    return (
      <div>
        {rows.map(({ fio, telefon: t }) => (
          <ListItemButton key={t} onClick={() => onChange({ telefon: t, fio })}>
            <ListItemText primary={t} secondary={fio} />
          </ListItemButton>
        ))}
      </div>
    );
  }}
</Async>
```

---

## `<Breadcrumbs2 />`

Config-driven breadcrumb/toolbar bar. Supports links, action buttons, and custom components via `IBreadcrumbs2Option[]`.

### Props

| Prop | Type | Description |
|---|---|---|
| `items` | `IBreadcrumbs2Option[]` | **Required.** Breadcrumb item definitions |
| `payload` | `any` | Forwarded to all `isVisible`, `isDisabled`, `compute` callbacks |
| `actions` | `IAction[]` | Additional toolbar action buttons (right side) |
| `onAction` | `(action: string) => void` | Called when any item with an `action` is clicked |

### `IBreadcrumbs2Option` shape

```ts
{
  type: Breadcrumbs2Type,           // Link | Button | Component
  action?: string,                  // emitted to onAction on click
  label?: ReactNode,                // static label
  compute?: (state) => string,      // dynamic label from payload/data
  icon?: React.ComponentType,       // icon for Button type
  sx?: SxProps,                     // custom styling (e.g. red background)
  isVisible?: (state) => boolean,   // hide item
  isDisabled?: (state) => boolean,  // disable button item
  element?: React.ComponentType,    // for Breadcrumbs2Type.Component
}
```

### Usage examples

**Simple navigation breadcrumb:**
```tsx
const options: IBreadcrumbs2Option[] = [
  {
    type: Breadcrumbs2Type.Link,
    isVisible: () => !!ioc.routerService.previousPath,
    action: "list-action",
    label: <KeyboardArrowLeft sx={{ display: "block" }} />,
  },
  {
    type: Breadcrumbs2Type.Link,
    action: "list-action",
    label: "База объектов",
  },
  {
    type: Breadcrumbs2Type.Link,
    action: "list-action",
    label: "История действий",
  },
];

<Breadcrumbs2 payload={formState} items={options} onAction={handleAction} />
```

**With action buttons and permissions:**
```tsx
const options: IBreadcrumbs2Option[] = [
  {
    type: Breadcrumbs2Type.Link,
    action: "list-action",
    compute: ({ data }) => `Объект №${data.object.index}`,
  },
  {
    type: Breadcrumbs2Type.Button,
    action: "publish-action",
    label: "Опубликовать в канал",
    isDisabled: ({ payload: { permissions } }) =>
      !permissions.hasAdvertisingPublicationPermission,
  },
  {
    type: Breadcrumbs2Type.Button,
    action: "archive-action",
    label: "В архив",
    sx: { background: "#d32f2f", color: "white", minWidth: "125px" },
    isDisabled: ({ data }) => data.one.system_is_archive,
  },
];

<Breadcrumbs2
  payload={formState}
  items={options}
  actions={actions}
  onAction={handleAction}
/>
```

**Custom component item (e.g. a filter row):**
```tsx
const options: IBreadcrumbs2Option[] = [
  {
    type: Breadcrumbs2Type.Component,
    element: FilterRow,        // React component rendered inline
  },
  {
    type: Breadcrumbs2Type.Button,
    action: "add-action",
    icon: Add,
    label: "Добавить",
  },
];

<Breadcrumbs2 payload={formState} items={options} onAction={handleAction} />
```

**Handling actions:**
```tsx
const handleAction = (action: string) => {
  if (action === "list-action") {
    ioc.routerService.push("/apartments");
  }
  if (action === "publish-action") {
    openPublishDialog();
  }
};
```

---

## `<ActionMenu />`

Three-dot (kebab) menu button with a dropdown of actions. Config-driven via `IOption[]`. Supports async `isVisible`/`isDisabled` guards, loading state, and custom content slots.

### Props

| Prop | Type | Description |
|---|---|---|
| `options` | `IOption[]` | **Required.** Menu item definitions |
| `payload` | `any` | Forwarded to all `isVisible`/`isDisabled` callbacks |
| `onAction` | `(action: string) => void` | Called when a menu item is clicked |
| `deps` | `any[]` | Re-evaluate async guards when changed |
| `transparent` | `boolean` | Transparent button background |
| `disabled` | `boolean` | Disable entire menu |
| `sx` | `SxProps` | Styling on the root element |
| `className` | `string` | CSS class |
| `BeforeContent` | `React.ComponentType` | Component rendered above the menu items |
| `onToggle` | `(open: boolean) => void` | Called when menu opens/closes |
| `onLoadStart` | `() => void` | Called when async guard evaluation starts |
| `onLoadEnd` | `() => void` | Called when async guard evaluation ends |
| `fallback` | `(e: Error) => void` | Error handler for async guards |

### `IOption` shape

```ts
{
  action: string,
  label: string,
  icon?: React.ComponentType,
  isVisible?: (payload) => boolean | Promise<boolean>,
  isDisabled?: (payload) => boolean | Promise<boolean>,
}
```

### Usage examples

**Simple menu with static options:**
```tsx
const options = [
  { action: "edit-action",   label: "Редактировать", icon: Edit },
  { action: "delete-action", label: "Удалить",        icon: Delete },
];

<ActionMenu options={options} onAction={handleAction} />
```

**With payload and permission guards:**
```tsx
<ActionMenu
  options={options}
  payload={payload}
  onAction={(action) => actionEmitter.next(action)}
/>
```

**With custom before-content slot (e.g. user/period picker above the list):**
```tsx
<ActionMenu
  className={classes.options}
  BeforeContent={UserPicker}
  payload={{ period, target }}
  transparent
  options={[
    { action: "all-action",  label: "Все",  isVisible: () => target !== "all" },
    { action: "mine-action", label: "Мои",  isVisible: () => target !== "mine" },
    { action: "week",        label: "Неделя" },
    { action: "month",       label: "Месяц" },
  ]}
  onAction={(v) => {
    if (v === "all-action")  { setTarget("all");  return; }
    if (v === "mine-action") { setTarget("mine"); return; }
    setPeriod(v);
  }}
/>
```

**Positioned absolutely over an image (with toggle tracking):**
```tsx
<ActionMenu
  disabled={loading}
  sx={{
    position: "absolute",
    top: 10,
    left: 10,
    opacity: focus || toggle ? 1 : 0,
    transition: "opacity 150ms",
  }}
  payload={{ ...payload, readonly }}
  options={options}
  onAction={handleAction}
  onToggle={(open) => setToggle(open)}
/>
```
