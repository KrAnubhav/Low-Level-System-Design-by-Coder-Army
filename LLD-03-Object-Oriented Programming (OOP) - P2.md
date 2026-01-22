### **Object-Oriented Programming (OOP) - P2**

## Recap

In the previous lecture, we covered two pillars of OOP:

- **Abstraction**
- **Encapsulation**

Today, we’ll cover the remaining two pillars:

- **Inheritance**
- **Polymorphism**

We’ll continue with the **Car example** used in the previous lecture for consistency and easier understanding.

---

## Inheritance (इनहेरिटेंस)

### What is Inheritance?

**Core Concept**: In real life, objects are related to each other through relationships, typically **parent-child relationships**. Since objects exhibit this inheritance property in real life, we must represent it programmatically.

**Definition**: Inheritance is a mechanism where one class (child) acquires the properties and behaviors of another class (parent).

### Real-World Example: Car Hierarchy

```
         Car (Parent)
         /         \
        /           \
Manual Car      Electric Car
(Child)          (Child)
```

### Parent: Car

**Characteristics (Common to all cars)**:

- Brand
- Model
- isEngineOn (boolean)
- currentSpeed

**Behaviors (Common to all cars)**:

- startEngine()
- stopEngine()
- accelerate()
- brake()

These features exist in **every car**, regardless of type.

---

### Child 1: Manual Car

**Specific Characteristics**:

- currentGear (integer)

**Specific Behaviors**:

- shiftGear()

These features are **specific to manual cars only** - not found in every car.

---

### Child 2: Electric Car

**Specific Characteristics**:

- batteryPercentage (integer)

**Specific Behaviors**:

- chargeBattery()

These features are **specific to electric cars only**.

---

### Key Insight

- **Parent class** (Car): Contains common features shared by all cars
- **Child classes** (Manual Car, Electric Car):
    - Inherit all common features from parent
    - Add their own specific features
    - **No need to redefine** parent features

---

## Implementation in Code

```java
// Parent Class
class Car {
    // Common characteristics
    String brand;
    String model;
    boolean isEngineOn;
    int currentSpeed;

    // Common behaviors
    void startEngine() { }
    void stopEngine() { }
    void accelerate() { }
    void brake() { }
}

// Child Class 1
class ManualCar extends Car {
    // Specific to manual car
    int currentGear;

    void shiftGear() { }
}

// Child Class 2
class ElectricCar extends Car {
    // Specific to electric car
    int batteryPercentage;

    void chargeBattery() { }
}
```

### Inheritance Relationship Diagram

```
┌─────────────────────────────────────────┐
│              Car (Parent)               │
│─────────────────────────────────────────│
│ - brand                                 │
│ - model                                 │
│ - isEngineOn                            │
│ - currentSpeed                          │
│─────────────────────────────────────────│
│ + startEngine()                         │
│ + stopEngine()                          │
│ + accelerate()                          │
│ + brake()                               │
└─────────────────────────────────────────┘
           ▲                 ▲
           │                 │
    ┌──────┘                 └──────┐
    │                               │
┌───────────────┐          ┌────────────────┐
│  ManualCar    │          │  ElectricCar   │
│───────────────│          │────────────────│
│ - currentGear │          │ - batteryPct   │
│───────────────│          │────────────────│
│ + shiftGear() │          │ + chargeBattery│
└───────────────┘          └────────────────┘
```

### How It Works

1. **ManualCar** inherits from Car using: `class ManualCar extends Car`
2. ManualCar automatically gets:
    - All public/protected characteristics from Car
    - All public/protected behaviors from Car
3. ManualCar only defines what’s **specific** to it:
    - currentGear
    - shiftGear()

**Result**: No code duplication! Both child classes can access parent features without redefining them.

---

## Access Modifiers in Inheritance

### The Three Access Modifiers

We now complete the understanding of all three access modifiers:

| Modifier | Accessible From | Use Case |
| --- | --- | --- |
| **public** | Anywhere (inside class, outside class, child classes) | Features meant to be publicly available |
| **private** | Only within the same class | Features meant to be hidden/internal |
| **protected** | Within the class + child classes (NOT outside) | Features meant for inheritance but not public access |

---

### Protected Keyword (Most Important for Inheritance)

**Definition**: `protected` members cannot be accessed from outside the class (like private), BUT can be accessed by child classes.

**Example**:

```java
class Car {
    protected String brand;  // Accessible in child classes
    private String model;    // NOT accessible in child classes
}

class ManualCar extends Car {
    void display() {
        System.out.println(brand);  // ✓ Works (protected)
        System.out.println(model);  // ✗ Error (private)
    }
}
```

**Key Point**:

- Private members are **never** accessible in child classes
- Protected members are **only** accessible in child classes and same class
- Public members are accessible **everywhere**

---

### Inheritance Type Syntax

When inheriting, we specify the inheritance type:

```java
class ManualCar extends Car  // In Java (public by default)
```

```cpp
class ManualCar : public Car { };     // Public inheritance
class ManualCar : private Car { };    // Private inheritance
class ManualCar : protected Car { };  // Protected inheritance
```

### What Does `public` Inheritance Mean?

When we write `class ManualCar extends Car`:

**If parent has**:

- `public` members → Stay `public` in child
- `protected` members → Stay `protected` in child
- `private` members → Not accessible in child (regardless of inheritance type)

**If we used `private` inheritance** (C++ only):

- `public` members in parent → Become `private` in child
- `protected` members in parent → Become `private` in child

**If we used `protected` inheritance** (C++ only):

- `public` members in parent → Become `protected` in child
- `protected` members in parent → Stay `protected` in child

**Note**: In real-world Java, we mostly use **public inheritance** (default). Private and protected inheritance are rarely used and mostly theoretical concepts.

---

## Why Inheritance is Useful

### Benefits

1. **Code Reusability**: Write common code once in parent class
2. **Avoid Duplication**: Child classes don’t repeat parent code
3. **Easier Maintenance**: Update parent, all children get the update
4. **Logical Organization**: Models real-world relationships
5. **Extensibility**: Easy to add new child classes

### Real-World Analogy

Think of inheritance like genetics:

- Parents pass traits (characteristics) to children
- Children inherit parent traits automatically
- Children also have unique traits of their own
- Children can have multiple siblings (other child classes)

---

## Summary of Inheritance

**Parent Class (Car)**:

- Contains common characteristics and behaviors
- Defines what ALL cars have

**Child Classes (ManualCar, ElectricCar)**:

- Inherit all parent features automatically
- Define only specific features unique to them
- Can access parent’s public and protected members
- Cannot access parent’s private members

**Syntax**:

- Java: `class ChildClass extends ParentClass`
- The `extends` keyword establishes parent-child relationship

**Access Control**:

- Use `protected` when you want child classes to access but not external code
- Use `private` when even child classes shouldn’t access
- Use `public` when everyone should access

---

## Key Takeaways

1. **Inheritance models parent-child relationships** from real world in code
2. **Parent class** contains common features shared by all children
3. **Child classes** inherit parent features + add their own specific features
4. **No code duplication** - write common code once in parent
5. **`protected` modifier** is specifically useful for inheritance (accessible to children only)
6. **Private members** are never accessible to child classes
7. **Inheritance is the easiest** of the four OOP pillars to understand
8. Use `extends` keyword (Java) or `:` operator (C++) to establish inheritance
9. **Public inheritance** (most common) keeps access modifiers unchanged in child
