<!-- {
  "path": "jqr-h/examples/basic-component",
  "name": "docs-jqr-h-examples-basic-component",
  "meta": {
    "title": "Basic Component",
    "order": 1
  }
} -->

# Example: Basic Counter Component

A simple counter component demonstrating the fundamentals of JQR-H: state, methods, computed properties, and reactive rendering with `h()`.

## Features

- ✅ State management with `state`
- ✅ Computed properties
- ✅ Methods and event handling
- ✅ Conditional rendering with `h.if`
- ✅ Dynamic classes and styles
- ✅ Reactive text binding

## Live Demo

```html live:no-code
<div id="demo-basic-counter"></div>
```

```html
<div id="demo-basic-counter"></div>
```

```js inject show
import { mount } from '@en32/jqr-h';

mount('#demo-basic-counter', {
  state: {
    count: 0,
    step: 1
  },
  computed: {
    isPositive() {
      return this.count > 0;
    },
    isNegative() {
      return this.count < 0;
    },
    countLabel() {
      if (this.count === 0) return 'Zero';
      if (this.count > 0) return `+${this.count}`;
      return `${this.count}`;
    }
  },
  methods: {
    increment() {
      this.count += this.step;
    },
    decrement() {
      this.count -= this.step;
    },
    reset() {
      this.count = 0;
    }
  },
  render: (h, ctx) => h('div', { class: 'en-card' }, [
    // Header
    h('div', { class: 'en-card__header' }, [
      h('h3', {}, ['Counter App'])
    ]),
    
    // Body
    h('div', { class: 'en-card__body' }, [
      // Count Display
      h('div', {
        class: (c) => ['en-text-center', 'en-mb-4', 'en-text-6xl', 'en-font-bold', {
          'en-text-success': c.isPositive,
          'en-text-danger': c.isNegative,
          'en-text-muted': c.count === 0
        }],
        text: (c) => c.countLabel
      }),
      
      // Controls
      h('div', { class: 'en-flex en-gap-2 en-mb-3' }, [
        h('button', {
          class: 'en-btn en-btn--danger en-flex-1',
          on: {
            click: (c) => c.decrement()
          }
        }, [`Decrement (-${ctx.step})`]),
        
        h('button', {
          class: 'en-btn en-btn--secondary',
          on: {
            click: (c) => c.reset()
          }
        }, ['Reset']),
        
        h('button', {
          class: 'en-btn en-btn--success en-flex-1',
          on: {
            click: (c) => c.increment()
          }
        }, [`Increment (+${ctx.step})`])
      ]),
      
      // Step Control
      h('div', { class: 'en-mb-3' }, [
        h('label', { class: 'en-form-label' }, ['Step Size:']),
        h('input', {
          class: 'en-input',
          attrs: {
            type: 'number',
            min: '1',
            max: '10'
          },
          model: 'step'
        })
      ]),
      
      // Info Messages
      h('div', {
        class: 'en-alert en-alert--success',
        if: (c) => c.isPositive
      }, [
        h('strong', {}, ['Positive! ']),
        'The count is above zero.'
      ]),
      
      h('div', {
        class: 'en-alert en-alert--danger',
        if: (c) => c.isNegative
      }, [
        h('strong', {}, ['Negative! ']),
        'The count is below zero.'
      ]),
      
      h('div', {
        class: 'en-alert en-alert--info',
        if: (c) => c.count === 0
      }, [
        h('strong', {}, ['Neutral! ']),
        'The count is zero.'
      ])
    ])
  ])
});
```

```js cleanup
onUnmount(() => {
  JQRH.unmount('#demo-basic-counter');
})
```

## Code Breakdown

### State Management

```js
state: {
  count: 0,    // Current count value
  step: 1      // Increment/decrement amount
}
```

State is reactive—changes automatically trigger re-renders of dependent bindings.

### Computed Properties

```js
computed: {
  isPositive() {
    return this.count > 0;
  },
  isNegative() {
    return this.count < 0;
  },
  countLabel() {
    if (this.count === 0) return 'Zero';
    if (this.count > 0) return `+${this.count}`;
    return `${this.count}`;
  }
}
```

Computed properties are cached and only recalculate when dependencies change.

### Methods

```js
methods: {
  increment() {
    this.count += this.step;
  },
  decrement() {
    this.count -= this.step;
  },
  reset() {
    this.count = 0;
  }
}
```

