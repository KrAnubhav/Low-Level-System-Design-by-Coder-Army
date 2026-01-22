# Document Editor Design: Applying SOLID Principles (LLD Case Study)

## Introduction

This lecture demonstrates how to approach Low Level Design (LLD) interviews by designing a **Document Editor application** (like Google Docs). The session covers iterating from a **bad design to an improved design** by applying SOLID principles and OOP concepts.

**Key Learning Objectives:**

- How to approach LLD problems in interviews
- Iterative design improvement process
- Practical application of SOLID principles
- Creating scalable, extensible architectures

---

## Problem Statement

### Requirements

Design a **Document Editor** that supports:

- Adding **text** elements
- Inserting **images** (via file path)
- **Rendering** the document (display content)
- **Saving** document to a file

### Scalability Requirements

- Design should be **extensible** to support future elements:
    - Tables
    - Videos
    - Fonts, new lines, tabs, spaces
    - Other editing features

**Current Scope:** Focus on Text and Image elements, but design for future extensibility.

---

## Design Approaches

### Top-Down vs Bottom-Up Approach

**Top-Down Approach:**

- Start with the **topmost/main object** of the application
- Then create smaller objects and their dependencies
- Work from high-level to low-level components

**Bottom-Up Approach:** ⭐ (Preferred for most LLD interviews)

- Start by creating **smallest objects** first
- Build their dependencies
- Then create larger objects that use them
- Work from low-level to high-level components

> Note: Most developers prefer bottom-up for LLD interviews. Some questions may benefit from top-down, but that depends on the specific problem.
> 

---

## Design Iteration 1: Bad Design (Monolithic Class)

### Architecture

```
┌─────────────────────────────────────┐
│      DocumentEditor                 │
├─────────────────────────────────────┤
│ - vector<string> documentElements   │
│ - string renderedDocument           │
├─────────────────────────────────────┤
│ + addText(string text)              │
│ + addImage(string path)             │
│ + renderDocument(): string          │
│ + saveToFile()                      │
└─────────────────────────────────────┘
```

### Implementation Details

**Class: DocumentEditor**

**Attributes:**

- `vector<string> documentElements` - Stores both text and image paths
- `string renderedDocument` - Caches rendered output to avoid re-rendering

**Methods:**

1. **addText(string text):** Adds text to the elements list
2. **addImage(string path):** Adds image path to the elements list
3. **renderDocument():** Loops through elements, determines if text/image, renders accordingly
4. **saveToFile():** Calls renderDocument(), writes to file using ofstream

### Code Walkthrough (Java equivalent)

```java
class DocumentEditor {
    private List<String> documentElements;
    private String renderedDocument;

    public void addText(String text) {
        documentElements.add(text);
    }

    public void addImage(String path) {
        documentElements.add(path);
    }

    public String renderDocument() {
        if (!renderedDocument.isEmpty()) {
            return renderedDocument;
        }

        StringBuilder result = new StringBuilder();
        for (String element : documentElements) {
            // Hack: Check if image by file extension
            if (element.length() > 4 &&
                (element.endsWith(".jpg") || element.endsWith(".png"))) {
                result.append("[Image: ").append(element).append("]\n");
            } else {
                result.append(element).append("\n");
            }
        }
        renderedDocument = result.toString();
        return renderedDocument;
    }

    public void saveToFile() {
        // Write renderedDocument to file
        // File handling logic
    }
}
```

### Problems with This Design ❌

1. **Single Responsibility Principle (SRP) Violation:**
    - DocumentEditor handles: storing elements, rendering logic, file operations
    - Multiple responsibilities in one class
2. **Open/Closed Principle (OCP) Violation:**
    - To add new element types (video, table), must **modify** existing code
    - Uses hacky string matching (`.jpg`, `.png`) to identify types
    - Not **open for extension**, requires **modification**
3. **Liskov Substitution Principle (LSP) Issues:**
    - No proper inheritance hierarchy
    - Cannot substitute different element types polymorphically
4. **Poor Maintainability:**
    - Mixing text and images in a string list
    - Rendering logic hardcoded with if-else checks
    - Adding new element types requires modifying renderDocument()
5. **Business Logic vs Architecture:**
    - LLD interviews focus on **code optimization and architecture**, not business logic details
    - Business logic can change, but architecture should be solid

