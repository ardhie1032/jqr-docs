<!-- {
  "path": "jqr/guides/lifecycle",
  "name": "docs-jqr-guides-lifecycle",
  "meta": {
    "title": "Component Lifecycle",
    "order": 7
  }
} -->

# Component Lifecycle

JQR components have a well-defined lifecycle with hooks that let you run code at specific stages. Understanding the lifecycle is essential for proper initialization, updates, and cleanup.

## Lifecycle Stages

A component goes through these stages:

1. **Creation** - State is initialized, component options are processed
2. **Mounting** - Component is attached to DOM, directives are processed
3. **Updating** - Reactive state changes trigger DOM updates
4. **Unmounting** - Component is destroyed, cleanup runs

## Lifecycle Hooks

JQR provides four main lifecycle hooks:

### Hook Signature & ctx Parameter

All lifecycle hooks receive **`ctx` as the second argument** (after `this`):

```js
{
  mounted(ctx) {
    // Access via ctx
    console.log(ctx.state, ctx.props, ctx.el);
    ctx.emit('ready');
  }
}
```

**The `ctx` object provides:**
- `ctx.state` - Reactive state object
- `ctx.props` - Read-only props
- `ctx.methods` - All component methods
- `ctx.constants` - Static constants
- `ctx.el` - Root jQuery element  
- `ctx.emit(event, payload)` - Emit events
- `ctx.watch(source, callback)` - Watch state
- `ctx.prefix` - Directive prefix

**Two calling styles work:**

```js
// Traditional function with `this`
{
  mounted() {
    console.log(this.$state);
    this.$emit('ready');
  }
}

// Using ctx (supports arrow functions!)
{
  mounted: (ctx) => {
    console.log(ctx.state);
    ctx.emit('ready');
  }
}
```

### `mounted`

Runs **after** the component is mounted to the DOM:

```html live:no-code
<div id="mounted-demo" class="en-card">
  <div class="en-card__body">
    <h3>mounted Hook</h3>
    <div class="en-alert en-alert--info">
      <div>Component mounted at: <strong $:text="mountTime"></strong></div>
      <div>DOM ready: <strong $:text="domReady ? 'Yes' : 'No'"></strong></div>
    </div>
  </div>
</div>
```

```html
<div id="mounted-demo">
  <div>Component mounted at: <strong $:text="mountTime"></strong></div>
  <div>DOM ready: <strong $:text="domReady ? 'Yes' : 'No'"></strong></div>
</div>
```

```js inject show
$('#mounted-demo').component({
  state: {
    mountTime: '',
    domReady: false
  },
  mounted(state, $el) {
    state.mountTime = new Date().toLocaleTimeString();
    state.domReady = true;
    console.log('Component mounted!', $el);
  }
});
```

```js cleanup
onUnmount(() => {
  $('#mounted-demo').component('destroy');
})
```

**When to use:**
- Initialize third-party libraries
- Focus input elements
- Start timers or intervals
- Fetch initial data
- Measure DOM elements

### `destroyed`

Runs **before** the component is destroyed:

```html live:no-code
<div id="unmount-demo" class="en-card">
  <div class="en-card__body">
    <h3>destroyed Hook</h3>
    <div class="en-alert en-alert--info">
      <div>Seconds elapsed: <strong $:text="seconds"></strong></div>
      <div class="en-text--muted">Timer will be cleaned up on unmount</div>
    </div>
  </div>
</div>
```

```html
<div id="unmount-demo">
  <div>Seconds elapsed: <strong $:text="seconds"></strong></div>
  <div>Timer will be cleaned up on unmount</div>
</div>
```

```js inject show
$('#unmount-demo').component({
  state: { seconds: 0 },
  mounted(state) {
    this.interval = setInterval(() => {
      state.seconds++;
    }, 1000);
  },
  destroyed() {
    clearInterval(this.interval);
    console.log('Cleanup complete!');
  }
});
```

```js cleanup
onUnmount(() => {
  $('#unmount-demo').component('destroy');
})
```

**When to use:**
- Clear timers and intervals
- Remove event listeners
- Destroy third-party library instances
- Cancel pending requests
- Release resources

### `updated`

Runs **after** the DOM updates from reactive changes:

```html live:no-code
<div id="updated-demo" class="en-card">
  <div class="en-card__body">
    <h3>updated Hook</h3>
    <div class="en-flex en-gap-2" style="margin-bottom: 1rem;">
      <button class="en-btn en-btn--primary" $:click="count++">Increment</button>
    </div>
    <div class="en-alert en-alert--info">
      <div>Count: <strong $:text="count"></strong></div>
      <div>Update count: <strong $:text="updateCount"></strong></div>
    </div>
  </div>
</div>
```

