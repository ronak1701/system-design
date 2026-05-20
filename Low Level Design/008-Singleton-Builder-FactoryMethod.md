# Creational Design Patterns: Singleton, Builder, and Factory Method

Creational design patterns provide various object creation mechanisms, which increase flexibility and reuse of existing code. This document explores three of the most commonly used creational patterns: **Singleton**, **Builder**, and **Factory Method**.

---

## 1. Singleton Pattern

### Introduction
The **Singleton** pattern ensures that a class has only one instance and provides a global point of access to that instance. It is typically used when exactly one object is needed to coordinate actions across the system.

### Real-World Analogy
Think of a print spooler in an operating system. There is one spooler managing all print jobs. Applications do not create their own spoolers. They submit jobs to the one that exists. If each application ran its own spooler, print jobs would conflict, pages would interleave, and the printer would produce garbage.
The single spooler coordinates everything.

### When to Use
- Managing a connection to a database.
- Creating a global logger.
- Managing application configuration settings.
- Implementing an in-memory cache.

### TypeScript Implementation

In TypeScript, a Singleton is usually implemented by making the constructor private, exposing a static method (`getInstance`) to access the instance, and keeping the instance itself as a static property.

```typescript
// singleton-example.ts

class DatabaseConnection {
  // 1. Static property to hold the single instance
  private static instance: DatabaseConnection;

  // 2. Private constructor prevents direct instantiation using `new`
  private constructor() {
    console.log("Initializing database connection...");
    // Actual connection logic would go here
  }

  // 3. Static method to get the single instance
  public static getInstance(): DatabaseConnection {
    if (!DatabaseConnection.instance) {
      DatabaseConnection.instance = new DatabaseConnection();
    }
    return DatabaseConnection.instance;
  }

  // Example business method
  public query(sql: string): void {
    console.log(`Executing query: ${sql}`);
  }
}

// --- Usage ---

// Client code
const db1 = DatabaseConnection.getInstance(); // Outputs: Initializing database connection...
db1.query("SELECT * FROM users"); // Outputs: Executing query: SELECT * FROM users

const db2 = DatabaseConnection.getInstance(); // Does not output "Initializing database connection..."
db2.query("SELECT * FROM orders"); // Outputs: Executing query: SELECT * FROM orders

// Checking if both variables point to the exact same instance
console.log(db1 === db2); // true

// const db3 = new DatabaseConnection(); // Error: Constructor of class 'DatabaseConnection' is private.
```

### Pros and Cons
**Pros:**
- Guarantees that only one instance of a class exists.
- Provides a global access point to that instance.
- The singleton object is initialized only when it's requested for the first time (lazy initialization).

**Cons:**
- Violates the Single Responsibility Principle (it manages its own creation and its primary business logic).
- Can mask bad design (components might become tightly coupled).
- Makes unit testing difficult, as global state is introduced.

---

## 2. Builder Pattern

### Introduction
The **Builder** pattern separates the construction of a complex object from its representation, allowing the same construction process to create various representations. It is extremely useful when an object requires multiple steps to be initialized or has many optional parameters.

### Real-World Analogy
Think of ordering a custom sandwich at Subway. You don't just ask for a "sandwich" and get a pre-configured one. Instead, you act as a director, instructing the "builder" (the sandwich artist) step-by-step: first the bread, then the meat, cheese, veggies, and finally the sauces. The same step-by-step process can produce entirely different sandwiches depending on the choices made at each step.

### When to Use
- Building complex queries (e.g., Query Builder like Knex.js).
- Creating complex configuration objects.
- Constructing complex UI components step-by-step.
- When an object needs a huge constructor with many optional parameters (Telescoping Constructor Anti-pattern).

### TypeScript Implementation

```typescript
// builder-example.ts

// The complex object we want to build
class UserProfile {
  public firstName!: string;
  public lastName!: string;
  public email!: string;
  public age?: number;
  public address?: string;
  public phoneNumber?: string;

  public displayInfo(): void {
    console.log(`User: ${this.firstName} ${this.lastName}, Email: ${this.email}, Age: ${this.age || 'N/A'}, Address: ${this.address || 'N/A'}`);
  }
}

// The Builder class
class UserProfileBuilder {
  private userProfile: UserProfile;

  constructor(firstName: string, lastName: string, email: string) {
    // Required parameters are passed in the constructor
    this.userProfile = new UserProfile();
    this.userProfile.firstName = firstName;
    this.userProfile.lastName = lastName;
    this.userProfile.email = email;
  }

  // Optional parameters are set via chainable methods
  public setAge(age: number): this {
    this.userProfile.age = age;
    return this; // Returning 'this' enables method chaining
  }

  public setAddress(address: string): this {
    this.userProfile.address = address;
    return this;
  }

  public setPhoneNumber(phoneNumber: string): this {
    this.userProfile.phoneNumber = phoneNumber;
    return this;
  }

  // The final build method that returns the constructed object
  public build(): UserProfile {
    return this.userProfile;
  }
}

// --- Usage ---

// Client code building a complex object step-by-step
const user1 = new UserProfileBuilder("John", "Doe", "john.doe@example.com")
  .setAge(30)
  .setAddress("123 Main St, Cityville")
  .build();

user1.displayInfo(); 
// Outputs: User: John Doe, Email: john.doe@example.com, Age: 30, Address: 123 Main St, Cityville

// Building an object with only required parameters
const user2 = new UserProfileBuilder("Jane", "Smith", "jane.smith@example.com").build();

user2.displayInfo();
// Outputs: User: Jane Smith, Email: jane.smith@example.com, Age: N/A, Address: N/A
```

