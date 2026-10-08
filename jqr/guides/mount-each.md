<!-- {
  "path": "jqr/guides/mount-each",
  "name": "docs-jqr-guides-mount-each",
  "meta": {
    "title": "Multiple Instances",
    "order": 16
  }
} -->

# Multiple Instances (mountEach)

`$.jqr.mountEach()` mounts components on multiple elements at once, with support for shared or independent state. Use it for widgets repeated across a page.

## The Problem

Using `$('#el').component()` only binds the **first** matching element:

```html
<!-- Three calculator widgets -->
<div class="calculator"></div>
<div class="calculator"></div>
<div class="calculator"></div>

<script>
  // ❌ Only binds first element!
  $('.calculator').component({
    state: { a: 0, b: 0 }
  });
</script>
```

**Manual loop approach:**
```js
// ❌ Bad: Creates shared state accidentally
const options = { state: { a: 0, b: 0 } };
$('.calculator').each((i, el) => {
  $(el).component(options); // All share same state object!
});
```

## The Solution: `mountEach()`

```typescript
$.jqr.mountEach(
  selector: string | HTMLElement | ArrayLike<HTMLElement>,
  options: ComponentOptions | MountEachOptionsFactory,
  config?: { template?: string, independent?: boolean }
): JqrComponent[]
```

### Basic Usage - Shared State

By default, all instances **share the same state**:

```html live:no-code
<div id="demo-shared" class="en-card">
  <div class="en-card__body">
    <h3>Shared State Example</h3>
    <p class="en-text--muted">Both counters share the same count:</p>
    <div class="en-grid en-grid--cols-12 en-gap-3">
      <div class="en-col-12 en-col-md-6">
        <div class="shared-counter en-card">
          <div class="en-card__body">
            <h4>Counter A</h4>
            <p>Count: <strong $:text="count"></strong></p>
            <button class="en-btn en-btn--sm en-btn--primary" $:click="increment">+1</button>
          </div>
        </div>
      </div>
      <div class="en-col-12 en-col-md-6">
        <div class="shared-counter en-card">
          <div class="en-card__body">
            <h4>Counter B</h4>
            <p>Count: <strong $:text="count"></strong></p>
            <button class="en-btn en-btn--sm en-btn--primary" $:click="increment">+1</button>
          </div>
        </div>
      </div>
    </div>
    <p class="en-text--muted en-text-sm" style="margin-top: 1rem;">
      Click either button - both counters update together
    </p>
  </div>
</div>
```

```html
<div id="demo-shared">
  <h3>Shared State Example</h3>
  <div class="shared-counter">
    <h4>Counter A</h4>
    <p>Count: <strong $:text="count"></strong></p>
    <button $:click="increment">+1</button>
  </div>
  <div class="shared-counter">
    <h4>Counter B</h4>
    <p>Count: <strong $:text="count"></strong></p>
    <button $:click="increment">+1</button>
  </div>
</div>
```

```js inject show
// Mount multiple elements with shared state
const x = JQR.mountEach('#demo-shared .shared-counter', {
  state: { count: 0 },
  methods: {
    increment() {
      this.count++;
    }
  }
});

```

```js cleanup
onUnmount(() => {
  const instances = JQR.mountEach('#demo-shared .shared-counter', {});
  instances.forEach(inst => inst.$destroy());
})
```

**Use Cases for Shared State:**
- Shopping cart summary (header + sidebar)
- Status indicator (multiple locations)
- Notification badge (repeated in nav)
- Live chat widget (multiple entry points)

### Independent Instances

Use `{ independent: true }` for separate state:

```html live:no-code
<div id="demo-independent" class="en-card">
  <div class="en-card__body">
    <h3>Independent State Example</h3>
    <p class="en-text--muted">Each calculator is independent:</p>
    <div class="en-grid en-grid--cols-12 en-gap-3">
      <div class="en-col-12 en-col-md-6">
        <div class="independent-calc en-card">
          <div class="en-card__body">
            <h4>Calculator 1</h4>
            <div class="en-flex en-gap-2 en-items-center" style="margin-bottom: 0.5rem;">
              <input class="en-input" $:model.number="a" type="number" placeholder="A">
              <span class="en-form-display en-w-auto en-px-5">+</span>
              <input class="en-input" $:model.number="b" type="number" placeholder="B">
            </div>
            <p>Result: <strong $:text="a + b"></strong></p>
          </div>
        </div>
      </div>
      <div class="en-col-12 en-col-md-6">
        <div class="independent-calc en-card">
          <div class="en-card__body">
            <h4>Calculator 2</h4>
            <div class="en-flex en-gap-2 en-items-center" style="margin-bottom: 0.5rem;">
              <input class="en-input" $:model.number="a" type="number" placeholder="A">
              <span class="en-form-display en-w-auto en-px-5">+</span>
              <input class="en-input" $:model.number="b" type="number" placeholder="B">
            </div>
            <p>Result: <strong $:text="a + b"></strong></p>
          </div>
        </div>
      </div>
    </div>
  </div>
</div>
```

```html
<div id="demo-independent">
  <h3>Independent State Example</h3>
  <div class="independent-calc">
    <h4>Calculator 1</h4>
    <input $:model.number="a" type="number">
    <span>+</span>
    <input $:model.number="b" type="number">
    <p>Result: <strong $:text="a + b"></strong></p>
  </div>
  <div class="independent-calc">
    <h4>Calculator 2</h4>
    <input $:model.number="a" type="number">
    <span>+</span>
    <input $:model.number="b" type="number">
    <p>Result: <strong $:text="a + b"></strong></p>
  </div>
</div>
```

```js inject show
// Each instance gets deep-cloned state
$.jqr.mountEach('#demo-independent .independent-calc', {
  state: { a: 0, b: 0 }
}, {
  independent: true  // ✅ Each calculator is separate
});
```

```js cleanup
onUnmount(() => {
  const instances = JQR.mountEach('#demo-independent .independent-calc', {});
  instances.forEach(inst => inst.$destroy());
})
```

**Use Cases for Independent State:**
- Repeated widgets (calculators, timers, forms)
- Multiple instances of same component
- Widgets with element-specific data

## Template Stamping

Use the `template` option to inject the same markup into all elements:

```html
<!-- Placeholder elements -->
<div class="widget"></div>
<div class="widget"></div>
<div class="widget"></div>

<!-- Template to stamp -->
<template id="widget-template">
  <div class="widget-body">
    <p $:text="message"></p>
    <button $:click="update">Update</button>
  </div>
</template>

<script>
  JQR.mountEach('.widget', {
    state: { message: 'Hello' },
    methods: { update(s) { s.message = 'Updated!'; } }
  }, {
    template: 'widget-template'  // Template ID or raw HTML
  });
</script>
```

**Template overwrites innerHTML** - Any existing content is replaced.

### Raw HTML Templates

```js
JQR.mountEach('.widget', {
  state: { count: 0 }
}, {
  template: `
    <div>
      <p>Count: <span $:text="count"></span></p>
      <button $:click="increment">+</button>
    </div>
  `
});
```

## Per-Element Options (Factory Pattern)

For element-specific configuration, pass a factory function:

```html
<div class="timer" data-duration="10"></div>
<div class="timer" data-duration="30"></div>
<div class="timer" data-duration="60"></div>

<script>
  JQR.mountEach('.timer', (el, index) => {
    const duration = parseInt(el.dataset.duration) || 10;
    
    return {
      state: {
        duration: duration,
        remaining: duration
      },
      methods: {
        start() {
          this.interval = setInterval(() => {
            if (this.remaining > 0) {
              this.remaining--;
            } else {
              clearInterval(this.interval);
            }
          }, 1000);
        }
      },
      destroyed() {
        clearInterval(this.interval);
      }
    };
  });
</script>
```

