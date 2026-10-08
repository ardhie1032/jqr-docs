<!-- {
  "path": "jqr/examples/ajax-form",
  "name": "docs-jqr-examples-ajax-form",
  "meta": {
    "title": "AJAX Form",
    "order": 2
  }
} -->

# Example: AJAX Form

A user registration form with async validation, API submission, and error handling.

## Features

- ✅ Real-time validation
- ✅ Async username availability check
- ✅ Form submission with loading state
- ✅ Error handling
- ✅ Success feedback

## Live Demo

```html live:no-code
<div id="demo-ajax-form" class="en-card">
  <div class="en-card__header">
    <h3>User Registration</h3>
  </div>
  <div class="en-card__body">
    <!-- Success Message -->
    <div class="en-alert en-alert--success en-mb-3" $:if="submitted">
      <strong>Success!</strong> Account created for <strong $:text="form.username"></strong>
    </div>

    <!-- Registration Form -->
    <form $:submit.prevent="handleSubmit" $:if="!submitted">
      <!-- Username -->
      <div class="en-mb-3">
        <label class="en-form-label">
          Username *
          <span class="en-text--muted en-text-sm" $:if="checkingUsername"> (checking...)</span>
        </label>
        <input 
          class="en-input" 
          $:model.trim="form.username" 
          $:class="{ 'en-input--error': errors.username, 'en-input--success': !errors.username && form.username && !checkingUsername }"
          placeholder="Choose a username">
        <div class="en-form-help en-text--danger" $:if="errors.username" $:text="errors.username"></div>
        <div class="en-form-help en-text--success" $:if="!errors.username && form.username && !checkingUsername && usernameAvailable">
          ✓ Username available
        </div>
      </div>

      <!-- Email -->
      <div class="en-mb-3">
        <label class="en-form-label">Email *</label>
        <input 
          class="en-input" 
          type="email" 
          $:model.trim="form.email"
          $:class="{ 'en-input--error': errors.email }"
          placeholder="your@email.com">
        <div class="en-form-help en-text--danger" $:if="errors.email" $:text="errors.email"></div>
      </div>

      <!-- Password -->
      <div class="en-mb-3">
        <label class="en-form-label">Password *</label>
        <input 
          class="en-input" 
          type="password" 
          $:model="form.password"
          $:class="{ 'en-input--error': errors.password }"
          placeholder="Minimum 8 characters">
        <div class="en-form-help en-text--danger" $:if="errors.password" $:text="errors.password"></div>
      </div>

      <!-- Confirm Password -->
      <div class="en-mb-3">
        <label class="en-form-label">Confirm Password *</label>
        <input 
          class="en-input" 
          type="password" 
          $:model="form.confirmPassword"
          $:class="{ 'en-input--error': errors.confirmPassword }"
          placeholder="Re-enter password">
        <div class="en-form-help en-text--danger" $:if="errors.confirmPassword" $:text="errors.confirmPassword"></div>
      </div>

      <!-- Terms -->
      <div class="en-mb-3">
        <label class="en-check">
          <input type="checkbox" $:model="form.agreeToTerms">
          <span>I agree to the terms and conditions *</span>
        </label>
      </div>

      <!-- Submit Button -->
      <button 
        class="en-btn en-btn--primary" 
        type="submit" 
        $:attr:disabled="!isValid || submitting">
        <span $:if="!submitting">Create Account</span>
        <span $:if="submitting">Creating...</span>
      </button>

      <!-- Server Error -->
      <div class="en-alert en-alert--danger en-mt-3" $:if="serverError">
        <strong>Error:</strong> <span $:text="serverError"></span>
      </div>
    </form>

    <!-- Reset Button -->
    <button class="en-btn en-btn--sm en-mt-3" $:if="submitted" $:click="reset()">
      Register Another Account
    </button>
  </div>
</div>
```

