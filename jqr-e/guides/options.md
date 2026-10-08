<!-- {
  "path": "jqr-e/guides/options",
  "name": "docs-jqr-e-guides-options",
  "meta": {
    "title": "Element Options",
    "order": 7
  }
} -->

# Element Options

Configure custom element behavior with `ElementOptions` to control attributes, reflection, mounting, and more.

## Overview

Element options are passed as the third parameter to `defineElement()`:

```js
JQRE.defineElement('my-button', definition, {
  attributes: { /* ... */ },
  reflect: [ /* ... */ ],
  expose: [ /* ... */ ],
  mount: 'auto',
  display: 'inline-block',
  keepAlive: false
});
```

## Available Options

| Option | Type | Default | Description |
|--------|------|---------|-------------|
| `attributes` | `object` | inferred | Attribute type coercion per prop |
| `reflect` | `string[]` | `[]` | Props to mirror back to attributes |
| `expose` | `string[]` | `[]` | Methods exposed on element |
| `mount` | `'auto' \| 'manual'` | `'auto'` | Mounting strategy |
| `display` | `string` | none | CSS display value |
| `keepAlive` | `boolean` | `false` | Keep component alive when disconnected |

## attributes

Control how HTML attribute strings are converted to typed prop values.

### Type Signature

```typescript
attributes?: {
  [propName: string]: AttributeType
}

type AttributeType = StringConstructor | NumberConstructor | BooleanConstructor | 'json'
```

### Coercion Types

| Type | Behavior | Example |
|------|----------|---------|
| `String` | Use as-is | `"hello"` → `"hello"` |
| `Number` | Parse with `Number()` | `"123"` → `123` |
| `Boolean` | Presence = true | `disabled` → `true` |
| `'json'` | Parse with `JSON.parse()` | `'[1,2]'` → `[1,2]` |

### Basic Usage

```html live:no-code
<div id="demo-options-attributes">
  <div class="en-card">
    <typed-element count="42" enabled items='["a","b","c"]'></typed-element>
  </div>
</div>
```

```html
<div id="demo-options-attributes">
  <typed-element 
    count="42" 
    enabled 
    items='["a","b","c"]'>
  </typed-element>
</div>
```

```js inject show
JQRE.defineElement('typed-element', {
  props: {
    count: 0,
    enabled: false,
    items: []
  },
  computed: {
    summary() {
      return `Count: ${this.count} (${typeof this.count}), Enabled: ${this.enabled} (${typeof this.enabled}), Items: ${this.items.length}`;
    }
  },
  template: `
    <div>
      <p $:text="summary"></p>
      <ul $:if="items.length > 0">
        <li $:for="item in items" $:key="item" $:text="item"></li>
      </ul>
    </div>
  `
}, {
  attributes: {
    count: Number,      // "42" → 42
    enabled: Boolean,   // presence → true
    items: 'json'       // '["a","b","c"]' → ["a","b","c"]
  }
});
```

```js cleanup
onUnmount(() => {
  $('#demo-options-attributes').component('destroy');
});
```

### Type Inference

When `attributes` is omitted, types are inferred from default values:

```js
JQRE.defineElement('auto-typed', {
  props: {
    count: 0,           // Inferred: Number
    name: '',           // Inferred: String
    visible: true,      // Inferred: Boolean
    data: {}            // Inferred: 'json'
  }
});
// No attributes option needed!
```

**Inference Rules:**
- `typeof default === 'number'` → `Number`
- `typeof default === 'boolean'` → `Boolean`
- `Array.isArray(default)` → `'json'`
- `typeof default === 'object'` → `'json'`
- Otherwise → `String`

### Explicit Override

Override inference when needed:

```js
JQRE.defineElement('override-types', {
  props: {
    userId: null,       // Default: String (null → String)
    config: null        // Default: String (null → String)
  }
}, {
  attributes: {
    userId: Number,     // Override: Number
    config: 'json'      // Override: JSON
  }
});
```

## reflect

Mirror internal prop changes back to HTML attributes.

### Type Signature

```typescript
reflect?: string[]
```

### Basic Usage

