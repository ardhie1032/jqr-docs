<!-- {
  "path": "jqr/examples/data-table",
  "name": "docs-jqr-examples-data-table",
  "meta": {
    "title": "Data Table",
    "order": 4
  }
} -->

# Data Table Example

A complete data table implementation with sorting, pagination, filtering, and selection. This example demonstrates working with larger datasets and common table patterns.

## Features

- **Sorting** - Click column headers to sort ascending/descending
- **Pagination** - Navigate through pages with configurable page size
- **Filtering** - Search across all columns
- **Selection** - Select individual rows or all rows
- **Status indicators** - Visual status badges
- **Responsive** - Works on mobile and desktop

## Complete Example

```html live:no-code
<div id="demo-data-table">
  <div class="en-card">
    <div class="en-card__header">
      <h3 class="en-card__title">User Management</h3>
    </div>
    
    <div class="en-card__body">
      <!-- Controls -->
      <div class="en-flex en-flex-wrap en-gap-3 en-mb-4">
        <div class="en-flex-1" style="min-width: 250px;">
          <input 
            class="en-input" 
            $:model.trim.debounce:300="searchQuery"
            type="search"
            placeholder="Search users...">
        </div>
        
        <div class="en-flex en-gap-2">
          <button 
            class="en-btn en-btn--secondary"
            $:click="clearFilters"
            $:disabled="!searchQuery && !sortColumn">
            Clear Filters
          </button>
          
          <select class="en-select" $:model.number="pageSize">
            <option value="5">5 per page</option>
            <option value="10">10 per page</option>
            <option value="20">20 per page</option>
          </select>
        </div>
      </div>
      
      <!-- Selection info -->
      <div 
        class="en-alert en-alert--info en-mb-3"
        $:if="selectedIds.length > 0">
        <span $:text="`${selectedIds.length} user(s) selected`"></span>
        <button 
          class="en-btn en-btn--sm en-btn--ghost en-ml-3"
          $:click="clearSelection">
          Clear
        </button>
      </div>
      
      <!-- Loading state -->
      <div class="en-text-center en-py-8" $:if="$loading.users">
        <div class="en-spinner en-spinner--lg"></div>
        <p class="en-text-muted en-mt-2">Loading users...</p>
      </div>
      
      <!-- Error state -->
      <div class="en-alert en-alert--danger" $:if="$error.users">
        <strong>Error:</strong> <span $:text="$error.users.message"></span>
      </div>
      
      <!-- Table -->
      <div class="en-table-container" $:if="!$loading.users && !$error.users">
        <table class="en-table en-table--hover">
          <thead>
            <tr>
              <th style="width: 40px;">
                <input 
                  type="checkbox"
                  $:checked="allSelected"
                  $:change="toggleAll">
              </th>
              <th $:click="() => toggleSort('name')" style="cursor: pointer;">
                Name
                <span $:if="sortColumn === 'name'">
                  <span $:if="sortDirection === 'asc'">↑</span>
                  <span $:if="sortDirection === 'desc'">↓</span>
                </span>
              </th>
              <th $:click="() => toggleSort('email')" style="cursor: pointer;">
                Email
                <span $:if="sortColumn === 'email'">
                  <span $:if="sortDirection === 'asc'">↑</span>
                  <span $:if="sortDirection === 'desc'">↓</span>
                </span>
              </th>
              <th $:click="() => toggleSort('role')" style="cursor: pointer;">
                Role
                <span $:if="sortColumn === 'role'">
                  <span $:if="sortDirection === 'asc'">↑</span>
                  <span $:if="sortDirection === 'desc'">↓</span>
                </span>
              </th>
              <th $:click="() => toggleSort('status')" style="cursor: pointer;">
                Status
                <span $:if="sortColumn === 'status'">
                  <span $:if="sortDirection === 'asc'">↑</span>
                  <span $:if="sortDirection === 'desc'">↓</span>
                </span>
              </th>
              <th style="width: 100px;">Actions</th>
            </tr>
          </thead>
          
          <tbody>
            <tr $:for="user in paginatedUsers" $:key="user.id">
              <td>
                <input 
                  type="checkbox"
                  $:checked="selectedIds.includes(user.id)"
                  $:change="() => toggleSelection(user.id)">
              </td>
              <td>
                <div class="en-flex en-items-center en-gap-2">
                  <img 
                    $:attr:src="user.avatar" 
                    $:attr:alt="user.name"
                    class="en-avatar en-avatar--sm"
                    style="width: 32px; height: 32px; border-radius: 50%;">
                  <span $:text="user.name"></span>
                </div>
              </td>
              <td>
                <code class="en-code" $:text="user.email"></code>
              </td>
              <td>
                <span 
                  class="en-badge"
                  $:class="{
                    'en-badge--primary': user.role === 'admin',
                    'en-badge--secondary': user.role === 'user',
                    'en-badge--info': user.role === 'moderator'
                  }"
                  $:text="user.role"></span>
              </td>
              <td>
                <span 
                  class="en-badge"
                  $:class="{
                    'en-badge--success': user.status === 'active',
                    'en-badge--warning': user.status === 'pending',
                    'en-badge--danger': user.status === 'inactive'
                  }"
                  $:text="user.status"></span>
              </td>
              <td>
                <div class="en-flex en-gap-1">
                  <button 
                    class="en-btn en-btn--sm en-btn--ghost"
                    $:click="() => viewUser(user)"
                    title="View">
                    👁
                  </button>
                  <button 
                    class="en-btn en-btn--sm en-btn--ghost"
                    $:click="() => editUser(user)"
                    title="Edit">
                    ✏️
                  </button>
                </div>
              </td>
            </tr>
            
            <!-- Empty state -->
            <tr $:if="paginatedUsers.length === 0">
              <td colspan="6" class="en-text-center en-py-8">
                <p class="en-text-muted">No users found</p>
              </td>
            </tr>
          </tbody>
        </table>
      </div>
      
      <!-- Pagination -->
      <div 
        class="en-flex en-justify-between en-items-center en-mt-4"
        $:if="!$loading.users && totalPages > 1">
        <div class="en-text-sm en-text-muted">
          <span $:text="`Showing ${startIndex + 1}-${Math.min(endIndex, filteredUsers.length)} of ${filteredUsers.length}`"></span>
        </div>
        
        <div class="en-flex en-gap-2">
          <button 
            class="en-btn en-btn--sm en-btn--secondary"
            $:click="previousPage"
            $:disabled="currentPage === 1">
            Previous
          </button>
          
          <div class="en-flex en-gap-1">
            <button
              $:for="page in visiblePages"
              $:key="page"
              class="en-btn en-btn--sm"
              $:class="{ 'en-btn--primary': currentPage === page, 'en-btn--secondary': currentPage !== page }"
              $:click="() => goToPage(page)"
              $:text="page"></button>
          </div>
          
          <button 
            class="en-btn en-btn--sm en-btn--secondary"
            $:click="nextPage"
            $:disabled="currentPage === totalPages">
            Next
          </button>
        </div>
      </div>
    </div>
  </div>
</div>
```

