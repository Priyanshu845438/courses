
# 🔧 Intermediate API Development

## Table of Contents
- [Advanced REST API Design](#advanced-rest-api-design)
- [API Authentication](#api-authentication)
- [Data Validation and Sanitization](#data-validation-and-sanitization)
- [Error Handling](#error-handling)
- [API Versioning](#api-versioning)
- [Rate Limiting](#rate-limiting)
- [Caching Strategies](#caching-strategies)
- [API Documentation](#api-documentation)
- [Testing APIs](#testing-apis)

---

## 🎯 Advanced REST API Design

### Resource Relationships

```javascript
const express = require('express');
const router = express.Router();

// Users with nested resources
router.get('/users/:userId/posts', async (req, res) => {
    try {
        const { userId } = req.params;
        const { page = 1, limit = 10 } = req.query;
        
        const posts = await Post.find({ userId })
            .populate('author', 'name email')
            .populate('comments', 'content createdAt')
            .skip((page - 1) * limit)
            .limit(parseInt(limit))
            .sort({ createdAt: -1 });
            
        const total = await Post.countDocuments({ userId });
        
        res.json({
            data: posts,
            pagination: {
                page: parseInt(page),
                limit: parseInt(limit),
                total,
                pages: Math.ceil(total / limit)
            }
        });
    } catch (error) {
        res.status(500).json({ error: error.message });
    }
});

// Bulk operations
router.post('/posts/bulk', async (req, res) => {
    try {
        const { action, ids, data } = req.body;
        
        switch (action) {
            case 'delete':
                await Post.deleteMany({ _id: { $in: ids } });
                res.json({ message: `${ids.length} posts deleted` });
                break;
                
            case 'update':
                const result = await Post.updateMany(
                    { _id: { $in: ids } },
                    { $set: data }
                );
                res.json({ modified: result.modifiedCount });
                break;
                
            default:
                res.status(400).json({ error: 'Invalid action' });
        }
    } catch (error) {
        res.status(500).json({ error: error.message });
    }
});
```

### Advanced Filtering and Searching

```javascript
// Advanced query builder
class QueryBuilder {
    constructor(Model) {
        this.Model = Model;
        this.query = Model.find();
    }
    
    filter(filters) {
        // Handle comparison operators
        Object.keys(filters).forEach(key => {
            const value = filters[key];
            
            if (typeof value === 'object' && value !== null) {
                // Handle operators like { age: { $gte: 18, $lte: 65 } }
                this.query = this.query.where(key, value);
            } else {
                this.query = this.query.where(key, value);
            }
        });
        
        return this;
    }
    
    search(searchTerm, fields) {
        if (searchTerm && fields) {
            const searchRegex = new RegExp(searchTerm, 'i');
            const searchConditions = fields.map(field => ({
                [field]: searchRegex
            }));
            
            this.query = this.query.or(searchConditions);
        }
        
        return this;
    }
    
    sort(sortBy) {
        if (sortBy) {
            // Handle sorting like "name,-createdAt"
            const sortFields = sortBy.split(',').reduce((acc, field) => {
                if (field.startsWith('-')) {
                    acc[field.substring(1)] = -1;
                } else {
                    acc[field] = 1;
                }
                return acc;
            }, {});
            
            this.query = this.query.sort(sortFields);
        }
        
        return this;
    }
    
    paginate(page = 1, limit = 10) {
        const skip = (page - 1) * limit;
        this.query = this.query.skip(skip).limit(parseInt(limit));
        return this;
    }
    
    populate(fields) {
        if (fields) {
            this.query = this.query.populate(fields);
        }
        return this;
    }
    
    async execute() {
        return await this.query.exec();
    }
}

// Usage in route
router.get('/posts', async (req, res) => {
    try {
        const {
            page = 1,
            limit = 10,
            sort = '-createdAt',
            search,
            ...filters
        } = req.query;
        
        const posts = await new QueryBuilder(Post)
            .filter(filters)
            .search(search, ['title', 'content'])
            .sort(sort)
            .paginate(page, limit)
            .populate('author', 'name email')
            .execute();
            
        res.json({
            success: true,
            data: posts,
            pagination: {
                page: parseInt(page),
                limit: parseInt(limit)
            }
        });
    } catch (error) {
        res.status(500).json({ error: error.message });
    }
});
```

---

## 🔐 API Authentication

### JWT Implementation

```javascript
const jwt = require('jsonwebtoken');
const bcrypt = require('bcryptjs');

// JWT utility functions
class JWTUtils {
    static generateTokens(payload) {
        const accessToken = jwt.sign(
            payload,
            process.env.JWT_ACCESS_SECRET,
            { expiresIn: '15m' }
        );
        
        const refreshToken = jwt.sign(
            payload,
            process.env.JWT_REFRESH_SECRET,
            { expiresIn: '7d' }
        );
        
        return { accessToken, refreshToken };
    }
    
    static verifyAccessToken(token) {
        return jwt.verify(token, process.env.JWT_ACCESS_SECRET);
    }
    
    static verifyRefreshToken(token) {
        return jwt.verify(token, process.env.JWT_REFRESH_SECRET);
    }
}

// Authentication middleware
const authenticateToken = (req, res, next) => {
    const authHeader = req.headers['authorization'];
    const token = authHeader && authHeader.split(' ')[1];
    
    if (!token) {
        return res.status(401).json({ error: 'Access token required' });
    }
    
    try {
        const decoded = JWTUtils.verifyAccessToken(token);
        req.user = decoded;
        next();
    } catch (error) {
        if (error.name === 'TokenExpiredError') {
            return res.status(401).json({ 
                error: 'Token expired',
                code: 'TOKEN_EXPIRED'
            });
        }
        
        return res.status(403).json({ error: 'Invalid token' });
    }
};

// Role-based authorization
const authorize = (...roles) => {
    return (req, res, next) => {
        if (!req.user) {
            return res.status(401).json({ error: 'Authentication required' });
        }
        
        if (!roles.includes(req.user.role)) {
            return res.status(403).json({ 
                error: 'Insufficient permissions',
                required: roles,
                current: req.user.role
            });
        }
        
        next();
    };
};

// Auth routes
router.post('/login', async (req, res) => {
    try {
        const { email, password } = req.body;
        
        const user = await User.findOne({ email }).select('+password');
        if (!user) {
            return res.status(401).json({ error: 'Invalid credentials' });
        }
        
        const isValidPassword = await bcrypt.compare(password, user.password);
        if (!isValidPassword) {
            return res.status(401).json({ error: 'Invalid credentials' });
        }
        
        const tokens = JWTUtils.generateTokens({
            id: user._id,
            email: user.email,
            role: user.role
        });
        
        // Store refresh token in database
        user.refreshToken = tokens.refreshToken;
        await user.save();
        
        res.json({
            success: true,
            tokens,
            user: {
                id: user._id,
                email: user.email,
                role: user.role
            }
        });
    } catch (error) {
        res.status(500).json({ error: error.message });
    }
});

router.post('/refresh', async (req, res) => {
    try {
        const { refreshToken } = req.body;
        
        if (!refreshToken) {
            return res.status(401).json({ error: 'Refresh token required' });
        }
        
        const decoded = JWTUtils.verifyRefreshToken(refreshToken);
        const user = await User.findById(decoded.id);
        
        if (!user || user.refreshToken !== refreshToken) {
            return res.status(403).json({ error: 'Invalid refresh token' });
        }
        
        const tokens = JWTUtils.generateTokens({
            id: user._id,
            email: user.email,
            role: user.role
        });
        
        user.refreshToken = tokens.refreshToken;
        await user.save();
        
        res.json({ success: true, tokens });
    } catch (error) {
        res.status(403).json({ error: 'Invalid refresh token' });
    }
});
```

---

## ✅ Data Validation and Sanitization

### Advanced Validation with Joi

```javascript
const Joi = require('joi');

// Custom validation schemas
const schemas = {
    user: Joi.object({
        name: Joi.string().min(2).max(50).required(),
        email: Joi.string().email().required(),
        password: Joi.string()
            .min(8)
            .pattern(/^(?=.*[a-z])(?=.*[A-Z])(?=.*\d)(?=.*[@$!%*?&])[A-Za-z\d@$!%*?&]/)
            .required()
            .messages({
                'string.pattern.base': 'Password must contain at least one uppercase, lowercase, number and special character'
            }),
        age: Joi.number().integer().min(13).max(120),
        role: Joi.string().valid('user', 'admin', 'moderator').default('user'),
        preferences: Joi.object({
            newsletter: Joi.boolean().default(false),
            notifications: Joi.boolean().default(true),
            theme: Joi.string().valid('light', 'dark').default('light')
        }).default({})
    }),
    
    post: Joi.object({
        title: Joi.string().min(5).max(200).required(),
        content: Joi.string().min(10).max(5000).required(),
        tags: Joi.array().items(Joi.string().min(2).max(20)).max(5),
        category: Joi.string().required(),
        published: Joi.boolean().default(false),
        publishedAt: Joi.date().when('published', {
            is: true,
            then: Joi.required(),
            otherwise: Joi.optional()
        })
    }),
    
    query: Joi.object({
        page: Joi.number().integer().min(1).default(1),
        limit: Joi.number().integer().min(1).max(100).default(10),
        sort: Joi.string().pattern(/^[-]?[a-zA-Z_]+$/),
        search: Joi.string().max(100),
        status: Joi.string().valid('active', 'inactive', 'pending')
    })
};

// Validation middleware
const validate = (schema, property = 'body') => {
    return (req, res, next) => {
        const { error, value } = schema.validate(req[property], {
            abortEarly: false,
            stripUnknown: true
        });
        
        if (error) {
            const errors = error.details.map(detail => ({
                field: detail.path.join('.'),
                message: detail.message,
                value: detail.context.value
            }));
            
            return res.status(400).json({
                error: 'Validation failed',
                details: errors
            });
        }
        
        req[property] = value;
        next();
    };
};

// Sanitization middleware
const sanitize = (req, res, next) => {
    const sanitizeString = (str) => {
        if (typeof str !== 'string') return str;
        
        return str
            .trim()
            .replace(/<script\b[^<]*(?:(?!<\/script>)<[^<]*)*<\/script>/gi, '')
            .replace(/javascript:/gi, '')
            .replace(/on\w+\s*=\s*["'][^"']*["']/gi, '');
    };
    
    const sanitizeObject = (obj) => {
        if (!obj || typeof obj !== 'object') return obj;
        
        if (Array.isArray(obj)) {
            return obj.map(sanitizeObject);
        }
        
        const sanitized = {};
        Object.keys(obj).forEach(key => {
            if (typeof obj[key] === 'string') {
                sanitized[key] = sanitizeString(obj[key]);
            } else if (typeof obj[key] === 'object') {
                sanitized[key] = sanitizeObject(obj[key]);
            } else {
                sanitized[key] = obj[key];
            }
        });
        
        return sanitized;
    };
    
    req.body = sanitizeObject(req.body);
    req.query = sanitizeObject(req.query);
    next();
};

// Usage
router.post('/users', 
    sanitize,
    validate(schemas.user),
    async (req, res) => {
        // req.body is now validated and sanitized
        const user = new User(req.body);
        await user.save();
        res.status(201).json({ success: true, user });
    }
);
```

---

## 🚨 Error Handling

### Centralized Error Handling

```javascript
// Custom error classes
class APIError extends Error {
    constructor(message, statusCode = 500, isOperational = true) {
        super(message);
        this.statusCode = statusCode;
        this.isOperational = isOperational;
        this.timestamp = new Date().toISOString();
        
        Error.captureStackTrace(this, this.constructor);
    }
}

class ValidationError extends APIError {
    constructor(message, details = []) {
        super(message, 400);
        this.details = details;
        this.type = 'VALIDATION_ERROR';
    }
}

class NotFoundError extends APIError {
    constructor(resource = 'Resource') {
        super(`${resource} not found`, 404);
        this.type = 'NOT_FOUND';
    }
}

class UnauthorizedError extends APIError {
    constructor(message = 'Unauthorized access') {
        super(message, 401);
        this.type = 'UNAUTHORIZED';
    }
}

// Error handling middleware
const errorHandler = (err, req, res, next) => {
    let error = { ...err };
    error.message = err.message;
    
    // Log error
    console.error(err);
    
    // Mongoose bad ObjectId
    if (err.name === 'CastError') {
        const message = 'Invalid ID format';
        error = new APIError(message, 400);
    }
    
    // Mongoose duplicate key
    if (err.code === 11000) {
        const field = Object.keys(err.keyValue)[0];
        const message = `${field} already exists`;
        error = new APIError(message, 400);
    }
    
    // Mongoose validation error
    if (err.name === 'ValidationError') {
        const message = Object.values(err.errors).map(val => val.message).join(', ');
        error = new ValidationError(message);
    }
    
    // JWT errors
    if (err.name === 'JsonWebTokenError') {
        error = new UnauthorizedError('Invalid token');
    }
    
    if (err.name === 'TokenExpiredError') {
        error = new UnauthorizedError('Token expired');
    }
    
    res.status(error.statusCode || 500).json({
        success: false,
        error: {
            message: error.message || 'Server Error',
            type: error.type || 'INTERNAL_ERROR',
            ...(process.env.NODE_ENV === 'development' && { stack: err.stack }),
            ...(error.details && { details: error.details }),
            timestamp: error.timestamp || new Date().toISOString(),
            requestId: req.headers['x-request-id']
        }
    });
};

// Async error wrapper
const asyncHandler = (fn) => (req, res, next) => {
    Promise.resolve(fn(req, res, next)).catch(next);
};

// 404 handler
const notFound = (req, res, next) => {
    const error = new NotFoundError(`Route ${req.originalUrl}`);
    next(error);
};

// Usage
router.get('/users/:id', asyncHandler(async (req, res, next) => {
    const user = await User.findById(req.params.id);
    
    if (!user) {
        throw new NotFoundError('User');
    }
    
    res.json({ success: true, user });
}));
```

---

## 📈 API Versioning

### URL Path Versioning

```javascript
// Version 1 routes
const v1Router = express.Router();

v1Router.get('/users', async (req, res) => {
    const users = await User.find().select('name email');
    res.json({ users });
});

// Version 2 routes
const v2Router = express.Router();

v2Router.get('/users', async (req, res) => {
    const users = await User.find().select('name email profile createdAt');
    res.json({ 
        users,
        version: '2.0',
        total: users.length
    });
});

// Mount versioned routes
app.use('/api/v1', v1Router);
app.use('/api/v2', v2Router);

// Header-based versioning middleware
const versionHandler = (req, res, next) => {
    const version = req.headers['api-version'] || '1.0';
    req.apiVersion = version;
    
    switch (version) {
        case '1.0':
            req.url = '/api/v1' + req.url;
            break;
        case '2.0':
            req.url = '/api/v2' + req.url;
            break;
        default:
            return res.status(400).json({
                error: 'Unsupported API version',
                supported: ['1.0', '2.0']
            });
    }
    
    next();
};
```

---

## 🛡️ Rate Limiting

### Advanced Rate Limiting

```javascript
const rateLimit = require('express-rate-limit');
const RedisStore = require('rate-limit-redis');
const redis = require('redis');

const redisClient = redis.createClient({
    host: process.env.REDIS_HOST,
    port: process.env.REDIS_PORT
});

// Different rate limits for different endpoints
const createRateLimit = (options) => {
    return rateLimit({
        store: new RedisStore({
            client: redisClient,
            prefix: 'rl:'
        }),
        windowMs: options.windowMs || 15 * 60 * 1000,
        max: options.max || 100,
        message: {
            error: 'Too many requests',
            retryAfter: Math.ceil(options.windowMs / 1000)
        },
        standardHeaders: true,
        legacyHeaders: false,
        keyGenerator: (req) => {
            return req.user ? `user:${req.user.id}` : req.ip;
        },
        skip: (req) => {
            // Skip rate limiting for admins
            return req.user && req.user.role === 'admin';
        }
    });
};

// Apply different rate limits
app.use('/api/auth/login', createRateLimit({
    windowMs: 15 * 60 * 1000, // 15 minutes
    max: 5 // 5 login attempts
}));

app.use('/api/posts', createRateLimit({
    windowMs: 60 * 1000, // 1 minute
    max: 30 // 30 requests per minute
}));

app.use('/api/', createRateLimit({
    windowMs: 15 * 60 * 1000, // 15 minutes
    max: 100 // 100 requests per 15 minutes
}));
```

---

## 📚 Complete API Example

```javascript
const express = require('express');
const mongoose = require('mongoose');
const cors = require('cors');
const helmet = require('helmet');
const compression = require('compression');

const app = express();

// Security middleware
app.use(helmet());
app.use(cors({
    origin: process.env.ALLOWED_ORIGINS?.split(',') || ['http://localhost:3000'],
    credentials: true
}));

// General middleware
app.use(compression());
app.use(express.json({ limit: '10mb' }));
app.use(express.urlencoded({ extended: true }));

// Request logging
app.use((req, res, next) => {
    req.startTime = Date.now();
    req.requestId = require('uuid').v4();
    
    console.log(`${new Date().toISOString()} - ${req.method} ${req.path} - ${req.requestId}`);
    next();
});

// Routes
app.use('/api/auth', authRoutes);
app.use('/api/users', authenticateToken, userRoutes);
app.use('/api/posts', authenticateToken, postRoutes);

// Error handling
app.use(notFound);
app.use(errorHandler);

const PORT = process.env.PORT || 5000;

mongoose.connect(process.env.MONGODB_URI)
    .then(() => {
        console.log('Connected to MongoDB');
        app.listen(PORT, '0.0.0.0', () => {
            console.log(`Server running on port ${PORT}`);
        });
    })
    .catch(error => {
        console.error('Database connection failed:', error);
        process.exit(1);
    });
```

This intermediate API development guide covers authentication, validation, error handling, versioning, rate limiting, and advanced REST API patterns essential for building production-ready APIs.
