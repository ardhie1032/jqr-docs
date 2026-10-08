<!-- {
  "path": "jqr-e/reference/api-reference",
  "name": "docs-jqr-e-reference-api-reference",
  "meta": {
    "title": "API Reference",
    "order": 1
  }
} -->

# API Reference

Complete API documentation for JQR-E (Custom Elements).

## Global Namespace

**Browser/IIFE:** `JQRE.*`  
**ESM:** `import * as JQRE from '@en32/jqr/element'`

```js
// Browser
JQRE.defineElement('my-button', definition);

// ESM
import { defineElement } from '@en32/jqr/element';
defineElement('my-button', definition);
```

---

## Functions

### `defineElement()`

Define a custom element backed by a JQR component.

#### Signature

```typescript
function defineElement(
  name: string,
  def?: ElementDefinition | AsyncComponent,
  options?: ElementOptions
): CustomElementConstructor
```

#### Parameters

**`name`** (string, required)
- Tag name for the custom element
- Must contain a hyphen (e.g., `'my-button'`)
- Must be lowercase
- Cannot be a reserved name

**`def`** (object, optional)
- Component definition (template or render-based)
- Can be async component from `$.jqr.registerAsync()`
- If omitted, component must already be registered

**`options`** (object, optional)
- Element configuration (see [ElementOptions](#elementoptions))

#### Returns

`CustomElementConstructor` - The custom element class

#### Examples

```js
// Simple definition
JQRE.defineElement('greeting-card', {
  state: { message: 'Hello' },
  template: `<div $:text="message"></div>`
});

// With options
JQRE.defineElement('user-card', {
  props: { userId: null },
  template: `<div>User {{ userId }}</div>`
}, {
  attributes: { userId: Number },
  reflect: ['userId'],
  display: 'block'
});

// Pre-registered component
$.jqr.register('my-button', buttonDef);
JQRE.defineElement('my-button');

// Async component
$.jqr.registerAsync('lazy-widget', {
  load: () => import('./widget.js'),
  props: { title: '' }
});
JQRE.defineElement('lazy-widget');
```

#### Errors

| Code | Message | Cause |
|------|---------|-------|
| `E071` | Invalid tag name | Name doesn't contain hyphen or is reserved |
| `E072` | No definition found | Component not registered and no def provided |
| `E073` | Async component needs props | Async component defined without `props` in registerAsync |

#### See Also

- [Defining Elements Guide](../guides/defining-elements.md)
- [Element Options](../guides/options.md)

---

### `defineElements()`

Define multiple custom elements at once.

#### Signature

```typescript
function defineElements(
  defs: Record<string, ElementDefinition | AsyncComponent>,
  options?: ElementOptions & { prefix?: string }
): void
```

#### Parameters

**`defs`** (object, required)
- Map of tag names to component definitions
- Keys become tag names (with optional prefix)

**`options`** (object, optional)
- `prefix` (string) - Prepended to all tag names
- All ElementOptions apply to all elements

#### Examples

```js
// Batch definition
JQRE.defineElements({
  'app-button': buttonDef,
  'app-input': inputDef,
  'app-card': cardDef
});

// With prefix
JQRE.defineElements({
  'button': buttonDef,
  'input': inputDef,
  'card': cardDef
}, {
  prefix: 'app',  // Creates: app-button, app-input, app-card
  display: 'block'
});

// Mixed with common options
JQRE.defineElements({
  'en-badge': badgeDef,
  'en-alert': alertDef
}, {
  reflect: ['variant'],
  display: 'inline-block'
});
```

#### See Also

- [Defining Elements Guide](../guides/defining-elements.md#defineelements)

---

### `defineRegistered()`

Auto-define custom elements for all registered components with valid tag names.

#### Signature

```typescript
function defineRegistered(options?: ElementOptions): void
```

#### Parameters

**`options`** (object, optional)
- Options applied to all defined elements

#### Behavior

- Scans `$.jqr` component registry
- Only defines components with valid tag names (containing hyphen)
- Skips names already defined as custom elements
- Skips reserved names

#### Example

```js
// Register components
$.jqr.register('my-button', buttonDef);
$.jqr.register('my-input', inputDef);
$.jqr.register('BadName', badDef);  // Skipped (no hyphen)

// Define all at once
JQRE.defineRegistered({
  reflect: ['disabled'],
  display: 'block'
});

// Now <my-button> and <my-input> work as custom elements
```

#### See Also

- [Defining Elements Guide](../guides/defining-elements.md#defineregistered)

---

### `isJqrElement()`

Check if an element is a JQR custom element.

#### Signature

```typescript
function isJqrElement(el: unknown): el is JqrElement
```

#### Parameters

**`el`** (any)
- Value to check

#### Returns

`boolean` - True if element was created by `defineElement()`

#### Example

```js
const el = document.querySelector('my-button');
if (JQRE.isJqrElement(el)) {
  console.log('Is JQR element');
  console.log(el.$instance);
}
```

---

### `getInstance()`

Get the component instance mounted on an element.

#### Signature

```typescript
function getInstance(el: Element): JqrComponent | undefined
```

#### Parameters

**`el`** (Element, required)
- DOM element to query

#### Returns

`JqrComponent | undefined` - The mounted instance, or undefined if not mounted

#### Example

```js
const el = document.querySelector('my-button');
const instance = JQRE.getInstance(el);

if (instance) {
  console.log(instance.$state);
  instance.$emit('click');
}
```

#### Alias

This is equivalent to `el.$instance` on JQR elements:

```js
// These are the same
const instance1 = JQRE.getInstance(el);
const instance2 = el.$instance;
```

---

## Types

### `ElementDefinition`

Component definition accepted by `defineElement()`.

```typescript
type ElementDefinition = ComponentOptions<any, any> & {
  render?: (h: any, ctx: any) => any
}
```

Any standard JQR `ComponentOptions`, plus optional `render()` for JQR-H integration.

**Properties:**
- All standard component options: `state`, `props`, `methods`, `computed`, `watch`, `template`, etc.
- `render()` - Optional render function (requires JQR-H loaded)

**Example:**

```typescript
const buttonDef: ElementDefinition = {
  props: {
    variant: 'default',
    disabled: false
  },
  methods: {
    click() {
      this.$emit('click');
    }
  },
  template: `
    <button $:disabled="disabled" $:click="click">
      <slot></slot>
    </button>
  `
};
```

---

### `ElementOptions`

Configuration options for `defineElement()`.

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

#### Properties

**`attributes`** (object, optional)
- Type coercion rules per prop
- Keys: prop names (camelCase)
- Values: `String`, `Number`, `Boolean`, or `'json'`

**`reflect`** (array, optional)
- Props to mirror back to HTML attributes
- Enables CSS selectors: `my-button[disabled]`

**`expose`** (array, optional)
- Method names to expose on element
- Allows: `element.methodName()`

**`mount`** (string, optional)
- `'auto'` (default): Mount when connected
- `'manual'`: Require explicit `mount()` call

**`display`** (string, optional)
- CSS display value injected as style rule
- Example: `'block'`, `'inline-block'`, `'flex'`

**`keepAlive`** (boolean, optional)
- `false` (default): Destroy on disconnect
- `true`: Keep component alive when disconnected

**Example:**

```typescript
const options: ElementOptions = {
  attributes: {
    userId: Number,
    config: 'json'
  },
  reflect: ['disabled', 'loading'],
  expose: ['open', 'close', 'toggle'],
  mount: 'auto',
  display: 'block',
  keepAlive: false
};

JQRE.defineElement('my-dialog', dialogDef, options);
```

**See:** [Element Options Guide](../guides/options.md)

---

### `AttributeType`

Attribute coercion type.

```typescript
type AttributeType = StringConstructor | NumberConstructor | BooleanConstructor | 'json'
```

| Type | Behavior |
|------|----------|
| `String` | Use value as-is |
| `Number` | Parse with `Number()` |
| `Boolean` | Presence = true, absence = false |
| `'json'` | Parse with `JSON.parse()` |

**Example:**

```typescript
const attributes: Record<string, AttributeType> = {
  name: String,
  age: Number,
  active: Boolean,
  data: 'json'
};
```

---

### `JqrElement`

Interface for custom elements created by JQR-E.

```typescript
interface JqrElement extends HTMLElement {
  readonly $instance: JqrComponent | undefined;
  readonly $mounted: Promise<JqrComponent>;
  mount(): JqrComponent | undefined;
  unmount(): void;
}
```

#### Properties

**`$instance`** (readonly)
- Type: `JqrComponent | undefined`
- The mounted component instance
- `undefined` if not mounted or destroyed

**`$mounted`** (readonly)
- Type: `Promise<JqrComponent>`
- Resolves when component finishes mounting
- Rejects if mounting fails
- For async components, waits for definition to load

#### Methods

**`mount()`**
- Manually mount the component
- Required for `mount: 'manual'` elements
- Returns instance or undefined
- Safe to call multiple times (idempotent)

**`unmount()`**
- Destroy the component
- Clears state and effects
- Restores original slot content
- Safe to call multiple times (idempotent)

**Example:**

```typescript
const el = document.querySelector('my-widget') as JqrElement;

// Access instance
if (el.$instance) {
  console.log(el.$instance.$state);
}

// Wait for mount
await el.$mounted;

// Manual control
el.mount();
el.unmount();
```

**See:** [Element API Guide](../guides/element-api.md)

---

## Component Instance API

Once mounted, custom elements expose the full JQR component API via `$instance`:

### Properties

| Property | Type | Description |
|----------|------|-------------|
| `$el` | `HTMLElement` | The custom element (host) |
| `$state` | `object` | Reactive state |
| `$props` | `object` | Props (readonly) |
| `$computed` | `object` | Computed values |
| `$ctx` | `object` | Methods context |
| `$refs` | `object` | Element references |

### Methods

| Method | Signature | Description |
|--------|-----------|-------------|
| `$setProps()` | `(props: object) => void` | Update props |
| `$emit()` | `(event: string, detail?: any) => void` | Emit custom event |
| `$watch()` | `(getter, callback, options?) => stop` | Watch reactive value |
| `$nextTick()` | `(callback) => void` | Run after DOM update |

**Example:**

```js
const el = document.querySelector('user-card');
const instance = el.$instance;

// Read state
console.log(instance.$state.username);

// Update props
instance.$setProps({ userId: 456 });

// Emit event
instance.$emit('select', { id: 123 });

// Watch value
const stop = instance.$watch(
  () => instance.$state.count,
  (newVal) => console.log('Count:', newVal)
);

// Call method
instance.$ctx.refresh();
```

**See:** [Instance Methods Guide](../../jqr/guides/instance-methods.md)

---

## Usage with h()

Custom elements work seamlessly with JQR-H's `h()` function:

```js
import { h, render } from '@en32/jqr-h';

// Define element
JQRE.defineElement('user-card', userCardDef);

// Use with h()
render((h) => 
  h('user-card', {
    attrs: {
      userId: (ctx) => ctx.currentUserId
    },
    on: {
      select: (ctx, event) => {
        ctx.selectedId = event.detail.id;
      }
    }
  }, [
    h('span', { slot: 'header' }, ['User Details'])
  ])
, '#app');
```

**Features:**
- Reactive prop binding via `attrs`
- Event handling via `on`
- Slot content via children
- Dynamic classes and styles

**See:** [h() Integration Guide](../guides/h-integration.md)

---

## Error Codes

JQR-E error codes are prefixed with `E07x`:

| Code | Message | Guide |
|------|---------|-------|
| `E071` | Invalid element name | [Defining Elements](../guides/defining-elements.md#tag-name-rules) |
| `E072` | No definition found | [Defining Elements](../guides/defining-elements.md#definition-patterns) |
| `E073` | Async component needs props | [Lifecycle](../guides/lifecycle.md#async-components) |
| `E074` | Attribute coercion failed | [Attributes & Props](../guides/attributes-and-props.md#type-coercion) |
| `E075` | Invalid attribute type | [Attributes & Props](../guides/attributes-and-props.md#coercion-types) |
| `E076` | Method not exposed | [Element Options](../guides/options.md#expose) |
| `E077` | Slot alias invalid | [Slots](../guides/slots.md) |
| `E078` | Render needs renderer | [h() Integration](../guides/h-integration.md) |
| `E079` | Element owned late | [Lifecycle](../guides/lifecycle.md#ownership-modes) |
| `E080` | Mount failed | [Lifecycle](../guides/lifecycle.md#standalone-mounting) |

**Error Reporting:**

```js
try {
  JQRE.defineElement('invalid', definition);
} catch (error) {
  console.error('Error code:', error.code);  // E071, E072, etc.
  console.error('Message:', error.message);
}
```

---

## TypeScript

Full TypeScript support with exported types:

```typescript
import {
  defineElement,
  defineElements,
  defineRegistered,
  isJqrElement,
  getInstance,
  type ElementDefinition,
  type ElementOptions,
  type AttributeType,
  type JqrElement
} from '@en32/jqr/element';

// Typed definition
const buttonDef: ElementDefinition = {
  props: {
    variant: 'default' as 'default' | 'primary' | 'danger',
    disabled: false
  }
};

// Typed options
const options: ElementOptions = {
  attributes: {
    disabled: Boolean
  },
  reflect: ['disabled', 'variant'],
  expose: ['click', 'focus'],
  display: 'inline-block'
};

// Define with full type safety
defineElement('app-button', buttonDef, options);

// Typed element reference
const el = document.querySelector('app-button') as JqrElement;
await el.$mounted;
console.log(el.$instance?.$state);
```

---

## Best Practices

### ✅ Do

- **Use semantic tag names** - `user-card`, `date-picker`, not `my-component`
- **Provide default values** - All props should have defaults
- **Document emitted events** - Use `emits` array or JSDoc
- **Type your definitions** - Use TypeScript for clarity
- **Keep elements focused** - Single responsibility principle

### ❌ Don't

- **Don't define elements conditionally** - Can cause registry conflicts
- **Don't redefine with different definitions** - First definition wins
- **Don't forget hyphens** - Tag names must contain hyphens
- **Don't expose everything** - Only expose public API methods
- **Don't skip cleanup** - Always call `unmount()` for `keepAlive` elements

---

## Browser Support

Custom Elements API requirements:
- Chrome 67+
- Firefox 63+
- Safari 10.1+
- Edge 79+

**Polyfill:** Use [@webcomponents/custom-elements](https://github.com/webcomponents/polyfills/tree/master/packages/custom-elements) for older browsers.

---

## See Also

### Guides
- [Defining Elements](../guides/defining-elements.md)
- [Attributes & Props](../guides/attributes-and-props.md)
- [Element API](../guides/element-api.md)
- [Lifecycle](../guides/lifecycle.md)
- [Element Options](../guides/options.md)

### Reference
- [Options Reference](./options-reference.md)
- [TypeScript Types](./types.md)

### Examples
- [Button Component](../examples/button-component.md)
- [Dialog Component](../examples/dialog-component.md)
