Sure! Let's go deep into the **Saga Pattern**, especially within the context of **microservices**, and explore both **choreography** and **orchestration** techniques. Then, we’ll cover **how to implement these in Java** with practical libraries and tools.

---

## 🧩 What is the Saga Pattern?

The **Saga Pattern** is a **microservices-based transaction management pattern** used to maintain **data consistency** across **distributed services**.

When a single transaction spans multiple services (and databases), you can't use a traditional ACID transaction. So, Sagas break it into a **sequence of local transactions** and define **compensating actions** to undo operations if something fails.

---

## 🔄 Key Concepts:

* **Local transaction**: A transaction that updates a service’s local database.
* **Compensating transaction**: An operation to undo the effects of a previous successful transaction in case of failure.

---

## 🎭 Two Saga Coordination Techniques

### 1. **Choreography** (Decentralized)

* Each service **listens for events** and **produces events**.
* No central controller.
* Services decide what to do based on events from others.

#### 🧠 Flow:

```text
Service A → emits Event A → 
Service B listens → does task B → emits Event B →
Service C listens → does task C → emits Event C → ...
```

#### ✅ Pros:

* Simple and scalable.
* Loosely coupled.

#### ❌ Cons:

* Hard to track and debug.
* Complex business logic is harder to coordinate.

---

### 2. **Orchestration** (Centralized)

* A central **orchestrator service** controls the saga.
* It tells each service what to do next and handles success/failure.

#### 🧠 Flow:

```text
Orchestrator → tells Service A → wait for response →
Orchestrator → tells Service B → wait for response → ...
```

#### ✅ Pros:

* Centralized control.
* Easier to implement complex workflows.

#### ❌ Cons:

* Orchestrator becomes a **central point of failure**.
* Slightly more coupling.

---

## 🛠️ How to Implement Saga Pattern in Java

### Libraries/Tools:

* **Axon Framework**
* **Eventuate Tram Saga** (by Chris Richardson)
* **Camunda / Zeebe** (for orchestration)
* **Apache Kafka / RabbitMQ** (for choreography)
* **Spring Boot** (as a base framework)

---

## ✅ Example 1: Choreography Saga using Spring Boot + Kafka

### Microservices:

* **Order Service**
* **Inventory Service**
* **Payment Service**

### Steps:

1. **Order Service** creates an order and publishes `OrderCreated` event.
2. **Inventory Service** listens, reserves inventory, and emits `InventoryReserved`.
3. **Payment Service** listens, deducts payment, and emits `PaymentCompleted`.
4. On failure, services emit compensation events (`PaymentFailed`, `InventoryRelease`).

### Tools:

* `spring-kafka`
* Event classes + Kafka topics
* Compensation logic inside listeners

---

## ✅ Example 2: Orchestration Saga using Camunda / Zeebe

### Tools:

* Camunda BPM or Zeebe Engine
* Spring Boot + Camunda Starter
* BPMN diagram to define saga flow

### Steps:

1. Define a **BPMN workflow**:

   * Create order → Reserve inventory → Deduct payment
2. Each service exposes **REST endpoints**.
3. Camunda invokes REST tasks and handles failures using compensation tasks.

### Java Code Sketch:

```java
@ExternalTaskSubscription("reserve-inventory")
public class InventoryWorker implements ExternalTaskHandler {
  @Override
  public void execute(ExternalTask task, ExternalTaskService service) {
    try {
      // Call inventory service
      service.complete(task);
    } catch (Exception e) {
      service.handleFailure(task, "Inventory error", e.getMessage(), 0, 1000);
    }
  }
}
```

---

## ⚖️ When to Use What?

| Scenario                       | Use Choreography | Use Orchestration |
| ------------------------------ | ---------------- | ----------------- |
| Simple flow with few services  | ✅ Yes            | Optional          |
| Complex workflows with retries | ❌ Hard to manage | ✅ Easier          |
| Loose coupling is a must       | ✅ Yes            | ❌ Less            |
| Full traceability is required  | ❌ Difficult      | ✅ Central control |
| You use a BPM engine           | ❌ Not needed     | ✅ Use it          |

