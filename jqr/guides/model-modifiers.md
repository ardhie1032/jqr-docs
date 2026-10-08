<!-- {
  "path": "jqr/guides/model-modifiers",
  "name": "docs-jqr-guides-model-modifiers",
  "meta": {
    "title": "Model Modifiers",
    "order": 4
  }
} -->

# Model Modifiers

JQR's `$:model` directive supports **modifiers** that transform input values or change sync behavior. Chain multiple modifiers together for powerful form handling.

## Available Modifiers

| Modifier | Purpose | Example |
|----------|---------|---------|
| `.number` | Convert to number | `$:model.number="age"` |
| `.trim` | Trim whitespace | `$:model.trim="name"` |
| `.lazy` | Sync on change (not input) | `$:model.lazy="bio"` |
| `.debounce` | Debounced updates | `$:model.debounce.300="search"` |
| `.throttle` | Throttled updates | `$:model.throttle.500="scroll"` |
| `.confirm` | Confirmation before change | `$:model.confirm="important"` |
| `.once` | Accept first change only | `$:model.once="readonly"` |

## `.number` - Convert to Number

Automatically converts input values to numbers:

```html live:no-code
<div id="number-demo" class="en-card">
  <div class="en-card__body">
    <h3>Price Calculator</h3>
    
    <div class="en-flex en-gap-3" style="margin-bottom: 1rem;">
      <div class="en-form-group">
        <label class="en-label">Price</label>
        <input class="en-input" type="number" $:model.number="price">
      </div>
      
      <div class="en-form-group">
        <label class="en-label">Quantity</label>
        <input class="en-input" type="number" $:model.number="quantity">
      </div>
    </div>
    
    <div class="en-alert en-alert--success">
      <strong>Total:</strong> $<span $:text="total"></span>
    </div>
    
    <div class="en-text--muted" style="margin-top: 0.5rem;">
      Type: <code $:text="priceType"></code>
    </div>
  </div>
</div>
```

```html
<div id="number-demo">
  <h3>Price Calculator</h3>
  
  <label>Price</label>
  <input type="number" $:model.number="price">
  
  <label>Quantity</label>
  <input type="number" $:model.number="quantity">
  
  <p><strong>Total:</strong> $<span $:text="total"></span></p>
  <p>Type: <code $:text="priceType"></code></p>
</div>
```

```js inject show
$('#number-demo').component({
  state: {
    price: 10,
    quantity: 1
  },
  computed: {
    total(c) {
      return c.price * c.quantity;
    },
    priceType(c) {
      return typeof c.price;
    }
  }
});
```

```js cleanup
onUnmount(() => {
  $('#number-demo').component('destroy');
})
```

**Without `.number`:**
```js
// Input "42" → state becomes string "42"
$:model="age"  // typeof age === "string"
```

**With `.number`:**
```js
// Input "42" → state becomes number 42
$:model.number="age"  // typeof age === "number"
```

**Edge cases:**
- Empty input `""` stays as `""` (not `0` or `NaN`)
- Invalid numbers become `NaN`

## `.trim` - Trim Whitespace

Removes leading and trailing whitespace:

```html live:no-code
<div id="trim-demo" class="en-card">
  <div class="en-card__body">
    <h3>Username Registration</h3>
    
    <div class="en-form-group">
      <label class="en-label">Username (with .trim)</label>
      <input 
        class="en-input" 
        $:model.trim="username" 
        placeholder="Try typing spaces...">
    </div>
    
    <div class="en-alert en-alert--muted">
      <strong>Stored value:</strong> "<code $:text="username"></code>"<br>
      <strong>Length:</strong> <span $:text="username.length"></span> characters
    </div>
    
    <div class="en-form-group" style="margin-top: 1rem;">
      <label class="en-label">Comment (without .trim)</label>
      <input 
        class="en-input" 
        $:model="comment" 
        placeholder="Spaces are kept">
    </div>
    
    <div class="en-alert en-alert--muted">
      <strong>Stored value:</strong> "<code $:text="comment"></code>"<br>
      <strong>Length:</strong> <span $:text="comment.length"></span> characters
    </div>
  </div>
</div>
```

