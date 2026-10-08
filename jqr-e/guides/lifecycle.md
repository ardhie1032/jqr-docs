<!-- {
  "path": "jqr-e/guides/lifecycle",
  "name": "docs-jqr-e-guides-lifecycle",
  "meta": {
    "title": "Lifecycle & Mounting",
    "order": 5
  }
} -->

# Lifecycle & Mounting

Understand how custom elements mount, unmount, and who owns their component lifecycle.

## Overview

JQR-E manages component lifecycle based on **ownership**. An element can be mounted by:

1. **Itself** (standalone mounting)
2. **Parent JQR component** (external ownership)
3. **`h()` from JQR-H** (external ownership)

The lifecycle system ensures:
- Components mount at the right time
- Ownership is clear and unambiguous
- Cleanup happens automatically
- Async components load seamlessly

## Ownership Modes

Every custom element operates in one of three modes:

### Idle Mode

Element exists but has no mounted component.

```js
const el = document.createElement('my-button');
// Mode: 'idle' - no component yet
```

### External Mode

Another component owns and manages the lifecycle.

```html
<div $:component:state="{ visible: true }">
  <!-- Parent component mounts this -->
  <my-button $:if="visible"></my-button>
</div>
```

```js
// Mode: 'external' - parent owns lifecycle
// Parent component creates, updates, destroys it
```

### Standalone Mode

Element mounts its own component.

```html
<!-- Mounts itself when connected -->
<my-button></my-button>
```

```js
// Mode: 'standalone' - element owns lifecycle
// Destroyed when disconnected (unless keepAlive: true)
```

## Connection Queue

Elements don't mount immediately in `connectedCallback` because:

**Problem:** During HTML parsing, `connectedCallback` fires at the **start tag** — before children exist.

**Solution:** Elements are queued and mounted together after DOM is ready.

### Queue Process

```html live:no-code
<div id="demo-lifecycle-queue">
  <div class="en-card">
    <connection-demo></connection-demo>
  </div>
</div>
```

```html
<div id="demo-lifecycle-queue">
  <connection-demo></connection-demo>
</div>
```

```js inject show
JQRE.defineElement('connection-demo', {
  state: {
    log: []
  },
  created() {
    this.log.push('1. created() - Component initialized');
  },
  mounted() {
    this.log.push('2. mounted() - Component mounted');
  },
  template: `
    <div>
      <h4>Lifecycle Events:</h4>
      <ol>
        <li $:for="entry in log" $:key="entry" $:text="entry"></li>
      </ol>
    </div>
  `
});
```

```js cleanup
onUnmount(() => {
  $('#demo-lifecycle-queue').component('destroy');
});
```

**Queue Behavior:**

1. **During Parsing:**
   - `connectedCallback` called → element added to queue
   - Mounting deferred until document ready

2. **After `DOMContentLoaded`:**
   - Queue processed in order
   - Outermost elements mount first (depth-first)
   - Parent claims children before they mount standalone

3. **Runtime Insertion:**
   - `connectedCallback` called → added to queue
   - Queue processed on next microtask
   - Immediate for already-parsed documents

### Queue Algorithm

```
1. jqr.scan() runs first
   → Declarative $:component elements become roots
   
2. Process queue by depth (outermost first)
   
3. For each element:
   - Already a component root? → adopt (external mode)
   - Has component ancestor? → wait for parent to claim
   - Otherwise → mount standalone
   
4. Unclaimed elements after timeout → mount standalone
```

## Standalone Mounting

Elements mount themselves when connected to DOM:

### Auto Mount (Default)

```html live:no-code
<div id="demo-lifecycle-auto">
  <div class="en-card">
    <button class="en-btn" onclick="
      const container = document.getElementById('demo-lifecycle-auto');
      const el = document.createElement('auto-button');
      container.querySelector('.demo-container').appendChild(el);
    ">Add Button</button>
    <div class="demo-container" style="margin-top: 10px; min-height: 50px; border: 1px dashed #ccc; padding: 10px"></div>
  </div>
</div>
```

