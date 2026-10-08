<!-- {
  "path": "jqr/reference/config-reference",
  "name": "docs-jqr-reference-config-reference",
  "meta": {
    "title": "Configuration Reference",
    "order": 3
  }
} -->

# Configuration Reference

Complete API reference for JQR configuration and global utilities.

## API Namespaces

JQR exposes two global namespaces with different purposes:

### `$.jqr.*` — Public Configuration API

Stable public APIs for everyday application development:
- Configuration: `config()`, `registerPrefix()`
- Registration: `register()`, `directive()`, `plugin()`
- Utilities: `nextTick()`, `debounce()`, `throttle()`, `loadScript()`, `scan()`, `mountEach()`
- Checks: `isDevMode()`, `isUsingFallback()`
- Confirmation: `dispatchConfirm()`, `confirmGate()`

**Browser/IIFE:** `$.jqr.config({...})`  
**ESM:** `import { jqr } from '@en32/jqr'; jqr.config({...})`

### `JQR.*` — Internal Primitives

Low-level reactivity primitives primarily for `@en32/jqr-h` and advanced use cases:
- Reactivity: `reactive()`, `effect()`, `computed()`, `watch()`, `toRaw()`
- Internal: `Component`, `getRegisteredComponent()`, etc.

**Browser/IIFE:** `JQR.reactive({...})`  
**ESM:** `import { reactive } from '@en32/jqr'`

> Most application code uses `$.jqr.*` APIs only. The `JQR.*` primitives are documented here for completeness but are typically not needed for standard component development.

---

## `$.jqr.config(options)`

Configure JQR globally. Call before mounting components.

### Signature

```typescript
interface ConfigOptions {
  devMode?: boolean;
  defaultPrefix?: string;
  transitionEngine?: 'native' | 'jquery';
  sanitizer?: ((html: string) => string) | null;
}

$.jqr.config(options: ConfigOptions): void
```

### Parameters

#### `devMode`

**Type:** `boolean`  
**Default:** `false`

Enable development-only warnings:
- Unknown directive names
- `$:for` without `$:key`
- Conflicting `$:if` + `$:show`
- Invalid prop writes
- Other potential issues

**Example:**

```js
$.jqr.config({ devMode: true });
```

```js
// Environment-based
$.jqr.config({
  devMode: process.env.NODE_ENV !== 'production'
});
```

#### `defaultPrefix`

**Type:** `string`  
**Default:** `'$'`

Set the default directive prefix. Automatically registers it if not already registered.

**Example:**

```js
$.jqr.config({ defaultPrefix: 'en' });
```

```html
<!-- Now use en: instead of $: -->
<div en:text="message"></div>
```

**Rules:**
- Must start with a letter
- Can contain letters, digits, hyphens
- No underscores or special characters

#### `transitionEngine`

**Type:** `'native' | 'jquery'`  
**Default:** `'native'`

Set the transition implementation:

- **`'native'`** - CSS class-based transitions (no jQuery required)
- **`'jquery'`** - Uses jQuery's `.fadeIn()`, `.fadeOut()`, etc. (requires full jQuery)

**Example:**

```js
// Use native CSS transitions
$.jqr.config({ transitionEngine: 'native' });

// Use jQuery animations (requires jQuery loaded)
$.jqr.config({ transitionEngine: 'jquery' });
```

**Note:** If `'jquery'` is set but jQuery isn't loaded, falls back to `'native'` with a dev warning.

#### `sanitizer`

**Type:** `(html: string) => string | null`  
**Default:** `null`

Register a global HTML sanitizer for `$:html` directive. Pass `null` to remove.

**Example:**

```js
// Using DOMPurify
$.jqr.config({
  sanitizer: (html) => DOMPurify.sanitize(html)
});

// Custom sanitizer
$.jqr.config({
  sanitizer: (html) => {
    return html.replace(/<script\b[^<]*(?:(?!<\/script>)<[^<]*)*<\/script>/gi, '');
  }
});

// Remove sanitizer
$.jqr.config({ sanitizer: null });
```

