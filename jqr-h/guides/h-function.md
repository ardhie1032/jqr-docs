<!-- {
  "path": "jqr-h/guides/h-function",
  "name": "docs-jqr-jqrh-guides-h-function",
  "meta": {
    "title": "h() Function",
    "order": 2
  }
} -->

# h() Function

The `h()` function (hyperscript) is the core API of JQR-H. It creates lightweight **node descriptions** that get rendered into real DOM by `mount()`.

## Basic Syntax

```js
h(tag, attributes, children)
```

**Parameters:**
- **`tag`** - HTML tag name (`'div'`, `'span'`) or component name/reference
- **`attributes`** - Object of attributes, bindings, and event handlers (optional)
- **`children`** - Array of child nodes or slot object (optional)

**Returns:** A `JqrNode` description object (not actual DOM).

## Simple Elements

### Basic Element

```html live:no-code
<div id="demo-basic" class="en-card">
  <!-- Rendered by JQR-H -->
</div>
```

```html
<div id="demo-basic">
  <!-- Rendered by JQR-H -->
</div>
```

```js inject show
JQRH.mount('#demo-basic', {
  state: {},
  render: (h) => h('div', { class: 'en-card__body' }, [
    h('h3', {}, ['Hello World'])
  ])
});
```

```js cleanup
onUnmount(() => {
  JQRH.unmount('#demo-basic');
})
```

### With Text Content

There are **two ways** to add text content:

**1. Reactive (via `text` attribute):**
```js
// Always reactive - re-evaluates when state changes
h('p', { text: ({state}) => state.message })
h('div', { text: ({state}) => `Count: ${state.count}` })
```

**2. Static (via children array):**
```js
// Static string - never changes
h('p', {}, ['Hello World'])

// Static variable - evaluated once at render time
const theme = 'dark';
h('div', {}, [theme])
```

**Key Difference:**
- `text` attribute → **reactive** (arrow function with state)
- Children array → **static** (plain strings/values)

```js
// ❌ Wrong - text attribute needs arrow function for state
h('p', { text: state.message })  // Won't be reactive!

// ✅ Correct - arrow function for reactivity
h('p', { text: ({state}) => state.message })

// ✅ Also correct - static string via children
h('p', {}, [state.message])  // Evaluated once, then static
```

### Empty Element

```js
// No attributes, no children
h('div')

// Explicitly null/empty
h('div', null, [])
```

## Attributes

### Static Attributes

```js
h('div', {
  class: 'card primary',
  attr: {
    id: 'my-card',
    'data-type': 'info',
    title: 'Hover message'
  }
})
```

### Reactive Attributes

```js
h('div', {
  attr: {
    id: ({state}) => `card-${state.id}`,
    disabled: ({state}) => state.isDisabled,
    href: ({state}) => state.url
  }
})
```

**Boolean attributes:**
- `true` → attribute with empty value
- `false`/`null`/`undefined` → attribute removed

```js
h('button', {
  attr: {
    disabled: ({state}) => state.busy  // disabled="" when true, removed when false
  }
})
```

## Classes

### Static Classes

```js
// String
h('div', { class: 'card primary' })

// Array
h('div', { class: ['card', 'primary'] })

// Object
h('div', { class: { card: true, primary: true } })
```

### Dynamic Classes

```html live:no-code
<div id="demo-classes" class="en-card en-p-4">
  <!-- Rendered by JQR-H -->
</div>
```

```html
<div id="demo-classes">
  <!-- Rendered by JQR-H -->
</div>
```

```js inject show
JQRH.mount('#demo-classes', {
  state: { isActive: false },
  render: (h, ctx) => 
    h('div', {}, [
      h('div', {
        class: ({state}) => ({
          'en-alert': true,
          'en-alert--success': state.isActive,
          'en-alert--warning': !state.isActive
        }),
        text: ({state}) => state.isActive ? 'Active' : 'Inactive'
      }),
      h('button', {
        class: 'en-btn en-btn--sm en-mt-2',
        on: { click: () => ctx.state.isActive = !ctx.state.isActive },
        text: () => 'Toggle'
      })
    ])
});
```

```js cleanup
onUnmount(() => {
  JQRH.unmount('#demo-classes');
})
```

### Mixed Static/Dynamic

```js
h('div', {
  class: ({state}) => [
    'card',                    // Always present
    { active: state.isActive }     // Conditional
  ]
})
```

## Styles

### Static Styles

```js
// Object (camelCase or kebab-case)
h('div', {
  style: {
    backgroundColor: 'blue',
    padding: '10px',
    'font-size': '14px'
  }
})

// CSS string
h('div', {
  style: 'background-color: blue; padding: 10px;'
})
```

### Dynamic Styles

```html live:no-code
<div id="demo-styles" class="en-card en-p-3">
  <!-- Rendered by JQR-H -->
</div>
```

```html
<div id="demo-styles">
  <!-- Rendered by JQR-H -->
</div>
```

