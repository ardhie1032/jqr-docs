<!-- {
  "path": "jqr/guides/creating-plugins",
  "name": "docs-jqr-guides-creating-plugins",
  "meta": {
    "title": "Creating Plugins",
    "order": 32
  }
} -->

# Creating Plugins

Learn how to build JQR plugins to package and share reusable functionality. This guide covers everything from basic plugin structure to publishing on npm.

## Plugin Structure

A JQR plugin is an object with an `install()` method:

```js
const myPlugin = {
  // Required: install method
  install(jqr, options = {}) {
    // Plugin initialization code
    // - Register directives
    // - Add global methods
    // - Extend component prototype
    // - Set up utilities
  },
  
  // Optional: plugin metadata
  version: '1.0.0',
  name: 'my-plugin'
};

export default myPlugin;
```

**The `install()` method receives:**
1. **`jqr`** - The JQR instance (same as `$.jqr`)
2. **`options`** - User-provided configuration object

## Basic Plugin Example

Let's create a simple toast notification plugin:

```js
const jqrToast = {
  install(jqr, options = {}) {
    // Default options
    const config = {
      duration: 3000,
      position: 'top-right',
      ...options
    };
    
    // Add global toast method
    jqr.toast = function(message, type = 'info') {
      const toast = document.createElement('div');
      toast.className = `toast toast-${type} toast-${config.position}`;
      toast.textContent = message;
      document.body.appendChild(toast);
      
      // Auto-remove after duration
      setTimeout(() => {
        toast.remove();
      }, config.duration);
    };
    
    // Add instance method for components
    $.fn.jqrComponent.prototype.$toast = function(message, type) {
      jqr.toast(message, type);
    };
  },
  
  version: '1.0.0'
};

export default jqrToast;
```

**Usage:**

```js
// Install plugin
$.jqr.use(jqrToast, { duration: 5000 });

// Use globally
$.jqr.toast('Hello!', 'success');

// Use in component
$('#app').component({
  methods: {
    save(state) {
      // Save logic...
      this.$toast('Saved!', 'success');
    }
  }
});
```

## Plugin API

### 1. Registering Directives

Add custom directives to JQR:

```js
const myPlugin = {
  install(jqr) {
    // Register directive
    jqr.directive('highlight', {
      bind(el, ctx) {
        el.style.backgroundColor = ctx.evaluate(ctx.expr) || 'yellow';
      },
      
      update(el, ctx) {
        el.style.backgroundColor = ctx.evaluate(ctx.expr) || 'yellow';
      },
      
      unbind(el) {
        el.style.backgroundColor = '';
      }
    });
  }
};
```

**Directive lifecycle hooks:**
- `bind(el, ctx)` - Called once when directive is bound
- `update(el, ctx)` - Called when expression value changes
- `unbind(el, ctx)` - Called when directive is removed

**Context object (`ctx`):**
- `ctx.expr` - Expression string
- `ctx.evaluate(expr)` - Evaluate expression
- `ctx.modifiers` - Object of modifiers (e.g., `{ once: true }`)
- `ctx.arg` - Directive argument (e.g., `color` in `$:bind:color`)
- `ctx.raw()` - Get raw value (for h() compatibility)

### 2. Adding Global Methods

Extend `$.jqr` with new methods:

```js
const apiPlugin = {
  install(jqr, options) {
    // Add global API object
    jqr.api = {
      async get(endpoint) {
        const res = await fetch(options.baseURL + endpoint);
        return res.json();
      },
      
      async post(endpoint, data) {
        const res = await fetch(options.baseURL + endpoint, {
          method: 'POST',
          headers: { 'Content-Type': 'application/json' },
          body: JSON.stringify(data)
        });
        return res.json();
      }
    };
  }
};

// Usage
$.jqr.use(apiPlugin, { baseURL: 'https://api.example.com' });
const users = await $.jqr.api.get('/users');
```

### 3. Adding Instance Methods

Extend component prototype with new methods:

```js
const loggerPlugin = {
  install(jqr, options) {
    // Add instance method
    $.fn.jqrComponent.prototype.$log = function(message, level = 'info') {
      if (options.debug || level === 'error') {
        const name = this.name || 'Anonymous';
        console[level](`[${name}]`, message);
      }
    };
  }
};

// Usage in component
$('#app').component({
  mounted() {
    this.$log('Component mounted');
  },
  
  methods: {
    save(state) {
      try {
        // Save logic...
        this.$log('Saved successfully');
      } catch (error) {
        this.$log(error.message, 'error');
      }
    }
  }
});
```