### Complete Example

```js
$.jqr.config({
  devMode: import.meta.env.DEV,
  defaultPrefix: 'app',
  transitionEngine: 'native',
  sanitizer: (html) => DOMPurify.sanitize(html, {
    ALLOWED_TAGS: ['b', 'i', 'em', 'strong', 'a', 'p', 'br'],
    ALLOWED_ATTR: ['href', 'title']
  })
});
```

---

## Prefix Management

### `$.jqr.registerPrefix(name)`

Register an additional directive prefix.

**Signature:**

```typescript
$.jqr.registerPrefix(name: string): void
```

**Parameters:**
- `name` - Prefix string (letters, digits, hyphens only)

**Example:**

```js
$.jqr.registerPrefix('en');
$.jqr.registerPrefix('v');
$.jqr.registerPrefix('x-alpine');
```

```html
<!-- All work simultaneously -->
<div $:text="a"></div>
<div en:text="b"></div>
<div v:text="c"></div>
<div x-alpine:text="d"></div>
```

**Throws:** Error if prefix is invalid (starts with digit, contains invalid characters)

**Note:** Calling with an already-registered prefix is a no-op (safe to call multiple times).

---

## Utility Methods

### `$.jqr.isDevMode()`

Check if dev mode is currently enabled.

**Signature:**

```typescript
$.jqr.isDevMode(): boolean
```

**Returns:** `true` if dev mode is enabled, `false` otherwise

**Example:**

```js
if ($.jqr.isDevMode()) {
  console.log('Running in development mode');
}
```

### `$.jqr.isUsingFallback()`

Check if using JQR's built-in `$` fallback or real jQuery.

**Signature:**

```typescript
$.jqr.isUsingFallback(): boolean
```

**Returns:** 
- `true` - Using JQR's minimal fallback
- `false` - Real jQuery is loaded

**Example:**

```js
if ($.jqr.isUsingFallback()) {
  console.log('Using JQR fallback - some jQuery features unavailable');
} else {
  console.log('Full jQuery available');
}
```

---

## Custom Directives

### `$.jqr.directive(name, definition)`

Register a custom directive.

**Signature:**

```typescript
interface DirectiveContext<S = any, P = any> {
  el: HTMLElement;
  $el: JQuery;
  prefix: string;
  name: string;
  arg?: string;
  modifiers: string[];
  expr: string;
  evaluate: (expr: string, extra?: Dict) => any;
  scope: Dict;
  instance: JqrComponent<S, P>;
  raw?: () => any;
}

interface DirectiveDefinition<S = any, P = any> {
  bind?(el: HTMLElement, ctx: DirectiveContext<S, P>): void;
  update?(el: HTMLElement, ctx: DirectiveContext<S, P>): void;
  unbind?(el: HTMLElement, ctx: DirectiveContext<S, P>): void;
}

$.jqr.directive(name: string, definition: DirectiveDefinition): void
```

**Parameters:**
- `name` - Directive name (without prefix)
- `definition` - Lifecycle hooks object

**Lifecycle Hooks:**

#### `bind(el, ctx)`

Called once when directive is first found during component scan.

**Use for:**
- Initial setup
- Event listener registration
- DOM element creation

#### `update(el, ctx)`

Called initially (after `bind`), then reactively re-runs when dependencies change.

**Automatically tracked** - reads via `ctx.evaluate()` register dependencies.

**Use for:**
- Updating DOM based on state
- Applying dynamic attributes
- Reactive logic

#### `unbind(el, ctx)`

Called when element is removed from DOM.

**Use for:**
- Cleanup
- Removing event listeners
- Canceling timers
- Destroying third-party widgets

**Example:**

```js
$.jqr.directive('tooltip', {
  bind(el, ctx) {
    console.log('Tooltip bound to', el);
  },
  update(el, ctx) {
    const message = ctx.evaluate(ctx.expr);
    el.title = message;
  },
  unbind(el, ctx) {
    el.title = '';
  }
});
```

**Usage:**

```html
<button $:tooltip="helpMessage">Help</button>
```