Methods are automatically bound to the component instance (`this` = context).

### Render Function Structure

The `render` function returns a tree of `h()` calls:

```js
render: (h, ctx) => h('div', { class: 'en-card' }, [
  h('div', { class: 'en-card__header' }, [
    h('h3', {}, ['Counter App'])
  ]),
  h('div', { class: 'en-card__body' }, [
    // ... body content
  ])
])
```

### Dynamic Classes

```js
class: (c) => ['en-text-center', 'en-mb-4', {
  'en-text-success': c.isPositive,
  'en-text-danger': c.isNegative,
  'en-text-muted': c.count === 0
}]
```

Classes can be:
- Static strings
- Functions returning strings, objects, or arrays
- Mixed (array of strings + conditional objects)

### Reactive Text

```js
text: (c) => c.countLabel
```

Text bindings are functions that receive context and return the display value.

### Event Handlers

```js
on: {
  click: (c) => c.increment()
}
```

Event handlers receive context and the event object.

### Conditional Rendering

```js
h('div', {
  class: 'en-alert en-alert--success',
  if: (c) => c.isPositive
}, ['Positive message'])
```

The `if` attribute controls whether the element is mounted/unmounted.

### Model Binding

```js
h('input', {
  class: 'en-input',
  attrs: { type: 'number' },
  model: 'step'
})
```

Two-way binding to state property—changes to input update state, state changes update input.

## Key Concepts Demonstrated

### 1. Fine-Grained Reactivity

Each binding (`text`, `class`, `if`) is wrapped in its own effect—only affected elements update when state changes:

```js
// Only the text updates when count changes
h('div', { text: (c) => c.count })

// Only classes update when isPositive changes
h('div', { class: (c) => ({ active: c.isPositive }) })
```

### 2. Context Access

Two ways to access component data:

```js
// Via context parameter (in bindings)
text: (c) => c.count

// Direct access (in render body)
['Step: ', ctx.step]
```

### 3. No Virtual DOM Diffing

JQR-H doesn't diff virtual DOM trees. Each reactive binding is a direct effect on the real DOM—when `count` changes, only the `text` binding re-runs and updates `textContent`.

### 4. Computed vs. Methods

```js
computed: {
  // Cached, only recalculates when dependencies change
  doubleCount() {
    return this.count * 2;
  }
},
methods: {
  // Runs every time it's called
  logCount() {
    console.log(this.count);
  }
}
```

## Variations

### Minimal Counter

Simplest possible counter:

```js
mount('#app', {
  state: { count: 0 },
  render: (h, ctx) => h('div', {}, [
    h('p', { text: (c) => c.count }),
    h('button', {
      on: { click: (c) => c.count++ }
    }, ['Increment'])
  ])
});
```

### With Animation

Add transitions when count changes:

```js
render: (h, ctx) => h('div', {}, [
  h('div', {
    text: (c) => c.count,
    style: (c) => ({
      fontSize: `${Math.min(100 + Math.abs(c.count) * 5, 200)}px`,
      transition: 'font-size 0.3s ease'
    })
  })
])
```

### With LocalStorage

Persist count across page reloads:

```js
{
  state: {
    count: parseInt(localStorage.getItem('count') || '0')
  },
  watch: {
    count(newVal) {
      localStorage.setItem('count', newVal);
    }
  },
  render: (h, ctx) => // ... as before
}
```

## Browser/IIFE Usage

```html
<script src="https://cdn.jsdelivr.net/npm/@en32/jqr@latest/dist/jqr.min.js"></script>
<script src="https://cdn.jsdelivr.net/npm/@en32/jqr-h@latest/dist/jqr-h.min.js"></script>

<div id="counter"></div>

<script>
  const { h, mount } = JQRH;
  
  mount('#counter', {
    state: { count: 0 },
    render: (h, ctx) => h('div', {}, [
      h('p', { text: (c) => c.count }),
      h('button', {
        on: { click: (c) => c.count++ }
      }, ['Click me'])
    ])
  });
</script>
```

## See Also

- [Render Functions](../guides/render-function.md) - Render function basics
- [h() Function](../guides/h-function.md) - Complete h() API
- [Components with h()](../guides/components-with-h.md) - Component composition
- [Dynamic Component](./dynamic-component.md) - Component switching
- [List Rendering](./list-rendering.md) - List with h.for
