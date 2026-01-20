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

### 1. Model Classes

#### MenuItem.h
```cpp
#ifndef MENUITEM_H
#define MENUITEM_H

#include <string>
using namespace std;

class MenuItem {
private:
    string code;
    string name;
    double price;

public:
    MenuItem(string code, string name, double price) 
        : code(code), name(name), price(price) {}
    
    // Getters
    string getCode() const { return code; }
    string getName() const { return name; }
    double getPrice() const { return price; }
    
    // Setters
    void setCode(string c) { code = c; }
    void setName(string n) { name = n; }
    void setPrice(double p) { price = p; }
};

#endif
```

#### Restaurant.h
```cpp
#ifndef RESTAURANT_H
#define RESTAURANT_H

#include <string>
#include <vector>
#include "MenuItem.h"
using namespace std;

class Restaurant {
private:
    static int nextRestaurantId;
    int id;
    string name;
    string location;
    vector<MenuItem*> menu;

public:
    Restaurant(string name, string location) 
        : id(nextRestaurantId++), name(name), location(location) {}
    
    ~Restaurant() {
        for(auto item : menu) delete item;
    }
    
    // Getters
    int getId() const { return id; }
    string getName() const { return name; }
    string getLocation() const { return location; }
    vector<MenuItem*> getMenu() const { return menu; }
    
    // Setters
    void setName(string n) { name = n; }
    void setLocation(string loc) { location = loc; }
    void addMenuItem(MenuItem* item) { menu.push_back(item); }
};

int Restaurant::nextRestaurantId = 0;

#endif
```

#### Cart.h
```cpp
#ifndef CART_H
#define CART_H

#include <vector>
#include "Restaurant.h"
#include "MenuItem.h"
using namespace std;

class Cart {
private:
    Restaurant* restaurant;
    vector<MenuItem*> items;
    double total;

public:
    Cart() : restaurant(nullptr), total(0.0) {}
    
    void addItem(MenuItem* item) {
        if (restaurant == nullptr) {
            cout << "Please select a restaurant first!" << endl;
            return;
        }
        items.push_back(item);
        total += item->getPrice();
    }
    
    double getTotalCost() const {
        double sum = 0;
        for(auto item : items) {
            sum += item->getPrice();
        }
        return sum;
    }
    
    bool isEmpty() const { return items.empty(); }
    
    void clear() {
        items.clear();
        total = 0.0;
        restaurant = nullptr;
    }
    
    // Getters/Setters
    Restaurant* getRestaurant() const { return restaurant; }
    void setRestaurant(Restaurant* r) { restaurant = r; }
    vector<MenuItem*> getItems() const { return items; }
};

#endif
```

#### User.h
```cpp
#ifndef USER_H
#define USER_H

#include <string>
#include "Cart.h"
using namespace std;

class User {
private:
    int id;
    string name;
    string address;
    Cart* cart;

public:
    User(int id, string name, string address) 
        : id(id), name(name), address(address) {
        cart = new Cart();
    }
    
    ~User() { delete cart; }
    
    // Getters
    int getId() const { return id; }
    string getName() const { return name; }
    string getAddress() const { return address; }
    Cart* getCart() const { return cart; }
    
    // Setters
    void setName(string n) { name = n; }
    void setAddress(string addr) { address = addr; }
};

#endif
```

### 2. Order Hierarchy

#### Order.h (Abstract)
```cpp
#ifndef ORDER_H
#define ORDER_H

#include <string>
#include <vector>
#include "User.h"
#include "Restaurant.h"
#include "MenuItem.h"
#include "../strategies/PaymentStrategy.h"
using namespace std;

class Order {
protected:
    static int nextOrderId;
    int id;
    User* user;
    Restaurant* restaurant;
    vector<MenuItem*> items;
    PaymentStrategy* paymentStrategy;
    double total;
    string scheduled;

public:
    Order() : id(nextOrderId++), user(nullptr), 
              restaurant(nullptr), paymentStrategy(nullptr), 
              total(0.0) {}
    
    virtual ~Order() {}
    
    void processPayment() {
        if (paymentStrategy) {
            paymentStrategy->pay(total);
        } else {
            cout << "Please choose payment method first!" << endl;
        }
    }
    
    // Virtual method to be overridden
    virtual string getType() const = 0;
    
    // Getters/Setters
    int getId() const { return id; }
    User* getUser() const { return user; }
    Restaurant* getRestaurant() const { return restaurant; }
    vector<MenuItem*> getItems() const { return items; }
    double getTotal() const { return total; }
    string getScheduled() const { return scheduled; }
    
    void setUser(User* u) { user = u; }
    void setRestaurant(Restaurant* r) { restaurant = r; }
    void setItems(vector<MenuItem*> i) { items = i; }
    void setPaymentStrategy(PaymentStrategy* ps) { paymentStrategy = ps; }
    void setTotal(double t) { total = t; }
    void setScheduled(string s) { scheduled = s; }
};

int Order::nextOrderId = 0;

#endif
```

