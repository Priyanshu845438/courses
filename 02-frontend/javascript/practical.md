
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