```html live:no-code
<div id="demo-options-reflect">
  <style>
    reflect-demo[disabled] { opacity: 0.5; pointer-events: none; }
    reflect-demo[variant="primary"] { color: #007bff; }
    reflect-demo[variant="danger"] { color: #dc3545; }
  </style>
  <div class="en-card">
    <reflect-demo variant="primary"></reflect-demo>
    <div class="en-flex en-gap-2" style="margin-top: 10px">
      <button class="en-btn en-btn--sm" onclick="
        const el = document.querySelector('#demo-options-reflect reflect-demo');
        el.$instance.$state.variant = 'danger';
      ">Set Danger</button>
      <button class="en-btn en-btn--sm" onclick="
        const el = document.querySelector('#demo-options-reflect reflect-demo');
        el.$instance.$state.disabled = !el.$instance.$state.disabled;
      ">Toggle Disabled</button>
    </div>
  </div>
</div>
```

```html
<style>
  reflect-demo[disabled] { opacity: 0.5; }
  reflect-demo[variant="primary"] { color: blue; }
  reflect-demo[variant="danger"] { color: red; }
</style>

<div id="demo-options-reflect">
  <reflect-demo variant="primary"></reflect-demo>
</div>
```

```js inject show
JQRE.defineElement('reflect-demo', {
  props: {
    variant: 'default',
    disabled: false
  },
  template: `
    <div>
      <strong>Status: </strong>
      <span $:text="disabled ? 'Disabled' : variant"></span>
    </div>
  `
}, {
  reflect: ['variant', 'disabled'],
  display: 'block'
});
```

```js cleanup
onUnmount(() => {
  $('#demo-options-reflect').component('destroy');
});
```

**When prop changes internally:**
```js
const el = document.querySelector('reflect-demo');
el.$instance.$state.variant = 'danger';
// → Attribute updates: variant="danger"
```

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
const disabled = document.querySelectorAll('my-button[disabled]');

// Find primary variant
const primary = document.querySelectorAll('my-button[variant="primary"]');
```

**Debugging:**
```js
// Visual inspection in DevTools
<my-button disabled variant="primary">
  <!-- Attributes visible in inspector -->
</my-button>
```

### Boolean Reflection

Boolean props reflect as presence/absence:

```js
JQRE.defineElement('toggle-btn', {
  props: {
    active: false
  }
}, {
  reflect: ['active']
});

// active: true  → <toggle-btn active>
// active: false → <toggle-btn>
```

### Performance Note

Reflection adds small overhead (writes to DOM). Only reflect props needed for:
- CSS styling
- External DOM queries
- Visual debugging

## expose

Make component methods accessible directly on the element.

### Type Signature

```typescript
expose?: string[]
```

### Basic Usage

```html live:no-code
<div id="demo-options-expose">
  <div class="en-card">
    <exposed-methods></exposed-methods>
    <div class="en-flex en-gap-2" style="margin-top: 10px">
      <button class="en-btn en-btn--sm" onclick="document.querySelector('#demo-options-expose exposed-methods').open()">
        Call open()
      </button>
      <button class="en-btn en-btn--sm" onclick="document.querySelector('#demo-options-expose exposed-methods').close()">
        Call close()
      </button>
      <button class="en-btn en-btn--sm" onclick="document.querySelector('#demo-options-expose exposed-methods').toggle()">
        Call toggle()
      </button>
    </div>
  </div>
</div>
```

```html
<div id="demo-options-expose">
  <exposed-methods></exposed-methods>
  
  <button onclick="document.querySelector('exposed-methods').open()">
    Open
  </button>
  <button onclick="document.querySelector('exposed-methods').close()">
    Close
  </button>
</div>
```

```js inject show
JQRE.defineElement('exposed-methods', {
  state: {
    isOpen: false
  },
  methods: {
    open() {
      this.isOpen = true;
      console.log('Opened!');
    },
    close() {
      this.isOpen = false;
      console.log('Closed!');
    },
    toggle() {
      this.isOpen = !this.isOpen;
    }
  },
  template: `
    <div>
      <p><strong>Status:</strong> <span $:text="isOpen ? 'Open' : 'Closed'"></span></p>
    </div>
  `
}, {
  expose: ['open', 'close', 'toggle']
});
```

```js cleanup
onUnmount(() => {
  $('#demo-options-expose').component('destroy');
});
```

**Direct method calls:**
```js
const el = document.querySelector('exposed-methods');
el.open();    // Calls component method
el.close();   // Calls component method
el.toggle();  // Calls component method
```

### Without expose

Methods require `$instance` access:

```js
const el = document.querySelector('my-dialog');