---

## Design Iteration 2: Introducing DocumentElement Abstraction

### Key Improvement: Separation of Concerns

**Problem Identified:** Everything in one class violates SRP

**Solution:** Create separate classes for different responsibilities

### New Architecture

```
┌────────────────────────────┐
│   DocumentElement          │ (Abstract/Interface)
│   (Interface)              │
├────────────────────────────┤
│ + render(): string         │
└────────────┬───────────────┘
             │
      ┌──────┴──────┐
      │             │
┌─────▼────┐  ┌────▼─────┐
│   Text   │  │  Image   │
├──────────┤  ├──────────┤
│ - content│  │ - path   │
├──────────┤  ├──────────┤
│+ render()│  │+ render()│
└──────────┘  └──────────┘
      │             │
      └─────┬───────┘
            │
      ┌─────▼─────────────────────────┐
      │       Document                │
      ├───────────────────────────────┤
      │ - List<DocumentElement>       │
      │   elements                    │
      ├───────────────────────────────┤
      │ + addElement(element)         │
      │ + render(): string            │
      └───────────────────────────────┘
```

### Implementation

**Interface: DocumentElement**

```java
interface DocumentElement {
    String render();
}
```

**Class: Text**

```java
class Text implements DocumentElement {
    private String content;

    public Text(String content) {
        this.content = content;
    }

    @Override
    public String render() {
        return content + "\n";
    }
}
```

**Class: Image**

```java
class Image implements DocumentElement {
    private String path;

    public Image(String path) {
        this.path = path;
    }

    @Override
    public String render() {
        return "[Image: " + path + "]\n";
        // Or use actual image rendering library
    }
}
```

**Class: Document**

```java
class Document {
    private List<DocumentElement> elements;

    public Document() {
        elements = new ArrayList<>();
    }

    public void addElement(DocumentElement el) {
        elements.add(el);
    }

    public String render() {
        StringBuilder result = new StringBuilder();
        for (DocumentElement el : elements) {
            result.append(el.render()); // Delegation!
        }
        return result.toString();
    }
}
```

### Key Concepts Applied

**1. Delegation Pattern:**

- Document **delegates** rendering responsibility to each element
- Document doesn’t know HOW to render, it just asks elements to render themselves
- Each element knows its own rendering logic

**2. Polymorphism:**

- Document works with `DocumentElement` interface
- Can add any element type without modifying Document class
- Runtime polymorphism through `render()` method

**PlainText Diagram:**

```
Document
   │
   ├─→ Text.render()
   ├─→ Image.render()
   ├─→ Text.render()
   └─→ Video.render() (future)

Each element handles its own rendering
Document just orchestrates the calls
```

### Advantages ✅

1. **SRP Followed:**
    - Text class: handles text rendering
    - Image class: handles image rendering
    - Document class: manages elements collection
2. **OCP Followed:**
    - Can add new element types (Video, Table) **without modifying** existing code
    - Just create new class implementing DocumentElement
    - **Open for extension, closed for modification**
3. **Better Maintainability:**
    - Clear separation of concerns
    - Each class has single responsibility
    - Easy to test individual components

---

## Design Iteration 3: Adding DocumentEditor Layer

### Problem

Still missing coordinating layer between client and document

### Complete Architecture

```
Client Code
    │
    ↓
DocumentEditor
    │
    ├─→ Document (1 to Many)
    │       │
    │       └─→ DocumentElement (Interface)
    │               ├─→ Text
    │               ├─→ Image
    │               └─→ (Future: Video, Table, etc.)
    │
    └─→ FileManager (for saving)
```

**Relationships:**

- DocumentEditor **HAS-A** Document (Composition)
- Document **HAS-MANY** DocumentElements (1-to-Many relationship)
- Text, Image **IS-A** DocumentElement (Inheritance)

### Updated Implementation

**Class: DocumentEditor**

```java
class DocumentEditor {
    private Document document;

    public DocumentEditor() {
        document = new Document();
    }

    public void addText(String content) {
        document.addElement(new Text(content));
    }

    public void addImage(String path) {
        document.addElement(new Image(path));
    }

    public String renderDocument() {
        return document.render();
    }

    public void saveToFile(String filename) {
        String content = document.render();
        // File writing logic
        try (FileWriter writer = new FileWriter(filename)) {
            writer.write(content);
            System.out.println("Document saved successfully");
        } catch (IOException e) {
            System.out.println("Unable to open file");
        }
    }
}
```

