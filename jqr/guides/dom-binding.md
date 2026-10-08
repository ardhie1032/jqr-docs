<!-- {
  "path": "jqr/guides/dom-binding",
  "name": "docs-jqr-guides-dom-binding",
  "meta": {
    "title": "DOM Binding",
    "order": 2
  }
} -->

# DOM Binding

JQR provides declarative directives to bind reactive data to the DOM. No manual `innerHTML` or `textContent` needed.

## Text Content: `$:text`

Binds reactive data to an element's text content:

```html live:no-code
<div id="text-demo" class="en-card">
  <div class="en-card__body">
    <h3>Text Binding</h3>
    <input class="en-input" $:model="message" placeholder="Type something...">
    <div class="en-alert en-alert--info" style="margin-top: 1rem;">
      You typed: <strong $:text="message"></strong>
    </div>
  </div>
</div>
```

```html
<div id="text-demo">
  <input $:model="message" placeholder="Type something...">
  <div>You typed: <strong $:text="message"></strong></div>
</div>
```

```js inject show
$('#text-demo').component({
  state: { message: 'Hello JQR' }
});
```

```js cleanup
onUnmount(() => {
  $('#text-demo').component('destroy');
})
```

### Type Handling

`$:text` automatically converts values to strings:

```js
state: {
  number: 42,           // → "42"
  boolean: true,        // → "true"
  array: [1, 2, 3],     // → JSON pretty-printed
  object: { x: 1 },     // → JSON pretty-printed
  nullValue: null,      // → "" (empty)
  undefined: undefined  // → "" (empty)
}
```

```html
<p $:text="number"></p>   <!-- 42 -->
<p $:text="array"></p>    <!-- [1, 2, 3] -->
<p $:text="object"></p>   <!-- { "x": 1 } -->
```

## HTML Content: `$:html`

Binds reactive HTML (use with caution - XSS risk):

```html live:no-code
<div id="html-demo" class="en-card">
  <div class="en-card__body">
    <h3>HTML Binding</h3>
    <div class="en-flex en-gap-2">
      <button class="en-btn en-btn--primary" $:click="setBold">Bold</button>
      <button class="en-btn en-btn--primary" $:click="setItalic">Italic</button>
      <button class="en-btn en-btn--primary" $:click="setLink">Link</button>
    </div>
    <div class="en-alert en-alert--warning" style="margin-top: 1rem;" $:html="html"></div>
  </div>
</div>
```

```html
<div id="html-demo">
  <button $:click="setBold">Bold</button>
  <button $:click="setItalic">Italic</button>
  <button $:click="setLink">Link</button>
  <div $:html="html"></div>
</div>
```

```js inject show
$('#html-demo').component({
  state: { html: '<strong>Bold text</strong>' },
  methods: {
    setBold() {
      this.html = '<strong>Bold text</strong>';
    },
    setItalic() {
      this.html = '<em>Italic text</em>';
    },
    setLink() {
      this.html = '<a href="#">Click me</a>';
    }
  }
});
```

```js cleanup
onUnmount(() => {
  $('#html-demo').component('destroy');
})
```

**⚠️ Security Warning:** Never use `$:html` with user input unless sanitized. Configure a global sanitizer:

```js
$.jqr.config({
  sanitizeHtml: (html) => DOMPurify.sanitize(html)
});
```

## Show/Hide: `$:show`

Toggles element visibility with `display: none`:

```html live:no-code
<div id="show-demo" class="en-card">
  <div class="en-card__body">
    <h3>Show/Hide</h3>
    <label class="en-check">
      <input type="checkbox" $:model="isVisible">
      <span>Show message</span>
    </label>
    <div class="en-alert en-alert--success" style="margin-top: 1rem;" $:show="isVisible">
      ✅ This message is visible!
    </div>
  </div>
</div>
```

```html
<div id="show-demo">
  <label>
    <input type="checkbox" $:model="isVisible">
    Show message
  </label>
  <div $:show="isVisible">
    ✅ This message is visible!
  </div>
</div>
```

```js inject show
$('#show-demo').component({
  state: { isVisible: true }
});
```

```js cleanup
onUnmount(() => {
  $('#show-demo').component('destroy');
})
```

**How it works:**
- `$:show="true"` → Element visible (restores original `display`)
- `$:show="false"` → Element hidden (`display: none`)

