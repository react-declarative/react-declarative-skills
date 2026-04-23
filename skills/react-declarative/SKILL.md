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

- [`references/installation.md`](references/installation.md) — install, peer deps, tsconfig, first form/grid, Next.js/Vite notes
- [`references/one-form.md`](references/one-form.md) — `<One />` props, TypedField schema, field callbacks, reloadSubject, apiRef
- [`references/state-management.md`](references/state-management.md) — handler, onChange/initial flag, payload vs context, isInvalid/isVisible/compute, dirty, defaultValue
- [`references/list-grid.md`](references/list-grid.md) — `<List />` props, IColumn, ColumnType, filters, actions, chips, rowActions, operations
- [`references/field-types.md`](references/field-types.md) — All FieldType values with examples (Text, Combo, Items, Switch, Slider, File, Component, Init, Phony…)
- [`references/layout-types.md`](references/layout-types.md) — All layout containers (Group, Paper, Tabs, Condition, Fragment, Box, Layout…)
- [`references/components.md`](references/components.md) — Scaffold2, KanbanView, WizardView
- [`references/hooks.md`](references/hooks.md) — Async hooks (useSinglerunAction, useQueuedAction, useAsyncValue…) + navigation hooks + useCollection/useModel
- [`references/reactive.md`](references/reactive.md) — Subject, BehaviorSubject, EventEmitter, useSubject, useChangeSubject, Source pipelines
- [`references/routing.md`](references/routing.md) — Switch router, ISwitchItem, guards, prefetch/unload, redirects, OutletView, useRouteParams
- [`references/dependency-injection.md`](references/dependency-injection.md) — provide/inject, TYPES symbols, scoped containers, route lifecycle with DI
- [`references/slot-factory.md`](references/slot-factory.md) — OneSlotFactory, ListSlotFactory, all slot interfaces, react-declarative-mantine
- [`references/advanced.md`](references/advanced.md) — AI-assisted form generation, playground, prompt workflow, reference sample list

## How-to guides (narrative explanations with full examples)

Load these for deeper "how does this work" context on cross-cutting topics:

- [`references/guide-async-data.md`](references/guide-async-data.md) — choosing between handler/useSinglerunAction/useQueuedAction/useAsyncProgress/useAsyncValue/ActionButton
- [`references/guide-conditional-fields.md`](references/guide-conditional-fields.md) — isVisible vs FieldType.Condition vs hidden prop, isDisabled, isReadonly, payload for RBAC
- [`references/guide-custom-fields.md`](references/guide-custom-fields.md) — FieldType.Component write-back, OneSlotFactory slot interfaces, when to use each
- [`references/guide-form-validation.md`](references/guide-form-validation.md) — isInvalid, cross-field rules, async validation, isIncorrect, dirty flag, getAvailableFields
- [`references/guide-routing.md`](references/guide-routing.md) — Switch setup, guards, prefetch/unload, redirects with params, OutletView, useRouteParams
- [`references/guide-architecture.md`](references/guide-architecture.md) — full project architecture: folder structure, DI wiring (types/config/ioc), DB/View service layers, declarative assets, route config, modal pattern, HOCs, entry point providers stack, checklist for adding a new entity

## Example schemas (real-world TypedField[] code)

Load these when you need a working example close to what the user is asking for:

- [`references/login_form_example.md`](references/login_form_example.md) — centered login card with email/password and action feedback
- [`references/account_info_example.md`](references/account_info_example.md) — account settings page with profile fields and permissions
- [`references/profile_card_form_example.md`](references/profile_card_form_example.md) — avatar layout, nested groups, Component injection
- [`references/settings_page_form_example.md`](references/settings_page_form_example.md) — settings page with Switch, Slider, Rating, Progress
- [`references/adaptive_form_example.md`](references/adaptive_form_example.md) — responsive form with desktopColumns/tabletColumns/phoneColumns breakpoints
- [`references/variant_form_example.md`](references/variant_form_example.md) — conditional field visibility with isVisible / FieldType.Condition
- [`references/order_info_form_example.md`](references/order_info_form_example.md) — multi-section order form with Paper, Date, Combo
- [`references/product_shape_form_example.md`](references/product_shape_form_example.md) — product data form with nested structure
- [`references/rate_card_form_example.md`](references/rate_card_form_example.md) — rate card / pricing form
- [`references/crypto_form_example.md`](references/crypto_form_example.md) — crypto / financial data entry form
- [`references/machine_learning_form_example.md`](references/machine_learning_form_example.md) — ML model configuration form
- [`references/dashboard_form_example.md`](references/dashboard_form_example.md) — dashboard layout with KPI cards and read-only display panels
- [`references/kpi_review_example.md`](references/kpi_review_example.md) — KPI review / metrics display form
- [`references/google_forms_like_example.md`](references/google_forms_like_example.md) — survey-style form similar to Google Forms
- [`references/custom_form_example.md`](references/custom_form_example.md) — custom layout with FieldType.Layout / FieldType.Component
- [`references/gallery_of_controls_example.md`](references/gallery_of_controls_example.md) — every FieldType in one form; use as a lookup when a less common field type is needed
- [`references/typography_example.md`](references/typography_example.md) — typography and display-only fields (FieldType.Typography, Line, Icon)

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

## When you're stuck — reference implementation

If something isn't working, the patterns are unclear, or the documentation doesn't cover your case, clone the reference CRM application — it's a complete real-world project using react-declarative end-to-end:

```bash
git clone https://github.com/react-declarative/react-pocketbase-crm.git
```

It demonstrates: routing with Switch + Scaffold2, forms with One, data grids with List, dependency injection with provide/inject, real-time with Subject/BehaviorSubject, and full TypeScript generics throughout. Browse `src/` to find working examples of any pattern you need.

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