```html
<div id="demo-data-table">
  <!-- Controls -->
  <div>
    <input 
      $:model.trim.debounce:300="searchQuery"
      type="search"
      placeholder="Search users...">
    
    <button 
      $:click="clearFilters"
      $:disabled="!searchQuery && !sortColumn">
      Clear Filters
    </button>
    
    <select $:model.number="pageSize">
      <option value="5">5 per page</option>
      <option value="10">10 per page</option>
      <option value="20">20 per page</option>
    </select>
  </div>
  
  <!-- Selection info -->
  <div $:if="selectedIds.length > 0">
    <span $:text="`${selectedIds.length} user(s) selected`"></span>
    <button $:click="clearSelection">Clear</button>
  </div>
  
  <!-- Loading -->
  <div $:if="$loading.users">Loading users...</div>
  
  <!-- Error -->
  <div $:if="$error.users">
    Error: <span $:text="$error.users.message"></span>
  </div>
  
  <!-- Table -->
  <table $:if="!$loading.users && !$error.users">
    <thead>
      <tr>
        <th>
          <input 
            type="checkbox"
            $:checked="allSelected"
            $:change="toggleAll">
        </th>
        <th $:click="() => toggleSort('name')">
          Name
          <span $:if="sortColumn === 'name'">
            <span $:if="sortDirection === 'asc'">↑</span>
            <span $:if="sortDirection === 'desc'">↓</span>
          </span>
        </th>
        <th $:click="() => toggleSort('email')">Email</th>
        <th $:click="() => toggleSort('role')">Role</th>
        <th $:click="() => toggleSort('status')">Status</th>
        <th>Actions</th>
      </tr>
    </thead>
    
    <tbody>
      <tr $:for="user in paginatedUsers" $:key="user.id">
        <td>
          <input 
            type="checkbox"
            $:checked="selectedIds.includes(user.id)"
            $:change="() => toggleSelection(user.id)">
        </td>
        <td>
          <img $:attr:src="user.avatar" $:attr:alt="user.name">
          <span $:text="user.name"></span>
        </td>
        <td $:text="user.email"></td>
        <td $:text="user.role"></td>
        <td $:text="user.status"></td>
        <td>
          <button $:click="() => viewUser(user)">View</button>
          <button $:click="() => editUser(user)">Edit</button>
        </td>
      </tr>
      
      <!-- Empty state -->
      <tr $:if="paginatedUsers.length === 0">
        <td colspan="6">No users found</td>
      </tr>
    </tbody>
  </table>
  
  <!-- Pagination -->
  <div $:if="!$loading.users && totalPages > 1">
    <div>
      <span $:text="`Showing ${startIndex + 1}-${Math.min(endIndex, filteredUsers.length)} of ${filteredUsers.length}`"></span>
    </div>
    
    <div>
      <button 
        $:click="previousPage"
        $:disabled="currentPage === 1">
        Previous
      </button>
      
      <button
        $:for="page in visiblePages"
        $:key="page"
        $:class="{ active: currentPage === page }"
        $:click="() => goToPage(page)"
        $:text="page"></button>
      
      <button 
        $:click="nextPage"
        $:disabled="currentPage === totalPages">
        Next
      </button>
    </div>
  </div>
</div>
```

