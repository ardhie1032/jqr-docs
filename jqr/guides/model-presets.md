<!-- {
  "path": "jqr/guides/model-presets",
  "name": "docs-jqr-guides-model-presets",
  "meta": {
    "title": "Model Presets",
    "order": 20
  }
} -->

# Model Presets

Model presets define how `$:model:<prop>` binds to native, non-form elements like `<dialog>` and `<details>`. This guide covers using and creating custom presets.

## What are Model Presets?

### The Problem

Form controls (`<input>`, `<select>`, `<textarea>`) have standard events (`input`, `change`) for two-way binding:

```html
<!-- ✅ Works: standard form control -->
<input $:model="username" />
<!-- Listens to 'input' event -->
```

But **non-form elements** have different properties and events:

```html
<!-- ❌ Won't work: <dialog> has .open property, not .value -->
<dialog $:model="isOpen">...</dialog>

<!-- ❌ What event should we listen to? -->
<details $:model="expanded">...</details>
```

### The Solution

Model presets tell JQR:
1. Which **DOM property** to bind
2. Which **event** to listen for

```html
<!-- ✅ Works: preset registered for <dialog> -->
<dialog $:model:open="showDialog">...</dialog>
<!-- Binds to .open property, listens to 'close' event -->
```

## Built-in Presets

### `<dialog>` Element

**Preset:** `{ prop: 'open', event: 'close' }`

```html live:no-code
<div id="demo-dialog" class="en-card en-card__body">
  <button class="en-btn en-btn--primary en-mb-3" $:click="showDialog = true">Open Dialog</button>
  <dialog $:model:open="showDialog" class="en-dialog">
    <div class="en-dialog__content">
      <h3 class="en-heading-3">Dialog Title</h3>
      <p class="en-mb-3">This is a native dialog element.</p>
      <button class="en-btn en-btn--sm" $:click="showDialog = false">Close</button>
    </div>
  </dialog>
</div>
```

```html
<div id="demo-dialog">
  <button $:click="showDialog = true">Open Dialog</button>
  
  <dialog $:model:open="showDialog">
    <h3>Dialog Title</h3>
    <p>This is a native dialog element.</p>
    <button $:click="showDialog = false">Close</button>
  </dialog>
</div>
```

```js inject show
$('#demo-dialog').component({
  state: { showDialog: false }
});
```

```js cleanup
onUnmount(() => {
  $('#demo-dialog').component('destroy');
})
```

**How it works:**
- `state.showDialog = true` → sets `dialog.open = true`
- User presses **Escape** or clicks backdrop → `close` event fires → sets `state.showDialog = false`

**Why preset needed:**
- `<dialog>` has `.open` property (not `.value`)
- Browser fires `close` event (not `input` or `change`)

### `<details>` Element

**Preset:** `{ prop: 'open', event: 'toggle' }`

```html live:no-code
<div id="demo-details" class="en-card en-card__body">
  <p class="en-mb-2">Details state: <code $:text="expanded ? 'Open' : 'Closed'"></code></p>
  <details $:model:open="expanded" class="en-details en-mb-2">
    <summary class="en-details__summary">Click to expand</summary>
    <div class="en-details__content en-p-3">
      <p class="en-mb-0">This content is collapsible.</p>
    </div>
  </details>
  <button class="en-btn en-btn--sm en-btn--outline" $:click="expanded = !expanded">Toggle Programmatically</button>
</div>
```

```html
<div id="demo-details">
  <p>Details state: <code $:text="expanded ? 'Open' : 'Closed'"></code></p>
  
  <details $:model:open="expanded">
    <summary>Click to expand</summary>
    <p>This content is collapsible.</p>
  </details>
  
  <button $:click="expanded = !expanded">Toggle Programmatically</button>
</div>
```

```js inject show
$('#demo-details').component({
  state: { expanded: false }
});
```

```js cleanup
onUnmount(() => {
  $('#demo-details').component('destroy');
})
```

**How it works:**
- `state.expanded = true` → sets `details.open = true`
- User clicks `<summary>` → `toggle` event fires → sets `state.expanded` to new value

## Using Model Presets

### Basic Syntax

```html
<element $:model:<prop>="stateKey"></element>
```

- `<prop>` - DOM property name (e.g., `open`, `value`, `checked`)
- `stateKey` - State property to bind

### With Built-in Presets

```html
<!-- Dialog -->
<dialog $:model:open="showModal">
  <p>Modal content</p>
  <button $:click="showModal = false">Close</button>
</dialog>

<!-- Details -->
<details $:model:open="sectionExpanded">
  <summary>Section Title</summary>
  <p>Section content</p>
</details>
```

### Explicit Event Override

Override the preset's event with `.on-<event>` modifier:

```html
<!-- Use 'cancel' event instead of preset's 'close' -->
<dialog $:model:open.on-cancel="showModal">...</dialog>
```

