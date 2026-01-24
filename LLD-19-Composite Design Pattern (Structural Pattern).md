# LLD-18: Composite Design Pattern (Structural Pattern)

## 📚 Table of Contents
1. [Introduction](#introduction)
2. [What is Composite Design Pattern?](#what-is-composite-design-pattern)
3. [Real-Life Analogy](#real-life-analogy)
4. [Problem Statement](#problem-statement)
5. [Solution: Composite Design Pattern](#solution-composite-design-pattern)
6. [Example Scenario: File System](#example-scenario-file-system)
7. [UML Diagrams](#uml-diagrams)
8. [Implementation in Java](#implementation-in-java)
9. [Standard UML Diagram](#standard-uml-diagram)
10. [Key Concepts](#key-concepts)
11. [Real-World Use Cases](#real-world-use-cases)
12. [Key Takeaways](#key-takeaways)
13. [Interview Q&A](#interview-qa)

---

## 🎯 Introduction

Welcome back to our LLD series! Today we're learning about the **Composite Design Pattern**.

### Important Context

So far, we've studied **foundational design patterns** - patterns that can be plugged into most generic problems. Patterns like Strategy, Factory, and Singleton are used almost everywhere.

**Now we're entering a new phase**: The patterns we'll study from here onwards are **use-case specific**. They solve very particular types of problems, not generic ones.

**Does this make them less important?** Absolutely not! 

If you're working on a real-world project or facing an interview problem that directly maps to these patterns, knowing them will be incredibly powerful. These patterns elegantly solve specific problem types - if you recognize the pattern, you can immediately apply the solution.

---

## 🤔 What is Composite Design Pattern?

**Simple Definition**: If you have a background in Data Structures and have studied **Trees**, then you already understand the Composite Design Pattern. You've used it!

**Core Idea**: Any problem statement that can be expressed in a **hierarchical form** (tree structure) with two types of nodes:
- **Leaf Nodes** (end points, no children)
- **Intermediate Nodes** (composite nodes, can have children)

Such problems are solved using the Composite Design Pattern.

---

## 🌳 Real-Life Analogy

### The Classic Example: File System

This is THE classic example for Composite Design Pattern. In fact, it seems like this pattern was made specifically for this problem!

**Scenario**: Designing a Folder System (like any Operating System)

In any file system:
- We have **Folders** (can contain more folders or files)
- We have **Files** (end points, cannot contain anything else)

```
Root/
├── file1.txt
├── file2.txt
├── Documents/
│   ├── resume.pdf
│   └── notes.txt
└── Images/
    └── photo.jpg
```

**Key Observation**:
- **Files** = Leaf Nodes (cannot be expanded further)
- **Folders** = Composite Nodes (can contain more folders/files)

This hierarchical structure is exactly what Composite Pattern handles!

---

## ❌ Problem Statement

### Designing a Folder System Without Design Patterns

Let's imagine we try to solve this from scratch, without knowing about Composite Design Pattern.

#### Naive Approach

**Step 1**: Create separate classes for File and Folder

```java
class File {
    String name;
    int size;
    String content;
    
    void open() {
        // Display file content
    }
}

class Folder {
    String name;
    List<File> files;
    List<Folder> folders;
    
    void ls() {
        // List directory contents
    }
}
```

**Relationship**:
- Folder can have **1-to-many Files**
- Folder can have **1-to-many Folders**

#### The Problems Begin

Now let's implement an `openAll()` method that expands the entire hierarchy.

**Problem 1: Need a Common List**

If we keep separate lists for files and folders, we can't maintain order. We need a **common list** to store both files and folders together.

**Solution**: Use a list of pointers (in C++) or references (in Java) to store both types.

```java
class Folder {
    String name;
    List<Object> commonList; // Can store both File and Folder
}
```

**Problem 2: Type Checking Everywhere**

Now when we implement `openAll()`, we need to check the type of each element:

```java
void openAll() {
    for (Object element : commonList) {
        if (element instanceof File) {
            // Print file name
            System.out.println(((File) element).name);
        } else if (element instanceof Folder) {
            // Recursively call openAll on folder
            ((Folder) element).openAll();
        }
    }
}
```

**Problem 3: Complexity Grows**

- Every method needs if-else checks
- Adding new methods (like `cd`, `getSize`) requires more type checking
- Code becomes messy and hard to maintain
- Violates Open-Closed Principle

---

## ✅ Solution: Composite Design Pattern

### The Core Principle

**Composite Design Pattern says**: 

> You have two types of elements - **Composite** and **Leaf**. Treat them the SAME way. Don't treat them differently.

**How?** By giving them a **common interface** (or abstract class).

### Key Benefits

1. **Uniform Treatment**: Both leaf and composite nodes implement the same interface
2. **No Type Checking**: Client doesn't need to know if it's dealing with a leaf or composite
3. **Polymorphism Power**: Call the same method on both types
4. **Easy to Extend**: Adding new operations is simpler

---

## 📁 Example Scenario: File System

### Hierarchy Structure

Let's design this file system:

```
Root/
├── file1.txt
├── file2.txt
├── Core/
│   ├── file3.txt
│   ├── file4.txt
│   └── User/
│       └── file5.txt
└── Images/
```

**Breakdown**:
- Root folder contains: 2 files (file1, file2) and 2 folders (Core, Images)
- Core folder contains: 2 files (file3, file4) and 1 folder (User)
- User folder contains: 1 file (file5)

### The Design

#### Step 1: Common Interface

```java
interface FileSystemItem {
    void ls(int indent);           // List directory
    void openAll(int indent);      // Expand entire hierarchy
    int getSize();                 // Get size
    String getName();              // Get name
    boolean isFolder();            // Check if folder
    FileSystemItem cd(String target); // Change directory
}
```

#### Step 2: File Class (Leaf Node)

```java
class File implements FileSystemItem {
    private String name;
    private int size;
    
    public File(String name, int size) {
        this.name = name;
        this.size = size;
    }
    
    @Override
    public void ls(int indent) {
        System.out.println(name);
    }
    
    @Override
    public void openAll(int indent) {
        System.out.println(name);
    }
    
    @Override
    public int getSize() {
        return size;
    }
    
    @Override
    public String getName() {
        return name;
    }
    
    @Override
    public boolean isFolder() {
        return false;
    }
    
    @Override
    public FileSystemItem cd(String target) {
        return null; // Cannot cd into a file
    }
}
```

**Key Points**:
- File is a **leaf node** - it's an endpoint
- `ls()` and `openAll()` just print the file name
- `cd()` returns null (operation not supported)
- `isFolder()` returns false

#### Step 3: Folder Class (Composite Node)

```java
class Folder implements FileSystemItem {
    private String name;
    private List<FileSystemItem> children;
    
    public Folder(String name) {
        this.name = name;
        this.children = new ArrayList<>();
    }
    
    public void add(FileSystemItem item) {
        children.add(item);
    }
    
    @Override
    public void ls(int indent) {
        String indentation = " ".repeat(indent);
        for (FileSystemItem child : children) {
            if (child.isFolder()) {
                System.out.println(indentation + "+ " + child.getName());
            } else {
                System.out.println(indentation + child.getName());
            }
        }
    }
    
    @Override
    public void openAll(int indent) {
        String indentation = " ".repeat(indent);
        System.out.println(indentation + name);
        
        for (FileSystemItem child : children) {
            child.openAll(indent + 4);
        }
    }
    
    @Override
    public int getSize() {
        int total = 0;
        for (FileSystemItem child : children) {
            total += child.getSize();
        }
        return total;
    }
    
    @Override
    public String getName() {
        return name;
    }
    
    @Override
    public boolean isFolder() {
        return true;
    }
    
    @Override
    public FileSystemItem cd(String target) {
        for (FileSystemItem child : children) {
            if (child.isFolder() && child.getName().equals(target)) {
                return child;
            }
        }
        return null;
    }
}
```

**Key Points**:
- Folder has **both IS-A and HAS-A relationship** with FileSystemItem
- `children` list stores FileSystemItem (can be File or Folder)
- All methods work **recursively** through the hierarchy
- No type checking needed!

---

## 📊 UML Diagrams

### Class Diagram

```
┌─────────────────────────────┐
│   <<interface>>             │
│   FileSystemItem            │
├─────────────────────────────┤
│ + ls(indent: int): void     │
│ + openAll(indent: int): void│
│ + getSize(): int            │
│ + getName(): String         │
│ + isFolder(): boolean       │
│ + cd(target: String): FSI   │
└─────────────────────────────┘
           △
           │
           │ implements
    ┌──────┴──────┐
    │             │
┌───▽────┐   ┌───▽────┐
│  File  │   │ Folder │◆────────┐
├────────┤   ├────────┤         │
│- name  │   │- name  │         │ 1..*
│- size  │   │        │         │
└────────┘   └────────┘         │
                 │               │
                 └───────────────┘
                   children: List<FileSystemItem>
```

**Relationships**:
1. **File IS-A FileSystemItem** (implements)
2. **Folder IS-A FileSystemItem** (implements)
3. **Folder HAS-A FileSystemItem** (composition - 1 to many)

### Sequence Diagram: openAll() Execution

```
Client          Root           File1          Docs          Resume
  │              │              │              │              │
  │─openAll()──>│              │              │              │
  │              │              │              │              │
  │              │─openAll()──>│              │              │
  │              │<─"file1"────│              │              │
  │              │              │              │              │
  │              │─openAll()──────────────────>│              │
  │              │              │              │─openAll()──>│
  │              │              │              │<─"resume"───│
  │              │<────────────────────────────│              │
  │              │              │              │              │
  │<─(complete)──│              │              │              │
```

**Flow Explanation**:
1. Client calls `openAll()` on Root
2. Root prints its name
3. Root loops through children, calling `openAll()` on each
4. If child is File → prints name and returns
5. If child is Folder → recursively calls `openAll()` on its children
6. Process continues until all leaf nodes are reached

---

## 💻 Implementation in Java

### Complete Working Code

```java
import java.util.*;

// Common Interface
interface FileSystemItem {
    void ls(int indent);
    void openAll(int indent);
    int getSize();
    String getName();
    boolean isFolder();
    FileSystemItem cd(String target);
}

// Leaf Node: File
class File implements FileSystemItem {
    private String name;
    private int size;
    
    public File(String name, int size) {
        this.name = name;
        this.size = size;
    }
    
    @Override
    public void ls(int indent) {
        System.out.println(" ".repeat(indent) + name);
    }
    
    @Override
    public void openAll(int indent) {
        System.out.println(" ".repeat(indent) + name);
    }
    
    @Override
    public int getSize() {
        return size;
    }
    
    @Override
    public String getName() {
        return name;
    }
    
    @Override
    public boolean isFolder() {
        return false;
    }
    
    @Override
    public FileSystemItem cd(String target) {
        return null; // Cannot cd into a file
    }
}

// Composite Node: Folder
class Folder implements FileSystemItem {
    private String name;
    private List<FileSystemItem> children;
    
    public Folder(String name) {
        this.name = name;
        this.children = new ArrayList<>();
    }
    
    public void add(FileSystemItem item) {
        children.add(item);
    }
    
    @Override
    public void ls(int indent) {
        String indentation = " ".repeat(indent);
        for (FileSystemItem child : children) {
            if (child.isFolder()) {
                System.out.println(indentation + "+ " + child.getName());
            } else {
                System.out.println(indentation + child.getName());
            }
        }
    }
    
    @Override
    public void openAll(int indent) {
        String indentation = " ".repeat(indent);
        System.out.println(indentation + name);
        
        for (FileSystemItem child : children) {
            child.openAll(indent + 4);
        }
    }
    
    @Override
    public int getSize() {
        int total = 0;
        for (FileSystemItem child : children) {
            total += child.getSize();
        }
        return total;
    }
    
    @Override
    public String getName() {
        return name;
    }
    
    @Override
    public boolean isFolder() {
        return true;
    }
    
    @Override
    public FileSystemItem cd(String target) {
        for (FileSystemItem child : children) {
            if (child.isFolder() && child.getName().equals(target)) {
                return child;
            }
        }
        return null; // Folder not found
    }
}

// Main Class
public class CompositePatternDemo {
    public static void main(String[] args) {
        // Create root folder
        Folder root = new Folder("Root");
        
        // Add files to root
        root.add(new File("file1.txt", 1));
        root.add(new File("file2.txt", 1));
        
        // Create docs folder
        Folder docs = new Folder("Docs");
        docs.add(new File("resume.pdf", 1));
        docs.add(new File("notes.txt", 1));
        
        // Create images folder
        Folder images = new Folder("Images");
        images.add(new File("photo.jpg", 1));
        
        // Add folders to root
        root.add(docs);
        root.add(images);
        
        // Test 1: openAll() - Expand entire hierarchy
        System.out.println("=== openAll() ===");
        root.openAll(0);
        
        // Test 2: ls() - List directory contents
        System.out.println("\n=== ls() ===");
        root.ls(0);
        
        // Test 3: ls() on specific folder
        System.out.println("\n=== ls() on Docs ===");
        docs.ls(0);
        
        // Test 4: cd() - Change directory
        System.out.println("\n=== cd() to Docs ===");
        FileSystemItem cwd = root.cd("Docs");
        if (cwd != null) {
            cwd.ls(0);
        } else {
            System.out.println("Could not cd into Docs");
        }
        
        // Test 5: getSize() - Calculate total size
        System.out.println("\n=== getSize() ===");
        System.out.println("Total size: " + root.getSize());
    }
}
```

### Expected Output

```
=== openAll() ===
Root
    file1.txt
    file2.txt
    Docs
        resume.pdf
        notes.txt
    Images
        photo.jpg

=== ls() ===
file1.txt
file2.txt
+ Docs
+ Images

=== ls() on Docs ===
resume.pdf
notes.txt

=== cd() to Docs ===
resume.pdf
notes.txt

=== getSize() ===
Total size: 5
```

### Code Walkthrough

#### 1. openAll() Method

**File's openAll()**:
```java
public void openAll(int indent) {
    System.out.println(" ".repeat(indent) + name);
}
```
- Simply prints the file name
- File is a leaf node - nothing more to expand

**Folder's openAll()**:
```java
public void openAll(int indent) {
    String indentation = " ".repeat(indent);
    System.out.println(indentation + name);
    
    for (FileSystemItem child : children) {
        child.openAll(indent + 4);
    }
}
```
- Prints folder name first
- Loops through all children
- Calls `openAll()` on each child (recursive)
- If child is File → prints name and returns
- If child is Folder → recursively expands its children

**Key Point**: No type checking! We just call `openAll()` on every child, regardless of type.

#### 2. getSize() Method

**File's getSize()**:
```java
public int getSize() {
    return size;
}
```
- Returns the file's size directly

**Folder's getSize()**:
```java
public int getSize() {
    int total = 0;
    for (FileSystemItem child : children) {
        total += child.getSize();
    }
    return total;
}
```
- Folder's size = sum of all children's sizes
- Loops through children, calling `getSize()` on each
- If child is File → returns its size
- If child is Folder → recursively calculates its total size
- Accumulates and returns total

**Execution Flow** (for Root):
```
Root.getSize()
├─ file1.getSize() → 1
├─ file2.getSize() → 1
├─ Docs.getSize()
│  ├─ resume.getSize() → 1
│  └─ notes.getSize() → 1
│  └─ returns 2
└─ Images.getSize()
   └─ photo.getSize() → 1
   └─ returns 1
Total: 1 + 1 + 2 + 1 = 5
```

#### 3. cd() Method

**Why do we need isFolder()?**

The `cd()` (change directory) operation should only work on folders, not files.

```java
public FileSystemItem cd(String target) {
    for (FileSystemItem child : children) {
        if (child.isFolder() && child.getName().equals(target)) {
            return child;
        }
    }
    return null;
}
```

- Loops through children
- Checks if child is a folder AND name matches target
- Returns the folder if found
- Returns null if not found

**File's cd()**:
```java
public FileSystemItem cd(String target) {
    return null; // Operation not supported
}
```
- Always returns null
- Cannot cd into a file

---

## 📐 Standard UML Diagram

### Generic Composite Pattern Structure

```
┌─────────────────────┐
│    Component        │
├─────────────────────┤
│ + operation(): void │
└─────────────────────┘
          △
          │
    ┌─────┴─────┐
    │           │
┌───▽───┐   ┌───▽────┐
│ Leaf  │   │Composite│◆────────┐
├───────┤   ├────────┤          │
│       │   │        │          │ 1..*
└───────┘   └────────┘          │
                │                │
                └────────────────┘
                  children: List<Component>
```

**Components**:

1. **Component** (Interface/Abstract Class)
   - Declares common operations
   - Both Leaf and Composite implement this

2. **Leaf**
   - Represents end objects (no children)
   - Implements Component operations
   - Example: File

3. **Composite**
   - Can have children (Leaf or Composite)
   - Has IS-A relationship with Component
   - Has HAS-A relationship with Component (1-to-many)
   - Implements operations by delegating to children
   - Example: Folder

**Key Relationships**:
- Leaf **IS-A** Component
- Composite **IS-A** Component
- Composite **HAS-MANY** Components

---

## 🔑 Key Concepts

### 1. Uniform Treatment

**The Core Principle**: Client treats individual objects and compositions uniformly.

```java
// Client doesn't care if it's a File or Folder
FileSystemItem item = root.cd("Docs");
item.openAll(0); // Works for both!
```

### 2. Recursive Composition

**Tree-like Structure**: Composites can contain other composites, forming a tree.

```
Root (Composite)
├── File (Leaf)
├── Docs (Composite)
│   ├── File (Leaf)
│   └── File (Leaf)
└── Images (Composite)
    └── File (Leaf)
```

### 3. Polymorphism Power

**No Type Checking Needed**:

```java
// Bad (without Composite Pattern)
for (Object item : list) {
    if (item instanceof File) {
        ((File) item).open();
    } else if (item instanceof Folder) {
        ((Folder) item).openAll();
    }
}

// Good (with Composite Pattern)
for (FileSystemItem item : children) {
    item.openAll(0); // Polymorphism!
}
```

### 4. Leaf vs Composite Behavior

| Aspect | Leaf (File) | Composite (Folder) |
|--------|-------------|-------------------|
| Children | None | Can have many |
| Operations | Direct implementation | Delegates to children |
| Example | Print name | Loop through children |
| Recursion | Base case | Recursive case |

---

## 🌍 Real-World Use Cases

### 1. File Systems ⭐

**Perfect Fit**: The classic example we've been using.

- **Leaf**: Files
- **Composite**: Folders/Directories
- **Operations**: ls, cd, openAll, getSize, delete, copy

### 2. UI Component Hierarchies

**Scenario**: Frontend frameworks (React, Angular, Vue)

```
Container (Composite)
├── Header (Composite)
│   ├── Logo (Leaf)
│   └── Navigation (Composite)
│       ├── MenuItem (Leaf)
│       └── MenuItem (Leaf)
├── Content (Composite)
│   ├── Article (Leaf)
│   └── Sidebar (Composite)
└── Footer (Leaf)
```

**Operations**: render(), update(), hide(), show()

### 3. Menu Systems

**Scenario**: Dropdown menus in applications

```
File (Menu)
├── New (MenuItem - Leaf)
├── Open (MenuItem - Leaf)
├── Save As (SubMenu - Composite)
│   ├── PDF (MenuItem - Leaf)
│   ├── Text (MenuItem - Leaf)
│   └── More Formats (SubMenu - Composite)
│       ├── XML (MenuItem - Leaf)
│       └── JSON (MenuItem - Leaf)
└── Exit (MenuItem - Leaf)
```

**Operations**: click(), display(), enable(), disable()

### 4. Organization Hierarchies

**Scenario**: Company structure

```
CEO (Composite)
├── CTO (Composite)
│   ├── Dev Manager (Composite)
│   │   ├── Developer (Leaf)
│   │   └── Developer (Leaf)
│   └── QA Manager (Composite)
│       └── Tester (Leaf)
└── CFO (Composite)
    └── Accountant (Leaf)
```

**Operations**: calculateSalary(), getEmployeeCount(), sendEmail()

### 5. Graphics Systems

**Scenario**: Drawing applications (Photoshop, Illustrator)

```
Canvas (Composite)
├── Circle (Leaf)
├── Rectangle (Leaf)
└── Group (Composite)
    ├── Line (Leaf)
    └── Triangle (Leaf)
```

**Operations**: draw(), move(), resize(), rotate()

### 6. Expression Trees

**Scenario**: Mathematical expressions

```
Expression: (2 + 3) * 4

       * (Composite)
      / \
     +   4 (Leaf)
    / \
   2   3 (Leaves)
```

**Operations**: evaluate(), toString()

---

## 🎓 Key Takeaways

### When to Use Composite Pattern

✅ **Use when**:
1. You need to represent **part-whole hierarchies**
2. You want clients to **treat individual objects and compositions uniformly**
3. Your problem naturally forms a **tree structure**
4. You have **leaf nodes** and **composite nodes**

❌ **Don't use when**:
1. Your structure is flat (no hierarchy)
2. Leaf and composite have very different behaviors
3. You don't need uniform treatment

### Advantages

1. **Simplifies Client Code**: No type checking needed
2. **Easy to Add New Components**: Open-Closed Principle
3. **Flexible Structure**: Can create complex hierarchies
4. **Polymorphism**: Leverage OOP principles

### Disadvantages

1. **Overly General Design**: Sometimes too generic
2. **Difficult to Restrict Components**: Hard to limit what can be added
3. **May Violate ISP**: Leaf nodes might have unsupported operations

### Design Principles Applied

1. **Open-Closed Principle**: Easy to add new component types
2. **Liskov Substitution**: Leaf and Composite are interchangeable
3. **Dependency Inversion**: Depend on Component abstraction
4. **Polymorphism**: Core to the pattern

---

## ❓ Interview Q&A

### Q1: What is the Composite Design Pattern? Explain with an example.

**Answer**:

The Composite Design Pattern is a structural pattern that composes objects into tree-like structures to represent part-whole hierarchies. It allows clients to treat individual objects and compositions of objects uniformly.

**Key Components**:
1. **Component**: Common interface for all objects
2. **Leaf**: End objects with no children
3. **Composite**: Objects that can have children

**Example**: File System
- Component = FileSystemItem interface
- Leaf = File (cannot have children)
- Composite = Folder (can contain files and folders)

**Benefit**: Client can call `openAll()` on both File and Folder without knowing which type it is. The polymorphism handles the rest.

---

### Q2: What problem does Composite Pattern solve?

**Answer**:

**Problem**: When dealing with hierarchical structures, we often need to:
1. Treat individual objects and groups of objects uniformly
2. Avoid type checking (instanceof) everywhere
3. Implement operations that work across the hierarchy

**Without Composite Pattern**:
```java
if (item instanceof File) {
    ((File) item).open();
} else if (item instanceof Folder) {
    ((Folder) item).openAll();
}
```
This violates Open-Closed Principle and makes code messy.

**With Composite Pattern**:
```java
item.openAll(); // Works for both!
```

**Solution**: Create a common interface that both leaf and composite implement. Client interacts only with the interface.

---

### Q3: Explain the difference between Leaf and Composite nodes.

**Answer**:

| Aspect | Leaf Node | Composite Node |
|--------|-----------|----------------|
| **Definition** | End point, no children | Can have children |
| **Example** | File | Folder |
| **Children** | Cannot have any | Can have Leaf or Composite |
| **Operations** | Direct implementation | Delegates to children |
| **Recursion** | Base case | Recursive case |
| **Relationship** | IS-A Component | IS-A + HAS-A Component |

**Example**:

**Leaf (File)**:
```java
public void openAll() {
    System.out.println(name); // Base case
}
```

**Composite (Folder)**:
```java
public void openAll() {
    System.out.println(name);
    for (FileSystemItem child : children) {
        child.openAll(); // Recursive case
    }
}
```

---

### Q4: What are the relationships in Composite Pattern?

**Answer**:

**Two Key Relationships**:

1. **IS-A Relationship** (Inheritance)
   - Leaf IS-A Component
   - Composite IS-A Component
   - Both implement the same interface

2. **HAS-A Relationship** (Composition)
   - Composite HAS-A list of Components
   - This list can contain both Leaf and Composite nodes

**Example**:
```java
class Folder implements FileSystemItem {  // IS-A
    private List<FileSystemItem> children; // HAS-A
}
```

**Similar Pattern**: Decorator Pattern also has IS-A + HAS-A relationships!

---

### Q5: How does recursion work in Composite Pattern?

**Answer**:

Composite Pattern heavily relies on **recursive tree traversal**.

**Example: openAll() execution**

```
Root.openAll()
├─ Print "Root"
├─ Call file1.openAll()
│  └─ Print "file1" (base case)
├─ Call Docs.openAll()
│  ├─ Print "Docs"
│  ├─ Call resume.openAll()
│  │  └─ Print "resume" (base case)
│  └─ Call notes.openAll()
│     └─ Print "notes" (base case)
└─ Done
```

**Key Points**:
1. **Base Case**: Leaf nodes simply execute and return
2. **Recursive Case**: Composite nodes call the same method on children
3. **Termination**: When all paths reach leaf nodes
4. **Similar to Tree Traversal**: Pre-order, in-order, post-order

---

### Q6: Why don't we need type checking in Composite Pattern?

**Answer**:

**Reason**: Polymorphism!

**Without Composite Pattern**:
```java
List<Object> items = new ArrayList<>();
for (Object item : items) {
    if (item instanceof File) {
        // Handle file
    } else if (item instanceof Folder) {
        // Handle folder
    }
}
```

**With Composite Pattern**:
```java
List<FileSystemItem> items = new ArrayList<>();
for (FileSystemItem item : items) {
    item.openAll(); // No type checking!
}
```

**How it works**:
1. Both File and Folder implement FileSystemItem
2. Client holds references to FileSystemItem
3. At runtime, correct implementation is called (polymorphism)
4. Client doesn't care about actual type

**Benefit**: Cleaner code, follows Open-Closed Principle

---

### Q7: How would you implement getSize() for a folder?

**Answer**:

**Concept**: Folder's size = sum of all children's sizes

**Implementation**:
```java
public int getSize() {
    int total = 0;
    for (FileSystemItem child : children) {
        total += child.getSize();
    }
    return total;
}
```

**Execution Flow**:
```
Root.getSize()
├─ file1.getSize() → 1
├─ file2.getSize() → 1
├─ Docs.getSize()
│  ├─ resume.getSize() → 1
│  └─ notes.getSize() → 1
│  └─ returns 2
└─ Total: 1 + 1 + 2 = 4
```

**Key Points**:
1. File's getSize() returns its size directly (base case)
2. Folder's getSize() recursively sums children (recursive case)
3. No type checking needed
4. Works for any depth of hierarchy

---

### Q8: What are real-world applications of Composite Pattern?

**Answer**:

**1. File Systems** ⭐
- Files (Leaf) and Folders (Composite)
- Operations: ls, cd, delete, copy, getSize

**2. UI Component Trees**
- Buttons, Labels (Leaf)
- Panels, Containers (Composite)
- Operations: render, update, hide

**3. Menu Systems**
- Menu Items (Leaf)
- Sub-menus (Composite)
- Operations: click, display, enable

**4. Organization Charts**
- Employees (Leaf)
- Managers (Composite)
- Operations: calculateSalary, getCount

**5. Graphics Systems**
- Shapes (Leaf)
- Groups (Composite)
- Operations: draw, move, resize

**6. Expression Trees**
- Numbers (Leaf)
- Operators (Composite)
- Operations: evaluate, toString

**Common Pattern**: Anywhere you have a tree-like hierarchy with uniform operations!

---

### Q9: What are the advantages and disadvantages of Composite Pattern?

**Answer**:

**Advantages** ✅:

1. **Simplifies Client Code**
   - No type checking needed
   - Uniform interface for all objects

2. **Easy to Extend**
   - Add new component types easily
   - Follows Open-Closed Principle

3. **Flexible Hierarchies**
   - Can create complex tree structures
   - Unlimited nesting

4. **Leverages Polymorphism**
   - Clean, object-oriented design

**Disadvantages** ❌:

1. **Overly General**
   - Sometimes too generic for specific needs
   - May add unnecessary complexity

2. **Difficult to Restrict**
   - Hard to limit what components can be added
   - Type safety concerns

3. **May Violate ISP**
   - Leaf nodes might have unsupported operations
   - Example: File's cd() returns null

4. **Runtime Errors**
   - Some operations might fail at runtime
   - Need careful error handling

---

### Q10: How is Composite Pattern different from Decorator Pattern?

**Answer**:

Both patterns have **IS-A + HAS-A** relationships, but serve different purposes.

| Aspect | Composite Pattern | Decorator Pattern |
|--------|------------------|-------------------|
| **Purpose** | Represent part-whole hierarchies | Add responsibilities dynamically |
| **Structure** | Tree (1-to-many) | Chain (1-to-1) |
| **Focus** | Uniform treatment of objects | Enhancing object behavior |
| **Children** | Multiple children | Single wrapped object |
| **Example** | File System | Coffee with add-ons |

**Composite**:
```java
class Folder implements FileSystemItem {
    List<FileSystemItem> children; // Many children
}
```

**Decorator**:
```java
class MilkDecorator implements Coffee {
    Coffee coffee; // Single wrapped object
}
```

**Key Difference**: Composite focuses on **structure** (tree), Decorator focuses on **behavior** (enhancement).

---

### Q11: When should you NOT use Composite Pattern?

**Answer**:

❌ **Don't use when**:

1. **Flat Structure**
   - No hierarchical relationship
   - Example: Simple list of items

2. **Very Different Behaviors**
   - Leaf and Composite have completely different operations
   - Forcing common interface doesn't make sense

3. **Type Restrictions Needed**
   - Need strict control over what can be added
   - Example: Folder can only contain Files, not other Folders

4. **Performance Critical**
   - Recursive calls might be too expensive
   - Need optimized, specific implementations

5. **Simple Use Case**
   - Overhead of pattern not justified
   - Direct implementation is clearer

**Example of Bad Use**:
```java
// Don't use Composite for a simple shopping cart
// Items don't form a hierarchy
List<Product> cart = new ArrayList<>();
```

---

### Q12: How does Composite Pattern follow SOLID principles?

**Answer**:

**1. Single Responsibility Principle (SRP)** ✅
- Each class has one responsibility
- File manages file data
- Folder manages children

**2. Open-Closed Principle (OCP)** ✅
- Easy to add new component types
- No need to modify existing code
- Example: Add ImageFile, VideoFile

**3. Liskov Substitution Principle (LSP)** ✅
- Leaf and Composite are interchangeable
- Client can use FileSystemItem reference

**4. Interface Segregation Principle (ISP)** ⚠️
- **Potential Violation**: Leaf might have unsupported operations
- Example: File's cd() returns null
- **Solution**: Use default methods or separate interfaces

**5. Dependency Inversion Principle (DIP)** ✅
- Client depends on FileSystemItem abstraction
- Not on concrete File or Folder classes

**Overall**: Composite Pattern aligns well with SOLID, with minor ISP considerations.

---

## 📝 Summary

The **Composite Design Pattern** is a powerful structural pattern for handling tree-like hierarchies. It allows you to treat individual objects (Leaf) and compositions (Composite) uniformly through a common interface.

**Key Points**:
- Perfect for hierarchical structures (files, UI, menus)
- Eliminates type checking through polymorphism
- Uses recursion for tree traversal
- Has both IS-A and HAS-A relationships
- Simplifies client code significantly

**Remember**: If you see a tree structure with leaf and composite nodes, think Composite Pattern!

---

**Happy Learning! 🚀**
