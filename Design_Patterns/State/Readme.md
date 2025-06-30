# Design Patterns: Complete Guide Starting with State Pattern

## Table of Contents
1. [Introduction to Design Patterns](#introduction-to-design-patterns)
2. [State Pattern Overview](#state-pattern-overview)
3. [Implementation Comparison: Go vs Java](#implementation-comparison-go-vs-java)
4. [Step-by-Step Implementation](#step-by-step-implementation)
5. [Complete Code Examples](#complete-code-examples)
6. [Language-Specific Analysis](#language-specific-analysis)
7. [Best Practices](#best-practices)

## Introduction to Design Patterns

Design patterns are reusable solutions to commonly occurring problems in software design. They represent best practices evolved over time and provide a shared vocabulary for developers.

### Benefits:
- **Reusability**: Proven solutions that can be applied across projects
- **Communication**: Common vocabulary for design discussions
- **Maintainability**: Well-structured code that's easier to modify
- **Flexibility**: Designs that adapt to changing requirements

## State Pattern Overview

The State pattern allows an object to alter its behavior when its internal state changes. The object will appear to change its class.

### Key Components:
- **Context**: The class that maintains a reference to the current state
- **State Interface**: Defines the interface for state-specific behavior
- **Concrete States**: Implement specific behaviors for each state

### When to Use:
- Object behavior depends on its state
- Operations have large conditional statements based on object state
- State transitions are complex and frequent

### Real-World Examples:
- Traffic light systems
- Document workflow (draft → review → published)
- Media player states (playing, paused, stopped)
- Network connection states

## Implementation Comparison: Go vs Java

Let's implement a **Media Player** example to demonstrate the State pattern in both languages.

## Step-by-Step Implementation

### Step 1: Define the State Interface

**Java Approach:**
```java
// State interface defining common behavior
public interface PlayerState {
    void play(MediaPlayer context);
    void pause(MediaPlayer context);
    void stop(MediaPlayer context);
    String getStatus();
}
```

**Go Approach:**
```go
// State interface - implicitly satisfied
type PlayerState interface {
    Play(context *MediaPlayer)
    Pause(context *MediaPlayer)
    Stop(context *MediaPlayer)
    GetStatus() string
}
```

**Key Differences:**
- **Java**: Explicit interface declaration with `public interface`
- **Go**: Interface automatically satisfied by any type implementing the methods
- **Java**: CamelCase method names
- **Go**: Exported methods start with capital letters

### Step 2: Implement Concrete States

**Java - Playing State:**
```java
public class PlayingState implements PlayerState {
    @Override
    public void play(MediaPlayer context) {
        System.out.println("Already playing");
    }
    
    @Override
    public void pause(MediaPlayer context) {
        System.out.println("Pausing playback");
        context.setState(new PausedState());
    }
    
    @Override
    public void stop(MediaPlayer context) {
        System.out.println("Stopping playback");
        context.setState(new StoppedState());
    }
    
    @Override
    public String getStatus() {
        return "Playing";
    }
}
```

**Go - Playing State:**
```go
type PlayingState struct{}

func (p *PlayingState) Play(context *MediaPlayer) {
    fmt.Println("Already playing")
}

func (p *PlayingState) Pause(context *MediaPlayer) {
    fmt.Println("Pausing playback")
    context.SetState(&PausedState{})
}

func (p *PlayingState) Stop(context *MediaPlayer) {
    fmt.Println("Stopping playback")
    context.SetState(&StoppedState{})
}

func (p *PlayingState) GetStatus() string {
    return "Playing"
}
```

**Key Differences:**
- **Java**: Classes implement interfaces explicitly with `implements`
- **Go**: Structs satisfy interfaces implicitly through method receivers
- **Java**: Method overriding with `@Override` annotation
- **Go**: Method receivers `(p *PlayingState)` define methods on types
- **Go**: Explicit pointer usage for state transitions

### Step 3: Create the Context Class

**Java Context:**
```java
public class MediaPlayer {
    private PlayerState currentState;
    private String currentTrack;
    
    public MediaPlayer(String track) {
        this.currentTrack = track;
        this.currentState = new StoppedState(); // Initial state
    }
    
    public void setState(PlayerState state) {
        this.currentState = state;
    }
    
    public void play() {
        currentState.play(this);
    }
    
    public void pause() {
        currentState.pause(this);
    }
    
    public void stop() {
        currentState.stop(this);
    }
    
    public String getStatus() {
        return currentState.getStatus();
    }
    
    public String getCurrentTrack() {
        return currentTrack;
    }
}
```

**Go Context:**
```go
type MediaPlayer struct {
    currentState PlayerState
    currentTrack string
}

func NewMediaPlayer(track string) *MediaPlayer {
    return &MediaPlayer{
        currentTrack: track,
        currentState: &StoppedState{}, // Initial state
    }
}

func (m *MediaPlayer) SetState(state PlayerState) {
    m.currentState = state
}

func (m *MediaPlayer) Play() {
    m.currentState.Play(m)
}

func (m *MediaPlayer) Pause() {
    m.currentState.Pause(m)
}

func (m *MediaPlayer) Stop() {
    m.currentState.Stop(m)
}

func (m *MediaPlayer) GetStatus() string {
    return m.currentState.GetStatus()
}

func (m *MediaPlayer) GetCurrentTrack() string {
    return m.currentTrack
}
```

**Key Differences:**
- **Java**: Constructor with `public MediaPlayer()`
- **Go**: Constructor function `NewMediaPlayer()` returning pointer
- **Java**: Private fields with getter methods
- **Go**: Exported fields (capital letters) or getter methods
- **Java**: `this` keyword for self-reference
- **Go**: Method receiver for accessing struct fields

## Complete Code Examples

### Java Complete Implementation

```java
// State interface
interface PlayerState {
    void play(MediaPlayer context);
    void pause(MediaPlayer context);
    void stop(MediaPlayer context);
    String getStatus();
}

// Concrete States
class StoppedState implements PlayerState {
    @Override
    public void play(MediaPlayer context) {
        System.out.println("Starting playback of: " + context.getCurrentTrack());
        context.setState(new PlayingState());
    }
    
    @Override
    public void pause(MediaPlayer context) {
        System.out.println("Cannot pause - player is stopped");
    }
    
    @Override
    public void stop(MediaPlayer context) {
        System.out.println("Already stopped");
    }
    
    @Override
    public String getStatus() {
        return "Stopped";
    }
}

class PlayingState implements PlayerState {
    @Override
    public void play(MediaPlayer context) {
        System.out.println("Already playing");
    }
    
    @Override
    public void pause(MediaPlayer context) {
        System.out.println("Pausing playback");
        context.setState(new PausedState());
    }
    
    @Override
    public void stop(MediaPlayer context) {
        System.out.println("Stopping playback");
        context.setState(new StoppedState());
    }
    
    @Override
    public String getStatus() {
        return "Playing";
    }
}

class PausedState implements PlayerState {
    @Override
    public void play(MediaPlayer context) {
        System.out.println("Resuming playback");
        context.setState(new PlayingState());
    }
    
    @Override
    public void pause(MediaPlayer context) {
        System.out.println("Already paused");
    }
    
    @Override
    public void stop(MediaPlayer context) {
        System.out.println("Stopping from pause");
        context.setState(new StoppedState());
    }
    
    @Override
    public String getStatus() {
        return "Paused";
    }
}

// Context class
class MediaPlayer {
    private PlayerState currentState;
    private String currentTrack;
    
    public MediaPlayer(String track) {
        this.currentTrack = track;
        this.currentState = new StoppedState();
    }
    
    public void setState(PlayerState state) {
        this.currentState = state;
    }
    
    public void play() {
        currentState.play(this);
    }
    
    public void pause() {
        currentState.pause(this);
    }
    
    public void stop() {
        currentState.stop(this);
    }
    
    public String getStatus() {
        return currentState.getStatus();
    }
    
    public String getCurrentTrack() {
        return currentTrack;
    }
}

// Demo class
public class MediaPlayerDemo {
    public static void main(String[] args) {
        MediaPlayer player = new MediaPlayer("Bohemian Rhapsody");
        
        System.out.println("=== Media Player State Pattern Demo ===");
        System.out.println("Track: " + player.getCurrentTrack());
        System.out.println("Initial Status: " + player.getStatus());
        System.out.println();
        
        // Test state transitions
        System.out.println("1. Trying to play:");
        player.play();
        System.out.println("Status: " + player.getStatus());
        System.out.println();
        
        System.out.println("2. Trying to pause:");
        player.pause();
        System.out.println("Status: " + player.getStatus());
        System.out.println();
        
        System.out.println("3. Trying to play again (resume):");
        player.play();
        System.out.println("Status: " + player.getStatus());
        System.out.println();
        
        System.out.println("4. Trying to stop:");
        player.stop();
        System.out.println("Status: " + player.getStatus());
        System.out.println();
        
        System.out.println("5. Trying to pause when stopped:");
        player.pause();
        System.out.println("Final Status: " + player.getStatus());
    }
}
```

### Go Complete Implementation

```go
package main

import "fmt"

// State interface
type PlayerState interface {
    Play(context *MediaPlayer)
    Pause(context *MediaPlayer)
    Stop(context *MediaPlayer)
    GetStatus() string
}

// Concrete States
type StoppedState struct{}

func (s *StoppedState) Play(context *MediaPlayer) {
    fmt.Printf("Starting playback of: %s\n", context.GetCurrentTrack())
    context.SetState(&PlayingState{})
}

func (s *StoppedState) Pause(context *MediaPlayer) {
    fmt.Println("Cannot pause - player is stopped")
}

func (s *StoppedState) Stop(context *MediaPlayer) {
    fmt.Println("Already stopped")
}

func (s *StoppedState) GetStatus() string {
    return "Stopped"
}

type PlayingState struct{}

func (p *PlayingState) Play(context *MediaPlayer) {
    fmt.Println("Already playing")
}

func (p *PlayingState) Pause(context *MediaPlayer) {
    fmt.Println("Pausing playback")
    context.SetState(&PausedState{})
}

func (p *PlayingState) Stop(context *MediaPlayer) {
    fmt.Println("Stopping playback")
    context.SetState(&StoppedState{})
}

func (p *PlayingState) GetStatus() string {
    return "Playing"
}

type PausedState struct{}

func (pa *PausedState) Play(context *MediaPlayer) {
    fmt.Println("Resuming playback")
    context.SetState(&PlayingState{})
}

func (pa *PausedState) Pause(context *MediaPlayer) {
    fmt.Println("Already paused")
}

func (pa *PausedState) Stop(context *MediaPlayer) {
    fmt.Println("Stopping from pause")
    context.SetState(&StoppedState{})
}

func (pa *PausedState) GetStatus() string {
    return "Paused"
}

// Context struct
type MediaPlayer struct {
    currentState PlayerState
    currentTrack string
}

// Constructor function
func NewMediaPlayer(track string) *MediaPlayer {
    return &MediaPlayer{
        currentTrack: track,
        currentState: &StoppedState{},
    }
}

func (m *MediaPlayer) SetState(state PlayerState) {
    m.currentState = state
}

func (m *MediaPlayer) Play() {
    m.currentState.Play(m)
}

func (m *MediaPlayer) Pause() {
    m.currentState.Pause(m)
}

func (m *MediaPlayer) Stop() {
    m.currentState.Stop(m)
}

func (m *MediaPlayer) GetStatus() string {
    return m.currentState.GetStatus()
}

func (m *MediaPlayer) GetCurrentTrack() string {
    return m.currentTrack
}

// Demo function
func main() {
    player := NewMediaPlayer("Bohemian Rhapsody")
    
    fmt.Println("=== Media Player State Pattern Demo ===")
    fmt.Printf("Track: %s\n", player.GetCurrentTrack())
    fmt.Printf("Initial Status: %s\n", player.GetStatus())
    fmt.Println()
    
    // Test state transitions
    fmt.Println("1. Trying to play:")
    player.Play()
    fmt.Printf("Status: %s\n", player.GetStatus())
    fmt.Println()
    
    fmt.Println("2. Trying to pause:")
    player.Pause()
    fmt.Printf("Status: %s\n", player.GetStatus())
    fmt.Println()
    
    fmt.Println("3. Trying to play again (resume):")
    player.Play()
    fmt.Printf("Status: %s\n", player.GetStatus())
    fmt.Println()
    
    fmt.Println("4. Trying to stop:")
    player.Stop()
    fmt.Printf("Status: %s\n", player.GetStatus())
    fmt.Println()
    
    fmt.Println("5. Trying to pause when stopped:")
    player.Pause()
    fmt.Printf("Final Status: %s\n", player.GetStatus())
}
```

## Language-Specific Analysis

### Java Characteristics

**Strengths:**
- **Strong Type System**: Compile-time type checking prevents many errors
- **Explicit Contracts**: Interface implementations are clearly declared
- **Rich Standard Library**: Extensive built-in functionality
- **Mature Tooling**: IDEs provide excellent refactoring and debugging support
- **Memory Management**: Automatic garbage collection

**Implementation Details:**
- **Access Modifiers**: `private`, `public`, `protected` for encapsulation
- **Inheritance**: Single inheritance with interface implementation
- **Method Overriding**: `@Override` annotation for clarity
- **Constructors**: Explicit constructor methods
- **Exception Handling**: Try-catch blocks for error management

### Go Characteristics

**Strengths:**
- **Simplicity**: Minimal syntax and concepts to learn
- **Implicit Interfaces**: Duck typing reduces coupling
- **Performance**: Compiled to native code, efficient execution
- **Concurrency**: Built-in goroutines and channels
- **Fast Compilation**: Quick build times

**Implementation Details:**
- **Struct Composition**: Embedding for code reuse instead of inheritance
- **Method Receivers**: Functions attached to types
- **Pointer Semantics**: Explicit memory management control
- **No Classes**: Structs and interfaces provide object-oriented features
- **Error Handling**: Explicit error return values

### Side-by-Side Comparison

| Aspect | Java | Go |
|--------|------|-----|
| **Interface Declaration** | Explicit with `implements` | Implicit satisfaction |
| **Method Definition** | Inside class with `@Override` | Receiver functions on structs |
| **Constructor** | Class constructor method | Function returning pointer |
| **Memory Management** | Garbage collection | Manual with garbage collection |
| **Type Safety** | Compile-time checking | Compile-time checking |
| **Error Handling** | Exceptions | Error return values |
| **Concurrency** | Threads and locks | Goroutines and channels |
| **Code Verbosity** | More verbose | More concise |

## Best Practices

### General State Pattern Best Practices

1. **Single Responsibility**: Each state should handle only its specific behavior
2. **State Transitions**: Keep transitions simple and well-defined
3. **Context Independence**: States shouldn't know about other states directly
4. **Immutable States**: Consider making state objects stateless
5. **Error Handling**: Handle invalid state transitions gracefully

### Java-Specific Best Practices

```java
// Use enums for simple states
public enum SimpleState {
    PLAYING, PAUSED, STOPPED
}

// Consider state factories for complex initialization
public class StateFactory {
    public static PlayerState createPlayingState(String metadata) {
        return new PlayingState(metadata);
    }
}

// Use abstract base class for common functionality
public abstract class BasePlayerState implements PlayerState {
    protected void logStateChange(String from, String to) {
        System.out.println("State changed: " + from + " -> " + to);
    }
}
```

### Go-Specific Best Practices

```go
// Use interfaces for testability
type StateLogger interface {
    LogTransition(from, to string)
}

// Consider state factories
func NewPlayerState(stateType string) PlayerState {
    switch stateType {
    case "playing":
        return &PlayingState{}
    case "paused":
        return &PausedState{}
    default:
        return &StoppedState{}
    }
}

// Use embedded structs for common functionality
type BaseState struct {
    logger StateLogger
}

func (b *BaseState) logTransition(from, to string) {
    if b.logger != nil {
        b.logger.LogTransition(from, to)
    }
}
```

### Performance Considerations

**Java:**
- State objects can be reused to reduce garbage collection
- Consider using flyweight pattern for stateless state objects
- Thread safety may require synchronization

**Go:**
- Struct allocation is generally cheap
- Consider pointer receivers for large states
- Goroutine safety may require channels or mutexes

### Testing Strategies

**Java Testing:**
```java
@Test
public void testStateTransition() {
    MediaPlayer player = new MediaPlayer("test.mp3");
    assertEquals("Stopped", player.getStatus());
    
    player.play();
    assertEquals("Playing", player.getStatus());
    
    player.pause();
    assertEquals("Paused", player.getStatus());
}
```

**Go Testing:**
```go
func TestStateTransition(t *testing.T) {
    player := NewMediaPlayer("test.mp3")
    assert.Equal(t, "Stopped", player.GetStatus())
    
    player.Play()
    assert.Equal(t, "Playing", player.GetStatus())
    
    player.Pause()
    assert.Equal(t, "Paused", player.GetStatus())
}
```

## Conclusion

The State pattern provides an elegant solution for managing object behavior that changes with internal state. Both Java and Go offer effective ways to implement this pattern, each with their own strengths:

- **Java** excels with its explicit type system, rich tooling, and object-oriented features
- **Go** shines with its simplicity, implicit interfaces, and excellent concurrency support