```js inject show
$('#demo-data-table').component({
  state: {
    searchQuery: '',
    sortColumn: null,
    sortDirection: 'asc',
    currentPage: 1,
    pageSize: 10,
    selectedIds: []
  },
  
  asyncState: {
    async users() {
      // Simulate network delay
      await new Promise(resolve => setTimeout(resolve, 800));
      
      // Fetch users from API
      const response = await fetch('/assets/api/jqr-docs/users.json');
      return response.json();
    }
  },
  
  computed: {
    // Filter users based on search query
    filteredUsers(state) {
      const query = state.searchQuery.toLowerCase();
      if (!query) return state.$data.users || [];
      
      return (state.$data.users || []).filter(user => {
        return (
          user.name.toLowerCase().includes(query) ||
          user.email.toLowerCase().includes(query) ||
          user.role.toLowerCase().includes(query) ||
          user.status.toLowerCase().includes(query)
        );
      });
    },
    
    // Sort filtered users
    sortedUsers(state) {
      const users = [...this.filteredUsers(state)];
      
      if (!state.sortColumn) return users;
      
      return users.sort((a, b) => {
        const aVal = a[state.sortColumn];
        const bVal = b[state.sortColumn];
        
        if (aVal < bVal) return state.sortDirection === 'asc' ? -1 : 1;
        if (aVal > bVal) return state.sortDirection === 'asc' ? 1 : -1;
        return 0;
      });
    },
    
    // Calculate pagination
    totalPages(state) {
      const total = this.sortedUsers(state).length;
      return Math.ceil(total / state.pageSize);
    },
    
    startIndex(state) {
      return (state.currentPage - 1) * state.pageSize;
    },
    
    endIndex(state) {
      return this.startIndex(state) + state.pageSize;
    },
    
    paginatedUsers(state) {
      const sorted = this.sortedUsers(state);
      return sorted.slice(this.startIndex(state), this.endIndex(state));
    },
    
    // Visible page numbers (show 5 pages max)
    visiblePages(state) {
      const total = this.totalPages(state);
      const current = state.currentPage;
      const delta = 2;
      
      let start = Math.max(1, current - delta);
      let end = Math.min(total, current + delta);
      
      // Adjust if at boundaries
      if (current <= delta) {
        end = Math.min(total, 1 + delta * 2);
      }
      if (current >= total - delta) {
        start = Math.max(1, total - delta * 2);
      }
      
      const pages = [];
      for (let i = start; i <= end; i++) {
        pages.push(i);
      }
      return pages;
    },
    
    // Selection state
    allSelected(state) {
      if (!state.$data.users || state.$data.users.length === 0) {
        return false;
      }
      const pageIds = this.paginatedUsers(state).map(u => u.id);
      return pageIds.every(id => state.selectedIds.includes(id));
    }
  },
  
  methods: {
    toggleSort(state, column) {
      if (state.sortColumn === column) {
        // Toggle direction
        state.sortDirection = state.sortDirection === 'asc' ? 'desc' : 'asc';
      } else {
        // New column
        state.sortColumn = column;
        state.sortDirection = 'asc';
      }
      
      // Reset to first page
      state.currentPage = 1;
    },
    
    clearFilters(state) {
      state.searchQuery = '';
      state.sortColumn = null;
      state.sortDirection = 'asc';
      state.currentPage = 1;
    },
    
    goToPage(state, page) {
      state.currentPage = page;
    },
    
    previousPage(state) {
      if (state.currentPage > 1) {
        state.currentPage--;
      }
    },
    
    nextPage(state) {
      const total = this.totalPages(state);
      if (state.currentPage < total) {
        state.currentPage++;
      }
    },
    
    toggleSelection(state, userId) {
      const index = state.selectedIds.indexOf(userId);
      if (index > -1) {
        state.selectedIds.splice(index, 1);
      } else {
        state.selectedIds.push(userId);
      }
    },
    
    toggleAll(state) {
      const pageIds = this.paginatedUsers(state).map(u => u.id);
      const allSelected = this.allSelected(state);
      
      if (allSelected) {
        // Deselect all on page
        state.selectedIds = state.selectedIds.filter(id => !pageIds.includes(id));
      } else {
        // Select all on page
        pageIds.forEach(id => {
          if (!state.selectedIds.includes(id)) {
            state.selectedIds.push(id);
          }
        });
      }
    },
    
    clearSelection(state) {
      state.selectedIds = [];
    },
    
    viewUser(state, user) {
      alert(`Viewing ${user.name}\n\nEmail: ${user.email}\nRole: ${user.role}\nStatus: ${user.status}`);
    },
    
    editUser(state, user) {
      alert(`Edit mode for ${user.name}`);
    }
  },
  
  watch: {
    // Reset to page 1 when search changes
    searchQuery() {
      this.currentPage = 1;
    },
    
    // Reset to page 1 when page size changes
    pageSize() {
      this.currentPage = 1;
    }
  }
});
```

