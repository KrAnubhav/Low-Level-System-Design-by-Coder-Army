# LLD-21: Proxy Design Pattern (Structural Pattern)

## 📚 Table of Contents
1. [Introduction](#introduction)
2. [What is Proxy Design Pattern?](#what-is-proxy-design-pattern)
3. [Core Concept](#core-concept)
4. [Types of Proxy](#types-of-proxy)
5. [Virtual Proxy](#virtual-proxy)
6. [Protection Proxy](#protection-proxy)
7. [Remote Proxy](#remote-proxy)
8. [UML Diagrams](#uml-diagrams)
9. [Implementation in Java](#implementation-in-java)
10. [Standard UML Diagram](#standard-uml-diagram)
11. [Key Concepts](#key-concepts)
12. [Real-World Use Cases](#real-world-use-cases)
13. [Key Takeaways](#key-takeaways)
14. [Interview Q&A](#interview-qa)

---

## 🎯 Introduction

Welcome back to our LLD series! Today we're learning about the **Proxy Design Pattern**.

### Pattern Context

This is another **use-case specific** design pattern. It solves very particular types of problems elegantly. While it's specific, it's:
- **Very easy to understand**
- **Extremely powerful** when the use case matches
- **Commonly used** in real-world applications

Let's dive in!

---

## 🤔 What is Proxy Design Pattern?

**Simple Definition**: A Proxy acts as a **representative** or **placeholder** for another object to control access to it.

**Core Idea**: 
- You have a **Real Object** (Resource)
- You don't want clients to access it directly
- You create a **Proxy** that sits between the client and the real object
- All requests go through the Proxy first

### The Basic Flow

```
User → Request → Proxy → Request → Resource
                    ↓
User ← Response ← Proxy ← Response ← Resource
```

**Key Point**: The user doesn't know they're talking to a Proxy. They think they're talking directly to the Resource!

---

## 🌳 Core Concept

### Why Introduce a Proxy?

**Several reasons**:

1. **Critical Data Protection**
   - Resource contains sensitive data
   - Proxy authenticates users before allowing access
   - Only authorized users can reach the resource

2. **Validation & Checks**
   - Proxy performs validation before forwarding requests
   - Prevents invalid data from reaching the resource
   - Adds security layers

3. **Remote Access**
   - Resource exists on a different server (over the internet)
   - Proxy is a local object that handles network communication
   - Client doesn't need to know about network details

### The Proxy Principle

**"The Proxy behaves exactly like the Real Object"**

- Client cannot differentiate between Proxy and Real Object
- Both implement the same interface
- Client doesn't need to know it's using a Proxy

---

## 📋 Types of Proxy

There are **three main types** of Proxy:

1. **Virtual Proxy** - Lazy loading of expensive resources
2. **Protection Proxy** - Access control and authentication
3. **Remote Proxy** - Represents objects in different address spaces

**Important**: 
- All three share the **same core concept**
- The **UML structure is identical** for all three
- Only the **intent** differs

If you understand one, you understand all three!

---

## 💾 Virtual Proxy

### Purpose

**Controls creation of expensive objects** by delaying instantiation until actually needed.

### Problem Statement

Imagine you have an `ImageDisplay` class that:
- Loads images from disk
- Applies compression algorithms
- Applies filters
- All this happens in the **constructor** (expensive operation!)

**The Issue**:
```java
ImageDisplay img = new ImageDisplay("photo.jpg");
// Constructor runs expensive operations immediately
// But what if we never call img.display()?
// We wasted memory and resources!
```

**Problems**:
1. Expensive operations run even if never used
2. Memory is wasted
3. Application becomes slow
4. Resources are consumed unnecessarily

### Solution: Virtual Proxy

**Idea**: Don't create the real object until its methods are actually called.

#### Design
<img width="680" height="446" alt="image" src="https://github.com/user-attachments/assets/36207be6-9df4-4f2c-b65c-de4b65b5ecdd" />


**Relationships**:
1. **ImageProxy IS-A IDisplay** (implements interface)
2. **RealImage IS-A IDisplay** (implements interface)
3. **ImageProxy HAS-A RealImage** (composition)

#### How It Works

**Step 1**: Client creates ImageProxy
```java
IDisplay img = new ImageProxy("photo.jpg");
// Only proxy is created, NOT the real image
// No expensive operations yet!
```

**Step 2**: Client calls display()
```java
img.display();
// NOW the proxy creates RealImage
// Expensive operations happen here
// Then forwards the display() call
```

**Step 3**: Subsequent calls
```java
img.display(); // Already created, just forward the call
```

### Example Code Structure

```java
// Interface
interface IDisplay {
    void display();
}

// Real Object (Expensive)
class RealImage implements IDisplay {
    private String path;
    
    public RealImage(String path) {
        this.path = path;
        loadFromDisk();      // Expensive!
        applyCompression();  // Expensive!
        applyFilters();      // Expensive!
    }
    
    public void display() {
        System.out.println("Displaying: " + path);
    }
}

// Proxy (Lightweight)
class ImageProxy implements IDisplay {
    private String path;
    private RealImage realImage; // Initially null
    
    public ImageProxy(String path) {
        this.path = path;
        this.realImage = null; // Don't create yet!
    }
    
    public void display() {
        // Lazy loading
        if (realImage == null) {
            realImage = new RealImage(path); // Create now!
        }
        realImage.display(); // Forward the call
    }
}
```

### Key Benefits

✅ **Memory Efficient**: Object created only when needed
✅ **Faster Startup**: No upfront expensive operations
✅ **Lazy Loading**: Deferred initialization

⚠️ **Trade-off**: First call to `display()` will be slower

### When to Use Virtual Proxy

- Loading large images/videos
- Database connections
- Heavy object initialization
- Resource-intensive operations

---

## 🔒 Protection Proxy

### Purpose

**Controls access to resources** by adding authentication and authorization checks.

### Problem Statement

You have a `DocumentReader` class that can unlock PDFs:
```java
class RealDocumentReader {
    void unlockPDF(String file, String password) {
        // Unlocks the PDF
    }
}
```

**The Issue**: 
- This is a **premium feature**
- Not all users should access it
- Only premium users should unlock PDFs

### Solution: Protection Proxy

**Idea**: Add a proxy that checks user permissions before allowing access.

#### Design
<img width="1362" height="992" alt="image" src="https://github.com/user-attachments/assets/77106109-47ca-4aa2-be10-65bf9ba03fb1" />


**Additional Component**:
```java
class User {
    String name;
    boolean isPremium;
}
```

#### How It Works

**Step 1**: Client has a user and wants to unlock PDF
```java
User user = new User("John", false); // Not premium
IDocumentReader reader = new DocumentProxy(user);
```

**Step 2**: Client calls unlockPDF()
```java
reader.unlockPDF("secret.pdf", "password");
// Proxy checks: Is user premium?
// If NO → Throw exception or deny access
// If YES → Forward to RealDocumentReader
```

### Example Code Structure

```java
// Interface
interface IDocumentReader {
    void unlockPDF(String file, String password);
}

// Real Object
class RealDocumentReader implements IDocumentReader {
    public void unlockPDF(String file, String password) {
        System.out.println("Unlocking PDF: " + file);
        // Actual unlock logic
    }
}

// Proxy with Protection
class DocumentProxy implements IDocumentReader {
    private User user;
    private RealDocumentReader realReader;
    
    public DocumentProxy(User user) {
        this.user = user;
        this.realReader = new RealDocumentReader();
    }
    
    public void unlockPDF(String file, String password) {
        // Protection check
        if (user.isPremium) {
            realReader.unlockPDF(file, password);
        } else {
            throw new SecurityException(
                "This feature is only for premium users!"
            );
        }
    }
}
```

### Key Benefits

✅ **Access Control**: Only authorized users can access
✅ **Security**: Protects critical resources
✅ **Authentication**: Validates user permissions
✅ **Separation of Concerns**: Security logic separate from business logic

### When to Use Protection Proxy

- Premium features
- Admin-only operations
- Role-based access control (RBAC)
- Sensitive data protection

---

## 🌐 Remote Proxy

### Purpose

**Represents an object that exists in a different address space** (different server, over the network).

### Problem Statement

You have a `DataService` that exists on a **remote server**:
- Client needs to fetch data from it
- Requires network connection
- Client shouldn't worry about network details

**The Issue**:
```java
// Without proxy
DataService service = connectToServer("192.168.1.100");
service.fetchData(); // Client handles network complexity
```

### Solution: Remote Proxy

**Idea**: Create a local proxy that handles all network communication.

#### Design
<img width="640" height="447" alt="image" src="https://github.com/user-attachments/assets/3d7e1d03-e5fb-4a5b-8568-ffe815d81943" />


```
┌─────────────────┐
│  IDataService   │ (Interface)
├─────────────────┤
│ + fetchData()   │
└─────────────────┘
        △
        │
    ┌───┴────┐
    │        │
┌───▽─────────┐  ┌▽──────────────┐
│DataService  │  │DataProxy      │
│(Remote)     │  │(Local)        │
├─────────────┤  ├───────────────┤
│+ fetchData()│  │- remoteService│◆──────┐
└─────────────┘  │+ fetchData()  │       │
  (Server 1)     └───────────────┘       │
                   (Server 2)             │
                        │                 │
                        └─────────────────┘
                     Network Connection
```

#### How It Works

**Step 1**: Client creates local proxy
```java
IDataService service = new DataProxy();
// No network connection yet
```

**Step 2**: Client calls fetchData()
```java
service.fetchData();
// Proxy establishes network connection
// Calls remote DataService over the network
// Returns data to client
```

**Client doesn't know**:
- Where the server is located
- How to establish connection
- Network protocols used

### Example Code Structure

```java
// Interface
interface IDataService {
    String fetchData();
}

// Real Object (on remote server)
class DataService implements IDataService {
    public String fetchData() {
        // Fetch from database
        return "Data from remote server";
    }
}

// Proxy (local)
class DataProxy implements IDataService {
    private DataService remoteService;
    
    public String fetchData() {
        // Lazy loading
        if (remoteService == null) {
            // Establish network connection
            System.out.println("Connecting to remote server...");
            remoteService = connectToRemoteServer();
        }
        
        // Forward call over network
        return remoteService.fetchData();
    }
    
    private DataService connectToRemoteServer() {
        // Network connection logic
        // IP, port, authentication, etc.
        return new DataService();
    }
}
```

### Key Benefits

✅ **Location Transparency**: Client doesn't know object is remote
✅ **Network Abstraction**: Hides network complexity
✅ **Lazy Connection**: Connect only when needed
✅ **Clean Separation**: Network logic separate from business logic

### When to Use Remote Proxy

- Microservices architecture
- Distributed systems
- API calls to external services
- Client-server applications

---

## 📊 UML Diagrams

### Generic Proxy Pattern Structure
<img width="1017" height="762" alt="image" src="https://github.com/user-attachments/assets/46459dfa-fb41-4a07-a1f2-ba846ab7665d" />

┌───────────────┐
│    Client     │
└───────────────┘
        |
        | uses
        v
┌────────────────────────┐
│        ISubject        │
│   <<interface>>        │
├────────────────────────┤
│ + operation()          │
└───────────▲────────────┘
            │
   ┌────────┴─────────┐
   │                  │
implements        implements
   │                  │
┌──────────────┐   ┌────────────────┐
│    Proxy     │   │  RealSubject   │
├──────────────┤   ├────────────────┤
│ - real:      │   │                │
│   RealSubject│   │                │
├──────────────┤   ├────────────────┤
│ + operation()│──▶│ + operation()  │
└──────────────┘   └────────────────┘

<img width="722" height="300" alt="image" src="https://github.com/user-attachments/assets/d42c1457-d9cf-4dcf-9813-c81156d10820" />


**Key Relationships**:
1. **Proxy IS-A ISubject** (implements same interface)
2. **RealSubject IS-A ISubject** (implements same interface)
3. **Proxy HAS-A RealSubject** (composition)

### Sequence Diagram: Proxy Flow

```
Client      Proxy       RealSubject
  │           │              │
  │─operation()→             │
  │           │              │
  │           │─checks/      │
  │           │  validation  │
  │           │              │
  │           │─operation()─→│
  │           │              │
  │           │←─result──────│
  │           │              │
  │←─result───│              │
  │           │              │
```

**Flow Explanation**:
1. Client calls operation() on Proxy
2. Proxy performs checks (authentication, lazy loading, etc.)
3. Proxy forwards call to RealSubject
4. RealSubject executes and returns result
5. Proxy returns result to Client

---

## 💻 Implementation in Java

### Complete Working Code: Virtual Proxy

```java
// Interface
interface IImage {
    void display();
}

// Real Object (Expensive to create)
class RealImage implements IImage {
    private String fileName;
    
    public RealImage(String fileName) {
        this.fileName = fileName;
        loadFromDisk(); // Expensive operation!
    }
    
    private void loadFromDisk() {
        System.out.println("Loading image from disk: " + fileName);
        // Simulate expensive operations
        try {
            Thread.sleep(2000); // 2 second delay
        } catch (InterruptedException e) {
            e.printStackTrace();
        }
        System.out.println("Image loaded successfully!");
    }
    
    @Override
    public void display() {
        System.out.println("Displaying image: " + fileName);
    }
}

// Proxy (Lightweight)
class ImageProxy implements IImage {
    private String fileName;
    private RealImage realImage; // Lazy loading
    
    public ImageProxy(String fileName) {
        this.fileName = fileName;
        this.realImage = null; // Don't create yet
        System.out.println("ImageProxy created (no loading yet)");
    }
    
    @Override
    public void display() {
        // Lazy loading - create only when needed
        if (realImage == null) {
            System.out.println("First time display - creating RealImage...");
            realImage = new RealImage(fileName);
        }
        realImage.display();
    }
}

// Client Code
public class VirtualProxyDemo {
    public static void main(String[] args) {
        System.out.println("=== Creating Image Proxy ===");
        IImage image1 = new ImageProxy("sample.jpg");
        
        System.out.println("\n=== First Display Call ===");
        image1.display(); // Loads and displays
        
        System.out.println("\n=== Second Display Call ===");
        image1.display(); // Just displays (already loaded)
        
        System.out.println("\n=== Creating Another Proxy ===");
        IImage image2 = new ImageProxy("photo.png");
        // Never call display() - no loading happens!
        
        System.out.println("\nProgram ends - image2 was never loaded!");
    }
}
```

### Expected Output

```
=== Creating Image Proxy ===
ImageProxy created (no loading yet)

=== First Display Call ===
First time display - creating RealImage...
Loading image from disk: sample.jpg
Image loaded successfully!
Displaying image: sample.jpg

=== Second Display Call ===
Displaying image: sample.jpg

=== Creating Another Proxy ===
ImageProxy created (no loading yet)

Program ends - image2 was never loaded!
```

### Code Walkthrough

#### 1. Proxy Creation

```java
IImage image1 = new ImageProxy("sample.jpg");
```

**What happens**:
- Only ImageProxy object is created
- `fileName` is stored
- `realImage` is set to `null`
- **No expensive operations** run
- Fast and lightweight!

#### 2. First display() Call

```java
image1.display();
```

**What happens**:
1. Proxy checks: `if (realImage == null)`
2. Condition is true (first call)
3. Proxy creates: `new RealImage(fileName)`
4. RealImage constructor runs expensive operations
5. Proxy forwards: `realImage.display()`

#### 3. Subsequent display() Calls

```java
image1.display();
```

**What happens**:
1. Proxy checks: `if (realImage == null)`
2. Condition is false (already created)
3. Proxy directly forwards: `realImage.display()`
4. No expensive operations!

#### 4. Unused Proxy

```java
IImage image2 = new ImageProxy("photo.png");
// Never call display()
```

**What happens**:
- Proxy is created (lightweight)
- RealImage is **never created**
- **No memory wasted**
- **No expensive operations**

**This is the power of Virtual Proxy!**

---

## 📐 Standard UML Diagram

### Complete Proxy Pattern Structure
                ┌──────────────────────┐
                │        Client        │
                ├──────────────────────┤
                │ - dataService:       │
                │   IDataService       │
                └───────────┬──────────┘
                            │
                            │ uses
                            ▼
            ┌────────────────────────────────┐
            │       <<interface>>            │
            │        IDataService            │
            ├────────────────────────────────┤
            │ + fetchData(): Data            │
            └───────────▲───────────▲─────────┘
                        │           │
          implements    │           │ implements
                        │           │
        ┌───────────────┴───────┐  ┌──────────────────────┐
        │      DataProxy       │   │      DataService     │
        ├──────────────────────┤   ├──────────────────────┤
        │ - dataService:       │   │                      │
        │   DataService        │   │                      │
        ├──────────────────────┤   ├──────────────────────┤
        │ + fetchData(): Data  │──▶│ + fetchData(): Data  │
        └──────────────────────┘   └──────────────────────┘



**Components**:

1. **ISubject (Interface)**
   - Declares common operations
   - Both RealSubject and Proxy implement this

2. **RealSubject**
   - The actual object with real functionality
   - May be expensive, remote, or protected

3. **Proxy**
   - Has IS-A relationship with ISubject
   - Has HAS-A relationship with RealSubject
   - Controls access to RealSubject
   - Adds additional behavior (lazy loading, authentication, etc.)

**The Magic**:
- Client holds reference to ISubject
- Client doesn't know if it's Proxy or RealSubject
- Proxy decides when/how to use RealSubject

---

## 🔑 Key Concepts

### 1. Indistinguishable Behavior

**Core Principle**: Client cannot tell the difference between Proxy and Real Object.

```java
ISubject obj = new Proxy(); // or new RealSubject()
obj.operation(); // Works the same way!
```

**Why?** Both implement the same interface.

### 2. IS-A + HAS-A Relationships

**Unique Pattern**: Proxy has BOTH relationships with RealSubject.

```java
class Proxy implements ISubject {  // IS-A
    private RealSubject real;      // HAS-A
}
```

**Similar to**: Decorator Pattern (also has IS-A + HAS-A)

**Difference**:
- **Decorator**: Adds functionality
- **Proxy**: Controls access

### 3. Lazy Loading

**Concept**: Defer expensive operations until absolutely necessary.

**Benefits**:
- Faster application startup
- Reduced memory usage
- Better resource management

**Trade-off**: First access is slower

### 4. Types Summary

| Type | Purpose | When to Use |
|------|---------|-------------|
| **Virtual** | Lazy loading | Expensive object creation |
| **Protection** | Access control | Authentication/Authorization |
| **Remote** | Location transparency | Distributed systems |

### 5. Hybrid Proxies

**You can combine types!**

Example: Remote + Protection Proxy
```java
class SecureRemoteProxy implements IService {
    private RemoteService remote;
    private User user;
    
    public void operation() {
        // Protection check
        if (!user.isAuthenticated()) {
            throw new SecurityException();
        }
        
        // Remote call
        if (remote == null) {
            remote = connectToServer();
        }
        remote.operation();
    }
}
```

---

## 🌍 Real-World Use Cases

### 1. Virtual Proxy Examples ⭐

**Image Loading in Web Browsers**
```java
// Browser doesn't load all images immediately
// Uses proxy to load images as you scroll
ImageProxy thumbnail = new ImageProxy("large-image.jpg");
// Image loads only when visible on screen
```

**Database Connection Pools**
```java
// Don't create all connections upfront
// Create connections as needed
ConnectionProxy conn = new ConnectionProxy();
// Real connection created when first query runs
```

**Lazy Loading in ORMs (Hibernate, JPA)**
```java
@Entity
class Order {
    @OneToMany(fetch = FetchType.LAZY)
    List<OrderItem> items; // Proxy until accessed
}
```

### 2. Protection Proxy Examples ⭐

**Premium Features**
```java
// Netflix, Spotify premium features
class VideoStreamingProxy {
    public void playHDVideo() {
        if (!user.isPremium()) {
            throw new AccessDeniedException();
        }
        realPlayer.playHDVideo();
    }
}
```

**Admin Operations**
```java
// Only admins can delete users
class UserManagementProxy {
    public void deleteUser(int userId) {
        if (!currentUser.isAdmin()) {
            throw new UnauthorizedException();
        }
        realService.deleteUser(userId);
    }
}
```

**File System Permissions**
```java
// Operating system file access
class FileProxy {
    public void delete() {
        if (!hasWritePermission()) {
            throw new SecurityException();
        }
        realFile.delete();
    }
}
```

### 3. Remote Proxy Examples ⭐

**Microservices Architecture**
```
┌──────────────┐         ┌──────────────┐
│ Cart Service │         │ User Service │
│              │         │  (Remote)    │
│  UserProxy   │────────→│              │
└──────────────┘  HTTP   └──────────────┘
   (Local)                  (Server 2)
```

**RMI (Remote Method Invocation)**
```java
// Java RMI uses proxy pattern
RemoteService proxy = (RemoteService) 
    Naming.lookup("rmi://server/Service");
proxy.remoteMethod(); // Calls over network
```

**Web Services / REST APIs**
```java
// Spring RestTemplate, Feign clients
@FeignClient(name = "user-service")
interface UserServiceProxy {
    @GetMapping("/users/{id}")
    User getUser(@PathVariable int id);
}
// Proxy handles HTTP communication
```

### 4. Framework Examples

**Spring Framework**
- **AOP Proxies**: Method interception
- **Transaction Management**: @Transactional uses proxies
- **Security**: @PreAuthorize uses proxies

**Hibernate**
- **Lazy Loading**: Collections loaded via proxies
- **Proxy Pattern**: For entity relationships

**Java Dynamic Proxies**
```java
Proxy.newProxyInstance(
    classLoader,
    interfaces,
    invocationHandler
);
```

---

## 🎓 Key Takeaways

### When to Use Proxy Pattern

✅ **Use when**:
1. You need to **control access** to an object
2. Object creation is **expensive**
3. Object is on a **remote server**
4. You need **lazy initialization**
5. You need **access control/authentication**
6. You want to add behavior **without modifying** the original class

❌ **Don't use when**:
1. Direct access is sufficient
2. No need for access control
3. Object creation is cheap
4. Adds unnecessary complexity

### Advantages

1. **Controlled Access**: Manage how objects are accessed
2. **Lazy Loading**: Create objects only when needed
3. **Security**: Add authentication/authorization
4. **Location Transparency**: Hide network complexity
5. **Open-Closed Principle**: Add behavior without modifying original

### Disadvantages

1. **Increased Complexity**: Extra layer of indirection
2. **Performance Overhead**: Additional method calls
3. **Delayed Response**: First access may be slower (lazy loading)

### Design Principles Applied

1. **Open-Closed Principle**: Open for extension, closed for modification
2. **Single Responsibility**: Proxy handles access control, Real object handles business logic
3. **Dependency Inversion**: Both depend on abstraction (interface)

---

## ❓ Interview Q&A

### Q1: What is the Proxy Design Pattern? Explain with an example.

**Answer**:

The Proxy Design Pattern provides a **surrogate or placeholder** for another object to control access to it.

**Key Components**:
1. **Subject Interface**: Common interface for both Proxy and RealSubject
2. **RealSubject**: The actual object with real functionality
3. **Proxy**: Controls access to RealSubject

**Example**: Image Loading (Virtual Proxy)

```java
interface IImage {
    void display();
}

class RealImage implements IImage {
    private String fileName;
    
    public RealImage(String fileName) {
        this.fileName = fileName;
        loadFromDisk(); // Expensive!
    }
    
    public void display() {
        System.out.println("Displaying: " + fileName);
    }
}

class ImageProxy implements IImage {
    private String fileName;
    private RealImage realImage; // Lazy loading
    
    public ImageProxy(String fileName) {
        this.fileName = fileName;
    }
    
    public void display() {
        if (realImage == null) {
            realImage = new RealImage(fileName);
        }
        realImage.display();
    }
}
```

**Benefit**: RealImage is created only when `display()` is called, not during proxy creation.

---

### Q2: What are the different types of Proxy? Explain each.

**Answer**:

There are **three main types** of Proxy:

**1. Virtual Proxy** (Lazy Loading)
- **Purpose**: Delay expensive object creation
- **Use Case**: Loading large images, database connections
- **Example**: Image viewer that loads images only when displayed

**2. Protection Proxy** (Access Control)
- **Purpose**: Control access based on permissions
- **Use Case**: Premium features, admin operations
- **Example**: PDF unlocker available only to premium users

**3. Remote Proxy** (Location Transparency)
- **Purpose**: Represent objects in different address spaces
- **Use Case**: Microservices, distributed systems
- **Example**: Local proxy for remote service over network

**Common Pattern**: All three share the same structure (IS-A + HAS-A relationships), but differ in **intent**.

---

### Q3: What is the difference between Proxy and Decorator patterns?

**Answer**:

Both patterns have **IS-A + HAS-A** relationships, but serve different purposes.

| Aspect | Proxy Pattern | Decorator Pattern |
|--------|---------------|-------------------|
| **Purpose** | Control access | Add functionality |
| **Intent** | Manage object lifecycle | Enhance behavior |
| **Creation** | Proxy creates real object | Decorator wraps existing object |
| **Focus** | Access control, lazy loading | Adding responsibilities |
| **Example** | Image lazy loading | Adding borders to images |

**Proxy**:
```java
class ImageProxy {
    private RealImage real; // Controls when to create
    
    public void display() {
        if (real == null) {
            real = new RealImage(); // Lazy creation
        }
        real.display();
    }
}
```

**Decorator**:
```java
class BorderDecorator {
    private Image image; // Already exists
    
    public void display() {
        drawBorder();    // Add functionality
        image.display(); // Delegate
    }
}
```

**Key Difference**: 
- **Proxy**: "Should I create/allow access?"
- **Decorator**: "What extra can I add?"

---

### Q4: How does Virtual Proxy implement lazy loading?

**Answer**:

**Lazy Loading**: Defer object creation until the object is actually needed.

**Implementation Steps**:

**Step 1**: Proxy holds a reference (initially null)
```java
class ImageProxy {
    private RealImage realImage = null; // Not created yet
}
```

**Step 2**: Check on method call
```java
public void display() {
    if (realImage == null) {
        // First time - create now
        realImage = new RealImage(fileName);
    }
    realImage.display();
}
```

**Step 3**: Subsequent calls use existing object
```java
// Second call
display(); // realImage != null, just forward
```

**Benefits**:
1. **Memory Efficient**: Object created only if used
2. **Faster Startup**: No upfront expensive operations
3. **Resource Management**: Better utilization

**Trade-off**: First method call is slower (creation overhead)

**Real-World Example**: Hibernate lazy loading
```java
@OneToMany(fetch = FetchType.LAZY)
List<Order> orders; // Loaded only when accessed
```

---

### Q5: What is a Protection Proxy? Provide a real-world example.

**Answer**:

**Protection Proxy**: Controls access to an object based on **access rights** or **permissions**.

**Purpose**: 
- Authentication
- Authorization
- Access control

**Real-World Example**: Premium Feature Access

```java
class User {
    String name;
    boolean isPremium;
}

interface IDocumentReader {
    void unlockPDF(String file, String password);
}

class RealDocumentReader implements IDocumentReader {
    public void unlockPDF(String file, String password) {
        System.out.println("Unlocking: " + file);
        // Actual unlock logic
    }
}

class DocumentProxy implements IDocumentReader {
    private User user;
    private RealDocumentReader realReader;
    
    public DocumentProxy(User user) {
        this.user = user;
        this.realReader = new RealDocumentReader();
    }
    
    public void unlockPDF(String file, String password) {
        // Protection check
        if (user.isPremium) {
            realReader.unlockPDF(file, password);
        } else {
            throw new SecurityException(
                "Premium feature - upgrade to access!"
            );
        }
    }
}
```

**Usage**:
```java
User freeUser = new User("John", false);
IDocumentReader reader = new DocumentProxy(freeUser);
reader.unlockPDF("doc.pdf", "pass"); // ❌ Exception!

User premiumUser = new User("Jane", true);
reader = new DocumentProxy(premiumUser);
reader.unlockPDF("doc.pdf", "pass"); // ✅ Works!
```

**Other Examples**:
- Admin-only operations
- File system permissions
- Role-based access control (RBAC)

---

### Q6: What is a Remote Proxy? How is it used in microservices?

**Answer**:

**Remote Proxy**: Represents an object that exists in a **different address space** (different server, process, or network location).

**Purpose**:
- Hide network complexity
- Provide location transparency
- Handle communication details

**How It Works**:

```
┌──────────────┐         ┌──────────────┐
│ Client       │         │ Real Service │
│              │         │  (Remote)    │
│  Proxy       │────────→│              │
│  (Local)     │  HTTP   │              │
└──────────────┘         └──────────────┘
   Server A                 Server B
```

**Example**: Microservices Architecture

```java
// Interface
interface IUserService {
    User getUser(int id);
}

// Real Service (on remote server)
class UserService implements IUserService {
    public User getUser(int id) {
        // Fetch from database
        return database.findUser(id);
    }
}

// Remote Proxy (local)
class UserServiceProxy implements IUserService {
    private UserService remoteService;
    
    public User getUser(int id) {
        // Lazy connection
        if (remoteService == null) {
            // Establish network connection
            remoteService = connectToServer("192.168.1.100");
        }
        
        // Make remote call
        return remoteService.getUser(id);
    }
    
    private UserService connectToServer(String ip) {
        // HTTP/gRPC/RMI connection logic
        return new UserService();
    }
}
```

**Client Code**:
```java
// Client doesn't know it's remote!
IUserService service = new UserServiceProxy();
User user = service.getUser(123); // Looks like local call
```

**Real-World Examples**:
- **Spring Cloud Feign**: REST client proxy
- **gRPC**: Generated client stubs
- **Java RMI**: Remote method invocation
- **CORBA**: Distributed object systems

---

### Q7: What are the relationships in Proxy Pattern?

**Answer**:

Proxy Pattern has **two key relationships**:

**1. IS-A Relationship** (Inheritance)
- Proxy IS-A Subject (implements interface)
- RealSubject IS-A Subject (implements interface)

**2. HAS-A Relationship** (Composition)
- Proxy HAS-A RealSubject (holds reference)

**Diagram**:
```
        ISubject
           △
           │
      ┌────┴────┐
      │         │
   Proxy    RealSubject
      │
      │ HAS-A
      └─────────→ RealSubject
```

**Code**:
```java
class Proxy implements ISubject {  // IS-A
    private RealSubject real;      // HAS-A
    
    public void operation() {
        real.operation(); // Delegates
    }
}
```

**Why Both?**
- **IS-A**: Ensures Proxy can substitute RealSubject (Liskov Substitution)
- **HAS-A**: Allows Proxy to control/delegate to RealSubject

**Similar Pattern**: Decorator also has IS-A + HAS-A

---

### Q8: How does Proxy Pattern follow SOLID principles?

**Answer**:

**1. Single Responsibility Principle (SRP)** ✅
- **Proxy**: Handles access control, lazy loading
- **RealSubject**: Handles business logic
- Each has one reason to change

**2. Open-Closed Principle (OCP)** ✅
- Add new proxy types without modifying RealSubject
- Example: Add logging proxy, caching proxy

```java
class LoggingProxy implements ISubject {
    private RealSubject real;
    
    public void operation() {
        log("Before operation");
        real.operation();
        log("After operation");
    }
}
```

**3. Liskov Substitution Principle (LSP)** ✅
- Proxy can replace RealSubject anywhere
- Both implement same interface

```java
ISubject obj = new Proxy(); // or new RealSubject()
obj.operation(); // Works either way
```

**4. Interface Segregation Principle (ISP)** ✅
- Clients depend on ISubject interface
- Not on concrete implementations

**5. Dependency Inversion Principle (DIP)** ✅
- Both Proxy and RealSubject depend on ISubject abstraction
- Not on each other directly

**Overall**: Proxy Pattern aligns very well with SOLID principles!

---

### Q9: What are the advantages and disadvantages of Proxy Pattern?

**Answer**:

**Advantages** ✅:

1. **Controlled Access**
   - Manage how and when objects are accessed
   - Add security layers

2. **Lazy Initialization**
   - Create expensive objects only when needed
   - Better memory management

3. **Location Transparency**
   - Hide network/remote complexity
   - Client doesn't know object is remote

4. **Security**
   - Add authentication/authorization
   - Protect sensitive resources

5. **Open-Closed Principle**
   - Add proxies without modifying original class
   - Easy to extend

**Disadvantages** ❌:

1. **Increased Complexity**
   - Extra layer of indirection
   - More classes to maintain

2. **Performance Overhead**
   - Additional method calls
   - Slight performance cost

3. **Delayed Response**
   - First access slower (lazy loading)
   - May not be suitable for real-time systems

4. **Debugging Difficulty**
   - Flow goes through multiple layers
   - Harder to trace issues

**Trade-off**: Complexity vs. Control

---

### Q10: When should you use Proxy Pattern?

**Answer**:

✅ **Use Proxy Pattern when**:

**1. Expensive Object Creation**
```java
// Large images, database connections
ImageProxy proxy = new ImageProxy("huge-file.jpg");
// Create real object only when needed
```

**2. Access Control Needed**
```java
// Premium features, admin operations
if (!user.isPremium()) {
    throw new AccessDeniedException();
}
```

**3. Remote Object Access**
```java
// Microservices, distributed systems
UserServiceProxy proxy = new UserServiceProxy();
// Handles network communication
```

**4. Lazy Loading Required**
```java
// Hibernate, JPA lazy loading
@OneToMany(fetch = FetchType.LAZY)
List<Order> orders; // Proxy until accessed
```

**5. Additional Behavior Without Modification**
```java
// Logging, caching, validation
// Without changing original class
```

❌ **Don't use when**:

1. **Direct Access is Sufficient**
   - No need for control/protection
   - Object creation is cheap

2. **Performance is Critical**
   - Extra layer adds overhead
   - Real-time systems

3. **Simple Use Case**
   - Overhead not justified
   - Adds unnecessary complexity

**Decision Criteria**:
- Do you need to control access? → Use Proxy
- Is object creation expensive? → Use Virtual Proxy
- Is object remote? → Use Remote Proxy
- Need authentication? → Use Protection Proxy

---

### Q11: Can you combine different types of Proxies?

**Answer**:

**Yes!** You can create **hybrid proxies** that combine multiple types.

**Example**: Remote + Protection Proxy

```java
class SecureRemoteProxy implements IService {
    private User user;
    private RemoteService remoteService;
    
    public void operation() {
        // Protection Proxy behavior
        if (!user.isAuthenticated()) {
            throw new SecurityException("Not authenticated!");
        }
        
        if (!user.hasPermission("EXECUTE")) {
            throw new AccessDeniedException("No permission!");
        }
        
        // Remote Proxy behavior
        if (remoteService == null) {
            // Lazy connection
            remoteService = connectToServer();
        }
        
        // Forward to remote service
        remoteService.operation();
    }
}
```

**Other Combinations**:

**Virtual + Protection**:
```java
class LazySecureProxy {
    public void operation() {
        // Check permissions first
        if (!hasAccess()) throw new SecurityException();
        
        // Then lazy load
        if (real == null) real = new RealObject();
        real.operation();
    }
}
```

**Remote + Virtual**:
```java
class LazyRemoteProxy {
    public void operation() {
        // Lazy connection
        if (connection == null) {
            connection = establishConnection();
        }
        
        // Lazy object creation
        if (remoteObject == null) {
            remoteObject = fetchRemoteObject();
        }
        
        remoteObject.operation();
    }
}
```

**Benefits**:
- Flexible design
- Combine concerns
- Reusable components

---

### Q12: How is Proxy Pattern used in Spring Framework?

**Answer**:

Spring Framework **heavily uses** Proxy Pattern in multiple areas:

**1. AOP (Aspect-Oriented Programming)**
```java
@Aspect
class LoggingAspect {
    @Before("execution(* com.example.service.*.*(..))")
    public void logBefore() {
        System.out.println("Method called");
    }
}
```
- Spring creates **dynamic proxy** for service classes
- Proxy intercepts method calls
- Executes aspect logic before/after

**2. Transaction Management**
```java
@Transactional
public void transferMoney(Account from, Account to, double amount) {
    // Business logic
}
```
- Spring creates **proxy** for class with `@Transactional`
- Proxy manages transaction lifecycle
- Begins transaction, commits/rolls back

**3. Security**
```java
@PreAuthorize("hasRole('ADMIN')")
public void deleteUser(int userId) {
    // Only admins can execute
}
```
- Spring Security uses **proxy**
- Checks permissions before method execution
- Protection Proxy pattern!

**4. Lazy Loading**
```java
@Lazy
@Component
class ExpensiveService {
    // Created only when first accessed
}
```
- Virtual Proxy pattern
- Bean created on first use

**5. Remote Services (Spring Cloud)**
```java
@FeignClient(name = "user-service")
interface UserClient {
    @GetMapping("/users/{id}")
    User getUser(@PathVariable int id);
}
```
- Feign creates **remote proxy**
- Handles HTTP communication
- Remote Proxy pattern!

**How Spring Creates Proxies**:
- **JDK Dynamic Proxy**: For interfaces
- **CGLIB Proxy**: For classes

---

## 📝 Summary

The **Proxy Design Pattern** provides a surrogate or placeholder for another object to control access to it.

**Key Points**:
- **Three Types**: Virtual (lazy loading), Protection (access control), Remote (location transparency)
- **Two Relationships**: IS-A (inheritance) + HAS-A (composition)
- **Core Principle**: Client cannot differentiate between Proxy and Real Object
- **Common Structure**: All three types share the same UML structure

**When to Use**:
- Expensive object creation → Virtual Proxy
- Access control needed → Protection Proxy
- Remote object access → Remote Proxy

**Real-World Examples**:
- Image lazy loading (Virtual)
- Premium features (Protection)
- Microservices (Remote)
- Spring AOP, Transactions, Security (All types)

**Remember**: If you need to control access to an object, think Proxy Pattern!

---

**Happy Learning! 🚀**
