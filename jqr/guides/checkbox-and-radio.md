<!-- {
  "path": "jqr/guides/checkbox-and-radio",
  "name": "docs-jqr-guides-checkbox-and-radio",
  "meta": {
    "title": "Checkbox & Radio",
    "order": 14
  }
} -->

# Checkbox & Radio

Checkboxes and radio buttons are essential form controls. JQR provides powerful data binding for both single and grouped inputs through the `$:model` directive.

## Single Checkbox (Boolean)

A single checkbox binds to a boolean value:

```html live:no-code
<div id="demo-single-checkbox" class="en-card">
  <div class="en-card__body">
    <h3>Terms & Conditions</h3>
    <label class="en-check en-block">
      <input type="checkbox" $:model="agreed">
      <span>I agree to the terms and conditions</span>
    </label>
    <button 
      class="en-btn en-btn--primary" 
      $:attr:disabled="!agreed"
      style="margin-top: 1rem;">
      Continue
    </button>
    <div class="en-text--muted" style="margin-top: 0.5rem;">
      Value: <code $:text="agreed"></code> (type: <code $:text="typeof agreed"></code>)
    </div>
  </div>
</div>
```

```html
<div id="demo-single-checkbox">
  <h3>Terms & Conditions</h3>
  <label>
    <input type="checkbox" $:model="agreed">
    I agree to the terms and conditions
  </label>
  <button $:attr:disabled="!agreed">Continue</button>
  <p>Value: <code $:text="agreed"></code></p>
</div>
```

```js inject show
$('#demo-single-checkbox').component({
  state: { agreed: false }
});
```

```js cleanup
onUnmount(() => {
  $('#demo-single-checkbox').component('destroy');
})
```

**How it works:**
- Unchecked: `agreed = false`
- Checked: `agreed = true`

## Multiple Checkboxes (Array)

Multiple checkboxes with the same model bind to an array:

```html live:no-code
<div id="demo-multiple-checkbox" class="en-card">
  <div class="en-card__body">
    <h3>Select Skills</h3>
    <div class="en-segmented en-segmented--accent en-segmented--pills en-mb-3">
      <label>
        <input type="checkbox" value="javascript" $:model="skills">
        <span>JavaScript</span>
      </label>
      <label>
        <input type="checkbox" value="typescript" $:model="skills">
        <span>TypeScript</span>
      </label>
      <label>
        <input type="checkbox" value="react" $:model="skills">
        <span>React</span>
      </label>
      <label>
        <input type="checkbox" value="vue" $:model="skills">
        <span>Vue</span>
      </label>
      <label>
        <input type="checkbox" value="angular" $:model="skills">
        <span>Angular</span>
      </label>
    </div>
    <div class="en-alert en-alert--info en-mb-3">
      <strong>Selected:</strong> 
      <span $:if="skills.length === 0">None</span>
      <span $:if="skills.length > 0" $:text="skills.join(', ')"></span>
    </div>
    <div class="en-text--muted">
      Array: <code $:text="JSON.stringify(skills)"></code>
    </div>
  </div>
</div>
```

```html
<div id="demo-multiple-checkbox">
  <h3>Select Skills</h3>
  
  <label>
    <input type="checkbox" value="javascript" $:model="skills">
    JavaScript
  </label>
  
  <label>
    <input type="checkbox" value="typescript" $:model="skills">
    TypeScript
  </label>
  
  <label>
    <input type="checkbox" value="react" $:model="skills">
    React
  </label>
  
  <label>
    <input type="checkbox" value="vue" $:model="skills">
    Vue
  </label>
  
  <p><strong>Selected:</strong> <span $:text="skills.join(', ')"></span></p>
  <p>Array: <code $:text="JSON.stringify(skills)"></code></p>
</div>
```

```js inject show
$('#demo-multiple-checkbox').component({
  state: { skills: ['javascript'] }  // Pre-select JavaScript
});
```

```js cleanup
onUnmount(() => {
  $('#demo-multiple-checkbox').component('destroy');
})
```

**How it works:**
1. State must be an **array**: `state: { skills: [] }`
2. Each checkbox has a unique `value` attribute
3. Checking adds value to array: `['javascript', 'react']`
4. Unchecking removes value from array

## Radio Buttons

Radio buttons allow selecting one value from a group:

```html live:no-code
<div id="demo-radio" class="en-card">
  <div class="en-card__body">
    <h3>Choose Size</h3>
    <div class="en-segmented en-segmented--accent en-segmented--pills en-mb-3">
      <label>
        <input type="radio" value="sm" $:model="size" name="size">
        <span>Small</span>
      </label>
      <label>
        <input type="radio" value="base" $:model="size" name="size">
        <span>Medium</span>
      </label>
      <label>
        <input type="radio" value="lg" $:model="size" name="size">
        <span>Large</span>
      </label>
      <label>
        <input type="radio" value="xl" $:model="size" name="size">
        <span>X-Large</span>
      </label>
    </div>
    <div class="en-alert en-alert--success en-mb-3" $:class="[`en-text-${size}`]">
      <strong>Selected size:</strong> <span $:text="size || 'None'"></span>
    </div>
    <div class="en-text--muted">
      Value: <code $:text="size"></code> (type: <code $:text="typeof size"></code>)
    </div>
  </div>
</div>
```

```html
<div id="demo-radio">
  <h3>Choose Size</h3>
  
  <label>
    <input type="radio" value="small" $:model="size" name="size">
    Small
  </label>
  
  <label>
    <input type="radio" value="medium" $:model="size" name="size">
    Medium
  </label>
  
  <label>
    <input type="radio" value="large" $:model="size" name="size">
    Large
  </label>
  
  <p><strong>Selected:</strong> <span $:text="size"></span></p>
</div>
```

```js inject show
$('#demo-radio').component({
  state: { size: 'medium' }  // Pre-select medium
});
```

```js cleanup
onUnmount(() => {
  $('#demo-radio').component('destroy');
})
```

**Key points:**
- All radios in a group must have the same `name` attribute
- Each radio has a unique `value` attribute
- State stores the selected value as a string
- Only one radio can be selected at a time

## Dynamic Options

Generate checkboxes or radios dynamically:

```html live:no-code
<div id="demo-dynamic-options" class="en-card">
  <div class="en-card__body">
    <h3>Select Colors</h3>
    <div class="en-segmented en" style="margin-bottom: 1rem;">
      <label 
        $:for="color in colors" 
        $:key="color.value"
        class="en-flex en-gap-2 en-items-center">
        <input type="checkbox" $:attr:value="color.value" $:model="selectedColors">
        <span $:text="color.label"></span>
      </label>
    </div>
    <div class="en-alert en-alert--secondary">
      <strong>Selected:</strong> 
      <span $:if="selectedColors.length === 0">None</span>
      <span $:if="selectedColors.length > 0" $:text="selectedColors.join(', ')"></span>
    </div>
  </div>
</div>
```

```html
<div id="demo-dynamic-options">
  <h3>Select Colors</h3>
  
  <label 
    $:for="color in colors" 
    $:key="color.value">
    <input type="checkbox" $:attr:value="color.value" $:model="selectedColors">
    <span $:text="color.label"></span>
  </label>
  
  <p><strong>Selected:</strong> <span $:text="selectedColors.join(', ')"></span></p>
</div>
```

```js inject show
$('#demo-dynamic-options').component({
  state: {
    colors: [
      { label: 'Red', value: 'danger' },
      { label: 'Green', value: 'success' },
      { label: 'Blue', value: 'primary' },
      { label: 'Yellow', value: 'warning' },
    ],
    selectedColors: []
  }
});
```

```js cleanup
onUnmount(() => {
  $('#demo-dynamic-options').component('destroy');
})
```

**Important:** Use `$:attr:value` (not plain `value`) when binding dynamic values in loops!

## Select All / None

Implement select all functionality:

```html live:no-code
<div id="demo-select-all" class="en-card">
  <div class="en-card__body">
    <h3>Select Features</h3>
    
    <div class="en-flex en-gap-2" style="margin-bottom: 1rem;">
      <button class="en-btn en-btn--sm" $:click="selectAll">Select All</button>
      <button class="en-btn en-btn--sm en-btn--outline" $:click="selectNone">Select None</button>
    </div>
    
    <div class="en-flex en-gap-3 en-flex-wrap" style="margin-bottom: 1rem;">
      <label 
        $:for="feature in availableFeatures" 
        $:key="feature"
        class="en-flex en-gap-2 en-items-center">
        <input type="checkbox" $:attr:value="feature" $:model="selectedFeatures">
        <span $:text="feature"></span>
      </label>
    </div>
    
    <div class="en-alert en-alert--info">
      <strong>Selected <span $:text="selectedFeatures.length"></span> of <span $:text="availableFeatures.length"></span> features</strong>
    </div>
  </div>
</div>
```

