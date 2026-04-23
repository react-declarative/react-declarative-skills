# Hooks Reference

---

## Async Hooks

All async hooks return `{ execute, loading, error }` unless noted otherwise. `loading` is `true` while the action runs.

---

### useSinglerunAction
**Only one invocation can run at a time.** Concurrent calls while loading are silently dropped.
Use for: upload buttons, submit buttons, any action where double-submit causes problems.

```ts
const { execute, loading, error } = useSinglerunAction(
  async (payload?) => {
    return await doSomething(payload);
  },
  {
    onLoadStart: () => void,
    onLoadEnd: (isOk: boolean) => void,
    fallback: (e: Error) => void,
    throwError: boolean,   // default false; true = re-throws from execute()
  }
);

// execute.clear() resets the single-run lock
```

### useQueuedAction
**Queues calls in order.** No calls are dropped — each waits for the previous to finish.
Use for: WebSocket events, real-time state updates where ordering matters.

```ts
const { execute, loading } = useQueuedAction(
  async ({ type, payload }) => {
    if (type === 'create') await api.create(payload);
    if (type === 'update') await api.update(payload);
  },
  { onLoadStart: () => setAppbarLoader(true), onLoadEnd: () => setAppbarLoader(false) }
);
// execute.cancel() — abandon current item
// execute.clear() — drain the entire queue
```

### useAsyncAction
**Cancels previous in-flight call** when `execute` is called again. Latest request wins.
Use for: live search, on-demand data loads, any "last one wins" pattern.

```ts
const { execute, loading, error } = useAsyncAction(
  async (payload?) => await fetchData(payload),
  { fallback: (e) => toast.error(e.message) }
);
```

### useAsyncProgress
**Processes an array of items sequentially** and tracks 0–100% progress.
Use for: bulk imports, multi-step operations with a progress bar.

```ts
const { execute, loading, progress, errors, label } = useAsyncProgress(
  async ({ label, data }) => {
    await api.createContact(data);
  },
  {
    delay: 0,                       // min ms per item
    onBegin: () => void,
    onEnd: (isOk: boolean) => void,
    onFinish: (data, errors, results) => void,
    onError: (errors) => boolean,   // return false to abort; true/undefined to continue
    onProgress: (percent: number) => void,
  }
);

// Usage:
execute(rows.map((row, i) => ({ label: `Row ${i+1}`, data: row })));

// In JSX:
<LinearProgress variant="determinate" value={progress} />
```

### useAsyncValue
Fetches data on mount (re-fetches when `deps` change). Like `useState + useEffect` for async data.

```ts
const [
  data,            // Data | null
  { execute, loading, error },   // manual re-fetch
  setData,         // optimistic update setter
  { waitForResult, data$ }       // utilities
] = useAsyncValue(
  async () => await api.getUser(userId),
  { deps: [userId], fallback: (e) => console.error(e) }
);

// Optimistic update after mutation (no refetch):
const handleSave = async (changes) => {
  const updated = await api.saveUser(userId, changes);
  setData(updated);
};

// Await first non-null value:
const user = await waitForResult();
```

### usePreventAction
**Counting semaphore** — tracks multiple in-flight operations. While any one runs, `loading` is `true`.
Use for: toolbars with multiple buttons that should all disable while any action is pending.

```ts
const { handleLoadStart, handleLoadEnd, loading } = usePreventAction();

// Attach to multiple ActionButton components:
<ActionButton disabled={loading} onLoadStart={handleLoadStart} onLoadEnd={handleLoadEnd} onClick={async () => ...}>
  Action 1
</ActionButton>
<ActionButton disabled={loading} onLoadStart={handleLoadStart} onLoadEnd={handleLoadEnd} onClick={async () => ...}>
  Action 2
</ActionButton>
```

---

## Navigation Hooks

### useRouteParams
Returns URL params from the first matching route. Reactively updates on location change.

```ts
const params = useRouteParams<{ userId: string }>(routes, history);
// Returns null when no route matches
// routes must have a path property using path-to-regexp syntax
```

### useRouteItem
Returns the entire matching route object (useful when routes carry metadata).

```ts
interface AppRoute extends ISwitchItem { title: string; requiresAuth: boolean; }
const route = useRouteItem<Record<string, any>, AppRoute>(routes, history);
```

### useLocalHistory
Creates an in-memory history instance, optionally synced with a parent.

```ts
const { history: localHistory } = useLocalHistory({
  pathname: '/step/1',         // initial path
  history: parentHistory,      // optional parent to sync with
  onNavigate: (update) => void,
});
```

## Routing utility functions

```ts
// Match URL against pattern, return params or null
parseRouteUrl('/users/:id', '/users/42')   // { params: { id: '42' } }
parseRouteUrl('/users/:id', '/posts/42')   // null

// Same as useRouteParams but synchronous / outside React
getRouteParams(routes, pathname)           // { userId: '42' } | null

// Build URL from template + params
toRouteUrl('/users/:userId/posts/:postId', { userId: '42', postId: '7' })
// → '/users/42/posts/7'
```

---

## Reactive Subject Hooks

### useSubject
Creates a stable `Subject<T>` across renders.

```ts
const subject = useSubject<void>();
subject.next();  // emit
subject.subscribe((v) => doSomething(v));  // returns unsubscribe fn
```

### useBehaviorSubject
Stable `BehaviorSubject<T>` with a stored current value.

```ts
const statusSubject = useBehaviorSubject<'idle' | 'loading' | 'done'>('idle');
statusSubject.next('loading');
statusSubject.data  // current value
```

### useChangeSubject
Converts a React value into a subject that emits when the value changes.

```ts
const dataChangeSubject = useChangeSubject(formData);
useSubscription(() => dataChangeSubject.subscribe((newData) => autoSave(newData)));
```

### useSubscription
`useEffect` wrapper for subjects — auto-unsubscribes on unmount, no dependency array needed.

```ts
useSubscription(() =>
  someSubject.subscribe((value) => {
    setState(value);
  })
);
```

---

## ActionButton / ActionIcon

Components that show a loading indicator while their async `onClick` is pending — no extra state needed.

```tsx
import { ActionButton, ActionIcon } from 'react-declarative';

<ActionButton onClick={async () => { await saveChanges(data); }}>
  Save changes
</ActionButton>

<ActionIcon onClick={async () => { await uploadFile(file); }}>
  <CloudUpload />
</ActionIcon>
```
