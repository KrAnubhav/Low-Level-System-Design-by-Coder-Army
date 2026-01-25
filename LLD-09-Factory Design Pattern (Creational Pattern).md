# LLD-09: Factory Design Pattern (Creational Pattern) 🏭

## Introduction

**Welcome to Factory Design Pattern!** 🎉

Yeh bhi **Strategy Pattern** ki tarah **most used design pattern** hai!

**Usage:**
- 99% LLD interviews mein lagega
- Real-world applications mein extensively use hota hai
- **Creational Pattern** - Object creation ke liye

---

## What is Factory? 🤔

### Real-World Factory

> **Real-world factory** = Jahan se products bante hain

### LLD Factory

> **Factory in LLD** = Ek class jahan se **objects** create hote hain

**Question:** Hume object creation ke liye alag class kyun chahiye?

**Answer:** Decouple karne ke liye! 🎯

---

## Why Factory Pattern? 

### Recap: Strategy Pattern

**Strategy Pattern mein kya tha?**

```
┌──────────┐
│  Client  │ (Robot)
└──────────┘
     │
     │ has-a (Composition)
     ▼
┌──────────────┐
│  Strategies  │
├──────────────┤
│ • Talkable   │
│ • Walkable   │
│ • Flyable    │
└──────────────┘
```

**Strategy mein assumption:**
- Objects **already created** hain
- Hum sirf methods call kar rahe hain
- Object creation logic **abstracted** hai

**Example:**
```java
// Yeh assume kiya tha
Talkable t = new NormalTalk();  // Already created somewhere
Walkable w = new NormalWalk();
Flyable f = new NoFly();

Robot robot = new Robot(t, w, f);  // Dependency Injection
```

---

### The Problem

**Har application mein 2 types ka logic hota hai:**

1. **Business Logic** 📊
   - Application ka main logic
   - Example: Notification kaise jayegi, kis-kis ko jayegi

2. **Object Creation Logic** 🏗️
   - Objects kab aur kaise create honge
   - `new` keyword ka use

**Problem:**
- Agar dono ek saath ho toh **code complex** ho jata hai
- Read karna mushkil
- Understand karna mushkil

**Solution:** 
> **Separate business logic from object creation logic!**

---

## Factory Pattern Goal 🎯

### Core Principle

> **"Decouple client from object creation"**

**Kaise?**

```
┌──────────┐                ┌──────────┐
│  Client  │ ─────asks────→ │ Factory  │
└──────────┘                └──────────┘
     ↑                            │
     │                            │ creates
     └────────returns object──────┘
```

**Benefits:**
- Client ko **pata nahi** object kaise banta hai
- Client ko **pata nahi** kya logic lagta hai
- Client bas bolta hai: "Mujhe object de do!"
- Factory object la ke de deti hai

---

## Types of Factory Patterns

Factory pattern ke **3 types** hain:

```
┌─────────────────────────────────┐
│   Factory Pattern Types         │
├─────────────────────────────────┤
│ 1. Simple Factory               │ ← Not a design pattern
│ 2. Factory Method               │ ← Proper design pattern
│ 3. Abstract Factory Method      │ ← Proper design pattern
└─────────────────────────────────┘
```

**Note:** 
- **Simple Factory** = Design principle (not pattern)
- **Factory Method** & **Abstract Factory** = Proper design patterns

---

## 1. Simple Factory

### Example: Burger Shop 🍔

**Scenario:** Ek burger shop hai jahan 3 types ke burgers milte hain

### Class Hierarchy

<img width="836" height="409" alt="image" src="https://github.com/user-attachments/assets/60edf0e8-85b1-4801-8c06-a2191337ca90" />


### Simple Factory Class

<img width="531" height="417" alt="image" src="https://github.com/user-attachments/assets/992cbe07-9aca-43e9-95cd-b083e8a24acb" />
<img width="650" height="162" alt="image" src="https://github.com/user-attachments/assets/11254760-7576-4253-905f-5ae4347adb77" />



### Java Implementation

