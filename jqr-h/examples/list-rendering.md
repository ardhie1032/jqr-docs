<!-- {
  "path": "jqr-h/examples/list-rendering",
  "name": "docs-jqr-h-examples-list-rendering",
  "meta": {
    "title": "List Rendering",
    "order": 3
  }
} -->

# Example: List Rendering with h.for

Demonstrates list rendering patterns using `h.for()`, including filtering, sorting, keyed rendering, nested lists, and dynamic CRUD operations.

## Features

- ✅ Keyed list rendering with `h.for`
- ✅ Add/remove items
- ✅ Filtering and sorting
- ✅ Nested components in lists
- ✅ Efficient DOM updates
- ✅ Search functionality

## Live Demo

```html live:no-code
<div id="demo-list-rendering"></div>
```

```html
<div id="demo-list-rendering"></div>
```

```js inject show
import { mount } from '@en32/jqr-h';

// User Card Component (nested)
const UserCard = {
  props: {
    user: null,
    onDelete: null
  },
  render: (h, ctx) => h('div', {
    class: 'en-card en-mb-2'
  }, [
    h('div', { class: 'en-card__body en-flex en-gap-3 en-flex-between' }, [
      // User Info
      h('div', {}, [
        h('h4', {
          class: 'en-mb-1',
          text: (c) => c.$props.user.name
        }),
        h('p', {
          class: 'en-text-sm en-text-muted',
          text: (c) => c.$props.user.email
        }),
        h('span', {
          class: (c) => ['en-badge', {
            'en-badge--success': c.$props.user.role === 'admin',
            'en-badge--primary': c.$props.user.role === 'user'
          }],
          text: (c) => c.$props.user.role
        })
      ]),
      
      // Actions
      h('div', {}, [
        h('button', {
          class: 'en-btn en-btn--sm en-btn--danger',
          on: {
            click: (c) => c.$props.onDelete(c.$props.user.id)
          }
        }, ['Delete'])
      ])
    ])
  ])
};

// Main App
mount('#demo-list-rendering', {
  state: {
    users: [
      { id: 1, name: 'Alice Johnson', email: 'alice@example.com', role: 'admin', active: true },
      { id: 2, name: 'Bob Smith', email: 'bob@example.com', role: 'user', active: true },
      { id: 3, name: 'Charlie Brown', email: 'charlie@example.com', role: 'user', active: false },
      { id: 4, name: 'Diana Prince', email: 'diana@example.com', role: 'admin', active: true },
      { id: 5, name: 'Eve Wilson', email: 'eve@example.com', role: 'user', active: true }
    ],
    searchQuery: '',
    filterRole: 'all',
    sortBy: 'name',
    newUser: {
      name: '',
      email: '',
      role: 'user'
    }
  },
  computed: {
    filteredAndSortedUsers() {
      let result = this.users;
      
      // Filter by search
      if (this.searchQuery) {
        const query = this.searchQuery.toLowerCase();
        result = result.filter(u => 
          u.name.toLowerCase().includes(query) ||
          u.email.toLowerCase().includes(query)
        );
      }
      
      // Filter by role
      if (this.filterRole !== 'all') {
        result = result.filter(u => u.role === this.filterRole);
      }
      
      // Sort
      result = [...result].sort((a, b) => {
        if (this.sortBy === 'name') {
          return a.name.localeCompare(b.name);
        } else if (this.sortBy === 'email') {
          return a.email.localeCompare(b.email);
        } else if (this.sortBy === 'role') {
          return a.role.localeCompare(b.role);
        }
        return 0;
      });
      
      return result;
    },
    userCount() {
      return this.filteredAndSortedUsers.length;
    },
    canAddUser() {
      return this.newUser.name.trim() && this.newUser.email.trim();
    }
  },
  methods: {
    deleteUser(id) {
      this.users = this.users.filter(u => u.id !== id);
    },
    addUser() {
      if (!this.canAddUser) return;
      
      const newId = Math.max(...this.users.map(u => u.id), 0) + 1;
      this.users.push({
        id: newId,
        name: this.newUser.name.trim(),
        email: this.newUser.email.trim(),
        role: this.newUser.role,
        active: true
      });
      
      // Reset form
      this.newUser.name = '';
      this.newUser.email = '';
      this.newUser.role = 'user';
    }
  },
  render: (h, ctx) => h('div', { class: 'en-card' }, [
    // Header
    h('div', { class: 'en-card__header' }, [
      h('h3', {}, ['User Management']),
      h('p', {
        class: 'en-text-sm en-text-muted',
        text: (c) => `Showing ${c.userCount} of ${c.users.length} users`
      })
    ]),
    
    // Body
    h('div', { class: 'en-card__body' }, [
      // Add User Form
      h('div', { class: 'en-mb-4' }, [
        h('h4', { class: 'en-mb-2' }, ['Add New User']),
        h('div', { class: 'en-flex en-gap-2 en-mb-2' }, [
          h('input', {
            class: 'en-input',
            attrs: { placeholder: 'Name' },
            model: {
              get: (c) => c.newUser.name,
              set: (c, v) => { c.newUser.name = v; }
            }
          }),
          h('input', {
            class: 'en-input',
            attrs: { placeholder: 'Email', type: 'email' },
            model: {
              get: (c) => c.newUser.email,
              set: (c, v) => { c.newUser.email = v; }
            }
          }),
          h('select', {
            class: 'en-select',
            model: {
              get: (c) => c.newUser.role,
              set: (c, v) => { c.newUser.role = v; }
            }
          }, [
            h('option', { attrs: { value: 'user' } }, ['User']),
            h('option', { attrs: { value: 'admin' } }, ['Admin'])
          ]),
          h('button', {
            class: 'en-btn en-btn--primary',
            attrs: {
              disabled: (c) => !c.canAddUser
            },
            on: {
              click: (c) => c.addUser()
            }
          }, ['Add'])
        ])
      ]),
      
      // Filters
      h('div', { class: 'en-flex en-gap-2 en-mb-3' }, [
        // Search
        h('input', {
          class: 'en-input en-flex-1',
          attrs: { placeholder: 'Search by name or email...' },
          model: 'searchQuery'
        }),
        
        // Role filter
        h('select', {
          class: 'en-select',
          model: 'filterRole'
        }, [
          h('option', { attrs: { value: 'all' } }, ['All Roles']),
          h('option', { attrs: { value: 'admin' } }, ['Admin']),
          h('option', { attrs: { value: 'user' } }, ['User'])
        ]),
        
        // Sort
        h('select', {
          class: 'en-select',
          model: 'sortBy'
        }, [
          h('option', { attrs: { value: 'name' } }, ['Sort by Name']),
          h('option', { attrs: { value: 'email' } }, ['Sort by Email']),
          h('option', { attrs: { value: 'role' } }, ['Sort by Role'])
        ])
      ]),
      
      // User List
      h('div', {
        if: (c) => c.userCount === 0
      }, [
        h('div', { class: 'en-alert en-alert--info' }, [
          'No users found matching your criteria.'
        ])
      ]),
      
      // Keyed List Rendering
      h('div', {
        if: (c) => c.userCount > 0
      }, [
        h.for(
          (c) => c.filteredAndSortedUsers,
          (user) => user.id,
          (user) => h(UserCard, {
            props: {
              user: () => user,
              onDelete: (c) => c.deleteUser
            }
          })
        )
      ])
    ])
  ])
});
```

