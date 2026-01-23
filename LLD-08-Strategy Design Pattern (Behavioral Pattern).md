# LLD-08: Strategy Design Pattern (Behavioral Pattern) 🎯

## Introduction

**Welcome to Design Patterns!** 🎉

Aaj se hum **Design Patterns** start kar rahe hain. Pehla pattern jo hum seekhenge wo hai **Strategy Design Pattern**.

---

## Design Patterns Kya Hain? 🤔

### Definition

> **Design Patterns** = Common problems ke tried-and-tested solutions

**Concept:**
- Jab bhi aap applications develop karte ho, **common problems** aati hain
- Ye problems pehli baar nahi aa rahi - bahut se developers pehle bhi face kar chuke hain
- Unhone already **solutions** find kar rakhe hain
- Hum unki **wisdom** use karte hain - yahi hai Design Patterns!

### Why Design Patterns?

**Application hamesha evolve hoti rehti hai:**
- New features aate rehte hain
- Application ko **flexible** hona chahiye
- Minimal code changes se new features integrate ho sakein
- Minimal time lagana chahiye

**Tools for Flexible Design:**
1. Object-Oriented Principles (OOP)
2. SOLID Design Principles
3. **Design Patterns** ✅

---

## Core Philosophy of Design Patterns

### The Golden Rule

> **"Change is the only constant"**

**Har design pattern yahi kehta hai:**

```
Application mein 2 parts hote hain:
┌─────────────────────────────────────┐
│ 1. Jo CHANGE NAHI hota (Static)    │
│ 2. Jo BAR-BAR CHANGE hota (Dynamic)│
└─────────────────────────────────────┘

Solution:
→ Jo change hota hai, usko SEPARATE karo
→ Jo change nahi hota, usse alag rakho
→ Isse flexibility milti hai
```

**Result:**
- Jab application change hogi, sirf dynamic part change hoga
- Static part unaffected rahega
- Easy to maintain and extend

---

## Problem Statement: Robot Simulation Application 🤖

### Requirements

**Banana hai:** Ek application jisme different types ke robots simulate karenge

**Features:**
- Robots **walk** kar sakte hain
- Robots **talk** kar sakte hain
- Robots **fly** kar sakte hain (kuch robots)
- Har robot **alag dikhta** hai (projection)

---

## Version 1: Bad Design (Using Inheritance) ❌

### Initial Design

```
┌─────────────────────┐
│      Robot          │
├─────────────────────┤
│ + walk()            │
│ + talk()            │
│ + projection()      │ ← Abstract method
└─────────────────────┘
         △
         │ extends
    ┌────┴────┐
    │         │
┌───┴──────┐ ┌┴──────────┐
│Companion │ │  Worker   │
│ Robot    │ │  Robot    │
├──────────┤ ├───────────┤
│+projection│ │+projection│
└──────────┘ └───────────┘
```

**Sab kuch theek lag raha hai... par wait!** 🤔

---

### Problem 1: Flying Robots

**Naya requirement:** Kuch robots **fly** bhi kar sakte hain!

**Example:** Sparrow Robot (udd sakta hai)

#### Approach 1: Add fly() in each robot ❌

```java
class SparrowRobot extends Robot {
    void projection() {
        // Sparrow jaise dikhega
    }
    
    void fly() {  // ✗ Naya method
        // Flying logic
    }
}

class CrowRobot extends Robot {
    void projection() {
        // Crow jaise dikhega
    }
    
    void fly() {  // ✗ Same flying logic - CODE DUPLICATION!
        // Flying logic (same as Sparrow)
    }
}
```

