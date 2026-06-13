# Data Redundancy & Data Recovery

This guide covers Data Redundancy and Data Recovery, explaining why we duplicate database data, the different methods of taking backups, and how continuous redundancy keeps modern apps online 24/7.

> **Reference Material:** This guide is based on the article [System Design for Beginners: Everything you need in one article](https://medium.com/@shivambhadani_/system-design-for-beginners-everything-you-need-in-one-article-c74eb702540b) by Shivam Bhadani.

---

## 1. Why Do We Make Databases Redundant?

In system design, **Redundancy** means keeping duplicate copies of the same data on multiple physical servers or across different geographic locations.

Imagine running a business where all your files are stored on a single USB drive. If you lose that drive, step on it, or it gets corrupted, your business instantly goes bankrupt. To prevent this, you copy those files to your laptop, an external hard drive, and cloud storage. That is redundancy.

### Core Reasons for Database Redundancy

1.  **High Availability (No Downtime):**
    If you only have one database server and it crashes or loses internet connection, your entire application goes offline. With redundancy, if your primary database goes down, a duplicate copy (replica) instantly takes over. Your users never even notice a glitch.
2.  **Disaster Recovery (No Data Loss):**
    If a fire, earthquake, or power grid failure destroys the data center where your database is hosted, having a duplicate database running in a different city or country ensures your data remains safe.
3.  **Read Scalability (Handling More Traffic):**
    Instead of forcing millions of users to query a single database, you can route "read" traffic (like browsing products or profiles) to different replicas.
    *   *Example:* Users in London query the European database replica, while users in Tokyo query the Asian replica. This speeds up the website for everyone.

---

## 2. Different Ways of Doing Data Backup

A **Backup** is a snapshot of your database taken at a specific point in time and stored in a secure, secondary location (like AWS S3). If your live database gets corrupted, hacked, or accidentally deleted, you can use the backup to restore the data to its previous state.

There are three main types of database backups, each with its own pros and cons:

```
Timeline: Day 1 (Full) ----> Day 2 (Changes) ----> Day 3 (Changes) ----> Day 4 (Changes)

Backup Strategies:
1. Full:         [=== All Data ===]  [=== All Data ===]  [=== All Data ===]  [=== All Data ===]
2. Incremental:  [=== All Data ===]  [Day 2 Changes]     [Day 3 Changes]     [Day 4 Changes]
3. Differential: [=== All Data ===]  [Day 2 Changes]     [Day 2+3 Changes]   [Day 2+3+4 Changes]
```

### A. Full Backup
*   **Layman Explanation:** You copy **every single piece of data** in the database.
*   **The Diary Analogy:** You have a diary. Every night, you photocopy the entire diary from page 1 to the end.
*   **Pros:** Very easy to restore. If your database crashes, you only need this one backup file to recover everything.
*   **Cons:** Takes a long time to run, requires massive storage space, and consumes high network bandwidth.

### B. Incremental Backup
*   **Layman Explanation:** You only back up the data that has changed **since the last backup of any type** (whether it was a full or incremental backup).
*   **The Diary Analogy:**
    *   Sunday: You do a full photocopy of your diary.
    *   Monday: You only photocopy the new pages written on Monday.
    *   Tuesday: You only photocopy the new pages written on Tuesday.
*   **Pros:** Extremely fast to create and uses very little storage space.
*   **Cons:** Hardest and slowest to restore. If your database crashes on Wednesday, you must first restore Sunday's full backup, then apply Monday's changes, then Tuesday's changes in exact chronological order. If one incremental backup file in the chain is corrupted, you cannot fully recover.

### C. Differential Backup
*   **Layman Explanation:** You back up all data that has changed **since the last full backup**.
*   **The Diary Analogy:**
    *   Sunday: You do a full photocopy of your diary.
    *   Monday: You photocopy everything written on Monday.
    *   Tuesday: You photocopy everything written on Monday *and* Tuesday.
    *   Wednesday: You photocopy everything written on Monday, Tuesday, *and* Wednesday.
*   **Pros:** Easier and faster to restore than incremental backups. If you crash on Wednesday, you only need two files: Sunday's full backup and Tuesday's differential backup.
*   **Cons:** Each daily backup file grows larger and larger until you perform your next full backup. It uses more storage space and takes longer to run than incremental backups.

### Backup Strategy Comparison

| Feature | Full Backup | Incremental Backup | Differential Backup |
| :--- | :--- | :--- | :--- |
| **Backup Speed** | Slowest | Fastest | Moderate |
| **Backup Size** | Largest | Smallest | Grows daily |
| **Restoration Speed** | Fastest | Slowest | Moderate |
| **Required for Restore** | The last full backup file | Last full backup + *all* incremental files in sequence | Last full backup + the *latest* differential file |

---

## 3. Continuous Redundancy

While daily backups are great, they have a major limitation: **data loss window**. If you take a backup at midnight and your database crashes at 11:00 PM, you lose 23 hours of user data. 

To prevent this, production systems use **Continuous Redundancy**, where data is replicated in real-time as soon as a write occurs.

```
Synchronous Replication (Immediate Safety, Slower Writes):
[Client] ---> Write ---> [Active Database] ---> Write ---> [Passive Database]
   ^                                                             |
   |                                                             v
Confirm <------------------- Confirm <------------------------ Confirm

Asynchronous Replication (Fast Writes, Risk of Minor Data Loss):
[Client] ---> Write ---> [Active Database] ---> Confirm ---> [Client]
                             | (in background)
                             v
                        [Passive Database]
```

### A. Active-Passive (Master-Slave) Replication
In this model, one database is designated as the **Active (Master)** node, and the others are **Passive (Slaves/Replicas)**.
*   **Writes:** All insert, update, and delete queries must go to the Active database.
*   **Replication:** The Active database immediately copies the write to the Passive databases.
*   **Reads:** Users can read data from either the Active or Passive databases.
*   **Failover:** If the Active database crashes, one of the Passive databases is promoted to be the new Active database.

### B. Active-Active (Multi-Master) Replication
In this model, **multiple database nodes** act as Active primaries at the same time.
*   **Writes:** A write request can be sent to Server A, Server B, or Server C.
*   **Replication:** The servers continually sync writes among themselves.
*   **Pros:** Excellent for apps with high write traffic. If Server A crashes, writes are seamlessly routed to Server B.
*   **Cons:** Very complex to manage. If a user changes their username to "Alice" on Server A, and another user changes the same record to "Bob" on Server B at the exact same millisecond, the system must resolve this conflict (which is historically difficult).

---

### Summary Checklist for Beginners
*   **Data Redundancy** protects against hardware crashes, data center disasters, and improves read speeds by routing users to their nearest server.
*   **Full Backups** capture everything but are slow; **Incremental** backups capture changes since any backup; **Differential** backups capture changes since the last full backup.
*   **Continuous Redundancy** replicates data live in real-time.
*   **Active-Passive** routes all writes to a single master server; **Active-Active** allows multiple servers to accept writes simultaneously.
