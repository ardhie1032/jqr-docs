<!-- {
  "path": "jqr/guides/ref",
  "name": "docs-jqr-guides-ref",
  "meta": {
    "title": "Element References ($:ref)",
    "order": 8
  }
} -->

# Element References ($:ref)

Direct access to DOM elements for cases where reactive directives aren't sufficient — focus management, measuring dimensions, or integrating third-party libraries.

## What is $:ref?

`$:ref="name"` marks an element so its jQuery object (or fallback) can be accessed directly via `this.$refs.name`, without manual querying (`document.querySelector`, etc.).

**Use ref when you need:**
- Focus management (input focus, blur)
- Measuring element dimensions (width, height, position)
- Integrating third-party libraries (charts, date pickers, WYSIWYG editors)
- Direct DOM manipulation that can't be expressed with reactive directives

**Don't use ref for:**
- Reading/writing form values (use `$:model`)
- Showing/hiding elements (use `$:if` or `$:show`)
- Setting text content (use `$:text`)
- CSS classes (use `$:class`)

## Basic Usage

```html live:no-code
<div id="demo-ref-basic" class="en-card">
  <div class="en-card__body">
    <h3>Basic Ref Usage</h3>
    <input class="en-input" $:ref="emailInput" type="email" placeholder="Enter email">
    <button class="en-btn en-btn--primary" $:click="focusInput()">Focus Input</button>
    <p class="en-text--muted" style="margin-top: 0.5rem;">Click button to focus the input field</p>
  </div>
</div>
```

```html
<div id="demo-ref-basic">
  <input $:ref="emailInput" type="email" placeholder="Enter email">
  <button $:click="focusInput()">Focus Input</button>
</div>
```

```js inject show
$('#demo-ref-basic').component({
  methods: {
    focusInput() {
      // Access element via this.$refs
      this.$refs.emailInput.trigger('focus');
    }
  }
});
```

```js cleanup
onUnmount(() => {
  $('#demo-ref-basic').component('destroy');
})
```

**What happens:**
1. `$:ref="emailInput"` registers the input element
2. `this.$refs.emailInput` is a jQuery object wrapping the input
3. Call jQuery methods directly: `.trigger('focus')`, `.val()`, etc.

## Ref Value is Literal

The attribute value (`name`) is **not evaluated as a JavaScript expression** — it's used as-is as a string key. This is the same convention used by `$:teleport:to.*`.

```html
<!-- ✅ Good - literal name -->
<input $:ref="username">
<canvas $:ref="myChart">

<!-- ❌ Wrong - don't use expressions -->
<input $:ref="$state.inputRef">  <!-- Literal key "$state.inputRef", not evaluated -->
<input $:ref="'dynamic'">        <!-- Literal key "'dynamic'", not "dynamic" -->
```

The ref name is always treated as a plain string identifier.

## Two Result Forms: Single vs Array

`this.$refs` type is `Dict<JQuery | JQuery[]>` — the form depends on **where** `$:ref` is written.

| Location | `this.$refs.name` Type | Reason |
|----------|------------------------|--------|
| Outside `$:for` | `JQuery` — single object | One element, one ref |
| Inside `$:for` template | `JQuery[]` — array, one per rendered item, ordered by current render order | Multiple elements share same ref name |

**Why array form?** If multiple items from `$:for` use the same ref name, single form would overwrite and only keep the last item. Inside `$:for`, JQR automatically collects them into an array, rebuilt from scratch on each render (safe against reorder/insert/remove — no stale entries).

### Single Ref Example

```html live:no-code
<div id="demo-ref-single" class="en-card">
  <div class="en-card__body">
    <h3>Single Ref</h3>
    <textarea class="en-input" $:ref="editor" placeholder="Type something..." rows="4"></textarea>
    <div class="en-flex en-gap-2" style="margin-top: 0.5rem;">
      <button class="en-btn en-btn--sm" $:click="measureHeight()">Measure Height</button>
      <button class="en-btn en-btn--sm en-btn--outline" $:click="clearText()">Clear</button>
    </div>
    <p class="en-text--muted" style="margin-top: 0.5rem;" $:if="height">
      Editor height: <strong $:text="height + 'px'"></strong>
    </p>
  </div>
</div>
```

