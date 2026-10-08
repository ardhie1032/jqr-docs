<!-- {
  "path": "jqr/guides/transitions",
  "name": "docs-jqr-guides-transitions",
  "meta": {
    "title": "Transitions & Animations",
    "order": 9
  }
} -->

# Transitions & Animations

JQR provides declarative transitions for elements that enter or leave the DOM. Add smooth animations to `$:if` conditionals without manual animation code.

## Basic Transitions

Use `$:transition` with `$:if` to animate show/hide:

```html live:no-code
<div id="basic-transition" class="en-card">
  <div class="en-card__body">
    <h3>Basic Transition</h3>
    <div class="en-flex en-gap-2" style="margin-bottom: 1rem;">
      <button class="en-btn en-btn--primary" $:click="show = !show" $:text="show ? 'Hide' : 'Show'"></button>
    </div>
    <div $:if="show" $:transition="'fade'" class="en-alert en-alert--success">
      <strong>Hello!</strong> This box fades in and out.
    </div>
  </div>
</div>

<style>
.fade-enter-from,
.fade-leave-to {
  opacity: 0;
}
.fade-enter-active,
.fade-leave-active {
  transition: opacity 0.3s ease;
}
</style>
```

```html
<div id="basic-transition">
  <button $:click="show = !show">Toggle</button>
  <div $:if="show" $:transition="'fade'">
    <strong>Hello!</strong> This box fades in and out.
  </div>
</div>

<style>
.fade-enter-from,
.fade-leave-to {
  opacity: 0;
}
.fade-enter-active,
.fade-leave-active {
  transition: opacity 0.3s ease;
}
</style>
```

```js inject show
$('#basic-transition').component({
  state: { show: true }
});
```

```js cleanup
onUnmount(() => {
  $('#basic-transition').component('destroy');
})
```

**How it works:**
1. Element with `$:if` becomes `true` → enter transition plays
2. Element with `$:if` becomes `false` → leave transition plays
3. CSS classes are applied/removed automatically

## Transition Classes

JQR applies these CSS classes during transitions:

### Enter Transition

```
1. Element created, not yet in DOM
   ↓
2. Add: fade-enter-from + fade-enter-active
   Insert element into DOM
   ↓
3. Force reflow (next frame)
   ↓
4. Remove: fade-enter-from
   ↓
5. Wait for transition/animation end
   ↓
6. Remove: fade-enter-active
   Done!
```

**Classes:**
- `{name}-enter-from` - Starting state (e.g., `opacity: 0`)
- `{name}-enter-active` - Active state (defines transition duration/easing)

### Leave Transition

```
1. Element in DOM, $:if becomes false
   ↓
2. Add: fade-leave-active + fade-leave-to
   ↓
3. Force reflow (next frame)
   ↓
4. Wait for transition/animation end
   ↓
5. Remove element from DOM
   Done!
```

**Classes:**
- `{name}-leave-active` - Active state (defines transition)
- `{name}-leave-to` - Ending state (e.g., `opacity: 0`)

## CSS Transitions

Define transitions with CSS `transition` property:

### Fade

```html live:no-code
<div id="fade-demo" class="en-card">
  <div class="en-card__body">
    <h3>Fade Transition</h3>
    <button class="en-btn en-btn--primary" $:click="visible = !visible">Toggle</button>
    <div $:if="visible" $:transition="'fade'" class="en-alert en-alert--info" style="margin-top: 1rem;">
      Fading in and out
    </div>
  </div>
</div>

<style>
.fade-enter-from, .fade-leave-to { opacity: 0; }
.fade-enter-active, .fade-leave-active { transition: opacity 0.3s ease; }
</style>
```

```html
<div id="fade-demo">
  <button $:click="visible = !visible">Toggle</button>
  <div $:if="visible" $:transition="'fade'">
    Fading in and out
  </div>
</div>
```

```css
.fade-enter-from,
.fade-leave-to {
  opacity: 0;
}

.fade-enter-active,
.fade-leave-active {
  transition: opacity 0.3s ease;
}
```

```js inject show
$('#fade-demo').component({
  state: { visible: true }
});
```

```js cleanup
onUnmount(() => {
  $('#fade-demo').component('destroy');
})
```

### Slide

