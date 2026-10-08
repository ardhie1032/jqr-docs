<!-- {
  "path": "jqr/guides/confirm-pattern",
  "name": "docs-jqr-guides-confirm-pattern",
  "meta": {
    "title": "Confirmation Pattern",
    "order": 15
  }
} -->

# Confirmation Pattern

Confirmations prevent accidental destructive actions by asking users to verify their intent. JQR provides built-in confirmation support through modifiers and events.

## Why Confirmations?

Confirmations protect users from:
- Accidental deletions
- Unintended form submissions
- Critical setting changes
- Data loss scenarios

**Good UX principle:** Confirm destructive or irreversible actions, but don't overuse confirmations for routine tasks.

## `.confirm` Modifier on Events

Add `.confirm` to any event handler to show a browser confirmation dialog:

```html live:no-code
<div id="demo-event-confirm" class="en-card">
  <div class="en-card__body">
    <h3>Delete Item</h3>
    <div class="en-alert en-alert--warning">
      <p style="margin: 0;">Click the delete button to see browser confirmation.</p>
    </div>
    <button 
      class="en-btn en-btn--danger" 
      style="margin-top: 1rem;"
      $:click.confirm="deleteItem"
      data-confirm-message="Are you sure you want to delete this item? This action cannot be undone.">
      Delete Item
    </button>
    <div class="en-alert en-alert--success" style="margin-top: 1rem;" $:if="deleted">
      Item deleted successfully!
    </div>
  </div>
</div>
```

```html
<div id="demo-event-confirm">
  <h3>Delete Item</h3>
  <button 
    $:click.confirm="deleteItem"
    data-confirm-message="Are you sure?">
    Delete Item
  </button>
  <div $:if="deleted">Item deleted!</div>
</div>
```

```js inject show
$('#demo-event-confirm').component({
  state: { deleted: false },
  methods: {
    deleteItem(state) {
      this.deleted = true;
      setTimeout(() => this.deleted = false, 3000);
    }
  }
});
```

```js cleanup
onUnmount(() => {
  $('#demo-event-confirm').component('destroy');
})
```

**How it works:**
1. User clicks button
2. Browser shows native `window.confirm()` with custom message
3. If user clicks "OK", handler executes
4. If user clicks "Cancel", handler is prevented

**Custom message:**
- Use `data-confirm-message` attribute for custom text
- Falls back to "Are you sure?" if not specified

## `.confirm` Modifier on Models

Confirm changes to form inputs before accepting them:

```html live:no-code
<div id="demo-model-confirm" class="en-card">
  <div class="en-card__body">
    <h3>Critical Setting</h3>
    <label class="en-check">
      <input type="checkbox" $:model.confirm="deleteMode" $:on:confirm="confirmDelete">
      <span>Enable delete mode (requires confirmation)</span>
    </label>
    <div 
      class="en-alert" 
      style="margin-top: 1rem;"
      $:class="deleteMode ? 'en-alert--danger' : 'en-alert--muted'">
      <strong>Delete mode: <span $:text="deleteMode ? 'ENABLED' : 'Disabled'"></span></strong>
    </div>
  </div>
</div>
```

```html
<div id="demo-model-confirm">
  <h3>Critical Setting</h3>
  <label>
    <input type="checkbox" $:model.confirm="deleteMode">
    Enable delete mode
  </label>
  <p>Status: <strong $:text="deleteMode ? 'ENABLED' : 'Disabled'"></strong></p>
</div>
```

```js inject show
$('#demo-model-confirm').component({
  state: { deleteMode: false },
  methods: {
    confirmDelete: (event) => {
      const { oldValue, newValue, accept, reject } = event.detail
      if (window.confirm(`"${newValue ? 'Enable' : 'Disable'}" deleted mode?`)) {
        accept()
      } else {
        reject()
      }
    }
  }
});
```

```js cleanup
onUnmount(() => {
  $('#demo-model-confirm').component('destroy');
})
```

**When to use:**
- Critical toggles (admin mode, delete mode)
- Permanent settings
- High-impact changes

## Custom Confirmation Handler

Override default `window.confirm()` with custom UI:

