<!-- {
  "path": "jqr/guides/events",
  "name": "docs-jqr-guides-events",
  "meta": {
    "title": "Events",
    "order": 3
  }
} -->

# Events

JQR provides declarative event handling with `$:event` directives. No need for manual `addEventListener` or jQuery `.on()`.

## Basic Event Handling

Attach event handlers using `$:eventname` syntax:

```html live:no-code
<div id="basic-events" class="en-card">
  <div class="en-card__body">
    <h3>Basic Events</h3>
    <div class="en-flex en-gap-2">
      <button class="en-btn en-btn--primary" $:click="count++">
        Clicked: <span $:text="count"></span>
      </button>
      <button class="en-btn en-btn--outline" $:click="count = 0">Reset</button>
    </div>
  </div>
</div>
```

```html
<div id="basic-events">
  <button $:click="count++">
    Clicked: <span $:text="count"></span>
  </button>
  <button $:click="count = 0">Reset</button>
</div>
```

```js inject show
$('#basic-events').component({
  state: { count: 0 }
});
```

```js cleanup
onUnmount(() => {
  $('#basic-events').component('destroy');
})
```

## Supported Events

JQR automatically recognizes common events:

**Mouse Events:**
- `$:click` - Click
- `$:dblclick` - Double click
- `$:mouseenter` / `$:mouseleave` - Hover
- `$:mousedown` / `$:mouseup` - Mouse press
- `$:contextmenu` - Right click

**Keyboard Events:**
- `$:keydown` / `$:keyup` - Key press
- `$:keypress` - Character key

**Form Events:**
- `$:input` - Input value changed
- `$:change` - Input changed and blurred
- `$:submit` - Form submitted
- `$:focus` / `$:blur` - Focus events

**Other Events:**
- `$:scroll` - Scroll
- `$:resize` - Window resize
- `$:load` - Resource loaded

## Calling Methods

Call component methods from events:

```html live:no-code
<div id="methods-demo" class="en-card">
  <div class="en-card__body">
    <h3>Method Calls</h3>
    <div class="en-flex en-gap-2">
      <button class="en-btn en-btn--primary" $:click="increment">+1</button>
      <button class="en-btn en-btn--primary" $:click="decrement">-1</button>
      <button class="en-btn en-btn--outline" $:click="reset">Reset</button>
    </div>
    <div class="en-alert en-alert--info" style="margin-top: 1rem;">
      Count: <strong $:text="count"></strong>
    </div>
  </div>
</div>
```

```html
<div id="methods-demo">
  <button $:click="increment">+1</button>
  <button $:click="decrement">-1</button>
  <button $:click="reset">Reset</button>
  <div>Count: <strong $:text="count"></strong></div>
</div>
```

```js inject show
$('#methods-demo').component({
  state: { count: 0 },
  methods: {
    increment() {
      this.count++;
    },
    decrement() {
      this.count--;
    },
    reset() {
      this.count = 0;
    }
  }
});
```

```js cleanup
onUnmount(() => {
  $('#methods-demo').component('destroy');
})
```

### Method Shorthand

For bare method names (no parentheses), `$event` is passed automatically:

```html live:no-code
<div id="shorthand-methods-demo">
  <!-- These are equivalent: -->
  <div class="en-flex en-mb-3">
    <button class="en-btn" $:click="handleClick($event)">Click</button>
    <button class="en-btn" $:click="handleClick">Click (shorthand)</button>
  </div>
  <pre $:text="result"></pre>
</div>
```

```js inject
$('#shorthand-methods-demo').component({
  state: {
    result: null,
  },
  methods: {
    handleClick(event) {
      this.result = String(event.target.outerHTML)
    }
  }
})
```

```html
<!-- These are equivalent: -->
<button $:click="handleClick">Click</button>
<button $:click="handleClick($event)">Click</button>
```

```js
methods: {
  handleClick(ctx, event) {
    console.log('Clicked:', event.target);
  }
}
```

### Passing Arguments

Pass custom arguments to methods:

```html
<button $:click="addItem('apple')">Add Apple</button>
<button $:click="addItem('banana')">Add Banana</button>
```

```js
methods: {
  addItem(item) {
    this.items.push(item);
  }
}
```

## Event Object: `$event`

Access the native event object with `$event`:

