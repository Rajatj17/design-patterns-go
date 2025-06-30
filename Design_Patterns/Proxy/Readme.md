# Design Patterns Guide: Proxy Pattern in Go vs Java

## Table of Contents
1. [Introduction to Design Patterns](#introduction)
2. [Proxy Pattern Overview](#proxy-pattern-overview)
3. [Implementation Comparison](#implementation-comparison)
4. [Real-World Examples](#real-world-examples)
5. [Best Practices](#best-practices)
6. [Performance Considerations](#performance-considerations)

## Introduction

Design patterns are reusable solutions to common problems in software design. They represent best practices evolved over time by experienced developers. This guide focuses on the **Proxy Pattern** and demonstrates how it's implemented differently in Go and Java.

## Proxy Pattern Overview

### Definition
The Proxy pattern provides a placeholder or surrogate for another object to control access to it. It acts as an intermediary between the client and the real object.

### When to Use
- **Virtual Proxy**: Expensive object creation (lazy loading)
- **Protection Proxy**: Access control and authentication
- **Remote Proxy**: Remote object access (RPC, web services)
- **Caching Proxy**: Cache expensive operations
- **Logging Proxy**: Add logging without modifying original object

### Structure
```
Client -> Proxy -> RealSubject
```

## Implementation Comparison

### Basic Proxy Pattern

#### Java Implementation

```java
// Subject interface
interface ImageViewer {
    void displayImage(String filename);
}

// Real Subject
class RealImageViewer implements ImageViewer {
    @Override
    public void displayImage(String filename) {
        System.out.println("Displaying image: " + filename);
        // Simulate expensive operation
        try {
            Thread.sleep(1000);
        } catch (InterruptedException e) {
            Thread.currentThread().interrupt();
        }
    }
}

// Proxy
class ImageViewerProxy implements ImageViewer {
    private RealImageViewer realImageViewer;
    private final String filename;
    
    public ImageViewerProxy(String filename) {
        this.filename = filename;
    }
    
    @Override
    public void displayImage(String filename) {
        if (realImageViewer == null) {
            System.out.println("Creating real image viewer...");
            realImageViewer = new RealImageViewer();
        }
        realImageViewer.displayImage(filename);
    }
}

// Client code
public class ProxyPatternDemo {
    public static void main(String[] args) {
        ImageViewer image = new ImageViewerProxy("photo.jpg");
        
        // Image will be loaded from disk
        image.displayImage("photo.jpg");
        
        // Image will not be loaded from disk again
        image.displayImage("photo.jpg");
    }
}
```

#### Go Implementation

```go
package main

import (
    "fmt"
    "time"
)

// Subject interface
type ImageViewer interface {
    DisplayImage(filename string)
}

// Real Subject
type RealImageViewer struct{}

func (r *RealImageViewer) DisplayImage(filename string) {
    fmt.Printf("Displaying image: %s\n", filename)
    // Simulate expensive operation
    time.Sleep(1 * time.Second)
}

// Proxy
type ImageViewerProxy struct {
    realImageViewer *RealImageViewer
    filename        string
}

func NewImageViewerProxy(filename string) *ImageViewerProxy {
    return &ImageViewerProxy{
        filename: filename,
    }
}

func (p *ImageViewerProxy) DisplayImage(filename string) {
    if p.realImageViewer == nil {
        fmt.Println("Creating real image viewer...")
        p.realImageViewer = &RealImageViewer{}
    }
    p.realImageViewer.DisplayImage(filename)
}

// Client code
func main() {
    image := NewImageViewerProxy("photo.jpg")
    
    // Image will be loaded from disk
    image.DisplayImage("photo.jpg")
    
    // Image will not be loaded from disk again
    image.DisplayImage("photo.jpg")
}
```

### Key Differences in Basic Implementation

| Aspect | Java | Go |
|--------|------|-----|
| **Interface Definition** | `interface` keyword with method signatures | `type InterfaceName interface` with method signatures |
| **Struct/Class** | Classes with explicit interface implementation | Structs with implicit interface satisfaction |
| **Constructor** | Constructor methods or builder pattern | Factory functions (e.g., `NewImageViewerProxy`) |
| **Memory Management** | Automatic garbage collection | Automatic garbage collection with explicit control |
| **Null Checking** | `null` checks | `nil` checks |

## Real-World Examples

### 1. Caching Proxy

#### Java Implementation

```java
import java.util.HashMap;
import java.util.Map;

interface DatabaseService {
    String getData(String key);
}

class RealDatabaseService implements DatabaseService {
    @Override
    public String getData(String key) {
        System.out.println("Fetching data from database for key: " + key);
        // Simulate database call
        try {
            Thread.sleep(2000);
        } catch (InterruptedException e) {
            Thread.currentThread().interrupt();
        }
        return "Data for " + key;
    }
}

class CachingDatabaseProxy implements DatabaseService {
    private final RealDatabaseService realService;
    private final Map<String, String> cache;
    
    public CachingDatabaseProxy() {
        this.realService = new RealDatabaseService();
        this.cache = new HashMap<>();
    }
    
    @Override
    public String getData(String key) {
        if (cache.containsKey(key)) {
            System.out.println("Cache hit for key: " + key);
            return cache.get(key);
        }
        
        System.out.println("Cache miss for key: " + key);
        String data = realService.getData(key);
        cache.put(key, data);
        return data;
    }
}

// Usage
public class CachingProxyDemo {
    public static void main(String[] args) {
        DatabaseService service = new CachingDatabaseProxy();
        
        // First call - cache miss
        System.out.println(service.getData("user123"));
        
        // Second call - cache hit
        System.out.println(service.getData("user123"));
    }
}
```

#### Go Implementation

```go
package main

import (
    "fmt"
    "time"
)

type DatabaseService interface {
    GetData(key string) string
}

type RealDatabaseService struct{}

func (r *RealDatabaseService) GetData(key string) string {
    fmt.Printf("Fetching data from database for key: %s\n", key)
    // Simulate database call
    time.Sleep(2 * time.Second)
    return fmt.Sprintf("Data for %s", key)
}

type CachingDatabaseProxy struct {
    realService *RealDatabaseService
    cache       map[string]string
}

func NewCachingDatabaseProxy() *CachingDatabaseProxy {
    return &CachingDatabaseProxy{
        realService: &RealDatabaseService{},
        cache:       make(map[string]string),
    }
}

func (c *CachingDatabaseProxy) GetData(key string) string {
    if data, exists := c.cache[key]; exists {
        fmt.Printf("Cache hit for key: %s\n", key)
        return data
    }
    
    fmt.Printf("Cache miss for key: %s\n", key)
    data := c.realService.GetData(key)
    c.cache[key] = data
    return data
}

// Usage
func main() {
    service := NewCachingDatabaseProxy()
    
    // First call - cache miss
    fmt.Println(service.GetData("user123"))
    
    // Second call - cache hit
    fmt.Println(service.GetData("user123"))
}
```

### 2. Protection Proxy (Access Control)

#### Java Implementation

```java
interface FileAccess {
    String readFile(String filename);
    void writeFile(String filename, String content);
}

class RealFileAccess implements FileAccess {
    @Override
    public String readFile(String filename) {
        return "Contents of " + filename;
    }
    
    @Override
    public void writeFile(String filename, String content) {
        System.out.println("Writing to " + filename + ": " + content);
    }
}

class ProtectedFileProxy implements FileAccess {
    private final RealFileAccess realFileAccess;
    private final String userRole;
    
    public ProtectedFileProxy(String userRole) {
        this.realFileAccess = new RealFileAccess();
        this.userRole = userRole;
    }
    
    @Override
    public String readFile(String filename) {
        if (hasReadPermission()) {
            return realFileAccess.readFile(filename);
        }
        throw new SecurityException("Access denied: insufficient permissions");
    }
    
    @Override
    public void writeFile(String filename, String content) {
        if (hasWritePermission()) {
            realFileAccess.writeFile(filename, content);
        } else {
            throw new SecurityException("Access denied: insufficient permissions");
        }
    }
    
    private boolean hasReadPermission() {
        return "admin".equals(userRole) || "user".equals(userRole);
    }
    
    private boolean hasWritePermission() {
        return "admin".equals(userRole);
    }
}

// Usage
public class ProtectionProxyDemo {
    public static void main(String[] args) {
        // Admin user
        FileAccess adminAccess = new ProtectedFileProxy("admin");
        System.out.println(adminAccess.readFile("config.txt"));
        adminAccess.writeFile("config.txt", "new config");
        
        // Regular user
        FileAccess userAccess = new ProtectedFileProxy("user");
        System.out.println(userAccess.readFile("config.txt"));
        
        try {
            userAccess.writeFile("config.txt", "malicious content");
        } catch (SecurityException e) {
            System.out.println("Error: " + e.getMessage());
        }
    }
}
```

#### Go Implementation

```go
package main

import (
    "errors"
    "fmt"
)

type FileAccess interface {
    ReadFile(filename string) (string, error)
    WriteFile(filename, content string) error
}

type RealFileAccess struct{}

func (r *RealFileAccess) ReadFile(filename string) (string, error) {
    return fmt.Sprintf("Contents of %s", filename), nil
}

func (r *RealFileAccess) WriteFile(filename, content string) error {
    fmt.Printf("Writing to %s: %s\n", filename, content)
    return nil
}

type ProtectedFileProxy struct {
    realFileAccess *RealFileAccess
    userRole       string
}

func NewProtectedFileProxy(userRole string) *ProtectedFileProxy {
    return &ProtectedFileProxy{
        realFileAccess: &RealFileAccess{},
        userRole:       userRole,
    }
}

func (p *ProtectedFileProxy) ReadFile(filename string) (string, error) {
    if p.hasReadPermission() {
        return p.realFileAccess.ReadFile(filename)
    }
    return "", errors.New("access denied: insufficient permissions")
}

func (p *ProtectedFileProxy) WriteFile(filename, content string) error {
    if p.hasWritePermission() {
        return p.realFileAccess.WriteFile(filename, content)
    }
    return errors.New("access denied: insufficient permissions")
}

func (p *ProtectedFileProxy) hasReadPermission() bool {
    return p.userRole == "admin" || p.userRole == "user"
}

func (p *ProtectedFileProxy) hasWritePermission() bool {
    return p.userRole == "admin"
}

// Usage
func main() {
    // Admin user
    adminAccess := NewProtectedFileProxy("admin")
    if content, err := adminAccess.ReadFile("config.txt"); err == nil {
        fmt.Println(content)
    }
    adminAccess.WriteFile("config.txt", "new config")
    
    // Regular user
    userAccess := NewProtectedFileProxy("user")
    if content, err := userAccess.ReadFile("config.txt"); err == nil {
        fmt.Println(content)
    }
    
    if err := userAccess.WriteFile("config.txt", "malicious content"); err != nil {
        fmt.Printf("Error: %s\n", err.Error())
    }
}
```

### 3. Logging Proxy

#### Java Implementation

```java
import java.time.LocalDateTime;

interface PaymentService {
    boolean processPayment(double amount, String cardNumber);
}

class RealPaymentService implements PaymentService {
    @Override
    public boolean processPayment(double amount, String cardNumber) {
        // Simulate payment processing
        System.out.println("Processing payment of $" + amount);
        return amount > 0 && cardNumber.length() == 16;
    }
}

class LoggingPaymentProxy implements PaymentService {
    private final RealPaymentService realPaymentService;
    
    public LoggingPaymentProxy() {
        this.realPaymentService = new RealPaymentService();
    }
    
    @Override
    public boolean processPayment(double amount, String cardNumber) {
        String maskedCard = maskCardNumber(cardNumber);
        
        System.out.println("[" + LocalDateTime.now() + "] Payment request - Amount: $" + 
                          amount + ", Card: " + maskedCard);
        
        long startTime = System.currentTimeMillis();
        boolean result = realPaymentService.processPayment(amount, cardNumber);
        long endTime = System.currentTimeMillis();
        
        System.out.println("[" + LocalDateTime.now() + "] Payment " + 
                          (result ? "successful" : "failed") + 
                          " - Duration: " + (endTime - startTime) + "ms");
        
        return result;
    }
    
    private String maskCardNumber(String cardNumber) {
        if (cardNumber.length() < 4) return "****";
        return "**** **** **** " + cardNumber.substring(cardNumber.length() - 4);
    }
}

// Usage
public class LoggingProxyDemo {
    public static void main(String[] args) {
        PaymentService paymentService = new LoggingPaymentProxy();
        
        paymentService.processPayment(100.50, "1234567890123456");
        paymentService.processPayment(-50.0, "9876543210987654");
    }
}
```

#### Go Implementation

```go
package main

import (
    "fmt"
    "strings"
    "time"
)

type PaymentService interface {
    ProcessPayment(amount float64, cardNumber string) bool
}

type RealPaymentService struct{}

func (r *RealPaymentService) ProcessPayment(amount float64, cardNumber string) bool {
    // Simulate payment processing
    fmt.Printf("Processing payment of $%.2f\n", amount)
    return amount > 0 && len(cardNumber) == 16
}

type LoggingPaymentProxy struct {
    realPaymentService *RealPaymentService
}

func NewLoggingPaymentProxy() *LoggingPaymentProxy {
    return &LoggingPaymentProxy{
        realPaymentService: &RealPaymentService{},
    }
}

func (l *LoggingPaymentProxy) ProcessPayment(amount float64, cardNumber string) bool {
    maskedCard := l.maskCardNumber(cardNumber)
    
    fmt.Printf("[%s] Payment request - Amount: $%.2f, Card: %s\n", 
               time.Now().Format("2006-01-02 15:04:05"), amount, maskedCard)
    
    startTime := time.Now()
    result := l.realPaymentService.ProcessPayment(amount, cardNumber)
    duration := time.Since(startTime)
    
    status := "successful"
    if !result {
        status = "failed"
    }
    
    fmt.Printf("[%s] Payment %s - Duration: %v\n", 
               time.Now().Format("2006-01-02 15:04:05"), status, duration)
    
    return result
}

func (l *LoggingPaymentProxy) maskCardNumber(cardNumber string) string {
    if len(cardNumber) < 4 {
        return "****"
    }
    return "**** **** **** " + cardNumber[len(cardNumber)-4:]
}

// Usage
func main() {
    paymentService := NewLoggingPaymentProxy()
    
    paymentService.ProcessPayment(100.50, "1234567890123456")
    paymentService.ProcessPayment(-50.0, "9876543210987654")
}
```

## Language-Specific Differences

### Type System

| Feature | Java | Go |
|---------|------|-----|
| **Interface Implementation** | Explicit (`implements` keyword) | Implicit (duck typing) |
| **Generics** | Full generics support | Limited generics (Go 1.18+) |
| **Inheritance** | Class-based inheritance | Composition over inheritance |
| **Null Safety** | NullPointerException prone | Explicit nil handling |

### Error Handling

#### Java
```java
// Exception-based error handling
try {
    result = service.processData();
} catch (ServiceException e) {
    logger.error("Service failed", e);
}
```

#### Go
```go
// Explicit error handling
result, err := service.ProcessData()
if err != nil {
    log.Printf("Service failed: %v", err)
}
```

### Memory Management

#### Java
- Automatic garbage collection
- Object pooling for performance
- References and object lifecycle managed by JVM

#### Go
- Garbage collection with lower latency
- Stack vs heap allocation optimization
- Explicit pointer management when needed

## Best Practices

### Java Best Practices
1. **Use interfaces** for loose coupling
2. **Implement proper equals/hashCode** for cached objects
3. **Handle exceptions** appropriately in proxy logic
4. **Consider thread safety** for shared proxies
5. **Use dependency injection** frameworks (Spring, Guice)

### Go Best Practices
1. **Return errors explicitly** instead of panicking
2. **Use pointer receivers** for methods that modify state
3. **Implement proper initialization** with factory functions
4. **Handle nil checks** appropriately
5. **Use context.Context** for cancellation and timeouts

### Common Best Practices (Both Languages)
1. **Keep proxy interface identical** to the real subject
2. **Implement lazy initialization** when appropriate
3. **Add proper logging and monitoring**
4. **Consider proxy chain patterns** for multiple concerns
5. **Document proxy behavior clearly**

## Performance Considerations

### Java Performance
- **JIT compilation** optimizes frequently used code paths
- **Object creation overhead** can be significant
- **GC pauses** may affect real-time applications
- **Thread contention** in multi-threaded proxy scenarios

### Go Performance
- **Compiled binaries** with predictable performance
- **Goroutines** for lightweight concurrency
- **Lower memory footprint** compared to Java
- **Faster startup times**

### Optimization Tips
1. **Pool expensive objects** when possible
2. **Use async patterns** for I/O-bound operations
3. **Implement circuit breakers** for fault tolerance
4. **Cache frequently accessed data**
5. **Monitor proxy performance metrics**

## Conclusion

The Proxy pattern is implemented differently in Java and Go due to their distinct language features:

- **Java** relies on explicit interface implementation and exception handling
- **Go** uses implicit interface satisfaction and explicit error handling
- Both languages support the pattern effectively, with trade-offs in verbosity, performance, and type safety

Choose the implementation approach that best fits your application's requirements, considering factors like performance, maintainability, and team expertise.