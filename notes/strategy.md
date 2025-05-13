## **Strategy Pattern**

Defines a family of algorithms, encapsulates each one, and makes them interchangeable.

---

## ✅ **Strategy Design Pattern – Complete Guide**

---

### 💡 What is the Strategy Pattern?

> The Strategy Pattern allows you to define a **family of algorithms** (strategies), encapsulate each one, and make them **interchangeable** at runtime.
> It helps in **choosing a behavior at runtime** without changing the client code.

---

### 🧩 Example

```java
// Strategy interface
interface PaymentStrategy {
    void pay(int amount);
}

// Concrete strategies
class CreditCardPayment implements PaymentStrategy {
    public void pay(int amount) {
        System.out.println("Paid " + amount + " via Credit Card");
    }
}

class PayPalPayment implements PaymentStrategy {
    public void pay(int amount) {
        System.out.println("Paid " + amount + " via PayPal");
    }
}

// Context class
class ShoppingCart {
    private PaymentStrategy paymentStrategy;

    public ShoppingCart(PaymentStrategy strategy) {
        this.paymentStrategy = strategy;
    }

    public void setPaymentStrategy(PaymentStrategy strategy) {
        this.paymentStrategy = strategy;
    }

    public void checkout(int amount) {
        if (paymentStrategy == null) {
            throw new IllegalStateException("Payment strategy not set");
        }
        paymentStrategy.pay(amount);
    }
}

// Client
public class Main {
    public static void main(String[] args) {
        ShoppingCart cart = new ShoppingCart(new CreditCardPayment());
        cart.checkout(100);

        // Change strategy at runtime
        cart.setPaymentStrategy(new PayPalPayment());
        cart.checkout(200);
    }
}
```

---

## 🎯 Interview Questions & Answers

---

### 🔍 **1. What is the Strategy Pattern?**

> It’s a behavioral design pattern used to define a family of algorithms and allow their interchangeability at runtime.
> Example: You can switch between payment methods without changing the `ShoppingCart` logic.

---

### 📦 **2. What are the key components in your example?**

* `PaymentStrategy`: Strategy interface
* `CreditCardPayment`, `PayPalPayment`: Concrete strategies
* `ShoppingCart`: Context using the strategy
* `Main`: Client that selects the strategy

---

### 🔁 **3. Why use Strategy Pattern?**

> * Removes conditional `if-else` or `switch` blocks
> * Makes the system **extensible** (Open/Closed Principle)
> * Promotes **code reuse** and clean separation of concerns

---

### ⚖️ **4. Pros and Cons of Strategy Pattern?**

✅ **Pros:**

* Cleaner code and easier testing
* Can switch behavior at runtime
* Easy to add new strategies

❌ **Cons:**

* Increases number of classes
* Client needs to know all strategies

---

### 🔀 **5. Real-world use cases?**

> * Payment gateways (as in your example)
> * Compression algorithms (zip, rar, tar)
> * Sorting strategies (bubble, quick, merge)
> * Authentication mechanisms (OAuth, JWT, LDAP)

---

### 🔒 **6. How does it follow SOLID principles?**

| Principle | Explanation                                                 |
| --------- | ----------------------------------------------------------- |
| **OCP**   | Add new strategy without changing existing code             |
| **SRP**   | Each strategy has its own single responsibility             |
| **DIP**   | `ShoppingCart` depends on the abstraction `PaymentStrategy` |

---

### 🧠 **7. How is Strategy Pattern different from State Pattern?**

| Aspect       | Strategy Pattern            | State Pattern                        |
| ------------ | --------------------------- | ------------------------------------ |
| Purpose      | Choose algorithm at runtime | Handle state transitions dynamically |
| Who changes? | Client selects the strategy | Object manages state internally      |
| Focus        | Behavior selection          | State transitions                    |

---

### 🚀 **8. Can this pattern be replaced with lambdas in Java 8+?**

