
# 🧪 Testing Fundamentals

## Table of Contents
- [What is Testing?](#what-is-testing)
- [Types of Testing](#types-of-testing)
- [Testing Tools](#testing-tools)
- [Unit Testing](#unit-testing)
- [Integration Testing](#integration-testing)
- [End-to-End Testing](#end-to-end-testing)
- [Frontend Testing](#frontend-testing)
- [API Testing](#api-testing)
- [Best Practices](#best-practices)

---

## 🎯 What is Testing?

**Testing** is the process of verifying that your code works as expected and prevents bugs from reaching production.

### Benefits of Testing:
- **Bug Prevention**: Catch errors early
- **Code Quality**: Improves code design
- **Confidence**: Safe refactoring and changes
- **Documentation**: Tests serve as living documentation
- **Regression Prevention**: Prevents old bugs from returning

### Testing Pyramid

```
    /\
   /  \
  / E2E\     End-to-End Tests (Few)
 /______\
/        \   Integration Tests (Some)
/__________\
/____________\ Unit Tests (Many)
```

---

## 🏗 Types of Testing

### Unit Testing
```javascript
// Testing individual functions/components
function add(a, b) {
  return a + b;
}

// Test
test('add function should return sum of two numbers', () => {
  expect(add(2, 3)).toBe(5);
  expect(add(-1, 1)).toBe(0);
  expect(add(0, 0)).toBe(0);
});
```

### Integration Testing
```javascript
// Testing interaction between components
const userService = require('./userService');
const database = require('./database');

test('user service should save user to database', async () => {
  const userData = { name: 'John', email: 'john@example.com' };
  const user = await userService.createUser(userData);
  
  expect(user).toHaveProperty('id');
  expect(user.name).toBe('John');
  
  // Verify in database
  const savedUser = await database.findUser(user.id);
  expect(savedUser).toBeTruthy();
});
```

### End-to-End Testing
```javascript
// Testing complete user workflows
describe('User Registration Flow', () => {
  test('user can register and login', async () => {
    // Navigate to registration page
    await page.goto('/register');
    
    // Fill registration form
    await page.fill('[data-testid="name"]', 'John Doe');
    await page.fill('[data-testid="email"]', 'john@example.com');
    await page.fill('[data-testid="password"]', 'password123');
    
    // Submit form
    await page.click('[data-testid="submit"]');
    
    // Verify success
    await expect(page).toHaveURL('/dashboard');
    await expect(page.locator('[data-testid="welcome"]')).toContainText('Welcome, John');
  });
});
```

---

## 🛠 Testing Tools

### JavaScript Testing Frameworks

| Framework | Best For | Features |
|-----------|----------|----------|
| **Jest** | Unit & Integration | Built-in mocking, snapshots |
| **Mocha** | Flexible testing | Highly configurable |
| **Vitest** | Vite projects | Fast, Jest-compatible |
| **Cypress** | E2E testing | Real browser testing |
| **Playwright** | Cross-browser E2E | Multiple browsers |

### Setup Jest

```bash
# Install Jest
npm install --save-dev jest

# For React projects
npm install --save-dev @testing-library/react @testing-library/jest-dom

# For TypeScript
npm install --save-dev @types/jest ts-jest
```

```json
// package.json
{
  "scripts": {
    "test": "jest",
    "test:watch": "jest --watch",
    "test:coverage": "jest --coverage"
  },
  "jest": {
    "testEnvironment": "jsdom",
    "setupFilesAfterEnv": ["<rootDir>/src/setupTests.js"]
  }
}
```

---

## ⚙️ Unit Testing

### Basic Jest Tests

```javascript
// math.js
function add(a, b) {
  return a + b;
}

function divide(a, b) {
  if (b === 0) {
    throw new Error('Division by zero');
  }
  return a / b;
}

function isEven(num) {
  return num % 2 === 0;
}

module.exports = { add, divide, isEven };
```

```javascript
// math.test.js
const { add, divide, isEven } = require('./math');

describe('Math functions', () => {
  // Basic assertions
  test('add should return sum of two numbers', () => {
    expect(add(2, 3)).toBe(5);
    expect(add(-1, 1)).toBe(0);
    expect(add(0.1, 0.2)).toBeCloseTo(0.3);
  });

  // Testing errors
  test('divide should throw error for division by zero', () => {
    expect(() => divide(10, 0)).toThrow('Division by zero');
    expect(() => divide(5, 0)).toThrow(Error);
  });

  // Boolean tests
  test('isEven should correctly identify even numbers', () => {
    expect(isEven(4)).toBeTruthy();
    expect(isEven(3)).toBeFalsy();
    expect(isEven(0)).toBe(true);
  });

  // Multiple test cases
  test.each([
    [2, 3, 5],
    [0, 0, 0],
    [-1, 1, 0],
    [100, 200, 300]
  ])('add(%i, %i) should return %i', (a, b, expected) => {
    expect(add(a, b)).toBe(expected);
  });
});
```

### Async Testing

```javascript
// userService.js
const axios = require('axios');

class UserService {
  async getUser(id) {
    const response = await axios.get(`/api/users/${id}`);
    return response.data;
  }

  async createUser(userData) {
    return new Promise((resolve) => {
      setTimeout(() => {
        resolve({
          id: Date.now(),
          ...userData,
          createdAt: new Date().toISOString()
        });
      }, 100);
    });
  }
}

module.exports = UserService;
```

```javascript
// userService.test.js
const axios = require('axios');
const UserService = require('./userService');

// Mock axios
jest.mock('axios');
const mockedAxios = axios;

describe('UserService', () => {
  let userService;

  beforeEach(() => {
    userService = new UserService();
    jest.clearAllMocks();
  });

  // Testing async/await
  test('getUser should return user data', async () => {
    const userData = { id: 1, name: 'John Doe' };
    mockedAxios.get.mockResolvedValue({ data: userData });

    const result = await userService.getUser(1);

    expect(result).toEqual(userData);
    expect(mockedAxios.get).toHaveBeenCalledWith('/api/users/1');
  });

  // Testing promises
  test('createUser should create user with timestamp', () => {
    const userData = { name: 'Jane', email: 'jane@example.com' };

    return userService.createUser(userData).then(result => {
      expect(result).toHaveProperty('id');
      expect(result).toHaveProperty('createdAt');
      expect(result.name).toBe('Jane');
    });
  });

  // Testing with done callback
  test('createUser callback style', (done) => {
    const userData = { name: 'Bob' };

    userService.createUser(userData).then(result => {
      expect(result.name).toBe('Bob');
      done();
    });
  });
});
```

### Mocking and Spies

```javascript
// fileManager.js
const fs = require('fs').promises;

class FileManager {
  async readFile(path) {
    const content = await fs.readFile(path, 'utf8');
    return content;
  }

  async writeFile(path, content) {
    await fs.writeFile(path, content);
    return true;
  }

  logOperation(operation) {
    console.log(`File operation: ${operation}`);
  }
}

module.exports = FileManager;
```

```javascript
// fileManager.test.js
const fs = require('fs').promises;
const FileManager = require('./fileManager');

// Mock fs module
jest.mock('fs', () => ({
  promises: {
    readFile: jest.fn(),
    writeFile: jest.fn()
  }
}));

describe('FileManager', () => {
  let fileManager;

  beforeEach(() => {
    fileManager = new FileManager();
    jest.clearAllMocks();
  });

  test('readFile should return file content', async () => {
    const mockContent = 'Hello, World!';
    fs.readFile.mockResolvedValue(mockContent);

    const result = await fileManager.readFile('test.txt');

    expect(result).toBe(mockContent);
    expect(fs.readFile).toHaveBeenCalledWith('test.txt', 'utf8');
  });

  test('writeFile should call fs.writeFile', async () => {
    fs.writeFile.mockResolvedValue();

    const result = await fileManager.writeFile('test.txt', 'content');

    expect(result).toBe(true);
    expect(fs.writeFile).toHaveBeenCalledWith('test.txt', 'content');
  });

  // Spying on methods
  test('logOperation should be called', () => {
    const spy = jest.spyOn(console, 'log').mockImplementation(() => {});

    fileManager.logOperation('read');

    expect(spy).toHaveBeenCalledWith('File operation: read');
    spy.mockRestore();
  });
});
```

---

## 🔗 Integration Testing

### Database Integration

```javascript
// userRepository.js
const mongoose = require('mongoose');

const userSchema = new mongoose.Schema({
  name: String,
  email: String,
  age: Number
});

const User = mongoose.model('User', userSchema);

class UserRepository {
  async create(userData) {
    const user = new User(userData);
    return await user.save();
  }

  async findByEmail(email) {
    return await User.findOne({ email });
  }

  async findAll() {
    return await User.find();
  }

  async deleteById(id) {
    return await User.findByIdAndDelete(id);
  }
}

module.exports = UserRepository;
```

```javascript
// userRepository.test.js
const mongoose = require('mongoose');
const { MongoMemoryServer } = require('mongodb-memory-server');
const UserRepository = require('./userRepository');

describe('UserRepository Integration Tests', () => {
  let mongoServer;
  let userRepository;

  beforeAll(async () => {
    // Start in-memory MongoDB
    mongoServer = await MongoMemoryServer.create();
    const mongoUri = mongoServer.getUri();
    await mongoose.connect(mongoUri);
    
    userRepository = new UserRepository();
  });

  afterAll(async () => {
    // Cleanup
    await mongoose.disconnect();
    await mongoServer.stop();
  });

  beforeEach(async () => {
    // Clear database before each test
    await mongoose.connection.db.dropDatabase();
  });

  test('should create and retrieve user', async () => {
    const userData = {
      name: 'John Doe',
      email: 'john@example.com',
      age: 30
    };

    // Create user
    const createdUser = await userRepository.create(userData);
    expect(createdUser).toHaveProperty('_id');
    expect(createdUser.name).toBe('John Doe');

    // Retrieve user
    const foundUser = await userRepository.findByEmail('john@example.com');
    expect(foundUser).toBeTruthy();
    expect(foundUser.name).toBe('John Doe');
  });

  test('should return null for non-existent user', async () => {
    const user = await userRepository.findByEmail('nonexistent@example.com');
    expect(user).toBeNull();
  });

  test('should delete user', async () => {
    const userData = { name: 'Jane', email: 'jane@example.com', age: 25 };
    const createdUser = await userRepository.create(userData);

    const deleted = await userRepository.deleteById(createdUser._id);
    expect(deleted).toBeTruthy();

    const foundUser = await userRepository.findByEmail('jane@example.com');
    expect(foundUser).toBeNull();
  });
});
```

### API Integration Testing

```javascript
// app.js
const express = require('express');
const UserRepository = require('./userRepository');

const app = express();
app.use(express.json());

const userRepository = new UserRepository();

app.post('/api/users', async (req, res) => {
  try {
    const user = await userRepository.create(req.body);
    res.status(201).json(user);
  } catch (error) {
    res.status(400).json({ error: error.message });
  }
});

app.get('/api/users', async (req, res) => {
  try {
    const users = await userRepository.findAll();
    res.json(users);
  } catch (error) {
    res.status(500).json({ error: error.message });
  }
});

module.exports = app;
```

```javascript
// app.test.js
const request = require('supertest');
const mongoose = require('mongoose');
const { MongoMemoryServer } = require('mongodb-memory-server');
const app = require('./app');

describe('API Integration Tests', () => {
  let mongoServer;

  beforeAll(async () => {
    mongoServer = await MongoMemoryServer.create();
    const mongoUri = mongoServer.getUri();
    await mongoose.connect(mongoUri);
  });

  afterAll(async () => {
    await mongoose.disconnect();
    await mongoServer.stop();
  });

  beforeEach(async () => {
    await mongoose.connection.db.dropDatabase();
  });

  describe('POST /api/users', () => {
    test('should create new user', async () => {
      const userData = {
        name: 'John Doe',
        email: 'john@example.com',
        age: 30
      };

      const response = await request(app)
        .post('/api/users')
        .send(userData)
        .expect(201);

      expect(response.body).toHaveProperty('_id');
      expect(response.body.name).toBe('John Doe');
    });

    test('should return 400 for invalid data', async () => {
      const response = await request(app)
        .post('/api/users')
        .send({}) // Empty data
        .expect(400);

      expect(response.body).toHaveProperty('error');
    });
  });

  describe('GET /api/users', () => {
    test('should return empty array initially', async () => {
      const response = await request(app)
        .get('/api/users')
        .expect(200);

      expect(response.body).toEqual([]);
    });

    test('should return users after creation', async () => {
      // Create user first
      await request(app)
        .post('/api/users')
        .send({ name: 'John', email: 'john@example.com', age: 30 });

      const response = await request(app)
        .get('/api/users')
        .expect(200);

      expect(response.body).toHaveLength(1);
      expect(response.body[0].name).toBe('John');
    });
  });
});
```

---

## 🌐 End-to-End Testing

### Playwright Setup

```bash
# Install Playwright
npm install --save-dev @playwright/test

# Install browsers
npx playwright install
```

```javascript
// playwright.config.js
module.exports = {
  testDir: './e2e',
  timeout: 30000,
  use: {
    baseURL: 'http://localhost:3000',
    screenshot: 'only-on-failure',
    video: 'retain-on-failure'
  },
  projects: [
    { name: 'chromium', use: { ...devices['Desktop Chrome'] } },
    { name: 'firefox', use: { ...devices['Desktop Firefox'] } },
    { name: 'webkit', use: { ...devices['Desktop Safari'] } }
  ]
};
```

### E2E Test Examples

```javascript
// e2e/login.spec.js
const { test, expect } = require('@playwright/test');

test.describe('User Authentication', () => {
  test.beforeEach(async ({ page }) => {
    await page.goto('/');
  });

  test('user can login with valid credentials', async ({ page }) => {
    // Navigate to login page
    await page.click('[data-testid="login-button"]');
    await expect(page).toHaveURL('/login');

    // Fill login form
    await page.fill('[data-testid="email"]', 'user@example.com');
    await page.fill('[data-testid="password"]', 'password123');

    // Submit form
    await page.click('[data-testid="submit"]');

    // Verify successful login
    await expect(page).toHaveURL('/dashboard');
    await expect(page.locator('[data-testid="user-menu"]')).toBeVisible();
  });

  test('user cannot login with invalid credentials', async ({ page }) => {
    await page.click('[data-testid="login-button"]');
    
    await page.fill('[data-testid="email"]', 'invalid@example.com');
    await page.fill('[data-testid="password"]', 'wrongpassword');
    await page.click('[data-testid="submit"]');

    // Should see error message
    await expect(page.locator('[data-testid="error-message"]')).toBeVisible();
    await expect(page.locator('[data-testid="error-message"]')).toContainText('Invalid credentials');
  });

  test('user can logout', async ({ page }) => {
    // Login first
    await page.goto('/login');
    await page.fill('[data-testid="email"]', 'user@example.com');
    await page.fill('[data-testid="password"]', 'password123');
    await page.click('[data-testid="submit"]');

    // Logout
    await page.click('[data-testid="user-menu"]');
    await page.click('[data-testid="logout"]');

    // Verify logout
    await expect(page).toHaveURL('/');
    await expect(page.locator('[data-testid="login-button"]')).toBeVisible();
  });
});
```

### Form Testing

```javascript
// e2e/contact-form.spec.js
const { test, expect } = require('@playwright/test');

test.describe('Contact Form', () => {
  test('user can submit contact form', async ({ page }) => {
    await page.goto('/contact');

    // Fill form
    await page.fill('[data-testid="name"]', 'John Doe');
    await page.fill('[data-testid="email"]', 'john@example.com');
    await page.selectOption('[data-testid="subject"]', 'General Inquiry');
    await page.fill('[data-testid="message"]', 'This is a test message');

    // Submit form
    await page.click('[data-testid="submit"]');

    // Wait for success message
    await expect(page.locator('[data-testid="success-message"]')).toBeVisible();
    await expect(page.locator('[data-testid="success-message"]')).toContainText('Thank you');

    // Verify form is reset
    await expect(page.locator('[data-testid="name"]')).toHaveValue('');
  });

  test('form validation works correctly', async ({ page }) => {
    await page.goto('/contact');

    // Try to submit empty form
    await page.click('[data-testid="submit"]');

    // Check validation messages
    await expect(page.locator('[data-testid="name-error"]')).toContainText('Name is required');
    await expect(page.locator('[data-testid="email-error"]')).toContainText('Email is required');

    // Fill invalid email
    await page.fill('[data-testid="email"]', 'invalid-email');
    await page.click('[data-testid="submit"]');

    await expect(page.locator('[data-testid="email-error"]')).toContainText('Invalid email');
  });
});
```

---

## ⚛️ Frontend Testing

### React Component Testing

```javascript
// Button.jsx
import React from 'react';

const Button = ({ children, onClick, variant = 'primary', disabled = false }) => {
  return (
    <button
      className={`btn btn-${variant}`}
      onClick={onClick}
      disabled={disabled}
      data-testid="button"
    >
      {children}
    </button>
  );
};

export default Button;
```

```javascript
// Button.test.jsx
import React from 'react';
import { render, screen, fireEvent } from '@testing-library/react';
import '@testing-library/jest-dom';
import Button from './Button';

describe('Button Component', () => {
  test('renders button with text', () => {
    render(<Button>Click me</Button>);
    
    const button = screen.getByTestId('button');
    expect(button).toBeInTheDocument();
    expect(button).toHaveTextContent('Click me');
  });

  test('calls onClick handler when clicked', () => {
    const mockClick = jest.fn();
    render(<Button onClick={mockClick}>Click me</Button>);
    
    const button = screen.getByTestId('button');
    fireEvent.click(button);
    
    expect(mockClick).toHaveBeenCalledTimes(1);
  });

  test('applies correct CSS classes', () => {
    render(<Button variant="secondary">Button</Button>);
    
    const button = screen.getByTestId('button');
    expect(button).toHaveClass('btn', 'btn-secondary');
  });

  test('is disabled when disabled prop is true', () => {
    render(<Button disabled>Disabled Button</Button>);
    
    const button = screen.getByTestId('button');
    expect(button).toBeDisabled();
  });

  test('does not call onClick when disabled', () => {
    const mockClick = jest.fn();
    render(<Button onClick={mockClick} disabled>Disabled</Button>);
    
    const button = screen.getByTestId('button');
    fireEvent.click(button);
    
    expect(mockClick).not.toHaveBeenCalled();
  });
});
```

### Form Component Testing

```javascript
// ContactForm.jsx
import React, { useState } from 'react';

const ContactForm = ({ onSubmit }) => {
  const [formData, setFormData] = useState({
    name: '',
    email: '',
    message: ''
  });
  const [errors, setErrors] = useState({});

  const validateForm = () => {
    const newErrors = {};
    
    if (!formData.name.trim()) {
      newErrors.name = 'Name is required';
    }
    
    if (!formData.email.trim()) {
      newErrors.email = 'Email is required';
    } else if (!/\S+@\S+\.\S+/.test(formData.email)) {
      newErrors.email = 'Invalid email format';
    }
    
    if (!formData.message.trim()) {
      newErrors.message = 'Message is required';
    }
    
    setErrors(newErrors);
    return Object.keys(newErrors).length === 0;
  };

  const handleSubmit = (e) => {
    e.preventDefault();
    
    if (validateForm()) {
      onSubmit(formData);
      setFormData({ name: '', email: '', message: '' });
    }
  };

  const handleChange = (e) => {
    const { name, value } = e.target;
    setFormData(prev => ({ ...prev, [name]: value }));
    
    // Clear error when user starts typing
    if (errors[name]) {
      setErrors(prev => ({ ...prev, [name]: '' }));
    }
  };

  return (
    <form onSubmit={handleSubmit} data-testid="contact-form">
      <div>
        <input
          type="text"
          name="name"
          value={formData.name}
          onChange={handleChange}
          placeholder="Name"
          data-testid="name-input"
        />
        {errors.name && <span data-testid="name-error">{errors.name}</span>}
      </div>
      
      <div>
        <input
          type="email"
          name="email"
          value={formData.email}
          onChange={handleChange}
          placeholder="Email"
          data-testid="email-input"
        />
        {errors.email && <span data-testid="email-error">{errors.email}</span>}
      </div>
      
      <div>
        <textarea
          name="message"
          value={formData.message}
          onChange={handleChange}
          placeholder="Message"
          data-testid="message-input"
        />
        {errors.message && <span data-testid="message-error">{errors.message}</span>}
      </div>
      
      <button type="submit" data-testid="submit-button">Submit</button>
    </form>
  );
};

export default ContactForm;
```

```javascript
// ContactForm.test.jsx
import React from 'react';
import { render, screen, fireEvent, waitFor } from '@testing-library/react';
import userEvent from '@testing-library/user-event';
import ContactForm from './ContactForm';

describe('ContactForm', () => {
  test('renders all form fields', () => {
    render(<ContactForm onSubmit={jest.fn()} />);
    
    expect(screen.getByTestId('name-input')).toBeInTheDocument();
    expect(screen.getByTestId('email-input')).toBeInTheDocument();
    expect(screen.getByTestId('message-input')).toBeInTheDocument();
    expect(screen.getByTestId('submit-button')).toBeInTheDocument();
  });

  test('shows validation errors for empty form', async () => {
    render(<ContactForm onSubmit={jest.fn()} />);
    
    fireEvent.click(screen.getByTestId('submit-button'));
    
    await waitFor(() => {
      expect(screen.getByTestId('name-error')).toHaveTextContent('Name is required');
      expect(screen.getByTestId('email-error')).toHaveTextContent('Email is required');
      expect(screen.getByTestId('message-error')).toHaveTextContent('Message is required');
    });
  });

  test('shows email validation error for invalid email', async () => {
    const user = userEvent.setup();
    render(<ContactForm onSubmit={jest.fn()} />);
    
    await user.type(screen.getByTestId('email-input'), 'invalid-email');
    fireEvent.click(screen.getByTestId('submit-button'));
    
    await waitFor(() => {
      expect(screen.getByTestId('email-error')).toHaveTextContent('Invalid email format');
    });
  });

  test('submits form with valid data', async () => {
    const mockSubmit = jest.fn();
    const user = userEvent.setup();
    
    render(<ContactForm onSubmit={mockSubmit} />);
    
    await user.type(screen.getByTestId('name-input'), 'John Doe');
    await user.type(screen.getByTestId('email-input'), 'john@example.com');
    await user.type(screen.getByTestId('message-input'), 'Test message');
    
    fireEvent.click(screen.getByTestId('submit-button'));
    
    await waitFor(() => {
      expect(mockSubmit).toHaveBeenCalledWith({
        name: 'John Doe',
        email: 'john@example.com',
        message: 'Test message'
      });
    });
  });

  test('clears form after successful submission', async () => {
    const mockSubmit = jest.fn();
    const user = userEvent.setup();
    
    render(<ContactForm onSubmit={mockSubmit} />);
    
    await user.type(screen.getByTestId('name-input'), 'John Doe');
    await user.type(screen.getByTestId('email-input'), 'john@example.com');
    await user.type(screen.getByTestId('message-input'), 'Test message');
    
    fireEvent.click(screen.getByTestId('submit-button'));
    
    await waitFor(() => {
      expect(screen.getByTestId('name-input')).toHaveValue('');
      expect(screen.getByTestId('email-input')).toHaveValue('');
      expect(screen.getByTestId('message-input')).toHaveValue('');
    });
  });
});
```

---

## 🔌 API Testing

### API Test Suite

```javascript
// api.test.js
const request = require('supertest');
const app = require('../app');

describe('User API', () => {
  describe('GET /api/users', () => {
    test('should return empty array initially', async () => {
      const response = await request(app)
        .get('/api/users')
        .expect(200);

      expect(response.body).toEqual([]);
    });
  });

  describe('POST /api/users', () => {
    test('should create new user with valid data', async () => {
      const userData = {
        name: 'John Doe',
        email: 'john@example.com',
        age: 30
      };

      const response = await request(app)
        .post('/api/users')
        .send(userData)
        .expect(201);

      expect(response.body).toMatchObject(userData);
      expect(response.body).toHaveProperty('id');
      expect(response.body).toHaveProperty('createdAt');
    });

    test('should return 400 for missing required fields', async () => {
      const response = await request(app)
        .post('/api/users')
        .send({ name: 'John' }) // Missing email
        .expect(400);

      expect(response.body).toHaveProperty('error');
      expect(response.body.error).toContain('email');
    });

    test('should return 400 for invalid email format', async () => {
      const userData = {
        name: 'John Doe',
        email: 'invalid-email',
        age: 30
      };

      const response = await request(app)
        .post('/api/users')
        .send(userData)
        .expect(400);

      expect(response.body.error).toContain('email');
    });
  });

  describe('GET /api/users/:id', () => {
    test('should return user by id', async () => {
      // Create user first
      const createResponse = await request(app)
        .post('/api/users')
        .send({ name: 'John', email: 'john@example.com', age: 30 });

      const userId = createResponse.body.id;

      // Get user by id
      const response = await request(app)
        .get(`/api/users/${userId}`)
        .expect(200);

      expect(response.body.id).toBe(userId);
      expect(response.body.name).toBe('John');
    });

    test('should return 404 for non-existent user', async () => {
      const response = await request(app)
        .get('/api/users/999')
        .expect(404);

      expect(response.body).toHaveProperty('error');
    });
  });

  describe('PUT /api/users/:id', () => {
    test('should update user', async () => {
      // Create user first
      const createResponse = await request(app)
        .post('/api/users')
        .send({ name: 'John', email: 'john@example.com', age: 30 });

      const userId = createResponse.body.id;

      // Update user
      const updateData = { name: 'John Updated', age: 31 };
      const response = await request(app)
        .put(`/api/users/${userId}`)
        .send(updateData)
        .expect(200);

      expect(response.body.name).toBe('John Updated');
      expect(response.body.age).toBe(31);
      expect(response.body.email).toBe('john@example.com'); // Should remain
    });
  });

  describe('DELETE /api/users/:id', () => {
    test('should delete user', async () => {
      // Create user first
      const createResponse = await request(app)
        .post('/api/users')
        .send({ name: 'John', email: 'john@example.com', age: 30 });

      const userId = createResponse.body.id;

      // Delete user
      await request(app)
        .delete(`/api/users/${userId}`)
        .expect(200);

      // Verify user is deleted
      await request(app)
        .get(`/api/users/${userId}`)
        .expect(404);
    });
  });
});
```

---

## ✅ Best Practices

### Test Organization

```javascript
// Good test structure
describe('UserService', () => {
  describe('createUser', () => {
    test('should create user with valid data', () => {
      // Test implementation
    });

    test('should throw error for duplicate email', () => {
      // Test implementation
    });
  });

  describe('updateUser', () => {
    test('should update existing user', () => {
      // Test implementation
    });

    test('should return null for non-existent user', () => {
      // Test implementation
    });
  });
});
```

### Test Data Management

```javascript
// Test data factory
const createUserData = (overrides = {}) => ({
  name: 'John Doe',
  email: 'john@example.com',
  age: 30,
  ...overrides
});

// Usage
test('should create user', () => {
  const userData = createUserData({ age: 25 });
  // Test with userData
});

test('should handle long names', () => {
  const userData = createUserData({ name: 'Very Long Name That Exceeds Normal Length' });
  // Test with userData
});
```

### Mock Management

```javascript
// Setup and cleanup mocks
describe('UserService with mocks', () => {
  beforeEach(() => {
    jest.clearAllMocks();
  });

  afterEach(() => {
    jest.restoreAllMocks();
  });

  test('should call external API', async () => {
    const mockFetch = jest.fn().mockResolvedValue({
      ok: true,
      json: () => Promise.resolve({ id: 1, name: 'John' })
    });
    global.fetch = mockFetch;

    // Test implementation
    
    expect(mockFetch).toHaveBeenCalledWith('/api/users/1');
  });
});
```

### Async Test Patterns

```javascript
// Using async/await
test('async operation should succeed', async () => {
  const result = await asyncFunction();
  expect(result).toBe('success');
});

// Using resolves/rejects
test('promise should resolve', () => {
  return expect(asyncFunction()).resolves.toBe('success');
});

test('promise should reject', () => {
  return expect(failingAsyncFunction()).rejects.toThrow('Error message');
});

// Testing with timers
test('delayed function should be called', (done) => {
  jest.useFakeTimers();
  
  const callback = jest.fn();
  delayedFunction(callback, 1000);
  
  jest.advanceTimersByTime(1000);
  
  expect(callback).toHaveBeenCalled();
  jest.useRealTimers();
  done();
});
```

## 📚 Quick Reference

### Jest Matchers

| Matcher | Purpose | Example |
|---------|---------|---------|
| `toBe()` | Exact equality | `expect(2 + 2).toBe(4)` |
| `toEqual()` | Deep equality | `expect(obj1).toEqual(obj2)` |
| `toBeNull()` | Null value | `expect(value).toBeNull()` |
| `toBeUndefined()` | Undefined value | `expect(value).toBeUndefined()` |
| `toBeTruthy()` | Truthy value | `expect(true).toBeTruthy()` |
| `toContain()` | Array/string contains | `expect(['a', 'b']).toContain('a')` |
| `toThrow()` | Function throws | `expect(() => error()).toThrow()` |

### Testing Libraries

| Library | Purpose | Use Case |
|---------|---------|----------|
| **Jest** | Test framework | Unit and integration tests |
| **React Testing Library** | React component testing | Component behavior testing |
| **Supertest** | HTTP testing | API endpoint testing |
| **Playwright** | E2E testing | Cross-browser testing |
| **MSW** | API mocking | Mock external services |

### Test Types

| Type | Scope | Speed | Cost |
|------|-------|-------|------|
| **Unit** | Single function/component | Fast | Low |
| **Integration** | Multiple components | Medium | Medium |
| **E2E** | Full application | Slow | High |

### Next Steps:
1. Practice TDD (Test-Driven Development)
2. Learn advanced testing patterns
3. Explore visual regression testing
4. Study performance testing
5. Implement CI/CD with testing

This testing guide provides comprehensive coverage of testing strategies for modern web development!
