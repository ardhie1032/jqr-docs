<!-- {
  "path": "jqr/guides/plugins",
  "name": "docs-jqr-guides-plugins",
  "meta": {
    "title": "Using Plugins",
    "order": 31
  }
} -->

# Using Plugins

Plugins extend JQR with additional features like internationalization, form validation, routing, and more. They integrate seamlessly with the core framework.

## What Are Plugins?

Plugins are packaged extensions that add functionality to JQR components:

**Common plugin types:**
- **Directives** - New directives (e.g., `$:validate`, `$:t`)
- **Utilities** - Helper functions (e.g., `$i18n.t()`, `$router.push()`)
- **Component Options** - New component properties (e.g., `i18n`, `validations`)
- **Global Features** - Framework-wide capabilities (e.g., state management, routing)

**Official plugins:**
- `@n32/jqr-h` - Render functions & hyperscript API
- `@n32/jqr-i18n` - Internationalization (i18n)
- `@n32/jqr-validate` - Form validation

## Installing Plugins

### Via CDN

Load plugin scripts after JQR core:

```html
<!-- Load JQR core -->
<script src="https://cdn.jsdelivr.net/npm/@en32/jqr@latest/dist/jqr.min.js"></script>

<!-- Load plugins -->
<script src="https://cdn.jsdelivr.net/npm/@en32/jqr-h@latest/dist/jqr-h.min.js"></script>
<script src="https://cdn.jsdelivr.net/npm/@en32/jqr-i18n@latest/dist/jqr-i18n.min.js"></script>
```

Plugins auto-register when loaded via CDN (they detect `window.$.jqr`).

### Via npm/bun

```bash
# Install with npm
npm install @en32/jqr-h @en32/jqr-i18n

# Or with bun
bun add @en32/jqr-h @en32/jqr-i18n
```

**Import and register:**

```js
import { $, jqr } from '@en32/jqr';
import jqrH from '@en32/jqr-h';
import jqrI18n from '@en32/jqr-i18n';

// Plugins auto-register on import (ESM)
// Or manually install:
$.jqr.use(jqrH);
$.jqr.use(jqrI18n, { /* options */ });
```

## Plugin Registration

### Automatic Registration (CDN)

When loaded via `<script>` tags, plugins auto-register:

```html
<script src="jqr.min.js"></script>
<script src="jqr-plugin.min.js"></script>
<!-- Plugin automatically calls $.jqr.use() -->
```

### Manual Registration (ESM)

```js
import { $ } from '@en32/jqr';
import myPlugin from 'jqr-plugin';

// Basic registration
$.jqr.use(myPlugin);

// With options
$.jqr.use(myPlugin, {
  option1: 'value',
  option2: true
});
```

### Registration Timing

**Important:** Plugins must be registered **before** mounting components.

```js
// ✅ Correct order
$.jqr.use(myPlugin);

$('#app').component({
  // Component can now use plugin features
});

// ❌ Wrong order
$('#app').component({ /* ... */ });

$.jqr.use(myPlugin);  // Too late! Component already mounted
```

## Using Plugin Features

Once installed, plugins extend JQR with new capabilities:

### Example: JQR-H (Render Functions)

```html
<div id="demo-jqr-h"></div>
```

```js
// JQR-H adds mount() and h() globally
JQRH.mount('#demo-jqr-h', {
  state: { count: 0 },
  render: (h) => 
    h('button', {
      on: { click: (s) => s.count++ },
      text: (s) => `Count: ${s.count}`
    })
});
```

### Example: I18n Plugin

```html
<div id="demo-i18n" $:component:state="{}">
  <!-- Plugin adds $:t directive -->
  <p $:t="'greeting'"></p>
  <p $:t="'welcome'" $:t:name="userName"></p>
</div>
```

```js
// Plugin adds $i18n global
$.jqr.use(jqrI18n, {
  locale: 'en',
  messages: {
    en: {
      greeting: 'Hello',
      welcome: 'Welcome, {name}!'
    },
    es: {
      greeting: 'Hola',
      welcome: '¡Bienvenido, {name}!'
    }
  }
});

// Access plugin API
$('#app').component({
  mounted() {
    // Plugin adds instance methods
    console.log(this.$i18n.locale);
    this.$i18n.setLocale('es');
  }
});
```

### Example: Validation Plugin

