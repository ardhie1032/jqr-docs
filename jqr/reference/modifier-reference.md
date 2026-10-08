<!-- {
  "path": "jqr/reference/modifier-reference",
  "name": "docs-jqr-reference-modifier-reference",
  "meta": {
    "title": "Modifier Reference",
    "order": 2
  }
} -->

# Modifier Reference

Complete API reference for all built-in JQR modifiers. Modifiers customize directive behavior and are added with dot notation.

## Modifier Syntax

```html
<element $:directive.modifier="expression"></element>
<element $:directive.modifier1.modifier2.modifier3="expression"></element>
```

Multiple modifiers can be chained and are processed in order (left to right).

---

## Model Modifiers

Used with `$:model` directive for form input binding.

### `.trim`

Trims whitespace from input value.

**Usage:**
```html
<input $:model.trim="username">
```

**Behavior:**
- Removes leading/trailing whitespace
- Updates on `input` event (real-time)

**Example:**
```html
<input $:model.trim="username" placeholder="Enter username">
<!-- "  john  " becomes "john" -->
```

**See:** [Model Modifiers Guide](../guides/model-modifiers.md#trim)

---

### `.number`

Parses input value as a number.

**Usage:**
```html
<input $:model.number="age" type="number">
```

**Behavior:**
- Converts string to number using `parseFloat()`
- Returns `NaN` if conversion fails
- Updates on `input` event

**Example:**
```html
<input $:model.number="price" type="number" step="0.01">
<!-- "123.45" becomes 123.45 (number) -->
```

**Combined:**
```html
<input $:model.trim.number="amount">
<!-- "  42  " becomes 42 (number) -->
```

**See:** [Model Modifiers Guide](../guides/model-modifiers.md#number)

---

### `.lazy`

Updates model on `change` event instead of `input`.

**Usage:**
```html
<input $:model.lazy="description">
```

**Behavior:**
- Updates only when input loses focus or Enter is pressed
- Reduces update frequency
- Better performance for expensive operations

**Example:**
```html
<input $:model.lazy="searchQuery">
<!-- Updates on blur or Enter, not on every keystroke -->
```

**Use Case:** Search fields, expensive computed values

**See:** [Model Modifiers Guide](../guides/model-modifiers.md#lazy)

---

### `.debounce` / `.debounce.N`

Delays model update until user stops typing.

**Usage:**
```html
<input $:model.debounce="query">
<input $:model.debounce.500="query">
```

**Behavior:**
- Default delay: 300ms
- Custom delay: `.debounce.N` (N = milliseconds)
- Resets timer on each keystroke
- Updates only after delay period of inactivity

**Examples:**
```html
<!-- 300ms delay (default) -->
<input $:model.debounce="searchQuery">

<!-- 500ms delay -->
<input $:model.debounce.500="searchQuery">

<!-- 1000ms delay -->
<input $:model.debounce.1000="expensiveOperation">
```

**Use Case:**
- Autocomplete/search (reduce API calls)
- Expensive computations
- Real-time validation

**See:** [Model Modifiers Guide](../guides/model-modifiers.md#debounce)

---

### `.throttle` / `.throttle.N`

Limits model update frequency (allows one update per interval).

**Usage:**
```html
<input $:model.throttle="value">
<input $:model.throttle.200="value">
```

**Behavior:**
- Default interval: 300ms
- Custom interval: `.throttle.N` (N = milliseconds)
- Allows first update immediately
- Blocks subsequent updates within interval
- Processes last value after interval

**Examples:**
```html
<!-- 300ms throttle (default) -->
<input $:model.throttle="scrollPosition">

<!-- 200ms throttle -->
<input $:model.throttle.200="mouseX">

<!-- 1000ms throttle -->
<input $:model.throttle.1000="heavyCalc">
```

**Debounce vs Throttle:**
- **Debounce:** Waits for silence (good for search)
- **Throttle:** Regular intervals (good for scroll/resize)

**Use Case:**
- Scroll position tracking
- Mouse movement tracking
- Window resize handlers

**See:** [Model Modifiers Guide](../guides/model-modifiers.md#throttle)

---

### Model Modifier Combinations

Modifiers can be combined and are applied in left-to-right order:

```html
<!-- Trim then parse as number -->
<input $:model.trim.number="price">

<!-- Parse as number, update lazily -->
<input $:model.number.lazy="quantity">

<!-- Trim, parse, debounce -->
<input $:model.trim.number.debounce.500="amount">

<!-- Trim, lazy update -->
<input $:model.trim.lazy="notes">
```

**Processing Order:**
1. Input event fires
2. `.trim` - Remove whitespace
3. `.number` - Parse to number
4. `.lazy` / `.debounce` / `.throttle` - Control timing
5. State updated

---

## Event Modifiers

Used with event directives (`$:click`, `$:submit`, `$:on:*`).

### `.prevent`

Calls `event.preventDefault()` automatically.

**Usage:**
```html
<form $:submit.prevent="onSubmit()">
<a $:click.prevent="handleLink()">
```

**Behavior:**
- Prevents default browser action
- Form: Prevents page reload
- Link: Prevents navigation

**Examples:**
```html
<!-- Prevent form submission reload -->
<form $:submit.prevent="handleSubmit()">
  <button type="submit">Submit</button>
</form>

<!-- Prevent link navigation -->
<a href="#" $:click.prevent="showModal()">Open Modal</a>

<!-- Prevent context menu -->
<div $:on:contextmenu.prevent="customMenu()">Right-click me</div>
```

**See:** [Events Guide](../guides/events.md#event-modifiers)

---

### `.stop`

Calls `event.stopPropagation()` automatically.

**Usage:**
```html
<div $:click="outer()">
  <button $:click.stop="inner()">Click</button>
</div>
```

**Behavior:**
- Stops event from bubbling up
- Parent handlers won't trigger

**Examples:**
```html
<!-- Prevent event bubbling -->
<div $:click="handleOuter()">
  <button $:click.stop="handleInner()">
    Only inner handler fires
  </button>
</div>

<!-- Stop propagation on specific events -->
<div $:on:mousedown.stop="handleMousedown()">
  Mousedown won't bubble
</div>
```

**See:** [Events Guide](../guides/events.md#stop-propagation)

---

### `.self`

Only trigger handler if event target is the element itself (not children).

**Usage:**
```html
<div $:click.self="handleSelf()">
  <button>Won't trigger</button>
</div>
```

**Behavior:**
- Checks `event.target === element`
- Ignores events from child elements

**Example:**
```html
<div class="modal-overlay" $:click.self="closeModal()">
  <div class="modal-content">
    <!-- Clicking here won't close modal -->
    <button>Cancel</button>
  </div>
</div>
<!-- Only clicking the overlay closes modal -->
```

**Use Case:** Modal overlays, dropdowns

**See:** [Events Guide](../guides/events.md#self-target)

---

### `.once`

Handler only triggers once, then removes itself.

**Usage:**
```html
<button $:click.once="initialize()">
```

**Behavior:**
- Fires only on first event
- Automatically unregisters after first trigger

**Examples:**
```html
<!-- One-time initialization -->
<button $:click.once="init()">Initialize</button>

<!-- Show tip once -->
<div $:on:mouseenter.once="showTip()">Hover for tip</div>

<!-- Track first click -->
<button $:click.once="trackFirstClick()">Track Me</button>
```

**Use Case:** Initialization, one-time tips, analytics

**See:** [Events Guide](../guides/events.md#once-modifier)

---

### `.capture`

Uses capture phase for event listener instead of bubble phase.

**Usage:**
```html
<div $:click.capture="handleCapture()">
```

**Behavior:**
- Event fires during capture phase (top-down)
- Before regular handlers (bottom-up)

**Example:**
```html
<div $:click.capture="outer()">
  <div $:click="middle()">
    <button $:click="inner()">Click</button>
  </div>
</div>
<!-- Order: outer (capture) → inner → middle → outer (bubble) -->
```

**Use Case:** Intercept events before children handle them

**See:** [Events Guide](../guides/events.md#capture-phase)

---

### `.passive`

Adds passive event listener (improves scroll performance).

**Usage:**
```html
<div $:on:scroll.passive="handleScroll()">
```

**Behavior:**
- Tells browser event won't call `preventDefault()`
- Allows browser to optimize scrolling
- Cannot use with `.prevent`

**Example:**
```html
<!-- Optimized scroll -->
<div $:on:scroll.passive="trackScroll()">
  Long scrollable content
</div>

<!-- Optimized touch -->
<div $:on:touchstart.passive="handleTouch()">
  Touch area
</div>
```

**Use Case:** Scroll/touch handlers that don't prevent default

**See:** [Events Guide](../guides/events.md#passive-listeners)

---

### Event Modifier Combinations

```html
<!-- Prevent default AND stop propagation -->
<a $:click.prevent.stop="handleClick()">

<!-- Self-target AND prevent default -->
<div $:click.self.prevent="handleOverlay()">

<!-- Once AND prevent default -->
<button $:click.once.prevent="initOnce()">

<!-- Capture AND stop -->
<div $:click.capture.stop="interceptClick()">
```

**Order doesn't affect `.prevent` or `.stop`, but affects `.self`, `.once`, `.capture`.**

---

## Keyboard Event Modifiers

Used with keyboard events (`$:keyup`, `$:keydown`, `$:keypress`).

### Key Aliases

**Common Keys:**

| Modifier | Key | Example |
|----------|-----|---------|
| `.enter` | Enter | `$:keyup.enter="submit()"` |
| `.tab` | Tab | `$:keydown.tab="nextField()"` |
| `.delete` | Delete/Backspace | `$:keyup.delete="remove()"` |
| `.esc` | Escape | `$:keyup.esc="close()"` |
| `.space` | Space | `$:keyup.space="toggle()"` |
| `.up` | Arrow Up | `$:keydown.up="moveUp()"` |
| `.down` | Arrow Down | `$:keydown.down="moveDown()"` |
| `.left` | Arrow Left | `$:keydown.left="moveLeft()"` |
| `.right` | Arrow Right | `$:keydown.right="moveRight()"` |

**Examples:**
```html
<!-- Submit on Enter -->
<input $:keyup.enter="submit()">

<!-- Close on Escape -->
<div $:keyup.esc="closeModal()">

<!-- Navigate with arrows -->
<div $:keydown.up="selectPrevious()"
     $:keydown.down="selectNext()">

<!-- Tab navigation -->
<input $:keydown.tab.prevent="handleTab()">
```

**See:** [Events Guide](../guides/events.md#keyboard-events)

---

### System Modifier Keys

**Modifier Keys:**

| Modifier | Key | Example |
|----------|-----|---------|
| `.ctrl` | Control | `$:keyup.ctrl.s="save()"` |
| `.alt` | Alt/Option | `$:keyup.alt.n="newItem()"` |
| `.shift` | Shift | `$:keyup.shift.enter="addLine()"` |
| `.meta` | Meta/Cmd/Win | `$:keyup.meta.k="search()"` |

**Examples:**
```html
<!-- Ctrl+S to save -->
<div $:keyup.ctrl.s.prevent="save()">

<!-- Ctrl+Enter to submit -->
<textarea $:keyup.ctrl.enter="submit()"></textarea>

<!-- Alt+N for new -->
<div $:keyup.alt.n="createNew()">

<!-- Cmd+K for search (Mac) -->
<div $:keyup.meta.k.prevent="openSearch()">
```

**Combination:**
```html
<!-- Ctrl+Shift+Delete -->
<div $:keyup.ctrl.shift.delete="deleteAll()">

<!-- Alt+Arrow keys -->
<div $:keydown.alt.up="moveUp()"
     $:keydown.alt.down="moveDown()">
```

**See:** [Events Guide](../guides/events.md#system-modifiers)

---

### Exact Modifier

**`.exact`** - Requires exact modifier combination (no extra keys).

**Usage:**
```html
<div $:click.ctrl.exact="handleCtrlClick()">
```

**Behavior:**
- `.ctrl` alone: Fires if Ctrl pressed (even with other modifiers)
- `.ctrl.exact`: Only fires if ONLY Ctrl pressed (no Shift, Alt, etc.)

**Example:**
```html
<!-- Fires with Ctrl, Ctrl+Shift, Ctrl+Alt, etc. -->
<button $:click.ctrl="handleCtrl()">

<!-- Fires ONLY with Ctrl (no other modifiers) -->
<button $:click.ctrl.exact="handleCtrlOnly()">

<!-- Fires ONLY when no modifiers pressed -->
<button $:click.exact="handlePlainClick()">
```

**See:** [Events Guide](../guides/events.md#exact-modifier)

---

## Mouse Button Modifiers

Used with mouse events to filter by button.

**Button Modifiers:**

| Modifier | Button | Example |
|----------|--------|---------|
| `.left` | Left (primary) | `$:click.left="handleLeft()"` |
| `.right` | Right (secondary) | `$:click.right="handleRight()"` |
| `.middle` | Middle (wheel) | `$:click.middle="handleMiddle()"` |

**Examples:**
```html
<!-- Left click only -->
<div $:click.left="selectItem()">

<!-- Right click (context menu) -->
<div $:click.right.prevent="showContextMenu()">

<!-- Middle click (open in new tab) -->
<a $:click.middle="openInNewTab()">

<!-- Different actions per button -->
<div $:mousedown.left="startDrag()"
     $:mousedown.right.prevent="showMenu()">
```

**See:** [Events Guide](../guides/events.md#mouse-buttons)

---

## Transition Modifiers

Used with `$:transition` directive (CSS transitions/animations).

### `.css`

Forces CSS-based transition detection.

**Usage:**
```html
<div $:transition.css="fade">
```

**Behavior:**
- Waits for CSS `transitionend` / `animationend` events
- Default when no duration specified

---

### `.duration`

Custom transition duration (overrides CSS).

**Usage:**
```html
<div $:transition.duration.500="fade">
```

**Behavior:**
- Specifies duration in milliseconds
- Doesn't wait for CSS events
- Forces exact timing

**Example:**
```html
<div $:transition.duration.300="slide">
<!-- Transition completes after 300ms, regardless of CSS -->
```

---

### `.mode`

Controls transition mode for entering/leaving elements.

**Values:**
- `out-in` - Leave first, then enter
- `in-out` - Enter first, then leave (default)

**Usage:**
```html
<div $:transition.mode.out-in="fade">
```

**Example:**
```html
<!-- Wait for old to leave before showing new -->
<div $:if="showA" $:transition.mode.out-in="fade">View A</div>
<div $:if="!showA" $:transition.mode.out-in="fade">View B</div>
```

**See:** [Transitions Guide](../guides/transitions.md)

---

## Teleport Modifiers

Used with `$:teleport` directive in arg form (`$:teleport:to[.modifiers]="target"`).

### `.defer`

Waits for target element to appear in DOM using `MutationObserver`.

**Usage:**
```html
<div $:teleport:to.defer="#late-target">...</div>
```

**Behavior:**
- Without defer: Target must exist now, fails immediately if missing
- With defer: Sets up observer, waits for target to appear
- Once target found: Teleports content and disconnects observer
- Auto cleanup: Observer stops if component destroys before target appears

**Examples:**
```html
<!-- Wait for conditional target -->
<div $:teleport:to.defer="#dynamic-root">
  Content waits for target
</div>

<!-- Wait for async-loaded target -->
<div $:teleport:to.defer="#async-portal">
  Waits for AJAX content
</div>

<!-- Combine with mode modifiers -->
<div $:teleport:to.defer.prepend="body">
  Defer + prepend mode
</div>
```

**Use Cases:**
- Target rendered by `$:if` conditionally
- Target comes from async data (`$:for` over fetched array)
- Target inside component that mounts later
- Target created by third-party library

**See:** [Teleport Guide](../guides/teleport.md#deferred-teleport)

---

### Mode Modifiers

Control insertion position relative to target element.

#### `.append` (default)

Appends content to end of target's children.

**Usage:**
```html
<div $:teleport:to.append="body">...</div>
<div $:teleport:to="body">...</div> <!-- Same, append is default -->
```

**Behavior:**
- Adds element as last child of target
- Default mode if no mode modifier specified

---

#### `.prepend`

Prepends content to start of target's children.

**Usage:**
```html
<div $:teleport:to.prepend="#modal-root">...</div>
```

**Behavior:**
- Adds element as first child of target

**Example:**
```html
<!-- Always at top of body -->
<div $:teleport:to.prepend="body" class="top-banner">
  Important notice
</div>
```

---

#### `.insert-after`

Inserts content after target element itself (as sibling).

**Usage:**
```html
<div $:teleport:to.insert-after="#header">...</div>
```

**Behavior:**
- Places element immediately after target in DOM
- Element becomes sibling, not child

**Example:**
```html
<!-- Insert banner after header -->
<div $:teleport:to.insert-after="#site-header" class="promo-banner">
  Special offer!
</div>
```

---

#### `.insert-before`

Inserts content before target element itself (as sibling).

**Usage:**
```html
<div $:teleport:to.insert-before="#footer">...</div>
```

**Behavior:**
- Places element immediately before target in DOM
- Element becomes sibling, not child

**Example:**
```html
<!-- Insert newsletter above footer -->
<div $:teleport:to.insert-before="#site-footer" class="newsletter">
  Subscribe now!
</div>
```

---

### Combining Teleport Modifiers

Defer can be combined with mode modifiers:

```html
<!-- Defer + prepend -->
<div $:teleport:to.defer.prepend="#portal">
  Waits for target, then prepends
</div>

<!-- Defer + insert-after -->
<div $:teleport:to.defer.insert-after="#dynamic-element">
  Waits, then inserts after
</div>

<!-- Defer + insert-before -->
<div $:teleport:to.defer.insert-before="#footer">
  Waits, then inserts before
</div>

<!-- Defer + append (explicit) -->
<div $:teleport:to.defer.append="body">
  Waits, then appends (same as just .defer)
</div>
```

**Order doesn't matter:**
```html
<!-- These are equivalent -->
<div $:teleport:to.defer.prepend="#target">...</div>
<div $:teleport:to.prepend.defer="#target">...</div>
```

---

### Teleport: Arg vs Object Form

**Arg form** (uses modifiers):
```html
<div $:teleport:to.defer.prepend="#portal">...</div>
```

**Object form** (no modifiers, uses object properties):
```html
<div $:teleport="{ to: '#portal', defer: true, mode: 'prepend' }">...</div>
```

**Key differences:**
- Arg form: Target is literal string (not reactive)
- Object form: Target evaluated as JS expression (can be reactive)
- Arg form: Cleaner syntax for static targets
- Object form: Required for dynamic/reactive targets

**See:** [Teleport Guide](../guides/teleport.md#two-syntax-forms)

---

## Custom Modifiers

Define custom modifiers for your directives:

```js
$.jqr.directive('tooltip', {
  bind(el, ctx) {
    const modifiers = ctx.modifiers;
    
    if (modifiers.top) {
      el.dataset.placement = 'top';
    } else if (modifiers.bottom) {
      el.dataset.placement = 'bottom';
    }
    
    if (modifiers.large) {
      el.dataset.size = 'large';
    }
    
    el.title = ctx.evaluate(ctx.expr);
  }
});
```

**Usage:**
```html
<button $:tooltip.top.large="helpText">Help</button>
```

**See:** [Custom Directives Guide](../guides/custom-directives.md)

---

## Modifier Chaining

Modifiers are applied left-to-right:

```html
<!-- Model: trim → number → debounce -->
<input $:model.trim.number.debounce.500="amount">

<!-- Events: prevent → stop → once -->
<button $:click.prevent.stop.once="init()">

<!-- Keyboard: ctrl → shift → enter → prevent -->
<input $:keyup.ctrl.shift.enter.prevent="submit()">
```

**Order Matters:**
```html
<!-- Debounce after trim/number transformation -->
<input $:model.trim.number.debounce="value">

<!-- Prevent first, then stop propagation -->
<a $:click.prevent.stop="handleLink()">
```

---

## Quick Reference Table

### Model Modifiers

| Modifier | Effect | Example |
|----------|--------|---------|
| `.trim` | Remove whitespace | `$:model.trim="name"` |
| `.number` | Parse as number | `$:model.number="age"` |
| `.lazy` | Update on change | `$:model.lazy="description"` |
| `.debounce[.N]` | Delay updates | `$:model.debounce.500="query"` |
| `.throttle[.N]` | Limit frequency | `$:model.throttle.200="scroll"` |

### Event Modifiers

| Modifier | Effect | Example |
|----------|--------|---------|
| `.prevent` | Prevent default | `$:submit.prevent="save()"` |
| `.stop` | Stop propagation | `$:click.stop="inner()"` |
| `.self` | Self target only | `$:click.self="overlay()"` |
| `.once` | Fire once | `$:click.once="init()"` |
| `.capture` | Capture phase | `$:click.capture="intercept()"` |
| `.passive` | Passive listener | `$:scroll.passive="track()"` |

### Key Modifiers

| Modifier | Key | Example |
|----------|-----|---------|
| `.enter` | Enter | `$:keyup.enter="submit()"` |
| `.tab` | Tab | `$:keydown.tab="next()"` |
| `.delete` | Delete | `$:keyup.delete="remove()"` |
| `.esc` | Escape | `$:keyup.esc="close()"` |
| `.space` | Space | `$:keyup.space="toggle()"` |
| `.up/.down/.left/.right` | Arrows | `$:keydown.up="moveUp()"` |

### System Modifiers

| Modifier | Key | Example |
|----------|-----|---------|
| `.ctrl` | Control | `$:keyup.ctrl.s="save()"` |
| `.alt` | Alt | `$:keyup.alt.n="new()"` |
| `.shift` | Shift | `$:click.shift="selectRange()"` |
| `.meta` | Cmd/Win | `$:keyup.meta.k="search()"` |
| `.exact` | Exact combo | `$:click.ctrl.exact="action()"` |

### Mouse Modifiers

| Modifier | Button | Example |
|----------|--------|---------|
| `.left` | Left button | `$:click.left="select()"` |
| `.right` | Right button | `$:click.right="menu()"` |
| `.middle` | Middle button | `$:click.middle="newTab()"` |

### Teleport Modifiers

| Modifier | Effect | Example |
|----------|--------|---------|
| `.defer` | Wait for target | `$:teleport:to.defer="#late"` |
| `.append` | Append to target (default) | `$:teleport:to.append="body"` |
| `.prepend` | Prepend to target | `$:teleport:to.prepend="#root"` |
| `.insert-after` | Insert after target | `$:teleport:to.insert-after="#header"` |
| `.insert-before` | Insert before target | `$:teleport:to.insert-before="#footer"` |

---

## Best Practices

### 1. Use Debounce for Search

```html
<!-- ✅ Good - reduces API calls -->
<input $:model.debounce.500="searchQuery">

<!-- ❌ Bad - updates on every keystroke -->
<input $:model="searchQuery">
```

### 2. Always Prevent Form Submit

```html
<!-- ✅ Good - prevents page reload -->
<form $:submit.prevent="handleSubmit()">

<!-- ❌ Bad - page reloads on submit -->
<form $:submit="handleSubmit()">
```

### 3. Trim User Input

```html
<!-- ✅ Good - clean data -->
<input $:model.trim="username">

<!-- ❌ Bad - stores whitespace -->
<input $:model="username">
```

### 4. Use Passive for Scroll

```html
<!-- ✅ Good - better performance -->
<div $:on:scroll.passive="handleScroll()">

<!-- ⚠️ Okay but slower -->
<div $:on:scroll="handleScroll()">
```

### 5. Combine Modifiers Wisely

```html
<!-- ✅ Good - clean number input -->
<input $:model.trim.number.lazy="amount">

<!-- ✅ Good - keyboard shortcut -->
<div $:keyup.ctrl.s.prevent="save()">

<!-- ❌ Overkill - too many modifiers -->
<input $:model.trim.number.lazy.debounce.500.throttle.200="value">
```

---

## Next Steps

- [Directive Reference](./directive-reference.md) - All directives
- [Model Modifiers Guide](../guides/model-modifiers.md) - Detailed examples
- [Events Guide](../guides/events.md) - Event handling patterns
- [Forms & Models](../guides/forms-and-models.md) - Form binding
