<!-- {
  "path": "jqr/guides/forms-and-models",
  "name": "docs-jqr-guides-forms-and-models",
  "meta": {
    "title": "Forms & Models",
    "order": 8
  }
} -->

# Forms & Models

Form handling is one of the most common tasks in web development. JQR provides the `$:model` directive for easy two-way data binding between form inputs and component state.

## What is `$:model`?

`$:model` creates a two-way binding:
1. **State → Input:** When state changes, input value updates
2. **Input → State:** When user types, state updates automatically

**Without `$:model`:**
```html
<input type="text" $:attr:value="username" $:input="username = $event.target.value">
```

<!--
$.jqr.directive('value', {
  update(el, ctx) {
    const v = ctx.evaluate(ctx.expr)
    if (el.value !== v) el.value = v   // properti, bukan attribute
  }
});
-->

**With `$:model`:**
```html
<input type="text" $:model="username">
```

Much cleaner!

## Basic Text Input

### Simple Binding

```html live:no-code
<div id="demo-basic-text" class="en-card en-card__body">
  <input class="en-input en-mb-2" $:model="message" placeholder="Type something...">
  <p class="en-text--muted">You typed: <code $:text="message || '(nothing)'"></code></p>
  <p class="en-text--muted">Length: <code $:text="message.length"></code> characters</p>
</div>
```

```html
<div id="demo-basic-text">
  <input $:model="message" placeholder="Type something...">
  <p>You typed: <code $:text="message || '(nothing)'"></code></p>
  <p>Length: <code $:text="message.length"></code> characters</p>
</div>
```

```js inject show
$('#demo-basic-text').component({
  state: { message: '' }
});
```

```js cleanup
onUnmount(() => {
  $('#demo-basic-text').component('destroy');
})
```

### Input Types

`$:model` works with all text-based inputs:

```html
<!-- Text -->
<input type="text" $:model="username">

<!-- Email -->
<input type="email" $:model="email">

<!-- Password -->
<input type="password" $:model="password">

<!-- Number -->
<input type="number" $:model="age">

<!-- Search -->
<input type="search" $:model="searchQuery">

<!-- URL -->
<input type="url" $:model="website">

<!-- Tel -->
<input type="tel" $:model="phone">
```

## Textarea

Multi-line text input:

```html live:no-code
<div id="demo-textarea" class="en-card en-card__body">
  <textarea class="en-textarea en-mb-2" $:model="description" placeholder="Enter description..." rows="4"></textarea>
  <p class="en-text--muted">Character count: <code $:text="description.length"></code> / 500</p>
  <div class="en-alert en-alert--warning" $:if="description.length > 500">
    Description is too long!
  </div>
</div>
```

```html
<div id="demo-textarea">
  <textarea $:model="description" placeholder="Enter description..." rows="4"></textarea>
  <p>Character count: <code $:text="description.length"></code> / 500</p>
  <div $:if="description.length > 500">Description is too long!</div>
</div>
```

```js inject show
$('#demo-textarea').component({
  state: { description: '' }
});
```

```js cleanup
onUnmount(() => {
  $('#demo-textarea').component('destroy');
})
```

## Checkbox

### Single Checkbox (Boolean)

```html live:no-code
<div id="demo-checkbox-boolean" class="en-card en-card__body">
  <label class="en-check">
    <input type="checkbox" $:model="agreedToTerms">
    <span>I agree to the terms and conditions</span>
  </label>
  <p class="en-text--muted en-mt-2">Agreed: <code $:text="agreedToTerms ? 'Yes' : 'No'"></code></p>
  <button class="en-btn en-btn--sm en-btn--primary en-mt-2" $:attr:disabled="!agreedToTerms">
    Continue
  </button>
</div>
```

```html
<div id="demo-checkbox-boolean">
  <label>
    <input type="checkbox" $:model="agreedToTerms">
    I agree to the terms and conditions
  </label>
  <p>Agreed: <code $:text="agreedToTerms ? 'Yes' : 'No'"></code></p>
  <button $:attr:disabled="!agreedToTerms">Continue</button>
</div>
```

