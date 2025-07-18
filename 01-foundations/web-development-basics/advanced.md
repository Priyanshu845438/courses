
# 🚀 Web Development Basics - Advanced

## 🌐 Advanced Web Protocols and Technologies

### HTTP/2 and HTTP/3

#### HTTP/2 Features
```javascript
// HTTP/2 Server Push example (Node.js)
const http2 = require('http2');
const fs = require('fs');

const server = http2.createSecureServer({
  key: fs.readFileSync('key.pem'),
  cert: fs.readFileSync('cert.pem')
});

server.on('stream', (stream, headers) => {
  if (headers[':path'] === '/') {
    // Push resources before sending HTML
    stream.pushStream({ ':path': '/style.css' }, (err, pushStream) => {
      if (!err) {
        pushStream.respondWithFile('style.css');
      }
    });
    
    stream.pushStream({ ':path': '/script.js' }, (err, pushStream) => {
      if (!err) {
        pushStream.respondWithFile('script.js');
      }
    });
    
    // Send main HTML response
    stream.respondWithFile('index.html');
  }
});
```

#### HTTP/3 (QUIC Protocol)
```javascript
// HTTP/3 connection example
const { createQuicSocket } = require('quic');

const socket = createQuicSocket({
  endpoint: { port: 1234 },
  client: {
    requestOCSP: true,
    sessionTicket: true
  }
});

socket.connect({
  address: 'example.com',
  port: 443,
  servername: 'example.com'
});
```

### WebSocket Protocol

#### Advanced WebSocket Implementation
```javascript
// WebSocket server with authentication and rooms
const WebSocket = require('ws');
const jwt = require('jsonwebtoken');

class WebSocketServer {
  constructor() {
    this.wss = new WebSocket.Server({ port: 8080 });
    this.clients = new Map();
    this.rooms = new Map();
    this.setupServer();
  }

  setupServer() {
    this.wss.on('connection', (ws, req) => {
      this.handleConnection(ws, req);
    });
  }

  handleConnection(ws, req) {
    // Extract token from query parameters
    const url = new URL(req.url, `http://${req.headers.host}`);
    const token = url.searchParams.get('token');
    
    try {
      const decoded = jwt.verify(token, process.env.JWT_SECRET);
      ws.userId = decoded.userId;
      ws.username = decoded.username;
      
      this.clients.set(ws.userId, ws);
      
      ws.on('message', (message) => {
        this.handleMessage(ws, JSON.parse(message));
      });
      
      ws.on('close', () => {
        this.handleDisconnect(ws);
      });
      
      // Send welcome message
      ws.send(JSON.stringify({
        type: 'welcome',
        message: 'Connected successfully'
      }));
      
    } catch (error) {
      ws.close(1008, 'Invalid token');
    }
  }

  handleMessage(ws, message) {
    switch (message.type) {
      case 'join_room':
        this.joinRoom(ws, message.room);
        break;
      case 'leave_room':
        this.leaveRoom(ws, message.room);
        break;
      case 'room_message':
        this.broadcastToRoom(message.room, message.data, ws.userId);
        break;
      case 'private_message':
        this.sendPrivateMessage(ws, message.targetUserId, message.data);
        break;
    }
  }

  joinRoom(ws, roomId) {
    if (!this.rooms.has(roomId)) {
      this.rooms.set(roomId, new Set());
    }
    this.rooms.get(roomId).add(ws.userId);
    ws.currentRoom = roomId;
    
    // Notify room members
    this.broadcastToRoom(roomId, {
      type: 'user_joined',
      username: ws.username
    }, ws.userId);
  }

  broadcastToRoom(roomId, data, excludeUserId = null) {
    const room = this.rooms.get(roomId);
    if (room) {
      room.forEach(userId => {
        if (userId !== excludeUserId) {
          const client = this.clients.get(userId);
          if (client && client.readyState === WebSocket.OPEN) {
            client.send(JSON.stringify(data));
          }
        }
      });
    }
  }
}

// Start server
const wsServer = new WebSocketServer();
```

### Advanced HTTP Security

#### Security Headers Implementation
```javascript
// Express.js security middleware
const helmet = require('helmet');
const rateLimit = require('express-rate-limit');

