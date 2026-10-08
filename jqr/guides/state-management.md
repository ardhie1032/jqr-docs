<!-- {
  "path": "jqr/guides/state-management",
  "name": "docs-jqr-guides-state-management",
  "meta": {
    "title": "State Management",
    "order": 13
  }
} -->

# State Management

State management is how you organize, share, and maintain application data across components. This guide covers patterns and best practices for managing state in JQR applications.

## State Organization Levels

JQR supports multiple levels of state organization:

1. **Component State** - Local to a single component
2. **Shared State** - Shared between multiple components
3. **Global State** - Application-wide state

## Component State

The simplest form - state local to a single component:

```html live:no-code
<div id="demo-component-state" class="en-card">
  <div class="en-card__body">
    <h3>Local Component State</h3>
    <div class="en-flex en-gap-2" style="margin-bottom: 1rem;">
      <input class="en-input" $:model="task" placeholder="Enter task...">
      <button class="en-btn en-btn--primary" $:click="addTask">Add</button>
    </div>
    <div class="en-alert en-alert--info" $:if="tasks.length === 0">
      No tasks yet. Add one above!
    </div>
    <ul class="en-list" $:if="tasks.length > 0">
      <li $:for="task in tasks" $:key="task" $:text="task"></li>
    </ul>
  </div>
</div>
```

```html
<div id="demo-component-state">
  <h3>Local Component State</h3>
  <input $:model="task" placeholder="Enter task...">
  <button $:click="addTask">Add</button>
  <div $:if="tasks.length === 0">No tasks yet</div>
  <ul $:if="tasks.length > 0">
    <li $:for="task in tasks" $:key="task" $:text="task"></li>
  </ul>
</div>
```

```js inject show
$('#demo-component-state').component({
  state: {
    task: '',
    tasks: []
  },
  methods: {
    addTask(state) {
      if (state.task.trim()) {
        state.tasks.push(state.task);
        state.task = '';
      }
    }
  }
});
```

```js cleanup
onUnmount(() => {
  $('#demo-component-state').component('destroy');
})
```

**When to use:**
- Data only used by one component
- Temporary UI state (form inputs, toggles)
- Component-specific calculations

## Shared State via Props

Share state by passing it down via props:

```html live:no-code
<template id="demo-counter-display">
  <div class="en-card">
    <div class="en-card__body">
      <h4 style="margin: 0;">Counter Display</h4>
      <p class="en-text--muted" style="margin: 0.5rem 0 0 0;">
        Current count: <code $:text="$props.value"></code>
      </p>
    </div>
  </div>
</template>

<div id="demo-shared-props">
  <div class="en-card">
    <div class="en-card__body">
      <h3>Shared State via Props</h3>
      <div class="en-flex en-gap-2" style="margin-bottom: 1rem;">
        <button class="en-btn en-btn--sm" $:click="count++">Increment</button>
        <button class="en-btn en-btn--sm" $:click="count--">Decrement</button>
        <button class="en-btn en-btn--sm en-btn--outline" $:click="count = 0">Reset</button>
      </div>
      <div $:component="demo-counter-display" $:prop:value="count"></div>
    </div>
  </div>
</div>
```

```html
<template id="demo-counter-display">
  <div>
    <h4>Counter Display</h4>
    <p>Current count: <code $:text="$props.value"></code></p>
  </div>
</template>

<div id="demo-shared-props">
  <h3>Shared State via Props</h3>
  <button $:click="count++">Increment</button>
  <button $:click="count--">Decrement</button>
  <button $:click="count = 0">Reset</button>
  <div $:component="counter-display" $:prop:value="count"></div>
</div>
```

```js inject show
$('#demo-shared-props').component({
  components: {
    'demo-counter-display': {
      props: { value: 0 }
    }
  },
  state: { count: 0 }
});
```

```js cleanup
onUnmount(() => {
  $('#demo-shared-props').component('destroy');
})
```

**When to use:**
- Parent-child data flow
- Simple hierarchies
- Configuration data

**Limitations:**
- Props only flow downward
- Deep nesting gets verbose ("prop drilling")

## Shared Reactive Object

Create a shared reactive object for sibling components using `JQR.reactive()`:

> **Note:** `JQR.reactive()` is available on the global `JQR` namespace (not `$.jqr`). It's a low-level reactivity primitive typically used for shared state patterns.
> - **Browser/IIFE:** `JQR.reactive({ ... })`
> - **ESM:** `import { reactive } from '@en32/jqr'`

```html live:no-code
<template id="demo-todo-input">
  <div class="en-card">
    <div class="en-card__body">
      <h4 style="margin: 0 0 0.5rem 0;">Add Todo</h4>
      <div class="en-flex en-gap-2">
        <input class="en-input" $:model="newTodo" placeholder="Enter todo...">
        <button class="en-btn en-btn--primary" $:click="add">Add</button>
      </div>
    </div>
  </div>
</template>

<template id="demo-todo-list">
  <div class="en-card">
    <div class="en-card__body">
      <h4 style="margin: 0 0 0.5rem 0;">Todo List</h4>
      <div class="en-alert en-alert--info" $:if="sharedStore.todos.length === 0">
        No todos yet
      </div>
      <ul class="en-list" $:if="sharedStore.todos.length > 0">
        <li $:for="todo in sharedStore.todos" $:key="todo" class="en-flex en-gap-2 en-items-center">
          <span $:text="todo"></span>
        </li>
      </ul>
    </div>
  </div>
</template>

<div id="demo-shared-state">
  <div class="en-card">
    <div class="en-card__body">
      <h3>Shared Reactive Object</h3>
      <div class="en-grid en-grid--cols-12 en-gap-3">
        <div class="en-col-12 en-col-md-6">
          <div $:component="demo-todo-input"></div>
        </div>
        <div class="en-col-12 en-col-md-6">
          <div $:component="demo-todo-list"></div>
        </div>
      </div>
    </div>
  </div>
</div>
```

```html
<template id="demo-todo-input">
  <div>
    <h4>Add Todo</h4>
    <input $:model="newTodo" placeholder="Enter todo...">
    <button $:click="add">Add</button>
  </div>
</template>

<template id="demo-todo-list">
  <div>
    <h4>Todo List</h4>
    <div $:if="sharedStore.todos.length === 0">No todos yet</div>
    <ul $:if="sharedStore.todos.length > 0">
      <li $:for="todo in sharedStore.todos" $:key="todo" $:text="todo"></li>
    </ul>
  </div>
</template>

<div id="demo-shared-state">
  <h3>Shared Reactive Object</h3>
  <div $:component="todo-input"></div>
  <div $:component="todo-list"></div>
</div>
```

```js inject show
// Create shared store
const todoStore = JQR.reactive({ todos: [] });

$('#demo-shared-state').component({
  components: {
    'demo-todo-input': {
      state: { newTodo: '', sharedStore: todoStore },
      methods: {
        add(state) {
          if (state.newTodo.trim()) {
            state.sharedStore.todos.push(state.newTodo);
            state.newTodo = '';
          }
        }
      }
    },
    'demo-todo-list': {
      state: { sharedStore: todoStore }
    }
  }
});
```

```js cleanup
onUnmount(() => {
  $('#demo-shared-state').component('destroy');
})
```

**How it works:**
1. Create shared reactive object: `JQR.reactive({ ... })`
2. Include in each component's state: `state: { sharedStore }`
3. All components sharing the object react to changes

**When to use:**
- Sibling components need same data
- Avoiding prop drilling
- Simple shared state

## Sharing Component State

**New Feature:** You can directly share one component's state with another component - no extra wrapping needed!

```html live:no-code
<div id="demo-shared-component-state">
  <div class="en-card">
    <div class="en-card__body">
      <h3>Sharing Component State</h3>
      <div class="en-grid en-grid--cols-12 en-gap-3">
        <div class="en-col-12 en-col-md-6">
          <div class="en-card">
            <div class="en-card__body">
              <h4 style="margin: 0 0 0.5rem 0;">Card Component</h4>
              <div class="en-flex en-gap-2">
                <input class="en-input" $:model="message" placeholder="Type something...">
              </div>
              <p class="en-text--muted en-text-sm" style="margin: 0.5rem 0 0 0;">
                Characters: <code $:text="message.length"></code>
              </p>
            </div>
          </div>
        </div>
        <div class="en-col-12 en-col-md-6">
          <div class="en-card">
            <div class="en-card__body">
              <h4 style="margin: 0 0 0.5rem 0;">Calendar Component</h4>
              <div class="en-alert en-alert--info" $:if="card.message.length === 0">
                No message yet
              </div>
              <div class="en-alert en-alert--success" $:if="card.message.length > 0">
                <strong>Message from card:</strong><br>
                <span $:text="card.message"></span>
              </div>
            </div>
          </div>
        </div>
      </div>
    </div>
  </div>
</div>
```

