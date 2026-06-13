# System Design for Beginners: Fundamental Concepts

System design is the process of planning and building the architecture of software systems. Below, we break down three of the most fundamental concepts using simple, real-world analogies in layman's language.

> **Reference Material:** This guide is based on the article [System Design for Beginners: Everything you need in one article](https://medium.com/@shivambhadani_/system-design-for-beginners-everything-you-need-in-one-article-c74eb702540b) by Shivam Bhadani.

---

## 1. Why Study System Design?

### The Lemonade Stand vs. Starbucks Analogy
To understand why system design is important, imagine you want to sell lemonade:

1. **The Small Lemonade Stand (Small Scale):**
   You set up a single table in front of your house with a jar, some lemons, sugar, and water. You handle everything yourself. If a customer comes, you squeeze the lemon, mix the sugar, take the cash, and hand them the cup. This is like building a basic web app on your laptop. It works perfectly fine for 5 to 10 customers a day.
   
2. **The Global Starbucks Chain (Large Scale):**
   Now, imagine you want to open 500 outlets across 10 countries, serving 10,000 customers every minute. 
   * You can't just buy a bigger jar or work faster. You will quickly run out of space, lemons, or energy.
   * You need to figure out:
     * How to get lemons delivered to all 500 shops daily without running out (Supply Chain / Data Storage).
     * How to train multiple cashiers and baristas so they don't crash into each other behind the counter (Concurrency / Load Balancing).
     * What happens if one cashier gets sick? How do you keep the store open? (Fault Tolerance / Redundancy).

**System design is the art of transitioning your software from a single "lemonade stand" to a global "Starbucks-like" operation.**

### Core Objectives (What We Solve)

*   **Scalability (Handling Growth):** Can your application handle 1 million users tomorrow if it goes viral? 
    *   *Example:* If a popular musician announces a concert, the ticketing website must handle a sudden flood of 500,000 users clicking "Buy" at the exact same second without crashing.
*   **Reliability & Fault Tolerance (Preventing Crashes):** What happens if a piece of hardware catches fire, or a network cable is cut?
    *   *Example:* If a checkout counter at a grocery store breaks down, the store doesn't close. The manager simply opens a second counter. In system design, we run duplicate servers so that if one fails, others take over instantly.
*   **Availability (Always Online):** Users expect services like Netflix or Google to be online 24/7, 365 days a year. System design ensures there is zero "downtime."
*   **Trade-off Decisions:** You cannot build a system that is perfectly secure, infinitely fast, absolutely consistent, and completely free. System design helps you choose what to prioritize.
    *   *Example:* A banking app must prioritize **Consistency** (it must show your exact account balance, even if the app takes an extra second to load). A social media feed can prioritize **Availability and Speed** (it's okay if a post shows up a few seconds late, as long as the feed loads instantly).

---

## 2. What is a Server?

In simple terms, a **server** is a computer that "serves" something to another computer (the **client**). It is just like a normal computer (like your laptop) but built to run 24/7 without shutting down, usually has a much faster internet connection, and does not have a monitor, keyboard, or mouse attached.

### The Restaurant Analogy
Think of the client-server model like dining at a restaurant:

```
[ Customer ]  ====== Ordering Food (Request) ======>  [ Kitchen ]
 (Client)     <===== Serving Food (Response) ======    (Server)
```

1.  **The Client (The Customer):** You sit at a table and look at the menu. You decide what you want (e.g., "Show me my Facebook profile page").
2.  **The Request (The Order):** The waiter takes your order and walks it to the kitchen.
3.  **The Server (The Kitchen):** The kitchen (Server) receives the order. The chefs look up the recipe (Application Logic), grab ingredients from the pantry (Database), cook the meal (Processing), and place it on a plate.
4.  **The Response (The Meal):** The waiter carries the hot plate back to your table. You eat the meal (your browser renders the webpage).

### Common Types of Servers
*   **Web Server (The Host):** Sends the basic building blocks of a website (HTML files, images, CSS styling) to your browser when you type in a URL (like `www.google.com`).
*   **Database Server (The Filing Cabinet):** A specialized computer designed to store, organize, and quickly retrieve huge amounts of data (like user passwords, bank transactions, or photos).
*   **Application Server (The Brain):** Executes the rules of the app. For example, when you request a ride on Uber, the application server runs the math to match you with the closest driver and calculate the price.

### Where do Servers live?
*   **On-Premise (Physical):** Servers placed physically inside a company's office closet or building basement.
*   **Cloud (Virtual):** Renting space on massive computers owned by companies like Amazon (AWS), Google (Google Cloud), or Microsoft (Azure). These servers sit in massive, climate-controlled warehouses (Data Centers) around the world.

---

## 3. Latency and Throughput

Latency and Throughput are the two most important ways we measure how "fast" and "strong" a system is. 

### Latency (Speed/Delay)
*   **Layman Definition:** Latency is the **wait time** or the delay. It is how long it takes for a single request to travel from your device to the server and come back with the answer.
*   **Measurement:** Milliseconds (ms). (1 millisecond is 1/1,000th of a second).
*   **Example:** When you click "Play" on a video, latency is the split-second delay before the video actually starts playing.

### Throughput (Capacity/Volume)
*   **Layman Definition:** Throughput is the **capacity** or the volume of work. It is how many requests the system can successfully handle in one second.
*   **Measurement:** Queries Per Second (QPS) or Transactions Per Second (TPS).
*   **Example:** A highway toll booth that can process 50 cars per minute has a higher throughput than one that can only process 10 cars per minute.

---

### Comparison Analogies

#### The Highway Analogy
Imagine a highway connecting City A to City B:
*   **Latency:** The time it takes for **one car** to drive from City A to City B (e.g., 45 minutes).
*   **Throughput:** The **number of cars** that pass under a bridge on the highway every hour (e.g., 2,000 cars/hour).

```
       <----------- Latency: 45 minutes for one car ---------->
[ City A ] ==================== Highway ==================== [ City B ]
             ====> [Car 1]     ====> [Car 2]     ====> [Car 3]
       <----- Throughput: 2,000 cars passing per hour ------>
```

#### The Post Office Analogy (The Trade-off)
Imagine you are shipping letters from New York to London:

*   **Scenario A (A Fast Sports Car):**
    You hire a driver in a sports car to carry a single envelope across a bridge.
    *   **Latency is Low (Fast):** The letter arrives in just 1 hour.
    *   **Throughput is Low (Small Volume):** You can only deliver 1 letter per hour.
*   **Scenario B (A Giant Cargo Truck):**
    You load 10,000 envelopes into a large, slow shipping truck.
    *   **Latency is High (Slow):** The truck takes 6 hours to cross the bridge.
    *   **Throughput is High (Large Volume):** You delivered 10,000 letters in 6 hours (which averages to ~1,666 letters per hour!).

### Why Both Matter in System Design
When designing software, we want **low latency** (so the user gets an instant response) and **high throughput** (so the system doesn't crash when millions of users use it at the same time). 

However, they don't always go hand-in-hand:
*   If you build an app that runs complex calculations (like high-quality video encoding), it might take a long time to finish one video (High Latency). But by using many computers, you can process 1,000 videos at the same time (High Throughput).
*   If you build a real-time multiplayer shooting game, you need the bullets to register immediately (Ultra-Low Latency, under 20ms). If latency goes up, players will experience "lag," making the game unplayable, even if the game servers can handle millions of players at once (High Throughput).
