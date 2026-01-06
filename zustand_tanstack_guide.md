# Zustand & TanStack Query Study Guide

## Table of Contents

1. [Understanding State Management](#understanding-state-management)
2. [Zustand Fundamentals](#zustand-fundamentals)
3. [TanStack Query Fundamentals](#tanstack-query-fundamentals)
4. [Real-World Patterns](#real-world-patterns)
5. [Best Practices](#best-practices)
6. [Common Pitfalls](#common-pitfalls)
7. [Practice Exercises](#practice-exercises)

---

## Understanding State Management

### The Theoretical Foundation

#### What is State?

State represents **data that changes over time** in your application. Every piece of state has three fundamental characteristics:

1. **Source of Truth** - Where does the data originate?
2. **Lifecycle** - How long should the data persist?
3. **Scope** - Who needs access to this data?

Understanding these characteristics determines which state management solution to use.

### The Two Types of State

**Client State (UI State)**

- Exists only in the browser
- Examples: modal open/closed, form inputs, theme preference
- Managed by: Zustand, Redux, Context API
- **Source**: User interactions
- **Lifecycle**: Session or shorter
- **Scope**: Usually component-specific or app-wide UI

**Server State (Remote State)**

- Lives on the server, cached locally
- Examples: user data, todos, products
- Managed by: TanStack Query, SWR, Apollo Client
- **Source**: Backend API
- **Lifecycle**: Until invalidated or stale
- **Scope**: Shared across app, potentially outdated

### The Core Problem: Asynchronous Data Synchronization

Server state introduces complexity because it's **asynchronous** and **potentially stale**. Consider these challenges:

1. **Cache Invalidation** - When is cached data "old enough" to refetch?
2. **Race Conditions** - What if two requests for the same data are in flight?
3. **Request Deduplication** - Should we fetch the same data twice?
4. **Background Updates** - How do we keep data fresh without blocking UI?
5. **Optimistic Updates** - Can we predict server responses for better UX?

Traditional client state tools (useState, Redux) don't solve these problems elegantly.

### Why This Distinction Matters

```typescript
// ❌ Common mistake: Managing server data in client state
const [users, setUsers] = useState([]);
const [loading, setLoading] = useState(false);

useEffect(() => {
  setLoading(true);
  fetchUsers().then(data => {
    setUsers(data);
    setLoading(false);
  });
}, []); // When do you refetch? How do you handle errors?

// ✅ Better: Let TanStack Query handle server state
const { data: users, isLoading } = useQuery({
  queryKey: ['users'],
  queryFn: fetchUsers
});
```

**What's happening under the hood?**

When you use TanStack Query, it manages:

- **Caching layer** - Stores fetched data in memory
- **Staleness tracking** - Knows when data might be outdated
- **Background refetching** - Updates stale data automatically
- **Request deduplication** - Combines simultaneous identical requests
- **Garbage collection** - Removes unused cache entries

This is why TanStack Query's API is so simple—it's doing complex work behind the scenes.

---

## Zustand Fundamentals

### Theoretical Foundation: The Observer Pattern

Zustand implements the **Observer Pattern**, where components "subscribe" to state changes and react when the observed state updates.

#### Core Concepts

**1. Single Source of Truth**
All state lives in one centralized store. Any component can access it without prop drilling.

**2. Immutable Updates**
State changes create new objects rather than mutating existing ones. This enables React to detect changes through reference equality (`oldState !== newState`).

**3. Selective Subscriptions**
Components subscribe to specific slices of state. Only components subscribed to changed state will re-render.

```typescript
// Component A subscribes to count
const count = useStore(state => state.count);

// Component B subscribes to user
const user = useStore(state => state.user);

// When count changes, only Component A re-renders
```

**Why does this matter?**
In React Context, all consumers re-render when any value changes. Zustand's selector pattern prevents unnecessary re-renders by tracking which components care about which data.

### Core Concept: The Store

Zustand creates a **reactive store** that components can subscribe to.

```typescript
import { create } from 'zustand';

// Define your store
const useStore = create((set) => ({
  // State
  count: 0,
  user: null,
  
  // Actions (functions that modify state)
  increment: () => set((state) => ({ count: state.count + 1 })),
  decrement: () => set((state) => ({ count: state.count - 1 })),
  setUser: (user) => set({ user }),
  reset: () => set({ count: 0, user: null })
}));
```

**Understanding the `set` function:**

```typescript
// Functional form - receives current state
set((state) => ({ count: state.count + 1 }))

// Object form - merges with existing state
set({ count: 5 })

// Replace entire state
set(() => ({ count: 0, user: null }), true) // second param = replace
```

Zustand **automatically merges** state updates. You don't need to spread the entire state:

```typescript
// You write this:
set({ count: 5 })

// Zustand does this internally:
set((state) => ({ ...state, count: 5 }))
```

### Using the Store

**Theory Question:** Why do we need to "select" specific parts of state instead of just accessing the whole store?

**Answer:** React re-renders when references change. If you subscribe to the entire store, your component re-renders on EVERY state change, even changes you don't care about.

```typescript
function Counter() {
  // ❌ Anti-pattern: Subscribe to everything
  const store = useStore();
  // Problem: Re-renders when count, user, settings, or ANYTHING changes
  
  return <p>{store.count}</p>;
}
```

**What's happening:**

1. Component subscribes to entire store object
2. ANY state change creates a new store object
3. React sees `oldStore !== newStore`
4. Component re-renders (even if count didn't change)

**The Fix:** Selective subscription using a selector function:

```typescript
function Counter() {
  // ✅ Correct: Subscribe only to count
  const count = useStore((state) => state.count);
  const increment = useStore((state) => state.increment);
  
  return (
    <div>
      <p>{count}</p>
      <button onClick={increment}>+1</button>
    </div>
  );
}
```

**What's happening now:**

1. Zustand calls your selector: `(state) => state.count`
2. Returns the value: `5`
3. Stores the result and watches for changes
4. On next state update, runs selector again
5. Compares old result (`5`) with new result (`5`)
6. If same → no re-render! If different (`6`) → re-render

**Why actions need selection too:**

```typescript
const increment = useStore((state) => state.increment);
```

**Theory:** Functions in JavaScript are compared by reference. The `increment` function reference never changes, so selecting it won't cause re-renders. But we select it to access it in a type-safe way.

**Without selection:**

```typescript
// ❌ Can't do this - useStore needs a selector
const increment = useStore.increment; // TypeScript error
```

**Mental Model:** Think of selectors as "subscriptions". You're telling Zustand: "Alert me only when THIS specific piece of data changes."

### Key Feature: Selector Optimization

Let's dive deeper into what makes selectors powerful:

```typescript
// Example 1: Primitive value
const count = useStore((state) => state.count);
// Compares with ===
// Old: 5, New: 5 → No render
// Old: 5, New: 6 → Render ✓

// Example 2: Derived value
const isPositive = useStore((state) => state.count > 0);
// Old: true, New: true → No render
// Old: false, New: true → Render ✓

// Example 3: Object property (CAREFUL!)
const userName = useStore((state) => state.user?.name);
// Old: "Alice", New: "Alice" → No render
// Old: "Alice", New: "Bob" → Render ✓
```

**Common Pitfall:** Selecting objects

```typescript
// ❌ Problem: Creates new object every time
const user = useStore((state) => ({ 
  name: state.user.name,
  email: state.user.email 
}));
// { name: "Alice" } !== { name: "Alice" }  (different references!)
// Result: Re-renders every time Zustand checks, even if values unchanged

// ✅ Solution 1: Select the object directly
const user = useStore((state) => state.user);
// Same object reference unless user actually changes

// ✅ Solution 2: Use shallow equality
import { shallow } from 'zustand/shallow';

const user = useStore(
  (state) => ({ name: state.user.name, email: state.user.email }),
  shallow
);
// Compares object properties instead of reference
```

**Why does `shallow` exist?**

JavaScript's default comparison (`===`) checks if two values point to the same memory location:

```javascript
const a = { name: "Alice" };
const b = { name: "Alice" };
console.log(a === b); // false (different objects in memory)

const c = a;
console.log(a === c); // true (same object reference)
```

Zustand's `shallow` helper compares one level deep:

```javascript
shallow(
  { name: "Alice", age: 30 },
  { name: "Alice", age: 30 }
); // true (same properties and values)
```

**When to use shallow:**

- Selecting multiple primitive values into an object
- You need specific properties but not the whole object
- You want to avoid re-renders from unrelated object changes

**When NOT to use shallow:**

- Selecting a single primitive value (unnecessary overhead)
- Selecting an object that's already in state (use direct selection)
- Need deep equality (shallow only checks one level)

### Real-World Example: Auth Store

```typescript
interface AuthStore {
  user: User | null;
  token: string | null;
  isAuthenticated: boolean;
  login: (email: string, password: string) => Promise<void>;
  logout: () => void;
  updateProfile: (data: Partial<User>) => void;
}

const useAuthStore = create<AuthStore>((set, get) => ({
  user: null,
  token: null,
  isAuthenticated: false,
  
  login: async (email, password) => {
    const { user, token } = await loginAPI(email, password);
    set({ user, token, isAuthenticated: true });
  },
  
  logout: () => {
    set({ user: null, token: null, isAuthenticated: false });
    localStorage.removeItem('token');
  },
  
  updateProfile: (data) => {
    const currentUser = get().user;
    if (currentUser) {
      set({ user: { ...currentUser, ...data } });
    }
  }
}));
```

### Middleware: Persist

**The Problem:** When you refresh the page, all JavaScript state disappears.

**Why this happens:**

```text
User sets theme to "dark"
  ↓
Stored in JavaScript memory (RAM)
  ↓
User refreshes page
  ↓
Browser clears memory and restarts
  ↓
Theme is back to "light" (default)
```

**Without Persist Middleware:**

```typescript
const usePreferencesStore = create((set) => ({
  theme: 'light',
  language: 'en',
  setTheme: (theme) => set({ theme }),
}));

// Problem: Every refresh loses user preferences
// User has to re-select theme every visit
```

**With Persist Middleware:**

```typescript
import { persist } from 'zustand/middleware';

const usePreferencesStore = create(
  persist(
    (set) => ({
      theme: 'light',
      language: 'en',
      setTheme: (theme) => set({ theme }),
      setLanguage: (language) => set({ language })
    }),
    {
      name: 'user-preferences', // localStorage key
    }
  )
);
```

**What's happening under the hood:**

1. **On state change:**

```javascript
// User calls setTheme('dark')
set({ theme: 'dark' })
  ↓
// Persist middleware intercepts
  ↓
// Saves to localStorage
localStorage.setItem('user-preferences', JSON.stringify({
  state: { theme: 'dark', language: 'en' },
  version: 0
}))
  ↓
// Updates Zustand state (component re-renders)
```

2. **On page load:**

```javascript
// Browser starts
  ↓
// Persist middleware checks localStorage
const saved = localStorage.getItem('user-preferences')
  ↓
// If found, restore state
if (saved) {
  const { state } = JSON.parse(saved);
  // Initial state is now { theme: 'dark', language: 'en' }
}
```

**Why do we need a middleware for this? Can't we just use localStorage directly?**

You could, but you'd have to manually:

```typescript
// ❌ Manual approach (tedious and error-prone)
const useStore = create((set) => ({
  theme: localStorage.getItem('theme') || 'light',
  setTheme: (theme) => {
    localStorage.setItem('theme', theme);  // Save
    set({ theme });  // Update state
  }
}));

// Problems:
// 1. Have to remember localStorage.setItem in every action
// 2. Have to parse/stringify manually
// 3. Have to handle initialization for each field
// 4. No automatic synchronization across tabs
// 5. No version migration support
```

```typescript
// ✅ With persist middleware (automatic)
const useStore = create(
  persist(
    (set) => ({
      theme: 'light',
      setTheme: (theme) => set({ theme })
      // Middleware handles ALL localStorage operations
    }),
    { name: 'user-preferences' }
  )
);
```

**Advanced: Partial persistence**

Sometimes you don't want to persist everything:

```typescript
const useStore = create(
  persist(
    (set) => ({
      // Persistent data
      theme: 'light',
      language: 'en',
      
      // Temporary data (don't persist)
      isMenuOpen: false,
      currentPage: 1,
      
      // Actions
      setTheme: (theme) => set({ theme }),
      toggleMenu: () => set((s) => ({ isMenuOpen: !s.isMenuOpen }))
    }),
    {
      name: 'user-preferences',
      partialPersist: (state) => ({
        // Only save these
        theme: state.theme,
        language: state.language
        // isMenuOpen and currentPage won't be saved
      })
    }
  )
);
```

**Why partial persistence?**
- Menu state should reset on page load (better UX)
- Current page might not exist anymore (navigation state)
- Sensitive data shouldn't be in localStorage (security)

**Storage options:**

```typescript
import { createJSONStorage } from 'zustand/middleware';

// Default: localStorage (persists forever)
persist(
  (set) => ({...}),
  { 
    name: 'my-store',
    storage: createJSONStorage(() => localStorage)
  }
)

// Alternative: sessionStorage (cleared when tab closes)
persist(
  (set) => ({...}),
  { 
    name: 'my-store',
    storage: createJSONStorage(() => sessionStorage)
  }
)
```

**When to use each:**

| Storage Type | Lifetime | Use Case |
|--------------|----------|----------|
| localStorage | Permanent (until cleared) | User preferences, settings, auth tokens |
| sessionStorage | Until tab closes | Temporary filters, form drafts, wizard state |
| No persistence | Until page refresh | UI state, temporary selections, hover states |

**Real-world example: Why persist matters**

Imagine an e-commerce site without persist:

```typescript
// User adds items to cart
// User accidentally closes tab
// User reopens site
// Cart is empty 😞
```

With persist:
```typescript
const useCartStore = create(
  persist(
    (set) => ({
      items: [],
      addItem: (item) => set((s) => ({ 
        items: [...s.items, item] 
      }))
    }),
    { name: 'shopping-cart' }
  )
);

// User adds items → Saved to localStorage
// User closes tab → Data still in localStorage
// User reopens site → Cart restored 😊
```

**Important limitation:**

localStorage only stores **strings**, so persist middleware:
1. Stringifies your state with `JSON.stringify()`
2. Parses it back with `JSON.parse()`

This means:
```typescript
// ✅ Works: JSON-serializable data
{ theme: 'dark', count: 5, user: { name: 'Alice' } }

// ❌ Doesn't work: Functions, Dates, undefined
{ 
  theme: 'dark',
  createdAt: new Date(),  // Becomes a string!
  callback: () => {}      // Lost!
}
```

**Solution for non-serializable data:**

```typescript
const useStore = create(
  persist(
    (set) => ({
      // Serializable (will persist)
      userId: null,
      theme: 'dark',
      
      // Non-serializable (won't persist, that's OK)
      currentDate: new Date(),
      
      // Actions (functions never persist anyway)
      setUser: (id) => set({ userId: id })
    }),
    { name: 'app-state' }
  )
);

// On load, recreate non-serializable data
useEffect(() => {
  useStore.setState({ currentDate: new Date() });
}, []);
```

### Middleware: Immer (Easier Mutations)

**The Problem:** JavaScript requires immutable updates for React to detect changes.

**Why immutability is necessary:**

React compares state using reference equality:
```javascript
const oldState = { items: [1, 2, 3] };
const newState = oldState;

// React checks:
oldState === newState  // true → No re-render!

// React needs:
oldState !== newState  // false → Re-render!
```

**Without Immer:** You must manually create new objects/arrays

```typescript
const useCartStore = create((set) => ({
  items: [],
  
  addItem: (product) => set((state) => ({
    // Must spread to create new array
    items: [...state.items, product]
  })),
  
  updateQuantity: (id, quantity) => set((state) => ({
    // Must map to create new array
    items: state.items.map(item =>
      item.id === id 
        ? { ...item, quantity }  // Must spread to create new object
        : item
    )
  })),
  
  removeItem: (id) => set((state) => ({
    // Must filter to create new array
    items: state.items.filter(item => item.id !== id)
  }))
}));
```

**Why is this tedious?**

1. **Deep nesting gets complex:**
```typescript
// Without Immer: Updating nested data is painful
updateAddress: (userId, newAddress) => set((state) => ({
  users: state.users.map(user =>
    user.id === userId
      ? {
          ...user,
          profile: {
            ...user.profile,
            address: {
              ...user.profile.address,
              ...newAddress  // Finally, the actual update!
            }
          }
        }
      : user
  )
}))
```

2. **Easy to make mistakes:**

```typescript
// ❌ BUG: Forgot to spread - mutates directly!
updateQuantity: (id, quantity) => set((state) => {
  const item = state.items.find(i => i.id === id);
  item.quantity = quantity;  // Mutates original!
  return { items: state.items };  // Same reference!
})
```

3. **Verbose and hard to read:**

```typescript
// The "what" (set quantity) is buried in "how" (map, spread, ternary)
```

**With Immer Middleware:**

```typescript
import { immer } from 'zustand/middleware/immer';

const useCartStore = create(
  immer((set) => ({
    items: [],
    
    addItem: (product) => set((state) => {
      // Just push! Immer handles immutability
      state.items.push(product);
    }),
    
    updateQuantity: (id, quantity) => set((state) => {
      // Mutate directly! Immer creates a new object behind the scenes
      const item = state.items.find(i => i.id === id);
      if (item) item.quantity = quantity;
    }),
    
    removeItem: (id) => set((state) => {
      // Find index and splice
      const index = state.items.findIndex(i => i.id === id);
      if (index !== -1) state.items.splice(index, 1);
    })
  }))
);
```

**What Immer does under the hood:**

```javascript
// When you write:
state.items.push(product);

// Immer does:
1. Creates a "draft" copy of state
2. Tracks all your mutations on the draft
3. Produces a new immutable state with changes applied
4. Returns the new state to Zustand

// You write mutable code, but get immutable results!
```

**The Magic: Proxy Objects**

Immer uses JavaScript Proxies to intercept your mutations:

```javascript
const draft = new Proxy(originalState, {
  set(target, property, value) {
    // Intercept writes
    markAsModified(target);
    return Reflect.set(target, property, value);
  }
});

// When you write:
draft.count = 5;

// Proxy captures it and marks the path as changed
```

**Comparing approaches for nested updates:**

```typescript
// Without Immer (verbose)
updateAddress: (userId, newAddress) => set((state) => ({
  users: state.users.map(user =>
    user.id === userId
      ? {
          ...user,
          profile: {
            ...user.profile,
            address: { ...user.profile.address, ...newAddress }
          }
        }
      : user
  )
}))

// With Immer (clear intent)
updateAddress: (userId, newAddress) => set((state) => {
  const user = state.users.find(u => u.id === userId);
  if (user) {
    Object.assign(user.profile.address, newAddress);
  }
})
```

**When to use Immer:**

✅ **Use Immer when:**

- Complex nested state structures
- Many array operations (push, splice, sort)
- Deep object updates
- Team prefers mutable-looking code
- Coming from Redux Toolkit (which uses Immer)

❌ **Skip Immer when:**

- Simple, flat state
- Only primitive values
- Performance-critical code (Immer has small overhead)
- State updates are already simple

**Performance consideration:**

```typescript
// Immer has overhead (creates proxies, tracks changes)
// For 1000 simple updates:
Without Immer: ~1ms
With Immer:    ~3ms

// For complex nested updates:
Without Immer: ~5ms (manual spreading)
With Immer:    ~3ms (optimized internally)

// Conclusion: Immer is worth it for complex state
```

**Real-world example: Todo app**

```typescript
// Without Immer: Careful spreading required
const useTodoStore = create((set) => ({
  todos: [],
  
  toggleTodo: (id) => set((state) => ({
    todos: state.todos.map(todo =>
      todo.id === id
        ? { ...todo, completed: !todo.completed }
        : todo
    )
  })),
  
  addSubtask: (todoId, subtask) => set((state) => ({
    todos: state.todos.map(todo =>
      todo.id === todoId
        ? { ...todo, subtasks: [...todo.subtasks, subtask] }
        : todo
    )
  }))
}));

// With Immer: Natural mutations
const useTodoStore = create(
  immer((set) => ({
    todos: [],
    
    toggleTodo: (id) => set((state) => {
      const todo = state.todos.find(t => t.id === id);
      if (todo) todo.completed = !todo.completed;
    }),
    
    addSubtask: (todoId, subtask) => set((state) => {
      const todo = state.todos.find(t => t.id === todoId);
      if (todo) todo.subtasks.push(subtask);
    })
  }))
);
```

**Common pitfall: Mixing patterns**

```typescript
// ❌ DON'T: Mix immutable and mutable patterns
set((state) => {
  state.items.push(newItem);  // Mutable
  return { ...state };        // Immutable - CONFUSING!
})

// ✅ DO: Pick one style
// Option 1: Fully mutable (with Immer)
set((state) => {
  state.items.push(newItem);
  // No return needed with Immer
})

// Option 2: Fully immutable (without Immer)
set((state) => ({
  items: [...state.items, newItem]
}))
```

**Mental Model:**

Think of Immer as a **"time machine"** for your state:

1. Takes a snapshot of current state (draft)
2. Lets you modify the draft freely
3. Produces a new timeline (new state) based on changes
4. Original timeline (old state) remains unchanged

This is why you can write `state.items.push()` but still get immutable updates!

---

## TanStack Query Fundamentals

### Theoretical Foundation: Caching & Synchronization

TanStack Query solves a fundamental problem: **keeping client-side data synchronized with server-side truth**. This involves several computer science concepts:

#### 1. Caching Strategy: Stale-While-Revalidate

TanStack Query uses a **stale-while-revalidate** caching strategy:

```
User requests data
  ↓
Is data in cache?
  ↓ Yes
Return cached data immediately (even if stale)
  ↓
Is data stale?
  ↓ Yes
Fetch fresh data in background
  ↓
Update cache & UI when fresh data arrives
```

**Why this approach?**

- **Better UX**: Users see data instantly (cached)
- **Fresh data**: Background refetch ensures accuracy
- **Offline resilience**: App works even without network

Compare to traditional approaches:

- **No cache**: Every render fetches (slow, redundant)
- **Cache forever**: Fast but potentially wrong
- **Stale-while-revalidate**: Best of both worlds

#### 2. Request Deduplication

When multiple components request the same data simultaneously, TanStack Query makes **only one network request**:

```typescript
// Component A renders
useQuery({ queryKey: ['user', 1], queryFn: fetchUser });

// Component B renders (same query key)
useQuery({ queryKey: ['user', 1], queryFn: fetchUser });

// Result: Only ONE fetch happens, both components share the response
```

**How it works:**

1. First query starts, creates promise, stores in cache
2. Second query sees same key, returns same promise
3. When promise resolves, both subscribers receive data

This prevents the **thundering herd problem** where duplicate requests overwhelm the server.

#### 3. Query Keys as Cache Identity

Query keys use **structural equality** (deep comparison) rather than reference equality:

```typescript
// These are considered IDENTICAL keys
['user', { id: 1, role: 'admin' }]
['user', { id: 1, role: 'admin' }]

// Order matters in arrays
['user', 1] !== [1, 'user']

// Object key order doesn't matter
{ id: 1, role: 'admin' } === { role: 'admin', id: 1 }
```

**Design principle**: Query keys should be **deterministic**. Same parameters = same key = same cached data.

#### 4. Garbage Collection

TanStack Query automatically removes unused cache entries to prevent memory leaks:

```
Query becomes inactive (no components using it)
  ↓
Start timer (default: 5 minutes)
  ↓
If query still inactive when timer expires
  ↓
Remove from cache
```

You can configure this with `cacheTime` (now called `gcTime` in v5).

### Core Concept: Queries vs Mutations

The **CQRS pattern** (Command Query Responsibility Segregation) separates reads from writes:

**Queries (Read)**

- GET requests
- Should not modify server state
- Can be cached
- Automatically refetched
- Example: `useQuery`

**Mutations (Write)**

- POST, PUT, DELETE requests
- Modify server state
- Not cached (each mutation is unique)
- Run only when explicitly called
- Example: `useMutation`

This separation allows TanStack Query to optimize each differently:

- Queries focus on caching and synchronization
- Mutations focus on error handling and optimistic updates

### Core Concept: Query States

Every query exists in one of several states at any time:

```text
Initial Load Flow:
  isLoading: true   →   isSuccess: true
  isPending: true   →   isPending: false
  data: undefined   →   data: [results]

Background Refetch Flow:
  isLoading: false      (data already exists)
  isFetching: true   →  isFetching: false
  data: [old data]   →  data: [new data]

Error Flow:
  isLoading: true   →   isError: true
  isPending: true   →   isPending: false
  data: undefined   →   data: undefined
                        error: Error object
```

**Key distinction**: 

- `isLoading` = First time fetching (no data yet)
- `isFetching` = Any fetch (includes background updates)

### Setup

**Why do we need a QueryClient and Provider?**

**The Problem:** React components need to share the same cache.

Without a shared cache:

```typescript
// Component A
const { data } = useQuery({ queryKey: ['users'], queryFn: fetchUsers });
// Fetches users → stores in Component A's cache

// Component B  
const { data } = useQuery({ queryKey: ['users'], queryFn: fetchUsers });
// Doesn't know about Component A's cache → fetches AGAIN!

// Result: Duplicate requests, wasted bandwidth, inconsistent data
```

**The Solution:** Central cache that all components access

```typescript
// main.tsx or App.tsx
import { QueryClient, QueryClientProvider } from '@tanstack/react-query';
import { ReactQueryDevtools } from '@tanstack/react-query-devtools';

// 1. Create the cache
const queryClient = new QueryClient({
  defaultOptions: {
    queries: {
      staleTime: 1000 * 60 * 5, // 5 minutes
      retry: 3,
    },
  },
});

function App() {
  return (
    // 2. Provide cache to all children
    <QueryClientProvider client={queryClient}>
      <YourApp />
      <ReactQueryDevtools initialIsOpen={false} />
    </QueryClientProvider>
  );
}
```

**What's happening:**

```
QueryClientProvider creates a React Context
  ↓
Passes queryClient down the tree
  ↓
All useQuery hooks access the same queryClient
  ↓
Shared cache = No duplicate requests!
```

**Understanding defaultOptions:**

These are **global defaults** for all queries in your app:

```typescript
const queryClient = new QueryClient({
  defaultOptions: {
    queries: {
      // How long until data is considered "stale"
      staleTime: 1000 * 60 * 5,  // 5 minutes
      
      // How many times to retry failed requests
      retry: 3,
      
      // How long to wait before retrying
      retryDelay: (attemptIndex) => Math.min(1000 * 2 ** attemptIndex, 30000),
      
      // Keep data in cache even when no components use it
      cacheTime: 1000 * 60 * 30,  // 30 minutes (now called gcTime in v5)
      
      // Refetch on window focus
      refetchOnWindowFocus: true,
      
      // Refetch when component mounts
      refetchOnMount: true,
    },
  },
});
```

**Why each option matters:**

**1. staleTime** - "How fresh does my data need to be?"

```typescript
staleTime: 0  // Always stale (refetch every time)
// Use for: Stock prices, live scores

staleTime: 1000 * 60 * 5  // Stale after 5 minutes
// Use for: User profiles, product lists

staleTime: Infinity  // Never stale
// Use for: Country list, static config
```

**What "stale" means:**

```text
Component mounts
  ↓
Has cached data?
  ↓ Yes
Return cached data immediately (fast!)
  ↓
Is data stale? (older than staleTime)
  ↓ Yes
Fetch fresh data in background
  ↓
Update UI when fresh data arrives
```

**2. retry** - "How persistent should we be?"

```typescript
retry: 3  // Try 3 times before giving up

// What happens:
Request 1: Failed (network error)
  ↓ Wait retryDelay
Request 2: Failed (still down)
  ↓ Wait longer
Request 3: Failed (still down)
  ↓ Wait even longer
Request 4: Failed
  ↓
Give up → isError: true
```

**Why retry is important:**

- Network hiccups are common (mobile users, WiFi issues)
- Servers sometimes have temporary issues
- Better UX (user doesn't see error immediately)

**When to reduce retries:**

```typescript
retry: 0  // Don't retry
// Use for: Authentication (wrong password won't fix itself)
// Use for: 404s (retrying won't help)
```

**3. cacheTime (gcTime in v5)** - "How long to remember?"

```typescript
// After last component unmounts:
cacheTime: 5 * 60 * 1000  // Keep for 5 minutes

// Timeline:
Component A mounts → Uses cached data
Component A unmounts → Start timer
[5 minutes pass]
Timer expires → Delete from cache
```

**Why this matters:**

```typescript
// User navigates: Home → Profile → Home

Without cache:
  Home loads → Fetch users (500ms)
  Click profile → Fetch profile (500ms)  
  Back to home → Fetch users again (500ms) ❌

With cache (cacheTime: 5 minutes):
  Home loads → Fetch users (500ms)
  Click profile → Fetch profile (500ms)
  Back to home → Instant! (0ms) ✓
```

**4. refetchOnWindowFocus** - "Update when user returns?"

```typescript
refetchOnWindowFocus: true  // Default

// User story:
User viewing product page
  ↓
Switches to another tab for 5 minutes
  ↓
Switches back
  ↓
TanStack Query refetches data (ensures freshness)
```

**Why this is good:**

- Prevents stale data (price might have changed)
- User sees fresh data without manual refresh
- Only happens if data is stale (respects staleTime)

**When to disable:**

```typescript
refetchOnWindowFocus: false
// Use for: Chat apps (use websockets instead)
// Use for: Form drafts (don't lose user input)
```

**ReactQueryDevtools - Your debugging superpower:**

```typescript
<ReactQueryDevtools initialIsOpen={false} />
```

**What you can see:**

- All queries and their states (fresh, stale, fetching)
- Cache contents (inspect actual data)
- Query timelines (when fetched, when will refetch)
- Network requests (which queries caused which fetches)

**Why it's essential during development:**

```text
Without DevTools:
  "Why is my data not updating?" 🤔
  "Is it cached or fetching?" 🤔
  "What's in the cache?" 🤔

With DevTools:
  [Open panel] → See exact query state
  [Check cache] → See cached data
  [View timeline] → See fetch history
```

**Pro tip:** DevTools only loads in development, automatically removed in production builds.

**Putting it together - Why this setup?**

```typescript
// ❌ Without TanStack Query setup
function UserProfile() {
  const [user, setUser] = useState(null);
  const [loading, setLoading] = useState(false);
  
  useEffect(() => {
    setLoading(true);
    fetchUser().then(setUser).finally(() => setLoading(false));
  }, []);
  
  // Problems:
  // - No caching (refetches every mount)
  // - No retry logic (fails immediately)
  // - No refetch on focus (data goes stale)
  // - Can't share data with other components
  // - Have to manage loading/error manually
}

// ✅ With TanStack Query
<QueryClientProvider client={queryClient}>
  <UserProfile />
  {/* Automatic caching, retries, refetching, sharing! */}
</QueryClientProvider>
```

**Mental Model:**

Think of QueryClient as a **smart database** in your browser:

- Stores fetched data (cache)
- Knows when data is old (staleTime)
- Automatically refreshes (background refetching)
- Shares data across components (no duplicates)
- Cleans up unused data (garbage collection)

The Provider is just React's way of making this database available to your entire app.

### Basic Query

**The Fundamental Question:** How do we get server data into our React component?

**Traditional Approach (useState + useEffect):**

```typescript
function Users() {
  const [data, setData] = useState(null);
  const [loading, setLoading] = useState(true);
  const [error, setError] = useState(null);
  
  useEffect(() => {
    setLoading(true);
    fetch('/api/users')
      .then(res => res.json())
      .then(data => {
        setData(data);
        setLoading(false);
      })
      .catch(error => {
        setError(error);
        setLoading(false);
      });
  }, []);
  
  if (loading) return <div>Loading...</div>;
  if (error) return <div>Error: {error.message}</div>;
  return <div>{data.map(user => <div key={user.id}>{user.name}</div>)}</div>;
}
```

**Problems with this approach:**

1. **Fetches every time component mounts** (no caching)

```text
User visits page → Fetch
User navigates away → Component unmounts
User returns → Fetch AGAIN (even if 1 second later)
```

2. **No sharing between components**

```text
Component A: const [users] = useState()  → Fetch users
Component B: const [users] = useState()  → Fetch users AGAIN
// Both components need the same data but fetch separately
```

3. **No automatic refetching** (data goes stale)

```text
User views data at 9:00 AM
Data changes on server at 9:30 AM
User still sees old data at 10:00 AM
```

4. **Complex error handling and retries**

```text
// Have to manually implement:
// - Retry logic
// - Exponential backoff
// - Error boundaries
// - Loading states for background refetches
```

**TanStack Query Approach:**

```typescript
import { useQuery } from '@tanstack/react-query';

function Users() {
  const { data, isLoading, error } = useQuery({
    queryKey: ['users'],
    queryFn: async () => {
      const res = await fetch('/api/users');
      return res.json();
    }
  });

  if (isLoading) return <div>Loading...</div>;
  if (error) return <div>Error: {error.message}</div>;

  return <div>{data.map(user => <div key={user.id}>{user.name}</div>)}</div>;
}
```

**What changed? Why is this better?**

**1. queryKey** - The cache identity

```typescript
queryKey: ['users']
```

**Why it's an array:**

- Allows hierarchical keys: `['users', 1, 'posts']`
- Supports complex keys: `['users', { status: 'active' }]`
- Easy to match patterns: Invalidate all user-related queries

**What happens:**

```text
Component A mounts
  ↓
useQuery checks cache for key ['users']
  ↓
Not found? → Execute queryFn → Store in cache under ['users']
  ↓
Component B mounts
  ↓  
useQuery checks cache for key ['users']
  ↓
Found! → Return cached data (no fetch needed!)
```

**Mental Model:** queryKey is like a filename. If two components request the same filename, they get the same file.

**2. queryFn** - The data fetcher

```typescript
queryFn: async () => {
  const res = await fetch('/api/users');
  return res.json();
}
```

**Important rules:**

**Must return a Promise:**

```typescript
// ✅ Correct
queryFn: async () => {
  const res = await fetch('/api/users');
  return res.json();
}

// ✅ Also correct
queryFn: () => {
  return fetch('/api/users').then(res => res.json());
}

// ❌ Wrong - doesn't return Promise
queryFn: () => {
  const data = someData;
  return data;
}
```

**Must throw on error:**
```typescript
// ❌ Wrong - silently returns error
queryFn: async () => {
  const res = await fetch('/api/users');
  return res;  // Returns response even if 404!
}

// ✅ Correct - throws on error
queryFn: async () => {
  const res = await fetch('/api/users');
  if (!res.ok) throw new Error('Failed to fetch');
  return res.json();
}

// ✅ Even better - axios throws automatically
queryFn: () => axios.get('/api/users').then(res => res.data)
```

**Why throw errors?**

- TanStack Query needs to know when fetch failed
- Triggers retry logic
- Sets `isError: true` and populates `error` object

**3. Destructuring the result**


```typescript
const { data, isLoading, error } = useQuery({...});
```

**Available properties explained:**

```typescript
const query = useQuery({...});

// Data states
query.data          // The successful result (undefined until loaded)
query.error         // Error object if queryFn threw

// Loading states (these are the confusing ones)
query.isLoading     // true ONLY on first load (no data yet)
query.isFetching    // true during ANY fetch (including background)
query.isPending     // true when no data exists yet (similar to isLoading)

// Success/Error states
query.isSuccess     // true when data exists
query.isError       // true when error exists

// Status (combines above)
query.status        // 'pending' | 'error' | 'success'
query.fetchStatus   // 'fetching' | 'paused' | 'idle'
```

**The confusing part: isLoading vs isFetching**

```typescript
// First mount:
isLoading: true    // No data yet!
isFetching: true   // Fetching now
data: undefined

// After data loads:
isLoading: false   // Have data now
isFetching: false  // Not fetching
data: [users]

// Background refetch (data is stale):
isLoading: false   // Still have data (old data)
isFetching: true   // Fetching new data in background
data: [users]      // Old data still shown

// After refetch completes:
isLoading: false
isFetching: false
data: [users]      // New data!
```

**When to use each:**

```typescript
// Use isLoading for initial "no data yet" state
if (isLoading) return <Spinner />;

// Use isFetching for "loading indicator" during background updates
if (isFetching) return <div>Updating... {data}</div>;

// Common pattern: Show data with loading indicator
<div>
  {isFetching && <LoadingBar />}
  {data && <DataList data={data} />}
</div>
```

**4. Automatic caching behavior**

```typescript
// Component A renders at 9:00:00
const { data } = useQuery({ 
  queryKey: ['users'], 
  queryFn: fetchUsers 
});
// Fetches users → Cached

// Component A unmounts at 9:00:05
// Cache kept for cacheTime (default: 5 minutes)

// Component B renders at 9:00:10
const { data } = useQuery({ 
  queryKey: ['users'], 
  queryFn: fetchUsers 
});
// Returns cached data (instant!)
// Checks: Is data stale?
//   If yes (older than staleTime) → Refetch in background
//   If no → Do nothing
```

**Why both staleTime AND cacheTime?**

```typescript
staleTime: "When should I refetch?"
cacheTime: "When should I forget?"

Example:
staleTime: 30 seconds
cacheTime: 5 minutes

Timeline:
0:00 → Fetch data
0:15 → Data still fresh (< 30s) → No refetch
0:45 → Data stale (> 30s) → Refetch in background
2:00 → Component unmounts
7:00 → Cache cleared (> 5min since unmount)
```

**Real-world example with different settings:**

```typescript
// Stock prices - Always fetch latest
const { data } = useQuery({
  queryKey: ['stock', symbol],
  queryFn: () => fetchStock(symbol),
  staleTime: 0,        // Always stale
  refetchInterval: 5000 // Refetch every 5 seconds
});

// User profile - Rarely changes
const { data } = useQuery({
  queryKey: ['user', userId],
  queryFn: () => fetchUser(userId),
  staleTime: 1000 * 60 * 10,  // Stale after 10 minutes
  cacheTime: 1000 * 60 * 30   // Keep cached for 30 minutes
});

// Country list - Never changes
const { data } = useQuery({
  queryKey: ['countries'],
  queryFn: fetchCountries,
  staleTime: Infinity,  // Never stale
  cacheTime: Infinity   // Never remove from cache
});
```

**Mental Model for useQuery:**

Think of useQuery as a **subscription** to a piece of server data:

```
Component subscribes: "I want 'users' data"
  ↓
TanStack Query: "Let me check my cache..."
  ↓
Found in cache? 
  ↓ Yes
Return immediately (fast!)
Is it stale?
  ↓ Yes
Fetch fresh data in background
Update subscribers when ready
  ↓
Component unsubscribes (unmounts)
  ↓
Start cache cleanup timer
```

You're not "fetching data", you're **subscribing to data** and TanStack Query handles all the fetching, caching, and synchronization logic for you!

### Query Keys: The Cache Identity

Query keys determine when React Query considers data the same:

```typescript
// Different cache entries
useQuery({ queryKey: ['users'] });
useQuery({ queryKey: ['users', 1] });
useQuery({ queryKey: ['users', 2] });
useQuery({ queryKey: ['users', { status: 'active' }] });

// Same cache entry (arrays are compared by value)
useQuery({ queryKey: ['users', 1] });
useQuery({ queryKey: ['users', 1] }); // Uses cached data
```

### Mutations

**The Core Concept:** Queries are for reading, mutations are for writing.

**Why separate reads from writes?**

Queries and mutations have fundamentally different behaviors:

| Aspect | Queries (Read) | Mutations (Write) |
|--------|----------------|-------------------|
| **When they run** | Automatically (on mount, refocus) | Only when you call them |
| **Caching** | Results are cached | Not cached (each is unique) |
| **Retries** | Automatic retries | Usually don't retry |
| **Multiple calls** | Deduplicated | Each call is independent |
| **Purpose** | Get current state | Change state |

**Example - Why this matters:**

```typescript
// ❌ Trying to use useQuery for writes (doesn't work well)
const { refetch } = useQuery({
  queryKey: ['createTodo'],
  queryFn: () => createTodo({ title: 'New Todo' }),
  enabled: false  // Don't run automatically
});

// Problems:
// 1. Have to manually trigger with refetch()
// 2. Result gets cached (wasteful - why cache a POST?)
// 3. No loading state per button click
// 4. Awkward API for something that should be simple
```

**The Right Way: useMutation**

```typescript
import { useMutation, useQueryClient } from '@tanstack/react-query';

function CreateTodo() {
  const queryClient = useQueryClient();
  
  const mutation = useMutation({
    mutationFn: (newTodo) => {
      return fetch('/api/todos', {
        method: 'POST',
        body: JSON.stringify(newTodo),
      });
    },
    onSuccess: () => {
      // Invalidate and refetch todos
      queryClient.invalidateQueries({ queryKey: ['todos'] });
    },
  });

  return (
    <button 
      onClick={() => mutation.mutate({ title: 'New Todo' })}
      disabled={mutation.isPending}
    >
      {mutation.isPending ? 'Creating...' : 'Create Todo'}
    </button>
  );
}
```

**Breaking down each part:**

**1. useMutation hook**

```typescript
const mutation = useMutation({
  mutationFn: (newTodo) => {...}
});
```

**Key difference from useQuery:**

- **useQuery runs automatically** (on mount, focus, etc.)
- **useMutation runs only when you call `.mutate()`**

```typescript
// useQuery
const query = useQuery({...});  // Starts fetching immediately!

// useMutation  
const mutation = useMutation({...});  // Does nothing yet
mutation.mutate(data);  // NOW it runs!
```

**Why this design?**

- Writes are user-initiated (button clicks, form submissions)
- You control exactly when they happen
- Multiple mutations can coexist without interfering

**2. mutationFn** - The actual API call

```typescript
mutationFn: (newTodo) => {
  return fetch('/api/todos', {
    method: 'POST',
    body: JSON.stringify(newTodo),
  });
}
```

**Important points:**

**Receives the data you pass to mutate:**

```typescript
mutation.mutate({ title: 'Buy milk' });
                  ↓
mutationFn: (newTodo) => {
  // newTodo is { title: 'Buy milk' }
}
```

**Must return a Promise:**

```typescript
// ✅ Async function (returns Promise)
mutationFn: async (data) => {
  const res = await fetch('/api/todos', {...});
  return res.json();
}

// ✅ Return Promise directly
mutationFn: (data) => {
  return fetch('/api/todos', {...}).then(r => r.json());
}

// ✅ Axios (returns Promise automatically)
mutationFn: (data) => axios.post('/api/todos', data)
```

**Should throw on error:**

```typescript
mutationFn: async (data) => {
  const res = await fetch('/api/todos', {
    method: 'POST',
    body: JSON.stringify(data)
  });
  
  if (!res.ok) {
    throw new Error(`HTTP ${res.status}`);  // Important!
  }
  
  return res.json();
}
```

**Why throw?** So TanStack Query knows the mutation failed and can:

- Set `isError: true`
- Call `onError` callback
- Not call `onSuccess` callback

**3. mutation.mutate()** - Triggering the mutation

```typescript
<button onClick={() => mutation.mutate({ title: 'New Todo' })}>
```

**What happens when you call mutate:**

```text
1. mutation.isPending becomes true
   ↓
2. Component re-renders (button shows "Creating...")
   ↓
3. mutationFn runs with the data you passed
   ↓
4. If successful:
   - mutation.isSuccess becomes true
   - mutation.data contains the result
   - onSuccess callback runs
   ↓
5. If failed:
   - mutation.isError becomes true
   - mutation.error contains the error
   - onError callback runs
```

**4. onSuccess callback** - What to do after success

```typescript
onSuccess: () => {
  queryClient.invalidateQueries({ queryKey: ['todos'] });
}
```

**Why invalidate queries?**

```text
User clicks "Create Todo"
  ↓
Mutation runs → Creates todo on server
  ↓
Server now has new todo
  ↓
But client's cached todos list is outdated!
  ↓
invalidateQueries marks the cache as stale
  ↓
Any components using that query refetch automatically
  ↓
UI updates with new todo!
```

**Without invalidation:**

```typescript
onSuccess: () => {
  // Do nothing
}

// Result:
// - Mutation succeeds
// - New todo exists on server
// - User doesn't see it (cache is stale)
// - User has to manually refresh page 😞
```

**5. mutation.isPending** - Loading state

```typescript
disabled={mutation.isPending}
{mutation.isPending ? 'Creating...' : 'Create Todo'}
```

**Why this is better than useState:**

```typescript
// ❌ Manual approach
const [loading, setLoading] = useState(false);

const handleCreate = async () => {
  setLoading(true);
  try {
    await createTodo({...});
    // Refetch todos
  } catch (error) {
    // Handle error
  } finally {
    setLoading(false);  // Easy to forget!
  }
};

// ✅ useMutation handles this automatically
const mutation = useMutation({...});
// mutation.isPending is true during request
// Automatically becomes false when done
```

**6. useQueryClient** - Accessing the cache

```typescript
const queryClient = useQueryClient();
```

**Why do we need this?**

The mutation needs to tell other parts of the app that data changed. QueryClient is the "control panel" for the cache:

```typescript
// Invalidate (mark as stale, trigger refetch)
queryClient.invalidateQueries({ queryKey: ['todos'] });

// Update cache directly (no refetch)
queryClient.setQueryData(['todos'], (old) => [...old, newTodo]);

// Remove from cache
queryClient.removeQueries({ queryKey: ['todos'] });

// Get current cached data
const todos = queryClient.getQueryData(['todos']);
```

**Real-world flow with detailed explanation:**

```typescript
function TodoApp() {
  const queryClient = useQueryClient();
  
  // Query: Get todos
  const { data: todos } = useQuery({
    queryKey: ['todos'],
    queryFn: fetchTodos
  });
  
  // Mutation: Create todo
  const createMutation = useMutation({
    mutationFn: createTodo,
    onSuccess: (newTodo) => {
      // Option 1: Invalidate (refetch from server)
      queryClient.invalidateQueries({ queryKey: ['todos'] });
      
      // Option 2: Update cache directly (instant UI update)
      // queryClient.setQueryData(['todos'], (old) => [...old, newTodo]);
    },
    onError: (error) => {
      alert(`Failed: ${error.message}`);
    }
  });
  
  const handleSubmit = (e) => {
    e.preventDefault();
    const title = e.target.title.value;
    createMutation.mutate({ title });
  };
  
  return (
    <div>
      <form onSubmit={handleSubmit}>
        <input name="title" />
        <button disabled={createMutation.isPending}>
          {createMutation.isPending ? 'Creating...' : 'Add Todo'}
        </button>
      </form>
      
      {createMutation.isError && (
        <div>Error: {createMutation.error.message}</div>
      )}
      
      {todos?.map(todo => (
        <div key={todo.id}>{todo.title}</div>
      ))}
    </div>
  );
}
```

**Timeline of what happens:**

```
1. User types "Buy milk" and clicks "Add Todo"
   ↓
2. handleSubmit runs → calls createMutation.mutate({ title: 'Buy milk' })
   ↓
3. mutation.isPending = true → Button shows "Creating..."
   ↓
4. mutationFn runs → POST /api/todos with data
   ↓
5a. If successful:
    - mutation.isSuccess = true
    - onSuccess runs
    - invalidateQueries marks ['todos'] as stale
    - useQuery sees stale data → refetches
    - New todo appears in list!
    
5b. If failed:
    - mutation.isError = true
    - mutation.error = Error object
    - onError runs
    - Alert shows error message
    - Button re-enabled for retry
```

**Mental Model:**

Think of mutations as **commands** you send to the server:

- You decide when to send them (`.mutate()`)
- They do their job (mutationFn)
- They report back (onSuccess/onError)
- They update related data (invalidateQueries)

Unlike queries (which are subscriptions that stay active), mutations are one-time actions triggered by user intent.

### Real-World Example: Todo App

```typescript
// api.ts
export async function fetchTodos() {
  const response = await axios.get('/api/todos');
  return response.data.data; // Unwrap your API response
}

export async function createTodo(todo: Omit<Todo, 'id'>) {
  const response = await axios.post('/api/todos', todo);
  return response.data.data;
}

export async function updateTodo(todo: Todo) {
  const response = await axios.put(`/api/todos/${todo.id}`, todo);
  return response.data.data;
}

export async function deleteTodo(id: string) {
  await axios.delete(`/api/todos/${id}`);
}

// TodoList.tsx
function TodoList() {
  const queryClient = useQueryClient();
  
  // Query
  const { data: todos, isLoading } = useQuery({
    queryKey: ['todos'],
    queryFn: fetchTodos,
  });
  
  // Create mutation
  const createMutation = useMutation({
    mutationFn: createTodo,
    onSuccess: () => {
      queryClient.invalidateQueries({ queryKey: ['todos'] });
    },
  });
  
  // Delete mutation
  const deleteMutation = useMutation({
    mutationFn: deleteTodo,
    onSuccess: () => {
      queryClient.invalidateQueries({ queryKey: ['todos'] });
    },
  });
  
  // Toggle mutation
  const toggleMutation = useMutation({
    mutationFn: updateTodo,
    onSuccess: () => {
      queryClient.invalidateQueries({ queryKey: ['todos'] });
    },
  });

  if (isLoading) return <div>Loading...</div>;

  return (
    <div>
      {todos?.map(todo => (
        <div key={todo.id}>
          <input
            type="checkbox"
            checked={todo.completed}
            onChange={() => toggleMutation.mutate({
              ...todo,
              completed: !todo.completed
            })}
          />
          <span>{todo.title}</span>
          <button onClick={() => deleteMutation.mutate(todo.id)}>
            Delete
          </button>
        </div>
      ))}
    </div>
  );
}
```

### Optimistic Updates

**Theory**: Optimistic UI assumes server operations will succeed and updates the UI immediately, then rolls back if the operation fails.

**Trade-off Analysis:**

```text
Without Optimistic Updates:
  User clicks → Show loading → Wait for server → Update UI
  Latency: Noticeable delay
  Feedback: Delayed but accurate

With Optimistic Updates:
  User clicks → Update UI immediately → Confirm with server
  Latency: Instant feedback
  Feedback: Fast but may need rollback
```

**When to use:**

- Low failure rate operations (99%+ success)
- User actions that feel sluggish without it (likes, toggles)
- When rollback is visually acceptable

**When NOT to use:**

- Financial transactions (money transfers)
- Operations with complex validation
- Actions where failure is common

Update the UI immediately, then rollback if the server request fails:

```typescript
const updateMutation = useMutation({
  mutationFn: updateTodo,
  
  onMutate: async (updatedTodo) => {
    // Cancel outgoing refetches
    await queryClient.cancelQueries({ queryKey: ['todos'] });
    
    // Snapshot previous value
    const previousTodos = queryClient.getQueryData(['todos']);
    
    // Optimistically update
    queryClient.setQueryData(['todos'], (old) =>
      old.map(todo => 
        todo.id === updatedTodo.id ? updatedTodo : todo
      )
    );
    
    // Return context for rollback
    return { previousTodos };
  },
  
  onError: (err, updatedTodo, context) => {
    // Rollback on error
    queryClient.setQueryData(['todos'], context.previousTodos);
  },
  
  onSettled: () => {
    // Always refetch after error or success
    queryClient.invalidateQueries({ queryKey: ['todos'] });
  },
});
```

**Understanding the flow:**

1. **onMutate** - Runs BEFORE the mutation request
   - Cancel any in-flight queries (prevents race conditions)
   - Save current state (for rollback)
   - Apply optimistic update (user sees change immediately)
   - Return context object (passed to onError)

2. **Mutation executes** - Network request happens

3. **onError** - If mutation fails
   - Restore previous state from context
   - User sees original data again

4. **onSuccess** - If mutation succeeds
   - Server confirms change
   - Data stays updated

5. **onSettled** - Always runs (success or error)
   - Refetch to ensure client/server sync
   - Catches any edge cases

**The Three-Tier Consistency Model:**

```
Tier 1: Instant (Optimistic)
  - Update local cache immediately
  - User sees change in ~0ms
  - May be incorrect

Tier 2: Eventually Consistent (onSuccess)
  - Server confirms mutation
  - Update confirmed in ~100-500ms
  - Likely correct

Tier 3: Strongly Consistent (onSettled refetch)
  - Refetch from server
  - Guaranteed correct in ~100-500ms
  - Handles edge cases (other users' changes, server-side modifications)
```

### Dependent Queries

```typescript
function UserPosts({ userId }) {
  // First query: Get user
  const { data: user } = useQuery({
    queryKey: ['user', userId],
    queryFn: () => fetchUser(userId),
  });
  
  // Second query: Get user's posts (only runs when user exists)
  const { data: posts } = useQuery({
    queryKey: ['posts', user?.id],
    queryFn: () => fetchUserPosts(user.id),
    enabled: !!user?.id, // Only run if user.id exists
  });
  
  return <div>...</div>;
}
```

### Pagination

**Theory**: Pagination divides large datasets into manageable chunks. Two common patterns exist:

**1. Page-Based Pagination**

- Navigate between discrete pages (1, 2, 3...)
- Each page replaces the previous one
- URL-friendly (shareable page numbers)
- Used when: Users need random access to pages

**2. Infinite Scroll (Cursor-Based)**

- Append new items to existing list
- Continuous scrolling experience
- Better for mobile/social feeds
- Used when: Linear browsing pattern

#### Page-Based Pagination

```typescript
function PaginatedPosts() {
  const [page, setPage] = useState(1);
  
  const { data, isLoading } = useQuery({
    queryKey: ['posts', page],
    queryFn: () => fetchPosts(page),
    keepPreviousData: true, // Show old data while fetching new
  });
  
  return (
    <div>
      {data?.posts.map(post => <Post key={post.id} {...post} />)}
      <button 
        onClick={() => setPage(p => p - 1)} 
        disabled={page === 1}
      >
        Previous
      </button>
      <button 
        onClick={() => setPage(p => p + 1)}
        disabled={!data?.hasMore}
      >
        Next
      </button>
    </div>
  );
}
```

**Understanding `keepPreviousData`:**

Without it:

```text
Page 1 displayed → User clicks Next
  ↓
Page 1 removed → Loading spinner
  ↓
Page 2 fetched → Page 2 displayed
```

With `keepPreviousData: true`:

```text
Page 1 displayed → User clicks Next
  ↓
Page 1 stays visible → Background indicator shows loading
  ↓
Page 2 fetched → Smooth transition to Page 2
```

This prevents layout shift and improves perceived performance.

**Query key behavior:**

```typescript
queryKey: ['posts', 1] // Cached separately
queryKey: ['posts', 2] // Cached separately
queryKey: ['posts', 3] // Cached separately

// Each page is cached independently
// Navigation between visited pages is instant
```

#### Infinite Queries (Load More)

**Theory**: Infinite queries maintain a **list of pages** rather than a single page. Each page is appended to the list.

```text
Page 1: [items 1-10]
  ↓ Load more
Page 2: [items 1-10, items 11-20]
  ↓ Load more
Page 3: [items 1-10, items 11-20, items 21-30]
```

```typescript
function InfinitePosts() {
  const {
    data,
    fetchNextPage,
    hasNextPage,
    isFetchingNextPage,
  } = useInfiniteQuery({
    queryKey: ['posts'],
    queryFn: ({ pageParam = 1 }) => fetchPosts(pageParam),
    getNextPageParam: (lastPage, pages) => {
      return lastPage.hasMore ? pages.length + 1 : undefined;
    },
  });
  
  return (
    <div>
      {data?.pages.map((page, i) => (
        <div key={i}>
          {page.posts.map(post => (
            <Post key={post.id} {...post} />
          ))}
        </div>
      ))}
      
      {hasNextPage && (
        <button 
          onClick={() => fetchNextPage()}
          disabled={isFetchingNextPage}
        >
          {isFetchingNextPage ? 'Loading...' : 'Load More'}
        </button>
      )}
    </div>
  );
}
```

**Data structure:**

```typescript
data = {
  pages: [
    { posts: [...], hasMore: true },   // Page 1
    { posts: [...], hasMore: true },   // Page 2
    { posts: [...], hasMore: false }   // Page 3 (last)
  ],
  pageParams: [1, 2, 3]
}
```

**Understanding `getNextPageParam`:**
This function determines the parameter for the next fetch. It receives:

- `lastPage` - The most recently fetched page
- `pages` - Array of all pages fetched so far

Return `undefined` to indicate no more pages exist.

**Cursor-based pagination** (alternative to page numbers):

```typescript
getNextPageParam: (lastPage) => {
  // Server returns a cursor for the next page
  return lastPage.nextCursor ?? undefined;
}

queryFn: ({ pageParam }) => 
  fetchPosts({ cursor: pageParam })
```

**Why cursors?**

- Handles real-time data (new items added don't shift pages)
- More efficient for large datasets
- Prevents duplicate items

Compare approaches:
```text
Offset-based (page numbers):
  Page 1: Items 0-9
  [New item inserted]
  Page 2: Items 10-19 (but item 10 was already shown on page 1!)

Cursor-based:
  Page 1: Items up to cursor "abc123"
  [New item inserted]
  Page 2: Items after cursor "abc123" (no duplicates)
```

---

## Real-World Patterns

### Mental Model: The Data Flow Architecture

Understanding how data flows through your application helps you make better architectural decisions:

```text
User Interaction
  ↓
UI State Changes (Zustand)
  ↓
Triggers Mutation
  ↓
Optimistic Update (Zustand or TanStack Query cache)
  ↓
Server Request
  ↓
Server Response
  ↓
Cache Invalidation/Update (TanStack Query)
  ↓
UI Re-renders with fresh data
```

**Key insight**: Separate **intention** (Zustand state like "modal open") from **data** (TanStack Query cache).

```typescript
// ❌ Mixing concerns
const useStore = create((set) => ({
  users: [],           // Server data
  modalOpen: false,    // UI state
  selectedId: null,    // UI state
  fetchUsers: async () => {...}  // Server operation
}));

// ✅ Separation of concerns
const useUIStore = create((set) => ({
  modalOpen: false,
  selectedId: null,
}));

// Server data handled separately
const { data: users } = useQuery({
  queryKey: ['users'],
  queryFn: fetchUsers
});
```

### Theory: Normalized vs Denormalized Data

**Normalized Data** (Relational model):

```typescript
{
  users: { 1: { id: 1, name: "Alice" } },
  posts: { 
    101: { id: 101, title: "Hello", authorId: 1 }
  }
}
```

**Denormalized Data** (Document model):

```typescript
{
  posts: [
    { 
      id: 101, 
      title: "Hello", 
      author: { id: 1, name: "Alice" } 
    }
  ]
}
```

**TanStack Query recommendation**: Store data as the API returns it (usually denormalized). Don't normalize unless you have a specific reason:

Reasons to normalize:

- Multiple entities reference same data
- Need to update entity everywhere simultaneously
- Memory constraints (large datasets)

Reasons to keep denormalized:

- Simpler code (matches API structure)
- Easier to work with
- TanStack Query handles cache invalidation

```typescript
// API returns denormalized data
// Keep it that way in the cache
const { data: posts } = useQuery({
  queryKey: ['posts'],
  queryFn: fetchPosts
});

// If you need normalized access, compute it
const postsById = useMemo(
  () => posts?.reduce((acc, post) => {
    acc[post.id] = post;
    return acc;
  }, {}),
  [posts]
);
```

### Theory: When to Colocate vs Separate State

**Colocation Principle**: Keep state as close as possible to where it's used.

```typescript
// ❌ Unnecessarily global
const useStore = create((set) => ({
  searchQuery: '',      // Only used in SearchBar
  sortOrder: 'desc',    // Only used in PostList
}));

// ✅ Colocated (component state)
function SearchBar() {
  const [query, setQuery] = useState('');
  // Only this component needs it
}

function PostList() {
  const [sortOrder, setSortOrder] = useState('desc');
  // Only this component needs it
}
```

**When to elevate to Zustand:**

1. **Multiple components** need the same state
2. **Unrelated components** (not parent-child) need to communicate
3. State should **persist** across component unmounts
4. State changes are **frequent** and should be optimized

Example - Modal state elevation:

```typescript
// Initially: Component state
function UserProfile() {
  const [modalOpen, setModalOpen] = useState(false);
  // Works fine if modal is within this component
}

// Later: Needs to be opened from navbar
// Elevate to Zustand
const useUIStore = create((set) => ({
  profileModalOpen: false,
  openProfileModal: () => set({ profileModalOpen: true }),
  closeProfileModal: () => set({ profileModalOpen: false })
}));
```

### Pattern 1: Combining Zustand + TanStack Query

```typescript
// UI State: Zustand
const useUIStore = create((set) => ({
  sidebarOpen: false,
  selectedTodoId: null,
  toggleSidebar: () => set((state) => ({ 
    sidebarOpen: !state.sidebarOpen 
  })),
  selectTodo: (id) => set({ selectedTodoId: id }),
}));

// Server State: TanStack Query
function TodoApp() {
  const { sidebarOpen, selectedTodoId } = useUIStore();
  const { data: todos } = useQuery({
    queryKey: ['todos'],
    queryFn: fetchTodos,
  });
  
  const selectedTodo = todos?.find(t => t.id === selectedTodoId);
  
  return (
    <div>
      <TodoList todos={todos} />
      {sidebarOpen && <Sidebar todo={selectedTodo} />}
    </div>
  );
}
```

### Pattern 2: Custom Hooks

```typescript
// useTodos.ts - Encapsulate all todo logic
export function useTodos() {
  const queryClient = useQueryClient();
  
  const todosQuery = useQuery({
    queryKey: ['todos'],
    queryFn: fetchTodos,
  });
  
  const createMutation = useMutation({
    mutationFn: createTodo,
    onSuccess: () => {
      queryClient.invalidateQueries({ queryKey: ['todos'] });
    },
  });
  
  const updateMutation = useMutation({
    mutationFn: updateTodo,
    onSuccess: () => {
      queryClient.invalidateQueries({ queryKey: ['todos'] });
    },
  });
  
  const deleteMutation = useMutation({
    mutationFn: deleteTodo,
    onSuccess: () => {
      queryClient.invalidateQueries({ queryKey: ['todos'] });
    },
  });
  
  return {
    todos: todosQuery.data,
    isLoading: todosQuery.isLoading,
    error: todosQuery.error,
    createTodo: createMutation.mutate,
    updateTodo: updateMutation.mutate,
    deleteTodo: deleteMutation.mutate,
    isCreating: createMutation.isPending,
    isUpdating: updateMutation.isPending,
    isDeleting: deleteMutation.isPending,
  };
}

// Usage in component
function TodoList() {
  const { todos, isLoading, createTodo, deleteTodo } = useTodos();
  
  // Clean component logic
}
```

### Pattern 3: Axios Interceptor for Clean API Calls

```typescript
// api/client.ts
import axios from 'axios';

const axiosInstance = axios.create({
  baseURL: import.meta.env.VITE_API_URL,
});

// Unwrap nested data.data structure
axiosInstance.interceptors.response.use(
  (response) => {
    // If your API wraps responses in { data: {...} }
    return response.data.data ?? response.data;
  },
  (error) => {
    // Handle errors globally
    if (error.response?.status === 401) {
      // Redirect to login
    }
    return Promise.reject(error);
  }
);

// Now your API functions are clean
export async function fetchTodos(): Promise<Todo[]> {
  return axiosInstance.get('/todos'); // No .data.data needed!
}

export async function createTodo(todo: CreateTodoDTO): Promise<Todo> {
  return axiosInstance.post('/todos', todo);
}
```

### Pattern 4: Error Boundaries with React Query

```typescript
// ErrorBoundary.tsx
import { useQueryErrorResetBoundary } from '@tanstack/react-query';
import { ErrorBoundary as ReactErrorBoundary } from 'react-error-boundary';

function ErrorFallback({ error, resetErrorBoundary }) {
  return (
    <div>
      <h2>Something went wrong</h2>
      <pre>{error.message}</pre>
      <button onClick={resetErrorBoundary}>Try again</button>
    </div>
  );
}

export function QueryErrorBoundary({ children }) {
  const { reset } = useQueryErrorResetBoundary();
  
  return (
    <ReactErrorBoundary
      onReset={reset}
      fallbackRender={ErrorFallback}
    >
      {children}
    </ReactErrorBoundary>
  );
}
```

### Pattern 5: Prefetching

```typescript
function UserList() {
  const queryClient = useQueryClient();
  const { data: users } = useQuery({
    queryKey: ['users'],
    queryFn: fetchUsers,
  });
  
  // Prefetch user details on hover
  const prefetchUser = (userId: string) => {
    queryClient.prefetchQuery({
      queryKey: ['user', userId],
      queryFn: () => fetchUser(userId),
      staleTime: 1000 * 60 * 5, // 5 minutes
    });
  };
  
  return (
    <div>
      {users?.map(user => (
        <div 
          key={user.id}
          onMouseEnter={() => prefetchUser(user.id)}
        >
          <Link to={`/users/${user.id}`}>{user.name}</Link>
        </div>
      ))}
    </div>
  );
}
```

---

## Best Practices

### Zustand Best Practices

1. **Keep stores focused and small**

```typescript
// ✅ Good: Single responsibility
const useAuthStore = create(...);
const useCartStore = create(...);
const useUIStore = create(...);

// ❌ Bad: God store
const useAppStore = create(...); // Everything in one store
```

2. **Use selectors to prevent unnecessary re-renders**

```typescript
// ❌ Bad: Re-renders on any state change
const { count, user, settings } = useStore();

// ✅ Good: Only re-renders when count changes
const count = useStore((state) => state.count);
```

3. **Use TypeScript for type safety**

```typescript
interface TodoStore {
  todos: Todo[];
  addTodo: (todo: Todo) => void;
  removeTodo: (id: string) => void;
}

const useTodoStore = create<TodoStore>((set) => ({
  todos: [],
  addTodo: (todo) => set((state) => ({ 
    todos: [...state.todos, todo] 
  })),
  removeTodo: (id) => set((state) => ({ 
    todos: state.todos.filter(t => t.id !== id) 
  })),
}));
```

4. **Don't store server data in Zustand**

```typescript
// ❌ Bad: Server data in Zustand
const useStore = create((set) => ({
  users: [],
  fetchUsers: async () => {
    const users = await fetchUsers();
    set({ users });
  }
}));

// ✅ Good: Use TanStack Query for server data
const { data: users } = useQuery({
  queryKey: ['users'],
  queryFn: fetchUsers
});
```

### TanStack Query Best Practices

1. **Structure query keys hierarchically**

```typescript
['todos']                           // All todos
['todos', { status: 'done' }]      // Filtered todos
['todos', 'detail', todoId]        // Single todo
['todos', 'detail', todoId, 'comments'] // Todo comments
```

2. **Set appropriate staleTime and cacheTime**

```typescript
// Fast-changing data (stock prices)
staleTime: 0,
cacheTime: 1000 * 60 * 5 // 5 minutes

// Slow-changing data (user profile)
staleTime: 1000 * 60 * 10, // 10 minutes
cacheTime: 1000 * 60 * 30  // 30 minutes

// Static data (country list)
staleTime: Infinity,
cacheTime: Infinity
```

3. **Handle loading and error states properly**

```typescript
function Component() {
  const { data, isLoading, isError, error } = useQuery({...});
  
  if (isLoading) return <Skeleton />;
  if (isError) return <ErrorMessage error={error} />;
  if (!data) return <Empty />;
  
  return <Content data={data} />;
}
```

4. **Use mutations for all write operations**

```typescript
// ❌ Bad: Manual state management
const handleUpdate = async () => {
  setLoading(true);
  try {
    await updateTodo(todo);
    // Manually refetch
    const todos = await fetchTodos();
    setTodos(todos);
  } catch (error) {
    // Handle error
  } finally {
    setLoading(false);
  }
};

// ✅ Good: Use mutation
const mutation = useMutation({
  mutationFn: updateTodo,
  onSuccess: () => {
    queryClient.invalidateQueries({ queryKey: ['todos'] });
  },
});
```

5. **Invalidate or update cache after mutations**

```typescript
// Option 1: Invalidate (triggers refetch)
onSuccess: () => {
  queryClient.invalidateQueries({ queryKey: ['todos'] });
}

// Option 2: Update cache directly (no refetch)
onSuccess: (newTodo) => {
  queryClient.setQueryData(['todos'], (old) => [...old, newTodo]);
}

// Option 3: Both (update immediately, then refetch for consistency)
onSuccess: (newTodo) => {
  queryClient.setQueryData(['todos'], (old) => [...old, newTodo]);
  queryClient.invalidateQueries({ queryKey: ['todos'] });
}
```

---

## Common Pitfalls

### Pitfall 1: Storing Derived State

```typescript
// ❌ Bad: Storing derived state
const useStore = create((set) => ({
  todos: [],
  completedCount: 0,
  addTodo: (todo) => set((state) => ({
    todos: [...state.todos, todo],
    completedCount: state.todos.filter(t => t.completed).length
  }))
}));

// ✅ Good: Compute on read
const useStore = create((set) => ({
  todos: [],
  addTodo: (todo) => set((state) => ({
    todos: [...state.todos, todo]
  }))
}));

// In component
const completedCount = useStore(
  (state) => state.todos.filter(t => t.completed).length
);
```

### Pitfall 2: Not Using Query Keys Correctly

```typescript
// ❌ Bad: Same key for different data
useQuery({ 
  queryKey: ['user'], 
  queryFn: () => fetchUser(userId) 
});

// ✅ Good: Include parameters in key
useQuery({ 
  queryKey: ['user', userId], 
  queryFn: () => fetchUser(userId) 
});
```

### Pitfall 3: Forgetting to Handle Loading States

```typescript
// ❌ Bad: Accessing data without checking
function Component() {
  const { data } = useQuery({...});
  return <div>{data.name}</div>; // Error if data is undefined!
}

// ✅ Good: Handle all states
function Component() {
  const { data, isLoading } = useQuery({...});
  
  if (isLoading) return <Spinner />;
  if (!data) return null;
  
  return <div>{data.name}</div>;
}
```

### Pitfall 4: Over-invalidating Queries

```typescript
// ❌ Bad: Invalidates ALL queries
queryClient.invalidateQueries();

// ✅ Good: Target specific queries
queryClient.invalidateQueries({ queryKey: ['todos'] });

// ✅ Better: Exact match
queryClient.invalidateQueries({ 
  queryKey: ['todos', todoId], 
  exact: true 
});
```

---

## Practice Exercises

### Exercise 1: Build a Shopping Cart

**Requirements:**

- Add/remove items (Zustand)
- Fetch product list (TanStack Query)
- Display cart total
- Persist cart to localStorage

**Hint:** Use Zustand's `persist` middleware for the cart, TanStack Query for products.

### Exercise 2: User Management Dashboard

**Requirements:**

- List users with pagination
- Create/update/delete users
- Optimistic updates for better UX
- Search and filter functionality

**Hint:** Use query keys like `['users', { page, search }]` for filtering.

### Exercise 3: Real-time Dashboard

**Requirements:**

- Fetch data every 10 seconds
- Show loading indicator during background refetch
- Handle errors gracefully
- Allow manual refresh

**Hint:** Use `refetchInterval` option in useQuery.

---

## Additional Resources

### Official Documentation

- [Zustand Documentation](https://github.com/pmndrs/zustand)
- [TanStack Query Documentation](https://tanstack.com/query/latest)

### Video Tutorials

- TkDodo's React Query Blog Series
- Jack Herrington's State Management Videos

### Tools

- React Query DevTools - Visual debugging
- Redux DevTools - Works with Zustand middleware

### Key Takeaways

1. **Use the right tool for the job**
   - Zustand: UI state, preferences, temporary data
   - TanStack Query: Server data, caching, synchronization

2. **Think about data flow**
   - Where does this data come from?
   - Who needs to access it?
   - How often does it change?

3. **Start simple, optimize later**
   - Basic useQuery is often enough
   - Add optimistic updates when UX demands it
   - Measure before optimizing

4. **Type everything**
   - TypeScript catches errors at compile time
   - Better IDE autocomplete
   - Self-documenting code