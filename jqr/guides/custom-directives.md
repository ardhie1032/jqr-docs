<!-- {
  "path": "jqr/guides/custom-directives",
  "name": "docs-jqr-guides-custom-directives",
  "meta": {
    "title": "Custom Directives",
    "order": 10
  }
} -->

# Custom Directives

While JQR provides many built-in directives (`$:text`, `$:if`, `$:model`, etc.), you can create your own custom directives to extend the framework with reusable DOM behaviors.

## What is a Custom Directive?

A custom directive is a named set of lifecycle hooks that run when an element with your directive is encountered:

```js
$.jqr.directive('tooltip', {
  bind(el, ctx) {
    // Runs once when directive is first found
  },
  update(el, ctx) {
    // Runs initially, then again when dependencies change
  },
  unbind(el, ctx) {
    // Runs when element is removed from DOM
  }
});
```

Once registered, use it like any built-in directive:

```html
<button $:tooltip="helpText">Help</button>
```

## Directive Lifecycle

### `bind(el, ctx)`

Called **once** when the directive is first encountered during component scanning.

**Use for:**
- Initial setup
- Event listener registration
- Creating DOM elements
- Any one-time initialization

```js
$.jqr.directive('focus', {
  bind(el, ctx) {
    el.focus();
  }
});
```

### `update(el, ctx)`

Called **initially** (right after `bind`), then **reactively re-runs** whenever a dependency accessed via `ctx.evaluate()` changes.

**Automatically tracked** — like `computed` or `watch`, reading state/props inside `update` registers dependencies.

**Use for:**
- Updating DOM based on reactive data
- Applying dynamic styles/classes
- Any logic that needs to re-run on state changes

```js
$.jqr.directive('color', {
  update(el, ctx) {
    const color = ctx.evaluate(ctx.expr);
    el.style.color = color; // Re-runs when color changes
  }
});
```

### `unbind(el, ctx)`

Called when the element is **removed from DOM** (e.g., parent destroyed, `$:if` becomes false, removed from `$:for` list).

**Use for:**
- Cleanup
- Removing event listeners
- Canceling timers/intervals
- Destroying third-party widgets

```js
$.jqr.directive('interval', {
  bind(el, ctx) {
    const id = setInterval(() => {
      console.log('tick');
    }, 1000);
    el._intervalId = id;
  },
  unbind(el, ctx) {
    clearInterval(el._intervalId);
  }
});
```

## Directive Context

The `ctx` parameter provides access to:

### `ctx.el` / `ctx.$el`

The element the directive is bound to:

```js
$.jqr.directive('example', {
  bind(el, ctx) {
    console.log(el);        // HTMLElement
    console.log(ctx.$el);   // jQuery object
    ctx.$el.addClass('example');
  }
});
```

### `ctx.expr`

The raw attribute value (unevaluated):

```html
<div $:tooltip="user.name"></div>
```

```js
$.jqr.directive('tooltip', {
  update(el, ctx) {
    console.log(ctx.expr); // "user.name" (string)
  }
});
```

### `ctx.evaluate(expr)`

Evaluates an expression against the current scope (state/props/computed/methods):

```js
$.jqr.directive('tooltip', {
  update(el, ctx) {
    const value = ctx.evaluate(ctx.expr);
    el.title = value;
  }
});
```

**Reading via `evaluate()` automatically tracks dependencies** — when the expression's value changes, `update` re-runs.

### `ctx.instance`

Access to the owning component instance:

```js
$.jqr.directive('log', {
  bind(el, ctx) {
    console.log('State:', ctx.instance.state);
    console.log('Props:', ctx.instance.props);
  }
});
```

### `ctx.modifiers`

Array of modifier strings:

```html
<div $:tooltip.top.red="message"></div>
```

```js
$.jqr.directive('tooltip', {
  update(el, ctx) {
    const position = ctx.modifiers.includes('top') ? 'top' : 'bottom';
    const color = ctx.modifiers.includes('red') ? 'red' : 'black';
    // Apply tooltip styling...
  }
});
```

### `ctx.arg`

Directive argument (if any):

```html
<div $:copy:href="url"></div>
```

```js
$.jqr.directive('copy', {
  update(el, ctx) {
    console.log(ctx.arg); // "href"
  }
});
```

### `ctx.prefix` / `ctx.name`

Directive metadata:

```js
$.jqr.directive('example', {
  bind(el, ctx) {
    console.log(ctx.prefix); // "$"
    console.log(ctx.name);   // "example"
  }
});
```

