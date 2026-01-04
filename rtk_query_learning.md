# RTK Query Mastery: Complete Learning Guide

A comprehensive 12-day roadmap to master Redux Toolkit Query (RTK Query) from fundamentals to advanced patterns.

---

## 📑 Table of Contents

### Getting Started
- [Prerequisites](#-prerequisites)
- [Learning Objectives](#-learning-objectives)
- [Daily Time Investment](#-suggested-daily-time-investment)
- [Progress Tracker](#-progress-tracker)

### Learning Phases
- [Phase 1: Core Concepts & Mental Models (Days 1-2)](#-phase-1-core-concepts--mental-models-days-1-2)
  - [Theoretical Foundation](#theoretical-foundation)
  - [The Problem Space](#1-the-problem-space---deep-dive)
  - [RTK Query's Core Philosophy](#2-rtk-querys-core-philosophy)
  - [API Slice Anatomy](#3-api-slice-anatomy---understanding-the-generated-code)
  - [Resources & Practice](#resources)
  
- [Phase 2: The Data Flow (Days 3-4)](#-phase-2-the-data-flow-days-3-4)
  - [Theoretical Foundation](#theoretical-foundation-1)
  - [Query Lifecycle](#1-query-lifecycle---the-complete-state-machine)
  - [Cache Keys and Tags](#2-cache-keys-and-tags---the-indexing-system)
  - [Automatic Refetching and Polling](#3-automatic-refetching-and-polling)
  - [Resources & Practice](#resources-1)
  
- [Phase 3: Advanced Cache Management (Days 5-6)](#-phase-3-advanced-cache-management-days-5-6)
  - [Tag-Based Invalidation Strategies](#1-tag-based-invalidation-strategies)
  - [Optimistic Updates](#2-optimistic-updates)
  - [Cache Entry Lifecycle](#3-cache-entry-lifecycle)
  - [Resources & Practice](#resources-2)
  
- [Phase 4: Customization & Control (Days 7-8)](#️-phase-4-customization--control-days-7-8)
  - [Base Queries and Custom Fetch](#1-base-queries-and-custom-fetch)
  - [Query Transformations](#2-query-transformations)
  - [Middleware and Lifecycle Hooks](#3-middleware-and-lifecycle-hooks)
  - [Resources & Practice](#resources-3)
  
- [Phase 5: Performance & Patterns (Days 9-10)](#-phase-5-performance--patterns-days-9-10)
  - [Performance Considerations](#1-performance-considerations)
  - [Advanced Patterns](#2-advanced-patterns)
  - [Testing Strategies](#3-testing-strategies)
  - [Resources & Practice](#resources-4)
  
- [Phase 6: Real-World Integration (Days 11-12)](#-phase-6-real-world-integration-days-11-12)
  - [Common Patterns](#1-common-patterns)
  - [Error Handling](#2-error-handling)
  - [DevTools and Debugging](#3-devtools-and-debugging)
  - [Final Project Ideas](#final-project-ideas)
  - [Resources](#resources-5)

### Reference Material
- [⚙️ Complete RTK Query Configuration Reference](#️-complete-rtk-query-configuration-reference)
  - [createApi Configuration Options](#createapi-configuration-options)
  - [fetchBaseQuery Configuration Options](#fetchbasequery-configuration-options)
  - [Endpoint Configuration Options](#endpoint-configuration-options)
  - [Hook Configuration Options](#hook-configuration-options)
  - [Return Values from Hooks](#return-values-from-hooks)
  - [Advanced: injectEndpoints](#advanced-createapi-with-injectendpoints)
  - [Configuration Decision Tree](#configuration-decision-tree)

### Additional Resources
- [🎓 Learning Tips](#-learning-tips)
- [🔑 Key Takeaways](#-key-takeaways)
- [📖 Additional Resources](#-additional-resources)
- [🤝 Contributing](#-contributing)

---

## 📋 Prerequisites

Before starting this guide, you should be comfortable with:
- React fundamentals (hooks: `useState`, `useEffect`, `useContext`)
- Redux basics (actions, reducers, store)
- Redux Toolkit (RTK) fundamentals
- Asynchronous JavaScript (Promises, async/await)

---

## 🎯 Learning Objectives

By the end of this guide, you will:
- Understand RTK Query's architecture and philosophy
- Master cache management and invalidation strategies
- Implement optimistic updates and real-time data sync
- Customize RTK Query for complex scenarios
- Apply performance optimization techniques
- Build production-ready applications with RTK Query

---

## 📚 Phase 1: Core Concepts & Mental Models (Days 1-2)

### Goal
Understand what RTK Query actually is and why it exists

### Topics

#### 1. The Problem Space
- Traditional Redux data fetching patterns (actions, reducers, thunks)
- The repetitive boilerplate problem
- Cache management challenges

**Key Insight:** Before RTK Query, every API endpoint required creating action types, action creators, reducers, and thunks. RTK Query eliminates this boilerplate by treating your API as a declarative data source.

#### 2. RTK Query's Core Philosophy
- It's Redux under the hood (demystifying the magic)
- Server state vs. client state
- Cache-first architecture

**Mental Model Shift:** RTK Query IS Redux. When you call `useGetPostsQuery()`, it dispatches actions to a Redux slice, stores normalized data, and manages subscriptions automatically.

**Server State vs Client State:**
- **Server state**: Data from your backend (posts, users, products) → RTK Query
- **Client state**: UI state (modal open, theme) → Regular Redux slices

#### 3. API Slice Anatomy
- `createApi` and what it generates
- Endpoints: queries vs mutations
- Auto-generated hooks and their naming

### Resources

1. **Official Docs - Why RTK Query**
   - https://redux-toolkit.js.org/rtk-query/overview
   - Read the "Motivation" section carefully

2. **Video: RTK Query Basics** (Mark Erikson - Redux maintainer)
   - https://www.youtube.com/watch?v=HyWYpM_S-2c
   - Watch 0:00-20:00 for foundational understanding

3. **Article: "Thinking in RTK Query"**
   - https://redux-toolkit.js.org/rtk-query/usage/customizing-queries
   - Focus on the mental model section

### Practice Exercise

Build a simple API slice for a basic CRUD operation (todo list or blog posts):
- Define an API slice with `createApi`
- Create query endpoints for GET operations
- Create mutation endpoints for POST/PUT/DELETE
- Use the auto-generated hooks in React components

### Self-Check Question

**In your own words, what's the fundamental difference between how traditional Redux and RTK Query approach data fetching?**

---

## 🔄 Phase 2: The Data Flow (Days 3-4)

### Goal
Deeply understand how data moves through RTK Query

### Theoretical Foundation

#### The Request-Response Cycle in Modern Web Applications

Before diving into RTK Query's data flow, let's understand the fundamental request-response cycle:

**Traditional HTTP Cycle:**
```
Client → HTTP Request → Network → Server → Process → Response → Network → Client
```

**RTK Query's Enhanced Cycle:**
```
Component → Cache Check → [Network?] → Normalization → Store → Subscription Notification → Component Update
```

**Key Innovation**: RTK Query intercepts the cycle at multiple points to add caching, deduplication, and subscription management.

#### The Subscription Model - A Publish-Subscribe Pattern

RTK Query implements a **publish-subscribe (pub-sub)** pattern for data management:

**Traditional Approach:**
```
Component A fetches → Stores in state → Component B also needs data → Fetches again
Result: 2 network requests, 2 copies of data
```

**RTK Query Approach:**
```
Component A subscribes → Library fetches (if needed) → Components B and C also subscribe → All share same cache
Result: 1 network request, 1 copy of data, 3 subscribers
```

**The Subscription Lifecycle:**

```
Component mounts → Creates subscription → Adds to subscriber count
     ↓
[Subscription Active]
     ↓
Component unmounts → Removes subscription → Decrements subscriber count
     ↓
Subscriber count = 0? → Start garbage collection timer
     ↓
Timer expires (keepUnusedDataFor) → Remove from cache
```

**Mathematical Model:**

For a cache entry with `n` subscribers:
- **Memory**: Single copy of data (O(1) space complexity)
- **Network**: Single request regardless of `n` (O(1) time complexity)
- **Updates**: O(n) notification (but highly optimized with React's batching)

Compare to traditional approach: O(n) space and O(n) network requests.

### Topics

#### 1. Query Lifecycle - The Complete State Machine

**Step-by-step flow when a component calls `useGetPostsQuery()`:**

**Stage 1: Invocation**
```javascript
const result = useGetPostsQuery(args)
```

The hook immediately:
1. Generates cache key from endpoint name + args: `"getPosts({\"filter\":\"recent\"})"`
2. Checks if a subscription already exists for this cache key
3. Increments subscriber count for this cache key

**Stage 2: Cache Check**
```
Does cache entry exist for this key?
├─ NO → Proceed to Stage 3 (Fetch)
└─ YES → Check staleness
    ├─ Fresh → Return cached data immediately (Skip to Stage 5)
    └─ Stale → Return cached data immediately AND proceed to Stage 3
```

**Staleness Determination:**
```javascript
isStale = (currentTime - lastFetchTime) > refetchOnMountOrArgChange
```

**Stage 3: Fetch Decision**
```
Should we fetch?
├─ No cache entry → YES, fetch
├─ Cache exists but stale → YES, fetch (background)
├─ Cache exists and fresh → NO, use cache
└─ Another component already fetching? → NO, subscribe to existing fetch
```

**The Deduplication Logic:**

RTK Query maintains a **request tracking map**:
```javascript
{
  "getPosts(undefined)": {
    requestId: "xyz123",
    status: "pending",
    subscribers: ["componentA", "componentB"]
  }
}
```

If Component B calls `useGetPostsQuery()` while Component A's request is pending:
- Component B subscribes to the SAME request
- Only ONE network request is made
- Both components receive the result

**Stage 4: Network Request & Response Handling**

```javascript
// RTK Query internally does:
1. dispatch({ type: 'api/executeQuery/pending', meta: { requestId, arg } })
2. const response = await baseQuery(queryFn(arg))
3. if (response.data) {
     dispatch({ 
       type: 'api/executeQuery/fulfilled',
       payload: response.data,
       meta: { requestId, arg }
     })
   } else {
     dispatch({
       type: 'api/executeQuery/rejected',
       payload: response.error,
       meta: { requestId, arg }
     })
   }
```

**Stage 5: Cache Storage & Normalization**

The reducer handles the fulfilled action:
```javascript
state.queries[cacheKey] = {
  status: 'fulfilled',
  data: normalize(response.data),
  requestId: 'xyz123',
  fulfilledTimeStamp: Date.now(),
  originalArgs: args,
  // ... metadata
}
```

**Normalization** (if configured):
- Transforms response to consistent shape
- Extracts IDs for relationship tracking
- Applies `transformResponse` if defined

**Stage 6: Subscription Notification**

```
Cache updated → Redux state changed → React re-renders → Hooks recompute
```

All subscribers (components using this cache key) receive the update simultaneously through React's batching mechanism.

**Stage 7: Cleanup (on unmount)**

```javascript
Component unmounts
  ↓
Subscription removed
  ↓
Subscriber count--
  ↓
if (subscriberCount === 0) {
  setTimeout(() => {
    if (still no subscribers) {
      delete cache entry
    }
  }, keepUnusedDataFor * 1000)
}
```

**The Subscription Model:**

This is brilliant. If 3 components all call `useGetPostsQuery()`:
- Only ONE network request is made
- All 3 components subscribe to the same cache entry
- When the last component unmounts, a timer starts
- After `keepUnusedDataFor` expires, cache is garbage collected

**Why This Matters:**

Consider a dashboard with 10 widgets, each displaying user data:
- **Traditional approach**: 10 separate requests, 10 copies of data in memory
- **RTK Query**: 1 request, 1 copy of data, 10 subscriptions

**Memory savings**: O(1) vs O(n)
**Network savings**: 1 request vs n requests
**Consistency**: All widgets always show same data

#### 2. Cache Keys and Tags - The Indexing System

**Cache Keys: The Identity System**

Every query generates a cache key. This is RTK Query's indexing mechanism.

**Default Key Generation:**

```javascript
cacheKey = serializeQueryArgs({
  endpointName: 'getPosts',
  queryArgs: { filter: 'recent', page: 1 }
})

// Results in: "getPosts({\"filter\":\"recent\",\"page\":1})"
```

**Key Properties:**

1. **Deterministic**: Same args always produce same key
2. **Unique**: Different args produce different keys
3. **Serializable**: Can be used as object keys

**This means:**
- `getPost(5)` and `getPost(10)` are separate cache entries
- Calling `getPost(5)` twice reuses the cache
- Changing arguments creates a new cache entry

**The Cache Key Space:**

Think of RTK Query's cache as a **hash map**:

```javascript
{
  "getPosts(undefined)": { data: [...], status: 'fulfilled', ... },
  "getPost(5)": { data: {...}, status: 'fulfilled', ... },
  "getPost(10)": { data: {...}, status: 'fulfilled', ... },
  "searchPosts({\"q\":\"redux\",\"page\":1})": { data: [...], ... }
}
```

Each key is independent. Updating one doesn't affect others (unless you use tags).

**Tags: The Relationship System**

Tags solve the **cache invalidation problem**. When you update data on the server, which cached queries should be refetched?

**The Problem:**
```
User edits Post #5
→ Need to refetch: getPost(5)
→ Also need to refetch: getPosts() (list contains this post)
→ Also need to refetch: getUserPosts(userId) (if user owns this post)
```

Without tags, you'd need to manually invalidate each cache key. With tags, you declare relationships.

**Tags: The Invalidation System:**

Tags link queries and mutations using a **many-to-many relationship**:

```javascript
// Query "provides" tags (declares what it contains)
getPost: builder.query({
  providesTags: (result, error, id) => [{ type: 'Post', id }]
})

getPosts: builder.query({
  providesTags: (result) => 
    result 
      ? [...result.map(({ id }) => ({ type: 'Post', id })), 'Post']
      : ['Post']
})

// Mutation "invalidates" tags (declares what it affects)
updatePost: builder.mutation({
  invalidatesTags: (result, error, { id }) => [{ type: 'Post', id }]
})
```

**How Invalidation Works:**

```
1. Mutation completes (updatePost with id: 5)
2. RTK Query checks: "What tags does this invalidate?"
   → [{ type: 'Post', id: 5 }]
3. RTK Query searches cache: "What queries provide these tags?"
   → getPost(5) provides { type: 'Post', id: 5 }
   → getPosts() provides { type: 'Post', id: 5 }
4. Mark these queries as "stale"
5. If queries have active subscribers, refetch immediately
6. If no subscribers, mark for refetch when subscribed again
```

**Tag Granularity:**

You can invalidate at different levels:

```javascript
// Invalidate specific post
[{ type: 'Post', id: 5 }]  // Only queries providing this exact tag

// Invalidate all posts
['Post']  // All queries providing any 'Post' tag

// Invalidate multiple types
['Post', 'Comment', 'User']  // Multiple entity types
```

**The Graph Model:**

Think of tags as creating a **dependency graph**:

```
         updatePost(5)
              ↓ invalidates
        { type: 'Post', id: 5 }
         ↙             ↘
    getPost(5)      getPosts()
      refetch         refetch
```

**List Invalidation Pattern:**

Common pattern for lists:

```javascript
providesTags: (result) =>
  result
    ? [
        // Each item gets its own tag
        ...result.map(({ id }) => ({ type: 'Post', id })),
        // List itself gets a tag
        { type: 'Post', id: 'LIST' }
      ]
    : [{ type: 'Post', id: 'LIST' }]

// Strategies:
deletePost: invalidatesTags: [{ type: 'Post', id: 'LIST' }]  // Refetch list
updatePost: invalidatesTags: (r, e, { id }) => [{ type: 'Post', id }]  // Specific item
createPost: invalidatesTags: [{ type: 'Post', id: 'LIST' }]  // New item affects list
```

When a mutation invalidates a tag, ALL queries providing that tag automatically refetch (if they have active subscribers).

#### 3. Automatic Refetching and Polling

**The Freshness Problem:**

Web applications face a fundamental challenge: **server state becomes stale**. Data you fetched 5 minutes ago may no longer be accurate.

**Strategies for Maintaining Freshness:**

1. **Never cache** (always refetch): Fresh but wasteful
2. **Cache forever**: Efficient but stale
3. **Smart refetching** (RTK Query's approach): Balance freshness and efficiency

**When queries automatically refetch:**
- `refetchOnMount`: When component mounts (default: true)
- `refetchOnFocus`: When window regains focus (default: false)  
- `refetchOnReconnect`: When internet reconnects (default: false)

**The Refetch Decision Tree:**

```
Component mounts with useQuery
  ↓
Cache exists for this query?
  ├─ NO → Always fetch
  └─ YES → Check refetchOnMount
      ├─ false → Use cache, don't fetch
      ├─ true → Fetch regardless of cache age
      └─ number (e.g., 60) → Fetch if cache older than 60s
```

**Polling with `pollingInterval`:**

Polling implements **time-based** invalidation. Instead of event-driven refetching (mount, focus, reconnect), polling refetches on a timer.

**The Polling Loop:**

```javascript
const { data } = useGetStockPriceQuery('AAPL', {
  pollingInterval: 5000, // 5000ms = 5 seconds
})

// Internally:
useEffect(() => {
  const intervalId = setInterval(() => {
    if (hasActiveSubscribers && !document.hidden) {
      refetch()
    }
  }, pollingInterval)
  
  return () => clearInterval(intervalId)
}, [pollingInterval])
```

**Advanced Polling Options:**

```javascript
const { data } = useGetDashboardQuery(undefined, {
  pollingInterval: 3000,
  skipPollingIfUnfocused: true, // Stop when tab loses focus
  refetchOnFocus: true, // Refetch when tab regains focus
})
```

**The Visibility Algorithm:**

```javascript
function shouldPoll(config) {
  if (!config.pollingInterval) return false
  if (subscriberCount === 0) return false
  if (config.skipPollingIfUnfocused && document.hidden) return false
  return true
}
```

**Key Behaviors:**
- Polling only happens while component is mounted (has subscribers)
- Multiple components with same query share the polling interval (1 request, not n)
- Polling stops when all components unmount
- `skipPollingIfUnfocused` saves bandwidth and battery by pausing when tab is hidden

**When to Use Polling:**

✅ **Good use cases:**
- Stock prices, crypto prices (fast-changing numeric data)
- Dashboard metrics that update every few seconds
- Order status tracking ("Your order is being prepared...")
- Live availability (seats remaining, inventory count)
- Bidding/auction systems

❌ **Avoid polling when:**
- Data changes infrequently (use `refetchOnMount` or manual refetch)
- Real-time updates are critical (use WebSockets for sub-second updates)
- You have many concurrent users (server load scales with users × poll rate)
- Mobile users on limited data plans (each poll = network request = battery drain)

**Performance Considerations:**

The **load multiplier effect**:

```
Users: 100
Poll interval: 5 seconds
Requests per minute: 100 × (60 / 5) = 1,200 requests/minute = 72,000/hour
```

For 1,000 users: 720,000 requests/hour

**Cost Analysis:**

1. **Server Load**: CPU cycles, database queries, bandwidth
2. **Client Cost**: Battery drain (network radio activation), data usage
3. **User Experience**: Stale data vs. freshness vs. performance

**Mitigation Strategies:**

**Strategy 1: Adaptive Polling (Context-Aware)**
```javascript
const [interval, setInterval] = useState(5000)

useEffect(() => {
  const handleVisibility = () => {
    // Slow down when tab is hidden
    setInterval(document.hidden ? 30000 : 5000)
  }
  document.addEventListener('visibilitychange', handleVisibility)
  return () => document.removeEventListener('visibilitychange', handleVisibility)
}, [])

const { data } = useGetDataQuery(undefined, { pollingInterval: interval })
```

**Strategy 2: User-Controlled Polling**
```javascript
// Let users choose their update frequency
const { data } = useGetDataQuery(undefined, {
  pollingInterval: userWantsRealtime ? 3000 : 0, // 0 disables polling
})
```

**Strategy 3: Built-in Optimization**
```javascript
// Use RTK Query's built-in optimizations
const { data } = useGetDataQuery(undefined, {
  pollingInterval: 5000,
  skipPollingIfUnfocused: true,  // Automatic pause
})
```

**Strategy 4: Exponential Backoff**
```javascript
// Slow down polling over time if no changes detected
const [interval, setInterval] = useState(3000)
const [noChangeCount, setNoChangeCount] = useState(0)

useEffect(() => {
  if (noChangeCount > 5) {
    setInterval(Math.min(interval * 1.5, 30000))  // Max 30s
  }
}, [noChangeCount])
```

**Polling vs. Alternatives:**

| Approach | Latency | Server Load | Complexity | Best For |
|----------|---------|-------------|------------|----------|
| **Polling (short)** | 1-5s | High | Low | Simple dashboards |
| **Polling (long)** | 30-60s | Medium | Low | Background sync |
| **WebSockets** | <100ms | Low (persistent) | High | Chat, real-time collab |
| **Server-Sent Events** | <1s | Low | Medium | One-way live updates |
| **Manual Refetch** | On-demand | Low | Low | User-triggered updates |
| **Tag Invalidation** | On-mutation | Low | Medium | CRUD apps |

**The Polling Decision Matrix:**

```
Data change frequency?
├─ < 1 second → WebSocket
├─ 1-10 seconds → Polling (with skipPollingIfUnfocused)
├─ 10-60 seconds → Long polling or refetchOnFocus
└─ > 1 minute → Manual refetch or mutation invalidation

User count?
├─ < 100 → Polling is fine
├─ 100-1000 → Use skipPollingIfUnfocused, consider longer intervals
└─ > 1000 → Consider WebSocket or Server-Sent Events

Mobile users?
└─ Use skipPollingIfUnfocused + refetchOnFocus (battery-friendly)
```

**Dynamic Polling Example:**

**Start/Stop Pattern:**
```javascript
const [enabled, setEnabled] = useState(false)

const { data } = useGetLiveDataQuery(undefined, {
  pollingInterval: enabled ? 2000 : 0,  // 0 = disabled
})

<button onClick={() => setEnabled(!enabled)}>
  {enabled ? '⏸ Pause' : '▶ Start'} Live Updates
</button>
```

**Multi-Rate Polling Pattern (Priority-Based):**

```javascript
// Critical data: poll fast
const { data: criticalData } = useGetCriticalQuery(undefined, {
  pollingInterval: 1000,  // Every second
})

// Normal data: moderate polling
const { data: normalData } = useGetNormalQuery(undefined, {
  pollingInterval: 10000,  // Every 10 seconds
})

// Background data: slow polling
const { data: slowData } = useGetSlowQuery(undefined, {
  pollingInterval: 60000,  // Every minute
})
```

**Conditional Polling (Smart):**

```javascript
const { data, isError } = useGetDataQuery(undefined, {
  pollingInterval: isError ? 30000 : 5000,  // Slower when errors occur
  skipPollingIfUnfocused: true,
})
```

### Resources

1. **Official Docs - Automated Re-fetching**
   - https://redux-toolkit.js.org/rtk-query/usage/automated-refetching
   - Essential reading for understanding tags

2. **Video: Cache Behavior Explained**
   - https://www.youtube.com/watch?v=gTP3nN1tZCk
   - Visual explanation of subscriptions

3. **Interactive Tutorial**
   - https://redux-toolkit.js.org/tutorials/rtk-query
   - Hands-on practice with the lifecycle

4. **Article: Cache Keys Deep Dive**
   - Search for "RTK Query cache keys" on the Redux blog
   - Understanding serializeQueryArgs

### Practice Exercise

Build a simple blog where:
- List view shows all posts
- Clicking a post shows its details
- Editing a post refetches both list and detail

**Challenge Question:** What happens if you have the list view and detail view both mounted, then edit a post? Which queries refetch and why?

---

## 🎯 Phase 3: Advanced Cache Management (Days 5-6)

### Goal
Master cache invalidation and optimization patterns

### Topics

#### 1. Tag-Based Invalidation Strategies

**Pattern 1: Invalidate entire list**
```javascript
providesTags: ['Posts'] // Simple string tag
invalidatesTags: ['Posts'] // Refetch everything with this tag
```

**Pattern 2: Granular invalidation**
```javascript
providesTags: (result) => 
  result 
    ? [...result.map(({ id }) => ({ type: 'Post', id })), 'Post']
    : ['Post']
```

Now you can invalidate:
- A specific post: `[{ type: 'Post', id: 5 }]`
- All posts: `['Post']`

#### 2. Optimistic Updates

Instead of waiting for the server, update the cache immediately:

```javascript
onQueryStarted: async (arg, { dispatch, queryFulfilled }) => {
  // Optimistically update
  const patchResult = dispatch(
    api.util.updateQueryData('getPosts', undefined, (draft) => {
      draft.push(newPost)
    })
  )
  
  try {
    await queryFulfilled
  } catch {
    patchResult.undo() // Rollback on error
  }
}
```

This uses Immer under the hood for immutable updates.

#### 3. Cache Entry Lifecycle

```
Component mounts → Query starts → Data cached
Component unmounts → Subscription removed
No subscribers? → Start keepUnusedDataFor timer (default 60s)
Timer expires → Cache entry removed
```

**Prefetching:**
Load data before the user needs it:

```javascript
dispatch(api.util.prefetch('getPosts', undefined, { force: true }))
```

### Resources

1. **Official Docs - Optimistic Updates**
   - https://redux-toolkit.js.org/rtk-query/usage/manual-cache-updates
   - Study the examples carefully

2. **Video: Advanced Cache Patterns** (Lenz Weber)
   - https://www.youtube.com/watch?v=9zySeP5vH9c
   - Real-world invalidation strategies

3. **GitHub Examples Repository**
   - https://github.com/reduxjs/redux-toolkit/tree/master/examples/query
   - Look at the "optimistic-update" example

4. **Article: Prefetching Strategies**
   - https://redux-toolkit.js.org/rtk-query/usage/prefetching
   - When and how to prefetch

### Practice Exercise

Build a Twitter-like feed where:
- New posts appear optimistically
- Failed posts show error and rollback
- Hovering over a user prefetches their profile
- Liking a post updates counts without refetching

**Question to Ponder:** When would you choose optimistic updates vs. pessimistic (wait for server)? What are the UX tradeoffs?

---

## ⚙️ Phase 4: Customization & Control (Days 7-8)

### Goal
Understand how to customize RTK Query for complex scenarios

### Topics

#### 1. Base Queries and Custom Fetch

`fetchBaseQuery` is a thin wrapper around `fetch`. But you can replace it entirely:

```javascript
const customBaseQuery = async (args, api, extraOptions) => {
  // Use axios, graphql, or anything
  // Must return { data } or { error }
}
```

**Use cases:**
- Using axios instead of fetch
- GraphQL integration
- Custom auth flows
- Request/response interceptors

#### 2. Query Transformations

Shape data before it hits the cache:

```javascript
transformResponse: (response) => {
  // Normalize nested data
  // Add computed fields
  // Filter out unnecessary data
  return normalized
}
```

#### 3. Middleware and Lifecycle Hooks

**`onQueryStarted`** runs when a query starts:
```javascript
onQueryStarted: async (arg, { dispatch, getState, queryFulfilled }) => {
  // Optimistic updates
  // Analytics tracking
  // Side effects
}
```

**`onCacheEntryAdded`** runs when a cache entry is created:
```javascript
onCacheEntryAdded: async (arg, { updateCachedData, cacheDataLoaded }) => {
  // WebSocket subscriptions
  // Streaming updates
  // Background sync
}
```

**Token Refresh Pattern:**

```javascript
const baseQueryWithReauth = async (args, api, extraOptions) => {
  let result = await baseQuery(args, api, extraOptions)
  
  if (result.error?.status === 401) {
    // Try to get a new token
    const refreshResult = await baseQuery('/refresh', api, extraOptions)
    
    if (refreshResult.data) {
      // Store new token and retry
      api.dispatch(setToken(refreshResult.data.token))
      result = await baseQuery(args, api, extraOptions)
    } else {
      // Logout user
      api.dispatch(logout())
    }
  }
  
  return result
}
```

### Resources

1. **Official Docs - Customizing Queries**
   - https://redux-toolkit.js.org/rtk-query/usage/customizing-queries
   - Comprehensive guide to all customization options

2. **Official Docs - Streaming Updates**
   - https://redux-toolkit.js.org/rtk-query/usage/streaming-updates
   - WebSocket integration patterns

3. **Video: Auth with RTK Query**
   - Search YouTube for "RTK Query authentication"
   - Multiple approaches to token management

4. **Code Example: GraphQL Base Query**
   - https://github.com/reduxjs/redux-toolkit/discussions/1163
   - Shows how to use GraphQL instead of REST

### Practice Exercise

Implement:
- Auth system with token refresh
- WebSocket connection that updates cached data in real-time
- Transform response to normalize nested user data

**Challenge Question:** Why does `onCacheEntryAdded` make more sense for WebSocket subscriptions than `onQueryStarted`? Think about component mount/unmount cycles.

---

## 🚀 Phase 5: Performance & Patterns (Days 9-10)

### Goal
Production-ready patterns and performance optimization

### Topics

#### 1. Performance Considerations

**Selector Optimization:**

By default, `useGetPostsQuery()` returns the entire result. This can cause unnecessary re-renders:

```javascript
// Re-renders when ANY post changes
const { data: posts } = useGetPostsQuery()

// Only re-renders when THIS specific post changes
const { post } = useGetPostsQuery(undefined, {
  selectFromResult: ({ data }) => ({
    post: data?.find(p => p.id === postId)
  })
})
```

**Bundle Size:**
- Code-split API slices by feature
- Use `injectEndpoints` to lazy-load endpoints
- Tree-shaking removes unused endpoints

**Memory Management:**
For large datasets:
- Use `keepUnusedDataFor` strategically (shorter for big data)
- Implement pagination instead of loading everything
- Consider normalized data structures

#### 2. Advanced Patterns

**Conditional Queries:**
```javascript
const { data } = useGetUserQuery(userId, {
  skip: !userId // Don't fetch if no userId
})
```

**Lazy Queries:**
```javascript
const [trigger, result] = useLazyGetPostQuery()

// Later...
trigger(postId)
```

**Combining RTK Query with Regular Redux:**

RTK Query data is just Redux state:

```javascript
const posts = useSelector(state => state.api.queries['getPosts(undefined)']?.data)
```

You can:
- Select RTK Query data in regular selectors
- Dispatch mutations from thunks
- Mix approaches as needed

#### 3. Testing Strategies

**Mock Service Worker (MSW) for API Mocking:**
```tsx
// src/mocks/handlers.ts
import { rest } from 'msw';

export const handlers = [
  rest.get('/api/posts', (req, res, ctx) => {
    return res(
      ctx.json({
        posts: [
          { id: 1, title: 'Test Post', content: 'Test content' },
        ],
      })
    );
  }),

  rest.post('/api/posts', async (req, res, ctx) => {
    const { title, content } = await req.json();

    return res(
      ctx.status(201),
      ctx.json({
        id: 2,
        title,
        content,
        createdAt: new Date().toISOString(),
      })
    );
  }),
];

// src/mocks/server.ts
import { setupServer } from 'msw';
import { handlers } from './handlers';

export const server = setupServer(...handlers);

// src/setupTests.ts
import { server } from './mocks/server';

beforeAll(() => server.listen());
afterEach(() => server.resetHandlers());
afterAll(() => server.close());
```

**Testing RTK Query Hooks:**
```tsx
// __tests__/features/posts/postsSlice.test.ts
import { renderHook, waitFor } from '@testing-library/react';
import { Provider } from 'react-redux';
import { store } from '@/app/store';
import { useGetPostsQuery } from '@/features/posts/postsSlice';

const wrapper = ({ children }) => (
  <Provider store={store}>{children}</Provider>
);

describe('Posts API', () => {
  it('fetches posts successfully', async () => {
    const { result } = renderHook(() => useGetPostsQuery(), { wrapper });

    await waitFor(() => {
      expect(result.current.isSuccess).toBe(true);
    });

    expect(result.current.data).toEqual({
      posts: [{ id: 1, title: 'Test Post', content: 'Test content' }],
    });
  });

  it('handles loading state', () => {
    const { result } = renderHook(() => useGetPostsQuery(), { wrapper });

    expect(result.current.isLoading).toBe(true);
    expect(result.current.data).toBeUndefined();
  });

  it('handles errors', async () => {
    // Mock a server error
    server.use(
      rest.get('/api/posts', (req, res, ctx) => {
        return res(ctx.status(500));
      })
    );

    const { result } = renderHook(() => useGetPostsQuery(), { wrapper });

    await waitFor(() => {
      expect(result.current.isError).toBe(true);
    });

    expect(result.current.error).toBeDefined();
  });
});
```

**Testing Cache Behavior:**
```tsx
// __tests__/cache-behavior.test.ts
import { renderHook, act } from '@testing-library/react';
import { Provider } from 'react-redux';
import { store } from '@/app/store';
import { useGetPostQuery, useUpdatePostMutation } from '@/features/posts/postsSlice';

const wrapper = ({ children }) => (
  <Provider store={store}>{children}</Provider>
);

describe('Cache Invalidation', () => {
  it('invalidates cache after mutation', async () => {
    const { result: queryResult } = renderHook(() => useGetPostQuery(1), { wrapper });
    const { result: mutationResult } = renderHook(() => useUpdatePostMutation(), { wrapper });

    // Wait for initial data
    await waitFor(() => {
      expect(queryResult.current.isSuccess).toBe(true);
    });

    const initialData = queryResult.current.data;

    // Perform mutation
    act(() => {
      mutationResult.current[0]({
        id: 1,
        title: 'Updated Title',
      });
    });

    // Wait for mutation to complete
    await waitFor(() => {
      expect(mutationResult.current[1].isSuccess).toBe(true);
    });

    // Query should refetch and get updated data
    await waitFor(() => {
      expect(queryResult.current.data).not.toBe(initialData);
    });
  });
});
```

**Testing Custom Base Queries:**
```tsx
// __tests__/baseQuery.test.ts
import { fetchBaseQuery } from '@reduxjs/toolkit/query';
import { createApi } from '@reduxjs/toolkit/query/react';

const baseQuery = fetchBaseQuery({ baseUrl: '/api' });

describe('Base Query', () => {
  it('adds authorization header', async () => {
    const customBaseQuery = fetchBaseQuery({
      baseUrl: '/api',
      prepareHeaders: (headers) => {
        headers.set('authorization', 'Bearer test-token');
        return headers;
      },
    });

    const result = await customBaseQuery(
      { url: '/test', method: 'GET' },
      { getState: () => ({}) },
      {}
    );

    // MSW will intercept and verify headers
    expect(result).toBeDefined();
  });
});
```

**Common Pitfalls:**
- **Error**: "connect ECONNREFUSED" - MSW server not started in tests
- **Cause**: Missing server setup in test files
- **Solution**: Ensure MSW server is started before tests and handlers are reset

- **Error**: "TypeError: Cannot read property 'data' of undefined" - incorrect hook usage
- **Cause**: Not wrapping components with Provider or incorrect hook usage
- **Solution**: Always use Provider wrapper in tests and destructure hooks correctly

- **Error**: Cache not invalidating in tests
- **Cause**: Not waiting for mutations to complete before checking cache updates
- **Solution**: Use proper async testing patterns and wait for mutation fulfillment

💡 **Best Practice:** Use realistic mock data that matches your API schema exactly.

💡 **Best Practice:** Test both success and error scenarios for all endpoints.

💡 **Best Practice:** Reset MSW handlers between tests to avoid state pollution.

### Resources

1. **Official Docs - Performance**
   - https://redux-toolkit.js.org/rtk-query/usage/usage-with-typescript#optimizing-query-returns
   - selectFromResult patterns

2. **Official Docs - Code Splitting**
   - https://redux-toolkit.js.org/rtk-query/usage/code-splitting
   - Lazy loading strategies

3. **Article: Testing RTK Query**
   - https://redux-toolkit.js.org/rtk-query/usage/testing
   - Mock strategies and patterns

4. **Video: Performance Optimization in Redux**
   - Mark Erikson's talks on Redux performance
   - Principles apply to RTK Query

### Practice Exercise

Refactor your previous examples to:
- Use `selectFromResult` to prevent unnecessary re-renders
- Implement conditional queries based on user permissions
- Write tests that mock the API slice

**Performance Challenge:** Profile your app with React DevTools. Can you identify components re-rendering unnecessarily due to RTK Query data changes?

---

## 🌐 Phase 6: Real-World Integration (Days 11-12)

### Goal
Put it all together in realistic scenarios

### Topics

#### 1. Common Patterns

**Pagination:**
```javascript
getPostsPaginated: builder.query({
  query: (page) => `/posts?page=${page}`,
  // Merge pages together
  serializeQueryArgs: ({ endpointName }) => endpointName,
  merge: (currentCache, newItems) => {
    currentCache.push(...newItems)
  },
  forceRefetch({ currentArg, previousArg }) {
    return currentArg !== previousArg
  }
})
```

**Infinite Scroll:**
```javascript
const [page, setPage] = useState(1)
const { data, isFetching } = useGetPostsPaginatedQuery(page)
// Increment page on scroll
```

**Debounced Search:**
```javascript
const [search, setSearch] = useState('')
const debouncedSearch = useDebounce(search, 500)
const { data } = useSearchPostsQuery(debouncedSearch, {
  skip: !debouncedSearch
})
```

**File Upload with Progress:**
Custom base query with axios for progress tracking

**Background Sync:**
Keep data fresh in the background with polling

#### 1.5 GraphQL Integration Patterns

RTK Query works excellently with GraphQL APIs, providing caching and state management for GraphQL queries and mutations.

**GraphQL Base Query:**
```tsx
// lib/graphql-base-query.ts
import { fetchBaseQuery } from '@reduxjs/toolkit/query/react';
import { print } from 'graphql/language/printer';

export const graphqlBaseQuery = ({ baseUrl }: { baseUrl: string }) => {
  const baseQuery = fetchBaseQuery({
    baseUrl,
    prepareHeaders: (headers) => {
      headers.set('Content-Type', 'application/json');
      return headers;
    },
  });

  return async (args: { document: any; variables?: any }, api, extraOptions) => {
    const result = await baseQuery(
      {
        url: '',
        method: 'POST',
        body: {
          query: print(args.document),
          variables: args.variables,
        },
      },
      api,
      extraOptions
    );

    if (result.error) {
      return { error: result.error };
    }

    return { data: result.data };
  };
};
```

**API Slice with GraphQL:**
```tsx
// features/api/graphqlApi.ts
import { createApi } from '@reduxjs/toolkit/query/react';
import { graphqlBaseQuery } from '@/lib/graphql-base-query';
import { gql } from '@apollo/client';

export const graphqlApi = createApi({
  reducerPath: 'graphqlApi',
  baseQuery: graphqlBaseQuery({ baseUrl: '/graphql' }),
  endpoints: (builder) => ({
    getPosts: builder.query({
      query: () => ({
        document: gql`
          query GetPosts {
            posts {
              id
              title
              content
              author {
                id
                name
              }
            }
          }
        `,
      }),
      transformResponse: (response: any) => response.data.posts,
    }),

    createPost: builder.mutation({
      query: ({ title, content }) => ({
        document: gql`
          mutation CreatePost($input: CreatePostInput!) {
            createPost(input: $input) {
              id
              title
              content
            }
          }
        `,
        variables: { input: { title, content } },
      }),
      transformResponse: (response: any) => response.data.createPost,
      invalidatesTags: ['Posts'],
    }),
  }),
});

export const { useGetPostsQuery, useCreatePostMutation } = graphqlApi;
```

**Using in Components:**
```tsx
// components/PostsList.tsx
import { useGetPostsQuery } from '@/features/api/graphqlApi';

export function PostsList() {
  const { data: posts, isLoading, error } = useGetPostsQuery();

  if (isLoading) return <div>Loading...</div>;
  if (error) return <div>Error: {error.message}</div>;

  return (
    <div>
      {posts?.map((post) => (
        <article key={post.id}>
          <h2>{post.title}</h2>
          <p>{post.content}</p>
          <small>By {post.author.name}</small>
        </article>
      ))}
    </div>
  );
}
```

**Common Pitfalls:**
- **Error**: "TypeError: print is not a function" - missing graphql dependency
- **Cause**: Not installing @apollo/client or graphql packages
- **Solution**: Install required packages: `npm install @apollo/client graphql`

- **Error**: "Invariant Violation: Apollo Client not found" - incorrect setup
- **Cause**: Mixing Apollo Client with RTK Query GraphQL base query
- **Solution**: Choose one approach; don't use both simultaneously

💡 **Best Practice:** Use GraphQL fragments to share field selections across queries.

💡 **Best Practice:** Leverage RTK Query's caching with GraphQL's precise data fetching.

#### 2. Error Handling

**Global Error Handling:**
```javascript
const errorLogger = (api) => (next) => (action) => {
  if (action.type.endsWith('/rejected')) {
    console.error('API Error:', action)
    // Show toast notification
  }
  return next(action)
}
```

**Retry Logic:**
```javascript
const baseQueryWithRetry = retry(
  fetchBaseQuery({ baseUrl: '/api' }),
  { maxRetries: 3 }
)
```

#### 3. DevTools and Debugging
- Redux DevTools with RTK Query
- Understanding the generated actions
- Debugging cache issues

### Resources

1. **Official Docs - Pagination**
   - https://redux-toolkit.js.org/rtk-query/usage/pagination
   - Multiple pagination strategies

2. **Official Docs - Error Handling**
   - https://redux-toolkit.js.org/rtk-query/usage/error-handling
   - Global and local patterns

3. **GitHub Discussions**
   - https://github.com/reduxjs/redux-toolkit/discussions
   - Search for specific patterns (file upload, infinite scroll, etc.)

4. **Real-World Examples Repository**
   - https://github.com/reduxjs/redux-toolkit/tree/master/examples
   - Full applications using RTK Query

5. **Redux DevTools**
   - https://github.com/reduxjs/redux-devtools
   - Learn to debug RTK Query state

### Final Project Ideas

Build one of these:
1. **Social media feed** with infinite scroll, real-time updates, optimistic posts
2. **E-commerce site** with product search, cart, checkout flow
3. **Dashboard** with multiple data sources, charts, real-time WebSocket data
4. **File manager** with upload progress, folder navigation, file preview

**Integration Challenge:** Can you combine all the patterns you've learned into one cohesive application?

---

## Common Pitfalls & Troubleshooting

### Cache Management Issues

**Problem: "Data not updating after mutation"**
- **Error**: UI shows stale data despite successful API calls
- **Cause**: Missing `invalidatesTags` in mutation or incorrect tag matching
- **Solution**: Ensure mutations specify `invalidatesTags` and queries provide matching tags

**Problem: Memory leaks from subscriptions**
- **Error**: Unmounted components still receiving updates
- **Cause**: Not properly unsubscribing from queries in cleanup
- **Solution**: RTK Query handles this automatically; check for manual subscription management

**Problem: Unexpected refetches**
- **Error**: Queries refetching when they shouldn't
- **Cause**: Incorrect `refetchOnMountOrArgChange` settings
- **Solution**: Fine-tune refetch policies per endpoint

### Query & Mutation Problems

**Problem: "Hook not found" errors**
- **Error**: `useGetPostsQuery is not a function`
- **Cause**: API slice not properly exported or imported
- **Solution**: Check exports and ensure API slice is added to store

**Problem: Infinite loading states**
- **Error**: Queries stuck in `isLoading: true`
- **Cause**: Base query not returning proper `{ data }` or `{ error }` format
- **Solution**: Ensure base query always returns the correct response format

**Problem: TypeScript errors with query responses**
- **Error**: `Property 'data' does not exist on type`
- **Cause**: Incorrect typing of API responses
- **Solution**: Define proper TypeScript interfaces for API responses

### Performance & Bundle Issues

**Problem: Large bundle sizes**
- **Error**: App bundle significantly increased after adding RTK Query
- **Cause**: Not code-splitting API slices
- **Solution**: Use `injectEndpoints` and dynamic imports for feature-based splitting

**Problem: Too many re-renders**
- **Error**: Components re-rendering on every state change
- **Cause**: Not using `selectFromResult` for derived data
- **Solution**: Extract specific data with selectors to prevent unnecessary updates

**Problem: Cache growing too large**
- **Error**: Memory usage increasing over time
- **Cause**: `keepUnusedDataFor` set too high or cache not garbage collected
- **Solution**: Adjust `keepUnusedDataFor` and ensure proper cleanup

### Integration Challenges

**Problem: RTK Query with other state libraries**
- **Error**: State conflicts or unexpected behavior
- **Cause**: Mixing RTK Query with Zustand/SWR without proper boundaries
- **Solution**: Use RTK Query for server state, other libraries for client state

**Problem: WebSocket integration not working**
- **Error**: Real-time updates not appearing
- **Cause**: Incorrect `onCacheEntryAdded` implementation
- **Solution**: Ensure proper async handling and cleanup in WebSocket callbacks

**Problem: Optimistic updates failing**
- **Error**: UI shows success but server rejects
- **Cause**: Not handling rollback in `onQueryStarted`
- **Solution**: Always implement proper error handling and rollback logic

### Testing Difficulties

**Problem: Tests failing with network errors**
- **Error**: `fetch` calls failing in test environment
- **Cause**: Not mocking API calls properly
- **Solution**: Use MSW or similar for comprehensive API mocking

**Problem: Cache state not resetting between tests**
- **Error**: Tests affecting each other through shared cache
- **Cause**: Not resetting RTK Query state between tests
- **Solution**: Clear cache or use isolated store instances per test

**Problem: Async testing timeouts**
- **Error**: Tests timing out waiting for data
- **Cause**: Not properly waiting for query resolution
- **Solution**: Use `waitFor` with proper conditions

💡 **Best Practice:** Use Redux DevTools to inspect cache state and debug issues.

💡 **Best Practice:** Start with simple queries and gradually add complexity.

💡 **Best Practice:** Always handle error states in your UI components.

---

## 📅 Suggested Daily Time Investment

- **30-45 minutes**: Reading/watching tutorials
- **60-90 minutes**: Hands-on coding practice
- **15-30 minutes**: Reflection and note-taking

**Total commitment:** ~2-3 hours per day for 12 days

---

## ⚙️ Complete RTK Query Configuration Reference

This comprehensive guide covers every configuration option available in RTK Query, organized by where they're used.

### `createApi` Configuration Options

These options are passed when you create your API slice:

#### `baseQuery` (required)
**Type:** `BaseQueryFn`

**What it does:** Defines how RTK Query makes HTTP requests. This is the foundation of all your API calls.

**Default:** None (you must provide one)

**Common values:**
```javascript
// Using fetchBaseQuery (most common)
baseQuery: fetchBaseQuery({ baseUrl: 'https://api.example.com' })

// Custom base query
baseQuery: async (args, api, extraOptions) => {
  // Your custom logic
  return { data: result }
}
```

**Deep dive:** 
- This function is called for EVERY query and mutation
- It receives the endpoint definition and must return `{ data }` or `{ error }`
- You can wrap it to add auth, retries, error handling, etc.
- Common pattern: wrap `fetchBaseQuery` with custom logic

**Real-world use:**
```javascript
const baseQueryWithAuth = async (args, api, extraOptions) => {
  const result = await fetchBaseQuery({
    baseUrl: '/api',
    prepareHeaders: (headers, { getState }) => {
      const token = getState().auth.token
      if (token) {
        headers.set('authorization', `Bearer ${token}`)
      }
      return headers
    }
  })(args, api, extraOptions)
  
  return result
}
```

---

#### `endpoints` (required)
**Type:** `(builder: EndpointBuilder) => EndpointDefinitions`

**What it does:** Defines all your API endpoints (queries and mutations).

**Structure:**
```javascript
endpoints: (builder) => ({
  getUser: builder.query({
    query: (id) => `/users/${id}`,
  }),
  updateUser: builder.mutation({
    query: ({ id, ...patch }) => ({
      url: `/users/${id}`,
      method: 'PATCH',
      body: patch,
    }),
  }),
})
```

**Key concepts:**
- `builder.query()` for GET requests (reading data)
- `builder.mutation()` for POST/PUT/PATCH/DELETE (changing data)
- Each endpoint becomes a hook: `useGetUserQuery`, `useUpdateUserMutation`

---

#### `reducerPath`
**Type:** `string`

**Default:** `'api'`

**What it does:** The key in your Redux store where RTK Query state lives.

```javascript
reducerPath: 'myApi', // Instead of default 'api'
```

**When to change:**
- Multiple API slices in one app (each needs unique path)
- Avoiding naming conflicts

**Store structure:**
```javascript
{
  myApi: {
    queries: { ... },
    mutations: { ... },
    provided: { ... },
    subscriptions: { ... },
  }
}
```

---

#### `tagTypes`
**Type:** `string[]`

**Default:** `[]`

**What it does:** Defines the types of tags used for cache invalidation across your API.

```javascript
tagTypes: ['Post', 'User', 'Comment'],
```

**Why it matters:**
- Must declare all tag types upfront
- TypeScript will validate tag usage
- Enables intelligent cache invalidation

**Mental model:** Think of tag types as "categories of data" in your API.

---

#### `keepUnusedDataFor`
**Type:** `number` (seconds)

**Default:** `60`

**What it does:** How long to keep cached data after all components unsubscribe.

```javascript
keepUnusedDataFor: 300, // 5 minutes
```

**Behavior:**
```
Component mounts → Query runs → Data cached
Component unmounts → Start timer (keepUnusedDataFor)
Timer expires → Remove from cache
Component re-mounts within timer → Reuse cache (no new request)
```

**When to adjust:**
- Increase for expensive queries that might be reused
- Decrease for large datasets to save memory
- Set to 0 for always-fresh data (refetch every time)

---

#### `refetchOnMountOrArgChange`
**Type:** `boolean | number`

**Default:** `false`

**What it does:** Controls whether to refetch when component mounts or query args change.

**Options:**
```javascript
// Boolean mode
refetchOnMountOrArgChange: true  // Always refetch on mount
refetchOnMountOrArgChange: false // Never refetch if cached

// Number mode (seconds)
refetchOnMountOrArgChange: 60 // Refetch if cache is older than 60s
```

**Use cases:**
- `true`: When data changes frequently
- `false`: When data is stable and bandwidth is limited
- `60`: Sweet spot - fresh data without excessive requests

---

#### `refetchOnFocus`
**Type:** `boolean`

**Default:** `false`

**What it does:** Refetch when browser tab/window regains focus.

```javascript
refetchOnFocus: true,
```

**User scenario:**
1. User opens your app
2. User switches to another tab for 10 minutes
3. User returns to your app
4. Data automatically refetches

**Best for:** Dashboards, live data, collaborative tools

---

#### `refetchOnReconnect`
**Type:** `boolean`

**Default:** `false`

**What it does:** Refetch when internet connection is restored.

```javascript
refetchOnReconnect: true,
```

**User scenario:**
1. User goes through a tunnel (loses connection)
2. App loses internet
3. Connection restored
4. Data automatically refetches

**Best for:** Mobile apps, real-time applications

---

### `fetchBaseQuery` Configuration Options

These options configure the built-in fetch wrapper:

#### `baseUrl`
**Type:** `string`

**What it does:** Prepended to all endpoint URLs.

```javascript
fetchBaseQuery({ baseUrl: 'https://api.example.com/v1' })

// Endpoint: query: () => '/users'
// Actual URL: https://api.example.com/v1/users
```

---

#### `prepareHeaders`
**Type:** `(headers: Headers, api: { getState, extra, endpoint, type, forced }) => Headers | void`

**What it does:** Modify headers before every request.

```javascript
prepareHeaders: (headers, { getState }) => {
  const token = getState().auth.token
  
  if (token) {
    headers.set('authorization', `Bearer ${token}`)
  }
  
  headers.set('Accept', 'application/json')
  
  return headers
}
```

**Common uses:**
- Add authentication tokens
- Set content types
- Add API keys
- Add custom headers

---

#### `fetchFn`
**Type:** `typeof fetch`

**Default:** `window.fetch`

**What it does:** Replace the fetch implementation.

```javascript
fetchFn: customFetch, // Use a polyfill or wrapper
```

**Use cases:**
- Node.js environments (no native fetch)
- Custom fetch implementations
- Request interceptors

---

#### `paramsSerializer`
**Type:** `(params: Record<string, any>) => string`

**What it does:** Customize how query parameters are serialized.

```javascript
paramsSerializer: (params) => {
  return new URLSearchParams(params).toString()
}
```

---

#### `timeout`
**Type:** `number` (milliseconds)

**Default:** None

**What it does:** Abort requests that take too long.

```javascript
fetchBaseQuery({ 
  baseUrl: '/api',
  timeout: 10000, // 10 seconds
})
```

---

### Endpoint Configuration Options

Options for individual `builder.query()` and `builder.mutation()`:

#### `query`
**Type:** `(arg) => string | FetchArgs`

**What it does:** Defines the request details for this endpoint.

**Simple string:**
```javascript
query: (id) => `/users/${id}`
```

**Full object:**
```javascript
query: (arg) => ({
  url: `/users/${arg.id}`,
  method: 'POST',
  body: arg.data,
  headers: { 'Content-Type': 'application/json' },
  params: { include: 'profile' },
})
```

---

#### `transformResponse`
**Type:** `(response, meta, arg) => any`

**What it does:** Transform the raw response before it's cached.

```javascript
transformResponse: (response) => {
  // Unwrap nested data
  return response.data
  
  // Add computed fields
  return response.map(item => ({
    ...item,
    fullName: `${item.firstName} ${item.lastName}`
  }))
  
  // Normalize data structure
  return normalizeData(response)
}
```

**Runs once:** When data first arrives from server

---

#### `transformErrorResponse`
**Type:** `(response, meta, arg) => any`

**What it does:** Transform error responses.

```javascript
transformErrorResponse: (response) => {
  return {
    status: response.status,
    message: response.data?.message || 'Unknown error',
    errors: response.data?.errors || []
  }
}
```

---

#### `providesTags`
**Type:** `(result, error, arg) => Tag[]`

**What it does:** Declares what cache tags this query provides.

**Patterns:**

```javascript
// Simple: entire list
providesTags: ['Posts']

// With IDs: individual items + list
providesTags: (result) =>
  result
    ? [
        ...result.map(({ id }) => ({ type: 'Post', id })),
        { type: 'Post', id: 'LIST' }
      ]
    : [{ type: 'Post', id: 'LIST' }]

// Conditional: based on args
providesTags: (result, error, userId) => [
  { type: 'Post', id: 'LIST' },
  { type: 'UserPosts', id: userId }
]
```

**Mental model:** "This query provides data about these tags"

---

#### `invalidatesTags`
**Type:** `(result, error, arg) => Tag[]`

**What it does:** Declares what cache tags this mutation invalidates.

```javascript
// Invalidate all posts
invalidatesTags: ['Posts']

// Invalidate specific post
invalidatesTags: (result, error, { id }) => [
  { type: 'Post', id }
]

// Invalidate multiple tags
invalidatesTags: (result, error, arg) => [
  { type: 'Post', id: arg.postId },
  { type: 'Comment', id: 'LIST' },
  'User' // All User queries
]
```

**Mental model:** "This mutation makes these tags stale"

---

#### `onQueryStarted`
**Type:** `async (arg, api) => void`

**What it does:** Runs when query/mutation starts, before completion.

**API object contains:**
- `dispatch`: Redux dispatch
- `getState`: Get Redux state
- `extra`: Extra middleware arg
- `requestId`: Unique request ID
- `queryFulfilled`: Promise that resolves when query completes
- `getCacheEntry`: Get current cache value
- `updateCachedData`: Update cache immediately

**Use cases:**

```javascript
// Optimistic updates
onQueryStarted: async (arg, { dispatch, queryFulfilled }) => {
  const patchResult = dispatch(
    api.util.updateQueryData('getPosts', undefined, (draft) => {
      draft.push(arg)
    })
  )
  
  try {
    await queryFulfilled
  } catch {
    patchResult.undo()
  }
}

// Side effects
onQueryStarted: async (arg, { dispatch, queryFulfilled }) => {
  await queryFulfilled
  dispatch(showSuccessToast('Post created!'))
}

// Analytics
onQueryStarted: async (arg, { queryFulfilled }) => {
  try {
    const { data } = await queryFulfilled
    analytics.track('Query Success', { endpoint: 'getUser' })
  } catch (error) {
    analytics.track('Query Error', { error })
  }
}
```

---

#### `onCacheEntryAdded`
**Type:** `async (arg, api) => void`

**What it does:** Runs when a new cache entry is created. Persists as long as cache entry exists.

**API object contains:**
- `cacheDataLoaded`: Promise resolving when initial data loads
- `cacheEntryRemoved`: Promise resolving when cache is removed
- `updateCachedData`: Update cache
- `dispatch`, `getState`, etc.

**Perfect for:**

```javascript
// WebSocket subscriptions
onCacheEntryAdded: async (
  arg,
  { updateCachedData, cacheDataLoaded, cacheEntryRemoved }
) => {
  // Wait for initial data
  await cacheDataLoaded
  
  // Create WebSocket connection
  const ws = new WebSocket('ws://api.example.com')
  
  ws.addEventListener('message', (event) => {
    const data = JSON.parse(event.data)
    
    updateCachedData((draft) => {
      // Update cache with WebSocket data
      Object.assign(draft, data)
    })
  })
  
  // Cleanup when cache entry is removed
  await cacheEntryRemoved
  ws.close()
}

// Streaming data
onCacheEntryAdded: async (arg, { updateCachedData, cacheDataLoaded }) => {
  await cacheDataLoaded
  
  const eventSource = new EventSource('/api/stream')
  
  eventSource.onmessage = (event) => {
    updateCachedData((draft) => {
      draft.items.push(JSON.parse(event.data))
    })
  }
}
```

**Key difference from `onQueryStarted`:**
- `onQueryStarted`: Runs on every query call
- `onCacheEntryAdded`: Runs once per cache entry (perfect for subscriptions)

---

#### `serializeQueryArgs`
**Type:** `(args) => string`

**What it does:** Customize how query arguments become cache keys.

**Default behavior:**
```javascript
// args: { id: 5, include: 'profile' }
// cache key: "getUser({\"id\":5,\"include\":\"profile\"})"
```

**Custom serialization:**
```javascript
serializeQueryArgs: ({ endpointName, queryArgs }) => {
  // Ignore certain args for caching
  const { page, ...rest } = queryArgs
  return `${endpointName}(${JSON.stringify(rest)})`
}
```

**Use case:** Merge paginated results
```javascript
serializeQueryArgs: ({ endpointName }) => {
  // All pages share same cache key
  return endpointName
},
merge: (currentCache, newItems) => {
  currentCache.push(...newItems)
}
```

---

#### `merge`
**Type:** `(currentCache, newData, otherArgs) => void`

**What it does:** Merge new data into existing cache.

**Use case:** Pagination
```javascript
merge: (currentCache, newItems) => {
  currentCache.items.push(...newItems.items)
  currentCache.total = newItems.total
}
```

**Works with:** `serializeQueryArgs` to share cache keys

---

#### `forceRefetch`
**Type:** `(params) => boolean`

**What it does:** Determine if query should refetch despite having cache.

```javascript
forceRefetch: ({ currentArg, previousArg, state, endpointState }) => {
  // Force refetch if page changed
  return currentArg !== previousArg
}
```

**Use case:** Pagination with merged cache

---

#### `keepUnusedDataFor`
**Type:** `number` (seconds)

**Default:** Inherits from `createApi` setting

**What it does:** Override global `keepUnusedDataFor` for this endpoint.

```javascript
// Expensive query - keep cache longer
getExpensiveData: builder.query({
  query: () => '/expensive',
  keepUnusedDataFor: 600, // 10 minutes
})

// Frequently changing - don't keep cache
getLiveData: builder.query({
  query: () => '/live',
  keepUnusedDataFor: 0, // Always refetch
})
```

---

### Hook Configuration Options

Options passed to `useQuery` and `useMutation` hooks:

#### `skip`
**Type:** `boolean`

**Default:** `false`

**What it does:** Conditionally skip the query.

```javascript
const { data } = useGetUserQuery(userId, {
  skip: !userId, // Don't run if no userId
})
```

**Behavior:**
- When `true`: No request made, previous cache used if exists
- When changes to `false`: Query runs immediately
- Useful for dependent queries

---

#### `pollingInterval`
**Type:** `number` (milliseconds)

**Default:** `0` (no polling)

**What it does:** Automatically refetch at regular intervals.

```javascript
const { data } = useGetStockPriceQuery('AAPL', {
  pollingInterval: 5000, // Refetch every 5 seconds
})
```

**Stops when:**
- Component unmounts
- `skip` becomes true
- Interval set to 0

---

#### `skipPollingIfUnfocused`
**Type:** `boolean`

**Default:** `false`

**What it does:** Pause polling when window/tab loses focus.

```javascript
const { data } = useGetDataQuery(undefined, {
  pollingInterval: 3000,
  skipPollingIfUnfocused: true, // Stop polling in background
})
```

**Battery/bandwidth saver** for mobile and background tabs

---

#### `refetchOnMountOrArgChange`
**Type:** `boolean | number`

**Default:** `false`

**What it does:** Override global refetch setting for this hook.

```javascript
// Always get fresh data
const { data } = useGetUserQuery(id, {
  refetchOnMountOrArgChange: true,
})

// Only refetch if cache is older than 30s
const { data } = useGetPostsQuery(undefined, {
  refetchOnMountOrArgChange: 30,
})
```

---

#### `refetchOnFocus`
**Type:** `boolean`

**Default:** Inherits from `createApi`

**What it does:** Override global refetch-on-focus for this hook.

```javascript
const { data } = useGetCriticalDataQuery(undefined, {
  refetchOnFocus: true, // Always fresh when returning to tab
})
```

---

#### `refetchOnReconnect`
**Type:** `boolean`

**Default:** Inherits from `createApi`

**What it does:** Override global refetch-on-reconnect for this hook.

```javascript
const { data } = useGetDataQuery(undefined, {
  refetchOnReconnect: true, // Refetch when back online
})
```

---

#### `selectFromResult`
**Type:** `(result) => any`

**What it does:** Select subset of data, memoized to prevent unnecessary re-renders.

```javascript
// Re-renders ONLY when this specific post changes
const { post } = useGetPostsQuery(undefined, {
  selectFromResult: ({ data }) => ({
    post: data?.find(p => p.id === postId)
  })
})

// Without selectFromResult: re-renders when ANY post changes
const { data } = useGetPostsQuery(undefined)
const post = data?.find(p => p.id === postId)
```

**Performance optimization:** Use when selecting small subset of large dataset

---

#### `subscriptionOptions`
**Type:** `{ pollingInterval, skipPollingIfUnfocused, refetchOnReconnect, refetchOnFocus }`

**What it does:** Group subscription-related options together (rarely used directly).

---

### Return Values from Hooks

Understanding what you get back:

#### Query Hook Returns
```javascript
const {
  // Data
  data,              // The actual response data
  currentData,       // Current data (might be outdated during refetch)
  error,             // Error object if request failed
  
  // Status booleans
  isLoading,         // First load, no data yet
  isFetching,        // Any fetch (including refetch)
  isSuccess,         // Has data, no error
  isError,           // Has error
  isUninitialized,   // Query hasn't run yet
  
  // Actions
  refetch,           // Manually trigger refetch
  
  // Metadata
  startedTimeStamp,  // When query started
  fulfilledTimeStamp, // When query completed
  requestId,         // Unique request ID
  endpointName,      // Name of endpoint
  originalArgs,      // Arguments passed to query
} = useGetPostsQuery()
```

**Status logic:**
- `isLoading`: First time fetching (`true` only on initial load)
- `isFetching`: Any time data is being fetched (includes refetch)
- `isSuccess`: Query completed successfully
- `isError`: Query failed

**Key difference:**
```javascript
// Initial load
isLoading: true, isFetching: true

// Refetching with cache
isLoading: false, isFetching: true, data: <cached>

// Success
isLoading: false, isFetching: false, data: <fresh>
```

---

#### Mutation Hook Returns
```javascript
const [
  trigger,           // Function to call mutation
  {
    data,            // Response data
    error,           // Error object
    isLoading,       // Mutation in progress
    isSuccess,       // Mutation succeeded
    isError,         // Mutation failed
    isUninitialized, // Mutation not called yet
    reset,           // Reset mutation state
  }
] = useCreatePostMutation()

// Call mutation
trigger({ title: 'Hello', body: 'World' })
```

---

### Advanced: `createApi` with `injectEndpoints`

**What it does:** Add endpoints after initial API creation (code splitting).

```javascript
// Base API
export const api = createApi({
  baseQuery: fetchBaseQuery({ baseUrl: '/api' }),
  endpoints: () => ({}), // Empty initially
})

// In feature module 1
export const userApi = api.injectEndpoints({
  endpoints: (builder) => ({
    getUser: builder.query({ query: (id) => `/users/${id}` }),
  }),
})

// In feature module 2  
export const postApi = api.injectEndpoints({
  endpoints: (builder) => ({
    getPost: builder.query({ query: (id) => `/posts/${id}` }),
  }),
  overrideExisting: false, // Warn if endpoint exists
})
```

**Benefits:**
- Lazy load endpoints
- Split API definition across files
- Better code organization

---

### Configuration Decision Tree

**How to choose settings:**

```
Need fresh data?
├─ YES, in real-time (< 1s) → WebSocket + onCacheEntryAdded
├─ YES, frequently (< 1min) → pollingInterval: 5000-30000
├─ YES, when user returns → refetchOnFocus: true
└─ NO, cache is fine → keepUnusedDataFor: 300+

Query is expensive?
├─ YES → keepUnusedDataFor: 600+, selectFromResult
└─ NO → Default settings fine

Data rarely changes?
├─ YES → refetchOnMountOrArgChange: false, long keepUnusedDataFor
└─ NO → Use polling or refetch strategies

Mobile users?
├─ YES → skipPollingIfUnfocused: true, refetchOnReconnect: true
└─ NO → Default settings fine
```

---

## 🎓 Learning Tips

1. **Build as you learn** - Don't just read, code alongside the tutorials
2. **Start simple** - Create small focused examples rather than one large app
3. **Use the official docs** - They're comprehensive and well-written
4. **Join the community** - Redux Toolkit GitHub discussions are helpful
5. **Debug with DevTools** - Install Redux DevTools to see what's happening
6. **Take notes** - Document your "aha" moments and confusion points
7. **Review regularly** - Revisit earlier phases as you learn advanced topics

---

## 🔑 Key Takeaways

By the end of this guide, you should understand:

1. **RTK Query is Redux** - It's not magic, just well-architected Redux patterns
2. **Cache-first thinking** - Always consider what's in the cache before fetching
3. **Tags are powerful** - The tag system enables sophisticated cache invalidation
4. **Subscriptions matter** - Multiple components can share the same query efficiently
5. **Customize when needed** - RTK Query is flexible for complex scenarios
6. **Performance by default** - But you can optimize further with selectors
7. **Real-world ready** - Patterns exist for pagination, auth, WebSockets, and more

---

## 📖 Additional Resources

### Official Documentation
- https://redux-toolkit.js.org/rtk-query/overview

### Community
- Reddit: r/reactjs, r/redux
- Discord: Reactiflux
- GitHub Discussions: reduxjs/redux-toolkit

### Advanced Topics
- RTK Query with TypeScript
- Code generation from OpenAPI specs
- GraphQL integration
- Offline-first applications

---

## 📝 Progress Tracker

Use this checklist to track your progress:

- [ ] Phase 1: Core Concepts completed
- [ ] Phase 2: Data Flow completed
- [ ] Phase 3: Cache Management completed
- [ ] Phase 4: Customization completed
- [ ] Phase 5: Performance completed
- [ ] Phase 6: Real-World Integration completed
- [ ] Final Project completed
- [ ] Feeling confident with RTK Query! 🎉

---

## 🤝 Contributing

Found an error or have a suggestion? This guide is meant to evolve. Consider:
- Testing all code examples
- Updating links if they break
- Adding new real-world patterns as they emerge
- Sharing your learning journey and insights

---

**Good luck on your RTK Query journey! Remember: the best way to learn is by building. Start coding today! 🚀**