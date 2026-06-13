# Consistency Deep Dive & Consistent Hashing

This guide covers database consistency levels (Strong vs. Eventual) and how Consistent Hashing prevents server overload, explained using simple real-world analogies and layman terms.

> **Reference Material:** This guide is based on the article [System Design for Beginners: Everything you need in one article](https://medium.com/@shivambhadani_/system-design-for-beginners-everything-you-need-in-one-article-c74eb702540b) by Shivam Bhadani.

---

## 1. Consistency Deep Dive

In a distributed system, we duplicate our data across multiple servers (replicas) to ensure that if one server crashes, we don't lose anything. But this introduces a challenge: **how do we keep all these replicas in sync?** 

Consistency refers to how quickly and reliably updates to our data are visible across all servers.

```
                   +-------------------+
                   |   User Writes A=5 |
                   +---------+---------+
                             |
                             v
                    +--------+--------+
                    |  Server 1 (A=5) |
                    +--------+--------+
                             |
         How fast does this  |  sync to Server 2?
                             v
                    +--------+--------+
                    |  Server 2 (A=?) |
                    +-----------------+
```

---

### A. Strong Consistency

**Strong Consistency** guarantees that once a write operation is complete, any subsequent read request from any user on any server will return that new value. The system acts as if there is only a single copy of the data.

#### The Shared Bank Account Analogy (ATM)
Imagine you and your spouse share a bank account that currently has **$1,000**.
1.  You walk up to an ATM in New York and withdraw **$900**.
2.  The NYC ATM updates the database to show a balance of **$100**.
3.  Under **Strong Consistency**, if your spouse checks the account balance at an ATM in London one millisecond later, the system *forces* the London ATM to wait or sync before replying. Your spouse is guaranteed to see **$100**, preventing them from withdrawing money that isn't there.
4.  If the network connection between NYC and London is broken, the system will refuse your withdrawal request entirely to avoid displaying inconsistent balances.

#### When to Choose Strong Consistency
You should prioritize strong consistency when displaying incorrect or stale data leads to critical failures:
*   **Financial Applications:** Checking account balances and money transfers.
*   **Inventory Management:** Booking airline seats or purchasing last-in-stock items (you don't want to double-sell).
*   **Security Systems:** Password updates or permission changes (if a user is blocked, they must be blocked instantly everywhere).

#### How to Achieve Strong Consistency
1.  **Two-Phase Commit (2PC):** A coordinator server asks all replica databases: *"Are you ready to commit this change?"* (Phase 1). If all replicas reply "Yes," the coordinator tells them all: *"Write it now!"* (Phase 2). If even one node is unreachable, the entire write is canceled.
2.  **Consensus Protocols (Raft / Paxos):** The servers run an election-based agreement process. A write is only confirmed to the user once a strict majority (quorum) of the servers agree and record the update.
3.  **Read-Write Quorums ($W + R > N$):** If you have $N$ total replicas, you require a write operation to succeed on $W$ nodes and a read operation to fetch data from $R$ nodes. If $W + R$ is greater than $N$, your read query is mathematically guaranteed to hit at least one node containing the latest write.

---

### B. Eventual Consistency

**Eventual Consistency** is a weaker consistency model. It allows replicas to be out of sync temporarily. If no new updates are made, all replicas will *eventually* receive the update and become consistent.

#### The Social Media Like Button Analogy
Imagine you post a picture on Instagram.
1.  You click "Like" on your post. Your phone talks to a server in California, and your screen shows **1 Like**.
2.  Under **Eventual Consistency**, the California server doesn't wait to update all other servers around the world. It immediately tells your app *"Done!"* so your app feels fast.
3.  In the meantime, your friend in Tokyo views your profile. Their phone connects to a Tokyo server which hasn't received the update yet. Your friend sees **0 Likes**.
4.  A few seconds later, the California server sends the update to Tokyo in the background. Now, your friend's feed updates to show **1 Like**.
5.  *It didn't matter that the likes were out of sync for a few seconds—no harm was done, and the user experience remained lightning-fast.*

#### When to Choose Eventual Consistency
Choose eventual consistency when speed (low latency) and keeping the system online (high availability) are more important than displaying 100% real-time accurate data:
*   **Social Networks:** Likes, comments, posts, subscriber counts.
*   **Streaming Platforms:** View counters on YouTube videos (it's okay if a video shows 10,000 views instead of 10,012 for a brief moment).
*   **DNS Propagation:** When you buy a website domain, it takes up to 24–48 hours for servers globally to find it.
*   **Product Catalogs:** Reviews and star ratings on shopping sites like Amazon.

#### How to Achieve Eventual Consistency
1.  **Asynchronous Replication:** The primary server records a write operation, immediately tells the client it succeeded, and then broadcasts the update to secondary servers in the background.
2.  **Gossip Protocol:** Servers periodically exchange their data with random neighbor nodes, like office gossip spreading around a room. Over time, the information propagates to every single server in the cluster.
3.  **Read Repair:** When a user reads data, the system queries multiple replicas. If it detects that Server B has an older version than Server A, it returns the latest version to the user and immediately sends a background update to Server B to fix (repair) its stale data.

---

## 2. Consistent Hashing

In a large system, we use multiple caching servers to store temporary data (like user profiles) to avoid overloading our primary database. **Consistent Hashing** is a technique used to distribute this data evenly across servers in a way that minimizes disruptions when servers are added or removed.

### The Problem: Traditional Hashing
Say you have 3 cache servers (Node 0, Node 1, Node 2) and you want to store user sessions. A simple way to distribute them is using the modulo operator:

$$\text{Server Index} = \text{Hash}(\text{User Key}) \pmod N$$

*(where $N$ is the number of servers)*

Let's say we have keys that hash to numbers:
*   `User_A` (hash = 10): $10 \pmod 3 = 1$ (goes to Server 1)
*   `User_B` (hash = 11): $11 \pmod 3 = 2$ (goes to Server 2)
*   `User_C` (hash = 12): $12 \pmod 3 = 0$ (goes to Server 0)

#### Why this crashes (The Node Failure Disaster)
If **Server 1 crashes**, our server count $N$ drops from 3 to 2. Let's recalculate the modulo for our keys:
*   `User_A` (hash = 10): $10 \pmod 2 = 0$ (moved from Server 1 to Server 0)
*   `User_B` (hash = 11): $11 \pmod 2 = 1$ (moved from Server 2 to Server 1)
*   `User_C` (hash = 12): $12 \pmod 2 = 0$ (stays on Server 0)

Notice that almost **all keys changed their assigned servers**! 
This causes a **Cache Storm**: suddenly, millions of requests result in cache misses because they are routed to the wrong servers. The requests flood your primary database all at once, crashing your entire system.

---

### The Solution: Consistent Hashing
Instead of using modulo arithmetic, Consistent Hashing maps both our **servers** and our **data keys** onto a circular ring (called a **Hash Ring**).

#### The Circular Neighborhood Analogy
Imagine a circular road around a neighborhood:
1.  **Placing the Restaurants (Servers):** We open 3 pizza restaurants: **Dominos**, **Pizza Hut**, and **Papa Johns** at different spots along this circular road.
2.  **Placing the Customers (Keys):** When customers order pizza, we locate their house address on the circular road.
3.  **Delivery Rule:** To find out which restaurant delivers to a house, the delivery driver starts at the house and drives **clockwise** along the circular road. The **first restaurant** they drive past gets the order.

```
                     [ Dominos (Node A) ]
                         /          \
                        /            \
       [ User_A ]      /              \     [ User_B ]
                      |    HASH RING   |
                       \              /
                        \            /
            [ Papa Johns ] -------- [ Pizza Hut (Node B) ]
```

*   **If Dominos closes down (Server Failure):** 
    Only the customers who were ordering from Dominos are affected. Their delivery drivers now drive past Dominos and continue clockwise to the next restaurant, **Pizza Hut**. 
    *The customers of Papa Johns and Pizza Hut are completely unaffected.*
*   **If we open a new restaurant (Server Scaling):** 
    We place it on the circle. It will only take over delivery for a small segment of the circle directly behind it. The rest of the neighborhood keeps ordering from their existing restaurants.

---

### Virtual Nodes (Preventing Hot Spots)
In a simple hash ring, servers might be mapped unevenly. For example, if **Dominos** and **Pizza Hut** end up right next to each other on the ring, Papa Johns might have to cover 80% of the neighborhood alone. This server will crash under the heavy load.

To solve this, Consistent Hashing uses **Virtual Nodes (Vnodes)**:
*   Instead of placing each physical server on the ring once, we place **multiple virtual copies** of each server (e.g., `Dominos-1`, `Dominos-2`, `PizzaHut-1`, `PizzaHut-2`) at different random positions on the ring.
*   This cuts the circle into many smaller segments, distributing the data keys evenly and ensuring no single server becomes a "hot spot."

```
                 Dominos-1   --   PizzaHut-2
                /                           \
         PapaJohns-2                      Dominos-2
              |            HASH RING          |
         PizzaHut-1                      PapaJohns-1
                \                           /
                 Dominos-3   --   PizzaHut-3
```

---

### Summary Checklist for Beginners
*   **Strong Consistency** guarantees immediate correctness but can make the system slow or unavailable during network outages (AP/CP trade-off).
*   **Eventual Consistency** focuses on speed and availability, accepting temporary out-of-sync states with the promise of syncing eventually.
*   **Traditional Modulo Hashing** causes massive cache disruptions when the number of servers changes.
*   **Consistent Hashing** uses a circular ring to map keys and nodes, ensuring that adding or removing a server only affects a tiny fraction ($\approx 1/N$) of the total keys.
