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

---

## GameInputEvents (`scripts/input_events.gd`)

```gdscript
class_name GameInputEvents

static var direction : Vector2
	
static func movement_input() -> Vector2:
	if Input.is_action_pressed("walk_left"):
		direction = Vector2.LEFT
	elif Input.is_action_pressed("walk_right"):
		direction = Vector2.RIGHT
	elif Input.is_action_pressed("walk_up"):
		direction = Vector2.UP
	elif Input.is_action_pressed("walk_down"):
		direction = Vector2.DOWN
	else:
		direction = Vector2.ZERO
		
	return direction
	
static func is_movement_input() -> bool:
	if direction == Vector2.ZERO:
		return false
	else:
		return true
```

**What it does:**
A **static helper class** that centralizes input polling. Any state can call `GameInputEvents.movement_input()` instead of repeating raw `Input.is_action_pressed()` checks. Also tracks whether movement was pressed via `is_movement_input()` — used by states to decide whether to transition.

**Key points:**

| Line(s) | What it does | C# Equivalent |
|---------|--------------|---------------|
| `class_name GameInputEvents` | Makes it a globally accessible type (no `preload` needed) | `public static class GameInputEvents` |
| `static var direction` | Shared state — the last direction returned | `private static Vector2 _direction;` |
| `static func movement_input() -> Vector2` | Polls input map and returns direction | `public static Vector2 MovementInput()` |
| `static func is_movement_input() -> bool` | Returns whether movement keys are held | `public static bool IsMovementInput()` |

**Why static?** No need to instance this as a node — it's pure logic. Any script anywhere can call `GameInputEvents.movement_input()` without a node reference.

**C# analogy:** This is like a `public static class InputHelper` with static methods you call from anywhere — no MonoBehaviour, no instantiation.

---

## Player (`scenes/characters/player/player.gd`)

```gdscript
class_name Player

extends CharacterBody2D

var player_direction : Vector2
```

**What it does:**
The root script for the player scene. Creates a `Player` type that states can reference via `@export var player: Player`. Stores the last movement direction so `IdleState` can play the correct idle animation based on where the player was last walking.

**Line by line:**

| Line | What it does | C# Equivalent |
|------|--------------|---------------|
| `class_name Player` | Makes `Player` a globally available type | `public class Player : CharacterBody2D` |
| `var player_direction : Vector2` | Holds the last facing direction. Written by `WalkState`, read by `IdleState` | `public Vector2 PlayerDirection { get; set; }` |

**C# analogy:** A lightweight `MonoBehaviour` (well, `Node`-derived) that exposes a public property for other components to read/write — like a shared blackboard between states.

---

## WalkState (`scenes/characters/player/walk_state.gd`)

```gdscript
extends NodeState

@export var player: Player
@export var animated_player_2d: AnimatedSprite2D
@export var speed: int = 50

func _on_process(_delta : float) -> void:
	pass

func _on_physics_process(_delta : float) -> void:
	var direction: Vector2 = GameInputEvents.movement_input();
	
	if direction == Vector2.UP:
		animated_player_2d.play("walk_back")
	elif direction == Vector2.RIGHT:
		animated_player_2d.play("walk_right")
	elif direction == Vector2.DOWN:
		animated_player_2d.play("walk_front")
	elif direction == Vector2.LEFT:
		animated_player_2d.play("walk_left")
	
	if direction != Vector2.ZERO:
		player.player_direction = direction
	
	player.velocity = direction * speed;
	player.move_and_slide()

func _on_next_transitions() -> void:
	if !GameInputEvents.is_movement_input():
		transition.emit("idle")

func _on_enter() -> void:
	pass

func _on_exit() -> void:
	animated_player_2d.stop()
```

**What it does:**
The walk state handles continuous movement. Each physics frame it reads input, plays the correct walk animation, updates `player_direction` (so idle knows which way to face), sets velocity, and calls `move_and_slide()`. When no movement input is detected, it transitions back to `idle`.

