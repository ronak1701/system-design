# Event-Driven Architecture & Distributed Systems

This guide explains Event-Driven Architecture (EDA), key event patterns, and the fundamentals of Distributed Systems using simple, everyday analogies to make these core system design concepts easy for beginners to understand.

> **Reference Material:** This guide is based on the article [System Design for Beginners: Everything you need in one article](https://medium.com/@shivambhadani_/system-design-for-beginners-everything-you-need-in-one-article-c74eb702540b) by Shivam Bhadani.

---

## 1. Event-Driven Architecture (EDA)

Traditional systems communicate using **Request-Response** (usually via HTTP APIs). If Service A wants Service B to do something, Service A calls Service B directly and waits for a response. 

**Event-Driven Architecture (EDA)** is a software design pattern where services communicate by emitting and reacting to **Events** instead of directly calling each other.

### A. Introduction to EDA

In EDA, we think of the system in terms of things that have already happened.
*   **What is an Event?** An Event is a statement of fact about something that occurred in the system. It is written in the past tense (e.g., `OrderPlaced`, `PaymentFailed`, `UserRegistered`).
*   **The Actor Roles:**
    *   **Event Producer (Publisher):** The service that detects a change or performs an action and shouts, *"Hey, this event just happened!"*
    *   **Event Router (Broker):** The intermediary that receives events and delivers them to anyone who is listening (e.g., Apache Kafka, RabbitMQ, AWS EventBridge).
    *   **Event Consumer (Subscriber):** The services that listen for specific events and do something in response when they occur.

#### The Group Chat Party Analogy

Imagine you want to invite your friends to a party this Saturday.

*   **Request-Response (Traditional):** 
    You call Friend A: *"Are you coming? Bring chips."* 
    You call Friend B: *"Are you coming? Bring soda."* 
    You call Friend C: *"Are you coming? Bring music."*
    *If Friend B doesn't pick up the phone, your entire plan stalls, and you waste time repeating yourself to everyone.*

*   **Event-Driven (EDA):**
    You post a message in your group chat (The Event Broker): **"I am hosting a party on Saturday at 8 PM! (The Event)"**
    You put your phone down and go about your day.
    *   Friend A sees the message and buys chips.
    *   Friend B sees the message and RSVP's.
    *   Friend C sees the message and prepares a playlist.
    
    *You didn't have to call them individually or tell them what to do. You just announced a fact (an event), and they reacted to it independently.*

---

### B. Why Use EDA?

1.  **Extreme Decoupling:** The producer of the event doesn't know who is listening, how many services are listening, or what they will do with the event. If you delete a consumer service, the producer doesn't break.
2.  **Easy Extensibility (Plug-and-Play):** If you want to add a new feature, you don't need to modify the code of the producer. 
    *   *Example:* If your "Order Service" emits an `OrderPlaced` event, and you want to add a new "Recommendation Service" to analyze purchases, you just point the new service to subscribe to `OrderPlaced`. The "Order Service" code remains completely untouched.
3.  **Improved Performance:** The user doesn't have to wait for five backend systems to finish processing. Once the primary action is recorded as an event, the user gets an instant confirmation, while other systems process the event in the background.

```
Synchronous Request-Response (Slow & Coupled):
[User] -> [Order Service] -> (wait) -> [Inventory Service] -> (wait) -> [Notification Service]
                                                                        
Asynchronous Event-Driven (Fast & Decoupled):
[User] -> [Order Service] -> Emits "OrderPlaced" Event
                                 |
                                 +--> [Inventory Service] (Reacts at own pace)
                                 |
                                 +--> [Notification Service] (Reacts at own pace)
```

---

### C. Event Notification Patterns

When designing an EDA, you must decide how much data to put inside the event message. There are two primary patterns:

#### 1. Simple Event Notification

In this pattern, the event is extremely lightweight. It only contains a notification that something happened, along with a reference/ID to look up details.

*   *Example Payload:*
    ```json
    {
      "event_type": "OrderPlaced",
      "order_id": 98765
    }
    ```
*   **How it works:** When the consumer receives this event, it knows order `98765` was placed, but it doesn't know the buyer's name, the price, or the items. The consumer must make a synchronous API callback to the publisher (e.g., `GET /orders/98765`) to fetch the full details.

##### The Package Pickup Slip Analogy
Imagine the post office puts a **small yellow slip** in your mailbox. 
* The slip only says: *"You have a package with ID #98765 waiting for you."*
* You cannot open the slip to get your package. You must walk to the post office, show the slip, and pick up the actual box (calling back the source API).

*   **Pros:** Small payload size (cheap to send); secure (only consumers with API authorization can fetch the details).
*   **Cons:** High callback traffic. If 10 consumer services react to the event, all 10 will bombard the publisher's API at the same time to fetch the details, potentially overloading the database.

---

#### 2. Event-Carried State Transfer

In this pattern, the event message contains **all the data** the consumer needs to do its work. The state of the resource is carried inside the event.

*   *Example Payload:*
    ```json
    {
      "event_type": "OrderPlaced",
      "order_id": 98765,
      "customer_email": "john@example.com",
      "total_amount": 149.99,
      "items": [
        {"item_name": "Wireless Headphones", "quantity": 1}
      ],
      "shipping_address": "123 Main St, New York, NY"
    }
    ```
*   **How it works:** The consumer reads the event and immediately processes it. It does not need to call the publisher's API because all required information is already inside the message.

##### The Direct Home Delivery Analogy
Instead of a yellow slip, the mail carrier delivers the **actual cardboard package** directly to your doorstep. You open it and use the contents immediately without going anywhere else.

*   **Pros:** Zero callback traffic; high performance; consumer services can still function even if the source publisher service goes completely offline.
*   **Cons:** Larger payload size; data replication (multiple services keep copies of the same data); schema management is harder (if you change the order database schema, you must update the event payload structure carefully to avoid breaking consumers).

---

## 2. Distributed Systems

A **Distributed System** is a collection of independent computers (nodes) that communicate over a network and work together to appear to the end user as a single, unified computer.

---

### A. Why Do We Use Distributed Systems?

Single servers have physical limits (CPU, RAM, storage). When your application grows to millions of users, a single server cannot handle the load. We distribute systems to achieve:
1.  **Horizontal Scalability:** Instead of buying a $50,000 supercomputer, we buy fifty $1,000 standard servers and share the work among them.
2.  **High Availability & Fault Tolerance:** If you have one server and it crashes, your app goes down. If you have 5 distributed servers and one crashes, the other 4 keep running, and your users never notice.
3.  **Lower Latency:** Placing servers in different geographic locations (e.g., US, Europe, Asia) so users can connect to the server physically closest to them for faster load times.

---

### B. Core Challenges of Distributed Systems

While distributed systems are highly scalable, they introduce complex problems that don't exist on a single computer.

#### 1. The Clock Synchronization Problem
In a single computer, the CPU has a physical clock that determines the exact order of events. In a distributed system, every server has its own hardware clock. 
*   Because of temperature changes, hardware age, and network drift, **clocks on different servers are never perfectly synchronized**.
*   *Why this matters:* If Server A writes a record at what it thinks is `10:00:01` and Server B updates that same record at what it thinks is `10:00:00` (due to clock drift), the system might overwrite the newer change with the older one because of incorrect timestamps.

#### 2. Network Partitions (The Broken Cable Problem)
In a distributed system, servers rely on the network to talk. Networks are inherently unreliable. Cables get cut, routers fail, and Wi-Fi drops.
*   A **Network Partition** occurs when some servers in a cluster can still talk to each other, but they lose connection to another group of servers in the same cluster.
*   The system splits into "islands" that cannot communicate.

#### 3. Consistency vs. Availability (The CAP Theorem)
When a network partition happens, a distributed system must make a choice:
*   **Choose Consistency (CP):** The system refuses to write updates on the partitioned nodes because it cannot guarantee all nodes will have the same data. It returns an error to the user (Sacrificing Availability).
*   **Choose Availability (AP):** The system allows updates on any node, even if it cannot sync with the others. The system remains online, but different users might see different data (Sacrificing Consistency).

#### The Pizza Shop Chain Analogy

Imagine you own a chain of pizza shops called **PizzaNet** with branches in **New York** and **London**. You offer a loyalty points system where customers get a free pizza after earning 100 points.

```
     [ New York Branch ]  <--- Network Connection --->  [ London Branch ]
     * Has Points DB                                     * Has Points DB
     * Customer earns points                             * Customer spends points
```

*   **The Happy Path (Connected):**
    A customer buys a pizza in New York and earns 20 points. New York immediately sends a message over the internet to London: *"Add 20 points to John's account."* Both databases are in sync.

*   **The Network Partition (Cables Cut):**
    An undersea internet cable breaks. The New York branch cannot talk to the London branch. John walks into the New York branch and wants to spend his loyalty points. 
    
    You have two options:
    
    1.  **Option A (Prioritize Consistency - CP):** 
        The New York manager says, *"Sorry John, our connection to London is down. We cannot verify your points balance right now, so we cannot let you redeem points."* 
        *Result:* The data remains perfectly correct (Consistent), but John is unhappy because the service is offline (Unavailable).
        
    2.  **Option B (Prioritize Availability - AP):** 
        The New York manager says, *"Welcome John! Our connection is down, but we will trust you. You can spend your points and get a free pizza."*
        *Result:* The system is fully operational (Available), but John might run over to London and spend the exact same points again before the connection recovers, leading to out-of-sync databases (Inconsistent).
