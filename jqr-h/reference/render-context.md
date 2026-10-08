<!-- {
  "path": "jqr-h/reference/render-context",
  "name": "docs-jqr-h-reference-render-context",
  "meta": {
    "title": "Render Context",
    "order": 3
  }
} -->

# Render Context

The `ctx` (context) object passed to `render(h, ctx)` and every binding function in JQR-H.

## What is Render Context?

The render context is the **same** `HookCtx` object used throughout JQR:

- Passed as second argument to `render(h, ctx)`
- Passed to every binding function: `text: (ctx) => ...`, `on: { click: (ctx, ev) => ... }`
- Same object as `instance.$ctx` and `instance.$this`
- Same object `computed`/lifecycle hooks receive in template-based components

**Key principle:** `ctx` and `this` are interchangeable in JQR components.

```js
render: (h, ctx) => h('button', {
  text: (c) => c.message,  // c === ctx
  on: {
    click: (c) => c.increment()  // c === ctx
  }
})
```

## Type Definition

```typescript
type RenderContext<S extends Dict = Dict, P extends Dict = Dict> = HookCtx<S, P>
type Ctx<S extends Dict = Dict> = HookCtx<S, Dict>
```

- `S`: State type
- `P`: Props type (defaults to `Dict` in bindings)

## Core Properties

### $state

The reactive state object. Read and write directly.

```typescript
$state: S
```

**Example:**

```js
render: (h, ctx) => h('div', {}, [
  h('p', { text: (c) => c.$state.count }),
  h('button', {
    on: {
      click: (c) => {
        c.$state.count++;  // Direct mutation is reactive
      }
    }
  }, ['Increment'])
])
```

### $props

Readonly props object.

```typescript
$props: Readonly<P>
```

**Example:**

```js
render: (h, ctx) => h('div', {}, [
  h('h2', { text: (c) => c.$props.title }),
  h('p', { text: (c) => `User ID: ${c.$props.userId}` })
])
```

### $refs

Element references saved via `ref` attribute.

```typescript
$refs: Dict<JQuery>
```

**Example:**

```js
render: (h, ctx) => h('div', {}, [
  h('input', { ref: 'searchInput' }),
  h('button', {
    on: {
      click: (c) => {
        c.$refs.searchInput.focus();
        c.$refs.searchInput.val('');
      }
    }
  }, ['Clear'])
])
```

### $el

jQuery object wrapping the component's root element.

```typescript
$el: JQuery
```

**Example:**

```js
methods: {
  scrollToTop() {
    this.$el.animate({ scrollTop: 0 }, 300);
  }
}
```

## State Access

All `state`, `computed`, `constants`, and `methods` keys are accessible **directly** on `ctx`:

```js
{
  state: { count: 0, message: 'Hello' },
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
  render: (h, ctx) => h('div', {}, [
    // Direct access (preferred in h())
    h('p', { text: (c) => c.message }),
    h('p', { text: (c) => c.doubled }),
    h('button', {
      on: { click: (c) => c.increment() }
    }, ['Increment']),
    
    // Via $state (equivalent)
    h('p', { text: (c) => c.$state.message })
  ])
}
```

**Both forms work:**
- `ctx.count` (shorthand, preferred)
- `ctx.$state.count` (explicit)

## Core Methods

### $emit()

Emit custom events to parent.

```typescript
$emit(eventName: string, payload?: any): void
```

**Example:**

```js
render: (h, ctx) => h('form', {
  on: {
    submit: (c, ev) => {
      ev.preventDefault();
      c.$emit('save', { name: c.formName, email: c.formEmail });
    }
  }
}, [
  h('input', { model: 'formName' }),
  h('input', { model: 'formEmail' }),
  h('button', {}, ['Save'])
])
```

### $watch()

Watch for changes in state.

```typescript
$watch<T>(
  getter: (ctx: Ctx<S>) => T,
  callback: (newValue: T, oldValue: T, ctx: Ctx<S>) => void,
  options?: { immediate?: boolean; deep?: boolean }
): () => void
```

**Returns:** Cleanup function to stop watching

**Example:**

