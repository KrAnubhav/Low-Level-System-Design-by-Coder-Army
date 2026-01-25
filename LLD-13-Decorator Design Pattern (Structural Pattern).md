# LLD-13: Decorator Design Pattern (Structural Pattern) 🎨

## Introduction

**Welcome Back Coder Army!**

Aaj ke lecture mein hum ek naya aur interesting design pattern seekhenge jiska naam hai **Decorator Design Pattern**. Ye pattern aapko zaroor kuch naya sikhayega aur aapki life ko as a developer better banayega.

---

## The Goal: Enhancing Responsibilities Dynamically

Humara aim kya hai is pattern ke saath?

Imagine humare paas ek object hai `Obj1`.
- Main chahta hoon ki main is object ko **additional responsibilities** (functionalities) provide kar sakun **runtime** pe.
- Compile time changes nahi, balki code run hote waqt hum behaviour change kar sakein.

### Example: "I did something"
Let's say `Obj1` ka ek method hai `doSomething()`.
- Default behavior: Returns "I did something".
- **Requirement:** Main chahta hoon ki dynamically iska output change ho jaye, jaise "I did something GREAT".

---

## Why Inheritance Fails? ❌

Sabse pehla thought aata hai **Inheritance** use karne ka. Hum base class ke methods ko child class mein override kar sakte hain.

### Scenario:
1. **Base Class:** `doSomething()` returns "Running".
2. **Child Class:** Overrides `doSomething()` to return "Running with heavy shoes".

**Problem:**
- Main **runtime** pe kaise decide karun ki kaunsa behavior chahiye?
- Main `Base b = new Child()` kar sakta hoon, to child ka method call hoga.
- Lekin agar mujhe multiple combinations chahiye, toh mujhe `if-else` lagana padega.

Isse badi problem tab aati hai **Class Explosion** ki, jo hum Mario game ke example se samjhenge.

> **Remmeber:** Inheritance is considered "bad" in many cases because it creates tight coupling and class explosion. Always favor **Composition over Inheritance**.

---

## Real World Example: Mario Game Power-ups 🍄⭐

Yaad hai bachpan mein hum **Mario** khelte the?

**Original State:**
- Mario chhota hota hai.

**Power-ups:**
1. **Mushroom:** Mario ki height badh jati hai (HeightUp).
2. **Flower:** Mario fireballs shoot kar sakta hai (GunPower).
3. **Star:** Mario invincible ho jata hai aur fast bhagta hai (StarPower).

### Attempts with Inheritance (The Wrong Way)

Agar hum inheritance use karein, toh humein har combination ke liye ek class banani padegi:

1. `Mario` (Base)
2. `MarioWithHeight`
3. `MarioWithGun`
4. `MarioWithStar`
5. `MarioWithHeightAndGun`
6. `MarioWithHeightAndStar`
7. `MarioWithGunAndStar`
8. `MarioWithHeightAndGunAndStar`

**Problem - Class Explosion:** 💥
- Jaise hi ek naya power-up add hoga (e.g., `FlyingAbility`), classes ki count exponentially badh jayegi.
- `MarioWithFlying`, `MarioWithFlyingAndGun`, etc.
- Isse manage karna impossible ho jayega.

---

## Solution: Decorator Pattern ✅

Decorator pattern kehta hai:
> "Ek object ko wrap (decorate) kar do dusre object ke saath."

### Concept
1. **Base Object:** `Obj1` (Returns "I did something")
2. **Decorator Object:** `Dec1` adds functionality.

**Flow:**
- Client calls `Dec1.doSomething()`
- `Dec1` calls `Obj1.doSomething()` -> gets "I did something"
- `Dec1` appends " AMAZINGLY" -> returns "I did something AMAZINGLY"

Hum in decorators ko **Stack (Chain)** kar sakte hain:
`Decorator2` wraps `Decorator1` wraps `Obj1`.

### The "Is-A" and "Has-A" Relationship

Ye pattern bahut unique hai kyunki ye **Inheritance (`Is-A`)** aur **Composition (`Has-A`)** dono use karta hai:

1. **Is-A (Inheritance):**
   - Decorator **IS A** Component (Base Type).
   - Kyun? Taaki hum Decorator ko wahan use kar sakein jahan Base object expected hai. Polymorphism ke liye.
   
2. **Has-A (Composition):**
   - Decorator **HAS A** Component.
   - Kyun? Taaki wo original object (ya dusre decorator) ka method call kar sake aur result ko enhance kar sake.

---

## Design & Implementation (Mario System) 🛠️

