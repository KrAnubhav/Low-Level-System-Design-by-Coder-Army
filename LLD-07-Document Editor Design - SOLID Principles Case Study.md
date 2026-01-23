# LLD-07: Document Editor Design - SOLID Principles Case Study 📝

## Introduction

**Aaj ka topic**: Ab tak jo bhi theory padhi hai (SOLID Design Principles, OOP Principles), usko hum ek **real-world example** mein apply karenge.

**Kya seekhenge:**
- Ek LLD problem ko kaise approach karte hain
- Bad design se start karke better design tak kaise pahunchte hain
- SOLID principles ko practically kaise use karte hain

---

## Problem Statement

**Banana hai**: Ek **Document Editor** (jaise Google Docs)

### Features Required

1. **Text** add kar sakte hain
2. **Images** add kar sakte hain
3. Document ko **render** kar sakte hain
4. Document ko **save** kar sakte hain (file mein)

### Future Requirements (Scalability)

- **Tables** support karna chahiye
- **Videos** support karna chahiye
- **Fonts, New Line, Tab, Space** support karna chahiye

**Goal**: Ek scalable design banana jisme naye features easily add ho sakein.

---

## Design Approaches

### Top-Down vs Bottom-Up

**Top-Down Approach:**
- Sabse pehle top-most object banate hain
- Phir uske andar chhote-chhote objects create karte hain
- Example: Pehle Document Editor banaenge, phir uske components

**Bottom-Up Approach:** ✅ (Hum yahi use karenge)
- Pehle sabse chhote objects banate hain
- Phir unki dependencies banate hain
- Phir bade objects create karte hain
- **Zyada tar developers yahi approach use karte hain**

---

## Version 1: Bad Design ❌

### Class Diagram

```
┌─────────────────────────────────┐
│     DocumentEditor              │
├─────────────────────────────────┤
│ - elements: vector<string>      │
│ - renderedDocument: string      │
├─────────────────────────────────┤
│ + addText(text: string)         │
│ + addImage(path: string)        │
│ + renderDocument(): string      │
│ + saveToFile()                  │
└─────────────────────────────────┘
```

### Design Decisions

**Element storage:**
- Text aur Image dono ko `vector<string>` mein store karenge
- Text → directly string
- Image → image ka path (string)

**Kyu ek hi list?**
- Agar alag-alag lists rakhe (text ki alag, image ki alag)
- Toh user ne kis order mein add kiya, wo determine karna mushkil hoga

### Java Implementation (Bad Design)

```java
import java.util.*;
import java.io.*;

class DocumentEditor {
    private List<String> elements;
    private String renderedDocument;
    
    public DocumentEditor() {
        this.elements = new ArrayList<>();
        this.renderedDocument = "";
    }
    
    // Add text element
    public void addText(String text) {
        elements.add(text);
    }
    
    // Add image element
    public void addImage(String imagePath) {
        elements.add(imagePath);
    }
    
    // Render document
    public String renderDocument() {
        if (!renderedDocument.isEmpty()) {
            return renderedDocument;
        }
        
        StringBuilder result = new StringBuilder();
        
        for (String element : elements) {
            // Hack: Check if it's an image
            if (element.length() > 4 && 
                (element.endsWith(".jpg") || element.endsWith(".png"))) {
                // Render as image
                result.append("[Image: ").append(element).append("]\n");
            } else {
                // Render as text
                result.append(element).append("\n");
            }
        }
        
        renderedDocument = result.toString();
        return renderedDocument;
    }
    
    // Save to file
    public void saveToFile() {
        try {
            FileWriter writer = new FileWriter("document.txt");
            String content = renderDocument();
            writer.write(content);
            writer.close();
            System.out.println("Document saved successfully!");
        } catch (IOException e) {
            System.out.println("Unable to open file");
        }
    }
}

// Client code
public class Main {
    public static void main(String[] args) {
        DocumentEditor editor = new DocumentEditor();
        
        editor.addText("Hello World");
        editor.addImage("photo.jpg");
        editor.addText("This is a Document Editor");
        
        // Print rendered document
        System.out.println(editor.renderDocument());
        
        // Save to file
        editor.saveToFile();
    }
}
```

