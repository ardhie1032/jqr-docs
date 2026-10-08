<!-- {
  "path": "jqr-h/guides/architecture",
  "name": "docs-jqr-jqrh-guides-architecture",
  "meta": {
    "title": "Architecture",
    "order": 4
  }
} -->

# JQR-H Architecture

JQR-H (`@en32/jqr-h`) implements a **direct DOM manipulation** approach with **fine-grained reactivity**, fundamentally different from Virtual DOM frameworks like React or Vue.

## Core Concepts

### 1. Direct DOM Creation

Unlike Virtual DOM frameworks that create intermediate representations and diff them, JQR-H creates **real DOM elements immediately**:

```js
// h() builds a description
const node = h('div', { text: (s) => s.message });

// renderNode() creates ACTUAL HTMLElement
const el = renderNode(node, instance, disposers);
// → el is a real <div> in memory, ready to append
```

**No VDOM diffing.** Every `h()` call describes what to create, and `renderNode()` creates it once.

### 2. Fine-Grained Reactivity

Each reactive binding becomes its own `effect()` that updates **only the specific DOM property it controls**:

```js
h('div', {
  text: (s) => s.message,      // effect #1 - updates el.textContent
  class: (s) => s.theme,       // effect #2 - updates el.className
  style: (s) => ({ color: s.color })  // effect #3 - updates el.style.color
})
```

When `state.message` changes, **only effect #1 runs** — the DOM node isn't recreated, and `class`/`style` aren't touched.

### 3. No Re-rendering

Template-based frameworks re-render entire component trees. JQR-H **never re-renders**:

- DOM nodes are created once
- Effects update properties in-place
- No component function re-execution
- No reconciliation/patching

## How It Works

### Building Phase

`h()` produces a **description object** (`JqrNode`) — not DOM, not JSX:

```js
const node = h('button', 
  { 
    text: (s) => s.label,
    on: { click: (s) => s.count++ }
  }
);

console.log(node);
// {
//   [JQR_NODE]: true,
//   tag: 'button',
//   attrs: { text: [Function], on: { click: [Function] } },
//   children: [],
//   slots: { default: [] }
// }
```

**No side effects.** Building descriptions is cheap — you can build and discard them without touching the DOM.

### Rendering Phase

`renderNode()` walks the description and creates real elements:

```js
export function renderNode(node, instance, disposers) {
  // 1. Create element
  const el = document.createElement(node.tag);
  
  // 2. Apply attributes - each becomes an effect
  applyAttrs(el, node.attrs, instance, disposers);
  
  // 3. Render children recursively
  for (const child of node.children) {
    appendChild(el, child, instance, disposers);
  }
  
  return el;  // Real HTMLElement, ready to use
}
```

### Reactivity Wiring

Each attribute is wrapped in an `effect()`:

```js
// Text attribute
if (attrs.text) {
  const { stop } = effect(() => {
    el.textContent = stringifyValue(attrs.text(state));
  });
  disposers.push(stop);  // Cleanup when component destroyed
}

// Class attribute
if (attrs.class) {
  const { stop } = effect(() => {
    const val = typeof attrs.class === 'function' 
      ? attrs.class(state) 
      : attrs.class;
    const cls = normalizeClass(val);
    if (cls) el.setAttribute('class', cls);
    else el.removeAttribute('class');
  });
  disposers.push(stop);
}
```

**Key insight:** Each effect tracks **only the state properties it reads**. Changing unrelated state doesn't trigger unrelated effects.

## Memory Management

### Disposer Pattern

Every effect and child component registers a cleanup function:

```js
const disposers = [];

// Render creates effects and nested components
const el = renderNode(node, instance, disposers);

// When done (parent destroyed, or h.for item removed):
disposers.forEach(fn => fn());
// → Every effect stopped, every child destroyed, no leaks
```

### List Rendering

`h.for()` implements **keyed diffing** without Virtual DOM:

```js
h.for(
  (s) => s.items,           // Items array
  (item) => item.id,        // Key function
  (item) => h('li', { text: () => item.name })
)
```

**How it works:**

1. Track `rendered = Map<key, {el, dispose}>`
2. On update: compute new keys, reuse existing entries, dispose removed ones
3. Only **new keys** trigger `renderNode()`, existing keys keep their DOM + effects

