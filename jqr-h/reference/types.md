<!-- {
  "path": "jqr-h/reference/types",
  "name": "docs-jqr-h-reference-types",
  "meta": {
    "title": "TypeScript Types",
    "order": 1
  }
} -->

# TypeScript Types

Complete TypeScript type reference for JQR-H (`@en32/jqr-h`).

## Core Types

### JqrNode

The description built by `h()`. Not rendered until passed to `render()`/`mount()`.

```typescript
interface JqrNode<S extends Dict = Dict> {
  readonly [JQR_NODE]: true;
  tag: string;
  componentRef?: ComponentOptions<any, any> | AsyncComponent;
  attrs: NodeAttrs<S>;
  children: Child<S>[];
  slots: Dict<Child<S>[]>;
}
```

**Properties:**
- `tag`: HTML tag name or registered component name
- `componentRef`: Component definition (when using `h(ComponentDef, ...)`)
- `attrs`: All attributes and bindings
- `children`: Alias for `slots.default`
- `slots`: Named slot content

### ForNode

The description built by `h.for()`. Renders as a keyed list.

```typescript
interface ForNode<S extends Dict = Dict, T = any> {
  readonly [JQR_FOR_NODE]: true;
  items: StateFn<S, T[]>;
  key: (item: T, index: number) => any;
  build: (item: T, index: number) => JqrNode<any>;
}
```

**Properties:**
- `items`: Function returning array of items
- `key`: Function extracting unique key from item
- `build`: Function building node for each item

### SlotNode

The result of `h.slot()`. An outlet for parent-supplied content.

```typescript
interface SlotNode<S extends Dict = Dict> {
  readonly [JQR_SLOT_NODE]: true;
  name: string;
  fallback: Child<S>[];
}
```

**Properties:**
- `name`: Slot name (default: `'default'`)
- `fallback`: Content shown when parent supplies nothing

## Context Types

### Ctx

The context every binding receives. Alias for `HookCtx<S, Dict>` from core JQR.

```typescript
type Ctx<S extends Dict = Dict> = HookCtx<S, Dict>
```

**Available Properties:**
- `$state`: Reactive state object
- `$props`: Readonly props object
- `$emit()`: Emit custom events
- `$watch()`: Watch for changes
- `$destroy()`: Destroy component
- `$refetch()`: Refetch async state
- All state/computed/methods/constants available directly

**Example:**
```typescript
// Both forms work
ctx.$state.count++;
ctx.count++;  // Same thing

// Call methods
ctx.increment();
ctx.$emit('change', value);
```

### RenderContext

The context passed to `render(h, ctx)`. Same as `Ctx`.

```typescript
type RenderContext<S extends Dict = Dict, P extends Dict = Dict> = HookCtx<S, P>
```

## Attribute Types

### NodeAttrs

All attributes/bindings accepted by `h(tag, attrs, children)`.

```typescript
interface NodeAttrs<S extends Dict = Dict> {
  class?: Getter<S, any>;
  style?: Getter<S, any>;
  attrs?: Dict<Getter<S, any>>;
  on?: Dict<(ctx: Ctx<S>, ev: Event) => void>;
  emit?: Dict<(ctx: Ctx<S>, payload: any) => void>;
  text?: StateFn<S, any>;
  html?: StateFn<S, any>;
  show?: StateFn<S, boolean>;
  if?: StateFn<S, boolean>;
  model?: string | Dict<string> | NamedModelBinding | FormModelBinding | ComputedModelBinding<S>;
  ref?: string;
  props?: Dict<StateFn<S, any>>;
  directive?: Dict<Getter<S, any>>;
  bind?: Getter<S, Dict<any>>;
  teleport?: Getter<S, string | { to: string; mode?: TeleportMode; defer?: boolean }>;
}
```

### Getter

A value that can be static or reactive.

```typescript
type Getter<S extends Dict = Dict, T = any> = T | ((ctx: Ctx<S>) => T)
```

