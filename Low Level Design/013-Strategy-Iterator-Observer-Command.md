# Behavioral Design Patterns: Strategy, Iterator, Observer, and Command

This document begins our exploration of **Behavioral** design patterns. While creational patterns focus on object creation and structural patterns on object composition, behavioral patterns are concerned with algorithms and the assignment of responsibilities between objects. They characterize complex control flow that's difficult to follow at runtime.

Here we cover four essential behavioral patterns: **Strategy**, **Iterator**, **Observer**, and **Command**.

---

## 1. Strategy Pattern

### Introduction
The **Strategy** pattern lets you define a family of algorithms, put each of them into a separate class, and make their objects interchangeable. It allows the algorithm to vary independently from clients that use it.

### Real-World Analogy
Think about how you might travel from your home to the airport. You have several options:

- Drive yourself: Flexible timing, but you pay for parking
- Taxi/Uber: Door-to-door service, variable pricing
- Public transit: Cheapest option, but takes longer
- Airport shuttle: Fixed schedule, moderate cost

Each of these is a "travel strategy." You (the traveler) decide which strategy to use based on factors like cost, time, and convenience. The important point is that you do not change how you "travel" as a concept. You just swap out the method.

The Strategy pattern works the same way.

### When to Use
- When you want to use different variants of an algorithm within an object and be able to switch from one algorithm to another during runtime.
- When you have a lot of similar classes that only differ in the way they execute some behavior.
- To isolate the business logic of a class from the implementation details of algorithms that may not be as important in the context of that logic.
- When your class has a massive conditional statement (`if/else` or `switch`) that switches between different variants of the same algorithm.

### TypeScript Implementation

```typescript
// strategy-example.ts

// --- 1. Strategy Interface ---
// Defines the common method for all supported algorithms.
interface PaymentStrategy {
  pay(amount: number): void;
}

// --- 2. Concrete Strategies ---
// Implement different variations of an algorithm.
class CreditCardPayment implements PaymentStrategy {
  constructor(private cardNumber: string, private cvv: string) {}

  pay(amount: number): void {
    console.log(`Paid ${amount} using Credit Card ending in ${this.cardNumber.slice(-4)}.`);
  }
}

class PayPalPayment implements PaymentStrategy {
  constructor(private email: string) {}

  pay(amount: number): void {
    console.log(`Paid ${amount} using PayPal account: ${this.email}.`);
  }
}

class UPIPayment implements PaymentStrategy {
  constructor(private upiId: string) {}

  pay(amount: number): void {
    console.log(`Paid ${amount} using UPI ID: ${this.upiId}.`);
  }
}

// --- 3. Context ---
// Maintains a reference to one of the concrete strategies and communicates with it via the strategy interface.
class ShoppingCart {
  private strategy: PaymentStrategy;

  // The context allows replacing a Strategy object at runtime.
  setPaymentStrategy(strategy: PaymentStrategy): void {
    this.strategy = strategy;
  }

  checkout(amount: number): void {
    if (!this.strategy) {
      console.log("Please select a payment method before checking out.");
      return;
    }
    // The context delegates the work to the Strategy object instead of implementing multiple versions of the algorithm on its own.
    this.strategy.pay(amount);
  }
}

// --- Usage ---

const cart = new ShoppingCart();

// Using Credit Card
cart.setPaymentStrategy(new CreditCardPayment("1234567890123456", "123"));
cart.checkout(100);

// Switching to PayPal at runtime
cart.setPaymentStrategy(new PayPalPayment("user@example.com"));
cart.checkout(250);

// Switching to UPI at runtime
cart.setPaymentStrategy(new UPIPayment("user@upi"));
cart.checkout(50);
```

### Pros and Cons
**Pros:**
- Swap algorithms used inside an object at runtime.
- Isolate the implementation details of an algorithm from the code that uses it.
- Replace inheritance with composition.
- **Open/Closed Principle:** Introduce new strategies without having to change the context.