```html live:no-code
<div id="event-object" class="en-card">
  <div class="en-card__body">
    <h3>Event Object</h3>
    <input 
      class="en-input" 
      $:keyup="lastKey = $event.key" 
      placeholder="Type something...">
    <div class="en-alert en-alert--info" style="margin-top: 1rem;">
      Last key pressed: <strong $:text="lastKey || 'None'"></strong>
    </div>
  </div>
</div>
```

```html
<div id="event-object">
  <input 
    $:keyup="lastKey = $event.key" 
    placeholder="Type something...">
  <div>Last key: <strong $:text="lastKey || 'None'"></strong></div>
</div>
```

```js inject show
$('#event-object').component({
  state: { lastKey: '' }
});
```

```js cleanup
onUnmount(() => {
  $('#event-object').component('destroy');
})
```

**Event properties:**
- `$event.target` - Element that triggered the event
- `$event.key` - Key pressed (keyboard events)
- `$event.clientX` / `$event.clientY` - Mouse position
- `$event.preventDefault()` - Prevent default action
- `$event.stopPropagation()` - Stop event bubbling

## Event Modifiers

Modifiers change event behavior without extra code:

### `.prevent` - Prevent Default

Calls `event.preventDefault()`:

```html
<form $:submit.prevent="handleSubmit">
  <input type="text">
  <button type="submit">Submit</button>
</form>
```

```js
methods: {
  handleSubmit(state) {
    // Form won't reload the page
    console.log('Form submitted');
  }
}
```

### `.stop` - Stop Propagation

Calls `event.stopPropagation()`:

```html
<div $:click="parentClick">
  <button $:click.stop="childClick">Click Me</button>
  <!-- Only childClick runs, parentClick is blocked -->
</div>
```

### `.once` - Fire Once

Event handler runs only once, then removes itself:

```html live:no-code
<div id="once-demo" class="en-card">
  <div class="en-card__body">
    <h3>Once Modifier</h3>
    <button class="en-btn en-btn--primary" $:click.once="count++">
      Click Once Only
    </button>
    <div class="en-alert en-alert--info" style="margin-top: 1rem;">
      Clicks: <strong $:text="count"></strong>
    </div>
    <p class="en-text--muted">Button only works once</p>
  </div>
</div>
```

```html
<div id="once-demo">
  <button $:click.once="count++">Click Once Only</button>
  <div>Clicks: <strong $:text="count"></strong></div>
</div>
```

```js inject show
$('#once-demo').component({
  state: { count: 0 }
});
```

```js cleanup
onUnmount(() => {
  $('#once-demo').component('destroy');
})
```

### `.capture` - Capture Mode

Uses capture phase instead of bubble phase:

```html
<div $:click.capture="handleCapture">
  <button $:click="handleClick">Click</button>
</div>
<!-- handleCapture runs BEFORE handleClick -->
```

### `.self` - Self Only

Only triggers if event target is the element itself (not children):

```html
<div $:click.self="handleSelf">
  Clicking here triggers the handler
  <button>But clicking this button does not</button>
</div>
```

### `.confirm` - Confirmation Dialog

Shows browser confirmation before executing:

```html live:no-code
<div id="confirm-demo" class="en-card">
  <div class="en-card__body">
    <h3>Confirm Modifier</h3>
    <button 
      class="en-btn en-btn--danger" 
      $:click.confirm="deleteItem"
      data-confirm-message="Are you sure?">
      Delete Item
    </button>
    <div class="en-alert en-alert--info" style="margin-top: 1rem;">
      Delete count: <strong $:text="deleteCount"></strong>
    </div>
  </div>
</div>
```

```html
<div id="confirm-demo">
  <button 
    $:click.confirm="deleteItem"
    data-confirm-message="Are you sure?">
    Delete Item
  </button>
  <div>Delete count: <strong $:text="deleteCount"></strong></div>
</div>
```

```js inject show
$('#confirm-demo').component({
  state: { deleteCount: 0 },
  methods: {
    deleteItem() {
      this.deleteCount++;
    }
  }
});
```

```js cleanup
onUnmount(() => {
  $('#confirm-demo').component('destroy');
})
```

Customize message with `data-confirm-message` attribute.

## Chaining Modifiers

Combine multiple modifiers:

```html
<form $:submit.prevent.stop="handleSubmit">
  <!-- Prevents default AND stops propagation -->
</form>

<button $:click.prevent.once="doOnce">
  <!-- Prevents default, runs once -->
</button>
```

**Order matters for some modifiers:**
```html
$:click.prevent.self  <!-- Prevents default only on self -->
$:click.self.prevent  <!-- Same as above -->
```

## Form Events

### Submit Event

```html live:no-code
<div id="form-submit" class="en-card">
  <div class="en-card__body">
    <h3>Form Submit</h3>
    <form $:submit.prevent="handleSubmit">
      <input 
        class="en-input" 
        $:model="username" 
        placeholder="Username"
        style="margin-bottom: 0.5rem;">
      <button class="en-btn en-btn--primary" type="submit" $:attr:disabled="username.length <= 3">Submit</button>
    </form>
    <div class="en-alert en-alert--success" style="margin-top: 1rem;" $:show="submitted">
      Submitted: <strong $:text="username"></strong>
    </div>
  </div>
</div>
```

```html
<div id="form-submit">
  <form $:submit.prevent="handleSubmit">
    <input $:model="username" placeholder="Username">
    <button type="submit" $:attr:disabled="username.length <= 3">Submit</button>
  </form>
  <div $:show="submitted">
    Submitted: <strong $:text="username"></strong>
  </div>
</div>
```

```js inject show
$('#form-submit').component({
  state: { 
    username: '',
    submitted: false
  },
  methods: {
    handleSubmit() {
      this.submitted = true;
      setTimeout(() => {
        this.submitted = false
        this.username = ''
      }, 3000);
    }
  }
});
```

```js cleanup
onUnmount(() => {
  $('#form-submit').component('destroy');
})
```

### Input Events

```html
<!-- Fires on every keystroke -->
<input $:input="handleInput">

<!-- Fires when input loses focus -->
<input $:change="handleChange">

<!-- Fires when user presses a key -->
<input $:keyup="handleKey">
```

## Keyboard Events

Access key information:

```html live:no-code
<div id="keyboard-demo" class="en-card">
  <div class="en-card__body">
    <h3>Keyboard Events</h3>
    <input 
      class="en-input" 
      $:keydown="handleKey($event)"
      placeholder="Press any key...">
    <div class="en-alert en-alert--info" style="margin-top: 1rem;">
      <div>Key: <strong $:text="lastKey"></strong></div>
      <div>Code: <strong $:text="lastCode"></strong></div>
      <div>Ctrl: <strong $:text="ctrlPressed"></strong></div>
    </div>
  </div>
</div>
```

```html
<div id="keyboard-demo">
  <input 
    $:keydown="handleKey($event)"
    placeholder="Press any key...">
  <div>
    <div>Key: <strong $:text="lastKey"></strong></div>
    <div>Code: <strong $:text="lastCode"></strong></div>
    <div>Ctrl: <strong $:text="ctrlPressed"></strong></div>
  </div>
</div>
```

```js inject show
$('#keyboard-demo').component({
  state: {
    lastKey: '',
    lastCode: '',
    ctrlPressed: false
  },
  methods: {
    handleKey(event) {
      this.lastKey = event.key;
      this.lastCode = event.code;
      this.ctrlPressed = event.ctrlKey;
    }
  }
});
```

```js cleanup
onUnmount(() => {
  $('#keyboard-demo').component('destroy');
})
```

**Common key checks:**
```js
if (event.key === 'Enter') { /* ... */ }
if (event.key === 'Escape') { /* ... */ }
if (event.ctrlKey && event.key === 's') { /* Save */ }
```

## Mouse Events

```html live:no-code
<div id="mouse-demo" class="en-card">
  <div class="en-card__body">
    <h3>Mouse Events</h3>
    <div 
      class="en-alert en-alert--info"
      style="cursor: pointer; user-select: none;"
      $:click="clicks++"
      $:dblclick="doubleClicks++"
      $:mouseenter="hovering = true"
      $:mouseleave="hovering = false">
      <div>Clicks: <strong $:text="clicks"></strong></div>
      <div>Double clicks: <strong $:text="doubleClicks"></strong></div>
      <div>Hovering: <strong $:text="hovering"></strong></div>
    </div>
  </div>
</div>
```

