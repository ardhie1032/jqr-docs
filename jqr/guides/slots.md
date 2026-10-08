<!-- {
  "path": "jqr/guides/slots",
  "name": "docs-jqr-guides-slots",
  "meta": {
    "title": "Slots",
    "order": 11
  }
} -->

# Slots

Slots enable content projection - letting parent components pass custom content into child component templates. Think of slots as placeholders that can be filled with any content.

## Why Slots?

Slots make components flexible and reusable:

**Without slots:**
```html
<!-- Rigid button component -->
<button class="btn">Click Me</button>
```

**With slots:**
```html
<!-- Flexible button that accepts any content -->
<button class="btn">
  <slot></slot>  <!-- Can be text, icons, images, etc -->
</button>
```

**Benefits:**
- Reusable UI components
- Flexible content composition
- Clean component APIs
- Better separation of concerns

## Basic Slot

Define a default slot in your component template:

```html
<!-- Component template -->
<template id="card-template">
  <div class="card">
    <div class="card-body">
      <slot $:default></slot>
    </div>
  </div>
</template>
```

```html
<!-- Usage -->
<div $:component="card">
  <h3>Hello World</h3>
  <p>This content goes into the slot!</p>
</div>
```

**Result:**
```html
<div class="card">
  <div class="card-body">
    <h3>Hello World</h3>
    <p>This content goes into the slot!</p>
  </div>
</div>
```

## Slot Syntax

JQR uses explicit slot markers:

```html
<!-- Default slot -->
<slot $:default></slot>

<!-- Named slot -->
<slot $:name="header"></slot>
<slot $:name="footer"></slot>
```

