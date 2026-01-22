### **Introduction to Low Level Design (LLD)**

## Why LLD Matters

Low Level Design is essential for building real-world applications like Swiggy, Zomato, Ola, and Uber. It bridges the gap between solving DSA problems on LeetCode and building scalable production systems.

**Key Benefits:**

- Essential for FAANG companies and top startups
- Foundation for understanding High Level Design (HLD)
- Required for working on open-source projects and production codebases
- Builds confidence for technical interviews

## Prerequisites

- **C++ knowledge** (all code examples will be in C++)
- Java code will also be provided
- DSA fundamentals

---

## Understanding LLD Through a Story

### The Quick Ride Application Scenario

**Characters:**

- **Anurag** - DSA expert, no LLD knowledge
- **Maurya** - DSA expert + LLD expert
- **Problem** - Build a ride-booking platform (similar to Ola/Uber)

### Anurag’s Approach (DSA-Only Mindset)

When asked to design Quick Ride, Anurag immediately jumped to algorithms:

### Problem 1: Route Finding

**Thinking Process:**

- City = Graph (intersections = nodes, roads = edges)
- Source to destination = Shortest path problem
- **Solution:** Dijkstra’s Algorithm

```
City Map:
Intersections (Nodes) → O
Roads (Edges) → ―――

Graph Representation:
    O―――O
    |   |
    O―――O
```

### Problem 2: Rider Assignment

**Thinking Process:**

- User needs nearest rider
- Multiple riders around user location

```
User (U) with riders around:
    R1
       \
        U―――R2
       /
    R3

    R4 (far away)
```

**Solution:**

- Use Min Heap (Priority Queue)
- Store riders R1, R2, R3, R4 in heap based on distance
- Pop operation gives closest rider

### Manager’s Feedback to Anurag

**What was missing:**

- ❌ What objects/entities exist in the application?
- ❌ How do objects interact with each other?
- ❌ How to maintain data security?
- ❌ How to integrate notifications?
- ❌ How to integrate payment gateway?
- ❌ How will system handle millions of users?

**Problem:** Anurag jumped directly to algorithms without considering application structure.

---

### Maurya’s Approach (LLD + DSA Mindset)

Maurya thinks about the **application structure first**, then algorithms.

### Step 1: Identify Objects/Entities

- **User** - person booking ride
- **Rider** - person providing ride
- **Location** - geographical coordinates
- **Notification** - alert system
- **Payment** - payment processing

### Step 2: Define Relationships

- How does User interact with Rider?
- How does Rider interact with Location?
- What is the flow between entities?

### Step 3: Data Security

- User’s phone number shouldn’t be visible to Rider after ride completion
- Rider’s phone number shouldn’t be visible to User after ride completion
- Sensitive data protection

### Step 4: Scalability

- How to handle millions of concurrent users?
- Design code that doesn’t break under heavy load

### Step 5: Then DSA

Only **after** designing the structure, apply DSA algorithms (what Anurag did).

---

## Core Focus Areas of LLD

### 1. Scalability

**Definition:** Ability of application to handle increasing number of users without breaking.

**Key Question:** Can the system sustain when millions of users join?

**Example:** Quick Ride should work smoothly whether 100 or 10 million users are using it.

---

### 2. Maintainability

**Definition:** How easily new features can be added without major code changes.

**Key Aspects:**

- Minimal effort to integrate new features
- Quick implementation time
- Easy debugging and updates

**Example:** Adding a “scheduled rides” feature should not require rewriting entire codebase.

---

### 3. Reusability

**Definition:** Code should be easily reusable across different applications (Plug & Play model).

**Important Concept: Tightly Coupled vs Loosely Coupled**

```
❌ Tightly Coupled Code:
- Code is fixed/locked to specific application
- Cannot be reused elsewhere
- Hard to modify

✅ Loosely Coupled Code:
- Application-independent
- Can be plugged into any system
- Easy to reuse and modify
```

**Examples:**

**Notification System:**

- Not specific to Quick Ride
- Can be used in Zomato, Swiggy, any app
- Should be designed as independent module

