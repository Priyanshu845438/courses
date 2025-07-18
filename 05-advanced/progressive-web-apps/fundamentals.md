
# 📱 Progressive Web Apps (PWA) Basics

## Table of Contents
- [What are Progressive Web Apps?](#what-are-progressive-web-apps)
- [PWA Core Features](#pwa-core-features)
- [Service Workers](#service-workers)
- [Web App Manifest](#web-app-manifest)
- [Caching Strategies](#caching-strategies)
- [Push Notifications](#push-notifications)
- [Installation and App-like Experience](#installation-and-app-like-experience)
- [PWA Development Tools](#pwa-development-tools)

---

## 🌟 What are Progressive Web Apps?

Progressive Web Apps (PWAs) are web applications that use modern web capabilities to deliver app-like experiences to users. They combine the best of web and mobile apps.

### Key Characteristics:
- **Progressive**: Work for every user, regardless of browser choice
- **Responsive**: Fit any form factor (desktop, mobile, tablet)
- **Connectivity Independent**: Work offline or with poor connectivity
- **App-like**: Feel like native apps with app-style interactions
- **Fresh**: Always up-to-date thanks to service workers
- **Safe**: Served via HTTPS to prevent tampering
- **Discoverable**: Identifiable as applications via manifest files
- **Re-engageable**: Support push notifications
- **Installable**: Can be installed on device home screens
- **Linkable**: Easily shared via URLs

---

## 🔧 PWA Core Features

### Service Workers
```javascript
// sw.js - Service Worker file
const CACHE_NAME = 'my-pwa-v1';
const urlsToCache = [
    '/',
    '/styles/main.css',
    '/scripts/main.js',
    '/images/logo.png',
    '/offline.html'
];

// Install event
self.addEventListener('install', (event) => {
    event.waitUntil(
        caches.open(CACHE_NAME)
            .then((cache) => {
                console.log('Opened cache');
                return cache.addAll(urlsToCache);
            })
    );
});

// Fetch event
self.addEventListener('fetch', (event) => {
    event.respondWith(
        caches.match(event.request)
            .then((response) => {
                // Return cached version or fetch from network
                return response || fetch(event.request)
                    .catch(() => {
                        // Return offline page for navigation requests
                        if (event.request.mode === 'navigate') {
                            return caches.match('/offline.html');
                        }
                    });
            })
    );
});

// Activate event
self.addEventListener('activate', (event) => {
    event.waitUntil(
        caches.keys().then((cacheNames) => {
            return Promise.all(
                cacheNames.map((cacheName) => {
                    if (cacheName !== CACHE_NAME) {
                        console.log('Deleting old cache:', cacheName);
                        return caches.delete(cacheName);
                    }
                })
            );
        })
    );
});
```

### Registering Service Worker
```javascript
// main.js
if ('serviceWorker' in navigator) {
    window.addEventListener('load', () => {
        navigator.serviceWorker.register('/sw.js')
            .then((registration) => {
                console.log('SW registered: ', registration);
                
                // Check for updates
                registration.addEventListener('updatefound', () => {
                    const newWorker = registration.installing;
                    newWorker.addEventListener('statechange', () => {
                        if (newWorker.state === 'installed' && navigator.serviceWorker.controller) {
                            // New content available, show update notification
                            showUpdateNotification();
                        }
                    });
                });
            })
            .catch((registrationError) => {
                console.log('SW registration failed: ', registrationError);
            });
    });
}

function showUpdateNotification() {
    const notification = document.createElement('div');
    notification.innerHTML = `
        <div style="position: fixed; top: 0; left: 0; right: 0; background: #333; color: white; padding: 10px; z-index: 1000;">
            New content available! 
            <button onclick="window.location.reload()">Refresh</button>
        </div>
    `;
    document.body.appendChild(notification);
}
```

---

## 📋 Web App Manifest

### manifest.json
```json
{
    "name": "My Progressive Web App",
    "short_name": "MyPWA",
    "description": "A sample PWA demonstrating core features",
    "start_url": "/",
    "display": "standalone",
    "orientation": "portrait-primary",
    "theme_color": "#2196F3",
    "background_color": "#ffffff",
    "scope": "/",
    "lang": "en",
    "categories": ["productivity", "utilities"],
    "icons": [
        {
            "src": "/images/icon-72x72.png",
            "sizes": "72x72",
            "type": "image/png",
            "purpose": "maskable any"
        },
        {
            "src": "/images/icon-96x96.png",
            "sizes": "96x96",
            "type": "image/png",
            "purpose": "maskable any"
        },
        {
            "src": "/images/icon-128x128.png",
            "sizes": "128x128",
            "type": "image/png",
            "purpose": "maskable any"
        },
        {
            "src": "/images/icon-144x144.png",
            "sizes": "144x144",
            "type": "image/png",
            "purpose": "maskable any"
        },
        {
            "src": "/images/icon-152x152.png",
            "sizes": "152x152",
            "type": "image/png",
            "purpose": "maskable any"
        },
        {
            "src": "/images/icon-192x192.png",
            "sizes": "192x192",
            "type": "image/png",
            "purpose": "maskable any"
        },
        {
            "src": "/images/icon-384x384.png",
            "sizes": "384x384",
            "type": "image/png",
            "purpose": "maskable any"
        },
        {
            "src": "/images/icon-512x512.png",
            "sizes": "512x512",
            "type": "image/png",
            "purpose": "maskable any"
        }
    ],
    "shortcuts": [
        {
            "name": "New Task",
            "short_name": "New",
            "description": "Create a new task",
            "url": "/new-task",
            "icons": [
                {
                    "src": "/images/new-task-icon.png",
                    "sizes": "96x96"
                }
            ]
        }
    ],
    "screenshots": [
        {
            "src": "/images/screenshot1.png",
            "sizes": "1280x720",
            "type": "image/png",
            "platform": "wide"
        },
        {
            "src": "/images/screenshot2.png",
            "sizes": "750x1334",
            "type": "image/png",
            "platform": "narrow"
        }
    ]
}
```

### HTML Integration
```html
<!DOCTYPE html>
<html lang="en">
<head>
    <meta charset="UTF-8">
    <meta name="viewport" content="width=device-width, initial-scale=1.0">
    <title>My PWA</title>
    
    <!-- PWA Manifest -->
    <link rel="manifest" href="/manifest.json">
    
    <!-- Theme color -->
    <meta name="theme-color" content="#2196F3">
    
    <!-- Apple-specific meta tags -->
    <meta name="apple-mobile-web-app-capable" content="yes">
    <meta name="apple-mobile-web-app-status-bar-style" content="default">
    <meta name="apple-mobile-web-app-title" content="MyPWA">
    <link rel="apple-touch-icon" href="/images/icon-152x152.png">
    
    <!-- Microsoft-specific meta tags -->
    <meta name="msapplication-TileImage" content="/images/icon-144x144.png">
    <meta name="msapplication-TileColor" content="#2196F3">
</head>
<body>
    <div id="app">
        <header>
            <h1>My Progressive Web App</h1>
            <button id="install-btn" style="display: none;">Install App</button>
        </header>
        
        <main>
            <p>Welcome to our PWA!</p>
            <div id="network-status"></div>
        </main>
    </div>
    
    <script src="/scripts/main.js"></script>
</body>
</html>
```

---

## 💾 Caching Strategies

### Cache First Strategy
```javascript
// Good for static assets that rarely change
self.addEventListener('fetch', (event) => {
    if (event.request.destination === 'image') {
        event.respondWith(
            caches.match(event.request).then((cachedResponse) => {
                return cachedResponse || fetch(event.request).then((response) => {
                    const responseClone = response.clone();
                    caches.open(CACHE_NAME).then((cache) => {
                        cache.put(event.request, responseClone);
                    });
                    return response;
                });
            })
        );
    }
});
```

### Network First Strategy
```javascript
// Good for dynamic content that changes frequently
self.addEventListener('fetch', (event) => {
    if (event.request.url.includes('/api/')) {
        event.respondWith(
            fetch(event.request)
                .then((response) => {
                    const responseClone = response.clone();
                    caches.open(CACHE_NAME).then((cache) => {
                        cache.put(event.request, responseClone);
                    });
                    return response;
                })
                .catch(() => {
                    return caches.match(event.request);
                })
        );
    }
});
```

### Stale While Revalidate
```javascript
// Good for content that should be fresh but can tolerate being stale
self.addEventListener('fetch', (event) => {
    event.respondWith(
        caches.match(event.request).then((cachedResponse) => {
            const fetchPromise = fetch(event.request).then((networkResponse) => {
                caches.open(CACHE_NAME).then((cache) => {
                    cache.put(event.request, networkResponse.clone());
                });
                return networkResponse;
            });
            
            return cachedResponse || fetchPromise;
        })
    );
});
```

---

## 🔔 Push Notifications

### Service Worker Push Event
```javascript
// sw.js
self.addEventListener('push', (event) => {
    const options = {
        body: event.data ? event.data.text() : 'Default notification body',
        icon: '/images/icon-192x192.png',
        badge: '/images/badge-72x72.png',
        vibrate: [200, 100, 200],
        data: {
            dateOfArrival: Date.now(),
            primaryKey: 1
        },
        actions: [
            {
                action: 'explore',
                title: 'Explore',
                icon: '/images/checkmark.png'
            },
            {
                action: 'close',
                title: 'Close',
                icon: '/images/xmark.png'
            }
        ]
    };
    
    event.waitUntil(
        self.registration.showNotification('PWA Notification', options)
    );
});

// Handle notification clicks
self.addEventListener('notificationclick', (event) => {
    event.notification.close();
    
    if (event.action === 'explore') {
        event.waitUntil(
            clients.openWindow('/')
        );
    } else if (event.action === 'close') {
        // Notification closed
    } else {
        // Default action - open app
        event.waitUntil(
            clients.openWindow('/')
        );
    }
});
```

### Subscribing to Push Notifications
```javascript
// main.js
async function subscribeToPush() {
    try {
        // Check if push notifications are supported
        if (!('PushManager' in window)) {
            console.log('Push messaging is not supported');
            return;
        }
        
        // Request permission
        const permission = await Notification.requestPermission();
        if (permission !== 'granted') {
            console.log('Permission not granted for notifications');
            return;
        }
        
        // Get service worker registration
        const registration = await navigator.serviceWorker.getRegistration();
        
        // Subscribe to push service
        const subscription = await registration.pushManager.subscribe({
            userVisibleOnly: true,
            applicationServerKey: urlBase64ToUint8Array('YOUR_VAPID_PUBLIC_KEY')
        });
        
        // Send subscription to server
        await fetch('/api/subscribe', {
            method: 'POST',
            headers: {
                'Content-Type': 'application/json'
            },
            body: JSON.stringify(subscription)
        });
        
        console.log('Push subscription successful');
    } catch (error) {
        console.error('Failed to subscribe to push notifications:', error);
    }
}

function urlBase64ToUint8Array(base64String) {
    const padding = '='.repeat((4 - base64String.length % 4) % 4);
    const base64 = (base64String + padding)
        .replace(/-/g, '+')
        .replace(/_/g, '/');
    
    const rawData = window.atob(base64);
    const outputArray = new Uint8Array(rawData.length);
    
    for (let i = 0; i < rawData.length; ++i) {
        outputArray[i] = rawData.charCodeAt(i);
    }
    return outputArray;
}
```

---

## 📲 Installation and App-like Experience

### Install Prompt
```javascript
// main.js
let deferredPrompt;
const installBtn = document.getElementById('install-btn');

window.addEventListener('beforeinstallprompt', (e) => {
    console.log('beforeinstallprompt fired');
    
    // Prevent Chrome 67 and earlier from automatically showing the prompt
    e.preventDefault();
    
    // Stash the event so it can be triggered later
    deferredPrompt = e;
    
    // Show install button
    installBtn.style.display = 'block';
    
    installBtn.addEventListener('click', () => {
        // Hide the install button
        installBtn.style.display = 'none';
        
        // Show the prompt
        deferredPrompt.prompt();
        
        // Wait for the user to respond to the prompt
        deferredPrompt.userChoice.then((choiceResult) => {
            if (choiceResult.outcome === 'accepted') {
                console.log('User accepted the install prompt');
            } else {
                console.log('User dismissed the install prompt');
            }
            deferredPrompt = null;
        });
    });
});

// Track installation
window.addEventListener('appinstalled', (evt) => {
    console.log('PWA was installed');
    // Hide install button if visible
    installBtn.style.display = 'none';
});
```

### Detecting Display Mode
```javascript
// Check if app is running in standalone mode
function isStandalone() {
    return window.matchMedia('(display-mode: standalone)').matches ||
           window.navigator.standalone === true;
}

if (isStandalone()) {
    console.log('App is running in standalone mode');
    // Hide browser-specific UI elements
    document.body.classList.add('standalone');
} else {
    console.log('App is running in browser');
    // Show install prompt
}

// Listen for display mode changes
window.matchMedia('(display-mode: standalone)').addEventListener('change', (e) => {
    if (e.matches) {
        console.log('Switched to standalone mode');
    } else {
        console.log('Switched to browser mode');
    }
});
```

---

## 🌐 Offline Functionality

### Network Status Detection
```javascript
// main.js
function updateNetworkStatus() {
    const networkStatus = document.getElementById('network-status');
    
    if (navigator.onLine) {
        networkStatus.innerHTML = '🟢 Online';
        networkStatus.className = 'online';
    } else {
        networkStatus.innerHTML = '🔴 Offline';
        networkStatus.className = 'offline';
    }
}

// Initial status
updateNetworkStatus();

// Listen for network changes
window.addEventListener('online', updateNetworkStatus);
window.addEventListener('offline', updateNetworkStatus);
```

### Background Sync
```javascript
// sw.js
self.addEventListener('sync', (event) => {
    if (event.tag === 'background-sync') {
        event.waitUntil(doBackgroundSync());
    }
});

async function doBackgroundSync() {
    try {
        // Get pending data from IndexedDB
        const pendingData = await getPendingData();
        
        // Send to server
        for (const data of pendingData) {
            await fetch('/api/sync', {
                method: 'POST',
                body: JSON.stringify(data)
            });
            
            // Remove from pending queue
            await removePendingData(data.id);
        }
    } catch (error) {
        console.error('Background sync failed:', error);
    }
}

// main.js - Register background sync
navigator.serviceWorker.ready.then((registration) => {
    return registration.sync.register('background-sync');
});
```

---

## 🛠️ PWA Development Tools

### Lighthouse PWA Audit
```javascript
// Performance monitoring
class PWAMetrics {
    static measureTTI() {
        return new Promise((resolve) => {
            if ('PerformanceLongTaskTiming' in window) {
                new PerformanceObserver((list) => {
                    for (const entry of list.getEntries()) {
                        console.log('Long task detected:', entry);
                    }
                }).observe({ entryTypes: ['longtask'] });
            }
            
            // Simplified TTI measurement
            window.addEventListener('load', () => {
                setTimeout(() => {
                    resolve(performance.now());
                }, 5000);
            });
        });
    }
    
    static measureFCP() {
        return new Promise((resolve) => {
            new PerformanceObserver((list) => {
                for (const entry of list.getEntries()) {
                    if (entry.name === 'first-contentful-paint') {
                        resolve(entry.startTime);
                    }
                }
            }).observe({ entryTypes: ['paint'] });
        });
    }
}

// Usage
PWAMetrics.measureFCP().then((fcp) => {
    console.log('First Contentful Paint:', fcp);
});

PWAMetrics.measureTTI().then((tti) => {
    console.log('Time to Interactive:', tti);
});
```

### Service Worker Update Strategies
```javascript
// sw.js
const version = 'v1.2.0';
const CACHE_NAME = `pwa-cache-${version}`;

// Skip waiting for faster updates
self.addEventListener('message', (event) => {
    if (event.data && event.data.type === 'SKIP_WAITING') {
        self.skipWaiting();
    }
});

// main.js
navigator.serviceWorker.addEventListener('controllerchange', () => {
    window.location.reload();
});

function updateServiceWorker() {
    navigator.serviceWorker.getRegistration().then((registration) => {
        if (registration.waiting) {
            registration.waiting.postMessage({ type: 'SKIP_WAITING' });
        }
    });
}
```

---

## 📱 Complete PWA Example

### HTML Structure
```html
<!DOCTYPE html>
<html lang="en">
<head>
    <meta charset="UTF-8">
    <meta name="viewport" content="width=device-width, initial-scale=1.0">
    <title>Task Manager PWA</title>
    <link rel="manifest" href="/manifest.json">
    <meta name="theme-color" content="#2196F3">
    <style>
        * {
            margin: 0;
            padding: 0;
            box-sizing: border-box;
        }
        
        body {
            font-family: 'Segoe UI', Tahoma, Geneva, Verdana, sans-serif;
            background: linear-gradient(135deg, #667eea 0%, #764ba2 100%);
            min-height: 100vh;
        }
        
        .container {
            max-width: 600px;
            margin: 0 auto;
            padding: 20px;
        }
        
        .app-header {
            background: white;
            padding: 20px;
            border-radius: 10px;
            margin-bottom: 20px;
            box-shadow: 0 4px 6px rgba(0, 0, 0, 0.1);
        }
        
        .status-bar {
            display: flex;
            justify-content: space-between;
            align-items: center;
            margin-bottom: 20px;
        }
        
        .network-status {
            padding: 5px 10px;
            border-radius: 15px;
            font-size: 12px;
            font-weight: bold;
        }
        
        .online {
            background: #4CAF50;
            color: white;
        }
        
        .offline {
            background: #f44336;
            color: white;
        }
        
        .install-btn {
            background: #2196F3;
            color: white;
            border: none;
            padding: 10px 20px;
            border-radius: 5px;
            cursor: pointer;
        }
        
        .task-form {
            background: white;
            padding: 20px;
            border-radius: 10px;
            margin-bottom: 20px;
            box-shadow: 0 4px 6px rgba(0, 0, 0, 0.1);
        }
        
        .task-input {
            width: 100%;
            padding: 15px;
            border: 2px solid #ddd;
            border-radius: 5px;
            margin-bottom: 10px;
            font-size: 16px;
        }
        
        .add-btn {
            width: 100%;
            background: #4CAF50;
            color: white;
            border: none;
            padding: 15px;
            border-radius: 5px;
            font-size: 16px;
            cursor: pointer;
        }
        
        .task-list {
            background: white;
            border-radius: 10px;
            box-shadow: 0 4px 6px rgba(0, 0, 0, 0.1);
            overflow: hidden;
        }
        
        .task-item {
            padding: 15px 20px;
            border-bottom: 1px solid #eee;
            display: flex;
            justify-content: space-between;
            align-items: center;
        }
        
        .task-item:last-child {
            border-bottom: none;
        }
        
        .delete-btn {
            background: #f44336;
            color: white;
            border: none;
            padding: 5px 10px;
            border-radius: 3px;
            cursor: pointer;
        }
    </style>
</head>
<body>
    <div class="container">
        <header class="app-header">
            <div class="status-bar">
                <h1>📝 Task Manager PWA</h1>
                <div id="network-status" class="network-status"></div>
            </div>
            <button id="install-btn" class="install-btn" style="display: none;">
                📲 Install App
            </button>
        </header>
        
        <div class="task-form">
            <input type="text" id="task-input" class="task-input" placeholder="Enter a new task...">
            <button id="add-btn" class="add-btn">Add Task</button>
        </div>
        
        <div id="task-list" class="task-list">
            <!-- Tasks will be added here -->
        </div>
    </div>
    
    <script>
        // PWA Task Manager
        class TaskManager {
            constructor() {
                this.tasks = this.loadTasks();
                this.bindEvents();
                this.render();
                this.updateNetworkStatus();
            }
            
            bindEvents() {
                document.getElementById('add-btn').addEventListener('click', () => {
                    this.addTask();
                });
                
                document.getElementById('task-input').addEventListener('keypress', (e) => {
                    if (e.key === 'Enter') {
                        this.addTask();
                    }
                });
                
                // Network status
                window.addEventListener('online', () => this.updateNetworkStatus());
                window.addEventListener('offline', () => this.updateNetworkStatus());
            }
            
            addTask() {
                const input = document.getElementById('task-input');
                const text = input.value.trim();
                
                if (text) {
                    const task = {
                        id: Date.now(),
                        text: text,
                        completed: false,
                        createdAt: new Date().toISOString()
                    };
                    
                    this.tasks.push(task);
                    this.saveTasks();
                    this.render();
                    input.value = '';
                }
            }
            
            deleteTask(id) {
                this.tasks = this.tasks.filter(task => task.id !== id);
                this.saveTasks();
                this.render();
            }
            
            render() {
                const taskList = document.getElementById('task-list');
                
                if (this.tasks.length === 0) {
                    taskList.innerHTML = '<div style="padding: 20px; text-align: center; color: #666;">No tasks yet. Add one above!</div>';
                    return;
                }
                
                taskList.innerHTML = this.tasks.map(task => `
                    <div class="task-item">
                        <span>${task.text}</span>
                        <button class="delete-btn" onclick="taskManager.deleteTask(${task.id})">
                            Delete
                        </button>
                    </div>
                `).join('');
            }
            
            saveTasks() {
                localStorage.setItem('pwa-tasks', JSON.stringify(this.tasks));
            }
            
            loadTasks() {
                const saved = localStorage.getItem('pwa-tasks');
                return saved ? JSON.parse(saved) : [];
            }
            
            updateNetworkStatus() {
                const statusEl = document.getElementById('network-status');
                if (navigator.onLine) {
                    statusEl.textContent = '🟢 Online';
                    statusEl.className = 'network-status online';
                } else {
                    statusEl.textContent = '🔴 Offline';
                    statusEl.className = 'network-status offline';
                }
            }
        }
        
        // PWA Installation
        let deferredPrompt;
        const installBtn = document.getElementById('install-btn');
        
        window.addEventListener('beforeinstallprompt', (e) => {
            e.preventDefault();
            deferredPrompt = e;
            installBtn.style.display = 'block';
            
            installBtn.addEventListener('click', () => {
                installBtn.style.display = 'none';
                deferredPrompt.prompt();
                
                deferredPrompt.userChoice.then((choiceResult) => {
                    if (choiceResult.outcome === 'accepted') {
                        console.log('User accepted the install prompt');
                    }
                    deferredPrompt = null;
                });
            });
        });
        
        // Initialize app
        const taskManager = new TaskManager();
        
        // Register service worker
        if ('serviceWorker' in navigator) {
            window.addEventListener('load', () => {
                navigator.serviceWorker.register('/sw.js')
                    .then((registration) => {
                        console.log('SW registered: ', registration);
                    })
                    .catch((registrationError) => {
                        console.log('SW registration failed: ', registrationError);
                    });
            });
        }
    </script>
</body>
</html>
```

This comprehensive PWA guide covers all the essential concepts and provides practical examples for building Progressive Web Apps. The example demonstrates a fully functional task manager PWA with offline capabilities, installation prompts, and responsive design.
