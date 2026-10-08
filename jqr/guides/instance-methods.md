<!-- {
  "path": "jqr/guides/instance-methods",
  "name": "docs-jqr-guides-instance-methods",
  "meta": {
    "title": "Instance Methods",
    "order": 8
  }
} -->

# Instance Methods

JQR component instances provide methods for programmatic control over your components. These methods let you watch state changes, manually trigger updates, manage props, and more.

## Accessing the Instance

The component instance is returned when you call `.component()`:

```js
const myComponent = $('#app').component({
  state: { count: 0 }
});

// Access instance methods
myComponent.$watch('count', (newVal, oldVal) => {
  console.log(`Count changed from ${oldVal} to ${newVal}`);
});
```

## Core Instance Methods

### `watch()` - Reactive Watching

Watch reactive state and run a callback when it changes:

```html live:no-code
<div id="demo-watch-basic" class="en-card">
  <div class="en-card__body">
    <h3>Watch Method</h3>
    <div class="en-flex en-gap-2" style="margin-bottom: 1rem;">
      <button class="en-btn en-btn--primary" $:click="count++">Increment</button>
    </div>
    <div class="en-alert en-alert--info">
      <div>Count: <strong $:text="count"></strong></div>
      <div>Watch fired: <strong $:text="watchCount"></strong> times</div>
    </div>
  </div>
</div>
```

```html
<div id="demo-watch-basic">
  <button $:click="count++">Increment</button>
  <div>Count: <strong $:text="count"></strong></div>
  <div>Watch fired: <strong $:text="watchCount"></strong> times</div>
</div>
```

```js inject show
const watchDemo = $('#demo-watch-basic').component({
  state: { 
    count: 0,
    watchCount: 0
  },
  constants: {
    hello: 'World',
  },
  mounted() {
    console.log(this.hello)
  }
});

// Watch a single state key
watchDemo.$watch('count', (newVal, oldVal) => {
  watchDemo.$state.watchCount++;
  console.log(`Count: ${oldVal} → ${newVal}`);
});
```

```js cleanup
onUnmount(() => {
  $('#demo-watch-basic').component('destroy');
})
```

**Syntax:**
```js
component.$watch(source, callback, options?)
```

**Parameters:**
- `source`: String (state key) or function (getter)
- `callback`: `(newValue, oldValue) => void`
- `options`: `{ deep?: boolean }`

### Watch with Getter Function

Watch computed values or nested paths:

```html live:no-code
<div id="demo-watch-getter" class="en-card">
  <div class="en-card__body">
    <h3>Watch with Getter</h3>
    <div class="en-flex en-gap-2" style="margin-bottom: 1rem;">
      <input class="en-input" $:model="firstName" placeholder="First name">
      <input class="en-input" $:model="lastName" placeholder="Last name">
    </div>
    <div class="en-alert en-alert--info">
      <div>Full name: <strong $:text="firstName + ' ' + lastName"></strong></div>
      <div>Changes: <strong $:text="nameChanges"></strong></div>
    </div>
  </div>
</div>
```

```html
<div id="demo-watch-getter">
  <input $:model="firstName" placeholder="First name">
  <input $:model="lastName" placeholder="Last name">
  <div>Full name: <strong $:text="firstName + ' ' + lastName"></strong></div>
  <div>Changes: <strong $:text="nameChanges"></strong></div>
</div>
```

```js inject show
const watchGetter = $('#demo-watch-getter').component({
  state: { 
    firstName: '',
    lastName: '',
    nameChanges: 0
  }
});

// Watch a computed value
watchGetter.$watch(
  () => `${watchGetter.$state.firstName} ${watchGetter.$state.lastName}`,
  (newName, oldName) => {
    watchGetter.$state.nameChanges++;
    console.log(`Name changed: ${oldName} → ${newName}`);
  }
);
```

```js cleanup
onUnmount(() => {
  $('#demo-watch-getter').component('destroy');
})
```

### Deep Watch

Watch for in-place mutations of nested objects/arrays:

```html live:no-code
<div id="demo-deep-watch" class="en-card">
  <div class="en-card__body">
    <h3>Deep Watch</h3>
    <div class="en-flex en-gap-2" style="margin-bottom: 1rem;">
      <button class="en-btn en-btn--primary" $:click="changeName">Change Name</button>
      <button class="en-btn en-btn--primary" $:click="increaseAge">Increase Age</button>
    </div>
    <div class="en-alert en-alert--info">
      <div>User: <strong $:text="user.name + ', ' + user.age"></strong></div>
      <div>Deep changes: <strong $:text="deepChanges"></strong></div>
    </div>
  </div>
</div>
```

