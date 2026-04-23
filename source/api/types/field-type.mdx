---
title: "FieldType enum: all field and layout type values"
description: "Complete reference for FieldType enum values in react-declarative — every input, toggle, display, and layout type with descriptions and usage guidance."
---

`FieldType` is the core enum that tells the `<One />` component what kind of field or layout container to render for each entry in your `fields` array. Every field definition object requires a `type` property set to one of these values. Understanding which `FieldType` to choose is the foundation of building forms and layouts with react-declarative.

```ts
import { FieldType } from 'react-declarative';
```

Under the hood, each enum value is stored as a `Symbol` (via `Symbol.for(value)`), so comparisons always use the enum key rather than comparing raw strings.

---

## Input fields

These field types render interactive input controls that read from and write to your data object.

| Enum key | String value | Description |
|---|---|---|
| `FieldType.Text` | `'text-field'` | A Material UI `TextField`. Supports single-line and multiline (set `inputRows > 1`) modes, input formatters, leading/trailing icons, and autocomplete. The most commonly used field type. |
| `FieldType.Complete` | `'complete-field'` | An autocomplete text field. Provide a `tip` array or async function that returns suggestions as the user types. |
| `FieldType.Combo` | `'combo-field'` | A searchable single-select dropdown backed by an `itemList`. Supports async item loading, translation via `tr`, and free-solo input. |
| `FieldType.Items` | `'items-field'` | A multi-select version of `Combo`. The value is an array of selected strings. |
| `FieldType.Choose` | `'choose-field'` | A read-only display that opens a custom selection modal via the `choose` callback. Use when picking from a complex external data source. |
| `FieldType.Tree` | `'tree-field'` | A hierarchical tree selector backed by an `itemTree` array of `ITreeNode` objects. |
| `FieldType.Dict` | `'dict-field'` | A dictionary/lookup field that queries a remote search API (`dictSearch`) and shows results in a modal. Supports creating new records via `dictOnAppend`. |
| `FieldType.File` | `'file-field'` | A file upload field. Provide an `upload` async callback to handle the upload and return a file URL, and optionally a `view` callback to open the file. Use `fileAccept` to restrict MIME types. |
| `FieldType.Date` | `'date-field'` | A date picker backed by a Material UI date input. The value is a date string. |
| `FieldType.Time` | `'time-field'` | A time picker. The value is a time string. |

---

## Toggle fields

These field types render boolean or discrete-choice controls.

| Enum key | String value | Description |
|---|---|---|
| `FieldType.Checkbox` | `'checkbox-field'` | A Material UI `Checkbox`. The value is `true` or `false`. |
| `FieldType.Switch` | `'switch-field'` | A Material UI `Switch` toggle. The value is `true` or `false`. Optionally show a second label for the active state via `switchActiveLabel`. |
| `FieldType.YesNo` | `'yesno-field'` | A yes/no toggle rendered as two radio-style buttons. Useful for explicit binary choices where a checkbox feels ambiguous. |
| `FieldType.Radio` | `'radio-field'` | A single radio button. Use multiple `Radio` fields with different `radioValue` strings to build a radio group — only one will be selected at a time. |

---

## Numeric and range fields

These field types work with numeric values.

| Enum key | String value | Description |
|---|---|---|
| `FieldType.Slider` | `'slider-field'` | A Material UI `Slider`. Configure `minSlider`, `maxSlider`, and `stepSlider`. Supports custom thumb/track/rail colors and labeled step marks via `sliderSteps`. |
| `FieldType.Rating` | `'rating-field'` | A star rating control. The value is an integer (1–5 by default). |
| `FieldType.Progress` | `'progress-field'` | A read-only progress bar. The value is a number interpreted as a percentage of `maxPercent`. Use `showPercentLabel` to show the number. |

---

## Display fields

These field types render non-interactive content.

| Enum key | String value | Description |
|---|---|---|
| `FieldType.Typography` | `'typography-field'` | Renders static text using a Material UI `Typography` variant. Set `typoVariant` to control the heading/body style. |
| `FieldType.Button` | `'button-field'` | A standalone button inside the form layout. Configure appearance with `buttonVariant`, `buttonSize`, and `buttonColor`. |
| `FieldType.Icon` | `'icon-field'` | An icon button. Pass the icon component via `icon` and configure `iconSize`, `iconColor`, and `iconBackground`. |
| `FieldType.Line` | `'line-field'` | A horizontal divider rule. Set `lineTransparent` to render without a visible border (useful for vertical spacing). |

