<!-- {
  "path": "jqr-e/guides/slots",
  "name": "docs-jqr-e-guides-slots",
  "meta": {
    "title": "Slots",
    "order": 4
  }
} -->

# Slots

Learn how to use slots for content projection in custom elements, supporting both template-based and render-based components.

## Overview

Slots allow custom elements to accept and display content from their users. JQR-E uses **Light DOM slots** (standard HTML `<slot>` element) for both template and render-based components.

**Key Features:**
- Native HTML `slot` attribute
- Named slots
- Default slot (unnamed content)
- Scoped slots (template-based only)
- `h.slot()` for render functions

## Basic Slots

### Default Slot

Content without `slot` attribute goes to the default slot:

```html live:no-code
<div id="demo-slots-basic">
  <div class="en-card">
    <simple-card>
      <p>This content goes into the default slot!</p>
      <p>Multiple elements are supported.</p>
    </simple-card>
  </div>
</div>
```

```html
<div id="demo-slots-basic">
  <simple-card>
    <p>This content goes into the default slot!</p>
    <p>Multiple elements are supported.</p>
  </simple-card>
</div>
```

```js inject show
JQRE.defineElement('simple-card', {
  template: `
    <div class="en-card">
      <div style="padding: 15px">
        <slot $:default></slot>
      </div>
    </div>
  `
});
```

```js cleanup
onUnmount(() => {
  $('#demo-slots-basic').component('destroy');
});
```

**Template Pattern:**
```html
<slot $:default></slot>
<!-- or -->
<slot></slot>  <!-- Works too, but explicit is clearer -->
```

### Named Slots

Use `slot` attribute to target specific slots:

```html live:no-code
<div id="demo-slots-named">
  <div class="en-card">
    <panel-card>
      <span slot="header">Panel Header</span>
      <p>This is the main content area.</p>
      <div slot="footer">
        <button class="en-btn en-btn--sm">Action</button>
      </div>
    </panel-card>
  </div>
</div>
```

```html
<div id="demo-slots-named">
  <panel-card>
    <span slot="header">Panel Header</span>
    <p>This is the main content area.</p>
    <div slot="footer">
      <button>Action</button>
    </div>
  </panel-card>
</div>
```

```js inject show
JQRE.defineElement('panel-card', {
  template: `
    <div class="en-card" style="padding: 0">
      <div style="padding: 15px; border-bottom: 1px solid #ddd; font-weight: bold; background: #f5f5f5">
        <slot $:name="'header'"></slot>
      </div>
      <div style="padding: 15px">
        <slot $:default></slot>
      </div>
      <div style="padding: 15px; border-top: 1px solid #ddd; background: #f9f9f9">
        <slot $:name="'footer'"></slot>
      </div>
    </div>
  `
});
```

```js cleanup
onUnmount(() => {
  $('#demo-slots-named').component('destroy');
});
```

**Template Pattern:**
```html
<slot $:name="'header'"></slot>
<slot $:name="'footer'"></slot>
```

## Slot Attributes

### Standard HTML Attribute

Use native `slot` attribute (lowercase):

```html
<my-card>
  <h2 slot="title">Card Title</h2>
  <p>Default content</p>
  <div slot="actions">
    <button>OK</button>
  </div>
</my-card>
```

### JQR Directive (Optional)

Can also use `$:slot` directive:

```html
<my-card>
  <h2 $:slot="'title'">Card Title</h2>
  <p>Default content</p>
</my-card>
```

Both syntaxes work identically. Native `slot` is preferred for portability.

## Fallback Content

Provide default content when slot is empty:

```html live:no-code
<div id="demo-slots-fallback">
  <div class="en-flex en-gap-3">
    <div style="flex: 1">
      <greeting-card></greeting-card>
    </div>
    <div style="flex: 1">
      <greeting-card>
        <span>Custom Greeting!</span>
      </greeting-card>
    </div>
  </div>
</div>
```

```html
<div id="demo-slots-fallback">
  <!-- Empty: uses fallback -->
  <greeting-card></greeting-card>
  
  <!-- Has content: overrides fallback -->
  <greeting-card>
    <span>Custom Greeting!</span>
  </greeting-card>
</div>
```

```js inject show
JQRE.defineElement('greeting-card', {
  template: `
    <div class="en-card">
      <div style="padding: 15px">
        <slot $:default>
          <p style="color: #999; font-style: italic">
            Default fallback content
          </p>
        </slot>
      </div>
    </div>
  `
});
```

