
# 🚀 Intermediate TypeScript

## Table of Contents
- [Advanced Types](#advanced-types)
- [Mapped Types](#mapped-types)
- [Conditional Types](#conditional-types)
- [Template Literal Types](#template-literal-types)
- [Modules and Namespaces](#modules-and-namespaces)
- [Decorators](#decorators)
- [Advanced Generics](#advanced-generics)
- [Type Guards and Assertions](#type-guards-and-assertions)

---

## 🔄 Advanced Types

### Union and Intersection Types

```typescript
// Union types
type Status = 'loading' | 'success' | 'error';
type StringOrNumber = string | number;

// Intersection types
interface User {
  id: number;
  name: string;
}

interface Permissions {
  canRead: boolean;
  canWrite: boolean;
  canDelete: boolean;
}

type UserWithPermissions = User & Permissions;

const admin: UserWithPermissions = {
  id: 1,
  name: 'Admin',
  canRead: true,
  canWrite: true,
  canDelete: true
};

// Discriminated unions
interface LoadingState {
  status: 'loading';
}

interface SuccessState {
  status: 'success';
  data: any;
}

interface ErrorState {
  status: 'error';
  error: string;
}

type AsyncState = LoadingState | SuccessState | ErrorState;

function handleState(state: AsyncState) {
  switch (state.status) {
    case 'loading':
      console.log('Loading...');
      break;
    case 'success':
      console.log('Data:', state.data); // TypeScript knows 'data' exists
      break;
    case 'error':
      console.log('Error:', state.error); // TypeScript knows 'error' exists
      break;
  }
}
```

### Literal Types and Type Narrowing

```typescript
// String literal types
type Theme = 'light' | 'dark' | 'auto';
type ButtonSize = 'small' | 'medium' | 'large';

// Numeric literal types
type HttpStatusCode = 200 | 404 | 500;

// Template literal types
type EventName = `on${Capitalize<string>}`;

// Type narrowing with typeof
function processValue(value: string | number) {
  if (typeof value === 'string') {
    // TypeScript knows value is string here
    return value.toUpperCase();
  } else {
    // TypeScript knows value is number here
    return value.toFixed(2);
  }
}

// Type narrowing with instanceof
class Dog {
  bark() { console.log('Woof!'); }
}

class Cat {
  meow() { console.log('Meow!'); }
}

function makeSound(animal: Dog | Cat) {
  if (animal instanceof Dog) {
    animal.bark(); // TypeScript knows it's a Dog
  } else {
    animal.meow(); // TypeScript knows it's a Cat
  }
}

// Type narrowing with 'in' operator
interface Bird {
  fly(): void;
  layEggs(): void;
}

interface Fish {
  swim(): void;
  layEggs(): void;
}

function move(animal: Bird | Fish) {
  if ('fly' in animal) {
    animal.fly(); // TypeScript knows it's a Bird
  } else {
    animal.swim(); // TypeScript knows it's a Fish
  }
}
```

---

## 🗺️ Mapped Types

### Basic Mapped Types

```typescript
// Make all properties optional
type Partial<T> = {
  [P in keyof T]?: T[P];
};

// Make all properties required
type Required<T> = {
  [P in keyof T]-?: T[P];
};

// Make all properties readonly
type Readonly<T> = {
  readonly [P in keyof T]: T[P];
};

// Example usage
interface User {
  id: number;
  name: string;
  email: string;
  age?: number;
}

type PartialUser = Partial<User>; // All properties optional
type RequiredUser = Required<User>; // All properties required (including age)
type ReadonlyUser = Readonly<User>; // All properties readonly

// Custom mapped type - add prefix to all keys
type Prefixed<T, P extends string> = {
  [K in keyof T as `${P}${string & K}`]: T[K];
};

type PrefixedUser = Prefixed<User, 'user'>;
// Result: { userid: number; username: string; useremail: string; userage?: number }
```

### Advanced Mapped Types

```typescript
// Transform property types
type Stringify<T> = {
  [K in keyof T]: string;
};

type StringUser = Stringify<User>;
// Result: { id: string; name: string; email: string; age: string }

// Filter properties by type
type PickByType<T, U> = {
  [K in keyof T as T[K] extends U ? K : never]: T[K];
};

interface Person {
  name: string;
  age: number;
  isActive: boolean;
  birthDate: Date;
}

type StringProperties = PickByType<Person, string>; // { name: string }
type NumberProperties = PickByType<Person, number>; // { age: number }

// Create getter types
type Getters<T> = {
  [K in keyof T as `get${Capitalize<string & K>}`]: () => T[K];
};

type UserGetters = Getters<User>;
// Result: { getId(): number; getName(): string; getEmail(): string; getAge(): number | undefined }

// Create setter types
type Setters<T> = {
  [K in keyof T as `set${Capitalize<string & K>}`]: (value: T[K]) => void;
};

type UserSetters = Setters<User>;
// Result: { setId(value: number): void; setName(value: string): void; ... }

// Combine getters and setters
type Accessors<T> = Getters<T> & Setters<T>;

class UserClass implements Accessors<User> {
  constructor(
    private id: number,
    private name: string,
    private email: string,
    private age?: number
  ) {}

  getId(): number { return this.id; }
  getName(): string { return this.name; }
  getEmail(): string { return this.email; }
  getAge(): number | undefined { return this.age; }

  setId(value: number): void { this.id = value; }
  setName(value: string): void { this.name = value; }
  setEmail(value: string): void { this.email = value; }
  setAge(value: number | undefined): void { this.age = value; }
}
```

---

## 🔀 Conditional Types

### Basic Conditional Types

```typescript
// Basic conditional type syntax: T extends U ? X : Y
type IsString<T> = T extends string ? true : false;

type Test1 = IsString<string>; // true
type Test2 = IsString<number>; // false

// More complex example
type NonNullable<T> = T extends null | undefined ? never : T;

type SafeString = NonNullable<string | null>; // string
type SafeNumber = NonNullable<number | undefined>; // number

// Function return type extraction
type ReturnType<T> = T extends (...args: any[]) => infer R ? R : any;

function getUserById(id: number): User {
  // Implementation
  return {} as User;
}

type UserReturnType = ReturnType<typeof getUserById>; // User

// Array element type extraction
type ArrayElement<T> = T extends (infer U)[] ? U : never;

type StringArrayElement = ArrayElement<string[]>; // string
type NumberArrayElement = ArrayElement<number[]>; // number
```

### Advanced Conditional Types

```typescript
// Deep readonly type
type DeepReadonly<T> = {
  readonly [P in keyof T]: T[P] extends object ? DeepReadonly<T[P]> : T[P];
};

interface NestedObject {
  user: {
    profile: {
      name: string;
      settings: {
        theme: string;
      };
    };
  };
}

type ReadonlyNested = DeepReadonly<NestedObject>;
// All nested properties are readonly

// Flatten nested types
type Flatten<T> = T extends any[] ? T[number] : T;

type FlatArray = Flatten<string[]>; // string
type FlatString = Flatten<string>; // string

// Promise unwrapping
type Awaited<T> = T extends Promise<infer U> ? Awaited<U> : T;

type PromiseString = Awaited<Promise<string>>; // string
type NestedPromise = Awaited<Promise<Promise<number>>>; // number

// Function argument types
type Parameters<T extends (...args: any) => any> = T extends (...args: infer P) => any ? P : never;

function createUser(name: string, age: number, isActive: boolean): User {
  return {} as User;
}

type CreateUserParams = Parameters<typeof createUser>; // [string, number, boolean]

// Object key filtering
type FilterKeys<T, U> = {
  [K in keyof T]: T[K] extends U ? K : never;
}[keyof T];

type StringKeys = FilterKeys<Person, string>; // "name"
type NumberKeys = FilterKeys<Person, number>; // "age"
```

---

## 📝 Template Literal Types

### Basic Template Literals

```typescript
// Basic template literal types
type Greeting = `Hello, ${string}!`;

const greeting1: Greeting = "Hello, World!"; // ✅
const greeting2: Greeting = "Hello, TypeScript!"; // ✅
// const greeting3: Greeting = "Hi there!"; // ❌

// Combining literal types
type Color = 'red' | 'blue' | 'green';
type Size = 'small' | 'medium' | 'large';

type ColoredSize = `${Color}-${Size}`;
// Result: "red-small" | "red-medium" | "red-large" | "blue-small" | ...

// Event handler types
type EventName = 'click' | 'focus' | 'blur';
type EventHandler = `on${Capitalize<EventName>}`;
// Result: "onClick" | "onFocus" | "onBlur"

// CSS property types
type CSSProperty = 'margin' | 'padding' | 'border';
type CSSDirection = 'top' | 'right' | 'bottom' | 'left';
type CSSDirectionalProperty = `${CSSProperty}-${CSSDirection}`;
// Result: "margin-top" | "margin-right" | ... | "border-left"
```

### Advanced Template Literal Patterns

```typescript
// Database table and column types
type TableName = 'users' | 'posts' | 'comments';
type ColumnName = 'id' | 'name' | 'email' | 'title' | 'content';

type TableColumn = `${TableName}.${ColumnName}`;

// SQL query builder types
type SelectQuery<T extends string> = `SELECT ${T} FROM ${TableName}`;
type WhereClause<T extends string> = `WHERE ${T}`;

type UserQuery = SelectQuery<'name, email'> & WhereClause<'id = ?'>;

// API endpoint types
type HTTPMethod = 'GET' | 'POST' | 'PUT' | 'DELETE';
type APIVersion = 'v1' | 'v2';
type Resource = 'users' | 'posts' | 'comments';

type APIEndpoint = `/api/${APIVersion}/${Resource}`;
type APIOperation = `${HTTPMethod} ${APIEndpoint}`;

// Route parameter extraction
type ExtractRouteParams<T extends string> = 
  T extends `${string}:${infer Param}/${infer Rest}`
    ? { [K in Param | keyof ExtractRouteParams<Rest>]: string }
    : T extends `${string}:${infer Param}`
    ? { [K in Param]: string }
    : {};

type UserRoute = '/users/:userId/posts/:postId';
type RouteParams = ExtractRouteParams<UserRoute>; // { userId: string; postId: string }

// Type-safe environment variables
type EnvPrefix = 'REACT_APP' | 'NEXT_PUBLIC';
type EnvVariable<T extends string> = `${EnvPrefix}_${Uppercase<T>}`;

type AppEnvVars = 
  | EnvVariable<'api_url'>
  | EnvVariable<'app_name'>
  | EnvVariable<'debug_mode'>;
// Result: "REACT_APP_API_URL" | "REACT_APP_APP_NAME" | ...
```

---

## 📦 Modules and Namespaces

### Module Declarations

```typescript
// Ambient module declarations
declare module 'custom-library' {
  export interface Config {
    apiKey: string;
    baseUrl: string;
  }

  export function initialize(config: Config): void;
  export function getData<T>(endpoint: string): Promise<T>;
}

// Using the declared module
import { initialize, getData, Config } from 'custom-library';

const config: Config = {
  apiKey: 'your-api-key',
  baseUrl: 'https://api.example.com'
};

initialize(config);

// Module augmentation
declare module 'express-serve-static-core' {
  interface Request {
    user?: {
      id: number;
      name: string;
    };
  }
}

// Now you can use req.user in Express middleware
import { Request, Response, NextFunction } from 'express';

function authMiddleware(req: Request, res: Response, next: NextFunction) {
  // req.user is now typed
  if (req.user) {
    console.log(`User ${req.user.name} is authenticated`);
  }
  next();
}

// Global augmentation
declare global {
  interface Window {
    myGlobalFunction: (data: any) => void;
  }

  namespace NodeJS {
    interface ProcessEnv {
      NODE_ENV: 'development' | 'production' | 'test';
      DATABASE_URL: string;
      JWT_SECRET: string;
    }
  }
}

// Usage
window.myGlobalFunction({ message: 'Hello' });
console.log(process.env.NODE_ENV); // Typed as 'development' | 'production' | 'test'
```

### Namespaces

```typescript
// Namespace declaration
namespace Geometry {
  export interface Point {
    x: number;
    y: number;
  }

  export interface Rectangle {
    topLeft: Point;
    bottomRight: Point;
  }

  export namespace Circle {
    export interface CircleType {
      center: Point;
      radius: number;
    }

    export function area(circle: CircleType): number {
      return Math.PI * circle.radius ** 2;
    }

    export function circumference(circle: CircleType): number {
      return 2 * Math.PI * circle.radius;
    }
  }

  export function distance(p1: Point, p2: Point): number {
    return Math.sqrt((p2.x - p1.x) ** 2 + (p2.y - p1.y) ** 2);
  }

  export function rectangleArea(rect: Rectangle): number {
    const width = rect.bottomRight.x - rect.topLeft.x;
    const height = rect.bottomRight.y - rect.topLeft.y;
    return width * height;
  }
}

// Usage
const point1: Geometry.Point = { x: 0, y: 0 };
const point2: Geometry.Point = { x: 3, y: 4 };
const distance = Geometry.distance(point1, point2); // 5

const circle: Geometry.Circle.CircleType = {
  center: { x: 0, y: 0 },
  radius: 5
};
const area = Geometry.Circle.area(circle);

// Merging namespaces
namespace Geometry {
  export function midpoint(p1: Point, p2: Point): Point {
    return {
      x: (p1.x + p2.x) / 2,
      y: (p1.y + p2.y) / 2
    };
  }
}

// Now midpoint is available in the Geometry namespace
const mid = Geometry.midpoint(point1, point2);
```

---

## 🎨 Decorators

### Class Decorators

```typescript
// Enable experimental decorators in tsconfig.json:
// "experimentalDecorators": true,
// "emitDecoratorMetadata": true

// Class decorator
function Component<T extends { new(...args: any[]): {} }>(constructor: T) {
  return class extends constructor {
    id = Math.random().toString(36);
    created = new Date();
  };
}

@Component
class UserComponent {
  constructor(public name: string) {}
}

const user = new UserComponent('John');
console.log(user.id); // Random ID
console.log(user.created); // Creation timestamp

// Decorator factory
function Logger(prefix: string) {
  return function<T extends { new(...args: any[]): {} }>(constructor: T) {
    return class extends constructor {
      constructor(...args: any[]) {
        super(...args);
        console.log(`${prefix}: Created instance of ${constructor.name}`);
      }
    };
  };
}

@Logger('DEBUG')
class Service {
  constructor(public name: string) {}
}

const service = new Service('UserService'); // Logs: "DEBUG: Created instance of Service"
```

### Method and Property Decorators

```typescript
// Method decorator for logging
function Log(target: any, propertyName: string, descriptor: PropertyDescriptor) {
  const method = descriptor.value;
  
  descriptor.value = function(...args: any[]) {
    console.log(`Calling ${propertyName} with arguments:`, args);
    const result = method.apply(this, args);
    console.log(`${propertyName} returned:`, result);
    return result;
  };
}

// Property decorator
function MinLength(length: number) {
  return function(target: any, propertyName: string) {
    let value: string;
    
    const getter = () => value;
    const setter = (newVal: string) => {
      if (newVal.length < length) {
        throw new Error(`${propertyName} must be at least ${length} characters`);
      }
      value = newVal;
    };
    
    Object.defineProperty(target, propertyName, {
      get: getter,
      set: setter,
      enumerable: true,
      configurable: true
    });
  };
}

// Parameter decorator
function Required(target: any, propertyName: string, parameterIndex: number) {
  const existingRequiredParameters: number[] = Reflect.getOwnMetadata('required', target, propertyName) || [];
  existingRequiredParameters.push(parameterIndex);
  Reflect.defineMetadata('required', existingRequiredParameters, target, propertyName);
}

function Validate(target: any, propertyName: string, descriptor: PropertyDescriptor) {
  const method = descriptor.value;
  
  descriptor.value = function(...args: any[]) {
    const requiredParameters: number[] = Reflect.getOwnMetadata('required', target, propertyName) || [];
    
    for (const parameterIndex of requiredParameters) {
      if (parameterIndex >= args.length || args[parameterIndex] === undefined || args[parameterIndex] === null) {
        throw new Error(`Parameter at index ${parameterIndex} is required`);
      }
    }
    
    return method.apply(this, args);
  };
}

class UserService {
  @MinLength(3)
  username!: string;

  @Log
  getUserById(id: number): User {
    return { id, name: 'John', email: 'john@example.com' };
  }

  @Validate
  createUser(@Required name: string, @Required email: string, age?: number): User {
    return { id: Date.now(), name, email, age };
  }
}
```

### Advanced Decorator Patterns

```typescript
// Memoization decorator
function Memoize(target: any, propertyName: string, descriptor: PropertyDescriptor) {
  const originalMethod = descriptor.value;
  const cache = new Map();

  descriptor.value = function(...args: any[]) {
    const key = JSON.stringify(args);
    
    if (cache.has(key)) {
      console.log(`Cache hit for ${propertyName}`);
      return cache.get(key);
    }

    const result = originalMethod.apply(this, args);
    cache.set(key, result);
    console.log(`Cache miss for ${propertyName}`);
    return result;
  };

  return descriptor;
}

// Retry decorator
function Retry(attempts: number = 3, delay: number = 1000) {
  return function(target: any, propertyName: string, descriptor: PropertyDescriptor) {
    const originalMethod = descriptor.value;

    descriptor.value = async function(...args: any[]) {
      let lastError: Error;

      for (let i = 0; i < attempts; i++) {
        try {
          return await originalMethod.apply(this, args);
        } catch (error) {
          lastError = error as Error;
          console.log(`Attempt ${i + 1} failed: ${error.message}`);
          
          if (i < attempts - 1) {
            await new Promise(resolve => setTimeout(resolve, delay));
          }
        }
      }

      throw lastError!;
    };

    return descriptor;
  };
}

// Rate limiting decorator
function RateLimit(maxCalls: number, windowMs: number) {
  const calls: number[] = [];

  return function(target: any, propertyName: string, descriptor: PropertyDescriptor) {
    const originalMethod = descriptor.value;

    descriptor.value = function(...args: any[]) {
      const now = Date.now();
      
      // Remove calls outside the window
      while (calls.length > 0 && calls[0] < now - windowMs) {
        calls.shift();
      }

      if (calls.length >= maxCalls) {
        throw new Error(`Rate limit exceeded for ${propertyName}`);
      }

      calls.push(now);
      return originalMethod.apply(this, args);
    };

    return descriptor;
  };
}

class APIService {
  @Memoize
  @Retry(3, 1000)
  async fetchUserData(userId: number): Promise<User> {
    // Simulate API call that might fail
    if (Math.random() < 0.3) {
      throw new Error('Network error');
    }
    
    return { id: userId, name: 'John', email: 'john@example.com' };
  }

  @RateLimit(5, 60000) // 5 calls per minute
  async sendNotification(message: string): Promise<void> {
    console.log(`Sending notification: ${message}`);
  }
}
```

This intermediate TypeScript guide covers advanced type system features, conditional types, template literals, modules, and decorators for building sophisticated type-safe applications.
