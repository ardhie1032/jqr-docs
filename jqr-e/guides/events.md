<!-- {
  "path": "jqr-e/guides/events",
  "name": "docs-jqr-e-guides-events",
  "meta": {
    "title": "Events",
    "order": 3
  }
} -->

# Events

Learn how custom elements emit and handle events using native DOM APIs alongside JQR's event system.

## Overview

When a custom element emits an event with `$emit()`, JQR-E dispatches **two events**:

1. **jQuery Event** (`jqr:<name>`) - For JQR components using `$:on:*`
2. **Native CustomEvent** (`<name>`) - For vanilla JavaScript using `addEventListener()`

This dual-dispatch ensures custom elements work seamlessly in both JQR components and plain HTML.

## Emitting Events

### Basic Emit

Use `$emit()` inside component methods:

```js
JQRE.defineElement('click-counter', {
  state: { count: 0 },
  emits: ['increment'],  // Document emitted events
  methods: {
    handleClick() {
      this.state.count++;
      this.$emit('increment', this.state.count);
    }
  },
  template: `
    <button $:click="handleClick()">
      Clicked <span $:text="count"></span> times
    </button>
  `
});
```

**Signature:**
```js
this.$emit(eventName, payload);
```

- `eventName` (string) - Event name (kebab-case recommended)
- `payload` (any) - Data to pass with event

### Live Example

```html live:no-code
<div id="demo-events-basic">
  <div class="en-card en-card__body">
    <event-button></event-button>
    <div id="demo-events-output" class="en-alert en-alert--info" style="margin-top: 10px">
      <span>Waiting for click...</span>
    </div>
  </div>
</div>
```

```html
<div id="demo-events-basic">
  <event-button></event-button>
  <div id="output">Waiting for click...</div>
</div>
```

```js inject show
JQRE.defineElement('event-button', {
  state: { clicks: 0 },
  emits: ['click-event'],
  methods: {
    handleClick() {
      this.clicks++;
      this.$emit('click-event', { 
        count: this.clicks,
        timestamp: Date.now()
      });
    }
  },
  template: `
    <button class="en-btn en-btn--primary" $:click="handleClick()">
      Click Me (<span $:text="clicks"></span>)
    </button>
  `
});

// Listen with native addEventListener
const btn = document.querySelector('#demo-events-basic event-button');
btn.addEventListener('click-event', (e) => {
  const output = document.getElementById('demo-events-output');
  output.innerHTML = `
    <strong>Event received!</strong><br>
    Count: ${e.detail.count}<br>
    Time: ${new Date(e.detail.timestamp).toLocaleTimeString()}
  `;
});
```

```js cleanup
onUnmount(() => {
  $('#demo-events-basic').component('destroy');
});
```

## Listening to Events

### Native addEventListener()

Use standard DOM API for standalone elements:

```js
const element = document.querySelector('my-button');

// Listen to custom event
element.addEventListener('press', (event) => {
  console.log('Button pressed!');
  console.log('Payload:', event.detail);
});

// Remove listener
element.removeEventListener('press', handler);
```

**Event Object:**
```js
CustomEvent {
  type: 'press',           // Event name
  detail: payload,         // Your payload
  bubbles: true,           // Events bubble up DOM
  composed: true,          // Cross shadow boundaries
  target: element,         // The custom element
  currentTarget: element   // Current listener element
}
```

### Inside JQR Components

Use `$:on:*` directive for reactive event handling:

```html live:no-code
<div id="demo-events-jqr">
  <div class="en-card en-card__body">
    <div $:component:state="{ message: 'Waiting...' }">
      <p class="en-form-display" $:text="message"></p>
      <notify-button $:on:notify="message = 'Button clicked at ' + new Date().toLocaleTimeString()"></notify-button>
    </div>
  </div>
</div>
```

```html
<div id="demo-events-jqr">
  <div $:component:state="{ message: 'Waiting...' }">
    <p $:text="message"></p>
    <notify-button $:on:notify="message = 'Button clicked at ' + new Date().toLocaleTimeString()"></notify-button>
  </div>
</div>
```

```js inject show
JQRE.defineElement('notify-button', {
  emits: ['notify'],
  methods: {
    sendNotification() {
      this.$emit('notify', { source: 'button' });
    }
  },
  template: `
    <button class="en-btn" $:click="sendNotification()">
      Send Notification
    </button>
  `
});
```

```js cleanup
onUnmount(() => {
  $('#demo-events-jqr').component('destroy');
});
```

### With jQuery

For jQuery users, use `.on()` with `jqr:` prefix:

```js
$('my-button').on('jqr:press', function(event, payload) {
  console.log('Pressed:', payload);
});
```

**Note:** Native event name doesn't have `jqr:` prefix, jQuery event does.

## Event Naming

### Recommended Convention

Use **action verbs** instead of DOM event names:

**✅ Good Names:**
- `press` (instead of `click`)
- `submit` (instead of `submit` - but okay for forms)
- `open`, `close`
- `expand`, `collapse`
- `select`, `deselect`
- `change` (generic state change)

**❌ Avoid:**
- `click` (conflicts with native event)
- `input` (conflicts with native event)
- `change` (conflicts with native event for inputs)
- `focus`, `blur` (conflicts with native events)

### Kebab-Case

Use kebab-case for multi-word events:

```js
this.$emit('user-selected', user);
this.$emit('item-added', item);
this.$emit('dialog-closed', result);
```

### Namespacing

For component libraries, consider prefixing:

```js
// Library: @company/ui-components
this.$emit('ui:select', item);
this.$emit('ui:change', value);
```

## Event Payloads

### Simple Values

```js
// Number
this.$emit('increment', 1);

// String
this.$emit('message', 'Hello!');

// Boolean
this.$emit('toggle', true);
```

### Objects

Pass structured data:

```js
this.$emit('user-selected', {
  id: 123,
  name: 'Alice',
  role: 'admin'
});
```

### Arrays

```js
this.$emit('items-changed', [
  { id: 1, name: 'Item 1' },
  { id: 2, name: 'Item 2' }
]);
```

### Multiple Values

Wrap in object (recommended):

```js
// ✅ Good: Single object
this.$emit('drag-end', {
  x: 100,
  y: 200,
  element: el
});

// ❌ Avoid: Multiple arguments (only first is used)
this.$emit('drag-end', 100, 200, el);  // Only 100 is passed!
```

## Two-Way Binding

Use `update:<prop>` convention for v-model style binding:

### Defining Component

```html live:no-code
<div id="demo-events-two-way">
  <div class="en-card en-card__body">
    <div $:component:state="{ count: 10 }">
      <p>Parent count: <strong $:text="count"></strong></p>
      <stepper-input $:model:value="count"></stepper-input>
    </div>
  </div>
</div>
```

```html
<div id="demo-events-two-way">
  <div $:component:state="{ count: 10 }">
    <p>Parent count: <strong $:text="count"></strong></p>
    <stepper-input $:model:value="count"></stepper-input>
  </div>
</div>
```

```js inject show
JQRE.defineElement('stepper-input', {
  props: {
    value: 0
  },
  emits: ['update:value'],
  methods: {
    increment() {
      this.$emit('update:value', this.$props.value + 1);
    },
    decrement() {
      this.$emit('update:value', this.$props.value - 1);
    }
  },
  template: `
    <div class="en-flex">
      <button class="en-btn en-btn--sm" $:click="decrement()">-</button>
      <span class="en-form-display en-form-display--sm" $:text="$props.value"></span>
      <button class="en-btn en-btn--sm" $:click="increment()">+</button>
    </div>
  `
});
```

```js cleanup
onUnmount(() => {
  $('#demo-events-two-way').component('destroy');
});
```

**Pattern:**
1. Prop: `value`
2. Event: `update:value`
3. Payload: new value

This enables `$:model:value` shorthand in parent.

## Event Bubbling

Native CustomEvents **bubble by default**:

```html live:no-code
<div id="demo-events-bubbling">
  <div class="en-card">
    <div id="demo-events-container" style="padding: 10px; border: 2px dashed #ccc">
      <p>Container (click inside card)</p>
      <bubble-card></bubble-card>
    </div>
    <div id="demo-events-bubble-log" class="en-alert en-alert--info" style="margin-top: 10px">
      <span>Click the card below...</span>
    </div>
  </div>
</div>
```

```html
<div id="demo-events-bubbling">
  <div id="container">
    <bubble-card></bubble-card>
  </div>
  <div id="log"></div>
</div>
```

```js inject show
JQRE.defineElement('bubble-card', {
  emits: ['card-click'],
  methods: {
    handleClick() {
      this.$emit('card-click', { message: 'Card clicked!' });
    }
  },
  template: `
    <div class="en-card" style="cursor: pointer; padding: 20px" $:click="handleClick()">
      <p>Click me!</p>
    </div>
  `
});

// Listen on parent container
const container = document.getElementById('demo-events-container');
container.addEventListener('card-click', (e) => {
  const log = document.getElementById('demo-events-bubble-log');
  log.innerHTML = `
    <strong>Event bubbled to container!</strong><br>
    Target: ${e.target.tagName.toLowerCase()}<br>
    Message: ${e.detail.message}
  `;
});
```