const securityMiddleware = (app) => {
  // Helmet for security headers
  app.use(helmet({
    contentSecurityPolicy: {
      directives: {
        defaultSrc: ["'self'"],
        styleSrc: ["'self'", "'unsafe-inline'", "https://fonts.googleapis.com"],
        fontSrc: ["'self'", "https://fonts.gstatic.com"],
        scriptSrc: ["'self'", "'unsafe-inline'"],
        imgSrc: ["'self'", "data:", "https:"],
        connectSrc: ["'self'", "wss:", "https:"],
        frameAncestors: ["'none'"],
        objectSrc: ["'none'"],
        upgradeInsecureRequests: [],
      },
    },
    hsts: {
      maxAge: 31536000,
      includeSubDomains: true,
      preload: true
    }
  }));

  // Rate limiting
  const limiter = rateLimit({
    windowMs: 15 * 60 * 1000, // 15 minutes
    max: 100, // limit each IP to 100 requests per windowMs
    message: 'Too many requests from this IP',
    standardHeaders: true,
    legacyHeaders: false,
  });
  
  app.use('/api/', limiter);

  // Additional security headers
  app.use((req, res, next) => {
    res.setHeader('X-Content-Type-Options', 'nosniff');
    res.setHeader('X-Frame-Options', 'DENY');
    res.setHeader('X-XSS-Protection', '1; mode=block');
    res.setHeader('Referrer-Policy', 'strict-origin-when-cross-origin');
    res.setHeader('Permissions-Policy', 'geolocation=(), microphone=(), camera=()');
    next();
  });
};
```

### Content Delivery Networks (CDN)

#### CDN Implementation Strategy
```javascript
// CDN resource optimization
class CDNManager {
  constructor() {
    this.cdnDomains = [
      'https://cdn1.example.com',
      'https://cdn2.example.com',
      'https://cdn3.example.com'
    ];
    this.resourceMap = new Map();
  }

  getOptimizedUrl(resource, type) {
    const hash = this.generateHash(resource);
    const cdnIndex = hash % this.cdnDomains.length;
    const cdnDomain = this.cdnDomains[cdnIndex];
    
    const optimizedResource = this.getOptimizedResource(resource, type);
    return `${cdnDomain}${optimizedResource}`;
  }

  getOptimizedResource(resource, type) {
    switch (type) {
      case 'image':
        return this.optimizeImage(resource);
      case 'css':
        return this.optimizeCSS(resource);
      case 'js':
        return this.optimizeJS(resource);
      default:
        return resource;
    }
  }

  optimizeImage(resource) {
    // Add format and quality parameters
    const params = new URLSearchParams({
      format: 'webp',
      quality: '80',
      width: '1200'
    });
    return `${resource}?${params.toString()}`;
  }

  optimizeCSS(resource) {
    // Add minification and cache parameters
    const params = new URLSearchParams({
      minify: 'true',
      cache: '31536000'
    });
    return `${resource}?${params.toString()}`;
  }

  generateHash(str) {
    let hash = 0;
    for (let i = 0; i < str.length; i++) {
      const char = str.charCodeAt(i);
      hash = ((hash << 5) - hash) + char;
      hash = hash & hash; // Convert to 32-bit integer
    }
    return Math.abs(hash);
  }
}

// Usage
const cdnManager = new CDNManager();
const imageUrl = cdnManager.getOptimizedUrl('/images/hero.jpg', 'image');
const cssUrl = cdnManager.getOptimizedUrl('/styles/main.css', 'css');
```

### Advanced Caching Strategies

#### Multi-layer Caching System
```javascript
// Redis-based caching with fallback
const redis = require('redis');
const NodeCache = require('node-cache');

class CacheManager {
  constructor() {
    this.redisClient = redis.createClient();
    this.nodeCache = new NodeCache({ stdTTL: 600 }); // 10 minutes
    this.setupRedis();
  }

  async setupRedis() {
    this.redisClient.on('error', (err) => {
      console.error('Redis error:', err);
    });
    
    await this.redisClient.connect();
  }

