# Guide: Custom Fields — Component Injection and SlotFactory

Two complementary ways to customize what renders inside a form.

## `FieldType.Component` — one-off JSX injection

Drop any React component into a specific slot in your schema. Receives full form data and can write back.

```tsx
const fields: TypedField[] = [
  {
    type: FieldType.Paper,
    fields: [
      {
        type: FieldType.Component,
        element: (props) => <StatusBadge {...props} />,
      },
    ],
  },
];
```

### Props available inside `element`

| Prop | Type | Description |
|------|------|-------------|
| `data` | `Data` | Current form data object |
| `payload` | `Payload` | External payload passed to `<One />` |
| `onChange` | `(data: Partial<Data>) => void` | Merge partial data back into form |
| `name` | `string` | The `name` from field schema |
| `invalid` | `string \| null` | Current validation error |
| `disabled` | `boolean` | Whether field is disabled |
| `readonly` | `boolean` | Whether field is read-only |

### Writing back to the form

```tsx
const PrefillButton = ({ onChange }) => (
  <button onClick={() => onChange({ firstName: 'Jane', lastName: 'Smith' })}>
    Prefill demo data
  </button>
);

{
  type: FieldType.Component,
  element: (props) => <PrefillButton onChange={props.onChange} />,
}
```

> `element` re-renders on every form change. Keep it a thin wrapper — put real logic inside the referenced component.

## `OneSlotFactory` — replace built-in renderers globally

Wrap `<One />` (or the entire app) to apply overrides everywhere inside the provider.

```tsx
import { OneSlotFactory, One } from 'react-declarative';

const App = () => (
  <OneSlotFactory Text={MyInput} CheckBox={MyCheckBox}>
    <One fields={fields} handler={handler} />
  </OneSlotFactory>
);
```

An inner `OneSlotFactory` overrides an outer one — stack them for section-level overrides.

### Available slots

Input: `Text`, `Date`, `Time`, `Slider`, `Rating`, `Progress`  
Selection: `CheckBox`, `Switch`, `YesNo`, `Radio`, `Combo`, `Items`, `Complete`, `Tree`, `Dict`, `Choose`  
Action: `Button`, `Icon`  
Display: `Line`, `Typography`, `File`

### Slot component props

Each slot receives a typed interface from `react-declarative` (e.g. `ITextSlot`, `ICheckBoxSlot`):

```tsx
import { ITextSlot } from 'react-declarative';

const MyInput = ({ value, onChange, disabled, readonly, title, description, invalid }: ITextSlot) => (
  <div>
    <label>{title}</label>
    <input
      value={value ?? ''}
      disabled={disabled}
      readOnly={readonly}
      onChange={(e) => onChange(e.target.value)}
    />
    {invalid && <span className="error">{invalid}</span>}
    {description && <small>{description}</small>}
  </div>
);
```

Common props for all slots: `value`, `onChange`, `disabled`, `readonly`, `invalid`, `incorrect`, `dirty`, `title`, `placeholder`, `description`, `loading`.

## When to use each

| Use `FieldType.Component` when... | Use `OneSlotFactory` when... |
|---|---|
| One-off widget unique to a single form | Every field of a type needs consistent custom design |
| Custom data visualizations (charts, badges) | Integrating a UI library (Mantine, Chakra, Ant Design) |
| Writes back partial data via `onChange` | Site-wide theme or accessibility enhancement |

## `react-declarative-mantine` — ready-made Mantine theme

```bash
npm install react-declarative-mantine
```

```tsx
import { MantineSlotFactory } from 'react-declarative-mantine';

<MantineSlotFactory>
  <One fields={fields} handler={handler} />
</MantineSlotFactory>
```

Schemas stay unchanged — Mantine-styled renderers are swapped in automatically.