```java
// ============= PRODUCT =============

abstract class Burger {
    public abstract void prepare();
}

class BasicBurger extends Burger {
    @Override
    public void prepare() {
        System.out.println("Preparing Basic Burger with bun and patty");
    }
}

class StandardBurger extends Burger {
    @Override
    public void prepare() {
        System.out.println("Preparing Standard Burger with bun, patty, cheese, and lettuce");
    }
}

class PremiumBurger extends Burger {
    @Override
    public void prepare() {
        System.out.println("Preparing Premium Burger - Gourmet style!");
    }
}

// ============= FACTORY =============

class BurgerFactory {
    public Burger createBurger(String type) {
        if (type.equals("basic")) {
            return new BasicBurger();
        } else if (type.equals("standard")) {
            return new StandardBurger();
        } else if (type.equals("premium")) {
            return new PremiumBurger();
        } else {
            return null;  // Invalid burger type
        }
    }
}

// ============= CLIENT =============

public class Main {
    public static void main(String[] args) {
        String type = "standard";
        
        // Create factory
        BurgerFactory factory = new BurgerFactory();
        
        // Get burger from factory
        Burger burger = factory.createBurger(type);
        
        // Use burger
        burger.prepare();
    }
}
```

**Output:**
```
Preparing Standard Burger with bun, patty, cheese, and lettuce
```

---

### Simple Factory - Standard UML

### Definition

> **Simple Factory:** A factory class that decides which concrete class to instantiate

**Key Point:** Type ke basis pe decide hota hai konsa concrete class banana hai

---

## 2. Factory Method Pattern

### Evolution from Simple Factory

**Simple Factory mein:**
- Product hierarchy thi (Burger → Basic, Standard, Premium)
- Factory **concrete** thi (BurgerFactory)

**Factory Method mein:**
- Product hierarchy bhi hai
- **Factory hierarchy bhi hai!** 🎯

### Example: Multiple Burger Franchises

**Scenario:**
- 2 franchises: **SingBurger** aur **KingBurger**
- SingBurger → Normal burgers banata hai
- KingBurger → Wheat burgers banata hai

### Extended Product Hierarchy

<img width="1440" height="819" alt="image" src="https://github.com/user-attachments/assets/c7a9d80d-f228-4454-ae41-b8c4760eb86f" />


### Factory Hierarchy

### Java Implementation

```java
// ============= PRODUCTS =============

abstract class Burger {
    public abstract void prepare();
}

// Normal Burgers
class BasicBurger extends Burger {
    @Override
    public void prepare() {
        System.out.println("Preparing Basic Burger with one patty and ketchup");
    }
}

class StandardBurger extends Burger {
    @Override
    public void prepare() {
        System.out.println("Preparing Standard Burger with one patty, cheese, and lettuce");
    }
}

class PremiumBurger extends Burger {
    @Override
    public void prepare() {
        System.out.println("Preparing Premium Burger - Gourmet buns!");
    }
}

// Wheat Burgers
class BasicWheatBurger extends Burger {
    @Override
    public void prepare() {
        System.out.println("Preparing Basic Wheat Burger with one patty and ketchup");
    }
}

class StandardWheatBurger extends Burger {
    @Override
    public void prepare() {
        System.out.println("Preparing Standard Wheat Burger with one patty, cheese, and lettuce");
    }
}

class PremiumWheatBurger extends Burger {
    @Override
    public void prepare() {
        System.out.println("Preparing Premium Wheat Burger - Healthy gourmet!");
    }
}

// ============= FACTORY HIERARCHY =============

abstract class BurgerFactory {
    public abstract Burger createBurger(String type);
}

// SingBurger - Makes normal burgers
class SingBurger extends BurgerFactory {
    @Override
    public Burger createBurger(String type) {
        if (type.equals("basic")) {
            return new BasicBurger();
        } else if (type.equals("standard")) {
            return new StandardBurger();
        } else if (type.equals("premium")) {
            return new PremiumBurger();
        }
        return null;
    }
}

// KingBurger - Makes wheat burgers
class KingBurger extends BurgerFactory {
    @Override
    public Burger createBurger(String type) {
        if (type.equals("basic")) {
            return new BasicWheatBurger();
        } else if (type.equals("standard")) {
            return new StandardWheatBurger();
        } else if (type.equals("premium")) {
            return new PremiumWheatBurger();
        }
        return null;
    }
}

// ============= CLIENT =============

public class Main {
    public static void main(String[] args) {
        String type = "basic";
        
        // Choose factory - KingBurger (wheat burgers)
        BurgerFactory factory = new KingBurger();
        
        // Get burger
        Burger burger = factory.createBurger(type);
        
        // Use burger
        burger.prepare();
    }
}
```

