# 🚰 Leaky Bucket Algorithm

---

## 🧩 1. What Problem Were We Facing?

Imagine a backend system receiving **thousands of API requests per second** from users around the world.

### 🚨 Problem:
- Sudden bursts of traffic would **overload the server**.
- Users sending too many requests could impact others' experience.
- Systems crashed or slowed down during **unexpected traffic spikes**.
- No fairness — **1 heavy user** could affect **1000 normal users**.

> 📍Example: A login service where users are allowed only 5 attempts per minute. But due to bots or misconfigured clients, some users were bombarding the server, causing delays or failures for others.

---

## 💡 2. How Leaky Bucket Helped?

The Leaky Bucket algorithm came in to control the **flow of requests** — like how water leaks slowly from a bucket with a hole.

### 🔧 Concept:
- Incoming requests are added to a **bucket (queue)**.
- Requests leak out at a **fixed rate** (like 1 request per second).
- If the bucket is full (too many requests), **new ones are dropped**.

> This way, the server handles requests at a steady pace, protecting it from sudden floods.

---

## 🏗️ 3. Where to Place It in Architecture?

Leaky Bucket should be placed **at the entry point** of your system — where requests first arrive.

### 🔹 In Monolithic Architecture:

---

## 🧱 4. In Monolith vs Microservices

### 🏛️ Monolithic:
- Add the leaky bucket logic as a middleware or servlet filter.
- Handles requests before they reach the core application logic.

### 🧩 Microservices:
- Implement rate limiting at API Gateway or an **API Management layer**.
- Can be applied per service or per endpoint for granular control.

---

## ⚙️ 5. How the Algorithm Works

- The bucket has **fixed size** (capacity of how many requests it can hold).
- Requests **enter** the bucket (queue).
- A **leak rate** defines how fast requests are processed.
- If a request arrives when the bucket is full → it’s **rejected**.

> 🔄 The system processes requests at a steady rate, avoiding overload.

---

## 🌐 6. Distributed Environments

In a distributed setup:
- Use a **centralized store** like **Redis** to track the bucket state.
- Ensure atomic operations to prevent race conditions.
- Optionally use tools like **Envoy, Kong, NGINX**, or **API Gateway** with built-in rate-limiting.

---

## ✅ Pros and ❌ Cons

### ✅ Pros:
- Ensures **stable traffic rate** to backend systems.
- Smooths out request bursts.
- Simple and deterministic behavior.
- Easy to visualize and reason about.

### ❌ Cons:
- Does **not allow bursts** (unlike Token Bucket).
- Might **drop important requests** if bucket is full.
- Needs careful tuning of leak rate and bucket size.

---

## 📚 Real-World Case Study: Cloudflare

**Problem:**  
DDoS attackers were sending millions of requests, trying to take down services.

**Solution:**  
Cloudflare used the **Leaky Bucket algorithm** at the edge (before requests hit actual services).  
It processed only a limited number of requests per second and dropped the rest, protecting their backend.

**Impact:**  
- No more sudden overloads.  
- Fair access to genuine users.  
- Systems remained healthy under attack.

---

## ⚖️ Trade-Offs

| Trade-Off                   | Explanation                                                    |
|----------------------------|----------------------------------------------------------------|
| Fairness vs Flexibility    | Very fair — but doesn’t allow even small bursts                |
| Simplicity vs Responsiveness | Easy to implement — but slower to react to sudden user needs |
| Throughput vs Protection   | May sacrifice throughput to protect downstream systems         |

---

## 🧠 TL;DR

**Leaky Bucket** ensures your system doesn't get flooded with traffic by leaking requests at a **fixed rate**.  
It’s perfect when you need **predictable and stable load**, even if it means dropping extra requests.

> Best used in login systems, authentication services, and gateways that can’t afford sudden traffic surges.