```js inject show
$('#demo-checkbox-boolean').component({
  state: { agreedToTerms: false }
});
```

```js cleanup
onUnmount(() => {
  $('#demo-checkbox-boolean').component('destroy');
})
```

**Behavior:**
- Initial: `false`
- Checked: `true`
- Unchecked: `false`

### Multiple Checkboxes (Array)

```html live:no-code
<div id="demo-checkbox-array" class="en-card en-card__body">
  <h4>Select your interests:</h4>
  <div class="en-flex en-flex-col en-gap-2 en-mb-3">
    <label class="en-check">
      <input type="checkbox" $:model="interests" value="coding">
      <span>Coding</span>
    </label>
    <label class="en-check">
      <input type="checkbox" $:model="interests" value="design">
      <span>Design</span>
    </label>
    <label class="en-check">
      <input type="checkbox" $:model="interests" value="music">
      <span>Music</span>
    </label>
    <label class="en-check">
      <input type="checkbox" $:model="interests" value="sports">
      <span>Sports</span>
    </label>
  </div>
  <p class="en-text--muted">Selected: <code $:text="interests.join(', ') || 'none'"></code></p>
</div>
```

```html
<div id="demo-checkbox-array">
  <h4>Select your interests:</h4>
  <label>
    <input type="checkbox" $:model="interests" value="coding">
    Coding
  </label>
  <label>
    <input type="checkbox" $:model="interests" value="design">
    Design
  </label>
  <label>
    <input type="checkbox" $:model="interests" value="music">
    Music
  </label>
  <label>
    <input type="checkbox" $:model="interests" value="sports">
    Sports
  </label>
  <p>Selected: <code $:text="interests.join(', ') || 'none'"></code></p>
</div>
```

```js inject show
$('#demo-checkbox-array').component({
  state: { interests: [] }
});
```

```js cleanup
onUnmount(() => {
  $('#demo-checkbox-array').component('destroy');
})
```

**Behavior:**
- Each checkbox has a `value` attribute
- Checked boxes add their value to the array
- Unchecked boxes remove their value from the array
- Array updates reactively

**See:** [Checkbox & Radio Guide](./checkbox-and-radio.md) for advanced patterns.

## Radio Buttons

```html live:no-code
<div id="demo-radio" class="en-card en-card__body">
  <h4>Select your plan:</h4>
  <div class="en-flex en-flex-col en-gap-2 en-mb-3">
    <label class="en-radio">
      <input type="radio" $:model="plan" value="free">
      <span>Free</span>
    </label>
    <label class="en-radio">
      <input type="radio" $:model="plan" value="pro">
      <span>Pro ($9/month)</span>
    </label>
    <label class="en-radio">
      <input type="radio" $:model="plan" value="enterprise">
      <span>Enterprise ($99/month)</span>
    </label>
  </div>
  <p class="en-text--muted">Selected plan: <strong $:text="plan || 'none'"></strong></p>
</div>
```

```html
<div id="demo-radio">
  <h4>Select your plan:</h4>
  <label>
    <input type="radio" $:model="plan" value="free">
    Free
  </label>
  <label>
    <input type="radio" $:model="plan" value="pro">
    Pro ($9/month)
  </label>
  <label>
    <input type="radio" $:model="plan" value="enterprise">
    Enterprise ($99/month)
  </label>
  <p>Selected plan: <strong $:text="plan || 'none'"></strong></p>
</div>
```

```js inject show
$('#demo-radio').component({
  state: { plan: 'free' }
});
```

```js cleanup
onUnmount(() => {
  $('#demo-radio').component('destroy');
})
```

**Behavior:**
- All radios share the same model variable
- Selected radio's `value` becomes the model value
- Only one can be selected at a time

## Select Dropdown

### Single Select

```html live:no-code
<div id="demo-select-single" class="en-card en-card__body">
  <label class="en-form-label">Select Country:</label>
  <select class="en-select en-mb-2" $:model="country">
    <option value="">-- Select --</option>
    <option value="us">United States</option>
    <option value="uk">United Kingdom</option>
    <option value="ca">Canada</option>
    <option value="id">Indonesia</option>
  </select>
  <p class="en-text--muted">Selected: <code $:text="country || 'none'"></code></p>
</div>
```