```html
<div id="demo-lifecycle-auto">
  <button onclick="
    const el = document.createElement('auto-button');
    document.getElementById('container').appendChild(el);
  ">Add Button</button>
  
  <div id="container"></div>
</div>
```

```js inject show
JQRE.defineElement('auto-button', {
  state: {
    count: 0
  },
  methods: {
    increment() {
      this.count++;
    }
  },
  template: `
    <button class="en-btn en-btn--sm" $:click="increment">
      Clicks: <span $:text="count"></span>
    </button>
  `
}, {
  mount: 'auto'  // Default behavior
});
```

```js cleanup
onUnmount(() => {
  $('#demo-lifecycle-auto').component('destroy');
});
```

**Flow:**
1. Element connected → added to queue
2. Queue processed → `mountStandalone()` called
3. Component created and mounted
4. `$mounted` promise resolves

### Manual Mount

Require explicit `mount()` call:

```html live:no-code
<div id="demo-lifecycle-manual">
  <div class="en-card">
    <div class="en-flex en-gap-2">
      <button class="en-btn en-btn--sm" onclick="
        const el = document.querySelector('#demo-lifecycle-manual manual-button');
        if (el) el.mount();
      ">Mount</button>
      <button class="en-btn en-btn--sm" onclick="
        const el = document.querySelector('#demo-lifecycle-manual manual-button');
        if (el) el.unmount();
      ">Unmount</button>
    </div>
    <manual-button style="display: block; margin-top: 10px"></manual-button>
  </div>
</div>
```

```html
<div id="demo-lifecycle-manual">
  <manual-button></manual-button>
  
  <button onclick="document.querySelector('manual-button').mount()">
    Mount
  </button>
  <button onclick="document.querySelector('manual-button').unmount()">
    Unmount
  </button>
</div>
```

```js inject show
JQRE.defineElement('manual-button', {
  state: {
    mounted: true
  },
  template: `
    <div>
      <p><strong>Status:</strong> Mounted ✅</p>
    </div>
  `
}, {
  mount: 'manual'  // Requires explicit mount()
});
```

```js cleanup
onUnmount(() => {
  $('#demo-lifecycle-manual').component('destroy');
});
```

**Use Cases:**
- Lazy initialization
- Deferred loading
- Conditional mounting based on app state

### Mount Return Value

```js
const el = document.querySelector('my-button');

const instance = el.mount();
// Returns: JqrComponent | undefined

if (instance) {
  console.log('Mounted successfully');
  console.log(instance.$props);
} else {
  console.log('Already mounted or mounting async');
}
```

## External Ownership

Parent components or `h()` own the lifecycle:

### Parent Component

```html live:no-code
<div id="demo-lifecycle-external">
  <div class="en-card">
    <div $:component:state="{ show: true }">
      <button class="en-btn en-btn--sm" $:click="show = !show">
        Toggle Child
      </button>
      <child-element $:if="show" style="display: block; margin-top: 10px"></child-element>
    </div>
  </div>
</div>
```

```html
<div id="demo-lifecycle-external">
  <div $:component:state="{ show: true }">
    <button $:click="show = !show">Toggle</button>
    
    <!-- Parent controls lifecycle -->
    <child-element $:if="show"></child-element>
  </div>
</div>
```

```js inject show
JQRE.defineElement('child-element', {
  state: {
    message: 'Controlled by parent'
  },
  mounted() {
    console.log('Child mounted by parent');
  },
  destroyed() {
    console.log('Child destroyed by parent');
  },
  template: `
    <div class="en-alert en-alert--info">
      <p $:text="message"></p>
    </div>
  `
});
```

```js cleanup
onUnmount(() => {
  $('#demo-lifecycle-external').component('destroy');
});
```

**Characteristics:**
- Parent creates and destroys component
- Element doesn't control its own lifecycle
- No automatic cleanup on disconnect
- `$mounted` still works

### With `h()`