**Line by line:**

| Line | What it does | C# Equivalent |
|------|--------------|---------------|
| `var speed: int = 50` | Movement speed (exported, tweakable in Inspector) | `public int Speed = 50;` |
| `direction = GameInputEvents.movement_input()` | Gets input direction from the static helper | `var direction = GameInputEvents.MovementInput();` |
| `animated_player_2d.play("walk_...")` | Plays correct walk animation for direction | `_sprite.Play("walk_...");` |
| `player.player_direction = direction` | Stores facing direction for idle state | `player.PlayerDirection = direction;` |
| `player.velocity = direction * speed` | Sets CharacterBody2D velocity | `player.Velocity = direction * speed;` |
| `player.move_and_slide()` | Applies movement with collision | `player.MoveAndSlide();` |
| `_on_next_transitions()` | Called every physics tick after `_on_physics_process` — asks "should I switch states?" | `public override void OnNextTransitions()` |
| `transition.emit("idle")` | Fires the signal — state machine picks it up and switches | `Transition?.Invoke("idle");` |
| `animated_player_2d.stop()` | Stops animation on exit — prevents last frame lingering | `_sprite.Stop();` |

**Flow:**
1. Player presses WASD → `IdleState._on_next_transitions()` detects input → emits `transition.emit("Walk")`
2. State machine calls `IdleState._on_exit()` then `WalkState._on_enter()`
3. Every physics frame, WalkState moves the player and plays walk animations
4. Player releases all keys → `WalkState._on_next_transitions()` sees no input → emits `transition.emit("idle")`
5. State machine switches back to IdleState, which plays the correct idle direction

**C# analogy:** A state in a Unity `StateMachine` behaviour — implements `OnEnter()`/`OnExit()`/`Update()` lifecycle.

---

## IdleState (updated — `scenes/characters/player/idle_state.gd`)

```gdscript
extends NodeState

@export var player : Player
@export var animated_sprite_2d :AnimatedSprite2D

var direction : Vector2

func _on_process(_delta : float) -> void:
	pass

func _on_physics_process(_delta : float) -> void:
	if player.player_direction == Vector2.UP:
		animated_sprite_2d.play("idle_back")
	elif player.player_direction == Vector2.RIGHT:
		animated_sprite_2d.play("idle_right")
	elif player.player_direction == Vector2.LEFT:
		animated_sprite_2d.play("idle_left")
	elif player.player_direction == Vector2.DOWN:
		animated_sprite_2d.play("idle_front")
	else :
		animated_sprite_2d.play("idle_front")


func _on_next_transitions() -> void:
	GameInputEvents.movement_input()
	
	if GameInputEvents.is_movement_input():
		transition.emit("Walk")
	

func _on_enter() -> void:
	pass

func _on_exit() -> void:
	animated_sprite_2d.stop()
```

**What changed from original:**
1. **`@export var player: Player`** — now uses the `Player` class instead of raw `CharacterBody2D`
2. **Uses `player.player_direction`** — reads the direction stored by WalkState instead of polling Input directly
3. **`_on_next_transitions()` delegates to `GameInputEvents`** — calls `movement_input()` then `is_movement_input()` to decide if it should transition to Walk
4. **`transition.emit("Walk")`** — capital W (must match state node name exactly — `Walk` not `walk`)
5. **`_on_exit()` stops animation** — prevents the last idle frame from persisting during walk

**C# analogy:** Same pattern as before, but now the state reads from a shared direction variable (like a public property) instead of re-implementing input logic — cleaner separation of concerns.

---

## DataTypes (`scripts/globals/data_types.gd`)

```gdscript
class_name DataTypes
enum Tools {
	None,
	AxeWood,
	TillGround,
	WaterCrops,
	PlantCorn,
	PlantTomato
}
```