## Creating Custom Presets

### Registration API

```js
$.jqr.registerModelPreset(tagName, preset);
```

**Parameters:**
- `tagName` - HTML tag name (lowercase)
- `preset` - Object with `{ prop: string, event: string }`

### Example: Custom Element

```js
// Register preset for custom element
$.jqr.registerModelPreset('my-dropdown', {
  prop: 'expanded',
  event: 'toggle'
});
```

```html
<!-- Now works without explicit event -->
<my-dropdown $:model:expanded="dropdownOpen">
  <option>Option 1</option>
  <option>Option 2</option>
</my-dropdown>
```

**Preset tells JQR:**
- Read/write `el.expanded` property
- Listen to `toggle` event for updates

### Example: Video Element

```js
// Preset for <video> paused state
$.jqr.registerModelPreset('video', {
  prop: 'paused',
  event: 'play'  // Could also use 'pause' or custom logic
});
```

```html
<video $:model:paused="isPaused" src="video.mp4"></video>
<button $:click="isPaused = !isPaused">
  <span $:text="isPaused ? 'Play' : 'Pause'"></span>
</button>
```

### Example: Audio Element

```js
// Preset for <audio> volume
$.jqr.registerModelPreset('audio', {
  prop: 'volume',
  event: 'volumechange'
});
```

```html
<audio $:model:volume="audioVolume" src="audio.mp3"></audio>
<input type="range" $:model.number="audioVolume" min="0" max="1" step="0.1" />
```

## Advanced Patterns

### Multiple Presets Per Tag

**Last registration wins:**

```js
// First registration
$.jqr.registerModelPreset('dialog', {
  prop: 'open',
  event: 'close'
});

// Override with custom event
$.jqr.registerModelPreset('dialog', {
  prop: 'open',
  event: 'my-custom-close'  // Now uses this
});
```

Use this to override built-in presets for polyfills or custom behavior.

### Preset with Modifier Fallback

```html
<!-- Uses preset's event -->
<dialog $:model:open="showModal">...</dialog>

<!-- Overrides with modifier -->
<dialog $:model:open.on-cancel="showModal">...</dialog>
```

**Priority:**
1. `.on-<event>` modifier (highest)
2. Registered preset for tag
3. `change` event with dev warning (fallback)

### Custom Elements with Multiple Properties

```js
// Preset for most common property
$.jqr.registerModelPreset('color-picker', {
  prop: 'value',
  event: 'change'
});
```

```html
<!-- Common case: use preset -->
<color-picker $:model:value="selectedColor"></color-picker>

<!-- Other properties: explicit event -->
<color-picker $:model:alpha.on-alphachange="transparency"></color-picker>
```

## Preset Internals

### How Presets Are Used

When you write:

```html
<dialog $:model:open="showModal">
```

JQR does:

1. **Check for explicit modifier:**
   - `.on-<event>` → use that event
   
2. **Check for preset:**
   - Look up `getModelPreset('dialog')`
   - Found: `{ prop: 'open', event: 'close' }`
   
3. **Bind two-way:**
   ```js
   // State → DOM
   effect(() => {
     el.open = state.showModal;
   });
   
   // DOM → State
   el.addEventListener('close', () => {
     state.showModal = el.open;
   });
   ```

### Without Preset

```html
<!-- No preset registered -->
<custom-element $:model:myProp="value">
```

**Result:** Dev warning + fallback to `change` event:

```
[jqr] $:model:myProp on <custom-element> has no known event 
(no preset registered) — defaulting to "change", which may never fire. 
Pass explicit .on-<event> modifier.
```

**Fix:** Register preset or use modifier:

```html
<!-- Option 1: Register preset -->
<script>
$.jqr.registerModelPreset('custom-element', {
  prop: 'myProp',
  event: 'propchange'
});
</script>

<!-- Option 2: Explicit modifier -->
<custom-element $:model:myProp.on-propchange="value">
```

## Preset Limitations

### 1. One Preset Per Tag

```js
// ❌ Can't register multiple presets for same tag
$.jqr.registerModelPreset('video', { prop: 'paused', event: 'play' });
$.jqr.registerModelPreset('video', { prop: 'volume', event: 'volumechange' });
// Second registration overwrites first
```

**Workaround:** Use explicit modifiers for non-preset properties:

```html
<video $:model:paused="isPaused"></video>  <!-- Uses preset -->
<video $:model:volume.on-volumechange="vol"></video>  <!-- Explicit -->
```

### 2. Property Only, No Value Transform

Presets only define the **event name**. The property is read/written as-is:

```js
// ❌ Can't transform value in preset
$.jqr.registerModelPreset('my-element', {
  prop: 'value',
  event: 'change',
  transform: (val) => val * 100  // Not supported
});
```

**Workaround:** Use computed properties:

```js
$('#app').component({
  state: { rawValue: 0 },
  computed: {
    displayValue: {
      get() { return this.state.rawValue * 100; },
      set(val) { this.state.rawValue = val / 100; }
    }
  }
});
```

```html
<my-element $:model:value="displayValue">
```

### 3. No Validation in Preset

Presets don't validate values — that's the component's job:

```js
$('#app').component({
  state: { volume: 0.5 },
  watch: {
    volume(newVal) {
      // Clamp volume
      if (newVal < 0) this.state.volume = 0;
      if (newVal > 1) this.state.volume = 1;
    }
  }
});
```

## Common Use Cases

### Polyfill Support

Override built-in preset for polyfilled element:

```js
// Polyfill dispatches 'polyfill-close' instead of 'close'
$.jqr.registerModelPreset('dialog', {
  prop: 'open',
  event: 'polyfill-close'
});
```

### Web Components

```js
// Custom web component
$.jqr.registerModelPreset('app-slider', {
  prop: 'value',
  event: 'slide'
});

$.jqr.registerModelPreset('app-toggle', {
  prop: 'checked',
  event: 'toggle'
});

$.jqr.registerModelPreset('app-datepicker', {
  prop: 'value',
  event: 'dateselect'
});
```

### Third-Party Widgets

```js
// jQuery UI slider
$.jqr.registerModelPreset('div[data-slider]', {
  prop: 'slider-value',  // Custom property you manage
  event: 'slidechange'
});
```

**Note:** Tag selector must be simple tag name (no attributes in preset). Use class/attribute logic in your component code.

## Best Practices

### 1. Register Presets Early

```js
// ✅ Good: Register before mounting components
$.jqr.registerModelPreset('my-widget', { /* ... */ });

$('#app').component({ /* ... */ });

// ❌ Bad: Register after mount
$('#app').component({ /* ... */ });
$.jqr.registerModelPreset('my-widget', { /* ... */ });  // Too late!
```

### 2. Use Semantic Event Names

```js
// ✅ Good: Clear, semantic event names
$.jqr.registerModelPreset('color-picker', {
  prop: 'value',
  event: 'colorchange'
});

// ❌ Bad: Generic event names
$.jqr.registerModelPreset('color-picker', {
  prop: 'value',
  event: 'update'  // Too generic
});
```

### 3. Document Custom Presets

```js
/**
 * Preset for <app-rating> component
 * Binds to .rating property (1-5)
 * Listens for 'ratingchange' event
 */
$.jqr.registerModelPreset('app-rating', {
  prop: 'rating',
  event: 'ratingchange'
});
```

### 4. Prefer Built-in Events

```js
// ✅ Good: Uses standard DOM event
$.jqr.registerModelPreset('contenteditable-div', {
  prop: 'textContent',
  event: 'input'  // Standard DOM event
});

// ❌ Bad: Custom event when standard exists
$.jqr.registerModelPreset('contenteditable-div', {
  prop: 'textContent',
  event: 'my-custom-input'  // Why custom?
});
```

## Debugging

### Enable Dev Mode

```js
$.jqr.config({ devMode: true });
```

Shows warnings for:
- Missing presets (fallback to `change`)
- Explicit event needed

### Check Registered Presets

```js
// In browser console
$.jqr.getModelPreset('dialog');
// → { prop: 'open', event: 'close' }

$.jqr.getModelPreset('details');
// → { prop: 'open', event: 'toggle' }

$.jqr.getModelPreset('unknown');
// → undefined
```

### Test Preset Binding

```js
$('#test').component({
  state: { value: false },
  mounted() {
    // Watch property changes
    console.log('Initial:', this.$el.find('dialog')[0].open);
    
    this.watch('value', (newVal) => {
      console.log('State changed to:', newVal);
      console.log('DOM property:', this.$el.find('dialog')[0].open);
    });
  }
});
```

## Summary

**Key Concepts:**

1. **Presets define binding** - Property name + event name for non-form elements
2. **Built-in presets** - `<dialog>` (open/close), `<details>` (open/toggle)
3. **Custom presets** - Register via `$.jqr.registerModelPreset(tag, { prop, event })`
4. **Override** - Use `.on-<event>` modifier to override preset
5. **Fallback** - No preset = `change` event with warning

**When to use:**
- Custom elements with non-standard events
- Web components
- Polyfilled native elements
- Third-party widgets

**API:**

```js
// Register
$.jqr.registerModelPreset(tag, { prop, event });

// Query
$.jqr.getModelPreset(tag);  // → { prop, event } | undefined
```

## Next Steps

- [Forms & Models Guide](./forms-and-models.md) - Complete model binding reference
- [Model Modifiers](./model-modifiers.md) - `.number`, `.trim`, `.debounce`, etc.
- [Custom Directives](./custom-directives.md) - Creating custom directives
- [Component Options](./component-options.md) - Full component API