```js
import { h, render } from '@en32/jqr-h';

JQRE.defineElement('counter-btn', counterDef);

// h() owns the lifecycle
render((h) => 
  h('counter-btn', {
    attrs: { count: 5 }
  })
, '#app');
```

## Component Lifecycle Hooks

Standard JQR lifecycle hooks work in custom elements:

### Lifecycle Order

```js
JQRE.defineElement('lifecycle-demo', {
  created() {
    // 1. Component instance created
    // State initialized, computed defined
    console.log('created');
  },
  
  mounted() {
    // 2. Component mounted to DOM
    // DOM is ready, effects running
    console.log('mounted');
  },
  
  updated() {
    // 3. Component updated (state/props changed)
    console.log('updated');
  },
  
  destroyed() {
    // 4. Component destroyed
    // Cleanup effects, remove listeners
    console.log('destroyed');
  }
});
```

**Order Example:**

```html live:no-code
<div id="demo-lifecycle-hooks">
  <div class="en-card">
    <hooks-demo></hooks-demo>
  </div>
</div>
```

```html
<div id="demo-lifecycle-hooks">
  <hooks-demo></hooks-demo>
</div>
```

```js inject show
JQRE.defineElement('hooks-demo', {
  state: {
    events: []
  },
  created() {
    this.events.push('1️⃣ created');
  },
  mounted() {
    this.events.push('2️⃣ mounted');
  },
  updated() {
    this.events.push('3️⃣ updated');
  },
  methods: {
    triggerUpdate() {
      this.events.push('⚡ Action triggered');
    }
  },
  template: `
    <div>
      <button class="en-btn en-btn--sm" $:click="triggerUpdate">Trigger Update</button>
      <ul style="margin-top: 10px">
        <li $:for="event in events" $:key="event" $:text="event"></li>
      </ul>
    </div>
  `
});
```

```js cleanup
onUnmount(() => {
  $('#demo-lifecycle-hooks').component('destroy');
});
```

## Slot Content Handling

Slot content is collected **before mounting**:

### Why Early Collection?

```html
<user-card>
  <span slot="header">John Doe</span>
  <p>Software Engineer</p>
</user-card>
```

**Problem:** If component mounts immediately, its template replaces light DOM, losing slot content.

**Solution:** Collect slots first, then mount.

### Collection Process

```
1. connectedCallback fires
2. Slot content collected into SlotGroups
3. Element added to mount queue
4. Component mounts with collected slots
5. fillSlotOutlets() places slot content
```

### Inline Components

Elements without template use their own markup:

```js
JQRE.defineElement('inline-wrapper', {
  // No template, no render()
  state: {
    active: false
  }
  // Element's children stay as-is
});
```

```html
<inline-wrapper>
  <!-- This markup is the template -->
  <div $:class="active ? 'active' : ''">
    Content
  </div>
</inline-wrapper>
```

## keepAlive Option

Keep components alive when disconnected:

```html live:no-code
<div id="demo-lifecycle-keepalive">
  <div class="en-card">
    <button class="en-btn en-btn--sm" onclick="
      const container = document.getElementById('demo-lifecycle-keepalive').querySelector('.demo-container');
      const el = container.querySelector('keepalive-counter');
      if (el) {
        el.remove();
        setTimeout(() => container.appendChild(el), 1000);
      }
    ">Remove & Re-add (1s)</button>
    <div class="demo-container" style="margin-top: 10px">
      <keepalive-counter></keepalive-counter>
    </div>
  </div>
</div>
```

```html
<div id="demo-lifecycle-keepalive">
  <button onclick="
    const el = document.querySelector('keepalive-counter');
    el.remove();
    setTimeout(() => document.body.appendChild(el), 1000);
  ">Remove & Re-add</button>
  
  <keepalive-counter></keepalive-counter>
</div>
```

```js inject show
JQRE.defineElement('keepalive-counter', {
  state: {
    count: 0,
    reconnects: 0
  },
  methods: {
    increment() {
      this.count++;
    }
  },
  mounted() {
    this.reconnects++;
  },
  template: `
    <div>
      <p>Count: <strong $:text="count"></strong></p>
      <p>Reconnects: <strong $:text="reconnects"></strong></p>
      <button class="en-btn en-btn--sm" $:click="increment">+1</button>
    </div>
  `
}, {
  keepAlive: true  // Survives disconnect/reconnect
});
```