```html
<form id="demo-validate" $:component:state="{ email: '' }">
  <!-- Plugin adds $:validate directive -->
  <input $:model="email" $:validate:email>
  <span $:if="$v.email.$invalid" $:text="$v.email.$errors[0]"></span>
  <button $:disabled="$v.$invalid">Submit</button>
</form>
```

```js
$.jqr.use(jqrValidate);

$('#demo-validate').component({
  validations: {
    email: { required: true, email: true }
  }
});
```

## Plugin Configuration

Plugins accept configuration options during registration:

### Global Configuration

```js
$.jqr.use(myPlugin, {
  // Global plugin settings
  apiKey: 'your-key',
  debug: true,
  timeout: 5000
});
```

### Per-Component Configuration

Some plugins support per-component options:

```js
$('#app').component({
  // Plugin-specific component options
  i18n: {
    locale: 'en',
    fallbackLocale: 'en'
  },
  
  validations: {
    username: { required: true, minLength: 3 }
  }
});
```

### Accessing Plugin Config

```js
// Access global plugin config
const config = $.jqr.config();
console.log(config.plugins.myPlugin);

// Inside component
$('#app').component({
  mounted() {
    // Plugin may expose instance properties
    console.log(this.$pluginName);
  }
});
```

## Plugin Lifecycle

Plugins hook into JQR's initialization process:

```
1. $.jqr.use(plugin, options) called
2. Plugin's install() function runs
3. Plugin registers features:
   - Directives ($.jqr.directive())
   - Global methods ($.jqr.myMethod = ...)
   - Instance methods ($.fn.jqrComponent.prototype.$method)
   - Config defaults
4. Components mount and use plugin features
```

### Plugin Installation Flow

```js
const myPlugin = {
  install(jqr, options = {}) {
    // 1. Register directives
    jqr.directive('my-directive', { /* ... */ });
    
    // 2. Add global methods
    jqr.myGlobalMethod = function() { /* ... */ };
    
    // 3. Add instance methods
    $.fn.jqrComponent.prototype.$myMethod = function() {
      // Access component instance via 'this'
    };
    
    // 4. Extend config
    jqr.config({ myPlugin: options });
  }
};

$.jqr.use(myPlugin, { setting: 'value' });
```

## Multiple Plugins

Register multiple plugins in sequence:

```js
// Register plugins one by one
$.jqr.use(pluginA);
$.jqr.use(pluginB, { option: true });
$.jqr.use(pluginC);

// All plugins work together
$('#app').component({
  // Use features from all plugins
});
```

**Plugin order matters** if plugins depend on each other:

```js
// ✅ Correct - router before auth (if auth depends on router)
$.jqr.use(routerPlugin);
$.jqr.use(authPlugin);

// ❌ Wrong - auth won't find router
$.jqr.use(authPlugin);
$.jqr.use(routerPlugin);
```

## Plugin Capabilities

### 1. Adding Directives

Plugins can add new directives:

```js
const tooltipPlugin = {
  install(jqr) {
    jqr.directive('tooltip', {
      bind(el, ctx) {
        el.title = ctx.evaluate(ctx.expr);
      },
      update(el, ctx) {
        el.title = ctx.evaluate(ctx.expr);
      }
    });
  }
};

$.jqr.use(tooltipPlugin);

// Now use $:tooltip directive
// <button $:tooltip="helpText">Help</button>
```

### 2. Adding Global Methods

```js
const apiPlugin = {
  install(jqr, options) {
    jqr.api = {
      get: async (url) => {
        const res = await fetch(options.baseURL + url);
        return res.json();
      },
      post: async (url, data) => {
        const res = await fetch(options.baseURL + url, {
          method: 'POST',
          body: JSON.stringify(data)
        });
        return res.json();
      }
    };
  }
};

$.jqr.use(apiPlugin, { baseURL: 'https://api.example.com' });

// Use globally
const data = await $.jqr.api.get('/users');
```

### 3. Adding Instance Methods

```js
const loggerPlugin = {
  install(jqr) {
    $.fn.jqrComponent.prototype.$log = function(message) {
      console.log(`[${this.name}]`, message);
    };
  }
};

$.jqr.use(loggerPlugin);

$('#app').component({
  mounted() {
    this.$log('Component mounted');  // Uses plugin method
  }
});
```

### 4. Extending Component Options

