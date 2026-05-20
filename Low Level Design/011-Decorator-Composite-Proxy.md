# Structural Design Patterns: Decorator, Composite, and Proxy

This document continues the exploration of Structural design patterns, focusing on the **Decorator**, **Composite**, and **Proxy** patterns. These patterns provide powerful ways to compose objects and manage access or structure.

---

## 1. Decorator Pattern

### Introduction
The **Decorator** pattern (also known as Wrapper) lets you attach new behaviors to objects by placing these objects inside special wrapper objects that contain the behaviors. It provides a flexible alternative to subclassing for extending functionality dynamically at runtime.

### Real-World Analogy
Think of a plain coffee. Now add milk. Now add sugar. Each addition enhances the original but doesn’t change the base.

The Decorator Pattern works the same way: stacking behaviors while keeping the core intact.

### When to Use
- When you need to assign extra behaviors to objects at runtime without breaking the code that uses these objects.
- When it's awkward or impossible to extend an object's behavior using inheritance (e.g., when a class is `final` or when there are too many combinations of behaviors).

### TypeScript Implementation

```typescript
// decorator-example.ts

// --- 1. Component Interface ---
// The common interface for both wrappers and wrapped objects.
interface Notifier {
  send(message: string): void;
}

// --- 2. Concrete Component ---
// The core object being wrapped.
class EmailNotifier implements Notifier {
  private emailAddresses: string[];

  constructor(emails: string[]) {
    this.emailAddresses = emails;
  }

  send(message: string): void {
    console.log(`Sending Email to ${this.emailAddresses.join(', ')}: ${message}`);
  }
}

// --- 3. Base Decorator ---
// Implements the same interface and delegates the work to the wrapped object.
abstract class NotifierDecorator implements Notifier {
  protected wrapper: Notifier;

  constructor(notifier: Notifier) {
    this.wrapper = notifier;
  }

  send(message: string): void {
    this.wrapper.send(message);
  }
}

// --- 4. Concrete Decorators ---
// Add extra behaviors either before or after delegating to the wrapped object.

class SMSDecorator extends NotifierDecorator {
  send(message: string): void {
    super.send(message); // Delegate to wrapped object
    console.log(`Sending SMS: ${message}`); // Add new behavior
  }
}

class SlackDecorator extends NotifierDecorator {
  send(message: string): void {
    super.send(message);
    console.log(`Sending Slack message: ${message}`);
  }
}

// --- Usage ---

// 1. Basic usage
let notifier: Notifier = new EmailNotifier(["admin@example.com"]);

// 2. Wrap it with SMS capabilities
notifier = new SMSDecorator(notifier);

// 3. Wrap it with Slack capabilities
notifier = new SlackDecorator(notifier);

// The client code uses the decorated object through the common interface.
// It will send Email, then SMS, then Slack.
notifier.send("Server is down!");
```

### Pros and Cons
**Pros:**
- You can extend an object's behavior without making a new subclass.
- You can add or remove responsibilities from an object at runtime.
- You can combine several behaviors by wrapping an object into multiple decorators.
- **Single Responsibility Principle:** You can divide a monolithic class that implements many possible variants of behavior into several smaller classes.

**Cons:**
- It's hard to remove a specific wrapper from the wrappers stack.
- It's hard to implement a decorator in such a way that its behavior doesn't depend on the order in the decorators stack.
- The initial configuration code of layers might look ugly.

---

## 2. Composite Pattern

### Introduction
The **Composite** pattern lets you compose objects into tree structures and then work with these structures as if they were individual objects. It allows clients to treat individual objects and compositions of objects uniformly.

### Real-World Analogy
Think of an army. It consists of divisions, which consist of brigades, which consist of regiments, which consist of squads, which consist of individual soldiers. When a general gives a "march" order, they don't have to address each soldier individually. They give the order to the top-level command, and it trickles down the hierarchy. 

The Composite pattern allows you to treat the entire army, a single brigade, or an individual soldier uniformly when issuing commands.

### When to Use
- When you have to implement a tree-like object structure.
- When you want the client code to treat both simple and complex elements uniformly.

### TypeScript Implementation

