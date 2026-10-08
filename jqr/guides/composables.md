<!-- {
  "path": "jqr/guides/composables",
  "name": "docs-jqr-guides-composables",
  "meta": {
    "title": "Composables",
    "order": 7
  }
} -->

# Composables

Composables are reusable functions built on top of JQR's reactivity primitives (`reactive`, `computed`, `watch`, `effect`). Similar to Vue's Composition API, they allow you to extract and share stateful logic across components.

## What are Composables?

A composable is a **plain JavaScript function** that:
- Uses JQR's reactivity primitives
- Returns reactive state and methods
- Can be reused across multiple components
- Encapsulates specific logic/feature

**Key difference from component options:**
- Composables are **functions**, not component definitions
- Can be called from `render()` functions
- State is independent from any component instance
- Perfect for JQR-H render-based components

## Basic Example

### Simple Counter Composable

```js
import { reactive, computed } from '@en32/jqr';

function useCounter(initial = 0) {
  const state = reactive({ count: initial });

  const doubled = computed(() => state.count * 2);

  function increment() {
    state.count++;
  }

  function decrement() {
    state.count--;
  }

  function reset() {
    state.count = initial;
  }

  return {
    // Reactive state
    count: () => state.count,
    
    // Computed value (access via .value)
    doubled,
    
    // Methods
    increment,
    decrement,
    reset
  };
}
```

### Using in Component

```html live:no-code
<div id="demo-counter" class="en-card">
  <!-- Rendered by JQR-H -->
</div>
```

```html
<div id="demo-counter">
  <!-- Rendered by JQR-H -->
</div>
```

```js inject show
// Composable
function useCounter(initial = 0) {
  const { reactive, computed } = JQR;
  const state = reactive({ count: initial });
  const doubled = computed(() => state.count * 2);

  return {
    count: () => state.count,
    doubled,
    increment: () => state.count++,
    decrement: () => state.count--,
    reset: () => state.count = initial
  };
}

// Component using composable
JQRH.mount('#demo-counter', {
  render: (h, ctx) => {
    const counter = useCounter(10);

    return h('div', {}, [
      h('p', { text: () => `Count: ${counter.count()} (doubled: ${counter.doubled.value})` }),
      h('div', { class: 'en-flex en-gap-2' }, [
        h('button', {
          class: 'en-btn en-btn--sm',
          on: { click: counter.decrement },
          text: () => '-'
        }),
        h('button', {
          class: 'en-btn en-btn--sm',
          on: { click: counter.increment },
          text: () => '+'
        }),
        h('button', {
          class: 'en-btn en-btn--sm',
          on: { click: counter.reset },
          text: () => 'Reset'
        })
      ])
    ]);
  }
});
```

```js cleanup
onUnmount(() => {
  JQRH.unmount('#demo-counter');
})
```

## Reactivity Primitives

JQR exports standalone reactivity functions:

### `reactive(obj)`

Creates a reactive proxy of an object:

```js
import { reactive } from '@en32/jqr';

const state = reactive({ count: 0, name: 'Alice' });

// Mutations trigger reactivity
state.count++; // ✅ Reactive
state.name = 'Bob'; // ✅ Reactive
```

### `computed(getter)`

Creates a computed value (memoized getter):

```js
import { computed } from '@en32/jqr';

const doubled = computed(() => state.count * 2);

// Access via .value
console.log(doubled.value); // Read computed value
```

**Important:** `computed()` returns a `ComputedRef` - access via `.value`, not directly!

### `watch(source, callback, options)`

Watch a reactive value and run callback on change:

```js
import { watch } from '@en32/jqr';

const stop = watch(
  () => state.count,        // Source getter
  (newVal, oldVal) => {     // Callback
    console.log(`${oldVal} → ${newVal}`);
  },
  { deep: false }           // Options
);

// Stop watching
stop();
```

### `watchEffect(fn)`

Automatically tracks dependencies and re-runs:

```js
import { watchEffect } from '@en32/jqr';

const stop = watchEffect(() => {
  console.log('Count is:', state.count);
  // Automatically re-runs when state.count changes
});

stop(); // Stop watching
```

### `effect(fn)`

Low-level effect (used internally):

```js
import { effect } from '@en32/jqr';

const { stop } = effect(() => {
  element.textContent = state.message;
});

stop(); // Stop effect
```

## Composable Patterns

### Pattern 1: Feature Composition

Extract specific features into composables:

```js
// useToggle.js
function useToggle(initial = false) {
  const state = reactive({ value: initial });

  function toggle() {
    state.value = !state.value;
  }

  function setTrue() {
    state.value = true;
  }

  function setFalse() {
    state.value = false;
  }

  return {
    value: () => state.value,
    toggle,
    on: setTrue,
    off: setFalse
  };
}
```

**Usage:**

```js
render: (h) => {
  const isOpen = useToggle(false);
  
  return h('div', {}, [
    h('button', {
      on: { click: isOpen.toggle },
      text: () => isOpen.value() ? 'Close' : 'Open'
    }),
    h('div', {
      show: isOpen.value,
      text: () => 'Content'
    })
  ]);
}
```

### Pattern 2: Async Data Fetching

```html live:no-code
<div id="demo-fetch" class="en-card">
  <!-- Rendered by JQR-H -->
</div>
```

```html
<div id="demo-fetch">
  <!-- Rendered by JQR-H -->
</div>
```

```js inject show
// useFetch composable
function useFetch(url) {
  const { reactive } = JQR;
  const state = reactive({
    data: null,
    loading: true,
    error: null
  });

  async function fetchData() {
    state.loading = true;
    state.error = null;
    
    try {
      const response = await fetch(url);
      state.data = await response.json();
    } catch (err) {
      state.error = err.message;
    } finally {
      state.loading = false;
    }
  }

  fetchData();

  return {
    data: () => state.data,
    loading: () => state.loading,
    error: () => state.error,
    refetch: fetchData
  };
}

// Component
JQRH.mount('#demo-fetch', {
  render: (h) => {
    const users = useFetch('/assets/api/jqr-docs/users.json');

    return h('div', {}, [
      h('button', {
        class: 'en-btn en-btn--sm en-mb-2',
        on: { click: users.refetch() },
        text: () => 'Refetch'
      }),
      users.loading()
        ? h('p', { text: () => 'Loading...' })
        : users.error()
          ? h('p', { class: 'en-text--error', text: () => `Error: ${users.error()}` })
          : h('ul', {},
              (users.data() || []).slice(0, 3).map(user =>
                h('li', { text: user.name })
              )
            )
    ]);
  }
});
```

```js cleanup
onUnmount(() => {
  JQRH.unmount('#demo-fetch');
})
```

### Pattern 3: Form Handling

```js
function useForm(initialValues) {
  const state = reactive({
    values: { ...initialValues },
    errors: {},
    touched: {},
    isSubmitting: false
  });

  function setValue(field, value) {
    state.values[field] = value;
    state.touched[field] = true;
  }

  function setError(field, error) {
    state.errors[field] = error;
  }

  function validate() {
    // Validation logic
    const errors = {};
    if (!state.values.email) {
      errors.email = 'Email required';
    }
    state.errors = errors;
    return Object.keys(errors).length === 0;
  }

  async function submit(handler) {
    if (!validate()) return;
    
    state.isSubmitting = true;
    try {
      await handler(state.values);
    } finally {
      state.isSubmitting = false;
    }
  }

  return {
    values: () => state.values,
    errors: () => state.errors,
    isSubmitting: () => state.isSubmitting,
    setValue,
    setError,
    validate,
    submit
  };
}
```

### Pattern 4: Mouse Position Tracking

