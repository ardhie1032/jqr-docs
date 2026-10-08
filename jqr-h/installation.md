<!-- {
  "path": "jqr-h/installation",
  "name": "docs-jqr-jqrh-installation",
  "meta": {
    "title": "Installation",
    "order": 1
  }
} -->

# Installation

JQR-H is a companion library to JQR that provides render functions and hyperscript API. It requires JQR as a peer dependency.

## Browser Usage (CDN)

The simplest way to get started is using a CDN. Load JQR first, then JQR-H:

```html
<!DOCTYPE html>
<html>
<head>
  <title>JQR-H Example</title>
</head>
<body>
  <div id="app"></div>

  <!-- Load JQR first -->
  <script src="https://cdn.jsdelivr.net/npm/@en32/jqr@latest/dist/jqr.min.js"></script>
  
  <!-- Load JQR-H -->
  <script src="https://cdn.jsdelivr.net/npm/@en32/jqr-h@latest/dist/jqr-h.min.js"></script>

  <script>
    // Global JQRH is now available
    const { mount } = JQRH;

    mount('#app', {
      state: { message: 'Hello from JQR-H!' },
      render: ({ h, state }) => 
        h('h1', { text: (s) => s.message })
    });
  </script>
</body>
</html>
```

### Global Variables

When loaded via CDN, JQR-H exposes:

- **`JQRH`** - Main namespace with `mount()`, `unmount()`, `h()`, etc.
- **`JQRH.h`** - Hyperscript function
- **`JQRH.mount`** - Mount render component
- **`JQRH.unmount`** - Unmount component
- **`JQRH.renderNode`** - Low-level render function

## Package Manager Installation

### Using npm

```bash
npm install @en32/jqr @en32/jqr-h
```

### Using Yarn

```bash
yarn add @en32/jqr @en32/jqr-h
```

### Using Bun

```bash
bun add @en32/jqr @en32/jqr-h
```

### Using pnpm

```bash
pnpm add @en32/jqr @en32/jqr-h
```

## Module Usage (ESM)

After installing via package manager, import the functions you need:

```js
import { mount, unmount } from '@en32/jqr-h';

mount('#app', {
  state: { count: 0 },
  render: (h, { state }) => 
    h('div', {}, [
      h('p', { text: (s) => `Count: ${s.count}` }),
      h('button', {
        on: { click: () => state.count++ },
        text: () => 'Increment'
      })
    ])
});
```

### TypeScript

JQR-H includes TypeScript definitions out of the box:

```typescript
import { h, mount, RenderComponentOptions } from '@en32/jqr-h';

interface AppState {
  count: number;
  message: string;
}

const options: RenderComponentOptions<AppState> = {
  state: { count: 0, message: 'Hello' },
  render: ({ h, state }) => 
    h('div', {}, [
      h('h1', { text: (s) => s.message }),
      h('p', { text: (s) => `Count: ${s.count}` })
    ])
};

mount('#app', options);
```

## Verify Installation

Create a simple test to verify JQR-H is working:

```html live:no-code
<div id="demo-verify" class="en-card en-px-3">
  <!-- Rendered by JQR-H -->
</div>
```

```html
<div id="demo-verify">
  <!-- Rendered by JQR-H -->
</div>
```

```js inject show
const { mount } = JQRH;

mount('#demo-verify', {
  state: { 
    framework: 'JQR-H',
    version: '0.0.1-dev'
  },
  render: (h, { state }) => 
    h('div', {}, [
      h('h3', {},  ['✅ JQR-H Loaded Successfully!']),
      h('p', { text: (s) => `Framework: ${s.framework}` }),
      h('p', { text: (s) => `Version: ${s.version}` })
    ])
});
```

```js cleanup
onUnmount(() => {
  JQRH.unmount('#demo-verify');
})
```

If you see the message above, JQR-H is installed correctly!

## Version Compatibility

JQR-H requires specific versions of JQR:

| JQR-H Version | JQR Version | Notes |
|---------------|-------------|-------|
| 1.x.x | ^1.0.0 | Current stable |
| 0.x.x | ^0.9.0 | Beta versions |

Always use matching major versions of JQR and JQR-H.

## File Sizes

Understanding bundle sizes:

| Build | Size (Minified) | Size (Gzipped) |
|-------|-----------------|----------------|
| jqr-h.js | ~12 KB | ~4 KB |
| jqr-h.min.js | ~8 KB | ~3 KB |

JQR-H is lightweight and adds minimal overhead to your JQR application.

## Build Variants

JQR-H provides multiple build formats:

```
dist/
├── jqr-h.js          # UMD development build
├── jqr-h.min.js      # UMD production build (use this for CDN)
├── jqr-h.esm.js      # ES Module build
└── jqr-h.cjs.js      # CommonJS build
```