```js inject show
JQRH.mount('#demo-styles', {
  state: { color: '#3b82f6', size: 16 },
  render: (h, ctx) => 
    h('div', {}, [
      h('p', {
        style: ({state}) => ({
          color: state.color,
          fontSize: `${state.size > 30 ? (state.size <= 10 ? 10 : state.size) : state.size}px`,
          fontWeight: 'bold'
        }),
        text: () => 'Styled text'
      }),
      h('div', { class: 'en-flex en-gap-2 en-mt-2' }, [
        h('input', {
          class: 'en-input',
          attr: { type: 'color' },
          model: 'color'
        }),
        h('input', {
          class: 'en-input',
          attr: { type: 'number', min: 10, max: 30 },
          model: 'size'
        })
      ])
    ])
});
```

```js cleanup
onUnmount(() => {
  JQRH.unmount('#demo-styles');
})
```

## Text and HTML

### Text Content

**Reactive text (via `text` attribute):**
```js
// Always use arrow function for reactivity
h('p', { text: ({state}) => state.message })

// Computed/expressions
h('p', { text: ({state}) => `Count: ${state.count}` })

// Auto-stringified (objects, arrays, etc.)
h('p', { text: ({state}) => state.data })  // JSON.stringify not needed
```

**Static text (via children):**
```js
// Plain string
h('p', {}, ['Hello World'])

// Variable evaluated once at render time
const userName = 'Alice';
h('p', {}, [userName])

// Mixed static content
h('p', {}, ['User: ', userName, ' (Admin)'])
```

**When to use which:**
- Use `text` attribute when the value **depends on state** (needs reactivity)
- Use children array when the value is **constant** or **computed once** at render time

### HTML Content

```js
// Use with caution - XSS risk!
h('div', {
  html: ({state}) => state.trustedHtml
})
```

⚠️ **Warning:** Only use `html` with trusted content. User input should use `text` instead.

## Events

### Basic Events

```js
h('button', {
  on: {
    click: (state, event) => {
      console.log('Clicked!', event);
      state.count++;
    }
  }
})
```

### Multiple Events

```html live:no-code
<div id="demo-events" class="en-card">
  <!-- Rendered by JQR-H -->
</div>
```

```html
<div id="demo-events">
  <!-- Rendered by JQR-H -->
</div>
```

```js inject show
JQRH.mount('#demo-events', {
  state: { 
    clicks: 0,
    isHovered: false
  },
  render: (h, ctx) => 
    h('button', {
      class: ({state}) => state.isHovered ? 'en-btn en-btn--primary' : 'en-btn',
      on: {
        click: ({state}) => {
          state.clicks++;
        },
        mouseenter: ({state}) => {
          state.isHovered = true;
        },
        mouseleave: ({state}) => {
          state.isHovered = false;
        }
      },
      text: ({state}) => `Clicks: ${state.clicks}`
    })
});
```

```js cleanup
onUnmount(() => {
  JQRH.unmount('#demo-events');
})
```

### Event Object

```js
h('input', {
  on: {
    keyup: (state, event) => {
      if (event.key === 'Enter') {
        // Handle enter key
      }
    },
    input: (state, event) => {
      state.value = event.target.value;
    }
  }
})
```

## Children

### Array of Children

```js
h('div', {}, [
  h('h1', { text: () => 'Title' }),
  h('p', { text: () => 'Paragraph' }),
  h('button', { text: () => 'Click' })
])
```

### Text Children

**Static text (evaluated once):**
```js
// Plain string
h('p', {}, ['Hello World'])

// Variable captured at render time
const greeting = 'Welcome!';
h('p', {}, [greeting])
```

**Reactive text (arrow function):**
```js
// Re-evaluates when state changes
h('p', {}, [
  ({state}) => `Count: ${state.count}`
])
```

**Mixed static + reactive:**
```js
h('p', {}, [
  'Total: ',              // Static
  ({state}) => state.total          // Reactive
])
```

**Comparison:**
```js
const userName = 'Alice';

// Static via children - captured once
h('div', {}, [userName])

// Reactive via text attribute - updates when state changes
h('div', { text: ({state}) => state.userName })
```

### Nested Elements

```js
h('div', { class: 'card' }, [
  h('header', {}, [
    h('h1', null, ['Title'])
  ]),
  h('main', {}, [
    h('p', { text: () => 'Content' })
  ]),
  h('footer', {}, [
    h('button', {}, ['Action'])
  ])
])
```

### Conditional Children

```js
h('div', {}, [
  h('h1', null, ['Title']),
  // Conditionally include children
  ...(state.showDetails ? [
    h('p', { text: () => 'Details here' })
  ] : [])
])
```

### Null Children (Ignored)

```js
h('div', {}, [
  h('p', { text: () => 'Always shown' }),
  null,                              // Ignored
  undefined,                         // Ignored
  state.show && h('p', { text: () => 'Conditional' })  // null if false
])
```

## Form Controls

### Text Input

```js
h('input', {
  class: 'en-input',
  attr: { 
    type: 'text',
    placeholder: 'Enter name'
  },
  model: 'username'  // Two-way binding
})
```

### Checkbox

```html live:no-code
<div id="demo-checkbox" class="en-card en-card__body">
  <!-- Rendered by JQR-H -->
</div>
```

