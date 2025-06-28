# SOLID Principles - Go, Java & Python Comparison

## S - Single Responsibility Principle (SRP)
**Definition**: A class should have only one reason to change, meaning it should have only one job or responsibility.

### Bad Example - Violates SRP

<details>
<summary><b>Go</b></summary>

```go
// Violates SRP - User handles data, persistence, and email
type User struct {
    ID    int
    Name  string
    Email string
}

func (u *User) GetName() string {
    return u.Name
}

func (u *User) SaveToDatabase() error {
    // Database logic - WRONG! Reason to change #1
    return nil
}

func (u *User) SendEmail() error {
    // Email logic - WRONG! Reason to change #2
    return nil
}
```
</details>

<details>
<summary><b>Java</b></summary>

```java
// Violates SRP - User handles data, persistence, and email
public class User {
    private int id;
    private String name;
    private String email;
    
    public String getName() {
        return name;
    }
    
    public void saveToDatabase() throws Exception {
        // Database logic - WRONG! Reason to change #1
    }
    
    public void sendEmail() throws Exception {
        // Email logic - WRONG! Reason to change #2
    }
}
```
</details>

<details>
<summary><b>Python</b></summary>

```python
# Violates SRP - User handles data, persistence, and email
class User:
    def __init__(self, id, name, email):
        self.id = id
        self.name = name
        self.email = email
    
    def get_name(self):
        return self.name
    
    def save_to_database(self):
        # Database logic - WRONG! Reason to change #1
        pass
    
    def send_email(self):
        # Email logic - WRONG! Reason to change #2
        pass
```
</details>

### Good Example - Follows SRP

<details>
<summary><b>Go</b></summary>

```go
// Follows SRP - Separated responsibilities
type User struct {
    ID    int
    Name  string
    Email string
}

func (u *User) GetName() string {
    return u.Name
}

type UserRepository struct {
    db Database
}

func (ur *UserRepository) Save(user *User) error {
    return nil
}

type EmailService struct {
    client EmailClient
}

func (es *EmailService) SendWelcomeEmail(user *User) error {
    return nil
}
```
</details>

<details>
<summary><b>Java</b></summary>

```java
// Follows SRP - Separated responsibilities
public class User {
    private int id;
    private String name;
    private String email;
    
    public String getName() {
        return name;
    }
}

public class UserRepository {
    private Database db;
    
    public void save(User user) throws Exception {
        // Database logic here
    }
}

public class EmailService {
    private EmailClient client;
    
    public void sendWelcomeEmail(User user) throws Exception {
        // Email logic here
    }
}
```
</details>

<details>
<summary><b>Python</b></summary>

```python
# Follows SRP - Separated responsibilities
class User:
    def __init__(self, id, name, email):
        self.id = id
        self.name = name
        self.email = email
    
    def get_name(self):
        return self.name

class UserRepository:
    def __init__(self, db):
        self.db = db
    
    def save(self, user):
        # Database logic here
        pass

class EmailService:
    def __init__(self, client):
        self.client = client
    
    def send_welcome_email(self, user):
        # Email logic here
        pass
```
</details>

---

## O - Open/Closed Principle (OCP)
**Definition**: Software entities should be open for extension but closed for modification.

### Bad Example - Violates OCP

<details>
<summary><b>Go</b></summary>

```go
type Rectangle struct {
    Width, Height float64
}

type Circle struct {
    Radius float64
}

// Violates OCP - must modify for new shapes
func CalculateArea(shapes []interface{}) float64 {
    var totalArea float64
    for _, shape := range shapes {
        switch s := shape.(type) {
        case Rectangle:
            totalArea += s.Width * s.Height
        case Circle:
            totalArea += math.Pi * s.Radius * s.Radius
        // Must add new case for every shape - VIOLATION!
        }
    }
    return totalArea
}
```
</details>

<details>
<summary><b>Java</b></summary>