```html
<div id="mouse-demo">
  <div 
    $:click="clicks++"
    $:dblclick="doubleClicks++"
    $:mouseenter="hovering = true"
    $:mouseleave="hovering = false">
    <div>Clicks: <strong $:text="clicks"></strong></div>
    <div>Double clicks: <strong $:text="doubleClicks"></strong></div>
    <div>Hovering: <strong $:text="hovering"></strong></div>
  </div>
</div>
```

```js inject show
$('#mouse-demo').component({
  state: {
    clicks: 0,
    doubleClicks: 0,
    hovering: false
  }
});
```

```js cleanup
onUnmount(() => {
  $('#mouse-demo').component('destroy');
})
```

## Custom Events

Listen to custom jQuery events:

```html
<div $:myevent="handleCustom"></div>
```

```js
// Trigger from elsewhere
$('#element').trigger('myevent', [payload]);
```

## Complete Example: Todo App

```html live:no-code
<div id="todo-app" class="en-card">
  <div class="en-card__body">
    <h3>Todo List</h3>
    <form $:submit.prevent="addTodo" class="en-flex en-gap-2" style="margin-bottom: 1rem;">
      <input 
        class="en-input" 
        $:model="newTodo" 
        placeholder="New todo...">
      <button class="en-btn en-btn--primary" type="submit">Add</button>
    </form>
    <div $:for="todo in todos" $:key="todo.id" class="en-flex en-gap-2 en-items-center" style="margin-bottom: 0.5rem;">
      <input type="checkbox" $:model="todo.done">
      <span $:text="todo.text" $:style="{ textDecoration: todo.done ? 'line-through' : 'none' }"></span>
      <button class="en-btn en-btn--sm en-btn--danger" $:click="removeTodo(todo.id)">×</button>
    </div>
    <div class="en-text--muted" $:show="todos.length === 0">
      No todos yet. Add one above!
    </div>
  </div>
</div>
```

```html
<div id="todo-app">
  <h3>Todo List</h3>
  <form $:submit.prevent="addTodo">
    <input $:model="newTodo" placeholder="New todo...">
    <button type="submit">Add</button>
  </form>
  
  <div $:for="todo in todos" $:key="todo.id">
    <input type="checkbox" $:model="todo.done">
    <span 
      $:text="todo.text" 
      $:style="{ textDecoration: todo.done ? 'line-through' : 'none' }">
    </span>
    <button $:click="removeTodo(todo.id)">×</button>
  </div>
  
  <div $:show="todos.length === 0">
    No todos yet!
  </div>
</div>
```

```js inject show
$('#todo-app').component({
  state: {
    newTodo: '',
    todos: [],
    nextId: 1
  },
  methods: {
    addTodo() {
      if (!this.newTodo.trim()) return;
      this.todos.push({
        id: this.nextId++,
        text: this.newTodo,
        done: false
      });
      this.newTodo = '';
    },
    removeTodo(id) {
      this.todos = this.todos.filter(t => t.id !== id);
    }
  }
});
```

```js cleanup
onUnmount(() => {
  $('#todo-app').component('destroy');
})
```

## Best Practices

### 1. Use `.prevent` for Forms

Always prevent default on form submissions:

```html
<!-- ✅ Good -->
<form $:submit.prevent="handleSubmit">

<!-- ❌ Bad - page will reload -->
<form $:submit="handleSubmit">
```

### 2. Keep Event Handlers Simple

```js
// ✅ Good - simple inline expression
$:click="count++"

// ✅ Good - method call
$:click="increment"

// ❌ Bad - complex logic inline
$:click="count++; items.push(count); doSomethingElse()"
// Use a method instead
```

### 3. Use Methods for Reusable Logic

```js
// ✅ Good
methods: {
  handleClick(id) {
    this.items = this.items.filter(item => item.id !== id);
  }
}

// Multiple buttons can call the same method
// <button $:click="handleClick(item.id)"></button>
```

### 4. Avoid Memory Leaks

Event listeners are automatically cleaned up when:
- Component is destroyed
- Element is removed (e.g., from `$:for`)
- Modifier `.once` fires

No manual cleanup needed!

## Next Steps

- [Forms and Models](./forms-and-models.md) - Two-way data binding
- [Lists and Conditionals](./lists-and-conditionals.md) - Dynamic lists with `$:for`
- [Component Options](./component-options.md) - Full component API