```js
// Simplified
effect(() => {
  const items = child.items(state);
  const usedKeys = new Set();
  
  items.forEach((item, index) => {
    const key = String(child.key(item, index));
    usedKeys.add(key);
    
    let entry = rendered.get(key);
    if (!entry) {
      // New item - render it
      const itemDisposers = [];
      const el = renderNode(child.build(item, index), instance, itemDisposers);
      entry = { el, dispose: () => itemDisposers.forEach(fn => fn()) };
      rendered.set(key, entry);
    }
    
    // Move to correct position
    parent.insertBefore(entry.el, cursor.nextSibling);
    cursor = entry.el;
  });
  
  // Dispose removed items
  for (const [key, entry] of rendered) {
    if (!usedKeys.has(key)) {
      entry.dispose();  // Stop all effects
      entry.el.remove();  // Remove from DOM
      rendered.delete(key);
    }
  }
});
```

**Result:** Items stay alive across updates. No recreation, no effect leak.

## Performance Characteristics

### Strengths

**1. Surgical Updates**

Only the changed property updates:

```js
state.username = 'Alice';  // Only el.textContent updates
state.theme = 'dark';      // Only el.className updates
```

No component re-execution, no reconciliation overhead.

**2. No Diffing Cost**

Virtual DOM frameworks:
- Build new VDOM tree
- Diff old vs new tree
- Patch DOM

JQR-H:
- Effect runs
- DOM property updates
- Done

**3. Predictable Memory**

Every effect/component is tracked. Cleanup is explicit and complete.

### Trade-offs

**1. Initial Render Cost**

Creating real DOM elements is more expensive than creating VDOM objects. For **very large** initial renders (1000+ elements), VDOM can be faster.

**Mitigation:** Use `h.for()` with pagination/virtualization.

**2. No Time Slicing**

Effects run synchronously. A single state change that triggers 1000 effects will block the main thread.

**Mitigation:** Batch updates, use `debounce`/`throttle` for high-frequency changes.

**3. Manual Optimization**

Virtual DOM frameworks can optimize across components. JQR-H effects are independent — you control granularity.

**Best practice:** One effect per concern. Don't combine unrelated state reads in a single effect.

## Comparison with Other Approaches

### Virtual DOM (React, Vue)

**Approach:**
- Component function → VDOM tree
- Diff VDOM → patch list
- Apply patches → DOM

**JQR-H:**
- `render()` → description
- Description → real DOM (once)
- State change → effect → DOM property

**Trade-off:**
- VDOM: Better for frequent full-component updates
- JQR-H: Better for frequent targeted updates

### Template Compilation (Svelte, Solid)

**Svelte:**
- Compiles templates to imperative DOM updates at build time
- No runtime framework (smaller bundle)

**Solid:**
- Fine-grained reactivity like JQR-H
- Compiled templates for optimal code

**JQR-H:**
- Runtime-based (no build step required)
- Fine-grained reactivity via `effect()`
- Manual `h()` calls (more verbose, more control)

**Trade-off:**
- Svelte/Solid: Better DX via templates + compiler
- JQR-H: More explicit, no build requirement, full JS flexibility

### jQuery Traditional

**jQuery:**
- Imperative DOM manipulation
- Manual state tracking
- No reactivity

**JQR-H:**
- Declarative descriptions
- Automatic reactivity
- Memory-safe cleanup

**Trade-off:**
- jQuery: Maximum control, maximum boilerplate
- JQR-H: Declarative DX with reactive automation

## Architectural Guarantees

### 1. One Element, One Creation

```js
const node = h('div', { text: (s) => s.msg });
const el1 = renderNode(node, instance, []);
const el2 = renderNode(node, instance, []);

console.log(el1 === el2);  // false - two separate elements
```

Each `renderNode()` call creates a new element. Descriptions are **templates**, not singletons.

### 2. Effect Isolation

```js
h('div', {
  text: (s) => s.a + s.b,    // Tracks s.a and s.b
  class: (s) => s.theme      // Tracks ONLY s.theme
});

state.theme = 'dark';  // Only class effect runs
state.a = 1;           // Only text effect runs
```

Effects track **only the state properties they read** (via Proxy). No over-triggering.

### 3. Deterministic Cleanup

