<!-- {
  "path": "jqr/guides/async-state",
  "name": "docs-jqr-guides-async-state",
  "meta": {
    "title": "Async State",
    "order": 6
  }
} -->

# Async State

JQR's `asyncState` feature makes it easy to fetch data from APIs and handle loading/error states automatically.

## How It Works

When you define an `asyncState` function, JQR automatically creates **three state properties**:

1. **`key`** - The fetched data (initially `undefined`)
2. **`keyLoading`** - Loading state (`true` while fetching)
3. **`keyError`** - Error state (set if the request fails)

```js
$('#app').component({
  asyncState: {
    async users(state, props) {
      const response = await fetch('/api/users');
      return response.json();
    }
  }
});

// Automatically creates:
// - state.users (undefined → data)
// - state.usersLoading (true → false)
// - state.usersError (undefined or error object)
```

## Basic Example

```html live:no-code
<div id="basic-async" class="en-card">
  <div class="en-card__body">
    <h3>Fetch Random User</h3>
    <button class="en-btn en-btn--primary" $:click="fetchUser">
      Fetch User
    </button>
    
    <div class="en-alert en-alert--info" style="margin-top: 1rem;" $:show="userLoading">
      Loading...
    </div>
    
    <div class="en-alert en-alert--danger" style="margin-top: 1rem;" $:show="userError">
      Error: <span $:text="userError?.message"></span>
    </div>
    
    <div class="en-card" style="margin-top: 1rem;" $:show="user && !userLoading">
      <div class="en-card__body">
        <h4 $:text="user?.name"></h4>
        <p $:text="user?.email"></p>
      </div>
    </div>
  </div>
</div>
```

```html
<div id="basic-async">
  <h3>Fetch Random User</h3>
  <button $:click="fetchUser">Fetch User</button>
  
  <div $:show="userLoading">Loading...</div>
  <div $:show="userError">
    Error: <span $:text="userError?.message"></span>
  </div>
  
  <div $:show="user && !userLoading">
    <h4 $:text="user?.name"></h4>
    <p $:text="user?.email"></p>
  </div>
</div>
```

```js inject show
$('#basic-async').component({
  asyncState: {
    async user() {
      // Simulated API call
      await new Promise(resolve => setTimeout(resolve, 1000));
      return {
        name: 'John Doe',
        email: 'john@example.com'
      };
    }
  },
  methods: {
    fetchUser() {
      this.refetch('user');
    }
  }
});
```

```js cleanup
onUnmount(() => {
  $('#basic-async').component('destroy');
})
```

## Reactive Dependencies

The `asyncState` function receives `state` and `props`. Reading from them is tracked - when values change, the function automatically re-runs:

```html live:no-code
<div id="reactive-async" class="en-card">
  <div class="en-card__body">
    <h3>Search Users</h3>
    <input 
      class="en-input" 
      $:model.debounce.500="query" 
      placeholder="Type to search...">
    
    <div class="en-alert en-alert--info" style="margin-top: 1rem;" $:show="resultsLoading">
      Searching...
    </div>
    
    <div style="margin-top: 1rem;" $:show="results && !resultsLoading">
      <div class="en-text--muted" $:show="results?.length === 0">
        No results found
      </div>
      <div 
        $:for="user in results" 
        $:key="user.id"
        class="en-alert en-alert--muted"
        style="margin-bottom: 0.5rem;">
        <strong $:text="user.name"></strong>
      </div>
    </div>
  </div>
</div>
```

```html
<div id="reactive-async">
  <h3>Search Users</h3>
  <input 
    $:model.debounce.500="query" 
    placeholder="Type to search...">
  
  <div $:show="resultsLoading">Searching...</div>
  
  <div $:show="results && !resultsLoading">
    <div $:show="results?.length === 0">No results</div>
    <div $:for="user in results" $:key="user.id">
      <strong $:text="user.name"></strong>
    </div>
  </div>
</div>
```

```js inject show
$('#reactive-async').component({
  state: { query: '' },
  asyncState: {
    async results(state) {
      if (!state.query) return [];
      
      // Simulated search
      await new Promise(resolve => setTimeout(resolve, 500));
      const allUsers = [
        { id: 1, name: 'Alice Johnson' },
        { id: 2, name: 'Bob Smith' },
        { id: 3, name: 'Charlie Brown' }
      ];
      
      return allUsers.filter(u => 
        u.name.toLowerCase().includes(state.query.toLowerCase())
      );
    }
  }
});
```

```js cleanup
onUnmount(() => {
  $('#reactive-async').component('destroy');
})
```

**How it works:**
- Reading `state.query` inside `results()` tracks it as a dependency
- When `query` changes, `results()` automatically re-runs
- Each new request cancels the previous one (stale requests are ignored)

## Working with Real APIs

### Fetch API