```html
<div id="updated-demo">
  <button $:click="count++">Increment</button>
  <div>Count: <strong $:text="count"></strong></div>
  <div>Update count: <strong $:text="updateCount"></strong></div>
</div>
```

```js inject show
$('#updated-demo').component({
  state: {
    count: 0,
    updateCount: 0
  },
  updated(state) {
    state.updateCount++;
    console.log('DOM updated!');
  }
});
```

```js cleanup
onUnmount(() => {
  $('#updated-demo').component('destroy');
})
```

**When to use:**
- React to DOM changes
- Trigger animations after updates
- Update third-party libraries with new data
- Perform post-update measurements

**⚠️ Warning:** Avoid modifying state in `updated` as it can cause infinite loops!

## Hook Parameters

All lifecycle hooks receive useful parameters:

```js
{
  created(state) {
    // state - reactive state object
    // Called before mounting
  },
  
  mounted(state, $el) {
    // state - reactive state object
    // $el - jQuery element of component root
  },
  
  updated(state, $el) {
    // Called after each reactive update
  },
  
  destroyed(state) {
    // Called before component destruction
  }
}
```

## Complete Example: Timer Component

```html live:no-code
<div id="timer-component" class="en-card">
  <div class="en-card__body">
    <h3>Timer Component</h3>
    <div class="en-alert en-alert--success">
      <div style="font-size: 2rem; font-weight: bold; margin-bottom: 0.5rem;" $:text="formattedTime"></div>
      <div class="en-flex en-gap-2">
        <button class="en-btn en-btn--primary" $:click="toggle" $:text="running ? 'Pause' : 'Start'"></button>
        <button class="en-btn en-btn--outline" $:click="reset">Reset</button>
      </div>
    </div>
    <div class="en-text--muted">Timer automatically cleans up on unmount</div>
  </div>
</div>
```

```html
<div id="timer-component">
  <h3>Timer Component</h3>
  <div $:text="formattedTime"></div>
  <button $:click="toggle" $:text="running ? 'Pause' : 'Start'"></button>
  <button $:click="reset">Reset</button>
</div>
```

```js inject show
const timer = $('#timer-component').component({
  state: {
    seconds: 0,
    running: false
  },
  
  computed: {
    formattedTime(state) {
      const mins = Math.floor(state.seconds / 60);
      const secs = state.seconds % 60;
      return `${mins.toString().padStart(2, '0')}:${secs.toString().padStart(2, '0')}`;
    }
  },
  
  methods: {
    toggle(state) {
      state.running = !state.running;
      if (state.running) {
        timer.startTimer();
      } else {
        timer.stopTimer();
      }
    },
    
    reset(state) {
      state.seconds = 0;
      state.running = false;
      timer.stopTimer();
    }
  },
  
  mounted() {
    console.log('Timer component mounted');
  },
  
  destroyed() {
    timer.stopTimer();
    console.log('Timer cleaned up');
  }
});

// Helper functions outside component
timer.interval = null;

timer.startTimer = function() {
  if (!this.interval) {
    this.interval = setInterval(() => {
      this.state.seconds++;
    }, 1000);
  }
};

timer.stopTimer = function() {
  if (this.interval) {
    clearInterval(this.interval);
    this.interval = null;
  }
};
```

```js cleanup
onUnmount(() => {
  $('#timer-component').component('destroy');
})
```

## Async Operations in Lifecycle

### Fetching Data on Mount

```html live:no-code
<div id="fetch-demo" class="en-card">
  <div class="en-card__body">
    <h3>Fetch on Mount</h3>
    <div $:show="loading" class="en-alert en-alert--info">
      Loading users...
    </div>
    <div $:show="!loading && users.length > 0">
      <div $:for="user in users.slice(0, 3)" $:key="user.id" class="en-flex en-gap-2 en-items-center" style="margin-bottom: 0.5rem;">
        <img $:attr:src="user.avatar" $:attr:alt="user.name" style="width: 32px; height: 32px; border-radius: 50%;">
        <div>
          <strong $:text="user.name"></strong>
          <div class="en-text--muted en-text--sm" $:text="user.email"></div>
        </div>
      </div>
    </div>
  </div>
</div>
```