```html
<div id="demo-deep-watch">
  <button $:click="changeName">Change Name</button>
  <button $:click="increaseAge">Increase Age</button>
  <div>User: <strong $:text="user.name + ', ' + user.age"></strong></div>
  <div>Deep changes: <strong $:text="deepChanges"></strong></div>
</div>
```

```js inject show
const deepWatch = $('#demo-deep-watch').component({
  state: { 
    user: { name: 'John', age: 25 },
    deepChanges: 0
  },
  
  methods: {
    changeName() {
      this.user.name = this.user.name === 'John' ? 'Jane' : 'John';
    },
    increaseAge(state) {
      this.user.age++;
    }
  }
});

// Deep watch detects nested changes
deepWatch.$watch(
  () => deepWatch.$state.user,
  (newVal, oldVal) => {
    deepWatch.$state.deepChanges++;
    console.log('User changed:', newVal);
  },
  { deep: true }  // Enable deep watching
);
```

```js cleanup
onUnmount(() => {
  $('#demo-deep-watch').component('destroy');
})
```

**Without `deep: true`**, the watch only fires when the object reference itself changes:

```js
// Only fires on reference change
component.$watch(() => component.$state.user, callback);

component.$state.user = { ...newUser }; // ✅ Fires
component.$state.user.name = 'Jane';    // ❌ Doesn't fire

// Fires on nested mutations too
component.$watch(() => component.$state.user, callback, { deep: true });

component.$state.user.name = 'Jane';    // ✅ Now fires!
```

## Async State Methods

### `refetch()` - Manually Refetch

Force an `asyncState` fetcher to re-run:

```html live:no-code
<div id="demo-refetch" class="en-card">
  <div class="en-card__body">
    <h3>Refetch Method</h3>
    <div class="en-flex en-gap-2" style="margin-bottom: 1rem;">
      <button class="en-btn en-btn--primary" $:click="refetchUsers">
        Refresh Users
      </button>
    </div>
    <div $:show="usersLoading" class="en-alert en-alert--info">
      Loading...
    </div>
    <div $:show="!usersLoading && users">
      <div $:for="user in (users || []).slice(0, 3)" $:key="user.id" class="en-flex en-gap-2 en-items-center" style="margin-bottom: 0.5rem;">
        <img $:attr:src="user.avatar" $:attr:alt="user.name" style="width: 32px; height: 32px; border-radius: 50%;">
        <strong $:text="user.name"></strong>
      </div>
    </div>
  </div>
</div>
```

```html
<div id="demo-refetch">
  <button $:click="refetchUsers">Refresh Users</button>
  <div $:show="usersLoading">Loading...</div>
  <div $:show="!usersLoading && users">
    <div $:for="user in users" $:key="user.id">
      <img $:attr:src="user.avatar" $:attr:alt="user.name">
      <strong $:text="user.name"></strong>
    </div>
  </div>
</div>
```

```js inject show
const refetchDemo = $('#demo-refetch').component({
  asyncState: {
    async users() {
      const response = await fetch('/assets/api/jqr-docs/users.json');
      return response.json();
    }
  },
  methods: {
    refetchUsers() {
      // Manually trigger a refetch
      refetchDemo.$refetch('users');
    }
  }
});

```

```js cleanup
onUnmount(() => {
  $('#demo-refetch').component('destroy');
})
```

**Usage:**
```js
component.$refetch(key)
```

- Forces immediate re-run of the async fetcher
- Bypasses dependency tracking
- Useful for "Retry" buttons or manual refresh

### `cancelAsync()` - Cancel Requests

Invalidate in-flight requests:

```js
const component = $('#app').component({
  asyncState: {
    async data() {
      return fetch('/api/data').then(r => r.json());
    }
  }
});

// Cancel pending request
component.$cancelAsync('data');

// Optionally refetch fresh data
component.$refetch('data');
```

**Usage:**
```js
component.$cancelAsync(key)
```

- Invalidates pending requests
- Marks key as no longer loading
- Does not trigger a new fetch

## Props Management

### `setProps()` - Update Props

Programmatically update component props:

```html live:no-code
<div id="demo-props" class="en-card">
  <div class="en-card__body">
    <h3>setProps Method</h3>
    <div class="en-flex en-gap-2" style="margin-bottom: 1rem;">
      <button class="en-btn en-btn--primary" $:click="updateTheme('dark')">Dark</button>
      <button class="en-btn en-btn--primary" $:click="updateTheme('light')">Light</button>
    </div>
    <div class="en-alert" $:class="{ 'en-alert--success': $props.theme === 'dark', 'en-alert--info': $props.theme === 'light' }">
      Current theme: <strong $:text="$props.theme"></strong>
    </div>
  </div>
</div>
```