```js cleanup
onUnmount(() => {
  JQRH.unmount('#demo-list-rendering');
})
```

## Code Breakdown

### h.for() Syntax

```js
h.for(
  items: (ctx) => Array,
  key: (item, index) => any,
  build: (item, index) => JqrNode
)
```

**Parameters:**
- `items`: Function returning array of items
- `key`: Function extracting unique key from each item
- `build`: Function building node for each item

### Keyed Rendering

```js
h.for(
  (c) => c.filteredAndSortedUsers,
  (user) => user.id,  // Unique key per user
  (user) => h(UserCard, { props: { user: () => user } })
)
```

**Why keys matter:**
- DOM nodes are reused when key stays the same
- Only changed/removed/added items update
- Preserves component state across renders
- Efficient animations and transitions

### Nested Components in Lists

```js
h.for(
  (c) => c.users,
  (user) => user.id,
  (user) => h(UserCard, {
    props: {
      user: () => user,
      onDelete: (c) => c.deleteUser
    }
  })
)
```

**Important:** Wrap item value in function for reactivity: `user: () => user`

### Filtering and Sorting

```js
computed: {
  filteredAndSortedUsers() {
    let result = this.users;
    
    // Filter
    if (this.searchQuery) {
      result = result.filter(u => 
        u.name.toLowerCase().includes(this.searchQuery.toLowerCase())
      );
    }
    
    // Sort
    result = [...result].sort((a, b) => 
      a.name.localeCompare(b.name)
    );
    
    return result;
  }
}
```

Use computed property for derived lists—only recalculates when dependencies change.

### CRUD Operations

**Add:**
```js
addUser() {
  const newId = Math.max(...this.users.map(u => u.id), 0) + 1;
  this.users.push({
    id: newId,
    name: this.newUser.name,
    email: this.newUser.email
  });
}
```

**Delete:**
```js
deleteUser(id) {
  this.users = this.users.filter(u => u.id !== id);
}
```

**Update:**
```js
updateUser(id, updates) {
  const index = this.users.findIndex(u => u.id === id);
  if (index !== -1) {
    this.users[index] = { ...this.users[index], ...updates };
  }
}
```

## Advanced Patterns

### Pattern 1: Inline List Items

Simple lists without nested components:

```js
h.for(
  (c) => c.items,
  (item) => item.id,
  (item) => h('div', { class: 'item' }, [
    h('h3', {}, [item.name]),
    h('p', {}, [item.description])
  ])
)
```

### Pattern 2: Nested Lists

Lists within lists:

```js
h.for(
  (c) => c.categories,
  (cat) => cat.id,
  (cat) => h('div', { class: 'category' }, [
    h('h2', {}, [cat.name]),
    h.for(
      () => cat.items,
      (item) => item.id,
      (item) => h('div', { class: 'item' }, [item.name])
    )
  ])
)
```

