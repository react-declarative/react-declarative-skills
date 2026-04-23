# Reactive Programming

Built-in reactive toolkit — no RxJS needed.

## Subject

Push-based emitter. Subscribers only receive values emitted **after** subscribing.

```typescript
import { Subject } from 'react-declarative';

const clickSubject = new Subject<MouseEvent>();

document.addEventListener('click', (e) => clickSubject.next(e));

const unsub = clickSubject.subscribe((e) => console.log(e.clientX));
unsub(); // cleanup
```

## BehaviorSubject

Stores the last value. New subscribers immediately receive the current value.

```typescript
import { BehaviorSubject } from 'react-declarative';

const userSubject = new BehaviorSubject<string | null>(null);
userSubject.next('alice');

userSubject.subscribe((user) => console.log(user)); // immediately 'alice'
console.log(userSubject.data); // synchronous read
```

## EventEmitter

Named-event pub/sub.

```typescript
import { EventEmitter } from 'react-declarative';
const emitter = new EventEmitter();
emitter.subscribe('login', (user) => console.log('logged in:', user));
emitter.emit('login', { name: 'alice' });
```

## React hooks

### useSubject

```tsx
const saveSubject = useSubject<void>();
useEffect(() => saveSubject.subscribe(() => save()), []);
<button onClick={() => saveSubject.next()}>Save</button>
```

### useChangeSubject

Turns a React value into a Subject that emits on every change.

```tsx
import { useChangeSubject, useSubscription } from 'react-declarative';

const AutoSaveForm = ({ data }) => {
  const dataChangeSubject = useChangeSubject(data);
  useSubscription(() => dataChangeSubject.subscribe((newData) => saveToServer(newData)));
  return <One fields={fields} handler={data} />;
};
```

### useSubscription

Wrapper over `useEffect` for subscriptions. Calls unsubscribe on unmount — no dependency array needed.

```tsx
const NotificationBell = () => {
  const [count, setCount] = useState(0);
  useSubscription(() => ioc.notificationService.countSubject.subscribe(setCount));
  return <span>{count}</span>;
};
```

## Source pipelines (MapReduce)

| Method | Purpose |
|---|---|
| `Source.join` | Merge multiple observers; emits when any emits |
| `Source.multicast` | Single upstream shared across all subscribers |
| `.reduce(fn, init)` | Accumulate values |
| `.filter(fn)` | Filter emissions |
| `.tap(fn)` | Side effect without changing the value |
| `.map(fn)` | Transform values |
| `Source.fromInterval(ms)` | Interval ticker |

### Example — face verification

```typescript
import { Source } from 'react-declarative';

const verifyCompleteEmitter = Source.multicast(() =>
  Source
    .join([captureStateEmitter, Source.fromInterval(1_000)])
    .reduce((acm, [{ state: isValid }]) => isValid ? acm + 1 : 0, 0)
    .tap((ticker) => { if (ticker === 1) mediaRecorder.beginCapture(); })
    .filter((ticker) => ticker === 3)
    .tap(() => mediaRecorder.endCapture())
);
```

## When to use what

- **Subject** — one-off event between unrelated components
- **BehaviorSubject** — shared state that must be read synchronously on mount
- **useChangeSubject** — react to prop/state changes without a useEffect dependency array
- **Source** — combine multiple streams with stateful logic
- **useState + useEffect** — local linear data flow
