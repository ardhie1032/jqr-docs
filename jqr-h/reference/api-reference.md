<!-- {
  "path": "jqr-h/reference/api-reference",
  "name": "docs-jqr-h-reference-api-reference",
  "meta": {
    "title": "API Reference",
    "order": 2
  }
} -->

# API Reference

Complete API documentation for JQR-H (`@en32/jqr-h`).

## Core Functions

### h()

Builds a virtual node description. No DOM side effects until rendered.

```typescript
function h<S extends Dict = Dict>(
  tag: string,
  attrs?: NodeAttrs<S> | null,
  children?: ChildrenArg<S>
): JqrNode<S>
```

**Parameters:**
- `tag`: HTML tag name or registered component name
- `attrs`: Attributes and bindings (optional)
- `children`: Child nodes (optional)

**Returns:** `JqrNode<S>` - Virtual node description

**Examples:**

```js
// Plain HTML element
h('div', { class: 'card' }, ['Hello'])

// With reactive bindings
h('p', { 
  text: (ctx) => ctx.message,
  class: (ctx) => ({ active: ctx.isActive })
})

// Registered component
h('user-card', {
  props: { userId: (ctx) => ctx.currentUser }
})
```

### h() - Component Reference

Build component node with direct reference (typed props).

```typescript
function h<S extends Dict = Dict, P extends Dict = Dict>(
  componentRef: ComponentOptions<any, P>,
  attrs?: NodeAttrs<S> & {
    props: { [K in keyof P]: StateFn<S, P[K]> };
  } | null,
  children?: ChildrenArg<S>
): JqrNode<S>
```

**Parameters:**
- `componentRef`: Component definition object
- `attrs`: Attributes with typed props
- `children`: Child nodes

**Example:**

```js
const UserCard = {
  props: { userId: 0, name: '' },
  template: '<div>...</div>'
};

// TypeScript checks props against UserCard.props
h(UserCard, {
  props: {
    userId: (ctx) => ctx.currentId,
    name: (ctx) => ctx.userName
  }
})
```

### h.for()

Builds a keyed list node.

```typescript
function h.for<S extends Dict = Dict, T = any>(
  items: StateFn<S, T[]>,
  key: (item: T, index: number) => any,
  build: (item: T, index: number) => JqrNode<any>
): ForNode<S, T>
```

**Parameters:**
- `items`: Function returning array of items
- `key`: Function extracting unique key from each item
- `build`: Function building node for each item

**Returns:** `ForNode<S, T>` - Keyed list description

**Example:**

```js
h.for(
  (ctx) => ctx.users,
  (user) => user.id,
  (user) => h('div', { class: 'user' }, [
    h('h3', {}, [user.name]),
    h('p', {}, [user.email])
  ])
)
```

### h.slot()

Creates a slot outlet for parent-supplied content.

```typescript
function h.slot<S extends Dict = Dict>(
  name?: string,
  fallback?: Child<S>[]
): SlotNode<S>
```

**Parameters:**
- `name`: Slot name (default: `'default'`)
- `fallback`: Content shown when parent supplies nothing

**Returns:** `SlotNode<S>` - Slot outlet description

**Examples:**

```js
// Default slot
h.slot()

// Named slot with fallback
h.slot('header', [
  h('h2', {}, ['Default Header'])
])

// Usage in component
render: (h, ctx) => h('div', { class: 'modal' }, [
  h('div', { class: 'modal-header' }, [h.slot('header')]),
  h('div', { class: 'modal-body' }, [h.slot()]),
  h('div', { class: 'modal-footer' }, [h.slot('footer')])
])
```

## Mounting Functions

### mount()

Mounts a render-based component onto a target element.

```typescript
function mount<S extends Dict = Dict, P extends Dict = Dict>(
  target: string | HTMLElement | JQuery,
  options: RenderComponentOptions<S, P>,
  mountOptions?: MountOptions
): JqrComponent<S, P>
```

**Parameters:**
- `target`: Target element selector, HTMLElement, or jQuery object
- `options`: Component options with `render()` function
- `mountOptions`: Optional mount configuration

**Returns:** `JqrComponent<S, P>` - Component instance

**⚠️ Important:** Target's existing content is **cleared first**. `render()` fully owns the container.

**Example:**

```js
import { mount } from '@en32/jqr-h';

const instance = mount('#app', {
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
});

// Access instance
console.log(instance.$state.count);
instance.$destroy();
```

### unmount()

Tears down a component previously mounted with `mount()`.

```typescript
function unmount(target: string | HTMLElement | JQuery): void
```

**Parameters:**
- `target`: Target element (same value passed to `mount()`)

**Returns:** `void`

**Example:**

```js
import { mount, unmount } from '@en32/jqr-h';

// Mount
mount('#app', { render: (h) => h('div', {}, ['Hello']) });

// Later: unmount
unmount('#app');
```