**See:** [Custom Directives Guide](../guides/custom-directives.md)

---

## Component Registration

### `$.jqr.register(name, options)`

Register a component definition globally for use with `$:component`.

**Signature:**

```typescript
$.jqr.register(name: string, options: ComponentOptions): void
```

**Parameters:**
- `name` - Component name (for `$:component="name"`)
- `options` - Component definition

**Example:**

```js
$.jqr.register('user-card', {
  props: ['userId'],
  state: { user: null },
  asyncState: {
    async user(state, props) {
      const res = await fetch(`/api/users/${props.userId}`);
      return res.json();
    }
  }
});
```

```html
<div $:component="user-card" $:prop:user-id="currentUserId"></div>
```

**See:** [Nested Components Guide](../guides/nested-components.md)

---

## Plugin System

### `$.jqr.plugin(plugin)`

Register a plugin - bundle of directives and setup logic.

**Signature:**

```typescript
interface JqrPlugin {
  name: string;
  directives?: Record<string, DirectiveDefinition>;
  install?(jqr: typeof $.jqr): void;
  onInstance?(instance: JqrComponent): void;
}

$.jqr.plugin(plugin: JqrPlugin): void
```

**Parameters:**
- `plugin` - Plugin definition

**Plugin Shape:**

#### `name`

Plugin identifier (for duplicate detection).

#### `directives`

Map of directive names to definitions. Each is registered via `$.jqr.directive()`.

#### `install(jqr)`

Called once after directives are registered. Use for:
- Attaching utilities to `$.jqr` namespace
- One-time setup
- Injecting CSS

#### `onInstance(instance)`

Called for each component instance after creation. Use for:
- Instance-level initialization
- Patching component methods
- Adding refs/disposers

**Example:**

```js
$.jqr.plugin({
  name: 'tooltip-plugin',
  directives: {
    tooltip: {
      bind(el, ctx) {
        el.title = ctx.evaluate(ctx.expr);
      }
    }
  },
  install(jqr) {
    console.log('Tooltip plugin installed');
    jqr.tooltipConfig = { delay: 200 };
  },
  onInstance(instance) {
    console.log('New component instance:', instance.el.id);
  }
});
```

**Note:** Registering the same `name` twice logs a dev warning but proceeds (last one wins).

---

## Timing Utilities

### `$.jqr.nextTick(callback)`

Schedule callback to run after the next DOM update.

**Signature:**

```typescript
$.jqr.nextTick(callback: () => void): void
```

**Example:**

```js
$('#app').component({
  state: { message: 'Hello' },
  methods: {
    update(state) {
      state.message = 'Updated';
      
      $.jqr.nextTick(() => {
        // DOM has been updated
        console.log('New text:', this.$el.text());
      });
    }
  }
});
```

### `$.jqr.debounce(fn, delay)`

Create a debounced function.

**Signature:**

```typescript
interface Cancelable {
  (...args: any[]): void;
  cancel(): void;
}

$.jqr.debounce<T extends (...args: any[]) => any>(
  fn: T,
  delay: number
): Cancelable
```

**Parameters:**
- `fn` - Function to debounce
- `delay` - Delay in milliseconds

**Returns:** Debounced function with `.cancel()` method

**Example:**

```js
const search = $.jqr.debounce((query) => {
  console.log('Searching for:', query);
  // API call...
}, 300);

search('hello');
search('hello world'); // Only this runs (after 300ms)

// Cancel pending call
search.cancel();
```

### `$.jqr.throttle(fn, delay)`

Create a throttled function.

**Signature:**

```typescript
$.jqr.throttle<T extends (...args: any[]) => any>(
  fn: T,
  delay: number
): Cancelable
```

**Parameters:**
- `fn` - Function to throttle
- `delay` - Minimum time between calls (ms)

**Returns:** Throttled function with `.cancel()` method

**Example:**

```js
const onScroll = $.jqr.throttle(() => {
  console.log('Scroll position:', window.scrollY);
}, 200);

window.addEventListener('scroll', onScroll);

// Cleanup
onScroll.cancel();
window.removeEventListener('scroll', onScroll);
```

