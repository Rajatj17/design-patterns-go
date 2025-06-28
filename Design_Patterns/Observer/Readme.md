# Observer Pattern - Interview Guide

## 1. Definition & Intent
**What is the Observer Pattern?**
- Defines a one-to-many dependency between objects
- When one object (Subject) changes state, all dependents (Observers) are automatically notified
- Promotes loose coupling between subject and observers
- Also known as Publisher-Subscriber (Pub/Sub) pattern

**Key Components:**
- **Subject/Publisher**: Maintains list of observers and notifies them of changes
- **Observer/Subscriber**: Interface for objects that should be notified
- **Concrete Observer**: Implements observer interface with specific update logic
- **Concrete Subject**: Stores state and notifies observers when it changes

## 2. Common Interview Questions

### Basic Questions
- **Q: What is the Observer pattern and when would you use it?**
- **Q: What's the difference between Observer and Pub/Sub?**
- **Q: How does Observer pattern promote loose coupling?**

### Advanced Questions
- **Q: How do you handle observer registration/deregistration at runtime?**
- **Q: What are the performance implications of the Observer pattern?**
- **Q: How do you implement Observer pattern in distributed systems?**
- **Q: How do you handle error propagation in Observer pattern?**
- **Q: What's the difference between push and pull models in Observer?**

## 3. Implementation Examples

### Basic Observer Pattern
```go
package main

import (
    "fmt"
    "sync"
)

// Observer interface
type Observer interface {
    Update(subject Subject)
    GetID() string
}

// Subject interface
type Subject interface {
    RegisterObserver(observer Observer)
    RemoveObserver(observer Observer)
    NotifyObservers()
}

// Concrete Subject - Stock Price
type StockPrice struct {
    symbol    string
    price     float64
    observers map[string]Observer
    mutex     sync.RWMutex
}

func NewStockPrice(symbol string) *StockPrice {
    return &StockPrice{
        symbol:    symbol,
        observers: make(map[string]Observer),
    }
}

func (s *StockPrice) RegisterObserver(observer Observer) {
    s.mutex.Lock()
    defer s.mutex.Unlock()
    s.observers[observer.GetID()] = observer
    fmt.Printf("Observer %s registered for %s\n", observer.GetID(), s.symbol)
}

func (s *StockPrice) RemoveObserver(observer Observer) {
    s.mutex.Lock()
    defer s.mutex.Unlock()
    delete(s.observers, observer.GetID())
    fmt.Printf("Observer %s removed from %s\n", observer.GetID(), s.symbol)
}

func (s *StockPrice) NotifyObservers() {
    s.mutex.RLock()
    defer s.mutex.RUnlock()
    
    for _, observer := range s.observers {
        go observer.Update(s) // Async notification
    }
}

func (s *StockPrice) SetPrice(price float64) {
    s.mutex.Lock()
    oldPrice := s.price
    s.price = price
    s.mutex.Unlock()
    
    if oldPrice != price {
        fmt.Printf("Stock %s price changed: $%.2f -> $%.2f\n", s.symbol, oldPrice, price)
        s.NotifyObservers()
    }
}

func (s *StockPrice) GetPrice() float64 {
    s.mutex.RLock()
    defer s.mutex.RUnlock()
    return s.price
}

func (s *StockPrice) GetSymbol() string {
    return s.symbol
}

// Concrete Observers
type StockAlert struct {
    id        string
    threshold float64
}

func NewStockAlert(id string, threshold float64) *StockAlert {
    return &StockAlert{id: id, threshold: threshold}
}

func (sa *StockAlert) Update(subject Subject) {
    if stockPrice, ok := subject.(*StockPrice); ok {
        price := stockPrice.GetPrice()
        if price >= sa.threshold {
            fmt.Printf("🚨 ALERT [%s]: %s reached $%.2f (threshold: $%.2f)\n", 
                sa.id, stockPrice.GetSymbol(), price, sa.threshold)
        }
    }
}

func (sa *StockAlert) GetID() string {
    return sa.id
}

type Portfolio struct {
    id     string
    stocks map[string]float64
    mutex  sync.RWMutex
}

func NewPortfolio(id string) *Portfolio {
    return &Portfolio{
        id:     id,
        stocks: make(map[string]float64),
    }
}

func (p *Portfolio) Update(subject Subject) {
    if stockPrice, ok := subject.(*StockPrice); ok {
        p.mutex.Lock()
        defer p.mutex.Unlock()
        
        symbol := stockPrice.GetSymbol()
        price := stockPrice.GetPrice()
        p.stocks[symbol] = price
        
        fmt.Printf("📊 Portfolio [%s]: Updated %s to $%.2f\n", p.id, symbol, price)
    }
}

func (p *Portfolio) GetID() string {
    return p.id
}

func (p *Portfolio) GetTotalValue() float64 {
    p.mutex.RLock()
    defer p.mutex.RUnlock()
    
    total := 0.0
    for _, price := range p.stocks {
        total += price
    }
    return total
}

// Usage
func main() {
    // Create subject
    appleStock := NewStockPrice("AAPL")
    
    // Create observers
    alert1 := NewStockAlert("HighAlert", 150.0)
    alert2 := NewStockAlert("LowAlert", 100.0)
    portfolio := NewPortfolio("MyPortfolio")
    
    // Register observers
    appleStock.RegisterObserver(alert1)
    appleStock.RegisterObserver(alert2)
    appleStock.RegisterObserver(portfolio)
    
    // Trigger updates
    appleStock.SetPrice(120.0)
    appleStock.SetPrice(155.0)
    appleStock.SetPrice(95.0)
    
    // Remove observer
    appleStock.RemoveObserver(alert1)
    appleStock.SetPrice(160.0)
}
```

