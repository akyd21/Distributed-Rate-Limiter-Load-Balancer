# 🚀 Distributed Rate Limiter & Load Balancer

A backend system built with **Java and Spring Boot** that combines a **Distributed Rate Limiter** and **Load Balancer** to control incoming traffic and distribute requests across multiple backend servers.

The project uses **Redis** to maintain rate-limit information in a distributed environment.

---

## 📌 Project Overview

Modern applications can receive thousands of requests from users at the same time.

Two common problems are:

1. **Too many requests** from a single user/IP can overload the system.
2. **Uneven traffic distribution** can cause one server to become overloaded while other servers remain underutilized.

This project solves both problems using:

* **Rate Limiter** → Controls how many requests a client can send.
* **Load Balancer** → Distributes allowed requests across multiple backend servers.
* **Redis** → Stores rate-limit data centrally.

### High-Level Flow

```text
                         CLIENT
                           |
                           v
                 +-------------------+
                 |   RATE LIMITER    |
                 |    Spring Boot    |
                 |      + Redis      |
                 +---------+---------+
                           |
                    Request Allowed?
                      /          \
                    YES           NO
                     |             |
                     v             v
             +---------------+   HTTP 429
             | LOAD BALANCER |   Too Many
             +-------+-------+   Requests
                     |
             +-------+-------+
             |       |       |
             v       v       v
         +-------+ +-------+ +-------+
         |Server1| |Server2| |Server3|
         +-------+ +-------+ +-------+
```

---

# 🎯 Objectives

The main objectives of this project are:

* Prevent API abuse.
* Protect backend servers from excessive traffic.
* Implement distributed rate limiting.
* Distribute traffic across multiple backend instances.
* Support multiple rate-limit strategies.
* Handle high traffic efficiently.
* Detect unhealthy backend servers.
* Demonstrate distributed-system concepts using Spring Boot and Redis.

---

# 🛠️ Technologies Used

| Technology                | Purpose                        |
| ------------------------- | ------------------------------ |
| Java                      | Programming language           |
| Spring Boot               | Backend framework              |
| Spring Data Redis         | Redis integration              |
| Redis                     | Distributed rate-limit storage |
| Spring Cloud LoadBalancer | Load balancing                 |
| Maven                     | Dependency management          |
| MySQL                     | Optional application database  |
| Postman                   | API testing                    |
| Docker                    | Optional containerization      |
| Git & GitHub              | Version control                |

---

# 🏗️ System Architecture

```text
                         +-------------+
                         |   Client    |
                         +------+------+
                                |
                                v
                    +-----------------------+
                    |    Rate Limiter       |
                    |                       |
                    | User / IP / API / Org |
                    +----------+------------+
                               |
                               v
                         +-----------+
                         |   Redis   |
                         +-----------+
                               |
                       Request Allowed
                               |
                               v
                    +-----------------------+
                    |    Load Balancer       |
                    +-----------+------------+
                                |
                +---------------+---------------+
                |               |               |
                v               v               v
         +-------------+ +-------------+ +-------------+
         | Backend #1  | | Backend #2  | | Backend #3  |
         | Spring Boot | | Spring Boot | | Spring Boot |
         +-------------+ +-------------+ +-------------+
```

---

# 🔐 Rate Limiter

The Rate Limiter controls the number of requests that a client can make within a specific time period.

For example:

```text
100 requests / minute / user
```

If a user sends:

```text
Request 1   → ALLOWED
Request 2   → ALLOWED
Request 3   → ALLOWED
...
Request 100 → ALLOWED
Request 101 → REJECTED
```

The rejected request returns:

```http
HTTP/1.1 429 Too Many Requests
```

---

# 🔑 Rate Limiting Strategies

This project can support multiple types of rate limiting.

## 1. Per User

Limits requests made by a particular authenticated user.

```text
User A → 100 requests/minute
User B → 100 requests/minute
```

User A and User B have separate limits.

---

## 2. Per IP

Limits requests based on the client's IP address.

```text
192.168.1.10 → 50 requests/minute
192.168.1.20 → 50 requests/minute
```

This is useful for controlling unauthenticated traffic.

---

## 3. Per API

Different APIs can have different limits.

