## Introduction

This lecture covers the **Strategy Design Pattern** - one of the most widely used behavioral design patterns. It demonstrates how to solve inheritance problems by **separating what changes from what stays constant**, enabling flexible and extensible code.

**Key Topics:**

- What are Design Patterns and why we use them
- Problems with inheritance-heavy designs
- Strategy Pattern implementation with Robot Simulation example
- Composition over Inheritance principle
- Real-world application scenarios

---

## What Are Design Patterns?

### Definition

Design patterns are **proven solutions to common recurring problems** in software development. They represent the collective wisdom of experienced developers who have encountered and solved similar problems before.

### Why Use Design Patterns?

**Core Philosophy:**
> “Change is the only constant” in application development

Applications continuously evolve with new features. Design patterns help us create **flexible designs** where:

- New features require **minimal code changes**
- New features require **minimal time** to integrate
- Code remains **maintainable and scalable**

**Tools for Flexible Design:**

1. **OOP Principles** (Encapsulation, Inheritance, Polymorphism, Abstraction)
2. **SOLID Design Principles**
3. **Design Patterns** ← Today’s focus

### The Golden Rule of Design Patterns

**All design patterns address one fundamental principle:**

> “Identify what changes and separate it from what stays constant”
> 
- **Static parts** → Keep them together, don’t modify
- **Dynamic parts** → Extract and encapsulate separately

This separation ensures that:

- Changes only affect the dynamic parts
- Static parts remain unaffected
- System is open for extension, closed for modification

---

## Problem Statement: Robot Simulation System

### Requirements

Design an application that **simulates different types of robots** with various behaviors:

**Current Behaviors:**

- **Walking** - Robots can walk
- **Talking** - Robots can speak
- **Flying** - Some robots can fly
- **Projection** - How each robot visually appears

**Robot Types (Initial):**

1. **Companion Robot** - Walks, talks, doesn’t fly
2. **Worker Robot** - Walks, talks, doesn’t fly

**Future Requirements:** System should be **extensible** to add:

- New robot types (Sparrow Robot, Crow Robot, Jet Robot, etc.)
- New behaviors (swimming, diving, etc.)
- Different implementations of existing behaviors (fly with wings vs fly with jet)

---

## Approach 1: The Inheritance Problem (Bad Design)

### Initial Design

```
┌─────────────────────┐
│      Robot          │ (Base Class)
├─────────────────────┤
│ - walk()            │
│ - talk()            │
│ - projection() ◇    │ (Abstract - each robot looks different)
└──────────┬──────────┘
           △
           │ extends
      ┌────┴────┐
      │         │
┌─────┴──┐  ┌──┴──────┐
│Companion│  │Worker   │
│Robot   │  │Robot    │
├────────┤  ├─────────┤
│project()│  │project()│
└────────┘  └─────────┘
```

**Seems fine initially** - Companion and Worker robots inherit walk() and talk(), and implement their own projection().

### Problem 1: Adding Flying Behavior

**New Requirement:** Sparrow Robot that can fly

**Attempted Solution 1:** Add fly() to individual robots

```java
class SparrowRobot extends Robot {
    void fly() { /* fly implementation */ }
    void projection() { /* sparrow appearance */ }
}
```

**Issue:** Violates **DRY Principle** (Don’t Repeat Yourself)

- If multiple robots fly the same way, we copy-paste fly() code
- Code duplication across CrowRobot, SparrowRobot, etc.

### Problem 2: Deeper Inheritance Hierarchy

**Attempted Solution 2:** Create intermediate class

```
Robot
  │
  ├── Flyable (fly() implemented here)
  │     ├── SparrowRobot
  │     └── CrowRobot
  │
  └── NonFlyable
        ├── CompanionRobot
        └── WorkerRobot
```

**Issue:** What if flying behavior differs?

- **Sparrow/Crow** fly with wings
- **Jet Robot** flies with jet propulsion
- Need to override fly() again → Back to code duplication!

### Problem 3: Combinatorial Explosion

**New Requirements:**

