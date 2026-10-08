<!-- {
  "path": "jqr/guides/props",
  "name": "docs-jqr-guides-props",
  "meta": {
    "title": "Props",
    "order": 12
  }
} -->

# Props

Props allow parent components to pass data down to child components. They enable reusable, configurable components that adapt to different contexts.

## Why Props?

Props solve the problem of component configuration:

**Without props:**
```html
<!-- Rigid, hardcoded component -->
<div id="user-card">
  <h3>John Doe</h3>
  <p>john@example.com</p>
</div>
```

**With props:**
```html
<!-- Flexible, reusable component -->
<div $:component="user-card" $:prop:name="userName" $:prop:email="userEmail">
  <!-- Component uses $props.name and $props.email -->
</div>
```

**Benefits:**
- Reusable components with different data
- Clear component API (what data it needs)
- Parent-child communication
- Type-safe data flow

## Basic Props

Define props in component options:

```html live:no-code
<template id="demo-greeting-component">
  <div class="en-card">
    <div class="en-card__body">
      <h4 class="en-text--primary" $:text="'Hello, ' + $props.name + '!'"></h4>
      <p class="en-text--muted" $:text="$props.message"></p>
    </div>
  </div>
</template>

<div id="demo-basic-props">
  <div $:component="demo-greeting-component" $:prop:name="userName" $:prop:message="greeting"></div>
</div>
```

```html
<template id="greeting-component">
  <div>
    <h3 $:text="'Hello, ' + $props.name + '!'"></h3>
    <p $:text="$props.message"></p>
  </div>
</template>

<div id="demo-basic-props">
  <div $:component="greeting-component" $:prop:name="userName" $:prop:message="greeting"></div>
</div>
```

```js inject show
$('#demo-basic-props').component({
  components: {
    'demo-greeting-component': {
      props: { name: 'Guest', message: 'Welcome!' }
    }
  },
  state: {
    userName: 'Alice',
    greeting: 'Nice to meet you!'
  }
});
```

```js cleanup
onUnmount(() => {
  $('#demo-basic-props').component('destroy');
})
```

## Props Declaration

Declare props with default values:

```js
{
  components: {
    'user-card': {
      props: {
        name: 'Anonymous',      // String default
        age: 0,                 // Number default
        active: false,          // Boolean default
        roles: []               // Array default
      }
    }
  }
}
```

**Props become available as `this.props` or `$props` in templates.**

## Passing Props

Use `$:prop:name` directive to pass data from parent:

```html
<!-- Static value -->
<div $:component="user-card" $:prop:name="'John'"></div>

<!-- Dynamic value from state -->
<div $:component="user-card" $:prop:name="currentUser.name"></div>

<!-- Expression -->
<div $:component="user-card" $:prop:age="user.age + 10"></div>

<!-- Multiple props -->
<div 
  $:component="user-card" 
  $:prop:name="user.name"
  $:prop:age="user.age"
  $:prop:active="user.isActive">
</div>
```

**⚠️ Important: Prop Naming Convention**

HTML attributes are **case-insensitive** and automatically converted to lowercase. This affects prop names:

```html
<!-- ❌ DON'T: camelCase prop names -->
<div $:component="user-card" $:prop:firstName="user.first"></div>
<!-- Becomes: $:prop:firstname (lowercase) -->

<!-- ✅ DO: Use kebab-case or lowercase -->
<div $:component="user-card" $:prop:first-name="user.first"></div>
<!-- Or: -->
<div $:component="user-card" $:prop:firstname="user.first"></div>
```

**In component definition:**
```js
// ❌ DON'T: camelCase won't match
props: { firstName: '' }  // Won't receive $:prop:firstName

// ✅ DO: Use kebab-case or lowercase
props: { 'first-name': '' }  // Matches $:prop:first-name
// Or:
props: { firstname: '' }     // Matches $:prop:firstname
```

**Best practice:** Use **kebab-case** for multi-word prop names to maintain readability.

## Accessing Props

Props are available in multiple contexts:

### In Templates

```html
<template id="user-template">
  <div>
    <h3 $:text="$props.name"></h3>
    <p $:text="'Age: ' + $props.age"></p>
    <span $:if="$props.active">Active</span>
  </div>
</template>
```

### In State Initializer

```js
{
  components: {
    'counter': {
      props: { initial: 0 },
      state: (props) => ({
        count: props.initial  // Initialize state from props
      })
    }
  }
}
```

### In Methods

```js
{
  components: {
    'user-card': {
      props: { name: '' },
      methods: {
        greet({ props }) {
          alert(`Hello, ${props.name}!`);
        }
      }
    }
  }
}
```

