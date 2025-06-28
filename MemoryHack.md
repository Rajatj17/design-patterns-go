## 🧠 Memory Hacks for Design Patterns

### **The "Creation Club" (Creational Patterns)**
Think of a **restaurant kitchen**:
- **Singleton** = "Single Chef" (only one head chef in kitchen)
- **Factory** = "Order Kitchen" (kitchen creates different dishes based on orders)
- **Builder** = "Recipe Steps" (step-by-step meal preparation)

**Memory trick**: "*Single chef takes Factory orders using Recipe steps*"

### **The "Structure Squad" (Structural Patterns)**
Think of **building architecture**:
- **Adapter** = "Power Adapter" (connects incompatible plugs)
- **Decorator** = "Christmas Tree" (add ornaments without changing tree)
- **Proxy** = "Security Guard" (controls access to VIP area)

**Memory trick**: "*Adapter plugs Decorate the building, Proxy guards entry*"

### **The "Behavior Brigade" (Behavioral Patterns)**
Think of **office dynamics**:
- **Observer** = "Group Chat" (everyone gets notified of updates)
- **Strategy** = "Multiple Routes" (GPS gives different route options)
- **Command** = "Remote Control" (buttons execute different actions)
- **State** = "Mood Ring" (behavior changes based on internal state)

**Memory trick**: "*Group chat shares Strategy routes via Remote commands based on Mood*"

## 🎯 Quick Recognition Shortcuts

### **Problem-to-Pattern Map**:
```
"How do I create?" → Factory family
"Only one instance?" → Singleton
"Need to notify many?" → Observer
"Multiple algorithms?" → Strategy
"Incompatible interfaces?" → Adapter
"Add features dynamically?" → Decorator
"Control access?" → Proxy
"Undo operations?" → Command
```

### **Code Smell Shortcuts**:
- See `new` everywhere? → **Factory**
- See `if-else` chains for algorithms? → **Strategy**
- See lots of `instanceof` checks? → **Visitor** or **Strategy**
- See complex conditional state logic? → **State**
- See global variables? → **Singleton** (maybe)

## 📝 Interview Cheat Sheet Format

### **The WWWH Framework** for any pattern:
- **What**: One-line definition
- **When**: Classic use case
- **Why**: Main benefit
- **How**: Code structure (interface + implementation)

Example for **Factory**:
- **What**: Creates objects without specifying exact class
- **When**: Multiple product types, creation logic varies
- **Why**: Loose coupling, easy to extend
- **How**: Factory interface + concrete factories + products

## 🎪 Story-Based Memory Tricks

### **The Singleton Story**: "The Highlander"
*"There can be only one!"* - Like the movie Highlander, Singleton ensures only one instance exists. Just like there's only one "chosen one."

### **The Observer Story**: "YouTube Notifications"
When your favorite YouTuber posts, all subscribers get notified. Observer = YouTube's notification system.

### **The Factory Story**: "Restaurant Menu"
You order "pasta" from menu, kitchen decides if it's spaghetti or lasagna. You don't care how it's made, just that you get pasta.

### **The Strategy Story**: "Navigation Apps"
Google Maps can take you home via highway, city streets, or scenic route. Same destination, different strategies.

## 🚀 Go-Specific Memory Aids

### **Go Pattern Signatures**:
```go
// Singleton signature
var instance *Thing
func GetInstance() *Thing { /* sync.Once */ }

// Factory signature  
func CreateThing(thingType string) Thing { /* switch */ }

// Observer signature
type Observer interface { Update() }
func (s *Subject) Notify() { /* range observers */ }

// Strategy signature
type Strategy interface { Execute() }
func (c *Context) SetStrategy(s Strategy) { /* assign */ }
```

### **Go Interface Hint**:
If you see an interface ending in `-er` (Logger, Writer, Reader), it's probably part of a pattern:
- `Logger` → Factory pattern
- `Observer` → Observer pattern  
- `Handler` → Chain of Responsibility
- `Visitor` → Visitor pattern

## 🎯 Interview Day Quick Reference

### **30-Second Pattern Pitch** (memorize these):
1. **Singleton**: "One instance, global access, thread-safe with sync.Once"
2. **Factory**: "Create objects without knowing exact type, switch statement"
3. **Observer**: "One-to-many notification, slice of observers"
4. **Strategy**: "Swap algorithms at runtime, interface with implementations"

### **Red Flag Answers to Avoid**:
- "Singleton is always good" → No, mention DI alternatives
- "Factory for everything" → No, mention when NOT to use
- "Patterns solve all problems" → No, mention when simple code is better

### **Green Flag Phrases**:
- "It depends on the use case..."
- "The trade-off is flexibility vs complexity..."
- "In Go, we'd typically use interfaces..."
- "For testing, I'd inject dependencies..."