```js
const analyticsPlugin = {
  install(jqr) {
    // Hook into component creation
    const originalMount = $.fn.jqrComponent;
    $.fn.jqrComponent = function(options) {
      if (options.analytics) {
        // Process analytics option
        options.created = (function(original) {
          return function() {
            trackPageView(options.analytics.page);
            if (original) original.call(this);
          };
        })(options.created);
      }
      return originalMount.call(this, options);
    };
  }
};

$.jqr.use(analyticsPlugin);

$('#app').component({
  analytics: {
    page: 'home'
  }
});
```

## Plugin Best Practices

### 1. Check for Existing Registration

Avoid double-registration:

```js
const myPlugin = {
  installed: false,
  
  install(jqr, options) {
    if (this.installed) {
      console.warn('Plugin already installed');
      return;
    }
    this.installed = true;
    
    // Plugin logic...
  }
};
```

### 2. Provide Default Options

```js
const myPlugin = {
  install(jqr, options = {}) {
    const config = {
      // Defaults
      timeout: 5000,
      debug: false,
      // Override with user options
      ...options
    };
    
    // Use config...
  }
};
```

### 3. Namespace Plugin Features

Avoid naming conflicts:

```js
const myPlugin = {
  install(jqr) {
    // Prefix directives
    jqr.directive('myplugin-tooltip', { /* ... */ });
    
    // Namespace globals
    jqr.myPlugin = { /* API */ };
    
    // Prefix instance methods
    $.fn.jqrComponent.prototype.$myPlugin = { /* methods */ };
  }
};
```

### 4. Document Dependencies

```js
const myPlugin = {
  install(jqr) {
    // Check for required plugins
    if (!jqr.requiredPlugin) {
      throw new Error('myPlugin requires requiredPlugin to be installed first');
    }
    
    // Plugin logic...
  }
};
```

### 5. Clean Up Resources

```js
const myPlugin = {
  install(jqr) {
    const cleanup = [];
    
    // Register cleanup
    jqr.directive('my-directive', {
      unbind(el) {
        // Clean up directive resources
      }
    });
    
    // Hook into component destroy
    const originalDestroy = $.fn.jqrComponent.prototype.destroy;
    $.fn.jqrComponent.prototype.destroy = function() {
      // Plugin cleanup
      originalDestroy.call(this);
    };
  }
};
```

## Common Plugin Patterns

### Pattern 1: Directive Plugin

```js
const directivePlugin = {
  install(jqr) {
    jqr.directive('focus', {
      bind(el) {
        el.focus();
      }
    });
  }
};

$.jqr.use(directivePlugin);
```

### Pattern 2: Utility Plugin

```js
const utilPlugin = {
  install(jqr, options) {
    jqr.utils = {
      debounce: (fn, delay) => {
        let timeout;
        return (...args) => {
          clearTimeout(timeout);
          timeout = setTimeout(() => fn(...args), delay);
        };
      },
      
      throttle: (fn, limit) => {
        let inThrottle;
        return (...args) => {
          if (!inThrottle) {
            fn(...args);
            inThrottle = true;
            setTimeout(() => inThrottle = false, limit);
          }
        };
      }
    };
  }
};

$.jqr.use(utilPlugin);

// Usage
const debouncedSearch = $.jqr.utils.debounce(search, 300);
```

### Pattern 3: Mixin Plugin

```js
const mixinPlugin = {
  install(jqr) {
    // Add global mixin
    jqr.mixin = function(mixin) {
      const originalMount = $.fn.jqrComponent;
      $.fn.jqrComponent = function(options) {
        // Merge mixin into component options
        const merged = { ...mixin, ...options };
        return originalMount.call(this, merged);
      };
    };
  }
};

$.jqr.use(mixinPlugin);

// Use mixin
$.jqr.mixin({
  created() {
    console.log('Global mixin created hook');
  }
});
```

### Pattern 4: Store Plugin

```js
const storePlugin = {
  install(jqr, options) {
    const store = jqr.reactive(options.state || {});
    
    jqr.store = store;
    
    $.fn.jqrComponent.prototype.$store = store;
  }
};

$.jqr.use(storePlugin, {
  state: {
    user: null,
    isAuthenticated: false
  }
});

// Access in components
$('#app').component({
  computed: {
    user: () => this.$store.user
  }
});
```

## Debugging Plugins

### Check Installation

