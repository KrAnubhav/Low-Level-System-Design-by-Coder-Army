### UML Diagrams - Complete Notes

## What are UML Diagrams?

UML (Unified Modeling Language) Diagrams are visual representations of system architecture and design. Instead of writing long paragraphs to explain an application’s structure, we use diagrams to communicate ideas more effectively and intuitively.

**Why UML Diagrams?**

- **Problem with Text**: Writing lengthy paragraphs about components, objects, entities, and their interactions is boring, non-intuitive, and difficult to understand
- **Solution**: Diagrammatic representation that clearly shows:
    - Which components/entities exist in the application
    - How they are connected to each other
    - How they interact and send messages to each other

**Interview Importance**: 99% of LLD interviews will require you to create UML diagrams before writing code.

---

## Types of UML Diagrams

UML Diagrams are divided into **two main categories**:

### 1. Structural Diagrams (Static Diagrams)

- Show the **structure** of the application
- Define which components exist
- Show how components are connected
- Represent the static interface of the application

### 2. Behavioral Diagrams (Dynamic Diagrams)

- Show how components **interact** with each other
- Demonstrate message passing between objects
- Represent method calls between components

**Total Count**: 7 Structural + 7 Behavioral = **14 UML Diagrams**

**For LLD/Interviews**: Only **2 diagrams** are essential:

1. **Class Diagram** (Structural)
2. **Sequence Diagram** (Behavioral)

**Why only these two?**

- The remaining 12 diagrams are use-case specific
- Rarely asked in interviews
- These two cover 99% of LLD requirements

---

## Class Diagram

### What is a Class Diagram?

Class Diagrams represent:

- All classes in your application
- How classes are connected/associated with each other

**Two main components to learn**:

1. Class Structure Representation
2. Class Relationships/Associations

---

### Part 1: Representing a Class

### Basic Structure

A class is represented by a **rectangle divided into 3 sections**:

```
┌─────────────────────┐
│    Class Name       │  ← Section 1: Class Name
├─────────────────────┤
│   Characteristics   │  ← Section 2: Attributes/Variables
│   (Variables)       │
├─────────────────────┤
│    Behaviors        │  ← Section 3: Methods/Functions
│    (Methods)        │
└─────────────────────┘
```

### Example: Car Class

**Class Definition:**

```java
class Car {
    // Characteristics (Variables)
    String brand;
    String model;
    int engineCC;

    // Behaviors (Methods)
    void startEngine() { }
    void stopEngine() { }
    void accelerate() { }
    void brake() { }
}
```

**UML Representation:**

```
┌─────────────────────┐
│        Car          │
├─────────────────────┤
│ brand : String      │
│ model : String      │
│ engineCC : int      │
├─────────────────────┤
│ startEngine() : void│
│ stopEngine() : void │
│ accelerate() : void │
│ brake() : void      │
└─────────────────────┘
```

**Key Points:**

- **Section 1**: Class name in the top rectangle
- **Section 2**: Variables/Attributes written as `variableName : DataType`
- **Section 3**: Methods written as `methodName() : ReturnType`
- **Format difference**: In diagrams, we write name first, then type (opposite of code)

---

### Access Modifiers in Class Diagrams

### Three Types of Access Modifiers

