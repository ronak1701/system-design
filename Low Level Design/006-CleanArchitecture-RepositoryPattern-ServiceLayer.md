# Clean Architecture: Repository Pattern & Service Layer in TypeScript

## Introduction

While the **Service Layer** pattern (as seen in MVCS) is excellent for extracting business logic from Controllers, applications often still suffer from tight coupling to their databases. If your Service layer makes direct calls to a specific ORM (like Prisma, TypeORM, or Mongoose), swapping out the database, upgrading the ORM SDK, or testing the service completely independently becomes exceptionally difficult.

Enter the **Repository Pattern**. A core pillar of **Clean Architecture**, this pattern introduces a strict abstraction boundary between your Core Business Logic (Service Layer) and your Data Access Logic (Database/ORM infrastructure).

---

## The Core Components

### Beginner Explanation

Imagine you manage a major, highly-profitable retail store:

- The **Service Layer** is you, the Store Manager. You know the exact business rules: how much discount to apply during VIP sales, checking if a customer has a banned status, and ensuring critical inventory doesn't drop below zero.
- The **Database/ORM** is the massive physical Warehouse block out back.
- The **Repository Pattern** is the Head of Logistics (the intermediary).

As the Store Manager, you don't go into the warehouse and operate the heavy forklift yourself (this would be tight coupling). Instead, you simply tell the front-desk of Logistics: _"I need 5 premium laptops."_ You don't care _how_ they get them, whether they use a forklift, a drone, or a hand-truck.

If the corporate office upgrades the physical warehouse entirely, your job as the Store Manager doesn't change at all, because you still just dictate needs to the Head of Logistics.

In modern engineering:

- **Service Layer (Domain):** Pure business logic. Completely unaware of SQL, NoSQL, or any specific database mechanism.
- **Repository Interface:** A strict TypeScript contract defining _what_ data operations are possible (e.g., `findById()`, `save()`).
- **Repository Implementation:** The concrete class that executes the actual ORM or database queries (e.g., `MongoUserRepository` or `PostgresUserRepository`).

---

## ❌ Bad Example: Tight Database Coupling in the Service Layer

In this anti-pattern, the `UserService` directly imports and utilizes a strictly specific database ORM (like Prisma). If you want to switch to Mongoose, change table setups, or write a unit test without spinning up a live Docker database, you are forced to painfully mock the complex ORM library.

```typescript
// ❌ Anti-pattern: The Service is strictly tied to 'Prisma'
import { PrismaClient } from "@prisma/client";

const prisma = new PrismaClient();

class UserService {
  async upgradeToPremium(userId: number) {
    // Direct Data Access (Coupled inherently to Prisma's exact syntax)
    const user = await prisma.user.findUnique({ where: { id: userId } });

    if (!user) {
      throw new Error("User not found");
    }
    if (user.isBanned) {
      throw new Error("Cannot upgrade a banned user");
    }

    // Direct Database Mutation
    const updatedUser = await prisma.user.update({
      where: { id: userId },
      data: { isPremium: true },
    });

    return updatedUser;
  }
}
```

---

## ✅ Good Example 1: Clean Architecture using the Repository Pattern

We aggressively decouple the Service from the database infrastructure by introducing a Repository Interface. The Service will strictly depend on the Interface definition (Dependency Inversion Principle).

### 1. The Domain Entity & Repository Contract (Interface)

This is pure, agnostic TypeScript. No database libraries are imported here.

```typescript
// The pure Domain Object (Entity)
export interface User {
  id: number;
  email: string;
  isBanned: boolean;
  isPremium: boolean;
}

// The Contract that any Data Source implementation MUST safely fulfill
export interface IUserRepository {
  findById(id: number): Promise<User | null>;
  save(user: User): Promise<User>;
}
```

### 2. The Concrete Repository Implementation (Data Access Layer)

This is where the dirty, database-specific query logic lives. We can easily have a `PostgresUserRepository`, a `MongoUserRepository`, or an `InMemoryUserRepository`.