**Note:** A plain `<slot>` without `$:default` or `$:name` is NOT treated as a slot outlet (by design, to avoid confusion with HTML5's native `<slot>` element).

## Default Slot Example

Register a component with a slot:

```html live:no-code
<div id="demo-slot-basic" class="en-card">
  <div class="en-card__body">
    <h3>Basic Slot</h3>
    <div class="en-alert en-alert--info">
      <strong>Attention!</strong> This is content passed through a slot.
    </div>
  </div>
</div>
```

```html
<template id="alert-template">
  <div class="alert">
    <slot $:default></slot>
  </div>
</template>
```

```html
<!-- Usage -->
<div $:component="custom-alert">
  <strong>Attention!</strong> This is an alert message.
</div>
```

```js inject show
// Note: In real usage, you'd register the component first
// For this demo, we're showing the concept without actual registration
```

```js cleanup
onUnmount(() => {
  // Cleanup if needed
})
```

## Named Slots

Use named slots for multiple content areas:

```html
<!-- Component template with named slots -->
<template id="card-with-header-template">
  <div class="card">
    <div class="card-header">
      <slot $:name="header"></slot>
    </div>
    <div class="card-body">
      <slot $:default></slot>
    </div>
    <div class="card-footer">
      <slot $:name="footer"></slot>
    </div>
  </div>
</template>
```

```html
<!-- Usage with $:slot directive -->
<div $:component="card-with-header">
  <div $:slot="header">
    <h3>Card Title</h3>
  </div>
  
  <p>This is the main content.</p>
  <p>It goes into the default slot.</p>
  
  <div $:slot="footer">
    <button>Action</button>
  </div>
</div>
```

**Result:**
```html
<div class="card">
  <div class="card-header">
    <h3>Card Title</h3>
  </div>
  <div class="card-body">
    <p>This is the main content.</p>
    <p>It goes into the default slot.</p>
  </div>
  <div class="card-footer">
    <button>Action</button>
  </div>
</div>
```

## Slot Assignment

**Parent side** - Use `$:slot` directive:
```html
<div $:slot="header">Header content</div>
<div $:slot="footer">Footer content</div>
<!-- No $:slot = default slot -->
<div>Default content</div>
```

**Child side** - Use `$:name` or `$:default`:
```html
<slot $:name="header"></slot>
<slot $:default></slot>
<slot $:name="footer"></slot>
```

## Fallback Content

Provide default content when slot is not filled:

```html
<template id="button-template">
  <button class="btn">
    <slot $:default>
      <!-- Fallback content -->
      Click Me
    </slot>
  </button>
</template>
```

**Usage:**
```html
<!-- Uses custom content -->
<div $:component="custom-btn">
  Submit Form
</div>

<!-- Uses fallback "Click Me" -->
<div $:component="custom-btn"></div>
```

## Complete Example: Modal

Modal component with header, body, and footer slots using native `<dialog>` element:

```html live:no-code
<template id="demo-modal-component">
  <dialog class="en-modal" $:model:open.on-close="isOpen">
    <header class="en-modal__header">
      <h3 class="en-modal__title"><slot $:name="header">Untitled</slot></h3>
      <button class="en-modal__close" $:click="close">&times;</button>
    </header>
    <div class="en-modal__body"><slot $:default>Hello</slot></div>
    <footer class="en-modal__footer"><slot $:name="footer"></slot></footer>
  </dialog>
</template>

<div id="demo-modal-with-slot">
  <div $:component="demo-modal-component" $:model:open="showModal">
    <p $:text="message"></p>
    <template $:slot="header">Confirmation</template>
    <template $:slot="footer">
      <button class="en-btn en-btn--sm" $:click="showModal = false">Close</button>
    </template>
  </div>
  <button class="en-btn" $:click="showModal = true">Show Modal</button>
</div>
```

```html
<template id="demo-modal-component">
  <dialog $:model:open.on-close="isOpen">
    <header>
      <h3><slot $:name="header">Untitled</slot></h3>
      <button $:click="close">&times;</button>
    </header>
    <div><slot $:default>Hello</slot></div>
    <footer>
      <slot $:name="footer"></slot>
    </footer>
  </dialog>
</template>

<div id="demo-modal-with-slot">
  <div $:component="demo-modal-component" $:model:open="showModal">
    <template $:slot="header">Confirmation</template>
    <p $:text="message"></p>
    <template $:slot="footer">
      <button $:click="showModal = false">Close</button>
    </template>
  </div>
  <button $:click="showModal = true">Show Modal</button>
</div>
```

```js inject show
$('#demo-modal-with-slot').component({
  components: {
    'demo-modal-component': {
      props: { open: false },
      emits: ['update:open'],
      state: (props) => ({ isOpen: props.open }),
      created() {
        this.watch(() => this.props.open, (val) => {
          this.state.isOpen = val
        })
      },
      methods: {
        close() {
          this.emit('update:open', false)
        }
      }
    }
  },
  state: { message: 'Are you sure?', showModal: false }
})
```

```js cleanup
onUnmount(() => {
  $('#demo-modal-with-slot').component('destroy');
})
```

## Slot Content Scope

**Important:** Slot content has access to **parent's** scope, not child's:

```html
<!-- Parent component -->
<div $:component="user-card">
  <!-- This has access to parent's 'userName' -->
  <p>Hello, <span $:text="userName"></span>!</p>
</div>
```

```html
<!-- Child template -->
<template id="user-card-template">
  <div class="card">
    <!-- This has access to child's state -->
    <h3 $:text="title"></h3>
    
    <!-- Slot content uses parent's scope -->
    <slot $:default></slot>
  </div>
</template>
```

**Rule:** Slot content sees parent data, template sees child data.

## Multiple Default Content

Only one piece of content without `$:slot` becomes the default slot:

```html
<div $:component="card">
  <!-- This goes to default slot -->
  <p>First paragraph</p>
  <p>Second paragraph</p>
  
  <!-- Named slots -->
  <div $:slot="header">Title</div>
  <div $:slot="footer">Actions</div>
</div>
```

## Dynamic Slots

Slot content can be fully dynamic:

```html live:no-code
<div id="demo-slot-dynamic" class="en-card">
  <div class="en-card__body">
    <h3>Dynamic Slot Content</h3>
    <div class="en-flex en-gap-2" style="margin-bottom: 1rem;">
      <button class="en-btn en-btn--sm en-btn--primary" $:click="contentType = 'text'">Text</button>
      <button class="en-btn en-btn--sm en-btn--primary" $:click="contentType = 'list'">List</button>
      <button class="en-btn en-btn--sm en-btn--primary" $:click="contentType = 'form'">Form</button>
    </div>
    
    <div class="en-alert en-alert--info">
      <div $:if="contentType === 'text'">
        <p style="margin: 0;">This is text content that changes dynamically.</p>
      </div>
      
      <div $:if="contentType === 'list'">
        <ul style="margin: 0; padding-left: 1.5rem;">
          <li>Item 1</li>
          <li>Item 2</li>
          <li>Item 3</li>
        </ul>
      </div>
      
      <div $:if="contentType === 'form'">
        <input class="en-input" placeholder="Enter text..." style="margin-bottom: 0.5rem;">
        <button class="en-btn en-btn--sm en-btn--primary">Submit</button>
      </div>
    </div>
  </div>
</div>
```

```html
<div id="demo-slot-dynamic">
  <button $:click="contentType = 'text'">Text</button>
  <button $:click="contentType = 'list'">List</button>
  <button $:click="contentType = 'form'">Form</button>
  
  <!-- Dynamic content with $:if -->
  <div $:if="contentType === 'text'">
    <p>Text content</p>
  </div>
  
  <div $:if="contentType === 'list'">
    <ul>
      <li $:for="item in items" $:text="item"></li>
    </ul>
  </div>
  
  <div $:if="contentType === 'form'">
    <input $:model="inputValue">
    <button $:click="submit">Submit</button>
  </div>
</div>
```

```js inject show
$('#demo-slot-dynamic').component({
  state: { contentType: 'text' }
});
```

```js cleanup
onUnmount(() => {
  $('#demo-slot-dynamic').component('destroy');
})
```

## Advanced Scoped Slot Patterns

### Conditional Rendering with Scope

```html
<template id="status-card-template">
  <div class="card">
    <slot $:name="content" 
          $:scope:is-active="status === 'active'"
          $:scope:status="status"></slot>
  </div>
</template>
```

```html
<div $:component="status-card">
  <template $:slot="content" $:scope="{ isActive, status }">
    <div $:class="{ 'text-success': isActive, 'text-muted': !isActive }">
      Status: <span $:text="status"></span>
    </div>
  </template>
</div>
```

### Computed Values in Scope

```html
<template id="price-card-template">
  <slot $:name="details"
        $:scope:price="basePrice"
        $:scope:discounted="basePrice * 0.8"
        $:scope:savings="basePrice * 0.2"></slot>
</template>
```

### Nested Scoped Slots

```html
<template id="category-list-template">
  <div $:for="category in categories" $:key="category.id">
    <slot $:name="category" $:scope:category="category">
      <!-- Nested iteration -->
      <div $:for="item in category.items" $:key="item.id">
        <slot $:name="item" $:scope:item="item" $:scope:category="category"></slot>
      </div>
    </slot>
  </div>
</template>
```

## Card Component Example

Reusable card with flexible content:

```html live:no-code
<div id="demo-slot-card" class="en-card">
  <div class="en-card__body">
    <h3>Flexible Card Component</h3>
    <p class="en-text--muted">This demonstrates how slots allow flexible content</p>
    
    <div class="en-card" style="margin-top: 1rem;">
      <div class="en-card__header">
        <h4 style="margin: 0;">User Profile</h4>
      </div>
      <div class="en-card__body">
        <p style="margin: 0.5rem 0;"><strong>Name:</strong> John Doe</p>
        <p style="margin: 0.5rem 0;"><strong>Email:</strong> john@example.com</p>
      </div>
      <div class="en-card__footer" style="display: flex; gap: 0.5rem;">
        <button class="en-btn en-btn--sm en-btn--primary">Edit</button>
        <button class="en-btn en-btn--sm en-btn--danger">Delete</button>
      </div>
    </div>
    
    <div class="en-card" style="margin-top: 1rem;">
      <div class="en-card__body">
        <p style="margin: 0;">Simple content without header/footer</p>
      </div>
    </div>
  </div>
</div>
```

```html
<template id="flexible-card-template">
  <div class="card">
    <div class="card-header" $:if="hasHeader">
      <slot $:name="header"></slot>
    </div>
    <div class="card-body">
      <slot $:default>
        <p>No content provided</p>
      </slot>
    </div>
    <div class="card-footer" $:if="hasFooter">
      <slot $:name="footer"></slot>
    </div>
  </div>
</template>
```

```html
<!-- Usage 1: Full card -->
<div $:component="flexible-card">
  <div $:slot="header">
    <h3>User Profile</h3>
  </div>
  
  <p>Name: John Doe</p>
  <p>Email: john@example.com</p>
  
  <div $:slot="footer">
    <button>Edit</button>
    <button>Delete</button>
  </div>
</div>

<!-- Usage 2: Body only -->
<div $:component="flexible-card">
  <p>Simple content without header/footer</p>
</div>
```

```js inject show
$('#demo-slot-card').component({
  state: {}
});
```

```js cleanup
onUnmount(() => {
  $('#demo-slot-card').component('destroy');
})
```

## Tab Component Pattern

Tabs with slot-based content:

```html
<template id="tabs-template">
  <div class="tabs">
    <div class="tabs-header">
      <button 
        $:for="tab in tabs" 
        $:key="tab"
        $:click="activeTab = tab"
        $:class="{ active: activeTab === tab }"
        $:text="tab">
      </button>
    </div>
    <div class="tabs-body">
      <slot $:name="content"></slot>
    </div>
  </div>
</template>
```

```html
<div $:component="tabs" $:prop:tabs="['Home', 'Profile', 'Settings']">
  <div $:slot="content">
    <div $:if="activeTab === 'Home'">Home content</div>
    <div $:if="activeTab === 'Profile'">Profile content</div>
    <div $:if="activeTab === 'Settings'">Settings content</div>
  </div>
</div>
```

## List Component Pattern

List with customizable item rendering using scoped slots:

```html
<template id="user-list-template">
  <div class="user-list">
    <div $:for="(user, index) in users" $:key="user.id" class="user-item">
      <!-- Expose user and index through scoped slot -->
      <slot $:name="item" $:scope:user="user" $:scope:index="index"></slot>
    </div>
  </div>
</template>
```

```html
<div $:component="user-list" $:prop:users="userList">
  <template $:slot="item" $:scope="{ user, index }">
    <!-- Parent controls rendering, child provides data -->
    <img $:attr:src="user.avatar">
    <div>
      <h4><span $:text="index + 1"></span>. <span $:text="user.name"></span></h4>
      <p $:text="user.email"></p>
    </div>
  </template>
</div>
```

## Best Practices

### 1. Always Mark Slots Explicitly

```html
<!-- ✅ Good - explicit markers -->
<slot $:default></slot>
<slot $:name="header"></slot>
<slot $:name="item" $:scope:data="itemData"></slot>

<!-- ❌ Bad - plain <slot> not recognized -->
<slot></slot>
```

### 2. Use Scoped Slots for Data-Driven Components

```html
<!-- ✅ Good - child manages data, parent controls presentation -->
<template id="list-template">
  <div $:for="item in items">
    <slot $:name="item" $:scope:item="item"></slot>
  </div>
</template>

<!-- ❌ Less flexible - parent can't customize rendering -->
<template id="list-template">
  <div $:for="item in items">
    <span $:text="item.name"></span>
  </div>
</template>
```

### 3. Use Inline Components for Simple Cases

```html
<!-- ✅ Good - simple, one-off component -->
<div $:component:state="{ count: 0 }">
  <button $:click="count++">Clicked <span $:text="count"></span> times</button>
</div>

<!-- ❌ Overkill - don't register if used once -->
<script>
  $.jqr.register('simple-counter', { state: { count: 0 } });
</script>
<div $:component="simple-counter">
  <button $:click="count++">Clicked <span $:text="count"></span> times</button>
</div>
```

### 4. Use `$:component:slot` for Simple Scoped Slots

```html
<!-- ✅ Good - clean shorthand for default slot -->
<div $:component="user-card" $:component:slot="{ user }">
  <h3 $:text="user.name"></h3>
</div>

<!-- ⚠️ More verbose - use when you need named slots -->
<div $:component="user-card">
  <template $:scope="{ user }">
    <h3 $:text="user.name"></h3>
  </template>
</div>
```

### 5. Check Slot Presence with `$slots`

```html
<!-- ✅ Good - avoid empty wrappers -->
<template $:if="$slots['header']">
  <div class="header-wrapper">
    <slot $:name="header"></slot>
  </div>
</template>

<!-- ❌ Bad - always renders wrapper even when empty -->
<div class="header-wrapper">
  <slot $:name="header"></slot>
</div>
```

### 6. Provide Fallback Content

```html
<!-- ✅ Good - has fallback -->
<slot $:default>
  <p>Default message</p>
</slot>

<!-- ⚠️ Okay but fragile - no fallback -->
<slot $:default></slot>
```

### 7. Name Slots and Scope Variables Semantically

```html
<!-- ✅ Good - clear purpose and names -->
<slot $:name="header"></slot>
<slot $:name="item" $:scope:user="currentUser" $:scope:index="position"></slot>

<!-- ❌ Confusing - generic names -->
<slot $:name="slot1" $:scope:data="x"></slot>
```

**Naming Convention:**
- Use **kebab-case** in HTML attributes: `$:scope:user-name`, `$:scope:item-count`
- Automatically converts to **camelCase** in JavaScript: `userName`, `itemCount`

**Example:**
```html
<!-- Child -->
<slot $:name="details" 
      $:scope:user-name="user.name" 
      $:scope:is-active="user.active"></slot>

<!-- Parent -->
<template $:slot="details" $:scope="{ userName, isActive }">
  <p><span $:text="userName"></span> - <span $:text="isActive ? 'Active' : 'Inactive'"></span></p>
</template>
```

### 6. Document Slot API

```html
<!-- Document what slots are available -->
<template id="card-template">
  <!--
    Slots:
    - header: Card title/header content
    - default: Main card body content  
    - footer: Action buttons/footer content
  -->
  <div class="card">
    <div class="card-header">
      <slot $:name="header"></slot>
    </div>
    <div class="card-body">
      <slot $:default></slot>
    </div>
    <div class="card-footer">
      <slot $:name="footer"></slot>
    </div>
  </div>
</template>
```

### 7. Keep Slot Content Simple

```html
<!-- ✅ Good - simple slot content -->
<div $:slot="header">
  <h3>Title</h3>
</div>

<!-- ⚠️ Avoid - complex nested logic in slots -->
<div $:slot="header">
  <div $:for="item in complexData">
    <div $:if="item.condition">
      <!-- Too complex -->
    </div>
  </div>
</div>
```

## Common Patterns

### Layout Components

```html
<template id="page-layout-template">
  <div class="page-layout">
    <header>
      <slot $:name="header"></slot>
    </header>
    <aside>
      <slot $:name="sidebar"></slot>
    </aside>
    <main>
      <slot $:default></slot>
    </main>
    <footer>
      <slot $:name="footer"></slot>
    </footer>
  </div>
</template>
```

### Wrapper Components

```html
<template id="error-boundary-template">
  <div $:if="!hasError">
    <slot $:default></slot>
  </div>
  <div $:if="hasError" class="error-message">
    <slot $:name="error">
      <p>Something went wrong</p>
    </slot>
  </div>
</template>
```

### Conditional Wrapper

```html
<template id="conditional-link-template">
  <a $:if="href" $:attr:href="href">
    <slot $:default></slot>
  </a>
  <span $:if="!href">
    <slot $:default></slot>
  </span>
</template>
```

## Debugging Slots

**Check if slot is filled:**
```js
// Slots are filled at render time
// Check DOM after component mounts
mounted(state, $el) {
  const hasContent = $el.find('[data-slot-filled]').length > 0;
  console.log('Slot filled:', hasContent);
}
```

**Debug scoped slots:**
```html
<!-- Add debug output in parent -->
<template $:slot="item" $:scope="{ item, index }">
  <div>
    <pre $:text="JSON.stringify({ item, index }, null, 2)"></pre>
  </div>
</template>
```

**Common issues:**
- Forgetting `$:default` or `$:name` markers
- Typo in slot name: `$:slot="header"` vs `$:name="heading"`
- Slot content accessing child scope instead of parent scope
- Scoped slot pattern mismatch: `$:scope="{ user }"` but child provides `$:scope:item`
- Inline component syntax errors: missing quotes, invalid JSON

**Dev warnings:**
- Plain `<slot>` without marker: Console warning
- Unused slot content: Warning about slot with no outlet
- Scoped slot parameter mismatch: Warning when parent expects different variables than child provides
- Invalid `$:scope` destructuring pattern: Warning with expected format

## Performance

**Slots are efficient:**
- Content moves directly into outlets (no cloning)
- Reactive bindings preserved
- No extra overhead vs inline content

**Cost:** Negligible. Slot filling happens once at mount.

## Scoped Slots

Scoped slots allow child components to pass data up to parent slot content - useful for list rendering, data tables, and any pattern where the child manages data but the parent controls presentation.

**Important:** HTML attribute names are case-insensitive. Always use **kebab-case** for multi-word scope variable names in HTML (`$:scope:user-name`), which automatically converts to **camelCase** in JavaScript (`userName`).

### Basic Scoped Slot

**Child component** exposes data through `$:scope:*` attributes:

```html
<template id="user-card-template">
  <div class="card">
    <!-- Expose 'user' data to parent -->
    <slot $:name="header" $:scope:user="currentUser"></slot>
    <div class="card-body">
      <slot $:default $:scope:user="currentUser"></slot>
    </div>
  </div>
</template>
```

**Parent component** receives data with `$:scope` pattern:

```html live:no-code
<div id="demo-scoped-slot" class="en-card">
  <div class="en-card__body">
    <h3>Scoped Slot Example</h3>
    <div class="en-card" style="margin-top: 1rem;">
      <div class="en-card__header">
        <h4 style="margin: 0;">User: <span style="color: var(--en-primary);">John Doe</span></h4>
      </div>
      <div class="en-card__body">
        <p style="margin: 0.5rem 0;"><strong>Email:</strong> john@example.com</p>
        <p style="margin: 0.5rem 0;"><strong>Role:</strong> Admin</p>
      </div>
    </div>
  </div>
</div>
```

```html
<div $:component="user-card">
  <template $:slot="header" $:scope="{ user }">
    <h3>User: <span $:text="user.name"></span></h3>
  </template>
  
  <template $:scope="{ user }">
    <p>Email: <span $:text="user.email"></span></p>
    <p>Role: <span $:text="user.role"></span></p>
  </template>
</div>
```

```js inject show
$('#demo-scoped-slot').component({
  state: {}
});
```

```js cleanup
onUnmount(() => {
  $('#demo-scoped-slot').component('destroy');
})
```

### List with Scoped Slots

Common pattern: child manages data iteration, parent controls item rendering:

```html live:no-code
<div id="demo-scoped-list" class="en-card">
  <div class="en-card__body">
    <h3>User List with Scoped Slots</h3>
    <div style="margin-top: 1rem;">
      <div style="padding: 0.5rem; border-bottom: 1px solid var(--en-border); display: flex; align-items: center; gap: 0.75rem;" $:for="(user, index) in users" $:key="user.id">
        <div style="width: 32px; height: 32px; border-radius: 50%; background: var(--en-primary); color: white; display: flex; align-items: center; justify-content: center; font-weight: bold;" $:text="index + 1"></div>
        <img $:attr:src="user.avatar" $:attr:alt="user.name" style="width: 40px; height: 40px; border-radius: 50%;">
        <div style="flex: 1;">
          <div style="font-weight: 600;" $:text="user.name"></div>
          <div style="font-size: 0.875rem; color: var(--en-text-muted);" $:text="user.email"></div>
        </div>
        <span class="en-badge en-badge--sm" $:class="{ 'en-badge--success': user.status === 'active', 'en-badge--secondary': user.status !== 'active' }" $:text="user.status"></span>
      </div>
    </div>
  </div>
</div>
```

```html
<template id="user-list-template">
  <div class="user-list">
    <div $:for="(user, index) in users" $:key="user.id" class="user-item">
      <!-- Expose both user and index to parent -->
      <slot $:name="item" $:scope:user="user" $:scope:index="index"></slot>
    </div>
  </div>
</template>
```

```html
<div $:component="user-list" $:prop:users="userList">
  <template $:slot="item" $:scope="{ user, index }">
    <div class="user-row">
      <span $:text="index + 1"></span>.
      <img $:attr:src="user.avatar" $:attr:alt="user.name">
      <div>
        <strong $:text="user.name"></strong>
        <small $:text="user.email"></small>
      </div>
      <span $:text="user.status"></span>
    </div>
  </template>
</div>
```

```js inject show
$('#demo-scoped-list').component({
  state: {
    users: [
      { id: 1, name: 'Alice Johnson', email: 'alice@example.com', status: 'active', avatar: 'https://i.pravatar.cc/150?img=1' },
      { id: 2, name: 'Bob Smith', email: 'bob@example.com', status: 'active', avatar: 'https://i.pravatar.cc/150?img=2' },
      { id: 3, name: 'Carol White', email: 'carol@example.com', status: 'inactive', avatar: 'https://i.pravatar.cc/150?img=3' }
    ]
  }
});
```

```js cleanup
onUnmount(() => {
  $('#demo-scoped-list').component('destroy');
})
```

### Variable Renaming

Rename scoped variables using object destructuring syntax:

```html
<!-- Child provides 'items' -->
<slot $:name="content" $:scope:items="productList"></slot>

<!-- Parent renames to 'products' -->
<template $:slot="content" $:scope="{ items: products }">
  <div $:for="product in products" $:text="product.name"></div>
</template>
```

### Multiple Scope Variables

Pass multiple variables through one slot:

```html
<!-- Child side -->
<slot $:name="row" 
      $:scope:item="currentItem" 
      $:scope:index="$index"
      $:scope:is-even="$index % 2 === 0"></slot>

<!-- Parent side -->
<template $:slot="row" $:scope="{ item, index, isEven }">
  <div $:class="{ even: isEven }">
    <span $:text="index"></span>: <span $:text="item.name"></span>
  </div>
</template>
```

### Data Table Pattern

Perfect use case for scoped slots:

```html
<template id="data-table-template">
  <table>
    <thead>
      <slot $:name="header"></slot>
    </thead>
    <tbody>
      <tr $:for="(row, index) in rows" $:key="index">
        <slot $:name="row" $:scope:row="row" $:scope:index="index"></slot>
      </tr>
    </tbody>
  </table>
</template>
```

```html
<div $:component="data-table" $:prop:rows="products">
  <template $:slot="header">
    <tr>
      <th>#</th>
      <th>Name</th>
      <th>Price</th>
      <th>Stock</th>
    </tr>
  </template>
  
  <template $:slot="row" $:scope="{ row, index }">
    <td $:text="index + 1"></td>
    <td $:text="row.name"></td>
    <td $:text="'$' + row.price"></td>
    <td $:text="row.stock"></td>
  </template>
</div>
```

### Scoped Slot with Default Slot

Default slot can also be scoped:

```html
<!-- Child -->
<template id="container-template">
  <div class="container">
    <!-- Default slot with scope -->
    <slot $:default $:scope:data="internalData"></slot>
  </div>
</template>

<!-- Parent -->
<div $:component="container">
  <template $:scope="{ data }">
    <p>Data from child: <span $:text="data.value"></span></p>
  </template>
</div>
```

### Scoped Slots in Loops

**New Feature:** Scoped slots now work inside `$:for` loops - scope values are evaluated per iteration!

```html live:no-code
<div id="demo-scoped-in-loop" class="en-card">
  <div class="en-card__body">
    <h3>Scoped Slot per Iteration</h3>
    <div>
      <div $:for="msg in messages" $:key="msg.id" class="en-flex en-gap-2 en-items-center en-mb-2">
        <div class="en-flex-1 en-form-display" $:text="msg.text"></div>
        <button class="en-btn en-btn--sm en-btn--danger" $:click="remove(msg.id)">Remove</button>
      </div>
    </div>
  </div>
</div>
```

```html
<!-- Child template with scoped slot inside $:for -->
<template id="message-list-template">
  <div $:for="msg in filteredMessages" $:key="msg.id">
    <slot $:name="message" $:scope:item="msg" $:scope:remove="remove">
      <!-- Fallback content -->
      <div>
        <div $:text="msg.text"></div>
        <button $:click="remove(msg.id)">Delete</button>
      </div>
    </slot>
  </div>
</template>

<!-- Parent usage -->
<div $:component="message-list">
  <template $:slot="message" $:scope="{ item, remove }">
    <span $:text="item.text"></span>
    <button $:click="remove">×</button>
  </template>
</div>
```

```js inject show
$('#demo-scoped-in-loop').component({
  state: {
    messages: [
      { id: 1, text: 'Hello World' },
      { id: 2, text: 'JQR is awesome' },
      { id: 3, text: 'Scoped slots work!' }
    ]
  },
  methods: {
    remove(id) {
      this.state.messages = this.state.messages.filter(m => m.id !== id);
    }
  }
});
```

```js cleanup
onUnmount(() => {
  $('#demo-scoped-in-loop').component('destroy');
})
```

**Key benefits:**
- Each iteration gets its own scope values
- `msg` (loop variable) is accessible in `$:scope:item="msg"`
- Methods like `remove` can be passed per item
- Perfect for lists with customizable item rendering

### Single Variable Syntax

**New Feature:** Use a single variable instead of destructuring:

```html
<!-- Destructuring (existing) -->
<template $:slot="header" $:scope="{ items, count }">
  <div $:text="items.length"></div>
</template>

<!-- Single variable (new) -->
<template $:slot="header" $:scope="scoped">
  <div $:text="scoped.items.length"></div>
  <div $:text="scoped.count"></div>
</template>
```

**When to use:**
- Many scope variables (avoid long destructuring)
- Unknown scope structure
- Prefer object access over individual variables

### Shorthand: `$:component:slot`

**New Feature:** Define default scoped slot directly on component element (like Vue's `v-slot`):

```html live:no-code
<div id="demo-component-slot" class="en-card">
  <div class="en-card__body">
    <h3>Component Slot Shorthand</h3>
    <div class="en-alert en-alert--info">
      <p style="margin: 0.5rem 0;"><strong>Username:</strong> <span>JohnDoe</span></p>
      <p style="margin: 0.5rem 0;"><strong>Role:</strong> <span>Admin</span></p>
    </div>
  </div>
</div>
```

```html
<!-- Without template wrapper - destructuring -->
<div $:component="user-card" $:component:slot="{ username, role }">
  <p><strong>Username:</strong> <span $:text="username"></span></p>
  <p><strong>Role:</strong> <span $:text="role"></span></p>
</div>

<!-- Without template wrapper - single variable -->
<div $:component="user-card" $:component:slot="scoped">
  <p><strong>Username:</strong> <span $:text="scoped.username"></span></p>
  <p><strong>Role:</strong> <span $:text="scoped.role"></span></p>
</div>
```

```js inject show
$('#demo-component-slot').component({
  components: {
    'user-card': {
      state: { username: 'JohnDoe', role: 'Admin' }
    }
  }
});
```

```js cleanup
onUnmount(() => {
  $('#demo-component-slot').component('destroy');
})
```

**Syntax:**
- `$:component:slot="{ var1, var2 }"` - Destructuring
- `$:component:slot="scoped"` - Single variable

**Important:**
- Only works for **default slot**
- Cannot mix with named slots (`$:slot="name"`)
- All children become part of the scoped slot content

**Comparison with template syntax:**
```html
<!-- Traditional (still works) -->
<div $:component="user-card">
  <template $:scope="{ username, role }">
    <p $:text="username"></p>
  </template>
</div>

<!-- Shorthand (new) -->
<div $:component="user-card" $:component:slot="{ username, role }">
  <p $:text="username"></p>
</div>
```

## Checking Slot Presence: `$slots`

**New Feature:** Check if a slot was filled by the parent using `$slots`:

```html live:no-code
<div id="demo-slots-check" class="en-card">
  <div class="en-card__body">
    <h3>Conditional Slots</h3>
    <div class="en-card">
      <div class="en-card__header">
        <h4 style="margin: 0;">Card with Header</h4>
      </div>
      <div class="en-card__body">
        <p style="margin: 0;">This card has a header slot filled.</p>
      </div>
    </div>
  </div>
</div>
```

```html
<!-- Child template - conditionally render slot container -->
<template id="smart-card-template">
  <div class="card">
    <template $:if="$slots['header']">
      <div class="card-header">
        <slot $:name="header"></slot>
      </div>
    </template>
    
    <div class="card-body">
      <slot $:default></slot>
    </div>
    
    <template $:if="$slots['footer']">
      <div class="card-footer">
        <slot $:name="footer"></slot>
      </div>
    </template>
  </div>
</template>

<!-- Usage - only header provided -->
<div $:component="smart-card">
  <div $:slot="header">
    <h3>Card Title</h3>
  </div>
  <p>Card content without footer.</p>
</div>
```

```js inject show
$('#demo-slots-check').component({
  components: {
    'smart-card': {
      template: `
        <div class="card">
          <div class="card-header">
            <slot $:name="header"></slot>
          </div>
          <div class="card-body">
            <slot $:default></slot>
          </div>
        </div>
      `
    }
  }
});
```

```js cleanup
onUnmount(() => {
  $('#demo-slots-check').component('destroy');
})
```

**Syntax:**
```js
$slots['slotName']  // true if filled, undefined if not
$slots.default      // can also use dot notation
```

**Use cases:**
- Show/hide slot wrappers (headers, footers)
- Conditional styling based on slot presence
- Avoid rendering empty containers
- Different layouts based on which slots are filled

**Example with multiple checks:**
```html
<template $:if="$slots['header'] && $slots['footer']">
  <div class="full-card">
    <slot $:name="header"></slot>
    <slot $:default></slot>
    <slot $:name="footer"></slot>
  </div>
</template>
<template $:if="!$slots['header'] && !$slots['footer']">
  <div class="simple-card">
    <slot $:default></slot>
  </div>
</template>
```

## Inline Component Definition

Define components directly in HTML without JavaScript registration!

### Basic Inline Component

Use `$:component:is` with an inline object to create components on the fly:

```html live:no-code
<div id="demo-inline-component" class="en-card">
  <div class="en-card__body">
    <h3>Inline Component</h3>
    <div class="en-alert en-alert--info en-mb-3">
      <p style="margin: 0.5rem 0;"><strong $:text="say"></strong></p>
    </div>
    <label class="en-field">
      <input class="en-input mb-3" $:model.debounce.500="say">
    </label>
  </div>
</div>
```

```html
<!-- Method 1: Full inline object with $:component:is -->
<div $:component:is="{ state: { say: 'Hello World!' } }">
  <p $:text="say"></p>
  <input $:model.debounce.500="say">
</div>
```

```js inject show
$('#demo-inline-component').component({
  state: { say: 'Hello World!' }
});
```

```js cleanup
onUnmount(() => {
  $('#demo-inline-component').component('destroy');
})
```

### Shorthand Syntax

Use `$:component:<key>` attributes for cleaner inline definitions:

```html live:no-code
<div id="demo-inline-shorthand" class="en-card">
  <div class="en-card__body">
    <h3>Inline Component (Shorthand)</h3>
    <div class="en-alert en-alert--success en-mb-3">
      <p style="margin: 0.5rem 0;"><span $:text="message"></span></p>
    </div>
    <label class="en-field">
      <input class="en-input" $:model="message">      
    </label>
  </div>
</div>
```

```html
<!-- Method 2: Shorthand attributes -->
<div $:component:state="{ say: 'Hello World!' }">
  <p $:text="say"></p>
  <input $:model="say">
</div>
```

**Available shorthands:**
- `$:component:state` → `state`
- `$:component:computed` → `computed`
- `$:component:methods` → `methods`
- `$:component:async-state` → `asyncState`
- `$:component:constants` → `constants`
- `$:component:props` → `props`

```js inject show
$('#demo-inline-shorthand').component({
  state: { message: 'Hello from shorthand!' }
});
```

```js cleanup
onUnmount(() => {
  $('#demo-inline-shorthand').component('destroy');
})
```

### Complex Inline Component

Combine multiple options:

```html live:no-code
<div id="demo-inline-complex" class="en-card">
  <div class="en-card__body">
    <h3>Complex Inline Component</h3>
    <div class="en-alert en-alert--info">
      <p style="margin: 0.5rem 0;">Count: <strong $:text="count"></strong></p>
      <p style="margin: 0.5rem 0;">Double: <strong $:text="double"></strong></p>
      <div class="en-flex en-gap-2" style="margin-top: 0.5rem;">
        <button class="en-btn en-btn--sm en-btn--primary" $:click="increment">+1</button>
        <button class="en-btn en-btn--sm en-btn--outline" $:click="count = 0">Reset</button>
      </div>
    </div>
  </div>
</div>
```

```html
<div $:component:state="{ count: 0 }"
     $:component:computed="{ double: (state) => state.count * 2 }"
     $:component:methods="{ increment: (state) => state.count++ }">
  <p>Count: <strong $:text="count"></strong></p>
  <p>Double: <strong $:text="double"></strong></p>
  <button $:click="increment">+1</button>
  <button $:click="count = 0">Reset</button>
</div>
```

```js inject show
$('#demo-inline-complex').component({
  state: { count: 0 },
  computed: {
    double: (state) => state.count * 2
  },
  methods: {
    increment: (state) => state.count++
  }
});
```

```js cleanup
onUnmount(() => {
  $('#demo-inline-complex').component('destroy');
})
```

### Inline with Async State

Fetch data without separate JavaScript:

```html live:no-code
<div id="demo-inline-async" class="en-card">
  <div class="en-card__body">
    <h3>Inline Component with Async Data</h3>
    <div $:show="usersLoading" class="en-alert en-alert--info">Loading users...</div>
    <div $:show="!usersLoading && users.length > 0" style="margin-top: 1rem;">
      <div $:for="user in users?.slice(0, 3)" $:key="user.id" style="padding: 0.5rem; border-bottom: 1px solid var(--en-border); display: flex; align-items: center; gap: 0.75rem;">
        <img $:attr:src="user.avatar" $:attr:alt="user.name" style="width: 40px; height: 40px; border-radius: 50%;">
        <div>
          <div style="font-weight: 600;" $:text="user.name"></div>
          <div style="font-size: 0.875rem; color: var(--en-text-muted);" $:text="user.email"></div>
        </div>
      </div>
    </div>
  </div>
</div>
```

```html
<div $:component:async-state="{
  users: async () => {
    const res = await fetch('/assets/api/jqr-docs/users.json');
    return res.json();
  }
}">
  <div $:show="usersLoading">Loading...</div>
  <div $:show="!usersLoading">
    <div $:for="user in users" $:key="user.id">
      <img $:attr:src="user.avatar">
      <span $:text="user.name"></span>
    </div>
  </div>
</div>
```

```js inject show
$('#demo-inline-async').component({
  asyncState: {
    users: async () => {
      const res = await fetch('/assets/api/jqr-docs/users.json');
      return res.json();
    }
  }
});
```

```js cleanup
onUnmount(() => {
  $('#demo-inline-async').component('destroy');
})
```

### When to Use Inline Components

**✅ Good for:**
- Quick prototypes
- One-off components
- Simple stateful widgets
- Self-contained demos

**❌ Not ideal for:**
- Reusable components (use `$.jqr.register()`)
- Complex logic (too verbose in HTML)
- Components used multiple times
- Components needing templates

**Rule of thumb:** If you're copying the same `$:component` definition twice, register it instead.

## Limitations

### Scoped Slots

JQR supports scoped slots for passing data from child to slot content!

Scoped slots let the child component expose data to the parent's slot content:

**Child side** - Define slot with scope values:
```html
<template id="user-list-template">
  <div class="user-list">
    <div $:for="user in users" $:key="user.id">
      <!-- Expose 'user' and 'index' to parent -->
      <slot $:name="item" $:scope:user="user" $:scope:index="$index"></slot>
    </div>
  </div>
</template>
```

**Parent side** - Receive scope values with `$:scope`:
```html
<div $:component="user-list">
  <template $:slot="item" $:scope="{ user, index }">
    <!-- Can access 'user' and 'index' from child -->
    <div>
      <span $:text="index + 1"></span>.
      <strong $:text="user.name"></strong>
      <span $:text="user.email"></span>
    </div>
  </template>
</div>
```

**Syntax:**

Child outlet:
```html
<slot $:name="slot-name" $:scope:variable-name="expression"></slot>
```

Parent content:
```html
<template $:slot="slot-name" $:scope="{ variableName }">
  <!-- Use variableName here (camelCase in JS) -->
</template>
```

**Important:** HTML attributes are case-insensitive, so use **kebab-case** in HTML attributes (`variable-name`), which automatically converts to **camelCase** in JavaScript scope (`variableName`).

**Renaming variables:**
```html
<!-- Child provides 'items' -->
<slot $:name="content" $:scope:items="data"></slot>

<!-- Parent renames to 'list' -->
<template $:slot="content" $:scope="{ items: list }">
  <div $:for="item in list">...</div>
</template>
```

### Single Default Slot

All content without `$:slot` goes to one default slot:

```html
<div $:component="card">
  <p>Paragraph 1</p>  <!-- Both go to -->
  <p>Paragraph 2</p>  <!-- default slot -->
</div>
```

## Invalid Syntax - Common Mistakes

Understanding what **doesn't** work helps avoid frustrating bugs. Here are common mistakes and why they fail:

### ❌ Bare `<slot>` Without Markers

```html
<!-- ❌ WRONG - plain <slot> not recognized -->
<template id="card-template">
  <div class="card">
    <slot></slot>  <!-- Ignored! Treated as regular HTML element -->
  </div>
</template>
```

**Why it fails:** By design, JQR requires explicit `$:default`, `$:name`, or `$:scope:*` markers. This avoids conflicts with HTML5's native `<slot>` element.

**Fix:**
```html
<!-- ✅ CORRECT -->
<slot $:default></slot>
```

**Dev warning:** Console warning "unrecognized slot" appears in dev mode.

---

### ❌ Invalid Slot Name Syntax

```html
<!-- ❌ WRONG - bare directive as slot name -->
<slot $:footer></slot>

<!-- ❌ WRONG - value on $:default -->
<slot $:default="items"></slot>
```

**Why it fails:**
- `$:footer` is not recognized (only `$:name="footer"` works)
- `$:default` doesn't accept values (use `$:scope:*` for data passing)

**Fix:**
```html
<!-- ✅ CORRECT - named slot -->
<slot $:name="footer"></slot>

<!-- ✅ CORRECT - scoped default slot -->
<slot $:default $:scope:items="itemList"></slot>
```

**Dev warnings:**
- `$:footer`: Two warnings ("unrecognized directive" + "no valid slot markers")
- `$:default="items"`: Value silently ignored, behaves like `$:default`

---

### ❌ Duplicate Slot Names

```html
<!-- ❌ WRONG - two slots with same name -->
<template id="bad-template">
  <div class="header">
    <slot $:name="header"></slot>
  </div>
  <div class="header-alt">
    <slot $:name="header"></slot>  <!-- Conflict! -->
  </div>
</template>
```

**Why it fails:** No automatic merging. Only one slot receives content (first found during traversal). The other gets its fallback content.

**Fix:** Use different names or redesign layout:
```html
<!-- ✅ CORRECT - unique names -->
<slot $:name="header"></slot>
<slot $:name="header-alt"></slot>

<!-- ✅ CORRECT - single slot -->
<div class="header-wrapper">
  <slot $:name="header"></slot>
</div>
```

---

### ❌ Mixing `$:component:slot` with Named Slots

```html
<!-- ❌ WRONG - can't mix shorthand with named slots -->
<div $:component="user-card" $:component:slot="{ user }">
  <template $:slot="header">Title</template>  <!-- Ignored! -->
  <p $:text="user.name"></p>
</div>
```

**Why it fails:** `$:component:slot` claims the **entire** default slot. Other slot grouping (`$:slot="..."`) is completely bypassed.

**Fix:** Choose one approach:
```html
<!-- ✅ CORRECT - shorthand only (default slot only) -->
<div $:component="user-card" $:component:slot="{ user }">
  <p $:text="user.name"></p>
</div>

<!-- ✅ CORRECT - template syntax (supports named slots) -->
<div $:component="user-card">
  <template $:slot="header">Title</template>
  <template $:scope="{ user }">
    <p $:text="user.name"></p>
  </template>
</div>
```

---

### ❌ Scoped Slots with Inline Components

```html
<!-- ❌ WRONG - scoped slots don't work with inline components -->
<div $:component:is="{ state: { items: [] } }">
  <template $:scope="{ items }">
    <!-- This won't work! -->
    <div $:for="item in items">...</div>
  </template>
</div>
```

**Why it fails:** Slot collection is completely skipped for inline components. The markup is directly bound to the **child** scope, not collected as slot groups.

**Fix:** For inline components, just write content directly (no `<template>` wrappers):
```html
<!-- ✅ CORRECT - direct content for inline components -->
<div $:component:is="{ state: { items: [] } }">
  <div $:for="item in items">...</div>
</div>
```

**For actual slot usage, register a proper component:**
```js
$.jqr.register('item-list', {
  state: { items: [] },
  template: `
    <div>
      <div $:for="item in items" $:key="item">
        <slot $:name="item" $:scope:item="item"></slot>
      </div>
    </div>
  `
});
```

---

### ❌ Invalid `$:scope` Patterns

```html
<!-- ❌ WRONG - invalid destructuring syntax -->
<template $:slot="item" $:scope="user, index">
  <!-- Missing curly braces -->
</template>

<template $:slot="item" $:scope="{ user: }">
  <!-- Incomplete destructuring -->
</template>

<template $:slot="item" $:scope="{{ user }}">
  <!-- Double braces (not Vue/Angular!) -->
</template>
```

**Why it fails:** `$:scope` pattern must be valid JavaScript destructuring or a plain identifier.

**Fix:**
```html
<!-- ✅ CORRECT - object destructuring -->
<template $:slot="item" $:scope="{ user, index }">...</template>

<!-- ✅ CORRECT - rename property -->
<template $:slot="item" $:scope="{ user: currentUser }">...</template>

<!-- ✅ CORRECT - single variable (captures entire scope object) -->
<template $:slot="item" $:scope="slotProps">
  <div $:text="slotProps.user"></div>
</template>
```

**Dev warning:** Invalid pattern triggers warning with expected format hint.

---

### ❌ Using `$:scope:*` on Parent Side

```html
<!-- ❌ WRONG - $:scope:key only works on child side -->
<div $:component="user-list">
  <template $:slot="item" $:scope:user="someValue">
    <!-- This does nothing! -->
  </template>
</div>
```

**Why it fails:** `$:scope:key="expr"` is for **defining** scope variables on child's `<slot>` outlet. Parents **receive** scope with `$:scope="{ pattern }"`.

**Fix:**
```html
<!-- ✅ CORRECT - child side defines -->
<template id="user-list-template">
  <slot $:name="item" $:scope:user="currentUser"></slot>
</template>

<!-- ✅ CORRECT - parent side receives -->
<div $:component="user-list">
  <template $:slot="item" $:scope="{ user }">
    <p $:text="user.name"></p>
  </template>
</div>
```

---

## `$slots` Object - Detailed Behavior

The `$slots` object allows checking which slots were filled by the parent. Understanding its behavior prevents common pitfalls:

### Basic Usage

```html
<template id="card-template">
  <div class="card">
    <!-- Only render header wrapper if parent filled it -->
    <template $:if="$slots['header']">
      <div class="card-header">
        <slot $:name="header"></slot>
      </div>
    </template>
    
    <div class="card-body">
      <slot $:default></slot>
    </div>
    
    <!-- Only render footer if filled -->
    <template $:if="$slots['footer']">
      <div class="card-footer">
        <slot $:name="footer"></slot>
      </div>
    </template>
  </div>
</template>
```

### How `$slots` Works

**Type:** Plain object `{ [slotName]: true | undefined }`

**Rules:**
1. **Rebuilt on each access** - Cheap operation (only a few slots at most)
2. **Only contains keys for non-empty slots** - Empty slots don't appear
3. **Whitespace-only content = empty** - `<template $:slot="x">   </template>` doesn't count
4. **Scoped slots always truthy** - If scoped slot group exists, key is present

**Examples:**

```html
<!-- Parent provides header and default -->
<div $:component="card">
  <div $:slot="header">Title</div>
  <p>Body content</p>
</div>

<!-- Inside card template: -->
$slots['header']  // true
$slots['default'] // true  
$slots['footer']  // undefined
```

```html
<!-- Parent provides empty header -->
<div $:component="card">
  <div $:slot="header">   </div>  <!-- Only whitespace -->
  <p>Body content</p>
</div>

<!-- Inside card template: -->
$slots['header']  // undefined (whitespace-only = empty)
$slots['default'] // true
```

```html
<!-- Parent provides scoped slot -->
<div $:component="user-list">
  <template $:slot="item" $:scope="{ user }">
    <p $:text="user.name"></p>
  </template>
</div>

<!-- Inside user-list template: -->
$slots['item']  // true (scoped slot exists)
```

### `$slots` vs Fallback Content

**Important distinction:** `$:if="$slots[...]"` and fallback content have different behaviors:

```html
<!-- Pattern 1: With $:if -->
<template $:if="$slots['footer']">
  <div class="footer-wrapper">
    <slot $:name="footer">
      <p>Default footer</p>  <!-- Never visible! -->
    </slot>
  </div>
</template>

<!-- Pattern 2: Without $:if -->
<div class="footer-wrapper">
  <slot $:name="footer">
    <p>Default footer</p>  <!-- Visible if parent doesn't fill slot -->
  </slot>
</div>
```

**When parent doesn't fill `footer`:**

| Pattern | Result |
|---------|--------|
| Pattern 1 (with `$:if`) | Entire wrapper doesn't render (fallback never seen) |
| Pattern 2 (without `$:if`) | Wrapper renders with fallback content |

**Choose consciously:**
- Use `$:if` when you want wrapper to **completely disappear** if slot empty
- Skip `$:if` when you want **fallback content to actually show**

### Accessing `$slots`

**Bracket notation (recommended):**
```html
$:if="$slots['header']"
$:if="$slots['footer-actions']"  <!-- Works with kebab-case -->
```

**Dot notation (only for simple names):**
```html
$:if="$slots.header"
$:if="$slots.default"
```

**Don't use for:**
```html
<!-- ❌ WRONG - kebab-case needs brackets -->
$:if="$slots.footer-actions"  <!-- Invalid identifier -->

<!-- ✅ CORRECT -->
$:if="$slots['footer-actions']"
```

### Non-Reactive Nature

**Important:** `$slots` reflects slot state at **mount time only**. It doesn't react to parent state changes.

```html
<!-- Parent -->
<div $:component="card">
  <div $:if="showHeader" $:slot="header">
    Dynamic Header
  </div>
</div>

<!-- Child template -->
<template $:if="$slots['header']">
  <slot $:name="header"></slot>
</template>
```

**Behavior:**
- If `showHeader` is `true` at mount → `$slots['header']` is `true`
- If `showHeader` changes to `false` later → `$slots['header']` **still `true`**
- Slot content's reactive bindings still work (because they're bound to parent scope)
- Only the **presence** of slot is fixed at mount

**Why:** Slot collection happens once per component instance during `$:component` resolution. Changing parent state updates expressions **inside** slot content, not slot group membership.

---

## Slots Inside `$:for` / `$:if` (Deferred Binding)

JQR fully supports scoped slots inside child component's `$:for` or `$:if` blocks - they bind **per iteration** automatically!

### Example: Slot in `$:for`

```html
<!-- Child template -->
<template id="message-list-template">
  <ul>
    <li $:for="msg in messages" $:key="msg.id">
      <!-- Scoped slot inside loop - binds per iteration -->
      <slot $:name="item" $:scope:message="msg" $:scope:remove="removeMessage"></slot>
    </li>
  </ul>
</template>

<!-- Parent usage -->
<div $:component="message-list">
  <template $:slot="item" $:scope="{ message, remove }">
    <span $:text="message.text"></span>
    <button $:click="remove(message.id)">×</button>
  </template>
</div>
```

**What happens:**
1. `$:for="msg in messages"` creates multiple `<li>` elements
2. Each iteration has its own `msg` variable (loop variable)
3. `$:scope:message="msg"` evaluates **per iteration** with that iteration's `msg`
4. Parent's slot content is bound separately for each item
5. Each `<button>` has correct `message.id` for its row

### Technical Details

**No special handling needed** - it just works! But understanding helps:

- Outlet binding happens **per render pass** for elements inside `$:for`
- Loop variable (`msg`) is in scope when `$:scope:message="msg"` evaluates
- Each item gets its own scope object with current iteration's values
- `$slots['item']` still reflects group-level presence (not per-iteration)

### Slot in `$:if`

```html
<!-- Child template -->
<template id="conditional-wrapper-template">
  <template $:if="showDetails">
    <div class="details-section">
      <slot $:name="details" $:scope:data="detailsData"></slot>
    </div>
  </template>
</template>

<!-- Parent usage -->
<div $:component="conditional-wrapper" $:prop:show-details="true">
  <template $:slot="details" $:scope="{ data }">
    <p $:text="data.info"></p>
  </template>
</div>
```

**Behavior:**
- If `showDetails` is `false` → slot outlet doesn't render (normal `$:if` behavior)
- If `showDetails` is `true` → slot outlet binds with current `detailsData`
- `$slots['details']` is `true` regardless (slot group exists from parent side)

### When to Use

**Perfect for:**
- List items with customizable rendering (data tables, user lists)
- Conditional sections that need parent-controlled content
- Complex nested structures (categories with items)

**Example use case:** Product categories with product items:

```html
<template id="category-list-template">
  <div $:for="category in categories" $:key="category.id" class="category">
    <!-- Category-level slot -->
    <slot $:name="category-header" $:scope:category="category"></slot>
    
    <!-- Per-product slot inside nested loop -->
    <div $:for="product in category.products" $:key="product.id">
      <slot $:name="product-item" 
            $:scope:product="product" 
            $:scope:category="category"></slot>
    </div>
  </div>
</template>
```

---

## Combining `$:teleport` with Slots

`$:teleport` and `$:component` work together seamlessly - slots fill first, then entire host teleports:

```html
<div $:component="alert-modal" $:teleport:to.defer="body">
  <template $:slot="header">Alert</template>
  <template $:slot="footer">
    <button>OK</button>
  </template>
  <p>This is an alert message.</p>
</div>
```

**Execution order:**
1. Slots collected and bound to child template
2. Child component mounts with filled slots
3. Final DOM structure (host + content) teleports to `body`

**No conflicts** - treat as independent features on same element.

**Use case:** Modals with dynamic content:
```html
<div $:component="modal" 
     $:teleport:to="body" 
     $:model:open="showModal">
  <template $:slot="header">
    <h3>Confirmation</h3>
  </template>
  
  <p>Are you sure you want to delete this item?</p>
  
  <template $:slot="footer">
    <button $:click="confirmDelete">Yes</button>
    <button $:click="showModal = false">No</button>
  </template>
</div>
```

---

## Validation Checklist

Before committing a component template, verify:

### Child Template (Slot Outlets)

- [ ] Every `<slot>` has one of: `$:default`, `$:name="..."`, or `$:scope:key="..."`
- [ ] No bare `<slot>` without markers
- [ ] No invalid syntax like `<slot $:footer>` (use `$:name="footer"`)
- [ ] No values on `$:default` (like `$:default="items"`)
- [ ] No duplicate slot names in same template
- [ ] Scoped slot variables use **kebab-case** in HTML (`$:scope:user-name`)
- [ ] Fallback content exists for optional slots (or intentionally omitted)

### Parent Usage

- [ ] Named slots wrapped in `<template $:slot="name">`
- [ ] Scoped slots use valid destructuring: `$:scope="{ var }"` or `$:scope="varName"`
- [ ] Not mixing `$:component:slot` with `<template $:slot="...">`
- [ ] Scoped slot variable names match child's exposed variables (accounting for kebab→camel conversion)
- [ ] No `$:scope:key` on parent side (that's child-only syntax)

### Dev Mode Active

- [ ] `$.jqr.config({ devMode: true })` enabled during development
- [ ] Console warnings reviewed and resolved
- [ ] No ignored/suppressed slot-related warnings

### Common Dev Warnings

**Watch for these in console:**

1. **"Unrecognized directive"** - Bare `<slot $:footer>` instead of `$:name="footer"`
2. **"No valid slot markers"** - Plain `<slot>` without markers
3. **"Slot not filled"** - Parent provides slot name that child doesn't have
4. **"Invalid $:scope pattern"** - Destructuring syntax error
5. **"Multiple slots with same name"** - Duplicate `$:name` values

**Enable dev mode:**
```js
// At app initialization
$.jqr.config({ devMode: true });
```

**In production:**
```js
// Disable warnings
$.jqr.config({ devMode: false });
```

---

## Summary Table: Two Sides, Two Vocabularies

| Concern | Child Side (Template) | Parent Side (Usage) |
|---------|----------------------|---------------------|
| **Mark default slot** | `<slot $:default>` | Direct content (no `$:slot`) |
| **Mark named slot** | `<slot $:name="foo">` | `<template $:slot="foo">` |
| **Expose scoped data** | `<slot $:scope:key="expr">` | `<template $:scope="{ key }">` |
| **Check if filled** | `$:if="$slots['name']"` | N/A |
| **Fallback content** | Inside `<slot>...</slot>` | N/A |
| **Attribute naming** | Use **kebab-case** (`$:scope:user-name`) | Destructure as **camelCase** (`{ userName }`) |

**Golden rules:**
1. **Child defines, parent fills**
2. **`$:scope` without arg = parent side**, **`$:scope:key` = child side**
3. **Always use explicit markers** (`$:default`, `$:name`, `$:scope:*`)
4. **Keep dev mode on** until production

---

## Next Steps

- [Props](./props.md) - Component props system & parent-child communication
- [Component Options](./component-options.md) - Component registration & composition
- [Teleport](./teleport.md) - Render outside hierarchy

