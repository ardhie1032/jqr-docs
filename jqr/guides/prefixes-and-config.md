<!-- {
  "path": "jqr/guides/prefixes-and-config",
  "name": "docs-jqr-guides-prefixes-and-config",
  "meta": {
    "title": "Prefixes & Configuration",
    "order": 11
  }
} -->

# Prefixes & Configuration

JQR provides configuration options to customize its behavior and support multiple directive prefixes beyond the default `$:`.

## Understanding Prefixes

Before diving into configuration, it's important to understand what prefixes are and how they differ from directive names.

### Two Orthogonal Axes

When you see an attribute like `$:on:click="foo"`, two independent aspects are at play:

| Axis | Part | Determines |
|------|------|------------|
| **Prefix/Namespace** | `$` or `en` (before first `:`) | *Who* (which component/root) can read this attribute |
| **Directive Name** | `on:click` vs `click` vs `model` | *What* this attribute does |

**Example breakdown:**

```
$ : on : click . prevent = "foo"
│   │    │      │           └── expr — JS expression evaluated in component scope
│   │    │      └─────────────── modifiers (separated by `.`)
│   │    └────────────────────── arg (part after second `:`)
│   └─────────────────────────── name (directive name)
└─────────────────────────────── prefix
```

### Directive Names: `on:click` vs `click`

Both end up calling the same `bindEvent()` function with identical modifier support. The difference is purely *how* the event name is found:

**Shorthand `$:click="foo"`:**

1. Is `click` a registered custom directive? → Use custom directive
2. Is `click` in `KNOWN_EVENTS`? → Bind DOM listener
3. Neither? → Dev warning + ignored

`KNOWN_EVENTS` is a closed list:
```
click, dblclick, input, change, submit, keydown, keyup, keypress,
focus, blur, mouseenter, mouseleave, mousedown, mouseup, mousemove,
mouseover, mouseout, contextmenu, wheel, scroll, drag, drop, dragover,
touchstart, touchend, touchmove, load, resize, reset, select, toggle
```

```html
<!-- ✅ Works: click is in KNOWN_EVENTS -->
<button $:click="count++">+1</button>

<!-- ❌ Doesn't work: pointerdown not in KNOWN_EVENTS -->
<button $:pointerdown="foo()">Never binds</button>
```

**Explicit `$:on:click="foo"`:**

No whitelist — event name taken directly from `arg`. This is the escape hatch for any event, including custom events.

```html
<!-- ✅ Works: any event name allowed -->
<button $:on:pointerdown="startDrag($event)">Works</button>
<div $:on:my-lib:ready="onReady()">Custom event works</div>
```

### Critical Difference: On Component Hosts

On elements that also have `$:component`, **behavior diverges**:

**`$:on:<event>` on component host:**
- Treats as **child component event** (`$emit`)
- Listener bound to **parent scope**
- Attribute removed before child component scans

**`$:click` on component host:**
- **Not treated specially**
- Not removed, scanned by **child component**
- Becomes DOM listener in **child scope**

```html
<!-- PARENT -->
<div $:component="user-card"
     $:prop:name="currentUser.name"
     $:on:save="handleSave($event)">   <!-- Catches this.$emit('save', payload) from child,
                                            evaluated in PARENT scope -->
</div>
```

```js
$.jqr.register('user-card', {
  template: `<button $:click="$emit('save', { id: 1 })">Save</button>`
});
```

**Rule of thumb:**

| Location | `$:on:x` | `$:x` |
|----------|----------|-------|
| Regular element | DOM listener `x` | DOM listener `x` (only if `x ∈ KNOWN_EVENTS`) |
| Component host | Listener for `$emit('x')` from child, parent scope | DOM listener in **child scope** |

**Best practice:** Always use `$:on:*` whenever the element is also a component host.

## Configuration API

Configure JQR globally using `$.jqr.config()`:

```js
$.jqr.config({
  devMode: true,
  defaultPrefix: 'en',
  transitionEngine: 'jquery',
  sanitizer: (html) => DOMPurify.sanitize(html)
});
```

**Call before mounting components** — configuration applies globally to all components created afterward.

## Dev Mode

### Enabling Dev Mode

```js
$.jqr.config({ devMode: true });
```

**What it does:**
- Enables development-only warnings
- Warns about unknown directives
- Warns about `$:for` without `$:key`
- Warns about conflicting `$:if` + `$:show`
- Warns about other potential issues

### Examples

```html live:no-code
<div id="demo-dev-mode" class="en-card en-card__body">
  <p>Check console for warnings (dev mode enabled)</p>
  <button class="en-btn en-btn--sm" $:click="test()">Test Dev Warnings</button>
</div>
```

```html
<div id="demo-dev-mode">
  <p>Check console for warnings (dev mode enabled)</p>
  <button $:click="test()">Test Dev Warnings</button>
</div>
```

```js inject show
// Enable dev mode
$.jqr.config({ devMode: true });

$('#demo-dev-mode').component({
  state: { items: [1, 2, 3] },
  methods: {
    test(state) {
      console.log('Dev mode is:', JQR.isDevMode());
    }
  }
});
```

```js cleanup
onUnmount(() => {
  $('#demo-dev-mode').component('destroy');
  $.jqr.config({ devMode: false }); // Reset
})
```

**Unknown Directive Warning:**

```html
<!-- ⚠️ Warning: unrecognized directive "$:typo" -->
<div $:typo="value"></div>
```

**Missing Key Warning:**

```html
<!-- ⚠️ Warning: $:for without $:key -->
<li $:for="item in items" $:text="item"></li>
```

**Conflicting Directives:**

```html
<!-- ⚠️ Warning: both $:if and $:show on same element -->
<div $:if="condition" $:show="otherCondition"></div>
```

### Production Mode

```js
// Disable warnings for production
$.jqr.config({ devMode: false });
```

**Default:** `false` (warnings disabled)

**Best practice:** Enable in development, disable in production builds.

## Directive Prefixes

### Prefix Must Be Registered

Only `$` is active by default. `en:*` **means nothing** until registered:

```js
$.jqr.prefix('en');                      // `en:*` recognized, default stays `$`
// OR
$.jqr.config({ defaultPrefix: 'en' });   // `en:*` recognized AND becomes default
```

**Note:** Registering a new default **doesn't disable** `$:*` — both coexist (intentional, for gradual migration).

```html
<!-- Before $.jqr.prefix('en') is called -->
<button en:click="foo">Never read — dead HTML attribute</button>
```

### Prefix is Per-Component Scanning Scope

Each `Component` stores one `_prefix`:

```js
this._prefix = options.prefix || getDefaultPrefix();
```

And all subtree scanning uses it: `scanSubtree(root, this._prefix, ...)`.