// ❌ Without expose
el.$instance.$ctx.open();

// ✅ With expose
el.open();
```

### Common Patterns

**Form Controls:**
```js
JQRE.defineElement('custom-input', definition, {
  expose: ['focus', 'blur', 'select', 'clear']
});

const input = document.querySelector('custom-input');
input.focus();
input.select();
```

**Dialogs:**
```js
JQRE.defineElement('app-dialog', definition, {
  expose: ['open', 'close']
});

const dialog = document.querySelector('app-dialog');
dialog.open();
```

**Data Widgets:**
```js
JQRE.defineElement('data-table', definition, {
  expose: ['refresh', 'export', 'filter']
});

const table = document.querySelector('data-table');
table.refresh();
table.export('csv');
```

### Method Safety

Exposed methods check mounting status:

```js
const el = document.querySelector('my-widget');

// Before mount
el.myMethod();  // ⚠️ Warning: not mounted

await el.$mounted;

// After mount
el.myMethod();  // ✅ Works
```

## mount

Control when the element mounts its component.

### Type Signature

```typescript
mount?: 'auto' | 'manual'
```

- **`'auto'`** (default) - Mount automatically when connected
- **`'manual'`** - Require explicit `mount()` call

### Auto Mount (Default)

```js
JQRE.defineElement('auto-widget', definition, {
  mount: 'auto'  // Default, can be omitted
});
```

```html
<auto-widget></auto-widget>
<!-- Mounts automatically when added to DOM -->
```

### Manual Mount

```html live:no-code
<div id="demo-options-mount">
  <div class="en-card">
    <div class="en-flex en-gap-2">
      <button class="en-btn en-btn--sm" onclick="document.querySelector('#demo-options-mount manual-widget').mount()">
        Mount Now
      </button>
      <button class="en-btn en-btn--sm" onclick="document.querySelector('#demo-options-mount manual-widget').unmount()">
        Unmount
      </button>
    </div>
    <manual-widget style="display: block; margin-top: 10px"></manual-widget>
  </div>
</div>
```

```html
<div id="demo-options-mount">
  <manual-widget></manual-widget>
  
  <button onclick="document.querySelector('manual-widget').mount()">
    Mount
  </button>
</div>
```

```js inject show
JQRE.defineElement('manual-widget', {
  state: {
    loadTime: null
  },
  mounted() {
    this.loadTime = new Date().toLocaleTimeString();
  },
  template: `
    <div class="en-alert en-alert--success">
      <p>Loaded at: <strong $:text="loadTime || 'Not loaded'"></strong></p>
    </div>
  `
}, {
  mount: 'manual'
});
```

```js cleanup
onUnmount(() => {
  $('#demo-options-mount').component('destroy');
});
```

**Requires explicit mount:**
```js
const el = document.querySelector('manual-widget');
el.mount();  // Now it mounts
```

### Use Cases

**Lazy Loading:**
```js
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

**Conditional Initialization:**
```js
if (userHasPermission) {
  document.querySelector('admin-panel').mount();
}
```

**Deferred Loading:**
```js
window.addEventListener('load', () => {
  setTimeout(() => {
    document.querySelectorAll('[data-defer]').forEach(el => {
      el.mount();
    });
  }, 1000);
});
```

## display

Set default CSS `display` value for the element.

### Type Signature

```typescript
display?: string
```

### Basic Usage

```js
JQRE.defineElement('inline-badge', definition, {
  display: 'inline-block'
});

JQRE.defineElement('block-panel', definition, {
  display: 'block'
});

JQRE.defineElement('flex-container', definition, {
  display: 'flex'
});
```

**Generated CSS:**
```css
inline-badge { display: inline-block; }
block-panel { display: block; }
flex-container { display: flex; }
```

### Default Behavior

Without `display` option:

```js
JQRE.defineElement('no-display', definition);
// Uses browser default: display: inline
```

### Common Values

| Value | Use Case |
|-------|----------|
| `'inline-block'` | Buttons, badges, icons |
| `'block'` | Panels, cards, sections |
| `'flex'` | Containers with flexbox layout |
| `'grid'` | Grid-based layouts |
| `'inline-flex'` | Inline flex containers |
| `'none'` | Initially hidden elements |

### Example

```html live:no-code
<div id="demo-options-display">
  <div class="en-card">
    <p>
      This is <display-inline>inline</display-inline> text.
      <display-block>This is block level.</display-block>
      More inline text.
    </p>
  </div>
</div>
```

