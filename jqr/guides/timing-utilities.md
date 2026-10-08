<!-- {
  "path": "jqr/guides/timing-utilities",
  "name": "docs-jqr-guides-timing-utilities",
  "meta": {
    "title": "Timing Utilities",
    "order": 15
  }
} -->

# Timing Utilities

JQR provides timing utilities for controlling when code executes: `nextTick()` for scheduling after DOM updates, and `debounce()`/`throttle()` for rate-limiting function calls.

## `$.jqr.nextTick()`

Schedule a callback to run after the current reactive update queue has finished flushing.

### When to Use

- Read DOM after state changes (measure dimensions, scroll position, etc.)
- Run code after all pending updates complete
- Ensure third-party libraries see updated DOM

### How It Works

JQR batches reactive updates using a microtask queue. Multiple state changes in the same synchronous tick trigger only one render per effect. `nextTick()` waits until this queue (including cascading updates) fully settles.

### Basic Usage

```html live:no-code
<div id="demo-nexttick" class="en-card">
  <div class="en-card__body">
    <h3>nextTick Example</h3>
    <button class="en-btn en-btn--primary" $:click="updateAndMeasure">
      Update & Measure
    </button>
    <div 
      id="demo-nexttick-box" 
      class="en-card" 
      style="margin-top: 1rem; transition: all 0.3s;"
      $:style="{ height: height + 'px' }">
      <div class="en-card__body">
        <p $:text="message"></p>
        <p class="en-text--muted en-text-sm">
          Measured height: <strong $:text="measuredHeight"></strong>px
        </p>
      </div>
    </div>
  </div>
</div>
```

```html
<div id="demo-nexttick">
  <h3>nextTick Example</h3>
  <button $:click="updateAndMeasure">Update & Measure</button>
  <div $:style="{ height: height + 'px' }" id="demo-nexttick-box">
    <p $:text="message"></p>
    <p>Measured: <strong $:text="measuredHeight"></strong>px</p>
  </div>
</div>
```

```js inject show
$('#demo-nexttick').component({
  state: {
    message: 'Short message',
    height: 100,
    measuredHeight: 0
  },
  methods: {
    async updateAndMeasure() {
      // Update state
      this.message = 'This is a much longer message that will change the height of the container significantly.';
      this.height = 150;
      
      // ❌ This would read OLD height (before DOM update)
      // const height = document.getElementById('demo-nexttick-box').offsetHeight;
      
      // ✅ Wait for DOM to update first
      await this.$nextTick();
      const box = document.getElementById('demo-nexttick-box');
      if(!box) return;

      box.addEventListener('transitionend', () => {
        this.measuredHeight = box.offsetHeight;
      }, { once: true });
    }
  }
});
```

```js cleanup
onUnmount(() => {
  $('#demo-nexttick').component('destroy');
})
```

### Common Patterns

**Read DOM after update:**
```js
methods: {
  addItem(state) {
    state.items.push({ id: Date.now(), name: 'New Item' });
    
    $.jqr.nextTick(() => {
      const list = this.$el.find('.item-list')[0];
      list.scrollTop = list.scrollHeight; // Scroll to bottom
    });
  }
}
```

**Integrate third-party libraries:**
```js
mounted(state, $el) {
  state.code = 'console.log("hello")';
  
  $.jqr.nextTick(() => {
    // DOM is ready, initialize syntax highlighter
    const pre = $el.find('pre code')[0];
    hljs.highlightElement(pre);
  });
}
```

**Focus input after showing:**
```js
methods: {
  showModal(state) {
    state.modalVisible = true;
    
    $.jqr.nextTick(() => {
      // Modal is now in DOM
      this.$el.find('.modal input')[0]?.focus();
    });
  }
}
```

### Async/Await Support

`nextTick()` returns a Promise, so you can use `await`:

```js
async updateData() {
  this.loading = true;
  const data = await fetchData();
  this.data = data;
  this.loading = false;
  
  // Wait for loading spinner to disappear and data to render
  await this.$nextTick();
  
  // Now safe to measure rendered content
  const height = this.$el.find('.data-container')[0].offsetHeight;
  console.log('Content height:', height);
}
```

### Cascading Updates

`nextTick()` waits for **all cascading updates** to settle:

```js
methods: {
  complexUpdate() {
    this.x = 1; // Triggers effects that depend on x
    
    this.$nextTick(() => {
      // Waits for:
      // 1. Initial update queue (x = 1)
      // 2. Any watch() callbacks that run and mutate more state
      // 3. Effects triggered by those mutations
      // ... until nothing is left queued
      console.log('Everything settled!');
    });
  }
}
```

## `$.jqr.debounce()`

Create a debounced function that delays execution until `delay` milliseconds have passed since the **last call**. Perfect for "wait until user stops typing" scenarios.

### Signature

```typescript
$.jqr.debounce<T extends (...args: any[]) => void>(
  fn: T,
  delay: number
): T & { cancel: () => void }
```

**Returns:** Debounced function with `.cancel()` method

### How It Works

Every call resets the timer. Only the **final call** in a burst actually executes (with that call's arguments).

```
Calls:     1   2   3         (burst stops)      Execute!
Time:  ----•---•---•-------------|--------------->
                                 └─ delay ms ────┘
```

### Basic Usage

```html live:no-code
<div id="demo-debounce" class="en-card">
  <div class="en-card__body">
    <h3>Debounce Example</h3>
    <div class="en-form-group">
      <label class="en-form-label">Search (debounced):</label>
      <input 
        class="en-input" 
        $:model="query"
        placeholder="Type to search..."
        $:on:input="handleSearch">
    </div>
    <div class="en-alert en-alert--info" style="margin-top: 1rem;">
      <div>Query: <strong $:text="query || '(empty)'"></strong></div>
      <div class="en-text-sm en-text--muted">Search executed: <span $:text="searchCount"></span> times</div>
    </div>
    <p class="en-text--muted en-text-sm">
      Function only executes 500ms after you stop typing
    </p>
  </div>
</div>
```

```html
<div id="demo-debounce">
  <h3>Debounce Example</h3>
  <label>Search (debounced):</label>
  <input $:model="query" $:on:input="handleSearch" placeholder="Search...">
  <div>
    <div>Query: <strong $:text="query"></strong></div>
    <div>Executed: <span $:text="searchCount"></span> times</div>
  </div>
</div>
```

```js inject show
$('#demo-debounce').component({
  state: {
    query: '',
    searchCount: 0
  },
  created() {
    // Create debounced function once
    this.debouncedSearch = this.$debounce((query) => {
      // console.log('Searching for:', query);
      this.searchCount++;
    }, 500);
  },
  methods: {
    handleSearch() {
      this.debouncedSearch(this.query);
    }
  },
  destroyed() {
    // Cancel pending calls on cleanup
    this.debouncedSearch.cancel();
  }
});
```

```js cleanup
onUnmount(() => {
  $('#demo-debounce').component('destroy');
})
```

### Common Patterns

**Search-as-you-type:**
```js
created() {
  this.searchAPI = $.jqr.debounce(async (query) => {
    const results = await fetch(`/api/search?q=${query}`);
    this.results = await results.json();
  }, 300);
},
methods: {
  onSearchInput() {
    this.searchAPI(this.searchQuery);
  }
}
```

**Auto-save:**
```js
created() {
  this.autoSave = this.$debounce((data) => {
    localStorage.setItem('draft', JSON.stringify(data));
    this.lastSaved = new Date();
  }, 1000);
},
methods: {
  onContentChange() {
    this.autoSave(this.content);
  }
}
```

**Window resize handler:**
```js
mounted() {
  this.handleResize = $.jqr.debounce(() => {
    this.state.windowWidth = window.innerWidth;
    this.state.windowHeight = window.innerHeight;
  }, 200);
  
  window.addEventListener('resize', this.handleResize);
},
destroyed() {
  window.removeEventListener('resize', this.handleResize);
  this.handleResize.cancel();
}
```

### Cancel Pending Calls

```js
// Cancel any pending execution
this.debouncedFn.cancel();

// Useful when:
// - Component is being destroyed
// - User navigates away
// - Form is reset
```

## `$.jqr.throttle()`

Create a throttled function that executes at most once per `delay` milliseconds window. Uses **leading + trailing edge** execution.

### Signature

```typescript
$.jqr.throttle<T extends (...args: any[]) => void>(
  fn: T,
  delay: number
): T & { cancel: () => void }
```

**Returns:** Throttled function with `.cancel()` method

### How It Works

- **Leading edge:** First call fires immediately
- **Trailing edge:** If calls arrive mid-window, the **last one** fires when window ends
- No calls are silently dropped

```
Calls:     1   2   3   4       5   6
Time:  ----•---•---•---•-------•---•---------->
       Execute¹    (wait)  Execute⁴   Execute⁶
       └─ delay ──┘        └─ delay ──┘
```

### Basic Usage

```html live:no-code
<div id="demo-throttle" class="en-card">
  <div class="en-card__body">
    <h3>Throttle Example</h3>
    <div class="en-form-group">
      <label class="en-form-label">Move mouse over box:</label>
      <div 
        class="en-card" 
        style="height: 200px; cursor: crosshair;"
        $:on:mousemove="handleMouseMove">
        <div class="en-card__body">
          <p>Mouse position (throttled):</p>
          <p>X: <strong $:text="mouseX"></strong>, Y: <strong $:text="mouseY"></strong></p>
          <p class="en-text--muted en-text-sm">Updates: <span $:text="updateCount"></span></p>
        </div>
      </div>
    </div>
    <p class="en-text--muted en-text-sm">
      Position updates max once per 100ms
    </p>
  </div>
</div>
```

```html
<div id="demo-throttle">
  <h3>Throttle Example</h3>
  <div style="height: 200px;" $:on:mousemove="handleMouseMove">
    <p>Mouse position (throttled):</p>
    <p>X: <strong $:text="mouseX"></strong>, Y: <strong $:text="mouseY"></strong></p>
    <p>Updates: <span $:text="updateCount"></span></p>
  </div>
</div>
```

```js inject show
$('#demo-throttle').component({
  state: {
    mouseX: 0,
    mouseY: 0,
    updateCount: 0
  },
  created() {
    this.throttledUpdate = this.$throttle((x, y) => {
      this.mouseX = x;
      this.mouseY = y;
      this.updateCount++;
    }, 100);
  },
  methods: {
    handleMouseMove(event) {
      this.throttledUpdate(event.clientX, event.clientY);
    }
  },
  destroyed() {
    this.throttledUpdate.cancel();
  }
});
```

```js cleanup
onUnmount(() => {
  $('#demo-throttle').component('destroy');
})
```

### Common Patterns

**Scroll handler:**
```js
mounted() {
  this.handleScroll = this.$throttle(() => {
    this.scrollY = window.scrollY;
    this.isNearBottom = (
      window.innerHeight + window.scrollY >= 
      document.body.offsetHeight - 100
    );
  }, 200);
  
  window.addEventListener('scroll', this.handleScroll);
},
destroyed() {
  window.removeEventListener('scroll', this.handleScroll);
  this.handleScroll.cancel();
}
```

**API rate limiting:**
```js
created() {
  this.trackEvent = this.$throttle((eventName, data) => {
    fetch('/api/analytics', {
      method: 'POST',
      body: JSON.stringify({ event: eventName, data })
    });
  }, 5000); // Max 1 request per 5 seconds
},
methods: {
  onClick() {
    this.trackEvent('button_click', { button: 'submit' });
  }
}
```

**Game loop / animation:**
```js
created() {
  this.gameUpdate = this.$throttle(() => {
    this.player.x += this.velocity.x;
    this.player.y += this.velocity.y;
    this.checkCollisions();
  }, 16); // ~60 FPS
},
mounted() {
  this.animationId = requestAnimationFrame(this.gameLoop);
},
methods: {
  gameLoop() {
    this.gameUpdate();
    this.animationId = requestAnimationFrame(this.gameLoop);
  }
}
```

## Debounce vs Throttle

Choose based on your use case:

| Use Case | Choose | Why |
|----------|--------|-----|
| Search-as-you-type | **Debounce** | Wait for user to stop typing |
| Auto-save | **Debounce** | Save after editing pause |
| Window resize | **Debounce** | Wait for resize to complete |
| Scroll tracking | **Throttle** | Regular position updates |
| Mouse movement | **Throttle** | Consistent sampling rate |
| API rate limiting | **Throttle** | Enforce max calls per period |
| Button spam prevention | **Throttle** | Allow first click immediately |

### Visualization

```
Input:     •••••••••••••••••           •••••

Debounce:                    Execute      Execute
           └──── wait ────┘              └─ wait ─┘

Throttle:  Execute  Execute  Execute   Execute
           └─ wait ─┘└─ wait ─┘        └─ wait ─┘
```

## Model Modifiers vs Manual Utilities

For `$:model` inputs, prefer built-in modifiers over manual utilities:

```html
<!-- ✅ Good: Built-in modifier -->
<input $:model.debounce.300="search">

<!-- ❌ Less efficient: Manual debouncing -->
<input $:model="search" $:on:input="debouncedSearch">
```

**Why modifiers are better:**
- Optimized for model binding
- Automatic cleanup
- Less boilerplate
- Better performance

See [Model Modifiers Guide](./model-modifiers.md) for details.

## Performance Considerations

### Create Once, Reuse

```js
// ✅ Good: Create in created/mounted
created() {
  this.debouncedSave = this.$debounce(this.save, 1000);
}

// ❌ Bad: Create on every call
methods: {
  onInput() {
    const debounced = this.$debounce(this.save, 1000); // New function every time!
    debounced(this.data);
  }
}
```

### Always Cleanup

```js
destroyed() {
  // Cancel pending calls
  this.debouncedFn?.cancel();
  this.throttledFn?.cancel();
  
  // Remove event listeners
  window.removeEventListener('scroll', this.handleScroll);
}
```

### Memory Leaks

```js
// ❌ Bad: Debounced function holds reference to old state
this.save = this.$debounce(() => {
  api.save(this.data); // Captures state reference!
}, 1000);

// ✅ Good: Pass data as arguments
this.save = this.$debounce((data) => {
  api.save(data);
}, 1000);

// Call with current data
this.save(this.data);
```

## TypeScript Support

Both utilities are fully typed:

```typescript
import { jqr } from '@en32/jqr';

// Type inference works
const save = jqr.debounce((data: UserData) => {
  api.save(data);
}, 1000);

save({ name: 'Alice' }); // ✅ Type-checked
save.cancel(); // ✅ Method exists

// Explicit typing
const search: ((query: string) => void) & { cancel: () => void } = 
  jqr.throttle((query: string) => {
    console.log(query);
  }, 300);
```

## Browser Compatibility

All utilities use standard APIs:
- `setTimeout` / `clearTimeout`
- `queueMicrotask` (ES2021, widely supported)
- `Promise` (ES2015)

**Supported:** All modern browsers (Chrome, Firefox, Safari, Edge)

## See Also

- [Performance Guide](./performance.md) - Optimization techniques
- [Model Modifiers](./model-modifiers.md) - Debounce/throttle for inputs
- [Async State](./async-state.md) - Data fetching patterns
- [Configuration Reference](../reference/config-reference.md) - All utility APIs
