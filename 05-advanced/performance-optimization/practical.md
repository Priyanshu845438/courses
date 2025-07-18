
# ⚡ Intermediate Performance Optimization

## Table of Contents
- [Advanced Caching Strategies](#advanced-caching-strategies)
- [Database Performance](#database-performance)
- [Frontend Performance](#frontend-performance)
- [Backend Optimization](#backend-optimization)
- [Network Performance](#network-performance)
- [Monitoring and Profiling](#monitoring-and-profiling)
- [Load Testing](#load-testing)
- [Memory Management](#memory-management)
- [API Optimization](#api-optimization)

---

## 🗄️ Advanced Caching Strategies

### Multi-Level Caching Architecture

```javascript
// cache-manager.js
const Redis = require('redis');
const NodeCache = require('node-cache');

class CacheManager {
    constructor() {
        // L1 Cache - In-memory (fastest)
        this.l1Cache = new NodeCache({ 
            stdTTL: 300, // 5 minutes
            maxKeys: 1000 
        });
        
        // L2 Cache - Redis (shared across instances)
        this.l2Cache = Redis.createClient({
            host: process.env.REDIS_HOST,
            port: process.env.REDIS_PORT,
            maxRetriesPerRequest: 3,
            retryDelayOnFailover: 100
        });
        
        // L3 Cache - Database query result cache
        this.queryCache = new Map();
    }
    
    async get(key, fetchFunction, options = {}) {
        const { ttl = 300, useL1 = true, useL2 = true } = options;
        
        // Try L1 cache first
        if (useL1) {
            const l1Result = this.l1Cache.get(key);
            if (l1Result !== undefined) {
                return l1Result;
            }
        }
        
        // Try L2 cache (Redis)
        if (useL2) {
            try {
                const l2Result = await this.l2Cache.get(key);
                if (l2Result) {
                    const parsed = JSON.parse(l2Result);
                    // Populate L1 cache
                    if (useL1) {
                        this.l1Cache.set(key, parsed, ttl);
                    }
                    return parsed;
                }
            } catch (error) {
                console.error('Redis cache error:', error);
            }
        }
        
        // Fetch from source
        const result = await fetchFunction();
        
        // Store in caches
        if (useL1) {
            this.l1Cache.set(key, result, ttl);
        }
        
        if (useL2) {
            try {
                await this.l2Cache.setex(key, ttl, JSON.stringify(result));
            } catch (error) {
                console.error('Redis set error:', error);
            }
        }
        
        return result;
    }
    
    async invalidate(pattern) {
        // Invalidate L1 cache
        this.l1Cache.flushAll();
        
        // Invalidate L2 cache by pattern
        try {
            const keys = await this.l2Cache.keys(pattern);
            if (keys.length > 0) {
                await this.l2Cache.del(keys);
            }
        } catch (error) {
            console.error('Cache invalidation error:', error);
        }
    }
}

// Usage example
const cacheManager = new CacheManager();

app.get('/api/users/:id', async (req, res) => {
    const userId = req.params.id;
    const cacheKey = `user:${userId}`;
    
    const user = await cacheManager.get(
        cacheKey,
        () => User.findById(userId),
        { ttl: 600, useL1: true, useL2: true }
    );
    
    res.json(user);
});
```

### Cache-Aside Pattern with Write-Through

```javascript
// cache-patterns.js
class CacheService {
    constructor(cache, database) {
        this.cache = cache;
        this.db = database;
    }
    
    // Cache-Aside (Lazy Loading)
    async getUser(userId) {
        const cacheKey = `user:${userId}`;
        
        // Try cache first
        let user = await this.cache.get(cacheKey);
        if (user) {
            return JSON.parse(user);
        }
        
        // Cache miss - fetch from database
        user = await this.db.users.findById(userId);
        if (user) {
            // Store in cache for future requests
            await this.cache.setex(cacheKey, 3600, JSON.stringify(user));
        }
        
        return user;
    }
    
    // Write-Through Pattern
    async updateUser(userId, userData) {
        const cacheKey = `user:${userId}`;
        
        // Update database first
        const updatedUser = await this.db.users.update(userId, userData);
        
        // Update cache immediately
        await this.cache.setex(cacheKey, 3600, JSON.stringify(updatedUser));
        
        // Invalidate related caches
        await this.invalidateUserRelatedCaches(userId);
        
        return updatedUser;
    }
    
    // Write-Behind (Write-Back) Pattern
    async updateUserAsync(userId, userData) {
        const cacheKey = `user:${userId}`;
        
        // Update cache immediately
        await this.cache.setex(cacheKey, 3600, JSON.stringify(userData));
        
        // Queue database update for later
        await this.queueDatabaseUpdate('users', userId, userData);
        
        return userData;
    }
    
    async invalidateUserRelatedCaches(userId) {
        const patterns = [
            `user:${userId}`,
            `user:${userId}:*`,
            `users:page:*`,
            `users:search:*`
        ];
        
        for (const pattern of patterns) {
            const keys = await this.cache.keys(pattern);
            if (keys.length > 0) {
                await this.cache.del(keys);
            }
        }
    }
}
```

---

## 🗃️ Database Performance

### Query Optimization

```javascript
// database-optimization.js
const mongoose = require('mongoose');

// Efficient query patterns
class UserService {
    // ❌ Inefficient: N+1 problem
    static async getUsersWithPostsBad() {
        const users = await User.find();
        for (const user of users) {
            user.posts = await Post.find({ userId: user._id });
        }
        return users;
    }
    
    // ✅ Efficient: Using aggregation pipeline
    static async getUsersWithPosts() {
        return User.aggregate([
            {
                $lookup: {
                    from: 'posts',
                    localField: '_id',
                    foreignField: 'userId',
                    as: 'posts'
                }
            },
            {
                $addFields: {
                    postCount: { $size: '$posts' }
                }
            },
            {
                $project: {
                    name: 1,
                    email: 1,
                    posts: { $slice: ['$posts', 5] }, // Limit posts
                    postCount: 1
                }
            }
        ]);
    }
    
    // Pagination with cursor-based approach
    static async getUsersPaginated(cursor = null, limit = 20) {
        const query = cursor ? { _id: { $gt: cursor } } : {};
        
        return User.find(query)
            .sort({ _id: 1 })
            .limit(limit + 1) // +1 to check if there's a next page
            .lean(); // Return plain objects, not Mongoose documents
    }
    
    // Efficient search with text indexing
    static async searchUsers(searchTerm, filters = {}) {
        const pipeline = [];
        
        // Text search stage
        if (searchTerm) {
            pipeline.push({
                $match: {
                    $text: { $search: searchTerm }
                }
            });
            
            pipeline.push({
                $addFields: {
                    score: { $meta: 'textScore' }
                }
            });
        }
        
        // Apply filters
        if (Object.keys(filters).length > 0) {
            pipeline.push({ $match: filters });
        }
        
        // Sort by relevance score
        if (searchTerm) {
            pipeline.push({
                $sort: { score: { $meta: 'textScore' } }
            });
        }
        
        return User.aggregate(pipeline);
    }
}

// Index strategies
const userSchema = new mongoose.Schema({
    name: { type: String, required: true },
    email: { type: String, required: true, unique: true },
    age: Number,
    city: String,
    createdAt: { type: Date, default: Date.now },
    lastLoginAt: Date,
    tags: [String]
});

// Compound index for common query patterns
userSchema.index({ city: 1, age: 1 });

// Text index for search
userSchema.index({ name: 'text', email: 'text' });

// TTL index for session data
userSchema.index({ lastLoginAt: 1 }, { expireAfterSeconds: 2592000 }); // 30 days

// Sparse index for optional fields
userSchema.index({ tags: 1 }, { sparse: true });
```

### Connection Pooling and Optimization

```javascript
// database-connection.js
const { Pool } = require('pg');
const mongoose = require('mongoose');

// PostgreSQL connection pool
const pgPool = new Pool({
    host: process.env.DB_HOST,
    port: process.env.DB_PORT,
    database: process.env.DB_NAME,
    user: process.env.DB_USER,
    password: process.env.DB_PASSWORD,
    
    // Connection pool settings
    min: 5,                    // Minimum connections
    max: 20,                   // Maximum connections
    idle: 10000,               // Close idle connections after 10s
    acquire: 60000,            // Max time to get connection
    evict: 1000,               // Check for idle connections every 1s
    
    // Performance settings
    statement_timeout: 30000,   // 30 second query timeout
    idle_in_transaction_session_timeout: 30000,
    
    // SSL settings for production
    ssl: process.env.NODE_ENV === 'production' ? {
        require: true,
        rejectUnauthorized: false
    } : false
});

// MongoDB connection optimization
mongoose.connect(process.env.MONGODB_URI, {
    // Connection pool settings
    maxPoolSize: 20,           // Maximum connections
    minPoolSize: 5,            // Minimum connections
    acquireWaitQueue: 50,      // Max waiting requests
    
    // Timeout settings
    serverSelectionTimeoutMS: 5000,
    socketTimeoutMS: 45000,
    connectTimeoutMS: 10000,
    
    // Buffer settings
    bufferMaxEntries: 0,       // Disable mongoose buffering
    bufferCommands: false,
    
    // Performance settings
    maxIdleTimeMS: 30000,      // Close connections after 30s idle
    heartbeatFrequencyMS: 10000, // Check server every 10s
    
    // Write concern
    writeConcern: {
        w: 'majority',
        j: true,
        wtimeout: 1000
    }
});

// Database query optimization utilities
class QueryOptimizer {
    static async executeWithRetry(queryFn, maxRetries = 3) {
        for (let attempt = 1; attempt <= maxRetries; attempt++) {
            try {
                return await queryFn();
            } catch (error) {
                if (attempt === maxRetries || !this.isRetryableError(error)) {
                    throw error;
                }
                
                const delay = Math.pow(2, attempt) * 1000; // Exponential backoff
                await new Promise(resolve => setTimeout(resolve, delay));
            }
        }
    }
    
    static isRetryableError(error) {
        return error.code === 'ECONNRESET' ||
               error.code === 'ETIMEDOUT' ||
               error.message.includes('connection');
    }
    
    static async batchQuery(items, queryFn, batchSize = 100) {
        const results = [];
        
        for (let i = 0; i < items.length; i += batchSize) {
            const batch = items.slice(i, i + batchSize);
            const batchResults = await Promise.all(
                batch.map(item => queryFn(item))
            );
            results.push(...batchResults);
        }
        
        return results;
    }
}
```

---

## 🎨 Frontend Performance

### Advanced Code Splitting

```javascript
// dynamic-imports.js
import { lazy, Suspense } from 'react';
import LoadingSpinner from './LoadingSpinner';

// Route-based code splitting
const Dashboard = lazy(() => import('./pages/Dashboard'));
const Profile = lazy(() => import('./pages/Profile'));
const Settings = lazy(() => import('./pages/Settings'));

// Component-based code splitting
const HeavyChart = lazy(() => 
    import('./components/HeavyChart').then(module => ({
        default: module.HeavyChart
    }))
);

// Feature-based code splitting
const AdminPanel = lazy(() => 
    import('./features/admin').then(module => ({
        default: module.AdminPanel
    }))
);

// Conditional loading based on user permissions
const ConditionalComponent = ({ userRole }) => {
    if (userRole === 'admin') {
        const AdminComponent = lazy(() => import('./AdminComponent'));
        return (
            <Suspense fallback={<LoadingSpinner />}>
                <AdminComponent />
            </Suspense>
        );
    }
    return <RegularComponent />;
};

// Preload critical components
const preloadComponent = (componentImport) => {
    componentImport();
};

// Preload on user interaction
const handleMouseEnter = () => {
    preloadComponent(() => import('./HeavyComponent'));
};
```

### Virtual Scrolling Implementation

```javascript
// virtual-scrolling.js
import { useState, useEffect, useMemo, useCallback } from 'react';

const VirtualList = ({ 
    items, 
    itemHeight, 
    containerHeight, 
    renderItem,
    overscan = 5 
}) => {
    const [scrollTop, setScrollTop] = useState(0);
    
    const visibleRange = useMemo(() => {
        const start = Math.floor(scrollTop / itemHeight);
        const end = Math.min(
            start + Math.ceil(containerHeight / itemHeight),
            items.length - 1
        );
        
        return {
            start: Math.max(0, start - overscan),
            end: Math.min(items.length - 1, end + overscan)
        };
    }, [scrollTop, itemHeight, containerHeight, items.length, overscan]);
    
    const visibleItems = useMemo(() => {
        const result = [];
        for (let i = visibleRange.start; i <= visibleRange.end; i++) {
            result.push({
                index: i,
                item: items[i],
                top: i * itemHeight
            });
        }
        return result;
    }, [items, visibleRange, itemHeight]);
    
    const handleScroll = useCallback((e) => {
        setScrollTop(e.target.scrollTop);
    }, []);
    
    const totalHeight = items.length * itemHeight;
    
    return (
        <div 
            style={{ height: containerHeight, overflow: 'auto' }}
            onScroll={handleScroll}
        >
            <div style={{ height: totalHeight, position: 'relative' }}>
                {visibleItems.map(({ index, item, top }) => (
                    <div
                        key={index}
                        style={{
                            position: 'absolute',
                            top,
                            left: 0,
                            right: 0,
                            height: itemHeight
                        }}
                    >
                        {renderItem(item, index)}
                    </div>
                ))}
            </div>
        </div>
    );
};

// Usage
const ItemList = ({ data }) => {
    return (
        <VirtualList
            items={data}
            itemHeight={50}
            containerHeight={400}
            renderItem={(item, index) => (
                <div style={{ padding: '10px', borderBottom: '1px solid #eee' }}>
                    {item.name}
                </div>
            )}
        />
    );
};
```

### Performance Monitoring

```javascript
// performance-monitor.js
class PerformanceMonitor {
    constructor() {
        this.metrics = new Map();
        this.observers = new Map();
        this.setupObservers();
    }
    
    setupObservers() {
        // Core Web Vitals monitoring
        this.observeWebVitals();
        
        // Resource loading monitoring
        this.observeResourceTiming();
        
        // Long task monitoring
        this.observeLongTasks();
        
        // Layout shift monitoring
        this.observeLayoutShifts();
    }
    
    observeWebVitals() {
        // First Contentful Paint
        const paintObserver = new PerformanceObserver((list) => {
            for (const entry of list.getEntries()) {
                if (entry.name === 'first-contentful-paint') {
                    this.recordMetric('FCP', entry.startTime);
                }
            }
        });
        paintObserver.observe({ entryTypes: ['paint'] });
        
        // Largest Contentful Paint
        const lcpObserver = new PerformanceObserver((list) => {
            const entries = list.getEntries();
            const lastEntry = entries[entries.length - 1];
            this.recordMetric('LCP', lastEntry.startTime);
        });
        lcpObserver.observe({ entryTypes: ['largest-contentful-paint'] });
        
        // First Input Delay
        const fidObserver = new PerformanceObserver((list) => {
            for (const entry of list.getEntries()) {
                this.recordMetric('FID', entry.processingStart - entry.startTime);
            }
        });
        fidObserver.observe({ entryTypes: ['first-input'] });
    }
    
    observeResourceTiming() {
        const resourceObserver = new PerformanceObserver((list) => {
            for (const entry of list.getEntries()) {
                this.analyzeResourceTiming(entry);
            }
        });
        resourceObserver.observe({ entryTypes: ['resource'] });
    }
    
    observeLongTasks() {
        if ('PerformanceObserver' in window) {
            const longTaskObserver = new PerformanceObserver((list) => {
                for (const entry of list.getEntries()) {
                    this.recordMetric('longTask', {
                        duration: entry.duration,
                        startTime: entry.startTime
                    });
                }
            });
            longTaskObserver.observe({ entryTypes: ['longtask'] });
        }
    }
    
    observeLayoutShifts() {
        const clsObserver = new PerformanceObserver((list) => {
            let cls = 0;
            for (const entry of list.getEntries()) {
                if (!entry.hadRecentInput) {
                    cls += entry.value;
                }
            }
            this.recordMetric('CLS', cls);
        });
        clsObserver.observe({ entryTypes: ['layout-shift'] });
    }
    
    recordMetric(name, value) {
        if (!this.metrics.has(name)) {
            this.metrics.set(name, []);
        }
        this.metrics.get(name).push({
            value,
            timestamp: Date.now()
        });
        
        // Send to analytics
        this.sendToAnalytics(name, value);
    }
    
    analyzeResourceTiming(entry) {
        const timing = {
            name: entry.name,
            size: entry.transferSize,
            duration: entry.responseEnd - entry.fetchStart,
            cacheHit: entry.transferSize === 0,
            protocol: entry.nextHopProtocol
        };
        
        // Detect slow resources
        if (timing.duration > 1000) { // > 1 second
            this.recordMetric('slowResource', timing);
        }
    }
    
    measureFunction(name, fn) {
        return async (...args) => {
            const start = performance.now();
            try {
                const result = await fn(...args);
                const duration = performance.now() - start;
                this.recordMetric(`function:${name}`, duration);
                return result;
            } catch (error) {
                const duration = performance.now() - start;
                this.recordMetric(`function:${name}:error`, duration);
                throw error;
            }
        };
    }
    
    sendToAnalytics(metric, value) {
        // Send to your analytics service
        if (window.gtag) {
            window.gtag('event', 'performance_metric', {
                metric_name: metric,
                metric_value: value,
                custom_parameter: window.location.pathname
            });
        }
    }
    
    getMetrics() {
        const summary = {};
        for (const [name, values] of this.metrics) {
            summary[name] = {
                count: values.length,
                average: values.reduce((sum, v) => sum + v.value, 0) / values.length,
                latest: values[values.length - 1]?.value
            };
        }
        return summary;
    }
}

// Usage
const monitor = new PerformanceMonitor();

// Measure specific functions
const optimizedFunction = monitor.measureFunction('apiCall', async (url) => {
    const response = await fetch(url);
    return response.json();
});
```

---

## 🚀 Backend Optimization

### Worker Threads for CPU-Intensive Tasks

```javascript
// worker-pool.js
const { Worker, isMainThread, parentPort, workerData } = require('worker_threads');
const os = require('os');

class WorkerPool {
    constructor(workerScript, poolSize = os.cpus().length) {
        this.workers = [];
        this.queue = [];
        this.workerIndex = 0;
        
        for (let i = 0; i < poolSize; i++) {
            this.createWorker(workerScript);
        }
    }
    
    createWorker(workerScript) {
        const worker = new Worker(workerScript);
        worker.on('message', (result) => {
            const { resolve, reject } = worker.currentTask;
            if (result.error) {
                reject(new Error(result.error));
            } else {
                resolve(result.data);
            }
            worker.currentTask = null;
            this.processQueue();
        });
        
        worker.on('error', (error) => {
            if (worker.currentTask) {
                worker.currentTask.reject(error);
                worker.currentTask = null;
            }
        });
        
        this.workers.push(worker);
    }
    
    execute(data) {
        return new Promise((resolve, reject) => {
            const task = { data, resolve, reject };
            
            // Find available worker
            const availableWorker = this.workers.find(w => !w.currentTask);
            
            if (availableWorker) {
                this.assignTask(availableWorker, task);
            } else {
                this.queue.push(task);
            }
        });
    }
    
    assignTask(worker, task) {
        worker.currentTask = task;
        worker.postMessage(task.data);
    }
    
    processQueue() {
        if (this.queue.length === 0) return;
        
        const availableWorker = this.workers.find(w => !w.currentTask);
        if (availableWorker) {
            const task = this.queue.shift();
            this.assignTask(availableWorker, task);
        }
    }
    
    terminate() {
        this.workers.forEach(worker => worker.terminate());
    }
}

// cpu-intensive-worker.js
const { parentPort } = require('worker_threads');

parentPort.on('message', (data) => {
    try {
        const result = performCPUIntensiveTask(data);
        parentPort.postMessage({ data: result });
    } catch (error) {
        parentPort.postMessage({ error: error.message });
    }
});

function performCPUIntensiveTask(data) {
    // Example: Complex calculation
    let result = 0;
    for (let i = 0; i < data.iterations; i++) {
        result += Math.sqrt(i * data.multiplier);
    }
    return result;
}

// Usage in main application
const workerPool = new WorkerPool('./cpu-intensive-worker.js');

app.post('/api/calculate', async (req, res) => {
    try {
        const result = await workerPool.execute({
            iterations: req.body.iterations,
            multiplier: req.body.multiplier
        });
        res.json({ result });
    } catch (error) {
        res.status(500).json({ error: error.message });
    }
});
```

### Advanced Middleware Pipeline

```javascript
// middleware-pipeline.js
const compression = require('compression');
const helmet = require('helmet');
const rateLimit = require('express-rate-limit');

// Custom middleware for request/response optimization
const optimizationMiddleware = {
    // Request body size limit with streaming
    bodyLimit: (limit = '10mb') => {
        return (req, res, next) => {
            if (req.headers['content-length'] > limit) {
                return res.status(413).json({ error: 'Request too large' });
            }
            next();
        };
    },
    
    // Response compression with custom logic
    smartCompression: () => {
        return compression({
            filter: (req, res) => {
                // Don't compress responses with this request header
                if (req.headers['x-no-compression']) {
                    return false;
                }
                
                // Don't compress already compressed content
                const contentType = res.getHeader('content-type');
                if (contentType && contentType.includes('image/')) {
                    return false;
                }
                
                return compression.filter(req, res);
            },
            level: 6, // Balance between compression ratio and CPU usage
            threshold: 1024 // Only compress if larger than 1KB
        });
    },
    
    // Smart caching headers
    cacheControl: (options = {}) => {
        const { 
            maxAge = 3600, 
            public = true, 
            etag = true,
            lastModified = true 
        } = options;
        
        return (req, res, next) => {
            // Set cache headers based on content type
            if (req.url.includes('/api/')) {
                // API responses - short cache
                res.set('Cache-Control', 'private, max-age=300');
            } else if (req.url.match(/\.(js|css|png|jpg|gif|ico|svg)$/)) {
                // Static assets - long cache
                res.set('Cache-Control', `public, max-age=${maxAge}, immutable`);
            } else {
                // HTML pages - moderate cache
                res.set('Cache-Control', `${public ? 'public' : 'private'}, max-age=${maxAge}`);
            }
            
            if (etag) {
                res.set('ETag', `"${Date.now()}"`);
            }
            
            if (lastModified) {
                res.set('Last-Modified', new Date().toUTCString());
            }
            
            next();
        };
    },
    
    // Request timing and logging
    requestTiming: () => {
        return (req, res, next) => {
            const start = process.hrtime();
            
            res.on('finish', () => {
                const [seconds, nanoseconds] = process.hrtime(start);
                const duration = seconds * 1000 + nanoseconds / 1e6;
                
                console.log(`${req.method} ${req.url} - ${res.statusCode} - ${duration.toFixed(2)}ms`);
                
                // Log slow requests
                if (duration > 1000) {
                    console.warn(`Slow request detected: ${req.method} ${req.url} - ${duration.toFixed(2)}ms`);
                }
            });
            
            next();
        };
    }
};

// Rate limiting with Redis backend
const createRateLimiter = (options = {}) => {
    const { windowMs = 15 * 60 * 1000, max = 100, keyGenerator } = options;
    
    return rateLimit({
        windowMs,
        max,
        keyGenerator: keyGenerator || ((req) => req.ip),
        store: new (require('rate-limit-redis'))({
            client: require('./redis-client'),
            prefix: 'rl:'
        }),
        standardHeaders: true,
        legacyHeaders: false,
        handler: (req, res) => {
            res.status(429).json({
                error: 'Too many requests',
                retryAfter: Math.round(windowMs / 1000)
            });
        }
    });
};

// Apply middleware pipeline
app.use(helmet());
app.use(optimizationMiddleware.requestTiming());
app.use(optimizationMiddleware.smartCompression());
app.use(optimizationMiddleware.bodyLimit('10mb'));
app.use(optimizationMiddleware.cacheControl());

// Different rate limits for different endpoints
app.use('/api/auth', createRateLimiter({ windowMs: 15 * 60 * 1000, max: 5 }));
app.use('/api/', createRateLimiter({ windowMs: 15 * 60 * 1000, max: 100 }));
```

This intermediate performance optimization guide covers advanced caching strategies, database optimization, frontend performance techniques, backend optimization, and comprehensive monitoring approaches essential for building high-performance applications.