| Access Modifier | Within Class | From Child Class | Outside Class |
| --- | --- | --- | --- |
| **Public (+)** | ✅ Yes | ✅ Yes | ✅ Yes |
| **Protected (#)** | ✅ Yes | ✅ Yes | ❌ No |
| **Private (-)** | ✅ Yes | ❌ No | ❌ No |

### Symbols for Access Modifiers

- **Public**: `+` (plus sign)
- **Protected**: `#` (hash sign)
- **Private**:  (minus sign)

### Example with Access Modifiers

**Java Code:**

```java
class Car {
    private String brand;
    protected String model;
    public int engineCC;

    public void startEngine() { }
    private void stopEngine() { }
    protected void accelerate() { }
}
```

**UML Representation:**

```
┌─────────────────────────┐
│         Car             │
├─────────────────────────┤
│ - brand : String        │
│ # model : String        │
│ + engineCC : int        │
├─────────────────────────┤
│ + startEngine() : void  │
│ - stopEngine() : void   │
│ # accelerate() : void   │
└─────────────────────────┘
```

**Explanation:**

- before `brand` = private (only accessible within class)
- `#` before `model` = protected (accessible in class + child classes)
- `+` before `engineCC` = public (accessible everywhere)

---

### Part 2: Class Relationships

There are **four main types of relationships** between classes:

### 1. Association

**Definition**: A basic “uses-a” or “has-a” relationship where one class uses or interacts with another class.

**Representation**: A **solid line** connecting two classes

**Example**: Driver and Car

```
┌──────────┐              ┌─────────┐
│  Driver  │─────────────→│   Car   │
└──────────┘              └─────────┘
```

**Java Code:**

```java
class Driver {
    void drive(Car car) {
        car.startEngine();
    }
}

class Car {
    void startEngine() { }
}
```

**Analogy**: Driver uses a Car, but doesn’t own it. The car can exist independently.

---

### 2. Aggregation (Weak “Has-A”)

**Definition**: A “has-a” relationship where the contained object can exist independently. The lifecycle of the part is **independent** of the whole.

**Representation**: A **solid line with an empty diamond** at the container end

**Example**: Department and Teacher

```
┌────────────┐ ◇─────────────→ ┌──────────┐
│ Department │                  │  Teacher │
└────────────┘                  └──────────┘
```

**Java Code:**

```java
class Department {
    List<Teacher> teachers;  // Department HAS teachers
}

class Teacher {
    String name;
    // Teacher can exist without Department
}
```

**Analogy**: A department has teachers, but if the department is dissolved, teachers continue to exist and can join other departments.

**Key Point**: The diamond is on the side of the container (Department).

---

### 3. Composition (Strong “Has-A”)

**Definition**: A strong “has-a” relationship where the part **cannot exist** without the whole. The lifecycle of the part is **dependent** on the whole.

**Representation**: A **solid line with a filled/solid diamond** at the container end

**Example**: House and Room

```
┌───────┐ ◆─────────────→ ┌───────┐
│ House │                 │  Room │
└───────┘                 └───────┘
```

**Java Code:**

```java
class House {
    List<Room> rooms;  // House OWNS rooms

    House() {
        rooms = new ArrayList<>();
        rooms.add(new Room());  // Rooms created with House
    }
}

class Room {
    String roomType;
    // Room cannot exist independently of House
}
```

**Analogy**: A house has rooms. If you destroy the house, the rooms are also destroyed. Rooms cannot exist without the house.

**Key Difference from Aggregation**:

- **Aggregation (empty diamond)**: Independent lifecycle (Teacher-Department)
- **Composition (filled diamond)**: Dependent lifecycle (Room-House)

---

### 4. Inheritance (IS-A Relationship)

**Definition**: A parent-child relationship where the child class inherits properties and methods from the parent class.

**Representation**: A **solid line with an empty triangle/arrow** pointing toward the parent

**Example**: Vehicle inheritance hierarchy

```
         ┌──────────┐
         │ Vehicle  │
         └──────────┘
              △
              │
      ┌───────┴───────┐
      │               │
┌─────┴────┐    ┌────┴─────┐
│   Car    │    │   Bike   │
└──────────┘    └──────────┘
```

**Java Code:**

```java
class Vehicle {
    String brand;
    void start() { }
}

class Car extends Vehicle {
    int numberOfDoors;
    void openTrunk() { }
}

class Bike extends Vehicle {
    boolean hasCarrier;
    void ringBell() { }
}
```

**Key Points**:

- Arrow points **toward the parent** (Vehicle)
- Child classes inherit all public/protected members of parent
- Represents “IS-A” relationship: Car **IS A** Vehicle, Bike **IS A** Vehicle

---

### Additional Relationships

### 5. Realization (Interface Implementation)

**Representation**: A **dashed line with an empty triangle** pointing toward the interface

**Example**: Payment interface implementation

```
         ┌──────────────┐
         │  <<interface>>│
         │   Payable    │
         └──────────────┘
              △
              ┆ (dashed)
              │
      ┌───────┴────────┐
      │                │
┌─────┴─────┐    ┌────┴──────┐
│CreditCard │    │  UPIPay   │
└───────────┘    └───────────┘
```

**Java Code:**

```java
interface Payable {
    void makePayment(double amount);
}

class CreditCard implements Payable {
    public void makePayment(double amount) {
        // Credit card payment logic
    }
}

class UPIPay implements Payable {
    public void makePayment(double amount) {
        // UPI payment logic
    }
}
```

---

### Cardinality (Multiplicity) in Relationships

Cardinality shows **how many instances** of one class relate to instances of another class.

**Common Notations:**

- `1` : Exactly one
- `0..1` : Zero or one
- or `0..*` : Zero or many
- `1..*` : One or many
- `n` : Exactly n

**Example: University and Student**

```
┌────────────┐ 1     0..* ┌──────────┐
│ University │───────────→│ Student  │
└────────────┘             └──────────┘
```

**Meaning**:

- One University can have zero or many Students (0..*)
- Each Student belongs to exactly one University (1)

**Another Example: Order and Product**

```
┌─────────┐ 1     1..* ┌──────────┐
│  Order  │───────────→│ Product  │
└─────────┘             └──────────┘
```

**Meaning**:

- One Order must have at least one Product (1..*)
- Products can be part of one Order

---

### Complete Class Diagram Example: E-Commerce System

```
                ┌─────────────────┐
                │   <<interface>> │
                │    Payable      │
                └─────────────────┘
                        △
                        ┆
                        │
         ┌──────────────┴──────────────┐
         │                             │
┌────────┴─────────┐         ┌─────────┴────────┐
│   CreditCard     │         │    UPIPay        │
└──────────────────┘         └──────────────────┘

┌──────────────────────┐ 1        0..* ┌─────────────────────┐
│      Customer        │◇──────────────→│       Order         │
├──────────────────────┤                ├─────────────────────┤
│ - customerId : int   │                │ - orderId : int     │
│ - name : String      │                │ - orderDate : Date  │
│ - email : String     │                │ - totalAmount:double│
├──────────────────────┤                ├─────────────────────┤
│ + placeOrder(): void │                │ + addProduct(): void│
│ + viewOrders(): List │                │ + checkout() : void │
└──────────────────────┘                └─────────────────────┘
                                                  │
                                                  │ 1..*
                                                  │
                                                  ▽
                                        ┌─────────────────────┐
                                        │      Product        │
                                        ├─────────────────────┤
                                        │ - productId : int   │
                                        │ - name : String     │
                                        │ - price : double    │
                                        ├─────────────────────┤
                                        │ + getDetails():String│
                                        └─────────────────────┘
```

**Relationships shown:**

- Customer **aggregates** Orders (empty diamond)
- Order **contains** Products (1..* cardinality)
- CreditCard and UPIPay **implement** Payable interface (dashed arrow)

---

## Sequence Diagram

### What is a Sequence Diagram?

Sequence Diagrams show how objects **interact** with each other over time. They visualize the flow of messages between objects in a specific use case or scenario.

**When to use:**

- To understand the flow of a particular feature/use case
- To visualize object interactions and method calls
- When explaining complex workflows
- Not always asked in interviews, but extremely helpful for understanding system behavior

---

### Components of Sequence Diagram

### 1. Actors/Objects

Represented as **boxes at the top** with vertical **lifelines** (dashed lines going down).

```
┌─────────┐    ┌─────────┐    ┌──────────┐
│  User   │    │  System │    │ Database │
└─────────┘    └─────────┘    └──────────┘
     │              │               │
     │(lifeline)    │               │
     │              │               │
```

**Types:**

- **Actor**: External entity (e.g., User, Admin) - represented as stick figure or box
- **Object**: Internal system components (e.g., Controller, Service, Database)

---

### 2. Messages (Method Calls)

Represented as **horizontal arrows** between lifelines.

**Types of Messages:**

**a) Synchronous Call (Solid arrow with filled head)**

