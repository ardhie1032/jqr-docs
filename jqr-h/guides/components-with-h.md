<!-- {
  "path": "jqr-h/guides/components-with-h",
  "name": "docs-jqr-jqrh-guides-components-with-h",
  "meta": {
    "title": "Components with h()",
    "order": 3
  }
} -->

# Components with h()

Building nested components with `h()` allows you to create reusable, composable UI pieces. This guide covers component composition, props, events, and slots using render functions.

## Component Basics

Components can be rendered two ways:

1. **By registry name** - String matching `$.jqr.register()` registry
2. **By reference** - Pass component definition object directly

### By Registry Name

First register the component:

```js
$.jqr.register('user-card', {
  props: { userId: null },
  state: { user: null },
  asyncState: {
    async user() {
      const response = await fetch(`/api/users/${this.props.userId}`);
      return response.json();
    }
  },
  render: (h, ctx) => 
    h('div', { class: 'en-card' }, [
      ctx.state.user
        ? h('div', {}, [
            h('h3', { text: () => ctx.state.user.name }),
            h('p', { text: () => ctx.state.user.email })
          ])
        : h('p', { text: () => 'Loading...' })
    ])
});
```

Then use it:

```js
h('user-card', {
  props: {
    userId: (c) => c.currentUserId
  }
})
```

### By Component Reference

Pass the definition directly (no registration needed):

```js
const UserCard = {
  props: { userId: null },
  state: { user: null },
  asyncState: {
    async user() {
      const response = await fetch(`/api/users/${this.props.userId}`);
      return response.json();
    }
  },
  render: (h, ctx) => 
    h('div', { class: 'en-card' }, [
      ctx.state.user
        ? h('div', {}, [
            h('h3', { text: () => ctx.user.name }),
            h('p', { text: () => ctx.user.email })
          ])
        : h('p', { text: () => 'Loading...' })
    ])
};

// Use directly
h(UserCard, {
  props: {
    userId: (c) => c.currentUserId
  }
})
```

**Advantages of component reference:**
- **Type safety** - TypeScript checks props against definition
- **No name conflicts** - No global registry pollution
- **Explicit imports** - Clear dependencies

## Passing Props

Props are passed via the `prop` attribute with reactive getters:

```html live:no-code
<div id="demo-props" class="en-card en-p-3">
  <!-- Rendered by JQR-H -->
</div>
```

```html
<div id="demo-props">
  <!-- Rendered by JQR-H -->
</div>
```

```js inject show
// Child component
const Greeting = {
  props: { name: 'Guest', color: 'black' },
  render: (h, ctx) => 
    h('p', {
      style: () => ({ color: ctx.$props.color }),
      text: () => `Hello, ${ctx.$props.name}!`
    })
};

// Parent component
JQRH.mount('#demo-props', {
  state: { 
    userName: 'Alice',
    themeColor: '#3b82f6'
  },
  render: (h, ctx) => 
    h('div', {}, [
      h(Greeting, {
        props: {
          name: (c) => c.userName,
          color: (c) => c.themeColor
        }
      }),
      h('input', {
        class: 'en-input en-mt-2',
        attrs: { placeholder: 'Enter name' },
        model: { state: 'userName', debounce: 300 },
      }),
      h('input', {
        class: 'en-input en-mt-2',
        attrs: { type: 'color' },
        model: 'themeColor'
      })
    ])
});
```

```js cleanup
onUnmount(() => {
  JQRH.unmount('#demo-props');
})
```

**Key points:**
- Props use **arrow functions** for reactivity
- Child receives props via `ctx.props` or `ctx.state.props`
- Props update automatically when parent state changes

## Component Events

Child components emit events via `this.emit()`, parent listens via `emit` attribute:

```html live:collapse
<div id="demo-events" class="en-card en-p-3">
  <!-- Rendered by JQR-H -->
</div>
```

