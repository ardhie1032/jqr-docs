<!-- {
  "path": "jqr/guides/nested-components",
  "name": "docs-jqr-guides-nested-components",
  "meta": {
    "title": "Nested Components",
    "order": 15
  }
} -->

# Nested Components

Nested components allow you to compose complex UIs by embedding one component inside another. This guide covers patterns for building reusable, composable components.

## Basic Nesting

### Using Registered Components

Register a child component globally:

```js
$.jqr.register('user-badge', {
  props: {
    name: { default: 'Guest' },
    role: { default: 'User' }
  },
  template: `
    <span class="badge">
      <strong $:text="$props.name"></strong> 
      (<span $:text="$props.role"></span>)
    </span>
  `
});
```

Use it in a parent:

```html
<div $:component:state="{ username: 'Alice', userRole: 'Admin' }">
  <div $:component="user-badge" 
       $:prop:name="username" 
       $:prop:role="userRole">
  </div>
</div>
```

### Inline Nested Components

Define components inline without global registration:

```html
<div id="app">
  <div $:component="user-card">
    <!-- Child component defined inline -->
    <div $:component:state="{ expanded: false }">
      <button $:click="expanded = !expanded">Toggle Details</button>
      <div $:if="expanded">
        <p>User details here...</p>
      </div>
    </div>
  </div>
</div>
```

## Communication Patterns

### Parent to Child: Props

Pass data down via `$:prop:*`:

```html live:no-code
<div id="demo-parent-to-child" class="en-card en-card__body">
  <div class="en-form-group en-mb-3">
    <label class="en-label">User Name:</label>
    <input class="en-input" $:model="parentName" />
  </div>
  <div $:component="demo-child-display" 
       $:prop:username="parentName">
  </div>
</div>
```

```html
<div id="demo-parent-to-child">
  <label>User Name:</label>
  <input $:model="parentName" />
  
  <div $:component="child-display" 
       $:prop:username="parentName">
  </div>
</div>
```

```js inject show
JQR.register('demo-child-display', {
  props: { username: { default: 'Guest' } },
  template: `<p>Hello, <strong $:text="$props.username"></strong>!</p>`
});

$('#demo-parent-to-child').component({
  state: { parentName: 'Alice' }
});
```

```js cleanup
onUnmount(() => {
  $('#demo-parent-to-child').component('destroy');
})
```

### Child to Parent: Events

Emit events from child to parent:

```html live:no-code
<div id="demo-child-to-parent" class="en-card en-card__body">
  <p class="en-mb-3">Notifications: <code $:text="notifications.length"></code></p>
  <div $:component="demo-notification-button" 
       $:on:notify="notifications.push($event)">
  </div>
  <ul class="en-list-group en-mt-3">
    <li class="en-list-group__item" $:for="msg, i in notifications" $:key="i" $:text="msg"></li>
  </ul>
</div>
```

```html
<div id="demo-child-to-parent">
  <p>Notifications: <code $:text="notifications.length"></code></p>
  
  <div $:component="demo-notification-button" 
       $:on:notify="notifications.push($event)">
  </div>
  
  <ul>
    <li $:for="msg, i in notifications" $:key="i" $:text="msg"></li>
  </ul>
</div>
```

```js inject show
JQR.register('demo-notification-button', {
  template: `
    <button class="en-btn" $:click="notify">Send Notification</button>
  `,
  methods: {
    notify() {
      this.$emit('notify', `Notification at ${new Date().toLocaleTimeString()}`);
    }
  }
});

$('#demo-child-to-parent').component({
  state: { notifications: [] }
});
```

```js cleanup
onUnmount(() => {
  $('#demo-child-to-parent').component('destroy');
})
```

### Two-Way Binding: v-model Pattern

Use `$:model:<prop>` for two-way binding:

```html live:no-code
<div id="demo-two-way" class="en-card en-card__body">
  <p class="en-mb-3">Parent count: <code $:text="count"></code></p>
  <div class="en-flex en-gap-2">
    <div $:component="demo-counter-control" $:model:value="count"></div>
    <button class="en-btn en-btn--sm en-btn--outline" $:click="count = 0">Reset</button>
  </div>
</div>
```

