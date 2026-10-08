<!-- {
  "path": "jqr-h/guides/slots-in-h",
  "name": "docs-jqr-jqrh-guides-slots-in-h",
  "meta": {
    "title": "Slots with Render Functions",
    "order": 6
  }
} -->

# Slots with Render Functions

Slots allow parent components to pass content to child components. JQR-H supports both default and named slots when using `h()` to build components.

## Basic Concept

### Template-Based Slots (Review)

In template-based components:

```html
<!-- Parent -->
<div $:component="card">
  <p>This content goes into the slot</p>
</div>

<!-- Child template (id="card") -->
<div class="card">
  <slot $:default></slot>
</div>
```

### h()-Based Slots

With `h()`, slots are passed as the third argument:

```js
// Parent render function
render(h) {
  return h('card', {}, [
    h('p', {}, ['This content goes into the slot'])
  ]);
}
```

The child component's `<template>` defines where slots appear.

## Default Slots

### Passing Default Slot Content

Pass children as an array (third argument):

```js
render(h, ctx) {
  return h('card', {}, [
    h('h2', {}, ['Title']),
    h('p', {}, ['This is the card content']),
    h('button', { 
      on: { click: () => ctx.close() } 
    }, ['Close'])
  ]);
}
```

### Receiving Default Slot

The child component's template uses `<slot $:default>`:

```html
<!-- Template id="card" -->
<div class="card">
  <div class="card-body">
    <slot $:default></slot>
  </div>
</div>
```

**Result:** The `h2`, `p`, and `button` from the parent are inserted where `<slot $:default>` appears.

## Named Slots

### Passing Named Slots

Instead of an array, pass an **object** with slot names:

```js
render(h, ctx) {
  return h('modal', {}, {
    default: [
      h('p', {}, ['This is the main content'])
    ],
    header: [
      h('h2', {}, ['Modal Title'])
    ],
    footer: [
      h('button', { 
        on: { click: () => ctx.close() } 
      }, ['Close'])
    ]
  });
}
```

### Receiving Named Slots

The child template defines multiple slot outlets:

```html
<!-- Template id="modal" -->
<div class="modal">
  <div class="modal-header">
    <slot $:name="header"></slot>
  </div>
  <div class="modal-body">
    <slot $:default></slot>
  </div>
  <div class="modal-footer">
    <slot $:name="footer"></slot>
  </div>
</div>
```

### Slot Name Matching

- `default: [...]` → `<slot $:default>` or `<slot>` (no name)
- `header: [...]` → `<slot $:name="header">`
- `footer: [...]` → `<slot $:name="footer">`

## Slot Requirements

### Child Must Have Template

Slots **require** the child component to have a registered `<template>`:

```js
// Register child with template
$.jqr.register('card', {
  template: `
    <div class="card">
      <slot $:default></slot>
    </div>
  `
});

// Or use external template
<template id="card">
  <div class="card">
    <slot $:default></slot>
  </div>
</template>
```

**Without template:**

```js
// ❌ This won't work - no template to insert slots into
$.jqr.register('card', {
  render(h) {
    return h('div', { class: 'card' });
  }
});

// ⚠️ devWarn: got slot content, but this component has no <template>
```

### Parent's Slot Content is Reactive

Slot content rendered by the parent **reads parent's state**:

```js
// Parent
render(h, ctx) {
  return h('card', {}, [
    h('p', {
      text: (s) => `Parent count: ${s.count}`  // Tracks PARENT state
    })
  ]);
}
```

The `(s) => s.count` reads from **parent's state**, not the child's.

## Reactive Slot Content

### Dynamic Children

```js
render(h, ctx) {
  return h('card', {}, [
    h('p', { 
      text: (s) => s.message  // Updates when parent's message changes
    })
  ]);
}
```

### Conditional Slots

```js
render(h, ctx) {
  return h('modal', {}, {
    default: [
      h('p', {}, ['Always visible'])
    ],
    footer: ctx.state.showActions ? [
      h('button', { on: { click: () => ctx.save() } }, ['Save']),
      h('button', { on: { click: () => ctx.cancel() } }, ['Cancel'])
    ] : []
  });
}
```

### Lists in Slots

```js
render(h, ctx) {
  return h('card', {}, [
    h('ul', {}, [
      h.for(
        (s) => s.items,
        (item) => item.id,
        (item) => h('li', { text: () => item.name })
      )
    ])
  ]);
}
```

## Complete Example

### Child Component (Card)

