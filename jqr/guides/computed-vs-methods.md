<!-- {
  "path": "jqr/guides/computed-vs-methods",
  "name": "docs-jqr-guides-computed-vs-methods",
  "meta": {
    "title": "Computed vs Methods",
    "order": 30
  }
} -->

# Computed vs Methods

Understanding when to use computed properties versus methods is crucial for writing efficient JQR components. This guide explains the differences and provides clear decision-making criteria.

## Key Differences

### Computed Properties

**Cached** - Result is cached until dependencies change:

```js
computed: {
  fullName(c) {
    console.log('Computing fullName');  // Only logs when firstName/lastName change
    return `${c.firstName} ${c.lastName}`;
  }
}
```

**Reactive** - Automatically tracks dependencies:

```html
<p $:text="fullName"></p>  <!-- Updates when firstName or lastName changes -->
```

### Methods

**Not cached** - Runs every time it's called:

```js
methods: {
  getFullName() {
    console.log('Getting fullName');  // Logs on EVERY access
    return `${this.firstName} ${this.lastName}`;
  }
}
```

**Manual invocation** - You decide when to run:

```html
<p $:text="getFullName()"></p>  <!-- Runs on every template evaluation -->
<button $:click="updateName()">Update</button>  <!-- Runs when clicked -->
```

## When to Use Computed

### 1. Derived State

Calculate values from existing state:

```js
$('#app').component({
  state: {
    items: [
      { name: 'Apple', price: 1.5 },
      { name: 'Banana', price: 0.8 },
      { name: 'Cherry', price: 2.0 }
    ]
  },
  computed: {
    // ✅ Perfect for computed: derived from state
    totalPrice(c) {
      return c.items.reduce((sum, item) => sum + item.price, 0);
    },
    itemCount(c) {
      return c.items.length;
    }
  }
});
```

```html
<p>Total: $<span $:text="totalPrice"></span></p>
<p>Items: <span $:text="itemCount"></span></p>
```

### 2. Expensive Calculations

Cache results of heavy computations:

```html live:no-code
<div id="demo-computed-expensive" class="en-card en-card__body">
  <div class="en-form-group en-mb-3">
    <label class="en-label">Search:</label>
    <input class="en-input" $:model="search" placeholder="Filter items..." />
  </div>
  <p class="en-text--muted en-text-sm">Filtered: <code $:text="filtered.length"></code> items</p>
  <ul class="en-list-group">
    <li class="en-list-group__item" $:for="item in filtered" $:key="item" $:text="item"></li>
  </ul>
</div>
```

```html
<div id="demo-computed-expensive">
  <label>Search:</label>
  <input $:model="search" />
  
  <p>Filtered: <code $:text="filtered.length"></code> items</p>
  <ul>
    <li $:for="item in filtered" $:key="item" $:text="item"></li>
  </ul>
</div>
```

```js inject show
$('#demo-computed-expensive').component({
  state: {
    search: '',
    items: ['Apple', 'Banana', 'Cherry', 'Date', 'Elderberry', 'Fig', 'Grape']
  },
  computed: {
    // ✅ Cached: only recalculates when items or search change
    filtered(c) {
      console.log('Filtering...');  // Check console - only runs when needed
      return c.items.filter(item => 
        item.toLowerCase().includes(c.search.toLowerCase())
      );
    }
  }
});
```

```js cleanup
onUnmount(() => {
  $('#demo-computed-expensive').component('destroy');
})
```

### 3. Multiple Template Uses

Access the same value multiple times without recalculation:

```js
computed: {
  // ✅ Called 10 times, calculated once per update
  activeCount() {
    return this.items.filter(i => i.active).length;
  }
}
```

```html
<p>Active: <span $:text="activeCount"></span></p>
<p>Inactive: <span $:text="items.length - activeCount"></span></p>
<div $:if="activeCount > 0">You have active items</div>
<div $:if="activeCount > 10">Too many active items!</div>
```

**Result:** `activeCount` computed **once**, used 4 times.