**Output:**
```
Hello World
[Image: photo.jpg]
This is a Document Editor

Document saved successfully!
```

---

## Problems with Bad Design ❌

### 1. **Single Responsibility Principle (SRP) Violation**

`DocumentEditor` class **multiple responsibilities** handle kar rahi hai:
- Text aur Image add karna
- Document render karna
- File mein save karna

**Multiple reasons to change:**
- Database storage logic change → modify `DocumentEditor`
- Invoice format change → modify `DocumentEditor`
- Price calculation change → modify `DocumentEditor`

### 2. **Open/Closed Principle (OCP) Violation**

Naye element types add karne ke liye:
- `DocumentEditor` class ko modify karna padega
- Naya method add karna padega
- Existing code change karna padega

**Example:** Video support add karna ho toh:
```java
public void addVideo(String videoPath) {
    elements.add(videoPath);
}
```
Yeh OCP violate karta hai!

### 3. **Other Principles**

- **LSP**: Yahan applicable nahi (inheritance nahi hai)
- **ISP**: Yahan applicable nahi (interfaces nahi hain)
- **DIP**: Yahan applicable nahi (abstraction nahi hai)

---

## Version 2: Improved Design ✅

### Key Improvements

1. **Separate Document Elements** - Polymorphism use karenge
2. **Separate Rendering Logic** - Document class banayenge
3. **Separate Persistence Logic** - Persistence hierarchy banayenge

### Class Diagram

```
┌──────────────────────┐
│   <<abstract>>       │
│  DocumentElement     │
├──────────────────────┤
│ + render(): string   │
└──────────────────────┘
         △
         │
    ┌────┴────┬─────────────┬──────────────┐
    │         │             │              │
┌───┴────┐ ┌─┴────────┐ ┌──┴─────────┐ ┌──┴────────┐
│ Text   │ │  Image   │ │  NewLine   │ │ TabSpace  │
│Element │ │ Element  │ │  Element   │ │  Element  │
├────────┤ ├──────────┤ ├────────────┤ ├───────────┤
│+render │ │ +render  │ │  +render   │ │  +render  │
└────────┘ └──────────┘ └────────────┘ └───────────┘

┌──────────────────────────────┐
│        Document              │
├──────────────────────────────┤
│ - elements: List<DocElement> │
├──────────────────────────────┤
│ + addElement(el)             │
│ + render(): string           │
└──────────────────────────────┘
         △
         │ has-a (1..*)
         │
┌──────────────────────┐
│   <<abstract>>       │
│    Persistence       │
├──────────────────────┤
│ + save(content)      │
└──────────────────────┘
         △
         │
    ┌────┴────┐
    │         │
┌───┴──────┐ ┌┴────────┐
│  File    │ │   DB    │
│ Storage  │ │ Storage │
├──────────┤ ├─────────┤
│ + save() │ │ + save()│
└──────────┘ └─────────┘

┌──────────────────────────────┐
│     DocumentEditor           │
├──────────────────────────────┤
│ - doc: Document              │
│ - storage: Persistence       │
│ - renderedDoc: string        │
├──────────────────────────────┤
│ + addText(text)              │
│ + addImage(path)             │
│ + renderDocument()           │
│ + saveDocument()             │
└──────────────────────────────┘
```

### Java Implementation (Improved Design)

