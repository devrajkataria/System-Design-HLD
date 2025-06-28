# 🚦 Rate Limiting & Throttling - System Design Notes

---

## 1️⃣ The Real-World Problem (Story Time)

Imagine you're running a ticket booking platform (like BookMyShow). A big movie drops and everyone tries to book at the same time. Suddenly:

- Your servers crash.
- Real users can't access the site.
- Bots flood the system with 1000s of requests.
- Backend services are overwhelmed.

Result? Downtime, angry users, lost revenue. 🚨  
That’s where **Rate Limiting & Throttling** come to the rescue!

---

## 2️⃣ The Solution: Rate Limiting & Throttling

### ✅ Rate Limiting

Rate Limiting controls **how many requests** a user/client/IP can make within a specific time window.

Example: “User can only send 100 requests per minute.”

### ✅ Throttling

Throttling kicks in **after** the limit is hit. It decides whether to:

- 🚫 Reject the request (usually with `429 Too Many Requests`)
- ⏳ Delay or queue it (so system doesn’t crash)

---

## 3️⃣ Where Does It Fit in Architecture?



- Rate Limiting usually goes at the **edge** (like an API Gateway or Load Balancer).
- Throttling can also be applied deeper in services for critical APIs.

---

## 4️⃣ Monolithic vs Microservices

### 🧱 In Monolithic Applications:

- Use in-memory or Redis counters within the app.
- Libraries like Bucket4j (Java), express-rate-limit (Node.js) help.
- Diagram shown below, how it should be used in monolithic applicaiton.

📝 How It Works:

You can use in-memory or Redis-based rate limit counters inside the app.

Middleware inside the monolith checks rate limits before processing requests.

Throttling logic (like sleep/delay or rejection) is also in the app.

                   +--------------------+
                   |      Client        |
                   +--------------------+
                             |
                             v
                   +--------------------+
                   |   Load Balancer    |
                   +--------------------+
                             |
                             v
         +-----------------------------------------+
         |      Monolithic Application             |
         |-----------------------------------------|
         |  1. Authentication Module                |
         |  2. Business Logic                       |
         |  3. Rate Limiting Middleware (in-app)    | <--- ✅ Apply Here
         |  4. Throttling Logic (in-app queue/retry)| <--- ✅ Apply Here
         |  5. Database Access                      |
         +-----------------------------------------+
                             |
                             v
                   +--------------------+
                   |     Database       |
                   +--------------------+


### 🧱 In Microservices Architecture:

- Add rate limiting at the **API Gateway** (Kong, AWS Gateway, NGINX, etc.).
- Use shared cache (like Redis) to maintain distributed limits.
- Diagram shown below, how it should be used in monolithic applicaiton.

📝 How It Works:

Use a central API Gateway to apply global rate limits per user, token, or IP.

Inside each microservice, you can still add local throttling logic for critical endpoints.

Redis (or similar) is used to store rate limit counters across services in a distributed setup.

                   +--------------------+
                   |      Client        |
                   +--------------------+
                             |
                             v
                   +--------------------+
                   |    API Gateway     |  <--- ✅ Apply Rate Limiting Here
                   | (e.g., Kong, AWS)  |
                   +--------------------+
                             |
                             v
           +-------------------------------------+
           |         Service Discovery /         |
           |        Load Balancer (Optional)     |
           +-------------------------------------+
                             |
        +-----------------------------------------------+
        |                Microservices                  |
        |-----------------------------------------------|
        |  Auth Service     🡒 Throttling Middleware     |
        |  User Service     🡒 Redis-based counters      |
        |  Payment Service  🡒 Circuit Breakers          |
        |  Notification     🡒 Rate Limit (per endpoint) |
        +-----------------------------------------------+
                             |
                             v
                   +--------------------+
                   |   Shared Database  |
                   +--------------------+


---

## 5️⃣ Algorithms / Types of Rate Limiting

| Algorithm        | Simple Explanation                                      | When to Use                     |
|------------------|----------------------------------------------------------|----------------------------------|
| Fixed Window      | Counter resets after fixed time (e.g., per minute)       | Simple use-cases                |
| Sliding Window    | More accurate, considers sliding time frame              | When fairness is important       |
| Token Bucket      | Tokens refill at fixed rate; requests consume tokens     | Allow burst + steady traffic     |
| Leaky Bucket      | Requests flow out at constant rate (like water leaking) | Smooth request handling          |

---

## 6️⃣ Using in a Distributed Environment

When you scale horizontally (multiple servers), rate limit tracking must be shared:

- Use **Redis** or similar distributed cache to store counters.
- Ensure atomicity using **Lua scripts** or Redis atomic commands.
- You can hash user/IP for sharding counters across nodes.

---

## 7️⃣ Pros and Cons

### ✅ Pros

- Keeps system stable under heavy traffic
- Prevents abuse & bot attacks
- Ensures fair usage across users
- Reduces cloud/server cost

### ❌ Cons

- If misconfigured, real users may get blocked
- Adds some complexity (especially distributed)
- Throttling can cause delay

---

## 8️⃣ Real-World Case Study: GitHub API

GitHub has strict rate limits:

- 60 requests/hour (unauthenticated)
- 5000 requests/hour (authenticated)
- On hitting limit:
  - You get `429 Too Many Requests`
  - Response also tells when the limit resets

This protects GitHub from abuse and ensures API remains available for all.

---

## 9️⃣ Trade-offs

| Factor               | With Rate Limiting         | Without Rate Limiting        |
|----------------------|----------------------------|-------------------------------|
| System Stability     | ✅ Reliable                | ❌ High risk of overload       |
| User Fairness        | ✅ Everyone gets a turn    | ❌ Bots may hog all traffic    |
| Performance          | ⚠️ Slight delay on limit   | ✅ Fast unless overloaded      |
| Setup Complexity     | ❌ Needs setup & testing   | ✅ Easy but risky              |

---

## 🔧 Tools & Libraries to Use

- 🔹 **Redis** – For storing counters
- 🔹 **NGINX** – Built-in rate limiting modules
- 🔹 **API Gateways** – Kong, AWS API Gateway, Apigee
- 🔹 **Java** – Bucket4j, Resilience4j
- 🔹 **Node.js** – express-rate-limit
- 🔹 **Spring Boot** – Use filters + Redis + Bucket4j

---

## 🔚 Final Thoughts

Rate Limiting and Throttling are **essential** in any production system. They protect your APIs, backend, and infra from getting abused or overloaded.

Netflix, GitHub, Twitter — all major platforms use it.  
So whether you’re building a small app or a massive system — **add this to your design toolbox**. 🧰

---