## Basic Examples

### Auto-Focus Input

```html live:no-code
<div id="demo-focus" class="en-card en-card__body">
  <div class="en-alert en-alert--info en-mb-3" $:if="!focused">
    Input will auto-focus when shown
  </div>
  <input class="en-input" $:if="focused" $:focus placeholder="I'm auto-focused!">
  <button class="en-btn en-btn--sm en-mt-2" $:click="focused = !focused">
    <span $:text="focused ? 'Hide' : 'Show'"></span> Input
  </button>
</div>
```

```html
<div id="demo-focus">
  <div $:if="!focused">Input will auto-focus when shown</div>
  <input $:if="focused" $:focus placeholder="I'm auto-focused!">
  <button $:click="focused = !focused">
    <span $:text="focused ? 'Hide' : 'Show'"></span> Input
  </button>
</div>
```

```js inject show
// Register directive
$.jqr.directive('focus', {
  bind(el, ctx) {
    el.focus();
  }
});

$('#demo-focus').component({
  state: { focused: false }
});
```

```js cleanup
onUnmount(() => {
  $('#demo-focus').component('destroy');
})
```

### Click Outside

```html live:no-code
<div id="demo-click-outside" class="en-card en-card__body">
  <div $:if="isOpen" $:click-outside="close()" class="en-card en-card--bordered" style="padding: 20px;">
    <p>Click outside this box to close</p>
    <p class="en-text--muted en-text-sm">Clicks: <span $:text="clicks"></span></p>
  </div>
  <button class="en-btn en-btn--sm en-mt-2" $:click="open()">
    Open Box
  </button>
</div>
```

```html
<div id="demo-click-outside">
  <div $:if="isOpen" $:click-outside="close()">
    <p>Click outside this box to close</p>
    <p>Clicks: <span $:text="clicks"></span></p>
  </div>
  <button $:click="open()">Open Box</button>
</div>
```

```js inject show
// Register directive
$.jqr.directive('click-outside', {
  bind(el, ctx) {
    const handler = (e) => {
      if (!el.contains(e.target)) {
        ctx.evaluate(ctx.expr);
      }
    };
    document.addEventListener('click', handler);
    el._clickOutsideHandler = handler;
  },
  unbind(el, ctx) {
    document.removeEventListener('click', el._clickOutsideHandler);
  }
});

$('#demo-click-outside').component({
  state: {
    isOpen: false,
    clicks: 0
  },
  methods: {
    open(state) {
      state.isOpen = true;
    },
    close(state) {
      state.isOpen = false;
      state.clicks++;
    }
  }
});
```

```js cleanup
onUnmount(() => {
  $('#demo-click-outside').component('destroy');
})
```

### Dynamic Color

```html live:no-code
<div id="demo-color" class="en-card en-card__body">
  <div class="en-flex en-gap-2 en-mb-3 en-flex-wrap">
    <button class="en-btn en-btn--sm" $:click="currentColor = 'red'">Red</button>
    <button class="en-btn en-btn--sm" $:click="currentColor = 'blue'">Blue</button>
    <button class="en-btn en-btn--sm" $:click="currentColor = 'green'">Green</button>
    <button class="en-btn en-btn--sm" $:click="currentColor = 'purple'">Purple</button>
  </div>
  <div $:color="currentColor" style="padding: 20px; border: 2px solid #ddd; border-radius: 4px;">
    <p style="margin: 0;">Text color changes based on selection</p>
  </div>
</div>
```

```html
<div id="demo-color">
  <button $:click="currentColor = 'red'">Red</button>
  <button $:click="currentColor = 'blue'">Blue</button>
  <button $:click="currentColor = 'green'">Green</button>
  <div $:color="currentColor">
    Text color changes based on selection
  </div>
</div>
```

```js inject show
// Register directive
$.jqr.directive('color', {
  update(el, ctx) {
    const color = ctx.evaluate(ctx.expr);
    el.style.color = color;
  }
});

$('#demo-color').component({
  state: { currentColor: 'red' }
});
```

```js cleanup
onUnmount(() => {
  $('#demo-color').component('destroy');
})
```

## Advanced Patterns

### With Modifiers

```html live:no-code
<div id="demo-modifiers" class="en-card en-card__body">
  <input class="en-input en-mb-2" $:model="text" placeholder="Type something...">
  <div class="en-flex en-gap-2 en-flex-wrap">
    <div $:badge="text">Default</div>
    <div $:badge.primary="text">Primary</div>
    <div $:badge.success="text">Success</div>
    <div $:badge.warning="text">Warning</div>
    <div $:badge.danger="text">Danger</div>
  </div>
</div>
```