```js cleanup
onUnmount(() => {
  $('#demo-data-table').component('destroy');
})
```

## Key Concepts

### 1. Async State Loading

```js
asyncState: {
  async users() {
    const response = await fetch('/api/users');
    return response.json();
  }
}

// Access in template
// $:if="$loading.users" - Shows loading state
// $:if="$error.users" - Shows error state
// state.$data.users - The loaded data
```

### 2. Search Filtering

```js
computed: {
  filteredUsers(state) {
    const query = state.searchQuery.toLowerCase();
    return (state.$data.users || []).filter(user => {
      return user.name.toLowerCase().includes(query) ||
             user.email.toLowerCase().includes(query);
    });
  }
}
```

**Debounced input:**
```html
<input $:model.trim.debounce:300="searchQuery">
```

### 3. Sorting

```js
methods: {
  toggleSort(state, column) {
    if (state.sortColumn === column) {
      // Toggle direction
      state.sortDirection = state.sortDirection === 'asc' ? 'desc' : 'asc';
    } else {
      state.sortColumn = column;
      state.sortDirection = 'asc';
    }
  }
}

computed: {
  sortedUsers(state) {
    const users = [...this.filteredUsers(state)];
    if (!state.sortColumn) return users;
    
    return users.sort((a, b) => {
      const aVal = a[state.sortColumn];
      const bVal = b[state.sortColumn];
      return state.sortDirection === 'asc' 
        ? aVal > bVal ? 1 : -1
        : aVal < bVal ? 1 : -1;
    });
  }
}
```

### 4. Pagination

```js
computed: {
  totalPages(state) {
    return Math.ceil(this.sortedUsers(state).length / state.pageSize);
  },
  
  paginatedUsers(state) {
    const start = (state.currentPage - 1) * state.pageSize;
    const end = start + state.pageSize;
    return this.sortedUsers(state).slice(start, end);
  }
}

methods: {
  nextPage(state) {
    if (state.currentPage < this.totalPages(state)) {
      state.currentPage++;
    }
  }
}
```

