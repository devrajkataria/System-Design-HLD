# 🪣 Token Bucket Algorithm 

---

## 📌 Problem Before Token Bucket (Real-World Story)

Imagine you are running a **payment gateway** like Razorpay or Paytm.  
Some clients are fair and send 2-3 requests per second.  
But one day, a **misconfigured client app** starts sending **1000 requests per second**.

🔴 **What happens?**
- Your **backend crashes** or gets overwhelmed.
- **Genuine users** face delays or timeouts.
- You lose **money** and **trust**.

Traditional systems didn’t have any mechanism to limit the request rate. You either served all requests or rejected them randomly once the system was overloaded.

---

## 💡 How Token Bucket Solves This

Token Bucket is a smart algorithm that **controls the rate of incoming requests** while still allowing **bursts** when needed.

### 🧠 How It Works:
- Imagine a **bucket** that holds tokens.
- Tokens are added at a **fixed rate** (e.g., 1 token every 100ms).
- Every incoming request must **take a token** to proceed.
- If the bucket is **empty**, the request is **rejected or delayed**.
- The bucket has a **max capacity**, so it can handle sudden bursts.

✅ **Example:**  
Your system adds 5 tokens/sec and the bucket holds max 10 tokens.  
A client sending 10 requests in one second can be served (if tokens are available).  
But if it sends 20 requests, 10 will be **denied or delayed**.

---

## 🏗️ Where to Place Token Bucket in Your Architecture?

You should place it **before your core business logic**, like:

- In **API Gateway** (e.g., Kong, NGINX, AWS API Gateway)
- In **Middleware** layer of your backend
- As part of **service-level** rate-limiting if you use microservices

---

## 🧱 Monolithic vs Microservice – How and Where to Use?

### 🧩 Monolithic Architecture:

    +------------+           +----------------------------+          +-------------------+           +-------------+
    |            |           |                            |          |                   |          |             |
    |   Client   + --------->+  Token Bucket Middleware   + -------->+  Business Logic   + -------->+   Database  |
    |            |           |  (e.g., Filter/Interceptor)|          |                   |          |             |
    +------------+           +----------------------------+          +-------------------+          +-------------+

- Token bucket logic is written in app code or middleware (e.g., Spring Filter, Express.js Middleware)

---

### 🧩 Microservices Architecture:

                      +-----------+
                      |  Client   |
                      +-----------+
                           |
                           v
                +------------------------+
                |   API Gateway / Ingress|
                |  (Token Bucket Applied)|
                +------------------------+
                      /           \
                     /             \
            +--------------+   +--------------+
            |  Service A   |   |  Service B   |
            +--------------+   +--------------+
                   |                  |
             +-----------+      +-----------+
             | Database A|      | Database B|
             +-----------+      +-----------+

- Token bucket logic is typically in API Gateway or Service Mesh (e.g., Istio + Envoy)
- Can also be in **sidecar containers**

---

## 🔢 Related Algorithms You Should Know

| Algorithm       | Allows Bursts? | Complexity | Use Case                          |
|----------------|----------------|------------|-----------------------------------|
| Token Bucket    | ✅ Yes         | ⭐⭐        | Bursty traffic with control       |
| Leaky Bucket    | ❌ No          | ⭐         | Smooth, constant-rate traffic     |
| Fixed Window    | ✅ Yes         | ⭐         | Simple counters, less accurate    |
| Sliding Window  | ✅ Yes         | ⭐⭐⭐       | More accurate, complex to manage  |

---

## 🌍 Using Token Bucket in a Distributed Environment

In distributed systems (multi-node):

- **Shared state needed** across nodes.
- Use **Redis**, **Memcached**, or **ZooKeeper** to store and manage token counts.
- Some cloud tools like **AWS API Gateway** or **Kong** already support distributed rate limiting.

Example using Redis:

    Client 
      ↓
    API Gateway or Middleware
      ↓
    Check token count in Redis (atomic operation)
      ├─✅ If token is available:
      │     • Decrement token count
      │     • Allow the request to proceed to the backend
      │
      └─❌ If no tokens are available:
            • Reject the request with HTTP 429 (Too Many Requests)


### 🔁 Notes

- **Token Refill Strategy**:  
  Tokens are added at a fixed rate (e.g., 1 token every 200ms) by a background job or using Redis’ TTL-based approach.

- **Client Isolation**:  
  Each client/IP/token uses a separate Redis key to manage its bucket, enabling fair usage and isolation.

- **Atomicity**:  
  To ensure race conditions don’t occur in distributed systems, Redis atomic operations like `INCR`, `DECR`, or **Lua scripts** are used.

- **Advantages of Redis**:
  - Fast in-memory operations
  - TTL support
  - Easily scalable using Redis Cluster
  - Supports atomic updates which are crucial for rate limiting

> This Redis-based design is lightweight, fast, and ideal for both monolithic and distributed microservices architectures.

---

## ✅ Pros and ❌ Cons

### ✅ Pros:
- Allows bursty traffic — not too strict.
- Easy to understand and implement.
- Flexible — supports different users/apps with custom configurations.
- Can be extended for:
  - Per-user limits
  - Per-IP limits
  - Per-endpoint limits

### ❌ Cons:
- Requires time synchronization and locking in distributed environments.
- Bursts may still overload downstream systems if not tuned properly.
- Slightly more complex than fixed window counters.

---

## 📚 Real-World Case Study: Stripe

**Problem:**  
High-volume merchants were sending a large number of requests, which risked degrading Stripe’s service for other users.

**Solution:**  
Stripe implemented **Token Bucket-based rate limiting per API key**.  
This allowed:
- Temporary spikes (useful for retry logic or batch jobs)
- But still enforced a maximum request rate per second

**Outcome:**  
- Ensured **fair usage** across clients  
- Prevented **API abuse**  
- Improved **reliability and stability** of backend systems

---

## ⚖️ Trade-Offs

| Trade-Off                     | Explanation                                                                 |
|------------------------------|-----------------------------------------------------------------------------|
| Bursty traffic vs Fairness   | Token Bucket allows bursts, but bucket size must be tuned for fairness     |
| Simplicity vs Control        | More flexible than fixed window, but needs more configuration               |
| Local vs Distributed         | Local is faster, distributed (e.g., with Redis) adds complexity             |
| Accuracy vs Performance      | More accurate rate limit = higher coordination/latency                      |

---

## 🧠 TL;DR

**Token Bucket** is a flexible and burst-friendly rate-limiting algorithm.  
It’s perfect for protecting APIs and services while allowing short traffic spikes.

> 📌 Best suited for: API Gateways, Microservices, Authentication Systems, and Billing APIs.



