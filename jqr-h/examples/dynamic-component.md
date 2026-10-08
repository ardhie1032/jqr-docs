<!-- {
  "path": "jqr-h/examples/dynamic-component",
  "name": "docs-jqr-h-examples-dynamic-component",
  "meta": {
    "title": "Dynamic Components",
    "order": 2
  }
} -->

# Example: Dynamic Component Switching

Demonstrates dynamic component switching using component references, tabs interface, and conditional rendering patterns with JQR-H.

## Features

- ✅ Component switching (tab interface)
- ✅ Multiple component definitions
- ✅ Props passing to active component
- ✅ Event handling from child components
- ✅ Conditional component rendering
- ✅ Nested component composition

## Live Demo

```html live:no-code
<div id="demo-dynamic-components"></div>
```

```html
<div id="demo-dynamic-components"></div>
```

```js inject show
import { mount } from '@en32/jqr-h';

// Component Definitions
const HomeView = {
  render: (h, ctx) => h('div', { class: 'en-card' }, [
    h('div', { class: 'en-card__body' }, [
      h('h2', {}, ['Welcome to Home']),
      h('p', {}, ['This is the home view. Select different tabs to see other components.'])
    ])
  ])
};

const ProfileView = {
  props: { userName: 'Guest' },
  state: {
    bio: 'Web developer and open source enthusiast.',
    location: 'Internet'
  },
  render: (h, ctx) => h('div', { class: 'en-card' }, [
    h('div', { class: 'en-card__body' }, [
      h('h2', { text: (c) => `${c.$props.userName}'s Profile` }),
      h('dl', {}, [
        h('dt', { class: 'en-font-bold' }, ['Bio:']),
        h('dd', { text: (c) => c.bio }),
        h('dt', { class: 'en-font-bold en-mt-2' }, ['Location:']),
        h('dd', { text: (c) => c.location })
      ])
    ])
  ])
};

const SettingsView = {
  state: {
    notifications: true,
    darkMode: false,
    language: 'en'
  },
  render: (h, ctx) => h('div', { class: 'en-card' }, [
    h('div', { class: 'en-card__body' }, [
      h('h2', {}, ['Settings']),
      
      // Notifications toggle
      h('div', { class: 'en-mb-3' }, [
        h('label', { class: 'en-check' }, [
          h('input', {
            attrs: { type: 'checkbox' },
            model: 'notifications'
          }),
          h('span', {}, ['Enable Notifications'])
        ])
      ]),
      
      // Dark mode toggle
      h('div', { class: 'en-mb-3' }, [
        h('label', { class: 'en-check' }, [
          h('input', {
            attrs: { type: 'checkbox' },
            model: 'darkMode'
          }),
          h('span', {}, ['Dark Mode'])
        ])
      ]),
      
      // Language select
      h('div', {}, [
        h('label', { class: 'en-form-label' }, ['Language:']),
        h('select', {
          class: 'en-select',
          model: 'language'
        }, [
          h('option', { attrs: { value: 'en' } }, ['English']),
          h('option', { attrs: { value: 'id' } }, ['Indonesian']),
          h('option', { attrs: { value: 'es' } }, ['Spanish'])
        ])
      ]),
      
      // Settings summary
      h('div', { class: 'en-alert en-alert--info en-mt-4' }, [
        h('strong', {}, ['Current Settings:']),
        h('ul', {}, [
          h('li', { text: (c) => `Notifications: ${c.notifications ? 'On' : 'Off'}` }),
          h('li', { text: (c) => `Dark Mode: ${c.darkMode ? 'Enabled' : 'Disabled'}` }),
          h('li', { text: (c) => `Language: ${c.language.toUpperCase()}` })
        ])
      ])
    ])
  ])
};