**Output:**
```
Preparing Basic Wheat Burger with one patty and ketchup
```

**Agar `SingBurger` use karte:**
```
Preparing Basic Burger with one patty and ketchup
```

---

### Factory Method - Standard UML

<img width="1058" height="820" alt="image" src="https://github.com/user-attachments/assets/7e75fe7b-f811-4d42-a988-273e15a0441b" />
<img width="613" height="250" alt="image" src="https://github.com/user-attachments/assets/7a5975ef-0e51-4138-852a-8722906df7fd" />


### Definition

> **Factory Method:** Define an interface for creating objects, but allow subclasses to decide which class to instantiate

**Key Point:** Subclasses decide kaunsa product banana hai!

---

## 3. Abstract Factory Pattern

### Evolution from Factory Method

**Factory Method mein:**
- Ek factory **ek type** ka product banati thi
- SingBurger → sirf burgers
- KingBurger → sirf burgers

**Abstract Factory mein:**
- Ek factory **multiple types** ke products banati hai! 🎯
- SingBurger → Burgers + Garlic Bread
- KingBurger → Burgers + Garlic Bread

### Example: Multiple Products

**Scenario:**
- Dono franchises ab **2 products** banate hain:
  1. Burgers
  2. Garlic Bread

### Product Hierarchies

**Product 1: Burger**
```
Burger
  ├── BasicBurger
  ├── StandardBurger
  ├── PremiumBurger
  ├── BasicWheatBurger
  ├── StandardWheatBurger
  └── PremiumWheatBurger
```

**Product 2: Garlic Bread**
```
GarlicBread
  ├── BasicGarlicBread
  ├── CheeseGarlicBread
  ├── BasicWheatGarlicBread
  └── CheeseWheatGarlicBread
```

### Factory Hierarchy
<img width="1238" height="692" alt="image" src="https://github.com/user-attachments/assets/4702d815-f59a-4dba-b970-fa644b3157fc" />


```
┌─────────────────────────┐
│   <<abstract>>          │
│    MealFactory          │
├─────────────────────────┤
│ + createBurger(type)    │
│ + createGarlicBread(type)│
└─────────────────────────┘
         △
         │
    ┌────┴────┐
    │         │
┌───┴──────┐ ┌┴──────────┐
│  Sing    │ │   King    │
│  Burger  │ │  Burger   │
├──────────┤ ├───────────┤
│+createB()│ │+createB() │
│+createGB()│ │+createGB()│
└──────────┘ └───────────┘
```

**SingBurger creates:**
- Normal Burgers
- Normal Garlic Bread

**KingBurger creates:**
- Wheat Burgers
- Wheat Garlic Bread

### Java Implementation

