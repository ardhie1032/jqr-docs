<!-- {
  "path": "jqr-h",
  "name": "docs-jqr-h-index",
  "meta": {
    "title": "Overview",
    "order": 1
  }
} -->

# JQR-H: Render Functions & Hyperscript

JQR-H is a core plugin that provides **render functions** and a **hyperscript API** (`h()`) for building components programmatically instead of using HTML templates.

## What is JQR-H?

JQR-H extends JQR with:

1. **`h()` function** - Hyperscript API for creating node descriptions
2. **`render()` option** - Build component UI with JavaScript/TypeScript
3. **Dynamic rendering** - Fully programmatic, type-safe component creation

## Why Render Functions?

While JQR's template-based approach (HTML with directives) is perfect for most use cases, render functions provide:

**Advantages:**
- **Full JavaScript power** - Use any JS logic, loops, conditionals
- **TypeScript support** - Strong typing for props, state, and nodes
- **Dynamic components** - Generate UI based on runtime data
- **No templates** - Pure JavaScript, no HTML parsing
- **Reusable builders** - Abstract common UI patterns into functions

**When to use:**
- Complex conditional rendering
- Highly dynamic UIs
- Component libraries
- Type-safe development
- Programmatic DOM generation

## Quick Example

**Template-based (standard JQR):**
```html
<template id="user-card">
  <div class="card">
    <h3 $:text="user.name"></h3>
    <p $:text="user.email"></p>
  </div>
</template>
```

**Render function (JQR-H):**
```js
JQRH.mount('#app', {
  state: { user: { name: 'Alice', email: 'alice@example.com' } },
  render: (h, ctx) => 
    h('div', { class: 'card' }, [
      h('h3', { text: (ctx) => ctx.user.name }),
      h('p', { text: (ctx) => ctx.user.email })
    ])
});
```

Both produce the same result, but render functions give you full JavaScript flexibility.

## Core API

### `h()` - Hyperscript Function

Create node descriptions (not Virtual DOM nodes):

```js
// Basic element
h('div', { class: 'container' }, [
  h('h1', { text: () => 'Hello' })
])

// With reactive attributes
h('p', { 
  text: (ctx) => ctx.message,
  class: (ctx) => ctx.isActive ? 'active' : ''
})

// Nested components
h('user-card', { 
  prop: { userId: (ctx) => ctx.currentUser }
})
```

### `h.for()` - List Rendering

Render lists efficiently:

```js
h.for(
  (state) => state.items,           // Items array
  (item) => item.id,                // Key function
  (item) => h('li', { text: () => item.name })  // Builder
)
```

### `render()` Option

Component option for render functions:

```js
JQRH.mount('#app', {
  state: { count: 0 },
  render: (h, ctx) => 
    h('div', {}, [
      h('p', { text: (c) => `Count: ${c.count}` }),
      h('button', { 
        on: { click: () => ctx.count++ },
        text: () => 'Increment'
      })
    ])
});
```

## Browser Usage (IIFE)

```html
<!-- Load JQR first -->
<script src="https://cdn.jsdelivr.net/npm/@en32/jqr@latest/dist/jqr.min.js"></script>

<!-- Load JQR-H -->
<script src="https://cdn.jsdelivr.net/npm/@en32/jqr-h@latest/dist/jqr-h.min.js"></script>

<div id="app"></div>

<script>
  // Global JQRH available
  JQRH.mount('#app', {
    state: { message: 'Hello from JQR-H!' },
    render: (h) => 
      h('h1', { text: (c) => c.message })
  });
</script>
```

## Module Usage (ESM)

```js
import { h, mount } from '@en32/jqr-h';

mount('#app', {
  state: { message: 'Hello!' },
  render: (h, ctx) => 
    h('h1', { text: (c) => c.message })
});
```

## Key Concepts

### Node Descriptions

`h()` creates **lightweight node descriptions** (plain objects), not Virtual DOM:

```js
const node = h('div', { class: 'card' });
// node is just an object: { tag: 'div', attrs: {...}, children: [] }
```

**Important:** JQR-H **does NOT use Virtual DOM**. Node descriptions are converted to real DOM once by `mount()` or `render()`. After that, updates are handled by **fine-grained reactivity**:

- Each dynamic attribute/text binding gets its own `effect()`
- When state changes, only the specific `effect()` re-runs and updates its DOM node
- **No re-rendering, no diffing, no reconciliation** - updates are surgical

This is similar to Alpine.js or SolidJS, not React/Vue's Virtual DOM approach.

### Reactive Getters

Attributes can be static or reactive:

```js
h('div', {
  class: 'static',                    // Static string
  text: (c) => c.message,     // Reactive getter
  style: (c) => ({ 
    color: c.isError ? 'red' : 'black' 
  })
})
```

Reactive getters re-run when state changes.

### Component Composition

Nest components with `h()`:

```js
h('div', {}, [
  h('header-component'),
  h('user-list', { props: { users: (c) => c.users } }),
  h('footer-component')
])
```

### Slots

Pass slot content to components:

```js
h('card', {}, {
  default: [
    h('p', { text: () => 'Card content' })
  ],
  footer: [
    h('button', { text: () => 'Save' })
  ]
})
```

## Complete Example

```html
<div id="todo-app"></div>

<script>
JQRH.mount('#todo-app', {
  state: {
    todos: [],
    newTodo: ''
  },
  methods: {
    addTodo() {
      if (this.newTodo.trim()) {
        this.todos.push({
          id: Date.now(),
          text: this.newTodo,
          done: false
        });
        this.newTodo = '';
      }
    },
    toggleTodo(id) {
      const todo = this.todos.find(t => t.id === id);
      if (todo) todo.done = !todo.done;
    }
  },
  render: (h, ctx) => 
    h('div', {}, [
      h('h1', { text: () => 'Todo App' }),
      
      // Input form
      h('div', {}, [
        h('input', {
          model: 'newTodo',
          attr: { placeholder: 'Enter todo...' }
        }),
        h('button', {
          on: { click: () => this.methods.addTodo(state) },
          text: 'Add'
        })
      ]),
      
      // Todo list
      h.for(
        (s) => s.todos,
        (todo) => todo.id,
        (todo) => h('div', {}, [
          h('input', {
            attr: { type: 'checkbox' },
            model: { get: () => todo.done, set: (v) => todo.done = v }
          }),
          h('span', {
            text: todo.text,
            style: () => ({
              textDecoration: todo.done ? 'line-through' : 'none'
            })
          })
        ])
      )
    ])
});
</script>
```

## When to Use Templates vs Render Functions

**Use Templates (standard JQR):**
- Simple, static layouts
- Designer-friendly HTML
- Quick prototyping
- Most common use cases

**Use Render Functions (JQR-H):**
- Complex conditional logic
- Highly dynamic UIs
- TypeScript projects
- Component libraries
- Need full JS power

**Use Both:**
- Templates for layout, render functions for complex widgets
- Mix and match as needed

## Performance

Render functions use **direct DOM binding + fine-grained reactivity**:
- **No Virtual DOM** - No re-rendering or diffing of component trees
- **Surgical updates** - Each binding (`text`, `class`, etc.) updates independently via `effect()`
- **Keyed list tracking** - `h.for()` uses key-based DOM node tracking (not vnode diffing)
- **Single build pass** - DOM is constructed once from `h()` calls, then updated in-place

This approach is similar to Alpine.js or SolidJS - the overhead per update scales with the number of changed values, not the size of your component.

## TypeScript Support

JQR-H has full TypeScript support:

```ts
import { h, mount, RenderComponentOptions } from '@en32/jqr-h';

interface State {
  count: number;
  message: string;
}

const options: RenderComponentOptions<State> = {
  state: { count: 0, message: 'Hello' },
  render: ({ h, state }) => 
    h('div', {}, [
      h('p', { text: (s) => `Count: ${s.count}` })
    ])
};

mount('#app', options);
```

## Next Steps

- [Installation](./installation.md) - Setup JQR-H
- [Render Functions](./guides/render-function.md) - Learn `render()` option
- [h() Function](./guides/h-function.md) - Master hyperscript API
- [Components with h()](./guides/components-with-h.md) - Build reusable components

## Package Info

- **Package:** `@en32/jqr-h`
- **Dependencies:** Requires `@en32/jqr`
- **Size:** ~8KB minified + gzipped
- **License:** MIT
