<!-- {
  "path": "jqr-h/guides/directives-in-h",
  "name": "docs-jqr-jqrh-guides-directives-in-h",
  "meta": {
    "title": "Using Directives in h()",
    "order": 5
  }
} -->

# Using Directives in h()

Custom directives registered via `$.jqr.directive()` can be used in `h()`-based components through the `directive` attribute. This guide covers how directives work in the render function context.

## Basic Usage

### Registering a Directive

First, register a custom directive:

```js
$.jqr.directive('tooltip', {
  bind(el, ctx) {
    // Initial setup
    el.dataset.tooltip = ctx.raw();
  },
  update(el, ctx) {
    // Reactive updates
    el.dataset.tooltip = ctx.raw();
  },
  unbind(el, ctx) {
    // Cleanup
    delete el.dataset.tooltip;
  }
});
```

### Using in h()

Use the directive via the `directive` attribute:

```js
render(h, ctx) {
  return h('button', {
    directive: {
      'tooltip': () => 'Click to save'
    }
  }, ['Save']);
}
```

**Dynamic value:**

```js
render(h, ctx) {
  return h('button', {
    directive: {
      'tooltip': (s) => `${s.count} items saved`
    },
    text: () => 'Save',
    on: { click: (s) => s.count++ }
  });
}
```

## Directive Context in h()

### Key Difference: No Expression Compiler

In template-based components, directives receive **jQR expressions** as strings:

```html
<!-- Template: directive gets "$:tooltip='message'" -->
<button $:tooltip="message"></button>
```

In `h()`, there's **no expression compiler** — everything is already JavaScript:

```js
// h(): directive gets the RESULT of the function
h('button', {
  directive: {
    'tooltip': (s) => s.message  // Already JS, not a string
  }
})
```

### Using ctx.raw()

Directives that work with `h()` must use `ctx.raw()`, not `ctx.evaluate()`:

```js
$.jqr.directive('my-directive', {
  update(el, ctx) {
    // ❌ Wrong - no expression to evaluate in h()
    const value = ctx.evaluate(ctx.expr);  // ctx.expr is always ''
    
    // ✅ Correct - get the resolved value
    const value = ctx.raw();
  }
});
```

**Why?** In `h()`, the value is already resolved (computed by the `StateFn`). There's no expression string to parse.

### Context Object

The `DirectiveContext` in `h()` provides:

```typescript
interface DirectiveContext {
  el: HTMLElement;              // The element
  $el: JQuery;                  // jQuery wrapper
  prefix: string;               // Prefix (from config)
  name: string;                 // Directive name
  arg?: string;                 // Argument (from key)
  modifiers: string[];          // Modifiers (from key)
  expr: string;                 // Always '' in h()
  evaluate: (expr: string) => any;  // Only works with ''
  scope: Dict;                  // Always {} in h()
  instance: JqrComponent;       // Component instance
  raw: () => any;               // ✅ Use this!
}
```

## Directive Keys

### Name Only

```js
h('div', {
  directive: {
    'tooltip': () => 'Hover me'
  }
});
```

Parsed as:
- `name`: `'tooltip'`
- `arg`: `undefined`
- `modifiers`: `[]`

### With Modifiers

```js
h('div', {
  directive: {
    'tooltip.top.arrow': () => 'Hover me'
  }
});
```

Parsed as:
- `name`: `'tooltip'`
- `arg`: `undefined`
- `modifiers`: `['top', 'arrow']`

### With Argument

```js
h('div', {
  directive: {
    'transition:name': () => 'fade'
  }
});
```

Parsed as:
- `name`: `'transition'`
- `arg`: `'name'`
- `modifiers`: `[]`

### Combined

```js
h('div', {
  directive: {
    'tooltip:placement.top.arrow': () => 'Hover me'
  }
});
```

Parsed as:
- `name`: `'tooltip'`
- `arg`: `'placement'`
- `modifiers`: `['top', 'arrow']`

## Lifecycle Hooks

### bind() - Called Once

Executed when the element is first created:

```js
$.jqr.directive('focus', {
  bind(el, ctx) {
    // Called once when element is rendered
    el.focus();
  }
});
```

```js
render(h) {
  return h('input', {
    directive: { 'focus': true }
  });
}
```

### update() - Called Reactively

Wrapped in an `effect()` — runs whenever dependencies change:

```js
$.jqr.directive('badge', {
  update(el, ctx) {
    // Runs every time ctx.raw() dependencies change
    const count = ctx.raw();
    el.dataset.badge = count > 0 ? String(count) : '';
  }
});
```

```js
render(h, ctx) {
  return h('button', {
    directive: {
      'badge': (s) => s.notifications  // Re-runs when notifications change
    },
    text: () => 'Inbox'
  });
}
```

### unbind() - Cleanup

Called when the element is removed (component destroyed, or `h.for()` item removed):

```js
$.jqr.directive('interval', {
  bind(el, ctx) {
    const id = setInterval(() => {
      console.log('tick');
    }, 1000);
    el.dataset.intervalId = String(id);
  },
  unbind(el, ctx) {
    // Cleanup when element removed
    const id = Number(el.dataset.intervalId);
    clearInterval(id);
  }
});
```

