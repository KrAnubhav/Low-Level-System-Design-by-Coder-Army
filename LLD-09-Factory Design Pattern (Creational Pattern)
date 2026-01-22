# Factory Design Pattern (Creational Pattern)

## Introduction

The **Factory Design Pattern** is one of the most frequently used creational design patterns in software development. It appears in approximately **99% of real-world applications** and is crucial for both interviews and production code.

**Key Topics:**

- What is Factory Design Pattern and why we need it
- Separation of Object Creation Logic from Business Logic
- Three types: Simple Factory, Factory Method, Abstract Factory
- Dependency Injection and Decoupling concepts
- Implementation with Burger Shop example

---

## Motivation: Why Factory Pattern?

### The Problem

**Recall: Strategy Design Pattern**
In the previous lecture on Strategy Pattern with the Robot example, we assumed that behavior objects (Talkable, Walkable, Flyable) were **already created** somewhere. We focused on **using** these objects, not **creating** them.

**PlainText Flow:**

```
Robot Class:
    - Has Talkable t (behavior)
    - Has Walkable w (behavior)
    - Has Flyable f (behavior)

We assumed:
    t = new NormalTalk()  // Already created somewhere
    w = new NormalWalk()  // Already created somewhere
    f = new NoFly()       // Already created somewhere

Then Robot delegates:
    talk() → t.talk()
    walk() → w.walk()
    fly()  → f.fly()
```

**The Missing Piece:** Where and how do we **actually create** these objects using the `new` keyword?

### Object Creation vs Business Logic

**Two Types of Logic in Applications:**

1. **Business Logic**
    - Core functionality of the application
    - Example (Notification System):
        - Which notifications to send?
        - To whom to send?
        - Email notification vs Push notification logic
        - When to trigger notifications?
2. **Object Creation Logic**
    - When to create which object?
    - Which concrete class to instantiate?
    - Conditional logic: If X condition, create Object A; If Y condition, create Object B

**Problem: Mixing Both = Complex Code**

```java
// BAD: Mixed business logic and object creation
public void sendNotification(String type, User user) {
    // Object creation logic mixed with business logic
    Notification notif;
    if (type.equals("email")) {
        notif = new EmailNotification();  // Creation
    } else if (type.equals("push")) {
        notif = new PushNotification();   // Creation
    }

    // Business logic
    notif.send(user);
    logNotification(notif);
    updateDatabase(user, notif);
}
```

**Issues:**

- Hard to read and understand
- Violates Single Responsibility Principle
- Client code knows too much about object creation
- Tight coupling between client and concrete classes

---

## The Factory Solution

### Core Idea

> “Separate object creation logic from business logic by delegating object creation to a dedicated Factory class”
> 

**Benefits:**

1. **Decoupling:** Client doesn’t know HOW objects are created
2. **Simplicity:** Client just asks factory: “Give me an object”
3. **Centralized Creation:** All creation logic in one place
4. **Easy Maintenance:** Change creation logic without affecting client

### Factory Pattern Workflow

```
┌─────────┐         requests         ┌─────────────┐
│ Client  │ ───────────────────────→ │   Factory   │
└─────────┘                          └──────┬──────┘
     ↑                                      │
     │                                      │ creates
     │ returns object                       ↓
     └──────────────────────────────── ┌──────────┐
                                       │  Object  │
                                       └──────────┘
```

**Key Point:** Factory handles the complexity of object creation; client remains simple and focused on business logic.

---

## Types of Factory Patterns

There are **three types** of Factory patterns:

1. **Simple Factory** - Design principle (not official pattern)
2. **Factory Method** - Official design pattern
3. **Abstract Factory** - Official design pattern (most complex)

**Learning Path:** Master Simple Factory → Factory Method becomes easy → Abstract Factory is extension

---

## Simple Factory Pattern

### Definition

**Simple Factory** is a **design principle** (not an official Gang of Four pattern), but it forms the foundation for understanding Factory Method and Abstract Factory.

**Concept:** A separate class with a method that encapsulates object creation logic based on input parameters.

---

## Example: Burger Shop Application

### Problem Statement