**What it does:** A global enum for tool types. The player's `current_tool` property uses this type, so the Inspector shows a dropdown. IdleState checks `player.current_tool == DataTypes.Tools.AxeWood` (etc.) to decide which state to transition to.

**C# analogy:** A standalone `public enum Tools { ... }` in its own file.

---

## GameInputEvents — updated (`scripts/game_input_events.gd`)

New method added:

```gdscript
static func use_tool() -> bool:
	var use_tool_value : bool = Input.is_action_just_pressed("hit")
	return use_tool_value
```

**What it does:** Checks for a single-press of the "hit" action (spacebar/F key). Returns `true` only on the frame the key is pressed (not held). Used by IdleState to trigger tool states.

**Key difference from `movement_input()`:** Uses `is_action_just_pressed` (one-shot) vs `is_action_pressed` (continuous hold).

**C# analogy:** `Input.GetKeyDown(KeyCode.Space)` vs `Input.GetKey(KeyCode.Space)`.

---

## Player — updated (`scenes/characters/player/player.gd`)

```gdscript
class_name Player
extends CharacterBody2D

@export var current_tool: DataTypes.Tools = DataTypes.Tools.None

var player_direction : Vector2
```

**What changed:** Added `@export var current_tool: DataTypes.Tools` — shows as a dropdown in the Inspector. Set to `AxeWood` (= 2) in `player.tscn` for testing.

**C# analogy:** `public Tools CurrentTool { get; set; }` with `[Export]` attribute.

---

## IdleState — updated with tool transitions (`scenes/characters/player/idle_state.gd`)

```gdscript
func _on_next_transitions() -> void:
	GameInputEvents.movement_input()
	
	if GameInputEvents.is_movement_input():
		transition.emit("Walk")
		
	if player.current_tool == DataTypes.Tools.AxeWood && GameInputEvents.use_tool():
		transition.emit("Chopping")
	
	if player.current_tool == DataTypes.Tools.TillGround && GameInputEvents.use_tool():
		transition.emit("Tilling")

	if player.current_tool == DataTypes.Tools.WaterCrops && GameInputEvents.use_tool():
		transition.emit("Watering")
```

**What changed:** Added tool transitions. Each checks `current_tool` against a specific `DataTypes.Tools` value AND requires the "hit" button to be pressed. The tool action only fires from Idle (not while walking) — intentional design.

**C# analogy:**
```csharp
if (CurrentTool == Tools.AxeWood && Input.GetKeyDown(KeyCode.Space))
    Fsm.TransitionTo("Chopping");
```

---

## Tool State Pattern: Chopping/Tilling/Watering

All three tool states are structurally identical. Here's ChoppingState as the representative:

```gdscript
extends NodeState

@export var player : Player
@export var animated_sprite_2d : AnimatedSprite2D

func _on_process(_delta : float) -> void:
	pass

func _on_physics_process(_delta : float) -> void:
	pass

func _on_next_transitions() -> void:
	if !animated_sprite_2d.is_playing():
		transition.emit("Idle")

func _on_enter() -> void:
	if player.player_direction == Vector2.UP:
		animated_sprite_2d.play("chopping_back")
	elif player.player_direction == Vector2.RIGHT:
		animated_sprite_2d.play("chopping_right")
	elif player.player_direction == Vector2.DOWN:
		animated_sprite_2d.play("chopping_front")
	elif player.player_direction == Vector2.LEFT:
		animated_sprite_2d.play("chopping_left")
	else:
		animated_sprite_2d.play("chopping_front")
		
func _on_exit() -> void:
	animated_sprite_2d.stop()
```

**What it does:** Enters → plays the correct direction animation once (loop=0) → auto-transitions back to Idle when the animation finishes.

**Key pattern:**
- `_on_physics_process` is empty — tool actions don't move the player
- `_on_next_transitions` checks `is_playing()` — when the animation finishes, return to Idle
- `_on_enter` uses `if/elif/else` for direction-based animation (fixed from original `if/if/if/else` bug)
- `_on_exit` stops the animation

