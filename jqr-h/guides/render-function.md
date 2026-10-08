<!-- {
  "path": "jqr-h/guides/render-function",
  "name": "docs-jqr-jqrh-guides-render-function",
  "meta": {
    "title": "Render Functions",
    "order": 1
  }
} -->

# Render Functions

Render functions provide a programmatic alternative to HTML templatestate. Instead of writing markup with `$:` directives, you build your component UI with JavaScript using the `h()` function.

## What is a Render Function?

A render function is a JavaScript function that **returns a node description** using `h()`. It receives two arguments:

1. **`h`** - The hyperscript function for creating node descriptions
2. **`ctx`** - The component context (state, methods, props, etc.)

```js
render: (h, ctx) => {
  return h('div', { class: 'card' }, [
    h('h1', { text: ({state}) => state.title })
  ]);
}
```

**Important:** JQR-H **does NOT use Virtual DOM**. It uses **direct DOM binding + fine-grained reactivity** (similar to Alpine.js/SolidJS). The DOM is built once from your `h()` calls, then each reactive binding (text, class, attribute) updates independently via `effect()` — no re-rendering, no diffing, no tree reconciliation.

## Basic Usage

### Simple Counter

```html live:no-code
<div id="demo-basic-counter" class="en-card en-bg-subtle en-p-3">
  <!-- Rendered by JQR-H -->
</div>
```

```html
<div id="demo-basic-counter">
  <!-- Rendered by JQR-H -->
</div>
```

```js inject show
JQRH.mount('#demo-basic-counter', {
  state: { count: 0 },
  render: (h, ctx) => 
    h('div', {}, [
      h('p', { text: ({state}) => `Count: ${state.count}` }),
      h('button', {
        class: 'en-btn en-btn--sm',
        on: { click: () => ctx.state.count++ },
        text: () => 'Increment'
      })
    ])
});
```

```js cleanup
onUnmount(() => {
  JQRH.unmount('#demo-basic-counter');
})
```

The `render` function:
- Builds UI with `h()` calls
- Returns a single root node
- Has access to `ctx.state` for reactive data
- Can bind events directly inline

## Component Context (`ctx`)

The second argument contains everything you need:

### State and Props

```js
render: (h, ctx) => {
  // Access state
  const { state, props } = ctx;
  
  return h('div', {}, [
    h('p', { text: ({state}) => state.message }),      // Reactive state
    h('p', { text: ({state}) => state.propstate.userId })  // Props
  ]);
}
```

### Methods

Methods are pre-bound and directly callable:

```js
JQRH.mount('#app', {
  state: { items: [] },
  methods: {
    addItem(name) {
      this.state.itemstate.push({ id: Date.now(), name });
    }
  },
  render: (h, ctx) => 
    h('button', {
      on: { click: () => ctx.addItem('New Item') },
      text: () => 'Add'
    })
});
```

No need for `.call()` or `bind()` - methods are ready to use.

### Instance Methods

All JQR component methods are available:

```js
render: (h, ctx) => {
  // ctx.watch() - Watch state changes
  // ctx.emit() - Emit events
  // ctx.destroy() - Destroy component
  // ctx.$el - Root element (jQuery)
  // ctx.$refs - Element references
  
  return h('div', { ref: 'container' }, [
    h('button', {
      on: { click: () => ctx.emit('save', ctx.state.data) },
      text: () => 'Save'
    })
  ]);
}
```

## Template vs Render Function

**Template (Standard JQR):**
```html
<div id="user-card">
  <h3 $:text="user.name"></h3>
  <p $:text="user.email"></p>
  <button $:click="greet()">Greet</button>
</div>

<script>
$('#user-card').component({
  state: { 
    user: { name: 'Alice', email: 'alice@example.com' }
  },
  methods: {
    greet(state) {
      alert(`Hello, ${state.user.name}!`);
    }
  }
});
</script>
```

**Render Function (JQR-H):**
```js
JQRH.mount('#user-card', {
  state: { 
    user: { name: 'Alice', email: 'alice@example.com' }
  },
  methods: {
    greet(state) {
      alert(`Hello, ${state.user.name}!`);
    }
  },
  render: (h, ctx) => 
    h('div', {}, [
      h('h3', { text: ({state}) => state.user.name }),
      h('p', { text: ({state}) => state.user.email }),
      h('button', {
        on: { click: () => ctx.greet() },
        text: () => 'Greet'
      })
    ])
});
```