```java
class Rectangle {
    private double width, height;
    // constructors, getters...
}

class Circle {
    private double radius;
    // constructors, getters...
}

// Violates OCP - must modify for new shapes
public class AreaCalculator {
    public double calculateArea(List<Object> shapes) {
        double totalArea = 0;
        for (Object shape : shapes) {
            if (shape instanceof Rectangle) {
                Rectangle r = (Rectangle) shape;
                totalArea += r.getWidth() * r.getHeight();
            } else if (shape instanceof Circle) {
                Circle c = (Circle) shape;
                totalArea += Math.PI * c.getRadius() * c.getRadius();
            }
            // Must add new if-else for every shape - VIOLATION!
        }
        return totalArea;
    }
}
```
</details>

<details>
<summary><b>Python</b></summary>

```python
class Rectangle:
    def __init__(self, width, height):
        self.width = width
        self.height = height

class Circle:
    def __init__(self, radius):
        self.radius = radius

# Violates OCP - must modify for new shapes
def calculate_area(shapes):
    total_area = 0
    for shape in shapes:
        if isinstance(shape, Rectangle):
            total_area += shape.width * shape.height
        elif isinstance(shape, Circle):
            total_area += 3.14159 * shape.radius * shape.radius
        # Must add new elif for every shape - VIOLATION!
    return total_area
```
</details>

### Good Example - Follows OCP

<details>
<summary><b>Go</b></summary>

```go
// Follows OCP - extensible without modification
type Shape interface {
    Area() float64
}

type Rectangle struct {
    Width, Height float64
}

func (r Rectangle) Area() float64 {
    return r.Width * r.Height
}

type Circle struct {
    Radius float64
}

func (c Circle) Area() float64 {
    return math.Pi * c.Radius * c.Radius
}

// Never needs modification
func CalculateArea(shapes []Shape) float64 {
    var totalArea float64
    for _, shape := range shapes {
        totalArea += shape.Area()
    }
    return totalArea
}

// Adding new shape - no modification needed
type Triangle struct {
    Base, Height float64
}

func (t Triangle) Area() float64 {
    return 0.5 * t.Base * t.Height
}
```
</details>

<details>
<summary><b>Java</b></summary>

```java
// Follows OCP - extensible without modification
interface Shape {
    double area();
}

class Rectangle implements Shape {
    private double width, height;
    
    public Rectangle(double width, double height) {
        this.width = width;
        this.height = height;
    }
    
    @Override
    public double area() {
        return width * height;
    }
}

class Circle implements Shape {
    private double radius;
    
    public Circle(double radius) {
        this.radius = radius;
    }
    
    @Override
    public double area() {
        return Math.PI * radius * radius;
    }
}

// Never needs modification
public class AreaCalculator {
    public double calculateArea(List<Shape> shapes) {
        double totalArea = 0;
        for (Shape shape : shapes) {
            totalArea += shape.area();
        }
        return totalArea;
    }
}

// Adding new shape - no modification needed
class Triangle implements Shape {
    private double base, height;
    
    public Triangle(double base, double height) {
        this.base = base;
        this.height = height;
    }
    
    @Override
    public double area() {
        return 0.5 * base * height;
    }
}
```
</details>

<details>
<summary><b>Python</b></summary>

```python
from abc import ABC, abstractmethod

# Follows OCP - extensible without modification
class Shape(ABC):
    @abstractmethod
    def area(self):
        pass

class Rectangle(Shape):
    def __init__(self, width, height):
        self.width = width
        self.height = height
    
    def area(self):
        return self.width * self.height

class Circle(Shape):
    def __init__(self, radius):
        self.radius = radius
    
    def area(self):
        return 3.14159 * self.radius * self.radius

# Never needs modification
def calculate_area(shapes):
    total_area = 0
    for shape in shapes:
        total_area += shape.area()
    return total_area

# Adding new shape - no modification needed
class Triangle(Shape):
    def __init__(self, base, height):
        self.base = base
        self.height = height
    
    def area(self):
        return 0.5 * self.base * self.height
```
</details>

---

