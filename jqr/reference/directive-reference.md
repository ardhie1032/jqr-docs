<!-- {
  "path": "jqr/reference/directive-reference",
  "name": "docs-jqr-reference-directive-reference",
  "meta": {
    "title": "Directive Reference",
    "order": 1
  }
} -->

# Directive Reference

Complete API reference for all built-in JQR directives. This is a quick lookup guide - for detailed examples and patterns, see the [Directives Overview](../guides/directives.md).

## Directive Syntax

```html
<element $:directive="expression"></element>
<element $:directive:argument="expression"></element>
<element $:directive.modifier1.modifier2="expression"></element>
```

## Directive Arguments

Many directives support arguments in the form `$:name:arg`. Here's a complete list of recognized patterns:

| Directive | Purpose | Example |
|-----------|---------|---------|
| `$:attr:<name>` | Bind any HTML attribute | `<img $:attr:src="url">` |
| `$:on:<event>` | Listen to any DOM event | `<div $:on:mouseenter="hover()">` |
| `$:model:<prop>` | Two-way bind to property | `<dialog $:model:open="isOpen">` |
| `$:prop:<key>` | Pass prop to component | `<div $:component="card" $:prop:user-id="id">` |
| `$:component:is` | Dynamic component | `<div $:component:is="currentView">` |
| `$:component:<key>` | Inline component option | `<div $:component:state="{ count: 0 }">` |
| `$:component:slot` | Scoped slot alias | `<template $:component:slot="data">` |
| `$:teleport:to` | Teleport target selector | `<div $:teleport:to="#modal">` |
| `$:scope:<key>` | Scoped slot value | `<slot $:scope:user="currentUser">` |

### Important: Attributes vs Directives

**Most HTML attributes are NOT directives.** You must use `$:attr:` prefix:

```html
<!-- ❌ WRONG - These trigger "unknownDirective" warning -->
<input $:placeholder="hint">
<button $:disabled="!valid">
<a $:href="link">

<!-- ✅ CORRECT - Use $:attr: -->
<input $:attr:placeholder="hint">
<button $:attr:disabled="!valid">
<a $:attr:href="link">
```

**Exception:** Some common attributes have dedicated directives:
- `$:value`, `$:checked` (form elements)
- `$:class`, `$:style` (styling)
- See full list below

---

## Content Directives

### `$:text`

Sets the text content of an element.

**Syntax:**
```html
<element $:text="expression"></element>
```

**Type:** `string | number | any` (auto-converted to string)

**Example:**
```html
<p $:text="message"></p>
<span $:text="count"></span>
<div $:text="`Hello, ${username}`"></div>
```

**Escaping:** Automatically escapes HTML (XSS-safe)

