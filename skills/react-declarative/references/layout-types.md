# Layout Types — All Layout Containers Reference

Layout types are `FieldType` values that act as containers. They do not store values (no `name` needed). They wrap child fields via `fields: TypedField[]` (multiple children) or `child: TypedField` (single child).

---

## The 12-column responsive grid

react-declarative uses a 12-column grid. All column values are strings (not numbers):

```ts
// Full width
{ columns: '12' }

// Half-width desktop, full on phone
{ desktopColumns: '6', tabletColumns: '12', phoneColumns: '12' }
```

---

## FieldType.Group
Invisible flex-row container. Primary layout primitive.

```ts
{
  type: FieldType.Group,
  phoneColumns: '12',
  tabletColumns: '6',
  desktopColumns: '4',
  fieldRightMargin: '0',   // remove spacing between siblings
  fields: [
    { type: FieldType.Text, name: 'firstName', title: 'First name' },
    { type: FieldType.Text, name: 'lastName', title: 'Last name' },
  ],
}
```

---

## FieldType.Paper
MUI Paper card surface.

```ts
{
  type: FieldType.Paper,
  fieldBottomMargin: '1',
  innerPadding: '16px',
  outlinePaper: true,    // outlined border instead of elevation
  fields: [
    { type: FieldType.Line, title: 'Contact' },
    { type: FieldType.Text, name: 'phone', title: 'Phone', columns: '6' },
    { type: FieldType.Text, name: 'fax', title: 'Fax', columns: '6' },
  ],
}
```

---

## FieldType.Outline
Bordered section, visually similar to outlined MUI TextField (like a fieldset).

```ts
{
  type: FieldType.Outline,
  title: 'Permissions',
  fields: [
    { type: FieldType.Checkbox, name: 'canRead',  title: 'Read',  columns: '4' },
    { type: FieldType.Checkbox, name: 'canWrite', title: 'Write', columns: '4' },
    { type: FieldType.Checkbox, name: 'canAdmin', title: 'Admin', columns: '4' },
  ],
}
```

---

## FieldType.Expansion
MUI Accordion / collapsible panel.

```ts
{
  type: FieldType.Expansion,
  title: 'Advanced settings',
  description: 'Optional config',
  expansionOpened: false,   // default collapsed; set true to start open
  fields: [
    { type: FieldType.Switch, name: 'darkMode', title: 'Dark mode' },
    { type: FieldType.Switch, name: 'notifications', title: 'Push notifications' },
  ],
}
```

---

## FieldType.Tabs
MUI tab bar. Each entry in `fields` is one tab pane (in order).

```ts
{
  type: FieldType.Tabs,
  tabList: ['Profile', 'Security', 'Billing'],
  tabVariant: 'standard',    // 'standard' | 'scrollable' | 'fullWidth'
  tabColor: 'primary',
  tabLine: true,             // show underline indicator
  tabKeepFlow: true,         // grow height to match content
  fields: [
    // Tab 0 — Profile
    { type: FieldType.Group, fields: [
      { type: FieldType.Text, name: 'displayName', title: 'Display name' },
    ]},
    // Tab 1 — Security
    { type: FieldType.Group, fields: [
      { type: FieldType.Text, name: 'password', title: 'Password', inputType: 'password' },
    ]},
    // Tab 2 — Billing
    { type: FieldType.Typography, typoVariant: 'body1', placeholder: 'Billing coming soon.' },
  ],
}
```

---

## FieldType.Condition
Renders children only when `condition` predicate returns `true`. Removes from DOM when false.

```ts
{
  type: FieldType.Condition,
  condition: ({ accountType }) => accountType === 'business',
  // async also supported:
  // condition: async ({ accountType }, payload) => await checkBusiness(accountType),
  conditionLoading: () => <CircularProgress size={20} />,   // shown while async resolves
  conditionElse: () => <p>Switch to business account.</p>,  // shown when false
  fields: [
    { type: FieldType.Text, name: 'companyName', title: 'Company name' },
    { type: FieldType.Text, name: 'vatNumber', title: 'VAT number' },
  ],
}
```

Note: `isVisible` on a field hides it but value stays in data. `FieldType.Condition` removes entire subtree from DOM.

---

## FieldType.Fragment
React Fragment — zero DOM overhead. Use for conditional group visibility without wrapper element.

```ts
{
  type: FieldType.Fragment,
  isVisible: ({ isExpanded }) => isExpanded,
  fields: [
    { type: FieldType.Text, name: 'extraField1', title: 'Extra 1' },
    { type: FieldType.Text, name: 'extraField2', title: 'Extra 2' },
  ],
}
```

---

## FieldType.Box
MUI Box with `sx` support for one-off style overrides.

```ts
{
  type: FieldType.Box,
  sx: { border: '1px solid', borderColor: 'divider', borderRadius: 1, p: 2 },
  fields: [...],
}
```

---

## FieldType.Div
Plain `<div>` wrapper for custom styling.

```ts
{ type: FieldType.Div, style: { display: 'flex', gap: 8 }, fields: [...] }
```

---

## FieldType.Center
Centers children both horizontally and vertically.

```ts
{ type: FieldType.Center, fields: [{ type: FieldType.Typography, placeholder: 'Centered text' }] }
```

---

## FieldType.Stretch
Stretches children to fill available container height.

```ts
{ type: FieldType.Stretch, fields: [...] }
```

---

## FieldType.Hero
Absolute/relative positioning container with per-breakpoint position+size props.

```ts
{
  type: FieldType.Hero,
  top: '0px', left: '0px', width: '200px', height: '100px',
  // also: phoneTop, tabletTop, desktopTop, phoneLeft, etc.
  fields: [...],
}
```

---

## FieldType.Layout
Fully custom layout component.

```ts
{
  type: FieldType.Layout,
  customLayout: ({ children, onChange, _fieldData }) => (
    <div style={{ border: '2px dashed hotpink', padding: 16 }}>
      <p>Score: {_fieldData.score}</p>
      {children}
    </div>
  ),
  fields: [
    { type: FieldType.Slider, name: 'score', minSlider: 0, maxSlider: 10 },
  ],
}
```

---

## Common layout props

All layout types (and fields) support:

```ts
fieldRightMargin: string,   // spacing between siblings
fieldBottomMargin: string,  // spacing below this element
sx: SxProps,                // MUI sx styling on container
style: React.CSSProperties,
className: string,
```