### renderNode()

Renders a virtual node into a real HTMLElement. Primarily internal.

```typescript
function renderNode<S extends Dict = Dict>(
  node: JqrNode<S>,
  instance: JqrComponent<S, any>,
  disposers: Disposer[]
): HTMLElement
```

**Parameters:**
- `node`: Virtual node description
- `instance`: Owning component instance
- `disposers`: Array collecting cleanup functions

**Returns:** `HTMLElement` - Rendered DOM element

**Note:** Usually not called directly. Used internally by `mount()` and component rendering.

## Node Attributes

### class

Sets element's CSS classes. Can be static or reactive.

```typescript
class?: Getter<S, any>
```

**Accepts:**
- `string`: Space-separated class names
- `object`: `{ [className]: boolean }` map
- `array`: Mix of strings and objects
- `function`: Returning any of the above

**Examples:**

```js
// Static string
{ class: 'card active' }

// Reactive object
{ class: (ctx) => ({
  active: ctx.isActive,
  disabled: ctx.isDisabled
}) }

// Mixed array
{ class: (ctx) => ['card', { active: ctx.isActive }] }
```

### style

Sets element's inline styles.

```typescript
style?: Getter<S, any>
```

**Accepts:**
- `object`: `{ property: value }` (camelCase or kebab-case)
- `string`: Raw CSS string
- `array`: Mix of objects and strings
- `function`: Returning any of the above

**Examples:**

```js
// Static object
{ style: { color: 'red', fontSize: '14px' } }

// Reactive
{ style: (ctx) => ({
  color: ctx.themeColor,
  display: ctx.isVisible ? 'block' : 'none'
}) }

// CSS string
{ style: 'color: red; font-size: 14px;' }
```

### attrs

Sets arbitrary HTML attributes.

```typescript
attrs?: Dict<Getter<S, any>>
```

**Values:**
- `null`/`undefined`/`false`: Removes attribute
- `true`: Sets empty value
- Other: Sets as attribute value

**Example:**

```js
{
  attrs: {
    href: (ctx) => ctx.profileUrl,
    disabled: (ctx) => ctx.isLoading,
    'data-id': (ctx) => ctx.userId,
    'aria-label': (ctx) => ctx.label
  }
}
```

### text

Sets element's `textContent`. Auto-stringifies non-strings.

```typescript
text?: StateFn<S, any>
```

**Example:**

```js
// String
{ text: (ctx) => ctx.message }

// Number (auto-stringified)
{ text: (ctx) => ctx.count }

// Object (auto-stringified as JSON)
{ text: (ctx) => ctx.userData }
```

### html

Sets element's `innerHTML`. **Beware XSS** - only use with trusted content.

```typescript
html?: StateFn<S, any>
```

**Example:**

```js
{
  html: (ctx) => sanitizedHtmlContent
}
```

### show

Toggles `display` style. Element stays mounted.

```typescript
show?: StateFn<S, boolean>
```

**Example:**

```js
{
  show: (ctx) => ctx.isVisible
}
```

### if

Conditional mount/unmount. Fully removes from DOM when false.

```typescript
if?: StateFn<S, boolean>
```

**Example:**

```js
{
  if: (ctx) => ctx.shouldRender
}
```

**Note:** If both `show` and `if` are set, `show` wins (with dev warning).

### on

DOM event listeners (static, attached once).

```typescript
on?: Dict<(ctx: Ctx<S>, ev: Event) => void>
```

**Example:**

```js
{
  on: {
    click: (ctx, ev) => {
      ev.preventDefault();
      ctx.handleClick();
    },
    input: (ctx, ev) => {
      ctx.query = ev.target.value;
    },
    keydown: (ctx, ev) => {
      if (ev.key === 'Enter') ctx.submit();
    }
  }
}
```

### emit

Listens for child component's custom events (components only).

```typescript
emit?: Dict<(ctx: Ctx<S>, payload: any) => void>
```

**Example:**

```js
h('user-form', {
  emit: {
    submit: (ctx, userData) => {
      ctx.saveUser(userData);
    },
    cancel: (ctx) => {
      ctx.showForm = false;
    }
  }
})
```

### props

Passes props to nested component (components only).

```typescript
props?: Dict<StateFn<S, any>>
```

**Example:**

```js
h('user-card', {
  props: {
    userId: (ctx) => ctx.currentUserId,
    theme: (ctx) => ctx.appTheme,
    editable: (ctx) => ctx.isAdmin
  }
})
```

### model

Two-way binding. Five forms depending on context.

```typescript
model?: string | Dict<string> | NamedModelBinding | FormModelBinding | ComputedModelBinding<S>
```

**Form 1: String (form controls)**

```js
// Binds to state.username
{ model: 'username' }
```

**Form 2: FormModelBinding (with timing modifiers)**