---

## 🧪 Testing and Challenges

* **Idempotency**: Services should handle duplicate events.
* **Ordering**: Message brokers like Kafka help preserve order.
* **Compensation logic**: Should be well-tested.
* **Observability**: Use distributed tracing (e.g., Zipkin, Jaeger).

## **simple working example of a Spring Boot + Kafka Choreography-based Saga** involving three services:

### 🧾 Scenario: Order Placement Workflow

Services involved:

1. **Order Service** – Creates an order and emits `OrderCreated` event.
2. **Inventory Service** – Reserves inventory after receiving the event.
3. **Payment Service** – Deducts payment after inventory is reserved.

---

### 🔧 Tech Stack

* **Spring Boot**
* **Apache Kafka**
* **Spring Kafka**
* JSON-based events

---

## 📁 Project Structure

Each service can be a separate Spring Boot project or module:

```
order-service/
inventory-service/
payment-service/
common/ (shared event classes)
```

---

## 🧱 Common: Shared Event Classes

```java
// OrderCreatedEvent.java
public class OrderCreatedEvent {
    private String orderId;
    private String productId;
    private int quantity;
    // Getters, setters, constructor
}

// InventoryReservedEvent.java
public class InventoryReservedEvent {
    private String orderId;
    private boolean success;
    // Getters, setters, constructor
}
```

---

## 1️⃣ Order Service

### application.yml

```yaml
spring:
  kafka:
    bootstrap-servers: localhost:9092
```

### Kafka Producer

```java
@Service
public class OrderService {

    @Autowired
    private KafkaTemplate<String, Object> kafkaTemplate;

    public void createOrder(String orderId, String productId, int quantity) {
        OrderCreatedEvent event = new OrderCreatedEvent(orderId, productId, quantity);
        kafkaTemplate.send("order-created", event);
    }
}
```

---

## 2️⃣ Inventory Service

### Kafka Listener

```java
@Service
public class InventoryService {

    @KafkaListener(topics = "order-created", groupId = "inventory-group")
    public void handleOrderCreated(OrderCreatedEvent event) {
        // Simulate inventory check
        boolean available = true; // Assume always available

        if (available) {
            InventoryReservedEvent reservedEvent = new InventoryReservedEvent(event.getOrderId(), true);
            kafkaTemplate.send("inventory-reserved", reservedEvent);
        } else {
            // Send compensation event if needed
        }
    }

    @Autowired
    private KafkaTemplate<String, Object> kafkaTemplate;
}
```

---

## 3️⃣ Payment Service

```java
@Service
public class PaymentService {

    @KafkaListener(topics = "inventory-reserved", groupId = "payment-group")
    public void handleInventoryReserved(InventoryReservedEvent event) {
        if (event.isSuccess()) {
            System.out.println("Processing payment for order: " + event.getOrderId());
            // Do payment logic here
        }
    }
}
```

---

## 🗃️ Kafka Configuration (All Services)