```html
<template id="user-card">
  <div class="card">
    <div class="card-header">
      <slot $:name="header"></slot>
    </div>
    <div class="card-body">
      <slot $:default></slot>
    </div>
    <div class="card-footer" $:if="hasFooter">
      <slot $:name="footer"></slot>
    </div>
  </div>
</template>
```

```js
$.jqr.register('user-card', {
  state: {},
  computed: {
    hasFooter() {
      // Check if footer slot has content
      return this.$el.find('[data-slot="footer"]').children().length > 0;
    }
  }
});
```

### Parent Component (h-based)

```js
mount('#app', {
  state: {
    user: { name: 'Alice', role: 'Admin' },
    editing: false
  },
  methods: {
    edit({ state }) {
      state.editing = true;
    },
    save({ state }) {
      state.editing = false;
      console.log('Saved:', state.user);
    }
  },
  render(h, ctx) {
    return h('user-card', {}, {
      header: [
        h('h2', { text: (s) => s.user.name }),
        h('span', { 
          class: 'badge',
          text: (s) => s.user.role 
        })
      ],
      default: [
        ctx.state.editing 
          ? h('input', { model: 'user.name' })
          : h('p', { text: (s) => `Hello, ${s.user.name}!` })
      ],
      footer: [
        ctx.state.editing 
          ? h('button', { on: { click: () => ctx.save() } }, ['Save'])
          : h('button', { on: { click: () => ctx.edit() } }, ['Edit'])
      ]
    });
  }
});
```

## Scoped Slots

Scoped slots allow the **child** to pass data back to the **parent's** slot content.

### Child Exposes Slot Props

```html
<template id="list">
  <ul>
    <li $:for="(item, i) in items" $:key="i">
      <slot $:default $:scope="{ item: item, index: i }"></slot>
    </li>
  </ul>
</template>
```

```js
$.jqr.register('list', {
  props: {
    items: { default: () => [] }
  }
});
```

### Parent Receives Slot Props

In `h()`, scoped slots are passed as **functions**:

```js
render(h, ctx) {
  return h('list', {
    prop: {
      items: (s) => s.users
    }
  }, {
    default: [
      // Slot content receives { item, index } from child
      (slotProps) => h('div', {}, [
        h('strong', { text: () => slotProps.item.name }),
        h('span', { text: () => ` (#${slotProps.index})` })
      ])
    ]
  });
}
```

**Note:** Scoped slot support in `h()` requires the slot content to be a **function** that receives `slotProps`.

### Scoped Slot Limitations

Current JQR-H implementation has limited scoped slot support:

- ✅ Scoped slots work in **template-based** components
- ⚠️ In `h()`, scoped slots require manual handling (see Advanced Patterns below)

## Advanced Patterns

### Fallback Content

Child template can provide fallback when parent doesn't pass content:

```html
<template id="card">
  <div class="card">
    <div class="card-header">
      <slot $:name="header">
        <!-- Fallback if no header slot provided -->
        <h3>Default Title</h3>
      </slot>
    </div>
    <slot $:default>
      <!-- Fallback if no default slot provided -->
      <p>No content provided</p>
    </slot>
  </div>
</template>
```

```js
// Parent doesn't pass header - fallback is used
render(h) {
  return h('card', {}, [
    h('p', {}, ['Only body content'])
  ]);
}
```

### Conditional Named Slots

```js
render(h, ctx) {
  const slots = {
    default: [
      h('p', {}, ['Main content'])
    ]
  };
  
  if (ctx.state.showHeader) {
    slots.header = [
      h('h2', {}, ['Title'])
    ];
  }
  
  if (ctx.state.showFooter) {
    slots.footer = [
      h('button', {}, ['Action'])
    ];
  }
  
  return h('card', {}, slots);
}
```

### Nested Component Slots

Slots can contain nested components:

```js
render(h, ctx) {
  return h('modal', {}, {
    default: [
      h('user-card', {
        prop: { user: (s) => s.currentUser }
      }, [
        h('p', {}, ['Nested slot content'])
      ])
    ],
    footer: [
      h('button-group', {}, [
        h('button', {}, ['OK']),
        h('button', {}, ['Cancel'])
      ])
    ]
  });
}
```

### Render Functions in Slots

Pass render logic as functions:

```js
render(h, ctx) {
  const renderItem = (item) => h('div', {
    class: 'item',
    text: () => item.name
  });
  
  return h('list', {}, [
    h.for(
      (s) => s.items,
      (item) => item.id,
      (item) => renderItem(item)
    )
  ]);
}
```

## Slots vs Props

### When to Use Slots

**Use slots for:**
- Rich HTML content (headings, paragraphs, images)
- Layout composition (header, body, footer)
- Parent-controlled rendering

```js
// ✅ Good - complex layout with slots
h('modal', {}, {
  header: [h('h2', {}, ['Title'])],
  default: [h('form', {}, [/* form fields */])],
  footer: [h('button', {}, ['Submit'])]
})
```

### When to Use Props

**Use props for:**
- Primitive data (strings, numbers, booleans)
- Objects/arrays the child controls
- Child-rendered content

```js
// ✅ Good - data passed as props
h('user-card', {
  prop: {
    name: (s) => s.user.name,
    role: (s) => s.user.role,
    avatar: (s) => s.user.avatarUrl
  }
})
```

## Debugging Slots

### Enable Dev Mode

```js
$.jqr.config({ devMode: true });
```

Shows warnings for:
- Slots passed to template-less components
- Slot names that don't match child's template

### Unused Slot Warning

```js
render(h) {
  return h('card', {}, {
    default: [h('p', {}, ['Body'])],
    nonexistent: [h('p', {}, ['Oops'])]  // No matching <slot $:name="nonexistent">
  });
}
// ⚠️ devWarn: got a "nonexistent" slot, but template has no matching <slot $:name="nonexistent">
```

### Missing Template Warning

```js
render(h) {
  return h('no-template-component', {}, [
    h('p', {}, ['This will be discarded'])
  ]);
}
// ⚠️ devWarn: got slot content, but this component has no <template>
```

## Limitations

### 1. Child Must Have Template

```js
// ❌ Won't work - render-based child can't receive slots
$.jqr.register('child', {
  render(h) {
    return h('div', {}, ['Child content']);
  }
});