**Payment Gateway:**

- Generic payment processing
- Reusable across all applications
- Plug & Play integration

**Rider Mapping Algorithm:**

- Current: Min-heap based closest rider assignment
- Future: May change to rating-based or time-based
- Algorithm should be **easily swappable** without breaking application
- Code should support multiple strategies

---

## DSA vs LLD vs HLD

### Key Differences

```
┌─────────────────────────────────────────────────────┐
│                 APPLICATION BUILDING                │
├─────────────┬─────────────────┬────────────────────┤
│    DSA      │      LLD        │       HLD          │
├─────────────┼─────────────────┼────────────────────┤
│ Algorithms  │ Code Structure  │ System Architecture│
│ Isolated    │ Object Design   │ Tech Stack         │
│ Problems    │ Relationships   │ Database Choice    │
│ Solutions   │ Interactions    │ Server Scaling     │
│             │ Design Patterns │ Cost Optimization  │
└─────────────┴─────────────────┴────────────────────┘
```

### Comparison Table

| Aspect | DSA | LLD | HLD |
| --- | --- | --- | --- |
| **Focus** | Solving isolated problems | Code structure & design | System architecture |
| **Scope** | Algorithms (search, sort) | Classes, objects, patterns | Servers, databases, tech stack |
| **Example** | Dijkstra’s algorithm | User, Rider class design | SQL vs NoSQL, AWS vs Azure |
| **Level** | Implementation detail | Application structure | Infrastructure design |
| **Goal** | Optimize operations | Maintainable, scalable code | Scalable distributed systems |

### Relationship

```
Real-World Application
        │
        ├── HLD (System Level)
        │   └── Tech stack, Database, Servers
        │
        ├── LLD (Code Level)
        │   └── Classes, Objects, Design Patterns
        │
        └── DSA (Algorithm Level)
            └── Dijkstra, Heap, Graph algorithms
```

**Analogy:**

- **HLD** = Building blueprint (where rooms go, plumbing, electricity)
- **LLD** = Interior design (how rooms are structured, furniture placement)
- **DSA** = Tools used (hammer, nails, screwdriver)

---

## LLD Diagrams

### Class Diagrams

- Used in Low Level Design
- Show classes, attributes, methods
- Show relationships between classes
- Implementation-level detail

**vs**

### Architecture Diagrams (HLD)

- Used in High Level Design
- Show servers, databases, load balancers
- Show system components and their interactions
- Infrastructure-level detail

```
LLD Class Diagram Example:

┌─────────────┐         ┌─────────────┐
│    User     │         │    Rider    │
├─────────────┤         ├─────────────┤
│ - id        │         │ - id        │
│ - name      │◄───────►│ - name      │
│ - location  │         │ - vehicle   │
├─────────────┤         ├─────────────┤
│ + bookRide()│         │ + acceptRide│
└─────────────┘         └─────────────┘
```

---

## Quick Ride Example - LLD Perspective

### HLD Questions for Quick Ride:

- Which tech stack to use?
- SQL or NoSQL database?
- How to scale servers?
- How to optimize costs?

### LLD Questions for Quick Ride:

- What are the entities? (User, Rider, Location)
- How do they interact?
- How to maintain data security?
- How to integrate notifications and payments?
- How to make code modular and reusable?

### DSA Questions for Quick Ride:

- How to find shortest path? (Dijkstra)
- How to find nearest rider? (Min Heap)
- How to handle concurrent requests? (Threading)

---

## Key Takeaways

1. **LLD is the bridge** between DSA problem-solving and building real applications
2. **Three pillars of LLD:** Scalability, Maintainability, Reusability
3. **Avoid tightly coupled code** - design for flexibility and reusability
4. **Think structure first, algorithms later** - don’t jump to DSA solutions immediately
5. **LLD focuses on code organization** - classes, objects, design patterns, relationships
6. **Plug & Play principle** - components should be independent and easily integrable
7. **All three are essential:** DSA (algorithms) + LLD (code structure) + HLD (system architecture) = Complete Application