```typescript
// composite-example.ts

// --- 1. Component Interface ---
// Describes operations that are common to both simple and complex elements of the tree.
interface FileSystemComponent {
  getName(): string;
  getSize(): number;
  print(indentation?: string): void;
}

// --- 2. Leaf ---
// A basic element of a tree that doesn't have sub-elements.
class FileItem implements FileSystemComponent {
  constructor(private name: string, private size: number) {}

  getName(): string { return this.name; }
  
  getSize(): number { return this.size; }
  
  print(indentation: string = ""): void {
    console.log(`${indentation}- File: ${this.name} (${this.size} KB)`);
  }
}

// --- 3. Composite ---
// An element that has sub-elements (children).
class Directory implements FileSystemComponent {
  private children: FileSystemComponent[] = [];

  constructor(private name: string) {}

  add(component: FileSystemComponent): void {
    this.children.push(component);
  }

  remove(component: FileSystemComponent): void {
    const index = this.children.indexOf(component);
    if (index !== -1) {
      this.children.splice(index, 1);
    }
  }

  getName(): string { return this.name; }

  // Recursively calculate the size of the directory
  getSize(): number {
    return this.children.reduce((total, child) => total + child.getSize(), 0);
  }

  print(indentation: string = ""): void {
    console.log(`${indentation}+ Directory: ${this.name} (Total: ${this.getSize()} KB)`);
    for (const child of this.children) {
      child.print(indentation + "  ");
    }
  }
}

// --- Usage ---

const file1 = new FileItem("document.txt", 15);
const file2 = new FileItem("photo.jpg", 1200);
const file3 = new FileItem("script.js", 5);

const docsFolder = new Directory("Documents");
docsFolder.add(file1);

const mediaFolder = new Directory("Media");
mediaFolder.add(file2);

const rootFolder = new Directory("Root");
rootFolder.add(docsFolder);
rootFolder.add(mediaFolder);
rootFolder.add(file3);

// The client treats files and directories uniformly.
rootFolder.print();
```

### Pros and Cons
**Pros:**
- You can work with complex tree structures more conveniently: use polymorphism and recursion to your advantage.
- **Open/Closed Principle:** You can introduce new element types into the app without breaking the existing code, which now works with the object tree.

**Cons:**
- It might be difficult to provide a common interface for classes whose functionality differs too much.

---

## 3. Proxy Pattern

### Introduction
The **Proxy** pattern lets you provide a substitute or placeholder for another object. A proxy controls access to the original object, allowing you to perform something either before or after the request gets through to the original object.

### Real-World Analogy
Think of a credit card. It's a proxy for a bank account, which is itself a proxy for a pile of cash. Both implement the same interface: they can be used for making payments. 

A proxy acts as a substitute for the real subject. A credit card controls access to the actual funds, provides security checks, and offers convenience compared to carrying physical cash. The Proxy pattern works similarly by controlling access to the underlying object, potentially adding caching, lazy loading, or security checks.

### When to Use
- **Lazy initialization (Virtual Proxy):** When you have a heavyweight object that wastes system resources by being always up, even though you only need it from time to time.
- **Access control (Protection Proxy):** When you want only specific clients to be able to use the service object.
- **Local execution of a remote service (Remote Proxy):** When the service object is located on a remote server.
- **Logging requests (Logging Proxy):** When you want to keep a history of requests to the service object.
- **Caching request results (Caching Proxy):** When you need to cache results of client requests and manage the life cycle of this cache.

### TypeScript Implementation

```typescript
// proxy-example.ts

// --- 1. Subject Interface ---
// Defines the common interface for RealSubject and Proxy.
interface VideoDownloader {
  downloadVideo(url: string): string;
}

// --- 2. Real Subject ---
// The actual object that performs the real work.
class RealVideoDownloader implements VideoDownloader {
  downloadVideo(url: string): string {
    console.log(`[Network Request] Downloading video from ${url}...`);
    // Simulate expensive network call
    return `Video content of ${url}`;
  }
}

// --- 3. Proxy ---
// Contains a reference to the Real Subject and controls access to it.
class CachedVideoDownloaderProxy implements VideoDownloader {
  private downloader: RealVideoDownloader;
  private cache: Record<string, string> = {};

  constructor(downloader: RealVideoDownloader) {
    this.downloader = downloader;
  }

  downloadVideo(url: string): string {
    if (!this.cache[url]) {
      // If it's not in the cache, delegate to the real object
      console.log(`Proxy: Cache miss for ${url}. Delegating to Real Subject.`);
      this.cache[url] = this.downloader.downloadVideo(url);
    } else {
      console.log(`Proxy: Cache hit for ${url}. Returning cached result.`);
    }
    return this.cache[url];
  }
}

// --- Usage ---

const realDownloader = new RealVideoDownloader();
const proxy = new CachedVideoDownloaderProxy(realDownloader);

// 1. First time downloading - goes to the real network
console.log(proxy.downloadVideo("https://example.com/video1"));

// 2. Second time downloading - returns cached result
console.log(proxy.downloadVideo("https://example.com/video1"));

// 3. Different video - goes to the real network
console.log(proxy.downloadVideo("https://example.com/video2"));
```

### Pros and Cons
**Pros:**
- You can control the service object without clients knowing about it.
- You can manage the lifecycle of the service object when clients don't care about it.
- The proxy works even if the service object isn't ready or is not available.
- **Open/Closed Principle:** You can introduce new proxies without changing the service or clients.

**Cons:**
- The code may become more complicated since you need to introduce a lot of new classes.
- The response from the service might get delayed.

---

## Summary
- **Decorator**: Dynamically adds new behaviors to objects without subclassing. Wraps the object to extend its functionality.
- **Composite**: Treats individual objects and compositions of objects uniformly, perfect for tree structures (like UI components or file systems).
- **Proxy**: Controls access to another object. Useful for lazy loading, caching, access control, or logging before delegating to the real object.
