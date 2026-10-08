<!-- {
  "path": "jqr",
  "name": "docs-jqr-index",
  "meta": {
    "title": "Overview",
    "order": 1
  }
} -->

# JQR: jQuery Reactive

JQR (jQuery Reactive) is a lightweight reactive framework that brings modern reactivity to jQuery. Build dynamic, data-driven interfaces using familiar HTML templates enhanced with reactive directives.

## What is JQR?

JQR extends jQuery with a **reactive component system** inspired by Vue.js, but designed to work seamlessly with existing jQuery codebases and vanilla HTML.

**Key Features:**
- ⚡ **Reactive State** - Automatic UI updates when data changes
- 🎯 **Directive-Based** - Declarative bindings in HTML (`$:text`, `$:if`, `$:for`, `$:model`)
- 🔌 **Progressive** - Drop into existing projects, no build step required
- 🪶 **Lightweight** - ~15KB minified + gzipped
- 📦 **No Dependencies** - Works standalone or with jQuery 4.0+
- 🧩 **Component System** - Reusable, composable UI components
- 🎨 **Template-Based** - Write HTML, not JavaScript (unless you want to)

## Quick Start

### Browser (CDN)

```html
<!DOCTYPE html>
<html>
<head>
  <title>JQR Quick Start</title>
</head>
<body>
  <div id="app">
    <h1 $:text="message"></h1>
    <input $:model="message" placeholder="Type something...">
    <p>Character count: <code $:text="message.length"></code></p>
  </div>

  <!-- Load JQR -->
  <script src="https://cdn.jsdelivr.net/npm/@en32/jqr@latest/dist/jqr.min.js"></script>

  <script>
    // Initialize component
    $('#app').component({
      state: {
        message: 'Hello, JQR!'
      }
    });
  </script>
</body>
</html>
```

### Module Usage (ESM)

```bash
npm install @en32/jqr
```

```js
import { $, jqr } from '@en32/jqr';

$('#app').component({
  state: {
    message: 'Hello, JQR!'
  }
});
```

## Core Concepts

### 1. Reactive State

State changes automatically update the DOM:

```html
<div id="counter">
  <p>Count: <span $:text="count"></span></p>
  <button $:click="count++">Increment</button>
</div>

<script>
$('#counter').component({
  state: { count: 0 }
});
</script>
```

When `count` changes, the UI updates instantly. No manual DOM manipulation needed!

### 2. Directives

Bind data to DOM with special `$:` attributes:

| Directive | Purpose | Example |
|-----------|---------|---------|
| `$:text` | Set text content | `<p $:text="message"></p>` |
| `$:if` | Conditional rendering | `<div $:if="isVisible">...</div>` |
| `$:for` | List rendering | `<li $:for="item in items"></li>` |
| `$:model` | Two-way binding | `<input $:model="name">` |
| `$:click` | Event handling | `<button $:click="save()">Save</button>` |
| `$:bind` | Attribute binding | `<div $:bind="{ class: 'active', style: {...} }"></div>` |

See [Directives Guide](./guides/directives.md) for complete list.

### 3. Components

Encapsulate UI logic into reusable components:

```html
<template id="user-card">
  <div class="card">
    <h3 $:text="user.name"></h3>
    <p $:text="user.email"></p>
  </div>
</template>

<div id="app">
  <div $:component="user-card"></div>
</div>

<script>
$.jqr.register('user-card', {
  state: {
    user: { name: 'Alice', email: 'alice@example.com' }
  }
});

$('#app').component();
</script>
```

### 4. Computed Values

Derive values from state:

```js
$('#app').component({
  state: {
    firstName: 'John',
    lastName: 'Doe'
  },
  computed: {
    fullName: () => `${this.firstName} ${this.lastName}`
  }
});
```

### 5. Async State

Fetch data reactively:

```js
$('#app').component({
  asyncState: {
    async users() {
      const response = await fetch('/api/users');
      return response.json();
    }
  }
});
```

Data loads automatically when component mounts, with `usersLoading` and `usersError` states built-in.

## Live Demo