```html
<div id="demo-checkbox">
  <!-- Rendered by JQR-H -->
</div>
```

```js inject show
JQRH.mount('#demo-checkbox', {
  state: { 
    agreed: false,
    subscribed: false
  },
  render: (h, ctx) => 
    h('div', { attr: {class: 'en-flex en-gap-3'} }, [
      h('label', { class: 'en-check' }, [
        h('input', {
          attr: { type: 'checkbox' },
          model: 'agreed'
        }),
        h('span', null, ['I agree to terms'])
      ]),
      h('label', { class: 'en-check' }, [
        h('input', {
          attr: { type: 'checkbox' },
          model: 'subscribed'
        }),
        h('span', null, ['Subscribe to newsletter'])
      ]),
      h('p', { 
        class: 'en-text--muted',
        text: ({state}) => `Agreed: ${state.agreed}, Subscribed: ${state.subscribed}`
      })
    ])
});
```

```js cleanup
onUnmount(() => {
  JQRH.unmount('#demo-checkbox');
})
```

### Select Dropdown

```js
h('select', {
  class: 'en-select',
  model: 'country'
}, [
  h('option', { attr: { value: 'us' }, text: () => 'United States' }),
  h('option', { attr: { value: 'uk' }, text: () => 'United Kingdom' }),
  h('option', { attr: { value: 'jp' }, text: () => 'Japan' })
])
```

### Textarea

```js
h('textarea', {
  class: 'en-textarea',
  attr: {
    placeholder: 'Enter message...',
    rows: 5
  },
  model: 'message'
})
```

## Model WITH Modifiers

JQR-H now supports **model modifiers** via object syntax!

### Three Model Forms

**1. String (simple):**
```js
h('input', { model: 'username' })
```

**2. FormModelBinding (with modifiers):**
```js
h('input', {
  model: { state: 'query', debounce: 400 }
})
```

**3. ComputedModelBinding (get/set):**
```js
h('input', {
  model: {
    get: (ctx) => ctx.state.fullName,
    set: (ctx, value) => { /* custom logic */ },
    debounce: 400
  }
})
```

### FormModelBinding - Timing Modifiers

For form controls (input, textarea):

```js
// Debounce - wait N ms after last keystroke
h('input', {
  model: { state: 'query', debounce: 500 }
})

// Throttle - max once per N ms
h('input', {
  model: { state: 'scroll', throttle: 300 }
})

// Lazy - update on blur, not input
h('input', {
  model: { state: 'email', lazy: true }
})
```

**Modifiers:**

| Modifier | Type | Description |
|----------|------|-------------|
| `debounce` | `number \| true` | Wait N ms after last keystroke (250ms if `true`) |
| `throttle` | `number` | Update max once per N ms window |
| `lazy` | `boolean` | Use `change` event instead of `input` |

**Rules:**
- `debounce` + `throttle` → `debounce` wins (warning)
- `lazy` + `debounce`/`throttle` → timing wins, `lazy` ignored (warning)
- Auto-canceled when element disposed

### ComputedModelBinding - Custom Logic

For complex get/set logic:

```js
h('input', {
  model: {
    // Getter - how to read value
    get: (ctx) => `${ctx.state.firstName} ${ctx.state.lastName}`,
    
    // Setter - how to write value
    set: (ctx, value) => {
      const [first, last] = value.split(' ');
      ctx.state.firstName = first || '';
      ctx.state.lastName = last || '';
    },
    
    // Optional: timing modifiers work here too!
    debounce: 400
  }
})
```

**Use cases:**
- Transform before storing (uppercase, trim, parse)
- Update multiple state fields
- Derived/formatted values

### Complete Examples

**Debounced Search:**

```html live:no-code
<div id="demo-debounce" class="en-card">
  <!-- Rendered by JQR-H -->
</div>
```

```html
<div id="demo-debounce">
  <!-- Rendered by JQR-H -->
</div>
```

```js inject show
JQRH.mount('#demo-debounce', {
  state: { 
    query: '',
    updateCount: 0
  },
  render: (h, ctx) => 
    h('div', {}, [
      h('p', { 
        class: 'en-mb-2',
        text: ({state}) => `Query: "${state.query}"` 
      }),
      h('input', {
        class: 'en-input en-mb-2',
        attr: { placeholder: 'Type to search (debounced 500ms)...' },
        model: { state: 'query', debounce: 500 }
      }),
      h('p', { 
        class: 'en-text--muted',
        text: ({state}) => `State updates: ${state.updateCount}`
      })
    ]),
  created(ctx) {
    ctx.watch('query', () => ctx.state.updateCount++);
  }
});
```

```js cleanup
onUnmount(() => {
  JQRH.unmount('#demo-debounce');
})
```

**Computed Model:**

```html live:no-code
<div id="demo-computed-model" class="en-card">
  <!-- Rendered by JQR-H -->
</div>
```

```html
<div id="demo-computed-model">
  <!-- Rendered by JQR-H -->
</div>
```

