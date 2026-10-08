<!-- {
  "path": "jqr-e/guides/h-integration",
  "name": "docs-jqr-e-guides-h-integration",
  "meta": {
    "title": "h() Integration",
    "order": 8
  }
} -->

# h() Integration

Learn how to use custom elements seamlessly with JQR-H's `h()` function for powerful render-based components.

## Overview

Custom elements defined with `defineElement()` work natively with `h()`:

```js
// Define custom element
JQRE.defineElement('user-card', userCardDef);

// Use in render function
render((h) => 
  h('user-card', {
    attrs: { userId: 123 }
  })
, '#app');
```

**Benefits:**
- **Type safety** - Full TypeScript support
- **Reactive bindings** - Props update automatically
- **Event handling** - Native CustomEvent integration
- **Slot content** - Rich composition patterns

## Basic Usage

### Creating Elements

Use the tag name as the element type:

```js
import { h, render } from '@en32/jqr-h';

// Define element first
JQRE.defineElement('greeting-card', {
  props: {
    name: 'World',
    greeting: 'Hello'
  },
  template: `
    <div>
      <h3 $:text="greeting + ', ' + name + '!'"></h3>
    </div>
  `
});

// Use with h()
render((h) => 
  h('greeting-card', {
    attrs: {
      name: 'Alice',
      greeting: 'Hi'
    }
  })
, '#app');
```

### Syntax

```js
h('custom-element', {
  attrs: { /* ... */ },      // Props as attributes
  on: { /* ... */ },         // Event listeners
  class: () => '...',        // Dynamic classes
  style: () => ({ /* */ })   // Dynamic styles
}, [
  // Children (slot content)
])
```

## Passing Props

Props are passed via the `attrs` object:

```html live:no-code
<div id="demo-h-props">
  <div class="en-card en-card__body">
  </div>
</div>
```

```html
<div id="demo-h-props"></div>
```

```js inject show
// Define element
JQRE.defineElement('prop-display', {
  props: {
    title: '',
    count: 0,
    active: false
  },
  template: `
    <div>
      <h4 $:text="title"></h4>
      <p>Count: <strong $:text="count"></strong></p>
      <p>Active: <strong $:text="active ? 'Yes' : 'No'"></strong></p>
    </div>
  `
});

// Use with h()
JQRH.render((h, ctx) => 
  h('div', {}, [
    h('prop-display', {
      attrs: {
        title: () => 'Dynamic Title',
        count: () => ctx.counter,
        active: () => ctx.isActive
      }
    })
  ])
, '#demo-h-props', {
  state: {
    counter: 5,
    isActive: true
  }
});
```

```js cleanup
onUnmount(() => {
  $('#demo-h-props').component('destroy');
});
```

### Static Props

For static values, use plain values:

```js
h('user-card', {
  attrs: {
    userId: 123,
    name: 'Alice'
  }
})
```

### Reactive Props

For reactive values, use functions:

```js
h('user-card', {
  attrs: {
    userId: (ctx) => ctx.currentUserId,
    name: (ctx) => ctx.currentUserName
  }
})
```

**Important:** Reactive props require function form for reactivity tracking.

## Event Handling

Custom elements emit native CustomEvent instances that integrate with `h()`:

```html live:no-code
<div id="demo-h-events">
  <div class="en-card en-card__body">
  </div>
</div>
```

```html
<div id="demo-h-events"></div>
```

```js inject show
// Define element with events
JQRE.defineElement('click-counter', {
  state: {
    count: 0
  },
  methods: {
    increment() {
      this.count++;
      this.$emit('increment', this.count);
    }
  },
  template: `
    <button class="en-btn en-btn--sm" $:click="increment">
      Clicks: <span $:text="count"></span>
    </button>
  `
});

// Use with event handler
JQRH.render((h, ctx) => 
  h('div', {}, [
    h('p', { text: () => `Total: ${ctx.total}` }),
    h('click-counter', {
      on: {
        increment: (ctx, event) => {
          ctx.total = event.detail;
          console.log('Received increment event:', event.detail);
        }
      }
    })
  ])
, '#demo-h-events', {
  state: {
    total: 0
  }
});
```

