<!-- {
  "path": "jqr/guides/reactivity",
  "name": "docs-jqr-guides-reactivity",
  "meta": {
    "title": "Reactivity",
    "order": 1
  }
} -->

# Reactivity

JQR's reactivity system automatically tracks dependencies and updates the DOM when state changes. No manual DOM manipulation needed.

## How It Works

When you create a component with `state`, JQR wraps it in a **reactive proxy**. Any read inside a directive (like `$:text="count"`) is tracked, and when that value changes, the directive automatically re-runs.

```html live:no-code
<div id="reactivity-demo" class="en-card">
  <div class="en-card__body">
    <h3>Reactive Counter</h3>
    <div class="en-grid en-grid--cols-12 en-grid--responsive en-gap-3 en-items-center">
      <button class="en-btn en-btn--primary en-col-sm-6 en-col-md-3" $:click="count++">Increment</button>
      <div class="en-form-display en-col-sm-6 en-col-md-6">Count: <code $:text="count"></code></div>
      <button class="en-btn en-btn--outline en-col-sm-6 en-col-md-3" $:click="count = 0">Reset</button>
    </div>
    <p class="en-text--muted" style="margin-top: 1rem;">
      The count updates automatically when you click the button.
    </p>
  </div>
</div>
```

```html
<div id="reactivity-demo">
  <h3>Reactive Counter</h3>
  <button $:click="count++">Increment</button>
  <div>Count: <code $:text="count"></code></div>
  <button $:click="count = 0">Reset</button>
</div>
```

```js inject show
$('#reactivity-demo').component({
  state: { count: 0 }
});
```

```js cleanup
onUnmount(() => {
  $('#reactivity-demo').component('destroy');
})
```

## Reactive State

Define reactive state in the `state` option:

```js
$('#app').component({
  state: {
    message: 'Hello',
    count: 0,
    user: { name: 'Alice', age: 25 }
  }
});
```

**All properties** in `state` are automatically reactive:
- **Primitives**: `count`, `message`
- **Objects**: `user.name`, `user.age`
- **Arrays**: `items[0]`, `items.length`

### Nested Reactivity

Objects and arrays inside `state` are deeply reactive:

```html live:no-code
<div id="nested-demo" class="en-card">
  <div class="en-card__body">
    <h3>Nested Objects</h3>
    <div class="en-flex en-gap-2">
      <input class="en-input" $:model="user.name" placeholder="Name">
      <input class="en-input" $:model.number="user.age" type="number" placeholder="Age">
    </div>
    <div class="en-alert en-alert--info" style="margin-top: 1rem;">
      <strong $:text="user.name"></strong> is <span $:text="user.age"></span> years old.
    </div>
  </div>
</div>
```

```html
<div id="nested-demo">
  <h3>Nested Objects</h3>
  <input $:model="user.name" placeholder="Name">
  <input $:model.number="user.age" type="number" placeholder="Age">
  <div>
    <strong $:text="user.name"></strong> is 
    <span $:text="user.age"></span> years old.
  </div>
</div>
```

```js inject show
$('#nested-demo').component({
  state: {
    user: { name: 'Alice', age: 25 }
  }
});
```

```js cleanup
onUnmount(() => {
  $('#nested-demo').component('destroy');
})
```

## Computed Values

Computed values are derived from state and **cached** until their dependencies change:

```js
$('#app').component({
  state: {
    firstName: 'John',
    lastName: 'Doe'
  },
  computed: {
    fullName(c) {
      return `${c.firstName} ${c.lastName}`;
    }
  }
});
```

Use computed values in directives like regular state:

```html
<p $:text="fullName"></p>
<!-- Automatically updates when firstName or lastName changes -->
```

### Computed Example

```html live:no-code
<div id="computed-demo" class="en-card">
  <div class="en-card__body">
    <h3>Computed Properties</h3>
    <div class="en-flex en-gap-2">
      <input class="en-input" $:model="firstName" placeholder="First name">
      <input class="en-input" $:model="lastName" placeholder="Last name">
    </div>
    <div class="en-alert en-alert--success" style="margin-top: 1rem;">
      Full Name: <strong $:text="fullName"></strong>
    </div>
    <div class="en-alert en-alert--info">
      Initials: <strong $:text="initials"></strong>
    </div>
  </div>
</div>
```

```html
<div id="computed-demo">
  <h3>Computed Properties</h3>
  <input $:model="firstName" placeholder="First name">
  <input $:model="lastName" placeholder="Last name">
  <div>Full Name: <strong $:text="fullName"></strong></div>
  <div>Initials: <strong $:text="initials"></strong></div>
</div>
```

```js inject show
$('#computed-demo').component({
  state: {
    firstName: 'John',
    lastName: 'Doe'
  },
  computed: {
    fullName(c) {
      return `${c.firstName} ${c.lastName}`;
    },
    initials(c) {
      return `${c.firstName[0] || ''}${c.lastName[0] || ''}`.toUpperCase();
    }
  }
});
```

```js cleanup
onUnmount(() => {
  $('#computed-demo').component('destroy');
})
```

### Why Use Computed?

