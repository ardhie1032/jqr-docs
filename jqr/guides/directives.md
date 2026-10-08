<!-- {
  "path": "jqr/guides/directives",
  "name": "docs-jqr-guides-directives",
  "meta": {
    "title": "Directives Overview",
    "order": 2
  }
} -->

# Directives Overview

Directives are special HTML attributes prefixed with `$:` that bind data to the DOM. They provide a declarative way to apply reactive behavior to your templates.

## What are Directives?

Directives tell JQR how to reactively update the DOM when state changes. Instead of manually manipulating elements, you declare the desired outcome in HTML.

**Example:**
```html
<!-- Without directives (manual) -->
<p id="message"></p>
<script>
  $('#message').text(state.message); // Must update manually
</script>

<!-- With directives (reactive) -->
<p $:text="message"></p>
<!-- Updates automatically when state.message changes! -->
```

## Directive Syntax

All directives follow the `$:` prefix convention:

```html
<element $:directive="expression"></element>
```

- **`$:`** - Directive prefix (configurable via `$.jqr.config()`)
- **`directive`** - Directive name (`text`, `if`, `for`, etc.)
- **`expression`** - JavaScript expression evaluated in component scope

### With Arguments

Some directives accept arguments after a colon:

```html
<div $:bind:class="{ active: isActive }"></div>
<div $:bind:style="{ color: themeColor }"></div>
```

### With Modifiers

Modifiers customize directive behavior, added with dots:

```html
<input $:model.trim.lazy="username">
<button $:click.prevent.once="submit()">Submit</button>
```

## Directive Arguments

JQR supports directives with arguments in the form `$:name:arg`. Here are the recognized patterns:

### Recognized Directive Arguments

| Directive | Description | Example |
|-----------|-------------|---------|
| `$:attr:<name>` | Bind any HTML attribute | `<img $:attr:src="url">` |
| `$:on:<event>` | Listen to any event | `<div $:on:mouseenter="hover()">` |
| `$:model:<prop>` | Two-way bind to DOM property | `<dialog $:model:open="isOpen">` |
| `$:prop:<key>` | Pass prop to component | `<div $:component="card" $:prop:user-id="id">` |
| `$:component:is` | Dynamic component | `<div $:component:is="currentView">` |
| `$:component:<key>` | Inline component option | `<div $:component:state="{ count: 0 }">` |
| `$:component:slot` | Slot alias | `<div $:component="card" $:component:slot="data">` |
| `$:teleport:to` | Teleport to selector | `<div $:teleport:to="#modal">` |
| `$:scope:<key>` | Scoped slot value | `<slot $:scope:user="currentUser">` |

### Important Notes

**`$:attr:<name>`** - Use this for any HTML attribute that doesn't have a dedicated directive:

```html
<!-- ✅ Correct - use $:attr: for HTML attributes -->
<input $:attr:placeholder="hint">
<a $:attr:href="link">
<button $:attr:disabled="!isValid">
<img $:attr:alt="description">

<!-- ✅ Alternative - use $:bind for multiple attributes -->
<input $:bind="{ placeholder: hint, disabled: !isValid }">
```

**`$:on:<event>`** - Handle any DOM event outside the common shortcuts:

```html
<!-- Common events have shortcuts -->
<button $:click="handler()">Click</button>

<!-- Use $:on: for less common events -->
<div $:on:mouseenter="onHover()">
<div $:on:mouseleave="onLeave()">
<div $:on:animationend="onAnimEnd()">
```

**`$:model:<prop>`** - Two modes:
- **Mode B (native elements):** Bind to DOM property like `open`, `checked`, `value`
- **Mode A (components):** Sugar for `$:prop:<key>` + `$:on:update:<key>`

```html
<!-- Mode B: Native element property -->
<dialog $:model:open="dialogOpen">
<details $:model:open="detailsOpen">

<!-- Mode A: Component prop -->
<div $:component="modal" $:model:open="isOpen">
<!-- Same as: $:prop:open="isOpen" $:on:update:open="isOpen = $event" -->
```

**`$:prop:<key>`** - Pass props to nested components. Keys must be kebab-case:

