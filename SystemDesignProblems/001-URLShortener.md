# Designing a URL Shortener (Bit.ly)

This guide walks through the step-by-step system design of a URL shortener, structured using a practical interview framework. It covers the core requirements, API design, high-level architecture, and deep-dive optimizations for scalability and high availability.

> **Reference Video:** This guide is based on the video [System Design Interview – Step-by-Step Guide](https://www.youtube.com/watch?v=iUU4O1sWtJA).

---

## 1. Define Requirements

Before writing any code or drawing components, we must define the scope of the system.

### Functional Requirements (Core Features)
*   **Generate Short URL:** The system must take a long URL entered by a user and return a shorter, unique URL.
*   **Redirect User:** When a user visits the short URL, the system must instantly redirect them to the original long URL.
*   **Custom Alias (Optional):** Users should be able to define a custom short code (e.g., `bit.ly/my-portfolio`).
*   **Link Expiration (Optional):** Users should be able to set an expiration time for the short link, after which the link becomes inactive.

### Non-Functional Requirements (Performance & Scale constraints)
*   **Low Latency (< 200ms):** The redirection process must be near-instantaneous so the user experience feels seamless.
*   **High Scalability:** The system must handle **100 million Daily Active Users (DAU)** and store up to **1 billion new URLs** every month.
*   **Uniqueness:** No two long URLs should generate the exact same short code unless specifically requested.
*   **High Availability over Strong Consistency:** It is far more important for the service to be always online (highly available) to redirect users, even if a newly created short link takes a few seconds to sync across all worldwide databases (eventual consistency).

---

## 2. Core Entities & API Design

Next, we establish the database entities and the contract between the client (web app) and our server.

### Core Entities (Data Models)

*   **URL Model:**
    ```json
    {
      "short_code": "string (Primary Key)",
      "long_url": "string (URL destination)",
      "user_id": "string (Foreign Key, optional)",
      "created_at": "timestamp",
      "expires_at": "timestamp (optional)"
    }
    ```
*   **User Model:**
    ```json
    {
      "user_id": "string (Primary Key)",
      "email": "string",
      "password_hash": "string",
      "created_at": "timestamp"
    }
    ```

### API Design (Client-Server Contract)

We will use standard RESTful APIs to communicate between the frontend client and the backend server.

#### 1. Shorten a URL
*   **HTTP Method:** `POST`
*   **Endpoint:** `/api/v1/urls`
*   **Request Body (JSON):**
    ```json
    {
      "long_url": "https://www.google.com/search?q=system+design+interview+guide",
      "custom_alias": "sys-guide",
      "expires_in_days": 30
    }
    ```
*   **Response Body (JSON):**
    ```json
    {
      "short_url": "https://bit.ly/sys-guide",
      "short_code": "sys-guide",
      "created_at": "2026-06-01T12:00:00Z",
      "expires_at": "2026-07-01T12:00:00Z"
    }
    ```

#### 2. Redirect to Original URL
*   **HTTP Method:** `GET`
*   **Endpoint:** `/{short_code}` (e.g., `GET /sys-guide`)
*   **Response Headers:**
    *   `Status Code:` `302 Found` (Redirect)
    *   `Location:` `https://www.google.com/search?q=system+design+interview+guide`

---

## 3. High-Level Design

At a basic level, the system needs to receive requests, map the short code to the long URL in a database, and redirect the user.

```
+----------+             GET /sys-guide            +---------------+
|          | ====================================> |               |
|  Client  |                                       | Web Server    |
| (Browser)| <==================================== | (App Node)    |
+----------+        302 Redirect Location          +-------+-------+
                                                           |
                                                      Query|Database
                                                           v
                                                   +-------+-------+
                                                   | Relational DB |
                                                   | (URL Mapping) |
                                                   +---------------+
```

### The Redirect Decision: 301 vs. 302 Redirects
When redirecting users, the choice of HTTP status code has a huge impact on system performance and metrics tracking:

*   **301 Moved Permanently:** The client's browser caches this redirect. Next time the user enters the short URL, the browser redirects them directly without hitting our servers.
    *   *Pros:* Reduces server load and hosting costs.
    *   *Cons:* We cannot track click analytics (like location, device, or total visit counts) after the first redirect.
*   **302 Found (Temporary Redirect - Preferred):** The browser is told that this redirect is temporary. The browser is forced to hit our servers for every single click.
    *   *Pros:* Allows us to collect accurate, real-time click analytics.
    *   *Cons:* Increases traffic load on our servers.
*   **Verdict:** We choose **302 Found** because analytics tracking is a vital business feature for URL shorteners.

---

## 4. Deep Dives & Optimization

To handle the scale of 100 million DAU and 1 billion URLs per month, we must optimize the bottlenecks of our high-level architecture.

### A. Generating Unique Short Codes
Simply hashing the long URL using MD5 or SHA256 will result in long strings (e.g., 32 characters) that need to be trimmed. Trimming hashes increases the risk of **collisions** (where two different URLs get the same short code).

#### The Solution: Unique Counter + Base62 Encoding
Instead of hashing, we use a centralized **Unique ID Generator** (like Snowflake ID or a distributed counter cluster) to generate a unique 64-bit number (e.g., `120485`).

We then convert this number to **Base62** (using letters `[a-z]`, `[A-Z]`, and digits `[0-9]`).
*   Example: Base10 number `120485` is converted to `W7` in Base62.
*   A 6-character Base62 string yields $62^6 \approx 56.8 \text{ billion}$ unique combinations, which is more than enough for our scale.

#### Obfuscation
Using sequential numbers allows competitors to guess other short links easily (e.g., `bit.ly/1`, `bit.ly/2`). We can pass the generated sequence IDs through a bijective obfuscation library (like **Sqids** - formerly Hashids) to scramble the sequence into random-looking strings (e.g., `bit.ly/X8d2Kl`) while guaranteeing uniqueness.

---

### B. Optimizing Latency
Redirection requires checking the database for every single click. Database disk lookups are slow and will fail under high load.

*   **Database Indexing:** We must create a primary index on the `short_code` field in our database to ensure $O(1)$ search lookup times.
*   **Read-Through Cache (Redis):** Because URL links follow a *Pareto Distribution* (a small percentage of popular links generate 80% of click traffic), we can place an in-memory cache (like Redis) in front of the database.
    *   *Cache Policy:* We store mappings in Redis with an **LRU (Least Recently Used)** eviction policy. Popular links remain in memory, allowing them to be read in under 5ms, avoiding hitting the database.

---

### C. Scalability (Horizontal Scaling)
As traffic grows, a single web server running our app will run out of memory and CPU.

*   **Load Balancers:** We place a Load Balancer (like Nginx or AWS ALB) in front of a cluster of web servers. The load balancer distributes incoming user requests evenly across the servers.
*   **Autoscaling:** We configure our web servers to scale horizontally. If CPU usage across all nodes exceeds 70%, the system automatically spins up new server instances to handle the traffic.

---

### D. High Availability (Fault Tolerance)
If a primary database crashes, we could lose our mapping records and break redirects worldwide.

*   **Read Replicas (Active-Passive):** We run one Master Database (Active) that handles all write operations (creating URLs), and replicates data to multiple Replica Databases (Passive) that only handle reads (redirects). If the Master database fails, one of the passive replicas is instantly promoted to Master (Failover).
*   **S3 Snapshots:** We take periodic, incremental snapshots of our database data and upload them to secure cloud storage (like AWS S3) for disaster recovery.
*   **Eventual Consistency:** We accept that it may take a few seconds for a new URL created on the Master database to sync to replicas worldwide. Read availability remains unaffected.
