
# 🚀 Advanced JavaScript

## Table of Contents
- [ES6+ Features](#es6-features)
- [Asynchronous Programming](#asynchronous-programming)
- [Object-Oriented Programming](#object-oriented-programming)
- [Functional Programming](#functional-programming)
- [Design Patterns](#design-patterns)
- [Performance Optimization](#performance-optimization)
- [Modern APIs](#modern-apis)
- [Error Handling](#error-handling)

---

## 🔧 ES6+ Features

### Destructuring Assignment

```javascript
// Array destructuring
const [first, second, ...rest] = [1, 2, 3, 4, 5];
console.log(first); // 1
console.log(rest);  // [3, 4, 5]

// Object destructuring
const user = { name: 'Alice', age: 30, email: 'alice@example.com' };
const { name, age, email: userEmail } = user;
console.log(name);      // 'Alice'
console.log(userEmail); // 'alice@example.com'

// Nested destructuring
const response = {
    data: {
        users: [
            { id: 1, name: 'John' },
            { id: 2, name: 'Jane' }
        ]
    }
};
const { data: { users: [firstUser] } } = response;
console.log(firstUser); // { id: 1, name: 'John' }
```

### Advanced Arrow Functions

```javascript
// Implicit return with objects
const createUser = (name, age) => ({ name, age, active: true });

// Async arrow functions
const fetchUserData = async (userId) => {
    const response = await fetch(`/api/users/${userId}`);
    return await response.json();
};

// Arrow functions in array methods
const numbers = [1, 2, 3, 4, 5];
const processedNumbers = numbers
    .filter(n => n % 2 === 0)
    .map(n => n * 2)
    .reduce((sum, n) => sum + n, 0);
```

### Template Literals and Tagged Templates

```javascript
// Advanced template literals
const createHTML = (title, content) => `
    <article>
        <h1>${title}</h1>
        <div class="content">
            ${content}
        </div>
        <time>${new Date().toISOString()}</time>
    </article>
`;

// Tagged template literals
function highlight(strings, ...values) {
    return strings.reduce((result, string, i) => {
        const value = values[i] ? `<mark>${values[i]}</mark>` : '';
        return result + string + value;
    }, '');
}

const searchTerm = 'JavaScript';
const message = highlight`Learn ${searchTerm} programming with advanced techniques`;
```

---

## ⚡ Asynchronous Programming

### Promises and Async/Await

```javascript
// Promise-based API wrapper
class APIClient {
    constructor(baseURL) {
        this.baseURL = baseURL;
    }
    
    async request(endpoint, options = {}) {
        try {
            const response = await fetch(`${this.baseURL}${endpoint}`, {
                headers: {
                    'Content-Type': 'application/json',
                    ...options.headers
                },
                ...options
            });
            
            if (!response.ok) {
                throw new Error(`HTTP error! status: ${response.status}`);
            }
            
            return await response.json();
        } catch (error) {
            console.error('API request failed:', error);
            throw error;
        }
    }
    
    async get(endpoint) {
        return this.request(endpoint);
    }
    
    async post(endpoint, data) {
        return this.request(endpoint, {
            method: 'POST',
            body: JSON.stringify(data)
        });
    }
}

// Usage
const api = new APIClient('https://api.example.com');

async function loadUserProfile(userId) {
    try {
        const user = await api.get(`/users/${userId}`);
        const posts = await api.get(`/users/${userId}/posts`);
        
        return { user, posts };
    } catch (error) {
        console.error('Failed to load user profile:', error);
        return null;
    }
}
```

### Promise Concurrency Patterns

```javascript
// Promise.all for parallel execution
async function loadDashboardData() {
    try {
        const [users, posts, analytics] = await Promise.all([
            fetch('/api/users').then(r => r.json()),
            fetch('/api/posts').then(r => r.json()),
            fetch('/api/analytics').then(r => r.json())
        ]);
        
        return { users, posts, analytics };
    } catch (error) {
        console.error('Failed to load dashboard data:', error);
    }
}

// Promise.allSettled for handling partial failures
async function loadOptionalData() {
    const results = await Promise.allSettled([
        fetch('/api/required-data').then(r => r.json()),
        fetch('/api/optional-data').then(r => r.json()),
        fetch('/api/fallback-data').then(r => r.json())
    ]);
    
    const processedResults = results.map((result, index) => {
        if (result.status === 'fulfilled') {
            return result.value;
        } else {
            console.warn(`Request ${index} failed:`, result.reason);
            return null;
        }
    });
    
    return processedResults;
}
```

---

## 🏗️ Object-Oriented Programming

### Advanced Classes

```javascript
// Abstract base class pattern
class Shape {
    constructor(color) {
        if (this.constructor === Shape) {
            throw new Error('Cannot instantiate abstract class');
        }
        this.color = color;
    }
    
    // Abstract method
    getArea() {
        throw new Error('Must implement getArea method');
    }
    
    // Concrete method
    describe() {
        return `A ${this.color} shape with area ${this.getArea()}`;
    }
}

class Rectangle extends Shape {
    constructor(width, height, color) {
        super(color);
        this.width = width;
        this.height = height;
    }
    
    getArea() {
        return this.width * this.height;
    }
    
    // Static method
    static createSquare(side, color) {
        return new Rectangle(side, side, color);
    }
    
    // Getter and setter
    get perimeter() {
        return 2 * (this.width + this.height);
    }
    
    set dimensions({ width, height }) {
        this.width = width;
        this.height = height;
    }
}

// Mixin pattern
const Drawable = {
    draw() {
        console.log(`Drawing ${this.constructor.name}`);
    },
    
    animate(duration) {
        console.log(`Animating for ${duration}ms`);
    }
};

// Apply mixin
Object.assign(Rectangle.prototype, Drawable);
```

### Proxy and Reflection

```javascript
// Observable object pattern using Proxy
function createObservable(target, observers = []) {
    return new Proxy(target, {
        set(obj, prop, value) {
            const oldValue = obj[prop];
            obj[prop] = value;
            
            // Notify observers
            observers.forEach(observer => {
                observer({ prop, oldValue, newValue: value });
            });
            
            return true;
        },
        
        get(obj, prop) {
            if (typeof obj[prop] === 'function') {
                return obj[prop].bind(obj);
            }
            return obj[prop];
        }
    });
}

// Usage
const user = createObservable(
    { name: 'John', age: 30 },
    [
        ({ prop, oldValue, newValue }) => {
            console.log(`${prop} changed from ${oldValue} to ${newValue}`);
        }
    ]
);

user.name = 'Jane'; // Logs: "name changed from John to Jane"
```

---

## 📊 Functional Programming

### Higher-Order Functions

```javascript
// Function composition
const compose = (...fns) => (value) => fns.reduceRight((acc, fn) => fn(acc), value);

const pipe = (...fns) => (value) => fns.reduce((acc, fn) => fn(acc), value);

// Example functions
const addTax = (rate) => (price) => price * (1 + rate);
const applyDiscount = (discount) => (price) => price * (1 - discount);
const formatCurrency = (price) => `$${price.toFixed(2)}`;

// Compose pricing pipeline
const calculatePrice = pipe(
    addTax(0.1),
    applyDiscount(0.15),
    formatCurrency
);

console.log(calculatePrice(100)); // "$93.50"
```

### Currying and Partial Application

```javascript
// Currying utility
const curry = (fn) => {
    return function curried(...args) {
        if (args.length >= fn.length) {
            return fn.apply(this, args);
        }
        return (...nextArgs) => curried.apply(this, [...args, ...nextArgs]);
    };
};

// Example: curried validation functions
const validate = curry((rule, message, value) => {
    return rule(value) ? { valid: true } : { valid: false, message };
});

const isRequired = (value) => value != null && value !== '';
const minLength = (min) => (value) => value.length >= min;
const isEmail = (value) => /^[^\s@]+@[^\s@]+\.[^\s@]+$/.test(value);

// Create specific validators
const validateRequired = validate(isRequired, 'Field is required');
const validateMinLength = validate(minLength(3), 'Minimum 3 characters');
const validateEmail = validate(isEmail, 'Invalid email format');

// Usage
const emailValidation = pipe(
    validateRequired,
    (result) => result.valid ? validateEmail(result.value || arguments[0]) : result
);
```

---

## 🎨 Design Patterns

### Module Pattern

```javascript
// Module with private variables and methods
const UserModule = (function() {
    // Private variables
    let users = [];
    let currentId = 0;
    
    // Private methods
    function generateId() {
        return ++currentId;
    }
    
    function validateUser(user) {
        return user.name && user.email;
    }
    
    // Public API
    return {
        add(userData) {
            if (!validateUser(userData)) {
                throw new Error('Invalid user data');
            }
            
            const user = {
                id: generateId(),
                ...userData,
                createdAt: new Date()
            };
            
            users.push(user);
            return user;
        },
        
        getById(id) {
            return users.find(user => user.id === id);
        },
        
        getAll() {
            return [...users]; // Return copy to prevent mutation
        },
        
        remove(id) {
            const index = users.findIndex(user => user.id === id);
            if (index !== -1) {
                return users.splice(index, 1)[0];
            }
            return null;
        }
    };
})();
```

### Observer Pattern

```javascript
class EventEmitter {
    constructor() {
        this.events = {};
    }
    
    on(event, callback) {
        if (!this.events[event]) {
            this.events[event] = [];
        }
        this.events[event].push(callback);
        
        // Return unsubscribe function
        return () => this.off(event, callback);
    }
    
    off(event, callback) {
        if (!this.events[event]) return;
        
        this.events[event] = this.events[event].filter(cb => cb !== callback);
    }
    
    emit(event, ...args) {
        if (!this.events[event]) return;
        
        this.events[event].forEach(callback => {
            try {
                callback(...args);
            } catch (error) {
                console.error(`Error in event handler for ${event}:`, error);
            }
        });
    }
    
    once(event, callback) {
        const onceWrapper = (...args) => {
            callback(...args);
            this.off(event, onceWrapper);
        };
        
        this.on(event, onceWrapper);
    }
}

// Usage example
class ShoppingCart extends EventEmitter {
    constructor() {
        super();
        this.items = [];
    }
    
    addItem(item) {
        this.items.push(item);
        this.emit('itemAdded', item, this.items.length);
    }
    
    removeItem(itemId) {
        const index = this.items.findIndex(item => item.id === itemId);
        if (index !== -1) {
            const removedItem = this.items.splice(index, 1)[0];
            this.emit('itemRemoved', removedItem, this.items.length);
        }
    }
}

const cart = new ShoppingCart();
cart.on('itemAdded', (item, count) => {
    console.log(`Added ${item.name}. Total items: ${count}`);
});
```

---

## ⚡ Performance Optimization

### Debouncing and Throttling

```javascript
// Debounce utility
function debounce(func, delay) {
    let timeoutId;
    
    return function debounced(...args) {
        clearTimeout(timeoutId);
        timeoutId = setTimeout(() => func.apply(this, args), delay);
    };
}

// Throttle utility
function throttle(func, limit) {
    let inThrottle;
    
    return function throttled(...args) {
        if (!inThrottle) {
            func.apply(this, args);
            inThrottle = true;
            setTimeout(() => inThrottle = false, limit);
        }
    };
}

// Usage examples
const searchInput = document.getElementById('search');
const searchHandler = debounce(async (query) => {
    const results = await fetch(`/api/search?q=${query}`);
    // Update UI with results
}, 300);

searchInput.addEventListener('input', (e) => {
    searchHandler(e.target.value);
});

// Throttled scroll handler
const scrollHandler = throttle(() => {
    const scrollPercentage = (window.scrollY / (document.body.scrollHeight - window.innerHeight)) * 100;
    console.log(`Scrolled: ${scrollPercentage.toFixed(2)}%`);
}, 100);

window.addEventListener('scroll', scrollHandler);
```

### Memoization

```javascript
// Memoization decorator
function memoize(fn, getKey = (...args) => JSON.stringify(args)) {
    const cache = new Map();
    
    return function memoized(...args) {
        const key = getKey(...args);
        
        if (cache.has(key)) {
            return cache.get(key);
        }
        
        const result = fn.apply(this, args);
        cache.set(key, result);
        return result;
    };
}

// Example: expensive calculation
const fibonacci = memoize((n) => {
    if (n < 2) return n;
    return fibonacci(n - 1) + fibonacci(n - 2);
});

// API call memoization with TTL
function memoizeWithTTL(fn, ttl = 60000) { // 1 minute default
    const cache = new Map();
    
    return function(...args) {
        const key = JSON.stringify(args);
        const now = Date.now();
        
        if (cache.has(key)) {
            const { value, timestamp } = cache.get(key);
            if (now - timestamp < ttl) {
                return value;
            }
            cache.delete(key);
        }
        
        const result = fn.apply(this, args);
        cache.set(key, { value: result, timestamp: now });
        return result;
    };
}
```

---

## 🌐 Modern APIs

### Intersection Observer

```javascript
// Lazy loading images
class LazyImageLoader {
    constructor() {
        this.imageObserver = new IntersectionObserver((entries) => {
            entries.forEach(entry => {
                if (entry.isIntersecting) {
                    this.loadImage(entry.target);
                    this.imageObserver.unobserve(entry.target);
                }
            });
        }, {
            threshold: 0.1,
            rootMargin: '50px'
        });
        
        this.init();
    }
    
    init() {
        const lazyImages = document.querySelectorAll('img[data-src]');
        lazyImages.forEach(img => this.imageObserver.observe(img));
    }
    
    loadImage(img) {
        img.src = img.dataset.src;
        img.classList.add('loaded');
        
        img.onload = () => {
            img.classList.add('fade-in');
        };
    }
}

// Infinite scrolling
class InfiniteScroll {
    constructor(container, loadMore) {
        this.container = container;
        this.loadMore = loadMore;
        this.loading = false;
        
        this.observer = new IntersectionObserver((entries) => {
            entries.forEach(entry => {
                if (entry.isIntersecting && !this.loading) {
                    this.handleLoadMore();
                }
            });
        });
        
        this.createSentinel();
    }
    
    createSentinel() {
        this.sentinel = document.createElement('div');
        this.sentinel.className = 'scroll-sentinel';
        this.container.appendChild(this.sentinel);
        this.observer.observe(this.sentinel);
    }
    
    async handleLoadMore() {
        this.loading = true;
        
        try {
            const hasMore = await this.loadMore();
            if (!hasMore) {
                this.observer.unobserve(this.sentinel);
                this.sentinel.remove();
            }
        } catch (error) {
            console.error('Failed to load more content:', error);
        } finally {
            this.loading = false;
        }
    }
}
```

### Web Workers

```javascript
// Main thread
class BackgroundProcessor {
    constructor() {
        this.worker = new Worker('data-processor.js');
        this.tasks = new Map();
        this.taskId = 0;
        
        this.worker.onmessage = (event) => {
            const { taskId, result, error } = event.data;
            const task = this.tasks.get(taskId);
            
            if (task) {
                if (error) {
                    task.reject(new Error(error));
                } else {
                    task.resolve(result);
                }
                this.tasks.delete(taskId);
            }
        };
    }
    
    async processData(data, operation) {
        return new Promise((resolve, reject) => {
            const taskId = ++this.taskId;
            this.tasks.set(taskId, { resolve, reject });
            
            this.worker.postMessage({
                taskId,
                data,
                operation
            });
        });
    }
    
    terminate() {
        this.worker.terminate();
    }
}

// Worker file (data-processor.js)
self.onmessage = function(event) {
    const { taskId, data, operation } = event.data;
    
    try {
        let result;
        
        switch (operation) {
            case 'sort':
                result = data.sort((a, b) => a - b);
                break;
            case 'filter':
                result = data.filter(item => item > 0);
                break;
            case 'map':
                result = data.map(item => item * 2);
                break;
            default:
                throw new Error(`Unknown operation: ${operation}`);
        }
        
        self.postMessage({ taskId, result });
    } catch (error) {
        self.postMessage({ taskId, error: error.message });
    }
};
```

---

## 🛠️ Error Handling

### Advanced Error Classes

```javascript
// Custom error classes
class APIError extends Error {
    constructor(message, status, code) {
        super(message);
        this.name = 'APIError';
        this.status = status;
        this.code = code;
    }
}

class ValidationError extends Error {
    constructor(field, message) {
        super(`Validation failed for ${field}: ${message}`);
        this.name = 'ValidationError';
        this.field = field;
    }
}

// Global error handler
class ErrorHandler {
    constructor() {
        this.handlers = new Map();
        this.setupGlobalHandlers();
    }
    
    setupGlobalHandlers() {
        // Unhandled promise rejections
        window.addEventListener('unhandledrejection', (event) => {
            console.error('Unhandled promise rejection:', event.reason);
            this.handleError(event.reason);
            event.preventDefault();
        });
        
        // Global error handler
        window.addEventListener('error', (event) => {
            console.error('Global error:', event.error);
            this.handleError(event.error);
        });
    }
    
    register(errorType, handler) {
        this.handlers.set(errorType, handler);
    }
    
    handleError(error) {
        const handler = this.handlers.get(error.constructor);
        
        if (handler) {
            handler(error);
        } else {
            this.defaultHandler(error);
        }
    }
    
    defaultHandler(error) {
        // Log to external service
        console.error('Unhandled error:', error);
        
        // Show user-friendly message
        this.showErrorNotification('Something went wrong. Please try again.');
    }
    
    showErrorNotification(message) {
        // Create and show error notification
        const notification = document.createElement('div');
        notification.className = 'error-notification';
        notification.textContent = message;
        document.body.appendChild(notification);
        
        setTimeout(() => {
            notification.remove();
        }, 5000);
    }
}

// Setup error handling
const errorHandler = new ErrorHandler();

errorHandler.register(APIError, (error) => {
    console.error(`API Error [${error.status}]:`, error.message);
    errorHandler.showErrorNotification('Server error. Please try again later.');
});

errorHandler.register(ValidationError, (error) => {
    console.error(`Validation Error:`, error.message);
    errorHandler.showErrorNotification(`Please check the ${error.field} field.`);
});
```

---

## 📚 Quick Reference

### Advanced Array Methods

```javascript
// Advanced array operations
const data = [
    { id: 1, name: 'Alice', age: 30, department: 'Engineering' },
    { id: 2, name: 'Bob', age: 25, department: 'Marketing' },
    { id: 3, name: 'Charlie', age: 35, department: 'Engineering' }
];

// Group by department
const groupBy = (array, key) => {
    return array.reduce((groups, item) => {
        const group = item[key];
        groups[group] = groups[group] || [];
        groups[group].push(item);
        return groups;
    }, {});
};

const grouped = groupBy(data, 'department');

// Find with complex conditions
const findUser = data.find(user => user.age > 25 && user.department === 'Engineering');

// Sorting with multiple criteria
const sorted = data.sort((a, b) => {
    if (a.department !== b.department) {
        return a.department.localeCompare(b.department);
    }
    return b.age - a.age;
});
```

### Modern JavaScript Features

| Feature | Example | Description |
|---------|---------|-------------|
| Optional Chaining | `user?.profile?.email` | Safe property access |
| Nullish Coalescing | `name ?? 'Default'` | Default for null/undefined |
| BigInt | `BigInt(123456789012345678901234567890)` | Arbitrary precision integers |
| Private Fields | `class User { #password }` | Private class members |
| Top-level await | `const data = await fetch('/api')` | Await in modules |

### Next Steps:
1. Practice with real-world projects
2. Learn modern frameworks (React, Vue, Angular)
3. Explore Node.js for backend development
4. Master TypeScript for type safety
5. Learn testing frameworks (Jest, Cypress)

This comprehensive guide covers advanced JavaScript concepts that will help you become a proficient developer. Keep practicing and building projects to master these techniques!