```js
{
  model: {
    state: 'searchQuery',
    debounce: 400
  }
}
```

**Form 3: ComputedModelBinding (nested paths)**

```js
{
  model: {
    get: (ctx) => ctx.$state.user.name,
    set: (ctx, value) => { ctx.$state.user.name = value; },
    debounce: true
  }
}
```

**Form 4: NamedModelBinding (DOM properties)**

```js
// For <dialog>, <details>, custom elements
{
  model: {
    prop: 'open',
    state: 'isOpen',
    on: 'toggle'
  }
}
```

**Form 5: Dict<string> (component two-way props)**

```js
h('modal', {
  model: {
    open: 'showModal'  // Sugar for props + emit
  }
})
// Equivalent to:
// props: { open: (ctx) => ctx.showModal }
// emit: { 'update:open': (ctx, v) => { ctx.showModal = v } }
```

### ref

Saves rendered element in `instance.$refs[name]`.

```typescript
ref?: string
```

**Example:**

```js
render: (h, ctx) => h('div', {}, [
  h('input', { ref: 'searchInput' }),
  h('button', {
    on: {
      click: (ctx) => {
        ctx.$refs.searchInput.focus();
      }
    }
  }, ['Focus Input'])
])
```

### directive

Custom directives registered with `$.jqr.directive()`.

```typescript
directive?: Dict<Getter<S, any>>
```

**Example:**

```js
// Assuming $.jqr.directive('tooltip', ...)
{
  directive: {
    'tooltip': (ctx) => ctx.helpText,
    'tooltip.top': 'Top tooltip',
    'focus:immediate': true
  }
}
```

### bind

Shorthand spreading multiple bindings.

```typescript
bind?: Getter<S, Dict<any>>
```

**Example:**

```js
{
  bind: (ctx) => ({
    class: 'card',
    attrs: { 'data-id': ctx.id },
    text: ctx.label
  })
}
```

### teleport

Moves element to another position in document.

```typescript
teleport?: Getter<S, string | { to: string; mode?: TeleportMode; defer?: boolean }>
```

**Examples:**

```js
// String form (append mode)
{ teleport: '#modal-container' }

// Object form
{
  teleport: {
    to: '#dropdown-root',
    mode: 'append',
    defer: true
  }
}

// Reactive
{
  teleport: (ctx) => ({
    to: ctx.teleportTarget,
    mode: 'prepend'
  })
}
```

## MountOptions

Optional configuration for `mount()`.

```typescript
interface MountOptions {
  slots?: SlotGroups;
}
```

**Properties:**
- `slots`: Slot content supplied by parent (rarely used directly)

## Integration with Core JQR

JQR-H components are full JQR components with all the same features:

**Available in render context:**
- `ctx.$state` - Reactive state
- `ctx.$props` - Props
- `ctx.$refs` - Element references
- `ctx.$emit()` - Emit events
- `ctx.$watch()` - Watch state
- `ctx.$destroy()` - Destroy component
- `ctx.$refetch()` - Refetch async state
- `ctx.$setProps()` - Update props
- All state/computed/methods/constants directly

**Example:**

```js
{
  state: { count: 0 },
  computed: {
    doubled() {
      return this.count * 2;
    }
  },
  methods: {
    increment() {
      this.count++;
    }
  },
  watch: {
    count(newVal, oldVal) {
      console.log(`Count changed from ${oldVal} to ${newVal}`);
    }
  },
  render: (h, ctx) => h('div', {}, [
    h('p', { text: (c) => `Count: ${c.count}` }),
    h('p', { text: (c) => `Doubled: ${c.doubled}` }),
    h('button', {
      on: { click: (c) => c.increment() }
    }, ['Increment'])
  ])
}
```

## Browser/IIFE Usage

```html
<script src="https://cdn.jsdelivr.net/npm/@en32/jqr@latest/dist/jqr.min.js"></script>
<script src="https://cdn.jsdelivr.net/npm/@en32/jqr-h@latest/dist/jqr-h.min.js"></script>

<script>
  const { h, mount } = JQRH;
  
  mount('#app', {
    state: { message: 'Hello' },
    render: (h, ctx) => h('div', {
      text: (c) => c.message
    })
  });
</script>
```

## ESM/Bundler Usage

```js
import { h, mount, unmount } from '@en32/jqr-h';
import type { JqrNode, NodeAttrs, RenderComponentOptions } from '@en32/jqr-h';

const options: RenderComponentOptions = {
  render: (h, ctx) => h('div', {}, ['Hello'])
};

const instance = mount('#app', options);
```

## See Also

- [h() Function Guide](../guides/h-function.md) - Detailed h() usage
- [Render Functions Guide](../guides/render-function.md) - Component rendering
- [TypeScript Types](./types.md) - Type definitions
- [Render Context](./render-context.md) - Context object reference