```html
<div id="demo-ref-single">
  <textarea $:ref="editor" placeholder="Type something..."></textarea>
  <button $:click="measureHeight()">Measure Height</button>
  <button $:click="clearText()">Clear</button>
  <p $:if="height">Height: <span $:text="height + 'px'"></span></p>
</div>
```

```js inject show
$('#demo-ref-single').component({
  state: { height: null },
  
  methods: {
    measureHeight() {
      // this.$refs.editor is a jQuery object
      this.height = this.$refs.editor.outerHeight();
    },
    
    clearText() {
      this.$refs.editor.val('');
    }
  }
});
```

```js cleanup
onUnmount(() => {
  $('#demo-ref-single').component('destroy');
})
```

### Array Ref Example (Inside $:for)

```html live:no-code
<div id="demo-ref-array" class="en-card">
  <div class="en-card__body">
    <h3>Array Refs in $:for</h3>
    <ul class="en-list">
      <li $:for="item in items" $:key="item.id" $:ref="row" class="en-list__item">
        <span $:text="item.name"></span>
      </li>
    </ul>
    <div class="en-flex en-gap-2" style="margin-top: 0.5rem;">
      <button class="en-btn en-btn--sm" $:click="highlightAll()">Highlight All</button>
      <button class="en-btn en-btn--sm en-btn--outline" $:click="focusFirst()">Focus First</button>
      <button class="en-btn en-btn--sm en-btn--outline" $:click="logCount()">Log Count</button>
    </div>
  </div>
</div>

<style>
.highlight-row {
  background-color: #fef3c7;
  transition: background-color 0.3s ease;
}
</style>
```

```html
<div id="demo-ref-array">
  <ul>
    <li $:for="item in items" $:key="item.id" $:ref="row">
      <span $:text="item.name"></span>
    </li>
  </ul>
  <button $:click="highlightAll()">Highlight All</button>
  <button $:click="logCount()">Log Count</button>
</div>
```

```js inject show
$('#demo-ref-array').component({
  state: {
    items: [
      { id: 1, name: 'Alice' },
      { id: 2, name: 'Bob' },
      { id: 3, name: 'Charlie' }
    ]
  },
  
  methods: {
    highlightAll() {
      // this.$refs.row is an array of jQuery objects
      this.$refs.row.forEach($li => {
        $li.addClass('highlight-row');
        setTimeout(() => $li.removeClass('highlight-row'), 1000);
      });
    },
    
    focusFirst() {
      if (this.$refs.row.length > 0) {
        this.$refs.row[0].trigger('focus');
      }
    },
    
    logCount() {
      console.log('Row count:', this.$refs.row.length);
    }
  }
});
```

```js cleanup
onUnmount(() => {
  $('#demo-ref-array').component('destroy');
})
```

**Key points:**
- `this.$refs.row` is `JQuery[]` — array of jQuery objects
- Array is rebuilt on every render (safe against reorder/insert/remove)
- Array order matches current render order
- Each array element is a jQuery object wrapping one `<li>`

## When Refs Are Available

`$:ref` is populated during the bind process (part of subtree scanning, same as other directives), so **not necessarily available** before the mount cycle completes.

**Safe practice:**
- Access `this.$refs` inside `mounted()` hook, not in `constructor`/`state` initializer
- For refs inside `$:for`, remember the array is rebuilt **every render pass** — don't store old references to array elements; always re-read `this.$refs.name` when needed

```js
$('#app').component({
  state: { items: ['a', 'b'] },
  
  // ❌ Bad - refs not ready yet
  constructor() {
    console.log(this.$refs.input); // undefined
  },
  
  // ✅ Good - refs ready in mounted
  mounted() {
    console.log(this.$refs.input); // jQuery object
    this.$refs.input.trigger('focus');
  }
});
```

## Common Use Cases

### 1. Focus Management

```html live:no-code
<div id="demo-ref-focus" class="en-card">
  <div class="en-card__body">
    <h3>Auto-focus on Mount</h3>
    <div class="en-flex en-gap-2" style="margin-bottom: 1rem;">
      <input class="en-input" $:ref="nameInput" placeholder="Name (auto-focused)">
      <button class="en-btn en-btn--primary" $:click="remount()">Remount</button>
    </div>
    <p class="en-text--muted">Input is focused automatically when component mounts</p>
  </div>
</div>
```

```html
<div id="demo-ref-focus">
  <input $:ref="nameInput" placeholder="Your name">
  <button $:click="remount()">Remount</button>
</div>
```

