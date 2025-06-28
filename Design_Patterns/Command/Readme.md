# Command Pattern: Implementation Comparison

## Java Implementation

### 1. Command Interface

```java
// Command interface - defines the contract
interface Command {
    void execute();
    void undo();    // For supporting undo operations
}
```

### 2. Receiver Classes (Objects that do the actual work)

```java
// Light receiver
class Light {
    private String location;
    private boolean isOn = false;
    
    public Light(String location) {
        this.location = location;
    }
    
    public void turnOn() {
        isOn = true;
        System.out.println(location + " light is ON");
    }
    
    public void turnOff() {
        isOn = false;
        System.out.println(location + " light is OFF");
    }
    
    public boolean isOn() {
        return isOn;
    }
}

// Stereo receiver
class Stereo {
    private String location;
    private boolean isOn = false;
    private int volume = 0;
    
    public Stereo(String location) {
        this.location = location;
    }
    
    public void turnOn() {
        isOn = true;
        System.out.println(location + " stereo is ON");
    }
    
    public void turnOff() {
        isOn = false;
        System.out.println(location + " stereo is OFF");
    }
    
    public void setVolume(int volume) {
        this.volume = volume;
        System.out.println(location + " stereo volume set to " + volume);
    }
    
    public int getVolume() {
        return volume;
    }
    
    public boolean isOn() {
        return isOn;
    }
}
```

### 3. Concrete Command Classes

```java
// Light On Command
class LightOnCommand implements Command {
    private Light light;
    
    public LightOnCommand(Light light) {
        this.light = light;
    }
    
    @Override
    public void execute() {
        light.turnOn();
    }
    
    @Override
    public void undo() {
        light.turnOff();
    }
}

// Light Off Command
class LightOffCommand implements Command {
    private Light light;
    
    public LightOffCommand(Light light) {
        this.light = light;
    }
    
    @Override
    public void execute() {
        light.turnOff();
    }
    
    @Override
    public void undo() {
        light.turnOn();
    }
}

// Stereo On with Volume Command
class StereoOnWithVolumeCommand implements Command {
    private Stereo stereo;
    private int previousVolume;
    
    public StereoOnWithVolumeCommand(Stereo stereo) {
        this.stereo = stereo;
    }
    
    @Override
    public void execute() {
        previousVolume = stereo.getVolume();
        stereo.turnOn();
        stereo.setVolume(11);
    }
    
    @Override
    public void undo() {
        stereo.turnOff();
        stereo.setVolume(previousVolume);
    }
}

// Stereo Off Command
class StereoOffCommand implements Command {
    private Stereo stereo;
    private int previousVolume;
    
    public StereoOffCommand(Stereo stereo) {
        this.stereo = stereo;
    }
    
    @Override
    public void execute() {
        previousVolume = stereo.getVolume();
        stereo.turnOff();
    }
    
    @Override
    public void undo() {
        stereo.turnOn();
        stereo.setVolume(previousVolume);
    }
}

// Null Object Pattern for empty slots
class NoCommand implements Command {
    @Override
    public void execute() {
        // Do nothing
    }
    
    @Override
    public void undo() {
        // Do nothing
    }
}

// Macro Command - executes multiple commands
class MacroCommand implements Command {
    private Command[] commands;
    
    public MacroCommand(Command[] commands) {
        this.commands = commands;
    }
    
    @Override
    public void execute() {
        for (Command command : commands) {
            command.execute();
        }
    }
    
    @Override
    public void undo() {
        // Undo in reverse order
        for (int i = commands.length - 1; i >= 0; i--) {
            commands[i].undo();
        }
    }
}
```

### 4. Invoker (Remote Control)

