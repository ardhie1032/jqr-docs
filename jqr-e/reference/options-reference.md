<!-- {
  "path": "jqr-e/reference/options-reference",
  "name": "docs-jqr-e-reference-options-reference",
  "meta": {
    "title": "Options Reference",
    "order": 2
  }
} -->

# Element Options Reference

Quick reference guide for `ElementOptions` configuration.

## Overview

```typescript
interface ElementOptions {
  attributes?: Record<string, AttributeType>;
  reflect?: string[];
  expose?: string[];
  mount?: 'auto' | 'manual';
  display?: string;
  keepAlive?: boolean;
}
```

**Usage:**

```js
JQRE.defineElement('my-element', definition, {
  attributes: { /* ... */ },
  reflect: [ /* ... */ ],
  expose: [ /* ... */ ],
  mount: 'auto',
  display: 'block',
  keepAlive: false
});
```

---

## `attributes`

Configure HTML attribute to prop type coercion.

### Type

```typescript
attributes?: Record<string, AttributeType>

type AttributeType = StringConstructor | NumberConstructor | BooleanConstructor | 'json'
```

### Coercion Rules

| Type | HTML → JavaScript | Example |
|------|-------------------|---------|
| `String` | Use as-is | `name="Alice"` → `"Alice"` |
| `Number` | `Number(value)` | `count="42"` → `42` |
| `Boolean` | Presence check | `disabled` → `true` |
| `'json'` | `JSON.parse(value)` | `data='[1,2]'` → `[1,2]` |

### Automatic Inference

When omitted, types are inferred from default values:

```js
props: {
  count: 0,        // → Number
  name: '',        // → String
  active: false,   // → Boolean
  items: []        // → 'json'
}
```

### Explicit Configuration

```js
{
  attributes: {
    userId: Number,
    userName: String,
    disabled: Boolean,
    config: 'json'
  }
}
```

### Boolean Coercion

| HTML | JavaScript |
|------|------------|
| `<el disabled>` | `true` |
| `<el>` | `false` |
| `<el disabled="false">` | `false` |
| `<el disabled="true">` | `true` |

### JSON Coercion

```html
<!-- Array -->
<el items='[1,2,3]'>

<!-- Object -->
<el config='{"theme":"dark"}'>

<!-- Invalid JSON → ignored with warning -->
<el data='not-json'>
```

### When to Use

**Explicit configuration needed:**
- Prop default is `null` or `undefined`
- Multiple props with same default type
- Override inference

**Inference sufficient:**
- Props have typed default values
- Standard Number, String, Boolean props

---

## `reflect`

Mirror prop changes back to HTML attributes.

### Type

```typescript
reflect?: string[]
```

### Behavior

When a prop value changes internally, the corresponding attribute updates:

```js
{
  reflect: ['disabled', 'variant', 'loading']
}
```

**Effect:**

```js
// JavaScript
element.$instance.$state.disabled = true;

// HTML updates automatically
<my-element disabled>
```

### Reflection Rules

| Prop Value | HTML Attribute |
|------------|----------------|
| `true` | `attr=""` (empty) |
| `false` / `null` / `undefined` | Attribute removed |
| `string` / `number` | `attr="value"` |
| `object` / `array` | `attr='{"json":"value"}'` |

### Use Cases

**CSS Styling:**
```css
my-button[disabled] {
  opacity: 0.5;
  cursor: not-allowed;
}

my-button[variant="primary"] {
  background: blue;
}
```

**DOM Queries:**
```js
// Find all disabled buttons
document.querySelectorAll('my-button[disabled]');

// Find specific variant
document.querySelectorAll('my-button[variant="danger"]');
```

**Visual Debugging:**
```html
<!-- Attributes visible in DevTools -->
<my-button disabled variant="primary" loading>
```

### Performance Note

Reflection adds small overhead (DOM writes). Only reflect props needed for:
- CSS selectors
- External DOM queries
- Visual inspection

**Good:**
```js
reflect: ['disabled', 'loading', 'variant']
```

**Avoid:**
```js
// Unnecessary - internal state
reflect: ['count', 'internalFlag', 'tempValue']
```

---

## `expose`

Expose component methods directly on element.

### Type

```typescript
expose?: string[]
```

### Behavior

Makes component methods callable without accessing `$instance`:

```js
{
  expose: ['open', 'close', 'toggle', 'focus']
}
```

**Effect:**

```js
// Without expose
element.$instance.$ctx.open();

// With expose
element.open();
```

### Requirements

- Methods must exist in component's `methods` object
- Non-existent methods trigger dev warning
- Element must be mounted for methods to work

### Common Patterns

**Form Controls:**
```js
expose: ['focus', 'blur', 'select', 'clear', 'reset']
```

