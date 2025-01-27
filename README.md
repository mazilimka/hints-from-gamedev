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

---

### a hint tweens of mobile controls
- game: [Crazy Snowflakes](https://github.com/mazilimka/Crazy-Snowflakes)
- Godot: v4.3.stable.official [77dcf97d8]
- **Don't forget to make rects inheritors of the CanvasLayer class!!!**
```gdscript
extends CharacterBody2D
func mobile_controlls_tween():
	get_tree().paused = true
	
	var tween1 := get_tree().create_tween().set_pause_mode(Tween.TWEEN_PAUSE_PROCESS).set_loops(3)
	tween1.tween_property(%LeftRect, "modulate", Color("a1ad0000"), 0.5)
	tween1.tween_property(%LeftRect, "modulate", Color("a1ad00"), 0.5)
	
	var tween2 := get_tree().create_tween().set_pause_mode(Tween.TWEEN_PAUSE_PROCESS).set_loops(3)
	tween2.tween_property(%RightRect, "modulate", Color("e4a1d200"), 0.5)
	tween2.tween_property(%RightRect, "modulate", Color("e4a1d2"), 0.5)
	
	var tween3 := get_tree().create_tween().set_pause_mode(Tween.TWEEN_PAUSE_PROCESS).set_loops(3)
	tween3.tween_property(%JumpRect, "modulate", Color("ff740200"), 0.5)
	tween3.tween_property(%JumpRect, "modulate", Color("ff7402"), 0.5)
	
	await tween1.finished
	await tween2.finished
	await tween3.finished
	
	tween1 = get_tree().create_tween().set_pause_mode(Tween.TWEEN_PAUSE_PROCESS)
	tween1.tween_property(%LeftRect, "modulate", Color("a1ad0000"), 0.1)
	
	tween2 = get_tree().create_tween().set_pause_mode(Tween.TWEEN_PAUSE_PROCESS)
	tween2.tween_property(%RightRect, "modulate", Color("e4a1d200"), 0.1)
	
	tween3 = get_tree().create_tween().set_pause_mode(Tween.TWEEN_PAUSE_PROCESS)
	tween3.tween_property(%JumpRect, "modulate", Color("ff740200"), 0.1)
	
	await tween1.finished
	await tween2.finished
	await tween3.finished
	
	get_tree().paused = false
```

---

### how to make sound settings
- game: [Crazy Snowflakes](https://github.com/mazilimka/Crazy-Snowflakes)
- Godot: v4.3.stable.official [77dcf97d8]
- **it is such an implementation in terms of dependence on the `Player` (`Global.Player`), but it works for fast and short games**
- it's the same with music

in settings script:
```gdscript
func _ready():
	sound_slider.volume = db_to_linear(Global.Player.broken_window_sound.volume_db)
	sound_slider.min_volume = 0
	sound_slider.max_volume = 1
	sound_slider.step = 0.001 # for smoothness (may 0.01 or 0.1)

func sound_changed(value: float):
	Global.Player.broken_window_sound.volume_db = linear_to_db(value)

```
