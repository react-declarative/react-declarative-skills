# Installation & First Steps

## Install

```bash
npm install react-declarative
```

Peer dependencies (must be installed separately):

```bash
npm install react react-dom @mui/material @mui/icons-material @emotion/react @emotion/styled
```

Minimum versions: React 18+, MUI v5+.

## TypeScript setup

`react-declarative` ships full TypeScript types. No `@types/` package needed.

```json
// tsconfig.json — recommended
{
  "compilerOptions": {
    "strict": true,
    "moduleResolution": "bundler",
    "jsx": "react-jsx"
  }
}
```

## First form

```tsx
import { One, TypedField, FieldType } from 'react-declarative';

interface IContactData {
  firstName: string;
  lastName: string;
  email: string;
  role: string;
  active: boolean;
}

const fields: TypedField<IContactData>[] = [
  {
    type: FieldType.Group,
    fields: [
      { type: FieldType.Text, name: 'firstName', title: 'First name', columns: '6' },
      { type: FieldType.Text, name: 'lastName',  title: 'Last name',  columns: '6' },
    ],
  },
  { type: FieldType.Text,   name: 'email',  title: 'Email', inputType: 'email' },
  { type: FieldType.Combo,  name: 'role',   title: 'Role',  itemList: ['admin', 'user', 'viewer'] },
  { type: FieldType.Switch, name: 'active', title: 'Active' },
];

export const ContactForm = () => (
  <One<IContactData>
    fields={fields}
    handler={() => ({ firstName: '', lastName: '', email: '', role: 'user', active: true })}
    onChange={(data, initial) => { if (!initial) console.log(data); }}
  />
);
```

## First data grid

```tsx
import { ListTyped, IColumn, TypedField, FieldType } from 'react-declarative';

interface IContact {
  id: string;
  name: string;
  email: string;
  role: string;
}

const columns: IColumn<IContact>[] = [
  { type: FieldType.Text, field: 'name',  headerName: 'Name',  width: 200 },
  { type: FieldType.Text, field: 'email', headerName: 'Email', width: 250 },
  { type: FieldType.Text, field: 'role',  headerName: 'Role',  width: 100 },
];

const filters: TypedField[] = [
  { type: FieldType.Text,  name: 'search', title: 'Search' },
  { type: FieldType.Combo, name: 'role',   title: 'Role', itemList: ['admin', 'user', 'viewer'] },
];

export const ContactList = () => (
  <ListTyped<IContact>
    columns={columns}
    filters={filters}
    handler={async ({ search, role }) => {
      const rows = await api.getContacts({ search, role });
      return { rows, total: rows.length };
    }}
  />
);
```

## Mantine UI skin (optional)

Replace all MUI inputs with Mantine equivalents — schema unchanged:

```bash
npm install react-declarative-mantine @mantine/core @mantine/hooks
```

```tsx
import { MantineSlotFactory } from 'react-declarative-mantine';
import { One } from 'react-declarative';

export const App = () => (
  <MantineSlotFactory>
    <One fields={fields} handler={handler} />
  </MantineSlotFactory>
);
```

Preview: https://react-declarative-mantine.github.io/

## CRA / Vite / Next.js notes

- **Vite**: works out of the box, no extra config
- **CRA**: works out of the box
- **Next.js**: use `'use client'` directive on any component that imports from `react-declarative`; SSR is not supported

## Verify installation

```tsx
import { One, FieldType } from 'react-declarative';

// Render a single text field — if this works, everything is installed correctly
<One
  fields={[{ type: FieldType.Text, name: 'test', title: 'Test field' }]}
  handler={() => ({ test: 'hello' })}
/>
```