**Cons:**
- If you only have a couple of algorithms and they rarely change, there's no real reason to overcomplicate the program with new classes and interfaces.
- Clients must be aware of the differences between strategies to be able to select a proper one.

---

## 2. Iterator Pattern

### Introduction
The **Iterator** pattern lets you traverse elements of a collection without exposing its underlying representation (list, stack, tree, graph, etc.). It extracts the traversal behavior of a collection into a separate object called an iterator.

### Real-World Analogy
Consider a TV remote control. When you press the "next channel" button, you do not need to know how the TV internally organizes its channel list. Maybe it is stored as an array, a linked list, or fetched from a satellite signal.

The remote provides a simple interface: next channel, previous channel. The complexity of channel management is hidden behind that interface.

The Iterator pattern works the same way. The iterator is like the remote control, providing a simple interface to move through a collection without exposing how that collection is structured internally.

### When to Use
- When your collection has a complex data structure under the hood, but you want to hide its complexity from clients (either for convenience or security reasons).
- To reduce duplication of the traversal code across your app.
- When you want your code to be able to traverse different data structures or when types of these structures are unknown beforehand.

### TypeScript Implementation

```typescript
// iterator-example.ts

// --- 1. Iterator Interface ---
// Declares the operations required for traversing a collection.
interface Iterator<T> {
  current(): T;
  next(): T;
  key(): number;
  valid(): boolean;
  rewind(): void;
}

// --- 2. Iterable Collection Interface ---
// Declares one or multiple methods for getting iterators compatible with the collection.
interface Aggregator {
  getIterator(): Iterator<string>;
}

// --- 3. Concrete Iterator ---
// Implements specific traversal algorithms.
class AlphabeticalOrderIterator implements Iterator<string> {
  private collection: WordsCollection;
  private position: number = 0;
  private reverse: boolean = false;

  constructor(collection: WordsCollection, reverse: boolean = false) {
    this.collection = collection;
    this.reverse = reverse;

    if (reverse) {
      this.position = collection.getCount() - 1;
    }
  }

  rewind(): void {
    this.position = this.reverse ? this.collection.getCount() - 1 : 0;
  }

  current(): string {
    return this.collection.getItems()[this.position];
  }

  key(): number {
    return this.position;
  }

  next(): string {
    const item = this.collection.getItems()[this.position];
    this.position += this.reverse ? -1 : 1;
    return item;
  }

  valid(): boolean {
    if (this.reverse) {
      return this.position >= 0;
    }
    return this.position < this.collection.getCount();
  }
}

// --- 4. Concrete Collection ---
// Returns one or several different iterators.
class WordsCollection implements Aggregator {
  private items: string[] = [];

  getItems(): string[] {
    return this.items;
  }

  getCount(): number {
    return this.items.length;
  }

  addItem(item: string): void {
    this.items.push(item);
  }

  getIterator(): Iterator<string> {
    return new AlphabeticalOrderIterator(this);
  }

  getReverseIterator(): Iterator<string> {
    return new AlphabeticalOrderIterator(this, true);
  }
}

// --- Usage ---

const collection = new WordsCollection();
collection.addItem("First");
collection.addItem("Second");
collection.addItem("Third");

console.log("Straight traversal:");
const iterator = collection.getIterator();
while (iterator.valid()) {
  console.log(iterator.next());
}

console.log("\nReverse traversal:");
const reverseIterator = collection.getReverseIterator();
while (reverseIterator.valid()) {
  console.log(reverseIterator.next());
}
```

### Pros and Cons
**Pros:**
- **Single Responsibility Principle:** You can clean up the client code and the collections by extracting bulky traversal algorithms into separate classes.
- **Open/Closed Principle:** You can implement new types of collections and iterators and pass them to existing code without breaking anything.
- You can iterate over the same collection in parallel because each iterator object contains its own iteration state.
- You can delay an iteration and continue it when needed.

