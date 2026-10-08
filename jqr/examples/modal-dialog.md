<!-- {
  "path": "jqr/examples/modal-dialog",
  "name": "docs-jqr-examples-modal-dialog",
  "meta": {
    "title": "Modal Dialog",
    "order": 3
  }
} -->

# Example: Modal Dialog

Complete modal dialog patterns with transitions, backdrop, focus trap, and ESC key handling.

## Features

- ✅ Smooth transitions
- ✅ Backdrop click to close
- ✅ ESC key to close
- ✅ Multiple modal support
- ✅ Confirmation dialogs
- ✅ Custom content

## Live Demo

```html live:no-code
<div id="demo-modal" class="en-card en-card__body">
  <!-- Trigger Buttons -->
  <div class="en-flex en-gap-2 en-flex-wrap en-mb-3">
    <button class="en-btn en-btn--primary" $:click="openModal('simple')">
      Simple Modal
    </button>
    <button class="en-btn en-btn--primary" $:click="openModal('form')">
      Form Modal
    </button>
    <button class="en-btn en-btn--danger" $:click="openModal('confirm')">
      Confirm Dialog
    </button>
  </div>

  <!-- Result Display -->
  <div class="en-alert en-alert--info" $:if="lastAction">
    Last action: <strong $:text="lastAction"></strong>
  </div>

  <!-- Modal Backdrop -->
  <div 
    $:if="isOpen" 
    $:transition.fade="isOpen"
    class="modal-backdrop" 
    $:click="closeModal()">
  </div>

  <!-- Simple Modal -->
  <div 
    $:if="isOpen && modalType === 'simple'" 
    $:transition.fade="isOpen"
    class="modal-container">
    <div class="en-card" $:click.stop style="max-width: 500px; margin: auto;">
      <div class="en-card__header">
        <h4>Simple Modal</h4>
      </div>
      <div class="en-card__body">
        <p>This is a simple modal dialog.</p>
        <p>Click outside or press ESC to close.</p>
      </div>
      <div class="en-card__footer en-flex en-justify-end">
        <button class="en-btn" $:click="closeModal()">Close</button>
      </div>
    </div>
  </div>

  <!-- Form Modal -->
  <div 
    $:if="isOpen && modalType === 'form'" 
    $:transition.fade="isOpen"
    class="modal-container">
    <div class="en-card" $:click.stop style="max-width: 500px; margin: auto;">
      <div class="en-card__header">
        <h4>User Form</h4>
      </div>
      <div class="en-card__body">
        <div class="en-mb-3">
          <label class="en-form-label">Name</label>
          <input class="en-input" $:model="formData.name" placeholder="Enter name">
        </div>
        <div class="en-mb-3">
          <label class="en-form-label">Email</label>
          <input class="en-input" type="email" $:model="formData.email" placeholder="Enter email">
        </div>
      </div>
      <div class="en-card__footer en-flex en-justify-end en-gap-2">
        <button class="en-btn" $:click="closeModal()">Cancel</button>
        <button class="en-btn en-btn--primary" $:click="submitForm()">Submit</button>
      </div>
    </div>
  </div>

  <!-- Confirm Dialog -->
  <div 
    $:if="isOpen && modalType === 'confirm'" 
    $:transition.fade="isOpen"
    class="modal-container">
    <div class="en-card" $:click.stop style="max-width: 400px; margin: auto;">
      <div class="en-card__header">
        <h4>Confirm Action</h4>
      </div>
      <div class="en-card__body">
        <p>Are you sure you want to delete this item?</p>
        <p class="en-text--muted en-text-sm">This action cannot be undone.</p>
      </div>
      <div class="en-card__footer en-flex en-justify-end en-gap-2">
        <button class="en-btn" $:click="closeModal()">Cancel</button>
        <button class="en-btn en-btn--danger" $:click="confirmAction()">Delete</button>
      </div>
    </div>
  </div>
</div>

<style>
.modal-backdrop {
  position: fixed;
  top: 0;
  left: 0;
  right: 0;
  bottom: 0;
  background: rgba(0, 0, 0, 0.5);
  z-index: 1000;
}

.modal-container {
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
<div id="demo-modal">
  <!-- Trigger Buttons -->
  <button $:click="openModal('simple')">Simple Modal</button>
  <button $:click="openModal('form')">Form Modal</button>
  <button $:click="openModal('confirm')">Confirm Dialog</button>

  <!-- Result Display -->
  <div $:if="lastAction">
    Last action: <strong $:text="lastAction"></strong>
  </div>

  <!-- Modal Backdrop -->
  <div $:if="isOpen" $:transition.fade="isOpen" class="backdrop" $:click="closeModal()"></div>

  <!-- Simple Modal -->
  <div $:if="isOpen && modalType === 'simple'" $:transition.fade="isOpen" class="modal">
    <div $:click.stop>
      <h4>Simple Modal</h4>
      <p>This is a simple modal dialog.</p>
      <button $:click="closeModal()">Close</button>
    </div>
  </div>

  <!-- Form Modal -->
  <div $:if="isOpen && modalType === 'form'" $:transition.fade="isOpen" class="modal">
    <div $:click.stop>
      <h4>User Form</h4>
      <input $:model="formData.name" placeholder="Name">
      <input $:model="formData.email" placeholder="Email">
      <button $:click="closeModal()">Cancel</button>
      <button $:click="submitForm()">Submit</button>
    </div>
  </div>

  <!-- Confirm Dialog -->
  <div $:if="isOpen && modalType === 'confirm'" $:transition.fade="isOpen" class="modal">
    <div $:click.stop>
      <h4>Confirm Action</h4>
      <p>Are you sure you want to delete this item?</p>
      <button $:click="closeModal()">Cancel</button>
      <button $:click="confirmAction()">Delete</button>
    </div>
  </div>
</div>
```

