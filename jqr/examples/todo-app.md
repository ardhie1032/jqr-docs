<!-- {
  "path": "jqr/examples/todo-app",
  "name": "docs-jqr-examples-todo-app",
  "meta": {
    "title": "Todo App",
    "order": 1
  }
} -->

# Example: Todo App

A complete todo application demonstrating lists, forms, computed properties, and local storage persistence.

## Features

- ✅ Add/remove todos
- ✅ Mark as complete
- ✅ Filter (all/active/completed)
- ✅ Clear completed
- ✅ LocalStorage persistence
- ✅ Todo count

## Live Demo

```html live:no-code
<div id="demo-todo-app" class="en-card">
  <pre $:text="todos"></pre>
  <div class="en-card__header">
    <h3>My Todo List</h3>
  </div>
  <div class="en-card__body">
    <!-- Add Todo Form -->
    <form $:submit.prevent="addTodo" class="en-mb-3">
      <div class="en-flex en-gap-2">
        <input 
          class="en-input" 
          $:model.trim="newTodo" 
          placeholder="What needs to be done?"
          $:attr:disabled="!canAdd">
        <button class="en-btn en-btn--primary" type="submit" $:attr:disabled="!canAdd || newTodo.length <= 3"">
          Add
        </button>
      </div>
    </form>

    <!-- Filter Tabs -->
    <div class="en-flex en-gap-2 en-mb-3">
      <button 
        class="en-btn en-btn--sm" 
        $:class="{ 'en-btn--primary': filter === 'all' }" 
        $:click="filter = 'all'">
        All (<span $:text="todos.length"></span>)
      </button>
      <button 
        class="en-btn en-btn--sm" 
        $:class="{ 'en-btn--primary': filter === 'active' }" 
        $:click="filter = 'active'">
        Active (<span $:text="activeCount"></span>)
      </button>
      <button 
        class="en-btn en-btn--sm" 
        $:class="{ 'en-btn--primary': filter === 'completed' }" 
        $:click="filter = 'completed'">
        Completed (<span $:text="completedCount"></span>)
      </button>
    </div>

    <!-- Todo List -->
    <div class="en-alert en-alert--info" $:if="filteredTodos.length === 0">
      <span $:if="filter === 'all'">No todos yet. Add one above!</span>
      <span $:if="filter === 'active'">No active todos</span>
      <span $:if="filter === 'completed'">No completed todos</span>
    </div>

    <ul class="en-list-group" $:if="filteredTodos.length > 0">
      <li $:for="todo in filteredTodos" $:key="todo.id" class="en-list-group__item en-flex en-gap-2 en-flex-between">
        <div>
          <label class="en-check">
            <input type="checkbox" $:model.confirm="todo.done"  $:on:confirm="handleToggleConfirm">
            <span 
              $:text="todo.text" 
              $:style="{ textDecoration: todo.done ? 'line-through' : 'none', opacity: todo.done ? 0.6 : 1 }">
            </span>
          </label>
        </div>
        <button 
          class="en-btn en-btn--sm en-btn--danger en-ml-auto"
          data-confirm-message="Are you sure?"
          $:click.confirm="removeTodo(todo.id)">
          Remove
        </button>
      </li>
    </ul>

    <!-- Actions -->
    <div class="en-flex en-gap-2 en-mt-3" $:if="todos.length > 0">
      <button 
        class="en-btn en-btn--sm" 
        $:click="toggleAll" 
        $:text="allDone ? 'Uncheck All' : 'Check All'">
      </button>
      <button 
        class="en-btn en-btn--sm en-btn--danger" 
        $:click="clearCompleted" 
        $:attr:disabled="completedCount === 0">
        Clear Completed
      </button>
    </div>
  </div>
</div>
```