---

## Layout containers

Layout types are field entries that contain other fields via the `fields` array or `child` property. They control structure and visual grouping rather than capturing input.

| Enum key | String value | Description |
|---|---|---|
| `FieldType.Group` | `'group-layout'` | A flat row-based grid container. Children are laid out using the `columns`/`phoneColumns`/`tabletColumns`/`desktopColumns` system. The default layout primitive. |
| `FieldType.Paper` | `'paper-layout'` | Wraps children in a Material UI `Paper` card surface. Use `innerPadding` to control spacing. |
| `FieldType.Outline` | `'outline-layout'` | Wraps children in a bordered outline container with an optional `title` label. Similar to a `fieldset`. |
| `FieldType.Expansion` | `'expansion-layout'` | A collapsible accordion panel. Set `expansionOpened` to control the default expanded state. |
| `FieldType.Tabs` | `'tabs-layout'` | Renders children as tabbed panels. Configure tab labels with `tabList`, default tab with `tabIndex`, and style options like `tabVariant`, `tabColor`, and `tabBackground`. |
| `FieldType.Fragment` | `'fragment-layout'` | A transparent wrapper — renders children without any wrapping DOM element. Useful for conditional grouping. |
| `FieldType.Div` | `'div-layout'` | Renders a plain `<div>` container around its children. |
| `FieldType.Box` | `'box-layout'` | A Material UI `Box` container. Supports the `sx` prop for custom styles. |
| `FieldType.Center` | `'center-layout'` | Centers its children both horizontally and vertically. |
| `FieldType.Stretch` | `'stretch-layout'` | Stretches its children to fill the available height of the container. |
| `FieldType.Hero` | `'hero-layout'` | A positioning container with explicit `top`, `left`, `right`, `bottom`, `width`, and `height` props (with responsive variants). Use for precise canvas-style layouts. |
| `FieldType.Condition` | `'condition-layout'` | Renders its children only when a `condition` predicate returns `true`. Supports async predicates, a loading component (`conditionLoading`), and an else component (`conditionElse`). |
| `FieldType.Layout` | `'custom-layout'` | Renders a completely custom layout component provided via the `customLayout` prop. Use when no built-in layout container fits your needs. |

---

## Special / utility fields

These field types provide meta-level behavior that doesn't map to a visible UI element.

| Enum key | String value | Description |
|---|---|---|
| `FieldType.Component` | `'component-field'` | Embeds an arbitrary React component into the form via the `element` prop. The component receives `ComponentFieldInstance<Data, Payload>` as props, including the current `data`, `payload`, `onChange`, and field metadata. |
| `FieldType.Init` | `'init-field'` | Invisible field that runs a side-effect on mount. Use it for initialization logic that depends on the form data at render time without rendering any UI. |
| `FieldType.Phony` | `'phony-field'` | A field that holds a computed or virtual value not directly bound to any UI control. Useful for injecting derived values into the data object via `compute`. |

---

## Example

```tsx
import { FieldType, TypedField, One } from 'react-declarative';

interface ProfileData {
  name: string;
  bio: string;
  role: string;
  active: boolean;
}

const fields: TypedField<ProfileData>[] = [
  {
    type: FieldType.Paper,
    fields: [
      {
        type: FieldType.Group,
        fields: [
          {
            type: FieldType.Text,
            name: 'name',
            title: 'Full name',
            columns: '6',
          },
          {
            type: FieldType.Combo,
            name: 'role',
            title: 'Role',
            itemList: ['admin', 'editor', 'viewer'],
            columns: '6',
          },
        ],
      },
      {
        type: FieldType.Text,
        name: 'bio',
        title: 'Bio',
        inputRows: 4,
      },
      {
        type: FieldType.Switch,
        name: 'active',
        title: 'Active',
        switchActiveLabel: 'Enabled',
      },
    ],
  },
];

export default function ProfileForm({ data, onChange }) {
  return <One fields={fields} data={data} onChange={onChange} />;
}
```