```html
<div id="demo-options-display">
  <p>
    This is <display-inline>inline</display-inline> text.
    <display-block>This is block level.</display-block>
    More text.
  </p>
</div>
```

```js inject show
JQRE.defineElement('display-inline', {
  template: `<span style="background: #e3f2fd; padding: 2px 6px; border-radius: 3px"><slot></slot></span>`
}, {
  display: 'inline'  // Stays inline
});

JQRE.defineElement('display-block', {
  template: `<div style="background: #fff3e0; padding: 10px; margin: 5px 0; border-radius: 4px"><slot></slot></div>`
}, {
  display: 'block'  // Takes full width
});
```

```js cleanup
onUnmount(() => {
  $('#demo-options-display').component('destroy');
});
```

### Style Injection

The `display` rule is injected once as a `<style>` element:

```html
<style data-jqr-element>
inline-badge { display: inline-block; }
block-panel { display: block; }
</style>
```

## keepAlive

Keep component alive when element is disconnected from DOM.

### Type Signature

```typescript
keepAlive?: boolean
```

- **`false`** (default) - Destroy component on disconnect
- **`true`** - Keep component alive when disconnected

### Default Behavior (keepAlive: false)

```html live:no-code
<div id="demo-options-keepalive-default">
  <div class="en-card">
    <button class="en-btn en-btn--sm" onclick="
      const container = document.getElementById('demo-options-keepalive-default').querySelector('.demo-container');
      const el = container.querySelector('default-widget');
      if (el) {
        el.remove();
        setTimeout(() => container.appendChild(el), 1000);
      }
    ">Remove & Re-add (State Lost)</button>
    <div class="demo-container" style="margin-top: 10px">
      <default-widget></default-widget>
    </div>
  </div>
</div>
```

```html
<div id="demo-options-keepalive-default">
  <button onclick="
    const el = document.querySelector('default-widget');
    el.remove();
    setTimeout(() => document.body.appendChild(el), 1000);
  ">Remove & Re-add</button>
  
  <default-widget></default-widget>
</div>
```

```js inject show
JQRE.defineElement('default-widget', {
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
}, {
  keepAlive: false  // Destroyed on disconnect
});
```

```js cleanup
onUnmount(() => {
  $('#demo-options-keepalive-default').component('destroy');
});
```

**Result:** State resets to initial value (count: 0)

### keepAlive: true

```html live:no-code
<div id="demo-options-keepalive-true">
  <div class="en-card">
    <button class="en-btn en-btn--sm" onclick="
      const container = document.getElementById('demo-options-keepalive-true').querySelector('.demo-container');
      const el = container.querySelector('persistent-widget');
      if (el) {
        el.remove();
        setTimeout(() => container.appendChild(el), 1000);
      }
    ">Remove & Re-add (State Kept!)</button>
    <div class="demo-container" style="margin-top: 10px">
      <persistent-widget></persistent-widget>
    </div>
  </div>
</div>
```

```html
<div id="demo-options-keepalive-true">
  <button onclick="
    const el = document.querySelector('persistent-widget');
    el.remove();
    setTimeout(() => document.body.appendChild(el), 1000);
  ">Remove & Re-add</button>
  
  <persistent-widget></persistent-widget>
</div>
```

```js inject show
JQRE.defineElement('persistent-widget', {
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
      <p>Count: <strong $:text="count"></strong> (survives disconnect)</p>
      <button class="en-btn en-btn--sm" $:click="increment">+1</button>
    </div>
  `
}, {
  keepAlive: true  // Survives disconnect
});
```

```js cleanup
onUnmount(() => {
  $('#demo-options-keepalive-true').component('destroy');
});
```

**Result:** State preserved across disconnect/reconnect

### Manual Cleanup

With `keepAlive: true`, call `unmount()` explicitly:

```js
const el = document.querySelector('persistent-widget');

// Remove from DOM (component still alive)
el.remove();

// Later: destroy explicitly
el.unmount();  // Now destroyed
```

### Use Cases

**Temporary Hide/Show:**
```js
// Hide without losing state
widget.style.display = 'none';

// Or remove from DOM (with keepAlive)
widget.remove();
```

**Moving Elements:**
```js
// Move between containers (state preserved)
containerA.removeChild(widget);
containerB.appendChild(widget);
```

**SPA Route Transitions:**
```js
// Cache widgets across route changes
router.on('leave', () => {
  widgets.forEach(w => w.remove());  // State kept
});