- Some robots **can’t walk** (NonWalkable)
- Some robots **can’t talk** (NonTalkable)
- Different **walking styles** (normal walk, fast walk, etc.)
- Different **talking styles** (normal talk, loud talk, etc.)
- Different **flying styles** (wings, jet, propeller, etc.)

**Result: Inheritance Hierarchy Nightmare**

```
                    Robot
                      │
        ┌─────────────┼─────────────┐
        │             │             │
    Talkable    NonTalkable       ...
        │             │
    ┌───┴───┐     ┌───┴───┐
Walkable NonWalk Walkable NonWalk
    │       │       │       │
Fly NoFly Fly NoFly Fly NoFly ...

→ Permutation/Combination explosion!
→ Unmanageable inheritance tree
→ Tight coupling
→ Fragile design
```

### Problems Summary ❌

1. **Poor Code Reuse** - Copying behavior implementations
2. **OCP Violation** - Need to modify existing code for new features
3. **Complex Hierarchy** - Inheritance tree becomes unmanageable
4. **Tight Coupling** - Hard to change behaviors independently
5. **Inflexible** - Can’t change behavior at runtime

**Famous Quote:**
> “The solution to inheritance is NOT more inheritance”

---

## Solution: Strategy Design Pattern ✅

### Pattern Definition

> “Define a family of algorithms, put them into separate classes so that they can be changed at runtime”
> 

**Key Concepts:**

- **Family of algorithms** = Different implementations of same behavior (e.g., different flying styles)
- **Separate classes** = Each behavior implementation in its own class
- **Changed at runtime** = Can dynamically switch behaviors

### Core Principle Applied

**Identify what changes:**

- Walking behavior (changes)
- Talking behavior (changes)
- Flying behavior (changes)

**Identify what stays constant:**

- Projection method (each robot unique, but structure constant)
- Robot base structure

**Solution:** Extract changing behaviors into separate class hierarchies

---

## Strategy Pattern Implementation

### Architecture Overview

```
┌────────────────────────────────────────┐
│            Robot (Main Class)          │
├────────────────────────────────────────┤
│ - walkBehavior: Walkable               │ → Composition
│ - talkBehavior: Talkable               │ → Composition
│ - flyBehavior: Flyable                 │ → Composition
├────────────────────────────────────────┤
│ + Robot(w, t, f)  // Constructor       │
│ + walk()      → delegates to w.walk()  │
│ + talk()      → delegates to t.talk()  │
│ + fly()       → delegates to f.fly()   │
│ + projection() // Abstract             │
└────────────────────────────────────────┘
                △
                │ extends
       ┌────────┴────────┐
       │                 │
CompanionRobot      WorkerRobot
  + projection()      + projection()

┌──────────────┐    ┌──────────────┐    ┌──────────────┐
│  <<interface>>│    │  <<interface>>│    │  <<interface>>│
│   Walkable   │    │   Talkable   │    │   Flyable    │
├──────────────┤    ├──────────────┤    ├──────────────┤
│ + walk()     │    │ + talk()     │    │ + fly()      │
└──────┬───────┘    └──────┬───────┘    └──────┬───────┘
       △                   △                   △
       │                   │                   │
  ┌────┴────┐         ┌────┴────┐        ┌────┴────┐
  │         │         │         │        │         │
NormalWalk NoWalk  NormalTalk NoTalk  NormalFly NoFly
FasterWalk        LoudTalk           JetFly
                                     WingFly
```

### Step-by-Step Implementation

### Step 1: Create Behavior Interfaces

**Walkable Interface:**

```java
interface Walkable {
    void walk();
}
```

**Talkable Interface:**

```java
interface Talkable {
    void talk();
}
```

**Flyable Interface:**

```java
interface Flyable {
    void fly();
}
```

### Step 2: Implement Concrete Behaviors

**Walking Implementations:**

```java
// Normal walking
class NormalWalk implements Walkable {
    @Override
    public void walk() {
        System.out.println("Walking normally");
    }
}

// Cannot walk
class NoWalk implements Walkable {
    @Override
    public void walk() {
        System.out.println("Cannot walk");
    }
}

// Fast walking (future extension)
class FasterWalk implements Walkable {
    @Override
    public void walk() {
        System.out.println("Walking faster");
    }
}
```

