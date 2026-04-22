# SOLID Principles in TypeScript: From Beginner to Advanced

## Introduction
The SOLID principles are a set of five design principles in object-oriented programming (OOP) intended to make software designs more understandable, flexible, and maintainable. Coined by Robert C. Martin (Uncle Bob), these principles help developers build robust systems that are easy to scale and refactor over time.

---

## 1. Single Responsibility Principle (SRP)
**"A class should have one, and only one, reason to change."**

### Beginner Explanation
Imagine a Swiss Army knife. It's useful, but if the scissors break, you might have to buy a whole new knife. Now imagine having a separate pair of scissors. If they break, you just replace the scissors. In programming, a class should do one specific job. If it does too many things (managing database, sending emails, processing payments), then changing one part might accidentally break another.

### ❌ Bad Example: The "God" Class
This class handles user creation, database saving, and email notifications. It has *multiple* reasons to change.
```typescript
class UserService {
  // Responsibility 1: User logic
  createUser(name: string, email: string) {
    const user = { name, email };
    this.saveToDatabase(user);
    this.sendWelcomeEmail(user);
  }

  // Responsibility 2: Database logic
  saveToDatabase(user: any) {
    console.log(`Saving ${user.name} to DB`);
  }

  // Responsibility 3: Notification logic
  sendWelcomeEmail(user: any) {
    console.log(`Sending email to ${user.email}`);
  }
}
```

### ✅ Good Example: Segregated Responsibilities
We split the responsibilities into smaller, focused classes.
```typescript
class UserRepository {
  save(user: any) {
    console.log(`Saving ${user.name} to DB`);
  }
}

class EmailService {
  sendWelcomeEmail(email: string) {
    console.log(`Sending email to ${email}`);
  }
}

class UserService {
  constructor(
    private userRepository: UserRepository,
    private emailService: EmailService
  ) {}

  createUser(name: string, email: string) {
    const user = { name, email };
    this.userRepository.save(user);
    this.emailService.sendWelcomeEmail(user.email);
  }
}
```

### 🧠 Advanced Perspective
SRP is not just about avoiding large classes; it's about **cohesion** and **domain boundaries**. A "reason to change" directly correlates to the business actors relying on that module. For instance, if the Database Administrator requests a schema change, the `UserRepository` changes. If the Marketing team requests a change to the welcome email, the `EmailService` changes. If one class handles both, those two independent business actors are dangerously coupled in the codebase.

---

## 2. Open/Closed Principle (OCP)
**"Software entities (classes, modules, functions, etc.) should be open for extension, but closed for modification."**

### Beginner Explanation
Think of a standard electrical outlet. If you buy a new TV, you don't rewire your house to plug it in. You just plug the new TV into the existing outlet. The outlet is *closed for modification* (you don't tear down the wall), but *open for extension* (you can plug in an infinite variety of appliances). 
In code, you should be able to add new features without changing existing, well-tested code.

### ❌ Bad Example: Modifying code for new features
Every time we add a new payment method, we have to modify the `PaymentProcessor` class. This risks breaking existing payment methods.
```typescript
class PaymentProcessor {
  processPayment(amount: number, type: 'credit_card' | 'paypal' | 'crypto') {
    if (type === 'credit_card') {
      console.log(`Processing credit card payment of ${amount}`);
    } else if (type === 'paypal') {
      console.log(`Processing PayPal payment of ${amount}`);
    } else if (type === 'crypto') {
      console.log(`Processing Crypto payment of ${amount}`);
    }
  }
}
```

### ✅ Good Example: Extending behavior via Interfaces
We use an interface to define the contract, and create new classes for new payment methods. The `PaymentProcessor` works with *any* class that implements the interface.
```typescript
interface IPaymentMethod {
  pay(amount: number): void;
}

class CreditCardPayment implements IPaymentMethod {
  pay(amount: number) {
    console.log(`Processing credit card payment of ${amount}`);
  }
}

class PayPalPayment implements IPaymentMethod {
  pay(amount: number) {
    console.log(`Processing PayPal payment of ${amount}`);
  }
}

// We can add Crypto anytime without modifying PaymentProcessor!
class CryptoPayment implements IPaymentMethod {
  pay(amount: number) {
    console.log(`Processing Crypto payment of ${amount}`);
  }
}

class PaymentProcessor {
  process(amount: number, paymentMethod: IPaymentMethod) {
    paymentMethod.pay(amount);
  }
}
```

