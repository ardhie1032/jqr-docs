<!-- {
  "path": "jqr-e/guides/element-api",
  "name": "docs-jqr-e-guides-element-api",
  "meta": {
    "title": "Element API",
    "order": 6
  }
} -->

# Element API

Learn how to interact with custom elements from JavaScript using the Element API.

## Overview

Custom elements created by `defineElement()` expose four main APIs:

| API | Type | Description |
|-----|------|-------------|
| `$instance` | Property (readonly) | The mounted component instance |
| `$mounted` | Property (readonly) | Promise that resolves when mounted |
| `mount()` | Method | Mount the element manually |
| `unmount()` | Method | Unmount and destroy the component |

These APIs provide a bridge between the DOM element and its JQR component.

## `$instance` Property

Access the mounted component instance directly.

### Type Signature

```typescript
element.$instance: JqrComponent | undefined
```

- Returns `JqrComponent` if mounted
- Returns `undefined` if not mounted yet or destroyed

### Basic Usage

```html live:no-code
<div id="demo-api-instance">
  <div class="en-card">
    <counter-widget></counter-widget>
    <div class="en-flex en-gap-2" style="margin-top: 10px">
      <button class="en-btn en-btn--sm" onclick="
        const el = document.querySelector('#demo-api-instance counter-widget');
        console.log('Instance:', el.$instance);
        if (el.$instance) {
          console.log('Props:', el.$instance.$props);
          console.log('State:', el.$instance.$state);
        }
      ">Log Instance</button>
    </div>
  </div>
</div>
```

```html
<div id="demo-api-instance">
  <counter-widget></counter-widget>
  
  <button onclick="
    const el = document.querySelector('counter-widget');
    console.log(el.$instance);
  ">Log Instance</button>
</div>
```

```js inject show
JQRE.defineElement('counter-widget', {
  state: {
    count: 0
  },
  methods: {
    increment() {
      this.count++;
    }
  },
  template: `
    <div>
      <p>Count: <strong $:text="count"></strong></p>
      <button class="en-btn en-btn--sm" $:click="increment">+1</button>
    </div>
  `
});
```

```js cleanup
onUnmount(() => {
  $('#demo-api-instance').component('destroy');
});
```

### Accessing Component APIs

Once you have the instance, access all component APIs:

```js
const el = document.querySelector('my-widget');
const instance = el.$instance;

if (instance) {
  // Read state
  console.log(instance.$state);
  console.log(instance.$props);
  
  // Access computed
  console.log(instance.$computed);
  
  // Call methods
  instance.$ctx.myMethod();
  
  // Set props
  instance.$setProps({ value: 100 });
  
  // Emit events
  instance.$emit('change', newValue);
  
  // Access element
  console.log(instance.$el);  // The custom element
}
```

### Before Mount

Before mounting, `$instance` is `undefined`:

```js
const el = document.createElement('my-widget');
console.log(el.$instance);  // undefined (not mounted yet)

document.body.appendChild(el);
// Still undefined immediately after appendChild

await el.$mounted;
console.log(el.$instance);  // JqrComponent ✅
```

### After Unmount

After unmounting, `$instance` returns to `undefined`:

```js
const el = document.querySelector('my-widget');
console.log(el.$instance);  // JqrComponent

el.unmount();
console.log(el.$instance);  // undefined (destroyed)
```

### Use Cases

**Direct State Manipulation:**

```html live:no-code
<div id="demo-api-instance-state">
  <div class="en-card">
    <state-demo></state-demo>
    <div class="en-flex en-gap-2" style="margin-top: 10px">
      <button class="en-btn en-btn--sm" onclick="
        const el = document.querySelector('#demo-api-instance-state state-demo');
        if (el.$instance) {
          el.$instance.$state.count = 100;
        }
      ">Set to 100</button>
      <button class="en-btn en-btn--sm" onclick="
        const el = document.querySelector('#demo-api-instance-state state-demo');
        if (el.$instance) {
          el.$instance.$state.count = 0;
        }
      ">Reset</button>
    </div>
  </div>
</div>
```

```html
<div id="demo-api-instance-state">
  <state-demo></state-demo>
  
  <button onclick="
    const el = document.querySelector('state-demo');
    el.$instance.$state.count = 100;
  ">Set to 100</button>
</div>
```

```js inject show
JQRE.defineElement('state-demo', {
  state: {
    count: 0
  },
  template: `
    <div>
      <p>Count: <strong $:text="count"></strong></p>
    </div>
  `
});
```

```js cleanup
onUnmount(() => {
  $('#demo-api-instance-state').component('destroy');
});
```