```js inject show
// Counter component that emits events
const Counter = {
  props: { count: 0 },
  state: { say: 'hello' },
  methods: {
    increment() {
      const newCount = this.$props.count + 1;
      this.emit('update', newCount);
    },
    decrement() {
      const newCount = this.$props.count - 1;
      this.emit('update', newCount);
    }
  },
  render: (h, ctx) => 
    h('div', { class: 'en-flex en-gap-2 en-items-center' }, [
      h('button', {
        class: 'en-btn',
        on: { click: () => ctx.decrement() },
        text: () => '-'
      }),
      h('span', { 
        class: 'en-form-display',
        text: () => `Total: ${ctx.$props.count}` 
      }),
      h('button', {
        class: 'en-btn',
        on: { click: () => ctx.increment() },
        text: () => '+'
      })
    ])
};

// Parent listens to events
JQRH.mount('#demo-events', {
  state: { total: 0 },
  render: (h, ctx) => 
    h('div', {}, [
      h('p', { 
        class: 'en-mb-3',
        text: (c) => `Total: ${c.total}` 
      }),
      h(Counter, {
        props: {
          count: (c) => c.total
        },
        emit: {
          update: (c, newCount) => {
            c.total = newCount;
          }
        }
      })
    ])
});
```

```js cleanup
onUnmount(() => {
  JQRH.unmount('#demo-events');
})
```

**Event pattern:**
1. Child calls `this.emit('eventName', payload)`
2. Parent listens via `emit: { eventName: (state, payload) => {...} }`
3. Payload can be any data

## Two-Way Binding (v-model Pattern)

For components with two-way bindings, use the `model` attribute (Mode A):

```html live
<div id="demo-model" class="en-card">
  <!-- Rendered by JQR-H -->
</div>
```

```js inject show
// Custom input component
const CustomInput = {
  props: { value: '' },
  methods: {
    handleInput(event) {
      console.log(event)
      this.$emit('update:value', event.target.value);
    }
  },
  render: (h, ctx) => 
    h('input', {
      class: 'en-input',
      attrs: { 
        value: () => ctx.$props.value,
        placeholder: 'Type something...'
      },
      /** 
      on: {
        input: (_ctx, event) => ctx.handleInput(event),
      },
      on: {
        input(_ctx, event) {
          return ctx.handleInput(event)
        },
      },
      */
      on: {
        input: (_ctx, event) => {
          console.log(_ctx.props);
          ctx.emit('update:value', event.target.value)
        }
      }
    })
};

// Parent with model binding
JQRH.mount('#demo-model', {
  state: { text: '' },
  render: (h, ctx) => 
    h('div', {}, [
      h(CustomInput, {
        model: {
          value: 'text'  // Maps prop 'value' to state 'text'
        }
      }),
      h('p', { 
        class: 'en-mt-2',
        text: (c) => `You typed: ${c.text || '(nothing)'}` 
      })
    ])
});
```

```js cleanup
onUnmount(() => {
  JQRH.unmount('#demo-model');
})
```

**`model` is sugar for:**
```js
// This:
h(CustomInput, {
  model: { value: 'text' }
})

// Is equivalent to:
h(CustomInput, {
  props: {
    value: (c) => c.text
  },
  emit: {
    'update:value': (ctx, newValue) => {
      ctx.text = newValue;
    }
  }
})
```

**Convention:** Child emits `update:propName` for two-way props.

## Component Slots

Slots allow passing content to child components:

```html live:no-code
<div id="demo-slots" class="en-card">
  <!-- Rendered by JQR-H -->
</div>
```

```html
<div id="demo-slots">
  <!-- Rendered by JQR-H -->
</div>
```

```js inject show
// Register card component with slots
$.jqr.register('card-component', {
  template: `
    <div class="en-card">
      <header class="en-card__header">
        <h3 class="en-card__title"><slot $:name="header">Default Title</slot></h3>
      </header>
      <div class="en-card__body">
        <slot $:default>Default content</slot>
      </div>
      <footer class="en-card__footer">
        <slot $:name="footer"></slot>
      </footer>
    </div>
  `
});

// Parent passes slot content
JQRH.mount('#demo-slots', {
  state: { message: 'This is slot content!' },
  render: (h, ctx) => 
    h('card-component', {}, {
      header: [
        h('span', { text: () => 'Custom Header' })
      ],
      default: [
        h('p', { text: (s) => s.message })
      ],
      footer: [
        h('button', {
          class: 'en-btn en-btn--sm',
          on: { click: () => alert('Clicked!') },
          text: () => 'Action'
        })
      ]
    })
});
```

```js cleanup
onUnmount(() => {
  JQRH.unmount('#demo-slots');
})
```