```html
<div id="demo-select-single">
  <label>Select Country:</label>
  <select $:model="country">
    <option value="">-- Select --</option>
    <option value="us">United States</option>
    <option value="uk">United Kingdom</option>
    <option value="ca">Canada</option>
    <option value="id">Indonesia</option>
  </select>
  <p>Selected: <code $:text="country || 'none'"></code></p>
</div>
```

```js inject show
$('#demo-select-single').component({
  state: { country: '' }
});
```

```js cleanup
onUnmount(() => {
  $('#demo-select-single').component('destroy');
})
```

### Multiple Select

```html live:no-code
<div id="demo-select-multiple" class="en-card en-card__body">
  <label class="en-form-label">Select Languages (Ctrl/Cmd + Click):</label>
  <select class="en-select en-mb-2" $:model="languages" multiple size="5">
    <option value="js">JavaScript</option>
    <option value="py">Python</option>
    <option value="java">Java</option>
    <option value="cpp">C++</option>
    <option value="go">Go</option>
  </select>
  <p class="en-text--muted">Selected: <code $:text="languages.join(', ') || 'none'"></code></p>
</div>
```

```html
<div id="demo-select-multiple">
  <label>Select Languages (Ctrl/Cmd + Click):</label>
  <select $:model="languages" multiple size="5">
    <option value="js">JavaScript</option>
    <option value="py">Python</option>
    <option value="java">Java</option>
    <option value="cpp">C++</option>
    <option value="go">Go</option>
  </select>
  <p>Selected: <code $:text="languages.join(', ') || 'none'"></code></p>
</div>
```

```js inject show
$('#demo-select-multiple').component({
  state: { languages: [] }
});
```

```js cleanup
onUnmount(() => {
  $('#demo-select-multiple').component('destroy');
})
```

**Behavior:**
- Single select: model is a string (option value)
- Multiple select: model is an array (selected option values)

### Dynamic Options

```html live:no-code
<div id="demo-select-dynamic" class="en-card en-card__body">
  <label class="en-form-label">Select User:</label>
  <select class="en-select en-mb-2" $:model="selectedUserId">
    <option value="">-- Select User --</option>
    <option $:for="user in users" $:key="user.id" $:attr:value="user.id" $:text="user.name"></option>
  </select>
  <p class="en-text--muted">Selected ID: <code $:text="selectedUserId || 'none'"></code></p>
</div>
```

```html
<div id="demo-select-dynamic">
  <label>Select User:</label>
  <select $:model="selectedUserId">
    <option value="">-- Select User --</option>
    <option $:for="user in users" $:key="user.id" $:attr:value="user.id" $:text="user.name"></option>
  </select>
  <p>Selected ID: <code $:text="selectedUserId || 'none'"></code></p>
</div>
```

```js inject show
$('#demo-select-dynamic').component({
  state: {
    selectedUserId: '',
    users: [
      { id: 1, name: 'Alice' },
      { id: 2, name: 'Bob' },
      { id: 3, name: 'Carol' }
    ]
  }
});
```

```js cleanup
onUnmount(() => {
  $('#demo-select-dynamic').component('destroy');
})
```

## Model Modifiers

Modifiers customize how `$:model` processes input.

### `.trim`

Removes leading/trailing whitespace:

```html live:no-code
<div id="demo-trim" class="en-card en-card__body">
  <div class="en-mb-3">
    <label class="en-form-label">Without .trim:</label>
    <input class="en-input" $:model="nameWithoutTrim" placeholder="Try typing with spaces...">
    <p class="en-text--muted en-text-sm">Value: "<code $:text="nameWithoutTrim"></code>" (length: <code $:text="nameWithoutTrim.length"></code>)</p>
  </div>
  <div>
    <label class="en-form-label">With .trim:</label>
    <input class="en-input" $:model.trim="nameWithTrim" placeholder="Spaces automatically removed">
    <p class="en-text--muted en-text-sm">Value: "<code $:text="nameWithTrim"></code>" (length: <code $:text="nameWithTrim.length"></code>)</p>
  </div>
</div>
```