```html
<div id="trim-demo">
  <h3>Username Registration</h3>
  
  <label>Username (with .trim)</label>
  <input $:model.trim="username" placeholder="Try typing spaces...">
  
  <p>
    <strong>Stored:</strong> "<code $:text="username"></code>"<br>
    <strong>Length:</strong> <span $:text="username.length"></span>
  </p>
  
  <label>Comment (without .trim)</label>
  <input $:model="comment" placeholder="Spaces kept">
  
  <p>
    <strong>Stored:</strong> "<code $:text="comment"></code>"<br>
    <strong>Length:</strong> <span $:text="comment.length"></span>
  </p>
</div>
```

```js inject show
$('#trim-demo').component({
  state: {
    username: '',
    comment: ''
  }
});
```

```js cleanup
onUnmount(() => {
  $('#trim-demo').component('destroy');
})
```

**Use cases:**
- Username/email fields
- Search queries
- Any input where leading/trailing spaces are meaningless

## `.lazy` - Sync on Change

By default, `$:model` syncs on every keystroke (`input` event). Use `.lazy` to sync only when the field loses focus (`change` event):

```html live:no-code
<div id="lazy-demo" class="en-card">
  <div class="en-card__body">
    <h3>Input Sync Comparison</h3>
    
    <div class="en-flex en-gap-3">
      <div class="en-form-group">
        <label class="en-label">Default (immediate)</label>
        <input class="en-input" $:model="immediate">
        <div class="en-text--muted">
          Updates: <span $:text="immediateCount"></span>
        </div>
      </div>
      
      <div class="en-form-group">
        <label class="en-label">With .lazy (on blur)</label>
        <input class="en-input" $:model.lazy="lazy">
        <div class="en-text--muted">
          Updates: <span $:text="lazyCount"></span>
        </div>
      </div>
    </div>
    
    <div class="en-alert en-alert--info" style="margin-top: 1rem;">
      Try typing in both fields. The lazy input only updates when you click away or press Tab.
    </div>
  </div>
</div>
```

```html
<div id="lazy-demo">
  <h3>Input Sync Comparison</h3>
  
  <div>
    <label>Default (immediate)</label>
    <input $:model="immediate">
    <p>Updates: <span $:text="immediateCount"></span></p>
  </div>
  
  <div>
    <label>With .lazy (on blur)</label>
    <input $:model.lazy="lazy">
    <p>Updates: <span $:text="lazyCount"></span></p>
  </div>
  
  <p>Try typing in both fields. Lazy only updates on blur.</p>
</div>
```

```js inject show
$('#lazy-demo').component({
  state: {
    immediate: '',
    lazy: '',
    immediateCount: 0,
    lazyCount: 0
  },
  watch: {
    'immediate': (c) => {
      c.immediateCount++;
    },
    lazy: (c) => {
      c.lazyCount++;
    }
  }
});
```

```js cleanup
onUnmount(() => {
  $('#lazy-demo').component('destroy');
})
```

**When to use `.lazy`:**
- Large text areas where immediate sync is expensive
- Fields that trigger side effects (validations, API calls)
- Better UX when you don't need live updates

## `.debounce` - Debounced Updates

Delays state updates until typing pauses. Perfect for search boxes and autocomplete:

```html live:no-code
<div id="debounce-demo" class="en-card">
  <div class="en-card__body">
    <h3>Search Users</h3>
    
    <input 
      class="en-input" 
      $:model.debounce.500="query" 
      placeholder="Type to search...">
    
    <div class="en-text--muted" style="margin-top: 0.5rem;">
      Search count: <span $:text="searchCount"></span>
    </div>
    
    <div class="en-alert en-alert--info" $:show="query" style="margin-top: 1rem;">
      Searching for: "<strong $:text="query"></strong>"
    </div>
  </div>
</div>
```

```html
<div id="debounce-demo">
  <h3>Search Users</h3>
  
  <input 
    $:model.debounce.500="query" 
    placeholder="Type to search...">
  
  <p>Search count: <span $:text="searchCount"></span></p>
  <p $:show="query">Searching: "<strong $:text="query"></strong>"</p>
</div>
```