**Examples:**
```typescript
// Static
{ class: 'card' }

// Reactive
{ class: (ctx) => ({ active: ctx.isOpen }) }
```

### StateFn

Only the reactive form (function required).

```typescript
type StateFn<S extends Dict = Dict, T = any> = (ctx: Ctx<S>) => T
```

Used for bindings that only make sense as functions:
- `text`
- `html`
- `show`
- `if`
- Event handlers
- `props`

## Model Binding Types

### NamedModelBinding

Mode B: Named DOM property two-way binding.

```typescript
interface NamedModelBinding {
  prop: string;
  state: string;
  on?: string;
}
```

**Example:**
```typescript
// Bind dialog.open to state.showDialog
model: {
  prop: 'open',
  state: 'showDialog',
  on: 'toggle'  // Optional
}
```

### FormModelBinding

Form control binding with timing modifiers.

```typescript
interface FormModelBinding {
  state: string;
  debounce?: number | true;
  throttle?: number | true;
  lazy?: boolean;
}
```

**Example:**
```typescript
// Debounced input
model: {
  state: 'searchQuery',
  debounce: 400
}
```

### ComputedModelBinding

Form control binding with custom getter/setter.

```typescript
interface ComputedModelBinding<S extends Dict = Dict> {
  get: StateFn<S, any>;
  set: (ctx: Ctx<S>, value: any) => void;
  debounce?: number | true;
  throttle?: number | true;
  lazy?: boolean;
}
```

**Example:**
```typescript
// Nested path binding
model: {
  get: (ctx) => ctx.$state.user.name,
  set: (ctx, value) => { ctx.$state.user.name = value; },
  debounce: true
}
```

## Function Types

### HFunction

The `h()` function signature.

```typescript
interface HFunction {
  // Plain HTML tag or registered component name
  <S extends Dict = Dict>(
    tag: string,
    attrs?: NodeAttrs<S> | null,
    children?: ChildrenArg<S>
  ): JqrNode<S>;
  
  // Component definition with typed props
  <S extends Dict = Dict, P extends Dict = Dict>(
    componentRef: ComponentOptions<any, P>,
    attrs?: (Omit<NodeAttrs<S>, 'props' | 'model'> & {
      props: { [K in keyof P]: StateFn<S, P[K]> };
      model?: Partial<{ [K in keyof P]: string }>;
    }) | null,
    children?: ChildrenArg<S>
  ): JqrNode<S>;
  
  // Async component
  <S extends Dict = Dict>(
    componentRef: AsyncComponent,
    attrs?: NodeAttrs<S> | null,
    children?: ChildrenArg<S>
  ): JqrNode<S>;
  
  // h.for
  for<S extends Dict = Dict, T = any>(
    items: StateFn<S, T[]>,
    key: (item: T, index: number) => any,
    build: (item: T, index: number) => JqrNode<any>
  ): ForNode<S, T>;
  
  // h.slot
  slot<S extends Dict = Dict>(
    name?: string,
    fallback?: Child<S>[]
  ): SlotNode<S>;
}
```

## Child Types

### Child

Anything that can be a child in `h()`.

```typescript
type Child<S extends Dict = Dict> =
  | JqrNode<any>
  | ForNode<S, any>
  | SlotNode<S>
  | string
  | number
  | StateFn<S, string | number>
  | JQuery
  | Node
```

**Examples:**
```typescript
// Static text
h('div', {}, ['Hello'])

// Reactive text
h('div', {}, [(ctx) => ctx.message])

// Nested elements
h('div', {}, [
  h('p', {}, ['Paragraph']),
  h('span', {}, ['Span'])
])

// h.for list
h('ul', {}, [
  h.for(
    (ctx) => ctx.items,
    (item) => item.id,
    (item) => h('li', {}, [item.name])
  )
])

// Raw DOM/jQuery
h('div', {}, [
  $('#existing-element')[0],
  $('<span>jQuery</span>')
])
```

