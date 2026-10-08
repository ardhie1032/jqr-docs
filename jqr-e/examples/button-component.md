<!-- {
  "path": "jqr-e/examples/button-component",
  "name": "docs-jqr-e-examples-button-component",
  "meta": {
    "title": "Button Component",
    "order": 1
  }
} -->

# Example: Button Component

Build a reusable button component with variants, loading states, and disabled handling.

## Features

- ✅ Multiple variants (default, primary, danger, success)
- ✅ Loading state with spinner
- ✅ Disabled state
- ✅ Size variants (sm, md, lg)
- ✅ Icon support
- ✅ Click event with confirmation
- ✅ Attribute reflection for CSS styling

## Component Definition

```js
JQRE.defineElement('app-button', {
  props: {
    variant: 'default',
    size: 'md',
    disabled: false,
    loading: false,
    icon: null,
    confirm: false,
    confirmMessage: 'Are you sure?'
  },
  
  computed: {
    classes() {
      const base = 'en-btn';
      const variant = `en-btn--${this.$props.variant}`;
      const size = this.$props.size !== 'md' ? `en-btn--${this.$props.size}` : '';
      return [base, variant, size].filter(Boolean).join(' ');
    },
    
    isDisabled() {
      return this.$props.disabled || this.$props.loading;
    }
  },
  
  methods: {
    async handleClick(event) {
      if (this.isDisabled) {
        event.preventDefault();
        return;
      }
      
      if (this.$props.confirm) {
        const confirmed = confirm(this.$props.confirmMessage);
        if (!confirmed) {
          event.preventDefault();
          return;
        }
      }
      
      this.$emit('click', { button: this });
    }
  },
  
  template: `
    <button 
      $:class="classes"
      $:attr:disabled="isDisabled"
      $:click="handleClick">
      
      <span $:if="loading" class="spinner">⏳</span>
      <span $:if="icon && !loading" $:text="icon"></span>
      <slot></slot>
    </button>
  `
}, {
  attributes: {
    disabled: Boolean,
    loading: Boolean,
    confirm: Boolean
  },
  reflect: ['variant', 'size', 'disabled', 'loading'],
  expose: ['handleClick'],
  display: 'inline-block'
});
```

## Live Demo

```html live:no-code
<div id="demo-button-component" class="en-card en-card__body">
  <div class="en-mb-4">
    <h4 class="en-mb-2">Variants</h4>
    <div class="en-flex en-gap-2 en-flex-wrap">
      <app-button>Default</app-button>
      <app-button variant="primary">Primary</app-button>
      <app-button variant="success">Success</app-button>
      <app-button variant="danger">Danger</app-button>
    </div>
  </div>

  <div class="en-mb-4">
    <h4 class="en-mb-2">Sizes</h4>
    <div class="en-flex en-gap-2 en-items-center en-flex-wrap">
      <app-button size="sm" variant="primary">Small</app-button>
      <app-button size="md" variant="primary">Medium</app-button>
      <app-button size="lg" variant="primary">Large</app-button>
    </div>
  </div>

  <div class="en-mb-4">
    <h4 class="en-mb-2">States</h4>
    <div class="en-flex en-gap-2 en-flex-wrap">
      <app-button variant="primary" disabled>Disabled</app-button>
      <app-button variant="primary" loading>Loading</app-button>
    </div>
  </div>

  <div class="en-mb-4">
    <h4 class="en-mb-2">With Icons</h4>
    <div class="en-flex en-gap-2 en-flex-wrap">
      <app-button variant="primary" icon="📁">Open</app-button>
      <app-button variant="success" icon="💾">Save</app-button>
      <app-button variant="danger" icon="🗑️">Delete</app-button>
    </div>
  </div>

  <div class="en-mb-4">
    <h4 class="en-mb-2">With Confirmation</h4>
    <div class="en-flex en-gap-2 en-flex-wrap">
      <app-button variant="danger" confirm confirm-message="Delete this item?">
        Delete with Confirm
      </app-button>
    </div>
  </div>

  <div class="en-mb-4">
    <h4 class="en-mb-2">Interactive Demo</h4>
    <div class="en-flex en-gap-2 en-flex-wrap en-mb-2">
      <app-button id="demo-btn-interactive" variant="primary">
        Click Me (Count: <span id="demo-btn-count">0</span>)
      </app-button>
      <button class="en-btn" onclick="document.querySelector('#demo-btn-interactive').loading = true; setTimeout(() => document.querySelector('#demo-btn-interactive').loading = false, 2000)">
        Simulate Loading
      </button>
      <button class="en-btn" onclick="document.querySelector('#demo-btn-interactive').disabled = !document.querySelector('#demo-btn-interactive').disabled">
        Toggle Disabled
      </button>
    </div>
    <div class="en-alert en-alert--info" id="demo-btn-message" style="display: none"></div>
  </div>
</div>
```

