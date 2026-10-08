<!-- {
  "path": "jqr-e/reference/types",
  "name": "docs-jqr-e-reference-types",
  "meta": {
    "title": "TypeScript Types",
    "order": 3
  }
} -->

# TypeScript Types

Complete TypeScript type reference for JQR-E (`@en32/jqr/element`).

## Core Types

### AttributeType

Type coercion for HTML attribute to prop conversion.

```typescript
type AttributeType = 
  | StringConstructor 
  | NumberConstructor 
  | BooleanConstructor 
  | 'json'
```

**Coercion Rules:**

| Type | Behavior | Example |
|------|----------|---------|
| `String` | Use attribute value as-is | `name="Alice"` → `"Alice"` |
| `Number` | Parse with `Number()` | `count="42"` → `42` |
| `Boolean` | Attribute presence = true | `disabled` → `true` |
| `'json'` | Parse with `JSON.parse()` | `data='[1,2]'` → `[1,2]` |

**Usage:**

```typescript
const attributes: Record<string, AttributeType> = {
  userId: Number,
  userName: String,
  isActive: Boolean,
  config: 'json'
};
```

**Special Behaviors:**

**Boolean:**
- Attribute present → `true`
- Attribute absent → `false`
- Attribute = `"false"` → `false`
- Any other value → `true`

**Number:**
- Valid number string → number value
- Invalid/NaN → `NaN` (handle in component)
- Empty string → `0`

**JSON:**
- Valid JSON → parsed object/array
- Invalid JSON → ignored with dev warning
- Falls back to default value

---

### ElementDefinition

Component definition accepted by `defineElement()`.

```typescript
type ElementDefinition = ComponentOptions<any, any> & {
  render?: (h: any, ctx: any) => any
}
```

Any standard JQR `ComponentOptions`, plus optional `render()` function.

**Properties:**

All standard component options:
- `state` - Reactive state
- `props` - Component props
- `computed` - Computed properties
- `methods` - Component methods
- `watch` - Watchers
- `template` - HTML template
- `components` - Nested components
- `created`, `mounted`, `updated`, `destroyed` - Lifecycle hooks
- `asyncState` - Async data loading
- `emits` - Event declarations

Plus optional:
- `render()` - Render function (requires JQR-H)

**Examples:**

```typescript
// Template-based
const buttonDef: ElementDefinition = {
  props: {
    variant: 'default',
    disabled: false
  },
  methods: {
    handleClick() {
      this.$emit('click');
    }
  },
  template: `
    <button 
      $:disabled="disabled"
      $:click="handleClick">
      <slot></slot>
    </button>
  `
};

// Render-based (requires JQR-H)
const cardDef: ElementDefinition = {
  props: {
    title: ''
  },
  render: (h, ctx) => h('div', { class: 'card' }, [
    h('h3', { text: () => ctx.$props.title }),
    h.slot()
  ])
};
```

---

### ElementOptions

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

**Properties:**

**`attributes`** (optional)
- Type: `Record<string, AttributeType>`
- Attribute type coercion per prop
- Overrides type inference

**`reflect`** (optional)
- Type: `string[]`
- Props to mirror back to attributes
- Enables CSS selectors

**`expose`** (optional)
- Type: `string[]`
- Method names to expose on element
- Enables `element.methodName()`

**`mount`** (optional)
- Type: `'auto' | 'manual'`
- Default: `'auto'`
- Mounting strategy

**`display`** (optional)
- Type: `string`
- CSS display value
- Injected as style rule

**`keepAlive`** (optional)
- Type: `boolean`
- Default: `false`
- Keep component alive when disconnected

**Example:**

```typescript
const options: ElementOptions = {
  attributes: {
    userId: Number,
    disabled: Boolean,
    config: 'json'
  },
  reflect: ['disabled', 'loading'],
  expose: ['open', 'close'],
  mount: 'auto',
  display: 'block',
  keepAlive: false
};

JQRE.defineElement('my-dialog', dialogDef, options);
```

---

### JqrElement

Interface for custom elements created by `defineElement()`.