```
     │──────────────────→│
     │  methodName()    │
```

- Caller waits for response
- Most common type

**b) Asynchronous Call (Solid arrow with line head)**

```
     │──────────────────>│
     │  methodName()    │
```

- Caller doesn’t wait for response
- Continues execution immediately

**c) Return Message (Dashed arrow)**

```
     │←- - - - - - - - -│
     │   return value   │
```

- Shows return from a method call

---

### 3. Activation Boxes

**Thin vertical rectangles** on lifelines showing when an object is active/processing.

```
     │
     │────────→│
     │         ║  ← Activation box (object is processing)
     │←────────║
     │         │
```

---

### Complete Sequence Diagram Example: Online Shopping Checkout

```
User        WebUI       OrderController    PaymentService    Database
 │            │                │                  │             │
 │            │                │                  │             │
 │─checkout()→│                │                  │             │
 │            │                │                  │             │
 │            │─validateCart()→│                  │             │
 │            │                ║                  │             │
 │            │                ║─getCartItems()──→│             │
 │            │                ║                  ║             │
 │            │                ║                  ║─SELECT *───→│
 │            │                ║                  ║             ║
 │            │                ║←─────items──────║←────data────║
 │            │                ║                  │             │
 │            │←─────valid─────║                  │             │
 │            │                │                  │             │
 │            │─processPayment()─────────────────→│             │
 │            │                │                  ║             │
 │            │                │                  ║─deductAmount→│
 │            │                │                  ║             ║
 │            │                │                  ║←──success───║
 │            │                │                  │             │
 │            │←──────paymentSuccess──────────────│             │
 │            │                │                  │             │
 │←─success───│                │                  │             │
 │            │                │                  │             │
```