```js
asyncState: {
  async users() {
    const response = await fetch('/assets/api/jqr-docs/users.json');
    if (!response.ok) throw new Error('Failed to fetch');
    return response.json();
  }
}
```

### jQuery AJAX

```js
asyncState: {
  users() {
    // jQuery's $.get returns a jqXHR (PromiseLike)
    return $.get('/assets/api/jqr-docs/users.json');
  }
}
```

### Axios

```js
asyncState: {
  async users() {
    const response = await axios.get('/assets/api/jqr-docs/users.json');
    return response.data;
  }
}
```

## Complete Example: User List

```html live:no-code
<div id="user-list" class="en-card">
  <div class="en-card__body">
    <h3>User Directory</h3>
    
    <div class="en-flex en-gap-2" style="margin-bottom: 1rem;">
      <input 
        class="en-input" 
        $:model.debounce.300="filter" 
        placeholder="Filter by name...">
      <button class="en-btn en-btn--outline" $:click="refreshUsers">
        Refresh
      </button>
    </div>
    
    <div class="en-alert en-alert--info" $:show="usersLoading">
      Loading users...
    </div>
    
    <div class="en-alert en-alert--danger" $:show="usersError">
      Failed to load users. 
      <button class="en-btn en-btn--sm en-btn--danger" $:click="refreshUsers">
        Retry
      </button>
    </div>
    
    <div $:show="filteredUsers && !usersLoading">
      <div class="en-text--muted" $:show="filteredUsers.length === 0">
        No users found
      </div>
      <div 
        $:for="user in filteredUsers" 
        $:key="user.id"
        class="en-card"
        style="margin-bottom: 0.5rem;">
        <div class="en-card__body">
          <strong $:text="user.name"></strong>
          <div class="en-text--muted" $:text="user.email"></div>
        </div>
      </div>
    </div>
  </div>
</div>
```

```html
<div id="user-list">
  <h3>User Directory</h3>
  
  <input 
    $:model.debounce.300="filter" 
    placeholder="Filter by name...">
  <button $:click="refreshUsers">Refresh</button>
  
  <div $:show="usersLoading">Loading...</div>
  <div $:show="usersError">
    Error. <button $:click="refreshUsers">Retry</button>
  </div>
  
  <div $:show="filteredUsers && !usersLoading">
    <div $:show="filteredUsers.length === 0">No users</div>
    <div $:for="user in filteredUsers" $:key="user.id">
      <strong $:text="user.name"></strong>
      <div $:text="user.email"></div>
    </div>
  </div>
</div>
```

```js inject show
$('#user-list').component({
  state: { filter: '' },
  asyncState: {
    async users() {
      await new Promise(resolve => setTimeout(resolve, 800));
      return [
        { id: 1, name: 'Alice Johnson', email: 'alice@example.com' },
        { id: 2, name: 'Bob Smith', email: 'bob@example.com' },
        { id: 3, name: 'Charlie Brown', email: 'charlie@example.com' },
        { id: 4, name: 'Diana Prince', email: 'diana@example.com' }
      ];
    }
  },
  computed: {
    filteredUsers(state) {
      if (!state.users) return [];
      if (!state.filter) return state.users;
      
      const q = state.filter.toLowerCase();
      return state.users.filter(u => 
        u.name.toLowerCase().includes(q) ||
        u.email.toLowerCase().includes(q)
      );
    }
  },
  methods: {
    refreshUsers() {
      this.refetch('users');
    }
  }
});
```

```js cleanup
onUnmount(() => {
  $('#user-list').component('destroy');
})
```

## Manual Refetch: `refetch()`

Force a manual re-run of an async fetcher:

```js
methods: {
  refresh() {
    this.refetch('users'); // Re-runs the 'users' fetcher
  }
}
```

```html
<button $:click="refresh">Refresh</button>
```

**Use cases:**
- Refresh button
- Retry after error
- Manual data reload

## Cancel In-Flight Requests: `cancelAsync()`

Cancel a pending request without re-running:

```js
methods: {
  cancel() {
    this.cancelAsync('users');
    // usersLoading becomes false
    // When the request eventually settles, it's ignored
  }
}
```

## Error Handling

Errors are automatically caught and stored in `{key}Error`:

```js
asyncState: {
  async users() {
    const response = await fetch('/assets/api/jqr-docs/users.json');
    if (!response.ok) {
      throw new Error(`HTTP ${response.status}`);
    }
    return response.json();
  }
}
```

Display errors:

```html
<div $:show="usersError" class="error">
  Error: <span $:text="usersError?.message"></span>
  <button $:click="retryFetch">Retry</button>
</div>
```

```js
methods: {
  retryFetch() {
    this.refetch('users');
  }
}
```

## Loading States

Show loading indicators:

```html
<!-- Simple loading -->
<div $:show="usersLoading">Loading...</div>

<!-- Conditional content -->
<div $:show="!usersLoading && users">
  <div $:for="user in users">...</div>
</div>

<!-- Disable buttons while loading -->
<button $:attr:disabled="usersLoading" $:click="submit">
  <span $:show="!usersLoading">Submit</span>
  <span $:show="usersLoading">Submitting...</span>
</button>
```

## Multiple Async States

Define multiple async fetchers:

```js
asyncState: {
  async users() {
    return fetch('/assets/api/jqr-docs/users.json').then(r => r.json());
  },
  async posts() {
    return fetch('/assets/api/jqr-docs/posts.json').then(r => r.json());
  },
  async comments() {
    return fetch('/assets/api/jqr-docs/comments.json').then(r => r.json());
  }
}
// Creates: users, usersLoading, usersError
//          posts, postsLoading, postsError
//          comments, commentsLoading, commentsError
```

## Dependent Requests

Make one request depend on another:

```js
state: { userId: null },
asyncState: {
  async user(state) {
    if (!state.userId) return null;
    const res = await fetch(`/assets/api/jqr-docs/users.json`);
    const users = await res.json();
    return users.find(u => u.id === state.userId);
  },
  async posts(state) {
    // Reads state.user - automatically refetches when user changes
    if (!state.user) return [];
    const res = await fetch(`/assets/api/jqr-docs/posts.json`);
    const allPosts = await res.json();
    return allPosts.filter(p => p.userId === state.user.id);
  }
}
```

## Props as Dependencies

Use `props` in async functions:

```js
$.jqr.register('user-card', {
  props: ['userId'],
  asyncState: {
    async user(state, props) {
      // Reading props.userId tracks it
      const res = await fetch(`/assets/api/jqr-docs/users.json`);
      const users = await res.json();
      return users.find(u => u.id === props.userId);
    }
  }
});
```

When `userId` prop changes, `user()` automatically refetches.

## Best Practices

### 1. Handle Empty States

```js
asyncState: {
  async results(state) {
    if (!state.query) return []; // Don't fetch if no query
    const res = await fetch(`/assets/api/jqr-docs/posts.json`);
    const posts = await res.json();
    return posts.filter(p => 
      p.title.toLowerCase().includes(state.query.toLowerCase())
    );
  }
}
```

### 2. Debounce User Input

```html
<!-- Prevents refetch on every keystroke -->
<input $:model.debounce.500="query">
```

### 3. Show Meaningful Loading States

```html
<!-- ❌ Bad: flickering for fast requests -->
<div $:show="usersLoading">Loading...</div>

<!-- ✅ Better: only show after delay -->
<div $:show="usersLoading && slowLoad">Loading...</div>
```

### 4. Always Handle Errors

```html
<div $:show="usersError">
  Something went wrong. <button $:click="retryUsers">Retry</button>
</div>
```

```js
methods: {
  retryUsers() {
    this.refetch('users');
  }
}
```

### 5. Provide Retry Mechanism

```js
methods: {
  retry(state) {
    state.usersError = undefined; // Clear error first
    this.refetch('users');
  }
}
```

```html
<button $:click="retry">Retry</button>
```

## Stale Request Handling

JQR automatically handles stale requests:

```js
asyncState: {
  async results(state) {
    // User types "cat" → request A starts
    // User types "dog" before A finishes → request B starts
    // Request A finishes → ignored (stale)
    // Request B finishes → used
    const res = await fetch(`/assets/api/jqr-docs/posts.json`);
    const posts = await res.json();
    return posts.filter(p => 
      p.title.toLowerCase().includes(state.query.toLowerCase())
    );
  }
}
```

**No race conditions** - only the latest request matters.

## Limitations

### Not for Mutations

`asyncState` is for **fetching data**, not posting/updating:

```js
// ❌ Don't use asyncState for mutations
asyncState: {
  async submitForm(state) {
    return fetch('/assets/api/jqr-docs/save.json', { 
      method: 'POST', 
      body: JSON.stringify(state) 
    });
  }
}

// ✅ Use methods instead
methods: {
  async submit(state) {
    state.isSubmitting = true;
    try {
      await fetch('/assets/api/jqr-docs/save.json', { 
        method: 'POST', 
        body: JSON.stringify(state) 
      });
      state.success = true;
    } catch (err) {
      state.error = err.message;
    } finally {
      state.isSubmitting = false;
    }
  }
}
```

### Runs Immediately

`asyncState` runs **eagerly** when the component mounts:

```js
// This fetches immediately, not on first read
asyncState: {
  async users() { /* ... */ }
}
```

If you need lazy loading, use a method + manual state management.

## Next Steps

- [Model Modifiers](./model-modifiers.md) - Advanced form binding
- [Forms and Models](./forms-and-models.md) - Two-way data binding
- [Component Options](./component-options.md) - Full component API