## Reactive Values

### Static Value

```js
h('div', {
  directive: {
    'tooltip': () => 'Static text'  // Never changes
  }
});
```

`update()` runs once, never again.

### Dynamic Value

```js
h('div', {
  directive: {
    'tooltip': (s) => `Count: ${s.count}`  // Tracks s.count
  }
});
```

`update()` re-runs whenever `s.count` changes.

### Computed Value

```js
h('div', {
  directive: {
    'tooltip': (s) => s.items.length > 0 
      ? `${s.items.length} items` 
      : 'No items'
  }
});
```

`update()` re-runs whenever `s.items` or `s.items.length` changes.

## Multiple Directives

Use multiple directives on one element:

```js
render(h, ctx) {
  return h('button', {
    directive: {
      'tooltip': (s) => s.helpText,
      'ripple': true,
      'track:click': (s) => ({ id: s.userId, action: 'save' })
    },
    text: () => 'Save',
    on: { click: (s) => ctx.save() }
  });
}
```

Each directive gets its own lifecycle calls.

## Accessing Component Instance

Use `ctx.instance` to access component state/methods:

```js
$.jqr.directive('auto-save', {
  bind(el, ctx) {
    const interval = setInterval(() => {
      // Access component state
      const isDirty = ctx.instance.state.isDirty;
      if (isDirty) {
        // Call component method
        ctx.instance.options.methods?.save?.call(
          ctx.instance, 
          ctx.instance.state
        );
      }
    }, 5000);
    
    el.dataset.intervalId = String(interval);
  },
  unbind(el, ctx) {
    clearInterval(Number(el.dataset.intervalId));
  }
});
```

```js
render(h) {
  return h('form', {
    directive: { 'auto-save': true }
  }, [
    h('input', { model: 'content' })
  ]);
}
```

## Working with Modifiers

Extract modifiers from `ctx.modifiers`:

```js
$.jqr.directive('tooltip', {
  bind(el, ctx) {
    const placement = ctx.modifiers.includes('top') ? 'top'
                    : ctx.modifiers.includes('bottom') ? 'bottom'
                    : 'right';  // default
    
    el.dataset.tooltipPlacement = placement;
  },
  update(el, ctx) {
    el.dataset.tooltip = ctx.raw();
  }
});
```

```js
render(h) {
  return h('button', {
    directive: {
      'tooltip.top': () => 'Hover me'
    }
  });
}
```

## Working with Arguments

Extract argument from `ctx.arg`:

```js
$.jqr.directive('transition', {
  update(el, ctx) {
    const transitionName = ctx.arg || 'fade';  // Default to 'fade'
    const isActive = ctx.raw();
    
    el.classList.toggle(`${transitionName}-enter`, isActive);
    el.classList.toggle(`${transitionName}-leave`, !isActive);
  }
});
```

```js
render(h, ctx) {
  return h('div', {
    directive: {
      'transition:slide': (s) => s.isVisible
    }
  }, ['Content']);
}
```

## Combining with Other Attributes

Directives work alongside other attributes:

```js
render(h, ctx) {
  return h('button', {
    text: (s) => s.label,
    class: (s) => ({ primary: s.isPrimary }),
    attr: { disabled: (s) => s.isLoading },
    on: { click: (s) => ctx.submit() },
    directive: {
      'loading': (s) => s.isLoading,
      'tooltip': (s) => s.helpText
    }
  });
}
```

**Order of application:**
1. `text`, `html`
2. `class`, `style`, `show`, `if`
3. `attr`
4. `on`
5. `directive`
6. `model`
7. `ref`
8. `bind` (applied last, can override earlier ones)

## Common Patterns

### Tooltip Directive

```js
$.jqr.directive('tooltip', {
  bind(el, ctx) {
    const tooltip = document.createElement('div');
    tooltip.className = 'tooltip';
    tooltip.style.position = 'absolute';
    tooltip.style.display = 'none';
    document.body.appendChild(tooltip);
    
    el.dataset.tooltipId = String(Date.now());
    
    el.addEventListener('mouseenter', () => {
      tooltip.textContent = el.dataset.tooltip || '';
      tooltip.style.display = 'block';
      const rect = el.getBoundingClientRect();
      tooltip.style.left = rect.left + 'px';
      tooltip.style.top = (rect.top - 30) + 'px';
    });
    
    el.addEventListener('mouseleave', () => {
      tooltip.style.display = 'none';
    });
  },
  update(el, ctx) {
    el.dataset.tooltip = String(ctx.raw());
  },
  unbind(el) {
    const tooltips = document.querySelectorAll('.tooltip');
    tooltips.forEach(t => t.remove());
  }
});
```

### Click Outside Directive

```js
$.jqr.directive('click-outside', {
  bind(el, ctx) {
    const handler = (e) => {
      if (!el.contains(e.target)) {
        const callback = ctx.raw();
        if (typeof callback === 'function') {
          callback();
        }
      }
    };
    
    document.addEventListener('click', handler);
    el.dataset.clickOutsideHandler = 'attached';
  },
  unbind(el) {
    // Note: In real implementation, store handler reference
    // to remove the exact listener
    document.removeEventListener('click', () => {});
  }
});
```

