# Hooks & Modal Patterns Reference

All symbols imported from `react-declarative`.

---

## `useQueryPagination` — persist List filters in the URL

Saves and restores `<ListTyped />` filter / pagination / sort state as URL query parameters. Navigating away and back restores the exact state the user left.

### Signature

```ts
const { listProps, getFilterData } = useQueryPagination(
  initialState,   // Partial<FilterData> — merged with URL params on mount
  options,
);
```

### Options

| Key | Type | Description |
|---|---|---|
| `limit` | `number` | Default page size |
| `prefix` | `string` | Namespace for URL params — use per-page unique key to avoid collisions |
| `noCleanupOnLeave` | `boolean` | Keep URL params when leaving the page |
| `noCleanupExtra` | `boolean` | Keep extra (non-standard) params |
| `fallback` | `(e: Error) => void` | Error handler |

### Returns

| Key | Description |
|---|---|
| `listProps` | Spread directly onto `<ListTyped />` — contains `filterData`, `page`, `limit`, `sort`, `chips`, `search`, and their onChange callbacks |
| `getFilterData()` | Returns the current filter object synchronously — use for "create from filter" actions |

### Usage

```tsx
// CommonPage.tsx
const { listProps, getFilterData } = useQueryPagination(
  {
    limit: CC_DEFAULT_LIMIT,
    ...ioc.routerService.locationState,   // restore state on back-navigation
  },
  {
    fallback:        ioc.errorService.handleGlobalError,
    noCleanupOnLeave: true,
    noCleanupExtra:   true,
    prefix:          "apartment_all",     // unique per page/tab
  },
);

return (
  <ListTyped
    {...listProps}           // ← spread here
    {...listActionProps}
    columns={columns}
    handler={paginator}
  />
);
```

**Using `getFilterData` for actions:**
```ts
const handleCreateFromFilter = async () => {
  const filterData = getFilterData();
  if (!Object.keys(filterData).some(Boolean)) {
    await pickAlert();  // warn user to set at least one filter
    return;
  }
  await ioc.bidViewService.createFromFilter(filterData);
};
```

---

## `useOne` — form dialog hook

Opens a `<One />` form in a modal dialog. Returns an observable that resolves to the submitted data (or `null` if cancelled).

### Signature

```ts
const pickData = useOne<TData>(options);
// Call to open:
const result = await pickData().toPromise();  // TData | null
```

### Options

| Key | Type | Description |
|---|---|---|
| `title` | `string` | Dialog title |
| `fields` | `TypedField[]` | Form schema |
| `handler` | `async (payload) => TData` | Load initial data |
| `payload` | `any` | Forwarded to field callbacks |
| `large` | `boolean` | Full-screen / large dialog |

### Usage

```tsx
const pickPublish = useOne({
  title: "Публикация в OLX",
  fields,
  handler: async () => ({
    owner_phone: user.rabochij_telefon,
    owner_fio: await ioc.userRequestService.getUserLabelShort(userId),
  }),
});

// Open and await result:
const data = await pickPublish().toPromise();
if (data) {
  await ioc.olxService.publish(data);
}
```

---

## `usePrompt` — text input dialog

Opens a simple single-field text dialog. Returns the entered string or `null` if cancelled.

### Signature

```ts
const pickText = usePrompt(options);
const result = await pickText().toPromise();  // string | null
```

### Options

| Key | Type | Description |
|---|---|---|
| `title` | `string` | Dialog title |
| `placeholder` | `string` | Input placeholder |

### Usage

```tsx
const pickComment = usePrompt({
  title: "Укажите причину архивации",
  placeholder: "Введите комментарий",
});

const comment = await pickComment().toPromise();
if (comment) {
  await ioc.apartmentViewService.archive(id, comment);
}
```

---

## `useAlert` — alert dialog

Opens a simple informational dialog. Awaitable — resolves when the user dismisses it.

### Signature

```ts
const showAlert = useAlert(options);
await showAlert();
```

### Options

| Key | Type | Description |
|---|---|---|
| `title` | `string` | Alert title |
| `description` | `string` | Alert body text |

### Usage

```tsx
const pickAlert = useAlert({
  title: "Невозможно создать заявку",
  description: "Укажите хотя бы один фильтр",
});

const handleCreate = async () => {
  if (!hasFilters) {
    await pickAlert();
    return;
  }
  await createBid();
};
```

---

