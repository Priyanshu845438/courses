
# ⚡ Performance Optimization Basics

## Table of Contents
- [Performance Fundamentals](#performance-fundamentals)
- [Frontend Optimization](#frontend-optimization)
- [Backend Optimization](#backend-optimization)
- [Database Optimization](#database-optimization)
- [Caching Strategies](#caching-strategies)
- [Bundle Optimization](#bundle-optimization)
- [Image Optimization](#image-optimization)
- [Monitoring Performance](#monitoring-performance)

---

## 🎯 Performance Fundamentals

### Key Performance Metrics

```javascript
// Core Web Vitals
const webVitals = {
  // Largest Contentful Paint (LCP) - Loading
  LCP: 'Should be less than 2.5 seconds',
  
  // First Input Delay (FID) - Interactivity
  FID: 'Should be less than 100 milliseconds',
  
  // Cumulative Layout Shift (CLS) - Visual Stability
  CLS: 'Should be less than 0.1'
};

// Additional metrics
const performanceMetrics = {
  TTFB: 'Time to First Byte',
  FCP: 'First Contentful Paint',
  TTI: 'Time to Interactive',
  FMP: 'First Meaningful Paint'
};
```

### Performance Budget

```javascript
// performance-budget.json
{
  "budget": [
    {
      "path": "/*",
      "timings": [
        {
          "metric": "interactive",
          "budget": 5000
        },
        {
          "metric": "first-meaningful-paint",
          "budget": 2000
        }
      ],
      "resourceSizes": [
        {
          "resourceType": "script",
          "budget": 400
        },
        {
          "resourceType": "total",
          "budget": 2000
        }
      ]
    }
  ]
}
```

---

## 🎨 Frontend Optimization

### Critical Rendering Path

```html
<!DOCTYPE html>
<html lang="en">
<head>
    <meta charset="UTF-8">
    <meta name="viewport" content="width=device-width, initial-scale=1.0">
    
    <!-- Critical CSS inline -->
    <style>
        /* Critical above-the-fold styles */
        body { font-family: Arial, sans-serif; margin: 0; }
        .header { background: #333; color: white; padding: 1rem; }
        .hero { height: 400px; background: #f4f4f4; }
    </style>
    
    <!-- Preload critical resources -->
    <link rel="preload" href="/fonts/main.woff2" as="font" type="font/woff2" crossorigin>
    <link rel="preload" href="/css/main.css" as="style">
    <link rel="preload" href="/js/main.js" as="script">
    
    <!-- DNS prefetch for external domains -->
    <link rel="dns-prefetch" href="//fonts.googleapis.com">
    <link rel="dns-prefetch" href="//api.example.com">
    
    <!-- Async load non-critical CSS -->
    <link rel="preload" href="/css/non-critical.css" as="style" onload="this.onload=null;this.rel='stylesheet'">
    <noscript><link rel="stylesheet" href="/css/non-critical.css"></noscript>
</head>
<body>
    <!-- Content -->
    
    <!-- Defer non-critical JavaScript -->
    <script src="/js/main.js" defer></script>
    <script src="/js/analytics.js" async></script>
</body>
</html>
```

### Lazy Loading

```javascript
// Intersection Observer for lazy loading
class LazyLoader {
    constructor() {
        this.imageObserver = new IntersectionObserver(
            (entries, observer) => {
                entries.forEach(entry => {
                    if (entry.isIntersecting) {
                        const img = entry.target;
                        this.loadImage(img);
                        observer.unobserve(img);
                    }
                });
            },
            {
                rootMargin: '50px 0px',
                threshold: 0.01
            }
        );
        
        this.componentObserver = new IntersectionObserver(
            (entries) => {
                entries.forEach(entry => {
                    if (entry.isIntersecting) {
                        this.loadComponent(entry.target);
                    }
                });
            }
        );
        
        this.init();
    }
    
    init() {
        // Lazy load images
        document.querySelectorAll('img[data-src]').forEach(img => {
            this.imageObserver.observe(img);
        });
        
        // Lazy load components
        document.querySelectorAll('[data-component]').forEach(el => {
            this.componentObserver.observe(el);
        });
    }
    
    loadImage(img) {
        const src = img.dataset.src;
        const srcset = img.dataset.srcset;
        
        // Create new image to preload
        const imageLoader = new Image();
        imageLoader.onload = () => {
            img.src = src;
            if (srcset) img.srcset = srcset;
            img.classList.add('loaded');
        };
        imageLoader.src = src;
    }
    
    async loadComponent(element) {
        const componentName = element.dataset.component;
        
        try {
            // Dynamic import for code splitting
            const module = await import(`./components/${componentName}.js`);
            const Component = module.default;
            
            // Initialize component
            new Component(element);
            element.classList.add('component-loaded');
        } catch (error) {
            console.error(`Failed to load component ${componentName}:`, error);
        }
    }
}

// Initialize lazy loader
document.addEventListener('DOMContentLoaded', () => {
    new LazyLoader();
});
```

### React Performance Optimization

```jsx
import React, { memo, useMemo, useCallback, lazy, Suspense } from 'react';

// Lazy load components
const HeavyComponent = lazy(() => import('./HeavyComponent'));

// Memoized component
const ExpensiveListItem = memo(({ item, onUpdate }) => {
    const expensiveCalculation = useMemo(() => {
        return item.data.reduce((sum, val) => sum + val * 2, 0);
    }, [item.data]);
    
    return (
        <div>
            <h3>{item.title}</h3>
            <p>Calculated: {expensiveCalculation}</p>
            <button onClick={() => onUpdate(item.id)}>Update</button>
        </div>
    );
});

function OptimizedList({ items, updateItem }) {
    // Memoize callback to prevent unnecessary re-renders
    const handleUpdate = useCallback((id) => {
        updateItem(id);
    }, [updateItem]);
    
    // Memoize filtered items
    const visibleItems = useMemo(() => {
        return items.filter(item => item.visible);
    }, [items]);
    
    return (
        <div>
            {/* Virtual scrolling for large lists */}
            <VirtualizedList
                items={visibleItems}
                renderItem={(item) => (
                    <ExpensiveListItem
                        key={item.id}
                        item={item}
                        onUpdate={handleUpdate}
                    />
                )}
                itemHeight={100}
                containerHeight={400}
            />
            
            {/* Lazy load heavy component */}
            <Suspense fallback={<div>Loading...</div>}>
                <HeavyComponent />
            </Suspense>
        </div>
    );
}

// Virtual scrolling implementation
function VirtualizedList({ items, renderItem, itemHeight, containerHeight }) {
    const [scrollTop, setScrollTop] = useState(0);
    
    const visibleCount = Math.ceil(containerHeight / itemHeight);
    const startIndex = Math.floor(scrollTop / itemHeight);
    const endIndex = Math.min(startIndex + visibleCount, items.length);
    
    const visibleItems = items.slice(startIndex, endIndex);
    const totalHeight = items.length * itemHeight;
    const offsetY = startIndex * itemHeight;
    
    return (
        <div
            style={{ height: containerHeight, overflow: 'auto' }}
            onScroll={(e) => setScrollTop(e.target.scrollTop)}
        >
            <div style={{ height: totalHeight, position: 'relative' }}>
                <div style={{ transform: `translateY(${offsetY}px)` }}>
                    {visibleItems.map(renderItem)}
                </div>
            </div>
        </div>
    );
}
```

---

## 🔧 Backend Optimization

### Node.js Performance

```javascript
// Cluster for multi-core utilization
const cluster = require('cluster');
const numCPUs = require('os').cpus().length;

if (cluster.isMaster) {
    console.log(`Master ${process.pid} is running`);
    
    // Fork workers
    for (let i = 0; i < numCPUs; i++) {
        cluster.fork();
    }
    
    cluster.on('exit', (worker, code, signal) => {
        console.log(`Worker ${worker.process.pid} died`);
        cluster.fork(); // Restart worker
    });
} else {
    // Worker process
    require('./app.js');
    console.log(`Worker ${process.pid} started`);
}

// Express.js optimizations
const express = require('express');
const compression = require('compression');
const helmet = require('helmet');

const app = express();

// Security and compression middleware
app.use(helmet());
app.use(compression());

// Efficient middleware ordering
app.use(express.static('public', {
    maxAge: '1y', // Cache static files for 1 year
    etag: false
}));

// Connection pooling for database
const mongoose = require('mongoose');

mongoose.connect(process.env.MONGODB_URI, {
    maxPoolSize: 10, // Maintain up to 10 socket connections
    serverSelectionTimeoutMS: 5000, // Keep trying to send operations for 5 seconds
    socketTimeoutMS: 45000, // Close sockets after 45 seconds of inactivity
    bufferCommands: false, // Disable mongoose buffering
    bufferMaxEntries: 0 // Disable mongoose buffering
});

// Async error handling
const asyncHandler = (fn) => (req, res, next) => {
    Promise.resolve(fn(req, res, next)).catch(next);
};

// Optimized route handlers
app.get('/api/users', asyncHandler(async (req, res) => {
    const { page = 1, limit = 10 } = req.query;
    
    // Pagination and field selection
    const users = await User
        .find()
        .select('name email createdAt') // Only select needed fields
        .limit(limit * 1)
        .skip((page - 1) * limit)
        .lean(); // Return plain objects instead of Mongoose documents
    
    res.json({
        users,
        totalPages: Math.ceil(await User.countDocuments() / limit),
        currentPage: page
    });
}));
```

### API Rate Limiting

```javascript
const rateLimit = require('express-rate-limit');
const slowDown = require('express-slow-down');

// Rate limiting
const limiter = rateLimit({
    windowMs: 15 * 60 * 1000, // 15 minutes
    max: 100, // limit each IP to 100 requests per windowMs
    message: 'Too many requests from this IP, please try again later.',
    standardHeaders: true,
    legacyHeaders: false
});

// Slow down repeated requests
const speedLimiter = slowDown({
    windowMs: 15 * 60 * 1000, // 15 minutes
    delayAfter: 50, // allow 50 requests per 15 minutes at full speed
    delayMs: 500 // slow down subsequent requests by 500ms per request
});

// Different limits for different routes
const authLimiter = rateLimit({
    windowMs: 15 * 60 * 1000,
    max: 5, // Stricter limit for auth routes
    message: 'Too many authentication attempts, please try again later.'
});

app.use('/api/', limiter);
app.use('/api/', speedLimiter);
app.use('/auth/', authLimiter);
```

---

## 🗄️ Database Optimization

### MongoDB Optimization

```javascript
// Efficient queries with indexes
db.users.createIndex({ email: 1 }); // Single field index
db.posts.createIndex({ userId: 1, createdAt: -1 }); // Compound index
db.posts.createIndex({ title: "text", content: "text" }); // Text index

// Aggregation pipeline optimization
const pipeline = [
    // Match early to reduce documents
    { $match: { status: 'published', createdAt: { $gte: lastWeek } } },
    
    // Project only needed fields
    { $project: { title: 1, author: 1, views: 1, createdAt: 1 } },
    
    // Sort (should use index)
    { $sort: { views: -1 } },
    
    // Limit results
    { $limit: 10 }
];

const popularPosts = await Post.aggregate(pipeline);

// Efficient pagination with cursor-based approach
async function getPaginatedPosts(cursor = null, limit = 10) {
    const query = cursor 
        ? { createdAt: { $lt: cursor } }
        : {};
    
    const posts = await Post
        .find(query)
        .sort({ createdAt: -1 })
        .limit(limit + 1) // Get one extra to check if there's a next page
        .lean();
    
    const hasNextPage = posts.length > limit;
    if (hasNextPage) posts.pop(); // Remove the extra item
    
    const nextCursor = posts.length > 0 
        ? posts[posts.length - 1].createdAt 
        : null;
    
    return {
        posts,
        nextCursor: hasNextPage ? nextCursor : null,
        hasNextPage
    };
}
```

### SQL Query Optimization

```sql
-- Use appropriate indexes
CREATE INDEX idx_users_email ON users(email);
CREATE INDEX idx_posts_user_date ON posts(user_id, created_at);
CREATE INDEX idx_posts_status_date ON posts(status, created_at);

-- Efficient queries
-- ❌ Avoid SELECT *
SELECT * FROM posts;

-- ✅ Select only needed columns
SELECT id, title, excerpt, created_at FROM posts;

-- ❌ Avoid functions in WHERE clauses
SELECT * FROM posts WHERE YEAR(created_at) = 2023;

-- ✅ Use range queries
SELECT * FROM posts 
WHERE created_at >= '2023-01-01' 
AND created_at < '2024-01-01';

-- ❌ Avoid LIKE with leading wildcards
SELECT * FROM users WHERE email LIKE '%@gmail.com';

-- ✅ Use proper indexing and specific queries
SELECT * FROM users WHERE email LIKE 'john%';

-- Efficient JOINs
SELECT 
    u.username,
    p.title,
    p.created_at
FROM users u
INNER JOIN posts p ON u.id = p.user_id
WHERE u.active = 1 
AND p.status = 'published'
ORDER BY p.created_at DESC
LIMIT 10;

-- Pagination with OFFSET can be slow for large datasets
-- ❌ Slow for large offsets
SELECT * FROM posts ORDER BY created_at DESC LIMIT 10 OFFSET 10000;

-- ✅ Use cursor-based pagination
SELECT * FROM posts 
WHERE created_at < '2023-01-01 12:00:00'
ORDER BY created_at DESC 
LIMIT 10;
```

---

## 💾 Caching Strategies

### In-Memory Caching

```javascript
const NodeCache = require('node-cache');
const cache = new NodeCache({ stdTTL: 600 }); // 10 minutes default TTL

// Cache middleware
const cacheMiddleware = (duration = 300) => {
    return (req, res, next) => {
        const key = req.originalUrl;
        const cached = cache.get(key);
        
        if (cached) {
            console.log('Cache hit for:', key);
            return res.json(cached);
        }
        
        // Override res.json to cache the response
        const originalJson = res.json;
        res.json = function(data) {
            cache.set(key, data, duration);
            console.log('Cached response for:', key);
            originalJson.call(this, data);
        };
        
        next();
    };
};

// Usage
app.get('/api/posts', cacheMiddleware(600), async (req, res) => {
    const posts = await Post.find().lean();
    res.json(posts);
});
```

### Redis Caching

```javascript
const redis = require('redis');
const client = redis.createClient({
    host: process.env.REDIS_HOST || 'localhost',
    port: process.env.REDIS_PORT || 6379
});

// Cache abstraction
class CacheService {
    constructor() {
        this.client = client;
    }
    
    async get(key) {
        try {
            const cached = await this.client.get(key);
            return cached ? JSON.parse(cached) : null;
        } catch (error) {
            console.error('Cache get error:', error);
            return null;
        }
    }
    
    async set(key, data, ttl = 3600) {
        try {
            await this.client.setex(key, ttl, JSON.stringify(data));
        } catch (error) {
            console.error('Cache set error:', error);
        }
    }
    
    async del(key) {
        try {
            await this.client.del(key);
        } catch (error) {
            console.error('Cache delete error:', error);
        }
    }
    
    async flush() {
        try {
            await this.client.flushall();
        } catch (error) {
            console.error('Cache flush error:', error);
        }
    }
}

const cacheService = new CacheService();

// Cache-aside pattern
async function getUser(userId) {
    const cacheKey = `user:${userId}`;
    
    // Try to get from cache first
    let user = await cacheService.get(cacheKey);
    
    if (!user) {
        // Cache miss - get from database
        user = await User.findById(userId).lean();
        
        if (user) {
            // Cache for 1 hour
            await cacheService.set(cacheKey, user, 3600);
        }
    }
    
    return user;
}

// Write-through pattern
async function updateUser(userId, updateData) {
    // Update database
    const user = await User.findByIdAndUpdate(
        userId, 
        updateData, 
        { new: true }
    ).lean();
    
    // Update cache
    const cacheKey = `user:${userId}`;
    await cacheService.set(cacheKey, user, 3600);
    
    return user;
}
```

### HTTP Caching

```javascript
// HTTP caching headers
app.use('/api/static', (req, res, next) => {
    // Cache for 1 year
    res.set('Cache-Control', 'public, max-age=31536000, immutable');
    res.set('Expires', new Date(Date.now() + 31536000000).toUTCString());
    next();
});

app.use('/api/dynamic', (req, res, next) => {
    // Cache for 5 minutes, allow stale for 1 hour
    res.set('Cache-Control', 'public, max-age=300, stale-while-revalidate=3600');
    next();
});

// ETags for conditional requests
app.use('/api/users/:id', async (req, res) => {
    const user = await User.findById(req.params.id);
    
    if (!user) {
        return res.status(404).json({ error: 'User not found' });
    }
    
    // Generate ETag based on user data
    const etag = crypto
        .createHash('md5')
        .update(JSON.stringify(user) + user.updatedAt.getTime())
        .digest('hex');
    
    res.set('ETag', `"${etag}"`);
    
    // Check if client has current version
    if (req.headers['if-none-match'] === `"${etag}"`) {
        return res.status(304).end();
    }
    
    res.json(user);
});
```

---

## 📦 Bundle Optimization

### Webpack Configuration

```javascript
// webpack.config.js
const path = require('path');
const TerserPlugin = require('terser-webpack-plugin');
const MiniCssExtractPlugin = require('mini-css-extract-plugin');
const OptimizeCSSAssetsPlugin = require('optimize-css-assets-webpack-plugin');
const { BundleAnalyzerPlugin } = require('webpack-bundle-analyzer');

module.exports = {
    mode: 'production',
    entry: {
        main: './src/index.js',
        vendor: ['react', 'react-dom', 'lodash']
    },
    output: {
        path: path.resolve(__dirname, 'dist'),
        filename: '[name].[contenthash].js',
        chunkFilename: '[name].[contenthash].chunk.js',
        publicPath: '/static/'
    },
    optimization: {
        minimizer: [
            new TerserPlugin({
                parallel: true,
                terserOptions: {
                    compress: {
                        drop_console: true // Remove console.log in production
                    }
                }
            }),
            new OptimizeCSSAssetsPlugin()
        ],
        splitChunks: {
            chunks: 'all',
            cacheGroups: {
                vendor: {
                    test: /[\\/]node_modules[\\/]/,
                    name: 'vendors',
                    chunks: 'all'
                },
                common: {
                    name: 'common',
                    minChunks: 2,
                    chunks: 'all',
                    enforce: true
                }
            }
        }
    },
    plugins: [
        new MiniCssExtractPlugin({
            filename: '[name].[contenthash].css'
        }),
        process.env.ANALYZE && new BundleAnalyzerPlugin()
    ].filter(Boolean),
    module: {
        rules: [
            {
                test: /\.js$/,
                exclude: /node_modules/,
                use: 'babel-loader'
            },
            {
                test: /\.css$/,
                use: [MiniCssExtractPlugin.loader, 'css-loader']
            }
        ]
    }
};

// Tree shaking with ES modules
// ✅ Good - allows tree shaking
import { debounce } from 'lodash-es';

// ❌ Bad - imports entire library
import _ from 'lodash';
```

### Code Splitting

```javascript
// Dynamic imports for code splitting
const HomePage = lazy(() => import('./pages/HomePage'));
const AboutPage = lazy(() => import('./pages/AboutPage'));
const UserPage = lazy(() => 
    import('./pages/UserPage').then(module => ({
        default: module.UserPage
    }))
);

// Route-based code splitting
function App() {
    return (
        <Router>
            <Suspense fallback={<div>Loading...</div>}>
                <Routes>
                    <Route path="/" element={<HomePage />} />
                    <Route path="/about" element={<AboutPage />} />
                    <Route path="/user/:id" element={<UserPage />} />
                </Routes>
            </Suspense>
        </Router>
    );
}

// Component-based code splitting
function ProductList() {
    const [showFilters, setShowFilters] = useState(false);
    const [FilterComponent, setFilterComponent] = useState(null);
    
    const loadFilters = async () => {
        if (!FilterComponent) {
            const module = await import('./ProductFilters');
            setFilterComponent(() => module.default);
        }
        setShowFilters(true);
    };
    
    return (
        <div>
            <button onClick={loadFilters}>
                Show Filters
            </button>
            {showFilters && FilterComponent && (
                <FilterComponent />
            )}
        </div>
    );
}
```

---

## 🖼️ Image Optimization

### Responsive Images

```html
<!-- Responsive images with srcset -->
<img
    src="image-800w.jpg"
    srcset="
        image-400w.jpg 400w,
        image-800w.jpg 800w,
        image-1200w.jpg 1200w
    "
    sizes="
        (max-width: 400px) 100vw,
        (max-width: 800px) 50vw,
        33vw
    "
    alt="Description"
    loading="lazy"
>

<!-- Art direction with picture element -->
<picture>
    <source 
        media="(max-width: 799px)" 
        srcset="mobile-image.jpg"
    >
    <source 
        media="(min-width: 800px)" 
        srcset="desktop-image.jpg"
    >
    <img src="fallback-image.jpg" alt="Description">
</picture>

<!-- Modern image formats with fallbacks -->
<picture>
    <source srcset="image.avif" type="image/avif">
    <source srcset="image.webp" type="image/webp">
    <img src="image.jpg" alt="Description">
</picture>
```

### Image Processing Service

```javascript
const sharp = require('sharp');
const path = require('path');

class ImageProcessor {
    constructor() {
        this.supportedFormats = ['jpeg', 'jpg', 'png', 'webp', 'avif'];
        this.cache = new Map();
    }
    
    async processImage(inputPath, options = {}) {
        const {
            width,
            height,
            quality = 80,
            format = 'webp',
            fit = 'cover'
        } = options;
        
        const cacheKey = `${inputPath}-${JSON.stringify(options)}`;
        
        if (this.cache.has(cacheKey)) {
            return this.cache.get(cacheKey);
        }
        
        try {
            let pipeline = sharp(inputPath);
            
            // Resize if dimensions provided
            if (width || height) {
                pipeline = pipeline.resize(width, height, { fit });
            }
            
            // Convert format and optimize
            switch (format) {
                case 'webp':
                    pipeline = pipeline.webp({ quality });
                    break;
                case 'avif':
                    pipeline = pipeline.avif({ quality });
                    break;
                case 'jpeg':
                case 'jpg':
                    pipeline = pipeline.jpeg({ quality, progressive: true });
                    break;
                case 'png':
                    pipeline = pipeline.png({ 
                        compressionLevel: 9,
                        adaptiveFiltering: true
                    });
                    break;
            }
            
            const processedBuffer = await pipeline.toBuffer();
            this.cache.set(cacheKey, processedBuffer);
            
            return processedBuffer;
        } catch (error) {
            console.error('Image processing error:', error);
            throw error;
        }
    }
    
    // Express middleware for on-the-fly image processing
    middleware() {
        return async (req, res, next) => {
            const { width, height, quality, format } = req.query;
            const imagePath = path.join(__dirname, 'images', req.params.image);
            
            try {
                const processedImage = await this.processImage(imagePath, {
                    width: width ? parseInt(width) : undefined,
                    height: height ? parseInt(height) : undefined,
                    quality: quality ? parseInt(quality) : undefined,
                    format: format || 'webp'
                });
                
                res.set('Content-Type', `image/${format || 'webp'}`);
                res.set('Cache-Control', 'public, max-age=31536000');
                res.send(processedImage);
            } catch (error) {
                next(error);
            }
        };
    }
}

const imageProcessor = new ImageProcessor();

// Route for dynamic image processing
app.get('/images/:image', imageProcessor.middleware());
```

---

## 📊 Monitoring Performance

### Performance Monitoring

```javascript
// Web Performance API
class PerformanceMonitor {
    constructor() {
        this.metrics = {};
        this.init();
    }
    
    init() {
        // Monitor page load performance
        window.addEventListener('load', () => {
            this.collectNavigationMetrics();
            this.collectResourceMetrics();
        });
        
        // Monitor Core Web Vitals
        this.observeCoreWebVitals();
        
        // Monitor user interactions
        this.observeUserMetrics();
    }
    
    collectNavigationMetrics() {
        const navigation = performance.getEntriesByType('navigation')[0];
        
        this.metrics.navigation = {
            dns: navigation.domainLookupEnd - navigation.domainLookupStart,
            connection: navigation.connectEnd - navigation.connectStart,
            ttfb: navigation.responseStart - navigation.requestStart,
            download: navigation.responseEnd - navigation.responseStart,
            domParsing: navigation.domContentLoadedEventEnd - navigation.responseEnd,
            totalPageLoad: navigation.loadEventEnd - navigation.navigationStart
        };
        
        this.sendMetrics('navigation', this.metrics.navigation);
    }
    
    collectResourceMetrics() {
        const resources = performance.getEntriesByType('resource');
        
        const resourceMetrics = {
            totalResources: resources.length,
            totalSize: resources.reduce((sum, resource) => 
                sum + (resource.transferSize || 0), 0
            ),
            slowestResource: Math.max(...resources.map(r => r.duration)),
            resourceTypes: {}
        };
        
        // Group by resource type
        resources.forEach(resource => {
            const type = this.getResourceType(resource.name);
            if (!resourceMetrics.resourceTypes[type]) {
                resourceMetrics.resourceTypes[type] = {
                    count: 0,
                    totalSize: 0,
                    totalDuration: 0
                };
            }
            
            resourceMetrics.resourceTypes[type].count++;
            resourceMetrics.resourceTypes[type].totalSize += resource.transferSize || 0;
            resourceMetrics.resourceTypes[type].totalDuration += resource.duration;
        });
        
        this.sendMetrics('resources', resourceMetrics);
    }
    
    observeCoreWebVitals() {
        // Largest Contentful Paint
        new PerformanceObserver((list) => {
            const entries = list.getEntries();
            const lastEntry = entries[entries.length - 1];
            this.sendMetrics('lcp', { value: lastEntry.startTime });
        }).observe({ entryTypes: ['largest-contentful-paint'] });
        
        // First Input Delay
        new PerformanceObserver((list) => {
            const firstInput = list.getEntries()[0];
            this.sendMetrics('fid', { 
                value: firstInput.processingStart - firstInput.startTime 
            });
        }).observe({ entryTypes: ['first-input'] });
        
        // Cumulative Layout Shift
        let clsValue = 0;
        new PerformanceObserver((list) => {
            for (const entry of list.getEntries()) {
                if (!entry.hadRecentInput) {
                    clsValue += entry.value;
                }
            }
            this.sendMetrics('cls', { value: clsValue });
        }).observe({ entryTypes: ['layout-shift'] });
    }
    
    observeUserMetrics() {
        // Time to Interactive estimation
        let timeToInteractive = 0;
        
        // Long tasks that block the main thread
        new PerformanceObserver((list) => {
            list.getEntries().forEach((entry) => {
                this.sendMetrics('long-task', {
                    duration: entry.duration,
                    startTime: entry.startTime
                });
            });
        }).observe({ entryTypes: ['longtask'] });
        
        // User interactions
        ['click', 'scroll', 'keypress'].forEach(eventType => {
            document.addEventListener(eventType, (event) => {
                const startTime = performance.now();
                
                requestAnimationFrame(() => {
                    const duration = performance.now() - startTime;
                    this.sendMetrics('interaction', {
                        type: eventType,
                        duration,
                        target: event.target.tagName
                    });
                });
            }, { passive: true });
        });
    }
    
    getResourceType(url) {
        if (url.match(/\.(css)$/)) return 'css';
        if (url.match(/\.(js)$/)) return 'javascript';
        if (url.match(/\.(png|jpg|jpeg|gif|webp|svg)$/)) return 'image';
        if (url.match(/\.(woff|woff2|ttf|eot)$/)) return 'font';
        return 'other';
    }
    
    sendMetrics(type, data) {
        // Send to analytics service
        if ('sendBeacon' in navigator) {
            navigator.sendBeacon('/api/metrics', JSON.stringify({
                type,
                data,
                timestamp: Date.now(),
                url: window.location.href
            }));
        }
    }
}

// Initialize performance monitoring
const performanceMonitor = new PerformanceMonitor();
```

---

## 📚 Summary

### Key Performance Areas:
- **Frontend**: Critical rendering path, lazy loading, code splitting
- **Backend**: Clustering, caching, database optimization
- **Database**: Proper indexing, query optimization, pagination
- **Caching**: Multiple layers (memory, Redis, HTTP)
- **Images**: Responsive images, modern formats, lazy loading
- **Monitoring**: Core Web Vitals, performance metrics

### Performance Checklist:
- [ ] Enable compression (gzip/brotli)
- [ ] Optimize images (WebP, AVIF, responsive)
- [ ] Implement caching strategies
- [ ] Use CDN for static assets
- [ ] Minify and bundle assets
- [ ] Implement lazy loading
- [ ] Optimize database queries
- [ ] Monitor Core Web Vitals
- [ ] Set performance budgets

### Next Steps:
1. Implement performance monitoring
2. Conduct regular performance audits
3. Learn advanced optimization techniques
4. Study platform-specific optimizations
5. Practice with real-world performance issues
