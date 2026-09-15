# Godot Game Development Activity Guide

A teaching guide for introducing youth to **Godot 4**, building on a
previous MakeCode Arcade activity.

> **Goal:** In about one hour, build the beginnings of a real 2D
> platformer while introducing scenes, nodes, physics, collision,
> GDScript, functions, and signals.

------------------------------------------------------------------------

## What We Are Building

A simple 2D platformer:

-   Move left and right
-   Jump
-   Land on platforms
-   Collect coins
-   Eventually add hazards, score, enemies, a finish line, and
    additional levels

The emphasis is **not** on creating a polished game in one night. The
goal is to show that the programming concepts learned in MakeCode
transfer directly into a full game engine.

------------------------------------------------------------------------

# 1. Create the Godot Project

Create a new Godot 4 project.

Suggested name:

``` text
Deacon Dash
```

For a simple 2D game, the **Compatibility** renderer is a good choice,
especially if the project may eventually run on assorted computers.

Create a **2D Scene** and rename the root node:

``` text
Game
```

Save it as:

``` text
game.tscn
```

Initially:

``` text
Game (Node2D)
```

### Teaching point

A **scene** is a reusable collection of nodes. Our entire level is a
scene, but individual things such as the Player and Coin can also be
their own scenes.

------------------------------------------------------------------------

# 2. Create the Player Scene

Create another scene using:

``` text
CharacterBody2D
```

Rename it:

``` text
Player
```

Save it as:

``` text
player.tscn
```

Add two children:

``` text
Player (CharacterBody2D)
├── Sprite2D
└── CollisionShape2D
```

## Sprite2D

Assign a PNG to the Sprite2D's **Texture** property.

If the image is too large, adjust:

``` text
Sprite2D → Transform → Scale
```

Keep the Player node itself at a scale of `1, 1`.

## CollisionShape2D

Select the CollisionShape2D and choose:

``` text
Shape → New RectangleShape2D
```

Resize the rectangle so that it roughly covers the player's body.

It does **not** need to perfectly trace the artwork. A slightly smaller
collision box often feels better.

### Teaching point

``` text
Sprite2D          = what the player LOOKS like
CollisionShape2D  = where the physics engine thinks the player IS
```

------------------------------------------------------------------------

# 3. Add Player Movement

Select the Player node and attach a script.

Godot 4 can generate the basic CharacterBody2D movement template.

A simplified version is:

``` gdscript
extends CharacterBody2D

const SPEED = 300.0
const JUMP_VELOCITY = -400.0


func _physics_process(delta: float) -> void:
    # Add gravity.
    if not is_on_floor():
        velocity += get_gravity() * delta

    # Jump.
    if Input.is_action_just_pressed("ui_accept") and is_on_floor():
        jump()

    # Left/right movement.
    var direction := Input.get_axis("ui_left", "ui_right")

    if direction:
        velocity.x = direction * SPEED
    else:
        velocity.x = move_toward(velocity.x, 0, SPEED)

    move_and_slide()


func jump():
    velocity.y = JUMP_VELOCITY
```

------------------------------------------------------------------------

# 4. Explain the Player Code

## Constants

``` gdscript
const SPEED = 300.0
const JUMP_VELOCITY = -400.0
```

Instead of scattering numbers throughout the program, we give important
values meaningful names.

Let the students experiment.

Try:

``` gdscript
const SPEED = 1000.0
```

or:

``` gdscript
const JUMP_VELOCITY = -800.0
```

Then run the game and see what happens.

This gives an immediate connection between **changing code** and
**changing game behavior**.

------------------------------------------------------------------------

## The Physics Loop

``` gdscript
func _physics_process(delta: float) -> void:
```

Connect this to MakeCode's **Forever** block.

Godot repeatedly calls `_physics_process()` while the game is running.

`delta` represents the amount of time since the previous physics update
and helps movement/physics behave consistently over time.

------------------------------------------------------------------------

## Gravity

``` gdscript
if not is_on_floor():
    velocity += get_gravity() * delta
```

If the player isn't standing on something, gravity pulls the player
downward.

------------------------------------------------------------------------

## Why Is Jump Negative?

``` gdscript
const JUMP_VELOCITY = -400.0
```

Godot's 2D coordinates work approximately like this:

``` text
             -Y
              ↑
              |
    -X  ←-----+-----→ +X
              |
              ↓
             +Y
```

Therefore negative Y means **up**.

------------------------------------------------------------------------

## Input

``` gdscript
var direction := Input.get_axis("ui_left", "ui_right")
```

This produces roughly:

``` text
Left        Nothing        Right

 -1  ←--------- 0 --------→  1
```

Then:

``` gdscript
velocity.x = direction * SPEED
```

turns that direction into horizontal movement.

------------------------------------------------------------------------

## Moving the Character

``` gdscript
move_and_slide()
```

This tells CharacterBody2D to move according to its velocity while
interacting with collision objects.

------------------------------------------------------------------------

# 5. Introduce a Custom Function

Instead of placing the jump code directly inside `_physics_process()`,
create:

``` gdscript
func jump():
    velocity.y = JUMP_VELOCITY
```