### 4. Extending Component Options

Hook into component creation to add new options:

```js
const analyticsPlugin = {
  install(jqr, options) {
    // Store original component function
    const originalComponent = $.fn.jqrComponent;
    
    // Override with wrapper
    $.fn.jqrComponent = function(componentOptions) {
      // Check if component has analytics option
      if (componentOptions && componentOptions.analytics) {
        const analytics = componentOptions.analytics;
        
        // Wrap lifecycle hooks
        const originalCreated = componentOptions.created;
        componentOptions.created = function() {
          // Track page view
          options.tracker.track('pageview', analytics.page);
          
          // Call original created
          if (originalCreated) {
            originalCreated.call(this);
          }
        };
      }
      
      // Call original component function
      return originalComponent.call(this, componentOptions);
    };
    
    // Preserve jQuery plugin structure
    $.fn.jqrComponent.prototype = originalComponent.prototype;
  }
};

// Usage
$.jqr.use(analyticsPlugin, {
  tracker: {
    track: (event, data) => console.log(event, data)
  }
});

$('#app').component({
  analytics: {
    page: 'home'
  }
  // Component tracks pageview on creation
});
```

### 5. Using JQR Utilities

Plugins can use JQR's built-in utilities:

```js
const storePlugin = {
  install(jqr) {
    // Use jqr.reactive() for reactive state
    const store = jqr.reactive({
      user: null,
      isAuthenticated: false
    });
    
    // Use jqr.effect() for reactive effects
    jqr.effect(() => {
      console.log('User changed:', store.user);
    });
    
    // Use jqr.computed() for computed values
    store.userName = jqr.computed(() => {
      return store.user?.name || 'Guest';
    });
    
    // Expose store
    jqr.store = store;
    $.fn.jqrComponent.prototype.$store = store;
  }
};
```

## Plugin Design Patterns

### Pattern 1: Directive-Only Plugin

Simple plugin that only adds directives:

```js
const focusPlugin = {
  install(jqr) {
    jqr.directive('focus', {
      bind(el, ctx) {
        const shouldFocus = ctx.evaluate(ctx.expr);
        if (shouldFocus !== false) {
          // Delay to ensure element is in DOM
          setTimeout(() => el.focus(), 0);
        }
      },
      
      update(el, ctx) {
        const shouldFocus = ctx.evaluate(ctx.expr);
        if (shouldFocus) {
          el.focus();
        }
      }
    });
    
    jqr.directive('blur', {
      bind(el) {
        el.blur();
      }
    });
  }
};

export default focusPlugin;
```

### Pattern 2: Utility Plugin

Plugin that provides helper functions:

```js
const datePlugin = {
  install(jqr, options = {}) {
    const config = {
      locale: 'en-US',
      ...options
    };
    
    jqr.date = {
      format(date, format = 'short') {
        return new Intl.DateTimeFormat(config.locale, {
          dateStyle: format
        }).format(date);
      },
      
      relative(date) {
        const rtf = new Intl.RelativeTimeFormat(config.locale);
        const diff = date - Date.now();
        const days = Math.round(diff / (1000 * 60 * 60 * 24));
        return rtf.format(days, 'day');
      },
      
      parse(str) {
        return new Date(str);
      }
    };
    
    // Add to component prototype
    $.fn.jqrComponent.prototype.$date = jqr.date;
  }
};

export default datePlugin;
```

### Pattern 3: State Management Plugin

Plugin that manages global state:

```js
const storePlugin = {
  install(jqr, options = {}) {
    // Create reactive store
    const store = jqr.reactive(options.state || {});
    
    // Add mutations
    const mutations = options.mutations || {};
    const commit = (mutation, payload) => {
      if (mutations[mutation]) {
        mutations[mutation](store, payload);
      }
    };
    
    // Add actions
    const actions = options.actions || {};
    const dispatch = async (action, payload) => {
      if (actions[action]) {
        return await actions[action]({ state: store, commit }, payload);
      }
    };
    
    // Expose API
    jqr.store = { state: store, commit, dispatch };
    
    // Add to components
    $.fn.jqrComponent.prototype.$store = jqr.store;
  }
};

export default storePlugin;

// Usage
$.jqr.use(storePlugin, {
  state: {
    count: 0
  },
  mutations: {
    increment(state) {
      state.count++;
    }
  },
  actions: {
    async incrementAsync({ commit }) {
      await new Promise(resolve => setTimeout(resolve, 1000));
      commit('increment');
    }
  }
});
```

