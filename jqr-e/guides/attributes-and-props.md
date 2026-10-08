<!-- {
  "path": "jqr-e/guides/attributes-and-props",
  "name": "docs-jqr-e-guides-attributes-and-props",
  "meta": {
    "title": "Attributes & Props",
    "order": 2
  }
} -->

# Attributes & Props

Learn how JQR-E handles the relationship between HTML attributes and component props, including automatic type coercion and property reflection.

## Overview

Custom elements bridge HTML attributes (strings) and JavaScript props (typed values). JQR-E automatically:

- **Observes** attributes defined in `props`
- **Coerces** attribute strings to correct types
- **Syncs** between attributes and properties
- **Reflects** props back to attributes (optional)

## Attributes vs Properties

### Attributes (HTML)

Attributes are always **strings** in HTML:

```html
<user-card 
  user-id="123"
  active="true"
  name="John Doe">
</user-card>
```

### Properties (JavaScript)

Properties are **typed values** in JavaScript:

```js
const card = document.querySelector('user-card');

card.userId;    // number: 123
card.active;    // boolean: true
card.name;      // string: "John Doe"
```

## Type Coercion

JQR-E automatically converts attribute strings to the correct prop types.

### Coercion Types

| Type | Behavior | Example |
|------|----------|---------|
| `String` | Use value as-is | `"hello"` → `"hello"` |
| `Number` | Parse with `Number()` | `"123"` → `123` |
| `Boolean` | Attribute presence | `disabled` → `true` |
| `'json'` | Parse with `JSON.parse()` | `'{"x":1}'` → `{x:1}` |

### Automatic Type Inference

Types are inferred from default values in `props`:

```js
JQRE.defineElement('my-component', {
  props: {
    count: 0,              // number (default is 0)
    disabled: false,       // boolean (default is false)
    name: '',              // string (default is '')
    items: []              // json (default is array)
  }
});
```

**Inference Rules:**
- `typeof defaultValue === 'number'` → `Number`
- `typeof defaultValue === 'boolean'` → `Boolean`
- `Array.isArray(defaultValue)` → `'json'`
- `typeof defaultValue === 'object'` → `'json'`
- Otherwise → `String`

### Explicit Type Configuration

Override inference with `attributes` option:

```js
JQRE.defineElement('my-component', {
  props: {
    userId: 0,
    config: null,
    data: {}
  }
}, {
  attributes: {
    userId: Number,       // Explicit Number
    config: 'json',       // Explicit JSON
    data: 'json'          // Explicit JSON
  }
});
```

## Boolean Attributes

Boolean attributes work like native HTML:

```html live:no-code
<div id="demo-attrs-boolean">
  <div class="en-card">
    <toggle-button disabled></toggle-button>
  </div>
</div>
```

```html
<div id="demo-attrs-boolean">
  <!-- Presence = true -->
  <toggle-button disabled></toggle-button>
  
  <!-- Absence = false -->
  <toggle-button></toggle-button>
  
  <!-- Explicit "false" = false -->
  <toggle-button disabled="false"></toggle-button>
</div>
```

```js inject show
JQRE.defineElement('toggle-button', {
  props: {
    disabled: false
  },
  state: {
    on: false
  },
  methods: {
    toggle() {
      if (!this.$props.disabled) {
        this.on = !this.on;
      }
    }
  },
  template: `
    <button 
      $:disabled="disabled"
      $:click="toggle"
      $:class="on ? 'en-btn en-btn--primary' : 'en-btn'">
      <span $:text="on ? 'ON' : 'OFF'"></span>
    </button>
  `
}, {
  reflect: ['disabled']
});
```

```js cleanup
onUnmount(() => {
  $('#demo-attrs-boolean').component('destroy');
});
```

**Rules:**
- Attribute present → `true`
- Attribute absent → `false`
- Attribute = `"false"` → `false`
- Any other value → `true`

## Number Attributes

Numbers are parsed with `Number()`:

```html live:no-code
<div id="demo-attrs-number">
  <div class="en-card">
    <progress-bar value="75" max="100"></progress-bar>
  </div>
</div>
```

```html
<div id="demo-attrs-number">
  <progress-bar value="75" max="100"></progress-bar>
</div>
```

