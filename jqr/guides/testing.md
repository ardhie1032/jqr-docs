<!-- {
  "path": "jqr/guides/testing",
  "name": "docs-jqr-guides-testing",
  "meta": {
    "title": "Testing Components",
    "order": 31
  }
} -->

# Testing Components

Testing JQR components ensures your application behaves correctly and helps prevent regressions. This guide covers strategies and patterns for testing JQR components effectively.

## Testing Approaches

### 1. Unit Testing

Test individual component logic in isolation:

```js
// counter.test.js
import { $ } from '@en32/jqr';

describe('Counter Component', () => {
  let component;
  
  beforeEach(() => {
    // Create test container
    document.body.innerHTML = '<div id="test"></div>';
    
    // Mount component
    component = $('#test').component({
      state: { count: 0 },
      methods: {
        increment(state) {
          state.count++;
        }
      }
    });
  });
  
  afterEach(() => {
    // Cleanup
    component.destroy();
    document.body.innerHTML = '';
  });
  
  test('initial state', () => {
    expect(component.state.count).toBe(0);
  });
  
  test('increment method', () => {
    component.options.methods.increment(component.state);
    expect(component.state.count).toBe(1);
  });
});
```

### 2. Integration Testing

Test component interactions with DOM:

```js
describe('Counter Component Integration', () => {
  beforeEach(() => {
    document.body.innerHTML = `
      <div id="test">
        <p id="count" $:text="count"></p>
        <button id="inc" $:click="increment()">+</button>
      </div>
    `;
    
    $('#test').component({
      state: { count: 0 },
      methods: {
        increment(state) {
          state.count++;
        }
      }
    });
  });
  
  test('button click updates DOM', () => {
    const button = document.getElementById('inc');
    const display = document.getElementById('count');
    
    expect(display.textContent).toBe('0');
    
    button.click();
    expect(display.textContent).toBe('1');
    
    button.click();
    expect(display.textContent).toBe('2');
  });
});
```

### 3. End-to-End Testing

Test full user flows (using Playwright, Cypress, etc.):

```js
// e2e/counter.spec.js (Playwright)
import { test, expect } from '@playwright/test';

test('counter increments correctly', async ({ page }) => {
  await page.goto('/counter.html');
  
  // Initial state
  await expect(page.locator('#count')).toHaveText('0');
  
  // Click button
  await page.click('#inc');
  await expect(page.locator('#count')).toHaveText('1');
  
  // Multiple clicks
  await page.click('#inc');
  await page.click('#inc');
  await expect(page.locator('#count')).toHaveText('3');
});
```

## Testing with Vitest

### Setup

```bash
npm install -D vitest jsdom @testing-library/dom
```

**vitest.config.js:**

```js
import { defineConfig } from 'vitest/config';

export default defineConfig({
  test: {
    environment: 'jsdom',
    globals: true,
    setupFiles: './test/setup.js'
  }
});
```

**test/setup.js:**

```js
import { $ } from '@en32/jqr';
import '@testing-library/jest-dom';

// Make $ available globally
global.$ = $;

// Add custom matchers if needed
expect.extend({
  toHaveState(component, expected) {
    const pass = Object.keys(expected).every(
      key => component.state[key] === expected[key]
    );
    
    return {
      pass,
      message: () => `Expected state ${JSON.stringify(component.state)} to match ${JSON.stringify(expected)}`
    };
  }
});
```

### Basic Tests