```html
<!-- ✅ Correct - kebab-case -->
<div $:component="user-card" $:prop:user-id="currentId">

<!-- ❌ Wrong - camelCase not preserved by HTML parser -->
<div $:component="user-card" $:prop:userId="currentId">
```

**`$:component:<key>`** - Inline component definition. Recognized keys:
- `prefix`, `state`, `computed`, `methods`, `async-state`, `props`
- `template`, `components`, `constants`, `emits`
- `created`, `mounted`, `updated`, `destroyed`

```html
<!-- Inline component with multiple options -->
<div $:component 
     $:component:state="{ count: 0 }"
     $:component:methods="{ inc: s => s.count++ }">
  <p $:text="count"></p>
  <button $:click="inc()">+</button>
</div>
```

## Common Pitfalls

### ❌ Using Attribute Names as Directives

HTML attributes are NOT directives. You cannot use them directly with `$:`:

```html
<!-- ❌ WRONG - These don't work -->
<input $:placeholder="hint">
<button $:disabled="!isValid">
<a $:href="link">
<input $:readonly="true">
<img $:alt="description">
<div $:title="tooltip">
```

**Why?** These attributes are not in JQR's directive list. They fall to the `default` branch and trigger `unknownDirective` warning.

**✅ Solution 1: Use `$:attr:<name>`**

```html
<input $:attr:placeholder="hint">
<button $:attr:disabled="!isValid">
<a $:attr:href="link">
<input $:attr:readonly="true">
<img $:attr:alt="description">
<div $:attr:title="tooltip">
```

**✅ Solution 2: Use `$:bind`**

```html
<input $:bind="{ placeholder: hint, disabled: !isValid }">
<a $:bind="{ href: link, title: tooltip }">
```

### ⚠️ Exception: Known Directives

Some common attributes DO have dedicated directives:

```html
<!-- ✅ These work - they ARE directives -->
<input $:value="text">        <!-- $:value directive -->
<input $:checked="isChecked"> <!-- $:checked directive -->
<div $:class="className">     <!-- $:class directive -->
<div $:style="styles">        <!-- $:style directive -->
```

**When in doubt:** Check the [Directive Reference](../reference/directive-reference.md) to see if an attribute has a dedicated directive. If not, use `$:attr:` or `$:bind`.

## Built-in Directives

### Text & Content

| Directive | Purpose | Example |
|-----------|---------|---------|
| `$:text` | Set text content | `<p $:text="message"></p>` |
| `$:html` | Set HTML content | `<div $:html="richContent"></div>` |

### Attributes

| Directive | Purpose | Example |
|-----------|---------|---------|
| `$:attr:*` | Set attribute | `<img $:attr:src="imageUrl">` |
| `$:class` | Bind classes | `<div $:class="{ active: isActive }"></div>` |
| `$:style` | Bind styles | `<div $:style="{ color: textColor }"></div>` |
| `$:bind` | Multi-attribute binding | `<div $:bind="{ class: 'card', style: {...} }"></div>` |

### Conditionals

| Directive | Purpose | Example |
|-----------|---------|---------|
| `$:if` | Conditional rendering (removes from DOM) | `<div $:if="isVisible">...</div>` |
| `$:show` | Toggle visibility (display: none) | `<div $:show="isVisible">...</div>` |

### Lists

| Directive | Purpose | Example |
|-----------|---------|---------|
| `$:for` | List rendering | `<li $:for="item in items" $:text="item"></li>` |
| `$:key` | List item key | `<li $:for="user in users" $:key="user.id"></li>` |

### Forms

| Directive | Purpose | Example |
|-----------|---------|---------|
| `$:model` | Two-way data binding | `<input $:model="username">` |
| `$:model.*` | With modifiers | `<input $:model.number.trim="age">` |

### Events

| Directive | Purpose | Example |
|-----------|---------|---------|
| `$:click` | Click event | `<button $:click="save()">Save</button>` |
| `$:input` | Input event | `<input $:input="handleInput()">` |
| `$:submit` | Form submit | `<form $:submit.prevent="onSubmit()">` |
| `$:on:*` | Any event | `<div $:on:mouseenter="hover = true">` |

### Components