```java
@Configuration
public class KafkaConfig {
    @Bean
    public ProducerFactory<String, Object> producerFactory() {
        return new DefaultKafkaProducerFactory<>(Map.of(
            ProducerConfig.BOOTSTRAP_SERVERS_CONFIG, "localhost:9092",
            ProducerConfig.KEY_SERIALIZER_CLASS_CONFIG, StringSerializer.class,
            ProducerConfig.VALUE_SERIALIZER_CLASS_CONFIG, JsonSerializer.class
        ));
    }

    @Bean
    public KafkaTemplate<String, Object> kafkaTemplate() {
        return new KafkaTemplate<>(producerFactory());
    }

    @Bean
    public ConsumerFactory<String, Object> consumerFactory() {
        return new DefaultKafkaConsumerFactory<>(Map.of(
            ConsumerConfig.BOOTSTRAP_SERVERS_CONFIG, "localhost:9092",
            ConsumerConfig.KEY_DESERIALIZER_CLASS_CONFIG, StringDeserializer.class,
            ConsumerConfig.VALUE_DESERIALIZER_CLASS_CONFIG, JsonDeserializer.class,
            JsonDeserializer.TRUSTED_PACKAGES, "*"
        ));
    }

    @Bean
    public ConcurrentKafkaListenerContainerFactory<String, Object> kafkaListenerContainerFactory() {
        var factory = new ConcurrentKafkaListenerContainerFactory<String, Object>();
        factory.setConsumerFactory(consumerFactory());
        return factory;
    }
}
```

---

## ▶️ Run Flow

1. Call `orderService.createOrder(...)` in Order Service.
2. It emits an event to Kafka topic `order-created`.
3. Inventory Service listens, reserves stock, emits `inventory-reserved`.
4. Payment Service listens and processes payment.

---


Sure! Let's go deep into the **Saga Pattern**, especially within the context of **microservices**, and explore both **choreography** and **orchestration** techniques. Then, we’ll cover **how to implement these in Java** with practical libraries and tools.

---

## 🧩 What is the Saga Pattern?

The **Saga Pattern** is a **microservices-based transaction management pattern** used to maintain **data consistency** across **distributed services**.

When a single transaction spans multiple services (and databases), you can't use a traditional ACID transaction. So, Sagas break it into a **sequence of local transactions** and define **compensating actions** to undo operations if something fails.

---

## 🔄 Key Concepts:

* **Local transaction**: A transaction that updates a service’s local database.
* **Compensating transaction**: An operation to undo the effects of a previous successful transaction in case of failure.

---

## 🎭 Two Saga Coordination Techniques

### 1. **Choreography** (Decentralized)

* Each service **listens for events** and **produces events**.
* No central controller.
* Services decide what to do based on events from others.

#### 🧠 Flow:

```text
Service A → emits Event A → 
Service B listens → does task B → emits Event B →
Service C listens → does task C → emits Event C → ...
```

#### ✅ Pros:

* Simple and scalable.
* Loosely coupled.

#### ❌ Cons:

* Hard to track and debug.
* Complex business logic is harder to coordinate.

---

### 2. **Orchestration** (Centralized)

* A central **orchestrator service** controls the saga.
* It tells each service what to do next and handles success/failure.

#### 🧠 Flow:

```text
Orchestrator → tells Service A → wait for response →
Orchestrator → tells Service B → wait for response → ...
```

#### ✅ Pros:

* Centralized control.
* Easier to implement complex workflows.

#### ❌ Cons:

* Orchestrator becomes a **central point of failure**.
* Slightly more coupling.

---

## 🛠️ How to Implement Saga Pattern in Java

### Libraries/Tools:

* **Axon Framework**
* **Eventuate Tram Saga** (by Chris Richardson)
* **Camunda / Zeebe** (for orchestration)
* **Apache Kafka / RabbitMQ** (for choreography)
* **Spring Boot** (as a base framework)

---

## ✅ Example 1: Choreography Saga using Spring Boot + Kafka

### Microservices:

* **Order Service**
* **Inventory Service**
* **Payment Service**

### Steps:

1. **Order Service** creates an order and publishes `OrderCreated` event.
2. **Inventory Service** listens, reserves inventory, and emits `InventoryReserved`.
3. **Payment Service** listens, deducts payment, and emits `PaymentCompleted`.
4. On failure, services emit compensation events (`PaymentFailed`, `InventoryRelease`).

### Tools:

* `spring-kafka`
* Event classes + Kafka topics
* Compensation logic inside listeners

---

## ✅ Example 2: Orchestration Saga using Camunda / Zeebe