```html
<div id="fetch-demo">
  <h3>Fetch on Mount</h3>
  <div $:show="loading">Loading users...</div>
  <div $:show="!loading">
    <div $:for="user in users" $:key="user.id">
      <img $:attr:src="user.avatar" $:attr:alt="user.name">
      <strong $:text="user.name"></strong>
      <div $:text="user.email"></div>
    </div>
  </div>
</div>
```

```js inject show
$('#fetch-demo').component({
  state: {
    users: [],
    loading: true
  },
  
  async mounted(state) {
    try {
      const response = await fetch('/assets/api/jqr-docs/users.json');
      state.users = await response.json();
    } catch (error) {
      console.error('Failed to fetch users:', error);
    } finally {
      state.loading = false;
    }
  }
});
```

```js cleanup
onUnmount(() => {
  $('#fetch-demo').component('destroy');
})
```

### Canceling Requests on Unmount

```js
const fetchComponent = $('#element').component({
  state: {
    data: null,
    loading: false
  },
  
  mounted(state) {
    fetchComponent.abortController = new AbortController();
    
    fetch('/api/data', { 
      signal: fetchComponent.abortController.signal 
    })
      .then(res => res.json())
      .then(data => state.data = data)
      .catch(err => {
        if (err.name !== 'AbortError') {
          console.error('Fetch failed:', err);
        }
      });
  },
  
  destroyed() {
    // Cancel pending request
    fetchComponent.abortController.abort();
  }
});
```

## Third-Party Library Integration

### Initializing Libraries

```html
<div id="chart-component">
  <canvas id="myChart"></canvas>
</div>
```

```js
const chartComponent = $('#chart-component').component({
  state: {
    data: [10, 20, 30, 40, 50]
  },
  
  mounted(state, $el) {
    // Initialize Chart.js (example)
    const canvas = $el.find('#myChart')[0];
    chartComponent.chart = new Chart(canvas, {
      type: 'line',
      data: {
        labels: ['Jan', 'Feb', 'Mar', 'Apr', 'May'],
        datasets: [{
          label: 'Sales',
          data: state.data
        }]
      }
    });
  },
  
  updated(state) {
    // Update chart when data changes
    if (chartComponent.chart) {
      chartComponent.chart.data.datasets[0].data = state.data;
      chartComponent.chart.update();
    }
  },
  
  destroyed() {
    // Cleanup
    if (chartComponent.chart) {
      chartComponent.chart.destroy();
    }
  }
});
```

## Access to Component Instance

The component method returns an instance that you can store and access:

```js
const myComponent = $('#element').component({
  state: { count: 0 },
  
  mounted(state, $el) {
    // Store data on the component instance
    myComponent.myInterval = setInterval(() => {
      state.count++;
    }, 1000);
  },
  
  destroyed() {
    // Access stored data from instance
    clearInterval(myComponent.myInterval);
  }
});

// Access from outside
console.log(myComponent.state.count);
myComponent.myMethod();
```

Inside methods, you can also access the instance via the method context:

```js
const timer = $('#element').component({
  methods: {
    start(state) {
      // Access component instance
      timer.customData = 'some value';
      timer.startInterval();
    }
  }
});

timer.startInterval = function() {
  this.interval = setInterval(() => {
    this.state.count++;
  }, 1000);
};
```

## Lifecycle Flow Diagram

```
Component Creation
       ↓
   Initialize State
       ↓
   Process Options
       ↓
  ┌─→ created() ────────────┐
  │         ↓                │
  │    Mount to DOM          │
  │         ↓                │
  │    mounted() ────────────┤
  │                          │
  │   Reactive Updates       │
  │         ↓                │
  │    DOM Updates           │
  │         ↓                │
  └──── updated() ←──────────┘
              
              ↓
       User triggers destroy
              ↓
        destroyed()
              ↓
        Cleanup & Remove
```

## Best Practices

### 1. Always Clean Up Resources

```js
// ✅ Good
const myComponent = $('#element').component({
  state: { count: 0 },
  
  mounted(state) {
    myComponent.interval = setInterval(() => {
      state.count++;
    }, 1000);
  },
  
  destroyed() {
    clearInterval(myComponent.interval);
  }
});

// ❌ Bad - memory leak!
$('#element').component({
  mounted(state) {
    setInterval(() => {
      state.count++;
    }, 1000);
  }
  // No cleanup!
});
```

### 2. Use Async/Await in mounted