```js inject show
$('#debounce-demo').component({
  state: {
    query: '',
    searchCount: 0
  },
  watch: {
    query() {
      this.searchCount++;
    }
  }
});
```

```js cleanup
onUnmount(() => {
  $('#debounce-demo').component('destroy');
})
```

**Syntax:**
```html
<!-- Default 250ms delay -->
<input $:model.debounce="search">

<!-- Custom delay (500ms) -->
<input $:model.debounce.500="search">
```

**How it works:**
- User types "hello"
- Timer starts after each keystroke
- If 500ms pass with no new keystrokes, state updates
- If user types again, timer resets

**Use cases:**
- Search/autocomplete inputs
- API calls on user input
- Expensive computed properties
- Live validation

## `.throttle` - Throttled Updates

Limits update frequency to once per time period:

```html live:no-code
<div id="throttle-demo" class="en-card">
  <div class="en-card__body">
    <h3>Scroll Position Tracker</h3>
    
    <div class="en-form-group">
      <label class="en-label">Scroll value (throttled)</label>
      <input 
        class="en-input" 
        type="range" 
        min="0" 
        max="100" 
        $:model.throttle.100.number="position">
    </div>
    
    <div class="en-alert en-alert--success">
      Position: <strong $:text="position"></strong>
    </div>
    
    <div class="en-text--muted">
      Updates: <span $:text="updateCount"></span>
    </div>
  </div>
</div>
```

```html
<div id="throttle-demo">
  <h3>Scroll Position Tracker</h3>
  
  <label>Scroll value (throttled)</label>
  <input 
    type="range" 
    min="0" 
    max="100" 
    $:model.throttle.100.number="position">
  
  <p>Position: <strong $:text="position"></strong></p>
  <p>Updates: <span $:text="updateCount"></span></p>
</div>
```

```js inject show
$('#throttle-demo').component({
  state: {
    position: 50,
    updateCount: 0
  },
  watch: {
    position() {
      this.state.updateCount++;
    }
  }
});
```

```js cleanup
onUnmount(() => {
  $('#throttle-demo').component('destroy');
})
```

**Debounce vs Throttle:**

| Scenario | Debounce | Throttle |
|----------|----------|----------|
| User types "hello" fast | Updates **once** after pause | Updates **periodically** while typing |
| Scroll/resize events | Updates when scrolling **stops** | Updates at **regular intervals** during scroll |
| Best for | Search, autocomplete | Scroll position, mouse tracking |

**Syntax:**
```html
<!-- Default 250ms interval -->
<input $:model.throttle="value">

<!-- Custom interval (1000ms = 1 second) -->
<input $:model.throttle.1000="value">
```

## `.confirm` - Confirmation Required

Ask for user confirmation before accepting changes:

```html live:no-code
<div id="confirm-demo" class="en-card">
  <div class="en-card__body">
    <h3>Critical Setting</h3>
    
    <div class="en-form-group">
      <label class="en-label">
        <input type="checkbox" $:model.confirm="deleteEnabled">
        Enable delete mode (requires confirmation)
      </label>
    </div>
    
    <div class="en-alert" $:class="deleteEnabled ? 'en-alert--danger' : 'en-alert--muted'">
      Delete mode: <strong $:text="deleteEnabled ? 'ENABLED' : 'Disabled'"></strong>
    </div>
  </div>
</div>
```

```html
<div id="confirm-demo">
  <h3>Critical Setting</h3>
  
  <label>
    <input type="checkbox" $:model.confirm="deleteEnabled">
    Enable delete mode
  </label>
  
  <p>
    Status: <strong $:text="deleteEnabled ? 'ENABLED' : 'Disabled'"></strong>
  </p>
</div>
```

```js inject show
$('#confirm-demo').component({
  state: {
    deleteEnabled: false
  }
});

// Listen for confirmation events
$(document).on('jqr:confirm', (e) => {
  const detail = e.detail;
  const confirmed = confirm('Are you sure you want to change this critical setting?');
  
  if (confirmed) {
    detail.accept();
  } else {
    detail.reject();
  }
});
```