**Problem:** 
- Har flying robot mein `fly()` method **copy-paste** karna padega
- **DRY Principle violation** (Don't Repeat Yourself)

---

#### Approach 2: Add fly() in parent class ❌

```
┌─────────────────────┐
│      Robot          │
├─────────────────────┤
│ + walk()            │
│ + talk()            │
│ + fly()             │ ← Added here
│ + projection()      │
└─────────────────────┘
         △
         │
    ┌────┴────┬─────────┐
    │         │         │
Companion  Worker  Sparrow
```

**Problem:**
- Ab **saare robots** fly kar sakte hain! 😱
- Companion aur Worker ko fly nahi karna chahiye

---

#### Approach 3: Deeper Inheritance Hierarchy ❌

```
        Robot
          │
    ┌─────┴─────┐
    │           │
Flyable    Non-Flyable
    │           │
┌───┴───┐   ┌───┴───┐
Sparrow Crow Companion Worker
```

**Thoda better... par phir naya problem!** 🤯

---

### Problem 2: Different Flying Methods

**Naya requirement:** Kuch robots **jets** se udte hain!

**Example:** JetRobot (jet se udta hai, wings se nahi)

```
            Robot
              │
        ┌─────┴─────┐
        │           │
    Flyable    Non-Flyable
        │
    ┌───┴────┐
    │        │
FlyWithWings FlyWithJet
    │            │
┌───┴───┐    ┌───┴────┐
Sparrow Crow Jet1  Jet2
```

**Hierarchy aur complex ho gayi!** 😵

---

### Problem 3: Non-Walking Robots

**Aur naya requirement:** Kuch robots **walk nahi** kar sakte!

**Ab kya karenge?**
- Walkable / Non-Walkable hierarchy?
- Talkable / Non-Talkable hierarchy?

```
                    Robot
                      │
        ┌─────────────┼─────────────┐
        │             │             │
    Talkable    Non-Talkable   Walkable...
        │             │
    ┌───┴───┐     ┌───┴───┐
Flyable NonFly  Flyable NonFly
    │       │       │       │
  Walk  NoWalk   Walk  NoWalk
    │       │       │       │
   ...     ...     ...     ...
```

**Permutations & Combinations ka explosion!** 💥

---

## Problems with Inheritance Approach

### Summary of Issues

❌ **Code Reuse nahi mil raha**
- Har variation ke liye copy-paste

❌ **Naye features add karne mein bahut changes**
- Existing code modify karna pad raha hai
- **Open/Closed Principle violation**

❌ **Inheritance hierarchy bahut complex**
- Maintain karna mushkil
- Understand karna mushkil

### Famous Quote

> **"The solution to inheritance is NOT more inheritance!"**

---

## Solution: Strategy Design Pattern ✅

### Definition

> **"Define a family of algorithms, put them into separate classes so that they can be changed at runtime"**

**Matlab:**
1. **Family of algorithms** define karo (walk, talk, fly)
2. **Separate classes** mein daalo
3. **Runtime pe change** kar sako

---

## Strategy Pattern Implementation

### Step 1: Identify What Changes

**Robot class mein kya change ho raha hai?**
- ✅ `walk()` - Different ways to walk
- ✅ `talk()` - Different ways to talk  
- ✅ `fly()` - Different ways to fly
- ❌ `projection()` - Har robot apna hi projection hai (not changing)

**Action:** Walk, Talk, Fly ko **separate** karo!

---

### Step 2: Create Strategy Interfaces

**Har behavior ke liye ek interface:**

```java
// Walking Strategy
interface Walkable {
    void walk();
}

// Talking Strategy
interface Talkable {
    void talk();
}

// Flying Strategy
interface Flyable {
    void fly();
}
```

---

### Step 3: Create Concrete Strategies

**Har strategy ke different implementations:**

#### Walking Strategies

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
```

#### Talking Strategies

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

#### Flying Strategies

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

// Flying with jet
class FlyWithJet implements Flyable {
    @Override
    public void fly() {
        System.out.println("Flying with jet engines");
    }
}
```

---

### Step 4: Use Composition in Robot Class

**Inheritance se Composition pe switch karo:**

```java
class Robot {
    // HAS-A relationship (Composition)
    private Walkable walkBehavior;
    private Talkable talkBehavior;
    private Flyable flyBehavior;
    
    // Constructor - Runtime pe decide hoga behavior
    public Robot(Walkable w, Talkable t, Flyable f) {
        this.walkBehavior = w;
        this.talkBehavior = t;
        this.flyBehavior = f;
    }
    
    // Delegation - Apna kaam delegate karo
    public void walk() {
        walkBehavior.walk();  // Delegate to strategy
    }
    
    public void talk() {
        talkBehavior.talk();  // Delegate to strategy
    }
    
    public void fly() {
        flyBehavior.fly();  // Delegate to strategy
    }
    
    // Abstract method - har robot override karega
    public abstract void projection();
}
```

**Robot ab ek "dumb object" ban gaya:**
- Khud kuch nahi karta
- Sab kaam **delegate** kar deta hai strategies ko

---

### Step 5: Create Specific Robots

```java
class CompanionRobot extends Robot {
    public CompanionRobot(Walkable w, Talkable t, Flyable f) {
        super(w, t, f);
    }
    
    @Override
    public void projection() {
        System.out.println("I look like a companion robot");
    }
}

class WorkerRobot extends Robot {
    public WorkerRobot(Walkable w, Talkable t, Flyable f) {
        super(w, t, f);
    }
    
    @Override
    public void projection() {
        System.out.println("I look like a worker robot");
    }
}
```

---

### Step 6: Client Code (Usage)

```java
public class Main {
    public static void main(String[] args) {
        // Companion Robot: Walks, Talks, Cannot Fly
        Robot companion = new CompanionRobot(
            new NormalWalk(),
            new NormalTalk(),
            new NoFly()
        );
        
        companion.walk();  // "Walking normally"
        companion.talk();  // "Talking normally"
        companion.fly();   // "Cannot fly"
        
        // Worker Robot: Cannot Walk, Cannot Talk, Flies with Jet
        Robot worker = new WorkerRobot(
            new NoWalk(),
            new NoTalk(),
            new FlyWithJet()
        );
        
        worker.walk();  // "Cannot walk"
        worker.talk();  // "Cannot talk"
        worker.fly();   // "Flying with jet engines"
    }
}
```

**Output:**
```
Walking normally
Talking normally
Cannot fly
Cannot walk
Cannot talk
Flying with jet engines
```

---

## Complete Java Implementation

```java
// ============= STRATEGIES =============

// Walking Strategies
interface Walkable {
    void walk();
}

class NormalWalk implements Walkable {
    @Override
    public void walk() {
        System.out.println("Walking normally");
    }
}

class NoWalk implements Walkable {
    @Override
    public void walk() {
        System.out.println("Cannot walk");
    }
}

// Talking Strategies
interface Talkable {
    void talk();
}

class NormalTalk implements Talkable {
    @Override
    public void talk() {
        System.out.println("Talking normally");
    }
}

class NoTalk implements Talkable {
    @Override
    public void talk() {
        System.out.println("Cannot talk");
    }
}

// Flying Strategies
interface Flyable {
    void fly();
}

class NormalFly implements Flyable {
    @Override
    public void fly() {
        System.out.println("Flying with wings");
    }
}

class NoFly implements Flyable {
    @Override
    public void fly() {
        System.out.println("Cannot fly");
    }
}

class FlyWithJet implements Flyable {
    @Override
    public void fly() {
        System.out.println("Flying with jet engines");
    }
}

// ============= CLIENT (CONTEXT) =============

abstract class Robot {
    // Composition - HAS-A relationship
    private Walkable walkBehavior;
    private Talkable talkBehavior;
    private Flyable flyBehavior;
    
    public Robot(Walkable w, Talkable t, Flyable f) {
        this.walkBehavior = w;
        this.talkBehavior = t;
        this.flyBehavior = f;
    }
    
    // Delegation methods
    public void walk() {
        walkBehavior.walk();
    }
    
    public void talk() {
        talkBehavior.talk();
    }
    
    public void fly() {
        flyBehavior.fly();
    }
    
    // Abstract method
    public abstract void projection();
}

// ============= CONCRETE ROBOTS =============

class CompanionRobot extends Robot {
    public CompanionRobot(Walkable w, Talkable t, Flyable f) {
        super(w, t, f);
    }
    
    @Override
    public void projection() {
        System.out.println("I look like a companion robot");
    }
}

class WorkerRobot extends Robot {
    public WorkerRobot(Walkable w, Talkable t, Flyable f) {
        super(w, t, f);
    }
    
    @Override
    public void projection() {
        System.out.println("I look like a worker robot");
    }
}

// ============= MAIN =============

public class Main {
    public static void main(String[] args) {
        System.out.println("=== Companion Robot ===");
        Robot companion = new CompanionRobot(
            new NormalWalk(),
            new NormalTalk(),
            new NoFly()
        );
        companion.walk();
        companion.talk();
        companion.fly();
        companion.projection();
        
        System.out.println("\n=== Worker Robot ===");
        Robot worker = new WorkerRobot(
            new NoWalk(),
            new NoTalk(),
            new FlyWithJet()
        );
        worker.walk();
        worker.talk();
        worker.fly();
        worker.projection();
    }
}
```

**Output:**
```
=== Companion Robot ===
Walking normally
Talking normally
Cannot fly
I look like a companion robot

=== Worker Robot ===
Cannot walk
Cannot talk
Flying with jet engines
I look like a worker robot
```

---

## Benefits of Strategy Pattern ✅

### 1. Easy to Add New Behaviors

**Naya flying behavior chahiye?**

```java
class FlyWithRocket implements Flyable {
    @Override
    public void fly() {
        System.out.println("Flying with rockets! 🚀");
    }
}

// Use karo
Robot rocket = new CompanionRobot(
    new NormalWalk(),
    new NormalTalk(),
    new FlyWithRocket()  // ✓ Naya behavior!
);
```

**Koi existing code change nahi karna pada!** 🎉

### 2. Runtime Flexibility

**Runtime pe behavior change kar sakte ho:**

```java
Robot robot = new CompanionRobot(
    new NormalWalk(),  // Initially walks
    new NormalTalk(),
    new NoFly()
);

// Later, change behavior
robot = new CompanionRobot(
    new NoWalk(),      // Now cannot walk
    new NormalTalk(),
    new FlyWithJet()   // Now flies with jet!
);
```

### 3. No Code Duplication

- Har strategy ek hi jagah define hai
- **DRY Principle** follow ho raha hai

### 4. Open/Closed Principle

- **Open for extension** - Naye strategies add kar sakte ho
- **Closed for modification** - Existing code change nahi karna

### 5. Composition over Inheritance

> **"Favor Composition over Inheritance"**

**Yahi Strategy Pattern ka main message hai!**

---

## Removing Last Bit of Inheritance (Optional)

**Question:** Abhi bhi `CompanionRobot` aur `WorkerRobot` inheritance use kar rahe hain?

**Answer:** Haan! Projection ke liye.

**Solution:** Projection ko bhi strategy bana do!

```java
// Projection Strategy
interface Projectable {
    void project();
}

class CompanionProjection implements Projectable {
    @Override
    public void project() {
        System.out.println("I look like a companion");
    }
}

class WorkerProjection implements Projectable {
    @Override
    public void project() {
        System.out.println("I look like a worker");
    }
}

// Robot class (NO inheritance needed!)
class Robot {
    private Walkable walkBehavior;
    private Talkable talkBehavior;
    private Flyable flyBehavior;
    private Projectable projectionBehavior;  // ← New!
    
    public Robot(Walkable w, Talkable t, Flyable f, Projectable p) {
        this.walkBehavior = w;
        this.talkBehavior = t;
        this.flyBehavior = f;
        this.projectionBehavior = p;
    }
    
    public void walk() { walkBehavior.walk(); }
    public void talk() { talkBehavior.talk(); }
    public void fly() { flyBehavior.fly(); }
    public void project() { projectionBehavior.project(); }
}

// Usage - NO child classes needed!
Robot companion = new Robot(
    new NormalWalk(),
    new NormalTalk(),
    new NoFly(),
    new CompanionProjection()
);
```

**Ab inheritance bilkul nahi hai!** 🎊

---

## Standard UML Diagram

```
┌──────────────┐
│   Client     │
├──────────────┤
│ - strategy   │
├──────────────┤
│ + execute()  │
└──────────────┘
       │
       │ has-a
       ▼
┌──────────────┐
│ <<interface>>│
│   Strategy   │
├──────────────┤
│ + run()      │
└──────────────┘
       △
       │ implements
   ┌───┼───┬───────┐
   │   │   │       │
┌──┴┐ ┌┴─┐ ┌┴──┐ ┌─┴──┐
│CS1│ │CS2│ │CS3│ │CS4 │
└───┘ └──┘ └───┘ └────┘
Concrete Strategies
```

**Components:**
1. **Client** - Jo strategies use karta hai (Robot)
2. **Strategy** - Interface/Abstract class (Walkable, Talkable, Flyable)
3. **Concrete Strategies** - Implementations (NormalWalk, NoWalk, etc.)

---

## Real-Life Examples 🌍

### 1. Payment System 💳

```java
class PaymentSystem {
    private PaymentStrategy strategy;
    
    public PaymentSystem(PaymentStrategy strategy) {
        this.strategy = strategy;
    }
    
    public void payNow(double amount) {
        strategy.pay(amount);
    }
}

interface PaymentStrategy {
    void pay(double amount);
}

class UPIPayment implements PaymentStrategy {
    public void pay(double amount) {
        System.out.println("Paid ₹" + amount + " via UPI");
    }
}

class CreditCardPayment implements PaymentStrategy {
    public void pay(double amount) {
        System.out.println("Paid ₹" + amount + " via Credit Card");
    }
}

class NetBankingPayment implements PaymentStrategy {
    public void pay(double amount) {
        System.out.println("Paid ₹" + amount + " via Net Banking");
    }
}

// Usage
PaymentSystem payment = new PaymentSystem(new UPIPayment());
payment.payNow(500);  // "Paid ₹500 via UPI"

payment = new PaymentSystem(new CreditCardPayment());
payment.payNow(1000);  // "Paid ₹1000 via Credit Card"
```

---

### 2. Sorting Algorithms 📊

```java
class Sorter {
    private SortStrategy strategy;
    
    public Sorter(SortStrategy strategy) {
        this.strategy = strategy;
    }
    
    public void sort(int[] arr) {
        strategy.sort(arr);
    }
}

interface SortStrategy {
    void sort(int[] arr);
}

class QuickSort implements SortStrategy {
    public void sort(int[] arr) {
        System.out.println("Sorting using Quick Sort");
        // Quick sort logic
    }
}

class MergeSort implements SortStrategy {
    public void sort(int[] arr) {
        System.out.println("Sorting using Merge Sort");
        // Merge sort logic
    }
}

class BubbleSort implements SortStrategy {
    public void sort(int[] arr) {
        System.out.println("Sorting using Bubble Sort");
        // Bubble sort logic
    }
}

// Usage
int[] data = {5, 2, 8, 1, 9};

Sorter sorter = new Sorter(new QuickSort());
sorter.sort(data);  // "Sorting using Quick Sort"

sorter = new Sorter(new MergeSort());
sorter.sort(data);  // "Sorting using Merge Sort"
```

---

## Key Takeaways 🎯

### Strategy Pattern in One Line

> **"Favor Composition over Inheritance"**

### When to Use Strategy Pattern?

✅ Jab **multiple algorithms** ho same kaam ke liye  
✅ Jab **runtime pe** algorithm change karna ho  
✅ Jab **inheritance hierarchy** complex ho rahi ho  
✅ Jab **code duplication** avoid karna ho  

### Core Concepts

1. **Identify what varies** - Jo change hota hai
2. **Encapsulate it** - Separate classes mein daalo
3. **Program to interface** - Concrete classes pe depend mat karo
4. **Use composition** - HAS-A relationship use karo

---

## Summary

| Aspect | Inheritance Approach ❌ | Strategy Pattern ✅ |
|--------|------------------------|---------------------|
| **Flexibility** | Low | High |
| **Code Reuse** | Duplication | Single place |
| **New Features** | Modify existing | Add new class |
| **Complexity** | High (deep hierarchy) | Low (flat structure) |
| **OCP** | Violates | Follows |
| **Runtime Change** | Cannot | Can |

---

## Concluding Points

✅ **Strategy Pattern** = One of the MOST useful design patterns  
✅ Almost **har LLD interview** mein use hoga  
✅ **Composition > Inheritance** - Hamesha yaad rakho  
✅ **Separate what varies** - Core principle  
✅ **Runtime flexibility** - Biggest advantage  

---

## Practice Exercise 💪

**Try implementing:**

1. **Navigation System** - Different routes (Fastest, Shortest, Scenic)
2. **Compression Tool** - Different algorithms (ZIP, RAR, 7Z)
3. **Image Filter** - Different filters (Grayscale, Sepia, Blur)
4. **Discount Calculator** - Different strategies (Seasonal, Bulk, Member)

---

*Next: Factory Design Pattern (Creational Pattern)*

**Remember:** Strategy Pattern is a **Behavioral Pattern** - It defines how objects interact and distribute responsibility! 🎯