**Calling Internal Methods:**

```js
const el = document.querySelector('form-widget');
const instance = el.$instance;

if (instance) {
  // Call validation method
  const isValid = instance.$ctx.validate();
  
  // Trigger submit
  instance.$ctx.submit();
  
  // Reset form
  instance.$ctx.reset();
}
```

**Reading Computed Values:**

```js
const el = document.querySelector('price-calculator');
const instance = el.$instance;

if (instance) {
  console.log('Total:', instance.$computed.total);
  console.log('Tax:', instance.$computed.tax);
  console.log('Grand Total:', instance.$computed.grandTotal);
}
```

## `$mounted` Promise

Wait for component to finish mounting.

### Type Signature

```typescript
element.$mounted: Promise<JqrComponent>
```

- Resolves with component instance when mounted
- For async components, resolves after definition loads
- Rejects if mounting fails
- Resolves immediately if already mounted

### Basic Usage

```js
const el = document.querySelector('my-widget');

// Wait for mount
const instance = await el.$mounted;
console.log('Mounted:', instance);

// Now safe to access APIs
console.log(instance.$state);
instance.$ctx.initialize();
```

### Async Components

For async components, `$mounted` waits for the definition to load:

```html live:no-code
<div id="demo-api-mounted-async">
  <div class="en-card">
    <p>Check console for loading sequence</p>
  </div>
</div>
```

```html
<div id="demo-api-mounted-async">
  <async-widget></async-widget>
</div>
```

```js inject show
// Define async component
$.jqr.registerAsync('async-widget', {
  load: async () => {
    console.log('1. Loading definition...');
    await new Promise(resolve => setTimeout(resolve, 1000));
    console.log('2. Definition loaded');
    return {
      state: { loaded: true },
      template: `<div>✅ Async component loaded!</div>`
    };
  },
  props: { title: '' }
});

JQRE.defineElement('async-widget');

// Wait for mount
const el = document.createElement('async-widget');
document.getElementById('demo-api-mounted-async').querySelector('.en-card').appendChild(el);

el.$mounted.then(instance => {
  console.log('3. Component mounted:', instance);
});
```

```js cleanup
onUnmount(() => {
  $('#demo-api-mounted-async').component('destroy');
});
```

**Timeline:**
1. Element connected → queued
2. `$mounted` promise created
3. Definition loads (async)
4. Component mounts
5. `$mounted` resolves

### Error Handling

Handle mounting failures gracefully:

```js
const el = document.querySelector('my-widget');

try {
  const instance = await el.$mounted;
  console.log('Success:', instance);
} catch (error) {
  console.error('Failed to mount:', error);
  // Show fallback UI or retry
}
```

**Common Errors:**
- Definition failed to load (async)
- Template compilation error
- Invalid component definition
- Render function threw error

### Multiple Waits

`$mounted` can be awaited multiple times:

```js
const el = document.querySelector('my-widget');

// Wait in multiple places
const instance1 = await el.$mounted;
const instance2 = await el.$mounted;

console.log(instance1 === instance2);  // true (same instance)
```

**After Unmount:**

```js
const el = document.querySelector('my-widget');

const instance1 = await el.$mounted;
console.log('Mounted:', instance1);

el.unmount();

// New promise for next mount
el.mount();
const instance2 = await el.$mounted;
console.log('Remounted:', instance2);

console.log(instance1 === instance2);  // false (new instance)
```

### Use Cases

**Initialize After Mount:**

```js
const el = document.querySelector('chart-widget');

await el.$mounted;

// Now safe to initialize chart
const instance = el.$instance;
instance.$ctx.renderChart(data);
```

**Synchronization:**

```js
// Wait for multiple widgets
const widgets = document.querySelectorAll('data-widget');
const instances = await Promise.all(
  Array.from(widgets).map(el => el.$mounted)
);

console.log('All widgets mounted:', instances);
```

**Conditional Logic:**

```js
const el = document.querySelector('lazy-widget');

// Set loading state
showSpinner();

try {
  await el.$mounted;
  hideSpinner();
  // Proceed with logic
} catch (error) {
  showError('Failed to load widget');
}
```

## `mount()` Method

Manually mount the element's component.

### Type Signature

```typescript
element.mount(): JqrComponent | undefined
```

- Returns `JqrComponent` if mounted successfully
- Returns `undefined` if already mounted or loading async
- Never throws (errors are logged)

### Basic Usage

