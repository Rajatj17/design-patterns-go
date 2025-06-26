# Singleton Pattern - Interview Guide

## 1. Definition & Intent
**What is the Singleton Pattern?**
- Ensures a class has only one instance and provides global point of access to it
- Restricts instantiation of a class to a single object
- Useful when exactly one object is needed to coordinate actions across the system

## 2. Common Interview Questions

### Basic Questions
- **Q: What is the Singleton pattern and when would you use it?**
- **Q: What are the key components of Singleton pattern?**
- **Q: How do you implement a thread-safe Singleton?**

### Advanced Questions
- **Q: What are the problems with Singleton pattern? Why is it considered an anti-pattern?**
- **Q: How does Singleton pattern violate SOLID principles?**
- **Q: What are alternatives to Singleton pattern?**
- **Q: How do you test code that uses Singletons?**

## 3. Implementation Examples

### Basic Implementation (Not Thread-Safe)
```go
package main

import "database/sql"

type DatabaseConnection struct {
    db *sql.DB
}

var instance *DatabaseConnection

func GetInstance() *DatabaseConnection {
    if instance == nil {
        instance = &DatabaseConnection{}
        // Initialize database connection
    }
    return instance
}

func (d *DatabaseConnection) Connect() {
    // connection logic
}
```

### Thread-Safe Implementations

#### 1. Using Mutex (Simple but Performance Impact)
```go
package main

import (
    "database/sql"
    "sync"
)

type DatabaseConnection struct {
    db *sql.DB
}

var (
    instance *DatabaseConnection
    mutex    sync.Mutex
)

func GetInstance() *DatabaseConnection {
    mutex.Lock()
    defer mutex.Unlock()
    
    if instance == nil {
        instance = &DatabaseConnection{}
        // Initialize database connection
    }
    return instance
}
```

#### 2. Using sync.Once (Preferred - Go Idiomatic)
```go
package main

import (
    "database/sql"
    "sync"
)

type DatabaseConnection struct {
    db *sql.DB
}

var (
    instance *DatabaseConnection
    once     sync.Once
)

func GetInstance() *DatabaseConnection {
    once.Do(func() {
        instance = &DatabaseConnection{}
        // Initialize database connection
    })
    return instance
}

func (d *DatabaseConnection) Connect() error {
    // connection logic
    return nil
}
```

#### 3. Package-Level Variable (Eager Initialization)
```go
package database

import "database/sql"

type Connection struct {
    db *sql.DB
}

// Initialized when package is imported
var Instance = &Connection{}

func init() {
    // Initialize database connection
    Instance.db, _ = sql.Open("mysql", "connection_string")
}

func (c *Connection) Query(query string) (*sql.Rows, error) {
    return c.db.Query(query)
}
```

#### 4. Using Atomic Operations (Advanced)
```go
package main

import (
    "sync"
    "sync/atomic"
    "unsafe"
)

type DatabaseConnection struct {
    db interface{} // your actual DB connection
}

var (
    instance unsafe.Pointer
    mutex    sync.Mutex
)

func GetInstance() *DatabaseConnection {
    if atomic.LoadPointer(&instance) == nil {
        mutex.Lock()
        defer mutex.Unlock()
        if atomic.LoadPointer(&instance) == nil {
            atomic.StorePointer(&instance, unsafe.Pointer(&DatabaseConnection{}))
        }
    }
    return (*DatabaseConnection)(atomic.LoadPointer(&instance))
}
```

#### 5. Interface-Based Singleton (Best Practice for Testing)
```go
package main

import (
    "database/sql"
    "sync"
)

// Define interface for dependency injection
type DatabaseConnectionInterface interface {
    Query(query string) (*sql.Rows, error)
    Close() error
}

type DatabaseConnection struct {
    db *sql.DB
}

var (
    instance DatabaseConnectionInterface
    once     sync.Once
)

func GetInstance() DatabaseConnectionInterface {
    once.Do(func() {
        instance = &DatabaseConnection{}
        // Initialize database connection
    })
    return instance
}

func (d *DatabaseConnection) Query(query string) (*sql.Rows, error) {
    return d.db.Query(query)
}

func (d *DatabaseConnection) Close() error {
    return d.db.Close()
}
```

## 4. Real-World Use Cases

### Appropriate Uses:
- **Database Connection Pools**: Managing limited database connections
- **Logger Classes**: Centralized logging mechanism
- **Configuration Managers**: Application-wide settings
- **Cache Managers**: Shared cache instances
- **Thread Pools**: Managing worker threads