### In Computed

```js
{
  components: {
    'user-info': {
      props: { firstName: '', lastName: '' },
      state: {},
      computed: {
        fullName: (state, props) => `${props.firstName} ${props.lastName}`
      }
    }
  }
}
```

### In AsyncState

```js
{
  components: {
    'user-posts': {
      props: { userId: null },
      asyncState: {
        async posts(state, props) {
          const res = await fetch(`/api/users/${props.userId}/posts`);
          return res.json();
        }
      }
    }
  }
}
```

## Read-Only Props

**Important:** Props are **read-only** in child components.

```js
// ❌ This will NOT work (console warning, no-op)
methods: {
  updateName({ props }) {
    props.name = 'New Name';  // Read-only, ignored!
  }
}

// ✅ Instead, emit event to parent
methods: {
  updateName({ props }) {
    this.emit('update:name', 'New Name');
  }
}
```

**Why read-only?** Props follow **one-way data flow**: parent controls the data, child receives and displays it.

## Dynamic Props Example

Props update automatically when parent state changes:

```html live:no-code
<template id="demo-user-card-component">
  <div class="en-card">
    <div class="en-card__body">
      <div class="en-flex en-gap-3 en-items-center">
        <div class="en-avatar en-avatar--lg" $:style="{ backgroundColor: $props.color }">
          <span $:text="$props.name.charAt(0).toUpperCase()"></span>
        </div>
        <div>
          <h4 style="margin: 0;" $:text="$props.name"></h4>
          <p class="en-text--muted" style="margin: 0;" $:text="$props.role"></p>
        </div>
      </div>
    </div>
  </div>
</template>

<div id="demo-dynamic-props">
  <div class="en-card">
    <div class="en-card__body">
      <h3>Dynamic Props</h3>
      <div class="en-flex en-gap-2" style="margin-bottom: 1rem;">
        <button class="en-btn en-btn--sm" $:click="selectedUser = users[0]">User 1</button>
        <button class="en-btn en-btn--sm" $:click="selectedUser = users[1]">User 2</button>
        <button class="en-btn en-btn--sm" $:click="selectedUser = users[2]">User 3</button>
      </div>
      
      <div $:component="demo-user-card-component" 
           $:prop:name="selectedUser.name"
           $:prop:role="selectedUser.role"
           $:prop:color="selectedUser.color">
      </div>
    </div>
  </div>
</div>
```

```html
<template id="user-card-component">
  <div>
    <div style="display: flex; gap: 1rem; align-items: center;">
      <div $:text="$props.name.charAt(0)"></div>
      <div>
        <h4 $:text="$props.name"></h4>
        <p $:text="$props.role"></p>
      </div>
    </div>
  </div>
</template>

<div id="demo-dynamic-props">
  <button $:click="selectedUser = users[0]">User 1</button>
  <button $:click="selectedUser = users[1]">User 2</button>
  <button $:click="selectedUser = users[2]">User 3</button>
  
  <div $:component="user-card-component" 
       $:prop:name="selectedUser.name"
       $:prop:role="selectedUser.role"
       $:prop:color="selectedUser.color">
  </div>
</div>
```

```js inject show
$('#demo-dynamic-props').component({
  components: {
    'demo-user-card-component': {
      props: { name: '', role: '', color: '#ccc' }
    }
  },
  state: {
    users: [
      { name: 'Alice Johnson', role: 'Frontend Developer', color: '#3b82f6' },
      { name: 'Bob Smith', role: 'Backend Developer', color: '#10b981' },
      { name: 'Carol Williams', role: 'Designer', color: '#f59e0b' }
    ],
    selectedUser: { name: 'Alice Johnson', role: 'Frontend Developer', color: '#3b82f6' }
  }
});
```

```js cleanup
onUnmount(() => {
  $('#demo-dynamic-props').component('destroy');
})
```

## Props with Slots

Combine props with slots for maximum flexibility:

```html live:no-code
<template id="demo-card-component">
  <div class="en-card">
    <div class="en-card__header">
      <h4 style="margin: 0;" $:text="$props.title"></h4>
    </div>
    <div class="en-card__body">
      <slot $:default>No content</slot>
    </div>
    <div class="en-card__footer" $:if="$props.showFooter">
      <slot $:name="footer">
        <button class="en-btn en-btn--sm">OK</button>
      </slot>
    </div>
  </div>
</template>

<div id="demo-props-slots">
  <div $:component="demo-card-component" $:prop:title="cardTitle" $:prop:showFooter="true">
    <p>This is the main content of the card.</p>
    <template $:slot="footer">
      <button class="en-btn en-btn--sm en-btn--primary" $:click="handleSave">Save</button>
      <button class="en-btn en-btn--sm en-btn--outline" $:click="handleCancel">Cancel</button>
    </template>
  </div>
</div>
```

