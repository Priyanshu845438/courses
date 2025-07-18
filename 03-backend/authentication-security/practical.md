
# 🔐 Intermediate Authentication & Security

## Table of Contents
- [JWT Implementation](#jwt-implementation)
- [OAuth 2.0 Integration](#oauth-20-integration)
- [Session Management](#session-management)
- [Password Security](#password-security)
- [Role-Based Access Control](#role-based-access-control)
- [Security Headers](#security-headers)
- [API Security](#api-security)
- [Input Validation](#input-validation)
- [Rate Limiting](#rate-limiting)

---

## 🎯 JWT Implementation

### JWT Structure and Creation

```javascript
const jwt = require('jsonwebtoken');
const bcrypt = require('bcryptjs');

// JWT Token Generation
const generateToken = (userId, email, role) => {
    const payload = {
        userId,
        email,
        role,
        iat: Date.now() / 1000,
        exp: Math.floor(Date.now() / 1000) + (60 * 60 * 24) // 24 hours
    };

    return jwt.sign(payload, process.env.JWT_SECRET, {
        expiresIn: '24h',
        issuer: 'your-app-name',
        audience: 'your-app-users'
    });
};

// JWT Verification Middleware
const verifyToken = (req, res, next) => {
    const token = req.header('Authorization')?.replace('Bearer ', '');
    
    if (!token) {
        return res.status(401).json({ message: 'Access denied. No token provided.' });
    }

    try {
        const decoded = jwt.verify(token, process.env.JWT_SECRET);
        req.user = decoded;
        next();
    } catch (error) {
        if (error.name === 'TokenExpiredError') {
            return res.status(401).json({ message: 'Token expired' });
        }
        return res.status(400).json({ message: 'Invalid token' });
    }
};
```

### Refresh Token Implementation

```javascript
// Refresh Token Model
const refreshTokens = new Map(); // Use Redis in production

const generateRefreshToken = (userId) => {
    const refreshToken = jwt.sign(
        { userId },
        process.env.REFRESH_TOKEN_SECRET,
        { expiresIn: '7d' }
    );
    
    refreshTokens.set(refreshToken, userId);
    return refreshToken;
};

// Token Refresh Endpoint
app.post('/api/auth/refresh', (req, res) => {
    const { refreshToken } = req.body;
    
    if (!refreshToken || !refreshTokens.has(refreshToken)) {
        return res.status(403).json({ message: 'Invalid refresh token' });
    }

    try {
        const decoded = jwt.verify(refreshToken, process.env.REFRESH_TOKEN_SECRET);
        const user = getUserById(decoded.userId);
        
        const newAccessToken = generateToken(user.id, user.email, user.role);
        
        res.json({ accessToken: newAccessToken });
    } catch (error) {
        refreshTokens.delete(refreshToken);
        return res.status(403).json({ message: 'Invalid refresh token' });
    }
});
```

---

## 🔑 OAuth 2.0 Integration

### Google OAuth Setup

```javascript
const passport = require('passport');
const GoogleStrategy = require('passport-google-oauth20').Strategy;

passport.use(new GoogleStrategy({
    clientID: process.env.GOOGLE_CLIENT_ID,
    clientSecret: process.env.GOOGLE_CLIENT_SECRET,
    callbackURL: "/auth/google/callback"
}, async (accessToken, refreshToken, profile, done) => {
    try {
        let user = await User.findOne({ googleId: profile.id });
        
        if (!user) {
            user = await User.create({
                googleId: profile.id,
                email: profile.emails[0].value,
                name: profile.displayName,
                avatar: profile.photos[0].value
            });
        }
        
        return done(null, user);
    } catch (error) {
        return done(error, null);
    }
}));

// OAuth Routes
app.get('/auth/google',
    passport.authenticate('google', { scope: ['profile', 'email'] })
);

app.get('/auth/google/callback',
    passport.authenticate('google', { failureRedirect: '/login' }),
    (req, res) => {
        const token = generateToken(req.user.id, req.user.email, req.user.role);
        res.redirect(`/dashboard?token=${token}`);
    }
);
```

---

## 🛡️ Role-Based Access Control

### RBAC Implementation

```javascript
// Role definitions
const ROLES = {
    ADMIN: 'admin',
    MODERATOR: 'moderator',
    USER: 'user'
};

const PERMISSIONS = {
    CREATE_POST: 'create_post',
    EDIT_POST: 'edit_post',
    DELETE_POST: 'delete_post',
    MANAGE_USERS: 'manage_users'
};

// Role-Permission mapping
const rolePermissions = {
    [ROLES.ADMIN]: [
        PERMISSIONS.CREATE_POST,
        PERMISSIONS.EDIT_POST,
        PERMISSIONS.DELETE_POST,
        PERMISSIONS.MANAGE_USERS
    ],
    [ROLES.MODERATOR]: [
        PERMISSIONS.CREATE_POST,
        PERMISSIONS.EDIT_POST,
        PERMISSIONS.DELETE_POST
    ],
    [ROLES.USER]: [
        PERMISSIONS.CREATE_POST
    ]
};

// Permission check middleware
const requirePermission = (permission) => {
    return (req, res, next) => {
        const userRole = req.user.role;
        const userPermissions = rolePermissions[userRole] || [];
        
        if (!userPermissions.includes(permission)) {
            return res.status(403).json({ 
                message: 'Insufficient permissions' 
            });
        }
        
        next();
    };
};

// Usage
app.delete('/api/posts/:id', 
    verifyToken, 
    requirePermission(PERMISSIONS.DELETE_POST),
    deletePost
);
```

---

## 🔒 Advanced Password Security

### Password Hashing with Salt

```javascript
const bcrypt = require('bcryptjs');
const crypto = require('crypto');

// Enhanced password hashing
const hashPassword = async (password) => {
    const saltRounds = 12;
    const salt = await bcrypt.genSalt(saltRounds);
    return bcrypt.hash(password, salt);
};

// Password strength validation
const validatePassword = (password) => {
    const minLength = 8;
    const hasUpperCase = /[A-Z]/.test(password);
    const hasLowerCase = /[a-z]/.test(password);
    const hasNumbers = /\d/.test(password);
    const hasSpecialChar = /[!@#$%^&*(),.?":{}|<>]/.test(password);
    
    const errors = [];
    
    if (password.length < minLength) {
        errors.push('Password must be at least 8 characters long');
    }
    if (!hasUpperCase) {
        errors.push('Password must contain at least one uppercase letter');
    }
    if (!hasLowerCase) {
        errors.push('Password must contain at least one lowercase letter');
    }
    if (!hasNumbers) {
        errors.push('Password must contain at least one number');
    }
    if (!hasSpecialChar) {
        errors.push('Password must contain at least one special character');
    }
    
    return {
        isValid: errors.length === 0,
        errors
    };
};

// Password reset functionality
const generateResetToken = () => {
    return crypto.randomBytes(32).toString('hex');
};

const passwordResetTokens = new Map();

app.post('/api/auth/forgot-password', async (req, res) => {
    const { email } = req.body;
    const user = await User.findOne({ email });
    
    if (!user) {
        return res.json({ message: 'If email exists, reset link has been sent' });
    }
    
    const resetToken = generateResetToken();
    const expiresAt = Date.now() + 3600000; // 1 hour
    
    passwordResetTokens.set(resetToken, { userId: user.id, expiresAt });
    
    // Send email with reset link
    await sendPasswordResetEmail(email, resetToken);
    
    res.json({ message: 'If email exists, reset link has been sent' });
});
```

---

## 🛡️ Security Headers

### Helmet.js Configuration

```javascript
const helmet = require('helmet');

app.use(helmet({
    contentSecurityPolicy: {
        directives: {
            defaultSrc: ["'self'"],
            styleSrc: ["'self'", "'unsafe-inline'", "https://fonts.googleapis.com"],
            fontSrc: ["'self'", "https://fonts.gstatic.com"],
            scriptSrc: ["'self'"],
            imgSrc: ["'self'", "data:", "https:"],
            connectSrc: ["'self'"],
            frameSrc: ["'none'"],
            objectSrc: ["'none'"],
            mediaSrc: ["'self'"],
            formAction: ["'self'"],
        },
    },
    crossOriginEmbedderPolicy: true,
    crossOriginOpenerPolicy: true,
    crossOriginResourcePolicy: { policy: "cross-origin" },
    dnsPrefetchControl: true,
    frameguard: { action: 'deny' },
    hidePoweredBy: true,
    hsts: {
        maxAge: 31536000,
        includeSubDomains: true,
        preload: true
    },
    ieNoOpen: true,
    noSniff: true,
    originAgentCluster: true,
    permittedCrossDomainPolicies: false,
    referrerPolicy: { policy: "no-referrer" },
    xssFilter: true,
}));
```

---

## 📊 Rate Limiting

### Express Rate Limiter

```javascript
const rateLimit = require('express-rate-limit');
const RedisStore = require('rate-limit-redis');
const redis = require('redis');

const redisClient = redis.createClient();

// General rate limiting
const generalLimiter = rateLimit({
    store: new RedisStore({
        client: redisClient,
        prefix: 'rl:general:'
    }),
    windowMs: 15 * 60 * 1000, // 15 minutes
    max: 100, // limit each IP to 100 requests per windowMs
    message: 'Too many requests from this IP, please try again later.'
});

// Strict rate limiting for auth endpoints
const authLimiter = rateLimit({
    store: new RedisStore({
        client: redisClient,
        prefix: 'rl:auth:'
    }),
    windowMs: 15 * 60 * 1000, // 15 minutes
    max: 5, // limit each IP to 5 requests per windowMs
    message: 'Too many authentication attempts, please try again later.'
});

// Apply rate limiting
app.use('/api/', generalLimiter);
app.use('/api/auth/', authLimiter);
```

---

## 🔍 Input Validation

### Joi Validation Middleware

```javascript
const Joi = require('joi');

const validateRequest = (schema) => {
    return (req, res, next) => {
        const { error } = schema.validate(req.body);
        
        if (error) {
            return res.status(400).json({
                message: 'Validation error',
                details: error.details.map(detail => ({
                    field: detail.path.join('.'),
                    message: detail.message
                }))
            });
        }
        
        next();
    };
};

// Validation schemas
const userRegistrationSchema = Joi.object({
    email: Joi.string().email().required(),
    password: Joi.string().min(8).pattern(new RegExp('^(?=.*[a-z])(?=.*[A-Z])(?=.*[0-9])(?=.*[!@#\$%\^&\*])')).required(),
    name: Joi.string().min(2).max(50).required(),
    age: Joi.number().integer().min(18).max(120).optional()
});

const loginSchema = Joi.object({
    email: Joi.string().email().required(),
    password: Joi.string().required()
});

// Apply validation
app.post('/api/auth/register', validateRequest(userRegistrationSchema), register);
app.post('/api/auth/login', validateRequest(loginSchema), login);
```

This intermediate authentication guide covers JWT implementation, OAuth integration, session management, RBAC, password security, security headers, and input validation - essential skills for building secure web applications.