### Tools:

* Camunda BPM or Zeebe Engine
* Spring Boot + Camunda Starter
* BPMN diagram to define saga flow

### Steps:

1. Define a **BPMN workflow**:

   * Create order → Reserve inventory → Deduct payment
2. Each service exposes **REST endpoints**.
3. Camunda invokes REST tasks and handles failures using compensation tasks.

### Java Code Sketch:

```java
@ExternalTaskSubscription("reserve-inventory")
public class InventoryWorker implements ExternalTaskHandler {
  @Override
  public void execute(ExternalTask task, ExternalTaskService service) {
    try {
      // Call inventory service
      service.complete(task);
    } catch (Exception e) {
      service.handleFailure(task, "Inventory error", e.getMessage(), 0, 1000);
    }
  }
}
```

---

## ⚖️ When to Use What?

| Scenario                       | Use Choreography | Use Orchestration |
| ------------------------------ | ---------------- | ----------------- |
| Simple flow with few services  | ✅ Yes            | Optional          |
| Complex workflows with retries | ❌ Hard to manage | ✅ Easier          |
| Loose coupling is a must       | ✅ Yes            | ❌ Less            |
| Full traceability is required  | ❌ Difficult      | ✅ Central control |
| You use a BPM engine           | ❌ Not needed     | ✅ Use it          |

---

## 🧪 Testing and Challenges

* **Idempotency**: Services should handle duplicate events.
* **Ordering**: Message brokers like Kafka help preserve order.
* **Compensation logic**: Should be well-tested.
* **Observability**: Use distributed tracing (e.g., Zipkin, Jaeger).

---

## Want to Get Started with Java Code?

I can give you:

* A simple working **Spring Boot + Kafka** choreography example.
* Or a **Camunda orchestration sample** with BPMN and REST services.

Let me know which one you’d prefer, and I’ll guide you step by step.


Great! Here's a **simple working example of a Spring Boot + Kafka Choreography-based Saga** involving three services:

### 🧾 Scenario: Order Placement Workflow

Services involved:

1. **Order Service** – Creates an order and emits `OrderCreated` event.
2. **Inventory Service** – Reserves inventory after receiving the event.
3. **Payment Service** – Deducts payment after inventory is reserved.

---

### 🔧 Tech Stack

* **Spring Boot**
* **Apache Kafka**
* **Spring Kafka**
* JSON-based events

---

## 📁 Project Structure

Each service can be a separate Spring Boot project or module:

```
order-service/
inventory-service/
payment-service/
common/ (shared event classes)
```

---

## 🧱 Common: Shared Event Classes

```java
// OrderCreatedEvent.java
public class OrderCreatedEvent {
    private String orderId;
    private String productId;
    private int quantity;
    // Getters, setters, constructor
}

// InventoryReservedEvent.java
public class InventoryReservedEvent {
    private String orderId;
    private boolean success;
    // Getters, setters, constructor
}
```

---

## 1️⃣ Order Service

### application.yml

```yaml
spring:
  kafka:
    bootstrap-servers: localhost:9092
```

### Kafka Producer

```java
@Service
public class OrderService {

    @Autowired
    private KafkaTemplate<String, Object> kafkaTemplate;

    public void createOrder(String orderId, String productId, int quantity) {
        OrderCreatedEvent event = new OrderCreatedEvent(orderId, productId, quantity);
        kafkaTemplate.send("order-created", event);
    }
}
```

---

## 2️⃣ Inventory Service

### Kafka Listener

```java
@Service
public class InventoryService {

    @KafkaListener(topics = "order-created", groupId = "inventory-group")
    public void handleOrderCreated(OrderCreatedEvent event) {
        // Simulate inventory check
        boolean available = true; // Assume always available

        if (available) {
            InventoryReservedEvent reservedEvent = new InventoryReservedEvent(event.getOrderId(), true);
            kafkaTemplate.send("inventory-reserved", reservedEvent);
        } else {
            // Send compensation event if needed
        }
    }

    @Autowired
    private KafkaTemplate<String, Object> kafkaTemplate;
}
```