**Talking Implementations:**

```java
// Normal talking
class NormalTalk implements Talkable {
    @Override
    public void talk() {
        System.out.println("Talking normally");
    }
}

// Cannot talk
class NoTalk implements Talkable {
    @Override
    public void talk() {
        System.out.println("Cannot talk");
    }
}
```

**Flying Implementations:**

```java
// Normal flying (with wings)
class NormalFly implements Flyable {
    @Override
    public void fly() {
        System.out.println("Flying with wings");
    }
}

// Cannot fly
class NoFly implements Flyable {
    @Override
    public void fly() {
        System.out.println("Cannot fly");
    }
}

// Jet propulsion flying
class JetFly implements Flyable {
    @Override
    public void fly() {
        System.out.println("Flying with jet propulsion");
    }
}
```

### Step 3: Create Robot Base Class with Composition

```java
abstract class Robot {
    // Composition: HAS-A relationship
    private Walkable walkBehavior;
    private Talkable talkBehavior;
    private Flyable flyBehavior;

    // Constructor: Inject behaviors
    public Robot(Walkable w, Talkable t, Flyable f) {
        this.walkBehavior = w;
        this.talkBehavior = t;
        this.flyBehavior = f;
    }

    // Delegation methods
    public void walk() {
        walkBehavior.walk();  // Delegates to behavior object
    }

    public void talk() {
        talkBehavior.talk();  // Delegates to behavior object
    }

    public void fly() {
        flyBehavior.fly();    // Delegates to behavior object
    }

    // Setter methods for runtime behavior change (optional)
    public void setWalkBehavior(Walkable w) {
        this.walkBehavior = w;
    }

    public void setTalkBehavior(Talkable t) {
        this.talkBehavior = t;
    }

    public void setFlyBehavior(Flyable f) {
        this.flyBehavior = f;
    }

    // Abstract method - each robot implements its own
    public abstract void projection();
}
```

### Step 4: Create Concrete Robot Classes

```java
class CompanionRobot extends Robot {
    // Constructor specifies exact behaviors
    public CompanionRobot() {
        super(
            new NormalWalk(),   // Can walk normally
            new NormalTalk(),   // Can talk normally
            new NoFly()         // Cannot fly
        );
    }

    @Override
    public void projection() {
        System.out.println("Companion Robot appearance: Friendly design");
    }
}

class WorkerRobot extends Robot {
    public WorkerRobot() {
        super(
            new NormalWalk(),   // Can walk normally
            new NormalTalk(),   // Can talk normally
            new NoFly()         // Cannot fly
        );
    }

    @Override
    public void projection() {
        System.out.println("Worker Robot appearance: Industrial design");
    }
}

class SparrowRobot extends Robot {
    public SparrowRobot() {
        super(
            new NormalWalk(),   // Can walk
            new NormalTalk(),   // Can talk
            new NormalFly()     // Can fly with wings!
        );
    }

    @Override
    public void projection() {
        System.out.println("Sparrow Robot appearance: Bird-like with wings");
    }
}

class JetRobot extends Robot {
    public JetRobot() {
        super(
            new NoWalk(),       // Cannot walk
            new NoTalk(),       // Cannot talk
            new JetFly()        // Flies with jet!
        );
    }

    @Override
    public void projection() {
        System.out.println("Jet Robot appearance: Aerodynamic jet design");
    }
}
```

### Step 5: Client Code Usage