#### DeliveryOrder.h
```cpp
#ifndef DELIVERYORDER_H
#define DELIVERYORDER_H

#include "Order.h"

class DeliveryOrder : public Order {
private:
    string userAddress;

public:
    DeliveryOrder() : Order(), userAddress("") {}
    
    string getType() const override {
        return "DELIVERY";
    }
    
    string getUserAddress() const { return userAddress; }
    void setUserAddress(string addr) { userAddress = addr; }
};

#endif
```

#### PickupOrder.h
```cpp
#ifndef PICKUPORDER_H
#define PICKUPORDER_H

#include "Order.h"

class PickupOrder : public Order {
private:
    string restaurantAddress;

public:
    PickupOrder() : Order(), restaurantAddress("") {}
    
    string getType() const override {
        return "PICKUP";
    }
    
    string getRestaurantAddress() const { return restaurantAddress; }
    void setRestaurantAddress(string addr) { restaurantAddress = addr; }
};

#endif
```

### 3. Manager Classes (Singleton)

#### RestaurantManager.h
```cpp
#ifndef RESTAURANTMANAGER_H
#define RESTAURANTMANAGER_H

#include <vector>
#include "../models/Restaurant.h"
using namespace std;

class RestaurantManager {
private:
    static RestaurantManager* instance;
    vector<Restaurant*> restaurants;
    
    // Private constructor for Singleton
    RestaurantManager() {}

public:
    static RestaurantManager* getInstance() {
        if (instance == nullptr) {
            instance = new RestaurantManager();
        }
        return instance;
    }
    
    void addRestaurant(Restaurant* r) {
        restaurants.push_back(r);
    }
    
    vector<Restaurant*> searchByLocation(string location) {
        vector<Restaurant*> result;
        for (auto restaurant : restaurants) {
            if (restaurant->getLocation() == location) {
                result.push_back(restaurant);
            }
        }
        return result;
    }
    
    vector<Restaurant*> getAllRestaurants() const {
        return restaurants;
    }
};

RestaurantManager* RestaurantManager::instance = nullptr;

#endif
```

#### OrderManager.h
```cpp
#ifndef ORDERMANAGER_H
#define ORDERMANAGER_H

#include <vector>
#include <iostream>
#include "../models/Order.h"
using namespace std;

class OrderManager {
private:
    static OrderManager* instance;
    vector<Order*> orders;
    
    OrderManager() {}

public:
    static OrderManager* getInstance() {
        if (instance == nullptr) {
            instance = new OrderManager();
        }
        return instance;
    }
    
    void addOrder(Order* order) {
        orders.push_back(order);
    }
    
    void listOrders() {
        cout << "\n=== All Orders ===" << endl;
        for (auto order : orders) {
            cout << "Order ID: " << order->getId() 
                 << " | Type: " << order->getType()
                 << " | Total: $" << order->getTotal() << endl;
        }
    }
};

OrderManager* OrderManager::instance = nullptr;

#endif
```

### 4. Factory Pattern (Order Creation)

#### OrderFactory.h (Interface)
```cpp
#ifndef ORDERFACTORY_H
#define ORDERFACTORY_H

#include "../models/Order.h"
#include "../models/User.h"
#include "../models/Cart.h"
#include "../models/Restaurant.h"
#include "../strategies/PaymentStrategy.h"
#include <string>
using namespace std;

class OrderFactory {
public:
    virtual Order* createOrder(
        User* user,
        Cart* cart,
        Restaurant* restaurant,
        vector<MenuItem*> items,
        PaymentStrategy* paymentStrategy,
        string orderType,
        double totalCost
    ) = 0;
    
    virtual ~OrderFactory() {}
};

#endif
```

#### NowOrderFactory.h
```cpp
#ifndef NOWORDERFACTORY_H
#define NOWORDERFACTORY_H

#include "OrderFactory.h"
#include "../models/DeliveryOrder.h"
#include "../models/PickupOrder.h"
#include "../utils/TimeUtils.h"

class NowOrderFactory : public OrderFactory {
public:
    Order* createOrder(
        User* user,
        Cart* cart,
        Restaurant* restaurant,
        vector<MenuItem*> items,
        PaymentStrategy* paymentStrategy,
        string orderType,
        double totalCost
    ) override {
        Order* order = nullptr;
        
        if (orderType == "DELIVERY") {
            DeliveryOrder* deliveryOrder = new DeliveryOrder();
            deliveryOrder->setUserAddress(user->getAddress());
            order = deliveryOrder;
        } else {
            PickupOrder* pickupOrder = new PickupOrder();
            pickupOrder->setRestaurantAddress(restaurant->getLocation());
            order = pickupOrder;
        }
        
        // Set common order properties
        order->setUser(user);
        order->setRestaurant(restaurant);
        order->setItems(items);
        order->setPaymentStrategy(paymentStrategy);
        order->setScheduled(TimeUtils::getCurrentTime());
        order->setTotal(totalCost);
        
        return order;
    }
};

#endif
```