**Cons:**
- Applying the pattern can be an overkill if your app only works with simple collections.
- Using an iterator may be less efficient than going through elements of some specialized collections directly.

---

## 3. Observer Pattern

### Introduction
The **Observer** pattern (also known as Publish-Subscribe) lets you define a subscription mechanism to notify multiple objects about any events that happen to the object they're observing. It allows an object (Subject) to maintain a list of its dependents (Observers) and notify them automatically of any state changes.

### Real-World Analogy
Think about a newspaper subscription. You subscribe to a newspaper publisher. Every morning, the publisher prints the paper and delivers a copy to every subscriber on its list. The publisher does not know whether you read the sports section, clip coupons, or just check the headlines. It does not care. It delivers the paper, and each subscriber decides what to do with it.

When you cancel your subscription, the deliveries stop. When a new neighbor subscribes, they start getting the paper. The publisher's printing logic never changes.

The Observer pattern works the same way: the subject (publisher) broadcasts updates, and observers (subscribers) react however they choose.

### When to Use
- When changes to the state of one object may require changing other objects, and the actual set of objects is unknown beforehand or changes dynamically.
- When some objects in your app must observe others, but only for a limited time or in specific cases.
- Highly used in UI frameworks, Event Management systems, and Reactivity (e.g., RxJS).

### TypeScript Implementation

```typescript
// observer-example.ts

// --- 1. Subscriber/Observer Interface ---
// Declares the notification interface.
interface Observer {
  update(subject: Subject): void;
}

// --- 2. Publisher/Subject Interface ---
// Declares a set of methods for managing subscribers.
interface Subject {
  attach(observer: Observer): void;
  detach(observer: Observer): void;
  notify(): void;
}

// --- 3. Concrete Subject ---
// Owns some important state and notifies observers when the state changes.
class WeatherStation implements Subject {
  public temperature: number = 0;
  private observers: Observer[] = [];

  attach(observer: Observer): void {
    const isExist = this.observers.includes(observer);
    if (isExist) {
      return console.log('WeatherStation: Observer has been attached already.');
    }
    this.observers.push(observer);
  }

  detach(observer: Observer): void {
    const observerIndex = this.observers.indexOf(observer);
    if (observerIndex === -1) {
      return console.log('WeatherStation: Nonexistent observer.');
    }
    this.observers.splice(observerIndex, 1);
  }

  notify(): void {
    console.log(`WeatherStation: Notifying ${this.observers.length} observers...`);
    for (const observer of this.observers) {
      observer.update(this);
    }
  }

  // The Subject's business logic
  setTemperature(temp: number): void {
    console.log(`\nWeatherStation: Temperature changed to ${temp}°C`);
    this.temperature = temp;
    this.notify(); // State changed, notify observers
  }
}

// --- 4. Concrete Observers ---
// React to the updates issued by the Subject.
class PhoneDisplay implements Observer {
  update(subject: Subject): void {
    if (subject instanceof WeatherStation) {
      console.log(`PhoneDisplay: The temperature is now ${subject.temperature}°C`);
    }
  }
}

class WindowDisplay implements Observer {
  update(subject: Subject): void {
    if (subject instanceof WeatherStation) {
      console.log(`WindowDisplay: The weather outside is ${subject.temperature}°C`);
    }
  }
}

// --- Usage ---

const weatherStation = new WeatherStation();

const phoneDisplay = new PhoneDisplay();
const windowDisplay = new WindowDisplay();

weatherStation.attach(phoneDisplay);
weatherStation.attach(windowDisplay);

// State change will notify both displays
weatherStation.setTemperature(25);

// Detaching an observer dynamically
weatherStation.detach(windowDisplay);

// State change will notify only the remaining display
weatherStation.setTemperature(30);
```

### Pros and Cons
**Pros:**
- **Open/Closed Principle:** You can introduce new subscriber classes without having to change the publisher's code (and vice versa if there's a publisher interface).
- You can establish relations between objects at runtime.

