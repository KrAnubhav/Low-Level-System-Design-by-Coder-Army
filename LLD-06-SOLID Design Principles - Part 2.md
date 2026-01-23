---
title: "LLD-06: SOLID Design Principles - Part 2"
date: 2026-01-23
tags: [lld, solid, design-principles, lsp, isp, dip, java]
---

# LLD-06: SOLID Design Principles - Part 2 🏗️

## 📋 Table of Contents
1. [Recap](#recap)
2. [Liskov Substitution Principle - Deep Dive](#liskov-substitution-principle-deep-dive)
3. [Interface Segregation Principle (ISP)](#interface-segregation-principle-isp)
4. [Dependency Inversion Principle (DIP)](#dependency-inversion-principle-dip)

---

## Recap

### Previously Covered (Part 1):
✅ **S** - Single Responsibility Principle (SRP)  
✅ **O** - Open/Closed Principle (OCP)  
✅ **L** - Liskov Substitution Principle (LSP) - Basic Concept

### Today's Focus (Part 2):
🎯 **L** - Liskov Substitution Principle (LSP) - **Advanced Guidelines**  
🎯 **I** - Interface Segregation Principle (ISP)  
🎯 **D** - Dependency Inversion Principle (DIP)

---

## Liskov Substitution Principle - Deep Dive

### Why LSP Needs Special Attention ⚠️

**The Paradox**: LSP seems simple but is the **most frequently violated** principle!

**Why?**
- Violations are **hard to identify**
- Becomes tricky in real-world scenarios
- Critical for interviews and production code

**Solution**: Follow specific **guidelines/rules** to never break LSP

---

## LSP Guidelines Framework

LSP guidelines are divided into **three categories**:

```
┌─────────────────────────────┐
│   LSP Guidelines            │
├─────────────────────────────┤
│ 1. Signature Rules          │
│    • Method Arguments       │
│    • Return Types           │
│    • Exceptions             │
│                             │
│ 2. Property Rules           │
│    • Pre-conditions         │
│    • Post-conditions        │
│    • Invariants             │
│                             │
│ 3. Method Rules             │
│    • History Rule           │
│    • Constraint Rule        │
└─────────────────────────────┘
```

---

## Understanding "Broader" and "Narrower" Types

Before diving into rules, understand these fundamental terms:

### Broader (Parent/Ancestor)

```
┌──────────┐
│  Animal  │ ← Broader (Parent)
└──────────┘
     △
     │ extends
     │
┌──────────┐
│   Dog    │ ← Narrower (Child)
└──────────┘
```

- **Broader** = Parent class or any ancestor in the hierarchy
- **Example**: `Animal` is broader than `Dog`

### Narrower (Child/Descendant)

- **Narrower** = Child class or any descendant in the hierarchy
- **Example**: `Dog` is narrower than `Animal`

---

## 1. Signature Rules

### What is a Method Signature?

A method signature consists of:
- Method **name**
- Method **parameters** (arguments)
- Method **return type**

---

### Rule A: Method Argument Rule

> **Arguments in overridden method should be SAME or BROADER than parent**

#### ✅ Correct Example

```java
class Parent {
    void solve(String s) {
        System.out.println("Parent solving: " + s);
    }
}

class Child extends Parent {
    @Override
    void solve(String s) {  // ✓ Same type - Correct
        System.out.println("Child solving: " + s);
    }
}
```

#### ❌ Incorrect Example

```java
class Parent {
    void solve(String s) {
        System.out.println("Parent solving: " + s);
    }
}

class Child extends Parent {
    void solve(int i) {  // ✗ Different type - Wrong!
        // This doesn't override - it's a different method
        System.out.println("Child solving: " + i);
    }
}
```

#### Why This Rule?

```
┌──────────┐
│  Client  │ ← Expects Parent
└──────────┘
     │
     │ calls solve() with String
     ▼
┌──────────┐
│  Parent  │  solve(String s)
│    or    │
│  Child   │  solve(String s)  ← Must accept same or broader
└──────────┘
```

**Client Code:**
```java
class Client {
    private Parent p;
    
    Client(Parent p) {
        this.p = p;
    }
    
    void execute() {
        p.solve("Hello");  // Client passes String
    }
}

// Usage
Client c1 = new Client(new Parent());
c1.execute();  // Works

Client c2 = new Client(new Child());
c2.execute();  // Also works - LSP satisfied
```

**Key Point**: Java and C++ **enforce** this rule at compile time. You cannot violate it!

---

### Rule B: Return Type Rule

> **Return type in overridden method should be SAME or NARROWER than parent**

#### Class Hierarchy Setup

```
┌──────────┐
│  Animal  │ ← Parent of Dog
└──────────┘
     △
     │ extends
     │
┌──────────┐
│   Dog    │ ← Child of Animal
└──────────┘
```

#### ✅ Correct Examples

```java
class Animal {
    void makeSound() {
        System.out.println("Some sound");
    }
}

class Dog extends Animal {
    @Override
    void makeSound() {
        System.out.println("Bark");
    }
}

class Parent {
    Animal getAnimal() {  // Returns Animal
        return new Animal();
    }
}

// Option 1: Return same type
class Child1 extends Parent {
    @Override
    Animal getAnimal() {  // ✓ Same type
        return new Animal();
    }
}

// Option 2: Return narrower (subtype) - Covariant Return
class Child2 extends Parent {
    @Override
    Dog getAnimal() {  // ✓ Narrower type (Dog IS-A Animal)
        return new Dog();
    }
}
```

#### ❌ Incorrect Example

```java
class Parent {
    Animal getAnimal() {
        return new Animal();
    }
}

class Child extends Parent {
    @Override
    Object getAnimal() {  // ✗ Broader type - Not allowed!
        return new Object();
    }
}
```

#### Why This Rule?

```
Client expects Animal (or its parent)
    ↓
Parent.getAnimal() → Animal
    ↓
Child.getAnimal() → Dog (Dog IS-A Animal) ✓
                  → Animal ✓
                  → Object ✗ (too broad - may not be Animal)
```

**Client Code:**
```java
class Client {
    void processAnimal(Parent p) {
        Animal a = p.getAnimal();  // Client expects Animal
        a.makeSound();  // Works! Even if it's a Dog
    }
}

// Usage
Client client = new Client();
client.processAnimal(new Parent());  // Works
client.processAnimal(new Child2());  // Works - returns Dog (which IS-A Animal)
```

**Key Point**: Returning `Dog` when `Animal` is expected is safe because Dog IS-A Animal (Covariant Return Type)

---

### Rule C: Exception Rule

> **Overridden method should throw SAME, NARROWER, or NO exceptions - NEVER broader**

#### Exception Hierarchy

```
         Exception
             △
             │
    ┌────────┴────────┐
    │                 │
LogicException  RuntimeException
    △
    │
OutOfRangeException
```

#### ✅ Correct Examples

```java
class LogicException extends Exception {
    LogicException(String msg) { super(msg); }
}

class OutOfRangeException extends LogicException {
    OutOfRangeException(String msg) { super(msg); }
}

class Parent {
    void process() throws LogicException {
        throw new LogicException("Parent error");
    }
}

// Option 1: Throw same exception
class Child1 extends Parent {
    @Override
    void process() throws LogicException {  // ✓ Same
        throw new LogicException("Child error");
    }
}

// Option 2: Throw narrower (subtype) exception
class Child2 extends Parent {
    @Override
    void process() throws OutOfRangeException {  // ✓ Narrower
        throw new OutOfRangeException("Child specific error");
    }
}

// Option 3: Throw no exception
class Child3 extends Parent {
    @Override
    void process() {  // ✓ No exception
        System.out.println("Safe implementation");
    }
}
```

#### ❌ Incorrect Example

```java
class Parent {
    void process() throws LogicException {
        throw new LogicException("Parent error");
    }
}

class Child extends Parent {
    @Override
    void process() throws Exception {  // ✗ Broader than LogicException
        throw new Exception("Too broad");
    }
}
```

#### Why This Rule?

**Client Code:**
```java
class Client {
    void execute(Parent p) {
        try {
            p.process();  // p could be Parent or Child
        } catch (LogicException e) {
            // Client prepared to handle LogicException
            // and its subtypes (OutOfRangeException)
            System.out.println("Caught: " + e.getMessage());
        }
    }
}

// Usage
Client client = new Client();
client.execute(new Parent());  // Works - catches LogicException
client.execute(new Child2());  // Works - catches OutOfRangeException (subtype)
```

**Problem if Child throws broader exception:**
```java
// If Child throws Exception (broader than LogicException)
try {
    Parent p = new Child();  // Substituting Child for Parent
    p.process();
} catch (LogicException e) {
    // Only catching LogicException
}
// Exception NOT caught - Program crashes! 💥
```

---

### Summary Table: Signature Rules

| Rule Component | Parent Has | Child Can Have |
|---|---|---|
| **Arguments** | Type T | Same (T) or Broader (Parent of T) |
| **Return Type** | Type T | Same (T) or Narrower (Child of T) |
| **Exceptions** | Exception E | Same (E), Narrower (Child of E), or None |

**Memory Trick:**
- **Arguments**: Can be **BROADER** (more accepting)
- **Return Type**: Can be **NARROWER** (more specific)
- **Exceptions**: Can be **NARROWER** or **NONE** (less risky)

---

## 2. Property Rules

Property rules deal with **class invariants** - conditions that must remain true.

---

### Rule A: Pre-conditions Rule

> **Pre-conditions CANNOT be strengthened in child class**

**Pre-condition**: Requirements that must be satisfied **before** executing a method.

#### ✅ Correct Example

```java
class Parent {
    void processAge(int age) {
        // Pre-condition: age >= 0
        if (age < 0) {
            throw new IllegalArgumentException("Age must be non-negative");
        }
        System.out.println("Processing age: " + age);
    }
}

class Child extends Parent {
    @Override
    void processAge(int age) {
        // ✓ Same pre-condition: age >= 0
        if (age < 0) {
            throw new IllegalArgumentException("Age must be non-negative");
        }
        System.out.println("Child processing age: " + age);
    }
}
```

#### ❌ Incorrect Example (Strengthening pre-condition)

```java
class Parent {
    void processAge(int age) {
        // Pre-condition: age >= 0
        if (age < 0) {
            throw new IllegalArgumentException("Age must be non-negative");
        }
        System.out.println("Processing age: " + age);
    }
}

class Child extends Parent {
    @Override
    void processAge(int age) {
        // ✗ Strengthened: age >= 18 (more restrictive)
        if (age < 18) {
            throw new IllegalArgumentException("Age must be 18+");
        }
        System.out.println("Child processing age: " + age);
    }
}

// Problem:
Parent p = new Child();
p.processAge(10);  // Client expects this to work (age >= 0)
                   // But Child rejects it (requires age >= 18)
                   // LSP violated! 💥
```

**Why it fails:**
- Parent accepts: age >= 0
- Child accepts: age >= 18
- Client code written for Parent expects age >= 0 to work
- Substituting Child breaks client code

---

### Rule B: Post-conditions Rule

> **Post-conditions CANNOT be weakened in child class**

**Post-condition**: Guarantees provided **after** method execution.

#### ✅ Correct Example

```java
class Parent {
    int getDiscount() {
        // Post-condition: returns value between 10-50
        int discount = calculateDiscount();
        if (discount < 10 || discount > 50) {
            throw new IllegalStateException("Discount out of range");
        }
        return discount;
    }
    
    private int calculateDiscount() {
        return 30; // Example
    }
}

class Child extends Parent {
    @Override
    int getDiscount() {
        // ✓ Same or stronger post-condition: 20-40 (within 10-50)
        int discount = calculateDiscount();
        if (discount < 20 || discount > 40) {
            throw new IllegalStateException("Discount out of range");
        }
        return discount;
    }
    
    private int calculateDiscount() {
        return 35; // Example
    }
}
```

#### ❌ Incorrect Example (Weakening post-condition)

```java
class Parent {
    int getDiscount() {
        // Post-condition: returns value between 10-50
        int discount = calculateDiscount();
        if (discount < 10 || discount > 50) {
            throw new IllegalStateException("Discount out of range");
        }
        return discount;
    }
    
    private int calculateDiscount() {
        return 30;
    }
}

class Child extends Parent {
    @Override
    int getDiscount() {
        // ✗ Weakened: 5-60 (outside 10-50 range)
        int discount = calculateDiscount();
        if (discount < 5 || discount > 60) {
            throw new IllegalStateException("Discount out of range");
        }
        return discount;
    }
    
    private int calculateDiscount() {
        return 5; // Could return 5 or 60
    }
}

// Problem:
Parent p = new Child();
int d = p.getDiscount();
// Client expects: 10 <= d <= 50
// Child might return: d = 5 or d = 60
// LSP violated! 💥
```

---

### Rule C: Invariants Rule

> **Class invariants must be PRESERVED by child class**

**Invariant**: Conditions that must **always** be true for the object.

#### ✅ Correct Example

```java
class BankAccount {
    protected double balance;
    
    // Invariant: balance >= 0 (never negative)
    
    BankAccount(double initialBalance) {
        if (initialBalance < 0) {
            throw new IllegalArgumentException("Initial balance cannot be negative");
        }
        this.balance = initialBalance;
    }
    
    void withdraw(double amount) {
        if (balance - amount < 0) {
            throw new IllegalArgumentException("Insufficient funds");
        }
        balance -= amount;
        // Invariant maintained: balance >= 0
    }
    
    double getBalance() {
        return balance;
    }
}

class SavingsAccount extends BankAccount {
    SavingsAccount(double initialBalance) {
        super(initialBalance);
    }
    
    @Override
    void withdraw(double amount) {
        // ✓ Maintains invariant: balance >= 0
        super.withdraw(amount);  // Uses parent's validation
    }
}
```

#### ❌ Incorrect Example (Breaking invariant)

```java
class BankAccount {
    protected double balance;
    
    // Invariant: balance >= 0 (never negative)
    
    BankAccount(double initialBalance) {
        if (initialBalance < 0) {
            throw new IllegalArgumentException("Initial balance cannot be negative");
        }
        this.balance = initialBalance;
    }
    
    void withdraw(double amount) {
        if (balance - amount < 0) {
            throw new IllegalArgumentException("Insufficient funds");
        }
        balance -= amount;
    }
    
    double getBalance() {
        return balance;
    }
}

class OverdraftAccount extends BankAccount {
    OverdraftAccount(double initialBalance) {
        super(initialBalance);
    }
    
    @Override
    void withdraw(double amount) {
        // ✗ Breaks invariant: allows negative balance
        balance -= amount;  // No check!
        // balance can now be < 0
    }
}

// Problem:
BankAccount acc = new OverdraftAccount(100);
acc.withdraw(150);
// Client expects: balance >= 0 (invariant)
// OverdraftAccount allows: balance = -50
// LSP violated! 💥
```

---

## 3. Method Rules

Method rules deal with behavioral consistency.

### Rule A: History Rule

> **Child class should not modify IMMUTABLE properties of parent**

#### ✅ Correct Example

```java
class Point {
    private final int x;  // Immutable
    private final int y;  // Immutable
    
    Point(int x, int y) {
        this.x = x;
        this.y = y;
    }
    
    int getX() { return x; }
    int getY() { return y; }
}

class ColoredPoint extends Point {
    private final String color;
    
    ColoredPoint(int x, int y, String color) {
        super(x, y);
        this.color = color;
    }
    
    String getColor() { return color; }
    
    // ✓ Doesn't modify parent's immutable properties
}
```

#### ❌ Incorrect Example (Violates immutability)

```java
class Point {
    private final int x;  // Immutable
    private final int y;  // Immutable
    
    Point(int x, int y) {
        this.x = x;
        this.y = y;
    }
    
    int getX() { return x; }
    int getY() { return y; }
}

class MutablePoint extends Point {
    private int x, y;  // Shadows parent's fields
    
    MutablePoint(int x, int y) {
        super(x, y);
        this.x = x;
        this.y = y;
    }
    
    // ✗ Violates parent's immutability contract
    void setX(int x) { this.x = x; }
    void setY(int y) { this.y = y; }
}
```

### Rule B: Constraint Rule

> **Child must respect all constraints defined by parent**

---

## Summary: LSP Guidelines

### Signature Rules
✅ **Arguments**: Same or Broader  
✅ **Return Type**: Same or Narrower  
✅ **Exceptions**: Same, Narrower, or None

### Property Rules
✅ **Pre-conditions**: Cannot strengthen  
✅ **Post-conditions**: Cannot weaken  
✅ **Invariants**: Must preserve

### Method Rules
✅ **History**: Maintain immutability  
✅ **Constraints**: Respect parent constraints

---

## Interface Segregation Principle (ISP)

### Definition 📝

> **"Clients should not be forced to depend on interfaces they do not use"**

### Key Concept

- Create **specific, focused** interfaces
- Classes implement only methods they **actually need**
- Avoid **interface pollution** (too many unrelated methods)
- **Many small interfaces > One large interface**

### Real-World Analogy: Universal Remote 🎮

❌ **Bad Design**: One universal remote with buttons for:
- TV
- AC
- Refrigerator
- Washing Machine
- Microwave

**Problems:**
- Too complicated
- Most buttons unused for any single device
- Confusing interface

✅ **Good Design**: Separate remotes for each device
- TV remote: Only TV controls
- AC remote: Only AC controls
- Each device gets only what it needs

---

### Example: Worker Management System

#### ❌ Problem: Violating ISP

**Class Diagram:**

```
┌──────────────────────────┐
│   <<interface>>          │
│      Worker              │
├──────────────────────────┤
│ + work() : void          │
│ + eat() : void           │ ← Not all workers eat!
│ + sleep() : void         │ ← Not all workers sleep!
│ + getSalary() : double   │
└──────────────────────────┘
         △
         ┆ implements
    ┌────┼─────┐
    │          │
┌───┴────┐  ┌──┴───────┐
│ Human  │  │  Robot   │
│ Worker │  │  Worker  │
└────────┘  └──────────┘
```

**Why is this BAD?**

The `Worker` interface has **four methods**, but:
- `RobotWorker` doesn't eat
- `RobotWorker` doesn't sleep
- `RobotWorker` doesn't get salary

**Problems:**
- `RobotWorker` forced to implement methods it doesn't need
- Must throw exceptions or provide dummy implementations
- Interface too large and unfocused
- Violates ISP

#### Java Implementation (Violating ISP)

```java
interface Worker {
    void work();
    void eat();        // ✗ Not all workers eat!
    void sleep();      // ✗ Not all workers sleep!
    double getSalary();
}

class HumanWorker implements Worker {
    public void work() {
        System.out.println("Human working");
    }
    
    public void eat() {
        System.out.println("Human eating lunch");
    }
    
    public void sleep() {
        System.out.println("Human sleeping");
    }
    
    public double getSalary() {
        return 50000.0;
    }
}

class RobotWorker implements Worker {
    public void work() {
        System.out.println("Robot working 24/7");
    }
    
    // ✗ Forced to implement unnecessary methods!
    public void eat() {
        throw new UnsupportedOperationException("Robots don't eat");
    }
    
    public void sleep() {
        throw new UnsupportedOperationException("Robots don't sleep");
    }
    
    public void getSalary() {
        return 0;  // Robots don't get salary
    }
}

// Usage
public class Main {
    public static void main(String[] args) {
        Worker human = new HumanWorker();
        human.work();
        human.eat();
        human.sleep();
        
        Worker robot = new RobotWorker();
        robot.work();
        robot.eat();   // Throws exception! 💥
        robot.sleep(); // Throws exception! 💥
    }
}
```

---

#### ✅ Solution: Following ISP

**Correct Class Diagram:**

```
┌─────────────┐  ┌─────────────┐  ┌──────────────┐  ┌─────────────┐
│<<interface>>│  │<<interface>>│  │ <<interface>>│  │<<interface>>│
│  Workable   │  │   Eatable   │  │  Sleepable   │  │  Payable    │
├─────────────┤  ├─────────────┤  ├──────────────┤  ├─────────────┤
│+ work():void│  │+ eat():void │  │+ sleep():void│  │+getSalary():│
│             │  │             │  │              │  │   double    │
└─────────────┘  └─────────────┘  └──────────────┘  └─────────────┘
      △               △                 △                  △
      ┆               ┆                 ┆                  ┆
      └───────┬───────┴─────────┬───────┴──────────────────┘
              │                 │
      ┌───────┴────────┐  ┌─────┴────────┐
      │  HumanWorker   │  │ RobotWorker  │
      │                │  │              │
      │ implements:    │  │ implements:  │
      │ • Workable     │  │ • Workable   │
      │ • Eatable      │  │              │
      │ • Sleepable    │  │ (only what   │
      │ • Payable      │  │  it needs)   │
      └────────────────┘  └──────────────┘
```

**Benefits:**
- Each interface has a **single responsibility**
- Classes implement only what they **actually need**
- No dummy implementations or exceptions
- Follows ISP perfectly

#### Java Implementation (Following ISP)

```java
// Segregated interfaces
interface Workable {
    void work();
}

interface Eatable {
    void eat();
}

interface Sleepable {
    void sleep();
}

interface Payable {
    double getSalary();
}

// Human implements all relevant interfaces
class HumanWorker implements Workable, Eatable, Sleepable, Payable {
    public void work() {
        System.out.println("Human working");
    }
    
    public void eat() {
        System.out.println("Human eating lunch");
    }
    
    public void sleep() {
        System.out.println("Human sleeping");
    }
    
    public double getSalary() {
        return 50000.0;
    }
}

// Robot implements only what it needs
class RobotWorker implements Workable {
    public void work() {
        System.out.println("Robot working 24/7");
    }
    // No need to implement eat(), sleep(), getSalary()
}

// Usage
public class Main {
    public static void main(String[] args) {
        Workable human = new HumanWorker();
        human.work();
        
        Workable robot = new RobotWorker();
        robot.work();
        
        // Type-safe: Can't call eat() or sleep() on robot
        // robot.eat();   // Compile error! ✓
        // robot.sleep(); // Compile error! ✓
        
        // Can still use full functionality for human
        if (human instanceof Eatable) {
            ((Eatable) human).eat();
        }
    }
}
```

---

## Dependency Inversion Principle (DIP)

### Definition 📝

> **"High-level modules should not depend on low-level modules. Both should depend on abstractions"**  
> **"Abstractions should not depend on details. Details should depend on abstractions"**

### Key Concept

- **High-level modules**: Business logic, core functionality
- **Low-level modules**: Implementation details (database, file system, etc.)
- **Abstraction**: Interfaces or abstract classes

**Traditional Approach (Bad):**
```
High-level Module → Low-level Module
(Business Logic)    (Implementation)
```

**DIP Approach (Good):**
```
High-level Module → Abstraction ← Low-level Module
(Business Logic)    (Interface)   (Implementation)
```

---

### Real-World Analogy: Power Socket 🔌

**Traditional Approach (Bad):**
- Each device has a unique, hardwired connection
- Changing device requires rewiring

**DIP Approach (Good):**
- Standard power socket (abstraction)
- Any device with compatible plug can connect
- Devices depend on socket standard, not specific wiring

---

### Example: Notification System

#### ❌ Problem: Violating DIP

**Class Diagram:**

```
┌──────────────────────┐
│   UserService        │ ← High-level module
├──────────────────────┤
│ - emailService       │ ← Depends on low-level
├──────────────────────┤
│ + registerUser()     │
└──────────────────────┘
         │
         │ depends on
         ▼
┌──────────────────────┐
│   EmailService       │ ← Low-level module
├──────────────────────┤
│ + sendEmail()        │
└──────────────────────┘
```

**Why is this BAD?**

- `UserService` (high-level) directly depends on `EmailService` (low-level)
- Tightly coupled
- Hard to change notification method (SMS, Push, etc.)
- Violates DIP

#### Java Implementation (Violating DIP)

```java
// Low-level module
class EmailService {
    public void sendEmail(String message) {
        System.out.println("Sending email: " + message);
    }
}

// High-level module
class UserService {
    private EmailService emailService;  // ✗ Depends on concrete class
    
    public UserService() {
        this.emailService = new EmailService();  // ✗ Tight coupling
    }
    
    public void registerUser(String username) {
        System.out.println("User registered: " + username);
        emailService.sendEmail("Welcome " + username);
    }
}

// Usage
public class Main {
    public static void main(String[] args) {
        UserService userService = new UserService();
        userService.registerUser("John");
        
        // Problem: What if we want to send SMS instead?
        // We have to modify UserService class!
    }
}
```

**Problems:**
- Can't easily switch to SMS or Push notifications
- Must modify `UserService` to add new notification types
- Violates Open/Closed Principle too!

---

#### ✅ Solution: Following DIP

**Correct Class Diagram:**

```
┌──────────────────────┐
│   UserService        │ ← High-level module
├──────────────────────┤
│ - notifier           │ ← Depends on abstraction
├──────────────────────┤
│ + registerUser()     │
└──────────────────────┘
         │
         │ depends on
         ▼
┌──────────────────────┐
│  <<interface>>       │
│  NotificationService │ ← Abstraction
├──────────────────────┤
│ + send(String)       │
└──────────────────────┘
         △
         ┆ implements
    ┌────┼────┬────────┐
    │         │        │
┌───┴────┐ ┌─┴─────┐ ┌┴────────┐
│Email   │ │ SMS   │ │ Push    │ ← Low-level modules
│Service │ │Service│ │ Service │
├────────┤ ├───────┤ ├─────────┤
│+ send()│ │+ send()│ │+ send() │
└────────┘ └───────┘ └─────────┘
```

**Benefits:**
- `UserService` depends on abstraction, not concrete implementation
- Easy to add new notification types
- Follows Open/Closed Principle
- Follows DIP perfectly

#### Java Implementation (Following DIP)

```java
// Abstraction
interface NotificationService {
    void send(String message);
}

// Low-level module 1
class EmailService implements NotificationService {
    @Override
    public void send(String message) {
        System.out.println("Sending email: " + message);
    }
}

// Low-level module 2
class SMSService implements NotificationService {
    @Override
    public void send(String message) {
        System.out.println("Sending SMS: " + message);
    }
}

// Low-level module 3
class PushService implements NotificationService {
    @Override
    public void send(String message) {
        System.out.println("Sending push notification: " + message);
    }
}

// High-level module
class UserService {
    private NotificationService notifier;  // ✓ Depends on abstraction
    
    // Dependency Injection via constructor
    public UserService(NotificationService notifier) {
        this.notifier = notifier;
    }
    
    public void registerUser(String username) {
        System.out.println("User registered: " + username);
        notifier.send("Welcome " + username);
    }
}

// Usage
public class Main {
    public static void main(String[] args) {
        // Can easily switch notification methods
        UserService emailUser = new UserService(new EmailService());
        emailUser.registerUser("John");
        
        UserService smsUser = new UserService(new SMSService());
        smsUser.registerUser("Jane");
        
        UserService pushUser = new UserService(new PushService());
        pushUser.registerUser("Bob");
        
        // Adding new notification type? Just create new class!
        // No need to modify UserService
    }
}
```

**Output:**
```
User registered: John
Sending email: Welcome John
User registered: Jane
Sending SMS: Welcome Jane
User registered: Bob
Sending push notification: Welcome Bob
```

---

### Dependency Injection

**DIP is often implemented using Dependency Injection (DI):**

**Three types of DI:**

1. **Constructor Injection** (Recommended)
```java
class UserService {
    private NotificationService notifier;
    
    public UserService(NotificationService notifier) {
        this.notifier = notifier;
    }
}
```

2. **Setter Injection**
```java
class UserService {
    private NotificationService notifier;
    
    public void setNotifier(NotificationService notifier) {
        this.notifier = notifier;
    }
}
```

3. **Interface Injection**
```java
interface NotifierInjector {
    void injectNotifier(NotificationService notifier);
}

class UserService implements NotifierInjector {
    private NotificationService notifier;
    
    public void injectNotifier(NotificationService notifier) {
        this.notifier = notifier;
    }
}
```

---

## Summary 📚

### Liskov Substitution Principle (LSP)
✅ Child classes must be substitutable for parent classes  
✅ Follow signature, property, and method rules  
✅ Don't strengthen pre-conditions or weaken post-conditions  

### Interface Segregation Principle (ISP)
✅ Many small interfaces > One large interface  
✅ Classes implement only what they need  
✅ No dummy implementations or exceptions  

### Dependency Inversion Principle (DIP)
✅ Depend on abstractions, not concrete implementations  
✅ High-level modules independent of low-level modules  
✅ Use dependency injection for flexibility  

---

## Complete SOLID Recap 🎯

| Principle | Key Rule | Benefit |
|---|---|---|
| **SRP** | One class = One responsibility | Easy to maintain |
| **OCP** | Open for extension, closed for modification | Easy to extend |
| **LSP** | Child substitutable for parent | Reliable inheritance |
| **ISP** | Many small interfaces | No forced dependencies |
| **DIP** | Depend on abstractions | Loose coupling |

---

## Practice Questions 💡

1. Identify ISP violations in your current codebase
2. Refactor a fat interface into smaller, focused interfaces
3. Design a payment system following DIP (Cash, Card, UPI, Wallet)
4. Create a logging system with DIP (Console, File, Database)
5. Apply all SOLID principles to design a simple e-commerce system

---

*Remember: SOLID principles work together! Mastering them will make you a better software engineer.* 🚀
