# Behavioral Design Patterns: State, Template Method, and Chain of Responsibility

Continuing our exploration of **Behavioral** design patterns, we now focus on patterns that manage state transitions, define algorithmic skeletons, and handle request delegation. These patterns help in creating highly decoupled, maintainable, and flexible systems by defining clear communication patterns and responsibilities among objects.

Here we cover three essential behavioral patterns: **State**, **Template Method**, and **Chain of Responsibility**.

---

## 1. State Pattern

### Introduction
The **State** pattern lets an object alter its behavior when its internal state changes. It appears as if the object changed its class. This pattern extracts state-related behaviors into separate state classes, allowing the original object (context) to delegate work to an instance of these state classes.

### Real-World Analogy
Think about a traffic light. It has three states: red, yellow, and green. The behavior at each state is different: cars stop, cars prepare to stop, or cars go.

Each state knows what it does and when to transition to the next one. Red knows it should eventually become green. Green knows it should eventually become yellow. The traffic light itself just follows whichever state is active.

That is exactly how the State pattern works: the context (traffic light) delegates to the current state, and each state manages its own transitions.

### When to Use
- When you have an object that behaves differently depending on its current state, the number of states is enormous, and the state-specific code changes frequently.
- When you have a class polluted with massive conditionals that alter how the class behaves according to the current values of the class's fields.
- When you have a lot of duplicate code across similar states and transitions of a condition-based state machine.

### TypeScript Implementation

```typescript
// state-example.ts

// --- 1. State Interface ---
// Declares the state-specific methods.
interface AudioPlayerState {
  clickPlay(): void;
  clickNext(): void;
  clickPrevious(): void;
}

// --- 2. Context ---
// Defines the interface of interest to clients. It also maintains a reference to an instance of a State subclass, which represents the current state.
class AudioPlayer {
  private state: AudioPlayerState;

  constructor() {
    // Initializing with a default state
    this.state = new StoppedState(this);
  }

  // The Context allows changing the State object at runtime.
  public transitionTo(state: AudioPlayerState): void {
    console.log(`AudioPlayer: Transition to ${(<any>state).constructor.name}.`);
    this.state = state;
  }

  // The Context delegates part of its behavior to the current State object.
  public clickPlay(): void {
    this.state.clickPlay();
  }

  public clickNext(): void {
    this.state.clickNext();
  }

  public clickPrevious(): void {
    this.state.clickPrevious();
  }
}

// --- 3. Concrete States ---
// Implement various behaviors associated with a state of the Context.
class PlayingState implements AudioPlayerState {
  constructor(private player: AudioPlayer) {}

  clickPlay(): void {
    console.log("Pausing playback.");
    this.player.transitionTo(new PausedState(this.player));
  }

  clickNext(): void {
    console.log("Playing next track.");
  }

  clickPrevious(): void {
    console.log("Playing previous track.");
  }
}

class PausedState implements AudioPlayerState {
  constructor(private player: AudioPlayer) {}

  clickPlay(): void {
    console.log("Resuming playback.");
    this.player.transitionTo(new PlayingState(this.player));
  }

  clickNext(): void {
    console.log("Skipping to next track but remaining paused.");
  }

  clickPrevious(): void {
    console.log("Skipping to previous track but remaining paused.");
  }
}

class StoppedState implements AudioPlayerState {
  constructor(private player: AudioPlayer) {}

  clickPlay(): void {
    console.log("Starting playback from the beginning.");
    this.player.transitionTo(new PlayingState(this.player));
  }

  clickNext(): void {
    console.log("Action ignored. Player is stopped.");
  }

  clickPrevious(): void {
    console.log("Action ignored. Player is stopped.");
  }
}

// --- Usage ---

console.log("--- Client interacts with Context ---");
const player = new AudioPlayer();

player.clickPlay();     // Starts playback, transitions to PlayingState
player.clickNext();     // Plays next track (PlayingState behavior)
player.clickPlay();     // Pauses playback, transitions to PausedState
player.clickNext();     // Skips but remains paused (PausedState behavior)
```

### Pros and Cons
**Pros:**
- **Single Responsibility Principle:** Organize the code related to particular states into separate classes.
- **Open/Closed Principle:** Introduce new states without changing existing state classes or the context.
- Simplifies the code of the context by eliminating bulky state machine conditionals (`if/else` or `switch`).