## `useModalManager` — modal stack

Manages a push/pop stack of modals. Required provider: `<ModalManagerProvider />` at the app root.

### Signature

```ts
const { push, pop, clear, total } = useModalManager();
```

### Returns

| Key | Description |
|---|---|
| `push(config)` | Push a new modal onto the stack |
| `pop()` | Remove the top modal |
| `clear()` | Remove all modals |
| `total` | Number of modals currently open |

### `push` config

```ts
push({
  id: string,            // unique identifier
  render: ReactNode,     // what to render inside the modal
  onInit?: () => void,   // called synchronously when pushed
  onMount?: () => void,  // called after modal mounts
});
```

### Usage

```tsx
const { push, pop, clear, total } = useModalManager();

// In a modal header — show back button only when nested
const { total } = useModalManager();
<IconButton sx={{ display: total === 1 ? "none" : "flex" }} onClick={pop}>
  <ArrowBack />
</IconButton>

// Open a modal:
push({
  id: "apartment",
  render,
  onInit:  () => history.push(DEFAULT_PATH),
  onMount: () => pickData(id),
});

// Close from inside the modal:
pop();

// Close all (e.g. after navigation):
clear();
```

---

## `useActionModal` — One form in a modal

The standard edit/create dialog. Wraps a `<One />` form with a submit button and title bar. Returns `{ render, pickData }` — mount `render` once at component level, call `pickData()` to open.

### Signature

```ts
const { render, pickData } = useActionModal<TData>(options);
```

### Options

| Key | Type | Description |
|---|---|---|
| `title` | `string` | Dialog title |
| `fields` | `TypedField[]` | Form schema |
| `handler` | `async () => TData` | Load initial data |
| `payload` | `any` | Forwarded to field callbacks |
| `features` | `any` | Permissions object |
| `onSubmit` | `async (data, payload) => boolean` | Submit handler — return `false` to keep dialog open |
| `submitLabel` | `string` | Submit button label |
| `AfterTitle` | `React.ComponentType<{ onClose }>` | Close/action buttons in title bar |
| `outlinePaper` | `boolean` | Outlined paper style |
| `fullScreen` | `boolean` | Full-screen mode |
| `sizeRequest` | `() => { width, height }` | Dialog size |
| `withActionButton` | `boolean` | Show the submit button |
| `incomingTransform` | `(data) => data` | Transform data before showing in form |
| `outgoingTransform` | `(data) => data` | Transform data before passing to `onSubmit` |
| `onLoadStart` | `() => void` | Loading start callback |
| `onLoadEnd` | `() => void` | Loading end callback |

### Usage

**Standard create dialog:**
```tsx
const { render, pickData } = useActionModal({
  title: "Создать шаблон",
  fields,
  payload,
  submitLabel: "Создать",
  AfterTitle: ({ onClose }) => (
    <IconButton size="small" onClick={onClose}><Close /></IconButton>
  ),
  onSubmit: async (data) => {
    if (data) await ioc.smsTemplateService.create(data);
    return true;
  },
});

// Mount render once (in JSX):
return <>{render}</>;

// Open on button click:
pickData();
```

**Edit dialog with permissions and loading state:**
```tsx
const { render, pickData } = useActionModal<IAdvertisingDto>({
  title: "Редактировать",
  fields: advertising_fields,
  handler,
  payload,
  features: payload.permissions,
  outlinePaper: true,
  sizeRequest: CC_FULLSCREEN_SIZE_REQUEST,
  submitLabel: "Сохранить",
  onLoadStart: () => ioc.layoutService.setAppbarLoader(true),
  onLoadEnd:   () => ioc.layoutService.setAppbarLoader(false),
  onSubmit: async (data, payload) => {
    try {
      await ioc.advertisingViewService.update(id, data);
      return true;
    } catch (e) {
      ioc.alertService.notify(parseErrorMessage(e));
      return false;         // keeps dialog open
    }
  },
});
```

**With data transforms (e.g. number↔string conversion):**
```tsx
const { render, pickData } = useActionModal({
  title: "Редактировать",
  fields: olx_fields,
  handler: () => data,
  fullScreen: true,
  incomingTransform: (data) =>
    Object.fromEntries(
      Object.entries(data).map(([k, v]) => [
        k,
        OLX_NUMERIC_FIELD_SET.has(k) ? (v ? String(v) : "") : v,
      ]),
    ),
  outgoingTransform: (data) =>
    Object.fromEntries(
      Object.entries(data).map(([k, v]) => [
        k,
        OLX_NUMERIC_FIELD_SET.has(k) ? (v ? parseInt(v as string) : null) : v,
      ]),
    ),
  onSubmit: async (data) => {
    if (data) await ioc.olxService.update(data.$id, data);
    return true;
  },
});
```

