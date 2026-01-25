# LLD-22: Chain of Responsibility Design Pattern (Behavioral Pattern)

## 📚 Table of Contents
1. [Introduction](#introduction)
2. [What is Chain of Responsibility Pattern?](#what-is-chain-of-responsibility-pattern)
3. [Core Concept](#core-concept)
4. [Classic Example: ATM Money Dispenser](#classic-example-atm-money-dispenser)
5. [Design Structure](#design-structure)
6. [UML Diagrams](#uml-diagrams)
7. [Implementation in Java](#implementation-in-java)
8. [Standard UML Diagram](#standard-uml-diagram)
9. [Chain of Responsibility vs Linked List](#chain-of-responsibility-vs-linked-list)
10. [Key Concepts](#key-concepts)
11. [Real-World Use Cases](#real-world-use-cases)
12. [Interview Problems](#interview-problems)
13. [Key Takeaways](#key-takeaways)
14. [Interview Q&A](#interview-qa)

---

## 🎯 Introduction

Welcome back to our LLD series! Today we're learning about the **Chain of Responsibility Design Pattern**.

### Pattern Context

This is another **use-case specific** design pattern. Many LLD interview problems can be solved elegantly using this pattern. We'll discuss:
- The core pattern concept
- Classic ATM example
- Real interview problems that use this pattern

Let's dive in!

---

## 🤔 What is Chain of Responsibility Pattern?

**Simple Definition**: A pattern where a request passes through a **chain of handlers**, and each handler decides whether to process the request or pass it to the next handler.

**Core Idea**: 
- You have a **Client** that sends a request
- The request goes through a **chain of receivers** (R1 → R2 → R3 → ...)
- Each receiver checks: "Can I handle this?"
  - **Yes** → Process and respond
  - **No** → Pass to the next receiver

### The Basic Flow

```
Client → Request → R1 → Request → R2 → Request → R3
                    ↓              ↓              ↓
                 Check          Check          Check
                    ↓              ↓              ↓
Client ← Response ← R1 ← Response ← R2 ← Response ← R3
```

**Key Point**: Receivers are linked like a **chain** (similar to linked list structure).

---

## 🌳 Core Concept

### The Chain Structure

**Receivers are connected**:
- R1 has a reference to R2
- R2 has a reference to R3
- R3 has a reference to R4 (and so on...)

**Request Flow**:
1. Client sends request to R1
2. R1 checks: "Can I fulfill this request?"
   - **Yes** → Process and return response
   - **No** → Forward to R2
3. R2 checks: "Can I fulfill this request?"
   - **Yes** → Process and return response
   - **No** → Forward to R3
4. Continue until request is handled or chain ends

**Response Flow**:
- Responses travel back through the chain (recursively)
- Like a recursive tree execution

### Why Use This Pattern?

**Benefits**:
1. **Decoupling**: Client doesn't know which handler will process the request
2. **Flexibility**: Easy to add/remove handlers
3. **Single Responsibility**: Each handler has one job
4. **Dynamic Chain**: Can change the chain at runtime

---

## 💰 Classic Example: ATM Money Dispenser

This is the **standard example** for understanding Chain of Responsibility.

### Problem Statement

**Scenario**: You go to an ATM to withdraw money.

**Inside the ATM**:
- Different **handlers** for different denominations
- ₹1000 notes in one section
- ₹500 notes in another section
- ₹200 notes in another section
- ₹100 notes in another section

**Why separate handlers?**
- Different note sizes
- Different validation logic
- Limited quantity per denomination
- Need to dispense largest notes first

### How It Works

**Example 1**: Withdraw ₹3000

```
Request: ₹3000
↓
₹1000 Handler (has 5 notes)
  - Can fulfill? Yes! (3 notes available)
  - Dispense: 3 × ₹1000
  - Remaining: ₹0
  - Response: ₹3000 dispensed ✓
```

**Example 2**: Withdraw ₹6000

```
Request: ₹6000
↓
₹1000 Handler (has 5 notes)
  - Can fulfill completely? No
  - Dispense: 5 × ₹1000 = ₹5000
  - Remaining: ₹1000
  - Forward to next handler
    ↓
    ₹500 Handler (has 10 notes)
      - Can fulfill ₹1000? Yes! (2 notes)
      - Dispense: 2 × ₹500
      - Remaining: ₹0
      - Response: ₹6000 dispensed ✓
```

**Example 3**: Withdraw ₹150

```
Request: ₹150
↓
₹1000 Handler → Can't handle → Forward
↓
₹500 Handler → Can't handle → Forward
↓
₹100 Handler (has 20 notes)
  - Can fulfill ₹150? Partially (₹100 only)
  - Options:
    1. Dispense ₹100, fail remaining ₹50
    2. Fail entire transaction
  - Decision depends on business logic
```

### The Chain

```
Client
  ↓
₹1000 Handler → ₹500 Handler → ₹200 Handler → ₹100 Handler → null
```

**Each handler**:
- Tries to fulfill the request
- Dispenses what it can
- Forwards remaining amount to next handler

---

## 🏗️ Design Structure

### Initial Approach (Tightly Coupled)

**Problem**: If we hardcode references between concrete handlers:

```
1000Handler → 500Handler → 200Handler → 100Handler
```

**Issues**:
- What if ₹500 notes are demonetized?
- Need to change code in 1000Handler
- Tight coupling between handlers

### Better Approach (Loosely Coupled)

**Solution**: Use abstraction!
<img width="500" height="425" alt="image" src="https://github.com/user-attachments/assets/d7dce8a2-6a4d-44b8-93f8-fbc5ef035efb" />


```
┌─────────────────────┐
│  MoneyHandler       │ (Abstract)
├─────────────────────┤
│ - nextHandler       │ ← Reference to itself
│ + setNextHandler()  │
│ + dispense()        │ ← Abstract
└─────────────────────┘
         △
         │
    ┌────┴────┐
    │         │
┌───▽────┐  ┌▽────────┐
│1000    │  │500      │
│Handler │  │Handler  │
└────────┘  └─────────┘
```

**Key Design Decisions**:

1. **Abstract Handler Class**
   - Has a reference to `MoneyHandler` (itself)
   - Provides `setNextHandler()` method
   - Declares abstract `dispense()` method

2. **Concrete Handlers**
   - Inherit from `MoneyHandler`
   - Implement `dispense()` logic
   - Don't know about other handlers

3. **Client Sets the Chain**
   - Client decides the order
   - Can change dynamically
   - Handlers remain independent

---

## 📊 UML Diagrams

### Class Diagram
<img width="488" height="351" alt="image" src="https://github.com/user-attachments/assets/d116735c-d029-44df-8168-844a75d2fca4" />
<img width="540" height="201" alt="image" src="https://github.com/user-attachments/assets/93e4ea0c-6205-4c04-b5fe-0ab1dc7ae8a2" />


```
┌─────────────────────────────┐
│   MoneyHandler (Abstract)   │
├─────────────────────────────┤
│ # nextHandler: MoneyHandler │ ← Self-reference
├─────────────────────────────┤
│ + MoneyHandler()            │
│ + setNextHandler(handler)   │
│ + dispense(amount): void    │ ← Abstract
└─────────────────────────────┘
              △
              │
      ┌───────┴────────┬────────────┐
      │                │            │
┌─────▽──────┐  ┌──────▽─────┐  ┌──▽─────────┐
│1000Handler │  │500Handler  │  │100Handler  │
├────────────┤  ├────────────┤  ├────────────┤
│- numNotes  │  │- numNotes  │  │- numNotes  │
├────────────┤  ├────────────┤  ├────────────┤
│+ dispense()│  │+ dispense()│  │+ dispense()│
└────────────┘  └────────────┘  └────────────┘
```

**Relationships**:
- **Inheritance**: All handlers inherit from `MoneyHandler`
- **Self-Reference**: `MoneyHandler` has reference to `MoneyHandler` (next)
- **Client**: Holds reference to first handler

### Sequence Diagram: Withdraw ₹6000

```
Client    1000Handler    500Handler    100Handler
  │            │              │             │
  │─dispense(6000)→          │             │
  │            │              │             │
  │            │─Check: Can handle?         │
  │            │  (5 notes available)       │
  │            │─Dispense: 5×₹1000          │
  │            │─Remaining: ₹1000           │
  │            │              │             │
  │            │─dispense(1000)→            │
  │            │              │             │
  │            │              │─Check: Can handle?
  │            │              │  (2 notes needed)
  │            │              │─Dispense: 2×₹500
  │            │              │─Remaining: ₹0
  │            │              │             │
  │            │←─────────────│             │
  │←───────────│              │             │
  │            │              │             │
```

---

## 💻 Implementation in Java

### Complete Working Code

```java
// Abstract Handler
abstract class MoneyHandler {
    protected MoneyHandler nextHandler; // Self-reference
    
    public MoneyHandler() {
        this.nextHandler = null;
    }
    
    public void setNextHandler(MoneyHandler next) {
        this.nextHandler = next;
    }
    
    // Abstract method - each handler implements differently
    public abstract void dispense(int amount);
}

// Concrete Handler 1: ₹1000 Notes
class Handler1000 extends MoneyHandler {
    private int numNotes;
    
    public Handler1000(int numNotes) {
        this.numNotes = numNotes;
    }
    
    @Override
    public void dispense(int amount) {
        // Calculate notes needed
        int notesNeeded = amount / 1000;
        
        // Check if we have enough notes
        if (notesNeeded > numNotes) {
            notesNeeded = numNotes; // Dispense what we have
            numNotes = 0;
        } else {
            numNotes -= notesNeeded; // Reduce available notes
        }
        
        // Dispense notes
        if (notesNeeded > 0) {
            System.out.println("Dispensing " + notesNeeded + " x ₹1000 notes");
        }
        
        // Calculate remaining amount
        int remaining = amount - (notesNeeded * 1000);
        
        // Forward to next handler if needed
        if (remaining > 0) {
            if (nextHandler != null) {
                nextHandler.dispense(remaining);
            } else {
                System.out.println("Error: Remaining amount of ₹" + 
                    remaining + " cannot be fulfilled. Insufficient funds.");
            }
        }
    }
}

// Concrete Handler 2: ₹500 Notes
class Handler500 extends MoneyHandler {
    private int numNotes;
    
    public Handler500(int numNotes) {
        this.numNotes = numNotes;
    }
    
    @Override
    public void dispense(int amount) {
        int notesNeeded = amount / 500;
        
        if (notesNeeded > numNotes) {
            notesNeeded = numNotes;
            numNotes = 0;
        } else {
            numNotes -= notesNeeded;
        }
        
        if (notesNeeded > 0) {
            System.out.println("Dispensing " + notesNeeded + " x ₹500 notes");
        }
        
        int remaining = amount - (notesNeeded * 500);
        
        if (remaining > 0) {
            if (nextHandler != null) {
                nextHandler.dispense(remaining);
            } else {
                System.out.println("Error: Remaining amount of ₹" + 
                    remaining + " cannot be fulfilled.");
            }
        }
    }
}

// Concrete Handler 3: ₹200 Notes
class Handler200 extends MoneyHandler {
    private int numNotes;
    
    public Handler200(int numNotes) {
        this.numNotes = numNotes;
    }
    
    @Override
    public void dispense(int amount) {
        int notesNeeded = amount / 200;
        
        if (notesNeeded > numNotes) {
            notesNeeded = numNotes;
            numNotes = 0;
        } else {
            numNotes -= notesNeeded;
        }
        
        if (notesNeeded > 0) {
            System.out.println("Dispensing " + notesNeeded + " x ₹200 notes");
        }
        
        int remaining = amount - (notesNeeded * 200);
        
        if (remaining > 0) {
            if (nextHandler != null) {
                nextHandler.dispense(remaining);
            } else {
                System.out.println("Error: Remaining amount of ₹" + 
                    remaining + " cannot be fulfilled.");
            }
        }
    }
}

// Concrete Handler 4: ₹100 Notes
class Handler100 extends MoneyHandler {
    private int numNotes;
    
    public Handler100(int numNotes) {
        this.numNotes = numNotes;
    }
    
    @Override
    public void dispense(int amount) {
        int notesNeeded = amount / 100;
        
        if (notesNeeded > numNotes) {
            notesNeeded = numNotes;
            numNotes = 0;
        } else {
            numNotes -= notesNeeded;
        }
        
        if (notesNeeded > 0) {
            System.out.println("Dispensing " + notesNeeded + " x ₹100 notes");
        }
        
        int remaining = amount - (notesNeeded * 100);
        
        if (remaining > 0) {
            System.out.println("Error: Remaining amount of ₹" + 
                remaining + " cannot be fulfilled.");
        }
    }
}

// Client Code
public class ATMDispenser {
    public static void main(String[] args) {
        // Create handlers with available notes
        MoneyHandler h1000 = new Handler1000(3);  // 3 notes of ₹1000
        MoneyHandler h500 = new Handler500(5);    // 5 notes of ₹500
        MoneyHandler h200 = new Handler200(10);   // 10 notes of ₹200
        MoneyHandler h100 = new Handler100(20);   // 20 notes of ₹100
        
        // Set up the chain
        h1000.setNextHandler(h500);
        h500.setNextHandler(h200);
        h200.setNextHandler(h100);
        
        // Test Case 1: Withdraw ₹300
        System.out.println("=== Withdrawing ₹300 ===");
        h1000.dispense(300);
        
        System.out.println("\n=== Withdrawing ₹3000 ===");
        h1000.dispense(3000);
        
        System.out.println("\n=== Withdrawing ₹4000 ===");
        h1000.dispense(4000);
    }
}
```

### Expected Output

```
=== Withdrawing ₹300 ===
Dispensing 1 x ₹200 notes
Dispensing 1 x ₹100 notes

=== Withdrawing ₹3000 ===
Dispensing 3 x ₹1000 notes

=== Withdrawing ₹4000 ===
Dispensing 3 x ₹1000 notes
Dispensing 2 x ₹500 notes
```

### Code Walkthrough

#### 1. Abstract Handler Class

```java
abstract class MoneyHandler {
    protected MoneyHandler nextHandler; // Self-reference
    
    public void setNextHandler(MoneyHandler next) {
        this.nextHandler = next;
    }
    
    public abstract void dispense(int amount);
}
```

**Key Points**:
- `nextHandler` is **protected** (accessible to subclasses)
- `setNextHandler()` allows dynamic chain building
- `dispense()` is **abstract** (each handler implements differently)

#### 2. Concrete Handler Logic

```java
public void dispense(int amount) {
    // 1. Calculate notes needed
    int notesNeeded = amount / denomination;
    
    // 2. Check availability
    if (notesNeeded > numNotes) {
        notesNeeded = numNotes; // Dispense what we have
    }
    
    // 3. Dispense notes
    if (notesNeeded > 0) {
        System.out.println("Dispensing...");
    }
    
    // 4. Calculate remaining
    int remaining = amount - (notesNeeded * denomination);
    
    // 5. Forward to next handler
    if (remaining > 0 && nextHandler != null) {
        nextHandler.dispense(remaining);
    }
}
```

**Flow**:
1. Calculate how many notes needed
2. Check if we have enough
3. Dispense available notes
4. Calculate remaining amount
5. Forward to next handler if needed

#### 3. Setting Up the Chain

```java
// Create handlers
MoneyHandler h1000 = new Handler1000(3);
MoneyHandler h500 = new Handler500(5);
MoneyHandler h200 = new Handler200(10);
MoneyHandler h100 = new Handler100(20);

// Link them
h1000.setNextHandler(h500);
h500.setNextHandler(h200);
h200.setNextHandler(h100);
// h100.nextHandler = null (default)
```

**Chain**: 1000 → 500 → 200 → 100 → null

#### 4. Client Usage

```java
// Client only knows the first handler
MoneyHandler atm = h1000;

// Make request
atm.dispense(300);
```

**Client doesn't know**:
- How many handlers exist
- Which handler will process the request
- Internal chain structure

---

## 📐 Standard UML Diagram

### Generic Chain of Responsibility Structure

```
┌─────────────────────────┐
│      IHandler           │
├─────────────────────────┤
│ - next: IHandler        │ ← Self-reference
├─────────────────────────┤
│ + handleRequest()       │
└─────────────────────────┘
            △
            │
      ┌─────┴──────┐
      │            │
┌─────▽──────┐  ┌──▽──────────┐
│ConcreteH1  │  │ConcreteH2   │
├────────────┤  ├─────────────┤
│+ handleReq │  │+ handleReq  │
└────────────┘  └─────────────┘

     Client
       │
       │ uses
       ▽
    IHandler
```

**Components**:

1. **IHandler (Abstract)**
   - Declares `handleRequest()` method
   - Has reference to next handler
   - May provide default implementation

2. **ConcreteHandler1, ConcreteHandler2**
   - Implement `handleRequest()`
   - Decide: process or forward

3. **Client**
   - Sends request to first handler
   - Doesn't know which handler will process

**Flow**:
```
Client → ConcreteH1.handleRequest()
           ↓
         Can handle?
           ↓
      Yes → Process
      No  → next.handleRequest()
              ↓
            ConcreteH2.handleRequest()
              ↓
            Can handle?
              ↓
            Yes → Process
            No  → next.handleRequest()
```

---

## 🔗 Chain of Responsibility vs Linked List

### Similarities

Both have a **chain structure**:
- Nodes/Handlers linked together
- Each has reference to next
- Traversal through the chain

### Key Differences

| Aspect | Linked List | Chain of Responsibility |
|--------|-------------|-------------------------|
| **Purpose** | Data structure | Design pattern |
| **Intent** | Store data | Handle requests |
| **Node Type** | Same class | Different classes (polymorphic) |
| **Reference** | Same type (Node → Node) | Interface type (IHandler → IHandler) |
| **Behavior** | Store/retrieve data | Process/forward requests |
| **Traversal** | Linear iteration | Conditional forwarding |

### Detailed Comparison

**Linked List**:
```java
class Node {
    int data;
    Node next; // Same type reference
}

Node n1 = new Node(1);
Node n2 = new Node(2);
n1.next = n2; // Same class objects
```

**Chain of Responsibility**:
```java
interface IHandler {
    IHandler next; // Interface reference
    void handle();
}

class Handler1000 implements IHandler { ... }
class Handler500 implements IHandler { ... }

IHandler h1 = new Handler1000();
IHandler h2 = new Handler500();
h1.next = h2; // Different class objects!
```

**Key Insight**:
- **Linked List**: All nodes are same class
- **Chain of Responsibility**: Handlers are different classes implementing same interface

**Similar to Composite Pattern**:
- Composite Pattern → Tree structure
- Chain of Responsibility → Linked List structure
- Both use polymorphism with data structures

---

## 🔑 Key Concepts

### 1. Self-Reference in Abstract Class

**Core Technique**: Abstract class has reference to itself

```java
abstract class Handler {
    protected Handler nextHandler; // Self-reference!
    
    public void setNext(Handler next) {
        this.nextHandler = next;
    }
}
```

**Why?**
- Allows any subclass to reference any other subclass
- Loose coupling between handlers
- Dynamic chain building

### 2. Request Processing Logic

**Each handler follows this pattern**:

```java
public void handleRequest(Request req) {
    if (canHandle(req)) {
        // Process the request
        process(req);
    } else if (nextHandler != null) {
        // Forward to next handler
        nextHandler.handleRequest(req);
    } else {
        // End of chain - handle failure
        handleFailure(req);
    }
}
```

### 3. SOLID Principles Applied

**Single Responsibility Principle (SRP)**:
- Each handler has ONE job
- 1000Handler only handles ₹1000 notes
- 500Handler only handles ₹500 notes

**Example**: If ₹500 notes change (demonetization):
- Only modify Handler500
- Other handlers remain unchanged

**Open-Closed Principle (OCP)**:
- Open for extension: Add new handlers
- Closed for modification: Don't change existing handlers

**Example**: Add ₹50 handler:
```java
class Handler50 extends MoneyHandler {
    // New handler - no changes to existing code!
}

// Just update the chain
h100.setNextHandler(new Handler50(10));
```

### 4. Why Not If-Else?

**Bad Approach**:
```java
class MoneyDispenser {
    public void dispense(int amount) {
        if (amount >= 1000) {
            // Handle 1000
        } else if (amount >= 500) {
            // Handle 500
        } else if (amount >= 200) {
            // Handle 200
        } else if (amount >= 100) {
            // Handle 100
        }
    }
}
```

**Problems**:
1. ❌ Violates OCP (need to modify for new denominations)
2. ❌ Violates SRP (one class does everything)
3. ❌ Hard to test individual handlers
4. ❌ Tight coupling

**Chain of Responsibility Approach**:
```java
// Separate handlers
class Handler1000 extends MoneyHandler { ... }
class Handler500 extends MoneyHandler { ... }

// Easy to add new handler
class Handler50 extends MoneyHandler { ... }
```

**Benefits**:
1. ✅ Follows OCP (add handlers without modifying existing)
2. ✅ Follows SRP (each handler has one job)
3. ✅ Easy to test
4. ✅ Loose coupling

---

## 🌍 Real-World Use Cases

### 1. Logger System ⭐

**Problem**: Design a logging system with different log levels.

**Log Levels** (hierarchy):
1. **INFO** (Level 1) - Informational messages
2. **DEBUG** (Level 2) - Debugging information
3. **ERROR** (Level 3) - Error messages

**Design**:
```java
abstract class Logger {
    protected Logger nextLogger;
    protected int level;
    
    public void setNext(Logger next) {
        this.nextLogger = next;
    }
    
    public void log(int level, String message) {
        if (this.level <= level) {
            write(message);
        }
        if (nextLogger != null) {
            nextLogger.log(level, message);
        }
    }
    
    protected abstract void write(String message);
}

class InfoLogger extends Logger {
    public InfoLogger() {
        this.level = 1;
    }
    
    protected void write(String message) {
        System.out.println("INFO: " + message);
    }
}

class DebugLogger extends Logger {
    public DebugLogger() {
        this.level = 2;
    }
    
    protected void write(String message) {
        System.out.println("DEBUG: " + message);
    }
}

class ErrorLogger extends Logger {
    public ErrorLogger() {
        this.level = 3;
    }
    
    protected void write(String message) {
        System.err.println("ERROR: " + message);
    }
}
```

**Usage**:
```java
// Set up chain
Logger info = new InfoLogger();
Logger debug = new DebugLogger();
Logger error = new ErrorLogger();

info.setNext(debug);
debug.setNext(error);

// Log messages
info.log(1, "Application started"); // INFO only
info.log(2, "Variable value: 42");  // INFO + DEBUG
info.log(3, "Null pointer!");       // INFO + DEBUG + ERROR
```

**This is a STANDARD interview question!**

### 2. Leave Request System ⭐ (Homework)

**Problem**: Design a leave approval system with hierarchy.

**Hierarchy**:
1. **Team Lead** - Can approve ≤ 2 days
2. **Manager** - Can approve ≤ 5 days
3. **Director** - Can approve ≤ 10 days
4. **Reject** - If > 10 days

**Design Structure**:
```
Employee → TeamLead → Manager → Director → null
```

**Flow**:
```
Employee requests 3 days leave
↓
TeamLead: Can I approve? (≤2 days) → No
↓
Manager: Can I approve? (≤5 days) → Yes! Approved ✓
```

**Implementation Hint**:
```java
abstract class LeaveHandler {
    protected LeaveHandler nextHandler;
    protected int maxDays;
    
    public void approveLeave(int days) {
        if (days <= maxDays) {
            System.out.println("Approved by " + getRole());
        } else if (nextHandler != null) {
            nextHandler.approveLeave(days);
        } else {
            System.out.println("Leave rejected - too many days");
        }
    }
    
    protected abstract String getRole();
}

class TeamLead extends LeaveHandler {
    public TeamLead() { maxDays = 2; }
    protected String getRole() { return "Team Lead"; }
}

class Manager extends LeaveHandler {
    public Manager() { maxDays = 5; }
    protected String getRole() { return "Manager"; }
}

class Director extends LeaveHandler {
    public Director() { maxDays = 10; }
    protected String getRole() { return "Director"; }
}
```

**Try implementing this yourself!** 🚀

### 3. Other Real-World Examples

**Event Handling (GUI)**:
```
Button Click → Button Handler → Panel Handler → Window Handler
```

**Middleware in Web Frameworks**:
```
Request → Authentication → Logging → Validation → Controller
```

**Exception Handling**:
```
try-catch blocks form a chain of exception handlers
```

**Servlet Filters**:
```
HTTP Request → Filter1 → Filter2 → Filter3 → Servlet
```

---

## 🎯 Interview Problems

### Problem 1: Logger System

**Question**: Design a logger system with INFO, DEBUG, and ERROR levels.

**Solution**: Use Chain of Responsibility with log levels.

**Key Points**:
- Each logger has a level
- Logs at or above its level
- Forwards to next logger in chain

### Problem 2: Leave Request System

**Question**: Design a leave approval system with Team Lead, Manager, and Director.

**Solution**: Chain of approvers with different approval limits.

**Key Points**:
- Each approver has max days limit
- Approves if within limit
- Forwards to next approver otherwise

### Problem 3: Support Ticket System

**Question**: Design a customer support system with L1, L2, L3 support.

**Solution**: Chain of support levels.

**Flow**:
```
Ticket → L1 Support (basic issues)
       → L2 Support (technical issues)
       → L3 Support (critical issues)
       → Escalate to engineering
```

### Problem 4: Discount Calculator

**Question**: Apply discounts based on customer type (Regular, Premium, VIP).

**Solution**: Chain of discount handlers.

**Example**:
```
Price: ₹1000
↓
Regular: 5% off → ₹950
↓
Premium: Additional 10% off → ₹855
↓
VIP: Additional 15% off → ₹726.75
```

### Common Interview Questions

**When to use Chain of Responsibility?**
- Multiple handlers for a request
- Handler determined at runtime
- Hierarchical processing
- Request can be handled by multiple handlers

**Identify the pattern**:
- "Design a system with multiple levels of approval"
- "Create a logging system with different levels"
- "Implement request processing pipeline"
- "Build a middleware system"

→ Think Chain of Responsibility!

---

## 🎓 Key Takeaways

### When to Use Chain of Responsibility

✅ **Use when**:
1. **Multiple handlers** can process a request
2. **Handler is unknown** at compile time
3. **Hierarchical processing** is needed
4. **Decoupling** sender from receiver
5. **Dynamic chain** configuration required

❌ **Don't use when**:
1. Only one handler exists
2. Handler is known at compile time
3. No hierarchy or chain needed
4. Performance is critical (chain traversal overhead)

### Advantages

1. **Reduced Coupling**
   - Client doesn't know which handler processes request
   - Handlers don't know about each other

2. **Flexibility**
   - Add/remove handlers dynamically
   - Change order of handlers

3. **Single Responsibility**
   - Each handler has one job
   - Easy to maintain and test

4. **Open-Closed Principle**
   - Add new handlers without modifying existing code

### Disadvantages

1. **No Guarantee of Handling**
   - Request might reach end of chain unhandled
   - Need to handle this case

2. **Debugging Difficulty**
   - Hard to trace which handler processed request
   - Flow goes through multiple objects

3. **Performance Overhead**
   - Request traverses through chain
   - Multiple method calls

### Design Principles Applied

1. **Single Responsibility Principle (SRP)** ✅
   - Each handler has one responsibility

2. **Open-Closed Principle (OCP)** ✅
   - Open for extension (add handlers)
   - Closed for modification (don't change existing)

3. **Dependency Inversion Principle (DIP)** ✅
   - Depend on abstraction (IHandler)
   - Not on concrete handlers

---

## ❓ Interview Q&A

### Q1: What is the Chain of Responsibility Design Pattern? Explain with an example.

**Answer**:

The Chain of Responsibility pattern allows a request to pass through a **chain of handlers**, where each handler decides whether to process the request or forward it to the next handler.

**Key Components**:
1. **Handler Interface**: Declares method to handle requests
2. **Concrete Handlers**: Implement handling logic
3. **Chain**: Handlers linked together
4. **Client**: Sends request to first handler

**Example**: ATM Money Dispenser

```java
abstract class MoneyHandler {
    protected MoneyHandler nextHandler;
    
    public void setNext(MoneyHandler next) {
        this.nextHandler = next;
    }
    
    public abstract void dispense(int amount);
}

class Handler1000 extends MoneyHandler {
    public void dispense(int amount) {
        if (amount >= 1000) {
            int notes = amount / 1000;
            System.out.println("Dispensing " + notes + " x ₹1000");
            int remaining = amount % 1000;
            if (remaining > 0 && nextHandler != null) {
                nextHandler.dispense(remaining);
            }
        } else if (nextHandler != null) {
            nextHandler.dispense(amount);
        }
    }
}
```

**Usage**:
```java
MoneyHandler h1000 = new Handler1000();
MoneyHandler h500 = new Handler500();
h1000.setNext(h500);

h1000.dispense(1500); // 1×₹1000 + 1×₹500
```

---

### Q2: How is Chain of Responsibility different from a Linked List?

**Answer**:

While both have a chain structure, they serve different purposes.

| Aspect | Linked List | Chain of Responsibility |
|--------|-------------|-------------------------|
| **Purpose** | Data structure | Design pattern |
| **Intent** | Store data | Handle requests |
| **Node Type** | Same class | Different classes |
| **Reference** | Node → Node | IHandler → IHandler |
| **Traversal** | Linear | Conditional |

**Linked List**:
```java
class Node {
    int data;
    Node next; // Same type
}
```
- All nodes are same class
- Stores data
- Linear traversal

**Chain of Responsibility**:
```java
abstract class Handler {
    Handler next; // Same interface, different implementations
}

class Handler1000 extends Handler { }
class Handler500 extends Handler { }
```
- Handlers are different classes
- Processes requests
- Conditional forwarding

**Key Difference**: 
- Linked List: Homogeneous (same type)
- Chain of Responsibility: Polymorphic (different types, same interface)

**Similar to Composite Pattern**:
- Composite → Tree structure
- Chain of Responsibility → Linked List structure

---

### Q3: Why use Chain of Responsibility instead of if-else statements?

**Answer**:

**If-Else Approach**:
```java
class MoneyDispenser {
    public void dispense(int amount) {
        if (amount >= 1000) {
            // Handle 1000
        } else if (amount >= 500) {
            // Handle 500
        } else if (amount >= 100) {
            // Handle 100
        }
    }
}
```

**Problems**:
1. ❌ **Violates OCP**: Need to modify code to add new denomination
2. ❌ **Violates SRP**: One class does everything
3. ❌ **Tight Coupling**: All logic in one place
4. ❌ **Hard to Test**: Can't test handlers independently
5. ❌ **Not Flexible**: Can't change order dynamically

**Chain of Responsibility Approach**:
```java
abstract class MoneyHandler {
    protected MoneyHandler next;
    public abstract void dispense(int amount);
}

class Handler1000 extends MoneyHandler { }
class Handler500 extends MoneyHandler { }
```

**Benefits**:
1. ✅ **Follows OCP**: Add new handler without modifying existing
2. ✅ **Follows SRP**: Each handler has one job
3. ✅ **Loose Coupling**: Handlers independent
4. ✅ **Easy to Test**: Test each handler separately
5. ✅ **Flexible**: Change chain dynamically

**Example**: Add ₹50 handler
```java
// Just create new class
class Handler50 extends MoneyHandler { }

// Update chain
h100.setNext(new Handler50());
// No changes to existing handlers!
```

---

### Q4: How does Chain of Responsibility follow SOLID principles?

**Answer**:

**1. Single Responsibility Principle (SRP)** ✅

Each handler has ONE responsibility:
```java
class Handler1000 extends MoneyHandler {
    // Only handles ₹1000 notes
}

class Handler500 extends MoneyHandler {
    // Only handles ₹500 notes
}
```

**Benefit**: If ₹500 notes change (demonetization), only modify Handler500.

**2. Open-Closed Principle (OCP)** ✅

Open for extension, closed for modification:
```java
// Add new handler - NO changes to existing code
class Handler50 extends MoneyHandler {
    public void dispense(int amount) { ... }
}

// Just update the chain
h100.setNext(new Handler50());
```

**3. Liskov Substitution Principle (LSP)** ✅

Any handler can replace another:
```java
MoneyHandler handler = new Handler1000(); // or Handler500
handler.dispense(amount); // Works with any handler
```

**4. Interface Segregation Principle (ISP)** ✅

Handlers only implement what they need:
```java
abstract class MoneyHandler {
    public abstract void dispense(int amount);
    // Only one method - no unnecessary methods
}
```

**5. Dependency Inversion Principle (DIP)** ✅

Depend on abstraction, not concrete classes:
```java
abstract class MoneyHandler {
    protected MoneyHandler next; // Abstraction, not concrete
}
```

**Overall**: Chain of Responsibility is a SOLID-friendly pattern!

---

### Q5: What are the advantages and disadvantages of Chain of Responsibility?

**Answer**:

**Advantages** ✅:

1. **Reduced Coupling**
   - Client doesn't know which handler processes request
   - Handlers don't know about each other

2. **Flexibility**
   - Add/remove handlers dynamically
   - Change order at runtime

3. **Single Responsibility**
   - Each handler has one job
   - Easy to maintain

4. **Open-Closed Principle**
   - Add new handlers without modifying existing

5. **Simplified Client**
   - Client only knows first handler
   - No complex routing logic

**Disadvantages** ❌:

1. **No Guarantee of Handling**
   - Request might not be handled
   - Need to handle end-of-chain case

2. **Debugging Difficulty**
   - Hard to trace execution flow
   - Request goes through multiple handlers

3. **Performance Overhead**
   - Chain traversal takes time
   - Multiple method calls

4. **Configuration Complexity**
   - Need to set up chain correctly
   - Order matters

**Example of No Guarantee**:
```java
// Request ₹150, but only ₹100 handler exists
h100.dispense(150);
// Dispenses ₹100, remaining ₹50 unhandled
```

**Trade-off**: Flexibility vs. Guaranteed handling

---

### Q6: When should you use Chain of Responsibility pattern?

**Answer**:

✅ **Use when**:

**1. Multiple Handlers**
```java
// Multiple levels of support
L1Support → L2Support → L3Support → Engineering
```

**2. Handler Unknown at Compile Time**
```java
// Don't know which handler will process
request.process(); // Could be any handler in chain
```

**3. Hierarchical Processing**
```java
// Approval hierarchy
TeamLead → Manager → Director
```

**4. Request Can Be Handled by Multiple Handlers**
```java
// Logging - multiple loggers can log same message
InfoLogger → DebugLogger → ErrorLogger
```

**5. Decoupling Sender from Receiver**
```java
// Client doesn't know which handler processes
client.sendRequest(request);
```

❌ **Don't use when**:

**1. Only One Handler**
```java
// No need for chain
processor.process(request);
```

**2. Handler Known at Compile Time**
```java
// Direct call is better
specificHandler.handle(request);
```

**3. Performance Critical**
```java
// Chain traversal adds overhead
// Use direct dispatch instead
```

**4. All Requests Must Be Handled**
```java
// Chain might not handle all requests
// Use Strategy or Command instead
```

**Decision Criteria**:
- Multiple handlers? → Consider Chain of Responsibility
- Hierarchical processing? → Chain of Responsibility
- Dynamic handler selection? → Chain of Responsibility
- Single handler? → Don't use

---

### Q7: How do you implement a Logger System using Chain of Responsibility?

**Answer**:

**Problem**: Design a logging system with INFO, DEBUG, and ERROR levels.

**Solution**:

```java
// Abstract Logger
abstract class Logger {
    protected Logger nextLogger;
    protected int level;
    
    public static final int INFO = 1;
    public static final int DEBUG = 2;
    public static final int ERROR = 3;
    
    public void setNext(Logger next) {
        this.nextLogger = next;
    }
    
    public void log(int level, String message) {
        if (this.level <= level) {
            write(message);
        }
        if (nextLogger != null) {
            nextLogger.log(level, message);
        }
    }
    
    protected abstract void write(String message);
}

// Concrete Loggers
class InfoLogger extends Logger {
    public InfoLogger() {
        this.level = INFO;
    }
    
    protected void write(String message) {
        System.out.println("INFO: " + message);
    }
}

class DebugLogger extends Logger {
    public DebugLogger() {
        this.level = DEBUG;
    }
    
    protected void write(String message) {
        System.out.println("DEBUG: " + message);
    }
}

class ErrorLogger extends Logger {
    public ErrorLogger() {
        this.level = ERROR;
    }
    
    protected void write(String message) {
        System.err.println("ERROR: " + message);
    }
}

// Usage
public class LoggerDemo {
    public static void main(String[] args) {
        // Set up chain
        Logger info = new InfoLogger();
        Logger debug = new DebugLogger();
        Logger error = new ErrorLogger();
        
        info.setNext(debug);
        debug.setNext(error);
        
        // Log messages
        info.log(Logger.INFO, "Application started");
        // Output: INFO: Application started
        
        info.log(Logger.DEBUG, "Variable x = 42");
        // Output: INFO: Variable x = 42
        //         DEBUG: Variable x = 42
        
        info.log(Logger.ERROR, "Null pointer exception");
        // Output: INFO: Null pointer exception
        //         DEBUG: Null pointer exception
        //         ERROR: Null pointer exception
    }
}
```

**Key Points**:
- Each logger has a level
- Logs if message level >= logger level
- Forwards to next logger (all loggers can log)

**This is a STANDARD interview question!**

---

### Q8: Design a Leave Request System using Chain of Responsibility.

**Answer**:

**Problem**: Employee requests leave, approved by Team Lead (≤2 days), Manager (≤5 days), or Director (≤10 days).

**Solution**:

```java
// Abstract Handler
abstract class LeaveHandler {
    protected LeaveHandler nextHandler;
    protected int maxDays;
    protected String role;
    
    public void setNext(LeaveHandler next) {
        this.nextHandler = next;
    }
    
    public void approveLeave(String employee, int days) {
        if (days <= maxDays) {
            System.out.println(role + " approved " + days + 
                " days leave for " + employee);
        } else if (nextHandler != null) {
            System.out.println(role + " cannot approve - forwarding...");
            nextHandler.approveLeave(employee, days);
        } else {
            System.out.println("Leave request rejected - too many days");
        }
    }
}

// Concrete Handlers
class TeamLead extends LeaveHandler {
    public TeamLead() {
        this.maxDays = 2;
        this.role = "Team Lead";
    }
}

class Manager extends LeaveHandler {
    public Manager() {
        this.maxDays = 5;
        this.role = "Manager";
    }
}

class Director extends LeaveHandler {
    public Director() {
        this.maxDays = 10;
        this.role = "Director";
    }
}

// Usage
public class LeaveRequestDemo {
    public static void main(String[] args) {
        // Set up chain
        LeaveHandler teamLead = new TeamLead();
        LeaveHandler manager = new Manager();
        LeaveHandler director = new Director();
        
        teamLead.setNext(manager);
        manager.setNext(director);
        
        // Test cases
        teamLead.approveLeave("John", 1);
        // Output: Team Lead approved 1 days leave for John
        
        teamLead.approveLeave("Jane", 3);
        // Output: Team Lead cannot approve - forwarding...
        //         Manager approved 3 days leave for Jane
        
        teamLead.approveLeave("Bob", 7);
        // Output: Team Lead cannot approve - forwarding...
        //         Manager cannot approve - forwarding...
        //         Director approved 7 days leave for Bob
        
        teamLead.approveLeave("Alice", 15);
        // Output: Team Lead cannot approve - forwarding...
        //         Manager cannot approve - forwarding...
        //         Director cannot approve - forwarding...
        //         Leave request rejected - too many days
    }
}
```

**Flow**:
```
Employee requests 3 days
↓
Team Lead (max 2) → Can't approve → Forward
↓
Manager (max 5) → Can approve → Approved ✓
```

**Try implementing this yourself!** 🚀

---

### Q9: What is the difference between Chain of Responsibility and Composite Pattern?

**Answer**:

Both patterns use tree-like structures, but serve different purposes.

| Aspect | Chain of Responsibility | Composite Pattern |
|--------|------------------------|-------------------|
| **Structure** | Linear chain (linked list) | Tree hierarchy |
| **Intent** | Handle requests | Treat objects uniformly |
| **Processing** | One handler processes | All nodes can process |
| **Traversal** | Sequential (stop when handled) | Recursive (all nodes) |
| **Example** | ATM dispenser | File system |

**Chain of Responsibility**:
```
H1 → H2 → H3 → H4
```
- Linear chain
- Request stops at first handler that can process
- One handler processes

**Composite Pattern**:
```
      Root
     /    \
   Folder File
   /   \
 File  File
```
- Tree structure
- Operation executed on all nodes
- All nodes participate

**Example**:

**Chain of Responsibility** (ATM):
```java
h1000.dispense(3000);
// Only ₹1000 handler processes (3 notes)
// Stops - doesn't go to ₹500 handler
```

**Composite Pattern** (File System):
```java
root.display();
// Displays root
// Recursively displays all folders and files
// All nodes participate
```

**Key Difference**:
- **Chain**: Stop at first match
- **Composite**: Process all nodes

---

### Q10: How do you handle the case when no handler can process the request?

**Answer**:

**Problem**: Request reaches end of chain without being handled.

**Solutions**:

**1. Check for Null in Last Handler**
```java
class Handler100 extends MoneyHandler {
    public void dispense(int amount) {
        // ... dispense logic ...
        
        int remaining = amount % 100;
        if (remaining > 0) {
            if (nextHandler != null) {
                nextHandler.dispense(remaining);
            } else {
                // End of chain - handle failure
                System.out.println("Cannot dispense ₹" + remaining);
            }
        }
    }
}
```

**2. Default Handler at End**
```java
class DefaultHandler extends MoneyHandler {
    public void dispense(int amount) {
        System.out.println("Cannot fulfill request for ₹" + amount);
        // Or throw exception
        throw new InsufficientFundsException();
    }
}

// Set up chain
h1000.setNext(h500);
h500.setNext(h200);
h200.setNext(h100);
h100.setNext(new DefaultHandler()); // Catch-all
```

**3. Return Boolean**
```java
abstract class MoneyHandler {
    public boolean dispense(int amount) {
        if (canHandle(amount)) {
            process(amount);
            return true;
        } else if (nextHandler != null) {
            return nextHandler.dispense(amount);
        }
        return false; // Not handled
    }
}

// Client checks
if (!handler.dispense(amount)) {
    System.out.println("Request could not be fulfilled");
}
```

**4. Throw Exception**
```java
class Handler100 extends MoneyHandler {
    public void dispense(int amount) {
        // ... logic ...
        
        if (remaining > 0 && nextHandler == null) {
            throw new RequestNotHandledException(
                "Cannot dispense ₹" + remaining
            );
        }
    }
}
```

**Best Practice**: 
- Use Default Handler for graceful failure
- Or return boolean for client to check
- Avoid silent failures

---

### Q11: Can you give real-world examples of Chain of Responsibility?

**Answer**:

**1. Servlet Filters (Java Web)**
```java
Request → AuthFilter → LoggingFilter → ValidationFilter → Servlet
```

**2. Middleware in Express.js**
```javascript
app.use(authMiddleware);
app.use(loggingMiddleware);
app.use(validationMiddleware);
```

**3. Exception Handling**
```java
try {
    // code
} catch (IOException e) {
    // Handle IOException
} catch (SQLException e) {
    // Handle SQLException
} catch (Exception e) {
    // Handle any other exception
}
```

**4. Event Bubbling in DOM**
```
Button → Div → Body → Document
```
- Click event bubbles up
- Each element can handle or pass

**5. Spring Security Filter Chain**
```
Request → SecurityContextFilter → LogoutFilter → 
          AuthenticationFilter → AuthorizationFilter
```

**6. Logging Frameworks (Log4j, SLF4J)**
```
Logger → Appender1 → Appender2 → Appender3
```

**7. ATM Machines**
```
₹2000 → ₹500 → ₹200 → ₹100
```

**8. Customer Support**
```
L1 Support → L2 Support → L3 Support → Engineering
```

---

### Q12: What are common mistakes when implementing Chain of Responsibility?

**Answer**:

**Mistake 1: Not Handling End of Chain**
```java
// ❌ Bad
public void handle(Request req) {
    if (!canHandle(req)) {
        nextHandler.handle(req); // NullPointerException!
    }
}

// ✅ Good
public void handle(Request req) {
    if (!canHandle(req)) {
        if (nextHandler != null) {
            nextHandler.handle(req);
        } else {
            handleFailure(req);
        }
    }
}
```

**Mistake 2: Circular Chain**
```java
// ❌ Bad - creates infinite loop
h1.setNext(h2);
h2.setNext(h3);
h3.setNext(h1); // Circular!

// ✅ Good - linear chain
h1.setNext(h2);
h2.setNext(h3);
h3.setNext(null);
```

**Mistake 3: Tight Coupling**
```java
// ❌ Bad
class Handler1000 {
    private Handler500 next; // Concrete type!
}

// ✅ Good
class Handler1000 {
    private MoneyHandler next; // Abstract type
}
```

**Mistake 4: Not Following SRP**
```java
// ❌ Bad - handler does too much
class Handler1000 {
    public void dispense(int amount) {
        // Dispense 1000
        // Dispense 500
        // Dispense 200
        // All in one handler!
    }
}

// ✅ Good - each handler has one job
class Handler1000 {
    public void dispense(int amount) {
        // Only handle 1000
        // Forward rest to next
    }
}
```

**Mistake 5: Modifying Request**
```java
// ❌ Bad - modifies original request
public void handle(Request req) {
    req.amount -= 1000; // Mutates request!
    nextHandler.handle(req);
}

// ✅ Good - create new request or calculate remaining
public void handle(Request req) {
    int remaining = req.amount - 1000;
    nextHandler.handle(new Request(remaining));
}
```

**Best Practices**:
1. Always check for null before forwarding
2. Avoid circular chains
3. Use abstraction for next handler
4. Follow SRP - one handler, one job
5. Don't mutate requests

---

## 📝 Summary

The **Chain of Responsibility Design Pattern** allows a request to pass through a chain of handlers, where each handler decides whether to process the request or forward it to the next handler.

**Key Points**:
- **Chain Structure**: Handlers linked like a linked list
- **Request Flow**: Client → H1 → H2 → H3 → ... → null
- **Processing**: Each handler checks if it can handle, processes or forwards
- **Decoupling**: Client doesn't know which handler processes

**When to Use**:
- Multiple handlers for a request
- Handler unknown at compile time
- Hierarchical processing
- Dynamic chain configuration

**Real-World Examples**:
- ATM money dispenser (classic example)
- Logger system (standard interview question)
- Leave request system (homework!)
- Servlet filters
- Exception handling

**SOLID Principles**:
- **SRP**: Each handler has one job
- **OCP**: Add handlers without modifying existing
- **DIP**: Depend on abstraction

**Remember**: If you see "multiple levels", "hierarchy", or "approval chain" in a problem, think Chain of Responsibility!

---

**Happy Learning! 🚀**
