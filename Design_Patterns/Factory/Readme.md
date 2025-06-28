# Factory Pattern - Interview Guide

## 1. Definition & Intent
**What is the Factory Pattern?**
- Creates objects without specifying their exact classes
- Provides an interface for creating families of related objects
- Encapsulates object creation logic and promotes loose coupling
- Delegates object instantiation to factory methods or classes

**Types of Factory Patterns:**
- **Simple Factory** (Factory Method)
- **Factory Method Pattern**
- **Abstract Factory Pattern**

## 2. Common Interview Questions

### Basic Questions
- **Q: What is the Factory pattern and when would you use it?**
- **Q: What's the difference between Factory Method and Abstract Factory?**
- **Q: How does Factory pattern promote loose coupling?**

### Advanced Questions
- **Q: How do you handle factory registration and discovery?**
- **Q: What are the trade-offs of using Factory pattern?**
- **Q: How do you implement Factory pattern in microservices architecture?**
- **Q: How does Factory pattern relate to Dependency Injection?**

## 3. Implementation Examples

### Simple Factory Pattern
```go
package main

import (
    "fmt"
    "errors"
)

// Product interface
type Logger interface {
    Log(message string)
}

// Concrete products
type FileLogger struct {
    filename string
}

func (f *FileLogger) Log(message string) {
    fmt.Printf("File[%s]: %s\n", f.filename, message)
}

type ConsoleLogger struct{}

func (c *ConsoleLogger) Log(message string) {
    fmt.Printf("Console: %s\n", message)
}

type DatabaseLogger struct {
    connectionString string
}

func (d *DatabaseLogger) Log(message string) {
    fmt.Printf("DB[%s]: %s\n", d.connectionString, message)
}

// Simple Factory
type LoggerFactory struct{}

func (lf *LoggerFactory) CreateLogger(loggerType string) (Logger, error) {
    switch loggerType {
    case "file":
        return &FileLogger{filename: "app.log"}, nil
    case "console":
        return &ConsoleLogger{}, nil
    case "database":
        return &DatabaseLogger{connectionString: "db://localhost"}, nil
    default:
        return nil, errors.New("unknown logger type")
    }
}

// Usage
func main() {
    factory := &LoggerFactory{}
    
    logger, err := factory.CreateLogger("console")
    if err != nil {
        panic(err)
    }
    
    logger.Log("Hello, World!")
}
```

### Factory Method Pattern
```go
package main

import "fmt"

// Product interface
type PaymentProcessor interface {
    ProcessPayment(amount float64) error
}

// Concrete products
type CreditCardProcessor struct{}

func (c *CreditCardProcessor) ProcessPayment(amount float64) error {
    fmt.Printf("Processing $%.2f via Credit Card\n", amount)
    return nil
}

type PayPalProcessor struct{}

func (p *PayPalProcessor) ProcessPayment(amount float64) error {
    fmt.Printf("Processing $%.2f via PayPal\n", amount)
    return nil
}

type BankTransferProcessor struct{}

func (b *BankTransferProcessor) ProcessPayment(amount float64) error {
    fmt.Printf("Processing $%.2f via Bank Transfer\n", amount)
    return nil
}

// Creator interface (Factory Method)
type PaymentProcessorFactory interface {
    CreateProcessor() PaymentProcessor
}

// Concrete creators
type CreditCardFactory struct{}

func (c *CreditCardFactory) CreateProcessor() PaymentProcessor {
    return &CreditCardProcessor{}
}

type PayPalFactory struct{}

func (p *PayPalFactory) CreateProcessor() PaymentProcessor {
    return &PayPalProcessor{}
}

type BankTransferFactory struct{}

func (b *BankTransferFactory) CreateProcessor() PaymentProcessor {
    return &BankTransferProcessor{}
}

// Client code
type PaymentService struct {
    factory PaymentProcessorFactory
}

func NewPaymentService(factory PaymentProcessorFactory) *PaymentService {
    return &PaymentService{factory: factory}
}

func (ps *PaymentService) ProcessOrder(amount float64) error {
    processor := ps.factory.CreateProcessor()
    return processor.ProcessPayment(amount)
}

// Usage
func main() {
    // Can easily switch payment methods
    service := NewPaymentService(&PayPalFactory{})
    service.ProcessOrder(100.50)
    
    service = NewPaymentService(&CreditCardFactory{})
    service.ProcessOrder(200.75)
}
```