| Directive | Purpose | Example |
|-----------|---------|---------|
| `$:component` | Render component | `<div $:component="user-card"></div>` |
| `$:component:*` | Inline component | `<div $:component:state="{ count: 0 }">` |
| `$:prop:*` | Pass prop | `<div $:component="card" $:prop:title="pageTitle">` |

### Slots

| Directive | Purpose | Example |
|-----------|---------|---------|
| `$:slot` | Named slot | `<template $:slot="header">...</template>` |
| `$:scope` | Scoped slot | `<template $:slot="item" $:scope="{ user }">` |
| `$:default` | Default slot marker | `<slot $:default></slot>` |
| `$:name` | Named slot outlet | `<slot $:name="header"></slot>` |

### Special

| Directive | Purpose | Example |
|-----------|---------|---------|
| `$:ref` | Element reference | `<input $:ref="usernameInput">` |
| `$:teleport` | Portal rendering | `<div $:teleport="#modal-target">` |
| `$:transition` | Transition effects | `<div $:transition="fade">` |

## Quick Examples

### Text Content

```html live:no-code
<div id="demo-text" class="en-card en-card__body">
  <p $:text="message"></p>
  <input class="en-input" $:model="message" placeholder="Type something...">
</div>
```

```html
<div id="demo-text">
  <p $:text="message"></p>
  <input $:model="message" placeholder="Type something...">
</div>
```

```js inject show
$('#demo-text').component({
  state: { message: 'Hello, World!' }
});
```

```js cleanup
onUnmount(() => {
  $('#demo-text').component('destroy');
})
```

### Conditional Rendering

```html live:no-code
<div id="demo-if" class="en-card en-card__body">
  <div class="en-alert en-alert--success" $:if="isVisible">
    This message is conditionally rendered
  </div>
  <button class="en-btn en-btn--sm" $:click="isVisible = !isVisible">
    <span $:text="isVisible ? 'Hide' : 'Show'"></span>
  </button>
</div>
```

```html
<div id="demo-if">
  <div $:if="isVisible">This message is conditionally rendered</div>
  <button $:click="isVisible = !isVisible">
    <span $:text="isVisible ? 'Hide' : 'Show'"></span>
  </button>
</div>
```

```js inject show
$('#demo-if').component({
  state: { isVisible: true }
});
```

```js cleanup
onUnmount(() => {
  $('#demo-if').component('destroy');
})
```

### List Rendering

```html live:no-code
<div id="demo-for" class="en-card en-card__body">
  <ul class="en-list-group">
    <li class="en-list-group__item" $:for="fruit in fruits" $:key="fruit" $:text="fruit"></li>
  </ul>
</div>
```

```html
<div id="demo-for">
  <ul>
    <li $:for="fruit in fruits" $:key="fruit" $:text="fruit"></li>
  </ul>
</div>
```

```js inject show
$('#demo-for').component({
  state: {
    fruits: ['Apple', 'Banana', 'Cherry', 'Date']
  }
});
```

```js cleanup
onUnmount(() => {
  $('#demo-for').component('destroy');
})
```

### Class Binding

```html live:no-code
<div id="demo-class" class="en-card en-card__body">
  <div class="en-alert" $:class="{ 'en-alert--success': isActive, 'en-alert--warning': !isActive }">
    <span $:text="isActive ? 'Active' : 'Inactive'"></span>
  </div>
  <button class="en-btn en-btn--sm en-mt-2" $:click="isActive = !isActive">Toggle</button>
</div>
```

```html
<div id="demo-class">
  <div $:class="{ active: isActive, inactive: !isActive }">
    <span $:text="isActive ? 'Active' : 'Inactive'"></span>
  </div>
  <button $:click="isActive = !isActive">Toggle</button>
</div>
```

```js inject show
$('#demo-class').component({
  state: { isActive: true }
});
```

```js cleanup
onUnmount(() => {
  $('#demo-class').component('destroy');
})
```

### Event Handling

```html live:no-code
<div id="demo-click" class="en-card en-card__body">
  <p>Count: <code $:text="count"></code></p>
  <div class="en-flex en-gap-2">
    <button class="en-btn en-btn--sm" $:click="count++">Increment</button>
    <button class="en-btn en-btn--sm" $:click="count--">Decrement</button>
    <button class="en-btn en-btn--sm en-btn--outline" $:click="count = 0">Reset</button>
  </div>
</div>
```