```html
<div id="demo-ajax-form">
  <h3>User Registration</h3>
  
  <!-- Success Message -->
  <div $:if="submitted">
    Success! Account created for <strong $:text="form.username"></strong>
  </div>

  <!-- Registration Form -->
  <form $:submit.prevent="handleSubmit" $:if="!submitted">
    <!-- Username -->
    <div>
      <label>
        Username *
        <span $:if="checkingUsername">(checking...)</span>
      </label>
      <input $:model.trim="form.username" placeholder="Choose a username">
      <div $:if="errors.username" $:text="errors.username"></div>
      <div $:if="!errors.username && form.username && !checkingUsername && usernameAvailable">
        ✓ Username available
      </div>
    </div>

    <!-- Email -->
    <div>
      <label>Email *</label>
      <input type="email" $:model.trim="form.email" placeholder="your@email.com">
      <div $:if="errors.email" $:text="errors.email"></div>
    </div>

    <!-- Password -->
    <div>
      <label>Password *</label>
      <input type="password" $:model="form.password" placeholder="Minimum 8 characters">
      <div $:if="errors.password" $:text="errors.password"></div>
    </div>

    <!-- Confirm Password -->
    <div>
      <label>Confirm Password *</label>
      <input type="password" $:model="form.confirmPassword" placeholder="Re-enter password">
      <div $:if="errors.confirmPassword" $:text="errors.confirmPassword"></div>
    </div>

    <!-- Terms -->
    <label>
      <input type="checkbox" $:model="form.agreeToTerms">
      I agree to the terms and conditions *
    </label>

    <!-- Submit Button -->
    <button type="submit" $:attr:disabled="!isValid || submitting">
      <span $:if="!submitting">Create Account</span>
      <span $:if="submitting">Creating...</span>
    </button>

    <!-- Server Error -->
    <div $:if="serverError">
      <strong>Error:</strong> <span $:text="serverError"></span>
    </div>
  </form>

  <!-- Reset Button -->
  <button $:if="submitted" $:click="reset()">Register Another Account</button>
</div>
```

```js inject show
$('#demo-ajax-form').component({
  state: {
    form: {
      username: '',
      email: '',
      password: '',
      confirmPassword: '',
      agreeToTerms: false
    },
    errors: {},
    checkingUsername: false,
    usernameAvailable: false,
    submitting: false,
    submitted: false,
    serverError: ''
  },
  
  computed: {
    isValid: (c) => {
      return (
        c.form.username.length >= 3 &&
        c.form.email.includes('@') &&
        c.form.password.length >= 8 &&
        c.form.password === c.form.confirmPassword &&
        c.form.agreeToTerms &&
        Object.keys(c.errors).length === 0 &&
        !c.checkingUsername
      );
    }
  },
  
  watch: {
    'form.username': {
      async handler(newVal) {
        this.errors.username = '';
        
        if (!newVal) {
          this.usernameAvailable = false;
          return;
        }
        
        if (newVal.length < 3) {
          this.errors.username = 'Username must be at least 3 characters';
          this.usernameAvailable = false;
          return;
        }
        
        // Check availability (simulated API call)
        this.checkingUsername = true;
        await new Promise(resolve => setTimeout(resolve, 500));
        
        // Simulate: 'admin' is taken
        if (newVal.toLowerCase() === 'admin') {
          this.errors.username = 'Username is already taken';
          this.usernameAvailable = false;
        } else {
          this.usernameAvailable = true;
        }
        
        this.checkingUsername = false;
      },
      debounce: 500
    },
    
    'form.email'(newVal) {
      if (!newVal) {
        this.errors.email = '';
      } else if (!newVal.includes('@') || !newVal.includes('.')) {
        this.errors.email = 'Please enter a valid email address';
      } else {
        this.errors.email = '';
      }
    },
    
    'form.password'(newVal) {
      if (!newVal) {
        this.errors.password = '';
      } else if (newVal.length < 8) {
        this.errors.password = 'Password must be at least 8 characters';
      } else {
        this.errors.password = '';
      }
      
      // Re-validate confirm password
      if (this.form.confirmPassword) {
        this.validateConfirmPassword();
      }
    },
    
    'form.confirmPassword'() {
      this.validateConfirmPassword();
    }
  },
  
  methods: {
    validateConfirmPassword() {
      if (!this.form.confirmPassword) {
        this.errors.confirmPassword = '';
      } else if (this.form.password !== this.form.confirmPassword) {
        this.errors.confirmPassword = 'Passwords do not match';
      } else {
        this.errors.confirmPassword = '';
      }
    },
    
    async handleSubmit() {
      if (!this.computed.isValid) return;
      
      this.submitting = true;
      this.serverError = '';
      
      try {
        // Simulate API call
        await new Promise(resolve => setTimeout(resolve, 1500));
        
        // Simulate 10% chance of server error
        if (Math.random() < 0.1) {
          throw new Error('Server error. Please try again.');
        }
        
        // Success
        console.log('Form submitted:', this.form);
        this.submitted = true;
      } catch (error) {
        this.serverError = error.message;
      } finally {
        this.submitting = false;
      }
    },
    
    reset() {
      this.form = {
        username: '',
        email: '',
        password: '',
        confirmPassword: '',
        agreeToTerms: false
      };
      this.errors = {};
      this.submitted = false;
      this.serverError = '';
      this.usernameAvailable = false;
    }
  }
});
```

