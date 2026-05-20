# Structural Design Patterns: Bridge and Flyweight

This document continues the exploration of Structural design patterns, focusing on the **Bridge** and **Flyweight** patterns. These patterns provide ways to organize complex class hierarchies and optimize memory usage for large numbers of objects.

---

## 1. Bridge Pattern

### Introduction
The **Bridge** pattern lets you split a large class or a set of closely related classes into two separate hierarchies—abstraction and implementation—which can be developed independently. It decouples an abstraction from its implementation so that the two can vary independently.

### Real-World Analogy
Think of a TV remote control and the TV itself. The remote is the abstraction: it has buttons for power, volume, and channel. The TV is the implementation: it contains the circuits that actually change volume, switch channels, and toggle power. You can swap remotes without changing the TV, and you can swap TVs without changing the remote.

A basic remote works with a Samsung TV. The same Samsung TV works with a universal remote that has extra buttons. The remote hierarchy and the TV hierarchy vary independently, connected only by the infrared signal between them. That signal is the bridge.

### When to Use
- When you want to divide and organize a monolithic class that has several variants of some functionality (for example, if the class can work with various database servers).
- When you need to extend a class in several orthogonal (independent) dimensions.
- When you need to be able to switch implementations at runtime without affecting the client code.

### TypeScript Implementation

```typescript
// bridge-example.ts

// --- 1. Implementation Interface ---
// Defines the interface for all implementation classes.
interface Device {
  isEnabled(): boolean;
  enable(): void;
  disable(): void;
  getVolume(): number;
  setVolume(percent: number): void;
}

// --- 2. Concrete Implementations ---
// Platform-specific implementations.
class Tv implements Device {
  private on: boolean = false;
  private volume: number = 30;

  isEnabled(): boolean { return this.on; }
  enable(): void { this.on = true; }
  disable(): void { this.on = false; }
  getVolume(): number { return this.volume; }
  setVolume(percent: number): void { this.volume = percent; }
}

class Radio implements Device {
  private on: boolean = false;
  private volume: number = 10;

  isEnabled(): boolean { return this.on; }
  enable(): void { this.on = true; }
  disable(): void { this.on = false; }
  getVolume(): number { return this.volume; }
  setVolume(percent: number): void { this.volume = percent; }
}

// --- 3. Abstraction ---
// Defines the control part of the two class hierarchies. It maintains a reference to an object of the Implementation hierarchy and delegates all of the real work to this object.
class RemoteControl {
  protected device: Device;

  constructor(device: Device) {
    this.device = device;
  }

  togglePower(): void {
    if (this.device.isEnabled()) {
      this.device.disable();
      console.log("Remote: device turned off.");
    } else {
      this.device.enable();
      console.log("Remote: device turned on.");
    }
  }

  volumeDown(): void {
    this.device.setVolume(this.device.getVolume() - 10);
    console.log(`Remote: volume down to ${this.device.getVolume()}`);
  }

  volumeUp(): void {
    this.device.setVolume(this.device.getVolume() + 10);
    console.log(`Remote: volume up to ${this.device.getVolume()}`);
  }
}

// --- 4. Refined Abstraction ---
// Extends the control logic without changing the device classes.
class AdvancedRemoteControl extends RemoteControl {
  mute(): void {
    this.device.setVolume(0);
    console.log("Advanced Remote: device muted.");
  }
}

// --- Usage ---

// Client code can work with any device using the same remote abstraction.
console.log("Testing TV with basic remote:");
const tv = new Tv();
const remote = new RemoteControl(tv);
remote.togglePower();
remote.volumeUp();

console.log("\nTesting Radio with advanced remote:");
const radio = new Radio();
const advancedRemote = new AdvancedRemoteControl(radio);
advancedRemote.togglePower();
advancedRemote.mute();
```

### Pros and Cons
**Pros:**
- You can create platform-independent classes and apps.
- The client code works with high-level abstractions. It isn't exposed to the platform details.
- **Open/Closed Principle:** You can introduce new abstractions and implementations independently from each other.
- **Single Responsibility Principle:** You can focus on high-level logic in the abstraction and on platform details in the implementation.

**Cons:**
- You might make the code more complicated by applying the pattern to a highly cohesive class.

---

