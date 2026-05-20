# Structural Design Patterns: Adapter and Facade

This document explores Structural design patterns, focusing on the **Adapter** and **Facade** patterns. Structural patterns deal with object composition, helping to ensure that if one part of a system changes, the entire structure does not need to change.

---

## 1. Adapter Pattern

### Introduction
The **Adapter** pattern (also known as Wrapper) allows objects with incompatible interfaces to collaborate. It acts as a bridge between two incompatible interfaces by wrapping an existing class with a new interface.

### Real-World Analogy
Imagine you're traveling from the United States to Europe. Your laptop charger uses a Type A plug (used in the US), but European wall sockets expect a Type C plug.

You can’t plug your charger in directly, the interfaces don’t match.

Instead of buying a new charger, you use a travel plug adapter. This device accepts your Type A plug and converts it into a Type C shape that fits into the European socket.

The Adapter pattern does the same thing for software interfaces.

### When to Use
- When you want to use an existing class, but its interface isn't compatible with the rest of your code.
- When you want to reuse several existing subclasses that lack some common functionality that can't be added to the superclass.

### TypeScript Implementation

```typescript
// adapter-example.ts

// --- 1. Target Interface ---
// This is the interface your application expects to work with.
interface Logger {
  log(message: string): void;
}

// --- 2. Adaptee (Incompatible Interface) ---
// This is an existing class or third-party library that has a different interface.
class ThirdPartyCloudLogger {
  sendToServer(msg: string, priority: number): void {
    console.log(`[Cloud Priority ${priority}]: ${msg}`);
  }
}

// --- 3. Adapter ---
// Makes the Adaptee compatible with the Target Interface.
class CloudLoggerAdapter implements Logger {
  private cloudLogger: ThirdPartyCloudLogger;

  constructor(cloudLogger: ThirdPartyCloudLogger) {
    this.cloudLogger = cloudLogger;
  }

  // Translates the Target interface method to the Adaptee's method.
  log(message: string): void {
    // We map the simple log method to the more complex sendToServer method.
    this.cloudLogger.sendToServer(message, 1);
  }
}

// --- Usage ---

// Client code only knows about the Target Interface.
function runApplication(logger: Logger) {
  logger.log("Application started successfully.");
}

// Using a standard, compatible logger (hypothetical)
class ConsoleLogger implements Logger {
  log(message: string): void {
    console.log(`[Console]: ${message}`);
  }
}
runApplication(new ConsoleLogger());

// When we want to use the incompatible ThirdPartyCloudLogger, we use the Adapter.
const adaptee = new ThirdPartyCloudLogger();
const adapter = new CloudLoggerAdapter(adaptee);

runApplication(adapter);
```

### Pros and Cons
**Pros:**
- **Single Responsibility Principle:** You can separate the interface or data conversion code from the primary business logic of the program.
- **Open/Closed Principle:** You can introduce new types of adapters into the program without breaking the existing client code.

**Cons:**
- The overall complexity of the code increases because you need to introduce a set of new interfaces and classes.

---

## 2. Facade Pattern

### Introduction
The **Facade** pattern provides a simplified, higher-level interface to a complex subsystem of classes, a library, or a framework. It hides the complexity of the subsystem and provides a single, easy-to-use entry point.

### Real-World Analogy
Think of a high-end hotel. As a guest (the client), you don't want to individually contact housekeeping for a fresh towel, the restaurant for dinner reservations, and the valet for your car. Instead, you call the Concierge Desk (the Facade).

You make a simple request to the concierge, like "I'd like dinner reservations at 8 PM and my car ready afterwards." The concierge then interacts with all the necessary hotel departments (the subsystem) to fulfill your request.

You, as the guest, are shielded from this internal complexity. The Concierge Desk provides a simplified interface to the hotel's services.

### When to Use
- When you need to have a limited but straightforward interface to a complex subsystem.
- When you want to structure a subsystem into layers.

### TypeScript Implementation

```typescript
// facade-example.ts

// --- 1. Complex Subsystem Components ---

class VideoFile {
  constructor(public filename: string) {}
}

class OggCompressionCodec {
  type = "ogg";
}

class MPEG4CompressionCodec {
  type = "mp4";
}

class CodecFactory {
  extract(file: VideoFile) {
    console.log(`CodecFactory: extracting video audio...`);
    return "codec_data";
  }
}

class BitrateReader {
  static read(filename: string, codec: any) {
    console.log(`BitrateReader: reading file ${filename}...`);
    return "buffer";
  }
  static convert(buffer: string, destinationCodec: any) {
    console.log(`BitrateReader: writing file...`);
    return "converted_buffer";
  }
}

class AudioMixer {
  fix(result: string) {
    console.log("AudioMixer: fixing audio...");
    return "final_video_file";
  }
}

// --- 2. Facade ---
// Provides a simple interface for a complex underlying process.

class VideoConverterFacade {
  convertVideo(filename: string, format: string): string {
    console.log(`VideoConverterFacade: starting conversion to ${format}...`);
    
    const file = new VideoFile(filename);
    const sourceCodec = new CodecFactory().extract(file);
    
    let destinationCodec;
    if (format === "mp4") {
      destinationCodec = new MPEG4CompressionCodec();
    } else {
      destinationCodec = new OggCompressionCodec();
    }
    
    const buffer = BitrateReader.read(filename, sourceCodec);
    let result = BitrateReader.convert(buffer, destinationCodec);
    
    result = new AudioMixer().fix(result);
    
    console.log("VideoConverterFacade: conversion completed.");
    return result;
  }
}

// --- Usage ---

// The client code doesn't need to interact with or understand the complex subsystem.
// It just uses the simple Facade.

const converter = new VideoConverterFacade();
const mp4Video = converter.convertVideo("funny-cats.ogg", "mp4");
```

### Pros and Cons
**Pros:**
- You can isolate your code from the complexity of a subsystem.
- Reduces the number of dependencies between client code and the subsystem.

**Cons:**
- A facade can become a "god object" coupled to all classes of an app if you are not careful.

---

## Summary
- **Adapter**: Use when you need to make existing incompatible interfaces work together. It wraps an object to change its interface.
- **Facade**: Use when you want to provide a simple, easy-to-use interface to a complex subsystem or set of classes. It simplifies the interaction for the client.