```html
<div id="demo-button-component">
  <!-- Variants -->
  <app-button>Default</app-button>
  <app-button variant="primary">Primary</app-button>
  <app-button variant="success">Success</app-button>
  <app-button variant="danger">Danger</app-button>

  <!-- Sizes -->
  <app-button size="sm">Small</app-button>
  <app-button size="md">Medium</app-button>
  <app-button size="lg">Large</app-button>

  <!-- States -->
  <app-button disabled>Disabled</app-button>
  <app-button loading>Loading</app-button>

  <!-- With Icons -->
  <app-button icon="📁">Open</app-button>
  <app-button icon="💾">Save</app-button>
  <app-button icon="🗑️">Delete</app-button>

  <!-- With Confirmation -->
  <app-button confirm confirm-message="Are you sure?">
    Delete
  </app-button>
</div>
```

```js inject show
// Define the button component
JQRE.defineElement('app-button', {
  props: {
    variant: 'default',
    size: 'md',
    disabled: false,
    loading: false,
    icon: null,
    confirm: false,
    confirmMessage: 'Are you sure?'
  },
  
  computed: {
    classes() {
      const base = 'en-btn';
      const variant = `en-btn--${this.$props.variant}`;
      const size = this.$props.size !== 'md' ? `en-btn--${this.$props.size}` : '';
      return [base, variant, size].filter(Boolean).join(' ');
    },
    
    isDisabled() {
      return this.$props.disabled || this.$props.loading;
    }
  },
  
  methods: {
    async handleClick(event) {
      if (this.isDisabled) {
        event.preventDefault();
        return;
      }
      
      if (this.$props.confirm) {
        // Same zero-setup gate `$:click.confirm` uses under the hood
        // (falls back to window.confirm(message), but stays overridable
        // via a `$:on:confirm` listener anywhere up the tree) — called
        // directly as an instance method, rather than the `.confirm`
        // modifier itself, since whether to gate at all depends on the
        // `confirm` prop.
        const confirmed = await this.$confirmGate(this.$props.confirmMessage);
        if (!confirmed) {
          event.preventDefault();
          return;
        }
      }
      
      this.$emit('click', { button: this });
    }
  },
  
  template: `
    <button 
      $:class="classes"
      $:attr:disabled="isDisabled"
      $:click="handleClick">
      
      <span $:if="$props.loading">⏳ </span>
      <span $:if="$props.icon && !$props.loading" $:text="$props.icon + ' '"></span>
      <slot $:default></slot>
    </button>
  `
}, {
  attributes: {
    disabled: Boolean,
    loading: Boolean,
    confirm: Boolean
  },
  reflect: ['variant', 'size', 'disabled', 'loading'],
  expose: ['handleClick'],
  display: 'inline-block'
});

// Interactive demo setup — wired declaratively via $:ref/$:click/$:on:click
// on the `#demo-btn-interactive-root` above, instead of querySelector +
// addEventListener.
$('#demo-btn-interactive-root').component({
  state: {
    clickCount: 0
  },
  methods: {
    handleInteractiveClick() {
      this.clickCount++;
    },
    simulateLoading() {
      this.$refs.interactiveBtn.prop('loading', true);
      setTimeout(() => this.$refs.interactiveBtn.prop('loading', false), 2000);
    },
    toggleDisabled() {
      const el = this.$refs.interactiveBtn[0];
      el.disabled = !el.disabled;
    }
  }
});
```

```js cleanup
onUnmount(() => {
  $('#demo-button-component').component('destroy');
});
```

## Usage Examples

### Basic Usage

```html
<app-button>Click Me</app-button>
<app-button variant="primary">Primary Action</app-button>
<app-button variant="danger">Delete</app-button>
```

### With Props

```html
<app-button 
  variant="primary" 
  size="lg" 
  disabled>
  Large Disabled Button
</app-button>

<app-button 
  variant="success" 
  loading>
  Saving...
</app-button>
```

### With Icons

```html
<app-button variant="primary" icon="📁">Open File</app-button>
<app-button variant="success" icon="✓">Confirm</app-button>
<app-button variant="danger" icon="×">Cancel</app-button>
```

### With Confirmation

```html
<app-button 
  variant="danger" 
  confirm 
  confirm-message="Delete this item permanently?">
  Delete
</app-button>
```

### Listening to Events

```html
<app-button id="my-button" variant="primary">
  Click Me
</app-button>

<script>
  const button = document.querySelector('#my-button');
  
  button.addEventListener('jqr:click', (event) => {
    console.log('Button clicked!', event.detail);
  });
</script>
```

### Programmatic Control

```js
const button = document.querySelector('app-button');

// Show loading state
button.loading = true;

// Simulate async operation
setTimeout(() => {
  button.loading = false;
}, 2000);

// Disable button
button.disabled = true;

// Change variant
button.variant = 'success';
```

### With JavaScript API

```js
const button = document.querySelector('app-button');

// Wait for mount
await button.$mounted;

// Access instance
console.log(button.$instance.$props.variant);

// Programmatically trigger click
button.handleClick(new Event('click'));

