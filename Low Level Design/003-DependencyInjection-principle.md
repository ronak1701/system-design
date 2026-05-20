# Dependency Injection (DI) Pattern in TypeScript

## Introduction
While the Dependency Inversion Principle (DIP from SOLID) provides the architectural theory that high-level modules should not depend on low-level modules, **Dependency Injection (DI)** is the highly practical design pattern used to implement this theory. DI is a technique where an object receives other objects that it depends on, rather than instantiating them itself.

---

## The Core Concept
**"Don't call us, we'll call you." (The Hollywood Principle)**

### Beginner Explanation
Imagine you want to drive a car. If you had to personally build the engine, attach the wheels, and wire the electronics every single time you wanted to go for a drive, that would be exhausting and completely inflexible. Instead, a "factory" builds the car and "injects" the engine into it. You just jump in and drive. 
In code, if a `UserService` needs a `Database` to save users, the `UserService` shouldn't create the `Database` internally using `new Database()`. Instead, you should provide (inject) the completely built database object into the `UserService`.

### ❌ Bad Example: Tightly Coupled Code (No DI)
The `OrderService` heavily depends on creating a `MySQLDatabase` instance inside its own constructor. If we decide to switch to a `PostgreSQLDatabase` or mock the database for testing, we are forced to rewrite the `OrderService` code.
```typescript
class MySQLDatabase {
  save(data: any) {
    console.log("Saving order to MySQL...");
  }
}

class OrderService {
  private db: MySQLDatabase;

  constructor() {
    // Bad: Hardcoding the dependency creation!
    this.db = new MySQLDatabase();
  }

  createOrder(order: any) {
    this.db.save(order);
  }
}
```

### ✅ Good Example: Constructor Injection (Using DI)
We "inject" the dependency through the constructor. The `OrderService` no longer cares *how* the database is created, only that it gets one that fulfills its requirements defined by an interface.
```typescript
interface IDatabase {
  save(data: any): void;
}

class MySQLDatabase implements IDatabase {
  save(data: any) {
    console.log("Saving order to MySQL...");
  }
}

class MongoDBDatabase implements IDatabase {
  save(data: any) {
    console.log("Saving order to MongoDB...");
  }
}

class OrderService {
  // Good: The dependency is "injected" from the outside!
  constructor(private db: IDatabase) {}

  createOrder(order: any) {
    this.db.save(order);
  }
}

// Usage (Wiring things up at a higher "composition root" level)
const mySqlDb = new MySQLDatabase();
const mongoDb = new MongoDBDatabase();

// We effortlessly swap out the database without touching OrderService logic
const serviceForWeb = new OrderService(mySqlDb);
const serviceForMobile = new OrderService(mongoDb);
```

### 🧠 Advanced Perspective: Testing and IoC Containers
In large enterprise applications, wiring up thousands of dependencies manually like `new OrderService(new Database(new Logger(), new Config()))` becomes incredibly tedious. This is where **Inversion of Control (IoC) Containers** come in. Popular Node.js frameworks like NestJS, or libraries like InversifyJS, automatically manage the creation, resolution, and injection of these objects behind the scenes. 

Furthermore, DI is the absolute bedrock of robust unit testing. Because `OrderService` expects an abstraction, you can effortlessly inject a `MockDatabase` during tests. This guarantees that you are only testing the pure logic of the `OrderService`, rather than executing slow, stateful database queries.

---

## Types of Dependency Injection

Dependency injection typically comes in three distinct flavors based on *how* the dependency is handed to the client class.

### 1. Constructor Injection (Most Common & Highly Recommended)
Dependencies are provided strictly through the class constructor. This pattern is globally recommended because it ensures the object is fully initialized, structurally valid, and completely ready to use immediately upon instantiation.
```typescript
class Logger {
  log(message: string) { console.log(message); }
}

class PaymentProcessor {
  // Constructor Injection
  constructor(private logger: Logger) {}

  process() {
    this.logger.log("Payment processed smoothly.");
  }
}
```

### 2. Setter / Property Injection
Dependencies are assigned via public setter methods or directly onto properties. This technique is primarily useful for *optional* dependencies, or resolving complex circular dependency issues (though circular dependencies usually hint at a larger design flaw).
```typescript
class NotificationService {
  private logger?: Logger; // Optional dependency

  // Setter Injection
  setLogger(logger: Logger) {
    this.logger = logger;
  }

  notify() {
    if (this.logger) {
      this.logger.log("Notification sent successfully.");
    }
  }
}
```

### 3. Interface / Method Injection
The dependency is injected directly into the specific method that requires it, rather than being stored persistently as a class-level property.
```typescript
class ReportGenerator {
  // Method Injection
  generateAndLog(data: any, logger: Logger) {
    console.log("Generating report for:", data);
    logger.log("Report generation finished.");
  }
}
```

---

## Summary Cheat Sheet

| Injection Type | When to Use | Advantages |
| :--- | :--- | :--- |
| **Constructor Injection** | Best default approach. Used for all mandatory dependencies. | Guarantees class is fully functional when created. Promotes immutable dependencies. |
| **Setter Injection** | Framework specifics or entirely optional dependencies. | Flexible; allows swapping dependencies dynamically at runtime after object creation. |
| **Method Injection** | When a dependency is only needed temporarily for a single, specific operation. | Prevents polluting the class state with dependencies needed by only a fraction of its methods. |