```html live:no-code
<div id="demo-custom-confirm" class="en-card">
  <div class="en-card__body">
    <h3>Custom Confirmation</h3>
    <button 
      class="en-btn en-btn--primary" 
      $:click.confirm="performAction"
      data-confirm-message="This will change your settings.">
      Perform Action
    </button>
    
    <div class="en-alert en-alert--success" $:if="actionPerformed">
      Action completed!
    </div>
  </div>
</div>

<dialog id="demo-custom-dialog" class="en-modal">
  <div class="en-modal__header">
    <h3 class="en-modal__title" id="demo-confirm-title">Confirm Action</h3>
  </div>
  <div class="en-modal__body">
    <p id="demo-confirm-message">Are you sure?</p>
  </div>
  <div class="en-modal__footer" style="display: flex; gap: 0.5rem; justify-content: flex-end;">
    <button class="en-btn en-btn--primary" id="demo-confirm-ok">OK</button>
    <button class="en-btn en-btn--outline" id="demo-confirm-cancel">Cancel</button>
  </div>
</dialog>
```

```html
<div id="demo-custom-confirm">
  <h3>Custom Confirmation</h3>
  <button 
    $:click.confirm="performAction"
    data-confirm-message="This will change your settings.">
    Perform Action
  </button>
  <div $:if="actionPerformed">Action completed!</div>
</div>

<dialog id="demo-custom-dialog">
  <h3 id="demo-confirm-title">Confirm</h3>
  <p id="demo-confirm-message">Are you sure?</p>
  <button id="demo-confirm-ok">OK</button>
  <button id="demo-confirm-cancel">Cancel</button>
</dialog>
```

```js inject show
$('#demo-custom-confirm').component({
  state: { actionPerformed: false },
  methods: {
    performAction() {
      this.actionPerformed = true;
      setTimeout(() => this.actionPerformed = false, 3000);
    }
  },
  mounted(state, $el) {          // <-- bukan onMounted
    this.$el.on('confirm', (e) => {
      e.preventDefault();
      const detail = e.detail;
      const message = detail.message;

      const dialog = document.getElementById('demo-custom-dialog');
      const messageEl = document.getElementById('demo-confirm-message');
      const okBtn = document.getElementById('demo-confirm-ok');
      const cancelBtn = document.getElementById('demo-confirm-cancel');

      messageEl.textContent = message;
      dialog.showModal();

      const handleOk = () => { detail.accept(); dialog.close(); cleanup(); };
      const handleCancel = () => { detail.reject(); dialog.close(); cleanup(); };
      const cleanup = () => {
        okBtn.removeEventListener('click', handleOk);
        cancelBtn.removeEventListener('click', handleCancel);
      };
      okBtn.addEventListener('click', handleOk);
      cancelBtn.addEventListener('click', handleCancel);
    });
  }
});
```

```js cleanup
onUnmount(() => {
  $('#demo-custom-confirm').component('destroy');
})
```

*Alternative:*

```html live:collapse
<div id="demo-custom-confirm-2">
  <h3>Custom Confirmation</h3>
  <button class="en-btn"
    $:click.confirm="performAction"
    $:on:confirm="confirmAction"
    data-confirm-message="This will change your settings.">
    Perform Action
  </button>
  <div $:if="actionPerformed">Action completed!</div>
</div>

<dialog id="demo-custom-dialog-2" style="position: fixed; margin: auto;">
  <h4 class="confirm-title">Confirm</h4>
  <p class="confirm-message">Are you sure?</p>
  <button class="confirm-ok">OK</button>
  <button class="confirm-cancel">Cancel</button>
</dialog>
```