### Pattern 4: Lifecycle Mixin Plugin

Plugin that adds hooks to all components:

```js
const lifecyclePlugin = {
  install(jqr, options = {}) {
    const originalComponent = $.fn.jqrComponent;
    
    $.fn.jqrComponent = function(componentOptions) {
      if (!componentOptions) {
        return originalComponent.call(this, componentOptions);
      }
      
      // Wrap each lifecycle hook
      ['created', 'mounted', 'unmount'].forEach(hook => {
        const original = componentOptions[hook];
        componentOptions[hook] = function(...args) {
          // Before hook
          options.onBefore?.(hook, this);
          
          // Original hook
          const result = original?.call(this, ...args);
          
          // After hook
          options.onAfter?.(hook, this);
          
          return result;
        };
      });
      
      return originalComponent.call(this, componentOptions);
    };
    
    $.fn.jqrComponent.prototype = originalComponent.prototype;
  }
};

export default lifecyclePlugin;

// Usage
$.jqr.use(lifecyclePlugin, {
  onBefore(hook, component) {
    console.log(`Before ${hook}:`, component.name);
  },
  onAfter(hook, component) {
    console.log(`After ${hook}:`, component.name);
  }
});
```

## Best Practices

### 1. Prevent Double Registration

```js
const myPlugin = {
  installed: false,
  
  install(jqr, options) {
    if (this.installed) {
      if (jqr.config().devMode) {
        console.warn('[myPlugin] Already installed');
      }
      return;
    }
    
    this.installed = true;
    
    // Plugin logic...
  }
};
```

### 2. Provide Configuration Defaults

```js
const myPlugin = {
  install(jqr, options = {}) {
    // Merge with defaults
    const config = {
      timeout: 5000,
      debug: false,
      retries: 3,
      ...options
    };
    
    // Store config for access
    jqr.config({ myPlugin: config });
    
    // Use config...
  }
};
```

### 3. Use Namespacing

Avoid naming conflicts by namespacing your features:

```js
const myPlugin = {
  install(jqr) {
    // ✅ Good - namespaced
    jqr.myPlugin = { /* API */ };
    jqr.directive('myplugin-action', { /* ... */ });
    $.fn.jqrComponent.prototype.$myPlugin = { /* methods */ };
    
    // ❌ Bad - could conflict
    jqr.api = { /* ... */ };
    jqr.directive('action', { /* ... */ });
    $.fn.jqrComponent.prototype.$api = { /* ... */ };
  }
};
```

### 4. Check Dependencies

```js
const myPlugin = {
  install(jqr) {
    // Check JQR version
    if (!jqr.version || jqr.version < '1.0.0') {
      throw new Error('myPlugin requires JQR v1.0.0+');
    }
    
    // Check required features
    if (!jqr.reactive) {
      throw new Error('myPlugin requires reactive API');
    }
    
    // Check for other plugins
    if (!jqr.router) {
      console.warn('myPlugin works best with jqr-router');
    }
    
    // Plugin logic...
  }
};
```

### 5. Provide TypeScript Types

Create type definitions for your plugin:

```typescript
// types/index.d.ts
import { JQR } from '@en32/jqr';

declare module '@en32/jqr' {
  interface JQR {
    myPlugin: {
      doSomething(): void;
    };
  }
  
  interface JQRComponent {
    $myPlugin: {
      method(): void;
    };
  }
}

export interface MyPluginOptions {
  option1?: string;
  option2?: boolean;
}

export interface MyPlugin {
  install(jqr: JQR, options?: MyPluginOptions): void;
  version: string;
}

declare const myPlugin: MyPlugin;
export default myPlugin;
```

### 6. Document Your Plugin

Include comprehensive documentation:

```js
/**
 * My JQR Plugin
 * 
 * @description Adds X functionality to JQR components
 * @version 1.0.0
 * @author Your Name
 * 
 * @example
 * // Install plugin
 * $.jqr.use(myPlugin, {
 *   option: 'value'
 * });
 * 
 * // Use in component
 * $('#app').component({
 *   mounted() {
 *     this.$myPlugin.method();
 *   }
 * });
 */
const myPlugin = {
  install(jqr, options) {
    // Implementation...
  }
};

export default myPlugin;
```

### 7. Handle Cleanup

Ensure your plugin cleans up resources:

```js
const myPlugin = {
  install(jqr) {
    // Track resources
    const timers = new WeakMap();
    
    // Add feature that needs cleanup
    $.fn.jqrComponent.prototype.$interval = function(fn, delay) {
      const timer = setInterval(() => fn.call(this), delay);
      timers.set(this, timer);
      return timer;
    };
    
    // Hook into unmount for cleanup
    const originalDestroy = $.fn.jqrComponent.prototype.destroy;
    $.fn.jqrComponent.prototype.destroy = function() {
      // Clean up timers
      const timer = timers.get(this);
      if (timer) {
        clearInterval(timer);
        timers.delete(this);
      }
      
      // Call original destroy
      return originalDestroy.call(this);
    };
  }
};
```

## Testing Plugins

### Unit Testing

Test your plugin in isolation:

```js
// test/myPlugin.test.js
import { $, jqr } from '@en32/jqr';
import myPlugin from '../src/myPlugin';

describe('myPlugin', () => {
  beforeEach(() => {
    // Reset JQR state
    delete $.jqr.myPlugin;
  });
  
  test('installs without errors', () => {
    expect(() => {
      $.jqr.use(myPlugin);
    }).not.toThrow();
  });
  
  test('adds global method', () => {
    $.jqr.use(myPlugin);
    expect(typeof $.jqr.myPlugin).toBe('object');
  });
  
  test('adds instance method', () => {
    $.jqr.use(myPlugin);
    const div = document.createElement('div');
    document.body.appendChild(div);
    
    $(div).component({});
    const instance = $(div).data('jqr-component');
    
    expect(typeof instance.$myPlugin).toBe('object');
  });
  
  test('respects options', () => {
    $.jqr.use(myPlugin, { debug: true });
    const config = $.jqr.config().myPlugin;
    expect(config.debug).toBe(true);
  });
});
```

### Integration Testing

Test plugin with real components:

```js
// test/integration.test.js
describe('myPlugin integration', () => {
  test('works in component', async () => {
    $.jqr.use(myPlugin);
    
    const div = document.createElement('div');
    div.innerHTML = '<button $:click="doAction">Click</button>';
    document.body.appendChild(div);
    
    const component = $(div).component({
      methods: {
        doAction() {
          this.$myPlugin.method();
        }
      }
    });
    
    const button = div.querySelector('button');
    button.click();
    
    // Assert plugin behavior...
  });
});
```

## Packaging for Distribution

### Package Structure

```
my-jqr-plugin/
├── src/
│   └── index.js          # Source code
├── dist/
│   ├── my-plugin.js      # UMD build
│   ├── my-plugin.min.js  # Minified UMD
│   └── my-plugin.esm.js  # ES Module
├── types/
│   └── index.d.ts        # TypeScript definitions
├── test/
│   └── *.test.js         # Tests
├── README.md             # Documentation
├── package.json          # Package manifest
├── LICENSE               # License file
└── rollup.config.js      # Build config
```

### package.json

```json
{
  "name": "@username/jqr-plugin",
  "version": "1.0.0",
  "description": "A JQR plugin for X",
  "main": "dist/my-plugin.js",
  "module": "dist/my-plugin.esm.js",
  "types": "types/index.d.ts",
  "files": [
    "dist",
    "types",
    "README.md"
  ],
  "scripts": {
    "build": "rollup -c",
    "test": "jest",
    "prepublishOnly": "npm run build && npm test"
  },
  "keywords": ["jqr", "jquery-reactive", "plugin"],
  "peerDependencies": {
    "@en32/jqr": "^1.0.0"
  },
  "devDependencies": {
    "@en32/jqr": "^1.0.0",
    "rollup": "^3.0.0",
    "jest": "^29.0.0"
  }
}
```

### Build Configuration (Rollup)

```js
// rollup.config.js
import { terser } from 'rollup-plugin-terser';

export default [
  // UMD build
  {
    input: 'src/index.js',
    output: {
      file: 'dist/my-plugin.js',
      format: 'umd',
      name: 'MyPlugin',
      globals: {
        '@en32/jqr': 'JQR'
      }
    },
    external: ['@en32/jqr']
  },
  
  // Minified UMD
  {
    input: 'src/index.js',
    output: {
      file: 'dist/my-plugin.min.js',
      format: 'umd',
      name: 'MyPlugin',
      globals: {
        '@en32/jqr': 'JQR'
      }
    },
    external: ['@en32/jqr'],
    plugins: [terser()]
  },
  
  // ES Module
  {
    input: 'src/index.js',
    output: {
      file: 'dist/my-plugin.esm.js',
      format: 'es'
    },
    external: ['@en32/jqr']
  }
];
```