```html
<div id="demo-props">
  <button $:click="updateTheme('dark')">Dark</button>
  <button $:click="updateTheme('light')">Light</button>
  <div>Current theme: <strong $:text="theme"></strong></div>
</div>
```

```js inject show
const propsDemo = $('#demo-props').component({
  props: { theme: 'light' },
  state: {},
  methods: {
    updateTheme(newTheme) {
      propsDemo.$setProps({ theme: newTheme });
    }
  }
});
```

```js cleanup
onUnmount(() => {
  $('#demo-props').component('destroy');
})
```

**Usage:**
```js
component.$setProps({ key: newValue })
```

- Updates props reactively
- Triggers re-renders where props are used
- Useful for parent-child communication

## Event Methods

### `emit()` - Emit Custom Events

Emit custom events from a component:

```js
const childComponent = $('#child').component({
  methods: {
    notifyParent(state) {
      // Emit event with payload
      childComponent.emit('update', { value: state.count });
    }
  }
});

// Parent listens with $:on
// <div $:component="child" $:on:update="handleUpdate($event)"></div>
```

**Usage:**
```js
component.$emit(eventName, payload?)
```

- Triggers `jqr:${eventName}` jQuery event
- Payload available as `$event.detail` or second argument
- Used for nested component communication

### `on()` - Delegate Event Listeners

Add delegated event listeners:

```js
const listComponent = $('#list').component({
  state: { items: ['A', 'B', 'C'] }
});

// Listen to clicks on .item elements
listComponent.on('click', '.item', (state, event) => {
  console.log('Item clicked:', event.target.textContent);
});
```

**Usage:**
```js
component.$on(eventName, selector, callback)
```

- Uses jQuery event delegation
- Automatically passes `state` to callback
- Works with dynamically added elements

## DOM Query Methods

### `bind()` - Manual Binding

Manually bind state to elements:

```js
const component = $('#app').component({
  state: { message: 'Hello' }
});

// Bind state to elements
component.$bind('.message', 'text', (state) => state.message);
component.$bind('.status', 'class', (state) => state.active ? 'active' : '');
component.$bind('.count', 'data-count', (state) => state.count);
```

**Usage:**
```js
component.$bind(selector, attrOrType, getter)
```

- Reactive binding outside directives
- Types: `'text'`, `'html'`, `'show'`, `'class'`, or any attribute
- Less common than directives

## Lifecycle & Cleanup

### `destroy()` - Destroy Component

Manually destroy a component:

```js
const component = $('#app').component({
  state: { count: 0 }
});

// Later, clean up
component.destroy();
```

**What happens:**
- Calls `destroyed` hook
- Removes all event listeners
- Stops all reactive effects
- Clears internal disposers
- Removes component data from element

**Usage:**
```js
component.destroy()
```

### `onDispose()` - Register Cleanup

Register additional cleanup callbacks:

```js
const component = $('#app').component({
  mounted() {
    const ws = new WebSocket('ws://localhost:8080');
    
    // Register cleanup
    component.onDispose(() => {
      ws.close();
      console.log('WebSocket closed');
    });
  }
});
```

**Usage:**
```js
component.onDispose(callback)
```

- Runs on `destroy()`
- Called alongside `destroyed`
- Useful for external library cleanup

## State Access

### Direct State Access

Access and modify state from outside:

```js
const component = $('#app').component({
  state: { count: 0 }
});

// Read state
console.log(component.state.count); // 0

// Modify state (triggers reactivity)
component.state.count = 10;

// Access props (read-only)
console.log(component.props.theme);
```

### Computed Access

Access computed properties:

```js
const component = $('#app').component({
  state: { count: 0 },
  computed: {
    double(state) {
      return state.count * 2;
    }
  }
});

// Access computed value through state
console.log(component.state.count);   // 0
// Computed properties are internal, accessed via expressions
```

## Complete Example: Dashboard