```js
render(h, ctx) {
  return h('div', {
    class: 'modal',
    directive: {
      'click-outside': () => ctx.close
    }
  }, [
    h('div', { class: 'modal-content' }, ['Modal content'])
  ]);
}
```

### Lazy Load Images

```js
$.jqr.directive('lazy', {
  bind(el, ctx) {
    const observer = new IntersectionObserver((entries) => {
      entries.forEach(entry => {
        if (entry.isIntersecting) {
          const img = entry.target;
          img.src = img.dataset.src;
          observer.unobserve(img);
        }
      });
    });
    
    el.dataset.src = ctx.raw();
    observer.observe(el);
    (el as any).__lazyObserver = observer;
  },
  update(el, ctx) {
    el.dataset.src = ctx.raw();
  },
  unbind(el) {
    const observer = (el as any).__lazyObserver;
    if (observer) {
      observer.disconnect();
      delete (el as any).__lazyObserver;
    }
  }
});
```

```js
render(h, ctx) {
  return h('img', {
    directive: {
      'lazy': (s) => s.imageUrl
    },
    attr: { alt: () => 'Product image' }
  });
}
```

## Limitations

### 1. No Expression String

```js
// ❌ This doesn't work - no expression compiler
$.jqr.directive('eval', {
  update(el, ctx) {
    const result = ctx.evaluate('state.a + state.b');  // Error!
  }
});
```

**Solution:** Pass the computed value directly:

```js
h('div', {
  directive: {
    'eval': (s) => s.a + s.b  // Already computed
  }
});
```

### 2. No Scope Variables

```js
// ❌ ctx.scope is always {} in h()
$.jqr.directive('log', {
  update(el, ctx) {
    console.log(ctx.scope.item);  // undefined!
  }
});
```

**Solution:** Capture variables in the closure:

```js
h.for(
  (s) => s.items,
  (item) => item.id,
  (item) => h('div', {
    directive: {
      'log': () => item  // Closure captures item
    }
  })
)
```

### 3. Directive Not Registered

If a directive name doesn't exist:

```js
h('div', {
  directive: {
    'nonexistent': () => 'value'
  }
});
// ⚠️ devWarn: no directive named "nonexistent" is registered
```

**Solution:** Always register directives first:

```js
$.jqr.directive('nonexistent', {
  update(el, ctx) { /* ... */ }
});
```

## Best Practices

### 1. Always Use ctx.raw()

```js
// ✅ Good - works in both h() and templates
$.jqr.directive('my-dir', {
  update(el, ctx) {
    const value = ctx.raw();
  }
});
```

### 2. Store Cleanup References

```js
// ✅ Good - proper cleanup
$.jqr.directive('interval', {
  bind(el, ctx) {
    const id = setInterval(() => {}, 1000);
    (el as any).__intervalId = id;  // Store on element
  },
  unbind(el) {
    clearInterval((el as any).__intervalId);
  }
});
```

### 3. Handle Missing Values

```js
// ✅ Good - defensive
$.jqr.directive('tooltip', {
  update(el, ctx) {
    const text = ctx.raw();
    el.dataset.tooltip = text != null ? String(text) : '';
  }
});
```

### 4. Use TypeScript for Type Safety

```typescript
interface TooltipValue {
  text: string;
  placement: 'top' | 'bottom' | 'left' | 'right';
}

$.jqr.directive('tooltip', {
  update(el, ctx) {
    const value = ctx.raw() as TooltipValue;
    el.dataset.tooltip = value.text;
    el.dataset.placement = value.placement;
  }
});
```

## Debugging

### Enable Dev Mode

```js
$.jqr.config({ devMode: true });
```

This shows warnings for:
- Unregistered directive names
- Attempts to call `ctx.evaluate()` with non-empty string

### Log Context

```js
$.jqr.directive('debug', {
  update(el, ctx) {
    console.log({
      name: ctx.name,
      arg: ctx.arg,
      modifiers: ctx.modifiers,
      value: ctx.raw(),
      instance: ctx.instance
    });
  }
});
```

## Summary

**Key Points:**

1. Directives work in `h()` via `directive` attribute
2. Use `ctx.raw()`, not `ctx.evaluate()` (no expression compiler)
3. Directive keys support modifiers and arguments
4. `update()` is reactive — wrapped in `effect()`
5. `unbind()` is called on cleanup (disposers)
6. Access component via `ctx.instance`

**When to use:**

- Reusable DOM behavior (tooltips, animations, tracking)
- Third-party library integration (chart libraries, date pickers)
- Custom attribute logic (lazy loading, intersection observer)

## Next Steps

- [Architecture](./architecture.md) - Understanding JQR-H internals
- [Custom Directives Guide](../../jqr/guides/custom-directives.md) - Creating directives
- [h() Function Reference](./h-function.md) - Complete `h()` API
- [Components with h()](./components-with-h.md) - Building components