---

## 3️⃣ Payment Service

```java
@Service
public class PaymentService {

    @KafkaListener(topics = "inventory-reserved", groupId = "payment-group")
    public void handleInventoryReserved(InventoryReservedEvent event) {
        if (event.isSuccess()) {
            System.out.println("Processing payment for order: " + event.getOrderId());
            // Do payment logic here
        }
    }
}
```

---

## 🗃️ Kafka Configuration (All Services)

```java
@Configuration
public class KafkaConfig {
    @Bean
    public ProducerFactory<String, Object> producerFactory() {
        return new DefaultKafkaProducerFactory<>(Map.of(
            ProducerConfig.BOOTSTRAP_SERVERS_CONFIG, "localhost:9092",
            ProducerConfig.KEY_SERIALIZER_CLASS_CONFIG, StringSerializer.class,
            ProducerConfig.VALUE_SERIALIZER_CLASS_CONFIG, JsonSerializer.class
        ));
    }

    @Bean
    public KafkaTemplate<String, Object> kafkaTemplate() {
        return new KafkaTemplate<>(producerFactory());
    }

    @Bean
    public ConsumerFactory<String, Object> consumerFactory() {
        return new DefaultKafkaConsumerFactory<>(Map.of(
            ConsumerConfig.BOOTSTRAP_SERVERS_CONFIG, "localhost:9092",
            ConsumerConfig.KEY_DESERIALIZER_CLASS_CONFIG, StringDeserializer.class,
            ConsumerConfig.VALUE_DESERIALIZER_CLASS_CONFIG, JsonDeserializer.class,
            JsonDeserializer.TRUSTED_PACKAGES, "*"
        ));
    }

    @Bean
    public ConcurrentKafkaListenerContainerFactory<String, Object> kafkaListenerContainerFactory() {
        var factory = new ConcurrentKafkaListenerContainerFactory<String, Object>();
        factory.setConsumerFactory(consumerFactory());
        return factory;
    }
}
```

---

## ▶️ Run Flow

1. Call `orderService.createOrder(...)` in Order Service.
2. It emits an event to Kafka topic `order-created`.
3. Inventory Service listens, reserves stock, emits `inventory-reserved`.
4. Payment Service listens and processes payment.

---

## 💥 Scenario: Inventory or Payment Fails

We’ll enhance the saga to:

* Handle **inventory failures**.
* Handle **payment failures**.
* Trigger **compensation** (e.g., cancel order, release inventory) via **additional Kafka events**.

---

## 🔁 Updated Flow (With Failure Paths)

```
1. Order Service → emits OrderCreated
2. Inventory Service → tries to reserve
   - Success → emits InventoryReserved
   - Failure → emits InventoryFailed
3. Payment Service → tries to process
   - Success → emits PaymentCompleted
   - Failure → emits PaymentFailed
4. Order Service → listens for success/failure events → mark order status accordingly
5. Inventory Service → listens for PaymentFailed → releases inventory
```

---

## 🔨 New Event Classes

```java
public class InventoryFailedEvent {
    private String orderId;
    private String reason;
    // constructor, getters
}

public class PaymentFailedEvent {
    private String orderId;
    private String reason;
    // constructor, getters
}

public class PaymentCompletedEvent {
    private String orderId;
    // constructor, getters
}
```

---

## 🔁 Inventory Service (Compensating for Payment Failure)

### 1. Handle Inventory Reservation

```java
@KafkaListener(topics = "order-created")
public void handleOrderCreated(OrderCreatedEvent event) {
    boolean success = inventoryAvailable(event.getProductId(), event.getQuantity());

    if (success) {
        kafkaTemplate.send("inventory-reserved", new InventoryReservedEvent(event.getOrderId(), true));
    } else {
        kafkaTemplate.send("inventory-failed", new InventoryFailedEvent(event.getOrderId(), "Out of stock"));
    }
}
```