**Flow Explanation:**

1. User initiates checkout on WebUI
2. WebUI calls validateCart() on OrderController
3. OrderController calls getCartItems() from PaymentService
4. PaymentService queries Database for cart items
5. Data flows back through return messages
6. WebUI processes payment through PaymentService
7. PaymentService updates Database
8. Success message propagates back to User

---

### Sequence Diagram Example 2: ATM Withdrawal

```
Customer      ATM         Bank System    Database
   │           │               │            │
   │           │               │            │
   │─insertCard→│              │            │
   │           │               │            │
   │─enterPIN─→│               │            │
   │           │               │            │
   │           │─validatePIN()→│            │
   │           │               ║            │
   │           │               ║─checkPIN()→│
   │           │               ║            ║
   │           │               ║←─valid─────║
   │           │               │            │
   │           │←─authorized───│            │
   │           │               │            │
   │─enterAmount│              │            │
   │           │               │            │
   │           │─withdraw()───→│            │
   │           │               ║            │
   │           │               ║─checkBalance→│
   │           │               ║←─balance────║
   │           │               ║             │
   │           │               ║─deduct()───→│
   │           │               ║←─success────║
   │           │               │             │
   │           │←─dispenseCash─│             │
   │           │               │             │
   │←─getCash──│               │             │
   │           │               │             │
```

**Key Elements:**

- Clear time progression from top to bottom
- Shows all interactions between components
- Helps understand the complete flow of withdrawal process

---

### When to Use Sequence Diagrams?

✅ **Use when:**

- Explaining complex workflows
- Showing order of operations
- Demonstrating object interactions in a specific scenario
- Understanding API call sequences
- Clarifying async vs sync operations

❌ **Not needed when:**

- Simple CRUD operations
- Static structure is sufficient
- Only class relationships matter

---

## Key Takeaways

### UML Diagram Essentials

- **Two main types**: Structural (static) and Behavioral (dynamic)
- **For LLD interviews**: Master Class Diagrams and Sequence Diagrams
- 99% of interviews require Class Diagrams before coding

### Class Diagram

- Three sections: Name, Attributes (characteristics), Methods (behaviors)
- Access modifiers: `+` (public), `#` (protected),  (private)
- Four main relationships:
    - **Association**: Basic connection (solid line)
    - **Aggregation**: Weak has-a (empty diamond) - independent lifecycle
    - **Composition**: Strong has-a (filled diamond) - dependent lifecycle
    - **Inheritance**: IS-A relationship (arrow to parent)
- Use cardinality to show multiplicity (1, 0..1, *, 1..*)

### Sequence Diagram

- Shows time-based interaction flow
- Components: Actors/Objects (boxes), Lifelines (dashed vertical), Messages (arrows), Activation boxes
- Helps clarify complex workflows and method call sequences
- Not always required but extremely useful for understanding behavior

### Interview Strategy

- Always draw Class Diagram before writing code
- Use Sequence Diagrams for complex interaction scenarios
- Practice drawing diagrams for common LLD problems (Parking Lot, Library Management, etc.)
- Clear diagrams demonstrate strong design thinking
