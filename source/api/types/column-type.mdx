---
title: "ColumnType enum: Text, Action, Compute, and Component"
description: "API reference for the ColumnType enum in react-declarative — all five column types, what each renders in a List grid, and IColumn definition examples."
---

`ColumnType` controls how each column in a `<List />` component renders its cell content. You set it on the `type` property of an `IColumn` definition object. Each type has a distinct rendering strategy — from plain text to interactive action menus to fully custom React components.

```ts
import { ColumnType } from 'react-declarative';
```

---

## ColumnType.Text

Renders the cell value as a plain text string. This is the most commonly used column type. The value is read directly from the row data using the column's `field` property, or computed dynamically via `compute`.

```ts
{
  type: ColumnType.Text,
  field: 'email',
  headerName: 'Email',
  width: 200,
}
```

Use `compute` when you need to derive a display value from multiple fields:

```ts
{
  type: ColumnType.Text,
  headerName: 'Full Name',
  compute: (row) => `${row.firstName} ${row.lastName}`,
  width: 180,
}
```

---

## ColumnType.Action

Renders a column of action buttons or a context menu. You provide an `actions` array and the column renders a `...` overflow menu (or inline icon buttons) for each row. Use this for row-level operations like Edit, Delete, or View.

```ts
import { ColumnType } from 'react-declarative';

{
  type: ColumnType.Action,
  headerName: 'Actions',
  width: 80,
  actions: [
    {
      label: 'Edit',
      action: 'edit',
    },
    {
      label: 'Delete',
      action: 'delete',
    },
  ],
}
```

Handle the selected action in the `List` component's `onRowAction` prop:

```tsx
<List
  columns={columns}
  handler={() => fetchRows()}
  onRowAction={(action, row) => {
    if (action === 'edit') navigate(`/items/${row.id}/edit`);
    if (action === 'delete') deleteItem(row.id);
  }}
/>
```

---

## ColumnType.CheckBox

Renders a read-only `Checkbox` in each cell. The cell value is interpreted as a boolean — truthy values show a checked box, falsy values show an unchecked one. Use this to visualize boolean flags like `isActive`, `isVerified`, or `isPaid`.

```ts
{
  type: ColumnType.CheckBox,
  field: 'isActive',
  headerName: 'Active',
  width: 80,
}
```

---

## ColumnType.Compute

Calls a synchronous `compute` function for each row and renders the returned string as the cell content. Use this type when you need a derived display value that does not correspond to a single field on the row object.

Unlike `ColumnType.Text` with a `compute` callback, `ColumnType.Compute` makes the computed intent explicit in the type and is preferred when the column value is entirely derived.

```ts
{
  type: ColumnType.Compute,
  headerName: 'Status',
  width: 120,
  compute: (row) => {
    if (row.deletedAt) return 'Deleted';
    if (!row.verifiedAt) return 'Pending';
    return 'Active';
  },
}
```

---

## ColumnType.Component

Renders an arbitrary React component inside the cell, receiving the full row object as props. Use this when you need rich cell content — badges, avatars, sparklines, links — that plain text cannot express.

```ts
{
  type: ColumnType.Component,
  headerName: 'Progress',
  width: 150,
  element: ({ row }) => (
    <LinearProgress variant="determinate" value={row.completionPercent} />
  ),
}
```

The `element` prop receives an object with at minimum a `row` property typed to your row data shape.

---

## Full example

```tsx
import { List, ColumnType, IColumn } from 'react-declarative';

interface Order {
  id: string;
  customer: string;
  total: number;
  paid: boolean;
  status: 'pending' | 'shipped' | 'delivered';
}

const columns: IColumn<Order>[] = [
  {
    type: ColumnType.Text,
    field: 'customer',
    headerName: 'Customer',
    width: 200,
  },
  {
    type: ColumnType.Compute,
    headerName: 'Total',
    width: 100,
    compute: (row) => `$${row.total.toFixed(2)}`,
  },
  {
    type: ColumnType.CheckBox,
    field: 'paid',
    headerName: 'Paid',
    width: 70,
  },
  {
    type: ColumnType.Component,
    headerName: 'Status',
    width: 130,
    element: ({ row }) => <StatusBadge status={row.status} />,
  },
  {
    type: ColumnType.Action,
    headerName: '',
    width: 60,
    actions: [
      { label: 'View',   action: 'view' },
      { label: 'Cancel', action: 'cancel' },
    ],
  },
];

export default function OrderList() {
  return (
    <List
      columns={columns}
      handler={() => fetchOrders()}
      onRowAction={(action, row) => {
        if (action === 'view')   navigate(`/orders/${row.id}`);
        if (action === 'cancel') cancelOrder(row.id);
      }}
    />
  );
}
```
