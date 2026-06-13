# Caching, Blob Storage & CDN

This guide explains how Caching (Redis), Blob Storage (AWS S3), and Content Delivery Networks (CDNs) speed up systems and handle massive files using simple, everyday analogies.

> **Reference Material:** This guide is based on the article [System Design for Beginners: Everything you need in one article](https://medium.com/@shivambhadani_/system-design-for-beginners-everything-you-need-in-one-article-c74eb702540b) by Shivam Bhadani.

---

## 1. Caching

Caching is one of the most powerful ways to speed up an application. It relies on a simple rule: **avoid doing the same slow work twice.**

### A. Caching Introduction

#### The Study Table Analogy
Imagine you are writing a research paper in a library.
*   **The Database (The Bookshelves):** Down the street, there is a giant public library. Walking there, searching the index, finding a book, and walking back takes **30 minutes** (Slow Disk Read).
*   **The Cache (Your Study Table):** Instead of walking to the library every time you need a fact, you walk there once, borrow the 5 books you need, and stack them on your desk. When you need to look something up, you check your desk first.
    *   **Cache Hit:** The book is on your desk. You read it in **2 seconds**.
    *   **Cache Miss:** The book isn't on your desk. You must spend 30 minutes walking to the library to fetch it and place it on your desk for future use.

**A cache is a fast, temporary storage layer that sits in memory (RAM) and stores copies of frequently requested data.**

---

### B. Benefits of Caching

1.  **Ultra-Low Latency (Fast Speed):** Reading data from a server's RAM (memory) takes microseconds, whereas reading from a database's hard drive takes milliseconds. Caching makes your app feel instant.
2.  **Reduced Database Load:** By answering common questions from the cache, you prevent your SQL database from getting overwhelmed by duplicate queries.
3.  **Cost Savings:** Since the cache handles most of the read traffic, you need fewer database servers, saving database licensing and hosting costs.

---

### C. Types of Caches

*   **Client-Side Cache (Browser):** Your web browser stores logo images, fonts, and stylesheets on your laptop's hard drive. When you reload a website, your browser loads these files locally instead of downloading them again.
*   **Server-Side Cache (Database Cache):** A dedicated server (like Redis) placed between your application code and the primary database to cache query results.
*   **CDN Cache (Edge Cache):** Distributed cache servers positioned at the edge of the internet network, physically close to users.

---

### D. Redis Deep Dive

**Redis** (Remote Dictionary Server) is the industry standard for server-side caching.

#### Why is Redis so fast?
Unlike databases like PostgreSQL or MySQL that save data to physical hard drives (SSDs/HDDs), **Redis stores all its data directly in the RAM**. RAM is incredibly fast, allowing Redis to handle hundreds of thousands of read and write requests per second.

#### Cache Invalidation (The Stale Data Problem)
If you update a user's profile picture in the database, but your cache still holds the old picture, the user will see outdated (stale) data. To prevent this, caches use eviction policies:
1.  **TTL (Time-to-Live):** Every item in the cache is given an expiration timer (e.g., 15 minutes). When the timer hits zero, the item is deleted. The next request will result in a cache miss and fetch the fresh data.
2.  **LRU (Least Recently Used):** When the cache memory gets full, the system automatically deletes the item that hasn't been accessed for the longest time to make room for new data.

---

## 2. Blob Storage

When designing applications, we store user accounts and comments in database tables. But where do we store massive files like videos, images, and audio?

### A. What is a Blob, and why do we need Blob Storage?

*   **Blob** stands for **Binary Large Object**. It refers to any massive chunk of unstructured data—such as a JPG image, an MP4 video, a PDF invoice, or a backup ZIP file.
*   **Why not store Blobs in a database?**
    *   Relational databases (SQL) are built for structured tables with searchable keys.
    *   If you store a 500 MB video file inside a row of your database:
        *   The database memory gets clogged.
        *   Backing up the database takes hours.
        *   Query speeds drop drastically for all other users.
*   **The Solution:** Upload the raw file to a cheap **Blob Storage** service, get a unique URL pointing to the file, and save only that text URL in your database.

```
                  +-----------------------------------+
                  |        Client Browser             |
                  +-----------------------------------+
                       /                         \
         1. Upload File                           3. Request Avatar URL
                     /                             \
                    v                               v
         +--------------------+             +------------------+
         |    Blob Storage    |             |  Relational DB   |
         |     (AWS S3)       |             |  (PostgreSQL)    |
         +--------------------+             +------------------+
         | avatar_file.png    |             | user_id | url    |
         +--------------------+             +------------------+
         | Returns static URL | ----------> | 45      | s3_url |
         +--------------------+             +------------------+
```

---

### B. AWS S3 (Simple Storage Service)

AWS S3 is the most popular cloud Blob Storage service.
*   **Buckets:** Think of buckets as root folders in the cloud (e.g., `my-user-uploads-bucket`). You create a bucket and define who has permission to view its contents.
*   **Objects:** Every file you upload to S3 is called an Object. It contains the raw file content and key-value metadata (e.g., file type, upload date).
*   **11 Nines Durability:** S3 is designed to provide **99.999999999%** durability. It does this by automatically copying your file to at least three different physical data centers. Even if a lightning strike destroys an entire AWS building, your file is safe.

---

## 3. Content Delivery Network (CDN)

A CDN is a global network of servers designed to deliver static website files to users as fast as possible.

### A. CDN Introduction

#### The Ice Cream Shop Analogy
Imagine you live in Sydney, Australia, and you want to order ice cream from a famous shop in New York.
*   **Without a CDN:** If they ship one ice cream cone across the ocean for every order, it will take 20 hours to arrive, cost a fortune in shipping, and melt completely before it reaches you (High Latency).
*   **With a CDN:** The New York shop rents local freezer trucks parked in Sydney, Melbourne, and Brisbane, fills them with ice cream, and leaves them there. When you order, a local truck delivers the ice cream to your door in 10 minutes.

**In software, a CDN places cache servers (Edge Servers) close to major cities worldwide, caching static assets (videos, images, JavaScript files) so users can download them in milliseconds.**

---

### B. How does CDN work?

```
[ User in UK ] ----> 1. Request static image ----> [ UK Edge Server ] (Cache Hit!)
                                                           |
                                                2. Returns Image instantly
                                                           v
                                                     [ User in UK ]
```

1.  A user in London visits a website hosted on an **Origin Server** in New York.
2.  The request for the homepage logo is intercepted and routed to the nearest **CDN Edge Server** in London.
3.  **Cache Hit:** If the London Edge Server already has a copy of the logo, it sends it to the user instantly.
4.  **Cache Miss:** If it does not have the logo, the London server fetches it from the New York Origin Server once, saves a copy locally for future UK users, and delivers it to the client.

---

### C. Key Concepts in CDNs

*   **Origin Server:** The main backend server where the source of truth for your application files resides.
*   **Edge Servers (Points of Presence / PoPs):** Globally distributed cache servers that sit close to users to deliver content quickly.
*   **Push CDN:** The developer manually uploads files to the CDN before users ask for them. Best for massive, rare updates (e.g., game patch downloads).
*   **Pull CDN:** The CDN automatically "pulls" files from the Origin Server when a user requests them for the first time. Best for websites with standard traffic and static assets.
