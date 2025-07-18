
# 🚀 Basic Node.js

## Table of Contents
- [What is Node.js?](#what-is-nodejs)
- [Installation and Setup](#installation-and-setup)
- [Node.js Fundamentals](#nodejs-fundamentals)
- [Modules and NPM](#modules-and-npm)
- [File System Operations](#file-system-operations)
- [HTTP Server](#http-server)
- [Express.js Basics](#expressjs-basics)
- [Working with JSON](#working-with-json)
- [Environment Variables](#environment-variables)
- [Best Practices](#best-practices)

---

## 🎯 What is Node.js?

**Node.js** is a JavaScript runtime built on Chrome's V8 JavaScript engine that allows you to run JavaScript on the server-side.

### Key Features:
- **Server-side JavaScript**: Run JS outside the browser
- **Event-driven**: Non-blocking, asynchronous programming
- **NPM**: Vast package ecosystem
- **Cross-platform**: Works on Windows, macOS, Linux
- **Fast**: Built on V8 engine for high performance

### Node.js vs Traditional Server Technologies

| Feature | Node.js | PHP | Python |
|---------|---------|-----|--------|
| **Language** | JavaScript | PHP | Python |
| **Concurrency** | Event-driven | Thread-based | Thread-based |
| **Performance** | High (V8) | Moderate | Moderate |
| **Learning Curve** | Easy (if you know JS) | Easy | Easy |

---

## 🛠 Installation and Setup

### Installing Node.js

1. **Download from official website**: [nodejs.org](https://nodejs.org)
2. **Using Package Managers**:

```bash
# macOS with Homebrew
brew install node

# Ubuntu/Debian
sudo apt update
sudo apt install nodejs npm

# Windows with Chocolatey
choco install nodejs

# Using Node Version Manager (Recommended)
curl -o- https://raw.githubusercontent.com/nvm-sh/nvm/v0.39.0/install.sh | bash
nvm install node
nvm use node
```

### Verify Installation

```bash
node --version    # Check Node.js version
npm --version     # Check NPM version
```

### Project Setup

```bash
# Create a new directory
mkdir my-node-app
cd my-node-app

# Initialize a new Node.js project
npm init -y

# This creates package.json file
```

---

## 🏗 Node.js Fundamentals

### Running JavaScript Files

```javascript
// hello.js
console.log("Hello, Node.js!");
console.log("Current directory:", __dirname);
console.log("Current file:", __filename);
console.log("Node.js version:", process.version);
```

```bash
# Run the file
node hello.js
```

### Global Objects

```javascript
// globals.js
console.log("=== Global Objects ===");

// Process object
console.log("Platform:", process.platform);
console.log("Arguments:", process.argv);
console.log("Environment:", process.env.NODE_ENV);

// Console methods
console.log("Regular log");
console.error("Error message");
console.warn("Warning message");
console.time("Timer");
console.timeEnd("Timer");

// Timers
setTimeout(() => {
    console.log("Executed after 1 second");
}, 1000);

setInterval(() => {
    console.log("Executed every 2 seconds");
}, 2000);

// Clear interval after 10 seconds
setTimeout(() => {
    process.exit(0);
}, 10000);
```

### Understanding Event Loop

```javascript
// eventloop.js
console.log("Start");

// Immediate execution
setImmediate(() => {
    console.log("setImmediate");
});

// Next tick (highest priority)
process.nextTick(() => {
    console.log("nextTick");
});

// Timer
setTimeout(() => {
    console.log("setTimeout");
}, 0);

// Promises
Promise.resolve().then(() => {
    console.log("Promise");
});

console.log("End");

// Output order: Start, End, nextTick, Promise, setTimeout, setImmediate
```

---

## 📦 Modules and NPM

### Creating Modules

```javascript
// math.js - Module export
function add(a, b) {
    return a + b;
}

function subtract(a, b) {
    return a - b;
}

function multiply(a, b) {
    return a * b;
}

function divide(a, b) {
    if (b === 0) {
        throw new Error("Division by zero");
    }
    return a / b;
}

// Export individual functions
module.exports = {
    add,
    subtract,
    multiply,
    divide
};

// Alternative export syntax
// exports.add = add;
// exports.subtract = subtract;
```

```javascript
// calculator.js - Using modules
const math = require('./math');

console.log("Addition:", math.add(5, 3));
console.log("Subtraction:", math.subtract(5, 3));
console.log("Multiplication:", math.multiply(5, 3));
console.log("Division:", math.divide(5, 3));

// Destructuring import
const { add, multiply } = require('./math');
console.log("Using destructured functions:", add(10, 5), multiply(10, 5));
```

### ES6 Modules (Modern Syntax)

```javascript
// math-es6.js
export function add(a, b) {
    return a + b;
}

export function subtract(a, b) {
    return a - b;
}

export default function multiply(a, b) {
    return a * b;
}
```

```javascript
// calculator-es6.js
import multiply, { add, subtract } from './math-es6.js';

console.log(add(5, 3));
console.log(subtract(5, 3));
console.log(multiply(5, 3));
```

To use ES6 modules, add to package.json:
```json
{
  "type": "module"
}
```

### NPM Package Management

```bash
# Install packages
npm install express          # Production dependency
npm install -g nodemon      # Global installation
npm install --save-dev jest # Development dependency

# Package.json scripts
npm run start
npm run dev
npm run test

# Other useful commands
npm list                    # List installed packages
npm outdated               # Check for outdated packages
npm update                 # Update packages
npm uninstall package-name # Remove packages
```

### Common Useful Packages

```javascript
// package.json
{
  "dependencies": {
    "express": "^4.18.0",      // Web framework
    "mongoose": "^7.0.0",      // MongoDB ODM
    "dotenv": "^16.0.0",       // Environment variables
    "cors": "^2.8.5",          // Cross-origin requests
    "bcryptjs": "^2.4.3",      // Password hashing
    "jsonwebtoken": "^9.0.0",  // JWT tokens
    "axios": "^1.3.0"          // HTTP client
  },
  "devDependencies": {
    "nodemon": "^2.0.20",      // Auto-restart server
    "jest": "^29.0.0",         // Testing framework
    "supertest": "^6.3.0"      // API testing
  }
}
```

---

## 📁 File System Operations

### Reading and Writing Files

```javascript
// filesystem.js
const fs = require('fs');
const path = require('path');

// Synchronous file operations (blocking)
try {
    // Write file
    fs.writeFileSync('example.txt', 'Hello, Node.js!');
    console.log('File written successfully');
    
    // Read file
    const data = fs.readFileSync('example.txt', 'utf8');
    console.log('File content:', data);
    
    // Check if file exists
    if (fs.existsSync('example.txt')) {
        console.log('File exists');
    }
} catch (error) {
    console.error('Error:', error.message);
}

// Asynchronous file operations (non-blocking)
fs.writeFile('async-example.txt', 'Async file content', (err) => {
    if (err) {
        console.error('Error writing file:', err);
        return;
    }
    console.log('Async file written');
    
    // Read the file
    fs.readFile('async-example.txt', 'utf8', (err, data) => {
        if (err) {
            console.error('Error reading file:', err);
            return;
        }
        console.log('Async file content:', data);
    });
});

// Promise-based file operations
const fsPromises = require('fs').promises;

async function fileOperations() {
    try {
        await fsPromises.writeFile('promise-example.txt', 'Promise-based content');
        const data = await fsPromises.readFile('promise-example.txt', 'utf8');
        console.log('Promise file content:', data);
        
        // Get file stats
        const stats = await fsPromises.stat('promise-example.txt');
        console.log('File size:', stats.size, 'bytes');
        console.log('Created:', stats.birthtime);
        console.log('Modified:', stats.mtime);
    } catch (error) {
        console.error('Promise error:', error.message);
    }
}

fileOperations();
```

### Working with Directories

```javascript
// directories.js
const fs = require('fs').promises;
const path = require('path');

async function directoryOperations() {
    try {
        // Create directory
        await fs.mkdir('test-folder', { recursive: true });
        console.log('Directory created');
        
        // List directory contents
        const files = await fs.readdir('.');
        console.log('Current directory contents:', files);
        
        // Create subdirectory and file
        await fs.mkdir('test-folder/subfolder', { recursive: true });
        await fs.writeFile('test-folder/test-file.txt', 'Test content');
        
        // Walk through directory
        async function walkDirectory(dirPath) {
            const items = await fs.readdir(dirPath);
            
            for (const item of items) {
                const fullPath = path.join(dirPath, item);
                const stats = await fs.stat(fullPath);
                
                if (stats.isDirectory()) {
                    console.log(`Directory: ${fullPath}`);
                    await walkDirectory(fullPath); // Recursive
                } else {
                    console.log(`File: ${fullPath} (${stats.size} bytes)`);
                }
            }
        }
        
        await walkDirectory('test-folder');
        
    } catch (error) {
        console.error('Directory error:', error.message);
    }
}

directoryOperations();
```

---

## 🌐 HTTP Server

### Basic HTTP Server

```javascript
// server.js
const http = require('http');
const url = require('url');

const server = http.createServer((req, res) => {
    const parsedUrl = url.parse(req.url, true);
    const path = parsedUrl.pathname;
    const query = parsedUrl.query;
    const method = req.method;
    
    // Set response headers
    res.setHeader('Content-Type', 'application/json');
    res.setHeader('Access-Control-Allow-Origin', '*');
    
    console.log(`${method} ${path}`);
    
    // Route handling
    if (path === '/' && method === 'GET') {
        res.statusCode = 200;
        res.end(JSON.stringify({
            message: 'Welcome to Node.js server!',
            timestamp: new Date().toISOString()
        }));
    } else if (path === '/api/users' && method === 'GET') {
        res.statusCode = 200;
        res.end(JSON.stringify({
            users: [
                { id: 1, name: 'John Doe', email: 'john@example.com' },
                { id: 2, name: 'Jane Smith', email: 'jane@example.com' }
            ]
        }));
    } else if (path === '/api/user' && method === 'GET') {
        const userId = query.id;
        if (!userId) {
            res.statusCode = 400;
            res.end(JSON.stringify({ error: 'User ID is required' }));
            return;
        }
        
        res.statusCode = 200;
        res.end(JSON.stringify({
            user: { id: userId, name: `User ${userId}` }
        }));
    } else {
        res.statusCode = 404;
        res.end(JSON.stringify({ error: 'Not Found' }));
    }
});

const PORT = process.env.PORT || 3000;
server.listen(PORT, () => {
    console.log(`Server running on port ${PORT}`);
    console.log(`Visit: http://localhost:${PORT}`);
});
```

### Handling POST Requests

```javascript
// post-server.js
const http = require('http');

const server = http.createServer((req, res) => {
    if (req.method === 'POST' && req.url === '/api/data') {
        let body = '';
        
        // Collect data chunks
        req.on('data', chunk => {
            body += chunk.toString();
        });
        
        // Process complete data
        req.on('end', () => {
            try {
                const data = JSON.parse(body);
                console.log('Received data:', data);
                
                // Process the data
                const response = {
                    message: 'Data received successfully',
                    received: data,
                    timestamp: new Date().toISOString()
                };
                
                res.setHeader('Content-Type', 'application/json');
                res.statusCode = 200;
                res.end(JSON.stringify(response));
                
            } catch (error) {
                res.statusCode = 400;
                res.end(JSON.stringify({ error: 'Invalid JSON' }));
            }
        });
    } else {
        res.statusCode = 404;
        res.end(JSON.stringify({ error: 'Not Found' }));
    }
});

server.listen(3000, () => {
    console.log('POST server running on port 3000');
});
```

---

## 🚀 Express.js Basics

### Basic Express Server

```javascript
// app.js
const express = require('express');
const app = express();

// Middleware
app.use(express.json()); // Parse JSON bodies
app.use(express.urlencoded({ extended: true })); // Parse URL-encoded bodies

// Static files
app.use(express.static('public'));

// Basic routes
app.get('/', (req, res) => {
    res.json({
        message: 'Welcome to Express!',
        timestamp: new Date().toISOString()
    });
});

app.get('/api/users', (req, res) => {
    const users = [
        { id: 1, name: 'John Doe', email: 'john@example.com' },
        { id: 2, name: 'Jane Smith', email: 'jane@example.com' },
        { id: 3, name: 'Bob Johnson', email: 'bob@example.com' }
    ];
    res.json(users);
});

// Route with parameters
app.get('/api/users/:id', (req, res) => {
    const userId = parseInt(req.params.id);
    const user = { id: userId, name: `User ${userId}`, email: `user${userId}@example.com` };
    res.json(user);
});

// POST route
app.post('/api/users', (req, res) => {
    const { name, email } = req.body;
    
    if (!name || !email) {
        return res.status(400).json({ error: 'Name and email are required' });
    }
    
    const newUser = {
        id: Date.now(),
        name,
        email,
        created: new Date().toISOString()
    };
    
    res.status(201).json(newUser);
});

// Error handling middleware
app.use((err, req, res, next) => {
    console.error(err.stack);
    res.status(500).json({ error: 'Something went wrong!' });
});

// 404 handler
app.use((req, res) => {
    res.status(404).json({ error: 'Route not found' });
});

const PORT = process.env.PORT || 3000;
app.listen(PORT, () => {
    console.log(`Express server running on port ${PORT}`);
});
```

### Express Middleware

```javascript
// middleware.js
const express = require('express');
const app = express();

// Custom middleware
const logger = (req, res, next) => {
    console.log(`${new Date().toISOString()} - ${req.method} ${req.url}`);
    next(); // Continue to next middleware
};

const authenticate = (req, res, next) => {
    const token = req.headers.authorization;
    
    if (!token) {
        return res.status(401).json({ error: 'Authentication required' });
    }
    
    // Simple token validation (in real app, use proper JWT validation)
    if (token === 'Bearer valid-token') {
        req.user = { id: 1, name: 'Authenticated User' };
        next();
    } else {
        res.status(403).json({ error: 'Invalid token' });
    }
};

// Apply middleware
app.use(logger);
app.use(express.json());

// Public routes
app.get('/public', (req, res) => {
    res.json({ message: 'This is a public route' });
});

// Protected routes
app.use('/api/protected', authenticate);

app.get('/api/protected/profile', (req, res) => {
    res.json({
        message: 'Protected route accessed',
        user: req.user
    });
});

app.listen(3000, () => {
    console.log('Middleware server running on port 3000');
});
```

---

## 📄 Working with JSON

### JSON Operations

```javascript
// json-operations.js
const fs = require('fs').promises;

// Sample data
const users = [
    { id: 1, name: 'John Doe', email: 'john@example.com', age: 30 },
    { id: 2, name: 'Jane Smith', email: 'jane@example.com', age: 25 },
    { id: 3, name: 'Bob Johnson', email: 'bob@example.com', age: 35 }
];

async function jsonOperations() {
    try {
        // Write JSON to file
        await fs.writeFile('users.json', JSON.stringify(users, null, 2));
        console.log('JSON file created');
        
        // Read JSON from file
        const data = await fs.readFile('users.json', 'utf8');
        const parsedUsers = JSON.parse(data);
        console.log('Parsed users:', parsedUsers);
        
        // JSON manipulation
        const updatedUsers = parsedUsers.map(user => ({
            ...user,
            fullInfo: `${user.name} (${user.age} years old)`
        }));
        
        // Filter users
        const youngUsers = updatedUsers.filter(user => user.age < 30);
        console.log('Young users:', youngUsers);
        
        // Save filtered data
        await fs.writeFile('young-users.json', JSON.stringify(youngUsers, null, 2));
        
    } catch (error) {
        console.error('JSON operation error:', error.message);
    }
}

jsonOperations();
```

### JSON API with Express

```javascript
// json-api.js
const express = require('express');
const fs = require('fs').promises;
const path = require('path');

const app = express();
app.use(express.json());

const dataFile = 'data.json';

// Helper function to read data
async function readData() {
    try {
        const data = await fs.readFile(dataFile, 'utf8');
        return JSON.parse(data);
    } catch (error) {
        return { users: [], nextId: 1 };
    }
}

// Helper function to write data
async function writeData(data) {
    await fs.writeFile(dataFile, JSON.stringify(data, null, 2));
}

// Get all users
app.get('/api/users', async (req, res) => {
    try {
        const data = await readData();
        res.json(data.users);
    } catch (error) {
        res.status(500).json({ error: 'Failed to read users' });
    }
});

// Get user by ID
app.get('/api/users/:id', async (req, res) => {
    try {
        const data = await readData();
        const user = data.users.find(u => u.id === parseInt(req.params.id));
        
        if (!user) {
            return res.status(404).json({ error: 'User not found' });
        }
        
        res.json(user);
    } catch (error) {
        res.status(500).json({ error: 'Failed to read user' });
    }
});

// Create new user
app.post('/api/users', async (req, res) => {
    try {
        const { name, email, age } = req.body;
        
        if (!name || !email || !age) {
            return res.status(400).json({ error: 'Name, email, and age are required' });
        }
        
        const data = await readData();
        const newUser = {
            id: data.nextId,
            name,
            email,
            age,
            created: new Date().toISOString()
        };
        
        data.users.push(newUser);
        data.nextId++;
        
        await writeData(data);
        res.status(201).json(newUser);
        
    } catch (error) {
        res.status(500).json({ error: 'Failed to create user' });
    }
});

// Update user
app.put('/api/users/:id', async (req, res) => {
    try {
        const data = await readData();
        const userIndex = data.users.findIndex(u => u.id === parseInt(req.params.id));
        
        if (userIndex === -1) {
            return res.status(404).json({ error: 'User not found' });
        }
        
        data.users[userIndex] = {
            ...data.users[userIndex],
            ...req.body,
            updated: new Date().toISOString()
        };
        
        await writeData(data);
        res.json(data.users[userIndex]);
        
    } catch (error) {
        res.status(500).json({ error: 'Failed to update user' });
    }
});

// Delete user
app.delete('/api/users/:id', async (req, res) => {
    try {
        const data = await readData();
        const userIndex = data.users.findIndex(u => u.id === parseInt(req.params.id));
        
        if (userIndex === -1) {
            return res.status(404).json({ error: 'User not found' });
        }
        
        const deletedUser = data.users.splice(userIndex, 1)[0];
        await writeData(data);
        
        res.json(deletedUser);
        
    } catch (error) {
        res.status(500).json({ error: 'Failed to delete user' });
    }
});

app.listen(3000, () => {
    console.log('JSON API server running on port 3000');
});
```

---

## 🔐 Environment Variables

### Using dotenv

```bash
# Install dotenv
npm install dotenv
```

```javascript
// .env file (never commit this to version control)
PORT=3000
DB_HOST=localhost
DB_USER=myuser
DB_PASS=mypassword
API_KEY=your-secret-api-key
NODE_ENV=development
```

```javascript
// config.js
require('dotenv').config();

const config = {
    port: process.env.PORT || 3000,
    database: {
        host: process.env.DB_HOST || 'localhost',
        user: process.env.DB_USER || 'defaultuser',
        password: process.env.DB_PASS || 'defaultpass'
    },
    apiKey: process.env.API_KEY,
    isDevelopment: process.env.NODE_ENV === 'development'
};

module.exports = config;
```

```javascript
// app-with-env.js
const express = require('express');
const config = require('./config');

const app = express();

app.get('/config', (req, res) => {
    res.json({
        port: config.port,
        environment: process.env.NODE_ENV,
        // Never expose sensitive data like API keys
        hasApiKey: !!config.apiKey
    });
});

app.listen(config.port, () => {
    console.log(`Server running on port ${config.port}`);
    if (config.isDevelopment) {
        console.log('Running in development mode');
    }
});
```

---

## ✅ Best Practices

### Project Structure

```
my-node-app/
├── src/
│   ├── controllers/
│   │   ├── userController.js
│   │   └── authController.js
│   ├── models/
│   │   └── User.js
│   ├── routes/
│   │   ├── userRoutes.js
│   │   └── authRoutes.js
│   ├── middleware/
│   │   ├── auth.js
│   │   └── validation.js
│   ├── utils/
│   │   └── helpers.js
│   └── app.js
├── tests/
│   ├── unit/
│   └── integration/
├── public/
├── .env
├── .gitignore
├── package.json
├── README.md
└── server.js
```

### Error Handling

```javascript
// error-handling.js
const express = require('express');
const app = express();

// Custom error class
class AppError extends Error {
    constructor(message, statusCode) {
        super(message);
        this.statusCode = statusCode;
        this.isOperational = true;
    }
}

// Async error wrapper
const asyncHandler = (fn) => {
    return (req, res, next) => {
        Promise.resolve(fn(req, res, next)).catch(next);
    };
};

// Example route with error handling
app.get('/api/users/:id', asyncHandler(async (req, res, next) => {
    const userId = req.params.id;
    
    if (!userId || isNaN(userId)) {
        throw new AppError('Invalid user ID', 400);
    }
    
    // Simulate database operation
    const user = await getUserById(userId);
    
    if (!user) {
        throw new AppError('User not found', 404);
    }
    
    res.json(user);
}));

async function getUserById(id) {
    // Simulate async operation
    if (id === '999') {
        throw new Error('Database connection failed');
    }
    
    return id === '1' ? { id: 1, name: 'John Doe' } : null;
}

// Global error handler
app.use((err, req, res, next) => {
    console.error(err);
    
    if (err instanceof AppError) {
        return res.status(err.statusCode).json({
            error: err.message
        });
    }
    
    // Handle unexpected errors
    res.status(500).json({
        error: 'Internal server error'
    });
});

app.listen(3000);
```

### Security Best Practices

```javascript
// security.js
const express = require('express');
const helmet = require('helmet');
const rateLimit = require('express-rate-limit');
const validator = require('validator');

const app = express();

// Security middleware
app.use(helmet()); // Set security headers

// Rate limiting
const limiter = rateLimit({
    windowMs: 15 * 60 * 1000, // 15 minutes
    max: 100, // limit each IP to 100 requests per windowMs
    message: 'Too many requests from this IP'
});
app.use(limiter);

app.use(express.json({ limit: '10mb' })); // Limit payload size

// Input validation
function validateUserInput(req, res, next) {
    const { email, name } = req.body;
    
    if (email && !validator.isEmail(email)) {
        return res.status(400).json({ error: 'Invalid email format' });
    }
    
    if (name && !validator.isLength(name, { min: 2, max: 50 })) {
        return res.status(400).json({ error: 'Name must be 2-50 characters' });
    }
    
    next();
}

app.post('/api/users', validateUserInput, (req, res) => {
    // Safe to process validated input
    res.json({ message: 'User created successfully' });
});

app.listen(3000);
```

---

## 📚 Quick Reference

### Node.js Core Modules

| Module | Purpose | Common Methods |
|--------|---------|----------------|
| `fs` | File system | `readFile()`, `writeFile()`, `mkdir()` |
| `http` | HTTP server/client | `createServer()`, `request()` |
| `path` | File paths | `join()`, `resolve()`, `extname()` |
| `url` | URL parsing | `parse()`, `format()` |
| `crypto` | Cryptography | `createHash()`, `randomBytes()` |
| `os` | Operating system | `platform()`, `cpus()`, `freemem()` |

### Express.js Methods

| Method | Purpose | Example |
|--------|---------|---------|
| `app.get()` | Handle GET requests | `app.get('/api/users', handler)` |
| `app.post()` | Handle POST requests | `app.post('/api/users', handler)` |
| `app.use()` | Apply middleware | `app.use(express.json())` |
| `res.json()` | Send JSON response | `res.json({ message: 'Hello' })` |
| `res.status()` | Set status code | `res.status(404).json({ error: 'Not found' })` |

### Best Practices:
1. Use async/await for asynchronous operations
2. Always handle errors properly
3. Use environment variables for configuration
4. Validate and sanitize user input
5. Follow RESTful API conventions
6. Use proper HTTP status codes
7. Implement rate limiting and security headers

### Next Steps:
1. Learn database integration (MongoDB, PostgreSQL)
2. Study authentication and authorization
3. Explore testing with Jest or Mocha
4. Learn about deployment and DevOps
5. Build a complete REST API project

This basic Node.js guide provides the foundation for server-side JavaScript development. Practice building APIs and web applications to master these concepts!
