# LLD-10: Singleton Design Pattern (Creational Pattern)

## 📚 Table of Contents
1. [Introduction](#introduction)
2. [What is Singleton Design Pattern?](#what-is-singleton-design-pattern)
3. [Memory Fundamentals](#memory-fundamentals)
4. [Basic Implementation](#basic-implementation)
5. [Thread-Safe Singleton](#thread-safe-singleton)
6. [Double-Checked Locking](#double-checked-locking)
7. [Eager Initialization](#eager-initialization)
8. [Implementation in Java](#implementation-in-java)
9. [Real-World Use Cases](#real-world-use-cases)
10. [When NOT to Use Singleton](#when-not-to-use-singleton)
11. [Advantages and Disadvantages](#advantages-and-disadvantages)
12. [Key Takeaways](#key-takeaways)
13. [Interview Q&A](#interview-qa)

---

## 🎯 Introduction

Welcome back to our LLD series! Today we're learning about the **Singleton Design Pattern**.

### Pattern Context

Singleton is:
- The **easiest** design pattern to implement
- The most **basic** design pattern
- **Very commonly used** in real-world applications
- Frequently asked in LLD interviews

Despite being simple, it has important nuances (thread safety, memory management) that we'll explore!

---

## 🤔 What is Singleton Design Pattern?

**Simple Definition**: A Singleton is a **special type of class** that allows only **ONE instance** to be created.

**Core Idea**: 
- You can create only **one object** of the class
- If you try to create another object, it returns the **same first object**
- Multiple objects **cannot exist** for this class

### The Singleton Guarantee

```
First call:  Creates new object → Returns object
Second call: Returns same object (no new creation)
Third call:  Returns same object (no new creation)
...
```

**Key Point**: No matter how many times you try to create an object, you always get the **same instance**.

---

## 💾 Memory Fundamentals

Before implementing Singleton, let's understand how objects are created in memory.

### Two Types of Memory

**1. Stack Memory**
- Stores **primitive data types** (int, char, bool, etc.)
- Stores **references/pointers** to objects
- Fast access
- Limited size

**2. Heap Memory**
- Stores **non-primitive data types** (objects, arrays)
- Stores actual object data
- Larger size
- Slower access

### Object Creation Process

**Code**:
```java
A a = new A();
```

**What happens under the hood?**

**Step 1**: Memory allocation in heap
```
Heap Memory: [  A object allocated  ]
```

**Step 2**: Constructor is called
```
A() constructor executes
- Initializes member variables
- Sets default values
```

**Step 3**: Reference stored in stack
```
Stack Memory: a → points to heap object
```

**Complete Picture**:
```
Stack Memory          Heap Memory
┌─────────┐          ┌──────────────┐
│ a (ref) │─────────→│  A object    │
└─────────┘          │  - int x     │
                     │  - char y    │
                     └──────────────┘
```

### Constructor Basics

**What is a Constructor?**
- Special method called when object is created
- Same name as class
- Initializes object state

**Default Constructor**:
```java
class A {
    private int x;
    private char y;
    
    public A() {  // Constructor
        x = 0;
        y = '\0';
    }
}
```

**If you don't write a constructor**:
- Language provides a **default constructor** automatically
- You don't see it, but it exists

**Key Insight**: When you write `new A()`, you're calling the constructor!

---

## 🏗️ Basic Implementation

### Problem: How to Restrict Object Creation?

**Goal**: Allow only ONE object to be created.

**The Culprit**: The `new` keyword!
```java
A a1 = new A();  // First object
A a2 = new A();  // Second object (we want to prevent this!)
```

### Solution Steps

#### Step 1: Make Constructor Private

**Idea**: If constructor is private, it can't be called from outside!

```java
class Singleton {
    private Singleton() {  // Private constructor
        System.out.println("Singleton constructor called");
    }
}

// Client code
public class Main {
    public static void main(String[] args) {
        Singleton s1 = new Singleton();  // ❌ Error: Constructor is inaccessible!
    }
}
```

**Problem Solved?** Not quite! Now we can't create ANY objects.

#### Step 2: Provide Public Static Method

**Idea**: Provide a public method to get the instance.

```java
class Singleton {
    private Singleton() {
        System.out.println("Singleton constructor called");
    }

    public static Singleton getInstance() {
        return new Singleton();  // Still creates multiple objects!
    }
}
```

**Why Static?**
- To call `getInstance()`, we need: `Singleton.getInstance()`
- **Static methods** belong to the class, not objects
- Can be called without creating an object

**Problem**: Still creates multiple objects each time `getInstance()` is called!

#### Step 3: Store and Reuse Instance

**Idea**: Store the created instance and return it every time.

```java
class Singleton {
    private static Singleton instance = null;  // Stores the single instance
    
    private Singleton() {
        System.out.println("Singleton constructor called");
    }

    public static Singleton getInstance() {
        if (instance == null) {
            instance = new Singleton();  // Create only if not exists
        }
        return instance;  // Return existing instance
    }
}
```

**How It Works**:

**First Call**:
```java
Singleton s1 = Singleton.getInstance();
// instance == null → Create new object
// Constructor called → "Singleton constructor called"
// Return instance
```

**Second Call**:
```java
Singleton s2 = Singleton.getInstance();
// instance != null → Don't create new object
// Return existing instance
// s1 == s2 ✓
```

**Perfect!** This is the basic Singleton implementation.

---

## 🔒 Thread-Safe Singleton

### The Problem: Race Condition

**Scenario**: Multiple threads call `getInstance()` simultaneously.

```
Thread T1                    Thread T2
    │                            │
    ├─ getInstance()             │
    │  if (instance == null)     │
    │      ✓ (true)              │
    │                            ├─ getInstance()
    │                            │  if (instance == null)
    │                            │      ✓ (true)
    │                            │
    ├─ new Singleton()           │
    │  (creates object 1)        │
    │                            ├─ new Singleton()
    │                            │  (creates object 2) ❌
```

**Problem**: Both threads create separate objects! Singleton guarantee broken.

### Solution 1: Simple Locking

**Idea**: Use synchronized keyword to lock the method.

```java
class Singleton {
    private static Singleton instance = null;
    
    private Singleton() {
        System.out.println("Singleton constructor called");
    }

    public static synchronized Singleton getInstance() {
        if (instance == null) {
            instance = new Singleton();
        }
        return instance;
    }
}
```

**How It Works**:
```
Thread T1                    Thread T2
    │                            │
    ├─ getInstance()             │
    │  (T1 acquires lock)        │
    │                            ├─ getInstance()
    │                            │  (T2 waits...)
    │  if (instance == null)     │
    │  new Singleton()           │
    │  (T1 releases lock)        │
    │                            │
    │                            │  (T2 acquires lock)
    │                            │  if (instance == null)
    │                            │      ✗ (false - already created)
    │                            │  return instance
    │                            │  (T2 releases lock)
```

**Problem with Simple Locking**: 
- Locking is an **expensive operation**
- Every call to `getInstance()` acquires lock
- Even when instance already exists!

---

## 🔐 Double-Checked Locking

### Optimization: Check Before Locking

**Idea**: Only lock if instance doesn't exist.

```java
class Singleton {
    private static volatile Singleton instance = null;
    
    private Singleton() {
        System.out.println("Singleton constructor called");
    }

    public static Singleton getInstance() {
        // First check (without lock)
        if (instance == null) {
            synchronized (Singleton.class) {  // Lock only if needed
                // Second check (with lock)
                if (instance == null) {
                    instance = new Singleton();
                }
            }
        }
        
        return instance;
    }
}
```

### Why Two Checks?

**First Check (Outside Lock)**:
```java
if (instance == null) {
    // Only enter if instance not created
}
```
- **Purpose**: Avoid locking if instance already exists
- **Benefit**: Fast path for subsequent calls
- **No lock overhead** when instance exists

**Second Check (Inside Lock)**:
```java
synchronized (Singleton.class) {
    if (instance == null) {  // Check again!
        instance = new Singleton();
    }
}
```
- **Purpose**: Ensure only one thread creates instance
- **Why needed?** Multiple threads might pass first check

### The Race Condition Scenario

**Without Second Check**:
```
Thread T1                    Thread T2
    │                            │
    ├─ if (instance == null)     │
    │      ✓ (true)              │
    │                            ├─ if (instance == null)
    │                            │      ✓ (true)
    │  synchronized ✓            │
    │  (T1 acquires lock)        │
    │                            │  synchronized ⏳
    │                            │  (T2 waits...)
    │  new Singleton()           │
    │  (creates object 1)        │
    │  (T1 releases lock)        │
    │                            │
    │                            │  (T2 acquires lock)
    │                            │  new Singleton() ❌
    │                            │  (creates object 2)
```

**With Second Check**:
```
Thread T1                    Thread T2
    │                            │
    ├─ if (instance == null)     │
    │      ✓ (true)              │
    │                            ├─ if (instance == null)
    │                            │      ✓ (true)
    │  synchronized ✓            │
    │  if (instance == null)     │
    │      ✓ (true)              │
    │                            │  synchronized ⏳
    │  new Singleton()           │
    │  (T1 releases lock)        │
    │                            │
    │                            │  (T2 acquires lock)
    │                            │  if (instance == null)
    │                            │      ✗ (false) ✓
    │                            │  (T2 releases lock)
```

**Perfect!** This is the **Double-Checked Locking** pattern.

### Benefits

✅ **Thread-safe**: Only one instance created
✅ **Performance**: Lock only when necessary
✅ **Efficient**: Fast path for existing instance

---

## ⚡ Eager Initialization

### Simplest Approach: Initialize at Declaration

**Idea**: Create instance when class is loaded (before `main()`).

```java
class Singleton {
    private static final Singleton instance = new Singleton();
    
    private Singleton() {
        System.out.println("Singleton constructor called");
    }

    public static Singleton getInstance() {
        return instance;  // Just return (no checks needed!)
    }
}
```

### How It Works

**Timeline**:
```
1. Program starts
2. Static members initialized → instance = new Singleton()
3. Constructor called → "Singleton constructor called"
4. main() starts
5. Any call to getInstance() → returns existing instance
```

**Code**:
```java
public class Main {
    public static void main(String[] args) {
        Singleton s1 = Singleton.getInstance();  // Returns existing
        Singleton s2 = Singleton.getInstance();  // Returns existing
        
        System.out.println(s1 == s2);  // true
    }
}
```

**Output**:
```
Singleton constructor called
true
```

### Advantages

✅ **Simple**: No if-else checks needed
✅ **Thread-safe**: Initialized before any threads run
✅ **No locking**: No mutex overhead

### Disadvantages

❌ **Memory waste**: Object created even if never used
❌ **Slow startup**: Expensive initialization before `main()`
❌ **No lazy loading**: Can't defer creation

### When to Use Eager Initialization

✅ **Use when**:
- Object is **lightweight** (small memory footprint)
- Object is **definitely needed** in the application
- Initialization is **fast**

❌ **Don't use when**:
- Object is **expensive** to create
- Object might **not be used**
- Want **lazy loading**

---

## 💻 Implementation in Java

### Complete Java Implementation (Lazy + Thread-Safe)

```java
public class Singleton {
    private static volatile Singleton instance = null;
    
    // Private constructor
    private Singleton() {
        System.out.println("Singleton constructor called - New object created");
    }

    // Thread-safe getInstance with double-checked locking
    public static Singleton getInstance() {
        // First check (without lock)
        if (instance == null) {
            synchronized (Singleton.class) {
                // Second check (with lock)
                if (instance == null) {
                    instance = new Singleton();
                }
            }
        }
        
        return instance;
    }
    
    // Example method
    public void showMessage() {
        System.out.println("Hello from Singleton!");
    }
}

// Client code
public class Main {
    public static void main(String[] args) {
        System.out.println("=== Creating first instance ===");
        Singleton s1 = Singleton.getInstance();
        
        System.out.println("\n=== Creating second instance ===");
        Singleton s2 = Singleton.getInstance();
        
        System.out.println("\n=== Checking if same instance ===");
        System.out.println("s1 == s2: " + (s1 == s2));  // true
        
        System.out.println("\n=== Using singleton ===");
        s1.showMessage();
        s2.showMessage();
    }
}
```

**Output**:
```
=== Creating first instance ===
Singleton constructor called - New object created

=== Creating second instance ===

=== Checking if same instance ===
s1 == s2: true

=== Using singleton ===
Hello from Singleton!
Hello from Singleton!
```

### Java Synchronized Method (Simple Thread-Safe)

```java
public class Singleton {
    private static Singleton instance = null;
    
    // Private constructor
    private Singleton() {
        System.out.println("Singleton constructor called");
    }
    
    // Thread-safe getInstance
    public static synchronized Singleton getInstance() {
        if (instance == null) {
            instance = new Singleton();
        }
        return instance;
    }
    
    // Example method
    public void showMessage() {
        System.out.println("Hello from Singleton!");
    }
}

// Client code
public class Main {
    public static void main(String[] args) {
        Singleton s1 = Singleton.getInstance();
        Singleton s2 = Singleton.getInstance();
        
        System.out.println("s1 == s2: " + (s1 == s2));  // true
        
        s1.showMessage();
    }
}
```

### Java Double-Checked Locking

```java
public class Singleton {
    private static volatile Singleton instance = null;
    
    private Singleton() {
        System.out.println("Singleton constructor called");
    }
    
    public static Singleton getInstance() {
        // First check (without lock)
        if (instance == null) {
            synchronized (Singleton.class) {
                // Second check (with lock)
                if (instance == null) {
                    instance = new Singleton();
                }
            }
        }
        return instance;
    }
}
```

**Note**: `volatile` keyword ensures visibility across threads.

### Java Eager Initialization

```java
public class Singleton {
    private static final Singleton instance = new Singleton();
    
    private Singleton() {
        System.out.println("Singleton constructor called");
    }
    
    public static Singleton getInstance() {
        return instance;
    }
}
```

---

## 🌍 Real-World Use Cases

### 1. Logging System ⭐

**Problem**: Application needs to log messages from multiple services.

**Why Singleton?**
- **Single log file**: All logs should go to one file
- **Consistency**: Logs should be in order
- **Resource sharing**: One logger instance shared across application

**Without Singleton**:
```java
// Payment Service
Logger logger1 = new Logger();
logger1.log("Payment processed");

// Notification Service
Logger logger2 = new Logger();
logger2.log("Notification sent");

// Problem: Multiple logger instances!
// - Logs may be out of order
// - Multiple file handles
// - Memory waste
```

**With Singleton**:
```java
// Payment Service
Logger logger = Logger.getInstance();
logger.log("Payment processed");

// Notification Service
Logger logger = Logger.getInstance();  // Same instance!
logger.log("Notification sent");

// Benefits:
// - Single logger instance
// - Logs in order
// - Shared resource
```

**Implementation**:
```java
import java.io.*;
import java.time.LocalDateTime;

class Logger {
    private static Logger instance = null;
    private PrintWriter logFile;
    
    private Logger() {
        try {
            logFile = new PrintWriter(new FileWriter("app.log", true));
        } catch (IOException e) {
            e.printStackTrace();
        }
    }

    public static Logger getInstance() {
        if (instance == null) {
            instance = new Logger();
        }
        return instance;
    }
    
    public void log(String message) {
        logFile.println("[" + LocalDateTime.now() + "] " + message);
        logFile.flush();
    }
}
```

### 2. Database Connection ⭐

**Problem**: Database connections are expensive to create.

**Why Singleton?**
- **Expensive operation**: Creating connection takes time and resources
- **Limited connections**: Database has connection limit
- **Resource pooling**: Reuse single connection

**Without Singleton**:
```java
// User Service
DBConnection conn1 = new DBConnection("localhost", 3306);
conn1.query("SELECT * FROM users");

// Order Service
DBConnection conn2 = new DBConnection("localhost", 3306);
conn2.query("SELECT * FROM orders");

// Problem:
// - Multiple connections created
// - Expensive initialization repeated
// - May exceed connection limit
```

**With Singleton**:
```java
// User Service
DBConnection conn = DBConnection.getInstance();
conn.query("SELECT * FROM users");

// Order Service
DBConnection conn = DBConnection.getInstance();  // Same connection!
conn.query("SELECT * FROM orders");

// Benefits:
// - Single connection reused
// - One-time initialization
// - Resource efficient
```

**Implementation**:
```java
import java.sql.*;

class DBConnection {
    private static DBConnection instance = null;
    private Connection connection;
    
    private DBConnection(String host, int port) {
        System.out.println("Establishing database connection...");
        try {
            // Expensive operation
            String url = "jdbc:mysql://" + host + ":" + port + "/mydb";
            connection = DriverManager.getConnection(url, "user", "password");
        } catch (SQLException e) {
            e.printStackTrace();
        }
    }

    public static DBConnection getInstance() {
        if (instance == null) {
            instance = new DBConnection("localhost", 3306);
        }
        return instance;
    }
    
    public void query(String sql) {
        try {
            Statement stmt = connection.createStatement();
            stmt.execute(sql);
        } catch (SQLException e) {
            e.printStackTrace();
        }
    }
}
```

### 3. Configuration Manager ⭐

**Problem**: Application configuration should be consistent across services.

**Why Singleton?**
- **Single source of truth**: One configuration for entire app
- **Consistency**: All services use same config
- **Prevent modification**: No service can change shared config

**Without Singleton**:
```java
// Payment Service
Config config1 = new Config();
String apiKey1 = config1.get("API_KEY");

// Notification Service
Config config2 = new Config();
config2.set("API_KEY", "new_key");  // Modifies its own copy!

// Problem:
// - Different configs in different services
// - Inconsistent state
// - Hard to manage
```

**With Singleton**:
```java
// Payment Service
Config config = Config.getInstance();
String apiKey = config.get("API_KEY");

// Notification Service
Config config = Config.getInstance();  // Same instance!
String apiKey = config.get("API_KEY");  // Same API key

// Benefits:
// - Single configuration instance
// - Consistent across services
// - Single source of truth
```

**Implementation**:
```java
import java.util.*;
import java.io.*;

class Config {
    private static Config instance = null;
    private Map<String, String> settings;
    
    private Config() {
        settings = new HashMap<>();
        loadFromFile("config.ini");
    }

    public static Config getInstance() {
        if (instance == null) {
            instance = new Config();
        }
        return instance;
    }
    
    private void loadFromFile(String filename) {
        // Load configuration from file
        try (BufferedReader br = new BufferedReader(new FileReader(filename))) {
            String line;
            while ((line = br.readLine()) != null) {
                String[] parts = line.split("=");
                if (parts.length == 2) {
                    settings.put(parts[0].trim(), parts[1].trim());
                }
            }
        } catch (IOException e) {
            e.printStackTrace();
        }
    }
    
    public String get(String key) {
        return settings.get(key);
    }
    
    public void set(String key, String value) {
        settings.put(key, value);
    }
}
```

### 4. Other Real-World Examples

**Cache Manager**:
```java
CacheManager cache = CacheManager.getInstance();
cache.put("user:123", userData);
```

**Thread Pool**:
```java
ThreadPool pool = ThreadPool.getInstance();
pool.submit(task);
```

**Application State**:
```java
AppState state = AppState.getInstance();
state.setCurrentUser(user);
```

---

## ⚠️ When NOT to Use Singleton

### 1. When Multiple Instances Are Needed

**Example**: Game with multiple players

```java
// ❌ Bad: Using Singleton for Player
class Player {
    private static Player instance = null;
    private String name;
    private int score;
    
    public static Player getInstance() {
        if (instance == null) {
            instance = new Player();
        }
        return instance;
    }
    
    public void setName(String name) {
        this.name = name;
    }
}

// Problem: All players share same instance!
Player player1 = Player.getInstance();
player1.setName("Alice");

Player player2 = Player.getInstance();
player2.setName("Bob");  // Overwrites Alice!

// player1 and player2 are SAME object!
```

**✅ Correct Approach**:
```java
class Player {
    private String name;
    private int score;
    
    public Player(String name) {
        this.name = name;
        this.score = 0;
    }
}

// Each player is separate instance
Player player1 = new Player("Alice");
Player player2 = new Player("Bob");
```

### 2. When Testing Becomes Difficult

**Problem**: Singleton makes unit testing hard

```java
class UserService {
    private Logger logger = Logger.getInstance();  // Tight coupling!
    
    public void createUser(User user) {
        logger.log("Creating user: " + user.getName());
        // ... create user logic
    }
}

// Testing problem:
// - Can't mock Logger
// - Can't test in isolation
// - Shared state across tests
```

**✅ Better Approach**: Dependency Injection
```java
class UserService {
    private Logger logger;
    
    public UserService(Logger logger) {
        this.logger = logger;
    }
    
    public void createUser(User user) {
        logger.log("Creating user: " + user.getName());
    }
}

// Easy to test
MockLogger mockLogger = new MockLogger();
UserService service = new UserService(mockLogger);
```

### 3. When Thread Safety Adds Complexity

**Problem**: Thread-safe Singleton is complex

```java
// Complex thread-safe implementation
class Singleton {
    private static volatile Singleton instance = null;
    
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

// Problems:
// - Complex to understand
// - Hard to test
// - Potential deadlocks
// - Performance overhead
```

**When to Avoid**:
- If you don't need thread safety
- If complexity outweighs benefits
- If simpler alternatives exist

---

## ⚖️ Advantages and Disadvantages

### Advantages ✅

**1. Controlled Access**
- Only one instance exists
- Global point of access
- Prevents accidental multiple instances

**2. Memory Efficiency**
- Single instance shared across application
- Saves memory (especially for expensive objects)
- Reduces resource usage

**3. Lazy Initialization**
- Object created only when needed
- Faster application startup
- No wasted resources

**4. Global State**
- Shared state across application
- Consistent configuration
- Single source of truth

**5. Easy to Implement**
- Simple pattern
- Few lines of code
- Well-understood

### Disadvantages ❌

**1. Global State Issues**
- Hidden dependencies
- Hard to track who uses singleton
- Tight coupling

**2. Testing Difficulties**
- Hard to mock
- Shared state across tests
- Tests not isolated

**3. Thread Safety Complexity**
- Requires locking mechanisms
- Performance overhead
- Potential deadlocks

**4. Violates Single Responsibility**
- Class manages its own lifecycle
- Class provides business logic
- Two responsibilities!

**5. Hidden Dependencies**
- Not clear from API what dependencies exist
- Hard to understand code flow
- Reduces code readability

### Trade-offs

| Aspect | Benefit | Cost |
|--------|---------|------|
| **Memory** | Saves memory | May waste if not used (eager) |
| **Thread Safety** | Ensures single instance | Adds complexity |
| **Global Access** | Easy to use | Hidden dependencies |
| **Lazy Loading** | Defers creation | First access slower |

---

## 🎓 Key Takeaways

### Implementation Checklist

✅ **Private constructor** - Prevents external instantiation
✅ **Static instance variable** - Holds the single instance
✅ **Static getInstance() method** - Provides global access
✅ **Thread safety** - Use locking or eager initialization

### Three Main Approaches

**1. Lazy Initialization (Basic)**
```java
if (instance == null) {
    instance = new Singleton();
}
return instance;
```
- ✅ Simple
- ❌ Not thread-safe

**2. Thread-Safe (Double-Checked Locking)**
```java
if (instance == null) {
    synchronized (Singleton.class) {
        if (instance == null) {
            instance = new Singleton();
        }
    }
}
return instance;
```
- ✅ Thread-safe
- ✅ Performance optimized
- ❌ Complex

**3. Eager Initialization**
```java
private static final Singleton instance = new Singleton();
```
- ✅ Simple
- ✅ Thread-safe
- ❌ No lazy loading

### When to Use Singleton

✅ **Use when**:
- Need exactly one instance
- Global access required
- Resource is expensive
- Examples: Logger, DB Connection, Config

❌ **Don't use when**:
- Need multiple instances
- Testing is important
- Want loose coupling
- Simpler alternatives exist

---

## ❓ Interview Q&A

### Q1: What is the Singleton Design Pattern? Explain with an example.

**Answer**:

The Singleton Design Pattern ensures that a class has **only one instance** and provides a **global point of access** to it.

**Key Components**:
1. **Private constructor** - Prevents external instantiation
2. **Static instance variable** - Holds the single instance
3. **Static getInstance() method** - Returns the instance

**Example**:
```java
class Singleton {
    private static Singleton instance = null;
    
    private Singleton() {  // Private constructor
        System.out.println("Singleton created");
    }

    public static Singleton getInstance() {
        if (instance == null) {
            instance = new Singleton();
        }
        return instance;
    }
}

// Usage
public class Main {
    public static void main(String[] args) {
        Singleton s1 = Singleton.getInstance();
        Singleton s2 = Singleton.getInstance();
        
        System.out.println(s1 == s2);  // true (same instance)
    }
}
```

**Output**:
```
Singleton created
true
```

**Benefit**: Only one instance created, memory efficient.

---

### Q2: Why do we make the constructor private in Singleton?

**Answer**:

**Purpose**: To prevent external code from creating new instances using `new` keyword.

**Without Private Constructor**:
```java
class Singleton {
    public Singleton() {}  // Public constructor
}

// Problem: Can create multiple instances!
Singleton s1 = new Singleton();
Singleton s2 = new Singleton();
// s1 != s2 (different objects)
```

**With Private Constructor**:
```java
class Singleton {
    private Singleton() {}  // Private constructor

    public static Singleton getInstance() {
        // Only way to get instance
    }
}

// Can't create directly
Singleton s = new Singleton();  // ❌ Error: Constructor is private!

// Must use getInstance()
Singleton s = Singleton.getInstance();  // ✓ Correct
```

**Key Point**: Private constructor **forces** users to use `getInstance()`, which controls instance creation.

---

### Q3: What is the difference between Lazy and Eager initialization in Singleton?

**Answer**:

| Aspect | Lazy Initialization | Eager Initialization |
|--------|---------------------|----------------------|
| **When created** | When first accessed | At program start |
| **Memory** | Saves if not used | Wastes if not used |
| **Startup time** | Faster | Slower |
| **Thread safety** | Needs locking | Inherently safe |
| **Complexity** | More complex | Simple |

**Lazy Initialization**:
```java
class Singleton {
    private static Singleton instance = null;
    
    public static Singleton getInstance() {
        if (instance == null) {  // Create on first call
            instance = new Singleton();
        }
        return instance;
    }
}
```

**Eager Initialization**:
```java
class Singleton {
    private static final Singleton instance = new Singleton();
    
    public static Singleton getInstance() {
        return instance;  // Already created
    }
}
```

**When to use each**:
- **Lazy**: Object is expensive, might not be used
- **Eager**: Object is lightweight, definitely needed

---

### Q4: How do you make Singleton thread-safe?

**Answer**:

**Problem**: Multiple threads can create multiple instances.

**Solution 1: Simple Locking**
```java
class Singleton {
    private static Singleton instance = null;

    public static synchronized Singleton getInstance() {
        if (instance == null) {
            instance = new Singleton();
        }
        return instance;
    }
}
```

**Problem**: Lock acquired every time (expensive!)

**Solution 2: Double-Checked Locking** (Better)
```java
class Singleton {
    private static volatile Singleton instance = null;

    public static Singleton getInstance() {
        // First check (without lock)
        if (instance == null) {
            synchronized (Singleton.class) {
                // Second check (with lock)
                if (instance == null) {
                    instance = new Singleton();
                }
            }
        }
        return instance;
    }
}
```

**Why two checks?**
1. **First check**: Avoid locking if already created (fast path)
2. **Second check**: Ensure only one thread creates instance

**Solution 3: Eager Initialization** (Simplest)
```java
private static final Singleton instance = new Singleton();
```
- Created before threads start
- No locking needed
- Thread-safe by default

---

### Q5: What is Double-Checked Locking and why is it needed?

**Answer**:

**Double-Checked Locking**: Check twice - once before lock, once after lock.

**Code**:
```java
if (instance == null) {        // First check
    synchronized (Singleton.class) {
        if (instance == null) {    // Second check
            instance = new Singleton();
        }
    }
}
```

**Why First Check?**
- **Purpose**: Avoid locking if instance exists
- **Benefit**: Performance optimization
- **When**: After first creation, all calls skip lock

**Why Second Check?**
- **Purpose**: Ensure only one thread creates instance
- **Scenario**: Multiple threads pass first check

**Example Without Second Check**:
```
Thread T1                    Thread T2
    │                            │
    ├─ if (instance == null)     │
    │      ✓ (true)              │
    │                            ├─ if (instance == null)
    │                            │      ✓ (true)
    │  synchronized ✓            │
    │  new Singleton()           │
    │  (T1 releases lock)        │
    │                            │  synchronized ✓
    │                            │  new Singleton() ❌
    │                            │  (creates second instance!)
```

**With Second Check**:
```
Thread T1                    Thread T2
    │                            │
    ├─ if (instance == null)     │
    │      ✓ (true)              │
    │                            ├─ if (instance == null)
    │                            │      ✓ (true)
    │  synchronized ✓            │
    │  if (instance == null)     │
    │      ✓ (true)              │
    │  new Singleton()           │
    │  (T1 releases lock)        │
    │                            │  synchronized ✓
    │                            │  if (instance == null)
    │                            │      ✗ (false) ✓
    │                            │  (T2 releases lock)
```

**Perfect!** Only one instance created.

---

### Q6: What are real-world use cases of Singleton?

**Answer**:

**1. Logging System**
```java
Logger logger = Logger.getInstance();
logger.log("Payment processed");
```
- **Why**: Single log file, consistent logging
- **Benefit**: All services use same logger

**2. Database Connection**
```java
DBConnection db = DBConnection.getInstance();
db.query("SELECT * FROM users");
```
- **Why**: Expensive to create, limited connections
- **Benefit**: Reuse single connection

**3. Configuration Manager**
```java
Config config = Config.getInstance();
String apiKey = config.get("API_KEY");
```
- **Why**: Single source of truth
- **Benefit**: Consistent config across app

**4. Cache Manager**
```java
Cache cache = Cache.getInstance();
cache.put("user:123", userData);
```
- **Why**: Shared cache across services
- **Benefit**: Memory efficient

**5. Thread Pool**
```java
ThreadPool pool = ThreadPool.getInstance();
pool.submit(task);
```
- **Why**: Limited threads, expensive to create
- **Benefit**: Resource pooling

**Common Pattern**: Resource-intensive, shared-state objects.

---

### Q7: What are the disadvantages of Singleton?

**Answer**:

**1. Global State Issues**
```java
// Hidden dependency
class UserService {
    void createUser() {
        Logger.getInstance().log("Creating user");
        // Not clear from signature that Logger is used
    }
}
```
- **Problem**: Hidden dependencies
- **Impact**: Hard to understand code flow

**2. Testing Difficulties**
```java
// Hard to mock
class PaymentService {
    void processPayment() {
        DBConnection db = DBConnection.getInstance();
        // Can't inject mock database for testing
    }
}
```
- **Problem**: Tight coupling
- **Impact**: Can't test in isolation

**3. Thread Safety Complexity**
```java
// Complex implementation
if (instance == null) {
    synchronized (Singleton.class) {
        if (instance == null) {
            instance = new Singleton();
        }
    }
}
```
- **Problem**: Requires locking
- **Impact**: Performance overhead, potential deadlocks

**4. Violates Single Responsibility**
```java
class Singleton {
    // Manages its own lifecycle
    static Singleton getInstance();
    
    // Provides business logic
    void doSomething();
}
```
- **Problem**: Two responsibilities
- **Impact**: Harder to maintain

**5. Memory Waste (Eager)**
```java
private static final Singleton instance = new Singleton();  // Created immediately
// What if never used? Memory wasted!
```

**Trade-off**: Convenience vs. Good design principles.

---

### Q8: When should you NOT use Singleton?

**Answer**:

❌ **Don't use when**:

**1. Multiple Instances Needed**
```java
// ❌ Bad: Singleton for Player
Player p1 = Player.getInstance();
Player p2 = Player.getInstance();
// p1 == p2 (same player!)

// ✅ Good: Regular class
Player p1 = new Player("Alice");
Player p2 = new Player("Bob");
```

**2. Testing is Important**
```java
// ❌ Bad: Hard to test
class Service {
    void process() {
        Logger.getInstance().log("Processing");
    }
}

// ✅ Good: Dependency injection
class Service {
    Logger logger;
    Service(Logger logger) {
        this.logger = logger;
    }
}
```

**3. Want Loose Coupling**
```java
// ❌ Bad: Tight coupling
DBConnection.getInstance().query("...");

// ✅ Good: Inject dependency
class Repository {
    DBConnection db;
    Repository(DBConnection db) {
        this.db = db;
    }
}
```

**4. Simpler Alternatives Exist**
```java
// ❌ Bad: Singleton for simple config
Config.getInstance().get("API_KEY");

// ✅ Good: Pass as parameter
void processPayment(Config config) {
    String apiKey = config.get("API_KEY");
}
```

**Rule of Thumb**: Use Singleton only when you're **absolutely sure** you need exactly one instance.

---

### Q9: How do you prevent copying of Singleton instance?

**Answer**:

**Problem**: Even with private constructor, instance can be copied (in languages that support cloning).

**Java Solution**: Override `clone()` method to prevent cloning.

```java
public class Singleton {
    private static Singleton instance = null;
    
    private Singleton() {}
    
    public static Singleton getInstance() {
        if (instance == null) {
            instance = new Singleton();
        }
        return instance;
    }
    
    // Prevent cloning
    @Override
    protected Object clone() throws CloneNotSupportedException {
        throw new CloneNotSupportedException("Cannot clone singleton instance");
    }
}
```

**Now cloning is prevented**:
```java
Singleton s1 = Singleton.getInstance();
Singleton s2 = (Singleton) s1.clone();  // ❌ Error: CloneNotSupportedException
```

---

### Q10: What is the difference between Singleton and Static class?

**Answer**:

| Aspect | Singleton | Static Class |
|--------|-----------|--------------|
| **Instance** | One object instance | No instance (all static) |
| **Inheritance** | Can inherit, be inherited | Cannot inherit |
| **Interface** | Can implement interfaces | Cannot implement interfaces |
| **Polymorphism** | Supports polymorphism | No polymorphism |
| **Lazy loading** | Supports lazy loading | No lazy loading |
| **State** | Can maintain state | Only static state |

**Singleton**:
```java
class Singleton {
    private static Singleton instance = null;
    private int state;  // Instance variable
    
    public static Singleton getInstance() {
        if (instance == null) {
            instance = new Singleton();
        }
        return instance;
    }
    
    public void setState(int s) {
        state = s;
    }
}

// Can inherit
class SpecialSingleton extends Singleton {}

// Has instance
Singleton s = Singleton.getInstance();
```

**Static Class**:
```java
class StaticClass {
    public static int state;  // Only static members
    public static void doSomething() {}
}

// No instance
StaticClass.doSomething();  // Call directly

// Cannot instantiate
```

**When to use each**:
- **Singleton**: Need object, inheritance, interfaces
- **Static**: Simple utility functions, no state

---

### Q11: Can Singleton be destroyed? How?

**Answer**:

**Yes**, Singleton can be destroyed, but it requires explicit handling.

**Problem**: Memory leak if not cleaned up (though Java has garbage collection).

**Solution 1: Manual Cleanup (Reset)**
```java
class Singleton {
    private static Singleton instance = null;

    public static Singleton getInstance() {
        if (instance == null) {
            instance = new Singleton();
        }
        return instance;
    }
    
    public static void destroy() {
        instance = null;  // Allow garbage collection
    }
}

// Usage
public class Main {
    public static void main(String[] args) {
        Singleton s = Singleton.getInstance();
        // Use singleton...
        
        Singleton.destroy();  // Clean up (reset)
    }
}
```

**Solution 2: Lazy Holder Pattern (Automatic Cleanup)**
```java
class Singleton {
    private Singleton() {}
    
    private static class SingletonHolder {
        private static final Singleton INSTANCE = new Singleton();
    }
    
    public static Singleton getInstance() {
        return SingletonHolder.INSTANCE;
    }
}

// Automatically garbage collected when no longer referenced
```

**Best Practice**: Java's garbage collector handles cleanup automatically when no references exist.

---

### Q12: What is Bill Pugh Singleton (Lazy Holder Pattern)?

**Answer**:

**Bill Pugh Singleton (Lazy Holder Pattern)**: Uses static inner class for thread-safe, lazy initialization.

**Implementation**:
```java
public class Singleton {
    private Singleton() {
        System.out.println("Singleton created");
    }

    // Static inner class - loaded only when getInstance() is called
    private static class SingletonHolder {
        private static final Singleton INSTANCE = new Singleton();
    }

    public static Singleton getInstance() {
        return SingletonHolder.INSTANCE;  // Lazy initialization!
    }
    
    public void doSomething() {
        System.out.println("Doing something");
    }
}

// Usage
public class Main {
    public static void main(String[] args) {
        Singleton s1 = Singleton.getInstance();
        Singleton s2 = Singleton.getInstance();
        
        System.out.println(s1 == s2);  // true (same instance)
        
        s1.doSomething();
    }
}
```

**How It Works**:
1. **First call**: `SingletonHolder` class is loaded, `INSTANCE` is created (constructor called)
2. **Subsequent calls**: Same `INSTANCE` returned
3. **Program end**: `INSTANCE` garbage collected automatically

**Advantages**:
✅ **Thread-safe** (JVM guarantees thread-safe class loading)
✅ **Lazy initialization** (created only when getInstance() is called)
✅ **Automatic cleanup** (garbage collector handles it)
✅ **Simple implementation** (no explicit synchronization needed)
✅ **No memory leaks**

**Why Thread-Safe?**
- JVM guarantees thread-safe initialization of static fields during class loading
- No manual synchronization needed!

**This is the RECOMMENDED way** to implement Singleton in Java!

---

## 📝 Summary

The **Singleton Design Pattern** ensures a class has only **one instance** and provides a **global point of access** to it.

**Key Implementation Steps**:
1. **Private constructor** - Prevent external instantiation
2. **Static instance variable** - Hold the single instance
3. **Static getInstance() method** - Provide global access

**Three Main Approaches**:
- **Lazy Initialization**: Create when first accessed
- **Eager Initialization**: Create at program start
- **Thread-Safe**: Use double-checked locking or Bill Pugh Singleton

**Real-World Use Cases**:
- Logging System
- Database Connection
- Configuration Manager
- Cache Manager
- Thread Pool

**When to Use**:
✅ Need exactly one instance
✅ Global access required
✅ Resource is expensive
✅ Shared state needed

**When NOT to Use**:
❌ Multiple instances needed
❌ Testing is important
❌ Want loose coupling
❌ Simpler alternatives exist

**Best Practice**: Use Bill Pugh Singleton (Lazy Holder Pattern) in Java for simplicity and safety!

---

**Happy Learning! 🚀**
