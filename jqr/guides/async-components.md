<!-- {
  "path": "jqr/guides/async-components",
  "name": "docs-jqr-guides-async-components",
  "meta": {
    "title": "Async Components",
    "order": 16
  }
} -->

# Async Components

Async components (also called lazy-loaded components) are components whose definitions are loaded on-demand, only when they're actually needed. This improves initial page load performance by splitting code into smaller chunks.

## Why Use Async Components?

**Performance Benefits:**
- Reduce initial bundle size
- Faster initial page load
- Load components only when needed
- Better code splitting

**Use Cases:**
- Heavy components (charts, rich editors)
- Modal dialogs and overlays
- Admin panels or rarely-used features
- Route-based code splitting

## Defining Async Components

### Basic Definition

Use `$.jqr.defineAsync()` to create an async component:

```js
// Define with dynamic import
const userCard = $.jqr.defineAsync(() => import('/components/user-card.js'));

// Or with URL shorthand
const reportViewer = $.jqr.defineAsync('/components/report-viewer.js');
```

### Registering Async Components

Register an async component globally by name:

```js
// With loader function
$.jqr.registerAsync('user-card', () => import('/components/user-card.js'));

// With URL shorthand
$.jqr.registerAsync('chart-widget', '/components/chart-widget.js');
```

**Note:** `$.jqr.register()` and `$.jqr.registerAsync()` share the same registry. Registering a component with the same name replaces the previous one, whether sync or async.

## The Loader Function

The loader can be:

1. **A function returning a Promise:**
```js
() => import('/components/my-component.js')
```

2. **A module URL string (shorthand):**
```js
'/components/my-component.js'  // equivalent to: () => import('/components/my-component.js')
```

### What the Loader Returns

The loader's resolved value is normalized to a component definition:

**1. ES Module (default export):**
```js
// /components/card-button.js
export default {
  props: { label: 'Press' },
  template: '<button $:text="$props.label"></button>'
};
```

**2. Plain Object:**
```js
() => Promise.resolve({
  template: '<div>Hello</div>'
})
```

**3. Classic Script (self-registering):**
```js
// /components/legacy-card.js
$.jqr.register('legacy-card', {
  template: '<b>Legacy Card</b>'
});
```
```js
// Loader returns undefined/null - component self-registered
$.jqr.registerAsync('legacy-card', () => $.jqr.loadScript('/components/legacy-card.js'));
```

**4. Invalid:** Anything else rejects with `AsyncLoadError` (kind: `'invalid'`).

## Using Async Components

### In Templates

Use async components exactly like regular components:

```html live:no-code
<div id="demo-async-basic">
  <div class="en-card">
    <h3 class="en-card__title">User Profile</h3>
    <div class="en-card__body">
      <div $:component="user-card" $:prop:user-id="userId"></div>
    </div>
  </div>
</div>
```

```html
<div id="demo-async-basic">
  <h3>User Profile</h3>
  <div $:component="user-card" $:prop:user-id="userId"></div>
</div>
```

```js
// Register async component: `/assets/examples/jqr/user-card.js`
JQR.register('user-card', {
  props: { userId: 1 },
  template: `
    <div class="user-card">
      <strong>User #<span $:text="$props.userId"></span></strong>
      <p>Profile loaded!</p>
    </div>
  `
});
```

```js inject show
// Load async component
JQR.registerAsync('user-card', () => JQR.loadScript('/assets/examples/jqr/user-card.js'));

$('#demo-async-basic').component({
  state: { userId: 42 }
});
```

```js cleanup
onUnmount(() => {
  $('#demo-async-basic').component('destroy');
})
```

### Inline Async Components

Use `$:component:async` to define the loader directly on an element:

```html
<!-- Loader function -->
<div $:component:async="() => import('/components/chart.js')" 
     $:prop:data="chartData">
</div>

<!-- Promise -->
<div $:component:async="fetchComponent()" 
     $:prop:config="config">
</div>

<!-- URL string -->
<div $:component:async="'/components/widget.js'">
</div>
```

**Shared Loading:** The same URL, function, or Promise reference creates ONE shared async component. Multiple hosts mean one load.

