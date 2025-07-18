
# 🚀 Advanced Express.js

## Table of Contents
- [Microservices Architecture](#microservices-architecture)
- [WebSocket Integration](#websocket-integration)
- [GraphQL with Express](#graphql-with-express)
- [Advanced Security](#advanced-security)
- [Performance Monitoring](#performance-monitoring)
- [Testing Strategies](#testing-strategies)
- [Deployment Patterns](#deployment-patterns)
- [Scaling Strategies](#scaling-strategies)

---

## 🏗️ Microservices Architecture

### Service Discovery and Communication

```javascript
const express = require('express');
const axios = require('axios');
const consul = require('consul')();

class ServiceRegistry {
    constructor() {
        this.services = new Map();
    }
    
    async register(serviceName, serviceUrl, healthCheck) {
        const serviceId = `${serviceName}-${Date.now()}`;
        
        await consul.agent.service.register({
            id: serviceId,
            name: serviceName,
            address: serviceUrl.split('://')[1].split(':')[0],
            port: parseInt(serviceUrl.split(':')[2]),
            check: {
                http: `${serviceUrl}${healthCheck}`,
                interval: '10s'
            }
        });
        
        this.services.set(serviceName, { id: serviceId, url: serviceUrl });
        return serviceId;
    }
    
    async discover(serviceName) {
        try {
            const services = await consul.health.service(serviceName);
            return services[0]?.Service || null;
        } catch (error) {
            console.error('Service discovery error:', error);
            return null;
        }
    }
}

// API Gateway implementation
class APIGateway {
    constructor() {
        this.app = express();
        this.serviceRegistry = new ServiceRegistry();
        this.setupMiddleware();
        this.setupRoutes();
    }
    
    setupMiddleware() {
        this.app.use(express.json());
        this.app.use(this.rateLimiter());
        this.app.use(this.authMiddleware());
    }
    
    setupRoutes() {
        // Dynamic service routing
        this.app.use('/api/:service/*', async (req, res) => {
            const serviceName = req.params.service;
            const path = req.params[0];
            
            try {
                const service = await this.serviceRegistry.discover(serviceName);
                if (!service) {
                    return res.status(404).json({ error: 'Service not found' });
                }
                
                const serviceUrl = `http://${service.Address}:${service.Port}`;
                const response = await axios({
                    method: req.method,
                    url: `${serviceUrl}/${path}`,
                    data: req.body,
                    params: req.query,
                    headers: {
                        'Authorization': req.headers.authorization,
                        'Content-Type': 'application/json'
                    }
                });
                
                res.status(response.status).json(response.data);
            } catch (error) {
                console.error('Gateway error:', error);
                res.status(500).json({ error: 'Internal server error' });
            }
        });
    }
    
    rateLimiter() {
        const requests = new Map();
        
        return (req, res, next) => {
            const key = req.ip;
            const now = Date.now();
            const windowMs = 60 * 1000; // 1 minute
            const maxRequests = 100;
            
            if (!requests.has(key)) {
                requests.set(key, []);
            }
            
            const userRequests = requests.get(key);
            const validRequests = userRequests.filter(time => time > now - windowMs);
            
            if (validRequests.length >= maxRequests) {
                return res.status(429).json({ error: 'Rate limit exceeded' });
            }
            
            validRequests.push(now);
            requests.set(key, validRequests);
            next();
        };
    }
    
    authMiddleware() {
        return (req, res, next) => {
            if (req.path.startsWith('/api/auth/')) {
                return next();
            }
            
            const token = req.headers.authorization;
            if (!token) {
                return res.status(401).json({ error: 'Authorization required' });
            }
            
            // Validate token with auth service
            next();
        };
    }
}
```

---

## 🔌 WebSocket Integration

### Real-time Features with Socket.io

```javascript
const express = require('express');
const http = require('http');
const socketIo = require('socket.io');
const jwt = require('jsonwebtoken');

const app = express();
const server = http.createServer(app);
const io = socketIo(server, {
    cors: {
        origin: process.env.CLIENT_URL,
        methods: ['GET', 'POST']
    }
});

// Socket authentication middleware
io.use((socket, next) => {
    const token = socket.handshake.auth.token;
    
    if (!token) {
        return next(new Error('Authentication error'));
    }
    
    jwt.verify(token, process.env.JWT_SECRET, (err, decoded) => {
        if (err) {
            return next(new Error('Authentication error'));
        }
        
        socket.userId = decoded.userId;
        socket.username = decoded.username;
        next();
    });
});

// Real-time chat implementation
class ChatService {
    constructor(io) {
        this.io = io;
        this.rooms = new Map();
        this.setupEventHandlers();
    }
    
    setupEventHandlers() {
        this.io.on('connection', (socket) => {
            console.log(`User ${socket.username} connected`);
            
            // Join room
            socket.on('join-room', (roomId) => {
                socket.join(roomId);
                
                if (!this.rooms.has(roomId)) {
                    this.rooms.set(roomId, new Set());
                }
                
                this.rooms.get(roomId).add(socket.userId);
                
                // Notify room about new user
                socket.to(roomId).emit('user-joined', {
                    userId: socket.userId,
                    username: socket.username
                });
            });
            
            // Handle messages
            socket.on('send-message', async (data) => {
                const { roomId, message } = data;
                
                try {
                    // Save message to database
                    const savedMessage = await Message.create({
                        content: message,
                        author: socket.userId,
                        room: roomId,
                        timestamp: new Date()
                    });
                    
                    // Populate author info
                    await savedMessage.populate('author', 'username avatar');
                    
                    // Broadcast to room
                    this.io.to(roomId).emit('new-message', {
                        id: savedMessage._id,
                        content: savedMessage.content,
                        author: savedMessage.author,
                        timestamp: savedMessage.timestamp
                    });
                } catch (error) {
                    socket.emit('error', { message: 'Failed to send message' });
                }
            });
            
            // Handle typing indicators
            socket.on('typing-start', (roomId) => {
                socket.to(roomId).emit('user-typing', {
                    userId: socket.userId,
                    username: socket.username
                });
            });
            
            socket.on('typing-stop', (roomId) => {
                socket.to(roomId).emit('user-stopped-typing', {
                    userId: socket.userId
                });
            });
            
            // Handle disconnection
            socket.on('disconnect', () => {
                console.log(`User ${socket.username} disconnected`);
                
                // Remove user from all rooms
                this.rooms.forEach((users, roomId) => {
                    if (users.has(socket.userId)) {
                        users.delete(socket.userId);
                        socket.to(roomId).emit('user-left', {
                            userId: socket.userId,
                            username: socket.username
                        });
                    }
                });
            });
        });
    }
}

// Initialize chat service
const chatService = new ChatService(io);

// REST API for chat history
app.get('/api/rooms/:roomId/messages', async (req, res) => {
    const { roomId } = req.params;
    const { page = 1, limit = 50 } = req.query;
    
    try {
        const messages = await Message.find({ room: roomId })
            .populate('author', 'username avatar')
            .sort({ timestamp: -1 })
            .limit(limit * 1)
            .skip((page - 1) * limit);
        
        res.json(messages.reverse());
    } catch (error) {
        res.status(500).json({ error: 'Failed to fetch messages' });
    }
});
```

---

## 🎯 GraphQL with Express

### GraphQL Schema and Resolvers

```javascript
const { ApolloServer } = require('apollo-server-express');
const { gql } = require('apollo-server-express');

// GraphQL schema
const typeDefs = gql`
    type User {
        id: ID!
        username: String!
        email: String!
        posts: [Post!]!
        createdAt: String!
    }
    
    type Post {
        id: ID!
        title: String!
        content: String!
        author: User!
        comments: [Comment!]!
        createdAt: String!
        updatedAt: String!
    }
    
    type Comment {
        id: ID!
        content: String!
        author: User!
        post: Post!
        createdAt: String!
    }
    
    type Query {
        users: [User!]!
        user(id: ID!): User
        posts: [Post!]!
        post(id: ID!): Post
        searchPosts(query: String!): [Post!]!
    }
    
    type Mutation {
        createUser(input: CreateUserInput!): User!
        createPost(input: CreatePostInput!): Post!
        updatePost(id: ID!, input: UpdatePostInput!): Post!
        deletePost(id: ID!): Boolean!
        createComment(input: CreateCommentInput!): Comment!
    }
    
    type Subscription {
        postAdded: Post!
        commentAdded(postId: ID!): Comment!
    }
    
    input CreateUserInput {
        username: String!
        email: String!
        password: String!
    }
    
    input CreatePostInput {
        title: String!
        content: String!
    }
    
    input UpdatePostInput {
        title: String
        content: String
    }
    
    input CreateCommentInput {
        content: String!
        postId: ID!
    }
`;

// GraphQL resolvers
const resolvers = {
    Query: {
        users: async () => {
            return await User.find().sort({ createdAt: -1 });
        },
        
        user: async (_, { id }) => {
            return await User.findById(id);
        },
        
        posts: async (_, args, { user }) => {
            if (!user) throw new Error('Authentication required');
            return await Post.find().populate('author').sort({ createdAt: -1 });
        },
        
        post: async (_, { id }) => {
            return await Post.findById(id).populate('author');
        },
        
        searchPosts: async (_, { query }) => {
            return await Post.find({
                $or: [
                    { title: { $regex: query, $options: 'i' } },
                    { content: { $regex: query, $options: 'i' } }
                ]
            }).populate('author');
        }
    },
    
    Mutation: {
        createUser: async (_, { input }) => {
            const hashedPassword = await bcrypt.hash(input.password, 12);
            
            const user = new User({
                username: input.username,
                email: input.email,
                password: hashedPassword
            });
            
            return await user.save();
        },
        
        createPost: async (_, { input }, { user }) => {
            if (!user) throw new Error('Authentication required');
            
            const post = new Post({
                title: input.title,
                content: input.content,
                author: user.id
            });
            
            const savedPost = await post.save();
            await savedPost.populate('author');
            
            // Publish subscription
            pubsub.publish('POST_ADDED', { postAdded: savedPost });
            
            return savedPost;
        },
        
        updatePost: async (_, { id, input }, { user }) => {
            if (!user) throw new Error('Authentication required');
            
            const post = await Post.findById(id);
            if (!post) throw new Error('Post not found');
            
            if (post.author.toString() !== user.id) {
                throw new Error('Not authorized');
            }
            
            Object.assign(post, input);
            post.updatedAt = new Date();
            
            return await post.save();
        },
        
        deletePost: async (_, { id }, { user }) => {
            if (!user) throw new Error('Authentication required');
            
            const post = await Post.findById(id);
            if (!post) throw new Error('Post not found');
            
            if (post.author.toString() !== user.id) {
                throw new Error('Not authorized');
            }
            
            await Post.findByIdAndDelete(id);
            return true;
        },
        
        createComment: async (_, { input }, { user }) => {
            if (!user) throw new Error('Authentication required');
            
            const comment = new Comment({
                content: input.content,
                author: user.id,
                post: input.postId
            });
            
            const savedComment = await comment.save();
            await savedComment.populate('author');
            
            // Publish subscription
            pubsub.publish('COMMENT_ADDED', { 
                commentAdded: savedComment,
                postId: input.postId 
            });
            
            return savedComment;
        }
    },
    
    Subscription: {
        postAdded: {
            subscribe: () => pubsub.asyncIterator(['POST_ADDED'])
        },
        
        commentAdded: {
            subscribe: (_, { postId }) => 
                pubsub.asyncIterator([`COMMENT_ADDED_${postId}`])
        }
    },
    
    User: {
        posts: async (user) => {
            return await Post.find({ author: user.id }).sort({ createdAt: -1 });
        }
    },
    
    Post: {
        author: async (post) => {
            return await User.findById(post.author);
        },
        
        comments: async (post) => {
            return await Comment.find({ post: post.id })
                .populate('author')
                .sort({ createdAt: -1 });
        }
    }
};

// Apollo Server setup
const server = new ApolloServer({
    typeDefs,
    resolvers,
    context: ({ req }) => {
        const token = req.headers.authorization || '';
        
        try {
            const user = jwt.verify(token.replace('Bearer ', ''), process.env.JWT_SECRET);
            return { user };
        } catch {
            return {};
        }
    }
});

// Apply GraphQL middleware
server.applyMiddleware({ app, path: '/graphql' });
```

---

## 🛡️ Advanced Security

### OAuth 2.0 and JWT Implementation

```javascript
const passport = require('passport');
const GoogleStrategy = require('passport-google-oauth20').Strategy;
const JwtStrategy = require('passport-jwt').Strategy;
const ExtractJwt = require('passport-jwt').ExtractJwt;

// JWT Strategy
passport.use(new JwtStrategy({
    jwtFromRequest: ExtractJwt.fromAuthHeaderAsBearerToken(),
    secretOrKey: process.env.JWT_SECRET
}, async (jwtPayload, done) => {
    try {
        const user = await User.findById(jwtPayload.sub);
        if (user) {
            return done(null, user);
        }
        return done(null, false);
    } catch (error) {
        return done(error, false);
    }
}));

// Google OAuth Strategy
passport.use(new GoogleStrategy({
    clientID: process.env.GOOGLE_CLIENT_ID,
    clientSecret: process.env.GOOGLE_CLIENT_SECRET,
    callbackURL: '/auth/google/callback'
}, async (accessToken, refreshToken, profile, done) => {
    try {
        let user = await User.findOne({ googleId: profile.id });
        
        if (!user) {
            user = await User.create({
                googleId: profile.id,
                username: profile.displayName,
                email: profile.emails[0].value,
                avatar: profile.photos[0].value
            });
        }
        
        return done(null, user);
    } catch (error) {
        return done(error, null);
    }
}));

// Security middleware
const securityMiddleware = {
    // CSRF protection
    csrfProtection: (req, res, next) => {
        const token = req.headers['x-csrf-token'];
        const sessionToken = req.session.csrfToken;
        
        if (!token || token !== sessionToken) {
            return res.status(403).json({ error: 'Invalid CSRF token' });
        }
        
        next();
    },
    
    // SQL injection protection
    sqlInjectionProtection: (req, res, next) => {
        const suspiciousPatterns = [
            /('|(\\\')|(%27)|(%3D)|(;)|(%3B)|(\\))/i,
            /((\%3D)|(=))[^\n]*((\%27)|(\')|(\-\-)|(%3B)|(;))/i,
            /\w*((\%27)|(\'))((\%6F)|o|(\%4F))((\%72)|r|(\%52))/i,
            /((\%27)|(\'))union/i
        ];
        
        const checkValue = (value) => {
            if (typeof value === 'string') {
                return suspiciousPatterns.some(pattern => pattern.test(value));
            }
            return false;
        };
        
        const checkObject = (obj) => {
            for (let key in obj) {
                if (checkValue(obj[key]) || checkValue(key)) {
                    return true;
                }
                if (typeof obj[key] === 'object' && obj[key] !== null) {
                    if (checkObject(obj[key])) {
                        return true;
                    }
                }
            }
            return false;
        };
        
        if (checkObject(req.query) || checkObject(req.body) || checkObject(req.params)) {
            return res.status(400).json({ error: 'Suspicious input detected' });
        }
        
        next();
    }
};

// Advanced authentication routes
app.get('/auth/google', passport.authenticate('google', {
    scope: ['profile', 'email']
}));

app.get('/auth/google/callback',
    passport.authenticate('google', { failureRedirect: '/login' }),
    (req, res) => {
        const token = jwt.sign(
            { sub: req.user.id, username: req.user.username },
            process.env.JWT_SECRET,
            { expiresIn: '1h' }
        );
        
        res.redirect(`${process.env.CLIENT_URL}/auth/callback?token=${token}`);
    }
);
```

---

## 📊 Performance Monitoring

### Application Performance Monitoring

```javascript
const prometheus = require('prom-client');

// Create metrics
const httpRequestsTotal = new prometheus.Counter({
    name: 'http_requests_total',
    help: 'Total number of HTTP requests',
    labelNames: ['method', 'route', 'status_code']
});

const httpRequestDuration = new prometheus.Histogram({
    name: 'http_request_duration_seconds',
    help: 'Duration of HTTP requests in seconds',
    labelNames: ['method', 'route', 'status_code'],
    buckets: [0.1, 0.5, 1, 2, 5]
});

const activeConnections = new prometheus.Gauge({
    name: 'active_connections',
    help: 'Number of active connections'
});

// Monitoring middleware
const monitoringMiddleware = (req, res, next) => {
    const startTime = Date.now();
    
    res.on('finish', () => {
        const duration = (Date.now() - startTime) / 1000;
        const labels = {
            method: req.method,
            route: req.route?.path || req.path,
            status_code: res.statusCode
        };
        
        httpRequestsTotal.inc(labels);
        httpRequestDuration.observe(labels, duration);
    });
    
    next();
};

// Health check endpoint
app.get('/health', (req, res) => {
    const healthCheck = {
        uptime: process.uptime(),
        message: 'OK',
        timestamp: Date.now(),
        checks: {
            database: 'connected',
            redis: 'connected',
            memory: {
                usage: process.memoryUsage(),
                percentage: Math.round((process.memoryUsage().rss / 1024 / 1024) * 100) / 100
            }
        }
    };
    
    res.status(200).json(healthCheck);
});

// Metrics endpoint
app.get('/metrics', (req, res) => {
    res.set('Content-Type', prometheus.register.contentType);
    res.end(prometheus.register.metrics());
});

// Error tracking
class ErrorTracker {
    constructor() {
        this.errorCounts = new Map();
    }
    
    track(error, req) {
        const key = `${error.name}:${error.message}`;
        const count = this.errorCounts.get(key) || 0;
        this.errorCounts.set(key, count + 1);
        
        // Log error details
        console.error({
            error: error.message,
            stack: error.stack,
            url: req.url,
            method: req.method,
            timestamp: new Date().toISOString(),
            count: count + 1
        });
        
        // Send to monitoring service
        this.sendToMonitoring(error, req);
    }
    
    sendToMonitoring(error, req) {
        // Implementation for sending to monitoring service
        // (e.g., Sentry, New Relic, custom logging service)
    }
}

const errorTracker = new ErrorTracker();
```

---

## 🧪 Testing Strategies

### Comprehensive Testing Suite

```javascript
const request = require('supertest');
const app = require('../app');
const mongoose = require('mongoose');
const User = require('../models/User');
const jwt = require('jsonwebtoken');

describe('Advanced API Tests', () => {
    let server;
    let authToken;
    let testUser;
    
    beforeAll(async () => {
        // Connect to test database
        await mongoose.connect(process.env.TEST_DATABASE_URL);
        server = app.listen(0);
        
        // Create test user
        testUser = await User.create({
            username: 'testuser',
            email: 'test@example.com',
            password: 'hashedpassword'
        });
        
        authToken = jwt.sign(
            { sub: testUser._id, username: testUser.username },
            process.env.JWT_SECRET
        );
    });
    
    afterAll(async () => {
        await mongoose.connection.dropDatabase();
        await mongoose.connection.close();
        server.close();
    });
    
    describe('Authentication Flow', () => {
        test('should authenticate valid JWT token', async () => {
            const response = await request(app)
                .get('/api/protected')
                .set('Authorization', `Bearer ${authToken}`)
                .expect(200);
            
            expect(response.body.user.id).toBe(testUser._id.toString());
        });
        
        test('should reject invalid JWT token', async () => {
            await request(app)
                .get('/api/protected')
                .set('Authorization', 'Bearer invalid-token')
                .expect(401);
        });
    });
    
    describe('Rate Limiting', () => {
        test('should enforce rate limits', async () => {
            const promises = Array(101).fill().map(() =>
                request(app)
                    .get('/api/users')
                    .set('Authorization', `Bearer ${authToken}`)
            );
            
            const responses = await Promise.all(promises);
            const rateLimitedResponses = responses.filter(r => r.status === 429);
            
            expect(rateLimitedResponses.length).toBeGreaterThan(0);
        });
    });
    
    describe('WebSocket Integration', () => {
        test('should handle WebSocket connections', (done) => {
            const io = require('socket.io-client');
            const client = io('http://localhost:3000', {
                auth: { token: authToken }
            });
            
            client.on('connect', () => {
                client.emit('join-room', 'test-room');
                
                client.on('user-joined', (data) => {
                    expect(data.userId).toBe(testUser._id.toString());
                    client.disconnect();
                    done();
                });
            });
        });
    });
    
    describe('Database Transactions', () => {
        test('should handle transaction rollback on error', async () => {
            const session = await mongoose.startSession();
            
            try {
                await session.withTransaction(async () => {
                    await User.create([{ username: 'user1' }], { session });
                    await User.create([{ username: 'user1' }], { session }); // Duplicate error
                });
            } catch (error) {
                expect(error.name).toBe('MongoError');
            }
            
            const userCount = await User.countDocuments({ username: 'user1' });
            expect(userCount).toBe(0);
            
            await session.endSession();
        });
    });
});

// Load testing
const loadTest = async () => {
    const startTime = Date.now();
    const concurrentRequests = 100;
    const totalRequests = 1000;
    
    const results = {
        success: 0,
        errors: 0,
        responseTimes: []
    };
    
    for (let i = 0; i < totalRequests; i += concurrentRequests) {
        const batch = Array(concurrentRequests).fill().map(async () => {
            const requestStart = Date.now();
            
            try {
                const response = await request(app)
                    .get('/api/posts')
                    .set('Authorization', `Bearer ${authToken}`);
                
                const responseTime = Date.now() - requestStart;
                results.responseTimes.push(responseTime);
                
                if (response.status === 200) {
                    results.success++;
                } else {
                    results.errors++;
                }
            } catch (error) {
                results.errors++;
            }
        });
        
        await Promise.all(batch);
    }
    
    const totalTime = Date.now() - startTime;
    const averageResponseTime = results.responseTimes.reduce((a, b) => a + b, 0) / results.responseTimes.length;
    
    console.log({
        totalRequests,
        totalTime,
        requestsPerSecond: totalRequests / (totalTime / 1000),
        averageResponseTime,
        successRate: (results.success / totalRequests) * 100
    });
};
```

---

## 📚 Summary

### Advanced Express.js Capabilities:
- **Microservices**: Service discovery, API gateway, inter-service communication
- **Real-time**: WebSocket integration, live updates, subscriptions
- **GraphQL**: Schema definition, resolvers, subscriptions
- **Advanced Security**: OAuth 2.0, JWT, CSRF protection, input validation
- **Monitoring**: Prometheus metrics, health checks, error tracking
- **Testing**: Unit, integration, load testing strategies
- **Performance**: Caching, optimization, profiling

### Enterprise Patterns:
1. **Implement circuit breakers** for resilience
2. **Use distributed tracing** for debugging
3. **Implement graceful shutdown** handling
4. **Use blue-green deployments** for zero downtime
5. **Implement comprehensive logging** with structured data
6. **Use event sourcing** for audit trails
7. **Implement CQRS** for complex domains
8. **Use message queues** for async processing

### Next Steps:
1. Study event-driven architectures
2. Learn about distributed systems patterns
3. Explore serverless computing
4. Study container orchestration
5. Learn about service mesh architectures
