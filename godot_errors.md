# Godot Errors & Debugging Reference

Comprehensive reference for troubleshooting Godot 4 games. Written for a C# developer learning GDScript.

---

## 1. How to Read Godot Error Messages

When an error occurs, Godot prints to the **Output panel** (bottom of editor) and the **Debugger > Errors tab**.

```
ERROR: Invalid get index 'health' (on base: 'null instance').
  at: GDScript::_process (res://scripts/enemy.gd:42)
  at: GDScript::_on_hit (res://scripts/enemy.gd:78)
  at: GDScript::_on_body_entered (res://scripts/player.gd:55)
```

| Part | Meaning |
|------|---------|
| `ERROR:` / `WARNING:` | Severity |
| `Invalid get index 'health'` | What went wrong |
| `on base: 'null instance'` | The object type (null = missing reference) |
| `enemy.gd:42` | File & line number where error originated |
| Stack frames below | Call chain — **top is most recent**, bottom is oldest |

**C# analogy:** This is like a `NullReferenceException` with a full stack trace. The error message is the exception type+message, and the stack frames show the call chain.

**Where to look first:**
1. The **top frame** of the stack is where the error actually happened
2. The **error message** tells you what operation failed
3. If the base is `null instance` → you're calling something on a null reference

---

## 2. Print / Log Methods Reference

| Method | Where it shows | Use Case | C# Analogy |
|--------|---------------|----------|------------|
| `print("msg")` | Output panel | General logging | `Console.WriteLine()` |
| `printt("a", "b")` | Output panel (tab-separated) | Tabular data | — |
| `prints("a", "b")` | Output panel (space-separated) | Concatenated args | — |
| `printraw("msg")` | stdout only (NOT in editor panel) | Raw output | — |
| `print_debug("msg")` | Output panel (only in debug builds) | Debug-only logs | `#if DEBUG` |
| `print_verbose("msg")` | Only if `--verbose` flag is on | Hidden diagnostic logs | — |
| `printerr("msg")` | stderr (red in Output) | Errors | `Console.Error.WriteLine()` |
| `push_error("msg")` | Debugger > Errors tab (red, with stack trace) | **Preferred for errors** | `Log.Error()` |
| `push_warning("msg")` | Debugger > Errors tab (yellow, with stack trace) | **Preferred for warnings** | `Log.Warning()` |
| `print_rich("[b]bold[/b]")` | Output panel (BBCode formatting) | Formatted logs | — |
| `assert(condition)` | Halts if false (debug only) | Contract checks | `Debug.Assert()` |
| `breakpoint` | Pauses debugger at that line | Breakpoint in code | `Debugger.Break()` |

### Quick tips
- `push_error()` shows in both Output AND Debugger Errors tab with full stack trace
- `print_verbose()` is great for leaving diagnostic logs you can toggle later
- `assert()` is for "this should NEVER happen" checks during development
- `breakpoint` keyword is persistent even across machines (unlike gutter click breakpoints)

---

## 3. Error Catalog

### 3.1 Null Instance Errors

**Common messages:**
```
ERROR: Invalid get index 'X' (on base: 'null instance').
ERROR: Attempt to call function 'play' in base 'null instance'.
ERROR: Invalid assignment of property or key 'position' with value of type 'Vector2' on a base object of type 'null instance'.
```

**C# analogy:** `NullReferenceException: Object reference not set to an instance of an object.`

**Causes (most common first):**

| Cause | Explanation | Fix |
|-------|-------------|-----|
| Typo in node name | `$Player` vs `$player` — case-sensitive | Copy exact name from Scene dock |
| Accessing node before it exists | `var x = $Node` runs in `_init()` before tree ready | Use `@onready var x = $Node` |
| Wrong path | `$Enemy` (direct child) but node is `$Enemies/Enemy` | Use full path or `%UniqueName` |
| Using type instead of name | `$Timer` but node is named `CooldownTimer` | Match the actual node name |
| `%` without Unique Name | Using `%HealthBar` but node not marked as unique | Right-click → "Access as Scene Unique Name" |
| Freed node | Called `queue_free()` then accessed it | Check `is_instance_valid(target)` before use |

**Prevention patterns:**
```gdscript
# BAD — runs before scene tree is ready
var health_bar = $UI/HealthBar

# GOOD — waits for _ready
@onready var health_bar = $UI/HealthBar

# SAFE CHECK — before using a potentially freed node
if is_instance_valid(some_node):
    some_node.do_thing()

# GODOT 4 SAFE CALL — skips if null (returns null)
var result = node?.get_child(0)
```

### 3.2 Parser / Syntax Errors

```
Parse Error: Expected ':' after identifier
Parse Error: Invalid indentation
Parse Error: Unexpected token
Parse Error: Expected expression, found end of file
```

