## ✅ **Top Microservices Design Patterns (with real-world use cases & Java/Spring Cloud examples)**

---

### 1. **API Gateway Pattern**

**What it does:**
Single entry point for all client requests. It routes, authenticates, and transforms requests.

**Why it's used:**

* Avoid exposing internal services.
* Handle cross-cutting concerns (auth, rate limiting, logging).

**Example (Spring Cloud Gateway):**

```yaml
# application.yml
spring:
  cloud:
    gateway:
      routes:
        - id: user-service
          uri: http://localhost:8081
          predicates:
            - Path=/users/**
```

🟢 Tools: **Spring Cloud Gateway**, **Zuul**, **Kong**, **NGINX**
🟠 Interview Tip: Know how it differs from Load Balancer.

---

### 2. **Service Discovery Pattern**

**What it does:**
Services register themselves so others can discover them dynamically.

**Why it's used:**

* Services come and go, scale in/out.
* Avoid hardcoding service URLs.

**Example (Spring Cloud + Eureka):**

```java
@EnableEurekaClient // on the microservice
public class ProductServiceApp { }
```

🟢 Tools: **Netflix Eureka**, **Consul**, **Zookeeper**
🟠 Interview Tip: Be ready to explain client-side vs server-side discovery.

---

### 3. **Circuit Breaker Pattern**

**What it does:**
Prevents cascading failures by breaking the connection to failing services temporarily.

**Why it's used:**

* Improves fault tolerance.
* Avoids overloading slow or down services.

**Example (Resilience4j):**

```java
@CircuitBreaker(name = "orderService", fallbackMethod = "fallbackOrder")
public String placeOrder() {
    // call to external service
}
public String fallbackOrder(Throwable t) {
    return "Service temporarily unavailable. Please try later.";
}
```

🟢 Tools: **Resilience4j**, **Hystrix (deprecated)**
🟠 Interview Tip: Mention fallback, timeout, retry, and threshold.

---

### 4. **Config Server Pattern**

**What it does:**
Centralized configuration management for all microservices.

**Why it's used:**

* Avoid duplication of configs.
* Easy config updates without redeploy.

**Example (Spring Cloud Config):**

```properties
# application.properties of microservice
spring.config.import=optional:configserver:http://localhost:8888
```

🟢 Tool: **Spring Cloud Config Server**
🟠 Interview Tip: Know how config refresh works using Actuator and Bus.

---

### 5. **Strangler Fig Pattern**

**What it does:**
Gradually replaces a legacy monolith by building new microservices around it.

**Why it's used:**

* Avoid big-bang rewrite.
* Enable progressive migration.

🟢 Use Case: Replace existing functionality piece-by-piece.

🟠 Interview Tip: Give example like redirecting `/orders` endpoint to new microservice, but keeping old endpoints in monolith.

---

### 6. **Bulkhead Pattern**

**What it does:**
Isolates failures to one service or thread pool so it doesn’t bring down the entire system.

**Why it's used:**

* Prevent one slow component from affecting others.

**Example (Resilience4j bulkhead):**

```java
@Bulkhead(name = "inventoryService", type = Bulkhead.Type.THREADPOOL)
public String getInventory() { ... }
```

🟠 Interview Tip: Compare it to compartments in a ship — even if one floods, others stay afloat.

---

### 7. **Saga Pattern (For Distributed Transactions)**

**What it does:**
Manages data consistency across services using a sequence of local transactions.

**Why it's used:**

* No 2-phase commit in microservices.
* Each service updates its data and triggers the next.

**Types:**

* **Choreography** – no central coordinator; services react to events.
* **Orchestration** – central coordinator tells each service what to do.

**Example (Choreography):**

* Order Service → Event: `OrderCreated`
* Inventory Service → Listens & reserves stock → Emits `StockReserved`
* Payment Service → Listens & processes payment...

🟠 Interview Tip: Clearly explain both types with sequence examples.

---

### 8. **Database per Service Pattern**

**What it does:**
Each microservice manages its own database.

**Why it's used:**

* Avoid tight coupling.
* Better autonomy and scalability.

🟢 Combine with Saga for consistency
🟠 Interview Tip: Know the trade-offs (joins, reporting difficulty).

---

### 9. **CQRS (Command Query Responsibility Segregation)**

**What it does:**
Separates read and write operations into different models.

**Why it's used:**

* Performance optimization.
* Different scaling strategies.

**Use Case:** Write uses Kafka + DB, Read uses Materialized View or Cache.

🟠 Interview Tip: Mention it helps in eventual consistency + high read performance.