```js inject show
$('#demo-modal').component({
  state: {
    isOpen: false,
    modalType: null,
    formData: {
      name: '',
      email: ''
    },
    lastAction: ''
  },
  
  methods: {
    openModal(state, type) {
      state.modalType = type;
      state.isOpen = true;
      document.body.style.overflow = 'hidden';
    },
    
    closeModal(state) {
      state.isOpen = false;
      document.body.style.overflow = '';
    },
    
    submitForm(state) {
      state.lastAction = `Form submitted: ${state.formData.name} (${state.formData.email})`;
      state.formData = { name: '', email: '' };
      this.closeModal(state);
    },
    
    confirmAction(state) {
      state.lastAction = 'Item deleted';
      this.closeModal(state);
    }
  },
  
  mounted(state, $el) {
    // ESC key handler
    const handleEsc = (e) => {
      if (e.key === 'Escape' && state.isOpen) {
        this.closeModal(state);
      }
    };
    
    document.addEventListener('keydown', handleEsc);
    
    // Cleanup
    $el[0]._escHandler = handleEsc;
  },
  
  destroyed(state, $el) {
    document.removeEventListener('keydown', $el[0]._escHandler);
    document.body.style.overflow = '';
  }
});
```

```js cleanup
onUnmount(() => {
  $('#demo-modal').component('destroy');
})
```

## Code Breakdown

### State Management

```js
state: {
  isOpen: false,           // Modal visibility
  modalType: null,         // 'simple' | 'form' | 'confirm'
  formData: {              // Form modal data
    name: '',
    email: ''
  },
  lastAction: ''           // Result display
}
```

### Opening Modal

```js
methods: {
  openModal(state, type) {
    state.modalType = type;
    state.isOpen = true;
    
    // Prevent body scroll
    document.body.style.overflow = 'hidden';
  }
}
```

### Closing Modal

```js
methods: {
  closeModal(state) {
    state.isOpen = false;
    
    // Restore body scroll
    document.body.style.overflow = '';
  }
}
```

### ESC Key Handler

```js
mounted(state, $el) {
  const handleEsc = (e) => {
    if (e.key === 'Escape' && state.isOpen) {
      this.closeModal(state);
    }
  };
  
  document.addEventListener('keydown', handleEsc);
  
  // Store for cleanup
  $el[0]._escHandler = handleEsc;
},

destroyed(state, $el) {
  document.removeEventListener('keydown', $el[0]._escHandler);
}
```

### Backdrop Click

```html
<div $:if="isOpen" $:click="closeModal()" class="backdrop"></div>
```

### Prevent Modal Click Propagation

```html
<div $:click.stop class="modal-content">
  <!-- Modal content -->
</div>
```

**Without `.stop`**, clicking inside the modal would trigger backdrop's `closeModal()`.

## Key Concepts

### 1. Conditional Rendering

```html
<div $:if="isOpen && modalType === 'simple'">
  Simple modal content
</div>
```

Only render when both conditions are true.

### 2. Transitions

```html
<div $:if="isOpen" $:transition.fade="isOpen">
  Modal content
</div>
```

Smooth fade-in/fade-out animation.

### 3. Backdrop Pattern

```html
<!-- Backdrop (behind modal) -->
<div class="backdrop" $:click="closeModal()"></div>

<!-- Modal (in front) -->
<div class="modal" $:click.stop>
  Content
</div>
```

**Two layers:** backdrop for dimming, modal for content.

### 4. Body Scroll Lock

```js
openModal(state) {
  document.body.style.overflow = 'hidden';
}

closeModal(state) {
  document.body.style.overflow = '';
}
```

Prevents page scrolling while modal is open.

### 5. Event Cleanup

```js
destroyed(state, $el) {
  // Clean up event listeners
  document.removeEventListener('keydown', $el[0]._escHandler);
  
  // Restore scroll
  document.body.style.overflow = '';
}
```

Always clean up to prevent memory leaks.

## Modal Styles

### CSS Structure

