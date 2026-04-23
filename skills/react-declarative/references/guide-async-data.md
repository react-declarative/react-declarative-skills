# Guide: Async Data Patterns

React-declarative ships a cohesive set of primitives for every async data pattern: loading initial form data, preventing duplicate submissions, queuing actions, tracking batch progress, and fetching data inside hooks.

## Loading form data with `handler`

```tsx
// Form — async handler
<One
  fields={fields}
  handler={async () => {
    const data = await fetchUserProfile(userId);
    return data;
  }}
/>

// List — must return { rows, total }
<ListTyped
  columns={columns}
  filters={filters}
  handler={async ({ search, sort, pagination, filterData }) => {
    const { items, count } = await fetchUsers({ q: search, ...filterData });
    return { rows: items, total: count };
  }}
/>
```

## `useSinglerunAction` — prevent duplicate calls

Only one invocation runs at a time. Concurrent calls while loading are silently dropped.

```tsx
const { execute, loading, error } = useSinglerunAction(async () => {
  await saveForm(formData);
});

<button disabled={loading} onClick={execute}>
  {loading ? 'Saving...' : 'Save'}
</button>
```

## `useQueuedAction` — ordered async execution

Each call waits for the previous to finish — no drops, no cancellations. Use for WebSocket events where ordering matters.

```tsx
const { execute } = useQueuedAction(
  async ({ type, payload }) => {
    if (type === 'create-action') await createRecord(payload);
    if (type === 'update-action') await updateRecord(payload);
    if (type === 'remove-action') await deleteRecord(payload);
  },
  {
    onLoadStart: () => setAppbarLoader(true),
    onLoadEnd: () => setAppbarLoader(false),
  }
);

// execute.cancel() — abandon current item
// execute.clear() — drain entire queue
useEffect(() => kanbanService.createSubject.subscribe(execute), []);
```

## `useAsyncProgress` — batch processing with progress

Processes an array of items one at a time, reports 0–100%.

```tsx
const { execute } = useAsyncProgress(
  async ({ data }) => { await createContact(data); },
  {
    onProgress: (percent) => setProgress(percent),
    onError: (errors) => setErrors(errors),
    onEnd: (isOk) => { if (isOk) history.replace('/report'); },
  }
);

// Each item: { label: string, data: any }
execute(rows.map((row, idx) => ({ data: row, label: `Row №${idx + 2}` })));

<LinearProgress variant="determinate" value={progress} />
```

## `useAsyncValue` — async data in components

Fetches on mount, re-fetches when `deps` change.

```tsx
const [
  data,                           // Data | null
  { loading, error, execute },    // manual re-fetch
  setData,                        // optimistic update
  { waitForResult, data$ }        // advanced utilities
] = useAsyncValue(
  async () => await fetchUserProfile(userId),
  { deps: [userId] }
);

// Optimistic update after mutation (no refetch):
const handleSave = async (changes) => {
  const updated = await saveUserProfile(userId, changes);
  setData(updated);
};
```

## `useAsyncAction` — general async with cancellation

Cancels previous in-flight call when `execute` is called again (last one wins).

```tsx
const { loading, error, execute } = useAsyncAction(
  async (userId: string) => await fetchUserProfile(userId),
  { fallback: (e) => console.error('Failed:', e) }
);
```

## `ActionButton` / `ActionIcon`

Show a loading indicator while async `onClick` is pending — no extra state needed.

```tsx
<ActionButton onClick={async () => { await saveChanges(formData); }}>
  Save changes
</ActionButton>

<ActionIcon onClick={async () => { await uploadFile(selectedFile); }}>
  <CloudUpload />
</ActionIcon>
```

## Choosing the right primitive

| Situation | Use |
|-----------|-----|
| Load initial form data | `handler` prop on `<One />` |
| Load list data with pagination | `handler` on `<ListTyped />` returning `{ rows, total }` |
| Prevent double-submit | `useSinglerunAction` |
| Real-time ordered updates | `useQueuedAction` |
| Bulk import with progress bar | `useAsyncProgress` |
| Fetch remote data in a component | `useAsyncValue` |
| General async with cancellation | `useAsyncAction` |
| Button that shows spinner while async runs | `ActionButton` / `ActionIcon` |