### Pros and Cons
**Pros:**
- Allows constructing objects step-by-step or deferring construction steps.
- Solves the telescoping constructor problem (avoids long lists of parameters).
- Promotes immutability (the builder can return a fully initialized, immutable object).
- Single Responsibility Principle (isolates complex construction code from the business logic).

**Cons:**
- Increases overall code complexity by requiring the creation of multiple new classes (the Builder itself, sometimes a Director class).

---

## 3. Factory Method Pattern

### Introduction
The **Factory Method** pattern defines an interface for creating an object, but lets subclasses decide which class to instantiate. It defers the instantiation to subclasses, providing a way to encapsulate object creation logic and decouple the client from the concrete classes it needs to instantiate.

### Real-World Analogy
Think of a food delivery platform. You place an order. If the system is designed like a Simple Factory, there’s one centralized kitchen deciding whether to cook pizza, sushi, or burgers.
But with the Factory Method, each restaurant (Pizza Place, Sushi Bar, Burger Joint) has its own kitchen and knows how to prepare its food. The platform just asks the appropriate kitchen to handle it.

### When to Use
- When you don't know beforehand the exact types and dependencies of the objects your code should work with.
- When you want to provide users of your library or framework with a way to extend its internal components.
- Designing a payment gateway where different payment methods (Credit Card, PayPal, Crypto) have different implementations but share a common interface.

### TypeScript Implementation

```typescript
// factory-method-example.ts

// 1. The Product Interface
interface PaymentProcessor {
  processPayment(amount: number): void;
}

// 2. Concrete Products
class CreditCardProcessor implements PaymentProcessor {
  processPayment(amount: number): void {
    console.log(`Processing credit card payment of $${amount}`);
    // Implementation details for Credit Card
  }
}

class PayPalProcessor implements PaymentProcessor {
  processPayment(amount: number): void {
    console.log(`Processing PayPal payment of $${amount}`);
    // Implementation details for PayPal
  }
}

class CryptoProcessor implements PaymentProcessor {
  processPayment(amount: number): void {
    console.log(`Processing Cryptocurrency payment of $${amount}`);
    // Implementation details for Crypto
  }
}

// 3. The Creator (Factory)
// Can be an abstract class with a factory method, or a standard class with a parameterized factory method.
class PaymentProcessorFactory {
  // Parameterized factory method
  public static createProcessor(type: string): PaymentProcessor {
    switch (type.toLowerCase()) {
      case 'creditcard':
        return new CreditCardProcessor();
      case 'paypal':
        return new PayPalProcessor();
      case 'crypto':
        return new CryptoProcessor();
      default:
        throw new Error(`Unsupported payment processor type: ${type}`);
    }
  }
}

// --- Usage ---

// Client code only depends on the interface and the factory.
// It doesn't need to know about the concrete classes (CreditCardProcessor, etc.)

try {
  const processor1 = PaymentProcessorFactory.createProcessor('creditcard');
  processor1.processPayment(150.00); // Outputs: Processing credit card payment of $150

  const processor2 = PaymentProcessorFactory.createProcessor('paypal');
  processor2.processPayment(45.50); // Outputs: Processing PayPal payment of $45.5

  const processor3 = PaymentProcessorFactory.createProcessor('crypto');
  processor3.processPayment(999.99); // Outputs: Processing Cryptocurrency payment of $999.99

  // const processor4 = PaymentProcessorFactory.createProcessor('banktransfer'); // Throws Error
} catch (error) {
  console.error(error.message);
}
```

### Pros and Cons
**Pros:**
- **Decoupling:** Avoids tight coupling between the creator and concrete products.
- **Single Responsibility Principle:** Moves the product creation code into one place, making the code easier to support.
- **Open/Closed Principle:** You can introduce new types of products into the program without breaking existing client code (e.g., adding an `ApplePayProcessor`).

**Cons:**
- The code may become more complicated since you need to introduce a lot of new subclasses to implement the pattern.

---

## Summary
- **Singleton**: When you need exactly *one* instance of an object to coordinate state or resources across an application.
- **Builder**: When creating an object involves multiple complex steps, or when dealing with classes requiring numerous optional arguments.
- **Factory Method**: When you want to delegate the responsibility of object instantiation to a centralized factory, decoupling the client from the concrete implementation classes.
