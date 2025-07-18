
# 🚀 Basic TypeScript

## Table of Contents
- [What is TypeScript?](#what-is-typescript)
- [Setup and Configuration](#setup-and-configuration)
- [Basic Types](#basic-types)
- [Functions](#functions)
- [Interfaces and Objects](#interfaces-and-objects)
- [Classes](#classes)
- [Generics](#generics)
- [Working with React](#working-with-react)
- [Best Practices](#best-practices)

---

## 🎯 What is TypeScript?

**TypeScript** is a statically typed superset of JavaScript that compiles to plain JavaScript.

### Key Benefits:
- **Type Safety**: Catch errors at compile time
- **Better IDE Support**: IntelliSense and autocomplete
- **Code Documentation**: Types serve as documentation
- **Refactoring**: Safer code changes
- **Modern JavaScript**: Latest ES features

### TypeScript vs JavaScript

| Feature | TypeScript | JavaScript |
|---------|------------|------------|
| **Typing** | Static | Dynamic |
| **Compilation** | Required | Not required |
| **Error Detection** | Compile-time | Runtime |
| **IDE Support** | Excellent | Good |

---

## 🛠 Setup and Configuration

### Installation

```bash
# Global installation
npm install -g typescript

# Project installation
npm install --save-dev typescript @types/node

# Initialize TypeScript config
npx tsc --init
```

### tsconfig.json

```json
{
  "compilerOptions": {
    "target": "ES2020",
    "module": "commonjs",
    "lib": ["ES2020", "DOM"],
    "outDir": "./dist",
    "rootDir": "./src",
    "strict": true,
    "esModuleInterop": true,
    "skipLibCheck": true,
    "forceConsistentCasingInFileNames": true,
    "resolveJsonModule": true,
    "declaration": true,
    "declarationMap": true,
    "sourceMap": true
  },
  "include": ["src/**/*"],
  "exclude": ["node_modules", "dist"]
}
```

### Development Workflow

```bash
# Compile TypeScript
npx tsc

# Watch mode
npx tsc --watch

# Run with ts-node
npx ts-node src/index.ts
```

---

## 📝 Basic Types

### Primitive Types

```typescript
// String
let message: string = "Hello, TypeScript!";
let template: string = `Value is ${42}`;

// Number
let age: number = 25;
let price: number = 99.99;
let hex: number = 0xf00d;
let binary: number = 0b1010;

// Boolean
let isActive: boolean = true;
let isComplete: boolean = false;

// Undefined and Null
let notDefined: undefined = undefined;
let empty: null = null;

// Any (avoid when possible)
let dynamic: any = 42;
dynamic = "string";
dynamic = true;

// Unknown (safer than any)
let userInput: unknown = getUserInput();
if (typeof userInput === "string") {
    console.log(userInput.toUpperCase());
}

// Void
function logMessage(): void {
    console.log("This function returns nothing");
}

// Never
function throwError(message: string): never {
    throw new Error(message);
}
```

### Array Types

```typescript
// Array syntax
let numbers: number[] = [1, 2, 3, 4, 5];
let names: string[] = ["Alice", "Bob", "Charlie"];

// Generic array syntax
let scores: Array<number> = [95, 87, 92];
let messages: Array<string> = ["Hello", "World"];

// Mixed arrays with union types
let mixed: (string | number)[] = ["hello", 42, "world", 100];

// Readonly arrays
let readonlyNumbers: readonly number[] = [1, 2, 3];
// readonlyNumbers.push(4); // Error!

// Tuple types
let person: [string, number] = ["Alice", 30];
let point: [number, number] = [10, 20];

// Named tuples
let namedPoint: [x: number, y: number] = [10, 20];
```

### Object Types

```typescript
// Object type annotation
let user: { name: string; age: number } = {
    name: "John",
    age: 30
};

// Optional properties
let optionalUser: { name: string; age?: number } = {
    name: "Jane"
};

// Readonly properties
let readonlyUser: { readonly id: number; name: string } = {
    id: 1,
    name: "Bob"
};
// readonlyUser.id = 2; // Error!

// Index signatures
let scores: { [subject: string]: number } = {
    math: 95,
    science: 87,
    english: 92
};
```

---

## 🔧 Functions

### Function Types

```typescript
// Function declaration
function add(a: number, b: number): number {
    return a + b;
}

// Function expression
const multiply = function(a: number, b: number): number {
    return a * b;
};

// Arrow function
const divide = (a: number, b: number): number => {
    return a / b;
};

// Short arrow function
const square = (x: number): number => x * x;

// Function type annotation
let operation: (x: number, y: number) => number;
operation = add;
operation = multiply;
```

### Optional and Default Parameters

```typescript
// Optional parameters
function greet(name: string, greeting?: string): string {
    return `${greeting || "Hello"}, ${name}!`;
}

console.log(greet("Alice"));           // "Hello, Alice!"
console.log(greet("Bob", "Hi"));       // "Hi, Bob!"

// Default parameters
function createUser(name: string, age: number = 18): object {
    return { name, age };
}

console.log(createUser("Charlie"));         // { name: "Charlie", age: 18 }
console.log(createUser("Diana", 25));       // { name: "Diana", age: 25 }

// Rest parameters
function sum(...numbers: number[]): number {
    return numbers.reduce((total, num) => total + num, 0);
}

console.log(sum(1, 2, 3, 4, 5)); // 15
```

### Function Overloads

```typescript
// Function overloads
function format(value: string): string;
function format(value: number): string;
function format(value: boolean): string;
function format(value: string | number | boolean): string {
    return String(value);
}

console.log(format("hello"));  // "hello"
console.log(format(42));       // "42"
console.log(format(true));     // "true"
```

---

## 🏗 Interfaces and Objects

### Basic Interfaces

```typescript
// Interface definition
interface User {
    id: number;
    name: string;
    email: string;
    age?: number;              // Optional property
    readonly createdAt: Date;  // Readonly property
}

// Using the interface
const user: User = {
    id: 1,
    name: "John Doe",
    email: "john@example.com",
    createdAt: new Date()
};

// Function using interface
function displayUser(user: User): string {
    return `${user.name} (${user.email})`;
}
```

### Extending Interfaces

```typescript
// Base interface
interface Animal {
    name: string;
    age: number;
}

// Extended interface
interface Dog extends Animal {
    breed: string;
    bark(): void;
}

// Multiple inheritance
interface Pet {
    owner: string;
}

interface DomesticDog extends Dog, Pet {
    isVaccinated: boolean;
}

// Implementation
const myDog: DomesticDog = {
    name: "Buddy",
    age: 3,
    breed: "Golden Retriever",
    owner: "Alice",
    isVaccinated: true,
    bark() {
        console.log("Woof!");
    }
};
```

### Method Signatures

```typescript
interface Calculator {
    add(a: number, b: number): number;
    subtract(a: number, b: number): number;
    multiply(a: number, b: number): number;
    divide(a: number, b: number): number;
}

class BasicCalculator implements Calculator {
    add(a: number, b: number): number {
        return a + b;
    }
    
    subtract(a: number, b: number): number {
        return a - b;
    }
    
    multiply(a: number, b: number): number {
        return a * b;
    }
    
    divide(a: number, b: number): number {
        if (b === 0) throw new Error("Division by zero");
        return a / b;
    }
}
```

---

## 🎭 Classes

### Basic Classes

```typescript
class Person {
    // Properties
    private _id: number;
    protected name: string;
    public email: string;
    
    // Constructor
    constructor(id: number, name: string, email: string) {
        this._id = id;
        this.name = name;
        this.email = email;
    }
    
    // Getter
    get id(): number {
        return this._id;
    }
    
    // Method
    public introduce(): string {
        return `Hi, I'm ${this.name}`;
    }
    
    // Protected method
    protected validateEmail(email: string): boolean {
        return email.includes("@");
    }
}

// Usage
const person = new Person(1, "Alice", "alice@example.com");
console.log(person.introduce());
console.log(person.id);
```

### Inheritance

```typescript
class Employee extends Person {
    private salary: number;
    private department: string;
    
    constructor(id: number, name: string, email: string, salary: number, department: string) {
        super(id, name, email);
        this.salary = salary;
        this.department = department;
    }
    
    // Override method
    public introduce(): string {
        return `${super.introduce()}. I work in ${this.department}.`;
    }
    
    // New method
    public getSalaryInfo(): string {
        return `Salary: $${this.salary}`;
    }
}

const employee = new Employee(2, "Bob", "bob@company.com", 75000, "Engineering");
console.log(employee.introduce());
```

### Abstract Classes

```typescript
abstract class Shape {
    protected name: string;
    
    constructor(name: string) {
        this.name = name;
    }
    
    // Abstract method - must be implemented by subclasses
    abstract calculateArea(): number;
    
    // Concrete method
    public getName(): string {
        return this.name;
    }
}

class Rectangle extends Shape {
    private width: number;
    private height: number;
    
    constructor(width: number, height: number) {
        super("Rectangle");
        this.width = width;
        this.height = height;
    }
    
    calculateArea(): number {
        return this.width * this.height;
    }
}

class Circle extends Shape {
    private radius: number;
    
    constructor(radius: number) {
        super("Circle");
        this.radius = radius;
    }
    
    calculateArea(): number {
        return Math.PI * this.radius ** 2;
    }
}
```

---

## 🔄 Generics

### Basic Generics

```typescript
// Generic function
function identity<T>(arg: T): T {
    return arg;
}

// Usage
let stringResult = identity<string>("hello");
let numberResult = identity<number>(42);
let booleanResult = identity<boolean>(true);

// Type inference
let inferredResult = identity("world"); // TypeScript infers string type

// Generic array function
function getFirstElement<T>(arr: T[]): T | undefined {
    return arr.length > 0 ? arr[0] : undefined;
}

let firstString = getFirstElement(["a", "b", "c"]);
let firstNumber = getFirstElement([1, 2, 3]);
```

### Generic Interfaces

```typescript
interface ApiResponse<T> {
    data: T;
    success: boolean;
    message: string;
}

interface User {
    id: number;
    name: string;
    email: string;
}

// Usage
const userResponse: ApiResponse<User> = {
    data: { id: 1, name: "John", email: "john@example.com" },
    success: true,
    message: "User fetched successfully"
};

const usersResponse: ApiResponse<User[]> = {
    data: [
        { id: 1, name: "John", email: "john@example.com" },
        { id: 2, name: "Jane", email: "jane@example.com" }
    ],
    success: true,
    message: "Users fetched successfully"
};
```

### Generic Classes

```typescript
class Container<T> {
    private items: T[] = [];
    
    add(item: T): void {
        this.items.push(item);
    }
    
    get(index: number): T | undefined {
        return this.items[index];
    }
    
    getAll(): T[] {
        return [...this.items];
    }
    
    size(): number {
        return this.items.length;
    }
}

// Usage
const stringContainer = new Container<string>();
stringContainer.add("hello");
stringContainer.add("world");

const numberContainer = new Container<number>();
numberContainer.add(1);
numberContainer.add(2);
numberContainer.add(3);
```

### Generic Constraints

```typescript
// Constraint with extends
interface Lengthwise {
    length: number;
}

function logLength<T extends Lengthwise>(arg: T): T {
    console.log(arg.length);
    return arg;
}

logLength("hello");        // OK - string has length
logLength([1, 2, 3]);      // OK - array has length
// logLength(42);          // Error - number doesn't have length

// Keyof constraint
function getProperty<T, K extends keyof T>(obj: T, key: K): T[K] {
    return obj[key];
}

const person = { name: "Alice", age: 30, email: "alice@example.com" };
const name = getProperty(person, "name");    // string
const age = getProperty(person, "age");      // number
// const invalid = getProperty(person, "invalid"); // Error
```

---

## ⚛️ Working with React

### React Component Types

```typescript
import React, { useState, useEffect } from 'react';

// Props interface
interface ButtonProps {
    text: string;
    onClick: () => void;
    disabled?: boolean;
    variant?: 'primary' | 'secondary' | 'danger';
}

// Functional component
const Button: React.FC<ButtonProps> = ({ 
    text, 
    onClick, 
    disabled = false, 
    variant = 'primary' 
}) => {
    return (
        <button 
            onClick={onClick}
            disabled={disabled}
            className={`btn btn-${variant}`}
        >
            {text}
        </button>
    );
};

// Component with children
interface CardProps {
    title: string;
    children: React.ReactNode;
}

const Card: React.FC<CardProps> = ({ title, children }) => {
    return (
        <div className="card">
            <h3>{title}</h3>
            <div className="card-content">
                {children}
            </div>
        </div>
    );
};
```

### React Hooks with TypeScript

```typescript
interface User {
    id: number;
    name: string;
    email: string;
}

const UserProfile: React.FC = () => {
    // useState with type
    const [user, setUser] = useState<User | null>(null);
    const [loading, setLoading] = useState<boolean>(true);
    const [error, setError] = useState<string>("");

    // useEffect
    useEffect(() => {
        const fetchUser = async () => {
            try {
                setLoading(true);
                const response = await fetch('/api/user');
                const userData: User = await response.json();
                setUser(userData);
            } catch (err) {
                setError(err instanceof Error ? err.message : 'An error occurred');
            } finally {
                setLoading(false);
            }
        };

        fetchUser();
    }, []);

    if (loading) return <div>Loading...</div>;
    if (error) return <div>Error: {error}</div>;
    if (!user) return <div>No user found</div>;

    return (
        <Card title="User Profile">
            <p>Name: {user.name}</p>
            <p>Email: {user.email}</p>
        </Card>
    );
};
```

### Event Handlers

```typescript
const ContactForm: React.FC = () => {
    const [formData, setFormData] = useState({
        name: '',
        email: '',
        message: ''
    });

    // Input change handler
    const handleInputChange = (e: React.ChangeEvent<HTMLInputElement | HTMLTextAreaElement>) => {
        const { name, value } = e.target;
        setFormData(prev => ({
            ...prev,
            [name]: value
        }));
    };

    // Form submit handler
    const handleSubmit = (e: React.FormEvent<HTMLFormElement>) => {
        e.preventDefault();
        console.log('Form submitted:', formData);
    };

    // Button click handler
    const handleReset = (e: React.MouseEvent<HTMLButtonElement>) => {
        e.preventDefault();
        setFormData({ name: '', email: '', message: '' });
    };

    return (
        <form onSubmit={handleSubmit}>
            <input
                type="text"
                name="name"
                value={formData.name}
                onChange={handleInputChange}
                placeholder="Name"
            />
            <input
                type="email"
                name="email"
                value={formData.email}
                onChange={handleInputChange}
                placeholder="Email"
            />
            <textarea
                name="message"
                value={formData.message}
                onChange={handleInputChange}
                placeholder="Message"
            />
            <button type="submit">Submit</button>
            <button type="button" onClick={handleReset}>Reset</button>
        </form>
    );
};
```

---

## ✅ Best Practices

### Type Definitions

```typescript
// Use descriptive type names
type UserRole = 'admin' | 'user' | 'guest';
type ApiStatus = 'loading' | 'success' | 'error';

// Prefer interfaces for object shapes
interface UserPreferences {
    theme: 'light' | 'dark';
    language: string;
    notifications: boolean;
}

// Use type aliases for complex types
type EventHandler<T> = (event: T) => void;
type AsyncOperation<T> = () => Promise<T>;

// Utility types
interface User {
    id: number;
    name: string;
    email: string;
    role: UserRole;
}

// Partial user for updates
type UserUpdate = Partial<User>;

// Pick specific fields
type UserSummary = Pick<User, 'id' | 'name'>;

// Omit sensitive fields
type PublicUser = Omit<User, 'email'>;
```

### Error Handling

```typescript
// Custom error types
class ApiError extends Error {
    constructor(
        message: string,
        public statusCode: number,
        public endpoint: string
    ) {
        super(message);
        this.name = 'ApiError';
    }
}

// Result type for error handling
type Result<T, E = Error> = 
    | { success: true; data: T }
    | { success: false; error: E };

// Safe API function
async function fetchUser(id: number): Promise<Result<User, ApiError>> {
    try {
        const response = await fetch(`/api/users/${id}`);
        
        if (!response.ok) {
            return {
                success: false,
                error: new ApiError(
                    'Failed to fetch user',
                    response.status,
                    `/api/users/${id}`
                )
            };
        }
        
        const user: User = await response.json();
        return { success: true, data: user };
    } catch (error) {
        return {
            success: false,
            error: new ApiError(
                'Network error',
                0,
                `/api/users/${id}`
            )
        };
    }
}

// Usage
const userResult = await fetchUser(1);
if (userResult.success) {
    console.log(userResult.data.name);
} else {
    console.error(userResult.error.message);
}
```

### Configuration and Environment

```typescript
// Environment variables with validation
interface Config {
    apiUrl: string;
    apiKey: string;
    environment: 'development' | 'staging' | 'production';
    features: {
        enableAnalytics: boolean;
        enableChat: boolean;
    };
}

function loadConfig(): Config {
    const config: Config = {
        apiUrl: process.env.REACT_APP_API_URL || 'http://localhost:3000',
        apiKey: process.env.REACT_APP_API_KEY || '',
        environment: (process.env.NODE_ENV as Config['environment']) || 'development',
        features: {
            enableAnalytics: process.env.REACT_APP_ENABLE_ANALYTICS === 'true',
            enableChat: process.env.REACT_APP_ENABLE_CHAT === 'true'
        }
    };

    // Validation
    if (!config.apiKey && config.environment === 'production') {
        throw new Error('API key is required in production');
    }

    return config;
}

export const config = loadConfig();
```

## 📚 Quick Reference

### Essential TypeScript Types

| Type | Description | Example |
|------|-------------|---------|
| `string` | Text data | `let name: string = "John";` |
| `number` | Numeric data | `let age: number = 30;` |
| `boolean` | True/false | `let active: boolean = true;` |
| `Array<T>` | Array of type T | `let nums: number[] = [1, 2, 3];` |
| `object` | Object type | `let user: { name: string } = { name: "John" };` |
| `any` | Any type (avoid) | `let data: any = anything;` |
| `unknown` | Unknown type (safer) | `let input: unknown = userInput;` |
| `void` | No return value | `function log(): void { }` |
| `never` | Never returns | `function error(): never { throw new Error(); }` |

### Common Utility Types

| Type | Purpose | Example |
|------|---------|---------|
| `Partial<T>` | All properties optional | `Partial<User>` |
| `Required<T>` | All properties required | `Required<User>` |
| `Pick<T, K>` | Select specific properties | `Pick<User, 'id' | 'name'>` |
| `Omit<T, K>` | Exclude specific properties | `Omit<User, 'password'>` |
| `Record<K, T>` | Object with specific keys | `Record<string, number>` |

### Next Steps:
1. Practice with real projects
2. Learn advanced TypeScript features
3. Explore type-driven development
4. Study popular TypeScript libraries
5. Master TypeScript with frameworks

This TypeScript guide provides the foundation for type-safe JavaScript development!