Design a system for a burger shop that:

- Sells multiple types of burgers
- Each burger type has different preparation method
- Client (customer) requests burger by type
- System should create appropriate burger object

### Step 1: Product Hierarchy

**Base Class: Burger**

```java
abstract class Burger {
    public abstract void prepare();  // Each burger prepares differently
}
```

**Concrete Products:**

```java
class BasicBurger extends Burger {
    @Override
    public void prepare() {
        System.out.println("Preparing Basic Burger with bun and sauce");
    }
}

class StandardBurger extends Burger {
    @Override
    public void prepare() {
        System.out.println("Preparing Standard Burger with one patty, cheese, and lettuce");
    }
}

class PremiumBurger extends Burger {
    @Override
    public void prepare() {
        System.out.println("Preparing Premium Burger - Gourmet style with special ingredients");
    }
}
```

**Hierarchy Diagram:**

```
        ┌──────────┐
        │  Burger  │ (Abstract)
        └────┬─────┘
             △ extends
        ┌────┴─────────┬──────────┐
        │              │          │
┌───────┴────┐  ┌──────┴─────┐  ┌┴──────────┐
│BasicBurger │  │StandardBurger│ │PremiumBurger│
└────────────┘  └────────────┘  └────────────┘
```

### Step 2: The Factory Class

**BurgerFactory: Responsible for Object Creation**

```java
class BurgerFactory {

    // Factory Method: Takes type, returns appropriate object
    public Burger createBurger(String type) {

        if (type.equals("BASIC")) {
            return new BasicBurger();

        } else if (type.equals("STANDARD")) {
            return new StandardBurger();

        } else if (type.equals("PREMIUM")) {
            return new PremiumBurger();

        } else {
            System.out.println("Invalid burger type");
            return null;
        }
    }
}
```

**Key Points:**

- **Single Responsibility:** Factory’s only job is creating burgers
- **Encapsulation:** Creation logic hidden from client
- **Input Parameter:** `type` determines which object to create
- **Return Type:** Returns base class `Burger` (polymorphism)

### Step 3: Client Code

**Client: Uses Factory to Get Objects**

```java
public class Main {
    public static void main(String[] args) {

        // 1. Get user preference (could come from UI, API, etc.)
        String userChoice = "STANDARD";

        // 2. Create factory instance
        BurgerFactory myBurgerFactory = new BurgerFactory();

        // 3. Ask factory for burger object
        Burger burger = myBurgerFactory.createBurger(userChoice);

        // 4. Use the burger (business logic)
        burger.prepare();
    }
}
```

**Output:**

```
Preparing Standard Burger with one patty, cheese, and lettuce
```

---

## Architecture Analysis

### UML Diagram: Simple Factory

```
┌─────────────────┐
│  BurgerFactory  │ (Factory)
├─────────────────┤
│ + createBurger()│ ──────┐ creates & returns
└─────────────────┘       │
                          ↓
                   ┌──────────┐
                   │  Burger  │ (Product)
                   └────┬─────┘
                        △
                   ┌────┴────┬──────────┐
                   │         │          │
            BasicBurger StandardBurger PremiumBurger

┌─────────┐         uses         ┌─────────────┐
│ Client  │ ──────────────────→ │BurgerFactory│
└─────────┘                      └─────────────┘
     │                                  │
     │                                  │
     └──────── uses ──────────────→ ┌──────┐
                                   │Burger│
                                   └──────┘
```

### Relationship: Composition

**BurgerFactory HAS-A Burger**

- Factory returns Burger objects
- Composition relationship (not inheritance)
- Factory “composes” products

---

## Complete Code Implementation