```html live:no-code
<div id="slide-demo" class="en-card">
  <div class="en-card__body">
    <h3>Slide Transition</h3>
    <button class="en-btn en-btn--primary" $:click="visible = !visible">Toggle</button>
    <div $:if="visible" $:transition="'slide'" class="en-alert en-alert--success" style="margin-top: 1rem; overflow: hidden;">
      Sliding in and out
    </div>
  </div>
</div>

<style>
.slide-enter-from, .slide-leave-to {
  max-height: 0;
  opacity: 0;
  transform: translateY(-20px);
}
.slide-enter-active, .slide-leave-active {
  transition: all 0.3s ease;
  max-height: 200px;
}
</style>
```

```html
<div id="slide-demo">
  <button $:click="visible = !visible">Toggle</button>
  <div $:if="visible" $:transition="'slide'">
    Sliding in and out
  </div>
</div>
```

```css
.slide-enter-from,
.slide-leave-to {
  max-height: 0;
  opacity: 0;
  transform: translateY(-20px);
}

.slide-enter-active,
.slide-leave-active {
  transition: all 0.3s ease;
  max-height: 200px;
}
```

```js inject show
$('#slide-demo').component({
  state: { visible: true }
});
```

```js cleanup
onUnmount(() => {
  $('#slide-demo').component('destroy');
})
```

### Scale

```html live:no-code
<div id="scale-demo" class="en-card">
  <div class="en-card__body">
    <h3>Scale Transition</h3>
    <button class="en-btn en-btn--primary" $:click="visible = !visible">Toggle</button>
    <div $:if="visible" $:transition="'scale'" class="en-alert en-alert--warning" style="margin-top: 1rem;">
      Scaling in and out
    </div>
  </div>
</div>

<style>
.scale-enter-from, .scale-leave-to {
  opacity: 0;
  transform: scale(0.8);
}
.scale-enter-active, .scale-leave-active {
  transition: all 0.3s cubic-bezier(0.34, 1.56, 0.64, 1);
}
</style>
```

```html
<div id="scale-demo">
  <button $:click="visible = !visible">Toggle</button>
  <div $:if="visible" $:transition="'scale'">
    Scaling in and out
  </div>
</div>
```

```css
.scale-enter-from,
.scale-leave-to {
  opacity: 0;
  transform: scale(0.8);
}

.scale-enter-active,
.scale-leave-active {
  transition: all 0.3s cubic-bezier(0.34, 1.56, 0.64, 1);
}
```

```js inject show
$('#scale-demo').component({
  state: { visible: true }
});
```

```js cleanup
onUnmount(() => {
  $('#scale-demo').component('destroy');
})
```

## CSS Animations

Use CSS `@keyframes` for complex animations:

```html live:no-code
<div id="bounce-demo" class="en-card">
  <div class="en-card__body">
    <h3>Bounce Animation</h3>
    <button class="en-btn en-btn--primary" $:click="visible = !visible">Toggle</button>
    <div $:if="visible" $:transition="'bounce'" class="en-alert en-alert--info" style="margin-top: 1rem;">
      Bouncing in and out
    </div>
  </div>
</div>

<style>
@keyframes bounceIn {
  0% { opacity: 0; transform: scale(0.3); }
  50% { opacity: 1; transform: scale(1.05); }
  70% { transform: scale(0.9); }
  100% { transform: scale(1); }
}

@keyframes bounceOut {
  0% { transform: scale(1); }
  50% { transform: scale(1.05); }
  100% { opacity: 0; transform: scale(0.3); }
}

.bounce-enter-active {
  animation: bounceIn 0.5s;
}

.bounce-leave-active {
  animation: bounceOut 0.5s;
}
</style>
```

```html
<div id="bounce-demo">
  <button $:click="visible = !visible">Toggle</button>
  <div $:if="visible" $:transition="'bounce'">
    Bouncing in and out
  </div>
</div>
```

```css
@keyframes bounceIn {
  0% { opacity: 0; transform: scale(0.3); }
  50% { opacity: 1; transform: scale(1.05); }
  70% { transform: scale(0.9); }
  100% { transform: scale(1); }
}

@keyframes bounceOut {
  0% { transform: scale(1); }
  50% { transform: scale(1.05); }
  100% { opacity: 0; transform: scale(0.3); }
}

.bounce-enter-active {
  animation: bounceIn 0.5s;
}

.bounce-leave-active {
  animation: bounceOut 0.5s;
}
```

