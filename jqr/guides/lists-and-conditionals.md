<!-- {
  "path": "jqr/guides/lists-and-conditionals",
  "name": "docs-jqr-guides-lists-and-conditionals",
  "meta": {
    "title": "Lists & Conditionals",
    "order": 9
  }
} -->

# Lists & Conditionals

Dynamic content is at the heart of modern web applications. JQR provides `$:for` for rendering lists and `$:if`/`$:show` for conditional rendering.

## List Rendering with `$:for`

### Basic List

The `$:for` directive renders an element for each item in an array:

```html live:no-code
<div id="demo-basic-list" class="en-card en-card__body">
  <h4>Fruit List:</h4>
  <ul class="en-list">
    <li $:for="fruit in fruits" $:text="fruit"></li>
  </ul>
</div>
```

```html
<div id="demo-basic-list">
  <h4>Fruit List:</h4>
  <ul>
    <li $:for="fruit in fruits" $:text="fruit"></li>
  </ul>
</div>
```

```js inject show
$('#demo-basic-list').component({
  state: {
    fruits: ['Apple', 'Banana', 'Cherry', 'Date', 'Elderberry']
  }
});
```

```js cleanup
onUnmount(() => {
  $('#demo-basic-list').component('destroy');
})
```

**Syntax:**
```html
<element $:for="item in array"></element>
```

### With Index

Access the index of each item:

```html live:no-code
<div id="demo-list-index" class="en-card en-card__body">
  <h4>Numbered List:</h4>
  <ul class="en-list">
    <li $:for="(item, index) in items">
      <strong $:text="index + 1"></strong>. <span $:text="item"></span>
    </li>
  </ul>
</div>
```

```html
<div id="demo-list-index">
  <h4>Numbered List:</h4>
  <ul>
    <li $:for="(item, index) in items">
      <strong $:text="index + 1"></strong>. <span $:text="item"></span>
    </li>
  </ul>
</div>
```

```js inject show
$('#demo-list-index').component({
  state: {
    items: ['First item', 'Second item', 'Third item']
  }
});
```

```js cleanup
onUnmount(() => {
  $('#demo-list-index').component('destroy');
})
```

**Syntax:**
```html
<element $:for="(item, index) in array"></element>
```

### Object Iteration

Iterate over object properties:

```html live:no-code
<div id="demo-object-iteration" class="en-card en-card__body">
  <h4>User Profile:</h4>
  <dl class="en-list">
    <div $:for="(value, key) in user" class="en-mb-2">
      <dt class="en-font-bold" $:text="key"></dt>
      <dd $:text="value"></dd>
    </div>
  </dl>
</div>
```

```html
<div id="demo-object-iteration">
  <h4>User Profile:</h4>
  <dl>
    <div $:for="(value, key) in user">
      <dt $:text="key"></dt>
      <dd $:text="value"></dd>
    </div>
  </dl>
</div>
```

```js inject show
$('#demo-object-iteration').component({
  state: {
    user: {
      name: 'Alice',
      email: 'alice@example.com',
      role: 'Admin',
      status: 'Active'
    }
  }
});
```

```js cleanup
onUnmount(() => {
  $('#demo-object-iteration').component('destroy');
})
```

**Syntax:**
```html
<element $:for="(value, key) in object"></element>
<element $:for="(value, key, index) in object"></element>
```

### Number Range

Render a specific number of elements:

```html live:no-code
<div id="demo-range" class="en-card en-card__body">
  <h4>Numbers 1-5:</h4>
  <div class="en-flex en-gap-2">
    <span class="en-badge" $:for="n in 5" $:text="n + 1"></span>
  </div>
</div>
```

```html
<div id="demo-range">
  <h4>Numbers 1-5:</h4>
  <span $:for="n in 5" $:text="n + 1"></span>
</div>
```

```js inject show
$('#demo-range').component({
  state: {}
});
```

```js cleanup
onUnmount(() => {
  $('#demo-range').component('destroy');
})
```

**Syntax:**
```html
<element $:for="n in number"></element>
```

**Note:** Index starts at 0, so use `n + 1` for 1-based numbering.

## List Keys with `$:key`

### Why Keys Matter

Keys help JQR efficiently update lists by identifying which items have changed, been added, or removed.

**Without keys:**
- JQR re-creates all elements on every change
- Component state is lost
- Poor performance for large lists