### 2. Compensate on Payment Failure

```java
@KafkaListener(topics = "payment-failed")
public void handlePaymentFailed(PaymentFailedEvent event) {
    System.out.println("Releasing inventory for order " + event.getOrderId());
    // Add logic to rollback inventory
}
```

---

## 💳 Payment Service (Compensating Logic)

```java
@KafkaListener(topics = "inventory-reserved")
public void handleInventoryReserved(InventoryReservedEvent event) {
    if (!event.isSuccess()) return;

    boolean paymentSuccess = processPayment(event.getOrderId());

    if (paymentSuccess) {
        kafkaTemplate.send("payment-completed", new PaymentCompletedEvent(event.getOrderId()));
    } else {
        kafkaTemplate.send("payment-failed", new PaymentFailedEvent(event.getOrderId(), "Card declined"));
    }
}
```

---

## 📦 Order Service (Final Status Handling)

```java
@KafkaListener(topics = {"inventory-failed", "payment-failed", "payment-completed"})
public void handleAll(Object event) {
    if (event instanceof InventoryFailedEvent e) {
        System.out.println("Order " + e.getOrderId() + " failed due to inventory: " + e.getReason());
    } else if (event instanceof PaymentFailedEvent e) {
        System.out.println("Order " + e.getOrderId() + " failed due to payment: " + e.getReason());
    } else if (event instanceof PaymentCompletedEvent e) {
        System.out.println("Order " + e.getOrderId() + " completed successfully!");
    }
}
```

---

## 🛠️ Best Practices for Compensation

* All **compensating actions must be idempotent**.
* Maintain **saga state** in a database if needed (for observability).
* Use **dedicated topics** for compensating events (`inventory-release`, `order-cancelled`, etc.).
* Add **DLQ (Dead Letter Queues)** to capture failed events.

---

## ✅ Final Kafka Topics Used

| Topic                | Purpose                        |
| -------------------- | ------------------------------ |
| `order-created`      | Trigger saga                   |
| `inventory-reserved` | Forward on success             |
| `inventory-failed`   | Compensation on inventory fail |
| `payment-completed`  | Saga success signal            |
| `payment-failed`     | Trigger compensation           |

---

## **BPMN** stands for **Business Process Model and Notation**.

It's a **graphical language** used to model and describe **business workflows** clearly and visually. Think of it like a **flowchart**, but specifically designed for **business processes**.

---

## 🔍 What BPMN Looks Like

Here’s a simple visual breakdown:

```
[Start] --> [Task A] --> [Task B] --> [End]
```

Each box and arrow has meaning and can be interpreted by both humans (business analysts) and machines (workflow engines like Camunda).

---

## 🧱 Core BPMN Elements

| Element           | Symbol                | Meaning                                  |
| ----------------- | --------------------- | ---------------------------------------- |
| **Start Event**   | ○ (Circle)            | Where the process begins                 |
| **Task**          | ▭ (Rounded rectangle) | An action or step (manual or automated)  |
| **Sequence Flow** | → (Arrow)             | Shows order of tasks                     |
| **End Event**     | ◎ (Bold circle)       | Process ends                             |
| **Gateway**       | ⊕ / ⨂ (Diamond)       | Decision point (if/else, parallel, etc.) |
| **Subprocess**    | ▭ with +              | Collapsible reusable logic               |
| **Message/Event** | ✉️ or ⚠️ etc.         | Signals between processes                |

---

## 🎯 Why BPMN?

* ✅ **Visual clarity** – Non-tech and tech people can understand it.
* ✅ **Standardized** – It’s globally used and supported.
* ✅ **Executable** – Engines like **Camunda**, **Flowable**, and **Activiti** can **run BPMN diagrams**.
* ✅ **Flexible** – Supports simple to complex workflows including loops, decisions, compensation, error handling, and parallel steps.

