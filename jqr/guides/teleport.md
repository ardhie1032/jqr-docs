<!-- {
  "path": "jqr/guides/teleport",
  "name": "docs-jqr-guides-teleport",
  "meta": {
    "title": "Teleport",
    "order": 10
  }
} -->

# Teleport

Teleport lets you render a component's content in a different place in the DOM tree, outside its normal parent hierarchy. Perfect for modals, toasts, tooltips, and overlays that need to escape CSS constraints.

## Why Teleport?

Some UI elements need to be at the document root to work correctly:

**Common Issues:**
- Modal inside a `overflow: hidden` container
- Tooltip clipped by `position: relative` parent
- Toast notification inheriting `z-index` from deep nesting
- Full-screen overlay blocked by ancestor transforms

**Teleport solves this** by rendering content at a target location (usually `body` or a dedicated portal root) while keeping the component logic together.

## Basic Usage

Use `$:teleport` directive to move an element:

```html live:no-code
<div id="demo-teleport-basic" class="en-card">
  <div class="en-card__body">
    <h3>Basic Teleport</h3>
    <button class="en-btn en-btn--primary" $:click="show = !show">
      Toggle Toast
    </button>
    
    <div $:if="show" $:teleport:to="body" class="toast-notification">
      ✓ This is rendered at document body!
    </div>
  </div>
</div>

<style>
.toast-notification {
  position: fixed;
  top: 20px;
  right: 20px;
  background: #10b981;
  color: white;
  padding: 1rem 1.5rem;
  border-radius: 8px;
  box-shadow: 0 4px 12px rgba(0, 0, 0, 0.15);
  z-index: 9999;
}
</style>
```

```html
<div id="demo-teleport-basic">
  <button $:click="show = !show">Toggle Toast</button>
  
  <div $:if="show" $:teleport:to="body">
    ✓ This is rendered at document body!
  </div>
</div>
```

```js inject show
$('#demo-teleport-basic').component({
  state: { show: false }
});
```

```js cleanup
onUnmount(() => {
  $('#demo-teleport-basic').component('destroy');
})
```

**What happens:**
1. Component is mounted in `#demo-teleport-basic`
2. `$:teleport:to="body"` moves the toast to `<body>`
3. Element stays reactive and bound to component state
4. When component destroys, toast is cleaned up automatically

## Two Syntax Forms

Teleport supports two forms: **object form** (reactive, evaluates expressions) and **arg form** (literal, simpler).

### Object Form (Reactive)

Accepts a JavaScript expression that evaluates to a string or object:

```html
<!-- Simple string target -->
<div $:teleport="'body'">...</div>
<div $:teleport="'#modal-root'">...</div>

<!-- Reactive target (changes when state.target changes) -->
<div $:teleport="state.target">...</div>

<!-- Object with options -->
<div $:teleport="{ to: '#portal', mode: 'prepend' }">...</div>

<!-- Object with defer option (waits for target) -->
<div $:teleport="{ to: '#late-target', defer: true }">...</div>
```

### Arg Form (Literal, Non-Reactive)

Simpler syntax using modifiers, target is treated as **literal string** (not evaluated):

```html
<!-- Basic - default append mode -->
<div $:teleport:to="#modal-root">...</div>

<!-- With mode modifiers -->
<div $:teleport:to.prepend="body">...</div>
<div $:teleport:to.append="#portal">...</div>
<div $:teleport:to.insert-after="#header">...</div>
<div $:teleport:to.insert-before="#footer">...</div>

<!-- With defer modifier (waits for target to appear) -->
<div $:teleport:to.defer.append="#late-target">...</div>
<div $:teleport:to.defer.prepend="body">...</div>
```

**Key differences:**

| Feature | Object Form | Arg Form |
|---------|-------------|----------|
| Syntax | `$:teleport="expr"` | `$:teleport:to.modifiers="target"` |
| Target value | JS expression (evaluated) | Literal string |
| Reactive target | ✅ Yes (target can change) | ❌ No (static) |
| Mode | `mode:` property or default | Modifier (`.append`, `.prepend`, etc.) |
| Defer | `defer: true` property | `.defer` modifier |
| Use case | Dynamic targets, reactive | Static targets, cleaner syntax |