```js
function useMouse() {
  const state = reactive({ x: 0, y: 0 });

  function update(event) {
    state.x = event.clientX;
    state.y = event.clientY;
  }

  // Setup listener
  window.addEventListener('mousemove', update);

  // Return cleanup function
  function cleanup() {
    window.removeEventListener('mousemove', update);
  }

  return {
    x: () => state.x,
    y: () => state.y,
    cleanup
  };
}
```

## Important Considerations

### 1. ComputedRef Access

`computed()` returns a `ComputedRef` - **must access via `.value`**:

```js
const doubled = computed(() => state.count * 2);

// ❌ Wrong
h('p', { text: () => doubled })  // Returns ComputedRef object

// ✅ Correct
h('p', { text: () => doubled.value })  // Returns actual value
```

### 2. Call Inside render()

Call composables **inside `render()`** for per-instance state:

```js
// ✅ Correct - each instance gets own state
render: (h) => {
  const counter = useCounter(0);  // Called per instance
  return h('div', { text: () => counter.count() });
}

// ❌ Wrong - state shared across all instances!
const counter = useCounter(0);  // Called once at module level
const MyComponent = {
  render: (h) => h('div', { text: () => counter.count() })
};
```

### 3. Manual Cleanup

Composables don't auto-cleanup - you must handle it:

```js
render: (h, ctx) => {
  const mouse = useMouse();

  // Cleanup when component destroyed
  ctx.watch('', () => {}, { deep: false });  // Dummy to get cleanup
  
  return h('div', {
    text: () => `Mouse: ${mouse.x()}, ${mouse.y()}`
  });
},

destroyed() {
  // Call cleanup if composable provides it
  if (mouse.cleanup) mouse.cleanup();
}
```

**Better pattern:** Return cleanup function from composable, call in `destroyed()` hook.

### 4. Shared State (Global Store)

For **shared state across components**, call composable at module level:

```js
// store.js - Singleton store
const store = (() => {
  const state = reactive({ user: null, theme: 'dark' });

  function setUser(user) {
    state.user = user;
  }

  function setTheme(theme) {
    state.theme = theme;
  }

  return {
    state,  // Direct state access
    setUser,
    setTheme
  };
})();

// Component A
render: (h) => h('div', { text: () => store.state.theme })

// Component B
render: (h) => h('div', { text: () => store.state.theme })
// Both read the same shared state!
```

## Complete Example: Todo with Composables

```html live:no-code
<div id="demo-todo-composable" class="en-card">
  <!-- Rendered by JQR-H -->
</div>
```

```html
<div id="demo-todo-composable">
  <!-- Rendered by JQR-H -->
</div>
```

