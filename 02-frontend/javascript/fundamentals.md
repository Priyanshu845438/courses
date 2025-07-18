
# 🚀 Basic JavaScript

## Table of Contents
- [What is JavaScript?](#what-is-javascript)
- [Variables and Data Types](#variables-and-data-types)
- [Operators](#operators)
- [Control Structures](#control-structures)
- [Functions](#functions)
- [Arrays](#arrays)
- [Objects](#objects)
- [DOM Manipulation](#dom-manipulation)
- [Events](#events)
- [Best Practices](#best-practices)

---

## 🎯 What is JavaScript?

**JavaScript** is a high-level, interpreted programming language that makes web pages interactive and dynamic.

### Key Features:
- **Client-side scripting**: Runs in the browser
- **Dynamic typing**: Variables can hold different types
- **Event-driven**: Responds to user interactions
- **Object-oriented**: Supports objects and classes
- **Functional programming**: Functions are first-class citizens

### JavaScript vs Other Languages

| Feature | JavaScript | Python | Java |
|---------|------------|---------|------|
| **Typing** | Dynamic | Dynamic | Static |
| **Compilation** | Interpreted | Interpreted | Compiled |
| **Use Case** | Web development | General purpose | Enterprise apps |
| **Syntax** | C-like | English-like | C-like |

---

## 📦 Variables and Data Types

### Variable Declaration

```javascript
// var (old way - avoid)
var name = "John";

// let (block-scoped, can be reassigned)
let age = 25;
age = 26; // OK

// const (block-scoped, cannot be reassigned)
const pi = 3.14159;
// pi = 3.14; // Error!

// const with objects/arrays (contents can change)
const person = { name: "Alice" };
person.name = "Bob"; // OK - changing property
person.age = 30; // OK - adding property
```

### Data Types

#### Primitive Types

```javascript
// Number
let integer = 42;
let decimal = 3.14;
let negative = -10;
let scientific = 2.5e6; // 2,500,000

// String
let singleQuotes = 'Hello';
let doubleQuotes = "World";
let templateLiteral = `Hello ${name}!`;
let multiline = `This is a
multi-line string`;

// Boolean
let isTrue = true;
let isFalse = false;

// Undefined
let notDefined;
console.log(notDefined); // undefined

// Null
let empty = null;

// Symbol (ES6+)
let symbol = Symbol('id');

// BigInt (ES2020+)
let bigNumber = 123456789012345678901234567890n;
```

#### Type Checking

```javascript
// typeof operator
console.log(typeof 42);          // "number"
console.log(typeof "hello");     // "string"
console.log(typeof true);        // "boolean"
console.log(typeof undefined);   // "undefined"
console.log(typeof null);        // "object" (known quirk)
console.log(typeof {});          // "object"
console.log(typeof []);          // "object"
console.log(typeof function(){}); // "function"

// Better array check
console.log(Array.isArray([]));  // true
console.log(Array.isArray({}));  // false
```

### String Methods

```javascript
let text = "Hello, World!";

// Length
console.log(text.length); // 13

// Case conversion
console.log(text.toLowerCase()); // "hello, world!"
console.log(text.toUpperCase()); // "HELLO, WORLD!"

// Substring extraction
console.log(text.charAt(0));       // "H"
console.log(text.substring(0, 5)); // "Hello"
console.log(text.slice(7, 12));    // "World"

// Search
console.log(text.indexOf("World")); // 7
console.log(text.includes("Hello")); // true

// Replace
console.log(text.replace("World", "JavaScript")); // "Hello, JavaScript!"

// Split
console.log("a,b,c".split(",")); // ["a", "b", "c"]

// Trim
console.log("  hello  ".trim()); // "hello"
```

### Number Methods

```javascript
let num = 3.14159;

// Rounding
console.log(Math.round(num));    // 3
console.log(Math.floor(num));    // 3
console.log(Math.ceil(num));     // 4
console.log(num.toFixed(2));     // "3.14"

// Math object
console.log(Math.max(1, 2, 3));  // 3
console.log(Math.min(1, 2, 3));  // 1
console.log(Math.random());      // Random number 0-1
console.log(Math.pow(2, 3));     // 8
console.log(Math.sqrt(16));      // 4

// Number conversion
console.log(Number("123"));      // 123
console.log(parseInt("123px"));  // 123
console.log(parseFloat("3.14")); // 3.14
```

---

## 🔧 Operators

### Arithmetic Operators

```javascript
let a = 10;
let b = 3;

console.log(a + b);  // 13 (Addition)
console.log(a - b);  // 7  (Subtraction)
console.log(a * b);  // 30 (Multiplication)
console.log(a / b);  // 3.333... (Division)
console.log(a % b);  // 1  (Modulus/Remainder)
console.log(a ** b); // 1000 (Exponentiation)

// Increment/Decrement
let count = 5;
console.log(count++); // 5 (post-increment)
console.log(count);   // 6
console.log(++count); // 7 (pre-increment)
console.log(count--); // 7 (post-decrement)
console.log(count);   // 6
```

### Comparison Operators

```javascript
let x = 5;
let y = "5";

// Equality
console.log(x == y);  // true (loose equality)
console.log(x === y); // false (strict equality)
console.log(x != y);  // false (loose inequality)
console.log(x !== y); // true (strict inequality)

// Comparison
console.log(x > 3);   // true
console.log(x < 10);  // true
console.log(x >= 5);  // true
console.log(x <= 4);  // false
```

### Logical Operators

```javascript
let isLoggedIn = true;
let hasPermission = false;

// AND
console.log(isLoggedIn && hasPermission); // false

// OR
console.log(isLoggedIn || hasPermission); // true

// NOT
console.log(!isLoggedIn); // false

// Short-circuit evaluation
let user = null;
let name = user && user.name; // null (doesn't try user.name)
let defaultName = name || "Guest"; // "Guest"
```

### Assignment Operators

```javascript
let num = 10;

num += 5;  // num = num + 5  (15)
num -= 3;  // num = num - 3  (12)
num *= 2;  // num = num * 2  (24)
num /= 4;  // num = num / 4  (6)
num %= 5;  // num = num % 5  (1)
num **= 3; // num = num ** 3 (1)
```

---

## 🎮 Control Structures

### Conditional Statements

```javascript
let score = 85;

// if statement
if (score >= 90) {
    console.log("Grade: A");
} else if (score >= 80) {
    console.log("Grade: B");
} else if (score >= 70) {
    console.log("Grade: C");
} else if (score >= 60) {
    console.log("Grade: D");
} else {
    console.log("Grade: F");
}

// Ternary operator
let status = score >= 60 ? "Pass" : "Fail";
console.log(status);

// Switch statement
let day = "Monday";

switch (day) {
    case "Monday":
        console.log("Start of work week");
        break;
    case "Tuesday":
    case "Wednesday":
    case "Thursday":
        console.log("Midweek");
        break;
    case "Friday":
        console.log("TGIF!");
        break;
    case "Saturday":
    case "Sunday":
        console.log("Weekend!");
        break;
    default:
        console.log("Invalid day");
}
```

### Loops

```javascript
// for loop
for (let i = 0; i < 5; i++) {
    console.log(`Count: ${i}`);
}

// while loop
let count = 0;
while (count < 3) {
    console.log(`While count: ${count}`);
    count++;
}

// do-while loop
let num = 0;
do {
    console.log(`Do-while: ${num}`);
    num++;
} while (num < 2);

// for...of loop (arrays)
let colors = ["red", "green", "blue"];
for (let color of colors) {
    console.log(color);
}

// for...in loop (objects)
let person = { name: "John", age: 30, city: "New York" };
for (let key in person) {
    console.log(`${key}: ${person[key]}`);
}

// break and continue
for (let i = 0; i < 10; i++) {
    if (i === 3) continue; // Skip 3
    if (i === 7) break;    // Stop at 7
    console.log(i);
}
```

---

## 🔧 Functions

### Function Declaration

```javascript
// Function declaration
function greet(name) {
    return `Hello, ${name}!`;
}

// Function expression
let greetExpression = function(name) {
    return `Hello, ${name}!`;
};

// Arrow function (ES6+)
let greetArrow = (name) => {
    return `Hello, ${name}!`;
};

// Arrow function (short syntax)
let greetShort = name => `Hello, ${name}!`;

// Usage
console.log(greet("Alice"));      // "Hello, Alice!"
console.log(greetArrow("Bob"));   // "Hello, Bob!"
console.log(greetShort("Charlie")); // "Hello, Charlie!"
```

### Function Parameters

```javascript
// Default parameters
function greetWithDefault(name = "Guest") {
    return `Hello, ${name}!`;
}

console.log(greetWithDefault());        // "Hello, Guest!"
console.log(greetWithDefault("John"));  // "Hello, John!"

// Rest parameters
function sum(...numbers) {
    return numbers.reduce((total, num) => total + num, 0);
}

console.log(sum(1, 2, 3, 4, 5)); // 15

// Destructuring parameters
function createUser({name, age, email}) {
    return {
        name,
        age,
        email,
        id: Math.random()
    };
}

let user = createUser({
    name: "Alice",
    age: 25,
    email: "alice@example.com"
});
```

### Higher-Order Functions

```javascript
// Function that returns a function
function createMultiplier(factor) {
    return function(number) {
        return number * factor;
    };
}

let double = createMultiplier(2);
let triple = createMultiplier(3);

console.log(double(5)); // 10
console.log(triple(4)); // 12

// Function that takes a function as parameter
function processNumbers(numbers, operation) {
    return numbers.map(operation);
}

let numbers = [1, 2, 3, 4, 5];
let doubled = processNumbers(numbers, x => x * 2);
let squared = processNumbers(numbers, x => x ** 2);

console.log(doubled); // [2, 4, 6, 8, 10]
console.log(squared); // [1, 4, 9, 16, 25]
```

---

## 📊 Arrays

### Array Creation and Access

```javascript
// Array creation
let fruits = ["apple", "banana", "orange"];
let numbers = [1, 2, 3, 4, 5];
let mixed = [1, "hello", true, null];
let empty = [];

// Array constructor
let arr = new Array(3); // [undefined, undefined, undefined]
let arr2 = new Array(1, 2, 3); // [1, 2, 3]

// Access elements
console.log(fruits[0]);    // "apple"
console.log(fruits[2]);    // "orange"
console.log(fruits.length); // 3

// Modify elements
fruits[1] = "grape";
console.log(fruits); // ["apple", "grape", "orange"]
```

### Array Methods

```javascript
let fruits = ["apple", "banana"];

// Add elements
fruits.push("orange");        // Add to end
fruits.unshift("grape");      // Add to beginning
console.log(fruits); // ["grape", "apple", "banana", "orange"]

// Remove elements
let last = fruits.pop();      // Remove from end
let first = fruits.shift();   // Remove from beginning
console.log(last);   // "orange"
console.log(first);  // "grape"
console.log(fruits); // ["apple", "banana"]

// Slice and splice
let numbers = [1, 2, 3, 4, 5];
let sliced = numbers.slice(1, 4);    // [2, 3, 4] (doesn't modify original)
let spliced = numbers.splice(2, 2);  // [3, 4] (modifies original)
console.log(numbers); // [1, 2, 5]

// Join and split
let joined = fruits.join(", ");      // "apple, banana"
let split = "a,b,c".split(",");      // ["a", "b", "c"]

// Search
let index = fruits.indexOf("banana"); // 1
let found = fruits.includes("apple"); // true
```

### Array Iteration

```javascript
let numbers = [1, 2, 3, 4, 5];

// forEach
numbers.forEach(function(num, index) {
    console.log(`Index ${index}: ${num}`);
});

// map (creates new array)
let doubled = numbers.map(num => num * 2);
console.log(doubled); // [2, 4, 6, 8, 10]

// filter (creates new array)
let evens = numbers.filter(num => num % 2 === 0);
console.log(evens); // [2, 4]

// reduce
let sum = numbers.reduce((total, num) => total + num, 0);
console.log(sum); // 15

// find
let found = numbers.find(num => num > 3);
console.log(found); // 4

// some and every
let hasEven = numbers.some(num => num % 2 === 0);  // true
let allPositive = numbers.every(num => num > 0);   // true
```

---

## 🏗️ Objects

### Object Creation

```javascript
// Object literal
let person = {
    name: "John",
    age: 30,
    city: "New York",
    isEmployed: true
};

// Constructor function
function Person(name, age) {
    this.name = name;
    this.age = age;
    this.greet = function() {
        return `Hello, I'm ${this.name}`;
    };
}

let john = new Person("John", 30);

// Object.create()
let personPrototype = {
    greet: function() {
        return `Hello, I'm ${this.name}`;
    }
};

let alice = Object.create(personPrototype);
alice.name = "Alice";
alice.age = 25;
```

### Object Properties

```javascript
let person = {
    name: "John",
    age: 30,
    "full name": "John Doe"
};

// Dot notation
console.log(person.name); // "John"
person.age = 31;

// Bracket notation
console.log(person["full name"]); // "John Doe"
let property = "name";
console.log(person[property]); // "John"

// Add new properties
person.email = "john@example.com";
person["phone"] = "123-456-7890";

// Delete properties
delete person.age;
console.log(person); // {name: "John", "full name": "John Doe", email: "john@example.com", phone: "123-456-7890"}
```

### Object Methods

```javascript
let person = {
    name: "John",
    age: 30,
    
    // Method
    greet: function() {
        return `Hello, I'm ${this.name}`;
    },
    
    // Arrow function (be careful with 'this')
    getAge: () => {
        return this.age; // 'this' doesn't work as expected
    },
    
    // ES6 method syntax
    introduce() {
        return `I'm ${this.name} and I'm ${this.age} years old`;
    }
};

console.log(person.greet());      // "Hello, I'm John"
console.log(person.introduce());  // "I'm John and I'm 30 years old"
```

### Object Utilities

```javascript
let person = {
    name: "John",
    age: 30,
    city: "New York"
};

// Object.keys()
let keys = Object.keys(person);
console.log(keys); // ["name", "age", "city"]

// Object.values()
let values = Object.values(person);
console.log(values); // ["John", 30, "New York"]

// Object.entries()
let entries = Object.entries(person);
console.log(entries); // [["name", "John"], ["age", 30], ["city", "New York"]]

// Object.assign()
let contact = { email: "john@example.com", phone: "123-456-7890" };
let fullPerson = Object.assign({}, person, contact);
console.log(fullPerson);

// Spread operator (ES6+)
let fullPersonSpread = { ...person, ...contact };
console.log(fullPersonSpread);

// Check if property exists
console.log("name" in person);           // true
console.log(person.hasOwnProperty("age")); // true
```

---

## 🌐 DOM Manipulation

### Selecting Elements

```javascript
// Select by ID
let element = document.getElementById("myId");

// Select by class
let elements = document.getElementsByClassName("myClass");
let element = document.querySelector(".myClass");
let allElements = document.querySelectorAll(".myClass");

// Select by tag
let paragraphs = document.getElementsByTagName("p");
let firstP = document.querySelector("p");
let allPs = document.querySelectorAll("p");

// Advanced selectors
let element = document.querySelector("#myId .myClass");
let elements = document.querySelectorAll("div > p");
```

### Modifying Elements

```javascript
// Get/Set content
let element = document.getElementById("myDiv");
console.log(element.innerHTML);    // Get HTML content
console.log(element.textContent);  // Get text content

element.innerHTML = "<strong>Bold text</strong>";
element.textContent = "Plain text";

// Get/Set attributes
console.log(element.getAttribute("class"));
element.setAttribute("class", "newClass");
element.removeAttribute("data-old");

// Modern attribute access
console.log(element.id);
element.id = "newId";
console.log(element.className);
element.className = "class1 class2";

// Style manipulation
element.style.color = "red";
element.style.backgroundColor = "yellow";
element.style.fontSize = "16px";

// CSS classes
element.classList.add("newClass");
element.classList.remove("oldClass");
element.classList.toggle("active");
console.log(element.classList.contains("active"));
```

### Creating and Removing Elements

```javascript
// Create elements
let div = document.createElement("div");
let text = document.createTextNode("Hello World");

// Set properties
div.id = "newDiv";
div.className = "container";
div.innerHTML = "<p>This is a paragraph</p>";

// Append to DOM
document.body.appendChild(div);

// Insert at specific position
let parent = document.getElementById("parent");
let newChild = document.createElement("span");
let firstChild = parent.firstElementChild;
parent.insertBefore(newChild, firstChild);

// Remove elements
let elementToRemove = document.getElementById("removeMe");
elementToRemove.remove(); // Modern way
// elementToRemove.parentNode.removeChild(elementToRemove); // Old way
```

---

## 🎯 Events

### Event Listeners

```javascript
// Add event listener
let button = document.getElementById("myButton");

button.addEventListener("click", function(event) {
    console.log("Button clicked!");
    console.log(event.target);
});

// Arrow function
button.addEventListener("click", (event) => {
    console.log("Button clicked with arrow function!");
});

// Named function
function handleClick(event) {
    console.log("Button clicked with named function!");
}
button.addEventListener("click", handleClick);

// Remove event listener
button.removeEventListener("click", handleClick);
```

### Common Events

```javascript
// Mouse events
element.addEventListener("click", handleClick);
element.addEventListener("dblclick", handleDoubleClick);
element.addEventListener("mousedown", handleMouseDown);
element.addEventListener("mouseup", handleMouseUp);
element.addEventListener("mouseover", handleMouseOver);
element.addEventListener("mouseout", handleMouseOut);

// Keyboard events
document.addEventListener("keydown", function(event) {
    console.log(`Key pressed: ${event.key}`);
    console.log(`Key code: ${event.keyCode}`);
});

document.addEventListener("keyup", function(event) {
    console.log(`Key released: ${event.key}`);
});

// Form events
let form = document.getElementById("myForm");
let input = document.getElementById("myInput");

form.addEventListener("submit", function(event) {
    event.preventDefault(); // Prevent form submission
    console.log("Form submitted");
});

input.addEventListener("focus", function() {
    console.log("Input focused");
});

input.addEventListener("blur", function() {
    console.log("Input lost focus");
});

input.addEventListener("input", function(event) {
    console.log(`Input value: ${event.target.value}`);
});

// Window events
window.addEventListener("load", function() {
    console.log("Page loaded");
});

window.addEventListener("resize", function() {
    console.log("Window resized");
});
```

### Event Object

```javascript
button.addEventListener("click", function(event) {
    // Event properties
    console.log(event.type);        // "click"
    console.log(event.target);      // Element that triggered the event
    console.log(event.currentTarget); // Element that the event listener is attached to
    console.log(event.clientX);     // Mouse X coordinate
    console.log(event.clientY);     // Mouse Y coordinate
    console.log(event.timeStamp);   // Time when event occurred
    
    // Event methods
    event.preventDefault();         // Prevent default behavior
    event.stopPropagation();       // Stop event bubbling
});
```

---

## 💡 Best Practices

### Code Organization

```javascript
// Use meaningful variable names
let userAge = 25;              // Good
let a = 25;                    // Bad

// Use const for values that don't change
const PI = 3.14159;
const CONFIG = {
    API_URL: "https://api.example.com",
    TIMEOUT: 5000
};

// Use let for variables that change
let currentUser = null;
let score = 0;

// Function naming
function calculateTotal(items) {    // Good - verb + noun
    return items.reduce((sum, item) => sum + item.price, 0);
}

function total(items) {            // Less clear
    return items.reduce((sum, item) => sum + item.price, 0);
}
```

### Error Handling

```javascript
// Try-catch for error handling
try {
    let result = riskyOperation();
    console.log(result);
} catch (error) {
    console.error("An error occurred:", error.message);
} finally {
    console.log("This always runs");
}

// Input validation
function divide(a, b) {
    if (typeof a !== "number" || typeof b !== "number") {
        throw new Error("Both parameters must be numbers");
    }
    
    if (b === 0) {
        throw new Error("Cannot divide by zero");
    }
    
    return a / b;
}

// Safe DOM manipulation
let element = document.getElementById("myElement");
if (element) {
    element.textContent = "New content";
} else {
    console.warn("Element not found");
}
```

### Performance Tips

```javascript
// Cache DOM queries
let button = document.getElementById("myButton"); // Cache this
for (let i = 0; i < 100; i++) {
    button.textContent = `Count: ${i}`;
}

// Instead of:
for (let i = 0; i < 100; i++) {
    document.getElementById("myButton").textContent = `Count: ${i}`; // Repeated query
}

// Use event delegation
document.addEventListener("click", function(event) {
    if (event.target.matches(".button")) {
        handleButtonClick(event.target);
    }
});

// Instead of adding listeners to each button:
// let buttons = document.querySelectorAll(".button");
// buttons.forEach(button => {
//     button.addEventListener("click", handleButtonClick);
// });
```

## 📚 Complete Example

```html
<!DOCTYPE html>
<html lang="en">
<head>
    <meta charset="UTF-8">
    <meta name="viewport" content="width=device-width, initial-scale=1.0">
    <title>JavaScript Todo App</title>
    <style>
        body {
            font-family: Arial, sans-serif;
            max-width: 600px;
            margin: 0 auto;
            padding: 20px;
        }
        .todo-item {
            display: flex;
            justify-content: space-between;
            align-items: center;
            padding: 10px;
            border: 1px solid #ddd;
            margin: 5px 0;
            border-radius: 5px;
        }
        .todo-item.completed {
            background-color: #f0f0f0;
            text-decoration: line-through;
        }
        button {
            padding: 5px 10px;
            margin: 0 2px;
            cursor: pointer;
        }
        input[type="text"] {
            padding: 10px;
            width: 70%;
            margin-right: 10px;
        }
    </style>
</head>
<body>
    <h1>JavaScript Todo App</h1>
    
    <div>
        <input type="text" id="todoInput" placeholder="Enter a new todo...">
        <button onclick="addTodo()">Add Todo</button>
    </div>
    
    <div id="todoList"></div>
    
    <script>
        // Todo app state
        let todos = [];
        let todoIdCounter = 1;
        
        // Todo class
        class Todo {
            constructor(text) {
                this.id = todoIdCounter++;
                this.text = text;
                this.completed = false;
                this.createdAt = new Date();
            }
            
            toggle() {
                this.completed = !this.completed;
            }
        }
        
        // Add new todo
        function addTodo() {
            const input = document.getElementById('todoInput');
            const text = input.value.trim();
            
            if (text === '') {
                alert('Please enter a todo text');
                return;
            }
            
            const todo = new Todo(text);
            todos.push(todo);
            input.value = '';
            
            renderTodos();
        }
        
        // Toggle todo completion
        function toggleTodo(id) {
            const todo = todos.find(t => t.id === id);
            if (todo) {
                todo.toggle();
                renderTodos();
            }
        }
        
        // Delete todo
        function deleteTodo(id) {
            todos = todos.filter(t => t.id !== id);
            renderTodos();
        }
        
        // Render todos to DOM
        function renderTodos() {
            const todoList = document.getElementById('todoList');
            todoList.innerHTML = '';
            
            todos.forEach(todo => {
                const todoElement = document.createElement('div');
                todoElement.className = `todo-item ${todo.completed ? 'completed' : ''}`;
                
                todoElement.innerHTML = `
                    <span>${todo.text}</span>
                    <div>
                        <button onclick="toggleTodo(${todo.id})">
                            ${todo.completed ? 'Undo' : 'Complete'}
                        </button>
                        <button onclick="deleteTodo(${todo.id})">Delete</button>
                    </div>
                `;
                
                todoList.appendChild(todoElement);
            });
            
            // Update statistics
            updateStats();
        }
        
        // Update statistics
        function updateStats() {
            const total = todos.length;
            const completed = todos.filter(t => t.completed).length;
            const remaining = total - completed;
            
            console.log(`Total: ${total}, Completed: ${completed}, Remaining: ${remaining}`);
        }
        
        // Handle Enter key in input
        document.getElementById('todoInput').addEventListener('keypress', function(event) {
            if (event.key === 'Enter') {
                addTodo();
            }
        });
        
        // Load sample todos
        function loadSampleTodos() {
            const sampleTodos = [
                'Learn JavaScript basics',
                'Practice DOM manipulation',
                'Build a todo app',
                'Learn about events'
            ];
            
            sampleTodos.forEach(text => {
                todos.push(new Todo(text));
            });
            
            renderTodos();
        }
        
        // Initialize app
        document.addEventListener('DOMContentLoaded', function() {
            loadSampleTodos();
            console.log('Todo app initialized!');
        });
    </script>
</body>
</html>
```

## 🎯 Quick Reference

### Essential Methods

| Method | Description | Example |
|--------|-------------|---------|
| `console.log()` | Print to console | `console.log("Hello");` |
| `typeof` | Check data type | `typeof 42` |
| `parseInt()` | Parse integer | `parseInt("123")` |
| `parseFloat()` | Parse float | `parseFloat("3.14")` |
| `isNaN()` | Check if Not a Number | `isNaN("hello")` |
| `Array.isArray()` | Check if array | `Array.isArray([])` |

### Common Patterns

```javascript
// Check if element exists
if (element) {
    // Do something
}

// Default value
let name = user.name || "Guest";

// Convert to number
let num = +stringValue;
let num = Number(stringValue);

// Convert to string
let str = "" + numberValue;
let str = String(numberValue);

// Check if array has items
if (array.length > 0) {
    // Process array
}
```

This comprehensive guide covers all JavaScript fundamentals. Practice these concepts by building small projects and experimenting with the code examples!