**Cannot Combine:** `$:component:async` cannot be used with `$:component="name"` or `$:component:is` (ignored with dev warning).

## Loading States

### While Loading

**Inside a Component Template:**
- Slot content is collected and bound to parent's scope
- Host marked with `data-jqr-async="pending"`
- Props evaluated when definition arrives (current values used)

**At Top Level (outside any component):**
- Authored markup stays visible
- Element counts as component root (nothing inside boots separately)
- Host marked with `data-jqr-async="pending"`

### After Loading

- Props evaluated with current values
- Component mounts
- `data-jqr-async` attribute removed

### If Host Removed Before Loading

- Nothing is mounted
- Placeholder removed (if any)
- Definition still cached for next time

## Placeholders

### Loading Placeholder

Show content while the component loads:

```js
$.jqr.registerAsync('heavy-chart', () => import('/components/chart.js'), {
  loading: '<div class="spinner">Loading chart...</div>',
  delay: 200  // Wait 200ms before showing (default)
});
```

**Delay Option:**
- Default: `200` ms
- `0` = show immediately
- Loads finishing before delay never show the placeholder

### Error Placeholder

Show content when loading fails:

```js
$.jqr.registerAsync('report', () => import('/components/report.js'), {
  error: '<div class="error">Failed to load report. <button onclick="location.reload()">Retry</button></div>',
  timeout: 10000  // Fail after 10 seconds
});
```

**Note:** JQR does NOT provide a default error placeholder. If you don't specify `error`, failed loads will:
- Mark host with `data-jqr-async="error"` (for CSS styling)
- Log error to `console.error`
- Show nothing visually

Always provide an `error` placeholder for production applications.

### Error Component with Retry

Pass a component definition to handle errors with retry logic:

```html live:no-code
<div id="demo-async-error">
  <div class="en-card">
    <div class="en-card__body">
      <div $:component="flaky-component"></div>
    </div>
  </div>
</div>
```

```html
<div id="demo-async-error">
  <div $:component="flaky-component"></div>
</div>
```

```js inject show
// Simulate flaky loading
let attemptCount = 0;
JQR.registerAsync('flaky-component', () => {
  attemptCount++;
  return new Promise((resolve, reject) => {
    setTimeout(() => {
      if (attemptCount < 2) {
        reject(new Error('Network error'));
      } else {
        resolve({
          template: '<div class="success">✓ Loaded successfully!</div>'
        });
      }
    }, 500);
  });
}, {
  loading: '<p>Loading...</p>',
  error: {
    props: ['error', 'retry'],
    template: `
      <div class="error">
        <p>❌ Loading failed: <code $:text="$props.error.message"></code></p>
        <button class="en-btn" $:click="$props.retry()">Retry</button>
      </div>
    `
  },
  delay: 100,
  timeout: 3000
});

$('#demo-async-error').component();
```

```js cleanup
onUnmount(() => {
  $('#demo-async-error').component('destroy');
})
```

**Error Component Props:**
- `error`: The `AsyncLoadError` object
- `retry`: Function to retry loading

**How Error Components Work:**

Error placeholders are **regular JQR components** mounted via `mountComponent()`. The `error` and `retry` props are passed just like any other component props:

```js
// Internally, JQR does this:
mountComponent(wrapper, errorComponentDef, {
  props: {
    error: asyncLoadError,      // The error object
    retry: () => retryLoad()    // Closure to retry
  }
});
```

This means:
- `retry` is a callback prop (like `onSave`, `onClose`, etc.)
- You access it via `$props.retry()` in templates
- You can pass any component definition as `error` option
- Props are NOT limited to serializable data—functions work fine

### Placeholder DOM Structure

Placeholders are rendered inside the host:

```html
<!-- While loading -->
<div $:component="user-card" data-jqr-async="pending">
  <div data-jqr-async-placeholder>
    <div class="spinner">Loading...</div>
  </div>
</div>

<!-- On error -->
<div $:component="report" data-jqr-async="error">
  <div data-jqr-async-placeholder>
    <div class="error">Failed to load</div>
  </div>
</div>

<!-- After successful mount -->
<div $:component="user-card">
  <!-- Actual component content -->
</div>
```