```java
import java.util.*;
import java.io.*;

// ============= Document Element Hierarchy =============

abstract class DocumentElement {
    public abstract String render();
}

class TextElement extends DocumentElement {
    private String text;
    
    public TextElement(String text) {
        this.text = text;
    }
    
    @Override
    public String render() {
        return text;
    }
}

class ImageElement extends DocumentElement {
    private String imagePath;
    
    public ImageElement(String imagePath) {
        this.imagePath = imagePath;
    }
    
    @Override
    public String render() {
        return "[Image: " + imagePath + "]";
    }
}

class NewLineElement extends DocumentElement {
    @Override
    public String render() {
        return "\n";
    }
}

class TabSpaceElement extends DocumentElement {
    @Override
    public String render() {
        return "\t";
    }
}

// ============= Document Class =============

class Document {
    private List<DocumentElement> elements;
    
    public Document() {
        this.elements = new ArrayList<>();
    }
    
    public void addElement(DocumentElement element) {
        elements.add(element);
    }
    
    public String render() {
        StringBuilder result = new StringBuilder();
        
        // Delegation: Har element khud ko render karta hai
        for (DocumentElement el : elements) {
            result.append(el.render());
        }
        
        return result.toString();
    }
}

// ============= Persistence Hierarchy =============

abstract class Persistence {
    public abstract void save(String content);
}

class FileStorage extends Persistence {
    @Override
    public void save(String content) {
        try {
            FileWriter writer = new FileWriter("document.txt");
            writer.write(content);
            writer.close();
            System.out.println("Document saved to file successfully!");
        } catch (IOException e) {
            System.out.println("Unable to save file");
        }
    }
}

class DBStorage extends Persistence {
    @Override
    public void save(String content) {
        // Database saving logic
        System.out.println("Saving to database...");
        // SQL connection, query execution, etc.
    }
}

// ============= Document Editor =============

class DocumentEditor {
    private Document doc;
    private Persistence storage;
    private String renderedDocument;
    
    public DocumentEditor(Document doc, Persistence storage) {
        this.doc = doc;
        this.storage = storage;
        this.renderedDocument = "";
    }
    
    // Delegation: Document ko delegate karta hai
    public void addText(String text) {
        doc.addElement(new TextElement(text));
    }
    
    public void addImage(String imagePath) {
        doc.addElement(new ImageElement(imagePath));
    }
    
    public void addNewLine() {
        doc.addElement(new NewLineElement());
    }
    
    public void addTabSpace() {
        doc.addElement(new TabSpaceElement());
    }
    
    // Delegation: Document ko delegate karta hai
    public String renderDocument() {
        if (!renderedDocument.isEmpty()) {
            return renderedDocument;
        }
        
        renderedDocument = doc.render();
        return renderedDocument;
    }
    
    // Delegation: Persistence ko delegate karta hai
    public void saveDocument() {
        storage.save(renderDocument());
    }
}

// ============= Client Code =============

public class Main {
    public static void main(String[] args) {
        // Create document and storage
        Document doc = new Document();
        Persistence storage = new FileStorage();
        
        // Create editor
        DocumentEditor editor = new DocumentEditor(doc, storage);
        
        // Add elements
        editor.addText("Hello World");
        editor.addNewLine();
        editor.addTabSpace();
        editor.addText("This is a real world document editor");
        editor.addNewLine();
        editor.addImage("photo.jpg");
        
        // Render and print
        System.out.println(editor.renderDocument());
        
        // Save document
        editor.saveDocument();
    }
}
```

**Output:**
```
Hello World

	This is a real world document editor
[Image: photo.jpg]

Document saved to file successfully!
```

---

## SOLID Principles Analysis ✅

### 1. Single Responsibility Principle (SRP) ✅

**Har class ek hi responsibility handle karti hai:**

- **`DocumentElement`**: Sirf rendering ka contract define karta hai
- **`TextElement`, `ImageElement`**: Apne aap ko render karna
- **`Document`**: Elements ki list manage karna
- **`Persistence`**: Storage ka contract define karna
- **`FileStorage`, `DBStorage`**: Specific storage implementation
- **`DocumentEditor`**: Client se interact karna, delegation karna

### 2. Open/Closed Principle (OCP) ✅

**Naye features easily add kar sakte hain:**

```java
// Naya element type add karna - NO modification needed!
class VideoElement extends DocumentElement {
    private String videoPath;
    
    public VideoElement(String videoPath) {
        this.videoPath = videoPath;
    }
    
    @Override
    public String render() {
        return "[Video: " + videoPath + "]";
    }
}

// Naya storage type add karna - NO modification needed!
class CloudStorage extends Persistence {
    @Override
    public void save(String content) {
        System.out.println("Saving to cloud...");
    }
}
```

**Koi bhi existing class modify nahi karna pada!**

### 3. Liskov Substitution Principle (LSP) ✅

**Subclasses apne parent classes ko replace kar sakti hain:**