**When to use which:**
- **Object form:** When target needs to be dynamic/reactive
- **Arg form:** When target is static and known upfront (most common)

## Target Selectors

Both forms accept any CSS selector as target:

```html
<!-- Document body -->
<div $:teleport:to="body">...</div>
<div $:teleport="'body'">...</div>

<!-- Specific ID -->
<div $:teleport:to="#modal-root">...</div>
<div $:teleport="'#modal-root'">...</div>

<!-- Class -->
<div $:teleport:to=".portal-container">...</div>
<div $:teleport="'.portal-container'">...</div>

<!-- Data attribute -->
<div $:teleport:to="[data-portal]">...</div>
<div $:teleport="'[data-portal]'">...</div>
```

**Target availability:**
- Without `defer`: Target must exist when component mounts, or you'll get console error
- With `defer`: Waits for target to appear in DOM (using `MutationObserver`)

## Teleport Modes

Control **where** content is inserted relative to the target:

### Object Form

```html
<!-- Append (default) - adds to end of target's children -->
<div $:teleport="{ to: 'body', mode: 'append' }">...</div>

<!-- Prepend - adds to beginning of target's children -->
<div $:teleport="{ to: 'body', mode: 'prepend' }">...</div>

<!-- Insert after target element itself -->
<div $:teleport="{ to: '#header', mode: 'insertAfter' }">...</div>

<!-- Insert before target element itself -->
<div $:teleport="{ to: '#footer', mode: 'insertBefore' }">...</div>
```

### Arg Form (with modifiers)

```html
<!-- Append (default) -->
<div $:teleport:to.append="body">...</div>

<!-- Prepend -->
<div $:teleport:to.prepend="body">...</div>

<!-- Insert after target -->
<div $:teleport:to.insert-after="#header">...</div>

<!-- Insert before target -->
<div $:teleport:to.insert-before="#footer">...</div>
```

**Mode reference:**

| Mode | Object Property | Arg Modifier | Behavior |
|------|----------------|--------------|----------|
| Append (default) | `mode: 'append'` | `.append` or none | Add to end of target's children |
| Prepend | `mode: 'prepend'` | `.prepend` | Add to start of target's children |
| Insert After | `mode: 'insertAfter'` | `.insert-after` | Insert after target element |
| Insert Before | `mode: 'insertBefore'` | `.insert-before` | Insert before target element |

### Mode Examples

```html live:no-code
<div id="demo-teleport-modes" class="en-card">
  <div class="en-card__body">
    <h3>Teleport Modes</h3>
    <div class="en-flex en-gap-2" style="margin-bottom: 1rem;">
      <button class="en-btn en-btn--sm" $:click="mode = 'append'">Append</button>
      <button class="en-btn en-btn--sm" $:click="mode = 'prepend'">Prepend</button>
    </div>
    
    <div id="target-container" class="en-alert en-alert--info">
      <p style="margin: 0.5rem 0;"><strong>Target Container</strong></p>
      <p style="margin: 0; font-size: 0.875rem;">Mode: <strong $:text="mode"></strong></p>
    </div>
    
    <div $:teleport="{ to: '#target-container', mode: mode }" class="en-badge en-badge--success">
      Teleported Item
    </div>
  </div>
</div>
```

```html
<div id="demo-teleport-modes">
  <button $:click="mode = 'append'">Append</button>
  <button $:click="mode = 'prepend'">Prepend</button>
  
  <div id="target-container">
    <strong>Target Container</strong>
    <p>Mode: <span $:text="mode"></span></p>
  </div>
  
  <div $:teleport="{ to: '#target-container', mode: mode }">
    Teleported Item
  </div>
</div>
```

```js inject show
$('#demo-teleport-modes').component({
  state: { mode: 'append' }
});
```

```js cleanup
onUnmount(() => {
  $('#demo-teleport-modes').component('destroy');
})
```

## Deferred Teleport

By default, if the target doesn't exist when teleport executes, you get an immediate console error. **Defer mode** changes this: it waits for the target to appear using `MutationObserver`.

### When to Use Defer

Use defer when the target element will be created **after** the teleporting component:

**Common scenarios:**
- Target is rendered conditionally with `$:if`
- Target comes from async data (`$:for` over fetched array)
- Target is inside another component that mounts later
- Target is created by a third-party library or AJAX response

