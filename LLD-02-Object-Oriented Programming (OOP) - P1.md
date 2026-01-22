### Object-Oriented Programming (OOP) - P1

## Introduction

OOP is a **strong foundation** for understanding Low-Level Design (LLD). Once you master OOP concepts, understanding LLD becomes much easier. This lecture covers OOP concepts and its four pillars with practical examples.

---

## History of Programming Languages

### Why Study History?

To understand OOP, we must first understand **why** we needed to evolve to object-oriented programming. What problems existed in earlier languages that OOP solved?

### 1. Machine Language

**What it was:**

- Programming directly in **binary code** (0s and 1s)
- Directly interacted with CPU
- Code example: `01010110101...`

**Problems:**

- **Highly error-prone** - One wrong bit changes entire code
- **Tedious process** - Extremely difficult to write
- **Not scalable** - Building large applications was impossible
- Direct CPU execution without abstraction

```
Example: 01010110101001...
```

### 2. Assembly Level Language

**What it introduced:**

- **Mnemonics** - English keywords instead of binary
- Example: `MOV A, 61H` (Move value 61H to register A)

**Problems:**

- Still **tightly coupled with hardware** - Hardware change requires code change
- **Prone to errors** - Managing individual instructions and registers manually
- **Low scalability** - Not suitable for large software
- **Limited abstractions** - No loops or methods as we know them today
- **Tedious** - Still difficult to work with

### 3. Procedural Programming

**What it introduced:**

- **Functions** - Code modularization
- **Loops** - for, while iterations
- **Blocks** - if-else, switch statements
- Example: **C language**

**Characteristics:**

- Code as a **recipe book** - “Do this, then do that”
- Set of sequential instructions
- Could solve small-level problems

**Limitations:**

- ❌ Could NOT build **large-scale applications**
- ❌ Could NOT solve **complex real-world problems**
- ❌ Lacked **real-world modeling**
- ❌ No **data security**
- ❌ Limited **scalability** and **reusability**

---

## Why Object-Oriented Programming?

### Three Major Problems OOP Solves

### 1. **Real-World Modeling**

- Need to model code the way real world works
- Real world consists of **objects** that interact with each other
- Examples: You, me, microphone, laptop, camera - all are objects
- Objects interact: Speaker interacts with microphone, you interact with laptop

### 2. **Data Security**

- Procedural programming lacked proper data protection
- OOP introduces **encapsulation** to secure data

### 3. **Highly Scalable & Reusable Applications**

- Build complex, enterprise-level applications
- Examples: Uber clone, Ola clone, Zomato clone, Paytm clone
- Code reusability through inheritance and polymorphism

---

## Core OOP Concepts

### Understanding Objects

**An object has two components:**

1. **Characteristics** (Properties/Attributes)
    - Unique identifiers that define the object
    - In programming: **Variables** or **Data Members**
2. **Behavior** (Actions)
    - Functions the object can perform
    - In programming: **Methods** or **Member Functions**

### Real-Life Example: Car

### Car Characteristics:

- **Brand** - String
- **Model** - String
- **Engine** - Engine object
- **isEngineOn** - Boolean

### Car Behaviors:

- `start()` - Start the car
- `stop()` - Stop the car
- `shiftGear()` - Change gear
- `accelerate()` - Increase speed
- `applyBrake()` - Apply brakes

### Classes: Blueprint of Objects

**Class** = Blueprint to create objects

```java
class Car {
    // Characteristics (Data Members)
    String brand;
    String model;
    boolean isEngineOn;

    // Behaviors (Methods)
    void start() {
        // start logic
    }

    void stop() {
        // stop logic
    }

    void shiftGear() {
        // gear shift logic
    }
}
```

**Creating objects from class:**

```java
Car myCar = new Car(); // Creating a car object
```

---

## Problem with Procedural Programming: Example

### Scenario: Car and Owner

**In Procedural Programming (without OOP):**

To represent a car, we’d declare separate variables:

```java
String brand;
String model;
boolean isEngineOn;

// Separate methods
void start() { }
void stop() { }
void shiftGear() { }
```

**Problem:** This represents ONE car.

Now add an **Owner**:

- Owner is also an object
- Owner has name
- Owner **owns** the car
- Owner can **drive()** the car

**Real-world interaction:** Owner owns Car, Owner drives Car

**In Procedural Code:**

```java
// Car variables
String carBrand;
String carModel;
boolean carEngineOn;

// Owner variables
String ownerName;

// Car methods
void carStart() { }
void carStop() { }
void carShiftGear() { }

// Owner methods
void ownerDrive() { }
```

### Problems Arise:

1. **Poor Real-World Modeling**
    - No clear relationship between Owner and Car
    - All variables are scattered globally
    - Difficult to understand who owns what
    - Cannot easily represent: “Owner owns Car”
2. **Data Security Issues**
    - All variables are **global** or accessible everywhere
    - Anyone can modify `carBrand`, `ownerName`, etc.
    - No protection mechanism
    - Data can be accidentally corrupted
3. **Scalability Issues**
    - What if there are **multiple cars**?
    - Need: `car1Brand`, `car2Brand`, `car3Brand`…
    - Need: `car1Start()`, `car2Start()`, `car3Start()`…
    - Code becomes **unmanageable** quickly
    - **No code reusability**
4. **Complexity Explosion**
    - Adding more objects (Driver, Mechanic, Garage) makes code exponentially complex
    - Maintenance nightmare
    - Debugging becomes extremely difficult

---

## OOP Solution: Classes and Objects

### With OOP Approach

```java
class Car {
    // Private data members (Data Security)
    private String brand;
    private String model;
    private boolean isEngineOn;

    // Public methods (Controlled access)
    public void start() {
        isEngineOn = true;
        System.out.println("Car started");
    }

    public void stop() {
        isEngineOn = false;
        System.out.println("Car stopped");
    }

    public void shiftGear() {
        System.out.println("Gear shifted");
    }
}

class Owner {
    private String name;
    private Car car; // Owner HAS-A Car (Composition)

    public void drive() {
        car.start();
        System.out.println(name + " is driving");
    }
}
```

**Creating objects:**

```java
Car myCar = new Car();
Owner owner = new Owner();
owner.setCar(myCar); // Owner owns this car
owner.drive(); // Owner drives his car
```

### Benefits:

✅ **Real-World Modeling**

- Clear relationship: Owner HAS-A Car
- Code structure mirrors real-world relationships
- Easy to understand and maintain

✅ **Data Security (Encapsulation)**

- Private variables cannot be accessed directly
- Only through public methods
- Controlled access prevents accidental modification

✅ **Scalability & Reusability**

- Create multiple car objects easily: `car1`, `car2`, `car3`
- Same class blueprint reused
- Easy to add new features

✅ **Maintainability**

- Changes to Car class don’t affect Owner class (Loose coupling)
- Modular code structure
- Easy to debug and extend

---

## Visualizing the Concept

### Procedural vs OOP:

```
┌─────────────────────────────┐
│   PROCEDURAL APPROACH       │
├─────────────────────────────┤
│ carBrand                    │
│ carModel                    │
│ ownerName                   │  <- All scattered
│ car1Brand                   │
│ car2Brand                   │  <- Duplication
│ carStart()                  │
│ car1Start()                 │  <- More duplication
│ ownerDrive()                │
└─────────────────────────────┘

❌ No clear structure
❌ No data protection
❌ Hard to scale
```

```
┌──────────────────────┐    ┌──────────────────────┐
│   CAR CLASS          │    │   OWNER CLASS        │
├──────────────────────┤    ├──────────────────────┤
│ - brand              │◄───│ - name               │
│ - model              │owns│ - car                │
│ - isEngineOn         │    │                      │
├──────────────────────┤    ├──────────────────────┤
│ + start()            │    │ + drive()            │
│ + stop()             │    │                      │
│ + shiftGear()        │    │                      │
└──────────────────────┘    └──────────────────────┘

✅ Clear structure
✅ Data encapsulated
✅ Easily scalable
```

---

## Key Concept: Real-World = Objects Interacting

**Mental Model:**

- Everything in real world is an **object**
- Objects have **characteristics** (what they are)
- Objects have **behaviors** (what they do)
- Objects **interact** with each other

**Examples of Object Interactions:**

- Speaker speaks into Microphone → Sound captured
- User interacts with Laptop → Video plays
- Instructor looks at Camera → Video recorded
- **Owner drives Car** → Car moves

**OOP Goal:** Model these interactions in code naturally

---

## Key Takeaways

1. **History Context**: OOP evolved to solve limitations of Machine, Assembly, and Procedural languages
2. **Three Core Problems OOP Solves:**
    - Real-world modeling capability
    - Data security through encapsulation
    - Building scalable & reusable applications
3. **Object = Characteristics + Behavior**
    - Characteristics → Data members/variables
    - Behavior → Methods/functions
4. **Class = Blueprint**, Object = Instance
    - Class defines structure
    - Objects are actual instances created from class
5. **OOP vs Procedural:**
    - Procedural: Scattered variables, no protection, poor scalability
    - OOP: Encapsulated data, clear structure, highly scalable
6. **Foundation for LLD**: Mastering OOP is essential for understanding Low-Level Design patterns and principles
