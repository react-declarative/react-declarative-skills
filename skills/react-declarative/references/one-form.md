# One — Form Component Reference

## Overview

`<One />` (alias `<OneTyped />`) renders a form from a `TypedField[]` schema. State is managed automatically — no `useState` needed. The component is generic: `One<Data, Payload>`.

## Key props

| Prop | Type | Notes |
|---|---|---|
| `fields` | `TypedField[]` | **Required.** The schema describing fields and layout. |
| `handler` | `Data \| ((payload) => Data \| Promise<Data>) \| null` | Loads initial data. Async-safe. Called on mount and on `reloadSubject` emission. |
| `data` | `Data \| null` | Alternative controlled pattern (do not mix with `handler`). |
| `onChange` | `(data: Data, initial: boolean) => void` | Fires on every change. `initial=true` on first load — skip saves then. |
| `payload` | `Payload \| (() => Payload)` | Context passed to all field callbacks. Does **not** trigger re-renders (use `context` for that). |
| `context` | `Record<string, any>` | Like `payload` but participates in change detection — triggers re-renders. |
| `dirty` | `boolean` | Show validation errors immediately on all fields without waiting for focus/blur. |
| `readonly` | `boolean` | All fields read-only. |
| `disabled` | `boolean` | All fields disabled. |
| `fieldDebounce` | `number` | Debounce ms for `FieldType.Text` before committing to form state. |
| `changeDelay` | `number` | Debounce ms applied to `onChange` emissions globally. |
| `fallback` | `(e: Error) => void` | Error handler if `handler` rejects. |
| `onReady` | `() => void` | Fired once after all fields finish first render. |
| `onInvalid` | `(name, msg, payload) => void` | Called when a field fails validation. |
| `onFocus` / `onBlur` | `(name, data, payload, onValueChange, onChange) => void` | Field focus/blur events. |
| `onClick` | `(name, data, payload, onValueChange, onChange, e) => void` | Field click. |
| `reloadSubject` | `TSubject<void>` | Emit to re-call `handler` and re-hydrate the form. |
| `changeSubject` | `TSubject<Data>` | Emit to replace data as initial (initial=true). |
| `updateSubject` | `TSubject<Data>` | Emit to update data as edit (initial=false). |
| `apiRef` | `React.Ref<IOneApi>` | Imperative handle: reload, read values, force re-validation. |
| `outlinePaper` | `boolean` | Converts Paper layouts to outlined style. |
| `transparentPaper` | `boolean` | Paper layouts with transparent background. |
| `baseline` / `noBaseline` | `boolean` | Field alignment in rows. |

## TypedField — field schema object

Every entry in `fields` must have `type: FieldType.*`. All other props are optional and depend on the type.

### Common props on every field

```ts
{
  type: FieldType,          // required
  name: string,             // maps to Data key; omit for layout containers
  title: string,            // label
  description: string,      // helper text below field
  placeholder: string,      // placeholder text
  defaultValue: Value | ((payload) => Value),
  columns: string,          // e.g. '6' = half-width (out of 12)
  phoneColumns: string,
  tabletColumns: string,
  desktopColumns: string,
  phoneHidden: boolean,
  tabletHidden: boolean,
  desktopHidden: boolean,
  hidden: boolean | ((payload) => boolean),  // static RBAC/feature-flag hide
}
```

### Field state callbacks (all receive `(data: Data, payload: Payload)`)

```ts
isInvalid(data, payload): null | string   // null = valid; string = error message (blocks submit)
isIncorrect(data, payload): null | string // warning, does not block submit
isVisible(data, payload): boolean         // false = remove from DOM (value preserved)
isDisabled(data, payload): boolean        // true = greys out and blocks input
isReadonly(data, payload): boolean        // true = read-only display
compute(data, payload): Value | Promise<Value>  // derived display value, always read-only
```

All can be async (return `Promise<boolean>` / `Promise<string | null>`).

### Icons on Text fields

```ts
leadingIcon: React.ComponentType,
trailingIcon: React.ComponentType,
leadingIconClick(value, data, payload, onValueChange, onChange): void,
trailingIconClick(value, data, payload, onValueChange, onChange): void,
```

### Focus/blur on individual fields

```ts
focus(name, data, payload, onValueChange, onChange): void,
blur(name, data, payload, onValueChange, onChange): void,
```

## TypedField generics

```ts
TypedField<Data = IAnything, Payload = IAnything>
```

- Without generics: all callbacks receive `any`
- With `Data`: `isInvalid`, `isVisible`, etc. receive typed `Data` — catches typos in field names at compile time
- With `Payload`: callbacks receive typed context object

## State management flow

1. `handler` runs → returns `Data`
2. `defaultValue` fills any missing keys
3. `onChange(data, true)` fires (initial)
4. User edits → `onChange(data, false)` fires on every change

## defaultValue vs handler

- `defaultValue` only fills keys that handler returned as `null`, `undefined`, or missing
- `defaultValue` can be a function `(payload) => Value` for runtime-dependent defaults

## Auto-save pattern

```tsx
const autoSave = useMemo(() => debounce(async (data) => {
  await fetch('/api/save', { method: 'PUT', body: JSON.stringify(data) });
}, 600), []);

<OneTyped
  fields={fields}
  handler={fetchData}
  fieldDebounce={300}
  onChange={(data, initial) => { if (!initial) autoSave(data); }}
/>
```

## Validation

```ts
// isInvalid: blocking
isInvalid({ email }) {
  return /^[^\s@]+@[^\s@]+\.[^\s@]+$/.test(email) ? null : 'Invalid email';
},

// Cross-field validation
isInvalid: ({ from, to }) => {
  if (from && to && parseInt(from) > parseInt(to)) return 'From must not exceed To';
  return null;
},

// Non-blocking warning
isIncorrect: ({ description }) =>
  description?.length < 20 ? 'Description too short' : null,
```

The `dirty` prop on a field or on `<One />` shows errors immediately without waiting for user interaction.

## validation shorthand

```ts
{ type: FieldType.Text, name: 'firstName', title: 'Name', validation: { required: true } }
```

## getAvailableFields utility

```ts
import { getAvailableFields } from 'react-declarative';
// Returns only fields whose isVisible is truthy — use before submitting to strip hidden fields
const visible = getAvailableFields(fields, formData, payload);
```