## L - Liskov Substitution Principle (LSP)
**Definition**: Objects of a superclass should be replaceable with objects of a subclass without breaking the application.

### Bad Example - Violates LSP

<details>
<summary><b>Go</b></summary>

```go
// Violates LSP - Square changes Rectangle behavior
type Rectangle struct {
    width, height float64
}

func (r *Rectangle) SetWidth(w float64) {
    r.width = w
}

func (r *Rectangle) SetHeight(h float64) {
    r.height = h
}

func (r *Rectangle) Area() float64 {
    return r.width * r.height
}

type Square struct {
    Rectangle
}

// Violates LSP - unexpected behavior
func (s *Square) SetWidth(w float64) {
    s.width = w
    s.height = w // Side effect!
}

func (s *Square) SetHeight(h float64) {
    s.width = h  // Side effect!
    s.height = h
}

// This function expects Rectangle behavior
func ProcessRectangle(r *Rectangle) {
    r.SetWidth(5)
    r.SetHeight(4)
    // Expects 20, but Square gives 16!
    fmt.Printf("Area: %f", r.Area())
}
```
</details>

<details>
<summary><b>Java</b></summary>

```java
// Violates LSP - Square changes Rectangle behavior
class Rectangle {
    protected double width, height;
    
    public void setWidth(double width) {
        this.width = width;
    }
    
    public void setHeight(double height) {
        this.height = height;
    }
    
    public double area() {
        return width * height;
    }
}

class Square extends Rectangle {
    // Violates LSP - unexpected behavior
    @Override
    public void setWidth(double width) {
        this.width = width;
        this.height = width; // Side effect!
    }
    
    @Override
    public void setHeight(double height) {
        this.width = height;  // Side effect!
        this.height = height;
    }
}

// This function expects Rectangle behavior
public void processRectangle(Rectangle r) {
    r.setWidth(5);
    r.setHeight(4);
    // Expects 20, but Square gives 16!
    System.out.println("Area: " + r.area());
}
```
</details>

<details>
<summary><b>Python</b></summary>

```python
# Violates LSP - Square changes Rectangle behavior
class Rectangle:
    def __init__(self):
        self._width = 0
        self._height = 0
    
    def set_width(self, width):
        self._width = width
    
    def set_height(self, height):
        self._height = height
    
    def area(self):
        return self._width * self._height

class Square(Rectangle):
    # Violates LSP - unexpected behavior
    def set_width(self, width):
        self._width = width
        self._height = width  # Side effect!
    
    def set_height(self, height):
        self._width = height   # Side effect!
        self._height = height

# This function expects Rectangle behavior
def process_rectangle(r):
    r.set_width(5)
    r.set_height(4)
    # Expects 20, but Square gives 16!
    print(f"Area: {r.area()}")
```
</details>

### Good Example - Follows LSP

<details>
<summary><b>Go</b></summary>

```go
// Follows LSP - proper abstraction
type Shape interface {
    Area() float64
}

type Rectangle struct {
    width, height float64
}

func NewRectangle(w, h float64) *Rectangle {
    return &Rectangle{width: w, height: h}
}

func (r *Rectangle) Area() float64 {
    return r.width * r.height
}

type Square struct {
    side float64
}

func NewSquare(side float64) *Square {
    return &Square{side: side}
}

func (s *Square) Area() float64 {
    return s.side * s.side
}

// Both can be used interchangeably
func ProcessShape(s Shape) {
    fmt.Printf("Area: %f", s.Area())
}
```
</details>

<details>
<summary><b>Java</b></summary>

```java
// Follows LSP - proper abstraction
interface Shape {
    double area();
}

class Rectangle implements Shape {
    private double width, height;
    
    public Rectangle(double width, double height) {
        this.width = width;
        this.height = height;
    }
    
    @Override
    public double area() {
        return width * height;
    }
}

class Square implements Shape {
    private double side;
    
    public Square(double side) {
        this.side = side;
    }
    
    @Override
    public double area() {
        return side * side;
    }
}

// Both can be used interchangeably
public void processShape(Shape s) {
    System.out.println("Area: " + s.area());
}
```
</details>