```html
<div id="demo-modifiers">
  <input $:model="text" placeholder="Type something...">
  <div $:badge="text">Default</div>
  <div $:badge.primary="text">Primary</div>
  <div $:badge.success="text">Success</div>
  <div $:badge.warning="text">Warning</div>
  <div $:badge.danger="text">Danger</div>
</div>
```

```js inject show
// Register directive with modifier support
$.jqr.directive('badge', {
  update(el, ctx) {
    const count = ctx.evaluate(ctx.expr).length;
    
    // Determine badge color from modifiers
    let color = '#6c757d'; // default
    if (ctx.modifiers.includes('primary')) color = '#007bff';
    if (ctx.modifiers.includes('success')) color = '#28a745';
    if (ctx.modifiers.includes('warning')) color = '#ffc107';
    if (ctx.modifiers.includes('danger')) color = '#dc3545';
    
    // Create or update badge
    let badge = el.querySelector('.badge');
    if (!badge) {
      badge = document.createElement('span');
      badge.className = 'badge';
      badge.style.cssText = 'margin-left: 8px; padding: 2px 8px; border-radius: 12px; font-size: 12px; color: white;';
      el.appendChild(badge);
    }
    
    badge.textContent = count;
    badge.style.backgroundColor = color;
  }
});

$('#demo-modifiers').component({
  state: { text: '' }
});
```

```js cleanup
onUnmount(() => {
  $('#demo-modifiers').component('destroy');
})
```

### With Arguments

```js
$.jqr.directive('animate', {
  update(el, ctx) {
    const value = ctx.evaluate(ctx.expr);
    const property = ctx.arg || 'opacity'; // $:animate:width, $:animate:opacity, etc.
    
    el.style.transition = `${property} 0.3s`;
    el.style[property] = value;
  }
});
```

```html
<div $:animate:opacity="visible ? 1 : 0"></div>
<div $:animate:width="expanded ? '100%' : '50%'"></div>
```

### Accessing Component Methods

```js
$.jqr.directive('auto-save', {
  update(el, ctx) {
    const value = ctx.evaluate(ctx.expr);
    
    // Call component method
    ctx.instance.methods.saveData(ctx.instance.state, value);
  }
});
```

```html
<input $:model="content" $:auto-save.debounce.1000="content">
```

```js
$('#app').component({
  state: { content: '' },
  methods: {
    saveData(state, value) {
      console.log('Auto-saving:', value);
      // Save to server...
    }
  }
});
```

### Third-Party Integration

```js
// Integrate Tippy.js tooltip library
$.jqr.directive('tippy', {
  bind(el, ctx) {
    // Initialize on first bind
    el._tippyInstance = tippy(el, {
      content: ctx.evaluate(ctx.expr)
    });
  },
  update(el, ctx) {
    // Update content when state changes
    const content = ctx.evaluate(ctx.expr);
    if (el._tippyInstance) {
      el._tippyInstance.setContent(content);
    }
  },
  unbind(el, ctx) {
    // Cleanup on destroy
    if (el._tippyInstance) {
      el._tippyInstance.destroy();
    }
  }
});
```

```html
<button $:tippy="tooltipMessage">Hover me</button>
```

## Working with `$:bind`

Custom directives can be forwarded through `$:bind`:

```js
$.jqr.directive('tooltip', {
  update(el, ctx) {
    // Support both direct usage and $:bind
    const value = ctx.raw ? ctx.raw() : ctx.evaluate(ctx.expr);
    el.title = value;
  }
});
```

Now both work:

```html
<!-- Direct usage -->
<button $:tooltip="message">Help</button>

<!-- Via $:bind -->
<button $:bind="{ tooltip: message }">Help</button>
```

**Pattern:**
```js
const value = ctx.raw ? ctx.raw() : ctx.evaluate(ctx.expr);
```

- `ctx.raw` exists when forwarded via `$:bind` — value is already resolved
- Otherwise, use `ctx.evaluate(ctx.expr)` for normal usage

## Best Practices

### 1. Clean Up Resources

Always clean up in `unbind`:

```js
// ✅ Good
$.jqr.directive('timer', {
  bind(el, ctx) {
    el._timer = setInterval(() => { /* ... */ }, 1000);
  },
  unbind(el, ctx) {
    clearInterval(el._timer);
  }
});

// ❌ Bad - memory leak
$.jqr.directive('timer', {
  bind(el, ctx) {
    setInterval(() => { /* ... */ }, 1000);
  }
  // No unbind - interval keeps running!
});
```