```java
public class Main {
    public static void main(String[] args) {
        // Create different robots
        Robot robot1 = new CompanionRobot();
        Robot robot2 = new WorkerRobot();
        Robot robot3 = new SparrowRobot();
        Robot robot4 = new JetRobot();

        // Robot 1 - Companion
        System.out.println("=== Companion Robot ===");
        robot1.projection();
        robot1.walk();    // Walking normally
        robot1.talk();    // Talking normally
        robot1.fly();     // Cannot fly

        // Robot 3 - Sparrow
        System.out.println("\n=== Sparrow Robot ===");
        robot3.projection();
        robot3.walk();    // Walking normally
        robot3.talk();    // Talking normally
        robot3.fly();     // Flying with wings

        // Robot 4 - Jet
        System.out.println("\n=== Jet Robot ===");
        robot4.projection();
        robot4.walk();    // Cannot walk
        robot4.talk();    // Cannot talk
        robot4.fly();     // Flying with jet propulsion

        // Dynamic behavior change at runtime!
        System.out.println("\n=== Changing Robot 1 Behavior ===");
        robot1.setFlyBehavior(new JetFly());
        robot1.fly();     // Now flies with jet!
    }
}
```

**Output:**

```
=== Companion Robot ===
Companion Robot appearance: Friendly design
Walking normally
Talking normally
Cannot fly

=== Sparrow Robot ===
Sparrow Robot appearance: Bird-like with wings
Walking normally
Talking normally
Flying with wings

=== Jet Robot ===
Jet Robot appearance: Aerodynamic jet design
Cannot walk
Cannot talk
Flying with jet propulsion

=== Changing Robot 1 Behavior ===
Flying with jet propulsion
```

---

## Key Concepts Explained

### Delegation Pattern

**Definition:** An object handles a request by delegating to a helper object

**In Our Example:**

- Robot doesn’t **implement** walk(), talk(), fly()
- Robot **delegates** to behavior objects
- Robot only knows the **interface**, not implementation

**PlainText Flow:**

```
Client calls: robot1.walk()
              ↓
Robot class receives call
              ↓
Robot delegates: walkBehavior.walk()
              ↓
NormalWalk executes its walk() implementation
              ↓
Output: "Walking normally"
```

### Composition Over Inheritance

**Inheritance (IS-A):**

```
SparrowRobot IS-A Robot
→ Tight coupling
→ Compile-time binding
→ Cannot change at runtime
```

**Composition (HAS-A):**

```
Robot HAS-A Walkable behavior
Robot HAS-A Talkable behavior
Robot HAS-A Flyable behavior

→ Loose coupling
→ Runtime binding
→ Can change behaviors dynamically
```

**Why Composition is Better Here:**

1. **Flexibility:** Mix and match any combination of behaviors
2. **No explosion:** Don’t need class for every permutation
3. **Runtime changes:** Can swap behaviors on the fly
4. **Single Responsibility:** Each behavior class has one job
5. **Easy testing:** Test behaviors independently

### Polymorphism in Action

**Runtime Polymorphism:**

```java
Walkable w = new NormalWalk();  // Upcasting
w.walk();  // Calls NormalWalk's implementation

w = new NoWalk();  // Change reference
w.walk();  // Calls NoWalk's implementation
```

**Benefits:**

- Robot doesn’t care about concrete class
- Works with any Walkable implementation
- New behaviors don’t require Robot modification

---

## Advantages of Strategy Pattern ✅

### 1. Open/Closed Principle

- **Open for extension:** Add new behaviors (e.g., SwimBehavior) without modifying existing code
- **Closed for modification:** Robot class doesn’t change when adding new fly implementations

### 2. Single Responsibility Principle

- Each behavior class has **one job**
- NormalWalk only handles walking
- Robot only coordinates behaviors

### 3. Code Reusability

- Write NormalWalk **once**, use in multiple robots
- No code duplication
- DRY principle satisfied

### 4. Runtime Flexibility

```java
robot.setFlyBehavior(new JetFly());  // Change at runtime
robot.fly();  // Now uses jet flying
```

### 5. Easy Testing

- Test each behavior in isolation
- Mock behaviors for robot testing
- Independent unit tests

### 6. Maintainability

- Bug in flying? Fix only Flyable implementations
- Change doesn’t ripple through system
- Clear separation of concerns

### 7. Scalability

Adding new robot with unique combination:

```java
class HybridRobot extends Robot {
    public HybridRobot() {
        super(
            new FasterWalk(),   // New walk style
            new LoudTalk(),     // New talk style
            new JetFly()        // Existing fly style
        );
    }
}
```