---

## `useOutletModal` — multi-tab modal with routing

A modal that contains a `<OutletView />` with internal routing — several tabs/views inside one dialog. Used for complex read/edit modals (e.g. apartment details with Object / Owner / Documents tabs).

### Signature

```ts
const { render, pickData } = useOutletModal(options);
```

### Options

| Key | Type | Description |
|---|---|---|
| `routes` | `IOutletModal[]` | Tab route definitions |
| `pathname` | `string` | Default route on open |
| `history` | `History` | Internal memory history |
| `title` | `string` | Dialog title |
| `fetchState` | `async (id) => tuple` | Load all data needed for all tabs |
| `mapInitialData` | `(id, state) => TData` | Map loaded state to form `initialData` |
| `mapPayload` | `(id, state) => TPayload` | Map loaded state to `payload` |
| `onSubmit` | `(id, data) => boolean \| Promise<boolean>` | Submit button handler |
| `onClose` | `() => void` | Close handler |
| `submitLabel` | `string` | Submit button label |
| `withActionButton` | `boolean` | Show submit button |
| `withStaticAction` | `boolean` | Non-async submit variant |
| `sizeRequest` | `() => { width, height }` | Dialog size |
| `fullScreen` | `boolean` | Full-screen mode |
| `animation` | `string` | Transition animation (`"none"` to disable) |
| `BeforeTitle` | `React.ComponentType<{ onClose, id }>` | Left side of title bar |
| `AfterTitle` | `React.ComponentType<{ onClose, id }>` | Right side of title bar (close/action buttons) |
| `onLoadStart` | `() => void` | Loading start |
| `onLoadEnd` | `() => void` | Loading end |
| `fallback` | `(e: Error) => void` | Error handler |

### Usage pattern

```tsx
// 1. Define routes (tabs)
const routes: IOutletModal[] = [
  {
    id: "object",
    element: ObjectView,
    isActive: (p) => !!parseRouteUrl("/apartment/object", p),
  },
  {
    id: "owner_contact",
    element: OwnerContactView,
    isActive: (p) => !!parseRouteUrl("/apartment/owner_contact", p),
  },
  {
    id: "documents",
    element: DocumentsView,
    isActive: (p) => !!parseRouteUrl("/apartment/documents", p),
  },
];

// 2. Create hook
const { render, pickData } = useOutletModal({
  history,
  pathname: "/apartment/object",
  routes,
  title: " ",
  sizeRequest: CC_FULLSCREEN_SIZE_REQUEST,
  withActionButton: true,
  animation: "none",

  // Load all tab data in one request:
  fetchState: async (id) => [
    await ioc.contactViewService.read(contactId),
    await ioc.apartmentViewService.tryRead(id),
    await ioc.permissionService.getPermissions({ apartmentId: id }),
  ],
  mapInitialData: (_, [owner_contact, object]) => ({
    owner_contact,
    object,
    documents: object?.apartment_files || [],
  }),
  mapPayload: (id, [, , permissions]) => ({
    id,
    permissions,
    readonly: true,
    modal: true,
  }),

  // Custom title bar:
  BeforeTitle: ({ id }) => (
    <Async>
      {async () => {
        const apt = await ioc.apartmentViewService.read(id);
        return <Typography variant="h6">Объект №{apt.index}</Typography>;
      }}
    </Async>
  ),
  AfterTitle: ({ onClose, id }) => (
    <Stack direction="row" gap={1}>
      <IconButton size="small" onClick={() => { ioc.routerService.push(`/apartment_edit/${id}`); clear(); }}>
        <Edit />
      </IconButton>
      <IconButton size="small" onClick={onClose}><Close /></IconButton>
    </Stack>
  ),

  submitLabel: "Открыть",
  onSubmit: (id) => {
    ioc.routerService.push(`/apartment_view/${id}/object`);
    clear();
    return true;
  },
  onClose: () => pop(),
});

// 3. Mount render once, call pickData(id) to open
push({ id: "apartment", render, onMount: () => pickData(apartmentId) });
```