### Event-Driven System with Channels
```go
package main

import (
    "context"
    "fmt"
    "sync"
    "time"
)

// Event types
type EventType string

const (
    UserRegistered EventType = "user.registered"
    OrderCreated   EventType = "order.created"
    PaymentFailed  EventType = "payment.failed"
)

// Event structure
type Event struct {
    Type      EventType
    Data      interface{}
    Timestamp time.Time
}

// Event handler function type
type EventHandler func(ctx context.Context, event Event) error

// Event bus using channels
type EventBus struct {
    handlers map[EventType][]EventHandler
    eventCh  chan Event
    ctx      context.Context
    cancel   context.CancelFunc
    wg       sync.WaitGroup
    mutex    sync.RWMutex
}

func NewEventBus(ctx context.Context) *EventBus {
    busCtx, cancel := context.WithCancel(ctx)
    bus := &EventBus{
        handlers: make(map[EventType][]EventHandler),
        eventCh:  make(chan Event, 100), // Buffered channel
        ctx:      busCtx,
        cancel:   cancel,
    }
    
    // Start event processing goroutine
    bus.wg.Add(1)
    go bus.processEvents()
    
    return bus
}

func (eb *EventBus) Subscribe(eventType EventType, handler EventHandler) {
    eb.mutex.Lock()
    defer eb.mutex.Unlock()
    eb.handlers[eventType] = append(eb.handlers[eventType], handler)
}

func (eb *EventBus) Publish(eventType EventType, data interface{}) error {
    event := Event{
        Type:      eventType,
        Data:      data,
        Timestamp: time.Now(),
    }
    
    select {
    case eb.eventCh <- event:
        return nil
    case <-eb.ctx.Done():
        return eb.ctx.Err()
    default:
        return fmt.Errorf("event bus channel full")
    }
}

func (eb *EventBus) processEvents() {
    defer eb.wg.Done()
    
    for {
        select {
        case event := <-eb.eventCh:
            eb.handleEvent(event)
        case <-eb.ctx.Done():
            return
        }
    }
}

func (eb *EventBus) handleEvent(event Event) {
    eb.mutex.RLock()
    handlers := eb.handlers[event.Type]
    eb.mutex.RUnlock()
    
    for _, handler := range handlers {
        go func(h EventHandler) {
            if err := h(eb.ctx, event); err != nil {
                fmt.Printf("Error handling event %s: %v\n", event.Type, err)
            }
        }(handler)
    }
}

func (eb *EventBus) Close() {
    eb.cancel()
    close(eb.eventCh)
    eb.wg.Wait()
}

// Example event handlers
type EmailService struct{}

func (es *EmailService) SendWelcomeEmail(ctx context.Context, event Event) error {
    userData := event.Data.(map[string]interface{})
    email := userData["email"].(string)
    fmt.Printf("📧 Sending welcome email to %s\n", email)
    return nil
}

func (es *EmailService) SendOrderConfirmation(ctx context.Context, event Event) error {
    orderData := event.Data.(map[string]interface{})
    email := orderData["email"].(string)
    orderID := orderData["order_id"].(string)
    fmt.Printf("📧 Sending order confirmation to %s for order %s\n", email, orderID)
    return nil
}

type AnalyticsService struct{}

func (as *AnalyticsService) TrackUserRegistration(ctx context.Context, event Event) error {
    userData := event.Data.(map[string]interface{})
    userID := userData["user_id"].(string)
    fmt.Printf("📊 Analytics: User %s registered\n", userID)
    return nil
}

func (as *AnalyticsService) TrackOrderCreated(ctx context.Context, event Event) error {
    orderData := event.Data.(map[string]interface{})
    orderID := orderData["order_id"].(string)
    amount := orderData["amount"].(float64)
    fmt.Printf("📊 Analytics: Order %s created for $%.2f\n", orderID, amount)
    return nil
}

type NotificationService struct{}

func (ns *NotificationService) SendPushNotification(ctx context.Context, event Event) error {
    userData := event.Data.(map[string]interface{})
    userID := userData["user_id"].(string)
    fmt.Printf("📱 Push notification sent to user %s\n", userID)
    return nil
}

// Usage
func main() {
    ctx := context.Background()
    eventBus := NewEventBus(ctx)
    defer eventBus.Close()
    
    // Create services
    emailService := &EmailService{}
    analyticsService := &AnalyticsService{}
    notificationService := &NotificationService{}
    
    // Subscribe to events
    eventBus.Subscribe(UserRegistered, emailService.SendWelcomeEmail)
    eventBus.Subscribe(UserRegistered, analyticsService.TrackUserRegistration)
    eventBus.Subscribe(UserRegistered, notificationService.SendPushNotification)
    
    eventBus.Subscribe(OrderCreated, emailService.SendOrderConfirmation)
    eventBus.Subscribe(OrderCreated, analyticsService.TrackOrderCreated)
    
    // Publish events
    eventBus.Publish(UserRegistered, map[string]interface{}{
        "user_id": "user123",
        "email":   "user@example.com",
        "name":    "John Doe",
    })
    
    eventBus.Publish(OrderCreated, map[string]interface{}{
        "order_id": "order456",
        "user_id":  "user123",
        "email":    "user@example.com",
        "amount":   99.99,
    })
    
    // Wait for events to be processed
    time.Sleep(2 * time.Second)
}
```