```java
// ============ Product Hierarchy ============
abstract class Burger {
    public abstract void prepare();
}

class BasicBurger extends Burger {
    @Override
    public void prepare() {
        System.out.println("Preparing Basic Burger with bun and sauce");
    }
}

class StandardBurger extends Burger {
    @Override
    public void prepare() {
        System.out.println("Preparing Standard Burger with one patty, cheese, and lettuce");
    }
}

class PremiumBurger extends Burger {
    @Override
    public void prepare() {
        System.out.println("Preparing Premium Burger - Gourmet with special ingredients");
    }
}

// ============ Factory ============
class BurgerFactory {
    public Burger createBurger(String type) {
        if (type.equals("BASIC")) {
            return new BasicBurger();
        } else if (type.equals("STANDARD")) {
            return new StandardBurger();
        } else if (type.equals("PREMIUM")) {
            return new PremiumBurger();
        } else {
            System.out.println("Invalid burger type!");
            return null;
        }
    }
}

// ============ Client ============
public class Main {
    public static void main(String[] args) {
        String userChoice = "STANDARD";

        BurgerFactory myBurgerFactory = new BurgerFactory();
        Burger burger = myBurgerFactory.createBurger(userChoice);

        if (burger != null) {
            burger.prepare();
        }
    }
}
```

**Output:**

```
Preparing Standard Burger with one patty, cheese, and lettuce
```

---

## Dependency Injection Explained

### What is Dependency Injection?

**Definition:** Providing dependencies (objects) to a class from external source instead of creating them internally.

**In Our Examples:**

**Strategy Pattern (Robot Example):**

```java
// Dependencies: Talkable, Walkable, Flyable
class Robot {
    private Talkable talkBehavior;    // Dependency
    private Walkable walkBehavior;    // Dependency
    private Flyable flyBehavior;      // Dependency

    // Constructor Injection
    public Robot(Talkable t, Walkable w, Flyable f) {
        this.talkBehavior = t;   // Injected
        this.walkBehavior = w;   // Injected
        this.flyBehavior = f;    // Injected
    }

    // Robot depends on these to function
    public void talk() {
        talkBehavior.talk();  // Delegates to dependency
    }
}
```

**Key Points:**

- **Dependent:** Robot (depends on behaviors to function)
- **Dependencies:** Talkable, Walkable, Flyable (objects Robot needs)
- **Injection:** Passing dependencies via constructor/setter
- **Abstraction:** Robot doesn’t create dependencies itself

**Benefits:**

1. **Loose Coupling:** Robot doesn’t know concrete implementations
2. **Testability:** Easy to inject mock objects for testing
3. **Flexibility:** Can change behaviors at runtime
4. **Separation:** Creation logic separated from usage logic

---

## Decoupling: The Ultimate Goal

### Why Decouple?

**Throughout LLD (Low Level Design):**

- OOP Principles
- SOLID Principles
- Design Patterns

**All aim for:** Decoupling client from implementation details

### Decoupling in Factory Pattern

**What Client Doesn’t Know:**

- How objects are created
- Which concrete class is instantiated
- Complexity of creation logic
- Conditional logic for creation

**What Client Knows:**

- Factory exists
- Factory has method to get objects
- Object has methods to use

**Result:** Clean separation of concerns

---

## Why Simple Factory is NOT an Official Pattern

### Characteristics of Simple Factory

**Limitations:**

1. **No polymorphism in factory itself** - Single concrete factory class
2. **Violates Open/Closed Principle** - Must modify factory for new types
3. **Not extensible** - Adding new product requires changing factory code

**Example: Adding New Burger**

```java
// Need to MODIFY existing factory (violates OCP)
class BurgerFactory {
    public Burger createBurger(String type) {
        if (type.equals("BASIC")) {
            return new BasicBurger();
        } else if (type.equals("STANDARD")) {
            return new StandardBurger();
        } else if (type.equals("PREMIUM")) {
            return new PremiumBurger();
        }
        // ❌ Must add new else-if for every new burger type!
        else if (type.equals("VEGAN")) {
            return new VeganBurger();
        }
        // Code changes required = Not truly extensible
    }
}
```

**Why It’s a Principle, Not Pattern:**

- Design patterns should follow **Open/Closed Principle**
- Simple Factory requires modification for extension
- It’s a **stepping stone** to true Factory patterns

---

## Advantages of Simple Factory ✅

Despite not being an official pattern, Simple Factory provides significant benefits:

### 1. Centralized Object Creation

All creation logic in one place - easy to find and maintain

### 2. Decoupling

Client code separated from concrete class instantiation

