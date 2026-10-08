<!-- {
  "path": "jqr/guides/performance",
  "name": "docs-jqr-guides-performance",
  "meta": {
    "title": "Performance Tips",
    "order": 14
  }
} -->

# Performance Tips

This guide covers performance optimization techniques based on JQR's actual implementation. These tips are specific to how JQR's reactivity, directives, and rendering work internally.

## Reactivity System

### Use `computed` for Expensive Calculations

**Problem:** Every directive (`$:text`, `$:show`, etc.) wraps its expression in an `effect()` that re-runs whenever dependencies change. Expensive calculations in directives run every time, without caching.

**Solution:** Move heavy calculations to `computed` — results are cached and only recomputed when dependencies change.

```html live:no-code
<div id="demo-computed-perf" class="en-card">
  <div class="en-card__body">
    <h3>Computed Performance</h3>
    <div class="en-form-group">
      <label class="en-form-label">Search:</label>
      <input class="en-input" $:model="search" placeholder="Type to search...">
    </div>
    <p class="en-text--muted">
      Found: <strong $:text="filteredCount"></strong> items
    </p>
    <template $:for="item in filteredItems">
      <div $:text="item"></div>
    </template>
    <div class="en-text-sm en-text--muted">
      Using computed - calculation runs only when search changes
    </div>
  </div>
</div>
```

```html
<div id="demo-computed-perf">
  <h3>Computed Performance</h3>
  <input $:model="search" placeholder="Search...">
  <p>Found: <strong $:text="filteredCount"></strong> items</p>
</div>
```

```js inject show
$('#demo-computed-perf').component({
  state: {
    search: '',
    items: ['Apple', 'Banana', 'Cherry', 'Date', 'Elderberry']
  },
  computed: {
    // ✅ Good: Heavy calculation cached in computed
    filteredItems(state) {
      return state.search.length > 0 ? state.items.filter(item => 
        item.toLowerCase().includes(state.search.toLowerCase())
      ) : [] ;
    },
    filteredCount(state) {
      return state.filteredItems.length;
    }
  }
});
```

```js cleanup
onUnmount(() => {
  $('#demo-computed-perf').component('destroy');
})
```

**❌ Bad - Expensive calculation in every directive:**
```html
<div $:text="items.filter(i => i.includes(search)).length"></div>
<div $:show="items.filter(i => i.includes(search)).length > 0"></div>
<!-- filter() runs twice per render -->
```

**✅ Good - Calculate once, use everywhere:**
```js
computed: {
  filteredItems(state) {
    return state.items.filter(i => i.includes(state.search));
  }
}
```

```html
<div $:text="filteredItems.length"></div>
<div $:show="filteredItems.length > 0"></div>
<!-- Computed value cached, reused everywhere -->
```

### Computed is Eager, Not Lazy

**Important:** `computed()` recalculates immediately when any dependency changes, even if `.value` isn't being read at that moment.

**Avoid:** Creating expensive computed chains that rarely get used.

**Consider:** For rarely-used calculations, use `methods` instead and call on-demand:

```js
// ❌ Bad: Computed always runs when items change
computed: {
  expensiveStats(state) {
    return state.items.reduce((acc, item) => {
      // Complex, rarely-needed calculation
    }, {});
  }
}

// ✅ Better: Calculate only when needed
methods: {
  getStats(state) {
    return state.items.reduce((acc, item) => {
      // Only runs when explicitly called
    }, {});
  }
}
```

### Non-Reactive Data Types

**Built-in objects are NOT reactive internally:**
- `Date`, `Map`, `Set`, `RegExp`, `WeakMap`, etc.
- In-place mutations **don't** trigger re-renders

```js
// ❌ Won't trigger reactivity
state.myMap.set('key', 'value');  // No update!
state.myDate.setTime(Date.now()); // No update!

// ✅ Replace the entire object
state.myMap = new Map(state.myMap).set('key', 'value');
state.myDate = new Date(Date.now());

// ✅ Or use plain objects/arrays
state.data = { key: 'value' };  // Reactive
state.timestamps = [Date.now()]; // Reactive
```

### Keep Non-Reactive Data Out of State