```html
<template id="demo-card-component">
  <div>
    <header>
      <h4 $:text="$props.title"></h4>
    </header>
    <div>
      <slot $:default>No content</slot>
    </div>
    <footer $:if="$props.showFooter">
      <slot $:name="footer">
        <button>OK</button>
      </slot>
    </footer>
  </div>
</template>

<div id="demo-props-slots">
  <div $:component="card-component" $:prop:title="cardTitle" $:prop:showFooter="true">
    <template $:slot="header">Custom Header</template>
    <p>Card content here</p>
    <template $:slot="footer">
      <button $:click="handleSave">Save</button>
      <button $:click="handleCancel">Cancel</button>
    </template>
  </div>
</div>
```

```js inject show
$('#demo-props-slots').component({
  components: {
    'demo-card-component': {
      props: { title: 'Card', showFooter: false }
    }
  },
  state: {
    cardTitle: 'User Settings'
  },
  methods: {
    handleSave() { alert('Saved!'); },
    handleCancel() { alert('Cancelled!'); }
  }
});
```

```js cleanup
onUnmount(() => {
  $('#demo-props-slots').component('destroy');
})
```

## Two-Way Binding with v-model

Use `$:model:propName` for two-way prop binding:

```html live:no-code
<template id="demo-counter-component">
  <div class="en-card">
    <div class="en-card__body">
      <div class="en-flex en-gap-2 en-items-center">
        <button class="en-btn en-btn--sm" $:click="decrement">−</button>
        <span class="en-form-display en-form-display--ghost" $:text="$props.value"></span>
        <button class="en-btn en-btn--sm" $:click="increment">+</button>
      </div>
    </div>
  </div>
</template>

<div id="demo-vmodel-props">
  <div class="en-card">
    <div class="en-card__body">
      <h3>Two-Way Binding</h3>
      <p>Parent count: <code $:text="count"></code></p>
      <div $:component="demo-counter-component" $:model:value="count"></div>
    </div>
  </div>
</div>
```

```html
<template id="demo-counter-component">
  <div>
    <button $:click="decrement">−</button>
    <span $:text="$props.value"></span>
    <button $:click="increment">+</button>
  </div>
</template>

<div id="demo-vmodel-props">
  <p>Parent count: <code $:text="count"></code></p>
  <div $:component="counter-component" $:model:value="count"></div>
</div>
```

```js inject show
$('#demo-vmodel-props').component({
  components: {
    'demo-counter-component': {
      props: { value: 0 },
      emits: ['update:value'],
      methods: {
        increment() {
          this.emit('update:value', this.props.value + 1);
        },
        decrement() {
          this.emit('update:value', this.props.value - 1);
        }
      }
    }
  },
  state: { count: 5 }
});
```

```js cleanup
onUnmount(() => {
  $('#demo-vmodel-props').component('destroy');
})
```

**How it works:**
1. Parent: `$:model:value="count"` → binds `count` to child's `value` prop
2. Child emits: `this.emit('update:value', newValue)`
3. Parent automatically updates: `count = newValue`

## Props with AsyncState

Load data based on props:

```html live:no-code
<template id="demo-user-posts-component">
  <div class="en-card">
    <div class="en-card__body">
      <div $:if="loading" class="en-spinner"></div>
      <div $:if="!loading && posts && posts.length === 0">
        <p class="en-text--muted">No posts found</p>
      </div>
      <div $:if="!loading && posts && posts.length > 0">
        <div $:for="post in posts" $:key="post.id" class="en-card" style="margin-bottom: 0.5rem;">
          <div class="en-card__body">
            <h5 style="margin: 0 0 0.25rem 0;" $:text="post.title"></h5>
            <p class="en-text--muted" style="margin: 0; font-size: 0.875rem;" $:text="post.excerpt"></p>
            <div style="margin-top: 0.5rem; font-size: 0.75rem; color: #9ca3af;">
              <span $:text="'❤️ ' + post.likes"></span>
              <span style="margin-left: 1rem;" $:text="'💬 ' + post.comments"></span>
            </div>
          </div>
        </div>
      </div>
    </div>
  </div>
</template>

<div id="demo-async-props">
  <div class="en-card">
    <div class="en-card__body">
      <h3>User Posts</h3>
      <div class="en-flex en-gap-2" style="margin-bottom: 1rem;">
        <button class="en-btn en-btn--sm" $:click="selectedUserId = 1">User 1</button>
        <button class="en-btn en-btn--sm" $:click="selectedUserId = 2">User 2</button>
        <button class="en-btn en-btn--sm" $:click="selectedUserId = 3">User 3</button>
      </div>
      
      <div $:component="demo-user-posts-component" $:prop:user-id="selectedUserId"></div>
    </div>
  </div>
</div>
```