**The three files are:**
| File | State name | Animation prefix |
|------|-----------|-----------------|
| `chopping_state.gd` | Chopping | `chopping_*` |
| `tilling_state.gd` | Tilling | `tilling_*` |
| `watering_state.gd` | Watering | `watering_*` |

**C# analogy:** A one-shot "ability" state — enters, plays animation, exits when done. Like a `PlayerState` that auto-completes.

---

## HitComponent (`scenes/components/hit_component.gd`)

```gdscript
class_name HitComponent
extends Area2D

@export var current_tool : DataTypes.Tools = DataTypes.Tools.None
@export var hit_damage : int = 1
```

**What it does:**
Pure data component that sits on the **player**. Tells the tree *what tool* hit it and *how much damage* it deals. No signals, no logic — just two exported properties. The collision shape is disabled by default and only enabled during the chopping animation (via ChoppingState).

**Key points:**

| Line | What it does | C# Equivalent |
|------|--------------|---------------|
| `class_name HitComponent` | Globally accessible type | `public class HitComponent : Area2D` |
| `@export var current_tool` | Which tool this hit uses (Inspector dropdown) | `[Export] public Tools CurrentTool;` |
| `@export var hit_damage` | Damage amount per hit | `[Export] public int HitDamage = 1;` |

**Physics layers:** `collision_layer = 8` (Tool), `collision_mask = 16` (Object) — "I exist on the Tool layer, I detect Objects"

**C# analogy:** A plain `MonoBehaviour` with public fields — no logic, just configuration. The "dumb data" half of a component pattern.

---

## HurtComponent (`scenes/components/hurt_component.gd`)

```gdscript
class_name HurtComponent
extends Area2D

@export var tool : DataTypes.Tools = DataTypes.Tools.None

signal hurt

func _on_area_entered(area: Area2D) -> void:
    var hit_component = area as HitComponent
    if tool == hit_component.current_tool:
        hurt.emit(hit_component.hit_damage)
```

**What it does:**
Sits on the **tree** (or any damageable object). When its area overlaps with a HitComponent, it checks: "does my tool type match what hit me?" If yes → emit `hurt` with the damage amount. This is how one component can handle different tool interactions (axe on trees, pickaxe on rocks, etc.).

**Key points:**

| Line | What it does | C# Equivalent |
|------|--------------|---------------|
| `@export var tool` | What tool type this object responds to | `[Export] public Tools Tool;` |
| `signal hurt` | Emitted when hit by matching tool | `[Signal] public delegate void HurtEventHandler(int hitDamage);` |
| `var hit_component = area as HitComponent` | Cast the overlapping area to HitComponent | `var hitComp = area as HitComponent;` |
| `if tool == hit_component.current_tool` | Only react if the tool matches | Guard clause |
| `hurt.emit(hit_component.hit_damage)` | Pass damage amount to listeners | `Hurt?.Invoke(hitDamage);` |

**Physics layers:** `collision_layer = 16` (Object), `collision_mask = 8` (Tool) — mirror opposite of HitComponent

**C# analogy:** A trigger collider that checks tag/type before firing an event. Like `OnTriggerEnter2D` with a type check.

---

## DamageComponent (`scenes/components/damage_component.gd`)

```gdscript
class_name DamageComponent
extends Node2D

@export var max_damage = 1
@export var current_damage = 0

signal max_damage_reached

func apply_damage(damage: int) -> void:
    current_damage = clamp(current_damage + damage, 0, max_damage)
    if current_damage == max_damage:
        max_damage_reached.emit()
```

**What it does:**
Sits on the **tree** (or any destructible). Tracks health with `clamp()` so damage never exceeds max. When health hits max → emit `max_damage_reached`. Generic enough for any object that takes damage (trees, rocks, chests, etc.).

**Key points:**

