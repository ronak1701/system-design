# Auxiliary Design Principles in TypeScript: DRY, KISS, and YAGNI

## Introduction
While the SOLID principles provide a robust framework for object-oriented design, everyday software development is also heavily guided by three highly pragmatic aphorisms: **DRY**, **KISS**, and **YAGNI**. These principles focus on code maintainability, cognitive load reduction, and preventing over-engineering.

---

## 1. DRY (Don't Repeat Yourself)
**"Every piece of knowledge must have a single, unambiguous, authoritative representation within a system."**

### Beginner Explanation
If you find yourself copy-pasting the exact same code in multiple places, you are violating DRY. The problem with copy-pasted code is the maintenance burden: if you find a bug or need to update the logic, you have to track down and change it everywhere. Instead, extract that logic into one centralized place (like a function, utility class, or base class) and reuse it.

### ❌ Bad Example: Copy-Pasted Logic
```typescript
class AdminService {
  createUser(email: string) {
    // Validation logic duplicated!
    if (!email.includes('@') || email.length < 5) {
      throw new Error("Invalid email format");
    }
    console.log(`Admin created: ${email}`);
  }
}

class GuestService {
  createGuest(email: string) {
    // Validation logic duplicated!
    if (!email.includes('@') || email.length < 5) {
      throw new Error("Invalid email format");
    }
    console.log(`Guest created: ${email}`);
  }
}
```

### ✅ Good Example: Reusing Centralized Logic
We consolidate the validation logic into a single, highly testable place.
```typescript
class EmailValidator {
  static isValid(email: string): boolean {
    return email.includes('@') && email.length >= 5;
  }
}

class AdminService {
  createUser(email: string) {
    if (!EmailValidator.isValid(email)) throw new Error("Invalid email format");
    console.log(`Admin created: ${email}`);
  }
}

class GuestService {
  createGuest(email: string) {
    if (!EmailValidator.isValid(email)) throw new Error("Invalid email format");
    console.log(`Guest created: ${email}`);
  }
}
```

### 🧠 Advanced Perspective
DRY isn't merely about text duplication; it's about **Knowledge Duplication**. Sometimes, two pieces of code look identical but belong to entirely different business domains (accidental duplication). In such cases, forcing them to share a utility function tightly couples two modules that shouldn't be coupled. True DRY implies that a change in a specific business rule should require modifying only one logical place in your system. If two identical-looking functions change for different reasons over time, they were never truly violating DRY in the first place!

---

## 2. KISS (Keep It Simple, Stupid)
**"Most systems work best if they are kept simple rather than made complicated."**

### Beginner Explanation
Don't write highly complex, abstract code just to show off your skills. Code is read far more often than it is written. Simple, straightforward code is easier to debug, test, and maintain. If there is a direct way to solve a problem, heavily prefer it over an overly generalized, complex architecture.

### ❌ Bad Example: Over-Engineering a Simple Problem
Here, we abstract a simple age check into a Factory and an Interface unnecessarily.
```typescript
interface IAgeChecker {
  check(age: number): boolean;
}

class AdultAgeChecker implements IAgeChecker {
  public check(age: number): boolean {
    return age >= 18;
  }
}

class AgeCheckerFactory {
  static createChecker(type: string): IAgeChecker {
    if (type === 'adult') return new AdultAgeChecker();
    throw new Error('Unknown checker type');
  }
}

class User {
  constructor(public age: number) {}
}

// Consuming this logic is exhausting
const user = new User(20);
const checker = AgeCheckerFactory.createChecker('adult');
const isAdult = checker.check(user.age);
```

### ✅ Good Example: Simple, Readable Logic
The logic directly expresses the intent without indirection.
```typescript
class User {
  constructor(public age: number) {}

  isAdult(): boolean {
    return this.age >= 18;
  }
}

const user = new User(20);
const isAdult = user.isAdult();
```

### 🧠 Advanced Perspective
"Simple" does not mean "Easy". Keeping an architecture simple as business requirements scale requires deep domain knowledge, relentless refactoring, and restraint. KISS warns against **Premature Abstraction**. When designing enterprise systems, apply design patterns (like Factory, Strategy, or Decorator) *only* when the problem explicitly demands it. Unnecessary layers of indirection increase cognitive load and obscure the application's actual control flow, providing zero architectural benefit.

---

## 3. YAGNI (You Aren't Gonna Need It)
**"Always implement things when you actually need them, never when you just foresee that you need them."**

### Beginner Explanation
Imagine packing for a 2-day beach trip and bringing snow boots "just in case." You're carrying extra weight for no reason. In coding, don't build features, database columns, or add functional parameters today because you *think* you might need them next month. Build exactly what is required for the current task. Extra, unused code means compiling extra logic, writing extra tests, maintaining potential bugs, and slowing down the team.

### ❌ Bad Example: Coding for the Unknown Future
Building a robust export system supporting multiple unused formats, anticipating a "future requirement" that the Product Manager hasn't actually requested.
```typescript
class ReportGenerator {
  // We only require PDF for the current sprint.
  // But let's build CSV, XML, and JSON just in case they ask later!
  generate(data: any, format: 'pdf' | 'csv' | 'xml' | 'json') {
    switch (format) {
      case 'pdf':
        console.log("Generating PDF...");
        break;
      case 'csv':
        console.log("Generating CSV... (Not requested, wasting time)");
        break;
      case 'xml':
        console.log("Generating XML... (Not requested, writing tests for nothing)");
        break;
      case 'json':
        console.log("Generating JSON... (Not requested, adding tech debt)");
        break;
    }
  }
}
```

### ✅ Good Example: Coding for Today's Requirements
```typescript
class ReportGenerator {
  // Implement exactly what the business asked for right now.
  generatePDF(data: any) {
    console.log("Generating PDF...");
  }
}

// When and if the business later asks for CSV, we can easily add it.
// Half the time, the required features change before we even get to them!
```

### 🧠 Advanced Perspective
YAGNI goes hand-in-hand with Agile methodologies and **Lean Software Development**. Speculative generality often leads to dead code and design debt. It locks your system into a rigid design based on assumptions. However, "building for today" does not excuse writing rigid, intertwined code. Rely on foundational practices (like SOLID) so that your architecture easily allows refactoring. This makes it effortless to add those speculative features *when* the requirement actually arrives, rather than pre-building them.

---

## Summary Cheat Sheet

| Principle | Meaning | Core Takeaway |
| :--- | :--- | :--- |
| **DRY** | Don't Repeat Yourself | Centralize business logic to avoid identical maintenance across multiple files. |
| **KISS** | Keep It Simple, Stupid | Avoid premature abstractions and over-engineering. Straightforward code > clever code. |
| **YAGNI** | You Aren't Gonna Need It | Do not build speculative features. Implement only what is required today. |