```js
// components/greeting.test.js
import { describe, test, expect, beforeEach, afterEach } from 'vitest';
import { $ } from '@en32/jqr';

describe('Greeting Component', () => {
  let container;
  let component;
  
  beforeEach(() => {
    container = document.createElement('div');
    container.innerHTML = `
      <div id="greeting">
        <p $:text="message"></p>
        <button $:click="updateMessage()">Update</button>
      </div>
    `;
    document.body.appendChild(container);
    
    component = $('#greeting').component({
      state: {
        message: 'Hello'
      },
      methods: {
        updateMessage(state) {
          state.message = 'Updated!';
        }
      }
    });
  });
  
  afterEach(() => {
    component.destroy();
    container.remove();
  });
  
  test('renders initial message', () => {
    const p = container.querySelector('p');
    expect(p.textContent).toBe('Hello');
  });
  
  test('updates message on button click', () => {
    const button = container.querySelector('button');
    const p = container.querySelector('p');
    
    button.click();
    
    expect(p.textContent).toBe('Updated!');
    expect(component.state.message).toBe('Updated!');
  });
});
```

## Testing Patterns

### Testing State

```js
test('state changes correctly', () => {
  const component = $('#test').component({
    state: { count: 0, name: 'Test' }
  });
  
  // Test initial state
  expect(component.state.count).toBe(0);
  expect(component.state.name).toBe('Test');
  
  // Modify state
  component.state.count = 5;
  expect(component.state.count).toBe(5);
  
  component.destroy();
});
```

### Testing Computed Properties

```js
test('computed property calculates correctly', () => {
  const component = $('#test').component({
    state: {
      firstName: 'John',
      lastName: 'Doe'
    },
    computed: {
      fullName(state) {
        return `${state.firstName} ${state.lastName}`;
      }
    }
  });
  
  expect(component.fullName).toBe('John Doe');
  
  component.state.firstName = 'Jane';
  expect(component.fullName).toBe('Jane Doe');
  
  component.destroy();
});
```

### Testing Methods

```js
test('methods execute correctly', () => {
  const component = $('#test').component({
    state: { items: [] },
    methods: {
      addItem(state, item) {
        state.items.push(item);
      },
      clearItems(state) {
        state.items = [];
      }
    }
  });
  
  // Test addItem
  component.options.methods.addItem(component.state, 'Item 1');
  expect(component.state.items).toEqual(['Item 1']);
  
  component.options.methods.addItem(component.state, 'Item 2');
  expect(component.state.items).toEqual(['Item 1', 'Item 2']);
  
  // Test clearItems
  component.options.methods.clearItems(component.state);
  expect(component.state.items).toEqual([]);
  
  component.destroy();
});
```

### Testing Async State

```js
import { waitFor } from '@testing-library/dom';

test('async state loads data', async () => {
  // Mock fetch
  global.fetch = vi.fn(() =>
    Promise.resolve({
      json: () => Promise.resolve([{ id: 1, name: 'Item 1' }])
    })
  );
  
  const component = $('#test').component({
    state: {},
    asyncState: {
      async items() {
        const res = await fetch('/api/items');
        return res.json();
      }
    }
  });
  
  // Initially loading
  expect(component.$async.items.loading).toBe(true);
  expect(component.$async.items.data).toBeNull();
  
  // Wait for data
  await waitFor(() => {
    expect(component.$async.items.loading).toBe(false);
  });
  
  expect(component.$async.items.data).toEqual([{ id: 1, name: 'Item 1' }]);
  expect(component.$async.items.error).toBeNull();
  
  component.destroy();
});
```

### Testing Events

```js
test('$emit triggers parent handlers', () => {
  document.body.innerHTML = `
    <div id="parent">
      <div id="child" $:component="child" $:on:custom="handleCustom($event)"></div>
    </div>
  `;
  
  const handler = vi.fn();
  
  const parent = $('#parent').component({
    state: {},
    methods: {
      handleCustom(state, payload) {
        handler(payload);
      }
    }
  });
  
  $.jqr.register('child', {
    template: '<button $:click="$emit(\'custom\', \'test-data\')">Click</button>'
  });
  
  const button = document.querySelector('#child button');
  button.click();
  
  expect(handler).toHaveBeenCalledWith('test-data');
  
  parent.destroy();
});
```

### Testing Props