**See:** [DOM Binding Guide](../guides/dom-binding.md#text-content)

---

### `$:html`

Sets the inner HTML of an element.

**Syntax:**
```html
<element $:html="expression"></element>
```

**Type:** `string`

**Example:**
```html
<div $:html="richContent"></div>
<div $:html="'<strong>Bold text</strong>'"></div>
```

**⚠️ Security Warning:** Only use with trusted content. Vulnerable to XSS attacks if used with user input.

**See:** [DOM Binding Guide](../guides/dom-binding.md#html-content)

---

## Attribute Directives

### `$:attr:*`

Sets any HTML attribute dynamically.

**Syntax:**
```html
<element $:attr:attributeName="expression"></element>
```

**Type:** `string | number | boolean | null`

**Boolean Attributes:**
- `true` → attribute with empty value
- `false` / `null` / `undefined` → attribute removed

**Examples:**
```html
<img $:attr:src="imageUrl">
<a $:attr:href="linkUrl">
<button $:attr:disabled="isDisabled">
<input $:attr:placeholder="placeholderText">
<div $:attr:id="elementId">
<div $:attr:data-id="userId">
```

**⚠️ Important:** `$:attr:` is required for most HTML attributes. Direct usage like `$:placeholder` or `$:disabled` will NOT work (they are not directives and will trigger `unknownDirective` warning).

**When to use:**
- Any standard HTML attribute: `href`, `src`, `alt`, `title`, `placeholder`, `readonly`, etc.
- Data attributes: `data-*`
- ARIA attributes: `aria-*`
- Custom attributes

**Exceptions (have dedicated directives):**
- `$:value`, `$:checked` - Use these instead of `$:attr:value` / `$:attr:checked`
- `$:class`, `$:style` - Use these for styling

**Special Cases:**
```html
<!-- Boolean attributes -->
<input $:attr:disabled="true">     <!-- disabled="" -->
<input $:attr:disabled="false">    <!-- attribute removed -->

<!-- Multiple attributes -->
<img $:attr:src="url" $:attr:alt="description" $:attr:width="size">
```

**See:** [DOM Binding Guide](../guides/dom-binding.md#attributes)

---

### `$:class`

Binds CSS classes dynamically.

**Syntax:**
```html
<element $:class="expression"></element>
```

**Type:** `string | Array | Object`

**String:**
```html
<div $:class="'card primary'"></div>
<div $:class="cardClass"></div>
```

**Array:**
```html
<div $:class="['card', activeClass]"></div>
<div $:class="[baseClass, { active: isActive }]"></div>
```

**Object:**
```html
<div $:class="{ active: isActive, disabled: !isEnabled }"></div>
<div $:class="{ 'card-large': isLarge, 'card-small': !isLarge }"></div>
```

**Combined with Static:**
```html
<div class="card" $:class="{ active: isActive }"></div>
<!-- Result: class="card active" (when isActive is true) -->
```

**See:** [DOM Binding Guide](../guides/dom-binding.md#class-binding)

---

### `$:style`

Binds inline styles dynamically.

**Syntax:**
```html
<element $:style="expression"></element>
```

**Type:** `Object | string`

**Object (camelCase):**
```html
<div $:style="{ backgroundColor: bgColor, fontSize: '14px' }"></div>
<div $:style="{ color: textColor, padding: spacing + 'px' }"></div>
```

**Object (kebab-case):**
```html
<div $:style="{ 'background-color': bgColor, 'font-size': '14px' }"></div>
```

**String:**
```html
<div $:style="'color: red; padding: 10px;'"></div>
<div $:style="customStyles"></div>
```

**Auto Units:**
```html
<!-- These properties auto-append 'px' when number provided -->
<div $:style="{ width: 200, height: 100, padding: 20 }"></div>
<!-- Result: width: 200px; height: 100px; padding: 20px -->
```

**See:** [DOM Binding Guide](../guides/dom-binding.md#style-binding)

---

### `$:bind`

Multi-attribute shorthand binding.

**Syntax:**
```html
<element $:bind="objectExpression"></element>
<element $:bind:attributeName="expression"></element>
```

**Type:** `Object`

**Object Mode:**
```html
<div $:bind="{ 
  text: message, 
  class: { active: isActive },
  style: { color: textColor },
  attr: { id: elementId }
}"></div>
```

**Sub-Attribute Mode:**
```html
<div $:bind:class="{ active: isActive }"></div>
<div $:bind:style="{ color: textColor }"></div>
<div $:bind:text="message"></div>
```

**Equivalent to:**
```html
<div $:text="message" $:class="{ active: isActive }"></div>
```

**See:** [Bind Directive Guide](../guides/bind-directive.md)

---

## Conditional Directives

### `$:if`

Conditionally renders element (removes from DOM when false).

**Syntax:**
```html
<element $:if="condition"></element>
```

**Type:** `boolean | any` (truthy/falsy)

**Examples:**
```html
<div $:if="isVisible">Conditionally rendered</div>
<p $:if="count > 0">Count: <span $:text="count"></span></p>
<div $:if="user">Welcome, <span $:text="user.name"></span></div>
```

**With `<template>`:**
```html
<template $:if="isLoggedIn">
  <div>Header</div>
  <div>Content</div>
  <div>Footer</div>
</template>
```

**Behavior:**
- `true` / truthy → Element inserted into DOM
- `false` / falsy → Element removed from DOM
- Re-creation on toggle (not just hidden)

**See:** [DOM Binding Guide](../guides/dom-binding.md#conditional-rendering), [Lists & Conditionals](../guides/lists-and-conditionals.md)

---

### `$:show`

Toggles element visibility (uses `display: none`).

**Syntax:**
```html
<element $:show="condition"></element>
```

**Type:** `boolean | any` (truthy/falsy)

**Examples:**
```html
<div $:show="isVisible">Toggle me</div>
<p $:show="hasError">Error message</p>
```

**Behavior:**
- `true` / truthy → `display` restored to original value
- `false` / falsy → `display: none`
- Element stays in DOM

**vs `$:if`:**
- `$:if` → Removes/creates DOM nodes (expensive, destroys state)
- `$:show` → Toggles CSS (cheap, preserves state)

**See:** [DOM Binding Guide](../guides/dom-binding.md#visibility)

---

## List Directives

### `$:for`

Renders a list of elements by iterating over an array.

**Syntax:**
```html
<element $:for="item in array"></element>
<element $:for="(item, index) in array"></element>
<element $:for="item in count"></element>
```

**Type:** `Array | Number | Object`

**Array Iteration:**
```html
<li $:for="fruit in fruits" $:text="fruit"></li>
<div $:for="user in users">
  <p $:text="user.name"></p>
</div>
```

**With Index:**
```html
<li $:for="(item, index) in items">
  <span $:text="index + 1"></span>. <span $:text="item"></span>
</li>
```

**Number Range:**
```html
<span $:for="n in 5" $:text="n"></span>
<!-- Renders: 0 1 2 3 4 -->
```

**Object Iteration:**
```html
<div $:for="(value, key) in user">
  <strong $:text="key"></strong>: <span $:text="value"></span>
</div>
```

**With `<template>`:**
```html
<template $:for="item in items">
  <div>Header: <span $:text="item"></span></div>
  <div>Body content</div>
</template>
```

**⚠️ Always use with `$:key`** for efficient updates.

**See:** [Lists & Conditionals Guide](../guides/lists-and-conditionals.md)

---

### `$:key`

Provides a unique key for list items to optimize rendering.

**Syntax:**
```html
<element $:for="item in items" $:key="uniqueValue"></element>
```

**Type:** `string | number`

**Examples:**
```html
<li $:for="user in users" $:key="user.id" $:text="user.name"></li>
<div $:for="item in items" $:key="item.uuid"></div>
```

**When to Use:**
- Always use with `$:for` when items have unique IDs
- When list order can change
- When items can be added/removed

**When NOT to Use:**
- Static lists that never change
- Index as key (defeats the purpose)

**See:** [Lists & Conditionals Guide](../guides/lists-and-conditionals.md#list-keys)

---

## Form Directives

### `$:model`

Two-way data binding for form inputs.

**Syntax:**
```html
<input $:model="stateProperty">
<input $:model.modifier1.modifier2="stateProperty">
```

**Type:** Depends on input type

**Supported Elements:**
- `<input type="text|email|url|tel|search|password">`
- `<input type="number">`
- `<input type="checkbox">`
- `<input type="radio">`
- `<textarea>`
- `<select>`

**Text Input:**
```html
<input $:model="username" type="text">
<!-- Updates state.username on input -->
```

**Checkbox (Boolean):**
```html
<input $:model="isAgreed" type="checkbox">
<!-- state.isAgreed = true/false -->
```

**Checkbox (Array):**
```html
<input $:model="selectedItems" type="checkbox" value="item1">
<input $:model="selectedItems" type="checkbox" value="item2">
<!-- state.selectedItems = ['item1', 'item2'] -->
```

**Radio:**
```html
<input $:model="gender" type="radio" value="male">
<input $:model="gender" type="radio" value="female">
<!-- state.gender = 'male' or 'female' -->
```

**Select:**
```html
<select $:model="country">
  <option value="us">United States</option>
  <option value="uk">United Kingdom</option>
</select>
```

**Textarea:**
```html
<textarea $:model="message"></textarea>
```

**Available Modifiers:**
- `.trim` - Trim whitespace
- `.number` - Parse as number
- `.lazy` - Update on `change` instead of `input`
- `.debounce.N` - Debounce input (N milliseconds)
- `.throttle.N` - Throttle input (N milliseconds)

**See:** [Forms & Models Guide](../guides/forms-and-models.md), [Model Modifiers Reference](./modifier-reference.md)

---

### `$:model:*` (Mode B)

Two-way binding to a DOM element property.

**Syntax:**
```html
<element $:model:propertyName="stateProperty">
<element $:model:propertyName.on-eventName="stateProperty">
```

**Purpose:** Bind directly to a DOM property (not an attribute) on native elements like `<dialog>` or `<details>`.

**Common Use Cases:**

```html
<!-- Dialog open state -->
<dialog $:model:open="dialogOpen">
  <p>Dialog content</p>
  <button $:click="dialogOpen = false">Close</button>
</dialog>

<!-- Details open state -->
<details $:model:open="detailsOpen">
  <summary>Click to expand</summary>
  <p>Hidden content</p>
</details>
```

**How it works:**
1. Reads the initial value from `element[property]`
2. Updates state when the property changes
3. Updates the property when state changes

**Event Trigger:**
- Modifier `.on-<event>` - Use specific event (e.g., `.on-toggle`)
- Preset per tag - Built-in presets for `<dialog>` and `<details>` (event: `toggle`)
- Fallback - Uses `change` event (with dev warning if no preset)

**Custom Presets:**
```js
// Register custom model preset
$.jqr.registerModelPreset('my-element', {
  prop: 'isOpen',
  event: 'toggle'
});
```

**Component Mode (Mode A):**
On `$:component` host elements, `$:model:<prop>` is sugar for:
```html
<!-- These are equivalent -->
<div $:component="modal" $:model:open="isOpen">
<div $:component="modal" $:prop:open="isOpen" $:on:update:open="isOpen = $event">
```

**See:** [Model Presets Guide](../guides/model-presets.md)

---

## Event Directives

### `$:on:*`

Generic event handler (supports any DOM event).

**Syntax:**
```html
<element $:on:eventName="handler"></element>
<element $:on:eventName.modifier="handler"></element>
```

**Purpose:** Listen to any DOM event, including those without dedicated shorthand directives.

**Common Use Cases:**
```html
<!-- Mouse events -->
<div $:on:mouseenter="isHovered = true">
<div $:on:mouseleave="isHovered = false">
<div $:on:mousemove="trackMouse($event)">

<!-- Keyboard events -->
<input $:on:keydown="handleKeyDown($event)">
<input $:on:keyup="handleKeyUp($event)">

<!-- Focus events -->
<input $:on:focus="isFocused = true">
<input $:on:blur="isFocused = false">

<!-- Animation/Transition events -->
<div $:on:animationend="onAnimationEnd()">
<div $:on:transitionend="onTransitionEnd()">

<!-- Touch events -->
<div $:on:touchstart="handleTouch($event)">
<div $:on:touchmove="handleMove($event)">
```

**Component Events:**
On `$:component` host elements, `$:on:` listens to custom events emitted by child components:

```html
<!-- Parent template -->
<div $:component="child" $:on:save="handleSave($event)">

<!-- Child component -->
<button $:click="$emit('save', data)">Save</button>
```

**Colon in Event Names:**
Arguments can contain colons for namespaced events:

```html
<!-- Listen to "update:open" event -->
<div $:component="modal" $:on:update:open="isOpen = $event">
```

**Event Object:**
```html
<input $:on:keyup="handleKeyup($event)">
```
```js
methods: {
  handleKeyup(state, event) {
    console.log(event.key, event.target.value);
  }
}
```

**Supported Modifiers:** Same as shorthand directives (`.prevent`, `.stop`, `.once`, `.capture`, `.self`, `.confirm`)

**Examples:**
```html
<div $:on:click="handleClick()"></div>
<input $:on:input="handleInput($event)">
<form $:on:submit.prevent="handleSubmit()">
<div $:on:mouseenter="isHovered = true">
<div $:on:mouseleave="isHovered = false">
```

**See:** [Events Guide](../guides/events.md)

---

### `$:click`

Shorthand for click events.

**Syntax:**
```html
<element $:click="expression"></element>
```

**Examples:**
```html
<button $:click="count++">Increment</button>
<button $:click="save()">Save</button>
<a $:click.prevent="handleLink($event)">Link</a>
```

**Equivalent to:** `$:on:click`

---

### `$:input`

Shorthand for input events.

**Syntax:**
```html
<input $:input="expression">
```

**Example:**
```html
<input $:input="handleTyping($event)">
```

**Equivalent to:** `$:on:input`

---

### `$:change`

Shorthand for change events.

**Syntax:**
```html
<input $:change="expression">
```

**Example:**
```html
<select $:change="handleChange()">
```

**Equivalent to:** `$:on:change`

---

### `$:submit`

Shorthand for form submit events.

**Syntax:**
```html
<form $:submit="expression"></form>
```

**Example:**
```html
<form $:submit.prevent="handleSubmit()">
  <!-- form fields -->
</form>
```

**Common Modifiers:**
- `.prevent` - Prevent default (no page reload)

**Equivalent to:** `$:on:submit`

---

### Event Modifiers

**Available on all event directives:**

| Modifier | Description |
|----------|-------------|
| `.prevent` | Calls `event.preventDefault()` |
| `.stop` | Calls `event.stopPropagation()` |
| `.self` | Only trigger if event.target is the element itself |
| `.once` | Trigger handler only once |
| `.capture` | Use capture mode for event listener |
| `.passive` | Adds passive event listener (improves scroll perf) |

**Key Modifiers (keyboard events):**
- `.enter`, `.tab`, `.delete`, `.esc`, `.space`
- `.up`, `.down`, `.left`, `.right`
- `.ctrl`, `.alt`, `.shift`, `.meta`

**Mouse Modifiers:**
- `.left`, `.right`, `.middle`

**Examples:**
```html
<form $:submit.prevent="onSubmit()">
<button $:click.once="init()">
<div $:click.self="handleSelf()">
<input $:keyup.enter="search()">
<input $:keyup.ctrl.s.prevent="save()">
```

**See:** [Events Guide](../guides/events.md), [Modifier Reference](./modifier-reference.md)

---

## Component Directives

### `$:component`

Renders a registered component by name.

**Syntax:**
```html
<element $:component="componentName"></element>
```

**Type:** `string` (component registry name)

**Example:**
```html
<div $:component="user-card"></div>
<div $:component="todo-item"></div>
```

**Register component:**
```js
$.jqr.register('user-card', {
  props: { userId: null },
  template: '<div>...</div>'
});
```

**See:** [Component Options Guide](../guides/component-options.md#components)

---

### `$:component:*`

Inline component definition or attribute.

**Syntax:**
```html
<element $:component:state="stateObject"></element>
<element $:component:is="componentObject"></element>
<element $:component:slot="scopeExpression"></element>
```

**State Shorthand:**
```html
<div $:component:state="{ count: 0 }">
  <p $:text="count"></p>
  <button $:click="count++">+</button>
</div>
```

**Inline Definition:**
```html
<div $:component:is="{ 
  state: { message: 'Hello' },
  methods: { greet() { alert(this.state.message); } }
}">
  <button $:click="greet()">Greet</button>
</div>
```

**Slot Shorthand:**
```html
<div $:component="user-list" $:component:slot="{ users, count }">
  <li $:for="user in users" $:text="user.name"></li>
</div>
```

**See:** [Slots Guide](../guides/slots.md#shorthand)

---

### `$:prop:*`

Passes props to child components.

**Syntax:**
```html
<element $:component="name" $:prop:propName="expression"></element>
```

**Examples:**
```html
<div $:component="user-card" $:prop:userId="currentUserId"></div>
<div $:component="post" 
     $:prop:title="postTitle" 
     $:prop:content="postContent"></div>
```

**Dynamic Props:**
```html
<div $:component="card" $:prop:data="userData"></div>
```

**See:** [Props Guide](../guides/props.md)

---

## Slot Directives

### `$:slot`

Targets a named slot in parent component.

**Syntax:**
```html
<template $:slot="slotName"></template>
```

**Example:**
```html
<div $:component="card">
  <template $:slot="header">
    <h3>Card Title</h3>
  </template>
  
  <template $:slot="default">
    <p>Card content</p>
  </template>
</div>
```

**Default Slot:**
```html
<!-- Without $:slot, goes to default slot -->
<div $:component="card">
  <p>This goes to default slot</p>
</div>
```

**See:** [Slots Guide](../guides/slots.md)

---

### `$:scope`

Receives scoped slot data from parent.

**Syntax:**
```html
<template $:slot="name" $:scope="{ prop1, prop2 }"></template>
<template $:slot="name" $:scope="scopedData"></template>
```

**Destructuring:**
```html
<template $:slot="item" $:scope="{ user, index }">
  <p><span $:text="index"></span>: <span $:text="user.name"></span></p>
</template>
```

**Single Variable:**
```html
<template $:slot="item" $:scope="itemData">
  <p $:text="itemData.title"></p>
</template>
```

**See:** [Slots Guide](../guides/slots.md#scoped-slots)

---

### `$:default`

Marks a default slot outlet in component template.

**Syntax:**
```html
<slot $:default></slot>
<slot $:default>Fallback content</slot>
```

**Example:**
```html
<template id="card">
  <div class="card">
    <slot $:default>
      <p>Default card content</p>
    </slot>
  </div>
</template>
```

**See:** [Slots Guide](../guides/slots.md#default-slot)

---

### `$:name`

Marks a named slot outlet in component template.

**Syntax:**
```html
<slot $:name="slotName"></slot>
```

**Example:**
```html
<template id="layout">
  <header>
    <slot $:name="header"></slot>
  </header>
  <main>
    <slot $:default></slot>
  </main>
  <footer>
    <slot $:name="footer"></slot>
  </footer>
</template>
```

**See:** [Slots Guide](../guides/slots.md#named-slots)

---

### `$:scope:*`

Provides scoped data from child to parent slot (kebab-case).

**Syntax:**
```html
<slot $:name="item" $:scope:variable-name="expression"></slot>
```

**Example:**
```html
<slot $:name="item" 
      $:scope:user="currentUser" 
      $:scope:index="itemIndex"></slot>
```

**Kebab-case → camelCase:**
```html
<!-- Child defines with kebab-case -->
<slot $:scope:user-name="user.name"></slot>

<!-- Parent receives as camelCase -->
<template $:scope="{ userName }">
  <span $:text="userName"></span>
</template>
```

**See:** [Slots Guide](../guides/slots.md#scoped-slots)

---

## Special Directives

### `$:ref`

Creates a reference to DOM element accessible via `this.$refs`.

**Syntax:**
```html
<element $:ref="refName"></element>
```

**Value:** Ref name is **literal** (not evaluated as JS expression), same convention as `$:teleport:to.*`.

**Result Type:**

| Location | `this.$refs.name` Type |
|----------|------------------------|
| Outside `$:for` | `JQuery` — single object |
| Inside `$:for` | `JQuery[]` — array, one per item |

**Single Ref Examples:**
```html
<!-- Access via this.$refs.usernameInput -->
<input $:ref="usernameInput" type="text">

<!-- Access via this.$refs.submitButton -->
<button $:ref="submitButton">Submit</button>

<!-- Access via this.$refs.chartCanvas -->
<canvas $:ref="chartCanvas"></canvas>
```

**Array Ref Example (inside $:for):**
```html
<ul>
  <li $:for="item in items" $:key="item.id" $:ref="row">
    {{ item.name }}
  </li>
</ul>

<!-- Access via this.$refs.row[0], this.$refs.row[1], ... -->
```

**Usage in Component:**
```js
mounted() {
  // Single ref
  this.$refs.usernameInput.trigger('focus');
  
  // Array ref
  this.$refs.row.forEach(($li, i) => {
    console.log(i, $li.text());
  });
  
  // Raw DOM element (for third-party libraries)
  const canvas = this.$refs.chartCanvas[0];
  new Chart(canvas, { /* config */ });
}
```

**When Available:**
- ✅ Inside `mounted()` hook and later
- ❌ Not in `constructor()` or `state` initializer

**Use Cases:**
- Focus management
- Measuring element dimensions
- Integrating third-party libraries
- Scroll to element
- Direct DOM manipulation

**Don't Use For:**
- Form values (use `$:model`)
- Text content (use `$:text`)
- CSS classes (use `$:class`)
- Show/hide (use `$:if` or `$:show`)

**See:** [Element References Guide](../guides/ref.md)

---

### `$:teleport`

Moves element to different location in DOM (portal).

**Two Forms:**

1. **Object Form** (reactive):
```html
<element $:teleport="expression"></element>
```

2. **Arg Form** (literal, simpler):
```html
<element $:teleport:to[.modifiers]="target"></element>
```

**Object Form Examples:**
```html
<!-- String target -->
<div $:teleport="'body'">
  <div class="modal">Modal</div>
</div>

<!-- Object with options -->
<div $:teleport="{ to: '#modal-root', mode: 'prepend' }">
  <div class="modal">Modal</div>
</div>

<!-- With defer (waits for target) -->
<div $:teleport="{ to: '#late-target', defer: true }">
  <div class="content">Content</div>
</div>

<!-- Reactive target -->
<div $:teleport="state.targetPortal">
  <div class="dynamic">Dynamic portal</div>
</div>
```

**Arg Form Examples:**
```html
<!-- Basic -->
<div $:teleport:to="body">Toast</div>
<div $:teleport:to="#modal-root">Modal</div>

<!-- With mode modifiers -->
<div $:teleport:to.prepend="body">Prepend</div>
<div $:teleport:to.append="#portal">Append (default)</div>
<div $:teleport:to.insert-after="#header">Insert after</div>
<div $:teleport:to.insert-before="#footer">Insert before</div>

<!-- With defer modifier (waits for target) -->
<div $:teleport:to.defer="#dynamic-target">Waits</div>
<div $:teleport:to.defer.prepend="body">Defer + prepend</div>
```

**Modes:**
| Mode | Object | Arg Modifier | Behavior |
|------|--------|--------------|----------|
| Append | `mode: 'append'` | `.append` (default) | Add to end of target |
| Prepend | `mode: 'prepend'` | `.prepend` | Add to start of target |
| Insert After | `mode: 'insertAfter'` | `.insert-after` | After target element |
| Insert Before | `mode: 'insertBefore'` | `.insert-before` | Before target element |

**Defer Option:**
- **Without defer:** Target must exist now, fails immediately if missing
- **With defer:** Uses `MutationObserver` to wait for target to appear
- Useful when target is rendered conditionally or loaded async

**Target:**
- Any CSS selector string
- Object form: Evaluated as JS expression (can be reactive)
- Arg form: Treated as literal string (not reactive)

**Behavior:**
- Element moved to target location
- Stays reactive to component state
- Auto cleanup on component destroy
- Defer observers auto-cleanup too

**See:** [Teleport Guide](../guides/teleport.md)

---

### `$:transition`

Applies CSS transition/animation effects.

**Syntax:**
```html
<element $:transition="transitionName"></element>
<element $:transition="{ name: 'fade', mode: 'out-in' }"></element>
```

**Simple:**
```html
<div $:if="isVisible" $:transition="fade">
  Content with fade transition
</div>
```

**With Options:**
```html
<div $:if="isVisible" $:transition="{ 
  name: 'slide',
  duration: 500,
  mode: 'out-in'
}">
  Animated content
</div>
```

**CSS Classes Applied:**
- `{name}-enter-from` - Enter start
- `{name}-enter-active` - Enter duration
- `{name}-enter-to` - Enter end
- `{name}-leave-from` - Leave start
- `{name}-leave-active` - Leave duration
- `{name}-leave-to` - Leave end

**See:** [Transitions Guide](../guides/transitions.md)

---

## Custom Directives

Create your own directives with `$.jqr.directive()`:

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

**See:** [Custom Directives Guide](../guides/custom-directives.md)

---

## Directive Priority

When multiple directives are on same element:

1. `$:for` - Creates elements
2. `$:if` / `$:show` - Controls visibility
3. `$:component` - Initializes component
4. `$:bind` - Multi-attribute
5. Other directives - No specific order
6. Events - Attached last

**Example:**
```html
<!-- Correct order -->
<div $:for="item in items" 
     $:if="item.visible" 
     $:class="{ active: item.active }"
     $:click="select(item)">
</div>
```

---

## Configuration

Change directive prefix:

```js
$.jqr.config({
  prefix: 'v-'  // Use Vue-style
});
```

```html
<p v-text="message"></p>
<div v-if="isVisible"></div>
```

**See:** [Prefixes & Config Guide](../guides/prefixes-and-config.md)

---

## Quick Reference Table

| Category | Directives |
|----------|-----------|
| **Content** | `$:text`, `$:html` |
| **Attributes** | `$:attr:*`, `$:class`, `$:style`, `$:bind` |
| **Conditionals** | `$:if`, `$:show` |
| **Lists** | `$:for`, `$:key` |
| **Forms** | `$:model` |
| **Events** | `$:on:*`, `$:click`, `$:input`, `$:change`, `$:submit` |
| **Components** | `$:component`, `$:component:*`, `$:prop:*` |
| **Slots** | `$:slot`, `$:scope`, `$:default`, `$:name`, `$:scope:*` |
| **Special** | `$:ref`, `$:teleport`, `$:transition` |

---

## Next Steps

- [Modifier Reference](./modifier-reference.md) - All directive modifiers
- [Directives Overview](../guides/directives.md) - Detailed guide with examples
- [Events Guide](../guides/events.md) - Event handling patterns
- [Forms & Models](../guides/forms-and-models.md) - Form binding details
- [Lists & Conditionals](../guides/lists-and-conditionals.md) - List rendering patterns
