<!-- {
  "path": "jqr/examples/recipes",
  "name": "docs-jqr-examples-recipes",
  "meta": {
    "title": "Common Patterns",
    "order": 5
  }
} -->

# Common Patterns & Recipes

A collection of reusable patterns and solutions for common scenarios in JQR applications.

## Table of Contents

- [Form Patterns](#form-patterns)
- [Data Loading](#data-loading)
- [UI Patterns](#ui-patterns)
- [State Management](#state-management)
- [Performance](#performance)
- [Integration](#integration)

---

## Form Patterns

### Auto-save Form

Automatically save form changes after a delay:

```html
<form id="demo-autosave">
  <input $:model.debounce:1000="title" placeholder="Title">
  <textarea $:model.debounce:1000="content" placeholder="Content"></textarea>
  
  <div $:if="saving">Saving...</div>
  <div $:if="lastSaved">
    Last saved: <span $:text="lastSaved"></span>
  </div>
</form>
```

```js
$('#demo-autosave').component({
  state: {
    title: '',
    content: '',
    saving: false,
    lastSaved: null
  },
  
  watch: {
    async title(newVal, oldVal) {
      if (oldVal !== undefined) {
        await this.save(this);
      }
    },
    
    async content(newVal, oldVal) {
      if (oldVal !== undefined) {
        await this.save(this);
      }
    }
  },
  
  methods: {
    async save(state) {
      state.saving = true;
      
      try {
        await fetch('/api/save', {
          method: 'POST',
          headers: { 'Content-Type': 'application/json' },
          body: JSON.stringify({
            title: state.title,
            content: state.content
          })
        });
        
        state.lastSaved = new Date().toLocaleTimeString();
      } catch (error) {
        console.error('Save failed:', error);
      } finally {
        state.saving = false;
      }
    }
  }
});
```

### Form with Dirty Check

Warn users about unsaved changes:

```html
<form id="demo-dirty-check">
  <input $:model="name" placeholder="Name">
  <input $:model="email" placeholder="Email">
  
  <div $:if="isDirty">
    <strong>You have unsaved changes!</strong>
  </div>
  
  <button $:click="save" $:disabled="!isDirty">Save</button>
  <button $:click="reset" $:disabled="!isDirty">Reset</button>
</form>
```

```js
$('#demo-dirty-check').component({
  state: {
    name: '',
    email: '',
    originalData: null
  },
  
  computed: {
    isDirty(state) {
      if (!state.originalData) return false;
      
      return state.name !== state.originalData.name ||
             state.email !== state.originalData.email;
    }
  },
  
  created() {
    this.originalData = {
      name: this.name,
      email: this.email
    };
    
    // Warn on page unload
    window.addEventListener('beforeunload', this._handleBeforeUnload);
  },
  
  methods: {
    _handleBeforeUnload(state, event) {
      if (this.isDirty(state)) {
        event.preventDefault();
        event.returnValue = '';
      }
    },
    
    async save(state) {
      await fetch('/api/save', {
        method: 'POST',
        body: JSON.stringify({ name: state.name, email: state.email })
      });
      
      // Update original data
      state.originalData = {
        name: state.name,
        email: state.email
      };
    },
    
    reset(state) {
      state.name = state.originalData.name;
      state.email = state.originalData.email;
    }
  },
  
  unmount() {
    window.removeEventListener('beforeunload', this._handleBeforeUnload);
  }
});
```

### Multi-step Form

```html
<div id="demo-multistep">
  <div $:if="step === 1">
    <h3>Step 1: Personal Info</h3>
    <input $:model="name" placeholder="Name">
    <input $:model="email" placeholder="Email">
    <button $:click="nextStep">Next</button>
  </div>
  
  <div $:if="step === 2">
    <h3>Step 2: Address</h3>
    <input $:model="address" placeholder="Address">
    <input $:model="city" placeholder="City">
    <button $:click="prevStep">Back</button>
    <button $:click="nextStep">Next</button>
  </div>
  
  <div $:if="step === 3">
    <h3>Step 3: Review</h3>
    <p>Name: <span $:text="name"></span></p>
    <p>Email: <span $:text="email"></span></p>
    <p>Address: <span $:text="address"></span></p>
    <p>City: <span $:text="city"></span></p>
    <button $:click="prevStep">Back</button>
    <button $:click="submit">Submit</button>
  </div>
  
  <!-- Progress indicator -->
  <div>
    Step <span $:text="step"></span> of 3
  </div>
</div>
```

```js
$('#demo-multistep').component({
  state: {
    step: 1,
    name: '',
    email: '',
    address: '',
    city: ''
  },
  
  methods: {
    nextStep(state) {
      if (state.step < 3) {
        state.step++;
      }
    },
    
    prevStep(state) {
      if (state.step > 1) {
        state.step--;
      }
    },
    
    async submit(state) {
      await fetch('/api/submit', {
        method: 'POST',
        body: JSON.stringify({
          name: state.name,
          email: state.email,
          address: state.address,
          city: state.city
        })
      });
      
      alert('Form submitted!');
      state.step = 1;
    }
  }
});
```

---

## Data Loading

### Infinite Scroll

Load more items as user scrolls:

```html
<div id="demo-infinite-scroll">
  <div $:for="item in items" $:key="item.id">
    <h4 $:text="item.title"></h4>
    <p $:text="item.body"></p>
  </div>
  
  <div $:ref="sentinel" style="height: 20px;"></div>
  
  <div $:if="loading">Loading more...</div>
  <div $:if="!hasMore">No more items</div>
</div>
```

```js
$('#demo-infinite-scroll').component({
  state: {
    items: [],
    page: 1,
    loading: false,
    hasMore: true
  },
  
  async mounted() {
    await this.loadMore(this);
    
    // Set up intersection observer
    const observer = new IntersectionObserver(
      (entries) => {
        if (entries[0].isIntersecting && !this.loading && this.hasMore) {
          this.loadMore(this);
        }
      },
      { threshold: 0.1 }
    );
    
    observer.observe(this.$refs.sentinel[0]);
    
    // Cleanup
    this._observer = observer;
  },
  
  methods: {
    async loadMore(state) {
      if (state.loading || !state.hasMore) return;
      
      state.loading = true;
      
      try {
        const response = await fetch(`/api/items?page=${state.page}`);
        const data = await response.json();
        
        state.items.push(...data.items);
        state.page++;
        state.hasMore = data.hasMore;
      } finally {
        state.loading = false;
      }
    }
  },
  
  unmount() {
    if (this._observer) {
      this._observer.disconnect();
    }
  }
});
```

### Retry Failed Requests

```html
<div id="demo-retry">
  <div $:if="$loading.data">Loading...</div>
  
  <div $:if="$error.data">
    <p>Error: <span $:text="$error.data.message"></span></p>
    <button $:click="() => $refetch('data')">Retry</button>
  </div>
  
  <div $:if="$data.data">
    <p $:text="$data.data.message"></p>
  </div>
</div>
```

```js
$('#demo-retry').component({
  asyncState: {
    async data() {
      const maxRetries = 3;
      let lastError;
      
      for (let attempt = 1; attempt <= maxRetries; attempt++) {
        try {
          const response = await fetch('/api/data');
          
          if (!response.ok) {
            throw new Error(`HTTP ${response.status}`);
          }
          
          return await response.json();
        } catch (error) {
          lastError = error;
          
          if (attempt < maxRetries) {
            // Wait before retry (exponential backoff)
            await new Promise(resolve => 
              setTimeout(resolve, Math.pow(2, attempt) * 1000)
            );
          }
        }
      }
      
      // All retries failed
      throw lastError;
    }
  }
});
```

### Polling for Updates

```html
<div id="demo-polling">
  <div $:if="$loading.status">Checking status...</div>
  
  <div $:if="$data.status">
    <p>Status: <span $:text="$data.status.state"></span></p>
    <p>Updated: <span $:text="$data.status.timestamp"></span></p>
  </div>
</div>
```

```js
$('#demo-polling').component({
  state: {
    pollingInterval: null
  },
  
  asyncState: {
    async status() {
      const response = await fetch('/api/status');
      return response.json();
    }
  },
  
  mounted() {
    // Poll every 5 seconds
    this.pollingInterval = setInterval(() => {
      this.$refetch('status');
    }, 5000);
  },
  
  unmount() {
    if (this.pollingInterval) {
      clearInterval(this.pollingInterval);
    }
  }
});
```

---

## UI Patterns

### Dropdown Menu

```html
<div id="demo-dropdown">
  <button $:click="toggleMenu">
    Menu <span $:text="isOpen ? '▲' : '▼'"></span>
  </button>
  
  <ul $:if="isOpen" $:click.stop>
    <li $:click="selectItem('Profile')">Profile</li>
    <li $:click="selectItem('Settings')">Settings</li>
    <li $:click="selectItem('Logout')">Logout</li>
  </ul>
</div>
```

```js
$('#demo-dropdown').component({
  state: {
    isOpen: false
  },
  
  mounted() {
    // Close on outside click
    this._handleClickOutside = (event) => {
      if (!this.$el[0].contains(event.target)) {
        this.isOpen = false;
      }
    };
    
    document.addEventListener('click', this._handleClickOutside);
  },
  
  methods: {
    toggleMenu(state) {
      state.isOpen = !state.isOpen;
    },
    
    selectItem(state, item) {
      alert(`Selected: ${item}`);
      state.isOpen = false;
    }
  },
  
  unmount() {
    document.removeEventListener('click', this._handleClickOutside);
  }
});
```

### Tabs

```html
<div id="demo-tabs">
  <div>
    <button 
      $:for="tab in tabs"
      $:key="tab.id"
      $:click="() => selectTab(tab.id)"
      $:class="{ active: activeTab === tab.id }"
      $:text="tab.label"></button>
  </div>
  
  <div>
    <div $:if="activeTab === 'home'">Home content</div>
    <div $:if="activeTab === 'profile'">Profile content</div>
    <div $:if="activeTab === 'settings'">Settings content</div>
  </div>
</div>
```

```js
$('#demo-tabs').component({
  state: {
    activeTab: 'home',
    tabs: [
      { id: 'home', label: 'Home' },
      { id: 'profile', label: 'Profile' },
      { id: 'settings', label: 'Settings' }
    ]
  },
  
  methods: {
    selectTab(state, tabId) {
      state.activeTab = tabId;
    }
  }
});
```

### Toast Notifications

```html
<div id="demo-toasts">
  <button $:click="() => addToast('success', 'Success!')">
    Show Success
  </button>
  <button $:click="() => addToast('error', 'Error!')">
    Show Error
  </button>
  
  <div style="position: fixed; top: 20px; right: 20px;">
    <div 
      $:for="toast in toasts" 
      $:key="toast.id"
      $:class="`toast toast-${toast.type}`">
      <span $:text="toast.message"></span>
      <button $:click="() => removeToast(toast.id)">×</button>
    </div>
  </div>
</div>
```

```js
$('#demo-toasts').component({
  state: {
    toasts: [],
    nextId: 1
  },
  
  methods: {
    addToast(state, type, message) {
      const id = state.nextId++;
      
      state.toasts.push({ id, type, message });
      
      // Auto-remove after 3 seconds
      setTimeout(() => {
        this.removeToast(state, id);
      }, 3000);
    },
    
    removeToast(state, id) {
      const index = state.toasts.findIndex(t => t.id === id);
      if (index > -1) {
        state.toasts.splice(index, 1);
      }
    }
  }
});
```

### Accordion

```html
<div id="demo-accordion">
  <div $:for="item in items" $:key="item.id">
    <button $:click="() => toggle(item.id)">
      <span $:text="item.title"></span>
      <span $:text="openItems.includes(item.id) ? '−' : '+'"></span>
    </button>
    
    <div $:if="openItems.includes(item.id)">
      <p $:text="item.content"></p>
    </div>
  </div>
</div>
```

```js
$('#demo-accordion').component({
  state: {
    openItems: [],
    items: [
      { id: 1, title: 'Section 1', content: 'Content 1...' },
      { id: 2, title: 'Section 2', content: 'Content 2...' },
      { id: 3, title: 'Section 3', content: 'Content 3...' }
    ]
  },
  
  methods: {
    toggle(state, itemId) {
      const index = state.openItems.indexOf(itemId);
      if (index > -1) {
        state.openItems.splice(index, 1);
      } else {
        state.openItems.push(itemId);
      }
    }
  }
});
```

---

## State Management

### Shared State Across Components

```js
// Create shared store
const userStore = $.jqr.reactive({
  user: null,
  isAuthenticated: false,
  
  login(userData) {
    this.user = userData;
    this.isAuthenticated = true;
  },
  
  logout() {
    this.user = null;
    this.isAuthenticated = false;
  }
});

// Use in components
$('#header').component({
  state: { store: userStore },
  
  computed: {
    userName: (state) => state.store.user?.name || 'Guest'
  }
});

$('#sidebar').component({
  state: { store: userStore },
  
  computed: {
    isLoggedIn: (state) => state.store.isAuthenticated
  }
});
```

### Undo/Redo

```html
<div id="demo-undo">
  <input $:model="text" placeholder="Type something...">
  
  <button $:click="undo" $:disabled="!canUndo">Undo</button>
  <button $:click="redo" $:disabled="!canRedo">Redo</button>
  
  <p $:text="text"></p>
</div>
```

```js
$('#demo-undo').component({
  state: {
    text: '',
    history: [''],
    historyIndex: 0
  },
  
  computed: {
    canUndo: (state) => state.historyIndex > 0,
    canRedo: (state) => state.historyIndex < state.history.length - 1
  },
  
  watch: {
    text(newVal) {
      // Add to history
      const newHistory = this.history.slice(0, this.historyIndex + 1);
      newHistory.push(newVal);
      this.history = newHistory;
      this.historyIndex = newHistory.length - 1;
    }
  },
  
  methods: {
    undo(state) {
      if (state.historyIndex > 0) {
        state.historyIndex--;
        state.text = state.history[state.historyIndex];
      }
    },
    
    redo(state) {
      if (state.historyIndex < state.history.length - 1) {
        state.historyIndex++;
        state.text = state.history[state.historyIndex];
      }
    }
  }
});
```

### Local Storage Persistence

```html
<div id="demo-persist">
  <input $:model="notes" placeholder="Your notes...">
  <p>Notes are automatically saved</p>
</div>
```

```js
$('#demo-persist').component({
  state: {
    notes: localStorage.getItem('notes') || ''
  },
  
  watch: {
    notes(newVal) {
      localStorage.setItem('notes', newVal);
    }
  }
});
```

---

## Performance

### Virtual Scrolling (Large Lists)

```html
<div id="demo-virtual-scroll" style="height: 400px; overflow-y: auto;">
  <div $:style="{ height: totalHeight + 'px', position: 'relative' }">
    <div 
      $:for="item in visibleItems"
      $:key="item.index"
      $:style="{ 
        position: 'absolute',
        top: item.offset + 'px',
        height: itemHeight + 'px'
      }">
      Item <span $:text="item.index"></span>
    </div>
  </div>
</div>
```

```js
$('#demo-virtual-scroll').component({
  state: {
    scrollTop: 0,
    itemHeight: 50,
    containerHeight: 400,
    totalItems: 10000
  },
  
  computed: {
    totalHeight(state) {
      return state.totalItems * state.itemHeight;
    },
    
    visibleItems(state) {
      const startIndex = Math.floor(state.scrollTop / state.itemHeight);
      const visibleCount = Math.ceil(state.containerHeight / state.itemHeight);
      const endIndex = Math.min(startIndex + visibleCount + 1, state.totalItems);
      
      const items = [];
      for (let i = startIndex; i < endIndex; i++) {
        items.push({
          index: i,
          offset: i * state.itemHeight
        });
      }
      return items;
    }
  },
  
  mounted() {
    this.$el.on('scroll', () => {
      this.scrollTop = this.$el.scrollTop();
    });
  }
});
```

### Lazy Load Images

```html
<div id="demo-lazy-images">
  <img 
    $:for="image in images"
    $:key="image.id"
    $:ref="lazyImage"
    $:attr:data-src="image.url"
    src="placeholder.jpg"
    alt="">
</div>
```

```js
$('#demo-lazy-images').component({
  state: {
    images: [
      { id: 1, url: 'image1.jpg' },
      { id: 2, url: 'image2.jpg' },
      // ...
    ]
  },
  
  mounted() {
    const observer = new IntersectionObserver((entries) => {
      entries.forEach(entry => {
        if (entry.isIntersecting) {
          const img = entry.target;
          img.src = img.dataset.src;
          observer.unobserve(img);
        }
      });
    });
    
    this.$refs.lazyImage.forEach(img => {
      observer.observe(img[0]);
    });
    
    this._observer = observer;
  },
  
  unmount() {
    if (this._observer) {
      this._observer.disconnect();
    }
  }
});
```

### Debounced Computed

```js
$('#app').component({
  state: {
    searchQuery: '',
    debouncedQuery: ''
  },
  
  watch: {
    searchQuery: $.jqr.debounce(function(newVal) {
      this.debouncedQuery = newVal;
    }, 500)
  },
  
  asyncState: {
    async results(state) {
      if (!state.debouncedQuery) return [];
      
      const response = await fetch(`/api/search?q=${state.debouncedQuery}`);
      return response.json();
    }
  }
});
```

---

## Integration

### jQuery Plugin Integration

```html
<div id="demo-datepicker">
  <input $:ref="dateInput" $:model="selectedDate">
  <p>Selected: <span $:text="selectedDate || 'None'"></span></p>
</div>
```

```js
$('#demo-datepicker').component({
  state: {
    selectedDate: ''
  },
  
  mounted() {
    // Initialize jQuery UI datepicker
    this.$refs.dateInput.datepicker({
      dateFormat: 'yy-mm-dd',
      onSelect: (dateText) => {
        this.selectedDate = dateText;
      }
    });
  },
  
  unmount() {
    // Clean up
    this.$refs.dateInput.datepicker('destroy');
  }
});
```

### Chart.js Integration

```html
<div id="demo-chart">
  <canvas $:ref="chartCanvas"></canvas>
  
  <button $:click="updateData">Update Data</button>
</div>
```

```js
$('#demo-chart').component({
  state: {
    data: [12, 19, 3, 5, 2, 3]
  },
  
  mounted() {
    const ctx = this.$refs.chartCanvas[0].getContext('2d');
    
    this._chart = new Chart(ctx, {
      type: 'bar',
      data: {
        labels: ['Red', 'Blue', 'Yellow', 'Green', 'Purple', 'Orange'],
        datasets: [{
          label: '# of Votes',
          data: this.data
        }]
      }
    });
  },
  
  watch: {
    data(newData) {
      if (this._chart) {
        this._chart.data.datasets[0].data = newData;
        this._chart.update();
      }
    }
  },
  
  methods: {
    updateData(state) {
      state.data = state.data.map(() => Math.floor(Math.random() * 20));
    }
  },
  
  unmount() {
    if (this._chart) {
      this._chart.destroy();
    }
  }
});
```

### WebSocket Integration

```html
<div id="demo-websocket">
  <div $:if="connected">Connected</div>
  <div $:if="!connected">Disconnected</div>
  
  <div $:for="message in messages" $:key="message.id">
    <span $:text="message.text"></span>
  </div>
  
  <input $:model="newMessage" placeholder="Type message...">
  <button $:click="sendMessage">Send</button>
</div>
```

```js
$('#demo-websocket').component({
  state: {
    connected: false,
    messages: [],
    newMessage: '',
    nextId: 1
  },
  
  created() {
    this._ws = new WebSocket('wss://echo.websocket.org');
    
    this._ws.onopen = () => {
      this.connected = true;
    };
    
    this._ws.onmessage = (event) => {
      this.messages.push({
        id: this.nextId++,
        text: event.data
      });
    };
    
    this._ws.onclose = () => {
      this.connected = false;
    };
  },
  
  methods: {
    sendMessage(state) {
      if (state.newMessage && this._ws) {
        this._ws.send(state.newMessage);
        state.newMessage = '';
      }
    }
  },
  
  unmount() {
    if (this._ws) {
      this._ws.close();
    }
  }
});
```

---

## Best Practices

### 1. Component Composition

Break complex components into smaller, reusable pieces:

```html
<div id="app">
  <user-card $:for="user in users" $:key="user.id" $:bind:user="user"></user-card>
</div>
```

```js
$('#app').component({
  components: {
    'user-card': {
      template: '#user-card-template',
      props: ['user'],
      // Component logic...
    }
  }
});
```

### 2. Error Boundaries

Wrap error-prone sections:

```html
<div id="app">
  <div $:if="!$error.data">
    <!-- Normal content -->
  </div>
  
  <div $:if="$error.data">
    <h3>Something went wrong</h3>
    <p $:text="$error.data.message"></p>
    <button $:click="() => $refetch('data')">Try Again</button>
  </div>
</div>
```

### 3. Loading States

Always show loading states:

```html
<div $:if="$loading.data">Loading...</div>
<div $:if="!$loading.data && $data.data">
  <!-- Content -->
</div>
```

### 4. Cleanup Resources

Clean up in `unmount`:

```js
unmount() {
  // Clear timers
  if (this._timer) clearInterval(this._timer);
  
  // Remove event listeners
  document.removeEventListener('click', this._handler);
  
  // Disconnect observers
  if (this._observer) this._observer.disconnect();
  
  // Close connections
  if (this._ws) this._ws.close();
}
```

---

## Related Documentation

- [Todo App](./todo-app.md) - Complete app example
- [Data Table](./data-table.md) - Table with sorting/pagination
- [AJAX Form](./ajax-form.md) - Form submission patterns
- [Async State Guide](../guides/async-state.md) - Data loading
- [State Management Guide](../guides/state-management.md) - State patterns

