
# 🚀 Advanced TypeScript

## Table of Contents
- [Type System Deep Dive](#type-system-deep-dive)
- [Advanced Generics](#advanced-generics)
- [Meta-programming](#meta-programming)
- [Compiler API](#compiler-api)
- [Performance Optimization](#performance-optimization)
- [Design Patterns](#design-patterns)
- [Advanced Configuration](#advanced-configuration)
- [Integration Patterns](#integration-patterns)

---

## 🔍 Type System Deep Dive

### Variance and Type Relationships

```typescript
// Covariance, Contravariance, and Invariance

// Arrays are covariant in TypeScript (unsafe but practical)
class Animal {
  name: string = '';
}

class Dog extends Animal {
  breed: string = '';
}

class Cat extends Animal {
  lives: number = 9;
}

// Covariance: Dog[] is assignable to Animal[]
const dogs: Dog[] = [new Dog()];
const animals: Animal[] = dogs; // ✅ Allowed (covariant)

// Function parameters are contravariant
type AnimalHandler = (animal: Animal) => void;
type DogHandler = (dog: Dog) => void;

const handleAnimal: AnimalHandler = (animal) => console.log(animal.name);
const handleDog: DogHandler = handleAnimal; // ✅ Allowed (contravariant)

// Function return types are covariant
type AnimalFactory = () => Animal;
type DogFactory = () => Dog;

const createDog: DogFactory = () => new Dog();
const createAnimal: AnimalFactory = createDog; // ✅ Allowed (covariant)

// Bivariance with method declarations (consider using function properties)
interface EventHandler {
  handle(event: MouseEvent): void; // Bivariant
}

interface StrictEventHandler {
  handle: (event: MouseEvent) => void; // Contravariant
}
```

### Advanced Type Manipulation

```typescript
// Deep manipulation utilities
type DeepPartial<T> = {
  [P in keyof T]?: T[P] extends object ? DeepPartial<T[P]> : T[P];
};

type DeepRequired<T> = {
  [P in keyof T]-?: T[P] extends object ? DeepRequired<T[P]> : T[P];
};

type DeepReadonly<T> = {
  readonly [P in keyof T]: T[P] extends object ? DeepReadonly<T[P]> : T[P];
};

// Recursive type operations
type Paths<T, Prefix extends string = ''> = {
  [K in keyof T]: T[K] extends object 
    ? K extends string
      ? Paths<T[K], `${Prefix}${K}.`> | `${Prefix}${K}`
      : never
    : K extends string
    ? `${Prefix}${K}`
    : never;
}[keyof T];

interface NestedObject {
  user: {
    profile: {
      name: string;
      age: number;
    };
    settings: {
      theme: string;
      notifications: boolean;
    };
  };
  metadata: {
    created: Date;
  };
}

type ObjectPaths = Paths<NestedObject>;
// Result: "user" | "user.profile" | "user.profile.name" | "user.profile.age" | ...

// Get type at path
type GetByPath<T, Path extends string> = 
  Path extends `${infer K}.${infer Rest}`
    ? K extends keyof T
      ? GetByPath<T[K], Rest>
      : never
    : Path extends keyof T
    ? T[Path]
    : never;

type NameType = GetByPath<NestedObject, 'user.profile.name'>; // string
type ThemeType = GetByPath<NestedObject, 'user.settings.theme'>; // string
```

### Higher-Kinded Types Simulation

```typescript
// Simulating Higher-Kinded Types with interfaces
interface HKT {
  readonly _URI: string;
  readonly _A: unknown;
}

interface URItoKind<A> extends HKT {
  readonly _A: A;
}

// Functor type class
interface Functor<F extends HKT> {
  readonly map: <A, B>(fa: F & { _A: A }, f: (a: A) => B) => F & { _A: B };
}

// Option type implementation
interface OptionHKT extends HKT {
  readonly _URI: 'Option';
}

type Option<A> = OptionHKT & { _A: A } & (Some<A> | None);

interface Some<A> {
  readonly _tag: 'Some';
  readonly value: A;
}

interface None {
  readonly _tag: 'None';
}

const some = <A>(value: A): Option<A> => ({ _tag: 'Some', value } as any);
const none: Option<never> = { _tag: 'None' } as any;

// Functor instance for Option
const optionFunctor: Functor<OptionHKT> = {
  map: <A, B>(fa: Option<A>, f: (a: A) => B): Option<B> => {
    switch (fa._tag) {
      case 'Some':
        return some(f(fa.value));
      case 'None':
        return none;
    }
  }
};

// Usage
const result = optionFunctor.map(
  some(5),
  x => x * 2
); // Option<number>
```

---

## 🧬 Advanced Generics

### Generic Constraints and Conditional Types

```typescript
// Advanced generic constraints
interface Lengthwise {
  length: number;
}

function logLength<T extends Lengthwise>(arg: T): T {
  console.log(arg.length);
  return arg;
}

// Using keyof with generics
function getProperty<T, K extends keyof T>(obj: T, key: K): T[K] {
  return obj[key];
}

// Conditional types with generics
type NonFunctionPropertyNames<T> = {
  [K in keyof T]: T[K] extends Function ? never : K;
}[keyof T];

type NonFunctionProperties<T> = Pick<T, NonFunctionPropertyNames<T>>;

interface Example {
  id: number;
  name: string;
  getValue(): number;
  setValue(value: number): void;
}

type ExampleData = NonFunctionProperties<Example>; // { id: number; name: string }

// Recursive generic types
type JsonValue = 
  | string
  | number
  | boolean
  | null
  | JsonValue[]
  | { [key: string]: JsonValue };

type SerializableState<T> = {
  [K in keyof T]: T[K] extends JsonValue 
    ? T[K]
    : T[K] extends object
    ? SerializableState<T[K]>
    : never;
};

// Generic builders
class QueryBuilder<T> {
  private conditions: Array<(item: T) => boolean> = [];
  private sortFn?: (a: T, b: T) => number;
  private limitCount?: number;

  where<K extends keyof T>(
    key: K,
    operator: 'eq' | 'gt' | 'lt' | 'gte' | 'lte',
    value: T[K]
  ): QueryBuilder<T> {
    this.conditions.push(item => {
      switch (operator) {
        case 'eq': return item[key] === value;
        case 'gt': return item[key] > value;
        case 'lt': return item[key] < value;
        case 'gte': return item[key] >= value;
        case 'lte': return item[key] <= value;
      }
    });
    return this;
  }

  orderBy<K extends keyof T>(key: K, direction: 'asc' | 'desc' = 'asc'): QueryBuilder<T> {
    this.sortFn = (a, b) => {
      const aVal = a[key];
      const bVal = b[key];
      const comparison = aVal > bVal ? 1 : aVal < bVal ? -1 : 0;
      return direction === 'asc' ? comparison : -comparison;
    };
    return this;
  }

  limit(count: number): QueryBuilder<T> {
    this.limitCount = count;
    return this;
  }

  execute(data: T[]): T[] {
    let result = data.filter(item => 
      this.conditions.every(condition => condition(item))
    );

    if (this.sortFn) {
      result = result.sort(this.sortFn);
    }

    if (this.limitCount !== undefined) {
      result = result.slice(0, this.limitCount);
    }

    return result;
  }
}

// Usage
interface User {
  id: number;
  name: string;
  age: number;
  isActive: boolean;
}

const users: User[] = [
  { id: 1, name: 'Alice', age: 30, isActive: true },
  { id: 2, name: 'Bob', age: 25, isActive: false },
  { id: 3, name: 'Charlie', age: 35, isActive: true },
];

const result = new QueryBuilder<User>()
  .where('isActive', 'eq', true)
  .where('age', 'gte', 30)
  .orderBy('age', 'desc')
  .limit(1)
  .execute(users);
```

### Distributive Conditional Types

```typescript
// Distributive conditional types work on union types
type ToArray<T> = T extends any ? T[] : never;

type StrArrOrNumArr = ToArray<string | number>; // string[] | number[]

// Non-distributive version
type ToArrayNonDistributive<T> = [T] extends [any] ? T[] : never;

type NonDistResult = ToArrayNonDistributive<string | number>; // (string | number)[]

// Practical example: Event system
type EventMap = {
  'user:login': { userId: string; timestamp: Date };
  'user:logout': { userId: string };
  'post:created': { postId: string; authorId: string; title: string };
  'post:deleted': { postId: string };
};

type EventHandler<T extends keyof EventMap> = (data: EventMap[T]) => void;

type AllEventHandlers = {
  [K in keyof EventMap]: EventHandler<K>;
};

class EventEmitter {
  private handlers: Partial<AllEventHandlers> = {};

  on<T extends keyof EventMap>(event: T, handler: EventHandler<T>): void {
    this.handlers[event] = handler as any;
  }

  emit<T extends keyof EventMap>(event: T, data: EventMap[T]): void {
    const handler = this.handlers[event];
    if (handler) {
      (handler as EventHandler<T>)(data);
    }
  }
}

// Usage with full type safety
const emitter = new EventEmitter();

emitter.on('user:login', (data) => {
  console.log(`User ${data.userId} logged in at ${data.timestamp}`);
});

emitter.emit('user:login', {
  userId: '123',
  timestamp: new Date()
});
```

---

## 🔮 Meta-programming

### Reflection and Runtime Type Information

```typescript
// Using reflect-metadata for runtime type information
import 'reflect-metadata';

// Decorator for dependency injection
function Injectable<T extends { new(...args: any[]): {} }>(constructor: T) {
  const paramTypes = Reflect.getMetadata('design:paramtypes', constructor) || [];
  Reflect.defineMetadata('injectable', true, constructor);
  Reflect.defineMetadata('paramtypes', paramTypes, constructor);
  return constructor;
}

function Inject(token: string) {
  return function(target: any, propertyKey: string | symbol | undefined, parameterIndex: number) {
    const existingTokens = Reflect.getMetadata('inject:tokens', target) || {};
    existingTokens[parameterIndex] = token;
    Reflect.defineMetadata('inject:tokens', existingTokens, target);
  };
}

// Simple DI container
class Container {
  private services = new Map<string, any>();
  private factories = new Map<string, () => any>();

  register<T>(token: string, factory: () => T): void {
    this.factories.set(token, factory);
  }

  get<T>(token: string): T {
    if (this.services.has(token)) {
      return this.services.get(token);
    }

    const factory = this.factories.get(token);
    if (!factory) {
      throw new Error(`Service ${token} not found`);
    }

    const instance = factory();
    this.services.set(token, instance);
    return instance;
  }

  create<T extends { new(...args: any[]): {} }>(constructor: T): InstanceType<T> {
    const paramTypes = Reflect.getMetadata('paramtypes', constructor) || [];
    const tokens = Reflect.getMetadata('inject:tokens', constructor) || {};

    const args = paramTypes.map((paramType: any, index: number) => {
      const token = tokens[index];
      if (token) {
        return this.get(token);
      }
      
      // Try to resolve by constructor name
      const serviceName = paramType.name.toLowerCase();
      try {
        return this.get(serviceName);
      } catch {
        throw new Error(`Cannot resolve parameter ${index} of ${constructor.name}`);
      }
    });

    return new constructor(...args);
  }
}

// Usage
interface ILogger {
  log(message: string): void;
}

@Injectable
class ConsoleLogger implements ILogger {
  log(message: string): void {
    console.log(`[LOG] ${message}`);
  }
}

interface IUserRepository {
  findById(id: string): Promise<User | null>;
}

@Injectable
class DatabaseUserRepository implements IUserRepository {
  constructor(@Inject('logger') private logger: ILogger) {}

  async findById(id: string): Promise<User | null> {
    this.logger.log(`Finding user with ID: ${id}`);
    // Implementation
    return null;
  }
}

@Injectable
class UserService {
  constructor(
    @Inject('userRepository') private userRepository: IUserRepository,
    @Inject('logger') private logger: ILogger
  ) {}

  async getUser(id: string): Promise<User | null> {
    this.logger.log(`Getting user: ${id}`);
    return this.userRepository.findById(id);
  }
}

// Setup
const container = new Container();
container.register('logger', () => new ConsoleLogger());
container.register('userRepository', () => container.create(DatabaseUserRepository));
container.register('userService', () => container.create(UserService));

const userService = container.get<UserService>('userService');
```

### Type-Level Computation

```typescript
// Type-level arithmetic
type Tuple<T extends ReadonlyArray<any>> = T;
type Length<T extends ReadonlyArray<any>> = T['length'];

// Addition at type level
type Add<A extends number, B extends number> = Length<[
  ...Tuple<Expand<A>>,
  ...Tuple<Expand<B>>
]>;

type Expand<N extends number, Counter extends ReadonlyArray<any> = []> = 
  Counter['length'] extends N ? Counter : Expand<N, [...Counter, any]>;

type Sum = Add<3, 4>; // 7

// Comparison at type level
type IsEqual<A, B> = A extends B ? (B extends A ? true : false) : false;
type IsGreater<A extends number, B extends number> = 
  IsEqual<A, B> extends true 
    ? false 
    : Expand<A> extends [...Expand<B>, ...any[]] 
    ? true 
    : false;

// Type-level list operations
type Head<T extends ReadonlyArray<any>> = T extends readonly [infer H, ...any[]] ? H : never;
type Tail<T extends ReadonlyArray<any>> = T extends readonly [any, ...infer T] ? T : [];
type Last<T extends ReadonlyArray<any>> = T extends readonly [...any[], infer L] ? L : never;

type Reverse<T extends ReadonlyArray<any>, Acc extends ReadonlyArray<any> = []> = 
  T extends readonly [infer H, ...infer Rest] 
    ? Reverse<Rest, [H, ...Acc]>
    : Acc;

type TestReverse = Reverse<[1, 2, 3, 4]>; // [4, 3, 2, 1]

// Type-level string operations
type Split<S extends string, D extends string> = 
  S extends `${infer T}${D}${infer U}` ? [T, ...Split<U, D>] : [S];

type Join<T extends ReadonlyArray<string>, D extends string> = 
  T extends readonly [infer F, ...infer R] 
    ? F extends string
      ? R extends ReadonlyArray<string>
        ? R['length'] extends 0
          ? F
          : `${F}${D}${Join<R, D>}`
        : never
      : never
    : '';

type TestSplit = Split<'a,b,c', ','>; // ['a', 'b', 'c']
type TestJoin = Join<['a', 'b', 'c'], '-'>; // 'a-b-c'
```

---

## 🔧 Compiler API

### Custom Transformers

```typescript
import * as ts from 'typescript';

// Custom transformer to add runtime type checks
function createRuntimeTypeChecker(): ts.TransformerFactory<ts.SourceFile> {
  return (context: ts.TransformationContext) => {
    return (sourceFile: ts.SourceFile) => {
      function visit(node: ts.Node): ts.Node {
        // Transform function declarations to add runtime type checks
        if (ts.isFunctionDeclaration(node) && node.parameters.length > 0) {
          const typeChecks = node.parameters.map((param, index) => {
            if (param.type && ts.isTypeReferenceNode(param.type)) {
              const typeName = param.type.typeName.getText();
              return ts.factory.createIfStatement(
                ts.factory.createBinaryExpression(
                  ts.factory.createTypeOfExpression(
                    ts.factory.createIdentifier(`arguments[${index}]`)
                  ),
                  ts.SyntaxKind.ExclamationEqualsEqualsToken,
                  ts.factory.createStringLiteral(typeName.toLowerCase())
                ),
                ts.factory.createThrowStatement(
                  ts.factory.createNewExpression(
                    ts.factory.createIdentifier('TypeError'),
                    undefined,
                    [ts.factory.createStringLiteral(
                      `Expected ${param.name?.getText()} to be of type ${typeName}`
                    )]
                  )
                )
              );
            }
            return undefined;
          }).filter(Boolean) as ts.Statement[];

          if (typeChecks.length > 0) {
            const newBody = ts.factory.createBlock([
              ...typeChecks,
              ...(node.body?.statements || [])
            ]);

            return ts.factory.updateFunctionDeclaration(
              node,
              node.decorators,
              node.modifiers,
              node.asteriskToken,
              node.name,
              node.typeParameters,
              node.parameters,
              node.type,
              newBody
            );
          }
        }

        return ts.visitEachChild(node, visit, context);
      }

      return ts.visitNode(sourceFile, visit);
    };
  };
}

// Program analysis utilities
function analyzeProgram(fileNames: string[], options: ts.CompilerOptions) {
  const program = ts.createProgram(fileNames, options);
  const checker = program.getTypeChecker();

  const diagnostics = ts.getPreEmitDiagnostics(program);
  
  // Analyze each source file
  for (const sourceFile of program.getSourceFiles()) {
    if (sourceFile.isDeclarationFile) continue;

    ts.forEachChild(sourceFile, visit);
  }

  function visit(node: ts.Node) {
    // Analyze function declarations
    if (ts.isFunctionDeclaration(node)) {
      const symbol = checker.getSymbolAtLocation(node.name!);
      if (symbol) {
        const type = checker.getTypeOfSymbolAtLocation(symbol, node);
        const signature = checker.getSignatureFromDeclaration(node);
        
        console.log(`Function: ${symbol.name}`);
        console.log(`Type: ${checker.typeToString(type)}`);
        
        if (signature) {
          console.log(`Parameters: ${signature.parameters.length}`);
          signature.parameters.forEach((param, index) => {
            const paramType = checker.getTypeOfSymbolAtLocation(param, node);
            console.log(`  ${param.name}: ${checker.typeToString(paramType)}`);
          });
        }
      }
    }

    ts.forEachChild(node, visit);
  }

  return {
    diagnostics,
    checker,
    program
  };
}

// Code generation utilities
function generateTypeGuards(interfaces: ts.InterfaceDeclaration[], checker: ts.TypeChecker) {
  return interfaces.map(interfaceNode => {
    const symbol = checker.getSymbolAtLocation(interfaceNode.name);
    if (!symbol) return '';

    const type = checker.getTypeOfSymbolAtLocation(symbol, interfaceNode);
    const properties = type.getProperties();

    const checks = properties.map(prop => {
      const propType = checker.getTypeOfSymbolAtLocation(prop, interfaceNode);
      const typeString = checker.typeToString(propType);
      
      return `  typeof obj.${prop.name} === '${typeString.toLowerCase()}'`;
    }).join(' &&\n');

    return `
export function is${symbol.name}(obj: any): obj is ${symbol.name} {
  return obj !== null &&
    typeof obj === 'object' &&
${checks};
}`;
  }).join('\n\n');
}
```

---

## ⚡ Performance Optimization

### Type-Level Performance

```typescript
// Optimized utility types to reduce compilation time

// Avoid deep recursion - use iterative approach when possible
type OptimizedPick<T, K extends keyof T> = {
  [P in K]: T[P];
};

// Use distributive conditional types carefully
type EfficientExtract<T, U> = T extends U ? T : never;

// Prefer mapped types over conditional types when possible
type MapToString<T> = {
  [K in keyof T]: string;
};

// Cache expensive type computations
type CachedTypeCache = {
  [K: string]: any;
};

// Use module augmentation instead of global types when possible
declare module './types' {
  interface UserExtensions {
    customProperty: string;
  }
}

// Optimize generic constraints
interface OptimizedConstraint<T extends Record<string, any>> {
  data: T;
  keys: keyof T;
}

// Instead of complex conditional types, use function overloads
function processData(data: string): string;
function processData(data: number): number;
function processData(data: boolean): boolean;
function processData(data: any): any {
  return data;
}
```

### Runtime Performance Patterns

```typescript
// Object pooling for frequently created objects
class ObjectPool<T> {
  private pool: T[] = [];
  private createFn: () => T;
  private resetFn: (obj: T) => void;

  constructor(createFn: () => T, resetFn: (obj: T) => void, initialSize = 10) {
    this.createFn = createFn;
    this.resetFn = resetFn;
    
    for (let i = 0; i < initialSize; i++) {
      this.pool.push(createFn());
    }
  }

  acquire(): T {
    const obj = this.pool.pop();
    if (obj) {
      return obj;
    }
    return this.createFn();
  }

  release(obj: T): void {
    this.resetFn(obj);
    this.pool.push(obj);
  }
}

// Memoization with type safety
function memoize<Args extends readonly any[], Return>(
  fn: (...args: Args) => Return,
  keyFn?: (...args: Args) => string
): (...args: Args) => Return {
  const cache = new Map<string, Return>();
  
  return (...args: Args): Return => {
    const key = keyFn ? keyFn(...args) : JSON.stringify(args);
    
    if (cache.has(key)) {
      return cache.get(key)!;
    }
    
    const result = fn(...args);
    cache.set(key, result);
    return result;
  };
}

// Lazy evaluation
class Lazy<T> {
  private value?: T;
  private computed = false;
  
  constructor(private factory: () => T) {}
  
  get(): T {
    if (!this.computed) {
      this.value = this.factory();
      this.computed = true;
    }
    return this.value!;
  }
  
  isComputed(): boolean {
    return this.computed;
  }
}

// Type-safe event emitter with performance optimizations
interface TypedEventMap {
  [event: string]: any[];
}

class TypedEventEmitter<TEventMap extends TypedEventMap> {
  private listeners = new Map<keyof TEventMap, Array<(...args: any[]) => void>>();
  private onceListeners = new Map<keyof TEventMap, Array<(...args: any[]) => void>>();

  on<TEvent extends keyof TEventMap>(
    event: TEvent,
    listener: (...args: TEventMap[TEvent]) => void
  ): this {
    if (!this.listeners.has(event)) {
      this.listeners.set(event, []);
    }
    this.listeners.get(event)!.push(listener);
    return this;
  }

  once<TEvent extends keyof TEventMap>(
    event: TEvent,
    listener: (...args: TEventMap[TEvent]) => void
  ): this {
    if (!this.onceListeners.has(event)) {
      this.onceListeners.set(event, []);
    }
    this.onceListeners.get(event)!.push(listener);
    return this;
  }

  emit<TEvent extends keyof TEventMap>(
    event: TEvent,
    ...args: TEventMap[TEvent]
  ): boolean {
    let hasListeners = false;

    // Regular listeners
    const listeners = this.listeners.get(event);
    if (listeners && listeners.length > 0) {
      hasListeners = true;
      for (const listener of listeners) {
        listener(...args);
      }
    }

    // Once listeners
    const onceListeners = this.onceListeners.get(event);
    if (onceListeners && onceListeners.length > 0) {
      hasListeners = true;
      for (const listener of onceListeners) {
        listener(...args);
      }
      this.onceListeners.delete(event);
    }

    return hasListeners;
  }

  off<TEvent extends keyof TEventMap>(
    event: TEvent,
    listener: (...args: TEventMap[TEvent]) => void
  ): this {
    const listeners = this.listeners.get(event);
    if (listeners) {
      const index = listeners.indexOf(listener);
      if (index !== -1) {
        listeners.splice(index, 1);
      }
    }
    return this;
  }
}

// Usage
interface AppEvents {
  'user:login': [userId: string, timestamp: Date];
  'user:logout': [userId: string];
  'data:updated': [data: any];
}

const emitter = new TypedEventEmitter<AppEvents>();

emitter.on('user:login', (userId, timestamp) => {
  // userId and timestamp are properly typed
  console.log(`User ${userId} logged in at ${timestamp}`);
});
```

This advanced TypeScript guide covers deep type system concepts, meta-programming, compiler API usage, and performance optimization techniques for building sophisticated TypeScript applications.
