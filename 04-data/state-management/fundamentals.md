
# ⚡ State Management Basics

## Table of Contents
- [What is State Management?](#what-is-state-management)
- [Local Component State](#local-component-state)
- [Lifting State Up](#lifting-state-up)
- [Context API](#context-api)
- [Redux Fundamentals](#redux-fundamentals)
- [Redux Toolkit](#redux-toolkit)
- [State Management Patterns](#state-management-patterns)
- [Performance Optimization](#performance-optimization)

---

## 🎯 What is State Management?

**State Management** is the process of managing data that changes over time in your application.

### Types of State:
- **Local State**: Component-specific data
- **Global State**: Application-wide data
- **Server State**: Data from APIs
- **URL State**: Data in the URL

### State Management Solutions:
```
Local State ────┐
                ├──→ React Context
Shared State ───┤
                ├──→ Redux
Global State ───┘
                └──→ Zustand
```

---

## 🏠 Local Component State

### useState Hook

```jsx
import React, { useState } from 'react';

function Counter() {
    const [count, setCount] = useState(0);
    const [name, setName] = useState('');
    
    const increment = () => setCount(count + 1);
    const decrement = () => setCount(count - 1);
    const reset = () => setCount(0);
    
    return (
        <div>
            <h2>Counter: {count}</h2>
            <button onClick={increment}>+</button>
            <button onClick={decrement}>-</button>
            <button onClick={reset}>Reset</button>
            
            <input
                type="text"
                value={name}
                onChange={(e) => setName(e.target.value)}
                placeholder="Enter name"
            />
            <p>Hello, {name}!</p>
        </div>
    );
}
```

### useReducer Hook

```jsx
import React, { useReducer } from 'react';

// Reducer function
const counterReducer = (state, action) => {
    switch (action.type) {
        case 'INCREMENT':
            return { count: state.count + 1 };
        case 'DECREMENT':
            return { count: state.count - 1 };
        case 'RESET':
            return { count: 0 };
        case 'SET_COUNT':
            return { count: action.payload };
        default:
            throw new Error(`Unknown action: ${action.type}`);
    }
};

function AdvancedCounter() {
    const [state, dispatch] = useReducer(counterReducer, { count: 0 });
    
    return (
        <div>
            <h2>Count: {state.count}</h2>
            <button onClick={() => dispatch({ type: 'INCREMENT' })}>
                +
            </button>
            <button onClick={() => dispatch({ type: 'DECREMENT' })}>
                -
            </button>
            <button onClick={() => dispatch({ type: 'RESET' })}>
                Reset
            </button>
            <button onClick={() => dispatch({ type: 'SET_COUNT', payload: 10 })}>
                Set to 10
            </button>
        </div>
    );
}
```

---

## ⬆️ Lifting State Up

### Shared State Between Components

```jsx
import React, { useState } from 'react';

// Child components
function TemperatureInput({ scale, temperature, onTemperatureChange }) {
    const handleChange = (e) => {
        onTemperatureChange(e.target.value);
    };
    
    return (
        <fieldset>
            <legend>Enter temperature in {scale}:</legend>
            <input value={temperature} onChange={handleChange} />
        </fieldset>
    );
}

function BoilingVerdict({ celsius }) {
    if (celsius >= 100) {
        return <p>The water would boil.</p>;
    }
    return <p>The water would not boil.</p>;
}

// Conversion functions
function toCelsius(fahrenheit) {
    return (fahrenheit - 32) * 5 / 9;
}

function toFahrenheit(celsius) {
    return (celsius * 9 / 5) + 32;
}

function tryConvert(temperature, convert) {
    const input = parseFloat(temperature);
    if (Number.isNaN(input)) {
        return '';
    }
    const output = convert(input);
    const rounded = Math.round(output * 1000) / 1000;
    return rounded.toString();
}

// Parent component with lifted state
function Calculator() {
    const [temperature, setTemperature] = useState('');
    const [scale, setScale] = useState('c');
    
    const handleCelsiusChange = (temperature) => {
        setScale('c');
        setTemperature(temperature);
    };
    
    const handleFahrenheitChange = (temperature) => {
        setScale('f');
        setTemperature(temperature);
    };
    
    const celsius = scale === 'f' ? tryConvert(temperature, toCelsius) : temperature;
    const fahrenheit = scale === 'c' ? tryConvert(temperature, toFahrenheit) : temperature;
    
    return (
        <div>
            <TemperatureInput
                scale="Celsius"
                temperature={celsius}
                onTemperatureChange={handleCelsiusChange}
            />
            <TemperatureInput
                scale="Fahrenheit"
                temperature={fahrenheit}
                onTemperatureChange={handleFahrenheitChange}
            />
            <BoilingVerdict celsius={parseFloat(celsius)} />
        </div>
    );
}
```

---

## 🌐 Context API

### Creating and Using Context

```jsx
import React, { createContext, useContext, useReducer } from 'react';

// Create Context
const ThemeContext = createContext();
const UserContext = createContext();

// Theme Provider
const themeReducer = (state, action) => {
    switch (action.type) {
        case 'TOGGLE_THEME':
            return {
                ...state,
                darkMode: !state.darkMode
            };
        case 'SET_THEME':
            return {
                ...state,
                darkMode: action.payload
            };
        default:
            return state;
    }
};

function ThemeProvider({ children }) {
    const [state, dispatch] = useReducer(themeReducer, {
        darkMode: false,
        primaryColor: '#007bff'
    });
    
    const toggleTheme = () => {
        dispatch({ type: 'TOGGLE_THEME' });
    };
    
    const setTheme = (darkMode) => {
        dispatch({ type: 'SET_THEME', payload: darkMode });
    };
    
    return (
        <ThemeContext.Provider value={{
            ...state,
            toggleTheme,
            setTheme
        }}>
            {children}
        </ThemeContext.Provider>
    );
}

// User Provider
function UserProvider({ children }) {
    const [user, setUser] = useState(null);
    const [loading, setLoading] = useState(false);
    
    const login = async (email, password) => {
        setLoading(true);
        try {
            const response = await fetch('/api/login', {
                method: 'POST',
                headers: { 'Content-Type': 'application/json' },
                body: JSON.stringify({ email, password })
            });
            const userData = await response.json();
            setUser(userData);
        } catch (error) {
            console.error('Login failed:', error);
        } finally {
            setLoading(false);
        }
    };
    
    const logout = () => {
        setUser(null);
        localStorage.removeItem('token');
    };
    
    return (
        <UserContext.Provider value={{
            user,
            loading,
            login,
            logout
        }}>
            {children}
        </UserContext.Provider>
    );
}

// Custom hooks
export const useTheme = () => {
    const context = useContext(ThemeContext);
    if (!context) {
        throw new Error('useTheme must be used within ThemeProvider');
    }
    return context;
};

export const useUser = () => {
    const context = useContext(UserContext);
    if (!context) {
        throw new Error('useUser must be used within UserProvider');
    }
    return context;
};

// Usage in components
function Header() {
    const { darkMode, toggleTheme } = useTheme();
    const { user, logout } = useUser();
    
    return (
        <header className={darkMode ? 'dark' : 'light'}>
            <h1>My App</h1>
            <button onClick={toggleTheme}>
                {darkMode ? '☀️' : '🌙'}
            </button>
            {user ? (
                <div>
                    <span>Welcome, {user.name}!</span>
                    <button onClick={logout}>Logout</button>
                </div>
            ) : (
                <button>Login</button>
            )}
        </header>
    );
}

// App with providers
function App() {
    return (
        <ThemeProvider>
            <UserProvider>
                <div className="app">
                    <Header />
                    <main>
                        {/* Your app content */}
                    </main>
                </div>
            </UserProvider>
        </ThemeProvider>
    );
}
```

---

## 🔄 Redux Fundamentals

### Core Concepts

```
Action ──→ Reducer ──→ Store ──→ Component
  ↑                                  ↓
  └────────── dispatch ──────────────┘
```

### Basic Redux Setup

```jsx
// actions/counterActions.js
export const INCREMENT = 'INCREMENT';
export const DECREMENT = 'DECREMENT';
export const RESET = 'RESET';

export const increment = () => ({ type: INCREMENT });
export const decrement = () => ({ type: DECREMENT });
export const reset = () => ({ type: RESET });

// reducers/counterReducer.js
import { INCREMENT, DECREMENT, RESET } from '../actions/counterActions';

const initialState = {
    count: 0
};

export const counterReducer = (state = initialState, action) => {
    switch (action.type) {
        case INCREMENT:
            return {
                ...state,
                count: state.count + 1
            };
        case DECREMENT:
            return {
                ...state,
                count: state.count - 1
            };
        case RESET:
            return {
                ...state,
                count: 0
            };
        default:
            return state;
    }
};

// store/store.js
import { createStore, combineReducers } from 'redux';
import { counterReducer } from '../reducers/counterReducer';

const rootReducer = combineReducers({
    counter: counterReducer
});

export const store = createStore(rootReducer);

// index.js
import React from 'react';
import ReactDOM from 'react-dom';
import { Provider } from 'react-redux';
import { store } from './store/store';
import App from './App';

ReactDOM.render(
    <Provider store={store}>
        <App />
    </Provider>,
    document.getElementById('root')
);

// components/Counter.js
import React from 'react';
import { useSelector, useDispatch } from 'react-redux';
import { increment, decrement, reset } from '../actions/counterActions';

function Counter() {
    const count = useSelector(state => state.counter.count);
    const dispatch = useDispatch();
    
    return (
        <div>
            <h2>Count: {count}</h2>
            <button onClick={() => dispatch(increment())}>+</button>
            <button onClick={() => dispatch(decrement())}>-</button>
            <button onClick={() => dispatch(reset())}>Reset</button>
        </div>
    );
}

export default Counter;
```

---

## 🛠️ Redux Toolkit

### Modern Redux with RTK

```jsx
// features/counterSlice.js
import { createSlice } from '@reduxjs/toolkit';

const counterSlice = createSlice({
    name: 'counter',
    initialState: {
        value: 0
    },
    reducers: {
        increment: (state) => {
            state.value += 1;
        },
        decrement: (state) => {
            state.value -= 1;
        },
        incrementByAmount: (state, action) => {
            state.value += action.payload;
        },
        reset: (state) => {
            state.value = 0;
        }
    }
});

export const { increment, decrement, incrementByAmount, reset } = counterSlice.actions;
export default counterSlice.reducer;

// store/store.js
import { configureStore } from '@reduxjs/toolkit';
import counterReducer from '../features/counterSlice';

export const store = configureStore({
    reducer: {
        counter: counterReducer
    }
});

// Async actions with createAsyncThunk
import { createAsyncThunk, createSlice } from '@reduxjs/toolkit';

export const fetchUserById = createAsyncThunk(
    'users/fetchById',
    async (userId) => {
        const response = await fetch(`/api/users/${userId}`);
        return response.json();
    }
);

const usersSlice = createSlice({
    name: 'users',
    initialState: {
        entities: [],
        loading: 'idle',
        error: null
    },
    reducers: {
        // Regular reducers
    },
    extraReducers: (builder) => {
        builder
            .addCase(fetchUserById.pending, (state) => {
                state.loading = 'pending';
            })
            .addCase(fetchUserById.fulfilled, (state, action) => {
                state.loading = 'idle';
                state.entities.push(action.payload);
            })
            .addCase(fetchUserById.rejected, (state, action) => {
                state.loading = 'idle';
                state.error = action.error.message;
            });
    }
});

export default usersSlice.reducer;
```

---

## 🎨 State Management Patterns

### Container/Presentational Pattern

```jsx
// Presentational Component
function TodoListView({ todos, onToggle, onDelete, onAdd }) {
    const [newTodo, setNewTodo] = useState('');
    
    const handleSubmit = (e) => {
        e.preventDefault();
        if (newTodo.trim()) {
            onAdd(newTodo);
            setNewTodo('');
        }
    };
    
    return (
        <div>
            <form onSubmit={handleSubmit}>
                <input
                    value={newTodo}
                    onChange={(e) => setNewTodo(e.target.value)}
                    placeholder="Add todo..."
                />
                <button type="submit">Add</button>
            </form>
            
            <ul>
                {todos.map(todo => (
                    <li key={todo.id}>
                        <input
                            type="checkbox"
                            checked={todo.completed}
                            onChange={() => onToggle(todo.id)}
                        />
                        <span className={todo.completed ? 'completed' : ''}>
                            {todo.text}
                        </span>
                        <button onClick={() => onDelete(todo.id)}>
                            Delete
                        </button>
                    </li>
                ))}
            </ul>
        </div>
    );
}

// Container Component
function TodoListContainer() {
    const todos = useSelector(state => state.todos);
    const dispatch = useDispatch();
    
    const handleToggle = (id) => {
        dispatch(toggleTodo(id));
    };
    
    const handleDelete = (id) => {
        dispatch(deleteTodo(id));
    };
    
    const handleAdd = (text) => {
        dispatch(addTodo(text));
    };
    
    return (
        <TodoListView
            todos={todos}
            onToggle={handleToggle}
            onDelete={handleDelete}
            onAdd={handleAdd}
        />
    );
}
```

### Custom Hooks Pattern

```jsx
// Custom hook for todo management
function useTodos() {
    const [todos, setTodos] = useState([]);
    const [loading, setLoading] = useState(false);
    const [error, setError] = useState(null);
    
    const addTodo = useCallback(async (text) => {
        setLoading(true);
        try {
            const response = await fetch('/api/todos', {
                method: 'POST',
                headers: { 'Content-Type': 'application/json' },
                body: JSON.stringify({ text })
            });
            const newTodo = await response.json();
            setTodos(prev => [...prev, newTodo]);
        } catch (err) {
            setError(err.message);
        } finally {
            setLoading(false);
        }
    }, []);
    
    const toggleTodo = useCallback((id) => {
        setTodos(prev => 
            prev.map(todo => 
                todo.id === id 
                    ? { ...todo, completed: !todo.completed }
                    : todo
            )
        );
    }, []);
    
    const deleteTodo = useCallback((id) => {
        setTodos(prev => prev.filter(todo => todo.id !== id));
    }, []);
    
    const fetchTodos = useCallback(async () => {
        setLoading(true);
        try {
            const response = await fetch('/api/todos');
            const todosData = await response.json();
            setTodos(todosData);
        } catch (err) {
            setError(err.message);
        } finally {
            setLoading(false);
        }
    }, []);
    
    useEffect(() => {
        fetchTodos();
    }, [fetchTodos]);
    
    return {
        todos,
        loading,
        error,
        addTodo,
        toggleTodo,
        deleteTodo,
        refreshTodos: fetchTodos
    };
}

// Usage
function TodoApp() {
    const {
        todos,
        loading,
        error,
        addTodo,
        toggleTodo,
        deleteTodo
    } = useTodos();
    
    if (loading) return <div>Loading...</div>;
    if (error) return <div>Error: {error}</div>;
    
    return (
        <TodoListView
            todos={todos}
            onAdd={addTodo}
            onToggle={toggleTodo}
            onDelete={deleteTodo}
        />
    );
}
```

---

## ⚡ Performance Optimization

### Memoization

```jsx
import React, { memo, useMemo, useCallback } from 'react';

// Memoized component
const ExpensiveComponent = memo(({ items, onItemClick }) => {
    const expensiveValue = useMemo(() => {
        return items.reduce((sum, item) => sum + item.value, 0);
    }, [items]);
    
    return (
        <div>
            <p>Total: {expensiveValue}</p>
            {items.map(item => (
                <Item
                    key={item.id}
                    item={item}
                    onClick={onItemClick}
                />
            ))}
        </div>
    );
});

function ParentComponent() {
    const [items, setItems] = useState([]);
    const [filter, setFilter] = useState('');
    
    const filteredItems = useMemo(() => {
        return items.filter(item => 
            item.name.toLowerCase().includes(filter.toLowerCase())
        );
    }, [items, filter]);
    
    const handleItemClick = useCallback((itemId) => {
        setItems(prev => 
            prev.map(item => 
                item.id === itemId 
                    ? { ...item, selected: !item.selected }
                    : item
            )
        );
    }, []);
    
    return (
        <div>
            <input
                value={filter}
                onChange={(e) => setFilter(e.target.value)}
                placeholder="Filter items..."
            />
            <ExpensiveComponent
                items={filteredItems}
                onItemClick={handleItemClick}
            />
        </div>
    );
}
```

### Redux Performance

```jsx
// Selective subscriptions
function TodoItem({ todoId }) {
    const todo = useSelector(state => 
        state.todos.find(todo => todo.id === todoId)
    );
    
    // This component only re-renders when this specific todo changes
    return (
        <div>
            <span>{todo.text}</span>
            <input
                type="checkbox"
                checked={todo.completed}
                onChange={() => dispatch(toggleTodo(todoId))}
            />
        </div>
    );
}

// Reselect for memoized selectors
import { createSelector } from 'reselect';

const selectTodos = state => state.todos;
const selectFilter = state => state.filter;

const selectVisibleTodos = createSelector(
    [selectTodos, selectFilter],
    (todos, filter) => {
        switch (filter) {
            case 'SHOW_COMPLETED':
                return todos.filter(todo => todo.completed);
            case 'SHOW_ACTIVE':
                return todos.filter(todo => !todo.completed);
            default:
                return todos;
        }
    }
);
```

---

## 📚 Summary

### Key Concepts:
- **Local state** for component-specific data
- **Context API** for moderate global state
- **Redux** for complex global state
- **Custom hooks** for reusable state logic
- **Performance optimization** with memoization

### When to Use Each:
- **useState**: Simple local state
- **useReducer**: Complex local state logic
- **Context**: Moderate global state (theme, user)
- **Redux**: Complex global state with time travel debugging
- **Custom hooks**: Reusable stateful logic

### Next Steps:
1. Practice with different state management patterns
2. Learn advanced Redux patterns (RTK Query)
3. Explore other solutions (Zustand, Jotai, Valtio)
4. Master performance optimization techniques
5. Build complex applications with proper state architecture
