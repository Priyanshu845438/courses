
# 🚀 Basic Express.js

## Table of Contents
- [What is Express.js?](#what-is-expressjs)
- [Setup and Installation](#setup-and-installation)
- [Basic Server](#basic-server)
- [Routing](#routing)
- [Middleware](#middleware)
- [Request and Response](#request-and-response)
- [Error Handling](#error-handling)
- [Authentication](#authentication)
- [Database Integration](#database-integration)
- [Best Practices](#best-practices)

---

## 🎯 What is Express.js?

**Express.js** is a minimal and flexible Node.js web application framework that provides robust features for web and mobile applications.

### Key Features:
- **Fast Development**: Minimal setup required
- **Middleware**: Powerful plugin system
- **Routing**: Flexible URL routing
- **Template Engines**: Support for multiple view engines
- **Static Files**: Built-in static file serving

### Express vs Other Frameworks

| Framework | Language | Learning Curve | Performance |
|-----------|----------|----------------|-------------|
| **Express** | JavaScript | Easy | High |
| **Koa** | JavaScript | Moderate | High |
| **Fastify** | JavaScript | Moderate | Very High |
| **NestJS** | TypeScript | Steep | High |

---

## 🛠 Setup and Installation

### Project Initialization

```bash
# Create new directory
mkdir my-express-app
cd my-express-app

# Initialize npm project
npm init -y

# Install Express
npm install express

# Install development dependencies
npm install --save-dev nodemon
```

### Package.json Scripts

```json
{
  "name": "my-express-app",
  "version": "1.0.0",
  "description": "",
  "main": "server.js",
  "scripts": {
    "start": "node server.js",
    "dev": "nodemon server.js",
    "test": "echo \"Error: no test specified\" && exit 1"
  },
  "dependencies": {
    "express": "^4.18.0"
  },
  "devDependencies": {
    "nodemon": "^3.0.0"
  }
}
```

### Project Structure

```
my-express-app/
├── server.js          # Main server file
├── routes/            # Route handlers
│   ├── auth.js
│   ├── users.js
│   └── posts.js
├── middleware/        # Custom middleware
│   ├── auth.js
│   └── validation.js
├── models/           # Data models
├── controllers/      # Route controllers
├── public/          # Static files
│   ├── css/
│   ├── js/
│   └── images/
├── views/           # Template files
├── config/          # Configuration files
└── package.json
```

---

## 🚀 Basic Server

### Simple Express Server

```javascript
// server.js
const express = require('express');
const app = express();
const PORT = process.env.PORT || 5000;

// Basic route
app.get('/', (req, res) => {
    res.send('Hello, Express!');
});

// Start server
app.listen(PORT, '0.0.0.0', () => {
    console.log(`Server running on port ${PORT}`);
});
```

### Server with JSON Support

```javascript
// server.js
const express = require('express');
const app = express();
const PORT = process.env.PORT || 5000;

// Middleware
app.use(express.json()); // Parse JSON bodies
app.use(express.urlencoded({ extended: true })); // Parse URL-encoded bodies

// Basic routes
app.get('/', (req, res) => {
    res.json({
        message: 'Welcome to Express API',
        timestamp: new Date().toISOString()
    });
});

app.get('/health', (req, res) => {
    res.status(200).json({
        status: 'OK',
        uptime: process.uptime(),
        timestamp: new Date().toISOString()
    });
});

// Start server
app.listen(PORT, '0.0.0.0', () => {
    console.log(`Server running on http://0.0.0.0:${PORT}`);
});
```

---

## 🛣 Routing

### Basic Routes

```javascript
const express = require('express');
const app = express();

// GET route
app.get('/users', (req, res) => {
    res.json([
        { id: 1, name: 'John Doe', email: 'john@example.com' },
        { id: 2, name: 'Jane Smith', email: 'jane@example.com' }
    ]);
});

// POST route
app.post('/users', (req, res) => {
    const { name, email } = req.body;
    
    if (!name || !email) {
        return res.status(400).json({
            error: 'Name and email are required'
        });
    }
    
    const newUser = {
        id: Date.now(),
        name,
        email,
        createdAt: new Date().toISOString()
    };
    
    res.status(201).json(newUser);
});

// PUT route
app.put('/users/:id', (req, res) => {
    const userId = parseInt(req.params.id);
    const { name, email } = req.body;
    
    // Simulate user update
    const updatedUser = {
        id: userId,
        name,
        email,
        updatedAt: new Date().toISOString()
    };
    
    res.json(updatedUser);
});

// DELETE route
app.delete('/users/:id', (req, res) => {
    const userId = parseInt(req.params.id);
    
    res.json({
        message: `User ${userId} deleted successfully`
    });
});
```

### Route Parameters

```javascript
// Route parameters
app.get('/users/:id', (req, res) => {
    const userId = req.params.id;
    
    res.json({
        id: userId,
        name: `User ${userId}`,
        email: `user${userId}@example.com`
    });
});

// Multiple parameters
app.get('/users/:userId/posts/:postId', (req, res) => {
    const { userId, postId } = req.params;
    
    res.json({
        userId,
        postId,
        title: `Post ${postId} by User ${userId}`
    });
});

// Optional parameters
app.get('/products/:category/:subcategory?', (req, res) => {
    const { category, subcategory } = req.params;
    
    res.json({
        category,
        subcategory: subcategory || 'all'
    });
});
```

### Query Parameters

```javascript
// Query parameters
app.get('/search', (req, res) => {
    const { 
        q,           // search query
        page = 1,    // default page
        limit = 10   // default limit
    } = req.query;
    
    if (!q) {
        return res.status(400).json({
            error: 'Query parameter "q" is required'
        });
    }
    
    res.json({
        query: q,
        page: parseInt(page),
        limit: parseInt(limit),
        results: [
            { id: 1, title: `Result for ${q}` },
            { id: 2, title: `Another result for ${q}` }
        ]
    });
});

// Multiple query parameters
app.get('/filter', (req, res) => {
    const filters = req.query;
    
    res.json({
        message: 'Filtered results',
        filters,
        count: Object.keys(filters).length
    });
});
```

### Router Module

```javascript
// routes/users.js
const express = require('express');
const router = express.Router();

// Middleware for this router
router.use((req, res, next) => {
    console.log('Users route accessed at:', new Date().toISOString());
    next();
});

// Get all users
router.get('/', (req, res) => {
    res.json([
        { id: 1, name: 'John', email: 'john@example.com' },
        { id: 2, name: 'Jane', email: 'jane@example.com' }
    ]);
});

// Get user by ID
router.get('/:id', (req, res) => {
    const userId = parseInt(req.params.id);
    
    if (isNaN(userId)) {
        return res.status(400).json({ error: 'Invalid user ID' });
    }
    
    res.json({
        id: userId,
        name: `User ${userId}`,
        email: `user${userId}@example.com`
    });
});

// Create new user
router.post('/', (req, res) => {
    const { name, email } = req.body;
    
    if (!name || !email) {
        return res.status(400).json({
            error: 'Name and email are required'
        });
    }
    
    const newUser = {
        id: Date.now(),
        name,
        email,
        createdAt: new Date().toISOString()
    };
    
    res.status(201).json(newUser);
});

module.exports = router;
```

```javascript
// server.js - Using router
const express = require('express');
const usersRouter = require('./routes/users');

const app = express();

app.use(express.json());

// Use router
app.use('/api/users', usersRouter);

app.listen(5000, '0.0.0.0', () => {
    console.log('Server running on port 5000');
});
```

---

## 🔧 Middleware

### Built-in Middleware

```javascript
const express = require('express');
const path = require('path');
const app = express();

// Parse JSON bodies
app.use(express.json());

// Parse URL-encoded bodies
app.use(express.urlencoded({ extended: true }));

// Serve static files
app.use(express.static(path.join(__dirname, 'public')));

// Serve static files with custom path
app.use('/assets', express.static(path.join(__dirname, 'assets')));
```

### Custom Middleware

```javascript
// Logger middleware
const logger = (req, res, next) => {
    const timestamp = new Date().toISOString();
    console.log(`${timestamp} - ${req.method} ${req.path}`);
    next();
};

// Authentication middleware
const authenticate = (req, res, next) => {
    const token = req.headers.authorization;
    
    if (!token) {
        return res.status(401).json({ error: 'No token provided' });
    }
    
    // Simple token validation (use proper JWT in production)
    if (token === 'Bearer valid-token') {
        req.user = { id: 1, name: 'John Doe' };
        next();
    } else {
        res.status(403).json({ error: 'Invalid token' });
    }
};

// Validation middleware
const validateUser = (req, res, next) => {
    const { name, email } = req.body;
    
    if (!name || !email) {
        return res.status(400).json({
            error: 'Name and email are required'
        });
    }
    
    if (!email.includes('@')) {
        return res.status(400).json({
            error: 'Invalid email format'
        });
    }
    
    next();
};

// Apply middleware
app.use(logger);

// Protected routes
app.use('/api/protected', authenticate);

app.get('/api/protected/profile', (req, res) => {
    res.json({
        message: 'This is a protected route',
        user: req.user
    });
});

// Route with validation
app.post('/api/users', validateUser, (req, res) => {
    const { name, email } = req.body;
    
    res.status(201).json({
        id: Date.now(),
        name,
        email,
        createdAt: new Date().toISOString()
    });
});
```

### Third-party Middleware

```javascript
const express = require('express');
const cors = require('cors');
const helmet = require('helmet');
const morgan = require('morgan');
const rateLimit = require('express-rate-limit');

const app = express();

// Security middleware
app.use(helmet());

// CORS middleware
app.use(cors({
    origin: ['http://localhost:3000', 'https://yourdomain.com'],
    credentials: true
}));

// Logging middleware
app.use(morgan('combined'));

// Rate limiting
const limiter = rateLimit({
    windowMs: 15 * 60 * 1000, // 15 minutes
    max: 100, // limit each IP to 100 requests per windowMs
    message: 'Too many requests from this IP'
});
app.use(limiter);

// Body parsing
app.use(express.json({ limit: '10mb' }));
app.use(express.urlencoded({ extended: true }));
```

---

## 📨 Request and Response

### Request Object

```javascript
app.get('/request-info', (req, res) => {
    const requestInfo = {
        // URL information
        originalUrl: req.originalUrl,
        path: req.path,
        query: req.query,
        params: req.params,
        
        // HTTP information
        method: req.method,
        headers: req.headers,
        userAgent: req.get('User-Agent'),
        
        // Client information
        ip: req.ip,
        ips: req.ips,
        hostname: req.hostname,
        protocol: req.protocol,
        secure: req.secure,
        
        // Body (for POST/PUT requests)
        body: req.body,
        
        // Cookies
        cookies: req.cookies
    };
    
    res.json(requestInfo);
});

// File upload handling
const multer = require('multer');
const upload = multer({ dest: 'uploads/' });

app.post('/upload', upload.single('file'), (req, res) => {
    if (!req.file) {
        return res.status(400).json({ error: 'No file uploaded' });
    }
    
    res.json({
        message: 'File uploaded successfully',
        file: {
            originalName: req.file.originalname,
            filename: req.file.filename,
            size: req.file.size,
            mimetype: req.file.mimetype
        }
    });
});
```

### Response Object

```javascript
app.get('/response-examples', (req, res) => {
    // Set status code
    res.status(200);
    
    // Set headers
    res.set('X-Custom-Header', 'MyValue');
    res.header('X-Another-Header', 'AnotherValue');
    
    // Send JSON response
    res.json({
        message: 'Success',
        data: { id: 1, name: 'Example' }
    });
});

// Different response types
app.get('/text', (req, res) => {
    res.send('Plain text response');
});

app.get('/html', (req, res) => {
    res.send('<h1>HTML Response</h1><p>This is HTML content</p>');
});

app.get('/json', (req, res) => {
    res.json({ message: 'JSON response', timestamp: Date.now() });
});

app.get('/redirect', (req, res) => {
    res.redirect('/');
});

app.get('/download', (req, res) => {
    res.download(path.join(__dirname, 'files/document.pdf'));
});

// Set cookies
app.get('/set-cookie', (req, res) => {
    res.cookie('username', 'john_doe', {
        maxAge: 900000,
        httpOnly: true,
        secure: false // Set to true in production with HTTPS
    });
    
    res.json({ message: 'Cookie set' });
});

// Clear cookies
app.get('/clear-cookie', (req, res) => {
    res.clearCookie('username');
    res.json({ message: 'Cookie cleared' });
});
```

---

## ⚠️ Error Handling

### Basic Error Handling

```javascript
// Error handling middleware (must be last)
app.use((err, req, res, next) => {
    console.error(err.stack);
    
    // Default error response
    res.status(err.status || 500).json({
        error: {
            message: err.message || 'Internal Server Error',
            status: err.status || 500
        }
    });
});

// 404 handler (must be after all routes)
app.use((req, res) => {
    res.status(404).json({
        error: {
            message: 'Route not found',
            status: 404,
            path: req.path
        }
    });
});
```

### Custom Error Classes

```javascript
// Custom error classes
class AppError extends Error {
    constructor(message, statusCode) {
        super(message);
        this.statusCode = statusCode;
        this.status = `${statusCode}`.startsWith('4') ? 'fail' : 'error';
        this.isOperational = true;
        
        Error.captureStackTrace(this, this.constructor);
    }
}

class ValidationError extends AppError {
    constructor(message) {
        super(message, 400);
    }
}

class NotFoundError extends AppError {
    constructor(message = 'Resource not found') {
        super(message, 404);
    }
}

// Async error wrapper
const asyncHandler = (fn) => {
    return (req, res, next) => {
        Promise.resolve(fn(req, res, next)).catch(next);
    };
};

// Usage in routes
app.get('/users/:id', asyncHandler(async (req, res, next) => {
    const userId = parseInt(req.params.id);
    
    if (isNaN(userId)) {
        throw new ValidationError('Invalid user ID');
    }
    
    if (userId > 1000) {
        throw new NotFoundError('User not found');
    }
    
    // Simulate async operation
    const user = await getUserById(userId);
    
    res.json(user);
}));

async function getUserById(id) {
    // Simulate database operation
    return new Promise((resolve, reject) => {
        setTimeout(() => {
            if (id === 999) {
                reject(new Error('Database connection failed'));
            } else {
                resolve({ id, name: `User ${id}`, email: `user${id}@example.com` });
            }
        }, 100);
    });
}

// Enhanced error handler
app.use((err, req, res, next) => {
    // Log error
    console.error(`Error ${err.status || 500}: ${err.message}`);
    
    // Handle specific error types
    if (err instanceof ValidationError) {
        return res.status(err.statusCode).json({
            error: {
                type: 'ValidationError',
                message: err.message
            }
        });
    }
    
    if (err instanceof NotFoundError) {
        return res.status(err.statusCode).json({
            error: {
                type: 'NotFoundError',
                message: err.message
            }
        });
    }
    
    // Handle unexpected errors
    res.status(err.statusCode || 500).json({
        error: {
            type: 'ServerError',
            message: process.env.NODE_ENV === 'production' 
                ? 'Something went wrong' 
                : err.message
        }
    });
});
```

---

## 🔐 Authentication

### Basic Authentication

```javascript
const bcrypt = require('bcryptjs');
const jwt = require('jsonwebtoken');

// In-memory user store (use database in production)
const users = [];

// Register route
app.post('/api/auth/register', async (req, res) => {
    try {
        const { name, email, password } = req.body;
        
        // Validation
        if (!name || !email || !password) {
            return res.status(400).json({
                error: 'Name, email, and password are required'
            });
        }
        
        // Check if user exists
        const existingUser = users.find(u => u.email === email);
        if (existingUser) {
            return res.status(400).json({
                error: 'User already exists'
            });
        }
        
        // Hash password
        const hashedPassword = await bcrypt.hash(password, 10);
        
        // Create user
        const user = {
            id: Date.now(),
            name,
            email,
            password: hashedPassword,
            createdAt: new Date().toISOString()
        };
        
        users.push(user);
        
        // Generate JWT token
        const token = jwt.sign(
            { userId: user.id, email: user.email },
            process.env.JWT_SECRET || 'your-secret-key',
            { expiresIn: '24h' }
        );
        
        res.status(201).json({
            message: 'User created successfully',
            token,
            user: {
                id: user.id,
                name: user.name,
                email: user.email
            }
        });
        
    } catch (error) {
        res.status(500).json({ error: 'Registration failed' });
    }
});

// Login route
app.post('/api/auth/login', async (req, res) => {
    try {
        const { email, password } = req.body;
        
        // Validation
        if (!email || !password) {
            return res.status(400).json({
                error: 'Email and password are required'
            });
        }
        
        // Find user
        const user = users.find(u => u.email === email);
        if (!user) {
            return res.status(401).json({
                error: 'Invalid credentials'
            });
        }
        
        // Check password
        const isPasswordValid = await bcrypt.compare(password, user.password);
        if (!isPasswordValid) {
            return res.status(401).json({
                error: 'Invalid credentials'
            });
        }
        
        // Generate JWT token
        const token = jwt.sign(
            { userId: user.id, email: user.email },
            process.env.JWT_SECRET || 'your-secret-key',
            { expiresIn: '24h' }
        );
        
        res.json({
            message: 'Login successful',
            token,
            user: {
                id: user.id,
                name: user.name,
                email: user.email
            }
        });
        
    } catch (error) {
        res.status(500).json({ error: 'Login failed' });
    }
});

// JWT Authentication middleware
const authenticateToken = (req, res, next) => {
    const authHeader = req.headers['authorization'];
    const token = authHeader && authHeader.split(' ')[1];
    
    if (!token) {
        return res.status(401).json({ error: 'Access token required' });
    }
    
    jwt.verify(token, process.env.JWT_SECRET || 'your-secret-key', (err, user) => {
        if (err) {
            return res.status(403).json({ error: 'Invalid or expired token' });
        }
        
        req.user = user;
        next();
    });
};

// Protected route
app.get('/api/auth/profile', authenticateToken, (req, res) => {
    const user = users.find(u => u.id === req.user.userId);
    
    if (!user) {
        return res.status(404).json({ error: 'User not found' });
    }
    
    res.json({
        id: user.id,
        name: user.name,
        email: user.email,
        createdAt: user.createdAt
    });
});
```

---

## 🗄️ Database Integration

### MongoDB with Mongoose

```javascript
const mongoose = require('mongoose');

// Connect to MongoDB
mongoose.connect('mongodb://localhost:27017/myapp', {
    useNewUrlParser: true,
    useUnifiedTopology: true
});

// User schema
const userSchema = new mongoose.Schema({
    name: {
        type: String,
        required: true,
        trim: true
    },
    email: {
        type: String,
        required: true,
        unique: true,
        lowercase: true
    },
    password: {
        type: String,
        required: true,
        minlength: 6
    }
}, {
    timestamps: true
});

const User = mongoose.model('User', userSchema);

// CRUD operations
app.get('/api/users', async (req, res) => {
    try {
        const users = await User.find().select('-password');
        res.json(users);
    } catch (error) {
        res.status(500).json({ error: error.message });
    }
});

app.post('/api/users', async (req, res) => {
    try {
        const user = new User(req.body);
        await user.save();
        res.status(201).json(user);
    } catch (error) {
        res.status(400).json({ error: error.message });
    }
});

app.get('/api/users/:id', async (req, res) => {
    try {
        const user = await User.findById(req.params.id).select('-password');
        if (!user) {
            return res.status(404).json({ error: 'User not found' });
        }
        res.json(user);
    } catch (error) {
        res.status(500).json({ error: error.message });
    }
});

app.put('/api/users/:id', async (req, res) => {
    try {
        const user = await User.findByIdAndUpdate(
            req.params.id,
            req.body,
            { new: true, runValidators: true }
        ).select('-password');
        
        if (!user) {
            return res.status(404).json({ error: 'User not found' });
        }
        
        res.json(user);
    } catch (error) {
        res.status(400).json({ error: error.message });
    }
});

app.delete('/api/users/:id', async (req, res) => {
    try {
        const user = await User.findByIdAndDelete(req.params.id);
        if (!user) {
            return res.status(404).json({ error: 'User not found' });
        }
        res.json({ message: 'User deleted successfully' });
    } catch (error) {
        res.status(500).json({ error: error.message });
    }
});
```

---

## ✅ Best Practices

### Project Structure

```javascript
// config/database.js
const mongoose = require('mongoose');

const connectDB = async () => {
    try {
        const conn = await mongoose.connect(process.env.MONGODB_URI, {
            useNewUrlParser: true,
            useUnifiedTopology: true
        });
        console.log(`MongoDB Connected: ${conn.connection.host}`);
    } catch (error) {
        console.error('Database connection failed:', error);
        process.exit(1);
    }
};

module.exports = connectDB;

// models/User.js
const mongoose = require('mongoose');

const userSchema = new mongoose.Schema({
    name: { type: String, required: true },
    email: { type: String, required: true, unique: true },
    password: { type: String, required: true }
}, { timestamps: true });

module.exports = mongoose.model('User', userSchema);

// controllers/userController.js
const User = require('../models/User');

exports.getUsers = async (req, res) => {
    try {
        const users = await User.find().select('-password');
        res.json(users);
    } catch (error) {
        res.status(500).json({ error: error.message });
    }
};

exports.createUser = async (req, res) => {
    try {
        const user = new User(req.body);
        await user.save();
        res.status(201).json(user);
    } catch (error) {
        res.status(400).json({ error: error.message });
    }
};

// routes/users.js
const express = require('express');
const { getUsers, createUser } = require('../controllers/userController');
const router = express.Router();

router.get('/', getUsers);
router.post('/', createUser);

module.exports = router;
```

### Environment Configuration

```javascript
// config/config.js
require('dotenv').config();

module.exports = {
    port: process.env.PORT || 5000,
    mongoURI: process.env.MONGODB_URI || 'mongodb://localhost:27017/myapp',
    jwtSecret: process.env.JWT_SECRET || 'your-secret-key',
    nodeEnv: process.env.NODE_ENV || 'development'
};

// .env file
PORT=5000
MONGODB_URI=mongodb://localhost:27017/myapp
JWT_SECRET=your-super-secret-jwt-key
NODE_ENV=development
```

## 📚 Quick Reference

### Essential Express Methods

| Method | Purpose | Example |
|--------|---------|---------|
| `app.get()` | Handle GET requests | `app.get('/users', handler)` |
| `app.post()` | Handle POST requests | `app.post('/users', handler)` |
| `app.put()` | Handle PUT requests | `app.put('/users/:id', handler)` |
| `app.delete()` | Handle DELETE requests | `app.delete('/users/:id', handler)` |
| `app.use()` | Apply middleware | `app.use(express.json())` |
| `app.listen()` | Start server | `app.listen(5000, callback)` |

### Common Middleware

| Middleware | Purpose | Usage |
|------------|---------|-------|
| `express.json()` | Parse JSON bodies | `app.use(express.json())` |
| `express.static()` | Serve static files | `app.use(express.static('public'))` |
| `cors()` | Enable CORS | `app.use(cors())` |
| `helmet()` | Security headers | `app.use(helmet())` |
| `morgan()` | HTTP logging | `app.use(morgan('combined'))` |

### Status Codes

| Code | Meaning | Use Case |
|------|---------|----------|
| 200 | OK | Successful GET, PUT, PATCH |
| 201 | Created | Successful POST |
| 400 | Bad Request | Invalid request data |
| 401 | Unauthorized | Authentication required |
| 403 | Forbidden | Access denied |
| 404 | Not Found | Resource doesn't exist |
| 500 | Internal Server Error | Server error |

### Next Steps:
1. Learn database integration (MongoDB/PostgreSQL)
2. Study authentication and authorization
3. Explore API documentation (Swagger)
4. Learn testing (Jest, Supertest)
5. Master deployment and DevOps

This Express.js guide provides the foundation for building robust web APIs and applications!