## Loading Options

### Timeout

Specify how long to wait before considering the load failed:

```js
$.jqr.registerAsync('slow-component', () => import('/components/slow.js'), {
  timeout: 5000  // 5 seconds
});
```

- Default: no timeout
- `0` = no timeout
- Failed loads trigger the `error` placeholder

### Configuration Precedence

Options merge from most specific to least specific:

1. **Mount-time options** (via `mountComponent()` or `$:component:<option>`)
2. **Component's own options** (in `defineAsync`/`registerAsync`)
3. **Global defaults** (via `$.jqr.config()`)

```js
// Set global defaults
$.jqr.config({
  async: {
    loading: '<span class="spinner"></span>',
    delay: 100
  }
});

// Component-specific (overrides global)
$.jqr.registerAsync('user-card', () => import('/components/user-card.js'), {
  delay: 300  // This component waits 300ms
});
```

Clear global defaults:

```js
$.jqr.config({ async: null });
```

## Load Management

### Manual Loading (Prefetching)

Load a component before it's needed:

```js
// By registered name
await $.jqr.loadComponent('user-card');

// By async component reference
const chart = $.jqr.defineAsync(() => import('/components/chart.js'));
await $.jqr.loadComponent(chart);

// Check if loaded
if (chart.isLoaded()) {
  console.log('Chart is ready!');
}
```

### Load Options

Control individual load calls:

```js
const controller = new AbortController();

await chart.load({
  timeout: 3000,           // Timeout for this call only
  signal: controller.signal  // Abort signal
});

// Cancel the load
controller.abort();
```

**Per-call timeout:** Rejects for this caller only. The load continues and is cached if it succeeds.

**Abort signal:** Cancels this caller's wait. When ALL waiters abort, the loader's signal is aborted too.

### Shared Loading

An async component loads **once**, shared by all callers:

- **Concurrent calls** get the same result
- **After successful load**: resolves immediately from cache
- **Failed loads NOT cached**: Returns to `idle`, next load tries again

```js
// Both calls share the same load
const promise1 = $.jqr.loadComponent('user-card');
const promise2 = $.jqr.loadComponent('user-card');

console.log(promise1 === promise2);  // false (different promises)
// But they wait for the same underlying load
```

## Error Handling

### AsyncLoadError

All async loading errors are `AsyncLoadError` instances:

```js
try {
  await $.jqr.loadComponent('broken-component');
} catch (err) {
  console.log(err.name);       // 'AsyncLoadError' or 'AbortError'
  console.log(err.kind);       // 'network', 'timeout', 'invalid', 'aborted'
  console.log(err.component);  // Component name (if registered)
  console.log(err.cause);      // Underlying error (if any)
}
```

**Error Kinds:**
- `'network'`: Loader rejected/threw (import failed, fetch error, missing chunk after deploy)
- `'timeout'`: Load exceeded timeout
- `'invalid'`: Loader result is not a valid component definition
- `'aborted'`: Load was cancelled via AbortSignal

**Error Logging:** Failed loads are always logged to `console.error`, with or without an `error` placeholder.

**No Synchronous Throws:** Errors are reported through the returned Promise. Invalid arguments raise immediately.

## Advanced: Load Then Mount

Once loaded, async components work like regular components:

```js
// Register async
$.jqr.registerAsync('card-button', () => import('/components/card-button.js'));

// Load it
await $.jqr.loadComponent('card-button');

// Now scan will mount it immediately (no loading wait)
$.jqr.scan();
```

This is useful for:
- Prefetching components
- Avoiding loading states when you know you'll need it
- Progressive enhancement patterns

## Integration with h()

Async components work seamlessly with `h()`:

```js
import { h, render } from '@en32/jqr-h';

// By registered name
h('user-card', { props: { userId: 42 } })

// By async component reference
const chart = $.jqr.defineAsync(() => import('/components/chart.js'));
h(chart, { props: { data: chartData } })

// Via components option
render((h) => h('chart-widget', { props: { data } }), {
  components: {
    'chart-widget': $.jqr.defineAsync('/components/chart.js')
  }
})
```

