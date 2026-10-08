<!-- {
  "path": "jqr-e/guides/defining-elements",
  "name": "docs-jqr-e-guides-defining-elements",
  "meta": {
    "title": "Defining Elements",
    "order": 1
  }
} -->

# Defining Custom Elements

Learn how to transform JQR components into native custom elements using the `defineElement()` API.

## Overview

JQR-E provides three main functions for defining custom elements:

- **`defineElement(name, def?, options?)`** - Define a single custom element
- **`defineElements(defs, options?)`** - Define multiple elements at once
- **`defineRegistered(options?)`** - Auto-define all registered components

## Basic Usage

### defineElement()

The primary function for creating custom elements:

```js
JQRE.defineElement(name, definition, options);
```

**Parameters:**
- `name` (string) - Tag name (must contain a hyphen, lowercase)
- `definition` (object, optional) - Component definition (template or render-based)
- `options` (object, optional) - Element configuration

**Returns:** `CustomElementConstructor`

### Simple Example

```html live:no-code
<div id="demo-define-basic">
  <div class="en-card">
    <greeting-card></greeting-card>
  </div>
</div>
```

```html
<div id="demo-define-basic">
  <greeting-card></greeting-card>
</div>
```

```js inject show
// Define custom element with inline definition
JQRE.defineElement('greeting-card', {
  state: { 
    greeting: 'Hello', 
    name: 'World' 
  },
  template: `
    <div>
      <h3 $:text="greeting + ', ' + name + '!'"></h3>
      <p>This is a custom element! 🎉</p>
    </div>
  `
});
```

```js cleanup
onUnmount(() => {
  $('#demo-define-basic').component('destroy');
});
```

## Definition Patterns

### Pattern 1: Inline Definition

Provide the component definition directly:

```js
JQRE.defineElement('user-badge', {
  props: {
    username: '',
    role: 'user'
  },
  template: `
    <div class="badge">
      <span $:text="username"></span>
      <small $:text="'(' + role + ')'"></small>
    </div>
  `
});
```

### Pattern 2: Pre-registered Component

Register first, then define element (useful for organizing code):

```js
// Step 1: Register component
JQR.register('user-badge', {
  props: { username: '', role: 'user' },
  template: `...`
});

// Step 2: Define as custom element (reuses registered definition)
JQRE.defineElement('user-badge');
```

### Pattern 3: Separate Definition Object

Keep definition as reusable object:

```js
const userBadgeDef = {
  props: { username: '', role: 'user' },
  template: `...`
};

// Use in multiple places
JQR.register('user-badge', userBadgeDef);
JQRE.defineElement('user-badge', userBadgeDef);
```

## Template-Based Components

Components using HTML templates:

```html live:no-code
<div id="demo-define-template">
  <div class="en-card">
    <counter-display></counter-display>
  </div>
</div>
```

```html
<div id="demo-define-template">
  <counter-display></counter-display>
</div>
```

```js inject show
JQRE.defineElement('counter-display', {
  state: { count: 0 },
  computed: {
    message() {
      return this.count === 0 
        ? 'Click to start' 
        : `Clicked ${this.count} time${this.count === 1 ? '' : 's'}`;
    }
  },
  methods: {
    increment() {
      this.count++;
    }
  },
  template: `
    <div>
      <p $:text="message"></p>
      <button $:click="increment()">Click Me</button>
    </div>
  `
});
```

```js cleanup
onUnmount(() => {
  $('#demo-define-template').component('destroy');
});
```

### Template Resolution

Templates are resolved in this order:

1. **`definition.template`** - Inline template string
2. **`<template id="tag-name">`** - Template element in page
3. **Element's own markup** - Content inside custom element tag

```html
<!-- Option 1: Inline template (shown above) -->

<!-- Option 2: Template element -->
<template id="my-card">
  <div class="card">
    <slot></slot>
  </div>
</template>

<script>
  JQRE.defineElement('my-card', {
    // template: id matches automatically
  });
</script>

<!-- Option 3: Inline markup -->
<my-card>
  <p>This becomes the template</p>
</my-card>
```

## Render-Based Components

Components using `render()` function from JQR-H:

```html live:no-code
<div id="demo-define-render">
  <div class="en-card en-card__body">
    <dynamic-counter></dynamic-counter>
  </div>
</div>
```

```html
<div id="demo-define-render">
  <dynamic-counter></dynamic-counter>
</div>
```

```js inject show
JQRE.defineElement('dynamic-counter', {
  state: { count: 0 },
  render: (h, ctx) => {
    // const { count } = ctx;
    return h('div', {}, [
      h('p', {}, [
        h('strong', {}, ['Count: ']),
        h('span', { text: () => ctx.count }, [])
      ]),
      h('div', { class: 'en-flex' }, [
        h('button', {
          class: 'en-btn',
          on: { click: () => ctx.count-- }
        }, ['-']),
        h('button', {
          class: 'en-btn',
          on: { click: () => ctx.count++ }
        }, ['+'])
      ])
    ]);
  }
}, {
  display: 'block'
});
```