### ChildrenArg

Third argument to `h()`: array or named slots object.

```typescript
type ChildrenArg<S extends Dict = Dict> = Child<S>[] | Dict<Child<S>[]>
```

**Examples:**
```typescript
// Default slot (array)
h('modal', {}, [
  h('p', {}, ['Content'])
])

// Named slots (object)
h('modal', {}, {
  default: [h('p', {}, ['Body'])],
  header: [h('h2', {}, ['Title'])],
  footer: [h('button', {}, ['Close'])]
})
```

## Component Types

### RenderComponentOptions

Component options with `render()` function.

```typescript
interface RenderComponentOptions<S extends Dict = Dict, P extends Dict = Dict> 
  extends ComponentOptions<S, P> {
  render: (h: HFunction, ctx: RenderContext<S, P>) => JqrNode<S>;
}
```

**Example:**
```typescript
const component: RenderComponentOptions<{ count: number }> = {
  state: { count: 0 },
  methods: {
    increment() {
      this.count++;
    }
  },
  render: (h, ctx) => h('div', {}, [
    h('p', { text: (c) => `Count: ${c.count}` }),
    h('button', {
      on: { click: (c) => c.increment() }
    }, ['Increment'])
  ])
};
```

## Utility Types

### Dict

Generic object with string keys.

```typescript
type Dict<T = any> = Record<string, T>
```

### MountOptions

Options for `mount()` function.

```typescript
interface MountOptions {
  slots?: SlotGroups;
  // ... other options from core JQR
}
```

## Type Guards

Functions to check node types at runtime:

```typescript
function isJqrNode(value: unknown): value is JqrNode<any>
function isForNode(value: unknown): value is ForNode<any, any>
function isSlotNode(value: unknown): value is SlotNode<any>
```

## Usage Examples

### Typed Component

```typescript
interface UserState {
  name: string;
  age: number;
}

interface UserProps {
  userId: number;
}

const UserCard: RenderComponentOptions<UserState, UserProps> = {
  props: {
    userId: 0
  },
  state: {
    name: '',
    age: 0
  },
  render: (h, ctx) => h('div', { class: 'user-card' }, [
    h('h2', { text: (c) => c.name }),
    h('p', { text: (c) => `Age: ${c.age}` }),
    h('p', { text: (c) => `ID: ${c.$props.userId}` })
  ])
};
```

### Typed Attributes

```typescript
interface AppState {
  isActive: boolean;
  count: number;
  theme: 'light' | 'dark';
}

const attrs: NodeAttrs<AppState> = {
  class: (ctx) => ({
    active: ctx.isActive,
    [`theme-${ctx.theme}`]: true
  }),
  attrs: {
    'data-count': (ctx) => ctx.count
  },
  on: {
    click: (ctx, ev) => {
      ctx.count++;
      ctx.$emit('increment', ctx.count);
    }
  }
};
```

### Typed h.for

```typescript
interface Item {
  id: number;
  name: string;
}

interface State {
  items: Item[];
}

const list = h.for<State, Item>(
  (ctx) => ctx.items,
  (item) => item.id,
  (item) => h('li', {}, [item.name])
);
```

## Imports

All types are exported from `@en32/jqr-h`:

```typescript
import type {
  JqrNode,
  ForNode,
  SlotNode,
  NodeAttrs,
  Child,
  ChildrenArg,
  Getter,
  StateFn,
  Ctx,
  RenderContext,
  HFunction,
  RenderComponentOptions,
  NamedModelBinding,
  FormModelBinding,
  ComputedModelBinding,
  MountOptions
} from '@en32/jqr-h';
```

## See Also

- [h() Function](../guides/h-function.md) - Using the h() function
- [Render Functions](../guides/render-function.md) - Component render functions
- [API Reference](./api-reference.md) - Complete API documentation