### Object Form with Defer

```html
<!-- Wait for target to appear -->
<div $:teleport="{ to: '#late-target', defer: true }">
  Content teleports when #late-target appears
</div>

<!-- Combine with mode -->
<div $:teleport="{ to: '#late-target', defer: true, mode: 'prepend' }">
  Prepends to target when available
</div>
```

### Arg Form with Defer

```html
<!-- Basic defer -->
<div $:teleport:to.defer="#late-target">...</div>

<!-- Defer with prepend mode -->
<div $:teleport:to.defer.prepend="#late-target">...</div>

<!-- Defer with insert-after -->
<div $:teleport:to.defer.insert-after="#dynamic-element">...</div>
```

### Defer Example: Conditional Target

```html live:no-code
<div id="demo-teleport-defer" class="en-card">
  <div class="en-card__body">
    <h3>Deferred Teleport</h3>
    <div class="en-flex en-gap-2" style="margin-bottom: 1rem;">
      <button class="en-btn en-btn--primary" $:click="showTarget = !showTarget">
        <span $:text="showTarget ? 'Hide Target' : 'Show Target'"></span>
      </button>
    </div>
    
    <div class="en-alert en-alert--info" style="margin-bottom: 1rem;">
      <p style="margin: 0;">Target visible: <strong $:text="showTarget ? 'Yes' : 'No'"></strong></p>
    </div>
    
    <!-- Target appears conditionally -->
    <div $:if="showTarget" id="deferred-target" class="en-alert en-alert--success">
      <p style="margin: 0;"><strong>Target Container</strong></p>
    </div>
    
    <!-- Teleported content waits for target -->
    <div $:teleport:to.defer="#deferred-target" class="en-badge en-badge--primary">
      I waited for the target!
    </div>
  </div>
</div>
```

```html
<div id="demo-teleport-defer">
  <button $:click="showTarget = !showTarget">
    Toggle Target
  </button>
  
  <p>Target visible: <span $:text="showTarget"></span></p>
  
  <!-- Target appears conditionally -->
  <div $:if="showTarget" id="deferred-target">
    Target Container
  </div>
  
  <!-- This waits for #deferred-target to exist -->
  <div $:teleport:to.defer="#deferred-target">
    I waited for the target!
  </div>
</div>
```

```js inject show
$('#demo-teleport-defer').component({
  state: { showTarget: false }
});
```

```js cleanup
onUnmount(() => {
  $('#demo-teleport-defer').component('destroy');
})
```

**What happens:**
1. Component mounts, `#deferred-target` doesn't exist yet
2. Defer mode sets up `MutationObserver` watching for target
3. When you click "Show Target", `$:if="showTarget"` creates the element
4. Observer detects the target and immediately teleports content
5. If component destroys before target appears, observer is cleaned up automatically

### Defer Example: Async Content

Waiting for dynamically loaded content:

```html live:no-code
<div id="demo-teleport-async" class="en-card">
  <div class="en-card__body">
    <h3>Async Target</h3>
    <button class="en-btn en-btn--primary" $:click="loadTarget()">
      Load Target
    </button>
    
    <div class="en-alert en-alert--info" style="margin: 1rem 0;">
      <p style="margin: 0;">Status: <strong $:text="status"></strong></p>
    </div>
    
    <!-- Target loaded asynchronously -->
    <div $:if="targetLoaded" id="async-target" class="en-alert en-alert--success">
      <p style="margin: 0;"><strong>Async Target Container</strong></p>
    </div>
    
    <!-- Deferred teleport waits for async target -->
    <div $:teleport:to.defer.prepend="#async-target" class="en-badge en-badge--warning">
      ⏳ Waiting for async target...
    </div>
  </div>
</div>
```

```html
<div id="demo-teleport-async">
  <button $:click="loadTarget()">Load Target</button>
  
  <p>Status: <span $:text="status"></span></p>
  
  <!-- Simulated async target -->
  <div $:if="targetLoaded" id="async-target">
    Async Target
  </div>
  
  <!-- Waits for async target -->
  <div $:teleport:to.defer.prepend="#async-target">
    ⏳ Waiting...
  </div>
</div>
```

