---
title: Designing a Food Delivery Application (Zomato/Swiggy Clone)
lecture: LLD-11
topic: Low-Level Design
tags: [design-pattern, oop, solid, factory-pattern, strategy-pattern, singleton, system-design]
difficulty: Hard
created: 2026-01-20
updated: 2026-01-20
---

# Lecture 11: Build Zomato Food Delivery App | System Design

> **Lecture:** LLD-11  
> **Topic:** [[Low-Level Design/README|Low-Level Design]]  
> **Previous:** [[LLD-10|LLD-10. Previous Topic]]  
> **Next:** [[LLD-12|LLD-12. Next Topic]]

---

## 🧐 The Problem / Real-World Scenario

Imagine you're tasked with building a **food delivery application** like Zomato or Swiggy from scratch. This isn't just about creating a simple app—it's about designing a **scalable, maintainable, and extensible system** that handles:

- **Multiple restaurants** with their menus
- **User interactions**: searching restaurants, adding items to cart, placing orders
- **Different order types**: delivery orders vs. pickup orders
- **Payment processing** through various methods (UPI, Credit Card, Net Banking)
- **Notifications** when orders are placed
- **Future extensibility**: scheduled orders, new payment methods, new order types

**The Challenge:** Design this system in a way that follows **OOP principles**, uses appropriate **design patterns**, and maintains **loose coupling** between components.

> [!important] Interview Context
> In an LLD interview, you're NOT expected to build a complete working application in 1 hour. The interviewer wants to see:
> - How you **gather requirements** and narrow down scope
> - How you **think through the entire flow**
> - How you **apply design principles** (SOLID, design patterns)
> - How you **draw relationships** between objects
> - How you write **clean, working code**

---

## 🖼️ Understanding the Concept / System Architecture

### High-Level Flow

```
┌─────────┐
│  User   │
└────┬────┘
     │
     ▼
┌─────────────────────┐
│ Search Restaurants  │ ◄─── Restaurant Manager
│  (by location)      │
└─────────┬───────────┘
          │
          ▼
┌─────────────────────┐
│ View Restaurant     │
│ & Menu Items        │
└─────────┬───────────┘
          │
          ▼
┌─────────────────────┐
│  Add Items to Cart  │
└─────────┬───────────┘
          │
          ▼
┌─────────────────────┐
│   Place Order       │ ◄─── Order Factory
│ (Delivery/Pickup)   │
└─────────┬───────────┘
          │
          ▼
┌─────────────────────┐
│  Process Payment    │ ◄─── Payment Strategy
└─────────┬───────────┘
          │
          ▼
┌─────────────────────┐
│ Send Notification   │ ◄─── Notification Service
└─────────────────────┘
```

---

## 💡 Core Concepts / Design Principles

### Functional Requirements
1. ✅ User can **search for restaurants** based on location
2. ✅ User can **add items to cart**
3. ✅ User can **checkout and make payments**
4. ✅ User should be **notified** once order is placed successfully
5. ✅ Support **two order types**: Delivery Order & Pickup Order

### Non-Functional Requirements
1. **Scalability**: System should handle multiple restaurants, users, orders
2. **Extensibility**: Easy to add new payment methods, order types
3. **Loose Coupling**: Components should interact through well-defined interfaces
4. **Single Responsibility**: Each class should have one clear purpose

> [!note] Scope Narrowing
> Through discussion with the interviewer, we decided:
> - Focus on **user-centric flow** (not delivery agent flow)
> - **Payment service** is a third-party integration (not building from scratch)
> - **Notification service** is external (just integrate it)

### Design Approach: Bottom-Up vs Top-Down

**Bottom-Up Approach** (Used in this design):
- Start with **smaller objects** (MenuItem, Restaurant)
- Build relationships between them
- Then create **larger objects** (Managers, Factories) that use them

**Top-Down Approach**:
- Start with **larger objects** first
- Break them down into smaller components

> [!tip] When to Use Bottom-Up
> Bottom-up works well when you can clearly identify small, independent entities first. It makes relationship building easier as you scale up.

---

## 🛠️ Step-by-Step Design Process

### Step 1: Identify Core Entities (Models)

**Small Objects First:**
1. **MenuItem** - Individual food item
2. **Restaurant** - Has multiple menu items
3. **User** - The customer
4. **Cart** - User's shopping cart
5. **Order** - Placed order (abstract)
   - DeliveryOrder (concrete)
   - PickupOrder (concrete)

### Step 2: Identify Manager Classes

**Managers** handle CRUD operations and act as **single point of contact**:
1. **RestaurantManager** - Manages all restaurants (Singleton)
2. **OrderManager** - Manages all orders (Singleton)

> [!important] Why Singleton for Managers?
> Managers maintain a **single list** of entities throughout the application. Multiple instances would create inconsistent state.

### Step 3: Apply Design Patterns

**Strategy Pattern** - Payment Processing:
- Different payment methods (UPI, Credit Card, Net Banking)
- Client doesn't need to know which strategy is used

**Factory Method Pattern** - Order Creation:
- Different order factories (NowOrderFactory, ScheduledOrderFactory)
- Each can create different order types (Delivery, Pickup)

**Singleton Pattern** - Managers:
- Ensures single instance throughout application

### Step 4: Create Orchestrator

**TomatoApp** - Single point of contact for client (frontend):
- Delegates requests to appropriate managers
- Hides internal complexity from client

> [!note] Trade-off Alert
> Orchestrator violates **Single Responsibility Principle** and **Principle of Least Knowledge**, but provides a clean API for clients. This is an acceptable trade-off for simplicity.

---

## 🎨 Class Diagram