// Main App Component
mount('#demo-dynamic-components', {
  state: {
    activeTab: 'home',
    userName: 'Alice'
  },
  computed: {
    activeComponent() {
      switch (this.activeTab) {
        case 'home': return HomeView;
        case 'profile': return ProfileView;
        case 'settings': return SettingsView;
        default: return HomeView;
      }
    }
  },
  methods: {
    switchTab(tab) {
      this.activeTab = tab;
    }
  },
  render: (h, ctx) => h('div', { class: 'en-card' }, [
    // Tabs Header
    h('div', { class: 'en-card__header' }, [
      h('div', { class: 'en-flex en-gap-2' }, [
        h('button', {
          class: (c) => ['en-btn', 'en-btn--sm', {
            'en-btn--primary': c.activeTab === 'home'
          }],
          on: {
            click: (c) => c.switchTab('home')
          }
        }, ['Home']),
        
        h('button', {
          class: (c) => ['en-btn', 'en-btn--sm', {
            'en-btn--primary': c.activeTab === 'profile'
          }],
          on: {
            click: (c) => c.switchTab('profile')
          }
        }, ['Profile']),
        
        h('button', {
          class: (c) => ['en-btn', 'en-btn--sm', {
            'en-btn--primary': c.activeTab === 'settings'
          }],
          on: {
            click: (c) => c.switchTab('settings')
          }
        }, ['Settings'])
      ])
    ]),
    
    // Dynamic Component Content
    h('div', { class: 'en-card__body' }, [
      // Render active component
      h((c) => c.activeComponent, {
        props: {
          userName: (c) => c.userName
        }
      })
    ])
  ])
});
```

```js cleanup
onUnmount(() => {
  JQRH.unmount('#demo-dynamic-components');
})
```

## Code Breakdown

### Component Definitions

Each view is a separate component definition:

```js
const HomeView = {
  render: (h, ctx) => h('div', {}, [
    h('h2', {}, ['Home'])
  ])
};

const ProfileView = {
  props: { userName: 'Guest' },
  render: (h, ctx) => h('div', {}, [
    h('h2', { text: (c) => c.$props.userName })
  ])
};
```

### Dynamic Component Selection

Use computed property to select active component:

```js
computed: {
  activeComponent() {
    switch (this.activeTab) {
      case 'home': return HomeView;
      case 'profile': return ProfileView;
      case 'settings': return SettingsView;
      default: return HomeView;
    }
  }
}
```

### Rendering Dynamic Component

Pass component reference directly to `h()`:

```js
h((c) => c.activeComponent, {
  props: {
    userName: (c) => c.userName
  }
})
```

**Key point:** Component reference must be wrapped in function for reactivity.

### Props Passing

Props are passed as functions (reactive):

```js
props: {
  userName: (c) => c.userName
}
```

When `userName` changes in parent, child receives updated prop.

## Alternative Patterns

### Pattern 1: Direct Component Map

Store components in an object:

```js
{
  state: {
    activeTab: 'home',
    components: {
      home: HomeView,
      profile: ProfileView,
      settings: SettingsView
    }
  },
  render: (h, ctx) => h('div', {}, [
    h((c) => c.components[c.activeTab])
  ])
}
```

### Pattern 2: Conditional Rendering

Use separate `h()` calls with `if`:

```js
render: (h, ctx) => h('div', {}, [
  h(HomeView, { if: (c) => c.activeTab === 'home' }),
  h(ProfileView, { if: (c) => c.activeTab === 'profile' }),
  h(SettingsView, { if: (c) => c.activeTab === 'settings' })
])
```

### Pattern 3: Registered Components

Use string component names:

```js
// Register components
$.jqr.register('home-view', HomeView);
$.jqr.register('profile-view', ProfileView);
$.jqr.register('settings-view', SettingsView);