---

## 🛠️ BPMN + Camunda

* You **draw BPMN diagrams** using **Camunda Modeler**.
* You **deploy** the diagram into a **Camunda engine**.
* Camunda **runs the process**, calls your REST services, and tracks progress.
* It acts as the **brain** of your business process orchestration.

---

## 📌 Example Use Case

### 💡 Order Saga

BPMN for an order flow:

```
[Start] → [Create Order] → [Reserve Inventory] → [Charge Payment] → [End]

                    ↘ (error) → [Release Inventory] → [Cancel Order] → [End]
```

---

## 🧠 Summary

| BPMN Is...              | BPMN Is Not...            |
| ----------------------- | ------------------------- |
| A standard for modeling | A programming language    |
| Executable with engines | Only for drawing diagrams |
| Ideal for microservices | Limited to simple flows   |

---

## **Camunda-based Saga Orchestration** example using **BPMN** and **REST-based microservices**.

---

## 📌 Scenario: Order Fulfillment (Same Business Flow)

Services:

* **Order Service**
* **Inventory Service**
* **Payment Service**

Camunda will:

* Orchestrate the flow using a **BPMN diagram**
* Call REST endpoints of services
* Handle errors via **boundary events + compensation**

---

## 🧱 Architecture Overview

```
Client → Camunda Orchestrator → (REST) Order / Inventory / Payment microservices
```

---

## 🗂️ Project Structure

* `camunda-orchestrator` – Spring Boot + Camunda
* `order-service` – Spring Boot REST
* `inventory-service` – Spring Boot REST
* `payment-service` – Spring Boot REST

---

## 🧭 BPMN Diagram (Simplified)

Here's a visual description:

```
Start → [Order Service] → [Inventory Service] → [Payment Service] → End

On any failure → Compensation (Cancel Order / Release Inventory)
```

### 👉 BPMN XML Snippet (Camunda Modeler)

```xml
<bpmn:process id="orderSaga" isExecutable="true">
  <bpmn:startEvent id="start" name="Start"/>
  
  <bpmn:sequenceFlow sourceRef="start" targetRef="orderTask"/>
  
  <bpmn:serviceTask id="orderTask" name="Create Order" camunda:type="external" camunda:topic="create-order"/>
  <bpmn:serviceTask id="inventoryTask" name="Reserve Inventory" camunda:type="external" camunda:topic="reserve-inventory"/>
  <bpmn:serviceTask id="paymentTask" name="Process Payment" camunda:type="external" camunda:topic="process-payment"/>
  
  <bpmn:endEvent id="end" name="End"/>
  
  <bpmn:sequenceFlow sourceRef="orderTask" targetRef="inventoryTask"/>
  <bpmn:sequenceFlow sourceRef="inventoryTask" targetRef="paymentTask"/>
  <bpmn:sequenceFlow sourceRef="paymentTask" targetRef="end"/>
  
  <!-- Compensation handlers -->
  <bpmn:boundaryEvent id="paymentFailed" attachedToRef="paymentTask">
    <bpmn:errorEventDefinition errorRef="PaymentError"/>
  </bpmn:boundaryEvent>
  
  <bpmn:serviceTask id="releaseInventoryTask" name="Release Inventory" camunda:type="external" camunda:topic="release-inventory"/>
  <bpmn:sequenceFlow sourceRef="paymentFailed" targetRef="releaseInventoryTask"/>
  <bpmn:sequenceFlow sourceRef="releaseInventoryTask" targetRef="end"/>
</bpmn:process>
```

---

## 🧠 Camunda Orchestrator Code (Spring Boot)

### Maven Dependencies

```xml
<dependency>
  <groupId>org.camunda.bpm.springboot</groupId>
  <artifactId>camunda-bpm-spring-boot-starter</artifactId>
</dependency>
```

