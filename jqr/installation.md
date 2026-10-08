<!-- {
  "path": "jqr/installation",
  "name": "docs-jqr-installation",
  "meta": {
    "title": "Installation",
    "order": 1
  }
} -->

# Installation

Get started with JQR in under 2 minutes.

## Browser (IIFE) - Recommended

The fastest way to use JQR is via a direct script tag in your HTML:

```html
<!DOCTYPE html>
<html lang="en">
<head>
  <meta charset="UTF-8">
  <meta name="viewport" content="width=device-width, initial-scale=1.0">
  <title>JQR Quick Start</title>
</head>
<body>
  
  <div id="app">
    <h1 $:text="message"></h1>
    <button $:click="count++">Clicked: <span $:text="count"></span></button>
  </div>

  <!-- JQR Core (includes minimal $ fallback) -->
  <script src="./path/to/jqr.min.js"></script>
  
  <script>
    $('#app').component({
      state: {
        message: 'Hello JQR!',
        count: 0
      }
    });
  </script>
  
</body>
</html>
```

### What You Get

When you include `jqr.min.js` (or `jqr.js` for development), you get:

- **Global `JQR` object** with all core APIs
- **`$.jqr` namespace** for configuration and registration
- **Minimal `$` fallback** if jQuery isn't already loaded
- **All reactive directives** (`$:text`, `$:model`, `$:click`, etc.)

### With jQuery (Optional)

JQR works with or without jQuery. If you want to use full jQuery features:

```html
<!-- Optional: Include jQuery 4.0.0 first -->
<script src="https://code.jquery.com/jquery-4.0.0.min.js"></script>

<!-- Then JQR -->
<script src="./path/to/jqr.min.js"></script>
```

If jQuery is present, JQR uses it. Otherwise, JQR's built-in minimal `$` fallback handles everything needed for reactivity.

## CDN (Coming Soon)

CDN support will be available when JQR reaches stable release:

```html
<!-- Future CDN usage -->
<script src="https://cdn.jsdelivr.net/npm/@en32/jqr@0.1/dist/jqr.min.js"></script>
```

## NPM / Bun (Module)

For build-tool workflows (Vite, Webpack, etc.):

```bash
# Using Bun (recommended)
bun add @en32/jqr

# Using npm
npm install @en32/jqr
```

```js
// ESM import
import { $, jqr } from '@en32/jqr';

$('#app').component({
  state: { message: 'Hello from modules!' }
});
```

## File Structure

When you build JQR, you get these files in `dist/`:

| File | Description | Use Case |
|------|-------------|----------|
| `jqr.js` | IIFE, unminified | Development (browser) |
| `jqr.min.js` | IIFE, minified | Production (browser) |
| `index.mjs` | ESM, unminified | Module bundlers |
| `index.d.ts` | TypeScript definitions | Type checking |

## Quick Start Example

```html live:no-code
<div $:component:state="{ to: 'World' }" class="en-card">
  <div class="en-card__body">
    <h2 class="en-alert en-alert--info" $:text="'Hello, ' + to + '!'"></h2>
    <div class="en-flex en-gap-2">
      <input class="en-input" $:model="to">
    </div>
  </div>
</div>
```

```html
<div $:component:state="{ to: 'World' }">
  <h3 $:text="'Hello, ' + to + '!'"></h3>
  <input $:model="to">
</div>
```

```html live:no-code
<div id="quick-start" class="en-card">
  <div class="en-card__body">
    <h3 $:text="greeting"></h3>
    <div class="en-flex en-gap-2">
      <input class="en-input" $:model="name" type="text" placeholder="Your name">
      <button class="en-btn en-btn--primary" $:attr:disabled="name.length <= 0" $:click="reset">Reset</button>
    </div>
    <p class="en-text--muted" style="margin-top: 1rem;" $:show="name">
      Hello, <strong $:text="name"></strong>!
    </p>
  </div>
</div>
```

```html
<div id="quick-start">
  <h3 $:text="greeting"></h3>
  <input $:model="name" type="text" placeholder="Your name">
  <button $:click="reset" $:attr:disabled="name.length <= 0">Reset</button>
  <p $:show="name">Hello, <strong $:text="name"></strong>!</p>
</div>
```

```js inject show
$('#quick-start').component({
  state: {
    greeting: 'Welcome to JQR',
    name: ''
  },
  methods: {
    reset() {
      this.name = '';
    }
  }
});
```

```js cleanup
onUnmount(() => {
  $('#quick-start').component('destroy');
})
```

## Next Steps

Now that you have JQR installed:

1. Learn about [Reactivity](./reactivity.md) - How JQR's reactive system works
2. Explore [DOM Binding](./dom-binding.md) - Bind data to the DOM
3. Handle [Events](./events.md) - User interactions and event handling
4. Read the [API Reference](../API.md) - Complete API documentation

## Browser Support

JQR works in all modern browsers:

- ✅ Chrome/Edge (latest)
- ✅ Firefox (latest)
- ✅ Safari (latest)
- ✅ Mobile browsers (iOS Safari, Chrome Mobile)

**Note:** JQR uses modern JavaScript features (Proxy, ES6+). IE11 is not supported.

## Troubleshooting

### `$ is not defined`

Make sure `jqr.min.js` is loaded before your component initialization code:

```html
<!-- ✅ Correct order -->
<script src="./jqr.min.js"></script>
<script>
  $('#app').component({ /* ... */ });
</script>

<!-- ❌ Wrong order -->
<script>
  $('#app').component({ /* ... */ });
</script>
<script src="./jqr.min.js"></script>
```

### Directives Not Working

Ensure your HTML element has an `id` and you're calling `.component()` on it:

```html
<!-- ✅ Correct -->
<div id="app" $:text="message"></div>
<script>
  $('#app').component({ state: { message: 'Hello' } });
</script>

<!-- ❌ Missing component() call -->
<div id="app" $:text="message"></div>
<!-- Directives won't work without .component() -->
```

## Version Info

Current version: `0.1.0-dev`

Check your installed version:

```js
console.log(JQR.version); // Browser
console.log(jqr.version); // Module
```