```js inject show
$('#bounce-demo').component({
  state: { visible: true }
});
```

```js cleanup
onUnmount(() => {
  $('#bounce-demo').component('destroy');
})
```

## Transition Options

Pass an options object instead of a string:

```html
<div $:if="show" $:transition="{ name: 'fade', duration: 500 }">
  Content
</div>
```

```js
$('#app').component({
  state: {
    show: true,
    transitionOpts: {
      name: 'fade',
      duration: 500  // milliseconds
    }
  }
});
```

### Available Options

```js
{
  name: 'fade',          // CSS class prefix
  duration: 300,         // Explicit duration (ms)
  appear: false,         // Play transition on initial mount
  enter: 'fadeIn',       // jQuery method for enter (jQuery engine only)
  leave: 'fadeOut'       // jQuery method for leave (jQuery engine only)
}
```

### Duration

Explicit duration as fallback:

```html
<div $:if="show" $:transition="{ name: 'custom', duration: 1000 }">
  Content
</div>
```

**When duration is used:**
- Fallback if CSS transition/animation not detected
- Ensures animation completes even without `transitionend` event

### Appear Transition

Play transition on initial mount:

```html live:no-code
<div id="appear-demo" class="en-card">
  <div class="en-card__body">
    <h3>Appear Transition</h3>
    <button class="en-btn en-btn--primary" $:click="remount">Remount Component</button>
    <div style="margin-top: 1rem;">
      <div $:if="mounted" $:transition="{ name: 'fade', appear: true }" class="en-alert en-alert--success">
        This fades in on mount!
      </div>
    </div>
  </div>
</div>

<style>
.fade-enter-from, .fade-leave-to { opacity: 0; }
.fade-enter-active, .fade-leave-active { transition: opacity 0.5s ease; }
</style>
```

```html
<div id="appear-demo">
  <button $:click="remount">Remount</button>
  <div $:if="mounted" $:transition="{ name: 'fade', appear: true }">
    This fades in on mount!
  </div>
</div>
```

```js inject show
const appearDemo = $('#appear-demo').component({
  state: { mounted: true },
  methods: {
    remount(state) {
      state.mounted = false;
      setTimeout(() => state.mounted = true, 50);
    }
  }
});
```

```js cleanup
onUnmount(() => {
  $('#appear-demo').component('destroy');
})
```

## jQuery Engine (Optional)

Use jQuery animation methods instead of CSS:

```js
// Configure globally
$.jqr.config({ transitionEngine: 'jquery' });
```

```html
<div $:if="show" $:transition="{ enter: 'fadeIn', leave: 'fadeOut', duration: 300 }">
  Content
</div>
```

**jQuery methods:**
- `fadeIn` / `fadeOut`
- `slideDown` / `slideUp`
- `show` / `hide` (with duration)

**Note:** Requires jQuery 4.0+. Falls back to native engine if jQuery isn't loaded.

## Multiple Transitions

Different transitions for different elements:

```html live:no-code
<div id="multi-transition" class="en-card">
  <div class="en-card__body">
    <h3>Multiple Transitions</h3>
    <div class="en-flex en-gap-2" style="margin-bottom: 1rem;">
      <button class="en-btn en-btn--primary" $:click="tab = 'fade'">Fade</button>
      <button class="en-btn en-btn--primary" $:click="tab = 'slide'">Slide</button>
      <button class="en-btn en-btn--primary" $:click="tab = 'scale'">Scale</button>
    </div>
    
    <div $:if="tab === 'fade'" $:transition="'fade'" class="en-alert en-alert--info">
      Fade transition
    </div>
    
    <div $:if="tab === 'slide'" $:transition="'slide'" class="en-alert en-alert--success">
      Slide transition
    </div>
    
    <div $:if="tab === 'scale'" $:transition="'scale'" class="en-alert en-alert--warning">
      Scale transition
    </div>
  </div>
</div>

<style>
.fade-enter-from, .fade-leave-to { opacity: 0; }
.fade-enter-active, .fade-leave-active { transition: opacity 0.3s ease; }

.slide-enter-from, .slide-leave-to {
  opacity: 0;
  transform: translateY(-20px);
  max-height: 0;
}
.slide-enter-active, .slide-leave-active {
  transition: all 0.3s ease;
  max-height: 200px;
}

.scale-enter-from, .scale-leave-to {
  opacity: 0;
  transform: scale(0.8);
}
.scale-enter-active, .scale-leave-active {
  transition: all 0.3s cubic-bezier(0.34, 1.56, 0.64, 1);
}
</style>
```