<details>
<summary><b>Python</b></summary>

```python
from abc import ABC, abstractmethod

# Follows LSP - proper abstraction
class Shape(ABC):
    @abstractmethod
    def area(self):
        pass

class Rectangle(Shape):
    def __init__(self, width, height):
        self.width = width
        self.height = height
    
    def area(self):
        return self.width * self.height

class Square(Shape):
    def __init__(self, side):
        self.side = side
    
    def area(self):
        return self.side * self.side

# Both can be used interchangeably
def process_shape(s):
    print(f"Area: {s.area()}")
```
</details>

---

## I - Interface Segregation Principle (ISP)
**Definition**: Clients should not be forced to depend on interfaces they don't use.

### Bad Example - Violates ISP

<details>
<summary><b>Go</b></summary>

```go
// Fat interface - violates ISP
type Worker interface {
    Work()
    Eat()
    Sleep()
}

type HumanWorker struct {
    name string
}

func (h *HumanWorker) Work() {
    fmt.Println("Human working")
}

func (h *HumanWorker) Eat() {
    fmt.Println("Human eating")
}

func (h *HumanWorker) Sleep() {
    fmt.Println("Human sleeping")
}

// Forced to implement unnecessary methods
type RobotWorker struct {
    model string
}

func (r *RobotWorker) Work() {
    fmt.Println("Robot working")
}

func (r *RobotWorker) Eat() {
    panic("Robots don't eat!")
}

func (r *RobotWorker) Sleep() {
    panic("Robots don't sleep!")
}
```
</details>

<details>
<summary><b>Java</b></summary>

```java
// Fat interface - violates ISP
interface Worker {
    void work();
    void eat();
    void sleep();
}

class HumanWorker implements Worker {
    @Override
    public void work() {
        System.out.println("Human working");
    }
    
    @Override
    public void eat() {
        System.out.println("Human eating");
    }
    
    @Override
    public void sleep() {
        System.out.println("Human sleeping");
    }
}

// Forced to implement unnecessary methods
class RobotWorker implements Worker {
    @Override
    public void work() {
        System.out.println("Robot working");
    }
    
    @Override
    public void eat() {
        throw new UnsupportedOperationException("Robots don't eat!");
    }
    
    @Override
    public void sleep() {
        throw new UnsupportedOperationException("Robots don't sleep!");
    }
}
```
</details>

<details>
<summary><b>Python</b></summary>

```python
from abc import ABC, abstractmethod

# Fat interface - violates ISP
class Worker(ABC):
    @abstractmethod
    def work(self):
        pass
    
    @abstractmethod
    def eat(self):
        pass
    
    @abstractmethod
    def sleep(self):
        pass

class HumanWorker(Worker):
    def work(self):
        print("Human working")
    
    def eat(self):
        print("Human eating")
    
    def sleep(self):
        print("Human sleeping")

# Forced to implement unnecessary methods
class RobotWorker(Worker):
    def work(self):
        print("Robot working")
    
    def eat(self):
        raise NotImplementedError("Robots don't eat!")
    
    def sleep(self):
        raise NotImplementedError("Robots don't sleep!")
```
</details>

### Good Example - Follows ISP

<details>
<summary><b>Go</b></summary>

```go
// Segregated interfaces - follows ISP
type Workable interface {
    Work()
}

type Eater interface {
    Eat()
}

type Sleeper interface {
    Sleep()
}

type HumanWorker struct {
    name string
}

func (h *HumanWorker) Work() {
    fmt.Println("Human working")
}

func (h *HumanWorker) Eat() {
    fmt.Println("Human eating")
}

func (h *HumanWorker) Sleep() {
    fmt.Println("Human sleeping")
}

// Only implements what it needs
type RobotWorker struct {
    model string
}

func (r *RobotWorker) Work() {
    fmt.Println("Robot working")
}

// Usage
func ManageWorker(w Workable) {
    w.Work()
}

func FeedWorker(e Eater) {
    e.Eat()
}
```
</details>