```js cleanup
onUnmount(() => {
  $('#confirm-demo').component('destroy');
  $(document).off('jqr:confirm');
})
```

**How it works:**
1. User interacts with the input
2. JQR fires a `jqr:confirm` custom event
3. Your code calls `detail.accept()` or `detail.reject()`
4. State updates only if accepted

**Use cases:**
- Destructive actions
- Critical settings
- Financial inputs
- Data loss warnings

## `.once` - Accept First Change Only

Makes the input read-only after the first change:

```html live:no-code
<div id="once-demo" class="en-card">
  <div class="en-card__body">
    <h3>Registration Code (One-Time Entry)</h3>
    
    <div class="en-form-group">
      <label class="en-label">Enter code (you can only set this once)</label>
      <input 
        class="en-input" 
        $:model.once.trim="registrationCode" 
        placeholder="Enter code..."
        $:attr:disabled="registrationCode !== ''">
    </div>
    
    <div class="en-alert en-alert--info" $:show="!registrationCode">
      Once you enter a code, it cannot be changed.
    </div>
    
    <div class="en-alert en-alert--success" $:show="registrationCode">
      Locked code: <strong $:text="registrationCode"></strong>
    </div>
    
    <button 
      class="en-btn en-btn--sm en-btn--outline" 
      $:click="reset" 
      $:show="registrationCode">
      Reset Demo
    </button>
  </div>
</div>
```

```html
<div id="once-demo">
  <h3>Registration Code</h3>
  
  <label>Enter code (one-time only)</label>
  <input 
    $:model.once.trim="registrationCode" 
    placeholder="Enter code..."
    $:attr:disabled="registrationCode !== ''">
  
  <p $:show="!registrationCode">
    Once entered, it cannot be changed.
  </p>
  
  <p $:show="registrationCode">
    Locked: <strong $:text="registrationCode"></strong>
  </p>
  
  <button $:click="reset" $:show="registrationCode">
    Reset Demo
  </button>
</div>
```

```js inject show
$('#once-demo').component({
  state: {
    registrationCode: ''
  },
  methods: {
    reset(state) {
      state.registrationCode = '';
    }
  }
});
```

```js cleanup
onUnmount(() => {
  $('#once-demo').component('destroy');
})
```

**Use cases:**
- Registration codes
- One-time setup values
- Immutable user choices
- Prevent accidental overwrites

## Chaining Modifiers

Combine multiple modifiers for complex behavior:

```html
<!-- Trim + convert to number -->
<input $:model.trim.number="price">

<!-- Debounced + trimmed search -->
<input $:model.debounce.300.trim="search">

<!-- Lazy + number conversion -->
<input $:model.lazy.number="age">
```

**Complete Example: Advanced Search Form**

```html live:no-code
<div id="advanced-search" class="en-card">
  <div class="en-card__body">
    <h3>Product Search</h3>
    
    <div class="en-flex en-gap-2" style="margin-bottom: 1rem;">
      <input 
        class="en-input" 
        $:model.debounce.300.trim="searchQuery" 
        placeholder="Search products...">
      
      <input 
        class="en-input" 
        type="number" 
        $:model.number="minPrice" 
        placeholder="Min price">
      
      <input 
        class="en-input" 
        type="number" 
        $:model.number="maxPrice" 
        placeholder="Max price">
    </div>
    
    <div class="en-alert en-alert--muted">
      <strong>Query:</strong> "<span $:text="searchQuery"></span>"<br>
      <strong>Price range:</strong> $<span $:text="minPrice || 0"></span> - $<span $:text="maxPrice || '∞'"></span><br>
      <strong>Search count:</strong> <span $:text="searchCount"></span>
    </div>
  </div>
</div>
```

```html
<div id="advanced-search">
  <h3>Product Search</h3>
  
  <input 
    $:model.debounce.300.trim="searchQuery" 
    placeholder="Search...">
  
  <input 
    type="number" 
    $:model.number="minPrice" 
    placeholder="Min">
  
  <input 
    type="number" 
    $:model.number="maxPrice" 
    placeholder="Max">
  
  <p>
    Query: "<span $:text="searchQuery"></span>"<br>
    Range: $<span $:text="minPrice || 0"></span> - 
           $<span $:text="maxPrice || '∞'"></span><br>
    Searches: <span $:text="searchCount"></span>
  </p>
</div>
```

