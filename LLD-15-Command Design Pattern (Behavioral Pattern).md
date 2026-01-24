# LLD-15: Command Design Pattern (Behavioral Pattern)

---

## 📋 Table of Contents
- [Introduction](#introduction)
- [Problem Statement](#problem-statement)
- [Solution: Command Pattern](#solution-command-pattern)
- [Example: Smart Home Automation](#example-smart-home-automation)
- [UML Diagrams](#uml-diagrams)
- [Implementation](#implementation)
- [Key Concepts](#key-concepts)
- [Real-World Use Cases](#real-world-use-cases)
- [Key Takeaways](#key-takeaways)
- [Interview Q&A](#interview-qa)

---

## 🎯 Introduction

The **Command Design Pattern** is a behavioral design pattern that encapsulates a request as an object, thereby allowing you to parameterize clients with different requests, queue or log requests, and support undoable operations.

**Definition:**
> "The Command pattern encapsulates a request as an object, thereby letting you parameterize clients with different requests, queue and log requests, and support undoable operations."

**Key Concept:**
- Turns **requests/actions into objects**
- **Decouples** sender from receiver
- Enables **undo/redo** functionality
- Supports **dynamic command assignment**

---

## ❓ Problem Statement

### Scenario: Direct Coupling Between Source and Receiver

Imagine you have a **source** object that needs to send a request to a **receiver** object:

```
┌─────────┐                    ┌──────────┐
│ Source  │───────────────────▶│ Receiver │
└─────────┘   Direct Call      └──────────┘
              receiver.run()
```

**Problems with Direct Approach:**
1. **Tight Coupling:** Source is tightly coupled to receiver
2. **No Flexibility:** Can't change receiver dynamically
3. **No Undo:** Can't undo operations
4. **Violates OCP:** Adding new operations requires modifying source

### Example: Remote Control Without Command Pattern

```java
// BAD: Tight coupling
public class RemoteControl {
    private Light light;
    private Fan fan;
    private AC ac;
    
    public void pressLightButton() {
        light.on(); // Tightly coupled to Light
    }
    
    public void pressFanButton() {
        fan.on(); // Tightly coupled to Fan
    }
    
    // What if we want to change what button does?
    // We need to modify RemoteControl class! ❌
}
```

**Problems:**
- Remote is tightly coupled to specific appliances
- Can't dynamically assign buttons to different appliances
- Can't implement undo functionality
- Violates Open/Closed Principle

---

## 💡 Solution: Command Pattern

### Introduce Command as an Object

Instead of direct coupling, introduce a **Command** object between source and receiver:

```
┌─────────┐         ┌─────────┐         ┌──────────┐
│ Source  │────────▶│ Command │────────▶│ Receiver │
└─────────┘         └─────────┘         └──────────┘
   (Invoker)         (Request           (Knows how
                     as Object)          to perform)
```

**How it works:**
1. **Source (Invoker)** calls `command.execute()`
2. **Command** knows which **Receiver** to call
3. **Receiver** performs the actual work

**Benefits:**
- ✅ **Loose Coupling:** Source doesn't know about receiver
- ✅ **Dynamic Assignment:** Can change commands at runtime
- ✅ **Undo/Redo:** Commands can be undone
- ✅ **Follows OCP:** Add new commands without modifying source

---

## 🏠 Example: Smart Home Automation

### Scenario

Build a smart home automation app with a remote control that has multiple buttons. Each button can control different appliances (lights, fans, AC, etc.).

**Requirements:**
1. Multiple buttons on remote
2. Each button can be **dynamically assigned** to any appliance
3. Buttons should **toggle** (on/off) functionality
4. Should support **undo** operations

### Without Command Pattern (Bad Approach)

```
┌──────────────────┐
│ Remote Control   │
│                  │
│  [Button 1]      │──▶ Directly calls light.on()
│  [Button 2]      │──▶ Directly calls fan.on()
│  [Button 3]      │──▶ Directly calls ac.on()
└──────────────────┘
```

**Problems:**
- Remote is tightly coupled to Light, Fan, AC
- Can't change what button does without modifying Remote class
- No undo functionality

### With Command Pattern (Good Approach)

```
┌──────────────────┐
│ Remote Control   │
│                  │
│  [Button 1]      │──▶ Command 1 ──▶ Light
│  [Button 2]      │──▶ Command 2 ──▶ Fan
│  [Button 3]      │──▶ Command 3 ──▶ AC
└──────────────────┘
```

**Benefits:**
- Remote only knows about Command interface
- Can dynamically assign any command to any button
- Each command knows how to execute and undo itself

---

## 🏗️ UML Diagrams

### Class Diagram

```
┌─────────────────────┐
│  RemoteControl      │
│   (Invoker)         │
├─────────────────────┤
│ - buttons: Command[]│
│ - pressed: boolean[]│
├─────────────────────┤
│ + setCommand(i, cmd)│
│ + pressButton(i)    │
└─────────────────────┘
         │
         │ HAS-A (array)
         ▼
┌─────────────────────┐
│  <<interface>>      │
│     ICommand        │
├─────────────────────┤
│ + execute(): void   │
│ + undo(): void      │
└─────────────────────┘
         △
         │ implements
         ├──────────────┬──────────────┐
         │              │              │
┌────────────────┐ ┌────────────┐ ┌────────────┐
│ LightCommand   │ │ FanCommand │ │  ACCommand │
├────────────────┤ ├────────────┤ ├────────────┤
│ - light: Light │ │ - fan: Fan │ │ - ac: AC   │
├────────────────┤ ├────────────┤ ├────────────┤
│ + execute()    │ │ + execute()│ │ + execute()│
│ + undo()       │ │ + undo()   │ │ + undo()   │
└────────────────┘ └────────────┘ └────────────┘
         │              │              │
         │ HAS-A        │ HAS-A        │ HAS-A
         ▼              ▼              ▼
┌────────────┐   ┌────────────┐ ┌────────────┐
│   Light    │   │    Fan     │ │     AC     │
│ (Receiver) │   │ (Receiver) │ │ (Receiver) │
├────────────┤   ├────────────┤ ├────────────┤
│ + on()     │   │ + on()     │ │ + on()     │
│ + off()    │   │ + off()    │ │ + off()    │
└────────────┘   └────────────┘ └────────────┘
```

### Sequence Diagram

```
User      RemoteControl    LightCommand      Light
 │             │                │              │
 │ pressButton(0)              │              │
 │────────────▶│                │              │
 │             │                │              │
 │             │ execute()      │              │
 │             │───────────────▶│              │
 │             │                │              │
 │             │                │  on()        │
 │             │                │─────────────▶│
 │             │                │              │
 │             │                │  "Light ON"  │
 │             │                │◀─────────────│
 │             │                │              │
 │             │   Success      │              │
 │             │◀───────────────│              │
 │             │                │              │
 │  Light ON   │                │              │
 │◀────────────│                │              │
 │             │                │              │
 │             │                │              │
 │ pressButton(0) again         │              │
 │────────────▶│                │              │
 │             │                │              │
 │             │ undo()         │              │
 │             │───────────────▶│              │
 │             │                │              │
 │             │                │  off()       │
 │             │                │─────────────▶│
 │             │                │              │
 │             │                │  "Light OFF" │
 │             │                │◀─────────────│
 │             │                │              │
```

### Component Interaction

```
┌─────────────────────────────────────────────────────┐
│                  Command Pattern                    │
│                                                     │
│  ┌──────────┐         ┌──────────┐                │
│  │ Invoker  │────────▶│ Command  │                │
│  │(Remote)  │         │Interface │                │
│  └──────────┘         └──────────┘                │
│                            △                        │
│                            │                        │
│              ┌─────────────┼─────────────┐         │
│              │             │             │         │
│         ┌─────────┐  ┌─────────┐  ┌─────────┐    │
│         │Concrete │  │Concrete │  │Concrete │    │
│         │Command1 │  │Command2 │  │Command3 │    │
│         └─────────┘  └─────────┘  └─────────┘    │
│              │             │             │         │
│              ▼             ▼             ▼         │
│         ┌─────────┐  ┌─────────┐  ┌─────────┐    │
│         │Receiver1│  │Receiver2│  │Receiver3│    │
│         │(Light)  │  │ (Fan)   │  │  (AC)   │    │
│         └─────────┘  └─────────┘  └─────────┘    │
└─────────────────────────────────────────────────────┘
```

---

## 💻 Implementation

### Step 1: Command Interface

```java
// ICommand.java (Command Interface)
public interface ICommand {
    void execute();
    void undo();
}
```

**Explanation:**
- **Abstract interface** defining command contract
- `execute()`: Performs the action
- `undo()`: Reverses the action
- All concrete commands must implement both methods

---

### Step 2: Receiver Classes (Appliances)

```java
// Light.java (Receiver)
public class Light {
    private String location;
    
    public Light(String location) {
        this.location = location;
    }
    
    public void on() {
        System.out.println(location + " Light is ON");
    }
    
    public void off() {
        System.out.println(location + " Light is OFF");
    }
}

// Fan.java (Receiver)
public class Fan {
    private String location;
    
    public Fan(String location) {
        this.location = location;
    }
    
    public void on() {
        System.out.println(location + " Fan is ON");
    }
    
    public void off() {
        System.out.println(location + " Fan is OFF");
    }
}
```

**Explanation:**
- **Receivers** are the objects that perform actual work
- Each receiver knows how to perform specific operations
- Light and Fan have `on()` and `off()` methods
- Location parameter makes output more descriptive

**Why not create a common Appliance interface?**
- Different appliances have different capabilities
- Light: just on/off
- Fan: on/off + speed control
- AC: on/off + temperature + timer
- Violates Liskov Substitution Principle to force common interface
- Better to have concrete receivers with specific capabilities

---

### Step 3: Concrete Command Classes

```java
// LightCommand.java (Concrete Command)
public class LightCommand implements ICommand {
    private Light light;
    
    public LightCommand(Light light) {
        this.light = light;
    }
    
    @Override
    public void execute() {
        light.on();
    }
    
    @Override
    public void undo() {
        light.off();
    }
}

// FanCommand.java (Concrete Command)
public class FanCommand implements ICommand {
    private Fan fan;
    
    public FanCommand(Fan fan) {
        this.fan = fan;
    }
    
    @Override
    public void execute() {
        fan.on();
    }
    
    @Override
    public void undo() {
        fan.off();
    }
}
```

**Explanation:**
- **Concrete commands** implement `ICommand` interface
- Each command **HAS-A** receiver (composition)
- `execute()`: Calls receiver's action method (e.g., `light.on()`)
- `undo()`: Calls receiver's reverse method (e.g., `light.off()`)

**Key Points:**
1. Command knows **which receiver** to call
2. Command knows **which method** to call on receiver
3. Invoker (Remote) doesn't need to know about receivers
4. Easy to add new commands without modifying existing code

---

### Step 4: Invoker (Remote Control)

```java
// RemoteControl.java (Invoker)
public class RemoteControl {
    private static final int NUMBER_OF_BUTTONS = 4;
    
    private ICommand[] buttons;
    private boolean[] buttonPressed;
    
    public RemoteControl() {
        buttons = new ICommand[NUMBER_OF_BUTTONS];
        buttonPressed = new boolean[NUMBER_OF_BUTTONS];
        
        // Initialize all buttons to null
        for (int i = 0; i < NUMBER_OF_BUTTONS; i++) {
            buttons[i] = null;
            buttonPressed[i] = false;
        }
    }
    
    // Set command for a specific button
    public void setCommand(int index, ICommand command) {
        // Validate index
        if (index < 0 || index >= NUMBER_OF_BUTTONS) {
            System.out.println("Invalid button index!");
            return;
        }
        
        // Delete old command if exists
        if (buttons[index] != null) {
            buttons[index] = null;
        }
        
        // Assign new command
        buttons[index] = command;
        buttonPressed[index] = false; // Reset pressed state
    }
    
    // Press button (toggle functionality)
    public void pressButton(int index) {
        // Validate index
        if (index < 0 || index >= NUMBER_OF_BUTTONS) {
            System.out.println("Invalid button index!");
            return;
        }
        
        // Check if command is assigned
        if (buttons[index] == null) {
            System.out.println("No command assigned at button " + index);
            return;
        }
        
        // Toggle: execute or undo based on current state
        if (!buttonPressed[index]) {
            // Button not pressed yet, execute command
            buttons[index].execute();
            buttonPressed[index] = true;
        } else {
            // Button already pressed, undo command
            buttons[index].undo();
            buttonPressed[index] = false;
        }
    }
}
```

**Explanation:**

**Data Members:**
- `buttons[]`: Array of commands (one per button)
- `buttonPressed[]`: Tracks state of each button (on/off)

**Constructor:**
- Initializes all buttons to `null`
- Sets all button states to `false` (off)

**setCommand(index, command):**
- Assigns a command to a specific button
- Validates index
- Deletes old command if exists
- Resets button state to `false`

**pressButton(index):**
- Validates index and checks if command exists
- **Toggle functionality:**
  - If `buttonPressed[index]` is `false`: Call `execute()`, set to `true`
  - If `buttonPressed[index]` is `true`: Call `undo()`, set to `false`
- This creates on/off toggle behavior

**Key Design Decisions:**
1. **Array vs Vector:** Using array with fixed size for simplicity
2. **Toggle State:** `buttonPressed[]` tracks whether button is in "on" state
3. **Null Check:** Prevents errors when button has no command assigned
4. **Dynamic Assignment:** Can change command at runtime using `setCommand()`

---

### Step 5: Client Code

```java
// Main.java (Client)
public class Main {
    public static void main(String[] args) {
        // Step 1: Create receivers (appliances)
        Light livingRoomLight = new Light("Living Room");
        Fan ceilingFan = new Fan("Ceiling");
        
        // Step 2: Create invoker (remote control)
        RemoteControl remote = new RemoteControl();
        
        // Step 3: Create commands and assign to buttons
        remote.setCommand(0, new LightCommand(livingRoomLight));
        remote.setCommand(1, new FanCommand(ceilingFan));
        
        // Step 4: Simulate button presses
        System.out.println("=== Testing Toggle Functionality ===\n");
        
        // Press button 0 (Light) - Should turn ON
        System.out.println("Pressing button 0 (first time):");
        remote.pressButton(0);
        
        // Press button 0 again (Light) - Should turn OFF
        System.out.println("\nPressing button 0 (second time):");
        remote.pressButton(0);
        
        // Press button 1 (Fan) - Should turn ON
        System.out.println("\nPressing button 1 (first time):");
        remote.pressButton(1);
        
        // Press button 1 again (Fan) - Should turn OFF
        System.out.println("\nPressing button 1 (second time):");
        remote.pressButton(1);
        
        // Press button 2 (No command assigned)
        System.out.println("\nPressing button 2 (no command):");
        remote.pressButton(2);
    }
}
```

**Output:**
```
=== Testing Toggle Functionality ===

Pressing button 0 (first time):
Living Room Light is ON

Pressing button 0 (second time):
Living Room Light is OFF

Pressing button 1 (first time):
Ceiling Fan is ON

Pressing button 1 (second time):
Ceiling Fan is OFF

Pressing button 2 (no command):
No command assigned at button 2
```

**Explanation:**

**Execution Flow:**
1. Create receivers (Light, Fan)
2. Create invoker (RemoteControl)
3. Create commands and assign to buttons
4. Press buttons to test toggle functionality

**First Press (Button 0):**
```
User → remote.pressButton(0)
     → buttonPressed[0] is false
     → buttons[0].execute()
     → LightCommand.execute()
     → light.on()
     → "Living Room Light is ON"
     → buttonPressed[0] = true
```

**Second Press (Button 0):**
```
User → remote.pressButton(0)
     → buttonPressed[0] is true
     → buttons[0].undo()
     → LightCommand.undo()
     → light.off()
     → "Living Room Light is OFF"
     → buttonPressed[0] = false
```

**Key Points:**
- Remote doesn't know about Light or Fan
- Remote only knows about ICommand interface
- Easy to add new appliances without changing Remote
- Toggle functionality works perfectly

---

## 🔑 Key Concepts

### 1. Encapsulation of Requests

**Concept:** Treat requests/actions as first-class objects

```java
// Request as object
ICommand command = new LightCommand(light);

// Can store in variables
ICommand[] commands = new ICommand[10];

// Can pass as parameters
remote.setCommand(0, command);

// Can store in collections
List<ICommand> commandHistory = new ArrayList<>();
```

---

### 2. Decoupling Invoker and Receiver

**Without Command Pattern:**
```java
// Tight coupling
public class Remote {
    private Light light;
    
    public void pressButton() {
        light.on(); // Remote knows about Light
    }
}
```

**With Command Pattern:**
```java
// Loose coupling
public class Remote {
    private ICommand command;
    
    public void pressButton() {
        command.execute(); // Remote doesn't know about Light
    }
}
```

**Benefits:**
- Remote doesn't depend on concrete receivers
- Easy to add new receivers
- Follows Dependency Inversion Principle

---

### 3. Undo/Redo Functionality

**Implementation:**

```java
public class TextEditor {
    private Stack<ICommand> commandHistory = new Stack<>();
    private Stack<ICommand> redoStack = new Stack<>();
    
    public void executeCommand(ICommand command) {
        command.execute();
        commandHistory.push(command);
        redoStack.clear(); // Clear redo stack on new command
    }
    
    public void undo() {
        if (!commandHistory.isEmpty()) {
            ICommand command = commandHistory.pop();
            command.undo();
            redoStack.push(command);
        }
    }
    
    public void redo() {
        if (!redoStack.isEmpty()) {
            ICommand command = redoStack.pop();
            command.execute();
            commandHistory.push(command);
        }
    }
}
```

**Example Commands:**
```java
// Bold text command
public class BoldCommand implements ICommand {
    private TextEditor editor;
    private int start, end;
    
    @Override
    public void execute() {
        editor.applyBold(start, end);
    }
    
    @Override
    public void undo() {
        editor.removeBold(start, end);
    }
}
```

---

### 4. Command Queue/Logging

**Queuing Commands:**
```java
public class CommandQueue {
    private Queue<ICommand> queue = new LinkedList<>();
    
    public void addCommand(ICommand command) {
        queue.offer(command);
    }
    
    public void processCommands() {
        while (!queue.isEmpty()) {
            ICommand command = queue.poll();
            command.execute();
        }
    }
}
```

**Logging Commands:**
```java
public class CommandLogger {
    private List<ICommand> log = new ArrayList<>();
    
    public void logCommand(ICommand command) {
        log.add(command);
        System.out.println("Logged: " + command.getClass().getSimpleName());
    }
    
    public void replayCommands() {
        for (ICommand command : log) {
            command.execute();
        }
    }
}
```

---

### 5. Macro Commands (Composite Commands)

**Concept:** Group multiple commands into one

```java
public class MacroCommand implements ICommand {
    private List<ICommand> commands;
    
    public MacroCommand(List<ICommand> commands) {
        this.commands = commands;
    }
    
    @Override
    public void execute() {
        for (ICommand command : commands) {
            command.execute();
        }
    }
    
    @Override
    public void undo() {
        // Undo in reverse order
        for (int i = commands.size() - 1; i >= 0; i--) {
            commands.get(i).undo();
        }
    }
}

// Usage
List<ICommand> nightModeCommands = Arrays.asList(
    new LightCommand(light),
    new ACCommand(ac),
    new CurtainCommand(curtain)
);

ICommand nightMode = new MacroCommand(nightModeCommands);
remote.setCommand(0, nightMode); // One button for multiple actions
```

---

## 🌍 Real-World Use Cases

### 1. Text Editors (Undo/Redo)

**Scenario:** Microsoft Word, Google Docs, VS Code

```java
// Text editing commands
public class InsertTextCommand implements ICommand {
    private TextEditor editor;
    private String text;
    private int position;
    
    @Override
    public void execute() {
        editor.insertText(text, position);
    }
    
    @Override
    public void undo() {
        editor.deleteText(position, text.length());
    }
}

public class DeleteTextCommand implements ICommand {
    private TextEditor editor;
    private String deletedText;
    private int position;
    
    @Override
    public void execute() {
        deletedText = editor.getText(position, length);
        editor.deleteText(position, length);
    }
    
    @Override
    public void undo() {
        editor.insertText(deletedText, position);
    }
}

public class BoldCommand implements ICommand {
    private TextEditor editor;
    private int start, end;
    
    @Override
    public void execute() {
        editor.applyBold(start, end);
    }
    
    @Override
    public void undo() {
        editor.removeBold(start, end);
    }
}
```

**How Ctrl+Z works:**
```java
Stack<ICommand> history = new Stack<>();

// User types "Hello"
ICommand cmd1 = new InsertTextCommand(editor, "Hello", 0);
cmd1.execute();
history.push(cmd1);

// User presses Ctrl+Z
ICommand lastCommand = history.pop();
lastCommand.undo(); // "Hello" is removed
```

---

### 2. Photoshop/Image Editors

**Scenario:** Adobe Photoshop, GIMP

```java
public class BrightnessCommand implements ICommand {
    private Image image;
    private int delta;
    private int previousBrightness;
    
    @Override
    public void execute() {
        previousBrightness = image.getBrightness();
        image.setBrightness(previousBrightness + delta);
    }
    
    @Override
    public void undo() {
        image.setBrightness(previousBrightness);
    }
}

public class CropCommand implements ICommand {
    private Image image;
    private Rectangle cropArea;
    private Image originalImage;
    
    @Override
    public void execute() {
        originalImage = image.clone();
        image.crop(cropArea);
    }
    
    @Override
    public void undo() {
        image = originalImage;
    }
}
```

---

### 3. Keyboard Shortcuts

**Scenario:** Custom keyboard shortcuts in IDEs, OS

```java
public class KeyboardShortcutManager {
    private Map<String, ICommand> shortcuts = new HashMap<>();
    
    public void registerShortcut(String key, ICommand command) {
        shortcuts.put(key, command);
    }
    
    public void handleKeyPress(String key) {
        ICommand command = shortcuts.get(key);
        if (command != null) {
            command.execute();
        }
    }
}

// Usage
KeyboardShortcutManager manager = new KeyboardShortcutManager();

// Ctrl+B for brightness
manager.registerShortcut("Ctrl+B", new BrightnessCommand(monitor, 10));

// Ctrl+V for volume
manager.registerShortcut("Ctrl+V", new VolumeCommand(speaker, 5));

// User presses Ctrl+B
manager.handleKeyPress("Ctrl+B"); // Brightness increases
```

---

### 4. Transaction Systems

**Scenario:** Database transactions, financial systems

```java
public class TransferMoneyCommand implements ICommand {
    private Account from;
    private Account to;
    private double amount;
    
    @Override
    public void execute() {
        from.debit(amount);
        to.credit(amount);
    }
    
    @Override
    public void undo() {
        to.debit(amount);
        from.credit(amount);
    }
}

// Transaction manager
public class TransactionManager {
    private List<ICommand> transactions = new ArrayList<>();
    
    public void executeTransaction(ICommand transaction) {
        try {
            transaction.execute();
            transactions.add(transaction);
        } catch (Exception e) {
            // Rollback all transactions
            rollback();
        }
    }
    
    public void rollback() {
        for (int i = transactions.size() - 1; i >= 0; i--) {
            transactions.get(i).undo();
        }
        transactions.clear();
    }
}
```

---

### 5. Game Development

**Scenario:** Player actions in games

```java
public class MoveCommand implements ICommand {
    private Player player;
    private int deltaX, deltaY;
    private int previousX, previousY;
    
    @Override
    public void execute() {
        previousX = player.getX();
        previousY = player.getY();
        player.move(deltaX, deltaY);
    }
    
    @Override
    public void undo() {
        player.setPosition(previousX, previousY);
    }
}

public class AttackCommand implements ICommand {
    private Player player;
    private Enemy enemy;
    private int damage;
    
    @Override
    public void execute() {
        enemy.takeDamage(damage);
    }
    
    @Override
    public void undo() {
        enemy.heal(damage);
    }
}

// Replay system
public class GameReplay {
    private List<ICommand> recordedCommands = new ArrayList<>();
    
    public void record(ICommand command) {
        recordedCommands.add(command);
        command.execute();
    }
    
    public void replay() {
        for (ICommand command : recordedCommands) {
            command.execute();
        }
    }
}
```

---

### 6. Task Scheduling

**Scenario:** Cron jobs, scheduled tasks

```java
public class TaskScheduler {
    private Queue<ICommand> scheduledTasks = new PriorityQueue<>();
    
    public void scheduleTask(ICommand task, long delay) {
        scheduledTasks.offer(task);
        // Schedule for execution after delay
    }
    
    public void executePendingTasks() {
        while (!scheduledTasks.isEmpty()) {
            ICommand task = scheduledTasks.poll();
            task.execute();
        }
    }
}
```

---

## 🎯 Key Takeaways

### When to Use Command Pattern

✅ **Use Command Pattern when:**
1. You need to **parameterize objects** with operations
2. You need to **queue operations** for later execution
3. You need to support **undo/redo** functionality
4. You need to **log operations** for auditing
5. You want to **decouple** sender from receiver
6. You need to support **macro commands** (composite operations)

❌ **Don't Use Command Pattern when:**
1. Operations are simple and don't need undo
2. No need for queuing or logging
3. Adding unnecessary complexity
4. Performance is critical (extra layer of indirection)

---

### Design Principles Applied

1. **Single Responsibility Principle:**
   - Command: Encapsulates one action
   - Receiver: Performs actual work
   - Invoker: Triggers commands

2. **Open/Closed Principle:**
   - Add new commands without modifying invoker
   - Extend functionality by creating new command classes

3. **Dependency Inversion Principle:**
   - Invoker depends on ICommand interface
   - Not on concrete command classes

4. **Liskov Substitution Principle:**
   - Any ICommand implementation can be used
   - Invoker works with all commands uniformly

---

### Benefits

1. ✅ **Decoupling:** Invoker and receiver are decoupled
2. ✅ **Flexibility:** Easy to add new commands
3. ✅ **Undo/Redo:** Built-in support for reversible operations
4. ✅ **Queuing:** Can queue commands for later execution
5. ✅ **Logging:** Can log all operations
6. ✅ **Macro Commands:** Can combine multiple commands
7. ✅ **Dynamic Binding:** Can change commands at runtime

---

### Drawbacks

1. ❌ **Increased Complexity:** More classes to manage
2. ❌ **Memory Overhead:** Storing command history
3. ❌ **Performance:** Extra layer of indirection

---

## ❓ Interview Q&A

### Q1: What is the Command Design Pattern?

**Answer:**
The Command Design Pattern is a behavioral pattern that encapsulates a request as an object, thereby allowing you to parameterize clients with different requests, queue or log requests, and support undoable operations.

**Key Components:**
1. **Command Interface:** Defines execute() and undo() methods
2. **Concrete Commands:** Implement command interface
3. **Receiver:** Performs actual work
4. **Invoker:** Triggers commands
5. **Client:** Creates commands and sets up invoker

**Example:**
```java
// Command as object
ICommand command = new LightCommand(light);
remote.setCommand(0, command);
remote.pressButton(0); // Executes command
```

---

### Q2: What problem does Command Pattern solve?

**Answer:**

**Problems Solved:**
1. **Tight Coupling:** Decouples sender from receiver
2. **No Undo:** Enables undo/redo functionality
3. **Static Binding:** Allows dynamic command assignment
4. **No Logging:** Enables operation logging and queuing

**Example:**
```java
// Without Command Pattern (Bad)
public class Remote {
    private Light light;
    public void pressButton() {
        light.on(); // Tight coupling
    }
}

// With Command Pattern (Good)
public class Remote {
    private ICommand command;
    public void pressButton() {
        command.execute(); // Loose coupling
    }
}
```

---

### Q3: How does Command Pattern enable undo functionality?

**Answer:**

**Undo Implementation:**
1. Each command stores previous state
2. `execute()` performs action and saves state
3. `undo()` restores previous state
4. Use stack to track command history

**Example:**
```java
public class LightCommand implements ICommand {
    private Light light;
    
    @Override
    public void execute() {
        light.on(); // Turn on
    }
    
    @Override
    public void undo() {
        light.off(); // Reverse action
    }
}

// Undo manager
Stack<ICommand> history = new Stack<>();

// Execute command
command.execute();
history.push(command);

// Undo (Ctrl+Z)
ICommand last = history.pop();
last.undo();
```

---

### Q4: What's the difference between Command and Strategy patterns?

**Answer:**

| Aspect | Command Pattern | Strategy Pattern |
|--------|----------------|------------------|
| **Intent** | Encapsulate request as object | Encapsulate algorithm |
| **Focus** | What to do | How to do |
| **Undo** | Supports undo/redo | No undo support |
| **State** | Stores receiver and parameters | Stateless algorithm |
| **Usage** | Actions, transactions | Algorithm selection |

**Command Example:**
```java
ICommand command = new SaveCommand(document);
command.execute(); // What: Save
command.undo();    // Can undo
```

**Strategy Example:**
```java
SortStrategy strategy = new QuickSort();
sorter.setStrategy(strategy);
sorter.sort(data); // How: QuickSort algorithm
// No undo
```

---

### Q5: Can you give a real-world example of Command Pattern?

**Answer:**

**1. Text Editor (Microsoft Word):**
```java
// Every action is a command
ICommand insertText = new InsertTextCommand("Hello");
ICommand bold = new BoldCommand(0, 5);
ICommand delete = new DeleteCommand(0, 2);

// Execute
insertText.execute(); // "Hello"
bold.execute();       // "**Hello**"

// Undo (Ctrl+Z)
bold.undo();          // "Hello"
insertText.undo();    // ""
```

**2. Smart Home Remote:**
```java
// Dynamic button assignment
remote.setCommand(0, new LightCommand(light));
remote.setCommand(1, new ACCommand(ac));

// Press buttons
remote.pressButton(0); // Light ON
remote.pressButton(0); // Light OFF (undo)
```

**3. Database Transactions:**
```java
ICommand transfer = new TransferCommand(acc1, acc2, 1000);
transfer.execute();

// Rollback on error
transfer.undo();
```

---

### Q6: How do you implement macro commands?

**Answer:**

**Macro Command:** Combines multiple commands into one

```java
public class MacroCommand implements ICommand {
    private List<ICommand> commands;
    
    public MacroCommand(List<ICommand> commands) {
        this.commands = commands;
    }
    
    @Override
    public void execute() {
        for (ICommand command : commands) {
            command.execute();
        }
    }
    
    @Override
    public void undo() {
        // Undo in reverse order
        for (int i = commands.size() - 1; i >= 0; i--) {
            commands.get(i).undo();
        }
    }
}

// Usage: Night mode (turn off all lights, turn on AC)
List<ICommand> nightMode = Arrays.asList(
    new LightCommand(light1),
    new LightCommand(light2),
    new ACCommand(ac)
);

ICommand nightModeCommand = new MacroCommand(nightMode);
remote.setCommand(0, nightModeCommand);

remote.pressButton(0); // All execute together
remote.pressButton(0); // All undo together
```

---

### Q7: Why does Command have HAS-A relationship with Receiver, not Command interface?

**Answer:**

**Reason:** Intent and responsibility

**ICommand (Interface):**
- Defines **what** can be done (execute, undo)
- Doesn't know **who** will do it
- Abstract contract

**Concrete Command (e.g., LightCommand):**
- Knows **who** will do it (Light)
- Knows **how** to do it (call light.on())
- Concrete implementation

**Example:**
```java
// ICommand doesn't know about receivers
public interface ICommand {
    void execute(); // What to do
    void undo();    // What to undo
}

// LightCommand knows about Light
public class LightCommand implements ICommand {
    private Light light; // HAS-A Light
    
    @Override
    public void execute() {
        light.on(); // How to do it
    }
}
```

**Why not HAS-A in interface?**
- Interface is abstract, doesn't have implementation
- Different commands work with different receivers
- Keeps interface generic and reusable

---

### Q8: Why not create a common Appliance interface for all receivers?

**Answer:**

**Problem:** Violates Liskov Substitution Principle

**Different appliances have different capabilities:**
- **Light:** on(), off()
- **Fan:** on(), off(), setSpeed()
- **AC:** on(), off(), setTemperature(), setTimer()
- **Refrigerator:** on(), off(), setTemperature(), setMode()

**If we force common interface:**
```java
public interface Appliance {
    void on();
    void off();
    // What about setSpeed()? setTemperature()?
}

// Fan forced to implement only on/off
public class Fan implements Appliance {
    public void on() { }
    public void off() { }
    // Can't expose setSpeed() through interface!
}
```

**Solution:** Concrete receivers with specific capabilities
```java
// Each receiver has its own interface
public class Light {
    public void on() { }
    public void off() { }
}

public class Fan {
    public void on() { }
    public void off() { }
    public void setSpeed(int speed) { } // Fan-specific
}

// Each command works with specific receiver
public class FanCommand implements ICommand {
    private Fan fan; // Concrete receiver
    
    public void execute() {
        fan.on();
        fan.setSpeed(3); // Can use fan-specific methods
    }
}
```

---

### Q9: How does Command Pattern support logging and queuing?

**Answer:**

**1. Logging Commands:**
```java
public class CommandLogger {
    private List<ICommand> log = new ArrayList<>();
    
    public void execute(ICommand command) {
        command.execute();
        log.add(command);
        System.out.println("Logged: " + command);
    }
    
    public void replay() {
        for (ICommand command : log) {
            command.execute();
        }
    }
}

// Usage
logger.execute(new SaveCommand());
logger.execute(new PrintCommand());
logger.replay(); // Replay all actions
```

**2. Queuing Commands:**
```java
public class CommandQueue {
    private Queue<ICommand> queue = new LinkedList<>();
    
    public void addCommand(ICommand command) {
        queue.offer(command);
    }
    
    public void processCommands() {
        while (!queue.isEmpty()) {
            ICommand command = queue.poll();
            command.execute();
        }
    }
}

// Usage: Batch processing
queue.addCommand(new EmailCommand());
queue.addCommand(new BackupCommand());
queue.addCommand(new ReportCommand());
queue.processCommands(); // Execute all in order
```

**3. Scheduled Execution:**
```java
public class Scheduler {
    private Map<Long, ICommand> scheduled = new HashMap<>();
    
    public void schedule(ICommand command, long time) {
        scheduled.put(time, command);
    }
    
    public void executeScheduled() {
        long now = System.currentTimeMillis();
        for (Map.Entry<Long, ICommand> entry : scheduled.entrySet()) {
            if (entry.getKey() <= now) {
                entry.getValue().execute();
            }
        }
    }
}
```

---

### Q10: What are the advantages and disadvantages of Command Pattern?

**Answer:**

**Advantages:**
- ✅ **Decoupling:** Invoker and receiver are decoupled
- ✅ **Undo/Redo:** Easy to implement reversible operations
- ✅ **Queuing:** Can queue commands for later execution
- ✅ **Logging:** Can log all operations for auditing
- ✅ **Macro Commands:** Can combine multiple commands
- ✅ **Dynamic Binding:** Can change commands at runtime
- ✅ **Open/Closed:** Add new commands without modifying invoker

**Disadvantages:**
- ❌ **Complexity:** More classes to manage (one per command)
- ❌ **Memory:** Storing command history consumes memory
- ❌ **Performance:** Extra layer of indirection
- ❌ **Overhead:** May be overkill for simple operations

**When to Use:**
- Text editors (undo/redo)
- Transaction systems (rollback)
- Task scheduling
- Remote controls
- Keyboard shortcuts

**When NOT to Use:**
- Simple CRUD operations
- No need for undo
- Performance-critical systems
- Simple request-response

---

## 🎓 Summary

The **Command Design Pattern** is essential for:
- Encapsulating requests as objects
- Decoupling sender from receiver
- Implementing undo/redo functionality
- Queuing and logging operations
- Supporting macro commands

**Remember:**
- Command **encapsulates** request as object
- Invoker **doesn't know** about receiver
- Command **HAS-A** receiver
- Supports **undo/redo** operations
- Enables **queuing and logging**

**Real-world analogy:** Just like a restaurant order slip encapsulates a customer's request (what to cook) without the waiter knowing how to cook it, the Command pattern encapsulates a request as an object, allowing the invoker to trigger actions without knowing the implementation details! 🍽️

---