### 3. Code Reusability

Multiple clients can use same factory

### 4. Easy Testing

Mock factory for unit tests

### 5. Simplified Client Code

```java
// WITHOUT Factory (messy)
Burger burger;
if (type.equals("BASIC")) {
    burger = new BasicBurger();
} else if (type.equals("STANDARD")) {
    burger = new StandardBurger();
}
// ... more logic ...
burger.prepare();

// WITH Factory (clean)
Burger burger = factory.createBurger(type);
burger.prepare();
```

---

## Limitations & When NOT to Use ❌

### Limitations

1. **Not Extensible:** Adding new products requires modifying factory
2. **Single Factory:** No polymorphic behavior in factory itself
3. **Violates OCP:** Changes require code modification
4. **String Parameters:** Type-safety issues with string-based types

### When NOT to Use

- **Simple object creation** - If `new Object()` suffices, don’t over-engineer
- **No conditional logic** - If only one type exists
- **Minimal variation** - If objects don’t vary much
- **Performance critical** - Extra method call overhead (though negligible)

---

## Common Use Cases

### Real-World Examples

**1. Database Connection Factory**

```java
class ConnectionFactory {
    public Connection createConnection(String dbType) {
        if (dbType.equals("MySQL")) {
            return new MySQLConnection();
        } else if (dbType.equals("PostgreSQL")) {
            return new PostgreSQLConnection();
        } else if (dbType.equals("MongoDB")) {
            return new MongoDBConnection();
        }
        return null;
    }
}
```

**2. Document Parser Factory**

```java
class ParserFactory {
    public Parser createParser(String fileType) {
        if (fileType.equals("PDF")) {
            return new PDFParser();
        } else if (fileType.equals("DOCX")) {
            return new DOCXParser();
        } else if (fileType.equals("TXT")) {
            return new TXTParser();
        }
        return null;
    }
}
```

**3. Payment Processor Factory**

```java
class PaymentFactory {
    public PaymentProcessor createProcessor(String method) {
        if (method.equals("CARD")) {
            return new CardProcessor();
        } else if (method.equals("PAYPAL")) {
            return new PayPalProcessor();
        } else if (method.equals("CRYPTO")) {
            return new CryptoProcessor();
        }
        return null;
    }
}
```

---

## Improvements & Variations

### Using Enum Instead of String

**Problem:** String-based types are error-prone

```java
factory.createBurger("STANDRD");  // Typo! Runtime error
```

**Solution:** Use enums for type safety

```java
enum BurgerType {
    BASIC, STANDARD, PREMIUM
}

class BurgerFactory {
    public Burger createBurger(BurgerType type) {
        switch (type) {
            case BASIC:
                return new BasicBurger();
            case STANDARD:
                return new StandardBurger();
            case PREMIUM:
                return new PremiumBurger();
            default:
                return null;
        }
    }
}

// Usage - Compile-time safety!
Burger burger = factory.createBurger(BurgerType.STANDARD);
```

### Static Factory Method

**Make factory method static if factory has no state:**

```java
class BurgerFactory {
    // Static method - no need to create factory instance
    public static Burger createBurger(String type) {
        if (type.equals("BASIC")) {
            return new BasicBurger();
        }
        // ... rest of logic
    }
}

// Usage - No factory instance needed
Burger burger = BurgerFactory.createBurger("STANDARD");
```

---

## Comparison: Direct Instantiation vs Factory

### Without Factory

```java
// Client code knows concrete classes
public class Restaurant {
    public void serveBurger(String type) {
        Burger burger;

        // Client handles creation logic
        if (type.equals("BASIC")) {
            burger = new BasicBurger();
        } else if (type.equals("STANDARD")) {
            burger = new StandardBurger();
        } else if (type.equals("PREMIUM")) {
            burger = new PremiumBurger();
        } else {
            System.out.println("Invalid type");
            return;
        }

        burger.prepare();
        // Business logic...
    }
}
```

**Issues:**

- Creation logic mixed with business logic
- Multiple places might need same logic (code duplication)
- Client tightly coupled to concrete classes
- Hard to test and maintain

### With Factory