```html live:no-code
<div id="demo-overview" class="en-card">
  <div class="en-card__body">
    <h3>Interactive Todo List</h3>
    <div class="en-flex en-gap-2 en-mb-3">
      <input class="en-input" $:model="newTodo" placeholder="Add a todo..." $:keyup.enter="addTodo()">
      <button class="en-btn en-btn--primary" $:click="addTodo()">Add</button>
    </div>
    <div class="en-alert en-alert--info" $:if="todos.length === 0">
      No todos yet. Add one above!
    </div>
    <ul class="en-list" $:if="todos.length > 0">
      <li class="en-flex en-items-center en-gap-2 en-mb-2" $:for="todo in todos" $:key="todo.id">
        <input type="checkbox" $:model="todo.done">
        <span $:text="todo.text" $:style="{ textDecoration: todo.done ? 'line-through' : 'none', opacity: todo.done ? 0.6 : 1 }"></span>
        <button class="en-btn en-btn--sm en-btn--danger en-ml-auto" $:click="removeTodo(todo.id)">Remove</button>
      </li>
    </ul>
    <p class="en-text--muted en-mt-3">
      <strong $:text="remaining"></strong> remaining, 
      <strong $:text="completed"></strong> completed
    </p>
  </div>
</div>
```

```html
<div id="demo-overview">
  <h3>Interactive Todo List</h3>
  <div>
    <input $:model="newTodo" placeholder="Add a todo..." $:keyup.enter="addTodo()">
    <button $:click="addTodo()">Add</button>
  </div>
  <div $:if="todos.length === 0">No todos yet. Add one above!</div>
  <ul $:if="todos.length > 0">
    <li $:for="todo in todos" $:key="todo.id">
      <input type="checkbox" $:model="todo.done">
      <span $:text="todo.text" $:style="{ textDecoration: todo.done ? 'line-through' : 'none' }"></span>
      <button $:click="removeTodo(todo.id)">Remove</button>
    </li>
  </ul>
  <p>
    <strong $:text="remaining"></strong> remaining, 
    <strong $:text="completed"></strong> completed
  </p>
</div>
```

```js inject show
$('#demo-overview').component({
  state: {
    newTodo: '',
    todos: []
  },
  computed: {
    remaining: (ctx) => ctx.todos.filter(t => !t.done).length,
    completed: (ctx) => ctx.todos.filter(t => t.done).length
  },
  methods: {
    addTodo() {
      if (this.newTodo.trim()) {
        this.todos.push({
          id: Date.now(),
          text: state.newTodo,
          done: false
        });
        this.newTodo = '';
      }
    },
    removeTodo(id) {
      const index = this.todos.findIndex(t => t.id === id);
      if (index > -1) this.todos.splice(index, 1);
    }
  }
});
```

```js cleanup
onUnmount(() => {
  $('#demo-overview').component('destroy');
})
```

## When to Use JQR?

**✅ Perfect for:**
- Adding interactivity to existing jQuery projects
- Building dynamic forms and dashboards
- Server-rendered apps needing client-side reactivity
- Progressive enhancement of static HTML
- Projects where build tools are impractical
- Developers comfortable with HTML/jQuery but want modern reactivity

**⚠️ Consider Alternatives:**
- Large SPAs with complex routing → Vue, React, or Angular
- Mobile apps → React Native, Flutter
- Need server-side rendering → Next.js, Nuxt
- Team prefers JSX/TSX → React with TypeScript

JQR is designed for **incremental adoption** - start small, grow as needed.

## Component Options

A complete JQR component can include:

```js
$('#app').component({
  // Reactive state
  state: {
    count: 0,
    user: null
  },
  
  // Props (for nested components)
  props: {
    userId: null
  },
  
  // Computed values
  computed: {
    doubleCount: (s) => s.count * 2
  },
  
  // Methods
  methods: {
    increment(state) {
      state.count++;
    }
  },
  
  // Async data fetching
  asyncState: {
    async user() {
      const res = await fetch(`/api/users/${this.$props.userId}`);
      return res.json();
    }
  },
  
  // Lifecycle hooks
  created() {
    console.log('Component created');
  },
  
  mounted() {
    console.log('Component mounted, DOM ready');
  },
  
  updated() {
    console.log('Component updated');
  },
  
  destroyed() {
    console.log('Component destroyed');
  },
  
  // Watchers
  watch: {
    count(newVal, oldVal) {
      console.log(`Count changed: ${oldVal} → ${newVal}`);
    }
  }
});
```

See [Component Options Guide](./guides/component-options.md) for details.

## Architecture

JQR uses **direct DOM binding + fine-grained reactivity** (similar to Alpine.js/SolidJS):

1. **State** is made reactive using ES6 Proxies
2. **Directives** create fine-grained `effect()` bindings on real DOM nodes
3. When state changes, only affected effects re-run and update their specific nodes
4. **No Virtual DOM** - No re-rendering, no diffing, no tree reconciliation
5. Each directive binding updates independently - surgical, not tree-based

**Philosophy:** Progressive enhancement over real HTML (not compiled templates or JSX). Your markup is the source of truth, enhanced with reactive directives.

