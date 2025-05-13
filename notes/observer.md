## **Observer Design Pattern**

---

Defines a one-to-many dependency so when one object changes state, all its dependents are notified.

## ✅ Complete Working Example – **Observer Pattern**

```java
import java.util.ArrayList;
import java.util.List;

// Observer Interface
interface Observer {
    void update(String message);
}

// Concrete Observer
class EmailObserver implements Observer {
    public void update(String message) {
        System.out.println("Email received: " + message);
    }
}

class SMSObserver implements Observer {
    public void update(String message) {
        System.out.println("SMS received: " + message);
    }
}

// Subject (Observable)
class NotificationService {
    private List<Observer> observers = new ArrayList<>();
    
    public void subscribe(Observer o) {
        observers.add(o);
    }

    public void unsubscribe(Observer o) {
        observers.remove(o);
    }

    public void notifyAllObservers(String msg) {
        for (Observer o : observers) {
            o.update(msg);
        }
    }
}

// Client
public class Main {
    public static void main(String[] args) {
        NotificationService service = new NotificationService();
        
        Observer email = new EmailObserver();
        Observer sms = new SMSObserver();
        
        service.subscribe(email);
        service.subscribe(sms);

        service.notifyAllObservers("New update available!");
        
        service.unsubscribe(sms);
        service.notifyAllObservers("Second update.");
    }
}
```

---

## 🎯 Interview Questions & Answers on Observer Pattern

---

### 🔍 **1. What is the Observer Design Pattern?**

> **Answer:**
> The Observer Pattern defines a **one-to-many dependency** between objects so that when one object (the subject) changes state, **all its dependents (observers)** are notified and updated automatically.

---

### 📦 **2. Where is it used in real life or in Java?**

> **Answer:**

* **Real life:** YouTube subscriptions, WhatsApp group notifications.
* **Java:**

  * `java.util.Observer` (deprecated now)
  * Event handling in GUI frameworks like Swing or JavaFX
  * Listeners in the **Spring Framework** (`@EventListener`)

---

### 🔁 **3. Difference between Observer and Pub-Sub?**

| Aspect         | Observer Pattern                    | Pub-Sub Pattern                        |
| -------------- | ----------------------------------- | -------------------------------------- |
| Tight coupling | Subject holds reference to observer | Publisher and Subscriber are decoupled |
| Communication  | Direct method call (`update()`)     | Usually via message broker/event bus   |
| Sync/Async     | Typically synchronous               | Can be asynchronous                    |

---

### 🔄 **4. What are Subject and Observer in your code?**

> **Answer:**

* **Subject:** `NotificationService`
* **Observers:** `EmailObserver`, `SMSObserver`
  The subject notifies observers using `notifyAllObservers()` when an event (message) occurs.

---

### ⚖️ **5. What are the pros and cons of Observer Pattern?**

> ✅ **Pros:**

* Loose coupling between subject and observers
* Easy to add/remove observers
* Automatic update propagation

> ❌ **Cons:**

* Notification order not guaranteed
* Can cause memory leaks if observers aren’t unsubscribed
* Complex when there are many observers

---

### 🧠 **6. How does Observer Pattern follow SOLID principles?**

| Principle | How it’s followed                                                |
| --------- | ---------------------------------------------------------------- |
| SRP       | Subject handles state & notification, Observer handles action    |
| OCP       | New observers can be added without changing subject              |
| DIP       | Subject depends on abstraction (`Observer`) not concrete classes |

---

### 🧪 **7. Can you implement a real-world use case using this pattern?**

> **Answer:**
> Sure. For example, in an **e-commerce system**, when an order is placed:

* Email system, inventory system, and SMS system can **observe** the order service.
* When a new order is placed, they’re all notified automatically.

---

### 🔒 **8. Is Observer Pattern thread-safe in your code?**

> **Answer:**
> No, not yet. If `subscribe()`, `unsubscribe()`, and `notifyAllObservers()` are called from multiple threads, the list should be synchronized using `CopyOnWriteArrayList` or synchronization blocks.
