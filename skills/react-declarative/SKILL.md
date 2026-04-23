---
name: react-declarative
description: >
  Expert assistant for the `react-declarative` TypeScript/React library built on top of MUI v5.
  Use this skill whenever the user:
  - Asks how to build a form, data grid, kanban board, wizard, or app shell with react-declarative
  - Asks about any react-declarative API: TypedField, FieldType, One, OneTyped, List, ListTyped, Scaffold2, KanbanView, WizardView
  - Asks about field types (Text, Combo, Items, Switch, Checkbox, Rating, Slider, File, Date, etc.)
  - Asks about layout containers (Group, Paper, Outline, Expansion, Tabs, Condition, Fragment, Layout)
  - Asks about state management in react-declarative: handler, onChange, payload, isInvalid, isVisible, isDisabled
  - Asks about async hooks: useSinglerunAction, useQueuedAction, useAsyncValue, useAsyncProgress, useAsyncAction
  - Asks about MVVM collection hooks: useCollection, useModel, useEntity
  - Asks about routing: Switch component, ISwitchItem, useRouteParams, useRouteItem, parseRouteUrl
  - Asks about reactive programming: Subject, BehaviorSubject, Source, useSubject, useChangeSubject
  - Asks about dependency injection: provide, inject, createServiceManager, IService
  - Asks about slot factory customization: OneSlotFactory, ListSlotFactory
  - Has errors in react-declarative code and needs help fixing them
  - Wants to scaffold a project or understand project structure
  - Asks about AI-assisted form generation with TypedField schemas
  - Imports from `react-declarative` in their code
  Trigger even if the user just shows a `.tsx` file using One/List/Scaffold2/KanbanView and asks a question about it.
---

# react-declarative Expert

`react-declarative` is a TypeScript React library that turns `TypedField[]` JSON schemas into fully functional MUI v5-based forms, data grids, kanban boards, wizards, and app shells — with automatic state management, no manual `useState`/`useEffect` wiring needed.

## Core concepts

- **Schema-first**: UI is described as plain `TypedField[]` arrays, not JSX trees
- **`<One />`** renders forms; **`<List />`** renders data grids; both read from a `handler` and emit updates via `onChange`
- **`payload`** prop carries external context (user roles, feature flags) into every field callback without polluting form data
- **All field callbacks** (`isInvalid`, `isVisible`, `isDisabled`, `compute`) receive the full data object — cross-field logic is built-in

## Quick decision guide

| User wants | Recommend |
|---|---|
| A form | `<One />` / `<OneTyped />` with `TypedField[]` |
| A data grid | `<List />` / `<ListTyped />` with `IColumn[]` |
| An app shell | `<Scaffold2 />` / `<Scaffold3 />` with `IScaffold2Group[]` |
| A kanban board | `<KanbanView />` with `IBoardColumn[]` + `IBoardItem[]` |
| A multi-step wizard | `<WizardView />` with `IWizardStep[]` + `IWizardOutlet[]` |
| Custom field renderers | `OneSlotFactory` / `ListSlotFactory` |
| Async action dedup | `useSinglerunAction` |
| Ordered async queue | `useQueuedAction` |
| Async data in component | `useAsyncValue` |
| Batch progress | `useAsyncProgress` |
| Reactive observable list | `useCollection` |
| Route params in component | `useRouteParams` |

## Reference files

Load these when you need deep detail on a specific area:

- [`references/one-form.md`](references/one-form.md) — `<One />` props, TypedField, field callbacks, state management
- [`references/list-grid.md`](references/list-grid.md) — `<List />` props, IColumn, filters, actions, chips
- [`references/components.md`](references/components.md) — Scaffold2, KanbanView, WizardView
- [`references/field-types.md`](references/field-types.md) — All FieldType values with examples
- [`references/layout-types.md`](references/layout-types.md) — All layout containers (Group, Paper, Tabs, Condition, etc.)
- [`references/hooks.md`](references/hooks.md) — All async hooks + collection/MVVM hooks + navigation hooks
- [`references/advanced.md`](references/advanced.md) — Routing (Switch), reactive programming (Subject/Source), DI (provide/inject), SlotFactory, AI generation

## Minimal working examples

### Form with One

```tsx
import { One, TypedField, FieldType } from 'react-declarative';

interface IUser { firstName: string; email: string; role: string; }

const fields: TypedField<IUser>[] = [
  { type: FieldType.Text, name: 'firstName', title: 'First name' },
  { type: FieldType.Text, name: 'email', title: 'Email',
    isInvalid: ({ email }) => /^[^\s@]+@[^\s@]+\.[^\s@]+$/.test(email) ? null : 'Invalid email' },
  { type: FieldType.Combo, name: 'role', title: 'Role', itemList: ['admin', 'editor', 'viewer'] },
];

export const UserForm = () => (
  <One<IUser>
    fields={fields}
    handler={async () => fetchUser()}
    onChange={(data, initial) => { if (!initial) save(data); }}
  />
);
```

### Data grid with ListTyped

```tsx
import { ListTyped, IColumn, ColumnType, useArrayPaginator } from 'react-declarative';

const columns: IColumn<{}, IRow>[] = [
  { type: ColumnType.Text, field: 'name', headerName: 'Name', width: '200px', sortable: true },
  { type: ColumnType.CheckBox, field: 'active', headerName: 'Active', width: '80px' },
];

export const UserGrid = () => (
  <ListTyped withSearch withArrowPagination columns={columns} handler={useArrayPaginator(rows)} />
);
```

## Installation

```bash
npm install --save react-declarative tss-react @mui/material @emotion/react @emotion/styled
```

Requires **MUI v5** (`@mui/material ^5.5.0`). Not compatible with MUI v4 or v6.

Lite variant (forms only, smaller footprint):
```bash
npm install --save react-declarative-lite
```

## Key rules when generating code

1. **`name` maps directly to a data key** — layout fields (Group, Paper, etc.) never need `name`
2. **Column values are strings** — `columns: '6'`, not `columns: 6`
3. **`handler` can be async** — no need for external loading state
4. **`onChange` fires with `initial: true`** on first load — skip saves on initial emission
5. **`payload` is not form data** — use it for user roles, feature flags; it does not trigger re-renders (use `context` prop if you need reactive context)
6. **`isInvalid` returns `null` for valid, string for error** — returning `null` is required (not `undefined`)
7. **`TypedField<Data, Payload>`** generics give full IntelliSense on field names and callbacks — always recommend using them
8. **`OneTyped` / `ListTyped`** are stricter wrappers — prefer them in new code
9. **Playground**: <https://react-declarative-playground.github.io/> — paste a `fields` array to preview instantly