```js inject show
let focusInstance;

function mountFocusDemo() {
  focusInstance = $('#demo-ref-focus').component({
    mounted() {
      // Auto-focus on mount
      this.$refs.nameInput.trigger('focus');
    },
    
    methods: {
      remount() {
        $('#demo-ref-focus').component('destroy');
        setTimeout(() => mountFocusDemo(), 100);
      }
    }
  });
}

mountFocusDemo();
```

```js cleanup
onUnmount(() => {
  if (focusInstance) {
    $('#demo-ref-focus').component('destroy');
  }
})
```

### 2. Measuring Dimensions

```html live:no-code
<div id="demo-ref-measure" class="en-card">
  <div class="en-card__body">
    <h3>Measure Element Dimensions</h3>
    <div $:ref="box" class="en-alert en-alert--info" style="padding: 2rem;">
      <p style="margin: 0;">This is a box to measure</p>
    </div>
    <button class="en-btn en-btn--primary" $:click="measure()" style="margin-top: 1rem;">
      Measure Box
    </button>
    <div $:if="dimensions" class="en-alert en-alert--success" style="margin-top: 1rem;">
      <p style="margin: 0;"><strong>Dimensions:</strong></p>
      <ul style="margin: 0.5rem 0 0 0; padding-left: 1.5rem;">
        <li>Width: <code $:text="dimensions.width + 'px'"></code></li>
        <li>Height: <code $:text="dimensions.height + 'px'"></code></li>
        <li>Offset Top: <code $:text="dimensions.top + 'px'"></code></li>
        <li>Offset Left: <code $:text="dimensions.left + 'px'"></code></li>
      </ul>
    </div>
  </div>
</div>
```

```html
<div id="demo-ref-measure">
  <div $:ref="box" class="box">
    This is a box to measure
  </div>
  <button $:click="measure()">Measure Box</button>
  <div $:if="dimensions">
    <p>Width: <span $:text="dimensions.width"></span>px</p>
    <p>Height: <span $:text="dimensions.height"></span>px</p>
  </div>
</div>
```

```js inject show
$('#demo-ref-measure').component({
  state: { dimensions: null },
  
  methods: {
    measure() {
      const $box = this.$refs.box;
      this.dimensions = {
        width: $box.outerWidth(),
        height: $box.outerHeight(),
        top: $box.offset().top,
        left: $box.offset().left
      };
    }
  }
});
```

```js cleanup
onUnmount(() => {
  $('#demo-ref-measure').component('destroy');
})
```

### 3. Third-Party Library Integration

```html live:no-code
<div id="demo-ref-library" class="en-card">
  <div class="en-card__body">
    <h3>Third-Party Library (Simulated)</h3>
    <canvas $:ref="chartCanvas" width="400" height="200" style="border: 1px solid #e5e7eb; border-radius: 4px;"></canvas>
    <button class="en-btn en-btn--primary" $:click="initChart()" style="margin-top: 1rem;">
      Initialize Chart
    </button>
  </div>
</div>
```

```html
<div id="demo-ref-library">
  <canvas $:ref="chartCanvas" width="400" height="200"></canvas>
  <button $:click="initChart()">Initialize Chart</button>
</div>
```

```js inject show
$('#demo-ref-library').component({
  methods: {
    initChart() {
      // Access raw DOM element with [0]
      const canvas = this.$refs.chartCanvas[0];
      const ctx = canvas.getContext('2d');
      
      // Draw simple bar chart (simulated library usage)
      ctx.clearRect(0, 0, canvas.width, canvas.height);
      ctx.fillStyle = '#3b82f6';
      ctx.fillRect(50, 150, 60, -100);
      ctx.fillRect(150, 150, 60, -70);
      ctx.fillRect(250, 150, 60, -130);
      
      ctx.fillStyle = '#1f2937';
      ctx.font = '12px sans-serif';
      ctx.fillText('Jan', 60, 170);
      ctx.fillText('Feb', 160, 170);
      ctx.fillText('Mar', 260, 170);
      
      // In real usage:
      // new Chart(canvas, { ... })
      // new Picker(canvas, { ... })
    }
  }
});
```

```js cleanup
onUnmount(() => {
  $('#demo-ref-library').component('destroy');
})
```