### 2. Use `update` for Reactive Logic

Put reactive logic in `update`, not `bind`:

```js
// ✅ Good - updates automatically
$.jqr.directive('color', {
  update(el, ctx) {
    el.style.color = ctx.evaluate(ctx.expr);
  }
});

// ❌ Bad - only runs once
$.jqr.directive('color', {
  bind(el, ctx) {
    el.style.color = ctx.evaluate(ctx.expr);
  }
});
```

### 3. Store State on Element

For element-specific state:

```js
$.jqr.directive('counter', {
  bind(el, ctx) {
    el._count = 0; // Store on element
    el.addEventListener('click', () => {
      el._count++;
      el.textContent = `Clicked ${el._count} times`;
    });
  }
});
```

### 4. Check for Existing Instances

When integrating third-party libraries:

```js
$.jqr.directive('chart', {
  bind(el, ctx) {
    if (!el._chartInstance) {
      el._chartInstance = new Chart(el, {
        type: 'bar',
        data: ctx.evaluate(ctx.expr)
      });
    }
  },
  update(el, ctx) {
    if (el._chartInstance) {
      el._chartInstance.data = ctx.evaluate(ctx.expr);
      el._chartInstance.update();
    }
  },
  unbind(el, ctx) {
    if (el._chartInstance) {
      el._chartInstance.destroy();
    }
  }
});
```

### 5. Use Modifiers for Variants

Instead of multiple directives:

```js
// ✅ Good - one directive with modifiers
$.jqr.directive('alert', {
  update(el, ctx) {
    const msg = ctx.evaluate(ctx.expr);
    if (ctx.modifiers.includes('error')) {
      el.className = 'alert alert-error';
    } else {
      el.className = 'alert alert-info';
    }
    el.textContent = msg;
  }
});
```

```html
<div $:alert="message"></div>
<div $:alert.error="errorMessage"></div>
```

## Common Patterns

### Debounced Input Handler

```js
$.jqr.directive('debounce-input', {
  bind(el, ctx) {
    const delay = ctx.modifiers[0] || 300; // $:debounce-input.500
    let timer;
    
    const handler = () => {
      clearTimeout(timer);
      timer = setTimeout(() => {
        ctx.evaluate(ctx.expr);
      }, delay);
    };
    
    el.addEventListener('input', handler);
    el._debounceHandler = handler;
  },
  unbind(el, ctx) {
    el.removeEventListener('input', el._debounceHandler);
  }
});
```

### Intersection Observer

```js
$.jqr.directive('lazy-load', {
  bind(el, ctx) {
    const observer = new IntersectionObserver((entries) => {
      entries.forEach(entry => {
        if (entry.isIntersecting) {
          ctx.evaluate(ctx.expr);
          observer.disconnect();
        }
      });
    });
    
    observer.observe(el);
    el._observer = observer;
  },
  unbind(el, ctx) {
    if (el._observer) {
      el._observer.disconnect();
    }
  }
});
```

```html
<img $:lazy-load="loadImage()" data-src="large-image.jpg">
```

### Copy to Clipboard

```js
$.jqr.directive('copy', {
  bind(el, ctx) {
    el.addEventListener('click', () => {
      const text = ctx.evaluate(ctx.expr);
      navigator.clipboard.writeText(text).then(() => {
        console.log('Copied:', text);
      });
    });
  }
});
```

```html
<button $:copy="shareUrl">Copy Link</button>
```

## Directive Registration Timing

Register directives **before** mounting components:

```js
// ✅ Good - register first
$.jqr.directive('tooltip', { /* ... */ });

$('#app').component({ /* ... */ });

// ❌ Bad - component won't find directive
$('#app').component({ /* ... */ });

$.jqr.directive('tooltip', { /* ... */ });
```

## Dev Mode Warnings

Unknown directives trigger warnings in dev mode:

```js
$.jqr.config({ devMode: true });
```

```html
<!-- Warning: unrecognized directive "$:typo" -->
<div $:typo="value"></div>
```

See: [Configuration Guide](./prefixes-and-config.md)

## Next Steps

- [Directive Reference](../reference/directive-reference.md) - All built-in directives
- [Bind Directive](./bind-directive.md) - Using `$:bind` with custom directives
- [Configuration](./prefixes-and-config.md) - Dev mode and other settings
- [Plugins](./plugins.md) - Packaging directives as plugins