```html
<div id="demo-todo-app">
  <h3>My Todo List</h3>
  <!-- Add Todo Form -->
  <form $:submit.prevent="addTodo()">
    <input $:model.trim="newTodo" placeholder="What needs to be done?" $:attr:disabled="!canAdd">
    <button type="submit" $:attr:disabled="!canAdd">Add</button>
  </form>

  <!-- Filter Tabs -->
  <button $:class="{ active: filter === 'all' }" $:click="filter = 'all'">
    All (<span $:text="todos.length"></span>)
  </button>
  <button $:class="{ active: filter === 'active' }" $:click="filter = 'active'">
    Active (<span $:text="activeCount"></span>)
  </button>
  <button $:class="{ active: filter === 'completed' }" $:click="filter = 'completed'">
    Completed (<span $:text="completedCount"></span>)
  </button>

  <!-- Empty State -->
  <div $:if="filteredTodos.length === 0">
    <span $:if="filter === 'all'">No todos yet. Add one above!</span>
    <span $:if="filter === 'active'">No active todos</span>
    <span $:if="filter === 'completed'">No completed todos</span>
  </div>

  <!-- Todo List -->
  <ul $:if="filteredTodos.length > 0">
    <li $:for="todo in filteredTodos" $:key="todo.id">
      <input type="checkbox" $:model="todo.done">
      <span $:text="todo.text" $:style="{ textDecoration: todo.done ? 'line-through' : 'none' }"></span>
      <button $:click="removeTodo(todo.id)">Remove</button>
    </li>
  </ul>

  <!-- Actions -->
  <div $:if="todos.length > 0">
    <button $:click="toggleAll()" $:text="allDone ? 'Uncheck All' : 'Check All'"></button>
    <button $:click="clearCompleted()" $:attr:disabled="completedCount === 0">
      Clear Completed
    </button>
  </div>
</div>
```

```js inject show
$('#demo-todo-app').component({
  state: {
    newTodo: '',
    filter: 'all',
    todos: JSON.parse(localStorage.getItem('jqr-todos') || '[]')
  },
  
  computed: {
    canAdd: (c) => c.todos.length <= 10,
    
    activeCount: (c) => c.todos.filter(t => !t.done).length,
    
    completedCount: (c) => c.todos.filter(t => t.done).length,
    
    allDone: (c) => c.todos.length > 0 && c.todos.every(t => t.done),
    
    filteredTodos: (c) => {
      if (c.filter === 'active') {
        return c.todos.filter(t => !t.done);
      }
      if (c.filter === 'completed') {
        return c.todos.filter(t => t.done);
      }
      return c.todos;
    }
  },
  
  watch: {
    todos: {
      handler(todos) {
        // localStorage.setItem('jqr-todos', JSON.stringify(todos));
      },
      deep: true
    }
  },
  
  methods: {
    handleDeleteConfirm($event) {
      const { oldValue, newValue, accept, reject } = $event.detail
      confirm('Are you sure?') ? accept() : reject()
    },
    handleToggleConfirm($event) {
      const { oldValue, newValue, accept, reject } = $event.detail
      confirm((newValue ? 'Not Yet' : 'Done') + ' checklist?') ? accept() : reject()
    },
    addTodo() {
      if (!this.canAdd) return;
      
      this.todos.push({
        id: Date.now(),
        text: this.newTodo,
        done: false
      });
      this.newTodo = '';
    },
    
    removeTodo(id) {
      const index = this.todos.findIndex(t => t.id === id);
      if (index !== -1) {
        this.todos.splice(index, 1);
      }
    },
    
    toggleAll() {
      const newState = !this.computed.allDone;
      this.todos.forEach(t => {
        t.done = newState;
      });
    },
    
    clearCompleted() {
      this.todos = this.todos.filter(t => !t.done);
    }
  }
});
```

```js cleanup
onUnmount(() => {
  $('#demo-todo-app').component('destroy');
})
```

## Code Breakdown

### State

```js
state: {
  newTodo: '',          // Input field value
  filter: 'all',        // Current filter: 'all' | 'active' | 'completed'
  todos: []             // Array of { id, text, done }
}
```

### Computed Properties

```js
computed: {
  // Can add new todo?
  canAdd: (s) => s.newTodo.length > 0,
  
  // Count active todos
  activeCount: (s) => s.todos.filter(t => !t.done).length,
  
  // Count completed todos
  completedCount: (s) => s.todos.filter(t => t.done).length,
  
  // Are all todos completed?
  allDone: (s) => s.todos.length > 0 && s.todos.every(t => t.done),
  
  // Filter todos based on current filter
  filteredTodos: (s) => {
    if (s.filter === 'active') return s.todos.filter(t => !t.done);
    if (s.filter === 'completed') return s.todos.filter(t => t.done);
    return s.todos;
  }
}
```

### LocalStorage Persistence

```js
watch: {
  todos: {
    handler(todos) {
      localStorage.setItem('jqr-todos', JSON.stringify(todos));
    },
    deep: true  // Watch nested changes (todo.done)
  }
}
```

