# SOLID Design Principles - Part 2

## Recap

This lecture continues the SOLID Design Principles series. Previously covered:

- **S** - Single Responsibility Principle
- **O** - Open/Closed Principle
- **L** - Liskov Substitution Principle

**Remaining to cover**:

- **I** - Interface Segregation Principle
- **D** - Dependency Inversion Principle

---

## Liskov Substitution Principle (LSP) - Deep Dive

### Why LSP Needs Special Attention

**Key Challenge**: LSP appears simple but is the **most frequently violated** principle because:

- Violations are **hard to identify**
- Becomes tricky in practice
- Critical for live projects and interviews

**Solution**: Follow specific **guidelines/rules** to never break LSP and write well-maintained code.

---

## LSP Guidelines Framework

LSP guidelines are divided into **three categories**:

1. **Signature Rules**
2. **Property Rules**
3. **Method Rules**

---

## Understanding “Broader” and “Narrower” Types

**Before diving into rules, understand these terms:**

### Broader (Broad)

- **Parent class** or any **ancestor** in the hierarchy
- **Example**: `Animal` is broader than `Dog`

### Narrower (Narrow)

- **Child class** or any **descendant** in the hierarchy
- **Example**: `Dog` is narrower than `Animal`

**Hierarchy Example**:

```
Animal (Broader)
  ↑
  │ extends
  │
Dog (Narrower)
```

---

## 1. Signature Rules

### What is a Method Signature?

A method signature consists of:

- Method **name**
- Method **parameters** (arguments)
- Method **return type**

### Signature Rules have 3 Sub-Rules:

### A. Method Argument Rule

**Rule**: Arguments in overridden method should be **same** or **broader** than parent.

**Hierarchy Setup**:

```java
class Parent {
    void solve(String s) {
        // Parent logic
    }
}

class Child extends Parent {
    void solve(String s) {  // ✓ Same type - Correct
        // Child logic
    }
}
```

**Why This Rule?**

```
┌──────────┐
│  Client  │  ← Expects Parent
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

**Bad Example (Violates Rule)**:

```java
class Parent {
    void solve(String s) {
        // ...
    }
}

class Child extends Parent {
    void solve(int i) {  // ✗ Different type - Wrong!
        // This doesn't override - it's a different method
    }
}
```

**Why it fails**:

- Client expects to pass `String` (as per Parent contract)
- If Child accepts `int`, client code breaks
- `int` is neither same nor broader than `String`

**Note**: C++ and Java **enforce** this rule - you cannot compile code that violates method argument rules.

**Code Demonstration**:

```java
class Parent {
    void print(String message) {
        System.out.println("Parent: " + message);
    }
}

class Child extends Parent {
    void print(String message) {  // ✓ Correct - same type
        System.out.println("Child: " + message);
    }
}

class Client {
    private Parent p;

    Client(Parent p) {
        this.p = p;
    }

    void printMessage() {
        p.print("Hello");  // Works with both Parent and Child
    }
}

// Usage
Client c1 = new Client(new Parent());
c1.printMessage();  // Works

Client c2 = new Client(new Child());
c2.printMessage();  // Also works - LSP satisfied
```

---

### B. Return Type Rule

**Rule**: Return type in overridden method should be **same** or **narrower** (subtype) than parent.

**Hierarchy Setup**:

```
Animal (Parent of Dog)
  ↑
  │
Dog (Child of Animal)
```

**Class Structure**:

```java
class Animal {
    // Animal class
}

class Dog extends Animal {
    // Dog IS-A Animal
}

class Parent {
    Animal random() {  // Returns Animal
        return new Animal();
    }
}

class Child extends Parent {
    // Option 1: Return same type
    Animal random() {  // ✓ Correct
        return new Animal();
    }

    // Option 2: Return narrower (subtype)
    Dog random() {  // ✓ Also correct (covariant return)
        return new Dog();
    }
}
```

**Why This Rule?**

```
Client expects Animal (or its parent)
    ↓
Parent.random() → Animal
    ↓
Child.random() → Dog (Dog IS-A Animal) ✓
                → Animal ✓
                → Object ✗ (too broad - may not be Animal)
