
# 📱 Intermediate Progressive Web Apps (PWA)

## Table of Contents
- [Advanced Service Worker Patterns](#advanced-service-worker-patterns)
- [Background Sync](#background-sync)
- [Push Notifications](#push-notifications)
- [App Shell Architecture](#app-shell-architecture)
- [Offline Data Management](#offline-data-management)
- [PWA Performance](#pwa-performance)
- [Web App Manifest](#web-app-manifest)
- [Installation and Engagement](#installation-and-engagement)
- [PWA Testing](#pwa-testing)

---

## ⚙️ Advanced Service Worker Patterns

### Cache-First with Network Fallback

```javascript
// sw-advanced-patterns.js

// Cache strategies for different resource types
const CACHE_STRATEGIES = {
    // Static assets - Cache First
    CACHE_FIRST: 'cache-first',
    // API data - Network First with Cache Fallback
    NETWORK_FIRST: 'network-first',
    // Images - Cache First with Network Fallback
    CACHE_NETWORK_FALLBACK: 'cache-network-fallback',
    // Analytics - Network Only
    NETWORK_ONLY: 'network-only',
    // App Shell - Cache Only
    CACHE_ONLY: 'cache-only'
};

class AdvancedCacheManager {
    constructor() {
        this.CACHE_VERSION = 'v1.2.0';
        this.STATIC_CACHE = `static-${this.CACHE_VERSION}`;
        this.DYNAMIC_CACHE = `dynamic-${this.CACHE_VERSION}`;
        this.API_CACHE = `api-${this.CACHE_VERSION}`;
        this.IMAGE_CACHE = `images-${this.CACHE_VERSION}`;
        
        this.MAX_CACHE_SIZE = {
            [this.DYNAMIC_CACHE]: 50,
            [this.API_CACHE]: 30,
            [this.IMAGE_CACHE]: 60
        };
        
        this.CACHE_EXPIRY = {
            [this.API_CACHE]: 5 * 60 * 1000, // 5 minutes
            [this.IMAGE_CACHE]: 7 * 24 * 60 * 60 * 1000, // 7 days
            [this.DYNAMIC_CACHE]: 24 * 60 * 60 * 1000 // 24 hours
        };
    }
    
    async handleRequest(event) {
        const { request } = event;
        const url = new URL(request.url);
        
        // Route requests based on patterns
        if (this.isStaticAsset(url)) {
            return this.cacheFirst(request, this.STATIC_CACHE);
        }
        
        if (this.isAPIRequest(url)) {
            return this.networkFirstWithExpiry(request, this.API_CACHE);
        }
        
        if (this.isImageRequest(url)) {
            return this.cacheNetworkFallback(request, this.IMAGE_CACHE);
        }
        
        if (this.isAnalytics(url)) {
            return this.networkOnly(request);
        }
        
        // Default: Network first for HTML pages
        return this.networkFirst(request, this.DYNAMIC_CACHE);
    }
    
    async cacheFirst(request, cacheName) {
        try {
            const cache = await caches.open(cacheName);
            const cachedResponse = await cache.match(request);
            
            if (cachedResponse) {
                // Update cache in background if needed
                this.updateCacheInBackground(request, cacheName);
                return cachedResponse;
            }
            
            const networkResponse = await fetch(request);
            await cache.put(request, networkResponse.clone());
            await this.trimCache(cacheName);
            
            return networkResponse;
        } catch (error) {
            console.error('Cache first failed:', error);
            return this.createErrorResponse('Content unavailable offline');
        }
    }
    
    async networkFirstWithExpiry(request, cacheName) {
        try {
            const cache = await caches.open(cacheName);
            
            // Try network first
            const networkResponse = await fetch(request);
            
            if (networkResponse.ok) {
                // Add timestamp to response headers
                const responseWithTimestamp = new Response(networkResponse.body, {
                    status: networkResponse.status,
                    statusText: networkResponse.statusText,
                    headers: {
                        ...Object.fromEntries(networkResponse.headers),
                        'sw-cached-at': Date.now().toString()
                    }
                });
                
                await cache.put(request, responseWithTimestamp.clone());
                await this.trimCache(cacheName);
                return responseWithTimestamp;
            }
        } catch (error) {
            console.log('Network failed, trying cache:', error.message);
        }
        
        // Fallback to cache
        const cache = await caches.open(cacheName);
        const cachedResponse = await cache.match(request);
        
        if (cachedResponse) {
            // Check if cached response is expired
            const cachedAt = cachedResponse.headers.get('sw-cached-at');
            const expiry = this.CACHE_EXPIRY[cacheName];
            
            if (cachedAt && expiry) {
                const age = Date.now() - parseInt(cachedAt);
                if (age > expiry) {
                    console.log('Cached response expired');
                    return this.createStaleResponse(cachedResponse);
                }
            }
            
            return cachedResponse;
        }
        
        return this.createErrorResponse('No cached data available');
    }
    
    async cacheNetworkFallback(request, cacheName) {
        const cache = await caches.open(cacheName);
        const cachedResponse = await cache.match(request);
        
        if (cachedResponse) {
            return cachedResponse;
        }
        
        try {
            const networkResponse = await fetch(request);
            
            if (networkResponse.ok) {
                await cache.put(request, networkResponse.clone());
                await this.trimCache(cacheName);
            }
            
            return networkResponse;
        } catch (error) {
            return this.createErrorResponse('Image unavailable offline');
        }
    }
    
    async networkFirst(request, cacheName) {
        try {
            const networkResponse = await fetch(request);
            
            if (networkResponse.ok) {
                const cache = await caches.open(cacheName);
                await cache.put(request, networkResponse.clone());
                await this.trimCache(cacheName);
            }
            
            return networkResponse;
        } catch (error) {
            const cache = await caches.open(cacheName);
            const cachedResponse = await cache.match(request);
            
            if (cachedResponse) {
                return cachedResponse;
            }
            
            return this.createOfflinePage();
        }
    }
    
    async networkOnly(request) {
        try {
            return await fetch(request);
        } catch (error) {
            // Fail silently for analytics
            return new Response('', { status: 200 });
        }
    }
    
    async updateCacheInBackground(request, cacheName) {
        try {
            const networkResponse = await fetch(request);
            
            if (networkResponse.ok) {
                const cache = await caches.open(cacheName);
                await cache.put(request, networkResponse);
            }
        } catch (error) {
            // Background update failed - not critical
            console.log('Background cache update failed:', error);
        }
    }
    
    async trimCache(cacheName) {
        const maxSize = this.MAX_CACHE_SIZE[cacheName];
        if (!maxSize) return;
        
        const cache = await caches.open(cacheName);
        const keys = await cache.keys();
        
        if (keys.length <= maxSize) return;
        
        // Remove oldest entries
        const keysToDelete = keys.slice(0, keys.length - maxSize);
        await Promise.all(keysToDelete.map(key => cache.delete(key)));
    }
    
    isStaticAsset(url) {
        return /\.(css|js|woff|woff2|ttf|svg|ico)$/.test(url.pathname);
    }
    
    isAPIRequest(url) {
        return url.pathname.startsWith('/api/');
    }
    
    isImageRequest(url) {
        return /\.(png|jpg|jpeg|gif|webp|avif)$/.test(url.pathname);
    }
    
    isAnalytics(url) {
        return url.hostname.includes('analytics') || 
               url.hostname.includes('gtag') ||
               url.pathname.includes('analytics');
    }
    
    createErrorResponse(message) {
        return new Response(JSON.stringify({ error: message }), {
            status: 503,
            headers: { 'Content-Type': 'application/json' }
        });
    }
    
    createStaleResponse(response) {
        // Add header to indicate stale data
        return new Response(response.body, {
            status: response.status,
            statusText: response.statusText,
            headers: {
                ...Object.fromEntries(response.headers),
                'x-cache-status': 'stale'
            }
        });
    }
    
    async createOfflinePage() {
        const cache = await caches.open(this.STATIC_CACHE);
        return cache.match('/offline.html') || 
               new Response('You are offline', { 
                   status: 503,
                   headers: { 'Content-Type': 'text/html' }
               });
    }
}

// Service Worker installation
const cacheManager = new AdvancedCacheManager();

self.addEventListener('fetch', event => {
    event.respondWith(cacheManager.handleRequest(event));
});
```

---

## 🔄 Background Sync

### Reliable Background Data Synchronization

```javascript
// background-sync.js

class BackgroundSyncManager {
    constructor() {
        this.SYNC_TAG = 'background-sync';
        this.RETRY_TAG = 'background-retry';
        this.PERIODIC_TAG = 'periodic-sync';
        
        this.pendingRequests = new Map();
        this.maxRetries = 3;
        this.retryDelays = [1000, 5000, 15000]; // Progressive delays
    }
    
    // Queue request for background sync
    async queueRequest(tag, data) {
        const timestamp = Date.now();
        const requestId = `${tag}-${timestamp}`;
        
        // Store in IndexedDB for persistence
        await this.storeRequest(requestId, {
            tag,
            data,
            timestamp,
            retries: 0,
            maxRetries: this.maxRetries
        });
        
        // Register for background sync
        if ('serviceWorker' in navigator && 'sync' in window.ServiceWorkerRegistration.prototype) {
            const registration = await navigator.serviceWorker.ready;
            await registration.sync.register(tag);
        }
        
        return requestId;
    }
    
    // Handle sync event in service worker
    async handleSync(event) {
        const { tag } = event;
        
        switch (tag) {
            case this.SYNC_TAG:
                await this.processPendingRequests();
                break;
            case this.RETRY_TAG:
                await this.retryFailedRequests();
                break;
            case this.PERIODIC_TAG:
                await this.performPeriodicSync();
                break;
            default:
                await this.processTaggedRequests(tag);
        }
    }
    
    async processPendingRequests() {
        const requests = await this.getAllPendingRequests();
        
        for (const request of requests) {
            try {
                await this.executeRequest(request);
                await this.removeRequest(request.id);
            } catch (error) {
                await this.handleRequestFailure(request, error);
            }
        }
    }
    
    async executeRequest(request) {
        const { tag, data } = request;
        
        switch (tag) {
            case 'user-data-update':
                return this.syncUserData(data);
            case 'offline-form-submission':
                return this.submitForm(data);
            case 'analytics-data':
                return this.sendAnalytics(data);
            case 'file-upload':
                return this.uploadFile(data);
            default:
                throw new Error(`Unknown sync tag: ${tag}`);
        }
    }
    
    async syncUserData(data) {
        const response = await fetch('/api/user/sync', {
            method: 'POST',
            headers: {
                'Content-Type': 'application/json',
            },
            body: JSON.stringify(data)
        });
        
        if (!response.ok) {
            throw new Error(`Sync failed: ${response.status}`);
        }
        
        return response.json();
    }
    
    async submitForm(data) {
        const response = await fetch(data.url, {
            method: data.method || 'POST',
            headers: {
                'Content-Type': 'application/json',
                ...data.headers
            },
            body: JSON.stringify(data.formData)
        });
        
        if (!response.ok) {
            throw new Error(`Form submission failed: ${response.status}`);
        }
        
        // Notify user of successful submission
        await this.showNotification('Form submitted successfully!');
        
        return response.json();
    }
    
    async uploadFile(data) {
        const formData = new FormData();
        
        // Reconstruct file from stored data
        const blob = new Blob([data.fileData], { type: data.fileType });
        formData.append('file', blob, data.fileName);
        
        Object.entries(data.metadata || {}).forEach(([key, value]) => {
            formData.append(key, value);
        });
        
        const response = await fetch(data.uploadUrl, {
            method: 'POST',
            body: formData
        });
        
        if (!response.ok) {
            throw new Error(`File upload failed: ${response.status}`);
        }
        
        return response.json();
    }
    
    async handleRequestFailure(request, error) {
        request.retries++;
        request.lastError = error.message;
        request.lastAttempt = Date.now();
        
        if (request.retries < request.maxRetries) {
            // Schedule retry with exponential backoff
            const delay = this.retryDelays[request.retries - 1] || 30000;
            request.nextRetry = Date.now() + delay;
            
            await this.updateRequest(request.id, request);
            
            // Register for retry sync
            if ('serviceWorker' in navigator) {
                const registration = await navigator.serviceWorker.ready;
                await registration.sync.register(this.RETRY_TAG);
            }
        } else {
            // Max retries reached
            await this.handlePermanentFailure(request, error);
        }
    }
    
    async handlePermanentFailure(request, error) {
        // Store failed request for manual review
        await this.storeFailedRequest(request, error);
        
        // Notify user
        await this.showNotification(
            'Some data could not be synchronized. Please check your connection.',
            { 
                tag: 'sync-failure',
                data: { requestId: request.id }
            }
        );
        
        // Remove from pending requests
        await this.removeRequest(request.id);
    }
    
    async retryFailedRequests() {
        const now = Date.now();
        const requests = await this.getAllPendingRequests();
        const retryRequests = requests.filter(req => 
            req.retries > 0 && req.nextRetry <= now
        );
        
        for (const request of retryRequests) {
            try {
                await this.executeRequest(request);
                await this.removeRequest(request.id);
            } catch (error) {
                await this.handleRequestFailure(request, error);
            }
        }
    }
    
    // IndexedDB operations
    async openDatabase() {
        return new Promise((resolve, reject) => {
            const request = indexedDB.open('sync-store', 1);
            
            request.onerror = () => reject(request.error);
            request.onsuccess = () => resolve(request.result);
            
            request.onupgradeneeded = (event) => {
                const db = event.target.result;
                
                if (!db.objectStoreNames.contains('pending-requests')) {
                    const store = db.createObjectStore('pending-requests', { keyPath: 'id' });
                    store.createIndex('tag', 'tag', { unique: false });
                    store.createIndex('timestamp', 'timestamp', { unique: false });
                }
                
                if (!db.objectStoreNames.contains('failed-requests')) {
                    db.createObjectStore('failed-requests', { keyPath: 'id' });
                }
            };
        });
    }
    
    async storeRequest(id, request) {
        const db = await this.openDatabase();
        const transaction = db.transaction(['pending-requests'], 'readwrite');
        const store = transaction.objectStore('pending-requests');
        
        await store.put({ id, ...request });
    }
    
    async getAllPendingRequests() {
        const db = await this.openDatabase();
        const transaction = db.transaction(['pending-requests'], 'readonly');
        const store = transaction.objectStore('pending-requests');
        
        return new Promise((resolve, reject) => {
            const request = store.getAll();
            request.onsuccess = () => resolve(request.result);
            request.onerror = () => reject(request.error);
        });
    }
    
    async removeRequest(id) {
        const db = await this.openDatabase();
        const transaction = db.transaction(['pending-requests'], 'readwrite');
        const store = transaction.objectStore('pending-requests');
        
        await store.delete(id);
    }
    
    async updateRequest(id, request) {
        await this.storeRequest(id, request);
    }
    
    async storeFailedRequest(request, error) {
        const db = await this.openDatabase();
        const transaction = db.transaction(['failed-requests'], 'readwrite');
        const store = transaction.objectStore('failed-requests');
        
        await store.put({
            id: request.id,
            request,
            error: error.message,
            failedAt: Date.now()
        });
    }
    
    async showNotification(message, options = {}) {
        if ('Notification' in window && Notification.permission === 'granted') {
            const registration = await navigator.serviceWorker.ready;
            await registration.showNotification(message, {
                icon: '/icons/icon-192x192.png',
                badge: '/icons/badge-72x72.png',
                ...options
            });
        }
    }
}

// Service Worker registration
const syncManager = new BackgroundSyncManager();

self.addEventListener('sync', event => {
    event.waitUntil(syncManager.handleSync(event));
});

// Usage in main app
class OfflineFormHandler {
    constructor() {
        this.syncManager = new BackgroundSyncManager();
    }
    
    async submitForm(formData, url) {
        if (navigator.onLine) {
            // Direct submission when online
            try {
                const response = await fetch(url, {
                    method: 'POST',
                    headers: { 'Content-Type': 'application/json' },
                    body: JSON.stringify(formData)
                });
                
                if (response.ok) {
                    return response.json();
                }
            } catch (error) {
                // Network error - queue for background sync
            }
        }
        
        // Queue for background sync
        const requestId = await this.syncManager.queueRequest('offline-form-submission', {
            formData,
            url,
            method: 'POST',
            timestamp: Date.now()
        });
        
        return { queued: true, requestId };
    }
}
```

---

## 🔔 Push Notifications

### Advanced Push Notification System

```javascript
// push-notifications.js

class PushNotificationManager {
    constructor() {
        this.publicVapidKey = 'YOUR_PUBLIC_VAPID_KEY';
        this.notificationQueue = [];
        this.maxNotifications = 5;
        this.notificationCooldown = 5000; // 5 seconds between notifications
        this.lastNotificationTime = 0;
    }
    
    async initialize() {
        // Check for browser support
        if (!('Notification' in window) || !('serviceWorker' in navigator) || !('PushManager' in window)) {
            throw new Error('Push notifications not supported');
        }
        
        // Request permission
        const permission = await this.requestPermission();
        if (permission !== 'granted') {
            throw new Error('Notification permission denied');
        }
        
        // Subscribe to push service
        const subscription = await this.subscribeToPush();
        
        // Send subscription to server
        await this.sendSubscriptionToServer(subscription);
        
        return subscription;
    }
    
    async requestPermission() {
        if (Notification.permission === 'granted') {
            return 'granted';
        }
        
        if (Notification.permission === 'denied') {
            return 'denied';
        }
        
        return await Notification.requestPermission();
    }
    
    async subscribeToPush() {
        const registration = await navigator.serviceWorker.ready;
        
        // Check if already subscribed
        let subscription = await registration.pushManager.getSubscription();
        
        if (!subscription) {
            // Create new subscription
            subscription = await registration.pushManager.subscribe({
                userVisibleOnly: true,
                applicationServerKey: this.urlBase64ToUint8Array(this.publicVapidKey)
            });
        }
        
        return subscription;
    }
    
    async sendSubscriptionToServer(subscription) {
        await fetch('/api/notifications/subscribe', {
            method: 'POST',
            headers: {
                'Content-Type': 'application/json',
            },
            body: JSON.stringify({
                subscription,
                userId: this.getCurrentUserId(),
                preferences: this.getNotificationPreferences()
            })
        });
    }
    
    // Handle push events in service worker
    async handlePushEvent(event) {
        const options = {
            body: 'You have a new notification',
            icon: '/icons/icon-192x192.png',
            badge: '/icons/badge-72x72.png',
            tag: 'default',
            requireInteraction: false,
            actions: [
                {
                    action: 'view',
                    title: 'View',
                    icon: '/icons/view.png'
                },
                {
                    action: 'dismiss',
                    title: 'Dismiss',
                    icon: '/icons/dismiss.png'
                }
            ],
            data: {}
        };
        
        try {
            const payload = event.data ? event.data.json() : {};
            
            // Merge payload with default options
            Object.assign(options, payload);
            
            // Apply notification rules
            if (await this.shouldShowNotification(options)) {
                await this.showNotification(options);
            }
        } catch (error) {
            console.error('Error handling push event:', error);
            // Show default notification on error
            await registration.showNotification('New Notification', options);
        }
    }
    
    async shouldShowNotification(options) {
        // Rate limiting
        const now = Date.now();
        if (now - this.lastNotificationTime < this.notificationCooldown) {
            return false;
        }
        
        // Check user preferences
        const preferences = await this.getStoredPreferences();
        if (!preferences.enabled) {
            return false;
        }
        
        // Check notification type preferences
        if (options.type && !preferences.types[options.type]) {
            return false;
        }
        
        // Check quiet hours
        if (this.isQuietHours(preferences.quietHours)) {
            // Queue for later or show as silent
            if (options.priority === 'high') {
                options.silent = true;
            } else {
                await this.queueNotification(options);
                return false;
            }
        }
        
        // Check duplicate notifications
        if (await this.isDuplicate(options)) {
            return false;
        }
        
        return true;
    }
    
    async showNotification(options) {
        const registration = await navigator.serviceWorker.ready;
        
        // Group similar notifications
        if (options.group) {
            const existingNotifications = await registration.getNotifications({
                tag: options.group
            });
            
            if (existingNotifications.length > 0) {
                options.body = `${existingNotifications.length + 1} new messages`;
                options.tag = options.group;
            }
        }
        
        await registration.showNotification(options.title, options);
        this.lastNotificationTime = Date.now();
        
        // Track notification analytics
        await this.trackNotification('shown', options);
    }
    
    async handleNotificationClick(event) {
        const { notification, action } = event;
        const { data } = notification;
        
        notification.close();
        
        switch (action) {
            case 'view':
                await this.openNotificationUrl(data.url);
                break;
            case 'dismiss':
                // Just close - already handled above
                break;
            default:
                // Default action - open app
                await this.openApp(data.url);
        }
        
        // Track click analytics
        await this.trackNotification('clicked', { action, data });
    }
    
    async openNotificationUrl(url) {
        const clients = await self.clients.matchAll({
            type: 'window',
            includeUncontrolled: true
        });
        
        // Check if app is already open
        for (const client of clients) {
            if (client.url === url && 'focus' in client) {
                return client.focus();
            }
        }
        
        // Open new window
        if (clients.openWindow) {
            return clients.openWindow(url);
        }
    }
    
    async openApp(url = '/') {
        const clients = await self.clients.matchAll({
            type: 'window',
            includeUncontrolled: true
        });
        
        // Focus existing client if available
        if (clients.length > 0) {
            const client = clients[0];
            client.postMessage({
                type: 'NAVIGATE',
                url: url
            });
            return client.focus();
        }
        
        // Open new window
        return self.clients.openWindow(url);
    }
    
    isQuietHours(quietHours) {
        if (!quietHours || !quietHours.enabled) {
            return false;
        }
        
        const now = new Date();
        const currentTime = now.getHours() * 60 + now.getMinutes();
        const startTime = this.parseTime(quietHours.start);
        const endTime = this.parseTime(quietHours.end);
        
        if (startTime <= endTime) {
            return currentTime >= startTime && currentTime < endTime;
        } else {
            // Quiet hours span midnight
            return currentTime >= startTime || currentTime < endTime;
        }
    }
    
    parseTime(timeString) {
        const [hours, minutes] = timeString.split(':').map(Number);
        return hours * 60 + minutes;
    }
    
    async isDuplicate(options) {
        const registration = await navigator.serviceWorker.ready;
        const existingNotifications = await registration.getNotifications();
        
        return existingNotifications.some(notification => 
            notification.tag === options.tag ||
            (notification.data.id && notification.data.id === options.data.id)
        );
    }
    
    async queueNotification(options) {
        this.notificationQueue.push({
            ...options,
            queuedAt: Date.now()
        });
        
        // Limit queue size
        if (this.notificationQueue.length > this.maxNotifications) {
            this.notificationQueue.shift();
        }
        
        // Store in IndexedDB for persistence
        await this.storeQueuedNotifications();
    }
    
    async processQueuedNotifications() {
        const preferences = await this.getStoredPreferences();
        
        if (this.isQuietHours(preferences.quietHours)) {
            return; // Still in quiet hours
        }
        
        while (this.notificationQueue.length > 0) {
            const notification = this.notificationQueue.shift();
            
            // Check if notification is still relevant (not too old)
            const age = Date.now() - notification.queuedAt;
            if (age < 24 * 60 * 60 * 1000) { // Less than 24 hours old
                await this.showNotification(notification);
                
                // Small delay between notifications
                await new Promise(resolve => setTimeout(resolve, 1000));
            }
        }
        
        await this.storeQueuedNotifications();
    }
    
    async trackNotification(event, data) {
        // Send analytics to server
        try {
            await fetch('/api/analytics/notifications', {
                method: 'POST',
                headers: { 'Content-Type': 'application/json' },
                body: JSON.stringify({
                    event,
                    data,
                    timestamp: Date.now(),
                    userId: this.getCurrentUserId()
                })
            });
        } catch (error) {
            console.error('Failed to track notification:', error);
        }
    }
    
    // Utility methods
    urlBase64ToUint8Array(base64String) {
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
    
    getCurrentUserId() {
        // Implement based on your auth system
        return localStorage.getItem('userId') || 'anonymous';
    }
    
    getNotificationPreferences() {
        // Default preferences
        return {
            enabled: true,
            types: {
                message: true,
                update: true,
                marketing: false
            },
            quietHours: {
                enabled: false,
                start: '22:00',
                end: '08:00'
            }
        };
    }
    
    async getStoredPreferences() {
        // Retrieve from IndexedDB or localStorage
        const stored = localStorage.getItem('notificationPreferences');
        return stored ? JSON.parse(stored) : this.getNotificationPreferences();
    }
    
    async storeQueuedNotifications() {
        localStorage.setItem('queuedNotifications', 
            JSON.stringify(this.notificationQueue));
    }
    
    async loadQueuedNotifications() {
        const stored = localStorage.getItem('queuedNotifications');
        this.notificationQueue = stored ? JSON.parse(stored) : [];
    }
}

// Service Worker event listeners
const pushManager = new PushNotificationManager();

self.addEventListener('push', event => {
    event.waitUntil(pushManager.handlePushEvent(event));
});

self.addEventListener('notificationclick', event => {
    event.waitUntil(pushManager.handleNotificationClick(event));
});

// Initialize in main app
if ('serviceWorker' in navigator) {
    navigator.serviceWorker.ready.then(registration => {
        const pushManager = new PushNotificationManager();
        pushManager.initialize().catch(console.error);
    });
}
```

This intermediate PWA guide covers advanced service worker patterns, background synchronization, push notifications, offline capabilities, and user engagement features essential for building robust progressive web applications.