```js cleanup
onUnmount(() => {
  $('#demo-slots-fallback').component('destroy');
});
```

**Pattern:**
```html
<slot $:default>
  <p>This appears when slot is empty</p>
</slot>
```

## Scoped Slots

Pass data from component to slot content (template-based only):

```html live:no-code
<div id="demo-slots-scoped">
  <div class="en-card">
    <user-list></user-list>
  </div>
</div>
```

```html
<div id="demo-slots-scoped">
  <user-list>
    <template slot="item" $:scope="{ user }">
      <div style="padding: 8px; border: 1px solid #ddd; border-radius: 4px">
        <strong $:text="user.name"></strong>
        <span style="color: #666" $:text="' (' + user.role + ')'"></span>
      </div>
    </template>
  </user-list>
</div>
```

```js inject show
JQRE.defineElement('user-list', {
  state: {
    users: [
      { id: 1, name: 'Alice', role: 'Admin' },
      { id: 2, name: 'Bob', role: 'User' },
      { id: 3, name: 'Charlie', role: 'Editor' }
    ]
  },
  template: `
    <div>
      <div $:for="user in users" $:key="user.id" style="margin-bottom: 8px">
        <slot $:name="'item'" $:scope="{ user }">
          <div style="padding: 8px; background: #f5f5f5">
            <span $:text="user.name"></span>
          </div>
        </slot>
      </div>
    </div>
  `
});
```

```js cleanup
onUnmount(() => {
  $('#demo-slots-scoped').component('destroy');
});
```

**Pattern:**
```html
<!-- In component template -->
<slot $:name="'item'" $:scope="{ user, index }"></slot>

<!-- In usage -->
<template slot="item" $:scope="{ user, index }">
  <div $:text="user.name"></div>
</template>
```

## Render-Based Slots

Use `h.slot()` for render functions (requires JQR-H):

### Basic Usage

```js
JQRE.defineElement('render-card', {
  render: (h) => 
    h('div', { class: () => 'card' }, [
      h('header', {}, [h.slot('header')]),
      h('main', {}, [h.slot()]),  // Default slot
      h('footer', {}, [h.slot('footer')])
    ])
});
```

**Signature:**
```js
h.slot(name?, fallback?)
```

- `name` (string, optional) - Slot name (omit for default)
- `fallback` (Child[], optional) - Fallback content

### With Fallback

```js
JQRE.defineElement('info-box', {
  render: (h) =>
    h('div', { class: () => 'box' }, [
      h.slot('title', [
        h('h3', {}, ['Default Title'])  // Fallback
      ]),
      h.slot(undefined, [
        h('p', {}, ['No content provided'])  // Default fallback
      ])
    ])
});
```

### Complete Example

```html live:no-code
<div id="demo-slots-render">
  <div class="en-card">
    <render-panel>
      <div slot="icon">📦</div>
      <div slot="title">Package Information</div>
      <p>This content uses render-based slots with h.slot()!</p>
      <div slot="actions">
        <button class="en-btn en-btn--sm en-btn--primary">Install</button>
      </div>
    </render-panel>
  </div>
</div>
```

```html
<div id="demo-slots-render">
  <render-panel>
    <div slot="icon">📦</div>
    <div slot="title">Package Information</div>
    <p>This content uses render-based slots!</p>
    <div slot="actions">
      <button>Install</button>
    </div>
  </render-panel>
</div>
```

```js inject show
JQRE.defineElement('render-panel', {
  render: (h) =>
    h('div', { 
      style: () => ({ 
        border: '1px solid #ddd', 
        borderRadius: '8px',
        padding: '0'
      }) 
    }, [
      // Header with icon and title
      h('div', { 
        style: () => ({ 
          display: 'flex', 
          alignItems: 'center', 
          gap: '12px',
          padding: '16px',
          borderBottom: '1px solid #ddd',
          background: '#f5f5f5'
        })
      }, [
        h('div', { 
          style: () => ({ fontSize: '24px' })
        }, [h.slot('icon', [h('span', {}, ['📄'])])]),
        h('div', { 
          style: () => ({ 
            flex: '1', 
            fontWeight: 'bold',
            fontSize: '18px'
          })
        }, [h.slot('title', [h('span', {}, ['Untitled'])])])
      ]),
      
      // Main content
      h('div', { 
        style: () => ({ padding: '16px' })
      }, [h.slot()]),
      
      // Footer actions
      h('div', { 
        style: () => ({ 
          padding: '16px',
          borderTop: '1px solid #ddd',
          background: '#f9f9f9',
          display: 'flex',
          justifyContent: 'flex-end'
        })
      }, [h.slot('actions')])
    ])
}, {
  display: 'block'
});
```