### Reactive Streams Pattern
```go
package main

import (
    "context"
    "fmt"
    "sync"
    "time"
)

// Stream interface
type Stream interface {
    Subscribe(observer StreamObserver) Subscription
    Emit(value interface{})
    Close()
}

// Stream observer interface
type StreamObserver interface {
    OnNext(value interface{})
    OnError(err error)
    OnComplete()
}

// Subscription interface
type Subscription interface {
    Unsubscribe()
    IsSubscribed() bool
}

// Concrete stream implementation
type ReactiveStream struct {
    observers map[string]StreamObserver
    mutex     sync.RWMutex
    closed    bool
}

func NewReactiveStream() *ReactiveStream {
    return &ReactiveStream{
        observers: make(map[string]StreamObserver),
    }
}

func (rs *ReactiveStream) Subscribe(observer StreamObserver) Subscription {
    rs.mutex.Lock()
    defer rs.mutex.Unlock()
    
    id := fmt.Sprintf("observer_%d", len(rs.observers))
    rs.observers[id] = observer
    
    return &StreamSubscription{
        stream:     rs,
        observerID: id,
        active:     true,
    }
}

func (rs *ReactiveStream) Emit(value interface{}) {
    rs.mutex.RLock()
    defer rs.mutex.RUnlock()
    
    if rs.closed {
        return
    }
    
    for _, observer := range rs.observers {
        go observer.OnNext(value)
    }
}

func (rs *ReactiveStream) EmitError(err error) {
    rs.mutex.RLock()
    defer rs.mutex.RUnlock()
    
    for _, observer := range rs.observers {
        go observer.OnError(err)
    }
}

func (rs *ReactiveStream) Close() {
    rs.mutex.Lock()
    defer rs.mutex.Unlock()
    
    if rs.closed {
        return
    }
    
    rs.closed = true
    for _, observer := range rs.observers {
        go observer.OnComplete()
    }
    rs.observers = make(map[string]StreamObserver)
}

func (rs *ReactiveStream) removeObserver(observerID string) {
    rs.mutex.Lock()
    defer rs.mutex.Unlock()
    delete(rs.observers, observerID)
}

// Subscription implementation
type StreamSubscription struct {
    stream     *ReactiveStream
    observerID string
    active     bool
    mutex      sync.Mutex
}

func (ss *StreamSubscription) Unsubscribe() {
    ss.mutex.Lock()
    defer ss.mutex.Unlock()
    
    if ss.active {
        ss.stream.removeObserver(ss.observerID)
        ss.active = false
    }
}

func (ss *StreamSubscription) IsSubscribed() bool {
    ss.mutex.Lock()
    defer ss.mutex.Unlock()
    return ss.active
}

// Concrete observers
type LoggingObserver struct {
    name string
}

func (lo *LoggingObserver) OnNext(value interface{}) {
    fmt.Printf("[%s] Received: %v\n", lo.name, value)
}

func (lo *LoggingObserver) OnError(err error) {
    fmt.Printf("[%s] Error: %v\n", lo.name, err)
}

func (lo *LoggingObserver) OnComplete() {
    fmt.Printf("[%s] Stream completed\n", lo.name)
}

type MetricsObserver struct {
    count int
    mutex sync.Mutex
}

func (mo *MetricsObserver) OnNext(value interface{}) {
    mo.mutex.Lock()
    defer mo.mutex.Unlock()
    mo.count++
    fmt.Printf("📊 Metrics: Total events received: %d\n", mo.count)
}

func (mo *MetricsObserver) OnError(err error) {
    fmt.Printf("📊 Metrics: Error occurred: %v\n", err)
}

func (mo *MetricsObserver) OnComplete() {
    mo.mutex.Lock()
    defer mo.mutex.Unlock()
    fmt.Printf("📊 Metrics: Final count: %d events\n", mo.count)
}

// Usage
func main() {
    stream := NewReactiveStream()
    
    // Create observers
    logger := &LoggingObserver{name: "Logger"}
    metrics := &MetricsObserver{}
    
    // Subscribe
    loggerSub := stream.Subscribe(logger)
    metricsSub := stream.Subscribe(metrics)
    
    // Emit some values
    go func() {
        for i := 1; i <= 5; i++ {
            stream.Emit(fmt.Sprintf("Event %d", i))
            time.Sleep(500 * time.Millisecond)
        }
        
        // Unsubscribe logger after 3 seconds
        time.Sleep(1 * time.Second)
        loggerSub.Unsubscribe()
        
        // Emit more values
        for i := 6; i <= 8; i++ {
            stream.Emit(fmt.Sprintf("Event %d", i))
            time.Sleep(500 * time.Millisecond)
        }
        
        stream.Close()
    }()
    
    // Wait for completion
    time.Sleep(5 * time.Second)
    
    fmt.Printf("Logger still subscribed: %v\n", loggerSub.IsSubscribed())
    fmt.Printf("Metrics still subscribed: %v\n", metricsSub.IsSubscribed())
}
```