```

**Key Point**:

- Returning `Dog` when `Animal` is expected is safe (Dog IS-A Animal)
- Returning broader type (like `Object`) when `Animal` expected is unsafe

**Plain Text Diagram**:

```
Return Type Rule Flow:

Client Code:
  ↓
  Expects: Animal
  ↓
Parent.random() returns Animal
  ↓
  ┌─────────────────────┐
  │ LSP Substitution    │
  └─────────────────────┘
  ↓
Child.random() can return:
  • Animal (same) ✓
  • Dog (narrower) ✓
  • Object (broader) ✗
```

**Code Example**:

```java
class Animal {
    void makeSound() {
        System.out.println("Some sound");
    }
}

class Dog extends Animal {
    void makeSound() {
        System.out.println("Bark");
    }
}

class Parent {
    Animal getAnimal() {
        return new Animal();
    }
}

class Child extends Parent {
    Dog getAnimal() {  // ✓ Covariant return - returning narrower type
        return new Dog();
    }
}

// Usage
Parent p = new Child();
Animal a = p.getAnimal();  // Client expects Animal
a.makeSound();  // Works! Dog IS-A Animal
```

---

### C. Exception Rule

**Rule**: Overridden method should throw **same**, **narrower**, or **no** exceptions - NEVER broader exceptions.

**Exception Hierarchy**:

```
        Exception
            ↑
            │
   ┌────────┴────────┐
   │                 │
LogicException  RuntimeException
   ↑
   │
OutOfRangeException
```

**Class Structure**:

```java
class Parent {
    void someMethod() throws LogicException {
        // May throw LogicException
        throw new LogicException("Parent error");
    }
}
```

**Correct Child Implementations**:

```java
// Option 1: Throw same exception
class Child1 extends Parent {
    void someMethod() throws LogicException {  // ✓ Same
        throw new LogicException("Child error");
    }
}

// Option 2: Throw narrower (subtype) exception
class Child2 extends Parent {
    void someMethod() throws OutOfRangeException {  // ✓ Narrower
        throw new OutOfRangeException("Child specific error");
    }
}

// Option 3: Throw no exception
class Child3 extends Parent {
    void someMethod() {  // ✓ No exception
        // Safe implementation
    }
}
```

**Incorrect Child Implementation**:

```java
// ✗ Wrong: Throwing broader exception
class Child4 extends Parent {
    void someMethod() throws Exception {  // ✗ Broader than LogicException
        throw new Exception("Too broad");
    }
}
```

**Why This Rule?**

```
Client Code:

try {
    p.someMethod();  // p could be Parent or Child
} catch (LogicException e) {
    // Client prepared to handle LogicException
    // and its subtypes (OutOfRangeException)
}
```

**Problem if Child throws broader exception**:

```java
class Parent {
    void method() throws LogicException { }
}

class Child extends Parent {
    void method() throws RuntimeException {  // ✗ Broader!
        throw new RuntimeException();
    }
}

// Client code
try {
    Parent p = new Child();  // Substituting Child for Parent
    p.method();
} catch (LogicException e) {
    // Only catching LogicException
}
// RuntimeException NOT caught - Program crashes!
```

**Code Demonstration**:

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

class Child extends Parent {
    void process() throws OutOfRangeException {  // ✓ Narrower
        throw new OutOfRangeException("Child error");
    }
}

class Client {
    void execute(Parent p) {
        try {
            p.process();
        } catch (LogicException e) {
            System.out.println("Caught: " + e.getMessage());
            // Handles both LogicException and OutOfRangeException
        }
    }
}

// Usage
Client client = new Client();
client.execute(new Parent());  // Works - catches LogicException
client.execute(new Child());   // Works - catches OutOfRangeException (subtype)
```

**Summary Table**:

| Rule Component | Parent | Child Can Have |
| --- | --- | --- |
| **Arguments** | Type T | Same (T) or Broader (Parent of T) |
| **Return Type** | Type T | Same (T) or Narrower (Child of T) |
| **Exceptions** | Exception E | Same (E), Narrower (Child of E), or None |

