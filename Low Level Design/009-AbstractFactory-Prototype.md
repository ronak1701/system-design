# Creational Design Patterns: Abstract Factory and Prototype

This document continues the exploration of creational design patterns, focusing on the **Abstract Factory** and **Prototype** patterns. These patterns provide advanced mechanisms for object creation, enhancing flexibility, and promoting code reusability.

---

## 1. Abstract Factory Pattern

### Introduction
The **Abstract Factory** pattern provides an interface for creating families of related or dependent objects without specifying their concrete classes. While the Factory Method pattern creates one type of object, an Abstract Factory creates an entire family of related objects.

### Real-World Analogy
Think of a furniture shop. They sell families of related products: Chair, Sofa, and Coffee Table. These products are available in different variants: Modern, Victorian, and Art Deco.
The Abstract Factory gives you an interface for creating each item (a `createChair()`, `createSofa()`), but you use a specific factory (like `ModernFurnitureFactory` or `VictorianFurnitureFactory`) to ensure all the pieces you get belong to the same matching style.

### When to Use
- When your code needs to work with various families of related products, but you don't want it to depend on the concrete classes of those products (they might be unknown beforehand or you simply want to allow for future extensibility).
- When you have a system that must be configured with one of multiple families of products (e.g., UI elements for Windows vs. macOS).

### TypeScript Implementation

```typescript
// abstract-factory-example.ts

// --- 1. Abstract Products ---
// Families of products share the same interfaces.

interface Button {
  render(): void;
  onClick(): void;
}

interface Checkbox {
  render(): void;
  toggle(): void;
}

// --- 2. Concrete Products ---
// Implementations of the abstract products, grouped by variants (Windows, MacOS).

class WindowsButton implements Button {
  render(): void { console.log("Rendering Windows Button"); }
  onClick(): void { console.log("Handling Windows Button click"); }
}

class WindowsCheckbox implements Checkbox {
  render(): void { console.log("Rendering Windows Checkbox"); }
  toggle(): void { console.log("Toggling Windows Checkbox state"); }
}

class MacOSButton implements Button {
  render(): void { console.log("Rendering MacOS Button"); }
  onClick(): void { console.log("Handling MacOS Button click"); }
}

class MacOSCheckbox implements Checkbox {
  render(): void { console.log("Rendering MacOS Checkbox"); }
  toggle(): void { console.log("Toggling MacOS Checkbox state"); }
}

// --- 3. Abstract Factory ---
// Declares creation methods for each abstract product.

interface GUIFactory {
  createButton(): Button;
  createCheckbox(): Checkbox;
}

// --- 4. Concrete Factories ---
// Implement the creation methods of the abstract factory. Each concrete factory corresponds to a specific variant of products.

class WindowsFactory implements GUIFactory {
  createButton(): Button { return new WindowsButton(); }
  createCheckbox(): Checkbox { return new WindowsCheckbox(); }
}

class MacOSFactory implements GUIFactory {
  createButton(): Button { return new MacOSButton(); }
  createCheckbox(): Checkbox { return new MacOSCheckbox(); }
}

// --- Usage ---

// Client code only interacts with the factories and products through abstract interfaces.
class Application {
  private button: Button;
  private checkbox: Checkbox;

  constructor(factory: GUIFactory) {
    this.button = factory.createButton();
    this.checkbox = factory.createCheckbox();
  }

  paint(): void {
    this.button.render();
    this.checkbox.render();
  }
}

// Application configurator
const osType = "MacOS"; // This could be dynamically determined at runtime
let factory: GUIFactory;

if (osType === "Windows") {
  factory = new WindowsFactory();
} else {
  factory = new MacOSFactory();
}

const app = new Application(factory);
app.paint();
// Outputs:
// Rendering MacOS Button
// Rendering MacOS Checkbox
```

### Pros and Cons
**Pros:**
- You can be sure that the products you're getting from a factory are compatible with each other.
- Avoids tight coupling between concrete products and client code.
- **Single Responsibility Principle:** You can extract the product creation code into one place, making the code easier to support.
- **Open/Closed Principle:** You can introduce new variants of products without breaking existing client code.