## 4. Real-World Use Cases

### Backend Engineering Applications:
- **Event-Driven Architecture**: Microservices communication
- **Database Change Streams**: React to data changes
- **Message Queues**: Pub/Sub messaging systems
- **WebSocket Connections**: Real-time updates to clients
- **Monitoring & Alerting**: System metric thresholds
- **Audit Logging**: Track system events
- **Cache Invalidation**: Update caches when data changes

### System Design Example:
```go
// E-commerce order processing system
type OrderProcessor struct {
    observers []OrderObserver
}

type OrderObserver interface {
    OnOrderCreated(order Order)
    OnPaymentProcessed(order Order)
    OnOrderShipped(order Order)
}

// Observers: InventoryService, EmailService, AnalyticsService, etc.
```

## 5. Problems & Criticisms

### Issues with Observer Pattern:
1. **Memory Leaks**: Observers not properly unregistered
2. **Performance**: Too many observers can slow down notifications
3. **Order Dependency**: No guarantee of notification order
4. **Error Propagation**: One observer failure shouldn't affect others
5. **Circular Dependencies**: Observers triggering more events
6. **Debugging Difficulty**: Hard to trace event flows

### Common Pitfalls:
```go
// ❌ Memory leak - observer never removed
func badExample() {
    subject := NewSubject()
    observer := NewObserver()
    subject.RegisterObserver(observer)
    // observer goes out of scope but still registered
}

// ✅ Proper cleanup
func goodExample() {
    subject := NewSubject()
    observer := NewObserver()
    subject.RegisterObserver(observer)
    defer subject.RemoveObserver(observer)
}
```

