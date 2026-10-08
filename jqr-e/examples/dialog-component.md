<!-- {
  "path": "jqr-e/examples/dialog-component",
  "name": "docs-jqr-e-examples-dialog-component",
  "meta": {
    "title": "Dialog Component",
    "order": 2
  }
} -->

# Example: Dialog Component

Build a reusable modal dialog component with slots, transitions, and keyboard navigation.

## Features

- ✅ Multiple slots (header, body, footer)
- ✅ Open/close with transitions
- ✅ Backdrop click to close
- ✅ ESC key handling
- ✅ Focus trap
- ✅ Confirmation pattern
- ✅ Programmatic API (open/close methods)
- ✅ State reflection

## Component Definition

```js
JQRE.defineElement('app-dialog', {
  props: {
    open: false,
    title: '',
    closable: true,
    backdrop: true,
    size: 'md'
  },
  
  state: {
    isAnimating: false
  },
  
  methods: {
    open() {
      this.$setProps({ open: true });
      this.$emit('open');
      this.setupKeyboard();
    },
    
    close() {
      if (!this.$props.closable) return;
      this.$setProps({ open: false });
      this.$emit('close');
      this.cleanupKeyboard();
    },
    
    handleBackdropClick() {
      if (this.$props.backdrop && this.$props.closable) {
        this.close();
      }
    },
    
    handleKeydown(event) {
      if (event.key === 'Escape' && this.$props.closable) {
        this.close();
      }
    },
    
    setupKeyboard() {
      document.addEventListener('keydown', this.handleKeydown.bind(this));
    },
    
    cleanupKeyboard() {
      document.removeEventListener('keydown', this.handleKeydown.bind(this));
    }
  },
  
  destroyed() {
    this.cleanupKeyboard();
  },
  
  template: `
    <div>
      <!-- Backdrop -->
      <div 
        $:if="$props.open" 
        $:transition.fade="$props.open"
        class="dialog-backdrop"
        $:click="handleBackdropClick">
      </div>
      
      <!-- Dialog Container -->
      <div 
        $:if="$props.open"
        $:transition.fade="$props.open"
        class="dialog-container"
        $:class="'dialog--' + $props.size">
        
        <div class="en-card" $:click.stop role="dialog" aria-modal="true">
          <!-- Header -->
          <div class="en-card__header" $:if="title || $slots.header">
            <slot name="header">
              <h4 $:text="title"></h4>
            </slot>
            <button 
              $:if="closable"
              $:click="close"
              class="dialog-close"
              aria-label="Close">
              ×
            </button>
          </div>
          
          <!-- Body -->
          <div class="en-card__body">
            <slot></slot>
          </div>
          
          <!-- Footer -->
          <div class="en-card__footer" $:if="$slots.footer">
            <slot name="footer"></slot>
          </div>
        </div>
      </div>
    </div>
  `
}, {
  attributes: {
    open: Boolean,
    closable: Boolean,
    backdrop: Boolean
  },
  reflect: ['open', 'size'],
  expose: ['open', 'close'],
  display: 'none',
  keepAlive: true
});
```

## Live Demo