```text
/api/login       → 5 requests/minute
/api/products    → 100 requests/minute
/api/payment     → 10 requests/minute
```

This allows sensitive APIs to have stricter limits.

---


---

# 🪣 Rate Limiting Algorithm

The project can use the **Token Bucket algorithm**.

### Example

Suppose:

```text
Bucket Capacity = 5 tokens
Refill Rate = 1 token/second
```

Initially:

```text
Bucket
+---+---+---+---+---+
| ● | ● | ● | ● | ● |
+---+---+---+---+---+
```

Each request consumes one token.

```text
Request → Token consumed
```

After 5 requests:

```text
+---+---+---+---+---+
|   |   |   |   |   |
+---+---+---+---+---+

No token → Request rejected
```

Tokens are continuously refilled according to the configured refill rate.

---

# ⚡ Why Redis?

Redis is used because the rate limiter needs a fast and shared storage system.

Without Redis:

```text
Server 1 → Counter = 10
Server 2 → Counter = 5
Server 3 → Counter = 8
```

Each server may maintain a different counter.

With Redis:

```text
             Redis
               |
       +-------+-------+
       |       |       |
    Server1 Server2 Server3
       \       |       /
        \      |      /
         Shared Counter
```

All backend instances can access the same rate-limit information.

---

# ⚖️ Load Balancer

After a request passes the Rate Limiter, the Load Balancer decides which backend server should handle it.

Example:

```text
Request 1 → Server 1
Request 2 → Server 2
Request 3 → Server 3
Request 4 → Server 1
Request 5 → Server 2
Request 6 → Server 3
```

This is an example of **Round Robin Load Balancing**.

---

# 🔄 Round Robin

Round Robin distributes requests sequentially.

```text
             Load Balancer
                   |
       +-----------+-----------+
       |           |           |
       v           v           v
    Server 1    Server 2    Server 3

Request 1 → Server 1
Request 2 → Server 2
Request 3 → Server 3
Request 4 → Server 1
Request 5 → Server 2
Request 6 → Server 3
```

---

# 🏥 Health Checking

The Load Balancer can check whether backend servers are healthy.

Example:

```text
Server 1 → UP   ✅
Server 2 → UP   ✅
Server 3 → DOWN ❌
```

The Load Balancer removes Server 3 from request distribution.

```text
             Load Balancer
                  |
          +-------+-------+
          |               |
          v               v
       Server 1         Server 2
          ✅                ✅

       Server 3
          ❌
        Removed
```

When Server 3 becomes healthy again, it can be added back to the pool.

---

# 📂 Project Structure

```text
rate-limiter-load-balancer/
│
├── src/
│   ├── main/
│   │   ├── java/
│   │   │   └── com.example.project/
│   │   │
│   │   │       ├── controller/
│   │   │       │   └── ApiController.java
│   │   │
│   │   │       ├── service/
│   │   │       │   ├── RateLimiterService.java
│   │   │       │   └── LoadBalancerService.java
│   │   │
│   │   │       ├── config/
│   │   │       │   ├── RedisConfig.java
│   │   │       │   └── LoadBalancerConfig.java
│   │   │
│   │   │       ├── model/
│   │   │       │   └── RateLimitConfig.java
│   │   │
│   │   │       ├── repository/
│   │   │       │   └── RateLimitRepository.java
│   │   │       │
│   │   │       └── ProjectApplication.java
│   │   │
│   │   └── resources/
│   │       └── application.properties
│   │
│   └── test/
│
├── pom.xml
├── README.md
└── docker-compose.yml
```

---

# 🔄 Request Processing

Every request follows these steps:

### Step 1 — Client sends request

```http
GET /api/products
Authorization: Bearer <token>
```

### Step 2 — Identify client

The system identifies the client using:

```text
User ID
IP Address
API
Organization ID
```

### Step 3 — Check Redis

The Rate Limiter checks the current request count/token information.

### Step 4 — Rate Limit Decision

If the request is within the limit:

```text
ALLOW ✅
```

Otherwise:

```text
REJECT ❌
HTTP 429
```

### Step 5 — Load Balancing

Allowed requests are sent to a healthy backend server.

```text
Rate Limiter
     |
     v
Load Balancer
     |
     +----> Server 1
     |
     +----> Server 2
     |
     +----> Server 3
```

