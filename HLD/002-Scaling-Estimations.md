# Scaling & Estimations

This guide explains scaling types, auto-scaling mechanisms, and back-of-the-envelope estimations using simple, everyday language and analogies.

> **Reference Material:** This guide is based on the article [System Design for Beginners: Everything you need in one article](https://medium.com/@shivambhadani_/system-design-for-beginners-everything-you-need-in-one-article-c74eb702540b) by Shivam Bhadani.

---

## 1. Scaling and its Types

When a system scales, it means it is adapting to handle a growing volume of work, such as more users logging in, more requests being processed, or more data being stored. 

### The Bakery Analogy
Imagine you run a small local bakery. You start with **one oven** and **one baker**, producing **50 loaves of bread** per day. Suddenly, your bread goes viral on social media, and **500 customers** show up every morning wanting bread. 

You cannot serve 500 customers with your current setup. You need to **scale** your bakery. There are two main ways to do this:

---

### A. Vertical Scaling (Scaling Up)
*   **The Analogy:** Instead of buying a new shop, you throw away your small oven and buy a **giant, high-tech industrial oven** that can bake 500 loaves of bread at the same time. You also send your baker to intensive training to make them faster.
*   **In Software terms:** You keep the same single server, but you upgrade it. You replace the hardware with a faster processor (CPU), add more memory (RAM), and get a larger hard drive (SSD).
*   **Pros:**
    *   **Very Simple:** You don't have to change how you work. The recipes (code) and baking steps remain exactly the same.
    *   **No Coordination Needed:** Since everything happens inside one kitchen (one computer), there is no delay in talking to other kitchens.
*   **Cons:**
    *   **The Hardware Limit:** Ovens can only get so big before they violate the laws of physics or become impossible to manufacture. Similarly, there is a hard limit to the maximum RAM or CPU a single computer can support.
    *   **Single Point of Failure (SPOF):** If your giant oven breaks down or catches fire, your entire bakery goes out of business until it's fixed.
    *   **Expensive:** Buying top-tier, ultra-premium hardware is exponentially more expensive than buying multiple mid-range computers.

---

### B. Horizontal Scaling (Scaling Out)
*   **The Analogy:** Instead of one giant oven, you buy **nine more normal-sized ovens** and place them side-by-side. You also hire nine more bakers to operate them.
*   **In Software terms:** Instead of upgrading your existing server, you add more servers (nodes) of the same specification next to it, sharing the workload.
*   **Pros:**
    *   **Virtually Infinite Growth:** If you get even more customers next month, you don't need to replace anything; you just buy oven number 11, 12, and 13.
    *   **High Availability (No Single Point of Failure):** If Oven #3 breaks down, the other 9 ovens can still keep baking bread. Your bakery stays open.
*   **Cons:**
    *   **High Complexity:** You now need a "manager" standing at the door to distribute the customers and dough to the 10 different ovens equally. In software, this manager is called a **Load Balancer**.
    *   **Consistency Issues:** If one baker changes the recipe, you have to find a way to make sure all 10 bakers update their recipes at the exact same time so the bread tastes identical everywhere.

---

## 2. Auto Scaling

Auto scaling is like a smart autopilot for your servers, adjusting their count up and down dynamically depending on how busy your app is.

### The Restaurant Waiter Analogy
Imagine you run a restaurant that is extremely busy during lunch (12 PM to 2 PM) but practically empty in the afternoon (3 PM to 5 PM).

*   **Option A (Over-provisioning):** You hire **50 waiters** for the whole day. 
    *   *Result:* At lunch, service is great. But at 4 PM, you are paying 48 waiters to sit around doing nothing. You waste a lot of money.
*   **Option B (Under-provisioning):** You hire only **3 waiters** to save money.
    *   *Result:* At 4 PM, they are relaxed. But at lunch, they get completely overwhelmed, service becomes extremely slow, and angry customers walk out.
*   **The Auto Scaling Solution:** You partner with a magic staffing agency. The moment a line starts forming outside your restaurant, the agency instantly sends temporary waiters. As soon as the rush dies down and the restaurant empties, those temporary waiters go home. You only pay for what you actually use.

```
[ Normal Traffic: 2 Servers ] ---- CPU Load hits 80% ----> [ Scale Out: +3 Servers (Total 5) ]
[ Night Traffic: CPU drops 20% ] ------------------------> [ Scale In: Terminate 3 Servers ]
```

### How it Works in the Cloud
Cloud platforms (like AWS or Google Cloud) monitor your servers using simple rules:
1.  **Monitor CPU:** The system watches how hard your servers are working.
2.  **Add Servers (Scale Out):** If CPU usage across your servers stays above **75%** for more than 3 minutes, the cloud automatically launches 2 new servers and instructs the Load Balancer to start sending them traffic.
3.  **Remove Servers (Scale In):** If CPU usage falls below **25%** (e.g., at 3 AM), the cloud automatically shuts down the extra servers to save you money.

---

## 3. Back-of-the-envelope Estimation

Back-of-the-envelope estimation is the process of doing quick, rough mental math (on a "napkin" or envelope) to figure out if your design idea is realistic before you actually build it.

### The Wedding Planning Analogy
If you are planning a wedding, you don't order food down to the exact gram, but you also don't guess blindly. You do quick math:
*   *"I expect about 100 guests."*
*   *"Each guest will probably drink 3 glasses of water."*
*   *"So, I need about 300 glasses of water."*

This quick estimate tells you that you need to buy a few water dispensors, not book a water tanker truck (over-estimating) and not buy a single 2-liter bottle (under-estimating).

---

### Key Numbers to Remember for System Design
When doing system design math, keep these numbers in your back pocket:
*   **Seconds in a day:** $60 \times 60 \times 24 = 86,400$ seconds. (We round it to **100,000** to make mental division easy).
*   **Data Sizes:**
    *   1 character of text = **1 Byte**
    *   1 KB (Kilobyte) = 1,000 Bytes (approx. 1 paragraph of text)
    *   1 MB (Megabyte) = 1,000 KB (approx. 1 book or a compressed image)
    *   1 GB (Gigabyte) = 1,000 MB (approx. a high-definition movie)
    *   1 TB (Terabyte) = 1,000 GB
*   **Mental Multipliers:**
    *   $1 \text{ Million} \div 100,000 \text{ seconds} = 10 \text{ per second}$
    *   $10 \text{ Million} \div 100,000 \text{ seconds} = 100 \text{ per second}$
    *   $100 \text{ Million} \div 100,000 \text{ seconds} = 1,000 \text{ per second}$

---

### Step-by-Step Example: Designing a Photo-Sharing App

Let's do the napkin math for a simple photo-sharing app like Instagram:

#### Assumptions:
1.  **Daily Active Users (DAU):** 10 Million people open our app every day.
2.  **Uploads:** 10% of users upload 1 photo per day.
3.  **Views:** Every user views about 20 photos per day.
4.  **Photo Size:** The average uploaded photo is compressed to **2 Megabytes (MB)**.

---

#### Step 1: Calculate Traffic (Queries Per Second - QPS)

We need to know how many write (upload) requests and read (view) requests our servers will receive per second.

*   **Write QPS (Uploads):**
    $$\text{Daily Uploads} = 10,000,000 \text{ users} \times 10\% = 1,000,000 \text{ uploads/day}$$
    $$\text{Average Upload QPS} = \frac{1,000,000 \text{ uploads}}{100,000 \text{ seconds/day}} = 10 \text{ uploads/second}$$

*   **Read QPS (Views):**
    $$\text{Daily Views} = 10,000,000 \text{ users} \times 20 \text{ views/day} = 200,000,000 \text{ views/day}$$
    $$\text{Average View QPS} = \frac{200,000,000 \text{ views}}{100,000 \text{ seconds/day}} = 2,000 \text{ views/second}$$
    *   *Peak Traffic:* During busy hours, traffic can spike. We multiply average QPS by **2x** to estimate peak load:
        $$\text{Peak View QPS} = 2,000 \times 2 = 4,000 \text{ views/second}$$

---

#### Step 2: Calculate Storage Capacity

We need to know how much storage space we will need to buy to keep these photos.

*   **Daily Storage:**
    $$\text{Daily Storage} = 1,000,000 \text{ photos/day} \times 2 \text{ MB/photo} = 2,000,000 \text{ MB/day}$$
    $$\text{Daily Storage} = 2,000 \text{ GB/day} = 2 \text{ TB/day}$$

*   **Yearly Storage:**
    $$\text{Yearly Storage} = 2 \text{ TB/day} \times 365 \text{ days} = 730 \text{ TB/year}$$

*   **5-Year Storage:**
    $$\text{5-Year Storage} = 730 \text{ TB/year} \times 5 = 3,650 \text{ TB} \approx 3.65 \text{ PB (Petabytes)}$$

### What does this estimation tell us?
This simple math tells us two major things:
1.  **Bandwidth:** Our system needs to be able to stream $2,000 \text{ reads/sec} \times 2 \text{ MB} = 4 \text{ GB}$ of data out to users every second.
2.  **Storage Solution:** We cannot store 3.65 Petabytes of photos on a standard database server. We must upload photos to a specialized, cheap distributed file system (like AWS S3) and only keep the text links (URLs) in our database.