```html
<div id="multi-transition">
  <button $:click="tab = 'fade'">Fade</button>
  <button $:click="tab = 'slide'">Slide</button>
  <button $:click="tab = 'scale'">Scale</button>
  
  <div $:if="tab === 'fade'" $:transition="'fade'">
    Fade transition
  </div>
  
  <div $:if="tab === 'slide'" $:transition="'slide'">
    Slide transition
  </div>
  
  <div $:if="tab === 'scale'" $:transition="'scale'">
    Scale transition
  </div>
</div>
```

```js inject show
$('#multi-transition').component({
  state: { tab: 'fade' }
});
```

```js cleanup
onUnmount(() => {
  $('#multi-transition').component('destroy');
})
```

## Modal Example

Complete modal with transition:

```html live:no-code
<div id="modal-demo" class="en-card">
  <div class="en-card__body">
    <h3>Modal with Transition</h3>
    <button class="en-btn en-btn--primary" $:click="showModal = true">
      Open Modal
    </button>
    
    <div $:if="showModal" $:transition="'modal'" class="modal-backdrop" $:click="showModal = false">
      <div class="modal-content en-card" $:click.stop="void 0">
        <div class="en-card__body">
          <h4>Modal Title</h4>
          <p>This modal has a smooth fade transition.</p>
          <button class="en-btn en-btn--primary" $:click="showModal = false">
            Close
          </button>
        </div>
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
  display: flex;
  align-items: center;
  justify-content: center;
  z-index: 1000;
}

.modal-content {
  max-width: 500px;
  width: 90%;
}

.modal-enter-from,
.modal-leave-to {
  opacity: 0;
}

.modal-enter-from .modal-content,
.modal-leave-to .modal-content {
  transform: scale(0.9) translateY(-20px);
}

.modal-enter-active,
.modal-leave-active {
  transition: opacity 0.3s ease;
}

.modal-enter-active .modal-content,
.modal-leave-active .modal-content {
  transition: transform 0.3s cubic-bezier(0.34, 1.56, 0.64, 1);
}
</style>
```

```html
<div id="modal-demo">
  <button $:click="showModal = true">Open Modal</button>
  
  <div $:if="showModal" $:transition="'modal'" class="modal-backdrop" $:click="showModal = false">
    <div class="modal-content" $:click.stop="void 0">
      <h4>Modal Title</h4>
      <p>This modal has a smooth fade transition.</p>
      <button $:click="showModal = false">Close</button>
    </div>
  </div>
</div>
```

```css
.modal-backdrop {
  position: fixed;
  top: 0; left: 0; right: 0; bottom: 0;
  background: rgba(0, 0, 0, 0.5);
  display: flex;
  align-items: center;
  justify-content: center;
}

.modal-enter-from,
.modal-leave-to {
  opacity: 0;
}

.modal-enter-from .modal-content,
.modal-leave-to .modal-content {
  transform: scale(0.9) translateY(-20px);
}

.modal-enter-active,
.modal-leave-active {
  transition: opacity 0.3s ease;
}

.modal-enter-active .modal-content,
.modal-leave-active .modal-content {
  transition: transform 0.3s cubic-bezier(0.34, 1.56, 0.64, 1);
}
```

```js inject show
$('#modal-demo').component({
  state: { showModal: false }
});
```

```js cleanup
onUnmount(() => {
  $('#modal-demo').component('destroy');
})
```

## Toast Notification

Auto-dismissing notification:

```html live:no-code
<div id="toast-demo" class="en-card">
  <div class="en-card__body">
    <h3>Toast Notification</h3>
    <button class="en-btn en-btn--primary" $:click="showToast">
      Show Toast
    </button>
    
    <div $:if="toastVisible" $:transition="'toast'" class="toast en-alert en-alert--success">
      ✓ Action completed successfully!
    </div>
  </div>
</div>

<style>
.toast {
  position: fixed;
  top: 20px;
  right: 20px;
  z-index: 1000;
  min-width: 250px;
}

.toast-enter-from,
.toast-leave-to {
  opacity: 0;
  transform: translateX(100px);
}

.toast-enter-active,
.toast-leave-active {
  transition: all 0.3s cubic-bezier(0.34, 1.56, 0.64, 1);
}
</style>
```

