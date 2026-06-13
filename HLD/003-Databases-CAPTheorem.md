# Databases & CAP Theorem

This guide explains the CAP Theorem, database scaling techniques, and the differences between SQL and NoSQL databases using simple, everyday analogies.

> **Reference Material:** This guide is based on the article [System Design for Beginners: Everything you need in one article](https://medium.com/@shivambhadani_/system-design-for-beginners-everything-you-need-in-one-article-c74eb702540b) by Shivam Bhadani.

---

## 1. CAP Theorem

The CAP Theorem is a rule that explains the trade-offs we must make when building a system that spreads its data across multiple computers (a **distributed system**). 

### The Notebook Analogy
Imagine you and your friend are working as phone operators. You both keep a notebook next to your desk to write down phone numbers.
*   **Consistency (C):** If a customer calls and updates a phone number, both you and your friend must have the *exact same* number in your notebooks immediately. No one should read old, outdated information.
*   **Availability (A):** If a customer calls, they must get an answer. You cannot say "Sorry, I can't look at my notebook right now; please call back later." You must always respond.
*   **Partition Tolerance (P):** You and your friend are in different rooms, and the phone line connecting your rooms is cut (this is a network partition). You can no longer talk to each other to sync updates.

```
       [ You (Room 1) ]             [ Friend (Room 2) ]
        [ Notebook X ]               [ Notebook Y ]
              \                             /
               \--- (Phone Line Cut!) ---/
```

#### The Trade-off during a Failure
If a customer calls your friend and changes a phone number, your friend updates their notebook. Now, another customer calls *you* to ask for that same phone number. Since the phone line is cut, you don't know about the update. What do you do?

*   **Choice A: Prioritize Consistency (CP System)**
    You refuse to answer the caller, or you return an error: *"Sorry, my line is down, I cannot give you this number right now."* 
    *   *Result:* You keep the system **consistent** (no one gets the wrong number), but you lose **availability** (you couldn't answer the customer).
*   **Choice B: Prioritize Availability (AP System)**
    You answer the customer and read the old number from your notebook.
    *   *Result:* You keep the system **available** (you gave an answer), but you lose **consistency** (the customer got old, incorrect information).

**In the real world, network cuts (P) are inevitable. Therefore, you must choose to build either a CP system or an AP system.**

---

## 2. Scaling of Databases

As your app grows, your database will receive more write requests (saving data) and read requests (retrieving data). To keep it from slowing down or crashing, you need database scaling.

### A. Indexing
*   **The Analogy:** Think of a cookbook index. Instead of turning page-by-page to find a recipe for "Chocolate Cake" (which is a "full table scan"), you flip to the back of the book, look under 'C' for "Chocolate Cake", find "Page 143", and go straight there.
*   **How it works:** A database index is a hidden table that maps keys (like user IDs) directly to their location on the disk.
*   **The Trade-off:**
    *   **Fast Reads:** Finding data becomes incredibly quick.
    *   **Slow Writes:** Every time you add a new recipe (insert a row), you must also write it down in the index at the back of the book. This adds extra work.

---

### B. Partitioning
*   **The Analogy:** Splitting a giant drawer of paper files into smaller folders.
*   **Vertical Partitioning:** Splitting columns. E.g., keeping a user's lightweight `username` and `password` in Table A, and moving their heavy `profile picture` and `bio text` to Table B on the same disk.
*   **Horizontal Partitioning:** Splitting rows. E.g., putting customers from the year 2024 into Partition A and customers from 2025 into Partition B. The database system handles this automatically behind the scenes.

---

### C. Master-Slave Architecture
Designed to scale **read-heavy** apps (apps where users read data much more than they write it, like Twitter or Instagram).

*   **The Analogy:** A school teacher (Master) writing notes on a blackboard, and 30 students (Slaves) copying those notes onto their notebooks.
    *   If you want to **Write** (update notes), you must do it on the blackboard (Master).
    *   If you want to **Read** (study notes), you read it from your own notebook (Slaves) instead of crowding around the blackboard.
*   **Pros:**
    *   Saves the Master from getting overwhelmed by read requests.
    *   If the Master server dies, one of the Slaves can be promoted to become the new Master.
*   **Cons:**
    *   **Replication Lag:** It takes a split second for the students to copy the blackboard. If a student reads their notebook too quickly, they might read old notes before they are updated.

---

### D. Multi-master Setup
*   **The Analogy:** A group project where two students can write on the blackboard at the same time.
*   **How it works:** Multiple database servers accept write requests, and they continuously sync their changes with each other.
*   **Pros:** If one writer server is down, you can write to the other. Excellent write performance.
*   **Cons:** **Conflict resolution.** If Student A writes "X" and Student B writes "Y" in the same spot at the exact same second, the database must use complicated logic to decide which one is correct.

---

### E. Database Sharding (Horizontal Scaling)
*   **The Analogy:** Splitting a massive phone directory into two booklets. Booklet A-M is kept at House 1, and Booklet N-Z is kept at House 2.
*   **How it works:** Distributing rows of a single database table across completely separate physical database servers (shards) based on a **Sharding Key** (e.g., routing users by their last names).

```
                            [ Client App ]
                                  |
                        [ Routing Controller ]
                       /                      \
        Usernames starting A-M          Usernames starting N-Z
                     /                          \
                    v                            v
          +-------------------+        +-------------------+
          |  Shard 1 Server   |        |  Shard 2 Server   |
          |  (Db rows A-M)    |        |  (Db rows N-Z)    |
          +-------------------+        +-------------------+
```

---

### F. Disadvantages of Sharding
While sharding allows databases to scale infinitely, it comes with massive penalties:
1.  **Cross-Shard Joins:** If you want to join data from Shard 1 and Shard 2 (e.g., matching a user on Shard 1 with a purchase record on Shard 2), it is extremely slow and complicated.
2.  **Re-sharding:** If Shard 1 gets full, you have to change your routing formula and move millions of rows of data across servers, which often causes app downtime.
3.  **Hot Shards (The Celebrity Problem):** If a famous user (like Elon Musk) is routed to Shard 1, Shard 1 will get millions of requests and crash, while Shard 2 sits empty.

---

## 3. SQL vs. NoSQL Databases

Choosing a database is like choosing a storage system for your room.

```
       SQL Database (Relational)                   NoSQL Database (Non-Relational)
      +-------------------------+                 +-------------------------------+
      |  Predefined Grid/Box    |                 |      Flexible Storage Bin     |
      |  - Every item has a slot|                 |      - Throw items inside     |
      |  - Strict, organized    |                 |      - Dynamic structures     |
      +-------------------------+                 +-------------------------------+
```

### A. SQL Database (Relational)
*   **The Analogy:** An Excel spreadsheet with fixed columns. Every row must fill in the exact columns specified.
*   **How it works:** Stores data in tables with rows and columns. Uses relationships (e.g., linking a `User` table to an `Order` table using IDs).
*   **Key Feature:** Follows **ACID** properties, which guarantee that transactions are 100% secure and accurate (e.g., money never disappears during a transfer).
*   *Examples:* PostgreSQL, MySQL, MS SQL Server.

### B. NoSQL Database (Non-Relational)
*   **The Analogy:** A folder full of sticky notes (JSON documents). One note might have a name and phone number; another note might have a name, email, and home address. They don't have to match.
*   **How it works:** Stores data in flexible formats like Key-Value pairs, Documents, or Graphs.
*   *Examples:* MongoDB (Document), Redis (Key-Value), Cassandra (Column-Family).

---

### C. Scaling in SQL vs. NoSQL
*   **SQL Scaling:** SQL databases are designed to run on a single machine. They scale **Vertically** (buying a bigger computer). While you *can* shard SQL databases, it is highly complex and painful.
*   **NoSQL Scaling:** NoSQL databases were built from day one to scale **Horizontally**. They natively split, replicate, and shard data across hundreds of cheap servers automatically.

---

### D. When to use which Database?

#### Choose SQL if:
*   You are building financial systems, checkout pages, or accounting software (where ACID compliance and accuracy are non-negotiable).
*   Your data structure is highly relational (e.g., a school database where Students belong to Classes, taught by Teachers, who have Departments).
*   The data structure is stable and won't change frequently.

#### Choose NoSQL if:
*   You need to scale to handle massive amounts of writes and reads (e.g., real-time chat apps, gaming leaderboards, IoT sensor data).
*   Your data doesn't have complex relationships (e.g., simple user profiles, shopping carts).
*   Your schema is dynamic or changes constantly (e.g., storing product details where clothes have "sizes", but books have "page counts").
