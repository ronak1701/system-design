# Proxy

This guide covers Proxies, explaining the difference between Forward Proxies and Reverse Proxies using simple real-world analogies, and demonstrates how to build your own path-based Reverse Proxy in Node.js.

> **Reference Material:** This guide is based on the article [System Design for Beginners: Everything you need in one article](https://medium.com/@shivambhadani_/system-design-for-beginners-everything-you-need-in-one-article-c74eb702540b) by Shivam Bhadani.

---

## 1. What is a Proxy?

In computer networks, a **Proxy** (or proxy server) is a middleman computer that sits between a client (like your laptop) and a server (like Google.com) and passes network traffic back and forth between them.

Instead of your computer talking directly to a server, it talks to the proxy, which then talks to the server on your behalf. 

```
Direct Connection:
[ Your Computer ] =====================================> [ Website Server ]

With a Proxy (Middleman):
[ Your Computer ] ======> [ Proxy Server (Middleman) ] ======> [ Website Server ]
```

---

## 2. Forward Proxy

A **Forward Proxy** sits in front of the **client** (user). When a user makes a request to access a website, the request goes to the forward proxy first. The proxy then retrieves the webpage from the internet and sends it back to the user.

```
                  +----------------------------------+
                  |           CLIENT SIDE            |
                  |                                  |
[ Client 1 ] --->+   +----------------------------+  |                  +----------------+
                 |   |                            |  |                  |                |
[ Client 2 ] --->+   |  Forward Proxy (Middleman) +=====( Internet )====>  Web Server    |
                 |   |                            |  |                  |                |
[ Client 3 ] --->+   +----------------------------+  |                  +----------------+
                  +----------------------------------+
```

### The Secret Agent Analogy
Imagine you want to buy a package from a store, but you don't want the store owner to know who you are, where you live, or what your face looks like. 
*   Instead of going yourself, you hire a **Secret Agent (Forward Proxy)**.
*   You tell the agent: *"Go buy this book for me."*
*   The agent goes to the store, buys the book under their own name, and brings it back to you.
*   The store owner knows a book was bought, but they only saw the agent's ID. **Your identity is completely hidden.**

### Key Use Cases
*   **Anonymity & Privacy:** Hiding your real IP address from websites you visit.
*   **Bypassing Restrictions (VPNs):** Accessing websites that are blocked in your country or office network.
*   **Content Filtering:** Schools or companies use forward proxies to block students or employees from accessing distracting websites (like social media).

---

## 3. Reverse Proxy

A **Reverse Proxy** sits in front of the **backend servers**. When a client makes a request to a website, the request goes to the reverse proxy. The proxy determines which internal backend server should handle the request, fetches the response, and returns it to the client.

The client has no idea which specific backend server processed their request.

```
                                                     +-----------------------------+
                                                     |         SERVER SIDE         |
                                                     |                             |
                 +----------------------------+      |   +-----------------------+ |
                 |                            |      |   |  Product Microservice | |
[ Client ] ====> |  Reverse Proxy (Receiver)  +=========>|  (Port 3001)          | |
                 |                            |      |   +-----------------------+ |
                 +----------------------------+      |   +-----------------------+ |
                                                     |   |  Order Microservice   | |
                                                     |   |  (Port 3002)          | |
                                                     |   +-----------------------+ |
                                                     +-----------------------------+
```

### The Restaurant Receptionist Analogy
Imagine you walk into a large, busy restaurant:
*   You do not walk directly into the kitchen, choose a chef, and tell them what to cook.
*   Instead, you talk to the **Receptionist at the front desk (Reverse Proxy)**.
*   You place your order. The receptionist looks at the kitchen and passes your order to an available chef who specializes in that dish.
*   Once cooked, the receptionist brings the plate to your table.
*   **You get your meal, but you have no idea which chef cooked it.** The kitchen's internal workings are hidden from you.

### Key Use Cases
*   **Security:** Hiding the IP addresses of your backend servers so hackers cannot attack them directly.
*   **Load Balancing:** Evenly distributing incoming user traffic across multiple servers so no single server crashes from overload.
*   **SSL Termination:** Handling the encryption (HTTPS) at the proxy level. The internal backend servers can communicate using fast, unencrypted HTTP.
*   **Caching:** Storing static files (like images and HTML) on the proxy so it can serve them instantly without bothering the backend databases.

---

## 4. Building our own Reverse Proxy

We can build a simple, path-based reverse proxy using native Node.js. 

In this example, our proxy will listen on port `8080`.
*   If a request comes in for `/products`, it routes the request to a mock **Product Microservice** running on port `3001`.
*   If a request comes in for `/orders`, it routes the request to a mock **Order Microservice** running on port `3002`.

### The Code (`proxy.js`)

Create a file named `proxy.js` and paste the following code:

```javascript
const http = require('http');

// Define where our backend microservices are running
const TARGETS = {
  '/products': { host: 'localhost', port: 3001 },
  '/orders': { host: 'localhost', port: 3002 }
};

// Create the Reverse Proxy Server
const proxy = http.createServer((clientReq, clientRes) => {
  // Find which service should handle this path
  const path = clientReq.url;
  let target = null;

  // Simple path matching
  if (path.startsWith('/products')) {
    target = TARGETS['/products'];
  } else if (path.startsWith('/orders')) {
    target = TARGETS['/orders'];
  }

  // If the path doesn't match any service, return 404
  if (!target) {
    clientRes.writeHead(404, { 'Content-Type': 'text/plain' });
    clientRes.end('Error: Service Not Found');
    return;
  }

  console.log(`[Proxy] Routing request ${clientReq.method} ${path} -> Port ${target.port}`);

  // Configure the request we will make to the backend service
  const proxyOptions = {
    host: target.host,
    port: target.port,
    path: clientReq.url,
    method: clientReq.method,
    headers: clientReq.headers // Forward headers (like Auth tokens)
  };

  // Forward the request to the target microservice
  const proxyReq = http.request(proxyOptions, (targetRes) => {
    // Forward the status code and headers back to the client
    clientRes.writeHead(targetRes.statusCode, targetRes.headers);
    
    // Pipe the response data from the backend back to the client
    targetRes.pipe(clientRes);
  });

  // Handle errors (e.g. target microservice is offline)
  proxyReq.on('error', (err) => {
    console.error(`[Proxy Error] Failed to connect to port ${target.port}:`, err.message);
    clientRes.writeHead(502, { 'Content-Type': 'text/plain' });
    clientRes.end('Error: Bad Gateway (Target service is offline)');
  });

  // Pipe the request body data (if any, like POST JSON data) to the backend
  clientReq.pipe(proxyReq);
});

// Start our Reverse Proxy
proxy.listen(8080, () => {
  console.log('Reverse Proxy is running on http://localhost:8080');
});
```

### How to Test It

1.  **Start the Mock Product Service** (in a terminal):
    ```javascript
    // Start a mock server on port 3001
    require('http').createServer((req, res) => {
      res.end('Product list: [Apples, Oranges, Bananas]');
    }).listen(3001);
    ```
2.  **Start the Mock Order Service** (in a terminal):
    ```javascript
    // Start a mock server on port 3002
    require('http').createServer((req, res) => {
      res.end('Order list: [Order #101 - Placed]');
    }).listen(3002);
    ```
3.  **Run the Proxy:** `node proxy.js`
4.  **Send Requests:**
    *   Visiting `http://localhost:8080/products` will return the response from the Product service on port 3001.
    *   Visiting `http://localhost:8080/orders` will return the response from the Order service on port 3002.
    *   Visiting `http://localhost:8080/other` will return `Error: Service Not Found`.

---

### Summary Checklist for Beginners
*   A **Proxy** is a middleman server passing traffic between client and server.
*   **Forward Proxies** protect the **client's** identity (like a secret agent buying tickets).
*   **Reverse Proxies** protect the **server's** identity (like a receptionist coordinating order routing).
*   You can build a path-based reverse proxy in Node.js by intercepting requests and forwarding them to specific ports using `http.request`.