**With keys:**
- JQR reuses existing elements
- Component state is preserved
- Optimal performance

### Using Keys

```html live:no-code
<div id="demo-keys" class="en-card en-card__body">
  <h4>User List:</h4>
  <button class="en-btn en-btn--sm en-mb-2" $:click="shuffle()">Shuffle Users</button>
  <ul class="en-list">
    <li $:for="user in users" $:key="user.id" class="en-flex en-gap-2 en-items-center">
      <span class="en-badge" $:text="user.id"></span>
      <strong $:text="user.name"></strong>
    </li>
  </ul>
</div>
```

```html
<div id="demo-keys">
  <h4>User List:</h4>
  <button $:click="shuffle()">Shuffle Users</button>
  <ul>
    <li $:for="user in users" $:key="user.id">
      <span $:text="user.id"></span> - <strong $:text="user.name"></strong>
    </li>
  </ul>
</div>
```

```js inject show
$('#demo-keys').component({
  state: {
    users: [
      { id: 1, name: 'Alice' },
      { id: 2, name: 'Bob' },
      { id: 3, name: 'Carol' },
      { id: 4, name: 'Dave' }
    ]
  },
  methods: {
    shuffle() {
      this.users = [...this.users].sort(() => Math.random() - 0.5);
    }
  }
});
```

```js cleanup
onUnmount(() => {
  $('#demo-keys').component('destroy');
})
```

**Syntax:**
```html
<element $:for="item in items" $:key="item.id"></element>
```

### Key Requirements