```js inject show
JQRH.mount('#demo-computed-model', {
  state: { 
    firstName: 'John',
    lastName: 'Doe'
  },
  render: (h, ctx) => 
    h('div', {}, [
      h('p', { 
        class: 'en-mb-2',
        text: ({state}) => `Name: ${state.firstName} ${state.lastName}` 
      }),
      h('input', {
        class: 'en-input',
        attr: { placeholder: 'Enter full name (first last)' },
        model: {
          get: (ctx) => `${ctx.state.firstName} ${ctx.state.lastName}`,
          set: (ctx, value) => {
            const [first, ...rest] = value.split(' ');
            ctx.state.firstName = first || '';
            ctx.state.lastName = rest.join(' ') || '';
          }
        }
      })
    ])
});
```

```js cleanup
onUnmount(() => {
  JQRH.unmount('#demo-computed-model');
})
```

### Migration from Manual

**Before (manual debounce):**
```js
const debounce = (fn, ms) => {
  let timeout;
  return (...args) => {
    clearTimeout(timeout);
    timeout = setTimeout(() => fn(...args), ms);
  };
};

h('input', {
  attr: { value: ({state}) => state.query },
  on: { input: debounce((s, e) => { s.query = e.target.value; }, 500) }
})
```

**After (built-in):**
```js
h('input', {
  model: { state: 'query', debounce: 500 }
})
```

### Transform Modifiers (Still Manual)

Transforms like `.number`, `.trim` require manual implementation:

```js
// .number - parse as number
h('input', {
  attr: { type: 'number', value: ({state}) => state.count },
  on: { input: (s, e) => { s.count = Number(e.target.value); } }
})

// .trim - trim whitespace
h('input', {
  attr: { value: ({state}) => state.name },
  on: { input: (s, e) => { s.name = e.target.value.trim(); } }
})
```

Or use computed model:

```js
h('input', {
  model: {
    get: ({state}) => state.count,
    set: (ctx, v) => { ctx.state.count = Number(v); }  // .number
  }
})
```

### Best Practice

| Use Case | Recommendation |
|----------|----------------|
| Simple string input | `model: 'field'` |
| Debounce/throttle/lazy | `model: { state: 'field', debounce: 500 }` |
| Complex logic | `model: { get, set }` |
| Transform value | Computed model or manual `attr` + `on` |

## Visibility Control

### show Attribute

```html live:no-code
<div id="demo-show" class="en-card en-card__body">
  <!-- Rendered by JQR-H -->
</div>
```

```html
<div id="demo-show">
  <!-- Rendered by JQR-H -->
</div>
```

```js inject show
JQRH.mount('#demo-show', {
  state: { isVisible: true },
  render: (h, ctx) => 
    h('div', {}, [
      h('p', {
        show: ({state}) => state.isVisible,
        text: () => 'Now you see me'
      }),
      h('button', {
        class: 'en-btn en-btn--sm en-mt-2',
        on: { click: () => ctx.state.isVisible = !ctx.state.isVisible },
        text: ({state}) => state.isVisible ? 'Hide' : 'Show'
      })
    ])
});
```

```js cleanup
onUnmount(() => {
  JQRH.unmount('#demo-show');
})
```

The `show` attribute toggles `display: none` - element stays in DOM.

### if Attribute (Alias)

```js
// Same as show, just different name
h('p', {
  if: ({state}) => state.condition,
  text: () => 'Conditionally visible'
})
```

## Teleport

Move elements to different locations in the DOM tree (portals) using the `teleport` attribute.

### Basic Teleport

```js
// Static target
h('div', {
  teleport: '#modal-root',
  class: 'modal'
}, [
  h('h3', { text: () => 'Modal Title' }),
  h('p', { text: () => 'Modal content' })
])

// Reactive target
h('div', {
  teleport: ({state}) => state.currentPortal,
  text: () => 'Dynamic portal'
})
```

### Teleport with Options

Use object form for advanced options:

```js
// With defer (waits for target to appear)
h('div', {
  teleport: {
    to: '#late-target',
    defer: true  // Uses MutationObserver to wait
  },
  text: () => 'Waits for target'
})

// With mode
h('div', {
  teleport: {
    to: 'body',
    mode: 'prepend'  // 'append' (default), 'prepend', 'insertAfter', 'insertBefore'
  },
  text: () => 'Prepended to body'
})

// Combine defer + mode
h('div', {
  teleport: {
    to: '#dynamic-portal',
    defer: true,
    mode: 'insertAfter'
  },
  text: () => 'Deferred + positioned'
})
```

### Teleport Forms

**String form** - Simple target selector:
```js
teleport: '#portal'
teleport: 'body'
teleport: '.modal-root'
```

**Object form** - With options:
```js
teleport: {
  to: '#portal',           // Target selector (required)
  mode?: 'append',         // Insertion mode (optional)
  defer?: true             // Wait for target (optional)
}
```

**Reactive** - Arrow function:
```js
teleport: ({state}) => state.targetPortal
teleport: ({state}) => ({ to: state.target, defer: true })
```

### Insertion Modes

| Mode | Behavior |
|------|----------|
| `'append'` | Add to end of target's children (default) |
| `'prepend'` | Add to start of target's children |
| `'insertAfter'` | Insert after target element (as sibling) |
| `'insertBefore'` | Insert before target element (as sibling) |