### `$.jqr.loadScript(url, options?)`

Load external JavaScript files (classic or ES module).

**Signature:**

```typescript
interface LoadScriptOptions {
  module?: boolean;      // Load as ES module (default: false)
  integrity?: string;    // Subresource Integrity hash
}

$.jqr.loadScript(url: string, options?: LoadScriptOptions): Promise<void>
```

**Parameters:**
- `url` - Script URL to load
- `options.module` - Load as `<script type="module">` (default: `false`)
- `options.integrity` - SRI hash for security (automatically adds `crossorigin="anonymous"`)

**Returns:** Promise that resolves when script loads

**Features:**
- **Shared loading:** Same URL = one load (concurrent calls wait together)
- **Caching:** Successful loads cached (subsequent calls resolve immediately)
- **Retry on failure:** Failed loads NOT cached (can retry)
- **SRI support:** Subresource Integrity for CDN scripts

**Examples:**

```js
// Load classic script
await $.jqr.loadScript('/lib/plugin.js');

// Load ES module
await $.jqr.loadScript('/lib/module.js', { module: true });

// Load from CDN with SRI
await $.jqr.loadScript(
  'https://cdn.example.com/lib.js',
  { integrity: 'sha384-oqVuAfXRKap7fdgcCY5uykM6+R9GqQ8K/ux...' }
);

// Concurrent loading (shared)
const [p1, p2] = await Promise.all([
  $.jqr.loadScript('/lib/shared.js'),
  $.jqr.loadScript('/lib/shared.js')  // Waits for same load
]);

// Error handling
try {
  await $.jqr.loadScript('/lib/missing.js');
} catch (err) {
  console.error('Load failed:', err);
  // Can retry (failure not cached)
  await $.jqr.loadScript('/lib/missing.js');
}
```

**Use with Async Components:**

```js
// Self-registering legacy component
$.jqr.registerAsync('legacy-widget', async () => {
  await $.jqr.loadScript('/lib/legacy-widget.js');
  return null;  // Component registered itself
});

// Load dependencies then component
$.jqr.registerAsync('chart', async () => {
  // Load charting library first
  await $.jqr.loadScript('https://cdn.example.com/charts.js', {
    integrity: 'sha384-...'
  });
  // Then load component
  const module = await import('/components/chart.js');
  return module.default;
});
```

**Error Code:** Failed loads throw error with code `'jqr.async.scriptLoadFailed'` (E081).

---

## Reactivity Primitives

> **Note:** These reactivity primitives are available on the **`JQR`** global namespace (not `$.jqr`). They are low-level APIs primarily used by `@en32/jqr-h` and advanced use cases. Most application code doesn't need them directly.
>
> - **Browser/IIFE:** Use `JQR.reactive()`, `JQR.effect()`, etc.
> - **ESM/Bundler:** Use `import { reactive, effect } from '@en32/jqr'`

### `JQR.reactive(obj)`

Make an object reactive.

**Signature:**

```typescript
JQR.reactive<T extends object>(obj: T): T
```

**Example:**

```js
const data = JQR.reactive({
  count: 0,
  items: []
});

JQR.effect(() => {
  console.log('Count changed:', data.count);
});

data.count++; // Logs: "Count changed: 1"
```

### `JQR.effect(fn)`

Run a function and track its dependencies.

**Signature:**

```typescript
JQR.effect(fn: () => void): () => void
```

**Returns:** Cleanup function

**Example:**

```js
const data = JQR.reactive({ x: 0 });

const cleanup = JQR.effect(() => {
  console.log('x is now:', data.x);
});

data.x = 10; // Logs: "x is now: 10"

cleanup(); // Stop tracking
```

### `JQR.computed(fn)`

Create a computed value.

**Signature:**

```typescript
interface ComputedRef<T> {
  value: T;
}

JQR.computed<T>(fn: () => T): ComputedRef<T>
```

**Example:**