**Behavior:**
- Host element returned immediately
- Slot content built against parent (immediately)
- Component fills in when definition arrives
- Tree structure (`h.for`, `h.if`) unaffected by async loading

**Don't Pass Promises Directly:**
```js
// ❌ WRONG - warns and renders empty <div>
h(import('/components/chart.js'), ...)

// ✅ CORRECT - wrap with defineAsync
const chart = $.jqr.defineAsync(() => import('/components/chart.js'));
h(chart, ...)
```

## Advanced: Component Swapping

### Reactive Props Function

Pass a function to `props` for reactive prop updates:

```js
const handle = $.jqr.mountComponent(el, 'user-card', {
  props: () => ({
    userId: currentUserId,  // Tracked reactively
    theme: appTheme
  })
});
```

**Behavior:**
- Function re-runs when tracked state changes
- Props updated via `$setProps` (no remount)
- Props that disappear fall back to defaults
- Cannot mix with `handle.setProps()` (switches function off)

### swap() Method

Replace the mounted component with another:

```js
// Basic swap (immediate)
await handle.swap('other-card');

// Keep old component until new one is ready
await handle.swap(asyncCard, { keepUntilReady: true });
```

**Options:**

**1. Replace (default):**
- Current component destroyed immediately
- New component mounted (shows loading placeholder if async)

**2. keepUntilReady:**
- Current component stays mounted and visible
- New component loads in background
- Swap happens in one synchronous step when ready
- No placeholder, no gap
- If load fails, current component stays (promise rejects)

**Swapping Behavior:**
- Swapping to same ref (by reference): only updates props/on/slots, never remounts
- Only latest `swap()` wins: Previous pending swaps resolve immediately
- Promise never hangs: Destroying handle or superseding swap always settles it

```js
// Full options
await handle.swap('different-card', {
  keepUntilReady: true,
  props: { newId: 123 },
  on: { save: () => console.log('saved') },
  slots: { default: '<p>New slot content</p>' }
});
```

## Remounting and Reloading

### Understanding the Difference

**Important distinction:**
- **Remount** = Create new component instance (from cached definition)
- **Reload** = Fetch definition from network again
- **Props sync** = Update existing instance props (no remount)

Once an async component loads successfully, its definition is **cached permanently**. Remounting creates a new instance but uses the cached definition—the loader is NOT called again.

### Component Resolution is Static

```html
<div $:component="user-card" $:prop:user-id="userId"></div>
```

The component name `"user-card"` is resolved **once** when the element is first scanned. Changing props (`userId`) only syncs the prop value to the existing instance—it does NOT remount or reload.

### Method 1: Toggle with $:if

Force remount by hiding and showing the host element:

```html
<div id="demo-remount">
  <template $:if="showCard">
    <div $:component="user-card" $:prop:user-id="userId"></div>
  </template>
  <button $:click="reload()">Reload Component</button>
</div>
```

```js
$('#demo-remount').component({
  state: { userId: 42, showCard: true },
  methods: {
    async reload() {
      this.showCard = false;
      await this.$nextTick();  // Wait for DOM removal
      this.showCard = true;    // Creates new instance
    }
  }
});
```

### Method 2: Use $:for with $:key

Change the key to force remount without `nextTick()`:

```html
<div id="demo-remount">
  <template $:for="k in [reloadKey]" $:key="k">
    <div $:component="user-card" $:prop:user-id="userId"></div>
  </template>
  <button $:click="reloadKey++">Reload</button>
</div>
```

```js
$('#demo-remount').component({
  state: { userId: 42, reloadKey: 0 }
});
```

**How it works:** Different `$:key` value = different element = remount.

### Method 3: mountComponent() + swap()

For programmatic control, use `mountComponent()` API:

```js
const el = document.querySelector('#card-container');
const handle = $.jqr.mountComponent(el, 'user-card', {
  props: { userId: 42 }
});

// Remount (destroy old, create new)
await handle.swap('user-card');

// With options
await handle.swap('user-card', {
  props: { userId: 99 }
});
```

**Note:** `swap()` to the same component reference destroys and recreates the instance.

### Forcing Network Reload

All three methods above only **remount from cached definition**. To actually reload from network (e.g., for testing retry scenarios):