| Line | What it does | C# Equivalent |
|------|--------------|---------------|
| `@export var max_damage = 1` | Health pool (Inspector-editable) | `[Export] public int MaxDamage = 1;` |
| `@export var current_damage = 0` | Current health | `public int CurrentDamage;` |
| `signal max_damage_reached` | Emitted when health is depleted | `[Signal] public delegate void MaxDamageReachedEventHandler();` |
| `clamp(current_damage + damage, 0, max_damage)` | Add damage but never go below 0 or above max | `Mathf.Clamp(CurrentDamage + damage, 0, MaxDamage)` |
| `if current_damage == max_damage` | Check if fully destroyed | Guard clause |

**C# analogy:** A health component — `TakeDamage(int amount)` with clamping. Same as `Health.cs` in most Unity tutorials.

---

## SmallTree (`scenes/objects/trees/small_tree.gd`)

```gdscript
extends Sprite2D
@onready var damage_component: DamageComponent = $DamageComponent
@onready var hurt_component: HurtComponent = $HurtComponent

func _ready() -> void:
    hurt_component.hurt.connect(on_hurt)
    damage_component.max_damage_reached.connect(on_max_damage_reached)

func on_hurt(hit_damage : int) -> void:
    damage_component.apply_damage(hit_damage)

func on_max_damage_reached() -> void:
    queue_free()
```

**What it does:**
The **glue script** that connects components together via signals. When hurt → pass damage to DamageComponent. When max damage reached → destroy the tree.

**Key points:**

| Line | What it does | C# Equivalent |
|------|--------------|---------------|
| `@onready var damage_component` | Cache reference to child node | `private DamageComponent _damageComp;` → assign in `Start()` |
| `hurt_component.hurt.connect(on_hurt)` | Subscribe to signal | `hurtComponent.Hurt += OnHurt;` |
| `damage_component.apply_damage(hit_damage)` | Forward damage to health tracker | `damageComp.ApplyDamage(hitDamage);` |
| `queue_free()` | Remove node from scene tree | `Destroy(gameObject);` |

**Flow:** `HurtComponent.hurt` → `on_hurt()` → `DamageComponent.apply_damage()` → if max reached → `max_damage_reached` → `on_max_damage_reached()` → `queue_free()`

**C# analogy:** A `MonoBehaviour` that wires up events in `Start()` — like connecting UI buttons to handlers. The "controller" in an MVC-ish pattern.

---

## SmallTree — updated (`scenes/objects/trees/small_tree.gd`)

```gdscript
extends Sprite2D
@onready var damage_component: DamageComponent = $DamageComponent
@onready var hurt_component: HurtComponent = $HurtComponent

var log_scene = preload("res://scenes/objects/trees/log.tscn")

func _ready() -> void:
    hurt_component.hurt.connect(on_hurt)
    damage_component.max_damage_reached.connect(on_max_damage_reached)

func on_hurt(hit_damage : int) -> void:
    damage_component.apply_damage(hit_damage)

func on_max_damage_reached() -> void:
    call_deferred("add_log_scene")
    print("max damage reached")
    queue_free()

func add_log_scene() -> void:
    var log_instance = log_scene.instantiate() as Node2D
    log_instance.global_position = global_position
    get_parent().add_child((log_instance))
```

