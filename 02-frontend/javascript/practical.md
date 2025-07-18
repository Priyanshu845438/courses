
# 🚀 Intermediate JavaScript

## Table of Contents
- [Advanced Functions](#advanced-functions)
- [Object-Oriented Programming](#object-oriented-programming)
- [Asynchronous JavaScript](#asynchronous-javascript)
- [Error Handling](#error-handling)
- [Modules and Imports](#modules-and-imports)
- [Advanced Array Methods](#advanced-array-methods)
- [Regular Expressions](#regular-expressions)
- [Browser APIs](#browser-apis)
- [Performance Optimization](#performance-optimization)
- [Testing Fundamentals](#testing-fundamentals)

---

## 🔧 Advanced Functions

### Closures

```javascript
// Basic closure
function outerFunction(x) {
    // This is the outer function's scope
    
    function innerFunction(y) {
        // This is the inner function's scope
        return x + y; // Access to outer function's variable
    }
    
    return innerFunction;
}

const addFive = outerFunction(5);
console.log(addFive(3)); // 8

// Practical closure example - Counter
function createCounter() {
    let count = 0;
    
    return {
        increment: () => ++count,
        decrement: () => --count,
        getValue: () => count,
        reset: () => { count = 0; }
    };
}

const counter = createCounter();
console.log(counter.increment()); // 1
console.log(counter.increment()); // 2
console.log(counter.getValue());  // 2
counter.reset();
console.log(counter.getValue());  // 0
```

### Function Currying

```javascript
// Basic currying
function curry(func) {
    return function curried(...args) {
        if (args.length >= func.length) {
            return func.apply(this, args);
        } else {
            return function(...nextArgs) {
                return curried(...args, ...nextArgs);
            };
        }
    };
}

// Example usage
function multiply(a, b, c) {
    return a * b * c;
}

const curriedMultiply = curry(multiply);
console.log(curriedMultiply(2)(3)(4)); // 24
console.log(curriedMultiply(2, 3)(4)); // 24
console.log(curriedMultiply(2)(3, 4)); // 24

// Practical currying example
const log = curry((level, message, timestamp) => {
    console.log(`[${timestamp}] ${level}: ${message}`);
});

const logError = log('ERROR');
const logInfo = log('INFO');

logError('Database connection failed', new Date().toISOString());
logInfo('User logged in', new Date().toISOString());
```

### Higher-Order Functions

```javascript
// Function composition
const compose = (...fns) => (value) => fns.reduceRight((acc, fn) => fn(acc), value);

const addOne = x => x + 1;
const multiplyByTwo = x => x * 2;
const square = x => x * x;

const composedFunction = compose(square, multiplyByTwo, addOne);
console.log(composedFunction(3)); // (3 + 1) * 2 = 8, 8^2 = 64

// Pipe function (left to right)
const pipe = (...fns) => (value) => fns.reduce((acc, fn) => fn(acc), value);

const pipedFunction = pipe(addOne, multiplyByTwo, square);
console.log(pipedFunction(3)); // Same result: 64

// Memoization
function memoize(fn) {
    const cache = new Map();
    
    return function(...args) {
        const key = JSON.stringify(args);
        
        if (cache.has(key)) {
            console.log('Cache hit!');
            return cache.get(key);
        }
        
        const result = fn.apply(this, args);
        cache.set(key, result);
        return result;
    };
}

// Expensive function example
const fibonacci = memoize(function(n) {
    if (n <= 1) return n;
    return fibonacci(n - 1) + fibonacci(n - 2);
});

console.log(fibonacci(40)); // Much faster with memoization
```

---

## 🏗️ Object-Oriented Programming

### Classes and Inheritance

```javascript
// Base class
class Animal {
    constructor(name, species) {
        this.name = name;
        this.species = species;
        this._energy = 100; // Private-like property
    }
    
    // Method
    speak() {
        console.log(`${this.name} makes a sound`);
    }
    
    // Getter
    get energy() {
        return this._energy;
    }
    
    // Setter
    set energy(value) {
        if (value < 0) value = 0;
        if (value > 100) value = 100;
        this._energy = value;
    }
    
    // Static method
    static getSpeciesCount() {
        return Animal.count || 0;
    }
    
    // Private method (ES2022)
    #calculateAge(birthYear) {
        return new Date().getFullYear() - birthYear;
    }
}

// Inheritance
class Dog extends Animal {
    constructor(name, breed) {
        super(name, 'Canine'); // Call parent constructor
        this.breed = breed;
        this.tricks = [];
    }
    
    // Override parent method
    speak() {
        console.log(`${this.name} barks: Woof! Woof!`);
    }
    
    // New method
    learnTrick(trick) {
        this.tricks.push(trick);
        console.log(`${this.name} learned ${trick}!`);
    }
    
    performTrick() {
        if (this.tricks.length === 0) {
            console.log(`${this.name} doesn't know any tricks yet`);
            return;
        }
        
        const trick = this.tricks[Math.floor(Math.random() * this.tricks.length)];
        console.log(`${this.name} performs ${trick}!`);
        this.energy -= 10;
    }
}

// Usage
const myDog = new Dog('Buddy', 'Golden Retriever');
myDog.speak(); // Buddy barks: Woof! Woof!
myDog.learnTrick('sit');
myDog.learnTrick('roll over');
myDog.performTrick(); // Buddy performs sit!
console.log(myDog.energy); // 90
```

### Prototypes and Prototype Chain

```javascript
// Constructor function approach
function Person(name, age) {
    this.name = name;
    this.age = age;
}

// Adding methods to prototype
Person.prototype.greet = function() {
    return `Hello, I'm ${this.name} and I'm ${this.age} years old`;
};

Person.prototype.haveBirthday = function() {
    this.age++;
    console.log(`Happy birthday! ${this.name} is now ${this.age}`);
};

// Create instances
const john = new Person('John', 30);
const jane = new Person('Jane', 25);

console.log(john.greet()); // Hello, I'm John and I'm 30 years old
jane.haveBirthday(); // Happy birthday! Jane is now 26

// Prototype chain exploration
console.log(john.__proto__ === Person.prototype); // true
console.log(Person.prototype.__proto__ === Object.prototype); // true
console.log(Object.prototype.__proto__); // null

// Adding methods to built-in prototypes (use carefully!)
Array.prototype.last = function() {
    return this[this.length - 1];
};

const numbers = [1, 2, 3, 4, 5];
console.log(numbers.last()); // 5
```

### Design Patterns

```javascript
// Singleton Pattern
class Database {
    constructor() {
        if (Database.instance) {
            return Database.instance;
        }
        
        this.connection = 'Connected to database';
        Database.instance = this;
    }
    
    query(sql) {
        console.log(`Executing: ${sql}`);
    }
}

const db1 = new Database();
const db2 = new Database();
console.log(db1 === db2); // true

// Factory Pattern
class VehicleFactory {
    static createVehicle(type, options) {
        switch (type) {
            case 'car':
                return new Car(options);
            case 'motorcycle':
                return new Motorcycle(options);
            case 'truck':
                return new Truck(options);
            default:
                throw new Error('Unknown vehicle type');
        }
    }
}

class Car {
    constructor({ make, model, year }) {
        this.type = 'car';
        this.make = make;
        this.model = model;
        this.year = year;
        this.wheels = 4;
    }
}

class Motorcycle {
    constructor({ make, model, year }) {
        this.type = 'motorcycle';
        this.make = make;
        this.model = model;
        this.year = year;
        this.wheels = 2;
    }
}

// Observer Pattern
class EventEmitter {
    constructor() {
        this.events = {};
    }
    
    on(event, listener) {
        if (!this.events[event]) {
            this.events[event] = [];
        }
        this.events[event].push(listener);
    }
    
    off(event, listenerToRemove) {
        if (!this.events[event]) return;
        
        this.events[event] = this.events[event].filter(
            listener => listener !== listenerToRemove
        );
    }
    
    emit(event, data) {
        if (!this.events[event]) return;
        
        this.events[event].forEach(listener => listener(data));
    }
}

// Usage
const emitter = new EventEmitter();

const userLoginHandler = (user) => console.log(`${user.name} logged in`);
const analyticsHandler = (user) => console.log(`Track login for ${user.id}`);

emitter.on('userLogin', userLoginHandler);
emitter.on('userLogin', analyticsHandler);

emitter.emit('userLogin', { id: 1, name: 'John' });
```

---

## ⏰ Asynchronous JavaScript

### Promises

```javascript
// Creating a Promise
function fetchUserData(userId) {
    return new Promise((resolve, reject) => {
        // Simulate API call
        setTimeout(() => {
            if (userId > 0) {
                resolve({
                    id: userId,
                    name: 'John Doe',
                    email: 'john@example.com'
                });
            } else {
                reject(new Error('Invalid user ID'));
            }
        }, 1000);
    });
}

// Using Promises
fetchUserData(1)
    .then(user => {
        console.log('User data:', user);
        return fetchUserData(2); // Chain another promise
    })
    .then(user2 => {
        console.log('Second user:', user2);
    })
    .catch(error => {
        console.error('Error:', error.message);
    })
    .finally(() => {
        console.log('Request completed');
    });

// Promise.all - Wait for all promises
const promise1 = fetchUserData(1);
const promise2 = fetchUserData(2);
const promise3 = fetchUserData(3);

Promise.all([promise1, promise2, promise3])
    .then(users => {
        console.log('All users:', users);
    })
    .catch(error => {
        console.error('One or more requests failed:', error);
    });

// Promise.race - First to complete wins
Promise.race([promise1, promise2, promise3])
    .then(firstUser => {
        console.log('First user received:', firstUser);
    });

// Promise.allSettled - Wait for all, regardless of outcome
Promise.allSettled([promise1, promise2, promise3])
    .then(results => {
        results.forEach((result, index) => {
            if (result.status === 'fulfilled') {
                console.log(`User ${index + 1}:`, result.value);
            } else {
                console.log(`User ${index + 1} failed:`, result.reason);
            }
        });
    });
```

### Async/Await

```javascript
// Basic async/await
async function getUserData(userId) {
    try {
        const user = await fetchUserData(userId);
        console.log('User:', user);
        return user;
    } catch (error) {
        console.error('Error fetching user:', error.message);
        throw error;
    }
}

// Sequential execution
async function getMultipleUsersSequential() {
    try {
        const user1 = await fetchUserData(1);
        const user2 = await fetchUserData(2);
        const user3 = await fetchUserData(3);
        
        return [user1, user2, user3];
    } catch (error) {
        console.error('Error:', error.message);
    }
}

// Parallel execution
async function getMultipleUsersParallel() {
    try {
        const users = await Promise.all([
            fetchUserData(1),
            fetchUserData(2),
            fetchUserData(3)
        ]);
        
        return users;
    } catch (error) {
        console.error('Error:', error.message);
    }
}

// Real-world example: API with retry logic
async function fetchWithRetry(url, maxRetries = 3) {
    let lastError;
    
    for (let i = 0; i < maxRetries; i++) {
        try {
            const response = await fetch(url);
            
            if (!response.ok) {
                throw new Error(`HTTP error! status: ${response.status}`);
            }
            
            return await response.json();
        } catch (error) {
            lastError = error;
            console.log(`Attempt ${i + 1} failed:`, error.message);
            
            if (i < maxRetries - 1) {
                // Wait before retry (exponential backoff)
                await new Promise(resolve => setTimeout(resolve, 1000 * Math.pow(2, i)));
            }
        }
    }
    
    throw lastError;
}

// Usage
async function loadData() {
    try {
        const data = await fetchWithRetry('https://api.example.com/data');
        console.log('Data loaded:', data);
    } catch (error) {
        console.error('Failed to load data after retries:', error.message);
    }
}
```

### Generators and Iterators

```javascript
// Basic generator
function* numberGenerator() {
    yield 1;
    yield 2;
    yield 3;
    return 'done';
}

const gen = numberGenerator();
console.log(gen.next()); // { value: 1, done: false }
console.log(gen.next()); // { value: 2, done: false }
console.log(gen.next()); // { value: 3, done: false }
console.log(gen.next()); // { value: 'done', done: true }

// Infinite generator
function* fibonacciGenerator() {
    let a = 0, b = 1;
    while (true) {
        yield a;
        [a, b] = [b, a + b];
    }
}

const fib = fibonacciGenerator();
for (let i = 0; i < 10; i++) {
    console.log(fib.next().value);
}

// Async generator
async function* asyncDataGenerator() {
    for (let i = 1; i <= 5; i++) {
        await new Promise(resolve => setTimeout(resolve, 1000));
        yield `Data chunk ${i}`;
    }
}

async function processAsyncData() {
    for await (const chunk of asyncDataGenerator()) {
        console.log('Received:', chunk);
    }
}

// Custom iterator
class Range {
    constructor(start, end) {
        this.start = start;
        this.end = end;
    }
    
    *[Symbol.iterator]() {
        for (let i = this.start; i <= this.end; i++) {
            yield i;
        }
    }
}

const range = new Range(1, 5);
for (const num of range) {
    console.log(num); // 1, 2, 3, 4, 5
}
```

---

## 🚨 Error Handling

### Try-Catch-Finally

```javascript
// Basic error handling
function divideNumbers(a, b) {
    try {
        if (typeof a !== 'number' || typeof b !== 'number') {
            throw new TypeError('Both arguments must be numbers');
        }
        
        if (b === 0) {
            throw new Error('Division by zero is not allowed');
        }
        
        return a / b;
    } catch (error) {
        console.error('Error in divideNumbers:', error.message);
        return null;
    } finally {
        console.log('Division operation completed');
    }
}

console.log(divideNumbers(10, 2)); // 5
console.log(divideNumbers(10, 0)); // null
console.log(divideNumbers('10', 2)); // null

// Custom error classes
class ValidationError extends Error {
    constructor(message) {
        super(message);
        this.name = 'ValidationError';
    }
}

class NetworkError extends Error {
    constructor(message, statusCode) {
        super(message);
        this.name = 'NetworkError';
        this.statusCode = statusCode;
    }
}

// Using custom errors
function validateUser(user) {
    if (!user.name) {
        throw new ValidationError('Name is required');
    }
    
    if (!user.email || !user.email.includes('@')) {
        throw new ValidationError('Valid email is required');
    }
    
    if (user.age < 0 || user.age > 150) {
        throw new ValidationError('Age must be between 0 and 150');
    }
}

async function fetchUserFromAPI(userId) {
    try {
        const response = await fetch(`/api/users/${userId}`);
        
        if (!response.ok) {
            throw new NetworkError(
                `Failed to fetch user: ${response.statusText}`,
                response.status
            );
        }
        
        const user = await response.json();
        validateUser(user);
        
        return user;
    } catch (error) {
        if (error instanceof ValidationError) {
            console.error('Validation error:', error.message);
        } else if (error instanceof NetworkError) {
            console.error(`Network error (${error.statusCode}):`, error.message);
        } else {
            console.error('Unexpected error:', error.message);
        }
        
        throw error; // Re-throw for caller to handle
    }
}
```

### Error Boundaries (Concept)

```javascript
// Global error handler
window.addEventListener('error', (event) => {
    console.error('Global error:', event.error);
    // Send error to logging service
    logError(event.error);
});

window.addEventListener('unhandledrejection', (event) => {
    console.error('Unhandled promise rejection:', event.reason);
    // Prevent default browser behavior
    event.preventDefault();
    // Log the error
    logError(event.reason);
});

// Error logging utility
class ErrorLogger {
    static log(error, context = {}) {
        const errorReport = {
            message: error.message,
            stack: error.stack,
            timestamp: new Date().toISOString(),
            userAgent: navigator.userAgent,
            url: window.location.href,
            context
        };
        
        // In a real app, send to error tracking service
        console.log('Error Report:', errorReport);
        
        // Could send to services like Sentry, LogRocket, etc.
        // this.sendToErrorService(errorReport);
    }
}

function logError(error, context) {
    ErrorLogger.log(error, context);
}
```

---

## 📦 Modules and Imports

### ES6 Modules

```javascript
// math.js - Named exports
export function add(a, b) {
    return a + b;
}

export function subtract(a, b) {
    return a - b;
}

export const PI = 3.14159;

export class Calculator {
    static multiply(a, b) {
        return a * b;
    }
    
    static divide(a, b) {
        if (b === 0) throw new Error('Division by zero');
        return a / b;
    }
}

// user.js - Default export
export default class User {
    constructor(name, email) {
        this.name = name;
        this.email = email;
    }
    
    getDisplayName() {
        return `${this.name} (${this.email})`;
    }
}

// api.js - Mixed exports
export { fetchUser as getUser } from './userService.js';

export async function fetchPosts() {
    const response = await fetch('/api/posts');
    return response.json();
}

export const API_BASE_URL = 'https://api.example.com';

export default {
    baseURL: API_BASE_URL,
    timeout: 5000
};

// main.js - Importing
import User from './user.js'; // Default import
import { add, subtract, PI, Calculator } from './math.js'; // Named imports
import * as MathUtils from './math.js'; // Namespace import
import apiConfig, { fetchPosts, getUser } from './api.js'; // Mixed imports

// Usage
const user = new User('John Doe', 'john@example.com');
console.log(user.getDisplayName());

console.log(add(5, 3)); // 8
console.log(Calculator.multiply(4, 7)); // 28
console.log(MathUtils.PI); // 3.14159

// Dynamic imports
async function loadModule() {
    try {
        const { default: heavyModule } = await import('./heavyModule.js');
        heavyModule.initialize();
    } catch (error) {
        console.error('Failed to load module:', error);
    }
}

// Conditional imports
if (window.innerWidth > 768) {
    import('./desktopFeatures.js').then(module => {
        module.enableDesktopFeatures();
    });
} else {
    import('./mobileFeatures.js').then(module => {
        module.enableMobileFeatures();
    });
}
```

---

## 🔄 Advanced Array Methods

### Functional Array Programming

```javascript
// Sample data
const products = [
    { id: 1, name: 'Laptop', price: 999, category: 'Electronics', inStock: true },
    { id: 2, name: 'Phone', price: 699, category: 'Electronics', inStock: false },
    { id: 3, name: 'Book', price: 19, category: 'Books', inStock: true },
    { id: 4, name: 'Headphones', price: 199, category: 'Electronics', inStock: true },
    { id: 5, name: 'Tablet', price: 499, category: 'Electronics', inStock: false }
];

// Complex filtering and mapping
const availableElectronics = products
    .filter(product => product.category === 'Electronics')
    .filter(product => product.inStock)
    .map(product => ({
        ...product,
        discountedPrice: product.price * 0.9
    }))
    .sort((a, b) => a.price - b.price);

console.log(availableElectronics);

// Reduce examples
const totalValue = products.reduce((sum, product) => sum + product.price, 0);

const productsByCategory = products.reduce((acc, product) => {
    const category = product.category;
    if (!acc[category]) {
        acc[category] = [];
    }
    acc[category].push(product);
    return acc;
}, {});

const priceStats = products.reduce((stats, product) => {
    stats.total += product.price;
    stats.count++;
    stats.min = Math.min(stats.min, product.price);
    stats.max = Math.max(stats.max, product.price);
    stats.average = stats.total / stats.count;
    return stats;
}, { total: 0, count: 0, min: Infinity, max: -Infinity, average: 0 });

console.log(priceStats);

// Advanced array methods
const numbers = [1, 2, 3, 4, 5, 6, 7, 8, 9, 10];

// Find methods
const firstEven = numbers.find(n => n % 2 === 0); // 2
const firstEvenIndex = numbers.findIndex(n => n % 2 === 0); // 1
const lastEven = numbers.findLast(n => n % 2 === 0); // 10 (ES2022)

// Test methods
const allPositive = numbers.every(n => n > 0); // true
const hasEven = numbers.some(n => n % 2 === 0); // true

// Flat and flatMap
const nested = [[1, 2], [3, 4], [5, 6]];
const flattened = nested.flat(); // [1, 2, 3, 4, 5, 6]

const deepNested = [1, [2, [3, [4]]]];
const deepFlattened = deepNested.flat(3); // [1, 2, 3, 4]

const words = ['hello', 'world'];
const letters = words.flatMap(word => word.split('')); // ['h','e','l','l','o','w','o','r','l','d']

// Array.from examples
const range = Array.from({ length: 5 }, (_, i) => i + 1); // [1, 2, 3, 4, 5]
const doubled = Array.from([1, 2, 3], x => x * 2); // [2, 4, 6]

// Custom array utilities
Array.prototype.chunk = function(size) {
    const chunks = [];
    for (let i = 0; i < this.length; i += size) {
        chunks.push(this.slice(i, i + size));
    }
    return chunks;
};

Array.prototype.unique = function() {
    return [...new Set(this)];
};

Array.prototype.groupBy = function(keyFn) {
    return this.reduce((groups, item) => {
        const key = keyFn(item);
        if (!groups[key]) {
            groups[key] = [];
        }
        groups[key].push(item);
        return groups;
    }, {});
};

// Usage
const nums = [1, 2, 3, 4, 5, 6, 7, 8, 9, 10];
console.log(nums.chunk(3)); // [[1,2,3], [4,5,6], [7,8,9], [10]]

const duplicates = [1, 2, 2, 3, 3, 3, 4];
console.log(duplicates.unique()); // [1, 2, 3, 4]

const grouped = products.groupBy(p => p.category);
console.log(grouped);
```

---

## 🔍 Regular Expressions

### Pattern Matching

```javascript
// Basic regex patterns
const emailRegex = /^[^\s@]+@[^\s@]+\.[^\s@]+$/;
const phoneRegex = /^\(\d{3}\)\s\d{3}-\d{4}$/; // (123) 456-7890
const urlRegex = /^https?:\/\/(www\.)?[-a-zA-Z0-9@:%._\+~#=]{1,256}\.[a-zA-Z0-9()]{1,6}\b([-a-zA-Z0-9()@:%_\+.~#?&//=]*)$/;

// Test patterns
console.log(emailRegex.test('user@example.com')); // true
console.log(phoneRegex.test('(123) 456-7890')); // true
console.log(urlRegex.test('https://www.example.com')); // true

// String methods with regex
const text = 'The quick brown fox jumps over the lazy dog';

// Search
console.log(text.search(/brown/)); // 10
console.log(text.match(/\b\w{4}\b/g)); // ['over', 'lazy']

// Replace
const censored = text.replace(/\b\w{3}\b/g, '***');
console.log(censored); // '*** quick brown *** jumps over *** lazy ***'

// Split
const words = text.split(/\s+/);
console.log(words);

// Advanced regex features
const advancedRegex = /(?<year>\d{4})-(?<month>\d{2})-(?<day>\d{2})/;
const dateString = '2024-03-15';
const match = dateString.match(advancedRegex);

if (match) {
    console.log(match.groups); // { year: '2024', month: '03', day: '15' }
}

// Validation functions
class Validator {
    static email(email) {
        const regex = /^[^\s@]+@[^\s@]+\.[^\s@]+$/;
        return regex.test(email);
    }
    
    static password(password) {
        // At least 8 chars, 1 uppercase, 1 lowercase, 1 number, 1 special char
        const regex = /^(?=.*[a-z])(?=.*[A-Z])(?=.*\d)(?=.*[@$!%*?&])[A-Za-z\d@$!%*?&]{8,}$/;
        return regex.test(password);
    }
    
    static creditCard(cardNumber) {
        // Remove spaces and dashes
        const cleaned = cardNumber.replace(/[\s-]/g, '');
        
        // Check if all digits and 13-19 length
        const regex = /^\d{13,19}$/;
        if (!regex.test(cleaned)) return false;
        
        // Luhn algorithm
        let sum = 0;
        let isEven = false;
        
        for (let i = cleaned.length - 1; i >= 0; i--) {
            let digit = parseInt(cleaned[i]);
            
            if (isEven) {
                digit *= 2;
                if (digit > 9) digit -= 9;
            }
            
            sum += digit;
            isEven = !isEven;
        }
        
        return sum % 10 === 0;
    }
    
    static phone(phone) {
        const patterns = [
            /^\(\d{3}\)\s\d{3}-\d{4}$/, // (123) 456-7890
            /^\d{3}-\d{3}-\d{4}$/, // 123-456-7890
            /^\d{10}$/, // 1234567890
            /^\+1\d{10}$/ // +11234567890
        ];
        
        return patterns.some(pattern => pattern.test(phone));
    }
}

// Text processing utilities
class TextProcessor {
    static extractEmails(text) {
        const emailRegex = /[^\s@]+@[^\s@]+\.[^\s@]+/g;
        return text.match(emailRegex) || [];
    }
    
    static extractUrls(text) {
        const urlRegex = /https?:\/\/(www\.)?[-a-zA-Z0-9@:%._\+~#=]{1,256}\.[a-zA-Z0-9()]{1,6}\b([-a-zA-Z0-9()@:%_\+.~#?&//=]*)/g;
        return text.match(urlRegex) || [];
    }
    
    static extractHashtags(text) {
        const hashtagRegex = /#[a-zA-Z0-9_]+/g;
        return text.match(hashtagRegex) || [];
    }
    
    static extractMentions(text) {
        const mentionRegex = /@[a-zA-Z0-9_]+/g;
        return text.match(mentionRegex) || [];
    }
    
    static cleanHtml(html) {
        return html.replace(/<[^>]*>/g, '');
    }
    
    static slug(text) {
        return text
            .toLowerCase()
            .trim()
            .replace(/[^\w\s-]/g, '')
            .replace(/[\s_-]+/g, '-')
            .replace(/^-+|-+$/g, '');
    }
}

// Usage examples
const sampleText = "Contact us at info@example.com or visit https://www.example.com. Follow us @company #javascript #coding";

console.log(TextProcessor.extractEmails(sampleText));
console.log(TextProcessor.extractUrls(sampleText));
console.log(TextProcessor.extractHashtags(sampleText));
console.log(TextProcessor.extractMentions(sampleText));
console.log(TextProcessor.slug("Hello World! This is a Test."));
```

---

## 🌐 Browser APIs

### Fetch API and HTTP Requests

```javascript
// Basic fetch examples
class ApiClient {
    constructor(baseURL) {
        this.baseURL = baseURL;
        this.defaultHeaders = {
            'Content-Type': 'application/json'
        };
    }
    
    async request(endpoint, options = {}) {
        const url = `${this.baseURL}${endpoint}`;
        const config = {
            headers: { ...this.defaultHeaders, ...options.headers },
            ...options
        };
        
        try {
            const response = await fetch(url, config);
            
            if (!response.ok) {
                throw new Error(`HTTP error! status: ${response.status}`);
            }
            
            const contentType = response.headers.get('content-type');
            if (contentType && contentType.includes('application/json')) {
                return await response.json();
            }
            
            return await response.text();
        } catch (error) {
            console.error('Request failed:', error);
            throw error;
        }
    }
    
    async get(endpoint, params = {}) {
        const queryString = new URLSearchParams(params).toString();
        const url = queryString ? `${endpoint}?${queryString}` : endpoint;
        return this.request(url);
    }
    
    async post(endpoint, data) {
        return this.request(endpoint, {
            method: 'POST',
            body: JSON.stringify(data)
        });
    }
    
    async put(endpoint, data) {
        return this.request(endpoint, {
            method: 'PUT',
            body: JSON.stringify(data)
        });
    }
    
    async delete(endpoint) {
        return this.request(endpoint, {
            method: 'DELETE'
        });
    }
}

// Usage
const api = new ApiClient('https://jsonplaceholder.typicode.com');

async function demonstrateAPI() {
    try {
        // GET request
        const posts = await api.get('/posts', { userId: 1 });
        console.log('Posts:', posts);
        
        // POST request
        const newPost = await api.post('/posts', {
            title: 'New Post',
            body: 'This is a new post',
            userId: 1
        });
        console.log('Created post:', newPost);
        
        // PUT request
        const updatedPost = await api.put('/posts/1', {
            id: 1,
            title: 'Updated Post',
            body: 'This post has been updated',
            userId: 1
        });
        console.log('Updated post:', updatedPost);
        
    } catch (error) {
        console.error('API error:', error);
    }
}
```

### Local Storage and Session Storage

```javascript
// Storage utility class
class StorageManager {
    constructor(storageType = 'localStorage') {
        this.storage = window[storageType];
    }
    
    set(key, value) {
        try {
            const serializedValue = JSON.stringify(value);
            this.storage.setItem(key, serializedValue);
            return true;
        } catch (error) {
            console.error('Error saving to storage:', error);
            return false;
        }
    }
    
    get(key, defaultValue = null) {
        try {
            const item = this.storage.getItem(key);
            return item ? JSON.parse(item) : defaultValue;
        } catch (error) {
            console.error('Error reading from storage:', error);
            return defaultValue;
        }
    }
    
    remove(key) {
        this.storage.removeItem(key);
    }
    
    clear() {
        this.storage.clear();
    }
    
    has(key) {
        return this.storage.getItem(key) !== null;
    }
    
    keys() {
        return Object.keys(this.storage);
    }
    
    size() {
        return this.storage.length;
    }
    
    // Set with expiration
    setWithExpiry(key, value, ttl) {
        const now = new Date();
        const item = {
            value: value,
            expiry: now.getTime() + ttl
        };
        this.set(key, item);
    }
    
    getWithExpiry(key) {
        const item = this.get(key);
        if (!item) return null;
        
        const now = new Date();
        if (now.getTime() > item.expiry) {
            this.remove(key);
            return null;
        }
        
        return item.value;
    }
}

// Usage
const localStorage = new StorageManager('localStorage');
const sessionStorage = new StorageManager('sessionStorage');

// Save user preferences
localStorage.set('userPreferences', {
    theme: 'dark',
    language: 'en',
    notifications: true
});

// Cache data with expiration (1 hour)
localStorage.setWithExpiry('cachedData', { data: 'some data' }, 60 * 60 * 1000);

// Retrieve data
const preferences = localStorage.get('userPreferences');
const cachedData = localStorage.getWithExpiry('cachedData');
```

### URL and URLSearchParams

```javascript
// URL manipulation utilities
class UrlManager {
    constructor(url = window.location.href) {
        this.url = new URL(url);
    }
    
    getParam(key) {
        return this.url.searchParams.get(key);
    }
    
    setParam(key, value) {
        this.url.searchParams.set(key, value);
        return this;
    }
    
    removeParam(key) {
        this.url.searchParams.delete(key);
        return this;
    }
    
    hasParam(key) {
        return this.url.searchParams.has(key);
    }
    
    getAllParams() {
        const params = {};
        for (const [key, value] of this.url.searchParams) {
            params[key] = value;
        }
        return params;
    }
    
    updateUrl(pushState = true) {
        if (pushState) {
            history.pushState({}, '', this.url.toString());
        } else {
            history.replaceState({}, '', this.url.toString());
        }
    }
    
    toString() {
        return this.url.toString();
    }
    
    // Static methods
    static parseQuery(queryString) {
        const params = new URLSearchParams(queryString);
        const result = {};
        for (const [key, value] of params) {
            result[key] = value;
        }
        return result;
    }
    
    static buildQuery(params) {
        return new URLSearchParams(params).toString();
    }
}

// Usage
const urlManager = new UrlManager();
urlManager
    .setParam('page', '2')
    .setParam('sort', 'name')
    .removeParam('debug')
    .updateUrl();

// Router utility
class SimpleRouter {
    constructor() {
        this.routes = new Map();
        this.currentRoute = null;
        
        window.addEventListener('popstate', () => {
            this.handleRoute();
        });
    }
    
    addRoute(path, handler) {
        this.routes.set(path, handler);
    }
    
    navigate(path, pushState = true) {
        if (pushState) {
            history.pushState({}, '', path);
        }
        this.handleRoute();
    }
    
    handleRoute() {
        const path = window.location.pathname;
        
        if (this.routes.has(path)) {
            this.currentRoute = path;
            this.routes.get(path)();
        } else {
            // Handle 404 or default route
            console.log('Route not found:', path);
        }
    }
    
    start() {
        this.handleRoute();
    }
}

// Usage
const router = new SimpleRouter();

router.addRoute('/', () => {
    console.log('Home page');
});

router.addRoute('/about', () => {
    console.log('About page');
});

router.addRoute('/contact', () => {
    console.log('Contact page');
});

router.start();
```

---

## ⚡ Performance Optimization

### Debouncing and Throttling

```javascript
// Debounce function
function debounce(func, delay) {
    let timeoutId;
    
    return function debounced(...args) {
        clearTimeout(timeoutId);
        
        timeoutId = setTimeout(() => {
            func.apply(this, args);
        }, delay);
    };
}

// Throttle function
function throttle(func, delay) {
    let lastCall = 0;
    
    return function throttled(...args) {
        const now = Date.now();
        
        if (now - lastCall >= delay) {
            lastCall = now;
            func.apply(this, args);
        }
    };
}

// Advanced debounce with immediate execution option
function advancedDebounce(func, delay, immediate = false) {
    let timeoutId;
    
    return function debounced(...args) {
        const callNow = immediate && !timeoutId;
        
        clearTimeout(timeoutId);
        
        timeoutId = setTimeout(() => {
            timeoutId = null;
            if (!immediate) func.apply(this, args);
        }, delay);
        
        if (callNow) func.apply(this, args);
    };
}

// Usage examples
const searchInput = document.getElementById('search');
const scrollHandler = throttle(() => {
    console.log('Scroll event triggered');
}, 100);

const searchHandler = debounce((event) => {
    console.log('Searching for:', event.target.value);
    // Perform search API call
}, 300);

searchInput.addEventListener('input', searchHandler);
window.addEventListener('scroll', scrollHandler);

// Performance monitoring
class PerformanceMonitor {
    static measureFunction(func, name) {
        return function(...args) {
            const start = performance.now();
            const result = func.apply(this, args);
            const end = performance.now();
            
            console.log(`${name} took ${(end - start).toFixed(2)} milliseconds`);
            return result;
        };
    }
    
    static measureAsync(asyncFunc, name) {
        return async function(...args) {
            const start = performance.now();
            const result = await asyncFunc.apply(this, args);
            const end = performance.now();
            
            console.log(`${name} took ${(end - start).toFixed(2)} milliseconds`);
            return result;
        };
    }
    
    static getMemoryUsage() {
        if (performance.memory) {
            return {
                used: Math.round(performance.memory.usedJSHeapSize / 1048576),
                total: Math.round(performance.memory.totalJSHeapSize / 1048576),
                limit: Math.round(performance.memory.jsHeapSizeLimit / 1048576)
            };
        }
        return null;
    }
    
    static logPerformanceEntries() {
        const entries = performance.getEntriesByType('navigation');
        if (entries.length > 0) {
            const entry = entries[0];
            console.log('Page Load Performance:', {
                domContentLoaded: entry.domContentLoadedEventEnd - entry.domContentLoadedEventStart,
                loadComplete: entry.loadEventEnd - entry.loadEventStart,
                totalTime: entry.loadEventEnd - entry.navigationStart
            });
        }
    }
}

// Lazy loading utility
class LazyLoader {
    constructor() {
        this.observer = new IntersectionObserver(
            this.handleIntersection.bind(this),
            { threshold: 0.1 }
        );
    }
    
    observe(element, loadFunction) {
        element._lazyLoadFunction = loadFunction;
        this.observer.observe(element);
    }
    
    handleIntersection(entries) {
        entries.forEach(entry => {
            if (entry.isIntersecting) {
                const element = entry.target;
                if (element._lazyLoadFunction) {
                    element._lazyLoadFunction();
                    this.observer.unobserve(element);
                }
            }
        });
    }
}

// Usage
const lazyLoader = new LazyLoader();
const lazyImages = document.querySelectorAll('.lazy-image');

lazyImages.forEach(img => {
    lazyLoader.observe(img, () => {
        img.src = img.dataset.src;
        img.classList.remove('lazy-image');
    });
});
```

---

## 🧪 Testing Fundamentals

### Unit Testing Concepts

```javascript
// Simple test framework
class SimpleTest {
    constructor() {
        this.tests = [];
        this.passed = 0;
        this.failed = 0;
    }
    
    describe(description, callback) {
        console.group(`📋 ${description}`);
        callback();
        console.groupEnd();
    }
    
    it(description, callback) {
        try {
            callback();
            this.passed++;
            console.log(`✅ ${description}`);
        } catch (error) {
            this.failed++;
            console.error(`❌ ${description}`);
            console.error(error.message);
        }
    }
    
    expect(actual) {
        return {
            toBe: (expected) => {
                if (actual !== expected) {
                    throw new Error(`Expected ${expected}, but got ${actual}`);
                }
            },
            
            toEqual: (expected) => {
                if (JSON.stringify(actual) !== JSON.stringify(expected)) {
                    throw new Error(`Expected ${JSON.stringify(expected)}, but got ${JSON.stringify(actual)}`);
                }
            },
            
            toBeTruthy: () => {
                if (!actual) {
                    throw new Error(`Expected truthy value, but got ${actual}`);
                }
            },
            
            toBeFalsy: () => {
                if (actual) {
                    throw new Error(`Expected falsy value, but got ${actual}`);
                }
            },
            
            toContain: (expected) => {
                if (!actual.includes(expected)) {
                    throw new Error(`Expected array to contain ${expected}`);
                }
            },
            
            toThrow: () => {
                let threw = false;
                try {
                    actual();
                } catch (error) {
                    threw = true;
                }
                if (!threw) {
                    throw new Error('Expected function to throw an error');
                }
            }
        };
    }
    
    summary() {
        console.log(`\n📊 Test Summary: ${this.passed} passed, ${this.failed} failed`);
    }
}

// Example functions to test
function add(a, b) {
    return a + b;
}

function divide(a, b) {
    if (b === 0) {
        throw new Error('Division by zero');
    }
    return a / b;
}

function filterEvenNumbers(numbers) {
    return numbers.filter(n => n % 2 === 0);
}

class Calculator {
    constructor() {
        this.result = 0;
    }
    
    add(value) {
        this.result += value;
        return this;
    }
    
    multiply(value) {
        this.result *= value;
        return this;
    }
    
    getResult() {
        return this.result;
    }
    
    reset() {
        this.result = 0;
        return this;
    }
}

// Test cases
const test = new SimpleTest();

test.describe('Math Functions', () => {
    test.it('should add two numbers correctly', () => {
        test.expect(add(2, 3)).toBe(5);
        test.expect(add(-1, 1)).toBe(0);
        test.expect(add(0, 0)).toBe(0);
    });
    
    test.it('should handle division correctly', () => {
        test.expect(divide(10, 2)).toBe(5);
        test.expect(divide(7, 2)).toBe(3.5);
    });
    
    test.it('should throw error for division by zero', () => {
        test.expect(() => divide(5, 0)).toThrow();
    });
});

test.describe('Array Functions', () => {
    test.it('should filter even numbers', () => {
        test.expect(filterEvenNumbers([1, 2, 3, 4, 5, 6])).toEqual([2, 4, 6]);
        test.expect(filterEvenNumbers([1, 3, 5])).toEqual([]);
        test.expect(filterEvenNumbers([])).toEqual([]);
    });
});

test.describe('Calculator Class', () => {
    test.it('should calculate correctly with method chaining', () => {
        const calc = new Calculator();
        const result = calc.add(5).multiply(2).add(3).getResult();
        test.expect(result).toBe(13);
    });
    
    test.it('should reset correctly', () => {
        const calc = new Calculator();
        calc.add(10).reset();
        test.expect(calc.getResult()).toBe(0);
    });
});

// Mock functions for testing async code
class MockAPIClient {
    constructor(shouldFail = false) {
        this.shouldFail = shouldFail;
    }
    
    async fetchUser(id) {
        return new Promise((resolve, reject) => {
            setTimeout(() => {
                if (this.shouldFail) {
                    reject(new Error('API Error'));
                } else {
                    resolve({ id, name: `User ${id}` });
                }
            }, 100);
        });
    }
}

// Testing async functions
async function testAsyncFunctions() {
    console.group('🔄 Async Tests');
    
    // Test successful API call
    try {
        const apiClient = new MockAPIClient(false);
        const user = await apiClient.fetchUser(1);
        console.log('✅ Should fetch user successfully');
    } catch (error) {
        console.error('❌ Should fetch user successfully - Failed');
    }
    
    // Test failed API call
    try {
        const apiClient = new MockAPIClient(true);
        await apiClient.fetchUser(1);
        console.error('❌ Should handle API error - Failed');
    } catch (error) {
        console.log('✅ Should handle API error');
    }
    
    console.groupEnd();
}

// Run tests
test.summary();
testAsyncFunctions();

// Property-based testing concept
function generateRandomArray(length = 10) {
    return Array.from({ length }, () => Math.floor(Math.random() * 100));
}

function testProperty(name, property, generator, iterations = 100) {
    console.group(`🎲 Property Test: ${name}`);
    
    let passed = 0;
    for (let i = 0; i < iterations; i++) {
        const input = generator();
        try {
            property(input);
            passed++;
        } catch (error) {
            console.error(`Failed with input:`, input, error.message);
        }
    }
    
    console.log(`✅ ${passed}/${iterations} tests passed`);
    console.groupEnd();
}

// Example property tests
testProperty(
    'Array sort should not change length',
    (arr) => {
        const original = [...arr];
        const sorted = arr.sort();
        if (sorted.length !== original.length) {
            throw new Error('Length changed after sorting');
        }
    },
    generateRandomArray
);

testProperty(
    'Reversing twice should return original',
    (arr) => {
        const original = [...arr];
        const reversed = arr.reverse().reverse();
        if (JSON.stringify(reversed) !== JSON.stringify(original)) {
            throw new Error('Double reverse did not return original');
        }
    },
    generateRandomArray
);
```

## 📚 Complete Example: Task Management App

```html
<!DOCTYPE html>
<html lang="en">
<head>
    <meta charset="UTF-8">
    <meta name="viewport" content="width=device-width, initial-scale=1.0">
    <title>Advanced Task Manager</title>
    <style>
        * {
            margin: 0;
            padding: 0;
            box-sizing: border-box;
        }

        body {
            font-family: 'Segoe UI', Tahoma, Geneva, Verdana, sans-serif;
            background: linear-gradient(135deg, #667eea 0%, #764ba2 100%);
            min-height: 100vh;
            padding: 20px;
        }

        .container {
            max-width: 800px;
            margin: 0 auto;
            background: white;
            border-radius: 15px;
            box-shadow: 0 20px 40px rgba(0, 0, 0, 0.1);
            overflow: hidden;
        }

        .header {
            background: linear-gradient(135deg, #4facfe 0%, #00f2fe 100%);
            color: white;
            padding: 30px;
            text-align: center;
        }

        .header h1 {
            font-size: 2.5rem;
            margin-bottom: 10px;
        }

        .main-content {
            padding: 30px;
        }

        .task-form {
            display: flex;
            gap: 10px;
            margin-bottom: 30px;
            flex-wrap: wrap;
        }

        .task-input {
            flex: 1;
            padding: 15px;
            border: 2px solid #e1e8ed;
            border-radius: 8px;
            font-size: 16px;
            min-width: 200px;
        }

        .task-input:focus {
            outline: none;
            border-color: #4facfe;
        }

        .priority-select {
            padding: 15px;
            border: 2px solid #e1e8ed;
            border-radius: 8px;
            font-size: 16px;
        }

        .add-btn {
            background: linear-gradient(135deg, #4facfe 0%, #00f2fe 100%);
            color: white;
            border: none;
            padding: 15px 25px;
            border-radius: 8px;
            font-size: 16px;
            cursor: pointer;
            transition: transform 0.2s;
        }

        .add-btn:hover {
            transform: translateY(-2px);
        }

        .filters {
            display: flex;
            gap: 10px;
            margin-bottom: 20px;
            flex-wrap: wrap;
        }

        .filter-btn {
            background: #f8f9fa;
            border: 2px solid #e1e8ed;
            padding: 10px 20px;
            border-radius: 20px;
            cursor: pointer;
            transition: all 0.2s;
        }

        .filter-btn.active {
            background: #4facfe;
            color: white;
            border-color: #4facfe;
        }

        .task-list {
            min-height: 300px;
        }

        .task-item {
            background: #f8f9fa;
            border: 1px solid #e1e8ed;
            border-radius: 8px;
            padding: 20px;
            margin-bottom: 15px;
            display: flex;
            align-items: center;
            gap: 15px;
            transition: all 0.3s;
        }

        .task-item.completed {
            opacity: 0.6;
            background: #e8f5e8;
        }

        .task-item.high-priority {
            border-left: 4px solid #ff4757;
        }

        .task-item.medium-priority {
            border-left: 4px solid #ffa502;
        }

        .task-item.low-priority {
            border-left: 4px solid #26de81;
        }

        .task-checkbox {
            width: 20px;
            height: 20px;
            cursor: pointer;
        }

        .task-content {
            flex: 1;
        }

        .task-text {
            font-size: 16px;
            margin-bottom: 5px;
        }

        .task-text.completed {
            text-decoration: line-through;
        }

        .task-meta {
            font-size: 12px;
            color: #657786;
        }

        .task-actions {
            display: flex;
            gap: 10px;
        }

        .action-btn {
            background: none;
            border: none;
            font-size: 18px;
            cursor: pointer;
            padding: 5px;
            border-radius: 4px;
            transition: background 0.2s;
        }

        .action-btn:hover {
            background: #e1e8ed;
        }

        .stats {
            display: grid;
            grid-template-columns: repeat(auto-fit, minmax(150px, 1fr));
            gap: 20px;
            margin-top: 30px;
        }

        .stat-card {
            background: linear-gradient(135deg, #667eea 0%, #764ba2 100%);
            color: white;
            padding: 20px;
            border-radius: 8px;
            text-align: center;
        }

        .stat-number {
            font-size: 2rem;
            font-weight: bold;
            margin-bottom: 5px;
        }

        .empty-state {
            text-align: center;
            padding: 60px 20px;
            color: #657786;
        }

        .empty-state-icon {
            font-size: 4rem;
            margin-bottom: 20px;
        }

        @media (max-width: 600px) {
            .task-form {
                flex-direction: column;
            }

            .task-input {
                min-width: 100%;
            }

            .filters {
                justify-content: center;
            }

            .task-item {
                flex-direction: column;
                align-items: stretch;
                gap: 10px;
            }

            .task-actions {
                justify-content: center;
            }
        }
    </style>
</head>
<body>
    <div class="container">
        <header class="header">
            <h1>🚀 Advanced Task Manager</h1>
            <p>Manage your tasks with style and efficiency</p>
        </header>

        <main class="main-content">
            <form class="task-form" id="taskForm">
                <input 
                    type="text" 
                    class="task-input" 
                    id="taskInput" 
                    placeholder="What needs to be done?"
                    required
                >
                <select class="priority-select" id="prioritySelect">
                    <option value="low">Low Priority</option>
                    <option value="medium" selected>Medium Priority</option>
                    <option value="high">High Priority</option>
                </select>
                <button type="submit" class="add-btn">Add Task</button>
            </form>

            <div class="filters">
                <button class="filter-btn active" data-filter="all">All</button>
                <button class="filter-btn" data-filter="active">Active</button>
                <button class="filter-btn" data-filter="completed">Completed</button>
                <button class="filter-btn" data-filter="high">High Priority</button>
                <button class="filter-btn" data-filter="medium">Medium Priority</button>
                <button class="filter-btn" data-filter="low">Low Priority</button>
            </div>

            <div class="task-list" id="taskList">
                <div class="empty-state" id="emptyState">
                    <div class="empty-state-icon">📝</div>
                    <h3>No tasks yet</h3>
                    <p>Add your first task to get started!</p>
                </div>
            </div>

            <div class="stats">
                <div class="stat-card">
                    <div class="stat-number" id="totalTasks">0</div>
                    <div>Total Tasks</div>
                </div>
                <div class="stat-card">
                    <div class="stat-number" id="completedTasks">0</div>
                    <div>Completed</div>
                </div>
                <div class="stat-card">
                    <div class="stat-number" id="pendingTasks">0</div>
                    <div>Pending</div>
                </div>
                <div class="stat-card">
                    <div class="stat-number" id="completionRate">0%</div>
                    <div>Completion Rate</div>
                </div>
            </div>
        </main>
    </div>

    <script>
        class AdvancedTaskManager {
            constructor() {
                this.tasks = this.loadTasks();
                this.currentFilter = 'all';
                this.taskIdCounter = this.getNextId();
                
                this.bindEvents();
                this.render();
            }
            
            bindEvents() {
                // Form submission
                document.getElementById('taskForm').addEventListener('submit', (e) => {
                    e.preventDefault();
                    this.addTask();
                });
                
                // Filter buttons
                document.querySelectorAll('.filter-btn').forEach(btn => {
                    btn.addEventListener('click', (e) => {
                        this.setFilter(e.target.dataset.filter);
                    });
                });
                
                // Task list events (event delegation)
                document.getElementById('taskList').addEventListener('click', (e) => {
                    const taskItem = e.target.closest('.task-item');
                    if (!taskItem) return;
                    
                    const taskId = parseInt(taskItem.dataset.taskId);
                    
                    if (e.target.classList.contains('task-checkbox')) {
                        this.toggleTask(taskId);
                    } else if (e.target.textContent === '✏️') {
                        this.editTask(taskId);
                    } else if (e.target.textContent === '🗑️') {
                        this.deleteTask(taskId);
                    }
                });
                
                // Keyboard shortcuts
                document.addEventListener('keydown', (e) => {
                    if (e.ctrlKey || e.metaKey) {
                        switch (e.key) {
                            case 'a':
                                e.preventDefault();
                                this.setFilter('all');
                                break;
                            case 'c':
                                e.preventDefault();
                                this.setFilter('completed');
                                break;
                            case 'p':
                                e.preventDefault();
                                this.setFilter('active');
                                break;
                        }
                    }
                });
            }
            
            addTask() {
                const taskInput = document.getElementById('taskInput');
                const prioritySelect = document.getElementById('prioritySelect');
                
                const text = taskInput.value.trim();
                if (!text) return;
                
                const task = {
                    id: this.taskIdCounter++,
                    text: text,
                    completed: false,
                    priority: prioritySelect.value,
                    createdAt: new Date().toISOString(),
                    completedAt: null
                };
                
                this.tasks.push(task);
                this.saveTasks();
                this.render();
                
                // Reset form
                taskInput.value = '';
                taskInput.focus();
                
                // Show success animation
                this.showNotification(`Task "${text}" added successfully!`);
            }
            
            toggleTask(id) {
                const task = this.tasks.find(t => t.id === id);
                if (!task) return;
                
                task.completed = !task.completed;
                task.completedAt = task.completed ? new Date().toISOString() : null;
                
                this.saveTasks();
                this.render();
                
                const message = task.completed ? 
                    `Task "${task.text}" completed! 🎉` : 
                    `Task "${task.text}" marked as active`;
                this.showNotification(message);
            }
            
            editTask(id) {
                const task = this.tasks.find(t => t.id === id);
                if (!task) return;
                
                const newText = prompt('Edit task:', task.text);
                if (newText && newText.trim() && newText.trim() !== task.text) {
                    task.text = newText.trim();
                    this.saveTasks();
                    this.render();
                    this.showNotification('Task updated successfully!');
                }
            }
            
            deleteTask(id) {
                const task = this.tasks.find(t => t.id === id);
                if (!task) return;
                
                if (confirm(`Are you sure you want to delete "${task.text}"?`)) {
                    this.tasks = this.tasks.filter(t => t.id !== id);
                    this.saveTasks();
                    this.render();
                    this.showNotification('Task deleted successfully!');
                }
            }
            
            setFilter(filter) {
                this.currentFilter = filter;
                
                // Update active filter button
                document.querySelectorAll('.filter-btn').forEach(btn => {
                    btn.classList.toggle('active', btn.dataset.filter === filter);
                });
                
                this.render();
            }
            
            getFilteredTasks() {
                switch (this.currentFilter) {
                    case 'completed':
                        return this.tasks.filter(t => t.completed);
                    case 'active':
                        return this.tasks.filter(t => !t.completed);
                    case 'high':
                        return this.tasks.filter(t => t.priority === 'high');
                    case 'medium':
                        return this.tasks.filter(t => t.priority === 'medium');
                    case 'low':
                        return this.tasks.filter(t => t.priority === 'low');
                    default:
                        return this.tasks;
                }
            }
            
            render() {
                this.renderTasks();
                this.renderStats();
            }
            
            renderTasks() {
                const taskList = document.getElementById('taskList');
                const emptyState = document.getElementById('emptyState');
                const filteredTasks = this.getFilteredTasks();
                
                if (filteredTasks.length === 0) {
                    emptyState.style.display = 'block';
                    return;
                }
                
                emptyState.style.display = 'none';
                
                const tasksHTML = filteredTasks
                    .sort((a, b) => {
                        // Sort by completion status, then by priority, then by creation date
                        if (a.completed !== b.completed) {
                            return a.completed ? 1 : -1;
                        }
                        
                        const priorityOrder = { high: 3, medium: 2, low: 1 };
                        if (priorityOrder[a.priority] !== priorityOrder[b.priority]) {
                            return priorityOrder[b.priority] - priorityOrder[a.priority];
                        }
                        
                        return new Date(b.createdAt) - new Date(a.createdAt);
                    })
                    .map(task => this.createTaskHTML(task))
                    .join('');
                
                taskList.innerHTML = tasksHTML;
            }
            
            createTaskHTML(task) {
                const createdDate = new Date(task.createdAt).toLocaleDateString();
                const completedDate = task.completedAt ? 
                    new Date(task.completedAt).toLocaleDateString() : null;
                
                return `
                    <div class="task-item ${task.completed ? 'completed' : ''} ${task.priority}-priority" 
                         data-task-id="${task.id}">
                        <input type="checkbox" class="task-checkbox" ${task.completed ? 'checked' : ''}>
                        <div class="task-content">
                            <div class="task-text ${task.completed ? 'completed' : ''}">${task.text}</div>
                            <div class="task-meta">
                                Priority: ${task.priority.charAt(0).toUpperCase() + task.priority.slice(1)} • 
                                Created: ${createdDate}
                                ${completedDate ? ` • Completed: ${completedDate}` : ''}
                            </div>
                        </div>
                        <div class="task-actions">
                            <button class="action-btn" title="Edit task">✏️</button>
                            <button class="action-btn" title="Delete task">🗑️</button>
                        </div>
                    </div>
                `;
            }
            
            renderStats() {
                const total = this.tasks.length;
                const completed = this.tasks.filter(t => t.completed).length;
                const pending = total - completed;
                const completionRate = total > 0 ? Math.round((completed / total) * 100) : 0;
                
                document.getElementById('totalTasks').textContent = total;
                document.getElementById('completedTasks').textContent = completed;
                document.getElementById('pendingTasks').textContent = pending;
                document.getElementById('completionRate').textContent = `${completionRate}%`;
            }
            
            showNotification(message) {
                // Create notification element
                const notification = document.createElement('div');
                notification.style.cssText = `
                    position: fixed;
                    top: 20px;
                    right: 20px;
                    background: #4facfe;
                    color: white;
                    padding: 15px 20px;
                    border-radius: 8px;
                    box-shadow: 0 4px 12px rgba(0, 0, 0, 0.2);
                    z-index: 1000;
                    transform: translateX(100%);
                    transition: transform 0.3s ease;
                `;
                notification.textContent = message;
                
                document.body.appendChild(notification);
                
                // Animate in
                setTimeout(() => {
                    notification.style.transform = 'translateX(0)';
                }, 10);
                
                // Animate out and remove
                setTimeout(() => {
                    notification.style.transform = 'translateX(100%)';
                    setTimeout(() => {
                        document.body.removeChild(notification);
                    }, 300);
                }, 3000);
            }
            
            saveTasks() {
                localStorage.setItem('advanced-tasks', JSON.stringify(this.tasks));
            }
            
            loadTasks() {
                const saved = localStorage.getItem('advanced-tasks');
                return saved ? JSON.parse(saved) : [];
            }
            
            getNextId() {
                const maxId = this.tasks.reduce((max, task) => Math.max(max, task.id || 0), 0);
                return maxId + 1;
            }
        }
        
        // Initialize the app
        document.addEventListener('DOMContentLoaded', () => {
            new AdvancedTaskManager();
        });
    </script>
</body>
</html>
```

This comprehensive Intermediate JavaScript guide covers essential concepts for advancing your JavaScript skills. Practice these examples and build projects to reinforce your understanding!
# 🚀 JavaScript Practical Exercises

## 🎯 Learning Objectives
By completing these exercises, you will:
- Practice JavaScript fundamentals and ES6+ features
- Build interactive web applications
- Work with APIs and asynchronous programming
- Implement common programming patterns
- Debug and optimize JavaScript code

---

## 🔧 Exercise 1: DOM Manipulation and Events

### Task: Build an Interactive Todo App

Create a fully functional todo application with local storage.

```html
<!DOCTYPE html>
<html lang="en">
<head>
    <meta charset="UTF-8">
    <meta name="viewport" content="width=device-width, initial-scale=1.0">
    <title>Todo App</title>
    <link rel="stylesheet" href="todo-style.css">
</head>
<body>
    <div class="todo-app">
        <header class="app-header">
            <h1>My Todo App</h1>
            <p class="date" id="current-date"></p>
        </header>
        
        <form class="todo-form" id="todo-form">
            <input 
                type="text" 
                id="todo-input" 
                placeholder="What needs to be done?"
                maxlength="100"
                required
            >
            <button type="submit" class="btn-add">Add Task</button>
        </form>
        
        <div class="todo-filters">
            <button class="filter-btn active" data-filter="all">All</button>
            <button class="filter-btn" data-filter="active">Active</button>
            <button class="filter-btn" data-filter="completed">Completed</button>
        </div>
        
        <ul class="todo-list" id="todo-list">
            <!-- Todo items will be inserted here -->
        </ul>
        
        <div class="todo-stats">
            <span class="todo-count" id="todo-count">0 items left</span>
            <button class="btn-clear" id="clear-completed">Clear Completed</button>
        </div>
    </div>
    
    <script src="todo-script.js"></script>
</body>
</html>
```

**Your JavaScript Task (`todo-script.js`):**

```javascript
// Todo App Implementation
class TodoApp {
    constructor() {
        this.todos = this.loadTodos();
        this.currentFilter = 'all';
        this.init();
    }

    init() {
        this.bindEvents();
        this.updateDate();
        this.render();
    }

    bindEvents() {
        // Form submission
        document.getElementById('todo-form').addEventListener('submit', (e) => {
            e.preventDefault();
            this.addTodo();
        });

        // Filter buttons
        document.querySelectorAll('.filter-btn').forEach(btn => {
            btn.addEventListener('click', (e) => {
                this.setFilter(e.target.dataset.filter);
            });
        });

        // Clear completed button
        document.getElementById('clear-completed').addEventListener('click', () => {
            this.clearCompleted();
        });

        // Input validation
        const todoInput = document.getElementById('todo-input');
        todoInput.addEventListener('input', (e) => {
            this.validateInput(e.target);
        });
    }

    addTodo() {
        const input = document.getElementById('todo-input');
        const text = input.value.trim();
        
        if (!text) return;

        const todo = {
            id: Date.now().toString(),
            text: text,
            completed: false,
            createdAt: new Date().toISOString()
        };

        this.todos.unshift(todo);
        this.saveTodos();
        this.render();
        
        // Clear input and show success animation
        input.value = '';
        this.showNotification('Task added successfully!');
    }

    deleteTodo(id) {
        this.todos = this.todos.filter(todo => todo.id !== id);
        this.saveTodos();
        this.render();
        this.showNotification('Task deleted!');
    }

    toggleTodo(id) {
        const todo = this.todos.find(todo => todo.id === id);
        if (todo) {
            todo.completed = !todo.completed;
            this.saveTodos();
            this.render();
        }
    }

    editTodo(id, newText) {
        const todo = this.todos.find(todo => todo.id === id);
        if (todo && newText.trim()) {
            todo.text = newText.trim();
            this.saveTodos();
            this.render();
            this.showNotification('Task updated!');
        }
    }

    setFilter(filter) {
        this.currentFilter = filter;
        
        // Update active filter button
        document.querySelectorAll('.filter-btn').forEach(btn => {
            btn.classList.toggle('active', btn.dataset.filter === filter);
        });
        
        this.render();
    }

    clearCompleted() {
        const completedCount = this.todos.filter(todo => todo.completed).length;
        if (completedCount === 0) {
            this.showNotification('No completed tasks to clear!');
            return;
        }

        this.todos = this.todos.filter(todo => !todo.completed);
        this.saveTodos();
        this.render();
        this.showNotification(`${completedCount} completed tasks cleared!`);
    }

    getFilteredTodos() {
        switch (this.currentFilter) {
            case 'active':
                return this.todos.filter(todo => !todo.completed);
            case 'completed':
                return this.todos.filter(todo => todo.completed);
            default:
                return this.todos;
        }
    }

    render() {
        const todoList = document.getElementById('todo-list');
        const filteredTodos = this.getFilteredTodos();

        // Clear current todos
        todoList.innerHTML = '';

        if (filteredTodos.length === 0) {
            todoList.innerHTML = `
                <li class="no-todos">
                    <p>No tasks ${this.currentFilter === 'all' ? '' : this.currentFilter} yet!</p>
                </li>
            `;
        } else {
            filteredTodos.forEach(todo => {
                const todoElement = this.createTodoElement(todo);
                todoList.appendChild(todoElement);
            });
        }

        this.updateStats();
    }

    createTodoElement(todo) {
        const li = document.createElement('li');
        li.className = `todo-item ${todo.completed ? 'completed' : ''}`;
        li.setAttribute('data-id', todo.id);

        li.innerHTML = `
            <div class="todo-content">
                <label class="todo-checkbox">
                    <input type="checkbox" ${todo.completed ? 'checked' : ''}>
                    <span class="checkmark"></span>
                </label>
                <span class="todo-text" ${!todo.completed ? 'contenteditable="true"' : ''}>${todo.text}</span>
            </div>
            <div class="todo-actions">
                <button class="btn-edit" title="Edit task">✏️</button>
                <button class="btn-delete" title="Delete task">🗑️</button>
            </div>
        `;

        // Bind events for this todo item
        this.bindTodoEvents(li, todo);

        return li;
    }

    bindTodoEvents(todoElement, todo) {
        // Toggle completion
        const checkbox = todoElement.querySelector('input[type="checkbox"]');
        checkbox.addEventListener('change', () => {
            this.toggleTodo(todo.id);
        });

        // Edit todo text
        const todoText = todoElement.querySelector('.todo-text');
        if (!todo.completed) {
            todoText.addEventListener('blur', (e) => {
                this.editTodo(todo.id, e.target.textContent);
            });

            todoText.addEventListener('keydown', (e) => {
                if (e.key === 'Enter') {
                    e.preventDefault();
                    e.target.blur();
                }
            });
        }

        // Delete todo
        const deleteBtn = todoElement.querySelector('.btn-delete');
        deleteBtn.addEventListener('click', () => {
            if (confirm('Are you sure you want to delete this task?')) {
                this.deleteTodo(todo.id);
            }
        });

        // Edit button
        const editBtn = todoElement.querySelector('.btn-edit');
        editBtn.addEventListener('click', () => {
            todoText.focus();
        });
    }

    updateStats() {
        const activeTodos = this.todos.filter(todo => !todo.completed);
        const count = activeTodos.length;
        const todoCount = document.getElementById('todo-count');
        
        todoCount.textContent = `${count} ${count === 1 ? 'item' : 'items'} left`;

        // Update clear button visibility
        const clearBtn = document.getElementById('clear-completed');
        const completedCount = this.todos.filter(todo => todo.completed).length;
        clearBtn.style.opacity = completedCount > 0 ? '1' : '0.5';
        clearBtn.disabled = completedCount === 0;
    }

    updateDate() {
        const now = new Date();
        const options = { 
            weekday: 'long', 
            year: 'numeric', 
            month: 'long', 
            day: 'numeric' 
        };
        
        document.getElementById('current-date').textContent = 
            now.toLocaleDateString('en-US', options);
    }

    validateInput(input) {
        const value = input.value.trim();
        const isValid = value.length > 0 && value.length <= 100;
        
        input.classList.toggle('invalid', !isValid && value.length > 0);
        
        // Update submit button state
        const submitBtn = document.querySelector('.btn-add');
        submitBtn.disabled = !isValid;
    }

    showNotification(message) {
        // Create notification element
        const notification = document.createElement('div');
        notification.className = 'notification';
        notification.textContent = message;
        
        document.body.appendChild(notification);
        
        // Trigger animation
        setTimeout(() => notification.classList.add('show'), 100);
        
        // Remove after 3 seconds
        setTimeout(() => {
            notification.classList.remove('show');
            setTimeout(() => notification.remove(), 300);
        }, 3000);
    }

    // Local Storage Methods
    saveTodos() {
        localStorage.setItem('todos', JSON.stringify(this.todos));
    }

    loadTodos() {
        try {
            const todos = localStorage.getItem('todos');
            return todos ? JSON.parse(todos) : [];
        } catch (error) {
            console.error('Error loading todos:', error);
            return [];
        }
    }
}

// Initialize the app when DOM is loaded
document.addEventListener('DOMContentLoaded', () => {
    new TodoApp();
});

// Keyboard shortcuts
document.addEventListener('keydown', (e) => {
    // Ctrl/Cmd + K to focus on input
    if ((e.ctrlKey || e.metaKey) && e.key === 'k') {
        e.preventDefault();
        document.getElementById('todo-input').focus();
    }
});
```

**Supporting CSS (`todo-style.css`):**

```css
/* Todo App Styles */
* {
    margin: 0;
    padding: 0;
    box-sizing: border-box;
}

body {
    font-family: 'Segoe UI', Tahoma, Geneva, Verdana, sans-serif;
    background: linear-gradient(135deg, #667eea 0%, #764ba2 100%);
    min-height: 100vh;
    padding: 20px;
}

.todo-app {
    max-width: 600px;
    margin: 0 auto;
    background: white;
    border-radius: 12px;
    box-shadow: 0 20px 40px rgba(0,0,0,0.1);
    overflow: hidden;
}

.app-header {
    background: linear-gradient(135deg, #667eea, #764ba2);
    color: white;
    padding: 30px;
    text-align: center;
}

.app-header h1 {
    font-size: 2rem;
    margin-bottom: 8px;
}

.date {
    opacity: 0.9;
    font-size: 0.9rem;
}

.todo-form {
    padding: 20px;
    display: flex;
    gap: 10px;
}

#todo-input {
    flex: 1;
    padding: 12px 16px;
    border: 2px solid #e1e5e9;
    border-radius: 8px;
    font-size: 1rem;
    transition: border-color 0.3s ease;
}

#todo-input:focus {
    outline: none;
    border-color: #667eea;
}

#todo-input.invalid {
    border-color: #e74c3c;
}

.btn-add {
    padding: 12px 24px;
    background: #667eea;
    color: white;
    border: none;
    border-radius: 8px;
    font-weight: 600;
    cursor: pointer;
    transition: background 0.3s ease;
}

.btn-add:hover:not(:disabled) {
    background: #5a6fd8;
}

.btn-add:disabled {
    background: #bdc3c7;
    cursor: not-allowed;
}

.todo-filters {
    display: flex;
    justify-content: center;
    gap: 0;
    margin: 0 20px;
    border-radius: 8px;
    overflow: hidden;
    border: 2px solid #e1e5e9;
}

.filter-btn {
    flex: 1;
    padding: 10px;
    background: white;
    border: none;
    cursor: pointer;
    transition: all 0.3s ease;
}

.filter-btn.active {
    background: #667eea;
    color: white;
}

.todo-list {
    list-style: none;
    max-height: 400px;
    overflow-y: auto;
}

.todo-item {
    display: flex;
    align-items: center;
    justify-content: space-between;
    padding: 16px 20px;
    border-bottom: 1px solid #f1f3f4;
    transition: all 0.3s ease;
}

.todo-item:hover {
    background: #f8f9fa;
}

.todo-item.completed {
    opacity: 0.6;
}

.todo-item.completed .todo-text {
    text-decoration: line-through;
}

.todo-content {
    display: flex;
    align-items: center;
    flex: 1;
    gap: 12px;
}

.todo-checkbox {
    position: relative;
    cursor: pointer;
}

.todo-checkbox input {
    opacity: 0;
    position: absolute;
}

.checkmark {
    width: 20px;
    height: 20px;
    border: 2px solid #ddd;
    border-radius: 4px;
    display: block;
    transition: all 0.3s ease;
}

.todo-checkbox input:checked + .checkmark {
    background: #667eea;
    border-color: #667eea;
}

.todo-checkbox input:checked + .checkmark::after {
    content: '✓';
    color: white;
    font-size: 14px;
    position: absolute;
    top: 50%;
    left: 50%;
    transform: translate(-50%, -50%);
}

.todo-text {
    flex: 1;
    padding: 4px;
    border-radius: 4px;
    transition: background 0.3s ease;
}

.todo-text:focus {
    outline: none;
    background: #f8f9fa;
}

.todo-actions {
    display: flex;
    gap: 8px;
    opacity: 0;
    transition: opacity 0.3s ease;
}

.todo-item:hover .todo-actions {
    opacity: 1;
}

.btn-edit,
.btn-delete {
    background: none;
    border: none;
    padding: 6px;
    border-radius: 4px;
    cursor: pointer;
    transition: background 0.3s ease;
}

.btn-edit:hover {
    background: #e3f2fd;
}

.btn-delete:hover {
    background: #ffebee;
}

.no-todos {
    text-align: center;
    padding: 40px 20px;
    color: #888;
}

.todo-stats {
    display: flex;
    justify-content: space-between;
    align-items: center;
    padding: 16px 20px;
    background: #f8f9fa;
    font-size: 0.9rem;
    color: #666;
}

.btn-clear {
    background: none;
    border: none;
    color: #e74c3c;
    cursor: pointer;
    transition: opacity 0.3s ease;
}

.btn-clear:hover:not(:disabled) {
    text-decoration: underline;
}

.btn-clear:disabled {
    color: #bdc3c7;
    cursor: not-allowed;
}

/* Notification */
.notification {
    position: fixed;
    top: 20px;
    right: 20px;
    background: #2ecc71;
    color: white;
    padding: 12px 20px;
    border-radius: 8px;
    box-shadow: 0 4px 12px rgba(0,0,0,0.2);
    transform: translateX(100%);
    transition: transform 0.3s ease;
    z-index: 1000;
}

.notification.show {
    transform: translateX(0);
}

/* Responsive */
@media (max-width: 640px) {
    .todo-app {
        margin: 0;
        border-radius: 0;
        min-height: 100vh;
    }
    
    .todo-form {
        flex-direction: column;
    }
    
    .todo-item {
        padding: 12px 16px;
    }
    
    .todo-actions {
        opacity: 1;
    }
}
```

---

## 🌐 Exercise 2: API Integration and Async Programming

### Task: Build a Weather Dashboard

Create a weather application that fetches data from an API and displays current weather and forecast.

```html
<!DOCTYPE html>
<html lang="en">
<head>
    <meta charset="UTF-8">
    <meta name="viewport" content="width=device-width, initial-scale=1.0">
    <title>Weather Dashboard</title>
    <link rel="stylesheet" href="weather-style.css">
</head>
<body>
    <div class="weather-app">
        <header class="weather-header">
            <h1>Weather Dashboard</h1>
            <div class="search-container">
                <input 
                    type="text" 
                    id="city-search" 
                    placeholder="Enter city name..."
                    autocomplete="off"
                >
                <button id="search-btn">Search</button>
                <button id="location-btn" title="Use current location">📍</button>
            </div>
        </header>

        <div class="loading" id="loading">
            <div class="spinner"></div>
            <p>Loading weather data...</p>
        </div>

        <div class="error-message" id="error-message">
            <p>Something went wrong. Please try again.</p>
        </div>

        <div class="weather-content" id="weather-content">
            <div class="current-weather">
                <div class="weather-main">
                    <div class="location">
                        <h2 id="city-name">--</h2>
                        <p id="date-time">--</p>
                    </div>
                    <div class="temperature">
                        <span id="current-temp">--°</span>
                        <div class="weather-icon">
                            <img id="weather-icon" src="" alt="Weather icon">
                        </div>
                    </div>
                </div>
                
                <div class="weather-details">
                    <div class="detail-item">
                        <span class="label">Feels like</span>
                        <span id="feels-like">--°</span>
                    </div>
                    <div class="detail-item">
                        <span class="label">Humidity</span>
                        <span id="humidity">--%</span>
                    </div>
                    <div class="detail-item">
                        <span class="label">Wind Speed</span>
                        <span id="wind-speed">-- km/h</span>
                    </div>
                    <div class="detail-item">
                        <span class="label">Pressure</span>
                        <span id="pressure">-- hPa</span>
                    </div>
                    <div class="detail-item">
                        <span class="label">Visibility</span>
                        <span id="visibility">-- km</span>
                    </div>
                    <div class="detail-item">
                        <span class="label">UV Index</span>
                        <span id="uv-index">--</span>
                    </div>
                </div>
            </div>

            <div class="forecast-section">
                <h3>5-Day Forecast</h3>
                <div class="forecast-container" id="forecast-container">
                    <!-- Forecast items will be inserted here -->
                </div>
            </div>
        </div>

        <div class="recent-searches">
            <h3>Recent Searches</h3>
            <div class="recent-cities" id="recent-cities">
                <!-- Recent cities will be inserted here -->
            </div>
        </div>
    </div>

    <script src="weather-script.js"></script>
</body>
</html>
```

**Your JavaScript Task (`weather-script.js`):**

```javascript
// Weather Dashboard Implementation
class WeatherDashboard {
    constructor() {
        // Use OpenWeatherMap API (you'll need to get a free API key)
        this.API_KEY = 'YOUR_API_KEY_HERE'; // Replace with your actual API key
        this.BASE_URL = 'https://api.openweathermap.org/data/2.5';
        this.recentSearches = this.loadRecentSearches();
        this.init();
    }

    init() {
        this.bindEvents();
        this.renderRecentSearches();
        this.loadDefaultCity();
    }

    bindEvents() {
        // Search functionality
        document.getElementById('search-btn').addEventListener('click', () => {
            this.searchWeather();
        });

        document.getElementById('city-search').addEventListener('keypress', (e) => {
            if (e.key === 'Enter') {
                this.searchWeather();
            }
        });

        // Geolocation
        document.getElementById('location-btn').addEventListener('click', () => {
            this.getCurrentLocationWeather();
        });

        // Search input with debouncing
        let searchTimeout;
        document.getElementById('city-search').addEventListener('input', (e) => {
            clearTimeout(searchTimeout);
            searchTimeout = setTimeout(() => {
                if (e.target.value.length > 2) {
                    this.showSearchSuggestions(e.target.value);
                }
            }, 300);
        });
    }

    async searchWeather() {
        const cityInput = document.getElementById('city-search');
        const cityName = cityInput.value.trim();
        
        if (!cityName) {
            this.showError('Please enter a city name');
            return;
        }

        await this.getWeatherData(cityName);
        cityInput.value = '';
    }

    async getCurrentLocationWeather() {
        if (!navigator.geolocation) {
            this.showError('Geolocation is not supported by this browser');
            return;
        }

        this.showLoading();

        try {
            const position = await this.getCurrentPosition();
            const { latitude, longitude } = position.coords;
            await this.getWeatherByCoords(latitude, longitude);
        } catch (error) {
            this.showError('Unable to get your location');
        }
    }

    getCurrentPosition() {
        return new Promise((resolve, reject) => {
            navigator.geolocation.getCurrentPosition(resolve, reject, {
                timeout: 10000,
                enableHighAccuracy: true
            });
        });
    }

    async getWeatherData(cityName) {
        this.showLoading();

        try {
            // Get current weather
            const currentResponse = await fetch(
                `${this.BASE_URL}/weather?q=${cityName}&appid=${this.API_KEY}&units=metric`
            );

            if (!currentResponse.ok) {
                throw new Error(`City not found: ${cityName}`);
            }

            const currentData = await currentResponse.json();

            // Get forecast data
            const forecastResponse = await fetch(
                `${this.BASE_URL}/forecast?q=${cityName}&appid=${this.API_KEY}&units=metric`
            );

            const forecastData = await forecastResponse.json();

            // Display weather data
            this.displayWeatherData(currentData, forecastData);
            this.addToRecentSearches(cityName);
            this.hideLoading();
            this.hideError();

        } catch (error) {
            this.showError(error.message);
            this.hideLoading();
        }
    }

    async getWeatherByCoords(lat, lon) {
        try {
            // Get current weather by coordinates
            const currentResponse = await fetch(
                `${this.BASE_URL}/weather?lat=${lat}&lon=${lon}&appid=${this.API_KEY}&units=metric`
            );

            const currentData = await currentResponse.json();

            // Get forecast data
            const forecastResponse = await fetch(
                `${this.BASE_URL}/forecast?lat=${lat}&lon=${lon}&appid=${this.API_KEY}&units=metric`
            );

            const forecastData = await forecastResponse.json();

            this.displayWeatherData(currentData, forecastData);
            this.addToRecentSearches(currentData.name);
            this.hideLoading();
            this.hideError();

        } catch (error) {
            this.showError('Unable to fetch weather data');
            this.hideLoading();
        }
    }

    displayWeatherData(currentData, forecastData) {
        // Update current weather
        document.getElementById('city-name').textContent = 
            `${currentData.name}, ${currentData.sys.country}`;
        
        document.getElementById('date-time').textContent = 
            this.formatDateTime(new Date());
        
        document.getElementById('current-temp').textContent = 
            `${Math.round(currentData.main.temp)}°C`;
        
        document.getElementById('feels-like').textContent = 
            `${Math.round(currentData.main.feels_like)}°C`;
        
        document.getElementById('humidity').textContent = 
            `${currentData.main.humidity}%`;
        
        document.getElementById('wind-speed').textContent = 
            `${Math.round(currentData.wind.speed * 3.6)} km/h`;
        
        document.getElementById('pressure').textContent = 
            `${currentData.main.pressure} hPa`;
        
        document.getElementById('visibility').textContent = 
            `${(currentData.visibility / 1000).toFixed(1)} km`;

        // Weather icon
        const iconCode = currentData.weather[0].icon;
        document.getElementById('weather-icon').src = 
            `https://openweathermap.org/img/wn/${iconCode}@2x.png`;
        document.getElementById('weather-icon').alt = 
            currentData.weather[0].description;

        // UV Index (would need additional API call in real implementation)
        document.getElementById('uv-index').textContent = 'N/A';

        // Update forecast
        this.displayForecast(forecastData);

        // Show weather content
        document.getElementById('weather-content').style.display = 'block';
    }

    displayForecast(forecastData) {
        const forecastContainer = document.getElementById('forecast-container');
        forecastContainer.innerHTML = '';

        // Group forecast by day (every 8th item for daily forecast)
        const dailyForecasts = [];
        for (let i = 0; i < forecastData.list.length; i += 8) {
            dailyForecasts.push(forecastData.list[i]);
        }

        // Take only next 5 days
        dailyForecasts.slice(0, 5).forEach(forecast => {
            const forecastItem = this.createForecastItem(forecast);
            forecastContainer.appendChild(forecastItem);
        });
    }

    createForecastItem(forecast) {
        const div = document.createElement('div');
        div.className = 'forecast-item';

        const date = new Date(forecast.dt * 1000);
        const dayName = date.toLocaleDateString('en-US', { weekday: 'short' });
        const iconCode = forecast.weather[0].icon;

        div.innerHTML = `
            <div class="forecast-day">${dayName}</div>
            <img class="forecast-icon" 
                 src="https://openweathermap.org/img/wn/${iconCode}.png" 
                 alt="${forecast.weather[0].description}">
            <div class="forecast-temps">
                <span class="forecast-high">${Math.round(forecast.main.temp_max)}°</span>
                <span class="forecast-low">${Math.round(forecast.main.temp_min)}°</span>
            </div>
            <div class="forecast-desc">${forecast.weather[0].main}</div>
        `;

        return div;
    }

    addToRecentSearches(cityName) {
        // Remove if already exists
        this.recentSearches = this.recentSearches.filter(city => 
            city.toLowerCase() !== cityName.toLowerCase()
        );

        // Add to beginning
        this.recentSearches.unshift(cityName);

        // Keep only last 5
        this.recentSearches = this.recentSearches.slice(0, 5);

        this.saveRecentSearches();
        this.renderRecentSearches();
    }

    renderRecentSearches() {
        const container = document.getElementById('recent-cities');
        container.innerHTML = '';

        if (this.recentSearches.length === 0) {
            container.innerHTML = '<p class="no-recent">No recent searches</p>';
            return;
        }

        this.recentSearches.forEach(city => {
            const button = document.createElement('button');
            button.className = 'recent-city-btn';
            button.textContent = city;
            button.addEventListener('click', () => {
                this.getWeatherData(city);
            });
            container.appendChild(button);
        });
    }

    showLoading() {
        document.getElementById('loading').style.display = 'flex';
        document.getElementById('weather-content').style.display = 'none';
        document.getElementById('error-message').style.display = 'none';
    }

    hideLoading() {
        document.getElementById('loading').style.display = 'none';
    }

    showError(message) {
        const errorElement = document.getElementById('error-message');
        errorElement.querySelector('p').textContent = message;
        errorElement.style.display = 'block';
        document.getElementById('weather-content').style.display = 'none';
    }

    hideError() {
        document.getElementById('error-message').style.display = 'none';
    }

    formatDateTime(date) {
        const options = {
            weekday: 'long',
            year: 'numeric',
            month: 'long',
            day: 'numeric',
            hour: '2-digit',
            minute: '2-digit'
        };
        return date.toLocaleDateString('en-US', options);
    }

    loadDefaultCity() {
        // Load weather for a default city or user's last searched city
        const lastSearch = this.recentSearches[0];
        if (lastSearch) {
            this.getWeatherData(lastSearch);
        } else {
            this.getWeatherData('London'); // Default city
        }
    }

    // Local Storage Methods
    saveRecentSearches() {
        localStorage.setItem('recentSearches', JSON.stringify(this.recentSearches));
    }

    loadRecentSearches() {
        try {
            const searches = localStorage.getItem('recentSearches');
            return searches ? JSON.parse(searches) : [];
        } catch (error) {
            console.error('Error loading recent searches:', error);
            return [];
        }
    }

    // Search suggestions (simplified version)
    async showSearchSuggestions(query) {
        // In a real app, you might use a geocoding API for suggestions
        // For this example, we'll skip this feature
        console.log('Search suggestions for:', query);
    }
}

// Initialize the weather dashboard
document.addEventListener('DOMContentLoaded', () => {
    new WeatherDashboard();
});

// Service Worker registration for offline functionality (optional)
if ('serviceWorker' in navigator) {
    window.addEventListener('load', () => {
        navigator.serviceWorker.register('/sw.js')
            .then(registration => {
                console.log('SW registered: ', registration);
            })
            .catch(registrationError => {
                console.log('SW registration failed: ', registrationError);
            });
    });
}
```

---

## 🧮 Exercise 3: Advanced JavaScript Patterns

### Task: Build a Calculator with History and Memory Functions

Create a scientific calculator with advanced features.

```javascript
// Advanced Calculator Implementation
class ScientificCalculator {
    constructor() {
        this.currentInput = '0';
        this.previousInput = '';
        this.operator = '';
        this.memory = 0;
        this.history = this.loadHistory();
        this.isNewCalculation = true;
        this.init();
    }

    init() {
        this.createCalculatorUI();
        this.bindEvents();
        this.updateDisplay();
        this.renderHistory();
    }

    createCalculatorUI() {
        const calculatorHTML = `
            <div class="calculator">
                <div class="calculator-header">
                    <h2>Scientific Calculator</h2>
                    <button class="btn-history" id="toggle-history">History</button>
                </div>
                
                <div class="calculator-body">
                    <div class="calculator-main">
                        <div class="display">
                            <div class="display-previous" id="display-previous"></div>
                            <div class="display-current" id="display-current">0</div>
                        </div>
                        
                        <div class="calculator-grid">
                            <!-- Memory Functions -->
                            <button class="btn btn-memory" data-action="memory-clear">MC</button>
                            <button class="btn btn-memory" data-action="memory-recall">MR</button>
                            <button class="btn btn-memory" data-action="memory-add">M+</button>
                            <button class="btn btn-memory" data-action="memory-subtract">M-</button>
                            
                            <!-- Scientific Functions -->
                            <button class="btn btn-function" data-action="sin">sin</button>
                            <button class="btn btn-function" data-action="cos">cos</button>
                            <button class="btn btn-function" data-action="tan">tan</button>
                            <button class="btn btn-function" data-action="log">log</button>
                            
                            <button class="btn btn-function" data-action="ln">ln</button>
                            <button class="btn btn-function" data-action="sqrt">√</button>
                            <button class="btn btn-function" data-action="power">x²</button>
                            <button class="btn btn-function" data-action="factorial">x!</button>
                            
                            <!-- Number Input -->
                            <button class="btn btn-clear" data-action="clear">C</button>
                            <button class="btn btn-clear" data-action="clear-entry">CE</button>
                            <button class="btn btn-operation" data-action="backspace">⌫</button>
                            <button class="btn btn-operation" data-operator="/">&divide;</button>
                            
                            <button class="btn btn-number" data-number="7">7</button>
                            <button class="btn btn-number" data-number="8">8</button>
                            <button class="btn btn-number" data-number="9">9</button>
                            <button class="btn btn-operation" data-operator="*">&times;</button>
                            
                            <button class="btn btn-number" data-number="4">4</button>
                            <button class="btn btn-number" data-number="5">5</button>
                            <button class="btn btn-number" data-number="6">6</button>
                            <button class="btn btn-operation" data-operator="-">&minus;</button>
                            
                            <button class="btn btn-number" data-number="1">1</button>
                            <button class="btn btn-number" data-number="2">2</button>
                            <button class="btn btn-number" data-number="3">3</button>
                            <button class="btn btn-operation" data-operator="+">+</button>
                            
                            <button class="btn btn-number" data-number="0" style="grid-column: span 2;">0</button>
                            <button class="btn btn-number" data-action="decimal">.</button>
                            <button class="btn btn-equals" data-action="equals">=</button>
                        </div>
                    </div>
                    
                    <div class="calculator-history" id="calculator-history">
                        <div class="history-header">
                            <h3>History</h3>
                            <button class="btn-clear-history" id="clear-history">Clear</button>
                        </div>
                        <div class="history-list" id="history-list">
                            <!-- History items will be inserted here -->
                        </div>
                    </div>
                </div>
            </div>
        `;

        document.body.innerHTML = calculatorHTML;
    }

    bindEvents() {
        // Number buttons
        document.querySelectorAll('[data-number]').forEach(button => {
            button.addEventListener('click', (e) => {
                const number = e.target.dataset.number;
                if (number !== undefined) {
                    this.inputNumber(number);
                } else if (e.target.dataset.action === 'decimal') {
                    this.inputDecimal();
                }
            });
        });

        // Operator buttons
        document.querySelectorAll('[data-operator]').forEach(button => {
            button.addEventListener('click', (e) => {
                this.inputOperator(e.target.dataset.operator);
            });
        });

        // Action buttons
        document.querySelectorAll('[data-action]').forEach(button => {
            button.addEventListener('click', (e) => {
                this.handleAction(e.target.dataset.action);
            });
        });

        // History toggle
        document.getElementById('toggle-history').addEventListener('click', () => {
            this.toggleHistory();
        });

        // Clear history
        document.getElementById('clear-history').addEventListener('click', () => {
            this.clearHistory();
        });

        // Keyboard support
        document.addEventListener('keydown', (e) => {
            this.handleKeyboard(e);
        });
    }

    inputNumber(number) {
        if (this.isNewCalculation) {
            this.currentInput = number;
            this.isNewCalculation = false;
        } else {
            this.currentInput = this.currentInput === '0' ? number : this.currentInput + number;
        }
        this.updateDisplay();
    }

    inputDecimal() {
        if (this.isNewCalculation) {
            this.currentInput = '0.';
            this.isNewCalculation = false;
        } else if (!this.currentInput.includes('.')) {
            this.currentInput += '.';
        }
        this.updateDisplay();
    }

    inputOperator(operator) {
        if (this.operator && !this.isNewCalculation) {
            this.calculate();
        }

        this.operator = operator;
        this.previousInput = this.currentInput;
        this.isNewCalculation = true;
        this.updateDisplay();
    }

    handleAction(action) {
        switch (action) {
            case 'equals':
                this.calculate();
                break;
            case 'clear':
                this.clear();
                break;
            case 'clear-entry':
                this.clearEntry();
                break;
            case 'backspace':
                this.backspace();
                break;
            case 'memory-clear':
                this.memoryClear();
                break;
            case 'memory-recall':
                this.memoryRecall();
                break;
            case 'memory-add':
                this.memoryAdd();
                break;
            case 'memory-subtract':
                this.memorySubtract();
                break;
            default:
                this.scientificFunction(action);
                break;
        }
    }

    calculate() {
        if (!this.operator || this.isNewCalculation) return;

        const prev = parseFloat(this.previousInput);
        const current = parseFloat(this.currentInput);
        let result;

        try {
            switch (this.operator) {
                case '+':
                    result = prev + current;
                    break;
                case '-':
                    result = prev - current;
                    break;
                case '*':
                    result = prev * current;
                    break;
                case '/':
                    if (current === 0) {
                        throw new Error('Cannot divide by zero');
                    }
                    result = prev / current;
                    break;
                default:
                    return;
            }

            // Add to history
            const calculation = `${prev} ${this.getOperatorSymbol(this.operator)} ${current} = ${result}`;
            this.addToHistory(calculation);

            this.currentInput = this.formatResult(result);
            this.operator = '';
            this.previousInput = '';
            this.isNewCalculation = true;

        } catch (error) {
            this.currentInput = 'Error';
            this.isNewCalculation = true;
        }

        this.updateDisplay();
    }

    scientificFunction(func) {
        const current = parseFloat(this.currentInput);
        let result;

        try {
            switch (func) {
                case 'sin':
                    result = Math.sin(this.toRadians(current));
                    break;
                case 'cos':
                    result = Math.cos(this.toRadians(current));
                    break;
                case 'tan':
                    result = Math.tan(this.toRadians(current));
                    break;
                case 'log':
                    if (current <= 0) throw new Error('Invalid input for log');
                    result = Math.log10(current);
                    break;
                case 'ln':
                    if (current <= 0) throw new Error('Invalid input for ln');
                    result = Math.log(current);
                    break;
                case 'sqrt':
                    if (current < 0) throw new Error('Invalid input for sqrt');
                    result = Math.sqrt(current);
                    break;
                case 'power':
                    result = Math.pow(current, 2);
                    break;
                case 'factorial':
                    if (current < 0 || !Number.isInteger(current)) {
                        throw new Error('Invalid input for factorial');
                    }
                    result = this.factorial(current);
                    break;
                default:
                    return;
            }

            // Add to history
            const calculation = `${func}(${current}) = ${result}`;
            this.addToHistory(calculation);

            this.currentInput = this.formatResult(result);
            this.isNewCalculation = true;

        } catch (error) {
            this.currentInput = 'Error';
            this.isNewCalculation = true;
        }

        this.updateDisplay();
    }

    // Memory functions
    memoryClear() {
        this.memory = 0;
        this.showMemoryIndicator();
    }

    memoryRecall() {
        this.currentInput = this.formatResult(this.memory);
        this.isNewCalculation = true;
        this.updateDisplay();
    }

    memoryAdd() {
        this.memory += parseFloat(this.currentInput);
        this.showMemoryIndicator();
    }

    memorySubtract() {
        this.memory -= parseFloat(this.currentInput);
        this.showMemoryIndicator();
    }

    showMemoryIndicator() {
        // Visual indicator that memory has a value
        const indicator = document.querySelector('.memory-indicator') || 
                         document.createElement('div');
        indicator.className = 'memory-indicator';
        indicator.textContent = this.memory !== 0 ? 'M' : '';
        
        if (!document.querySelector('.memory-indicator')) {
            document.querySelector('.display').appendChild(indicator);
        }
    }

    // Utility functions
    clear() {
        this.currentInput = '0';
        this.previousInput = '';
        this.operator = '';
        this.isNewCalculation = true;
        this.updateDisplay();
    }

    clearEntry() {
        this.currentInput = '0';
        this.isNewCalculation = true;
        this.updateDisplay();
    }

    backspace() {
        if (this.currentInput.length > 1) {
            this.currentInput = this.currentInput.slice(0, -1);
        } else {
            this.currentInput = '0';
            this.isNewCalculation = true;
        }
        this.updateDisplay();
    }

    formatResult(result) {
        // Handle very large or very small numbers
        if (Math.abs(result) > 1e15 || (Math.abs(result) < 1e-10 && result !== 0)) {
            return result.toExponential(6);
        }
        
        // Round to avoid floating point precision issues
        const rounded = Math.round(result * 1e10) / 1e10;
        return rounded.toString();
    }

    toRadians(degrees) {
        return degrees * (Math.PI / 180);
    }

    factorial(n) {
        if (n === 0 || n === 1) return 1;
        let result = 1;
        for (let i = 2; i <= n; i++) {
            result *= i;
        }
        return result;
    }

    getOperatorSymbol(operator) {
        const symbols = { '+': '+', '-': '−', '*': '×', '/': '÷' };
        return symbols[operator] || operator;
    }

    updateDisplay() {
        document.getElementById('display-current').textContent = this.currentInput;
        
        const previousDisplay = document.getElementById('display-previous');
        if (this.operator && this.previousInput) {
            previousDisplay.textContent = 
                `${this.previousInput} ${this.getOperatorSymbol(this.operator)}`;
        } else {
            previousDisplay.textContent = '';
        }
    }

    // History management
    addToHistory(calculation) {
        const historyItem = {
            calculation,
            timestamp: new Date().toLocaleString()
        };
        
        this.history.unshift(historyItem);
        
        // Keep only last 20 calculations
        this.history = this.history.slice(0, 20);
        
        this.saveHistory();
        this.renderHistory();
    }

    renderHistory() {
        const historyList = document.getElementById('history-list');
        historyList.innerHTML = '';

        if (this.history.length === 0) {
            historyList.innerHTML = '<p class="no-history">No calculations yet</p>';
            return;
        }

        this.history.forEach(item => {
            const historyItem = document.createElement('div');
            historyItem.className = 'history-item';
            historyItem.innerHTML = `
                <div class="history-calculation">${item.calculation}</div>
                <div class="history-time">${item.timestamp}</div>
            `;
            
            // Click to use result
            historyItem.addEventListener('click', () => {
                const result = item.calculation.split(' = ')[1];
                if (result && result !== 'Error') {
                    this.currentInput = result;
                    this.isNewCalculation = true;
                    this.updateDisplay();
                }
            });
            
            historyList.appendChild(historyItem);
        });
    }

    toggleHistory() {
        const historyPanel = document.getElementById('calculator-history');
        historyPanel.classList.toggle('visible');
    }

    clearHistory() {
        this.history = [];
        this.saveHistory();
        this.renderHistory();
    }

    handleKeyboard(event) {
        event.preventDefault();
        
        const key = event.key;
        
        if (key >= '0' && key <= '9') {
            this.inputNumber(key);
        } else if (key === '.') {
            this.inputDecimal();
        } else if (['+', '-', '*', '/'].includes(key)) {
            this.inputOperator(key);
        } else if (key === 'Enter' || key === '=') {
            this.calculate();
        } else if (key === 'Escape') {
            this.clear();
        } else if (key === 'Backspace') {
            this.backspace();
        }
    }

    // Local Storage
    saveHistory() {
        localStorage.setItem('calculatorHistory', JSON.stringify(this.history));
    }

    loadHistory() {
        try {
            const history = localStorage.getItem('calculatorHistory');
            return history ? JSON.parse(history) : [];
        } catch (error) {
            console.error('Error loading history:', error);
            return [];
        }
    }
}

// Initialize calculator when DOM is loaded
document.addEventListener('DOMContentLoaded', () => {
    new ScientificCalculator();
});
```

---

## 🎯 Practice Assignments

### Assignment 1: Expense Tracker
Build an expense tracking app with:
- Add/edit/delete expenses
- Category filtering
- Monthly/yearly summaries
- Data visualization with charts
- Import/export functionality

### Assignment 2: Quiz Application
Create an interactive quiz app featuring:
- Multiple question types (multiple choice, true/false, text input)
- Timer functionality
- Score calculation and feedback
- Progress tracking
- Leaderboard with local storage

### Assignment 3: Image Gallery with Filters
Develop a photo gallery with:
- Lazy loading images
- Filter by categories/tags
- Search functionality
- Lightbox view
- Infinite scroll
- Responsive masonry layout

## 🔍 Self-Assessment Questions

1. How do you handle asynchronous operations in JavaScript?
2. What's the difference between `let`, `const`, and `var`?
3. How do arrow functions differ from regular functions?
4. What are Promises and how do they work?
5. How do you debug JavaScript code effectively?
6. What are the different ways to manipulate the DOM?
7. How do you handle errors in JavaScript applications?

## 📚 Additional Resources

- [MDN JavaScript Guide](https://developer.mozilla.org/en-US/docs/Web/JavaScript/Guide)
- [JavaScript.info](https://javascript.info/) - Modern JavaScript tutorial
- [Eloquent JavaScript](https://eloquentjavascript.net/) - Free online book
- [You Don't Know JS](https://github.com/getify/You-Dont-Know-JS) - Book series
- [JavaScript30](https://javascript30.com/) - 30-day challenge

---

**Next:** Proceed to `advanced.md` for advanced JavaScript patterns and frameworks preparation.