```java
// ============= PRODUCT 1: BURGER =============

abstract class Burger {
    public abstract void prepare();
}

class BasicBurger extends Burger {
    @Override
    public void prepare() {
        System.out.println("Preparing Basic Burger with one patty and ketchup");
    }
}

class StandardBurger extends Burger {
    @Override
    public void prepare() {
        System.out.println("Preparing Standard Burger with patty, cheese, lettuce");
    }
}

class PremiumBurger extends Burger {
    @Override
    public void prepare() {
        System.out.println("Preparing Premium Burger - Gourmet!");
    }
}

class BasicWheatBurger extends Burger {
    @Override
    public void prepare() {
        System.out.println("Preparing Basic Wheat Burger with one patty and ketchup");
    }
}

class StandardWheatBurger extends Burger {
    @Override
    public void prepare() {
        System.out.println("Preparing Standard Wheat Burger with patty, cheese, lettuce");
    }
}

class PremiumWheatBurger extends Burger {
    @Override
    public void prepare() {
        System.out.println("Preparing Premium Wheat Burger - Healthy gourmet!");
    }
}

// ============= PRODUCT 2: GARLIC BREAD =============

abstract class GarlicBread {
    public abstract void prepare();
}

class BasicGarlicBread extends GarlicBread {
    @Override
    public void prepare() {
        System.out.println("Preparing Basic Garlic Bread");
    }
}

class CheeseGarlicBread extends GarlicBread {
    @Override
    public void prepare() {
        System.out.println("Preparing Cheese Garlic Bread with extra cheese");
    }
}

class BasicWheatGarlicBread extends GarlicBread {
    @Override
    public void prepare() {
        System.out.println("Preparing Basic Wheat Garlic Bread");
    }
}

class CheeseWheatGarlicBread extends GarlicBread {
    @Override
    public void prepare() {
        System.out.println("Preparing Cheese Wheat Garlic Bread with extra cheese");
    }
}

// ============= ABSTRACT FACTORY =============

abstract class MealFactory {
    public abstract Burger createBurger(String type);
    public abstract GarlicBread createGarlicBread(String type);
}

// SingBurger - Normal products
class SingBurger extends MealFactory {
    @Override
    public Burger createBurger(String type) {
        if (type.equals("basic")) {
            return new BasicBurger();
        } else if (type.equals("standard")) {
            return new StandardBurger();
        } else if (type.equals("premium")) {
            return new PremiumBurger();
        }
        return null;
    }
    
    @Override
    public GarlicBread createGarlicBread(String type) {
        if (type.equals("basic")) {
            return new BasicGarlicBread();
        } else if (type.equals("cheese")) {
            return new CheeseGarlicBread();
        }
        return null;
    }
}

// KingBurger - Wheat products
class KingBurger extends MealFactory {
    @Override
    public Burger createBurger(String type) {
        if (type.equals("basic")) {
            return new BasicWheatBurger();
        } else if (type.equals("standard")) {
            return new StandardWheatBurger();
        } else if (type.equals("premium")) {
            return new PremiumWheatBurger();
        }
        return null;
    }
    
    @Override
    public GarlicBread createGarlicBread(String type) {
        if (type.equals("basic")) {
            return new BasicWheatGarlicBread();
        } else if (type.equals("cheese")) {
            return new CheeseWheatGarlicBread();
        }
        return null;
    }
}

// ============= CLIENT =============

public class Main {
    public static void main(String[] args) {
        String burgerType = "basic";
        String garlicBreadType = "cheese";
        
        // Choose factory - KingBurger (wheat products)
        MealFactory factory = new KingBurger();
        
        // Create products
        Burger burger = factory.createBurger(burgerType);
        GarlicBread garlicBread = factory.createGarlicBread(garlicBreadType);
        
        // Use products
        burger.prepare();
        garlicBread.prepare();
    }
}
```

**Output:**
```
Preparing Basic Wheat Burger with one patty and ketchup
Preparing Cheese Wheat Garlic Bread with extra cheese
```

---

### Abstract Factory - Standard UML

```
┌──────────────┐  ┌──────────────┐
│  Product A   │  │  Product B   │
└──────────────┘  └──────────────┘
       △                 △
       │                 │
   ┌───┴───┐         ┌───┴───┐
   │       │         │       │
┌──┴──┐ ┌─┴──┐   ┌──┴──┐ ┌─┴──┐
│CP-A1│ │CP-A2│   │CP-B1│ │CP-B2│
└─────┘ └────┘   └─────┘ └────┘
   ↑      ↑         ↑      ↑
   │      │         │      │
   └──┬───┘         └──┬───┘
      │                │
┌─────┴────────────────┴─────┐
│   <<abstract>>             │
│      Factory               │
├────────────────────────────┤
│ + createProductA()         │
│ + createProductB()         │
└────────────────────────────┘
         △
         │
    ┌────┴────┐
    │         │
┌───┴────┐ ┌─┴────────┐
│Factory │ │ Factory  │
│   1    │ │    2     │
└────────┘ └──────────┘
```

### Definition

> **Abstract Factory:** Provides an interface for creating families of related objects without specifying their concrete classes

**Key Point:** 
- **Families of related objects** = Ek factory multiple products banati hai
- SingBurger → Normal Burger + Normal Garlic Bread (related family)
- KingBurger → Wheat Burger + Wheat Garlic Bread (related family)

---

## Comparison Table