```js
const disposers = [];
const el = renderNode(node, instance, disposers);

// Later:
disposers.forEach(fn => fn());
// → ALL effects stopped
// → ALL nested components destroyed
// → NO memory leaks
```

Every reactive subscription has an explicit cleanup path. No reliance on GC to break cycles.

### 4. Nested Component Isolation

Child components are **real `Component` instances**, same as template-based components:

```js
const child = h(UserCard, { prop: { name: (s) => s.user } });
const el = renderNode(child, parent, disposers);

// Child has its own:
// - state
// - lifecycle (created, mounted, destroyed)
// - effects
// - cleanup
```

Parent destruction tears down children via `disposers.push(() => child.destroy())`.

## Implementation Notes

### Why `effect()` and Not Signals?

JQR-H uses JQR's `effect()` (coarse-grained, runs entire function) instead of fine-grained signals:

**Reason:** Compatibility with existing JQR components. `state` is a single reactive proxy, not individual signals.

**Trade-off:**
- Signals: Track individual properties, update only consumers of that property
- `effect()`: Track all reads in closure, re-run entire closure

**Impact:** Minimal for typical UI code (effects are small, targeted).

### Why Description Objects, Not JSX?

JSX requires a build step (Babel/TypeScript). JQR-H runs in plain ES6:

```js
// JSX (requires build)
<div className={theme} onClick={() => count++}>
  {message}
</div>

// h() (plain JS)
h('div', {
  class: (s) => s.theme,
  on: { click: (s) => s.count++ },
  text: (s) => s.message
})
```

**Trade-off:**
- JSX: More familiar syntax
- `h()`: No build step, more explicit control

## Best Practices

### 1. Keep Effects Small

```js
// ❌ Bad - large effect tracks many things
h('div', {
  bind: (s) => ({
    text: s.user.name,
    class: { active: s.user.active },
    'attr:title': s.user.tooltip,
    style: { color: s.theme.primaryColor }
  })
});

// ✅ Good - each concern is its own effect
h('div', {
  text: (s) => s.user.name,
  class: (s) => ({ active: s.user.active }),
  attr: { title: (s) => s.user.tooltip },
  style: (s) => ({ color: s.theme.primaryColor })
});
```

Smaller effects = more precise tracking = fewer unnecessary updates.

### 2. Hoist Static Content

```js
// ❌ Bad - rebuilds array on every render() call
render(h) {
  return h('ul', {}, [
    h('li', {}, ['Static Item 1']),
    h('li', {}, ['Static Item 2']),
    h('li', { text: (s) => s.dynamic })
  ]);
}

// ✅ Good - static nodes built once
const STATIC_ITEMS = [
  h('li', {}, ['Static Item 1']),
  h('li', {}, ['Static Item 2'])
];

render(h, ctx) {
  return h('ul', {}, [
    ...STATIC_ITEMS,
    h('li', { text: (s) => s.dynamic })
  ]);
}
```

Descriptions are cheap, but no need to rebuild constants.

### 3. Use Keys in Lists

```js
// ❌ Bad - no key, recreates everything
h.for(
  (s) => s.items,
  (item, i) => i,  // Index as key - unstable!
  (item) => h('div', { text: () => item.name })
)

// ✅ Good - stable key, reuses elements
h.for(
  (s) => s.items,
  (item) => item.id,  // Stable, unique key
  (item) => h('div', { text: () => item.name })
)
```

Stable keys preserve DOM nodes + effects across reorders/updates.

## Summary

**JQR-H Architecture:**
- **Direct DOM:** Real elements, no intermediate representation
- **Fine-grained:** Each binding is its own effect
- **No re-rendering:** Elements created once, updated in-place
- **Explicit cleanup:** Disposers guarantee no leaks
- **Keyed diffing:** `h.for()` preserves elements by key

**When to use:**
- Frequent targeted updates (one property changing often)
- Memory-constrained environments (no VDOM overhead)
- No build step requirement (runtime-only apps)

**When to reconsider:**
- Very large initial renders (1000+ elements)
- Frequent full-component updates (VDOM batching may help)

## Next Steps

- [Using Directives in h()](./directives-in-h.md) - Custom directives with `h()`
- [Slots with Render Functions](./slots-in-h.md) - Named slots in components
- [h() Function Reference](./h-function.md) - Complete `h()` API
- [Performance Guide](../../jqr/guides/performance.md) - Optimization tips