```js
test('component receives and reacts to props', () => {
  document.body.innerHTML = `
    <div id="parent">
      <div $:component="child" $:prop:message="parentMessage"></div>
    </div>
  `;
  
  $.jqr.register('child', {
    props: { message: {} },
    template: '<p $:text="$props.message"></p>'
  });
  
  const parent = $('#parent').component({
    state: { parentMessage: 'Hello' }
  });
  
  const p = document.querySelector('#parent p');
  expect(p.textContent).toBe('Hello');
  
  // Update parent state
  parent.state.parentMessage = 'Updated';
  
  // Check child updated
  expect(p.textContent).toBe('Updated');
  
  parent.destroy();
});
```

### Testing Watchers

```js
test('watcher fires on state change', () => {
  const watchFn = vi.fn();
  
  const component = $('#test').component({
    state: { count: 0 },
    mounted() {
      this.watch('count', (newVal, oldVal) => {
        watchFn(newVal, oldVal);
      });
    }
  });
  
  component.state.count = 5;
  expect(watchFn).toHaveBeenCalledWith(5, 0);
  
  component.state.count = 10;
  expect(watchFn).toHaveBeenCalledWith(10, 5);
  
  component.destroy();
});
```

### Testing Lifecycle Hooks

```js
test('lifecycle hooks execute in order', () => {
  const order = [];
  
  const component = $('#test').component({
    state: {},
    created() {
      order.push('created');
    },
    mounted() {
      order.push('mounted');
    },
    destroyed() {
      order.push('destroyed');
    }
  });
  
  expect(order).toEqual(['created', 'mounted']);
  
  component.destroy();
  
  expect(order).toEqual(['created', 'mounted', 'destroyed']);
});
```

## Testing DOM Updates

### Testing Directives

```js
test('$:text updates on state change', () => {
  document.body.innerHTML = '<div id="test"><p $:text="message"></p></div>';
  
  const component = $('#test').component({
    state: { message: 'Initial' }
  });
  
  const p = document.querySelector('p');
  expect(p.textContent).toBe('Initial');
  
  component.state.message = 'Updated';
  expect(p.textContent).toBe('Updated');
  
  component.destroy();
});

test('$:if adds/removes elements', () => {
  document.body.innerHTML = '<div id="test"><p $:if="show">Visible</p></div>';
  
  const component = $('#test').component({
    state: { show: true }
  });
  
  expect(document.querySelector('p')).toBeTruthy();
  
  component.state.show = false;
  expect(document.querySelector('p')).toBeFalsy();
  
  component.state.show = true;
  expect(document.querySelector('p')).toBeTruthy();
  
  component.destroy();
});

test('$:for renders list', () => {
  document.body.innerHTML = `
    <div id="test">
      <ul>
        <li $:for="item in items" $:key="item" $:text="item"></li>
      </ul>
    </div>
  `;
  
  const component = $('#test').component({
    state: { items: ['A', 'B', 'C'] }
  });
  
  const lis = document.querySelectorAll('li');
  expect(lis.length).toBe(3);
  expect(lis[0].textContent).toBe('A');
  expect(lis[1].textContent).toBe('B');
  expect(lis[2].textContent).toBe('C');
  
  // Add item
  component.state.items.push('D');
  expect(document.querySelectorAll('li').length).toBe(4);
  
  component.destroy();
});
```

### Testing Form Bindings

```js
test('$:model creates two-way binding', () => {
  document.body.innerHTML = `
    <div id="test">
      <input $:model="username" />
      <p $:text="username"></p>
    </div>
  `;
  
  const component = $('#test').component({
    state: { username: 'John' }
  });
  
  const input = document.querySelector('input');
  const p = document.querySelector('p');
  
  // State → DOM
  expect(input.value).toBe('John');
  expect(p.textContent).toBe('John');
  
  // DOM → State
  input.value = 'Jane';
  input.dispatchEvent(new Event('input'));
  
  expect(component.state.username).toBe('Jane');
  expect(p.textContent).toBe('Jane');
  
  component.destroy();
});
```

## Mocking & Stubbing

### Mocking Fetch