```js inject show
$('#demo-custom-confirm-2').component({
  state: { actionPerformed: false },
  methods: {
    performAction() {
      this.actionPerformed = true;
      setTimeout(() => this.actionPerformed = false, 3000);
    },
    confirmAction: (event) => {
      event.preventDefault();
      const { message, accept, reject } = event.detail;

      const dialog = document.getElementById('demo-custom-dialog-2');
      const messageEl = dialog.querySelector('.confirm-message');
      const okBtn = dialog.querySelector('.confirm-ok');
      const cancelBtn = dialog.querySelector('.confirm-cancel');

      messageEl.textContent = message;
      dialog.showModal();

      const handleOk = () => { accept(); dialog.close(); cleanup(); };
      const handleCancel = () => { reject(); dialog.close(); cleanup(); };
      const cleanup = () => {
        okBtn.removeEventListener('click', handleOk);
        cancelBtn.removeEventListener('click', handleCancel);
      };
      okBtn.addEventListener('click', handleOk);
      cancelBtn.addEventListener('click', handleCancel);
    }
  }
});
```

**Key API:**
- Listen to `confirm` event: `$:on:confirm="handler"`
- `event.detail.accept()` - Proceed with action
- `event.detail.reject()` - Cancel action
- `event.detail.oldValue` - Previous value (for models)
- `event.detail.newValue` - New value (for models)

## Modal Confirmation Pattern

Common pattern: confirmation modal before destructive action:

```html live:no-code
<template id="demo-confirm-modal">
  <dialog class="en-modal" $:model:open.on-close="isOpen">
    <div class="en-modal__header">
      <h3 class="en-modal__title"><slot $:name="title">Confirm</slot></h3>
      <button class="en-modal__close" $:click="cancel">&times;</button>
    </div>
    <div class="en-modal__body">
      <slot $:default>Are you sure?</slot>
    </div>
    <div class="en-modal__footer" style="display: flex; gap: 0.5rem; justify-content: flex-end;">
      <button class="en-btn en-btn--primary" $:click="confirm">
        <slot $:name="confirm-text">Confirm</slot>
      </button>
      <button class="en-btn en-btn--outline" $:click="cancel">Cancel</button>
    </div>
  </dialog>
</template>

<div id="demo-modal-pattern">
  <div class="en-card">
    <div class="en-card__body">
      <h3>Delete Account</h3>
      <p class="en-text--muted">This action requires confirmation</p>
      <button class="en-btn en-btn--danger" $:click="showDeleteModal = true">
        Delete Account
      </button>
      
      <div class="en-alert en-alert--success" style="margin-top: 1rem;" $:if="accountDeleted">
        Account deleted successfully!
      </div>
      
      <div $:component="demo-confirm-modal" $:model:open="showDeleteModal">
        <template $:slot="title">Delete Account</template>
        <p style="margin: 0;">Are you sure you want to delete your account? This action cannot be undone.</p>
        <template $:slot="confirm-text">Delete</template>
      </div>
    </div>
  </div>
</div>
```

```html
<template id="demo-confirm-modal">
  <dialog $:model:open.on-close="isOpen">
    <h3><slot $:name="title">Confirm</slot></h3>
    <div><slot $:default>Are you sure?</slot></div>
    <button $:click="confirm">OK</button>
    <button $:click="cancel">Cancel</button>
  </dialog>
</template>

<div id="demo-modal-pattern">
  <h3>Delete Account</h3>
  <button $:click="showDeleteModal = true">Delete Account</button>
  <div $:if="accountDeleted">Account deleted!</div>
  
  <div $:component="confirm-modal" $:model:open="showDeleteModal">
    <template $:slot="title">Delete Account</template>
    <p>Are you sure? This cannot be undone.</p>
  </div>
</div>
```

```js inject show
$('#demo-modal-pattern').component({
  components: {
    'demo-confirm-modal': {
      props: { open: false },
      emits: ['update:open', 'confirm', 'cancel'],
      state: (c) => {
        return { isOpen: c.open }
      },
      created() {
        this.$watch(() => this.$props.open, (val) => {
          this.isOpen = val;
        });
      },
      methods: {
        confirm() {
          this.$emit('confirm');
          this.$emit('update:open', false);
        },
        cancel() {
          this.$emit('cancel');
          this.$emit('update:open', false);
        }
      }
    }
  },
  state: {
    showDeleteModal: false,
    accountDeleted: false
  },
  mounted() {
    this.$el.on('confirm', () => {
      this.accountDeleted = true;
      setTimeout(() => this.accountDeleted = false, 3000);
    });
  }
});
```