| Error | Likely Cause |
|-------|-------------|
| `Expected ':'` | Missing colon after `if`, `for`, `func`, `class`, `else`, `elif`, `match` |
| `Invalid indentation` | Mixed tabs and spaces, or wrong indent level |
| `Unexpected token` | Type hint syntax wrong — correct: `var hp: int` / `func get_hp() -> int:` |
| `Unexpected "Literal"` | Number or string literal at class body level (like we had with the 429 error text) |
| `Identifier not found` | Typo in variable/function name, or using reserved word |
| `Mismatched bracket` | Unclosed `()`, `[]`, or `{}` |
| `Expected expression` | Missing argument in function call or expression |

**GDScript type hint syntax (Godot 4):**
```gdscript
var hp: int = 100                    # variable type
func take_damage(amount: int) -> void:  # parameter → return type
```

**C# analogy for syntax errors:**
- Missing `:` → missing `{` after if/for/function
- Indentation errors → the same as mismatched braces in C#, but visually different
- Type hint syntax wrong → `amount: int` vs C# `int amount`

### 3.3 Signal Connection Failures

```
ERROR: Nonexistent function 'on_button_pressed' in base 'Node'.
ERROR: Signal 'action' is already connected.
```

| Cause | Explanation |
|-------|-------------|
| Method name typo | The connected method doesn't exist on the target node |
| Wrong target | Connected to wrong node that doesn't have the method |
| Double connection | Connected the same signal twice (without checking `is_connected()`) |
| Node freed | Signal fires after target node was freed |

**Fix pattern:**
```gdscript
# Check before connecting to avoid double connections
if not button.pressed.is_connected(_on_button_pressed):
    button.pressed.connect(_on_button_pressed)

# Disconnect when no longer needed
button.pressed.disconnect(_on_button_pressed)
```

**C# analogy:** Like subscribing to a C# event with `+=` — if you subscribe twice, the handler fires twice.

### 3.4 Node Not Found

```
ERROR: Node not found: "SomeName" (no global scope).
```

| Cause | Fix |
|-------|-----|
| Path is wrong (not a direct child) | Use full relative path: `$Parent/Child` |
| Node renamed in editor | Update all script references (Ctrl+Shift+F to find old name) |
| Using `$` outside the node's tree | Only use `$` when the script is attached to a node in the tree |
| Scene not instanced yet | Access in `_ready()` not `_init()` |

### 3.5 Type Mismatch / Invalid Operand

```
ERROR: Invalid operands 'String' and 'int' in operator '+'.
ERROR: Invalid type in function 'play' in base 'Nil'.
```

**Causes:**
- Trying to add a string and number without conversion
- Calling a method on a variable that's the wrong type
- Accessing an index on something that isn't an array/dictionary

**Fix:**
```gdscript
# Convert string to int before arithmetic
var total = int("5") + 10

# Add type hints to catch these early
var score: int = 0
```

### 3.6 Index Out of Range

```
ERROR: Index 5 out of range (size 3).
```

**C# analogy:** `IndexOutOfRangeException`

**Causes:**
- Accessing `array[5]` when array only has 3 elements
- Forgetting arrays are 0-indexed (first element is `[0]`)
- String index beyond its length

**Fix:**
```gdscript
# Always check bounds
if index < array.size():
    var item = array[index]

# Safe get with default
var item = array.get(index, default_value)
```

### 3.7 Cyclic Reference

```
ERROR: Cyclic reference detected.
Socket Error 54 (on Mac/Linux)
```

**Cause:** Two scenes trying to `preload()` each other. Scene A preloads Scene B, Scene B preloads Scene A — infinite loop.

**Fix:** Replace one `preload` with a runtime `load()` or use a string path:
```gdscript
# BAD - cyclic
const SceneB = preload("res://scene_b.tscn")

# GOOD
@onready var SceneB = load("res://scene_b.tscn")
```

### 3.8 Can't change state while flushing queries

```
ERROR: Can't change state while flushing queries.
```

**Cause:** Modifying the scene tree (adding/removing nodes) inside a `_physics_process()` callback during a collision/area callback.

**Fix:** Use `call_deferred()` to defer tree changes:
```gdscript
func _on_body_entered(body):
    call_deferred("_remove_enemy", body)

func _remove_enemy(body):
    body.queue_free()
```

---

## 4. Debugging Tools

### 4.1 Breakpoints

| Method | How | Persistence |
|--------|-----|-------------|
| Click gutter (red dot) | Click line number area in script editor | Persists across editor restarts |
| `breakpoint` keyword | Write `breakpoint` in code | Persists across machines (version control) |

**When paused, you can:**
- Inspect **Stack Trace** — what called what
- View **Local Variables** — current function's variables
- View **Member Variables** — the object's properties
- Use **REPL** — evaluate expressions like `direction`, `player.position`
- **Step Over (F10)** — next line, skips into functions
- **Step Into (F11)** — goes into function calls