```html
<div id="demo-select-all">
  <h3>Select Features</h3>
  
  <button $:click="selectAll">Select All</button>
  <button $:click="selectNone">Select None</button>
  
  <label 
    $:for="feature in availableFeatures" 
    $:key="feature">
    <input type="checkbox" $:attr:value="feature" $:model="selectedFeatures">
    <span $:text="feature"></span>
  </label>
  
  <p>Selected: <span $:text="selectedFeatures.length"></span></p>
</div>
```

```js inject show
$('#demo-select-all').component({
  state: {
    availableFeatures: ['Dark Mode', 'Notifications', 'Auto Save', 'Spell Check', 'Cloud Sync'],
    selectedFeatures: []
  },
  methods: {
    selectAll(state) {
      state.selectedFeatures = [...state.availableFeatures];
    },
    selectNone(state) {
      state.selectedFeatures = [];
    }
  }
});
```

```js cleanup
onUnmount(() => {
  $('#demo-select-all').component('destroy');
})
```

## Indeterminate Checkbox

Master checkbox with indeterminate state:

```html live:no-code
<div id="demo-indeterminate" class="en-card">
  <div class="en-card__body">
    <h3>Select Items</h3>
    
    <label class="en-check">
      <input 
        type="checkbox" 
        $:model="allSelected"
        $:indeterminate="someSelected && !allSelected">
      <strong>Select All</strong>
    </label>
    
    <div class="en-flex en-gap-2 en-mb-3">
      <label 
        $:for="item in items" 
        $:key="item.id"
        class="en-check">
        <input type="checkbox" $:attr:value="item.id" $:model="selectedItems">
        <span $:text="item.name"></span>
      </label>
    </div>
    
    <div class="en-alert en-alert--info" style="margin-top: 1rem;">
      Selected: <span $:text="selectedItems.length"></span> / <span $:text="items.length"></span>
    </div>
  </div>
</div>
```

```html
<div id="demo-indeterminate">
  <h3>Select Items</h3>
  
  <label>
    <input 
      type="checkbox" 
      $:model="allSelected"
      $:indeterminate="someSelected && !allSelected">
    Select All
  </label>
  
  <label 
    $:for="item in items" 
    $:key="item.id">
    <input type="checkbox" $:attr:value="item.id" $:model="selectedItems">
    <span $:text="item.name"></span>
  </label>
  
  <p>Selected: <span $:text="selectedItems.length"></span></p>
</div>
```

```js inject show
$('#demo-indeterminate').component({
  state: {
    items: [
      { id: 1, name: 'Item 1' },
      { id: 2, name: 'Item 2' },
      { id: 3, name: 'Item 3' },
      { id: 4, name: 'Item 4' }
    ],
    selectedItems: []
  },
  computed: {
    allSelected: {
      get(state) {
        return state.selectedItems.length === state.items.length;
      },
      set(state, value) {
        state.selectedItems = value 
          ? state.items.map(item => item.id)
          : [];
      }
    },
    someSelected(state) {
      return state.selectedItems.length > 0 && 
             state.selectedItems.length < state.items.length;
    }
  }
});
```

```js cleanup
onUnmount(() => {
  $('#demo-indeterminate').component('destroy');
})
```

**Key techniques:**
- Use computed with getter/setter for master checkbox
- `$:indeterminate` directive for visual state
- Master checkbox controls all child checkboxes

## True/False Values

Customize true/false values for single checkboxes:

```html live:no-code
<div id="demo-custom-values" class="en-card">
  <div class="en-card__body">
    <h3>Privacy Settings</h3>
    <label class="en-flex en-gap-2 en-items-center">
      <input 
        type="checkbox" 
        $:model="visibility"
        $:true-value="'public'"
        $:false-value="'private'">
      <span>Make profile public</span>
    </label>
    <div class="en-alert en-alert--info" style="margin-top: 1rem;">
      <strong>Visibility:</strong> <code $:text="visibility"></code>
    </div>
  </div>
</div>
```

```html
<div id="demo-custom-values">
  <h3>Privacy Settings</h3>
  <label>
    <input 
      type="checkbox" 
      $:model="visibility"
      $:true-value="'public'"
      $:false-value="'private'">
    Make profile public
  </label>
  <p><strong>Visibility:</strong> <code $:text="visibility"></code></p>
</div>
```

```js inject show
$('#demo-custom-values').component({
  state: { visibility: 'private' }
});
```

```js cleanup
onUnmount(() => {
  $('#demo-custom-values').component('destroy');
})
```

**Custom values:**
- `$:true-value`: Value when checked (default: `true`)
- `$:false-value`: Value when unchecked (default: `false`)