```html
<!-- Card component -->
<div id="card">
  <h4>Card Component</h4>
  <input $:model="message" placeholder="Type something...">
  <p>Characters: <code $:text="message.length"></code></p>
</div>

<!-- Calendar component references card's state -->
<div id="calendar">
  <h4>Calendar Component</h4>
  <div $:if="card.message.length === 0">No message yet</div>
  <div $:if="card.message.length > 0">
    <strong>Message from card:</strong>
    <span $:text="card.message"></span>
  </div>
</div>
```

```js inject show
// Create card component first
const card = $('#demo-shared-component-state .en-col-md-6:nth-child(1) .en-card').component({
  state: { message: '' }
});

// Create calendar component that shares card's state
$('#demo-shared-component-state .en-col-md-6:nth-child(2) .en-card').component({
  state: { 
    card: card.state  // ← Direct reference to card's state!
  }
});
```

```js cleanup
onUnmount(() => {
  $('#demo-shared-component-state .en-col-md-6:nth-child(1) .en-card').component('destroy');
  $('#demo-shared-component-state .en-col-md-6:nth-child(2) .en-card').component('destroy');
})
```

**How it works:**

```js
const card = $('#card').component({
  state: { users: [] }
});

const calendar = $('#calendar').component({
  state: { card: card.state }  // ← Share card's state
});
```

**Key points:**
- `calendar.state.card` is **the exact same reactive proxy** as `card.state`
- Not a copy - it's the same object reference
- Changes in either component update both automatically
- Two-way reactive binding works perfectly

**Why this works:**
1. JQR's `reactive()` never wraps an already-reactive object twice
2. When you assign `card.state` to `calendar.state.card`, it keeps the same proxy
3. Both components watch the same reactive object

**Important notes:**

**✅ Order matters:**
```js
// ✅ Good - card created first
const card = $('#card').component({ state: { count: 0 } });
const calendar = $('#calendar').component({ state: { card: card.state } });

// ❌ Bad - card doesn't exist yet
const calendar = $('#calendar').component({ state: { card: card.state } });
const card = $('#card').component({ state: { count: 0 } });
```

**✅ Destroying components:**
```js
card.destroy();  // Removes DOM bindings, but state object remains valid
calendar.state.card.count++;  // ✅ Still works! State persists
```

**⚠️ When NOT to use:**
- Parent-child components → Use `$:props` instead
- Need one-way data flow → Use `$:props` (read-only)
- Complex apps → Use Store Pattern for centralized logic

**When to use:**
- Independent sibling components
- Two-way shared state needed
- Simple coordination without parent container
- Alternative to global store for small cases

**Comparison with other patterns:**

```js
// Pattern 1: Shared reactive object (manual)
const shared = JQR.reactive({ count: 0 });
const card = $('#card').component({ state: { shared } });
const calendar = $('#calendar').component({ state: { shared } });

// Pattern 2: Share component state (automatic)
const card = $('#card').component({ state: { count: 0 } });
const calendar = $('#calendar').component({ state: { card: card.state } });

// Both work the same way! Pattern 2 is just more direct.
```

## Store Pattern

Create a centralized store with actions:

```js
// store.js
const store = JQR.reactive({
  // State
  count: 0,
  user: null,
  
  // Actions
  increment() {
    this.count++;
  },
  
  decrement() {
    this.count--;
  },
  
  setUser(user) {
    this.user = user;
  }
});

export default store;
```

**Usage in components:**

```js
import store from './store.js';

$('#app').component({
  state: { store },
  methods: {
    handleIncrement() {
      this.state.store.increment();
    }
  }
});
```

**Benefits:**
- Centralized state logic
- Clear mutations via actions
- Easy to test
- TypeScript friendly