**Conditional breakpoint pattern:**
```gdscript
if current_state.name == "hurt":
    breakpoint  # only pauses when in hurt state
```

### 4.2 Debug Menu Options (under Debug menu at top of editor)

| Option | What it does |
|--------|-------------|
| **Visible Collision Shapes** | See collision polygons at runtime |
| **Visible Navigation** | See navigation meshes/polygons |
| **Visible Paths** | See Curve2D/3D paths |
| **Debug CanvasItem Redraws** | Flash when nodes redraw |
| **Synchronize Scene Changes** | Live-edit scenes while running |
| **Synchronize Script Changes** | Live-edit scripts while running |

### 4.3 Inspecting Live Nodes (Remote)

When the game is running, the Scene dock has a "Remote" toggle. Switch from "Local" to **"Remote"** to see the live scene tree and inspect/modify node properties in real time.

### 4.4 Profiler (Debugger > Profiler tab)

| Tab | What it shows |
|-----|---------------|
| **Profiler** | Per-function execution time — find slow scripts |
| **Monitors** | Live graphs: FPS, memory, draw calls, physics steps, nodes |
| **Visual Profiler** | Per-frame renderer cost breakdown |

**How to use the Profiler:**
1. Run the game
2. Switch to editor, open **Debugger > Profiler**
3. Click **Start**
4. Reproduce the slow scenario
5. Click **Stop**
6. Sort by **Self Time** (not Total Time) to find YOUR slow functions

**Custom Monitors** — track game-specific metrics:
```gdscript
# In _ready():
Performance.add_custom_monitor("game/enemies", _count_enemies)

func _count_enemies() -> int:
    return get_tree().get_nodes_in_group("enemies").size()
```
Shows up in **Debugger > Monitors** under "Game" category.

### 4.5 Remote Debugging (for exported builds)

1. Enable **Remote Debug** in Export preset
2. Check **Debug > Deploy with Remote Debug**
3. Run the exported game — it will connect back to the editor
4. Use breakpoints and inspect live nodes as if running locally

### 4.6 File Logging

Enable in **Project Settings > Debug > File Logging**:
```
user://logs/godot.log
```
Log files are rotated (up to 5 files kept by default). Even crash backtraces get written here.

---

## 5. Input Troubleshooting Checklist

When `Input.is_action_pressed("xyz")` always returns false:

**Step 1 — Does the action exist?**
```gdscript
print(InputMap.has_action("walk_up"))  # if false → action doesn't exist
```
If false, go to **Project Settings > Input Map** and define it.

