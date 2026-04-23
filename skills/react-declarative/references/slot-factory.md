# Slot Customization

## FieldType.Component — one-off injection

```typescript
{
  type: FieldType.Component,
  element: (props) => <MyWidget {...props} />,
}
```

Props inside `element`:

| Prop | Type | Description |
|---|---|---|
| `data` | `Data` | Full form data object |
| `payload` | `Payload` | External payload |
| `onChange` | `(data: Partial<Data>) => void` | Write partial data back to the form |
| `invalid` | `string \| null` | Current validation error |
| `disabled` / `readonly` | `boolean` | Field states |

```tsx
// Write-back example
{ type: FieldType.Component,
  element: ({ onChange }) => (
    <button onClick={() => onChange({ firstName: 'Jane', lastName: 'Smith' })}>Prefill</button>
  ) }
```

> `element` re-renders on every form change. Memoize the component or use `watchOneContext: true`.

## OneSlotFactory — global renderer replacement

```tsx
import { OneSlotFactory, One } from 'react-declarative';
import { MyTextInput } from './MyTextInput';
import { MyCheckBox  } from './MyCheckBox';

export const App = () => (
  <OneSlotFactory Text={MyTextInput} CheckBox={MyCheckBox}>
    <One fields={fields} handler={handler} />
  </OneSlotFactory>
);
```

An inner `OneSlotFactory` overrides an outer one — stack them for section-level overrides.

### Available slots

| Slot | Interface | FieldType |
|---|---|---|
| `Text` | `ITextSlot` | `FieldType.Text` |
| `CheckBox` | `ICheckBoxSlot` | `FieldType.CheckBox` |
| `Radio` | `IRadioSlot` | `FieldType.Radio` |
| `Combo` | `IComboSlot` | `FieldType.Combo` |
| `Items` | `IItemsSlot` | `FieldType.Items` |
| `Choose` | `IChooseSlot` | `FieldType.Choose` |
| `Date` | `IDateSlot` | `FieldType.Date` |
| `Time` | `ITimeSlot` | `FieldType.Time` |
| `Switch` | `ISwitchSlot` | `FieldType.Switch` |
| `YesNo` | `IYesNoSlot` | `FieldType.YesNo` |
| `Slider` | `ISliderSlot` | `FieldType.Slider` |
| `Rating` | `IRatingSlot` | `FieldType.Rating` |
| `Progress` | `IProgressSlot` | `FieldType.Progress` |
| `File` | `IFileSlot` | `FieldType.File` |
| `Dict` | `IDictSlot` | `FieldType.Dict` |
| `Tree` | `ITreeSlot` | `FieldType.Tree` |
| `Complete` | `ICompleteSlot` | `FieldType.Complete` |
| `Typography` | `ITypographySlot` | `FieldType.Typography` |
| `Button` | `IButtonSlot` | `FieldType.Button` |
| `Icon` | `IIconSlot` | `FieldType.Icon` |
| `Line` | `ILineSlot` | `FieldType.Line` |

### Custom slot example

```tsx
import { ITextSlot } from 'react-declarative';

export const MyTextInput = ({ value, onChange, title, disabled, readonly, invalid, dirty, description }: ITextSlot) => (
  <div>
    <label>{title}</label>
    <input value={String(value ?? '')} disabled={disabled || readonly}
      onChange={(e) => onChange(e.target.value)} />
    {dirty && invalid && <span className="error">{invalid}</span>}
    {description && <small>{description}</small>}
  </div>
);
```

Common props for all slots: `value`, `onChange`, `disabled`, `readonly`, `invalid`, `incorrect`, `dirty`, `title`, `placeholder`, `description`, `loading`.

## ListSlotFactory

```tsx
import { ListSlotFactory, List } from 'react-declarative';
<ListSlotFactory BodyRow={MyBodyRow}>
  <List columns={columns} handler={fetchContacts} />
</ListSlotFactory>
```

## react-declarative-mantine

Drop-in `OneSlotFactory` with Mantine components. Schemas stay unchanged.

```bash
npm install react-declarative-mantine
```

```tsx
import { MantineSlotFactory } from 'react-declarative-mantine';
<MantineSlotFactory>
  <One fields={fields} handler={handler} />
</MantineSlotFactory>
```

Preview: https://react-declarative-mantine.github.io/

## Component vs OneSlotFactory

- **FieldType.Component** — unique widget for a single form; custom visualizations; write-back via `onChange`
- **OneSlotFactory** — design-system-wide component for the whole app; Mantine/Chakra/Ant Design integration