```js cleanup
onUnmount(() => {
  $('#demo-h-events').component('destroy');
});
```

### Event Signature

```js
h('my-element', {
  on: {
    eventName: (ctx, event) => {
      // ctx: render context
      // event: CustomEvent instance
      console.log(event.detail);  // Event payload
    }
  }
})
```

### Multiple Events

```js
h('form-input', {
  on: {
    input: (ctx, ev) => {
      ctx.value = ev.detail;
    },
    blur: (ctx, ev) => {
      ctx.touched = true;
    },
    focus: (ctx, ev) => {
      ctx.focused = true;
    }
  }
})
```

## Slot Content

Pass children as slot content:

```html live:no-code
<div id="demo-h-slots">
  <div class="en-card en-card__body">
  </div>
</div>
```

```html
<div id="demo-h-slots"></div>
```

```js inject show
// Define element with slots
JQRE.defineElement('card-wrapper', {
  template: `
    <div class="en-card">
      <div class="en-card__header">
        <slot name="header">Default Header</slot>
      </div>
      <div class="en-card__body">
        <slot>Default Body</slot>
      </div>
    </div>
  `
});

// Use with slot content
JQRH.render((h) => 
  h('card-wrapper', {}, [
    h('h3', { slot: 'header' }, ['Custom Header']),
    h('p', {}, ['This is the body content'])
  ])
, '#demo-h-slots');
```

```js cleanup
onUnmount(() => {
  $('#demo-h-slots').component('destroy');
});
```

### Default Slot

Children without `slot` attribute go to default slot:

```js
h('card-wrapper', {}, [
  h('p', {}, ['Default slot content']),
  h('span', {}, ['More content'])
])
```

### Named Slots

Use `slot` attribute for named slots:

```js
h('card-wrapper', {}, [
  h('h3', { slot: 'header' }, ['Header Content']),
  h('p', {}, ['Body Content']),
  h('div', { slot: 'footer' }, ['Footer Content'])
])
```

### Using h.slot()

For scoped slots, use `h.slot()`:

```js
h('data-table', {}, [
  h.slot('item', ['userId'], (ctx, userId) => 
    h('div', {}, [
      h('span', { text: () => `User: ${userId}` })
    ])
  )
])
```

## Dynamic Classes and Styles

Apply reactive classes and styles:

```html live:no-code
<div id="demo-h-classes">
  <div class="en-card en-card__body">
  </div>
</div>
```

```html
<div id="demo-h-classes"></div>
```

```js inject show
JQRE.defineElement('styled-box', {
  template: `
    <div>
      <slot></slot>
    </div>
  `
}, {
  display: 'block'
});

JQRH.render((h, ctx) => 
  h('div', {}, [
    h('button', {
      class: 'en-btn en-btn--sm',
      on: {
        click: () => {
          ctx.variant = ctx.variant === 'primary' ? 'danger' : 'primary';
        }
      }
    }, ['Toggle Variant']),
    h('styled-box', {
      class: (ctx) => `en-alert en-alert--${ctx.variant}`,
      style: (ctx) => ({
        marginTop: '10px',
        padding: '10px'
      })
    }, [
      h('span', { text: (ctx) => `Current variant: ${ctx.variant}` })
    ])
  ])
, '#demo-h-classes', {
  state: {
    variant: 'primary'
  }
});
```

```js cleanup
onUnmount(() => {
  $('#demo-h-classes').component('destroy');
});
```

### Dynamic Class

```js
h('my-element', {
  class: (ctx) => ctx.isActive ? 'active' : 'inactive'
})
```

### Dynamic Style

```js
h('my-element', {
  style: (ctx) => ({
    backgroundColor: ctx.color,
    opacity: ctx.visible ? 1 : 0
  })
})
```

## Complex Example

Combining props, events, slots, and styling:

```html live:no-code
<div id="demo-h-complex">
  <div class="en-card en-card__body">
  </div>
</div>
```

```html
<div id="demo-h-complex"></div>
```