```js inject show
$('#advanced-search').component({
  state: {
    searchQuery: '',
    minPrice: 0,
    maxPrice: 0,
    searchCount: 0
  },
  watch: {
    searchQuery() {
      this.state.searchCount++;
    },
    minPrice() {
      this.state.searchCount++;
    },
    maxPrice() {
      this.state.searchCount++;
    }
  }
});
```

```js cleanup
onUnmount(() => {
  $('#advanced-search').component('destroy');
})
```

## Modifier Conflicts

Some modifiers don't make sense together:

### `.debounce` + `.throttle`
**Conflict:** Both control timing, but in different ways.

```html
<!-- ❌ Bad: .debounce wins, .throttle ignored -->
<input $:model.debounce.throttle="value">
```

**Warning:**
```
[jqr] $:model="value" combines .debounce and .throttle 
— .debounce takes precedence, .throttle is ignored.
```

### `.lazy` + `.debounce`/`.throttle`
**Conflict:** `.lazy` changes the event (`change` vs `input`), but timing modifiers need `input` events to work.

```html
<!-- ❌ Bad: .lazy ignored when timing modifier present -->
<input $:model.lazy.debounce.300="value">
```

**Warning:**
```
[jqr] $:model="value" combines .lazy with .debounce/.throttle 
— .lazy is ignored (input event needed for timing).
```

### `.confirm` + `.once`
**Conflict:** After the first change (`.once`), there's nothing left to confirm.

```html
<!-- ❌ Bad: .once wins, .confirm ignored -->
<input $:model.confirm.once="value">
```

**Warning:**
```
[jqr] $:model="value" combines .confirm and .once 
— they cancel out; .once wins, .confirm ignored.
```

## Default Timing Delay

When `.debounce` or `.throttle` is used without a number, the default is **250ms**:

```html
<!-- These are equivalent: -->
<input $:model.debounce="query">
<input $:model.debounce.250="query">

<input $:model.throttle="position">
<input $:model.throttle.250="position">
```

## Best Practices

### 1. Use `.number` with `type="number"`

```html
<!-- ✅ Good: type and modifier match -->
<input type="number" $:model.number="age">

<!-- ⚠️ Works but less semantic -->
<input type="text" $:model.number="age">
```

### 2. Combine `.debounce` with `.trim` for search

```html
<!-- ✅ Best for search inputs -->
<input $:model.debounce.300.trim="searchQuery">
```

### 3. Use `.lazy` for expensive operations

```html
<!-- ✅ Reduces re-renders on large textareas -->
<textarea $:model.lazy="biography"></textarea>
```

### 4. Don't over-debounce

```html
<!-- ❌ Too slow, feels unresponsive -->
<input $:model.debounce.2000="search">

<!-- ✅ Sweet spot: 200-500ms -->
<input $:model.debounce.300="search">
```

### 5. File inputs need special handling

Model modifiers work on file inputs, but `.number` doesn't make sense:

```html
<!-- ✅ File metadata extraction -->
<input type="file" $:model.meta="uploadedFile">

<!-- ❌ Don't use .number with files -->
<input type="file" $:model.number="file">
```

## Modifier Order

The order of modifiers matters for readability but not behavior:

```html
<!-- Both work identically: -->
<input $:model.trim.number="price">
<input $:model.number.trim="price">
```

**Convention:** Put timing modifiers first, then transformations:

```html
<!-- ✅ Recommended order -->
<input $:model.debounce.300.trim.number="value">
```

## Browser Compatibility

All modifiers work in all modern browsers (Chrome, Firefox, Safari, Edge). No polyfills needed.

**Input event support:**
- Desktop: All modern browsers
- Mobile: iOS 5+, Android 4+

## Next Steps

- [Async State](./async-state.md) - Fetching data with automatic loading states
- [Forms and Models](../forms-and-models.md) - Complete form binding guide
- [Events](./events.md) - Event handling with `$:click`, `$:on-*`