```html live:no-code
<div id="demo-api-mount">
  <div class="en-card">
    <div class="en-flex en-gap-2">
      <button class="en-btn en-btn--sm" onclick="
        const el = document.querySelector('#demo-api-mount manual-mount');
        const instance = el.mount();
        console.log('Mount returned:', instance);
      ">Mount</button>
      <button class="en-btn en-btn--sm" onclick="
        const el = document.querySelector('#demo-api-mount manual-mount');
        el.unmount();
      ">Unmount</button>
    </div>
    <manual-mount style="display: block; margin-top: 10px"></manual-mount>
  </div>
</div>
```

```html
<div id="demo-api-mount">
  <manual-mount></manual-mount>
  
  <button onclick="document.querySelector('manual-mount').mount()">
    Mount
  </button>
  <button onclick="document.querySelector('manual-mount').unmount()">
    Unmount
  </button>
</div>
```

```js inject show
JQRE.defineElement('manual-mount', {
  state: {
    mountTime: null
  },
  mounted() {
    this.mountTime = new Date().toLocaleTimeString();
  },
  template: `
    <div class="en-alert en-alert--success">
      <p>Mounted at: <strong $:text="mountTime || 'Not mounted'"></strong></p>
    </div>
  `
}, {
  mount: 'manual'  // Requires explicit mount()
});
```

```js cleanup
onUnmount(() => {
  $('#demo-api-mount').component('destroy');
});
```

### Manual Mount Strategy

Required when `mount: 'manual'` is set:

```js
JQRE.defineElement('lazy-panel', definition, {
  mount: 'manual'
});
```

```html
<lazy-panel></lazy-panel>  <!-- Not mounted automatically -->
```

```js
// Must call mount() explicitly
const el = document.querySelector('lazy-panel');
el.mount();
```

### Idempotency

Calling `mount()` multiple times is safe:

```js
const el = document.querySelector('my-widget');

el.mount();  // Mounts
el.mount();  // No-op (already mounted)
el.mount();  // No-op (still mounted)
```

### Lazy Loading Pattern

```html live:no-code
<div id="demo-api-mount-lazy">
  <div class="en-card">
    <button class="en-btn en-btn--sm" onclick="
      const el = document.querySelector('#demo-api-mount-lazy lazy-load');
      el.mount();
    ">Load Widget</button>
    <lazy-load style="display: block; margin-top: 10px"></lazy-load>
  </div>
</div>
```

```html
<div id="demo-api-mount-lazy">
  <button onclick="document.querySelector('lazy-load').mount()">
    Load Widget
  </button>
  
  <lazy-load></lazy-load>
</div>
```

```js inject show
JQRE.defineElement('lazy-load', {
  state: {
    loading: false,
    data: null
  },
  async mounted() {
    this.loading = true;
    await new Promise(resolve => setTimeout(resolve, 500));
    this.data = { items: ['Item 1', 'Item 2', 'Item 3'] };
    this.loading = false;
  },
  template: `
    <div>
      <div $:if="loading">Loading...</div>
      <ul $:if="data && !loading">
        <li $:for="item in data.items" $:key="item" $:text="item"></li>
      </ul>
    </div>
  `
}, {
  mount: 'manual'
});
```

```js cleanup
onUnmount(() => {
  $('#demo-api-mount-lazy').component('destroy');
});
```

### Intersection Observer Pattern

Mount when element becomes visible:

```js
const observer = new IntersectionObserver((entries) => {
  entries.forEach(entry => {
    if (entry.isIntersecting) {
      const el = entry.target;
      el.mount();
      observer.unobserve(el);
    }
  });
});

document.querySelectorAll('heavy-widget').forEach(el => {
  observer.observe(el);
});
```

### Use Cases

**On-Demand Initialization:**

```js
// Mount when user clicks expand
expandButton.addEventListener('click', () => {
  const panel = document.querySelector('details-panel');
  panel.mount();
});
```

**Conditional Mounting:**

```js
if (userHasPermission) {
  document.querySelector('admin-panel').mount();
}
```

**Deferred Loading:**

```js
// Mount after critical content loads
window.addEventListener('load', () => {
  setTimeout(() => {
    document.querySelectorAll('[data-defer]').forEach(el => {
      el.mount();
    });
  }, 1000);
});
```

## `unmount()` Method

Destroy the component and clean up resources.

### Type Signature

```typescript
element.unmount(): void
```

- Destroys the mounted component
- Restores original slot content
- Clears state and effects
- Safe to call multiple times

### Basic Usage

```js
const el = document.querySelector('my-widget');
el.unmount();

console.log(el.$instance);  // undefined (destroyed)
```

### Effect