## 6. Modern Alternatives & Best Practices

### Go Channels vs Observer:
```go
// Channel-based approach (often preferred in Go)
func channelApproach() {
    events := make(chan Event, 100)
    
    // Multiple subscribers
    go emailSubscriber(events)
    go analyticsSubscriber(events)
    go loggingSubscriber(events)
    
    // Publisher
    events <- Event{Type: "user.created"}
}
```

### Message Queue Integration:
```go
// Using message queues for distributed observers
type MessageQueueObserver struct {
    queue MessageQueue
    topic string
}

func (mqo *MessageQueueObserver) Update(subject Subject) {
    message := createMessage(subject)
    mqo.queue.Publish(mqo.topic, message)
}
```

## 7. Interview Red Flags to Avoid

### Don't Say:
- "Observer pattern is always better than polling"
- "All events should use Observer pattern"
- "Observer pattern has no performance issues"

### Do Say:
- "Observer is useful for loose coupling but has memory management considerations"
- "In Go, channels are often preferred for event handling"
- "Need to consider error handling and observer lifecycle"

## 8. Advanced Topics

### Error Handling Strategies:
```go
func (s *SafeSubject) NotifyObservers() {
    for _, observer := range s.observers {
        func(obs Observer) {
            defer func() {
                if r := recover(); r != nil {
                    log.Printf("Observer panic: %v", r)
                }
            }()
            obs.Update(s)
        }(observer)
    }
}
```

### Async vs Sync Notifications:
```go
// Sync - blocks until all observers complete
func (s *Subject) NotifySync() {
    for _, obs := range s.observers {
        obs.Update(s)
    }
}

// Async - doesn't block
func (s *Subject) NotifyAsync() {
    for _, obs := range s.observers {
        go obs.Update(s)
    }
}
```

## 9. Key Takeaways for Interview

1. **Understand the core concept**: One-to-many notification
2. **Know Go-specific approaches**: Channels vs traditional Observer
3. **Discuss real-world applications**: Event-driven systems, monitoring
4. **Address common problems**: Memory leaks, error handling
5. **Compare with alternatives**: Message queues, channels

## 10. Practice Questions

1. Implement an Observer pattern for a chat system
2. Design an event-driven order processing system
3. How would you implement Observer pattern across microservices?
4. Create an Observer system with priority-based notifications
5. Implement Observer pattern with filtering (conditional notifications)
6. Design a reactive stream system for real-time data processing
7. How do you handle back-pressure in Observer pattern?

### Go-Specific Considerations:
- **Goroutine safety** in observer management
- **Channel-based implementations** vs traditional Observer
- **Context cancellation** for observer cleanup
- **Error handling** without panic propagation
- **Memory management** and observer lifecycle
- **Performance considerations** with many observers