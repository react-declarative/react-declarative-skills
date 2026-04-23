# Guide: Form Validation

Validation lives entirely inside the field schema — no separate validation library needed.

## `isInvalid` — blocking validation

Receives the full form data object. Return `null` for valid, a string for an error message.

```tsx
{
  type: FieldType.Text,
  name: 'email',
  title: 'Email',
  isInvalid({ email }) {
    const expr = /^[\w-.]+@([\w-]+\.)+[\w-]{2,4}$/g;
    return expr.test(email) ? null : 'Invalid email address';
  },
}
```

> `isInvalid` receives the **whole data object** — destructure what you need.

## Cross-field validation

```tsx
const fields: TypedField[] = [
  {
    type: FieldType.Text, name: 'from', title: 'From', defaultValue: '50',
    isInvalid: ({ from, to }) => {
      if (!from || !to) return null;
      return parseInt(from) > parseInt(to) ? 'From must not exceed To' : null;
    },
  },
  {
    type: FieldType.Text, name: 'to', title: 'To', defaultValue: '150',
    isInvalid: ({ from, to }) => {
      if (!from || !to) return null;
      return parseInt(from) > parseInt(to) ? 'To must not be less than From' : null;
    },
  },
];
```

## Async validation

`isInvalid` can be `async`. Built-in debouncing prevents hammering the server on every keystroke.

```tsx
{
  type: FieldType.Text,
  name: 'username',
  title: 'Username',
  isInvalid: async ({ username }) => {
    if (!username) return 'Username is required';
    const taken = await checkUsernameAvailability(username);
    return taken ? 'This username is already taken' : null;
  },
}
```

## `isIncorrect` — non-blocking warning

Shows a warning but does not block submission.

```tsx
{
  type: FieldType.Text,
  name: 'description',
  isIncorrect: ({ description }) =>
    description && description.length < 20
      ? 'Descriptions are typically at least 20 characters'
      : null,
}
```

## `validation.required` shorthand

Use for simple required checks; use `isInvalid` for anything more complex.

```tsx
{
  type: FieldType.Text,
  name: 'firstName',
  title: 'First name',
  validation: { required: true },
}
```

## `dirty` flag — show errors immediately

By default a field waits for focus/blur before showing errors. `dirty: true` skips this.

```tsx
{ type: FieldType.Text, name: 'email', dirty: true, isInvalid({ email }) { /* ... */ } }
```

Set `dirty` on `<One />` to apply to all fields (validate-on-submit pattern):

```tsx
<One fields={fields} dirty handler={fetchData} />
```

## Reacting to form invalidity

```tsx
// Field-level
{
  type: FieldType.Text,
  name: 'email',
  isInvalid({ email }) { /* ... */ },
  invalidity(name, errorMessage, payload) {
    console.log(`${name} is invalid: ${errorMessage}`);
  },
}

// Form-level
<One
  fields={fields}
  handler={() => initialData}
  invalidity={(name, errorMessage) => setIsFormValid(false)}
/>
```

## `getAvailableFields` — strip hidden fields before submit

```tsx
import { getAvailableFields } from 'react-declarative';

const visibleFields = getAvailableFields(fields, formData, payload);
const fieldNames = visibleFields.map((f) => f.name);
// use fieldNames to strip hidden fields before sending to API
```