```java
// Remote Control - The invoker
class RemoteControl {
    private Command[] onCommands;
    private Command[] offCommands;
    private Command undoCommand;
    
    public RemoteControl() {
        onCommands = new Command[7];
        offCommands = new Command[7];
        
        Command noCommand = new NoCommand();
        for (int i = 0; i < 7; i++) {
            onCommands[i] = noCommand;
            offCommands[i] = noCommand;
        }
        undoCommand = noCommand;
    }
    
    public void setCommand(int slot, Command onCommand, Command offCommand) {
        onCommands[slot] = onCommand;
        offCommands[slot] = offCommand;
    }
    
    public void onButtonPressed(int slot) {
        onCommands[slot].execute();
        undoCommand = onCommands[slot];
    }
    
    public void offButtonPressed(int slot) {
        offCommands[slot].execute();
        undoCommand = offCommands[slot];
    }
    
    public void undoButtonPressed() {
        undoCommand.undo();
    }
    
    @Override
    public String toString() {
        StringBuilder sb = new StringBuilder();
        sb.append("\n------ Remote Control ------\n");
        for (int i = 0; i < onCommands.length; i++) {
            sb.append("[slot ").append(i).append("] ")
              .append(onCommands[i].getClass().getSimpleName())
              .append("    ")
              .append(offCommands[i].getClass().getSimpleName())
              .append("\n");
        }
        sb.append("[undo] ").append(undoCommand.getClass().getSimpleName()).append("\n");
        return sb.toString();
    }
}
```

### 5. Java Usage Example

```java
public class CommandPatternDemo {
    public static void main(String[] args) {
        // Create invoker
        RemoteControl remoteControl = new RemoteControl();
        
        // Create receivers
        Light livingRoomLight = new Light("Living Room");
        Light kitchenLight = new Light("Kitchen");
        Stereo stereo = new Stereo("Living Room");
        
        // Create commands
        LightOnCommand livingRoomLightOn = new LightOnCommand(livingRoomLight);
        LightOffCommand livingRoomLightOff = new LightOffCommand(livingRoomLight);
        
        LightOnCommand kitchenLightOn = new LightOnCommand(kitchenLight);
        LightOffCommand kitchenLightOff = new LightOffCommand(kitchenLight);
        
        StereoOnWithVolumeCommand stereoOnWithVolume = new StereoOnWithVolumeCommand(stereo);
        StereoOffCommand stereoOff = new StereoOffCommand(stereo);
        
        // Set commands to remote control slots
        remoteControl.setCommand(0, livingRoomLightOn, livingRoomLightOff);
        remoteControl.setCommand(1, kitchenLightOn, kitchenLightOff);
        remoteControl.setCommand(2, stereoOnWithVolume, stereoOff);
        
        // Create macro command
        Command[] partyOn = {livingRoomLightOn, stereoOnWithVolume, kitchenLightOn};
        Command[] partyOff = {livingRoomLightOff, stereoOff, kitchenLightOff};
        MacroCommand partyOnMacro = new MacroCommand(partyOn);
        MacroCommand partyOffMacro = new MacroCommand(partyOff);
        
        remoteControl.setCommand(3, partyOnMacro, partyOffMacro);
        
        System.out.println(remoteControl);
        
        // Test individual commands
        System.out.println("=== Testing Individual Commands ===");
        remoteControl.onButtonPressed(0);  // Living room light on
        remoteControl.offButtonPressed(0); // Living room light off
        remoteControl.undoButtonPressed(); // Undo (light back on)
        
        System.out.println("\n=== Testing Macro Command ===");
        remoteControl.onButtonPressed(3);  // Party mode on
        System.out.println("--- Undoing party mode ---");
        remoteControl.undoButtonPressed(); // Undo entire party mode
    }
}
```

## Go Implementation

### 1. Command Interface

```go
// Command interface
type Command interface {
    Execute()
    Undo()
}
```

### 2. Receiver Structs

```go
// Light receiver
type Light struct {
    Location string
    IsOn     bool
}

func NewLight(location string) *Light {
    return &Light{
        Location: location,
        IsOn:     false,
    }
}

func (l *Light) TurnOn() {
    l.IsOn = true
    fmt.Printf("%s light is ON\n", l.Location)
}

func (l *Light) TurnOff() {
    l.IsOn = false
    fmt.Printf("%s light is OFF\n", l.Location)
}

// Stereo receiver
type Stereo struct {
    Location string
    IsOn     bool
    Volume   int
}

func NewStereo(location string) *Stereo {
    return &Stereo{
        Location: location,
        IsOn:     false,
        Volume:   0,
    }
}

func (s *Stereo) TurnOn() {
    s.IsOn = true
    fmt.Printf("%s stereo is ON\n", s.Location)
}

func (s *Stereo) TurnOff() {
    s.IsOn = false
    fmt.Printf("%s stereo is OFF\n", s.Location)
}

func (s *Stereo) SetVolume(volume int) {
    s.Volume = volume
    fmt.Printf("%s stereo volume set to %d\n", s.Location, volume)
}
```

### 3. Concrete Command Structs