### Abstract Factory Pattern
```go
package main

import "fmt"

// Abstract products
type Database interface {
    Connect() error
    Query(sql string) ([]map[string]interface{}, error)
}

type MessageQueue interface {
    Publish(topic string, message interface{}) error
    Subscribe(topic string) (<-chan interface{}, error)
}

// MySQL implementations
type MySQLDatabase struct {
    host string
}

func (m *MySQLDatabase) Connect() error {
    fmt.Printf("Connecting to MySQL at %s\n", m.host)
    return nil
}

func (m *MySQLDatabase) Query(sql string) ([]map[string]interface{}, error) {
    fmt.Printf("Executing MySQL query: %s\n", sql)
    return nil, nil
}

type RabbitMQ struct {
    host string
}

func (r *RabbitMQ) Publish(topic string, message interface{}) error {
    fmt.Printf("Publishing to RabbitMQ topic %s: %v\n", topic, message)
    return nil
}

func (r *RabbitMQ) Subscribe(topic string) (<-chan interface{}, error) {
    fmt.Printf("Subscribing to RabbitMQ topic: %s\n", topic)
    return make(chan interface{}), nil
}

// PostgreSQL implementations
type PostgreSQLDatabase struct {
    host string
}

func (p *PostgreSQLDatabase) Connect() error {
    fmt.Printf("Connecting to PostgreSQL at %s\n", p.host)
    return nil
}

func (p *PostgreSQLDatabase) Query(sql string) ([]map[string]interface{}, error) {
    fmt.Printf("Executing PostgreSQL query: %s\n", sql)
    return nil, nil
}

type ApacheKafka struct {
    brokers []string
}

func (k *ApacheKafka) Publish(topic string, message interface{}) error {
    fmt.Printf("Publishing to Kafka topic %s: %v\n", topic, message)
    return nil
}

func (k *ApacheKafka) Subscribe(topic string) (<-chan interface{}, error) {
    fmt.Printf("Subscribing to Kafka topic: %s\n", topic)
    return make(chan interface{}), nil
}

// Abstract Factory interface
type InfrastructureFactory interface {
    CreateDatabase() Database
    CreateMessageQueue() MessageQueue
}

// Concrete factories
type MySQLRabbitMQFactory struct{}

func (m *MySQLRabbitMQFactory) CreateDatabase() Database {
    return &MySQLDatabase{host: "mysql.example.com"}
}

func (m *MySQLRabbitMQFactory) CreateMessageQueue() MessageQueue {
    return &RabbitMQ{host: "rabbitmq.example.com"}
}

type PostgreSQLKafkaFactory struct{}

func (p *PostgreSQLKafkaFactory) CreateDatabase() Database {
    return &PostgreSQLDatabase{host: "postgres.example.com"}
}

func (p *PostgreSQLKafkaFactory) CreateMessageQueue() MessageQueue {
    return &ApacheKafka{brokers: []string{"kafka1.example.com", "kafka2.example.com"}}
}

// Client code
type Application struct {
    db    Database
    queue MessageQueue
}

func NewApplication(factory InfrastructureFactory) *Application {
    return &Application{
        db:    factory.CreateDatabase(),
        queue: factory.CreateMessageQueue(),
    }
}

func (app *Application) Start() error {
    if err := app.db.Connect(); err != nil {
        return err
    }
    
    app.queue.Publish("app.started", "Application initialized")
    return nil
}

// Usage
func main() {
    // Development environment
    devFactory := &MySQLRabbitMQFactory{}
    devApp := NewApplication(devFactory)
    devApp.Start()
    
    fmt.Println("---")
    
    // Production environment
    prodFactory := &PostgreSQLKafkaFactory{}
    prodApp := NewApplication(prodFactory)
    prodApp.Start()
}
```

### Registry-Based Factory (Advanced)
```go
package main

import (
    "errors"
    "fmt"
    "sync"
)

// Product interface
type NotificationSender interface {
    Send(recipient, message string) error
}

// Concrete products
type EmailSender struct{}

func (e *EmailSender) Send(recipient, message string) error {
    fmt.Printf("Email to %s: %s\n", recipient, message)
    return nil
}

type SMSSender struct{}

func (s *SMSSender) Send(recipient, message string) error {
    fmt.Printf("SMS to %s: %s\n", recipient, message)
    return nil
}

type PushNotificationSender struct{}

func (p *PushNotificationSender) Send(recipient, message string) error {
    fmt.Printf("Push to %s: %s\n", recipient, message)
    return nil
}

// Factory function type
type NotificationSenderFactory func() NotificationSender

// Registry-based factory
type NotificationFactory struct {
    factories map[string]NotificationSenderFactory
    mutex     sync.RWMutex
}

func NewNotificationFactory() *NotificationFactory {
    return &NotificationFactory{
        factories: make(map[string]NotificationSenderFactory),
    }
}

func (nf *NotificationFactory) Register(senderType string, factory NotificationSenderFactory) {
    nf.mutex.Lock()
    defer nf.mutex.Unlock()
    nf.factories[senderType] = factory
}

func (nf *NotificationFactory) Create(senderType string) (NotificationSender, error) {
    nf.mutex.RLock()
    defer nf.mutex.RUnlock()
    
    factory, exists := nf.factories[senderType]
    if !exists {
        return nil, errors.New("unknown sender type: " + senderType)
    }
    
    return factory(), nil
}

func (nf *NotificationFactory) GetAvailableTypes() []string {
    nf.mutex.RLock()
    defer nf.mutex.RUnlock()
    
    types := make([]string, 0, len(nf.factories))
    for senderType := range nf.factories {
        types = append(types, senderType)
    }
    return types
}

// Usage
func main() {
    factory := NewNotificationFactory()
    
    // Register factories
    factory.Register("email", func() NotificationSender {
        return &EmailSender{}
    })
    factory.Register("sms", func() NotificationSender {
        return &SMSSender{}
    })
    factory.Register("push", func() NotificationSender {
        return &PushNotificationSender{}
    })
    
    // Use factory
    sender, err := factory.Create("email")
    if err != nil {
        panic(err)
    }
    
    sender.Send("user@example.com", "Hello!")
    
    // List available types
    fmt.Println("Available notification types:", factory.GetAvailableTypes())
}
```