```js inject show
JQRE.defineElement('progress-bar', {
  props: {
    value: 0,
    max: 100
  },
  computed: {
    percentage() {
      return Math.round((this.value / this.max) * 100);
    },
    widthStyle() {
      return { width: this.percentage + '%' };
    }
  },
  template: `
    <div>
      <div style="background: #e0e0e0; border-radius: 4px; overflow: hidden; height: 24px">
        <div style="background: #4caf50; height: 100%; transition: width 0.3s; display: flex; align-items: center; justify-content: center; color: white; font-size: 12px" $:style="widthStyle">
          <span $:text="percentage + '%'"></span>
        </div>
      </div>
    </div>
  `
}, {
  display: 'block'
});
```

```js cleanup
onUnmount(() => {
  $('#demo-attrs-number').component('destroy');
});
```

**Examples:**
- `value="42"` → `42`
- `value="3.14"` → `3.14`
- `value="abc"` → `NaN` (handle in component)
- `value=""` → `0`

## JSON Attributes

Complex data structures use JSON:

```html live:no-code
<div id="demo-attrs-json">
  <div class="en-card">
    <user-list users='[{"name":"Alice"},{"name":"Bob"}]'></user-list>
  </div>
</div>
```

```html
<div id="demo-attrs-json">
  <user-list users='[{"name":"Alice"},{"name":"Bob"}]'></user-list>
</div>
```

```js inject show
JQRE.defineElement('user-list', {
  props: {
    users: []
  },
  template: `
    <ul>
      <li $:for="user in $props.users" $:key="user.name">
        <span $:text="user.name"></span>
      </li>
    </ul>
  `
}, {
  attributes: {
    users: 'json'
  }
});
```

```js cleanup
onUnmount(() => {
  $('#demo-attrs-json').component('destroy');
});
```

**Important:**
- Use **single quotes** for attribute, **double quotes** for JSON
- Invalid JSON is ignored with dev warning
- Default value is used if parsing fails

## Property Access

### Getting Properties

Access props directly on the element:

```js
const el = document.querySelector('my-component');

// Read properties
console.log(el.userId);      // 123
console.log(el.userName);    // "Alice"
console.log(el.isActive);    // true
```

### Setting Properties

Set props to trigger reactive updates:

```js
const el = document.querySelector('my-component');

// Set properties (triggers reactivity)
el.userId = 456;
el.userName = "Bob";
el.isActive = false;
```

**Live Example:**

```html live:no-code
<div id="demo-attrs-property-access">
  <div class="en-card">
    <display-prop value="Initial"></display-prop>
    <div class="en-flex en-gap-2" style="margin-top: 10px">
      <button class="en-btn en-btn--sm" onclick="document.querySelector('#demo-attrs-property-access display-prop').value = 'Updated!'">
        Update Property
      </button>
      <button class="en-btn en-btn--sm" onclick="document.querySelector('#demo-attrs-property-access display-prop').setAttribute('value', 'Via Attribute')">
        Update Attribute
      </button>
    </div>
  </div>
</div>
```

```html
<div id="demo-attrs-property-access">
  <display-prop value="Initial"></display-prop>
  
  <button onclick="document.querySelector('display-prop').value = 'Updated!'">
    Update Property
  </button>
  
  <button onclick="document.querySelector('display-prop').setAttribute('value', 'Via Attribute')">
    Update Attribute
  </button>
</div>
```

```js inject show
JQRE.defineElement('display-prop', {
  props: {
    value: ''
  },
  template: `
    <div>
      <strong>Value: </strong>
      <span $:text="value"></span>
    </div>
  `
});
```

```js cleanup
onUnmount(() => {
  $('#demo-attrs-property-access').component('destroy');
});
```

## Kebab-Case Conversion

HTML attributes use kebab-case, JavaScript properties use camelCase:

```js
JQRE.defineElement('my-component', {
  props: {
    userId: 0,           // camelCase
    firstName: '',       // camelCase
    isActive: false      // camelCase
  }
});
```

```html
<!-- HTML: kebab-case -->
<my-component 
  user-id="123"
  first-name="John"
  is-active>
</my-component>
```

```js
// JavaScript: camelCase
const el = document.querySelector('my-component');
console.log(el.userId);      // 123
console.log(el.firstName);   // "John"
console.log(el.isActive);    // true
```

**Conversion Rules:**
- `userId` ↔ `user-id`
- `firstName` ↔ `first-name`
- `isActive` ↔ `is-active`
- `data` ↔ `data` (no conversion needed)

## Property Reflection

Reflect props back to attributes for CSS styling:

```js
JQRE.defineElement('status-badge', {
  props: {
    status: 'idle',
    loading: false
  }
}, {
  reflect: ['status', 'loading']
});
```

**Usage:**

```html live:no-code
<div id="demo-attrs-reflect">
  <style>
    status-badge[status="active"] { color: green; }
    status-badge[status="error"] { color: red; }
    status-badge[loading] { opacity: 0.5; }
  </style>
  <div class="en-card">
    <div class="en-flex en-gap-3">
      <status-badge status="idle" style="font-weight: bold">Idle</status-badge>
      <status-badge status="active" style="font-weight: bold; color: green">Active</status-badge>
      <status-badge status="error" style="font-weight: bold; color: red">Error</status-badge>
    </div>
  </div>
</div>
```

```html
<style>
  status-badge[status="active"] { color: green; }
  status-badge[status="error"] { color: red; }
  status-badge[loading] { opacity: 0.5; }
</style>

<status-badge status="idle">Idle</status-badge>
<status-badge status="active">Active</status-badge>
<status-badge status="error">Error</status-badge>
```

```js inject show
JQRE.defineElement('status-badge', {
  props: {
    status: 'idle',
    loading: false
  },
  template: `<span><slot></slot></span>`
}, {
  reflect: ['status', 'loading'],
  display: 'inline-block'
});
```

```js cleanup
onUnmount(() => {
  $('#demo-attrs-reflect').component('destroy');
});
```

When prop changes internally, attribute updates automatically:

```js
const badge = document.querySelector('status-badge');
badge.status = 'active';  // attribute updates: status="active"
```

## Default Values

Props without matching attributes use default values:

```js
JQRE.defineElement('config-panel', {
  props: {
    theme: 'light',        // Default: 'light'
    timeout: 5000,         // Default: 5000
    enabled: true          // Default: true
  }
});
```

```html
<!-- Uses all defaults -->
<config-panel></config-panel>

<!-- Overrides theme only -->
<config-panel theme="dark"></config-panel>
```

**Removing Attributes:**

```js
const el = document.querySelector('config-panel');

// Remove attribute → restores default
el.removeAttribute('theme');   // → theme: 'light' (default)
```

## Priority Rules

When both attribute and property are set:

### Initial Values

**Properties win over attributes:**

```js
const el = document.createElement('my-comp');
el.value = 100;                    // Property set first
el.setAttribute('value', '50');    // Attribute set second
// Result: value = 100 (property wins)
```

### After Mount

**Last write wins:**

```js
const el = document.querySelector('my-comp');

// Property write
el.value = 100;

// Attribute write
el.setAttribute('value', '50');  // → value: 50

// Property write again
el.value = 75;                   // → value: 75
```

## Reactive Bindings

Inside JQR components, use directives for reactive props:

### With `$:prop:*`

```html
<div $:component:state="{ userId: 123, name: 'Alice' }">
  <!-- Reactive prop binding -->
  <user-card 
    $:prop:user-id="userId"
    $:prop:name="name">
  </user-card>
</div>
```

### With `$:model:*`

Two-way binding for props:

```html live:no-code
<div id="demo-attrs-model">
  <div class="en-card">
    <div $:component:state="{ count: 5 }">
      <p>External count: <strong $:text="count"></strong></p>
      <number-input $:model:value="count"></number-input>
    </div>
  </div>
</div>
```

```html
<div id="demo-attrs-model">
  <div $:component:state="{ count: 5 }">
    <p>External count: <strong $:text="count"></strong></p>
    <number-input $:model:value="count"></number-input>
  </div>
</div>
```

```js inject show
JQRE.defineElement('number-input', {
  props: {
    value: 0
  },
  emits: ['update:value'],
  methods: {
    updateValue(delta) {
      const newValue = this.$props.value + delta;
      this.$emit('update:value', newValue);
    }
  },
  template: `
    <div style="display: flex; gap: 5px; align-items: center">
      <button class="en-btn en-btn--sm" $:click="updateValue(-1)">-</button>
      <input 
        type="number" 
        class="en-input" 
        style="width: 80px; text-align: center"
        $:value="$props.value"
        $:input="$emit('update:value', Number($event.target.value))">
      <button class="en-btn en-btn--sm" $:click="updateValue(1)">+</button>
    </div>
  `
});
```

```js cleanup
onUnmount(() => {
  $('#demo-attrs-model').component('destroy');
});
```

## Observed Attributes