```html
<div id="demo-trim">
  <div>
    <label>Without .trim:</label>
    <input $:model="nameWithoutTrim" placeholder="Try typing with spaces...">
    <p>Value: "<code $:text="nameWithoutTrim"></code>"</p>
  </div>
  <div>
    <label>With .trim:</label>
    <input $:model.trim="nameWithTrim" placeholder="Spaces automatically removed">
    <p>Value: "<code $:text="nameWithTrim"></code>"</p>
  </div>
</div>
```

```js inject show
$('#demo-trim').component({
  state: {
    nameWithoutTrim: '',
    nameWithTrim: ''
  }
});
```

```js cleanup
onUnmount(() => {
  $('#demo-trim').component('destroy');
})
```

### `.number`

Parses input as a number:

```html live:no-code
<div id="demo-number" class="en-card en-card__body">
  <label class="en-form-label">Enter Age:</label>
  <input class="en-input en-mb-2" type="number" $:model.number="age">
  <p class="en-text--muted">Age: <code $:text="age"></code> (type: <code $:text="typeof age"></code>)</p>
  <p class="en-text--muted">Next year: <code $:text="age + 1"></code></p>
</div>
```

```html
<div id="demo-number">
  <label>Enter Age:</label>
  <input type="number" $:model.number="age">
  <p>Age: <code $:text="age"></code> (type: <code $:text="typeof age"></code>)</p>
  <p>Next year: <code $:text="age + 1"></code></p>
</div>
```

```js inject show
$('#demo-number').component({
  state: { age: 25 }
});
```

```js cleanup
onUnmount(() => {
  $('#demo-number').component('destroy');
})
```

### `.lazy`

Updates on `change` event (not `input`):

```html live:no-code
<div id="demo-lazy" class="en-card en-card__body">
  <div class="en-mb-3">
    <label class="en-form-label">Immediate (default):</label>
    <input class="en-input" $:model="immediate">
    <p class="en-text--muted en-text-sm">Updates: <code $:text="immediateCount"></code> times</p>
  </div>
  <div>
    <label class="en-form-label">Lazy (on blur/enter):</label>
    <input class="en-input" $:model.lazy="lazy">
    <p class="en-text--muted en-text-sm">Updates: <code $:text="lazyCount"></code> times</p>
  </div>
</div>
```

```html
<div id="demo-lazy">
  <div>
    <label>Immediate (default):</label>
    <input $:model="immediate">
    <p>Updates: <code $:text="immediateCount"></code> times</p>
  </div>
  <div>
    <label>Lazy (on blur/enter):</label>
    <input $:model.lazy="lazy">
    <p>Updates: <code $:text="lazyCount"></code> times</p>
  </div>
</div>
```

```js inject show
$('#demo-lazy').component({
  state: {
    immediate: '',
    lazy: '',
    immediateCount: 0,
    lazyCount: 0
  },
  watch: {
    immediate() {
      this.state.immediateCount++;
    },
    lazy() {
      this.state.lazyCount++;
    }
  }
});
```

```js cleanup
onUnmount(() => {
  $('#demo-lazy').component('destroy');
})
```

### `.debounce`

Delays update until user stops typing:

```html live:no-code
<div id="demo-debounce" class="en-card en-card__body">
  <label class="en-form-label">Search (500ms debounce):</label>
  <input class="en-input en-mb-2" $:model.debounce.500="searchQuery" placeholder="Type to search...">
  <p class="en-text--muted">Searching for: <code $:text="searchQuery || '(empty)'"></code></p>
  <p class="en-text--muted en-text-sm">Updates after you stop typing for 500ms</p>
</div>
```

```html
<div id="demo-debounce">
  <label>Search (500ms debounce):</label>
  <input $:model.debounce.500="searchQuery" placeholder="Type to search...">
  <p>Searching for: <code $:text="searchQuery || '(empty)'"></code></p>
</div>
```

