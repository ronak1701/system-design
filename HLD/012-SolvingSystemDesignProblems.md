# How to Solve Any System Design Problem

This guide outlines a step-by-step framework to approach and solve any system design problem or interview question. It translates complex architectural engineering steps into a structured roadmap, using simple analogies and beginner-friendly guidelines.

> **Reference Material:** This guide is based on the article [System Design for Beginners: Everything you need in one article](https://medium.com/@shivambhadani_/system-design-for-beginners-everything-you-need-in-one-article-c74eb702540b) by Shivam Bhadani.

---

## The System Design Roadmap: A 7-Step Framework

When asked to design a system (e.g., *"Design Twitter"* or *"Design a URL Shortener"*), beginners often make the mistake of immediately drawing databases and servers. Without knowing the scale, features, or requirements, this approach leads to failure.

Instead, follow this structured, chronological 7-step roadmap.

```
+-------------------------------------------------------------+
|  Step 1: Clarify Requirements (Functional & Non-Functional) |
+------------------------------+------------------------------+
                               |
                               v
+-------------------------------------------------------------+
|  Step 2: Back-of-the-Envelope Estimations (Traffic & Storage)|
+------------------------------+------------------------------+
                               |
                               v
+-------------------------------------------------------------+
|  Step 3: API Design (Define Endpoints & Inputs)             |
+------------------------------+------------------------------+
                               |
                               v
+-------------------------------------------------------------+
|  Step 4: Database Schema Design (SQL vs NoSQL decisions)    |
+------------------------------+------------------------------+
                               |
                               v
+-------------------------------------------------------------+
|  Step 5: High-Level Architecture (Core System Map)          |
+------------------------------+------------------------------+
                               |
                               v
+-------------------------------------------------------------+
|  Step 6: Deep Dive & Scaling (Caches, Queues, Replicas)     |
+------------------------------+------------------------------+
                               |
                               v
+-------------------------------------------------------------+
|  Step 7: Trade-offs & Failures (CAP Theorem & Resiliency)   |
+-------------------------------------------------------------+
```

---

### Step 1: Understand the Scope & Clarify Requirements
*Time allocation: 5–10 minutes*

**The House Analogy:** Never start laying bricks before asking the buyer if they want a small doghouse or a 100-story luxury skyscraper. The blueprints are completely different.

In this step, ask questions to define the boundaries of your system. Split these into two categories:

#### A. Functional Requirements (What the system DOES)
List the core features the user can interact with. Keep this list small (3 to 4 features) so you can cover them in detail.
*   *Example for Twitter:* Can users post tweets? Can users follow other users? Can users view a timeline of tweets? Can tweets contain images/videos?

#### B. Non-Functional Requirements (How WELL the system does it)
Define the constraints and quality of the system.
*   *Scale:* How many Daily Active Users (DAU) will use the app? (e.g., 100 million).
*   *Availability:* Does the app need to be online 24/7/365? (e.g., 99.99% uptime, "four nines").
*   *Latency:* How fast must the app respond? (e.g., loading the timeline should take less than 200 milliseconds).
*   *Consistency:* If a user posts a tweet, does everyone globally need to see it immediately (Strong Consistency), or is a delay of a few seconds acceptable (Eventual Consistency)?

---

### Step 2: Back-of-the-Envelope Estimation
*Time allocation: 5 minutes*

**The Wedding Analogy:** Before cooking a meal, you must know if you are feeding 4 family members or a wedding guest list of 800. It determines the size of the pots and the amount of ingredients you buy.

Use simple math to estimate the scale of your hardware requirements. Focus on two main metrics:

1.  **Traffic (Requests Per Second - RPS):**
    *   If you have 10 million Daily Active Users (DAU) and each user views their feed 5 times a day:
        $$\text{Total Daily Read Requests} = 10,000,000 \times 5 = 50,000,000 \text{ reads/day}$$
    *   To find requests per second, divide by the number of seconds in a day (approx. 100,000):
        $$\text{Read RPS} = \frac{50,000,000}{100,000} = 500 \text{ requests/second}$$
2.  **Storage (Hard Drive Capacity):**
    *   If users write 1 million tweets per day, and each tweet contains text (approx. 100 bytes):
        $$\text{Daily Storage} = 1,000,000 \times 100 \text{ bytes} = 100 \text{ Megabytes/day}$$
        $$\text{Yearly Storage} = 100 \text{ MB/day} \times 365 \text{ days} \approx 36.5 \text{ Gigabytes/year}$$
    *   *Conclusion:* You don't need a complex database cluster for storage; 36.5 GB easily fits on a single hard drive! (However, if they upload video, the storage needs will skyrocket, requiring Blob Storage like AWS S3).

---

### Step 3: API Design
*Time allocation: 5 minutes*

**The Restaurant Analogy:** Design the menu before cooking the food. The menu defines what the customer can order (endpoints) and what they will get in return.

Write down the primary API contracts that the frontend (client) will use to communicate with the backend. Use standard RESTful conventions:

*   **Endpoint to post a tweet:**
    *   `POST /v1/tweets`
    *   *Parameters (JSON Input):* `{ "user_id": "123", "content": "Hello World!" }`
    *   *Return:* `{ "tweet_id": "987", "status": "success", "created_at": "2026-05-31T12:00:00Z" }`
*   **Endpoint to read a feed:**
    *   `GET /v1/feed?user_id=123&limit=20`
    *   *Return:* An array of tweet objects.

---

### Step 4: Database Schema Design
*Time allocation: 5 minutes*

**The Office Filing Cabinet Analogy:** Determine how files are categorized and stored. Will you use structured filing cabinets (SQL tables) or flexible storage boxes (NoSQL documents)?

1.  **Define the Tables/Entities:**
    *   `Users`: `user_id` (Primary Key), `username`, `email`, `created_on`
    *   `Tweets`: `tweet_id` (Primary Key), `user_id` (Foreign Key), `content`, `created_on`
2.  **SQL vs. NoSQL Decision:**
    *   *Choose SQL (Relational)* if you need strict data integrity, complex queries, or financial transactions (ACID compliance).
    *   *Choose NoSQL (Non-Relational)* if you are storing unstructured data (like chat messages), need massive scale with simple read/writes, or require high write speeds.

---

### Step 5: High-Level Architecture Design
*Time allocation: 5–10 minutes*

**The City Map Analogy:** Draw a map showing the entry highways, toll booths, and main streets connecting buildings.

Outline the end-to-end flow of requests. Sketch or describe the core components:
1.  **Client:** The web browser or mobile app.
2.  **DNS (Domain Name System):** Resolves the website URL to an IP address.
3.  **Load Balancer:** Distributes incoming internet traffic across multiple servers.
4.  **Web Servers (API Gateways):** Receives the requests, validates user authentication, and routes them.
5.  **Databases:** Persists the user and application data.

---

### Step 6: Deep Dive into Bottlenecks & Scale
*Time allocation: 15 minutes*

**The Highway Traffic Jam Analogy:** Find where traffic slows down and fix it by adding lanes, building bridges (caching), or creating detour routes (queuing).

This is where you make the system scalable by addressing specific failures:
*   **Problem:** The database is getting crushed by read traffic.
    *   *Solution:* Place a **Cache (Redis)** in front of the database to store popular data in memory (RAM). Add **Read Replicas** so reads are handled by follower nodes.
*   **Problem:** Images/Videos are loading slowly for users far away from your servers.
    *   *Solution:* Use a **Content Delivery Network (CDN)** to cache media files on edge servers physically close to users.
*   **Problem:** Sending emails or processing videos takes too long, slowing down user requests.
    *   *Solution:* Introduce a **Message Queue (Apache Kafka / RabbitMQ)** to offload tasks to background workers asynchronously.

---

### Step 7: Trade-offs & Failures
*Time allocation: 5 minutes*

**The Sports Car Analogy:** You built a car that is fast and cheap, but it lacks safety features or off-road capabilities. Every system has trade-offs.

Wrap up by discussing the limitations of your design:
*   **CAP Theorem Choices:** Did you prioritize consistency over availability? (e.g., choosing CP for billing, or AP for the social media feed).
*   **Single Points of Failure:** What happens if the load balancer dies? (Solution: run active-passive load balancers).
*   **Cost vs. Complexity:** Explain that adding more caches and replicas improves speed but increases server hosting costs and software complexity.

---

### Summary Checklist for Beginners
*   **Do not jump straight into drawing architecture.**
*   **Clarify scope first** (Functional & Non-Functional).
*   **Quantify the scale** using Back-of-the-Envelope estimations.
*   **Map out the data flow** (APIs $\rightarrow$ Databases $\rightarrow$ High-level block diagram).
*   **Solve bottlenecks** with scaling patterns (Load balancers, Caches, CDN, Queues).
*   **Acknowledge trade-offs**; there is no single "correct" design.