#### ScheduledOrderFactory.h
```cpp
#ifndef SCHEDULEDORDERFACTORY_H
#define SCHEDULEDORDERFACTORY_H

#include "OrderFactory.h"
#include "../models/DeliveryOrder.h"
#include "../models/PickupOrder.h"

class ScheduledOrderFactory : public OrderFactory {
public:
    Order* createOrder(
        User* user,
        Cart* cart,
        Restaurant* restaurant,
        vector<MenuItem*> items,
        PaymentStrategy* paymentStrategy,
        string orderType,
        double totalCost
    ) override {
        Order* order = nullptr;
        
        if (orderType == "DELIVERY") {
            DeliveryOrder* deliveryOrder = new DeliveryOrder();
            deliveryOrder->setUserAddress(user->getAddress());
            order = deliveryOrder;
        } else {
            PickupOrder* pickupOrder = new PickupOrder();
            pickupOrder->setRestaurantAddress(restaurant->getLocation());
            order = pickupOrder;
        }
        
        order->setUser(user);
        order->setRestaurant(restaurant);
        order->setItems(items);
        order->setPaymentStrategy(paymentStrategy);
        order->setScheduled("2026-01-21 18:00"); // Scheduled for later
        order->setTotal(totalCost);
        
        return order;
    }
};

#endif
```

### 5. Strategy Pattern (Payment)

#### PaymentStrategy.h (Interface)
```cpp
#ifndef PAYMENTSTRATEGY_H
#define PAYMENTSTRATEGY_H

class PaymentStrategy {
public:
    virtual void pay(double amount) = 0;
    virtual ~PaymentStrategy() {}
};

#endif
```

#### UPIPayment.h
```cpp
#ifndef UPIPAYMENT_H
#define UPIPAYMENT_H

#include "PaymentStrategy.h"
#include <iostream>
#include <string>
using namespace std;

class UPIPayment : public PaymentStrategy {
private:
    string mobileNumber;

public:
    UPIPayment(string mobile) : mobileNumber(mobile) {}
    
    void pay(double amount) override {
        cout << "Paid $" << amount << " using UPI: " 
             << mobileNumber << endl;
    }
};

#endif
```

#### CreditCardPayment.h
```cpp
#ifndef CREDITCARDPAYMENT_H
#define CREDITCARDPAYMENT_H

#include "PaymentStrategy.h"
#include <iostream>
#include <string>
using namespace std;

class CreditCardPayment : public PaymentStrategy {
private:
    string cardNumber;

public:
    CreditCardPayment(string card) : cardNumber(card) {}
    
    void pay(double amount) override {
        cout << "Paid $" << amount << " using Credit Card: " 
             << cardNumber << endl;
    }
};

#endif
```

### 6. Services

#### NotificationService.h
```cpp
#ifndef NOTIFICATIONSERVICE_H
#define NOTIFICATIONSERVICE_H

#include "../models/Order.h"
#include <iostream>
using namespace std;

class NotificationService {
public:
    void notify(Order* order) {
        cout << "\n📧 NOTIFICATION SENT 📧" << endl;
        cout << "Order ID: " << order->getId() << endl;
        cout << "User: " << order->getUser()->getName() << endl;
        cout << "Restaurant: " << order->getRestaurant()->getName() << endl;
        cout << "Type: " << order->getType() << endl;
        cout << "Total: $" << order->getTotal() << endl;
        cout << "Scheduled: " << order->getScheduled() << endl;
        cout << "Your order has been placed successfully!" << endl;
    }
};

#endif
```

### 7. Orchestrator (TomatoApp)

