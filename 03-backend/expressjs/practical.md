
# 🚀 Intermediate Express.js

## Table of Contents
- [Advanced Routing](#advanced-routing)
- [Custom Middleware](#custom-middleware)
- [Template Engines](#template-engines)
- [Session Management](#session-management)
- [File Uploads](#file-uploads)
- [API Development](#api-development)
- [Database Integration](#database-integration)
- [Security Best Practices](#security-best-practices)
- [Performance Optimization](#performance-optimization)

---

## 🛤️ Advanced Routing

### Route Parameters and Validation

```javascript
const express = require('express');
const { body, param, validationResult } = require('express-validator');
const router = express.Router();

// Multiple route parameters
router.get('/users/:userId/posts/:postId', [
    param('userId').isMongoId().withMessage('Invalid user ID'),
    param('postId').isMongoId().withMessage('Invalid post ID')
], (req, res) => {
    const errors = validationResult(req);
    if (!errors.isEmpty()) {
        return res.status(400).json({ errors: errors.array() });
    }
    
    const { userId, postId } = req.params;
    // Handle logic
    res.json({ userId, postId });
});

// Route with query parameters
router.get('/search', [
    query('q').isLength({ min: 1 }).withMessage('Query is required'),
    query('page').optional().isInt({ min: 1 }).withMessage('Page must be positive integer'),
    query('limit').optional().isInt({ min: 1, max: 100 }).withMessage('Limit must be between 1-100')
], (req, res) => {
    const errors = validationResult(req);
    if (!errors.isEmpty()) {
        return res.status(400).json({ errors: errors.array() });
    }
    
    const { q, page = 1, limit = 10 } = req.query;
    // Search logic
    res.json({ query: q, page: parseInt(page), limit: parseInt(limit) });
});
```

### Router Modules

```javascript
// routes/users.js
const express = require('express');
const router = express.Router();
const userController = require('../controllers/userController');
const auth = require('../middleware/auth');

// Middleware for all user routes
router.use(auth.requireAuth);

// CRUD operations
router.get('/', userController.getAllUsers);
router.get('/:id', userController.getUserById);
router.post('/', userController.createUser);
router.put('/:id', userController.updateUser);
router.delete('/:id', userController.deleteUser);

module.exports = router;

// app.js
const userRoutes = require('./routes/users');
app.use('/api/users', userRoutes);
```

---

## 🔧 Custom Middleware

### Authentication Middleware

```javascript
const jwt = require('jsonwebtoken');

// Authentication middleware
const authenticateToken = (req, res, next) => {
    const authHeader = req.headers['authorization'];
    const token = authHeader && authHeader.split(' ')[1];
    
    if (!token) {
        return res.status(401).json({ error: 'Access token required' });
    }
    
    jwt.verify(token, process.env.JWT_SECRET, (err, user) => {
        if (err) {
            return res.status(403).json({ error: 'Invalid token' });
        }
        req.user = user;
        next();
    });
};

// Role-based authorization
const authorize = (roles) => {
    return (req, res, next) => {
        if (!req.user) {
            return res.status(401).json({ error: 'Authentication required' });
        }
        
        if (!roles.includes(req.user.role)) {
            return res.status(403).json({ error: 'Insufficient permissions' });
        }
        
        next();
    };
};

// Usage
app.get('/admin/users', authenticateToken, authorize(['admin']), (req, res) => {
    // Admin only route
});
```

### Request Logging Middleware

```javascript
const fs = require('fs');
const path = require('path');

// Custom logging middleware
const requestLogger = (req, res, next) => {
    const timestamp = new Date().toISOString();
    const logEntry = `${timestamp} - ${req.method} ${req.url} - ${req.ip}\n`;
    
    fs.appendFile(path.join(__dirname, 'logs', 'requests.log'), logEntry, (err) => {
        if (err) console.error('Logging error:', err);
    });
    
    next();
};

// Rate limiting middleware
const rateLimit = (windowMs, maxRequests) => {
    const requests = new Map();
    
    return (req, res, next) => {
        const key = req.ip;
        const now = Date.now();
        const windowStart = now - windowMs;
        
        if (!requests.has(key)) {
            requests.set(key, []);
        }
        
        const userRequests = requests.get(key);
        const validRequests = userRequests.filter(time => time > windowStart);
        
        if (validRequests.length >= maxRequests) {
            return res.status(429).json({
                error: 'Too many requests',
                retryAfter: Math.ceil(windowMs / 1000)
            });
        }
        
        validRequests.push(now);
        requests.set(key, validRequests);
        next();
    };
};

// Usage
app.use('/api/', rateLimit(15 * 60 * 1000, 100)); // 100 requests per 15 minutes
```

---

## 🎨 Template Engines

### EJS Integration

```javascript
const express = require('express');
const app = express();

// Set EJS as view engine
app.set('view engine', 'ejs');
app.set('views', path.join(__dirname, 'views'));

// Route with template rendering
app.get('/dashboard', authenticateToken, async (req, res) => {
    try {
        const user = await User.findById(req.user.id);
        const stats = await getDashboardStats(user.id);
        
        res.render('dashboard', {
            title: 'Dashboard',
            user: user,
            stats: stats,
            currentYear: new Date().getFullYear()
        });
    } catch (error) {
        res.status(500).render('error', { error: error.message });
    }
});
```

### Handlebars Integration

```javascript
const exphbs = require('express-handlebars');

// Configure Handlebars
app.engine('handlebars', exphbs.engine({
    defaultLayout: 'main',
    helpers: {
        formatDate: (date) => {
            return new Date(date).toLocaleDateString();
        },
        ifEquals: (a, b, options) => {
            if (a === b) {
                return options.fn(this);
            }
            return options.inverse(this);
        }
    }
}));

app.set('view engine', 'handlebars');
```

---

## 📦 Session Management

### Express Session with Redis

```javascript
const session = require('express-session');
const RedisStore = require('connect-redis')(session);
const redis = require('redis');

// Redis client setup
const redisClient = redis.createClient({
    host: process.env.REDIS_HOST || 'localhost',
    port: process.env.REDIS_PORT || 6379
});

// Session configuration
app.use(session({
    store: new RedisStore({ client: redisClient }),
    secret: process.env.SESSION_SECRET,
    resave: false,
    saveUninitialized: false,
    cookie: {
        secure: process.env.NODE_ENV === 'production',
        httpOnly: true,
        maxAge: 24 * 60 * 60 * 1000 // 24 hours
    }
}));

// Session-based authentication
app.post('/login', async (req, res) => {
    const { username, password } = req.body;
    
    try {
        const user = await User.findOne({ username });
        if (!user || !await bcrypt.compare(password, user.password)) {
            return res.status(401).json({ error: 'Invalid credentials' });
        }
        
        req.session.userId = user._id;
        req.session.username = user.username;
        req.session.role = user.role;
        
        res.json({ message: 'Login successful', user: { id: user._id, username: user.username } });
    } catch (error) {
        res.status(500).json({ error: 'Login failed' });
    }
});

// Session check middleware
const requireSession = (req, res, next) => {
    if (!req.session.userId) {
        return res.status(401).json({ error: 'Authentication required' });
    }
    next();
};
```

---

## 📁 File Uploads

### Multer Configuration

```javascript
const multer = require('multer');
const path = require('path');

// Storage configuration
const storage = multer.diskStorage({
    destination: (req, file, cb) => {
        cb(null, 'uploads/');
    },
    filename: (req, file, cb) => {
        const uniqueSuffix = Date.now() + '-' + Math.round(Math.random() * 1E9);
        cb(null, file.fieldname + '-' + uniqueSuffix + path.extname(file.originalname));
    }
});

// File filter
const fileFilter = (req, file, cb) => {
    const allowedTypes = ['image/jpeg', 'image/png', 'image/gif'];
    if (allowedTypes.includes(file.mimetype)) {
        cb(null, true);
    } else {
        cb(new Error('Invalid file type. Only JPEG, PNG, and GIF allowed.'), false);
    }
};

// Multer instance
const upload = multer({
    storage: storage,
    fileFilter: fileFilter,
    limits: {
        fileSize: 5 * 1024 * 1024 // 5MB limit
    }
});

// File upload routes
app.post('/upload/single', upload.single('image'), (req, res) => {
    if (!req.file) {
        return res.status(400).json({ error: 'No file uploaded' });
    }
    
    res.json({
        message: 'File uploaded successfully',
        file: {
            originalname: req.file.originalname,
            filename: req.file.filename,
            path: req.file.path,
            size: req.file.size
        }
    });
});

app.post('/upload/multiple', upload.array('images', 5), (req, res) => {
    if (!req.files || req.files.length === 0) {
        return res.status(400).json({ error: 'No files uploaded' });
    }
    
    const fileInfo = req.files.map(file => ({
        originalname: file.originalname,
        filename: file.filename,
        path: file.path,
        size: file.size
    }));
    
    res.json({
        message: 'Files uploaded successfully',
        files: fileInfo
    });
});
```

---

## 🛡️ Security Best Practices

### Helmet and Security Headers

```javascript
const helmet = require('helmet');
const rateLimit = require('express-rate-limit');

// Basic security headers
app.use(helmet({
    contentSecurityPolicy: {
        directives: {
            defaultSrc: ["'self'"],
            styleSrc: ["'self'", "'unsafe-inline'"],
            scriptSrc: ["'self'"],
            imgSrc: ["'self'", "data:", "https:"]
        }
    }
}));

// Rate limiting
const limiter = rateLimit({
    windowMs: 15 * 60 * 1000, // 15 minutes
    max: 100, // limit each IP to 100 requests per windowMs
    message: 'Too many requests from this IP'
});

app.use('/api/', limiter);

// Input sanitization
const mongoSanitize = require('express-mongo-sanitize');
app.use(mongoSanitize());

// XSS protection
const xss = require('xss');
app.use((req, res, next) => {
    if (req.body) {
        Object.keys(req.body).forEach(key => {
            if (typeof req.body[key] === 'string') {
                req.body[key] = xss(req.body[key]);
            }
        });
    }
    next();
});
```

---

## 🚀 Performance Optimization

### Caching Strategies

```javascript
const redis = require('redis');
const client = redis.createClient();

// Cache middleware
const cacheMiddleware = (duration = 300) => {
    return async (req, res, next) => {
        const key = `cache:${req.originalUrl}`;
        
        try {
            const cachedData = await client.get(key);
            if (cachedData) {
                return res.json(JSON.parse(cachedData));
            }
            
            // Store original json method
            const originalJson = res.json;
            res.json = function(data) {
                // Cache the response
                client.setex(key, duration, JSON.stringify(data));
                return originalJson.call(this, data);
            };
            
            next();
        } catch (error) {
            console.error('Cache error:', error);
            next();
        }
    };
};

// Usage
app.get('/api/posts', cacheMiddleware(600), async (req, res) => {
    const posts = await Post.find().populate('author');
    res.json(posts);
});
```

### Compression and Optimization

```javascript
const compression = require('compression');

// Enable gzip compression
app.use(compression());

// Serve static files efficiently
app.use('/static', express.static('public', {
    maxAge: '1d',
    etag: true,
    lastModified: true
}));

// Database query optimization
const optimizedQuery = async (req, res) => {
    const { page = 1, limit = 10, search } = req.query;
    const skip = (page - 1) * limit;
    
    let query = {};
    if (search) {
        query = {
            $or: [
                { title: { $regex: search, $options: 'i' } },
                { content: { $regex: search, $options: 'i' } }
            ]
        };
    }
    
    const [posts, total] = await Promise.all([
        Post.find(query)
            .populate('author', 'username')
            .sort({ createdAt: -1 })
            .skip(skip)
            .limit(parseInt(limit))
            .lean(), // Use lean() for better performance
        Post.countDocuments(query)
    ]);
    
    res.json({
        posts,
        pagination: {
            page: parseInt(page),
            limit: parseInt(limit),
            total,
            pages: Math.ceil(total / limit)
        }
    });
};
```

---

## 📚 Summary

### Intermediate Express.js Capabilities:
- **Advanced Routing**: Parameters, validation, modular routes
- **Custom Middleware**: Authentication, logging, rate limiting
- **Template Engines**: EJS, Handlebars integration
- **Session Management**: Redis-based sessions
- **File Uploads**: Multer configuration and handling
- **Security**: Helmet, sanitization, XSS protection
- **Performance**: Caching, compression, query optimization

### Best Practices:
1. **Use middleware chains** for reusable functionality
2. **Implement proper error handling** at all levels
3. **Validate and sanitize** all user inputs
4. **Cache frequently accessed data**
5. **Use compression** for better performance
6. **Implement rate limiting** to prevent abuse
7. **Structure routes** in separate modules
8. **Use environment variables** for configuration

### Next Steps:
1. Learn about microservices architecture
2. Implement WebSocket connections
3. Study containerization with Docker
4. Explore serverless deployment options
5. Learn about API documentation with Swagger