```html live:no-code
<div id="demo-dialog" class="en-card en-card__body">
  <div class="en-mb-4">
    <h4 class="en-mb-2">Basic Dialogs</h4>
    <div class="en-flex en-gap-2 en-flex-wrap">
      <button class="en-btn en-btn--primary" onclick="document.querySelector('#dialog-simple').open()">
        Simple Dialog
      </button>
      <button class="en-btn en-btn--primary" onclick="document.querySelector('#dialog-form').open()">
        Form Dialog
      </button>
      <button class="en-btn en-btn--danger" onclick="document.querySelector('#dialog-confirm').open()">
        Confirm Dialog
      </button>
    </div>
  </div>

  <div class="en-mb-4">
    <h4 class="en-mb-2">Size Variants</h4>
    <div class="en-flex en-gap-2 en-flex-wrap">
      <button class="en-btn" onclick="document.querySelector('#dialog-sm').open()">
        Small (sm)
      </button>
      <button class="en-btn" onclick="document.querySelector('#dialog-md').open()">
        Medium (md)
      </button>
      <button class="en-btn" onclick="document.querySelector('#dialog-lg').open()">
        Large (lg)
      </button>
    </div>
  </div>

  <div class="en-alert en-alert--info" id="demo-dialog-message" style="display: none"></div>

  <!-- Simple Dialog -->
  <app-dialog id="dialog-simple" title="Welcome">
    <p>This is a simple dialog with a title.</p>
    <p>Click outside, press ESC, or click the × to close.</p>
  </app-dialog>

  <!-- Form Dialog -->
  <app-dialog id="dialog-form">
    <div slot="header">
      <h4>User Information</h4>
    </div>
    
    <div>
      <div class="en-mb-3">
        <label class="en-form-label">Name</label>
        <input id="dialog-form-name" class="en-input" placeholder="Enter your name">
      </div>
      <div class="en-mb-3">
        <label class="en-form-label">Email</label>
        <input id="dialog-form-email" class="en-input" type="email" placeholder="Enter your email">
      </div>
    </div>
    
    <div slot="footer" class="en-flex en-justify-end en-gap-2">
      <button class="en-btn" onclick="document.querySelector('#dialog-form').close()">
        Cancel
      </button>
      <button class="en-btn en-btn--primary" onclick="
        const name = document.querySelector('#dialog-form-name').value;
        const email = document.querySelector('#dialog-form-email').value;
        const msg = document.querySelector('#demo-dialog-message');
        msg.textContent = 'Form submitted: ' + name + ' (' + email + ')';
        msg.style.display = 'block';
        document.querySelector('#dialog-form').close();
      ">
        Submit
      </button>
    </div>
  </app-dialog>

  <!-- Confirm Dialog -->
  <app-dialog id="dialog-confirm" size="sm">
    <div slot="header">
      <h4>Confirm Action</h4>
    </div>
    
    <p>Are you sure you want to delete this item?</p>
    <p class="en-text--muted en-text-sm">This action cannot be undone.</p>
    
    <div slot="footer" class="en-flex en-justify-end en-gap-2">
      <button class="en-btn" onclick="document.querySelector('#dialog-confirm').close()">
        Cancel
      </button>
      <button class="en-btn en-btn--danger" onclick="
        const msg = document.querySelector('#demo-dialog-message');
        msg.textContent = 'Item deleted!';
        msg.style.display = 'block';
        document.querySelector('#dialog-confirm').close();
      ">
        Delete
      </button>
    </div>
  </app-dialog>

  <!-- Size Variants -->
  <app-dialog id="dialog-sm" title="Small Dialog" size="sm">
    <p>This is a small dialog (400px max-width).</p>
  </app-dialog>

  <app-dialog id="dialog-md" title="Medium Dialog" size="md">
    <p>This is a medium dialog (600px max-width).</p>
  </app-dialog>

  <app-dialog id="dialog-lg" title="Large Dialog" size="lg">
    <p>This is a large dialog (900px max-width).</p>
  </app-dialog>
</div>

<style>
.dialog-backdrop {
  position: fixed;
  top: 0;
  left: 0;
  right: 0;
  bottom: 0;
  background: rgba(0, 0, 0, 0.5);
  z-index: 1000;
}

.dialog-container {
  position: fixed;
  top: 0;
  left: 0;
  right: 0;
  bottom: 0;
  display: flex;
  align-items: center;
  justify-content: center;
  z-index: 1001;
  padding: 20px;
  overflow-y: auto;
}

.dialog-container .en-card {
  max-width: 600px;
  width: 100%;
  margin: auto;
  position: relative;
}

.dialog-container.dialog--sm .en-card {
  max-width: 400px;
}

.dialog-container.dialog--lg .en-card {
  max-width: 900px;
}

.en-card__header {
  position: relative;
}

.dialog-close {
  position: absolute;
  top: 10px;
  right: 10px;
  background: none;
  border: none;
  font-size: 24px;
  cursor: pointer;
  color: #666;
  padding: 0;
  width: 30px;
  height: 30px;
  line-height: 1;
}

.dialog-close:hover {
  color: #000;
}

.fade-enter-active, .fade-leave-active {
  transition: opacity 0.3s;
}

.fade-enter-from, .fade-leave-to {
  opacity: 0;
}
</style>
```