## Disabled State

Disable checkboxes and radios conditionally:

```html live:no-code
<div id="demo-disabled" class="en-card">
  <div class="en-card__body">
    <h3>Premium Features</h3>
    
    <label class="en-flex en-gap-2 en-items-center" style="margin-bottom: 1rem;">
      <input type="checkbox" $:model="isPremium">
      <strong>Enable Premium Account</strong>
    </label>
    
    <div class="en-flex en-flex-col en-gap-2" style="margin-left: 1.5rem;">
      <label class="en-flex en-gap-2 en-items-center">
        <input 
          type="checkbox" 
          $:model="features"
          value="cloud-storage"
          $:attr:disabled="!isPremium">
        <span $:class="{ 'en-text--muted': !isPremium }">Cloud Storage</span>
      </label>
      
      <label class="en-flex en-gap-2 en-items-center">
        <input 
          type="checkbox" 
          $:model="features"
          value="priority-support"
          $:attr:disabled="!isPremium">
        <span $:class="{ 'en-text--muted': !isPremium }">Priority Support</span>
      </label>
      
      <label class="en-flex en-gap-2 en-items-center">
        <input 
          type="checkbox" 
          $:model="features"
          value="advanced-analytics"
          $:attr:disabled="!isPremium">
        <span $:class="{ 'en-text--muted': !isPremium }">Advanced Analytics</span>
      </label>
    </div>
    
    <div class="en-alert en-alert--info" style="margin-top: 1rem;" $:if="features.length > 0">
      <strong>Enabled features:</strong> <span $:text="features.join(', ')"></span>
    </div>
  </div>
</div>
```

```html
<div id="demo-disabled">
  <h3>Premium Features</h3>
  
  <label>
    <input type="checkbox" $:model="isPremium">
    Enable Premium Account
  </label>
  
  <label>
    <input 
      type="checkbox" 
      $:model="features"
      value="cloud-storage"
      $:attr:disabled="!isPremium">
    Cloud Storage
  </label>
  
  <label>
    <input 
      type="checkbox" 
      $:model="features"
      value="priority-support"
      $:attr:disabled="!isPremium">
    Priority Support
  </label>
  
  <p $:if="features.length > 0">
    Enabled: <span $:text="features.join(', ')"></span>
  </p>
</div>
```

```js inject show
$('#demo-disabled').component({
  state: {
    isPremium: false,
    features: []
  }
});
```

```js cleanup
onUnmount(() => {
  $('#demo-disabled').component('destroy');
})
```

## Validation

Validate checkbox/radio selections:

```html live:no-code
<div id="demo-validation" class="en-card">
  <div class="en-card__body">
    <h3>Survey Form</h3>
    
    <div class="en-form-group">
      <label class="en-label">Which languages do you know? (Select at least 2)</label>
      <div class="en-flex en-gap-3 en-flex-wrap">
        <label 
          $:for="lang in languages" 
          $:key="lang"
          class="en-flex en-gap-2 en-items-center">
          <input type="checkbox" $:attr:value="lang" $:model="selectedLanguages">
          <span $:text="lang"></span>
        </label>
      </div>
      <div class="en-form-help en-text--danger" $:if="showError && selectedLanguages.length < 2">
        Please select at least 2 languages
      </div>
    </div>
    
    <div class="en-form-group">
      <label class="en-label">Experience level (Required)</label>
      <div class="en-flex en-gap-3">
        <label 
          $:for="level in levels" 
          $:key="level"
          class="en-flex en-gap-2 en-items-center">
          <input type="radio" $:attr:value="level" $:model="experience" name="experience">
          <span $:text="level"></span>
        </label>
      </div>
      <div class="en-form-help en-text--danger" $:if="showError && !experience">
        Please select your experience level
      </div>
    </div>
    
    <button class="en-btn en-btn--primary" $:click="submit">Submit</button>
    
    <div class="en-alert en-alert--success" style="margin-top: 1rem;" $:if="submitted">
      <strong>Form submitted successfully!</strong>
    </div>
  </div>
</div>
```

```html
<div id="demo-validation">
  <h3>Survey Form</h3>
  
  <label>Which languages do you know? (Select at least 2)</label>
  <label 
    $:for="lang in languages" 
    $:key="lang">
    <input type="checkbox" $:attr:value="lang" $:model="selectedLanguages">
    <span $:text="lang"></span>
  </label>
  <div $:if="showError && selectedLanguages.length < 2">
    Please select at least 2 languages
  </div>
  
  <label>Experience level (Required)</label>
  <label 
    $:for="level in levels" 
    $:key="level">
    <input type="radio" $:attr:value="level" $:model="experience" name="experience">
    <span $:text="level"></span>
  </label>
  <div $:if="showError && !experience">
    Please select your experience level
  </div>
  
  <button $:click="submit">Submit</button>
</div>
```