## Module Pattern

Organize related state and logic into modules:

```js
// modules/cart.js
export const cartModule = {
  state: JQR.reactive({
    items: [],
    total: 0
  }),
  
  actions: {
    addItem(item) {
      this.state.items.push(item);
      this.updateTotal();
    },
    
    removeItem(index) {
      this.state.items.splice(index, 1);
      this.updateTotal();
    },
    
    updateTotal() {
      this.state.total = this.state.items.reduce(
        (sum, item) => sum + item.price, 
        0
      );
    },
    
    clear() {
      this.state.items = [];
      this.state.total = 0;
    }
  },
  
  getters: {
    itemCount() {
      return this.state.items.length;
    },
    
    isEmpty() {
      return this.state.items.length === 0;
    }
  }
};
```

**Usage:**

```js
import { cartModule } from './modules/cart.js';

$('#cart').component({
  state: {
    cart: cartModule.state
  },
  computed: {
    itemCount: () => cartModule.getters.itemCount(),
    isEmpty: () => cartModule.getters.isEmpty()
  },
  methods: {
    addToCart(state, item) {
      cartModule.actions.addItem(item);
    }
  }
});
```

## Event Bus Pattern

Use events for loose coupling between components:

```js
// eventBus.js
export const eventBus = {
  listeners: {},
  
  on(event, callback) {
    if (!this.listeners[event]) {
      this.listeners[event] = [];
    }
    this.listeners[event].push(callback);
  },
  
  off(event, callback) {
    if (!this.listeners[event]) return;
    this.listeners[event] = this.listeners[event].filter(
      cb => cb !== callback
    );
  },
  
  emit(event, data) {
    if (!this.listeners[event]) return;
    this.listeners[event].forEach(callback => callback(data));
  }
};
```

**Usage:**

```js
import { eventBus } from './eventBus.js';

// Component A - Emits event
$('#component-a').component({
  methods: {
    doSomething(state) {
      eventBus.emit('user:updated', { name: 'Alice' });
    }
  }
});

// Component B - Listens to event
$('#component-b').component({
  mounted() {
    this.handler = (data) => {
      console.log('User updated:', data);
    };
    eventBus.on('user:updated', this.handler);
  },
  destroyed() {
    eventBus.off('user:updated', this.handler);
  }
});
```

**When to use:**
- Components don't have parent-child relationship
- Decoupled communication
- Plugin systems

**⚠️ Cleanup warning:** Always remove event listeners in `destroyed`!

## Composition Pattern

Composable state modules for reuse:

```js
// composables/useCounter.js
export function useCounter(initial = 0) {
  const state = JQR.reactive({
    count: initial
  });
  
  return {
    state,
    increment: () => state.count++,
    decrement: () => state.count--,
    reset: () => state.count = initial
  };
}
```

**Usage:**

```js
import { useCounter } from './composables/useCounter.js';

$('#app').component({
  state: () => {
    const counter = useCounter(10);
    return {
      counter: counter.state,
      increment: counter.increment,
      decrement: counter.decrement
    };
  }
});
```

## Async State Management

Handle loading and error states with async data:

```html live:no-code
<div id="demo-async-state" class="en-card">
  <div class="en-card__body">
    <h3>Async State Management</h3>
    <div class="en-flex en-gap-2" style="margin-bottom: 1rem;">
      <button class="en-btn en-btn--sm en-btn--primary" $:click="fetchUsers">Fetch Users</button>
      <button class="en-btn en-btn--sm en-btn--outline" $:click="reset">Reset</button>
    </div>
    
    <div $:if="loading" class="en-spinner"></div>
    
    <div $:if="error" class="en-alert en-alert--danger">
      <strong>Error:</strong> <span $:text="error"></span>
    </div>
    
    <div $:if="!loading && !error && users && users.length > 0">
      <div class="en-alert en-alert--success" style="margin-bottom: 0.5rem;" $:for="user in users.slice(0, 3)" $:key="user.id">
        <strong $:text="user.name"></strong>
      </div>
    </div>
  </div>
</div>
```