**Without computed** (recalculates every time):
```html
<p $:text="firstName + ' ' + lastName"></p>
<p $:text="firstName + ' ' + lastName"></p>
<!-- Expression runs twice, even if values didn't change -->
```

**With computed** (cached):
```html
<p $:text="fullName"></p>
<p $:text="fullName"></p>
<!-- Computed once, reused everywhere -->
```

## Methods

Methods are functions that can modify state:

```js
$('#app').component({
  state: { count: 0 },
  methods: {
    increment(c) {
      c.count++;
    },
    reset(c) {
      c.count = 0;
    }
  }
});
```

Call methods from directives:

```html
<button $:click="increment">+1</button>
<button $:click="reset">Reset</button>
```

### Methods vs Computed

| Feature | Methods | Computed |
|---------|---------|----------|
| Can modify state | ✅ Yes | ❌ No |
| Cached | ❌ No | ✅ Yes |
| Called from directives | ✅ `$:click="method()"` | ✅ `$:text="computed"` |
| Use case | Actions, events | Derived data |

## Reactivity Deep Dive

### What's Tracked?

JQR tracks:
- ✅ Property reads (`state.count`)
- ✅ Nested property reads (`state.user.name`)
- ✅ Array access (`state.items[0]`)
- ✅ Array length (`state.items.length`)

### What's Reactive?

These changes trigger updates:
- ✅ Direct assignment: `state.count = 5`
- ✅ Increment/decrement: `state.count++`
- ✅ Nested assignment: `state.user.name = 'Bob'`
- ✅ Array methods: `state.items.push(x)`, `state.items.splice(...)`

### Array Reactivity

All mutating array methods are reactive:

```js
state.items.push(item);      // ✅ Reactive
state.items.pop();           // ✅ Reactive
state.items.splice(0, 1);    // ✅ Reactive
state.items[0] = newItem;    // ✅ Reactive
state.items.sort();          // ✅ Reactive
```

Non-mutating methods need reassignment:

```js
state.items = state.items.filter(x => x.active); // ✅ Reactive
state.items.filter(x => x.active);               // ❌ Not reactive
```

### Calculator Example

```html live:no-code
<div id="calculator" class="en-card">
  <div class="en-card__body">
    <h3>Reactive Calculator</h3>
    <div class="en-flex en-gap-3">
      <div class="en-form-display en-form-display--ghost">
        Result: <code $:text="result"></code>
      </div>
      <div class="en-flex en-gap-2">
        <input class="en-input en-w-24" $:model.number="a" type="number">
        <select class="en-select en-w-auto" $:model="operator">
          <option value="+">+</option>
          <option value="-">-</option>
          <option value="*">×</option>
          <option value="/">÷</option>
        </select>
        <input class="en-input en-w-24" $:model.number="b" type="number">
      </div>
    </div>
  </div>
</div>
```

```html
<div id="calculator">
  <h3>Reactive Calculator</h3>
  <div>Result: <code $:text="result"></code></div>
  <input $:model.number="a" type="number">
  <select $:model="operator">
    <option value="+">+</option>
    <option value="-">-</option>
    <option value="*">×</option>
    <option value="/">÷</option>
  </select>
  <input $:model.number="b" type="number">
</div>
```

```js inject show
$('#calculator').component({
  state: {
    a: 10,
    b: 5,
    operator: '+'
  },
  computed: {
    result(ctx) {
      const { a, b, operator } = ctx.$state;
      switch (operator) {
        case '+': return a + b;
        case '-': return a - b;
        case '*': return a * b;
        case '/': return b !== 0 ? (a / b).toFixed(2) : 'Error';
        default: return 0;
      }
    }
  }
});
```

```js cleanup
onUnmount(() => {
  $('#calculator').component('destroy');
})
```

## Advanced: Manual Reactivity

For advanced use cases, JQR exposes low-level reactivity primitives:

```js
import { reactive, effect } from '@en32/jqr';

const state = reactive({ count: 0 });

effect(() => {
  console.log('Count is:', state.count);
});

state.count++; // Logs: "Count is: 1"
```

**Most apps don't need this** - the component system handles it automatically.

## Performance Tips

### 1. Use Computed for Expensive Operations

```js
// ❌ Recalculates every time
$:text="items.filter(x => x.active).length"

// ✅ Cached until items changes
computed: {
  activeCount(ctx) {
    return ctx.state.items.filter(x => x.active).length;
  }
}
```

### 2. Avoid Deep Nesting

```js
// ❌ Slower
state.data.users.items[0].profile.settings.theme = 'dark';

// ✅ Faster
const settings = state.data.users.items[0].profile.settings;
settings.theme = 'dark';
```

### 3. Batch Updates

Updates are automatically batched within the same synchronous execution:

```js
methods: {
  updateMultiple(state) {
    state.count++;      // }
    state.message = 'x'; // } All three updates
    state.items.push(1); // } render once
  }
}
```

## Next Steps

- [DOM Binding](./dom-binding.md) - Bind reactive data to the DOM
- [Events](./events.md) - Handle user interactions
- [Component Options](./component-options.md) - Full component API
