
# ⚙️ DevOps & Deployment Basics

## Table of Contents
- [What is DevOps?](#what-is-devops)
- [Development Environment](#development-environment)
- [Version Control Workflow](#version-control-workflow)
- [Continuous Integration](#continuous-integration)
- [Continuous Deployment](#continuous-deployment)
- [Containerization with Docker](#containerization-with-docker)
- [Cloud Deployment](#cloud-deployment)
- [Monitoring and Logging](#monitoring-and-logging)

---

## 🎯 What is DevOps?

**DevOps** is a set of practices that combines software development (Dev) and IT operations (Ops) to shorten the development lifecycle and provide continuous delivery.

### DevOps Lifecycle:
```
Plan → Code → Build → Test → Release → Deploy → Operate → Monitor
  ↑                                                              ↓
  └──────────────────── Feedback Loop ────────────────────────────┘
```

### Key Principles:
- **Automation**: Automate repetitive tasks
- **Continuous Integration**: Frequent code integration
- **Continuous Deployment**: Automated deployments
- **Infrastructure as Code**: Manage infrastructure programmatically
- **Monitoring**: Continuous application and infrastructure monitoring

---

## 💻 Development Environment

### Environment Variables

```bash
# .env file (never commit to version control)
NODE_ENV=development
PORT=5000
DATABASE_URL=mongodb://localhost:27017/myapp
JWT_SECRET=your-secret-key
API_KEY=your-api-key

# .env.example (commit this template)
NODE_ENV=development
PORT=5000
DATABASE_URL=mongodb://localhost:27017/myapp
JWT_SECRET=your-secret-key-here
API_KEY=your-api-key-here
```

```javascript
// Using environment variables in Node.js
require('dotenv').config();

const config = {
    port: process.env.PORT || 3000,
    mongoUrl: process.env.DATABASE_URL || 'mongodb://localhost:27017/myapp',
    jwtSecret: process.env.JWT_SECRET,
    nodeEnv: process.env.NODE_ENV || 'development'
};

// Different configurations for different environments
const configs = {
    development: {
        database: {
            host: 'localhost',
            name: 'myapp_dev'
        },
        logging: true
    },
    test: {
        database: {
            host: 'localhost',
            name: 'myapp_test'
        },
        logging: false
    },
    production: {
        database: {
            host: process.env.DB_HOST,
            name: process.env.DB_NAME
        },
        logging: false
    }
};

module.exports = configs[process.env.NODE_ENV || 'development'];
```

### Package Management

```json
// package.json with scripts
{
  "name": "my-app",
  "version": "1.0.0",
  "scripts": {
    "start": "node server.js",
    "dev": "nodemon server.js",
    "test": "jest",
    "test:watch": "jest --watch",
    "test:coverage": "jest --coverage",
    "build": "webpack --mode production",
    "lint": "eslint .",
    "lint:fix": "eslint . --fix",
    "prepare": "husky install"
  },
  "dependencies": {
    "express": "^4.18.0",
    "mongoose": "^6.0.0",
    "dotenv": "^16.0.0"
  },
  "devDependencies": {
    "nodemon": "^2.0.0",
    "jest": "^28.0.0",
    "eslint": "^8.0.0",
    "husky": "^8.0.0",
    "lint-staged": "^13.0.0"
  }
}
```

---

## 🌿 Version Control Workflow

### Git Workflow (GitFlow)

```bash
# Main branches
main        # Production-ready code
develop     # Integration branch

# Supporting branches
feature/*   # New features
release/*   # Release preparation
hotfix/*    # Emergency fixes

# Example workflow
git checkout develop
git pull origin develop

# Create feature branch
git checkout -b feature/user-authentication
# Work on feature...
git add .
git commit -m "Add user authentication"
git push origin feature/user-authentication

# Create pull request
# After review and approval, merge to develop
git checkout develop
git pull origin develop
git merge feature/user-authentication
git push origin develop

# Delete feature branch
git branch -d feature/user-authentication
git push origin --delete feature/user-authentication
```

### Pre-commit Hooks

```json
// .husky/pre-commit
#!/usr/bin/env sh
. "$(dirname -- "$0")/_/husky.sh"

npm run lint
npm run test

// package.json
{
  "lint-staged": {
    "*.js": [
      "eslint --fix",
      "git add"
    ],
    "*.{js,css,md}": "prettier --write"
  }
}
```

---

## 🔄 Continuous Integration

### GitHub Actions CI

```yaml
# .github/workflows/ci.yml
name: CI

on:
  push:
    branches: [ main, develop ]
  pull_request:
    branches: [ main, develop ]

jobs:
  test:
    runs-on: ubuntu-latest

    strategy:
      matrix:
        node-version: [16.x, 18.x]

    steps:
    - uses: actions/checkout@v3

    - name: Use Node.js ${{ matrix.node-version }}
      uses: actions/setup-node@v3
      with:
        node-version: ${{ matrix.node-version }}
        cache: 'npm'

    - name: Install dependencies
      run: npm ci

    - name: Run linter
      run: npm run lint

    - name: Run tests
      run: npm run test:coverage

    - name: Upload coverage to Codecov
      uses: codecov/codecov-action@v3
      with:
        file: ./coverage/lcov.info

  build:
    runs-on: ubuntu-latest
    needs: test

    steps:
    - uses: actions/checkout@v3

    - name: Use Node.js
      uses: actions/setup-node@v3
      with:
        node-version: '18.x'
        cache: 'npm'

    - name: Install dependencies
      run: npm ci

    - name: Build application
      run: npm run build

    - name: Upload build artifacts
      uses: actions/upload-artifact@v3
      with:
        name: build-files
        path: dist/
```

### Jest Testing Configuration

```javascript
// jest.config.js
module.exports = {
  testEnvironment: 'node',
  coverageDirectory: 'coverage',
  collectCoverageFrom: [
    'src/**/*.js',
    '!src/index.js',
    '!src/config/*.js'
  ],
  testMatch: [
    '**/__tests__/**/*.js',
    '**/?(*.)+(spec|test).js'
  ],
  setupFilesAfterEnv: ['<rootDir>/tests/setup.js']
};

// tests/setup.js
const mongoose = require('mongoose');

beforeAll(async () => {
  await mongoose.connect('mongodb://localhost:27017/test_db');
});

afterAll(async () => {
  await mongoose.connection.dropDatabase();
  await mongoose.connection.close();
});

afterEach(async () => {
  const collections = mongoose.connection.collections;
  for (const key in collections) {
    await collections[key].deleteMany({});
  }
});
```

---

## 🚀 Continuous Deployment

### Deployment Pipeline

```yaml
# .github/workflows/deploy.yml
name: Deploy

on:
  push:
    branches: [ main ]

jobs:
  deploy:
    runs-on: ubuntu-latest

    steps:
    - uses: actions/checkout@v3

    - name: Setup Node.js
      uses: actions/setup-node@v3
      with:
        node-version: '18.x'
        cache: 'npm'

    - name: Install dependencies
      run: npm ci

    - name: Run tests
      run: npm test

    - name: Build application
      run: npm run build

    - name: Deploy to Heroku
      uses: akhileshns/heroku-deploy@v3.12.12
      with:
        heroku_api_key: ${{ secrets.HEROKU_API_KEY }}
        heroku_app_name: "your-app-name"
        heroku_email: "your-email@example.com"

    - name: Deploy to Vercel
      uses: amondnet/vercel-action@v20
      with:
        vercel-token: ${{ secrets.VERCEL_TOKEN }}
        vercel-args: '--prod'
        vercel-org-id: ${{ secrets.ORG_ID }}
        vercel-project-id: ${{ secrets.PROJECT_ID }}
```

### Environment-specific Deployments

```yaml
# Multi-environment deployment
name: Deploy

on:
  push:
    branches: [ main, develop ]

jobs:
  deploy-staging:
    if: github.ref == 'refs/heads/develop'
    runs-on: ubuntu-latest
    environment: staging
    
    steps:
    - name: Deploy to Staging
      run: |
        echo "Deploying to staging environment"
        # Deploy commands for staging

  deploy-production:
    if: github.ref == 'refs/heads/main'
    runs-on: ubuntu-latest
    environment: production
    
    steps:
    - name: Deploy to Production
      run: |
        echo "Deploying to production environment"
        # Deploy commands for production
```

---

## 🐳 Containerization with Docker

### Dockerfile

```dockerfile
# Dockerfile
FROM node:18-alpine

# Set working directory
WORKDIR /app

# Copy package files
COPY package*.json ./

# Install dependencies
RUN npm ci --only=production

# Copy source code
COPY . .

# Create non-root user
RUN addgroup -g 1001 -S nodejs
RUN adduser -S nodejs -u 1001

# Change ownership of the app directory
RUN chown -R nodejs:nodejs /app
USER nodejs

# Expose port
EXPOSE 3000

# Health check
HEALTHCHECK --interval=30s --timeout=3s --start-period=5s --retries=3 \
  CMD curl -f http://localhost:3000/health || exit 1

# Start the application
CMD ["npm", "start"]
```

### Docker Compose

```yaml
# docker-compose.yml
version: '3.8'

services:
  app:
    build: .
    ports:
      - "3000:3000"
    environment:
      - NODE_ENV=production
      - DATABASE_URL=mongodb://mongo:27017/myapp
    depends_on:
      - mongo
      - redis
    volumes:
      - ./logs:/app/logs

  mongo:
    image: mongo:5.0
    ports:
      - "27017:27017"
    environment:
      - MONGO_INITDB_ROOT_USERNAME=admin
      - MONGO_INITDB_ROOT_PASSWORD=password
    volumes:
      - mongo_data:/data/db

  redis:
    image: redis:7-alpine
    ports:
      - "6379:6379"
    command: redis-server --appendonly yes
    volumes:
      - redis_data:/data

  nginx:
    image: nginx:alpine
    ports:
      - "80:80"
    volumes:
      - ./nginx.conf:/etc/nginx/nginx.conf
    depends_on:
      - app

volumes:
  mongo_data:
  redis_data:
```

### Multi-stage Build

```dockerfile
# Multi-stage Dockerfile for optimization
FROM node:18-alpine AS builder

WORKDIR /app
COPY package*.json ./
RUN npm ci

COPY . .
RUN npm run build

# Production stage
FROM node:18-alpine AS production

WORKDIR /app

# Copy package files and install production dependencies
COPY package*.json ./
RUN npm ci --only=production && npm cache clean --force

# Copy built application from builder stage
COPY --from=builder /app/dist ./dist

# Create non-root user
RUN addgroup -g 1001 -S nodejs && adduser -S nodejs -u 1001
RUN chown -R nodejs:nodejs /app
USER nodejs

EXPOSE 3000
CMD ["node", "dist/server.js"]
```

---

## ☁️ Cloud Deployment

### Heroku Deployment

```json
// package.json
{
  "scripts": {
    "start": "node server.js",
    "heroku-postbuild": "npm run build"
  }
}
```

```javascript
// Procfile
web: npm start
```

```bash
# Heroku CLI commands
heroku create your-app-name
heroku config:set NODE_ENV=production
heroku config:set DATABASE_URL=your-mongodb-url
heroku config:set JWT_SECRET=your-jwt-secret

git push heroku main

# View logs
heroku logs --tail

# Scale dynos
heroku ps:scale web=2
```

### Vercel Deployment

```json
// vercel.json
{
  "version": 2,
  "builds": [
    {
      "src": "server.js",
      "use": "@vercel/node"
    }
  ],
  "routes": [
    {
      "src": "/(.*)",
      "dest": "/server.js"
    }
  ],
  "env": {
    "NODE_ENV": "production"
  }
}
```

### Netlify Deployment

```toml
# netlify.toml
[build]
  publish = "dist"
  command = "npm run build"

[build.environment]
  NODE_VERSION = "18"

[[redirects]]
  from = "/api/*"
  to = "/.netlify/functions/:splat"
  status = 200

[[redirects]]
  from = "/*"
  to = "/index.html"
  status = 200
```

---

## 📊 Monitoring and Logging

### Application Logging

```javascript
// logger.js
const winston = require('winston');

const logger = winston.createLogger({
  level: process.env.LOG_LEVEL || 'info',
  format: winston.format.combine(
    winston.format.timestamp(),
    winston.format.errors({ stack: true }),
    winston.format.json()
  ),
  defaultMeta: { service: 'my-app' },
  transports: [
    new winston.transports.File({ filename: 'logs/error.log', level: 'error' }),
    new winston.transports.File({ filename: 'logs/combined.log' })
  ]
});

if (process.env.NODE_ENV !== 'production') {
  logger.add(new winston.transports.Console({
    format: winston.format.simple()
  }));
}

module.exports = logger;

// Usage in application
const logger = require('./logger');

app.use((req, res, next) => {
  logger.info(`${req.method} ${req.url}`, {
    ip: req.ip,
    userAgent: req.get('User-Agent')
  });
  next();
});

app.use((err, req, res, next) => {
  logger.error('Unhandled error', {
    error: err.message,
    stack: err.stack,
    url: req.url,
    method: req.method
  });
  res.status(500).json({ error: 'Internal server error' });
});
```

### Health Checks

```javascript
// health.js
const express = require('express');
const mongoose = require('mongoose');
const router = express.Router();

router.get('/health', async (req, res) => {
  const health = {
    uptime: process.uptime(),
    message: 'OK',
    timestamp: Date.now(),
    checks: {
      database: 'unknown',
      memory: 'unknown'
    }
  };

  try {
    // Database check
    await mongoose.connection.db.admin().ping();
    health.checks.database = 'healthy';
  } catch (error) {
    health.checks.database = 'unhealthy';
    health.message = 'Degraded';
  }

  // Memory check
  const memUsage = process.memoryUsage();
  const memUsageMB = Math.round(memUsage.heapUsed / 1024 / 1024);
  health.checks.memory = `${memUsageMB}MB used`;

  if (health.checks.database === 'unhealthy') {
    return res.status(503).json(health);
  }

  res.json(health);
});

module.exports = router;
```

### Performance Monitoring

```javascript
// monitoring.js
const prometheus = require('prom-client');

// Create metrics
const httpRequestDuration = new prometheus.Histogram({
  name: 'http_request_duration_seconds',
  help: 'Duration of HTTP requests in seconds',
  labelNames: ['method', 'route', 'status_code']
});

const httpRequestsTotal = new prometheus.Counter({
  name: 'http_requests_total',
  help: 'Total number of HTTP requests',
  labelNames: ['method', 'route', 'status_code']
});

// Middleware to collect metrics
const collectMetrics = (req, res, next) => {
  const start = Date.now();
  
  res.on('finish', () => {
    const duration = (Date.now() - start) / 1000;
    const route = req.route ? req.route.path : req.path;
    
    httpRequestDuration
      .labels(req.method, route, res.statusCode)
      .observe(duration);
      
    httpRequestsTotal
      .labels(req.method, route, res.statusCode)
      .inc();
  });
  
  next();
};

// Expose metrics endpoint
const metricsRoute = (req, res) => {
  res.set('Content-Type', prometheus.register.contentType);
  res.end(prometheus.register.metrics());
};

module.exports = { collectMetrics, metricsRoute };
```

---

## 📚 Summary

### Key DevOps Concepts:
- **Environment Management**: Separate dev, staging, production
- **CI/CD Pipelines**: Automated testing and deployment
- **Containerization**: Docker for consistent environments
- **Cloud Deployment**: Various platforms and strategies
- **Monitoring**: Logging, health checks, metrics

### Best Practices:
1. **Automate everything** possible
2. **Use environment variables** for configuration
3. **Implement proper logging** and monitoring
4. **Set up health checks** for applications
5. **Use infrastructure as code**
6. **Monitor application performance**
7. **Plan for rollbacks** and disaster recovery

### Next Steps:
1. Practice with CI/CD pipelines
2. Learn Infrastructure as Code (Terraform, AWS CDK)
3. Study container orchestration (Kubernetes)
4. Explore cloud platforms (AWS, Azure, GCP)
5. Learn monitoring tools (Grafana, Prometheus, ELK stack)
