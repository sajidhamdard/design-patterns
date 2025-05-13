## ✅ What is the **Decorator Pattern**?

* The **Decorator Pattern** is a **structural design pattern** that allows behavior to be added to an individual object, dynamically, **without modifying the class itself**.
* It follows the **Open/Closed Principle**: open for extension, but closed for modification.
* It wraps the original object and provides additional functionalities.

---

### 🧠 Example Code

```java
interface Car {
    void assemble();
}

class BasicCar implements Car {
    public void assemble() {
        System.out.print("Basic Car");
    }
}

class SportsCarDecorator implements Car {
    private Car car;

    public SportsCarDecorator(Car c) {
        this.car = c;
    }

    public void assemble() {
        car.assemble(); // delegate to original
        System.out.print(" + Sports features");
    }
}
```

### ✅ Usage:

```java
public class DecoratorDemo {
    public static void main(String[] args) {
        Car sportsCar = new SportsCarDecorator(new BasicCar());
        sportsCar.assemble();
    }
}
```

**Output:**

```
Basic Car + Sports features
```

You can stack decorators like this:

```java
Car luxurySportsCar = new LuxuryCarDecorator(new SportsCarDecorator(new BasicCar()));
```

---

### ❓ Common Interview Questions and Sample Answers

---

#### 1. **What is the Decorator Pattern?**

**Answer:**

> The Decorator Pattern allows us to dynamically add new behavior or responsibilities to objects without modifying their structure. It uses composition to wrap the original object and adds functionality before or after delegating calls to the wrapped object.

---

#### 2. **How is the Decorator Pattern different from Inheritance?**

**Answer:**

> Inheritance adds behavior at compile-time by extending classes, whereas the Decorator Pattern adds behavior at runtime by wrapping objects. Decorator is more flexible because it allows combinations of behaviors without creating many subclasses.

---

#### 3. **Where is Decorator Pattern used in real life?**

**Answer:**

* Java I/O streams (`BufferedReader`, `InputStreamReader`)
* UI frameworks: wrapping UI components to add borders, scroll bars, etc.
* Logging frameworks to add timestamps, log levels, etc.

**Example in Java:**

```java
BufferedReader reader = new BufferedReader(new InputStreamReader(System.in));
```

---

#### 4. **Which SOLID principle does the Decorator Pattern support?**

**Answer:**

> The Decorator Pattern supports the **Open/Closed Principle** — we can extend the functionality of an object without modifying its source code.

---

#### 5. **What are the advantages of using the Decorator Pattern?**

**Answer:**

* Promotes **code reuse** through composition.
* Adds responsibilities **dynamically at runtime**.
* Avoids large inheritance hierarchies.
* Follows **Open/Closed Principle**.

---

#### 6. **Can you stack multiple decorators together?**

**Answer:**

> Yes. Decorators can be nested to add multiple behaviors.
> For example:

```java
Car decoratedCar = new SportsCarDecorator(new LuxuryCarDecorator(new BasicCar()));
decoratedCar.assemble();
```

---

#### 7. **What are the drawbacks of the Decorator Pattern?**

**Answer:**

* Can result in many small classes.
* Harder to debug due to multiple wrappers.
* Order of decorators affects the final behavior, which can lead to subtle bugs.

---

### 🔁 Optional: Add more decorators

```java
class LuxuryCarDecorator implements Car {
    private Car car;
    public LuxuryCarDecorator(Car c) { this.car = c; }

    public void assemble() {
        car.assemble();
        System.out.print(" + Luxury features");
    }
}
```

---

### ✅ Summary

| Aspect             | Details                                          |
| ------------------ | ------------------------------------------------ |
| Pattern Type       | Structural                                       |
| Use Case           | Add new features without modifying existing code |
| Principle Followed | Open/Closed Principle                            |
| Real-life Example  | Java I/O classes                                 |
| Java Feature Used  | Composition & Interfaces                         |