**What changed from original:**
1. **`var log_scene = preload(...)`** — loads the Log scene at parse time (like `using static import` in C#)
2. **`call_deferred("add_log_scene")`** — schedules `add_log_scene()` for the end of the frame (like `Invoke(() => ...)` in Unity). **Critical:** Can't add children to the scene tree during a physics callback — `call_deferred` defers the add to avoid the "Can't change state while flushing queries" error
3. **`add_log_scene()`** — instantiates the log, sets its position to the tree's position, adds it to the **parent** of the tree (not the tree itself — the tree is about to be freed)
4. **`get_parent().add_child()`** — adds the log to the same node that contained the tree (the trees layer in the tilemap), so the log persists in the world

**Key pattern:** `queue_free()` removes the tree immediately, but `call_deferred` ensures the log spawn happens *after* the current frame's physics processing is done. Without this, you'd get the "Can't change state while flushing queries" error.

**C# analogy:** `call_deferred("method_name")` ≈ `Invoke(() => MethodName(), 0f)` in Unity (runs at end of frame). Or `await Task.Yield()` + `MethodName()`.

---

## LargeTree (`scenes/objects/trees/large_tree.gd`)

```gdscript
extends Sprite2D
@onready var damage_component: DamageComponent = $DamageComponent
@onready var hurt_component: HurtComponent = $HurtComponent

var log_scene = preload("res://scenes/objects/trees/log.tscn")

func _ready() -> void:
    hurt_component.hurt.connect(on_hurt)
    damage_component.max_damage_reached.connect(on_max_damage_reached)

func on_hurt(hit_damage : int) -> void:
    damage_component.apply_damage(hit_damage)

func on_max_damage_reached() -> void:
    call_deferred("add_log_scene")
    print("max damage reached")
    queue_free()

func add_log_scene() -> void:
    var log_instance = log_scene.instantiate() as Node2D
    log_instance.global_position = global_position
    get_parent().add_child((log_instance))
```

**What it does:**
Identical script to SmallTree — same glue pattern. The difference is in the scene (`.tscn`):

| Property | SmallTree | LargeTree |
|----------|-----------|-----------|
| Atlas region | `Rect2(0, 0, 16, 32)` | `Rect2(16, 0, 32, 32)` |
| Position | `(0, -12)` | `(0, -15)` |
| HurtComponent collision | `Rect2(10, 23)` | `Rect2(10, 24)` |
| max_damage | 3 | 5 |
| StaticBody collision | Circle, r=4 | Circle, r=5 |

**Reusability in action:** Same components (HurtComponent, DamageComponent), same script pattern, just different exported values. The LargeTree takes 5 hits to destroy instead of 3.

**C# analogy:** Same `MonoBehaviour` script, different prefab settings — like having `Tree prefab` with `health=3` and `BigTree prefab` with `health=5` using the same `TreeController.cs`.

---

## CollectableComponent (`scenes/components/collectible_component.gd`)

```gdscript
class_name CollectableComponent
extends  Area2D

@export var collectable_name : String

func _on_body_entered(body: Node2D) -> void:
    if body is Player:
        print("Collected")
        get_parent().queue_free()
```

**What it does:**
Sits on the Log (or any dropped item). When a body enters its area, it checks if that body is a `Player`. If yes → prints "Collected" → frees its parent (the log node). The `body_entered` signal is connected via the `.tscn` file.

**Key points:**

| Line | What it does | C# Equivalent |
|------|--------------|---------------|
| `class_name CollectableComponent` | Globally accessible type | `public class CollectableComponent : Area2D` |
| `@export var collectable_name : String` | Name of the item (Inspector-editable) | `[Export] public string CollectableName;` |
| `if body is Player` | Type check — only Player triggers collection | `if (body is Player)` |
| `get_parent().queue_free()` | Destroys the parent node (the log/item) | `GetParent().QueueFree();` |

**Physics layers (from `.tscn`):**
- `collision_layer = 32` (Layer 6 — Collectable layer, not yet defined in project settings)
- `collision_mask = 2` (Layer 2 — Player) — "I detect Player bodies entering me"

**Why `get_parent()` instead of `self`?**
The CollectableComponent is a *child* of the Log node. If you `queue_free()` the component itself, the log sprite stays in the world orphaned. Freeing the parent removes the entire log scene.

**C# analogy:** A trigger collider `OnTriggerEnter2D` that checks tag == "Player" then `Destroy(transform.parent.gameObject)`.

---

## Log (`scenes/objects/trees/log.tscn`)

No script — purely scene-based:
- **Sprite2D** — atlas texture from `basic_grass_biom_things.png`, region `Rect2(80, 32, 16, 16)`
- **CollectableComponent** (instanced) — `collectable_name = "log"`
- **CollisionShape2D** — CircleShape2D, radius=8, child of CollectableComponent

**What it does:**
A pickup item spawned when a tree is destroyed. The player walks over it → CollectableComponent detects body_entered → log is freed.

---

## ChoppingState — updated (`scenes/characters/player/chopping_state.gd`)

```gdscript
extends NodeState

@export var player : Player
@export var animated_sprite_2d : AnimatedSprite2D
@export var hit_component_collision_shape : CollisionShape2D

func _ready() -> void:
    hit_component_collision_shape.disabled = true
    hit_component_collision_shape.position = Vector2(0,0)

func _on_process(_delta : float) -> void:
    pass

func _on_physics_process(_delta : float) -> void:
    pass

func _on_next_transitions() -> void:
    if !animated_sprite_2d.is_playing():
        transition.emit("Idle")

func _on_enter() -> void:
    if player.player_direction == Vector2.UP:
        animated_sprite_2d.play("chopping_back")
        hit_component_collision_shape.position = Vector2(0,-18)
    elif player.player_direction == Vector2.RIGHT:
        animated_sprite_2d.play("chopping_right")
        hit_component_collision_shape.position = Vector2(9,0)
    elif player.player_direction == Vector2.DOWN:
        animated_sprite_2d.play("chopping_front")
        hit_component_collision_shape.position = Vector2(0,3)
    elif player.player_direction == Vector2.LEFT:
        animated_sprite_2d.play("chopping_left")
        hit_component_collision_shape.position = Vector2(-9,0)
    else:
        animated_sprite_2d.play("chopping_front")
    hit_component_collision_shape.disabled = false

func _on_exit() -> void:
    animated_sprite_2d.stop()
    hit_component_collision_shape.disabled = true
```

```gdscript
extends NodeState

@export var player : Player
@export var animated_sprite_2d : AnimatedSprite2D
@export var hit_component_collision_shape : CollisionShape2D

func _ready() -> void:
    hit_component_collision_shape.disabled = true
    hit_component_collision_shape.position = Vector2(0,0)

func _on_enter() -> void:
    if player.player_direction == Vector2.UP:
        animated_sprite_2d.play("chopping_back")
        hit_component_collision_shape.position = Vector2(0,-18)
    elif player.player_direction == Vector2.RIGHT:
        animated_sprite_2d.play("chopping_right")
        hit_component_collision_shape.position = Vector2(9,0)
    elif player.player_direction == Vector2.DOWN:
        animated_sprite_2d.play("chopping_front")
        hit_component_collision_shape.position = Vector2(0,3)
    elif player.player_direction == Vector2.LEFT:
        animated_sprite_2d.play("chopping_left")
        hit_component_collision_shape.position = Vector2(-9,0)
    else:
        animated_sprite_2d.play("chopping_front")
    hit_component_collision_shape.disabled = false

func _on_exit() -> void:
    animated_sprite_2d.stop()
    hit_component_collision_shape.disabled = true
```

**What changed from original:**
1. **`@export var hit_component_collision_shape`** — reference to the HitComponent's CollisionShape2D on the player
2. **`_ready()` disables the shape** — hit detection OFF by default, only active during chop
3. **`_on_enter()` positions the shape per direction** — different Vector2 offsets so the hit box appears in front of the player (up=-18y, right=+9x, down=+3y, left=-9x)
4. **`_on_enter()` enables the shape** — `disabled = false` activates the Area2D overlap
5. **`_on_exit()` disables the shape** — `disabled = true` stops hit detection when leaving chop state

**C# analogy:** Like enabling/disabling a trigger collider during an attack animation — common in action games for frame-accurate hitboxes.