<details>
<summary><b>Java</b></summary>

```java
// Segregated interfaces - follows ISP
interface Workable {
    void work();
}

interface Eater {
    void eat();
}

interface Sleeper {
    void sleep();
}

class HumanWorker implements Workable, Eater, Sleeper {
    @Override
    public void work() {
        System.out.println("Human working");
    }
    
    @Override
    public void eat() {
        System.out.println("Human eating");
    }
    
    @Override
    public void sleep() {
        System.out.println("Human sleeping");
    }
}

// Only implements what it needs
class RobotWorker implements Workable {
    @Override
    public void work() {
        System.out.println("Robot working");
    }
}

// Usage
public void manageWorker(Workable w) {
    w.work();
}

public void feedWorker(Eater e) {
    e.eat();
}
```
</details>

<details>
<summary><b>Python</b></summary>

```python
from abc import ABC, abstractmethod

# Segregated interfaces - follows ISP
class Workable(ABC):
    @abstractmethod
    def work(self):
        pass

class Eater(ABC):
    @abstractmethod
    def eat(self):
        pass

class Sleeper(ABC):
    @abstractmethod
    def sleep(self):
        pass

class HumanWorker(Workable, Eater, Sleeper):
    def work(self):
        print("Human working")
    
    def eat(self):
        print("Human eating")
    
    def sleep(self):
        print("Human sleeping")

# Only implements what it needs
class RobotWorker(Workable):
    def work(self):
        print("Robot working")

# Usage
def manage_worker(w: Workable):
    w.work()

def feed_worker(e: Eater):
    e.eat()
```
</details>

---

## D - Dependency Inversion Principle (DIP)
**Definition**: High-level modules should not depend on low-level modules. Both should depend on abstractions.

### Bad Example - Violates DIP

<details>
<summary><b>Go</b></summary>

```go
// Violates DIP - direct dependency on concrete class
type MySQLDatabase struct{}

func (db *MySQLDatabase) Save(data string) error {
    fmt.Println("Saving to MySQL:", data)
    return nil
}

// High-level module depends on low-level module
type OrderService struct {
    database *MySQLDatabase // Direct dependency!
}

func NewOrderService() *OrderService {
    return &OrderService{
        database: &MySQLDatabase{}, // Hard-coded!
    }
}

func (os *OrderService) CreateOrder(orderData string) error {
    return os.database.Save(orderData)
}
```
</details>

<details>
<summary><b>Java</b></summary>

```java
// Violates DIP - direct dependency on concrete class
class MySQLDatabase {
    public void save(String data) {
        System.out.println("Saving to MySQL: " + data);
    }
}

// High-level module depends on low-level module
class OrderService {
    private MySQLDatabase database; // Direct dependency!
    
    public OrderService() {
        this.database = new MySQLDatabase(); // Hard-coded!
    }
    
    public void createOrder(String orderData) {
        database.save(orderData);
    }
}
```
</details>

<details>
<summary><b>Python</b></summary>

```python
# Violates DIP - direct dependency on concrete class
class MySQLDatabase:
    def save(self, data):
        print(f"Saving to MySQL: {data}")

# High-level module depends on low-level module
class OrderService:
    def __init__(self):
        self.database = MySQLDatabase()  # Direct dependency!
    
    def create_order(self, order_data):
        self.database.save(order_data)
```
</details>

### Good Example - Follows DIP

<details>
<summary><b>Go</b></summary>