```js
methods: {
  async forceReload() {
    // Register with new URL (cache-bust)
    $.jqr.registerAsync('user-card', 
      () => import(`/components/user-card.js?v=${Date.now()}`)
    );
    
    // Then remount using one of the methods above
    this.showCard = false;
    await this.$nextTick();
    this.showCard = true;
  }
}
```

**Why query string?** Both `import()` and `$.jqr.loadScript()` cache by URL. Without cache-busting, they return the already-loaded module/script immediately.

### When Remounting is Needed

**Common scenarios:**
- User clicked "refresh" button
- Switching between different items (use `$:key` with item ID)
- Testing error→retry flows
- Dev mode: hot reload simulation

**Usually NOT needed:**
- Just updating props (use reactive binding)
- Refetching data (use `$refetch()` in `asyncState`)
- Changing state (already reactive)

### Error Placeholder Retry vs. Remount

The `retry()` function in error placeholders **retries the load**, not just remounting:

```js
error: {
  props: ['error', 'retry'],
  template: `
    <button $:click="$props.retry()">Retry</button>
  `
}
```

This works because:
1. Failed loads are NOT cached (component stays in `'idle'` state)
2. `retry()` calls the loader again
3. If it succeeds, the component mounts

This is different from remounting an already-loaded component.

## Best Practices

### When to Use Async Components

**Good Use Cases:**
```js
// Heavy, rarely-used features
$.jqr.registerAsync('video-editor', '/components/video-editor.js');

// Modal dialogs (loaded when opened)
$.jqr.registerAsync('settings-modal', '/components/settings-modal.js');

// Admin panels
$.jqr.registerAsync('admin-dashboard', '/components/admin-dashboard.js');

// Route-specific components
$.jqr.registerAsync('user-profile-page', '/pages/user-profile.js');
```

**Avoid for:**
- Small components (overhead > benefit)
- Critical above-the-fold content
- Frequently-used components

### Optimize Loading

**Prefetch on Hover:**
```js
$('[data-action="open-settings"]').on('mouseenter', () => {
  $.jqr.loadComponent('settings-modal');  // Start loading
});

$('[data-action="open-settings"]').on('click', async () => {
  // Already loading or loaded - fast!
  await $.jqr.loadComponent('settings-modal');
  $('#settings').component('settings-modal', { ... });
});
```

**Prefetch on Idle:**
```js
if ('requestIdleCallback' in window) {
  requestIdleCallback(() => {
    $.jqr.loadComponent('report-viewer');
  });
}
```

### Error Recovery

Always provide error placeholders with retry:

```js
$.jqr.registerAsync('critical-widget', loader, {
  error: {
    props: ['error', 'retry'],
    template: `
      <div class="error-state">
        <p>Failed to load widget</p>
        <button $:click="$props.retry()">Retry</button>
        <details>
          <summary>Error Details</summary>
          <code $:text="$props.error.message"></code>
        </details>
      </div>
    `
  },
  timeout: 10000
});
```

### Configure Global Defaults

Set reasonable defaults for all async components:

```js
$.jqr.config({
  async: {
    loading: '<div class="loading-spinner"></div>',
    delay: 150,
    timeout: 15000,
    error: {
      props: ['error', 'retry'],
      template: `
        <div class="async-error">
          <p>Failed to load component</p>
          <button $:click="$props.retry()">Retry</button>
        </div>
      `
    }
  }
});
```

## API Reference

### Functions

| Function | Description |
|----------|-------------|
| `$.jqr.defineAsync(loader, options?)` | Define an async component |
| `$.jqr.registerAsync(name, loader, options?)` | Register an async component globally |
| `$.jqr.loadComponent(ref)` | Load a component by name or async component reference |
| `$.jqr.loadScript(url, options?)` | Load external script (classic or module) |
| `component.$isLoaded()` | Check if async component has loaded |
| `component.$load(options?)` | Load async component with options |

### Options