**Step 2 — Check the name exactly (case-sensitive)**
- Code: `"walk_up"` → Input Map: `walk_up` ✅
- Code: `"WalkUp"` → Input Map: `walk_up` ❌ (won't match)

**Step 3 — Does the action have keys bound?**
In Input Map, click the action and verify at least one key/button is assigned.

**Step 4 — Are you using the right method?**
| Use case | Method | C# Analogy |
|----------|--------|------------|
| Held down (continuous) | `Input.is_action_pressed("action")` | `Input.GetKey(KeyCode.W)` |
| Just pressed (one shot) | `Input.is_action_just_pressed("action")` | `Input.GetKeyDown(KeyCode.W)` |
| Just released | `Input.is_action_just_released("action")` | `Input.GetKeyUp(KeyCode.W)` |

**Step 5 — Is a UI node eating the input?**
- A focused Control (Button, TextEdit) consumes input before gameplay
- Set any full-screen Control's **Mouse > Filter** to **Ignore**
- Use `_unhandled_input()` instead of `_input()` for gameplay (fires after UI is done)

**Step 6 — Is the node processing inputs?**
- Node must be in the scene tree
- `process_mode` must not be `PROCESS_MODE_DISABLED` or `PROCESS_MODE_PAUSE_ONLY` while paused
- If using `_input()` or `_unhandled_input()`, the node must have the callback defined

**Step 7 — Check dead zones**
- For analog sticks: default dead zone is 0.5 — adjust if the stick isn't registering
- `Input.set_joy_axis_deadzone(0.3)` to lower globally

---

## 6. Null Safety Patterns

| Pattern | When to use | Example |
|---------|-------------|---------|
| `@onready` | Accessing scene children | `@onready var sprite = $AnimatedSprite2D` |
| `is_instance_valid()` | Before using a potentially freed node | `if is_instance_valid(target): target.hit()` |
| `?.` safe call (Godot 4) | Chaining calls that might be null | `node?.get_child(0)?.queue_free()` |
| `is_inside_tree()` | Checking if a node is still in the tree | `if node.is_inside_tree():` |
| `@export` + Inspector | Assign from editor — always non-null if assigned | `@export var player: CharacterBody2D` |
| `@export var` with `@onready var` | For node refs that must survive renames | Just use `@onready var` + Unique Name |

**C# analogy:**
- `@onready` = waiting until `Awake()`/`Start()` to initialize references
- `is_instance_valid()` = checking `obj != null && !obj.IsDestroyed()`
- `?.` = the same null-conditional operator in C#

---

## 7. Performance Profiling Quick Guide

### Common bottlenecks in GDScript (from most to least likely):

| Bottleneck | Symptom | Fix |
|------------|---------|-----|
| Untyped variables | Slow loops | Add types: `var hp: int = 100` |
| `get_node()` / `$` in hot loop | Slow `_process()` | Cache in `@onready var` |
| `get_tree().get_nodes_in_group()` per frame | Lag when many nodes | Cache the result once |
| String concat in `_process()` | GC pressure, micro-stutters | Use `"%d %d" % [a, b]` |
| Signals connected twice | Double-firing callbacks | Check `is_connected()` first |
| Unthrottled `_process()` logic | High CPU usage | Move to `_physics_process` or use timers |

### Profiling workflow:
1. Run game, start Profiler (Debugger > Profiler > Start)
2. Reproduce the issue
3. Stop Profiler
4. Sort by **Self Time** → the function at top is YOUR code that takes longest
5. Fix the bottleneck, repeat

---

## 8. Exported Game Debugging

| Issue | Fix |
|-------|-----|
| "Works in editor, not in export" | Enable **File Logging** (`debug/file_logging/enable_file_logging`) |
| Missing assets | Check non-resource file filter in Export dialog (add `*.json`, `*.txt`, etc.) |
| Game crashes silently | Enable `GDScript > Always Track Call Stacks` in project settings |
| Export fails | Install export templates from Godot menu → Editor → Manage Export Templates |
| Controller not working | Windows supports only 4 controllers via XInput |

### Crash log location:
```
user://logs/godot.log
```
Search for `Program crashed with signal` in the file.

---

## 9. Quick Reference Tables

### Error → Most Likely Cause → Fix

| Error Message | Most Likely Cause | Quick Fix |
|---------------|-------------------|-----------|
| `null instance` | Node path wrong or `@onready` missing | Use `@onready var` + check path spelling |
| `Node not found` | Path doesn't match Scene dock | Copy exact path from Scene dock |
| `Nonexistent function` | Signal connected to wrong method name | Check spelling of connected method |
| `Expected ':'` | Missing colon after if/func/for/etc. | Add `:` at end of line |
| `Invalid indentation` | Mixed tabs/spaces | Use **only tabs** in GDScript |
| `Unexpected token` | Type hint syntax wrong | Use `var name: Type` not `Type name` |
| `Index out of range` | Off-by-one or empty array | Check `size()` before indexing |
| `Cyclic reference` | Two scripts preload each other | Replace one with `load()` |
| `Can't change state while flushing queries` | Modifying tree inside physics callback | Use `call_deferred()` |
| `Input.is_action_pressed()` always false | Action name typo or not defined | Check Input Map + action name spelling |

### ASCII value reference for common keys:
- W = 87, Up arrow = 4194320
- A = 65, Left arrow = 4194319
- S = 83, Down arrow = 4194322
- D = 68, Right arrow = 4194321
- Space = 32
- Enter = 4194306
- Escape = 4194305

### GDScript vs C# quick reference:

| Concept | GDScript | C# |
|---------|----------|----|
| Node reference with delay | `@onready var x = $Path` | `private Node x;` → assign in `_Ready()` |
| Event / Signal | `signal something` / `emit()` | `event Action Something` / `Something?.Invoke()` |
| Subscribe to signal | `node.signal.connect(callable)` | `node.Something += Handler` |
| Unsubscribe | `node.signal.disconnect(callable)` | `node.Something -= Handler` |
| Null check | `is_instance_valid(x)` | `x != null && !x.IsDestroyed()` |
| Safe navigation | `x?.property` | `x?.Property` |
| Dictionary | `var d = {}` / `d.get(k, default)` | `var d = new Dictionary<TKey,TValue>()` |
| Type hint | `var x: int = 5` | `int x = 5` |
| Function | `func do(x: int) -> bool:` | `bool Do(int x)` |
| Print | `print("msg")` | `Console.WriteLine("msg")` |
| Error log | `push_error("msg")` | `Debug.LogError("msg")` |
| Assert | `assert(condition)` | `Debug.Assert(condition)` |
| Frame update | `_process(delta)` | `Update()` |
| Fixed update | `_physics_process(delta)` | `FixedUpdate()` |
| Pause debugger | `breakpoint` | `Debugger.Break()` |

---

*Keep adding to this as you encounter new errors. This is your personal debugging dataset.*