```js
test('handles API errors', async () => {
  global.fetch = vi.fn(() =>
    Promise.reject(new Error('Network error'))
  );
  
  const component = $('#test').component({
    state: {},
    asyncState: {
      async data() {
        const res = await fetch('/api/data');
        return res.json();
      }
    }
  });
  
  await waitFor(() => {
    expect(component.$async.data.loading).toBe(false);
  });
  
  expect(component.$async.data.error).toBe('Network error');
  expect(component.$async.data.data).toBeNull();
  
  component.destroy();
});
```

### Mocking jQuery Plugins

```js
test('component uses jQuery plugin', () => {
  // Mock plugin
  $.fn.myPlugin = vi.fn();
  
  const component = $('#test').component({
    state: {},
    mounted() {
      this.$el.myPlugin({ option: 'value' });
    }
  });
  
  expect($.fn.myPlugin).toHaveBeenCalledWith({ option: 'value' });
  
  component.destroy();
});
```

### Stubbing Methods

```js
test('method calls another method', () => {
  const component = $('#test').component({
    state: {},
    methods: {
      save(state) {
        this.validate();
        // Save logic...
      },
      validate() {
        // Validation logic...
      }
    }
  });
  
  // Spy on validate
  const validateSpy = vi.spyOn(component.options.methods, 'validate');
  
  component.options.methods.save.call(component, component.state);
  
  expect(validateSpy).toHaveBeenCalled();
  
  component.destroy();
});
```

## Testing Utilities

### Test Helper Functions

```js
// test/helpers.js
export function createTestComponent(options) {
  const container = document.createElement('div');
  container.id = `test-${Date.now()}`;
  document.body.appendChild(container);
  
  const component = $(`#${container.id}`).component(options);
  
  return {
    component,
    container,
    cleanup() {
      component.destroy();
      container.remove();
    }
  };
}

export function waitForNextTick() {
  return new Promise(resolve => setTimeout(resolve, 0));
}

export function triggerEvent(element, eventType, data = {}) {
  const event = new Event(eventType, { bubbles: true });
  Object.assign(event, data);
  element.dispatchEvent(event);
}
```

**Usage:**

```js
import { createTestComponent, waitForNextTick } from './test/helpers';

test('using helper', async () => {
  const { component, cleanup } = createTestComponent({
    state: { count: 0 },
    methods: {
      async increment(state) {
        await waitForNextTick();
        state.count++;
      }
    }
  });
  
  await component.options.methods.increment.call(component, component.state);
  expect(component.state.count).toBe(1);
  
  cleanup();
});
```

### Custom Matchers

```js
// test/matchers.js
export const customMatchers = {
  toHaveState(received, expected) {
    const pass = Object.keys(expected).every(
      key => received.state[key] === expected[key]
    );
    
    return {
      pass,
      message: () => pass
        ? `Expected component not to have state ${JSON.stringify(expected)}`
        : `Expected component to have state ${JSON.stringify(expected)}, but got ${JSON.stringify(received.state)}`
    };
  },
  
  toHaveAsyncData(received, asyncKey, expected) {
    const actual = received.$async[asyncKey]?.data;
    const pass = JSON.stringify(actual) === JSON.stringify(expected);
    
    return {
      pass,
      message: () => pass
        ? `Expected async ${asyncKey} not to have data ${JSON.stringify(expected)}`
        : `Expected async ${asyncKey} to have data ${JSON.stringify(expected)}, but got ${JSON.stringify(actual)}`
    };
  }
};

// In setup.js
expect.extend(customMatchers);
```

**Usage:**

```js
test('custom matcher', () => {
  const component = $('#test').component({
    state: { count: 5, name: 'Test' }
  });
  
  expect(component).toHaveState({ count: 5, name: 'Test' });
  
  component.destroy();
});
```

## Best Practices

### 1. Isolate Tests

```js
// ✅ Good: Each test is independent
describe('Counter', () => {
  let component;
  
  beforeEach(() => {
    component = createComponent();
  });
  
  afterEach(() => {
    component.destroy();
  });
  
  test('test 1', () => { /* ... */ });
  test('test 2', () => { /* ... */ });
});