```go
// Light On Command
type LightOnCommand struct {
    Light *Light
}

func NewLightOnCommand(light *Light) *LightOnCommand {
    return &LightOnCommand{Light: light}
}

func (loc *LightOnCommand) Execute() {
    loc.Light.TurnOn()
}

func (loc *LightOnCommand) Undo() {
    loc.Light.TurnOff()
}

// Light Off Command
type LightOffCommand struct {
    Light *Light
}

func NewLightOffCommand(light *Light) *LightOffCommand {
    return &LightOffCommand{Light: light}
}

func (loff *LightOffCommand) Execute() {
    loff.Light.TurnOff()
}

func (loff *LightOffCommand) Undo() {
    loff.Light.TurnOn()
}

// Stereo On with Volume Command
type StereoOnWithVolumeCommand struct {
    Stereo         *Stereo
    PreviousVolume int
}

func NewStereoOnWithVolumeCommand(stereo *Stereo) *StereoOnWithVolumeCommand {
    return &StereoOnWithVolumeCommand{
        Stereo:         stereo,
        PreviousVolume: 0,
    }
}

func (sov *StereoOnWithVolumeCommand) Execute() {
    sov.PreviousVolume = sov.Stereo.Volume
    sov.Stereo.TurnOn()
    sov.Stereo.SetVolume(11)
}

func (sov *StereoOnWithVolumeCommand) Undo() {
    sov.Stereo.TurnOff()
    sov.Stereo.SetVolume(sov.PreviousVolume)
}

// Stereo Off Command
type StereoOffCommand struct {
    Stereo         *Stereo
    PreviousVolume int
}

func NewStereoOffCommand(stereo *Stereo) *StereoOffCommand {
    return &StereoOffCommand{
        Stereo:         stereo,
        PreviousVolume: 0,
    }
}

func (soff *StereoOffCommand) Execute() {
    soff.PreviousVolume = soff.Stereo.Volume
    soff.Stereo.TurnOff()
}

func (soff *StereoOffCommand) Undo() {
    soff.Stereo.TurnOn()
    soff.Stereo.SetVolume(soff.PreviousVolume)
}

// No Command (Null Object Pattern)
type NoCommand struct{}

func (nc *NoCommand) Execute() {
    // Do nothing
}

func (nc *NoCommand) Undo() {
    // Do nothing
}

// Macro Command
type MacroCommand struct {
    Commands []Command
}

func NewMacroCommand(commands []Command) *MacroCommand {
    return &MacroCommand{Commands: commands}
}

func (mc *MacroCommand) Execute() {
    for _, command := range mc.Commands {
        command.Execute()
    }
}

func (mc *MacroCommand) Undo() {
    // Undo in reverse order
    for i := len(mc.Commands) - 1; i >= 0; i-- {
        mc.Commands[i].Undo()
    }
}
```

### 4. Invoker (Remote Control)

```go
// Remote Control - The invoker
type RemoteControl struct {
    OnCommands  []Command
    OffCommands []Command
    UndoCommand Command
}

func NewRemoteControl() *RemoteControl {
    noCommand := &NoCommand{}
    onCommands := make([]Command, 7)
    offCommands := make([]Command, 7)
    
    for i := 0; i < 7; i++ {
        onCommands[i] = noCommand
        offCommands[i] = noCommand
    }
    
    return &RemoteControl{
        OnCommands:  onCommands,
        OffCommands: offCommands,
        UndoCommand: noCommand,
    }
}

func (rc *RemoteControl) SetCommand(slot int, onCommand, offCommand Command) {
    rc.OnCommands[slot] = onCommand
    rc.OffCommands[slot] = offCommand
}

func (rc *RemoteControl) OnButtonPressed(slot int) {
    rc.OnCommands[slot].Execute()
    rc.UndoCommand = rc.OnCommands[slot]
}

func (rc *RemoteControl) OffButtonPressed(slot int) {
    rc.OffCommands[slot].Execute()
    rc.UndoCommand = rc.OffCommands[slot]
}

func (rc *RemoteControl) UndoButtonPressed() {
    rc.UndoCommand.Undo()
}

func (rc *RemoteControl) String() string {
    var sb strings.Builder
    sb.WriteString("\n------ Remote Control ------\n")
    
    for i, onCmd := range rc.OnCommands {
        sb.WriteString(fmt.Sprintf("[slot %d] %T    %T\n", 
            i, onCmd, rc.OffCommands[i]))
    }
    sb.WriteString(fmt.Sprintf("[undo] %T\n", rc.UndoCommand))
    
    return sb.String()
}
```