**Slot syntax:**
```js
// Third argument is slots object
h('component-name', { /* attrs */ }, {
  default: [/* default slot content */],
  header: [/* named slot: header */],
  footer: [/* named slot: footer */]
})
```

**Notes:**
- Slots require component has `template` with `<slot>` outlets
- Default slot uses key `'default'`
- Slot content is reactive to **parent** state, not child state

## Complete Example: Todo List

```html live:no-code
<div id="demo-todo" class="en-card en-p-3">
  <!-- Rendered by JQR-H -->
</div>
```

```html
<div id="demo-todo">
  <!-- Rendered by JQR-H -->
</div>
```

```js inject show
// TodoItem component
const TodoItem = {
  props: { todo: null },
  render: (h, ctx) => 
    h('div', { 
      class: 'en-flex en-items-center en-gap-2 en-p-2 en-bg-gray-50 en-rounded en-mb-2'
    }, [
      h('label', { class: 'en-check' }, [
        h('input', {
          attrs: { 
            type: 'checkbox',
            checked: (c) => ctx.$props.todo.done
          },
          on: {
            change: (c, e) => {
              ctx.$emit('toggle', c.$props.todo.id);
            }
          }
        }),
      ]),
      h('span', {
        style: (c) => ({
          textDecoration: ctx.$props.todo.done ? 'line-through' : 'none',
          opacity: ctx.$props.todo.done ? '0.6' : '1',
          flex: '1'
        }),
        text: () => ctx.$props.todo.text
      }),
      h('button', {
        class: 'en-btn en-btn--sm en-btn--danger',
        on: { click: () => ctx.$emit('remove', ctx.$props.todo.id) },
        text: () => '×'
      })
    ])
};

// Main app
JQRH.mount('#demo-todo', {
  state: {
    newTodo: '',
    todos: [
      { id: 1, text: 'Learn JQR-H', done: true },
      { id: 2, text: 'Build components', done: false }
    ]
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
    },
    removeTodo(id) {
      this.todos = this.todos.filter(t => t.id !== id);
    }
  },
  render: (h, ctx) => 
    h('div', {}, [
      h('h4', { text: () => 'Todo List' }),
      
      // Add form
      h('div', { class: 'en-flex en-gap-2 en-mb-3' }, [
        h('input', {
          class: 'en-input',
          attrs: { placeholder: 'Add todo...' },
          model: 'newTodo',
          on: {
            keyup: (c, e) => {
              if (e.key === 'Enter') ctx.addTodo();
            }
          }
        }),
        h('button', {
          class: 'en-btn en-btn--sm en-btn--primary',
          on: { click: () => ctx.addTodo() },
          text: () => 'Add'
        })
      ]),
      
      // Todo list with components
      h.for(
        (s) => s.todos,
        (todo) => todo.id,
        (todo) => h(TodoItem, {
          props: {
            todo: () => todo
          },
          emit: {
            toggle: (c, id) => ctx.toggleTodo(id),
            remove: (c, id) => ctx.removeTodo(id)
          }
        })
      )
    ])
});
```

```js cleanup
onUnmount(() => {
  JQRH.unmount('#demo-todo');
})
```

This example demonstrates:
- Reusable `TodoItem` component
- Props passing (`todo` object)
- Event emission (`toggle`, `remove`)
- Component in `h.for()` list
- Parent methods handling events

## Component Composition Patterns

### Container/Presentational Pattern

**Presentational component** (pure UI):
```js
const UserCard = {
  props: { user: null },
  render: (h, ctx) => 
    h('div', { class: 'en-card' }, [
      h('h3', { text: (s) => s.props.user.name }),
      h('p', { text: (s) => s.props.user.email })
    ])
};
```

**Container component** (logic + data):
```js
const UserContainer = {
  props: { userId: null },
  state: { user: null, loading: true },
  asyncState: {
    async user() {
      const res = await fetch(`/api/users/${this.props.userId}`);
      return res.json();
    }
  },
  render: (h, ctx) => 
    ctx.state.user
      ? h(UserCard, {
          props: { user: () => ctx.state.user }
        })
      : h('p', { text: () => 'Loading...' })
};
```

### Higher-Order Component Pattern

Wrap components with additional functionality:

```js
function withLoading(Component) {
  return {
    props: { loading: false, ...Component.props },
    render: (h, ctx) => 
      ctx.props.loading
        ? h('div', { class: 'en-spinner' }, ['Loading...'])
        : h(Component, { props: ctx.props })
  };
}

const UserCardWithLoading = withLoading(UserCard);
```

### Render Prop Pattern

Pass render function as props:

```js
const DataFetcher = {
  props: { 
    url: '',
    renderData: null  // Function prop
  },
  state: { data: null, loading: true },
  created() {
    fetch(this.props.url)
      .then(r => r.json())
      .then(data => {
        this.state.data = data;
        this.state.loading = false;
      });
  },
  render: (h, ctx) => 
    ctx.state.loading
      ? h('p', { text: () => 'Loading...' })
      : ctx.props.renderData(h, ctx.state.data)
};

// Usage
h(DataFetcher, {
  props: {
    url: () => '/api/users',
    renderData: (h, users) => 
      h('ul', {}, users.map(u => 
        h('li', { text: () => u.name })
      ))
  }
})
```

## TypeScript Support

Component reference with type checking:

```typescript
interface UserCardProps {
  userId: number;
  showEmail?: boolean;
}

const UserCard: RenderComponentOptions<any, UserCardProps> = {
  props: { userId: 0, showEmail: true },
  state: { user: null },
  // ... implementation
};

// TypeScript checks props
h(UserCard, {
  props: {
    userId: (s) => s.currentUser,
    showEmail: (s) => s.settings.showEmail
    // Type error if prop doesn't exist!
  }
})
```

## Best Practices

### 1. Prop Validation

Define default props:

```js
const Button = {
  props: {
    label: 'Click me',
    variant: 'primary',
    disabled: false
  },
  render: (h, ctx) => 
    h('button', {
      class: ({props}) => `en-btn en-btn--${props.variant}`,
      attrs: { disabled: ({props}) => props.disabled },
      text: ({props}) => props.label
    })
};
```

### 2. Single Responsibility

Keep components focused:

```js
// ❌ Bad - too many responsibilities
const UserDashboard = {
  // Fetching, validation, UI, navigation...
};

// ✅ Good - focused components
const UserProfile = { /* Display user info */ };
const UserSettings = { /* Manage settings */ };
const Dashboard = { /* Compose others */ };
```

### 3. Prop Drilling Alternative

Use event bus or shared state for deeply nested data:

```js
// Instead of passing through many levels
const sharedState = JQR.reactive({ theme: 'dark' });

const DeepChild = {
  computed: {
    theme: () => sharedState.theme
  },
  render: (h, ctx) => 
    h('div', { text: (s) => s.computed.theme })
};
```

### 4. Component Registration

Register commonly-used components globally:

```js
// Register once
$.jqr.register('icon', IconComponent);
$.jqr.register('button', ButtonComponent);
$.jqr.register('card', CardComponent);

// Use everywhere by name
h('card', {}, [
  h('button', { props: { icon: () => 'save' } })
])
```

## Debugging Components

### Inspect Props

```js
const DebugProps = {
  props: { data: null },
  mounted() {
    console.log('Received props:', this.props);
  },
  render: (h, ctx) => 
    h('div', { text: (s) => JSON.stringify(s.props) })
};
```

### Event Logging

```js
h(MyComponent, {
  emit: {
    save: (state, data) => {
      console.log('Save event:', data);
      state.items.push(data);
    }
  }
})
```

## Summary

**Component Creation:**
- By name: `h('registered-name', ...)`
- By reference: `h(ComponentObject, ...)`

**Props:**
- Pass via `props: { propName: (ctx) => ctx.value }`
- Access via `ctx.$props.propName`

**Events:**
- Emit: `this.emit('eventName', payload)`
- Listen: `emit: { eventName: (state, payload) => {...} }`

**Two-Way Binding:**
- Use `model: { propName: 'stateKey' }`
- Child emits `update:propName`

**Slots:**
- Pass: `h('comp', {}, { default: [...], footer: [...] })`
- Requires template with `<slot/>` outlets

**Best Practices:**
- Define default props
- Single responsibility
- Type checking with component references
- Register common components globally

## Next Steps

- [Render Functions](./render-function.md) - Learn render basics
- [h() Function](./h-function.md) - Complete h() API
- [API Reference](../reference/api-reference.md) - Full documentation
