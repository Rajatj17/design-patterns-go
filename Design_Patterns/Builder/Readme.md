# Design Patterns Guide: Builder Pattern

## Table of Contents
1. [Overview](#overview)
2. [Builder Pattern Fundamentals](#builder-pattern-fundamentals)
3. [Implementation in Java](#implementation-in-java)
4. [Implementation in Go](#implementation-in-go)
5. [Side-by-Side Comparison](#side-by-side-comparison)
6. [Advanced Patterns](#advanced-patterns)
7. [Use Cases and Examples](#use-cases-and-examples)
8. [Best Practices](#best-practices)
9. [Pros and Cons](#pros-and-cons)

## Overview

The **Builder Pattern** is a creational design pattern that provides a flexible solution for constructing complex objects step by step. It separates the construction of a complex object from its representation, allowing the same construction process to create different representations.

### Key Components:
- **Product**: The complex object being built
- **Builder**: Abstract interface for creating parts of a Product object
- **ConcreteBuilder**: Implements the Builder interface and constructs/assembles parts
- **Director**: Constructs an object using the Builder interface (optional)

### Real-world Analogy:
Think of building a house where you have a blueprint (Director), various specialists (ConcreteBuilders) like plumbers, electricians, and carpenters, and the final house (Product). Each specialist knows how to build their part, and the blueprint coordinates the overall construction.

## Builder Pattern Fundamentals

### Problem It Solves:
- Creating objects with many optional parameters
- Avoiding telescoping constructor anti-pattern
- Ensuring object immutability while allowing flexible construction
- Providing clear, readable object construction code

### When to Use:
- Objects have many parameters (especially optional ones)
- Object construction is complex and multi-step
- You need different representations of the same object
- You want to ensure object immutability

### Structure:
```
Director
├── construct() -> Product
└── uses Builder interface

Builder (interface)
├── buildPartA()
├── buildPartB()
└── getResult() -> Product

ConcreteBuilder (implements Builder)
├── buildPartA() implementation
├── buildPartB() implementation
└── getResult() implementation

Product
├── partA
├── partB
└── complex object with multiple components
```

## Implementation in Java

### Java Implementation - Computer Builder Example

```java
// Product class - Computer
class Computer {
    // Required parameters
    private final String cpu;
    private final int ram;
    
    // Optional parameters
    private final boolean hasGraphicsCard;
    private final boolean hasSSD;
    private final String operatingSystem;
    private final boolean hasWifi;
    private final boolean hasBluetooth;
    private final int storageSize;
    private final String screenSize;
    
    // Private constructor - only accessible through Builder
    private Computer(ComputerBuilder builder) {
        this.cpu = builder.cpu;
        this.ram = builder.ram;
        this.hasGraphicsCard = builder.hasGraphicsCard;
        this.hasSSD = builder.hasSSD;
        this.operatingSystem = builder.operatingSystem;
        this.hasWifi = builder.hasWifi;
        this.hasBluetooth = builder.hasBluetooth;
        this.storageSize = builder.storageSize;
        this.screenSize = builder.screenSize;
    }
    
    // Getters
    public String getCpu() { return cpu; }
    public int getRam() { return ram; }
    public boolean hasGraphicsCard() { return hasGraphicsCard; }
    public boolean hasSSD() { return hasSSD; }
    public String getOperatingSystem() { return operatingSystem; }
    public boolean hasWifi() { return hasWifi; }
    public boolean hasBluetooth() { return hasBluetooth; }
    public int getStorageSize() { return storageSize; }
    public String getScreenSize() { return screenSize; }
    
    @Override
    public String toString() {
        return "Computer{" +
                "cpu='" + cpu + '\'' +
                ", ram=" + ram + "GB" +
                ", hasGraphicsCard=" + hasGraphicsCard +
                ", hasSSD=" + hasSSD +
                ", operatingSystem='" + operatingSystem + '\'' +
                ", hasWifi=" + hasWifi +
                ", hasBluetooth=" + hasBluetooth +
                ", storageSize=" + storageSize + "GB" +
                ", screenSize='" + screenSize + '\'' +
                '}';
    }
    
    // Static nested Builder class
    public static class ComputerBuilder {
        // Required parameters
        private String cpu;
        private int ram;
        
        // Optional parameters - initialized to default values
        private boolean hasGraphicsCard = false;
        private boolean hasSSD = false;
        private String operatingSystem = "None";
        private boolean hasWifi = false;
        private boolean hasBluetooth = false;
        private int storageSize = 500;
        private String screenSize = "15 inch";
        
        // Constructor with required parameters
        public ComputerBuilder(String cpu, int ram) {
            this.cpu = cpu;
            this.ram = ram;
        }
        
        // Methods for optional parameters - return Builder for chaining
        public ComputerBuilder setGraphicsCard(boolean hasGraphicsCard) {
            this.hasGraphicsCard = hasGraphicsCard;
            return this;
        }
        
        public ComputerBuilder setSSD(boolean hasSSD) {
            this.hasSSD = hasSSD;
            return this;
        }
        
        public ComputerBuilder setOperatingSystem(String operatingSystem) {
            this.operatingSystem = operatingSystem;
            return this;
        }
        
        public ComputerBuilder setWifi(boolean hasWifi) {
            this.hasWifi = hasWifi;
            return this;
        }
        
        public ComputerBuilder setBluetooth(boolean hasBluetooth) {
            this.hasBluetooth = hasBluetooth;
            return this;
        }
        
        public ComputerBuilder setStorageSize(int storageSize) {
            this.storageSize = storageSize;
            return this;
        }
        
        public ComputerBuilder setScreenSize(String screenSize) {
            this.screenSize = screenSize;
            return this;
        }
        
        // Build method to create Computer instance
        public Computer build() {
            return new Computer(this);
        }
    }
}

// Director class (optional) - defines common build sequences
class ComputerDirector {
    private ComputerBuilder builder;
    
    public ComputerDirector(ComputerBuilder builder) {
        this.builder = builder;
    }
    
    public Computer buildGamingComputer() {
        return builder
                .setGraphicsCard(true)
                .setSSD(true)
                .setOperatingSystem("Windows 11")
                .setWifi(true)
                .setBluetooth(true)
                .setStorageSize(1000)
                .setScreenSize("27 inch")
                .build();
    }
    
    public Computer buildOfficeComputer() {
        return builder
                .setGraphicsCard(false)
                .setSSD(true)
                .setOperatingSystem("Windows 11")
                .setWifi(true)
                .setBluetooth(false)
                .setStorageSize(500)
                .setScreenSize("24 inch")
                .build();
    }
    
    public Computer buildBudgetComputer() {
        return builder
                .setGraphicsCard(false)
                .setSSD(false)
                .setOperatingSystem("Linux")
                .setWifi(true)
                .setBluetooth(false)
                .setStorageSize(250)
                .setScreenSize("21 inch")
                .build();
    }
}

// Client code demonstrating usage
public class BuilderPatternDemo {
    public static void main(String[] args) {
        // Method 1: Direct builder usage
        System.out.println("=== Direct Builder Usage ===");
        
        Computer gamingPC = new Computer.ComputerBuilder("Intel i9", 32)
                .setGraphicsCard(true)
                .setSSD(true)
                .setOperatingSystem("Windows 11")
                .setWifi(true)
                .setBluetooth(true)
                .setStorageSize(2000)
                .setScreenSize("32 inch")
                .build();
        
        Computer basicPC = new Computer.ComputerBuilder("Intel i3", 8)
                .setWifi(true)
                .setOperatingSystem("Ubuntu")
                .build();
        
        System.out.println("Gaming PC: " + gamingPC);
        System.out.println("Basic PC: " + basicPC);
        
        // Method 2: Using Director
        System.out.println("\n=== Using Director ===");
        
        ComputerDirector director = new ComputerDirector(
            new Computer.ComputerBuilder("AMD Ryzen 7", 16)
        );
        
        Computer officePC = director.buildOfficeComputer();
        System.out.println("Office PC: " + officePC);
        
        // New director for different CPU/RAM combo
        ComputerDirector budgetDirector = new ComputerDirector(
            new Computer.ComputerBuilder("AMD Ryzen 3", 8)
        );
        
        Computer budgetPC = budgetDirector.buildBudgetComputer();
        System.out.println("Budget PC: " + budgetPC);
    }
}
```

### Java Output:
```
=== Direct Builder Usage ===
Gaming PC: Computer{cpu='Intel i9', ram=32GB, hasGraphicsCard=true, hasSSD=true, operatingSystem='Windows 11', hasWifi=true, hasBluetooth=true, storageSize=2000GB, screenSize='32 inch'}
Basic PC: Computer{cpu='Intel i3', ram=8GB, hasGraphicsCard=false, hasSSD=false, operatingSystem='Ubuntu', hasWifi=true, hasBluetooth=false, storageSize=500GB, screenSize='15 inch'}

=== Using Director ===
Office PC: Computer{cpu='AMD Ryzen 7', ram=16GB, hasGraphicsCard=false, hasSSD=true, operatingSystem='Windows 11', hasWifi=true, hasBluetooth=false, storageSize=500GB, screenSize='24 inch'}
Budget PC: Computer{cpu='AMD Ryzen 3', ram=8GB, hasGraphicsCard=false, hasSSD=false, operatingSystem='Linux', hasWifi=true, hasBluetooth=false, storageSize=250GB, screenSize='21 inch'}
```

## Implementation in Go

### Go Implementation - Computer Builder Example

```go
package main

import (
    "fmt"
    "strings"
)

// Product struct - Computer
type Computer struct {
    // Required parameters
    CPU string
    RAM int
    
    // Optional parameters
    HasGraphicsCard bool
    HasSSD          bool
    OperatingSystem string
    HasWifi         bool
    HasBluetooth    bool
    StorageSize     int
    ScreenSize      string
}

// String method for Computer
func (c *Computer) String() string {
    var features []string
    
    features = append(features, fmt.Sprintf("CPU: %s", c.CPU))
    features = append(features, fmt.Sprintf("RAM: %dGB", c.RAM))
    features = append(features, fmt.Sprintf("Graphics Card: %t", c.HasGraphicsCard))
    features = append(features, fmt.Sprintf("SSD: %t", c.HasSSD))
    features = append(features, fmt.Sprintf("OS: %s", c.OperatingSystem))
    features = append(features, fmt.Sprintf("WiFi: %t", c.HasWifi))
    features = append(features, fmt.Sprintf("Bluetooth: %t", c.HasBluetooth))
    features = append(features, fmt.Sprintf("Storage: %dGB", c.StorageSize))
    features = append(features, fmt.Sprintf("Screen: %s", c.ScreenSize))
    
    return fmt.Sprintf("Computer{%s}", strings.Join(features, ", "))
}

// Builder struct
type ComputerBuilder struct {
    computer *Computer
}

// Constructor for Builder with required parameters
func NewComputerBuilder(cpu string, ram int) *ComputerBuilder {
    return &ComputerBuilder{
        computer: &Computer{
            CPU:             cpu,
            RAM:             ram,
            HasGraphicsCard: false,
            HasSSD:          false,
            OperatingSystem: "None",
            HasWifi:         false,
            HasBluetooth:    false,
            StorageSize:     500,
            ScreenSize:      "15 inch",
        },
    }
}

// Builder methods for optional parameters - return *ComputerBuilder for chaining
func (cb *ComputerBuilder) SetGraphicsCard(hasGraphicsCard bool) *ComputerBuilder {
    cb.computer.HasGraphicsCard = hasGraphicsCard
    return cb
}

func (cb *ComputerBuilder) SetSSD(hasSSD bool) *ComputerBuilder {
    cb.computer.HasSSD = hasSSD
    return cb
}

func (cb *ComputerBuilder) SetOperatingSystem(os string) *ComputerBuilder {
    cb.computer.OperatingSystem = os
    return cb
}

func (cb *ComputerBuilder) SetWifi(hasWifi bool) *ComputerBuilder {
    cb.computer.HasWifi = hasWifi
    return cb
}

func (cb *ComputerBuilder) SetBluetooth(hasBluetooth bool) *ComputerBuilder {
    cb.computer.HasBluetooth = hasBluetooth
    return cb
}

func (cb *ComputerBuilder) SetStorageSize(storageSize int) *ComputerBuilder {
    cb.computer.StorageSize = storageSize
    return cb
}

func (cb *ComputerBuilder) SetScreenSize(screenSize string) *ComputerBuilder {
    cb.computer.ScreenSize = screenSize
    return cb
}

// Build method to return the final Computer
func (cb *ComputerBuilder) Build() *Computer {
    // Create a copy to ensure immutability
    result := *cb.computer
    return &result
}

// Reset method to reuse builder
func (cb *ComputerBuilder) Reset(cpu string, ram int) *ComputerBuilder {
    cb.computer = &Computer{
        CPU:             cpu,
        RAM:             ram,
        HasGraphicsCard: false,
        HasSSD:          false,
        OperatingSystem: "None",
        HasWifi:         false,
        HasBluetooth:    false,
        StorageSize:     500,
        ScreenSize:      "15 inch",
    }
    return cb
}

// Director struct (optional) - defines common build sequences
type ComputerDirector struct {
    builder *ComputerBuilder
}

// Constructor for Director
func NewComputerDirector(builder *ComputerBuilder) *ComputerDirector {
    return &ComputerDirector{
        builder: builder,
    }
}

// Predefined build sequences
func (cd *ComputerDirector) BuildGamingComputer() *Computer {
    return cd.builder.
        SetGraphicsCard(true).
        SetSSD(true).
        SetOperatingSystem("Windows 11").
        SetWifi(true).
        SetBluetooth(true).
        SetStorageSize(1000).
        SetScreenSize("27 inch").
        Build()
}

func (cd *ComputerDirector) BuildOfficeComputer() *Computer {
    return cd.builder.
        SetGraphicsCard(false).
        SetSSD(true).
        SetOperatingSystem("Windows 11").
        SetWifi(true).
        SetBluetooth(false).
        SetStorageSize(500).
        SetScreenSize("24 inch").
        Build()
}

func (cd *ComputerDirector) BuildBudgetComputer() *Computer {
    return cd.builder.
        SetGraphicsCard(false).
        SetSSD(false).
        SetOperatingSystem("Linux").
        SetWifi(true).
        SetBluetooth(false).
        SetStorageSize(250).
        SetScreenSize("21 inch").
        Build()
}

// Functional Options Pattern (Go-idiomatic alternative)
type ComputerOption func(*Computer)

func WithGraphicsCard(hasGraphicsCard bool) ComputerOption {
    return func(c *Computer) {
        c.HasGraphicsCard = hasGraphicsCard
    }
}

func WithSSD(hasSSD bool) ComputerOption {
    return func(c *Computer) {
        c.HasSSD = hasSSD
    }
}

func WithOperatingSystem(os string) ComputerOption {
    return func(c *Computer) {
        c.OperatingSystem = os
    }
}

func WithWifi(hasWifi bool) ComputerOption {
    return func(c *Computer) {
        c.HasWifi = hasWifi
    }
}

func WithBluetooth(hasBluetooth bool) ComputerOption {
    return func(c *Computer) {
        c.HasBluetooth = hasBluetooth
    }
}

func WithStorageSize(storageSize int) ComputerOption {
    return func(c *Computer) {
        c.StorageSize = storageSize
    }
}

func WithScreenSize(screenSize string) ComputerOption {
    return func(c *Computer) {
        c.ScreenSize = screenSize
    }
}

// Functional constructor
func NewComputer(cpu string, ram int, options ...ComputerOption) *Computer {
    computer := &Computer{
        CPU:             cpu,
        RAM:             ram,
        HasGraphicsCard: false,
        HasSSD:          false,
        OperatingSystem: "None",
        HasWifi:         false,
        HasBluetooth:    false,
        StorageSize:     500,
        ScreenSize:      "15 inch",
    }
    
    for _, option := range options {
        option(computer)
    }
    
    return computer
}

// Client code demonstrating usage
func main() {
    fmt.Println("=== Direct Builder Usage ===")
    
    // Method 1: Traditional Builder Pattern
    gamingPC := NewComputerBuilder("Intel i9", 32).
        SetGraphicsCard(true).
        SetSSD(true).
        SetOperatingSystem("Windows 11").
        SetWifi(true).
        SetBluetooth(true).
        SetStorageSize(2000).
        SetScreenSize("32 inch").
        Build()
    
    basicPC := NewComputerBuilder("Intel i3", 8).
        SetWifi(true).
        SetOperatingSystem("Ubuntu").
        Build()
    
    fmt.Printf("Gaming PC: %s\n", gamingPC)
    fmt.Printf("Basic PC: %s\n", basicPC)
    
    // Method 2: Using Director
    fmt.Println("\n=== Using Director ===")
    
    director := NewComputerDirector(NewComputerBuilder("AMD Ryzen 7", 16))
    officePC := director.BuildOfficeComputer()
    fmt.Printf("Office PC: %s\n", officePC)
    
    // Reuse director with different specs
    director.builder.Reset("AMD Ryzen 3", 8)
    budgetPC := director.BuildBudgetComputer()
    fmt.Printf("Budget PC: %s\n", budgetPC)
    
    // Method 3: Functional Options Pattern (Go-idiomatic)
    fmt.Println("\n=== Functional Options Pattern ===")
    
    workstationPC := NewComputer("Intel Xeon", 64,
        WithGraphicsCard(true),
        WithSSD(true),
        WithOperatingSystem("Linux"),
        WithWifi(true),
        WithBluetooth(true),
        WithStorageSize(4000),
        WithScreenSize("34 inch"),
    )
    
    fmt.Printf("Workstation PC: %s\n", workstationPC)
}
```

### Go Output:
```
=== Direct Builder Usage ===
Gaming PC: Computer{CPU: Intel i9, RAM: 32GB, Graphics Card: true, SSD: true, OS: Windows 11, WiFi: true, Bluetooth: true, Storage: 2000GB, Screen: 32 inch}
Basic PC: Computer{CPU: Intel i3, RAM: 8GB, Graphics Card: false, SSD: false, OS: Ubuntu, WiFi: true, Bluetooth: false, Storage: 500GB, Screen: 15 inch}

=== Using Director ===
Office PC: Computer{CPU: AMD Ryzen 7, RAM: 16GB, Graphics Card: false, SSD: true, OS: Windows 11, WiFi: true, Bluetooth: false, Storage: 500GB, Screen: 24 inch}
Budget PC: Computer{CPU: AMD Ryzen 3, RAM: 8GB, Graphics Card: false, SSD: false, OS: Linux, WiFi: true, Bluetooth: false, Storage: 250GB, Screen: 21 inch}

=== Functional Options Pattern ===
Workstation PC: Computer{CPU: Intel Xeon, RAM: 64GB, Graphics Card: true, SSD: true, OS: Linux, WiFi: true, Bluetooth: true, Storage: 4000GB, Screen: 34 inch}
```

## Side-by-Side Comparison

### 1. Builder Definition

**Java:**
```java
public static class ComputerBuilder {
    private String cpu;
    private int ram;
    // ... other fields
    
    public ComputerBuilder(String cpu, int ram) {
        this.cpu = cpu;
        this.ram = ram;
    }
    
    public ComputerBuilder setGraphicsCard(boolean hasGraphicsCard) {
        this.hasGraphicsCard = hasGraphicsCard;
        return this;
    }
    
    public Computer build() {
        return new Computer(this);
    }
}
```

**Go:**
```go
type ComputerBuilder struct {
    computer *Computer
}

func NewComputerBuilder(cpu string, ram int) *ComputerBuilder {
    return &ComputerBuilder{
        computer: &Computer{
            CPU: cpu,
            RAM: ram,
            // ... default values
        },
    }
}

func (cb *ComputerBuilder) SetGraphicsCard(hasGraphicsCard bool) *ComputerBuilder {
    cb.computer.HasGraphicsCard = hasGraphicsCard
    return cb
}

func (cb *ComputerBuilder) Build() *Computer {
    result := *cb.computer
    return &result
}
```

**Key Differences:**
- Java uses static nested class, Go uses separate struct with constructor function
- Java builds final object in constructor, Go modifies internal object and copies on build
- Java uses `this` implicitly, Go uses explicit receiver `cb`

### 2. Product Construction

**Java:**
```java
private Computer(ComputerBuilder builder) {
    this.cpu = builder.cpu;
    this.ram = builder.ram;
    // ... copy all fields from builder
}
```

**Go:**
```go
func (cb *ComputerBuilder) Build() *Computer {
    result := *cb.computer  // Copy struct
    return &result
}
```

**Key Differences:**
- Java uses private constructor taking builder as parameter
- Go copies the struct and returns pointer to copy
- Java ensures immutability at construction time, Go ensures it at build time

### 3. Method Chaining

**Java:**
```java
Computer pc = new Computer.ComputerBuilder("Intel i7", 16)
    .setGraphicsCard(true)
    .setSSD(true)
    .build();
```

**Go:**
```go
pc := NewComputerBuilder("Intel i7", 16).
    SetGraphicsCard(true).
    SetSSD(true).
    Build()
```

**Key Differences:**
- Java uses dot notation throughout
- Go typically uses dot on new lines for readability
- Both return the builder instance for chaining

### 4. Go-Specific: Functional Options Pattern

**Go Functional Options:**
```go
type ComputerOption func(*Computer)

func WithGraphicsCard(hasGraphicsCard bool) ComputerOption {
    return func(c *Computer) {
        c.HasGraphicsCard = hasGraphicsCard
    }
}

func NewComputer(cpu string, ram int, options ...ComputerOption) *Computer {
    computer := &Computer{/* defaults */}
    for _, option := range options {
        option(computer)
    }
    return computer
}

// Usage
pc := NewComputer("Intel i7", 16,
    WithGraphicsCard(true),
    WithSSD(true),
)
```

**Key Differences:**
- Go's functional options pattern is more idiomatic
- Uses variadic functions and closures
- More flexible than traditional builder pattern
- No separate builder struct needed

### 5. Director Implementation

**Java:**
```java
public class ComputerDirector {
    private ComputerBuilder builder;
    
    public ComputerDirector(ComputerBuilder builder) {
        this.builder = builder;
    }
    
    public Computer buildGamingComputer() {
        return builder
                .setGraphicsCard(true)
                .setSSD(true)
                // ... more configurations
                .build();
    }
}
```

**Go:**
```go
type ComputerDirector struct {
    builder *ComputerBuilder
}

func NewComputerDirector(builder *ComputerBuilder) *ComputerDirector {
    return &ComputerDirector{builder: builder}
}

func (cd *ComputerDirector) BuildGamingComputer() *Computer {
    return cd.builder.
        SetGraphicsCard(true).
        SetSSD(true).
        // ... more configurations
        Build()
}
```

**Key Differences:**
- Java uses class with constructor, Go uses struct with constructor function
- Similar method structure and usage patterns
- Go uses pointer receivers, Java uses implicit `this`

## Advanced Patterns

### 1. Validation in Builder

**Java with Validation:**
```java
public Computer build() {
    validateConfiguration();
    return new Computer(this);
}

private void validateConfiguration() {
    if (ram < 4) {
        throw new IllegalStateException("RAM must be at least 4GB");
    }
    if (hasGraphicsCard && storageSize < 500) {
        throw new IllegalStateException("Graphics card requires at least 500GB storage");
    }
}
```

**Go with Validation:**
```go
func (cb *ComputerBuilder) Build() (*Computer, error) {
    if err := cb.validateConfiguration(); err != nil {
        return nil, err
    }
    result := *cb.computer
    return &result, nil
}

func (cb *ComputerBuilder) validateConfiguration() error {
    if cb.computer.RAM < 4 {
        return fmt.Errorf("RAM must be at least 4GB")
    }
    if cb.computer.HasGraphicsCard && cb.computer.StorageSize < 500 {
        return fmt.Errorf("graphics card requires at least 500GB storage")
    }
    return nil
}
```

### 2. Generic Builder (Java)

```java
public abstract class GenericBuilder<T> {
    protected T product;
    
    public abstract T build();
    
    protected void validateProduct() {
        // Common validation logic
    }
}

public class ComputerBuilder extends GenericBuilder<Computer> {
    // Implementation
}
```

### 3. Interface-based Builder (Go)

```go
type Builder interface {
    Build() (interface{}, error)
    Reset()
}

type ComputerBuilder struct {
    computer *Computer
}

func (cb *ComputerBuilder) Build() (interface{}, error) {
    return cb.computer, nil
}

func (cb *ComputerBuilder) Reset() {
    cb.computer = &Computer{/* defaults */}
}
```

## Use Cases and Examples

### 1. Configuration Objects
**Use Case:** Application configuration with many optional settings, database connections, API clients.

### 2. SQL Query Building
**Use Case:** Building complex SQL queries with optional WHERE clauses, JOINs, ORDER BY, etc.

```java
// Java SQL Builder Example
String query = new SQLQueryBuilder("users")
    .select("name", "email")
    .where("age > 18")
    .orderBy("name")
    .limit(10)
    .build();
```

### 3. HTTP Request Building
**Use Case:** Creating HTTP requests with optional headers, parameters, authentication.

### 4. Document/Report Generation
**Use Case:** Generating documents with optional sections, formatting, metadata.

### 5. Game Character Creation
**Use Case:** RPG characters with optional skills, equipment, attributes.

## Best Practices

### Java Best Practices:
1. **Use static nested builder class** for clean encapsulation
2. **Make product class immutable** with private constructor
3. **Validate in build() method** before creating product
4. **Use method chaining** for fluent interface
5. **Consider using validation annotations** like `@NotNull`, `@Valid`
6. **Implement proper toString()** for debugging
7. **Use generics** for reusable builders

### Go Best Practices:
1. **Consider functional options pattern** for Go-idiomatic approach
2. **Use pointer receivers** for builder methods
3. **Copy structs in Build()** to ensure immutability
4. **Return errors from Build()** for validation failures
5. **Provide Reset() method** for builder reuse
6. **Use constructor functions** instead of direct struct creation
7. **Export only necessary fields and methods**

### Common Best Practices:
1. **Separate required and optional parameters** clearly
2. **Provide sensible defaults** for optional parameters
3. **Use director for common configurations**
4. **Keep builder interface simple** and focused
5. **Document expected usage patterns**
6. **Consider thread safety** if builders will be shared
7. **Implement proper error handling** and validation

## Pros and Cons

### Advantages:
- **Eliminates telescoping constructors** - no need for multiple constructor overloads
- **Improves readability** - clear, self-documenting code
- **Enforces immutability** - products can be immutable while allowing flexible construction
- **Supports validation** - can validate before creating product
- **Flexible construction** - different representations of same product
- **Method chaining** - fluent, readable interface

### Disadvantages:
- **Increased complexity** - more classes/structs and code
- **Performance overhead** - additional object creation
- **Memory usage** - builder objects consume memory
- **Learning curve** - more complex than simple constructors
- **Over-engineering** - may be overkill for simple objects

### Java-Specific Pros:
- **Strong typing** - compile-time checking of method calls
- **Static nested classes** - clean encapsulation
- **Mature tooling** - IDE support for builder generation
- **Established patterns** - well-known in Java community

### Java-Specific Cons:
- **Verbose syntax** - more boilerplate code
- **Memory overhead** - object headers and garbage collection
- **No multiple inheritance** - limits builder composition

### Go-Specific Pros:
- **Functional options** - more idiomatic and flexible
- **Simpler syntax** - less boilerplate
- **Better performance** - more efficient memory usage
- **Flexible approaches** - multiple valid implementation patterns

### Go-Specific Cons:
- **Less IDE support** - for automatic builder generation
- **Manual memory management awareness** - need to consider pointers vs values
- **Less established patterns** - fewer standard approaches

## Conclusion

The Builder pattern is excellent for creating complex objects with many optional parameters. Both Java and Go provide effective implementations:

**Choose Java Builder Pattern when:**
- You need strong typing and compile-time validation
- Working in enterprise environments with established Java patterns
- Building complex objects with strict validation requirements
- Team prefers explicit, verbose code structure

**Choose Go Builder Pattern when:**
- You prefer more idiomatic Go code (functional options)
- Performance and memory efficiency