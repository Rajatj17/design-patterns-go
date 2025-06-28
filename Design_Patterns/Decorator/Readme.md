# Design Patterns: A Comprehensive Guide

## Introduction

Design patterns are reusable solutions to commonly occurring problems in software design. They represent best practices evolved over time by experienced developers and provide a common vocabulary for discussing design solutions.

## The Decorator Pattern

### Overview

The Decorator pattern allows you to dynamically add new functionality to objects without altering their structure. It provides a flexible alternative to subclassing for extending functionality by wrapping objects in decorator classes that implement the same interface.

### Key Components:
- **Component**: Defines the interface for objects that can have responsibilities added dynamically
- **ConcreteComponent**: The original object to which additional responsibilities can be attached
- **Decorator**: Maintains a reference to a Component object and defines an interface that conforms to Component's interface
- **ConcreteDecorator**: Adds responsibilities to the component

### When to Use:
- You want to add responsibilities to objects dynamically and transparently
- You want to add responsibilities to objects without affecting other objects
- Extension by subclassing is impractical or would result in an explosion of subclasses

---

## Implementation Comparison: Go vs Java

### 1. Component Interface Definition

**Go Implementation:**
```go
type Coffee interface {
    GetDescription() string
    GetCost() float64
}
```

**Java Implementation:**
```java
interface Coffee {
    String getDescription();
    double getCost();
}
```

**Key Differences:**
- Go uses `type` keyword and capitalizes method names for public access
- Java uses `interface` keyword with camelCase method names
- Go's interface is implicitly implemented, Java's is explicitly implemented

### 2. Concrete Component (Base Object)

**Go Implementation:**
```go
type SimpleCoffee struct{}

func (c *SimpleCoffee) GetDescription() string {
    return "Simple Coffee"
}

func (c *SimpleCoffee) GetCost() float64 {
    return 2.00
}
```

**Java Implementation:**
```java
class SimpleCoffee implements Coffee {
    @Override
    public String getDescription() {
        return "Simple Coffee";
    }
    
    @Override
    public double getCost() {
        return 2.00;
    }
}
```

**Key Differences:**
- Go uses struct with receiver methods, no explicit interface implementation
- Java uses class with explicit `implements` keyword and `@Override` annotations
- Go's method receivers can be pointers or values

### 3. Base Decorator

**Go Implementation:**
```go
type CoffeeDecorator struct {
    Coffee Coffee  // Composition
}

func (cd *CoffeeDecorator) GetDescription() string {
    return cd.Coffee.GetDescription()
}

func (cd *CoffeeDecorator) GetCost() float64 {
    return cd.Coffee.GetCost()
}
```

**Java Implementation:**
```java
abstract class CoffeeDecorator implements Coffee {
    protected Coffee coffee;  // Composition
    
    public CoffeeDecorator(Coffee coffee) {
        this.coffee = coffee;
    }
    
    @Override
    public String getDescription() {
        return coffee.getDescription();
    }
    
    @Override
    public double getCost() {
        return coffee.getCost();
    }
}
```

**Key Differences:**
- Go uses struct embedding for composition
- Java uses abstract class with protected field and constructor
- Go doesn't have constructors - initialization is done through factory functions or direct field assignment

### 4. Concrete Decorators

**Go Implementation:**
```go
// Milk Decorator
type MilkDecorator struct {
    CoffeeDecorator  // Embedding
}

func NewMilkDecorator(coffee Coffee) *MilkDecorator {
    return &MilkDecorator{
        CoffeeDecorator: CoffeeDecorator{Coffee: coffee},
    }
}

func (md *MilkDecorator) GetDescription() string {
    return md.Coffee.GetDescription() + ", Milk"
}

func (md *MilkDecorator) GetCost() float64 {
    return md.Coffee.GetCost() + 0.50
}

// Sugar Decorator
type SugarDecorator struct {
    CoffeeDecorator
}

func NewSugarDecorator(coffee Coffee) *SugarDecorator {
    return &SugarDecorator{
        CoffeeDecorator: CoffeeDecorator{Coffee: coffee},
    }
}

func (sd *SugarDecorator) GetDescription() string {
    return sd.Coffee.GetDescription() + ", Sugar"
}

func (sd *SugarDecorator) GetCost() float64 {
    return sd.Coffee.GetCost() + 0.25
}
```