```js inject show
$('#demo-teleport-async').component({
  state: {
    targetLoaded: false,
    status: 'Ready'
  },
  
  methods: {
    async loadTarget(state) {
      state.status = 'Loading...';
      
      // Simulate async operation
      await new Promise(resolve => setTimeout(resolve, 2000));
      
      state.targetLoaded = true;
      state.status = 'Loaded!';
    }
  }
});
```

```js cleanup
onUnmount(() => {
  $('#demo-teleport-async').component('destroy');
})
```

### Defer Behavior Details

**Immediate mode (default, no defer):**
- Target must exist now
- Fails immediately with console error if missing
- No waiting, no observer

**Defer mode:**
- If target exists now → teleports immediately (no waiting)
- If target missing → sets up `MutationObserver` on `<body>`
- Observer watches for target selector to appear
- Once found → teleports content and disconnects observer
- If component destroys first → observer auto-cleanup, no memory leak

**Important notes:**
- In object form with reactive target, `defer` only applies to the **first** teleport attempt. Subsequent reactive changes to `state.target` teleport immediately (no defer).
- Defer uses `MutationObserver` on `document.body` with `{ childList: true, subtree: true }` — efficient for modern browsers.
- If target never appears and component stays mounted, observer keeps waiting (until component destroy).



## Modal Pattern

Classic modal with teleport to escape layout constraints:

```html live:no-code
<div id="demo-teleport-modal" class="en-card">
  <div class="en-card__body">
    <h3>Modal with Teleport</h3>
    <button class="en-btn en-btn--primary" $:click="showModal = true">
      Open Modal
    </button>
    
    <div $:if="showModal" $:teleport:to="body" class="modal-overlay" $:click="showModal = false">
      <div class="modal-dialog en-card" $:click.stop="void 0">
        <div class="en-card__body">
          <h4>Modal Title</h4>
          <p>This modal is teleported to body, so it always appears on top.</p>
          <div class="en-flex en-gap-2">
            <button class="en-btn en-btn--primary" $:click="showModal = false">Close</button>
          </div>
        </div>
      </div>
    </div>
  </div>
</div>

<style>
.modal-overlay {
  position: fixed;
  top: 0;
  left: 0;
  right: 0;
  bottom: 0;
  background: rgba(0, 0, 0, 0.5);
  display: flex;
  align-items: center;
  justify-content: center;
  z-index: 9999;
}

.modal-dialog {
  max-width: 500px;
  width: 90%;
  margin: 0;
}
</style>
```

```html
<div id="demo-teleport-modal">
  <button $:click="showModal = true">Open Modal</button>
  
  <div $:if="showModal" $:teleport:to="body" class="modal-overlay" $:click="showModal = false">
    <div class="modal-dialog" $:click.stop="void 0">
      <h4>Modal Title</h4>
      <p>This modal is teleported to body.</p>
      <button $:click="showModal = false">Close</button>
    </div>
  </div>
</div>
```

```js inject show
$('#demo-teleport-modal').component({
  state: { showModal: false }
});
```

```js cleanup
onUnmount(() => {
  $('#demo-teleport-modal').component('destroy');
})
```

**Benefits:**
- Modal appears at `<body>` level
- No `z-index` issues from parent containers
- No clipping from `overflow: hidden`
- Clean DOM structure

## Toast Notifications

Auto-dismissing notifications:

```html live:no-code
<div id="demo-teleport-toast" class="en-card">
  <div class="en-card__body">
    <h3>Toast Notifications</h3>
    <div class="en-flex en-gap-2">
      <button class="en-btn en-btn--success" $:click="showToast('success')">Success</button>
      <button class="en-btn en-btn--danger" $:click="showToast('error')">Error</button>
      <button class="en-btn en-btn--primary" $:click="showToast('info')">Info</button>
    </div>
    
    <div $:if="toast.visible" $:teleport:to="body" class="toast-container">
      <div class="toast" $:class="{
        'toast--success': toast.type === 'success',
        'toast--error': toast.type === 'error',
        'toast--info': toast.type === 'info'
      }">
        <span $:text="toast.message"></span>
      </div>
    </div>
  </div>
</div>

<style>
.toast-container {
  position: fixed;
  top: 20px;
  right: 20px;
  z-index: 9999;
}

.toast {
  padding: 1rem 1.5rem;
  border-radius: 8px;
  color: white;
  box-shadow: 0 4px 12px rgba(0, 0, 0, 0.15);
  animation: slideIn 0.3s ease;
}

.toast--success { background: #10b981; }
.toast--error { background: #ef4444; }
.toast--info { background: #3b82f6; }

@keyframes slideIn {
  from {
    transform: translateX(100%);
    opacity: 0;
  }
  to {
    transform: translateX(0);
    opacity: 1;
  }
}
</style>
```

