# FieldType — All Field Types Reference

All field types are values of the `FieldType` enum imported from `react-declarative`.

---

## Input fields

### FieldType.Text
MUI TextField. Most commonly used.
```ts
{
  type: FieldType.Text,
  name: 'phone',
  title: 'Phone',
  inputType: 'tel',                     // hint keyboard on mobile
  inputRows: 3,                         // multiline textarea
  inputAutocomplete: 'on',
  inputFormatterTemplate: '####-###-###',  // mask
  inputFormatterSymbol: '#',
  inputFormatterAllowed: /^[0-9]/,
  outlined: true,                       // outlined variant
  leadingIcon: PhoneIcon,
  trailingIcon: ClearIcon,
  trailingIconClick(value, data, payload, onValueChange, onChange) {
    onValueChange('');
  },
}
```

### FieldType.Combo
Single-select autocomplete dropdown.
```ts
{
  type: FieldType.Combo,
  name: 'gender',
  title: 'Gender',
  itemList: ['male', 'female', 'other'],        // or async function
  async itemList() { return await fetchOptions(); },
  async tr(value) { return translations[value]; }, // translate opaque keys
  defaultValue: 'male',
  freeSolo: true,   // allow values not in list
}
```

### FieldType.Items
Multi-select chip input (like Autocomplete with multiple). Value is `string[]`.
```ts
{
  type: FieldType.Items,
  name: 'tags',
  title: 'Tags',
  itemList: ['design', 'dev', 'pm'],
  freeSolo: true,
}
```

### FieldType.Complete
Free-text with autocomplete suggestions.
```ts
{
  type: FieldType.Complete,
  name: 'city',
  title: 'City',
  async tip(value) { return await searchCities(value); },
}
```

### FieldType.Choose
Read-only display that opens a custom modal picker via `choose` callback.
```ts
{
  type: FieldType.Choose,
  name: 'assignee',
  title: 'Assignee',
  async choose(data, payload) {
    return await openUserPickerDialog();  // returns selected value
  },
}
```

### FieldType.Dict
Searchable dictionary/lookup field backed by paginated `dictSearch`.
```ts
{
  type: FieldType.Dict,
  name: 'product',
  title: 'Product',
  async dictSearch(query) { return await searchProducts(query); },
  async dictOnAppend(query) { return await createProduct(query); },  // allow creating new
}
```

### FieldType.Tree
Hierarchical tree selector.
```ts
{
  type: FieldType.Tree,
  name: 'category',
  title: 'Category',
  itemTree: [
    { id: 'electronics', label: 'Electronics', children: [
      { id: 'phones', label: 'Phones' },
    ]},
  ],
}
```

### FieldType.File
File upload.
```ts
{
  type: FieldType.File,
  name: 'avatar',
  title: 'Avatar',
  fileAccept: 'image/jpeg,image/png',
  async upload(file) {
    const formData = new FormData();
    formData.append('file', file);
    const res = await fetch('/api/upload', { method: 'POST', body: formData });
    const { url } = await res.json();
    return url;  // stored as string value
  },
  async view(value) { window.open(value); },
}
```

### FieldType.Date / FieldType.Time
```ts
{ type: FieldType.Date, name: 'birthDate', title: 'Date of birth', defaultValue: datetime.currentDate() }
{ type: FieldType.Time, name: 'meetTime', title: 'Meeting time', defaultValue: datetime.currentTime() }
```

---

## Toggle fields

### FieldType.Checkbox
```ts
{ type: FieldType.Checkbox, name: 'agree', title: 'I agree to the terms' }
```

### FieldType.Switch
```ts
{
  type: FieldType.Switch,
  name: 'notifications',
  title: 'Email notifications',
  switchActiveLabel: 'Enabled',  // second label when on
  defaultValue: true,
}
```

### FieldType.YesNo
```ts
{ type: FieldType.YesNo, name: 'approved', title: 'Approved?' }
```

### FieldType.Radio
All radios with the same `name` form a group. Set unique `radioValue` on each.
```ts
{ type: FieldType.Radio, name: 'plan', radioValue: 'free',       title: 'Free',       columns: '4' }
{ type: FieldType.Radio, name: 'plan', radioValue: 'pro',        title: 'Pro',        columns: '4' }
{ type: FieldType.Radio, name: 'plan', radioValue: 'enterprise', title: 'Enterprise', columns: '4' }
```

---

## Numeric / Range fields

### FieldType.Slider
```ts
{
  type: FieldType.Slider,
  name: 'volume',
  minSlider: 0,
  maxSlider: 100,
  stepSlider: 5,
  defaultValue: 50,
  leadingIcon: VolumeDown,
  leadingIconClick(value, _data, _payload, onChange) { onChange(Math.max(0, Number(value) - 5)); },
  trailingIcon: VolumeUp,
  trailingIconClick(value, _data, _payload, onChange) { onChange(Math.min(100, Number(value) + 5)); },
}
```

### FieldType.Rating
```ts
{ type: FieldType.Rating, name: 'score', title: 'Score', defaultValue: 3 }
```

### FieldType.Progress
Read-only progress bar. Bind to same `name` as a Slider for live indicator.
```ts
{ type: FieldType.Progress, name: 'completion', maxPercent: 100, showPercentLabel: true }
```

---

## Display fields (no input)

### FieldType.Typography
```ts
{
  type: FieldType.Typography,
  typoVariant: 'h6',        // 'h1'–'h6', 'body1', 'body2', 'caption', etc.
  placeholder: 'Section title',
}
```

### FieldType.Line
Horizontal divider with optional label.
```ts
{ type: FieldType.Line, title: 'Personal details' }
{ type: FieldType.Line, lineTransparent: true }  // spacing without visible rule
```

### FieldType.Button
```ts
{
  type: FieldType.Button,
  title: 'Submit',
  buttonVariant: 'contained',   // 'text' | 'outlined' | 'contained'
  buttonSize: 'medium',
  buttonColor: 'primary',
  icon: SendIcon,
  click(name, data, payload, onValueChange, onChange) {
    submitForm(data);
  },
}
```

### FieldType.Icon
```ts
{
  type: FieldType.Icon,
  icon: InfoIcon,
  iconSize: 24,
  iconColor: 'primary',
  iconBackground: 'transparent',
}
```

---

## Special / utility fields

### FieldType.Component
Inject arbitrary React component. Receives full data + payload.
```ts
{
  type: FieldType.Component,
  element: ({ data, payload, onChange }) => (
    <div style={{ padding: 8 }}>
      Preview: <strong>{data.firstName} {data.lastName}</strong>
      <button onClick={() => onChange({ firstName: 'Jane' })}>Prefill</button>
    </div>
  ),
}
```

### FieldType.Init
Invisible field; `compute` runs once on mount to set initial derived values.
```ts
{
  type: FieldType.Init,
  name: 'computedField',
  compute: async (data, payload) => await deriveValue(data),
}
```