```html live:no-code
<div id="demo-dashboard" class="en-card">
  <div class="en-card__body">
    <h3>Dashboard Controls</h3>
    <div class="en-flex en-gap-2" style="margin-bottom: 1rem;">
      <button class="en-btn en-btn--primary" $:click="increment">+</button>
      <button class="en-btn en-btn--danger" $:click="reset">Reset</button>
      <button class="en-btn en-btn--outline" $:click="toggleTheme">Toggle Theme</button>
    </div>
    <div class="en-alert" $:class="{ 'en-alert--success': isDark, 'en-alert--info': !isDark }">
      <div>Count: <strong $:text="count"></strong></div>
      <div>Theme: <strong $:text="theme"></strong></div>
      <div>Total changes: <strong $:text="changeCount"></strong></div>
    </div>
  </div>
</div>
```

```html
<div id="demo-dashboard">
  <h3>Dashboard Controls</h3>
  <button $:click="increment">+</button>
  <button $:click="reset">Reset</button>
  <button $:click="toggleTheme">Toggle Theme</button>
  <div>
    <div>Count: <strong $:text="count"></strong></div>
    <div>Theme: <strong $:text="theme"></strong></div>
    <div>Total changes: <strong $:text="changeCount"></strong></div>
  </div>
</div>
```

```js inject show
const dashboard = $('#demo-dashboard').component({
  state: {
    count: 0,
    theme: 'light',
    changeCount: 0
  },
  
  computed: {
    isDark(state) {
      return state.theme === 'dark';
    }
  },
  
  methods: {
    increment(state) {
      state.count++;
    },
    
    reset(state) {
      state.count = 0;
    },
    
    toggleTheme(state) {
      state.theme = state.theme === 'light' ? 'dark' : 'light';
    }
  },
  
  mounted() {
    // Watch for any state changes
    dashboard.$watch(
      () => dashboard.$state,
      () => {
        dashboard.$state.changeCount++;
      },
      { deep: true }
    );
    
    // Log theme changes
    dashboard.$watch('theme', (newTheme, oldTheme) => {
      console.log(`Theme switched: ${oldTheme} → ${newTheme}`);
    });
  }
});
```

```js cleanup
onUnmount(() => {
  $('#demo-dashboard').component('destroy');
})
```

## Best Practices

### 1. Store Component Reference

```js
// ✅ Good - store reference for later use
const myComponent = $('#app').component({...});
myComponent.$watch('count', callback);

// ❌ Bad - can't access instance methods
$('#app').component({...});
```

### 2. Clean Up Watchers

Watchers are automatically cleaned up on `destroy()`:

```js
const component = $('#app').component({
  mounted() {
    // This watcher is auto-cleaned on destroy
    component.$watch('count', (val) => {
      console.log('Count:', val);
    });
  }
});
```

### 3. Use Getters for Computed Watches

```js
// ✅ Good - watches computed expression
component.$watch(
  () => component.state.firstName + ' ' + component.state.lastName,
  callback
);

// ✅ Also good - watches single key
component.$watch('firstName', callback);
```

### 4. Deep Watch Only When Needed

```js
// ✅ Good - primitive value, no deep watch needed
component.$watch('count', callback);

// ✅ Good - nested object, deep watch enabled
component.$watch('user', callback, { deep: true });

// ❌ Wasteful - deep watch on primitive
component.$watch('count', callback, { deep: true });
```

### 5. Emit Declared Events

```js
// ✅ Good - declare emitted events
const component = $('#child').component({
  emits: ['update', 'delete'],
  methods: {
    notifyParent() {
      component.emit('update', { data: 'value' });
    }
  }
});

// ⚠️ Warning - undeclared event (still works)
component.emit('unknown');  // Console warning
```

## Performance Tips

### Debounce Watchers

For expensive operations, debounce your watchers:

```js
let timeoutId;

component.$watch('searchQuery', (newQuery) => {
  clearTimeout(timeoutId);
  timeoutId = setTimeout(() => {
    // Expensive search operation
    performSearch(newQuery);
  }, 300);
});
```

### Use Specific Watches

```js
// ✅ Efficient - watches specific key
component.$watch('theme', callback);

// ❌ Inefficient - watches entire state
component.$watch(() => component.state, callback, { deep: true });
```

### Batch Updates

Group related updates together:

```js
// ✅ Good - single batch update
component.state.firstName = 'John';
component.state.lastName = 'Doe';
component.state.age = 30;
// All watchers run once in next microtask

// ❌ Less efficient - but still works
function updateUser() {
  component.state.firstName = 'John';
  await nextTick();  // Unnecessary wait
  component.state.lastName = 'Doe';
}
```

## Next Steps

- [Lifecycle](./lifecycle.md) - Component lifecycle hooks
- [Async State](./async-state.md) - Declarative data fetching
- [Transitions](./transitions.md) - Animations and transitions