**Don't** put large static data that doesn't need reactivity into `state`:

```js
// ❌ Bad: Reactive proxy wraps everything recursively
state: {
  countries: [...], // 200+ country objects
  translations: {...}, // 1000+ translation keys
  config: {...}     // Static configuration
}

// ✅ Good: Use non-reactive constants
constants: {
  countries: Object.freeze([...]),
  translations: Object.freeze({...}),
  config: Object.freeze({...})
}

// ✅ Or module-level constants
const COUNTRIES = [...];
const TRANSLATIONS = {...};
```

### Batch State Changes

Updates are batched via microtask — multiple synchronous state changes trigger only one re-render per effect.

```js
// ✅ Good: All changes batched into one render
methods: {
  updateUser(state) {
    state.user.name = 'Alice';
    state.user.email = 'alice@example.com';
    state.user.role = 'admin';
    // Only renders once
  }
}

// ❌ Bad: async breaks batching
async updateUser(state) {
  state.user.name = 'Alice';
  await someAsyncOp();          // Renders here
  state.user.email = 'alice@example.com'; // Renders again
}
```

### Avoid Read-Mutate Cycles

**Never mutate state inside expressions that render it:**

```js
// ❌ DANGER: Infinite loop or forced stop after 100 runs
$:text="items.sort().join(', ')"  // sort() mutates!
$:text="items.reverse()[0]"       // reverse() mutates!

// ✅ Good: Mutate in methods, not in templates
methods: {
  sortedItems(state) {
    return [...state.items].sort(); // New array
  }
}
```

```html
<div $:text="sortedItems().join(', ')"></div>
```

## List Rendering (`$:for`)

### Always Use `$:key` for Objects

**Without `$:key`:** Items keyed by **index position** — insert/delete/reorder causes wrong nodes to be reused.

```html live:no-code
<div id="demo-key-perf" class="en-card">
  <div class="en-card__body">
    <h3>$:key Performance</h3>
    <button class="en-btn en-btn--sm en-btn--primary" $:click="shuffle">
      Shuffle List
    </button>
    <p class="en-text--muted en-text-sm" style="margin-top: 0.5rem;">
      With $:key, DOM nodes are properly tracked during reorder
    </p>
    <ul class="en-list" style="margin-top: 1rem;">
      <li $:for="user in users" $:key="user.id">
        <strong $:text="user.name"></strong>
        <span class="en-text--muted"> (ID: <span $:text="user.id"></span>)</span>
      </li>
    </ul>
  </div>
</div>
```

```html
<div id="demo-key-perf">
  <h3>$:key Performance</h3>
  <button $:click="shuffle">Shuffle</button>
  <ul>
    <li $:for="user in users" $:key="user.id">
      <strong $:text="user.name"></strong> (ID: <span $:text="user.id"></span>)
    </li>
  </ul>
</div>
```

```js inject show
$('#demo-key-perf').component({
  state: {
    users: [
      { id: 1, name: 'Alice' },
      { id: 2, name: 'Bob' },
      { id: 3, name: 'Charlie' }
    ]
  },
  methods: {
    shuffle(state) {
      state.users = [...state.users].sort(() => Math.random() - 0.5);
    }
  }
});
```

```js cleanup
onUnmount(() => {
  $('#demo-key-perf').component('destroy');
})
```

**Enable dev mode to catch missing keys:**
```js
$.jqr.config({ devMode: true });
// Warns once per template when $:key is missing
```

### Optimize Large Lists

For large lists where only a small portion changes frequently:

```js
computed: {
  // ✅ Cache filtered/sorted results
  visibleItems(state) {
    return state.items
      .filter(i => i.visible)
      .sort((a, b) => a.order - b.order);
  }
}
```

```html
<!-- $:for doesn't get new array every render if nothing changed -->
<div $:for="item in visibleItems" $:key="item.id">...</div>
```

## Expression Performance

### Expressions Use `with(scope)`

Expression strings are compiled and cached globally, but execution uses `with(scope)` which **cannot be JIT-optimized** by JavaScript engines.

**For complex logic:** Use `methods` or `computed` (regular functions, JIT-optimizable) instead of long expressions:

```html
<!-- ❌ Bad: Complex logic in template -->
<div $:text="users.filter(u => u.role === 'admin').map(u => u.name.toUpperCase()).join(', ')"></div>

<!-- ✅ Good: Move to computed -->
<div $:text="adminNames"></div>
```

```js
computed: {
  adminNames(state) {
    return state.users
      .filter(u => u.role === 'admin')
      .map(u => u.name.toUpperCase())
      .join(', ');
  }
}
```

### Avoid Expensive Function Calls in Templates

```html
<!-- ❌ Bad: Called for every item -->
<div $:for="item in items" $:key="item.id">
  <code $:text="JSON.stringify(item)"></code>
</div>

<!-- ✅ Good: Precompute before rendering -->
```

```js
computed: {
  itemsWithJson(state) {
    return state.items.map(item => ({
      ...item,
      json: JSON.stringify(item)
    }));
  }
}
```

```html
<div $:for="item in itemsWithJson" $:key="item.id">
  <code $:text="item.json"></code>
</div>
```

## Watch Performance

### Deep Watch Uses `JSON.stringify`

`{ deep: true }` creates snapshots via `JSON.stringify()` for comparison — expensive for large objects.

```js
// ❌ Expensive for large arrays
this.watch(
  () => state,
  callback,
  { deep: true }
);

// ✅ Watch specific properties
this.watch(
  () => state.items.length,
  callback
);

// ✅ Watch nested path
this.watch(
  () => state.user.name,
  callback
);
```

### Keep Watch Getters Narrow

Only properties **read inside the getter** are tracked:

```js
// ❌ Bad: Tracks entire state object
this.watch(
  () => state,
  (newVal) => {
    console.log(newVal.user.name); // Only need user.name
  }
);

// ✅ Good: Only tracks user.name
this.watch(
  () => state.user.name,
  (newName) => {
    console.log(newName);
  }
);
```

## Model Input Optimization

### Use Built-in Modifiers

For inputs that trigger expensive operations:

```html live:no-code
<div id="demo-model-modifiers" class="en-card">
  <div class="en-card__body">
    <h3>Model Modifiers</h3>
    <div class="en-form-group">
      <label class="en-form-label">Debounced (300ms):</label>
      <input 
        class="en-input" 
        $:model.debounce.300="search"
        placeholder="Type to search...">
    </div>
    <p class="en-text--muted en-text-sm">
      Updates only after you stop typing for 300ms
    </p>
    <div class="en-alert en-alert--info" style="margin-top: 1rem;">
      Search value: <strong $:text="search || '(empty)'"></strong>
    </div>
  </div>
</div>
```

```html
<div id="demo-model-modifiers">
  <h3>Model Modifiers</h3>
  <label>Debounced (300ms):</label>
  <input $:model.debounce.300="search" placeholder="Search...">
  <p>Search: <strong $:text="search || '(empty)'"></strong></p>
</div>
```

```js inject show
$('#demo-model-modifiers').component({
  state: { search: '' }
});
```

```js cleanup
onUnmount(() => {
  $('#demo-model-modifiers').component('destroy');
})
```

**Available modifiers:**
- `$:model.debounce="query"` - Default 250ms delay
- `$:model.debounce.500="query"` - Custom delay
- `$:model.throttle="query"` - Throttle updates
- `$:model.lazy="field"` - Update on `change`, not `input`

**Better than manual:**
```js
// ❌ Manual debouncing is more expensive
this.watch(
  () => state.query,
  $.jqr.debounce((val) => {
    // Search logic
  }, 300)
);
```

## Async State Optimization

### Use Loading States for Better UX

`asyncState` automatically creates `{key}Loading` and `{key}Error`:

```html
<!-- ✅ Show loading, don't block entire UI -->
<div $:if="usersLoading">Loading users...</div>
<div $:if="usersError">Error loading users</div>
<div $:if="users && !usersLoading">
  <div $:for="user in users" $:key="user.id">...</div>
</div>
```

### Avoid Reactive Dependencies in Fetchers

Async functions automatically refetch when dependencies change:

```js
// ❌ Bad: Refetches on ANY state change
asyncState: {
  async data(state, props) {
    // Reading many reactive properties
    console.log(state.filters, state.sort, state.page);
    return fetch(`/api/data`).then(r => r.json());
  }
}

// ✅ Good: Only refetch when props change
asyncState: {
  async data(state, props) {
    // Only reads props.id - refetches when id changes
    return fetch(`/api/data/${props.id}`).then(r => r.json());
  }
}

// ✅ Manual refetch via button
methods: {
  refresh() {
    this.refetch('data');
  }
}
```

## Component Optimization

### Use `mountEach` for Multiple Instances

```js
// ❌ Bad: Reuses first instance
$('.calculator').component({...});

// ✅ Good: Each element gets independent instance
$.jqr.mountEach('.calculator', (el, index) => ({
  state: { count: index }
}));
```

### Cleanup in `destroyed()` Hook

Nested components auto-cleanup when removed, but manual resources need cleanup:

```js
{
  mounted(state, $el) {
    this.interval = setInterval(() => {
      state.time = Date.now();
    }, 1000);
  },
  destroyed() {
    // ✅ Clean up to prevent memory leaks
    clearInterval(this.interval);
  }
}
```

### Avoid Frequent Component Creation

```html
<!-- ❌ Bad: Component created/destroyed on every toggle -->
<div $:if="showWidget">
  <div $:component="heavy-widget"></div>
</div>

<!-- ✅ Better: Keep mounted, hide with CSS -->
<div $:show="showWidget">
  <div $:component="heavy-widget"></div>
</div>
```

## Development vs Production

### Disable Dev Mode in Production

```js
// Development
$.jqr.config({ devMode: true });
// Enables warnings for:
// - Unknown directives
// - Missing $:key in $:for
// - Conflicting directives
// - Invalid operations

// Production (default)
$.jqr.config({ devMode: false });
// Removes all dev warnings for better performance
```

### Configure Sanitizer Wisely

```js
// ❌ Global sanitizer runs on every $:html
$.jqr.config({
  sanitizer: (html) => DOMPurify.sanitize(html)
});

// ✅ Only use if you actually need $:html with untrusted content
// Otherwise, leave it undefined (default) for better performance
```

## Performance Checklist

**Reactivity:**
- ✅ Move expensive calculations to `computed`
- ✅ Keep static data out of `state` (use `constants` or module-level)
- ✅ Replace entire objects for `Map`/`Set`/`Date` instead of mutating
- ✅ Batch synchronous state changes
- ✅ Never mutate arrays in template expressions (`.sort()`, `.reverse()`)

**Lists:**
- ✅ Always use `$:key` for object arrays
- ✅ Cache filtered/sorted results in `computed`
- ✅ Enable `devMode` during development to catch missing keys

**Expressions:**
- ✅ Move complex logic to `methods` or `computed`
- ✅ Avoid expensive function calls in `$:for` items
- ✅ Precompute data transformations

**Watch:**
- ✅ Avoid `{ deep: true }` on large objects
- ✅ Watch specific properties, not entire state
- ✅ Keep watch getters narrow

**Inputs:**
- ✅ Use `$:model.debounce`/`.throttle`/`.lazy` modifiers
- ✅ Don't manually debounce in `watch()`

**Components:**
- ✅ Use `$.jqr.mountEach()` for multiple instances
- ✅ Clean up intervals/listeners in `destroyed()`
- ✅ Use `$:show` instead of `$:if` for frequently toggled components

**Production:**
- ✅ Disable dev mode: `$.jqr.config({ devMode: false })`
- ✅ Only configure sanitizer if needed

## Measuring Performance

Use browser DevTools to identify bottlenecks:

```js
// Profile a heavy operation
console.time('filterData');
const result = state.items.filter(heavyFilter);
console.timeEnd('filterData');

// Check computed cache hits
computed: {
  expensiveCalc(state) {
    console.log('expensiveCalc recomputed'); // Should be rare
    return heavyCalculation(state.data);
  }
}
```

## Next Steps

- [Reactivity Guide](./reactivity.md) - Understand the reactivity system
- [Async State Guide](./async-state.md) - Optimize data fetching
- [State Management](./state-management.md) - Organize state efficiently
- [Configuration Reference](../reference/config-reference.md) - All config options
