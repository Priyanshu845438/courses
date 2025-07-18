
# 🚀 Intermediate Node.js

## Table of Contents
- [Advanced File System Operations](#advanced-file-system-operations)
- [Streams and Buffers](#streams-and-buffers)
- [Process Management](#process-management)
- [Event-Driven Architecture](#event-driven-architecture)
- [Worker Threads](#worker-threads)
- [Performance Optimization](#performance-optimization)
- [Security Best Practices](#security-best-practices)
- [Database Integration](#database-integration)
- [Error Handling Patterns](#error-handling-patterns)

---

## 📁 Advanced File System Operations

### Asynchronous File Operations

```javascript
const fs = require('fs').promises;
const path = require('path');

// Advanced file operations
class FileManager {
    constructor(basePath) {
        this.basePath = basePath;
    }
    
    async createDirectory(dirPath) {
        try {
            await fs.mkdir(path.join(this.basePath, dirPath), { recursive: true });
            console.log(`Directory created: ${dirPath}`);
        } catch (error) {
            if (error.code !== 'EEXIST') {
                throw error;
            }
        }
    }
    
    async copyFile(source, destination) {
        try {
            await fs.copyFile(
                path.join(this.basePath, source),
                path.join(this.basePath, destination)
            );
            console.log(`File copied from ${source} to ${destination}`);
        } catch (error) {
            console.error('Copy failed:', error);
            throw error;
        }
    }
    
    async moveFile(source, destination) {
        try {
            await fs.rename(
                path.join(this.basePath, source),
                path.join(this.basePath, destination)
            );
            console.log(`File moved from ${source} to ${destination}`);
        } catch (error) {
            console.error('Move failed:', error);
            throw error;
        }
    }
    
    async getFileStats(filePath) {
        try {
            const stats = await fs.stat(path.join(this.basePath, filePath));
            return {
                size: stats.size,
                isFile: stats.isFile(),
                isDirectory: stats.isDirectory(),
                created: stats.birthtime,
                modified: stats.mtime,
                accessed: stats.atime
            };
        } catch (error) {
            console.error('Failed to get file stats:', error);
            throw error;
        }
    }
    
    async readDirectoryRecursive(dirPath) {
        const items = [];
        
        try {
            const entries = await fs.readdir(path.join(this.basePath, dirPath), { withFileTypes: true });
            
            for (const entry of entries) {
                const fullPath = path.join(dirPath, entry.name);
                
                if (entry.isDirectory()) {
                    const subItems = await this.readDirectoryRecursive(fullPath);
                    items.push({
                        name: entry.name,
                        type: 'directory',
                        path: fullPath,
                        children: subItems
                    });
                } else {
                    const stats = await this.getFileStats(fullPath);
                    items.push({
                        name: entry.name,
                        type: 'file',
                        path: fullPath,
                        ...stats
                    });
                }
            }
            
            return items;
        } catch (error) {
            console.error('Failed to read directory:', error);
            throw error;
        }
    }
    
    async watchDirectory(dirPath, callback) {
        try {
            const watcher = fs.watch(path.join(this.basePath, dirPath), { recursive: true });
            
            for await (const event of watcher) {
                callback(event);
            }
        } catch (error) {
            console.error('Failed to watch directory:', error);
            throw error;
        }
    }
}

// Usage example
const fileManager = new FileManager('./project');

(async () => {
    await fileManager.createDirectory('uploads/images');
    await fileManager.copyFile('template.html', 'uploads/index.html');
    
    const structure = await fileManager.readDirectoryRecursive('uploads');
    console.log('Directory structure:', JSON.stringify(structure, null, 2));
    
    // Watch for changes
    fileManager.watchDirectory('uploads', (event) => {
        console.log('File changed:', event);
    });
})();
```

---

## 🌊 Streams and Buffers

### Advanced Stream Processing

```javascript
const { Readable, Writable, Transform, pipeline } = require('stream');
const fs = require('fs');
const zlib = require('zlib');
const crypto = require('crypto');

// Custom readable stream
class NumberStream extends Readable {
    constructor(options = {}) {
        super(options);
        this.start = options.start || 1;
        this.end = options.end || 100;
        this.current = this.start;
    }
    
    _read() {
        if (this.current <= this.end) {
            this.push(`${this.current}\n`);
            this.current++;
        } else {
            this.push(null); // End of stream
        }
    }
}

// Custom transform stream
class SquareTransform extends Transform {
    constructor(options = {}) {
        super(options);
    }
    
    _transform(chunk, encoding, callback) {
        const numbers = chunk.toString().split('\n').filter(n => n.trim());
        const squares = numbers.map(n => {
            const num = parseInt(n);
            return isNaN(num) ? '' : `${num}^2 = ${num * num}`;
        }).filter(s => s).join('\n') + '\n';
        
        callback(null, squares);
    }
}

// Custom writable stream
class DatabaseWriter extends Writable {
    constructor(options = {}) {
        super(options);
        this.records = [];
    }
    
    _write(chunk, encoding, callback) {
        const data = chunk.toString();
        this.records.push({
            data: data.trim(),
            timestamp: new Date().toISOString()
        });
        
        console.log('Wrote to database:', data.trim());
        callback();
    }
    
    _final(callback) {
        console.log('Database writer finished. Total records:', this.records.length);
        callback();
    }
}

// Stream pipeline example
const processNumbers = () => {
    const numberStream = new NumberStream({ start: 1, end: 10 });
    const squareTransform = new SquareTransform();
    const databaseWriter = new DatabaseWriter();
    
    pipeline(
        numberStream,
        squareTransform,
        databaseWriter,
        (error) => {
            if (error) {
                console.error('Pipeline failed:', error);
            } else {
                console.log('Pipeline succeeded');
            }
        }
    );
};

// File processing with compression
const compressFile = (inputPath, outputPath) => {
    const readStream = fs.createReadStream(inputPath);
    const writeStream = fs.createWriteStream(outputPath);
    const gzipStream = zlib.createGzip();
    const hashStream = crypto.createHash('md5');
    
    pipeline(
        readStream,
        gzipStream,
        writeStream,
        (error) => {
            if (error) {
                console.error('Compression failed:', error);
            } else {
                console.log('File compressed successfully');
            }
        }
    );
    
    // Calculate hash while reading
    readStream.on('data', (chunk) => {
        hashStream.update(chunk);
    });
    
    readStream.on('end', () => {
        const hash = hashStream.digest('hex');
        console.log('File hash:', hash);
    });
};

// Buffer operations
class BufferUtils {
    static concatenateBuffers(buffers) {
        const totalLength = buffers.reduce((sum, buf) => sum + buf.length, 0);
        const result = Buffer.alloc(totalLength);
        
        let offset = 0;
        for (const buffer of buffers) {
            buffer.copy(result, offset);
            offset += buffer.length;
        }
        
        return result;
    }
    
    static splitBuffer(buffer, delimiter) {
        const chunks = [];
        let start = 0;
        
        for (let i = 0; i < buffer.length; i++) {
            if (buffer[i] === delimiter) {
                chunks.push(buffer.slice(start, i));
                start = i + 1;
            }
        }
        
        if (start < buffer.length) {
            chunks.push(buffer.slice(start));
        }
        
        return chunks;
    }
    
    static bufferToHex(buffer) {
        return buffer.toString('hex').toUpperCase();
    }
    
    static hexToBuffer(hex) {
        return Buffer.from(hex, 'hex');
    }
}

// Usage examples
processNumbers();
compressFile('input.txt', 'output.txt.gz');

const buffer1 = Buffer.from('Hello, ');
const buffer2 = Buffer.from('World!');
const combined = BufferUtils.concatenateBuffers([buffer1, buffer2]);
console.log('Combined buffer:', combined.toString());
```

---

## ⚡ Process Management

### Child Processes and Clustering

```javascript
const cluster = require('cluster');
const os = require('os');
const { spawn, exec, fork } = require('child_process');

// Cluster management
if (cluster.isMaster) {
    const numCPUs = os.cpus().length;
    console.log(`Master process ${process.pid} is running`);
    
    // Fork workers
    for (let i = 0; i < numCPUs; i++) {
        cluster.fork();
    }
    
    cluster.on('exit', (worker, code, signal) => {
        console.log(`Worker ${worker.process.pid} died`);
        cluster.fork(); // Restart worker
    });
    
    // Graceful shutdown
    process.on('SIGTERM', () => {
        console.log('Master process shutting down...');
        
        for (const id in cluster.workers) {
            cluster.workers[id].kill();
        }
        
        setTimeout(() => {
            process.exit(0);
        }, 5000);
    });
} else {
    // Worker process
    const express = require('express');
    const app = express();
    
    app.get('/', (req, res) => {
        res.json({ 
            message: 'Hello from worker',
            pid: process.pid,
            worker: cluster.worker.id
        });
    });
    
    const server = app.listen(3000, () => {
        console.log(`Worker ${process.pid} started`);
    });
    
    // Graceful shutdown for workers
    process.on('SIGTERM', () => {
        console.log(`Worker ${process.pid} shutting down...`);
        
        server.close(() => {
            process.exit(0);
        });
    });
}

// Process utilities
class ProcessManager {
    static async executeCommand(command, args = []) {
        return new Promise((resolve, reject) => {
            const child = spawn(command, args);
            let stdout = '';
            let stderr = '';
            
            child.stdout.on('data', (data) => {
                stdout += data.toString();
            });
            
            child.stderr.on('data', (data) => {
                stderr += data.toString();
            });
            
            child.on('close', (code) => {
                if (code === 0) {
                    resolve({ stdout, stderr });
                } else {
                    reject(new Error(`Command failed with code ${code}: ${stderr}`));
                }
            });
        });
    }
    
    static async executeScript(scriptPath, args = []) {
        return new Promise((resolve, reject) => {
            const child = fork(scriptPath, args);
            
            child.on('message', (message) => {
                console.log('Message from child:', message);
            });
            
            child.on('close', (code) => {
                if (code === 0) {
                    resolve();
                } else {
                    reject(new Error(`Script failed with code ${code}`));
                }
            });
        });
    }
    
    static monitorProcess(pid) {
        const interval = setInterval(() => {
            try {
                process.kill(pid, 0); // Check if process exists
                console.log(`Process ${pid} is running`);
            } catch (error) {
                console.log(`Process ${pid} is not running`);
                clearInterval(interval);
            }
        }, 1000);
        
        return interval;
    }
}

// Usage examples
(async () => {
    try {
        // Execute command
        const result = await ProcessManager.executeCommand('ls', ['-la']);
        console.log('Command output:', result.stdout);
        
        // Execute Node.js script
        await ProcessManager.executeScript('./worker-script.js', ['arg1', 'arg2']);
        
        // Monitor process
        const monitorInterval = ProcessManager.monitorProcess(process.pid);
        
        // Stop monitoring after 10 seconds
        setTimeout(() => {
            clearInterval(monitorInterval);
        }, 10000);
    } catch (error) {
        console.error('Process management error:', error);
    }
})();
```

---

## 🎯 Event-Driven Architecture

### Advanced Event Handling

```javascript
const EventEmitter = require('events');

// Custom event emitter with advanced features
class AdvancedEventEmitter extends EventEmitter {
    constructor() {
        super();
        this.eventHistory = [];
        this.middleware = [];
    }
    
    // Add middleware
    use(middleware) {
        this.middleware.push(middleware);
    }
    
    // Enhanced emit with middleware
    emit(eventName, ...args) {
        const event = {
            name: eventName,
            args: args,
            timestamp: new Date(),
            id: Date.now() + Math.random()
        };
        
        this.eventHistory.push(event);
        
        // Apply middleware
        let processed = event;
        for (const middleware of this.middleware) {
            processed = middleware(processed);
            if (!processed) break; // Middleware can cancel event
        }
        
        if (processed) {
            super.emit(eventName, ...args);
        }
    }
    
    // Get event history
    getEventHistory(eventName) {
        return this.eventHistory.filter(event => 
            eventName ? event.name === eventName : true
        );
    }
    
    // Event replay
    replay(eventName, fromTime) {
        const events = this.getEventHistory(eventName).filter(event =>
            event.timestamp >= fromTime
        );
        
        for (const event of events) {
            super.emit(event.name, ...event.args);
        }
    }
}

// Event-driven application example
class TaskManager extends AdvancedEventEmitter {
    constructor() {
        super();
        this.tasks = new Map();
        this.setupEventHandlers();
        this.setupMiddleware();
    }
    
    setupMiddleware() {
        // Logging middleware
        this.use((event) => {
            console.log(`[${event.timestamp.toISOString()}] Event: ${event.name}`);
            return event;
        });
        
        // Validation middleware
        this.use((event) => {
            if (event.name === 'task:create' && !event.args[0].title) {
                console.error('Task creation failed: title is required');
                return null; // Cancel event
            }
            return event;
        });
    }
    
    setupEventHandlers() {
        this.on('task:create', this.handleTaskCreate.bind(this));
        this.on('task:update', this.handleTaskUpdate.bind(this));
        this.on('task:complete', this.handleTaskComplete.bind(this));
        this.on('task:delete', this.handleTaskDelete.bind(this));
    }
    
    handleTaskCreate(taskData) {
        const task = {
            id: Date.now(),
            title: taskData.title,
            description: taskData.description || '',
            status: 'pending',
            createdAt: new Date(),
            updatedAt: new Date()
        };
        
        this.tasks.set(task.id, task);
        this.emit('task:created', task);
    }
    
    handleTaskUpdate(taskId, updates) {
        const task = this.tasks.get(taskId);
        if (!task) {
            this.emit('task:error', { error: 'Task not found', taskId });
            return;
        }
        
        Object.assign(task, updates, { updatedAt: new Date() });
        this.emit('task:updated', task);
    }
    
    handleTaskComplete(taskId) {
        const task = this.tasks.get(taskId);
        if (!task) {
            this.emit('task:error', { error: 'Task not found', taskId });
            return;
        }
        
        task.status = 'completed';
        task.completedAt = new Date();
        task.updatedAt = new Date();
        
        this.emit('task:completed', task);
    }
    
    handleTaskDelete(taskId) {
        const task = this.tasks.get(taskId);
        if (!task) {
            this.emit('task:error', { error: 'Task not found', taskId });
            return;
        }
        
        this.tasks.delete(taskId);
        this.emit('task:deleted', { taskId, task });
    }
    
    // Public API methods
    createTask(taskData) {
        this.emit('task:create', taskData);
    }
    
    updateTask(taskId, updates) {
        this.emit('task:update', taskId, updates);
    }
    
    completeTask(taskId) {
        this.emit('task:complete', taskId);
    }
    
    deleteTask(taskId) {
        this.emit('task:delete', taskId);
    }
    
    getAllTasks() {
        return Array.from(this.tasks.values());
    }
}

// Usage example
const taskManager = new TaskManager();

// Subscribe to events
taskManager.on('task:created', (task) => {
    console.log('New task created:', task.title);
});

taskManager.on('task:completed', (task) => {
    console.log('Task completed:', task.title);
});

taskManager.on('task:error', (error) => {
    console.error('Task error:', error);
});

// Use the task manager
taskManager.createTask({ title: 'Learn Node.js', description: 'Complete intermediate tutorial' });
taskManager.createTask({ title: 'Build API', description: 'Create REST API with Express' });

const tasks = taskManager.getAllTasks();
if (tasks.length > 0) {
    taskManager.completeTask(tasks[0].id);
}

// View event history
setTimeout(() => {
    const history = taskManager.getEventHistory();
    console.log('Event history:', history.map(e => ({ name: e.name, timestamp: e.timestamp })));
}, 1000);
```

---

## 🧵 Worker Threads

### CPU-Intensive Task Processing

```javascript
const { Worker, isMainThread, parentPort, workerData } = require('worker_threads');
const os = require('os');

// Main thread
if (isMainThread) {
    // Worker pool implementation
    class WorkerPool {
        constructor(workerScript, poolSize = os.cpus().length) {
            this.workerScript = workerScript;
            this.poolSize = poolSize;
            this.workers = [];
            this.queue = [];
            this.init();
        }
        
        init() {
            for (let i = 0; i < this.poolSize; i++) {
                this.createWorker();
            }
        }
        
        createWorker() {
            const worker = new Worker(__filename, {
                workerData: { isWorker: true }
            });
            
            worker.isAvailable = true;
            worker.on('message', (result) => {
                this.handleWorkerMessage(worker, result);
            });
            
            worker.on('error', (error) => {
                console.error('Worker error:', error);
                this.replaceWorker(worker);
            });
            
            this.workers.push(worker);
        }
        
        handleWorkerMessage(worker, result) {
            if (result.resolve) {
                result.resolve(result.data);
            }
            
            worker.isAvailable = true;
            this.processQueue();
        }
        
        replaceWorker(deadWorker) {
            const index = this.workers.indexOf(deadWorker);
            if (index > -1) {
                this.workers.splice(index, 1);
                this.createWorker();
            }
        }
        
        execute(taskData) {
            return new Promise((resolve, reject) => {
                const task = {
                    data: taskData,
                    resolve,
                    reject
                };
                
                this.queue.push(task);
                this.processQueue();
            });
        }
        
        processQueue() {
            if (this.queue.length === 0) return;
            
            const availableWorker = this.workers.find(w => w.isAvailable);
            if (!availableWorker) return;
            
            const task = this.queue.shift();
            availableWorker.isAvailable = false;
            
            availableWorker.postMessage({
                task: task.data,
                resolve: task.resolve,
                reject: task.reject
            });
        }
        
        async terminate() {
            await Promise.all(this.workers.map(worker => worker.terminate()));
        }
    }
    
    // CPU-intensive tasks
    const cpuIntensiveTasks = {
        fibonacci: (n) => {
            if (n <= 1) return n;
            return cpuIntensiveTasks.fibonacci(n - 1) + cpuIntensiveTasks.fibonacci(n - 2);
        },
        
        primeFactors: (n) => {
            const factors = [];
            let divisor = 2;
            
            while (n >= 2) {
                if (n % divisor === 0) {
                    factors.push(divisor);
                    n = n / divisor;
                } else {
                    divisor++;
                }
            }
            
            return factors;
        },
        
        matrixMultiplication: (a, b) => {
            const result = [];
            for (let i = 0; i < a.length; i++) {
                result[i] = [];
                for (let j = 0; j < b[0].length; j++) {
                    let sum = 0;
                    for (let k = 0; k < a[0].length; k++) {
                        sum += a[i][k] * b[k][j];
                    }
                    result[i][j] = sum;
                }
            }
            return result;
        }
    };
    
    // Example usage
    const runMainThread = async () => {
        console.log('Main thread starting...');
        
        const pool = new WorkerPool(__filename, 4);
        
        try {
            // Execute multiple tasks in parallel
            const tasks = [
                { type: 'fibonacci', n: 40 },
                { type: 'primeFactors', n: 1000000 },
                { type: 'fibonacci', n: 35 },
                { type: 'primeFactors', n: 999999 }
            ];
            
            console.log('Starting parallel execution...');
            const startTime = Date.now();
            
            const results = await Promise.all(
                tasks.map(task => pool.execute(task))
            );
            
            const endTime = Date.now();
            console.log(`All tasks completed in ${endTime - startTime}ms`);
            console.log('Results:', results);
            
        } catch (error) {
            console.error('Error:', error);
        } finally {
            await pool.terminate();
        }
    };
    
    runMainThread();
    
} else {
    // Worker thread
    const cpuIntensiveTasks = {
        fibonacci: (n) => {
            if (n <= 1) return n;
            return cpuIntensiveTasks.fibonacci(n - 1) + cpuIntensiveTasks.fibonacci(n - 2);
        },
        
        primeFactors: (n) => {
            const factors = [];
            let divisor = 2;
            
            while (n >= 2) {
                if (n % divisor === 0) {
                    factors.push(divisor);
                    n = n / divisor;
                } else {
                    divisor++;
                }
            }
            
            return factors;
        }
    };
    
    parentPort.on('message', (message) => {
        const { task, resolve, reject } = message;
        
        try {
            let result;
            
            switch (task.type) {
                case 'fibonacci':
                    result = cpuIntensiveTasks.fibonacci(task.n);
                    break;
                case 'primeFactors':
                    result = cpuIntensiveTasks.primeFactors(task.n);
                    break;
                default:
                    throw new Error(`Unknown task type: ${task.type}`);
            }
            
            parentPort.postMessage({
                data: result,
                resolve: resolve
            });
            
        } catch (error) {
            parentPort.postMessage({
                error: error.message,
                reject: reject
            });
        }
    });
}
```

---

## 📚 Summary

### Intermediate Node.js Capabilities:
- **File System**: Advanced file operations, directory management, file watching
- **Streams**: Custom readable/writable/transform streams, pipeline processing
- **Process Management**: Clustering, child processes, graceful shutdown
- **Event-Driven**: Advanced event emitters, middleware, event replay
- **Worker Threads**: CPU-intensive task processing, worker pools
- **Performance**: Buffer operations, memory management, optimization

### Best Practices:
1. **Use streams** for large file processing
2. **Implement clustering** for CPU-intensive applications
3. **Handle errors gracefully** in async operations
4. **Use worker threads** for CPU-bound tasks
5. **Monitor process health** and implement graceful shutdown
6. **Optimize buffer usage** for memory efficiency
7. **Implement proper event handling** patterns

### Next Steps:
1. Learn about microservices architecture
2. Study database integration patterns
3. Explore containerization with Docker
4. Learn about observability and monitoring
5. Study advanced security patterns