```js
const data = JQR.reactive({ a: 1, b: 2 });
const sum = JQR.computed(() => data.a + data.b);

console.log(sum.value); // 3

data.a = 5;
console.log(sum.value); // 7
```

### `JQR.watch(source, callback, options)`

Watch a reactive source and run callback on change.

**Signature:**

```typescript
interface WatchOptions {
  deep?: boolean;
}

JQR.watch<T>(
  source: () => T,
  callback: (newVal: T, oldVal: T) => void,
  options?: WatchOptions
): () => void
```

**Example:**

```js
const data = JQR.reactive({ count: 0 });

const cleanup = JQR.watch(
  () => data.count,
  (newVal, oldVal) => {
    console.log(`Changed from ${oldVal} to ${newVal}`);
  }
);

data.count = 5; // Logs: "Changed from 0 to 5"

cleanup(); // Stop watching
```

### `JQR.toRaw(proxy)`

Get the original non-reactive object from a proxy.

**Signature:**

```typescript
JQR.toRaw<T>(proxy: T): T
```

**Example:**

```js
const original = { count: 0 };
const reactive = JQR.reactive(original);

console.log(JQR.toRaw(reactive) === original); // true
```

---

## Confirmation System

### `$.jqr.dispatchConfirm(detail)`

Dispatch a global confirmation request event.

**Signature:**

```typescript
interface ConfirmEventDetail {
  message: string;
  context?: any;
}

$.jqr.dispatchConfirm(detail: ConfirmEventDetail): void
```

**Example:**

```js
$.jqr.dispatchConfirm({
  message: 'Are you sure you want to delete this?',
  context: { itemId: 123 }
});
```

**See:** [Confirmation Pattern Guide](../guides/confirm-pattern.md)

### `$.jqr.confirmGate(detail)`

Dispatch a confirmation gate event (for $:confirm.gate).

**Signature:**

```typescript
interface ConfirmGateEventDetail {
  message: string;
  context?: any;
  proceed: () => void;
  cancel: () => void;
}

$.jqr.confirmGate(detail: ConfirmGateEventDetail): void
```

**Internal use** - called by `$:confirm.gate` directive.

---

## Auto-Boot

### `$.jqr.scan()`

Manually scan document for `[jqr-autoboot]` elements.

**Signature:**

```typescript
$.jqr.scan(): void
```

**Example:**

```js
// Disable automatic scanning
$.jqr.config({ autoBoot: false });

// Later, manually trigger
$.jqr.scan();
```

**Note:** Auto-boot runs automatically on `DOMContentLoaded` unless disabled.

---

## Batch Mounting

### `$.jqr.mountEach(selector, factory)`

Mount multiple components with optional shared state.

**Signature:**

```typescript
interface MountEachConfig<S> {
  mode: 'independent' | 'shared';
  state?: S;
}

$.jqr.mountEach(
  selector: string,
  factory: (el: HTMLElement, index: number) => ComponentOptions | MountEachConfig
): JqrComponent[]
```

**Example:**

```js
// Independent instances
$.jqr.mountEach('.counter', (el, index) => ({
  state: { count: index }
}));

// Shared state
$.jqr.mountEach('.synced-counter', () => ({
  mode: 'shared',
  state: { count: 0 }
}));
```

**See:** [mountEach Guide](../guides/mount-each.md)

---

## TypeScript Types

All types are exported from `@en32/jqr`:

```typescript
import type {
  ComponentOptions,
  JqrComponent,
  DirectiveContext,
  DirectiveDefinition,
  Dict,
  WatchOptions,
  AsyncStateFn,
  MethodFn,
  DestructuredMethodFn,
  MethodContext,
  ComputedRef,
  JqrPlugin,
  TransitionEngine,
  ConfirmEventDetail
} from '@en32/jqr';
```

---

## Next Steps

- [Prefixes & Configuration Guide](../guides/prefixes-and-config.md) - Detailed usage examples
- [Custom Directives Guide](../guides/custom-directives.md) - Building directives
- [Reactivity Guide](../guides/reactivity.md) - Understanding reactive system
- [API Reference](./directive-reference.md) - Built-in directives
