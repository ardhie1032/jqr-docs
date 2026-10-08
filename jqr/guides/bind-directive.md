<!-- {
  "path": "jqr/guides/bind-directive",
  "name": "docs-jqr-guides-bind-directive",
  "meta": {
    "title": "Bind Directive",
    "order": 5
  }
} -->

# Bind Directive

The `$:bind` directive provides a shorthand for binding multiple attributes, classes, styles, and content from a single object expression.

## Basic Syntax

```html
<div $:bind="{ text: message, class: classes, style: styles }"></div>
```

Instead of:
```html
<div $:text="message" $:class="classes" $:style="styles"></div>
```

## How It Works

`$:bind` accepts an object where:
- Keys are directive names (`text`, `class`, `style`, etc.)
- Values are the expressions to bind

```js
$('#app').component({
  state: {
    title: 'Hello World',
    isActive: true,
    color: 'blue'
  }
});
```

```html
<div $:bind="{
  text: title,
  class: { active: isActive },
  style: { color: color }
}"></div>

<!-- Renders as: -->
<div class="active" style="color: blue;">Hello World</div>
```

## Supported Keys

### Built-in Bindings

| Key | Equivalent | Purpose |
|-----|------------|---------|
| `text` | `$:text` | Set text content |
| `html` | `$:html` | Set HTML content |
| `class` | `$:class` | Bind CSS classes |
| `style` | `$:style` | Bind inline styles |
| `show` | `$:show` | Toggle visibility |

### Attributes

Any other key becomes an attribute binding (like `$:attr:*`):

```html
<a $:bind="{
  href: url,
  title: tooltip,
  target: '_blank'
}">Link</a>

<!-- Equivalent to: -->
<a $:attr:href="url" $:attr:title="tooltip" target="_blank">Link</a>
```

## Text Content

```html live:no-code
<div id="bind-text" class="en-card">
  <div class="en-card__body">
    <h3>Text Binding</h3>
    
    <div class="en-form-group">
      <label class="en-label">Message</label>
      <input class="en-input" $:model="message">
    </div>
    
    <div class="en-alert en-alert--info" $:bind="{ text: message }"></div>
  </div>
</div>
```

```html
<div id="bind-text">
  <h3>Text Binding</h3>
  
  <label>Message</label>
  <input $:model="message">
  
  <div $:bind="{ text: message }"></div>
</div>
```

```js inject show
$('#bind-text').component({
  state: {
    message: 'Hello from $:bind!'
  }
});
```

```js cleanup
onUnmount(() => {
  $('#bind-text').component('destroy');
})
```

## HTML Content

```html live:no-code
<div id="bind-html" class="en-card">
  <div class="en-card__body">
    <h3>HTML Binding</h3>
    
    <div class="en-alert en-alert--warning" style="margin-bottom: 1rem;">
      <strong>Warning:</strong> Only use with trusted content!
    </div>
    
    <div $:bind="{ html: richContent }" class="en-alert en-alert--muted"></div>
  </div>
</div>
```

```html
<div id="bind-html">
  <h3>HTML Binding</h3>
  
  <div $:bind="{ html: richContent }"></div>
</div>
```

```js inject show
$('#bind-html').component({
  state: {
    richContent: '<strong>Bold</strong> and <em>italic</em> text'
  }
});
```

```js cleanup
onUnmount(() => {
  $('#bind-html').component('destroy');
})
```

## Class Binding

### Object Syntax

```html live:no-code
<div id="bind-class-obj" class="en-card">
  <div class="en-card__body">
    <h3>Class Binding (Object)</h3>
    
    <div class="en-flex en-gap-2" style="margin-bottom: 1rem;">
      <label class="en-label">
        <input type="checkbox" $:model="isActive">
        Active
      </label>
      <label class="en-label">
        <input type="checkbox" $:model="hasError">
        Error
      </label>
    </div>
    
    <div 
      $:bind="{ class: { 
        'en-alert--success': isActive,
        'en-alert--danger': hasError
      }}"
      class="en-alert">
      Status indicator
    </div>
  </div>
</div>
```

```html
<div id="bind-class-obj">
  <h3>Class Binding</h3>
  
  <label>
    <input type="checkbox" $:model="isActive"> Active
  </label>
  <label>
    <input type="checkbox" $:model="hasError"> Error
  </label>
  
  <div $:bind="{ class: { 
    active: isActive,
    error: hasError
  }}">
    Status indicator
  </div>
</div>
```