**Dialogs:**
```js
expose: ['open', 'close', 'toggle']
```

**Data Widgets:**
```js
expose: ['refresh', 'filter', 'sort', 'export']
```

**Media:**
```js
expose: ['play', 'pause', 'stop', 'seek']
```

### Safety

Exposed methods check mount status:

```js
const el = document.querySelector('my-dialog');

// Before mount → warning
el.open();  // ⚠️ "Element not mounted"

// After mount → works
await el.$mounted;
el.open();  // ✅
```

### Best Practices

**Expose:**
- Public API methods
- User-facing actions
- Common operations

**Don't Expose:**
- Internal utilities
- Private helpers
- State mutations

---

## `mount`

Control when element mounts its component.

### Type

```typescript
mount?: 'auto' | 'manual'
```

### Values

**`'auto'`** (default)
- Mount automatically when connected to DOM
- Standard behavior for most elements

**`'manual'`**
- Wait for explicit `mount()` call
- For lazy loading and deferred initialization

### Auto Mount

```js
{
  mount: 'auto'  // Default, can be omitted
}
```

```html
<my-widget></my-widget>
<!-- Mounts automatically -->
```

### Manual Mount

```js
{
  mount: 'manual'
}
```

```html
<my-widget></my-widget>
<!-- Doesn't mount until mount() called -->

<script>
  const el = document.querySelector('my-widget');
  el.mount();  // Now mounts
</script>
```

### Use Cases

**Auto (default):**
- Standard components
- Critical content
- Above-the-fold elements

**Manual:**
- Heavy components (charts, editors)
- Below-the-fold content
- Optional features
- Conditional rendering

### Lazy Loading Pattern

```js
// Manual mount + Intersection Observer
JQRE.defineElement('heavy-widget', definition, {
  mount: 'manual'
});

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

---

## `display`

Set default CSS `display` value.

### Type

```typescript
display?: string
```

### Behavior

Injects a CSS rule for the element:

```js
{
  display: 'block'
}
```

**Generated CSS:**
```css
my-element { display: block; }
```

### Default

Without `display` option, browser uses default: `inline`

### Common Values

| Value | Use Case |
|-------|----------|
| `'inline'` | Text-level elements, icons |
| `'inline-block'` | Buttons, badges, chips |
| `'block'` | Panels, cards, sections |
| `'flex'` | Flex containers |
| `'inline-flex'` | Inline flex containers |
| `'grid'` | Grid containers |
| `'none'` | Initially hidden |

### Examples

**Inline Elements:**
```js
JQRE.defineElement('app-badge', badgeDef, {
  display: 'inline-block'
});
```

```html
<p>Text with <app-badge>Badge</app-badge> inline.</p>
```

**Block Elements:**
```js
JQRE.defineElement('app-card', cardDef, {
  display: 'block'
});
```

```html
<app-card>Takes full width</app-card>
```

**Flex Containers:**
```js
JQRE.defineElement('flex-row', flexDef, {
  display: 'flex'
});
```

### Style Injection

The display rule is injected once in a `<style>` element:

```html
<style data-jqr-element>
app-badge { display: inline-block; }
app-card { display: block; }
</style>
```

### Override

Can be overridden with CSS:

```css
/* Override display option */
my-element {
  display: grid !important;
}
```

---

## `keepAlive`

Keep component alive when element disconnects from DOM.

### Type

```typescript
keepAlive?: boolean
```

### Values

**`false`** (default)
- Destroy component when disconnected
- Free memory and resources
- Clean state on reconnect

**`true`**
- Keep component alive when disconnected
- Preserve state across disconnect/reconnect
- Requires explicit `unmount()` for cleanup

### Default Behavior (keepAlive: false)

```js
{
  keepAlive: false  // Default, can be omitted
}
```

```js
const el = document.querySelector('my-widget');
el.remove();  // Component destroyed after microtask
```

**Result:**
- State cleared
- Effects stopped
- Memory freed
- Reconnect creates new instance

### Keep Alive (keepAlive: true)

```js
{
  keepAlive: true
}
```

```js
const el = document.querySelector('my-widget');
el.remove();  // Component stays alive
// State preserved
document.body.appendChild(el);  // Reconnects with same state
```

**Result:**
- State preserved
- Effects continue running
- Memory retained
- Reconnect uses same instance

### Use Cases

**Keep Alive:**
- Expensive initialization
- Temporary hide/show (`display: none`)
- Moving elements between containers
- SPA route caching
- Form state preservation

**Standard (destroy on disconnect):**
- Standard components
- Memory-sensitive scenarios
- Clean state requirements
- One-time use components

### Manual Cleanup

With `keepAlive: true`, call `unmount()` explicitly:

```js
const el = document.querySelector('my-widget');