---

## 2. Property Rules

Property rules deal with **class invariants** - conditions that must remain true.

### A. Pre-conditions Rule

**Rule**: Pre-conditions **cannot be strengthened** in child class.

**Pre-condition**: Requirements that must be satisfied **before** executing a method.

**Example**:

```java
class Parent {
    void processAge(int age) {
        // Pre-condition: age >= 0
        if (age < 0) {
            throw new IllegalArgumentException("Age must be non-negative");
        }
        // Process age
    }
}
```

**Correct Child**:

```java
class Child extends Parent {
    void processAge(int age) {
        // ✓ Same pre-condition: age >= 0
        if (age < 0) {
            throw new IllegalArgumentException("Age must be non-negative");
        }
        // Process age
    }
}
```

**Incorrect Child** (Strengthening pre-condition):

```java
class Child extends Parent {
    void processAge(int age) {
        // ✗ Strengthened: age >= 18 (more restrictive)
        if (age < 18) {
            throw new IllegalArgumentException("Age must be 18+");
        }
        // Process age
    }
}

// Problem:
Parent p = new Child();
p.processAge(10);  // Client expects this to work (age >= 0)
                   // But Child rejects it (requires age >= 18)
                   // LSP violated!
```

### B. Post-conditions Rule

**Rule**: Post-conditions **cannot be weakened** in child class.

**Post-condition**: Guarantees provided **after** method execution.

**Example**:

```java
class Parent {
    int getDiscount() {
        // Post-condition: returns value between 10-50
        int discount = calculateDiscount();
        assert discount >= 10 && discount <= 50;
        return discount;
    }
}
```

**Correct Child**:

```java
class Child extends Parent {
    int getDiscount() {
        // ✓ Same or stronger post-condition: 20-40 (within 10-50)
        int discount = calculateDiscount();
        assert discount >= 20 && discount <= 40;
        return discount;
    }
}
```

**Incorrect Child** (Weakening post-condition):

```java
class Child extends Parent {
    int getDiscount() {
        // ✗ Weakened: 5-60 (outside 10-50 range)
        int discount = calculateDiscount();
        assert discount >= 5 && discount <= 60;
        return discount;
    }
}

// Problem:
Parent p = new Child();
int d = p.getDiscount();
// Client expects: 10 <= d <= 50
// Child might return: d = 5 or d = 60
// LSP violated!
```

### C. Invariants Rule

**Rule**: Class invariants must be **preserved** by child class.

**Invariant**: Conditions that must **always** be true for the object.

**Example**:

```java
class BankAccount {
    private double balance;

    // Invariant: balance >= 0 (never negative)

    void withdraw(double amount) {
        if (balance - amount < 0) {
            throw new IllegalArgumentException("Insufficient funds");
        }
        balance -= amount;
        // Invariant maintained: balance >= 0
    }
}
```

**Correct Child**:

```java
class SavingsAccount extends BankAccount {
    void withdraw(double amount) {
        // ✓ Maintains invariant: balance >= 0
        super.withdraw(amount);  // Uses parent's validation
    }
}
```

**Incorrect Child** (Breaking invariant):

```java
class OverdraftAccount extends BankAccount {
    void withdraw(double amount) {
        // ✗ Breaks invariant: allows negative balance
        balance -= amount;  // No check!
        // balance can now be < 0
    }
}
```

---

## 3. Method Rules

Method rules deal with behavioral consistency.

### A. History Rule

**Rule**: Child class should not modify **immutable** properties of parent.