```js
// Append (default)
h('div', { teleport: 'body' })
h('div', { teleport: { to: 'body', mode: 'append' } })

// Prepend - first child
h('div', { teleport: { to: '#container', mode: 'prepend' } })

// Insert after target
h('div', { teleport: { to: '#header', mode: 'insertAfter' } })

// Insert before target
h('div', { teleport: { to: '#footer', mode: 'insertBefore' } })
```

### Defer Option

When `defer: true`, teleport waits for target to appear using `MutationObserver`:

```js
// Without defer - target must exist now
h('div', {
  teleport: '#portal',  // Error if #portal doesn't exist
  text: () => 'Content'
})

// With defer - waits for target
h('div', {
  teleport: { to: '#portal', defer: true },
  text: () => 'Waits for #portal to appear'
})
```

**Use cases for defer:**
- Target rendered conditionally (by `$:if`)
- Target loaded via async data
- Target inside another component that mounts later
- Target created by third-party library

### Teleport Example

```html live:no-code
<div id="demo-teleport-h" class="en-card en-p-3">
  <!-- Rendered by JQR-H -->
</div>
```

```css inject
.toast-notification-h {
  position: fixed;
  top: 20px;
  right: 20px;
  background: #10b981;
  color: white;
  padding: 1rem 1.5rem;
  border-radius: 8px;
  box-shadow: 0 4px 12px rgba(0, 0, 0, 0.15);
  z-index: 9999;
  animation: slideInRight 0.3s ease;
}

@keyframes slideInRight {
  from {
    transform: translateX(100%);
    opacity: 0;
  }
  to {
    transform: translateX(0);
    opacity: 1;
  }
}
```

```js inject show
JQRH.mount('#demo-teleport-h', {
  state: { showToast: false },
  
  methods: {
    show() {
      this.showToast = true;
      setTimeout(() => {
        this.showToast = false;
      }, 3000);
    }
  },
  
  render: (h, ctx) => 
    h('div', {}, [
      h('button', {
        class: 'en-btn en-btn--primary',
        on: { click: () => ctx.show() },
        text: () => 'Show Toast'
      }),
      
      // Toast teleports to body
      h('div', {
        if: () => ctx.showToast,
        teleport: 'body',
        class: 'toast-notification-h',
        text: () => '✓ Success message!'
      })
    ])
});
```

```js cleanup
onUnmount(() => {
  JQRH.unmount('#demo-teleport-h');
})
```

### Modal Pattern with Teleport

```js
JQRH.mount('#app', {
  state: { showModal: false },
  
  render: (h, ctx) => 
    h('div', {}, [
      h('button', {
        on: { click: () => ctx.showModal = true },
        text: () => 'Open Modal'
      }),
      
      // Modal teleports to body
      h('div', {
        if: () => ctx.showModal,
        teleport: 'body',
        class: 'modal-overlay',
        on: { click: () => ctx.showModal = false }
      }, [
        h('div', {
          class: 'modal-dialog',
          on: { click: (s, e) => e.stopPropagation() }
        }, [
          h('h3', { text: () => 'Modal Title' }),
          h('p', { text: () => 'Modal content here' }),
          h('button', {
            on: { click: () => ctx.showModal = false },
            text: () => 'Close'
          })
        ])
      ])
    ])
});
```

### Deferred Teleport Example

```js
// Target appears conditionally
JQRH.mount('#app', {
  state: { 
    showTarget: false,
    showContent: true
  },
  
  render: (h, ctx) => 
    h('div', {}, [
      // Toggle target visibility
      h('button', {
        on: { click: () => ctx.showTarget = !ctx.showTarget },
        text: ({state}) => state.showTarget ? 'Hide Target' : 'Show Target'
      }),
      
      // Conditional target
      h('div', {
        if: () => ctx.showTarget,
        attr: { id: 'dynamic-target' },
        class: 'portal-container',
        text: () => 'Target Container'
      }),
      
      // Content with deferred teleport (waits for target)
      h('div', {
        if: () => ctx.showContent,
        teleport: { 
          to: '#dynamic-target', 
          defer: true 
        },
        class: 'teleported-content',
        text: () => 'I wait for the target!'
      }) : null
    ])
});
```

**What happens:**
1. Component mounts, `#dynamic-target` doesn't exist yet
2. Defer sets up `MutationObserver` watching for target
3. When "Show Target" clicked, target is created
4. Observer detects target and teleports content immediately
5. If component unmounts before target appears, observer auto-cleanup

### Reactive Teleport Target

```js
JQRH.mount('#app', {
  state: { 
    currentPortal: '#portal-a',
    portals: ['#portal-a', '#portal-b', '#portal-c']
  },
  
  render: (h, ctx) => 
    h('div', {}, [
      // Switcher
      h('div', {}, [
        h.for(
          ({state}) => state.portals,
          (p) => p,
          (portal) => h('button', {
            on: { click: () => ctx.state.currentPortal = portal },
            text: () => portal
          })
        )
      ]),
      
      // Content moves between portals reactively
      h('div', {
        teleport: ({state}) => state.currentPortal,
        class: 'dynamic-content',
        text: ({state}) => `Currently in ${state.currentPortal}`
      }),
      
      // Portal containers
      h('div', { attr: { id: 'portal-a' } }),
      h('div', { attr: { id: 'portal-b' } }),
      h('div', { attr: { id: 'portal-c' } })
    ])
});
```

