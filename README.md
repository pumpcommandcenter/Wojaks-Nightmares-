# Wojaks-Nightmares-
Contra Style 2d game of Wojaks Nightmares 
#!/usr/bin/env python3
"""
Wojak's Nightmares - Cyberpunk Contra Style 2D Run & Gun Prototype
Built for Pastor Wojak / Church of the Pump

Controls:
- LEFT / RIGHT arrows or A/D : Move
- SPACE : Jump
- LEFT CLICK or CTRL : Shoot golden guns (limited ammo)
- R : Reload
- ESC : Quit

Goal: Survive the zombie degen dog horde, reach the end, and defeat The BarsWaste Monster
to proclaim the Church of the Pump!

This is a fully playable starter. Replace the colored rects with sprites from the reference images.
"""

import pygame
import random
import sys
import math

# --- Init ---
pygame.init()
pygame.mixer.init()

WIDTH, HEIGHT = 1280, 720
screen = pygame.display.set_mode((WIDTH, HEIGHT))
pygame.display.set_caption("Wojak's Nightmares - Cyberpunk Contra Style")
clock = pygame.time.Clock()
FONT = pygame.font.SysFont("Arial", 28, bold=True)
SMALL_FONT = pygame.font.SysFont("Arial", 20)
BIG_FONT = pygame.font.SysFont("Arial", 72, bold=True)

# Colors - Cyberpunk Neon
DARK_BG = (10, 10, 25)
NEON_PINK = (255, 0, 150)
NEON_CYAN = (0, 255, 255)
NEON_YELLOW = (255, 255, 0)
GOLD = (255, 215, 0)
RED = (255, 50, 50)
GREEN = (50, 255, 100)
WHITE = (255, 255, 255)
DARK_GRAY = (30, 30, 40)

# Game constants
GRAVITY = 0.8
JUMP_STRENGTH = -16
PLAYER_SPEED = 6
BULLET_SPEED = 18
ENEMY_SPEED = 3
SPAWN_RATE = 45  # frames between dog spawns

# --- Game State ---
class GameState:
    MENU = 0
    PLAYING = 1
    WIN = 2
    LOSE = 3

state = GameState.MENU
score = 0
wave = 1

