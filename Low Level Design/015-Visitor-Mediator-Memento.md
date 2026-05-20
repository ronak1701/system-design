# Behavioral Design Patterns: Visitor, Mediator, and Memento

This document concludes our exploration of **Behavioral** design patterns. While previous behavioral patterns (like Strategy, Command, and State) handled control flow and responsibilities, the patterns discussed here address complex object interactions, object state restoration, and extending object functionality without modifying their structure.

Here we cover three advanced behavioral patterns: **Visitor**, **Mediator**, and **Memento**.

---

## 1. Visitor Pattern

### Introduction
The **Visitor** pattern lets you separate algorithms from the objects on which they operate. It allows you to add new behaviors to existing class hierarchies without altering any existing code. This is achieved by creating an external "visitor" class that implements the new behavior and passing the original objects to this visitor.

### Real-World Analogy
Think about a home inspection. You have a house with different components: plumbing, electrical wiring, structural framing, and HVAC. Each specialist (a plumber, an electrician, a structural engineer, an HVAC technician) "visits" the house and inspects only what they understand.

The house does not need to know how to evaluate its own plumbing or wiring. It just opens the door and lets each inspector do their job. Adding a new type of inspection (say, a fire safety audit) does not require remodeling the house. You just bring in a new inspector.

The Visitor pattern works the same way: the elements (house components) accept visitors (inspectors), and new operations are new visitors.

### When to Use
- When you need to perform an operation on all elements of a complex object structure (e.g., an object tree).
- When you want to clean up the business logic of auxiliary behaviors.
- When a behavior makes sense only in some classes of a class hierarchy, but not in others.

### TypeScript Implementation

```typescript
// visitor-example.ts

// --- 1. Element Interface ---
// Declares an `accept` method that takes the base visitor interface as an argument.
interface Component {
  accept(visitor: Visitor): void;
}

// --- 2. Concrete Elements ---
// Implement the acceptance method. The purpose is to redirect the call to the proper visitor's method corresponding to the current element class.
class ConcreteComponentA implements Component {
  // Note that we're calling `visitConcreteComponentA`, which matches the current class name.
  public accept(visitor: Visitor): void {
    visitor.visitConcreteComponentA(this);
  }

  // Concrete Elements may have special methods that don't exist in their base class or interface.
  public exclusiveMethodOfComponentA(): string {
    return 'A';
  }
}

class ConcreteComponentB implements Component {
  public accept(visitor: Visitor): void {
    visitor.visitConcreteComponentB(this);
  }

  public specialMethodOfComponentB(): string {
    return 'B';
  }
}

// --- 3. Visitor Interface ---
// Declares a set of visiting methods that correspond to element classes.
interface Visitor {
  visitConcreteComponentA(element: ConcreteComponentA): void;
  visitConcreteComponentB(element: ConcreteComponentB): void;
}

// --- 4. Concrete Visitors ---
// Implement several versions of the same algorithm, which can work with all concrete component classes.
class ConcreteVisitor1 implements Visitor {
  public visitConcreteComponentA(element: ConcreteComponentA): void {
    console.log(`${element.exclusiveMethodOfComponentA()} + ConcreteVisitor1`);
  }

  public visitConcreteComponentB(element: ConcreteComponentB): void {
    console.log(`${element.specialMethodOfComponentB()} + ConcreteVisitor1`);
  }
}

class ConcreteVisitor2 implements Visitor {
  public visitConcreteComponentA(element: ConcreteComponentA): void {
    console.log(`${element.exclusiveMethodOfComponentA()} + ConcreteVisitor2`);
  }

  public visitConcreteComponentB(element: ConcreteComponentB): void {
    console.log(`${element.specialMethodOfComponentB()} + ConcreteVisitor2`);
  }
}

// --- Usage ---

function clientCode(components: Component[], visitor: Visitor) {
  for (const component of components) {
    component.accept(visitor);
  }
}

const components = [
  new ConcreteComponentA(),
  new ConcreteComponentB(),
];

console.log('The client code works with all visitors via the base Visitor interface:');
const visitor1 = new ConcreteVisitor1();
clientCode(components, visitor1);

console.log('');

console.log('It allows the same client code to work with different types of visitors:');
const visitor2 = new ConcreteVisitor2();
clientCode(components, visitor2);
```