```js inject show
$('#demo-debounce').component({
  state: { searchQuery: '' }
});
```

```js cleanup
onUnmount(() => {
  $('#demo-debounce').component('destroy');
})
```

**See:** [Model Modifiers Guide](./model-modifiers.md) for all modifiers.

## Complete Form Example

```html live:no-code
<div id="demo-complete-form" class="en-card">
  <div class="en-card__body">
    <h3>User Registration</h3>
    <form $:submit.prevent="handleSubmit()">
      <div class="en-mb-3">
        <label class="en-form-label">Username *</label>
        <input class="en-input" $:model.trim="form.username" placeholder="Enter username" required>
      </div>
      
      <div class="en-mb-3">
        <label class="en-form-label">Email *</label>
        <input class="en-input" type="email" $:model.trim="form.email" placeholder="your@email.com" required>
      </div>
      
      <div class="en-mb-3">
        <label class="en-form-label">Age</label>
        <input class="en-input" type="number" $:model.number="form.age" min="18" max="100">
      </div>
      
      <div class="en-mb-3">
        <label class="en-form-label">Country</label>
        <select class="en-select" $:model="form.country">
          <option value="">Select...</option>
          <option value="us">United States</option>
          <option value="uk">United Kingdom</option>
          <option value="id">Indonesia</option>
        </select>
      </div>
      
      <div class="en-mb-3">
        <label class="en-check">
          <input type="checkbox" $:model="form.newsletter">
          <span>Subscribe to newsletter</span>
        </label>
      </div>
      
      <div class="en-mb-3">
        <label class="en-check">
          <input type="checkbox" $:model="form.terms" required>
          <span>I agree to terms *</span>
        </label>
      </div>
      
      <button class="en-btn en-btn--primary" type="submit" $:attr:disabled="!isValid">
        Register
      </button>
    </form>
    
    <div class="en-alert en-alert--success en-mt-3" $:if="submitted">
      Form submitted successfully!
    </div>
  </div>
</div>
```

```html
<div id="demo-complete-form">
  <h3>User Registration</h3>
  <form $:submit.prevent="handleSubmit()">
    <div>
      <label>Username *</label>
      <input $:model.trim="form.username" placeholder="Enter username" required>
    </div>
    <div>
      <label>Email *</label>
      <input type="email" $:model.trim="form.email" placeholder="your@email.com" required>
    </div>
    <div>
      <label>Age</label>
      <input type="number" $:model.number="form.age" min="18" max="100">
    </div>
    <div>
      <label>Country</label>
      <select $:model="form.country">
        <option value="">Select...</option>
        <option value="us">United States</option>
        <option value="uk">United Kingdom</option>
        <option value="id">Indonesia</option>
      </select>
    </div>
    <div>
      <label>
        <input type="checkbox" $:model="form.newsletter">
        Subscribe to newsletter
      </label>
    </div>
    <div>
      <label>
        <input type="checkbox" $:model="form.terms" required>
        I agree to terms *
      </label>
    </div>
    <button type="submit" $:attr:disabled="!isValid">Register</button>
  </form>
  <div $:if="submitted">Form submitted successfully!</div>
</div>
```

```js inject show
$('#demo-complete-form').component({
  state: {
    form: {
      username: '',
      email: '',
      age: null,
      country: '',
      newsletter: false,
      terms: false
    },
    submitted: false
  },
  computed: {
    isValid: (s) => {
      return s.form.username.length >= 3 &&
             s.form.email.includes('@') &&
             s.form.terms;
    }
  },
  methods: {
    handleSubmit(state) {
      if (this.computed.isValid) {
        console.log('Form data:', state.form);
        state.submitted = true;
        setTimeout(() => {
          state.submitted = false;
        }, 3000);
      }
    }
  }
});
```

```js cleanup
onUnmount(() => {
  $('#demo-complete-form').component('destroy');
})
```

## Form Validation

### Basic Validation