**Pattern for libraries:**
```js
mounted() {
  // Get raw DOM element with [0]
  const element = this.$refs.myElement[0];
  
  // Initialize library
  this.chart = new Chart(element, { /* config */ });
  this.picker = new Pikaday({ field: element });
  this.editor = new Quill(element, { /* config */ });
}

unmount() {
  // Cleanup library instances
  if (this.chart) this.chart.destroy();
  if (this.picker) this.picker.destroy();
  if (this.editor) this.editor = null;
}
```

### 4. Scroll to Element

```html live:no-code
<div id="demo-ref-scroll" class="en-card">
  <div class="en-card__body">
    <h3>Scroll to Element</h3>
    <div style="height: 200px; overflow-y: scroll; border: 1px solid #e5e7eb; border-radius: 4px; padding: 1rem;">
      <p>Scroll content...</p>
      <p>More content...</p>
      <p>Even more content...</p>
      <p>Keep scrolling...</p>
      <div $:ref="target" class="en-alert en-alert--warning">
        <p style="margin: 0;">🎯 <strong>Target Element</strong></p>
      </div>
      <p>Content after target...</p>
      <p>More content...</p>
      <p>Final content...</p>
    </div>
    <button class="en-btn en-btn--primary" $:click="scrollToTarget()" style="margin-top: 1rem;">
      Scroll to Target
    </button>
  </div>
</div>
```

```html
<div id="demo-ref-scroll">
  <div style="height: 200px; overflow-y: scroll;">
    <p>Scroll content...</p>
    <p>More content...</p>
    <div $:ref="target">🎯 Target Element</div>
    <p>Content after target...</p>
  </div>
  <button $:click="scrollToTarget()">Scroll to Target</button>
</div>
```

```js inject show
$('#demo-ref-scroll').component({
  methods: {
    scrollToTarget() {
      // Scroll element into view
      this.$refs.target[0].scrollIntoView({
        behavior: 'smooth',
        block: 'center'
      });
    }
  }
});
```

```js cleanup
onUnmount(() => {
  $('#demo-ref-scroll').component('destroy');
})
```

## Accessing Raw DOM Element

jQuery objects wrap DOM elements. To access the raw element:

```js
// Get jQuery object
const $input = this.$refs.myInput;

// Get raw DOM element
const element = $input[0];  // First element in jQuery collection
const element = $input.get(0);  // Alternative syntax

// Use raw element
element.focus();  // Native DOM method
element.scrollIntoView();
element.getBoundingClientRect();
```

**When to use raw element:**
- Passing to third-party libraries (Chart.js, Pikaday, etc.)
- Using native DOM APIs (`getBoundingClientRect`, `scrollIntoView`, etc.)
- Canvas manipulation (`getContext('2d')`)

**When to use jQuery object:**
- jQuery methods (`.trigger()`, `.addClass()`, `.css()`, etc.)
- Chaining operations
- Cross-browser compatibility

## Refs and Custom Prefixes

Like other directives, `$:ref` follows the component's prefix — `en:ref="name"` works identically in a component with `prefix: 'en'`. The key name (`name`) in `this.$refs` is **not** affected by prefix; only the attribute name changes.

```html
<div en:component:state="{}">
  <input en:ref="search">
</div>
```

```js
// this.$refs.search remains the same, prefix doesn't affect key name
this.$refs.search.trigger('focus');
```

## Limitations

### 1. One Ref Per Element

An element can only have one `$:ref` — writing two names simultaneously is not supported (attributes can only appear once per element in HTML).

```html
<!-- ❌ Not supported -->
<input $:ref="input1" $:ref="input2">

<!-- ✅ Use one name -->
<input $:ref="mainInput">
```

### 2. Not a Reactive Directive

`$:ref` is not a reactive directive: there's no `update()` that re-runs when state changes. It's purely one-time registration (or once per render pass, specifically inside `$:for`).

```html
<!-- ❌ This won't work - ref name is literal -->
<input $:ref="$state.currentRef">

<!-- ✅ Use consistent ref name -->
<input $:ref="dynamicInput">
```

### 3. Don't Use for Form Values

Don't use `$:ref` to read/write form values — that's `$:model`'s job. `$:ref` is for cases that genuinely need raw DOM elements, not as a replacement for reactive binding.

