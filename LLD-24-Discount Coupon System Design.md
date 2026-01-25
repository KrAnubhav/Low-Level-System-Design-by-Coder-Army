# LLD-24: Discount Coupon System Design

## 📋 Table of Contents

1. [Introduction](#introduction)
2. [What is a Discount Coupon System?](#what-is-a-discount-coupon-system)
3. [Functional Requirements](#functional-requirements)
4. [System Flow](#system-flow)
5. [UML Design](#uml-design)
6. [Design Patterns Used](#design-patterns-used)
7. [Implementation](#implementation)
8. [Thread Safety](#thread-safety)
9. [Key Takeaways](#key-takeaways)

---

## 🎯 Introduction

Welcome to the Discount Coupon System Design lecture! This is a **real interview question** that was asked to experienced developers. Today we'll learn how to design a scalable, plug-and-play coupon management system that can be integrated into any e-commerce application.

**Real-world Applications**:
- Zomato, Swiggy (Food Delivery)
- Zepto, Blinkit (Quick Commerce)
- Amazon, Flipkart (E-commerce)
- Any application that offers discounts and coupons

**Why This Design Matters**:
- ✅ Frequently asked in LLD interviews
- ✅ Demonstrates multiple design patterns
- ✅ Real-world scalability challenges
- ✅ Complex business logic handling

---

## 💳 What is a Discount Coupon System?

A **Discount Coupon System** is a plug-and-play module that:
- Manages multiple types of coupons (Bank offers, Loyalty discounts, Seasonal offers)
- Applies different discount strategies (Flat, Percentage, Percentage with Cap)
- Handles coupon stacking (some coupons can be combined, others cannot)
- Works at both cart level and product level
- Ensures thread safety for limited coupon availability

**Example Flow**:
```
User browses products on Zepto
    ↓
Adds items to cart
    ↓
Views available coupons
    ↓
Applies Bank Coupon (20% off)
    ↓
Applies Loyalty Coupon (₹50 flat off)
    ↓
System calculates final discounted price
    ↓
User proceeds to payment
```

---

## 📝 Functional Requirements

### 1. Add New Coupons at Runtime
- System should support adding new coupon types dynamically
- **Design Principle**: Open-Closed Principle
- Examples: Bank coupons, Loyalty coupons, Seasonal offers, Bulk purchase coupons

### 2. Cart Level and Product Level Discounts

**Cart Level Discounts**:
- Applied to entire cart total
- Example: "Get ₹100 off on orders above ₹500"

**Product Level Discounts**:
- Applied to specific product categories
- Example: "5% off on Electronics", "10% off on Groceries"

### 3. Multiple Coupon Types

Support various coupon types:
- **Seasonal Offers**: Valid during specific seasons/festivals
- **Loyalty Discounts**: For premium/loyal customers
- **Banking Discounts**: Bank-specific offers (HDFC, Axis, etc.)
- **Bulk Purchase**: Discounts on bulk orders
- **Category-specific**: Discounts on specific product categories

### 4. Multiple Discount Strategies

**Three Main Strategies**:

1. **Flat Discount**
   - Fixed amount off
   - Example: "₹40 flat off"
   - Calculation: `finalPrice = originalPrice - 40`

2. **Percentage Discount**
   - Percentage-based discount
   - Example: "20% off"
   - Calculation: `finalPrice = originalPrice - (originalPrice * 0.20)`

3. **Percentage with Cap**
   - Percentage discount with upper limit
   - Example: "60% off up to ₹120"
   - Calculation: `discount = min((originalPrice * 0.60), 120)`

### 5. Coupon Stacking Rules

**Combinable Coupons**:
- Some coupons can be applied together
- Example: Bank coupon + Loyalty coupon

**Non-Combinable Coupons**:
- Some coupons are exclusive
- Example: "This offer cannot be combined with other offers"

**Implementation**:
- Each coupon has `isCombinable()` method
- System checks before applying next coupon

### 6. Thread Safety

**Requirement**: Handle concurrent users accessing limited coupons

**Scenario**:
- Only 4 coupons available
- 100 users trying to apply simultaneously
- Only first 4 users should get the coupon

**Implementation**:
- Use mutex locks (C++)
- Use synchronized blocks (Java)
- Atomic operations for coupon count

---

## 🔄 System Flow

### Happy Flow

```
┌─────────────────────────────────────────────────────────┐
│                    User Journey                         │
└─────────────────────────────────────────────────────────┘
                          │
                          ↓
            ┌─────────────────────────┐
            │   Browse Products       │
            │   (Zepto/Zomato/etc)   │
            └───────────┬─────────────┘
                        │
                        ↓
            ┌─────────────────────────┐
            │   Add to Cart           │
            │   (Multiple Products)   │
            └───────────┬─────────────┘
                        │
                        ↓
┌───────────────────────────────────────────────────────────┐
│           Coupon Management System                        │
│  ┌─────────────────────────────────────────────────┐    │
│  │  Step 1: View Available Coupons                 │    │
│  │  - Bank Offers                                   │    │
│  │  - Loyalty Discounts                             │    │
│  │  - Seasonal Offers                               │    │
│  └─────────────────────────────────────────────────┘    │
│                        │                                  │
│                        ↓                                  │
│  ┌─────────────────────────────────────────────────┐    │
│  │  Step 2: Apply Coupons (Chain of Responsibility)│    │
│  │                                                   │    │
│  │  Coupon 1 → isApplicable? → getDiscount()       │    │
│  │      ↓                                            │    │
│  │  Coupon 2 → isApplicable? → getDiscount()       │    │
│  │      ↓                                            │    │
│  │  Coupon 3 → isApplicable? → getDiscount()       │    │
│  └─────────────────────────────────────────────────┘    │
│                        │                                  │
│                        ↓                                  │
│  ┌─────────────────────────────────────────────────┐    │
│  │  Step 3: Calculate Final Price                   │    │
│  │  Original Total: ₹1000                           │    │
│  │  - Bank Coupon: -₹200 (20%)                     │    │
│  │  - Loyalty: -₹50 (flat)                         │    │
│  │  Final Total: ₹750                               │    │
│  └─────────────────────────────────────────────────┘    │
└───────────────────────────────────────────────────────────┘
                        │
                        ↓
            ┌─────────────────────────┐
            │   Proceed to Payment    │
            │   Pay ₹750              │
            └─────────────────────────┘
```

### Important Notes

**Plug-and-Play Design**:
- This is a standalone module
- Can be integrated into any e-commerce application
- Similar to Payment Gateway or Notification System

**Not Building**:
- User management (assumed to exist)
- Product catalog (generic Product class)
- Order management

**Building**:
- Coupon hierarchy
- Discount strategies
- Cart management
- Coupon application logic

---

## 🎨 UML Design

### Complete Class Diagram

```
┌─────────────────────────────────────────────────────────────┐
│                         Product                             │
├─────────────────────────────────────────────────────────────┤
│ - name: String                                              │
│ - category: String                                          │
│ - price: double                                             │
├─────────────────────────────────────────────────────────────┤
│ + getters/setters                                           │
└─────────────────────────────────────────────────────────────┘
                          │
                          │ HAS-A
                          ↓
┌─────────────────────────────────────────────────────────────┐
│                       CartItem                              │
├─────────────────────────────────────────────────────────────┤
│ - product: Product                                          │
│ - quantity: int                                             │
├─────────────────────────────────────────────────────────────┤
│ + getPrice(): double                                        │
│   // Returns: product.price * quantity                      │
└─────────────────────────────────────────────────────────────┘
                          │
                          │ HAS-A (1-to-many)
                          ↓
┌─────────────────────────────────────────────────────────────┐
│                          Cart                               │
├─────────────────────────────────────────────────────────────┤
│ - items: List<CartItem>                                     │
│ - isLoyaltyMember: boolean                                  │
│ - originalTotal: double                                     │
│ - finalTotal: double                                        │
├─────────────────────────────────────────────────────────────┤
│ + addProduct(p: Product, qty: int): void                    │
│ + applyDiscount(amount: double): void                       │
│ + getters/setters                                           │
└─────────────────────────────────────────────────────────────┘
                          │
                          │ Used by
                          ↓
┌─────────────────────────────────────────────────────────────┐
│              <<abstract>> DiscountStrategy                  │
├─────────────────────────────────────────────────────────────┤
│ + calculate(amount: double): double                         │
└──────────────────────┬──────────────────────────────────────┘
                       │
         ┌─────────────┼─────────────┬──────────────────┐
         │             │             │                  │
         ↓             ↓             ↓                  ↓
┌──────────────┐ ┌──────────────┐ ┌────────────────────────┐
│FlatDiscount  │ │PercentDiscount│ │PercentWithCapDiscount │
│Strategy      │ │Strategy       │ │Strategy                │
├──────────────┤ ├──────────────┤ ├────────────────────────┤
│- discount    │ │- percentage  │ │- percentage: double    │
│  Amount:     │ │  : double    │ │- cap: double           │
│  double      │ │              │ │                        │
├──────────────┤ ├──────────────┤ ├────────────────────────┤
│+ calculate() │ │+ calculate() │ │+ calculate()           │
└──────────────┘ └──────────────┘ └────────────────────────┘

┌─────────────────────────────────────────────────────────────┐
│                  <<abstract>> Coupon                        │
├─────────────────────────────────────────────────────────────┤
│ # next: Coupon                    // Chain of Responsibility│
│ # cart: Cart                                                │
├─────────────────────────────────────────────────────────────┤
│ + setNext(c: Coupon): void                                  │
│ + applyDiscount(cart: Cart): void    // Template Method     │
│ # isApplicable(cart: Cart): boolean  // Abstract            │
│ # isCombinable(cart: Cart): boolean  // Abstract            │
│ # getDiscount(cart: Cart): double    // Abstract            │
└──────────────────────┬──────────────────────────────────────┘
                       │
         ┌─────────────┼─────────────┬──────────────────┐
         │             │             │                  │
         ↓             ↓             ↓                  ↓
┌──────────────┐ ┌──────────────┐ ┌────────────┐ ┌──────────┐
│BankingCoupon │ │LoyaltyCoupon │ │BulkPurchase│ │Seasonal  │
│              │ │              │ │Coupon      │ │Coupon    │
├──────────────┤ ├──────────────┤ ├────────────┤ ├──────────┤
│- strategy:   │ │- strategy:   │ │- strategy: │ │- strategy│
│  Discount    │ │  Discount    │ │  Discount  │ │  :Discount│
│  Strategy    │ │  Strategy    │ │  Strategy  │ │  Strategy│
├──────────────┤ ├──────────────┤ ├────────────┤ ├──────────┤
│+ isApplicable│ │+ isApplicable│ │+isApplicable│ │+isApplicable│
│+ isCombinable│ │+ isCombinable│ │+isCombinable│ │+isCombinable│
│+ getDiscount │ │+ getDiscount │ │+getDiscount│ │+getDiscount│
└──────────────┘ └──────────────┘ └────────────┘ └──────────┘
```

### Relationships Summary

**Product → CartItem → Cart**:
- Composition relationship
- Cart manages CartItems
- CartItem wraps Product with quantity

**DiscountStrategy Hierarchy**:
- Strategy Pattern
- Three concrete strategies
- Used by Coupon classes

**Coupon Hierarchy**:
- Template Method Pattern (applyDiscount)
- Chain of Responsibility (next reference)
- Four concrete coupon types

---

## 🎭 Design Patterns Used

### 1. Strategy Pattern

**Used in**: Discount calculation

**Purpose**: Allow different discount calculation algorithms to be used interchangeably

**Implementation**:

```java
// Strategy Interface
public interface DiscountStrategy {
    double calculate(double amount);
}

// Concrete Strategy 1: Flat Discount
public class FlatDiscountStrategy implements DiscountStrategy {
    private double discountAmount;
    
    public FlatDiscountStrategy(double discountAmount) {
        this.discountAmount = discountAmount;
    }
    
    @Override
    public double calculate(double amount) {
        return amount - discountAmount;
    }
}

// Concrete Strategy 2: Percentage Discount
public class PercentDiscountStrategy implements DiscountStrategy {
    private double percentage;
    
    public PercentDiscountStrategy(double percentage) {
        this.percentage = percentage;
    }
    
    @Override
    public double calculate(double amount) {
        return amount - (amount * percentage / 100);
    }
}

// Concrete Strategy 3: Percentage with Cap
public class PercentWithCapStrategy implements DiscountStrategy {
    private double percentage;
    private double cap;
    
    public PercentWithCapStrategy(double percentage, double cap) {
        this.percentage = percentage;
        this.cap = cap;
    }
    
    @Override
    public double calculate(double amount) {
        double discount = amount * percentage / 100;
        discount = Math.min(discount, cap);
        return amount - discount;
    }
}
```

**Benefits**:
- Easy to add new discount strategies
- Coupons can switch strategies dynamically
- Follows Open-Closed Principle

### 2. Chain of Responsibility Pattern

**Used in**: Coupon chaining and application

**Purpose**: Allow multiple coupons to be applied in sequence, with each deciding whether to process or pass to next

**Implementation**:

```java
public abstract class Coupon {
    protected Coupon next;
    protected Cart cart;
    
    public void setNext(Coupon coupon) {
        this.next = coupon;
    }
    
    // Template Method
    public void applyDiscount(Cart cart) {
        this.cart = cart;
        
        // Check if this coupon is applicable
        if (isApplicable(cart)) {
            // Get discount from strategy
            double discount = getDiscount(cart);
            
            // Apply to cart
            cart.setFinalTotal(discount);
            
            // Check if we can apply next coupon
            if (isCombinable(cart) && next != null) {
                next.applyDiscount(cart);  // Chain continues
            }
        } else if (next != null) {
            // Skip this coupon, try next
            next.applyDiscount(cart);
        }
    }
    
    // Abstract methods to be implemented by subclasses
    protected abstract boolean isApplicable(Cart cart);
    protected abstract boolean isCombinable(Cart cart);
    protected abstract double getDiscount(Cart cart);
}
```

**Flow**:
```
BankingCoupon → LoyaltyCoupon → SeasonalCoupon → null
     │              │                │
     ↓              ↓                ↓
isApplicable?  isApplicable?   isApplicable?
     │              │                │
     ↓              ↓                ↓
getDiscount()  getDiscount()   getDiscount()
     │              │                │
     ↓              ↓                ↓
isCombinable?  isCombinable?   isCombinable?
```

**Benefits**:
- Decouples sender from receivers
- Dynamic chain configuration
- Easy to add/remove coupons from chain

### 3. Template Method Pattern

**Used in**: `applyDiscount()` method in Coupon class

**Purpose**: Define skeleton of discount application algorithm, let subclasses customize specific steps

**Implementation**:

```java
public abstract class Coupon {
    // Template Method - defines the algorithm structure
    public void applyDiscount(Cart cart) {
        // Step 1: Check applicability (implemented by subclass)
        if (!isApplicable(cart)) {
            if (next != null) {
                next.applyDiscount(cart);
            }
            return;
        }
        
        // Step 2: Calculate discount (implemented by subclass)
        double discount = getDiscount(cart);
        
        // Step 3: Apply discount to cart
        cart.setFinalTotal(discount);
        
        // Step 4: Check if combinable (implemented by subclass)
        if (isCombinable(cart) && next != null) {
            next.applyDiscount(cart);
        }
    }
    
    // Primitive operations - to be implemented by subclasses
    protected abstract boolean isApplicable(Cart cart);
    protected abstract boolean isCombinable(Cart cart);
    protected abstract double getDiscount(Cart cart);
}
```

**Benefits**:
- Enforces consistent discount application flow
- Subclasses customize behavior without changing structure
- Code reuse in base class

---

## 💻 Implementation

### 1. Product Class (Model)

```java
public class Product {
    private String name;
    private String category;
    private double price;
    
    public Product(String name, String category, double price) {
        this.name = name;
        this.category = category;
        this.price = price;
    }
    
    // Getters and Setters
    public String getName() { return name; }
    public String getCategory() { return category; }
    public double getPrice() { return price; }
}
```

### 2. CartItem Class

```java
public class CartItem {
    private Product product;
    private int quantity;
    
    public CartItem(Product product, int quantity) {
        this.product = product;
        this.quantity = quantity;
    }
    
    public double getPrice() {
        return product.getPrice() * quantity;
    }
    
    public Product getProduct() { return product; }
    public int getQuantity() { return quantity; }
}
```

### 3. Cart Class

```java
import java.util.ArrayList;
import java.util.List;

public class Cart {
    private List<CartItem> items;
    private boolean isLoyaltyMember;
    private double originalTotal;
    private double finalTotal;
    
    public Cart(boolean isLoyaltyMember) {
        this.items = new ArrayList<>();
        this.isLoyaltyMember = isLoyaltyMember;
        this.originalTotal = 0.0;
        this.finalTotal = 0.0;
    }
    
    public void addProduct(Product product, int quantity) {
        CartItem item = new CartItem(product, quantity);
        items.add(item);
        calculateTotal();
    }
    
    private void calculateTotal() {
        originalTotal = 0.0;
        for (CartItem item : items) {
            originalTotal += item.getPrice();
        }
        finalTotal = originalTotal;  // Initially same as original
    }
    
    public void applyDiscount(double discountedAmount) {
        this.finalTotal = discountedAmount;
    }
    
    // Getters and Setters
    public List<CartItem> getItems() { return items; }
    public boolean isLoyaltyMember() { return isLoyaltyMember; }
    public double getOriginalTotal() { return originalTotal; }
    public double getFinalTotal() { return finalTotal; }
    public void setFinalTotal(double finalTotal) { this.finalTotal = finalTotal; }
}
```

### 4. Discount Strategy Interface and Implementations

```java
// Strategy Interface
public interface DiscountStrategy {
    double calculate(double amount);
}

// Flat Discount Strategy
public class FlatDiscountStrategy implements DiscountStrategy {
    private double discountAmount;
    
    public FlatDiscountStrategy(double discountAmount) {
        this.discountAmount = discountAmount;
    }
    
    @Override
    public double calculate(double amount) {
        return amount - discountAmount;
    }
}

// Percentage Discount Strategy
public class PercentDiscountStrategy implements DiscountStrategy {
    private double percentage;
    
    public PercentDiscountStrategy(double percentage) {
        this.percentage = percentage;
    }
    
    @Override
    public double calculate(double amount) {
        return amount - (amount * percentage / 100);
    }
}

// Percentage with Cap Strategy
public class PercentWithCapStrategy implements DiscountStrategy {
    private double percentage;
    private double cap;
    
    public PercentWithCapStrategy(double percentage, double cap) {
        this.percentage = percentage;
        this.cap = cap;
    }
    
    @Override
    public double calculate(double amount) {
        double discount = amount * percentage / 100;
        discount = Math.min(discount, cap);
        return amount - discount;
    }
}
```

### 5. Coupon Base Class

```java
public abstract class Coupon {
    protected Coupon next;
    
    public void setNext(Coupon coupon) {
        this.next = coupon;
    }
    
    // Template Method
    public void applyDiscount(Cart cart) {
        // Check if applicable
        if (!isApplicable(cart)) {
            // Try next coupon in chain
            if (next != null) {
                next.applyDiscount(cart);
            }
            return;
        }
        
        // Get discount amount
        double discountedPrice = getDiscount(cart);
        
        // Apply to cart
        cart.applyDiscount(discountedPrice);
        
        // Check if we can apply next coupon
        if (isCombinable(cart) && next != null) {
            next.applyDiscount(cart);
        }
    }
    
    // Abstract methods - to be implemented by subclasses
    protected abstract boolean isApplicable(Cart cart);
    protected abstract boolean isCombinable(Cart cart);
    protected abstract double getDiscount(Cart cart);
}
```

### 6. Concrete Coupon Implementations

```java
// Banking Coupon
public class BankingCoupon extends Coupon {
    private DiscountStrategy strategy;
    private String bankName;
    
    public BankingCoupon(String bankName, DiscountStrategy strategy) {
        this.bankName = bankName;
        this.strategy = strategy;
    }
    
    @Override
    protected boolean isApplicable(Cart cart) {
        // Check if user is using the specific bank
        // For demo, always return true
        System.out.println("Checking if Banking Coupon is applicable for " + bankName);
        return true;
    }
    
    @Override
    protected boolean isCombinable(Cart cart) {
        // Banking coupons are usually not combinable
        System.out.println("Banking Coupon is NOT combinable");
        return false;
    }
    
    @Override
    protected double getDiscount(Cart cart) {
        double currentTotal = cart.getFinalTotal();
        double discountedPrice = strategy.calculate(currentTotal);
        System.out.println("Banking Coupon applied: ₹" + currentTotal + " → ₹" + discountedPrice);
        return discountedPrice;
    }
}

// Loyalty Coupon
public class LoyaltyCoupon extends Coupon {
    private DiscountStrategy strategy;
    
    public LoyaltyCoupon(DiscountStrategy strategy) {
        this.strategy = strategy;
    }
    
    @Override
    protected boolean isApplicable(Cart cart) {
        // Only applicable for loyalty members
        boolean applicable = cart.isLoyaltyMember();
        System.out.println("Checking if Loyalty Coupon is applicable: " + applicable);
        return applicable;
    }
    
    @Override
    protected boolean isCombinable(Cart cart) {
        // Loyalty coupons can be combined
        System.out.println("Loyalty Coupon is combinable");
        return true;
    }
    
    @Override
    protected double getDiscount(Cart cart) {
        double currentTotal = cart.getFinalTotal();
        double discountedPrice = strategy.calculate(currentTotal);
        System.out.println("Loyalty Coupon applied: ₹" + currentTotal + " → ₹" + discountedPrice);
        return discountedPrice;
    }
}

// Bulk Purchase Coupon
public class BulkPurchaseCoupon extends Coupon {
    private DiscountStrategy strategy;
    private double minimumAmount;
    
    public BulkPurchaseCoupon(double minimumAmount, DiscountStrategy strategy) {
        this.minimumAmount = minimumAmount;
        this.strategy = strategy;
    }
    
    @Override
    protected boolean isApplicable(Cart cart) {
        // Only applicable if cart total exceeds minimum
        boolean applicable = cart.getOriginalTotal() >= minimumAmount;
        System.out.println("Checking if Bulk Purchase Coupon is applicable (min: ₹" + 
                         minimumAmount + "): " + applicable);
        return applicable;
    }
    
    @Override
    protected boolean isCombinable(Cart cart) {
        // Bulk purchase coupons can be combined
        System.out.println("Bulk Purchase Coupon is combinable");
        return true;
    }
    
    @Override
    protected double getDiscount(Cart cart) {
        double currentTotal = cart.getFinalTotal();
        double discountedPrice = strategy.calculate(currentTotal);
        System.out.println("Bulk Purchase Coupon applied: ₹" + currentTotal + " → ₹" + discountedPrice);
        return discountedPrice;
    }
}

// Seasonal Coupon
public class SeasonalCoupon extends Coupon {
    private DiscountStrategy strategy;
    private String category;
    
    public SeasonalCoupon(String category, DiscountStrategy strategy) {
        this.category = category;
        this.strategy = strategy;
    }
    
    @Override
    protected boolean isApplicable(Cart cart) {
        // Check if cart has items from the specific category
        for (CartItem item : cart.getItems()) {
            if (item.getProduct().getCategory().equals(category)) {
                System.out.println("Seasonal Coupon is applicable for category: " + category);
                return true;
            }
        }
        System.out.println("Seasonal Coupon is NOT applicable (no " + category + " items)");
        return false;
    }
    
    @Override
    protected boolean isCombinable(Cart cart) {
        // Seasonal coupons can be combined
        System.out.println("Seasonal Coupon is combinable");
        return true;
    }
    
    @Override
    protected double getDiscount(Cart cart) {
        double currentTotal = cart.getFinalTotal();
        double discountedPrice = strategy.calculate(currentTotal);
        System.out.println("Seasonal Coupon applied: ₹" + currentTotal + " → ₹" + discountedPrice);
        return discountedPrice;
    }
}
```

### 7. Client Code (Demo)

```java
public class CouponSystemDemo {
    public static void main(String[] args) {
        System.out.println("========== Discount Coupon System Demo ==========\n");
        
        // Create products
        Product laptop = new Product("Dell Laptop", "Electronics", 50000);
        Product mouse = new Product("Wireless Mouse", "Electronics", 1000);
        Product book = new Product("Java Book", "Books", 500);
        
        // Create cart for loyalty member
        Cart cart = new Cart(true);
        cart.addProduct(laptop, 1);
        cart.addProduct(mouse, 2);
        cart.addProduct(book, 1);
        
        System.out.println("Cart Original Total: ₹" + cart.getOriginalTotal());
        System.out.println("\n--- Applying Coupons ---\n");
        
        // Create discount strategies
        DiscountStrategy flat50 = new FlatDiscountStrategy(50);
        DiscountStrategy percent10 = new PercentDiscountStrategy(10);
        DiscountStrategy percent20Cap100 = new PercentWithCapStrategy(20, 100);
        
        // Create coupons
        Coupon loyaltyCoupon = new LoyaltyCoupon(flat50);
        Coupon bulkCoupon = new BulkPurchaseCoupon(1000, percent10);
        Coupon seasonalCoupon = new SeasonalCoupon("Electronics", percent20Cap100);
        
        // Chain coupons (Chain of Responsibility)
        loyaltyCoupon.setNext(bulkCoupon);
        bulkCoupon.setNext(seasonalCoupon);
        
        // Apply coupons
        loyaltyCoupon.applyDiscount(cart);
        
        System.out.println("\n--- Final Result ---");
        System.out.println("Original Total: ₹" + cart.getOriginalTotal());
        System.out.println("Final Total (after discounts): ₹" + cart.getFinalTotal());
        System.out.println("Total Savings: ₹" + (cart.getOriginalTotal() - cart.getFinalTotal()));
        
        
        System.out.println("\n\n========== Testing Non-Combinable Coupon ==========\n");
        
        // Create new cart
        Cart cart2 = new Cart(false);
        cart2.addProduct(laptop, 1);
        
        System.out.println("Cart Original Total: ₹" + cart2.getOriginalTotal());
        System.out.println("\n--- Applying Banking Coupon (Non-Combinable) ---\n");
        
        // Banking coupon (non-combinable)
        Coupon bankingCoupon = new BankingCoupon("HDFC", new PercentDiscountStrategy(15));
        Coupon loyaltyCoupon2 = new LoyaltyCoupon(flat50);
        
        // Chain coupons
        bankingCoupon.setNext(loyaltyCoupon2);
        
        // Apply coupons
        bankingCoupon.applyDiscount(cart2);
        
        System.out.println("\n--- Final Result ---");
        System.out.println("Original Total: ₹" + cart2.getOriginalTotal());
        System.out.println("Final Total (after discounts): ₹" + cart2.getFinalTotal());
        System.out.println("Note: Loyalty coupon was NOT applied (Banking coupon is not combinable)");
    }
}
```

### Sample Output

```
========== Discount Coupon System Demo ==========

Cart Original Total: ₹52500.0

--- Applying Coupons ---

Checking if Loyalty Coupon is applicable: true
Loyalty Coupon applied: ₹52500.0 → ₹52450.0
Loyalty Coupon is combinable
Checking if Bulk Purchase Coupon is applicable (min: ₹1000.0): true
Bulk Purchase Coupon applied: ₹52450.0 → ₹47205.0
Bulk Purchase Coupon is combinable
Seasonal Coupon is applicable for category: Electronics
Seasonal Coupon applied: ₹47205.0 → ₹47105.0
Seasonal Coupon is combinable

--- Final Result ---
Original Total: ₹52500.0
Final Total (after discounts): ₹47105.0
Total Savings: ₹5395.0


========== Testing Non-Combinable Coupon ==========

Cart Original Total: ₹50000.0

--- Applying Banking Coupon (Non-Combinable) ---

Checking if Banking Coupon is applicable for HDFC
Banking Coupon applied: ₹50000.0 → ₹42500.0
Banking Coupon is NOT combinable

--- Final Result ---
Original Total: ₹50000.0
Final Total (after discounts): ₹42500.0
Note: Loyalty coupon was NOT applied (Banking coupon is not combinable)
```

---

## 🔒 Thread Safety

### Problem Statement

**Scenario**: Limited coupon availability
- Only 4 coupons available for a special offer
- 100 users trying to apply simultaneously
- Only first 4 users should get the coupon

### Solution: Mutex/Synchronized Blocks

**C++ Implementation**:

```cpp
#include <mutex>

class LimitedCoupon : public Coupon {
private:
    int availableCount;
    std::mutex mtx;
    
public:
    LimitedCoupon(int count, DiscountStrategy* strategy) 
        : availableCount(count), strategy(strategy) {}
    
    bool isApplicable(Cart* cart) override {
        std::lock_guard<std::mutex> lock(mtx);
        
        if (availableCount > 0) {
            availableCount--;
            return true;
        }
        
        return false;
    }
    
    // Other methods...
};
```

**Java Implementation**:

```java
import java.util.concurrent.atomic.AtomicInteger;

public class LimitedCoupon extends Coupon {
    private AtomicInteger availableCount;
    private DiscountStrategy strategy;
    
    public LimitedCoupon(int count, DiscountStrategy strategy) {
        this.availableCount = new AtomicInteger(count);
        this.strategy = strategy;
    }
    
    @Override
    protected synchronized boolean isApplicable(Cart cart) {
        if (availableCount.get() > 0) {
            availableCount.decrementAndGet();
            System.out.println("Coupon applied! Remaining: " + availableCount.get());
            return true;
        }
        
        System.out.println("Coupon sold out!");
        return false;
    }
    
    @Override
    protected boolean isCombinable(Cart cart) {
        return true;
    }
    
    @Override
    protected double getDiscount(Cart cart) {
        return strategy.calculate(cart.getFinalTotal());
    }
}
```

**Thread Safety Demo**:

```java
import java.util.concurrent.ExecutorService;
import java.util.concurrent.Executors;

public class ThreadSafetyDemo {
    public static void main(String[] args) {
        // Create limited coupon (only 4 available)
        Coupon limitedCoupon = new LimitedCoupon(4, new FlatDiscountStrategy(100));
        
        // Create thread pool with 10 threads
        ExecutorService executor = Executors.newFixedThreadPool(10);
        
        // Simulate 10 users trying to apply coupon
        for (int i = 0; i < 10; i++) {
            final int userId = i + 1;
            executor.submit(() -> {
                Cart cart = new Cart(false);
                cart.addProduct(new Product("Item", "General", 1000), 1);
                
                System.out.println("User " + userId + " trying to apply coupon...");
                limitedCoupon.applyDiscount(cart);
            });
        }
        
        executor.shutdown();
    }
}
```

**Output**:
```
User 1 trying to apply coupon...
Coupon applied! Remaining: 3
User 2 trying to apply coupon...
Coupon applied! Remaining: 2
User 3 trying to apply coupon...
Coupon applied! Remaining: 1
User 4 trying to apply coupon...
Coupon applied! Remaining: 0
User 5 trying to apply coupon...
Coupon sold out!
User 6 trying to apply coupon...
Coupon sold out!
...
```

---

## 🎓 Key Takeaways

### Design Patterns Summary

| Pattern | Usage | Benefit |
|---------|-------|---------|
| **Strategy** | Discount calculation | Interchangeable algorithms |
| **Chain of Responsibility** | Coupon chaining | Dynamic coupon application |
| **Template Method** | applyDiscount() | Consistent flow, customizable steps |

### SOLID Principles

1. **Single Responsibility**
   - Each class has one job
   - Product: Store product data
   - Cart: Manage cart items
   - Coupon: Apply discounts
   - Strategy: Calculate discounts

2. **Open-Closed Principle**
   - Open for extension (new coupons, new strategies)
   - Closed for modification (existing code unchanged)

3. **Liskov Substitution**
   - Any Coupon subclass can replace Coupon
   - Any DiscountStrategy can replace another

4. **Interface Segregation**
   - Small, focused interfaces
   - DiscountStrategy has single method

5. **Dependency Inversion**
   - Coupon depends on DiscountStrategy interface
   - Not on concrete implementations

### Architecture Highlights

**Plug-and-Play Design**:
```
Any E-commerce App
    ↓
Integrates Coupon System
    ↓
Uses Cart + Coupon classes
    ↓
No need to know internal details
```

**Scalability**:
- ✅ Easy to add new coupon types
- ✅ Easy to add new discount strategies
- ✅ Easy to modify coupon rules
- ✅ Thread-safe for concurrent users

### Real-world Integration

**Example: Integrating with Zomato Clone**:

```java
// In your Zomato application
public class Order {
    private Cart cart;
    private CouponManager couponManager;
    
    public void checkout() {
        // 1. User selects coupons
        List<Coupon> selectedCoupons = couponManager.getAvailableCoupons();
        
        // 2. Chain coupons
        for (int i = 0; i < selectedCoupons.size() - 1; i++) {
            selectedCoupons.get(i).setNext(selectedCoupons.get(i + 1));
        }
        
        // 3. Apply coupons
        if (!selectedCoupons.isEmpty()) {
            selectedCoupons.get(0).applyDiscount(cart);
        }
        
        // 4. Proceed to payment
        processPayment(cart.getFinalTotal());
    }
}
```

### Best Practices

1. ✅ Use Strategy for algorithm variations
2. ✅ Use Chain of Responsibility for sequential processing
3. ✅ Use Template Method for consistent flow
4. ✅ Implement thread safety for limited resources
5. ✅ Keep coupon logic separate from business logic
6. ✅ Make system plug-and-play
7. ✅ Follow SOLID principles

---

## 📊 Comparison: Design Alternatives

### Decorator vs Chain of Responsibility

**We chose Chain of Responsibility**, but Decorator Pattern could also work:

| Aspect | Chain of Responsibility | Decorator Pattern |
|--------|------------------------|-------------------|
| **Structure** | Linear chain with next reference | Nested wrapping |
| **Control** | Each handler decides to pass or stop | Each decorator adds behavior |
| **Our Use Case** | ✅ Better fit (some coupons stop chain) | Could work but less intuitive |
| **Flexibility** | Easy to skip handlers | All decorators apply |

**Decorator Alternative**:
```java
// With Decorator
Coupon baseCoupon = new BaseCoupon(cart);
Coupon withLoyalty = new LoyaltyDecorator(baseCoupon);
Coupon withBank = new BankingDecorator(withLoyalty);
withBank.apply();
```

**Chain of Responsibility (Our Choice)**:
```java
// With Chain of Responsibility
Coupon loyalty = new LoyaltyCoupon(strategy);
Coupon banking = new BankingCoupon(strategy);
loyalty.setNext(banking);
loyalty.applyDiscount(cart);  // May stop if not combinable
```

---

## 🎯 Interview Tips

When explaining this design in an interview:

1. **Start with Requirements**
   - Multiple coupon types
   - Different discount strategies
   - Coupon stacking rules
   - Thread safety

2. **Explain Pattern Choices**
   - Strategy: For discount algorithms
   - Chain of Responsibility: For coupon chaining
   - Template Method: For consistent flow
   - Mention Decorator as alternative

3. **Discuss Trade-offs**
   - Why Chain of Responsibility over Decorator
   - Complexity vs Flexibility
   - Performance considerations

4. **Show Extensibility**
   - How to add new coupon type
   - How to add new discount strategy
   - How to modify rules

5. **Address Thread Safety**
   - Limited coupon scenario
   - Mutex/Synchronized approach
   - Atomic operations

6. **Real-world Examples**
   - Zomato, Swiggy coupons
   - Amazon offers
   - Bank discounts

---

**Happy Learning! 🚀**

**Remember**: This is a **real interview question**. Practice implementing it from scratch. Try integrating it with your previous projects (like Zomato clone)!

**Homework**: Implement the additional features mentioned in requirements and integrate this coupon system with a real e-commerce application!
