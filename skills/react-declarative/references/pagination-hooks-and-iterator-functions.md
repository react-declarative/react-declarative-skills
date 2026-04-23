# Pagination Hooks & Iterator Functions

All symbols below are imported from `react-declarative` except `useBidOffsetPaginator`, which is a local project wrapper.

---

## Pagination Hooks

### `useArrayPaginator`

Wraps a `paginate` service method into the `handler` interface expected by `<ListTyped />`. Manages appbar loader state and error handling.

```ts
import { useArrayPaginator } from "react-declarative";

// api/useApartmentArrayPaginator.ts
export const useApartmentArrayPaginator = () =>
  useArrayPaginator(ioc.apartmentViewService.paginate, {
    onLoadStart: () => ioc.layoutService.setAppbarLoader(true),
    onLoadEnd:   () => ioc.layoutService.setAppbarLoader(false),
    fallback:    ioc.errorService.handleGlobalError,
  });
```

**Returns:** a `handler` value ready to pass directly to `<ListTyped handler={...} />`.

**Used in:** `src/api/use*ArrayPaginator.ts` — one file per entity (Apartment, Bid, Contact, User, Role).

---

### `useOffsetPaginator`

Implements offset-based pagination for custom infinite-scroll / load-more scenarios. The `handler` receives `(limit, offset, initial)` and must return an array of rows.

```ts
import { useOffsetPaginator, resolveDocuments } from "react-declarative";

const paginator = useOffsetPaginator<IApartmentRow>({
  handler: async (limit, offset, initial) => {
    const iterator = makeApartmentIterator({ filterData, payload, id });
    return await resolveDocuments(iterator(limit, offset));
  },
  fallback:      ioc.errorService.handleGlobalError,
  reloadSubject,           // Subject<void> — emit to refresh
});

// paginator.data   — accumulated rows
// paginator.loading
```

**Used in:** `src/components/AutoFind/hooks/use*OffsetPaginator.ts`.

---

### `useBidOffsetPaginator` *(local wrapper)*

Project-specific hook that wires `useOffsetPaginator` with bid-specific filter data, logging, and auto-reload on filter change.

```ts
// src/components/AutoFind/hooks/useBidOffsetPaginator.ts
export const useBidOffsetPaginator = ({ fieldData, payload }: IParams) => {
  const reloadSubject = useSubject<void>(bidColumnManager.reloadSubject);
  const fieldDataChangeSubject = useChangeSubject(fieldData);
  const getFilterData = useBidFilterData({ fieldData, payload });

  // Reload grid whenever filter data changes
  useOnce(() => fieldDataChangeSubject.subscribe(() => reloadSubject.next()));

  return {
    paginator: useOffsetPaginator<IApartmentRow>({
      handler: async (limit, offset, initial) => {
        const iterator = makeBidIterator({
          filterData: getFilterData(),
          payload,
          id: payload.id,
        });
        return await resolveDocuments(iterator(limit, offset));
      },
      fallback: ioc.errorService.handleGlobalError,
      reloadSubject,
    }),
    reloadSubject,
  };
};
```

**Returns:** `{ paginator, reloadSubject }`.

**Used in:** `AutoFind/components/BidAutoFind/BidAutoFind.tsx`.

---

## Iterator / Generator Utilities

These three functions create composable lazy iterators used by the AutoFind feature to search across multiple filtered subsets.

### `iteratePromise`

Wraps a single `async () => T[]` function into an iterator. Use when the full result set is fetched in one request (e.g. loading marked items from a lookup table).

```ts
import { iteratePromise, CANCELED_PROMISE_SYMBOL } from "react-declarative";

// makeGreenIterator — items the agent manually marked green
export const makeGreenIterator = ({ id }: IParams) =>
  iteratePromise(async () => {
    const markList = await ioc.autofindService.readApartmentMarkList(id);
    if (markList === CANCELED_PROMISE_SYMBOL) return [];
    return await Promise.all(
      markList
        .filter(({ color }) => color === "green")
        .map(({ apartmentId }) => ioc.apartmentViewService.read(apartmentId)),
    );
  });
```

Supports `CANCELED_PROMISE_SYMBOL` — return `[]` when the promise was cancelled to stop iteration cleanly.

---

### `iterateDocuments`

Creates a paginated iterator that makes repeated `createRequest({ limit, offset })` calls until exhausted. Use for large collections fetched page-by-page.

```ts
import { iterateDocuments } from "react-declarative";

const ITERATION_LIMIT = 25;

export const makeFrozenIterator = ({ filterData, payload }: IParams) =>
  iterateDocuments<IApartmentRow>({
    limit: ITERATION_LIMIT,
    async createRequest({ limit, offset }) {
      const result = await ioc.apartmentViewService.paginate(
        { ...filterData, _mode: "audit" },
        { limit, offset },
        [], {}, "",
        { ...payload, _archive: true },
      );
      const rows: IApartmentRow[] = "rows" in result ? result.rows : result;
      return rows.filter(({ system_is_archive }) => system_is_archive);
    },
  });
```