```java
// Document expects DocumentElement
List<DocumentElement> elements = new ArrayList<>();

// Koi bhi subtype pass kar sakte hain
elements.add(new TextElement("Hello"));     // ✓
elements.add(new ImageElement("pic.jpg"));  // ✓
elements.add(new NewLineElement());         // ✓

// Sab ka render() method properly work karega
for (DocumentElement el : elements) {
    el.render();  // Works for all!
}
```

### 4. Interface Segregation Principle (ISP) ✅

**Chhote, focused interfaces:**

- `DocumentElement` interface mein sirf `render()` hai
- `Persistence` interface mein sirf `save()` hai
- Koi unnecessary methods nahi hain

### 5. Dependency Inversion Principle (DIP) ✅

**High-level modules abstractions par depend karte hain:**

```
DocumentEditor → Persistence (abstraction) ← FileStorage/DBStorage
                                             (concrete implementations)

Document → DocumentElement (abstraction) ← TextElement/ImageElement
                                          (concrete implementations)
```

---

## Version 3: Further Improvements 🚀

### Problem Identified

**DocumentEditor ke paas bahut saare methods hain:**
- `addText()`, `addImage()`, `renderDocument()`, `saveDocument()`
- Yeh sab methods ko DocumentEditor ko **knowledge** hai
- Agar koi method change ho, toh DocumentEditor bhi change hoga

**Principle of Least Knowledge violation:**
- DocumentEditor ko bahut zyada classes ke baare mein pata hai
- Tight coupling ho rahi hai

### Solution: Separate Renderer

```
┌──────────────────────┐
│  DocumentRenderer    │
├──────────────────────┤
│ - doc: Document      │
├──────────────────────┤
│ + render(): string   │
└──────────────────────┘
```

**Document se render() method hata diya:**
- Document sirf CRUD operations karega (Create, Read, Update, Delete)
- DocumentRenderer rendering handle karega

### Updated Design

```
┌──────────────────────┐
│     Document         │
├──────────────────────┤
│ - elements: List     │
├──────────────────────┤
│ + addElement()       │
│ + getElements()      │ ← New method
└──────────────────────┘

┌──────────────────────┐
│  DocumentRenderer    │
├──────────────────────┤
│ - doc: Document      │
├──────────────────────┤
│ + render(): string   │
└──────────────────────┘

┌──────────────────────┐
│  DocumentEditor      │
├──────────────────────┤
│ + addText()          │
│ + addImage()         │
└──────────────────────┘
```

**Client ka responsibility:**
- Document create karna
- DocumentEditor se edit karna
- DocumentRenderer se render karna
- Persistence se save karna

---

## Final Thoughts 💭

### Trade-offs

**Koi bhi design perfect nahi hota:**
- Hamesha trade-offs hote hain
- SOLID principles **guidelines** hain, **laws** nahi
- Context ke according decisions lena padta hai

### Interview Tips

1. **Bad design se start karo** - Improvements dikhane ke liye
2. **SOLID principles apply karo** - Step by step
3. **Trade-offs discuss karo** - Kya compromise kar rahe ho
4. **Interviewer se feedback lo** - Design ko improve karte raho

### Key Learnings

✅ **Delegation** use karo - Har class apna kaam delegate kare  
✅ **Abstraction** use karo - Polymorphism ke liye  
✅ **Single Responsibility** follow karo - Ek class ek kaam  
✅ **Open/Closed** follow karo - Extension easy, modification hard  

---

## Practice Exercise 💪

**Try implementing:**

1. **Bold/Italic/Underline elements** add karo
2. **Table element** implement karo
3. **Multiple file formats** support karo (PDF, DOCX)
4. **Undo/Redo functionality** add karo

---

## Summary

| Version | Design Quality | SOLID Compliance |
|---------|---------------|------------------|
| Version 1 | ❌ Bad | 0/5 principles |
| Version 2 | ✅ Good | 5/5 principles |
| Version 3 | ✅ Better | 5/5 + Least Knowledge |

**Remember**: LLD mein perfect design nahi hota, **best possible design** hota hai given constraints ke saath!

---

*Next: Design Patterns (Strategy, Factory, Singleton, etc.)*