### Pros and Cons
**Pros:**
- **Open/Closed Principle:** You can introduce a new behavior that can work with objects of different classes without changing these classes.
- **Single Responsibility Principle:** You can move multiple versions of the same behavior into the same class.
- A visitor object can accumulate some useful information while working with various objects (useful for traversing trees).

**Cons:**
- You need to update all visitors each time a class gets added to or removed from the element hierarchy.
- Visitors might lack the necessary access to the private fields and methods of the elements they're supposed to work with.

---

## 2. Mediator Pattern

### Introduction
The **Mediator** pattern reduces chaotic dependencies between objects. The pattern restricts direct communications between the objects and forces them to collaborate only via a mediator object. This prevents components from explicitly referring to each other, promoting loose coupling.

### Real-World Analogy
Think about an air traffic control tower. Planes approaching an airport do not communicate with each other to negotiate landing order, runway assignments, or holding patterns. That would be chaos, especially with dozens of planes in the air simultaneously. Instead, every pilot talks only to the control tower.

The tower knows the position, speed, and fuel status of every plane and coordinates them accordingly: "Flight 237, hold at 5,000 feet. Flight 412, you are cleared for runway 28L."

The tower is the mediator. Planes are the components. If a new plane enters the airspace, the tower handles it. If a plane diverts, the tower adjusts. The other planes never need to know.

### When to Use
- When it's hard to change some of the classes because they are tightly coupled to a bunch of other classes.
- When you can't reuse a component in a different program because it's too dependent on other components.
- When you find yourself creating tons of component subclasses just to reuse some basic behavior in various contexts.

### TypeScript Implementation

```typescript
// mediator-example.ts

// --- 1. Mediator Interface ---
// Declares a method used by components to notify the mediator about various events.
interface Mediator {
  notify(sender: object, event: string): void;
}

// --- 2. Concrete Mediator ---
// Implements cooperative behavior by coordinating several components.
class ConcreteMediator implements Mediator {
  private component1: Component1;
  private component2: Component2;

  constructor(c1: Component1, c2: Component2) {
    this.component1 = c1;
    this.component1.setMediator(this);
    this.component2 = c2;
    this.component2.setMediator(this);
  }

  public notify(sender: object, event: string): void {
    if (event === 'A') {
      console.log('Mediator reacts on A and triggers following operations:');
      this.component2.doC();
    }

    if (event === 'D') {
      console.log('Mediator reacts on D and triggers following operations:');
      this.component1.doB();
      this.component2.doC();
    }
  }
}

// --- 3. Base Component ---
// Provides the basic functionality of storing a mediator's instance inside component objects.
class BaseComponent {
  protected mediator: Mediator;

  constructor(mediator: Mediator = null as any) {
    this.mediator = mediator;
  }

  public setMediator(mediator: Mediator): void {
    this.mediator = mediator;
  }
}

// --- 4. Concrete Components ---
// Implement various functionality. They don't depend on other components or any concrete mediator classes.
class Component1 extends BaseComponent {
  public doA(): void {
    console.log('Component 1 does A.');
    this.mediator.notify(this, 'A');
  }

  public doB(): void {
    console.log('Component 1 does B.');
    this.mediator.notify(this, 'B');
  }
}

class Component2 extends BaseComponent {
  public doC(): void {
    console.log('Component 2 does C.');
    this.mediator.notify(this, 'C');
  }

  public doD(): void {
    console.log('Component 2 does D.');
    this.mediator.notify(this, 'D');
  }
}

// --- Usage ---

const c1 = new Component1();
const c2 = new Component2();
const mediator = new ConcreteMediator(c1, c2);

console.log('Client triggers operation A.');
c1.doA();

console.log('');
console.log('Client triggers operation D.');
c2.doD();
```

### Pros and Cons
**Pros:**
- **Single Responsibility Principle:** You can extract the communications between various components into a single place, making it easier to comprehend and maintain.
- **Open/Closed Principle:** You can introduce new mediators without having to change the actual components.
- You can reduce coupling between various components of a program.
- You can reuse individual components more easily.

**Cons:**
- Over time, a mediator can evolve into a *God Object* (a class that knows too much or does too much).

---

## 3. Memento Pattern

### Introduction
The **Memento** pattern lets you save and restore the previous state of an object without revealing the details of its implementation. It's heavily used in implementing "Undo" features.

### Real-World Analogy
Think about playing a difficult level in a video game. Before entering a boss fight, you save your progress. The game engine creates a "save state" (memento) containing your current health, inventory, and location.