### Combining Teleport with Other Attributes

Teleport works with all other attributes:

```js
h('div', {
  teleport: { to: 'body', defer: true },
  class: ({state}) => ({ 'modal': true, 'active': state.isOpen }),
  style: ({state}) => ({ zIndex: state.zIndex }),
  show: ({state}) => state.isVisible,
  on: { click: ({state}) => state.handleClick() },
  text: ({state}) => state.message
})
```

### Technical Notes

**Timing:** `teleport` is applied after element creation but before it's attached to its natural parent. The move happens in a microtask to ensure `el.parentNode` exists.

**Cleanup:** When element is destroyed (component unmount, conditional render), it's automatically removed from teleport target. Defer observers also auto-cleanup.

**Reactivity:** When `teleport` value changes (reactive target), element moves to new location. With `defer`, only the **first** teleport attempt uses defer; subsequent reactive moves are immediate.

**No arg form:** Unlike template `$:teleport:to.<modifiers>`, `h()` only has object/string form because attributes are already plain JavaScript (no need for modifier parsing).

### Best Practices

**1. Use string form for static targets:**
```js
// ✅ Good - simple and clear
h('div', { teleport: 'body' })

// ⚠️ Verbose - unnecessary for static
h('div', { teleport: { to: 'body' } })
```

**2. Use defer for dynamic targets:**
```js
// ✅ Good - waits for target
h('div', { teleport: { to: '#late-target', defer: true } })

// ❌ Bad - will error if target doesn't exist
h('div', { teleport: '#late-target' })
```

**3. Always teleport overlays:**
```js
// ✅ Good - modal at body level
h('div', {
  teleport: 'body',
  class: 'modal-overlay'
})
```

**4. Use reactive form for dynamic portals:**
```js
// ✅ Good - portal changes reactively
h('div', { teleport: ({state}) => state.currentPortal })
```

## List Rendering: h.for()

### Basic List

```js
h.for(
  ({state}) => state.items,       // Items array
  (item) => item.id,            // Key function
  (item) => h('li', { text: () => item.name })  // Builder function
)
```

### Complete Example

```html live:no-code
<div id="demo-list" class="en-card en-card__body">
  <!-- Rendered by JQR-H -->
</div>
```

```html
<div id="demo-list">
  <!-- Rendered by JQR-H -->
</div>
```

```js inject show
JQRH.mount('#demo-list', {
  state: {
    users: [
      { id: 1, name: 'Alice', role: 'Admin' },
      { id: 2, name: 'Bob', role: 'User' },
      { id: 3, name: 'Carol', role: 'User' }
    ]
  },
  methods: {
    removeUser(id) {
      this.users = this.users.filter(u => u.id !== id);
    }
  },
  render: (h, ctx) => 
    h('div', {}, [
      h('h4', null, ['User List']),
      h('ul', { class: 'en-list-none' }, [
        h.for(
          ({state}) => state.users,
          (user) => user.id,
          (user) => h('li', { class: 'en-flex en-items-center en-justify-between en-mb-2 en-p-2 en-bg-gray-50 en-rounded' }, [
            h('div', {}, [
              h('strong', { text: () => user.name }),
              h('span', { class: 'en-text--muted en-ml-2', text: () => `(${user.role})` })
            ]),
            h('button', {
              class: 'en-btn en-btn--sm en-btn--danger',
              on: { click: () => ctx.removeUser(user.id) },
              text: () => 'Remove'
            })
          ])
        )
      ])
    ])
});
```

```js cleanup
onUnmount(() => {
  JQRH.unmount('#demo-list');
})
```

### Why Key Function?

The key function enables **efficient list updates**:

```js
h.for(
  ({state}) => state.items,
  (item) => item.id,  // ← Identifies which item is which
  (item) => h('div', { text: () => item.name })
)
```

When the list changes:
- Items with **same key** → reused (not rebuilt)
- Items with **new key** → created
- Items with **removed key** → destroyed

**Without keys:** All items rebuild on every change (slow + loses state).

### List with Index

```js
h.for(
  ({state}) => state.items,
  (item, index) => index,  // Use index as key (not recommended for dynamic lists)
  (item, index) => h('li', { text: () => `${index + 1}. ${item}` })
)
```

⚠️ **Warning:** Using index as key can cause issues when list is reordered.

## References

Save element references:

```js
JQRH.mount('#app', {
  state: {},
  mounted() {
    // Access ref after mount
    console.log(this.$refs.myInput);
    this.$refs.myInput.focus();
  },
  render: (h) => 
    h('input', {
      ref: 'myInput',
      class: 'en-input',
      attr: { placeholder: 'Type here...' }
    })
});
```

**Note:** Refs are available in `mounted()` hook, not `created()`.