Chalo Mario system ko design karte hain.

### 1. Abstract Component (Character)
Sabse upar ek abstract class ya interface hoga jo `Character` ko represent karega.

```java
public abstract class BaseCharacter {
    public abstract String getAbilities();
}
```

### 2. Concrete Component (Mario)
Ye humara base object hai.

```java
public class Mario extends BaseCharacter {
    @Override
    public String getAbilities() {
        return "Mario";
    }
}
```

### 3. Abstract Decorator
Ye key part hai. Ye `BaseCharacter` ko extend bhi karega aur usko hold bhi karega.

```java
public abstract class CharacterDecorator extends BaseCharacter {
    protected BaseCharacter character; // HAS-A Relationship
    
    public CharacterDecorator(BaseCharacter character) {
        this.character = character;
    }
    
    @Override
    public String getAbilities() {
        return character.getAbilities(); // Delegate to wrapped object
    }
}
```

### 4. Concrete Decorators
Ab hum specific power-ups banayenge.

**HeightUp (Mushroom):**
```java
public class HeightUpDecorator extends CharacterDecorator {
    public HeightUpDecorator(BaseCharacter character) {
        super(character);
    }
    
    @Override
    public String getAbilities() {
        // Original ability + New ability
        return character.getAbilities() + ", Height Up";
    }
}
```

**GunPower (Flower):**
```java
public class GunPowerDecorator extends CharacterDecorator {
    public GunPowerDecorator(BaseCharacter character) {
        super(character);
    }
    
    @Override
    public String getAbilities() {
        return character.getAbilities() + ", Gun Power";
    }
}
```

**StarPower (Star):**
```java
public class StarPowerDecorator extends CharacterDecorator {
    public StarPowerDecorator(BaseCharacter character) {
        super(character);
    }
    
    @Override
    public String getAbilities() {
        return character.getAbilities() + ", Star Power (Limited Time)";
    }
}
```

---

## How it Works (Visualizing the Chain) 🔗

Jab hum ye code likhte hain:

```java
BaseCharacter player = new StarPowerDecorator(
                            new GunPowerDecorator(
                                new HeightUpDecorator(
                                    new Mario()
                                )
                            )
                        );
```

**Execution Flow:**
1. **StarPower** calls `getAbilities()` on **GunPower**.
2. **GunPower** calls `getAbilities()` on **HeightUp**.
3. **HeightUp** calls `getAbilities()` on **Mario**.
4. **Mario** returns "Mario".
5. **HeightUp** adds ", Height Up" -> returns "Mario, Height Up".
6. **GunPower** adds ", Gun Power" -> returns "Mario, Height Up, Gun Power".
7. **StarPower** adds ", Star Power" -> Final Result!

Ye basically **Recursion** jaisa hai! Hum andar deep jaate hain base object tak aur fir wapis aate hue strings add karte hain.

---

## Standard UML Diagram 📊

```
      ┌────────────────┐
      │  Component     │
      │ (BaseCharacter)│
      └───────┬────────┘
              │
      ┌───────┴───────────────┐
      │                       │
┌─────▼───────┐        ┌──────▼────────────┐
│ConcreteComp │        │    Decorator      │◄───────┐
│  (Mario)    │        └──────┬────────────┘        │
└─────────────┘               │ triangle (is-a)     │ diamond (has-a)
                              ▼                     │
                      ┌────────────────┐            │
                      │ConcreteDeco A  │────────────┘
                      │(HeightUp)      │
                      └────────────────┘
```

**Components:**
1. **Component:** Defines the interface.
2. **ConcreteComponent:** Object to be decorated.
3. **Decorator:** Maintains reference to Component and conforms to Component interface.
4. **ConcreteDecorator:** Adds specific responsibilities.

---

## Complete Java Code 💻

### File: `MarioGame.java`