```js inject show
// Define expandable panel
JQRE.defineElement('expandable-panel', {
  props: {
    title: '',
    expanded: false
  },
  methods: {
    toggle() {
      this.$setProps({ expanded: !this.$props.expanded });
      this.$emit('toggle', this.$props.expanded);
    }
  },
  template: `
    <div>
      <div class="panel-header" style="cursor: pointer; padding: 10px; background: #f5f5f5; border-radius: 4px" $:click="toggle">
        <strong $:text="title"></strong>
        <span $:text="$props.expanded ? '▼' : '▶'" style="float: right"></span>
      </div>
      <div $:if="$props.expanded" style="padding: 10px; border: 1px solid #e0e0e0; border-top: none; border-radius: 0 0 4px 4px">
        <slot></slot>
      </div>
    </div>
  `
}, {
  reflect: ['expanded'],
  display: 'block'
});

// Use in render function
JQRH.render((h, ctx) => 
  h('div', {}, [
    h('p', { text: () => `Panel is ${ctx.isExpanded ? 'expanded' : 'collapsed'}` }),
    h('expandable-panel', {
      attrs: {
        title: () => 'Click to expand',
        expanded: () => ctx.isExpanded
      },
      on: {
        toggle: (ctx, event) => {
          ctx.isExpanded = event.detail;
        }
      },
      style: () => ({
        marginTop: '10px'
      })
    }, [
      h('p', {}, ['This is the panel content!']),
      h('button', {
        class: 'en-btn en-btn--sm',
        on: {
          click: () => console.log('Button clicked inside panel')
        }
      }, ['Action'])
    ])
  ])
, '#demo-h-complex', {
  state: {
    isExpanded: false
  }
});
```

```js cleanup
onUnmount(() => {
  $('#demo-h-complex').component('destroy');
});
```

## Parent-Child Communication

Custom elements in `h()` enable clean parent-child patterns:

### Props Down

```js
// Parent passes data down
h('child-element', {
  attrs: {
    userId: (ctx) => ctx.selectedUserId,
    config: (ctx) => ctx.appConfig
  }
})
```

### Events Up

```js
// Child emits events up
h('child-element', {
  on: {
    select: (ctx, event) => {
      ctx.selectedId = event.detail.id;
    },
    delete: (ctx, event) => {
      ctx.removeItem(event.detail.id);
    }
  }
})
```

### Bidirectional Binding

```js
// Two-way binding pattern
h('input-field', {
  attrs: {
    value: (ctx) => ctx.inputValue
  },
  on: {
    'update:value': (ctx, event) => {
      ctx.inputValue = event.detail;
    }
  }
})
```

## Integration with Components

Mix custom elements with regular `h()` components:

```js
render((h, ctx) => 
  h('div', {}, [
    // Native elements
    h('header', {}, [
      h('h1', {}, ['My App'])
    ]),
    
    // Custom elements
    h('nav-menu', {
      attrs: {
        active: (ctx) => ctx.currentRoute
      }
    }),
    
    // Regular h() components
    h(DataTable, {
      props: {
        data: (ctx) => ctx.tableData
      }
    }),
    
    // More custom elements
    h('footer-bar', {})
  ])
, '#app');
```

## Lifecycle Integration

Custom elements follow standard `h()` lifecycle:

```js
render((h, ctx) => 
  h('my-element', {
    // Element mounts when condition is true
    ...(ctx.showElement && {
      attrs: { visible: true }
    })
  })
, '#app');

// Update condition
ctx.showElement = true;  // Element mounts
ctx.showElement = false; // Element unmounts
```

**Mounting:**
- Element created when `h()` renders it
- Component mounts (via parent ownership)
- Lifecycle hooks run (`created`, `mounted`)

**Unmounting:**
- Element removed when no longer rendered
- Component destroys (parent owns it)
- Lifecycle hooks run (`destroyed`)

## Async Components

Use async components as custom elements:

```js
// Define async component
$.jqr.registerAsync('lazy-widget', {
  load: () => import('./widget.js').then(m => m.default),
  props: { title: '' },
  loading: { template: '<div>Loading...</div>' }
});

// Define as custom element
JQRE.defineElement('lazy-widget');

// Use with h()
render((h) => 
  h('lazy-widget', {
    attrs: {
      title: 'Dashboard'
    }
  })
, '#app');
```