### 🧠 Advanced Perspective
OCP is deeply related to **Polymorphism** and the **Strategy Pattern**. By relying on abstractions (interfaces/abstract classes) rather than concrete implementations, you invert the dependencies. This forms the foundation of plugin architectures. The key abstraction should represent the *axis of variation* (in the example above, the axis of variation is the payment method). Correctly predicting the axis of variation is difficult; over-engineering OCP can lead to unnecessary abstraction and complexity, so it is best applied when a module is known to require frequent changes of a similar nature.

---

## 3. Liskov Substitution Principle (LSP)
**"Objects of a superclass shall be replaceable with objects of its subclasses without breaking the application."**

### Beginner Explanation
If it looks like a duck, quacks like a duck, but needs batteries, you probably have the wrong abstraction. 
If class `B` inherits from class `A`, you should be able to use `B` anywhere `A` is expected, and the program should still run perfectly. A subclass should enhance, not break or restrict, the behavior of its parent class.

### ❌ Bad Example: The Square/Rectangle Problem
A square is mathematically a rectangle, but in OOP, setting the width of a square implies changing its height too, which violates the expectations of a generic Rectangle.
```typescript
class Rectangle {
  protected width: number = 0;
  protected height: number = 0;

  setWidth(width: number) { this.width = width; }
  setHeight(height: number) { this.height = height; }
  getArea() { return this.width * this.height; }
}

class Square extends Rectangle {
  // Violates LSP! Modifying how properties work.
  setWidth(width: number) {
    this.width = width;
    this.height = width;
  }
  setHeight(height: number) {
    this.width = height;
    this.height = height;
  }
}

// Function expecting a Rectangle breaks when given a Square
function calculateArea(rect: Rectangle) {
  rect.setWidth(4);
  rect.setHeight(5);
  // Expected: 20
  // Actual for Square: 25 (because setHeight(5) forced width to 5)
  console.log(`Area: ${rect.getArea()}`); 
}
```

### ✅ Good Example: Designing by Contract
Instead of forcing a flawed inheritance, we design abstractions based on shared behaviors.
```typescript
interface IShape {
  getArea(): number;
}

class Rectangle implements IShape {
  constructor(private width: number, private height: number) {}
  getArea() { return this.width * this.height; }
}

class Square implements IShape {
  constructor(private side: number) {}
  getArea() { return this.side * this.side; }
}

function calculateArea(shape: IShape) {
  // We strictly rely on the contract (getArea) without tampering with dimensions
  console.log(`Area: ${shape.getArea()}`);
}
```

### 🧠 Advanced Perspective
LSP is about enforcing **Contracts (Design by Contract)**. Subclasses must maintain the contracts of relationships expected by clients using the superclass. 
Specifically, **preconditions** cannot be strengthened in a subtype (you can't require more stringent inputs than the parent), and **postconditions** cannot be weakened (you can't guarantee less than the parent). Violations of LSP often manifest as heavy use of `instanceof` checks (type switching) or throwing `NotImplementedException` in overridden methods.

---

## 4. Interface Segregation Principle (ISP)
**"No client should be forced to depend on methods it does not use."**

### Beginner Explanation
Imagine going to a restaurant and the menu forces you to order a combo meal involving a burger, fries, cola, and a toy, but you only wanted a burger. The restaurant should separate the items so you can order just what you want. 
In code, a large, "fat" interface should be split into smaller, more specific ones so classes don't have to implement methods they don't need.

### ❌ Bad Example: Fat Interfaces
A worker might be a human or a robot. A robot doesn't need to eat, but the interface forces it.
```typescript
interface IWorker {
  work(): void;
  eat(): void;
  sleep(): void;
}

class HumanWorker implements IWorker {
  work() { console.log('Working...'); }
  eat() { console.log('Eating lunch...'); }
  sleep() { console.log('Sleeping...'); }
}

class RobotWorker implements IWorker {
  work() { console.log('Working 24/7...'); }
  eat() { 
    // Violates ISP: Robot is forced to implement this!
    throw new Error("Robots don't eat"); 
  }
  sleep() { 
    // Violates ISP
    throw new Error("Robots don't sleep"); 
  }
}
```