## Conditional: `$:if`

True conditional rendering - removes element from DOM when condition is false:

```html
<p $:if="isLoggedIn">Welcome back!</p>
```

**Behavior:**
- When `true`: Element is mounted and bound
- When `false`: Element is completely removed from DOM, all effects/listeners disposed
- Toggling back to `true`: Element is cloned and re-bound from scratch

**Performance:** Use `$:show` if you need cheaper toggle (keeps DOM mounted, just hidden).

### `$:if` on `<template>` Element

Use `<template $:if>` for conditional rendering without a wrapper element:

```html live:no-code
<div id="demo-if-template" class="en-card">
  <div class="en-card__body">
    <h3>Template with $:if</h3>
    <label class="en-check">
      <input type="checkbox" $:model="showDetails">
      <span>Show Details</span>
    </label>
    <div style="margin-top: 1rem;">
      <p>Always visible</p>
      <template $:if="showDetails">
        <p class="en-text--muted">Detail line 1</p>
        <p class="en-text--muted">Detail line 2</p>
        <p class="en-text--muted">Detail line 3</p>
      </template>
    </div>
  </div>
</div>
```

```html
<div id="demo-if-template">
  <label>
    <input type="checkbox" $:model="showDetails"> Show Details
  </label>
  <p>Always visible</p>
  <template $:if="showDetails">
    <p>Detail line 1</p>
    <p>Detail line 2</p>
    <p>Detail line 3</p>
  </template>
</div>
```

```js inject show
$('#demo-if-template').component({
  state: { showDetails: false }
});
```

```js cleanup
onUnmount(() => {
  $('#demo-if-template').component('destroy');
})
```

**Benefits:**
- No wrapper `<div>` needed
- Multiple elements rendered as siblings
- Useful for complex layouts where wrappers break CSS

## List Rendering: `$:for`

Render lists of items reactively:

### Basic `$:for`

```html live:no-code
<div id="demo-for-basic" class="en-card">
  <div class="en-card__body">
    <h3>List Rendering</h3>
    <ul style="list-style: none; padding: 0;">
      <li $:for="item in items" $:key="item" 
          class="en-alert en-alert--info" 
          style="margin-bottom: 0.5rem;"
          $:text="item"></li>
    </ul>
  </div>
</div>
```

```html
<div id="demo-for-basic">
  <ul>
    <li $:for="item in items" $:key="item" $:text="item"></li>
  </ul>
</div>
```

```js inject show
$('#demo-for-basic').component({
  state: { items: ['Apple', 'Banana', 'Orange'] }
});
```

```js cleanup
onUnmount(() => {
  $('#demo-for-basic').component('destroy');
})
```

### `$:for` on `<template>` Element

Use `<template $:for>` to render multiple elements per item without a wrapper:

```html live:no-code
<div id="demo-for-template" class="en-card">
  <div class="en-card__body">
    <h3>Template with $:for</h3>
    <div>
      <template $:for="lang in langs" $:key="lang.name">
        <span $:text="lang.name" 
              class="en-badge en-badge--primary" 
              style="margin-right: 0.5rem;"
              $:if="lang.name === 'PHP'"></span>
        <template $:if="lang.name !== 'PHP'">
          <strong $:text="lang.name" 
                  class="en-badge en-badge--danger" 
                  style="margin-right: 0.5rem;"></strong>
        </template>
      </template>
    </div>
  </div>
</div>
```

```html
<div $:component:state="{ langs: [
  {name: 'PHP'},
  {name: 'Rust'},
  {name: 'JavaScript'}
] }">
  <template $:for="lang in langs" $:key="lang.name">
    <span $:text="lang.name" $:if="lang.name === 'PHP'"></span>
    <template $:if="lang.name !== 'PHP'">
      <strong $:text="lang.name" style="color: red;"></strong>
    </template>
  </template>
</div>
```

```js inject show
$('#demo-for-template').component({
  state: {
    langs: [
      {name: 'PHP'},
      {name: 'Rust'},
      {name: 'JavaScript'}
    ]
  }
});
```

```js cleanup
onUnmount(() => {
  $('#demo-for-template').component('destroy');
})
```