### Which Build to Use?

- **Browser (CDN)**: `jqr-h.min.js`
- **Modern bundlers (Vite, Rollup)**: Auto-detects ESM build
- **Webpack**: Auto-detects appropriate build
- **Node.js**: CommonJS build

## Using with jQuery

JQR-H works with both full jQuery and JQR's built-in `$` fallback:

### With Full jQuery

```html
<!-- Load jQuery 4.0.0 -->
<script src="https://code.jquery.com/jquery-4.0.0.min.js"></script>

<!-- Load JQR -->
<script src="https://cdn.jsdelivr.net/npm/@en32/jqr@latest/dist/jqr.min.js"></script>

<!-- Load JQR-H -->
<script src="https://cdn.jsdelivr.net/npm/@en32/jqr-h@latest/dist/jqr-h.min.js"></script>
```

### Without jQuery (Fallback)

```html
<!-- Only JQR (includes $ fallback) -->
<script src="https://cdn.jsdelivr.net/npm/@en32/jqr@latest/dist/jqr.min.js"></script>

<!-- JQR-H -->
<script src="https://cdn.jsdelivr.net/npm/@en32/jqr-h@latest/dist/jqr-h.min.js"></script>
```

Both work identically. Use full jQuery only if you need advanced DOM manipulation features.

## Development Setup

For local development with hot module replacement:

### Vite

```js
// vite.config.js
import { defineConfig } from 'vite';

export default defineConfig({
  resolve: {
    alias: {
      '@en32/jqr': '/node_modules/@en32/jqr/dist/jqr.esm.js',
      '@en32/jqr-h': '/node_modules/@en32/jqr-h/dist/jqr-h.esm.js'
    }
  }
});
```

### Webpack

```js
// webpack.config.js
module.exports = {
  resolve: {
    alias: {
      '@en32/jqr': require.resolve('@en32/jqr/dist/jqr.esm.js'),
      '@en32/jqr-h': require.resolve('@en32/jqr-h/dist/jqr-h.esm.js')
    }
  }
};
```

## Troubleshooting

### JQR Not Found

**Error:** `Cannot find module '@en32/jqr'`

**Solution:** Install JQR first:
```bash
npm install @en32/jqr
```

### JQRH is Undefined

**Error:** `JQRH is not defined`

**Solution:** Make sure JQR-H script is loaded after JQR:
```html
<!-- Wrong order -->
<script src="jqr-h.min.js"></script>
<script src="jqr.min.js"></script>

<!-- Correct order -->
<script src="jqr.min.js"></script>
<script src="jqr-h.min.js"></script>
```

### Type Errors in TypeScript

**Error:** `Cannot find type definitions`

**Solution:** Make sure `@en32/jqr-h` is installed (not just `@en32/jqr`):
```bash
npm install @en32/jqr-h
```

### Version Mismatch

**Error:** Runtime errors or unexpected behavior

**Solution:** Ensure matching major versions:
```bash
npm list @en32/jqr @en32/jqr-h
```

If versions don't match, update:
```bash
npm update @en32/jqr @en32/jqr-h
```

## Next Steps

Now that JQR-H is installed:

- [Render Functions Guide](./guides/render-function.md) - Learn the `render()` option
- [h() Function Guide](./guides/h-function.md) - Master the hyperscript API
- [Components with h()](./guides/components-with-h.md) - Build reusable components

## CDN Providers

JQR-H is available on multiple CDNs:

### jsDelivr (Recommended)

```html
<script src="https://cdn.jsdelivr.net/npm/@en32/jqr-h@latest/dist/jqr-h.min.js"></script>
```

### unpkg

```html
<script src="https://unpkg.com/@en32/jqr-h@latest/dist/jqr-h.min.js"></script>
```

### Specific Version

Always recommended for production:

```html
<script src="https://cdn.jsdelivr.net/npm/@en32/jqr-h@1.0.0/dist/jqr-h.min.js"></script>
```

## Loading Strategy

For best performance, use async/defer when possible:

```html
<!-- Defer loading (recommended) -->
<script defer src="jqr.min.js"></script>
<script defer src="jqr-h.min.js"></script>
<script defer src="your-app.js"></script>
```

Both JQR and JQR-H scripts must use the same loading strategy (both defer or both normal).

## Summary

**Browser Usage:**
1. Load JQR script
2. Load JQR-H script
3. Use global `JQRH`

**Module Usage:**
1. Install packages: `npm install @en32/jqr @en32/jqr-h`
2. Import: `import { h, mount } from '@en32/jqr-h'`
3. Use in your code

JQR-H is now ready to use! Continue to the guides to learn how to build components with render functions.