```js cleanup
onUnmount(() => {
  $('#demo-define-render').component('destroy');
});
```

**Note:** Render-based components require `jqr.h.js` to be loaded.

## defineElements()

Define multiple elements at once:

```js
JQRE.defineElements(definitions, commonOptions);
```

**Parameters:**
- `definitions` (object) - Map of tag names to definitions
- `commonOptions` (object, optional) - Options applied to all elements
  - Can include `prefix` to prepend to all tag names

### Batch Definition

```js
JQRE.defineElements({
  'en-alert': {
    props: { type: 'info', message: '' },
    template: `<div class="alert" $:class="'alert--' + type" $:text="message"></div>`
  },
  'en-badge': {
    props: { text: '', variant: 'default' },
    template: `<span class="badge" $:class="'badge--' + variant" $:text="text"></span>`
  },
  'app-spinner': {
    state: { loading: true },
    template: `<div class="spinner" $:if="loading"></div>`
  }
});
```

### With Prefix

Use prefix to namespace your components:

```js
// Define with 'app-' prefix
JQRE.defineElements({
  'button': buttonDef,
  'input': inputDef,
  'card': cardDef
}, {
  prefix: 'app'
});

// Creates: <app-button>, <app-input>, <app-card>
```

**Practical Example:**

```html live:no-code
<div id="demo-define-multiple">
  <div class="en-flex en-gap-3">
    <app-badge text="New" variant="primary"></app-badge>
    <app-badge text="Hot" variant="danger"></app-badge>
  </div>
</div>
```

```html
<div id="demo-define-multiple">
  <app-badge text="New" variant="primary"></app-badge>
  <app-badge text="Hot" variant="danger"></app-badge>
</div>
```

```js inject show
JQRE.defineElements({
  'badge': {
    props: { text: '', variant: 'default' },
    template: `
      <span $:class="'en-badge en-badge--' + $props.variant">
        <span $:text="$props.text"></span>
      </span>
    `
  }
}, {
  prefix: 'app',
  display: 'inline-block'
});
```

```js cleanup
onUnmount(() => {
  $('#demo-define-multiple').component('destroy');
});
```

## defineRegistered()

Auto-define all registered components as custom elements:

```js
JQRE.defineRegistered(options);
```

This scans the component registry and defines custom elements for all valid tag names (names with hyphens).

**Parameters:**
- `options` (object, optional) - Options applied to all elements

### Example

```js
// Register components
JQR.register('my-button', buttonDef);
JQR.register('my-input', inputDef);
JQR.register('my-card', cardDef);

// Define all at once
JQRE.defineRegistered({
  display: 'block',
  reflect: ['disabled', 'loading']
});

// Now all registered components with hyphenated names are custom elements
```

### Filtering

Only components with valid tag names (containing hyphen) are defined:

```js
JQR.register('button', buttonDef);      // ❌ Skipped (no hyphen)
JQR.register('my-button', buttonDef);   // ✅ Defined
JQR.register('MyButton', buttonDef);    // ❌ Skipped (uppercase)
```

## Tag Name Rules

Custom element names must follow Web Components standards:

### Valid Names ✅

```js
JQRE.defineElement('my-button', def);        // ✅ Simple hyphen
JQRE.defineElement('app-user-card', def);    // ✅ Multiple hyphens
JQRE.defineElement('x-foo', def);            // ✅ Short names
```

### Invalid Names ❌

```js
JQRE.defineElement('button', def);           // ❌ No hyphen
JQRE.defineElement('MyButton', def);         // ❌ Uppercase
JQRE.defineElement('my_button', def);        // ❌ Underscore not hyphen
JQRE.defineElement('font-face', def);        // ❌ Reserved name
```

**Reserved Names:** 
Standard HTML element names with hyphens are reserved:
`annotation-xml`, `color-profile`, `font-face`, `font-face-src`, `font-face-uri`, `font-face-format`, `font-face-name`, `missing-glyph`

## Element Options

Configure element behavior with options:

```js
JQRE.defineElement('my-button', definition, {
  // Attribute type coercion
  attributes: {
    count: Number,
    disabled: Boolean,
    data: 'json'
  },
  
  // Reflect props back to attributes
  reflect: ['disabled', 'active'],
  
  // Expose methods on element
  expose: ['focus', 'click'],
  
  // CSS display value
  display: 'inline-block',
  
  // Mounting behavior
  mount: 'auto' // or 'manual'
});
```

See [Element Options](./options.md) for detailed documentation.

## Idempotency