### 5. Go Usage Example

```go
package main

import (
    "fmt"
    "strings"
)

func main() {
    // Create invoker
    remoteControl := NewRemoteControl()
    
    // Create receivers
    livingRoomLight := NewLight("Living Room")
    kitchenLight := NewLight("Kitchen")
    stereo := NewStereo("Living Room")
    
    // Create commands
    livingRoomLightOn := NewLightOnCommand(livingRoomLight)
    livingRoomLightOff := NewLightOffCommand(livingRoomLight)
    
    kitchenLightOn := NewLightOnCommand(kitchenLight)
    kitchenLightOff := NewLightOffCommand(kitchenLight)
    
    stereoOnWithVolume := NewStereoOnWithVolumeCommand(stereo)
    stereoOff := NewStereoOffCommand(stereo)
    
    // Set commands to remote control slots
    remoteControl.SetCommand(0, livingRoomLightOn, livingRoomLightOff)
    remoteControl.SetCommand(1, kitchenLightOn, kitchenLightOff)
    remoteControl.SetCommand(2, stereoOnWithVolume, stereoOff)
    
    // Create macro command
    partyOn := []Command{livingRoomLightOn, stereoOnWithVolume, kitchenLightOn}
    partyOff := []Command{livingRoomLightOff, stereoOff, kitchenLightOff}
    partyOnMacro := NewMacroCommand(partyOn)
    partyOffMacro := NewMacroCommand(partyOff)
    
    remoteControl.SetCommand(3, partyOnMacro, partyOffMacro)
    
    fmt.Println(remoteControl)
    
    // Test individual commands
    fmt.Println("=== Testing Individual Commands ===")
    remoteControl.OnButtonPressed(0)  // Living room light on
    remoteControl.OffButtonPressed(0) // Living room light off
    remoteControl.UndoButtonPressed() // Undo (light back on)
    
    fmt.Println("\n=== Testing Macro Command ===")
    remoteControl.OnButtonPressed(3)  // Party mode on
    fmt.Println("--- Undoing party mode ---")
    remoteControl.UndoButtonPressed() // Undo entire party mode
}
```

## Language-Specific Comparisons

### Interface Definition
- **Java**: Uses `interface` keyword with method signatures
- **Go**: Uses `type` keyword with interface definition, implicit implementation

### Constructor Patterns
- **Java**: Uses constructors and `new` keyword
- **Go**: Uses factory functions (e.g., `NewLightOnCommand`)

### Method Naming
- **Java**: Uses camelCase (`execute`, `undo`)
- **Go**: Uses PascalCase for public methods (`Execute`, `Undo`)

### String Representation
- **Java**: Overrides `toString()` method
- **Go**: Implements `String()` method for `fmt.Stringer` interface

### Array/Slice Handling
- **Java**: Uses arrays with fixed size
- **Go**: Uses slices with `make()` for initialization

## Real-World Applications

### Java Examples:
- **Swing/AWT**: Action objects in GUI frameworks
- **Spring Framework**: Command objects in web controllers
- **Database Transactions**: Transaction commands with rollback
- **Thread Pool Executors**: Runnable commands

### Go Examples:
- **HTTP Handlers**: Handler functions as commands
- **CLI Applications**: Cobra command framework
- **Job Queues**: Background job processing
- **Database Migrations**: Up/Down migration commands

## Benefits

1. **Decoupling**: Separates the object that invokes the operation from the object that performs it
2. **Flexibility**: Easy to add new commands without changing existing code
3. **Undo/Redo**: Built-in support for reversible operations
4. **Macro Commands**: Combine multiple commands into one
5. **Logging**: Easy to log and audit commands
6. **Queuing**: Commands can be stored and executed later

## Drawbacks

1. **Complexity**: Can increase the number of classes
2. **Memory**: Each command may store state for undo operations
3. **Indirection**: Adds a layer between invoker and receiver

## Command vs Strategy Pattern

| Aspect | Command | Strategy |
|--------|---------|----------|
| **Purpose** | Encapsulate requests | Encapsulate algorithms |
| **Focus** | What to do | How to do it |
| **State** | May store state for undo | Usually stateless |
| **Receiver** | Commands know their receivers | Strategies are independent |
| **Usage** | Actions, requests, transactions | Algorithms, calculations |