```js
// Log registered plugins (if plugin stores itself)
console.log($.jqr.plugins);

// Check for plugin features
console.log(typeof $.jqr.myPluginMethod);  // 'function'
console.log($.jqr.directives['plugin-directive']);  // Directive object
```

### Dev Mode Warnings

Enable dev mode to see plugin warnings:

```js
$.jqr.config({ devMode: true });

// Plugin warnings will appear in console
```

### Test Plugin Features

```js
// Test directive registration
const hasDirective = $.jqr.directives['my-directive'];
console.log('Directive registered:', !!hasDirective);

// Test global method
const hasMethod = typeof $.jqr.myMethod === 'function';
console.log('Method available:', hasMethod);

// Test instance method
$('#test').component({
  mounted() {
    const hasInstanceMethod = typeof this.$myMethod === 'function';
    console.log('Instance method available:', hasInstanceMethod);
  }
});
```

## Plugin Compatibility

### Version Checks

```js
const myPlugin = {
  install(jqr) {
    // Check JQR version
    const version = jqr.version || '1.0.0';
    const [major] = version.split('.');
    
    if (parseInt(major) < 2) {
      console.warn('Plugin requires JQR v2.0+');
    }
    
    // Plugin logic...
  }
};
```

### Feature Detection

```js
const myPlugin = {
  install(jqr) {
    // Check for required features
    if (!jqr.directive) {
      throw new Error('Directive API not available');
    }
    
    if (!jqr.reactive) {
      throw new Error('Reactive API not available');
    }
    
    // Safe to use features...
  }
};
```

## Troubleshooting

### Plugin Not Working

**Check registration order:**
```js
// ✅ Register before mounting
$.jqr.use(plugin);
$('#app').component({ /* ... */ });
```

**Check plugin loaded:**
```js
// For CDN
console.log(window.MyPlugin);  // Should exist

// For ESM
import myPlugin from 'jqr-plugin';
console.log(myPlugin);  // Should be object with install()
```

**Check console for errors:**
- Missing dependencies
- Version incompatibility
- Configuration errors

### Conflicting Plugins

**Namespace conflicts:**
```js
// Plugin A adds: $.jqr.api
// Plugin B adds: $.jqr.api (conflict!)

// Solution: Use namespaced names
// Plugin A: $.jqr.pluginA.api
// Plugin B: $.jqr.pluginB.api
```

**Directive conflicts:**
```js
// Both plugins add $:validate directive
// Last one wins, first is overwritten

// Solution: Check before registering
jqr.directive('validate', {
  install(jqr) {
    if (jqr.directives['validate']) {
      console.warn('$:validate already registered');
      return;
    }
    // Register...
  }
});
```

## Available Official Plugins

### @n32/jqr-h

Render functions and hyperscript API for programmatic DOM creation.

**Features:**
- `h()` function for creating virtual nodes
- `mount()` for rendering components
- Direct DOM manipulation with fine-grained reactivity

**Install:**
```html
<script src="https://cdn.jsdelivr.net/npm/@en32/jqr-h@latest/dist/jqr-h.min.js"></script>
```

**Learn more:** [JQR-H Documentation](../../jqr-h/README.md)

---

### @n32/jqr-i18n

Internationalization plugin for multi-language apps.

**Features:**
- Translation management
- Locale switching
- Interpolation & pluralization
- `$:t` directive for inline translations

**Install:**
```html
<script src="https://cdn.jsdelivr.net/npm/@en32/jqr-i18n@latest/dist/jqr-i18n.min.js"></script>
```

**Learn more:** [I18n Documentation](../../plugins/i18n/README.md)

---

### @n32/jqr-validate

Form validation plugin with built-in rules and async support.

**Features:**
- Built-in validation rules
- Custom validators
- Async validation
- Error message management
- `$:validate` directive

**Install:**
```html
<script src="https://cdn.jsdelivr.net/npm/@en32/jqr-validate@latest/dist/jqr-validate.min.js"></script>
```

**Learn more:** [Validation Documentation](../../plugins/validate/README.md)

---

## Creating Your Own Plugin

Ready to create a plugin? See the [Creating Plugins Guide](./creating-plugins.md) for a complete walkthrough.

## Next Steps

- [Creating Plugins](./creating-plugins.md) - Build your own plugin
- [Custom Directives](./custom-directives.md) - Directive API reference
- [Configuration](./prefixes-and-config.md) - JQR config options