Both produce identical results!

## Dynamic Rendering

Render functions excel at dynamic UIs:

```html live:no-code
<div id="demo-dynamic" class="en-card en-p-3 en-bg-subtle">
  <!-- Rendered by JQR-H -->
</div>
```

```html
<div id="demo-dynamic">
  <!-- Rendered by JQR-H -->
</div>
```

```js inject show
JQRH.mount('#demo-dynamic', {
  state: {
    viewMode: 'list',
    items: ['Apple', 'Banana', 'Cherry']
  },
  render: (h, ctx) => {
    const { state } = ctx;

    const listView = h('ul', {
      attr: { class: 'en-list-group' },
      show: (c) => c.state.viewMode === 'list'
    }, state.items.map(item => h('li', { class: 'en-list-group__item' }, [item])));

    const gridView = h('div', {
      class: 'en-flex en-flex--row en-flex--between en-gap-2',
      show: (c) => c.state.viewMode === 'grid'
    }, state.items.map(item =>
      h('div', { class: 'en-card en-w-full en-p-2', text: () => item })
    ));

    return h('div', {}, [
      h('div', { class: 'en-flex en-gap-2 en-mb-3' }, [
        h('button', {
          class: 'en-btn en-btn--sm',
          on: { click: () => state.viewMode = 'list' },
          text: () => 'List View'
        }),
        h('button', {
          class: 'en-btn en-btn--sm',
          on: { click: () => state.viewMode = 'grid' },
          text: () => 'Grid View'
        })
      ]),
      listView,
      gridView
    ]);
  }
});
```

```js cleanup
onUnmount(() => {
  JQRH.unmount('#demo-dynamic');
})
```

With render functions, you can:
- Use any JavaScript logic (if/else, switch, loops)
- Extract helper functions
- Dynamically choose what to render
- No template syntax limitations

## Reactive Getters

Attributes can be static or reactive:

```js
h('div', {
  // Static values
  class: 'card',
  attr: { id: 'my-card' },
  
  // Reactive values (arrow functions)
  text: ({state}) => state.message,
  style: ({state}) => ({ color: state.theme.color }),
  class: ({state}) => ({ active: state.isActive }),
  
  // Events are always static
  on: {
    click: () => ctx.toggle()
  }
})
```

**Rule:** Use arrow functions `(ctx) => ...` for values that depend on state.

## Lifecycle Integration

Render functions work with all lifecycle hooks:

```js
JQRH.mount('#app', {
  state: { data: null },
  
  created() {
    console.log('Component created');
  },
  
  mounted() {
    console.log('Component mounted, DOM ready');
    // Can access thistate.$el, thistate.$refs
  },
  
  destroyed() {
    console.log('Component destroyed');
  },
  
  render: (h, ctx) => 
    h('div', { text: ({state}) => state.data ?? 'Loading...' })
});
```

## Computed Values

Use arrow functions inline:

```html live:no-code
<div id="demo-computed" class="en-card">
  <!-- Rendered by JQR-H -->
</div>
```

```html
<div id="demo-computed">
  <!-- Rendered by JQR-H -->
</div>
```

```js inject show
JQRH.mount('#demo-computed', {
  state: { 
    firstName: 'John',
    lastName: 'Doe'
  },
  render: (h, ctx) => 
    h('div', {}, [
      h('p', { 
        text: ({state}) => `Full Name: ${state.firstName} ${state.lastName}` 
      }),
      h('input', {
        class: 'en-input en-mb-2',
        attr: { placeholder: 'First Name' },
        model: 'firstName'
      }),
      h('input', {
        class: 'en-input',
        attr: { placeholder: 'Last Name' },
        model: 'lastName'
      })
    ])
});
```

```js cleanup
onUnmount(() => {
  JQRH.unmount('#demo-computed');
})
```

Or use `computed` option:

```js
JQRH.mount('#app', {
  state: { firstName: 'John', lastName: 'Doe' },
  computed: {
    fullName: ({state}) => `${state.firstName} ${state.lastName}`
  },
  render: (h, ctx) => 
    h('p', { text: ({state}) => `Name: ${state.computed.fullName}` })
});
```

## Conditional Rendering

### Simple Conditionals