**Example**:

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
```

**Incorrect Child** (Violates immutability):

```java
class MutablePoint extends Point {
    private int x, y;

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

### B. Constraint Rule

**Rule**: Child must respect all constraints defined by parent.

### Summary of LSP Guidelines

**Signature Rules**:

- Arguments: Same or Broader
- Return Type: Same or Narrower
- Exceptions: Same, Narrower, or None

**Property Rules**:

- Pre-conditions: Cannot strengthen
- Post-conditions: Cannot weaken
- Invariants: Must preserve

**Method Rules**:

- History: Maintain immutability
- Constraints: Respect parent constraints

---

## 4. Interface Segregation Principle (ISP)

### Definition

> “Clients should not be forced to depend on interfaces they do not use”
> 

**Simplified**: Don’t create “fat” interfaces. Break large interfaces into smaller, focused ones.

### Core Concept

- Create **specific, focused** interfaces
- Classes implement only methods they **actually need**
- Avoid **interface pollution** (too many unrelated methods in one interface)
- Many small interfaces > One large interface

### Problem: Violating ISP

**Scenario**: Worker Management System

**Bad Design**:

```
┌──────────────────────────┐
│   <<interface>>          │
│      Worker              │
├──────────────────────────┤
│ + work() : void          │
│ + eat() : void           │ ← Not all workers eat
│ + sleep() : void         │ ← Not all workers sleep
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

**Code (Bad)**:

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

    public double getSalary() {
        return 0;  // Robots don't get salary
    }
}
```

**Problems**:

1. `RobotWorker` forced to implement `eat()` and `sleep()` which it doesn’t need
2. Must throw exceptions or provide dummy implementations
3. Interface too large and unfocused
4. Violates ISP
5. Client code may call invalid methods

### Solution: Following ISP

Break the large interface into **smaller, focused interfaces**.

**Good Design**:

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

**Code (Good)**:

```java
// Segregated interfaces - each with single responsibility
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
    // ✓ No need to implement eat(), sleep(), getSalary()
}

// Usage
Workable worker1 = new HumanWorker();
Workable worker2 = new RobotWorker();

worker1.work();  // ✓ Works
worker2.work();  // ✓ Works

// Only humans can eat
Eatable eater = new HumanWorker();
eater.eat();  // ✓ Works

// RobotWorker doesn't implement Eatable
// Eatable robot = new RobotWorker();  // ✗ Compile error - prevents mistakes!
```

### Benefits of Following ISP

1. **No Forced Implementation**: Classes implement only needed methods
2. **Better Flexibility**: Easy to add new worker types
3. **Clearer Contracts**: Each interface has focused purpose
4. **Easier Maintenance**: Changes affect fewer classes
5. **Better Code Organization**: Related methods grouped logically
6. **Compile-time Safety**: Cannot call methods that don’t make sense

### ISP in Action - Another Example

**Scenario**: Printer Functionality

**Bad Design** (Fat Interface):

```java
interface MultiFunctionDevice {
    void print();
    void scan();
    void fax();
    void photocopy();
}

class SimplePrinter implements MultiFunctionDevice {
    public void print() {
        System.out.println("Printing...");
    }

    // ✗ Forced to implement features it doesn't have
    public void scan() {
        throw new UnsupportedOperationException();
    }

    public void fax() {
        throw new UnsupportedOperationException();
    }

    public void photocopy() {
        throw new UnsupportedOperationException();
    }
}
```

**Good Design** (ISP Applied):

```java
interface Printer {
    void print();
}

interface Scanner {
    void scan();
}

interface Fax {
    void fax();
}

// Simple printer - only printing capability
class SimplePrinter implements Printer {
    public void print() {
        System.out.println("Printing...");
    }
}

// Advanced printer - multiple capabilities
class MultiFunctionPrinter implements Printer, Scanner, Fax {
    public void print() {
        System.out.println("Printing...");
    }

    public void scan() {
        System.out.println("Scanning...");
    }

    public void fax() {
        System.out.println("Faxing...");
    }
}
```

---

## 5. Dependency Inversion Principle (DIP)

### Definition

> “High-level modules should not depend on low-level modules. Both should depend on abstractions”
> 
> 
> **“Abstractions should not depend on details. Details should depend on abstractions”**
> 

**Simplified**: Depend on **interfaces/abstract classes**, not concrete implementations.

### Core Concept

- **High-level modules** (business logic) should NOT directly depend on **low-level modules** (implementation details)
- Both should depend on **abstractions** (interfaces/abstract classes)
- Creates **loose coupling**
- Easier testing and maintenance

### Problem: Violating DIP

**Scenario**: Notification System

**Bad Design**:

```
┌──────────────────────┐
│ NotificationService  │  ← High-level module
├──────────────────────┤
│ - emailService:      │
│   EmailService       │  ← Direct dependency on concrete class
├──────────────────────┤
│ + notify(msg): void  │
└──────────────────────┘
          │
          │ depends on
          ▼
┌──────────────────────┐
│   EmailService       │  ← Low-level module (concrete)
├──────────────────────┤
│ + send(msg): void    │
└──────────────────────┘
```

**Code (Bad)**:

```java
// Low-level module (concrete implementation)
class EmailService {
    void sendEmail(String message) {
        System.out.println("Sending email: " + message);
    }
}

// High-level module DIRECTLY depends on low-level module
class NotificationService {
    private EmailService emailService;

    NotificationService() {
        this.emailService = new EmailService();  // ✗ Tight coupling!
    }

    void sendNotification(String message) {
        emailService.sendEmail(message);
    }
}
```

**Problems**:

1. `NotificationService` **tightly coupled** to `EmailService`
2. Cannot easily switch to SMS or Push notifications
3. Difficult to test (cannot mock `EmailService`)
4. Adding new notification types requires modifying `NotificationService`
5. Violates OCP and DIP
6. High-level logic depends on low-level details

### Solution: Following DIP

Introduce an **abstraction** (interface) that both depend on.

**Good Design**:

```
                 ┌──────────────────────┐
                 │   <<interface>>      │
                 │  MessageService      │  ← Abstraction
                 ├──────────────────────┤
                 │ + send(msg): void    │
                 └──────────────────────┘
                          △
                          ┆ depends on
                          │
         ┌────────────────┼───────────────────┐
         │                                    │
┌────────┴─────────┐                 ┌───────┴────────┐
│ Notification     │                 │  Concrete      │
│ Service          │                 │  Services      │
│ (High-level)     │                 │  (Low-level)   │
└──────────────────┘                 └────────────────┘
                                              △
                                              │ implements
                         ┌────────────────────┼──────────────────┐
                         │                    │                  │
                   ┌─────┴────────┐  ┌────────┴────────┐ ┌──────┴──────────┐
                   │EmailService  │  │  SmsService     │ │PushNotification │
                   ├──────────────┤  ├─────────────────┤ ├─────────────────┤
                   │ + send()     │  │  + send()       │ │ + send()        │
                   └──────────────┘  └─────────────────┘ └─────────────────┘
```

**Code (Good)**:

```java
// Abstraction (interface) - high-level contract
interface MessageService {
    void send(String message);
}

// Low-level implementations
class EmailService implements MessageService {
    public void send(String message) {
        System.out.println("Sending email: " + message);
    }
}

class SmsService implements MessageService {
    public void send(String message) {
        System.out.println("Sending SMS: " + message);
    }
}

class PushNotificationService implements MessageService {
    public void send(String message) {
        System.out.println("Sending push notification: " + message);
    }
}

// High-level module depends on abstraction (not concrete class)
class NotificationService {
    private MessageService messageService;

    // ✓ Dependency Injection via constructor
    NotificationService(MessageService messageService) {
        this.messageService = messageService;
    }

    void sendNotification(String message) {
        messageService.send(message);  // Uses abstraction
    }
}

// Usage - Flexibility achieved!
NotificationService emailNotifier =
    new NotificationService(new EmailService());
emailNotifier.sendNotification("Hello via Email");

NotificationService smsNotifier =
    new NotificationService(new SmsService());
smsNotifier.sendNotification("Hello via SMS");

NotificationService pushNotifier =
    new NotificationService(new PushNotificationService());
pushNotifier.sendNotification("Hello via Push");
```

### Dependency Injection (DI)

**Three Types of Dependency Injection**:

### 1. Constructor Injection (Most Common)

```java
class Service {
    private Dependency dep;

    Service(Dependency dep) {  // Inject via constructor
        this.dep = dep;
    }
}
```

### 2. Setter Injection

```java
class Service {
    private Dependency dep;

    void setDependency(Dependency dep) {  // Inject via setter
        this.dep = dep;
    }
}
```

### 3. Interface Injection

Less common - dependency provides injector method.

### Benefits of Following DIP

1. **Loose Coupling**: High-level and low-level modules are independent
2. **Easy Testing**: Can inject mock implementations for testing
3. **Flexibility**: Easy to switch implementations without changing client code
4. **Maintainability**: Changes in low-level modules don’t affect high-level
5. **Extensibility**: Add new implementations without modifying existing code
6. **Follows OCP**: Open for extension, closed for modification

### Real-World Analogy for DIP

**Company Hierarchy Example**:

```
CEO (High-level)
  │
  │ communicates through
  ▼
Manager (Abstraction)
  │
  │ manages
  ▼
Developer/QA (Low-level)
```

**How it works**:

- **CEO** (high-level) doesn’t directly interact with **Developers** (low-level)
- **Manager** acts as **abstraction** between them
- If developers change, CEO doesn’t need to know
- CEO only cares about **what** needs to be done (interface), not **who** does it
- Manager handles **how** it gets done (implementation)

**Benefits**:

- CEO can work with any team (Developer, QA, Design) through same Manager interface
- Teams can be replaced/changed without affecting CEO
- Loose coupling between management levels

### Important Relationship

> “If Open/Closed Principle is the goal, then Dependency Inversion Principle is the solution”
> 

**Key Point**: To achieve OCP (Open/Closed Principle), you MUST follow DIP (Dependency Inversion Principle).

---

## Key Takeaways

### LSP Guidelines Summary

**Follow these rules to never violate LSP**:

**Signature Rules**:

- Method Arguments: Same or Broader in child
- Return Types: Same or Narrower in child
- Exceptions: Same, Narrower, or None in child

**Property Rules**:

- Pre-conditions: Cannot be strengthened
- Post-conditions: Cannot be weakened
- Invariants: Must be preserved

**Method Rules**:

- History: Maintain parent’s immutability
- Constraints: Respect all parent constraints

### ISP Key Points

- **Fat interfaces** lead to forced implementations
- Break interfaces into **small, focused** contracts
- Classes implement only **what they need**
- One interface = One **capability/role**
- Better than having unused methods throwing exceptions

### DIP Key Points

- **Never** depend on concrete classes in high-level modules
- Always depend on **abstractions** (interfaces/abstract classes)
- Use **Dependency Injection** to provide implementations
- Enables loose coupling, testing, and flexibility
- **DIP is the solution to achieve OCP**

### SOLID Principles Relationship

```
SRP → Each class has one responsibility
       ↓
OCP → Can extend without modifying
       ↓ (achieved through)
DIP → Depend on abstractions
       ↓ (enables)
LSP → Subtypes are substitutable
       ↓
ISP → Clients use only needed interfaces
```

### Interview Preparation

**Common Questions**:

1. Explain each SOLID principle with example
2. How do you identify LSP violations?
3. What’s the difference between ISP and SRP?
4. How does DIP enable OCP?
5. Provide code examples showing before/after applying principles

**Be Ready To**:

- Identify violations in given code
- Refactor code to follow SOLID principles
- Explain trade-offs and when to apply each principle
- Discuss real-world scenarios from your projects

---

## Final Summary

### Why SOLID Matters

- Creates **maintainable** codebases
- Reduces **bugs** and debugging time
- Makes code **readable** for teams
- Enables **scalability** as projects grow
- Industry **standard** for professional development

### How to Apply in Projects

1. **Start with SRP**: Keep classes focused
2. **Use abstractions**: Interfaces over concrete classes (DIP)
3. **Design carefully**: Think about substitutability (LSP)
4. **Keep interfaces small**: Focused contracts (ISP)
5. **Extend, don’t modify**: Add features via new classes (OCP)

### Practice Tips

- Review existing code for violations
- Refactor one principle at a time
- Practice with coding exercises
- Apply in personal projects
- Discuss with peers during code reviews

SOLID principles are the **foundation** of clean code and are essential for every software developer preparing for interviews and working on real-world projects.

⁂