```js cleanup
onUnmount(() => {
  $('#demo-ajax-form').component('destroy');
})
```

## Code Breakdown

### State Structure

```js
state: {
  form: {
    username: '',
    email: '',
    password: '',
    confirmPassword: '',
    agreeToTerms: false
  },
  errors: {},                    // Field-level errors
  checkingUsername: false,       // Loading state for async check
  usernameAvailable: false,      // Username availability result
  submitting: false,             // Form submission loading
  submitted: false,              // Success state
  serverError: ''                // Server-side error
}
```

### Async Username Validation

```js
watch: {
  'form.username': {
    async handler(newVal) {
      this.errors.username = '';
      
      if (newVal.length < 3) {
        this.errors.username = 'Username must be at least 3 characters';
        return;
      }
      
      // Check availability
      this.checkingUsername = true;
      const available = await checkUsername(newVal);
      this.checkingUsername = false;
      
      if (!available) {
        this.errors.username = 'Username is already taken';
      }
    },
    debounce: 500  // Wait 500ms after user stops typing
  }
}
```

### Real-time Validation

```js
watch: {
  'form.email'(newVal) {
    if (!newVal) {
      this.errors.email = '';
    } else if (!newVal.includes('@')) {
      this.errors.email = 'Please enter a valid email';
    } else {
      this.errors.email = '';
    }
  },
  
  'form.password'(newVal) {
    if (newVal && newVal.length < 8) {
      this.errors.password = 'Password must be at least 8 characters';
    } else {
      this.errors.password = '';
    }
  }
}
```

### Form Validation

```js
computed: {
  isValid: (c) => {
    return (
     c.form.username.length >= 3 &&
     c.form.email.includes('@') &&
     c.form.password.length >= 8 &&
     c.form.password === c.form.confirmPassword &&
     c.form.agreeToTerms &&
      Object.keys(s.errors).length === 0 &&
      !s.checkingUsername
    );
  }
}
```

### Form Submission

```js
methods: {
  async handleSubmit(state) {
    if (!this.computed.isValid) return;
    
    state.submitting = true;
    state.serverError = '';
    
    try {
      const response = await fetch('/api/register', {
        method: 'POST',
        headers: { 'Content-Type': 'application/json' },
        body: JSON.stringify(state.form)
      });
      
      if (!response.ok) {
        throw new Error(await response.text());
      }
      
      state.submitted = true;
    } catch (error) {
      state.serverError = error.message;
    } finally {
      state.submitting = false;
    }
  }
}
```

## Key Concepts

### 1. Debounced Async Validation

```js
watch: {
  'form.username': {
    async handler(newVal) {
      // API call...
    },
    debounce: 500  // Wait 500ms after typing stops
  }
}
```

**Why debounce?** Prevents API spam while user is still typing.

### 2. Loading States

```html
<span $:if="checkingUsername">(checking...)</span>
<button $:attr:disabled="submitting">
  <span $:if="submitting">Submitting...</span>
  <span $:if="!submitting">Submit</span>
</button>
```

**Provides feedback** during async operations.

### 3. Field-Level Errors

```js
state: {
  errors: {
    username: '',
    email: '',
    password: ''
  }
}
```

```html
<div $:if="errors.username" $:text="errors.username"></div>
```