```html
<template id="demo-user-posts-component">
  <div>
    <pre $:text="posts"></pre>
    <div $:if="loading">Loading...</div>
    <div $:if="!loading && posts && posts.length === 0">
      <p>No posts found</p>
    </div>
    <div $:if="!loading && posts && posts.length > 0">
      <div $:for="post in posts" $:key="post.id">
        <h5 $:text="post.title"></h5>
        <p $:text="post.excerpt"></p>
      </div>
    </div>
  </div>
</template>

<div id="demo-async-props">
  <button $:click="selectedUserId = 1">User 1</button>
  <button $:click="selectedUserId = 2">User 2</button>
  <button $:click="selectedUserId = 3">User 3</button>
  
  <div $:component="demo-user-posts-component" $:prop:user-id="selectedUserId"></div>
</div>
```

```js inject show
$('#demo-async-props').component({
  components: {
    'demo-user-posts-component': {
      props: { userId: null },
      state: { posts: [], loading: false },
      asyncState: {
        posts: async (state, props) => {
          const userId = props.userId; // <-- dibaca duluan, sinkron, sebelum await manapun
          state.loading = true;
          const res = await fetch('/assets/api/jqr-docs/posts.json').then(r => r.json());
          res.filter(post => post.userId === userId)
          state.loading = false;
          return res.filter(post => post.userId === userId);
        }
      },
    }
  },
  state: { selectedUserId: 1 },
});
```

```js cleanup
onUnmount(() => {
  $('#demo-async-props').component('destroy');
})
```

**Automatic refetch:** When `props.userId` changes, `asyncState.posts` re-runs automatically.

## Prop Reactivity

Props are reactive - when parent updates them, child re-renders:

```js
// Parent component
state: { userName: 'Alice' }

// Parent updates state
this.state.userName = 'Bob';
// → Child component's props.name updates automatically
// → Child's template re-renders with new value
```

**Watch props changes in child:**

```js
{
  components: {
    'user-card': {
      props: { name: '' },
      created() {
        this.watch(() => this.props.name, (newName, oldName) => {
          console.log(`Name changed from ${oldName} to ${newName}`);
        });
      }
    }
  }
}
```

## Props vs State

**When to use Props:**
- Data comes from parent component
- Data is controlled by parent
- Component is reusable with different data

**When to use State:**
- Data is internal to component
- Component controls the data
- Data doesn't need to come from parent

**Example:**

```js
{
  components: {
    'toggle-button': {
      // Props: configuration from parent
      props: { 
        label: 'Toggle',
        initialValue: false
      },
      
      // State: internal component state
      state: (props) => ({
        isOn: props.initialValue  // Initialize from prop
      }),
      
      methods: {
        toggle(state) {
          state.isOn = !state.isOn;  // Modify internal state
          this.emit('change', state.isOn);
        }
      }
    }
  }
}
```

## Constants from Props

Use `constants` option to derive read-only values from initial props:

```js
{
  components: {
    'data-table': {
      props: { columns: [] },
      
      // Computed once from initial props
      constants: (props) => ({
        columnCount: props.columns.length,
        hasActions: props.columns.some(c => c.type === 'action')
      }),
      
      // Available as this._constants in methods
      // Not reactive - computed only once at creation
    }
  }
}
```

**Difference from computed:**
- `constants`: Computed **once** at creation from initial props
- `computed`: Re-computed automatically when dependencies change

## Programmatic Props Update

Use `setProps()` to update child props programmatically:

```js
const childComponent = $('#child-element').component('instance');

// Update single prop
childComponent.setProps({ name: 'New Name' });

// Update multiple props
childComponent.setProps({ 
  name: 'Alice',
  age: 30,
  active: true
});
```

**When to use:**
- Imperatively controlling child components
- Dynamic prop updates outside template
- Integration with non-JQR code

## Common Patterns

### Config Object Prop

Pass entire config as single prop:

```html
<div $:component="data-table" $:prop:config="tableConfig"></div>
```

```js
state: {
  tableConfig: {
    columns: [...],
    pageSize: 10,
    sortable: true,
    filterable: true
  }
}
```

### Callback Props

Pass functions as props:

```html
<div $:component="custom-button" $:prop:onClick="handleClick"></div>
```