```js cleanup
onUnmount(() => {
  $('#demo-modal-pattern').component('destroy');
})
```

## Inline Confirmation

Alternative: inline confirmation without modal:

```html live:no-code
<div id="demo-inline-confirm" class="en-card">
  <div class="en-card__body">
    <h3>User Management</h3>
    <div 
      $:for="user in users" 
      $:key="user.id"
      class="en-flex en-gap-2 en-items-center"
      style="margin-bottom: 0.5rem; padding: 0.5rem; border: 1px solid #e5e7eb; border-radius: 0.25rem;">
      <span style="flex: 1;" $:text="user.name"></span>
      
      <div $:if="confirmingDelete !== user.id">
        <button 
          class="en-btn en-btn--sm en-btn--danger" 
          $:click="confirmingDelete = user.id">
          Delete
        </button>
      </div>
      
      <div $:if="confirmingDelete === user.id" class="en-flex en-gap-2">
        <span class="en-text--muted" style="font-size: 0.875rem;">Are you sure?</span>
        <button 
          class="en-btn en-btn--sm en-btn--danger" 
          $:click="deleteUser(user.id)">
          Yes, Delete
        </button>
        <button 
          class="en-btn en-btn--sm en-btn--outline" 
          $:click="confirmingDelete = null">
          Cancel
        </button>
      </div>
    </div>
  </div>
</div>
```

```html
<div id="demo-inline-confirm">
  <h3>User Management</h3>
  <div $:for="user in users" $:key="user.id">
    <span $:text="user.name"></span>
    
    <div $:if="confirmingDelete !== user.id">
      <button $:click="confirmingDelete = user.id">Delete</button>
    </div>
    
    <div $:if="confirmingDelete === user.id">
      <span>Are you sure?</span>
      <button $:click="deleteUser(user.id)">Yes</button>
      <button $:click="confirmingDelete = null">Cancel</button>
    </div>
  </div>
</div>
```

```js inject show
$('#demo-inline-confirm').component({
  state: {
    users: [
      { id: 1, name: 'Alice Johnson' },
      { id: 2, name: 'Bob Smith' },
      { id: 3, name: 'Carol Williams' }
    ],
    confirmingDelete: null
  },
  methods: {
    deleteUser(userId) {
      this.users = this.users.filter(u => u.id !== userId);
      this.confirmingDelete = null;
    }
  }
});
```

```js cleanup
onUnmount(() => {
  $('#demo-inline-confirm').component('destroy');
})
```

**Benefits:**
- No modal overlay needed
- Keeps context visible
- Faster for power users

**When to use:**
- Non-critical actions
- Frequent operations
- List/table actions

## Async Confirmation

Handle async operations after confirmation:

```html live:no-code
<div id="demo-async-confirm" class="en-card">
  <div class="en-card__body">
    <h3>Save Changes</h3>
    <textarea 
      class="en-input" 
      $:model="content" 
      rows="4" 
      placeholder="Enter content..."
      style="width: 100%; resize: vertical;"></textarea>
    
    <div class="en-flex en-gap-2" style="margin-top: 1rem;">
      <button 
        class="en-btn en-btn--primary" 
        $:click.confirm="saveChanges"
        $:attr:disabled="saving"
        data-confirm-message="Save changes to server?">
        <span $:if="!saving">Save</span>
        <span $:if="saving">Saving...</span>
      </button>
      
      <button 
        class="en-btn en-btn--outline" 
        $:click="content = originalContent"
        $:attr:disabled="!hasChanges">
        Discard
      </button>
    </div>
    
    <div class="en-alert en-alert--success" style="margin-top: 1rem;" $:if="saved">
      Changes saved successfully!
    </div>
    
    <div class="en-alert en-alert--danger" style="margin-top: 1rem;" $:if="error">
      <strong>Error:</strong> <span $:text="error"></span>
    </div>
  </div>
</div>
```

```html
<div id="demo-async-confirm">
  <h3>Save Changes</h3>
  <textarea $:model="content" rows="4"></textarea>
  
  <button 
    $:click.confirm="saveChanges"
    $:attr:disabled="saving"
    data-confirm-message="Save changes?">
    <span $:if="!saving">Save</span>
    <span $:if="saving">Saving...</span>
  </button>
  
  <div $:if="saved">Saved!</div>
  <div $:if="error">Error: <span $:text="error"></span></div>
</div>
```

