
# 🔐 Authentication & Security Basics

## Table of Contents
- [What is Authentication?](#what-is-authentication)
- [Authentication vs Authorization](#authentication-vs-authorization)
- [Password Security](#password-security)
- [Session Management](#session-management)
- [JWT Tokens](#jwt-tokens)
- [OAuth](#oauth)
- [Security Best Practices](#security-best-practices)
- [Common Vulnerabilities](#common-vulnerabilities)

---

## 🎯 What is Authentication?

**Authentication** is the process of verifying the identity of a user, device, or system.

### Types of Authentication:
- **Something you know** (passwords, PINs)
- **Something you have** (tokens, cards)
- **Something you are** (biometrics)

---

## 🔑 Authentication vs Authorization

```
┌─────────────────┐    ┌─────────────────┐
│  Authentication │    │  Authorization  │
│                 │    │                 │
│ "Who are you?"  │    │ "What can you   │
│                 │    │  do?"           │
│ Login process   │    │ Permissions     │
└─────────────────┘    └─────────────────┘
```

### Examples:
- **Authentication**: User logs in with email/password
- **Authorization**: User can edit only their own posts

---

## 🔒 Password Security

### Password Hashing

```javascript
const bcrypt = require('bcrypt');

// Hash password before storing
async function hashPassword(password) {
    const saltRounds = 12;
    return await bcrypt.hash(password, saltRounds);
}

// Verify password during login
async function verifyPassword(password, hashedPassword) {
    return await bcrypt.compare(password, hashedPassword);
}

// Example usage
const plainPassword = "userPassword123";
const hashedPassword = await hashPassword(plainPassword);
console.log("Hashed:", hashedPassword);

const isValid = await verifyPassword(plainPassword, hashedPassword);
console.log("Valid:", isValid); // true
```

### Password Requirements

```javascript
function validatePassword(password) {
    const requirements = {
        minLength: password.length >= 8,
        hasUppercase: /[A-Z]/.test(password),
        hasLowercase: /[a-z]/.test(password),
        hasNumber: /\d/.test(password),
        hasSpecialChar: /[!@#$%^&*(),.?":{}|<>]/.test(password)
    };
    
    const isValid = Object.values(requirements).every(req => req);
    
    return {
        isValid,
        requirements,
        score: Object.values(requirements).filter(Boolean).length
    };
}
```

---

## 🍪 Session Management

### Server-Side Sessions

```javascript
const express = require('express');
const session = require('express-session');
const MongoStore = require('connect-mongo');

const app = express();

app.use(session({
    secret: process.env.SESSION_SECRET,
    resave: false,
    saveUninitialized: false,
    store: MongoStore.create({
        mongoUrl: process.env.MONGODB_URI
    }),
    cookie: {
        secure: process.env.NODE_ENV === 'production',
        httpOnly: true,
        maxAge: 1000 * 60 * 60 * 24 // 24 hours
    }
}));

// Login route
app.post('/login', async (req, res) => {
    const { email, password } = req.body;
    
    // Verify user credentials
    const user = await User.findOne({ email });
    if (!user || !await verifyPassword(password, user.password)) {
        return res.status(401).json({ error: 'Invalid credentials' });
    }
    
    // Create session
    req.session.userId = user._id;
    req.session.email = user.email;
    
    res.json({ message: 'Login successful', user: { id: user._id, email: user.email } });
});

// Logout route
app.post('/logout', (req, res) => {
    req.session.destroy((err) => {
        if (err) {
            return res.status(500).json({ error: 'Could not log out' });
        }
        res.clearCookie('connect.sid');
        res.json({ message: 'Logout successful' });
    });
});
```

---

## 🎫 JWT Tokens

### JWT Structure

```
Header.Payload.Signature
eyJhbGciOiJIUzI1NiIsInR5cCI6IkpXVCJ9.eyJzdWIiOiIxMjM0NTY3ODkwIiwibmFtZSI6IkpvaG4gRG9lIiwiaWF0IjoxNTE2MjM5MDIyfQ.SflKxwRJSMeKKF2QT4fwpMeJf36POk6yJV_adQssw5c
```

### JWT Implementation

```javascript
const jwt = require('jsonwebtoken');

// Generate JWT token
function generateToken(payload) {
    return jwt.sign(payload, process.env.JWT_SECRET, {
        expiresIn: '24h',
        issuer: 'your-app-name'
    });
}

// Verify JWT token
function verifyToken(token) {
    try {
        return jwt.verify(token, process.env.JWT_SECRET);
    } catch (error) {
        throw new Error('Invalid token');
    }
}

// Middleware to protect routes
function authenticateToken(req, res, next) {
    const authHeader = req.headers['authorization'];
    const token = authHeader && authHeader.split(' ')[1]; // Bearer TOKEN
    
    if (!token) {
        return res.status(401).json({ error: 'Access token required' });
    }
    
    try {
        const decoded = verifyToken(token);
        req.user = decoded;
        next();
    } catch (error) {
        return res.status(403).json({ error: 'Invalid token' });
    }
}

// Login with JWT
app.post('/login', async (req, res) => {
    const { email, password } = req.body;
    
    const user = await User.findOne({ email });
    if (!user || !await verifyPassword(password, user.password)) {
        return res.status(401).json({ error: 'Invalid credentials' });
    }
    
    const token = generateToken({
        userId: user._id,
        email: user.email
    });
    
    res.json({
        message: 'Login successful',
        token,
        user: { id: user._id, email: user.email }
    });
});

// Protected route
app.get('/profile', authenticateToken, (req, res) => {
    res.json({ user: req.user });
});
```

---

## 🌐 OAuth

### OAuth 2.0 Flow

```javascript
const passport = require('passport');
const GoogleStrategy = require('passport-google-oauth20').Strategy;

passport.use(new GoogleStrategy({
    clientID: process.env.GOOGLE_CLIENT_ID,
    clientSecret: process.env.GOOGLE_CLIENT_SECRET,
    callbackURL: "/auth/google/callback"
}, async (accessToken, refreshToken, profile, done) => {
    try {
        // Check if user already exists
        let user = await User.findOne({ googleId: profile.id });
        
        if (user) {
            return done(null, user);
        }
        
        // Create new user
        user = await User.create({
            googleId: profile.id,
            name: profile.displayName,
            email: profile.emails[0].value,
            avatar: profile.photos[0].value
        });
        
        return done(null, user);
    } catch (error) {
        return done(error, null);
    }
}));

// Routes
app.get('/auth/google',
    passport.authenticate('google', { scope: ['profile', 'email'] })
);

app.get('/auth/google/callback',
    passport.authenticate('google', { failureRedirect: '/login' }),
    (req, res) => {
        // Generate JWT token
        const token = generateToken({
            userId: req.user._id,
            email: req.user.email
        });
        
        // Redirect with token
        res.redirect(`${process.env.CLIENT_URL}/auth/success?token=${token}`);
    }
);
```

---

## 🛡️ Security Best Practices

### Input Validation & Sanitization

```javascript
const validator = require('validator');
const rateLimit = require('express-rate-limit');
const helmet = require('helmet');

// Security middleware
app.use(helmet());

// Rate limiting
const loginLimiter = rateLimit({
    windowMs: 15 * 60 * 1000, // 15 minutes
    max: 5, // 5 attempts
    message: 'Too many login attempts, please try again later',
    standardHeaders: true,
    legacyHeaders: false,
});

app.use('/login', loginLimiter);

// Input validation
function validateUserInput(req, res, next) {
    const { email, password } = req.body;
    
    // Validate email
    if (!email || !validator.isEmail(email)) {
        return res.status(400).json({ error: 'Valid email required' });
    }
    
    // Validate password
    if (!password || password.length < 8) {
        return res.status(400).json({ error: 'Password must be at least 8 characters' });
    }
    
    // Sanitize email
    req.body.email = validator.normalizeEmail(email);
    
    next();
}
```

### CORS Configuration

```javascript
const cors = require('cors');

const corsOptions = {
    origin: process.env.NODE_ENV === 'production' 
        ? ['https://yourdomain.com']
        : ['http://localhost:3000', 'http://localhost:5000'],
    credentials: true,
    optionsSuccessStatus: 200,
    methods: ['GET', 'POST', 'PUT', 'DELETE'],
    allowedHeaders: ['Content-Type', 'Authorization']
};

app.use(cors(corsOptions));
```

---

## ⚠️ Common Vulnerabilities

### SQL Injection Prevention

```javascript
// ❌ Vulnerable to SQL injection
const query = `SELECT * FROM users WHERE email = '${email}'`;

// ✅ Safe with parameterized queries
const query = 'SELECT * FROM users WHERE email = ?';
db.query(query, [email], (err, results) => {
    // Handle results
});
```

### XSS Prevention

```javascript
const xss = require('xss');

// Sanitize user input
function sanitizeInput(input) {
    return xss(input, {
        whiteList: {}, // No HTML tags allowed
        stripIgnoreTag: true,
        stripIgnoreTagBody: ['script']
    });
}

// Use in routes
app.post('/comment', (req, res) => {
    const sanitizedComment = sanitizeInput(req.body.comment);
    // Save sanitized comment
});
```

### CSRF Protection

```javascript
const csrf = require('csurf');

const csrfProtection = csrf({
    cookie: {
        httpOnly: true,
        secure: process.env.NODE_ENV === 'production',
        sameSite: 'strict'
    }
});

app.use(csrfProtection);

// Provide CSRF token to client
app.get('/csrf-token', (req, res) => {
    res.json({ csrfToken: req.csrfToken() });
});
```

---

## 🧪 Testing Authentication

```javascript
const request = require('supertest');
const app = require('../app');

describe('Authentication', () => {
    test('should register new user', async () => {
        const response = await request(app)
            .post('/register')
            .send({
                email: 'test@example.com',
                password: 'TestPassword123!'
            });
        
        expect(response.status).toBe(201);
        expect(response.body.user.email).toBe('test@example.com');
    });
    
    test('should login with valid credentials', async () => {
        const response = await request(app)
            .post('/login')
            .send({
                email: 'test@example.com',
                password: 'TestPassword123!'
            });
        
        expect(response.status).toBe(200);
        expect(response.body.token).toBeDefined();
    });
    
    test('should reject invalid credentials', async () => {
        const response = await request(app)
            .post('/login')
            .send({
                email: 'test@example.com',
                password: 'wrongpassword'
            });
        
        expect(response.status).toBe(401);
    });
});
```

---

## 📚 Summary

### Key Concepts:
- **Password hashing** with bcrypt
- **Session management** for stateful auth
- **JWT tokens** for stateless auth
- **OAuth** for third-party authentication
- **Security best practices** (HTTPS, CORS, rate limiting)
- **Vulnerability prevention** (XSS, CSRF, SQL injection)

### Next Steps:
1. Implement authentication in your projects
2. Learn about multi-factor authentication (MFA)
3. Explore advanced security patterns
4. Study OAuth providers (Google, GitHub, Facebook)
5. Practice security testing and penetration testing

Remember: Security is not a feature, it's a foundation!