### 5. Selection

```js
state: {
  selectedIds: []
},

methods: {
  toggleSelection(state, userId) {
    const index = state.selectedIds.indexOf(userId);
    if (index > -1) {
      state.selectedIds.splice(index, 1);
    } else {
      state.selectedIds.push(userId);
    }
  },
  
  toggleAll(state) {
    const pageIds = this.paginatedUsers(state).map(u => u.id);
    const allSelected = pageIds.every(id => 
      state.selectedIds.includes(id)
    );
    
    if (allSelected) {
      // Deselect page
      state.selectedIds = state.selectedIds.filter(
        id => !pageIds.includes(id)
      );
    } else {
      // Select page
      pageIds.forEach(id => {
        if (!state.selectedIds.includes(id)) {
          state.selectedIds.push(id);
        }
      });
    }
  }
}
```

## Performance Tips

### 1. Computed Chains

Chain computed properties for better organization:

```js
computed: {
  // Step 1: Filter
  filteredUsers(state) {
    return state.$data.users.filter(/* ... */);
  },
  
  // Step 2: Sort
  sortedUsers(state) {
    return this.filteredUsers(state).sort(/* ... */);
  },
  
  // Step 3: Paginate
  paginatedUsers(state) {
    return this.sortedUsers(state).slice(/* ... */);
  }
}
```

Each computed only recalculates when its dependencies change.

### 2. Debounce Search

```html
<!-- Updates after 300ms of no typing -->
<input $:model.debounce:300="searchQuery">
```

Prevents filtering on every keystroke.

### 3. Key for Lists

```html
<tr $:for="user in users" $:key="user.id">
```

Helps JQR efficiently update the DOM when the list changes.

### 4. Lazy Load Data

```js
asyncState: {
  async users() {
    // Only load when component mounts
    const response = await fetch('/api/users');
    return response.json();
  }
}
```

## Variations

### Server-Side Pagination

```js
asyncState: {
  async users(state) {
    const params = new URLSearchParams({
      page: state.currentPage,
      limit: state.pageSize,
      search: state.searchQuery,
      sortBy: state.sortColumn,
      sortDir: state.sortDirection
    });
    
    const response = await fetch(`/api/users?${params}`);
    const data = await response.json();
    
    // Server returns: { users: [...], total: 100 }
    state.totalRecords = data.total;
    return data.users;
  }
},

computed: {
  totalPages(state) {
    return Math.ceil(state.totalRecords / state.pageSize);
  },
  
  paginatedUsers(state) {
    // Data already paginated by server
    return state.$data.users || [];
  }
}
```

### Bulk Actions

```js
methods: {
  async deleteSelected(state) {
    if (state.selectedIds.length === 0) return;
    
    if (!confirm(`Delete ${state.selectedIds.length} users?`)) {
      return;
    }
    
    try {
      await fetch('/api/users/bulk-delete', {
        method: 'POST',
        headers: { 'Content-Type': 'application/json' },
        body: JSON.stringify({ ids: state.selectedIds })
      });
      
      // Refresh data
      await this.$refetch('users');
      state.selectedIds = [];
    } catch (error) {
      alert('Delete failed: ' + error.message);
    }
  }
}
```

### Editable Cells

```html
<td>
  <input 
    $:if="editingId === user.id"
    $:model="user.name"
    $:blur="() => saveEdit(user)"
    type="text">
  
  <span 
    $:if="editingId !== user.id"
    $:dblclick="() => startEdit(user.id)"
    $:text="user.name"></span>
</td>
```

```js
state: {
  editingId: null
},

methods: {
  startEdit(state, userId) {
    state.editingId = userId;
  },
  
  async saveEdit(state, user) {
    state.editingId = null;
    
    await fetch(`/api/users/${user.id}`, {
      method: 'PATCH',
      headers: { 'Content-Type': 'application/json' },
      body: JSON.stringify(user)
    });
  }
}
```

## Related Examples

- [Todo App](./todo-app.md) - List management patterns
- [AJAX Form](./ajax-form.md) - Form submission and validation
- [Async State Guide](../guides/async-state.md) - Loading data patterns