```js
$('#form').component({
  state: {
    username: '',
    email: ''
  },
  computed: {
    usernameError: (s) => {
      if (!s.username) return 'Username is required';
      if (s.username.length < 3) return 'Username must be at least 3 characters';
      return null;
    },
    emailError: (s) => {
      if (!s.email) return 'Email is required';
      if (!s.email.includes('@')) return 'Invalid email format';
      return null;
    },
    isValid: (s) => {
      return !this.computed.usernameError && !this.computed.emailError;
    }
  }
});
```

```html
<div>
  <input $:model="username">
  <span class="error" $:if="usernameError" $:text="usernameError"></span>
</div>
<div>
  <input type="email" $:model="email">
  <span class="error" $:if="emailError" $:text="emailError"></span>
</div>
<button $:attr:disabled="!isValid">Submit</button>
```

### Async Validation

```js
$('#form').component({
  state: {
    username: '',
    checking: false,
    available: null
  },
  watch: {
    username: {
      async handler(newVal) {
        if (newVal.length < 3) return;
        
        this.state.checking = true;
        try {
          const res = await fetch(`/api/check-username?name=${newVal}`);
          const data = await res.json();
          this.state.available = data.available;
        } finally {
          this.state.checking = false;
        }
      },
      debounce: 500
    }
  }
});
```

```html
<input $:model="username">
<span $:if="checking">Checking...</span>
<span $:if="!checking && available === true">✓ Available</span>
<span $:if="!checking && available === false">✗ Already taken</span>
```

## Best Practices

### 1. Use `.trim` for Text Inputs

```html
<!-- ✅ Good -->
<input $:model.trim="username">

<!-- ❌ Bad - stores whitespace -->
<input $:model="username">
```

### 2. Use `.number` for Numeric Inputs

```html
<!-- ✅ Good - stored as number -->
<input type="number" $:model.number="age">

<!-- ❌ Bad - stored as string -->
<input type="number" $:model="age">
```

### 3. Provide Feedback

```html
<!-- ✅ Good - shows validation -->
<input $:model="email" $:class="{ invalid: emailError }">
<span $:if="emailError" $:text="emailError"></span>

<!-- ❌ Bad - no feedback -->
<input $:model="email">
```

### 4. Disable Submit on Invalid

```html
<!-- ✅ Good -->
<button type="submit" $:attr:disabled="!isValid">Submit</button>

<!-- ❌ Bad - allows invalid submission -->
<button type="submit">Submit</button>
```

### 5. Use Debounce for Search

```html
<!-- ✅ Good - reduces API calls -->
<input $:model.debounce.500="searchQuery">

<!-- ❌ Bad - API call on every keystroke -->
<input $:model="searchQuery">
```

## Common Patterns

### Confirm Password

```js
state: {
  password: '',
  confirmPassword: ''
},
computed: {
  passwordsMatch: (s) => s.password === s.confirmPassword && s.password.length > 0
}
```

```html
<input type="password" $:model="password" placeholder="Password">
<input type="password" $:model="confirmPassword" placeholder="Confirm Password">
<span $:if="!passwordsMatch && confirmPassword">Passwords don't match</span>
```

### Character Counter

```js
state: {
  bio: ''
},
computed: {
  bioRemaining: (s) => 200 - s.bio.length,
  bioExceeded: (s) => s.bio.length > 200
}
```

```html
<textarea $:model="bio" maxlength="200"></textarea>
<span $:class="{ error: bioExceeded }">
  <span $:text="bioRemaining"></span> characters remaining
</span>
```

### Auto-Save Draft

```js
watch: {
  content: {
    handler(newVal) {
      localStorage.setItem('draft', newVal);
      this.state.lastSaved = new Date().toLocaleTimeString();
    },
    debounce: 1000
  }
},
mounted() {
  this.state.content = localStorage.getItem('draft') || '';
}
```

## Next Steps

- [Model Modifiers](./model-modifiers.md) - All modifier options
- [Checkbox & Radio](./checkbox-and-radio.md) - Advanced checkbox/radio patterns
- [Modifier Reference](../reference/modifier-reference.md) - Complete modifier API
- [DOM Binding](./dom-binding.md) - Other binding techniques
- [Events](./events.md) - Form event handling