## Component Nodes

### By Registry Name

```js
// Assumes 'user-card' registered via $.jqr.register()
h('user-card', {
  prop: {
    userId: ({state}) => state.currentUserId
  }
})
```

### By Component Reference

```js
const UserCard = {
  props: { userId: null },
  // ... component definition
};

h(UserCard, {
  prop: {
    userId: ({state}) => state.currentUserId
  }
})
```

See [Components with h()](./components-with-h.md) for details.

## Advanced: Custom Directives

Custom directives work differently in `h()` compared to templates - values are **plain JavaScript**, not expression strings.

### Registering Directives

First, register the directive globally (same as for templates):

```js
$.jqr.directive('tooltip', {
  bind(el, ctx) {
    el.title = ctx.raw();  // Value already resolved, not an expression string
  },
  update(el, ctx) {
    el.title = ctx.raw();
  },
  unbind(el) {
    el.removeAttribute('title');
  }
});
```

**Important:** Directives must use `ctx.raw()` to be compatible with `h()`. Using only `ctx.evaluate(ctx.expr)` won't work because `expr` is always empty in render functions.

**Compatible pattern:**
```js
$.jqr.directive('myDir', {
  update(el, ctx) {
    // Works in both templates and h()
    const value = ctx.raw ? ctx.raw() : ctx.evaluate(ctx.expr);
    el.textContent = value;
  }
});
```

### Using Directives in h()

Use via `directive` attribute with the same key format as `$:bind` objects:

```js
h('button', {
  directive: {
    // Basic directive
    'tooltip': ({state}) => state.hint,
    
    // With argument (name:arg)
    'copy:href': ({state}) => state.shareUrl,
    
    // With modifiers (name.mod1.mod2)
    'highlight.once': true,
    
    // Static value (no function)
    'badge': 'New'
  },
  text: () => 'Hover me'
})
```

**Key differences from templates:**

| Template | h() |
|----------|-----|
| `$:tooltip="message"` | `directive: { tooltip: ({state}) => state.message }` |
| Value is expression string | Value is plain JS (static or arrow function) |
| Accessed via `ctx.evaluate()` | Accessed via `ctx.raw()` |
| Modifiers in attribute name | Modifiers in key: `'name.mod1.mod2'` |

### Complete Example

```html live:no-code
<div id="demo-directive" class="en-card en-p-3">
  <!-- Rendered by JQR-H -->
</div>
```

```html
<div id="demo-directive">
  <!-- Rendered by JQR-H -->
</div>
```

```js inject show
// Register custom directive
JQR.directive('tooltip', {
  bind(el, ctx) {
    const value = ctx.raw ? ctx.raw() : ctx.evaluate(ctx.expr);
    el.title = value;
    el.style.cursor = 'help';
    el.style.borderBottom = '1px dotted currentColor';
  },
  update(el, ctx) {
    const value = ctx.raw ? ctx.raw() : ctx.evaluate(ctx.expr);
    el.title = value;
  },
  unbind(el) {
    el.removeAttribute('title');
    el.style.cursor = '';
    el.style.borderBottom = '';
  }
});

JQRH.mount('#demo-directive', {
  state: { 
    tooltip: 'This is a helpful tooltip!',
    count: 0
  },
  render: (h, ctx) => 
    h('div', { class: 'en-grid en-grid--cols-2' }, [
      h('div', {
        class: 'en-form-display en-form-display--sm',
        directive: {
          tooltip: ({state}) => state.tooltip
        },
        text: () => 'Hover over me'
      }),
      h('button', {
        class: 'en-btn en-btn--sm en-ml-3',
        directive: {
          tooltip: ({state}) => `Clicked ${state.count} times`
        },
        on: { click: () => ctx.state.count++ },
        text: ({state}) => `Count: ${state.count}`
      })
    ])
});
```

```js cleanup
onUnmount(() => {
  JQRH.unmount('#demo-directive');
})
```

**Notes:**
- Directives not registered via `$.jqr.directive()` trigger a warning
- `update` is automatically wrapped in `effect()` for reactivity
- Values are always plain JavaScript, not expression strings
- Use `ctx.raw()` in directive implementation for `h()` compatibility

## Advanced: bind Shorthand

```js
h('div', {
  bind: ({state}) => ({
    text: () => state.message,
    class: { active: state.isActive },
    style: { color: state.color }
  })
})
```

Fans out to `text`, `class`, `style`, `attr`, etc. Useful for dynamic attribute objects.

## Practical Patterns

### Wrapper Functions

```js
function Card(h, title, children) {
  return h('div', { class: 'en-card' }, [
    h('div', { class: 'en-card__header' }, [
      h('h3', { class: 'en-card__title', text: () => title })
    ]),
    h('div', { class: 'en-card__body' }, children)
  ]);
}

// Usage
render: (h) => Card(h, 'My Card', [
  h('p', { text: () => 'Content here' })
])
```

### Conditional Wrapper

```js
function maybeWrap(h, condition, wrapper, child) {
  return condition ? wrapper(child) : child;
}

// Usage
maybeWrap(
  h,
  state.showCard,
  (child) => h('div', { class: 'card' }, [child]),
  h('p', { text: () => 'Content' })
)
```