```js
created() {
  // Watch single value
  this.$watch(
    (ctx) => ctx.searchQuery,
    (newVal, oldVal) => {
      console.log(`Query changed from "${oldVal}" to "${newVal}"`);
      this.fetchResults();
    }
  );
  
  // Watch multiple values
  this.$watch(
    (ctx) => [ctx.page, ctx.pageSize],
    ([newPage, newSize], [oldPage, oldSize]) => {
      console.log('Pagination changed');
      this.loadPage();
    }
  );
  
  // Deep watch
  this.$watch(
    (ctx) => ctx.filters,
    () => {
      this.applyFilters();
    },
    { deep: true }
  );
}
```

### $destroy()

Destroy the component and cleanup all effects.

```typescript
$destroy(): void
```

**Example:**

```js
methods: {
  close() {
    this.$emit('close');
    this.$destroy();
  }
}
```

### $setProps()

Update component props (internal use, rarely needed directly).

```typescript
$setProps(newProps: Partial<P>): void
```

### $refetch()

Refetch async state values.

```typescript
$refetch(key?: string): Promise<void>
```

**Example:**

```js
{
  asyncState: {
    async userData() {
      const res = await fetch(`/api/user/${this.$props.userId}`);
      return res.json();
    }
  },
  render: (h, ctx) => h('div', {}, [
    h('button', {
      on: {
        click: (c) => c.$refetch('userData')
      }
    }, ['Refresh'])
  ])
}
```

### $addDisposer()

Register cleanup function (internal, used by mount()).

```typescript
$addDisposer(fn: () => void): void
```

## Async State Properties

Async state keys expose loading/error states:

```js
{
  asyncState: {
    async users() {
      const res = await fetch('/api/users');
      return res.json();
    }
  },
  render: (h, ctx) => h('div', {}, [
    // Access data
    h('div', {
      if: (c) => c.users.$resolved
    }, [
      h.for(
        (c) => c.users,
        (u) => u.id,
        (u) => h('div', {}, [u.name])
      )
    ]),
    
    // Loading state
    h('div', {
      if: (c) => c.users.$pending
    }, ['Loading...']),
    
    // Error state
    h('div', {
      if: (c) => c.users.$error
    }, [
      h('p', { text: (c) => `Error: ${c.users.$error.message}` })
    ])
  ])
}
```

**Properties:**
- `users`: The resolved data (or `undefined`)
- `users.$pending`: `true` while loading
- `users.$resolved`: `true` when data available
- `users.$error`: Error object if failed (or `null`)

## Context in Different Scopes

### In render()

```js
render: (h, ctx) => {
  // ctx is RenderContext<S, P>
  console.log(ctx.$state);
  console.log(ctx.$props);
  
  return h('div', {}, [
    h('p', { text: (c) => c.message })
  ]);
}
```

### In Bindings

```js
render: (h, ctx) => h('div', {
  // Each binding gets ctx (or named c, _ctx, etc.)
  text: (c) => c.message,
  class: (c) => ({ active: c.isActive }),
  on: {
    click: (c, ev) => {
      c.handleClick();
    }
  },
  props: {
    userId: (c) => c.currentUserId
  }
})
```

### In Methods

```js
methods: {
  // Use 'this' in methods (same as ctx)
  increment() {
    this.count++;
    this.$emit('change', this.count);
  },
  
  async fetchData() {
    const data = await fetch(`/api/data/${this.$props.id}`);
    this.data = await data.json();
  }
}
```

### In Lifecycle Hooks

```js
created() {
  // 'this' is the context
  console.log(this.$state);
  this.$watch((c) => c.query, () => this.search());
},

mounted() {
  this.$refs.input.focus();
},

destroyed() {
  console.log('Component destroyed');
}
```

## Context vs this

They're the **same object**:

```js
{
  state: { count: 0 },
  methods: {
    increment() {
      this.count++;      // Using 'this'
    }
  },
  render: (h, ctx) => {
    // ctx === this (if we had 'this' here)
    return h('div', {}, [
      h('button', {
        on: {
          click: (c) => {
            c.count++;     // Using ctx
            c.increment(); // Calling method
          }
        }
      }, ['Increment'])
    ]);
  }
}
```

**Naming convention:**
- `ctx` in `render(h, ctx)`
- `c` or `_ctx` in bindings (shorter)
- `this` in methods/hooks (familiar)

## Practical Examples

### Form Handling

