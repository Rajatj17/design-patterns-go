# Design Patterns Guide: Adapter Pattern

## Table of Contents
1. [Introduction](#introduction)
2. [Adapter Pattern Overview](#adapter-pattern-overview)
3. [Implementation Comparison: Go vs Java](#implementation-comparison-go-vs-java)
4. [Real-World Example](#real-world-example)
5. [Key Differences](#key-differences)
6. [Best Practices](#best-practices)
7. [When to Use](#when-to-use)

## Introduction

The Adapter pattern is a structural design pattern that allows objects with incompatible interfaces to collaborate. It acts as a bridge between two incompatible interfaces by wrapping an existing class with a new interface.

## Adapter Pattern Overview

### Structure
- **Target**: The interface that the client expects
- **Adaptee**: The existing class with an incompatible interface
- **Adapter**: The class that implements the target interface and wraps the adaptee
- **Client**: The class that uses the target interface

### Problem Solved
When you have two classes that should work together but can't because of incompatible interfaces, the Adapter pattern provides a solution without modifying existing code.

## Implementation Comparison: Go vs Java

### Scenario: Media Player
Let's implement a media player that can play different audio formats. We have an existing `Mp3Player` but want to support `Mp4` and `Vlc` formats through an adapter.

---

## Go Implementation

### Step 1: Define the Target Interface

```go
// MediaPlayer is the target interface that client expects
type MediaPlayer interface {
    Play(audioType string, fileName string)
}
```

### Step 2: Create the Adaptee (Existing incompatible classes)

```go
// Mp4Player - existing class with different interface
type Mp4Player struct{}

func (m *Mp4Player) PlayMp4(fileName string) {
    fmt.Printf("Playing mp4 file: %s\n", fileName)
}

// VlcPlayer - another existing class with different interface
type VlcPlayer struct{}

func (v *VlcPlayer) PlayVlc(fileName string) {
    fmt.Printf("Playing vlc file: %s\n", fileName)
}
```

### Step 3: Create the Adapter

```go
// MediaAdapter implements MediaPlayer interface and adapts other players
type MediaAdapter struct {
    audioType string
    mp4Player *Mp4Player
    vlcPlayer *VlcPlayer
}

func NewMediaAdapter(audioType string) *MediaAdapter {
    adapter := &MediaAdapter{audioType: audioType}
    
    switch audioType {
    case "mp4":
        adapter.mp4Player = &Mp4Player{}
    case "vlc":
        adapter.vlcPlayer = &VlcPlayer{}
    }
    
    return adapter
}

func (m *MediaAdapter) Play(audioType string, fileName string) {
    switch audioType {
    case "mp4":
        m.mp4Player.PlayMp4(fileName)
    case "vlc":
        m.vlcPlayer.PlayVlc(fileName)
    default:
        fmt.Printf("Invalid media. %s format not supported\n", audioType)
    }
}
```

### Step 4: Create the Context (Client)

```go
// AudioPlayer implements MediaPlayer and uses adapter for unsupported formats
type AudioPlayer struct {
    adapter MediaPlayer
}

func (a *AudioPlayer) Play(audioType string, fileName string) {
    switch audioType {
    case "mp3":
        fmt.Printf("Playing mp3 file: %s\n", fileName)
    case "mp4", "vlc":
        a.adapter = NewMediaAdapter(audioType)
        a.adapter.Play(audioType, fileName)
    default:
        fmt.Printf("Invalid media. %s format not supported\n", audioType)
    }
}
```

### Step 5: Complete Go Example

```go
package main

import "fmt"

func main() {
    player := &AudioPlayer{}
    
    player.Play("mp3", "beyond_the_horizon.mp3")
    player.Play("mp4", "alone.mp4")
    player.Play("vlc", "far_far_away.vlc")
    player.Play("avi", "mind_me.avi")
}
```

---

## Java Implementation

### Step 1: Define the Target Interface

```java
// MediaPlayer is the target interface that client expects
public interface MediaPlayer {
    void play(String audioType, String fileName);
}
```

### Step 2: Create the Adaptee (Existing incompatible classes)

```java
// Mp4Player - existing class with different interface
public class Mp4Player {
    public void playMp4(String fileName) {
        System.out.println("Playing mp4 file: " + fileName);
    }
}

// VlcPlayer - another existing class with different interface
public class VlcPlayer {
    public void playVlc(String fileName) {
        System.out.println("Playing vlc file: " + fileName);
    }
}
```

### Step 3: Create Advanced Media Player Interface (Optional)

```java
// AdvancedMediaPlayer interface for advanced players
public interface AdvancedMediaPlayer {
    void playVlc(String fileName);
    void playMp4(String fileName);
}

// Concrete implementations
public class VlcPlayerImpl implements AdvancedMediaPlayer {
    @Override
    public void playVlc(String fileName) {
        System.out.println("Playing vlc file: " + fileName);
    }
    
    @Override
    public void playMp4(String fileName) {
        // Do nothing
    }
}

public class Mp4PlayerImpl implements AdvancedMediaPlayer {
    @Override
    public void playVlc(String fileName) {
        // Do nothing
    }
    
    @Override
    public void playMp4(String fileName) {
        System.out.println("Playing mp4 file: " + fileName);
    }
}
```

### Step 4: Create the Adapter

```java
// MediaAdapter implements MediaPlayer interface and adapts other players
public class MediaAdapter implements MediaPlayer {
    private AdvancedMediaPlayer advancedMusicPlayer;
    
    public MediaAdapter(String audioType) {
        switch (audioType.toLowerCase()) {
            case "vlc":
                advancedMusicPlayer = new VlcPlayerImpl();
                break;
            case "mp4":
                advancedMusicPlayer = new Mp4PlayerImpl();
                break;
            default:
                throw new IllegalArgumentException("Unsupported audio type: " + audioType);
        }
    }
    
    @Override
    public void play(String audioType, String fileName) {
        switch (audioType.toLowerCase()) {
            case "vlc":
                advancedMusicPlayer.playVlc(fileName);
                break;
            case "mp4":
                advancedMusicPlayer.playMp4(fileName);
                break;
            default:
                System.out.println("Invalid media. " + audioType + " format not supported");
        }
    }
}
```

### Step 5: Create the Context (Client)

```java
// AudioPlayer implements MediaPlayer and uses adapter for unsupported formats
public class AudioPlayer implements MediaPlayer {
    private MediaAdapter mediaAdapter;
    
    @Override
    public void play(String audioType, String fileName) {
        // Built-in support for mp3 music files
        if (audioType.equalsIgnoreCase("mp3")) {
            System.out.println("Playing mp3 file: " + fileName);
        }
        // MediaAdapter is providing support to play other file formats
        else if (audioType.equalsIgnoreCase("vlc") || audioType.equalsIgnoreCase("mp4")) {
            mediaAdapter = new MediaAdapter(audioType);
            mediaAdapter.play(audioType, fileName);
        } else {
            System.out.println("Invalid media. " + audioType + " format not supported");
        }
    }
}
```

### Step 6: Complete Java Example

```java
public class AdapterPatternDemo {
    public static void main(String[] args) {
        AudioPlayer audioPlayer = new AudioPlayer();
        
        audioPlayer.play("mp3", "beyond_the_horizon.mp3");
        audioPlayer.play("mp4", "alone.mp4");
        audioPlayer.play("vlc", "far_far_away.vlc");
        audioPlayer.play("avi", "mind_me.avi");
    }
}
```

---

## Real-World Example

### Database Connection Adapter

#### Go Implementation

```go
// Target interface
type DatabaseConnection interface {
    Connect() error
    Query(sql string) ([]map[string]interface{}, error)
    Close() error
}

// Legacy MySQL connector (Adaptee)
type LegacyMySQLConnector struct {
    host     string
    username string
    password string
}

func (l *LegacyMySQLConnector) EstablishConnection() error {
    fmt.Printf("Connecting to MySQL: %s\n", l.host)
    return nil
}

func (l *LegacyMySQLConnector) ExecuteQuery(query string) []map[string]interface{} {
    fmt.Printf("Executing MySQL query: %s\n", query)
    return []map[string]interface{}{{"id": 1, "name": "John"}}
}

func (l *LegacyMySQLConnector) CloseConnection() error {
    fmt.Println("Closing MySQL connection")
    return nil
}

// Adapter
type MySQLAdapter struct {
    legacyConnector *LegacyMySQLConnector
}

func NewMySQLAdapter(host, username, password string) *MySQLAdapter {
    return &MySQLAdapter{
        legacyConnector: &LegacyMySQLConnector{
            host:     host,
            username: username,
            password: password,
        },
    }
}

func (m *MySQLAdapter) Connect() error {
    return m.legacyConnector.EstablishConnection()
}

func (m *MySQLAdapter) Query(sql string) ([]map[string]interface{}, error) {
    result := m.legacyConnector.ExecuteQuery(sql)
    return result, nil
}

func (m *MySQLAdapter) Close() error {
    return m.legacyConnector.CloseConnection()
}
```

#### Java Implementation

```java
// Target interface
public interface DatabaseConnection {
    void connect() throws Exception;
    List<Map<String, Object>> query(String sql) throws Exception;
    void close() throws Exception;
}

// Legacy MySQL connector (Adaptee)
public class LegacyMySQLConnector {
    private String host;
    private String username;
    private String password;
    
    public LegacyMySQLConnector(String host, String username, String password) {
        this.host = host;
        this.username = username;
        this.password = password;
    }
    
    public void establishConnection() throws Exception {
        System.out.println("Connecting to MySQL: " + host);
    }
    
    public List<Map<String, Object>> executeQuery(String query) {
        System.out.println("Executing MySQL query: " + query);
        Map<String, Object> row = new HashMap<>();
        row.put("id", 1);
        row.put("name", "John");
        return Arrays.asList(row);
    }
    
    public void closeConnection() throws Exception {
        System.out.println("Closing MySQL connection");
    }
}

// Adapter
public class MySQLAdapter implements DatabaseConnection {
    private LegacyMySQLConnector legacyConnector;
    
    public MySQLAdapter(String host, String username, String password) {
        this.legacyConnector = new LegacyMySQLConnector(host, username, password);
    }
    
    @Override
    public void connect() throws Exception {
        legacyConnector.establishConnection();
    }
    
    @Override
    public List<Map<String, Object>> query(String sql) throws Exception {
        return legacyConnector.executeQuery(sql);
    }
    
    @Override
    public void close() throws Exception {
        legacyConnector.closeConnection();
    }
}
```

---

## Key Differences Between Go and Java Implementations

### 1. **Interface Definition**
- **Go**: Uses implicit interface implementation - no explicit `implements` keyword
- **Java**: Requires explicit interface implementation with `implements` keyword

### 2. **Object Creation**
- **Go**: Uses factory functions (e.g., `NewMediaAdapter()`) and struct literals
- **Java**: Uses constructors and `new` keyword

### 3. **Error Handling**
- **Go**: Multiple return values with explicit error handling
- **Java**: Exception-based error handling with try-catch blocks

### 4. **Memory Management**
- **Go**: Automatic garbage collection, pointers for efficiency
- **Java**: Automatic garbage collection, references instead of pointers

### 5. **Type System**
- **Go**: Static typing with type inference, composition over inheritance
- **Java**: Static typing with explicit inheritance and polymorphism

### 6. **Method Signatures**
- **Go**: Methods can be defined on any type, receiver-based
- **Java**: Methods belong to classes, this-based

---

## Best Practices

### Go Best Practices
1. **Use interfaces judiciously** - Keep them small and focused
2. **Prefer composition** - Use embedding and struct composition
3. **Handle errors explicitly** - Always check and handle errors
4. **Use factory functions** - For complex object initialization
5. **Keep adapters stateless** - When possible, avoid storing state

```go
// Good: Stateless adapter
type StatelessAdapter struct{}

func (s *StatelessAdapter) Adapt(adaptee Adaptee) Target {
    return &ConcreteTarget{adaptee: adaptee}
}

// Better: Factory function
func NewAdapter(adaptee Adaptee) Target {
    return &ConcreteTarget{adaptee: adaptee}
}
```

### Java Best Practices
1. **Use dependency injection** - Make adapters more testable
2. **Implement proper exception handling** - Use specific exception types
3. **Follow SOLID principles** - Especially Single Responsibility
4. **Use builder pattern** - For complex adapter configuration
5. **Make adapters thread-safe** - When used in concurrent environments

```java
// Good: Thread-safe adapter
public class ThreadSafeAdapter implements Target {
    private final Object lock = new Object();
    private final Adaptee adaptee;
    
    public ThreadSafeAdapter(Adaptee adaptee) {
        this.adaptee = adaptee;
    }
    
    @Override
    public void operation() {
        synchronized(lock) {
            adaptee.specificOperation();
        }
    }
}
```

---

## When to Use the Adapter Pattern

### Use When:
1. **Legacy system integration** - Connecting old systems with new interfaces
2. **Third-party library integration** - Adapting external APIs to your interface
3. **Interface incompatibility** - When you can't modify existing classes
4. **Testing** - Creating mock adapters for testing purposes
5. **API versioning** - Supporting multiple versions of an API

### Don't Use When:
1. **You can modify the source** - Direct modification is simpler
2. **Simple interface differences** - Method overloading might suffice
3. **Performance is critical** - Adapters add an extra layer
4. **The incompatibility is temporary** - Consider other solutions

---

## Output Example

When running either implementation:

```
Playing mp3 file: beyond_the_horizon.mp3
Playing mp4 file: alone.mp4
Playing vlc file: far_far_away.vlc
Invalid media. avi format not supported
```

---

## Summary

The Adapter pattern provides a clean way to integrate incompatible interfaces. While Go's implementation tends to be more concise due to implicit interfaces and composition, Java's explicit approach offers clearer contracts and stronger type safety. Both languages effectively implement the pattern, with the choice depending on your specific requirements and language preferences.

The key is to understand that the Adapter pattern is about **interface compatibility**, not feature enhancement. It allows existing code to work with new interfaces without modification, making it invaluable for system integration and legacy code maintenance.