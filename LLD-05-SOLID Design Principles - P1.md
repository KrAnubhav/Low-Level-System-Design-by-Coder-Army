---
title: "LLD-05: SOLID Design Principles - Part 1"
date: 2026-01-23
tags: [lld, solid, design-principles, srp, ocp, lsp, java]
---

# LLD-05: SOLID Design Principles - Part 1 🏗️

## 📋 Table of Contents
1. [Introduction](#introduction)
2. [Single Responsibility Principle (SRP)](#single-responsibility-principle-srp)
3. [Open/Closed Principle (OCP)](#openclosed-principle-ocp)
4. [Liskov Substitution Principle (LSP)](#liskov-substitution-principle-lsp)

---

## Introduction

### What We've Learned So Far
- **OOP Pillars**: Abstraction, Encapsulation, Inheritance, Polymorphism
- **Class & Object Creation**: How to create classes and objects
- **Data Hiding**: How to maintain data security

### The Real-World Challenge 🎯

In real-world projects, we deal with **thousands of classes**. Managing these classes becomes extremely difficult if we don't follow proper design principles.

**Problems without Design Principles:**
1. **Maintainability Issues** 🔧
   - Difficult to add new features
   - Requires extensive code changes
   - Introduces new bugs easily

2. **Readability Issues** 📖
   - Hard for new engineers to understand the codebase
   - High learning curve
   - Time-consuming code comprehension

3. **Debugging Nightmares** 🐛
   - Bugs are difficult to identify
   - Time-consuming debugging process
   - Tightly coupled code makes fixes risky

### Real-World Analogy: The Wiring Problem 🏠

Imagine a house where:
- Electrical wiring
- Internet cables
- Water pipes

All pass through the **same location** in a messy tangle.

**Problem**: If any wire has a fault, it's extremely difficult to:
- Identify which wire is faulty
- Replace the specific wire
- Avoid affecting other systems

**Same in Programming**: If classes are tightly coupled and messy, fixing bugs or adding features becomes a disaster.

---

## SOLID Design Principles Overview

**SOLID** is an acronym introduced by **Robert C. Martin** in 2000.

```
S - Single Responsibility Principle (SRP)
O - Open/Closed Principle (OCP)
L - Liskov Substitution Principle (LSP)
I - Interface Segregation Principle (ISP)
D - Dependency Inversion Principle (DIP)
```

These principles help us write:
✅ Clean code  
✅ Maintainable architecture  
✅ Scalable applications  

---

## Single Responsibility Principle (SRP)

### Definition 📝

> **"A class should have only ONE reason to change"**  
> **"A class should do only ONE thing"**

### Key Concept

- One class = One responsibility
- All attributes and methods in a class should serve that single responsibility
- Don't let a class handle multiple responsibilities

### Real-World Analogy: TV Remote 📺

A TV remote's job is to **control the TV only**.

❌ **Bad Design**: If the same remote also controls:
- Refrigerator
- Air Conditioner
- Washing Machine

It becomes:
- Too complicated
- Hard to maintain
- Difficult to debug

✅ **Good Design**: Each remote controls only one device.

---

### Example: E-Commerce Shopping Cart

#### ❌ Problem: Violating SRP

**Class Diagram:**

```
┌─────────────────┐
│    Product      │
├─────────────────┤
│ - name: String  │
│ - price: double │
└─────────────────┘
         △
         │ has-a (1..*)
         │
┌──────────────────────────────┐
│      ShoppingCart            │
├──────────────────────────────┤
│ - products: List<Product>    │
├──────────────────────────────┤
│ + calculateTotalPrice()      │
│ + printInvoice()             │ ← Multiple responsibilities!
│ + saveToDatabase()           │ ← Multiple responsibilities!
└──────────────────────────────┘
```

**Why is this BAD?**

The `ShoppingCart` class has **THREE responsibilities**:
1. Calculate total price
2. Print invoice
3. Save to database

**Problems:**
- If database logic changes → modify `ShoppingCart`
- If invoice printing logic changes → modify `ShoppingCart`
- If price calculation logic changes → modify `ShoppingCart`

Multiple reasons to change = **SRP Violation**

#### ✅ Solution: Following SRP

**Class Diagram:**

```
┌─────────────────┐
│    Product      │
├─────────────────┤
│ - name: String  │
│ - price: double │
└─────────────────┘
         △
         │ has-a (1..*)
         │
┌──────────────────────────────┐
│      ShoppingCart            │
├──────────────────────────────┤
│ - products: List<Product>    │
├──────────────────────────────┤
│ + calculateTotalPrice()      │ ← Single responsibility
└──────────────────────────────┘
         △                △
         │                │
         │ has-a          │ has-a
         │                │
┌────────────────────┐  ┌────────────────────┐
│ CartInvoicePrinter │  │  CartDBStorage     │
├────────────────────┤  ├────────────────────┤
│ - sc: ShoppingCart │  │ - sc: ShoppingCart │
├────────────────────┤  ├────────────────────┤
│ + printInvoice()   │  │ + saveToDatabase() │
└────────────────────┘  └────────────────────┘
```

**Benefits:**
- Each class has a **single responsibility**
- Changes to database logic → only modify `CartDBStorage`
- Changes to invoice printing → only modify `CartInvoicePrinter`
- Changes to price calculation → only modify `ShoppingCart`

---

### Java Implementation

#### ❌ Violating SRP

```java
class Product {
    private String name;
    private double price;
    
    public Product(String name, double price) {
        this.name = name;
        this.price = price;
    }
    
    public double getPrice() {
        return price;
    }
    
    public String getName() {
        return name;
    }
}

class ShoppingCart {
    private List<Product> products = new ArrayList<>();
    
    public void addProduct(Product product) {
        products.add(product);
    }
    
    public List<Product> getProducts() {
        return products;
    }
    
    // Responsibility 1: Calculate total price
    public double calculateTotalPrice() {
        double total = 0;
        for (Product product : products) {
            total += product.getPrice();
        }
        return total;
    }
    
    // Responsibility 2: Print invoice
    public void printInvoice() {
        System.out.println("===== INVOICE =====");
        for (Product product : products) {
            System.out.println(product.getName() + ": $" + product.getPrice());
        }
        System.out.println("Total: $" + calculateTotalPrice());
    }
    
    // Responsibility 3: Save to database
    public void saveToDatabase() {
        System.out.println("Saving shopping cart to database...");
        // Database logic here
    }
}

// Usage
public class Main {
    public static void main(String[] args) {
        ShoppingCart cart = new ShoppingCart();
        cart.addProduct(new Product("Laptop", 1500));
        cart.addProduct(new Product("Mouse", 50));
        
        cart.printInvoice();
        cart.saveToDatabase();
    }
}
```

#### ✅ Following SRP

```java
class Product {
    private String name;
    private double price;
    
    public Product(String name, double price) {
        this.name = name;
        this.price = price;
    }
    
    public double getPrice() {
        return price;
    }
    
    public String getName() {
        return name;
    }
}

class ShoppingCart {
    private List<Product> products = new ArrayList<>();
    
    public void addProduct(Product product) {
        products.add(product);
    }
    
    public List<Product> getProducts() {
        return products;
    }
    
    // Single responsibility: Calculate total price
    public double calculateTotalPrice() {
        double total = 0;
        for (Product product : products) {
            total += product.getPrice();
        }
        return total;
    }
}

// Separate class for invoice printing
class CartInvoicePrinter {
    private ShoppingCart cart;
    
    public CartInvoicePrinter(ShoppingCart cart) {
        this.cart = cart;
    }
    
    public void printInvoice() {
        System.out.println("===== INVOICE =====");
        for (Product product : cart.getProducts()) {
            System.out.println(product.getName() + ": $" + product.getPrice());
        }
        System.out.println("Total: $" + cart.calculateTotalPrice());
    }
}

// Separate class for database operations
class CartDBStorage {
    private ShoppingCart cart;
    
    public CartDBStorage(ShoppingCart cart) {
        this.cart = cart;
    }
    
    public void saveToDatabase() {
        System.out.println("Saving shopping cart to database...");
        // Database logic here
    }
}

// Usage
public class Main {
    public static void main(String[] args) {
        ShoppingCart cart = new ShoppingCart();
        cart.addProduct(new Product("Laptop", 1500));
        cart.addProduct(new Product("Mouse", 50));
        
        CartInvoicePrinter printer = new CartInvoicePrinter(cart);
        printer.printInvoice();
        
        CartDBStorage storage = new CartDBStorage(cart);
        storage.saveToDatabase();
    }
}
```

---

### Important Clarification ⚠️

**SRP does NOT mean "one method per class"**

A class can have **multiple methods**, but all methods should serve the **same responsibility**.

**Example:**
```java
class ShoppingCart {
    // All these methods serve the SAME responsibility: managing cart price
    public double calculateTotalPrice() { }
    public double calculateDiscount() { }
    public double calculateTax() { }
    public double getFinalPrice() { }
}
```

This is **still following SRP** because all methods relate to **price calculation**.

---

## Open/Closed Principle (OCP)

### Definition 📝

> **"A class should be OPEN for extension but CLOSED for modification"**

### Key Concept

- **Open for Extension**: You can add new features
- **Closed for Modification**: Don't change existing code

**Translation**: When adding new features, create new classes instead of modifying existing ones.

---

### Why is this Important?

Modifying existing code can:
- Introduce new bugs
- Break existing functionality
- Require extensive testing

Creating new classes:
- Keeps old code stable
- Reduces risk
- Easier to maintain

---

### Example: Database Persistence

#### ❌ Problem: Violating OCP

**Initial Design:**

```
┌─────────────────┐
│    Product      │
├─────────────────┤
│ - name: String  │
│ - price: double │
└─────────────────┘
         △
         │ has-a (1..*)
         │
┌──────────────────────────────┐
│      ShoppingCart            │
├──────────────────────────────┤
│ - products: List<Product>    │
├──────────────────────────────┤
│ + calculateTotalPrice()      │
└──────────────────────────────┘
         △
         │ has-a
         │
┌──────────────────────────────┐
│      DBStorage               │
├──────────────────────────────┤
│ - cart: ShoppingCart         │
├──────────────────────────────┤
│ + saveToDatabase()           │
│ + saveToMongo()              │ ← Adding new methods
│ + saveToFile()               │ ← Modifying existing class!
└──────────────────────────────┘
```

**Problem**: Every time we need a new storage type, we modify the `DBStorage` class.

This **violates OCP** because we're **modifying existing code**.

#### ✅ Solution: Following OCP

**Using Abstraction + Inheritance + Polymorphism:**

```
┌─────────────────┐
│    Product      │
├─────────────────┤
│ - name: String  │
│ - price: double │
└─────────────────┘
         △
         │ has-a (1..*)
         │
┌──────────────────────────────┐
│      ShoppingCart            │
├──────────────────────────────┤
│ - products: List<Product>    │
├──────────────────────────────┤
│ + calculateTotalPrice()      │
└──────────────────────────────┘
         △
         │ has-a
         │
┌──────────────────────────────┐
│   <<abstract>>               │
│   DBPersistence              │
├──────────────────────────────┤
│ - cart: ShoppingCart         │
├──────────────────────────────┤
│ + save() {abstract}          │
└──────────────────────────────┘
         △
         │
    ┌────┴────┬────────────┐
    │         │            │
┌───────────┐ ┌──────────┐ ┌──────────┐
│SQLStorage │ │MongoStore│ │FileStore │
├───────────┤ ├──────────┤ ├──────────┤
│+ save()   │ │+ save()  │ │+ save()  │
└───────────┘ └──────────┘ └──────────┘
```

**Benefits:**
- Adding new storage type = Create new class
- No modification to existing classes
- Follows OCP perfectly

---

### Java Implementation

#### ❌ Violating OCP

```java
class ShoppingCart {
    private List<Product> products = new ArrayList<>();
    
    public void addProduct(Product product) {
        products.add(product);
    }
    
    public List<Product> getProducts() {
        return products;
    }
    
    public double calculateTotalPrice() {
        double total = 0;
        for (Product product : products) {
            total += product.getPrice();
        }
        return total;
    }
}

class DBStorage {
    private ShoppingCart cart;
    
    public DBStorage(ShoppingCart cart) {
        this.cart = cart;
    }
    
    public void saveToSQL() {
        System.out.println("Saving to SQL database...");
    }
    
    // New feature: Save to MongoDB
    public void saveToMongo() {
        System.out.println("Saving to MongoDB...");
    }
    
    // New feature: Save to File
    public void saveToFile() {
        System.out.println("Saving to file...");
    }
}

// Usage
public class Main {
    public static void main(String[] args) {
        ShoppingCart cart = new ShoppingCart();
        cart.addProduct(new Product("Laptop", 1500));
        
        DBStorage storage = new DBStorage(cart);
        storage.saveToSQL();
        storage.saveToMongo();  // Modified existing class!
        storage.saveToFile();   // Modified existing class!
    }
}
```

#### ✅ Following OCP

```java
// Abstract class (Interface in Java terms)
abstract class DBPersistence {
    protected ShoppingCart cart;
    
    public DBPersistence(ShoppingCart cart) {
        this.cart = cart;
    }
    
    public abstract void save();
}

// Concrete implementation for SQL
class SQLPersistence extends DBPersistence {
    public SQLPersistence(ShoppingCart cart) {
        super(cart);
    }
    
    @Override
    public void save() {
        System.out.println("Saving shopping cart to SQL database...");
        // SQL-specific logic
    }
}

// Concrete implementation for MongoDB
class MongoPersistence extends DBPersistence {
    public MongoPersistence(ShoppingCart cart) {
        super(cart);
    }
    
    @Override
    public void save() {
        System.out.println("Saving shopping cart to MongoDB...");
        // MongoDB-specific logic
    }
}

// Concrete implementation for File
class FilePersistence extends DBPersistence {
    public FilePersistence(ShoppingCart cart) {
        super(cart);
    }
    
    @Override
    public void save() {
        System.out.println("Saving shopping cart to file...");
        // File I/O logic
    }
}

// Usage with Polymorphism
public class Main {
    public static void main(String[] args) {
        ShoppingCart cart = new ShoppingCart();
        cart.addProduct(new Product("Laptop", 1500));
        
        // Using polymorphism - same method, different behavior
        DBPersistence db1 = new SQLPersistence(cart);
        db1.save();  // Saves to SQL
        
        DBPersistence db2 = new MongoPersistence(cart);
        db2.save();  // Saves to MongoDB
        
        DBPersistence db3 = new FilePersistence(cart);
        db3.save();  // Saves to File
        
        // Adding new storage type? Just create a new class!
        // No need to modify existing code
    }
}
```

---

### Understanding Interfaces & Abstraction 🎯

**What is an Interface?**

An interface is a **contract** between a class and a client.

```
┌──────────┐         ┌────────────┐         ┌───────┐
│  Client  │ ◄────── │ Interface  │ ◄────── │ Class │
└──────────┘         └────────────┘         └───────┘
                     (Contract)
```

**Interface tells:**
- **WHAT** the class can do
- **NOT HOW** it does it

**Example:**
```java
// Interface declares methods (WHAT)
interface DBPersistence {
    void save();  // What it does, not how
}

// Class defines implementation (HOW)
class SQLPersistence implements DBPersistence {
    public void save() {
        // HOW to save to SQL
    }
}
```

---

### Polymorphism in Action 🎭

```java
abstract class A {
    public abstract void m1();
}

class A1 extends A {
    public void m1() { System.out.println("A1's m1"); }
}

class A2 extends A {
    public void m1() { System.out.println("A2's m1"); }
}

class A3 extends A {
    public void m1() { System.out.println("A3's m1"); }
}

// Polymorphism
A obj;

obj = new A1();
obj.m1();  // Prints: A1's m1

obj = new A2();
obj.m1();  // Prints: A2's m1

obj = new A3();
obj.m1();  // Prints: A3's m1
```

**Same variable, different behavior** = **Polymorphism**

---

## Liskov Substitution Principle (LSP)

### Definition 📝

> **"Subclasses should be substitutable for their base classes"**

### Key Concept

If you have:
- Class `A` (parent)
- Class `B` (child, extends A)

Then:
- Anywhere you use `A`, you should be able to use `B` without breaking the code
- Child class should **extend** parent's functionality, not **narrow** it

---

### Understanding LSP

```
┌─────────────────┐
│    Class A      │
│  (Parent)       │
├─────────────────┤
│ + m1()          │
│ + m2()          │
│ + m3()          │
└─────────────────┘
         △
         │ extends
         │
┌─────────────────┐
│    Class B      │
│  (Child)        │
├─────────────────┤
│ + m1()          │ ← Inherited
│ + m2()          │ ← Inherited
│ + m3()          │ ← Inherited
│ + m4()          │ ← New method (extends functionality)
│ + m5()          │ ← New method (extends functionality)
└─────────────────┘
```

**Client Code:**
```java
void someMethod(A obj) {
    obj.m1();  // Client expects m1, m2, m3
    obj.m2();
    obj.m3();
}

// LSP says this should work fine:
someMethod(new B());  // B has m1, m2, m3 + extra m4, m5
```

**Why it works:**
- B has all methods of A (m1, m2, m3)
- B adds extra methods (m4, m5)
- Client only uses m1, m2, m3
- No problem!

---

### Example: Bank Account System

#### ❌ Problem: Violating LSP

**Class Diagram:**

```
┌──────────────────────────┐
│   <<abstract>>           │
│      Account             │
├──────────────────────────┤
│ + deposit() {abstract}   │
│ + withdraw() {abstract}  │
└──────────────────────────┘
         △
         │
    ┌────┴────┬──────────────┐
    │         │              │
┌─────────┐ ┌──────────┐ ┌──────────────────┐
│Savings  │ │Current   │ │FixedDeposit      │
│Account  │ │Account   │ │Account           │
├─────────┤ ├──────────┤ ├──────────────────┤
│+deposit │ │+deposit  │ │+deposit()        │
│+withdraw│ │+withdraw │ │+withdraw() ❌    │
└─────────┘ └──────────┘ │  throws Exception│
                         └──────────────────┘
```

**Problem:**

Fixed Deposit accounts **cannot withdraw** money!

```java
class FixedDepositAccount extends Account {
    public void deposit(double amount) {
        // Works fine
    }
    
    public void withdraw(double amount) {
        throw new UnsupportedOperationException(
            "Withdrawal not allowed in Fixed Deposit"
        );
    }
}
```

**Client Code:**
```java
class BankClient {
    private List<Account> accounts;
    
    public void processTransactions() {
        for (Account acc : accounts) {
            acc.deposit(1000);   // Works for all
            acc.withdraw(500);   // BREAKS for FixedDeposit! 💥
        }
    }
}
```

**Why LSP is violated:**
- Client expects **all** `Account` objects to support `withdraw()`
- `FixedDepositAccount` throws exception
- Client wasn't aware of this exception
- Code breaks unexpectedly

---

#### ❌ Bad Solution: Tight Coupling

```java
class BankClient {
    public void processTransactions() {
        for (Account acc : accounts) {
            acc.deposit(1000);
            
            // Tight coupling - client knows about specific types
            if (acc instanceof FixedDepositAccount) {
                // Skip withdrawal
            } else {
                acc.withdraw(500);
            }
        }
    }
}
```

**Why this is BAD:**
1. **Violates OCP**: Every new account type requires modifying client code
2. **Tight Coupling**: Client is aware of all account types
3. **Defeats Abstraction**: Client should only know about the interface

---

#### ✅ Solution: Proper Hierarchy

**Correct Class Diagram:**

```
┌──────────────────────────────┐
│   <<abstract>>               │
│   DepositOnlyAccount         │
├──────────────────────────────┤
│ + deposit() {abstract}       │
└──────────────────────────────┘
         △
         │ extends
         │
┌──────────────────────────────┐
│   <<abstract>>               │
│   WithdrawableAccount        │
├──────────────────────────────┤
│ + deposit() {inherited}      │
│ + withdraw() {abstract}      │
└──────────────────────────────┘
         △                 △
         │                 │
    ┌────┴────┐       ┌────┴────┐
    │         │       │         │
┌─────────┐ ┌──────┐ ┌──────────────┐
│Savings  │ │Current│ │FixedDeposit  │
│Account  │ │Account│ │Account       │
├─────────┤ ├──────┤ ├──────────────┤
│+deposit │ │+deposit│+deposit()    │
│+withdraw│ │+withdraw└──────────────┘
└─────────┘ └──────┘      │
                           │ extends
                           │
                  ┌────────┴────────┐
                  │DepositOnlyAccount│
                  └─────────────────┘
```

**Key Changes:**
1. Split `Account` into two interfaces:
   - `DepositOnlyAccount` (only deposit)
   - `WithdrawableAccount` (deposit + withdraw)

2. `WithdrawableAccount` extends `DepositOnlyAccount`

3. Each account type extends the appropriate interface:
   - `SavingsAccount` → `WithdrawableAccount`
   - `CurrentAccount` → `WithdrawableAccount`
   - `FixedDepositAccount` → `DepositOnlyAccount`

---

### Java Implementation

#### ❌ Violating LSP

```java
abstract class Account {
    protected double balance;
    
    public abstract void deposit(double amount);
    public abstract void withdraw(double amount);
}

class SavingsAccount extends Account {
    public void deposit(double amount) {
        balance += amount;
        System.out.println("Deposited: $" + amount);
    }
    
    public void withdraw(double amount) {
        if (amount > balance) {
            System.out.println("Insufficient funds");
        } else {
            balance -= amount;
            System.out.println("Withdrawn: $" + amount);
        }
    }
}

class CurrentAccount extends Account {
    public void deposit(double amount) {
        balance += amount;
        System.out.println("Deposited: $" + amount);
    }
    
    public void withdraw(double amount) {
        if (amount > balance) {
            System.out.println("Insufficient funds");
        } else {
            balance -= amount;
            System.out.println("Withdrawn: $" + amount);
        }
    }
}

class FixedDepositAccount extends Account {
    public void deposit(double amount) {
        balance += amount;
        System.out.println("Deposited: $" + amount);
    }
    
    public void withdraw(double amount) {
        // LSP VIOLATION!
        throw new UnsupportedOperationException(
            "Withdrawal not allowed in Fixed Deposit"
        );
    }
}

// Client
class BankClient {
    private List<Account> accounts;
    
    public BankClient(List<Account> accounts) {
        this.accounts = accounts;
    }
    
    public void processTransactions() {
        for (Account acc : accounts) {
            try {
                acc.deposit(1000);
                acc.withdraw(500);  // Will fail for FixedDeposit!
            } catch (Exception e) {
                System.out.println("Error: " + e.getMessage());
            }
        }
    }
}

// Usage
public class Main {
    public static void main(String[] args) {
        List<Account> accounts = new ArrayList<>();
        accounts.add(new SavingsAccount());
        accounts.add(new CurrentAccount());
        accounts.add(new FixedDepositAccount());  // Problem!
        
        BankClient client = new BankClient(accounts);
        client.processTransactions();
    }
}
```

**Output:**
```
Deposited: $1000
Withdrawn: $500
Deposited: $1000
Withdrawn: $500
Deposited: $1000
Error: Withdrawal not allowed in Fixed Deposit
```

#### ✅ Following LSP

```java
// Base interface - only deposit
abstract class DepositOnlyAccount {
    protected double balance;
    
    public abstract void deposit(double amount);
}

// Extended interface - deposit + withdraw
abstract class WithdrawableAccount extends DepositOnlyAccount {
    public abstract void withdraw(double amount);
}

// Savings account - can deposit and withdraw
class SavingsAccount extends WithdrawableAccount {
    public void deposit(double amount) {
        balance += amount;
        System.out.println("Deposited: $" + amount);
    }
    
    public void withdraw(double amount) {
        if (amount > balance) {
            System.out.println("Insufficient funds");
        } else {
            balance -= amount;
            System.out.println("Withdrawn: $" + amount);
        }
    }
}

// Current account - can deposit and withdraw
class CurrentAccount extends WithdrawableAccount {
    public void deposit(double amount) {
        balance += amount;
        System.out.println("Deposited: $" + amount);
    }
    
    public void withdraw(double amount) {
        if (amount > balance) {
            System.out.println("Insufficient funds");
        } else {
            balance -= amount;
            System.out.println("Withdrawn: $" + amount);
        }
    }
}

// Fixed deposit - can only deposit
class FixedDepositAccount extends DepositOnlyAccount {
    public void deposit(double amount) {
        balance += amount;
        System.out.println("Deposited: $" + amount);
    }
    // No withdraw method - follows LSP!
}

// Client with separate lists
class BankClient {
    private List<DepositOnlyAccount> depositOnlyAccounts;
    private List<WithdrawableAccount> withdrawableAccounts;
    
    public BankClient(
        List<DepositOnlyAccount> depositOnly,
        List<WithdrawableAccount> withdrawable
    ) {
        this.depositOnlyAccounts = depositOnly;
        this.withdrawableAccounts = withdrawable;
    }
    
    public void processTransactions() {
        // Process deposit-only accounts
        for (DepositOnlyAccount acc : depositOnlyAccounts) {
            acc.deposit(1000);
            // Client knows: can't withdraw from these
        }
        
        // Process withdrawable accounts
        for (WithdrawableAccount acc : withdrawableAccounts) {
            acc.deposit(1000);
            acc.withdraw(500);  // Safe - all support withdraw
        }
    }
}

// Usage
public class Main {
    public static void main(String[] args) {
        List<DepositOnlyAccount> depositOnly = new ArrayList<>();
        depositOnly.add(new FixedDepositAccount());
        
        List<WithdrawableAccount> withdrawable = new ArrayList<>();
        withdrawable.add(new SavingsAccount());
        withdrawable.add(new CurrentAccount());
        
        BankClient client = new BankClient(depositOnly, withdrawable);
        client.processTransactions();
    }
}
```

**Output:**
```
Deposited: $1000
Deposited: $1000
Withdrawn: $500
Deposited: $1000
Withdrawn: $500
```

---

## Summary 📚

### Single Responsibility Principle (SRP)
✅ One class = One responsibility  
✅ Separate concerns into different classes  
✅ Use composition to break down responsibilities  

### Open/Closed Principle (OCP)
✅ Open for extension, closed for modification  
✅ Use abstraction + inheritance + polymorphism  
✅ Add new features by creating new classes  

### Liskov Substitution Principle (LSP)
✅ Child classes should be substitutable for parent classes  
✅ Don't narrow down parent's functionality  
✅ Create proper hierarchies based on behavior  

---

## Key Takeaways 🎯

1. **SRP**: Break large classes into smaller, focused classes
2. **OCP**: Use interfaces to allow extension without modification
3. **LSP**: Design inheritance hierarchies carefully - child should extend, not restrict
4. **All Three Work Together**: These principles complement each other

**Next**: Interface Segregation Principle (ISP) and Dependency Inversion Principle (DIP)

---

## Practice Questions 💡

1. Identify SRP violations in your current codebase
2. Refactor a class that handles multiple responsibilities
3. Design a payment system following OCP (Cash, Card, UPI, Wallet)
4. Create a shape hierarchy following LSP (Circle, Rectangle, Square)

---

*Remember: SOLID principles are guidelines, not strict rules. Use judgment based on your specific use case!*
