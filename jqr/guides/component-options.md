<!-- {
  "path": "jqr/guides/component-options",
  "name": "docs-jqr-guides-component-options",
  "meta": {
    "title": "Component Options",
    "order": 3
  }
} -->

# Component Options

This guide provides a complete reference for all options available when creating a JQR component. Each option controls different aspects of component behavior, from reactive state to lifecycle hooks.

## Component Structure

A JQR component is initialized with an options object:

```js
$('#app').component({
  // Options go here
});
```

All options are optional - even an empty object creates a valid (though minimal) component.

## Complete Options Overview

```js
$('#app').component({
  // === Data ===
  state: { /* reactive state */ },
  props: { /* component props */ },
  computed: { /* derived values */ },
  asyncState: { /* async data */ },
  
  // === Logic ===
  methods: { /* functions */ },
  watch: { /* state watchers */ },
  
  // === Lifecycle ===
  created() { /* ... */ },
  mounted() { /* ... */ },
  updated() { /* ... */ },
  destroyed() { /* ... */ },
  
  // === Components ===
  components: { /* nested components */ },
  template: '<div>...</div>',
  
  // === Render (JQR-H only) ===
  render: (h, ctx) => { /* ... */ }
});
```

---

## Data Options

### `state`

**Type:** `Object`  
**Reactive:** Yes

Defines the component's reactive state. Properties are made reactive using ES6 Proxies.

```js
state: {
  count: 0,
  message: 'Hello',
  user: null,
  items: []
}
```

**Access:**
- In template: `<p $:text="count"></p>`
- In methods: `this.count`
- Anywhere: `this.$state.count`

**Example:**

```html live:no-code
<div id="demo-state" class="en-card en-card__body">
  <p>Count: <code $:text="count"></code></p>
  <button class="en-btn en-btn--sm" $:click="count++">Increment</button>
</div>
```

```html
<div id="demo-state">
  <p>Count: <code $:text="count"></code></p>
  <button $:click="count++">Increment</button>
</div>
```

```js inject show
$('#demo-state').component({
  state: { count: 0 }
});
```

```js cleanup
onUnmount(() => {
  $('#demo-state').component('destroy');
})
```

**See:** [Reactivity Guide](./reactivity.md)

---

### `props`

**Type:** `Object`  
**Reactive:** Yes (read-only)

Defines properties passed from parent components. Props are reactive but should not be mutated directly.

```js
props: {
  userId: null,
  title: 'Default Title',
  count: 0
}
```

**Access:**
- In template: `<p $:text="$props.userId"></p>`
- In methods: Use `$props` object

**Pass props from parent:**
```html
<div $:component="user-card" $:prop:userId="currentUser"></div>
```

**Example:**

```html live:no-code
<template id="demo-props-child">
  <div class="en-card en-card__body">
    <p>Received prop: <code $:text="$props.message"></code></p>
  </div>
</template>

<div id="demo-props-parent">
  <input class="en-input en-mb-2" $:model="parentMessage" placeholder="Type something...">
  <div $:component="demo-props-child" $:prop:message="parentMessage"></div>
</div>
```

```html
<template id="demo-props-child">
  <div>
    <p>Received prop: <code $:text="$props.message"></code></p>
  </div>
</template>

<div id="demo-props-parent">
  <input $:model="parentMessage" placeholder="Type something...">
  <div $:component="demo-props-child" $:prop:message="parentMessage"></div>
</div>
```

```js inject show
$('#demo-props-parent').component({
  components: {
    'demo-props-child': {
      props: { message: '' }
    }
  },
  state: { parentMessage: 'Hello from parent!' }
});
```

```js cleanup
onUnmount(() => {
  $('#demo-props-parent').component('destroy');
})
```

**See:** [Props Guide](./props.md)

---

### `computed`

**Type:** `Object<Function>`  
**Reactive:** Yes (cached)

Defines derived values computed from state. Computed values are cached and only recalculate when dependencies change.

```js
computed: {
  fullName: (c) => `${c.firstName} ${c.lastName}`,
  itemCount: (c) => c.items.length,
  isValid: (c) => c.username.length >= 3
}
```

**Access:**
- In template: `<p $:text="fullName"></p>`
- In methods: `computed.fullName`

**Example:**