| Aspect | Simple Factory | Factory Method | Abstract Factory |
|--------|---------------|----------------|------------------|
| **Pattern?** | ❌ No (Principle) | ✅ Yes | ✅ Yes |
| **Factory Hierarchy** | ❌ No | ✅ Yes | ✅ Yes |
| **Products per Factory** | Multiple | 1 type | Multiple types |
| **Complexity** | Low | Medium | High |
| **Use Case** | Simple object creation | Multiple factories, 1 product | Multiple factories, multiple products |

---

## Real-World Example: Notification System 📱

### Scenario

Ek notification system hai jo different types ki notifications bhejta hai.

### Design

```
┌──────────────────────┐
│ NotificationSystem   │ ← Abstract
├──────────────────────┤
│ + send()             │
└──────────────────────┘
         △
         │
    ┌────┼────┬────────┐
    │    │    │        │
┌───┴─┐ ┌┴──┐ ┌┴────┐
│ SMS │ │Push│ │Email│
└─────┘ └───┘ └─────┘
   ↑     ↑     ↑
   │     │     │
   └─────┴─────┘
         │
┌────────┴─────────┐
│NotificationFactory│
├──────────────────┤
│+ create(type)    │
└──────────────────┘
```

### Implementation

```java
// Factory approach
NotificationFactory factory = new NotificationFactory();
NotificationSystem notif = factory.create("sms");
notif.send("Hello!");
```

---

## Factory vs Strategy 🤔

### Confusion Point

**Dono patterns se same problem solve ho sakti hai!**

**Example:** Notification System

**Factory Approach:**
```java
NotificationFactory factory = new NotificationFactory();
Notification notif = factory.create("sms");  // Object creation
notif.send();
```

**Strategy Approach:**
```java
NotificationSystem system = new NotificationSystem(new SMSStrategy());
system.send();  // Assumes object already created
```

---

### When to Use What?

**Use Factory when:**
✅ Object creation logic ko separate karna hai  
✅ Business logic se decouple karna hai  
✅ Runtime pe decide karna hai konsa object banana hai  

**Use Strategy when:**
✅ Algorithms ko vary karna hai runtime pe  
✅ Objects already created hain  
✅ Behavior change karna hai dynamically  

### Thumb Rule

**Question to ask yourself:**

1. **Kya mujhe algorithms vary karne hain?**
   - YES → Strategy Pattern
   - Objects already exist, sirf methods call karne hain

2. **Kya mujhe object creation logic separate karna hai?**
   - YES → Factory Pattern
   - Objects create karne hain with specific logic

**Remember:** 
- Multiple design patterns se **same problem** solve ho sakti hai
- **Intent** alag hota hai
- **Subjective opinion** hota hai
- Koi right/wrong answer nahi - better explanation matters!

---

## Summary 📚

### Factory Pattern Types

**1. Simple Factory**
- Ek factory class
- Type ke basis pe object create karta hai
- Not a design pattern (principle hai)

**2. Factory Method**
- Factory hierarchy
- Subclasses decide kaunsa product banana hai
- Proper design pattern

**3. Abstract Factory**
- Factory hierarchy
- Multiple products per factory
- Families of related objects
- Proper design pattern

### Key Takeaways

✅ **Decouple** object creation from business logic  
✅ **Client** ko object creation logic nahi pata  
✅ **Factory** responsible hai objects banane ke liye  
✅ **Scalable** - Naye products easily add kar sakte ho  
✅ **99% interviews** mein use hota hai  

---

## Practice Exercise 💪

**Try implementing:**

1. **Vehicle Factory** - Car, Bike, Truck (different types)
2. **Database Factory** - MySQL, PostgreSQL, MongoDB connections
3. **UI Component Factory** - Button, TextField, Dropdown (different themes)
4. **Payment Factory** - UPI, Card, NetBanking processors

---

## Concluding Points

✅ **Factory Pattern** = Object creation ke liye  
✅ **Creational Pattern** hai  
✅ **Most used** pattern in real-world  
✅ **Strategy vs Factory** - Intent alag hai  
✅ **Multiple solutions** possible - Best explanation matters  

---

*Next: Singleton Design Pattern (Creational Pattern)*

**Remember:** Factory Pattern is about **decoupling object creation** from business logic! 🏭