```html
<div id="demo-async-state">
  <h3>Async State Management</h3>
  <button $:click="fetchUsers">Fetch Users</button>
  <button $:click="reset">Reset</button>
  
  <div $:if="loading">Loading...</div>
  <div $:if="error">Error: <span $:text="error"></span></div>
  
  <div $:if="!loading && !error && users && users.length > 0">
    <div $:for="user in users.slice(0, 3)" $:key="user.id">
      <strong $:text="user.name"></strong>
    </div>
  </div>
</div>
```

```js inject show
$('#demo-async-state').component({
  state: {
    users: null,
    loading: false,
    error: null
  },
  methods: {
    async fetchUsers(state) {
      state.loading = true;
      state.error = null;
      try {
        const res = await fetch('/assets/api/jqr-docs/users.json');
        state.users = await res.json();
      } catch (err) {
        state.error = err.message;
      } finally {
        state.loading = false;
      }
    },
    reset(state) {
      state.users = null;
      state.loading = false;
      state.error = null;
    }
  }
});
```

```js cleanup
onUnmount(() => {
  $('#demo-async-state').component('destroy');
})
```

**Pattern:**
- `loading`: Boolean flag for loading state
- `error`: String/object for error state
- `data`: Actual data (null when not loaded)

## Local Storage Persistence

Persist state to localStorage:

```js
// utils/storage.js
export const storage = {
  save(key, value) {
    localStorage.setItem(key, JSON.stringify(value));
  },
  
  load(key, defaultValue = null) {
    const item = localStorage.getItem(key);
    return item ? JSON.parse(item) : defaultValue;
  },
  
  remove(key) {
    localStorage.removeItem(key);
  }
};

// Create reactive store with persistence
export function createPersistedStore(key, initialState) {
  const state = JQR.reactive(
    storage.load(key, initialState)
  );
  
  // Auto-save on any change
  JQR.watch(
    () => state,
    (newState) => storage.save(key, newState),
    { deep: true }
  );
  
  return state;
}
```

**Usage:**

```js
import { createPersistedStore } from './utils/storage.js';

const userPrefs = createPersistedStore('user-prefs', {
  theme: 'light',
  language: 'en'
});

$('#app').component({
  state: { prefs: userPrefs }
});
```

## State Debugging

Debug state changes with watchers:

```js
$('#app').component({
  state: { count: 0 },
  
  mounted() {
    // Log all state changes
    this.watch(
      () => this.state,
      (newState, oldState) => {
        console.log('State changed:', { old: oldState, new: newState });
      },
      { deep: true }
    );
    
    // Log specific property
    this.watch(
      () => this.state.count,
      (newVal, oldVal) => {
        console.log(`Count: ${oldVal} → ${newVal}`);
      }
    );
  }
});
```

## State Patterns Comparison

| Pattern | Best For | Complexity | Coupling |
|---------|----------|------------|----------|
| Component State | Local UI | Low | None |
| Props | Parent-Child | Low | Tight |
| Shared Reactive | Siblings | Low | Medium |
| **Shared Component State** | **Independent Siblings** | **Low** | **Medium** |
| Store | App State | Medium | Loose |
| Module | Domain Logic | Medium | Loose |
| Event Bus | Decoupled | Medium | Very Loose |
| Composables | Reusable Logic | Low | None |

## Best Practices

### 1. Start Simple

```js
// ✅ Good - Start with component state
state: { count: 0 }

// ❌ Avoid - Don't over-engineer early
const complexStore = createStoreWithMiddleware({ ... });
```

### 2. Keep State Flat

```js
// ✅ Good - Flat structure
state: {
  userId: 1,
  userName: 'Alice',
  userEmail: 'alice@example.com'
}

// ⚠️ Acceptable - One level
state: {
  user: { id: 1, name: 'Alice', email: 'alice@example.com' }
}

// ❌ Avoid - Deep nesting
state: {
  app: {
    user: {
      profile: {
        personal: { name: 'Alice' }
      }
    }
  }
}
```

### 3. Normalize Complex Data

```js
// ❌ Bad - Nested duplicates
state: {
  posts: [
    { id: 1, author: { id: 1, name: 'Alice' } },
    { id: 2, author: { id: 1, name: 'Alice' } }  // Duplicate!
  ]
}

// ✅ Good - Normalized
state: {
  posts: [
    { id: 1, authorId: 1 },
    { id: 2, authorId: 1 }
  ],
  users: {
    1: { id: 1, name: 'Alice' }
  }
}
```