```js inject show
// useTodoList composable
function useTodoList() {
  const { reactive, computed } = JQR;
  const state = reactive({
    items: [],
    filter: 'all'
  });

  const filtered = computed(() => {
    if (state.filter === 'active') {
      return state.items.filter(t => !t.done);
    }
    if (state.filter === 'completed') {
      return state.items.filter(t => t.done);
    }
    return state.items;
  });

  function add(text) {
    state.items.push({
      id: Date.now(),
      text,
      done: false
    });
  }

  function toggle(id) {
    const item = state.items.find(t => t.id === id);
    if (item) item.done = !item.done;
  }

  function remove(id) {
    state.items = state.items.filter(t => t.id !== id);
  }

  return {
    items: () => state.items,
    filter: () => state.filter,
    setFilter: (f) => { state.filter = f; },
    filtered,
    add,
    toggle,
    remove
  };
}

// Component
JQRH.mount('#demo-todo-composable', {
  state: { newTodo: '' },
  render: (h, ctx) => {
    const todos = useTodoList();

    return h('div', {}, [
      // Add form
      h('div', { class: 'en-flex en-gap-2 en-mb-2' }, [
        h('input', {
          class: 'en-input',
          attr: { placeholder: 'Add todo...' },
          model: 'newTodo',
          on: {
            keyup: (e) => {
              if (e.key === 'Enter' && this.newTodo.trim()) {
                todos.add(this.newTodo);
                this.newTodo = '';
              }
            }
          }
        }),
        h('button', {
          class: 'en-btn en-btn--sm',
          on: { click: () => {
            if (ctx.newTodo.trim()) {
              todos.add(ctx.newTodo);
              ctx.newTodo = '';
            }
          }},
          text: () => 'Add'
        })
      ]),

      // Filters
      h('div', { class: 'en-flex en-gap-2 en-mb-2' }, [
        ['all', 'active', 'completed'].map(f =>
          h('button', {
            class: () => f === todos.filter() ? 'en-btn en-btn--sm en-btn--primary' : 'en-btn en-btn--sm',
            on: { click: () => todos.setFilter(f) },
            text: () => f.charAt(0).toUpperCase() + f.slice(1)
          })
        )
      ]),

      // List
      h.for(
        () => todos.filtered.value,
        (todo) => todo.id,
        (todo) => h('div', { class: 'en-flex en-items-center en-gap-2 en-mb-1' }, [
          h('input', {
            attr: { type: 'checkbox', checked: todo.done },
            on: { change: () => todos.toggle(todo.id) }
          }),
          h('span', {
            style: { textDecoration: todo.done ? 'line-through' : 'none' },
            text: todo.text
          }),
          h('button', {
            class: 'en-btn en-btn--sm en-btn--danger',
            on: { click: () => todos.remove(todo.id) },
            text: () => '×'
          })
        ])
      )
    ]);
  }
});
```

```js cleanup
onUnmount(() => {
  JQRH.unmount('#demo-todo-composable');
})
```

## Benefits of Composables

1. **Reusability** - Share logic across components
2. **Separation of Concerns** - Extract features into focused functions
3. **Testability** - Test composables independently
4. **Type Safety** - Easy to type with TypeScript
5. **Flexibility** - Mix and match multiple composables
6. **No Template Dependency** - Pure JavaScript logic

## Composables vs Component Options

| Feature | Composables | Component Options |
|---------|-------------|-------------------|
| Where used | `render()` function | Component definition |
| State | `reactive()` explicit | `state` option |
| Computed | `.value` access | Auto-unwrapped |
| Reusability | Easy to share | Per-component |
| Cleanup | Manual | Auto via `destroyed()` |
| Best for | JQR-H render-based | Template-based |

## TypeScript Support

Composables work great with TypeScript:

```typescript
import { reactive, computed, ComputedRef } from '@en32/jqr';

interface CounterState {
  count: number;
}

interface UseCounterReturn {
  count: () => number;
  doubled: ComputedRef<number>;
  increment: () => void;
  decrement: () => void;
}

function useCounter(initial: number = 0): UseCounterReturn {
  const state = reactive<CounterState>({ count: initial });
  const doubled = computed(() => state.count * 2);

  return {
    count: () => state.count,
    doubled,
    increment: () => state.count++,
    decrement: () => state.count--
  };
}
```

## Summary

**Composables are:**
- Plain functions using JQR reactivity primitives
- Perfect for JQR-H render-based components
- Reusable across multiple components
- Flexible and type-safe

**Key APIs:**
- `reactive(obj)` - Create reactive state
- `computed(fn)` - Memoized getter (access via `.value`)
- `watch(source, cb)` - Watch reactive values
- `watchEffect(fn)` - Auto-tracking watcher
- `effect(fn)` - Low-level effect

**Remember:**
- Call inside `render()` for per-instance state
- Access `computed()` via `.value`
- Handle cleanup manually in `destroyed()`
- Use module-level for shared/global state

Composables bring Vue Composition API patterns to JQR, making code more modular and reusable!

## Next Steps

- [Render Functions](./render-function.md) - Where to use composables
- [h() Function](./h-function.md) - Building UI with composables
- [Components](./components-with-h.md) - Component patterns
