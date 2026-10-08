<!-- {
  "path": "jqr-e/installation",
  "name": "docs-jqr-e-installation",
  "meta": {
    "title": "Installation",
    "order": 2
  }
} -->

# Installation

This guide covers how to install and load the JQR-E (Custom Elements) module in your project.

## Prerequisites

JQR-E requires the core JQR framework to be loaded first:

- **JQR Core** (`@en32/jqr`) - Required
- **JQR-H** (`@en32/jqr/h`) - Optional, only for render-based components

## Browser (IIFE)

### CDN - Separate Files

Load modules individually for maximum flexibility:

```html
<!-- 1. Core framework (required) -->
<script src="https://cdn.jsdelivr.net/npm/@en32/jqr@latest/dist/jqr.core.min.js"></script>

<!-- 2. Render helpers (optional - only if using render-based components) -->
<script src="https://cdn.jsdelivr.net/npm/@en32/jqr@latest/dist/jqr.h.min.js"></script>

<!-- 3. Custom Elements module (NEW!) -->
<script src="https://cdn.jsdelivr.net/npm/@en32/jqr@latest/dist/jqr.e.min.js"></script>
```

**Global APIs:**
- `window.JQR` - Core framework
- `window.JQRH` - Render helpers (if loaded)
- `window.JQRE` - Custom Elements API

**Load Order:** The order of `jqr.h.js` and `jqr.e.js` doesn't matter. Both can be loaded in any sequence after core.

### CDN - All-in-One Bundle

For convenience, use the combined bundle:

```html
<!-- Includes: core + h + element -->
<script src="https://cdn.jsdelivr.net/npm/@en32/jqr@latest/dist/jqr.all.min.js"></script>
```

**Global API:**
- `window.JQR` - Contains all APIs (core + h + element merged)
- Access element functions: `JQR.defineElement()`, `JQR.defineElements()`, etc.

### Local Files

