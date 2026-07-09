# Scripts Notes — Code Explained

One entry per script. GDScript + C# comparisons for reference.

---

## NodeState (`scripts/state_machine/node_state.gd`)

```gdscript
class_name NodeState
extends Node

@warning_ignore("unused_signal")
signal transition 

func _on_process(_delta : float) -> void:
    pass

func _on_physics_process(_delta : float) -> void:
    pass

func _on_next_transitions() -> void:
    pass

func _on_enter() -> void:
    pass

func _on_exit() -> void:
    pass
```

**What it does:**
This is a **base class** for all states (idle, walk, chop, etc.). It defines the "shape" every state must follow — think of it like an **interface** or **abstract class** in C#. Each state can override these methods to define its own behavior.

| GDScript | C# Equivalent |
|----------|---------------|
| `class_name NodeState` | Makes it a globally accessible type — like a `public class` in its own file |
| `extends Node` | `class NodeState : Node` |
| `signal transition` | `[Signal] public delegate void TransitionEventHandler();` + `public event TransitionEventHandler Transition;` |
| `func _on_process(delta): pass` | `public virtual void OnProcess(float delta) { }` — virtual method to override |
| `_on_enter()` / `_on_exit()` | Called when entering/exiting a state — like `OnStateEnter()` / `OnStateExit()` |

**Key insight:** In C# you'd make this an `abstract class` and force overriding. Here it's just virtual methods with `pass` (no-op). The child states like `IdleState`, `WalkState` will `extends NodeState` and override only what they need.

---

## NodeStateMachine (`scripts/state_machine/node_state_machine.gd`)

```gdscript
class_name NodeStateMachine
extends Node

@export var initial_node_state : NodeState

var node_states : Dictionary = {}

var current_node_state : NodeState
var current_node_state_name : String

func _ready() -> void:
    for child in get_children():
        if child is NodeState:
            node_states[child.name.to_lower()] = child
            child.transition.connect(transition_to)
    if initial_node_state:
        initial_node_state._on_enter()
        current_node_state = initial_node_state

func _process(delta : float) -> void:
    if current_node_state:
        current_node_state._on_process(delta)

func _physics_process(delta: float) -> void:
    if current_node_state:
        current_node_state._on_physics_process(delta)
        current_node_state._on_next_transitions()

func transition_to(node_state_name : String) -> void:
    if node_state_name == current_node_state.name.to_lower():
        return
    var new_node_state = node_states.get(node_state_name.to_lower())
    if !new_node_state:
        return
    if current_node_state:
        current_node_state._on_exit()
    new_node_state._on_enter()
    current_node_state = new_node_state
    current_node_state_name = current_node_state.name.to_lower()
    print("Current State: ", current_node_state_name)
```

**What it does:**
This is the **controller** that manages all states. It holds a dictionary of states, tracks which one is active, and delegates `_process` / `_physics_process` calls to the current state. It also listens for `transition` signals from states to switch between them.

**Line by line:**

| Line(s) | What it does | C# Equivalent |
|---------|--------------|---------------|
| `@export var initial_node_state : NodeState` | Shows up in the Inspector so you can drag-drop the starting state | `[Export] public NodeState InitialNodeState;` |
| `var node_states : Dictionary = {}` | Stores all child states, keyed by name (lowercase) | `private Dictionary<string, NodeState> _nodeStates = new();` |
| `for child in get_children():` | Loops through all nodes under this one in the scene tree | `foreach (Node child in GetChildren())` |
| `if child is NodeState:` | Checks if the child is a state (filters out non-state nodes) | `if (child is NodeState state)` |
| `child.transition.connect(transition_to)` | Subscribes to the state's `transition` signal — like `+=` in C# | `state.Transition += TransitionTo;` |
| `func _process(delta)` | Called every frame — delegates to current state | `public override void _Process(double delta)` |
| `func _physics_process(delta)` | Called at fixed rate (60fps) — for physics stuff | `public override void _PhysicsProcess(double delta)` |
| `current_node_state._on_next_transitions()` | Checks if the current state wants to switch to a new state this frame | Called in `_PhysicsProcess` after the state's own physics runs |
| `func transition_to(...)` | The core method — switches from one state to another | `public void TransitionTo(string stateName)` |
| `if ... == current_node_state.name.to_lower(): return` | Guard: don't switch to the state we're already in | Early return |
| `var new_node_state = node_states.get(...)` | Looks up the target state by name | `_nodeStates.TryGetValue(stateName, out var newState)` |
| `current_node_state._on_exit()` | Tells the old state "you're done" | `_currentState.OnExit();` |
| `new_node_state._on_enter()` | Tells the new state "you're active now" | `newState.OnEnter();` |
| `print(...)` | Debug log to the Output panel | `GD.Print(...)` or `Console.WriteLine(...)` |

**The flow:**
1. When the game starts, `_ready()` collects all child states into a dictionary
2. It enters the `initial_node_state`
3. Every frame, it calls the current state's `_on_process(delta)`
4. Every physics tick, it calls `_on_physics_process(delta)` then checks if the state wants to transition
5. When a state fires `transition.emit("walk")`, the machine calls `transition_to("walk")`
6. `transition_to` calls `_on_exit()` on the old state, then `_on_enter()` on the new one

**C# analogy:** This is a **Game State Machine** pattern. In Unity you'd use `IState` interfaces and a `StateMachine` class. Same concept, different syntax.