**Nested conditionals inside loops:**
```html
<template $:for="item in items" $:key="item.id">
  <h3 $:text="item.title"></h3>
  <template $:if="item.hasDetails">
    <p $:text="item.description"></p>
    <p $:text="item.author"></p>
  </template>
</template>
```

**Benefits:**
- Renders content directly without wrapper elements
- Perfect for table rows, list items, or flex/grid layouts
- Nested `<template $:if>` inside `<template $:for>` works seamlessly

### With Index

```html
<div $:for="(item, index) in items" $:key="item">
  <span $:text="index + 1"></span>. <span $:text="item"></span>
</div>
```

### `$:key` for Performance

Always use `$:key` with `$:for` for efficient updates:

```html
<!-- ✅ Good - uses unique key -->
<div $:for="user in users" $:key="user.id" $:text="user.name"></div>

<!-- ⚠️ Works but less efficient -->
<div $:for="user in users" $:text="user.name"></div>
```

## Class Binding: `$:class`

Dynamically adds/removes CSS classes:

### String Syntax

```html
<div $:class="activeClass"></div>
```

```js
state: { activeClass: 'active highlight' }
```

### Object Syntax

```html live:no-code
<div id="class-demo" class="en-card">
  <div class="en-card__body">
    <h3>Class Binding</h3>
    <div class="en-flex en-gap-2">
      <label class="en-check">
        <input type="checkbox" $:model="isActive">
        <span>Active</span>
      </label>
      <label class="en-check">
        <input type="checkbox" $:model="hasError">
        <span>Error</span>
      </label>
    </div>
    <div 
      class="en-alert" 
      style="margin-top: 1rem;"
      $:class="{ 'en-alert--success': isActive, 'en-alert--danger': hasError }">
      Status indicator
    </div>
  </div>
</div>
```

```html
<div id="class-demo">
  <label>
    <input type="checkbox" $:model="isActive"> Active
  </label>
  <label>
    <input type="checkbox" $:model="hasError"> Error
  </label>
  <div 
    class="alert"
    $:class="{ 'active': isActive, 'error': hasError }">
    Status indicator
  </div>
</div>
```

```js inject show
$('#class-demo').component({
  state: {
    isActive: false,
    hasError: false
  }
});
```

```js cleanup
onUnmount(() => {
  $('#class-demo').component('destroy');
})
```

**Object syntax:** Keys are class names, values are booleans:

```js
$:class="{ active: isActive, disabled: !isEnabled }"
```

### Array Syntax

Combine multiple class expressions:

```html
<div $:class="[baseClass, { active: isActive }]"></div>
```

```js
state: {
  baseClass: 'btn btn-primary',
  isActive: true
}
// Result: class="btn btn-primary active"
```

**Static classes are preserved:**

```html
<div class="container" $:class="{ active: isActive }">
<!-- Always has "container", plus "active" when isActive is true -->
```

## Style Binding: `$:style`

Dynamically applies inline styles:

### Object Syntax

```html live:no-code
<div id="style-demo" class="en-card">
  <div class="en-card__body">
    <h3>Style Binding</h3>
    <div class="en-grid en-grid--cols-2 en-gap-2">
      <div>
        <input class="en-input" $:model="color" type="color" value="#3b82f6">
      </div>
      <div class="en-flex" style="align-self: center;">
        <input class="en-input en-my-auto" $:model.number="size" type="range" min="12" max="48">
      </div>
    </div>
    <div 
      class="en-alert en-alert--info" 
      style="margin-top: 1rem;"
      $:style="{ color: color, fontSize: size + 'px' }">
      Styled text (size: <span $:text="size"></span>px)
    </div>
  </div>
</div>
```

```html
<div id="style-demo">
  <input $:model="color" type="color">
  <input $:model.number="size" type="range" min="12" max="48">
  <div $:style="{ color: color, fontSize: size + 'px' }">
    Styled text
  </div>
</div>
```

```js inject show
$('#style-demo').component({
  state: {
    color: '#3b82f6',
    size: 16
  }
});
```

```js cleanup
onUnmount(() => {
  $('#style-demo').component('destroy');
})
```

**CSS properties** can be camelCase or kebab-case:

```js
$:style="{ fontSize: '16px' }"    // camelCase
$:style="{ 'font-size': '16px' }" // kebab-case
```

### String Syntax

```html
<div $:style="styleString"></div>
```