> Yes! Since `PaymentStrategy` is a **functional interface**, you can use lambdas:

```java
ShoppingCart cart = new ShoppingCart(amount -> System.out.println("Paid " + amount + " via UPI"));
cart.checkout(300);
```

---

## ✅ Strategy Pattern in Spring Boot – Real-world Example

---

### 💡 Use Case: Dynamic Payment Strategy in Spring Boot

We want to allow users to pay via different methods: **Credit Card**, **PayPal**, or **UPI** — and Spring should pick the right payment strategy at runtime based on a value like `"CREDIT_CARD"` or `"PAYPAL"`.

---

### 🛠️ Step-by-Step Implementation

---

### **1. Create the Strategy Interface**

```java
public interface PaymentStrategy {
    void pay(int amount);
    String getType(); // to identify strategy
}
```

---

### **2. Implement Concrete Strategies**

```java
import org.springframework.stereotype.Component;

@Component
public class CreditCardPayment implements PaymentStrategy {
    @Override
    public void pay(int amount) {
        System.out.println("Paid " + amount + " via Credit Card");
    }

    @Override
    public String getType() {
        return "CREDIT_CARD";
    }
}

@Component
public class PayPalPayment implements PaymentStrategy {
    @Override
    public void pay(int amount) {
        System.out.println("Paid " + amount + " via PayPal");
    }

    @Override
    public String getType() {
        return "PAYPAL";
    }
}

@Component
public class UpiPayment implements PaymentStrategy {
    @Override
    public void pay(int amount) {
        System.out.println("Paid " + amount + " via UPI");
    }

    @Override
    public String getType() {
        return "UPI";
    }
}
```

---

### **3. Strategy Resolver (Dynamic Selector)**

```java
import org.springframework.beans.factory.annotation.Autowired;
import org.springframework.stereotype.Component;

import java.util.List;

@Component
public class PaymentStrategyFactory {

    private final List<PaymentStrategy> strategies;

    @Autowired
    public PaymentStrategyFactory(List<PaymentStrategy> strategies) {
        this.strategies = strategies;
    }

    public PaymentStrategy getStrategy(String type) {
        return strategies.stream()
                .filter(s -> s.getType().equalsIgnoreCase(type))
                .findFirst()
                .orElseThrow(() -> new IllegalArgumentException("Unknown payment type: " + type));
    }
}
```

---

### **4. ShoppingCart Using Strategy Factory**

```java
import org.springframework.beans.factory.annotation.Autowired;
import org.springframework.stereotype.Service;

@Service
public class ShoppingCart {

    private final PaymentStrategyFactory factory;

    @Autowired
    public ShoppingCart(PaymentStrategyFactory factory) {
        this.factory = factory;
    }

    public void checkout(String paymentType, int amount) {
        PaymentStrategy strategy = factory.getStrategy(paymentType);
        strategy.pay(amount);
    }
}
```

---

### **5. REST Controller to Trigger Checkout**

```java
import org.springframework.beans.factory.annotation.Autowired;
import org.springframework.web.bind.annotation.*;

@RestController
@RequestMapping("/cart")
public class ShoppingCartController {

    @Autowired
    private ShoppingCart shoppingCart;

    @PostMapping("/checkout")
    public String checkout(@RequestParam String paymentType, @RequestParam int amount) {
        shoppingCart.checkout(paymentType, amount);
        return "Checkout complete using " + paymentType;
    }
}
```

---

### ✅ Sample API Call

```
POST /cart/checkout?paymentType=PAYPAL&amount=300
```

**Output:**

```
Paid 300 via PayPal
```

---

## 🎯 Advantages of This Spring Strategy Setup

* 🧩 Loose coupling between client and strategy
* 🔄 Easily add new strategies by adding `@Component` classes
* ✅ Follows **Open/Closed Principle**
* 🛠️ No manual `if-else` or `switch` logic
* 🧪 Easy to unit test strategies individually