**Factory receives:**
- `el` - The HTMLElement being mounted
- `index` - Position in matched set (0-indexed)

**Factory automatically implies `independent: true`** - Can't be used with shared state.

## Shared vs Independent: Deep Dive

### Shared State (Default)

All instances mount with **the same reactive state object**:

```js
const instances = JQR.mountEach('.counter', {
  state: { count: 0 }
});

// All instances reference the same state
instances[0].$state === instances[1].$state; // true

// Mutating in one updates all
instances[0].$state.count++; // All counters show 1
```

**How it works:**
- One `reactive()` state object
- Multiple `Component` instances
- All read/write same state
- Methods/computed automatically follow

**Benefits:**
- Synchronized UI updates
- Single source of truth
- Less memory usage
- Natural data flow

### Independent State

Each instance gets **deep-cloned state**:

```js
const instances = JQR.mountEach('.calculator', {
  state: { a: 0, b: 0 }
}, {
  independent: true
});

// Each has its own state
instances[0].$state !== instances[1].$state; // true

// Changes are isolated
instances[0].$state.a = 5;
instances[1].$state.a; // Still 0
```

**How it works:**
- State is deep-cloned per instance
- Nested objects/arrays copied recursively
- Each instance fully isolated

**Benefits:**
- No cross-instance interference
- Widget-style components
- Element-specific data

## Selecting Elements

Multiple ways to specify elements:

```js
// CSS selector
JQR.mountEach('.widget', options);

// Single HTMLElement
const el = document.querySelector('.widget');
JQR.mountEach(el, options);

// jQuery object / NodeList
JQR.mountEach($('.widget'), options);
$.jqr.mountEach(document.querySelectorAll('.widget'), options);

// Array of elements
const elements = [el1, el2, el3];
JQR.mountEach(elements, options);
```

## Return Value

`mountEach()` returns an array of `JqrComponent` instances:

```js
const instances = JQR.mountEach('.counter', {
  state: { count: 0 }
});

// Access individual instances
console.log(instances.length); // Number of mounted elements

// Call methods
instances[0].refetch('users');

// Destroy all
instances.forEach(inst => inst.$destroy());
```

## Common Patterns

### Synchronized Widgets

Shopping cart in multiple locations:

```js
JQR.mountEach('.mini-cart', {
  state: {
    items: [],
    total: 0
  },
  computed: {
    itemCount() {
      return this.items.length;
    }
  },
  methods: {
    addItem(item) {
      this.items.push(item);
      this.total += item.price;
    }
  }
});

// All .mini-cart elements stay in sync
```

### Independent Calculators

Multiple calculator widgets:

```js
JQR.mountEach('.calculator', {
  state: { a: 0, b: 0, op: '+' },
  computed: {
    result() {
      switch (this.op) {
        case '+': return this.a + this.b;
        case '-': return this.a - this.b;
        case '*': return this.a * this.b;
        case '/': return this.b !== 0 ? this.a / this.b : 0;
        default: return 0;
      }
    }
  }
}, {
  independent: true
});
```

### Element-Specific Configuration

Countdown timers with different durations:

```js
JQR.mountEach('.countdown', (el) => {
  const seconds = parseInt(el.dataset.seconds) || 60;
  
  return {
    state: {
      remaining: seconds,
      started: false
    },
    methods: {
      start() {
        if (this.started) return;
        this.started = true;
        
        this.interval = setInterval(() => {
          if (this.remaining > 0) {
            this.remaining--;
          } else {
            clearInterval(this.interval);
            this.started = false;
          }
        }, 1000);
      },
      reset() {
        clearInterval(this.interval);
        this.remaining = seconds;
        this.started = false;
      }
    },
    destroyed() {
      clearInterval(this.interval);
    }
  };
});
```

### Template with Shared State

Notification badges:

```html
<template id="badge-template">
  <span class="badge" $:text="count" $:show="count > 0"></span>
</template>

<nav>
  <a href="/messages">Messages <span class="notification-badge"></span></a>
  <a href="/alerts">Alerts <span class="notification-badge"></span></a>
</nav>

<script>
  $.jqr.mountEach('.notification-badge', {
    state: { count: 3 }
  }, {
    template: 'badge-template'
  });
  
  // Both badges update together
</script>
```

## Performance Considerations

### Shared State is Cheaper

Shared state uses less memory and creates fewer reactive proxies:

```js
// 100 elements, shared state = 1 reactive object
$.jqr.mountEach('.widget', { state: { x: 0 } });

// 100 elements, independent = 100 reactive objects
$.jqr.mountEach('.widget', { state: { x: 0 } }, { independent: true });
```

Use independent only when needed.

### Deep Clone Cost

Independent mode deep-clones state for each instance. For large state objects:

```js
// ❌ Expensive: Large state cloned 100 times
$.jqr.mountEach('.widget', {
  state: {
    bigArray: [...Array(10000)],
    deepObject: { /* nested data */ }
  }
}, {
  independent: true
});

// ✅ Better: Keep shared data outside state
const SHARED_DATA = Object.freeze([...Array(10000)]);

$.jqr.mountEach('.widget', {
  constants: { sharedData: SHARED_DATA },
  state: { index: 0 }
}, {
  independent: true
});
```

### Lazy Mounting

Don't mount all at once if there are many elements:

```js
// ❌ Bad: Mount 1000 widgets immediately
$.jqr.mountEach('.widget', options);

// ✅ Good: Use Intersection Observer for off-screen elements
const observer = new IntersectionObserver((entries) => {
  entries.forEach(entry => {
    if (entry.isIntersecting) {
      $(entry.target).component(options);
      observer.unobserve(entry.target);
    }
  });
});

$('.widget').each((i, el) => observer.observe(el));
```

## Error Handling

Errors in one instance don't affect others:

```js
const instances = $.jqr.mountEach('.widget', {
  state: { value: 0 },
  mounted() {
    if (Math.random() > 0.5) {
      throw new Error('Random failure!');
    }
  }
});

// Some instances mount successfully, others fail
// Failed instances log errors but don't break the rest
```

Each instance is scanned and mounted independently.

## Cleanup

Destroy all instances:

```js
const instances = $.jqr.mountEach('.widget', options);

// Later...
instances.forEach(instance => instance.destroy());
```

Or destroy by selector:

```js
$('.widget').each((i, el) => {
  $(el).component('destroy');
});
```

## TypeScript Support

Full type inference:

```typescript
import { jqr, ComponentOptions } from '@en32/jqr';

interface WidgetState {
  count: number;
  label: string;
}

const options: ComponentOptions<WidgetState> = {
  state: { count: 0, label: 'Widget' },
  methods: {
    increment() {
      this.count++; // ✅ Type-checked
    }
  }
};

const instances = jqr.mountEach<WidgetState>('.widget', options);
instances[0].$state.count; // ✅ Type: number
```

## Comparison: Manual vs mountEach

### Manual Approach

```js
// ❌ Verbose and error-prone
const options = { state: { count: 0 } };
const instances = [];

document.querySelectorAll('.counter').forEach((el, i) => {
  // Must deep-clone manually
  const clonedOptions = {
    ...options,
    state: JSON.parse(JSON.stringify(options.state))
  };
  
  const instance = $(el).component(clonedOptions);
  instances.push(instance);
});
```

### Using mountEach

```js
// ✅ Simple and correct
const instances = $.jqr.mountEach('.counter', {
  state: { count: 0 }
}, {
  independent: true
});
```

## See Also

- [Component Options](./component-options.md) - All component options
- [State Management](./state-management.md) - State sharing patterns
- [Performance Guide](./performance.md) - Optimization tips
- [Configuration Reference](../reference/config-reference.md) - API details