## 2. Flyweight Pattern

### Introduction
The **Flyweight** pattern is a structural design pattern that lets you fit more objects into the available amount of RAM by sharing common parts of state between multiple objects instead of keeping all of the data in each object.

### Real-World Analogy
Think of a video game rendering a forest with millions of trees. If each tree object stored its own 3D model, textures, and leaf colors, the game would quickly run out of RAM. 

Instead, you extract the shared intrinsic state (the 3D model, the bark texture, the leaf colors) into a few "Flyweight" objects like "PineTree" or "OakTree". The millions of individual tree objects in the forest only store their unique extrinsic state (their X, Y coordinates and size), and point to the shared Flyweight for the heavy data. This drastically reduces memory usage.

### When to Use
- When your application needs to spawn a huge number of similar objects.
- When this drains all available RAM on a target device.
- When the objects contain duplicate states which can be extracted and shared between multiple objects.

### TypeScript Implementation

```typescript
// flyweight-example.ts

// --- 1. Flyweight ---
// Contains the intrinsic state (shared state) that doesn't change and is the same for many objects.
class TreeType {
  constructor(
    private name: string,
    private color: string,
    private texture: string
  ) {}

  draw(canvas: string, x: number, y: number): void {
    console.log(`Drawing a ${this.color} ${this.name} tree at (${x}, ${y}) using ${this.texture} texture on ${canvas}.`);
  }
}

// --- 2. Flyweight Factory ---
// Manages the pool of flyweight objects. It ensures that flyweights are shared correctly.
class TreeFactory {
  private static treeTypes: { [key: string]: TreeType } = {};

  static getTreeType(name: string, color: string, texture: string): TreeType {
    const key = `${name}_${color}_${texture}`;
    if (!this.treeTypes[key]) {
      console.log(`TreeFactory: Creating new TreeType (${key})`);
      this.treeTypes[key] = new TreeType(name, color, texture);
    } else {
      console.log(`TreeFactory: Reusing existing TreeType (${key})`);
    }
    return this.treeTypes[key];
  }
}

// --- 3. Context ---
// Contains the extrinsic state (unique state) that changes per object and a reference to the flyweight object.
class Tree {
  constructor(
    private x: number,
    private y: number,
    private type: TreeType
  ) {}

  draw(canvas: string): void {
    // Delegates the work to the flyweight object, passing the extrinsic state as arguments.
    this.type.draw(canvas, this.x, this.y);
  }
}

// --- 4. Client ---
// The client creates the context objects and passes the intrinsic state to the factory to get the flyweight object.
class Forest {
  private trees: Tree[] = [];

  plantTree(x: number, y: number, name: string, color: string, texture: string): void {
    const type = TreeFactory.getTreeType(name, color, texture);
    const tree = new Tree(x, y, type);
    this.trees.push(tree);
  }

  draw(canvas: string): void {
    for (const tree of this.trees) {
      tree.draw(canvas);
    }
  }
}

// --- Usage ---

const forest = new Forest();
const canvas = "Main Canvas";

// Planting many trees, but only creating a few TreeType (Flyweight) objects.
forest.plantTree(1, 2, "Oak", "Green", "Rough");
forest.plantTree(5, 3, "Oak", "Green", "Rough");
forest.plantTree(10, 8, "Pine", "DarkGreen", "Spiky");
forest.plantTree(15, 12, "Oak", "Green", "Rough");
forest.plantTree(20, 15, "Pine", "DarkGreen", "Spiky");

console.log("\nDrawing the forest:");
forest.draw(canvas);
```

### Pros and Cons
**Pros:**
- You can save lots of RAM, assuming your program has tons of similar objects.

**Cons:**
- You might be trading RAM over CPU cycles when some of the context data needs to be recalculated each time somebody calls a flyweight method.
- The code becomes much more complicated. New team members will always be wondering why the state of an entity was separated in such a way.

---

## Summary
- **Bridge**: Splits a large class or a set of closely related classes into two separate hierarchies—abstraction and implementation—which can be developed independently. Useful for avoiding a Cartesian product of classes.
- **Flyweight**: Optimizes memory usage by sharing common parts of state between multiple objects instead of keeping all of the data in each object. Useful when dealing with a massive number of similar objects.
