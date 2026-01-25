# LLD-13: Decorator Design Pattern (Structural Pattern) 🎨

---

## 📋 Table of Contents

1. [Introduction](#introduction)
2. [The Core Philosophy](#the-core-philosophy)
   - [Composition over Inheritance](#composition-over-inheritance)
   - [Open/Closed Principle](#openclosed-principle)
3. [Problem Statement: The Class Explosion](#problem-statement-the-class-explosion)
   - [Scenario 1: The Coffee Shop Nightmare](#scenario-1-the-coffee-shop-nightmare)
   - [Scenario 2: The Gaming Character (Mario)](#scenario-2-the-gaming-character-mario)
   - [Why Inheritance Fails?](#why-inheritance-fails)
4. [Solution: The Decorator Pattern](#solution-the-decorator-pattern)
   - [Concept: Wrapping Objects](#concept-wrapping-objects)
   - [The "Is-A" and "Has-A" Duality](#the-is-a-and-has-a-duality)
   - [Visualizing the Wrapper Chain](#visualizing-the-wrapper-chain)
5. [Standard UML Architecture](#standard-uml-architecture)
6. [Implementation 1: Mario Game Power-ups](#implementation-1-mario-game-power-ups)
   - [Step-by-Step Code Construction](#step-by-step-code-construction)
   - [Execution Flow Analysis](#execution-flow-analysis)
7. [Implementation 2: Pizza Topping System](#implementation-2-pizza-topping-system)
   - [Problem](#problem)
   - [Code Solution](#code-solution)
8. [Real-World Deep Dive: Java I/O Streams](#real-world-deep-dive-java-io-streams)
   - [Understanding java.io Package](#understanding-javaio-package)
   - [Creating a Custom I/O Decorator](#creating-a-custom-io-decorator)
9. [Difference Between Patterns](#difference-between-patterns)
   - [Decorator vs Proxy](#decorator-vs-proxy)
   - [Decorator vs Adapter](#decorator-vs-adapter)
   - [Decorator vs Strategy](#decorator-vs-strategy)
10. [Advantages & Disadvantages](#advantages--disadvantages)
11. [Interview Questions (Q&A)](#interview-questions-qa)
12. [Conclusion](#conclusion)

---

## 🎯 Introduction

**Welcome Back Coder Army!**

Aaj ke lecture mein hum **Decorator Design Pattern** ko depth mein explore karenge. Yeh pattern sirf exam ya interview ke liye nahi, balki practical software development ke one of the most important principles ko embody karta hai.

Normally, jab hum kisi object ko nayi functionality dena chahte hain, humara pehla thought hota hai **Inheritance** (Subclassing). Par aaj hum dekhenge ki kaise Inheritance ek point ke baad "evil" ban jaati hai aur humara code maintain karna impossible ho jata hai.

**Decorator Pattern** humein allow karta hai ki hum functionalities ko **layers** ki tarah add karein, bilkul waise hi jaise hum thand mein kapde pehente hain.
- Pehle Shirt (Base Object)
- Fir Sweater (Decorator 1)
- Fir Jacket (Decorator 2)
- Fir Raincoat (Decorator 3)

End mein aap wahi insaan ho, par ab aapke paas **warmth** aur **waterproofing** ki additional properties hain!

---

## 🧠 The Core Philosophy

Decorator pattern ko samajhne ke liye humein Software Engineering ke do golden rules ko samajhna hoga.

### 1. Composition over Inheritance
Inheritance ("Is-A" relationship) static hoti hai. Compile time pe decide hoti hai.
- Agar `Dog` extends `Animal` hai, toh wo hamesha `Animal` rahega.
- Aap runtime pe `Dog` ko `Cat` nahi bana sakte.

Composition ("Has-A" relationship) dynamic hoti hai.
- Ek `Human` ke paas `Car` ho sakti hai.
- Runtime pe wo `Car` change karke `Bike` le sakta hai.

**Decorator Pattern kehta hai:**
> "Functionality inherit mat karo, functionality ko compose karo. Object ko wrap karo."

### 2. Open/Closed Principle (SOLID)
Ye principle kehta hai:
> "Classes should be **Open for Extension** but **Closed for Modification**."

Iska matlab:
- **Closed:** Hum existing working code (`Mario` class) ko touch nahi karenge taaki koi bug introduce na ho.
- **Open:** Hum nayi functionality (`GunPower`, `Flying`) add kar payenge bina purani class ko tode.

Decorator pattern is principle ka **perfect implementation** hai.

---

## ❓ Problem Statement: The Class Explosion

Chalo problems ko visualize karte hain taaki solution ki ahmiyat samajh aaye.

### Scenario 1: The Coffee Shop Nightmare ☕

Imagine aap ek Coffee Shop (like StarBucks) ka billing system bana rahe hain.

**Requirements:**
1. Base Beverages: `Espresso`, `Decaf`, `DarkRoast`
2. Add-ons (Condiments): `Milk`, `Mocha`, `Soy`, `Whip`

**Attempt 1: Brute Force Inheritance**

Agar hum har combination ke liye class banayein:

```
 Beverage
    ├── Espresso
    ├── EspressoWithMilk
    ├── EspressoWithMocha
    ├── EspressoWithMilkAndMocha
    ├── EspressoWithWhip
    ├── DarkRoast
    ├── DarkRoastWithMilk
    ├── DarkRoastWithSoy
    ...
```

**Calculation:**
Agar 4 beverages hain aur 4 condiments hain, aur humein sare combinations chahiye, toh classes ki counting **exponentially** badh jayegi.
Example: `DoubleMochaSoyLatteWithWhip` - Iske liye alag class? **Impossible!**

### Scenario 2: The Gaming Character (Mario) 🍄

Lectures mein diya gaya example: **Mario Game**.

**Base Character:** `Mario`
**Power-ups:**
1. `Mushroom` (Height Up)
2. `Flower` (Fireball Gun)
3. `Star` (Invincibility)
4. `Cape` (Flying)

**Inheritance Trap Example (Bad Code):**

```java
// Base Class
class Mario {
    void run() { System.out.println("Running"); }
    void jump() { System.out.println("Jumping"); }
}

// Subclassing for features
class MarioWithMushroom extends Mario {
    @Override
    void jump() { System.out.println("Jumping Higher!"); }
}

class MarioWithFlower extends Mario {
    void fire() { System.out.println("Shooting Fireballs!"); }
}

// THE PROBLEM: Need both?
class MarioWithMushroomAndFlower extends Mario {
    // Code Duplication !!
    // Hamein 'jump' logic Mushroom wala copy karna padega
    // Aur 'fire' logic Flower wala copy karna padega
    @Override
    void jump() { System.out.println("Jumping Higher!"); }
    void fire() { System.out.println("Shooting Fireballs!"); }
}
```

Is problem ko kehte hain **Class Explosion** 💥.
Jaise hi naya power-up aayega (e.g., `IceFlower`), humein purani saari combinations ke saath nayi classes banani padengi.

---

## 💡 Solution: The Decorator Pattern

Instead of inheritance causing a web of classes, we use **Wrappers**.

### Concept: Wrapping Objects

Hum ek object ko dusre object ke andar daal denge.

Imagine:
1. `Obj` (Original)
2. `Wrapper1` (Holds `Obj`)
3. `Wrapper2` (Holds `Wrapper1`)

Jab `Wrapper2` pe method call hoga, wo `Wrapper1` ko call karega, jo `Obj` ko call karega. Wapis aate hue har wrapper apni functionality add karega.

### The "Is-A" and "Has-A" Duality ☯️

Decorator pattern ki sabse khaas baat ye hai ki ye **dono** relationship use karta hai.

1. **Inheritance (Is-A): type matching ke liye.**
   - Decorator ko `BaseComponent` ka type hona padega taaki client ko pata na chale ki ye wrapper hai ya real object.
   - `MarioDecorator` **IS A** `Mario`.

2. **Composition (Has-A): behaviour chaining ke liye.**
   - Decorator ke paas ek reference hona chahiye us object ka jisko wo wrap kar raha hai.
   - `MarioDecorator` **HAS A** `Mario`.

### Visualizing the Wrapper Chain

Example: **Mario with Mushroom and Flower**

```
   Call triggered via Client
           │
           ▼
┌───────────────────────┐
│   FlowerDecorator     │
│  (Adds Functionality: │
│       Shooting)       │
│           │           │
│      HAS-A ref        │
│           │           │
│           ▼           │
│  ┌─────────────────┐  │
│  │MushroomDecorator│  │
│  │(Adds Func:      │  │
│  │  High Jump)     │  │
│  │        │        │  │
│  │    HAS-A ref    │  │
│  │        │        │  │
│  │        ▼        │  │
│  │  ┌───────────┐  │  │
│  │  │   Mario   │  │  │
│  │  │ (Original)│  │  │
│  │  └───────────┘  │  │
│  └─────────────────┘  │
└───────────────────────┘
```

---

## 🏗️ Standard UML Architecture

Is diagram ko dhyan se samajhna zaroori hai.

```
                  ┌────────────────────┐
                  │    <<interface>>   │
                  │      Component     │
                  ├────────────────────┤
                  │ + operation()      │
                  └─────────▲──────────┘
                            │
          ┌─────────────────┴───────────────────┐
          │                                     │
┌─────────┴─────────┐                 ┌─────────┴──────────┐
│ ConcreteComponent │                 │    Decorator       │
├───────────────────┤                 ├────────────────────┤
│ + operation()     │                 │ - component: Comp  │◄────────┐
└───────────────────┘                 ├────────────────────┤         │
                                      │ + operation()      │         │
                                      └─────────▲──────────┘         │
                                                │                    │
                          ┌─────────────────────┴────────────────┐   │
                          │                                      │   │
                  ┌───────┴──────────┐                  ┌────────┴───┴───────┐
                  │ ConcreteDecoratorA│                 │ ConcreteDecoratorB │
                  ├───────────────────┤                 ├────────────────────┤
                  │ + operation()     │                 │ + operation()      │
                  │ + addedBehavior() │                 │ + addedState       │
                  └───────────────────┘                 └────────────────────┘
```

**Components Explained:**

1.  **Component (Interface/Abstract Class):** Defines the common blueprint. Both the base object and decorators will implement this.
2.  **ConcreteComponent:** The original object (e.g., `SimpleCoffee`, `BaseMario`).
3.  **Decorator (Abstract):** The middleman. It implements `Component` AND contains a `Component`. Its `operation()` method usually just delegates to the contained component.
4.  **ConcreteDecorator:** The actual wrappers (e.g., `Milk`, `Sugar`, `Mushroom`). They modify the behaviour before or after calling the parent.

---

## 💻 Implementation 1: Mario Game Power-ups

Let's write a robust implementation for the Mario scenario.

### Step 1: Defines Abstract Component

Sabse pehle contract define karte hain. Sab kuch ek `Character` hai.

```java
// File: MarioGame/Character.java

/**
 * The Component Interface
 * Defines definitions for all concrete characters and decorators.
 */
public abstract class Character {
    
    String description = "Unknown Character";

    public String getDescription() {
        return description;
    }

    // Abstract method that costs or calculates power
    public abstract double getPowerLevel();
    
    // Abstract method to show all current abilities
    public abstract void showAbilities();
}
```

### Step 2: Define Concrete Component (Base Mario)

Ye humara "Naked Object" hai, bina kisi decoration ke.

```java
// File: MarioGame/Mario.java

public class Mario extends Character {
    
    public Mario() {
        description = "Base Mario";
    }

    @Override
    public double getPowerLevel() {
        return 10.0; // Basic power
    }

    @Override
    public void showAbilities() {
        System.out.println("-> Run");
        System.out.println("-> Normal Jump");
    }
}
```

### Step 3: Abstract Decorator Class

Ye magical class hai jo **bridge** ka kaam karti hai.

```java
// File: MarioGame/CharacterDecorator.java

public abstract class CharacterDecorator extends Character {
    
    // The "Has-A" relationship variable
    // We use 'protected' so child decorators can access it
    protected Character character;
    
    // Constructor injection to wrap the object
    public CharacterDecorator(Character character) {
        this.character = character;
    }
    
    // Force decorators to implement description customization
    public abstract String getDescription();
}
```

### Step 4: Concrete Decorators (Power-Ups)

**Decorator 1: Mushroom (Height Up)**

```java
// File: MarioGame/MushroomDecorator.java

public class MushroomDecorator extends CharacterDecorator {
    
    public MushroomDecorator(Character character) {
        super(character); // Wrap the passed character
    }
    
    @Override
    public String getDescription() {
        return character.getDescription() + ", Super Mushroom";
    }

    @Override
    public double getPowerLevel() {
        // Adds 5.0 to whatever the inner character's power was
        return character.getPowerLevel() + 5.0; 
    }
    
    @Override
    public void showAbilities() {
        character.showAbilities(); // Delegate current abilities
        System.out.println("-> Break Bricks (Added by Mushroom)"); // Add new ability
        System.out.println("-> High Jump (Added by Mushroom)");
    }
}
```

**Decorator 2: FireFlower (Shooting)**

```java
// File: MarioGame/FireFlowerDecorator.java

public class FireFlowerDecorator extends CharacterDecorator {
    
    public FireFlowerDecorator(Character character) {
        super(character);
    }
    
    @Override
    public String getDescription() {
        return character.getDescription() + ", Fire Flower";
    }

    @Override
    public double getPowerLevel() {
        return character.getPowerLevel() + 10.0;
    }
    
    @Override
    public void showAbilities() {
        character.showAbilities();
        System.out.println("-> Shoot Fireballs (Added by FireFlower)");
    }
}
```

**Decorator 3: Star (Invincibility)**

```java
// File: MarioGame/StarDecorator.java

public class StarDecorator extends CharacterDecorator {
    
    public StarDecorator(Character character) {
        super(character);
    }
    
    @Override
    public String getDescription() {
        return character.getDescription() + ", Starman";
    }

    @Override
    public double getPowerLevel() {
        return character.getPowerLevel() + 50.0;
    }
    
    @Override
    public void showAbilities() {
        character.showAbilities();
        System.out.println("-> Invincibility (Added by Star)");
    }
}
```

### Step 5: The Client Code

Ab dekhte hain magic kaise happen hota hai runtime pe.

```java
// File: MarioGame/GameEngine.java

public class GameEngine {
    public static void main(String[] args) {
        
        System.out.println("=== GAME START ===");
        
        // 1. Player starts as normal Mario
        Character player = new Mario();
        printCharacterStats(player);
        
        System.out.println("\n--- Player collected a Mushroom ---");
        // Wrap Mario in Mushroom
        player = new MushroomDecorator(player);
        printCharacterStats(player);
        
        System.out.println("\n--- Player collected a Fire Flower ---");
        // Wrap MushroomMario in FireFlower
        player = new FireFlowerDecorator(player);
        printCharacterStats(player);
        
        System.out.println("\n--- Player collected a Star ---");
        // Wrap everything in Star
        player = new StarDecorator(player);
        printCharacterStats(player);
        
        // Example of flexibility:
        // What if we want a Mario that has Double Mushrooms?
        // Inheritance me ye mushkil hota, yahan easy hai:
        System.out.println("\n--- Cheat Mode: Double Mushroom Mario ---");
        Character cheater = new MushroomDecorator(new MushroomDecorator(new Mario()));
        printCharacterStats(cheater);
    }
    
    public static void printCharacterStats(Character p) {
        System.out.println("Description: " + p.getDescription());
        System.out.println("Power Level: " + p.getPowerLevel());
        System.out.println("Abilities:");
        p.showAbilities();
    }
}
```

### Execution Output Analysis

```text
=== GAME START ===
Description: Base Mario
Power Level: 10.0
Abilities:
-> Run
-> Normal Jump

--- Player collected a Mushroom ---
Description: Base Mario, Super Mushroom
Power Level: 15.0  (10 + 5)
Abilities:
-> Run
-> Normal Jump
-> Break Bricks (Added by Mushroom)
-> High Jump (Added by Mushroom)

--- Player collected a Fire Flower ---
Description: Base Mario, Super Mushroom, Fire Flower
Power Level: 25.0 (15 + 10)
Abilities:
-> Run
.. (Previous abilities) ..
-> Shoot Fireballs (Added by FireFlower)

--- Player collected a Star ---
Description: Base Mario, Super Mushroom, Fire Flower, Starman
Power Level: 75.0 (25 + 50)
Abilities:
.. (All Previous) ..
-> Invincibility (Added by Star)
```

Is example se clearly dikhta hai ki kaise **Dynamic Responsibility** add ho rahi hai. Humne `Mario` class ko ek baar bhi modify nahi kiya!

---

## 🍕 Implementation 2: Pizza Topping System

Another classic example is Pizza Cost Calculation. This is very popular in interviews.

**Scenario:**
- Base Pizza types: `Margherita` ($100), `Farmhouse` ($150), `VegDelight` ($120).
- Toppings: `ExtraCheese` ($50), `Mushroom` ($40), `Jalapeno` ($30).

Humein cost calculate karni hai: `Margherita` + `ExtraCheese` + `Mushroom`.

### Code Solution

```java
// 1. Component Interface
abstract class Pizza {
    String description = "Unknown Pizza";
    public String getDescription() { return description; }
    public abstract int getCost();
}

// 2. Concrete Components
class Margherita extends Pizza {
    public Margherita() { description = "Margherita"; }
    public int getCost() { return 100; }
}

class Farmhouse extends Pizza {
    public Farmhouse() { description = "Farmhouse"; }
    public int getCost() { return 150; }
}

// 3. Decorator
abstract class ToppingsDecorator extends Pizza {
    Pizza pizza; // Composition
    public abstract String getDescription();
}

// 4. Concrete Decorators
class ExtraCheese extends ToppingsDecorator {
    public ExtraCheese(Pizza pizza) { this.pizza = pizza; }
    
    public String getDescription() {
        return pizza.getDescription() + ", Extra Cheese";
    }
    
    public int getCost() {
        return 50 + pizza.getCost(); // 50 is cost of cheese
    }
}

class Mushroom extends ToppingsDecorator {
    public Mushroom(Pizza pizza) { this.pizza = pizza; }
    
    public String getDescription() {
        return pizza.getDescription() + ", Mushroom";
    }
    
    public int getCost() {
        return 40 + pizza.getCost();
    }
}

// Usage
public class DominozStore {
    public static void main(String[] args) {
        // Order: Farmhouse with Double Cheese
        Pizza order = new Farmhouse();
        order = new ExtraCheese(order);
        order = new ExtraCheese(order); // Double cheese!
        
        System.out.println("Order: " + order.getDescription());
        System.out.println("Cost: ₹" + order.getCost());
    }
}
```

**Output:**
```
Order: Farmhouse, Extra Cheese, Extra Cheese
Cost: ₹250  (150 + 50 + 50)
```

Ye implementation dikhati hai ki kaise `Math` operations (cost calculation) bhi decorator pattern se easily handle hote hain.

---

## 🌍 Real-World Deep Dive: Java I/O Streams

Interviewers often ask: "Tell me a real-world example of Decorator Pattern in Java Libraries". The answer is **Java I/O**.

### Understanding java.io Package

Jab aap Java mein file read karte hain, aapne aksar ye syntax dekha hoga:

```java
InputStream in = new BufferedInputStream(new FileInputStream("test.txt"));
```

Ye kya hai? Ye **Decorator Pattern** hai!

1.  **Component:** `InputStream` (Abstract class)
2.  **ConcreteComponent:** `FileInputStream`, `ByteArrayInputStream`
3.  **Decorator:** `FilterInputStream`
4.  **ConcreteDecorators:** `BufferedInputStream`, `DataInputStream`, `GZIPInputStream`, `ObjectInputStream`

**Visualization:**

```java
new LineNumberInputStream(
    new BufferedInputStream(
        new FileInputStream("data.txt")
    )
);
```

- `FileInputStream` basics bytes read karta hai disk se.
- `BufferedInputStream` us stream ko **buffer** karta hai taaki performance fast ho.
- `LineNumberInputStream` us basic stream mein line counting ki **extra capability** add karta hai.

Dekha? Features layer by layer add ho rahe hain.

### Creating a Custom I/O Decorator

Chalo hum apna khud ka Java I/O Decorator banate hain jo sabhi letters ko **LowerCase** mein convert kar de read karte waqt.

```java
import java.io.*;

// Extending FilterInputStream (The Abstract Decorator in Java)
public class LowerCaseInputStream extends FilterInputStream {

    public LowerCaseInputStream(InputStream in) {
        super(in);
    }

    // Byte-by-byte read override
    @Override
    public int read() throws IOException {
        int c = super.read(); // Get original byte
        return (c == -1 ? c : Character.toLowerCase((char)c)); // Enhance it
    }

    // Buffer read override
    @Override
    public int read(byte[] b, int offset, int len) throws IOException {
        int result = super.read(b, offset, len);
        for (int i = offset; i < offset + result; i++) {
            b[i] = (byte)Character.toLowerCase((char)b[i]);
        }
        return result;
    }
}

// Testing it
public class InputTest {
    public static void main(String[] args) throws IOException {
        String content = "HELLO CODER ARMY";
        InputStream stream = new ByteArrayInputStream(content.getBytes());
        
        // Wrap our custom decorator
        InputStream lowerStream = new LowerCaseInputStream(stream);
        
        int c;
        while((c = lowerStream.read()) != -1) {
            System.out.print((char)c);
        }
        // Output: hello coder army
    }
}
```

Ye example show karta hai ki Decorator pattern kitna powerful hai. Humne standard Java library ko bina change kiye uske upar naya behavior add kar diya!

---

## ⚖️ Difference Between Patterns

Interview mein aksar confusions hoti hain in patterns ke beech.

### Decorator vs Proxy

| Decorator | Proxy |
| :--- | :--- |
| **Intent:** functionality  adds/enhance karna. | **Intent:** access control karna (lazy loading, security). |
| Client ko generally pata hota hai ki decoration ho rahi hai. | Client ko pata nahi hota ki wo proxy se baat kar raha hai. |
| Object creation client ke haath mein hota hai mostly. | Proxy often manages lifecycle of the real object internally. |
| **Relationship:** Smart Wrapper. | **Relationship:** Surrogate / Substitute. |

### Decorator vs Adapter

| Decorator | Adapter |
| :--- | :--- |
| Interface change **nahi** karta. Same interface rakhta hai. | Interface ko **change** karta hai taaki incompatible classes saath kaam kar sakein. |
| Responsibilities add karta hai. | Interface convert karta hai. |
| **Analogy:** Phone pe fancy cover lagana. | **Analogy:** American socket mein European plug lagana. |

### Decorator vs Strategy

| Decorator | Strategy |
| :--- | :--- |
| Object ka "Skin" change karta hai (External). | Object ki "Guts" change karta hai (Internal). |
| Recursive composition (Layers). | One at a time selection (Switching). |
| "Add-on" nature ka hota hai. | "Replacement" nature ka hota hai. |

---

## ✅ Advantages & Disadvantages

### Advantages (Pros)
1.  **Flexibility:** Inheritance se zyada flexible hai. Runtime pe capabilities add/remove kar sakte hain.
2.  **No Class Explosion:** Avoids creating `MarioWithGunAndStarAndFlower` type classes.
3.  **Single Responsibility Principle:** Har decorator sirf ek specific feature (like `Flying`) pe focus karta hai.
4.  **Runtime Configuration:** Objects ko on-the-fly configure kiya ja sakta hai config files ya user input se.

### Disadvantages (Cons)
1.  **Many Small Objects:** System mein bahot saare chote-chote objects ban jaate hain jo debugger mein trace karna mushkil kar dete hain. (Imagine debugging 10 layers wrapped).
2.  **Order Matters:** Kabhi kabhi decorators ka order matter karta hai. E.g., `Encrypt` then `Compress` vs `Compress` then `Encrypt` can give different results. Decorator pattern should ideally be independent of order, but practically it isn't always.
3.  **Complexity:** Initial code setup (Interfaces, Abstract Classes) thoda verbose lag sakta hai beginners ko.

---

## 🙋‍♂️ Interview Questions (Q&A)

**Q1: Kya hum ek object par multiple baar same decorator laga sakte hain?**
**Ans:** Haan, bilkul! Jaise humne Pizza example mein dekha, hum `Double Cheese` ke liye `ExtraCheese` decorator do baar wrap kar sakte hain. Inheritance mein ye karna namumkin ke barabar hota hai.

**Q2: Decorator Pattern mein Abstract Decorator class banani zaroori hai kya?**
**Ans:** Technically nahi, agar aapke paas sirf ek hi Concrete Decorator hai toh aap seedha implement kar sakte hain. Lekin Best Practice ye hai ki Abstract Decorator banayein taaki Code Duplication avoid ho (reference holding aur basic delegation logic parent mein reh sake).

**Q3: Java mein `final` classes ko decorate kar sakte hain?**
**Ans:** Decorator pattern works by implementing an Interface or extending a Class. Agar class `final` hai, aap usse extend nahi kar sakte. Is case mein aapko Interface based approach use karni padegi. Agar Interface bhi nahi hai, toh Decorator pattern use karna mushkil hai.

**Q4: Constructor mein itne saare `new` keywords (`new A(new B(new C()))`) gande lagte hain. Iska solution kya hai?**
**Ans:** Is problem ko solve karne ke liye **Factory Pattern** ya **Builder Pattern** ka use kiya jaata hai. Builder pattern allows clean creation syntax like `.withCheese().withMushroom().build()`.

**Q5: Decorator remove kaise karein runtime pe?**
**Ans:** Decorator pattern removal ke liye optimize nahi hai. Agar aapko `Stack` ke beech mein se ek layer hatani hai, toh aapko poora object graph dobara construct karna padega us layer ko skip karke. Ye thoda complex operation hai.

**Q6: Kaunsa SOLID principle Decorator follow karta hai?**
**Ans:** Mainly **Open/Closed Principle** (Add functionality without modifying code) aur **Single Responsibility Principle** (Each decorator does one thing). It also relies heavily on **Dependency Inversion** (Depend on abstractions, not concretions).

**Q7: Decorator pattern kab use nahi karna chahiye?**
**Ans:** 
- Jab object hierarchy already simple ho aur future mein changes expect na ho.
- Jab component ki identity important ho (kyunki decorator wrapper asli object ki identity chupa leta hai).
- Jab decorators ka order logic ko break kar sakta ho significant tareeke se.

---

## 🏁 Conclusion

Decorator pattern ek bahut powerful tool hai jo aapko **loose coupling** aur **high cohesion** achieve karne mein madad karta hai. 

Start mein ye complex lag sakta hai, par jab aap `java.io` package ya UI frameworks (Swing/React HOCs) use karte hain, toh aap realise karte hain ki ye pattern har jagah hai.

Mario game ho, Coffee shop ho, ya File handling system – jahan bhi **"Add-ons"** ya **"Extensions"** ka concept aaye, wahan **Decorator Pattern** aapka best friend hai.

**Code karte raho, seekhte raho! Happy Coding Coder Army!** 🚀