**✅ Good keys:**
- Unique among siblings
- Stable (don't change over time)
- Predictable (same item = same key)

```html
<!-- ✅ Good - unique ID -->
<li $:for="user in users" $:key="user.id"></li>

<!-- ✅ Good - unique identifier -->
<li $:for="post in posts" $:key="post.slug"></li>

<!-- ✅ Good - composite key -->
<li $:for="item in items" $:key="`${item.category}-${item.id}`"></li>
```

**❌ Bad keys:**
```html
<!-- ❌ Bad - index changes when list is reordered -->
<li $:for="(user, index) in users" $:key="index"></li>

<!-- ❌ Bad - random key breaks optimization -->
<li $:for="user in users" $:key="Math.random()"></li>

<!-- ❌ Bad - not unique -->
<li $:for="user in users" $:key="user.role"></li>
```

## Conditional Rendering

### `$:if` - True Conditional

`$:if` adds or removes elements from the DOM based on a condition:

```html live:no-code
<div id="demo-if" class="en-card en-card__body">
  <button class="en-btn en-btn--sm en-mb-3" $:click="toggle()">
    <span $:text="isVisible ? 'Hide' : 'Show'"></span> Message
  </button>
  <div class="en-alert en-alert--info" $:if="isVisible">
    <p>This element is conditionally rendered!</p>
    <p>It's completely removed from DOM when hidden.</p>
  </div>
</div>
```

```html
<div id="demo-if">
  <button $:click="toggle()">
    <span $:text="isVisible ? 'Hide' : 'Show'"></span> Message
  </button>
  <div $:if="isVisible">
    This element is conditionally rendered!
  </div>
</div>
```

```js inject show
$('#demo-if').component({
  state: { isVisible: true },
  methods: {
    toggle() {
      this.isVisible = !this.isVisible;
    }
  }
});
```

```js cleanup
onUnmount(() => {
  $('#demo-if').component('destroy');
})
```

**Behavior:**
- `true` / truthy → Element inserted into DOM
- `false` / falsy → Element removed from DOM
- Element is destroyed and re-created on toggle

**Use when:**
- Element is rarely shown
- Expensive to render
- Need to destroy component state

### `$:show` - CSS Toggle

`$:show` toggles element visibility using CSS `display` property:

```html live:no-code
<div id="demo-show" class="en-card en-card__body">
  <button class="en-btn en-btn--sm en-mb-3" $:click="toggle()">
    <span $:text="isVisible ? 'Hide' : 'Show'"></span> Message
  </button>
  <div class="en-alert en-alert--info" $:show="isVisible">
    <p>This element is hidden with CSS!</p>
    <p>It stays in DOM, just becomes display: none.</p>
  </div>
</div>
```

```html
<div id="demo-show">
  <button $:click="toggle()">
    <span $:text="isVisible ? 'Hide' : 'Show'"></span> Message
  </button>
  <div $:show="isVisible">
    This element is hidden with CSS!
  </div>
</div>
```

```js inject show
$('#demo-show').component({
  state: { isVisible: true },
  methods: {
    toggle() {
      this.isVisible = !this.isVisible;
    }
  }
});
```

```js cleanup
onUnmount(() => {
  $('#demo-show').component('destroy');
})
```

**Behavior:**
- `true` / truthy → `display` restored to original value
- `false` / falsy → `display: none` applied
- Element stays in DOM

**Use when:**
- Element toggles frequently
- Need to preserve component state
- Performance is critical (no DOM manipulation)

### `$:if` vs `$:show`

| Feature | `$:if` | `$:show` |
|---------|--------|----------|
| **DOM** | Removed/added | Always present |
| **Performance** | Higher cost | Lower cost |
| **State** | Destroyed | Preserved |
| **Initial render** | Skipped if false | Always rendered |
| **Toggle cost** | High | Low |
| **Use case** | Rare toggle | Frequent toggle |

**Example:**
```html
<!-- Use $:if for rarely shown modals -->
<div $:if="showModal" class="modal">...</div>

<!-- Use $:show for frequently toggled panels -->
<div $:show="isPanelOpen" class="panel">...</div>
```

## Combining Lists and Conditionals

### Filtering Lists

```html live:no-code
<div id="demo-filter" class="en-card en-card__body">
  <input class="en-input en-mb-3" $:model="searchQuery" placeholder="Search users...">
  <div class="en-alert en-alert--info" $:if="filteredUsers.length === 0">
    No users found matching "<span $:text="searchQuery"></span>"
  </div>
  <ul class="en-list" $:if="filteredUsers.length > 0">
    <li $:for="user in filteredUsers" $:key="user.id">
      <strong $:text="user.name"></strong> - <span $:text="user.email"></span>
    </li>
  </ul>
</div>
```

```html
<div id="demo-filter">
  <input $:model="searchQuery" placeholder="Search users...">
  <div $:if="filteredUsers.length === 0">
    No users found
  </div>
  <ul $:if="filteredUsers.length > 0">
    <li $:for="user in filteredUsers" $:key="user.id">
      <strong $:text="user.name"></strong> - <span $:text="user.email"></span>
    </li>
  </ul>
</div>
```

```js inject show
$('#demo-filter').component({
  state: {
    searchQuery: '',
    users: [
      { id: 1, name: 'Alice', email: 'alice@example.com' },
      { id: 2, name: 'Bob', email: 'bob@example.com' },
      { id: 3, name: 'Carol', email: 'carol@example.com' }
    ]
  },
  computed: {
    filteredUsers() {
      if (!this.searchQuery) return this.users;
      const query = this.searchQuery.toLowerCase();
      return this.users.filter(u => 
        u.name.toLowerCase().includes(query) ||
        u.email.toLowerCase().includes(query)
      );
    }
  }
});
```

```js cleanup
onUnmount(() => {
  $('#demo-filter').component('destroy');
})
```

### Conditional Items in Lists

```html live:no-code
<div id="demo-conditional-items" class="en-card en-card__body">
  <h4>Task List:</h4>
  <label class="en-check en-mb-3">
    <input type="checkbox" $:model="showCompleted">
    <span>Show completed tasks</span>
  </label>
  <ul class="en-list">
    <li $:for="task in tasks" 
        $:key="task.id" 
        $:if="showCompleted || !task.done"
        class="en-flex en-gap-2 en-items-center">
      <input type="checkbox" $:model="task.done">
      <span $:text="task.text" $:style="{ textDecoration: task.done ? 'line-through' : 'none' }"></span>
    </li>
  </ul>
</div>
```

```html
<div id="demo-conditional-items">
  <h4>Task List:</h4>
  <label>
    <input type="checkbox" $:model="showCompleted">
    Show completed tasks
  </label>
  <ul>
    <li $:for="task in tasks" 
        $:key="task.id" 
        $:if="showCompleted || !task.done">
      <input type="checkbox" $:model="task.done">
      <span $:text="task.text" $:style="{ textDecoration: task.done ? 'line-through' : 'none' }"></span>
    </li>
  </ul>
</div>
```

```js inject show
$('#demo-conditional-items').component({
  state: {
    showCompleted: true,
    tasks: [
      { id: 1, text: 'Buy groceries', done: false },
      { id: 2, text: 'Walk the dog', done: true },
      { id: 3, text: 'Write documentation', done: false },
      { id: 4, text: 'Review PRs', done: true }
    ]
  }
});
```

```js cleanup
onUnmount(() => {
  $('#demo-conditional-items').component('destroy');
})
```

**Note:** When using `$:if` inside `$:for`, the condition is checked for each item.

## Template Element

Use `<template>` for rendering multiple elements without a wrapper:

### Multiple Elements with `$:if`

```html live:no-code
<div id="demo-template-if" class="en-card en-card__body">
  <button class="en-btn en-btn--sm en-mb-3" $:click="hasUser = !hasUser">
    Toggle User
  </button>
  <template $:if="hasUser">
    <h4>User Profile</h4>
    <p><strong>Name:</strong> Alice</p>
    <p><strong>Role:</strong> Admin</p>
    <p><strong>Status:</strong> Active</p>
  </template>
  <p $:if="!hasUser" class="en-text--muted">No user logged in</p>
</div>
```

```html
<div id="demo-template-if">
  <button $:click="hasUser = !hasUser">Toggle User</button>
  <template $:if="hasUser">
    <h4>User Profile</h4>
    <p><strong>Name:</strong> Alice</p>
    <p><strong>Role:</strong> Admin</p>
    <p><strong>Status:</strong> Active</p>
  </template>
  <p $:if="!hasUser">No user logged in</p>
</div>
```

```js inject show
$('#demo-template-if').component({
  state: { hasUser: true }
});
```

```js cleanup
onUnmount(() => {
  $('#demo-template-if').component('destroy');
})
```

### Multiple Elements with `$:for`

```html live:no-code
<div id="demo-template-for" class="en-card en-card__body">
  <h4>Grouped Items:</h4>
  <div class="en-list">
    <template $:for="category in categories" $:key="category.name">
      <h5 class="en-font-bold en-mt-3" $:text="category.name"></h5>
      <ul class="en-list">
        <li $:for="item in category.items" $:key="item" $:text="item"></li>
      </ul>
    </template>
  </div>
</div>
```

```html
<div id="demo-template-for">
  <h4>Grouped Items:</h4>
  <template $:for="category in categories" $:key="category.name">
    <h5 $:text="category.name"></h5>
    <ul>
      <li $:for="item in category.items" $:key="item" $:text="item"></li>
    </ul>
  </template>
</div>
```

```js inject show
$('#demo-template-for').component({
  state: {
    categories: [
      { name: 'Fruits', items: ['Apple', 'Banana'] },
      { name: 'Vegetables', items: ['Carrot', 'Broccoli'] }
    ]
  }
});
```

```js cleanup
onUnmount(() => {
  $('#demo-template-for').component('destroy');
})
```

### Nested `$:if` in `<template>`

```html live:no-code
<div id="demo-nested-template" class="en-card en-card__body">
  <h4>Languages:</h4>
  <div class="en-flex en-gap-2 en-flex-wrap">
    <template $:for="lang in languages" $:key="lang.name">
      <span class="en-badge" $:if="lang.popular" $:text="lang.name"></span>
      <template $:if="!lang.popular">
        <span class="en-badge en-badge--outline" $:text="lang.name"></span>
      </template>
    </template>
  </div>
</div>
```

```html
<div id="demo-nested-template">
  <h4>Languages:</h4>
  <template $:for="lang in languages" $:key="lang.name">
    <span $:if="lang.popular" $:text="lang.name"></span>
    <template $:if="!lang.popular">
      <em $:text="lang.name"></em>
    </template>
  </template>
</div>
```

```js inject show
$('#demo-nested-template').component({
  state: {
    languages: [
      { name: 'JavaScript', popular: true },
      { name: 'Python', popular: true },
      { name: 'PHP', popular: false },
      { name: 'Ruby', popular: false }
    ]
  }
});
```

```js cleanup
onUnmount(() => {
  $('#demo-nested-template').component('destroy');
})
```

## Advanced Patterns

### Paginated List

```html live:no-code
<div id="demo-pagination" class="en-card en-card__body">
  <h4>Users (Page <span $:text="currentPage"></span>):</h4>
  <ul class="en-list en-mb-3">
    <li $:for="user in paginatedUsers" $:key="user.id" $:text="user.name"></li>
  </ul>
  <div class="en-flex en-gap-2">
    <button class="en-btn en-btn--sm" $:click="prevPage()" $:attr:disabled="currentPage === 1">
      Previous
    </button>
    <button class="en-btn en-btn--sm" $:click="nextPage()" $:attr:disabled="currentPage === totalPages">
      Next
    </button>
  </div>
  <p class="en-text--muted en-text-sm en-mt-2">
    Page <span $:text="currentPage"></span> of <span $:text="totalPages"></span>
  </p>
</div>
```

```html
<div id="demo-pagination">
  <h4>Users (Page <span $:text="currentPage"></span>):</h4>
  <ul>
    <li $:for="user in paginatedUsers" $:key="user.id" $:text="user.name"></li>
  </ul>
  <button $:click="prevPage()" $:attr:disabled="currentPage === 1">Previous</button>
  <button $:click="nextPage()" $:attr:disabled="currentPage === totalPages">Next</button>
  <p>Page <span $:text="currentPage"></span> of <span $:text="totalPages"></span></p>
</div>
```

```js inject show
$('#demo-pagination').component({
  state: {
    currentPage: 1,
    pageSize: 3,
    users: [
      { id: 1, name: 'Alice' },
      { id: 2, name: 'Bob' },
      { id: 3, name: 'Carol' },
      { id: 4, name: 'Dave' },
      { id: 5, name: 'Eve' },
      { id: 6, name: 'Frank' },
      { id: 7, name: 'Grace' }
    ]
  },
  computed: {
    totalPages: (c) => Math.ceil(c.users.length / c.pageSize),
    paginatedUsers: (c) => {
      const start = (c.currentPage - 1) * c.pageSize;
      const end = start + c.pageSize;
      return c.users.slice(start, end);
    }
  },
  methods: {
    nextPage() {
      if (this.currentPage < this.totalPages) {
        this.currentPage++;
      }
    },
    prevPage() {
      if (this.currentPage > 1) {
        this.currentPage--;
      }
    }
  }
});
```

```js cleanup
onUnmount(() => {
  $('#demo-pagination').component('destroy');
})
```

### Sorted and Filtered List

```html live:no-code
<div id="demo-sort-filter" class="en-card en-card__body">
  <div class="en-flex en-gap-2 en-mb-3">
    <input class="en-input" $:model="searchTerm" placeholder="Search...">
    <select class="en-select" $:model="sortBy">
      <option value="name">Sort by Name</option>
      <option value="age">Sort by Age</option>
    </select>
  </div>
  <ul class="en-list">
    <li $:for="person in sortedAndFiltered" $:key="person.id">
      <strong $:text="person.name"></strong> (age <span $:text="person.age"></span>)
    </li>
  </ul>
</div>
```

```html
<div id="demo-sort-filter">
  <input $:model="searchTerm" placeholder="Search...">
  <select $:model="sortBy">
    <option value="name">Sort by Name</option>
    <option value="age">Sort by Age</option>
  </select>
  <ul>
    <li $:for="person in sortedAndFiltered" $:key="person.id">
      <strong $:text="person.name"></strong> (age <span $:text="person.age"></span>)
    </li>
  </ul>
</div>
```

```js inject show
$('#demo-sort-filter').component({
  state: {
    searchTerm: '',
    sortBy: 'name',
    people: [
      { id: 1, name: 'Alice', age: 28 },
      { id: 2, name: 'Bob', age: 35 },
      { id: 3, name: 'Carol', age: 22 },
      { id: 4, name: 'Dave', age: 31 }
    ]
  },
  computed: {
    sortedAndFiltered: (c) => {
      let result = c.people;
      
      // Filter
      if (c.searchTerm) {
        const query = c.searchTerm.toLowerCase();
        result = result.filter(p => p.name.toLowerCase().includes(query));
      }
      
      // Sort
      result = [...result].sort((a, b) => {
        if (c.sortBy === 'name') {
          return a.name.localeCompare(b.name);
        } else {
          return a.age - b.age;
        }
      });
      
      return result;
    }
  }
});
```

```js cleanup
onUnmount(() => {
  $('#demo-sort-filter').component('destroy');
})
```

### Infinite Scroll

```js
$('#list').component({
  state: {
    items: [],
    page: 1,
    loading: false,
    hasMore: true
  },
  methods: {
    async loadMore() {
      if (this.loading || !this.hasMore) return;
      
      this.loading = true;
      try {
        const res = await fetch(`/api/items?page=${this.page}`);
        const data = await res.json();
        
        this.items.push(...data.items);
        this.page++;
        this.hasMore = data.hasMore;
      } finally {
        this.loading = false;
      }
    },
    handleScroll(event) {
      const el = event.target;
      const bottom = el.scrollHeight - el.scrollTop === el.clientHeight;
      
      if (bottom) {
        this.loadMore(this.$state);
      }
    }
  },
  mounted() {
    this.loadMore(this.$state);
  }
});
```

```html
<div $:on:scroll="handleScroll($event)" style="height: 400px; overflow-y: auto;">
  <div $:for="item in items" $:key="item.id">
    <p $:text="item.title"></p>
  </div>
  <div $:if="loading">Loading more...</div>
  <div $:if="!hasMore">No more items</div>
</div>
```

## Best Practices

### 1. Always Use Keys in Lists

```html
<!-- ✅ Good -->
<li $:for="user in users" $:key="user.id"></li>

<!-- ❌ Bad - inefficient updates -->
<li $:for="user in users"></li>
```

### 2. Use Computed for Filtering/Sorting

```js
// ✅ Good - cached, reactive
computed: {
  filteredItems: (c) => c.items.filter(i => i.active)
}

// ❌ Bad - runs every render
methods: {
  getFilteredItems() {
    return this.items.filter(i => i.active);
  }
}
```

### 3. Avoid Expensive Operations in Lists

```html
<!-- ❌ Bad - method call for each item -->
<li $:for="item in items" $:text="formatItem(item)"></li>

<!-- ✅ Good - prepare data in computed -->
<li $:for="item in formattedItems" $:text="item.formatted"></li>
```

### 4. Use `$:show` for Frequent Toggles

```html
<!-- ✅ Good - fast toggle -->
<div $:show="isExpanded">Frequently toggled content</div>

<!-- ⚠️ Okay for rare toggles -->
<div $:if="showModal">Modal content</div>
```

### 5. Provide Empty State

```html
<!-- ✅ Good - user feedback -->
<div $:if="items.length === 0">No items found</div>
<ul $:if="items.length > 0">
  <li $:for="item in items" $:key="item.id"></li>
</ul>

<!-- ❌ Bad - shows nothing -->
<ul>
  <li $:for="item in items" $:key="item.id"></li>
</ul>
```

## Performance Tips

### 1. Limit Rendered Items

```js
// Don't render 10,000 items at once
computed: {
  visibleItems: (c) => c.items.slice(0, 100)
}
```

### 2. Use Virtual Scrolling for Large Lists

For lists with thousands of items, consider virtual scrolling libraries or implement windowing.

### 3. Debounce Filter Input

```html
<input $:model.debounce.300="searchQuery">
```

### 4. Memoize Expensive Computations

```js
computed: {
  processedItems: (c) => {
    // Expensive operation
    return c.items.map(item => ({
      ...item,
      computed: heavyCalculation(item)
    }));
  }
}
```

## Common Pitfalls

### Direct Array Mutation

```js
// ❌ Bad - won't trigger updates
this.items[0] = newItem;

// ✅ Good - use array methods
this.items.splice(0, 1, newItem);
// or
this.items = [...this.items];
```

### Index as Key

```html
<!-- ❌ Bad - breaks on reorder -->
<li $:for="(item, index) in items" $:key="index"></li>

<!-- ✅ Good - stable unique key -->
<li $:for="item in items" $:key="item.id"></li>
```

### Missing Empty State

```html
<!-- ❌ Bad - confusing when empty -->
<ul>
  <li $:for="item in items" $:text="item"></li>
</ul>

<!-- ✅ Good - clear feedback -->
<div $:if="items.length === 0">No items to display</div>
<ul $:if="items.length > 0">
  <li $:for="item in items" $:text="item"></li>
</ul>
```

## Next Steps

- [Directive Reference](../reference/directive-reference.md) - `$:for`, `$:if`, `$:show` API
- [DOM Binding](./dom-binding.md) - Other binding techniques
- [Reactivity](./reactivity.md) - How arrays and objects stay reactive
- [State Management](./state-management.md) - Managing list data