```html live:no-code
<div id="demo-api-unmount">
  <div class="en-card">
    <div class="en-flex en-gap-2">
      <button class="en-btn en-btn--sm" onclick="
        const el = document.querySelector('#demo-api-unmount unmount-demo');
        el.unmount();
      ">Unmount</button>
      <button class="en-btn en-btn--sm" onclick="
        const el = document.querySelector('#demo-api-unmount unmount-demo');
        el.mount();
      ">Remount</button>
    </div>
    <unmount-demo style="display: block; margin-top: 10px">
      <p>Original slot content (restored on unmount)</p>
    </unmount-demo>
  </div>
</div>
```

```html
<div id="demo-api-unmount">
  <unmount-demo>
    <p>Original slot content</p>
  </unmount-demo>
  
  <button onclick="document.querySelector('unmount-demo').unmount()">
    Unmount
  </button>
  <button onclick="document.querySelector('unmount-demo').mount()">
    Remount
  </button>
</div>
```

```js inject show
JQRE.defineElement('unmount-demo', {
  state: {
    timestamp: null
  },
  mounted() {
    this.timestamp = new Date().toLocaleTimeString();
  },
  template: `
    <div class="en-alert en-alert--info">
      <p>Component mounted at: <strong $:text="timestamp"></strong></p>
      <slot></slot>
    </div>
  `
});
```

```js cleanup
onUnmount(() => {
  $('#demo-api-unmount').component('destroy');
});
```

**What Happens:**
1. Component's `destroyed()` hook called
2. All effects and watchers stopped
3. Event listeners removed
4. DOM replaced with original slot content
5. `$instance` becomes `undefined`
6. `$mounted` promise reset

### Idempotency

Calling `unmount()` multiple times is safe:

```js
const el = document.querySelector('my-widget');

el.unmount();  // Destroys component
el.unmount();  // No-op (already unmounted)
el.unmount();  // No-op (still unmounted)
```

### Cleanup Pattern

```js
const el = document.querySelector('my-widget');

// Cleanup before removal
el.unmount();
el.remove();
```

**For `keepAlive` Elements:**

```js
JQRE.defineElement('persistent-widget', definition, {
  keepAlive: true
});

const el = document.querySelector('persistent-widget');

// Without unmount: stays alive when disconnected
el.remove();  // Component still active

// With unmount: explicit cleanup
el.unmount();  // Component destroyed
el.remove();   // Element removed
```

### Reset Pattern

```html live:no-code
<div id="demo-api-unmount-reset">
  <div class="en-card">
    <div class="en-flex en-gap-2">
      <button class="en-btn en-btn--sm" onclick="
        const el = document.querySelector('#demo-api-unmount-reset reset-demo');
        el.$instance.$state.count++;
      ">Increment</button>
      <button class="en-btn en-btn--sm" onclick="
        const el = document.querySelector('#demo-api-unmount-reset reset-demo');
        el.unmount();
        el.mount();
      ">Reset (Remount)</button>
    </div>
    <reset-demo style="display: block; margin-top: 10px"></reset-demo>
  </div>
</div>
```

```html
<div id="demo-api-unmount-reset">
  <reset-demo></reset-demo>
  
  <button onclick="document.querySelector('reset-demo').$instance.$state.count++">
    Increment
  </button>
  
  <button onclick="
    const el = document.querySelector('reset-demo');
    el.unmount();
    el.mount();
  ">Reset</button>
</div>
```

```js inject show
JQRE.defineElement('reset-demo', {
  state: {
    count: 0
  },
  template: `
    <div>
      <p>Count: <strong $:text="count"></strong></p>
    </div>
  `
});
```

```js cleanup
onUnmount(() => {
  $('#demo-api-unmount-reset').component('destroy');
});
```

### Use Cases

**Manual Cleanup:**

```js
// Before navigating away
beforeUnload(() => {
  document.querySelectorAll('data-widget').forEach(el => {
    el.unmount();
  });
});
```

**Conditional Rendering:**

```js
// Show/hide with clean state
function toggleWidget(show) {
  const el = document.querySelector('my-widget');
  
  if (show) {
    el.mount();
  } else {
    el.unmount();
  }
}
```

**Memory Management:**

```js
// Free resources for off-screen widgets
const observer = new IntersectionObserver((entries) => {
  entries.forEach(entry => {
    const el = entry.target;
    
    if (!entry.isIntersecting) {
      el.unmount();  // Free memory
    } else {
      el.mount();    // Remount when visible
    }
  });
});
```

## Combining APIs

Use all APIs together for powerful control:

### Wait Then Access

```js
const el = document.querySelector('my-widget');

// Wait for mount
await el.$mounted;

// Access instance
const instance = el.$instance;
console.log(instance.$state);

// Manipulate
instance.$state.value = 100;

// Later: cleanup
el.unmount();
```

