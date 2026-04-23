# List — Data Grid Component Reference

## Overview

`<List />` (alias `<ListTyped />`) is a batteries-included data grid. Generic: `List<FilterData, RowData, Payload>`.

You supply:
- `handler` — fetches rows, receives current filter/pagination/sort state
- `columns` — column definitions (`IColumn[]`)
- `filters` — same `TypedField[]` schema as `<One />`, rendered as a filter panel
- `actions`, `rowActions`, `chips`, `operations` — toolbar and row-level interactions

## handler types

```ts
// Static array (client-side, no server call)
handler: RowData[]

// Async (server-side pagination)
handler: async (filterData, pagination, sort, chips, search, payload) => {
  return { rows: RowData[], total: number };
  // or just RowData[] when total is unknown
}

// pagination shape: { limit: number, offset: number }
// sort: IListSortItem[] e.g. [{ field: 'name', sort: 'asc' }]
// chips: Record<keyof RowData, boolean>
// search: string
```

`useArrayPaginator(rows)` wraps a static array into the async handler interface.

## Key props

| Prop | Type | Notes |
|---|---|---|
| `handler` | `ListHandler<FilterData, RowData>` | **Required.** |
| `columns` | `IColumn[]` | **Required.** |
| `filters` | `TypedField[]` | Filter panel schema (same as One). |
| `payload` | `Payload` | Forwarded to handler, column callbacks, action visibility. |
| `actions` | `IListAction[]` | Toolbar buttons. |
| `operations` | `IListOperation[]` | Bulk operations shown when rows are selected. |
| `rowActions` | `IListRowAction[]` | Per-row context menu. |
| `chips` | `IListChip[]` | Toggle chips above grid. |
| `withSearch` | `boolean` | Show search text box. |
| `withMobile` | `boolean` | Mobile card layout with `phoneOrder`/`phoneHidden` on columns. |
| `withArrowPagination` | `boolean` | Prev/Next arrows instead of page numbers. |
| `withRangePagination` | `boolean` | Page range selector. |
| `withSingleSort` | `boolean` | One sort column at a time. |
| `withSelectOnRowClick` | `boolean` | Toggle selection on row click. |
| `withToggledFilters` | `boolean` | Start with filter panel collapsed. |
| `selectionMode` | `SelectionMode` | None / Single / Multiple. |
| `limit` | `number` | Initial page size. |
| `page` | `number` | Initial page index (zero-based). |
| `rowsPerPage` | `number[]` | Options for rows-per-page selector. |
| `filterData` | `Partial<FilterData>` | Pre-populate filter form. |
| `sortModel` | `IListSortItem[]` | Initial sort. |
| `chipData` | `Partial<Record<keyof RowData, boolean>>` | Initial chip states. |
| `search` | `string` | Pre-populate search box. |
| `selectedRows` | `RowId[]` | Controlled selection. |
| `reloadSubject` | `TSubject<void>` | Emit to trigger fresh handler call. |
| `rerenderSubject` | `TSubject<void>` | Force visual re-render without refetch. |
| `setFilterDataSubject` | `TSubject<FilterData>` | Programmatically update filter form. |

## Callbacks

```ts
onRowClick(row, reload): void
onAction(action, selectedRows, reload): void
onRowAction(action, row, reload): void
onOperation(action, selectedRows, isAll, reload): void
onSelectedRows(rowIds, initialChange): void
onFilterChange(data): void
onSearchChange(search): void
onSortModelChange(sort): void
onRows(rows): void
fallback(e): void
onLoadStart(source): void
onLoadEnd(isOk, source): void
```

`reload(keepPagination?)` refreshes the grid — call after mutations.

## IColumn definition

```ts
{
  type: ColumnType,         // required — see ColumnType below
  field: string,            // key in RowData
  headerName: string,       // column header label
  width: string | ((containerWidth: number) => string | number),  // required
  sortable: boolean,
  compute: (row, payload) => Value | Promise<Value>,
  element: React.ComponentType<RowData & { _payload: Payload }>,
  phoneHidden: boolean,
  tabletHidden: boolean,
  primary: boolean,         // mobile layout priority
  isVisible: (params) => boolean,  // hide entire column conditionally
  columnMenu: IListActionOption[],
}
```

## ColumnType enum

| Value | Description |
|---|---|
| `ColumnType.Text` | Plain text, reads `field` or uses `compute` |
| `ColumnType.Compute` | Explicitly derived value via `compute` function |
| `ColumnType.CheckBox` | Read-only checkbox, truthy/falsy value |
| `ColumnType.Action` | Row action menu; define `actions` array on column |
| `ColumnType.Component` | Custom React component via `element` prop; receives row as props |

## Actions

```ts
// Toolbar actions
actions: IListAction[] = [
  { type: ActionType.Add, label: 'Create' },
  { type: ActionType.Menu, label: 'More', options: [...] },
]

// Per-row actions
rowActions: IListRowAction[] = [
  { label: 'Edit', action: 'edit', isVisible: ({ active }) => active },
  { label: 'Delete', action: 'delete' },
]

// Bulk operations (shown when rows selected)
operations: IListOperation[] = [
  { label: 'Export selected', action: 'export' }
]
```

## Chips (quick filter toggles)

```ts
chips: IListChip[] = [
  { name: 'isActive', label: 'Active', color: '#4caf50' },
  { name: 'isPremium', label: 'Premium', color: '#9c27b0' },
]
// Chip state passed to handler as chips: Record<keyof RowData, boolean>
```

## ListTyped vs List

Both are identical at runtime. `ListTyped` enforces generics at component level — prefer it in new code.

## Minimal example

```tsx
import { ListTyped, IColumn, ColumnType, TypedField, FieldType, useArrayPaginator } from 'react-declarative';

interface IRow { id: string; name: string; email: string; active: boolean; }

const columns: IColumn<{}, IRow>[] = [
  { type: ColumnType.Text, field: 'name', headerName: 'Name', width: '200px', sortable: true },
  { type: ColumnType.Text, field: 'email', headerName: 'Email', width: '250px' },
  { type: ColumnType.CheckBox, field: 'active', headerName: 'Active', width: '80px' },
  { type: ColumnType.Action, headerName: '', width: '60px',
    actions: [{ label: 'Edit', action: 'edit' }, { label: 'Delete', action: 'delete' }] },
];

const filters: TypedField[] = [
  { type: FieldType.Text, name: 'name', title: 'Name' },
];

export const UserList = ({ rows }) => (
  <ListTyped<{}, IRow>
    withSearch
    withMobile
    withArrowPagination
    filters={filters}
    columns={columns}
    handler={useArrayPaginator(rows)}
    onRowClick={(row) => navigate(`/users/${row.id}`)}
    onRowAction={(action, row, reload) => {
      if (action === 'edit') openEditDialog(row.id).then(reload);
      if (action === 'delete') deleteUser(row.id).then(reload);
    }}
  />
);
```
