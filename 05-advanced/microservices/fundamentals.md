
# 🏗️ Microservices Architecture Basics

## Table of Contents
- [Introduction to Microservices](#introduction-to-microservices)
- [Microservices vs Monolithic Architecture](#microservices-vs-monolithic-architecture)
- [Core Principles](#core-principles)
- [Service Communication](#service-communication)
- [API Gateway Pattern](#api-gateway-pattern)
- [Service Discovery](#service-discovery)
- [Data Management](#data-management)
- [Deployment Strategies](#deployment-strategies)

---

## 🌟 Introduction to Microservices

Microservices architecture is a design approach where applications are built as a collection of small, independently deployable services that communicate over well-defined APIs.

### Key Characteristics:
- **Loosely Coupled**: Services are independent
- **Highly Cohesive**: Each service has a single responsibility
- **Autonomous**: Teams can develop and deploy independently
- **Decentralized**: No central orchestrator
- **Fault Tolerant**: Failure in one service doesn't bring down the system

### Benefits:
- Scalability
- Technology diversity
- Faster development
- Better fault isolation
- Easier maintenance

### Challenges:
- Distributed system complexity
- Network latency
- Data consistency
- Testing complexity
- Operational overhead

---

## ⚖️ Microservices vs Monolithic Architecture

| Aspect | Monolithic | Microservices |
|--------|------------|---------------|
| **Deployment** | Single unit | Independent services |
| **Scaling** | Scale entire app | Scale individual services |
| **Technology** | Single stack | Technology diversity |
| **Team Structure** | Large teams | Small, focused teams |
| **Communication** | In-process calls | Network calls |
| **Data Storage** | Shared database | Service-specific databases |
| **Testing** | Simpler integration | Complex distributed testing |
| **Debugging** | Easier to trace | Distributed tracing needed |

### Evolution Path:
```
Monolithic Application
        ↓
Modular Monolith
        ↓
Service-Oriented Architecture (SOA)
        ↓
Microservices Architecture
```

---

## 🎯 Core Principles

### 1. Single Responsibility Principle
```javascript
// User Service - Handles only user-related operations
class UserService {
    async createUser(userData) {
        // Validate user data
        const validatedData = this.validateUserData(userData);
        
        // Create user in database
        const user = await this.userRepository.create(validatedData);
        
        // Publish user created event
        await this.eventBus.publish('user.created', {
            userId: user.id,
            email: user.email,
            timestamp: new Date()
        });
        
        return user;
    }
    
    async getUserById(userId) {
        return await this.userRepository.findById(userId);
    }
    
    async updateUser(userId, updateData) {
        const user = await this.userRepository.update(userId, updateData);
        
        await this.eventBus.publish('user.updated', {
            userId: user.id,
            changes: updateData,
            timestamp: new Date()
        });
        
        return user;
    }
    
    async deleteUser(userId) {
        await this.userRepository.delete(userId);
        
        await this.eventBus.publish('user.deleted', {
            userId,
            timestamp: new Date()
        });
    }
}

// Order Service - Handles only order-related operations
class OrderService {
    async createOrder(orderData) {
        // Validate order data
        const validatedData = this.validateOrderData(orderData);
        
        // Check user exists (call User Service)
        const user = await this.userServiceClient.getUserById(validatedData.userId);
        if (!user) {
            throw new Error('User not found');
        }
        
        // Create order
        const order = await this.orderRepository.create(validatedData);
        
        // Publish order created event
        await this.eventBus.publish('order.created', {
            orderId: order.id,
            userId: order.userId,
            total: order.total,
            timestamp: new Date()
        });
        
        return order;
    }
}
```

### 2. Database per Service
```javascript
// User Service Database Configuration
class UserDatabase {
    constructor() {
        this.connection = new PostgreSQLConnection({
            database: 'user_service_db',
            host: process.env.USER_DB_HOST,
            port: process.env.USER_DB_PORT,
            username: process.env.USER_DB_USERNAME,
            password: process.env.USER_DB_PASSWORD
        });
    }
}

// Order Service Database Configuration
class OrderDatabase {
    constructor() {
        this.connection = new MongoConnection({
            database: 'order_service_db',
            url: process.env.ORDER_DB_URL
        });
    }
}

// Product Service Database Configuration
class ProductDatabase {
    constructor() {
        this.connection = new RedisConnection({
            host: process.env.PRODUCT_CACHE_HOST,
            port: process.env.PRODUCT_CACHE_PORT
        });
        
        this.searchEngine = new ElasticsearchConnection({
            node: process.env.ELASTICSEARCH_URL
        });
    }
}
```

---

## 🔄 Service Communication

### 1. Synchronous Communication (HTTP/REST)
```javascript
// HTTP Client for Service Communication
class ServiceClient {
    constructor(baseURL, options = {}) {
        this.baseURL = baseURL;
        this.timeout = options.timeout || 5000;
        this.retries = options.retries || 3;
        this.circuitBreaker = new CircuitBreaker();
    }
    
    async get(endpoint, options = {}) {
        return this.makeRequest('GET', endpoint, null, options);
    }
    
    async post(endpoint, data, options = {}) {
        return this.makeRequest('POST', endpoint, data, options);
    }
    
    async makeRequest(method, endpoint, data, options) {
        const url = `${this.baseURL}${endpoint}`;
        
        // Circuit breaker check
        if (this.circuitBreaker.isOpen()) {
            throw new Error('Service temporarily unavailable');
        }
        
        let attempt = 0;
        while (attempt < this.retries) {
            try {
                const response = await fetch(url, {
                    method,
                    headers: {
                        'Content-Type': 'application/json',
                        'X-Request-ID': generateRequestId(),
                        'X-Service-Name': process.env.SERVICE_NAME,
                        ...options.headers
                    },
                    body: data ? JSON.stringify(data) : undefined,
                    timeout: this.timeout
                });
                
                if (!response.ok) {
                    throw new Error(`HTTP ${response.status}: ${response.statusText}`);
                }
                
                this.circuitBreaker.recordSuccess();
                return await response.json();
                
            } catch (error) {
                attempt++;
                this.circuitBreaker.recordFailure();
                
                if (attempt >= this.retries) {
                    throw error;
                }
                
                // Exponential backoff
                await this.delay(Math.pow(2, attempt) * 1000);
            }
        }
    }
    
    delay(ms) {
        return new Promise(resolve => setTimeout(resolve, ms));
    }
}

// Service Clients
class UserServiceClient extends ServiceClient {
    constructor() {
        super(process.env.USER_SERVICE_URL);
    }
    
    async getUserById(userId) {
        return this.get(`/users/${userId}`);
    }
    
    async createUser(userData) {
        return this.post('/users', userData);
    }
}

class OrderServiceClient extends ServiceClient {
    constructor() {
        super(process.env.ORDER_SERVICE_URL);
    }
    
    async getOrdersByUserId(userId) {
        return this.get(`/orders?userId=${userId}`);
    }
    
    async createOrder(orderData) {
        return this.post('/orders', orderData);
    }
}
```

### 2. Asynchronous Communication (Message Queues)
```javascript
const amqp = require('amqplib');

// Event Bus for Asynchronous Communication
class EventBus {
    constructor() {
        this.connection = null;
        this.channel = null;
        this.subscribers = new Map();
    }
    
    async connect() {
        this.connection = await amqp.connect(process.env.RABBITMQ_URL);
        this.channel = await this.connection.createChannel();
        
        // Declare exchanges
        await this.channel.assertExchange('events', 'topic', { durable: true });
        await this.channel.assertExchange('commands', 'direct', { durable: true });
    }
    
    async publish(eventType, data) {
        const message = {
            eventType,
            data,
            timestamp: new Date().toISOString(),
            serviceId: process.env.SERVICE_NAME,
            correlationId: generateCorrelationId()
        };
        
        await this.channel.publish(
            'events',
            eventType,
            Buffer.from(JSON.stringify(message)),
            { persistent: true }
        );
        
        console.log(`Published event: ${eventType}`);
    }
    
    async subscribe(pattern, handler) {
        const queue = await this.channel.assertQueue('', { exclusive: true });
        await this.channel.bindQueue(queue.queue, 'events', pattern);
        
        this.channel.consume(queue.queue, async (message) => {
            try {
                const content = JSON.parse(message.content.toString());
                await handler(content);
                this.channel.ack(message);
            } catch (error) {
                console.error('Error processing message:', error);
                this.channel.nack(message, false, false); // Dead letter
            }
        });
        
        this.subscribers.set(pattern, handler);
        console.log(`Subscribed to pattern: ${pattern}`);
    }
    
    async sendCommand(service, command, data) {
        const message = {
            command,
            data,
            timestamp: new Date().toISOString(),
            serviceId: process.env.SERVICE_NAME,
            correlationId: generateCorrelationId()
        };
        
        await this.channel.publish(
            'commands',
            service,
            Buffer.from(JSON.stringify(message)),
            { persistent: true }
        );
    }
}

// Usage in services
class OrderService {
    constructor() {
        this.eventBus = new EventBus();
        this.setupEventHandlers();
    }
    
    async setupEventHandlers() {
        await this.eventBus.connect();
        
        // Listen for user events
        await this.eventBus.subscribe('user.*', this.handleUserEvent.bind(this));
        
        // Listen for payment events
        await this.eventBus.subscribe('payment.*', this.handlePaymentEvent.bind(this));
    }
    
    async handleUserEvent(event) {
        switch (event.eventType) {
            case 'user.deleted':
                await this.cancelUserOrders(event.data.userId);
                break;
            case 'user.updated':
                await this.updateOrderUserInfo(event.data.userId, event.data.changes);
                break;
        }
    }
    
    async handlePaymentEvent(event) {
        switch (event.eventType) {
            case 'payment.completed':
                await this.confirmOrder(event.data.orderId);
                break;
            case 'payment.failed':
                await this.cancelOrder(event.data.orderId);
                break;
        }
    }
    
    async createOrder(orderData) {
        // Create order logic
        const order = await this.orderRepository.create(orderData);
        
        // Publish order created event
        await this.eventBus.publish('order.created', {
            orderId: order.id,
            userId: order.userId,
            items: order.items,
            total: order.total
        });
        
        // Send payment command
        await this.eventBus.sendCommand('payment-service', 'process.payment', {
            orderId: order.id,
            amount: order.total,
            userId: order.userId
        });
        
        return order;
    }
}
```

---

## 🚪 API Gateway Pattern

```javascript
const express = require('express');
const httpProxy = require('http-proxy-middleware');
const rateLimit = require('express-rate-limit');
const jwt = require('jsonwebtoken');

class APIGateway {
    constructor() {
        this.app = express();
        this.setupMiddleware();
        this.setupRoutes();
    }
    
    setupMiddleware() {
        // CORS
        this.app.use((req, res, next) => {
            res.header('Access-Control-Allow-Origin', '*');
            res.header('Access-Control-Allow-Methods', 'GET, POST, PUT, DELETE, OPTIONS');
            res.header('Access-Control-Allow-Headers', 'Origin, X-Requested-With, Content-Type, Accept, Authorization');
            next();
        });
        
        // Request logging
        this.app.use((req, res, next) => {
            console.log(`${new Date().toISOString()} - ${req.method} ${req.path}`);
            next();
        });
        
        // Rate limiting
        const limiter = rateLimit({
            windowMs: 15 * 60 * 1000, // 15 minutes
            max: 100, // Limit each IP to 100 requests per windowMs
            message: 'Too many requests from this IP'
        });
        this.app.use(limiter);
        
        // Body parsing
        this.app.use(express.json());
    }
    
    setupRoutes() {
        // Health check
        this.app.get('/health', (req, res) => {
            res.json({ status: 'healthy', timestamp: new Date().toISOString() });
        });
        
        // Authentication routes (no auth required)
        this.app.use('/auth', this.createProxy({
            target: process.env.AUTH_SERVICE_URL,
            pathRewrite: { '^/auth': '' }
        }));
        
        // Protected routes
        this.app.use('/api/users', 
            this.authenticateToken,
            this.createProxy({
                target: process.env.USER_SERVICE_URL,
                pathRewrite: { '^/api/users': '' }
            })
        );
        
        this.app.use('/api/orders',
            this.authenticateToken,
            this.createProxy({
                target: process.env.ORDER_SERVICE_URL,
                pathRewrite: { '^/api/orders': '' }
            })
        );
        
        this.app.use('/api/products',
            this.createProxy({
                target: process.env.PRODUCT_SERVICE_URL,
                pathRewrite: { '^/api/products': '' }
            })
        );
        
        // Aggregation endpoint
        this.app.get('/api/user/:userId/dashboard', 
            this.authenticateToken,
            this.getUserDashboard.bind(this)
        );
    }
    
    createProxy(options) {
        return httpProxy({
            ...options,
            changeOrigin: true,
            timeout: 30000,
            onError: (err, req, res) => {
                console.error('Proxy error:', err);
                res.status(503).json({
                    error: 'Service temporarily unavailable',
                    message: 'The requested service is currently unavailable'
                });
            },
            onProxyReq: (proxyReq, req, res) => {
                // Add correlation ID
                proxyReq.setHeader('X-Correlation-ID', req.correlationId || generateCorrelationId());
                
                // Add user context
                if (req.user) {
                    proxyReq.setHeader('X-User-ID', req.user.id);
                    proxyReq.setHeader('X-User-Roles', JSON.stringify(req.user.roles));
                }
            },
            onProxyRes: (proxyRes, req, res) => {
                // Add response headers
                res.setHeader('X-Response-Time', Date.now() - req.startTime);
            }
        });
    }
    
    authenticateToken(req, res, next) {
        const authHeader = req.headers['authorization'];
        const token = authHeader && authHeader.split(' ')[1];
        
        if (!token) {
            return res.status(401).json({ error: 'Access token required' });
        }
        
        jwt.verify(token, process.env.JWT_SECRET, (err, user) => {
            if (err) {
                return res.status(403).json({ error: 'Invalid or expired token' });
            }
            
            req.user = user;
            next();
        });
    }
    
    async getUserDashboard(req, res) {
        const userId = req.params.userId;
        const userServiceClient = new UserServiceClient();
        const orderServiceClient = new OrderServiceClient();
        
        try {
            // Parallel requests to multiple services
            const [user, orders] = await Promise.all([
                userServiceClient.getUserById(userId),
                orderServiceClient.getOrdersByUserId(userId)
            ]);
            
            // Aggregate response
            const dashboard = {
                user: {
                    id: user.id,
                    name: user.name,
                    email: user.email
                },
                orders: {
                    total: orders.length,
                    recent: orders.slice(0, 5),
                    totalValue: orders.reduce((sum, order) => sum + order.total, 0)
                },
                timestamp: new Date().toISOString()
            };
            
            res.json(dashboard);
            
        } catch (error) {
            console.error('Dashboard aggregation error:', error);
            res.status(500).json({
                error: 'Failed to load dashboard data',
                message: error.message
            });
        }
    }
    
    start(port = 3000) {
        this.app.listen(port, () => {
            console.log(`API Gateway running on port ${port}`);
        });
    }
}

// Circuit Breaker Implementation
class CircuitBreaker {
    constructor(options = {}) {
        this.failureThreshold = options.failureThreshold || 5;
        this.resetTimeout = options.resetTimeout || 60000;
        this.state = 'CLOSED'; // CLOSED, OPEN, HALF_OPEN
        this.failureCount = 0;
        this.lastFailureTime = null;
    }
    
    isOpen() {
        if (this.state === 'OPEN') {
            if (Date.now() - this.lastFailureTime >= this.resetTimeout) {
                this.state = 'HALF_OPEN';
                this.failureCount = 0;
                return false;
            }
            return true;
        }
        return false;
    }
    
    recordSuccess() {
        this.failureCount = 0;
        this.state = 'CLOSED';
    }
    
    recordFailure() {
        this.failureCount++;
        this.lastFailureTime = Date.now();
        
        if (this.failureCount >= this.failureThreshold) {
            this.state = 'OPEN';
        }
    }
}

// Start the API Gateway
const gateway = new APIGateway();
gateway.start(process.env.PORT || 3000);
```

---

## 🔍 Service Discovery

```javascript
// Service Registry
class ServiceRegistry {
    constructor() {
        this.services = new Map();
        this.healthCheckInterval = 30000; // 30 seconds
        this.startHealthChecks();
    }
    
    register(serviceName, instance) {
        if (!this.services.has(serviceName)) {
            this.services.set(serviceName, []);
        }
        
        const serviceInstances = this.services.get(serviceName);
        const existingInstance = serviceInstances.find(inst => 
            inst.id === instance.id
        );
        
        if (existingInstance) {
            Object.assign(existingInstance, instance);
        } else {
            serviceInstances.push({
                ...instance,
                registeredAt: new Date(),
                lastHeartbeat: new Date(),
                healthy: true
            });
        }
        
        console.log(`Service registered: ${serviceName}@${instance.host}:${instance.port}`);
    }
    
    deregister(serviceName, instanceId) {
        if (this.services.has(serviceName)) {
            const serviceInstances = this.services.get(serviceName);
            const index = serviceInstances.findIndex(inst => inst.id === instanceId);
            
            if (index !== -1) {
                serviceInstances.splice(index, 1);
                console.log(`Service deregistered: ${serviceName}@${instanceId}`);
            }
        }
    }
    
    discover(serviceName) {
        if (!this.services.has(serviceName)) {
            return [];
        }
        
        return this.services.get(serviceName)
            .filter(instance => instance.healthy)
            .map(instance => ({
                id: instance.id,
                host: instance.host,
                port: instance.port,
                metadata: instance.metadata
            }));
    }
    
    heartbeat(serviceName, instanceId) {
        if (this.services.has(serviceName)) {
            const serviceInstances = this.services.get(serviceName);
            const instance = serviceInstances.find(inst => inst.id === instanceId);
            
            if (instance) {
                instance.lastHeartbeat = new Date();
                instance.healthy = true;
            }
        }
    }
    
    async startHealthChecks() {
        setInterval(async () => {
            const now = new Date();
            const threshold = new Date(now.getTime() - this.healthCheckInterval * 2);
            
            for (const [serviceName, instances] of this.services) {
                for (const instance of instances) {
                    if (instance.lastHeartbeat < threshold) {
                        instance.healthy = false;
                        console.log(`Service unhealthy: ${serviceName}@${instance.id}`);
                    }
                }
            }
        }, this.healthCheckInterval);
    }
    
    getAllServices() {
        const result = {};
        for (const [serviceName, instances] of this.services) {
            result[serviceName] = instances.map(instance => ({
                id: instance.id,
                host: instance.host,
                port: instance.port,
                healthy: instance.healthy,
                lastHeartbeat: instance.lastHeartbeat,
                metadata: instance.metadata
            }));
        }
        return result;
    }
}

// Service Discovery Client
class ServiceDiscoveryClient {
    constructor(registryUrl) {
        this.registryUrl = registryUrl;
        this.serviceCache = new Map();
        this.cacheTimeout = 30000; // 30 seconds
    }
    
    async register(serviceName, instance) {
        const response = await fetch(`${this.registryUrl}/register`, {
            method: 'POST',
            headers: { 'Content-Type': 'application/json' },
            body: JSON.stringify({ serviceName, instance })
        });
        
        if (!response.ok) {
            throw new Error('Service registration failed');
        }
        
        // Start heartbeat
        this.startHeartbeat(serviceName, instance.id);
    }
    
    async discover(serviceName) {
        // Check cache first
        const cached = this.serviceCache.get(serviceName);
        if (cached && Date.now() - cached.timestamp < this.cacheTimeout) {
            return cached.instances;
        }
        
        // Fetch from registry
        const response = await fetch(`${this.registryUrl}/discover/${serviceName}`);
        if (!response.ok) {
            throw new Error(`Service discovery failed for ${serviceName}`);
        }
        
        const instances = await response.json();
        
        // Update cache
        this.serviceCache.set(serviceName, {
            instances,
            timestamp: Date.now()
        });
        
        return instances;
    }
    
    async startHeartbeat(serviceName, instanceId) {
        setInterval(async () => {
            try {
                await fetch(`${this.registryUrl}/heartbeat`, {
                    method: 'POST',
                    headers: { 'Content-Type': 'application/json' },
                    body: JSON.stringify({ serviceName, instanceId })
                });
            } catch (error) {
                console.error('Heartbeat failed:', error);
            }
        }, 15000); // 15 seconds
    }
    
    // Load balancing
    selectInstance(instances, strategy = 'round-robin') {
        if (instances.length === 0) {
            throw new Error('No healthy instances available');
        }
        
        switch (strategy) {
            case 'round-robin':
                return this.roundRobinSelect(instances);
            case 'random':
                return instances[Math.floor(Math.random() * instances.length)];
            case 'least-connections':
                return this.leastConnectionsSelect(instances);
            default:
                return instances[0];
        }
    }
    
    roundRobinSelect(instances) {
        const key = `rr_${instances.map(i => i.id).join('_')}`;
        if (!this.roundRobinCounters) this.roundRobinCounters = {};
        if (!this.roundRobinCounters[key]) this.roundRobinCounters[key] = 0;
        
        const index = this.roundRobinCounters[key] % instances.length;
        this.roundRobinCounters[key]++;
        
        return instances[index];
    }
}

// Service registration in microservice
class MicroserviceBootstrap {
    constructor(serviceName, port) {
        this.serviceName = serviceName;
        this.port = port;
        this.instanceId = `${serviceName}_${Date.now()}_${Math.random().toString(36).substr(2, 9)}`;
        this.discoveryClient = new ServiceDiscoveryClient(process.env.SERVICE_REGISTRY_URL);
    }
    
    async start() {
        // Start the service
        this.app = express();
        this.setupRoutes();
        
        this.server = this.app.listen(this.port, async () => {
            console.log(`${this.serviceName} started on port ${this.port}`);
            
            // Register with service registry
            await this.registerService();
        });
        
        // Graceful shutdown
        process.on('SIGTERM', () => this.shutdown());
        process.on('SIGINT', () => this.shutdown());
    }
    
    async registerService() {
        const instance = {
            id: this.instanceId,
            host: process.env.SERVICE_HOST || 'localhost',
            port: this.port,
            metadata: {
                version: process.env.SERVICE_VERSION || '1.0.0',
                environment: process.env.NODE_ENV || 'development'
            }
        };
        
        await this.discoveryClient.register(this.serviceName, instance);
        console.log(`Service registered: ${this.serviceName}`);
    }
    
    setupRoutes() {
        // Health check endpoint
        this.app.get('/health', (req, res) => {
            res.json({
                status: 'healthy',
                service: this.serviceName,
                instance: this.instanceId,
                timestamp: new Date().toISOString()
            });
        });
        
        // Add service-specific routes here
    }
    
    async shutdown() {
        console.log(`Shutting down ${this.serviceName}...`);
        
        // Deregister from service registry
        await this.discoveryClient.deregister(this.serviceName, this.instanceId);
        
        // Close server
        this.server.close(() => {
            console.log(`${this.serviceName} shut down gracefully`);
            process.exit(0);
        });
    }
}

// Usage
const service = new MicroserviceBootstrap('user-service', 3001);
service.start();
```

This comprehensive guide covers the fundamentals of microservices architecture, including service communication patterns, API gateway implementation, and service discovery mechanisms. The examples demonstrate practical implementation patterns for building scalable microservices systems.