```js inject show
$('#bind-class-obj').component({
  state: {
    isActive: true,
    hasError: false
  }
});
```

```js cleanup
onUnmount(() => {
  $('#bind-class-obj').component('destroy');
})
```

### Array Syntax

```html live:no-code
<div id="bind-class-arr" class="en-card">
  <div class="en-card__body">
    <h3>Class Binding (Array)</h3>
    
    <div class="en-form-group" style="margin-bottom: 1rem;">
      <label class="en-label">Theme</label>
      <select class="en-input" $:model="theme">
        <option value="light">Light</option>
        <option value="dark">Dark</option>
        <option value="auto">Auto</option>
      </select>
    </div>
    
    <div 
      $:bind="{ class: ['en-alert', `en-alert--${theme}`] }"
      $:text="`Theme: ${theme}`">
    </div>
  </div>
</div>
```

```html
<div id="bind-class-arr">
  <h3>Class Array</h3>
  
  <select $:model="theme">
    <option value="light">Light</option>
    <option value="dark">Dark</option>
  </select>
  
  <div $:bind="{ class: ['box', `theme-${theme}`] }">
    Themed box
  </div>
</div>
```

```js inject show
$('#bind-class-arr').component({
  state: {
    theme: 'light'
  }
});
```

```js cleanup
onUnmount(() => {
  $('#bind-class-arr').component('destroy');
})
```

## Style Binding

### Object Syntax

```html live:no-code
<div id="bind-style" class="en-card">
  <div class="en-card__body">
    <h3>Style Binding</h3>
    
    <div class="en-flex en-gap-2" style="margin-bottom: 1rem;">
      <div class="en-form-group">
        <label class="en-label">Color</label>
        <input type="color" $:model="color">
      </div>
      
      <div class="en-form-group">
        <label class="en-label">Size (px)</label>
        <input class="en-input" type="number" $:model.number="size">
      </div>
    </div>
    
    <div 
      $:bind="{ 
        style: { 
          color: color,
          fontSize: size + 'px',
          fontWeight: 'bold'
        },
        text: 'Styled text'
      }"
      class="en-alert en-alert--muted">
    </div>
  </div>
</div>
```

```html
<div id="bind-style">
  <h3>Style Binding</h3>
  
  <label>Color</label>
  <input type="color" $:model="color">
  
  <label>Size</label>
  <input type="number" $:model.number="size">
  
  <div $:bind="{ 
    style: { 
      color: color,
      fontSize: size + 'px'
    },
    text: 'Styled text'
  }"></div>
</div>
```

```js inject show
$('#bind-style').component({
  state: {
    color: '#3b82f6',
    size: 20
  }
});
```

```js cleanup
onUnmount(() => {
  $('#bind-style').component('destroy');
})
```

### String Syntax

```html
<div $:bind="{ style: 'color: red; font-size: 20px;' }"></div>
```

## Show/Hide

```html live:no-code
<div id="bind-show" class="en-card">
  <div class="en-card__body">
    <h3>Visibility Toggle</h3>
    
    <label class="en-label">
      <input type="checkbox" $:model="isVisible">
      Show message
    </label>
    
    <div 
      $:bind="{ 
        show: isVisible,
        text: 'This message can be toggled!',
        class: 'en-alert en-alert--success'
      }"
      style="margin-top: 1rem;">
    </div>
  </div>
</div>
```

```html
<div id="bind-show">
  <h3>Visibility</h3>
  
  <label>
    <input type="checkbox" $:model="isVisible">
    Show message
  </label>
  
  <div $:bind="{ 
    show: isVisible,
    text: 'Toggleable message'
  }"></div>
</div>
```

```js inject show
$('#bind-show').component({
  state: {
    isVisible: true
  }
});
```

```js cleanup
onUnmount(() => {
  $('#bind-show').component('destroy');
})
```

## Attribute Binding

Any key not in `text`, `html`, `class`, `style`, `show` becomes an attribute:

```html live:no-code
<div id="bind-attrs" class="en-card">
  <div class="en-card__body">
    <h3>Dynamic Link</h3>
    
    <div class="en-form-group" style="margin-bottom: 1rem;">
      <label class="en-label">URL</label>
      <input class="en-input" $:model="url" placeholder="https://example.com">
    </div>
    
    <div class="en-form-group" style="margin-bottom: 1rem;">
      <label class="en-label">Link Text</label>
      <input class="en-input" $:model="linkText">
    </div>
    
    <a 
      $:bind="{
        href: url,
        title: 'Visit ' + linkText,
        target: '_blank',
        text: linkText
      }"
      class="en-btn en-btn--primary">
    </a>
  </div>
</div>
```

```html
<div id="bind-attrs">
  <h3>Dynamic Link</h3>
  
  <label>URL</label>
  <input $:model="url">
  
  <label>Text</label>
  <input $:model="linkText">
  
  <a $:bind="{
    href: url,
    title: 'Visit ' + linkText,
    target: '_blank',
    text: linkText
  }"></a>
</div>
```

```js inject show
$('#bind-attrs').component({
  state: {
    url: 'https://github.com',
    linkText: 'GitHub'
  }
});
```

```js cleanup
onUnmount(() => {
  $('#bind-attrs').component('destroy');
})
```

## Complete Example: Dynamic Card

```html live:no-code
<div id="dynamic-card" class="en-card">
  <div class="en-card__body">
    <h3>User Card Builder</h3>
    
    <div class="en-flex en-gap-2" style="margin-bottom: 1rem;">
      <div class="en-form-group">
        <label class="en-label">Name</label>
        <input class="en-input" $:model="name">
      </div>
      
      <div class="en-form-group">
        <label class="en-label">Role</label>
        <input class="en-input" $:model="role">
      </div>
    </div>
    
    <div class="en-flex en-gap-2" style="margin-bottom: 1rem;">
      <label class="en-label">
        <input type="checkbox" $:model="isOnline">
        Online
      </label>
      
      <label class="en-label">
        <input type="checkbox" $:model="isPremium">
        Premium
      </label>
    </div>
    
    <div 
      $:bind="{
        class: {
          'en-card': true,
          'en-card--highlighted': isPremium
        },
        style: {
          borderLeft: isPremium ? '4px solid gold' : 'none'
        },
        'data-status': isOnline ? 'online' : 'offline'
      }">
      <div class="en-card__body">
        <h4 $:text="name"></h4>
        <p class="en-text--muted" $:text="role"></p>
        <span 
          $:bind="{
            class: {
              'en-badge': true,
              'en-badge--success': isOnline,
              'en-badge--muted': !isOnline
            },
            text: isOnline ? 'Online' : 'Offline'
          }">
        </span>
      </div>
    </div>
  </div>
</div>
```

```html
<div id="dynamic-card">
  <h3>User Card</h3>
  
  <label>Name</label>
  <input $:model="name">
  
  <label>Role</label>
  <input $:model="role">
  
  <label>
    <input type="checkbox" $:model="isOnline"> Online
  </label>
  
  <label>
    <input type="checkbox" $:model="isPremium"> Premium
  </label>
  
  <div $:bind="{
    class: {
      card: true,
      premium: isPremium
    },
    style: {
      borderLeft: isPremium ? '4px solid gold' : 'none'
    },
    'data-status': isOnline ? 'online' : 'offline'
  }">
    <h4 $:text="name"></h4>
    <p $:text="role"></p>
    <span $:bind="{
      class: {
        badge: true,
        online: isOnline
      },
      text: isOnline ? 'Online' : 'Offline'
    }"></span>
  </div>
</div>
```

```js inject show
$('#dynamic-card').component({
  state: {
    name: 'Alice Johnson',
    role: 'Senior Developer',
    isOnline: true,
    isPremium: false
  }
});
```

```js cleanup
onUnmount(() => {
  $('#dynamic-card').component('destroy');
})
```

## When to Use `$:bind`

### ✅ Good Use Cases

**1. Multiple attributes from one object:**
```html
<img $:bind="imageProps">
```

```js
computed: {
  imageProps(state) {
    return {
      src: state.imageUrl,
      alt: state.imageAlt,
      title: state.imageTitle,
      loading: 'lazy'
    };
  }
}
```

**2. Computed binding object:**
```html
<div $:bind="cardBindings"></div>
```

```js
computed: {
  cardBindings(state) {
    return {
      class: { active: state.isActive, disabled: state.isDisabled },
      style: { backgroundColor: state.bgColor },
      'data-id': state.id
    };
  }
}
```