`defineElement()` is **idempotent** - calling it multiple times with the same name is safe:

```js
// First call: defines element
JQRE.defineElement('my-button', defV1);

// Second call: ignored (already defined)
JQRE.defineElement('my-button', defV1);

// Different definition: warning in dev mode, ignored in production
JQRE.defineElement('my-button', defV2); // ⚠️ Dev warning
```

This is useful for:
- Hot Module Replacement (HMR)
- Module re-imports
- Conditional definitions

## Complete Example

Putting it all together:

```html live:no-code
<div id="demo-define-complete">
  <div class="en-card en-card__body">
    <color-picker color="blue"></color-picker>
  </div>
</div>
```

```html
<div id="demo-define-complete">
  <color-picker color="blue"></color-picker>
</div>
```

```js inject show
JQRE.defineElement('color-picker', {
  props: {
    color: 'red'
  },
  state: {
    colors: ['red', 'green', 'blue', 'yellow', 'purple']
  },
  computed: {
    colorStyle() {
      return { backgroundColor: this.$props.color };
    }
  },
  methods: {
    selectColor(newColor) {
      this.$setProps({ color: newColor });
      this.$emit('change', newColor);
    }
  },
  template: `
    <div>
      <div style="width: 100px; height: 100px; border: 2px solid #ccc; border-radius: 4px; margin-bottom: 10px" $:style="colorStyle"></div>
      <div style="display: flex; gap: 5px">
        <button 
          $:for="c in colors" 
          $:key="c"
          $:click="selectColor(c)"
          $:style="{ backgroundColor: c, width: '30px', height: '30px', border: $props.color === c ? '2px solid black' : '1px solid #ccc', cursor: 'pointer', borderRadius: '4px' }">
        </button>
      </div>
    </div>
  `
}, {
  reflect: ['color'],
  display: 'block'
});

// Listen to changes
$('#demo-define-complete color-picker').on('jqr:change', function(e, color) {
  console.log('Color changed to:', color);
});
```

```js cleanup
onUnmount(() => {
  $('#demo-define-complete').component('destroy');
});
```

## TypeScript Support

JQR-E includes full TypeScript support:

```ts
import { defineElement, ElementOptions } from '@en32/jqr/element';
import type { ComponentOptions } from '@en32/jqr';

interface ButtonProps {
  variant: 'default' | 'primary' | 'danger';
  disabled: boolean;
}

const buttonDef: ComponentOptions<{}, ButtonProps> = {
  props: {
    variant: 'default',
    disabled: false
  },
  template: `...`
};

const options: ElementOptions = {
  reflect: ['disabled'],
  expose: ['focus'],
  display: 'inline-block'
};

defineElement('my-button', buttonDef, options);
```

## Best Practices

### ✅ Do

- **Use descriptive names** with clear purpose: `user-card`, `date-picker`
- **Define elements once** at app initialization
- **Use prefix** for component libraries to avoid conflicts
- **Provide default values** for all props
- **Document emitted events** in definition

### ❌ Don't

- **Don't define elements conditionally** (can cause conflicts)
- **Don't use generic names** without prefix: `card`, `button`
- **Don't redefine** elements with different definitions
- **Don't forget hyphens** in tag names

## Common Patterns

### Lazy Definition

Define elements only when needed:

```js
function defineMyComponents() {
  if (customElements.get('my-button')) return; // Already defined
  
  JQRE.defineElements({
    'button': buttonDef,
    'input': inputDef
  }, { prefix: 'my' });
}

// Call when needed
document.addEventListener('DOMContentLoaded', defineMyComponents);
```

### Library Pattern

Export definitions for consumers:

```js
// my-components.js
export const buttonDef = { /* ... */ };
export const inputDef = { /* ... */ };

export function defineAll(prefix = 'app') {
  JQRE.defineElements({
    'button': buttonDef,
    'input': inputDef
  }, { prefix });
}

// Consumer
import { defineAll } from './my-components.js';
defineAll('custom');
// Creates: <custom-button>, <custom-input>
```

## Debugging

### Check if Element is Defined

```js
// Check browser registry
console.log(customElements.get('my-button')); // CustomElementConstructor or undefined

// Check JQR element
console.log(JQRE.isJqrElement(document.querySelector('my-button'))); // true
```

### View Registered Components

```js
// All registered components (JQR core)
console.log(JQR.components);
```

## Next Steps

- **[Attributes & Props](./attributes-and-props.md)** - Learn about type coercion and property binding
- **[Events](./events.md)** - Emit and handle custom events
- **[Slots](./slots.md)** - Use slots for content projection
- **[Lifecycle](./lifecycle.md)** - Understand mounting and ownership

## Related

- [Element Options Reference](../reference/options-reference.md)
- [API Reference](../reference/api-reference.md)
- [Button Component Example](../examples/button-component.md)