**No modification to existing classes!**

---

## When to Use Strategy Pattern

### Use Cases

1. **Multiple algorithms for same task**
    - Different sorting algorithms (QuickSort, MergeSort, BubbleSort)
    - Different compression formats (ZIP, RAR, TAR)
    - Different payment methods (Credit Card, PayPal, Crypto)
2. **Runtime algorithm selection**
    - Choose sorting based on data size
    - Select payment method at checkout
    - Pick rendering engine based on device
3. **Avoiding conditional logic**
    - Replace if-else chains with strategy objects
    - Eliminate switch statements for behavior selection
4. **Behavior-rich classes**
    - Many behaviors that can vary independently
    - Need to mix and match behaviors

### Real-World Examples

**Example 1: Payment Processing**

```java
interface PaymentStrategy {
    void pay(int amount);
}

class CreditCard implements PaymentStrategy {
    public void pay(int amount) { /* process card */ }
}

class PayPal implements PaymentStrategy {
    public void pay(int amount) { /* process PayPal */ }
}

class ShoppingCart {
    private PaymentStrategy paymentMethod;

    public void checkout(PaymentStrategy method) {
        this.paymentMethod = method;
        paymentMethod.pay(calculateTotal());
    }
}
```

**Example 2: Sorting Strategies**

```java
interface SortStrategy {
    void sort(int[] array);
}

class QuickSort implements SortStrategy { /* ... */ }
class MergeSort implements SortStrategy { /* ... */ }

class DataProcessor {
    private SortStrategy sortStrategy;

    public void processData(int[] data) {
        sortStrategy.sort(data);
        // Continue processing...
    }
}
```

**Example 3: Compression Strategies**

```java
interface CompressionStrategy {
    void compress(File file);
}

class ZipCompression implements CompressionStrategy { /* ... */ }
class RarCompression implements CompressionStrategy { /* ... */ }
```

---

## Addressing Common Questions

### Q1: Still Using Inheritance?

**Question:** “You said avoid inheritance, but CompanionRobot extends Robot. Isn’t that inheritance?”

**Answer:** Yes, but **strategic use** is key:

- **Acceptable:** Inheritance for **base structure** (projection method)
- **Problematic:** Inheritance for **behaviors** (walk, talk, fly)
- We use composition for **varying behaviors**
- Minimal inheritance hierarchy (just Robot → ConcreteRobots)

**Alternative:** Even projection can use Strategy Pattern:

```java
interface ProjectionStrategy {
    void project();
}

class Robot {
    private ProjectionStrategy projectionBehavior;
    // ... other behaviors
}
```

**100% composition, zero inheritance!**

### Q2: Too Many Classes?

**Question:** “This creates so many classes. Isn’t that overhead?”

**Answer:**

- **Yes**, more classes initially
- **But**, each class is **simple and focused**
- **Trade-off:** More classes vs complex hierarchy
- **Benefit:** Easy to understand, test, and maintain
- **Modern IDEs** handle many classes effortlessly

### Q3: Performance Impact?

**Question:** “Does delegation affect performance?”

**Answer:**

- **Minimal impact** - one extra method call
- **Modern JVMs** optimize delegation (inlining)
- **Flexibility gains** far outweigh tiny performance cost
- **Premature optimization** is root of all evil

### Q4: When NOT to Use?

**Answer:** Avoid Strategy Pattern when:

- Behaviors **never change**
- Only **one algorithm** exists
- **Simple conditional** suffices (don’t over-engineer)
- Performance is **extremely critical** (embedded systems)

---

## Comparison: Before vs After

### Before Strategy Pattern

```
Problems:
❌ Inheritance explosion
❌ Code duplication
❌ Tight coupling
❌ Hard to extend
❌ Cannot change behavior at runtime
❌ Violates OCP and SRP
```

### After Strategy Pattern

```
Benefits:
✅ Flat class hierarchy
✅ No code duplication
✅ Loose coupling
✅ Easy to extend
✅ Runtime behavior changes
✅ Follows OCP and SRP
✅ Testable components
```

---