```
┌─────────────────────────────────────────────────────────────────┐
│                         TomatoApp                               │
│                      (Orchestrator)                             │
├─────────────────────────────────────────────────────────────────┤
│ - restaurantManager: RestaurantManager                          │
│ - orderManager: OrderManager                                    │
│ - notificationService: NotificationService                      │
├─────────────────────────────────────────────────────────────────┤
│ + searchRestaurants(location): vector<Restaurant*>              │
│ + placeOrder(user, cart, paymentStrategy, orderType): Order*   │
└─────────────────────────────────────────────────────────────────┘
                              │
                              │ uses
                              ▼
        ┌─────────────────────────────────────────┐
        │   RestaurantManager (Singleton)         │
        ├─────────────────────────────────────────┤
        │ - restaurants: vector<Restaurant*>      │
        │ - instance: RestaurantManager*          │
        ├─────────────────────────────────────────┤
        │ + getInstance(): RestaurantManager*     │
        │ + addRestaurant(r: Restaurant*)         │
        │ + searchByLocation(loc): vector<Res*>   │
        └───────────────┬─────────────────────────┘
                        │ aggregation ◇
                        │ 1..*
                        ▼
        ┌─────────────────────────────────────────┐
        │         Restaurant (Model)              │
        ├─────────────────────────────────────────┤
        │ - id: int                               │
        │ - name: string                          │
        │ - location: string                      │
        │ - menu: vector<MenuItem*>               │
        ├─────────────────────────────────────────┤
        │ + getters/setters                       │
        └───────────────┬─────────────────────────┘
                        │ composition ◆
                        │ 1..*
                        ▼
        ┌─────────────────────────────────────────┐
        │         MenuItem (Model)                │
        ├─────────────────────────────────────────┤
        │ - code: string                          │
        │ - name: string                          │
        │ - price: double                         │
        ├─────────────────────────────────────────┤
        │ + getters/setters                       │
        └─────────────────────────────────────────┘

        ┌─────────────────────────────────────────┐
        │            User (Model)                 │
        ├─────────────────────────────────────────┤
        │ - id: int                               │
        │ - name: string                          │
        │ - address: string                       │
        │ - cart: Cart*                           │
        ├─────────────────────────────────────────┤
        │ + getters/setters                       │
        └───────────────┬─────────────────────────┘
                        │ composition ◆
                        │ 1..1
                        ▼
        ┌─────────────────────────────────────────┐
        │            Cart (Model)                 │
        ├─────────────────────────────────────────┤
        │ - restaurant: Restaurant*               │
        │ - items: vector<MenuItem*>              │
        │ - total: double                         │
        ├─────────────────────────────────────────┤
        │ + addItem(item: MenuItem*)              │
        │ + getTotalCost(): double                │
        │ + isEmpty(): bool                       │
        │ + clear()                               │
        └─────────────────────────────────────────┘

        ┌─────────────────────────────────────────┐
        │      OrderManager (Singleton)           │
        ├─────────────────────────────────────────┤
        │ - orders: vector<Order*>                │
        │ - instance: OrderManager*               │
        ├─────────────────────────────────────────┤
        │ + getInstance(): OrderManager*          │
        │ + addOrder(order: Order*)               │
        │ + listOrders()                          │
        └───────────────┬─────────────────────────┘
                        │ aggregation ◇
                        │ 1..*
                        ▼
        ┌─────────────────────────────────────────┐
        │       Order (Abstract Model)            │
        ├─────────────────────────────────────────┤
        │ - id: int                               │
        │ - user: User*                           │
        │ - restaurant: Restaurant*               │
        │ - items: vector<MenuItem*>              │
        │ - paymentStrategy: PaymentStrategy*     │
        │ - total: double                         │
        │ - scheduled: string                     │
        ├─────────────────────────────────────────┤
        │ + processPayment()                      │
        │ + getType(): string (virtual)           │
        │ + getters/setters                       │
        └───────────────┬─────────────────────────┘
                        │ inheritance
                ┌───────┴────────┐
                │                │
                ▼                ▼
    ┌───────────────────┐  ┌──────────────────┐
    │ DeliveryOrder     │  │  PickupOrder     │
    ├───────────────────┤  ├──────────────────┤
    │ - userAddress: str│  │ - restAddress:str│
    ├───────────────────┤  ├──────────────────┤
    │ + getType(): str  │  │ + getType(): str │
    └───────────────────┘  └──────────────────┘

        ┌─────────────────────────────────────────┐
        │   OrderFactory (Interface/Abstract)     │
        ├─────────────────────────────────────────┤
        │ + createOrder(...): Order* (virtual)    │
        └───────────────┬─────────────────────────┘
                        │ inheritance
                ┌───────┴────────┐
                │                │
                ▼                ▼
    ┌───────────────────┐  ┌──────────────────────┐
    │ NowOrderFactory   │  │ScheduledOrderFactory │
    ├───────────────────┤  ├──────────────────────┤
    │ + createOrder()   │  │ + createOrder()      │
    └─────────┬─────────┘  └──────────┬───────────┘
              │                       │
              │ creates               │ creates
              ▼                       ▼
         DeliveryOrder           DeliveryOrder
         PickupOrder             PickupOrder

        ┌─────────────────────────────────────────┐
        │  PaymentStrategy (Interface/Abstract)   │
        ├─────────────────────────────────────────┤
        │ + pay(amount: double): void (virtual)   │
        └───────────────┬─────────────────────────┘
                        │ inheritance
            ┌───────────┼───────────┐
            │           │           │
            ▼           ▼           ▼
    ┌──────────┐  ┌──────────┐  ┌──────────┐
    │   UPI    │  │  Credit  │  │   Net    │
    │ Payment  │  │   Card   │  │ Banking  │
    ├──────────┤  ├──────────┤  ├──────────┤
    │ + pay()  │  │ + pay()  │  │ + pay()  │
    └──────────┘  └──────────┘  └──────────┘

        ┌─────────────────────────────────────────┐
        │      NotificationService                │
        ├─────────────────────────────────────────┤
        │ + notify(order: Order*)                 │
        └─────────────────────────────────────────┘
```