This architecture makes JQR extremely fast and memory-efficient - updates are proportional to the number of changed values, not the size of the component tree.

## Ecosystem

### Core Libraries

- **@en32/jqr** - Core framework (this package)
- **@en32/jqr-h** - Render functions & hyperscript API (optional)

### Plugins

- **@en32/jqr-i18n** - Internationalization
- **@en32/jqr-validate** - Form validation

### Compatibility

- Works with or without jQuery 4.0+
- Built-in `$` fallback for basic DOM operations
- Compatible with all modern browsers (ES6+)

## Documentation

### Getting Started
- [Installation](./installation.md) - Setup and configuration

### Guides
- [Reactivity](./guides/reactivity.md) - How reactivity works
- [Directives](./guides/directives.md) - All built-in directives
- [Component Options](./guides/component-options.md) - Complete options reference
- [Lifecycle](./guides/lifecycle.md) - Component lifecycle hooks
- [Events](./guides/events.md) - Event handling
- [DOM Binding](./guides/dom-binding.md) - Text, attributes, classes, styles
- [Forms & Models](./guides/forms-and-models.md) - Form binding & two-way data
- [Lists & Conditionals](./guides/lists-and-conditionals.md) - `$:for` and `$:if`
- [Props](./guides/props.md) - Component props system
- [Slots](./guides/slots.md) - Content distribution
- [Async State](./guides/async-state.md) - Data fetching
- [State Management](./guides/state-management.md) - State patterns
- [Transitions](./guides/transitions.md) - Animations
- [Teleport](./guides/teleport.md) - Portal rendering

### Reference
- [Directive Reference](./reference/directive-reference.md) - Complete directive API
- [Modifier Reference](./reference/modifier-reference.md) - All modifiers

### Examples
- [Todo App](./examples/todo-app.md) - Complete example
- [AJAX Form](./examples/ajax-form.md) - Form with validation
- [Modal Dialog](./examples/modal-dialog.md) - Modal patterns

## Philosophy

JQR embraces:

1. **Progressive Enhancement** - Start simple, add complexity as needed
2. **HTML First** - Write templates in HTML, not JavaScript
3. **Familiar Patterns** - If you know jQuery and Vue, you know JQR
4. **No Magic** - Clear, predictable behavior
5. **Flexibility** - Use with or without jQuery, build tools, or frameworks

## Performance

**Benchmarks (operations/sec):**
- State updates: ~500,000 ops/sec
- DOM updates: ~50,000 ops/sec
- Component creation: ~10,000 ops/sec

**Why so fast?**

JQR's direct DOM binding approach has no Virtual DOM overhead:
- No tree re-rendering on state change
- No vnode object allocation or garbage collection
- No diffing algorithm (except for keyed `$:for` lists)
- Updates are surgical - only the specific nodes that depend on changed values are touched

This makes JQR particularly efficient for small-to-medium updates and progressive enhancement scenarios.

## Browser Support

- Chrome 60+
- Firefox 55+
- Safari 11+
- Edge 79+

Requires ES6 Proxy support. No IE11 support.

## TypeScript Support

JQR includes TypeScript definitions:

```typescript
import { Component, ComponentOptions } from '@en32/jqr';

interface AppState {
  count: number;
  message: string;
}

const options: ComponentOptions<AppState> = {
  state: { count: 0, message: 'Hello' },
  computed: {
    doubled: (ctx) => ctx.count * 2
  }
};

$('#app').component(options);
```

## Contributing

JQR is open source (MIT License). Contributions welcome!

- **GitHub:** https://github.com/enix32/jqr
- **Issues:** https://github.com/enix32/jqr/issues
- **Discussions:** https://github.com/enix32/jqr/discussions

## Next Steps

**New to JQR?**
1. [Installation](./installation.md) - Get started
2. [Reactivity Guide](./guides/reactivity.md) - Learn the basics
3. [Directives Overview](./guides/directives.md) - Master the syntax

**Building something?**
- [Component Options](./guides/component-options.md) - Full component API
- [Examples](./examples/todo-app.md) - Real-world patterns
- [State Management](./guides/state-management.md) - Scale your app

**Going deeper?**
- [@en32/jqr-h](../jqr-h/README.md) - Render functions (optional)
- [Custom Directives](./guides/custom-directives.md) - Extend JQR
- [Performance Guide](./guides/performance.md) - Optimize

## Version

Current version: **1.0.0-dev**

See [CHANGELOG](../../CHANGELOG.md) for release history.

---

**Happy coding with JQR!** 🚀