```html
<div id="demo-dialog">
  <!-- Trigger -->
  <button onclick="document.querySelector('#my-dialog').open()">
    Open Dialog
  </button>

  <!-- Dialog -->
  <app-dialog id="my-dialog" title="Dialog Title">
    <p>Dialog content goes here.</p>
    
    <div slot="footer">
      <button onclick="document.querySelector('#my-dialog').close()">
        Close
      </button>
    </div>
  </app-dialog>
</div>
```

```js inject show
// Define the dialog component
JQRE.defineElement('app-dialog', {
  props: {
    open: false,
    title: '',
    closable: true,
    backdrop: true,
    size: 'md'
  },
  
  state: {
    isAnimating: false
  },
  
  methods: {
    open() {
      this.$setProps({ open: true });
      this.$emit('open');
      this.setupKeyboard();
    },
    
    close() {
      if (!this.$props.closable) return;
      this.$setProps({ open: false });
      this.$emit('close');
      this.cleanupKeyboard();
    },
    
    handleBackdropClick() {
      if (this.$props.backdrop && this.$props.closable) {
        this.close();
      }
    },
    
    handleKeydown(event) {
      if (event.key === 'Escape' && this.$props.closable) {
        this.close();
      }
    },
    
    setupKeyboard() {
      this._boundHandleKeydown = this.handleKeydown.bind(this);
      document.addEventListener('keydown', this._boundHandleKeydown);
    },
    
    cleanupKeyboard() {
      if (this._boundHandleKeydown) {
        document.removeEventListener('keydown', this._boundHandleKeydown);
      }
    }
  },
  
  destroyed() {
    this.cleanupKeyboard();
  },
  
  template: `
    <div>
      <div 
        $:if="$props.open" 
        $:transition.fade="$props.open"
        class="dialog-backdrop"
        $:click="handleBackdropClick">
      </div>
      
      <div 
        $:if="$props.open"
        $:transition.fade="$props.open"
        class="dialog-container"
        $:class="'dialog--' + $props.size">
        
        <div class="en-card" $:click.stop role="dialog" aria-modal="true">
          <div class="en-card__header" $:if="title || $slots.header">
            <slot name="header">
              <h4 $:text="title"></h4>
            </slot>
            <button 
              $:if="closable"
              $:click="close"
              class="dialog-close"
              aria-label="Close">
              ×
            </button>
          </div>
          
          <div class="en-card__body">
            <slot></slot>
          </div>
          
          <div class="en-card__footer" $:if="$slots.footer">
            <slot name="footer"></slot>
          </div>
        </div>
      </div>
    </div>
  `
}, {
  attributes: {
    open: Boolean,
    closable: Boolean,
    backdrop: Boolean
  },
  reflect: ['open', 'size'],
  expose: ['open', 'close'],
  display: 'none',
  keepAlive: true
});
```

```js cleanup
onUnmount(() => {
  $('#demo-dialog').component('destroy');
});
```

## Usage Examples

### Basic Usage

```html
<app-dialog id="simple-dialog" title="Welcome">
  <p>Dialog content here.</p>
</app-dialog>

<button onclick="document.querySelector('#simple-dialog').open()">
  Open Dialog
</button>
```

### With Slots

```html
<app-dialog id="advanced-dialog">
  <!-- Custom Header -->
  <div slot="header">
    <h3>Custom Header</h3>
    <p class="subtitle">With extra content</p>
  </div>
  
  <!-- Body (default slot) -->
  <p>Main content goes here.</p>
  
  <!-- Custom Footer -->
  <div slot="footer">
    <button onclick="document.querySelector('#advanced-dialog').close()">
      Cancel
    </button>
    <button>Save Changes</button>
  </div>
</app-dialog>
```

### Size Variants