```html live:no-code
<div id="demo-computed" class="en-card en-card__body">
  <input class="en-input en-mb-2" $:model="firstName" placeholder="First name">
  <input class="en-input en-mb-2" $:model="lastName" placeholder="Last name">
  <p>Full name: <strong $:text="fullName"></strong></p>
</div>
```

```html
<div id="demo-computed">
  <input $:model="firstName" placeholder="First name">
  <input $:model="lastName" placeholder="Last name">
  <p>Full name: <strong $:text="fullName"></strong></p>
</div>
```

```js inject show
$('#demo-computed').component({
  state: {
    firstName: 'John',
    lastName: 'Doe'
  },
  computed: {
    fullName: (c) => `${c.firstName} ${c.lastName}`
  }
});
```

```js cleanup
onUnmount(() => {
  $('#demo-computed').component('destroy');
})
```

**See:** [Reactivity Guide](./reactivity.md)

---

### `asyncState`

**Type:** `Object<AsyncFunction>`  
**Reactive:** Yes

Defines async data that loads automatically when component mounts. Each property is an async function that returns data.

```js
asyncState: {
  async users() {
    const response = await fetch('/api/users');
    return response.json();
  },
  async profile() {
    const response = await fetch(`/api/users/${this.$props.userId}`);
    return response.json();
  }
}
```

**Access:**
- Data: `ctx.$state.users` (undefined until loaded)
- Loading: `ctx.$state.usersLoading` (boolean)
- Error: `ctx.$state.usersError` (undefined or Error)

**Methods:**
- Refetch: `this.$refetch('users')`
- Refetch all: `this.$refetch()`

**Example:**

```html live:no-code
<div id="demo-async" class="en-card en-card__body">
  <div $:if="usersLoading">Loading users...</div>
  <div $:if="usersError" class="en-alert en-alert--danger">
    Error: <span $:text="usersError.message"></span>
  </div>
  <div $:if="!usersLoading && !usersError && users">
    <ul class="en-list-group">
      <li class="en-list-group__item" $:for="user in users.slice(0, 3)" $:key="user.id" $:text="user.name"></li>
    </ul>
  </div>
</div>
```

```html
<div id="demo-async">
  <div $:if="usersLoading">Loading users...</div>
  <div $:if="usersError">Error: <span $:text="usersError.message"></span></div>
  <div $:if="!usersLoading && !usersError && users">
    <ul>
      <li $:for="user in users.slice(0, 3)" $:key="user.id" $:text="user.name"></li>
    </ul>
  </div>
</div>
```

```js inject show
$('#demo-async').component({
  state: { usersLoading: false, usersError: null },
  asyncState: {
    async users(c) {
      c.usersLoading = true
      const response = await fetch('/assets/api/jqr-docs/users.json');
      c.usersLoading = false
      return response.json();
    }
  }
});
```

```js cleanup
onUnmount(() => {
  $('#demo-async').component('destroy');
})
```

**See:** [Async State Guide](./async-state.md)

---

## Logic Options

### `methods`

**Type:** `Object<Function>`  
**Context:** Receives `(state, ...args)`

Defines functions that can modify state. Methods receive state as first parameter.

```js
methods: {
  increment(c) {
    c.count++;
  },
  addItem(c, item) {
    c.items.push(item);
  },
  async saveUser(c) {
    await fetch('/api/users', {
      method: 'POST',
      body: JSON.stringify(c.user)
    });
  }
}
```

**Call from template:**
```html
<button $:click="increment()">+</button>
<button $:click="addItem('New')">Add</button>
```

**Call from methods:**
```js
methods: {
  save(state) {
    this.validate(state);  // Call another method
  }
}
```

**Example:**

```html live:no-code
<div id="demo-methods" class="en-card en-card__body">
  <p>Counter: <code $:text="count"></code></p>
  <div class="en-flex en-gap-2">
    <button class="en-btn en-btn--sm" $:click="increment()">+1</button>
    <button class="en-btn en-btn--sm" $:click="decrement()">-1</button>
    <button class="en-btn en-btn--sm" $:click="reset()">Reset</button>
  </div>
</div>
```

```html
<div id="demo-methods">
  <p>Counter: <code $:text="count"></code></p>
  <button $:click="increment()">+1</button>
  <button $:click="decrement()">-1</button>
  <button $:click="reset()">Reset</button>
</div>
```

