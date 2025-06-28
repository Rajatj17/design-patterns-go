# Chain of Responsibility Pattern Guide: Go vs Java

## Table of Contents
1. [Introduction](##introduction)
2. [Pattern Overview](#pattern-overview)
3. [Basic Implementation Comparison](#basic-implementation-comparison)
4. [Advanced Examples](#advanced-examples)
5. [Real-World Scenarios](#real-world-scenarios)
6. [Key Differences](#key-differences)
7. [Best Practices](#best-practices)
8. [When to Use](#when-to-use)

## Introduction

The Chain of Responsibility pattern is a behavioral design pattern that lets you pass requests along a chain of handlers. Upon receiving a request, each handler decides either to process the request or to pass it to the next handler in the chain. This pattern decouples the sender of a request from its receivers by giving multiple objects a chance to handle the request.

## Pattern Overview

### Structure
- **Handler Interface**: Defines the interface for handling requests
- **Concrete Handlers**: Implement the handler interface and decide whether to handle the request
- **Client**: Initiates the request to a concrete handler object on the chain

### Benefits
- **Decoupling**: Sender doesn't need to know which handler will process the request
- **Flexibility**: Easy to add or remove handlers without changing existing code
- **Single Responsibility**: Each handler focuses on one type of processing
- **Dynamic Chain**: Chain can be composed at runtime

### Drawbacks
- **Performance**: Request may traverse the entire chain
- **Debugging**: Hard to observe runtime characteristics
- **No Guarantee**: Request might not be handled if no handler accepts it

---

## Basic Implementation Comparison

### Scenario: Request Processing System
Let's implement a request processing system where different types of requests need to be handled by appropriate handlers.

---

## Go Implementation

### Step 1: Define Request Types and Handler Interface

```go
package main

import (
    "fmt"
    "strings"
)

// RequestType represents different types of requests
type RequestType int

const (
    InfoRequest RequestType = iota
    DebugRequest
    ErrorRequest
)

func (rt RequestType) String() string {
    switch rt {
    case InfoRequest:
        return "INFO"
    case DebugRequest:
        return "DEBUG"
    case ErrorRequest:
        return "ERROR"
    default:
        return "UNKNOWN"
    }
}

// Request represents a request to be processed
type Request struct {
    Type    RequestType
    Message string
    Level   int
}

// Handler defines the interface for request handlers
type Handler interface {
    SetNext(handler Handler) Handler
    Handle(request *Request) bool
}

// BaseHandler provides common functionality for handlers
type BaseHandler struct {
    next Handler
}

// SetNext sets the next handler in the chain
func (h *BaseHandler) SetNext(handler Handler) Handler {
    h.next = handler
    return handler
}

// HandleNext passes the request to the next handler
func (h *BaseHandler) HandleNext(request *Request) bool {
    if h.next != nil {
        return h.next.Handle(request)
    }
    return false
}
```

### Step 2: Implement Concrete Handlers

```go
// InfoHandler handles info-level requests
type InfoHandler struct {
    BaseHandler
}

// NewInfoHandler creates a new info handler
func NewInfoHandler() *InfoHandler {
    return &InfoHandler{}
}

// Handle processes info requests
func (h *InfoHandler) Handle(request *Request) bool {
    if request.Type == InfoRequest {
        fmt.Printf("[INFO] %s\n", request.Message)
        return true
    }
    return h.HandleNext(request)
}

// DebugHandler handles debug-level requests
type DebugHandler struct {
    BaseHandler
}

// NewDebugHandler creates a new debug handler
func NewDebugHandler() *DebugHandler {
    return &DebugHandler{}
}

// Handle processes debug requests
func (h *DebugHandler) Handle(request *Request) bool {
    if request.Type == DebugRequest {
        fmt.Printf("[DEBUG] %s (Level: %d)\n", request.Message, request.Level)
        return true
    }
    return h.HandleNext(request)
}

// ErrorHandler handles error-level requests
type ErrorHandler struct {
    BaseHandler
}

// NewErrorHandler creates a new error handler
func NewErrorHandler() *ErrorHandler {
    return &ErrorHandler{}
}

// Handle processes error requests
func (h *ErrorHandler) Handle(request *Request) bool {
    if request.Type == ErrorRequest {
        fmt.Printf("[ERROR] %s (CRITICAL - Level: %d)\n", 
            strings.ToUpper(request.Message), request.Level)
        return true
    }
    return h.HandleNext(request)
}

// DefaultHandler handles any unprocessed requests
type DefaultHandler struct {
    BaseHandler
}

// NewDefaultHandler creates a new default handler
func NewDefaultHandler() *DefaultHandler {
    return &DefaultHandler{}
}

// Handle processes any remaining requests
func (h *DefaultHandler) Handle(request *Request) bool {
    fmt.Printf("[UNKNOWN] Unhandled request type: %s, Message: %s\n", 
        request.Type.String(), request.Message)
    return true
}
```

### Step 3: Chain Builder and Usage

```go
// ChainBuilder helps build handler chains
type ChainBuilder struct {
    first Handler
    last  Handler
}

// NewChainBuilder creates a new chain builder
func NewChainBuilder() *ChainBuilder {
    return &ChainBuilder{}
}

// Add adds a handler to the chain
func (cb *ChainBuilder) Add(handler Handler) *ChainBuilder {
    if cb.first == nil {
        cb.first = handler
        cb.last = handler
    } else {
        cb.last.SetNext(handler)
        cb.last = handler
    }
    return cb
}

// Build returns the first handler in the chain
func (cb *ChainBuilder) Build() Handler {
    return cb.first
}

// LogProcessor processes multiple requests through the chain
type LogProcessor struct {
    chain Handler
}

// NewLogProcessor creates a new log processor
func NewLogProcessor(chain Handler) *LogProcessor {
    return &LogProcessor{chain: chain}
}

// Process processes a request through the chain
func (lp *LogProcessor) Process(request *Request) {
    fmt.Printf("Processing request: %s\n", request.Type.String())
    handled := lp.chain.Handle(request)
    if !handled {
        fmt.Println("Request was not handled by any handler")
    }
    fmt.Println("---")
}

// ProcessBatch processes multiple requests
func (lp *LogProcessor) ProcessBatch(requests []*Request) {
    fmt.Println("=== Processing Batch ===")
    for _, request := range requests {
        lp.Process(request)
    }
}
```

### Step 4: Complete Go Example

```go
func main() {
    // Build the chain using the builder
    chain := NewChainBuilder().
        Add(NewInfoHandler()).
        Add(NewDebugHandler()).
        Add(NewErrorHandler()).
        Add(NewDefaultHandler()).
        Build()
    
    // Create processor
    processor := NewLogProcessor(chain)
    
    // Create test requests
    requests := []*Request{
        {Type: InfoRequest, Message: "Application started successfully", Level: 1},
        {Type: DebugRequest, Message: "Variable x = 42", Level: 2},
        {Type: ErrorRequest, Message: "Database connection failed", Level: 5},
        {Type: RequestType(99), Message: "Unknown request type", Level: 1},
    }
    
    // Process requests
    processor.ProcessBatch(requests)
    
    // Demonstrate dynamic chain modification
    fmt.Println("\n=== Modified Chain (without Debug Handler) ===")
    modifiedChain := NewChainBuilder().
        Add(NewInfoHandler()).
        Add(NewErrorHandler()).
        Add(NewDefaultHandler()).
        Build()
    
    modifiedProcessor := NewLogProcessor(modifiedChain)
    modifiedProcessor.Process(&Request{
        Type: DebugRequest, 
        Message: "This debug message will be handled by default handler", 
        Level: 2,
    })
}
```

---

## Java Implementation

### Step 1: Define Request Types and Handler Interface

```java
// RequestType enum
public enum RequestType {
    INFO("INFO"),
    DEBUG("DEBUG"),
    ERROR("ERROR");
    
    private final String name;
    
    RequestType(String name) {
        this.name = name;
    }
    
    @Override
    public String toString() {
        return name;
    }
}

// Request class
public class Request {
    private RequestType type;
    private String message;
    private int level;
    
    public Request(RequestType type, String message, int level) {
        this.type = type;
        this.message = message;
        this.level = level;
    }
    
    // Getters
    public RequestType getType() { return type; }
    public String getMessage() { return message; }
    public int getLevel() { return level; }
    
    // Setters
    public void setType(RequestType type) { this.type = type; }
    public void setMessage(String message) { this.message = message; }
    public void setLevel(int level) { this.level = level; }
}

// Handler interface
public interface Handler {
    Handler setNext(Handler handler);
    boolean handle(Request request);
}

// Abstract base handler
public abstract class BaseHandler implements Handler {
    protected Handler next;
    
    @Override
    public Handler setNext(Handler handler) {
        this.next = handler;
        return handler;
    }
    
    protected boolean handleNext(Request request) {
        if (next != null) {
            return next.handle(request);
        }
        return false;
    }
}
```

### Step 2: Implement Concrete Handlers

```java
// InfoHandler
public class InfoHandler extends BaseHandler {
    @Override
    public boolean handle(Request request) {
        if (request.getType() == RequestType.INFO) {
            System.out.printf("[INFO] %s%n", request.getMessage());
            return true;
        }
        return handleNext(request);
    }
}

// DebugHandler
public class DebugHandler extends BaseHandler {
    @Override
    public boolean handle(Request request) {
        if (request.getType() == RequestType.DEBUG) {
            System.out.printf("[DEBUG] %s (Level: %d)%n", 
                request.getMessage(), request.getLevel());
            return true;
        }
        return handleNext(request);
    }
}

// ErrorHandler
public class ErrorHandler extends BaseHandler {
    @Override
    public boolean handle(Request request) {
        if (request.getType() == RequestType.ERROR) {
            System.out.printf("[ERROR] %s (CRITICAL - Level: %d)%n", 
                request.getMessage().toUpperCase(), request.getLevel());
            return true;
        }
        return handleNext(request);
    }
}

// DefaultHandler
public class DefaultHandler extends BaseHandler {
    @Override
    public boolean handle(Request request) {
        System.out.printf("[UNKNOWN] Unhandled request type: %s, Message: %s%n", 
            request.getType().toString(), request.getMessage());
        return true;
    }
}
```

### Step 3: Chain Builder and Processor

```java
import java.util.ArrayList;
import java.util.List;

// ChainBuilder utility class
public class ChainBuilder {
    private Handler first;
    private Handler last;
    
    public ChainBuilder add(Handler handler) {
        if (first == null) {
            first = handler;
            last = handler;
        } else {
            last.setNext(handler);
            last = handler;
        }
        return this;
    }
    
    public Handler build() {
        return first;
    }
}

// LogProcessor
public class LogProcessor {
    private Handler chain;
    
    public LogProcessor(Handler chain) {
        this.chain = chain;
    }
    
    public void process(Request request) {
        System.out.printf("Processing request: %s%n", request.getType().toString());
        boolean handled = chain.handle(request);
        if (!handled) {
            System.out.println("Request was not handled by any handler");
        }
        System.out.println("---");
    }
    
    public void processBatch(List<Request> requests) {
        System.out.println("=== Processing Batch ===");
        for (Request request : requests) {
            process(request);
        }
    }
}
```

### Step 4: Complete Java Example

```java
import java.util.Arrays;
import java.util.List;

public class ChainOfResponsibilityDemo {
    public static void main(String[] args) {
        // Build the chain using the builder
        Handler chain = new ChainBuilder()
            .add(new InfoHandler())
            .add(new DebugHandler())
            .add(new ErrorHandler())
            .add(new DefaultHandler())
            .build();
        
        // Create processor
        LogProcessor processor = new LogProcessor(chain);
        
        // Create test requests
        List<Request> requests = Arrays.asList(
            new Request(RequestType.INFO, "Application started successfully", 1),
            new Request(RequestType.DEBUG, "Variable x = 42", 2),
            new Request(RequestType.ERROR, "Database connection failed", 5)
        );
        
        // Process requests
        processor.processBatch(requests);
        
        // Demonstrate dynamic chain modification
        System.out.println("\n=== Modified Chain (without Debug Handler) ===");
        Handler modifiedChain = new ChainBuilder()
            .add(new InfoHandler())
            .add(new ErrorHandler())
            .add(new DefaultHandler())
            .build();
        
        LogProcessor modifiedProcessor = new LogProcessor(modifiedChain);
        modifiedProcessor.process(new Request(RequestType.DEBUG, 
            "This debug message will be handled by default handler", 2));
    }
}
```

---

## Advanced Examples

### Middleware Chain Pattern (Go)

```go
// HTTPMiddleware represents HTTP middleware
type HTTPMiddleware interface {
    ServeHTTP(next http.HandlerFunc) http.HandlerFunc
}

// AuthMiddleware handles authentication
type AuthMiddleware struct {
    secretKey string
}

func NewAuthMiddleware(secretKey string) *AuthMiddleware {
    return &AuthMiddleware{secretKey: secretKey}
}

func (m *AuthMiddleware) ServeHTTP(next http.HandlerFunc) http.HandlerFunc {
    return func(w http.ResponseWriter, r *http.Request) {
        token := r.Header.Get("Authorization")
        if token == "" || !m.isValidToken(token) {
            http.Error(w, "Unauthorized", http.StatusUnauthorized)
            return
        }
        fmt.Println("Authentication passed")
        next(w, r)
    }
}

func (m *AuthMiddleware) isValidToken(token string) bool {
    return token == "Bearer valid-token"
}

// LoggingMiddleware logs requests
type LoggingMiddleware struct{}

func NewLoggingMiddleware() *LoggingMiddleware {
    return &LoggingMiddleware{}
}

func (m *LoggingMiddleware) ServeHTTP(next http.HandlerFunc) http.HandlerFunc {
    return func(w http.ResponseWriter, r *http.Request) {
        start := time.Now()
        fmt.Printf("Started %s %s\n", r.Method, r.URL.Path)
        
        next(w, r)
        
        fmt.Printf("Completed in %v\n", time.Since(start))
    }
}

// RateLimitMiddleware handles rate limiting
type RateLimitMiddleware struct {
    requests map[string][]time.Time
    limit    int
    window   time.Duration
    mu       sync.RWMutex
}

func NewRateLimitMiddleware(limit int, window time.Duration) *RateLimitMiddleware {
    return &RateLimitMiddleware{
        requests: make(map[string][]time.Time),
        limit:    limit,
        window:   window,
    }
}

func (m *RateLimitMiddleware) ServeHTTP(next http.HandlerFunc) http.HandlerFunc {
    return func(w http.ResponseWriter, r *http.Request) {
        clientIP := r.RemoteAddr
        
        m.mu.Lock()
        now := time.Now()
        
        // Clean old requests
        if times, exists := m.requests[clientIP]; exists {
            var validTimes []time.Time
            for _, t := range times {
                if now.Sub(t) < m.window {
                    validTimes = append(validTimes, t)
                }
            }
            m.requests[clientIP] = validTimes
        }
        
        // Check rate limit
        if len(m.requests[clientIP]) >= m.limit {
            m.mu.Unlock()
            http.Error(w, "Rate limit exceeded", http.StatusTooManyRequests)
            return
        }
        
        // Add current request
        m.requests[clientIP] = append(m.requests[clientIP], now)
        m.mu.Unlock()
        
        fmt.Printf("Rate limit check passed for %s\n", clientIP)
        next(w, r)
    }
}

// MiddlewareChain chains middlewares together
type MiddlewareChain struct {
    middlewares []HTTPMiddleware
}

func NewMiddlewareChain() *MiddlewareChain {
    return &MiddlewareChain{}
}

func (mc *MiddlewareChain) Add(middleware HTTPMiddleware) *MiddlewareChain {
    mc.middlewares = append(mc.middlewares, middleware)
    return mc
}

func (mc *MiddlewareChain) Build(finalHandler http.HandlerFunc) http.HandlerFunc {
    handler := finalHandler
    
    // Apply middlewares in reverse order
    for i := len(mc.middlewares) - 1; i >= 0; i-- {
        handler = mc.middlewares[i].ServeHTTP(handler)
    }
    
    return handler
}
```

### Validation Chain Pattern (Java)

```java
import java.util.*;
import java.util.function.Predicate;

// ValidationRequest
public class ValidationRequest {
    private Map<String, Object> data;
    private List<String> errors;
    
    public ValidationRequest(Map<String, Object> data) {
        this.data = data;
        this.errors = new ArrayList<>();
    }
    
    public Map<String, Object> getData() { return data; }
    public List<String> getErrors() { return errors; }
    public void addError(String error) { errors.add(error); }
    public boolean hasErrors() { return !errors.isEmpty(); }
}

// Validator interface
@FunctionalInterface
public interface Validator {
    boolean validate(ValidationRequest request);
}

// Concrete validators
public class RequiredFieldValidator implements Validator {
    private final String fieldName;
    
    public RequiredFieldValidator(String fieldName) {
        this.fieldName = fieldName;
    }
    
    @Override
    public boolean validate(ValidationRequest request) {
        Object value = request.getData().get(fieldName);
        if (value == null || value.toString().trim().isEmpty()) {
            request.addError(fieldName + " is required");
            return false;
        }
        return true;
    }
}

public class EmailValidator implements Validator {
    private final String fieldName;
    private static final String EMAIL_PATTERN = 
        "^[A-Za-z0-9+_.-]+@[A-Za-z0-9.-]+\\.[A-Za-z]{2,}$";
    
    public EmailValidator(String fieldName) {
        this.fieldName = fieldName;
    }
    
    @Override
    public boolean validate(ValidationRequest request) {
        Object value = request.getData().get(fieldName);
        if (value != null && !value.toString().matches(EMAIL_PATTERN)) {
            request.addError(fieldName + " must be a valid email address");
            return false;
        }
        return true;
    }
}

public class LengthValidator implements Validator {
    private final String fieldName;
    private final int minLength;
    private final int maxLength;
    
    public LengthValidator(String fieldName, int minLength, int maxLength) {
        this.fieldName = fieldName;
        this.minLength = minLength;
        this.maxLength = maxLength;
    }
    
    @Override
    public boolean validate(ValidationRequest request) {
        Object value = request.getData().get(fieldName);
        if (value != null) {
            int length = value.toString().length();
            if (length < minLength || length > maxLength) {
                request.addError(String.format("%s must be between %d and %d characters", 
                    fieldName, minLength, maxLength));
                return false;
            }
        }
        return true;
    }
}

// ValidationChain
public class ValidationChain {
    private final List<Validator> validators;
    
    public ValidationChain() {
        this.validators = new ArrayList<>();
    }
    
    public ValidationChain add(Validator validator) {
        validators.add(validator);
        return this;
    }
    
    public ValidationResult validate(Map<String, Object> data) {
        ValidationRequest request = new ValidationRequest(data);
        
        boolean allValid = true;
        for (Validator validator : validators) {
            if (!validator.validate(request)) {
                allValid = false;
                // Continue to collect all errors
            }
        }
        
        return new ValidationResult(allValid, request.getErrors());
    }
}

// ValidationResult
public class ValidationResult {
    private final boolean valid;
    private final List<String> errors;
    
    public ValidationResult(boolean valid, List<String> errors) {
        this.valid = valid;
        this.errors = new ArrayList<>(errors);
    }
    
    public boolean isValid() { return valid; }
    public List<String> getErrors() { return errors; }
}
```

---

## Real-World Scenarios

### 1. HTTP Request Processing Pipeline

**Go Implementation:**
```go
type HTTPRequest struct {
    Method  string
    Path    string
    Headers map[string]string
    Body    []byte
}

type HTTPResponse struct {
    StatusCode int
    Headers    map[string]string
    Body       []byte
}

type RequestProcessor interface {
    Process(req *HTTPRequest, res *HTTPResponse) bool
}
```

**Java Implementation:**
```java
public class HTTPRequest {
    private String method;
    private String path;
    private Map<String, String> headers;
    private byte[] body;
    
    // constructors, getters, setters
}

public interface RequestProcessor {
    boolean process(HTTPRequest request, HTTPResponse response);
}
```

### 2. Event Processing System

**Go Implementation:**
```go
type Event struct {
    Type      string
    Payload   map[string]interface{}
    Timestamp time.Time
}

type EventHandler interface {
    SetNext(handler EventHandler) EventHandler
    Handle(event *Event) bool
}
```

**Java Implementation:**
```java
public class Event {
    private String type;
    private Map<String, Object> payload;
    private LocalDateTime timestamp;
    
    // constructors, getters, setters
}

public interface EventHandler {
    EventHandler setNext(EventHandler handler);
    boolean handle(Event event);
}
```

---

## Key Differences Between Go and Java Implementations

### 1. **Interface Implementation**
- **Go**: Implicit interface implementation with composition
- **Java**: Explicit interface implementation with inheritance

### 2. **Method Naming Conventions**
- **Go**: Uses camelCase starting with uppercase for public methods
- **Java**: Uses camelCase starting with lowercase for methods

### 3. **Error Handling**
- **Go**: Explicit error handling with multiple return values
- **Java**: Exception-based error handling

### 4. **Memory Management**
- **Go**: Pointers and value types, garbage collected
- **Java**: Reference types only, garbage collected

### 5. **Generics**
- **Go**: Type parameters (Go 1.18+) or interface{} for older versions
- **Java**: Generics with type erasure

### 6. **Concurrency**
- **Go**: Goroutines and channels, built-in sync package
- **Java**: Threads and concurrent utilities, synchronized keyword

---

## Best Practices

### Go Best Practices

1. **Use Interfaces Wisely**
```go
// Good: Small, focused interface
type Handler interface {
    Handle(request Request) bool
}

// Avoid: Large interfaces
type BadHandler interface {
    Handle(request Request) bool
    Validate(request Request) error
    Transform(request Request) Request
    Log(message string)
}
```

2. **Implement Context Support**
```go
type Handler interface {
    Handle(ctx context.Context, request Request) bool
}
```

3. **Use Functional Options**
```go
type ChainOption func(*Chain)

func WithTimeout(timeout time.Duration) ChainOption {
    return func(c *Chain) {
        c.timeout = timeout
    }
}

func NewChain(opts ...ChainOption) *Chain {
    c := &Chain{}
    for _, opt := range opts {
        opt(c)
    }
    return c
}
```

### Java Best Practices

1. **Use Builder Pattern for Complex Chains**
```java
public class ChainBuilder<T> {
    private Handler<T> first;
    private Handler<T> last;
    
    public ChainBuilder<T> add(Handler<T> handler) {
        // implementation
        return this;
    }
    
    public Handler<T> build() {
        return first;
    }
}
```

2. **Implement Proper Exception Handling**
```java
public abstract class BaseHandler<T> implements Handler<T> {
    protected Handler<T> next;
    
    @Override
    public boolean handle(T request) {
        try {
            return doHandle(request);
        } catch (Exception e) {
            handleException(e, request);
            return handleNext(request);
        }
    }
    
    protected abstract boolean doHandle(T request) throws Exception;
    protected abstract void handleException(Exception e, T request);
}
```

3. **Use Generics for Type Safety**
```java
public interface Handler<T> {
    Handler<T> setNext(Handler<T> handler);
    boolean handle(T request);
}
```

---

## When to Use Chain of Responsibility

### Use When:
1. **Multiple Processing Options**: Multiple objects can handle a request, but you don't know which one until runtime
2. **Dynamic Configuration**: You want to specify handlers and their order dynamically
3. **Decoupling**: You want to decouple request senders from receivers
4. **Pipeline Processing**: Processing involves multiple steps that can be chained
5. **Middleware Systems**: Building middleware or filter systems

### Examples:
- **Web Middleware**: Authentication, logging, compression, etc.
- **Event Processing**: Different handlers for different event types
- **Validation**: Multiple validation rules applied in sequence
- **Help Systems**: Context-sensitive help with fallback options
- **Approval Workflows**: Different approval levels based on amount/type

### Don't Use When:
1. **Single Handler**: Only one object can handle the request
2. **Performance Critical**: The chain traversal overhead is unacceptable
3. **Simple Logic**: The processing logic is simple and doesn't benefit from chaining
4. **Guaranteed Processing**: Every request must be handled (no fallback acceptable)

---

## Output Examples

### Go Output:
```
=== Processing Batch ===
Processing request: INFO
[INFO] Application started successfully
---
Processing request: DEBUG
[DEBUG] Variable x = 42 (Level: 2)
---
Processing request: ERROR
[ERROR] DATABASE CONNECTION FAILED (CRITICAL - Level: 5)
---
Processing request: UNKNOWN
[UNKNOWN] Unhandled request type: UNKNOWN, Message: Unknown request type
---

=== Modified Chain (without Debug Handler) ===
Processing request: DEBUG
[UNKNOWN] Unhandled request type: DEBUG, Message: This debug message will be handled by default handler
---
```

### Java Output:
```
=== Processing Batch ===
Processing request: INFO
[INFO] Application started successfully
---
Processing request: DEBUG
[DEBUG] Variable x = 42 (Level: 2)
---
Processing request: ERROR
[ERROR] DATABASE CONNECTION FAILED (CRITICAL - Level: 5)
---

=== Modified Chain (without Debug Handler) ===
Processing request: DEBUG
[UNKNOWN] Unhandled request type: DEBUG, Message: This debug message will be handled by default handler
---
```

---

## Summary

The Chain of Responsibility pattern provides a flexible way to handle requests by passing them through a chain of potential handlers. Both Go and Java implementations offer their own advantages:

- **Go** emphasizes simplicity and composition with interfaces
- **Java** provides strong typing and inheritance-based structure

The pattern is particularly useful for building middleware systems, validation pipelines, and any scenario where multiple handlers might process a request. The key is to keep handlers focused on single responsibilities and make the chain easily configurable.