**Client Code Example:**

```java
public class Main {
    public static void main(String[] args) {
        DocumentEditor editor = new DocumentEditor();

        editor.addText("Hello World");
        editor.addImage("/path/to/image.jpg");
        editor.addText("This is a document editor");

        // Render to console
        System.out.println(editor.renderDocument());

        // Save to file
        editor.saveToFile("document.txt");
    }
}
```

**Output:**

```
Hello World
[Image: /path/to/image.jpg]
This is a document editor

Document saved successfully
```

---

## SOLID Principles Applied

### 1. Single Responsibility Principle (SRP) ✅

**Definition:** A class should have only one reason to change

**Application:**

- **Text class:** Only handles text rendering
- **Image class:** Only handles image rendering
- **Document class:** Only manages element collection and orchestrates rendering
- **DocumentEditor:** Coordinates operations, provides API to client

**Trade-off Discussion:**

- Some may argue Document knows about multiple element types
- **Counter:** Document only delegates; it doesn’t implement rendering logic
- Each class has **one primary responsibility**

### 2. Open/Closed Principle (OCP) ✅

**Definition:** Classes should be open for extension, closed for modification

**Application:**

- To add `Video` or `Table` elements:
    - Create new class implementing `DocumentElement`
    - **No modification** to Document, Text, or Image classes
    - Just add new functionality by extension

**Example: Adding Video Support**

```java
class Video implements DocumentElement {
    private String videoPath;

    public Video(String videoPath) {
        this.videoPath = videoPath;
    }

    @Override
    public String render() {
        return "[Video: " + videoPath + "]\n";
    }
}

// In DocumentEditor - add new method
public void addVideo(String path) {
    document.addElement(new Video(path));
}
```

**No changes needed to:**

- DocumentElement interface
- Document class
- Text/Image classes

### 3. Liskov Substitution Principle (LSP) ✅

**Definition:** Objects of a superclass should be replaceable with objects of subclasses without breaking functionality

**Application:**

- Any `DocumentElement` implementation can be used wherever `DocumentElement` is expected
- Document.render() works correctly with Text, Image, Video, or any future implementation
- All subtypes honor the contract defined by DocumentElement interface

**PlainText Example:**

```
DocumentElement element = new Text("Hello");
document.addElement(element);  // Works

element = new Image("pic.jpg");
document.addElement(element);  // Works

element = new Video("vid.mp4");
document.addElement(element);  // Works

All behave as DocumentElement, honor render() contract
```

### 4. Interface Segregation Principle (ISP) ✅

**Definition:** Clients should not be forced to depend on interfaces they don’t use

**Application:**

- DocumentElement interface has only `render()` method
- No unnecessary methods that not all elements need
- If some elements need additional capabilities, create separate interfaces

**Future Extension Example:**

```java
interface Editable {
    void edit(String newContent);
}

interface Resizable {
    void resize(int width, int height);
}

class Text implements DocumentElement, Editable {
    // Implements render() and edit()
}

class Image implements DocumentElement, Resizable {
    // Implements render() and resize()
}
```

### 5. Dependency Inversion Principle (DIP) ✅

**Definition:** High-level modules should not depend on low-level modules. Both should depend on abstractions.

**Application:**

- **Document** (high-level) depends on **DocumentElement** (abstraction), not concrete Text/Image classes
- Can inject any DocumentElement implementation
- Loose coupling between Document and specific element types

**PlainText Dependency Flow:**

```
Document (High-level)
    ↓ depends on
DocumentElement (Abstraction)
    ↑ implemented by
Text, Image, Video (Low-level)

NOT: Document → Text, Image (concrete classes)
```

---

## Complete UML Diagram