```html
<div id="demo-teleport-toast">
  <button $:click="showToast('success')">Success</button>
  <button $:click="showToast('error')">Error</button>
  
  <div $:if="toast.visible" $:teleport:to="body" class="toast-container">
    <div class="toast" $:class="{ 
      'toast--success': toast.type === 'success',
      'toast--error': toast.type === 'error'
    }">
      <span $:text="toast.message"></span>
    </div>
  </div>
</div>
```

```js inject show
const toastDemo = $('#demo-teleport-toast').component({
  state: {
    toast: {
      visible: false,
      type: 'info',
      message: ''
    }
  },
  
  methods: {
    showToast(state, type) {
      const messages = {
        success: '✓ Action completed successfully!',
        error: '✗ Something went wrong',
        info: 'ℹ Here is some information'
      };
      
      state.toast = {
        visible: true,
        type: type,
        message: messages[type]
      };
      
      setTimeout(() => {
        state.toast.visible = false;
      }, 3000);
    }
  }
});
```

```js cleanup
onUnmount(() => {
  $('#demo-teleport-toast').component('destroy');
})
```

## Tooltip Pattern

Context-aware tooltips:

```html live:no-code
<div id="demo-teleport-tooltip" class="en-card">
  <div class="en-card__body">
    <h3>Tooltip with Teleport</h3>
    <p>Hover over the button below:</p>
    <button 
      class="en-btn en-btn--primary" 
      $:mouseenter="showTooltip = true" 
      $:mouseleave="showTooltip = false">
      Hover Me
    </button>
    
    <div $:if="showTooltip" $:teleport:to="body" class="tooltip">
      This is a helpful tooltip!
    </div>
  </div>
</div>

<style>
.tooltip {
  position: fixed;
  bottom: 50%;
  left: 50%;
  transform: translate(-50%, -50px);
  background: #1f2937;
  color: white;
  padding: 0.5rem 1rem;
  border-radius: 6px;
  font-size: 0.875rem;
  white-space: nowrap;
  z-index: 9999;
  pointer-events: none;
}

.tooltip::after {
  content: '';
  position: absolute;
  top: 100%;
  left: 50%;
  transform: translateX(-50%);
  border: 6px solid transparent;
  border-top-color: #1f2937;
}
</style>
```

```html
<div id="demo-teleport-tooltip">
  <button 
    $:mouseenter="showTooltip = true" 
    $:mouseleave="showTooltip = false">
    Hover Me
  </button>
  
  <div $:if="showTooltip" $:teleport:to="body" class="tooltip">
    This is a helpful tooltip!
  </div>
</div>
```

```js inject show
$('#demo-teleport-tooltip').component({
  state: { showTooltip: false }
});
```

```js cleanup
onUnmount(() => {
  $('#demo-teleport-tooltip').component('destroy');
})
```

## Portal Root Pattern

Create dedicated portal containers:

```html
<!-- In your HTML -->
<div id="app"></div>
<div id="modal-root"></div>
<div id="toast-root"></div>
```

```html
<!-- In components - using arg form -->
<div $:if="showModal" $:teleport:to="#modal-root">
  <div class="modal">...</div>
</div>

<div $:if="toast" $:teleport:to="#toast-root">
  <div class="toast">...</div>
</div>
```

**Benefits:**
- Organized DOM structure
- Easy to style portal containers
- Better for debugging
- Clear separation of concerns

**With defer for late-created portals:**

If portal roots might be created dynamically (e.g., by a layout component that mounts after children), use defer:

```html
<div $:if="showModal" $:teleport:to.defer="#modal-root">
  <div class="modal">...</div>
</div>
```

