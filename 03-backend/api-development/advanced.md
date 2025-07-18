
# 🚀 Advanced API Development

## Table of Contents
- [GraphQL APIs](#graphql-apis)
- [Microservices API Design](#microservices-api-design)
- [Real-time APIs](#real-time-apis)
- [API Gateway Implementation](#api-gateway-implementation)
- [Advanced Security](#advanced-security)
- [Performance Optimization](#performance-optimization)
- [API Monitoring and Analytics](#api-monitoring-and-analytics)
- [Event-Driven Architecture](#event-driven-architecture)
- [API Documentation and Testing](#api-documentation-and-testing)

---

## 🎯 GraphQL APIs

### GraphQL Server Setup

```javascript
const { ApolloServer, gql } = require('apollo-server-express');
const { GraphQLScalarType } = require('graphql');
const { Kind } = require('graphql/language');

// Custom scalar types
const DateTimeType = new GraphQLScalarType({
    name: 'DateTime',
    description: 'A valid date time value.',
    serialize: (value) => value.toISOString(),
    parseValue: (value) => new Date(value),
    parseLiteral: (ast) => {
        if (ast.kind === Kind.STRING) {
            return new Date(ast.value);
        }
        return null;
    }
});

// GraphQL Schema
const typeDefs = gql`
    scalar DateTime
    
    type User {
        id: ID!
        email: String!
        name: String!
        posts: [Post!]!
        profile: UserProfile
        createdAt: DateTime!
        updatedAt: DateTime!
    }
    
    type UserProfile {
        bio: String
        avatar: String
        website: String
        location: String
    }
    
    type Post {
        id: ID!
        title: String!
        content: String!
        published: Boolean!
        author: User!
        comments: [Comment!]!
        tags: [String!]!
        createdAt: DateTime!
        updatedAt: DateTime!
    }
    
    type Comment {
        id: ID!
        content: String!
        author: User!
        post: Post!
        createdAt: DateTime!
    }
    
    input CreatePostInput {
        title: String!
        content: String!
        published: Boolean = false
        tags: [String!] = []
    }
    
    input UpdatePostInput {
        title: String
        content: String
        published: Boolean
        tags: [String!]
    }
    
    input UserFilter {
        search: String
        isActive: Boolean
        createdAfter: DateTime
    }
    
    type Query {
        # User queries
        me: User
        user(id: ID!): User
        users(filter: UserFilter, limit: Int = 10, offset: Int = 0): [User!]!
        
        # Post queries
        post(id: ID!): Post
        posts(authorId: ID, published: Boolean, limit: Int = 10, offset: Int = 0): [Post!]!
        searchPosts(query: String!, limit: Int = 10): [Post!]!
        
        # Analytics
        postStats(authorId: ID): PostStats
    }
    
    type Mutation {
        # User mutations
        updateProfile(input: UpdateProfileInput!): User!
        
        # Post mutations
        createPost(input: CreatePostInput!): Post!
        updatePost(id: ID!, input: UpdatePostInput!): Post!
        deletePost(id: ID!): Boolean!
        publishPost(id: ID!): Post!
        
        # Comment mutations
        addComment(postId: ID!, content: String!): Comment!
        deleteComment(id: ID!): Boolean!
    }
    
    type Subscription {
        postAdded(authorId: ID): Post!
        commentAdded(postId: ID!): Comment!
        userOnline: User!
    }
    
    type PostStats {
        totalPosts: Int!
        publishedPosts: Int!
        totalComments: Int!
        avgCommentsPerPost: Float!
    }
`;

// Resolvers
const resolvers = {
    DateTime: DateTimeType,
    
    Query: {
        me: async (parent, args, context) => {
            if (!context.user) {
                throw new AuthenticationError('Not authenticated');
            }
            return await User.findById(context.user.id).populate('profile');
        },
        
        user: async (parent, { id }) => {
            return await User.findById(id).populate('profile');
        },
        
        users: async (parent, { filter, limit, offset }) => {
            const query = {};
            
            if (filter) {
                if (filter.search) {
                    query.$or = [
                        { name: { $regex: filter.search, $options: 'i' } },
                        { email: { $regex: filter.search, $options: 'i' } }
                    ];
                }
                if (filter.isActive !== undefined) {
                    query.isActive = filter.isActive;
                }
                if (filter.createdAfter) {
                    query.createdAt = { $gte: filter.createdAfter };
                }
            }
            
            return await User.find(query)
                .populate('profile')
                .limit(limit)
                .skip(offset)
                .sort({ createdAt: -1 });
        },
        
        posts: async (parent, { authorId, published, limit, offset }) => {
            const query = {};
            if (authorId) query.author = authorId;
            if (published !== undefined) query.published = published;
            
            return await Post.find(query)
                .populate('author')
                .limit(limit)
                .skip(offset)
                .sort({ createdAt: -1 });
        },
        
        searchPosts: async (parent, { query, limit }) => {
            return await Post.find({
                $or: [
                    { title: { $regex: query, $options: 'i' } },
                    { content: { $regex: query, $options: 'i' } },
                    { tags: { $in: [new RegExp(query, 'i')] } }
                ],
                published: true
            })
            .populate('author')
            .limit(limit)
            .sort({ createdAt: -1 });
        }
    },
    
    Mutation: {
        createPost: async (parent, { input }, context) => {
            if (!context.user) {
                throw new AuthenticationError('Not authenticated');
            }
            
            const post = new Post({
                ...input,
                author: context.user.id
            });
            
            await post.save();
            await post.populate('author');
            
            // Publish subscription
            context.pubsub.publish('POST_ADDED', { 
                postAdded: post,
                authorId: context.user.id
            });
            
            return post;
        },
        
        updatePost: async (parent, { id, input }, context) => {
            const post = await Post.findById(id);
            
            if (!post) {
                throw new UserInputError('Post not found');
            }
            
            if (post.author.toString() !== context.user.id && context.user.role !== 'admin') {
                throw new ForbiddenError('Not authorized to update this post');
            }
            
            Object.assign(post, input);
            await post.save();
            await post.populate('author');
            
            return post;
        }
    },
    
    Subscription: {
        postAdded: {
            subscribe: (parent, { authorId }, context) => {
                if (authorId) {
                    return context.pubsub.asyncIterator([`POST_ADDED_${authorId}`]);
                }
                return context.pubsub.asyncIterator(['POST_ADDED']);
            }
        },
        
        commentAdded: {
            subscribe: (parent, { postId }, context) => {
                return context.pubsub.asyncIterator([`COMMENT_ADDED_${postId}`]);
            }
        }
    },
    
    // Field resolvers
    User: {
        posts: async (parent) => {
            return await Post.find({ author: parent.id });
        }
    },
    
    Post: {
        author: async (parent) => {
            return await User.findById(parent.author);
        },
        
        comments: async (parent) => {
            return await Comment.find({ post: parent.id }).populate('author');
        }
    }
};

// Create Apollo Server
const server = new ApolloServer({
    typeDefs,
    resolvers,
    context: ({ req, connection }) => {
        if (connection) {
            return connection.context;
        }
        
        return {
            user: req.user,
            pubsub: req.app.locals.pubsub
        };
    },
    subscriptions: {
        onConnect: (connectionParams, webSocket) => {
            // Authenticate subscription connection
            const token = connectionParams.authorization;
            if (token) {
                const user = jwt.verify(token, process.env.JWT_SECRET);
                return { user };
            }
            throw new Error('Authentication required for subscriptions');
        }
    }
});
```

---

## 🏗️ Microservices API Design

### Service Mesh Implementation

```javascript
// API Gateway with Service Discovery
class ServiceMesh {
    constructor() {
        this.services = new Map();
        this.healthChecks = new Map();
        this.loadBalancer = new LoadBalancer();
    }
    
    registerService(name, instances) {
        this.services.set(name, instances);
        this.startHealthCheck(name, instances);
    }
    
    async callService(serviceName, endpoint, options = {}) {
        const instances = this.getHealthyInstances(serviceName);
        if (!instances.length) {
            throw new Error(`No healthy instances for service: ${serviceName}`);
        }
        
        const instance = this.loadBalancer.selectInstance(instances);
        const url = `http://${instance.host}:${instance.port}${endpoint}`;
        
        return await this.makeRequest(url, {
            ...options,
            headers: {
                'X-Service-Name': 'api-gateway',
                'X-Request-ID': generateRequestId(),
                'X-Correlation-ID': options.correlationId || generateCorrelationId(),
                ...options.headers
            }
        });
    }
    
    async makeRequest(url, options) {
        const timeout = options.timeout || 5000;
        const retries = options.retries || 3;
        
        for (let attempt = 1; attempt <= retries; attempt++) {
            try {
                const response = await fetch(url, {
                    ...options,
                    timeout
                });
                
                if (!response.ok) {
                    throw new Error(`HTTP ${response.status}: ${response.statusText}`);
                }
                
                return await response.json();
            } catch (error) {
                if (attempt === retries) throw error;
                
                // Exponential backoff
                await new Promise(resolve => 
                    setTimeout(resolve, Math.pow(2, attempt) * 1000)
                );
            }
        }
    }
    
    getHealthyInstances(serviceName) {
        const instances = this.services.get(serviceName) || [];
        return instances.filter(instance => 
            this.healthChecks.get(`${serviceName}:${instance.id}`)
        );
    }
    
    startHealthCheck(serviceName, instances) {
        instances.forEach(instance => {
            const key = `${serviceName}:${instance.id}`;
            
            setInterval(async () => {
                try {
                    const response = await fetch(
                        `http://${instance.host}:${instance.port}/health`,
                        { timeout: 5000 }
                    );
                    
                    this.healthChecks.set(key, response.ok);
                } catch (error) {
                    this.healthChecks.set(key, false);
                }
            }, 30000);
        });
    }
}

// Circuit Breaker Pattern
class CircuitBreaker {
    constructor(options = {}) {
        this.state = 'CLOSED';
        this.failureCount = 0;
        this.successCount = 0;
        this.nextAttempt = Date.now();
        
        this.failureThreshold = options.failureThreshold || 5;
        this.successThreshold = options.successThreshold || 2;
        this.timeout = options.timeout || 60000;
    }
    
    async call(fn) {
        if (this.state === 'OPEN') {
            if (Date.now() < this.nextAttempt) {
                throw new Error('Circuit breaker is OPEN');
            }
            this.state = 'HALF_OPEN';
        }
        
        try {
            const result = await fn();
            this.onSuccess();
            return result;
        } catch (error) {
            this.onFailure();
            throw error;
        }
    }
    
    onSuccess() {
        this.failureCount = 0;
        
        if (this.state === 'HALF_OPEN') {
            this.successCount++;
            if (this.successCount >= this.successThreshold) {
                this.state = 'CLOSED';
                this.successCount = 0;
            }
        }
    }
    
    onFailure() {
        this.failureCount++;
        
        if (this.failureCount >= this.failureThreshold) {
            this.state = 'OPEN';
            this.nextAttempt = Date.now() + this.timeout;
        }
    }
}
```

---

## ⚡ Real-time APIs

### WebSocket API Implementation

```javascript
const WebSocket = require('ws');
const jwt = require('jsonwebtoken');

class WebSocketAPI {
    constructor(server) {
        this.wss = new WebSocket.Server({ 
            server,
            verifyClient: this.verifyClient.bind(this)
        });
        
        this.connections = new Map();
        this.rooms = new Map();
        this.messageHandlers = new Map();
        
        this.setupMessageHandlers();
        this.wss.on('connection', this.handleConnection.bind(this));
    }
    
    verifyClient(info) {
        const token = new URL(info.req.url, 'http://localhost').searchParams.get('token');
        
        try {
            const user = jwt.verify(token, process.env.JWT_SECRET);
            info.req.user = user;
            return true;
        } catch (error) {
            return false;
        }
    }
    
    handleConnection(ws, req) {
        const connectionId = generateId();
        const user = req.user;
        
        const connection = {
            id: connectionId,
            ws,
            user,
            rooms: new Set(),
            lastPing: Date.now()
        };
        
        this.connections.set(connectionId, connection);
        
        ws.on('message', (data) => {
            this.handleMessage(connectionId, data);
        });
        
        ws.on('close', () => {
            this.handleDisconnect(connectionId);
        });
        
        ws.on('pong', () => {
            connection.lastPing = Date.now();
        });
        
        // Send welcome message
        this.sendToConnection(connectionId, {
            type: 'welcome',
            connectionId,
            user: user
        });
    }
    
    setupMessageHandlers() {
        this.messageHandlers.set('join_room', this.handleJoinRoom.bind(this));
        this.messageHandlers.set('leave_room', this.handleLeaveRoom.bind(this));
        this.messageHandlers.set('send_message', this.handleSendMessage.bind(this));
        this.messageHandlers.set('typing_start', this.handleTypingStart.bind(this));
        this.messageHandlers.set('typing_stop', this.handleTypingStop.bind(this));
    }
    
    handleMessage(connectionId, data) {
        try {
            const message = JSON.parse(data);
            const handler = this.messageHandlers.get(message.type);
            
            if (handler) {
                handler(connectionId, message);
            } else {
                this.sendError(connectionId, 'Unknown message type');
            }
        } catch (error) {
            this.sendError(connectionId, 'Invalid message format');
        }
    }
    
    handleJoinRoom(connectionId, message) {
        const connection = this.connections.get(connectionId);
        const { room } = message;
        
        if (!room) {
            return this.sendError(connectionId, 'Room name required');
        }
        
        // Add connection to room
        if (!this.rooms.has(room)) {
            this.rooms.set(room, new Set());
        }
        
        this.rooms.get(room).add(connectionId);
        connection.rooms.add(room);
        
        // Notify room members
        this.broadcastToRoom(room, {
            type: 'user_joined',
            user: connection.user,
            room
        }, connectionId);
        
        // Send confirmation
        this.sendToConnection(connectionId, {
            type: 'room_joined',
            room,
            members: this.getRoomMembers(room)
        });
    }
    
    handleSendMessage(connectionId, message) {
        const connection = this.connections.get(connectionId);
        const { room, content } = message;
        
        if (!room || !content) {
            return this.sendError(connectionId, 'Room and content required');
        }
        
        if (!connection.rooms.has(room)) {
            return this.sendError(connectionId, 'Not in room');
        }
        
        const chatMessage = {
            type: 'new_message',
            id: generateId(),
            user: connection.user,
            room,
            content,
            timestamp: new Date().toISOString()
        };
        
        this.broadcastToRoom(room, chatMessage);
    }
    
    sendToConnection(connectionId, data) {
        const connection = this.connections.get(connectionId);
        if (connection && connection.ws.readyState === WebSocket.OPEN) {
            connection.ws.send(JSON.stringify(data));
        }
    }
    
    broadcastToRoom(room, data, excludeConnectionId = null) {
        const roomConnections = this.rooms.get(room);
        if (!roomConnections) return;
        
        roomConnections.forEach(connectionId => {
            if (connectionId !== excludeConnectionId) {
                this.sendToConnection(connectionId, data);
            }
        });
    }
    
    broadcast(data, excludeConnectionId = null) {
        this.connections.forEach((connection, connectionId) => {
            if (connectionId !== excludeConnectionId) {
                this.sendToConnection(connectionId, data);
            }
        });
    }
    
    getRoomMembers(room) {
        const roomConnections = this.rooms.get(room);
        if (!roomConnections) return [];
        
        return Array.from(roomConnections).map(connectionId => {
            const connection = this.connections.get(connectionId);
            return connection ? connection.user : null;
        }).filter(Boolean);
    }
    
    startHeartbeat() {
        setInterval(() => {
            const now = Date.now();
            
            this.connections.forEach((connection, connectionId) => {
                if (now - connection.lastPing > 60000) {
                    connection.ws.terminate();
                    this.handleDisconnect(connectionId);
                } else if (connection.ws.readyState === WebSocket.OPEN) {
                    connection.ws.ping();
                }
            });
        }, 30000);
    }
}
```

---

## 🛡️ Advanced Security

### OAuth 2.0 Implementation

```javascript
// OAuth 2.0 Server
class OAuthServer {
    constructor() {
        this.clients = new Map();
        this.authorizationCodes = new Map();
        this.accessTokens = new Map();
        this.refreshTokens = new Map();
    }
    
    registerClient(clientId, clientSecret, redirectUris, scopes) {
        this.clients.set(clientId, {
            clientSecret,
            redirectUris,
            scopes: scopes || ['read']
        });
    }
    
    // Authorization Code Flow
    authorize(req, res) {
        const { 
            client_id, 
            redirect_uri, 
            response_type, 
            scope, 
            state 
        } = req.query;
        
        // Validate client
        const client = this.clients.get(client_id);
        if (!client) {
            return res.status(400).json({ error: 'invalid_client' });
        }
        
        if (!client.redirectUris.includes(redirect_uri)) {
            return res.status(400).json({ error: 'invalid_redirect_uri' });
        }
        
        if (response_type !== 'code') {
            return res.status(400).json({ error: 'unsupported_response_type' });
        }
        
        // Generate authorization code
        const code = generateRandomString(32);
        this.authorizationCodes.set(code, {
            clientId: client_id,
            redirectUri: redirect_uri,
            scope: scope || 'read',
            userId: req.user.id,
            expiresAt: Date.now() + 600000 // 10 minutes
        });
        
        // Redirect with code
        const redirectUrl = new URL(redirect_uri);
        redirectUrl.searchParams.set('code', code);
        if (state) redirectUrl.searchParams.set('state', state);
        
        res.redirect(redirectUrl.toString());
    }
    
    // Token Exchange
    async token(req, res) {
        const { 
            grant_type, 
            code, 
            redirect_uri, 
            client_id, 
            client_secret,
            refresh_token
        } = req.body;
        
        // Validate client
        const client = this.clients.get(client_id);
        if (!client || client.clientSecret !== client_secret) {
            return res.status(401).json({ error: 'invalid_client' });
        }
        
        if (grant_type === 'authorization_code') {
            return this.handleAuthorizationCodeGrant(req, res, code, redirect_uri, client_id);
        } else if (grant_type === 'refresh_token') {
            return this.handleRefreshTokenGrant(req, res, refresh_token, client_id);
        } else {
            return res.status(400).json({ error: 'unsupported_grant_type' });
        }
    }
    
    handleAuthorizationCodeGrant(req, res, code, redirectUri, clientId) {
        const authCode = this.authorizationCodes.get(code);
        
        if (!authCode || authCode.expiresAt < Date.now()) {
            return res.status(400).json({ error: 'invalid_grant' });
        }
        
        if (authCode.clientId !== clientId || authCode.redirectUri !== redirectUri) {
            return res.status(400).json({ error: 'invalid_grant' });
        }
        
        // Generate tokens
        const accessToken = jwt.sign(
            { 
                userId: authCode.userId, 
                clientId, 
                scope: authCode.scope 
            },
            process.env.JWT_SECRET,
            { expiresIn: '1h' }
        );
        
        const refreshToken = generateRandomString(64);
        
        // Store tokens
        this.accessTokens.set(accessToken, {
            userId: authCode.userId,
            clientId,
            scope: authCode.scope,
            expiresAt: Date.now() + 3600000 // 1 hour
        });
        
        this.refreshTokens.set(refreshToken, {
            userId: authCode.userId,
            clientId,
            scope: authCode.scope
        });
        
        // Clean up authorization code
        this.authorizationCodes.delete(code);
        
        res.json({
            access_token: accessToken,
            token_type: 'Bearer',
            expires_in: 3600,
            refresh_token: refreshToken,
            scope: authCode.scope
        });
    }
    
    // API Key Authentication
    validateApiKey(req, res, next) {
        const apiKey = req.headers['x-api-key'];
        
        if (!apiKey) {
            return res.status(401).json({ error: 'API key required' });
        }
        
        // Validate API key (implement your own logic)
        const keyData = this.validateKey(apiKey);
        if (!keyData) {
            return res.status(401).json({ error: 'Invalid API key' });
        }
        
        // Check rate limits for this API key
        if (!this.checkRateLimit(apiKey)) {
            return res.status(429).json({ error: 'Rate limit exceeded' });
        }
        
        req.apiKey = keyData;
        next();
    }
}
```

---

## 📊 API Monitoring and Analytics

### Advanced Monitoring System

```javascript
class APIMonitoring {
    constructor() {
        this.metrics = new Map();
        this.alerts = [];
        this.dashboardData = {};
    }
    
    middleware() {
        return (req, res, next) => {
            const startTime = Date.now();
            const originalSend = res.send;
            
            res.send = function(data) {
                const endTime = Date.now();
                const responseTime = endTime - startTime;
                
                // Collect metrics
                this.collectMetrics(req, res, responseTime);
                
                // Call original send
                originalSend.call(this, data);
            }.bind(this);
            
            next();
        };
    }
    
    collectMetrics(req, res, responseTime) {
        const key = `${req.method}:${req.route?.path || req.path}`;
        
        if (!this.metrics.has(key)) {
            this.metrics.set(key, {
                count: 0,
                totalTime: 0,
                errors: 0,
                statusCodes: {},
                responseTimes: []
            });
        }
        
        const metric = this.metrics.get(key);
        metric.count++;
        metric.totalTime += responseTime;
        metric.responseTimes.push(responseTime);
        
        // Keep only last 1000 response times for percentile calculation
        if (metric.responseTimes.length > 1000) {
            metric.responseTimes.shift();
        }
        
        // Track status codes
        const statusCode = res.statusCode;
        metric.statusCodes[statusCode] = (metric.statusCodes[statusCode] || 0) + 1;
        
        if (statusCode >= 400) {
            metric.errors++;
        }
        
        // Check for alerts
        this.checkAlerts(key, metric, responseTime);
    }
    
    checkAlerts(endpoint, metric, responseTime) {
        // High response time alert
        if (responseTime > 5000) {
            this.createAlert('HIGH_RESPONSE_TIME', {
                endpoint,
                responseTime,
                threshold: 5000
            });
        }
        
        // High error rate alert
        const errorRate = metric.errors / metric.count;
        if (metric.count > 10 && errorRate > 0.1) {
            this.createAlert('HIGH_ERROR_RATE', {
                endpoint,
                errorRate: errorRate * 100,
                threshold: 10
            });
        }
    }
    
    createAlert(type, data) {
        const alert = {
            id: generateId(),
            type,
            data,
            timestamp: new Date().toISOString(),
            severity: this.getAlertSeverity(type)
        };
        
        this.alerts.push(alert);
        
        // Send notification (email, Slack, etc.)
        this.sendNotification(alert);
    }
    
    getMetrics() {
        const summary = {};
        
        this.metrics.forEach((metric, endpoint) => {
            const responseTimes = metric.responseTimes.sort((a, b) => a - b);
            
            summary[endpoint] = {
                requestCount: metric.count,
                averageResponseTime: metric.totalTime / metric.count,
                p50: this.getPercentile(responseTimes, 50),
                p95: this.getPercentile(responseTimes, 95),
                p99: this.getPercentile(responseTimes, 99),
                errorRate: (metric.errors / metric.count) * 100,
                statusCodes: metric.statusCodes
            };
        });
        
        return summary;
    }
    
    getPercentile(sortedArray, percentile) {
        const index = Math.ceil((percentile / 100) * sortedArray.length) - 1;
        return sortedArray[index] || 0;
    }
    
    // Health check endpoint
    healthCheck() {
        return (req, res) => {
            const health = {
                status: 'healthy',
                timestamp: new Date().toISOString(),
                uptime: process.uptime(),
                memory: process.memoryUsage(),
                cpu: process.cpuUsage(),
                version: process.env.npm_package_version
            };
            
            // Check database connectivity
            this.checkDatabaseHealth().then(dbStatus => {
                health.database = dbStatus;
                res.json(health);
            }).catch(error => {
                health.status = 'unhealthy';
                health.database = { status: 'error', error: error.message };
                res.status(503).json(health);
            });
        };
    }
}

// Usage
const monitoring = new APIMonitoring();
app.use(monitoring.middleware());
app.get('/health', monitoring.healthCheck());
app.get('/metrics', (req, res) => {
    res.json(monitoring.getMetrics());
});
```

This advanced API development guide covers GraphQL, microservices patterns, real-time APIs, advanced security, and comprehensive monitoring - essential for building enterprise-grade API systems.