```html
<div id="demo-click">
  <p>Count: <code $:text="count"></code></p>
  <button $:click="count++">Increment</button>
  <button $:click="count--">Decrement</button>
  <button $:click="count = 0">Reset</button>
</div>
```

```js inject show
$('#demo-click').component({
  state: { count: 0 }
});
```

```js cleanup
onUnmount(() => {
  $('#demo-click').component('destroy');
})
```

### Two-Way Binding

```html live:no-code
<div id="demo-model" class="en-card en-card__body">
  <input class="en-input en-mb-2" $:model="username" placeholder="Enter username...">
  <p class="en-text--muted">You typed: <code $:text="username || '(nothing)'"></code></p>
</div>
```

```html
<div id="demo-model">
  <input $:model="username" placeholder="Enter username...">
  <p>You typed: <code $:text="username || '(nothing)'"></code></p>
</div>
```

```js inject show
$('#demo-model').component({
  state: { username: '' }
});
```

```js cleanup
onUnmount(() => {
  $('#demo-model').component('destroy');
})
```

## Directive Categories

### 1. Content Directives
Manipulate element content:
- `$:text` - Safe text content
- `$:html` - Raw HTML (use cautiously)

**See:** [DOM Binding Guide](./dom-binding.md)

### 2. Attribute Directives
Control element attributes:
- `$:attr:*` - Any attribute
- `$:class` - CSS classes
- `$:style` - Inline styles
- `$:bind` - Multiple attributes

**See:** [DOM Binding Guide](./dom-binding.md), [Bind Directive Guide](./bind-directive.md)

### 3. Conditional Directives
Show/hide elements:
- `$:if` - Remove from DOM
- `$:show` - Toggle display

**See:** [Lists & Conditionals Guide](./lists-and-conditionals.md)

### 4. Loop Directives
Render lists:
- `$:for` - Iterate over arrays
- `$:key` - Track list items

**See:** [Lists & Conditionals Guide](./lists-and-conditionals.md)

### 5. Form Directives
Two-way data binding:
- `$:model` - Input binding
- Modifiers: `.trim`, `.number`, `.lazy`, `.debounce`, etc.

**See:** [Forms & Models Guide](./forms-and-models.md), [Model Modifiers Guide](./model-modifiers.md)

### 6. Event Directives
Handle user interactions:
- `$:click`, `$:input`, `$:submit`, etc.
- Modifiers: `.prevent`, `.stop`, `.once`, `.enter`, etc.

**See:** [Events Guide](./events.md)

### 7. Component Directives
Component composition:
- `$:component` - Render component
- `$:prop:*` - Pass props
- `$:component:*` - Inline definition

**See:** [Props Guide](./props.md), [Slots Guide](./slots.md)

### 8. Slot Directives
Content distribution:
- `$:slot` - Fill named slot
- `$:scope` - Scoped slot data
- `$:default`, `$:name` - Slot outlets

**See:** [Slots Guide](./slots.md)

### 9. Special Directives
Advanced features:
- `$:ref` - Element references (access DOM elements via `this.$refs`)
- `$:teleport` - Portal rendering (move elements to different DOM location)
- `$:transition` - Animations (CSS transitions/animations)

**See:** [Element References Guide](./ref.md), [Teleport Guide](./teleport.md), [Transitions Guide](./transitions.md)

## Expression Context

Directive expressions are evaluated in the component scope:

```js
$('#app').component({
  state: { count: 0 },
  computed: { doubled: (s) => s.count * 2 },
  methods: {
    increment(state) {
      state.count++;
    }
  }
});
```

**Available in expressions:**
- `state.*` - State properties (can omit `state.` prefix)
- `computed.*` - Computed values
- `methods.*` - Methods (call directly)
- `$props.*` - Props (nested components)
- `$state` - Full state object
- `{key}Loading`, `{key}Error` - Async state status (e.g., `usersLoading`, `usersError`)

**Examples:**
```html
<!-- Direct state access -->
<p $:text="count"></p>
<p $:text="state.count"></p> <!-- Also works -->

<!-- Computed values -->
<p $:text="doubled"></p>

<!-- Method calls -->
<button $:click="increment()">+</button>

<!-- Complex expressions -->
<p $:text="`Count: ${count}, Double: ${doubled}`"></p>
<div $:class="{ high: count > 10 }"></div>
```