```html
<div id="demo-two-way">
  <p>Parent count: <code $:text="count"></code></p>
  
  <div $:component="counter-control" $:model:value="count"></div>
  <button $:click="count = 0">Reset</button>
</div>
```

```js inject show
JQR.register('demo-counter-control', {
  props: { value: { default: 0 } },
  template: `
    <div>
      <button $:click="decrement()">-</button>
      <span $:text="$props.value"></span>
      <button $:click="increment()">+</button>
    </div>
  `,
  methods: {
    increment() {
      this.$emit('update:value', this.$props.value + 1);
    },
    decrement() {
      this.$emit('update:value', this.$props.value - 1);
    }
  }
});

$('#demo-two-way').component({
  state: { count: 5 }
});
```

```js cleanup
onUnmount(() => {
  $('#demo-two-way').component('destroy');
})
```

## Local Component Registration

Register components locally (scoped to one parent):

```js
$('#app').component({
  state: { /* ... */ },
  components: {
    // Only available inside this component
    'local-badge': {
      props: { text: {} },
      template: '<span class="badge" $:text="$props.text"></span>'
    },
    'local-card': {
      template: '<div class="card"><slot $:default></slot></div>'
    }
  }
});
```

```html
<div id="app">
  <!-- These only work inside #app -->
  <div $:component="local-badge" $:prop:text="'VIP'"></div>
  <div $:component="local-card">
    <p>Card content</p>
  </div>
</div>
```

**Benefits:**
- **Name isolation:** No global naming conflicts
- **Encapsulation:** Component only used where needed
- **Performance:** Smaller global registry

**Priority:** Local components checked **before** global registry.

## Slot Composition

### Passing Content to Children

```html live:no-code
<div id="demo-slots" class="en-card en-card__body">
  <div $:component="demo-card-wrapper">
    <template $:slot="header">
      <h3 class="en-heading-3">Card Title</h3>
    </template>
    <p class="en-mb-0">This is the main content area.</p>
    <template $:slot="footer">
      <button class="en-btn en-btn--sm en-btn--primary">Action</button>
    </template>
  </div>
</div>
```

```html
<div id="demo-slots">
  <div $:component="card-wrapper">
    <template $:slot="header">
      <h3>Card Title</h3>
    </template>
    
    <p>This is the main content area.</p>
    
    <template $:slot="footer">
      <button>Action</button>
    </template>
  </div>
</div>
```

```js inject show
JQR.register('demo-card-wrapper', {
  template: `
    <div class="card">
      <div class="card-header">
        <slot $:name="header"></slot>
      </div>
      <div class="card-body">
        <slot $:default></slot>
      </div>
      <div class="card-footer">
        <slot $:name="footer"></slot>
      </div>
    </div>
  `
});

$('#demo-slots').component({ state: {} });
```

```js cleanup
onUnmount(() => {
  $('#demo-slots').component('destroy');
})
```

### Scoped Slots

Child exposes data to parent via slot scope:

```js
JQR.register('user-list', {
  props: { users: { default: () => [] } },
  template: `
    <ul>
      <li $:for="(user, i) in $props.users" $:key="i">
        <slot $:default $:scope="{ user: user, index: i }"></slot>
      </li>
    </ul>
  `
});
```

```html
<div $:component="user-list" $:prop:users="users">
  <template $:slot="default" $:slot:scope="slotProps">
    <strong $:text="slotProps.user.name"></strong>
    (#<span $:text="slotProps.index"></span>)
  </template>
</div>
```

## Deep Nesting

### Multi-Level Hierarchy

```js
// Grandchild
JQR.register('status-badge', {
  props: { status: {} },
  template: '<span class="badge" $:text="$props.status"></span>'
});

// Child
JQR.register('user-info', {
  props: { user: {} },
  template: `
    <div>
      <p $:text="$props.user.name"></p>
      <div $:component="status-badge" 
           $:prop:status="$props.user.status">
      </div>
    </div>
  `
});

// Parent
$('#app').component({
  state: {
    currentUser: { name: 'Alice', status: 'Online' }
  }
});
```