```js
// ✅ Good
async mounted(state) {
  state.loading = true;
  try {
    const data = await fetchData();
    state.data = data;
  } catch (error) {
    state.error = error.message;
  } finally {
    state.loading = false;
  }
}

// ❌ Avoid nested promises
mounted(state) {
  fetchData()
    .then(data => {
      state.data = data;
    })
    .catch(error => {
      state.error = error;
    });
}
```

### 3. Don't Mutate State in updated

```js
// ❌ Bad - infinite loop!
updated(state) {
  state.updateCount++; // This triggers another update!
}

// ✅ Good - read-only operations
updated(state, $el) {
  const height = $el.height();
  console.log('Component height:', height);
}
```

### 4. Store Cleanup References

```js
// ✅ Good
const myComponent = $('#element').component({
  mounted() {
    myComponent.listeners = [];
    
    const resizeHandler = () => myComponent.handleResize();
    $(window).on('resize', resizeHandler);
    myComponent.listeners.push(() => {
      $(window).off('resize', resizeHandler);
    });
  },
  
  destroyed() {
    myComponent.listeners.forEach(cleanup => cleanup());
  }
});
```

### 5. Use mounted for DOM Operations

```js
// ✅ Good - DOM is ready
$('#element').component({
  mounted(state, $el) {
    $el.find('input').first().focus();
    const height = $el.height();
  }
});

// ❌ Bad - DOM might not be ready in methods
$('#element').component({
  methods: {
    init(state, $el) {
      // This works when called from mounted
      // but might fail if called too early
      $el.find('input').focus();
    }
  }
});
```

## Common Patterns

### Auto-save Form

```html
<div id="autosave-form">
  <input $:model="title" placeholder="Title">
  <textarea $:model="content" placeholder="Content"></textarea>
  <div $:show="saving">Saving...</div>
  <div $:show="saved">Saved!</div>
</div>
```

```js
const autosave = $('#autosave-form').component({
  state: {
    title: '',
    content: '',
    saving: false,
    saved: false
  },
  
  mounted(state) {
    // Auto-save on changes
    autosave.$watch(['title', 'content'], () => {
      autosave.save();
    });
  },
  
  destroyed() {
    clearTimeout(autosave.saveTimeout);
  }
});

autosave.save = async function() {
  this.state.saving = true;
  this.state.saved = false;
  
  // Debounce save
  clearTimeout(this.saveTimeout);
  this.saveTimeout = setTimeout(async () => {
    await fetch('/api/save', {
      method: 'POST',
      body: JSON.stringify({
        title: this.state.title,
        content: this.state.content
      })
    });
    this.state.saving = false;
    this.state.saved = true;
  }, 1000);
};
```

### WebSocket Connection

```js
const wsDemo = $('#websocket-demo').component({
  state: {
    messages: [],
    connected: false
  },
  
  mounted(state) {
    wsDemo.ws = new WebSocket('ws://localhost:8080');
    
    wsDemo.ws.onopen = () => {
      state.connected = true;
    };
    
    wsDemo.ws.onmessage = (event) => {
      state.messages.push(JSON.parse(event.data));
    };
    
    wsDemo.ws.onerror = (error) => {
      console.error('WebSocket error:', error);
    };
  },
  
  destroyed() {
    if (wsDemo.ws) {
      wsDemo.ws.close();
    }
  }
});
```

### Polling Data

```js
const polling = $('#polling-demo').component({
  state: {
    data: null,
    lastUpdate: null
  },
  
  mounted(state) {
    // Initial fetch
    polling.fetchData();
    
    // Poll every 5 seconds
    polling.pollInterval = setInterval(() => {
      polling.fetchData();
    }, 5000);
  },
  
  destroyed() {
    clearInterval(polling.pollInterval);
  }
});

polling.fetchData = async function() {
  const response = await fetch('/api/data');
  const data = await response.json();
  this.state.data = data;
  this.state.lastUpdate = new Date();
};
```

## Debugging Lifecycle

Log lifecycle events to understand component behavior:

```js
const debugComponent = $('#debug-component').component({
  state: { count: 0 },
  
  created(state) {
    console.log('🔨 Created', { state });
  },
  
  mounted(state, $el) {
    console.log('✅ Mounted', { state, $el });
  },
  
  updated(state, $el) {
    console.log('🔄 Updated', { state });
  },
  
  destroyed(state) {
    console.log('❌ Destroyed');
  }
});
```

## Next Steps

- [Instance Methods](./instance-methods.md) - Component API methods
- [Async State](./async-state.md) - Declarative async data fetching
- [Component Options](./component-options.md) - Complete component configuration