## Custom Directives

Create your own directives:

```js
$.jqr.directive('tooltip', {
  bind(el, ctx) {
    el.title = ctx.evaluate(ctx.expr);
  },
  update(el, ctx) {
    el.title = ctx.evaluate(ctx.expr);
  },
  unbind(el) {
    el.removeAttribute('title');
  }
});
```

**Usage:**
```html
<button $:tooltip="helpText">Help</button>
```

**See:** [Custom Directives Guide](./custom-directives.md)

## Directive Priority

When multiple directives are on the same element, they execute in this order:

1. `$:for` - Creates multiple elements
2. `$:if` / `$:show` - Controls visibility
3. `$:component` - Initializes component
4. `$:bind` - Sets multiple attributes
5. Other directives - Order doesn't matter
6. Event handlers (`$:click`, etc.) - Attached last

**Example:**
```html
<!-- Correct: $:for before $:if -->
<div $:for="item in items" $:if="item.visible" $:text="item.name"></div>
```

## Configuration

Customize the directive prefix:

```js
$.jqr.config({
  prefix: 'v-'  // Use Vue-style directives
});
```

**Then use:**
```html
<p v-text="message"></p>
<div v-if="isVisible"></div>
```

**See:** [Prefixes & Config Guide](./prefixes-and-config.md)

## Best Practices

### 1. Keep Expressions Simple

```html
<!-- ✅ Good -->
<p $:text="fullName"></p>

<!-- ❌ Bad - complex logic in template -->
<p $:text="firstName.trim().toUpperCase() + ' ' + lastName.trim().toUpperCase()"></p>
```

Use computed values for complex expressions.

### 2. Use Appropriate Directives

```html
<!-- ✅ Good - $:text for safety -->
<p $:text="userInput"></p>

<!-- ❌ Bad - $:html vulnerable to XSS -->
<p $:html="userInput"></p>
```

### 3. Always Use Keys in Lists

```html
<!-- ✅ Good - keyed list -->
<li $:for="user in users" $:key="user.id"></li>

<!-- ❌ Bad - no key, inefficient updates -->
<li $:for="user in users"></li>
```

### 4. Avoid Side Effects in Expressions

```html
<!-- ❌ Bad - modifies state in display -->
<p $:text="count++"></p>

<!-- ✅ Good - pure expression -->
<p $:text="count"></p>
<button $:click="count++">Increment</button>
```

### 5. Use Modifiers Appropriately

```html
<!-- ✅ Good - prevent default on form -->
<form $:submit.prevent="onSubmit()"></form>

<!-- ✅ Good - debounce input -->
<input $:model.debounce.500="searchQuery">
```

## Common Patterns

### Toggle Pattern

```html
<button $:click="isOpen = !isOpen">Toggle</button>
<div $:if="isOpen">Content</div>
```

### Filter Pattern

```html
<input $:model="searchQuery" placeholder="Search...">
<li $:for="user in users" $:if="user.name.includes(searchQuery)" $:text="user.name"></li>
```

### Disabled Button Pattern

```html
<button $:attr:disabled="!isValid || isLoading">Submit</button>
```

### Dynamic Class Pattern

```html
<div $:class="{ loading: isLoading, error: hasError, success: isSuccess }"></div>
```

## Summary

**Directives provide:**
- Declarative reactive bindings
- Clean, readable templates
- Automatic DOM updates
- Powerful expression evaluation
- Extensibility via custom directives

**Key directives to master:**
- `$:text`, `$:html` - Content
- `$:if`, `$:for` - Structure
- `$:model` - Forms
- `$:click`, `$:submit` - Events
- `$:class`, `$:style` - Styling
- `$:component` - Components

## Next Steps

- [DOM Binding](./dom-binding.md) - Text, attributes, classes, styles
- [Lists & Conditionals](./lists-and-conditionals.md) - `$:for` and `$:if`
- [Forms & Models](./forms-and-models.md) - Form binding
- [Events](./events.md) - Event handling
- [Directive Reference](../reference/directive-reference.md) - Complete API
- [Custom Directives](./custom-directives.md) - Create your own