```js cleanup
onUnmount(() => {
  $('#demo-lifecycle-keepalive').component('destroy');
});
```

**Behavior:**
- `keepAlive: false` (default) → destroyed on disconnect
- `keepAlive: true` → survives disconnect, component persists
- State preserved across disconnect/reconnect
- Manual `unmount()` required to destroy

**Use Cases:**
- Temporary hide/show (display: none)
- Moving elements in DOM
- SPA route transitions
- Preserving form state

## Async Components

Elements can mount before async component loads:

### Loading Flow

```js
// Define async component
JQR.registerAsync('lazy-widget', {
  load: () => import('./widget.js').then(m => m.default),
  props: { title: '' }  // Props must be declared up front
});

// Define element (before load)
JQRE.defineElement('lazy-widget');
```

```html
<!-- Element exists, but component loading -->
<lazy-widget title="Dashboard"></lazy-widget>
```

**Flow:**
1. Element connected → queued
2. Queue processed → `mountStandalone()` called
3. Async definition loading → `host.mountAsync()` called
4. Loading placeholder shown (if configured)
5. Definition loads → component mounts
6. `$mounted` resolves with instance

### Handling Load Failures

```js
const el = document.querySelector('lazy-widget');

try {
  const instance = await el.$mounted;
  console.log('Loaded:', instance);
} catch (error) {
  console.error('Failed to load:', error);
  // Error placeholder shown in DOM
}
```

**Error Handling:**
- Failed loads reject `$mounted` promise
- Error placeholder rendered (if configured)
- Element remains in DOM
- Can attempt remount with `unmount()` + `mount()`

## Unmounting

Destroy components explicitly or automatically:

### Automatic Unmount

Standalone components without `keepAlive`:

```js
const el = document.querySelector('my-button');
el.remove();  // Automatically destroyed after microtask
```

**Timing:**
- `disconnectedCallback` called immediately
- Destruction deferred to microtask
- Allows same-tick move operations

### Manual Unmount

```js
const el = document.querySelector('my-button');
el.unmount();  // Destroy immediately
```

**Effect:**
- Component destroyed
- State cleared
- Effects stopped
- Original light DOM restored
- `$mounted` promise reset

### Remounting

After unmount, element can mount again:

```html live:no-code
<div id="demo-lifecycle-remount">
  <div class="en-card">
    <div class="en-flex en-gap-2">
      <button class="en-btn en-btn--sm" onclick="document.querySelector('#demo-lifecycle-remount remount-demo').unmount()">
        Unmount
      </button>
      <button class="en-btn en-btn--sm" onclick="document.querySelector('#demo-lifecycle-remount remount-demo').mount()">
        Remount
      </button>
    </div>
    <remount-demo style="display: block; margin-top: 10px"></remount-demo>
  </div>
</div>
```

```html
<div id="demo-lifecycle-remount">
  <button onclick="document.querySelector('remount-demo').unmount()">
    Unmount
  </button>
  <button onclick="document.querySelector('remount-demo').mount()">
    Remount
  </button>
  
  <remount-demo></remount-demo>
</div>
```

```js inject show
JQRE.defineElement('remount-demo', {
  state: {
    mountCount: 0
  },
  mounted() {
    this.mountCount++;
  },
  template: `
    <div class="en-alert en-alert--success">
      <p>Mounted <strong $:text="mountCount"></strong> time(s)</p>
    </div>
  `
});
```

```js cleanup
onUnmount(() => {
  $('#demo-lifecycle-remount').component('destroy');
});
```

**Remount Behavior:**
- Original slot content restored
- State reset to initial values
- Props reread from attributes
- New component instance created

## Ownership Decision Algorithm

The 5-step algorithm for ownership:

### Step 1: Scan Declarative Roots