```java
// 1. Component Interface / Abstract Class
abstract class BaseCharacter {
    public abstract String getAbilities();
}

// 2. Concrete Component
class Mario extends BaseCharacter {
    @Override
    public String getAbilities() {
        return "Mario"; // Base ability
    }
}

// 3. Decorator Class
abstract class CharacterDecorator extends BaseCharacter {
    protected BaseCharacter character;
    
    public CharacterDecorator(BaseCharacter character) {
        this.character = character;
    }
}

// 4. Concrete Decorators
class HeightUpDecorator extends CharacterDecorator {
    public HeightUpDecorator(BaseCharacter character) {
        super(character);
    }
    
    @Override
    public String getAbilities() {
        return character.getAbilities() + ", Height Up";
    }
}

class GunPowerDecorator extends CharacterDecorator {
    public GunPowerDecorator(BaseCharacter character) {
        super(character);
    }
    
    @Override
    public String getAbilities() {
         return character.getAbilities() + ", Gun Power";
    }
}

class StarPowerDecorator extends CharacterDecorator {
    public StarPowerDecorator(BaseCharacter character) {
        super(character);
    }
    
    @Override
    public String getAbilities() {
         return character.getAbilities() + ", Star Power (Limited Time)";
    }
}

// 5. Client Code
public class MarioGame {
    public static void main(String[] args) {
        System.out.println("--- Level 1 Start ---");
        BaseCharacter player = new Mario();
        System.out.println(player.getAbilities());
        
        System.out.println("\n--- Ate Mushroom ---");
        player = new HeightUpDecorator(player);
        System.out.println(player.getAbilities());
        
        System.out.println("\n--- Got Flower ---");
        player = new GunPowerDecorator(player);
        System.out.println(player.getAbilities());
        
        System.out.println("\n--- Got Star ---");
        player = new StarPowerDecorator(player);
        System.out.println(player.getAbilities());
        
        // Custom Combination
        System.out.println("\n--- New Game: Flying Gun Mario ---");
        BaseCharacter weirdMario = new GunPowerDecorator(
                                    new HeightUpDecorator(
                                        new Mario()
                                    ));
        System.out.println(weirdMario.getAbilities());
    }
}
```

**Output:**
```text
--- Level 1 Start ---
Mario

--- Ate Mushroom ---
Mario, Height Up

--- Got Flower ---
Mario, Height Up, Gun Power

--- Got Star ---
Mario, Height Up, Gun Power, Star Power (Limited Time)

--- New Game: Flying Gun Mario ---
Mario, Height Up, Gun Power
```

---

## Other Real-World Use Cases 🌍

1. **Text Editor (like Google Docs):**
   - Base: `Text`
   - Decorators: `BoldDecorator`, `ItalicDecorator`, `UnderlineDecorator`.
   - Usage: `new Italic(new Bold(new Text("Hello")))`.

2. **Pizza Customization:**
   - Base: `Pizza`
   - Decorators: `CheeseDecorator`, `TomatoDecorator`, `JalapenoDecorator` (Calculating cost).

3. **Data Stream I/O (Java I/O):**
   - Java ka I/O package pure Decorator pattern hai!
   - `new BufferedReader(new FileReader(new File("...")))`
   - `BufferedReader` decorates `FileReader`, which reads from `File`.

4. **Web Request Validation/Processing:**
   - Base: `HttpRequest`
   - Decorators: `LoggingDecorator`, `AuthDecorator`, `CompressionDecorator`.
   - Request pehle Log hoti hai, fir Auth check hota hai, fir Compress hoti hai.

---

## Official Definition 📖

> **"The Decorator Pattern attaches additional responsibilities to an object dynamically. Decorators provide a flexible alternative to subclassing for extending functionality."**

**Key Takeaways:**
- **Dynamic:** Runtime pe behavior change.
- **Flexible:** Inheritance ke rigid structure se aazadi.
- **Recursive:** Object wrapping object wrapping object.

---

## Interview Questions (Q&A) ❓

**Q1: Inheritance vs Decorator Pattern?**
- **Inheritance:** Static (Compile time). Behaviour change karne ke liye nayi class banani padti hai.
- **Decorator:** Dynamic (Runtime). Object composition use karta hai. Nayi functionality existing object mein inject ho sakti hai bina class badle.

**Q2: Decorator Pattern mein Abstract Decorator class kyu zaroori hai?**
- Taaki ye interface (`Is-A`) maintain kar sake Base Component ke saath, aur ek common reference hold kar sake wrapped object ka (`Has-A`). Isse code duplication bachta hai (sab decorators mein `character` variable nahi likhna padta).

**Q3: Kahan use karein?**
- Jab functionality "layer by layer" add karni ho.
- Jab class extension (inheritance) possible na ho (final classes) ya practical na ho (class explosion).
- Examples: Java I/O Streams, UI libraries (Scrollbar around Window), Pizza toppings.

**Q4: Java IO aur Decorator Pattern?**
- `InputStream` (Abstract Component)
- `FileInputStream` (Concrete Component)
- `BufferedInputStream` (Decorator - Adds buffering)
- `GZipInputStream` (Decorator - Adds compression)
- Hum likhte hain: `new GZipInputStream(new BufferedInputStream(new FileInputStream(...)))`.

---

Happy Coding! 🚀