### Relationship Types

| Symbol | Relationship | Meaning |
|--------|-------------|---------|
| ───◆   | Composition | Strong ownership (child can't exist without parent) |
| ───◇   | Aggregation | Weak ownership (child can exist independently) |
| ───▷   | Inheritance | IS-A relationship |
| ───    | Association | Simple HAS-A relationship |

---

## 💻 Java Implementation

### Project Structure
```
com.tomato/
├── models/
│   ├── MenuItem.java
│   ├── Restaurant.java
│   ├── User.java
│   ├── Cart.java
│   ├── Order.java (abstract)
│   ├── DeliveryOrder.java
│   └── PickupOrder.java
├── managers/
│   ├── RestaurantManager.java (Singleton)
│   └── OrderManager.java (Singleton)
├── factories/
│   ├── OrderFactory.java (interface)
│   ├── NowOrderFactory.java
│   └── ScheduledOrderFactory.java
├── strategies/
│   ├── PaymentStrategy.java (interface)
│   ├── UPIPayment.java
│   ├── CreditCardPayment.java
│   └── NetBankingPayment.java
├── services/
│   └── NotificationService.java
├── utils/
│   └── TimeUtils.java
├── TomatoApp.java (Orchestrator)
└── Main.java (Driver)
```

---

### 1. Model Classes

#### MenuItem.java
```java
package com.tomato.models;

public class MenuItem {
    private String code;
    private String name;
    private double price;

    public MenuItem(String code, String name, double price) {
        this.code = code;
        this.name = name;
        this.price = price;
    }

    // Getters
    public String getCode() { return code; }
    public String getName() { return name; }
    public double getPrice() { return price; }

    // Setters
    public void setCode(String code) { this.code = code; }
    public void setName(String name) { this.name = name; }
    public void setPrice(double price) { this.price = price; }

    @Override
    public String toString() {
        return name + " ($" + price + ")";
    }
}
```

#### Restaurant.java
```java
package com.tomato.models;

import java.util.ArrayList;
import java.util.List;

public class Restaurant {
    private static int nextRestaurantId = 0;
    
    private int id;
    private String name;
    private String location;
    private List<MenuItem> menu;

    public Restaurant(String name, String location) {
        this.id = nextRestaurantId++;
        this.name = name;
        this.location = location;
        this.menu = new ArrayList<>();
    }

    public void addMenuItem(MenuItem item) {
        menu.add(item);
    }

    // Getters
    public int getId() { return id; }
    public String getName() { return name; }
    public String getLocation() { return location; }
    public List<MenuItem> getMenu() { return menu; }

    // Setters
    public void setName(String name) { this.name = name; }
    public void setLocation(String location) { this.location = location; }

    @Override
    public String toString() {
        return name + " (" + location + ")";
    }
}
```

#### Cart.java
```java
package com.tomato.models;

import java.util.ArrayList;
import java.util.List;

public class Cart {
    private Restaurant restaurant;
    private List<MenuItem> items;
    private double total;

    public Cart() {
        this.items = new ArrayList<>();
        this.total = 0.0;
    }

    public void addItem(MenuItem item) {
        if (restaurant == null) {
            System.out.println("⚠️ Please select a restaurant first!");
            return;
        }
        items.add(item);
        total += item.getPrice();
        System.out.println("✅ Added: " + item.getName() + " to cart");
    }

    public double getTotalCost() {
        double sum = 0;
        for (MenuItem item : items) {
            sum += item.getPrice();
        }
        return sum;
    }

    public boolean isEmpty() {
        return items.isEmpty();
    }

    public void clear() {
        items.clear();
        total = 0.0;
        restaurant = null;
        System.out.println("🗑️ Cart cleared");
    }

    // Getters
    public Restaurant getRestaurant() { return restaurant; }
    public List<MenuItem> getItems() { return items; }
    public double getTotal() { return total; }

    // Setters
    public void setRestaurant(Restaurant restaurant) { 
        this.restaurant = restaurant; 
    }
}
```

#### User.java
```java
package com.tomato.models;

public class User {
    private int id;
    private String name;
    private String address;
    private Cart cart;

    public User(int id, String name, String address) {
        this.id = id;
        this.name = name;
        this.address = address;
        this.cart = new Cart();
    }

    // Getters
    public int getId() { return id; }
    public String getName() { return name; }
    public String getAddress() { return address; }
    public Cart getCart() { return cart; }

    // Setters
    public void setName(String name) { this.name = name; }
    public void setAddress(String address) { this.address = address; }

    @Override
    public String toString() {
        return name + " (" + address + ")";
    }
}
```

---

### 2. Order Hierarchy

#### Order.java (Abstract Class)
```java
package com.tomato.models;

import com.tomato.strategies.PaymentStrategy;
import java.util.List;

public abstract class Order {
    private static int nextOrderId = 0;
    
    protected int id;
    protected User user;
    protected Restaurant restaurant;
    protected List<MenuItem> items;
    protected PaymentStrategy paymentStrategy;
    protected double total;
    protected String scheduled;

    public Order() {
        this.id = nextOrderId++;
    }

    public void processPayment() {
        if (paymentStrategy != null) {
            paymentStrategy.pay(total);
        } else {
            System.out.println("❌ Please choose payment method first!");
        }
    }

    // Abstract method - must be implemented by subclasses
    public abstract String getType();

    // Getters
    public int getId() { return id; }
    public User getUser() { return user; }
    public Restaurant getRestaurant() { return restaurant; }
    public List<MenuItem> getItems() { return items; }
    public double getTotal() { return total; }
    public String getScheduled() { return scheduled; }
    public PaymentStrategy getPaymentStrategy() { return paymentStrategy; }

    // Setters
    public void setUser(User user) { this.user = user; }
    public void setRestaurant(Restaurant restaurant) { this.restaurant = restaurant; }
    public void setItems(List<MenuItem> items) { this.items = items; }
    public void setPaymentStrategy(PaymentStrategy paymentStrategy) { 
        this.paymentStrategy = paymentStrategy; 
    }
    public void setTotal(double total) { this.total = total; }
    public void setScheduled(String scheduled) { this.scheduled = scheduled; }
}
```

#### DeliveryOrder.java
```java
package com.tomato.models;

public class DeliveryOrder extends Order {
    private String userAddress;

    public DeliveryOrder() {
        super();
        this.userAddress = "";
    }

    @Override
    public String getType() {
        return "DELIVERY";
    }

    // Getters & Setters
    public String getUserAddress() { return userAddress; }
    public void setUserAddress(String userAddress) { 
        this.userAddress = userAddress; 
    }

    @Override
    public String toString() {
        return "DeliveryOrder{" +
                "id=" + id +
                ", type=" + getType() +
                ", deliverTo=" + userAddress +
                ", total=$" + total +
                '}';
    }
}
```

#### PickupOrder.java
```java
package com.tomato.models;

public class PickupOrder extends Order {
    private String restaurantAddress;

    public PickupOrder() {
        super();
        this.restaurantAddress = "";
    }

    @Override
    public String getType() {
        return "PICKUP";
    }

    // Getters & Setters
    public String getRestaurantAddress() { return restaurantAddress; }
    public void setRestaurantAddress(String restaurantAddress) { 
        this.restaurantAddress = restaurantAddress; 
    }

    @Override
    public String toString() {
        return "PickupOrder{" +
                "id=" + id +
                ", type=" + getType() +
                ", pickupFrom=" + restaurantAddress +
                ", total=$" + total +
                '}';
    }
}
```

---

### 3. Manager Classes (Singleton Pattern)

#### RestaurantManager.java
```java
package com.tomato.managers;

import com.tomato.models.Restaurant;
import java.util.ArrayList;
import java.util.List;
import java.util.stream.Collectors;

public class RestaurantManager {
    // Singleton instance
    private static RestaurantManager instance;
    
    private List<Restaurant> restaurants;

    // Private constructor
    private RestaurantManager() {
        this.restaurants = new ArrayList<>();
    }

    // Thread-safe Singleton getInstance
    public static synchronized RestaurantManager getInstance() {
        if (instance == null) {
            instance = new RestaurantManager();
        }
        return instance;
    }

    public void addRestaurant(Restaurant restaurant) {
        restaurants.add(restaurant);
        System.out.println("✅ Restaurant added: " + restaurant.getName());
    }

    public List<Restaurant> searchByLocation(String location) {
        return restaurants.stream()
                .filter(r -> r.getLocation().equalsIgnoreCase(location))
                .collect(Collectors.toList());
    }

    public List<Restaurant> getAllRestaurants() {
        return new ArrayList<>(restaurants);
    }

    public void listAllRestaurants() {
        System.out.println("\n=== All Restaurants ===");
        for (Restaurant r : restaurants) {
            System.out.println("- " + r);
        }
    }
}
```

#### OrderManager.java
```java
package com.tomato.managers;

import com.tomato.models.Order;
import java.util.ArrayList;
import java.util.List;

public class OrderManager {
    // Singleton instance
    private static OrderManager instance;
    
    private List<Order> orders;

    // Private constructor
    private OrderManager() {
        this.orders = new ArrayList<>();
    }

    // Thread-safe Singleton getInstance
    public static synchronized OrderManager getInstance() {
        if (instance == null) {
            instance = new OrderManager();
        }
        return instance;
    }

    public void addOrder(Order order) {
        orders.add(order);
        System.out.println("✅ Order added to system: Order #" + order.getId());
    }

    public void listOrders() {
        System.out.println("\n========== ALL ORDERS ==========");
        if (orders.isEmpty()) {
            System.out.println("No orders yet!");
            return;
        }
        
        for (Order order : orders) {
            System.out.println("Order #" + order.getId() + 
                             " | Type: " + order.getType() + 
                             " | Total: $" + String.format("%.2f", order.getTotal()) +
                             " | Restaurant: " + order.getRestaurant().getName());
        }
        System.out.println("================================\n");
    }

    public List<Order> getAllOrders() {
        return new ArrayList<>(orders);
    }
}
```

---

### 4. Factory Pattern (Order Creation)

#### OrderFactory.java (Interface)
```java
package com.tomato.factories;

import com.tomato.models.*;
import com.tomato.strategies.PaymentStrategy;
import java.util.List;

public interface OrderFactory {
    Order createOrder(
        User user,
        Cart cart,
        Restaurant restaurant,
        List<MenuItem> items,
        PaymentStrategy paymentStrategy,
        String orderType,
        double totalCost
    );
}
```

#### NowOrderFactory.java
```java
package com.tomato.factories;

import com.tomato.models.*;
import com.tomato.strategies.PaymentStrategy;
import com.tomato.utils.TimeUtils;
import java.util.List;

public class NowOrderFactory implements OrderFactory {
    
    @Override
    public Order createOrder(
        User user,
        Cart cart,
        Restaurant restaurant,
        List<MenuItem> items,
        PaymentStrategy paymentStrategy,
        String orderType,
        double totalCost
    ) {
        Order order = null;
        
        // Create appropriate order type
        if (orderType.equalsIgnoreCase("DELIVERY")) {
            DeliveryOrder deliveryOrder = new DeliveryOrder();
            deliveryOrder.setUserAddress(user.getAddress());
            order = deliveryOrder;
        } else if (orderType.equalsIgnoreCase("PICKUP")) {
            PickupOrder pickupOrder = new PickupOrder();
            pickupOrder.setRestaurantAddress(restaurant.getLocation());
            order = pickupOrder;
        }
        
        // Set common order properties
        if (order != null) {
            order.setUser(user);
            order.setRestaurant(restaurant);
            order.setItems(items);
            order.setPaymentStrategy(paymentStrategy);
            order.setScheduled(TimeUtils.getCurrentTime());
            order.setTotal(totalCost);
        }
        
        return order;
    }
}
```

#### ScheduledOrderFactory.java
```java
package com.tomato.factories;

import com.tomato.models.*;
import com.tomato.strategies.PaymentStrategy;
import java.util.List;

public class ScheduledOrderFactory implements OrderFactory {
    
    @Override
    public Order createOrder(
        User user,
        Cart cart,
        Restaurant restaurant,
        List<MenuItem> items,
        PaymentStrategy paymentStrategy,
        String orderType,
        double totalCost
    ) {
        Order order = null;
        
        // Create appropriate order type
        if (orderType.equalsIgnoreCase("DELIVERY")) {
            DeliveryOrder deliveryOrder = new DeliveryOrder();
            deliveryOrder.setUserAddress(user.getAddress());
            order = deliveryOrder;
        } else if (orderType.equalsIgnoreCase("PICKUP")) {
            PickupOrder pickupOrder = new PickupOrder();
            pickupOrder.setRestaurantAddress(restaurant.getLocation());
            order = pickupOrder;
        }
        
        // Set common order properties
        if (order != null) {
            order.setUser(user);
            order.setRestaurant(restaurant);
            order.setItems(items);
            order.setPaymentStrategy(paymentStrategy);
            order.setScheduled("2026-01-21 18:00:00"); // Scheduled for later
            order.setTotal(totalCost);
        }
        
        return order;
    }
}
```

---

### 5. Strategy Pattern (Payment Processing)

#### PaymentStrategy.java (Interface)
```java
package com.tomato.strategies;

public interface PaymentStrategy {
    void pay(double amount);
}
```

#### UPIPayment.java
```java
package com.tomato.strategies;

public class UPIPayment implements PaymentStrategy {
    private String mobileNumber;

    public UPIPayment(String mobileNumber) {
        this.mobileNumber = mobileNumber;
    }

    @Override
    public void pay(double amount) {
        System.out.println("💳 Paid $" + String.format("%.2f", amount) + 
                         " using UPI: " + mobileNumber);
    }

    public String getMobileNumber() {
        return mobileNumber;
    }
}
```

#### CreditCardPayment.java
```java
package com.tomato.strategies;

public class CreditCardPayment implements PaymentStrategy {
    private String cardNumber;

    public CreditCardPayment(String cardNumber) {
        this.cardNumber = cardNumber;
    }

    @Override
    public void pay(double amount) {
        System.out.println("💳 Paid $" + String.format("%.2f", amount) + 
                         " using Credit Card: **** **** **** " + 
                         cardNumber.substring(cardNumber.length() - 4));
    }

    public String getCardNumber() {
        return cardNumber;
    }
}
```

#### NetBankingPayment.java
```java
package com.tomato.strategies;

public class NetBankingPayment implements PaymentStrategy {
    private String bankName;
    private String accountNumber;

    public NetBankingPayment(String bankName, String accountNumber) {
        this.bankName = bankName;
        this.accountNumber = accountNumber;
    }

    @Override
    public void pay(double amount) {
        System.out.println("💳 Paid $" + String.format("%.2f", amount) + 
                         " using Net Banking (" + bankName + "): " + 
                         "****" + accountNumber.substring(accountNumber.length() - 4));
    }
}
```

---

### 6. Services

#### NotificationService.java
```java
package com.tomato.services;

import com.tomato.models.Order;

public class NotificationService {
    
    public void notify(Order order) {
        System.out.println("\n" + "=".repeat(50));
        System.out.println("📧 NOTIFICATION SENT 📧");
        System.out.println("=".repeat(50));
        System.out.println("Order ID: #" + order.getId());
        System.out.println("Customer: " + order.getUser().getName());
        System.out.println("Restaurant: " + order.getRestaurant().getName());
        System.out.println("Order Type: " + order.getType());
        System.out.println("Total Amount: $" + String.format("%.2f", order.getTotal()));
        System.out.println("Scheduled: " + order.getScheduled());
        System.out.println("\n✅ Your order has been placed successfully!");
        System.out.println("=".repeat(50) + "\n");
    }
}
```

---

### 7. Utility Classes

#### TimeUtils.java
```java
package com.tomato.utils;

import java.time.LocalDateTime;
import java.time.format.DateTimeFormatter;

public class TimeUtils {
    private static final DateTimeFormatter formatter = 
        DateTimeFormatter.ofPattern("yyyy-MM-dd HH:mm:ss");

    public static String getCurrentTime() {
        return LocalDateTime.now().format(formatter);
    }
}
```

---

### 8. Orchestrator (TomatoApp)

#### TomatoApp.java
```java
package com.tomato;

import com.tomato.managers.RestaurantManager;
import com.tomato.managers.OrderManager;
import com.tomato.services.NotificationService;
import com.tomato.factories.*;
import com.tomato.models.*;
import com.tomato.strategies.PaymentStrategy;

import java.util.List;

public class TomatoApp {
    private RestaurantManager restaurantManager;
    private OrderManager orderManager;
    private NotificationService notificationService;

    public TomatoApp() {
        this.restaurantManager = RestaurantManager.getInstance();
        this.orderManager = OrderManager.getInstance();
        this.notificationService = new NotificationService();
        initializeRestaurants();
    }

    private void initializeRestaurants() {
        System.out.println("🍽️ Initializing restaurants...\n");
        
        // Restaurant 1: Pizza Hut
        Restaurant pizzaHut = new Restaurant("Pizza Hut", "Delhi");
        pizzaHut.addMenuItem(new MenuItem("P001", "Margherita Pizza", 12.99));
        pizzaHut.addMenuItem(new MenuItem("P002", "Pepperoni Pizza", 14.99));
        pizzaHut.addMenuItem(new MenuItem("P003", "Veggie Supreme", 13.99));
        
        // Restaurant 2: Burger King
        Restaurant burgerKing = new Restaurant("Burger King", "Delhi");
        burgerKing.addMenuItem(new MenuItem("B001", "Whopper", 8.99));
        burgerKing.addMenuItem(new MenuItem("B002", "Chicken Burger", 7.99));
        burgerKing.addMenuItem(new MenuItem("B003", "Veggie Burger", 6.99));
        
        // Restaurant 3: Subway
        Restaurant subway = new Restaurant("Subway", "Mumbai");
        subway.addMenuItem(new MenuItem("S001", "Veggie Delite Sub", 6.99));
        subway.addMenuItem(new MenuItem("S002", "Chicken Teriyaki", 8.99));
        
        // Restaurant 4: Domino's
        Restaurant dominos = new Restaurant("Domino's Pizza", "Delhi");
        dominos.addMenuItem(new MenuItem("D001", "Farmhouse Pizza", 11.99));
        dominos.addMenuItem(new MenuItem("D002", "Chicken Dominator", 15.99));
        
        restaurantManager.addRestaurant(pizzaHut);
        restaurantManager.addRestaurant(burgerKing);
        restaurantManager.addRestaurant(subway);
        restaurantManager.addRestaurant(dominos);
        
        System.out.println();
    }

    public List<Restaurant> searchRestaurants(String location) {
        return restaurantManager.searchByLocation(location);
    }

    public Order placeOrder(
        User user,
        PaymentStrategy paymentStrategy,
        String orderType,
        boolean scheduleForLater
    ) {
        Cart cart = user.getCart();
        
        if (cart.isEmpty()) {
            System.out.println("❌ Cart is empty! Please add items first.");
            return null;
        }
        
        // Select appropriate factory
        OrderFactory factory;
        if (scheduleForLater) {
            factory = new ScheduledOrderFactory();
            System.out.println("📅 Creating SCHEDULED order...");
        } else {
            factory = new NowOrderFactory();
            System.out.println("⚡ Creating IMMEDIATE order...");
        }
        
        // Create order
        Order order = factory.createOrder(
            user,
            cart,
            cart.getRestaurant(),
            cart.getItems(),
            paymentStrategy,
            orderType,
            cart.getTotalCost()
        );
        
        if (order == null) {
            System.out.println("❌ Failed to create order!");
            return null;
        }
        
        // Process payment
        System.out.println("\n💰 Processing payment...");
        order.processPayment();
        
        // Add to order manager
        orderManager.addOrder(order);
        
        // Send notification
        notificationService.notify(order);
        
        // Clear cart
        cart.clear();
        
        return order;
    }

    public void listAllOrders() {
        orderManager.listOrders();
    }

    public void listAllRestaurants() {
        restaurantManager.listAllRestaurants();
    }
}
```

---

### 9. Main Driver Code

#### Main.java
```java
package com.tomato;

import com.tomato.models.*;
import com.tomato.strategies.*;
import java.util.List;

public class Main {
    public static void main(String[] args) {
        System.out.println("🍅 Welcome to TOMATO - Food Delivery App 🍅\n");
        System.out.println("=".repeat(60));
        
        // Initialize the application
        TomatoApp app = new TomatoApp();
        
        // Create a user
        User user = new User(1, "Anubhav Garg", "123 Main Street, Connaught Place, Delhi");
        System.out.println("👤 User created: " + user + "\n");
        
        // Search restaurants in Delhi
        System.out.println("🔍 Searching for restaurants in Delhi...\n");
        List<Restaurant> restaurants = app.searchRestaurants("Delhi");
        
        System.out.println("Found " + restaurants.size() + " restaurants:");
        for (int i = 0; i < restaurants.size(); i++) {
            Restaurant r = restaurants.get(i);
            System.out.println((i + 1) + ". " + r.getName() + " - " + r.getLocation());
        }
        System.out.println();
        
        // ========== ORDER 1: DELIVERY ORDER (NOW) ==========
        System.out.println("=".repeat(60));
        System.out.println("📦 ORDER 1: DELIVERY ORDER (IMMEDIATE)");
        System.out.println("=".repeat(60));
        
        // Select Pizza Hut
        Restaurant pizzaHut = restaurants.get(0);
        user.getCart().setRestaurant(pizzaHut);
        System.out.println("🍕 Selected restaurant: " + pizzaHut.getName() + "\n");
        
        // Add items to cart
        System.out.println("🛒 Adding items to cart:");
        List<MenuItem> pizzaMenu = pizzaHut.getMenu();
        user.getCart().addItem(pizzaMenu.get(0)); // Margherita Pizza
        user.getCart().addItem(pizzaMenu.get(1)); // Pepperoni Pizza
        
        System.out.println("\n💵 Cart Total: $" + 
                         String.format("%.2f", user.getCart().getTotalCost()));
        
        // Place order with UPI payment
        PaymentStrategy upiPayment = new UPIPayment("9876543210");
        Order order1 = app.placeOrder(user, upiPayment, "DELIVERY", false);
        
        System.out.println("\n" + "=".repeat(60) + "\n");
        
        // ========== ORDER 2: PICKUP ORDER (SCHEDULED) ==========
        System.out.println("=".repeat(60));
        System.out.println("📦 ORDER 2: PICKUP ORDER (SCHEDULED)");
        System.out.println("=".repeat(60));
        
        // Select Burger King
        Restaurant burgerKing = restaurants.get(1);
        user.getCart().setRestaurant(burgerKing);
        System.out.println("🍔 Selected restaurant: " + burgerKing.getName() + "\n");
        
        // Add items to cart
        System.out.println("🛒 Adding items to cart:");
        List<MenuItem> burgerMenu = burgerKing.getMenu();
        user.getCart().addItem(burgerMenu.get(0)); // Whopper
        user.getCart().addItem(burgerMenu.get(2)); // Veggie Burger
        
        System.out.println("\n💵 Cart Total: $" + 
                         String.format("%.2f", user.getCart().getTotalCost()));
        
        // Place scheduled pickup order with Credit Card
        PaymentStrategy cardPayment = new CreditCardPayment("1234-5678-9012-3456");
        Order order2 = app.placeOrder(user, cardPayment, "PICKUP", true);
        
        System.out.println("\n" + "=".repeat(60) + "\n");
        
        // ========== ORDER 3: DELIVERY ORDER WITH NET BANKING ==========
        System.out.println("=".repeat(60));
        System.out.println("📦 ORDER 3: DELIVERY ORDER (NET BANKING)");
        System.out.println("=".repeat(60));
        
        // Select Domino's
        Restaurant dominos = restaurants.get(3);
        user.getCart().setRestaurant(dominos);
        System.out.println("🍕 Selected restaurant: " + dominos.getName() + "\n");
        
        // Add items to cart
        System.out.println("🛒 Adding items to cart:");
        List<MenuItem> dominosMenu = dominos.getMenu();
        user.getCart().addItem(dominosMenu.get(0)); // Farmhouse Pizza
        user.getCart().addItem(dominosMenu.get(1)); // Chicken Dominator
        
        System.out.println("\n💵 Cart Total: $" + 
                         String.format("%.2f", user.getCart().getTotalCost()));
        
        // Place order with Net Banking
        PaymentStrategy netBanking = new NetBankingPayment("HDFC Bank", "9876543210");
        Order order3 = app.placeOrder(user, netBanking, "DELIVERY", false);
        
        System.out.println("\n" + "=".repeat(60) + "\n");
        
        // ========== DISPLAY ALL ORDERS ==========
        app.listAllOrders();
        
        // ========== DISPLAY ALL RESTAURANTS ==========
        app.listAllRestaurants();
        
        System.out.println("\n🎉 Thank you for using TOMATO! 🍅");
    }
}
```

---

### 10. Expected Output

```
🍅 Welcome to TOMATO - Food Delivery App 🍅

============================================================
🍽️ Initializing restaurants...

✅ Restaurant added: Pizza Hut
✅ Restaurant added: Burger King
✅ Restaurant added: Subway
✅ Restaurant added: Domino's Pizza

👤 User created: Anubhav Garg (123 Main Street, Connaught Place, Delhi)

🔍 Searching for restaurants in Delhi...

Found 3 restaurants:
1. Pizza Hut - Delhi
2. Burger King - Delhi
3. Domino's Pizza - Delhi

============================================================
📦 ORDER 1: DELIVERY ORDER (IMMEDIATE)
============================================================
🍕 Selected restaurant: Pizza Hut

🛒 Adding items to cart:
✅ Added: Margherita Pizza to cart
✅ Added: Pepperoni Pizza to cart

💵 Cart Total: $27.98
⚡ Creating IMMEDIATE order...

💰 Processing payment...
💳 Paid $27.98 using UPI: 9876543210
✅ Order added to system: Order #0

==================================================
📧 NOTIFICATION SENT 📧
==================================================
Order ID: #0
Customer: Anubhav Garg
Restaurant: Pizza Hut
Order Type: DELIVERY
Total Amount: $27.98
Scheduled: 2026-01-20 23:56:03

✅ Your order has been placed successfully!
==================================================

🗑️ Cart cleared

============================================================

============================================================
📦 ORDER 2: PICKUP ORDER (SCHEDULED)
============================================================
🍔 Selected restaurant: Burger King

🛒 Adding items to cart:
✅ Added: Whopper to cart
✅ Added: Veggie Burger to cart

💵 Cart Total: $15.98
📅 Creating SCHEDULED order...

💰 Processing payment...
💳 Paid $15.98 using Credit Card: **** **** **** 3456
✅ Order added to system: Order #1

==================================================
📧 NOTIFICATION SENT 📧
==================================================
Order ID: #1
Customer: Anubhav Garg
Restaurant: Burger King
Order Type: PICKUP
Total Amount: $15.98
Scheduled: 2026-01-21 18:00:00

✅ Your order has been placed successfully!
==================================================

🗑️ Cart cleared

============================================================

========== ALL ORDERS ==========
Order #0 | Type: DELIVERY | Total: $27.98 | Restaurant: Pizza Hut
Order #1 | Type: PICKUP | Total: $15.98 | Restaurant: Burger King
Order #2 | Type: DELIVERY | Total: $27.98 | Restaurant: Domino's Pizza
================================

=== All Restaurants ===
- Pizza Hut (Delhi)
- Burger King (Delhi)
- Subway (Mumbai)
- Domino's Pizza (Delhi)

🎉 Thank you for using TOMATO! 🍅
```

---

## 🔄 Design Patterns Used

### 1. **Singleton Pattern**
- **Where:** RestaurantManager, OrderManager
- **Why:** Single source of truth for all restaurants/orders
- **Benefit:** Prevents multiple instances maintaining different states

### 2. **Factory Method Pattern**
- **Where:** OrderFactory → NowOrderFactory, ScheduledOrderFactory
- **Why:** Different ways to create orders (now vs scheduled)
- **Benefit:** Easy to add new order creation strategies

### 3. **Strategy Pattern**
- **Where:** PaymentStrategy → UPIPayment, CreditCardPayment, NetBanking
- **Why:** Multiple payment methods
- **Benefit:** Client doesn't need to know payment implementation details

### 4. **Template Method Pattern** (Implicit)
- **Where:** Order (abstract) → DeliveryOrder, PickupOrder
- **Why:** Common order structure with type-specific variations

---

## 🎯 When to Use / When NOT to Use

### ✅ Use This Design When:
1. Building **e-commerce or food delivery** platforms
2. Need **multiple payment methods** that can change at runtime
3. Require **different order types** (delivery, pickup, dine-in)
4. Want to **decouple** order creation from order management
5. Need **extensibility** for new features (scheduled orders, new payment methods)

### ❌ Avoid This Design When:
1. Building a **simple, single-restaurant** ordering system
2. Only **one payment method** is supported
3. **No future extensibility** is needed
4. **Performance is critical** (too many layers of abstraction)

---

## ⚖️ Pros and Cons

| Advantages ✅ | Disadvantages ❌ |
|--------------|------------------|
| **Highly extensible** - Easy to add new order types, payment methods | **Complex** - Many classes and relationships |
| **Loose coupling** - Components interact through interfaces | **Over-engineering** for small apps |
| **Single Responsibility** - Each class has one clear purpose | **Orchestrator violates SRP** (TomatoApp knows too much) |
| **Testable** - Can mock managers, factories, strategies | **Memory overhead** - Multiple singleton instances |
| **Follows SOLID principles** (mostly) | **Principle of Least Knowledge** violated in some places |

---

## 🔍 Real-World Examples

### Industry Usage:
1. **Zomato/Swiggy** - Actual food delivery platforms
2. **Amazon** - Order management and payment processing
3. **Uber Eats** - Restaurant and delivery management
4. **Spring Framework** - Uses Factory and Strategy patterns extensively
5. **Payment Gateways** (Razorpay, Stripe) - Strategy pattern for different payment methods

---

## 🎯 Key Takeaways

1. **Bottom-Up Design** works well when you can identify small entities first
2. **Managers are Singletons** to maintain consistent state
3. **Factory Pattern** separates object creation from business logic
4. **Strategy Pattern** enables runtime selection of algorithms
5. **Orchestrator** provides clean API but violates some SOLID principles (acceptable trade-off)
6. **Design Patterns aren't rules** - use them when they solve real problems
7. **Principle of Least Knowledge** - Pass only what's needed (avoid tight coupling)
8. **Trade-offs are inevitable** - Balance between simplicity and extensibility

> [!important] Interview Tip
> Always discuss trade-offs with your interviewer. Show that you understand:
> - Why you chose a particular design
> - What principles you're following/violating
> - Alternative approaches and their pros/cons

---

## 🔄 Comparison with Related Patterns

### Factory vs Builder Pattern
| Factory Pattern | Builder Pattern |
|----------------|-----------------|
| Creates objects in one step | Creates objects step-by-step |
| Used when creation logic varies | Used when object has many optional parameters |
| Returns different types (DeliveryOrder, PickupOrder) | Returns same type with different configurations |

### Strategy vs State Pattern
| Strategy Pattern | State Pattern |
|-----------------|---------------|
| Behavior selected by client | Behavior changes based on internal state |
| PaymentStrategy chosen at runtime | Order state changes (Pending → Confirmed → Delivered) |
| Context doesn't change | Context changes state |

---

## 📝 Additional Notes

### Design Improvements (Discussion Points)

**Q: Why pass so many parameters to `createOrder()`?**
- **Current:** Pass user, cart, restaurant, items, payment, type, total
- **Alternative:** Pass only user, payment, type (extract rest from user.cart)
- **Trade-off:** Current approach follows **Principle of Least Knowledge** (factory doesn't need to know cart structure)

**Q: Why have both OrderManager and OrderFactory?**
- **Separation of Concerns:** Factory creates, Manager manages (CRUD operations)
- **Alternative:** Combine into one class
- **Trade-off:** Current approach is more modular but adds complexity

**Q: Is TomatoApp a good design?**
- **Pros:** Single point of contact for client (clean API)
- **Cons:** Violates SRP and Principle of Least Knowledge
- **Alternative:** Use **Facade Pattern** or **Mediator Pattern** for better design

---

## 🚀 Extension Ideas

1. **Add Coupon/Discount System** (Decorator Pattern)
2. **Implement Order States** (State Pattern: Pending → Confirmed → Preparing → Delivered)
3. **Add Observer Pattern** for real-time order tracking
4. **Implement Repository Pattern** for database operations
5. **Add Authentication/Authorization** (Chain of Responsibility)
6. **Implement Caching** for restaurant search (Proxy Pattern)

---

**End of Notes** 📚