JQR-E automatically sets `observedAttributes` based on `props`:

```js
JQRE.defineElement('my-comp', {
  props: {
    userId: 0,
    userName: '',
    isActive: false
  }
});

// Internally creates:
// static observedAttributes = ['user-id', 'user-name', 'is-active'];
```

Changes to these attributes trigger `attributeChangedCallback` and update props.

## Property Upgrade

Elements created before `defineElement()` preserve property values:

```js
// Create element before definition
const el = document.createElement('my-comp');
el.value = 42;  // Property set on plain HTMLElement

// Define later
JQRE.defineElement('my-comp', {
  props: { value: 0 }
});

document.body.appendChild(el);
// Element upgrades, value: 42 preserved ✅
```

**Upgrade Process:**
1. Read existing property value
2. Delete property from element instance
3. Re-set value through custom accessor
4. Triggers reactivity correctly

## Prop Validation

Validate props in component lifecycle:

```js
JQRE.defineElement('range-input', {
  props: {
    min: 0,
    max: 100,
    value: 50
  },
  created() {
    const {min, max, value} = this.$props
    // Validate on initialization
    if (value < min) this.$props.value = min;
    if (value > max) this.$props.value = max;
  },
  watch: {
    value(newVal) {
      const {min, max} = this.$props
      // Validate on change
      if (newVal < this.min) {
        this.$setProps({ value: min });
      }
      if (newVal > this.max) {
        this.$setProps({ value: max });
      }
    }
  }
});
```

## Best Practices

### ✅ Do

- **Provide sensible defaults** for all props
- **Use boolean attributes** for on/off states
- **Use `'json'` for complex data** (arrays, objects)
- **Reflect UI state** (disabled, loading, variant)
- **Validate props** in lifecycle hooks
- **Document prop types** in comments or TypeScript

### ❌ Don't

- **Don't mutate objects/arrays** directly (use new references)
- **Don't forget kebab-case** in HTML attributes
- **Don't rely on attribute order** (undefined behavior)
- **Don't use props for large data** (use methods or events instead)

## Common Patterns

### Enum Props

```js
JQRE.defineElement('alert-box', {
  props: {
    type: 'info'  // 'info' | 'success' | 'warning' | 'error'
  },
  created() {
    const validTypes = ['info', 'success', 'warning', 'error'];
    if (!validTypes.includes(this.type)) {
      console.warn(`Invalid type: ${this.type}`);
      this.type = 'info';
    }
  }
});
```

### Computed from Props

```js
JQRE.defineElement('full-name', {
  props: {
    firstName: '',
    lastName: ''
  },
  computed: {
    fullName() {
      return `${this.firstName} ${this.lastName}`.trim();
    }
  }
});
```

### Props with Units

```js
JQRE.defineElement('delay-timer', {
  props: {
    duration: 1000,  // milliseconds
    unit: 'ms'       // 'ms' | 's'
  },
  computed: {
    durationMs() {
      return this.unit === 's' 
        ? this.duration * 1000 
        : this.duration;
    }
  }
});
```

## Troubleshooting

### Props Not Updating

**Problem:** Changes not reflected in UI.

**Solution:** Ensure prop is declared in `props`:

```js
// ❌ Wrong: prop not declared
JQRE.defineElement('my-comp', {
  state: { value: 0 }  // Not a prop!
});

// ✅ Correct: declare as prop
JQRE.defineElement('my-comp', {
  props: { value: 0 }
});
```

### Type Coercion Errors

**Problem:** Prop has wrong type.

**Solution:** Check `attributes` configuration:

```js
JQRE.defineElement('my-comp', {
  props: { count: 0 }
}, {
  attributes: {
    count: Number  // Explicit Number coercion
  }
});
```

### Reflection Not Working

**Problem:** Attribute not updating when prop changes.

**Solution:** Add prop to `reflect` array:

```js
JQRE.defineElement('my-comp', {
  props: { disabled: false }
}, {
  reflect: ['disabled']  // Enable reflection
});
```

## Next Steps

- **[Events](./events.md)** - Emit and handle custom events
- **[Slots](./slots.md)** - Use slots for content projection
- **[Element API](./element-api.md)** - Access instance and methods
- **[Options Reference](../reference/options-reference.md)** - Complete options documentation

## Related

- [Defining Elements](./defining-elements.md)
- [Lifecycle & Ownership](./lifecycle.md)
- [TypeScript Types](../reference/types.md)