**3. Props spreading:**
```html
<custom-element $:bind="elementProps"></custom-element>
```

### ❌ When Not to Use

**1. Single binding:**
```html
<!-- ❌ Overkill -->
<div $:bind="{ text: message }"></div>

<!-- ✅ Better -->
<div $:text="message"></div>
```

**2. Static attributes:**
```html
<!-- ❌ Unnecessary -->
<a $:bind="{ href: '#', target: '_blank' }">Link</a>

<!-- ✅ Better -->
<a href="#" target="_blank">Link</a>
```

## Comparison with Direct Directives

### `$:bind` vs Direct Directives

```html
<!-- Using $:bind -->
<div $:bind="{ 
  text: message,
  class: { active: isActive },
  style: { color: color }
}"></div>

<!-- Using direct directives -->
<div 
  $:text="message"
  $:class="{ active: isActive }"
  $:style="{ color: color }"></div>
```

**Pros of `$:bind`:**
- Cleaner when binding many properties
- Can use computed property for all bindings
- Good for dynamic prop spreading

**Pros of direct directives:**
- More explicit and readable
- Easier to see what's bound at a glance
- Better for one or two bindings

## Conflict Warning

If an element has both `$:bind` and an explicit directive for the same key, both run:

```html
<!-- ⚠️ Both run; last-scanned wins -->
<div 
  $:bind="{ text: 'From bind' }"
  $:text="'From directive'">
</div>
```

**Dev warning:**
```
[jqr] $:bind="{...}" on <div> also carries explicit $:text 
— both run; whichever is scanned last wins for overlapping keys.
```

**Recommendation:** Don't mix `$:bind` with explicit directives for the same property.

## Custom Directives

`$:bind` also works with custom directives registered via `$.jqr.directive()`:

```js
// Register custom directive
$.jqr.directive('tooltip', {
  mounted(el, { expr, evaluate }) {
    const update = () => {
      const text = evaluate(expr);
      el.setAttribute('title', text);
    };
    update();
    return update;
  }
});
```

```html
<!-- Use in $:bind -->
<button $:bind="{ 
  text: 'Hover me',
  tooltip: 'This is a tooltip!'
}"></button>
```

## Modifiers in `$:bind`

You can use modifiers in `$:bind` keys:

```html
<input $:bind="{ 
  'model.number': price,
  'model.trim': name
}">

<!-- Equivalent to: -->
<input $:model.number="price">
<input $:model.trim="name">
```

**Note:** This only works for binding multiple directives of the same type with different modifiers.

## Best Practices

### 1. Use Computed for Complex Bindings

```js
// ✅ Good: centralized logic
computed: {
  buttonBindings(state) {
    return {
      class: {
        'btn': true,
        'btn-primary': state.variant === 'primary',
        'btn-disabled': state.isDisabled
      },
      disabled: state.isDisabled,
      'aria-busy': state.isLoading
    };
  }
}
```

```html
<button $:bind="buttonBindings">Click me</button>
```

### 2. Keep Objects Readable

```html
<!-- ❌ Hard to read -->
<div $:bind="{ text: a, class: { x: b, y: c, z: d }, style: { color: e, fontSize: f + 'px' } }"></div>

<!-- ✅ Better: use computed or multiline -->
<div $:bind="elementConfig"></div>
```

### 3. Don't Overuse

```html
<!-- ❌ Overkill for simple cases -->
<div $:bind="{ text: 'Hello' }"></div>

<!-- ✅ Just use HTML -->
<div>Hello</div>
```

### 4. Combine with Static Attributes

```html
<!-- ✅ Static + dynamic -->
<a 
  class="link"
  target="_blank"
  $:bind="{ href: url, title: tooltip }">
  Link
</a>
```

## Performance

`$:bind` is just as efficient as individual directives:
- Each sub-binding is created once
- Only changed properties trigger updates
- No extra overhead vs direct directives

## Browser Compatibility

Works in all modern browsers. No polyfills needed.

## Next Steps

- [DOM Binding](./dom-binding.md) - `$:text`, `$:html`, `$:class`, `$:style`
- [Events](./events.md) - Event handling with `$:click`, `$:on-*`
- [Model Modifiers](./model-modifiers.md) - Advanced form binding