```js cleanup
onUnmount(() => {
  $('#demo-slots-render').component('destroy');
});
```

## Multiple Slots

Components can have unlimited slots:

```html live:no-code
<div id="demo-slots-multiple">
  <div class="en-card">
    <article-layout>
      <div slot="sidebar">
        <h4>Sidebar</h4>
        <ul>
          <li>Link 1</li>
          <li>Link 2</li>
        </ul>
      </div>
      <div slot="header">
        <h2>Article Title</h2>
        <p style="color: #666">By Author Name</p>
      </div>
      <div>
        <p>Main article content goes here in the default slot.</p>
      </div>
      <div slot="related">
        <h4>Related Articles</h4>
        <ul>
          <li>Related 1</li>
          <li>Related 2</li>
        </ul>
      </div>
    </article-layout>
  </div>
</div>
```

```html
<div id="demo-slots-multiple">
  <article-layout>
    <div slot="sidebar">...</div>
    <div slot="header">...</div>
    <div>Main content (default)</div>
    <div slot="related">...</div>
  </article-layout>
</div>
```

```js inject show
JQRE.defineElement('article-layout', {
  template: `
    <div style="display: grid; grid-template-columns: 200px 1fr; grid-template-rows: auto 1fr; gap: 16px">
      <aside style="grid-row: 1 / 3; border-right: 1px solid #ddd; padding-right: 16px">
        <slot $:name="'sidebar'"></slot>
      </aside>
      <header style="padding-bottom: 16px; border-bottom: 1px solid #ddd">
        <slot $:name="'header'"></slot>
      </header>
      <main>
        <slot $:default></slot>
      </main>
      <aside style="grid-column: 2; border-top: 1px solid #ddd; padding-top: 16px; margin-top: 16px">
        <slot $:name="'related'"></slot>
      </aside>
    </div>
  `
});
```

```js cleanup
onUnmount(() => {
  $('#demo-slots-multiple').component('destroy');
});
```

## Slot Behavior

### Content Ownership

Slot content belongs to the **parent scope**, not the component:

```html
<user-card>
  <!-- This content has access to parent's state, not component's -->
  <p $:text="parentMessage"></p>
  <button $:click="parentMethod()">Click</button>
</user-card>
```

### Content Moves, Not Clones

Slot content is **moved** to outlet, not duplicated:

```js
// ✅ Named slot can only be placed ONCE
h.slot('title')  // First placement: moves content
h.slot('title')  // Second placement: empty (content already moved)
```

### Whitespace Handling

Leading/trailing whitespace between tags is ignored:

```html
<!-- These are equivalent -->
<my-card>
  <p>Content</p>
</my-card>

<my-card><p>Content</p></my-card>
```

## Conditional Slots

Check if slot has content:

**Template-based (not directly supported - use fallback):**
```html
<slot $:default>
  <p>No content provided</p>
</slot>
```

**Render-based (check in JavaScript):**
```js
render: (h, ctx) => {
  const hasTitle = ctx.$slots?.title?.length > 0;
  
  return h('div', {}, [
    hasTitle && h('header', {}, [h.slot('title')]),
    h.slot()
  ]);
}
```

## Dynamic Slots

Slot names can be reactive in template-based components:

```html live:no-code
<div id="demo-slots-dynamic">
  <div class="en-card">
    <div $:component:state="{ activeTab: 'home' }">
      <button class="en-btn en-btn--sm" $:click="activeTab = 'home'">Home</button>
      <button class="en-btn en-btn--sm" $:click="activeTab = 'profile'">Profile</button>
      <button class="en-btn en-btn--sm" $:click="activeTab = 'settings'">Settings</button>
      
      <tab-panel style="margin-top: 10px">
        <div slot="home">Home tab content</div>
        <div slot="profile">Profile tab content</div>
        <div slot="settings">Settings tab content</div>
        <input type="hidden" $:attr:data-active="activeTab">
      </tab-panel>
    </div>
  </div>
</div>
```

```html
<div id="demo-slots-dynamic">
  <div $:component:state="{ activeTab: 'home' }">
    <button $:click="activeTab = 'home'">Home</button>
    <button $:click="activeTab = 'profile'">Profile</button>
    
    <tab-panel>
      <div slot="home">Home content</div>
      <div slot="profile">Profile content</div>
      <input type="hidden" $:attr:data-active="activeTab">
    </tab-panel>
  </div>
</div>
```

