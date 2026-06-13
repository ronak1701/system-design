# Microservices & Load Balancers

This guide explains microservices architectures, the role of API Gateways, and deep dives into Load Balancers and their routing algorithms using simple, everyday analogies.

> **Reference Material:** This guide is based on the article [System Design for Beginners: Everything you need in one article](https://medium.com/@shivambhadani_/system-design-for-beginners-everything-you-need-in-one-article-c74eb702540b) by Shivam Bhadani.

---

## 1. Microservices

In system design, deciding how to structure your codebase and deploy your application is a major architectural choice. The two primary options are **Monolithic** and **Microservices** architectures.

### A. What is Monolith and Microservice?

#### The Monolith (The Swiss Army Knife)
Imagine a **Swiss Army Knife**. It is a single, solid tool that contains a knife, scissors, a bottle opener, and a screwdriver all joined together in one handle.
*   **In Software:** A monolithic application is built as a **single, unified codebase**. All features (User signup, Product catalog, Cart, Payments) are compiled and deployed together on a single server, sharing a single database.
*   **Pros:** Easy to build, test, and deploy at first.
*   **Cons:** If one tool (like the scissors) breaks or gets stuck, the entire knife is unusable.

#### The Microservice (The Toolbox)
Imagine a **Toolbox**. Instead of one combined tool, you have a separate hammer, a separate screwdriver, and a separate pair of scissors.
*   **In Software:** You break the large application into **small, independent programs (services)**. Each service does exactly *one* job (e.g., a dedicated Payment Service, a dedicated Inventory Service) and runs on its own server with its own database.
*   **Pros:** If your scissors break, your hammer and screwdriver still work perfectly. You can swap out the scissors without buying a new toolbox.

```
       Monolithic Architecture                   Microservices Architecture
     +---------------------------+             +--------+  +--------+  +--------+
     |   Single Unified App      |             | User   |  | Order  |  | Payment|
     | (Users, Orders, Payments) |             | Service|  | Service|  | Service|
     +---------------------------+             +--------+  +--------+  +--------+
                   |                               |           |           |
             Single Database                 DB 1 (Users)  DB 2 (Orders) DB 3 (Payments)
```

---

### B. Why do we break our app into microservices?

As an application scales, a monolith becomes hard to manage. We break it into microservices for three major reasons:

1.  **Independent Scaling:**
    *   *Real-World:* On Black Friday, the **Checkout Service** gets 100x more traffic than the **About Us** page.
    *   *System Benefit:* With microservices, you can launch 20 new servers specifically to handle Checkout requests, while leaving the rest of the application running on standard low-cost servers. In a monolith, you would have to duplicate the *entire* application 20 times.
2.  **Independent Deployment:**
    *   *Real-World:* You want to fix a tiny typo on the Payment screen.
    *   *System Benefit:* In microservices, you only deploy the Payment Service update. The rest of the website remains online. In a monolith, you have to rebuild, test, and redeploy the entire application, which risks introducing bugs to unrelated modules and requires website downtime.
3.  **Fault Isolation:**
    *   *Real-World:* The Recommendation algorithm has a memory leak and crashes.
    *   *System Benefit:* With microservices, only the "Recommended Products" box on the homepage goes blank. Customers can still search for items, add them to their cart, and checkout. In a monolith, a memory crash in the recommendation code brings down the entire application.

---

### C. When to use Microservices?

**Rule of Thumb:** Start with a Monolith, and move to Microservices only when you hit a wall.
*   **Start with a Monolith:** Startups need to move fast. Building a monolith is faster, cheaper, and less complex to manage.
*   **Move to Microservices when:**
    1.  **Team Size:** You have 50+ developers working on the same codebase, constantly causing code conflicts.
    2.  **Scale:** Different features have wildly different resource needs (e.g., one page needs high CPU, another needs high memory).
    3.  **Deployment Bottlenecks:** It takes hours to build and deploy a single bug fix because the codebase is too massive.

---

### D. How do clients request in a microservice architecture? (API Gateway)

When an app is split into 10 different microservices running on 10 different servers, how does a user's browser know which server to talk to?

#### The Hotel Front Desk Analogy
Imagine walking into a large hotel. The hotel has a room service department, a laundry department, and a taxi booking desk.
*   Instead of walking through the hallways looking for the kitchen to order food, you simply pick up the room phone and dial **"0" (The Front Desk)**.
*   You tell the receptionist what you need, and they route your call to the kitchen, laundry, or valet.

```
                                  +------------------+
                                  |   Client App     |
                                  +------------------+
                                           |
                                 Requests (/users, /orders)
                                           |
                                           v
                                  +------------------+
                                  |   API Gateway    |  <-- "The Front Desk"
                                  +------------------+
                                   /                \
                             Route /users      Route /orders
                                 /                    \
                                v                      v
                     +--------------------+  +--------------------+
                     |    User Service    |  |   Order Service    |
                     +--------------------+  +--------------------+
```

#### The API Gateway
In a microservice system, the **API Gateway** acts as the Front Desk receptionist:
1.  **Single Entry Point:** Clients send all requests (e.g., `api.mywebsite.com`) to the API Gateway.
2.  **Routing:** The gateway inspects the URL path. If it contains `/payments`, it routes the request to the Payment Service.
3.  **Security & Authentication:** The gateway checks if the user's login token is valid *before* letting the request reach the internal services.
4.  **Rate Limiting:** The gateway counts requests and blocks users or bots sending too many requests per second.

---

## 2. Load Balancer Deep Dive

As your website grows, horizontal scaling adds multiple servers to handle the load. A **Load Balancer** is the traffic cop that directs incoming requests to these servers.

### A. Why do we need the Load Balancer?

#### The Grocery Cashier Analogy
Imagine a massive grocery store with **5 billing cashiers** and a line of 100 customers.
*   **Without a Manager:** Customers naturally crowd around Cashier #1 because they are closest to the entrance. Cashier #1 gets exhausted and slow, while Cashiers #2 to #5 sit idle scrolling on their phones.
*   **With a Manager:** A manager stands at the front of the queue, checking who is free and pointing the next customer to Cashier #2, #3, etc. 

A **Load Balancer** is that manager. It sits between client browsers and your server fleet, distributing requests evenly so that:
*   No single server is overloaded while others sit idle.
*   If Server #3 crashes, the load balancer stops routing traffic to it and sends all requests to the remaining healthy servers.

---

### B. Load Balancer Algorithms

Load balancers use specific algorithms (rules) to decide which server gets the next request:

#### 1. Round Robin
*   **Layman Explanation:** "Taking turns."
*   **How it works:** The load balancer sends the first request to Server A, the second to Server B, the third to Server C, and the fourth back to Server A.
*   **Best For:** Simple setups where all servers have identical hardware specifications and requests require similar processing power.

#### 2. Weighted Round Robin
*   **Layman Explanation:** "Stronger workers get more tasks."
*   **How it works:** If you have an expensive 8-Core server (Server A) and a cheap 2-Core server (Server B), you assign a "weight" of 4 to Server A and 1 to Server B. The load balancer will send 4 requests to Server A for every 1 request it sends to Server B.
*   **Best For:** Environments with a mix of old (weak) and new (strong) hardware.

#### 3. Least Connections
*   **Layman Explanation:** "Go to the shortest line."
*   **How it works:** The load balancer monitors active connections. When a request arrives, it is sent to the server that is currently serving the fewest active users.
*   **Best For:** Applications where requests take vastly different amounts of time to complete (e.g., Server A is handling a 2-hour file download while Server B is loading a 10ms profile page).

#### 4. IP Hash (Consistent Routing)
*   **Layman Explanation:** "Assigned cashiers."
*   **How it works:** The load balancer runs a math formula on the client's IP address (e.g., `192.168.1.5`). The math outputs a number that always maps that specific client to the exact same server (e.g., Server B).
*   **Best For:** Applications that store temporary user session data (like shopping cart details) in the server's local memory, rather than in a shared database.