  async get(key) {
    try {
      // Try Redis first
      const redisValue = await this.redisClient.get(key);
      if (redisValue) {
        return JSON.parse(redisValue);
      }
      
      // Fallback to node-cache
      const nodeValue = this.nodeCache.get(key);
      if (nodeValue) {
        // Store in Redis for next time
        await this.redisClient.setEx(key, 3600, JSON.stringify(nodeValue));
        return nodeValue;
      }
      
      return null;
    } catch (error) {
      console.error('Cache get error:', error);
      return this.nodeCache.get(key) || null;
    }
  }

  async set(key, value, ttl = 3600) {
    try {
      const stringValue = JSON.stringify(value);
      
      // Set in Redis
      await this.redisClient.setEx(key, ttl, stringValue);
      
      // Set in node-cache as backup
      this.nodeCache.set(key, value, Math.min(ttl, 600));
      
      return true;
    } catch (error) {
      console.error('Cache set error:', error);
      // Fallback to node-cache only
      this.nodeCache.set(key, value, Math.min(ttl, 600));
      return false;
    }
  }

  async invalidate(pattern) {
    try {
      // Invalidate Redis keys matching pattern
      const keys = await this.redisClient.keys(pattern);
      if (keys.length > 0) {
        await this.redisClient.del(keys);
      }
      
      // Invalidate node-cache keys
      const nodeKeys = this.nodeCache.keys();
      nodeKeys.forEach(key => {
        if (key.includes(pattern.replace('*', ''))) {
          this.nodeCache.del(key);
        }
      });
      
      return true;
    } catch (error) {
      console.error('Cache invalidation error:', error);
      return false;
    }
  }
}