```js
render: (h, ctx) => {
  const { state } = ctx;
  
  return h('div', {}, [
    state.isLoggedIn
      ? h('p', { text: 'Welcome back!' })
      : h('p', { text: 'Please log in' })
  ]);
}
```

### Multiple Conditions

```js
render: (h, ctx) => {
  const { state } = ctx;
  
  let message;
  if (state.status === 'loading') {
    message = h('p', { text: () => 'Loading...' });
  } else if (state.status === 'error') {
    message = h('p', { class: 'error', text: () => state.error });
  } else {
    message = h('p', { text: () => 'Data loaded successfully' });
  }
  
  return h('div', {}, [message]);
}
```

### Using `show` Attribute

```js
h('div', {}, [
  h('p', { 
    show: ({state}) => state.isVisible,
    text: () => 'This can be hidden'
  })
])
```

The `show` attribute toggles `display: none` (element stays in DOM).

## List Rendering

### Simple Array Map

```js
render: (h, ctx) => 
  h('ul', {}, 
    ctx.state.itemstate.map(item => 
      h('li', { text: () => item })
    )
  )
```

### With h.for() (Keyed)

For better performance with dynamic lists:

```js
render: (h, ctx) => 
  h('div', {}, [
    h.for(
      ({state}) => state.users,              // Items array
      (user) => user.id,           // Key function
      (user) => h('div', {}, [     // Builder function
        h('p', { text: () => user.name })
      ])
    )
  ])
```

See [h() Function Guide](./h-function.md) for more on `h.for()`.

## Event Handling

```js
h('button', {
  on: {
    click: (state, event) => {
      console.log('Clicked!', event);
      state.count++;
    },
    mouseenter: (state) => {
      state.isHovered = true;
    },
    mouseleave: (state) => {
      state.isHovered = false;
    }
  },
  text: () => 'Hover me'
})
```

All standard DOM events are supported.

## Form Binding

```html live:no-code
<div id="demo-form" class="en-card">
  <!-- Rendered by JQR-H -->
</div>
```

```html
<div id="demo-form">
  <!-- Rendered by JQR-H -->
</div>
```

```js inject show
JQRH.mount('#demo-form', {
  state: { 
    username: '',
    agree: false
  },
  render: (h, ctx) => 
    h('div', {}, [
      h('input', {
        class: 'en-input en-mb-2',
        attr: { 
          type: 'text',
          placeholder: 'Username' 
        },
        model: 'username'
      }),
      h('label', { class: 'en-flex en-items-center en-gap-2' }, [
        h('input', {
          attr: { type: 'checkbox' },
          model: 'agree'
        }),
        h('span', { text: () => 'I agree to terms' })
      ]),
      h('p', { 
        class: 'en-mt-3',
        text: ({state}) => `Username: ${state.username || '(empty)'}`
      }),
      h('p', { 
        text: ({state}) => `Agreed: ${state.agree ? 'Yes' : 'No'}`
      })
    ])
});
```

```js cleanup
onUnmount(() => {
  JQRH.unmount('#demo-form');
})
```

## Styling

### Static Classes

```js
h('div', { class: 'card primary' })
```

### Dynamic Classes

```js
h('div', { 
  class: ({state}) => ({
    active: state.isActive,
    disabled: state.isDisabled
  })
})
```

### Inline Styles

```js
h('div', {
  style: ({state}) => ({
    backgroundColor: state.theme.bg,
    color: state.theme.text,
    padding: '10px'
  })
})
```

## Extracting Helpers

Keep render functions clean:

```js
// Helper functions
function renderHeader(h, title) {
  return h('header', { class: 'en-card__header' }, [
    h('h3', { class: 'en-card__title', text: () => title })
  ]);
}

function renderItem(h, item) {
  return h('div', { class: 'item' }, [
    h('span', { text: () => item.name }),
    h('span', { text: () => item.value })
  ]);
}

// Component
JQRH.mount('#app', {
  state: { title: 'My App', items: [...] },
  render: (h, ctx) => 
    h('div', { class: 'en-card' }, [
      renderHeader(h, ctx.state.title),
      h('div', { class: 'en-card__body' },
        ctx.state.itemstate.map(item => renderItem(h, item))
      )
    ])
});
```

## Complete Example

```html live:no-code
<div id="demo-complete" class="en-card">
  <!-- Rendered by JQR-H -->
</div>
```