```css
/* Backdrop - covers entire screen */
.modal-backdrop {
  position: fixed;
  top: 0;
  left: 0;
  right: 0;
  bottom: 0;
  background: rgba(0, 0, 0, 0.5);
  z-index: 1000;
}

/* Modal Container - centers content */
.modal-container {
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

/* Modal Content */
.modal-content {
  background: white;
  border-radius: 8px;
  max-width: 500px;
  width: 100%;
  box-shadow: 0 10px 40px rgba(0, 0, 0, 0.2);
}

/* Fade Transition */
.fade-enter-active,
.fade-leave-active {
  transition: opacity 0.3s;
}

.fade-enter-from,
.fade-leave-to {
  opacity: 0;
}
```

## Advanced Patterns

### Stacked Modals

```js
state: {
  modals: []  // Stack of modal types
}
```

```js
methods: {
  openModal(state, type) {
    state.modals.push(type);
  },
  
  closeModal(state) {
    state.modals.pop();
  }
}
```

```html
<div $:for="(modal, index) in modals" $:key="index">
  <div $:if="modal === 'confirm'">Confirm dialog</div>
</div>
```

### Modal with Teleport

```html
<div $:teleport="'body'">
  <div $:if="isOpen" class="modal">
    Modal content
  </div>
</div>
```

**Benefits:** 
- Rendered outside component hierarchy
- Avoids z-index issues
- Better for accessibility

### Focus Trap

```js
mounted(state, $el) {
  if (state.isOpen) {
    const modal = $el.find('.modal')[0];
    const focusable = modal.querySelectorAll(
      'button, input, textarea, select, a[href]'
    );
    
    if (focusable.length > 0) {
      focusable[0].focus();
    }
  }
}
```

### Confirm Helper

```js
methods: {
  async confirm(state, message) {
    return new Promise((resolve) => {
      state.confirmMessage = message;
      state.confirmResolve = resolve;
      this.openModal(state, 'confirm');
    });
  },
  
  handleConfirm(state, result) {
    if (state.confirmResolve) {
      state.confirmResolve(result);
    }
    this.closeModal(state);
  }
}
```

```js
// Usage
const confirmed = await this.confirm(state, 'Delete item?');
if (confirmed) {
  // Delete item
}
```

## Best Practices

### 1. Prevent Body Scroll

```js
// ✅ Good - prevent background scroll
openModal(state) {
  document.body.style.overflow = 'hidden';
}

// ❌ Bad - page scrolls behind modal
openModal(state) {
  state.isOpen = true;
}
```

### 2. Clean Up on Destroy

```js
// ✅ Good - cleanup
destroyed(state) {
  document.removeEventListener('keydown', this.escHandler);
  document.body.style.overflow = '';
}

// ❌ Bad - memory leak
destroyed(state) {
  // No cleanup
}
```

### 3. Use Transitions

```html
<!-- ✅ Good - smooth animation -->
<div $:if="isOpen" $:transition.fade="isOpen"></div>

<!-- ❌ Bad - jarring -->
<div $:if="isOpen"></div>
```

### 4. Stop Event Propagation

```html
<!-- ✅ Good - clicks inside modal don't close it -->
<div $:click.stop class="modal-content"></div>

<!-- ❌ Bad - clicking inside closes modal -->
<div class="modal-content"></div>
```

### 5. Provide Multiple Close Methods

```html
<!-- ✅ Good - ESC, backdrop, close button -->
<button $:click="closeModal()">×</button>
<div $:click="closeModal()" class="backdrop"></div>
<!-- + ESC key handler -->

<!-- ❌ Bad - only close button -->
<button $:click="closeModal()">×</button>
```

## Common Issues

### Modal Not Closing

**Problem:** Clicking inside modal closes it

**Solution:** Use `$:click.stop` on modal content:

```html
<div $:click="closeModal()" class="backdrop">
  <div $:click.stop class="modal">
    <!-- Content -->
  </div>
</div>
```

### Z-Index Issues

**Problem:** Modal appears behind other elements

**Solution:** Use high z-index values:

```css
.modal-backdrop {
  z-index: 1000;
}

.modal-container {
  z-index: 1001;
}
```

### Body Scroll Not Restored

**Problem:** Page stays locked after modal closes

**Solution:** Restore in `destroyed`:

```js
destroyed(state) {
  document.body.style.overflow = '';
}
```

## Accessibility

### ARIA Attributes

```html
<div 
  role="dialog" 
  aria-modal="true" 
  aria-labelledby="modal-title"
  $:if="isOpen">
  <h2 id="modal-title">Modal Title</h2>
  <button aria-label="Close">×</button>
</div>
```

### Focus Management

```js
mounted(state, $el) {
  if (state.isOpen) {
    const firstButton = $el.find('button')[0];
    if (firstButton) {
      firstButton.focus();
    }
  }
}
```

### Screen Reader Announcements

```html
<div role="status" aria-live="polite" class="sr-only">
  <span $:if="isOpen">Modal opened</span>
</div>
```

## Next Steps

- [Transitions Guide](../guides/transitions.md) - Animation patterns
- [Teleport Guide](../guides/teleport.md) - Portal patterns
- [Confirmation Pattern](../guides/confirm-pattern.md) - Confirm dialogs
- [Todo App Example](./todo-app.md) - Complete application