```typescript
// Let's pretend we are using a robust PostgreSQL database
class PostgresUserRepository implements IUserRepository {
  private dbConnection: any; // Simulated DB connection object

  constructor(dbConnection: any) {
    this.dbConnection = dbConnection;
  }

  async findById(id: number): Promise<User | null> {
    // Database-specific raw SQL query goes here
    console.log(
      `[Database Layer] Executing: SELECT * FROM users WHERE id = ${id}`,
    );

    // Simulating a DB read resulting in a Domain User Object
    return { id, email: "alice@vip.com", isBanned: false, isPremium: false };
  }

  async save(user: User): Promise<User> {
    // Database-specific SQL update goes here
    console.log(
      `[Database Layer] Executing: UPDATE users SET isPremium = ${user.isPremium} WHERE id = ${user.id}`,
    );
    return user;
  }
}
```

### 3. The Service Layer (Pure Business Logic)

The Service relies solely on the `IUserRepository` abstraction. It has absolutely zero knowledge of PostgreSQL or structural SQL syntax.

```typescript
class UserService {
  // We strictly inject the Interface, NOT the concrete database implementation!
  constructor(private userRepository: IUserRepository) {}

  async upgradeToPremium(userId: number): Promise<User> {
    // Fetch data using the agnostic Contract abstraction
    const user = await this.userRepository.findById(userId);

    if (!user) {
      throw new Error("User not found");
    }

    // Execute Core Business Logic Rules
    if (user.isBanned) {
      throw new Error("Cannot upgrade a banned user");
    }

    // Mutate state purely in runtime memory
    user.isPremium = true;

    // Persist changes using the agnostic Contract abstraction
    return await this.userRepository.save(user);
  }
}
```

### Wiring it up (Dependency Injection & Testing)

By successfully swapping the implementation we pass directly into the `UserService` constructor, we can effortlessly change databases OR write blazing-fast unit tests without touching a real database.

```typescript
// -------------------------------------------------------------
// 1. In Production: Inject and Use the real Postgres Database
// -------------------------------------------------------------
const postgresDb = {}; // Simulating active Knex/PG connection
const prodRepository = new PostgresUserRepository(postgresDb);

// The Service is blissfully unaware it is speaking to Postgres
const productionService = new UserService(prodRepository);

productionService.upgradeToPremium(1);

// -------------------------------------------------------------
// 2. In Testing: Inject a Mock/In-Memory Repository (No DB needed!)
// -------------------------------------------------------------
class MockUserRepository implements IUserRepository {
  async findById(id: number) {
    return { id, email: "test@test.com", isBanned: false, isPremium: false };
  }
  async save(user: User) {
    return user;
  }
}

const testRepository = new MockUserRepository();

// Test the core logic instantly without spanning Docker containers
const testService = new UserService(testRepository);

testService
  .upgradeToPremium(99)
  .then(() => console.log("Unit Test Execution Passed!"));
```

---

## 🚀 Advanced Example 2: Full Clean Folder Structure & Flow

To deeply demonstrate Clean Architecture in action, let's examine a more complex application: a Ticket Booking System. Proper Clean Architecture relies heavily on layer separation visually expressed through the folder structure.

### 🏗️ 📁 Folder Structure
The folder structure directly reflects the architectural layers:

```text
src/
├── domain/                    (Core layer: Entities and Interfaces)
│   ├── entities/
│   │   └── Booking.ts
│   └── interfaces/
│       └── IBookingRepository.ts
│
├── usecases/                  (Application layer: Business logic/Services)
│   └── BookTicketUseCase.ts
│
├── infrastructure/            (External layer: Database, Frameworks)
│   └── repositories/
│       └── BookingRepository.ts
│
├── presentation/              (Interface layer: Controllers/Resolvers)
│   └── controllers/
│       └── BookTicketController.ts
│
├── container/                 (Dependency Injection)
│   └── container.ts           👈 DI setup
│
└── main/                      (Entry Point)
    └── index.ts               👈 execution start
```

### 🧠 1. Entity (Domain Layer)
Entities encapsulate the most general and high-level enterprise rules. They contain NO frameworks or imports.
```typescript
// domain/entities/Booking.ts
export class Booking {
  constructor(public userId: string, public movieId: string) {}

  isValid() {
    return !!this.userId && !!this.movieId;
  }
}
```