```js
jqr.scan();  // Mark all $:component elements
```

### Step 2: Sort by Depth

```js
// Outermost elements processed first
elements.sort((a, b) => depth(a) - depth(b));
```

### Step 3: Check Component Root

```js
if (isComponentRoot(el)) {
  adopt(el, info);  // External ownership
  continue;
}
```

### Step 4: Check Ancestor

```js
if (hasComponentAncestor(el)) {
  waiting.push(el);  // Wait for parent
  continue;
}
```

### Step 5: Mount Standalone

```js
mountStandalone(el, info);
```

**Example:**

```html
<div $:component="parent">
  <child-one></child-one>         <!-- Step 4: Wait -->
  <child-two $:component></child-two>  <!-- Step 1: Root -->
  <child-three></child-three>     <!-- Step 4: Wait -->
</div>
<child-four></child-four>         <!-- Step 5: Standalone -->
```

**Processing:**
1. `child-two` → Root (step 1)
2. `child-four` → Standalone (step 5)
3. `child-one`, `child-three` → Wait (step 4)
4. Parent mounts → claims `child-one`, `child-three`

## Best Practices

### ✅ Do

- **Use `keepAlive` sparingly** - only for expensive state
- **Declare props for async components** - required for early definition
- **Let parents control children** - don't force standalone mounting
- **Handle load failures** - catch `$mounted` rejections
- **Cleanup in `destroyed()`** - timers, listeners, observers

### ❌ Don't

- **Don't mount in connectedCallback** - use queue system
- **Don't assume instant mounting** - async components load later
- **Don't mutate during unmount** - component is being destroyed
- **Don't rely on disconnect order** - cleanup may be batched
- **Don't forget `unmount()`** - required for `keepAlive` elements

## Common Patterns

### Deferred Initialization

```js
JQRE.defineElement('heavy-widget', definition, {
  mount: 'manual'
});

// Mount when needed
const observer = new IntersectionObserver((entries) => {
  entries.forEach(entry => {
    if (entry.isIntersecting) {
      entry.target.mount();
      observer.unobserve(entry.target);
    }
  });
});

document.querySelectorAll('heavy-widget').forEach(el => {
  observer.observe(el);
});
```

### Conditional Remounting

```js
const el = document.querySelector('my-widget');

// Reset to clean state
el.unmount();
await el.$mounted;  // Wait for previous cleanup
el.mount();         // Fresh instance
```

### State Persistence

```js
JQRE.defineElement('stateful-widget', {
  state: {
    data: []
  },
  created() {
    // Restore from storage
    const saved = localStorage.getItem('widget-data');
    if (saved) {
      this.data = JSON.parse(saved);
    }
  },
  destroyed() {
    // Persist to storage
    localStorage.setItem('widget-data', JSON.stringify(this.data));
  }
}, {
  keepAlive: true  // Survives DOM changes
});
```

## Debugging

### Check Ownership Mode

```js
const el = document.querySelector('my-button');

// Access internal state (dev only)
console.log(el.$instance);  // Component instance or undefined
console.log(el.$mounted);   // Promise<Component>
```

### Monitor Lifecycle

```js
JQRE.defineElement('debug-element', {
  created() {
    console.log('created', this);
  },
  mounted() {
    console.log('mounted', this.$el);
  },
  updated() {
    console.log('updated');
  },
  destroyed() {
    console.log('destroyed');
  }
});
```

### Inspect Queue

```js
// Check if element is queued (internal API)
// Not exposed publicly, use $mounted instead
await el.$mounted;
console.log('Element mounted and ready');
```

## Next Steps

- **[Element API](./element-api.md)** - Access `$instance`, `$mounted`, `mount()`, `unmount()`
- **[Element Options](./options.md)** - Configure mounting behavior
- **[Defining Elements](./defining-elements.md)** - Review element creation

## Related

- [Component Lifecycle (JQR Core)](../../jqr/guides/lifecycle.md)
- [Async Components](../../jqr/guides/async-components.md)
- [API Reference](../reference/api-reference.md)