```html
<div id="app">
  <div $:component="user-info" $:prop:user="currentUser"></div>
</div>
```

**Data flow:**
```
Parent ($state.currentUser)
  → user-info ($props.user)
    → status-badge ($props.status)
```

### Avoid Prop Drilling

For deeply nested components, prefer:

**1. Shared reactive objects** (see [State Management](./state-management.md)):

```js
// Shared store
const userStore = $.jqr.reactive({ currentUser: null });

// Any component can access
JQR.register('deep-child', {
  computed: {
    user() {
      return userStore.currentUser;
    }
  }
});
```

**2. Event bus pattern:**

```js
const eventBus = $('<div>');

// Deep child emits
eventBus.trigger('user:updated', [userData]);

// Top parent listens
eventBus.on('user:updated', (e, data) => {
  this.user = data;
});
```

## Composition Patterns

### Container/Presentational

**Container (smart):** Handles logic, data fetching

```js
JQR.register('user-container', {
  state: { user: null, loading: false },
  asyncState: {
    async users() {
      const res = await fetch('/api/user');
      return res.json();
    }
  },
  template: `
    <div>
      <div $:if="users.loading">Loading...</div>
      <div $:if="users.data">
        <div $:component="user-presenter" 
             $:prop:user="users.data">
        </div>
      </div>
    </div>
  `
});
```

**Presentational (dumb):** Only displays data

```js
$.jqr.register('user-presenter', {
  props: { user: {} },
  template: `
    <div class="user-card">
      <h3 $:text="$props.user.name"></h3>
      <p $:text="$props.user.email"></p>
    </div>
  `
});
```

### Layout Components

```js
$.jqr.register('two-column-layout', {
  template: `
    <div class="layout">
      <aside class="sidebar">
        <slot $:name="sidebar"></slot>
      </aside>
      <main class="main">
        <slot $:default></slot>
      </main>
    </div>
  `
});
```

```html
<div $:component="two-column-layout">
  <template $:slot="sidebar">
    <nav>Navigation here</nav>
  </template>
  
  <article>Main content here</article>
</div>
```

### Higher-Order Components

Wrap components to add behavior:

```js
$.jqr.register('with-loading', {
  props: { loading: {}, componentName: {} },
  template: `
    <div>
      <div $:if="$props.loading">Loading...</div>
      <div $:if="!$props.loading">
        <div $:component="$props.componentName">
          <slot $:default></slot>
        </div>
      </div>
    </div>
  `
});
```

```html
<div $:component="with-loading" 
     $:prop:loading="isLoading" 
     $:prop:componentName="'user-card'">
  <!-- Slot content passed to user-card -->
</div>
```

## Lifecycle Coordination

### Parent Waits for Child

```js
$('#app').component({
  state: { childReady: false },
  mounted() {
    // Listen for child ready event
    this.$el.on('jqr:child-ready', () => {
      this.childReady = true;
    });
  }
});

$.jqr.register('child-component', {
  mounted() {
    // Notify parent
    this.$el.parent().trigger('jqr:child-ready');
  }
});
```

### Cleanup Cascades

Parent destruction automatically destroys children:

```js
const parent = $('#app').component({
  state: {}
});

// Later: destroys ALL nested components too
parent.destroy();
```

**No manual cleanup needed** — JQR tracks component hierarchy.

## Performance Considerations

### Lazy Loading

Load heavy components only when needed:

```html
<div $:if="showHeavyComponent">
  <div $:component="heavy-chart"></div>
</div>
```

### Conditional Mounting

```html
<!-- ❌ Bad: Recreates on every toggle -->
<div $:if="showWidget">
  <div $:component="widget"></div>
</div>

<!-- ✅ Better: Keep mounted, hide with CSS -->
<div $:show="showWidget">
  <div $:component="widget"></div>
</div>
```

### Prop Stability