### ✅ Good Example: Segregated Interfaces
We break down the fat interface into distinct roles.
```typescript
interface IWorkable {
  work(): void;
}

interface IFeedable {
  eat(): void;
}

interface ISleepable {
  sleep(): void;
}

// Human implements all needed interfaces
class HumanWorker implements IWorkable, IFeedable, ISleepable {
  work() { console.log('Working...'); }
  eat() { console.log('Eating lunch...'); }
  sleep() { console.log('Sleeping...'); }
}

// Robot only implements what it actually does
class RobotWorker implements IWorkable {
  work() { console.log('Working 24/7...'); }
}
```

### 🧠 Advanced Perspective
ISP is essentially SRP applied to Interfaces. It ensures that system components are decoupled. When an interface has too many methods, any change to one method might necessitate recompilation or updates in numerous classes that implement the interface, even if they don't use that specific method. By keeping interfaces granular (often acting as **Role Interfaces**), we improve modularity and make mocking in unit tests vastly simpler.

---

## 5. Dependency Inversion Principle (DIP)
**"High-level modules should not depend on low-level modules. Both should depend on abstractions (e.g., interfaces). Abstractions should not depend on details. Details (concrete implementations) should depend on abstractions."**

### Beginner Explanation
Don't solder your lamp straight to the power grid. Use a plug and a socket. 
High-level policy (your lamp) shouldn't be hardcoded to low-level details (the type of wall wiring). They should both depend on an abstraction (the standard electrical socket). In code, instead of a class instantiating its dependencies directly, it should request them as parameters usually done via Dependency Injection.

### ❌ Bad Example: Tightly Coupled Code
`Store` directly relies on the concrete `StripeAPI` class. If we want to switch to PayPal, we have to rewrite the `Store` class.
```typescript
class StripeAPI {
  charge(amount: number) {
    console.log(`Charged $${amount} via Stripe.`);
  }
}

class Store {
  private paymentAPI = new StripeAPI(); // Tight coupling!

  purchaseBike() {
    this.paymentAPI.charge(200);
  }
}
```

### ✅ Good Example: Dependency Inversion
The `Store` now depends on an interface (`IPaymentGateway`), not a specific implementation. The low-level module (`StripeAPI`) also implements this abstraction.
```typescript
interface IPaymentGateway {
  charge(amount: number): void;
}

class StripeAPI implements IPaymentGateway {
  charge(amount: number) {
    console.log(`Charged $${amount} via Stripe.`);
  }
}

class PayPalAPI implements IPaymentGateway {
  charge(amount: number) {
    console.log(`Charged $${amount} via PayPal.`);
  }
}

class Store {
  // Store relies on abstraction, injected via constructor
  constructor(private paymentGateway: IPaymentGateway) {}

  purchaseBike() {
    this.paymentGateway.charge(200);
  }
}

// Wiring it up at the "composition root"
const stripe = new StripeAPI();
const paypal = new PayPalAPI();

const store1 = new Store(stripe);
const store2 = new Store(paypal); // Easy to swap out!
```

### 🧠 Advanced Perspective
DIP is the core principle behind enterprise frameworks like Angular, NestJS, and Spring. It relies heavily on **Inversion of Control (IoC)** containers. By adhering to DIP, you enable architectural boundaries (like Hexagonal Architecture or Clean Architecture), where your core business logic (high-level modules) is utterly unaware of the delivery mechanisms, HTTP frameworks, or databases (low-level modules). This makes the core logic highly testable, framework-agnostic, and independent of external infrastructure changes.

---

## Summary Cheat Sheet

| Principle | Short Summary | Keyword |
| :--- | :--- | :--- |
| **S**RP | One class = One job. | Cohesion |
| **O**CP | Extend behavior without changing existing code. | Polymorphism |
| **L**SP | Subclasses shouldn't break the rules of parents. | Contracts |
| **I**SP | Don't force implementation of unused methods. | Role Interfaces |
| **D**IP | Rely on abstractions, not concretions (interfaces everywhere).| Decoupling |
