# Guide: Conditional Fields — Visibility, Disabled, RBAC

Three orthogonal callbacks control field appearance. All receive `(data, payload)`.

## `isVisible` — show/hide based on form state

Returns `true` to show, `false` to remove from DOM. Value is preserved in data while hidden.

```tsx
const fields: TypedField[] = [
  {
    type: FieldType.Combo,
    name: 'contactMethod',
    title: 'Preferred contact method',
    itemList: ['email', 'phone', 'post'],
  },
  {
    type: FieldType.Text,
    name: 'phoneNumber',
    title: 'Phone number',
    isVisible({ contactMethod }) { return contactMethod === 'phone'; },
  },
  {
    type: FieldType.Text,
    name: 'emailAddress',
    title: 'Email address',
    isVisible({ contactMethod }) { return contactMethod === 'email'; },
  },
];
```

## `isDisabled` — grey out and block input

```tsx
{
  type: FieldType.Text,
  name: 'email',
  isDisabled({ disabled }) { return disabled; },
}
```

Use static `disabled: true` when the condition never changes at runtime.

## `isReadonly` — read-only display (not greyed out)

```tsx
{ type: FieldType.Text, name: 'createdAt', isReadonly: () => true }
```

For computed values use `compute` instead — implies readonly and recalculates automatically:

```tsx
{
  type: FieldType.Text,
  name: 'fullName',
  compute: ({ firstName, lastName }) => `${firstName} ${lastName}`,
}
```

## `FieldType.Condition` — gate entire sections

Evaluates a predicate and renders its `fields` children only when truthy. Removes entire subtree from DOM when false (unlike `isVisible` which only hides).

```tsx
{
  type: FieldType.Condition,
  condition: async ({ accountType }, payload) => accountType === 'business',
  conditionLoading: () => <CircularProgress size={20} />,
  conditionElse: () => <p>Switch to a business account to see these fields.</p>,
  fields: [
    { type: FieldType.Text, name: 'companyName', title: 'Company name' },
    { type: FieldType.Text, name: 'vatNumber', title: 'VAT number' },
  ],
}
```

**Key difference:** `isVisible` hides the field but value stays in data. `FieldType.Condition` removes the entire subtree from DOM.

## `hidden` prop — static RBAC / feature-flag gates

Evaluated once per render using `payload`. Use for things that don't change during form interaction.

```tsx
{
  type: FieldType.Text,
  name: 'salary',
  title: 'Salary',
  hidden: ({ payload }) => !['hr', 'admin'].includes(payload.userRole),
}

<One fields={fields} handler={fetchEmployee} payload={{ userRole: currentUser.role }} />
```

## Responsive hiding — per device size

```tsx
{
  type: FieldType.Text,
  name: 'detailedNotes',
  phoneHidden: true,     // hidden on phones
  tabletHidden: false,
  desktopHidden: false,
}
```

## `payload` — passing external context

`payload` flows into every callback (`isVisible`, `isDisabled`, `isInvalid`, `hidden`, etc.). Use it for user roles and feature flags — it does NOT trigger re-renders (use `context` prop for reactive context).

```tsx
<One
  fields={fields}
  handler={() => initialData}
  payload={{ userRole: 'admin', features: new Set(['show-phone-number']) }}
/>

// In a field:
{
  type: FieldType.Button,
  title: 'Delete record',
  isVisible: (data, payload) => payload.userRole === 'admin',
}
```