```js inject show
$('#demo-async-confirm').component({
  state: {
    content: 'Hello World',
    originalContent: 'Hello World',
    saving: false,
    saved: false,
    error: null
  },
  computed: {
    hasChanges() {
      return this.content !== this.originalContent;
    }
  },
  methods: {
    async saveChanges() {
      this.saving = true;
      this.error = null;
      this.saved = false;
      
      try {
        // Simulate API call
        await new Promise(resolve => setTimeout(resolve, 1500));
        
        // Simulate random failure
        if (Math.random() < 0.3) {
          throw new Error('Network error');
        }
        
        this.originalContent = this.content;
        this.saved = true;
        
        setTimeout(() => this.saved = false, 3000);
      } catch (err) {
        this.error = err.message;
      } finally {
        this.saving = false;
      }
    }
  }
});
```

```js cleanup
onUnmount(() => {
  $('#demo-async-confirm').component('destroy');
})
```

## Confirmation Timeout

Auto-cancel confirmation after timeout:

```js
mounted() {
  this.$el.on('confirm', (e) => {
    e.preventDefault();
    
    const detail = e.detail;
    const dialog = showCustomDialog();
    
    // Auto-cancel after 10 seconds
    const timeout = setTimeout(() => {
      detail.reject();
      dialog.close();
    }, 10000);
    
    // Clear timeout if user responds
    dialog.onConfirm = () => {
      clearTimeout(timeout);
      detail.accept();
    };
    
    dialog.onCancel = () => {
      clearTimeout(timeout);
      detail.reject();
    };
  });
}
```

## Best Practices

### 1. Use Clear Confirmation Messages

```html
<!-- ✅ Good - Specific and clear -->
<button 
  $:click.confirm="deleteAccount"
  data-confirm-message="Delete your account? All data will be permanently lost.">
  Delete Account
</button>

<!-- ❌ Bad - Vague -->
<button 
  $:click.confirm="deleteAccount"
  data-confirm-message="Are you sure?">
  Delete Account
</button>
```

### 2. Match Button Color to Severity

```html
<!-- ✅ Good - Red for destructive -->
<button class="en-btn en-btn--danger" $:click.confirm="delete">
  Delete
</button>

<!-- ❌ Bad - Primary color for destructive -->
<button class="en-btn en-btn--primary" $:click.confirm="delete">
  Delete
</button>
```

### 3. Don't Overuse Confirmations

```html
<!-- ✅ Good - Confirm destructive action -->
<button $:click.confirm="deleteAccount">Delete Account</button>

<!-- ❌ Bad - Don't confirm routine actions -->
<button $:click.confirm="saveSettings">Save Settings</button>
```

### 4. Provide Undo Alternative

```js
// Instead of confirmation, allow undo
methods: {
  deleteItem(id) {
    const item = this.items.find(i => i.id === id);
    this.items = this.items.filter(i => i.id !== id);
    this.deletedItem = item;
    this.showUndo = true;
    
    // Auto-hide undo after 5 seconds
    clearTimeout(this.undoTimeout);
    this.undoTimeout = setTimeout(() => {
      this.showUndo = false;
    }, 5000);
  },
  undoDelete() {
    if (this.deletedItem) {
      this.items.push(this.deletedItem);
      this.showUndo = false;
    }
  }
}
```

### 5. Use Inline for Frequent Actions

For actions users perform often (like deleting list items), use inline confirmation instead of modal to reduce friction.

## Common Patterns

### Bulk Delete with Confirmation

```js
state: {
  selected: [],
  items: [...]
},
methods: {
  bulkDelete() {
    const count = this.selected.length;
    if (!confirm(`Delete ${count} items? This cannot be undone.`)) {
      return;
    }
    
    this.items = this.items.filter(
      item => !this.selected.includes(item.id)
    );
    this.selected = [];
  }
}
```

### Double Confirmation for Critical Actions