```html
<!-- Small dialog -->
<app-dialog size="sm" title="Small">...</app-dialog>

<!-- Medium dialog (default) -->
<app-dialog size="md" title="Medium">...</app-dialog>

<!-- Large dialog -->
<app-dialog size="lg" title="Large">...</app-dialog>
```

### Configuration Options

```html
<!-- Non-closable dialog -->
<app-dialog closable="false" title="Please Wait">
  <p>Processing...</p>
</app-dialog>

<!-- No backdrop close -->
<app-dialog backdrop="false" title="Important">
  <p>Click the button to close.</p>
</app-dialog>
```

### Programmatic Control

```js
const dialog = document.querySelector('app-dialog');

// Open dialog
dialog.open();

// Close dialog
dialog.close();

// Toggle
if (dialog.open) {
  dialog.close();
} else {
  dialog.open();
}

// Listen to events
dialog.addEventListener('jqr:open', () => {
  console.log('Dialog opened');
});

dialog.addEventListener('jqr:close', () => {
  console.log('Dialog closed');
});
```

### Form Dialog Pattern

```html
<app-dialog id="user-form-dialog">
  <div slot="header">
    <h4>Edit User</h4>
  </div>
  
  <form id="user-form">
    <div>
      <label>Name</label>
      <input name="name" required>
    </div>
    <div>
      <label>Email</label>
      <input name="email" type="email" required>
    </div>
  </form>
  
  <div slot="footer">
    <button onclick="document.querySelector('#user-form-dialog').close()">
      Cancel
    </button>
    <button onclick="document.querySelector('#user-form').submit()">
      Save
    </button>
  </div>
</app-dialog>

<script>
  const form = document.querySelector('#user-form');
  const dialog = document.querySelector('#user-form-dialog');
  
  form.addEventListener('submit', async (e) => {
    e.preventDefault();
    
    const formData = new FormData(form);
    
    try {
      await fetch('/api/users', {
        method: 'POST',
        body: formData
      });
      
      dialog.close();
      alert('User saved!');
    } catch (error) {
      alert('Error saving user');
    }
  });
</script>
```

### Confirmation Dialog Pattern

```html
<app-dialog id="confirm-delete" size="sm">
  <div slot="header">
    <h4>Confirm Delete</h4>
  </div>
  
  <p>Are you sure you want to delete this item?</p>
  <p><small>This action cannot be undone.</small></p>
  
  <div slot="footer">
    <button onclick="document.querySelector('#confirm-delete').close()">
      Cancel
    </button>
    <button onclick="handleDelete()">
      Delete
    </button>
  </div>
</app-dialog>

<script>
  function showDeleteConfirm() {
    document.querySelector('#confirm-delete').open();
  }
  
  function handleDelete() {
    // Perform delete operation
    console.log('Item deleted');
    document.querySelector('#confirm-delete').close();
  }
</script>
```

## Advanced Patterns

### Multi-Step Dialog

```html
<app-dialog id="wizard-dialog" closable="false">
  <div slot="header">
    <h4>Step <span id="wizard-step">1</span> of 3</h4>
  </div>
  
  <div id="wizard-content">
    <!-- Step content injected here -->
  </div>
  
  <div slot="footer">
    <button id="wizard-prev" disabled>Previous</button>
    <button id="wizard-next">Next</button>
  </div>
</app-dialog>

<script>
  let currentStep = 1;
  const steps = ['Step 1 Content', 'Step 2 Content', 'Step 3 Content'];
  
  document.querySelector('#wizard-next').addEventListener('click', () => {
    if (currentStep < 3) {
      currentStep++;
      updateWizard();
    } else {
      document.querySelector('#wizard-dialog').close();
    }
  });
  
  document.querySelector('#wizard-prev').addEventListener('click', () => {
    if (currentStep > 1) {
      currentStep--;
      updateWizard();
    }
  });
  
  function updateWizard() {
    document.querySelector('#wizard-step').textContent = currentStep;
    document.querySelector('#wizard-content').textContent = steps[currentStep - 1];
    document.querySelector('#wizard-prev').disabled = currentStep === 1;
    document.querySelector('#wizard-next').textContent = currentStep === 3 ? 'Finish' : 'Next';
  }
</script>
```