---

### Java External Task Workers

```java
@Component
public class OrderWorker implements ExternalTaskHandler {
    @Override
    public void execute(ExternalTask task, ExternalTaskService service) {
        String orderId = UUID.randomUUID().toString();
        // Call Order Service via REST
        restTemplate.postForObject("http://localhost:8081/orders", new Order(orderId), Void.class);
        service.complete(task, Map.of("orderId", orderId));
    }
}
```

```java
@Component
public class InventoryWorker implements ExternalTaskHandler {
    @Override
    public void execute(ExternalTask task, ExternalTaskService service) {
        String orderId = task.getVariable("orderId");
        try {
            restTemplate.postForObject("http://localhost:8082/inventory/reserve", Map.of("orderId", orderId), Void.class);
            service.complete(task);
        } catch (HttpClientErrorException e) {
            throw new BpmnError("InventoryError", "Inventory failed");
        }
    }
}
```

```java
@Component
public class PaymentWorker implements ExternalTaskHandler {
    @Override
    public void execute(ExternalTask task, ExternalTaskService service) {
        String orderId = task.getVariable("orderId");
        try {
            restTemplate.postForObject("http://localhost:8083/payment", Map.of("orderId", orderId), Void.class);
            service.complete(task);
        } catch (HttpClientErrorException e) {
            throw new BpmnError("PaymentError", "Payment failed");
        }
    }
}
```

```java
@Component
public class ReleaseInventoryWorker implements ExternalTaskHandler {
    @Override
    public void execute(ExternalTask task, ExternalTaskService service) {
        String orderId = task.getVariable("orderId");
        restTemplate.postForObject("http://localhost:8082/inventory/release", Map.of("orderId", orderId), Void.class);
        service.complete(task);
    }
}
```

---

### Worker Registration

```java
@Bean
public ExternalTaskClient client() {
    return ExternalTaskClient.create()
        .baseUrl("http://localhost:8080/engine-rest")
        .asyncResponseTimeout(10000)
        .build();
}
```

Register topics with the client:

```java
@PostConstruct
public void registerWorkers() {
    client.subscribe("create-order").handler(orderWorker).open();
    client.subscribe("reserve-inventory").handler(inventoryWorker).open();
    client.subscribe("process-payment").handler(paymentWorker).open();
    client.subscribe("release-inventory").handler(releaseInventoryWorker).open();
}
```

---

## 🌐 Microservices REST Endpoints (Sample)

### Order Service

```java
@PostMapping("/orders")
public ResponseEntity<?> createOrder(@RequestBody Order order) {
    // Save order to DB
    return ResponseEntity.ok().build();
}
```

### Inventory Service

```java
@PostMapping("/inventory/reserve")
public ResponseEntity<?> reserve(@RequestBody Map<String, String> body) {
    // Check and reserve inventory
    return ResponseEntity.ok().build();
}

@PostMapping("/inventory/release")
public ResponseEntity<?> release(@RequestBody Map<String, String> body) {
    // Release inventory
    return ResponseEntity.ok().build();
}
```

### Payment Service

```java
@PostMapping("/payment")
public ResponseEntity<?> pay(@RequestBody Map<String, String> body) {
    // Simulate payment logic (can fail randomly)
    if (Math.random() < 0.5) {
        throw new ResponseStatusException(HttpStatus.BAD_REQUEST, "Payment failed");
    }
    return ResponseEntity.ok().build();
}
```

---

## ✅ Summary

| Component             | Role                                                |
| --------------------- | --------------------------------------------------- |
| Camunda BPMN          | Defines workflow and orchestration logic            |
| External Task Workers | Java code to invoke REST microservices              |
| REST Services         | Business logic units (Order, Inventory, Payment)    |
| Compensation          | Modeled via BPMN boundary events + additional tasks |
