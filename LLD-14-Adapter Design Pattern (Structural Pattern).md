# LLD-14: Adapter Design Pattern (Structural Pattern)

---

## 📋 Table of Contents
- [Introduction](#introduction)
- [Real-Life Analogy](#real-life-analogy)
- [Problem Statement](#problem-statement)
- [Solution: Adapter Pattern](#solution-adapter-pattern)
- [Example Scenario](#example-scenario)
- [UML Diagrams](#uml-diagrams)
- [Implementation](#implementation)
- [Types of Adapters](#types-of-adapters)
- [Real-World Use Cases](#real-world-use-cases)
- [Key Takeaways](#key-takeaways)
- [Interview Q&A](#interview-qa)

---

## 🎯 Introduction

The **Adapter Design Pattern** is a structural design pattern that allows incompatible interfaces to work together. It acts as a bridge between two incompatible interfaces by converting the interface of one class into another interface that clients expect.

**Definition:**
> "The Adapter pattern converts the interface of a class into another interface that clients expect. Adapter lets classes work together that couldn't otherwise because of incompatible interfaces."

---

## 🔌 Real-Life Analogy

### Example 1: International Travel Adapter
- You're in a hotel room in the US with **US-type sockets**
- You have an **Indian charger** that doesn't fit
- **Solution:** Use an international adapter that:
  - Accepts your Indian charger plug
  - Plugs into the US socket
  - Acts as a bridge between incompatible interfaces

### Example 2: USB Type Conversion
- You have a **Type-C cable** for your phone
- Your charger accepts **Type-A** only
- **Solution:** Use a Type-C to Type-A adapter

### Puzzle Piece Analogy

```
┌─────────────┐         ┌─────────────┐
│  Existing   │         │   Third     │
│    Code     │    ✗    │   Party     │
│             │         │   Library   │
└─────────────┘         └─────────────┘
     (Piece A)              (Piece B)
   
   These pieces don't fit together!

┌─────────────┐    ┌──────────┐    ┌─────────────┐
│  Existing   │────│ Adapter  │────│   Third     │
│    Code     │    │          │    │   Party     │
│             │    │          │    │   Library   │
└─────────────┘    └──────────┘    └─────────────┘
     (Piece A)      (Bridge)         (Piece B)
   
   Now they fit perfectly!
```

---

## ❓ Problem Statement

### Scenario: Third-Party Integration

When developing applications, you often need to integrate third-party libraries or legacy code:

```
┌──────────────────┐
│  Your Existing   │
│   Application    │
│      Code        │
└──────────────────┘
         │
         │ Direct coupling (BAD!)
         ↓
┌──────────────────┐
│   Third Party    │
│     Library      │
│   (Vendor A)     │
└──────────────────┘
```

**Problems with Direct Coupling:**
1. **Tight Coupling:** Your code becomes dependent on the third-party library
2. **Difficult to Switch:** Changing vendors requires modifying your existing code
3. **Incompatible Interfaces:** Different libraries have different method signatures and return types
4. **Maintenance Nightmare:** Updates to the library break your code

### Better Approach: Introduce an Adapter

```
┌──────────────────┐
│  Your Existing   │
│   Application    │
│      Code        │
└──────────────────┘
         │
         │ Talks to
         ↓
    ┌─────────┐
    │ Adapter │ ← Decouples your code from vendor
    └─────────┘
         │
         │ Talks to
         ↓
┌──────────────────┐
│   Third Party    │
│     Library      │
│   (Vendor A)     │
└──────────────────┘
```

**Benefits:**
- Your existing code doesn't know which third-party library is being used
- Easy to switch vendors by just changing the adapter
- Loose coupling between your code and external dependencies

---

## 💡 Solution: Adapter Pattern

### How It Works

The Adapter pattern introduces a middle layer that:
1. **Inherits** from the target interface (IS-A relationship)
2. **Composes** the adaptee (HAS-A relationship)
3. **Translates** calls from the target interface to the adaptee's interface

### Flow Diagram

```
┌─────────────────────────────────────────────────────────┐
│                    Client Code                          │
│  (Expects Target Interface)                             │
└────────────────────┬────────────────────────────────────┘
                     │
                     │ Uses
                     ↓
            ┌─────────────────┐
            │ Target Interface│
            │  (IReports)     │
            │ + getJsonData() │
            └─────────────────┘
                     △
                     │ Implements (IS-A)
                     │
            ┌─────────────────────────────┐
            │        Adapter              │
            │ (XMLDataProviderAdapter)    │
            │ + getJsonData()             │
            │ - provider: XMLDataProvider │
            └─────────────────────────────┘
                     │
                     │ HAS-A (Composition)
                     ↓
            ┌─────────────────┐
            │     Adaptee     │
            │(XMLDataProvider)│
            │ + getXMLData()  │
            └─────────────────┘
```

---

## 📝 Example Scenario

### Requirement: Report Generation System

**Client Expectation:**
- Client needs reports in **JSON format**
- Client calls `getJsonData()` method

**Available Resource:**
- A third-party library that provides data in **XML format**
- Library has `getXMLData()` method

**Challenge:**
- Client expects JSON, but library provides XML
- Incompatible interfaces

**Solution:**
- Create an **Adapter** that:
  1. Implements the `IReports` interface (what client expects)
  2. Uses `XMLDataProvider` internally (HAS-A relationship)
  3. Converts XML to JSON

---

## 🏗️ UML Diagrams

### Class Diagram

```
┌─────────────────────────────────┐
│      <<interface>>              │
│        IReports                 │
├─────────────────────────────────┤
│ + getJsonData(rawData: String)  │
│   : String                      │
└─────────────────────────────────┘
              △
              │ implements
              │
┌─────────────────────────────────────────┐
│   XMLDataProviderAdapter                │
├─────────────────────────────────────────┤
│ - provider: XMLDataProvider             │
├─────────────────────────────────────────┤
│ + XMLDataProviderAdapter(provider)      │
│ + getJsonData(rawData: String): String  │
└─────────────────────────────────────────┘
              │
              │ HAS-A (composition)
              ↓
┌─────────────────────────────────┐
│    XMLDataProvider              │
├─────────────────────────────────┤
│ + getXMLData(rawData: String)   │
│   : String                      │
└─────────────────────────────────┘


┌─────────────────┐
│     Client      │
├─────────────────┤
│ + getReport()   │
└─────────────────┘
         │
         │ uses
         ↓
    IReports
```

### Sequence Diagram

```
Client          Adapter              Adaptee
  │               │                    │
  │ getJsonData() │                    │
  │──────────────>│                    │
  │               │                    │
  │               │  getXMLData()      │
  │               │───────────────────>│
  │               │                    │
  │               │  XML Data          │
  │               │<───────────────────│
  │               │                    │
  │               │ Convert XML to JSON│
  │               │────────┐           │
  │               │        │           │
  │               │<───────┘           │
  │               │                    │
  │  JSON Data    │                    │
  │<──────────────│                    │
  │               │                    │
```

---

## 💻 Implementation

### Step 1: Target Interface (What Client Expects)

```java
// IReports.java
public interface IReports {
    String getJsonData(String rawData);
}
```

**Explanation:**
- This is the interface that the **client expects**
- Client wants JSON data from raw input
- This is our **Target Interface**

---

### Step 2: Adaptee (Third-Party Library)

```java
// XMLDataProvider.java
public class XMLDataProvider {
    
    public String getXMLData(String rawData) {
        // Assuming rawData format: "name:id"
        // Example: "Alice:42"
        
        String[] parts = rawData.split(":");
        String name = parts[0];
        String id = parts[1];
        
        // Convert to XML format
        String xmlData = "<data>\n" +
                        "  <name>" + name + "</name>\n" +
                        "  <id>" + id + "</id>\n" +
                        "</data>";
        
        return xmlData;
    }
}
```

**Explanation:**
- This is the **third-party library** (Adaptee)
- It provides data in **XML format**
- Has a different interface than what client expects
- We cannot modify this class (it's external)

---

### Step 3: Adapter (Bridge Between Target and Adaptee)

```java
// XMLDataProviderAdapter.java
public class XMLDataProviderAdapter implements IReports {
    
    private XMLDataProvider provider;
    
    // Constructor injection
    public XMLDataProviderAdapter(XMLDataProvider provider) {
        this.provider = provider;
    }
    
    @Override
    public String getJsonData(String rawData) {
        // Step 1: Get XML data from the adaptee
        String xmlData = provider.getXMLData(rawData);
        
        // Step 2: Convert XML to JSON
        String jsonData = convertXMLToJSON(xmlData);
        
        // Step 3: Return JSON data
        return jsonData;
    }
    
    private String convertXMLToJSON(String xmlData) {
        // Extract name from XML
        int nameStart = xmlData.indexOf("<name>") + 6;
        int nameEnd = xmlData.indexOf("</name>");
        String name = xmlData.substring(nameStart, nameEnd);
        
        // Extract id from XML
        int idStart = xmlData.indexOf("<id>") + 4;
        int idEnd = xmlData.indexOf("</id>");
        String id = xmlData.substring(idStart, idEnd);
        
        // Create JSON format
        String jsonData = "{\n" +
                         "  \"name\": \"" + name + "\",\n" +
                         "  \"id\": \"" + id + "\"\n" +
                         "}";
        
        return jsonData;
    }
}
```

**Explanation:**
- **IS-A Relationship:** Implements `IReports` (so it can be used wherever `IReports` is expected)
- **HAS-A Relationship:** Contains `XMLDataProvider` (composition)
- **Translation Logic:** Converts XML to JSON in the `getJsonData()` method

**Key Points:**
1. Client calls `getJsonData()` on the adapter
2. Adapter internally calls `getXMLData()` on the adaptee
3. Adapter converts XML to JSON
4. Adapter returns JSON to the client

---

### Step 4: Client Code

```java
// Client.java
public class Client {
    
    public void getReport(IReports report, String rawData) {
        // Client only knows about IReports interface
        // Client doesn't care about the implementation details
        String jsonData = report.getJsonData(rawData);
        System.out.println("Report Data:");
        System.out.println(jsonData);
    }
}
```

**Explanation:**
- Client works with the **Target Interface** (`IReports`)
- Client has **no knowledge** of:
  - Third-party library (`XMLDataProvider`)
  - Adapter implementation
  - XML to JSON conversion
- Client just calls `getJsonData()` and gets JSON

---

### Step 5: Main Application

```java
// Main.java
public class Main {
    public static void main(String[] args) {
        // Step 1: Create the adaptee (third-party library)
        XMLDataProvider adaptee = new XMLDataProvider();
        
        // Step 2: Create the adapter (wraps the adaptee)
        IReports adapter = new XMLDataProviderAdapter(adaptee);
        
        // Step 3: Prepare raw data
        String rawData = "Alice:42";
        
        // Step 4: Client uses the adapter
        Client client = new Client();
        client.getReport(adapter, rawData);
    }
}
```

**Output:**
```
Report Data:
{
  "name": "Alice",
  "id": "42"
}
```

**Execution Flow:**
1. Client calls `adapter.getJsonData("Alice:42")`
2. Adapter calls `adaptee.getXMLData("Alice:42")`
3. Adaptee returns XML:
   ```xml
   <data>
     <name>Alice</name>
     <id>42</id>
   </data>
   ```
4. Adapter converts XML to JSON
5. Adapter returns JSON to client

---

## 🔄 Types of Adapters

### 1. Object Adapter (Recommended) ✅

**Uses Composition (HAS-A relationship)**

```
┌─────────────────┐
│     Target      │
│   (IReports)    │
└─────────────────┘
        △
        │ implements
        │
┌─────────────────────┐
│      Adapter        │
│ - adaptee: Adaptee  │ ←── HAS-A (Composition)
└─────────────────────┘
        │
        │ uses
        ↓
┌─────────────────┐
│     Adaptee     │
│(XMLDataProvider)│
└─────────────────┘
```

**Implementation:**
```java
public class XMLDataProviderAdapter implements IReports {
    private XMLDataProvider provider; // Composition
    
    public XMLDataProviderAdapter(XMLDataProvider provider) {
        this.provider = provider;
    }
    
    @Override
    public String getJsonData(String rawData) {
        String xmlData = provider.getXMLData(rawData);
        return convertXMLToJSON(xmlData);
    }
}
```

**Advantages:**
- ✅ Follows "Composition over Inheritance" principle
- ✅ More flexible
- ✅ Can adapt multiple adaptees
- ✅ Works in languages that don't support multiple inheritance (Java)

---

### 2. Class Adapter (Not Recommended) ❌

**Uses Multiple Inheritance (IS-A relationship)**

```
┌─────────────────┐         ┌─────────────────┐
│     Target      │         │     Adaptee     │
│   (IReports)    │         │(XMLDataProvider)│
└─────────────────┘         └─────────────────┘
        △                           △
        │ extends                   │ extends
        └───────────┬───────────────┘
                    │
            ┌───────────────┐
            │    Adapter    │
            └───────────────┘
```

**Implementation:**
```java
// This requires multiple inheritance
// NOT SUPPORTED IN JAVA!
public class XMLDataProviderAdapter 
    extends XMLDataProvider    // Inherit from Adaptee
    implements IReports {      // Implement Target
    
    @Override
    public String getJsonData(String rawData) {
        String xmlData = getXMLData(rawData); // Inherited method
        return convertXMLToJSON(xmlData);
    }
}
```

**Disadvantages:**
- ❌ Requires multiple inheritance (not supported in Java)
- ❌ Only works in C++
- ❌ Violates "Composition over Inheritance" principle
- ❌ Less flexible
- ❌ Tight coupling

**Conclusion:** Always prefer **Object Adapter** over **Class Adapter**

---

## 🌍 Real-World Use Cases

### 1. Third-Party Library Integration

**Scenario:** Payment Gateway Integration

```
┌──────────────────┐
│  Your E-commerce │
│   Application    │
└──────────────────┘
         │
         ↓
    ┌─────────┐
    │ Payment │
    │ Adapter │
    └─────────┘
         │
         ├──────────────┬──────────────┐
         ↓              ↓              ↓
    ┌─────────┐   ┌─────────┐   ┌─────────┐
    │ Razorpay│   │ Stripe  │   │ PayPal  │
    └─────────┘   └─────────┘   └─────────┘
```

**Example:**
```java
// Your application expects this interface
public interface IPaymentGateway {
    boolean processPayment(double amount);
}

// Razorpay has different methods
public class RazorpayAPI {
    public String initiateTransaction(int amountInPaise) {
        // Razorpay specific implementation
    }
}

// Adapter bridges the gap
public class RazorpayAdapter implements IPaymentGateway {
    private RazorpayAPI razorpay;
    
    @Override
    public boolean processPayment(double amount) {
        int amountInPaise = (int)(amount * 100);
        String result = razorpay.initiateTransaction(amountInPaise);
        return result.equals("SUCCESS");
    }
}
```

**Benefits:**
- Switch from Razorpay to Stripe by just changing the adapter
- Your application code remains unchanged
- Loose coupling

---

### 2. Legacy Code Integration

**Scenario:** Modern Application + Legacy System

```
┌──────────────────┐
│  Modern App      │
│  (Java 17+)      │
└──────────────────┘
         │
         ↓
    ┌─────────┐
    │ Legacy  │
    │ Adapter │
    └─────────┘
         │
         ↓
┌──────────────────┐
│  Legacy System   │
│  (Java 6)        │
└──────────────────┘
```

**Example:**
```java
// Modern interface
public interface IUserService {
    User getUserById(String userId);
}

// Legacy system
public class LegacyUserDatabase {
    public String[] fetchUser(int id) {
        // Returns array: [name, email, age]
    }
}

// Adapter
public class LegacyUserAdapter implements IUserService {
    private LegacyUserDatabase legacyDb;
    
    @Override
    public User getUserById(String userId) {
        int id = Integer.parseInt(userId);
        String[] userData = legacyDb.fetchUser(id);
        
        // Convert legacy format to modern User object
        return new User(userData[0], userData[1], userData[2]);
    }
}
```

---

### 3. Notification System

**Scenario:** Multiple Notification Channels

```java
// Target interface
public interface INotification {
    void send(String message, String recipient);
}

// WhatsApp API (third-party)
public class WhatsAppAPI {
    public void sendWhatsAppMessage(String phoneNumber, String text) {
        // WhatsApp specific implementation
    }
}

// Email API (third-party)
public class EmailAPI {
    public void sendEmail(String to, String subject, String body) {
        // Email specific implementation
    }
}

// WhatsApp Adapter
public class WhatsAppAdapter implements INotification {
    private WhatsAppAPI whatsapp;
    
    @Override
    public void send(String message, String recipient) {
        whatsapp.sendWhatsAppMessage(recipient, message);
    }
}

// Email Adapter
public class EmailAdapter implements INotification {
    private EmailAPI email;
    
    @Override
    public void send(String message, String recipient) {
        email.sendEmail(recipient, "Notification", message);
    }
}

// Client code
public class NotificationService {
    public void notify(INotification channel, String msg, String user) {
        channel.send(msg, user);
    }
}
```

---

### 4. Database Driver Adapters

**Real-world example:** JDBC (Java Database Connectivity)

```java
// JDBC provides a standard interface
Connection conn = DriverManager.getConnection(url);

// Behind the scenes, different adapters:
// - MySQL Driver Adapter
// - PostgreSQL Driver Adapter
// - Oracle Driver Adapter

// Each adapter implements the same JDBC interface
// but talks to different database systems
```

---

## 🎯 Key Takeaways

### When to Use Adapter Pattern

✅ **Use Adapter Pattern when:**
1. You want to use an existing class with an incompatible interface
2. You need to integrate third-party libraries
3. You want to create reusable code that works with unrelated classes
4. You need to work with legacy code
5. You want to decouple your code from external dependencies

❌ **Don't Use Adapter Pattern when:**
1. You can modify the source code directly
2. The interfaces are already compatible
3. You're adding unnecessary complexity

---

### Design Principles Applied

1. **Open/Closed Principle:** Open for extension (new adapters), closed for modification (existing code)
2. **Dependency Inversion:** Depend on abstractions (IReports), not concretions
3. **Single Responsibility:** Adapter has one job - translate interfaces
4. **Composition over Inheritance:** Use Object Adapter (composition) over Class Adapter (inheritance)

---

### Standard UML Structure

```
┌─────────────────┐
│  <<interface>>  │
│     Target      │
├─────────────────┤
│ + request()     │
└─────────────────┘
        △
        │ implements
        │
┌─────────────────────┐
│      Adapter        │
├─────────────────────┤
│ - adaptee: Adaptee  │
├─────────────────────┤
│ + request()         │
└─────────────────────┘
        │
        │ HAS-A
        ↓
┌─────────────────┐
│     Adaptee     │
├─────────────────┤
│ + specificReq() │
└─────────────────┘


┌─────────┐
│ Client  │ ──uses──> Target
└─────────┘
```

**Components:**
1. **Target:** Interface that client expects
2. **Adapter:** Implements Target, contains Adaptee
3. **Adaptee:** Existing class with incompatible interface
4. **Client:** Works with Target interface

---

## ❓ Interview Q&A

### Q1: What is the Adapter Design Pattern?

**Answer:**
The Adapter Design Pattern is a structural pattern that allows incompatible interfaces to work together. It acts as a bridge between two incompatible interfaces by wrapping an existing class with a new interface.

**Key Points:**
- Converts one interface to another
- Enables classes with incompatible interfaces to collaborate
- Uses composition (Object Adapter) or inheritance (Class Adapter)

---

### Q2: What's the difference between Object Adapter and Class Adapter?

**Answer:**

| Aspect | Object Adapter | Class Adapter |
|--------|---------------|---------------|
| **Relationship** | Uses Composition (HAS-A) | Uses Multiple Inheritance (IS-A) |
| **Flexibility** | More flexible, can adapt multiple adaptees | Less flexible, tied to one adaptee |
| **Language Support** | Works in all OOP languages | Only works in C++ |
| **Coupling** | Loose coupling | Tight coupling |
| **Recommendation** | ✅ Preferred | ❌ Avoid |

**Example:**
```java
// Object Adapter (Recommended)
class Adapter implements Target {
    private Adaptee adaptee; // Composition
}

// Class Adapter (Not possible in Java)
class Adapter extends Adaptee implements Target {
    // Multiple inheritance
}
```

---

### Q3: When should you use the Adapter Pattern?

**Answer:**

**Use Adapter Pattern when:**
1. **Third-Party Integration:** Integrating libraries with different interfaces (payment gateways, notification services)
2. **Legacy Code:** Working with old code that can't be modified
3. **Interface Incompatibility:** Two classes need to work together but have incompatible interfaces
4. **Vendor Independence:** Want to easily switch between different vendors/libraries

**Example Scenario:**
```java
// You want to switch from Razorpay to Stripe
// Without Adapter: Change code everywhere
// With Adapter: Just create a new StripeAdapter
```

---

### Q4: What's the difference between Adapter and Decorator patterns?

**Answer:**

| Aspect | Adapter | Decorator |
|--------|---------|-----------|
| **Intent** | Change interface | Add functionality |
| **Purpose** | Make incompatible interfaces work together | Enhance object behavior |
| **Interface** | Converts one interface to another | Keeps same interface |
| **Example** | XML to JSON converter | Adding encryption to a stream |

```java
// Adapter: Changes interface
IReports adapter = new XMLDataProviderAdapter(xmlProvider);
String json = adapter.getJsonData(data); // Different interface

// Decorator: Same interface, added behavior
InputStream encrypted = new EncryptedInputStream(fileStream);
encrypted.read(); // Same interface, enhanced behavior
```

---

### Q5: Can you give a real-world example of Adapter Pattern in Java?

**Answer:**

**1. Java I/O Streams:**
```java
// InputStreamReader adapts InputStream to Reader
FileInputStream fis = new FileInputStream("file.txt");
InputStreamReader adapter = new InputStreamReader(fis);
// Now you can use Reader interface methods
```

**2. Arrays.asList():**
```java
// Adapts array to List interface
String[] array = {"A", "B", "C"};
List<String> list = Arrays.asList(array);
// Now you can use List interface methods
```

**3. JDBC Drivers:**
```java
// Different database drivers adapt to JDBC interface
Connection conn = DriverManager.getConnection(url);
// Works with MySQL, PostgreSQL, Oracle, etc.
```

---

### Q6: What are the advantages and disadvantages of Adapter Pattern?

**Answer:**

**Advantages:**
- ✅ **Single Responsibility:** Separates interface conversion from business logic
- ✅ **Open/Closed Principle:** Add new adapters without modifying existing code
- ✅ **Loose Coupling:** Client code independent of third-party libraries
- ✅ **Reusability:** Same adapter can be used in multiple places
- ✅ **Flexibility:** Easy to switch implementations

**Disadvantages:**
- ❌ **Complexity:** Adds extra layer of abstraction
- ❌ **Performance:** Slight overhead due to additional method calls
- ❌ **Over-engineering:** Can be overkill for simple scenarios

---

### Q7: How does Adapter Pattern help with vendor lock-in?

**Answer:**

**Without Adapter (Vendor Lock-in):**
```java
// Your code directly uses Razorpay
public class OrderService {
    private RazorpayAPI razorpay = new RazorpayAPI();
    
    public void processOrder() {
        razorpay.initiateTransaction(amount);
        // If you switch to Stripe, you need to change this code!
    }
}
```

**With Adapter (No Vendor Lock-in):**
```java
// Your code uses interface
public class OrderService {
    private IPaymentGateway gateway;
    
    public void processOrder() {
        gateway.processPayment(amount);
        // Switch vendors by just injecting different adapter!
    }
}

// Easy to switch
IPaymentGateway gateway = new RazorpayAdapter();
// OR
IPaymentGateway gateway = new StripeAdapter();
```

---

### Q8: Can Adapter Pattern be used with Dependency Injection?

**Answer:**

**Yes! This is a powerful combination:**

```java
// Spring Boot example
@Service
public class NotificationService {
    
    private final INotification notificationChannel;
    
    // Dependency Injection
    @Autowired
    public NotificationService(INotification notificationChannel) {
        this.notificationChannel = notificationChannel;
    }
    
    public void sendNotification(String message, String recipient) {
        notificationChannel.send(message, recipient);
    }
}

// Configuration
@Configuration
public class AppConfig {
    
    @Bean
    public INotification notificationChannel() {
        // Switch implementation here
        return new WhatsAppAdapter(new WhatsAppAPI());
        // OR return new EmailAdapter(new EmailAPI());
    }
}
```

**Benefits:**
- Change implementation without touching service code
- Easy testing with mock adapters
- Configuration-driven vendor selection

---

### Q9: How do you test code that uses Adapter Pattern?

**Answer:**

**Testing is easier with Adapter Pattern:**

```java
// Mock the adapter for testing
@Test
public void testOrderProcessing() {
    // Create a mock adapter
    IPaymentGateway mockGateway = mock(IPaymentGateway.class);
    when(mockGateway.processPayment(100.0)).thenReturn(true);
    
    // Inject mock into service
    OrderService service = new OrderService(mockGateway);
    
    // Test your business logic
    boolean result = service.processOrder(100.0);
    
    // Verify
    assertTrue(result);
    verify(mockGateway).processPayment(100.0);
}

// No need to test actual third-party API!
```

---

### Q10: What's the relationship between Adapter Pattern and SOLID principles?

**Answer:**

**1. Single Responsibility Principle (SRP):**
- Adapter has one job: translate interfaces
- Business logic stays in client
- Conversion logic stays in adapter

**2. Open/Closed Principle (OCP):**
- Open for extension: Add new adapters
- Closed for modification: Don't change existing code

**3. Liskov Substitution Principle (LSP):**
- Adapter can substitute target interface
- Client works with any adapter implementation

**4. Interface Segregation Principle (ISP):**
- Client depends only on target interface
- Not forced to depend on adaptee

**5. Dependency Inversion Principle (DIP):**
- Client depends on abstraction (IReports)
- Not on concretion (XMLDataProvider)

```java
// DIP in action
public class Client {
    // Depends on abstraction
    private IReports report;
    
    // Not on concretion
    // private XMLDataProvider provider; ❌
}
```

---

## 🎓 Summary

The **Adapter Design Pattern** is essential for:
- Integrating third-party libraries
- Working with legacy code
- Decoupling your code from external dependencies
- Making incompatible interfaces work together

**Remember:**
- Use **Object Adapter** (composition) over Class Adapter (inheritance)
- Adapter **IS-A** Target and **HAS-A** Adaptee
- Client knows only about Target interface
- Adapter translates calls from Target to Adaptee

**Real-world analogy:** Just like a power adapter lets you use your Indian charger in a US socket, the Adapter pattern lets incompatible code interfaces work together! 🔌

---