## 4. Real-World Use Cases

### Appropriate Uses:
- **Database Connections**: Different database drivers (MySQL, PostgreSQL, MongoDB)
- **Payment Processing**: Different payment gateways (Stripe, PayPal, Square)
- **Cloud Services**: Different cloud providers (AWS, GCP, Azure)
- **Serialization**: Different formats (JSON, XML, Protocol Buffers)
- **Caching**: Different cache implementations (Redis, Memcached, In-memory)

### Microservices Example:
```go
// Service discovery factory
type ServiceFactory interface {
    CreateUserService() UserService
    CreateOrderService() OrderService
    CreatePaymentService() PaymentService
}

type LocalServiceFactory struct{}

func (l *LocalServiceFactory) CreateUserService() UserService {
    return &LocalUserService{}
}

type RemoteServiceFactory struct {
    serviceRegistry ServiceRegistry
}

func (r *RemoteServiceFactory) CreateUserService() UserService {
    endpoint := r.serviceRegistry.Discover("user-service")
    return &RemoteUserService{endpoint: endpoint}
}
```

## 5. Problems & Criticisms

### Issues with Factory Pattern:
1. **Complexity**: Can add unnecessary complexity for simple cases
2. **Runtime Dependencies**: Product types determined at runtime
3. **Violation of Open/Closed**: Adding new products may require factory changes
4. **Hidden Dependencies**: Factories may have complex internal dependencies

### When NOT to Use Factory:
- When you only have one product type
- When product creation is simple and unlikely to change
- When the calling code needs to know the concrete type anyway

## 6. Modern Alternatives & Best Practices

### Dependency Injection with Factories:
```go
type ServiceContainer struct {
    userServiceFactory    func() UserService
    paymentServiceFactory func() PaymentService
}

func (sc *ServiceContainer) GetUserService() UserService {
    return sc.userServiceFactory()
}

// Using with DI frameworks
func NewServiceContainer() *ServiceContainer {
    return &ServiceContainer{
        userServiceFactory: func() UserService {
            return NewUserService(NewUserRepository())
        },
        paymentServiceFactory: func() PaymentService {
            return NewPaymentService(NewPaymentGateway())
        },
    }
}
```

### Configuration-Driven Factory:
```go
type Config struct {
    DatabaseType string `json:"database_type"`
    CacheType    string `json:"cache_type"`
}

type ComponentFactory struct {
    config Config
}

func (cf *ComponentFactory) CreateDatabase() Database {
    switch cf.config.DatabaseType {
    case "mysql":
        return &MySQLDatabase{}
    case "postgres":
        return &PostgreSQLDatabase{}
    default:
        return &InMemoryDatabase{}
    }
}
```

## 7. Interview Red Flags to Avoid

### Don't Say:
- "Factory pattern should be used for all object creation"
- "Simple Factory and Factory Method are the same thing"
- "Factory pattern eliminates all dependencies"

### Do Say:
- "Factory pattern is useful when creation logic is complex or varies"
- "Consider dependency injection for better testability"
- "Abstract Factory ensures product families work together"

## 8. Follow-up Discussion Points

### Be Prepared to Discuss:
- **Plugin Architecture**: How factories enable plugin systems
- **Configuration Management**: Factory selection based on config
- **Performance Implications**: Factory overhead vs flexibility
- **Error Handling**: How to handle factory creation failures

### Advanced Topics:
- **Factory Chaining**: Composing multiple factories
- **Lazy Initialization**: Creating products on-demand
- **Thread Safety**: Concurrent access to factory registries
- **Factory Lifecycle**: Managing factory and product lifetimes

## 9. Key Takeaways for Interview

1. **Know all three types** of factory patterns and their differences
2. **Understand when to use each type** based on complexity needs
3. **Show real-world examples** from your experience
4. **Discuss trade-offs** between flexibility and complexity
5. **Connect to modern practices** like dependency injection

## 10. Practice Questions

1. Implement a factory for different HTTP clients (REST, GraphQL, gRPC)
2. Design a factory system for different cloud storage providers
3. How would you implement a factory with plugin loading in Go?
4. Create a factory that supports both sync and async product creation
5. Design a factory pattern for a multi-tenant application
6. How do you handle factory configuration in a distributed system?
7. Implement a factory with graceful degradation (fallback products)

### Go-Specific Considerations:
- **Function factories vs struct factories**
- **Interface-based product creation**
- **Package-level factory functions**
- **Error handling in factory methods**
- **Goroutine safety in factory registries**
- **Using reflection for dynamic factory registration**