### Step 6 — Backend Response

The selected server processes the request and sends the response back to the client.

---

# 🧪 Example API

## Get Products

```http
GET /api/products
```

Response:

```json
{
  "success": true,
  "message": "Products fetched successfully"
}
```

---

# 🚫 Rate Limit Exceeded

When the limit is exceeded:

```http
HTTP/1.1 429 Too Many Requests
```

Response:

```json
{
  "success": false,
  "message": "Rate limit exceeded. Try again later."
}
```

---

# ⚙️ Configuration Example

```properties
spring.application.name=rate-limiter

spring.data.redis.host=localhost
spring.data.redis.port=6379

rate-limit.capacity=100
rate-limit.refill-rate=100
rate-limit.window=60
```

---

# 🐳 Running Redis with Docker

If Docker is installed:

```bash
docker run -d \
  --name redis \
  -p 6379:6379 \
  redis
```

Check Redis:

```bash
docker ps
```

---

# ▶️ Running the Application

### 1. Clone the repository

```bash
git clone <your-repository-url>
```

### 2. Go to the project

```bash
cd rate-limiter-load-balancer
```

### 3. Start Redis

```bash
docker start redis
```

### 4. Build the project

```bash
mvn clean install
```

### 5. Run Spring Boot

```bash
mvn spring-boot:run
```

---

# 🧪 Testing

The project can be tested using **Postman**.

Test cases should include:

### Test 1 — Normal Request

```text
Request → Rate Limiter → Load Balancer → Server
```

Expected:

```text
200 OK
```

### Test 2 — Rate Limit Exceeded

Send requests continuously until the configured limit is reached.

Expected:

```text
429 Too Many Requests
```

### Test 3 — Load Distribution

Send multiple requests.

Expected:

```text
Request 1 → Server 1
Request 2 → Server 2
Request 3 → Server 3
Request 4 → Server 1
```

### Test 4 — Server Failure

Stop one backend server.

Expected:

```text
Load Balancer
      |
      +---- Server 1 ✅
      +---- Server 2 ❌
      +---- Server 3 ✅
```

Traffic should continue through healthy servers.

---

# 📊 Future Enhancements

Possible improvements include:

* [ ] Token Bucket Rate Limiter
* [ ] Sliding Window Rate Limiter
* [ ] Per-user limits
* [ ] Per-IP limits
* [ ] Per-API limits
* [ ] Per-organization limits
* [ ] Round Robin Load Balancing
* [ ] Weighted Load Balancing
* [ ] Health checks
* [ ] Automatic server registration
* [ ] Redis Cluster
* [ ] Docker deployment
* [ ] Prometheus monitoring
* [ ] Grafana dashboard
* [ ] Authentication and authorization
* [ ] Admin dashboard
* [ ] Rate-limit configuration through REST APIs

---

# 🎓 Learning Outcomes

By completing this project, you will learn:

* Spring Boot
* Redis
* Distributed systems
* Rate limiting
* Token Bucket algorithm
* Load balancing
* Round Robin algorithm
* Health checking
* REST APIs
* Scalability
* Fault tolerance
* Concurrency
* Docker
* System design

---

# 📌 Project Goals

The final system should be capable of:

```text
                    Client
                       |
                       v
               Rate Limiter
                       |
                 Redis Check
                       |
             +---------+---------+
             |                   |
          Allowed              Denied
             |                   |
             v                   v
       Load Balancer           HTTP 429
             |
       +-----+-----+
       |     |     |
       v     v     v
     API-1 API-2 API-3
       |     |     |
       +-----+-----+
             |
          Response
             |
             v
           Client
```

---

# 👨‍💻 Author

**Akash Raj**

Java | Spring Boot | Redis | Backend Development

---

# ⭐ Conclusion

This project demonstrates how **Rate Limiting and Load Balancing can work together without using an API Gateway**.

The Rate Limiter controls and protects the system from excessive requests, while the Load Balancer distributes the allowed requests among multiple backend servers.

Together, they provide a foundation for building a **scalable, reliable, and fault-tolerant backend system**.
#   D i s t r i b u t e d - R a t e - L i m i t e r - L o a d - B a l a n c e r 
 
 