**Cons:**
- Applying the pattern can be overkill if a state machine has only a few states or rarely changes.

---

## 2. Template Method Pattern

### Introduction
The **Template Method** pattern defines the skeleton of an algorithm in the superclass but lets subclasses override specific steps of the algorithm without changing its structure. It breaks down an algorithm into a series of steps, turning these steps into methods, and putting a series of calls to these methods inside a single "template method".

### Real-World Analogy
Think of a base cake recipe as the Template Method.

The recipe defines the overall flow, step by step:

- Preheat the oven (common step)
- Prepare the batter (varies by cake type, such as chocolate or vanilla; abstract step)
- Pour the batter into a pan (common step)
- Bake for X minutes (common step; X can be a hook or configurable value)
- Let the cake cool (common step)
- Frost the cake (optional step; hook method)

The key idea is that the sequence is fixed by the general recipe. Specific cake types (subclasses) only implement what differs, mainly how the batter is prepared, and they can optionally override the frosting step if they want a custom finish.

### When to Use
- When you want to let clients extend only particular steps of an algorithm, but not the whole algorithm or its structure.
- When you have several classes that contain almost identical algorithms with some minor differences. As a result, you might need to modify all classes when the algorithm changes.

### TypeScript Implementation

```typescript
// template-method-example.ts

// --- 1. Abstract Class ---
// Declares methods that act as steps of an algorithm, as well as the actual template method which calls these methods in a specific order.
abstract class DataMiner {
  
  // The template method defines the skeleton of an algorithm.
  // In TypeScript, we can't enforce 'final' on methods directly like in Java, 
  // but conceptually, subclasses shouldn't override this method.
  public mineData(path: string): void {
    const file = this.openFile(path);
    const rawData = this.extractData(file);
    const data = this.parseData(rawData);
    const analysis = this.analyzeData(data);
    this.sendReport(analysis);
    this.closeFile(file);
  }

  // These operations already have default implementations.
  protected openFile(path: string): string {
    console.log(`Opening file: ${path}`);
    return "file_reference";
  }

  protected closeFile(file: string): void {
    console.log(`Closing file: ${file}`);
  }

  protected analyzeData(data: string): string {
    console.log("Analyzing parsed data...");
    return "analysis_report";
  }

  protected sendReport(report: string): void {
    console.log("Sending report...");
  }

  // These operations have to be implemented in subclasses.
  protected abstract extractData(file: string): string;
  protected abstract parseData(rawData: string): string;
}

// --- 2. Concrete Classes ---
// Override all or some of the steps, but not the template method itself.
class PDFDataMiner extends DataMiner {
  protected extractData(file: string): string {
    console.log("Extracting data from PDF...");
    return "pdf_raw_data";
  }

  protected parseData(rawData: string): string {
    console.log("Parsing PDF data into structured format...");
    return "pdf_parsed_data";
  }
}

class CSVDataMiner extends DataMiner {
  protected extractData(file: string): string {
    console.log("Extracting data from CSV...");
    return "csv_raw_data";
  }

  protected parseData(rawData: string): string {
    console.log("Parsing CSV data into structured format...");
    return "csv_parsed_data";
  }

  // Overriding a default hook/step
  protected analyzeData(data: string): string {
    console.log("Analyzing CSV specific data structures...");
    return "csv_analysis_report";
  }
}

// --- Usage ---

console.log("--- Mining Data from PDF ---");
const pdfMiner = new PDFDataMiner();
pdfMiner.mineData("/docs/report.pdf");

console.log("\n--- Mining Data from CSV ---");
const csvMiner = new CSVDataMiner();
csvMiner.mineData("/docs/data.csv");
```

### Pros and Cons
**Pros:**
- You can let clients override only certain parts of a large algorithm, making them less affected by changes that happen to other parts of the algorithm.
- You can pull the duplicate code into a superclass.

**Cons:**
- Some clients may be limited by the provided skeleton of an algorithm.
- You might violate the *Liskov Substitution Principle* by suppressing a default step implementation via a subclass.
- Template methods tend to be harder to maintain the more steps they have.

---

## 3. Chain of Responsibility Pattern