**Clear, specific error messages** for each field.

### 4. Server Error Handling

```js
try {
  await submitForm();
} catch (error) {
  state.serverError = error.message;
}
```

```html
<div $:if="serverError">
  Error: <span $:text="serverError"></span>
</div>
```

### 5. Success State

```html
<div $:if="submitted">
  Success! Account created.
</div>

<form $:if="!submitted">
  <!-- Form fields -->
</form>
```

**Hide form** after successful submission.

## Real API Integration

### Fetch API

```js
async checkUsername(username) {
  const response = await fetch(`/api/check-username?name=${username}`);
  const data = await response.json();
  return data.available;
}

async handleSubmit(state) {
  const response = await fetch('/api/register', {
    method: 'POST',
    headers: { 'Content-Type': 'application/json' },
    body: JSON.stringify(state.form)
  });
  
  if (!response.ok) {
    throw new Error(await response.text());
  }
  
  return response.json();
}
```

### jQuery AJAX

```js
async checkUsername(username) {
  const data = await $.get('/api/check-username', { name: username });
  return data.available;
}

async handleSubmit(state) {
  try {
    await $.ajax({
      url: '/api/register',
      method: 'POST',
      contentType: 'application/json',
      data: JSON.stringify(state.form)
    });
    state.submitted = true;
  } catch (xhr) {
    state.serverError = xhr.responseText;
  }
}
```

### Axios

```js
async checkUsername(username) {
  const { data } = await axios.get('/api/check-username', {
    params: { name: username }
  });
  return data.available;
}

async handleSubmit(state) {
  try {
    await axios.post('/api/register', state.form);
    state.submitted = true;
  } catch (error) {
    state.serverError = error.response.data.message;
  }
}
```

## Validation Patterns

### Email Validation

```js
function isValidEmail(email) {
  return /^[^\s@]+@[^\s@]+\.[^\s@]+$/.test(email);
}

watch: {
  'form.email'(newVal) {
    if (newVal && !isValidEmail(newVal)) {
      this.errors.email = 'Invalid email address';
    } else {
      this.errors.email = '';
    }
  }
}
```

### Password Strength

```js
function getPasswordStrength(password) {
  if (password.length < 8) return 'weak';
  if (password.length < 12) return 'medium';
  if (/[A-Z]/.test(password) && /[0-9]/.test(password)) return 'strong';
  return 'medium';
}

computed: {
  passwordStrength: (s) => getPasswordStrength(s.form.password)
}
```

```html
<div>
  <span $:if="passwordStrength === 'weak'">❌ Weak</span>
  <span $:if="passwordStrength === 'medium'">⚠️ Medium</span>
  <span $:if="passwordStrength === 'strong'">✅ Strong</span>
</div>
```

## Best Practices

### 1. Debounce Expensive Validation

```js
// ✅ Good - wait for user to finish typing
watch: {
  'form.username': {
    async handler(newVal) {
      // API call...
    },
    debounce: 500
  }
}

// ❌ Bad - API call on every keystroke
watch: {
  'form.username': async function(newVal) {
    // API call...
  }
}
```

### 2. Clear Errors on Input

```js
watch: {
  'form.email'(newVal) {
    // Clear error when user starts typing
    this.errors.email = '';
    
    // Then validate
    if (newVal && !isValidEmail(newVal)) {
      this.errors.email = 'Invalid email';
    }
  }
}
```

### 3. Disable Submit During Submission

```html
<button $:attr:disabled="!isValid || submitting">
  Submit
</button>
```

### 4. Provide Loading Feedback

```html
<span $:if="checkingUsername">Checking availability...</span>
<span $:if="submitting">Submitting form...</span>
```

### 5. Handle Server Errors

```js
try {
  await submitForm();
} catch (error) {
  state.serverError = error.message || 'An error occurred';
}
```

## Next Steps

- [Async State Guide](../guides/async-state.md) - Managing async data
- [Forms Guide](../guides/forms-and-models.md) - Form handling patterns
- [Model Modifiers](../guides/model-modifiers.md) - Input modifiers
- [Modal Dialog Example](./modal-dialog.md) - Modal patterns