```go
// Follows DIP - depend on abstraction
type Database interface {
    Save(data string) error
}

type MySQLDatabase struct{}

func (db *MySQLDatabase) Save(data string) error {
    fmt.Println("Saving to MySQL:", data)
    return nil
}

type PostgreSQLDatabase struct{}

func (db *PostgreSQLDatabase) Save(data string) error {
    fmt.Println("Saving to PostgreSQL:", data)
    return nil
}

// High-level module depends on abstraction
type OrderService struct {
    database Database // Interface dependency!
}

func NewOrderService(db Database) *OrderService {
    return &OrderService{
        database: db, // Injected dependency!
    }
}

func (os *OrderService) CreateOrder(orderData string) error {
    return os.database.Save(orderData)
}

// Usage
func main() {
    mysqlDB := &MySQLDatabase{}
    orderService := NewOrderService(mysqlDB)
    
    postgresDB := &PostgreSQLDatabase{}
    orderService2 := NewOrderService(postgresDB)
}
```
</details>

<details>
<summary><b>Java</b></summary>

```java
// Follows DIP - depend on abstraction
interface Database {
    void save(String data);
}

class MySQLDatabase implements Database {
    @Override
    public void save(String data) {
        System.out.println("Saving to MySQL: " + data);
    }
}

class PostgreSQLDatabase implements Database {
    @Override
    public void save(String data) {
        System.out.println("Saving to PostgreSQL: " + data);
    }
}

// High-level module depends on abstraction
class OrderService {
    private Database database; // Interface dependency!
    
    public OrderService(Database database) {
        this.database = database; // Injected dependency!
    }
    
    public void createOrder(String orderData) {
        database.save(orderData);
    }
}

// Usage
public class Main {
    public static void main(String[] args) {
        Database mysqlDB = new MySQLDatabase();
        OrderService orderService = new OrderService(mysqlDB);
        
        Database postgresDB = new PostgreSQLDatabase();
        OrderService orderService2 = new OrderService(postgresDB);
    }
}
```
</details>

<details>
<summary><b>Python</b></summary>

```python
from abc import ABC, abstractmethod

# Follows DIP - depend on abstraction
class Database(ABC):
    @abstractmethod
    def save(self, data):
        pass

class MySQLDatabase(Database):
    def save(self, data):
        print(f"Saving to MySQL: {data}")

class PostgreSQLDatabase(Database):
    def save(self, data):
        print(f"Saving to PostgreSQL: {data}")

# High-level module depends on abstraction
class OrderService:
    def __init__(self, database: Database):
        self.database = database  # Injected dependency!
    
    def create_order(self, order_data):
        self.database.save(order_data)

# Usage
if __name__ == "__main__":
    mysql_db = MySQLDatabase()
    order_service = OrderService(mysql_db)
    
    postgres_db = PostgreSQLDatabase()
    order_service2 = OrderService(postgres_db)
```
</details>

---

## Language Comparison Summary

### **Go Advantages**
- **Implicit interfaces**: No need to explicitly declare interface implementation
- **Composition over inheritance**: Promotes better design
- **Simple syntax**: Easy to read and understand
- **Built-in dependency injection**: Natural fit for DI patterns

### **Java Advantages**
- **Explicit contracts**: Clear interface implementations
- **Rich ecosystem**: Extensive frameworks for DI (Spring, Guice)
- **Strong typing**: Compile-time error detection
- **Familiar to most interviewers**: Widely understood

### **Python Advantages**
- **Duck typing**: Flexible interface implementation
- **Clean syntax**: Less boilerplate code
- **ABC module**: Good support for abstract base classes
- **Dynamic nature**: Easy to mock and test

### **Interview Recommendations**

1. **Use Java if**: You're comfortable with it and the interviewer expects traditional OOP
2. **Use Go if**: You want to show modern design thinking and are confident explaining Go's approach
3. **Use Python if**: The role is Python-heavy and you can demonstrate good OOP practices

### **Key Takeaways**
- **Principles are language-agnostic**: Focus on concepts, not syntax
- **Explain your choices**: Why you chose certain design decisions
- **Show trade-offs**: Discuss pros/cons of different approaches
- **Be consistent**: Stick to one language throughout the interview unless asked to compare

All three languages can effectively demonstrate SOLID principles, but each has its own idioms and strengths. Choose the one you're most comfortable with and can explain confidently.