### Conditional Mount with Timeout

```js
const el = document.querySelector('lazy-widget');

// Try to mount
el.mount();

// Wait with timeout
const timeout = setTimeout(() => {
  console.warn('Mount timeout');
  el.unmount();
}, 5000);

try {
  await el.$mounted;
  clearTimeout(timeout);
  console.log('Mounted successfully');
} catch (error) {
  console.error('Mount failed:', error);
}
```

### Batch Operations

```js
// Mount all widgets
const widgets = document.querySelectorAll('data-widget');
widgets.forEach(el => el.mount());

// Wait for all
const instances = await Promise.all(
  Array.from(widgets).map(el => el.$mounted)
);

// Initialize all
instances.forEach(instance => {
  instance.$ctx.initialize();
});

// Later: cleanup all
widgets.forEach(el => el.unmount());
```

## Best Practices

### ✅ Do

- **Check `$instance` before use** - may be undefined
- **Await `$mounted` for async operations** - ensures component is ready
- **Call `unmount()` for cleanup** - especially with `keepAlive`
- **Use `mount()` for lazy loading** - improve initial page load
- **Handle mount errors** - catch `$mounted` rejections

### ❌ Don't

- **Don't mutate `$instance` directly** - may break reactivity
- **Don't assume instant mounting** - always await `$mounted`
- **Don't forget error handling** - async components can fail
- **Don't call APIs on destroyed elements** - check `$instance` first
- **Don't leak references** - clear event listeners before unmount

## TypeScript Support

Full type safety for Element API:

```typescript
import type { JqrElement, JqrComponent } from '@en32/jqr/element';

const el = document.querySelector('my-widget') as JqrElement;

// TypeScript knows these exist
const instance: JqrComponent | undefined = el.$instance;
const mounted: Promise<JqrComponent> = el.$mounted;
el.mount();
el.unmount();
```

## Common Patterns

### Safe Method Call

```js
function callMethod(el, methodName, ...args) {
  const instance = el.$instance;
  
  if (!instance) {
    console.warn('Element not mounted');
    return;
  }
  
  const method = instance.$ctx[methodName];
  
  if (typeof method !== 'function') {
    console.warn(`Method ${methodName} not found`);
    return;
  }
  
  return method(...args);
}

// Usage
callMethod(el, 'validate');
callMethod(el, 'submit', formData);
```

### Initialization Guard

```js
async function initializeWidget(el) {
  try {
    const instance = await el.$mounted;
    
    // Safe to initialize
    instance.$ctx.fetchData();
    instance.$ctx.setupListeners();
    
    return instance;
  } catch (error) {
    console.error('Failed to initialize:', error);
    // Show fallback UI
    el.innerHTML = '<p>Failed to load widget</p>';
    return null;
  }
}
```

### Lifecycle Manager

```js
class WidgetManager {
  constructor() {
    this.widgets = new Map();
  }
  
  async register(el) {
    const instance = await el.$mounted;
    this.widgets.set(el, instance);
    return instance;
  }
  
  unregister(el) {
    if (this.widgets.has(el)) {
      el.unmount();
      this.widgets.delete(el);
    }
  }
  
  cleanup() {
    this.widgets.forEach((_, el) => {
      el.unmount();
    });
    this.widgets.clear();
  }
}
```

## Debugging

### Log Instance State

```js
const el = document.querySelector('my-widget');

if (el.$instance) {
  console.log('State:', el.$instance.$state);
  console.log('Props:', el.$instance.$props);
  console.log('Computed:', el.$instance.$computed);
} else {
  console.log('Not mounted');
}
```

### Monitor Mount Status

```js
const el = document.querySelector('my-widget');

console.log('Before mount:', el.$instance);

el.mount();

el.$mounted.then(instance => {
  console.log('After mount:', instance);
  console.log('Same as $instance?', instance === el.$instance);
});
```

### Inspect Element

```js
// In browser console
const el = $0;  // Selected element in DevTools

console.log({
  tagName: el.tagName,
  instance: el.$instance,
  isMounted: !!el.$instance,
  state: el.$instance?.$state,
  props: el.$instance?.$props
});
```

## Next Steps

- **[Lifecycle & Mounting](./lifecycle.md)** - Understand ownership and mounting
- **[Element Options](./options.md)** - Configure elements
- **[h() Integration](./h-integration.md)** - Use with render functions

## Related

- [API Reference](../reference/api-reference.md)
- [Instance Methods (JQR Core)](../../jqr/guides/instance-methods.md)
- [Component Options](../../jqr/guides/component-options.md)