Load on mount:

```js
state: {
  todos: JSON.parse(localStorage.getItem('jqr-todos') || '[]')
}
```

### Methods

```js
methods: {
  // Add new todo
  addTodo(state) {
    if (!this.computed.canAdd) return;
    
    state.todos.push({
      id: Date.now(),
      text: state.newTodo,
      done: false
    });
    state.newTodo = '';
  },
  
  // Remove todo by ID
  removeTodo(state, id) {
    const index = state.todos.findIndex(t => t.id === id);
    if (index !== -1) {
      state.todos.splice(index, 1);
    }
  },
  
  // Toggle all todos
  toggleAll(state) {
    const newState = !this.computed.allDone;
    state.todos.forEach(t => {
      t.done = newState;
    });
  },
  
  // Remove completed todos
  clearCompleted(state) {
    state.todos = state.todos.filter(t => !t.done);
  }
}
```

## Key Concepts

### 1. List Rendering with Keys

```html
<li $:for="todo in filteredTodos" $:key="todo.id">
  <!-- ... -->
</li>
```

**Why keys?** Efficient DOM updates when list changes.

### 2. Form Handling

```html
<form $:submit.prevent="addTodo()">
  <input $:model.trim="newTodo">
  <button type="submit" $:attr:disabled="!canAdd">Add</button>
</form>
```

- `.prevent` modifier stops form submission
- `.trim` modifier removes whitespace
- Disable submit when input is empty

### 3. Computed Filters

```js
computed: {
  filteredTodos: (s) => {
    if (s.filter === 'active') return s.todos.filter(t => !t.done);
    if (s.filter === 'completed') return s.todos.filter(t => t.done);
    return s.todos;
  }
}
```

Automatically updates when `filter` or `todos` changes.

### 4. Conditional Styling

```html
<span $:style="{ textDecoration: todo.done ? 'line-through' : 'none' }">
  {{ todo.text }}
</span>
```

Dynamically apply strikethrough to completed todos.

### 5. Deep Watch

```js
watch: {
  todos: {
    handler(todos) {
      localStorage.setItem('jqr-todos', JSON.stringify(todos));
    },
    deep: true  // Detect changes to todo.done
  }
}
```

Without `deep: true`, changing `todo.done` wouldn't trigger the watcher.

## Variations

### With Categories

```js
state: {
  todos: [],
  categories: ['Work', 'Personal', 'Shopping']
}
```

```html
<select $:model="newTodo.category">
  <option $:for="cat in categories" $:key="cat" $:attr:value="cat" $:text="cat"></option>
</select>
```

### With Due Dates

```js
state: {
  todos: [
    { id: 1, text: 'Task', done: false, dueDate: '2024-12-31' }
  ]
}
```

```html
<input type="date" $:model="newTodo.dueDate">
```

### With Priority

```js
computed: {
  sortedTodos: (s) => {
    return [...s.todos].sort((a, b) => {
      if (a.priority !== b.priority) {
        return b.priority - a.priority;  // High priority first
      }
      return a.id - b.id;
    });
  }
}
```

## Best Practices

### 1. Use Unique IDs

```js
// ✅ Good - unique ID
id: Date.now() + Math.random()

// ❌ Bad - index changes on reorder
id: todos.length
```

### 2. Validate Input

```js
computed: {
  canAdd: (s) => s.newTodo.trim().length >= 3
}
```

```html
<button $:attr:disabled="!canAdd">Add</button>
```

### 3. Provide Feedback

```html
<div $:if="filteredTodos.length === 0">
  No todos found
</div>
```

### 4. Persist Data

```js
watch: {
  todos: {
    handler(todos) {
      localStorage.setItem('todos', JSON.stringify(todos));
    },
    deep: true
  }
}
```

### 5. Use Computed for Filters

```js
// ✅ Good - cached
computed: {
  activeTodos: (s) => s.todos.filter(t => !t.done)
}

// ❌ Bad - runs every render
methods: {
  getActiveTodos(state) {
    return state.todos.filter(t => !t.done);
  }
}
```

## Next Steps

- [AJAX Form Example](./ajax-form.md) - Form with async validation
- [Modal Dialog Example](./modal-dialog.md) - Modal patterns
- [State Management Guide](../guides/state-management.md) - Managing complex state
- [Forms Guide](../guides/forms-and-models.md) - Form best practices