**Cons:**
- The code may become more complicated than it should be, since a lot of new interfaces and classes are introduced along with the pattern.

---

## 2. Prototype Pattern

### Introduction
The **Prototype** pattern specifies the kinds of objects to create using a prototypical instance, and creates new objects by copying (cloning) this prototype. It allows you to produce clones of an object without coupling your code to the object's specific classes.

### Real-World Analogy
Think of biological cell division (mitosis). To create a new cell, an organism doesn't assemble one from scratch by gathering proteins and lipids one by one. Instead, an existing cell simply duplicates itself. The original cell acts as a prototype, and the newly created cell is a clone of the original, ready to function immediately. Alternatively, think of copying and pasting a complex document as a template, rather than retyping it from scratch.

### When to Use
- When the cost of creating a new object from scratch (using the `new` keyword and fully initializing it) is more expensive or complicated than copying an existing one.
- When your code shouldn't depend on the concrete classes of objects that you need to copy.
- To avoid building a hierarchy of factories that parallels the hierarchy of products.

### TypeScript Implementation

In TypeScript, you can implement cloning by defining a `clone()` method. The cloning process can be as simple as copying property values, or more complex (e.g., deep copying nested objects).

```typescript
// prototype-example.ts

// 1. Prototype Interface
interface Prototype<T> {
  clone(): T;
}

// 2. Concrete Prototype
class Enemy implements Prototype<Enemy> {
  private type: string;
  private health: number;
  private speed: number;
  private armored: boolean;
  private weapon: string;

  constructor(type: string, health: number, speed: number, armored: boolean, weapon: string) {
    console.log(`Constructor called for ${type}!`); // Notice this is only called when 'new' is used
    this.type = type;
    this.health = health;
    this.speed = speed;
    this.armored = armored;
    this.weapon = weapon;
  }

  // The clone method creates a new instance based on the current object's state
  clone(): Enemy {
    // In this simple case, we just instantiate a new object with identical properties.
    // However, in a real-world scenario with complex objects, you might use 
    // Object.create() or implement a deep copy strategy to avoid recalling expensive constructors.
    return new Enemy(
      this.type,
      this.health,
      this.speed,
      this.armored,
      this.weapon
    );
  }

  setHealth(health: number): void {
    this.health = health;
  }

  printStats(): void {
    console.log(
      `${this.type} [Health: ${this.health}, Speed: ${this.speed}, Armored: ${this.armored}, Weapon: ${this.weapon}]`
    );
  }
}

// --- Usage ---

// We create an initial 'prototype' enemy from scratch
const basicSoldier = new Enemy("Soldier", 100, 10, false, "Assault Rifle");

// Instead of setting up new enemies from scratch, we clone the prototype
const clone1 = basicSoldier.clone();
const clone2 = basicSoldier.clone();

console.log("\n--- Initial Stats ---");
basicSoldier.printStats();
clone1.printStats();
clone2.printStats();

// We can independently modify the clones without affecting the original prototype
clone1.setHealth(50);
clone2.setHealth(0);

console.log("\n--- After Modifications ---");
basicSoldier.printStats();
clone1.printStats();
clone2.printStats();
```

### Note on Deep vs. Shallow Copying
When using the Prototype pattern, you must carefully consider whether you need a **shallow copy** or a **deep copy**.
- **Shallow Copy:** Copies the primitive values, but references to objects are shared between the original and the clone.
- **Deep Copy:** Recursively copies all properties, meaning the clone and the original are completely independent (like using `structuredClone()` in modern JavaScript environments). 

### Pros and Cons
**Pros:**
- You can clone objects without coupling to their concrete classes.
- You can get rid of repeated initialization code in favor of cloning pre-built prototypes.
- You can produce complex objects more conveniently.
- Offers an alternative to inheritance when dealing with configuration presets for complex objects.

**Cons:**
- Cloning complex objects that have circular references might be very tricky.

---

## Summary
- **Abstract Factory**: Use when you need to create *families* of related or dependent objects (like UI elements for different OS themes) without tying your code to concrete classes.
- **Prototype**: Use when creating an object is expensive or complex, and you'd rather copy an existing instance (the prototype) and tweak it instead of starting from scratch.