**Consequence:** One component sees only one prefix. Directives with other prefixes in its subtree are silently ignored (no warning, because the regex doesn't match).

```js
$('#app-a').component({ prefix: '$',  state: { n: 0 } });
$('#app-b').component({ prefix: 'en', state: { n: 0 } });
```

```html
<div id="app-a">
  <button $:click="n++">Works</button>
  <button en:click="n++">IGNORED — #app-a only scans prefix `$`</button>
</div>

<div id="app-b">
  <button en:click="n++">Works</button>
  <button $:click="n++">IGNORED</button>
</div>
```

This is why `options.prefix` is resolved to a concrete prefix rather than left `undefined`: `buildAttrMatcher(undefined)` would mean "match all registered prefixes", making components without explicit `prefix` absorb directives from all prefixes in their subtree — the opposite of the isolation this feature is meant to provide.

### Prefix is Inherited by Child Components

```js
const child = new Component(hostEl, { ...def, prefix: this._prefix, props: initialProps });
```

Child component template **must** use parent's prefix. Writing a component library with `$:*` then mounting it in a page running `en:*` will make the child template inert.

### Prefix on Autoboot Determines Component Prefix

`initAutoBoot()` runs once on `DOMContentLoaded` and searches for `<prefix>:component` / `<prefix>:component:<key>` for **every** registered prefix, then:

```js
new Component(el, { ...def, prefix });
```

The prefix you write on the `component` attribute locks the entire subtree to that prefix:

```html
<!-- Root found via prefix `en` → entire content must use `en:` -->
<div en:component:state="{ username: '', count: 0 }">
  <input  en:model="username">
  <p      en:text="username"></p>
  <button en:click="count++">+1</button>

  <p $:text="username">IGNORED — subtree locked to prefix `en`</p>
</div>
```

Two side-by-side roots with different prefixes are perfectly valid, since each has its own `Component`:

```html
<div $:component="widget-a">  <button $:click="go()">…</button>  </div>
<div en:component="widget-b"> <button en:click="go()">…</button> </div>
```

### What Else Follows the Prefix

- `$:cloak` → `ensureCloakStyle()` injects CSS rules for the component's prefix, and `_uncloak()` removes `<prefix>:cloak`. So `en:cloak` only works in `en`-prefixed components.
- Slots (`$:slot`, `$:scope`), `$:for`, `$:if` — all searched via `getElementDirectives(el, this._prefix)`.

### Prefix Scope: Components Yes, Registries No

**For components: Yes**

As shown in earlier sections, one `Component` instance = one prefix. This is the isolation unit: two different apps/vendors can coexist on one page without absorbing each other's attributes.

**For registries: No**

All registries are **global and keyed by name only**, with no prefix dimension:

| Registry | API | Keyed By |
|----------|-----|----------|
| Components | `$.jqr.register(name, opts)` | `name` |
| Custom directives | `$.jqr.directive(name, def)` | `name` |
| Plugins | `$.jqr.plugin(def)` | `def.name` (only for duplicate detection; directives still go to global directive registry) |
| Model presets | `registerModelPreset(tag, preset)` | tag name |

**Practical consequence:**

```js
$.jqr.prefix('en');
$.jqr.directive('tooltip', { bind() { /* ... */ } });
```

```html
<span $:tooltip="'Hello'">…</span>   <!-- In `$` prefix component  → works -->
<span en:tooltip="'Hello'">…</span>  <!-- In `en` prefix component → works, SAME definition -->
```

Registering a `tooltip` directive **cannot** be limited to only `en:`. Prefix only determines which attributes are *seen*, not which directives *exist*.

The only registry scoping that truly exists is `options.components` — local component definitions for one instance, checked **before** the global registry when resolving `$:component="name"`:

```js
$('#app').component({
  prefix: 'en',
  components: {
    'row': { template: '<li en:text="label"></li>' }   // Only visible to #app
  }
});
```

### Use Cases

**Isolation between apps:**

```html
<!-- Two independent apps with different prefixes -->
<div $:component:state="{ count: 0 }">
  <button $:click="count++">App A: {{ count }}</button>
</div>

<div en:component:state="{ count: 0 }">
  <button en:click="count++">App B: {{ count }}</button>
</div>
```

**Framework-agnostic templates:**

```html
<!-- Works for both Vue and JQR -->
<div v:if="condition" v:text="message"></div>
```

```js
// JQR side
$.jqr.prefix('v');
```

### Prefix Rules

**Valid prefixes:**
- Must start with a letter
- Can contain letters, digits, and hyphens
- Examples: `$`, `en`, `v`, `bs`, `my-prefix`

**Invalid prefixes:**

```js
// ❌ Error: invalid prefix
$.jqr.prefix('123');       // Starts with number
$.jqr.prefix('my_prefix'); // Underscore not allowed
```

## HTML Sanitization

### Default Behavior

`$:html` directive is **XSS-risky** with untrusted content:

```html
<!-- ⚠️ DANGEROUS if userInput contains <script> tags -->
<div $:html="userInput"></div>
```

### Global Sanitizer

Register a sanitizer to protect all `$:html` usage:

```js
// Using DOMPurify
$.jqr.config({
  sanitizer: (html) => DOMPurify.sanitize(html)
});
```

Now all `$:html` is automatically sanitized:

```html
<!-- Safe - script tags removed by DOMPurify -->
<div $:html="userGeneratedContent"></div>
```

### Example Integration

```html live:no-code
<div id="demo-sanitizer" class="en-card en-card__body">
  <textarea class="en-textarea en-mb-2" $:model="userInput" rows="3" placeholder="Try entering \<script\>alert('XSS')\</script\>"></textarea>
  <p class="en-text--muted en-text-sm en-mb-2">Sanitized output:</p>
  <div class="en-card en-card--bordered" style="padding: 10px; min-height: 50px;" $:html="userInput"></div>
</div>
```

```html
<div id="demo-sanitizer">
  <textarea $:model="userInput" rows="3" placeholder="Try entering script tags"></textarea>
  <p>Sanitized output:</p>
  <div $:html="userInput"></div>
</div>
```

```js inject show
// Mock sanitizer (in real app, use DOMPurify)
$.jqr.config({
  sanitizer: (html) => {
    // Simple sanitization - removes script tags
    return html.replace(/<script\b[^<]*(?:(?!<\/script>)<[^<]*)*<\/script>/gi, '[script removed]');
  }
});

$('#demo-sanitizer').component({
  state: { userInput: '<b>Bold text</b> is safe' }
});
```

```js cleanup
onUnmount(() => {
  $('#demo-sanitizer').component('destroy');
  $.jqr.config({ sanitizer: null }); // Reset
})
```

### Removing Sanitizer

```js
$.jqr.config({ sanitizer: null });
```

## Transition Engine

### Native (Default)

Uses CSS classes for transitions:

```js
$.jqr.config({ transitionEngine: 'native' });
```

```html
<div $:transition="visible">Fades with CSS</div>
```

### jQuery Engine

Uses jQuery's `.fadeIn()`, `.fadeOut()`, etc.:

```js
$.jqr.config({ transitionEngine: 'jquery' });
```

**Requires:** jQuery loaded (not the built-in fallback)

```html
<!-- Load jQuery first -->
<script src="https://code.jquery.com/jquery-4.0.0.min.js"></script>
<script src="jqr.min.js"></script>

<script>
$.jqr.config({ transitionEngine: 'jquery' });
</script>
```

**Fallback:** If jQuery isn't loaded, falls back to `native` with a dev warning.

See: [Transitions Guide](./transitions.md)

## Complete Configuration Example

```js
// Main app initialization
$.jqr.config({
  // Development warnings
  devMode: process.env.NODE_ENV !== 'production',
  
  // Custom prefix
  defaultPrefix: 'app',
  
  // HTML sanitization
  sanitizer: (html) => DOMPurify.sanitize(html, {
    ALLOWED_TAGS: ['b', 'i', 'em', 'strong', 'a', 'p', 'br'],
    ALLOWED_ATTR: ['href', 'title']
  }),
  
  // Transition engine
  transitionEngine: 'native'
});

// Register additional prefixes
$.jqr.registerPrefix('v');    // Vue compatibility
$.jqr.registerPrefix('x');    // Alpine compatibility

// Now mount components
$('#app').component({
  state: { message: 'Hello World' }
});
```

## Configuration API Reference

### `$.jqr.config(options)`

**Options:**

| Option | Type | Default | Description |
|--------|------|---------|-------------|
| `devMode` | `boolean` | `false` | Enable development warnings |
| `defaultPrefix` | `string` | `'$'` | Default directive prefix |
| `transitionEngine` | `'native'` \| `'jquery'` | `'native'` | Transition implementation |
| `sanitizer` | `(html: string) => string` \| `null` | `null` | Global HTML sanitizer |

### `$.jqr.prefix(name)`

Register an additional directive prefix.

**Parameters:**
- `name` - Prefix string (letters, digits, hyphens only, must start with letter)

**Example:**

```js
$.jqr.prefix('en');
$.jqr.prefix('app');
```

**Note:** This is an alias for `$.jqr.registerPrefix(name)` (both work identically).

### `$.jqr.isDevMode()`

Check if dev mode is enabled.

**Returns:** `boolean`

**Example:**

```js
if ($.jqr.isDevMode()) {
  console.log('Development mode is active');
}
```

## Environment-Based Configuration

### Vite

```js
// vite.config.js defines import.meta.env.DEV

$.jqr.config({
  devMode: import.meta.env.DEV
});
```

### Webpack

```js
// webpack.config.js defines process.env.NODE_ENV

$.jqr.config({
  devMode: process.env.NODE_ENV !== 'production'
});
```

### Conditional Loading

```js
// Load DOMPurify only in production
if (process.env.NODE_ENV === 'production') {
  $.jqr.config({
    sanitizer: (html) => DOMPurify.sanitize(html)
  });
}
```

## Best Practices

### 1. Configure Once, Early

```js
// ✅ Good - before any components
$.jqr.config({ devMode: true });

$('#app').component({ /* ... */ });

// ❌ Bad - after components mounted
$('#app').component({ /* ... */ });

$.jqr.config({ devMode: true }); // Too late!
```

**Important:** Register prefixes **before** any components mount or before `DOMContentLoaded`, since autoboot scans based on the prefix list available at that time.

### 2. One Prefix Per Page/App

```js
// ✅ Good - consistent across app
$.jqr.config({ defaultPrefix: 'app' });

// ❌ Bad - mixing prefixes randomly
<div $:text="a"></div>
<div en:text="b"></div>
<div v:text="c"></div>
```

Only mix prefixes when you genuinely need to isolate two independent applications.

### 3. Use Dev Mode in Development

```js
// ✅ Good - catch issues early
$.jqr.config({
  devMode: process.env.NODE_ENV !== 'production'
});
```

**Note:** Typos in directive **names** (`$:clik`) only show warnings in dev mode; typos in **prefix** (`en:click` without registration) produce no warning at all.

### 4. Always Sanitize User HTML

```js
// ✅ Good - safe from XSS
$.jqr.config({
  sanitizer: (html) => DOMPurify.sanitize(html)
});

// ❌ Bad - vulnerable to XSS
<div $:html="userGeneratedContent"></div>
```

### 5. Use `$:on:*` for Component Events

```js
// ✅ Good - explicit component event listener
<div $:component="user-card"
     $:on:save="handleSave($event)">
</div>

// ❌ Bad - becomes DOM listener in child scope
<div $:component="user-card"
     $:click="handleSave($event)">
</div>
```

### 6. Library Components and Prefix

If distributing a component library:

```js
// ❌ Bad - hardcoded prefix in template
$.jqr.register('my-widget', {
  template: `<div $:text="label"></div>`  // Won't work if user uses en:
});

// ✅ Good - expose prefix as option or document requirement
$.jqr.register('my-widget', {
  template: `<div $:text="label"></div>`,
  // Document: "This component requires $ prefix"
});
```

## Common Patterns

### Development Setup

```js
// config/development.js
export function setupDevMode() {
  $.jqr.config({
    devMode: true,
    sanitizer: null // Skip sanitization in dev for easier debugging
  });
}
```

### Production Setup

```js
// config/production.js
export function setupProdMode() {
  $.jqr.config({
    devMode: false,
    sanitizer: (html) => DOMPurify.sanitize(html),
    transitionEngine: 'native' // Smaller bundle
  });
}
```

### Framework Migration

```js
// Migrating from Vue - support both prefixes temporarily
$.jqr.prefix('v');

// Old templates still work
<div v:if="condition" v:text="message"></div>

// New templates use JQR prefix
<div $:if="condition" $:text="message"></div>
```

**Note:** Custom prefix applies to **all directives** without exception — `v:for`, `v:if`, `v:model`, etc. all work identically to their `$:` counterparts. The switch/case in `_applyDirectives()` only ever sees `d.name`, never `d.prefix`.

## Troubleshooting

### Directives Not Working

**Problem:** Directives are ignored silently

**Solutions:**

1. **Check prefix matches:**

```js
// If using custom prefix
$.jqr.config({ defaultPrefix: 'en' });

// Must use en: not $:
<div en:text="message"></div>
```

2. **Ensure prefix is registered before autoboot:**

```js
// ✅ Good - before DOMContentLoaded
$.jqr.prefix('en');

// ❌ Bad - after DOMContentLoaded
// (autoboot already ran, didn't find en:component roots)
```

3. **Check component prefix inheritance:**

```html
<!-- Parent uses en: prefix -->
<div en:component:state="{}">
  <!-- Child must also use en:, not $: -->
  <button en:click="foo()">Works</button>
  <button $:click="foo()">Silently ignored</button>
</div>
```

### Event Handler Not Firing

**Problem:** Custom events don't work with shorthand

**Solution:** Use `$:on:*` for non-`KNOWN_EVENTS`:

```html
<!-- ❌ Doesn't work: pointerdown not in KNOWN_EVENTS -->
<button $:pointerdown="start()">Never binds</button>

<!-- ✅ Works: explicit event syntax -->
<button $:on:pointerdown="start()">Binds correctly</button>
```

### Component Event Not Reaching Parent

**Problem:** `$emit` in child not caught by parent

**Solution:** Use `$:on:*` (not shorthand) on component host:

```html
<!-- ❌ Wrong: becomes DOM listener in child scope -->
<div $:component="card" $:save="handleSave($event)"></div>

<!-- ✅ Correct: listens for $emit from child -->
<div $:component="card" $:on:save="handleSave($event)"></div>
```

### Sanitizer Not Applied

**Problem:** `$:html` still allows scripts

**Solution:** Ensure sanitizer is registered before mounting:

```js
// ✅ Register first
$.jqr.config({ sanitizer: DOMPurify.sanitize });

// Then mount
$('#app').component({...});
```

### Dev Warnings Not Showing

**Problem:** No warnings in console

**Solution:** Enable dev mode:

```js
$.jqr.config({ devMode: true });
```

### Warning Messages Show Wrong Prefix

**Problem:** Error says `$:if` but you're using `en:if`

**Explanation:** Some warning strings are hardcoded with `$:` — this is cosmetic only. The behavior is still correct.

## Summary

**Key Takeaways:**

1. **Prefix vs Name:** Prefix (`$` vs `en`) controls *who reads* the attribute; directive name (`on:click` vs `click`) controls *what it does*
2. **Shorthand limits:** `$:click` only works for `KNOWN_EVENTS`; use `$:on:*` for any event
3. **Component events:** Always use `$:on:*` on component hosts to catch `$emit`
4. **One prefix per component:** Each component scans only its own prefix; mixed prefixes in subtree are silently ignored
5. **Global registries:** Custom directives, plugins, and components are global — prefix doesn't create separate namespaces for them
6. **Configure early:** Register prefixes and config before mounting components or `DOMContentLoaded`

## Next Steps

- [Custom Directives](./custom-directives.md) - Creating your own directives
- [Configuration Reference](../reference/config-reference.md) - Complete API
- [Transitions](./transitions.md) - Using transition engines
- [Security](./security.md) - XSS protection and sanitization
