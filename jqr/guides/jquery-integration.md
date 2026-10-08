<!-- {
  "path": "jqr/guides/jquery-integration",
  "name": "docs-jqr-guides-jquery-integration",
  "meta": {
    "title": "jQuery Integration",
    "order": 17
  }
} -->

# jQuery Integration

JQR works with or without jQuery. It includes a minimal `$` fallback for basic DOM operations, but automatically uses full jQuery when available.

## How It Works

JQR detects jQuery at runtime and switches automatically:

1. **If jQuery is loaded** → Uses full jQuery (all features available)
2. **If jQuery is NOT loaded** → Uses minimal fallback (basic operations only)

**Detection is lazy:** JQR checks for jQuery on first `$` access, so script order matters.

## Using Full jQuery

Load jQuery **before** JQR to get full jQuery features:

```html
<!-- ✅ Correct order: jQuery first -->
<script src="https://code.jquery.com/jquery-4.0.0.min.js"></script>
<script src="https://cdn.jsdelivr.net/npm/@en32/jqr@latest/dist/jqr.min.js"></script>

<script>
  // Now using full jQuery
  $('#app').fadeIn().css('color', 'red');
  
  $.ajax({ url: '/api/data' });
  
  $('button').on('click', function() {
    $(this).addClass('active').siblings().removeClass('active');
  });
</script>
```

### With Module Bundlers

```bash
npm install jquery @en32/jqr
```

```js
// Import jQuery first
import jQuery from 'jquery';
window.$ = window.jQuery = jQuery;

// Then import JQR
import '@en32/jqr';

// JQR now uses full jQuery
$('#app').component({
  state: { message: 'Hello' }
});
```

## Using Fallback `$`

If jQuery isn't loaded, JQR uses its minimal fallback:

```html
<!-- No jQuery, just JQR -->
<script src="https://cdn.jsdelivr.net/npm/@en32/jqr@latest/dist/jqr.min.js"></script>

<script>
  // Fallback $ works for basic operations
  $('#app').component({
    state: { count: 0 }
  });
  
  // Basic DOM operations work
  $('#output').text('Hello');
  $('#box').toggle();
  
  // ❌ But advanced jQuery features are NOT available
  // $('#app').fadeIn();  // Error: fadeIn is not a function
  // $.ajax();            // Error: ajax is not a function
</script>
```

## Fallback Features

The minimal `$` supports **only what JQR needs internally**:

### Supported Methods

| Method | Description | Example |
|--------|-------------|---------|
| `$(selector)` | Select elements | `$('.items')` |
| `.find(sel)` | Find descendants | `$el.find('input')` |
| `.on(event, fn)` | Attach event | `$el.on('click', fn)` |
| `.on(event, sel, fn)` | Delegated event | `$el.on('click', '.btn', fn)` |
| `.off(event, fn)` | Remove event | `$el.off('click', fn)` |
| `.trigger(event, data)` | Trigger event | `$el.trigger('change')` |
| `.data(key, val)` | Get/set data | `$el.data('id', 123)` |
| `.removeData(key)` | Remove data | `$el.removeData('id')` |
| `.text(val?)` | Get/set text | `$el.text('Hello')` |
| `.html(val?)` | Get/set HTML | `$el.html('<b>Hi</b>')` |
| `.attr(name, val?)` | Get/set attribute | `$el.attr('href', '/home')` |
| `.toggle(show?)` | Toggle visibility | `$el.toggle(true)` |
| `.get(i?)` | Get DOM element(s) | `$el.get(0)` |
| `.each(fn)` | Iterate elements | `$el.each((i, el) => {})` |

### NOT Supported

Everything else is NOT available in fallback mode:

- ❌ Animations: `.fadeIn()`, `.slideDown()`, `.animate()`
- ❌ AJAX: `$.ajax()`, `$.get()`, `$.post()`
- ❌ Effects: `.show()`, `.hide()` (use `.toggle()` instead)
- ❌ Traversal: `.parent()`, `.siblings()`, `.closest()`
- ❌ Manipulation: `.append()`, `.remove()`, `.empty()`
- ❌ CSS: `.css()`, `.addClass()`, `.removeClass()`
- ❌ Utilities: `$.each()`, `$.map()`, `$.extend()`
- ❌ Deferred/Promise: `$.Deferred()`, `$.when()`
- ❌ 1000+ other jQuery methods