### Auto-registration for CDN

Support auto-registration when loaded via `<script>`:

```js
// src/index.js
const myPlugin = {
  install(jqr, options) {
    // Plugin logic...
  },
  version: '1.0.0'
};

// Auto-install if loaded via script tag
if (typeof window !== 'undefined' && window.$ && window.$.jqr) {
  window.$.jqr.use(myPlugin);
}

export default myPlugin;
```

## Publishing

### 1. Prepare for Publishing

```bash
# Build distribution files
npm run build

# Run tests
npm test

# Update version
npm version patch  # or minor, major
```

### 2. Publish to npm

```bash
# Login to npm
npm login

# Publish package
npm publish --access public
```

### 3. Publish to CDN

After publishing to npm, the package is automatically available on CDN:

```html
<!-- jsDelivr -->
<script src="https://cdn.jsdelivr.net/npm/@username/jqr-plugin@1.0.0/dist/my-plugin.min.js"></script>

<!-- unpkg -->
<script src="https://unpkg.com/@username/jqr-plugin@1.0.0/dist/my-plugin.min.js"></script>
```

### 4. Document Usage

Create comprehensive README.md:

````markdown
# My JQR Plugin

Short description of what your plugin does.

## Installation

### CDN

```html
<script src="https://cdn.jsdelivr.net/npm/@en32/jqr@latest/dist/jqr.min.js"></script>
<script src="https://cdn.jsdelivr.net/npm/@username/jqr-plugin@latest/dist/my-plugin.min.js"></script>
```

### npm

```bash
npm install @username/jqr-plugin
```

```js
import { $ } from '@en32/jqr';
import myPlugin from '@username/jqr-plugin';

$.jqr.use(myPlugin);
```

## Usage

Basic example of how to use the plugin.

## API

Document all methods, directives, and options.

## License

MIT
````

## Complete Example: Logger Plugin

Here's a complete, production-ready plugin example:

```js
/**
 * JQR Logger Plugin
 * Provides logging functionality for JQR components
 */
const jqrLogger = {
  install(jqr, options = {}) {
    // Configuration
    const config = {
      enabled: true,
      level: 'info',  // 'debug', 'info', 'warn', 'error'
      prefix: '[JQR]',
      timestamp: true,
      ...options
    };
    
    // Log levels
    const levels = { debug: 0, info: 1, warn: 2, error: 3 };
    const currentLevel = levels[config.level] || 1;
    
    // Logger implementation
    const logger = {
      _log(level, ...args) {
        if (!config.enabled || levels[level] < currentLevel) {
          return;
        }
        
        const prefix = config.timestamp
          ? `${config.prefix} [${new Date().toISOString()}]`
          : config.prefix;
        
        console[level](prefix, ...args);
      },
      
      debug(...args) { this._log('debug', ...args); },
      info(...args) { this._log('info', ...args); },
      warn(...args) { this._log('warn', ...args); },
      error(...args) { this._log('error', ...args); }
    };
    
    // Add to JQR global
    jqr.logger = logger;
    
    // Add to component instances
    $.fn.jqrComponent.prototype.$log = function(...args) {
      const name = this.name || 'Anonymous';
      logger.info(`[${name}]`, ...args);
    };
    
    // Hook into lifecycle if enabled
    if (config.logLifecycle) {
      const originalComponent = $.fn.jqrComponent;
      
      $.fn.jqrComponent = function(componentOptions) {
        if (componentOptions) {
          // Wrap lifecycle hooks
          ['created', 'mounted', 'unmount'].forEach(hook => {
            const original = componentOptions[hook];
            componentOptions[hook] = function(...args) {
              logger.debug(`Lifecycle: ${hook}`, this.name);
              return original?.call(this, ...args);
            };
          });
        }
        
        return originalComponent.call(this, componentOptions);
      };
      
      $.fn.jqrComponent.prototype = originalComponent.prototype;
    }
  },
  
  version: '1.0.0'
};

// Auto-install for CDN
if (typeof window !== 'undefined' && window.$ && window.$.jqr) {
  window.$.jqr.use(jqrLogger);
}

export default jqrLogger;
```

## Resources

- [Plugin Examples](https://github.com/jqr-plugins) - Collection of example plugins
- [Using Plugins](./plugins.md) - Guide for plugin users
- [Custom Directives](./custom-directives.md) - Directive API reference

## Next Steps

- Build and test your plugin
- Publish to npm
- Share with the community
- Contribute to official plugins