**Java Implementation:**
```java
// Milk Decorator
class MilkDecorator extends CoffeeDecorator {
    public MilkDecorator(Coffee coffee) {
        super(coffee);
    }
    
    @Override
    public String getDescription() {
        return coffee.getDescription() + ", Milk";
    }
    
    @Override
    public double getCost() {
        return coffee.getCost() + 0.50;
    }
}

// Sugar Decorator
class SugarDecorator extends CoffeeDecorator {
    public SugarDecorator(Coffee coffee) {
        super(coffee);
    }
    
    @Override
    public String getDescription() {
        return coffee.getDescription() + ", Sugar";
    }
    
    @Override
    public double getCost() {
        return coffee.getCost() + 0.25;
    }
}
```

**Key Differences:**
- Go uses factory functions (`NewXxxDecorator`) instead of constructors
- Java uses inheritance with `extends` and `super()` calls
- Go's struct embedding provides automatic delegation
- Java requires explicit constructor chaining

### 5. Usage Examples

**Go Usage:**
```go
func main() {
    // Basic coffee
    coffee := &SimpleCoffee{}
    fmt.Printf("Order: %s, Cost: $%.2f\n", 
        coffee.GetDescription(), coffee.GetCost())
    
    // Add decorations
    coffeeWithMilk := NewMilkDecorator(coffee)
    fmt.Printf("Order: %s, Cost: $%.2f\n", 
        coffeeWithMilk.GetDescription(), coffeeWithMilk.GetCost())
    
    // Chain decorators
    deluxeCoffee := NewSugarDecorator(
        NewMilkDecorator(&SimpleCoffee{}))
    fmt.Printf("Order: %s, Cost: $%.2f\n", 
        deluxeCoffee.GetDescription(), deluxeCoffee.GetCost())
}
```

**Java Usage:**
```java
public static void main(String[] args) {
    // Basic coffee
    Coffee coffee = new SimpleCoffee();
    System.out.printf("Order: %s, Cost: $%.2f%n", 
        coffee.getDescription(), coffee.getCost());
    
    // Add decorations
    Coffee coffeeWithMilk = new MilkDecorator(coffee);
    System.out.printf("Order: %s, Cost: $%.2f%n", 
        coffeeWithMilk.getDescription(), coffeeWithMilk.getCost());
    
    // Chain decorators
    Coffee deluxeCoffee = new SugarDecorator(
        new MilkDecorator(new SimpleCoffee()));
    System.out.printf("Order: %s, Cost: $%.2f%n", 
        deluxeCoffee.getDescription(), deluxeCoffee.getCost());
}
```

---

## Complete Working Examples

### Go Complete Implementation

```go
package main

import "fmt"

// Component interface
type Coffee interface {
    GetDescription() string
    GetCost() float64
}

// ConcreteComponent
type SimpleCoffee struct{}

func (c *SimpleCoffee) GetDescription() string {
    return "Simple Coffee"
}

func (c *SimpleCoffee) GetCost() float64 {
    return 2.00
}

// Base Decorator
type CoffeeDecorator struct {
    Coffee Coffee
}

func (cd *CoffeeDecorator) GetDescription() string {
    return cd.Coffee.GetDescription()
}

func (cd *CoffeeDecorator) GetCost() float64 {
    return cd.Coffee.GetCost()
}

// Concrete Decorators
type MilkDecorator struct {
    CoffeeDecorator
}

func NewMilkDecorator(coffee Coffee) *MilkDecorator {
    return &MilkDecorator{
        CoffeeDecorator: CoffeeDecorator{Coffee: coffee},
    }
}

func (md *MilkDecorator) GetDescription() string {
    return md.Coffee.GetDescription() + ", Milk"
}

func (md *MilkDecorator) GetCost() float64 {
    return md.Coffee.GetCost() + 0.50
}

type SugarDecorator struct {
    CoffeeDecorator
}

func NewSugarDecorator(coffee Coffee) *SugarDecorator {
    return &SugarDecorator{
        CoffeeDecorator: CoffeeDecorator{Coffee: coffee},
    }
}

func (sd *SugarDecorator) GetDescription() string {
    return sd.Coffee.GetDescription() + ", Sugar"
}

func (sd *SugarDecorator) GetCost() float64 {
    return sd.Coffee.GetCost() + 0.25
}

func main() {
    // Basic usage
    coffee := &SimpleCoffee{}
    fmt.Printf("Order: %s, Cost: $%.2f\n", 
        coffee.GetDescription(), coffee.GetCost())
    
    // With decorators
    deluxeCoffee := NewSugarDecorator(
        NewMilkDecorator(&SimpleCoffee{}))
    fmt.Printf("Order: %s, Cost: $%.2f\n", 
        deluxeCoffee.GetDescription(), deluxeCoffee.GetCost())
}
```