### 4. Separate UI State from Domain State

```js
// ✅ Good - Clear separation
state: {
  // Domain state
  users: [],
  posts: [],
  
  // UI state
  selectedUserId: null,
  isModalOpen: false,
  searchQuery: ''
}
```

### 5. Use Computed for Derived State

```js
// ❌ Bad - Storing derived state
state: {
  items: [1, 2, 3],
  itemCount: 3  // Duplicate!
}

// ✅ Good - Compute on demand
state: {
  items: [1, 2, 3]
},
computed: {
  itemCount: (state) => state.items.length
}
```

### 6. Handle Async Errors

```js
// ✅ Good - Proper error handling
async fetchData(state) {
  state.loading = true;
  state.error = null;
  try {
    const res = await fetch('/api/data');
    if (!res.ok) throw new Error('Request failed');
    state.data = await res.json();
  } catch (err) {
    state.error = err.message;
    console.error('Fetch error:', err);
  } finally {
    state.loading = false;
  }
}
```

### 7. Clean Up Subscriptions

```js
// ✅ Good - Cleanup in destroyed
mounted() {
  this.unwatch = this.watch(() => this.state.count, handleChange);
},
destroyed() {
  if (this.unwatch) this.unwatch();
}
```

### 8. Share Component State Directly

```js
// ✅ Good - Direct state sharing for siblings
const card = $('#card').component({ state: { count: 0 } });
const calendar = $('#calendar').component({ state: { card: card.state } });

// ✅ Also good - Manual shared reactive
const shared = JQR.reactive({ count: 0 });
const card = $('#card').component({ state: { shared } });
const calendar = $('#calendar').component({ state: { shared } });

// ❌ Avoid for parent-child - use props instead
const parent = $('#parent').component({ state: { data: {} } });
// Child should receive via $:prop, not state reference
```

## Common Pitfalls

### Mutating Props

```js
// ❌ Bad - Mutating props
methods: {
  updateProp({ props }) {
    props.value = 123;  // Read-only!
  }
}

// ✅ Good - Emit event
methods: {
  updateProp() {
    this.emit('update:value', 123);
  }
}
```

### Lost Reactivity

```js
// ❌ Bad - Lost reactivity
state: {
  items: [1, 2, 3]
},
methods: {
  replaceItems(state) {
    state.items = [4, 5, 6];  // ❌ Breaks watchers on old array
  }
}

// ✅ Good - Modify in place
methods: {
  replaceItems(state) {
    state.items.splice(0, state.items.length, 4, 5, 6);
  }
}
```

### Race Conditions

```js
// ❌ Bad - No request tracking
async fetchUser(state, id) {
  const res = await fetch(`/api/users/${id}`);
  state.user = await res.json();  // Old request may arrive late!
}

// ✅ Good - Track request ID
state: { requestId: 0 },
async fetchUser(state, id) {
  const requestId = ++state.requestId;
  const res = await fetch(`/api/users/${id}`);
  const data = await res.json();
  if (requestId === state.requestId) {  // Only update if latest
    state.user = data;
  }
}
```

## Performance Tips

### 1. Avoid Deep Watchers When Possible

```js
// ⚠️ Expensive
this.watch(() => this.state, handler, { deep: true });

// ✅ Better - Watch specific properties
this.watch(() => this.state.items, handler);
```

### 2. Batch State Updates

```js
// ❌ Multiple updates trigger multiple renders
state.firstName = 'John';
state.lastName = 'Doe';
state.email = 'john@example.com';

// ✅ Use nextTick or update together
Object.assign(state, {
  firstName: 'John',
  lastName: 'Doe',
  email: 'john@example.com'
});
```

### 3. Lazy Load Large State

```js
// ✅ Load on demand
asyncState: {
  async largeData() {
    // Only loads when accessed
    return fetch('/api/large-dataset').then(r => r.json());
  }
}
```

## Next Steps

- [Props](./props.md) - Component props system
- [Async State](./async-state.md) - Async data fetching
- [Instance Methods](./instance-methods.md) - Watch, refetch, and more
- [Reactivity](./reactivity.md) - How reactivity works