| Option | Type | Default | Description |
|--------|------|---------|-------------|
| `loading` | `string \| ComponentDef` | - | Placeholder while loading |
| `error` | `string \| ComponentDef` | - | Placeholder on error |
| `delay` | `number` | `200` | ms before showing loading placeholder |
| `timeout` | `number` | `0` | ms before load is considered failed (0 = no timeout) |
| `props` | `string[]` | - | Prop names for custom elements (before component loads) |

### Directives

| Directive | Description |
|-----------|-------------|
| `$:component="name"` | Mount registered async component |
| `$:component:async="loader"` | Inline async component loader |
| `$:component:is="asyncRef"` | Mount async component by reference |

### Types (TypeScript)

```typescript
import {
  AsyncComponent,
  AsyncLoader,
  AsyncComponentOptions,
  AsyncLoadOptions,
  AsyncLoadContext,
  AsyncLoadError,
  AsyncLoadErrorKind,
  defineAsyncComponent,
  loadComponent,
  loadScript,
  LoadScriptOptions,
  isAsyncComponent,
  getRegisteredEntry
} from '@en32/jqr';
```

## loadScript() Utility

Helper function for loading external scripts (classic or module).

### Function Signature

```typescript
function loadScript(
  url: string,
  options?: LoadScriptOptions
): Promise<void>
```

### Options

```typescript
interface LoadScriptOptions {
  module?: boolean;      // Load as ES module (default: false)
  integrity?: string;    // Subresource Integrity hash
}
```

### Features

**Shared Loading:**
- Same URL = one shared load (concurrent calls wait for same script)
- Successful loads are cached (subsequent calls resolve immediately)
- **Failed loads NOT cached** - next call tries again (consistent with async components)

**Module Support:**
- `module: true` creates `<script type="module">`
- `module: false` (default) creates classic `<script>`

**Subresource Integrity (SRI):**
- When `integrity` is provided, automatically adds `crossorigin="anonymous"`
- Ensures script integrity for CDN-loaded scripts

### Examples

**Basic Classic Script:**

```js
// Load classic script
await $.jqr.loadScript('/lib/legacy-plugin.js');

// Use with async component
$.jqr.registerAsync('legacy-widget', async () => {
  await $.jqr.loadScript('/lib/widget-lib.js');
  // Script has registered itself globally
  return null;  // Component self-registered
});
```

**ES Module Script:**

```js
// Load ES module
await $.jqr.loadScript('/lib/modern-plugin.js', {
  module: true
});
```

**With Subresource Integrity:**

```js
// Load from CDN with SRI
await $.jqr.loadScript(
  'https://cdn.example.com/plugin.js',
  {
    integrity: 'sha384-oqVuAfXRKap7fdgcCY5uykM6+R9GqQ8K/ux...'
  }
);

// Automatically adds crossorigin="anonymous"
```

**Concurrent Loading:**

```js
// Both calls share the same load
const promise1 = $.jqr.loadScript('/lib/shared.js');
const promise2 = $.jqr.loadScript('/lib/shared.js');

await Promise.all([promise1, promise2]);
// Script loaded only once
```

**Error Handling:**

```js
try {
  await $.jqr.loadScript('/lib/missing.js');
} catch (err) {
  console.error('Script load failed:', err);
  // Failed load NOT cached - can retry
  await $.jqr.loadScript('/lib/missing.js');  // Tries again
}
```

**With Async Components:**

```js
// Self-registering component
$.jqr.registerAsync('chart-widget', async () => {
  // Load external charting library first
  await $.jqr.loadScript('https://cdn.example.com/charts.js', {
    integrity: 'sha384-...'
  });
  
  // Then load component definition that depends on it
  await $.jqr.loadScript('/components/chart-widget.js');
  
  // Component registered itself via $.jqr.register()
  return null;
});
```

### Error Codes

Failed script loads throw an error with code `'jqr.async.scriptLoadFailed'` (E081).

### Browser Support

- Classic scripts: All browsers
- Module scripts: Modern browsers with ES6 module support
- SRI: Modern browsers (gracefully degrades in older browsers)

## See Also

- [Nested Components](./nested-components.md) - Component composition patterns
- [Component Options](./component-options.md) - All component definition options
- [Performance Tips](./performance.md) - Optimization strategies
- [JQR-H Integration](../../jqr-h/guides/components-with-h.md) - Using async with h()