// ❌ Bad: Tests depend on each other
describe('Counter', () => {
  const component = createComponent();
  
  test('test 1', () => {
    component.state.count = 5;
  });
  
  test('test 2', () => {
    expect(component.state.count).toBe(5);  // Depends on test 1!
  });
});
```

### 2. Test Behavior, Not Implementation

```js
// ❌ Bad: Testing implementation details
test('uses specific computed property', () => {
  expect(component.computed.filteredItems).toBeDefined();
});

// ✅ Good: Testing behavior
test('displays filtered items', () => {
  component.state.filter = 'active';
  const items = document.querySelectorAll('.item');
  expect(items.length).toBe(3);
});
```

### 3. Use Meaningful Test Names

```js
// ❌ Bad: Vague test names
test('it works', () => { /* ... */ });
test('test 1', () => { /* ... */ });

// ✅ Good: Descriptive test names
test('increments counter when button clicked', () => { /* ... */ });
test('displays error message when API fails', () => { /* ... */ });
```

### 4. Clean Up After Tests

```js
// ✅ Good: Always cleanup
afterEach(() => {
  component.destroy();
  document.body.innerHTML = '';
});
```

### 5. Test Edge Cases

```js
test('handles empty list', () => {
  component.state.items = [];
  expect(document.querySelectorAll('.item').length).toBe(0);
});

test('handles null props', () => {
  component.setProps({ user: null });
  expect(document.querySelector('.user-name')).toBeFalsy();
});

test('handles very long text', () => {
  component.state.message = 'x'.repeat(1000);
  expect(document.querySelector('p').textContent.length).toBe(1000);
});
```

## Common Pitfalls

### 1. Not Waiting for Async

```js
// ❌ Bad: Doesn't wait
test('loads data', () => {
  component.fetchData();
  expect(component.state.data).toBeDefined();  // Still loading!
});

// ✅ Good: Waits for async
test('loads data', async () => {
  await component.fetchData();
  expect(component.state.data).toBeDefined();
});
```

### 2. Not Cleaning Up

```js
// ❌ Bad: Memory leak
test('test 1', () => {
  const comp = $('#test').component({});
  // Never destroyed!
});

// ✅ Good: Cleanup
test('test 1', () => {
  const comp = $('#test').component({});
  // ... test
  comp.destroy();
});
```

### 3. Testing Too Much

```js
// ❌ Bad: Testing framework internals
test('reactivity system works', () => {
  expect(component._reactiveProxy).toBeDefined();
  expect(component._effects.length).toBeGreaterThan(0);
});

// ✅ Good: Testing component behavior
test('UI updates when state changes', () => {
  component.state.count = 5;
  expect(document.querySelector('#count').textContent).toBe('5');
});
```

## Summary

**Key Testing Principles:**

1. **Unit test** component logic (state, computed, methods)
2. **Integration test** DOM interactions
3. **E2E test** full user flows
4. **Isolate** tests (beforeEach, afterEach)
5. **Clean up** after every test
6. **Test behavior**, not implementation
7. **Mock external** dependencies (fetch, jQuery plugins)
8. **Wait for async** operations

**What to Test:**
- State changes
- Computed properties
- Method execution
- DOM updates (directives)
- Event handling
- Props and events between components
- Async data loading
- Error handling
- Edge cases

**Test Structure:**
```js
describe('Component Name', () => {
  beforeEach(() => { /* setup */ });
  afterEach(() => { /* cleanup */ });
  
  test('specific behavior', () => {
    // Arrange: Set up
    // Act: Do something
    // Assert: Check result
  });
});
```

## Next Steps

- [Component Options](./component-options.md) - Full component API
- [Instance Methods](./instance-methods.md) - Using component methods
- [Performance Tips](./performance.md) - Optimization strategies
- [Computed vs Methods](./computed-vs-methods.md) - When to use which
