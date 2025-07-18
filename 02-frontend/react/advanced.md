
# 🚀 Advanced React

## Table of Contents
- [React Architecture Patterns](#react-architecture-patterns)
- [Performance Optimization](#performance-optimization)
- [Advanced Hooks](#advanced-hooks)
- [Context and State Management](#context-and-state-management)
- [Error Boundaries and Suspense](#error-boundaries-and-suspense)
- [Testing Strategies](#testing-strategies)
- [Server-Side Rendering](#server-side-rendering)
- [Micro-frontends](#micro-frontends)

---

## 🏗 React Architecture Patterns

### Higher-Order Components (HOCs)

```jsx
import React from 'react';

// HOC for authentication
const withAuth = (WrappedComponent) => {
  return function AuthenticatedComponent(props) {
    const [user, setUser] = useState(null);
    const [loading, setLoading] = useState(true);

    useEffect(() => {
      const checkAuth = async () => {
        try {
          const userData = await getCurrentUser();
          setUser(userData);
        } catch (error) {
          console.error('Authentication failed:', error);
        } finally {
          setLoading(false);
        }
      };

      checkAuth();
    }, []);

    if (loading) {
      return <div>Loading...</div>;
    }

    if (!user) {
      return <div>Please log in to access this content.</div>;
    }

    return <WrappedComponent {...props} user={user} />;
  };
};

// Usage
const Dashboard = ({ user }) => (
  <div>
    <h1>Welcome, {user.name}!</h1>
    {/* Dashboard content */}
  </div>
);

const AuthenticatedDashboard = withAuth(Dashboard);
```

### Render Props Pattern

```jsx
// Data fetcher with render props
class DataFetcher extends Component {
  state = {
    data: null,
    loading: true,
    error: null
  };

  async componentDidMount() {
    try {
      const response = await fetch(this.props.url);
      const data = await response.json();
      this.setState({ data, loading: false });
    } catch (error) {
      this.setState({ error, loading: false });
    }
  }

  render() {
    return this.props.children(this.state);
  }
}

// Usage
function UserProfile({ userId }) {
  return (
    <DataFetcher url={`/api/users/${userId}`}>
      {({ data, loading, error }) => {
        if (loading) return <div>Loading...</div>;
        if (error) return <div>Error: {error.message}</div>;
        if (!data) return <div>No user found</div>;

        return (
          <div>
            <h1>{data.name}</h1>
            <p>{data.email}</p>
          </div>
        );
      }}
    </DataFetcher>
  );
}
```

### Compound Components

```jsx
// Accordion compound component
const AccordionContext = createContext();

function Accordion({ children, ...props }) {
  const [openIndex, setOpenIndex] = useState(null);

  const toggle = (index) => {
    setOpenIndex(openIndex === index ? null : index);
  };

  return (
    <AccordionContext.Provider value={{ openIndex, toggle }}>
      <div className="accordion" {...props}>
        {React.Children.map(children, (child, index) =>
          React.cloneElement(child, { index })
        )}
      </div>
    </AccordionContext.Provider>
  );
}

function AccordionItem({ children, index }) {
  const { openIndex, toggle } = useContext(AccordionContext);
  const isOpen = openIndex === index;

  return (
    <div className={`accordion-item ${isOpen ? 'open' : ''}`}>
      {React.Children.map(children, (child) =>
        React.cloneElement(child, { isOpen, toggle: () => toggle(index) })
      )}
    </div>
  );
}

function AccordionHeader({ children, toggle }) {
  return (
    <button className="accordion-header" onClick={toggle}>
      {children}
    </button>
  );
}

function AccordionPanel({ children, isOpen }) {
  return isOpen ? <div className="accordion-panel">{children}</div> : null;
}

// Usage
function App() {
  return (
    <Accordion>
      <AccordionItem>
        <AccordionHeader>Section 1</AccordionHeader>
        <AccordionPanel>Content for section 1</AccordionPanel>
      </AccordionItem>
      <AccordionItem>
        <AccordionHeader>Section 2</AccordionHeader>
        <AccordionPanel>Content for section 2</AccordionPanel>
      </AccordionItem>
    </Accordion>
  );
}
```

---

## ⚡ Performance Optimization

### React.memo and useMemo

```jsx
import React, { memo, useMemo, useCallback } from 'react';

// Memoized component
const ExpensiveComponent = memo(function ExpensiveComponent({ 
  data, 
  onItemClick,
  sortBy 
}) {
  // Expensive calculation
  const processedData = useMemo(() => {
    console.log('Processing data...');
    return data
      .filter(item => item.active)
      .sort((a, b) => {
        switch (sortBy) {
          case 'name':
            return a.name.localeCompare(b.name);
          case 'date':
            return new Date(b.date) - new Date(a.date);
          default:
            return 0;
        }
      })
      .map(item => ({
        ...item,
        displayName: `${item.name} (${item.category})`
      }));
  }, [data, sortBy]);

  return (
    <div>
      {processedData.map(item => (
        <div key={item.id} onClick={() => onItemClick(item.id)}>
          {item.displayName}
        </div>
      ))}
    </div>
  );
});

// Parent component with optimized callbacks
function DataList({ items }) {
  const [sortBy, setSortBy] = useState('name');
  const [selectedItems, setSelectedItems] = useState(new Set());

  // Memoized callback to prevent unnecessary re-renders
  const handleItemClick = useCallback((itemId) => {
    setSelectedItems(prev => {
      const newSet = new Set(prev);
      if (newSet.has(itemId)) {
        newSet.delete(itemId);
      } else {
        newSet.add(itemId);
      }
      return newSet;
    });
  }, []);

  // Memoized filtered data
  const filteredItems = useMemo(() => {
    return items.filter(item => !item.archived);
  }, [items]);

  return (
    <div>
      <select value={sortBy} onChange={(e) => setSortBy(e.target.value)}>
        <option value="name">Sort by Name</option>
        <option value="date">Sort by Date</option>
      </select>
      
      <ExpensiveComponent 
        data={filteredItems}
        onItemClick={handleItemClick}
        sortBy={sortBy}
      />
      
      <div>Selected: {selectedItems.size} items</div>
    </div>
  );
}
```

### Virtual Scrolling

```jsx
import { useState, useEffect, useRef, useMemo } from 'react';

function VirtualScrollList({ 
  items, 
  itemHeight, 
  containerHeight,
  renderItem 
}) {
  const [scrollTop, setScrollTop] = useState(0);
  const scrollElementRef = useRef();

  // Calculate visible range
  const visibleRange = useMemo(() => {
    const start = Math.floor(scrollTop / itemHeight);
    const visibleCount = Math.ceil(containerHeight / itemHeight);
    const end = Math.min(start + visibleCount + 1, items.length);
    
    return { start, end };
  }, [scrollTop, itemHeight, containerHeight, items.length]);

  // Calculate total height and visible items
  const totalHeight = items.length * itemHeight;
  const visibleItems = items.slice(visibleRange.start, visibleRange.end);

  const handleScroll = (e) => {
    setScrollTop(e.target.scrollTop);
  };

  return (
    <div
      ref={scrollElementRef}
      style={{
        height: containerHeight,
        overflow: 'auto'
      }}
      onScroll={handleScroll}
    >
      <div style={{ height: totalHeight, position: 'relative' }}>
        <div
          style={{
            transform: `translateY(${visibleRange.start * itemHeight}px)`
          }}
        >
          {visibleItems.map((item, index) => (
            <div
              key={visibleRange.start + index}
              style={{ height: itemHeight }}
            >
              {renderItem(item, visibleRange.start + index)}
            </div>
          ))}
        </div>
      </div>
    </div>
  );
}

// Usage
function App() {
  const items = Array.from({ length: 10000 }, (_, i) => ({
    id: i,
    name: `Item ${i}`,
    value: Math.random()
  }));

  return (
    <VirtualScrollList
      items={items}
      itemHeight={50}
      containerHeight={400}
      renderItem={(item) => (
        <div style={{ padding: '10px', borderBottom: '1px solid #ccc' }}>
          {item.name} - {item.value.toFixed(2)}
        </div>
      )}
    />
  );
}
```

### Code Splitting and Lazy Loading

```jsx
import { lazy, Suspense } from 'react';

// Lazy load components
const Dashboard = lazy(() => import('./components/Dashboard'));
const Profile = lazy(() => import('./components/Profile'));
const Settings = lazy(() => 
  import('./components/Settings').then(module => ({
    default: module.Settings
  }))
);

// Loading component
function LoadingSpinner() {
  return (
    <div className="loading-spinner">
      <div className="spinner"></div>
      <p>Loading...</p>
    </div>
  );
}

// Error boundary for lazy components
class LazyComponentErrorBoundary extends Component {
  constructor(props) {
    super(props);
    this.state = { hasError: false };
  }

  static getDerivedStateFromError(error) {
    return { hasError: true };
  }

  componentDidCatch(error, errorInfo) {
    console.error('Lazy component error:', error, errorInfo);
  }

  render() {
    if (this.state.hasError) {
      return (
        <div>
          <h2>Something went wrong loading this component.</h2>
          <button onClick={() => this.setState({ hasError: false })}>
            Try Again
          </button>
        </div>
      );
    }

    return this.props.children;
  }
}

// Router with lazy loading
function App() {
  return (
    <Router>
      <div className="app">
        <nav>
          <Link to="/dashboard">Dashboard</Link>
          <Link to="/profile">Profile</Link>
          <Link to="/settings">Settings</Link>
        </nav>

        <main>
          <LazyComponentErrorBoundary>
            <Suspense fallback={<LoadingSpinner />}>
              <Routes>
                <Route path="/dashboard" element={<Dashboard />} />
                <Route path="/profile" element={<Profile />} />
                <Route path="/settings" element={<Settings />} />
              </Routes>
            </Suspense>
          </LazyComponentErrorBoundary>
        </main>
      </div>
    </Router>
  );
}
```

---

## 🪝 Advanced Hooks

### Custom Hooks

```jsx
// useLocalStorage hook
function useLocalStorage(key, initialValue) {
  const [storedValue, setStoredValue] = useState(() => {
    try {
      const item = window.localStorage.getItem(key);
      return item ? JSON.parse(item) : initialValue;
    } catch (error) {
      console.error(`Error reading localStorage key "${key}":`, error);
      return initialValue;
    }
  });

  const setValue = useCallback((value) => {
    try {
      const valueToStore = value instanceof Function ? value(storedValue) : value;
      setStoredValue(valueToStore);
      window.localStorage.setItem(key, JSON.stringify(valueToStore));
    } catch (error) {
      console.error(`Error setting localStorage key "${key}":`, error);
    }
  }, [key, storedValue]);

  return [storedValue, setValue];
}

// useDebounce hook
function useDebounce(value, delay) {
  const [debouncedValue, setDebouncedValue] = useState(value);

  useEffect(() => {
    const handler = setTimeout(() => {
      setDebouncedValue(value);
    }, delay);

    return () => {
      clearTimeout(handler);
    };
  }, [value, delay]);

  return debouncedValue;
}

// useAsync hook
function useAsync(asyncFunction, dependencies = []) {
  const [state, setState] = useState({
    data: null,
    loading: true,
    error: null
  });

  useEffect(() => {
    let cancelled = false;
    
    setState({ data: null, loading: true, error: null });
    
    asyncFunction()
      .then(data => {
        if (!cancelled) {
          setState({ data, loading: false, error: null });
        }
      })
      .catch(error => {
        if (!cancelled) {
          setState({ data: null, loading: false, error });
        }
      });

    return () => {
      cancelled = true;
    };
  }, dependencies);

  return state;
}

// useIntersectionObserver hook
function useIntersectionObserver(elementRef, options = {}) {
  const [isIntersecting, setIsIntersecting] = useState(false);

  useEffect(() => {
    const element = elementRef.current;
    if (!element) return;

    const observer = new IntersectionObserver(
      ([entry]) => {
        setIsIntersecting(entry.isIntersecting);
      },
      options
    );

    observer.observe(element);

    return () => {
      observer.unobserve(element);
    };
  }, [elementRef, options]);

  return isIntersecting;
}

// Usage examples
function SearchComponent() {
  const [query, setQuery] = useState('');
  const [savedSearches, setSavedSearches] = useLocalStorage('savedSearches', []);
  const debouncedQuery = useDebounce(query, 300);

  const searchResults = useAsync(async () => {
    if (!debouncedQuery) return [];
    const response = await fetch(`/api/search?q=${debouncedQuery}`);
    return response.json();
  }, [debouncedQuery]);

  const elementRef = useRef();
  const isVisible = useIntersectionObserver(elementRef, { threshold: 0.1 });

  return (
    <div ref={elementRef}>
      <input
        value={query}
        onChange={(e) => setQuery(e.target.value)}
        placeholder="Search..."
      />
      
      {searchResults.loading && <div>Searching...</div>}
      {searchResults.error && <div>Error: {searchResults.error.message}</div>}
      {searchResults.data && (
        <div>
          {searchResults.data.map(result => (
            <div key={result.id}>{result.title}</div>
          ))}
        </div>
      )}
      
      <div>Component is {isVisible ? 'visible' : 'hidden'}</div>
    </div>
  );
}
```

### useReducer for Complex State

```jsx
// Complex state management with useReducer
const initialState = {
  items: [],
  selectedItems: new Set(),
  filter: 'all',
  sortBy: 'name',
  loading: false,
  error: null
};

function itemsReducer(state, action) {
  switch (action.type) {
    case 'SET_LOADING':
      return { ...state, loading: action.payload };
    
    case 'SET_ERROR':
      return { ...state, error: action.payload, loading: false };
    
    case 'SET_ITEMS':
      return { 
        ...state, 
        items: action.payload, 
        loading: false, 
        error: null 
      };
    
    case 'ADD_ITEM':
      return {
        ...state,
        items: [...state.items, action.payload]
      };
    
    case 'REMOVE_ITEM':
      return {
        ...state,
        items: state.items.filter(item => item.id !== action.payload),
        selectedItems: new Set([...state.selectedItems].filter(id => id !== action.payload))
      };
    
    case 'TOGGLE_ITEM_SELECTION':
      const newSelectedItems = new Set(state.selectedItems);
      if (newSelectedItems.has(action.payload)) {
        newSelectedItems.delete(action.payload);
      } else {
        newSelectedItems.add(action.payload);
      }
      return { ...state, selectedItems: newSelectedItems };
    
    case 'SELECT_ALL':
      return {
        ...state,
        selectedItems: new Set(state.items.map(item => item.id))
      };
    
    case 'CLEAR_SELECTION':
      return { ...state, selectedItems: new Set() };
    
    case 'SET_FILTER':
      return { ...state, filter: action.payload };
    
    case 'SET_SORT':
      return { ...state, sortBy: action.payload };
    
    default:
      return state;
  }
}

function ItemManager() {
  const [state, dispatch] = useReducer(itemsReducer, initialState);

  // Action creators
  const actions = useMemo(() => ({
    setLoading: (loading) => dispatch({ type: 'SET_LOADING', payload: loading }),
    setError: (error) => dispatch({ type: 'SET_ERROR', payload: error }),
    setItems: (items) => dispatch({ type: 'SET_ITEMS', payload: items }),
    addItem: (item) => dispatch({ type: 'ADD_ITEM', payload: item }),
    removeItem: (id) => dispatch({ type: 'REMOVE_ITEM', payload: id }),
    toggleSelection: (id) => dispatch({ type: 'TOGGLE_ITEM_SELECTION', payload: id }),
    selectAll: () => dispatch({ type: 'SELECT_ALL' }),
    clearSelection: () => dispatch({ type: 'CLEAR_SELECTION' }),
    setFilter: (filter) => dispatch({ type: 'SET_FILTER', payload: filter }),
    setSort: (sortBy) => dispatch({ type: 'SET_SORT', payload: sortBy })
  }), []);

  // Load items
  useEffect(() => {
    const loadItems = async () => {
      actions.setLoading(true);
      try {
        const response = await fetch('/api/items');
        const items = await response.json();
        actions.setItems(items);
      } catch (error) {
        actions.setError(error.message);
      }
    };
    
    loadItems();
  }, [actions]);

  // Filtered and sorted items
  const displayItems = useMemo(() => {
    let filtered = state.items;

    // Apply filter
    if (state.filter !== 'all') {
      filtered = filtered.filter(item => item.status === state.filter);
    }

    // Apply sort
    filtered.sort((a, b) => {
      switch (state.sortBy) {
        case 'name':
          return a.name.localeCompare(b.name);
        case 'date':
          return new Date(b.createdAt) - new Date(a.createdAt);
        default:
          return 0;
      }
    });

    return filtered;
  }, [state.items, state.filter, state.sortBy]);

  if (state.loading) return <div>Loading...</div>;
  if (state.error) return <div>Error: {state.error}</div>;

  return (
    <div>
      <div className="controls">
        <select 
          value={state.filter} 
          onChange={(e) => actions.setFilter(e.target.value)}
        >
          <option value="all">All Items</option>
          <option value="active">Active</option>
          <option value="completed">Completed</option>
        </select>

        <select 
          value={state.sortBy} 
          onChange={(e) => actions.setSort(e.target.value)}
        >
          <option value="name">Sort by Name</option>
          <option value="date">Sort by Date</option>
        </select>

        <button onClick={actions.selectAll}>Select All</button>
        <button onClick={actions.clearSelection}>Clear Selection</button>
      </div>

      <div className="items">
        {displayItems.map(item => (
          <div 
            key={item.id}
            className={state.selectedItems.has(item.id) ? 'selected' : ''}
            onClick={() => actions.toggleSelection(item.id)}
          >
            {item.name}
          </div>
        ))}
      </div>

      <div>Selected: {state.selectedItems.size} items</div>
    </div>
  );
}
```

---

## 🌍 Context and State Management

### Advanced Context Patterns

```jsx
// Theme context with reducer
const ThemeContext = createContext();

const themeReducer = (state, action) => {
  switch (action.type) {
    case 'SET_THEME':
      return { ...state, theme: action.payload };
    case 'TOGGLE_DARK_MODE':
      return { ...state, darkMode: !state.darkMode };
    case 'SET_FONT_SIZE':
      return { ...state, fontSize: action.payload };
    default:
      return state;
  }
};

function ThemeProvider({ children }) {
  const [state, dispatch] = useReducer(themeReducer, {
    theme: 'default',
    darkMode: false,
    fontSize: 'medium'
  });

  // Persist theme to localStorage
  useEffect(() => {
    const savedTheme = localStorage.getItem('theme');
    if (savedTheme) {
      dispatch({ type: 'SET_THEME', payload: JSON.parse(savedTheme) });
    }
  }, []);

  useEffect(() => {
    localStorage.setItem('theme', JSON.stringify(state));
  }, [state]);

  const value = useMemo(() => ({
    ...state,
    setTheme: (theme) => dispatch({ type: 'SET_THEME', payload: theme }),
    toggleDarkMode: () => dispatch({ type: 'TOGGLE_DARK_MODE' }),
    setFontSize: (size) => dispatch({ type: 'SET_FONT_SIZE', payload: size })
  }), [state]);

  return (
    <ThemeContext.Provider value={value}>
      <div 
        className={`app ${state.theme} ${state.darkMode ? 'dark' : 'light'}`}
        style={{ fontSize: state.fontSize === 'large' ? '18px' : '16px' }}
      >
        {children}
      </div>
    </ThemeContext.Provider>
  );
}

// Multiple contexts composition
function AppProviders({ children }) {
  return (
    <ThemeProvider>
      <AuthProvider>
        <DataProvider>
          <NotificationProvider>
            {children}
          </NotificationProvider>
        </DataProvider>
      </AuthProvider>
    </ThemeProvider>
  );
}

// Hook for using theme context
function useTheme() {
  const context = useContext(ThemeContext);
  if (!context) {
    throw new Error('useTheme must be used within a ThemeProvider');
  }
  return context;
}
```

### State Management with Zustand

```jsx
import { create } from 'zustand';
import { subscribeWithSelector } from 'zustand/middleware';

// Create store with Zustand
const useStore = create(
  subscribeWithSelector((set, get) => ({
    // State
    user: null,
    todos: [],
    filter: 'all',
    
    // Actions
    setUser: (user) => set({ user }),
    
    addTodo: (text) => set((state) => ({
      todos: [...state.todos, {
        id: Date.now(),
        text,
        completed: false,
        createdAt: new Date()
      }]
    })),
    
    toggleTodo: (id) => set((state) => ({
      todos: state.todos.map(todo =>
        todo.id === id ? { ...todo, completed: !todo.completed } : todo
      )
    })),
    
    removeTodo: (id) => set((state) => ({
      todos: state.todos.filter(todo => todo.id !== id)
    })),
    
    setFilter: (filter) => set({ filter }),
    
    // Computed values
    get filteredTodos() {
      const { todos, filter } = get();
      switch (filter) {
        case 'completed':
          return todos.filter(todo => todo.completed);
        case 'active':
          return todos.filter(todo => !todo.completed);
        default:
          return todos;
      }
    },
    
    get stats() {
      const { todos } = get();
      return {
        total: todos.length,
        completed: todos.filter(t => t.completed).length,
        active: todos.filter(t => !t.completed).length
      };
    }
  }))
);

// Subscribe to state changes
useStore.subscribe(
  (state) => state.todos,
  (todos) => {
    localStorage.setItem('todos', JSON.stringify(todos));
  }
);

// Component using Zustand store
function TodoApp() {
  const {
    todos,
    filter,
    filteredTodos,
    stats,
    addTodo,
    toggleTodo,
    removeTodo,
    setFilter
  } = useStore();

  const [inputValue, setInputValue] = useState('');

  const handleSubmit = (e) => {
    e.preventDefault();
    if (inputValue.trim()) {
      addTodo(inputValue.trim());
      setInputValue('');
    }
  };

  return (
    <div>
      <form onSubmit={handleSubmit}>
        <input
          value={inputValue}
          onChange={(e) => setInputValue(e.target.value)}
          placeholder="Add todo..."
        />
        <button type="submit">Add</button>
      </form>

      <div>
        <button 
          onClick={() => setFilter('all')}
          className={filter === 'all' ? 'active' : ''}
        >
          All ({stats.total})
        </button>
        <button 
          onClick={() => setFilter('active')}
          className={filter === 'active' ? 'active' : ''}
        >
          Active ({stats.active})
        </button>
        <button 
          onClick={() => setFilter('completed')}
          className={filter === 'completed' ? 'active' : ''}
        >
          Completed ({stats.completed})
        </button>
      </div>

      <ul>
        {filteredTodos.map(todo => (
          <li key={todo.id}>
            <input
              type="checkbox"
              checked={todo.completed}
              onChange={() => toggleTodo(todo.id)}
            />
            <span className={todo.completed ? 'completed' : ''}>
              {todo.text}
            </span>
            <button onClick={() => removeTodo(todo.id)}>Delete</button>
          </li>
        ))}
      </ul>
    </div>
  );
}
```

This advanced React guide covers architecture patterns, performance optimization, advanced hooks, state management, and modern React development techniques for building scalable applications.