### 🧠 2. Interface (Dependency Inversion)
The Repository Interface belongs to the Domain layer. Outer layers will implement this contract.
```typescript
// domain/interfaces/IBookingRepository.ts
import { Booking } from "../entities/Booking";

export interface IBookingRepository {
  save(booking: Booking): Promise<{ id: string }>;
}
```

### 🧠 3. Use Case (Service Layer)
Use cases contain application-specific business rules. Notice how it strictly relies on the injected interface.
```typescript
// usecases/BookTicketUseCase.ts
import { IBookingRepository } from "../domain/interfaces/IBookingRepository";
import { Booking } from "../domain/entities/Booking";

export class BookTicketUseCase {
  constructor(private repo: IBookingRepository) {}

  async execute(userId: string, movieId: string) {
    const booking = new Booking(userId, movieId);

    if (!booking.isValid()) {
      throw new Error("Invalid booking");
    }

    return this.repo.save(booking);
  }
}
```

### 🧠 4. Infrastructure (Data Access Layer)
The concrete implementation of the database logic.
```typescript
// infrastructure/repositories/BookingRepository.ts
import { IBookingRepository } from "../../domain/interfaces/IBookingRepository";
import { Booking } from "../../domain/entities/Booking";

export class BookingRepository implements IBookingRepository {
  async save(booking: Booking) {
    // Simulating database insertion logic
    return { id: "booking-id-123" };
  }
}
```

### 🧠 5. Controller (Presentation Layer)
Parses external HTTP requests and delegates work to the Use Cases.
```typescript
// presentation/controllers/BookTicketController.ts
import { BookTicketUseCase } from "../../usecases/BookTicketUseCase";

export class BookTicketController {
  constructor(private useCase: BookTicketUseCase) {}

  async handleRequest(req: any) {
    const { userId, movieId } = req.body;

    const result = await this.useCase.execute(userId, movieId);

    return { success: true, bookingId: result.id };
  }
}
```

### 🔥 6. DI Container (Central Wiring)
This file connects everything together, satisfying Dependency Injection principles.
```typescript
// container/container.ts
import { BookingRepository } from "../infrastructure/repositories/BookingRepository";
import { BookTicketUseCase } from "../usecases/BookTicketUseCase";
import { BookTicketController } from "../presentation/controllers/BookTicketController";

export const container = () => {
  const repo = new BookingRepository();
  const useCase = new BookTicketUseCase(repo);
  const controller = new BookTicketController(useCase);

  return { controller };
};
```

### 🚀 7. Entry Point
Run the application!
```typescript
// main/index.ts
import { container } from "../container/container";

(async () => {
  const { controller } = container();

  const fakeRequest = {
    body: { userId: "u1", movieId: "m101" },
  };

  const response = await controller.handleRequest(fakeRequest);

  console.log(response); // Expected: { success: true, bookingId: 'booking-id-123' }
})();
```

### 🔗 Final Flow Diagram
```text
Request
  ↓
Controller (Presentation Layer)
  ↓
UseCase (Application Layer)
  ↓
Interface (Domain Abstraction)
  ↓
Repository (Infrastructure DB Layer)
  ↓
Entity (Domain Rules)
  ↓
Response
```

---

## Summary Cheat Sheet

| Component                | Responsibility                                                                                                             | External Environment Dependency                                            |
| :----------------------- | :------------------------------------------------------------------------------------------------------------------------- | :------------------------------------------------------------------------- |
| **Domain Entities**      | Pure TS Definitions (`interface User`). Define what objects fundamentally look like.                                       | **Zero.** Pure isolated code.                                              |
| **Service Layer**        | The **Business Rules.** Executes algorithms and state-changes on Domain Entities.                                          | **Zero.** Absolutely agnostic to external frameworks or databases.         |
| **Repository Interface** | The Agnostic **Contract.** Defines standardized interaction methods like `find()` and `save()`.                            | **Zero.** Pure typescript abstraction mapping.                             |
| **Concrete Repository**  | The **Data Implementation.** Executes heavy, real queries (SQL, Mongo, API calls) based precisely on the Interface layout. | **High.** Heavily coupled implicitly to the specific database/ORM library. |
