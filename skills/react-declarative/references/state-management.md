# State Management

## handler — loading initial data

Accepts a plain object, sync fn, or async fn. Called on mount and on `reloadSubject` emission.

```tsx
<One fields={fields} handler={{ firstName: 'Jane' }} />

<One fields={fields}
  handler={async (payload) => fetch(`/api/users/${payload.userId}`).then(r => r.json())}
  payload={{ userId: '42' }} />
```

Alternative: `data` prop for controlled pattern. **Never mix `handler` and `data`.**

## onChange

`initial` is `true` on first emission after handler resolves — skip saves then.

```tsx
<One fields={fields} handler={fetchProfile}
  onChange={(data, initial) => { if (!initial) autoSave(data); }} />
```

Debounce for auto-save:
```tsx
const autoSave = useMemo(() => debounce(async (data) => {
  await fetch('/api/profile', { method: 'PUT', body: JSON.stringify(data) });
}, 800), []);

<One fields={fields} handler={fetchProfile} fieldDebounce={300} onChange={(data, initial) => { if (!initial) autoSave(data); }} />
```

## payload — external context

Does NOT trigger re-renders (by design). For reactive context use `context` prop instead.

```tsx
const fields: TypedField<ProfileData, AppPayload>[] = [{
  type: FieldType.Text, name: 'internalNote',
  isReadonly: (_data, payload) => payload.userRole !== 'admin',
}];

<One fields={fields} handler={fetchProfile} payload={{ userRole: currentUser.role }} />
```

## Per-field callbacks

### isInvalid — blocking validation
Must return `null` (valid) or a **string** (error). Never return `false` or `undefined`.

```typescript
{ type: FieldType.Text, name: 'email',
  isInvalid({ email }) {
    return /^[\w-.]+@([\w-]+\.)+[\w-]{2,4}$/g.test(email) ? null : 'Invalid email';
  } }

// Cross-field
{ type: FieldType.Text, name: 'to',
  isInvalid: ({ from, to }) =>
    from && to && parseInt(from) > parseInt(to) ? 'Must be > From' : null }

// Async (built-in debouncing — no need to debounce manually)
{ type: FieldType.Text, name: 'username',
  isInvalid: async ({ username }) => {
    if (!username) return 'Required';
    return await checkAvailability(username) ? 'Already taken' : null;
  } }
```

### isIncorrect — non-blocking warning
Shows warning but does not block submission.

### isVisible — show/hide
Returns `false` to remove from DOM. Value stays in data object.

```typescript
{ type: FieldType.Text, name: 'phoneNumber', isVisible: ({ contactMethod }) => contactMethod === 'phone' }
```

### isDisabled / isReadonly
```typescript
{ type: FieldType.Text, name: 'email', isDisabled: ({ locked }) => Boolean(locked) }
{ type: FieldType.Text, name: 'createdAt', isReadonly: () => true }
```

## compute — derived display values

Implies read-only. Value is NOT stored in data.

```typescript
{ type: FieldType.Text, name: 'fullName',
  compute: ({ firstName, lastName }) => [firstName, lastName].filter(Boolean).join(' ') }
```

## defaultValue vs handler

1. `handler` runs → returns initial data
2. `defaultValue` fills gaps where handler returned null/undefined/missing keys
3. First `onChange` fires with `initial: true`

```typescript
// handler returns { firstName: 'Jane' }
{ type: FieldType.Text, name: 'role',      defaultValue: 'viewer'  },  // used
{ type: FieldType.Text, name: 'firstName', defaultValue: 'Unknown' },  // NOT used

// defaultValue with payload
{ type: FieldType.Text, name: 'assignee', defaultValue: (payload) => payload.currentUsername }
```

## dirty flag — show errors immediately

```typescript
{ type: FieldType.Text, name: 'email', dirty: true, isInvalid({ email }) { ... } }
```

Or set `dirty` on `<One />` to apply to all fields (validate-on-submit pattern).

## Form-level invalidity / getAvailableFields

```tsx
<One fields={fields} invalidity={(name, msg) => setIsFormValid(false)} />

import { getAvailableFields } from 'react-declarative';
const visibleFields = getAvailableFields(fields, formData, payload);
const fieldNames = visibleFields.map(f => f.name); // strip hidden before submit
```