```js inject show
$('#demo-validation').component({
  state: {
    languages: ['JavaScript', 'Python', 'Java', 'C++', 'Ruby'],
    selectedLanguages: [],
    levels: ['Beginner', 'Intermediate', 'Advanced', 'Expert'],
    experience: '',
    showError: false,
    submitted: false
  },
  methods: {
    submit(state) {
      state.showError = true;
      
      const isValid = state.selectedLanguages.length >= 2 && state.experience;
      
      if (isValid) {
        state.submitted = true;
        state.showError = false;
        
        // Reset after 3 seconds
        setTimeout(() => {
          state.submitted = false;
          state.selectedLanguages = [];
          state.experience = '';
        }, 3000);
      }
    }
  }
});
```

```js cleanup
onUnmount(() => {
  $('#demo-validation').component('destroy');
})
```

## Best Practices

### 1. Always Use Arrays for Multiple Checkboxes

```js
// ✅ Good - Array for multiple checkboxes
state: { skills: [] }

// ❌ Bad - Boolean won't work for multiple values
state: { skills: false }
```

### 2. Use Unique `value` Attributes

```html
<!-- ✅ Good - Unique values -->
<input type="checkbox" value="javascript" $:model="skills">
<input type="checkbox" value="python" $:model="skills">

<!-- ❌ Bad - Same values -->
<input type="checkbox" value="lang" $:model="skills">
<input type="checkbox" value="lang" $:model="skills">
```

### 3. Group Radio Buttons with `name`

```html
<!-- ✅ Good - Same name for group -->
<input type="radio" value="s" $:model="size" name="size">
<input type="radio" value="m" $:model="size" name="size">

<!-- ❌ Bad - Different names -->
<input type="radio" value="s" $:model="size" name="size1">
<input type="radio" value="m" $:model="size" name="size2">
```

### 4. Use `$:attr:value` in Loops

```html
<!-- ✅ Good - Dynamic binding -->
<label $:for="item in items">
  <input type="checkbox" $:attr:value="item.id" $:model="selected">
  <span $:text="item.name"></span>
</label>

<!-- ❌ Bad - Static value -->
<label $:for="item in items">
  <input type="checkbox" value="item.id" $:model="selected">
</label>
```

### 5. Provide Visual Feedback

```html
<!-- ✅ Good - Show selection count -->
<p>Selected: <span $:text="selected.length"></span> items</p>

<!-- ✅ Good - Disable dependent controls -->
<button $:attr:disabled="selected.length === 0">
  Process Selected
</button>
```

## Common Patterns

### Checkbox List with Actions

```js
state: {
  items: [
    { id: 1, name: 'Item 1' },
    { id: 2, name: 'Item 2' }
  ],
  selected: []
},
methods: {
  deleteSelected(state) {
    state.items = state.items.filter(
      item => !state.selected.includes(item.id)
    );
    state.selected = [];
  }
}
```

### Toggle All with Master Checkbox

```js
computed: {
  allChecked: {
    get(state) {
      return state.items.length > 0 && 
             state.selected.length === state.items.length;
    },
    set(state, value) {
      state.selected = value 
        ? state.items.map(i => i.id) 
        : [];
    }
  }
}
```

### Radio with "Other" Option

```html
<label>
  <input type="radio" value="other" $:model="choice" name="choice">
  Other
</label>
<input 
  type="text" 
  $:model="otherValue"
  $:attr:disabled="choice !== 'other'"
  placeholder="Please specify...">
```

## Performance Tips

**For large lists:**
- Use `$:key` in loops for efficient rendering
- Consider virtual scrolling for 100+ items
- Debounce search/filter operations

```js
state: {
  allItems: [...], // Large array
  searchQuery: ''
},
computed: {
  filteredItems(state) {
    if (!state.searchQuery) return state.allItems;
    return state.allItems.filter(item => 
      item.name.includes(state.searchQuery)
    );
  }
}
```

## Next Steps

- [Model Modifiers](./model-modifiers.md) - Transform input values
- [Forms & Models](./forms-and-models.md) - Complete form handling
- [Events](./events.md) - Handle user interactions
- [DOM Binding](./dom-binding.md) - Conditional rendering & styling