```js
state: { styleString: 'color: red; font-size: 20px;' }
```

### Array Syntax

Merge multiple style objects:

```html
<div $:style="[baseStyles, conditionalStyles]"></div>
```

```js
state: {
  baseStyles: { color: 'blue' },
  conditionalStyles: { fontWeight: 'bold' }
}
```

**Static styles are preserved:**

```html
<div style="padding: 10px;" $:style="{ color: textColor }">
<!-- Always has padding, plus dynamic color -->
```

## Attribute Binding: `$:attr:*`

Binds any HTML attribute dynamically:

```html live:no-code
<div id="attr-demo" class="en-card">
  <div class="en-card__body">
    <h3>Attribute Binding</h3>
    <input class="en-input" $:model="url" placeholder="Enter image URL">
    <div style="margin-top: 1rem;">
      <img 
        $:attr:src="url" 
        $:attr:alt="'Dynamic image'"
        style="max-width: 100%; height: auto; border-radius: 8px;">
    </div>
  </div>
</div>
```

```html
<div id="attr-demo">
  <input $:model="url" placeholder="Enter image URL">
  <img 
    $:attr:src="url" 
    $:attr:alt="'Dynamic image'">
</div>
```

```js inject show
$('#attr-demo').component({
  state: {
    url: 'https://via.placeholder.com/300x200/3b82f6/ffffff?text=JQR'
  }
});
```

```js cleanup
onUnmount(() => {
  $('#attr-demo').component('destroy');
})
```

**Syntax:** `$:attr:name="expression"`

### Boolean Attributes

For boolean attributes (`disabled`, `checked`, `hidden`):
- `false` or `null` → attribute removed
- `true` → attribute set (empty value)
- Other values → converted to string

```html
<button $:attr:disabled="isLoading">Submit</button>
<!-- disabled when isLoading is true -->

<input type="checkbox" $:attr:checked="isChecked">
<!-- checked when isChecked is true -->
```

### Common Use Cases

```html
<!-- Links -->
<a $:attr:href="dynamicUrl">Link</a>

<!-- Images -->
<img $:attr:src="imageUrl" $:attr:alt="imageAlt">

<!-- Forms -->
<input $:attr:type="inputType" $:attr:placeholder="hint">

<!-- Data attributes -->
<div $:attr:data-id="userId"></div>

<!-- ARIA -->
<button $:attr:aria-label="buttonLabel">Icon</button>
```

### ⚠️ Common Pitfall: Attributes Are NOT Directives

**Important:** Most HTML attributes don't have dedicated directives. You must use `$:attr:` or `$:bind`:

```html
<!-- ❌ WRONG - These DON'T work -->
<input $:placeholder="hint">
<button $:disabled="!isValid">
<a $:href="link">
<input $:readonly="isReadOnly">
<img $:alt="description">
<div $:title="tooltip">
```

**Why?** Attributes like `placeholder`, `disabled`, `href`, etc. are NOT in JQR's directive list. Using them directly (e.g., `$:placeholder`) will:
1. Trigger `unknownDirective` warning in dev mode
2. Not bind the attribute at all
3. Fall through to the `default` branch with no effect

**✅ CORRECT - Use `$:attr:`:**

```html
<input $:attr:placeholder="hint">
<button $:attr:disabled="!isValid">
<a $:attr:href="link">
<input $:attr:readonly="isReadOnly">
<img $:attr:alt="description">
<div $:attr:title="tooltip">
```

**✅ ALSO CORRECT - Use `$:bind`:**

```html
<input $:bind="{ placeholder: hint, disabled: !isValid }">
<a $:bind="{ href: link, title: tooltip }">
<img $:bind="{ src: imageUrl, alt: description }">
```

**Exception:** Some common attributes DO have dedicated directives:
```html
<!-- ✅ These work - they ARE directives -->
<input $:value="text">        <!-- $:value directive exists -->
<input $:checked="isChecked"> <!-- $:checked directive exists -->
<div $:class="className">     <!-- $:class directive exists -->
<div $:style="styles">        <!-- $:style directive exists -->
```

**Rule of thumb:** If you're unsure whether an attribute has a directive, use `$:attr:` or check the [Directive Reference](../reference/directive-reference.md).

## Shorthand: `$:bind`