Then call it:

``` gdscript
if Input.is_action_just_pressed("ui_accept") and is_on_floor():
    jump()
```

### Teaching point

A function lets us give a behavior a name.

Instead of repeatedly explaining *how to jump*, other code can simply
say:

``` gdscript
jump()
```

This directly connects to custom functions from MakeCode/Python.

------------------------------------------------------------------------

# 6. Add the Player to the Game

Open:

``` text
game.tscn
```

Instantiate `player.tscn` as a child of Game.

The tree becomes:

``` text
Game (Node2D)
└── Player
    ├── Sprite2D
    └── CollisionShape2D
```

Move the **Player node**, not its Sprite2D child, to the desired
starting position.

For example:

``` text
X: 200
Y: 300
```

### Important

Generally keep the Sprite2D and CollisionShape2D centered around the
Player's local `(0, 0)`.

Move the **Player** when positioning it in the level.

------------------------------------------------------------------------

# 7. Create the Ground

Add:

``` text
StaticBody2D
```

as a child of Game and rename it:

``` text
Ground
```

Add:

``` text
CollisionShape2D
```

under Ground.

Your scene might now look like:

``` text
Game
├── Player
└── Ground (StaticBody2D)
    └── CollisionShape2D
```

For the CollisionShape2D choose:

``` text
New RectangleShape2D
```

Make it a long rectangle underneath the player.

Run the scene.

The player should:

-   fall because of gravity
-   stop on the ground
-   move with left/right
-   jump with Space

At this point you have a functioning platformer.

------------------------------------------------------------------------

# 8. Create Platforms

Create additional StaticBody2D nodes:

``` text
Game
├── Player
├── Ground
├── Platform1 (StaticBody2D)
│   └── CollisionShape2D
├── Platform2 (StaticBody2D)
│   └── CollisionShape2D
└── Platform3 (StaticBody2D)
    └── CollisionShape2D
```

`Ctrl+D` can quickly duplicate platforms.

Move and resize them to create a simple level.

This is a great point to let students customize their own level layouts.

------------------------------------------------------------------------

# 9. Make Platforms Visible

`StaticBody2D` and `CollisionShape2D` are physics objects and do not
render artwork in the running game.

For quick prototype graphics, add a `Polygon2D`:

``` text
Platform (StaticBody2D)
├── Polygon2D
└── CollisionShape2D
```

Draw a rectangle with the Polygon2D tools and choose a color in the
Inspector.

### Teaching point

Again, separate **visuals** from **physics**:

``` text
Polygon2D          = what the platform LOOKS like
CollisionShape2D   = where the platform physically EXISTS
```

The two do not automatically resize together.

A finished game would commonly use sprites or tilemaps for visuals with
collision geometry underneath.

------------------------------------------------------------------------

# 10. Create a Coin

Create a new scene with:

``` text
Area2D
```

as the root.

Rename it:

``` text
Coin
```

Save it as:

``` text
coin.tscn
```

Add:

``` text
Coin (Area2D)
├── Sprite2D
└── CollisionShape2D
```

## Coin Sprite

Assign a coin PNG to the Sprite2D's Texture property.

Resize the **Sprite2D** using its Transform → Scale property if
necessary.

## Coin Collision

For the CollisionShape2D use:

``` text
New CircleShape2D
```

Resize it to approximately match the visible coin.

------------------------------------------------------------------------

# 11. Introduce Area2D

A platform uses `StaticBody2D` because it should physically stop the
player.

A coin uses `Area2D` because we want the player to pass through it while
detecting that the player entered the area.

This makes Area2D useful for:

-   coins
-   power-ups
-   checkpoints
-   finish lines
-   trigger zones
-   lava/hazards
-   doors

------------------------------------------------------------------------

# 12. Introduce Signals

Select the Coin's `Area2D` node.

Open the **Node** panel and locate:

``` text
body_entered(body: Node2D)
```

Connect that signal to the Coin's script.

Godot creates something similar to:

``` gdscript
func _on_body_entered(body: Node2D) -> void:
    pass
```

Change it to:

``` gdscript
func _on_body_entered(body: Node2D) -> void:
    if body.name == "Player":
        queue_free()
```

Now instantiate `coin.tscn` into the Game and place it above a platform.

When the Player touches the coin, the coin disappears.

### Teaching connection to MakeCode

MakeCode:

``` text
When Player overlaps Food
    → do something
```

Godot:

``` text
body_entered signal
    → call a function
```

Explain it as:

> The Coin sends a signal saying, "Something entered me!" The function
> decides what to do about it.

`body_entered` is the **event/signal**.

`_on_body_entered()` is the **function responding to it**.

------------------------------------------------------------------------

# Optional: AnimatedSprite2D

For the one-hour student activity, a static `Sprite2D` is recommended.

Animation can easily consume a large portion of the activity.

For a demo version, however, `Sprite2D` can be replaced by:

``` text
AnimatedSprite2D
```

The Player scene becomes:

``` text
Player (CharacterBody2D)
├── AnimatedSprite2D
└── CollisionShape2D
```

Animations can be started explicitly:

``` gdscript
func _ready() -> void:
    $AnimatedSprite2D.play("idle")
```

Or selected according to player state:

``` gdscript
if not is_on_floor():
    $AnimatedSprite2D.play("jump")
elif direction != 0:
    $AnimatedSprite2D.play("run")
else:
    $AnimatedSprite2D.play("idle")
```

A single right-facing animation can be reused for left movement:

``` gdscript
$AnimatedSprite2D.flip_h = direction < 0
```

For an idle animation, make sure:

-   the animation is playing/autoplaying
-   **Loop** is enabled if appropriate

------------------------------------------------------------------------

# Suggested One-Hour Schedule

  Time         Build             Concepts
  ------------ ----------------- ------------------------------------------
  0--10 min    Player + ground   Nodes, scenes, physics
  10--20 min   Run + jump        GDScript, input, velocity, functions
  20--30 min   Platforms         Collision, level design
  30--40 min   Coins             Area2D, signals/events
  40--50 min   Customize         Level design, speed/jump experimentation
  50--60 min   Play/test         Debugging, iteration, sharing

Do **not** worry if the class doesn't finish everything. A playable
character jumping around a student-designed level is already a
successful activity.

------------------------------------------------------------------------

# Useful MakeCode → Godot Connections

  MakeCode concept          Godot concept
  ------------------------- ---------------------------------
  Sprite                    Sprite2D / CharacterBody2D
  Forever block             `_physics_process()`
  Controller movement       `Input.get_axis()`
  Sprite overlap            Area2D + signals
  Variables                 GDScript variables
  Custom function           GDScript `func`
  Move sprite               `velocity` + `move_and_slide()`
  Wall/platform collision   CollisionShape2D
  Game object               Scene
  Event                     Signal

The key message for the students:

> **These aren't "MakeCode concepts." They're programming concepts. The
> tools and syntax change, but the ideas stay the same.**

------------------------------------------------------------------------

# Good Teaching Experiments

Let the students intentionally modify values and predict the result
before running the game.

### Super speed

``` gdscript
const SPEED = 1000.0
```

Ask:

> What do you think will happen?

### Super jump

``` gdscript
const JUMP_VELOCITY = -800.0
```

Ask why the value is negative.

### Remove the floor check

Temporarily change:

``` gdscript
if Input.is_action_just_pressed("ui_accept") and is_on_floor():
```

to:

``` gdscript
if Input.is_action_just_pressed("ui_accept"):
```

Now the player can jump in midair.

Ask:

> Why did removing one condition create infinite jumping?

Then restore it.

These experiments help students understand the code rather than simply
copying it.

------------------------------------------------------------------------

# Where to Go Next

If this becomes a longer project, possible future features include:

-   Score counter/UI
-   Hazards or lava
-   Lives/health
-   Respawning
-   Checkpoints
-   Moving platforms
-   Enemies
-   Enemy AI
-   Multiple collectible types
-   Power-ups
-   Sound effects
-   Music
-   Animated characters
-   Camera following
-   Larger scrolling levels
-   Tilemaps
-   Multiple levels
-   Main menu
-   Pause menu
-   Win/lose screens
-   Save games
-   Boss fight
-   Gamepad support
-   Particles and visual effects
-   Credits

------------------------------------------------------------------------

# Possible Year-Long Project

Rather than treating this as a weekly programming lesson, treat the
group like a small **game studio**.

A possible progression:

### Phase 1 --- Design

Let the students decide:

-   What kind of game are we making?
-   Who is the player?
-   What is the objective?
-   What enemies exist?
-   What makes our game unique?
-   What should it look and sound like?

Set reasonable constraints:

-   2D
-   appropriate content
-   achievable mechanics
-   something that can grow gradually

### Phase 2 --- Core Game

Build:

-   player controller
-   physics
-   level system
-   collectibles
-   enemies
-   health
-   scoring
-   checkpoints

### Phase 3 --- Content

Create:

-   levels
-   enemy varieties
-   artwork
-   sound
-   music
-   story
-   secrets
-   power-ups

### Phase 4 --- Software Development

Gradually introduce:

-   Git/GitHub
-   source control
-   commits
-   branches
-   bugs/issues
-   feature requests
-   code review
-   reusable components
-   debugging
-   project backlog / Kanban
-   releases

### Phase 5 --- Polish and Ship

Finish with:

-   playtesting
-   bug fixing
-   balancing
-   menus
-   credits
-   export builds
-   final release

Put every contributor's name in the game's credits.

A final activity could be a **game launch night** where everyone plays
the completed game.

------------------------------------------------------------------------

# Instructor Notes

Keep the emphasis on experimentation.

Avoid spending too much time making artwork perfect during the early
activities. Placeholder rectangles, simple PNGs, and basic colors are
completely fine.

A useful teaching pattern is:

1.  Explain the idea.
2.  Predict what the code will do.
3.  Write/change a small amount of code.
4.  Run it immediately.
5.  Intentionally break something.
6.  Figure out why it broke.
7.  Fix it.
8.  Let students customize it.

The most important outcome isn't the platformer.

It's getting students to realize:

> **"I can make software."**