```js
{
  components: {
    'custom-button': {
      props: { onClick: null },
      methods: {
        handleButtonClick() {
          if (this.props.onClick) {
            this.props.onClick();
          }
        }
      }
    }
  },
  methods: {
    handleClick() {
      console.log('Button clicked!');
    }
  }
}
```

### Conditional Props

Different props based on condition:

```html
<div 
  $:component="user-card" 
  $:prop:name="user.name"
  $:prop:role="user.isAdmin ? 'Administrator' : 'User'">
</div>
```

### Props Spreading

Pass multiple props from object:

```js
state: {
  userProps: {
    name: 'Alice',
    age: 30,
    role: 'Developer'
  }
}
```

```html
<div 
  $:component="user-card" 
  $:prop:name="userProps.name"
  $:prop:age="userProps.age"
  $:prop:role="userProps.role">
</div>
```

## Validation & Types

JQR doesn't have built-in prop validation, but you can add your own:

```js
{
  components: {
    'user-card': {
      props: { age: 0 },
      created() {
        // Custom validation
        if (this.props.age < 0) {
          console.warn('Invalid age prop: must be positive');
        }
      }
    }
  }
}
```

**For TypeScript users:**

```typescript
interface UserCardProps {
  name: string;
  age: number;
  active: boolean;
}

const userCard: ComponentOptions<{}, UserCardProps> = {
  props: { name: '', age: 0, active: false },
  // ...
};
```

## Best Practices

### 1. Provide Defaults

```js
// ✅ Good - has defaults
props: { name: 'Guest', age: 0 }

// ❌ Risky - undefined if not passed
props: { name: undefined, age: undefined }
```

### 2. Keep Props Simple

```js
// ✅ Good - simple primitive values
props: { name: '', count: 0, active: false }

// ⚠️ Acceptable - simple objects
props: { user: null, config: {} }

// ❌ Avoid - complex nested objects
props: { 
  deeplyNested: { 
    data: { 
      with: { many: { levels: {} } }
    }
  }
}
```

### 3. Don't Mutate Props

```js
// ❌ Bad - mutating props
methods: {
  updateUser({ props }) {
    props.user.name = 'New Name';  // Don't do this!
  }
}

// ✅ Good - emit event to parent
methods: {
  updateUser({ props }) {
    this.emit('update:user', {
      ...props.user,
      name: 'New Name'
    });
  }
}
```

### 4. Use Descriptive Names

```js
// ✅ Good - clear purpose
props: { 'user-name': '', 'user-email': '', 'is-active': false }

// ❌ Confusing - unclear meaning
props: { name: '', text: '', flag: false }
```

**Remember:** Use kebab-case for multi-word props due to HTML's case-insensitivity.

### 5. Document Prop Purpose

```js
{
  components: {
    'data-table': {
      /**
       * Props:
       * - items: Array of data objects to display
       * - columns: Column configuration array
       * - pageSize: Number of items per page (default: 10)
       * - sortable: Enable column sorting (default: false)
       */
      props: {
        items: [],
        columns: [],
        pageSize: 10,
        sortable: false
      }
    }
  }
}
```

## Debugging Props

**Check prop values:**

```js
created() {
  console.log('Props:', this.props);
}
```

**Watch prop changes:**

```js
created() {
  this.watch(() => this.props.userId, (newId) => {
    console.log('userId changed to:', newId);
  });
}
```

**Common issues:**
- Typo in prop name: `$:prop:nmae` vs `$props.name`
- **camelCase prop names**: `$:prop:firstName` becomes `$:prop:firstname` (use kebab-case instead)
- Forgetting to declare prop in component options
- Trying to mutate props directly
- Not emitting events for two-way binding

## Performance

**Props are efficient:**
- Reactive updates only when values change
- Shallow comparison by default
- No unnecessary re-renders

**Cost:** Minimal. Props use the same reactive system as state.

## Limitations

### No Prop Destructuring in Templates

```html
<!-- ❌ Not supported -->
<div $:text="{ name, age } = $props"></div>

<!-- ✅ Use full path -->
<div $:text="$props.name"></div>
```

### Props Can't Be Computed in Parent Template

```html
<!-- ❌ Limited expression support -->
<div $:prop:fullName="$props.firstName + ' ' + $props.lastName"></div>

<!-- ✅ Use computed or method in parent -->
<div $:prop:fullName="fullName"></div>
```

## Next Steps

- [Slots](./slots.md) - Content projection
- [Component Options](./component-options.md) - Component registration & composition
- [Lifecycle](./lifecycle.md) - Component lifecycle hooks
- [Instance Methods](./instance-methods.md) - Component API methods