### Dynamic Tag

```js
function DynamicElement(h, tag, attrs, children) {
  return h(tag, attrs, children);
}

// Usage
DynamicElement(
  h,
  state.isHeading ? 'h1' : 'p',
  { text: () => 'Dynamic element' }
)
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
    message: '',
    messages: []
  },
  methods: {
    send() {
      if (this.message.trim()) {
        this.messages.push({
          id: Date.now(),
          text: this.message,
          time: new Date().toLocaleTimeString()
        });
        this.message = '';
      }
    }
  },
  render: (h, ctx) =>
    h('div', { attr: { class: 'en-card__body' } }, [
      h('h4', { text: () => 'Chat Demo' }),

      h('div', {
        class: 'en-mb-3',
        // `show` or `if`
        if: ({state}) => state.messages.length > 0,
        style: {
          maxHeight: '200px',
          overflowY: 'auto',
          border: '1px solid #ddd',
          padding: '10px',
          borderRadius: '4px'
        }
      }, [
        h.for(
          ({ state }) => state.messages,
          (msg) => msg.id,
          (msg) => h('div', { class: 'en-mb-2' }, [
            h('div', {
              class: 'en-bg-blue-50 en-p-2 en-rounded',
              text: () => msg.text
            }),
            h('small', {
              class: 'en-text--muted',
              text: () => msg.time
            })
          ])
        )
      ]),

      // Input form
      h('div', { class: 'en-flex en-gap-2' }, [
        h('input', {
          class: 'en-input',
          attr: {
            placeholder: 'Type message...',
            type: 'text'
          },
          model: 'message',
          on: {
            keyup: (ctx, e) => {
              if (e.key === 'Enter') ctx.methods.send();
            }
          }
        }),
        h('button', {
          class: 'en-btn en-btn--sm en-btn--primary',
          on: { click: (ctx) => ctx.methods.send() },
        }, ['Send'])
      ])
    ])
});
```

```js cleanup
onUnmount(() => {
  JQRH.unmount('#demo-complete');
})
```

## Summary

**`h()` creates node descriptions with:**

| Feature | Syntax | Notes |
|---------|--------|-------|
| Tag | `h('div')` | HTML tag or component name |
| Classes | `class: 'card'` or `({state}) => {...}` | String, array, object, or function |
| Styles | `style: {...}` or `({state}) => {...}` | Object or CSS string |
| Attributes | `attr: { href: '...' }` | Static or reactive |
| Events | `on: { click: (s, e) => ... }` | All DOM events |
| Text (reactive) | `text: ({state}) => state.message` | Arrow function, auto-stringified |
| Text (static) | `h('p', {}, ['text'])` | Via children array |
| HTML | `html: ({state}) => state.html` | Use with caution |
| Visibility | `show: ({state}) => state.visible` | Toggles display |
| Teleport | `teleport: '#target'` or `{ to, defer, mode }` | Move to different DOM location |
| Model (string) | `model: 'fieldName'` | Simple two-way binding |
| Model (with modifiers) | `model: { state: 'query', debounce: 500 }` | Debounce/throttle/lazy |
| Model (computed) | `model: { get, set }` | Custom getter/setter logic |
| Ref | `ref: 'myRef'` | Save to `$refs` |
| Children | `[h('p'), h('div')]` | Array of nodes |
| Lists | `h.for(items, key, build)` | Keyed rendering |

**Key Points:**
1. Arrow functions `({state}) => ...` for **reactive** values (depend on state)
2. Plain values or children array for **static** content (captured once)
3. No re-rendering - updates are fine-grained via `effect()`
4. Use `h.for()` for dynamic lists with keys
5. Refs available in `mounted()` hook
6. Extract helpers to keep code clean
7. **Model modifiers now supported** - `debounce`, `throttle`, `lazy` via object syntax
8. **Computed model** - `{ get, set }` for custom getter/setter logic
9. **Custom directives** use `ctx.raw()`, not `ctx.evaluate()`

### Static vs Reactive Comparison

```js
const theme = 'dark';
const state = { theme: 'dark', count: 0 };

// ✅ Static - captured once at render time
h('div', {}, [theme])                    // Never changes
h('div', { class: 'card' })              // Never changes

// ✅ Reactive - updates when state changes
h('div', { text: ({state}) => state.theme })       // Updates on state.theme change
h('div', { class: ({state}) => state.theme })      // Updates on state.theme change

// ❌ Wrong - won't be reactive!
h('div', { text: state.theme })          // Evaluated once, then static

// ✅ Mixed - static + reactive
h('div', {}, [
  'Theme: ',                             // Static
  ({state}) => state.theme                         // Reactive
])
```

**Rule of thumb:**
- **State-dependent?** → Use arrow function `({state}) => ...`
- **Constant or computed once?** → Use plain value or children array

## Next Steps

- [Components with h()](./components-with-h.md) - Build nested components
- [Render Functions Guide](./render-function.md) - Complete patterns
- [API Reference](../reference/api-reference.md) - Full API docs