### Java Complete Implementation

```java
// Component interface
interface Coffee {
    String getDescription();
    double getCost();
}

// ConcreteComponent
class SimpleCoffee implements Coffee {
    @Override
    public String getDescription() {
        return "Simple Coffee";
    }
    
    @Override
    public double getCost() {
        return 2.00;
    }
}

// Base Decorator
abstract class CoffeeDecorator implements Coffee {
    protected Coffee coffee;
    
    public CoffeeDecorator(Coffee coffee) {
        this.coffee = coffee;
    }
    
    @Override
    public String getDescription() {
        return coffee.getDescription();
    }
    
    @Override
    public double getCost() {
        return coffee.getCost();
    }
}

// Concrete Decorators
class MilkDecorator extends CoffeeDecorator {
    public MilkDecorator(Coffee coffee) {
        super(coffee);
    }
    
    @Override
    public String getDescription() {
        return coffee.getDescription() + ", Milk";
    }
    
    @Override
    public double getCost() {
        return coffee.getCost() + 0.50;
    }
}

class SugarDecorator extends CoffeeDecorator {
    public SugarDecorator(Coffee coffee) {
        super(coffee);
    }
    
    @Override
    public String getDescription() {
        return coffee.getDescription() + ", Sugar";
    }
    
    @Override
    public double getCost() {
        return coffee.getCost() + 0.25;
    }
}

// Demo
public class DecoratorDemo {
    public static void main(String[] args) {
        // Basic usage
        Coffee coffee = new SimpleCoffee();
        System.out.printf("Order: %s, Cost: $%.2f%n", 
            coffee.getDescription(), coffee.getCost());
        
        // With decorators
        Coffee deluxeCoffee = new SugarDecorator(
            new MilkDecorator(new SimpleCoffee()));
        System.out.printf("Order: %s, Cost: $%.2f%n", 
            deluxeCoffee.getDescription(), deluxeCoffee.getCost());
    }
}
```

---

## Language-Specific Considerations

### Go Advantages:
- **Implicit Interface Implementation**: Types automatically satisfy interfaces if they implement the required methods
- **Struct Embedding**: Provides automatic delegation and composition
- **Factory Functions**: Clear initialization patterns without constructors
- **Pointer Receivers**: Memory efficient when dealing with large structs

### Java Advantages:
- **Abstract Classes**: Provide partial implementation and enforce structure
- **Constructor Chaining**: Clear initialization hierarchy with `super()`
- **Access Modifiers**: Fine-grained control over field and method visibility
- **Annotations**: `@Override` provides compile-time checking

### Memory Management:
- **Go**: Garbage collected, but you control pointer vs value semantics
- **Java**: Garbage collected, all objects are references

### Error Handling:
- **Go**: Explicit error returns, no exceptions in this pattern
- **Java**: Can use exceptions for error conditions in decorators

---

## Real-World Applications

### Go Examples:
- HTTP middleware chains (Gin, Echo frameworks)
- I/O wrappers (bufio.Reader, gzip.Reader)
- Logging decorators

### Java Examples:
- Java I/O streams (BufferedReader, FileReader)
- Servlet filters
- Spring AOP proxies

## Benefits of Decorator Pattern

1. **Flexibility**: Add responsibilities dynamically
2. **Single Responsibility**: Each decorator has one reason to change
3. **Composition over Inheritance**: Avoids class explosion
4. **Runtime Configuration**: Decorators can be combined at runtime

## Drawbacks

1. **Complexity**: Can result in many small objects
2. **Debugging**: Stack traces can be deep with many decorators
3. **Interface Matching**: All decorators must implement the same interface