## Multiple Teleports

You can have multiple teleported elements:

```html
<div id="app">
  <!-- All these work simultaneously -->
  <div $:if="modal1" $:teleport:to="#modal-root">...</div>
  <div $:if="modal2" $:teleport:to="#modal-root">...</div>
  <div $:if="toast" $:teleport:to="#toast-root">...</div>
  <div $:if="tooltip" $:teleport:to="body">...</div>
</div>
```

**Each teleport:**
- Maintains its own reactive bindings
- Cleans up independently
- Can target same or different destinations

## Combining with Transitions

Teleport works seamlessly with `$:transition`:

```html
<div 
  $:if="show" 
  $:teleport:to="body"
  $:transition="'fade'">
  <div class="modal">
    Animated modal content
  </div>
</div>
```

**Order matters:**
1. `$:if` controls visibility
2. `$:transition` animates enter/leave
3. `$:teleport` moves to target location

## Reactivity Preserved

Teleported elements stay fully reactive:

```html live:no-code
<div id="demo-teleport-reactive" class="en-card">
  <div class="en-card__body">
    <h3>Reactive Teleport</h3>
    <p>Counter in component: <strong $:text="count"></strong></p>
    <button class="en-btn en-btn--primary" $:click="count++">Increment</button>
    <button class="en-btn en-btn--outline" $:click="show = !show" $:text="show ? 'Hide Panel' : 'Show Panel'"></button>
    
    <div $:if="show" $:teleport:to="body" class="reactive-panel">
      <div class="en-card">
        <div class="en-card__body">
          <h4>Teleported Panel</h4>
          <p>Counter here: <strong $:text="count"></strong></p>
          <p class="en-text--muted">Both counters stay in sync!</p>
        </div>
      </div>
    </div>
  </div>
</div>

<style>
.reactive-panel {
  position: fixed;
  bottom: 20px;
  right: 20px;
  z-index: 9999;
  max-width: 300px;
}
</style>
```

```html
<div id="demo-teleport-reactive">
  <p>Counter: <strong $:text="count"></strong></p>
  <button $:click="count++">Increment</button>
  <button $:click="show = !show">Toggle Panel</button>
  
  <div $:if="show" $:teleport:to="body" class="reactive-panel">
    <h4>Teleported Panel</h4>
    <p>Counter: <strong $:text="count"></strong></p>
    <p>Both stay in sync!</p>
  </div>
</div>
```

```js inject show
$('#demo-teleport-reactive').component({
  state: { 
    count: 0,
    show: false
  }
});
```

```js cleanup
onUnmount(() => {
  $('#demo-teleport-reactive').component('destroy');
})
```

**Key point:** Teleported content remains part of the component's reactive scope.

## Best Practices

### 1. Always Teleport Overlays

```html
<!-- ✅ Good - modal at body level -->
<div $:if="showModal" $:teleport="'body'">
  <div class="modal-overlay">...</div>
</div>

<!-- ❌ Bad - modal stuck in container -->
<div class="container" style="overflow: hidden">
  <div $:if="showModal">
    <div class="modal-overlay">...</div>
  </div>
</div>
```

### 2. Use Arg Form for Static Targets

```html
<!-- ✅ Good - clean, readable -->
<div $:teleport:to="body">...</div>
<div $:teleport:to.defer="#modal-root">...</div>

<!-- ❌ Verbose - unnecessary for static targets -->
<div $:teleport="'body'">...</div>
<div $:teleport="{ to: '#modal-root', defer: true }">...</div>
```

Use object form only when target needs to be dynamic/reactive.

### 3. Use defer for Dynamic Targets

```html
<!-- ✅ Good - waits for conditional target -->
<div $:teleport:to.defer="#dynamic-target">...</div>

<!-- ❌ Bad - will error if target doesn't exist yet -->
<div $:teleport:to="#dynamic-target">...</div>
```

If target might not exist at mount time, use `defer`.

### 4. Use Semantic Target IDs

```html
<!-- ✅ Good - clear purpose -->
<div $:teleport:to="#modal-root">...</div>
<div $:teleport:to="#toast-root">...</div>

<!-- ❌ Confusing - generic targets -->
<div $:teleport:to="#portal-1">...</div>
<div $:teleport:to="#portal-2">...</div>
```