```html
<!-- ❌ Bad: Creates new object every update -->
<div $:component="card" $:prop:config="{ theme: 'dark' }"></div>

<!-- ✅ Good: Stable reference -->
<div $:component="card" $:prop:config="cardConfig"></div>
```

```js
$('#app').component({
  state: {
    cardConfig: { theme: 'dark' }  // Stable object
  }
});
```

## Best Practices

### 1. Clear Communication

```js
// ✅ Good: Explicit props and events
<div $:component="user-card" 
     $:prop:user="currentUser"
     $:on:edit="handleEdit($event)"
     $:on:delete="handleDelete($event)">
</div>

// ❌ Bad: Shared state without clear ownership
<div $:component="user-card"></div>  // Modifies global state directly
```

### 2. Single Responsibility

```js
// ✅ Good: Each component has one purpose
$.jqr.register('user-avatar', { /* just displays avatar */ });
$.jqr.register('user-name', { /* just displays name */ });
$.jqr.register('user-card', { /* composes avatar + name */ });

// ❌ Bad: One component does everything
$.jqr.register('user-everything', { /* avatar, name, posts, comments... */ });
```

### 3. Prop Types

```js
// ✅ Good: Validate props
$.jqr.register('user-card', {
  props: {
    userId: { 
      required: true,
      validator: (val) => typeof val === 'number'
    }
  }
});

// ❌ Bad: No validation
$.jqr.register('user-card', {
  props: { userId: {} }
});
```

### 4. Avoid Deep Nesting

```
// ✅ Good: Flat hierarchy
App
├── Header
├── Sidebar
└── Content
    ├── PostList
    └── Post

// ❌ Bad: Too deep
App
└── Layout
    └── Container
        └── Wrapper
            └── Content
                └── Inner
                    └── Post
```

**Rule of thumb:** Max 3-4 levels deep.

### 5. Name Conventions

```js
// ✅ Good: Clear, descriptive names
$.jqr.register('user-profile-card', { /* ... */ });
$.jqr.register('todo-list-item', { /* ... */ });

// ❌ Bad: Generic, unclear names
$.jqr.register('card', { /* ... */ });  // What kind of card?
$.jqr.register('item', { /* ... */ });  // Item of what?
```

## Common Pitfalls

### 1. Forgetting $:on: Prefix

```html
<!-- ❌ Wrong: DOM click event on parent element -->
<div $:component="button" $:click="handleClick"></div>

<!-- ✅ Correct: Listen for child's $emit -->
<div $:component="button" $:on:click="handleClick"></div>
```

### 2. Direct State Mutation

```js
// ❌ Bad: Child mutates parent state
$.jqr.register('bad-child', {
  props: { user: {} },
  methods: {
    update() {
      this.$props.user.name = 'Changed';  // Mutates parent!
    }
  }
});

// ✅ Good: Child emits event
$.jqr.register('good-child', {
  props: { user: {} },
  methods: {
    update() {
      this.$emit('update', { name: 'Changed' });
    }
  }
});
```

### 3. Missing Keys in Lists

```html
<!-- ❌ Bad: No key -->
<div $:for="item in items" $:component="item-card"></div>

<!-- ✅ Good: With key -->
<div $:for="item in items" $:key="item.id" $:component="item-card"></div>
```

## Summary

**Key Patterns:**

1. **Props down, events up** - Parent passes data, child emits events
2. **Local registration** - Scope components to avoid global conflicts
3. **Slots for content** - Flexible composition via slots
4. **v-model for two-way** - `$:model:<prop>` sugar
5. **Shallow hierarchy** - Avoid deep nesting (max 3-4 levels)

**Communication:**
- Parent → Child: `$:prop:*`
- Child → Parent: `$emit()` + `$:on:*`
- Two-way: `$:model:<prop>`
- Siblings: Shared reactive object or event bus

## Next Steps

- [Props Guide](./props.md) - Deep dive into props
- [Slots Guide](./slots.md) - Slot patterns and scoped slots
- [State Management](./state-management.md) - Sharing state between components
- [Instance Methods](./instance-methods.md) - `$emit`, `$refs`, etc.
- [Component Options](./component-options.md) - Complete component API