```html
<!-- ❌ Bad - manual value management -->
<input $:ref="username">
<script>
  methods: {
    submit() {
      const value = this.$refs.username.val();  // Don't do this
    }
  }
</script>

<!-- ✅ Good - reactive binding -->
<input $:model="username">
<script>
  methods: {
    submit() {
      const value = this.username;  // Reactive, clean
    }
  }
</script>
```

### 4. Array Refs Rebuild on Each Render

For refs inside `$:for`, the array is completely rebuilt on each render. Don't store references to old array elements:

```js
// ❌ Bad - storing old reference
mounted() {
  this.savedRefs = this.$refs.row;  // Old array reference
}

methods: {
  doSomething() {
    // this.savedRefs might be stale after re-render
    this.savedRefs.forEach($el => /* ... */);
  }
}

// ✅ Good - always read fresh
methods: {
  doSomething() {
    // Always access this.$refs.row directly
    this.$refs.row.forEach($el => /* ... */);
  }
}
```

## Best Practices

### 1. Use Refs Sparingly

Prefer reactive directives over refs:

```html
<!-- ❌ Avoid - manual DOM manipulation -->
<div $:ref="message"></div>
<script>
  methods: {
    updateMessage() {
      this.$refs.message.text('New message');
    }
  }
</script>

<!-- ✅ Better - reactive -->
<div $:text="message"></div>
<script>
  state: { message: 'Hello' },
  methods: {
    updateMessage() {
      this.message = 'New message';
    }
  }
</script>
```

### 2. Access in Mounted Hook

```js
// ❌ Bad - refs not ready
constructor() {
  this.$refs.input.trigger('focus');  // undefined!
}

// ✅ Good - refs ready
mounted() {
  this.$refs.input.trigger('focus');
}
```

### 3. Cleanup Third-Party Libraries

```js
mounted() {
  this.chart = new Chart(this.$refs.canvas[0], { /* ... */ });
}

unmount() {
  // Important: cleanup to prevent memory leaks
  if (this.chart) {
    this.chart.destroy();
    this.chart = null;
  }
}
```

### 4. Check Array Length

```js
methods: {
  highlightFirst() {
    // ✅ Good - check length first
    if (this.$refs.items && this.$refs.items.length > 0) {
      this.$refs.items[0].addClass('highlight');
    }
  }
}
```

### 5. Use Descriptive Names

```html
<!-- ❌ Unclear -->
<input $:ref="input1">
<input $:ref="input2">
<canvas $:ref="canvas">

<!-- ✅ Clear purpose -->
<input $:ref="emailInput">
<input $:ref="passwordInput">
<canvas $:ref="chartCanvas">
```

## Common Patterns

### Modal Auto-focus

```js
$('#modal').component({
  state: { show: false },
  
  watch: {
    show(newValue) {
      if (newValue) {
        // Focus first input when modal opens
        this.$nextTick(() => {
          this.$refs.firstInput.trigger('focus');
        });
      }
    }
  }
});
```

### Form Validation Highlight

```js
$('#form').component({
  methods: {
    async submit(state) {
      const errors = await validateForm(state);
      
      if (errors.email) {
        // Highlight and focus error field
        this.$refs.emailInput
          .addClass('error')
          .trigger('focus');
      }
    }
  }
});
```

### Infinite Scroll Sentinel

```js
$('#list').component({
  mounted() {
    const observer = new IntersectionObserver(entries => {
      if (entries[0].isIntersecting) {
        this.loadMore();
      }
    });
    
    // Observe sentinel element
    observer.observe(this.$refs.sentinel[0]);
    
    this._observer = observer;
  },
  
  unmount() {
    if (this._observer) {
      this._observer.disconnect();
    }
  }
});
```

## Performance Considerations

**Refs are lightweight:**
- No performance cost for registration
- No reactive tracking overhead
- Direct reference, no lookups

**Inside $:for:**
- Array rebuilds on each render
- Cost proportional to list size
- Generally negligible unless thousands of items

**When optimizing:**
- Cache raw DOM element if used frequently within same render cycle
- Avoid accessing refs in tight loops
- Use `$:for` `$:key` to minimize re-renders

## Next Steps

- [Instance Methods](./instance-methods.md) - Other `this.$` APIs
- [Custom Directives](./custom-directives.md) - Create your own directives
- [Lifecycle](./lifecycle.md) - Component lifecycle hooks
- [DOM Binding](./dom-binding.md) - Reactive directives