```html
<button 
  $:click="step1"
  $:attr:disabled="!confirmedStep1">
  Step 1: Read Warning
</button>

<div $:if="confirmedStep1">
  <p>Are you absolutely sure? This cannot be undone.</p>
  <button 
    $:click.confirm="finalDelete"
    data-confirm-message="Last chance! Delete everything?">
    Yes, Delete Everything
  </button>
</div>
```

### Type-to-Confirm Pattern

```html
<input $:model="confirmText" placeholder="Type 'DELETE' to confirm">
<button 
  $:click="deleteAccount"
  $:attr:disabled="confirmText !== 'DELETE'">
  Delete Account
</button>
```

## Performance

Confirmations have minimal performance impact:
- Event listeners are efficient
- Modal rendering is fast
- No reactivity overhead

**Tip:** For custom modals, use `<dialog>` element for built-in browser support and accessibility.

## Advanced: `confirmGate()` API

JQR provides `$.jqr.confirmGate()` helper for implementing custom confirmation logic in directives or advanced scenarios.

### What is `confirmGate()`?

`confirmGate()` is a lower-level API designed specifically for **gating actions** (like button clicks) with automatic fallback to `window.confirm()`.

**Key difference from `dispatchConfirm()`:**
- `confirmGate()` - For event handlers/actions (auto-fallback to browser confirm)
- `dispatchConfirm()` - For model value changes (requires listener)

### API Signature

```typescript
interface ConfirmGateOptions {
  message?: string;      // Confirmation message
}

interface ConfirmGateResult {
  accepted: boolean;     // true if user confirmed
}

function confirmGate(
  el: HTMLElement,
  options: ConfirmGateOptions,
  eventName?: string     // Default: 'confirm'
): Promise<ConfirmGateResult>
```

### How It Works

**Without custom listener (default behavior):**

```js
// Automatically falls back to window.confirm()
const { accepted } = await $.jqr.confirmGate(button, {
  message: 'Delete this item?'
});

if (accepted) {
  deleteItem();
}

// User sees browser confirm dialog ✅
```

**With custom listener (custom modal):**

```js
// Add custom listener
document.addEventListener('confirm', (e) => {
  e.preventDefault(); // Prevent fallback to window.confirm()
  
  const detail = e.detail;
  
  // Show custom modal
  showCustomModal({
    message: detail.message,
    onConfirm: () => detail.accept(),
    onCancel: () => detail.reject()
  });
});

// Same confirmGate() call - now uses custom modal!
const { accepted } = await $.jqr.confirmGate(button, {
  message: 'Delete this item?'
});
```

### Event Detail API

When `confirm` event is dispatched:

```typescript
interface ConfirmGateDetail {
  message?: string;      // The confirmation message
  accept: () => void;    // Call to proceed with action
  reject: () => void;    // Call to cancel action
}

// Access in event listener
element.addEventListener('confirm', (e) => {
  const { message, accept, reject } = e.detail;
  
  if (e.defaultPrevented) {
    // Custom handling
    myModal.show(message, accept, reject);
  }
});
```

### Progressive Enhancement Pattern

Start with browser confirm, upgrade to custom modal later:

```html
<!-- Step 1: Works immediately with browser confirm -->
<button $:click.confirm="deleteItem" data-confirm-message="Delete?">
  Delete
</button>
```

```js
// Step 2: Add custom modal (no HTML changes needed!)
$('#app').component({
  mounted() {
    this.$el.on('confirm', (e) => {
      e.preventDefault();
      
      showFancyModal({
        message: e.detail.message,
        onOk: () => e.detail.accept(),
        onCancel: () => e.detail.reject()
      });
    });
  }
});
```

### Use in Custom Directives

Create directives that leverage `confirmGate()`:

```html live
<button data-confirm="delete" class="en-btn en-btn--primary"
  $:delete-confirm="deleteItem(item.id)"
  data-confirm-message="Delete this item?">
  Delete
</button>
```