### 5. Combine with $:if

```html
<!-- ✅ Good - conditionally teleport -->
<div $:if="show" $:teleport:to="body">...</div>

<!-- ❌ Bad - always in DOM -->
<div $:teleport:to="body" $:show="show">...</div>
```

### 6. Clean Up on Destroy

Teleported elements are **automatically cleaned up** when:
- Component is destroyed
- `$:if` becomes false
- Element is removed

No manual cleanup needed! Defer observers are also auto-cleaned.

### 7. Choose the Right Form

**Use arg form when:**
- Target is static and known upfront (90% of cases)
- You want cleaner, more readable syntax
- Target is a simple selector string

**Use object form when:**
- Target needs to be reactive (changes with state)
- Target comes from a computed value
- You need complex logic for target selection

```html
<!-- Arg form - most common -->
<div $:teleport:to.defer.prepend="#portal">...</div>

<!-- Object form - when dynamic -->
<div $:teleport="{ to: state.currentPortal, mode: 'append' }">...</div>
```

## Common Patterns

### Stacked Modals

```html
<!-- Both modals at body level -->
<div $:if="modal1" $:teleport:to="body" class="modal" style="z-index: 9999">
  Modal 1
  <button $:click="modal2 = true">Open Modal 2</button>
</div>

<div $:if="modal2" $:teleport:to="body" class="modal" style="z-index: 10000">
  Modal 2
  <button $:click="modal2 = false">Close</button>
</div>
```

### Toast Queue

```html
<div $:for="toast in toasts" $:key="toast.id" $:teleport:to="#toast-root">
  <div class="toast" $:text="toast.message"></div>
</div>
```

### Context Menu

```html
<div 
  $:if="contextMenu.show" 
  $:teleport:to="body"
  class="context-menu"
  $:style="{ 
    left: contextMenu.x + 'px', 
    top: contextMenu.y + 'px' 
  }">
  <button $:click="handleCopy">Copy</button>
  <button $:click="handlePaste">Paste</button>
</div>
```

### Dynamic Portal Selection

When you need reactive target, use object form:

```html
<!-- Target changes based on state -->
<div $:teleport="state.currentPortal">
  Content goes to different portals
</div>
```

```js
methods: {
  switchPortal(state) {
    // Reactively change teleport target
    state.currentPortal = state.isFullscreen ? 'body' : '#modal-root';
  }
}
```

## Limitations

### SSR Not Supported

Teleport is client-side only. For SSR, render teleported content directly at target location.

### Target Existence

**Without defer:**
```html
<!-- ❌ Will fail immediately if #portal doesn't exist -->
<div $:teleport:to="#portal">...</div>
```

**With defer:**
```html
<!-- ✅ Waits for #portal to appear -->
<div $:teleport:to.defer="#portal">...</div>
```

Use `defer` when target might not exist at mount time, or ensure portal roots exist:

```js
// Create portal root on app init
if (!document.getElementById('modal-root')) {
  const root = document.createElement('div');
  root.id = 'modal-root';
  document.body.appendChild(root);
}
```

### One-Way Move

Teleported elements don't "return" to original location. They stay at target until destroyed or re-teleported.

### Arg Form is Non-Reactive

Arg form treats target as literal string, not reactive:

```html
<!-- ❌ Won't react to state changes -->
<div $:teleport:to="state.target">...</div>

<!-- ✅ Use object form for reactive target -->
<div $:teleport="state.target">...</div>
```

## Performance

Teleporting is a **pure DOM move**:
- No re-rendering
- No rebinding
- Listeners stay attached
- State preserved
- Very fast

**Cost:** Negligible. Moving DOM nodes is a cheap operation.

## Debugging

**Check teleport target:**
```js
console.log(document.querySelector('#modal-root'));
```

**Inspect teleported elements:**
```js
// Elements have data-jqr attribute
document.querySelectorAll('[data-jqr]');
```

**Dev mode warnings:**
- Missing target: Console error with selector
- Invalid mode: Falls back to `append`

## Next Steps

- [Transitions](./transitions.md) - Animate teleported elements
- [Slots](./slots.md) - Content projection patterns
- [Component Options](./component-options.md) - Full component API