## Checking Which `$` is Active

Use `$.jqr.isUsingFallback()` to detect:

```js
if ($.jqr.isUsingFallback()) {
  console.log('Using JQR fallback - basic operations only');
} else {
  console.log('Using full jQuery - all features available');
}
```

### Conditional Feature Usage

```js
$('#app').component({
  mounted(state, $el) {
    if ($.jqr.isUsingFallback()) {
      // Fallback: Use native DOM
      this.el.style.opacity = '0';
      setTimeout(() => {
        this.el.style.transition = 'opacity 0.3s';
        this.el.style.opacity = '1';
      }, 10);
    } else {
      // jQuery: Use animations
      $el.fadeIn(300);
    }
  }
});
```

### Feature Detection Pattern

```js
function safeAnimate($el, show) {
  if ($.jqr.isUsingFallback()) {
    // Fallback: Instant toggle
    $el.toggle(show);
  } else {
    // jQuery: Animated
    show ? $el.fadeIn() : $el.fadeOut();
  }
}
```

## When to Use Full jQuery

### Use Full jQuery When:

✅ **You need animations**
```js
$('.modal').fadeIn();
$('.notification').slideDown();
```

✅ **You need AJAX helpers**
```js
$.get('/api/data', (data) => {
  state.items = data;
});
```

✅ **You need advanced selectors**
```js
$('div:visible:first');
$('input[type="text"]:enabled');
```

✅ **You need traversal**
```js
$(event.target).closest('.card').find('.title').text();
```

✅ **You need manipulation**
```js
$('.list').append('<li>New item</li>');
```

✅ **Your app already uses jQuery**
```js
// Existing jQuery code
$('.legacy-widget').doSomething();
```

### Use Fallback When:

✅ **You only use JQR directives**
```html
<div $:text="message"></div>
<input $:model="query">
<button $:click="submit">Submit</button>
```

✅ **You want minimal bundle size**
- JQR: ~15KB minified + gzipped
- JQR + jQuery: ~45KB minified + gzipped

✅ **Modern browsers only (no IE support needed)**

✅ **You use modern browser APIs**
```js
// Use Fetch API instead of $.ajax
async loadData() {
  const response = await fetch('/api/data');
  return response.json();
}
```

## Transition Engine and jQuery

JQR's transition system can use jQuery animations when configured:

```js
// Requires full jQuery
$.jqr.config({ transitionEngine: 'jquery' });
```

**If jQuery is NOT loaded:**
- Falls back to native CSS transitions
- Logs dev warning (if `devMode: true`)

```js
if ($.jqr.isUsingFallback()) {
  console.warn('jQuery transition engine requested but jQuery not loaded');
  // Automatically falls back to native engine
}
```

See [Transitions Guide](./transitions.md) for details.

## Teleport and jQuery

The `$:teleport` directive uses jQuery's DOM manipulation when available:

```html
<div $:teleport:append="'#modals'">
  <div class="modal">...</div>
</div>
```

**With jQuery:**
```js
$(el).appendTo(target); // Uses jQuery
```

**With fallback:**
```js
targetEl.appendChild(el); // Uses native DOM
```

Both work identically from the user's perspective.

## Best Practices

### 1. Decide Early

Choose your approach at project start:

**Option A: jQuery-free**
```html
<!-- Minimal: JQR only -->
<script src="jqr.min.js"></script>
```

**Option B: With jQuery**
```html
<!-- Full features -->
<script src="jquery.min.js"></script>
<script src="jqr.min.js"></script>
```

### 2. Don't Mix Detection

Avoid checking `$.jqr.isUsingFallback()` everywhere - make it a project-wide decision:

```js
// ❌ Bad: Checking in every component
mounted() {
  if ($.jqr.isUsingFallback()) {
    // ...
  } else {
    // ...
  }
}

// ✅ Good: One-time decision at app start
if ($.jqr.isUsingFallback()) {
  console.warn('jQuery not loaded - animations disabled');
  APP_CONFIG.enableAnimations = false;
}
```

### 3. Use Modern APIs

If not using jQuery, prefer modern browser APIs:

```js
// ✅ Use Fetch instead of $.ajax
async asyncState: {
  async users() {
    const response = await fetch('/api/users');
    return response.json();
  }
}

// ✅ Use native DOM instead of jQuery traversal
methods: {
  findButton(state) {
    const button = this.el.querySelector('.submit-btn');
    button.disabled = !state.isValid;
  }
}
```

### 4. Polyfills for Old Browsers

If you need IE11 support without jQuery:

```html
<!-- Polyfills for old browsers -->
<script src="https://polyfill.io/v3/polyfill.min.js?features=Promise,fetch"></script>
<script src="jqr.min.js"></script>
```

Or just use jQuery:

```html
<!-- jQuery supports old browsers -->
<script src="jquery.min.js"></script>
<script src="jqr.min.js"></script>
```

## Common Issues

### Issue: Features Not Available

**Problem:**
```js
$('#app').fadeIn();
// Error: fadeIn is not a function
```

**Solution:**
```html
<!-- Load jQuery before JQR -->
<script src="jquery.min.js"></script>
<script src="jqr.min.js"></script>
```

### Issue: Wrong jQuery Version

**Problem:**
```js
// Using jQuery 1.x or 2.x
```

**Solution:**
```html
<!-- Use jQuery 3.x or 4.x -->
<script src="https://code.jquery.com/jquery-4.0.0.min.js"></script>
```

JQR is tested with jQuery 3+ and 4+.

### Issue: Script Order

**Problem:**
```html
<!-- ❌ Wrong order -->
<script src="jqr.min.js"></script>
<script src="jquery.min.js"></script>
```

**Solution:**
```html
<!-- ✅ Correct order -->
<script src="jquery.min.js"></script>
<script src="jqr.min.js"></script>
```

JQR detects jQuery lazily, but jQuery must be loaded first.

### Issue: Multiple jQuery Versions

**Problem:**
```html
<script src="jquery-3.7.1.min.js"></script>
<script src="jquery-4.0.0.min.js"></script>
<script src="jqr.min.js"></script>
```

**Solution:**
Use only one jQuery version. JQR uses whichever is in `window.jQuery`.

## Bundle Size Comparison

| Setup | Minified | Gzipped | Notes |
|-------|----------|---------|-------|
| JQR only | ~25KB | ~9KB | Fallback `$` included |
| JQR + jQuery 4 | ~95KB | ~33KB | Full features |
| JQR + jQuery 3 | ~105KB | ~35KB | Full features |

**Recommendation:**
- Small projects / landing pages: JQR only
- Apps with animations / AJAX: JQR + jQuery

## TypeScript Support

Types adapt to jQuery presence:

```typescript
import { $ } from '@en32/jqr';

// Type includes jQuery methods if jQuery types are installed
import '@types/jquery';

$('#app').fadeIn(); // ✅ Type-checked if jQuery types present

// Without jQuery types
$('#app').fadeIn(); // ⚠️ Type error (method doesn't exist on fallback)
```

## Migration Path

### From jQuery to Fallback

If you want to remove jQuery dependency:

1. **Search for jQuery usage:**
```bash
grep -r "\.fadeIn\|\.ajax\|\.slideDown" src/
```

2. **Replace with native APIs:**
```js
// Before
$el.fadeIn();

// After
el.style.opacity = '0';
el.style.display = 'block';
el.offsetHeight; // Force reflow
el.style.transition = 'opacity 0.3s';
el.style.opacity = '1';
```

3. **Test without jQuery:**
```html
<!-- Remove jQuery -->
<!-- <script src="jquery.min.js"></script> -->
<script src="jqr.min.js"></script>
```

4. **Check for errors:**
```js
if ($.jqr.isUsingFallback()) {
  console.log('✅ Successfully using fallback');
}
```

### From Fallback to jQuery

If you need more features:

1. **Add jQuery:**
```html
<script src="https://code.jquery.com/jquery-4.0.0.min.js"></script>
<script src="jqr.min.js"></script>
```

2. **No code changes needed** - JQR auto-detects jQuery

3. **Enable jQuery features:**
```js
// Now available:
$.ajax({ url: '/api/data' });
$('.modal').fadeIn();
```

## See Also

- [Installation](../installation.md) - Setup options
- [Transitions](./transitions.md) - Animation system
- [Teleport](./teleport.md) - DOM portal patterns
- [Configuration Reference](../reference/config-reference.md) - All APIs