### Code Example - Configuration Manager:
```go
package config

import (
    "os"
    "sync"
)

type ConfigurationManager struct {
    settings map[string]string
}

var (
    instance *ConfigurationManager
    once     sync.Once
)

func GetInstance() *ConfigurationManager {
    once.Do(func() {
        instance = &ConfigurationManager{
            settings: make(map[string]string),
        }
        instance.loadConfiguration()
    })
    return instance
}

func (c *ConfigurationManager) GetProperty(key string) string {
    if value, exists := c.settings[key]; exists {
        return value
    }
    return ""
}

func (c *ConfigurationManager) loadConfiguration() {
    // Load from environment variables
    c.settings["db_host"] = os.Getenv("DB_HOST")
    c.settings["db_port"] = os.Getenv("DB_PORT")
    // Load from file, etc.
}
```

## 5. Problems & Criticisms

### Issues with Singleton:
1. **Hidden Dependencies**: Makes dependencies unclear
2. **Testing Difficulties**: Hard to mock and unit test
3. **Violates Single Responsibility**: Managing instance + business logic
4. **Global State**: Creates implicit coupling between classes
5. **Concurrency Issues**: Thread safety complications
6. **Subclassing Problems**: Difficult to extend

### Example of Testing Problem:
```go
// Hard to test because of hidden dependency
type OrderService struct{}

func (o *OrderService) ProcessOrder(order Order) error {
    // Hidden dependency on Singleton
    db := GetInstance()
    return db.Save(order)
}

// Testing becomes difficult - can't mock the database
func TestProcessOrder(t *testing.T) {
    service := &OrderService{}
    // How do we mock GetInstance()? Very difficult!
    err := service.ProcessOrder(Order{ID: 1})
    // Test becomes brittle and dependent on real database
}
```

## 6. Modern Alternatives

### Dependency Injection:
```go
// Instead of Singleton, use dependency injection
type DatabaseConnectionInterface interface {
    Save(order Order) error
}

type OrderService struct {
    dbConnection DatabaseConnectionInterface
}

func NewOrderService(dbConnection DatabaseConnectionInterface) *OrderService {
    return &OrderService{
        dbConnection: dbConnection,
    }
}

func (o *OrderService) ProcessOrder(order Order) error {
    return o.dbConnection.Save(order)
}

// Easy to test with mock
func TestProcessOrder(t *testing.T) {
    mockDB := &MockDatabase{}
    service := NewOrderService(mockDB)
    
    err := service.ProcessOrder(Order{ID: 1})
    assert.NoError(t, err)
}

type MockDatabase struct{}

func (m *MockDatabase) Save(order Order) error {
    // Mock implementation
    return nil
}
```

### Using Dependency Injection Frameworks:
```go
// Using wire (Google's dependency injection for Go)
//go:build wireinject

package main

import "github.com/google/wire"

func InitializeOrderService() *OrderService {
    wire.Build(
        NewDatabaseConnection,
        NewOrderService,
    )
    return nil
}

// Or using fx (Uber's dependency injection framework)
package main

import (
    "go.uber.org/fx"
)

func main() {
    fx.New(
        fx.Provide(
            NewDatabaseConnection,
            NewOrderService,
        ),
        fx.Invoke(runOrderService),
    ).Run()
}

func runOrderService(service *OrderService) {
    // Use the service
}
```

## 7. Interview Red Flags to Avoid

### Don't Say:
- "Singleton is always the best choice for global objects"
- "Thread safety doesn't matter in Singleton"
- "Singleton is the same as static class"

### Do Say:
- "Singleton should be used sparingly due to testing and coupling issues"
- "Modern frameworks provide better alternatives through DI containers"
- "Thread safety is crucial in multi-threaded environments"

## 8. Follow-up Discussion Points

### Be Prepared to Discuss:
- **Serialization Issues**: How to maintain singleton during deserialization
- **Reflection Attacks**: How reflection can break singleton
- **Classloader Issues**: Multiple classloaders can create multiple instances
- **Memory Leaks**: How singletons can cause memory leaks

### Advanced Topics:
- **Registry of Singletons**: Managing multiple singleton types
- **Lazy Loading vs Eager Loading**: Trade-offs in initialization
- **Singleton in Distributed Systems**: Challenges across multiple JVMs

## 9. Key Takeaways for Interview

1. **Know multiple implementation approaches** and their trade-offs
2. **Understand why it's controversial** in modern software design
3. **Be able to suggest alternatives** like dependency injection
4. **Discuss real-world scenarios** where you've used or avoided it
5. **Show awareness of testing implications**

## 10. Practice Questions

1. Implement a thread-safe Singleton using sync.Once in Go
2. Explain why sync.Once is preferred over mutex for Singleton in Go
3. How would you make a Singleton work with JSON serialization in Go?
4. What happens with Goroutines and Singleton pattern?
5. Design a system using dependency injection instead of Singleton in Go
6. How do you handle Singleton cleanup in Go (finalizers, context cancellation)?
7. Implement a Singleton that can be reset for testing purposes

### Go-Specific Considerations:
- **Package-level variables vs Singleton pattern**
- **sync.Once vs mutex performance**
- **Goroutine safety implications**
- **Testing strategies with interfaces**
- **Memory management and garbage collection**