# --- Player (Pastor Wojak) ---
class Player:
    def __init__(self):
        self.x = 150
        self.y = HEIGHT - 150
        self.width = 60
        self.height = 90
        self.vel_y = 0
        self.on_ground = False
        self.health = 100
        self.ammo = 60
        self.max_ammo = 60
        self.facing_right = True
        self.shoot_cooldown = 0
        self.invincible = 0

    def rect(self):
        return pygame.Rect(self.x, self.y, self.width, self.height)

    def update(self, keys, mouse_pressed):
        # Movement
        dx = 0
        if keys[pygame.K_LEFT] or keys[pygame.K_a]:
            dx = -PLAYER_SPEED
            self.facing_right = False
        if keys[pygame.K_RIGHT] or keys[pygame.K_d]:
            dx = PLAYER_SPEED
            self.facing_right = True

        self.x += dx
        self.x = max(50, min(self.x, 3000))  # World bounds

        # Gravity & Jump
        self.vel_y += GRAVITY
        self.y += self.vel_y

        # Ground collision (simple floor + platforms)
        ground_y = HEIGHT - 120
        if self.y + self.height > ground_y:
            self.y = ground_y - self.height
            self.vel_y = 0
            self.on_ground = True
        else:
            self.on_ground = False

        # Jump
        if (keys[pygame.K_SPACE] or keys[pygame.K_w]) and self.on_ground:
            self.vel_y = JUMP_STRENGTH
            self.on_ground = False

        # Shooting
        if self.shoot_cooldown > 0:
            self.shoot_cooldown -= 1

        if (mouse_pressed[0] or keys[pygame.K_LCTRL]) and self.shoot_cooldown == 0 and self.ammo > 0:
            self.shoot()
            self.ammo -= 1
            self.shoot_cooldown = 8  # fire rate

        # Invincibility frames
        if self.invincible > 0:
            self.invincible -= 1

    def shoot(self):
        # Create two bullets (dual golden guns)
        direction = 1 if self.facing_right else -1
        bullets.append(Bullet(self.x + self.width//2, self.y + 30, direction))
        bullets.append(Bullet(self.x + self.width//2, self.y + 50, direction))
        
        # Muzzle flash effect (simple)
        particles.append(Particle(self.x + self.width//2 + (30 * direction), self.y + 40, GOLD, 8))

    def take_damage(self, amount):
        if self.invincible <= 0:
            self.health -= amount
            self.invincible = 30
            if self.health <= 0:
                global state
                state = GameState.LOSE

    def draw(self, surface, camera_x):
        screen_x = self.x - camera_x
        
        # Simple cyberpunk Wojak body (replace with sprite later)
        color = (200, 200, 220) if self.invincible % 4 < 2 else (100, 100, 150)
        
        # Trench coat body
        pygame.draw.rect(surface, (20, 20, 40), (screen_x, self.y + 20, self.width, self.height - 20))
        # Head (Wojak style - bald)
        pygame.draw.ellipse(surface, color, (screen_x + 10, self.y, 40, 45))
        # Sad/determined eyes
        pygame.draw.rect(surface, (30, 30, 30), (screen_x + 18, self.y + 15, 8, 6))
        pygame.draw.rect(surface, (30, 30, 30), (screen_x + 32, self.y + 15, 8, 6))
        
        # Neon collar
        pygame.draw.rect(surface, NEON_CYAN, (screen_x + 5, self.y + 45, self.width - 10, 8))
        
        # Golden guns
        gun_color = GOLD
        if self.facing_right:
            pygame.draw.rect(surface, gun_color, (screen_x + self.width - 5, self.y + 25, 35, 12))
            pygame.draw.rect(surface, gun_color, (screen_x + self.width - 5, self.y + 45, 35, 12))
        else:
            pygame.draw.rect(surface, gun_color, (screen_x - 30, self.y + 25, 35, 12))
            pygame.draw.rect(surface, gun_color, (screen_x - 30, self.y + 45, 35, 12))

# --- Bullet ---
class Bullet:
    def __init__(self, x, y, direction):
        self.x = x
        self.y = y
        self.direction = direction
        self.speed = BULLET_SPEED
        self.width = 12
        self.height = 4

    def update(self):
        self.x += self.speed * self.direction

    def rect(self):
        return pygame.Rect(self.x, self.y, self.width, self.height)

    def draw(self, surface, camera_x):
        screen_x = self.x - camera_x
        pygame.draw.rect(surface, GOLD, (screen_x, self.y, self.width, self.height))
        # Neon trail
        pygame.draw.rect(surface, NEON_YELLOW, (screen_x - 8*self.direction, self.y + 1, 8, 2))

# --- Enemy: Zombie Degen Dog ---
class ZombieDog:
    def __init__(self, x):
        self.x = x
        self.y = HEIGHT - 140
        self.width = 70
        self.height = 55
        self.health = 30
        self.speed = ENEMY_SPEED + random.uniform(-0.5, 1.5)

    def update(self, player_x):
        # Simple AI: move toward player
        if self.x > player_x:
            self.x -= self.speed
        else:
            self.x += self.speed * 0.3

    def rect(self):
        return pygame.Rect(self.x, self.y, self.width, self.height)

    def draw(self, surface, camera_x):
        screen_x = self.x - camera_x
        # Body - mangy zombie dog
        pygame.draw.ellipse(surface, (80, 40, 30), (screen_x, self.y + 10, self.width, self.height - 10))
        # Head
        pygame.draw.circle(surface, (100, 50, 40), (screen_x + 55, self.y + 20), 22)
        # Glowing red eyes (cybernetic)
        pygame.draw.circle(surface, RED, (screen_x + 62, self.y + 15), 6)
        pygame.draw.circle(surface, (255, 100, 0), (screen_x + 62, self.y + 15), 3)
        # Open mouth with teeth
        pygame.draw.polygon(surface, (200, 200, 200), [
            (screen_x + 70, self.y + 25), (screen_x + 85, self.y + 30), (screen_x + 70, self.y + 35)
        ])
        # Degen collar
        pygame.draw.rect(surface, NEON_PINK, (screen_x + 10, self.y + 35, 40, 8))
        pygame.draw.rect(surface, WHITE, (screen_x + 20, self.y + 36, 20, 3))

# --- Boss: BarsWaste Monster ---
class BarsWasteMonster:
    def __init__(self, x):
        self.x = x
        self.y = HEIGHT - 250
        self.width = 180
        self.height = 200
        self.health = 400
        self.max_health = 400
        self.phase = 0
        self.attack_timer = 0
        self.speed = 1.5

    def update(self, player_x):
        self.attack_timer += 1
        # Slowly chase player
        if self.x > player_x + 100:
            self.x -= self.speed
        elif self.x < player_x - 100:
            self.x += self.speed * 0.5

        # Simple attack pattern
        if self.attack_timer % 90 == 0:
            # "Rug pull" attack - spawn extra dogs
            for _ in range(2):
                enemies.append(ZombieDog(self.x + random.randint(50, 150)))

    def take_damage(self, amount):
        self.health -= amount
        if self.health <= 0:
            global state, score
            state = GameState.WIN
            score += 500

    def rect(self):
        return pygame.Rect(self.x, self.y, self.width, self.height)

    def draw(self, surface, camera_x):
        screen_x = self.x - camera_x
        # Massive grotesque body (liquidity bars + waste)
        pygame.draw.rect(surface, (60, 20, 20), (screen_x, self.y + 50, self.width, self.height - 50))
        # Head with dev headset vibe
        pygame.draw.ellipse(surface, (120, 30, 30), (screen_x + 40, self.y, 100, 70))
        # Angry eyes
        pygame.draw.circle(surface, RED, (screen_x + 65, self.y + 30), 12)
        pygame.draw.circle(surface, RED, (screen_x + 105, self.y + 30), 12)
        # Glowing chart lines on body (bars waste)
        for i in range(5):
            pygame.draw.line(surface, NEON_PINK, 
                           (screen_x + 20 + i*30, self.y + 80), 
                           (screen_x + 30 + i*30, self.y + 140 + random.randint(-20, 20)), 4)
        # Health bar above boss
        bar_width = 160
        health_ratio = self.health / self.max_health
        pygame.draw.rect(surface, RED, (screen_x + 10, self.y - 30, bar_width, 12))
        pygame.draw.rect(surface, GREEN, (screen_x + 10, self.y - 30, bar_width * health_ratio, 12))

# --- Particles (Rain + Effects) ---
class Particle:
    def __init__(self, x, y, color, size=3):
        self.x = x
        self.y = y
        self.color = color
        self.size = size
        self.vel_y = random.uniform(4, 9)
        self.life = random.randint(20, 50)

    def update(self):
        self.y += self.vel_y
        self.life -= 1

    def draw(self, surface, camera_x):
        if self.life > 0:
            screen_x = self.x - camera_x
            pygame.draw.circle(surface, self.color, (int(screen_x), int(self.y)), self.size)

# --- Game Objects ---
player = Player()
bullets = []
enemies = []
particles = []
boss = None
camera_x = 0
spawn_timer = 0
game_time = 0

def reset_game():
    global player, bullets, enemies, particles, boss, camera_x, spawn_timer, score, wave, state, game_time
    player = Player()
    bullets = []
    enemies = []
    particles = []
    boss = None
    camera_x = 0
    spawn_timer = 0
    score = 0
    wave = 1
    game_time = 0
    state = GameState.PLAYING

def spawn_enemy():
    x = camera_x + WIDTH + random.randint(50, 300)
    enemies.append(ZombieDog(x))

def create_rain():
    for _ in range(80):
        x = random.randint(int(camera_x), int(camera_x) + WIDTH)
        y = random.randint(0, HEIGHT)
        particles.append(Particle(x, y, (180, 200, 255), random.randint(1, 3)))

# Initial rain
create_rain()

# --- Main Game Loop ---
running = True
while running:
    dt = clock.tick(60)
    keys = pygame.key.get_pressed()
    mouse_pressed = pygame.mouse.get_pressed()

    for event in pygame.event.get():
        if event.type == pygame.QUIT:
            running = False
        if event.type == pygame.KEYDOWN:
            if event.key == pygame.K_ESCAPE:
                running = False
            if state == GameState.MENU and event.key == pygame.K_RETURN:
                reset_game()
            if state in (GameState.WIN, GameState.LOSE) and event.key == pygame.K_r:
                reset_game()
            if state == GameState.PLAYING and event.key == pygame.K_r:
                player.ammo = player.max_ammo  # Reload

    # --- UPDATE ---
    if state == GameState.PLAYING:
        game_time += 1
        
        # Update player
        player.update(keys, mouse_pressed)
        
        # Camera follow (Contra style - follows player but allows some lookahead)
        target_camera = player.x - WIDTH // 3
        camera_x = camera_x * 0.9 + target_camera * 0.1
        
        # Update bullets
        for bullet in bullets[:]:
            bullet.update()
            if bullet.x < camera_x - 100 or bullet.x > camera_x + WIDTH + 100:
                bullets.remove(bullet)
        
        # Update enemies
        for enemy in enemies[:]:
            enemy.update(player.x)
            # Collision with player
            if enemy.rect().colliderect(player.rect()) and player.invincible <= 0:
                player.take_damage(15)
                enemies.remove(enemy)
                particles.append(Particle(enemy.x, enemy.y, RED, 15))
            # Bullet hits
            for bullet in bullets[:]:
                if enemy.rect().colliderect(bullet.rect()):
                    enemy.health -= 25
                    bullets.remove(bullet)
                    particles.append(Particle(bullet.x, bullet.y, GOLD, 8))
                    if enemy.health <= 0:
                        enemies.remove(enemy)
                        score += 25
                        particles.append(Particle(enemy.x, enemy.y, GREEN, 20))
                        break
        
        # Spawn zombie degen dogs
        spawn_timer += 1
        if spawn_timer > SPAWN_RATE and len(enemies) < 8 and not boss:
            spawn_enemy()
            spawn_timer = 0
            if random.random() < 0.3:  # occasional extra spawn
                spawn_enemy()
        
        # Boss trigger (after some progress or score)
        if not boss and (player.x > 1800 or score > 150):
            boss = BarsWasteMonster(camera_x + WIDTH + 200)
            # Clear some regular enemies
            enemies = enemies[:3]
        
        if boss:
            boss.update(player.x)
            # Boss bullet collisions
            for bullet in bullets[:]:
                if boss.rect().colliderect(bullet.rect()):
                    boss.take_damage(12)
                    bullets.remove(bullet)
                    particles.append(Particle(bullet.x, bullet.y, NEON_PINK, 10))
            # Boss touches player
            if boss.rect().colliderect(player.rect()) and player.invincible <= 0:
                player.take_damage(30)
        
        # Update particles (rain + effects)
        for p in particles[:]:
            p.update()
            if p.life <= 0:
                particles.remove(p)
        
        # Occasional new rain
        if random.random() < 0.4:
            x = camera_x + random.randint(0, WIDTH)
            particles.append(Particle(x, 0, (180, 200, 255), random.randint(1, 3)))
        
        # Win condition (defeated boss)
        if boss and boss.health <= 0:
            state = GameState.WIN
        
        # Lose condition already handled in take_damage

    # --- DRAW ---
    screen.fill(DARK_BG)
    
    # Draw rain / particles first (background layer)
    for p in particles:
        p.draw(screen, camera_x)
    
    if state == GameState.MENU:
        # Title screen using the vibe of the reference
        title = BIG_FONT.render("WOJAK'S NIGHTMARES", True, NEON_PINK)
        screen.blit(title, (WIDTH//2 - title.get_width()//2, 150))
        
        subtitle = FONT.render("CYBERPUNK CONTRA STYLE", True, NEON_CYAN)
        screen.blit(subtitle, (WIDTH//2 - subtitle.get_width()//2, 230))
        
        story = [
            "You bought the dip... went to bed happy.",
            "Woke up to a world of evil.",
            "Your portfolio was liquidated.",
            "The dev turned off the livestream.",
            "",
            "Now you must face the horde of zombie degen dogs",
            "and The BarsWaste Monster to proclaim",
            "THE CHURCH OF THE PUMP!"
        ]
        y = 320
        for line in story:
            txt = SMALL_FONT.render(line, True, WHITE)
            screen.blit(txt, (WIDTH//2 - txt.get_width()//2, y))
            y += 32
        
        prompt = FONT.render("PRESS ENTER TO BEGIN THE NIGHTMARE", True, GOLD)
        screen.blit(prompt, (WIDTH//2 - prompt.get_width()//2, HEIGHT - 120))

    elif state == GameState.PLAYING:
        # Draw ground / platforms
        ground_y = HEIGHT - 120
        pygame.draw.rect(screen, (25, 25, 45), (0, ground_y, WIDTH, HEIGHT - ground_y))
        # Neon reflections on ground
        pygame.draw.line(screen, NEON_CYAN, (0, ground_y), (WIDTH, ground_y), 3)
        
        # Draw distant cyberpunk city silhouette (simple)
        for i in range(8):
            building_x = 100 + i * 180 - (camera_x * 0.3) % 400
            h = 150 + (i % 3) * 80
            pygame.draw.rect(screen, (15, 15, 35), (building_x, ground_y - h, 80, h))
            # Window lights
            if i % 2 == 0:
                pygame.draw.rect(screen, NEON_PINK, (building_x + 20, ground_y - h + 30, 15, 15))
        
        # Draw player
        player.draw(screen, camera_x)
        
        # Draw bullets
        for bullet in bullets:
            bullet.draw(screen, camera_x)
        
        # Draw enemies
        for enemy in enemies:
            enemy.draw(screen, camera_x)
        
        # Draw boss
        if boss:
            boss.draw(screen, camera_x)
        
        # Draw particles on top
        for p in particles:
            p.draw(screen, camera_x)
        
        # UI
        # Health bar
        pygame.draw.rect(screen, RED, (30, 30, 200, 22))
        pygame.draw.rect(screen, GREEN, (30, 30, 200 * (player.health / 100), 22))
        health_txt = SMALL_FONT.render(f"PASTOR WOJAK  {int(player.health)}%", True, WHITE)
        screen.blit(health_txt, (35, 28))
        
        # Ammo
        ammo_txt = FONT.render(f"GOLDEN GUNS  {player.ammo} / {player.max_ammo}", True, GOLD)
        screen.blit(ammo_txt, (WIDTH - 320, 30))
        
        # Score & Wave
        score_txt = FONT.render(f"DEGEN KILLS: {score}", True, NEON_CYAN)
        screen.blit(score_txt, (30, 70))
        
        if boss:
            boss_txt = FONT.render("BOSS: THE BARSWASTE MONSTER", True, NEON_PINK)
            screen.blit(boss_txt, (WIDTH//2 - 220, 30))
        
        # Story reminder (subtle)
        if game_time < 300:
            hint = SMALL_FONT.render("Move with ARROWS â¢ Jump SPACE â¢ Shoot CTRL or MOUSE", True, (200, 200, 200))
            screen.blit(hint, (WIDTH//2 - 280, HEIGHT - 50))

    elif state == GameState.WIN:
        win_title = BIG_FONT.render("THE CHURCH OF THE PUMP RISES", True, GOLD)
        screen.blit(win_title, (WIDTH//2 - win_title.get_width()//2, 200))
        
        msg = [
            "You faced the zombie degen horde.",
            "You defeated The BarsWaste Monster.",
            "Your portfolio may be gone...",
            "But your spirit and the Pump are eternal.",
            "",
            f"Final Score: {score} Degen Kills"
        ]
        y = 320
        for line in msg:
            txt = FONT.render(line, True, WHITE)
            screen.blit(txt, (WIDTH//2 - txt.get_width()//2, y))
            y += 45
        
        again = FONT.render("PRESS R TO PLAY AGAIN  â¢  ESC TO QUIT", True, NEON_CYAN)
        screen.blit(again, (WIDTH//2 - again.get_width()//2, HEIGHT - 100))

    elif state == GameState.LOSE:
        lose_title = BIG_FONT.render("LIQUIDATED...", True, RED)
        screen.blit(lose_title, (WIDTH//2 - lose_title.get_width()//2, 200))
        
        msg = [
            "The nightmare consumed you.",
            "Another degen falls to the horde.",
            "But legends never truly die...",
            "The Church of the Pump will rise again."
        ]
        y = 320
        for line in msg:
            txt = FONT.render(line, True, WHITE)
            screen.blit(txt, (WIDTH//2 - txt.get_width()//2, y))
            y += 45
        
        again = FONT.render("PRESS R TO TRY AGAIN  â¢  ESC TO QUIT", True, NEON_CYAN)
        screen.blit(again, (WIDTH//2 - again.get_width()//2, HEIGHT - 100))

    pygame.display.flip()

pygame.quit()
sys.exit()
cd /home/workdir/artifacts
python wojak_nightmares.py
cd /home/workdir/artifacts
python wojak_nightmares.py
Wojaks_Nightmares/
├── assets/
│   ├── sprites/          ← Put poster + concept art PNGs here
│   ├── tilesets/
│   └── audio/
├── scenes/
│   ├── player/
│   ├── enemies/
│   ├── collectibles/
│   └── levels/
├── scripts/
├── ui/
└── audio/
# scripts/Player.gd
extends CharacterBody2D

@export var speed = 280.0
@export var acceleration = 1200.0
@export var friction = 800.0
@export var jump_velocity = -420.0

var health = 100
var lives = 3
var faith = 0
var solana_orbs = 0
var facing_right = true

@onready var animated_sprite = $AnimatedSprite2D
@onready var fireball_scene = preload("res://scenes/Fireball.tscn")

func _physics_process(delta):
    # Horizontal movement (smooth)
    var direction = Input.get_axis("ui_left", "ui_right")
    if direction:
        velocity.x = move_toward(velocity.x, direction * speed, acceleration * delta)
        facing_right = direction > 0
    else:
        velocity.x = move_toward(velocity.x, 0, friction * delta)

    # Gravity
    if not is_on_floor():
        velocity.y += get_gravity().y * delta

    # Jump
    if Input.is_action_just_pressed("ui_accept") and is_on_floor():
        velocity.y = jump_velocity

    move_and_slide()

    # Shooting (Fireball)
    if Input.is_action_just_pressed("shoot"):
        shoot_fireball()

    # Update facing
    animated_sprite.flip_h = not facing_right

func shoot_fireball():
    var fireball = fireball_scene.instantiate()
    fireball.position = position + Vector2(40 if facing_right else -40, -10)
    fireball.direction = 1 if facing_right else -1
    get_parent().add_child(fireball)
    
    # Play spatial sound
    $AudioStreamPlayer2D.play()

func collect_item(item_type: String):
    match item_type:
        "holy_candle":
            faith += 10
            # Update Faith Meter UI
        "solana_orb":
            solana_orbs += 1
        "green_herb", "first_aid":
            health = min(health + 30, 100)
        "weapon_upgrade":
            # Upgrade fireball or unlock spell
            pass
            # scripts/FloatingText.gd
extends Label

func show_text(text: String, color: Color = Color.RED):
    text = text
    modulate = color
    var tween = create_tween()
    tween.tween_property(self, "position:y", position.y - 60, 1.2)
    tween.tween_property(self, "modulate:a", 0.0, 0.8)
    tween.tween_callback(queue_free)
    assets/
    sprites/
    audio/
scenes/
    player/
    enemies/
    collectibles/
    levels/
    ui/
scripts/
extends CharacterBody2D
class_name Player

@export var speed: float = 320.0
@export var acceleration: float = 1400.0
@export var friction: float = 900.0
@export var jump_velocity: float = -480.0

var health: int = 100
var lives: int = 3
var faith: int = 0
var solana_orbs: int = 0

@onready var animated_sprite: AnimatedSprite2D = $AnimatedSprite2D
@onready var audio_player: AudioStreamPlayer2D = $AudioStreamPlayer2D

var gravity = ProjectSettings.get_setting("physics/2d/default_gravity")
var facing_right: bool = true

func _physics_process(delta: float) -> void:
    # Smooth horizontal movement
    var direction := Input.get_axis("ui_left", "ui_right")
    
    if direction != 0:
        velocity.x = move_toward(velocity.x, direction * speed, acceleration * delta)
        facing_right = direction > 0
    else:
        velocity.x = move_toward(velocity.x, 0, friction * delta)

    # Gravity
    if not is_on_floor():
        velocity.y += gravity * delta

    # Jump
    if Input.is_action_just_pressed("ui_accept") and is_on_floor():
        velocity.y = jump_velocity

    move_and_slide()

    # Update sprite direction
    if animated_sprite:
        animated_sprite.flip_h = not facing_right

    # Shooting
    if Input.is_action_just_pressed("shoot"):
        shoot_fireball()

func shoot_fireball() -> void:
    var fireball = preload("res://scenes/Fireball.tscn").instantiate()
    fireball.global_position = global_position + Vector2(45 if facing_right else -45, -15)
    fireball.direction = 1 if facing_right else -1
    get_parent().add_child(fireball)
    
    # Spatial audio
    if audio_player:
        audio_player.play()

func collect(item_type: String, value: int = 1) -> void:
    match item_type:
        "holy_candle":
            faith += value
            get_tree().call_group("ui", "update_faith", faith)
        "solana_orb":
            solana_orbs += value
        "green_herb", "first_aid":
            health = min(health + 35, 100)
        "weapon_upgrade":
            print("Weapon upgraded!")
            extends Area2D

@export var speed: float = 650.0
var direction: int = 1

func _physics_process(delta: float) -> void:
    position.x += speed * direction * delta

func _on_body_entered(body: Node2D) -> void:
    if body.is_in_group("enemy"):
        if body.has_method("take_damage"):
            body.take_damage(25)
    queue_free()
    extends CharacterBody2D

@export var speed: float = 120.0
var health: int = 40

@onready var floating_text_scene = preload("res://scenes/FloatingText.tscn")

func _physics_process(delta: float) -> void:
    # Simple movement toward player (basic AI)
    var player = get_tree().get_first_node_in_group("player")
    if player:
        var dir = sign(player.global_position.x - global_position.x)
        velocity.x = dir * speed
    move_and_slide()

func take_damage(amount: int) -> void:
    health -= amount
    if health <= 0:
        die()

func die() -> void:
    # Spawn floating horror text
    var text = floating_text_scene.instantiate()
    text.global_position = global_position
    get_parent().add_child(text)
    text.show_text(["RUGGED", "DEV WALLET MOVED", "ANOTHER SOUL LIQUIDATED"].pick_random())
    
    queue_free()
    extends Label

func show_text(message: String) -> void:
    text = message
    modulate = Color(1, 0.2, 0.2)
    var tween = create_tween()
    tween.tween_property(self, "position:y", position.y - 80, 1.0)
    tween.parallel().tween_property(self, "modulate:a", 0.0, 0.9)
    tween.tween_callback(queue_free)
    extends Area2D

@export var item_type: String = "holy_candle"

func _on_body_entered(body: Node2D) -> void:
    if body is Player:
        body.collect(item_type)
        queue_free()
        func _ready() -> void:
    # Enable spatial audio listening from the player
    var listener = AudioListener2D.new()
    add_child(listener)
    listener.make_current()
    Level1 (Node2D)
├── TileMapLayer_Ground (TileMapLayer)
├── TileMapLayer_Background (TileMapLayer)
├── TileMapLayer_Details (TileMapLayer)
├── ParallaxBackground (optional for depth)
├── Player
├── Camera2D
├── UI
│   └── FaithMeter
├── Enemies (Node)
└── Collectibles (Node)
extends Node2D

@export var next_level: PackedScene

func _ready() -> void:
    # Spawn some initial Mad Chadlers
    spawn_enemy(Vector2(600, 500))
    spawn_enemy(Vector2(900, 500))
    
    # Spawn collectibles
    spawn_collectible("holy_candle", Vector2(450, 480))
    spawn_collectible("solana_orb", Vector2(750, 520))

func spawn_enemy(pos: Vector2) -> void:
    var enemy = preload("res://scenes/enemies/MadChadler.tscn").instantiate()
    enemy.global_position = pos
    $Enemies.add_child(enemy)

func spawn_collectible(type: String, pos: Vector2) -> void:
    var collectible = preload("res://scenes/collectibles/Collectible.tscn").instantiate()
    collectible.item_type = type
    collectible.global_position = pos
    $Collectibles.add_child(collectible)
    extends Control

@onready var faith_bar: ProgressBar = $FaithBar
@onready var faith_label: Label = $FaithLabel

func _ready() -> void:
    add_to_group("ui")
    faith_bar.max_value = 100
    update_faith(0)

func update_faith(new_faith: int) -> void:
    faith_bar.value = new_faith
    faith_label.text = "FAITH: %d" % new_faith
    
    # Visual feedback when faith increases
    if new_faith > faith_bar.value:
        var tween = create_tween()
        tween.tween_property(faith_bar, "modulate", Color(0.2, 1, 0.4), 0.2)
        tween.tween_property(faith_bar, "modulate", Color.WHITE, 0.3)
        extends CharacterBody2D

@export var speed: float = 140.0
@export var health: int = 350

var player: Node2D
var attack_cooldown: bool = false

@onready var sprite: AnimatedSprite2D = $AnimatedSprite2D
@onready var attack_timer: Timer = $AttackTimer
@onready var audio: AudioStreamPlayer2D = $AudioStreamPlayer2D

func _ready() -> void:
    player = get_tree().get_first_node_in_group("player")
    attack_timer.timeout.connect(_perform_attack)

func _physics_process(delta: float) -> void:
    if player:
        var direction = sign(player.global_position.x - global_position.x)
        velocity.x = direction * speed
    move_and_slide()

func take_damage(amount: int) -> void:
    health -= amount
    if health <= 0:
        die()

func _perform_attack() -> void:
    if attack_cooldown or not player:
        return
    
    attack_cooldown = true
    
    var attack_type = randi() % 3
    match attack_type:
        0: # Rug Pull Charge
            _rug_pull_charge()
        1: # Honeypot Trap
            _spawn_honeypot()
        2: # Chart Manipulation (distort platforms)
            _chart_manipulation()
    
    await get_tree().create_timer(2.5).timeout
    attack_cooldown = false

func _rug_pull_charge() -> void:
    # Fast charge toward player
    var direction = sign(player.global_position.x - global_position.x)
    velocity.x = direction * 450
    print("BarsWaste used RUG PULL!")

func _spawn_honeypot() -> void:
    # Spawn trap projectile
    var trap = preload("res://scenes/projectiles/HoneypotTrap.tscn").instantiate()
    trap.global_position = global_position + Vector2(0, -30)
    get_parent().add_child(trap)
    print("BarsWaste deployed HONEYPOT!")

func _chart_manipulation() -> void:
    # Temporary platform distortion (placeholder)
    print("BarsWaste manipulated the CHARTS!")
    # Later: shake platforms or change gravity locally

func die() -> void:
    print("BarsWaste Monster defeated!")
    queue_free()
    extends "res://scripts/Collectible.gd"

func _ready() -> void:
    item_type = "holy_candle"
    extends "res://scripts/Collectible.gd"

func _ready() -> void:
    item_type = "solana_orb"
    ParallaxBackground
├── Parallax2D (FarBackground)     ← Speed 0.2
├── Parallax2D (MidBackground)     ← Speed 0.5
└── Parallax2D (Foreground)        ← Speed 0.8
extends Parallax2D

@export var scroll_speed: float = 0.3

func _process(delta: float) -> void:
    scroll_offset.x -= scroll_speed * delta * 50
    extends CharacterBody2D
class_name StateMachineBoss

enum State { IDLE, CHASE, ATTACK, SPECIAL, DEAD }

@export var max_health: int = 300
var current_state: State = State.IDLE
var health: int = max_health
var player: Node2D

signal boss_died

func _ready() -> void:
    player = get_tree().get_first_node_in_group("player")
    health = max_health

func change_state(new_state: State) -> void:
    if current_state == new_state:
        return
    current_state = new_state
    _on_state_changed(new_state)

func _on_state_changed(new_state: State) -> void:
    pass  # Override in child scripts

func take_damage(amount: int) -> void:
    if current_state == State.DEAD:
        return
    health -= amount
    if health <= 0:
        die()

func die() -> void:
    change_state(State.DEAD)
    emit_signal("boss_died")
    queue_free()
    extends StateMachineBoss

@export var charge_speed: float = 520.0
@export var normal_speed: float = 160.0

var is_charging: bool = false
var charge_direction: int = 1

@onready var attack_timer: Timer = $AttackTimer
@onready var sprite: AnimatedSprite2D = $AnimatedSprite2D

func _ready() -> void:
    super._ready()
    attack_timer.timeout.connect(_choose_attack)

func _physics_process(delta: float) -> void:
    if current_state == State.DEAD:
        return

    if player:
        var dir = sign(player.global_position.x - global_position.x)
        
        if current_state == State.CHASE:
            velocity.x = dir * normal_speed
        elif current_state == State.ATTACK and is_charging:
            velocity.x = charge_direction * charge_speed
        else:
            velocity.x = move_toward(velocity.x, 0, 300 * delta)

    move_and_slide()

func _choose_attack() -> void:
    if current_state != State.CHASE:
        return
    
    var distance = global_position.distance_to(player.global_position)
    
    if distance < 280:
        change_state(State.ATTACK)
        is_charging = true
        charge_direction = sign(player.global_position.x - global_position.x)
        await get_tree().create_timer(1.8).timeout
        is_charging = false
        change_state(State.CHASE)
    else:
        # Future: Add projectile or special attack
        pass
        func cast_spell(spell_name: String) -> void:
    if solana_orbs < 3:
        print("Not enough Solana Orbs!")
        return
    
    solana_orbs -= 3
    
    match spell_name:
        "acid_rain":
            _cast_acid_rain()
        "fire_storm":
            _cast_fire_storm()
        "lightning_tempest":
            _cast_lightning_tempest()

func _cast_acid_rain() -> void:
    print("Casting Acid Rain!")
    # Spawn acid particles/projectiles in area

func _cast_fire_storm() -> void:
    print("Casting Fire Storm!")

func _cast_lightning_tempest() -> void:
    print("Casting Lightning Tempest!")
    if Input.is_action_just_pressed("cast_acid"):
    cast_spell("acid_rain")
    shader_type canvas_item;

uniform float darkness : hint_range(0.0, 1.0) = 0.65;
uniform vec4 blood_tint : source_color = vec4(0.6, 0.1, 0.1, 1.0);

void fragment() {
    vec4 color = texture(TEXTURE, UV);
    color.rgb *= (1.0 - darkness);
    color.rgb = mix(color.rgb, blood_tint.rgb, 0.15);
    COLOR = color;
}
extends Area2D

@export var item_type: String = "holy_candle"
@export var value: int = 1

func _ready() -> void:
    body_entered.connect(_on_body_entered)

func _on_body_entered(body: Node2D) -> void:
    if body is Player:
        body.collect(item_type, value)
        queue_free()
        Level2_GothicCathedral
├── ParallaxBackground
│   ├── Parallax2D (Far Gothic Pillars) - Speed 0.15
│   ├── Parallax2D (Mid Stained Glass) - Speed 0.4
│   └── Parallax2D (Foreground Candles) - Speed 0.75
├── TileMapLayer_Ground
├── TileMapLayer_Walls
├── TileMapLayer_Details
├── Player
├── Camera2D
└── UI
#!/bin/bash
echo "Exporting Wojak's Nightmares to HTML5..."

GODOT_PATH="godot"   # Change if Godot is not in PATH

$GODOT_PATH --headless --export-release "Web" ./build/index.html

echo "Build complete! Open ./build/index.html in a browser."
chmod +x export_game.sh
./export_game.sh
# scripts/bosses/FalseMoonAttack.gd
extends StateMachineBoss

@export var tentacle_damage: int = 20
@export var fire_damage: int = 15

var is_attacking: bool = false

@onready var attack_timer: Timer = $AttackTimer
@onready var sprite: AnimatedSprite2D = $AnimatedSprite2D

func _ready() -> void:
    super._ready()
    attack_timer.timeout.connect(_perform_attack)

func _physics_process(delta: float) -> void:
    if current_state == State.DEAD: return
    
    if player and current_state == State.CHASE:
        var dir = sign(player.global_position.x - global_position.x)
        velocity.x = dir * 130
    move_and_slide()

func _perform_attack() -> void:
    if current_state != State.CHASE or is_attacking: return
    
    is_attacking = true
    change_state(State.ATTACK)
    
    var attack_choice = randi() % 3
    match attack_choice:
        0: _multi_tentacle_attack()
        1: _fire_breath()
        2: _tail_sting()
    
    await get_tree().create_timer(2.0).timeout
    is_attacking = false
    change_state(State.CHASE)

func _multi_tentacle_attack() -> void:
    print("False Moon: Multi-Tentacle Attack!")
    # Spawn multiple projectiles or area damage

func _fire_breath() -> void:
    print("False Moon: Fire Breath!")
    # Spawn fire particles/projectiles toward player

func _tail_sting() -> void:
    print("False Moon: Tail Sting!")
    if player:
        var distance = global_position.distance_to(player.global_position)
        if distance < 180:
            player.take_damage(25)
            # scripts/bosses/RUGPULL_Overlord.gd
extends StateMachineBoss

@export var acid_damage: int = 18

@onready var attack_timer: Timer = $AttackTimer

func _ready() -> void:
    super._ready()
    attack_timer.timeout.connect(_perform_attack)

func _physics_process(delta: float) -> void:
    if current_state == State.DEAD: return
    if player and current_state == State.CHASE:
        var dir = sign(player.global_position.x - global_position.x)
        velocity.x = dir * 110
    move_and_slide()

func _perform_attack() -> void:
    if current_state != State.CHASE: return
    change_state(State.SPECIAL)
    
    var choice = randi() % 3
    match choice:
        0: _shadow_teleport()
        1: _spawn_acid_pools()
        2: _chart_manipulation()
    
    await get_tree().create_timer(2.5).timeout
    change_state(State.CHASE)

func _shadow_teleport() -> void:
    print("RUGPULL: Shadow Teleport!")
    if player:
        global_position = player.global_position + Vector2(randf_range(-200, 200), -80)

func _spawn_acid_pools() -> void:
    print("RUGPULL: Acid Pools!")
    # Spawn acid damage zones on ground

func _chart_manipulation() -> void:
    print("RUGPULL: Chart Manipulation!")
    # Shake camera + temporarily alter platform physics
    # scripts/bosses/LegionOfLiquidation.gd
extends StateMachineBoss

@export var lightning_damage: int = 30

var phase: int = 1

@onready var attack_timer: Timer = $AttackTimer

func _ready() -> void:
    super._ready()
    max_health = 600
    health = max_health
    attack_timer.timeout.connect(_perform_attack)

func take_damage(amount: int) -> void:
    super.take_damage(amount)
    _check_phase()

func _check_phase() -> void:
    if health < max_health * 0.66 and phase == 1:
        phase = 2
        print("Legion Phase 2: Blood Wings Active")
    elif health < max_health * 0.33 and phase == 2:
        phase = 3
        print("Legion Phase 3: Full Liquidation")

func _perform_attack() -> void:
    if current_state == State.DEAD: return
    change_state(State.SPECIAL)
    
    match phase:
        1: _lightning_strike()
        2: _summon_minions()
        3: _blood_wing_dive()
    
    await get_tree().create_timer(3.0).timeout
    change_state(State.CHASE)

func _lightning_strike() -> void:
    print("Legion: Lightning Tempest!")

func _summon_minions() -> void:
    print("Legion: Summons Mad Chadlers!")

func _blood_wing_dive() -> void:
    print("Legion: Blood Wing Dive!")
    Level2_GothicCathedral (Node2D)
├── ParallaxBackground
│   ├── Parallax2D_Far (Speed 0.2)      ← Dark cathedral pillars
│   ├── Parallax2D_Mid (Speed 0.45)     ← Stained glass + candles
│   └── Parallax2D_Fore (Speed 0.8)     ← Floating debris / chains
├── TileMapLayer_Ground
├── TileMapLayer_Walls
├── TileMapLayer_Details (apply gothic shader here)
├── Player
├── Camera2D
├── Enemies
├── Bosses
└── UI
var lives: int = 3

func take_damage(amount: int) -> void:
    health -= amount
    if health <= 0:
        health = 100
        lives -= 1
        if lives <= 0:
            _game_over()
        else:
            # Respawn logic or checkpoint
            print("Lives remaining:", lives)

func _game_over() -> void:
    get_tree().change_scene_to_file("res://scenes/ui/GameOver.tscn")
    # autoload/DifficultyManager.gd
extends Node

var current_level: int = 1

func get_enemy_health_multiplier() -> float:
    return 1.0 + (current_level - 1) * 0.25

func get_enemy_speed_multiplier() -> float:
    return 1.0 + (current_level - 1) * 0.15

func get_boss_health_multiplier() -> float:
    return 1.0 + (current_level - 1) * 0.4
    extends Control

@onready var continue_button: Button = $ContinueButton
@onready var new_game_button: Button = $NewGameButton

func _ready() -> void:
    # Hide Continue if no save exists
    if not SaveSystem.has_save():
        continue_button.visible = false
    
    # Connect buttons
    new_game_button.pressed.connect(_on_new_game_pressed)
    continue_button.pressed.connect(_on_continue_pressed)
    
    # Optional: Play menu music
    # $MenuMusic.play()

func _on_new_game_pressed() -> void:
    SaveSystem.new_game()
    get_tree().change_scene_to_file("res://scenes/levels/Level1.tscn")

func _on_continue_pressed() -> void:
    SaveSystem.load_game()
    var level_path = "res://scenes/levels/Level%d.tscn" % SaveSystem.current_level
    get_tree().change_scene_to_file(level_path)
    extends Node

const SAVE_PATH = "user://wojak_nightmares_save.cfg"

var current_level: int = 1
var lives: int = 3
var faith: int = 0
var solana_orbs: int = 0
var health: int = 100

func new_game() -> void:
    current_level = 1
    lives = 3
    faith = 0
    solana_orbs = 0
    health = 100
    save_game()

func save_game() -> void:
    var config = ConfigFile.new()
    
    config.set_value("progress", "current_level", current_level)
    config.set_value("player", "lives", lives)
    config.set_value("player", "faith", faith)
    config.set_value("player", "solana_orbs", solana_orbs)
    config.set_value("player", "health", health)
    
    config.save(SAVE_PATH)
    print("Game saved successfully.")

func load_game() -> bool:
    var config = ConfigFile.new()
    var err = config.load(SAVE_PATH)
    
    if err != OK:
        print("No save file found.")
        return false
    
    current_level = config.get_value("progress", "current_level", 1)
    lives = config.get_value("player", "lives", 3)
    faith = config.get_value("player", "faith", 0)
    solana_orbs = config.get_value("player", "solana_orbs", 0)
    health = config.get_value("player", "health", 100)
    
    print("Game loaded successfully.")
    return true

func has_save() -> bool:
    return FileAccess.file_exists(SAVE_PATH)

func delete_save() -> void:
    if has_save():
        DirAccess.remove_absolute(SAVE_PATH)
        print("Save file deleted.")
       # When collecting items
func collect(item_type: String, value: int = 1) -> void:
    match item_type:
        "holy_candle":
            faith += value
        "solana_orb":
            solana_orbs += value
        ...
    SaveSystem.save_game()   # Auto-save on pickup

# When taking damage or losing a life
func take_damage(amount: int) -> void:
    ...
    SaveSystem.lives = lives
    SaveSystem.health = health
    SaveSystem.save_game()
    SaveSystem.current_level = 2
SaveSystem.save_game()
get_tree().change_scene_to_file("res://scenes/levels/Level2_GothicCathedral.tscn")
extends Control

@onready var orb_label: Label = $OrbLabel
@onready var acid_button: Button = $AcidRainButton
@onready var fire_button: Button = $FireStormButton
@onready var lightning_button: Button = $LightningTempestButton

func _ready() -> void:
    add_to_group("ui")
    update_ui()

func update_ui() -> void:
    var orbs = SaveSystem.solana_orbs
    orb_label.text = "Solana Orbs: %d" % orbs
    
    acid_button.disabled = orbs < 3
    fire_button.disabled = orbs < 3
    lightning_button.disabled = orbs < 3

func _on_acid_rain_pressed() -> void:
    get_tree().get_first_node_in_group("player").cast_spell("acid_rain")
    update_ui()

func _on_fire_storm_pressed() -> void:
    get_tree().get_first_node_in_group("player").cast_spell("fire_storm")
    update_ui()

func _on_lightning_tempest_pressed() -> void:
    get_tree().get_first_node_in_group("player").cast_spell("lightning_tempest")
    update_ui()
    extends Camera2D

@export var shake_strength: float = 30.0
@export var shake_fade: float = 5.0

var shake_amount: float = 0.0

func _ready() -> void:
    # Set camera limits (adjust per level)
    limit_left = 0
    limit_top = -200
    limit_right = 3000
    limit_bottom = 800

func _process(delta: float) -> void:
    if shake_amount > 0:
        shake_amount = lerp(shake_amount, 0.0, shake_fade * delta)
        offset = Vector2(randf_range(-shake_amount, shake_amount), 
                         randf_range(-shake_amount, shake_amount))

func apply_shake(strength: float = 25.0) -> void:
    shake_amount = strength
    get_tree().get_first_node_in_group("camera").apply_shake(40)
    extends Node

const SAVE_PATH := "user://wojak_save_slot_%d.dat"
const ENCRYPTION_KEY := "ChurchOfPump2026"  # Change this

var current_slot: int = 1
var current_level: int = 1
var lives: int = 3
var faith: int = 0
var solana_orbs: int = 0
var health: int = 100

func save_game(slot: int = -1) -> void:
    if slot == -1: slot = current_slot
    
    var path = SAVE_PATH % slot
    var file = FileAccess.open_encrypted_with_pass(path, FileAccess.WRITE, ENCRYPTION_KEY)
    
    if file == null:
        push_error("Failed to save game")
        return
    
    var data = {
        "current_level": current_level,
        "lives": lives,
        "faith": faith,
        "solana_orbs": solana_orbs,
        "health": health
    }
    
    file.store_string(JSON.stringify(data))
    file.close()
    print("Game saved to slot %d" % slot)

func load_game(slot: int = -1) -> bool:
    if slot == -1: slot = current_slot
    
    var path = SAVE_PATH % slot
    if not FileAccess.file_exists(path):
        return false
    
    var file = FileAccess.open_encrypted_with_pass(path, FileAccess.READ, ENCRYPTION_KEY)
    if file == null:
        return false
    
    var json_string = file.get_as_text()
    file.close()
    
    var data = JSON.parse_string(json_string)
    if data == null:
        return false
    
    current_level = data.get("current_level", 1)
    lives = data.get("lives", 3)
    faith = data.get("faith", 0)
    solana_orbs = data.get("solana_orbs", 0)
    health = data.get("health", 100)
    
    print("Game loaded from slot %d" % slot)
    return true

func has_save(slot: int) -> bool:
    return FileAccess.file_exists(SAVE_PATH % slot)

func auto_save() -> void:
    save_game(current_slot)  # Auto-save to current slot

func new_game(slot: int = 1) -> void:
    current_slot = slot
    current_level = 1
    lives = 3
    faith = 0
    solana_orbs = 0
    health = 100
    save_game(slot)
    SaveSystem.save_game(1)   # Save to Slot 1
SaveSystem.load_game(2)   # Load from Slot 2
extends Control

func _on_delete_save_pressed(slot: int) -> void:
    var path = "user://wojak_save_slot_%d.dat" % slot
    if FileAccess.file_exists(path):
        DirAccess.remove_absolute(path)
        print("Save slot %d deleted" % slot)
        extends Node

const SAVE_PATH := "user://wojak_save_slot_%d.dat"
const ENCRYPTION_KEY := "ChurchOfPump2026Secure"

var current_slot: int = 1
var current_level: int = 1
var lives: int = 3
var faith: int = 0
var solana_orbs: int = 0
var health: int = 100

func _ready() -> void:
    # Load last used slot on startup (optional)
    pass

func save_game(slot: int = -1) -> void:
    if slot == -1: slot = current_slot
    var path = SAVE_PATH % slot
    
    var file = FileAccess.open_encrypted_with_pass(path, FileAccess.WRITE, ENCRYPTION_KEY)
    if file == null:
        push_error("Save failed on slot %d" % slot)
        return
    
    var data = {
        "current_level": current_level,
        "lives": lives,
        "faith": faith,
        "solana_orbs": solana_orbs,
        "health": health,
        "timestamp": Time.get_datetime_string_from_system()
    }
    
    file.store_string(JSON.stringify(data))
    file.close()
    print("Game saved to slot %d" % slot)

func load_game(slot: int = -1) -> bool:
    if slot == -1: slot = current_slot
    var path = SAVE_PATH % slot
    
    if not FileAccess.file_exists(path):
        return false
    
    var file = FileAccess.open_encrypted_with_pass(path, FileAccess.READ, ENCRYPTION_KEY)
    if file == null: return false
    
    var json_string = file.get_as_text()
    file.close()
    
    var data = JSON.parse_string(json_string)
    if data == null: return false
    
    current_level = data.get("current_level", 1)
    lives = data.get("lives", 3)
    faith = data.get("faith", 0)
    solana_orbs = data.get("solana_orbs", 0)
    health = data.get("health", 100)
    
    current_slot = slot
    print("Loaded slot %d" % slot)
    return true

func has_save(slot: int) -> bool:
    return FileAccess.file_exists(SAVE_PATH % slot)

func auto_save() -> void:
    save_game(current_slot)

func new_game(slot: int = 1) -> void:
    current_slot = slot
    current_level = 1
    lives = 3
    faith = 0
    solana_orbs = 0
    health = 100
    save_game(slot)
    extends Area2D

@export var checkpoint_id: int = 1

func _ready() -> void:
    body_entered.connect(_on_body_entered)

func _on_body_entered(body: Node2D) -> void:
    if body is Player:
        SaveSystem.current_level = get_tree().current_scene.scene_file_path.get_file().to_int()
        SaveSystem.auto_save()
        print("Checkpoint reached! Game auto-saved.")
        # Optional: Play particle effect or sound
        func take_damage(amount: int) -> void:
    health -= amount
    if health <= 0:
        health = 100
        lives -= 1
        SaveSystem.lives = lives
        SaveSystem.health = health
        SaveSystem.auto_save()
        
        if lives <= 0:
            _game_over()
        else:
            _respawn()

func _respawn() -> void:
    # Simple respawn at last checkpoint or start of level
    global_position = Vector2(150, 500)  # Adjust per level
    print("Respawned. Lives left:", lives)

func _game_over() -> void:
    get_tree().change_scene_to_file("res://scenes/ui/GameOver.tscn")
    Level1
├── ParallaxBackground (optional)
├── TileMapLayer_Ground
├── TileMapLayer_Details
├── Player (with Camera2D child)
├── Enemies
├── Bosses
│   └── REKT
├── Collectibles
├── Checkpoints
└── UI
    ├── SpellUI
    └── FaithMeter
    func _cast_fire_storm() -> void:
    var particles = preload("res://scenes/particles/FireStormParticles.tscn").instantiate()
    particles.global_position = player.global_position
    get_parent().add_child(particles)
    # Add screen shake
    get_tree().get_first_node_in_group("camera").apply_shake(35)
    # In MainMenu.gd
func _on_slot_pressed(slot: int) -> void:
    if SaveSystem.has_save(slot):
        SaveSystem.load_game(slot)
        var level_scene = "res://scenes/levels/Level%d.tscn" % SaveSystem.current_level
        get_tree().change_scene_to_file(level_scene)
    else:
        SaveSystem.new_game(slot)
        get_tree().change_scene_to_file("res://scenes/levels/Level1.tscn")
        func _on_delete_slot_pressed(slot: int) -> void:
    var path = "user://wojak_save_slot_%d.dat" % slot
    if FileAccess.file_exists(path):
        DirAccess.remove_absolute(path)
        #!/bin/bash
godot --headless --export-release "Web" ./build/index.html
echo "Build complete. Upload the 'build' folder to itch.io"
extends Camera2D
class_name GameCamera

@export var max_shake_strength: float = 40.0
@export var shake_fade: float = 8.0

var shake_strength: float = 0.0
var noise: FastNoiseLite

func _ready() -> void:
    noise = FastNoiseLite.new()
    noise.seed = randi()
    noise.frequency = 2.0

func _process(delta: float) -> void:
    if shake_strength > 0:
        shake_strength = lerp(shake_strength, 0.0, shake_fade * delta)
        
        var offset_x = noise.get_noise_1d(Time.get_ticks_msec() * 0.01) * shake_strength
        var offset_y = noise.get_noise_1d(Time.get_ticks_msec() * 0.01 + 100) * shake_strength
        offset = Vector2(offset_x, offset_y)

func apply_shake(strength: float = 25.0) -> void:
    shake_strength = min(strength, max_shake_strength)
    get_tree().get_first_node_in_group("camera").apply_shake(35)
    extends Node

func set_master_volume(value: float) -> void:
    AudioServer.set_bus_volume_db(0, linear_to_db(value))

func set_music_volume(value: float) -> void:
    AudioServer.set_bus_volume_db(1, linear_to_db(value))

func set_sfx_volume(value: float) -> void:
    AudioServer.set_bus_volume_db(2, linear_to_db(value))

func play_sfx(stream: AudioStream, volume_db: float = 0.0) -> void:
    var player = AudioStreamPlayer.new()
    player.stream = stream
    player.volume_db = volume_db
    player.bus = "SFX"
    add_child(player)
    player.play()
    player.finished.connect(player.queue_free)
    extends Area2D

@export var checkpoint_id: String = "checkpoint_1"
static var last_checkpoint_position: Vector2 = Vector2.ZERO

func _ready() -> void:
    body_entered.connect(_on_body_entered)

func _on_body_entered(body: Node2D) -> void:
    if body is Player:
        last_checkpoint_position = global_position
        SaveSystem.auto_save()
        print("Checkpoint saved:", checkpoint_id)
        func _respawn() -> void:
    if Checkpoint.last_checkpoint_position != Vector2.ZERO:
        global_position = Checkpoint.last_checkpoint_position
    else:
        global_position = Vector2(150, 500)  # Default spawn
        Level1 (Node2D)
├── ParallaxBackground (optional)
├── TileMapLayer_Ground
├── TileMapLayer_Platforms
├── TileMapLayer_Details
├── Player
│   └── Camera2D (with GameCamera script + AudioListener2D)
├── Enemies
│   ├── MadChadler (x6–8)
│   └── MadChadler_Gunner (x2–3)
├── Bosses
│   └── REKT
├── Collectibles
│   ├── HolyCandle (x5)
│   └── SolanaOrb (x4)
├── Checkpoints
│   ├── Checkpoint (Start)
│   └── Checkpoint (Mid-level)
├── Particles (for atmosphere)
└── UI (CanvasLayer)
    ├── SpellUI
    ├── FaithMeter
    └── HUD
    extends Control

@onready var master_slider: HSlider = $MasterSlider
@onready var music_slider: HSlider = $MusicSlider
@onready var sfx_slider: HSlider = $SFXSlider

func _ready() -> void:
    master_slider.value = db_to_linear(AudioServer.get_bus_volume_db(0))
    music_slider.value = db_to_linear(AudioServer.get_bus_volume_db(1))
    sfx_slider.value = db_to_linear(AudioServer.get_bus_volume_db(2))

func _on_master_slider_value_changed(value: float) -> void:
    AudioManager.set_master_volume(value)

func _on_music_slider_value_changed(value: float) -> void:
    AudioManager.set_music_volume(value)

func _on_sfx_slider_value_changed(value: float) -> void:
    AudioManager.set_sfx_volume(value)

func _on_delete_slot_pressed(slot: int) -> void:
    var path = "user://wojak_save_slot_%d.dat" % slot
    if FileAccess.file_exists(path):
        DirAccess.remove_absolute(path)
        print("Deleted save slot", slot)
        #!/bin/bash
echo "Building Wojak's Nightmares for itch.io..."
godot --headless --export-release "itch.io Web" ./build/index.html
echo "Build complete!"
echo "Upload the entire 'build' folder to itch.io"
extends Control

func _ready() -> void:
    # Play victory music / particles
    pass

func _on_main_menu_pressed() -> void:
    get_tree().change_scene_to_file("res://scenes/ui/MainMenu.tscn")

func _on_quit_pressed() -> void:
    get_tree().quit()
    extends "res://scripts/MadChadler.gd"

@export var shoot_range: float = 280.0
@export var fire_rate: float = 1.5

var can_shoot: bool = true

func _physics_process(delta: float) -> void:
    if player:
        var distance = global_position.distance_to(player.global_position)
        if distance > shoot_range:
            var dir = sign(player.global_position.x - global_position.x)
            velocity.x = dir * speed
        else:
            velocity.x = move_toward(velocity.x, 0, 200 * delta)
            if can_shoot:
                _shoot()
    move_and_slide()

func _shoot() -> void:
    can_shoot = false
    print("MadChadler_Gunner shoots!")
    # Spawn bullet projectile here
    await get_tree().create_timer(fire_rate).timeout
    can_shoot = true
    extends "res://scripts/MadChadler.gd"

func _ready() -> void:
    super._ready()
    speed = 80
    health = 90

func _physics_process(delta: float) -> void:
    if player:
        var dir = sign(player.global_position.x - global_position.x)
        velocity.x = dir * speed
    move_and_slide()

func take_damage(amount: int) -> void:
    super.take_damage(amount)
    if health < 40:
        speed = 140  # Enrage
        extends "res://scripts/MadChadler.gd"

@export var grenade_cooldown: float = 4.0
var can_grenade: bool = true

func _physics_process(delta: float) -> void:
    if player:
        var dir = sign(player.global_position.x - global_position.x)
        velocity.x = dir * (speed * 1.3)
    move_and_slide()

func take_damage(amount: int) -> void:
    super.take_damage(amount)
    if can_grenade and health < 50:
        _throw_grenade()

func _throw_grenade() -> void:
    can_grenade = false
    print("Elite throws grenade!")
    # Spawn grenade projectile
    await get_tree().create_timer(grenade_cooldown).timeout
    can_grenade = true
    extends StateMachineBoss

enum AttackType { CHARGE, ROAR, STOMP }

@onready var attack_timer: Timer = $AttackTimer

func _ready() -> void:
    super._ready()
    attack_timer.timeout.connect(_choose_attack)

func _physics_process(delta: float) -> void:
    if current_state == State.DEAD: return

    match current_state:
        State.CHASE:
            _chase_player()
        State.ATTACK:
            pass  # Handled in attack functions

func _chase_player() -> void:
    if player:
        var dir = sign(player.global_position.x - global_position.x)
        velocity.x = dir * normal_speed
    move_and_slide()

func _choose_attack() -> void:
    if current_state != State.CHASE: return
    change_state(State.ATTACK)
    
    var attack = AttackType.values().pick_random()
    match attack:
        AttackType.CHARGE: _do_charge()
        AttackType.ROAR: _do_roar()
        AttackType.STOMP: _do_stomp()

func _do_charge() -> void:
    print("REKT charges!")
    # Fast movement toward player + screen shake
    await get_tree().create_timer(1.5).timeout
    change_state(State.CHASE)

func _do_roar() -> void:
    print("REKT roars!")
    get_tree().get_first_node_in_group("camera").apply_shake(30)
    await get_tree().create_timer(1.0).timeout
    change_state(State.CHASE)

func _do_stomp() -> void:
    print("REKT stomps!")
    # Spawn shockwave or damage area
    await get_tree().create_timer(1.2).timeout
    change_state(State.CHASE)
    extends Control

@onready var stats_label: Label = $StatsLabel
@onready var particles: GPUParticles2D = $VictoryParticles

func _ready() -> void:
    particles.emitting = true
    var orbs = SaveSystem.solana_orbs
    var faith = SaveSystem.faith
    stats_label.text = "Solana Orbs Collected: %d\nFaith Gained: %d" % [orbs, faith]

func _on_main_menu_pressed() -> void:
    get_tree().change_scene_to_file("res://scenes/ui/MainMenu.tscn")
    extends Control

@onready var continue_button: Button = $ContinueButton
@onready var slot_buttons = [$Slot1Button, $Slot2Button, $Slot3Button]

func _ready() -> void:
    continue_button.visible = SaveSystem.has_save(SaveSystem.current_slot)
    
    for i in range(3):
        var btn = slot_buttons[i]
        btn.text = "Slot %d" % (i + 1)
        if SaveSystem.has_save(i + 1):
            btn.text += " (Saved)"
        btn.pressed.connect(_on_slot_selected.bind(i + 1))

func _on_slot_selected(slot: int) -> void:
    if SaveSystem.has_save(slot):
        SaveSystem.load_game(slot)
    else:
        SaveSystem.new_game(slot)
    
    var level_path = "res://scenes/levels/Level%d.tscn" % SaveSystem.current_level
    get_tree().change_scene_to_file(level_path)
    ParallaxBackground
├── Parallax2D (Far)          ← Speed Scale: 0.2   (Distant buildings / pillars)
├── Parallax2D (Mid)          ← Speed Scale: 0.5   (Neon signs / stained glass)
├── Parallax2D (Close)        ← Speed Scale: 0.8   (Floating debris / chains)
└── Parallax2D (Foreground)   ← Speed Scale: 1.0   (Optional rain or particles)
extends Parallax2D

@export var scroll_speed: float = 20.0

func _process(delta: float) -> void:
    scroll_offset.x -= scroll_speed * delta
    extends Area2D
class_name Projectile

@export var speed: float = 600.0
@export var damage: int = 15
@export var lifetime: float = 3.0
var direction: int = 1

func _ready() -> void:
    body_entered.connect(_on_body_entered)
    await get_tree().create_timer(lifetime).timeout
    queue_free()

func _physics_process(delta: float) -> void:
    position.x += speed * direction * delta

func _on_body_entered(body: Node2D) -> void:
    if body is Player:
        body.take_damage(damage)
        queue_free()
    elif body.is_in_group("enemy") or body.is_in_group("boss"):
        if body.has_method("take_damage"):
            body.take_damage(damage)
        queue_free()
        func _shoot_bullet() -> void:
    var bullet = preload("res://scenes/projectiles/Bullet.tscn").instantiate()
    bullet.global_position = global_position
    bullet.direction = sign(player.global_position.x - global_position.x)
    get_parent().add_child(bullet)
    extends StateMachineBoss

@export var lightning_damage: int = 30
var phase: int = 1
var minion_count: int = 0

@onready var attack_timer: Timer = $AttackTimer
@onready var sprite: AnimatedSprite2D = $AnimatedSprite2D

func _ready() -> void:
    super._ready()
    max_health = 650
    health = max_health
    attack_timer.timeout.connect(_perform_attack)

func take_damage(amount: int) -> void:
    super.take_damage(amount)
    _check_phase_transition()

func _check_phase_transition() -> void:
    if health <= max_health * 0.66 and phase == 1:
        phase = 2
        print("Legion Phase 2: Blood Wings Unleashed")
        get_tree().get_first_node_in_group("camera").apply_shake(45)
    elif health <= max_health * 0.33 and phase == 2:
        phase = 3
        print("Legion Phase 3: Full Liquidation")

func _perform_attack() -> void:
    if current_state == State.DEAD: return
    change_state(State.SPECIAL)

    match phase:
        1: _lightning_strike()
        2: _summon_minions()
        3: _blood_wing_dive()

    await get_tree().create_timer(3.5).timeout
    change_state(State.CHASE)

func _lightning_strike() -> void:
    print("Legion: Lightning Tempest!")
    get_tree().get_first_node_in_group("camera").apply_shake(50)
    # Spawn lightning projectiles in arc

func _summon_minions() -> void:
    print("Legion: Summons Mad Chadlers!")
    for i in range(2):
        var minion = preload("res://scenes/enemies/MadChadler.tscn").instantiate()
        minion.global_position = global_position + Vector2(randf_range(-150, 150), -50)
        get_parent().add_child(minion)

func _blood_wing_dive() -> void:
    print("Legion: Blood Wing Dive!")
    if player:
        var direction = sign(player.global_position.x - global_position.x)
        velocity.x = direction * 450
    get_tree().get_first_node_in_group("camera").apply_shake(60)
    extends Node

var master_volume: float = 1.0
var music_volume: float = 0.8
var sfx_volume: float = 1.0

func _ready() -> void:
    # Set default bus volumes
    set_master_volume(master_volume)
    set_music_volume(music_volume)
    set_sfx_volume(sfx_volume)

func set_master_volume(value: float) -> void:
    master_volume = value
    AudioServer.set_bus_volume_db(0, linear_to_db(value))

func set_music_volume(value: float) -> void:
    music_volume = value
    AudioServer.set_bus_volume_db(1, linear_to_db(value))

func set_sfx_volume(value: float) -> void:
    sfx_volume = value
    AudioServer.set_bus_volume_db(2, linear_to_db(value))

func play_sfx(sound: AudioStream, volume_db: float = 0.0) -> void:
    var player = AudioStreamPlayer.new()
    player.stream = sound
    player.volume_db = volume_db
    player.bus = "SFX"
    add_child(player)
    player.play()
    player.finished.connect(player.queue_free)
    Level2_GothicCathedral
├── ParallaxBackground
│   ├── Parallax2D_Far (Speed 0.2)
│   ├── Parallax2D_Mid (Speed 0.5)
│   └── Parallax2D_Fore (Speed 0.85)
├── TileMapLayer_Ground
├── TileMapLayer_Walls
├── TileMapLayer_Details (apply gothic lighting shader)
├── Player
├── Camera2D (GameCamera + AudioListener2D)
├── Enemies
├── Bosses
│   └── LegionOfLiquidation
├── Checkpoints
├── Collectibles
└── UI
@onready var victory_text: Label = $VictoryText

func _ready() -> void:
    victory_text.text = "The Church of the Pump stands eternal.\nYou have reclaimed the flame."
    godot --headless --export-release "Web" ./build/index.html
    func play_gunshot() -> void:
    var sound = preload("res://assets/audio/sfx/gunshot.wav")
    play_sfx(sound, -2)

func play_explosion() -> void:
    var sound = preload("res://assets/audio/sfx/explosion.wav")
    play_sfx(sound, 3)

func play_boss_roar() -> void:
    var sound = preload("res://assets/audio/sfx/boss_roar.wav")
    play_sfx(sound, 0)

func play_acid_sizzle() -> void:
    var sound = preload("res://assets/audio/sfx/acid_sizzle.wav")
    play_sfx(sound, -5)
    extends Area2D

@export var damage_per_second: int = 12
@export var duration: float = 6.0

var player_inside: bool = false

func _ready() -> void:
    body_entered.connect(_on_body_entered)
    body_exited.connect(_on_body_exited)
    await get_tree().create_timer(duration).timeout
    queue_free()

func _on_body_entered(body: Node2D) -> void:
    if body is Player:
        player_inside = true
        AudioManager.play_acid_sizzle()
        _start_damage_over_time(body)

func _on_body_exited(body: Node2D) -> void:
    if body is Player:
        player_inside = false

func _start_damage_over_time(player: Player) -> void:
    while player_inside and is_instance_valid(player):
        player.take_damage(damage_per_second)
        await get_tree().create_timer(1.0).timeout
       ParallaxBackground
├── Parallax2D_Far      (Speed Scale = 0.2)   ← Distant buildings / cathedral silhouette
├── Parallax2D_Mid      (Speed Scale = 0.5)   ← Neon signs / stained glass windows
├── Parallax2D_Close    (Speed Scale = 0.85)  ← Floating chains / debris
└── Parallax2D_Fore     (Speed Scale = 1.0)   ← Optional light rain or particles
extends Parallax2D

@export var scroll_speed: float = 30.0

func _process(delta: float) -> void:
    scroll_offset.x -= scroll_speed * delta
    @onready var victory_text: Label = $VictoryText

func _ready() -> void:
    victory_text.text = "The Church of the Pump stands eternal.\nYou have reclaimed the flame."
   func _ready() -> void:
    var splash = $SplashParticles
    splash.emitting = true
    await get_tree().create_timer(0.6).timeout
    splash.emitting = false
    extends Control

@onready var health_bar: ProgressBar = $HealthBar
@onready var name_label: Label = $BossName

var boss: Node = null

func _ready() -> void:
    visible = false
    add_to_group("ui")

func show_for_boss(target_boss: Node, boss_name: String) -> void:
    boss = target_boss
    name_label.text = boss_name
    visible = true
    health_bar.max_value = boss.max_health
    health_bar.value = boss.health

func _process(delta: float) -> void:
    if boss and is_instance_valid(boss):
        health_bar.value = boss.health
        if boss.health <= 0:
            queue_free()
    else:
        queue_free()
        func _ready() -> void:
    super._ready()
    var health_ui = preload("res://scenes/ui/BossHealthBar.tscn").instantiate()
    get_tree().get_first_node_in_group("ui").add_child(health_ui)
    health_ui.show_for_boss(self, "REKT")
    var splash = preload("res://scenes/particles/AcidSplash.tscn").instantiate()
splash.global_position = global_position
get_parent().add_child(splash)
splash.emitting = true
func die() -> void:
    change_state(State.DEAD)
    emit_signal("boss_died")
    
    # Play death animation if available
    if has_node("AnimatedSprite2D"):
        var sprite = $AnimatedSprite2D
        if sprite.sprite_frames.has_animation("death"):
            sprite.play("death")
            await sprite.animation_finished
    
    # Optional: Death particles
    spawn_death_particles()
    
    queue_free()

func spawn_death_particles() -> void:
    var death_particles = preload("res://scenes/particles/BossDeathParticles.tscn").instantiate()
    death_particles.global_position = global_position
    get_parent().add_child(death_particles)
    func apply_acid_shake(strength: float = 20.0) -> void:
    shake_strength = strength * 1.2  # Slightly stronger + longer feel
    # Optional: Add slight green flash overlay if you have a CanvasLayer
    # In AcidPool or when acid hits player
get_tree().get_first_node_in_group("camera").apply_acid_shake(25)
AudioManager.play_acid_sizzle()
func die() -> void:
    change_state(State.DEAD)
    emit_signal("boss_died")
    
    # Acid death sequence
    velocity = Vector2.ZERO
    
    # Play acid death particles
    var acid_death = preload("res://scenes/particles/AcidDeathExplosion.tscn").instantiate()
    acid_death.global_position = global_position
    get_parent().add_child(acid_death)
    
    # Strong acid screen shake
    get_tree().get_first_node_in_group("camera").apply_acid_shake(60)
    
    # Optional: Play special acid death sound
    AudioManager.play_explosion()
    
    # Fade out sprite if it exists
    if has_node("AnimatedSprite2D"):
        var tween = create_tween()
        tween.tween_property($AnimatedSprite2D, "modulate:a", 0.0, 1.5)
    
    await get_tree().create_timer(2.0).timeout
    queue_free()
    # In AcidPool.gd - attach and configure in inspector or code
func setup_bubbles() -> void:
    var bubbles = $Bubbles
    bubbles.amount = 30
    bubbles.lifetime = 2.0
    bubbles.emission_shape = GPUParticles2D.EMISSION_SHAPE_RECTANGLE
    bubbles.emission_rect_extents = Vector2(60, 20)
    bubbles.velocity.x_curve = null
    bubbles.velocity.y_min = -60
    bubbles.velocity.y_max = -20
    bubbles.color = Color(0, 1, 0.6, 0.7)
    func _ready() -> void:
    var splash = $Splash
    splash.emitting = true
    AudioManager.play_acid_sizzle()
    
    await get_tree().create_timer(0.8).timeout
    splash.emitting = false
    extends Control

@onready var health_bar: ProgressBar = $HealthBar
@onready var name_label: Label = $BossName

func show_for_boss(boss: Node, boss_name: String) -> void:
    name_label.text = boss_name
    health_bar.max_value = boss.max_health
    health_bar.value = boss.health
    visible = true
    
    # Acid green color
    health_bar.modulate = Color(0.2, 1.0, 0.5)

func _process(delta: float) -> void:
    if is_instance_valid(boss):
        health_bar.value = boss.health
    else:
        queue_free()
        var health_ui = preload("res://scenes/ui/AcidBossHealthBar.tscn").instantiate()
get_tree().get_first_node_in_group("ui").add_child(health_ui)
health_ui.show_for_boss(self, "LEGION OF LIQUIDATION")
# In AcidPool.gd
@onready var anim_player: AnimationPlayer = $AnimationPlayer

func _ready() -> void:
    anim_player.play("acid_splash")
    func die() -> void:
    change_state(State.DEAD)
    
    if has_node("AnimatedSprite2D"):
        var sprite = $AnimatedSprite2D
        if sprite.sprite_frames.has_animation("death"):
            sprite.play("death")
            await sprite.animation_finished
    
    # Acid death particles + shake
    spawn_acid_death_effects()
    queue_free()

func spawn_acid_death_effects() -> void:
    var death_fx = preload("res://scenes/particles/AcidDeathExplosion.tscn").instantiate()
    death_fx.global_position = global_position
    get_parent().add_child(death_fx)
    
    get_tree().get_first_node_in_group("camera").apply_acid_shake(55)
    shader_type particles;

uniform vec4 acid_color : source_color = vec4(0.0, 1.0, 0.6, 0.9);
uniform float glow_intensity : hint_range(0.0, 5.0) = 1.5;
uniform float dissolve_speed : hint_range(0.0, 10.0) = 2.0;

void vertex() {
    // Slight wobble for organic acid feel
    VERTEX.x += sin(TIME * 3.0 + VERTEX.y * 0.1) * 2.0;
}

void fragment() {
    vec4 color = acid_color;
    
    // Glowing edge effect
    float glow = sin(UV.y * 10.0 + TIME * dissolve_speed) * 0.5 + 0.5;
    color.rgb += glow * glow_intensity * vec3(0.0, 0.4, 0.2);
    
    // Fade out over particle lifetime
    color.a *= (1.0 - CUSTOM.x); // CUSTOM.x = particle age (0 to 1)
    
    COLOR = color;
}
extends Camera2D
class_name GameCamera

@export var max_shake: float = 50.0
@export var shake_fade: float = 6.0

var shake_strength: float = 0.0
var noise: FastNoiseLite

func _ready() -> void:
    noise = FastNoiseLite.new()
    noise.seed = randi()
    noise.frequency = 1.8

func _process(delta: float) -> void:
    if shake_strength > 0:
        shake_strength = lerp(shake_strength, 0.0, shake_fade * delta)
        
        var shake_offset = Vector2(
            noise.get_noise_1d(Time.get_ticks_msec() * 0.015) * shake_strength,
            noise.get_noise_1d(Time.get_ticks_msec() * 0.015 + 50) * shake_strength
        )
        offset = shake_offset

func apply_shake(strength: float = 20.0) -> void:
    shake_strength = min(strength, max_shake)

func apply_acid_shake(strength: float = 25.0) -> void:
    # Acid feels more intense and slightly longer
    shake_strength = min(strength * 1.3, max_shake)
    shake_fade = 4.5  # Slower fade for acid
    # Normal explosion
camera.apply_shake(30)

# Acid attack
camera.apply_acid_shake(35)
extends Node2D

@onready var camera: Camera2D = $Camera2D
@onready var fade_rect: ColorRect = $FadeRect
@onready var victory_label: Label = $VictoryLabel
@onready var anim_player: AnimationPlayer = $AnimationPlayer

func _ready() -> void:
    victory_label.text = "The Church of the Pump stands eternal.\nYou have reclaimed the flame."
    victory_label.modulate.a = 0.0
    
    # Start cinematic sequence
    start_cinematic()

func start_cinematic() -> void:
    # Slow camera pan (if you have a wide background)
    var tween = create_tween()
    tween.tween_property(camera, "position:x", camera.position.x + 300, 6.0)
    
    # Fade in text
    var text_tween = create_tween()
    text_tween.tween_property(victory_label, "modulate:a", 1.0, 2.0)
    
    # Optional: Play victory particles
    if has_node("VictoryParticles"):
        $VictoryParticles.emitting = true
    
    # Wait then allow input to return to menu
    await get_tree().create_timer(8.0).timeout
    fade_to_menu()

func fade_to_menu() -> void:
    var fade_tween = create_tween()
    fade_tween.tween_property(fade_rect, "color:a", 1.0, 2.0)
    await fade_tween.finished
    get_tree().change_scene_to_file("res://scenes/ui/MainMenu.tscn")
    # Example setup in _ready() of AcidPool.gd
func setup_acid_particles() -> void:
    var bubbles = $Bubbles
    var material = bubbles.process_material as ParticleProcessMaterial
    
    if material:
        material.emission_shape = ParticleProcessMaterial.EMISSION_SHAPE_RECTANGLE
        material.emission_rect_extents = Vector2(70, 25)
        
        material.direction = Vector3(0, -1, 0)
        material.spread = 15.0
        material.initial_velocity_min = 25.0
        material.initial_velocity_max = 55.0
        
        material.gravity = Vector3(0, -20, 0)           # Gentle upward float
        material.damping_min = 0.8
        material.damping_max = 1.2
        
        material.scale_min = 0.6
        material.scale_max = 1.4
        material.scale_curve = null                     # Can add curve for shrinking
        
        material.color = Color(0.0, 1.0, 0.55, 0.85)    # Acid green
        shader_type canvas_item;

uniform float intensity : hint_range(0.0, 1.0) = 0.6;
uniform float time_scale : hint_range(0.1, 5.0) = 2.0;

void fragment() {
    vec2 uv = SCREEN_UV;
    
    // Subtle acid distortion
    float distortion = sin(uv.y * 40.0 + TIME * time_scale) * 0.003;
    uv.x += distortion * intensity;
    
    vec3 color = texture(SCREEN_TEXTURE, uv).rgb;
    
    // Slight green tint
    color.g = min(color.g + intensity * 0.15, 1.0);
    
    COLOR = vec4(color, 1.0);
}
shader_type canvas_item;

uniform float intensity : hint_range(0.0, 1.0) = 0.6;
uniform float time_scale : hint_range(0.1, 5.0) = 2.0;

void fragment() {
    vec2 uv = SCREEN_UV;
    
    // Subtle acid distortion
    float distortion = sin(uv.y * 40.0 + TIME * time_scale) * 0.003;
    uv.x += distortion * intensity;
    
    vec3 color = texture(SCREEN_TEXTURE, uv).rgb;
    
    // Slight green tint
    color.g = min(color.g + intensity * 0.15, 1.0);
    
    COLOR = vec4(color, 1.0);
}
shader_type canvas_item;

uniform float intensity : hint_range(0.0, 1.0) = 0.5;
uniform float aberration_amount : hint_range(0.0, 0.05) = 0.008;

void fragment() {
    vec2 uv = SCREEN_UV;
    
    // Chromatic aberration
    float r = texture(SCREEN_TEXTURE, uv + vec2(aberration_amount, 0.0) * intensity).r;
    float g = texture(SCREEN_TEXTURE, uv).g;
    float b = texture(SCREEN_TEXTURE, uv - vec2(aberration_amount, 0.0) * intensity).b;
    
    vec3 color = vec3(r, g, b);
    
    COLOR = vec4(color, 1.0);
}
# Example: Trigger during acid attack
func trigger_acid_effect(duration: float = 1.5) -> void:
    var aberration = $ChromaticAberration  # Your ColorRect
    var tween = create_tween()
    tween.tween_property(aberration.material, "shader_parameter/intensity", 0.7, 0.2)
    tween.tween_property(aberration.material, "shader_parameter/intensity", 0.0, duration)
    # In AcidPool.gd or a dedicated setup function
func setup_acid_particles() -> void:
    # === BUBBLES ===
    var bubbles = $Bubbles
    var mat = bubbles.process_material as ParticleProcessMaterial
    
    if mat:
        mat.emission_shape = ParticleProcessMaterial.EMISSION_SHAPE_RECTANGLE
        mat.emission_rect_extents = Vector2(65, 22)
        mat.direction = Vector3(0, -1, 0)
        mat.spread = 18.0
        mat.initial_velocity_min = 30.0
        mat.initial_velocity_max = 65.0
        mat.gravity = Vector3(0, -25, 0)
        mat.damping_min = 0.6
        mat.damping_max = 1.1
        mat.scale_min = 0.5
        mat.scale_max = 1.6
        mat.color = Color(0.0, 0.95, 0.55, 0.9)
    
    # === MIST ===
    var mist = $Mist
    var mist_mat = mist.process_material as ParticleProcessMaterial
    
    if mist_mat:
        mist_mat.emission_shape = ParticleProcessMaterial.EMISSION_SHAPE_RECTANGLE
        mist_mat.emission_rect_extents = Vector2(80, 30)
        mist_mat.direction = Vector3(0, -0.3, 0)
        mist_mat.spread = 40.0
        mist_mat.initial_velocity_min = 5.0
        mist_mat.initial_velocity_max = 15.0
        mist_mat.gravity = Vector3(0, -8, 0)
        mist_mat.damping_min = 0.3
        mist_mat.damping_max = 0.6
        mist_mat.scale_min = 1.2
        mist_mat.scale_max = 2.8
        mist_mat.color = Color(0.0, 1.0, 0.6, 0.35)
       shader_type canvas_item;

uniform sampler2D acid_texture : hint_default_white;
uniform vec4 acid_color : source_color = vec4(0.0, 1.0, 0.55, 1.0);
uniform float glow_strength : hint_range(0.0, 3.0) = 1.2;
uniform float wobble_speed : hint_range(0.1, 10.0) = 3.0;

void fragment() {
    // Wobble UV using sine
    vec2 wobble_uv = UV;
    wobble_uv.x += sin(UV.y * 12.0 + TIME * wobble_speed) * 0.015;
    wobble_uv.y += cos(UV.x * 8.0 + TIME * wobble_speed) * 0.01;

    vec4 tex = texture(acid_texture, wobble_uv);

    // Base acid color
    vec3 color = tex.rgb * acid_color.rgb;

    // Fresnel-style edge glow
    float fresnel = pow(1.0 - abs(dot(NORMAL, vec3(0.0, 0.0, 1.0))), 2.0);
    color += fresnel * acid_color.rgb * glow_strength;

    COLOR = vec4(color, tex.a * acid_color.a);
}
void fragment() {
    vec2 uv = UV;
    
    // Organic wobble using sine + fract
    float wobble = sin(TIME * 2.0 + fract(uv.y * 5.0) * 6.28) * 0.02;
    uv.x += wobble;
    
    // Smooth fade based on distance from center
    float dist = length(uv - vec2(0.5));
    float mask = smoothstep(0.6, 0.3, dist);
    
    COLOR = vec4(0.0, 1.0, 0.6, mask);
}
shader_type canvas_item;

uniform float intensity : hint_range(0.0, 2.0) = 0.6;
uniform float aberration_strength : hint_range(0.0, 0.03) = 0.012;
uniform float vignette : hint_range(0.0, 1.0) = 0.3;

void fragment() {
    vec2 uv = SCREEN_UV;
    
    // Chromatic aberration with slight curve
    float offset = aberration_strength * intensity;
    float r = texture(SCREEN_TEXTURE, uv + vec2(offset, 0.0)).r;
    float g = texture(SCREEN_TEXTURE, uv).g;
    float b = texture(SCREEN_TEXTURE, uv - vec2(offset * 0.8, 0.0)).b;
    
    vec3 color = vec3(r, g, b);
    
    // Optional vignette
    float vig = smoothstep(0.7, 0.3, length(uv - 0.5));
    color *= mix(1.0, vig, vignette);
    
    COLOR = vec4(color, 1.0);
}
shader_type canvas_item;

uniform float intensity : hint_range(0.0, 1.5) = 0.7;
uniform float distortion : hint_range(0.0, 0.02) = 0.008;
uniform float green_boost : hint_range(0.0, 0.5) = 0.12;

void fragment() {
    vec2 uv = SCREEN_UV;
    
    // Acid distortion
    float wave = sin(uv.y * 35.0 + TIME * 4.0) * distortion * intensity;
    uv.x += wave;
    
    vec3 color = texture(SCREEN_TEXTURE, uv).rgb;
    
    // Green acid tint
    color.g = min(color.g + green_boost * intensity, 1.0);
    
    // Subtle contrast boost
    color = pow(color, vec3(0.95));
    
    COLOR = vec4(color, 1.0);
}
# Example: Trigger acid screen effect
func trigger_acid_screen(duration: float = 1.8) -> void:
    var effect = $AcidScreenEffect  # Your ColorRect
    var tween = create_tween()
    tween.tween_property(effect.material, "shader_parameter/intensity", 1.0, 0.3)
    tween.tween_property(effect.material, "shader_parameter/intensity", 0.0, duration)
    var noise_texture = NoiseTexture2D.new()
noise_texture.noise = FastNoiseLite.new()
noise_texture.width = 512
noise_texture.height = 512

# Assign to your material
material.set_shader_parameter("noise_texture", noise_texture)
uniform sampler2D noise_texture : hint_default_black;

void fragment() {
    float n = texture(noise_texture, UV * 3.0).r;
    
    // Use noise for acid distortion
    vec2 distorted_uv = UV;
    distorted_uv.x += n * 0.03;
    
    COLOR = texture(TEXTURE, distorted_uv);
}
// Simple 2D value noise
float hash(vec2 p) {
    return fract(sin(dot(p, vec2(127.1, 311.7))) * 43758.5453123);
}

float noise(vec2 p) {
    vec2 i = floor(p);
    vec2 f = fract(p);
    
    float a = hash(i);
    float b = hash(i + vec2(1.0, 0.0));
    float c = hash(i + vec2(0.0, 1.0));
    float d = hash(i + vec2(1.0, 1.0));
    
    vec2 u = f * f * (3.0 - 2.0 * f);
    
    return mix(a, b, u.x) + (c - a) * u.y * (1.0 - u.x) + (d - b) * u.x * u.y;
}

// Fractal Brownian Motion (fBm) - layered noise
float fbm(vec2 p) {
    float value = 0.0;
    float amplitude = 0.5;
    float frequency = 1.0;
    
    for (int i = 0; i < 5; i++) {
        value += amplitude * noise(p * frequency);
        amplitude *= 0.5;
        frequency *= 2.0;
    }
    return value;
}
void fragment() {
    float n = fbm(UV * 4.0 + TIME * 0.3);
    float alpha = smoothstep(0.3, 0.7, n);
    
    COLOR = vec4(0.0, 1.0, 0.6, alpha * 0.4);
}
shader_type canvas_item;     // For 2D / UI
shader_type spatial;         // For 3D
shader_type particles;       // For particle shaders
uniform float intensity : hint_range(0.0, 1.0) = 0.5;
uniform vec4 color : source_color;
uniform sampler2D texture : hint_default_white;
varying vec2 world_uv;
precision mediump float;   // Usually not needed in Godot 4
shader_type canvas_item;

uniform float intensity : hint_range(0.0, 2.0) = 1.0;
uniform vec4 tint_color : source_color = vec4(1.0);

void vertex() {
    // Vertex code here
}

void fragment() {
    // Fragment code here
}
// Simple 2D hash function
float hash(vec2 p) {
    return fract(sin(dot(p, vec2(127.1, 311.7))) * 43758.5453123);
}

// 2D Value Noise
float noise(vec2 p) {
    vec2 i = floor(p);
    vec2 f = fract(p);
    
    float a = hash(i);
    float b = hash(i + vec2(1.0, 0.0));
    float c = hash(i + vec2(0.0, 1.0));
    float d = hash(i + vec2(1.0, 1.0));
    
    vec2 u = f * f * (3.0 - 2.0 * f);
    
    return mix(a, b, u.x) + (c - a) * u.y * (1.0 - u.x) + (d - b) * u.x * u.y;
}

// Fractal Brownian Motion (Animated)
float fbm(vec2 p, float time) {
    float value = 0.0;
    float amplitude = 0.5;
    float frequency = 1.0;
    
    for (int i = 0; i < 6; i++) {
        value += amplitude * noise(p * frequency + time * 0.3);
        amplitude *= 0.5;
        frequency *= 2.0;
    }
    return value;
}

void fragment() {
    vec2 uv = UV * 3.0;
    
    float n = fbm(uv, TIME);
    
    // Use for acid distortion or alpha
    float distortion = (n - 0.5) * 0.04;
    vec2 final_uv = UV + vec2(distortion, distortion * 0.5);
    
    COLOR = texture(TEXTURE, final_uv);
}
# In your AcidPool or effect script
var noise_tex = NoiseTexture2D.new()
noise_tex.noise = FastNoiseLite.new()
noise_tex.width = 512
noise_tex.height = 512
noise_tex.noise.fractal_octaves = 6
noise_tex.noise.fractal_lacunarity = 2.0

material.set_shader_parameter("noise_tex", noise_tex)
material.set_shader_parameter("time_scale", 0.4)
uniform sampler2D noise_tex;
uniform float time_scale = 0.4;

void fragment() {
    vec2 animated_uv = UV * 2.5 + TIME * time_scale;
    float n = texture(noise_tex, animated_uv).r;
    
    // Animated acid distortion
    vec2 distorted = UV + (n - 0.5) * 0.035;
    COLOR = texture(TEXTURE, distorted);
}
uniform sampler2D noise_tex;
uniform float time_scale = 0.4;

void fragment() {
    vec2 animated_uv = UV * 2.5 + TIME * time_scale;
    float n = texture(noise_tex, animated_uv).r;
    
    // Animated acid distortion
    vec2 distorted = UV + (n - 0.5) * 0.035;
    COLOR = texture(TEXTURE, distorted);
}
# In your AcidPool or effect script
var noise_tex = NoiseTexture2D.new()
noise_tex.noise = FastNoiseLite.new()
noise_tex.width = 512
noise_tex.height = 512
noise_tex.noise.fractal_octaves = 6
noise_tex.noise.fractal_lacunarity = 2.0

material.set_shader_parameter("noise_tex", noise_tex)
material.set_shader_parameter("time_scale", 0.4)
uniform sampler2D noise_tex;
uniform float time_scale = 0.4;

void fragment() {
    vec2 animated_uv = UV * 2.5 + TIME * time_scale;
    float n = texture(noise_tex, animated_uv).r;
    
    // Animated acid distortion
    vec2 distorted = UV + (n - 0.5) * 0.035;
    COLOR = texture(TEXTURE, distorted);
}
shader_type canvas_item;

uniform float intensity : hint_range(0.0, 2.0) = 0.0;
uniform float distortion_strength : hint_range(0.0, 0.015) = 0.007;
uniform float green_tint : hint_range(0.0, 0.4) = 0.15;

void fragment() {
    vec2 uv = SCREEN_UV;
    
    // Animated acid distortion
    float wave = sin(uv.y * 28.0 + TIME * 3.5) * distortion_strength * intensity;
    uv.x += wave;
    
    vec3 color = texture(SCREEN_TEXTURE, uv).rgb;
    
    // Acid green color shift
    color.g = min(color.g + green_tint * intensity, 1.0);
    
    // Slight contrast boost
    color = pow(color, vec3(0.92));
    
    COLOR = vec4(color, 1.0);
}
func trigger_acid_post_effect(duration: float = 2.0) -> void:
    var effect = $PostProcessing/AcidEffect
    var tween = create_tween()
    tween.tween_property(effect.material, "shader_parameter/intensity", 1.0, 0.25)
    tween.tween_property(effect.material, "shader_parameter/intensity", 0.0, duration)
    // Perlin noise implementation
vec2 fade(vec2 t) {
    return t * t * t * (t * (t * 6.0 - 15.0) + 10.0);
}

float perlin(vec2 p) {
    vec2 i = floor(p);
    vec2 f = fract(p);
    
    // Four corners
    float a = hash(i);
    float b = hash(i + vec2(1.0, 0.0));
    float c = hash(i + vec2(0.0, 1.0));
    float d = hash(i + vec2(1.0, 1.0));
    
    vec2 u = fade(f);
    
    return mix(
        mix(a, b, u.x),
        mix(c, d, u.x),
        u.y
    );
}

// Fractal Perlin (fBm variant)
float perlin_fbm(vec2 p, int octaves) {
    float value = 0.0;
    float amplitude = 0.5;
    float frequency = 1.0;
    
    for (int i = 0; i < octaves; i++) {
        value += amplitude * perlin(p * frequency);
        amplitude *= 0.5;
        frequency *= 2.0;
    }
    return value;
}

void fragment() {
    vec2 uv = UV * 4.0 + TIME * 0.2;
    
    float n = perlin_fbm(uv, 5);
    
    // Use for acid distortion or organic movement
    vec2 distorted = UV + (n - 0.5) * 0.025;
    
    COLOR = texture(TEXTURE, distorted);
}
var noise = FastNoiseLite.new()
noise.noise_type = FastNoiseLite.TYPE_PERLIN
noise.fractal_octaves = 5
noise.frequency = 0.02

var noise_tex = NoiseTexture2D.new()
noise_tex.noise = noise
noise_tex.width = 512
noise_tex.height = 512

# Assign to shader
material.set_shader_parameter("perlin_noise", noise_tex)
uniform sampler2D perlin_noise;
uniform float time_scale = 0.25;

void fragment() {
    vec2 animated_uv = UV * 3.0 + TIME * time_scale;
    float n = texture(perlin_noise, animated_uv).r;
    
    // Smooth acid-like distortion
    vec2 offset = (n - 0.5) * 0.03;
    COLOR = texture(TEXTURE, UV + offset);
}
@tool
extends CompositorEffect
class_name AcidDistortionEffect

@export var intensity: float = 0.0

func _render_callback(p_render_data: RenderData, p_render_scene_buffers: RenderSceneBuffersRD) -> void:
    # This is where you would apply custom rendering
    # For now, we recommend using a full-screen shader approach for simplicity
    pass
    # Control from code
func set_acid_intensity(value: float) -> void:
    $CanvasLayer/ColorRect.material.set_shader_parameter("intensity", value)
    shader_type canvas_item;

uniform float intensity : hint_range(0.0, 2.0) = 0.0;
uniform float aberration_strength : hint_range(0.0, 0.04) = 0.012;
uniform float vignette_strength : hint_range(0.0, 1.0) = 0.25;

void fragment() {
    vec2 uv = SCREEN_UV;
    
    // Chromatic aberration
    float offset = aberration_strength * intensity;
    float r = texture(SCREEN_TEXTURE, uv + vec2(offset, 0.0)).r;
    float g = texture(SCREEN_TEXTURE, uv).g;
    float b = texture(SCREEN_TEXTURE, uv - vec2(offset * 0.7, 0.0)).b;
    
    vec3 color = vec3(r, g, b);
    
    // Subtle vignette
    float vig = smoothstep(0.8, 0.4, length(uv - 0.5));
    color *= mix(1.0, vig, vignette_strength * intensity);
    
    COLOR = vec4(color, 1.0);
}
func trigger_chromatic_aberration(strength: float = 0.8, duration: float = 1.5) -> void:
    var effect = $CanvasLayer/ChromaticAberration  # Your ColorRect
    var tween = create_tween()
    tween.tween_property(effect.material, "shader_parameter/intensity", strength, 0.2)
    tween.tween_property(effect.material, "shader_parameter/intensity", 0.0, duration)
    func create_noise_texture(noise_type: FastNoiseLite.NoiseType) -> NoiseTexture2D:
    var noise = FastNoiseLite.new()
    noise.noise_type = noise_type
    noise.fractal_octaves = 5
    noise.frequency = 0.025
    
    # Optional: Add fractal layering
    noise.fractal_type = FastNoiseLite.FRACTAL_FBM
    
    var texture = NoiseTexture2D.new()
    texture.noise = noise
    texture.width = 512
    texture.height = 512
    
    return texture
    # Organic acid distortion (Recommended)
var acid_noise = create_noise_texture(FastNoiseLite.TYPE_SIMPLEX_SMOOTH)

# More structured / cellular acid look
var cellular_noise = create_noise_texture(FastNoiseLite.TYPE_CELLULAR)

# Classic smooth look
var perlin_noise = create_noise_texture(FastNoiseLite.TYPE_PERLIN)
uniform sampler2D noise_texture;

void fragment() {
    float n = texture(noise_texture, UV * 3.0 + TIME * 0.2).r;
    
    // Use noise for acid distortion
    vec2 offset = (n - 0.5) * 0.035;
    vec2 distorted_uv = UV + offset;
    
    COLOR = texture(TEXTURE, distorted_uv);
}
extends Node2D

@onready var camera: Camera2D = $Camera2D
@onready var fade_rect: ColorRect = $ColorRect
@onready var victory_label: Label = $VictoryLabel
@onready var stats_label: Label = $StatsLabel
@onready var particles: GPUParticles2D = $VictoryParticles
@onready var anim_player: AnimationPlayer = $AnimationPlayer
@onready var return_button: Button = $ReturnButton

func _ready() -> void:
    fade_rect.color.a = 1.0
    victory_label.text = "The Church of the Pump stands eternal.\nYou have reclaimed the flame."
    victory_label.modulate.a = 0.0
    stats_label.modulate.a = 0.0
    return_button.visible = false
    
    # Show stats from SaveSystem
    var orbs = SaveSystem.solana_orbs
    var faith = SaveSystem.faith
    stats_label.text = "Solana Orbs Collected: %d\nFaith Restored: %d" % [orbs, faith]
    
    particles.emitting = false
    start_cinematic()

func start_cinematic() -> void:
    # Slow camera pan + fade in
    var cam_tween = create_tween()
    cam_tween.tween_property(camera, "position:x", camera.position.x + 400, 7.0)
    
    # Fade out black overlay
    var fade_tween = create_tween()
    fade_tween.tween_property(fade_rect, "color:a", 0.0, 2.5)
    
    # Fade in victory text + particles
    await get_tree().create_timer(2.0).timeout
    particles.emitting = true
    
    var text_tween = create_tween()
    text_tween.tween_property(victory_label, "modulate:a", 1.0, 2.0)
    
    await get_tree().create_timer(1.5).timeout
    var stats_tween = create_tween()
    stats_tween.tween_property(stats_label, "modulate:a", 1.0, 1.5)
    
    await get_tree().create_timer(3.0).timeout
    return_button.visible = true
    return_button.pressed.connect(_on_return_pressed)

func _on_return_pressed() -> void:
    var fade_tween = create_tween()
    fade_tween.tween_property(fade_rect, "color:a", 1.0, 1.5)
    await fade_tween.finished
    get_tree().change_scene_to_file("res://scenes/ui/MainMenu.tscn")
extends Control

@onready var continue_button: Button = $ContinueButton
@onready var slot_buttons: Array[Button] = [$Slot1Button, $Slot2Button, $Slot3Button]

func _ready() -> void:
    _update_slot_buttons()
    continue_button.visible = SaveSystem.has_save(SaveSystem.current_slot)
    continue_button.pressed.connect(_on_continue_pressed)

func _update_slot_buttons() -> void:
    for i in range(3):
        var btn = slot_buttons[i]
        var slot_num = i + 1
        if SaveSystem.has_save(slot_num):
            btn.text = "Slot %d (Saved)" % slot_num
        else:
            btn.text = "Slot %d (New)" % slot_num
        btn.pressed.connect(_on_slot_selected.bind(slot_num))

func _on_slot_selected(slot: int) -> void:
    if SaveSystem.has_save(slot):
        SaveSystem.load_game(slot)
    else:
        SaveSystem.new_game(slot)
    get_tree().change_scene_to_file("res://scenes/levels/Level%d.tscn" % SaveSystem.current_level)

func _on_continue_pressed() -> void:
    SaveSystem.load_game(SaveSystem.current_slot)
    get_tree().change_scene_to_file("res://scenes/levels/Level%d.tscn" % SaveSystem.current_level)
#!/bin/bash
godot --headless --export-release "itch.io Web" ./build/index.html
echo "Build ready! Upload the entire 'build' folder to itch.io"
Wojaks_Nightmares/
├── project.godot
├── export_presets.cfg
├── .godot/                          # Auto-generated (ignore in git)
├── assets/
│   ├── sprites/
│   │   ├── player/
│   │   ├── enemies/
│   │   ├── bosses/
│   │   └── ui/
│   ├── textures/
│   │   ├── posters/                 # Church of the Pump poster
│   │   └── backgrounds/
│   ├── audio/
│   │   ├── music/
│   │   └── sfx/
│   └── particles/
├── scenes/
│   ├── player/
│   │   └── Player.tscn
│   ├── enemies/
│   │   ├── MadChadler.tscn
│   │   ├── MadChadler_Gunner.tscn
│   │   ├── MadChadler_Heavy.tscn
│   │   └── MadChadler_Elite.tscn
│   ├── bosses/
│   │   ├── REKT.tscn
│   │   ├── FalseMoonAttack.tscn
│   │   ├── RUGPULL_Overlord.tscn
│   │   └── LegionOfLiquidation.tscn
│   ├── levels/
│   │   ├── Level1.tscn
│   │   ├── Level2_GothicCathedral.tscn
│   │   ├── Level3_AcidicVoid.tscn
│   │   └── Level4_FinalCathedral.tscn
│   ├── ui/
│   │   ├── MainMenu.tscn
│   │   ├── OptionsMenu.tscn
│   │   ├── SpellUI.tscn
│   │   ├── FaithMeter.tscn
│   │   ├── BossHealthBar.tscn
│   │   ├── GameOver.tscn
│   │   └── VictoryScreen.tscn
│   ├── cinematics/
│   │   └── EndingCinematic.tscn
│   └── checkpoints/
│       └── Checkpoint.tscn
├── scripts/
│   ├── player/
│   │   └── Player.gd
│   ├── enemies/
│   ├── bosses/
│   ├── ui/
│   ├── systems/
│   │   ├── SaveSystem.gd          # Autoload
│   │   ├── AudioManager.gd        # Autoload
│   │   └── DifficultyManager.gd   # Autoload
│   └── cinematics/
│       └── EndingCinematic.gd
├── shaders/
│   ├── acid_particle.gdshader
│   ├── chromatic_aberration.gdshader
│   ├── acid_screen_effect.gdshader
│   └── gothic_lighting.gdshader
├── autoloads/                       # (optional folder for clarity)
├── addons/                          # If you use any
└── build/                           # Export folder (gitignored)
Level2_GothicCathedral (Node2D)
├── WorldEnvironment
├── ParallaxBackground
│   ├── Parallax2D_Far          (Speed Scale: 0.2)
│   ├── Parallax2D_Mid          (Speed Scale: 0.5)
│   └── Parallax2D_Fore         (Speed Scale: 0.85)
├── TileMapLayer_Ground
├── TileMapLayer_Platforms
├── TileMapLayer_Walls
├── TileMapLayer_Details        ← Apply gothic lighting shader here
├── Player
│   └── Camera2D (GameCamera + AudioListener2D)
├── Enemies (Node)
├── Bosses (Node)
├── Collectibles (Node)
├── Checkpoints (Node)
├── Particles (Node)            ← Ambient candle smoke + floating embers
└── UI (CanvasLayer)
    ├── SpellUI
    ├── FaithMeter
    └── BossHealthBar
extends StateMachineBoss

@export var acid_damage: int = 18
@export var teleport_cooldown: float = 4.0
@export var acid_pool_cooldown: float = 3.5

var can_teleport: bool = true
var can_spawn_acid: bool = true

@onready var attack_timer: Timer = $AttackTimer
@onready var sprite: AnimatedSprite2D = $AnimatedSprite2D

func _ready() -> void:
    super._ready()
    max_health = 420
    health = max_health
    attack_timer.timeout.connect(_perform_attack)

func _physics_process(delta: float) -> void:
    if current_state == State.DEAD:
        return
    
    if player and current_state == State.CHASE:
        var dir = sign(player.global_position.x - global_position.x)
        velocity.x = dir * 110
    move_and_slide()

func _perform_attack() -> void:
    if current_state != State.CHASE:
        return
    
    change_state(State.SPECIAL)
    
    var attack_choice = randi() % 3
    match attack_choice:
        0: _shadow_teleport()
        1: _spawn_acid_pools()
        2: _chart_manipulation()

    await get_tree().create_timer(2.8).timeout
    change_state(State.CHASE)

func _shadow_teleport() -> void:
    if not can_teleport or not player:
        return
    
    can_teleport = false
    print("RUGPULL: Shadow Teleport!")
    
    # Teleport behind player with acid splash
    global_position = player.global_position + Vector2(randf_range(-180, 180), -60)
    
    # Spawn acid pool at teleport location
    _spawn_acid_pool(global_position)
    
    get_tree().get_first_node_in_group("camera").apply_acid_shake(35)
    AudioManager.play_boss_roar()
    
    await get_tree().create_timer(teleport_cooldown).timeout
    can_teleport = true

func _spawn_acid_pools() -> void:
    if not can_spawn_acid:
        return
    
    can_spawn_acid = false
    print("RUGPULL: Acid Pools!")
    
    for i in range(3):
        var pos = global_position + Vector2(randf_range(-200, 200), 0)
        _spawn_acid_pool(pos)
    
    await get_tree().create_timer(acid_pool_cooldown).timeout
    can_spawn_acid = true

func _chart_manipulation() -> void:
    print("RUGPULL: Chart Manipulation!")
    get_tree().get_first_node_in_group("camera").apply_shake(40)
    
    # Temporarily make some platforms slippery or disappear
    # (You can connect this to platform scripts later)

func _spawn_acid_pool(pos: Vector2) -> void:
    var pool = preload("res://scenes/projectiles/AcidPool.tscn").instantiate()
    pool.global_position = pos
    get_parent().add_child(pool)
Level3_AcidicVoid (Node2D)
├── WorldEnvironment
├── ParallaxBackground
│   ├── Parallax2D_Far (Speed 0.15)
│   ├── Parallax2D_Mid (Speed 0.45)
│   └── Parallax2D_Fore (Speed 0.8)
├── TileMapLayer_Ground
├── TileMapLayer_Platforms      ← Many dissolving platforms
├── TileMapLayer_Walls
├── TileMapLayer_Details        ← Apply acid lighting shader
├── Player
│   └── Camera2D (GameCamera)
├── Enemies (Node)
├── Bosses (Node)               ← RUGPULL the Overlord
├── Collectibles (Node)
├── Checkpoints (Node)
├── AcidAreas (Node)            ← Acid pools & rain zones
└── UI (CanvasLayer)
extends StaticBody2D

@export var dissolve_time: float = 2.5
@export var warning_time: float = 1.0
@export var respawn_time: float = 6.0

@onready var sprite: Sprite2D = $Sprite2D
@onready var collision: CollisionShape2D = $CollisionShape2D

var is_dissolving: bool = false

func _ready() -> void:
    body_entered.connect(_on_body_entered)

func _on_body_entered(body: Node2D) -> void:
    if body is Player and not is_dissolving:
        start_dissolving()

func start_dissolving() -> void:
    is_dissolving = true
    
    # Warning flash
    var warning_tween = create_tween()
    warning_tween.tween_property(sprite, "modulate", Color(1, 0.3, 0.3), warning_time / 2)
    warning_tween.tween_property(sprite, "modulate", Color(1, 1, 1), warning_time / 2)
    
    await get_tree().create_timer(warning_time).timeout
    
    # Dissolve animation
    var dissolve_tween = create_tween()
    dissolve_tween.tween_property(sprite, "modulate:a", 0.0, dissolve_time)
    dissolve_tween.tween_property(sprite, "scale", Vector2(0.1, 0.1), dissolve_time)
    
    collision.disabled = true
    
    await get_tree().create_timer(dissolve_time).timeout
    
    # Respawn after some time
    await get_tree().create_timer(respawn_time).timeout
    respawn()

func respawn() -> void:
    sprite.modulate.a = 1.0
    sprite.scale = Vector2.ONE
    collision.disabled = false
    is_dissolving = false
extends Node2D

@onready var fade_rect: ColorRect = $ColorRect
@onready var victory_label: Label = $VictoryLabel
@onready var stats_label: Label = $StatsLabel
@onready var particles: GPUParticles2D = $VictoryParticles
@onready var return_button: Button = $ReturnButton

func _ready() -> void:
    fade_rect.color.a = 1.0
    victory_label.text = "The Church of the Pump stands eternal.\nYou have reclaimed the flame."
    victory_label.modulate.a = 0.0
    stats_label.modulate.a = 0.0
    return_button.visible = false
    particles.emitting = false
    
    # Load stats
    stats_label.text = "Solana Orbs: %d\nFaith Restored: %d" % [SaveSystem.solana_orbs, SaveSystem.faith]
    
    start_cinematic()

func start_cinematic() -> void:
    var fade_tween = create_tween()
    fade_tween.tween_property(fade_rect, "color:a", 0.0, 3.0)
    
    await get_tree().create_timer(2.5).timeout
    particles.emitting = true
    
    var text_tween = create_tween()
    text_tween.tween_property(victory_label, "modulate:a", 1.0, 2.5)
    
    await get_tree().create_timer(2.0).timeout
    var stats_tween = create_tween()
    stats_tween.tween_property(stats_label, "modulate:a", 1.0, 2.0)
    
    await get_tree().create_timer(4.0).timeout
    return_button.visible = true
    return_button.pressed.connect(_return_to_menu)

func _return_to_menu() -> void:
    var fade_tween = create_tween()
    fade_tween.tween_property(fade_rect, "color:a", 1.0, 2.0)
    await fade_tween.finished
    get_tree().change_scene_to_file("res://scenes/ui/MainMenu.tscn")
extends Control

@onready var continue_button: Button = $ContinueButton
@onready var slot_buttons: Array[Button] = [$Slot1Button, $Slot2Button, $Slot3Button]

func _ready() -> void:
    _refresh_slot_buttons()
    continue_button.visible = SaveSystem.has_save(SaveSystem.current_slot)
    continue_button.pressed.connect(_on_continue_pressed)

func _refresh_slot_buttons() -> void:
    for i in range(3):
        var btn = slot_buttons[i]
        var slot = i + 1
        if SaveSystem.has_save(slot):
            btn.text = "Slot %d (Saved)" % slot
        else:
            btn.text = "Slot %d (New Game)" % slot
        btn.pressed.connect(_on_slot_pressed.bind(slot))

func _on_slot_pressed(slot: int) -> void:
    if SaveSystem.has_save(slot):
        SaveSystem.load_game(slot)
    else:
        SaveSystem.new_game(slot)
    get_tree().change_scene_to_file("res://scenes/levels/Level%d.tscn" % SaveSystem.current_level)

func _on_continue_pressed() -> void:
    SaveSystem.load_game(SaveSystem.current_slot)
    get_tree().change_scene_to_file("res://scenes/levels/Level%d.tscn" % SaveSystem.current_level)
godot --headless --export-release "itch.io Web" ./build/index.html
extends Node
class_name ObjectPool

@export var scene: PackedScene
@export var initial_size: int = 10

var pool: Array[Node] = []

func _ready() -> void:
    for i in range(initial_size):
        var instance = scene.instantiate()
        instance.visible = false
        instance.set_process(false)
        add_child(instance)
        pool.append(instance)

func get_instance() -> Node:
    for instance in pool:
        if not instance.visible:
            instance.visible = true
            instance.set_process(true)
            return instance
    
    # If pool is empty, create a new one
    var new_instance = scene.instantiate()
    add_child(new_instance)
    pool.append(new_instance)
    return new_instance

func return_instance(instance: Node) -> void:
    instance.visible = false
    instance.set_process(false)
    # Optional: reset position/scale here if needed
@onready var platform_pool: ObjectPool = $PlatformPool

func spawn_dissolving_platform(pos: Vector2) -> void:
    var platform = platform_pool.get_instance()
    platform.global_position = pos
    platform.visible = true
platform_pool.return_instance(self)

HTML to test locally
godot --headless --export-release "itch.io Web" ./build/index.html
cd build
python -m http.server 8000
godot --headless --export-release "itch.io Web" ./build/index.html && cd build && python -m http.server 8000
godot --headless --export-release "itch.io Web" ./build/index.html
extends Node
class_name ObjectPool

@export var scene: PackedScene
@export var initial_size: int = 12
@export var auto_expand: bool = true

var _pool: Array[Node] = []
var _active: Array[Node] = []

func _ready() -> void:
    if scene:
        prewarm(initial_size)

func prewarm(amount: int) -> void:
    for i in range(amount):
        var instance = scene.instantiate()
        _deactivate(instance)
        add_child(instance)
        _pool.append(instance)

func get_instance() -> Node:
    if _pool.size() > 0:
        var instance = _pool.pop_back()
        _activate(instance)
        _active.append(instance)
        return instance
    
    if auto_expand and scene:
        var new_instance = scene.instantiate()
        _activate(new_instance)
        add_child(new_instance)
        _active.append(new_instance)
        return new_instance
    
    push_warning("ObjectPool is empty and auto_expand is disabled")
    return null

func return_instance(instance: Node) -> void:
    if instance in _active:
        _active.erase(instance)
        _deactivate(instance)
        _pool.append(instance)

func _activate(instance: Node) -> void:
    instance.visible = true
    instance.set_process(true)
    if instance.has_method("reset"):
        instance.reset()

func _deactivate(instance: Node) -> void:
    instance.visible = false
    instance.set_process(false)
extends StaticBody2D

@export var dissolve_time: float = 2.5
@export var warning_time: float = 1.0
@export var respawn_delay: float = 5.0

@onready var sprite: Sprite2D = $Sprite2D
@onready var collision: CollisionShape2D = $CollisionShape2D

var is_dissolving := false
var original_scale: Vector2
var original_modulate: Color

func _ready() -> void:
    original_scale = sprite.scale
    original_modulate = sprite.modulate
    body_entered.connect(_on_body_entered)

func reset() -> void:
    is_dissolving = false
    sprite.scale = original_scale
    sprite.modulate = original_modulate
    collision.disabled = false
    visible = true

func _on_body_entered(body: Node2D) -> void:
    if body is Player and not is_dissolving:
        start_dissolving()

func start_dissolving() -> void:
    is_dissolving = true
    
    # Warning flash
    var tween = create_tween()
    tween.tween_property(sprite, "modulate", Color(1.2, 0.4, 0.4), warning_time * 0.5)
    tween.tween_property(sprite, "modulate", original_modulate, warning_time * 0.5)
    
    await get_tree().create_timer(warning_time).timeout
    
    if not is_dissolving:
        return
    
    # Dissolve animation
    var dissolve_tween = create_tween()
    dissolve_tween.tween_property(sprite, "modulate:a", 0.0, dissolve_time)
    dissolve_tween.parallel().tween_property(sprite, "scale", Vector2(0.2, 0.2), dissolve_time)
    
    collision.disabled = true
    
    await get_tree().create_timer(dissolve_time + 0.2).timeout
    
    if is_instance_valid(self):
        get_parent().get_node("PlatformPool").return_instance(self)
#!/bin/bash
echo "Building optimized WASM version for itch.io..."
godot --headless --export-release "itch.io Web - WASM" ./build/index.html
echo "Build complete! Folder: ./build"
cd build
python -m http.server 8000
godot --headless --export-release "itch.io Web - Optimized" ./build/index.html
[rendering]
limits/time/max_frame_time = 0.1
limits/time/max_physics_steps = 8
extends Node
class_name ObjectPool

@export var scene: PackedScene
@export var initial_size: int = 15
@export var max_pool_size: int = 50

var _pool: Array[Node] = []
var _active: Array[Node] = []

func _ready() -> void:
    if scene:
        prewarm(initial_size)

func prewarm(amount: int) -> void:
    for i in range(amount):
        _create_new_instance()

func get_instance() -> Node:
    if _pool.size() > 0:
        var instance = _pool.pop_back()
        _activate(instance)
        _active.append(instance)
        return instance
    
    if _active.size() < max_pool_size and scene:
        var new_instance = _create_new_instance()
        _activate(new_instance)
        _active.append(new_instance)
        return new_instance
    
    push_warning("ObjectPool limit reached")
    return null

func return_instance(instance: Node) -> void:
    if not is_instance_valid(instance):
        return
    
    if instance in _active:
        _active.erase(instance)
    
    if _pool.size() < max_pool_size:
        _deactivate(instance)
        _pool.append(instance)
    else:
        instance.queue_free()  # Prevent memory leak

func _create_new_instance() -> Node:
    var instance = scene.instantiate()
    _deactivate(instance)
    add_child(instance)
    return instance

func _activate(instance: Node) -> void:
    instance.visible = true
    instance.set_process(true)
    if instance.has_method("reset"):
        instance.reset()

func _deactivate(instance: Node) -> void:
    instance.visible = false
    instance.set_process(false)

func clear_pool() -> void:
    for instance in _pool:
        if is_instance_valid(instance):
            instance.queue_free()
    _pool.clear()
    
    for instance in _active:
        if is_instance_valid(instance):
            instance.queue_free()
    _active.clear()
--wasm-simd
godot --headless --export-release "itch.io Web - Optimized" ./build/index.html -- --wasm-simd --initial-memory=33554432
--wasm-simd \
--initial-memory=33554432 \
--max-memory=268435456 \
--compress
sudo apt update
sudo apt install openjdk-17-jdk android-sdk
docker run -d \
  --name godot-android-builder \
  -v /path/to/godot:/godot \
  -v /path/to/android-sdk:/sdk \
  godot-android
Wojaks_Nightmares/
├── project.godot
├── export_presets.cfg
├── .gitignore
├── README.md
│
├── assets/
│   ├── sprites/
│   │   ├── player/
│   │   ├── enemies/
│   │   ├── bosses/
│   │   └── ui/
│   ├── textures/
│   │   ├── posters/
│   │   └── backgrounds/
│   ├── audio/
│   │   ├── music/
│   │   └── sfx/
│   └── particles/
│
├── scenes/
│   ├── player/
│   │   └── Player.tscn
│   ├── enemies/
│   │   ├── MadChadler.tscn
│   │   ├── MadChadler_Gunner.tscn
│   │   ├── MadChadler_Heavy.tscn
│   │   └── MadChadler_Elite.tscn
│   ├── bosses/
│   │   ├── REKT.tscn
│   │   ├── FalseMoonAttack.tscn
│   │   ├── RUGPULL_Overlord.tscn
│   │   └── LegionOfLiquidation.tscn
│   ├── levels/
│   │   ├── Level1.tscn
│   │   ├── Level2_GothicCathedral.tscn
│   │   ├── Level3_AcidicVoid.tscn
│   │   └── Level4_FinalCathedral.tscn
│   ├── ui/
│   │   ├── MainMenu.tscn
│   │   ├── OptionsMenu.tscn
│   │   ├── SpellUI.tscn
│   │   ├── FaithMeter.tscn
│   │   ├── BossHealthBar.tscn
│   │   ├── GameOver.tscn
│   │   └── VictoryScreen.tscn
│   ├── cinematics/
│   │   └── EndingCinematic.tscn
│   └── checkpoints/
│       └── Checkpoint.tscn
│
├── scripts/
│   ├── player/
│   │   └── Player.gd
│   ├── enemies/
│   ├── bosses/
│   ├── platforms/
│   │   └── DissolvingPlatform.gd
│   ├── ui/
│   ├── systems/
│   │   ├── SaveSystem.gd          # Autoload
│   │   ├── AudioManager.gd        # Autoload
│   │   ├── DifficultyManager.gd   # Autoload
│   │   └── ObjectPool.gd          # Autoload
│   └── cinematics/
│       └── EndingCinematic.gd
│
├── shaders/
│   ├── acid_particle.gdshader
│   ├── chromatic_aberration.gdshader
│   ├── acid_screen_effect.gdshader
│   └── gothic_lighting.gdshader
│
├── autoloads/                       # (optional - for clarity)
│
├── addons/
│
└── build/                           # Export output (gitignored)



    
    
    
    


    