```
┌────────────────────────┐
│  <<interface>>         │
│  DocumentElement       │
├────────────────────────┤
│ + render(): String     │
└───────────┬────────────┘
            △
            │ implements
   ┌────────┼────────┐
   │        │        │
┌──┴───┐ ┌─┴────┐ ┌─┴────────┐
│ Text │ │Image │ │Video     │
├──────┤ ├──────┤ ├──────────┤
│-text │ │-path │ │-videoPath│
├──────┤ ├──────┤ ├──────────┤
│render│ │render│ │render()  │
└──────┘ └──────┘ └──────────┘
   △        △         △
   │        │         │
   └────────┼─────────┘
            │ has-many
    ┌───────┴────────┐
    │   Document     │
    ├────────────────┤
    │ - elements: []  │
    ├────────────────┤
    │ + addElement() │
    │ + render()     │
    └────────┬───────┘
             △
             │ has-a
    ┌────────┴──────────┐
    │ DocumentEditor    │
    ├───────────────────┤
    │ - document        │
    ├───────────────────┤
    │ + addText()       │
    │ + addImage()      │
    │ + renderDocument()│
    │ + saveToFile()    │
    └───────────────────┘
             △
             │ uses
        Client Code
```

---

## Key Design Patterns Used

### 1. Strategy Pattern

- Different rendering strategies for different element types
- Each element encapsulates its rendering algorithm

### 2. Composite Pattern (Implied)

- Document contains multiple DocumentElements
- Can treat individual elements and collections uniformly

### 3. Delegation Pattern

- Document delegates rendering to individual elements
- Each element knows how to render itself

---

## Interview Approach Summary

### Step-by-Step Process

**1. Understand Requirements:**

- What features are needed?
- What should be extensible?
- What are current scope and future scope?

**2. Start with Bad Design:**

- Identify all components
- Create simple, working solution
- Don’t worry about principles initially

**3. Identify Problems:**

- Which SOLID principles are violated?
- What happens when requirements change?
- Where is tight coupling?

**4. Iterative Improvement:**

- Extract responsibilities into separate classes
- Introduce abstractions (interfaces)
- Apply delegation and polymorphism
- Ensure extensibility

**5. Validate Against SOLID:**

- Check each principle
- Explain trade-offs
- Justify design decisions

### Common Interview Discussion Points

**Q: Is Document violating SRP since it knows about multiple elements?**
A: No, Document only delegates. It doesn’t implement rendering logic. Delegation is a responsibility, but it’s a single responsibility.

**Q: Why not have separate lists for Text and Image?**
A: Would lose ordering information. One list of DocumentElement maintains insertion order and allows polymorphism.

**Q: What if we need element-specific operations?**
A: Can use visitor pattern or add type-specific methods to interface (with ISP considerations).

---

## Important Analogies from Lecture

### Real-World Analogy: Document as Orchestra Conductor

- **Document** = Conductor
- **DocumentElements** = Musicians
- Conductor doesn’t play instruments, just coordinates
- Each musician knows their own part (rendering)
- Adding new musician (element type) doesn’t change conductor’s role

### Code Evolution Analogy

```
Bad Design → Kitchen Chaos
Everything in one pot (DocumentEditor)

Good Design → Restaurant Kitchen
- Chef (DocumentEditor)
- Station chefs (Document)
- Specialized cooks (Text, Image, etc.)
Each knows their responsibility
```

---

## Key Takeaways

1. **Start Simple, Iterate:** Begin with working solution, then improve based on principles
2. **SOLID Principles are Guidelines:** Not strict laws; understand trade-offs and context
3. **Business Logic ≠ Architecture:** LLD focuses on **extensibility and maintainability**, not algorithm optimization
4. **Delegation is Powerful:** Separating “what to do” from “how to do it” enables flexibility
5. **Interfaces Enable Extensibility:** Program to interfaces, not implementations
6. **Bottom-Up Works Well:** Build small components first, then compose them
7. **OCP is Crucial:** Adding features should not require modifying existing, tested code
8. **Think Beyond Current Requirements:** Design for change and future extensibility
9. **Clear Responsibilities:** Each class should answer: “What is my one job?”
10. **Polymorphism Eliminates Conditionals:** Replace type-checking if-else with polymorphic method calls

---

## Practice Exercise

**Try This:** Extend the design to support:

1. **Table element** with rows and columns
2. **Bold/Italic formatting** for text
3. **Export to PDF** (in addition to text file)
4. **Undo/Redo** functionality

**Hint:** Consider which principles guide each addition and how to avoid modifying existing classes.