```typescript
interface JqrElement extends HTMLElement {
  readonly $instance: JqrComponent | undefined;
  readonly $mounted: Promise<JqrComponent>;
  mount(): JqrComponent | undefined;
  unmount(): void;
}
```

**Properties:**

**`$instance`** (readonly)
- Type: `JqrComponent | undefined`
- The mounted component instance
- `undefined` if not mounted or destroyed

**`$mounted`** (readonly)
- Type: `Promise<JqrComponent>`
- Resolves when component mounts
- Rejects if mounting fails
- Waits for async component to load

**Methods:**

**`mount()`**
- Returns: `JqrComponent | undefined`
- Manually mount the component
- Required for `mount: 'manual'`
- Idempotent (safe to call multiple times)

**`unmount()`**
- Returns: `void`
- Destroy the component
- Clears state and effects
- Restores original slot content

**Example:**

```typescript
const element = document.querySelector('my-widget') as JqrElement;

// Check instance
if (element.$instance) {
  console.log(element.$instance.$state);
}

// Wait for mount
const instance = await element.$mounted;
console.log('Mounted:', instance);

// Manual control
element.mount();
element.unmount();
```

**Property Access:**

All declared props are accessible:

```typescript
// Assuming props: { userId: 0, userName: '' }
const el = document.querySelector('user-card') as JqrElement;

// Read props
console.log(el.userId);      // number
console.log(el.userName);    // string

// Set props
el.userId = 123;
el.userName = 'Alice';

// Access instance
if (el.$instance) {
  console.log(el.$instance.$props.userId);
}
```

---

## Function Types

### defineElement()

Main function for defining custom elements.

```typescript
function defineElement(
  name: string,
  def?: ElementDefinition | AsyncComponent,
  options?: ElementOptions
): CustomElementConstructor
```

**Parameters:**

**`name`** (string, required)
- Tag name for custom element
- Must contain hyphen
- Must be lowercase
- Cannot be reserved name

**`def`** (ElementDefinition | AsyncComponent, optional)
- Component definition
- Can be async component
- If omitted, must be pre-registered

**`options`** (ElementOptions, optional)
- Element configuration

**Returns:**
- `CustomElementConstructor` - The custom element class

**Example:**

```typescript
import { defineElement, type ElementOptions } from '@en32/jqr/element';

const options: ElementOptions = {
  attributes: { count: Number },
  reflect: ['count'],
  display: 'block'
};

const ctor = defineElement('my-counter', counterDef, options);
console.log(ctor.name); // "JqrCustomElement"
```

---

### defineElements()

Define multiple elements at once.

```typescript
function defineElements(
  defs: Record<string, ElementDefinition | AsyncComponent>,
  options?: ElementOptions & { prefix?: string }
): void
```

**Parameters:**

**`defs`** (object, required)
- Map of tag names to definitions

**`options`** (object, optional)
- All `ElementOptions` fields
- Plus `prefix` for namespacing

**Example:**

```typescript
import { defineElements } from '@en32/jqr/element';

defineElements({
  'button': buttonDef,
  'input': inputDef,
  'card': cardDef
}, {
  prefix: 'app',  // Creates: app-button, app-input, app-card
  reflect: ['disabled'],
  display: 'block'
});
```

---

### defineRegistered()

Auto-define elements from registry.

```typescript
function defineRegistered(options?: ElementOptions): void
```

**Parameters:**

**`options`** (ElementOptions, optional)
- Options applied to all elements

**Example:**

```typescript
import { defineRegistered } from '@en32/jqr/element';

// Register components first
$.jqr.register('my-button', buttonDef);
$.jqr.register('my-input', inputDef);

// Define all valid names
defineRegistered({
  reflect: ['disabled'],
  display: 'block'
});
```

---

### isJqrElement()

Type guard for JQR elements.

```typescript
function isJqrElement(el: unknown): el is JqrElement
```

**Parameters:**
- `el` - Value to check

**Returns:**
- `boolean` - True if JQR custom element

**Example:**