**Cons:**
- Subscribers are notified in random order.
- Might cause memory leaks if observers are not correctly detached when no longer needed (Lapsed Listener Problem).

---

## 4. Command Pattern

### Introduction
The **Command** pattern turns a request into a stand-alone object that contains all information about the request. This transformation lets you pass requests as a method arguments, delay or queue a request's execution, and support undoable operations.

### Real-World Analogy
Think about ordering at a restaurant. You tell the waiter what you want (your request), and the waiter writes it on an order slip. The waiter does not cook the food. They carry the slip to the kitchen and hand it to the chef. The chef reads the slip and prepares the dish.

The order slip is the command object. The waiter is the invoker, carrying and delivering the request. The chef is the receiver, doing the actual work. The customer is the client, creating the request.

The waiter does not need to know how to cook, and the chef does not need to know who ordered. The slip decouples them completely. If you want to cancel, the waiter pulls the slip from the queue, the same slip that started the process can undo it.

### When to Use
- When you want to parametrize objects with operations.
- When you want to queue operations, schedule their execution, or execute them remotely.
- When you want to implement reversible operations (undo/redo).

### TypeScript Implementation

```typescript
// command-example.ts

// --- 1. Command Interface ---
// Declares the method for executing a command.
interface Command {
  execute(): void;
  undo(): void;
}

// --- 2. Receiver ---
// Contains the actual business logic. Commands delegate actual work to receivers.
class Light {
  turnOn(): void {
    console.log("The light is ON");
  }

  turnOff(): void {
    console.log("The light is OFF");
  }
}

// --- 3. Concrete Commands ---
// Implement various kinds of requests.
class LightOnCommand implements Command {
  constructor(private light: Light) {}

  execute(): void {
    this.light.turnOn();
  }

  undo(): void {
    this.light.turnOff();
  }
}

class LightOffCommand implements Command {
  constructor(private light: Light) {}

  execute(): void {
    this.light.turnOff();
  }

  undo(): void {
    this.light.turnOn();
  }
}

// --- 4. Invoker ---
// Responsible for initiating requests. It contains a reference to the command object.
class RemoteControl {
  private commandHistory: Command[] = [];

  executeCommand(command: Command): void {
    command.execute();
    this.commandHistory.push(command);
  }

  undoLastCommand(): void {
    const lastCommand = this.commandHistory.pop();
    if (lastCommand) {
      console.log("Undoing last command:");
      lastCommand.undo();
    } else {
      console.log("No commands to undo.");
    }
  }
}

// --- Usage ---

const livingRoomLight = new Light();
const turnOnLight = new LightOnCommand(livingRoomLight);
const turnOffLight = new LightOffCommand(livingRoomLight);

const remote = new RemoteControl();

// Execute commands
remote.executeCommand(turnOnLight);   // Output: The light is ON
remote.executeCommand(turnOffLight);  // Output: The light is OFF

// Undo operations
remote.undoLastCommand();             // Output: Undoing last command: The light is ON
remote.undoLastCommand();             // Output: Undoing last command: The light is OFF
remote.undoLastCommand();             // Output: No commands to undo.
```

### Pros and Cons
**Pros:**
- **Single Responsibility Principle:** You can decouple classes that invoke operations from classes that perform these operations.
- **Open/Closed Principle:** You can introduce new commands into the app without breaking existing client code.
- You can implement undo/redo.
- You can implement deferred execution of operations.
- You can assemble a set of simple commands into a complex one.

**Cons:**
- The code may become more complicated since you're introducing a whole new layer between senders and receivers.

---

## Summary
- **Strategy**: Defines a family of algorithms, encapsulating each one, and making them interchangeable dynamically.
- **Iterator**: Provides a way to sequentially access elements of a collection without exposing its underlying representation.
- **Observer**: Defines a one-to-many dependency between objects so that when one object changes state, all its dependents are notified and updated automatically.
- **Command**: Encapsulates a request as an object, thereby letting you parameterize clients with different requests, queue or log requests, and support undoable operations.