### Pattern 3: Grouped Lists

Group items by property:

```js
computed: {
  groupedByRole() {
    const groups = {};
    this.users.forEach(user => {
      if (!groups[user.role]) groups[user.role] = [];
      groups[user.role].push(user);
    });
    return Object.entries(groups);
  }
},
render: (h, ctx) => h('div', {}, [
  h.for(
    (c) => c.groupedByRole,
    ([role]) => role,
    ([role, users]) => h('div', { class: 'group' }, [
      h('h3', {}, [role]),
      h.for(
        () => users,
        (user) => user.id,
        (user) => h('div', {}, [user.name])
      )
    ])
  )
])
```

### Pattern 4: Virtual Scrolling (Large Lists)

For very large lists, use windowing:

```js
computed: {
  visibleItems() {
    const start = this.scrollOffset;
    const end = start + this.windowSize;
    return this.allItems.slice(start, end);
  }
},
render: (h, ctx) => h('div', {
  style: { height: '400px', overflowY: 'auto' },
  on: {
    scroll: (c, ev) => {
      c.scrollOffset = Math.floor(ev.target.scrollTop / c.itemHeight);
    }
  }
}, [
  h.for(
    (c) => c.visibleItems,
    (item) => item.id,
    (item) => h('div', {
      style: { height: `${ctx.itemHeight}px` }
    }, [item.name])
  )
])
```

## Performance Tips

### 1. Use Stable Keys

```js
// ❌ BAD - index changes when sorted
h.for(items, (item, i) => i, ...)

// ✅ GOOD - stable unique identifier
h.for(items, (item) => item.id, ...)
```

### 2. Memoize Complex Filters

```js
computed: {
  filteredItems() {
    // Expensive operation runs only when dependencies change
    return this.items.filter(heavyComputation);
  }
}
```

### 3. Avoid Inline Object Creation

```js
// ❌ BAD - creates new object every render
h.for(items, item => item.id, item => h('div', {
  props: { data: { id: item.id, name: item.name } }
}))

// ✅ GOOD - pass item directly
h.for(items, item => item.id, item => h('div', {
  props: { item: () => item }
}))
```

### 4. Use show vs. if for Toggles

```js
// For frequently toggled visibility
h.for(items, item => item.id, item => h('div', {
  show: (c) => c.showDetails  // Hides but keeps in DOM
}))
```

## Real-World Example: Data Table

```js
mount('#table', {
  state: {
    data: [],
    sortColumn: 'name',
    sortDirection: 'asc',
    selectedRows: []
  },
  computed: {
    sortedData() {
      return [...this.data].sort((a, b) => {
        const aVal = a[this.sortColumn];
        const bVal = b[this.sortColumn];
        const compare = aVal < bVal ? -1 : aVal > bVal ? 1 : 0;
        return this.sortDirection === 'asc' ? compare : -compare;
      });
    }
  },
  methods: {
    toggleSort(column) {
      if (this.sortColumn === column) {
        this.sortDirection = this.sortDirection === 'asc' ? 'desc' : 'asc';
      } else {
        this.sortColumn = column;
        this.sortDirection = 'asc';
      }
    },
    toggleRow(id) {
      const index = this.selectedRows.indexOf(id);
      if (index > -1) {
        this.selectedRows.splice(index, 1);
      } else {
        this.selectedRows.push(id);
      }
    }
  },
  render: (h, ctx) => h('table', { class: 'en-table' }, [
    h('thead', {}, [
      h('tr', {}, [
        h('th', {}),
        h('th', {
          on: { click: (c) => c.toggleSort('name') }
        }, ['Name']),
        h('th', {
          on: { click: (c) => c.toggleSort('email') }
        }, ['Email']),
        h('th', {
          on: { click: (c) => c.toggleSort('role') }
        }, ['Role'])
      ])
    ]),
    h('tbody', {}, [
      h.for(
        (c) => c.sortedData,
        (row) => row.id,
        (row) => h('tr', {}, [
          h('td', {}, [
            h('input', {
              attrs: { type: 'checkbox' },
              model: {
                get: (c) => c.selectedRows.includes(row.id),
                set: (c) => c.toggleRow(row.id)
              }
            })
          ]),
          h('td', {}, [row.name]),
          h('td', {}, [row.email]),
          h('td', {}, [row.role])
        ])
      )
    ])
  ])
});
```

## Key Takeaways

1. **Always use unique, stable keys** - Essential for efficient updates
2. **Wrap item values in functions** - `user: () => user` for reactivity
3. **Use computed properties** - For filtered/sorted lists
4. **Nested components work seamlessly** - Pass props and event handlers
5. **Keys are compared by value** - Same key = reuse node

## See Also

- [h() Function](../guides/h-function.md) - Complete h() API
- [Basic Component](./basic-component.md) - Simple component patterns
- [Dynamic Component](./dynamic-component.md) - Component switching
- [Lists & Conditionals](../../jqr/guides/lists-and-conditionals.md) - Template-based lists