```typescript
import { isJqrElement, type JqrElement } from '@en32/jqr/element';

const el = document.querySelector('my-button');

if (isJqrElement(el)) {
  // TypeScript knows el is JqrElement
  console.log(el.$instance);
  el.mount();
  el.unmount();
}
```

---

### getInstance()

Get component instance from element.

```typescript
function getInstance(el: Element): JqrComponent | undefined
```

**Parameters:**
- `el` - DOM element

**Returns:**
- `JqrComponent | undefined` - Instance or undefined

**Example:**

```typescript
import { getInstance } from '@en32/jqr/element';

const el = document.querySelector('my-button');
const instance = getInstance(el);

if (instance) {
  console.log(instance.$state);
  instance.$emit('click');
}
```

---

## Utility Types

### Dict

Generic object with string keys.

```typescript
type Dict<T = any> = Record<string, T>
```

Used throughout for object types.

---

### ComponentOptions

Standard JQR component options (from core).

```typescript
interface ComponentOptions<S extends Dict = Dict, P extends Dict = Dict> {
  state?: S;
  props?: P | string[];
  computed?: Dict<(this: HookCtx<S, P>) => any>;
  methods?: Dict<(this: HookCtx<S, P>, ...args: any[]) => any>;
  watch?: Dict<WatchOptions | WatchCallback>;
  template?: string;
  components?: Dict<ComponentOptions<any, any>>;
  created?: (this: HookCtx<S, P>) => void;
  mounted?: (this: HookCtx<S, P>) => void;
  updated?: (this: HookCtx<S, P>) => void;
  destroyed?: (this: HookCtx<S, P>) => void;
  asyncState?: Dict<AsyncStateGetter<S, P>>;
  emits?: string[];
  // ... more options
}
```

See [JQR Core Types](../../jqr/reference/config-reference.md) for complete details.

---

### JqrComponent

Component instance interface (from core).

```typescript
interface JqrComponent {
  $el: HTMLElement;
  $state: Dict;
  $props: Dict;
  $computed: Dict;
  $ctx: Dict;
  $refs: Dict<Element>;
  $setProps(props: Dict): void;
  $emit(event: string, detail?: any): void;
  $watch(getter: Function, callback: Function, options?: WatchOptions): () => void;
  $nextTick(callback: () => void): void;
  $destroy(): void;
  $refetch(key?: string): Promise<void>;
}
```

Available on `element.$instance`.

---

### AsyncComponent

Async component from `$.jqr.registerAsync()`.

```typescript
interface AsyncComponent {
  load(): Promise<ComponentOptions<any, any>>;
  options: {
    props?: string[] | Dict;
    loading?: ComponentOptions<any, any>;
    error?: ComponentOptions<any, any>;
    timeout?: number;
  };
  def?: ComponentOptions<any, any>;
  isLoaded(): boolean;
}
```

Can be passed to `defineElement()`:

```typescript
const asyncWidget = $.jqr.registerAsync('lazy-widget', {
  load: () => import('./widget.js'),
  props: ['title']
});

defineElement('lazy-widget', asyncWidget);
```

---

## Type Patterns

### Typed Element Definition

```typescript
import type { ElementDefinition, ElementOptions } from '@en32/jqr/element';

interface ButtonState {
  loading: boolean;
}

interface ButtonProps {
  variant: 'default' | 'primary' | 'danger';
  disabled: boolean;
}

const buttonDef: ElementDefinition = {
  props: {
    variant: 'default' as 'default' | 'primary' | 'danger',
    disabled: false
  },
  state: {
    loading: false
  },
  methods: {
    click() {
      if (!this.$props.disabled && !this.loading) {
        this.$emit('click');
      }
    }
  },
  template: `
    <button 
      $:disabled="$props.disabled || loading"
      $:click="click">
      <span $:if="loading">Loading...</span>
      <slot $:if="!loading"></slot>
    </button>
  `
};

const options: ElementOptions = {
  attributes: {
    disabled: Boolean
  },
  reflect: ['disabled', 'loading'],
  expose: ['click'],
  display: 'inline-block'
};

defineElement('app-button', buttonDef, options);
```