render(h) {
  return h('child', {}, [
    h('p', {}, ['Parent content'])  // Discarded with warning
  ]);
}
```

**Solution:** Use props instead, or give child a template.

### 2. Scoped Slots Have Limited Support

Manual scoped slot handling:

```js
// Parent manually extracts slot props
render(h, ctx) {
  return h('list', {
    prop: {
      items: (s) => s.items,
      renderItem: () => (item, index) => h('div', {
        text: () => `${item.name} (#${index})`
      })
    }
  });
}

// Child calls parent's renderItem
$.jqr.register('list', {
  props: { items: {}, renderItem: {} },
  render(h, ctx) {
    return h('ul', {}, [
      h.for(
        (s) => s.items,
        (item) => item.id,
        (item, i) => ctx.props.renderItem(item, i)
      )
    ]);
  }
});
```

### 3. Dynamic Slot Names Not Supported

```js
// ❌ Won't work - slot names must be static strings
const slotName = 'dynamic';
render(h) {
  return h('card', {}, {
    [slotName]: [h('p', {}, ['Content'])]  // Not matched to template
  });
}
```

## Best Practices

### 1. Prefer Named Slots for Clarity

```js
// ✅ Good - explicit slot names
h('modal', {}, {
  header: [...],
  default: [...],
  footer: [...]
})

// ❌ Less clear - default only
h('modal', {}, [
  ...headerContent,
  ...bodyContent,
  ...footerContent
])
```

### 2. Always Register Templates

```js
// ✅ Good - child has template for slots
<template id="card">
  <div class="card">
    <slot $:name="header"></slot>
    <slot $:default></slot>
  </div>
</template>

$.jqr.register('card', {});
```

### 3. Use Descriptive Slot Names

```js
// ✅ Good - clear purpose
{ header: [...], footer: [...], actions: [...] }

// ❌ Bad - unclear
{ slot1: [...], slot2: [...], slot3: [...] }
```

### 4. Document Expected Slots

```js
/**
 * Modal component
 * 
 * Slots:
 * - header: Modal title area
 * - default: Main modal content
 * - footer: Action buttons area
 */
$.jqr.register('modal', {
  template: '...'
});
```

## Summary

**Key Concepts:**

1. **Default slot:** Array as third argument → `h('comp', {}, [...])`
2. **Named slots:** Object as third argument → `h('comp', {}, { header: [...], footer: [...] })`
3. **Child needs template:** Slots require registered `<template>` on child
4. **Parent's state:** Slot content is reactive to parent's state
5. **Slot matching:** Slot names must match child's `<slot $:name="...">`

**When to use slots:**
- Rich HTML content composition
- Flexible layouts (headers, footers, sidebars)
- Parent-controlled rendering

**When to use props:**
- Simple data passing
- Child-controlled rendering
- Primitive values

## Next Steps

- [Components with h()](./components-with-h.md) - Building components
- [Architecture](./architecture.md) - Understanding reactivity
- [Slots Guide](../../jqr/guides/slots.md) - Template-based slots
- [h() Function Reference](./h-function.md) - Complete `h()` API
