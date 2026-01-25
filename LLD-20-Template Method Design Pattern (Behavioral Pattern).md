# LLD-20: Template Method Design Pattern (Behavioral Pattern)

## 📚 Table of Contents
1. [Introduction](#introduction)
2. [What is Template Method Pattern?](#what-is-template-method-pattern)
3. [Real-Life Analogy](#real-life-analogy)
4. [Problem Statement](#problem-statement)
5. [Solution: Template Method Pattern](#solution-template-method-pattern)
6. [Example Scenario: ML Model Training](#example-scenario-ml-model-training)
7. [UML Diagrams](#uml-diagrams)
8. [Implementation in Java](#implementation-in-java)
9. [Standard UML Diagram](#standard-uml-diagram)
10. [Key Concepts](#key-concepts)
11. [Real-World Use Cases](#real-world-use-cases)
12. [Key Takeaways](#key-takeaways)
13. [Interview Q&A](#interview-qa)

---

## 🎯 Introduction

Welcome back to our LLD series! Today we're learning about a very **interesting design pattern** - the **Template Method Design Pattern**.

### Why is this Pattern Special?

**Context**: By now, if you've followed this playlist, you've learned all the **core/foundational design patterns**. These are patterns you can plug into most generic problems. Congratulations on coming this far! You can now tackle most LLD problems.

**What's Different Now?** The patterns we're studying now are **problem-specific**. They solve very particular types of problems.

**Why is Template Method Special?**
- It solves a **specific use case** completely, as-is
- Perfect for **pipeline problems** in real-world applications
- Once you apply this pattern, your pipeline problems are solved
- Very **simple** to understand but **extremely powerful**

Let's dive in!

---

## 🤔 What is Template Method Pattern?
<img width="620" height="315" alt="image" src="https://github.com/user-attachments/assets/2cd9393c-bcff-4edc-9c78-a15315f143e7" />


**Core Concept**: The Template Method Pattern defines the **skeleton of an algorithm** in a base class, but lets subclasses **override specific steps** without changing the algorithm's structure.

**Key Idea**: 
- You have a **fixed sequence of steps** (pipeline)
- The **order must remain the same**
- But **individual step implementations** can vary

**Perfect for**: Any scenario where you need to enforce a strict execution order while allowing flexibility in how each step is implemented.

---

## 🌳 Real-Life Analogy

### Making Different Types of Beverages

Imagine you're making different beverages - Tea, Coffee, Hot Chocolate.

**Common Steps (Template)**:
1. Boil water
2. Add main ingredient (tea leaves/coffee powder/cocoa)
3. Add extras (sugar/milk)
4. Pour in cup
5. Serve

**The Order Never Changes!** But:
- Tea uses tea leaves
- Coffee uses coffee powder
- Hot Chocolate uses cocoa

The **template** (order) is fixed, but **implementations** differ.

---

## ❌ Problem Statement

### The Pipeline Problem

Imagine you're a **Data Scientist** working with different Machine Learning algorithms:
- Neural Networks
- Decision Trees
- Random Forest
- SVM (Support Vector Machines)

**Every ML model follows the same pipeline**:

```
Step 1: Load Data
Step 2: Preprocess Data
Step 3: Train Model
Step 4: Evaluate Results
Step 5: Save Model
```

### The Challenge

**What we want**:
- This **order must ALWAYS be followed**
- No matter which algorithm we use
- Steps should never be executed out of order

**What could go wrong without a pattern**:
- Developer might train before preprocessing ❌
- Developer might skip validation ❌
- Different developers might follow different orders ❌
- Code becomes inconsistent and error-prone ❌

**We need**: A way to **enforce this pipeline** so developers can't make mistakes!

---

## ✅ Solution: Template Method Pattern

### The Core Solution

**Template Method Pattern provides**:
1. A **template** (fixed order of execution)
2. Developers **cannot change the order**
3. Developers **can only implement individual steps**
4. The pattern **enforces consistency**

### How It Works

**Step 1**: Create an abstract base class with:
- A **template method** (defines the order) - **FINAL** (cannot be overridden)
- Abstract methods for each step (to be implemented by subclasses)

**Step 2**: Subclasses implement only the steps, not the order

**Step 3**: Client calls the template method, which executes steps in the correct order

---

## 🤖 Example Scenario: ML Model Training

### The Pipeline

Let's design a system for training different ML models with a fixed pipeline.

**Required Steps (in order)**:
1. **Load Data** - Load dataset from source
2. **Preprocess Data** - Clean, normalize, split data
3. **Train Model** - Train the algorithm
4. **Evaluate Model** - Check accuracy, loss
5. **Save Model** - Persist the trained model

**Different Models**:
- Neural Network
- Decision Tree
- (Future: Random Forest, SVM, etc.)

### The Design

#### Base Class: ModelTrainer

```java
abstract class ModelTrainer {
    
    // TEMPLATE METHOD - Defines the pipeline
    // FINAL - Cannot be overridden by subclasses
    public final void trainPipeline(String dataPath) {
        loadData(dataPath);
        preprocessData();
        trainModel();
        evaluateModel();
        saveModel();
    }
    
    // Abstract methods - Must be implemented by subclasses
    protected abstract void trainModel();
    protected abstract void evaluateModel();
    
    // Concrete methods - Common implementation
    protected void loadData(String path) {
        System.out.println("Loading dataset from: " + path);
    }
    
    protected void preprocessData() {
        System.out.println("Preprocessing: Splitting into train/test, normalizing...");
    }
    
    protected void saveModel() {
        System.out.println("Saving model to disk in standard format");
    }
}
```

**Key Points**:
- `trainPipeline()` is **FINAL** - enforces the order
- `trainModel()` and `evaluateModel()` are **abstract** - must be implemented
- `loadData()`, `preprocessData()`, `saveModel()` have **default implementations**

#### Concrete Class: Neural Network

```java
class NeuralNetworkTrainer extends ModelTrainer {
    
    @Override
    protected void trainModel() {
        System.out.println("Training Neural Network using 100 epochs...");
    }
    
    @Override
    protected void evaluateModel() {
        System.out.println("Evaluating accuracy and loss on validation set");
    }
    
    // Optionally override saveModel if needed
    @Override
    protected void saveModel() {
        System.out.println("Saving Neural Network in .h5 format");
    }
}
```

#### Concrete Class: Decision Tree

```java
class DecisionTreeTrainer extends ModelTrainer {
    
    @Override
    protected void trainModel() {
        System.out.println("Training Decision Tree with max_depth=10...");
    }
    
    @Override
    protected void evaluateModel() {
        System.out.println("Evaluating using confusion matrix and F1 score");
    }
    
    // Uses default saveModel() from base class
}
```

### Client Usage

```java
public class Client {
    public static void main(String[] args) {
        // Train Neural Network
        ModelTrainer nnTrainer = new NeuralNetworkTrainer();
        nnTrainer.trainPipeline("data/dataset.csv");
        
        System.out.println("\n" + "=".repeat(50) + "\n");
        
        // Train Decision Tree
        ModelTrainer dtTrainer = new DecisionTreeTrainer();
        dtTrainer.trainPipeline("data/dataset.csv");
    }
}
```

**What Happens**:
1. Client calls `trainPipeline()`
2. Template method executes steps in **fixed order**:
   - Load → Preprocess → Train → Evaluate → Save
3. Each model uses its own implementation for abstract methods
4. **Order is always guaranteed!**

---

## 📊 UML Diagrams

### Class Diagram
<img width="685" height="445" alt="image" src="https://github.com/user-attachments/assets/77104b33-0083-44e1-a678-9d42794bdfd0" />


```
┌─────────────────────────────────────┐
│      ModelTrainer (Abstract)        │
├─────────────────────────────────────┤
│ + trainPipeline(path): void [FINAL] │ ← Template Method
├─────────────────────────────────────┤
│ # loadData(path): void              │
│ # preprocessData(): void            │
│ # trainModel(): void [ABSTRACT]     │
│ # evaluateModel(): void [ABSTRACT]  │
│ # saveModel(): void                 │
└─────────────────────────────────────┘
                △
                │
        ┌───────┴────────┐
        │                │
┌───────▽────────┐  ┌────▽──────────┐
│ NeuralNetwork  │  │ DecisionTree  │
│    Trainer     │  │   Trainer     │
├────────────────┤  ├───────────────┤
│ + trainModel() │  │+ trainModel() │
│+ evaluateModel()│  │+evaluateModel()│
│ + saveModel()  │  │               │
└────────────────┘  └───────────────┘
```

**Relationships**:
- NeuralNetworkTrainer **IS-A** ModelTrainer
- DecisionTreeTrainer **IS-A** ModelTrainer
- Both override abstract methods
- Both inherit the template method

### Sequence Diagram: trainPipeline() Execution

```
Client    ModelTrainer    NeuralNetwork    DecisionTree
  │            │                │                │
  │─trainPipeline()→│           │                │
  │            │                │                │
  │            │─loadData()────→│                │
  │            │←───────────────│                │
  │            │                │                │
  │            │─preprocessData()→               │
  │            │←───────────────│                │
  │            │                │                │
  │            │─trainModel()──→│                │
  │            │  (polymorphic) │                │
  │            │←───────────────│                │
  │            │                │                │
  │            │─evaluateModel()→               │
  │            │  (polymorphic) │                │
  │            │←───────────────│                │
  │            │                │                │
  │            │─saveModel()───→│                │
  │            │←───────────────│                │
  │            │                │                │
  │←───────────│                │                │
```

**Flow Explanation**:
1. Client calls `trainPipeline()` on ModelTrainer reference
2. Template method executes steps in **fixed order**
3. Concrete methods (loadData, preprocessData, saveModel) execute from base class
4. Abstract methods (trainModel, evaluateModel) execute from **subclass** (polymorphism)
5. Order is **always** maintained

---

## 💻 Implementation in Java

### Complete Working Code

```java
// Abstract Base Class
abstract class ModelTrainer {
    
    /**
     * TEMPLATE METHOD - Defines the algorithm skeleton
     * FINAL - Cannot be overridden to ensure order is maintained
     */
    public final void trainPipeline(String dataPath) {
        loadData(dataPath);
        preprocessData();
        trainModel();
        evaluateModel();
        saveModel();
    }
    
    // Concrete method - Common implementation
    protected void loadData(String path) {
        System.out.println("Loading dataset from: " + path);
        System.out.println("Reading CSV, images, etc...");
    }
    
    // Concrete method - Common implementation
    protected void preprocessData() {
        System.out.println("Preprocessing data:");
        System.out.println("  - Splitting into train/test sets");
        System.out.println("  - Normalizing features");
        System.out.println("  - Handling missing values");
    }
    
    // Abstract method - Must be implemented by subclasses
    protected abstract void trainModel();
    
    // Abstract method - Must be implemented by subclasses
    protected abstract void evaluateModel();
    
    // Concrete method - Common implementation
    protected void saveModel() {
        System.out.println("Saving model to disk in standard format");
    }
}

// Concrete Implementation 1: Neural Network
class NeuralNetworkTrainer extends ModelTrainer {
    
    @Override
    protected void trainModel() {
        System.out.println("Training Neural Network:");
        System.out.println("  - Using 100 epochs");
        System.out.println("  - Batch size: 32");
        System.out.println("  - Optimizer: Adam");
    }
    
    @Override
    protected void evaluateModel() {
        System.out.println("Evaluating Neural Network:");
        System.out.println("  - Accuracy: 95.2%");
        System.out.println("  - Loss: 0.048");
    }
    
    // Override saveModel for custom format
    @Override
    protected void saveModel() {
        System.out.println("Saving Neural Network in .h5 format");
    }
}

// Concrete Implementation 2: Decision Tree
class DecisionTreeTrainer extends ModelTrainer {
    
    @Override
    protected void trainModel() {
        System.out.println("Training Decision Tree:");
        System.out.println("  - Max depth: 10");
        System.out.println("  - Min samples split: 2");
        System.out.println("  - Criterion: Gini");
    }
    
    @Override
    protected void evaluateModel() {
        System.out.println("Evaluating Decision Tree:");
        System.out.println("  - Accuracy: 92.8%");
        System.out.println("  - F1 Score: 0.91");
        System.out.println("  - Confusion Matrix: [[45, 5], [3, 47]]");
    }
    
    // Uses default saveModel() from base class
}

// Client Code
public class TemplateMethodDemo {
    public static void main(String[] args) {
        System.out.println("=== Neural Network Training ===\n");
        ModelTrainer nnTrainer = new NeuralNetworkTrainer();
        nnTrainer.trainPipeline("data/neural_dataset.csv");
        
        System.out.println("\n" + "=".repeat(60) + "\n");
        
        System.out.println("=== Decision Tree Training ===\n");
        ModelTrainer dtTrainer = new DecisionTreeTrainer();
        dtTrainer.trainPipeline("data/tree_dataset.csv");
    }
}
```

### Expected Output

```
=== Neural Network Training ===

Loading dataset from: data/neural_dataset.csv
Reading CSV, images, etc...
Preprocessing data:
  - Splitting into train/test sets
  - Normalizing features
  - Handling missing values
Training Neural Network:
  - Using 100 epochs
  - Batch size: 32
  - Optimizer: Adam
Evaluating Neural Network:
  - Accuracy: 95.2%
  - Loss: 0.048
Saving Neural Network in .h5 format

============================================================

=== Decision Tree Training ===

Loading dataset from: data/tree_dataset.csv
Reading CSV, images, etc...
Preprocessing data:
  - Splitting into train/test sets
  - Normalizing features
  - Handling missing values
Training Decision Tree:
  - Max depth: 10
  - Min samples split: 2
  - Criterion: Gini
Evaluating Decision Tree:
  - Accuracy: 92.8%
  - F1 Score: 0.91
  - Confusion Matrix: [[45, 5], [3, 47]]
Saving model to disk in standard format
```

### Code Walkthrough

#### 1. Template Method (trainPipeline)

```java
public final void trainPipeline(String dataPath) {
    loadData(dataPath);      // Step 1
    preprocessData();         // Step 2
    trainModel();            // Step 3 - Abstract
    evaluateModel();         // Step 4 - Abstract
    saveModel();             // Step 5
}
```

**Key Points**:
- **FINAL** keyword prevents overriding
- Defines the **exact order** of execution
- Calls both concrete and abstract methods
- Client only needs to call this one method

#### 2. Abstract Methods

```java
protected abstract void trainModel();
protected abstract void evaluateModel();
```

**Purpose**:
- **Must** be implemented by subclasses
- Allow different algorithms to have different implementations
- Called by template method at the right time

#### 3. Concrete Methods (Hook Methods)

```java
protected void loadData(String path) {
    // Common implementation
}

protected void preprocessData() {
    // Common implementation
}

protected void saveModel() {
    // Common implementation
}
```

**Purpose**:
- Provide **default implementation**
- Can be **optionally overridden** by subclasses
- Reduce code duplication

**Example**: 
- DecisionTree uses default `saveModel()`
- NeuralNetwork overrides `saveModel()` for custom format

#### 4. Polymorphism in Action

```java
ModelTrainer trainer = new NeuralNetworkTrainer();
trainer.trainPipeline("data.csv");
```

**What happens**:
1. Reference type: `ModelTrainer`
2. Object type: `NeuralNetworkTrainer`
3. Template method calls from `ModelTrainer`
4. Abstract methods execute from `NeuralNetworkTrainer` (polymorphism)
5. Order is guaranteed by template method

---

## 📐 Standard UML Diagram

### Generic Template Method Structure

```
┌─────────────────────────────┐
│   AbstractClass             │
├─────────────────────────────┤
│ + templateMethod(): void    │ ← FINAL - Defines algorithm
├─────────────────────────────┤
│ # step1(): void             │ ← Concrete method
│ # step2(): void [ABSTRACT]  │ ← Abstract method
│ # step3(): void [ABSTRACT]  │ ← Abstract method
└─────────────────────────────┘
              △
              │
      ┌───────┴────────┐
      │                │
┌─────▽─────┐    ┌─────▽─────┐
│ConcreteA  │    │ConcreteB  │
├───────────┤    ├───────────┤
│+ step2()  │    │+ step2()  │
│+ step3()  │    │+ step3()  │
└───────────┘    └───────────┘
```

**Components**:

1. **AbstractClass**
   - Contains the **template method** (final)
   - Defines algorithm skeleton
   - Has abstract methods (to be implemented)
   - May have concrete methods (default implementation)

2. **ConcreteClass A & B**
   - Implement abstract methods
   - Inherit template method
   - Can override concrete methods if needed

**Execution Flow**:
```
templateMethod() {
    step1();  // Concrete - executes from AbstractClass
    step2();  // Abstract - executes from ConcreteA/B
    step3();  // Abstract - executes from ConcreteA/B
}
```

---

## 🔑 Key Concepts

### 1. The Template Method

**Definition**: A method that defines the skeleton of an algorithm.

**Characteristics**:
- Declared as **FINAL** (Java) or **non-virtual** (C++)
- Cannot be overridden by subclasses
- Calls other methods in a specific order
- Enforces the algorithm structure

**Example**:
```java
public final void processPayment() {
    validateAccount();
    debitAmount();
    creditAmount();
    logTransaction();
}
```

### 2. Abstract vs Concrete Methods

| Type | Purpose | Implementation | Override |
|------|---------|----------------|----------|
| **Abstract** | Must vary per subclass | None (abstract) | Required |
| **Concrete** | Common behavior | Provided in base | Optional |

**Example**:
```java
// Abstract - Must implement
protected abstract void trainModel();

// Concrete - Can override
protected void saveModel() {
    // Default implementation
}
```

### 3. The Hollywood Principle

**"Don't call us, we'll call you"**

**Meaning**:
- High-level component (AbstractClass) controls the flow
- Low-level components (ConcreteClasses) are called when needed
- Subclasses don't control the algorithm flow

**In our example**:
- `ModelTrainer` (high-level) controls when to call `trainModel()`
- `NeuralNetworkTrainer` (low-level) just implements `trainModel()`
- Client doesn't decide the order

### 4. Inversion of Control

**Traditional Approach**:
```java
// Client controls the flow
trainer.loadData();
trainer.preprocess();
trainer.train();
trainer.evaluate();
trainer.save();
```

**Template Method Approach**:
```java
// Template controls the flow
trainer.trainPipeline();
```

**Benefit**: Consistency, no room for error

---

## 🌍 Real-World Use Cases

### 1. Payment Processing Systems ⭐

**Scenario**: Different payment methods (UPI, Credit Card, Debit Card, Net Banking)

**Pipeline**:
1. Validate account (sufficient balance)
2. Debit from source account
3. Credit to destination account
4. Log transaction
5. Send confirmation

**Implementation**:
```java
abstract class PaymentProcessor {
    public final void processPayment(double amount) {
        validateAccount(amount);
        debitAmount(amount);
        creditAmount(amount);
        logTransaction();
        sendConfirmation();
    }
    
    protected abstract void validateAccount(double amount);
    protected abstract void debitAmount(double amount);
    protected abstract void creditAmount(double amount);
    
    protected void logTransaction() {
        // Common implementation
    }
    
    protected void sendConfirmation() {
        // Common implementation
    }
}

class UPIPayment extends PaymentProcessor {
    // Implement abstract methods
}

class CreditCardPayment extends PaymentProcessor {
    // Implement abstract methods
}
```

**Why Template Method?**
- Order must be strict (can't credit before debit!)
- Different payment methods have different validation/processing
- Ensures consistency across all payment types

### 2. Data Processing Pipelines

**Scenario**: ETL (Extract, Transform, Load) operations

**Pipeline**:
1. Extract data from source
2. Validate data
3. Transform data
4. Load into destination
5. Generate report

**Different Sources**: CSV, JSON, XML, Database

### 3. Game Development

**Scenario**: Different game characters with common behavior flow

**Pipeline**:
1. Initialize character
2. Load assets
3. Setup AI
4. Start game loop
5. Cleanup

**Different Characters**: Warrior, Mage, Archer (each has different AI, assets)

### 4. Testing Frameworks

**Scenario**: Unit test execution

**Pipeline**:
1. Setup (before each test)
2. Execute test
3. Teardown (after each test)

**Example**: JUnit's `@Before`, `@Test`, `@After`

### 5. Document Generation

**Scenario**: Generate different document formats (PDF, Word, HTML)

**Pipeline**:
1. Create document structure
2. Add header
3. Add content
4. Add footer
5. Save document

**Different Formats**: Each format has different rendering logic

### 6. Web Frameworks

**Scenario**: HTTP request handling

**Pipeline**:
1. Parse request
2. Authenticate user
3. Authorize access
4. Execute business logic
5. Render response

**Example**: Spring MVC, Django

---

## 🎓 Key Takeaways

### When to Use Template Method Pattern

✅ **Use when**:
1. You have a **fixed algorithm structure** (pipeline)
2. Steps must execute in a **specific order**
3. Some steps vary, but the **order doesn't**
4. You want to **prevent developers from making mistakes**
5. You need to **enforce consistency** across implementations

❌ **Don't use when**:
1. Algorithm structure is not fixed
2. Order of steps can vary
3. Steps are completely independent
4. You need more flexibility in the flow

### Advantages

1. **Code Reuse**: Common code in base class
2. **Consistency**: Algorithm structure is guaranteed
3. **Flexibility**: Subclasses can customize specific steps
4. **Control**: Base class controls the flow (Hollywood Principle)
5. **Easy to Extend**: Add new implementations easily

### Disadvantages

1. **Rigid Structure**: Hard to change the algorithm flow
2. **Inheritance Required**: Tight coupling with base class
3. **Liskov Substitution**: Must ensure subclasses don't break the contract
4. **Debugging**: Flow is spread across multiple classes

### Design Principles Applied

1. **Open-Closed Principle**: Open for extension (new subclasses), closed for modification (template method is final)
2. **Hollywood Principle**: "Don't call us, we'll call you"
3. **Dependency Inversion**: High-level module (template) doesn't depend on low-level modules (concrete implementations)
4. **Single Responsibility**: Each class has one reason to change

---

## ❓ Interview Q&A

### Q1: What is the Template Method Design Pattern? Explain with an example.

**Answer**:

The Template Method Design Pattern is a behavioral pattern that defines the **skeleton of an algorithm** in a base class, but lets subclasses **override specific steps** without changing the algorithm's structure.

**Key Components**:
1. **Template Method**: Final method that defines the algorithm structure
2. **Abstract Methods**: Steps that must be implemented by subclasses
3. **Concrete Methods**: Steps with default implementation (can be overridden)

**Example**: ML Model Training

```java
abstract class ModelTrainer {
    // Template Method - FINAL
    public final void trainPipeline(String path) {
        loadData(path);
        preprocessData();
        trainModel();      // Abstract
        evaluateModel();   // Abstract
        saveModel();
    }
    
    protected abstract void trainModel();
    protected abstract void evaluateModel();
    
    protected void loadData(String path) { /* common */ }
    protected void preprocessData() { /* common */ }
    protected void saveModel() { /* common */ }
}
```

**Benefit**: Ensures all models follow the same pipeline: Load → Preprocess → Train → Evaluate → Save

---

### Q2: What problem does Template Method Pattern solve?

**Answer**:

**Problem**: When you have a **fixed sequence of steps** that must be executed in a specific order, but individual step implementations may vary.

**Without Template Method**:
```java
// Developer might make mistakes
model.trainModel();        // ❌ Training before loading data!
model.loadData();
model.evaluateModel();     // ❌ Wrong order
```

**Issues**:
1. Developers can execute steps in wrong order
2. Inconsistent implementations across the codebase
3. Hard to maintain and debug
4. Violates DRY principle

**With Template Method**:
```java
// Order is guaranteed
model.trainPipeline();  // ✅ Always: Load → Preprocess → Train → Evaluate → Save
```

**Solution**:
1. Enforces correct order
2. Prevents developer mistakes
3. Ensures consistency
4. Centralizes algorithm structure

---

### Q3: What is the difference between abstract and concrete methods in Template Method Pattern?

**Answer**:

| Aspect | Abstract Methods | Concrete Methods |
|--------|-----------------|------------------|
| **Purpose** | Steps that vary | Steps that are common |
| **Implementation** | None (must implement) | Provided in base class |
| **Override** | Required | Optional |
| **Example** | `trainModel()` | `loadData()` |

**Example**:

```java
abstract class ModelTrainer {
    // Abstract - MUST implement
    protected abstract void trainModel();
    protected abstract void evaluateModel();
    
    // Concrete - CAN override
    protected void loadData(String path) {
        System.out.println("Loading from: " + path);
    }
    
    protected void saveModel() {
        System.out.println("Saving model");
    }
}

class NeuralNetwork extends ModelTrainer {
    // Must implement abstract methods
    protected void trainModel() { /* NN specific */ }
    protected void evaluateModel() { /* NN specific */ }
    
    // Can optionally override concrete methods
    protected void saveModel() { /* Custom save */ }
}
```

**When to use which**:
- **Abstract**: When implementation MUST differ (e.g., training algorithm)
- **Concrete**: When implementation is common but CAN be customized (e.g., saving format)

---

### Q4: Why is the template method declared as final?

**Answer**:

**Reason**: To **prevent subclasses from changing the algorithm structure**.

**Without final**:
```java
class BadNeuralNetwork extends ModelTrainer {
    // ❌ Overriding template method - BAD!
    public void trainPipeline(String path) {
        trainModel();      // Wrong order!
        loadData(path);
        evaluateModel();
    }
}
```

**With final**:
```java
abstract class ModelTrainer {
    // ✅ Cannot be overridden
    public final void trainPipeline(String path) {
        loadData(path);
        preprocessData();
        trainModel();
        evaluateModel();
        saveModel();
    }
}

class GoodNeuralNetwork extends ModelTrainer {
    // ✅ Can only implement abstract methods
    protected void trainModel() { /* ... */ }
    protected void evaluateModel() { /* ... */ }
}
```

**Benefits**:
1. **Guarantees order**: Algorithm structure cannot be changed
2. **Prevents mistakes**: Developers can't accidentally break the flow
3. **Enforces contract**: Subclasses must follow the template
4. **Maintains consistency**: All implementations follow the same pattern

**In Java**: Use `final`
**In C++**: Make it non-virtual or use `final` (C++11)

---

### Q5: What is the Hollywood Principle? How does it relate to Template Method?

**Answer**:

**Hollywood Principle**: "Don't call us, we'll call you"

**Meaning**: 
- High-level components control when to call low-level components
- Low-level components don't control the flow
- Inversion of control

**In Template Method**:

```java
// High-level component (AbstractClass)
abstract class ModelTrainer {
    public final void trainPipeline() {
        loadData();
        trainModel();  // "We'll call you"
        saveModel();
    }
    
    protected abstract void trainModel();
}

// Low-level component (ConcreteClass)
class NeuralNetwork extends ModelTrainer {
    // "Don't call us"
    protected void trainModel() {
        // Just implement, don't control when it's called
    }
}
```

**Traditional Approach** (No Hollywood Principle):
```java
// Client controls everything
trainer.loadData();
trainer.trainModel();  // Client decides when
trainer.saveModel();
```

**Template Method Approach** (Hollywood Principle):
```java
// Template controls everything
trainer.trainPipeline();  // Template decides when to call trainModel()
```

**Benefits**:
1. **Centralized control**: Flow logic in one place
2. **Reduced coupling**: Subclasses don't need to know the full algorithm
3. **Easier maintenance**: Change flow in one place

---

### Q6: Can you have multiple template methods in one class?

**Answer**:

**Yes!** You can have multiple template methods for different workflows.

**Example**:

```java
abstract class DataProcessor {
    // Template Method 1: Full processing
    public final void processData(String path) {
        loadData(path);
        validateData();
        transformData();
        saveData();
    }
    
    // Template Method 2: Quick processing (skip validation)
    public final void quickProcess(String path) {
        loadData(path);
        transformData();
        saveData();
    }
    
    // Template Method 3: Validation only
    public final void validateOnly(String path) {
        loadData(path);
        validateData();
    }
    
    protected abstract void loadData(String path);
    protected abstract void validateData();
    protected abstract void transformData();
    protected abstract void saveData();
}
```

**Use Cases**:
1. Different workflows for the same domain
2. Full vs. partial processing
3. Different levels of detail

**Best Practice**: Keep template methods focused and cohesive

---

### Q7: How is Template Method different from Strategy Pattern?

**Answer**:

Both are behavioral patterns, but serve different purposes.

| Aspect | Template Method | Strategy Pattern |
|--------|----------------|------------------|
| **Purpose** | Define algorithm skeleton | Encapsulate algorithms |
| **Structure** | Inheritance-based | Composition-based |
| **Flexibility** | Fixed structure, vary steps | Swap entire algorithm |
| **Control** | Base class controls flow | Client controls which strategy |
| **Coupling** | Tight (inheritance) | Loose (composition) |

**Template Method**:
```java
abstract class ModelTrainer {
    public final void trainPipeline() {
        loadData();
        trainModel();  // Varies
        saveModel();
    }
    protected abstract void trainModel();
}

class NeuralNetwork extends ModelTrainer {
    protected void trainModel() { /* NN training */ }
}
```
- **Fixed flow**: Load → Train → Save
- **Vary step**: Only training step changes

**Strategy Pattern**:
```java
interface TrainingStrategy {
    void train();
}

class ModelTrainer {
    private TrainingStrategy strategy;
    
    public void trainModel() {
        strategy.train();  // Entire algorithm varies
    }
}
```
- **Flexible**: Can swap entire training algorithm at runtime
- **No fixed flow**: Just execute the strategy

**When to use which**:
- **Template Method**: When you have a fixed pipeline with varying steps
- **Strategy**: When you want to swap entire algorithms dynamically

---

### Q8: What are real-world examples of Template Method Pattern?

**Answer**:

**1. Payment Processing** ⭐
```
Pipeline: Validate → Debit → Credit → Log → Confirm
Different: UPI, Credit Card, Debit Card, Net Banking
```

**2. ETL Pipelines**
```
Pipeline: Extract → Validate → Transform → Load → Report
Different: CSV, JSON, XML, Database sources
```

**3. Testing Frameworks**
```
Pipeline: Setup → Execute Test → Teardown
Example: JUnit (@Before, @Test, @After)
```

**4. Web Frameworks**
```
Pipeline: Parse Request → Authenticate → Authorize → Execute → Render
Example: Spring MVC, Django
```

**5. Game Development**
```
Pipeline: Initialize → Load Assets → Setup AI → Game Loop → Cleanup
Different: Warrior, Mage, Archer characters
```

**6. Document Generation**
```
Pipeline: Create Structure → Header → Content → Footer → Save
Different: PDF, Word, HTML formats
```

**7. Build Systems**
```
Pipeline: Clean → Compile → Test → Package → Deploy
Example: Maven, Gradle
```

**Common Pattern**: Fixed sequence of steps, varying implementations

---

### Q9: What are the advantages and disadvantages of Template Method Pattern?

**Answer**:

**Advantages** ✅:

1. **Code Reuse**
   - Common code in base class
   - Avoid duplication

2. **Consistency**
   - Algorithm structure is guaranteed
   - All implementations follow same flow

3. **Control**
   - Base class controls the flow
   - Prevents developer mistakes

4. **Flexibility**
   - Subclasses can customize specific steps
   - Easy to add new implementations

5. **Maintainability**
   - Change algorithm structure in one place
   - Easy to understand the flow

**Disadvantages** ❌:

1. **Rigid Structure**
   - Hard to change the algorithm flow
   - All subclasses must follow the same structure

2. **Inheritance Required**
   - Tight coupling with base class
   - Can't use composition

3. **Liskov Substitution**
   - Subclasses must not break the contract
   - Can be tricky to maintain

4. **Debugging**
   - Flow is spread across multiple classes
   - Harder to trace execution

5. **Limited Flexibility**
   - Can't change order at runtime
   - Can't skip steps easily

**Trade-off**: Consistency vs. Flexibility

---

### Q10: How does Template Method Pattern follow SOLID principles?

**Answer**:

**1. Single Responsibility Principle (SRP)** ✅
- Each class has one responsibility
- Base class: Define algorithm structure
- Subclasses: Implement specific steps

**2. Open-Closed Principle (OCP)** ✅
- **Open for extension**: Add new subclasses
- **Closed for modification**: Template method is final

```java
// Add new model without modifying existing code
class RandomForestTrainer extends ModelTrainer {
    protected void trainModel() { /* RF training */ }
    protected void evaluateModel() { /* RF evaluation */ }
}
```

**3. Liskov Substitution Principle (LSP)** ✅
- Subclasses can replace base class
- Template method works with any subclass

```java
ModelTrainer trainer = new NeuralNetworkTrainer();
trainer.trainPipeline();  // Works

trainer = new DecisionTreeTrainer();
trainer.trainPipeline();  // Also works
```

**4. Interface Segregation Principle (ISP)** ⚠️
- **Potential violation**: Subclasses might not need all methods
- **Solution**: Split into smaller abstract classes if needed

**5. Dependency Inversion Principle (DIP)** ✅
- High-level module (template) doesn't depend on low-level modules
- Both depend on abstraction (abstract methods)

**Overall**: Template Method aligns well with SOLID, especially OCP and LSP

---

### Q11: Can you provide a step-by-step guide to implement Template Method Pattern?

**Answer**:

**Step 1: Identify the Algorithm**
- Find the sequence of steps
- Determine which steps are common and which vary

**Example**: Payment Processing
```
Steps: Validate → Debit → Credit → Log → Confirm
Common: Log, Confirm
Vary: Validate, Debit, Credit
```

**Step 2: Create Abstract Base Class**
```java
abstract class PaymentProcessor {
    // Template method
    public final void processPayment(double amount) {
        // Define the algorithm
    }
}
```

**Step 3: Add Template Method**
```java
public final void processPayment(double amount) {
    validateAccount(amount);
    debitAmount(amount);
    creditAmount(amount);
    logTransaction();
    sendConfirmation();
}
```

**Step 4: Declare Abstract Methods (for varying steps)**
```java
protected abstract void validateAccount(double amount);
protected abstract void debitAmount(double amount);
protected abstract void creditAmount(double amount);
```

**Step 5: Implement Concrete Methods (for common steps)**
```java
protected void logTransaction() {
    System.out.println("Transaction logged");
}

protected void sendConfirmation() {
    System.out.println("Confirmation sent");
}
```

**Step 6: Create Concrete Subclasses**
```java
class UPIPayment extends PaymentProcessor {
    protected void validateAccount(double amount) { /* UPI validation */ }
    protected void debitAmount(double amount) { /* UPI debit */ }
    protected void creditAmount(double amount) { /* UPI credit */ }
}

class CreditCardPayment extends PaymentProcessor {
    protected void validateAccount(double amount) { /* CC validation */ }
    protected void debitAmount(double amount) { /* CC debit */ }
    protected void creditAmount(double amount) { /* CC credit */ }
}
```

**Step 7: Use the Pattern**
```java
PaymentProcessor processor = new UPIPayment();
processor.processPayment(100.0);
```

**Checklist**:
- ✅ Template method is final
- ✅ Abstract methods for varying steps
- ✅ Concrete methods for common steps
- ✅ Subclasses implement abstract methods
- ✅ Client calls template method

---

### Q12: When should you NOT use Template Method Pattern?

**Answer**:

❌ **Don't use when**:

**1. Algorithm Structure is Not Fixed**
```java
// Bad use case
// Sometimes: Load → Process → Save
// Sometimes: Process → Load → Save
// Order varies - use Strategy instead
```

**2. Steps are Completely Independent**
```java
// Bad use case
// Steps can be executed in any order
// No dependencies between steps
// Use Command pattern or simple methods
```

**3. Need Runtime Flexibility**
```java
// Bad use case
// Need to skip steps at runtime
// Need to change order at runtime
// Use Strategy or Chain of Responsibility
```

**4. Too Many Variations**
```java
// Bad use case
// 20 different subclasses with slight variations
// Leads to class explosion
// Consider using Strategy with composition
```

**5. Simple Use Case**
```java
// Bad use case
// Only 2-3 simple steps
// Overhead of pattern not justified
// Just use simple methods
```

**Better Alternatives**:
- **Strategy Pattern**: When you need to swap entire algorithms
- **Chain of Responsibility**: When steps can be added/removed dynamically
- **Command Pattern**: When steps are independent
- **Simple Methods**: When use case is too simple

---

## 📝 Summary

The **Template Method Design Pattern** is a powerful behavioral pattern for enforcing a fixed algorithm structure while allowing flexibility in individual steps.

**Key Points**:
- Defines the **skeleton of an algorithm** in a base class
- **Template method** is final - cannot be overridden
- **Abstract methods** must be implemented by subclasses
- **Concrete methods** provide default implementation
- Follows the **Hollywood Principle** ("Don't call us, we'll call you")
- Perfect for **pipeline problems** with fixed order

**When to Use**:
- Fixed sequence of steps
- Steps must execute in specific order
- Some steps vary, but order doesn't
- Need to enforce consistency

**Real-World Examples**:
- Payment processing
- ML model training
- ETL pipelines
- Testing frameworks
- Document generation

**Remember**: If you have a pipeline with a fixed order but varying implementations, think Template Method!

---

**Happy Learning! 🚀**