router.on('enter', () => {
  widgets.forEach(w => container.appendChild(w));  // State restored
});
```

### Performance Trade-offs

**keepAlive: false (default)**
- ✅ Lower memory usage
- ✅ Clean state on remount
- ❌ Slower remount (rebuild component)

**keepAlive: true**
- ✅ Faster reconnect (no rebuild)
- ✅ State persists
- ❌ Higher memory usage (component stays in memory)

## Combining Options

Use multiple options together:

```js
JQRE.defineElement('feature-card', {
  props: {
    variant: 'default',
    disabled: false,
    count: 0
  },
  methods: {
    reset() {
      this.$setProps({ count: 0 });
    },
    increment() {
      this.$setProps({ count: this.count + 1 });
    }
  },
  template: `
    <div>
      <p>Count: <span $:text="count"></span></p>
      <button $:click="increment" $:disabled="disabled">+</button>
    </div>
  `
}, {
  // Type coercion
  attributes: {
    count: Number,
    disabled: Boolean
  },
  
  // CSS styling via attributes
  reflect: ['variant', 'disabled'],
  
  // Public API
  expose: ['reset', 'increment'],
  
  // Deferred mount
  mount: 'manual',
  
  // Layout
  display: 'block',
  
  // State persistence
  keepAlive: false
});
```

## Best Practices

### ✅ Do

- **Infer types when possible** - Let JQR figure out attribute types from defaults
- **Reflect sparingly** - Only props needed for CSS or DOM queries
- **Expose minimal API** - Only methods users need to call
- **Use `keepAlive` selectively** - Only for expensive state
- **Set `display` for layout** - Avoid browser's default `inline`

### ❌ Don't

- **Don't reflect everything** - Adds DOM write overhead
- **Don't expose internal methods** - Keep implementation private
- **Don't use `manual` by default** - Only for lazy loading
- **Don't forget `unmount()`** - Required for `keepAlive` cleanup
- **Don't over-configure** - Start simple, add options as needed

## TypeScript Support

Full type safety for options:

```typescript
import { defineElement, ElementOptions } from '@en32/jqr/element';

const options: ElementOptions = {
  attributes: {
    userId: Number,
    config: 'json'
  },
  reflect: ['disabled', 'variant'],
  expose: ['open', 'close'],
  mount: 'auto',
  display: 'block',
  keepAlive: false
};

defineElement('my-widget', definition, options);
```

## Common Patterns

### Button Component

```js
JQRE.defineElement('app-button', {
  props: {
    variant: 'default',
    disabled: false,
    loading: false
  },
  methods: {
    click() {
      if (!this.disabled && !this.loading) {
        this.$emit('click');
      }
    }
  }
}, {
  attributes: {
    disabled: Boolean,
    loading: Boolean
  },
  reflect: ['disabled', 'loading', 'variant'],
  expose: ['click', 'focus'],
  display: 'inline-block'
});
```

### Dialog Component

```js
JQRE.defineElement('app-dialog', {
  props: {
    open: false
  },
  methods: {
    open() {
      this.$setProps({ open: true });
    },
    close() {
      this.$setProps({ open: false });
    }
  }
}, {
  attributes: {
    open: Boolean
  },
  reflect: ['open'],
  expose: ['open', 'close'],
  display: 'none',  // Hidden by default
  keepAlive: true   // Preserve state
});
```

### Data Table

```js
JQRE.defineElement('data-table', {
  props: {
    pageSize: 10,
    currentPage: 1
  },
  methods: {
    refresh() { /* ... */ },
    export(format) { /* ... */ },
    filter(query) { /* ... */ }
  }
}, {
  attributes: {
    pageSize: Number,
    currentPage: Number
  },
  expose: ['refresh', 'export', 'filter'],
  mount: 'manual',  // Lazy load
  display: 'block'
});
```

## Next Steps

- **[h() Integration](./h-integration.md)** - Use custom elements with render functions
- **[Element API](./element-api.md)** - Access instance and methods
- **[Lifecycle](./lifecycle.md)** - Understand mounting behavior

## Related

- [Defining Elements](./defining-elements.md)
- [Attributes & Props](./attributes-and-props.md)
- [Options Reference](../reference/options-reference.md)