```js cleanup
onUnmount(() => {
  $('#demo-events-bubbling').component('destroy');
});
```

**Stop Bubbling:**

```js
element.addEventListener('my-event', (e) => {
  e.stopPropagation();  // Stop bubbling
});
```

## Event Modifiers

Use event modifiers in JQR components:

```html
<my-button $:on:press.prevent="handlePress()"></my-button>
<my-button $:on:press.stop="handlePress()"></my-button>
<my-button $:on:press.once="handlePress()"></my-button>
```

See [Model Modifiers](../../jqr/guides/model-modifiers.md) for full list.

## Documenting Events

### emits Array

Document all events your component emits:

```js
JQRE.defineElement('file-uploader', {
  emits: [
    'upload-start',
    'upload-progress',
    'upload-complete',
    'upload-error'
  ],
  methods: {
    startUpload() {
      this.$emit('upload-start', { filename: 'file.txt' });
      // ...
    }
  }
});
```

**Benefits:**
- Self-documentation
- Dev warnings for typos
- Better IDE support

### JSDoc Comments

For TypeScript/JSDoc users:

```js
/**
 * File uploader component
 * @fires upload-start - Emitted when upload begins
 * @fires upload-progress - Emitted during upload (payload: {percent})
 * @fires upload-complete - Emitted on success (payload: {url})
 * @fires upload-error - Emitted on failure (payload: {error})
 */
JQRE.defineElement('file-uploader', definition);
```

## Complete Example: Dialog

```html live:no-code
<div id="demo-events-dialog">
  <div class="en-card">
    <button class="en-btn en-btn--primary" id="demo-events-open-btn">
      Open Dialog
    </button>
    <simple-dialog id="demo-events-my-dialog">
      <h3>Confirm Action</h3>
      <p>Are you sure you want to continue?</p>
    </simple-dialog>
    <div id="demo-events-dialog-result" class="en-alert en-alert--info" style="margin-top: 10px">
      <span>Click "Open Dialog" to start...</span>
    </div>
  </div>
</div>
```

```html
<div id="demo-events-dialog">
  <button id="open-btn">Open Dialog</button>
  
  <simple-dialog id="my-dialog">
    <h3>Confirm Action</h3>
    <p>Are you sure you want to continue?</p>
  </simple-dialog>
  
  <div id="result"></div>
</div>
```

```js inject show
JQRE.defineElement('simple-dialog', {
  state: {
    open: false
  },
  emits: ['open', 'close', 'confirm', 'cancel'],
  methods: {
    openDialog() {
      this.state.open = true;
      this.$emit('open');
    },
    closeDialog() {
      this.state.open = false;
      this.$emit('close');
    },
    handleConfirm() {
      this.$emit('confirm', { timestamp: Date.now() });
      this.closeDialog();
    },
    handleCancel() {
      this.$emit('cancel');
      this.closeDialog();
    }
  },
  template: `
    <div $:if="open" style="position: fixed; inset: 0; background: rgba(0,0,0,0.5); display: flex; align-items: center; justify-content: center; z-index: 1000">
      <div class="en-card" style="min-width: 400px; max-width: 500px">
        <slot></slot>
        <div style="display: flex; gap: 8px; margin-top: 20px; justify-content: flex-end">
          <button class="en-btn" $:click="handleCancel()">Cancel</button>
          <button class="en-btn en-btn--primary" $:click="handleConfirm()">Confirm</button>
        </div>
      </div>
    </div>
  `
});

// Setup event listeners
const openBtn = document.getElementById('demo-events-open-btn');
const dialog = document.getElementById('demo-events-my-dialog');
const result = document.getElementById('demo-events-dialog-result');

openBtn.addEventListener('click', () => {
  dialog.openDialog();
});

dialog.addEventListener('confirm', (e) => {
  result.innerHTML = `<strong>✅ Confirmed!</strong><br>Time: ${new Date(e.detail.timestamp).toLocaleTimeString()}`;
  result.className = 'en-alert en-alert--success';
});

dialog.addEventListener('cancel', () => {
  result.innerHTML = '<strong>❌ Cancelled</strong>';
  result.className = 'en-alert en-alert--warning';
});

dialog.addEventListener('open', () => {
  result.innerHTML = '<strong>Dialog opened...</strong>';
  result.className = 'en-alert en-alert--info';
});

dialog.addEventListener('close', () => {
  if (result.innerHTML.includes('opened')) {
    result.innerHTML = '<strong>Dialog closed (no action)</strong>';
    result.className = 'en-alert en-alert--info';
  }
});
```