```js inject show
JQRE.defineElement('tab-panel', {
  computed: {
    activeTab() {
      return this.$el.find('[data-active]').attr('data-active') || 'home';
    }
  },
  template: `
    <div class="en-card" style="padding: 16px">
      <slot $:name="activeTab"></slot>
    </div>
  `
});
```

```js cleanup
onUnmount(() => {
  $('#demo-slots-dynamic').component('destroy');
});
```

## Nested Slots

Slots work with nested components:

```html
<outer-card>
  <inner-card slot="content">
    <p slot="title">Inner Title</p>
    <p>Inner content</p>
  </inner-card>
</outer-card>
```

Each component handles its own slots independently.

## Best Practices

### ✅ Do

- **Use native `slot` attribute** for better portability
- **Provide fallback content** for optional slots
- **Name slots semantically** (`header`, `footer`, not `slot1`, `slot2`)
- **Document required slots** in component docs
- **Keep slot content simple** for better reusability
- **Use scoped slots** for dynamic item rendering

### ❌ Don't

- **Don't place same slot twice** (render-based: content moves)
- **Don't rely on slot content ordering** in code
- **Don't use slots for data passing** (use props/events instead)
- **Don't deeply nest slots** (makes maintenance hard)
- **Don't use too many slots** (3-5 is usually enough)

## Common Patterns

### Card with Slots

```js
JQRE.defineElement('info-card', {
  template: `
    <div class="card">
      <div class="card-header">
        <slot $:name="'title'"></slot>
      </div>
      <div class="card-body">
        <slot $:default></slot>
      </div>
      <div class="card-footer">
        <slot $:name="'actions'"></slot>
      </div>
    </div>
  `
});
```

### List with Item Slot

```js
JQRE.defineElement('item-list', {
  props: { items: [] },
  template: `
    <ul>
      <li $:for="item in items" $:key="item.id">
        <slot $:name="'item'" $:scope="{ item }">
          <span $:text="item.name"></span>
        </slot>
      </li>
    </ul>
  `
});
```

### Modal with Slots

```js
JQRE.defineElement('modal-dialog', {
  state: { open: false },
  template: `
    <div $:if="open" class="modal-backdrop">
      <div class="modal">
        <div class="modal-header">
          <slot $:name="'header'"></slot>
          <button $:click="open = false">×</button>
        </div>
        <div class="modal-body">
          <slot $:default></slot>
        </div>
        <div class="modal-footer">
          <slot $:name="'footer'">
            <button $:click="open = false">Close</button>
          </slot>
        </div>
      </div>
    </div>
  `
});
```

## Limitations

### Scoped Slots in Render Functions

`h.slot()` **does not support** scoped slots. Scoped slots only work with template-based components.

**Workaround:** Use props and events for render-based components:

```js
// ❌ Not supported
h.slot('item', [], { scope: { item } })

// ✅ Use props instead
h('item-renderer', {
  props: { item: () => item }
})
```

### Slot Detection

Cannot reliably detect if slot has content before rendering (Light DOM limitation).

### Slot Order

Slot content processes in DOM order, not slot placement order.

## Troubleshooting

### Slot Not Showing

**Problem:** Content doesn't appear in slot.

**Solutions:**
```html
<!-- ❌ Wrong: missing $:default or $:name -->
<slot></slot>

<!-- ✅ Correct: explicit directive -->
<slot $:default></slot>
<slot $:name="'header'"></slot>
```

### Scoped Slot Not Working

**Problem:** `$:scope` data not available.

**Solutions:**
- Ensure using template-based component (not render-based)
- Check `$:scope` attribute syntax
- Verify slot name matches

### h.slot() Error

**Problem:** `h.slot is not a function`.

**Solution:** Ensure JQR-H is loaded:
```html
<script src="/jqr.h.js"></script>  <!-- Required! -->
<script src="/jqr.e.js"></script>
```

## Next Steps

- **[Element API](./element-api.md)** - Access component instance
- **[Lifecycle](./lifecycle.md)** - Mounting and ownership
- **[h Integration](./h-integration.md)** - Using with render functions
- **[Dialog Example](../examples/dialog-component.md)** - Complete dialog with slots

## Related

- [JQR Slots Guide](../../jqr/guides/slots.md)
- [JQR-H Slots Guide](../../jqr-h/guides/slots-in-h.md)
- [Defining Elements](./defining-elements.md)
- [Events](./events.md)