// Express middleware for caching
const cacheMiddleware = (cacheManager) => {
  return async (req, res, next) => {
    const cacheKey = `cache:${req.method}:${req.originalUrl}`;
    
    try {
      const cached = await cacheManager.get(cacheKey);
      if (cached) {
        res.set('X-Cache', 'HIT');
        return res.json(cached);
      }
      
      // Store original json method
      const originalJson = res.json;
      
      // Override json method to cache response
      res.json = function(data) {
        cacheManager.set(cacheKey, data, 300); // 5 minutes
        res.set('X-Cache', 'MISS');
        return originalJson.call(this, data);
      };
      
      next();
    } catch (error) {
      console.error('Cache middleware error:', error);
      next();
    }
  };
};
```

### Web Performance Optimization

#### Critical Resource Prioritization
```html
<!DOCTYPE html>
<html>
<head>
  <meta charset="UTF-8">
  <meta name="viewport" content="width=device-width, initial-scale=1.0">
  
  <!-- Preload critical resources -->
  <link rel="preload" href="/fonts/main.woff2" as="font" type="font/woff2" crossorigin>
  <link rel="preload" href="/css/critical.css" as="style">
  <link rel="preload" href="/js/main.js" as="script">
  
  <!-- DNS prefetch for external resources -->
  <link rel="dns-prefetch" href="//fonts.googleapis.com">
  <link rel="dns-prefetch" href="//api.example.com">
  
  <!-- Preconnect to critical origins -->
  <link rel="preconnect" href="https://fonts.gstatic.com" crossorigin>
  
  <!-- Critical CSS inline -->
  <style>
    /* Critical above-the-fold styles */
    body { font-family: -apple-system, BlinkMacSystemFont, 'Segoe UI', Roboto, sans-serif; }
    .header { height: 80px; background: #fff; }
    .hero { height: 50vh; background: linear-gradient(135deg, #667eea 0%, #764ba2 100%); }
  </style>
  
  <!-- Non-critical CSS with media attribute -->
  <link rel="stylesheet" href="/css/main.css" media="print" onload="this.media='all'">
  
  <!-- Fallback for non-JS users -->
  <noscript><link rel="stylesheet" href="/css/main.css"></noscript>
</head>
<body>
  <!-- Critical above-the-fold content -->
  <header class="header">
    <nav>...</nav>
  </header>
  
  <main>
    <section class="hero">
      <h1>Welcome</h1>
    </section>
  </main>
  
  <!-- Non-critical JavaScript loaded asynchronously -->
  <script async src="/js/analytics.js"></script>
  <script defer src="/js/main.js"></script>
</body>
</html>
```

### Advanced Browser APIs

#### Service Worker with Advanced Caching
```javascript
// sw.js - Advanced Service Worker
const CACHE_NAME = 'app-v1';
const RUNTIME_CACHE = 'runtime-cache';
const PRECACHE_URLS = [
  '/',
  '/css/main.css',
  '/js/main.js',
  '/manifest.json'
];

// Install event - precache resources
self.addEventListener('install', (event) => {
  event.waitUntil(
    caches.open(CACHE_NAME)
      .then((cache) => cache.addAll(PRECACHE_URLS))
      .then(() => self.skipWaiting())
  );
});

// Activate event - cleanup old caches
self.addEventListener('activate', (event) => {
  event.waitUntil(
    caches.keys()
      .then((cacheNames) => {
        return Promise.all(
          cacheNames.map((cacheName) => {
            if (cacheName !== CACHE_NAME && cacheName !== RUNTIME_CACHE) {
              return caches.delete(cacheName);
            }
          })
        );
      })
      .then(() => self.clients.claim())
  );
});

// Fetch event - advanced caching strategies
self.addEventListener('fetch', (event) => {
  const { request } = event;
  const url = new URL(request.url);
  
  // Different strategies for different resource types
  if (request.destination === 'image') {
    event.respondWith(cacheFirst(request));
  } else if (url.pathname.startsWith('/api/')) {
    event.respondWith(networkFirst(request));
  } else if (request.destination === 'document') {
    event.respondWith(staleWhileRevalidate(request));
  } else {
    event.respondWith(cacheFirst(request));
  }
});

// Cache-first strategy
async function cacheFirst(request) {
  const cachedResponse = await caches.match(request);
  if (cachedResponse) {
    return cachedResponse;
  }
  
  try {
    const networkResponse = await fetch(request);
    if (networkResponse.ok) {
      const cache = await caches.open(RUNTIME_CACHE);
      cache.put(request, networkResponse.clone());
    }
    return networkResponse;
  } catch (error) {
    // Return offline fallback
    return caches.match('/offline.html');
  }
}

// Network-first strategy
async function networkFirst(request) {
  try {
    const networkResponse = await fetch(request);
    if (networkResponse.ok) {
      const cache = await caches.open(RUNTIME_CACHE);
      cache.put(request, networkResponse.clone());
    }
    return networkResponse;
  } catch (error) {
    const cachedResponse = await caches.match(request);
    return cachedResponse || new Response('Offline', { status: 503 });
  }
}

// Stale-while-revalidate strategy
async function staleWhileRevalidate(request) {
  const cachedResponse = await caches.match(request);
  
  const fetchPromise = fetch(request)
    .then((networkResponse) => {
      if (networkResponse.ok) {
        const cache = caches.open(RUNTIME_CACHE);
        cache.then((c) => c.put(request, networkResponse.clone()));
      }
      return networkResponse;
    });
  
  return cachedResponse || fetchPromise;
}
```

This comprehensive advanced guide covers modern web protocols, security implementations, performance optimization, and advanced browser APIs that are essential for building production-ready web applications.

## 📚 Summary

### Key Advanced Concepts:
- **HTTP/2 & HTTP/3**: Modern protocol features and optimizations
- **WebSocket**: Real-time communication patterns
- **Security**: Advanced headers, rate limiting, and content security policies
- **CDN**: Content delivery and optimization strategies
- **Caching**: Multi-layer caching systems and strategies
- **Performance**: Critical resource prioritization and loading optimization
- **Service Workers**: Advanced caching and offline functionality

### Best Practices:
1. **Implement HTTP/2 server push** for critical resources
2. **Use WebSocket for real-time features** with proper authentication
3. **Apply comprehensive security headers** and rate limiting
4. **Optimize CDN usage** with proper resource distribution
5. **Implement multi-layer caching** with fallback strategies
6. **Prioritize critical resources** for better performance
7. **Use Service Workers** for offline functionality and caching

---

**Next:** Proceed to Git Version Control for advanced development workflows