#### TomatoApp.h
```cpp
#ifndef TOMATOAPP_H
#define TOMATOAPP_H

#include "managers/RestaurantManager.h"
#include "managers/OrderManager.h"
#include "services/NotificationService.h"
#include "factories/NowOrderFactory.h"
#include "factories/ScheduledOrderFactory.h"
#include <iostream>
using namespace std;

class TomatoApp {
private:
    RestaurantManager* restaurantManager;
    OrderManager* orderManager;
    NotificationService* notificationService;
    
    void initializeRestaurants() {
        // Sample restaurants
        Restaurant* r1 = new Restaurant("Pizza Hut", "Delhi");
        r1->addMenuItem(new MenuItem("P001", "Margherita Pizza", 12.99));
        r1->addMenuItem(new MenuItem("P002", "Pepperoni Pizza", 14.99));
        
        Restaurant* r2 = new Restaurant("Burger King", "Delhi");
        r2->addMenuItem(new MenuItem("B001", "Whopper", 8.99));
        r2->addMenuItem(new MenuItem("B002", "Chicken Burger", 7.99));
        
        Restaurant* r3 = new Restaurant("Subway", "Mumbai");
        r3->addMenuItem(new MenuItem("S001", "Veggie Sub", 6.99));
        
        restaurantManager->addRestaurant(r1);
        restaurantManager->addRestaurant(r2);
        restaurantManager->addRestaurant(r3);
    }

public:
    TomatoApp() {
        restaurantManager = RestaurantManager::getInstance();
        orderManager = OrderManager::getInstance();
        notificationService = new NotificationService();
        initializeRestaurants();
    }
    
    vector<Restaurant*> searchRestaurants(string location) {
        return restaurantManager->searchByLocation(location);
    }
    
    Order* placeOrder(
        User* user,
        PaymentStrategy* paymentStrategy,
        string orderType,
        bool scheduleForLater = false
    ) {
        Cart* cart = user->getCart();
        
        if (cart->isEmpty()) {
            cout << "Cart is empty!" << endl;
            return nullptr;
        }
        
        OrderFactory* factory;
        if (scheduleForLater) {
            factory = new ScheduledOrderFactory();
        } else {
            factory = new NowOrderFactory();
        }
        
        Order* order = factory->createOrder(
            user,
            cart,
            cart->getRestaurant(),
            cart->getItems(),
            paymentStrategy,
            orderType,
            cart->getTotalCost()
        );
        
        // Process payment
        order->processPayment();
        
        // Add to order manager
        orderManager->addOrder(order);
        
        // Send notification
        notificationService->notify(order);
        
        // Clear cart
        cart->clear();
        
        delete factory;
        return order;
    }
    
    void listAllOrders() {
        orderManager->listOrders();
    }
};

#endif
```

### 8. Utility Classes

#### TimeUtils.h
```cpp
#ifndef TIMEUTILS_H
#define TIMEUTILS_H

#include <string>
#include <ctime>
using namespace std;

class TimeUtils {
public:
    static string getCurrentTime() {
        time_t now = time(0);
        char* dt = ctime(&now);
        return string(dt);
    }
};

#endif
```

### 9. Main Driver Code

```cpp
#include "TomatoApp.h"
#include "strategies/UPIPayment.h"
#include "strategies/CreditCardPayment.h"
#include <iostream>
using namespace std;

int main() {
    // Initialize the app
    TomatoApp* app = new TomatoApp();
    
    // Create a user
    User* user = new User(1, "Anubhav", "123 Main St, Delhi");
    
    // Search restaurants in Delhi
    cout << "🔍 Searching restaurants in Delhi..." << endl;
    vector<Restaurant*> restaurants = app->searchRestaurants("Delhi");
    
    cout << "\nFound " << restaurants.size() << " restaurants:" << endl;
    for (auto r : restaurants) {
        cout << "- " << r->getName() << " (" << r->getLocation() << ")" << endl;
    }
    
    // Select a restaurant and add items to cart
    Restaurant* selectedRestaurant = restaurants[0];
    user->getCart()->setRestaurant(selectedRestaurant);
    
    cout << "\n🍕 Adding items to cart from " 
         << selectedRestaurant->getName() << endl;
    
    vector<MenuItem*> menu = selectedRestaurant->getMenu();
    user->getCart()->addItem(menu[0]); // Margherita Pizza
    user->getCart()->addItem(menu[1]); // Pepperoni Pizza
    
    cout << "Cart Total: $" << user->getCart()->getTotalCost() << endl;
    
    // Place order with UPI payment
    cout << "\n📦 Placing DELIVERY order..." << endl;
    PaymentStrategy* upiPayment = new UPIPayment("9876543210");
    Order* order1 = app->placeOrder(user, upiPayment, "DELIVERY", false);
    
    // Add more items for second order
    user->getCart()->setRestaurant(restaurants[1]);
    user->getCart()->addItem(restaurants[1]->getMenu()[0]);
    
    // Place scheduled pickup order
    cout << "\n📦 Placing SCHEDULED PICKUP order..." << endl;
    PaymentStrategy* cardPayment = new CreditCardPayment("1234-5678-9012");
    Order* order2 = app->placeOrder(user, cardPayment, "PICKUP", true);
    
    // List all orders
    app->listAllOrders();
    
    // Cleanup
    delete app;
    delete user;
    delete upiPayment;
    delete cardPayment;
    
    return 0;
}
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