### 4. Conditional Logic

```js
computed: {
  // ✅ Clean conditional logic
  statusMessage() {
    if (this.loading) return 'Loading...';
    if (this.error) return 'Error occurred';
    if (this.data.length === 0) return 'No data';
    return `${this.data.length} items loaded`;
  }
}
```

```html
<p $:text="statusMessage"></p>
```

## When to Use Methods

### 1. Actions & Side Effects

Perform operations, trigger changes:

```html live:no-code
<div id="demo-methods-actions" class="en-card en-card__body">
  <p class="en-mb-2">Count: <code $:text="count"></code></p>
  <div class="en-flex en-gap-2">
    <button class="en-btn en-btn--sm en-btn--primary" $:click="increment()">+1</button>
    <button class="en-btn en-btn--sm en-btn--danger" $:click="decrement()">-1</button>
    <button class="en-btn en-btn--sm en-btn--outline" $:click="reset()">Reset</button>
  </div>
</div>
```

```html
<div id="demo-methods-actions">
  <p>Count: <code $:text="count"></code></p>
  <button $:click="increment()">+1</button>
  <button $:click="decrement()">-1</button>
  <button $:click="reset()">Reset</button>
</div>
```

```js inject show
$('#demo-methods-actions').component({
  state: { count: 0 },
  methods: {
    // ✅ Actions belong in methods
    increment() {
      this.count++;
    },
    decrement() {
      this.count--;
    },
    reset(state) {
      this.count = 0;
    }
  }
});
```

```js cleanup
onUnmount(() => {
  $('#demo-methods-actions').component('destroy');
})
```

### 2. Accepting Parameters

Pass arguments to logic:

```js
methods: {
  // ✅ Methods can accept parameters
  add(a, b) {
    return a + b;
  },
  formatCurrency(amount) {
    return `$${this.toFixed(2)}`;
  },
  filterByType(type) {
    return this.items.filter(i => i.type === type);
  }
}
```

```html
<p $:text="formatCurrency(totalPrice)"></p>
<div $:for="item in filterByType('fruit')" $:key="item.id">
  ...
</div>
```

### 3. Event Handlers

Respond to user interactions:

```html live:no-code
<div id="demo-event-handlers-interactions">
  <form $:submit="handleSubmit">
    <div class="en-field en-mb-3">
      <input class="en-input" $:input="handleInput" />
      <div>Length: <small $:text="textLength"></small></div>
    </div>
    <div class="en-file-upload en-file-upload--compact en-mb-3">
      <div class="en-file-upload__zone">
        <input type="file" $:change="onFileSelect" class="en-file-upload__input" />
        
        <div class="en-file-upload__icon">
          <svg xmlns="http://www.w3.org/2000/svg" fill="none" viewBox="0 0 24 24" stroke="currentColor">
            <path stroke-linecap="round" stroke-linejoin="round" stroke-width="2" d="M15.172 7l-6.586 6.586a2 2 0 102.828 2.828l6.414-6.586a4 4 0 00-5.656-5.656l-6.415 6.585a6 6 0 108.486 8.486L20.5 13" />
          </svg>
        </div>
        
        <div class="en-file-upload__text">
          <div class="en-file-upload__title">Attach files</div>
          <div class="en-file-upload__hint">Click or drag files here</div>
        </div>
      </div>
    </div>
    <pre class="en-mb-3" $:text="$state" $:show="showdata"></pre>
    <button class="en-btn" type="submit">Save</button>
  </form>
</div>
```

```js inject show
$('#demo-event-handlers-interactions').component({
  state: { text: '', file: null, showdata: false },
  computed: {
    textLength: (ctx) => ctx.text.length 
  },
  methods: {
    // ✅ Event handlers are methods
    handleSubmit(event) {
      event.preventDefault();
      this.showdata = true
      // Process form...
    },
    handleInput(event) {
      // console.log('User typed:', event.target.value);
      this.showdata = false
      this.text = event.target.value
    },
    onFileSelect(event) {
      this.showdata = false
      const file = event.target.files[0];

      // Handle file...
      if (file) {
        this.file = {
          name: file.name,
          size: file.size,
          type: file.type,
          lastModified: file.lastModified,
        };
      }

    }
  }
})
```

