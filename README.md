# hints-from-gamedev
Solutions to the problems I have encountered

*Sorry my bad english*

### character falling effect
- game: [Crazy Snowflakes](https://github.com/mazilimka/Crazy-Snowflakes)
- Godot: v4.3.stable.official [77dcf97d8]
```gdscript
extends CharacterBody2D
var is_first_touch := true # to avoid falling at the start of the game
var was_on_floor := false

func _physics_process(delta):
  move_and_slide()
	if not is_on_floor():
		velocity.y += gravity * delta
		was_on_floor = true
	else:
		velocity.y = 0.0
		was_on_floor = false
	var collision := move_and_collide(Vector2.DOWN, false)
	if collision and was_on_floor:
		if collision.get_collider().name == "DownWall":
			if not is_first_touch:
				launch_particles()
			is_first_touch = false
	was_on_floor = is_on_floor()
```
*Note:* `move_and_slide` **before** `move_and_collide`

---

### camera shake tween
- thanks [Single-Minded Ryan](https://youtube.com/@single-mindedryan?si=ZSO7ywCCIFKe9_9s)
- game: [Crazy Snowflakes](https://github.com/mazilimka/Crazy-Snowflakes)
- Godot: v4.3.stable.official [77dcf97d8]
```gdscript
# Player:
extends CharacterBody2D
var health := 3:
	set(value):
		if health != value:
			health = value
			if health == 0:
				death()
				return
			Global.Main.update_health(health)
			camera_shake()
var camera_shake_noice: FastNoiseLite

func _ready():
  	camera_shake_noice = FastNoiseLite.new()

func camera_shake(intensity: float = 20.0, time: float = 0.2) -> Tween:
	var camera_tween := create_tween()
	camera_tween.tween_method(start_camera_shake, intensity, 1.0, 0.2)
	return camera_tween

func start_camera_shake(intensity: float):
	var camera_offset := camera_shake_noice.get_noise_1d(Time.get_ticks_usec())
	shake_camera.offset = Vector2(camera_offset, camera_offset) * intensity
```

---

### controlled jump
- game: [Crazy Snowflakes](https://github.com/mazilimka/Crazy-Snowflakes)
- Godot: v4.3.stable.official [77dcf97d8]
```gdscript
extends CharacterBody2D
@export var gravity := 2000.0
@export var jump_velocity := -750.0
@export var max_jump_speed := -200.0

var max_jump_time := 0.5
var hold_time := 0.0

func _physics_process(delta):
	if not is_on_floor():
		velocity.y += gravity * delta
	else:
		velocity.y = 0.0
		is_jumping = false
	
	if Input.is_action_pressed("jump") and hold_time < max_jump_time and not is_jumping:
		hold_time += delta
		velocity.y = lerp(jump_velocity, max_jump_speed, hold_time / max_jump_time)
	if Input.is_action_just_released("jump"):
		is_jumping = true
		hold_time = 0.0

	move_and_slide()
```