**Config:**
| Key | Type | Description |
|---|---|---|
| `limit` | `number` | Page size per request |
| `createRequest` | `async ({ limit, offset }) => T[]` | Returns one page; empty array signals end |

---

### `iterateUnion`

Merges an array of iterators into a single sequential iterator. Results from each source appear in order.

```ts
import { iterateUnion } from "react-declarative";

export const makeApartmentIterator = ({ filterData, payload, id }: IParams) =>
  iterateUnion([
    makeGreenIterator({ filterData, payload, id }),      // iteratePromise
    makeUnfrozenIterator({ filterData, payload, id }),   // iterateDocuments
    makeFrozenIterator({ filterData, payload, id }),     // iterateDocuments
    makeRedIterator({ filterData, payload, id }),        // iteratePromise
  ]);
```

The combined iterator is then consumed by `useOffsetPaginator` via `resolveDocuments(iterator(limit, offset))`.

**Iterator pipeline in AutoFind:**

```
iterateUnion([
  iteratePromise  — green marks (manual)
  iterateDocuments — active (unfrozen) records
  iterateDocuments — archived (frozen) records
  iteratePromise  — red marks (manual)
])
      ↓
resolveDocuments(iterator(limit, offset))
      ↓
useOffsetPaginator handler
```

---

## UI Management Hooks

### `useColumnConfig`

Manages which columns are visible in a `<ListTyped />` grid. Persists user preferences to `localStorage` by `storageKey`.

```ts
import { useColumnConfig } from "react-declarative";

const { columns, pickColumns, render } = useColumnConfig({
  columns:    apartment_columns,   // IColumn[]
  storageKey: "apartment_columns", // localStorage key
});

// Pass columns to the grid
<ListTyped columns={columns} ... />

// Toolbar button to open column picker
<Button onClick={pickColumns}>Configure columns</Button>

// Render the picker modal
{render}
```

**Returns:**
| Key | Description |
|---|---|
| `columns` | Filtered `IColumn[]` based on current user config |
| `pickColumns` | Opens the column picker dialog |
| `render` | ReactNode — mount anywhere to enable the dialog |

**Used in:** All list pages (`SellPage`, `RentPage`, `RemovePage`, `CommonPage`) and AutoFind views.

---

### `useGridAction`

Manages row selection and action dispatch for a `<ListTyped />` grid. Handles fetching full row data before calling `onAction`.

```ts
import { useGridAction } from "react-declarative";

const { gridProps, commitAction, deselectAll } = useGridAction({
  fetchRow:  (id) => ioc.apartmentViewService.read(id),
  onAction:  async (action, rows, deselectAll) => {
    if (action === "pdf-action") {
      await handlePDFPublish(rows);
      deselectAll();
    }
    if (action === "telegram-action") {
      const channel = await pickPublishTelegram().toPromise();
      if (!channel) return;
      for (const row of rows) {
        await publishTelegram({ data: row, telegram_channel_id: channel.id });
      }
      deselectAll();
    }
  },
  onSelectionChange,
  selectedRows:  upperSelectedRows,
  onLoadStart:   handleLoadStart,
  onLoadEnd:     handleLoadEnd,
  fallback:      ioc.errorService.handleGlobalError,
});

// Wire to external subjects
useOnce(() => reloadSubject.subscribe(deselectAll));
useOnce(() => actionSubject.subscribe(commitAction));

// Pass gridProps to the list
<ListTyped {...gridProps} ... />
```

**Config:**
| Key | Description |
|---|---|
| `fetchRow(id)` | Loads full row data by ID before action |
| `onAction(action, rows, deselectAll)` | Action handler; `rows` are fully loaded |
| `onSelectionChange` | Called when selection changes |
| `selectedRows` | Controlled selection |
| `onLoadStart/End` | Loading indicator callbacks |
| `fallback` | Error handler |

**Returns:**
| Key | Description |
|---|---|
| `gridProps` | Spread onto `<ListTyped />` |
| `commitAction(action)` | Programmatically trigger an action |
| `deselectAll()` | Clear all selected rows |

**Used in:** AutoFind views, AuditPages, ReviewPages.

---

### `useMediaContext`

Returns responsive breakpoint information from the nearest MUI `ThemeProvider`.

```ts
import { useMediaContext } from "react-declarative";

const { isMobile } = useMediaContext();

// Hide bulk actions on mobile
{!isMobile && <BulkActionToolbar />}
```

**Returns (commonly used):**
| Key | Type | Description |
|---|---|---|
| `isMobile` | `boolean` | Viewport is below MUI `sm` breakpoint |

**Used throughout:** AutoFind, Calendar, LeadProvider, SmsCard, and most list/form pages.