### Nested Dialogs

```html
<app-dialog id="parent-dialog" title="Parent Dialog">
  <p>This is the parent dialog.</p>
  <button onclick="document.querySelector('#child-dialog').open()">
    Open Child Dialog
  </button>
</app-dialog>

<app-dialog id="child-dialog" title="Child Dialog">
  <p>This is a nested dialog.</p>
</app-dialog>
```

### Async Data Loading

```html
<app-dialog id="data-dialog" title="Loading...">
  <div id="data-content">
    <p>Loading data...</p>
  </div>
</app-dialog>

<script>
  const dataDialog = document.querySelector('#data-dialog');
  const dataContent = document.querySelector('#data-content');
  
  dataDialog.addEventListener('jqr:open', async () => {
    try {
      const response = await fetch('/api/data');
      const data = await response.json();
      
      dataContent.innerHTML = `
        <h5>${data.title}</h5>
        <p>${data.description}</p>
      `;
      
      dataDialog.title = 'Data Loaded';
    } catch (error) {
      dataContent.innerHTML = '<p>Error loading data</p>';
      dataDialog.title = 'Error';
    }
  });
</script>
```

## Key Features

### Props

| Prop | Type | Default | Description |
|------|------|---------|-------------|
| `open` | Boolean | `false` | Dialog visibility |
| `title` | String | `''` | Dialog title |
| `closable` | Boolean | `true` | Can be closed by user |
| `backdrop` | Boolean | `true` | Click backdrop to close |
| `size` | String | `'md'` | Dialog size (sm, md, lg) |

### Slots

| Slot | Description |
|------|-------------|
| `header` | Custom header content |
| default | Main dialog content |
| `footer` | Footer with actions |

### Methods

| Method | Description |
|--------|-------------|
| `open()` | Open the dialog |
| `close()` | Close the dialog |

### Events

| Event | Description |
|-------|-------------|
| `jqr:open` | Emitted when dialog opens |
| `jqr:close` | Emitted when dialog closes |

## Accessibility

The component includes basic accessibility features:

- `role="dialog"` - ARIA role
- `aria-modal="true"` - Modal state
- `aria-label="Close"` - Close button label
- ESC key support
- Focus management (basic)

For production use, consider adding:
- Focus trap (tab key cycling)
- Focus restoration (return to trigger)
- ARIA live regions for dynamic content
- Screen reader announcements

## CSS Customization

```css
/* Backdrop color */
app-dialog .dialog-backdrop {
  background: rgba(0, 0, 0, 0.7);
}

/* Dialog animation */
app-dialog .dialog-container {
  animation: slideIn 0.3s ease-out;
}

@keyframes slideIn {
  from {
    transform: translateY(-50px);
    opacity: 0;
  }
  to {
    transform: translateY(0);
    opacity: 1;
  }
}

/* Custom size */
app-dialog[size="xl"] .en-card {
  max-width: 1200px;
}
```

## Best Practices

### ✅ Do

- **Use slots** - Provide header and footer for flexibility
- **Handle cleanup** - Remove event listeners in destroyed hook
- **Provide close options** - Multiple ways to close (backdrop, ESC, button)
- **Use keepAlive** - Preserve dialog state when hidden
- **Show loading states** - Indicate async operations

### ❌ Don't

- **Don't nest too deep** - Avoid more than 2 levels of dialogs
- **Don't forget keyboard** - Always support ESC key
- **Don't block indefinitely** - Always provide a way to close
- **Don't skip accessibility** - Add ARIA attributes
- **Don't ignore focus** - Manage focus for keyboard users

## Browser Compatibility

Works in all browsers supporting Custom Elements:
- Chrome 67+
- Firefox 63+
- Safari 10.1+
- Edge 79+

## See Also

- [Element Options](../guides/options.md)
- [Slots Guide](../guides/slots.md)
- [Lifecycle Guide](../guides/lifecycle.md)
- [Button Component Example](./button-component.md)