```js inject show
$('#demo-methods').component({
  state: { count: 0 },
  methods: {
    increment(c) {
      c.count++;
    },
    decrement(c) {
      c.count--;
    },
    reset(c) {
      c.count = 0;
    }
  }
});
```

```js cleanup
onUnmount(() => {
  $('#demo-methods').component('destroy');
})
```

---

### `watch`

**Type:** `Object<Function | Object>`  
**Context:** `(newValue, oldValue)`

Watch state changes and react with side effects.

**Simple watcher:**
```js
watch: {
  count(newVal, oldVal) {
    console.log(`Count changed: ${oldVal} → ${newVal}`);
  }
}
```

**Deep watcher:**
```js
watch: {
  user: {
    handler(newVal, oldVal) {
      console.log('User object changed');
    },
    deep: true
  }
}
```

**Immediate execution:**
```js
watch: {
  userId: {
    handler(newVal) {
      this.fetchUser(newVal);
    },
    immediate: true  // Run on mount
  }
}
```

**Dotted path:**
```js
watch: {
  // dotted path into state / computed / props
  'page.title'(val) {
    document.title = val
  },
  '$pros.variant'(variant) {
    this.variant = variant
  }
}
```

**Example:**

```html live:no-code
<div id="demo-watch" class="en-card en-card__body">
  <input class="en-input en-mb-2" $:model="searchQuery" placeholder="Type to search...">
  <p class="en-text--muted">Search triggered: <code $:text="searchCount"></code> times</p>
</div>
```

```html
<div id="demo-watch">
  <input $:model="searchQuery" placeholder="Type to search...">
  <p>Search triggered: <code $:text="searchCount"></code> times</p>
</div>
```

```js inject show
$('#demo-watch').component({
  state: {
    searchQuery: '',
    searchCount: 0
  },
  watch: {
    searchQuery(newVal, oldVal) {
      if (newVal !== oldVal) {
        this.searchCount++;
      }
    }
  }
});
```

```js cleanup
onUnmount(() => {
  $('#demo-watch').component('destroy');
})
```