```html
<div id="demo-complete">
  <!-- Rendered by JQR-H -->
</div>
```

```js inject show
JQRH.mount('#demo-complete', {
  state: {
    todos: [],
    newTodo: '',
    filter: 'all'
  },
  computed: {
    filteredTodos: ({state}) => {
      if (state.filter === 'active') {
        return state.todostate.filter(t => !t.done);
      }
      if (state.filter === 'completed') {
        return state.todostate.filter(t => t.done);
      }
      return state.todos;
    }
  },
  methods: {
    addTodo(state) {
      if (state.newTodo.trim()) {
        state.todostate.push({
          id: Date.now(),
          text: state.newTodo,
          done: false
        });
        state.newTodo = '';
      }
    },
    toggleTodo(state, id) {
      const todo = state.todostate.find(t => t.id === id);
      if (todo) todo.done = !todo.done;
    }
  },
  render: (h, ctx) => {
    const { state, computed } = ctx;
    
    return h('div', {}, [
      // Input form
      h('div', { class: 'en-flex en-gap-2 en-mb-3' }, [
        h('input', {
          class: 'en-input',
          attr: { placeholder: 'Add todo...' },
          model: 'newTodo',
          on: {
            keyup: (s, e) => {
              if (e.key === 'Enter') ctx.addTodo();
            }
          }
        }),
        h('button', {
          class: 'en-btn en-btn--sm',
          on: { click: () => ctx.addTodo() },
        }, ['Add'])
      ]),
      
      // Filters
      h('div', { class: 'en-flex en-gap-2 en-mb-3' }, [
        h('button', {
          class: ({state}) => state.filter === 'all' ? 'en-btn en-btn--sm en-btn--primary' : 'en-btn en-btn--sm',
          on: { click: () => state.filter = 'all' },
          text: () => 'All'
        }),
        h('button', {
          class: ({state}) => state.filter === 'active' ? 'en-btn en-btn--sm en-btn--primary' : 'en-btn en-btn--sm',
          on: { click: () => state.filter = 'active' },
          text: () => 'Active'
        }),
        h('button', {
          class: ({state}) => state.filter === 'completed' ? 'en-btn en-btn--sm en-btn--primary' : 'en-btn en-btn--sm',
          on: { click: () => state.filter = 'completed' },
          text: () => 'Completed'
        })
      ]),
      
      // Todo list
      h.for(
        ({state}) => state.filteredTodos,
        (todo) => todo.id,
        (todo) => h('div', { class: 'en-flex en-items-center en-gap-2 en-mb-2' }, [
          h('input', {
            attr: { type: 'checkbox' },
            model: {
              get: () => todo.done,
              set: (v) => todo.done = v
            },
            on: { change: () => ctx.toggleTodo(todo.id) }
          }),
          h('span', {
            style: ({state}) => ({
              textDecoration: todo.done ? 'line-through' : 'none',
              opacity: todo.done ? '0.6' : '1'
            }),
            text: () => todo.text
          })
        ])
      )
    ]);
  }
});
```

```js cleanup
onUnmount(() => {
  JQRH.unmount('#demo-complete');
})
```

## When to Use Render Functions

**Use render functions when:**
- Building complex, highly dynamic UIs
- Need full JavaScript control flow
- Creating reusable component libraries
- Working with TypeScript for type safety
- Prefer programmatic over declarative

**Use templates when:**
- Simple, straightforward UIs
- Designer/team collaboration
- Rapid prototyping
- Prefer HTML-like syntax

## Key Takeaways

1. **Render functions return node descriptions** - Built with `h()`, rendered by `mount()`
2. **Direct DOM manipulation** - No Virtual DOM or diffing, updates are surgical and immediate
3. **Fine-grained reactivity** - Each binding updates independently via `effect()`
4. **Context provides everything** - `ctx.state`, `ctx.methods`, lifecycle, etc.
5. **Arrow functions for reactivity** - `({state}) => state.value` for dynamic attributes
6. **Full JavaScript power** - Use any JS logic, loops, conditionals
7. **Same lifecycle** - `created`, `mounted`, `destroyed` work identically
8. **Extract helpers** - Keep render functions clean and readable

## Next Steps

- [h() Function Guide](./h-function.md) - Deep dive into hyperscript API
- [Components with h()](./components-with-h.md) - Build nested components
- [API Reference](../reference/api-reference.md) - Complete API documentation