```html
<div id="toast-demo">
  <button $:click="showToast">Show Toast</button>
  
  <div $:if="toastVisible" $:transition="'toast'" class="toast">
    ✓ Action completed successfully!
  </div>
</div>
```

```css
.toast {
  position: fixed;
  top: 20px;
  right: 20px;
  z-index: 1000;
}

.toast-enter-from,
.toast-leave-to {
  opacity: 0;
  transform: translateX(100px);
}

.toast-enter-active,
.toast-leave-active {
  transition: all 0.3s cubic-bezier(0.34, 1.56, 0.64, 1);
}
```

```js inject show
const toastDemo = $('#toast-demo').component({
  state: { toastVisible: false },
  methods: {
    showToast(state) {
      state.toastVisible = true;
      setTimeout(() => {
        state.toastVisible = false;
      }, 3000);
    }
  }
});
```

```js cleanup
onUnmount(() => {
  $('#toast-demo').component('destroy');
})
```

## Best Practices

### 1. Name Your Transitions

```css
/* ✅ Good - clear naming */
.fade-enter-from { opacity: 0; }
.fade-enter-active { transition: opacity 0.3s; }

.slide-enter-from { transform: translateY(-20px); }
.slide-enter-active { transition: transform 0.3s; }
```

### 2. Match Enter/Leave Durations

```css
/* ✅ Good - consistent timing */
.fade-enter-active,
.fade-leave-active {
  transition: opacity 0.3s ease;
}

/* ❌ Inconsistent */
.fade-enter-active { transition: opacity 0.3s; }
.fade-leave-active { transition: opacity 0.6s; }
```

### 3. Use Hardware Acceleration

```css
/* ✅ Good - GPU accelerated */
.slide-enter-from {
  transform: translateY(-20px);
}

/* ❌ Avoid - CPU heavy */
.slide-enter-from {
  margin-top: -20px;
}
```

### 4. Set Explicit Duration for Animations

```html
<!-- ✅ Good - explicit duration -->
<div $:transition="{ name: 'bounce', duration: 500 }"></div>

<!-- ⚠️ May timeout early without explicit duration -->
<div $:transition="'bounce'"></div>
```

### 5. Combine with Other Directives

```html
<!-- ✅ Works together -->
<div 
  $:if="show" 
  $:transition="'fade'"
  $:class="{ 'highlighted': isActive }">
  Content
</div>
```

## Performance Tips

### Use `will-change`

Hint browser about upcoming animations:

```css
.modal-content {
  will-change: transform, opacity;
}
```

### Prefer Transform/Opacity

Most performant properties:

```css
/* ✅ Fast */
.transition-active {
  transition: opacity 0.3s, transform 0.3s;
}

/* ❌ Slow */
.transition-active {
  transition: width 0.3s, height 0.3s, top 0.3s;
}
```

### Reduce Motion

Respect user preferences:

```css
@media (prefers-reduced-motion: reduce) {
  .fade-enter-active,
  .fade-leave-active {
    transition-duration: 0.01ms !important;
  }
}
```

## Limitations

### Only Works with `$:if`

Transitions currently only work with `$:if` conditionals:

```html
<!-- ✅ Works -->
<div $:if="show" $:transition="'fade'"></div>

<!-- ❌ Doesn't work yet -->
<div $:for="item in items" $:transition="'fade'"></div>
<div $:show="visible" $:transition="'fade'"></div>
```

### No Nested Transitions

Only the outermost `$:if` transition applies:

```html
<!-- Only outer transition plays -->
<div $:if="show" $:transition="'fade'">
  <div $:if="innerShow" $:transition="'slide'">
    Inner content
  </div>
</div>
```

## Next Steps

- [Lists and Conditionals](./lists-and-conditionals.md) - Dynamic rendering
- [Lifecycle](./lifecycle.md) - Component lifecycle hooks
- [Teleport](./teleport.md) - Render outside hierarchy