If you lose the fight, you don't have to restart the game from the beginning. You simply load your previous save state. The save state is an opaque object to you—you can't manually edit the binary file to give yourself more health. Only the game engine (the originator) knows how to read and restore from that specific save file. The save menu (the caretaker) simply holds onto the list of save files for you.

### When to Use
- When you want to produce snapshots of the object's state to be able to restore a previous state of the object.
- When direct access to the object's fields/getters/setters violates its encapsulation.

### TypeScript Implementation

```typescript
// memento-example.ts

// --- 1. Originator ---
// Holds some important state that may change over time. It also defines a method for saving its state inside a memento and another method for restoring the state from it.
class Editor {
  private content: string;

  constructor(content: string) {
    this.content = content;
    console.log(`Editor: Initial content is: "${this.content}"`);
  }

  // The Originator's business logic may affect its internal state.
  public type(words: string): void {
    console.log(`Editor: Typing "${words}"...`);
    this.content += " " + words;
  }

  public getContent(): string {
    return this.content;
  }

  // Saves the current state inside a memento.
  public save(): Memento {
    return new ConcreteMemento(this.content);
  }

  // Restores the Originator's state from a memento object.
  public restore(memento: Memento): void {
    this.content = memento.getState();
    console.log(`Editor: State has changed to: "${this.content}"`);
  }
}

// --- 2. Memento Interface ---
// Provides a way to retrieve the memento's metadata, such as creation date or name.
// However, it doesn't expose the Originator's state.
interface Memento {
  getName(): string;
  getDate(): string;
  getState(): string; // Usually internal or hidden from Caretaker, but needed by Originator
}

// --- 3. Concrete Memento ---
// Contains the infrastructure for storing the Originator's state.
class ConcreteMemento implements Memento {
  private state: string;
  private date: string;

  constructor(state: string) {
    this.state = state;
    this.date = new Date().toISOString().slice(0, 19).replace('T', ' ');
  }

  public getState(): string {
    return this.state;
  }

  public getName(): string {
    return `${this.date} / (${this.state.slice(0, 9)}...)`;
  }

  public getDate(): string {
    return this.date;
  }
}

// --- 4. Caretaker ---
// Doesn't depend on the Concrete Memento class. Therefore, it doesn't have access to the originator's state, stored inside the memento.
class CommandHistory {
  private mementos: Memento[] = [];
  private originator: Editor;

  constructor(originator: Editor) {
    this.originator = originator;
  }

  public backup(): void {
    console.log('\nCommandHistory: Saving Editor\'s state...');
    this.mementos.push(this.originator.save());
  }

  public undo(): void {
    if (!this.mementos.length) {
      return;
    }
    const memento = this.mementos.pop()!;
    console.log(`\nCommandHistory: Restoring state to: ${memento.getName()}`);
    this.originator.restore(memento);
  }

  public showHistory(): void {
    console.log('\nCommandHistory: Here\'s the list of mementos:');
    for (const memento of this.mementos) {
      console.log(memento.getName());
    }
  }
}

// --- Usage ---

const editor = new Editor("Initial text.");
const history = new CommandHistory(editor);

history.backup();
editor.type("This is the first sentence.");

history.backup();
editor.type("And the second one.");

history.backup();
editor.type("Oh wait, this is a mistake.");

console.log('');
console.log(`Current Editor Content: ${editor.getContent()}`);

history.showHistory();

console.log('\nClient: Now, let\'s rollback!');
history.undo();

console.log('\nClient: Once more!');
history.undo();
```

### Pros and Cons
**Pros:**
- You can produce snapshots of the object's state without violating its encapsulation.
- You can simplify the originator's code by letting the caretaker maintain the history of the originator's state.

**Cons:**
- The app might consume lots of RAM if clients create mementos too often.
- Caretakers should track the originator's lifecycle to be able to destroy obsolete mementos.
- Most dynamic programming languages (like JavaScript/TypeScript) can't guarantee that the state within the memento stays untouched, requiring discipline.

---

## Summary
- **Visitor**: Allows adding new operations to existing object structures without modifying those structures. It's useful for operating on a whole hierarchy of distinct objects.
- **Mediator**: Centralizes complex communications and control between related objects. It promotes loose coupling by keeping objects from referring to each other explicitly.
- **Memento**: Captures and externalizes an object's internal state without violating encapsulation, allowing the object to be restored to this state later (useful for undo/redo).