```html
<form $:submit="handleSubmit">
  <input $:input="handleInput" />
  <input type="file" $:change="onFileSelect" />
</form>
```

### 4. Async Operations

Fetch data, wait for operations:

```js
methods: {
  // ✅ Async logic belongs in methods
  async fetchData() {
    this.loading = true;
    try {
      const res = await fetch('/api/data');
      this.data = await res.json();
    } catch (err) {
      this.error = err.message;
    } finally {
      this.loading = false;
    }
  },
  async saveChanges() {
    await fetch('/api/save', {
      method: 'POST',
      body: JSON.stringify(this.form)
    });
  }
}
```

### 5. Non-Reactive Calculations

Compute values that don't depend on state:

```js
methods: {
  // ✅ Pure utility functions
  validateEmail(email) {
    return /^[^\s@]+@[^\s@]+\.[^\s@]+$/.test(email);
  },
  generateId() {
    return `id-${Date.now()}-${Math.random()}`;
  },
  formatDate(timestamp) {
    return new Date(timestamp).toLocaleDateString();
  }
}
```

## Common Patterns

### Pattern 1: Computed for Display, Method for Action

```html live
<div id="cart-demo">
  <pre $:text="$state"></pre>
</div>
```

```js inject show
$('#cart-demo').component({
  state: {
    items: [
      { id: 1, name: 'Apple', price: 1.5, qty: 2 },
      { id: 2, name: 'Banana', price: 0.8, qty: 5 }
    ]
  },
  computed: {
    // ✅ Computed: derived display values
    subtotal() {
      return this.items.reduce((sum, item) => sum + (item.price * item.qty), 0);
    },
    tax() {
      return this.subtotal * 0.1;
    },
    total() {
      return this.subtotal + this.tax;
    }
  },
  methods: {
    // ✅ Methods: user actions
    removeItem(id) {
      this.items = this.items.filter(i => i.id !== id);
    },
    updateQuantity(id, qty) {
      const item = this.items.find(i => i.id === id);
      if (item) item.qty = qty;
    },
    async checkout() {
      await fetch('/api/checkout', {
        method: 'POST',
        body: JSON.stringify({ items: this.items, total: this.total })
      });
    }
  }
});
```

### Pattern 2: Computed Chains

```js
computed: {
  // ✅ Chain computed properties
  filteredItems() {
    return this.items.filter(i => i.active);
  },
  sortedItems() {
    return [...this.filteredItems].sort((a, b) => a.name.localeCompare(b.name));
  },
  paginatedItems() {
    const start = (this.page - 1) * this.pageSize;
    return this.sortedItems.slice(start, start + this.pageSize);
  }
}
```

### Pattern 3: Method Calls Computed

```js
computed: {
  // ✅ Computed can call methods for reusable logic
  validationErrors() {
    return {
      email: this.validateEmail(this.email) ? null : 'Invalid email',
      phone: this.validatePhone(this.phone) ? null : 'Invalid phone'
    };
  }
},
methods: {
  validateEmail(email) {
    return /^[^\s@]+@[^\s@]+\.[^\s@]+$/.test(email);
  },
  validatePhone(phone) {
    return /^\d{10}$/.test(phone);
  }
}
```

## Performance Comparison

### Computed Advantage

```js
// ❌ Method: Runs 1000 times
methods: {
  expensiveCalc() {
    return this.items.map(i => heavyComputation(i));
  }
}
```

```html
<!-- Called 1000 times in template render -->
<div $:for="item in expensiveCalc()" $:key="item.id">...</div>
```

**vs**

```js
// ✅ Computed: Runs once per items change
computed: {
  expensiveCalc() {
    return this.items.map(i => heavyComputation(i));
  }
}
```

