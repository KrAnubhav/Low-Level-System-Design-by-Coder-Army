# LLD-17: Facade Design Pattern (Structural Pattern)

---

## 📋 Table of Contents
- [Introduction](#introduction)
- [Problem Statement](#problem-statement)
- [Solution: Facade Pattern](#solution-facade-pattern)
- [Example: Computer Boot-Up System](#example-computer-boot-up-system)
- [UML Diagrams](#uml-diagrams)
- [Implementation](#implementation)
- [Principle of Least Knowledge](#principle-of-least-knowledge)
- [Facade vs Adapter Pattern](#facade-vs-adapter-pattern)
- [Real-World Use Cases](#real-world-use-cases)
- [Key Takeaways](#key-takeaways)
- [Interview Q&A](#interview-qa)

---

## 🎯 Introduction

The **Facade Design Pattern** is a structural design pattern that provides a simplified, unified interface to a complex subsystem. It hides the complexity of the system and exposes only what is necessary to the client.

**Definition:**
> "The Facade pattern provides a simplified unified interface to a set of complex subsystems. It hides the complexity of the system and exposes only what is necessary to the user."

**Key Concept:**
- Acts as a **gateway** between client and complex subsystem
- Client interacts with a **single simple interface**
- Facade handles all complex interactions internally

---

## ❓ Problem Statement

### Scenario: Complex Subsystem

Imagine you have a very complex subsystem with multiple classes that interact with each other:

```
┌─────┐     ┌─────┐     ┌─────┐
│  A  │────▶│  B  │────▶│  C  │
└─────┘     └─────┘     └─────┘
   │           │           │
   │           ▼           ▼
   │        ┌─────┐     ┌─────┐
   └───────▶│  D  │────▶│  E  │
            └─────┘     └─────┘

Complex Subsystem with Multiple Dependencies
```

**Problems:**
1. **High Complexity:** Client needs to understand all classes and their interactions
2. **Tight Coupling:** Client is tightly coupled to the subsystem
3. **Difficult to Use:** Client must call multiple methods in correct order
4. **Hard to Maintain:** Changes in subsystem affect client code

**Example Without Facade:**
```java
// Client has to interact with complex subsystem directly
public class Client {
    public void startSystem() {
        A a = new A();
        B b = new B();
        C c = new C();
        D d = new D();
        E e = new E();
        
        // Client must know the correct order and dependencies
        a.initialize();
        b.setup(a);
        c.configure(b);
        d.prepare(a, b);
        e.start(d);
        
        // Too complex! Client knows too much!
    }
}
```

---

## 💡 Solution: Facade Pattern

### Introduce a Facade (Gateway)

Instead of letting the client interact with the complex subsystem directly, introduce a **Facade** class:

```
┌─────────┐
│ Client  │
└─────────┘
     │
     │ Calls simple method
     ▼
┌──────────────┐
│   Facade     │  ← Gateway
│ getWorkDone()│
└──────────────┘
     │
     │ Handles complexity
     ▼
┌────────────────────────────────┐
│    Complex Subsystem           │
│  ┌─────┐  ┌─────┐  ┌─────┐   │
│  │  A  │─▶│  B  │─▶│  C  │   │
│  └─────┘  └─────┘  └─────┘   │
│     │        │        │       │
│     ▼        ▼        ▼       │
│  ┌─────┐  ┌─────┐            │
│  │  D  │─▶│  E  │            │
│  └─────┘  └─────┘            │
└────────────────────────────────┘
```

**Benefits:**
1. ✅ **Simplified Interface:** Client calls one method
2. ✅ **Loose Coupling:** Client doesn't know about subsystem
3. ✅ **Easy to Use:** No need to understand complex interactions
4. ✅ **Easy to Maintain:** Changes in subsystem don't affect client

**Example With Facade:**
```java
// Client interacts with simple facade
public class Client {
    public void startSystem() {
        Facade facade = new Facade();
        facade.getWorkDone(); // That's it!
    }
}
```

---

## 💻 Example: Computer Boot-Up System

### Scenario

When you press the power button on your computer, many complex operations happen behind the scenes:

**Complex Subsystem Components:**
1. **Power Supply** - Provides power
2. **CPU** - Initializes
3. **Memory** - Self-test
4. **Hard Drive** - Spins up
5. **BIOS** - Boots up
6. **Operating System** - Loads
7. **Cooling System** - Starts fans

**Without Facade (Complex):**
```java
// Client has to know all these steps
PowerSupply ps = new PowerSupply();
ps.on();

CPU cpu = new CPU();
cpu.initialize();

Memory memory = new Memory();
memory.selfTest();

HardDrive hd = new HardDrive();
hd.spinUp();

BIOS bios = new BIOS(cpu, memory);
bios.bootUp();

OS os = new OS();
os.load();

CoolingSystem cooling = new CoolingSystem();
cooling.startFans();

// Too complex for a simple "start computer" operation!
```

**With Facade (Simple):**
```java
// Client just calls one method
ComputerFacade computer = new ComputerFacade();
computer.startComputer(); // Done!
```

---

## 🏗️ UML Diagrams

### Class Diagram: Computer Boot-Up Example

```
┌─────────────────┐
│     Client      │
│    (Main)       │
└─────────────────┘
         │
         │ uses
         ▼
┌──────────────────────────────────┐
│      ComputerFacade              │
├──────────────────────────────────┤
│ - powerSupply: PowerSupply       │
│ - cpu: CPU                       │
│ - memory: Memory                 │
│ - hardDrive: HardDrive           │
│ - bios: BIOS                     │
│ - os: OperatingSystem            │
│ - cooling: CoolingSystem         │
├──────────────────────────────────┤
│ + startComputer(): void          │
└──────────────────────────────────┘
         │
         │ coordinates
         ▼
┌────────────────────────────────────────────┐
│         Complex Subsystem                  │
│                                            │
│  ┌──────────────┐  ┌──────────────┐      │
│  │ PowerSupply  │  │ CoolingSystem│      │
│  ├──────────────┤  ├──────────────┤      │
│  │ + on()       │  │ + startFans()│      │
│  └──────────────┘  └──────────────┘      │
│                                            │
│  ┌──────────────┐  ┌──────────────┐      │
│  │     CPU      │  │    Memory    │      │
│  ├──────────────┤  ├──────────────┤      │
│  │ + init()     │  │ + selfTest() │      │
│  └──────────────┘  └──────────────┘      │
│         △  △              │               │
│         │  │              │               │
│         │  └──────┬───────┘               │
│         │         │                       │
│  ┌──────────────────────┐                │
│  │       BIOS           │                │
│  ├──────────────────────┤                │
│  │ - cpu: CPU           │                │
│  │ - memory: Memory     │                │
│  ├──────────────────────┤                │
│  │ + bootUp()           │                │
│  └──────────────────────┘                │
│                                            │
│  ┌──────────────┐  ┌──────────────┐      │
│  │  HardDrive   │  │OperatingSys  │      │
│  ├──────────────┤  ├──────────────┤      │
│  │ + spinUp()   │  │ + load()     │      │
│  └──────────────┘  └──────────────┘      │
└────────────────────────────────────────────┘
```

### Sequence Diagram

```
Client      Facade          PowerSupply  CPU  Memory  HardDrive  BIOS  OS  Cooling
  │           │                  │        │      │        │       │    │     │
  │ start()   │                  │        │      │        │       │    │     │
  │──────────>│                  │        │      │        │       │    │     │
  │           │                  │        │      │        │       │    │     │
  │           │ on()             │        │      │        │       │    │     │
  │           │─────────────────>│        │      │        │       │    │     │
  │           │                  │        │      │        │       │    │     │
  │           │ startFans()      │        │      │        │       │    │     │
  │           │──────────────────────────────────────────────────────────────>│
  │           │                  │        │      │        │       │    │     │
  │           │ initialize()     │        │      │        │       │    │     │
  │           │─────────────────────────>│      │        │       │    │     │
  │           │                  │        │      │        │       │    │     │
  │           │ selfTest()       │        │      │        │       │    │     │
  │           │────────────────────────────────>│        │       │    │     │
  │           │                  │        │      │        │       │    │     │
  │           │ spinUp()         │        │      │        │       │    │     │
  │           │──────────────────────────────────────────>│       │    │     │
  │           │                  │        │      │        │       │    │     │
  │           │ bootUp()         │        │      │        │       │    │     │
  │           │────────────────────────────────────────────────>│    │     │
  │           │                  │        │      │        │       │    │     │
  │           │ load()           │        │      │        │       │    │     │
  │           │──────────────────────────────────────────────────────>│     │
  │           │                  │        │      │        │       │    │     │
  │  Success  │                  │        │      │        │       │    │     │
  │<──────────│                  │        │      │        │       │    │     │
  │           │                  │        │      │        │       │    │     │
```

### Standard UML Structure

```
┌─────────────┐
│   Client    │
└─────────────┘
       │
       │ uses
       ▼
┌─────────────┐
│   Facade    │
├─────────────┤
│ + execute() │
└─────────────┘
       │
       │ coordinates
       ▼
┌──────────────────────────────┐
│    Complex Subsystem         │
│  ┌────┐  ┌────┐  ┌────┐    │
│  │ A  │  │ B  │  │ C  │    │
│  └────┘  └────┘  └────┘    │
│  ┌────┐  ┌────┐            │
│  │ D  │  │ E  │            │
│  └────┘  └────┘            │
└──────────────────────────────┘
```

---

## 💻 Implementation

### Step 1: Subsystem Classes

```java
// PowerSupply.java
public class PowerSupply {
    public void on() {
        System.out.println("Power Supply: Providing power to all components");
    }
}

// CoolingSystem.java
public class CoolingSystem {
    public void startFans() {
        System.out.println("Cooling System: Starting fans");
    }
}

// CPU.java
public class CPU {
    public void initialize() {
        System.out.println("CPU: Initializing processor");
    }
}

// Memory.java
public class Memory {
    public void selfTest() {
        System.out.println("Memory: Running self-test");
    }
}

// HardDrive.java
public class HardDrive {
    public void spinUp() {
        System.out.println("Hard Drive: Spinning up");
    }
}

// BIOS.java
public class BIOS {
    private CPU cpu;
    private Memory memory;
    
    public BIOS(CPU cpu, Memory memory) {
        this.cpu = cpu;
        this.memory = memory;
    }
    
    public void bootUp() {
        System.out.println("BIOS: Booting up");
        cpu.initialize();
        memory.selfTest();
    }
}

// OperatingSystem.java
public class OperatingSystem {
    public void load() {
        System.out.println("Operating System: Loading OS");
    }
}
```

**Explanation:**
- These are all the **subsystem classes**
- Each handles a specific responsibility
- They have complex dependencies on each other
- Client shouldn't interact with these directly

---

### Step 2: Facade Class

```java
// ComputerFacade.java
public class ComputerFacade {
    
    // Subsystem components
    private PowerSupply powerSupply;
    private CoolingSystem coolingSystem;
    private CPU cpu;
    private Memory memory;
    private HardDrive hardDrive;
    private BIOS bios;
    private OperatingSystem os;
    
    // Constructor - Initialize all subsystem components
    public ComputerFacade() {
        this.powerSupply = new PowerSupply();
        this.coolingSystem = new CoolingSystem();
        this.cpu = new CPU();
        this.memory = new Memory();
        this.hardDrive = new HardDrive();
        this.bios = new BIOS(cpu, memory);
        this.os = new OperatingSystem();
    }
    
    // Simplified interface for client
    public void startComputer() {
        System.out.println("Starting Computer...\n");
        
        // Step 1: Power on
        powerSupply.on();
        
        // Step 2: Start cooling
        coolingSystem.startFans();
        
        // Step 3: Initialize CPU
        cpu.initialize();
        
        // Step 4: Test memory
        memory.selfTest();
        
        // Step 5: Spin up hard drive
        hardDrive.spinUp();
        
        // Step 6: Boot BIOS
        bios.bootUp();
        
        // Step 7: Load OS
        os.load();
        
        System.out.println("\nComputer booted successfully!");
    }
}
```

**Explanation:**
- **Facade** contains references to all subsystem components
- **Single method** `startComputer()` provides simplified interface
- **Hides complexity** - Client doesn't need to know the boot sequence
- **Coordinates** all subsystem interactions internally

**Key Points:**
1. Facade **HAS-A** relationship with all subsystem classes
2. Facade provides a **unified interface**
3. Client only knows about the Facade, not the subsystem

---

### Step 3: Client Code

```java
// Main.java (Client)
public class Main {
    public static void main(String[] args) {
        // Create facade
        ComputerFacade computer = new ComputerFacade();
        
        // Start computer with one simple call
        computer.startComputer();
        
        // Client doesn't need to know about:
        // - PowerSupply, CPU, Memory, HardDrive, BIOS, OS, CoolingSystem
        // - The correct order of operations
        // - Dependencies between components
    }
}
```

**Output:**
```
Starting Computer...

Power Supply: Providing power to all components
Cooling System: Starting fans
CPU: Initializing processor
Memory: Running self-test
Hard Drive: Spinning up
BIOS: Booting up
CPU: Initializing processor
Memory: Running self-test
Operating System: Loading OS

Computer booted successfully!
```

**Explanation:**
- Client code is **extremely simple**
- Just creates facade and calls one method
- **No knowledge** of complex subsystem
- **Loose coupling** - Client depends only on Facade

---

## 📚 Principle of Least Knowledge (Law of Demeter)

The Facade pattern helps implement the **Principle of Least Knowledge**, also known as the **Law of Demeter**.

### What is the Principle of Least Knowledge?

**Definition:**
> "Talk only to your immediate friends. A class should have limited knowledge about other classes and should only interact with its immediate dependencies."

### The Problem

```
┌─────┐         ┌─────┐         ┌─────┐
│  A  │────────▶│  B  │────────▶│  C  │
└─────┘         └─────┘         └─────┘
  HAS-A           HAS-A
```

**Bad Practice (Violates Principle):**
```java
public class A {
    private B b;
    
    public void method() {
        // A gets C from B and calls C's method
        C c = b.getC();  // ❌ BAD!
        c.doSomething(); // ❌ A shouldn't know about C!
    }
}
```

**Why is this bad?**
- **Tight Coupling:** A is now coupled to both B and C
- **Fragile Design:** Changes in C affect A
- **Violates Encapsulation:** A knows too much about B's internals

**Good Practice (Follows Principle):**
```java
public class A {
    private B b;
    
    public void method() {
        // A asks B to do the work
        b.doWorkWithC(); // ✅ GOOD! A only talks to B
    }
}

public class B {
    private C c;
    
    public void doWorkWithC() {
        // B handles interaction with C
        c.doSomething();
    }
}
```

---

### The Four Rules of Least Knowledge

A method in a class should **ONLY** call methods of:

#### Rule 1: The Object Itself

```java
public class A {
    
    // ✅ Can call its own methods
    public void m1() {
        m2(); // Calling own method - OK
    }
    
    public void m2() {
        System.out.println("Method 2");
    }
}
```

#### Rule 2: Objects Passed as Parameters

```java
public class A {
    
    // ✅ Can call methods on parameter objects
    public void m3(B b) {
        b.doSomething(); // Parameter object - OK
    }
}
```

#### Rule 3: Objects Created Locally

```java
public class A {
    
    // ✅ Can call methods on locally created objects
    public void m4() {
        D d = new D(); // Created locally
        d.doSomething(); // OK
    }
}
```

#### Rule 4: Component Objects (HAS-A relationship)

```java
public class A {
    private B b; // HAS-A relationship
    
    // ✅ Can call methods on component objects
    public void m5() {
        b.doSomething(); // Component object - OK
    }
}
```

---

### Complete Example: Following the Rules

```java
// Class A with all allowed interactions
public class A {
    private B b; // Component (HAS-A)
    
    public A() {
        this.b = new B();
    }
    
    public void complexOperation(C paramC) {
        // Rule 1: Call own methods
        this.helperMethod();
        
        // Rule 2: Call methods on parameters
        paramC.doWork();
        
        // Rule 3: Call methods on locally created objects
        D localD = new D();
        localD.process();
        
        // Rule 4: Call methods on component objects
        b.execute();
    }
    
    private void helperMethod() {
        System.out.println("Helper method");
    }
}

public class B {
    private C c; // B HAS-A C
    
    public void execute() {
        // B can interact with C
        c.doWork();
    }
    
    // ❌ DON'T expose C to A
    // public C getC() { return c; }
}
```

---

### What NOT to Do

```java
// ❌ VIOLATION: Chain of calls
public class A {
    private B b;
    
    public void badMethod() {
        // Getting C from B and calling C's method
        C c = b.getC();      // ❌ Violates principle
        c.doSomething();     // ❌ A shouldn't know about C
        
        // Even worse: Chain of calls
        b.getC().getD().doWork(); // ❌❌❌ Very bad!
    }
}

// ✅ CORRECT: Ask B to do the work
public class A {
    private B b;
    
    public void goodMethod() {
        // Let B handle interaction with C
        b.doWorkWithC(); // ✅ Correct
    }
}

public class B {
    private C c;
    
    public void doWorkWithC() {
        c.doSomething(); // B handles C
    }
}
```

---

### Benefits of Principle of Least Knowledge

1. ✅ **Loose Coupling:** Classes are less dependent on each other
2. ✅ **Better Encapsulation:** Internal details are hidden
3. ✅ **Easier Maintenance:** Changes in one class don't ripple through
4. ✅ **More Flexible:** Easy to modify and extend

### Visual Summary

```
❌ BAD: Too much knowledge
┌─────┐
│  A  │────────┐
└─────┘        │
   │           │
   │ knows     │ knows
   ▼           ▼
┌─────┐     ┌─────┐
│  B  │────▶│  C  │
└─────┘     └─────┘

A knows about both B and C (Tight coupling)


✅ GOOD: Limited knowledge
┌─────┐
│  A  │
└─────┘
   │
   │ knows only
   ▼
┌─────┐
│  B  │────▶┌─────┐
└─────┘     │  C  │
            └─────┘

A knows only about B (Loose coupling)
```

---

## 🔄 Facade vs Adapter Pattern

Many people confuse **Facade** and **Adapter** patterns because they look similar. Here's the key difference:

### Intent Comparison

| Aspect | Facade Pattern | Adapter Pattern |
|--------|---------------|-----------------|
| **Intent** | Hide complexity of subsystem | Make incompatible interfaces work together |
| **Purpose** | Simplify interface | Convert interface |
| **Focus** | Provide easier access | Enable compatibility |
| **Subsystem** | Complex but compatible | Incompatible interfaces |
| **Number of Classes** | Multiple subsystem classes | Usually one adaptee |

---

### Visual Comparison

#### Facade Pattern
```
┌─────────┐
│ Client  │
└─────────┘
     │
     │ Simple interface
     ▼
┌──────────┐
│  Facade  │  ← Hides complexity
└──────────┘
     │
     │ Complex interactions
     ▼
┌────────────────────┐
│ Complex Subsystem  │
│  (Many classes)    │
└────────────────────┘

Goal: Simplify access to complex system
```

#### Adapter Pattern
```
┌─────────┐
│ Client  │ (Expects Interface A)
└─────────┘
     │
     ▼
┌──────────┐
│ Adapter  │  ← Converts interface
└──────────┘
     │
     ▼
┌──────────┐
│ Adaptee  │ (Has Interface B)
└──────────┘

Goal: Make incompatible interfaces compatible
```

---

### Code Comparison

#### Facade Example
```java
// Client expects simple interface
public class Client {
    public void work() {
        Facade facade = new Facade();
        facade.doComplexWork(); // Simple call
    }
}

// Facade hides complexity
public class Facade {
    private SubsystemA a;
    private SubsystemB b;
    private SubsystemC c;
    
    public void doComplexWork() {
        // Coordinates multiple subsystems
        a.step1();
        b.step2();
        c.step3();
    }
}
```

#### Adapter Example
```java
// Client expects ITarget interface
public class Client {
    public void work(ITarget target) {
        target.request(); // Expects this method
    }
}

// Adapter converts Adaptee to ITarget
public class Adapter implements ITarget {
    private Adaptee adaptee;
    
    @Override
    public void request() {
        // Converts interface
        adaptee.specificRequest();
    }
}
```

---

### When to Use Which?

**Use Facade when:**
- ✅ You have a complex subsystem
- ✅ You want to provide a simpler interface
- ✅ You want to decouple client from subsystem
- ✅ You want to hide implementation details

**Use Adapter when:**
- ✅ You have incompatible interfaces
- ✅ You want to reuse existing class with different interface
- ✅ You're integrating third-party libraries
- ✅ You need to convert one interface to another

---

## 🌍 Real-World Use Cases

### 1. Game Engine (Unity)

**Scenario:** Loading a game

```java
// Without Facade - Complex
public class GameClient {
    public void loadGame() {
        GameAssets assets = new GameAssets();
        assets.loadTextures();
        assets.loadModels();
        assets.loadSounds();
        
        MemoryManager memory = new MemoryManager();
        memory.allocateHeap();
        memory.setupGarbageCollection();
        
        PhysicsEngine physics = new PhysicsEngine();
        physics.initializeGravity();
        physics.setupCollisionDetection();
        
        RenderingEngine renderer = new RenderingEngine();
        renderer.initializeOpenGL();
        renderer.setupShaders();
        
        // Too complex!
    }
}

// With Facade - Simple
public class GameClient {
    public void loadGame() {
        GameFacade game = new GameFacade();
        game.startGame(); // That's it!
    }
}

public class GameFacade {
    private GameAssets assets;
    private MemoryManager memory;
    private PhysicsEngine physics;
    private RenderingEngine renderer;
    
    public void startGame() {
        System.out.println("Loading game...");
        
        // Load assets
        assets.loadTextures();
        assets.loadModels();
        assets.loadSounds();
        
        // Setup memory
        memory.allocateHeap();
        memory.setupGarbageCollection();
        
        // Initialize physics
        physics.initializeGravity();
        physics.setupCollisionDetection();
        
        // Setup rendering
        renderer.initializeOpenGL();
        renderer.setupShaders();
        
        System.out.println("Game loaded successfully!");
    }
}
```

---

### 2. Payment Gateway

**Scenario:** Processing a payment

```java
// Payment subsystem classes
public class BalanceChecker {
    public boolean checkBalance(String accountId, double amount) {
        System.out.println("Checking balance...");
        return true; // Simplified
    }
}

public class PinValidator {
    public boolean validatePin(String pin) {
        System.out.println("Validating PIN...");
        return true; // Simplified
    }
}

public class FraudDetection {
    public boolean checkFraud(String accountId, double amount) {
        System.out.println("Running fraud detection...");
        return false; // No fraud
    }
}

public class TransactionProcessor {
    public String processTransaction(String from, String to, double amount) {
        System.out.println("Processing transaction...");
        return "TXN123456"; // Transaction ID
    }
}

public class NotificationService {
    public void sendNotification(String accountId, String message) {
        System.out.println("Sending notification: " + message);
    }
}

// Facade
public class PaymentFacade {
    private BalanceChecker balanceChecker;
    private PinValidator pinValidator;
    private FraudDetection fraudDetection;
    private TransactionProcessor transactionProcessor;
    private NotificationService notificationService;
    
    public PaymentFacade() {
        this.balanceChecker = new BalanceChecker();
        this.pinValidator = new PinValidator();
        this.fraudDetection = new FraudDetection();
        this.transactionProcessor = new TransactionProcessor();
        this.notificationService = new NotificationService();
    }
    
    public boolean makePayment(String fromAccount, String toAccount, 
                               double amount, String pin) {
        System.out.println("Initiating payment...\n");
        
        // Step 1: Validate PIN
        if (!pinValidator.validatePin(pin)) {
            System.out.println("Invalid PIN!");
            return false;
        }
        
        // Step 2: Check balance
        if (!balanceChecker.checkBalance(fromAccount, amount)) {
            System.out.println("Insufficient balance!");
            return false;
        }
        
        // Step 3: Fraud detection
        if (fraudDetection.checkFraud(fromAccount, amount)) {
            System.out.println("Fraudulent transaction detected!");
            return false;
        }
        
        // Step 4: Process transaction
        String txnId = transactionProcessor.processTransaction(
            fromAccount, toAccount, amount
        );
        
        // Step 5: Send notification
        notificationService.sendNotification(
            fromAccount, 
            "Payment of $" + amount + " successful. TXN ID: " + txnId
        );
        
        System.out.println("\nPayment completed successfully!");
        return true;
    }
}

// Client
public class PaymentClient {
    public static void main(String[] args) {
        PaymentFacade payment = new PaymentFacade();
        
        // Simple call - all complexity hidden
        payment.makePayment("ACC001", "ACC002", 1000.0, "1234");
    }
}
```

---

### 3. Home Theater System

```java
// Subsystem classes
public class DVDPlayer {
    public void on() { System.out.println("DVD Player ON"); }
    public void play(String movie) { 
        System.out.println("Playing: " + movie); 
    }
}

public class Projector {
    public void on() { System.out.println("Projector ON"); }
    public void setInput(String input) { 
        System.out.println("Projector input: " + input); 
    }
}

public class SoundSystem {
    public void on() { System.out.println("Sound System ON"); }
    public void setVolume(int level) { 
        System.out.println("Volume set to: " + level); 
    }
}

public class Lights {
    public void dim(int level) { 
        System.out.println("Lights dimmed to: " + level + "%"); 
    }
}

// Facade
public class HomeTheaterFacade {
    private DVDPlayer dvd;
    private Projector projector;
    private SoundSystem sound;
    private Lights lights;
    
    public HomeTheaterFacade() {
        this.dvd = new DVDPlayer();
        this.projector = new Projector();
        this.sound = new SoundSystem();
        this.lights = new Lights();
    }
    
    public void watchMovie(String movie) {
        System.out.println("Get ready to watch a movie...\n");
        
        lights.dim(10);
        projector.on();
        projector.setInput("DVD");
        sound.on();
        sound.setVolume(5);
        dvd.on();
        dvd.play(movie);
        
        System.out.println("\nEnjoy your movie!");
    }
    
    public void endMovie() {
        System.out.println("Shutting down home theater...");
        // Turn off all components
    }
}

// Client
public class Client {
    public static void main(String[] args) {
        HomeTheaterFacade theater = new HomeTheaterFacade();
        theater.watchMovie("Inception");
    }
}
```

---

### 4. E-commerce Order Processing

```java
// Facade for order processing
public class OrderFacade {
    private InventoryService inventory;
    private PaymentService payment;
    private ShippingService shipping;
    private NotificationService notification;
    private InvoiceService invoice;
    
    public boolean placeOrder(Order order) {
        // Check inventory
        if (!inventory.checkAvailability(order.getItems())) {
            return false;
        }
        
        // Process payment
        if (!payment.processPayment(order.getPaymentInfo())) {
            return false;
        }
        
        // Reserve inventory
        inventory.reserveItems(order.getItems());
        
        // Create shipment
        String trackingId = shipping.createShipment(order);
        
        // Generate invoice
        invoice.generateInvoice(order);
        
        // Send notifications
        notification.sendOrderConfirmation(order, trackingId);
        
        return true;
    }
}
```

---

## 🎯 Key Takeaways

### When to Use Facade Pattern

✅ **Use Facade Pattern when:**
1. You have a complex subsystem with many classes
2. You want to provide a simple interface to clients
3. You want to decouple client from subsystem implementation
4. You want to layer your subsystems
5. You want to hide implementation details

❌ **Don't Use Facade Pattern when:**
1. The subsystem is already simple
2. Clients need fine-grained control over subsystem
3. You're just adding unnecessary abstraction

---

### Design Principles Applied

1. **Principle of Least Knowledge:** Client knows only about Facade
2. **Loose Coupling:** Client decoupled from subsystem
3. **Single Responsibility:** Facade coordinates subsystem
4. **Encapsulation:** Hides subsystem complexity

---

### Benefits

1. ✅ **Simplified Interface:** Easy to use
2. ✅ **Loose Coupling:** Client independent of subsystem
3. ✅ **Flexibility:** Easy to change subsystem
4. ✅ **Layering:** Can create layers of facades
5. ✅ **Maintainability:** Changes isolated to facade

---

### Drawbacks

1. ❌ **God Object:** Facade can become too large
2. ❌ **Limited Functionality:** May not expose all features
3. ❌ **Additional Layer:** Slight performance overhead

---

## ❓ Interview Q&A

### Q1: What is the Facade Design Pattern?

**Answer:**
The Facade Design Pattern is a structural pattern that provides a simplified, unified interface to a complex subsystem. It hides the complexity and exposes only what's necessary to the client.

**Key Points:**
- Acts as a gateway to complex subsystem
- Provides simple interface
- Hides implementation details
- Decouples client from subsystem

**Example:**
```java
// Instead of this (complex)
cpu.init();
memory.test();
bios.boot();
os.load();

// Client does this (simple)
computer.startComputer();
```

---

### Q2: What is the Principle of Least Knowledge?

**Answer:**
The Principle of Least Knowledge (Law of Demeter) states: **"Talk only to your immediate friends."**

**Four Rules - A method can call methods on:**
1. The object itself
2. Objects passed as parameters
3. Objects created locally
4. Component objects (HAS-A relationship)

**What NOT to do:**
```java
// ❌ BAD: Chain of calls
a.getB().getC().doWork();

// ✅ GOOD: Ask B to do the work
a.doWorkWithC();
```

**Benefits:**
- Loose coupling
- Better encapsulation
- Easier maintenance

---

### Q3: What's the difference between Facade and Adapter patterns?

**Answer:**

| Aspect | Facade | Adapter |
|--------|--------|---------|
| **Intent** | Hide complexity | Convert interface |
| **Purpose** | Simplify | Enable compatibility |
| **Subsystem** | Complex but compatible | Incompatible interfaces |
| **Focus** | Easier access | Interface conversion |

**Facade Example:**
```java
// Simplifies complex subsystem
facade.startComputer(); // Hides boot complexity
```

**Adapter Example:**
```java
// Converts XML to JSON
IReports adapter = new XMLAdapter(xmlProvider);
String json = adapter.getJsonData(); // Interface conversion
```

**Key Difference:**
- **Facade:** "I want to make this easier to use"
- **Adapter:** "I want to make these incompatible things work together"

---

### Q4: Can you give real-world examples of Facade Pattern?

**Answer:**

**1. Game Engines (Unity, Unreal):**
```java
GameFacade game = new GameFacade();
game.startGame(); // Loads assets, physics, rendering, etc.
```

**2. Payment Gateways:**
```java
PaymentFacade payment = new PaymentFacade();
payment.makePayment(amount); // Handles validation, fraud check, etc.
```

**3. Java Libraries:**
```java
// JDBC is a facade over different database drivers
Connection conn = DriverManager.getConnection(url);
```

**4. Home Automation:**
```java
SmartHomeFacade home = new SmartHomeFacade();
home.leaveHome(); // Locks doors, turns off lights, sets thermostat
```

---

### Q5: What are the advantages and disadvantages of Facade Pattern?

**Answer:**

**Advantages:**
- ✅ **Simplified Interface:** Easy for clients to use
- ✅ **Loose Coupling:** Client independent of subsystem
- ✅ **Flexibility:** Easy to change subsystem implementation
- ✅ **Layering:** Can create multiple facades for different purposes
- ✅ **Maintainability:** Changes isolated to facade

**Disadvantages:**
- ❌ **God Object Risk:** Facade can become too large
- ❌ **Limited Functionality:** May not expose all subsystem features
- ❌ **Additional Layer:** Slight performance overhead
- ❌ **Over-abstraction:** Can hide too much, making debugging harder

---

### Q6: Can a system have multiple facades?

**Answer:**

**Yes! You can have multiple facades for different purposes:**

```java
// Facade for basic users
public class BasicComputerFacade {
    public void startComputer() {
        // Simple boot
    }
}

// Facade for advanced users
public class AdvancedComputerFacade {
    public void startComputer() {
        // Simple boot
    }
    
    public void startInSafeMode() {
        // Advanced boot option
    }
    
    public void runDiagnostics() {
        // Diagnostic tools
    }
}

// Facade for administrators
public class AdminComputerFacade {
    // Even more advanced options
}
```

**Benefits:**
- Different facades for different user types
- Layered architecture
- Separation of concerns

---

### Q7: How does Facade Pattern help with testing?

**Answer:**

**Facade makes testing easier:**

```java
// Without Facade - Hard to test
public class OrderService {
    public void processOrder() {
        // Directly uses many subsystems
        inventory.check();
        payment.process();
        shipping.create();
        // Hard to mock all these!
    }
}

// With Facade - Easy to test
public class OrderService {
    private OrderFacade orderFacade;
    
    public void processOrder() {
        orderFacade.placeOrder(order);
    }
}

// Test
@Test
public void testOrderProcessing() {
    OrderFacade mockFacade = mock(OrderFacade.class);
    when(mockFacade.placeOrder(any())).thenReturn(true);
    
    OrderService service = new OrderService(mockFacade);
    // Easy to test!
}
```

**Benefits:**
- Mock only the facade
- Simpler test setup
- Faster tests (no subsystem dependencies)

---

### Q8: Can Facade Pattern be combined with other patterns?

**Answer:**

**Yes! Facade works well with other patterns:**

**1. Facade + Singleton:**
```java
public class ComputerFacade {
    private static ComputerFacade instance;
    
    private ComputerFacade() {}
    
    public static ComputerFacade getInstance() {
        if (instance == null) {
            instance = new ComputerFacade();
        }
        return instance;
    }
}
```

**2. Facade + Factory:**
```java
public class FacadeFactory {
    public static ComputerFacade createFacade(String type) {
        if (type.equals("basic")) {
            return new BasicComputerFacade();
        } else {
            return new AdvancedComputerFacade();
        }
    }
}
```

**3. Facade + Strategy:**
```java
public class PaymentFacade {
    private PaymentStrategy strategy;
    
    public void setStrategy(PaymentStrategy strategy) {
        this.strategy = strategy;
    }
    
    public void makePayment(double amount) {
        // Use strategy internally
        strategy.pay(amount);
    }
}
```

---

### Q9: How does Facade Pattern support the Open/Closed Principle?

**Answer:**

**Facade supports OCP by:**

**1. Open for Extension:**
```java
// Can add new methods to facade
public class ComputerFacade {
    public void startComputer() { }
    
    // Add new functionality
    public void startInSafeMode() { }
    public void restart() { }
}
```

**2. Closed for Modification:**
```java
// Client code doesn't change when subsystem changes
public class Client {
    public void work() {
        facade.startComputer(); // Same call
        // Even if subsystem implementation changes
    }
}
```

**3. Easy to Extend Subsystem:**
```java
public class ComputerFacade {
    // Add new subsystem component
    private SecuritySystem security;
    
    public void startComputer() {
        // Existing code
        powerSupply.on();
        cpu.initialize();
        
        // Add new functionality
        security.runSecurityCheck();
    }
}
```

---

### Q10: What's the relationship between Facade Pattern and Microservices?

**Answer:**

**Facade Pattern is very relevant in Microservices:**

**API Gateway as Facade:**
```
┌─────────────┐
│   Client    │
└─────────────┘
       │
       ▼
┌─────────────┐
│ API Gateway │ ← Facade
└─────────────┘
       │
       ├──────────┬──────────┬──────────┐
       ▼          ▼          ▼          ▼
┌──────────┐┌──────────┐┌──────────┐┌──────────┐
│  User    ││  Order   ││ Payment  ││ Shipping │
│ Service  ││ Service  ││ Service  ││ Service  │
└──────────┘└──────────┘└──────────┘└──────────┘
```

**Benefits:**
- Single entry point for clients
- Hides microservice complexity
- Can aggregate multiple service calls
- Handles authentication, routing, load balancing

**Example:**
```java
@RestController
public class APIGatewayFacade {
    
    @Autowired
    private UserService userService;
    
    @Autowired
    private OrderService orderService;
    
    @Autowired
    private PaymentService paymentService;
    
    @PostMapping("/api/checkout")
    public Response checkout(CheckoutRequest request) {
        // Facade coordinates multiple microservices
        User user = userService.getUser(request.getUserId());
        Order order = orderService.createOrder(request.getItems());
        Payment payment = paymentService.processPayment(order);
        
        return new Response(order, payment);
    }
}
```

---

## 🎓 Summary

The **Facade Design Pattern** is essential for:
- Simplifying complex subsystems
- Providing easy-to-use interfaces
- Decoupling clients from implementation details
- Following the Principle of Least Knowledge

**Remember:**
- Facade **hides complexity**
- Provides **unified interface**
- Client knows **only about facade**
- Supports **loose coupling**

**Real-world analogy:** Just like pressing a single power button starts your computer (hiding all the complex boot processes), the Facade pattern provides a simple interface to complex subsystems! 💻

---
