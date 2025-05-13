## Singleton Pattern

Ensures only one instance of a class is created and provides global access to it.

```java
public class Singleton {
    private static Singleton instance;

    private Singleton() {} // private constructor

    public static synchronized Singleton getInstance() {
        if (instance == null) {
            instance = new Singleton();
        }
        return instance;
    }
}
```

---

Absolutely! Here's a clear and concise answer to each of the cross-questions listed earlier, based on the Singleton class you provided:

---

### 🔁 **1. What is Singleton Design Pattern and Why Do We Use It?**

> **Answer:**
> The Singleton pattern ensures that a class has only **one instance** throughout the application and provides a **global access point** to it. It’s useful when exactly one object is needed to coordinate actions across a system—for example, configuration managers, logging, or database connections.

---

### 🔒 **2. Why is the Constructor Private?**

> **Answer:**
> The constructor is marked `private` to **prevent external instantiation** of the class. This ensures that no other class can create a new instance, enforcing the Singleton constraint.

---

### 🧵 **3. Is Your Code Thread-Safe?**

> **Answer:**
> Yes, this implementation is **thread-safe** because `getInstance()` is marked as `synchronized`, so only one thread can execute it at a time.
> However, this approach has a **performance drawback**—after the instance is initialized, threads still acquire the lock, which is unnecessary.

---

### 🐌 **4. Can We Improve the Performance of This Singleton?**

> **Answer:**
> Yes. We can use **Double-Checked Locking** to reduce synchronization overhead:

```java
public class Singleton {
    private static volatile Singleton instance;

    private Singleton() {}

    public static Singleton getInstance() {
        if (instance == null) {
            synchronized (Singleton.class) {
                if (instance == null) {
                    instance = new Singleton();
                }
            }
        }
        return instance;
    }
}
```

This avoids locking once the instance is initialized.

---

### 📦 **5. What Are Other Ways to Implement Singleton?**

> **Answer:**

* **Eager Initialization:**

  ```java
  public class Singleton {
      private static final Singleton instance = new Singleton();
      private Singleton() {}
      public static Singleton getInstance() {
          return instance;
      }
  }
  ```

  ✅ Simple, thread-safe. ❌ Instance is created even if not used.

* **Static Inner Class (Bill Pugh):**

  ```java
  public class Singleton {
      private Singleton() {}
      private static class Holder {
          private static final Singleton instance = new Singleton();
      }
      public static Singleton getInstance() {
          return Holder.instance;
      }
  }
  ```

  ✅ Lazy-loaded, thread-safe, no synchronization overhead.

* **Enum-based Singleton:**

  ```java
  public enum Singleton {
      INSTANCE;
  }
  ```

  ✅ Simplest and safest (prevents serialization, reflection attacks).

---

### 🔍 **6. Can Singleton Be Broken? How Do You Prevent That?**

> **Answer:**

* **Reflection** can bypass the private constructor:

  ```java
  Constructor<Singleton> constructor = Singleton.class.getDeclaredConstructor();
  constructor.setAccessible(true);
  Singleton obj = constructor.newInstance();
  ```

  ✅ **Fix:** Throw exception inside the constructor if `instance != null`.

* **Serialization** can break it by creating a new instance on deserialization.
  ✅ **Fix:** Add `readResolve()` method:

  ```java
  protected Object readResolve() {
      return getInstance();
  }
  ```

---

### 🧪 **7. How Would You Test a Singleton Class?**

> **Answer:**

* Use assertions to verify that multiple calls to `getInstance()` return the **same reference**.
* Mocking is hard with Singletons—often leads to tight coupling.
* Use dependency injection for better testability if possible.

---

### 🧠 **8. What Are the Drawbacks of Singleton Pattern?**

> **Answer:**

* **Global state** can lead to unpredictable behavior and bugs.
* **Tight coupling** makes it hard to replace or mock in tests.
* Violates **Single Responsibility Principle** (controls instantiation + business logic).
* Can make **unit testing** and **parallelism** difficult.

---

### ☁️ **9. Where Is Singleton Commonly Used in Java?**

> **Answer:**

* `java.lang.Runtime.getRuntime()`
* `java.awt.Desktop.getDesktop()`
* Logging frameworks like Log4j or SLF4J
* Spring Beans (default scope is Singleton)
* Configuration or Cache managers

---