// Remove from DOM (still alive)
el.remove();

// Later: destroy explicitly
el.unmount();  // Now destroyed
```

### Performance Trade-offs

| Aspect | keepAlive: false | keepAlive: true |
|--------|------------------|-----------------|
| Memory | Lower ✅ | Higher ❌ |
| Reconnect Speed | Slower (rebuild) | Faster (reuse) ✅ |
| State | Clean on reconnect ✅ | Preserved ✅ |
| Cleanup | Automatic ✅ | Manual required ❌ |

---

## Complete Example

Combining all options:

```js
JQRE.defineElement('feature-panel', {
  props: {
    variant: 'default',
    disabled: false,
    userId: null,
    config: {}
  },
  state: {
    loading: false,
    expanded: false
  },
  methods: {
    toggle() {
      this.expanded = !this.expanded;
    },
    refresh() {
      this.loading = true;
      // ... refresh logic
    },
    reset() {
      this.expanded = false;
      this.loading = false;
    }
  },
  template: `
    <div>
      <div $:click="toggle">
        <slot name="header"></slot>
      </div>
      <div $:if="expanded">
        <slot></slot>
      </div>
    </div>
  `
}, {
  // Type coercion
  attributes: {
    userId: Number,
    disabled: Boolean,
    config: 'json'
  },
  
  // Reflect for CSS styling
  reflect: ['variant', 'disabled', 'loading', 'expanded'],
  
  // Public API
  expose: ['toggle', 'refresh', 'reset'],
  
  // Lazy loading
  mount: 'manual',
  
  // Layout
  display: 'block',
  
  // State persistence
  keepAlive: true
});
```

**CSS Styling:**
```css
feature-panel[disabled] {
  opacity: 0.5;
  pointer-events: none;
}

feature-panel[variant="primary"] {
  border-color: blue;
}

feature-panel[loading] {
  cursor: wait;
}

feature-panel[expanded] {
  /* ... */
}
```

**Usage:**
```html
<feature-panel 
  variant="primary"
  user-id="123"
  config='{"theme":"dark"}'>
  
  <h3 slot="header">Panel Title</h3>
  <p>Panel content</p>
</feature-panel>

<script>
  const panel = document.querySelector('feature-panel');
  
  // Wait to mount (manual mode)
  panel.mount();
  
  // Use exposed methods
  panel.toggle();
  panel.refresh();
  
  // Access instance
  console.log(panel.$instance.$state.expanded);
</script>
```

---

## Option Precedence

When options conflict or interact:

### Attributes vs Inference

Explicit `attributes` override inference:

```js
props: {
  value: null  // Would infer String
}
attributes: {
  value: Number  // Explicit wins
}
```

### Reflect + keepAlive

Both work together:

```js
{
  reflect: ['count'],
  keepAlive: true
}
// Reflected attributes update even when disconnected
```

### Mount + keepAlive

Independent options:

```js
{
  mount: 'manual',  // How it mounts
  keepAlive: true   // What happens on disconnect
}
```

---

## TypeScript

Full type safety:

```typescript
import { ElementOptions, AttributeType } from '@en32/jqr/element';

const options: ElementOptions = {
  attributes: {
    userId: Number,
    config: 'json' as const  // Explicit literal type
  },
  reflect: ['disabled', 'variant'],
  expose: ['open', 'close'],
  mount: 'auto',
  display: 'block',
  keepAlive: false
};
```

---

## Best Practices

### Attributes

✅ **Use inference** when props have typed defaults  
✅ **Explicit config** for `null` defaults or override  
❌ **Don't over-configure** - inference handles most cases

### Reflect

✅ **Reflect** props needed for CSS or DOM queries  
✅ **Keep list short** for performance  
❌ **Don't reflect** internal state or computed values

### Expose

✅ **Expose** public API methods only  
✅ **Document** exposed methods clearly  
❌ **Don't expose** internal utilities or private methods

### Mount

✅ **Use auto** for standard components  
✅ **Use manual** for heavy/lazy components  
❌ **Don't forget** to call `mount()` for manual mode

### Display

✅ **Set display** to avoid browser default `inline`  
✅ **Match** semantic meaning (block for panels)  
❌ **Don't skip** - explicit is better than default

### keepAlive

✅ **Use sparingly** - only for expensive state  
✅ **Remember** to call `unmount()` for cleanup  
❌ **Don't use** by default - standard behavior is safer

---

## See Also

- [Element Options Guide](../guides/options.md) - Detailed explanations with examples
- [API Reference](./api-reference.md) - Complete API documentation
- [Defining Elements](../guides/defining-elements.md) - Getting started
- [Element API](../guides/element-api.md) - Runtime API