// Update props
button.$instance.$setProps({
  variant: 'primary',
  loading: true
});
```

## CSS Styling

The button reflects its state to attributes for CSS styling:

```css
/* Style by variant */
app-button[variant="primary"] {
  /* Additional styles */
}

/* Style by state */
app-button[disabled] {
  cursor: not-allowed;
}

app-button[loading] {
  position: relative;
}

/* Style by size */
app-button[size="lg"] {
  font-size: 1.2em;
}
```

## Advanced Patterns

### Loading with Async Operation

```html
<app-button id="save-button" variant="primary">
  Save
</app-button>

<script>
  const saveBtn = document.querySelector('#save-button');
  
  saveBtn.addEventListener('jqr:click', async () => {
    saveBtn.loading = true;
    
    try {
      await fetch('/api/save', { method: 'POST' });
      saveBtn.variant = 'success';
      saveBtn.textContent = 'Saved!';
      
      setTimeout(() => {
        saveBtn.variant = 'primary';
        saveBtn.textContent = 'Save';
      }, 2000);
    } catch (error) {
      saveBtn.variant = 'danger';
      saveBtn.textContent = 'Error!';
    } finally {
      saveBtn.loading = false;
    }
  });
</script>
```

### Form Submit Button

```html
<form id="user-form">
  <input name="username" required>
  <input name="email" type="email" required>
  
  <app-button 
    id="submit-btn" 
    variant="primary"
    type="submit">
    Submit
  </app-button>
</form>

<script>
  const form = document.querySelector('#user-form');
  const submitBtn = document.querySelector('#submit-btn');
  
  form.addEventListener('submit', async (e) => {
    e.preventDefault();
    submitBtn.loading = true;
    
    const formData = new FormData(form);
    
    try {
      await fetch('/api/users', {
        method: 'POST',
        body: formData
      });
      alert('Success!');
    } catch (error) {
      alert('Error!');
    } finally {
      submitBtn.loading = false;
    }
  });
</script>
```

### Button Group

```html
<div class="button-group">
  <app-button variant="primary">Left</app-button>
  <app-button variant="primary">Middle</app-button>
  <app-button variant="primary">Right</app-button>
</div>

<style>
.button-group {
  display: inline-flex;
}

.button-group app-button:not(:first-child) button {
  border-left: none;
  border-top-left-radius: 0;
  border-bottom-left-radius: 0;
}

.button-group app-button:not(:last-child) button {
  border-top-right-radius: 0;
  border-bottom-right-radius: 0;
}
</style>
```

## Customization

### Extending the Component

```js
JQRE.defineElement('icon-button', {
  props: {
    variant: 'default',
    icon: '🔹'
  },
  template: `
    <app-button $:bind="{ variant }">
      <span $:text="icon"></span>
      <slot></slot>
    </app-button>
  `
});
```

### Custom Variants

Add custom variants via CSS:

```css
app-button[variant="custom"] button {
  background: linear-gradient(45deg, #667eea 0%, #764ba2 100%);
  color: white;
  border: none;
}
```

```html
<app-button variant="custom">Custom Style</app-button>
```

## Key Features

### Props

| Prop | Type | Default | Description |
|------|------|---------|-------------|
| `variant` | String | `'default'` | Button style variant |
| `size` | String | `'md'` | Button size |
| `disabled` | Boolean | `false` | Disable button |
| `loading` | Boolean | `false` | Show loading state |
| `icon` | String | `null` | Icon text/emoji |
| `confirm` | Boolean | `false` | Show confirmation dialog |
| `confirmMessage` | String | `'Are you sure?'` | Confirmation message |

### Events

| Event | Detail | Description |
|-------|--------|-------------|
| `jqr:click` | `{ button: Element }` | Emitted on click (after confirmation if enabled) |

### Methods

| Method | Description |
|--------|-------------|
| `handleClick()` | Programmatically trigger click |

### CSS Classes

Uses EN-CSS button classes:
- `en-btn` - Base button
- `en-btn--{variant}` - Variant styles
- `en-btn--{size}` - Size styles

## Best Practices

### ✅ Do

- **Use semantic variants** - Choose variant based on action meaning
- **Show loading states** - Provide feedback during async operations
- **Disable during loading** - Prevent multiple submissions
- **Use confirmation** - For destructive actions
- **Provide icons** - Enhance visual communication

### ❌ Don't

- **Don't nest buttons** - Invalid HTML structure
- **Don't forget events** - Listen to `jqr:click`, not native `click`
- **Don't skip loading states** - Always show progress
- **Don't overuse confirm** - Only for destructive/important actions

## Browser Compatibility

Works in all browsers supporting Custom Elements:
- Chrome 67+
- Firefox 63+
- Safari 10.1+
- Edge 79+

## See Also

- [Defining Elements](../guides/defining-elements.md)
- [Element Options](../guides/options.md)
- [Events Guide](../guides/events.md)
- [Dialog Component Example](./dialog-component.md)