**Loading Flow:**
1. Element renders with loading placeholder
2. Definition loads asynchronously
3. Component mounts with loaded definition
4. `$mounted` promise resolves

## TypeScript Support

Full type safety with custom elements:

```typescript
import { h, render } from '@en32/jqr-h';
import { defineElement } from '@en32/jqr/element';

// Define element
defineElement('typed-button', {
  props: {
    variant: 'default' as 'default' | 'primary' | 'danger',
    disabled: false
  },
  emits: ['click']
});

// Use with types
render((h) => 
  h('typed-button', {
    attrs: {
      variant: 'primary',  // Type checked!
      disabled: false
    },
    on: {
      click: (ctx, event) => {
        // event.detail is typed
      }
    }
  })
, '#app');
```

## Best Practices

### ✅ Do

- **Use reactive props** - Wrap dynamic values in functions
- **Handle events** - Listen to component events via `on`
- **Pass slot content** - Use children for rich composition
- **Mix with h() components** - Combine both patterns
- **Type your elements** - Use TypeScript for safety

### ❌ Don't

- **Don't mutate props directly** - Use event emitters
- **Don't forget reactivity** - Wrap dynamic attrs in functions
- **Don't over-nest** - Keep component tree shallow
- **Don't skip slot attributes** - Named slots need `slot` attr
- **Don't access DOM directly** - Use component APIs

## Common Patterns

### Form Controls

```js
render((h, ctx) => 
  h('form', {}, [
    h('custom-input', {
      attrs: {
        value: (ctx) => ctx.username,
        label: 'Username'
      },
      on: {
        'update:value': (ctx, ev) => {
          ctx.username = ev.detail;
        }
      }
    }),
    h('custom-select', {
      attrs: {
        value: (ctx) => ctx.role,
        options: (ctx) => ctx.roles
      },
      on: {
        change: (ctx, ev) => {
          ctx.role = ev.detail;
        }
      }
    })
  ])
, '#form');
```

### Data Tables

```js
render((h, ctx) => 
  h('data-table', {
    attrs: {
      data: (ctx) => ctx.users,
      columns: (ctx) => ctx.columns
    },
    on: {
      rowClick: (ctx, ev) => {
        ctx.selectedUser = ev.detail;
      },
      sort: (ctx, ev) => {
        ctx.sortColumn = ev.detail.column;
      }
    }
  }, [
    // Custom cell templates
    h.slot('actions', ['row'], (ctx, row) => 
      h('button', {
        on: {
          click: () => ctx.deleteUser(row.id)
        }
      }, ['Delete'])
    )
  ])
, '#app');
```

### Navigation

```js
render((h, ctx) => 
  h('nav-menu', {
    attrs: {
      items: (ctx) => ctx.menuItems,
      active: (ctx) => ctx.currentRoute
    },
    on: {
      navigate: (ctx, ev) => {
        ctx.currentRoute = ev.detail.route;
      }
    }
  })
, '#nav');
```

## Debugging

### Inspect Element

```js
const el = document.querySelector('my-element');

console.log('Instance:', el.$instance);
console.log('Props:', el.$instance.$props);
console.log('State:', el.$instance.$state);
```

### Log Events

```js
h('my-element', {
  on: {
    '*': (ctx, event) => {
      console.log('Event:', event.type, event.detail);
    }
  }
})
```

### Check Mounting

```js
const el = document.querySelector('my-element');

await el.$mounted;
console.log('Element mounted:', el.$instance);
```

## Next Steps

- **[Element Options](./options.md)** - Configure element behavior
- **[Slots](./slots.md)** - Advanced slot patterns
- **[Components with h()](../../jqr-h/guides/components-with-h.md)** - h() component patterns

## Related

- [h() Function](../../jqr-h/guides/h-function.md)
- [Render Functions](../../jqr-h/guides/render-function.md)
- [Defining Elements](./defining-elements.md)
- [Element API](./element-api.md)