Bind multiple properties from a single object:

```html live:no-code
<div id="bind-demo" class="en-card">
  <div class="en-card__body">
    <h3>Multiple Bindings</h3>
    <div class="en-flex en-gap-2">
      <label class="en-check">
        <input type="checkbox" $:model="isActive">
        <span>Active</span>
      </label>
    </div>
    <div style="margin-top: 1rem;" $:bind="boxStyles">
      Dynamic box
    </div>
  </div>
</div>
```

```html
<div id="bind-demo">
  <label>
    <input type="checkbox" $:model="isActive"> Active
  </label>
  <div $:bind="boxStyles">Dynamic box</div>
</div>
```

```js inject show
$('#bind-demo').component({
  state: { isActive: false },
  computed: {
    boxStyles(c) {
      return {
        text: c.isActive ? 'Active!' : 'Inactive',
        class: c.isActive ? 'en-alert en-alert--success' : 'en-alert en-alert--muted',
        style: { 
          padding: '1rem',
          textAlign: 'center'
        }
      };
    }
  }
});
```

```js cleanup
onUnmount(() => {
  $('#bind-demo').component('destroy');
})
```

**Equivalent to:**

```html
<div 
  $:text="text"
  $:class="class"
  $:style="style">
```

Supported keys:
- `text` → `$:text`
- `html` → `$:html`
- `class` → `$:class`
- `style` → `$:style`
- `show` → `$:show`
- Any other key → `$:attr:key`

## Complete Example

```html live:no-code
<div id="complete-demo" class="en-card">
  <div class="en-card__body">
    <h3>Product Card</h3>
    <div class="en-flex en-gap-2" style="margin-bottom: 1rem;">
      <select class="en-select" $:model="selectedProduct">
        <option value="laptop">Laptop</option>
        <option value="phone">Phone</option>
        <option value="tablet">Tablet</option>
      </select>
      <label class="en-check">
        <input type="checkbox" $:model="inStock">
        <span>In&nbsp;Stock</span>
      </label>
    </div>
    <div 
      class="en-alert" 
      $:class="{ 'en-alert--success': inStock, 'en-alert--muted': !inStock }">
      <h4 $:text="product.name" style="margin: 0 0 0.5rem 0;"></h4>
      <p $:text="product.description" style="margin: 0 0 0.5rem 0;"></p>
      <div 
        $:style="{ fontSize: '1.25rem', fontWeight: 'bold', color: inStock ? '#10b981' : '#6b7280' }"
        $:text="'$' + product.price">
      </div>
      <div style="margin-top: 0.5rem;" $:show="!inStock" class="en-text--muted">
        Out of stock
      </div>
    </div>
  </div>
</div>
```

```html
<div id="complete-demo">
  <h3>Product Card</h3>
  <select $:model="selectedProduct">
    <option value="laptop">Laptop</option>
    <option value="phone">Phone</option>
    <option value="tablet">Tablet</option>
  </select>
  <label>
    <input type="checkbox" $:model="inStock"> In Stock
  </label>
  
  <div $:class="{ 'success': inStock, 'muted': !inStock }">
    <h4 $:text="product.name"></h4>
    <p $:text="product.description"></p>
    <div 
      $:style="{ fontSize: '1.25rem', fontWeight: 'bold', color: inStock ? 'green' : 'gray' }"
      $:text="'$' + product.price">
    </div>
    <div $:show="!inStock">Out of stock</div>
  </div>
</div>
```

```js inject show
$('#complete-demo').component({
  state: {
    selectedProduct: 'laptop',
    inStock: true,
    products: {
      laptop: { name: 'Pro Laptop', description: 'High-performance laptop', price: 1299 },
      phone: { name: 'Smart Phone', description: '5G smartphone', price: 799 },
      tablet: { name: 'Tablet Pro', description: 'Premium tablet', price: 599 }
    }
  },
  computed: {
    product(c) {
      return c.products[c.selectedProduct];
    }
  }
});
```

```js cleanup
onUnmount(() => {
  $('#complete-demo').component('destroy');
})
```

## Next Steps

- [Events](./events.md) - Handle user interactions
- [Forms and Models](./forms-and-models.md) - Two-way data binding
- [Lists and Conditionals](./lists-and-conditionals.md) - Render lists with `$:for`