```js
{
  state: {
    formData: {
      name: '',
      email: ''
    },
    errors: {}
  },
  methods: {
    validate() {
      const errors = {};
      if (!this.formData.name) errors.name = 'Name required';
      if (!this.formData.email) errors.email = 'Email required';
      this.errors = errors;
      return Object.keys(errors).length === 0;
    },
    submit() {
      if (this.validate()) {
        this.$emit('submit', this.formData);
      }
    }
  },
  render: (h, ctx) => h('form', {
    on: {
      submit: (c, ev) => {
        ev.preventDefault();
        c.submit();
      }
    }
  }, [
    h('input', {
      model: {
        get: (c) => c.formData.name,
        set: (c, v) => { c.formData.name = v; }
      }
    }),
    h('p', {
      if: (c) => c.errors.name,
      text: (c) => c.errors.name,
      class: 'error'
    }),
    h('input', {
      model: {
        get: (c) => c.formData.email,
        set: (c, v) => { c.formData.email = v; }
      }
    }),
    h('p', {
      if: (c) => c.errors.email,
      text: (c) => c.errors.email,
      class: 'error'
    }),
    h('button', {}, ['Submit'])
  ])
}
```

### Async Data with Refs

```js
{
  asyncState: {
    async products() {
      const res = await fetch('/api/products');
      return res.json();
    }
  },
  methods: {
    scrollToProduct(productId) {
      const el = this.$refs[`product-${productId}`];
      if (el) el[0].scrollIntoView({ behavior: 'smooth' });
    }
  },
  render: (h, ctx) => h('div', {}, [
    h('div', {
      if: (c) => c.products.$pending
    }, ['Loading products...']),
    
    h('div', {
      if: (c) => c.products.$resolved
    }, [
      h.for(
        (c) => c.products,
        (p) => p.id,
        (p) => h('div', {
          ref: `product-${p.id}`,
          class: 'product'
        }, [
          h('h3', {}, [p.name]),
          h('p', {}, [p.description])
        ])
      )
    ])
  ])
}
```

### Watchers and Effects

```js
{
  state: {
    searchQuery: '',
    results: [],
    isSearching: false
  },
  methods: {
    async search() {
      if (!this.searchQuery) {
        this.results = [];
        return;
      }
      
      this.isSearching = true;
      try {
        const res = await fetch(`/api/search?q=${this.searchQuery}`);
        this.results = await res.json();
      } finally {
        this.isSearching = false;
      }
    }
  },
  created() {
    // Debounced search
    let timeoutId;
    this.$watch(
      (c) => c.searchQuery,
      () => {
        clearTimeout(timeoutId);
        timeoutId = setTimeout(() => this.search(), 300);
      }
    );
  },
  render: (h, ctx) => h('div', {}, [
    h('input', {
      model: {
        state: 'searchQuery',
        debounce: 300
      },
      attrs: {
        placeholder: 'Search...'
      }
    }),
    h('div', {
      if: (c) => c.isSearching
    }, ['Searching...']),
    h('div', {}, [
      h.for(
        (c) => c.results,
        (r) => r.id,
        (r) => h('div', { class: 'result' }, [r.title])
      )
    ])
  ])
}
```

## TypeScript Support

```typescript
interface AppState {
  count: number;
  message: string;
}

interface AppProps {
  initialCount: number;
}

const component: RenderComponentOptions<AppState, AppProps> = {
  props: {
    initialCount: 0
  },
  state: {
    count: 0,
    message: 'Hello'
  },
  created() {
    // 'this' is typed as HookCtx<AppState, AppProps>
    this.count = this.$props.initialCount;
  },
  render: (h, ctx) => {
    // ctx is RenderContext<AppState, AppProps>
    return h('div', {}, [
      h('p', {
        text: (c) => {
          // c is Ctx<AppState>
          return `Count: ${c.count}`;
        }
      })
    ]);
  }
};
```

## Summary

**Context is:**
- ✅ Passed to `render(h, ctx)`
- ✅ Passed to every binding function
- ✅ Same as `instance.$ctx` and `instance.$this`
- ✅ Same as `this` in methods/hooks
- ✅ One unified object for all component logic

**Context provides:**
- State access (`$state` or direct)
- Props access (`$props`)
- Methods (`$emit`, `$watch`, `$destroy`, etc.)
- Refs (`$refs`)
- Element reference (`$el`)

## See Also

- [Render Functions](../guides/render-function.md) - Using render context
- [h() Function](../guides/h-function.md) - Context in bindings
- [API Reference](./api-reference.md) - Complete API
- [TypeScript Types](./types.md) - Type definitions