## UML Diagram Summary

```
┌─────────────────────────┐
│   Context (Robot)       │
├─────────────────────────┤
│ - strategy: Strategy    │
├─────────────────────────┤
│ + executeStrategy()     │ ──────> Delegates to strategy
└────────┬────────────────┘
         │ uses
         ▼
┌──────────────────────┐
│  <<interface>>       │
│  Strategy            │
├──────────────────────┤
│ + execute()          │
└──────────┬───────────┘
           △
           │ implements
     ┌─────┴─────┐
     │           │
ConcreteA    ConcreteB
```

**Components:**

1. **Context** (Robot) - Maintains reference to Strategy
2. **Strategy Interface** (Walkable, Talkable, Flyable) - Common interface
3. **Concrete Strategies** (NormalWalk, NoFly, etc.) - Implementations

---

## Code Organization Best Practices

### Package Structure

```
com.example.robot
├── Robot.java                    // Abstract base
├── CompanionRobot.java
├── WorkerRobot.java
├── behaviors
│   ├── walking
│   │   ├── Walkable.java        // Interface
│   │   ├── NormalWalk.java
│   │   ├── NoWalk.java
│   │   └── FasterWalk.java
│   ├── talking
│   │   ├── Talkable.java
│   │   ├── NormalTalk.java
│   │   └── NoTalk.java
│   └── flying
│       ├── Flyable.java
│       ├── NormalFly.java
│       ├── NoFly.java
│       └── JetFly.java
└── Main.java
```

### Naming Conventions

- **Interfaces:** Adjective form (Walkable, Flyable, Comparable)
- **Implementations:** Descriptive nouns (NormalWalk, JetFly, QuickSort)
- **Context:** Domain object (Robot, ShoppingCart, Document)

---

## Important Interview Points

### Pattern Category

**Behavioral Pattern** - Defines how objects interact and distribute responsibility

### Related Patterns

1. **State Pattern** - Similar to Strategy but behavior changes based on internal state
2. **Template Method** - Defines algorithm skeleton, subclasses fill in steps
3. **Command Pattern** - Encapsulates request as object

### Key Interview Questions

**Q: Difference between Strategy and State Pattern?**
A:

- **Strategy:** Client chooses algorithm explicitly
- **State:** Object changes behavior based on internal state automatically

**Q: Why interfaces over abstract classes?**
A:

- Java supports **multiple interfaces**, only one abstract class
- Interfaces enforce **contract**, no implementation details
- More **flexible** for future changes

---

## Key Takeaways

1. **Inheritance Problems:** Over-reliance on inheritance leads to rigid, complex hierarchies
2. **Separate Concerns:** Extract varying behaviors from static structure
3. **Composition > Inheritance:** HAS-A relationship provides more flexibility than IS-A
4. **Delegation is Powerful:** Objects delegate responsibilities to specialized helpers
5. **Runtime Flexibility:** Behaviors can be swapped dynamically
6. **Polymorphism Enables Extension:** Work with interfaces, not concrete classes
7. **Most Common Pattern:** Strategy appears in almost every application design
8. **SOLID Principles:** Strategy pattern naturally follows OCP and SRP
9. **Testability:** Isolated behavior classes are easy to unit test
10. **Real-World Ubiquity:** Payment systems, sorting algorithms, compression, rendering - all use Strategy

---

## Practice Exercise

**Challenge:** Extend the robot system to support:

1. **New Behaviors:**
    - Swimming (SwimBehavior: CanSwim, CannotSwim, FastSwim)
    - Sensing (SensingBehavior: VisualSensor, AudioSensor, NoSensor)
2. **New Robots:**
    - AquaBot (walks, talks, swims, doesn’t fly)
    - ScoutBot (doesn’t walk, doesn’t talk, flies, has visual sensor)
3. **Bonus:**
    - Add ability to change multiple behaviors at runtime
    - Create a RobotFactory to build robots with different configurations
    - Implement a GUI where user selects behaviors dynamically

**Hint:** Follow the same pattern - create interfaces for new behaviors, implement concrete classes, compose in Robot.