```js inject show
// Custom delete directive with built-in confirmation
JQR.directive('delete-confirm', {
  bind(el, { expr, evaluate, instance }) {
    el.addEventListener('click', async () => {
      const message = el.dataset.confirmMessage || 'Delete this item?';      
      const { accepted } = await $.jqr.confirmGate(el, message);
      
      if (accepted) {
        evaluate(expr); // Execute the handler
      }
    });
  }
});

$('[data-confirm="delete"]').component();
```

```html live
<button id="raw-delete-btn" class="en-btn en-btn--danger">Delete (raw JS)</button>

<dialog id="raw-confirm-dialog" style="position: fixed; margin: auto;">
  <p data-confirm-message>Are you sure?</p>
  <button data-confirm-ok class="en-btn en-btn--primary">OK</button>
  <button data-confirm-cancel class="en-btn en-btn--primary en-btn--outline">Cancel</button>
</dialog>
```

```js inject show
$('#raw-delete-btn').on('click confirm', async (e) => {
  const $dialog = $('#raw-confirm-dialog');
  const dialog = $dialog.get(0);
  const $messageEl = $dialog.find('[data-confirm-message]');
  const $okBtn = $dialog.find('[data-confirm-ok]');
  const $cancelBtn = $dialog.find('[data-confirm-cancel]');
  
  if (e.type === 'click') {
    const ok = await JQR.confirmGate(e.currentTarget, $messageEl.text() || 'Delete this item permanently?');
    if (!ok) { console.log('Canceled!'); return;} 
    console.log('Item deleted');
  }

  if (e.type === 'confirm') {
    e.preventDefault();
    const { message, accept, reject } = e.detail;

    $messageEl.text(message);
    dialog.showModal();

    const handleOk = () => { accept(); dialog.close(); cleanup(); };
    const handleCancel = () => { reject(); dialog.close(); cleanup(); };
    const cleanup = () => {
      $okBtn.off('click', handleOk);
      $cancelBtn.off('click', handleCancel);
    };
    $okBtn.on('click', handleOk);
    $cancelBtn.on('click', handleCancel);
  }
});
```

### Comparison Table

| Feature | `confirmGate()` | `dispatchConfirm()` |
|---------|-----------------|---------------------|
| **Purpose** | Gate actions (clicks) | Value changes (models) |
| **Fallback** | Auto `window.confirm()` | No fallback |
| **Use Case** | Event handlers | `$:model.confirm` |
| **Detail** | `{ message, accept, reject }` | `{ oldValue, newValue, accept, reject }` |
| **Best For** | Button clicks, deletions | Form input changes |

### When to Use Each

**Use `confirmGate()`:**
```js
// Confirming actions
button.addEventListener('click', async () => {
  const { accepted } = await $.jqr.confirmGate(button, {
    message: 'Delete account?'
  });
  if (accepted) deleteAccount();
});
```

**Use `dispatchConfirm()`:**
```js
// Confirming value changes (internal use in $:model.confirm)
const { accepted } = await $.jqr.dispatchConfirm(input, {
  oldValue: currentValue,
  newValue: newValue
});
if (accepted) applyChange(newValue);
```

### Benefits of `confirmGate()`

1. **Zero Config Required**
   - Works immediately with browser confirm
   - No setup needed for basic usage

2. **Progressive Enhancement**
   - Start simple, enhance later
   - No code changes when upgrading

3. **Consistent API**
   - Same event pattern as `dispatchConfirm()`
   - Familiar `accept()`/`reject()` methods

4. **Flexible Integration**
   - Works with any modal library
   - Custom styling and animations

### Example: Toast-based Confirmation

```js
$('#app').component({
  mounted() {
    this.$el.on('confirm', (e) => {
      e.preventDefault();
      
      const { message, accept, reject } = e.detail;
      
      // Show toast notification with actions
      showToast({
        message,
        actions: [
          { label: 'Confirm', onClick: accept },
          { label: 'Cancel', onClick: reject }
        ],
        duration: 5000,
        onTimeout: reject // Auto-cancel after timeout
      });
    });
  }
});
```

## Next Steps

- [Model Modifiers](./model-modifiers.md) - All model modifiers including `.confirm`
- [Events](./events.md) - Event handling patterns
- [Teleport](./teleport.md) - Modal rendering patterns
- [Slots](./slots.md) - Reusable modal components
