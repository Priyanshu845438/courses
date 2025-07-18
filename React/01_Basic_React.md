
# 🚀 Basic React

## Table of Contents
- [What is React?](#what-is-react)
- [Setting Up React](#setting-up-react)
- [JSX Basics](#jsx-basics)
- [Components](#components)
- [Props](#props)
- [State](#state)
- [Event Handling](#event-handling)
- [Conditional Rendering](#conditional-rendering)
- [Lists and Keys](#lists-and-keys)
- [Forms](#forms)
- [Best Practices](#best-practices)

---

## 🎯 What is React?

**React** is a JavaScript library for building user interfaces, especially web applications. It's maintained by Facebook and focuses on creating reusable UI components.

### Key Features:
- **Component-based**: Build encapsulated components
- **Virtual DOM**: Efficient rendering and updates
- **Unidirectional data flow**: Predictable state management
- **JSX**: Write HTML-like syntax in JavaScript
- **Reusable**: Components can be reused across the application

### React vs Other Frameworks

| Feature | React | Vue | Angular |
|---------|-------|-----|---------|
| **Learning Curve** | Moderate | Easy | Steep |
| **Size** | Small | Small | Large |
| **Type** | Library | Framework | Framework |
| **Data Binding** | One-way | Two-way | Two-way |

---

## 🛠 Setting Up React

### Create React App (Recommended for beginners)

```bash
# Install Node.js first, then run:
npx create-react-app my-app
cd my-app
npm start
```

### Basic Project Structure

```
my-app/
├── public/
│   ├── index.html
│   └── favicon.ico
├── src/
│   ├── components/
│   ├── App.js
│   ├── App.css
│   ├── index.js
│   └── index.css
├── package.json
└── README.md
```

### Alternative: Vite (Faster)

```bash
npm create vite@latest my-react-app -- --template react
cd my-react-app
npm install
npm run dev
```

---

## 🏗 JSX Basics

JSX (JavaScript XML) allows you to write HTML-like syntax in JavaScript.

### Basic JSX

```jsx
// Simple JSX element
const element = <h1>Hello, World!</h1>;

// JSX with variables
const name = "React";
const greeting = <h1>Hello, {name}!</h1>;

// JSX with expressions
const user = { name: "Alice", age: 25 };
const userInfo = (
    <div>
        <h2>{user.name}</h2>
        <p>Age: {user.age}</p>
        <p>Born in: {2024 - user.age}</p>
    </div>
);
```

### JSX Rules

```jsx
// 1. Must have one parent element
const valid = (
    <div>
        <h1>Title</h1>
        <p>Content</p>
    </div>
);

// Or use React Fragment
const alsoValid = (
    <>
        <h1>Title</h1>
        <p>Content</p>
    </>
);

// 2. Use className instead of class
const styledElement = <div className="container">Content</div>;

// 3. Self-closing tags must end with />
const image = <img src="photo.jpg" alt="Description" />;
const input = <input type="text" placeholder="Enter text" />;

// 4. Use camelCase for attributes
const button = <button onClick={handleClick}>Click me</button>;
```

---

## 🧩 Components

Components are the building blocks of React applications.

### Function Components (Recommended)

```jsx
// Simple function component
function Welcome() {
    return <h1>Welcome to React!</h1>;
}

// Arrow function component
const Greeting = () => {
    return <h2>Hello from arrow function!</h2>;
};

// Component with logic
function CurrentTime() {
    const now = new Date();
    const timeString = now.toLocaleTimeString();
    
    return (
        <div>
            <h3>Current Time</h3>
            <p>{timeString}</p>
        </div>
    );
}

// Using components
function App() {
    return (
        <div>
            <Welcome />
            <Greeting />
            <CurrentTime />
        </div>
    );
}
```

### Class Components (Legacy)

```jsx
import React, { Component } from 'react';

class Welcome extends Component {
    render() {
        return <h1>Welcome to React!</h1>;
    }
}

// With constructor
class Counter extends Component {
    constructor(props) {
        super(props);
        this.state = { count: 0 };
    }
    
    render() {
        return (
            <div>
                <p>Count: {this.state.count}</p>
            </div>
        );
    }
}
```

---

## 📨 Props

Props (properties) are how you pass data to components.

### Basic Props

```jsx
// Component that receives props
function UserCard(props) {
    return (
        <div className="user-card">
            <h3>{props.name}</h3>
            <p>Age: {props.age}</p>
            <p>Email: {props.email}</p>
        </div>
    );
}

// Using the component with props
function App() {
    return (
        <div>
            <UserCard 
                name="Alice Johnson" 
                age={25} 
                email="alice@example.com" 
            />
            <UserCard 
                name="Bob Smith" 
                age={30} 
                email="bob@example.com" 
            />
        </div>
    );
}
```

### Props Destructuring

```jsx
// Destructuring in function parameters
function UserCard({ name, age, email }) {
    return (
        <div className="user-card">
            <h3>{name}</h3>
            <p>Age: {age}</p>
            <p>Email: {email}</p>
        </div>
    );
}

// Default props
function Button({ text = "Click me", color = "blue" }) {
    return (
        <button style={{ backgroundColor: color }}>
            {text}
        </button>
    );
}

// Props validation with PropTypes
import PropTypes from 'prop-types';

function UserCard({ name, age, email }) {
    return (
        <div className="user-card">
            <h3>{name}</h3>
            <p>Age: {age}</p>
            <p>Email: {email}</p>
        </div>
    );
}

UserCard.propTypes = {
    name: PropTypes.string.isRequired,
    age: PropTypes.number.isRequired,
    email: PropTypes.string.isRequired
};
```

---

## 🔄 State

State allows components to manage their own data that can change over time.

### useState Hook

```jsx
import React, { useState } from 'react';

// Simple counter
function Counter() {
    const [count, setCount] = useState(0);
    
    return (
        <div>
            <p>Count: {count}</p>
            <button onClick={() => setCount(count + 1)}>
                Increment
            </button>
            <button onClick={() => setCount(count - 1)}>
                Decrement
            </button>
            <button onClick={() => setCount(0)}>
                Reset
            </button>
        </div>
    );
}

// Multiple state variables
function UserForm() {
    const [name, setName] = useState('');
    const [email, setEmail] = useState('');
    const [age, setAge] = useState(0);
    
    return (
        <form>
            <input
                type="text"
                placeholder="Name"
                value={name}
                onChange={(e) => setName(e.target.value)}
            />
            <input
                type="email"
                placeholder="Email"
                value={email}
                onChange={(e) => setEmail(e.target.value)}
            />
            <input
                type="number"
                placeholder="Age"
                value={age}
                onChange={(e) => setAge(parseInt(e.target.value))}
            />
            <p>Preview: {name}, {email}, {age}</p>
        </form>
    );
}
```

### State with Objects and Arrays

```jsx
// Object state
function Profile() {
    const [user, setUser] = useState({
        name: '',
        email: '',
        age: 0
    });
    
    const updateUser = (field, value) => {
        setUser(prevUser => ({
            ...prevUser,
            [field]: value
        }));
    };
    
    return (
        <div>
            <input
                placeholder="Name"
                onChange={(e) => updateUser('name', e.target.value)}
            />
            <input
                placeholder="Email"
                onChange={(e) => updateUser('email', e.target.value)}
            />
            <p>User: {JSON.stringify(user)}</p>
        </div>
    );
}

// Array state
function TodoList() {
    const [todos, setTodos] = useState([]);
    const [inputValue, setInputValue] = useState('');
    
    const addTodo = () => {
        if (inputValue.trim()) {
            setTodos([...todos, {
                id: Date.now(),
                text: inputValue,
                completed: false
            }]);
            setInputValue('');
        }
    };
    
    const removeTodo = (id) => {
        setTodos(todos.filter(todo => todo.id !== id));
    };
    
    return (
        <div>
            <input
                value={inputValue}
                onChange={(e) => setInputValue(e.target.value)}
                placeholder="Add a todo"
            />
            <button onClick={addTodo}>Add</button>
            <ul>
                {todos.map(todo => (
                    <li key={todo.id}>
                        {todo.text}
                        <button onClick={() => removeTodo(todo.id)}>
                            Remove
                        </button>
                    </li>
                ))}
            </ul>
        </div>
    );
}
```

---

## 🖱 Event Handling

### Basic Events

```jsx
function EventExamples() {
    const [message, setMessage] = useState('');
    
    const handleClick = () => {
        setMessage('Button was clicked!');
    };
    
    const handleMouseOver = () => {
        setMessage('Mouse is over the button');
    };
    
    const handleSubmit = (e) => {
        e.preventDefault();
        setMessage('Form submitted!');
    };
    
    const handleKeyPress = (e) => {
        if (e.key === 'Enter') {
            setMessage('Enter key pressed!');
        }
    };
    
    return (
        <div>
            <button 
                onClick={handleClick}
                onMouseOver={handleMouseOver}
            >
                Hover or Click Me
            </button>
            
            <form onSubmit={handleSubmit}>
                <input 
                    onKeyPress={handleKeyPress}
                    placeholder="Press Enter or submit"
                />
                <button type="submit">Submit</button>
            </form>
            
            <p>{message}</p>
        </div>
    );
}
```

### Passing Parameters to Event Handlers

```jsx
function ButtonList() {
    const [selectedId, setSelectedId] = useState(null);
    
    const buttons = [
        { id: 1, name: 'Button 1', color: 'red' },
        { id: 2, name: 'Button 2', color: 'blue' },
        { id: 3, name: 'Button 3', color: 'green' }
    ];
    
    const handleButtonClick = (id, name) => {
        setSelectedId(id);
        console.log(`Clicked ${name} with ID ${id}`);
    };
    
    return (
        <div>
            {buttons.map(button => (
                <button
                    key={button.id}
                    style={{ 
                        backgroundColor: button.color,
                        margin: '5px' 
                    }}
                    onClick={() => handleButtonClick(button.id, button.name)}
                >
                    {button.name}
                </button>
            ))}
            <p>Selected ID: {selectedId}</p>
        </div>
    );
}
```

---

## 🔀 Conditional Rendering

### If/Else with Ternary Operator

```jsx
function UserGreeting({ isLoggedIn, username }) {
    return (
        <div>
            {isLoggedIn ? (
                <h2>Welcome back, {username}!</h2>
            ) : (
                <h2>Please log in</h2>
            )}
        </div>
    );
}

// Usage
function App() {
    const [isLoggedIn, setIsLoggedIn] = useState(false);
    const [username, setUsername] = useState('Alice');
    
    return (
        <div>
            <UserGreeting 
                isLoggedIn={isLoggedIn} 
                username={username} 
            />
            <button onClick={() => setIsLoggedIn(!isLoggedIn)}>
                {isLoggedIn ? 'Logout' : 'Login'}
            </button>
        </div>
    );
}
```

### Logical AND Operator

```jsx
function Notifications({ notifications }) {
    return (
        <div>
            <h3>Notifications</h3>
            {notifications.length > 0 && (
                <p>You have {notifications.length} new notifications</p>
            )}
            {notifications.length === 0 && (
                <p>No new notifications</p>
            )}
        </div>
    );
}
```

### Multiple Conditions

```jsx
function StatusIndicator({ status }) {
    const getStatusColor = () => {
        switch(status) {
            case 'online': return 'green';
            case 'away': return 'yellow';
            case 'offline': return 'red';
            default: return 'gray';
        }
    };
    
    const getStatusMessage = () => {
        if (status === 'online') {
            return 'You are online and available';
        } else if (status === 'away') {
            return 'You are away';
        } else if (status === 'offline') {
            return 'You are offline';
        } else {
            return 'Status unknown';
        }
    };
    
    return (
        <div>
            <div 
                style={{
                    width: '20px',
                    height: '20px',
                    borderRadius: '50%',
                    backgroundColor: getStatusColor(),
                    display: 'inline-block',
                    marginRight: '10px'
                }}
            />
            <span>{getStatusMessage()}</span>
        </div>
    );
}
```

---

## 📋 Lists and Keys

### Rendering Lists

```jsx
function ShoppingList() {
    const items = [
        { id: 1, name: 'Apples', quantity: 5, price: 2.50 },
        { id: 2, name: 'Bananas', quantity: 3, price: 1.20 },
        { id: 3, name: 'Oranges', quantity: 8, price: 3.00 }
    ];
    
    return (
        <div>
            <h2>Shopping List</h2>
            <ul>
                {items.map(item => (
                    <li key={item.id}>
                        <strong>{item.name}</strong> - 
                        Quantity: {item.quantity}, 
                        Price: ${item.price}
                    </li>
                ))}
            </ul>
        </div>
    );
}

// More complex list with components
function ContactList({ contacts }) {
    return (
        <div>
            <h2>Contacts</h2>
            {contacts.map(contact => (
                <ContactCard 
                    key={contact.id}
                    contact={contact}
                />
            ))}
        </div>
    );
}

function ContactCard({ contact }) {
    return (
        <div style={{ 
            border: '1px solid #ccc', 
            padding: '10px', 
            margin: '10px 0' 
        }}>
            <h3>{contact.name}</h3>
            <p>Email: {contact.email}</p>
            <p>Phone: {contact.phone}</p>
        </div>
    );
}
```

### Filtering and Sorting Lists

```jsx
function ProductList() {
    const [products] = useState([
        { id: 1, name: 'Laptop', category: 'Electronics', price: 999 },
        { id: 2, name: 'T-Shirt', category: 'Clothing', price: 25 },
        { id: 3, name: 'Phone', category: 'Electronics', price: 699 },
        { id: 4, name: 'Jeans', category: 'Clothing', price: 60 }
    ]);
    
    const [filter, setFilter] = useState('All');
    const [sortBy, setSortBy] = useState('name');
    
    const filteredProducts = products.filter(product => 
        filter === 'All' || product.category === filter
    );
    
    const sortedProducts = [...filteredProducts].sort((a, b) => {
        if (sortBy === 'price') {
            return a.price - b.price;
        }
        return a[sortBy].localeCompare(b[sortBy]);
    });
    
    return (
        <div>
            <div>
                <label>Filter by category: </label>
                <select 
                    value={filter} 
                    onChange={(e) => setFilter(e.target.value)}
                >
                    <option value="All">All</option>
                    <option value="Electronics">Electronics</option>
                    <option value="Clothing">Clothing</option>
                </select>
                
                <label> Sort by: </label>
                <select 
                    value={sortBy} 
                    onChange={(e) => setSortBy(e.target.value)}
                >
                    <option value="name">Name</option>
                    <option value="price">Price</option>
                </select>
            </div>
            
            <div>
                {sortedProducts.map(product => (
                    <div key={product.id} style={{ margin: '10px 0' }}>
                        <h4>{product.name}</h4>
                        <p>Category: {product.category}</p>
                        <p>Price: ${product.price}</p>
                    </div>
                ))}
            </div>
        </div>
    );
}
```

---

## 📝 Forms

### Controlled Components

```jsx
function ContactForm() {
    const [formData, setFormData] = useState({
        name: '',
        email: '',
        message: '',
        category: 'general',
        newsletter: false
    });
    
    const [errors, setErrors] = useState({});
    
    const handleChange = (e) => {
        const { name, value, type, checked } = e.target;
        setFormData(prev => ({
            ...prev,
            [name]: type === 'checkbox' ? checked : value
        }));
    };
    
    const validateForm = () => {
        const newErrors = {};
        
        if (!formData.name.trim()) {
            newErrors.name = 'Name is required';
        }
        
        if (!formData.email.trim()) {
            newErrors.email = 'Email is required';
        } else if (!/\S+@\S+\.\S+/.test(formData.email)) {
            newErrors.email = 'Email is invalid';
        }
        
        if (!formData.message.trim()) {
            newErrors.message = 'Message is required';
        }
        
        setErrors(newErrors);
        return Object.keys(newErrors).length === 0;
    };
    
    const handleSubmit = (e) => {
        e.preventDefault();
        
        if (validateForm()) {
            console.log('Form submitted:', formData);
            // Here you would typically send data to a server
            alert('Form submitted successfully!');
            
            // Reset form
            setFormData({
                name: '',
                email: '',
                message: '',
                category: 'general',
                newsletter: false
            });
        }
    };
    
    return (
        <form onSubmit={handleSubmit} style={{ maxWidth: '500px' }}>
            <div style={{ marginBottom: '15px' }}>
                <label>Name:</label>
                <input
                    type="text"
                    name="name"
                    value={formData.name}
                    onChange={handleChange}
                    style={{ 
                        width: '100%', 
                        padding: '8px',
                        border: errors.name ? '1px solid red' : '1px solid #ccc'
                    }}
                />
                {errors.name && <span style={{ color: 'red', fontSize: '14px' }}>{errors.name}</span>}
            </div>
            
            <div style={{ marginBottom: '15px' }}>
                <label>Email:</label>
                <input
                    type="email"
                    name="email"
                    value={formData.email}
                    onChange={handleChange}
                    style={{ 
                        width: '100%', 
                        padding: '8px',
                        border: errors.email ? '1px solid red' : '1px solid #ccc'
                    }}
                />
                {errors.email && <span style={{ color: 'red', fontSize: '14px' }}>{errors.email}</span>}
            </div>
            
            <div style={{ marginBottom: '15px' }}>
                <label>Category:</label>
                <select
                    name="category"
                    value={formData.category}
                    onChange={handleChange}
                    style={{ width: '100%', padding: '8px' }}
                >
                    <option value="general">General</option>
                    <option value="support">Support</option>
                    <option value="sales">Sales</option>
                </select>
            </div>
            
            <div style={{ marginBottom: '15px' }}>
                <label>Message:</label>
                <textarea
                    name="message"
                    value={formData.message}
                    onChange={handleChange}
                    rows="4"
                    style={{ 
                        width: '100%', 
                        padding: '8px',
                        border: errors.message ? '1px solid red' : '1px solid #ccc'
                    }}
                />
                {errors.message && <span style={{ color: 'red', fontSize: '14px' }}>{errors.message}</span>}
            </div>
            
            <div style={{ marginBottom: '15px' }}>
                <label>
                    <input
                        type="checkbox"
                        name="newsletter"
                        checked={formData.newsletter}
                        onChange={handleChange}
                    />
                    Subscribe to newsletter
                </label>
            </div>
            
            <button 
                type="submit"
                style={{ 
                    backgroundColor: '#007bff', 
                    color: 'white', 
                    padding: '10px 20px',
                    border: 'none',
                    cursor: 'pointer'
                }}
            >
                Submit
            </button>
        </form>
    );
}
```

---

## ✅ Best Practices

### Component Organization

```jsx
// 1. Import external libraries first
import React, { useState, useEffect } from 'react';
import PropTypes from 'prop-types';

// 2. Import local components and utilities
import Button from './Button';
import { formatDate } from '../utils/dateUtils';

// 3. Component definition
function UserProfile({ userId }) {
    // 4. State declarations at the top
    const [user, setUser] = useState(null);
    const [loading, setLoading] = useState(true);
    
    // 5. Effects after state
    useEffect(() => {
        fetchUser(userId);
    }, [userId]);
    
    // 6. Event handlers and other functions
    const fetchUser = async (id) => {
        try {
            setLoading(true);
            const response = await fetch(`/api/users/${id}`);
            const userData = await response.json();
            setUser(userData);
        } catch (error) {
            console.error('Error fetching user:', error);
        } finally {
            setLoading(false);
        }
    };
    
    // 7. Early returns for loading/error states
    if (loading) return <div>Loading...</div>;
    if (!user) return <div>User not found</div>;
    
    // 8. Main render
    return (
        <div className="user-profile">
            <h2>{user.name}</h2>
            <p>Joined: {formatDate(user.joinDate)}</p>
            <Button onClick={() => console.log('Profile clicked')}>
                View Details
            </Button>
        </div>
    );
}

// 9. PropTypes after component
UserProfile.propTypes = {
    userId: PropTypes.number.isRequired
};

export default UserProfile;
```

### Performance Tips

```jsx
// Use React.memo for expensive components
const ExpensiveComponent = React.memo(function ExpensiveComponent({ data }) {
    const processedData = useMemo(() => {
        return data.map(item => ({
            ...item,
            processed: true
        }));
    }, [data]);
    
    return (
        <div>
            {processedData.map(item => (
                <div key={item.id}>{item.name}</div>
            ))}
        </div>
    );
});

// Use useCallback for event handlers in child components
function Parent() {
    const [count, setCount] = useState(0);
    const [name, setName] = useState('');
    
    const handleIncrement = useCallback(() => {
        setCount(prev => prev + 1);
    }, []);
    
    return (
        <div>
            <input 
                value={name} 
                onChange={(e) => setName(e.target.value)} 
            />
            <ChildComponent onIncrement={handleIncrement} />
            <p>Count: {count}</p>
        </div>
    );
}
```

---

## 📚 Quick Reference

### React Hooks Summary

| Hook | Purpose | Example |
|------|---------|---------|
| `useState` | Manage component state | `const [count, setCount] = useState(0)` |
| `useEffect` | Side effects | `useEffect(() => { /* effect */ }, [deps])` |
| `useMemo` | Memoize expensive calculations | `const value = useMemo(() => calc(), [deps])` |
| `useCallback` | Memoize functions | `const fn = useCallback(() => {}, [deps])` |

### JSX Best Practices:
1. Always use keys in lists
2. Keep components small and focused
3. Use meaningful component names
4. Prefer function components over class components
5. Use PropTypes or TypeScript for type checking

### Next Steps:
1. Learn React Hooks in depth
2. Explore React Router for navigation
3. Study state management (Context API, Redux)
4. Learn testing with React Testing Library
5. Build a complete project

This basic React guide provides the foundation for building interactive user interfaces. Practice building small projects to solidify these concepts!