---

### Typed Element Reference

```typescript
import type { JqrElement } from '@en32/jqr/element';

// Extend with specific props
interface AppButton extends JqrElement {
  variant: 'default' | 'primary' | 'danger';
  disabled: boolean;
  loading: boolean;
}

const button = document.querySelector('app-button') as AppButton;

// Typed prop access
button.variant = 'primary';
button.disabled = true;

// Instance access
if (button.$instance) {
  console.log(button.$instance.$state.loading);
}
```

---

### Typed Method Exposure

```typescript
interface DialogElement extends JqrElement {
  open(): void;
  close(): void;
  toggle(): void;
}

const dialog = document.querySelector('app-dialog') as DialogElement;

// TypeScript knows these exist
dialog.open();
dialog.close();
dialog.toggle();
```

---

### Typed Event Handling

```typescript
import type { JqrElement, JqrComponent } from '@en32/jqr/element';

interface ClickEventDetail {
  button: JqrElement;
  timestamp: number;
}

const button = document.querySelector('app-button');

button?.addEventListener('jqr:click', (event: CustomEvent<ClickEventDetail>) => {
  console.log('Clicked:', event.detail.button);
  console.log('At:', event.detail.timestamp);
});
```

---

### Typed Attributes Object

```typescript
import type { AttributeType } from '@en32/jqr/element';

const attributeConfig: Record<string, AttributeType> = {
  // Type checking ensures valid AttributeType
  userId: Number,
  userName: String,
  isActive: Boolean,
  config: 'json',
  // @ts-expect-error - Invalid type
  invalid: 'unknown'  // Error!
};
```

---

## Advanced Patterns

### Generic Element Factory

```typescript
import type { 
  ElementDefinition, 
  ElementOptions, 
  JqrElement 
} from '@en32/jqr/element';

function createTypedElement<
  TElement extends JqrElement = JqrElement
>(
  name: string,
  def: ElementDefinition,
  options?: ElementOptions
): { new(): TElement } {
  return defineElement(name, def, options) as any;
}

// Usage
interface MyButton extends JqrElement {
  variant: string;
  click(): void;
}

const ButtonClass = createTypedElement<MyButton>(
  'my-button',
  buttonDef,
  { expose: ['click'] }
);
```

---

### Prop Type Inference

```typescript
type InferProps<T extends ElementDefinition> = 
  T extends { props: infer P } ? P : never;

const userCardDef = {
  props: {
    userId: 0,
    userName: ''
  },
  template: `...`
} as const;

type UserCardProps = InferProps<typeof userCardDef>;
// { userId: number; userName: string }
```

---

## Type Imports

All types are exported from `@en32/jqr/element`:

```typescript
// Main types
import type {
  AttributeType,
  ElementDefinition,
  ElementOptions,
  JqrElement
} from '@en32/jqr/element';

// Functions (for type reference)
import type {
  defineElement,
  defineElements,
  defineRegistered,
  isJqrElement,
  getInstance
} from '@en32/jqr/element';

// Utility types
import type { Dict } from '@en32/jqr/element';

// From JQR core (also available)
import type {
  JqrComponent,
  ComponentOptions,
  AsyncComponent
} from '@en32/jqr';
```

---

## Browser vs ESM

**Browser (IIFE):**

Types are defined but accessed via globals:

```typescript
// No imports needed
declare global {
  const JQRE: {
    defineElement: typeof defineElement;
    // ...
  };
}

// Use directly
JQRE.defineElement('my-element', def);
```

**ESM:**

Import types and values:

```typescript
import { 
  defineElement,
  type ElementOptions,
  type JqrElement 
} from '@en32/jqr/element';

defineElement('my-element', def);
```

---

## See Also

- [API Reference](./api-reference.md) - Complete API documentation
- [Options Reference](./options-reference.md) - ElementOptions details
- [Element API Guide](../guides/element-api.md) - Using JqrElement
- [Defining Elements](../guides/defining-elements.md) - Getting started