**See:** [Instance Methods Guide](./instance-methods.md#watch)

---

## Lifecycle Options

### `created()`

**Type:** `Function`  
**When:** Before mounting, after initialization

Called after component instance is created but before DOM mounting. State is available, but `$el` and `$refs` are not.

```js
created() {
  console.log('Component created');
  console.log('State:', this.$state);
  // Can initialize data, setup watchers
}
```

**Use for:**
- Initial data setup
- Registering watchers
- Fetching non-async data

---

### `mounted()`

**Type:** `Function`  
**When:** After component is mounted to DOM

Called after component is inserted into DOM. `$el` and `$refs` are available.

```js
mounted() {
  console.log('Component mounted');
  console.log('Root element:', this.$el);
  console.log('Refs:', this.$refs);
  // Can access DOM, setup libraries
}
```

**Use for:**
- DOM manipulation
- Third-party library initialization
- Focus elements
- Start timers/intervals

---

### `updated()`

**Type:** `Function`  
**When:** After state changes and DOM updates

Called after any state change that affects the DOM.

```js
updated() {
  console.log('Component updated');
  // React to DOM updates
}
```

**Use for:**
- Post-update DOM operations
- Analytics tracking
- Scroll position adjustments

**⚠️ Warning:** Avoid modifying state here (causes infinite loop).

---

### `destroyed()`

**Type:** `Function`  
**When:** After component is destroyed

Called when component is destroyed. Cleanup phase.

```js
destroyed() {
  console.log('Component destroyed');
  // Cleanup
}
```

**Use for:**
- Clear timers/intervals
- Remove event listeners
- Cancel pending requests
- Cleanup third-party libraries

**Example:**

```html live:no-code
<div id="demo-lifecycle" class="en-card en-card__body">
  <p>Check browser console for lifecycle logs</p>
  <p>Time alive: <code $:text="secondsAlive"></code> seconds</p>
  <pre $:text="messages"/>
</div>
```

```html
<div id="demo-lifecycle">
  <p>Check browser console for lifecycle logs</p>
  <p>Time alive: <code $:text="secondsAlive"></code> seconds</p>
  <pre $:text="logs"/>
</div>
```

```js inject show
$('#demo-lifecycle').component({
  state: {
    secondsAlive: 0,
    messages: [],
  },
  created(ctx) {
    ctx.messages.push('[Lifecycle Demo] Component created')
  },
  computed: {
    logs: (c) => c.messages.join('\n'),
  },
  mounted(ctx) {
    console.log('mounted', this.$state)
    console.log('mounted', this.messages)
    ctx.messages.push('[Lifecycle Demo] Component mounted');
    this.timer = setInterval(() => {
      ctx.secondsAlive++;
      ctx.messages.push('[Lifecycle Demo] Component updated, seconds: ' + ctx.secondsAlive);
      if (ctx.secondsAlive > 10) {
        this.$destroy();
      }
    }, 1000);
  },
  updated(ctx) {
    console.log(this.secondsAlive)
  },
  destroyed(ctx) {
    ctx.messages.push('[Lifecycle Demo] Component destroyed');
    clearInterval(this.timer);
  }
});
```

<!--
mounted(ctx) {
  this.timer = setInterval(() => ctx.secondsAlive++, 1000);
  this.$watch('secondsAlive', (val) => {
    if (val >= 10) this.$destroy();
  });
}
-->

```js cleanup
onUnmount(() => {
  $('#demo-lifecycle').component('destroy');
})
```

**See:** [Lifecycle Guide](./lifecycle.md)

---

## Component Options

### `components`

**Type:** `Object`  
**Purpose:** Register nested components

Define child components used in template.

```js
components: {
  'user-card': {
    props: { userId: null },
    template: '<div>...</div>'
  },
  'todo-item': {
    props: { todo: null },
    template: '<li>...</li>'
  }
}
```

**Usage:**
```html
<div $:component="user-card" $:prop:userId="123"></div>
```

**See:** [Component Options Guide](./component-options.md#components)

---

### `template`

**Type:** `String | Element`  
**Purpose:** Inline template

Provide template as string or DOM element selector.

**String:**
```js
template: '<div><h1>Title</h1><p>Content</p></div>'
```

**Selector:**
```js
template: '#my-template'
```

```html
<template id="my-template">
  <div>...</div>
</template>
```

**Note:** Most components use `<template>` tags in HTML instead.

---

### `render()`

**Type:** `Function`  
**Purpose:** Programmatic rendering (JQR-H only)

Use render functions instead of templates. Requires `@en32/jqr-h` package.

```js
render: (h, ctx) => 
  h('div', {}, [
    h('h1', { text: (s) => s.title }),
    h('button', {
      on: { click: () => ctx.save() },
      text: 'Save'
    })
  ])
```

**See:** [JQR-H Documentation](../../jqr-h/README.md)

---

## Special Properties (Available in Lifecycle/Methods)

Inside lifecycle hooks and methods, `this.$*` provides:

### Component Instance

| Property | Type | Description |
|----------|------|-------------|
| `this.$state` | Object | Reactive state |
| `this.$props` | Object | Component props (read-only) |
| `this.$computed` | Object | Computed values |
| `this.$el` | jQuery | Root element |
| `this.$refs` | Object | Named references |

### Instance Methods

| Method | Description |
|--------|-------------|
| `this.$watch(expr, handler)` | Create watcher |
| `this.$emit(event, data)` | Emit event to parent |
| `this.$refetch(key?)` | Refetch async state |
| `this.$destroy()` | Destroy component |

**Example:**
```js
methods: {
  save() {
    // Access state
    console.log(this.user);
    
    // Access computed
    console.log(this.isValid);
    
    // Access DOM
    this.$el.addClass('saving');
    
    // Emit event
    this.$emit('save', this.user);
    
    // Create watcher
    this.$watch(() => this.status, (val) => {
      console.log('Status changed:', val);
    });
  }
}
```

---

## Complete Example

```html live:no-code
<div id="demo-complete" class="en-card en-card__body">
  <h4>User Profile Editor</h4>
  
  <div $:if="userLoading">Loading profile...</div>
  
  <div $:if="!userLoading && user">
    <div class="en-mb-2">
      <label class="en-form-label">Name</label>
      <input class="en-input" $:model.trim="user.name">
    </div>
    
    <div class="en-mb-2">
      <label class="en-form-label">Email</label>
      <input class="en-input" $:model.trim="user.email" type="email">
    </div>
    
    <p class="en-text--muted">
      <strong>Is Valid:</strong> <code $:text="isValid ? 'Yes' : 'No'"></code>
    </p>
    
    <button 
      class="en-btn en-btn--sm en-btn--primary" 
      $:click="saveProfile()" 
      $:attr:disabled="!isValid">
      Save Changes
    </button>
  </div>
</div>
```

```html
<div id="demo-complete">
  <h4>User Profile Editor</h4>
  <div $:if="userLoading">Loading profile...</div>
  <div $:if="!userLoading && user">
    <div>
      <label>Name</label>
      <input $:model.trim="user.name">
    </div>
    <div>
      <label>Email</label>
      <input $:model.trim="user.email" type="email">
    </div>
    <p><strong>Is Valid:</strong> <code $:text="isValid ? 'Yes' : 'No'"></code></p>
    <button $:click="saveProfile()" $:attr:disabled="!isValid">Save Changes</button>
  </div>
</div>
```

```js inject show
$('#demo-complete').component({
  asyncState: {
    async user() {
      // Simulate API call
      await new Promise(resolve => setTimeout(resolve, 500));
      return { name: 'Alice Smith', email: 'alice@example.com' };
    }
  },
  
  computed: {
    isValid: (c) => {
      if (!c.user) return false;
      return c.user.name.length > 0 && 
             c.user.email.includes('@');
    }
  },
  
  methods: {
    async saveProfile() {
      // Simulate save
      alert(`Saving profile:\nName: ${this.user.name}\nEmail: ${this.user.email}`);
    }
  },
  
  watch: {
    'user.name'(newVal) {
      console.log('Name changed to:', newVal);
    }
  },
  
  mounted() {
    console.log('[Profile Editor] Mounted');
  }
});
```

```js cleanup
onUnmount(() => {
  $('#demo-complete').component('destroy');
})
```

---

## Options Summary Table

| Option | Type | Purpose |
|--------|------|---------|
| `state` | Object | Reactive state |
| `props` | Object | Component properties |
| `computed` | Object | Derived values |
| `asyncState` | Object | Async data |
| `methods` | Object | Functions |
| `watch` | Object | State watchers |
| `created` | Function | Pre-mount lifecycle |
| `mounted` | Function | Post-mount lifecycle |
| `updated` | Function | Post-update lifecycle |
| `destroyed` | Function | Cleanup lifecycle |
| `components` | Object | Nested components |
| `template` | String | Inline template |
| `render` | Function | Render function (JQR-H) |

---

## Best Practices

### 1. Keep State Flat

```js
// ✅ Good - flat structure
state: {
  firstName: 'John',
  lastName: 'Doe',
  email: 'john@example.com'
}

// ❌ Bad - unnecessary nesting
state: {
  user: {
    personal: {
      name: {
        first: 'John',
        last: 'Doe'
      }
    }
  }
}
```

### 2. Use Computed for Derived Data

```js
// ✅ Good - computed
computed: {
  fullName: (c) => `${c.firstName} ${c.lastName}`
}

// ❌ Bad - storing derived data in state
state: {
  firstName: 'John',
  lastName: 'Doe',
  fullName: 'John Doe'  // Duplicate!
}
```

### 3. Cleanup in destroyed()

```js
mounted() {
  this.timer = setInterval(() => {...}, 1000);
},
destroyed() {
  clearInterval(this.timer);  // ✅ Always cleanup
}
```

### 4. Don't Mutate Props

```js
// ❌ Bad - mutating prop
methods: {
  updateUserId() {
    this.$props.userId = 123;  // ERROR!
  }
}

// ✅ Good - emit event
methods: {
  updateUserId() {
    this.$emit('update:userId', 123);
  }
}
```

---

## Next Steps

- [Reactivity Guide](./reactivity.md) - Understanding reactive state
- [Lifecycle Guide](./lifecycle.md) - Component lifecycle in depth
- [Instance Methods](./instance-methods.md) - Available methods
- [Async State Guide](./async-state.md) - Data fetching patterns
- [Props Guide](./props.md) - Component communication