```html
<!-- Cached result reused -->
<div $:for="item in expensiveCalc" $:key="item.id">...</div>
```

### Method Advantage

```js
// ❌ Computed: Can't accept parameters
computed: {
  filterByType() {
    // How do we pass 'type'?
  }
}

// ✅ Method: Flexible parameters
methods: {
  filterByType(type) {
    return this.items.filter(i => i.type === type);
  }
}
```

## Anti-Patterns

### 1. Computed with Side Effects

```js
// ❌ BAD: Computed should NOT mutate state
computed: {
  itemCount() {
    this.cached = this.items.length;  // Side effect!
    return this.items.length;
  }
}

// ✅ GOOD: Pure computation only
computed: {
  itemCount() {
    return this.items.length;
  }
}
```

### 2. Method for Simple Derivation

```js
// ❌ BAD: No caching, runs on every access
methods: {
  fullName() {
    return `${this.firstName} ${this.lastName}`;
  }
}

// ✅ GOOD: Cached automatically
computed: {
  fullName() {
    return `${this.firstName} ${this.lastName}`;
  }
}
```

### 3. Computed with External Dependencies

```js
// ❌ BAD: Depends on external state, not reactive
let externalValue = 10;

computed: {
  result() {
    return this.count + externalValue;  // Won't react to externalValue
  }
}

// ✅ GOOD: All dependencies in state
state: {
  count: 0,
  externalValue: 10
},
computed: {
  result() {
    return this.count + this.externalValue;  // Fully reactive
  }
}
```

### 4. Heavy Computation in Method Called from Template

```js
// ❌ BAD: Runs on every template evaluation
methods: {
  heavyFilter() {
    return this.items.filter(i => expensiveCheck(i));
  }
}
```

```html
<div $:for="item in heavyFilter()" $:key="item.id">...</div>
```

**vs**

```js
// ✅ GOOD: Cached, only runs when items change
computed: {
  heavyFilter() {
    return this.items.filter(i => expensiveCheck(i));
  }
}
```

```html
<div $:for="item in heavyFilter" $:key="item.id">...</div>
```

## Decision Matrix

| Scenario | Use Computed | Use Method |
|----------|--------------|------------|
| Derive value from state | ✅ | ❌ |
| Used multiple times in template | ✅ | ❌ |
| Expensive calculation | ✅ | ❌ |
| Needs parameters | ❌ | ✅ |
| Event handler | ❌ | ✅ |
| Async operation | ❌ | ✅ |
| Mutates state | ❌ | ✅ |
| Side effects (API call, logging) | ❌ | ✅ |
| Pure utility function | ❌ | ✅ |

## Quick Reference

**Use Computed when:**
- Value derived from state
- Result should be cached
- Used multiple times
- No parameters needed
- Pure function (no side effects)

**Use Methods when:**
- Action/command (changes state)
- Accepts parameters
- Event handler
- Async operation
- Has side effects
- Not used in template (or rarely)

## Testing Tip

Check if computed is caching correctly:

```js
computed: {
  expensiveCalc() {
    this.count('expensiveCalc');  // Count executions
    return /* ... */;
  }
}
```

**Expected:** Count increases only when dependencies change, not on every template render.

## Summary

**Computed Properties:**
- **Cached** until dependencies change
- **Reactive** to state
- Best for **derived values**
- **Read-only** in templates (`fullName`, not `fullName()`)

**Methods:**
- **Not cached**, runs every call
- **Manual invocation**
- Best for **actions & logic**
- **Callable** with parameters (`filterByType('fruit')`)

**Golden Rule:** If it's deriving a value from state and used in the template, use computed. If it's doing something (action, side effect, accepting params), use method.

## Next Steps

- [Reactivity Guide](./reactivity.md) - Understanding reactive system
- [Component Options](./component-options.md) - Complete API reference
- [Performance Tips](./performance.md) - Optimization strategies
- [Instance Methods](./instance-methods.md) - Built-in methods API