Download files from the [releases page](https://github.com/n32/jqr/releases):

```html
<script src="/assets/js/jqr.core.min.js"></script>
<script src="/assets/js/jqr.h.min.js"></script>   <!-- optional -->
<script src="/assets/js/jqr.e.min.js"></script>
```

## Package Manager (ESM)

### Installation

**npm:**
```bash
npm install @en32/jqr
```

**yarn:**
```bash
yarn add @en32/jqr
```

**pnpm:**
```bash
pnpm add @en32/jqr
```

**bun:**
```bash
bun add @en32/jqr
```

### Import

**Core + Element:**
```js
import '@en32/jqr';
import { defineElement, defineElements, defineRegistered } from '@en32/jqr/element';
```

**With Render Support:**
```js
import '@en32/jqr';
import { h, render } from '@en32/jqr/h';
import { defineElement } from '@en32/jqr/element';
```

**Note:** JQR core must be imported before using element functions (ensures registry is available).

## Verification

Verify that JQR-E is loaded correctly:

### Browser Console

```js
// IIFE
console.log(JQRE.defineElement);  // [Function]

// Or with all-in-one bundle
console.log(JQR.defineElement);   // [Function]
```

### ESM

```js
import { defineElement } from '@en32/jqr/element';

console.log(typeof defineElement); // "function"
```

## Quick Start

Once installed, define your first custom element:

```html live:no-code
<div id="demo-install-verify">
  <div class="en-card">
    <hello-world></hello-world>
  </div>
</div>
```

```html
<div id="demo-install-verify">
  <hello-world></hello-world>
</div>
```

```js inject show
// Define the custom element
JQRE.defineElement('hello-world', {
  state: { greeting: 'Hello from JQR-E!' },
  template: `
    <div>
      <h3 $:text="greeting"></h3>
      <p>Custom element is working! ✨</p>
    </div>
  `
});
```

```js cleanup
onUnmount(() => {
  $('#demo-install-verify').component('destroy');
});
```

If you see the greeting message, JQR-E is installed correctly!

## Development vs Production

### Development Build

Use unminified files for development (better error messages):

```html
<script src="/jqr.core.js"></script>
<script src="/jqr.h.js"></script>
<script src="/jqr.e.js"></script>
```

**Features:**
- Detailed error messages with error codes
- Development warnings
- Larger file size

### Production Build

Use minified files for production:

```html
<script src="/jqr.core.min.js"></script>
<script src="/jqr.h.min.js"></script>
<script src="/jqr.e.min.js"></script>
```

**Features:**
- Compact error codes (e.g., `[jqr-e#E003]`)
- No warnings
- Optimized file size

### ESM Conditional Exports

Modern bundlers automatically select the right build:

```json
{
  "exports": {
    "./element": {
      "development": "./dist/element/index.mjs",
      "production": "./dist/element/index.min.mjs",
      "default": "./dist/element/index.mjs"
    }
  }
}
```

Set `NODE_ENV=production` to use production builds.

## Browser Compatibility

JQR-E requires browsers that support Custom Elements v1:

| Browser | Minimum Version |
|---------|----------------|
| Chrome | 67+ |
| Firefox | 63+ |
| Safari | 13.1+ |
| Edge | 79+ |

**Not Supported:**
- Internet Explorer (any version)
- Legacy Edge (before Chromium)

### Polyfills

For older browsers, use the [Custom Elements polyfill](https://github.com/webcomponents/polyfills/tree/master/packages/custom-elements):

```html
<script src="https://unpkg.com/@webcomponents/custom-elements@1.5.0/custom-elements.min.js"></script>
<script src="/jqr.core.js"></script>
<script src="/jqr.e.js"></script>
```

## TypeScript Support

JQR-E includes TypeScript definitions:

```ts
import type { ElementOptions, JqrElement } from '@en32/jqr/element';

const options: ElementOptions = {
  attributes: {
    userId: Number,
    active: Boolean
  },
  reflect: ['active'],
  expose: ['open', 'close']
};
```

**Type Definitions Location:**
- `@en32/jqr/element` → `./dist/element/index.d.ts`

## Bundle Size

Approximate sizes (minified + gzipped):

| File | Size | Description |
|------|------|-------------|
| `jqr.core.min.js` | ~12 KB | Core framework only |
| `jqr.h.min.js` | ~5 KB | Render helpers |
| `jqr.e.min.js` | ~6 KB | Custom Elements module |
| **`jqr.all.min.js`** | **~23 KB** | **All modules combined** |

**Tree-shaking (ESM):**
```js
// Only imports what you use
import { defineElement } from '@en32/jqr/element';
// Result: ~18 KB (core + element only)
```

## Troubleshooting

### "JQRE is not defined"

**Problem:** Trying to use JQRE before script is loaded.

**Solution:**
```html
<!-- ❌ Wrong order -->
<script>
  JQRE.defineElement('my-el', {...});
</script>
<script src="/jqr.e.js"></script>

<!-- ✅ Correct order -->
<script src="/jqr.e.js"></script>
<script>
  JQRE.defineElement('my-el', {...});
</script>
```

### "Component definition not found"

**Problem:** Using `defineElement('name')` without definition and component not registered.

**Solution:**
```js
// ❌ Component not registered
JQRE.defineElement('my-button');

// ✅ Register first, then define
JQR.register('my-button', { template: '...' });
JQRE.defineElement('my-button');

// ✅ Or provide definition directly
JQRE.defineElement('my-button', { template: '...' });
```

### "render() requires jqr-h"

**Problem:** Defining render-based component without loading JQR-H.

**Solution:**
```html
<!-- ❌ Missing jqr-h -->
<script src="/jqr.core.js"></script>
<script src="/jqr.e.js"></script>

<!-- ✅ Include jqr-h for render components -->
<script src="/jqr.core.js"></script>
<script src="/jqr.h.js"></script>
<script src="/jqr.e.js"></script>
```

### ESM Import Errors

**Problem:** Module not found in bundler.

**Solution:**
```js
// ❌ Wrong path
import { defineElement } from '@en32/jqr-element';

// ✅ Correct path (subpath export)
import { defineElement } from '@en32/jqr/element';
```

## Next Steps

Now that JQR-E is installed, learn how to use it:

- **[Defining Elements](./guides/defining-elements.md)** - Create your first custom element
- **[Attributes & Props](./guides/attributes-and-props.md)** - Handle props and attributes
- **[Events](./guides/events.md)** - Emit and listen to events
- **[Slots](./guides/slots.md)** - Use slots for content projection

## Additional Resources

- [JQR Core Installation](../jqr/installation.md)
- [JQR-H Installation](../jqr-h/installation.md)
- [API Reference](./reference/api-reference.md)
- [GitHub Repository](https://github.com/n32/jqr)
