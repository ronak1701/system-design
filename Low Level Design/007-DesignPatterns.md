# Design Patterns (Gang of Four)

## Introduction

In software engineering, a **Design Pattern** is a general, reusable solution to a commonly occurring problem within a given context. Think of it not as a specific piece of code you can copy and paste, but rather as a blueprint or template for solving a design issue.

The concept was popularized by a book titled _Design Patterns: Elements of Reusable Object-Oriented Software_, authored by Erich Gamma, Richard Helm, Ralph Johnson, and John Vlissides—collectively known as the **"Gang of Four" (GoF)**.

The GoF categorized 23 design patterns into three main groups based on their purpose:

1. **Creational Patterns:** Deal with object creation mechanisms.
2. **Structural Patterns:** Deal with object composition and relationships.
3. **Behavioral Patterns:** Deal with communication and assignment of responsibilities between objects.

---

## 1. Creational Patterns (5 Patterns)
Creational patterns provide various object creation mechanisms, which increase flexibility and reuse of existing code while hiding the complex creation logic.

- **Singleton**: Ensures a class has only one instance and provides a global point of access to it (e.g., a single database connection).
- **Factory Method**: Defines an interface for creating an object, but lets subclasses decide which class to instantiate. It defers instantiation to subclasses.
- **Abstract Factory**: Provides an interface for creating families of related or dependent objects without specifying their concrete classes.
- **Builder**: Separates the construction of a complex object from its representation, allowing the same construction process to create various representations step-by-step.
- **Prototype**: Specifies the kinds of objects to create using a prototypical instance, and creates new objects by copying this prototype (cloning).

---

## 2. Structural Patterns (7 Patterns)
Structural patterns explain how to assemble objects and classes into larger structures while keeping these structures flexible and efficient.

- **Adapter**: Converts the interface of a class into another interface clients expect, allowing classes with incompatible interfaces to work together.
- **Bridge**: Decouples an abstraction from its implementation so that the two can vary independently.
- **Composite**: Composes objects into tree structures to represent part-whole hierarchies, allowing clients to treat individual objects and compositions of objects uniformly.
- **Decorator**: Attaches additional responsibilities to an object dynamically without altering its structure. A flexible alternative to subclassing.
- **Facade**: Provides a unified, simplified interface to a set of interfaces in a complex subsystem.
- **Flyweight**: Uses sharing to support large numbers of fine-grained objects efficiently, minimizing memory usage.
- **Proxy**: Provides a surrogate or placeholder for another object to control access to it (e.g., lazy loading, access control).

---

## 3. Behavioral Patterns (11 Patterns)
Behavioral patterns take care of effective communication and the assignment of responsibilities between objects.

- **Chain of Responsibility**: Passes requests along a chain of handlers. Upon receiving a request, each handler decides either to process it or to pass it to the next handler in the chain.
- **Command**: Encapsulates a request as an object, letting you parameterize clients with different requests, queue or log requests, and support undoable operations.
- **Interpreter**: Given a language, defines a representation for its grammar along with an interpreter that uses the representation to interpret sentences in the language.
- **Iterator**: Provides a way to access the elements of a collection sequentially without exposing its underlying representation.
- **Mediator**: Restricts direct communications between the objects and forces them to collaborate only via a mediator object, reducing chaotic dependencies.
- **Memento**: Captures and externalizes an object's internal state without violating encapsulation, so the object can be restored to this state later (undo functionality).
- **Observer**: Defines a one-to-many subscription mechanism to notify multiple objects automatically about any events that happen to the object they're observing.
- **State**: Allows an object to alter its behavior when its internal state changes. It appears as if the object changed its class.
- **Strategy**: Defines a family of algorithms, encapsulates each one into a separate class, and makes their objects interchangeable at runtime.
- **Template Method**: Defines the skeleton of an algorithm in a superclass but lets subclasses override specific steps of the algorithm without changing its structure.
- **Visitor**: Separates an algorithm from an object structure on which it operates, allowing you to add new behaviors to existing class hierarchies without altering any existing code.
