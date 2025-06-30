# Design Patterns Guide: Composite Pattern

## Table of Contents
1. [Overview](#overview)
2. [Composite Pattern Fundamentals](#composite-pattern-fundamentals)
3. [Implementation in Java](#implementation-in-java)
4. [Implementation in Go](#implementation-in-go)
5. [Side-by-Side Comparison](#side-by-side-comparison)
6. [Use Cases and Examples](#use-cases-and-examples)
7. [Best Practices](#best-practices)
8. [Pros and Cons](#pros-and-cons)

## Overview

The **Composite Pattern** is a structural design pattern that allows you to compose objects into tree structures to represent part-whole hierarchies. It lets clients treat individual objects and compositions of objects uniformly.

### Key Components:
- **Component**: Declares the interface for objects in the composition
- **Leaf**: Represents leaf objects in the composition (has no children)
- **Composite**: Defines behavior for components having children and stores child components

### Real-world Analogy:
Think of a file system where files and folders are treated uniformly. A folder can contain files or other folders, but both can be moved, copied, or deleted using the same operations.

## Composite Pattern Fundamentals

### Problem It Solves:
- You need to represent part-whole hierarchies of objects
- You want clients to ignore the difference between compositions of objects and individual objects
- You want to treat primitive and composite objects uniformly

### Structure:
```
Component (interface/abstract class)
├── Leaf (implements Component)
└── Composite (implements Component)
    ├── children: List<Component>
    ├── add(Component)
    ├── remove(Component)
    └── getChild(int)
```

## Implementation in Java

### Java Implementation - File System Example

```java
// Component interface
interface FileSystemComponent {
    void showDetails();
    int getSize();
    String getName();
}

// Leaf class - File
class File implements FileSystemComponent {
    private String name;
    private int size;
    
    public File(String name, int size) {
        this.name = name;
        this.size = size;
    }
    
    @Override
    public void showDetails() {
        System.out.println("File: " + name + " (Size: " + size + " KB)");
    }
    
    @Override
    public int getSize() {
        return size;
    }
    
    @Override
    public String getName() {
        return name;
    }
}

// Composite class - Directory
class Directory implements FileSystemComponent {
    private String name;
    private List<FileSystemComponent> components;
    
    public Directory(String name) {
        this.name = name;
        this.components = new ArrayList<>();
    }
    
    public void addComponent(FileSystemComponent component) {
        components.add(component);
    }
    
    public void removeComponent(FileSystemComponent component) {
        components.remove(component);
    }
    
    @Override
    public void showDetails() {
        System.out.println("Directory: " + name);
        for (FileSystemComponent component : components) {
            System.out.print("  ");
            component.showDetails();
        }
    }
    
    @Override
    public int getSize() {
        int totalSize = 0;
        for (FileSystemComponent component : components) {
            totalSize += component.getSize();
        }
        return totalSize;
    }
    
    @Override
    public String getName() {
        return name;
    }
}

// Client code
public class CompositePatternDemo {
    public static void main(String[] args) {
        // Create files
        File file1 = new File("document.txt", 100);
        File file2 = new File("image.jpg", 2000);
        File file3 = new File("config.xml", 50);
        
        // Create directories
        Directory rootDir = new Directory("root");
        Directory documentsDir = new Directory("documents");
        Directory imagesDir = new Directory("images");
        
        // Build the structure
        documentsDir.addComponent(file1);
        documentsDir.addComponent(file3);
        imagesDir.addComponent(file2);
        
        rootDir.addComponent(documentsDir);
        rootDir.addComponent(imagesDir);
        
        // Display structure and calculate size
        rootDir.showDetails();
        System.out.println("Total size: " + rootDir.getSize() + " KB");
    }
}
```

### Java Output:
```
Directory: root
  Directory: documents
    File: document.txt (Size: 100 KB)
    File: config.xml (Size: 50 KB)
  Directory: images
    File: image.jpg (Size: 2000 KB)
Total size: 2150 KB
```

## Implementation in Go

### Go Implementation - File System Example

```go
package main

import (
    "fmt"
    "strings"
)

// Component interface
type FileSystemComponent interface {
    ShowDetails(indent string)
    GetSize() int
    GetName() string
}

// Leaf struct - File
type File struct {
    name string
    size int
}

func NewFile(name string, size int) *File {
    return &File{
        name: name,
        size: size,
    }
}

func (f *File) ShowDetails(indent string) {
    fmt.Printf("%sFile: %s (Size: %d KB)\n", indent, f.name, f.size)
}

func (f *File) GetSize() int {
    return f.size
}

func (f *File) GetName() string {
    return f.name
}

// Composite struct - Directory
type Directory struct {
    name       string
    components []FileSystemComponent
}

func NewDirectory(name string) *Directory {
    return &Directory{
        name:       name,
        components: make([]FileSystemComponent, 0),
    }
}

func (d *Directory) AddComponent(component FileSystemComponent) {
    d.components = append(d.components, component)
}

func (d *Directory) RemoveComponent(component FileSystemComponent) {
    for i, comp := range d.components {
        if comp == component {
            d.components = append(d.components[:i], d.components[i+1:]...)
            break
        }
    }
}

func (d *Directory) ShowDetails(indent string) {
    fmt.Printf("%sDirectory: %s\n", indent, d.name)
    for _, component := range d.components {
        component.ShowDetails(indent + "  ")
    }
}

func (d *Directory) GetSize() int {
    totalSize := 0
    for _, component := range d.components {
        totalSize += component.GetSize()
    }
    return totalSize
}

func (d *Directory) GetName() string {
    return d.name
}

// Client code
func main() {
    // Create files
    file1 := NewFile("document.txt", 100)
    file2 := NewFile("image.jpg", 2000)
    file3 := NewFile("config.xml", 50)
    
    // Create directories
    rootDir := NewDirectory("root")
    documentsDir := NewDirectory("documents")
    imagesDir := NewDirectory("images")
    
    // Build the structure
    documentsDir.AddComponent(file1)
    documentsDir.AddComponent(file3)
    imagesDir.AddComponent(file2)
    
    rootDir.AddComponent(documentsDir)
    rootDir.AddComponent(imagesDir)
    
    // Display structure and calculate size
    rootDir.ShowDetails("")
    fmt.Printf("Total size: %d KB\n", rootDir.GetSize())
}
```

### Go Output:
```
Directory: root
  Directory: documents
    File: document.txt (Size: 100 KB)
    File: config.xml (Size: 50 KB)
  Directory: images
    File: image.jpg (Size: 2000 KB)
Total size: 2150 KB
```

## Side-by-Side Comparison

### 1. Interface Definition

**Java:**
```java
interface FileSystemComponent {
    void showDetails();
    int getSize();
    String getName();
}
```

**Go:**
```go
type FileSystemComponent interface {
    ShowDetails(indent string)
    GetSize() int
    GetName() string
}
```

**Key Differences:**
- Java uses `interface` keyword, Go uses `type` with `interface`
- Go uses PascalCase for public methods (exported)
- Go passes parameters explicitly (indent string) while Java manages state internally

### 2. Leaf Implementation

**Java (File class):**
```java
class File implements FileSystemComponent {
    private String name;
    private int size;
    
    public File(String name, int size) {
        this.name = name;
        this.size = size;
    }
    
    @Override
    public void showDetails() {
        System.out.println("File: " + name + " (Size: " + size + " KB)");
    }
    // ... other methods
}
```

**Go (File struct):**
```go
type File struct {
    name string
    size int
}

func NewFile(name string, size int) *File {
    return &File{
        name: name,
        size: size,
    }
}

func (f *File) ShowDetails(indent string) {
    fmt.Printf("%sFile: %s (Size: %d KB)\n", indent, f.name, f.size)
}
```

**Key Differences:**
- Java uses classes with constructors, Go uses structs with constructor functions
- Java has explicit `implements` keyword, Go implements interfaces implicitly
- Go uses receiver methods `(f *File)`, Java uses `this` implicitly
- Java uses access modifiers (`private`, `public`), Go uses case sensitivity

### 3. Composite Implementation

**Java (Directory class):**
```java
class Directory implements FileSystemComponent {
    private String name;
    private List<FileSystemComponent> components;
    
    public Directory(String name) {
        this.name = name;
        this.components = new ArrayList<>();
    }
    
    public void addComponent(FileSystemComponent component) {
        components.add(component);
    }
}
```

**Go (Directory struct):**
```go
type Directory struct {
    name       string
    components []FileSystemComponent
}

func NewDirectory(name string) *Directory {
    return &Directory{
        name:       name,
        components: make([]FileSystemComponent, 0),
    }
}

func (d *Directory) AddComponent(component FileSystemComponent) {
    d.components = append(d.components, component)
}
```

**Key Differences:**
- Java uses `ArrayList<T>`, Go uses slices `[]T`
- Java's `add()` method vs Go's `append()` function
- Java uses camelCase, Go uses PascalCase for public methods

### 4. Memory Management

**Java:**
- Automatic garbage collection
- Objects allocated on heap
- Reference-based

**Go:**
- Automatic garbage collection
- Stack vs heap allocation optimization
- Can use pointers or values

### 5. Error Handling

**Java:**
```java
public void removeComponent(FileSystemComponent component) {
    components.remove(component); // May throw exception
}
```

**Go:**
```go
func (d *Directory) RemoveComponent(component FileSystemComponent) {
    for i, comp := range d.components {
        if comp == component {
            d.components = append(d.components[:i], d.components[i+1:]...)
            break
        }
    }
    // No exception thrown, silent failure
}
```

## Use Cases and Examples

### 1. GUI Components
**Use Case:** Building a UI where containers (panels, windows) can hold other components (buttons, text fields, or other containers).

### 2. Organization Hierarchy
**Use Case:** Representing company structure where departments contain employees or sub-departments.

### 3. Menu Systems
**Use Case:** Creating nested menu structures where menu items can be individual commands or submenus.

### 4. Expression Trees
**Use Case:** Mathematical expressions where operands can be numbers or other expressions.

## Best Practices

### Java Best Practices:
1. **Use abstract classes** when you need default implementations
2. **Implement proper equals/hashCode** for component comparison
3. **Consider using generics** for type safety
4. **Handle null checks** in composite operations
5. **Use ArrayList for better performance** than LinkedList for most cases

### Go Best Practices:
1. **Use pointers for structs** to avoid copying
2. **Initialize slices properly** with `make()` or literal syntax
3. **Consider embedding** for composition over inheritance
4. **Use interfaces efficiently** - keep them small and focused
5. **Handle nil checks** explicitly

### Common Best Practices:
1. **Keep interfaces minimal** - follow Interface Segregation Principle
2. **Provide both add and remove operations** for composites
3. **Consider thread safety** in concurrent environments
4. **Implement proper error handling** for edge cases
5. **Document the tree structure** clearly

## Pros and Cons

### Advantages:
- **Uniform treatment**: Clients can treat individual and composite objects uniformly
- **Extensibility**: Easy to add new types of components
- **Flexibility**: Can create complex tree structures
- **Simplified client code**: No need to distinguish between leaf and composite objects

### Disadvantages:
- **Overgeneralization**: Can make the design overly general
- **Type safety**: May need runtime checks to ensure operations are valid
- **Performance**: Traversing deep hierarchies can be expensive
- **Complexity**: Can be overkill for simple hierarchies

### Java-Specific Pros:
- Strong type system with compile-time checks
- Rich collection framework
- Mature ecosystem and tooling

### Java-Specific Cons:
- More verbose syntax
- Requires explicit interface implementation
- Memory overhead from object headers

### Go-Specific Pros:
- Implicit interface implementation
- Better performance characteristics
- Simpler syntax
- Built-in concurrency support

### Go-Specific Cons:
- Less mature ecosystem
- No generics (in older versions)
- Manual memory management awareness needed
- Limited inheritance mechanisms

## Conclusion

The Composite pattern is powerful for representing hierarchical structures. Java's object-oriented approach provides strong typing and explicit contracts, while Go's composition-based approach offers simplicity and performance. Choose based on your specific requirements:

- **Choose Java** when you need strong typing, extensive tooling, and complex object hierarchies
- **Choose Go** when you prioritize performance, simplicity, and concurrent operations