// Render with computed name
{
  computed: {
    activeComponentName() {
      return `${this.activeTab}-view`;
    }
  },
  render: (h, ctx) => h('div', {}, [
    h((c) => c.activeComponentName)
  ])
}
```

## Advanced: Transition Effects

Add smooth transitions when switching:

```js
render: (h, ctx) => h('div', {}, [
  h('div', {
    style: {
      transition: 'opacity 0.3s ease',
      opacity: '1'
    }
  }, [
    h((c) => c.activeComponent, {
      props: {
        userName: (c) => c.userName
      }
    })
  ])
])
```

## Advanced: Keep Component State

By default, switching destroys the old component. To preserve state, use conditional rendering:

```js
render: (h, ctx) => h('div', {}, [
  h('div', {
    show: (c) => c.activeTab === 'home'
  }, [
    h(HomeView)
  ]),
  h('div', {
    show: (c) => c.activeTab === 'profile'
  }, [
    h(ProfileView)
  ])
])
```

**Difference:**
- `if`: Unmounts/remounts (state lost)
- `show`: Hides/shows (state preserved)

## Advanced: Component Communication

### Child to Parent Events

```js
const ChildComponent = {
  methods: {
    handleAction() {
      this.$emit('action', { data: 'value' });
    }
  },
  render: (h, ctx) => h('button', {
    on: {
      click: (c) => c.handleAction()
    }
  }, ['Trigger Action'])
};

// Parent
h(ChildComponent, {
  emit: {
    action: (ctx, payload) => {
      console.log('Child emitted:', payload);
    }
  }
})
```

### Props Down, Events Up

Classic pattern:

```js
// Parent
{
  state: { count: 0 },
  render: (h, ctx) => h('div', {}, [
    h(CountDisplay, {
      props: {
        count: (c) => c.count
      },
      emit: {
        increment: (c) => c.count++,
        decrement: (c) => c.count--
      }
    })
  ])
}

// Child
const CountDisplay = {
  props: { count: 0 },
  render: (h, ctx) => h('div', {}, [
    h('p', { text: (c) => c.$props.count }),
    h('button', {
      on: { click: (c) => c.$emit('increment') }
    }, ['+']),
    h('button', {
      on: { click: (c) => c.$emit('decrement') }
    }, ['-'])
  ])
};
```

## Real-World Example: Router-Like Navigation

```js
mount('#app', {
  state: {
    currentRoute: 'home',
    routes: {
      home: HomeView,
      profile: ProfileView,
      settings: SettingsView,
      about: AboutView
    }
  },
  created() {
    // Listen to hash changes
    window.addEventListener('hashchange', () => {
      this.currentRoute = location.hash.slice(1) || 'home';
    });
    
    // Set initial route
    this.currentRoute = location.hash.slice(1) || 'home';
  },
  methods: {
    navigate(route) {
      location.hash = route;
    }
  },
  computed: {
    activeComponent() {
      return this.routes[this.currentRoute] || this.routes.home;
    }
  },
  render: (h, ctx) => h('div', {}, [
    // Navigation
    h('nav', {}, [
      h('button', {
        on: { click: (c) => c.navigate('home') }
      }, ['Home']),
      h('button', {
        on: { click: (c) => c.navigate('profile') }
      }, ['Profile']),
      h('button', {
        on: { click: (c) => c.navigate('settings') }
      }, ['Settings'])
    ]),
    
    // Active view
    h((c) => c.activeComponent)
  ])
});
```

## Key Takeaways

1. **Component references must be reactive** - Wrap in function: `(c) => c.activeComponent`
2. **Props are reactive functions** - `props: { x: (c) => c.x }`
3. **Each switch creates new instance** - Use `show` instead of `if` to preserve state
4. **Events flow up** - Use `emit` attribute to listen to child events
5. **TypeScript support** - Component references are type-checked

## See Also

- [Components with h()](../guides/components-with-h.md) - Component composition
- [Basic Component](./basic-component.md) - Simple component example
- [List Rendering](./list-rendering.md) - Dynamic lists
- [Nested Components](../../jqr/guides/nested-components.md) - Template-based nesting
