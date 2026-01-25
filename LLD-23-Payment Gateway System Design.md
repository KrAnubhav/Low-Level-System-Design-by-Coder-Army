# LLD-23: Payment Gateway System Design

## 📋 Table of Contents

1. [Introduction](#introduction)
2. [What is a Payment Gateway?](#what-is-a-payment-gateway)
3. [Functional Requirements](#functional-requirements)
4. [System Flow](#system-flow)
5. [Understanding Gateway Concept](#understanding-gateway-concept)
6. [UML Design](#uml-design)
7. [Design Patterns Used](#design-patterns-used)
8. [Implementation](#implementation)
9. [Additional Features (Homework)](#additional-features-homework)
10. [Key Takeaways](#key-takeaways)

---

## 🎯 Introduction

Welcome to the Payment Gateway System Design lecture! Today we'll learn how to design and integrate a payment gateway system that can be used by any application.

**Important Distinction**:
- ✅ **What we're building**: A Payment Gateway System (plug-and-play for any application)
- ❌ **What we're NOT building**: Internal banking system architecture (like Paytm's or GPay's internal system)

**Real-world Usage**:
- Amazon, Zomato, Ola, Uber, Swiggy - all integrate payment gateways
- Our system will be a **plug-and-play** solution that any application can integrate

---

## 💳 What is a Payment Gateway?

A **Payment Gateway** is a middleware that:
- Sits between your application and banking systems
- Handles payment processing requests
- Communicates with third-party payment providers (Paytm, Razorpay, GPay, etc.)
- Returns payment success/failure status

**Example Flow**:
```
User Orders Food on Zomato
    ↓
Adds to Cart
    ↓
Proceeds to Payment
    ↓
Selects Payment Method (Paytm/Razorpay/GPay)
    ↓
Payment Gateway Processes Request
    ↓
Banking System Processes Payment
    ↓
Success/Failure Response
```

---

## 📝 Functional Requirements

### 1. Support Multiple Providers
- Should support multiple payment providers (Paytm, Razorpay, etc.)
- Should be able to easily add new gateways in the future
- **Design Principle**: Open-Closed Principle

### 2. Standard Payment Flow
A standardized payment flow with required validations:

**Three-Step Process** (Always in this order):
1. **Validate** - Check if payment request is valid
   - Sufficient funds in account
   - Valid currency
   - Valid request parameters

2. **Initiate** - Start the actual payment process
   - Call to payment provider (Paytm/Razorpay)
   - Handle payment processing

3. **Confirm** - Confirm payment status
   - Verify payment success/failure
   - Return final status

**Important**: The order MUST be maintained: Validate → Initiate → Confirm

### 3. Error Handling and Retry Mechanism

**Retry Mechanism**:
- If payment fails, retry automatically
- Configurable retry count (e.g., 3 retries for Paytm, 5 for Razorpay)
- Stop retrying once payment succeeds

**Possible Failure Reasons**:
- Server issues
- Network problems
- Insufficient funds
- Invalid request

**Error Handling**:
- Log all errors
- Display appropriate error messages
- Handle exceptions gracefully

---

## 🔄 System Flow

### Happy Flow Diagram

```
┌─────────────────┐
│     Client      │ (Amazon, Zomato, etc.)
│  (Application)  │
└────────┬────────┘
         │
         │ Creates PaymentRequest
         ↓
┌─────────────────────────────────────────┐
│       Payment Gateway System            │
│  ┌──────────────────────────────────┐  │
│  │   Payment Controller (Entry)     │  │
│  └──────────┬───────────────────────┘  │
│             ↓                           │
│  ┌──────────────────────────────────┐  │
│  │      Payment Service             │  │
│  └──────────┬───────────────────────┘  │
│             ↓                           │
│  ┌──────────────────────────────────┐  │
│  │   Payment Gateway Proxy          │  │
│  │   (Retry Mechanism)              │  │
│  └──────────┬───────────────────────┘  │
│             ↓                           │
│  ┌──────────────────────────────────┐  │
│  │   Payment Gateway                │  │
│  │   (Template Method)              │  │
│  │   - Validate                     │  │
│  │   - Initiate                     │  │
│  │   - Confirm                      │  │
│  └──────────┬───────────────────────┘  │
│             ↓                           │
│  ┌──────────────────────────────────┐  │
│  │  Paytm Gateway / Razorpay Gateway│  │
│  └──────────┬───────────────────────┘  │
└─────────────┼───────────────────────────┘
              │
              │ HTTP Call (Internet)
              ↓
┌─────────────────────────────────────────┐
│      Banking Systems (External)         │
│  ┌────────────────┐  ┌──────────────┐  │
│  │ Paytm Banking  │  │   Razorpay   │  │
│  │    System      │  │   Banking    │  │
│  └────────────────┘  └──────────────┘  │
└─────────────────────────────────────────┘
```

---

## � End-to-End Flow Explanation

### Complete Payment Processing Flow

Let's understand how a payment request flows through our entire system, step by step.

#### Step-by-Step Flow

```
┌──────────────────────────────────────────────────────────────────┐
│                    STEP 1: Client Creates Request                │
└──────────────────────────────────────────────────────────────────┘

Client Application (Amazon/Zomato/Swiggy)
    │
    │ Creates PaymentRequest object
    │ - sender: "Alice"
    │ - receiver: "Bob"  
    │ - amount: 1000.0
    │ - currency: "INR"
    │
    ↓

┌──────────────────────────────────────────────────────────────────┐
│              STEP 2: Call Payment Controller                     │
└──────────────────────────────────────────────────────────────────┘

PaymentController.getInstance()
    │
    │ handlePayment(paymentRequest, GatewayType.PAYTM)
    │
    ↓

┌──────────────────────────────────────────────────────────────────┐
│              STEP 3: Factory Creates Gateway                     │
└──────────────────────────────────────────────────────────────────┘

GatewayFactory.getInstance()
    │
    │ createGateway(GatewayType.PAYTM)
    │
    ├─→ Creates PaytmGateway instance
    │   └─→ Initializes PaytmBankingSystem
    │
    ├─→ Wraps in PaymentGatewayProxy
    │   └─→ Sets maxRetries = 3 (for Paytm)
    │
    └─→ Returns PaymentGatewayProxy(PaytmGateway)

┌──────────────────────────────────────────────────────────────────┐
│              STEP 4: Set Gateway in Service                      │
└──────────────────────────────────────────────────────────────────┘

PaymentService
    │
    │ setGateway(paymentGatewayProxy)
    │
    └─→ Stores reference to gateway

┌──────────────────────────────────────────────────────────────────┐
│              STEP 5: Process Payment (Proxy Layer)               │
└──────────────────────────────────────────────────────────────────┘

PaymentGatewayProxy.processPayment(paymentRequest)
    │
    │ Retry Loop (attempts = 0; attempts < 3)
    │
    ├─→ Attempt 1:
    │   │
    │   ├─→ Calls real.processPayment(pr)
    │   │   └─→ Delegates to PaytmGateway
    │   │
    │   └─→ If fails: attempts++, retry
    │
    ├─→ Attempt 2:
    │   │
    │   ├─→ Calls real.processPayment(pr)
    │   │
    │   └─→ If succeeds: return true
    │
    └─→ If all attempts fail: return false

┌──────────────────────────────────────────────────────────────────┐
│         STEP 6: Template Method (PaytmGateway)                   │
└──────────────────────────────────────────────────────────────────┘

PaytmGateway.processPayment(paymentRequest)
    │
    │ Template Method enforces order:
    │
    ├─→ Step 6a: validate(paymentRequest)
    │   │
    │   ├─→ Check amount > 0
    │   ├─→ Check currency == "INR"
    │   ├─→ Check sender/receiver valid
    │   │
    │   └─→ If validation fails: throw Exception
    │
    ├─→ Step 6b: initiate(paymentRequest)
    │   │
    │   ├─→ Prepare payment request
    │   ├─→ Log initiation
    │   │
    │   └─→ If initiation fails: throw Exception
    │
    ├─→ Step 6c: confirm(paymentRequest)
    │   │
    │   ├─→ Calls ibs.processPayment(amount)
    │   │   └─→ PaytmBankingSystem.processPayment(1000.0)
    │   │
    │   └─→ If confirmation fails: throw Exception
    │
    └─→ Return true (all steps succeeded)

┌──────────────────────────────────────────────────────────────────┐
│         STEP 7: Banking System Processing                       │
└──────────────────────────────────────────────────────────────────┘

PaytmBankingSystem.processPayment(1000.0)
    │
    ├─→ Makes HTTP call to Paytm servers
    │   └─→ Sends payment request over internet
    │
    ├─→ Paytm Banking System processes payment
    │   ├─→ Checks account balance
    │   ├─→ Verifies transaction
    │   └─→ Deducts amount
    │
    └─→ Returns success/failure (boolean)

┌──────────────────────────────────────────────────────────────────┐
│         STEP 8: Response Propagation                            │
└──────────────────────────────────────────────────────────────────┘

Response flows back through layers:

PaytmBankingSystem (returns true/false)
    ↓
PaytmGateway.confirm() (returns true/false)
    ↓
PaytmGateway.processPayment() (returns true/false)
    ↓
PaymentGatewayProxy.processPayment() (handles retry)
    ↓
PaymentService.processPayment() (returns true/false)
    ↓
PaymentController.handlePayment() (returns true/false)
    ↓
Client Application (receives final result)

┌──────────────────────────────────────────────────────────────────┐
│         STEP 9: Client Receives Response                        │
└──────────────────────────────────────────────────────────────────┘

Client Application
    │
    ├─→ If result == true:
    │   └─→ Payment successful!
    │       └─→ Show success message to user
    │
    └─→ If result == false:
        └─→ Payment failed!
            └─→ Show error message to user
```

### Detailed Flow with Example

**Scenario**: User pays ₹1000 on Zomato using Paytm

```java
// 1. Zomato creates payment request
PaymentRequest request = new PaymentRequest("User123", "Zomato", 1000.0, "INR");

// 2. Zomato calls Payment Controller
PaymentController controller = PaymentController.getInstance();
boolean result = controller.handlePayment(request, GatewayType.PAYTM);

// Inside handlePayment():
// 3. Factory creates gateway
GatewayFactory factory = GatewayFactory.getInstance();
PaymentGateway gateway = factory.createGateway(GatewayType.PAYTM);
    // Creates: PaytmGateway
    // Wraps in: PaymentGatewayProxy (3 retries)

// 4. Set gateway in service
paymentService.setGateway(gateway);

// 5. Process payment
boolean result = paymentService.processPayment(request);
    // Calls: PaymentGatewayProxy.processPayment()
    
    // Inside Proxy:
    // Attempt 1:
    //   PaytmGateway.processPayment()
    //     → validate() ✓
    //     → initiate() ✓
    //     → confirm() → PaytmBankingSystem.processPayment()
    //         → HTTP call to Paytm
    //         → Returns: false (payment failed)
    //   Result: Failed, retry...
    
    // Attempt 2:
    //   PaytmGateway.processPayment()
    //     → validate() ✓
    //     → initiate() ✓
    //     → confirm() → PaytmBankingSystem.processPayment()
    //         → HTTP call to Paytm
    //         → Returns: true (payment success!)
    //   Result: Success!
    
    // Returns: true

// 6. Zomato receives result
if (result) {
    System.out.println("Payment successful! Order confirmed.");
} else {
    System.out.println("Payment failed! Please try again.");
}
```

### Key Points in the Flow

1. **Single Entry Point**: 
   - Client only interacts with `PaymentController`
   - No need to know about internal classes

2. **Factory Pattern**:
   - Factory decides which gateway to create
   - Automatically wraps in proxy for retry

3. **Proxy Pattern**:
   - Transparently adds retry mechanism
   - Client doesn't know about retries

4. **Template Method**:
   - Enforces Validate → Initiate → Confirm order
   - Cannot skip or reorder steps

5. **Separation of Concerns**:
   - Controller: Entry point
   - Service: Business logic
   - Gateway: Payment processing
   - Proxy: Cross-cutting concerns (retry)
   - Banking System: External communication

### Flow Diagram with Retry Scenario

```
Client Request (₹1000)
    ↓
PaymentController
    ↓
GatewayFactory
    ├─→ Creates PaytmGateway
    └─→ Wraps in Proxy (3 retries)
    ↓
PaymentService
    ↓
PaymentGatewayProxy
    │
    ├─→ Attempt 1:
    │   ├─→ PaytmGateway.processPayment()
    │   │   ├─→ validate() ✓
    │   │   ├─→ initiate() ✓
    │   │   └─→ confirm() → Paytm API → ✗ FAILED
    │   └─→ Retry...
    │
    ├─→ Attempt 2:
    │   ├─→ PaytmGateway.processPayment()
    │   │   ├─→ validate() ✓
    │   │   ├─→ initiate() ✓
    │   │   └─→ confirm() → Paytm API → ✓ SUCCESS
    │   └─→ Return true
    │
    ↓
Client receives SUCCESS
```

### What Happens at Each Layer?

| Layer | Responsibility | What It Does |
|-------|---------------|--------------|
| **Client** | Initiate payment | Creates PaymentRequest, calls controller |
| **Controller** | Entry point | Routes request to service |
| **Factory** | Object creation | Creates appropriate gateway + proxy |
| **Service** | Business logic | Manages gateway, processes payment |
| **Proxy** | Retry mechanism | Retries on failure, delegates to real gateway |
| **Gateway** | Payment flow | Validates → Initiates → Confirms |
| **Banking System** | External API | Makes HTTP call to Paytm/Razorpay |

### Error Handling in Flow

```
PaymentGatewayProxy (Retry Layer)
    │
    ├─→ Try: real.processPayment()
    │   │
    │   └─→ PaytmGateway.processPayment()
    │       │
    │       ├─→ validate() → throws Exception
    │       │   └─→ Caught by Proxy → Retry
    │       │
    │       ├─→ initiate() → throws Exception
    │       │   └─→ Caught by Proxy → Retry
    │       │
    │       └─→ confirm() → throws Exception
    │           └─→ Caught by Proxy → Retry
    │
    └─→ After maxRetries: return false
```

---

## �🚪 Understanding Gateway Concept

### What is a Gateway?

A **Gateway** is the entry/exit point of your application for external communication.

```
┌─────────────────────────────────────────┐
│         Your Application                │
│                                         │
│  ┌──────┐    ┌──────┐    ┌──────┐     │
│  │Class1│────│Class2│────│Class3│     │
│  └──────┘    └──────┘    └──────┘     │
│                    │                   │
│                    ↓                   │
│            ┌──────────────┐            │
│            │   GATEWAY    │◄───────────┼─── Application Boundary
│            └──────┬───────┘            │
└────────────────────┼────────────────────┘
                     │
                     │ HTTP Calls
                     ↓
            ┌────────────────┐
            │    Internet    │
            │  External APIs │
            └────────────────┘
```

**Key Points**:
- Gateway is the **single point** for external communication
- Internal classes don't need to know how to make external calls
- Gateway handles all HTTP/network communication
- Similar to **Proxy Pattern** (Remote Proxy)

**Why Use Gateway?**
- **Separation of Concerns**: Internal classes focus on business logic
- **Single Responsibility**: Gateway handles external communication
- **Easy to Maintain**: Changes to external APIs only affect gateway
- **Testability**: Can mock gateway for testing

---

## 🎨 UML Design

### Class Diagram

```
┌─────────────────────────────────────────────────────────────┐
│                    PaymentRequest                           │
├─────────────────────────────────────────────────────────────┤
│ - sender: String                                            │
│ - receiver: String                                          │
│ - amount: double                                            │
│ - currency: String                                          │
├─────────────────────────────────────────────────────────────┤
│ + getters/setters                                           │
└─────────────────────────────────────────────────────────────┘

┌─────────────────────────────────────────────────────────────┐
│              <<interface>> IBankingSystem                   │
├─────────────────────────────────────────────────────────────┤
│ + processPayment(amount: double): boolean                   │
└──────────────────────┬──────────────────────────────────────┘
                       │
         ┌─────────────┴─────────────┐
         │                           │
         ↓                           ↓
┌──────────────────────┐    ┌──────────────────────┐
│ PaytmBankingSystem   │    │ RazorpayBankingSystem│
├──────────────────────┤    ├──────────────────────┤
│ + processPayment()   │    │ + processPayment()   │
└──────────────────────┘    └──────────────────────┘

┌─────────────────────────────────────────────────────────────┐
│                    PaymentGateway                           │
├─────────────────────────────────────────────────────────────┤
│ # ibs: IBankingSystem                                       │
│ # pr: PaymentRequest                                        │
├─────────────────────────────────────────────────────────────┤
│ + processPayment(pr: PaymentRequest): boolean               │
│ # validate(pr: PaymentRequest): boolean                     │
│ # initiate(pr: PaymentRequest): boolean                     │
│ # confirm(pr: PaymentRequest): boolean                      │
└──────────────────────┬──────────────────────────────────────┘
                       │
         ┌─────────────┼─────────────┬──────────────────┐
         │             │             │                  │
         ↓             ↓             ↓                  ↓
┌──────────────┐ ┌──────────────┐ ┌──────────────────────────┐
│PaytmGateway  │ │RazorpayGateway│ │PaymentGatewayProxy       │
├──────────────┤ ├──────────────┤ ├──────────────────────────┤
│+ validate()  │ │+ validate()  │ │- real: PaymentGateway    │
│+ initiate()  │ │+ initiate()  │ ├──────────────────────────┤
│+ confirm()   │ │+ confirm()   │ │+ processPayment()        │
└──────────────┘ └──────────────┘ │+ validate()              │
                                  │+ initiate()              │
                                  │+ confirm()               │
                                  └──────────────────────────┘

┌─────────────────────────────────────────────────────────────┐
│                  <<enum>> GatewayType                       │
├─────────────────────────────────────────────────────────────┤
│ PAYTM                                                       │
│ RAZORPAY                                                    │
└─────────────────────────────────────────────────────────────┘

┌─────────────────────────────────────────────────────────────┐
│            GatewayFactory (Singleton)                       │
├─────────────────────────────────────────────────────────────┤
│ - instance: GatewayFactory                                  │
├─────────────────────────────────────────────────────────────┤
│ + getInstance(): GatewayFactory                             │
│ + createGateway(type: GatewayType): PaymentGateway          │
└─────────────────────────────────────────────────────────────┘

┌─────────────────────────────────────────────────────────────┐
│              PaymentService                                 │
├─────────────────────────────────────────────────────────────┤
│ - pg: PaymentGateway                                        │
├─────────────────────────────────────────────────────────────┤
│ + setGateway(pg: PaymentGateway): void                      │
│ + processPayment(): boolean                                 │
└─────────────────────────────────────────────────────────────┘

┌─────────────────────────────────────────────────────────────┐
│          PaymentController (Singleton)                      │
├─────────────────────────────────────────────────────────────┤
│ - ps: PaymentService                                        │
├─────────────────────────────────────────────────────────────┤
│ + handlePayment(pr: PaymentRequest): boolean                │
└─────────────────────────────────────────────────────────────┘
```

---

## 🎭 Design Patterns Used

### 1. Template Method Pattern

**Used in**: `PaymentGateway` class

**Purpose**: Standardize payment flow (Validate → Initiate → Confirm)

```java
public abstract class PaymentGateway {
    // Template Method - defines the algorithm structure
    public boolean processPayment(PaymentRequest pr) {
        if (!validate(pr)) {
            throw new Exception("Validation failed");
        }
        
        if (!initiate(pr)) {
            throw new Exception("Initiation failed");
        }
        
        if (!confirm(pr)) {
            throw new Exception("Confirmation failed");
        }
        
        return true;
    }
    
    // Abstract methods - to be implemented by subclasses
    protected abstract boolean validate(PaymentRequest pr);
    protected abstract boolean initiate(PaymentRequest pr);
    protected abstract boolean confirm(PaymentRequest pr);
}
```

**Benefits**:
- Enforces standard payment flow
- Subclasses can customize individual steps
- Flow order is guaranteed

### 2. Proxy Pattern (Protection Proxy)

**Used in**: `PaymentGatewayProxy` class

**Purpose**: Add retry mechanism without modifying gateway classes

```java
public class PaymentGatewayProxy extends PaymentGateway {
    private PaymentGateway real;
    private int maxRetries = 3;
    
    @Override
    public boolean processPayment(PaymentRequest pr) {
        int attempts = 0;
        
        while (attempts < maxRetries) {
            try {
                return real.processPayment(pr);  // Delegate to real object
            } catch (Exception e) {
                attempts++;
                System.out.println("Retry attempt: " + attempts);
            }
        }
        
        return false;
    }
    
    // Delegate other methods to real object
    @Override
    protected boolean validate(PaymentRequest pr) {
        return real.validate(pr);
    }
}
```

**Benefits**:
- Adds retry logic without changing gateway classes
- Single Responsibility: Proxy handles retries, Gateway handles payment
- Easy to modify retry logic

### 3. Factory Pattern

**Used in**: `GatewayFactory` class

**Purpose**: Create appropriate gateway based on type

```java
public class GatewayFactory {
    private static GatewayFactory instance;
    
    public static GatewayFactory getInstance() {
        if (instance == null) {
            instance = new GatewayFactory();
        }
        return instance;
    }
    
    public PaymentGateway createGateway(GatewayType type) {
        PaymentGateway gateway;
        
        switch (type) {
            case PAYTM:
                gateway = new PaytmGateway();
                break;
            case RAZORPAY:
                gateway = new RazorpayGateway();
                break;
            default:
                throw new IllegalArgumentException("Unknown gateway type");
        }
        
        // Wrap in proxy for retry mechanism
        return new PaymentGatewayProxy(gateway);
    }
}
```

### 4. Singleton Pattern

**Used in**: `GatewayFactory`, `PaymentController`

**Purpose**: Ensure single instance throughout application

---

## 💻 Implementation

### 1. PaymentRequest (Model Class)

```java
public class PaymentRequest {
    private String sender;
    private String receiver;
    private double amount;
    private String currency;
    
    public PaymentRequest(String sender, String receiver, double amount, String currency) {
        this.sender = sender;
        this.receiver = receiver;
        this.amount = amount;
        this.currency = currency;
    }
    
    // Getters and Setters
    public String getSender() { return sender; }
    public String getReceiver() { return receiver; }
    public double getAmount() { return amount; }
    public String getCurrency() { return currency; }
}
```

### 2. Banking System Interface

```java
public interface IBankingSystem {
    boolean processPayment(double amount);
}
```

### 3. Banking System Implementations

```java
// Paytm Banking System (20% success rate for simulation)
public class PaytmBankingSystem implements IBankingSystem {
    @Override
    public boolean processPayment(double amount) {
        System.out.println("Processing payment through Paytm: " + amount);
        
        // Simulate 20% success rate
        return Math.random() < 0.2;
    }
}

// Razorpay Banking System (90% success rate for simulation)
public class RazorpayBankingSystem implements IBankingSystem {
    @Override
    public boolean processPayment(double amount) {
        System.out.println("Processing payment through Razorpay: " + amount);
        
        // Simulate 90% success rate
        return Math.random() < 0.9;
    }
}
```

### 4. Payment Gateway (Template Method)

```java
public abstract class PaymentGateway {
    protected IBankingSystem ibs;
    protected PaymentRequest pr;
    
    public PaymentGateway() {
        this.ibs = null;
    }
    
    // Template Method
    public boolean processPayment(PaymentRequest pr) {
        this.pr = pr;
        
        if (!validate(pr)) {
            throw new RuntimeException("Validation failed");
        }
        
        if (!initiate(pr)) {
            throw new RuntimeException("Initiation failed");
        }
        
        if (!confirm(pr)) {
            throw new RuntimeException("Confirmation failed");
        }
        
        return true;
    }
    
    // Abstract methods to be implemented by subclasses
    protected abstract boolean validate(PaymentRequest pr);
    protected abstract boolean initiate(PaymentRequest pr);
    protected abstract boolean confirm(PaymentRequest pr);
}
```

### 5. Concrete Gateway Implementations

```java
// Paytm Gateway
public class PaytmGateway extends PaymentGateway {
    public PaytmGateway() {
        super();
        this.ibs = new PaytmBankingSystem();
    }
    
    @Override
    protected boolean validate(PaymentRequest pr) {
        System.out.println("Paytm: Validating payment request");
        // Validation logic: check currency, amount, etc.
        return pr.getAmount() > 0 && pr.getCurrency().equals("INR");
    }
    
    @Override
    protected boolean initiate(PaymentRequest pr) {
        System.out.println("Paytm: Initiating payment");
        return true;
    }
    
    @Override
    protected boolean confirm(PaymentRequest pr) {
        System.out.println("Paytm: Confirming payment");
        return ibs.processPayment(pr.getAmount());
    }
}

// Razorpay Gateway
public class RazorpayGateway extends PaymentGateway {
    public RazorpayGateway() {
        super();
        this.ibs = new RazorpayBankingSystem();
    }
    
    @Override
    protected boolean validate(PaymentRequest pr) {
        System.out.println("Razorpay: Validating payment request");
        return pr.getAmount() > 0;
    }
    
    @Override
    protected boolean initiate(PaymentRequest pr) {
        System.out.println("Razorpay: Initiating payment");
        return true;
    }
    
    @Override
    protected boolean confirm(PaymentRequest pr) {
        System.out.println("Razorpay: Confirming payment");
        return ibs.processPayment(pr.getAmount());
    }
}
```

### 6. Payment Gateway Proxy (Retry Mechanism)

```java
public class PaymentGatewayProxy extends PaymentGateway {
    private PaymentGateway real;
    private int maxRetries;
    
    public PaymentGatewayProxy(PaymentGateway real, int maxRetries) {
        this.real = real;
        this.maxRetries = maxRetries;
    }
    
    @Override
    public boolean processPayment(PaymentRequest pr) {
        int attempts = 0;
        
        while (attempts < maxRetries) {
            try {
                System.out.println("\n=== Attempt " + (attempts + 1) + " ===");
                boolean result = real.processPayment(pr);
                
                if (result) {
                    System.out.println("✓ Payment successful!");
                    return true;
                }
                
                System.out.println("✗ Payment failed, retrying...");
                attempts++;
                
            } catch (Exception e) {
                System.out.println("✗ Error: " + e.getMessage());
                attempts++;
            }
        }
        
        System.out.println("✗ Payment failed after " + maxRetries + " attempts");
        return false;
    }
    
    @Override
    protected boolean validate(PaymentRequest pr) {
        return real.validate(pr);
    }
    
    @Override
    protected boolean initiate(PaymentRequest pr) {
        return real.initiate(pr);
    }
    
    @Override
    protected boolean confirm(PaymentRequest pr) {
        return real.confirm(pr);
    }
}
```

### 7. Gateway Type Enum

```java
public enum GatewayType {
    PAYTM,
    RAZORPAY
}
```

### 8. Gateway Factory (Singleton)

```java
public class GatewayFactory {
    private static GatewayFactory instance;
    
    private GatewayFactory() {}
    
    public static GatewayFactory getInstance() {
        if (instance == null) {
            instance = new GatewayFactory();
        }
        return instance;
    }
    
    public PaymentGateway createGateway(GatewayType type) {
        PaymentGateway gateway;
        int retries;
        
        switch (type) {
            case PAYTM:
                gateway = new PaytmGateway();
                retries = 3;  // Paytm: 3 retries (lower success rate)
                break;
                
            case RAZORPAY:
                gateway = new RazorpayGateway();
                retries = 5;  // Razorpay: 5 retries
                break;
                
            default:
                throw new IllegalArgumentException("Unknown gateway type: " + type);
        }
        
        // Wrap in proxy for retry mechanism
        return new PaymentGatewayProxy(gateway, retries);
    }
}
```

### 9. Payment Service

```java
public class PaymentService {
    private PaymentGateway pg;
    
    public void setGateway(PaymentGateway pg) {
        this.pg = pg;
    }
    
    public boolean processPayment(PaymentRequest pr) {
        if (pg == null) {
            throw new RuntimeException("Payment gateway not set");
        }
        
        return pg.processPayment(pr);
    }
}
```

### 10. Payment Controller (Singleton)

```java
public class PaymentController {
    private static PaymentController instance;
    private PaymentService ps;
    
    private PaymentController() {
        this.ps = new PaymentService();
    }
    
    public static PaymentController getInstance() {
        if (instance == null) {
            instance = new PaymentController();
        }
        return instance;
    }
    
    public boolean handlePayment(PaymentRequest pr, GatewayType type) {
        // Step 1: Get gateway from factory
        GatewayFactory factory = GatewayFactory.getInstance();
        PaymentGateway pg = factory.createGateway(type);
        
        // Step 2: Set gateway in service
        ps.setGateway(pg);
        
        // Step 3: Process payment
        return ps.processPayment(pr);
    }
}
```

### 11. Client Code (Main)

```java
public class Main {
    public static void main(String[] args) {
        // Create payment request
        PaymentRequest pr = new PaymentRequest(
            "Alice",      // sender
            "Bob",        // receiver
            1000.0,       // amount
            "INR"         // currency
        );
        
        // Get controller instance
        PaymentController controller = PaymentController.getInstance();
        
        // Test with Paytm (20% success rate, 3 retries)
        System.out.println("\n========== Testing with PAYTM ==========");
        boolean paytmResult = controller.handlePayment(pr, GatewayType.PAYTM);
        System.out.println("\nFinal Result: " + (paytmResult ? "SUCCESS" : "FAILED"));
        
        // Test with Razorpay (90% success rate, 5 retries)
        System.out.println("\n\n========== Testing with RAZORPAY ==========");
        boolean razorpayResult = controller.handlePayment(pr, GatewayType.RAZORPAY);
        System.out.println("\nFinal Result: " + (razorpayResult ? "SUCCESS" : "FAILED"));
    }
}
```

### Sample Output

```
========== Testing with PAYTM ==========

=== Attempt 1 ===
Paytm: Validating payment request
Paytm: Initiating payment
Paytm: Confirming payment
Processing payment through Paytm: 1000.0
✗ Payment failed, retrying...

=== Attempt 2 ===
Paytm: Validating payment request
Paytm: Initiating payment
Paytm: Confirming payment
Processing payment through Paytm: 1000.0
✓ Payment successful!

Final Result: SUCCESS


========== Testing with RAZORPAY ==========

=== Attempt 1 ===
Razorpay: Validating payment request
Razorpay: Initiating payment
Razorpay: Confirming payment
Processing payment through Razorpay: 1000.0
✓ Payment successful!

Final Result: SUCCESS
```

---

## 🎯 Additional Features (Homework)

Extend the payment gateway system with these features:

### 1. Transaction History
- Store all payment transactions
- Provide query methods to retrieve transaction history
- Include transaction ID, timestamp, status, amount

### 2. Payment Status Tracking
- Implement status enum: PENDING, PROCESSING, SUCCESS, FAILED
- Update status at each step
- Provide real-time status updates

### 3. Multiple Currency Support
- Add currency conversion
- Support USD, EUR, GBP, INR, etc.
- Integrate with currency exchange API

### 4. Webhook Support
- Implement callback mechanism
- Notify client application on payment completion
- Support async payment notifications

### 5. Payment Limits
- Daily transaction limits
- Per-transaction limits
- User-based limits

### 6. Fraud Detection
- Add basic fraud detection rules
- Check for suspicious patterns
- Block/flag suspicious transactions

### 7. Refund Mechanism
- Implement payment refund
- Partial and full refund support
- Refund status tracking

---

## 🎓 Key Takeaways

### Design Patterns Applied

1. **Template Method Pattern**
   - Standardizes payment flow
   - Ensures Validate → Initiate → Confirm order
   - Allows customization of individual steps

2. **Proxy Pattern**
   - Adds retry mechanism
   - Separates retry logic from payment logic
   - Protection proxy for validation

3. **Factory Pattern**
   - Creates appropriate gateway
   - Encapsulates object creation
   - Easy to add new gateways

4. **Singleton Pattern**
   - Single instance of factory and controller
   - Consistent state throughout application

### SOLID Principles

1. **Single Responsibility**
   - Each class has one responsibility
   - Proxy handles retries
   - Gateway handles payment
   - Service handles business logic

2. **Open-Closed Principle**
   - Open for extension (new gateways)
   - Closed for modification (existing code)

3. **Liskov Substitution**
   - Any gateway can replace PaymentGateway
   - Polymorphic behavior

4. **Interface Segregation**
   - Small, focused interfaces
   - IBankingSystem has single method

5. **Dependency Inversion**
   - Depend on abstractions (interfaces)
   - Not on concrete implementations

### Architecture Highlights

1. **Layered Architecture**
   ```
   Controller Layer (Entry Point)
        ↓
   Service Layer (Business Logic)
        ↓
   Gateway Layer (External Communication)
        ↓
   Banking System (Third-party)
   ```

2. **Plug-and-Play Design**
   - Single entry point (PaymentController)
   - Easy integration for any application
   - Minimal client code changes

3. **Separation of Concerns**
   - Controller: Request handling
   - Service: Business logic
   - Gateway: External communication
   - Proxy: Cross-cutting concerns (retry)

### Best Practices

1. ✅ Use enums for fixed types
2. ✅ Implement retry mechanism in proxy
3. ✅ Standardize flow with template method
4. ✅ Use factory for object creation
5. ✅ Keep controllers thin, services fat
6. ✅ Handle errors gracefully
7. ✅ Log all important events
8. ✅ Make system extensible

---

## 📊 Comparison: With vs Without Design Patterns

### Without Design Patterns ❌

```java
public class PaymentService {
    public boolean processPayment(String gateway, PaymentRequest pr) {
        if (gateway.equals("PAYTM")) {
            // Paytm specific code
            // Validate
            // Initiate
            // Confirm
            // Retry logic
        } else if (gateway.equals("RAZORPAY")) {
            // Razorpay specific code
            // Validate
            // Initiate
            // Confirm
            // Retry logic
        }
        // Adding new gateway requires modifying this class!
    }
}
```

**Problems**:
- Violates Open-Closed Principle
- Code duplication
- Hard to maintain
- Difficult to test
- No separation of concerns

### With Design Patterns ✅

```java
// Clean, extensible, maintainable
PaymentController controller = PaymentController.getInstance();
boolean result = controller.handlePayment(pr, GatewayType.PAYTM);
```

**Benefits**:
- Follows SOLID principles
- Easy to extend
- Easy to test
- Clear separation of concerns
- Reusable components

---

## 🎯 Interview Tips

When explaining this design in an interview:

1. **Start with Requirements**
   - Clearly state functional requirements
   - Mention non-functional requirements (scalability, maintainability)

2. **Explain the Flow**
   - Draw the flow diagram
   - Explain client → controller → service → gateway → banking system

3. **Justify Design Patterns**
   - Explain WHY you chose each pattern
   - Template Method: For standardized flow
   - Proxy: For retry mechanism
   - Factory: For object creation
   - Singleton: For single instance

4. **Discuss Trade-offs**
   - Complexity vs Flexibility
   - Performance vs Maintainability

5. **Mention Extensibility**
   - How to add new gateways
   - How to modify retry logic
   - How to add new features

6. **Talk About Real-world Usage**
   - How companies like Amazon, Zomato use similar systems
   - Integration with actual payment providers

---

**Happy Learning! 🚀**

**Remember**: This is a **plug-and-play** payment gateway system that any application can integrate. The key is to make it extensible, maintainable, and follow SOLID principles!