```js cleanup
onUnmount(() => {
  $('#demo-events-dialog').component('destroy');
});
```

## Event vs Method

When to use events vs exposed methods:

### Use Events When:
- ✅ Notifying parent of state changes
- ✅ User interactions (clicks, selections)
- ✅ Async operations completing
- ✅ Two-way binding (`update:*`)

### Use Methods When:
- ✅ Parent controlling child behavior
- ✅ Imperative actions (open, close, focus)
- ✅ Getting internal state
- ✅ Synchronous operations

**Example:**

```js
JQRE.defineElement('media-player', {
  emits: ['play', 'pause', 'end'],  // Events: notify parent
  methods: {
    play() { /* ... */ },           // Methods: control from parent
    pause() { /* ... */ },
    seek(time) { /* ... */ }
  }
}, {
  expose: ['play', 'pause', 'seek']  // Expose methods
});
```

```js
const player = document.querySelector('media-player');

// Call methods (imperative)
player.play();
player.seek(30);

// Listen to events (reactive)
player.addEventListener('end', () => {
  console.log('Playback ended');
});
```

## Best Practices

### ✅ Do

- **Document all events** in `emits` array
- **Use descriptive names** that indicate action
- **Keep payloads simple** and serializable
- **Follow conventions** (`update:*` for two-way binding)
- **Test event bubbling** if relying on it
- **Namespace events** in component libraries

### ❌ Don't

- **Don't emit on every state change** (performance)
- **Don't use native event names** (click, input, change)
- **Don't emit events synchronously in constructor** (element not ready)
- **Don't pass DOM elements** in payload (use IDs/refs)
- **Don't rely on event order** (async nature)

## Common Patterns

### Progress Events

```js
JQRE.defineElement('async-task', {
  emits: ['start', 'progress', 'complete', 'error'],
  methods: {
    async runTask() {
      this.$emit('start');
      
      try {
        for (let i = 0; i <= 100; i += 10) {
          await new Promise(r => setTimeout(r, 100));
          this.$emit('progress', { percent: i });
        }
        this.$emit('complete', { success: true });
      } catch (error) {
        this.$emit('error', { error: error.message });
      }
    }
  }
});
```

### Form Events

```js
JQRE.defineElement('custom-form', {
  emits: ['submit', 'reset', 'validate'],
  methods: {
    handleSubmit() {
      const isValid = this.validate();
      this.$emit('validate', { valid: isValid });
      
      if (isValid) {
        this.$emit('submit', this.getFormData());
      }
    }
  }
});
```

### Selection Events

```js
JQRE.defineElement('item-list', {
  emits: ['select', 'deselect', 'select-all', 'clear-selection'],
  methods: {
    selectItem(item) {
      this.$emit('select', { item, selectedItems: this.selected });
    }
  }
});
```

## Debugging Events

### Log All Events

```js
const element = document.querySelector('my-element');

// Log all custom events
['event1', 'event2', 'event3'].forEach(eventName => {
  element.addEventListener(eventName, (e) => {
    console.log(`[${eventName}]`, e.detail);
  });
});
```

### Event Inspector

```js
function inspectEvents(element, events) {
  events.forEach(eventName => {
    element.addEventListener(eventName, (e) => {
      console.group(`Event: ${eventName}`);
      console.log('Target:', e.target);
      console.log('Detail:', e.detail);
      console.log('Time:', new Date().toISOString());
      console.groupEnd();
    });
  });
}

inspectEvents(myElement, ['open', 'close', 'change']);
```

## TypeScript Support

Type-safe event handling:

```ts
import { defineElement } from '@en32/jqr/element';

interface ButtonEvents {
  press: { button: 'left' | 'right' };
  'long-press': { duration: number };
}

// Define with types
defineElement('typed-button', {
  emits: ['press', 'long-press'],
  methods: {
    handlePress() {
      this.$emit('press', { button: 'left' });
    }
  }
});

// Type-safe listening
const btn = document.querySelector('typed-button')!;
btn.addEventListener('press', (e: CustomEvent<ButtonEvents['press']>) => {
  console.log(e.detail.button); // ✅ Typed
});
```

## Next Steps

- **[Slots](./slots.md)** - Content projection with slots
- **[Element API](./element-api.md)** - Access instance and call methods
- **[Lifecycle](./lifecycle.md)** - Understanding mounting and ownership
- **[Examples](../examples/button-component.md)** - Complete component examples

## Related

- [JQR Events Guide](../../jqr/guides/events.md)
- [Attributes & Props](./attributes-and-props.md)
- [Defining Elements](./defining-elements.md)