### Introduction
The **Chain of Responsibility** pattern lets you pass requests along a chain of handlers. Upon receiving a request, each handler decides either to process the request or to pass it to the next handler in the chain.

### Real-World Analogy
Think about calling customer support. You explain your issue to the first-level agent. If they can solve it, great. If not, they escalate to a second-level specialist. That specialist might handle it or escalate further to a manager, who might escalate to engineering.

You, the caller, do not decide who handles your issue. You just start at the beginning, and the request moves through the chain until someone resolves it.

The Chain of Responsibility pattern works the same way: the client sends a request to the first handler, and it flows through the chain until one handler processes it or the chain ends.

### When to Use
- When your program is expected to process different kinds of requests in various ways, but the exact types of requests and their sequences are unknown beforehand.
- When it's essential to execute several handlers in a particular order.
- When the set of handlers and their order are supposed to change at runtime.

### TypeScript Implementation

```typescript
// chain-of-responsibility-example.ts

// --- 1. Handler Interface ---
// Declares a method for building the chain of handlers and a method for executing a request.
interface Handler {
  setNext(handler: Handler): Handler;
  handle(request: string): string | null;
}

// --- 2. Base Handler ---
// Optional class where you can put the boilerplate code that's common to all handler classes.
abstract class AbstractHandler implements Handler {
  private nextHandler: Handler | null = null;

  public setNext(handler: Handler): Handler {
    this.nextHandler = handler;
    // Returning a handler from here will let us link handlers in a convenient way like this:
    // monkey.setNext(squirrel).setNext(dog);
    return handler;
  }

  public handle(request: string): string | null {
    if (this.nextHandler) {
      return this.nextHandler.handle(request);
    }
    return null;
  }
}

// --- 3. Concrete Handlers ---
// Contain the actual code for processing requests. Upon receiving a request, each handler must decide whether to process it and whether to pass it along the chain.
class AuthHandler extends AbstractHandler {
  public handle(request: string): string | null {
    if (request === "unauthorized_request") {
      return "AuthHandler: Access denied. Unauthenticated user.";
    }
    console.log("AuthHandler: Authentication successful. Passing to next...");
    return super.handle(request);
  }
}

class ValidationHandler extends AbstractHandler {
  public handle(request: string): string | null {
    if (request === "invalid_data_request") {
      return "ValidationHandler: Bad Request. Data validation failed.";
    }
    console.log("ValidationHandler: Data validated. Passing to next...");
    return super.handle(request);
  }
}

class CacheHandler extends AbstractHandler {
  public handle(request: string): string | null {
    if (request === "cached_request") {
      return "CacheHandler: Returning cached response.";
    }
    console.log("CacheHandler: Cache miss. Passing to next...");
    return super.handle(request);
  }
}

// --- Usage ---

// Client code sets up the chain.
const auth = new AuthHandler();
const validation = new ValidationHandler();
const cache = new CacheHandler();

auth.setNext(validation).setNext(cache);

// The client can send a request to any handler in the chain, not just the first one.
function clientCode(handler: Handler, requests: string[]) {
  for (const req of requests) {
    console.log(`\nClient: Sending request '${req}'`);
    const result = handler.handle(req);
    if (result) {
      console.log(`  ${result}`);
    } else {
      console.log(`  Request '${req}' was left unhandled.`);
    }
  }
}

const requests = [
  "unauthorized_request",
  "invalid_data_request",
  "cached_request",
  "valid_new_request"
];

// Sending requests through the complete chain
clientCode(auth, requests);
```

### Pros and Cons
**Pros:**
- You can control the order of request handling.
- **Single Responsibility Principle:** You can decouple classes that invoke operations from classes that perform operations.
- **Open/Closed Principle:** You can introduce new handlers into the app without breaking the existing client code.

**Cons:**
- Some requests may end up unhandled if they reach the end of the chain without being processed.

---

## Summary
- **State**: Allows an object to alter its behavior when its internal state changes. It encapsulates state-specific behaviors into individual classes.
- **Template Method**: Defines the skeleton of an algorithm in a superclass, allowing subclasses to override specific steps without altering the overall algorithm's structure.
- **Chain of Responsibility**: Passes a request along a chain of potential handlers until one of them handles it, decoupling the sender of a request from its receivers.
