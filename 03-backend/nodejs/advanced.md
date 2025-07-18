
# 🚀 Advanced Node.js

## Table of Contents
- [Microservices Architecture](#microservices-architecture)
- [Advanced Async Patterns](#advanced-async-patterns)
- [Custom Modules and Packages](#custom-modules-and-packages)
- [Performance Profiling](#performance-profiling)
- [Advanced Security](#advanced-security)
- [Observability and Monitoring](#observability-and-monitoring)
- [Memory Management](#memory-management)
- [Native Addons](#native-addons)
- [Enterprise Patterns](#enterprise-patterns)

---

## 🏗️ Microservices Architecture

### Service Discovery and Communication

```javascript
const http = require('http');
const https = require('https');
const EventEmitter = require('events');

// Service registry implementation
class ServiceRegistry extends EventEmitter {
    constructor() {
        super();
        this.services = new Map();
        this.healthChecks = new Map();
        this.startHealthChecking();
    }
    
    register(service) {
        const serviceId = `${service.name}-${service.version}-${Date.now()}`;
        const serviceInfo = {
            id: serviceId,
            name: service.name,
            version: service.version,
            host: service.host,
            port: service.port,
            metadata: service.metadata || {},
            registeredAt: new Date(),
            lastHealthCheck: new Date(),
            healthy: true
        };
        
        this.services.set(serviceId, serviceInfo);
        this.emit('service:registered', serviceInfo);
        
        // Setup health check
        if (service.healthCheck) {
            this.healthChecks.set(serviceId, {
                url: service.healthCheck,
                interval: service.healthCheckInterval || 30000
            });
        }
        
        return serviceId;
    }
    
    unregister(serviceId) {
        const service = this.services.get(serviceId);
        if (service) {
            this.services.delete(serviceId);
            this.healthChecks.delete(serviceId);
            this.emit('service:unregistered', service);
        }
    }
    
    discover(serviceName, version) {
        const services = Array.from(this.services.values())
            .filter(service => 
                service.name === serviceName &&
                service.healthy &&
                (!version || service.version === version)
            );
        
        // Load balancing - round robin
        if (services.length > 0) {
            const index = Math.floor(Math.random() * services.length);
            return services[index];
        }
        
        return null;
    }
    
    startHealthChecking() {
        setInterval(() => {
            this.performHealthChecks();
        }, 10000); // Check every 10 seconds
    }
    
    async performHealthChecks() {
        for (const [serviceId, healthCheck] of this.healthChecks) {
            const service = this.services.get(serviceId);
            if (!service) continue;
            
            try {
                const response = await this.makeHealthCheckRequest(service, healthCheck);
                if (response.statusCode === 200) {
                    service.healthy = true;
                    service.lastHealthCheck = new Date();
                } else {
                    service.healthy = false;
                    this.emit('service:unhealthy', service);
                }
            } catch (error) {
                service.healthy = false;
                this.emit('service:unhealthy', service);
            }
        }
    }
    
    makeHealthCheckRequest(service, healthCheck) {
        return new Promise((resolve, reject) => {
            const options = {
                hostname: service.host,
                port: service.port,
                path: healthCheck.url,
                method: 'GET',
                timeout: 5000
            };
            
            const req = http.request(options, (res) => {
                resolve(res);
            });
            
            req.on('error', reject);
            req.on('timeout', () => reject(new Error('Health check timeout')));
            req.end();
        });
    }
}

// Circuit breaker pattern
class CircuitBreaker {
    constructor(options = {}) {
        this.failureThreshold = options.failureThreshold || 5;
        this.recoveryTimeout = options.recoveryTimeout || 60000;
        this.monitoringPeriod = options.monitoringPeriod || 10000;
        
        this.state = 'CLOSED'; // CLOSED, OPEN, HALF_OPEN
        this.failureCount = 0;
        this.lastFailureTime = null;
        this.successCount = 0;
    }
    
    async call(fn, ...args) {
        if (this.state === 'OPEN') {
            if (Date.now() - this.lastFailureTime > this.recoveryTimeout) {
                this.state = 'HALF_OPEN';
                this.successCount = 0;
            } else {
                throw new Error('Circuit breaker is OPEN');
            }
        }
        
        try {
            const result = await fn(...args);
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
            if (this.successCount >= this.failureThreshold) {
                this.state = 'CLOSED';
            }
        }
    }
    
    onFailure() {
        this.failureCount++;
        this.lastFailureTime = Date.now();
        
        if (this.failureCount >= this.failureThreshold) {
            this.state = 'OPEN';
        }
    }
    
    getState() {
        return {
            state: this.state,
            failureCount: this.failureCount,
            lastFailureTime: this.lastFailureTime
        };
    }
}

// Message queue implementation
class MessageQueue extends EventEmitter {
    constructor() {
        super();
        this.queues = new Map();
        this.consumers = new Map();
        this.deadLetterQueue = [];
    }
    
    publish(queueName, message, options = {}) {
        if (!this.queues.has(queueName)) {
            this.queues.set(queueName, []);
        }
        
        const messageWithMetadata = {
            id: Date.now() + Math.random(),
            content: message,
            timestamp: new Date(),
            attempts: 0,
            maxAttempts: options.maxAttempts || 3,
            delay: options.delay || 0
        };
        
        if (options.delay > 0) {
            setTimeout(() => {
                this.queues.get(queueName).push(messageWithMetadata);
                this.processQueue(queueName);
            }, options.delay);
        } else {
            this.queues.get(queueName).push(messageWithMetadata);
            this.processQueue(queueName);
        }
        
        return messageWithMetadata.id;
    }
    
    subscribe(queueName, handler, options = {}) {
        if (!this.consumers.has(queueName)) {
            this.consumers.set(queueName, []);
        }
        
        this.consumers.get(queueName).push({
            handler,
            options
        });
        
        this.processQueue(queueName);
    }
    
    processQueue(queueName) {
        const queue = this.queues.get(queueName);
        const consumers = this.consumers.get(queueName);
        
        if (!queue || !consumers || queue.length === 0 || consumers.length === 0) {
            return;
        }
        
        const message = queue.shift();
        const consumer = consumers[0]; // Simple round-robin
        
        this.processMessage(message, consumer, queueName);
    }
    
    async processMessage(message, consumer, queueName) {
        try {
            message.attempts++;
            await consumer.handler(message.content);
            this.emit('message:processed', { message, queueName });
        } catch (error) {
            this.emit('message:error', { message, error, queueName });
            
            if (message.attempts < message.maxAttempts) {
                // Retry with exponential backoff
                const delay = Math.pow(2, message.attempts) * 1000;
                setTimeout(() => {
                    this.queues.get(queueName).push(message);
                    this.processQueue(queueName);
                }, delay);
            } else {
                // Send to dead letter queue
                this.deadLetterQueue.push({
                    message,
                    queueName,
                    finalError: error,
                    timestamp: new Date()
                });
                this.emit('message:dead', { message, queueName });
            }
        }
        
        // Process next message
        setImmediate(() => this.processQueue(queueName));
    }
    
    getQueueStats(queueName) {
        const queue = this.queues.get(queueName) || [];
        const consumers = this.consumers.get(queueName) || [];
        
        return {
            queueName,
            messageCount: queue.length,
            consumerCount: consumers.length,
            deadLetterCount: this.deadLetterQueue.filter(item => item.queueName === queueName).length
        };
    }
}
```

---

## 🔄 Advanced Async Patterns

### Async Context and Resource Management

```javascript
const { AsyncLocalStorage } = require('async_hooks');

// Async context management
class AsyncContext {
    constructor() {
        this.storage = new AsyncLocalStorage();
    }
    
    run(context, fn) {
        return this.storage.run(context, fn);
    }
    
    get(key) {
        const context = this.storage.getStore();
        return context ? context[key] : undefined;
    }
    
    set(key, value) {
        const context = this.storage.getStore();
        if (context) {
            context[key] = value;
        }
    }
    
    getAll() {
        return this.storage.getStore() || {};
    }
}

// Request context for web applications
const requestContext = new AsyncContext();

// Async resource pool
class AsyncResourcePool {
    constructor(createResource, destroyResource, options = {}) {
        this.createResource = createResource;
        this.destroyResource = destroyResource;
        this.minSize = options.minSize || 1;
        this.maxSize = options.maxSize || 10;
        this.idleTimeout = options.idleTimeout || 30000;
        
        this.pool = [];
        this.active = new Set();
        this.waiting = [];
        this.destroyed = false;
        
        this.initialize();
    }
    
    async initialize() {
        for (let i = 0; i < this.minSize; i++) {
            const resource = await this.createResource();
            this.pool.push({
                resource,
                createdAt: Date.now(),
                lastUsed: Date.now()
            });
        }
        
        // Start idle timeout checker
        this.startIdleTimeoutChecker();
    }
    
    async acquire() {
        if (this.destroyed) {
            throw new Error('Pool is destroyed');
        }
        
        // Check if there's an available resource
        if (this.pool.length > 0) {
            const item = this.pool.shift();
            this.active.add(item);
            return item.resource;
        }
        
        // Create new resource if under max size
        if (this.active.size < this.maxSize) {
            const resource = await this.createResource();
            const item = {
                resource,
                createdAt: Date.now(),
                lastUsed: Date.now()
            };
            this.active.add(item);
            return resource;
        }
        
        // Wait for resource to become available
        return new Promise((resolve, reject) => {
            this.waiting.push({ resolve, reject });
        });
    }
    
    release(resource) {
        const item = Array.from(this.active).find(item => item.resource === resource);
        if (!item) {
            throw new Error('Resource not found in active set');
        }
        
        this.active.delete(item);
        item.lastUsed = Date.now();
        
        // Check if someone is waiting
        if (this.waiting.length > 0) {
            const { resolve } = this.waiting.shift();
            this.active.add(item);
            resolve(resource);
        } else {
            this.pool.push(item);
        }
    }
    
    async destroy() {
        this.destroyed = true;
        
        // Reject all waiting promises
        for (const { reject } of this.waiting) {
            reject(new Error('Pool is being destroyed'));
        }
        this.waiting = [];
        
        // Destroy all resources
        const allResources = [...this.pool, ...this.active];
        for (const item of allResources) {
            try {
                await this.destroyResource(item.resource);
            } catch (error) {
                console.error('Error destroying resource:', error);
            }
        }
        
        this.pool = [];
        this.active.clear();
    }
    
    startIdleTimeoutChecker() {
        setInterval(() => {
            const now = Date.now();
            const itemsToDestroy = [];
            
            for (let i = this.pool.length - 1; i >= 0; i--) {
                const item = this.pool[i];
                if (now - item.lastUsed > this.idleTimeout && this.pool.length > this.minSize) {
                    itemsToDestroy.push(this.pool.splice(i, 1)[0]);
                }
            }
            
            // Destroy idle resources
            for (const item of itemsToDestroy) {
                this.destroyResource(item.resource).catch(console.error);
            }
        }, 5000);
    }
    
    getStats() {
        return {
            poolSize: this.pool.length,
            activeSize: this.active.size,
            waitingSize: this.waiting.length,
            totalSize: this.pool.length + this.active.size
        };
    }
}

// Advanced Promise utilities
class PromiseUtils {
    static async promiseAllSettledWithLimit(promises, limit) {
        const results = [];
        const executing = [];
        
        for (const promise of promises) {
            const p = Promise.resolve(promise).then(
                value => ({ status: 'fulfilled', value }),
                reason => ({ status: 'rejected', reason })
            );
            
            results.push(p);
            
            if (promises.length >= limit) {
                executing.push(p);
                
                if (executing.length >= limit) {
                    await Promise.race(executing);
                    executing.splice(executing.findIndex(p => p === await Promise.race(executing)), 1);
                }
            }
        }
        
        return Promise.all(results);
    }
    
    static async retry(fn, options = {}) {
        const maxAttempts = options.maxAttempts || 3;
        const delay = options.delay || 1000;
        const backoffFactor = options.backoffFactor || 2;
        
        let lastError;
        
        for (let attempt = 1; attempt <= maxAttempts; attempt++) {
            try {
                return await fn();
            } catch (error) {
                lastError = error;
                
                if (attempt === maxAttempts) {
                    throw error;
                }
                
                const waitTime = delay * Math.pow(backoffFactor, attempt - 1);
                await new Promise(resolve => setTimeout(resolve, waitTime));
            }
        }
        
        throw lastError;
    }
    
    static timeout(promise, ms) {
        return Promise.race([
            promise,
            new Promise((_, reject) => 
                setTimeout(() => reject(new Error('Promise timeout')), ms)
            )
        ]);
    }
    
    static async waterfall(tasks) {
        let result;
        
        for (const task of tasks) {
            result = await task(result);
        }
        
        return result;
    }
}

// Usage examples
const main = async () => {
    // Request context example
    const express = require('express');
    const app = express();
    
    app.use((req, res, next) => {
        const context = {
            requestId: Date.now() + Math.random(),
            userId: req.headers['user-id'],
            timestamp: new Date()
        };
        
        requestContext.run(context, () => {
            next();
        });
    });
    
    app.get('/api/user', (req, res) => {
        const requestId = requestContext.get('requestId');
        const userId = requestContext.get('userId');
        
        res.json({ requestId, userId, message: 'Hello' });
    });
    
    // Resource pool example
    const dbPool = new AsyncResourcePool(
        async () => {
            // Create database connection
            return { id: Math.random(), connected: true };
        },
        async (connection) => {
            // Close database connection
            console.log('Closing connection:', connection.id);
        },
        { minSize: 2, maxSize: 5 }
    );
    
    // Use resource pool
    const connection = await dbPool.acquire();
    console.log('Using connection:', connection.id);
    dbPool.release(connection);
};

main().catch(console.error);
```

---

## 📦 Custom Modules and Packages

### Advanced Module System

```javascript
// Custom module loader
class ModuleLoader {
    constructor() {
        this.cache = new Map();
        this.loading = new Map();
        this.hooks = {
            beforeLoad: [],
            afterLoad: [],
            onError: []
        };
    }
    
    addHook(event, callback) {
        if (this.hooks[event]) {
            this.hooks[event].push(callback);
        }
    }
    
    async loadModule(modulePath, options = {}) {
        if (this.cache.has(modulePath)) {
            return this.cache.get(modulePath);
        }
        
        if (this.loading.has(modulePath)) {
            return this.loading.get(modulePath);
        }
        
        const loadPromise = this._loadModule(modulePath, options);
        this.loading.set(modulePath, loadPromise);
        
        try {
            const module = await loadPromise;
            this.cache.set(modulePath, module);
            return module;
        } finally {
            this.loading.delete(modulePath);
        }
    }
    
    async _loadModule(modulePath, options) {
        try {
            // Execute before load hooks
            for (const hook of this.hooks.beforeLoad) {
                await hook(modulePath, options);
            }
            
            const module = require(modulePath);
            
            // Execute after load hooks
            for (const hook of this.hooks.afterLoad) {
                await hook(modulePath, module);
            }
            
            return module;
        } catch (error) {
            // Execute error hooks
            for (const hook of this.hooks.onError) {
                await hook(modulePath, error);
            }
            
            throw error;
        }
    }
    
    clearCache(modulePath) {
        if (modulePath) {
            this.cache.delete(modulePath);
            delete require.cache[require.resolve(modulePath)];
        } else {
            this.cache.clear();
            Object.keys(require.cache).forEach(key => {
                delete require.cache[key];
            });
        }
    }
    
    getLoadedModules() {
        return Array.from(this.cache.keys());
    }
}

// Plugin system
class PluginSystem extends EventEmitter {
    constructor() {
        super();
        this.plugins = new Map();
        this.hooks = new Map();
    }
    
    async loadPlugin(pluginPath, config = {}) {
        try {
            const pluginModule = require(pluginPath);
            const plugin = new pluginModule(config);
            
            if (typeof plugin.init === 'function') {
                await plugin.init();
            }
            
            this.plugins.set(plugin.name, plugin);
            this.emit('plugin:loaded', plugin);
            
            return plugin;
        } catch (error) {
            this.emit('plugin:error', { pluginPath, error });
            throw error;
        }
    }
    
    registerHook(hookName, callback) {
        if (!this.hooks.has(hookName)) {
            this.hooks.set(hookName, []);
        }
        
        this.hooks.get(hookName).push(callback);
    }
    
    async executeHook(hookName, ...args) {
        const callbacks = this.hooks.get(hookName) || [];
        const results = [];
        
        for (const callback of callbacks) {
            try {
                const result = await callback(...args);
                results.push(result);
            } catch (error) {
                this.emit('hook:error', { hookName, error });
            }
        }
        
        return results;
    }
    
    getPlugin(name) {
        return this.plugins.get(name);
    }
    
    getAllPlugins() {
        return Array.from(this.plugins.values());
    }
}

// Package manager utilities
class PackageManager {
    constructor() {
        this.installedPackages = new Map();
        this.dependencies = new Map();
    }
    
    async installPackage(packageName, version = 'latest') {
        if (this.installedPackages.has(packageName)) {
            console.log(`Package ${packageName} already installed`);
            return;
        }
        
        console.log(`Installing ${packageName}@${version}...`);
        
        // Simulate package installation
        await new Promise(resolve => setTimeout(resolve, 1000));
        
        const packageInfo = {
            name: packageName,
            version: version,
            installedAt: new Date(),
            dependencies: await this.getDependencies(packageName)
        };
        
        this.installedPackages.set(packageName, packageInfo);
        
        // Install dependencies
        for (const dep of packageInfo.dependencies) {
            await this.installPackage(dep.name, dep.version);
        }
        
        console.log(`Package ${packageName}@${version} installed successfully`);
    }
    
    async getDependencies(packageName) {
        // Simulate dependency resolution
        const mockDependencies = {
            'express': [
                { name: 'body-parser', version: '1.19.0' },
                { name: 'cookie-parser', version: '1.4.5' }
            ],
            'mongoose': [
                { name: 'bson', version: '1.1.6' }
            ]
        };
        
        return mockDependencies[packageName] || [];
    }
    
    uninstallPackage(packageName) {
        if (!this.installedPackages.has(packageName)) {
            console.log(`Package ${packageName} not installed`);
            return;
        }
        
        // Check if other packages depend on this
        const dependents = this.findDependents(packageName);
        if (dependents.length > 0) {
            console.log(`Cannot uninstall ${packageName}. Required by: ${dependents.join(', ')}`);
            return;
        }
        
        this.installedPackages.delete(packageName);
        console.log(`Package ${packageName} uninstalled`);
    }
    
    findDependents(packageName) {
        const dependents = [];
        
        for (const [name, info] of this.installedPackages) {
            if (info.dependencies.some(dep => dep.name === packageName)) {
                dependents.push(name);
            }
        }
        
        return dependents;
    }
    
    listPackages() {
        return Array.from(this.installedPackages.entries()).map(([name, info]) => ({
            name,
            version: info.version,
            installedAt: info.installedAt
        }));
    }
    
    getPackageInfo(packageName) {
        return this.installedPackages.get(packageName);
    }
}
```

---

## 🔍 Performance Profiling

### Advanced Performance Monitoring

```javascript
const { performance, PerformanceObserver } = require('perf_hooks');
const v8 = require('v8');

// Performance profiler
class PerformanceProfiler {
    constructor() {
        this.metrics = new Map();
        this.observers = new Map();
        this.setupObservers();
    }
    
    setupObservers() {
        // HTTP performance observer
        const httpObserver = new PerformanceObserver((list) => {
            for (const entry of list.getEntries()) {
                this.recordMetric('http', {
                    name: entry.name,
                    duration: entry.duration,
                    startTime: entry.startTime
                });
            }
        });
        
        httpObserver.observe({ entryTypes: ['http'] });
        this.observers.set('http', httpObserver);
        
        // GC observer
        const gcObserver = new PerformanceObserver((list) => {
            for (const entry of list.getEntries()) {
                this.recordMetric('gc', {
                    kind: entry.kind,
                    duration: entry.duration,
                    startTime: entry.startTime
                });
            }
        });
        
        gcObserver.observe({ entryTypes: ['gc'] });
        this.observers.set('gc', gcObserver);
    }
    
    recordMetric(category, data) {
        if (!this.metrics.has(category)) {
            this.metrics.set(category, []);
        }
        
        this.metrics.get(category).push({
            ...data,
            timestamp: Date.now()
        });
        
        // Keep only last 1000 entries
        const entries = this.metrics.get(category);
        if (entries.length > 1000) {
            entries.splice(0, entries.length - 1000);
        }
    }
    
    mark(name) {
        performance.mark(name);
    }
    
    measure(name, startMark, endMark) {
        performance.measure(name, startMark, endMark);
        
        const entries = performance.getEntriesByName(name);
        const entry = entries[entries.length - 1];
        
        this.recordMetric('custom', {
            name: name,
            duration: entry.duration,
            startTime: entry.startTime
        });
    }
    
    getMetrics(category) {
        return this.metrics.get(category) || [];
    }
    
    getMemoryUsage() {
        const memUsage = process.memoryUsage();
        const heapStats = v8.getHeapStatistics();
        
        return {
            process: {
                rss: memUsage.rss,
                heapTotal: memUsage.heapTotal,
                heapUsed: memUsage.heapUsed,
                external: memUsage.external
            },
            v8: {
                totalHeapSize: heapStats.total_heap_size,
                totalHeapSizeExecutable: heapStats.total_heap_size_executable,
                totalPhysicalSize: heapStats.total_physical_size,
                totalAvailableSize: heapStats.total_available_size,
                usedHeapSize: heapStats.used_heap_size,
                heapSizeLimit: heapStats.heap_size_limit
            }
        };
    }
    
    getCPUUsage() {
        const usage = process.cpuUsage();
        return {
            user: usage.user,
            system: usage.system,
            total: usage.user + usage.system
        };
    }
    
    generateReport() {
        const report = {
            timestamp: new Date().toISOString(),
            uptime: process.uptime(),
            memory: this.getMemoryUsage(),
            cpu: this.getCPUUsage(),
            metrics: {}
        };
        
        for (const [category, entries] of this.metrics) {
            const durations = entries.map(entry => entry.duration).filter(d => d);
            if (durations.length > 0) {
                report.metrics[category] = {
                    count: entries.length,
                    avgDuration: durations.reduce((a, b) => a + b, 0) / durations.length,
                    minDuration: Math.min(...durations),
                    maxDuration: Math.max(...durations)
                };
            }
        }
        
        return report;
    }
}

// CPU profiler
class CPUProfiler {
    constructor() {
        this.profiling = false;
        this.profile = null;
    }
    
    startProfiling() {
        if (this.profiling) {
            throw new Error('Profiling already started');
        }
        
        const inspector = require('inspector');
        const session = new inspector.Session();
        session.connect();
        
        return new Promise((resolve, reject) => {
            session.post('Profiler.enable', (err) => {
                if (err) {
                    reject(err);
                    return;
                }
                
                session.post('Profiler.start', (err) => {
                    if (err) {
                        reject(err);
                        return;
                    }
                    
                    this.profiling = true;
                    this.session = session;
                    resolve();
                });
            });
        });
    }
    
    stopProfiling() {
        if (!this.profiling) {
            throw new Error('Profiling not started');
        }
        
        return new Promise((resolve, reject) => {
            this.session.post('Profiler.stop', (err, { profile }) => {
                if (err) {
                    reject(err);
                    return;
                }
                
                this.profile = profile;
                this.profiling = false;
                this.session.disconnect();
                resolve(profile);
            });
        });
    }
    
    saveProfile(filename) {
        if (!this.profile) {
            throw new Error('No profile available');
        }
        
        const fs = require('fs');
        fs.writeFileSync(filename, JSON.stringify(this.profile, null, 2));
    }
    
    analyzeProfile() {
        if (!this.profile) {
            throw new Error('No profile available');
        }
        
        const nodes = this.profile.nodes;
        const samples = this.profile.samples;
        
        // Count samples per function
        const functionCounts = {};
        for (const sample of samples) {
            const node = nodes.find(n => n.id === sample);
            if (node) {
                const functionName = node.callFrame.functionName || 'anonymous';
                functionCounts[functionName] = (functionCounts[functionName] || 0) + 1;
            }
        }
        
        // Sort by count
        const sortedFunctions = Object.entries(functionCounts)
            .sort(([, a], [, b]) => b - a)
            .slice(0, 10);
        
        return {
            totalSamples: samples.length,
            topFunctions: sortedFunctions.map(([name, count]) => ({
                name,
                count,
                percentage: (count / samples.length * 100).toFixed(2)
            }))
        };
    }
}

// Example usage
const profiler = new PerformanceProfiler();

// Profile a function
const profiledFunction = (name, fn) => {
    return async (...args) => {
        const startMark = `${name}-start`;
        const endMark = `${name}-end`;
        
        profiler.mark(startMark);
        
        try {
            const result = await fn(...args);
            profiler.mark(endMark);
            profiler.measure(name, startMark, endMark);
            return result;
        } catch (error) {
            profiler.mark(endMark);
            profiler.measure(name, startMark, endMark);
            throw error;
        }
    };
};

// Example usage
const slowFunction = profiledFunction('slowFunction', async () => {
    // Simulate slow operation
    await new Promise(resolve => setTimeout(resolve, 100));
    return 'result';
});

// Generate performance report
setInterval(() => {
    const report = profiler.generateReport();
    console.log('Performance Report:', JSON.stringify(report, null, 2));
}, 10000);
```

---

## 📚 Summary

### Advanced Node.js Capabilities:
- **Microservices**: Service registry, circuit breaker, message queues
- **Async Patterns**: Context management, resource pooling, promise utilities
- **Module System**: Custom loaders, plugin systems, package management
- **Performance**: Profiling, monitoring, memory management
- **Security**: Advanced authentication, input validation, rate limiting
- **Observability**: Metrics collection, distributed tracing, logging

### Enterprise Best Practices:
1. **Implement circuit breakers** for service resilience
2. **Use async context** for request tracing
3. **Implement resource pooling** for database connections
4. **Profile performance** regularly in production
5. **Monitor memory usage** and garbage collection
6. **Use structured logging** for better observability
7. **Implement graceful shutdown** for all services
8. **Use health checks** for service monitoring

### Next Steps:
1. Study container orchestration with Kubernetes
2. Learn about service mesh architectures
3. Explore serverless computing patterns
4. Study event sourcing and CQRS
5. Learn about distributed system patterns