```java
// Client delegates creation to factory
public class Restaurant {
    private BurgerFactory factory = new BurgerFactory();

    public void serveBurger(String type) {
        // Factory handles creation
        Burger burger = factory.createBurger(type);

        if (burger != null) {
            burger.prepare();
            // Business logic...
        }
    }
}
```

**Benefits:**

- Clean separation
- Reusable factory
- Client focused on business logic
- Easy to test with mock factory

---

## Testing Benefits

### Mocking Factory for Unit Tests

```java
// Mock factory for testing
class MockBurgerFactory extends BurgerFactory {
    @Override
    public Burger createBurger(String type) {
        // Return test doubles
        return new MockBurger();
    }
}

// Test
@Test
public void testRestaurant() {
    Restaurant restaurant = new Restaurant();
    restaurant.setFactory(new MockBurgerFactory());  // Inject mock

    // Test business logic without real burgers
    restaurant.serveBurger("STANDARD");

    // Assertions...
}
```

---

## Preview: Factory Method Pattern

**Simple Factory Limitation:** Factory class itself is not extensible

**Factory Method Solution:** Make factory creation polymorphic too!

**Teaser:**

```
Simple Factory:
    - Single concrete factory
    - If-else for creation

Factory Method:
    - Abstract factory class
    - Subclasses override creation method
    - Each subclass creates specific product
    - Follows Open/Closed Principle ✅
```

**We’ll explore this in the next section!**

---

## Key Takeaways

1. **Separation of Concerns:** Object creation logic ≠ Business logic
2. **Factory = Creator:** Dedicated class responsible for creating objects
3. **Decoupling Goal:** Client doesn’t know object creation details
4. **Simple Factory Basics:** Foundation for understanding Factory Method and Abstract Factory
5. **Dependency Injection:** Providing dependencies externally rather than creating internally
6. **Not a True Pattern:** Simple Factory violates OCP, hence not an official pattern
7. **Real-World Usage:** Extremely common in production code (99% applications)
8. **Polymorphism Trick:** Factory returns base type, actual object is concrete subclass
9. **Centralized Logic:** All creation decisions in one place
10. **Next Steps:** Factory Method extends this concept with polymorphic factories

---

## Important Interview Points

### Pattern Category

**Creational Pattern** - Deals with object creation mechanisms

### Key Differences

**Simple Factory vs Factory Method:**

- **Simple Factory:** Single factory class with if-else
- **Factory Method:** Abstract factory, subclasses decide creation

**When Asked:** “Why Factory Pattern?”
**Answer:**

- Separate creation logic from business logic
- Decouple client from concrete classes
- Centralize object creation
- Follow Single Responsibility Principle

### Common Interview Questions

**Q: Why not use `new` directly in client?**
A:

- Tight coupling to concrete classes
- Creation logic scattered across codebase
- Hard to maintain and test
- Violates dependency inversion principle

**Q: Simple Factory vs Factory Method?**
A:

- Simple Factory: Design principle, one concrete factory
- Factory Method: Design pattern, polymorphic factories
- Factory Method follows OCP, Simple Factory doesn’t

**Q: When to use Factory Pattern?**
A:

- Multiple types of similar objects
- Creation logic is complex
- Need to decouple client from concrete classes
- Anticipate adding more types in future

---

## Practice Exercise

**Challenge:** Extend the Burger Shop system:

1. **Add New Burgers:**
    - VeganBurger (plant-based patty)
    - SpicyBurger (extra spicy ingredients)
2. **Add Toppings System:**
    - Each burger can have optional toppings
    - Factory should handle topping configuration
3. **Implement Error Handling:**
    - Return default burger instead of null for invalid types
    - Log all burger creations
4. **Convert to Enum:**
    - Replace string types with BurgerType enum
    - Ensure compile-time type safety

**Bonus:**

- Add price calculation in factory
- Implement builder pattern for complex burgers
- Create static factory methods

---

*These notes cover Simple Factory pattern with practical implementation. This forms the foundation for understanding Factory Method and Abstract Factory patterns, which we’ll cover next. The pattern is fundamental to decoupling object creation from usage in production applications*
