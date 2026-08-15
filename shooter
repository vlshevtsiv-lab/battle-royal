import pygame
import math
import random
import sys
import time

# -------------------------------------------------------------------
# ENGINE CONFIGURATION & CONSTANTS
# -------------------------------------------------------------------
pygame.init()
pygame.font.init()

RESOLUTIONS = {
    "720p": (1280, 720),
    "1080p": (1920, 1080),
    "1440p": (2560, 1440),
    "4K": (3840, 2160)
}
CURRENT_RES_KEY = "720p"
SCREEN_WIDTH, SCREEN_HEIGHT = RESOLUTIONS[CURRENT_RES_KEY]

screen = pygame.display.set_mode((SCREEN_WIDTH, SCREEN_HEIGHT))
pygame.display.set_caption("2D Battle Royale - Improved Bot AI")
clock = pygame.time.Clock()

TARGET_FPS = 240
FONT_MAIN = pygame.font.SysFont("Arial", 18, bold=True)
FONT_LARGE = pygame.font.SysFont("Arial", 48, bold=True)

COLOR_BG = (34, 139, 34)
COLOR_PLAYER = (0, 122, 255)
COLOR_BOT = (220, 20, 60)
COLOR_TEXT = (255, 255, 255)
COLOR_HP_BAR = (50, 205, 50)
COLOR_DAMAGE = (255, 215, 0)

# -------------------------------------------------------------------
# WEAPON STAT DATA STRUCTS
# -------------------------------------------------------------------
WEAPON_STATS = {
    "Pistol": {
        "fire_rate": 7.0,
        "mag_size": 30,
        "reload_time": 3.5,
        "body_damage": 23,
        "head_damage": 41,
        "color": (200, 200, 200)
    },
    "Assault Rifle": {
        "fire_rate": 5.0,
        "mag_size": 32,
        "reload_time": 4.5,
        "body_damage": 30,
        "head_damage": 55,
        "color": (139, 69, 19)
    },
    "Sniper": {
        "fire_rate": 1.0,
        "mag_size": 1,
        "reload_time": 5.0,
        "body_damage": 95,
        "head_damage": 150,
        "color": (50, 50, 50)
    }
}

# -------------------------------------------------------------------
# GAME CLASSES
# -------------------------------------------------------------------

class DamageText:
    def __init__(self, text, x, y):
        self.text = str(text)
        self.x = x
        self.y = y
        self.lifetime = 1.0
        self.spawn_time = time.time()

    def is_alive(self):
        return time.time() - self.spawn_time < self.lifetime

    def draw(self, surface):
        alpha = int(255 * (1 - (time.time() - self.spawn_time) / self.lifetime))
        text_surf = FONT_MAIN.render(self.text, True, COLOR_DAMAGE)
        text_surf.set_alpha(max(0, alpha))
        surface.blit(text_surf, (self.x, self.y - int((time.time() - self.spawn_time) * 30)))


class Bullet:
    def __init__(self, x, y, angle, damage, is_headshot, owner):
        self.x = x
        self.y = y
        self.speed = 1200
        self.vx = math.cos(angle) * self.speed
        self.vy = math.sin(angle) * self.speed
        self.damage = damage
        self.is_headshot = is_headshot
        self.owner = owner
        self.alive = True

    def update(self, dt):
        self.x += self.vx * dt
        self.y += self.vy * dt
        if self.x < 0 or self.x > SCREEN_WIDTH or self.y < 0 or self.y > SCREEN_HEIGHT:
            self.alive = False

    def draw(self, surface):
        pygame.draw.circle(surface, (255, 255, 0), (int(self.x), int(self.y)), 4)


class GunPickup:
    def __init__(self, x, y, name):
        self.x = x
        self.y = y
        self.name = name
        self.radius = 15

    def draw(self, surface):
        color = WEAPON_STATS[self.name]["color"]
        pygame.draw.circle(surface, color, (self.x, self.y), self.radius)
        label = FONT_MAIN.render(self.name, True, (255, 255, 255))
        surface.blit(label, (self.x - 20, self.y - 25))


class Character:
    def __init__(self, x, y, name="Bot", is_player=False):
        self.x = x
        self.y = y
        self.name = name
        self.is_player = is_player
        self.radius = 20
        self.head_radius = 8
        self.speed = 200

        self.health = 100
        self.max_health = 100
        self.lives = 3
        self.is_dead = False

        self.inventory = ["Pistol", None, None, None, None]
        self.current_slot = 0
        self.current_mag = WEAPON_STATS["Pistol"]["mag_size"]
        self.last_shot_time = 0
        self.reloading = False
        self.reload_start_time = 0

    def active_weapon_name(self):
        return self.inventory[self.current_slot]

    def get_weapon_data(self):
        w_name = self.active_weapon_name()
        return WEAPON_STATS.get(w_name, None)

    def reload(self):
        w_data = self.get_weapon_data()
        if w_data and not self.reloading and self.current_mag < w_data["mag_size"]:
            self.reloading = True
            self.reload_start_time = time.time()

    def update_reload(self):
        if self.reloading:
            w_data = self.get_weapon_data()
            if w_data and (time.time() - self.reload_start_time >= w_data["reload_time"]):
                self.current_mag = w_data["mag_size"]
                self.reloading = False

    def shoot(self, target_x, target_y, bullets_list, accuracy_spread=0.0):
        """
        accuracy_spread: Radians of random offset added to shot angle (0.0 = perfect aim).
        """
        if self.is_dead or self.reloading:
            return

        w_data = self.get_weapon_data()
        if not w_data:
            return

        now = time.time()
        fire_delay = 1.0 / w_data["fire_rate"]
        
        if now - self.last_shot_time >= fire_delay:
            if self.current_mag > 0:
                self.current_mag -= 1
                self.last_shot_time = now

                # Base angle toward target
                dx = target_x - self.x
                dy = target_y - self.y
                base_angle = math.atan2(dy, dx)

                # Add inaccuracy spread offset for bots
                spread_offset = random.uniform(-accuracy_spread, accuracy_spread)
                final_angle = base_angle + spread_offset

                is_headshot = random.random() < 0.15
                damage = w_data["head_damage"] if is_headshot else w_data["body_damage"]

                bullets_list.append(Bullet(self.x, self.y, final_angle, damage, is_headshot, self))

                if self.current_mag == 0:
                    self.reload()

    def take_damage(self, amount, damage_texts):
        if self.is_dead:
            return
        self.health -= amount
        damage_texts.append(DamageText(amount, self.x, self.y - 30))

        if self.health <= 0:
            self.lives -= 1
            if self.lives > 0:
                self.health = 100
                self.x = random.randint(100, SCREEN_WIDTH - 100)
                self.y = random.randint(100, SCREEN_HEIGHT - 100)
            else:
                self.is_dead = True

    def draw(self, surface):
        if self.is_dead:
            return

        color = COLOR_PLAYER if self.is_player else COLOR_BOT
        pygame.draw.circle(surface, color, (int(self.x), int(self.y)), self.radius)
        pygame.draw.circle(surface, (255, 220, 177), (int(self.x), int(self.y)), self.head_radius)

        # Health Bar
        bar_w = 40
        bar_h = 6
        hp_percent = max(0, self.health / self.max_health)
        pygame.draw.rect(surface, (50, 50, 50), (self.x - bar_w/2, self.y - 35, bar_w, bar_h))
        pygame.draw.rect(surface, COLOR_HP_BAR, (self.x - bar_w/2, self.y - 35, bar_w * hp_percent, bar_h))

        name_surf = FONT_MAIN.render(f"{self.name} ({self.lives} L)", True, COLOR_TEXT)
        surface.blit(name_surf, (self.x - name_surf.get_width()/2, self.y - 55))


# -------------------------------------------------------------------
# GAME ENGINE
# -------------------------------------------------------------------
class GameEngine:
    def __init__(self):
        self.state = "PLAY"
        self.player_name = "Player1"
        self.level = 1
        self.wave_cooldown = 0.0
        self.master_volume = 0.5
        self.sensitivity = 1.0
        self.fullscreen = False
        self.settings_items = ["Volume", "Sensitivity", "Fullscreen"]
        self.settings_selected = 0

        self.player = Character(SCREEN_WIDTH // 2, SCREEN_HEIGHT // 2, name="You", is_player=True)
        self.bots = []
        self.bullets = []
        self.pickups = []
        self.damage_texts = []

        self.start_level()

        pygame.mouse.set_visible(False)
        pygame.event.set_grab(True)

    def start_level(self):
        bot_count = min(10, 4 + self.level)
        self.bots = [
            Character(
                random.randint(100, SCREEN_WIDTH - 100),
                random.randint(100, SCREEN_HEIGHT - 100),
                name=f"Bot {i+1}"
            ) for i in range(bot_count)
        ]

        pickup_count = min(6, 2 + self.level // 2)
        self.pickups = [
            GunPickup(
                random.randint(80, SCREEN_WIDTH - 80),
                random.randint(80, SCREEN_HEIGHT - 80),
                random.choice(list(WEAPON_STATS.keys()))
            ) for _ in range(pickup_count)
        ]

        self.bullets.clear()
        self.damage_texts.append(DamageText(f"Level {self.level}", SCREEN_WIDTH / 2, 80))
        self.player.max_health = 100 + min(50, (self.level - 1) * 5)
        self.player.health = self.player.max_health
        self.player.reloading = False
        self.player.current_mag = self.player.get_weapon_data()["mag_size"]

        if self.level >= 6:
            self.grant_weapon_bonus("Sniper")
        elif self.level >= 3:
            self.grant_weapon_bonus("Assault Rifle")

        self.player.x = SCREEN_WIDTH // 2
        self.player.y = SCREEN_HEIGHT // 2
        self.wave_cooldown = 2.0

    def toggle_fullscreen(self):
        self.fullscreen = not self.fullscreen
        global screen
        flags = pygame.FULLSCREEN if self.fullscreen else 0
        screen = pygame.display.set_mode((SCREEN_WIDTH, SCREEN_HEIGHT), flags)

    def adjust_setting(self, direction):
        selection = self.settings_items[self.settings_selected]
        if selection == "Volume":
            self.master_volume = min(1.0, max(0.0, self.master_volume + direction * 0.05))
        elif selection == "Sensitivity":
            self.sensitivity = min(2.0, max(0.5, self.sensitivity + direction * 0.1))
        elif selection == "Fullscreen":
            if direction != 0:
                self.toggle_fullscreen()

    def grant_weapon_bonus(self, weapon_name):
        if weapon_name in self.player.inventory:
            return
        free_slot = next((idx for idx, item in enumerate(self.player.inventory) if item is None), None)
        if free_slot is not None:
            self.player.inventory[free_slot] = weapon_name
        else:
            self.player.inventory[1] = weapon_name

    def bot_accuracy_spread(self):
        return max(0.02, 0.45 - (self.level - 1) * 0.06)

    def bot_shoot_rate(self):
        return min(0.08, 0.01 + (self.level - 1) * 0.007)

    def bot_speed_modifier(self):
        return min(1.2, 0.28 + (self.level - 1) * 0.09)

    def get_settings_item_rects(self):
        base_x = SCREEN_WIDTH / 2 - 180
        base_y = SCREEN_HEIGHT / 2 - 40
        return [pygame.Rect(base_x, base_y + idx * 32, 360, 28) for idx in range(len(self.settings_items))]

    def update_settings_hover(self, mouse_pos):
        for idx, rect in enumerate(self.get_settings_item_rects()):
            if rect.collidepoint(mouse_pos):
                self.settings_selected = idx
                break

    def apply_settings_click(self, mouse_pos):
        for idx, rect in enumerate(self.get_settings_item_rects()):
            if rect.collidepoint(mouse_pos):
                self.settings_selected = idx
                selection = self.settings_items[idx]
                if selection == "Fullscreen":
                    self.toggle_fullscreen()
                elif selection == "Volume":
                    if mouse_pos[0] < SCREEN_WIDTH / 2:
                        self.adjust_setting(-1)
                    else:
                        self.adjust_setting(1)
                elif selection == "Sensitivity":
                    if mouse_pos[0] < SCREEN_WIDTH / 2:
                        self.adjust_setting(-1)
                    else:
                        self.adjust_setting(1)
                break

    def handle_events(self):
        for event in pygame.event.get():
            if event.type == pygame.QUIT:
                pygame.quit()
                sys.exit()
            if event.type == pygame.KEYDOWN:
                if event.key == pygame.K_ESCAPE:
                    self.state = "PLAY" if self.state == "SETTINGS" else "SETTINGS"
                    pygame.mouse.set_visible(self.state == "SETTINGS")
                    pygame.event.set_grab(self.state != "SETTINGS")
                elif self.state == "PLAY":
                    if event.key == pygame.K_1:
                        self.player.current_slot = 0
                    elif event.key == pygame.K_2:
                        self.player.current_slot = 1
                    elif event.key == pygame.K_3:
                        self.player.current_slot = 2
                    elif event.key == pygame.K_r:
                        self.player.reload()
                elif self.state == "SETTINGS":
                    if event.key == pygame.K_UP:
                        self.settings_selected = (self.settings_selected - 1) % len(self.settings_items)
                    elif event.key == pygame.K_DOWN:
                        self.settings_selected = (self.settings_selected + 1) % len(self.settings_items)
                    elif event.key == pygame.K_LEFT:
                        self.adjust_setting(-1)
                    elif event.key == pygame.K_RIGHT:
                        self.adjust_setting(1)
                    elif event.key == pygame.K_RETURN:
                        if self.settings_items[self.settings_selected] == "Fullscreen":
                            self.toggle_fullscreen()
                    elif event.key == pygame.K_f:
                        self.toggle_fullscreen()
            if self.state == "SETTINGS" and event.type == pygame.MOUSEMOTION:
                self.update_settings_hover(event.pos)
            if self.state == "SETTINGS" and event.type == pygame.MOUSEBUTTONDOWN and event.button == 1:
                self.apply_settings_click(event.pos)
            if self.state == "PLAY" and event.type == pygame.MOUSEBUTTONDOWN and event.button == 1:
                mx, my = pygame.mouse.get_pos()
                self.player.shoot(mx, my, self.bullets, accuracy_spread=0.05)

    def update(self, dt):
        if self.state != "PLAY":
            return

        keys = pygame.key.get_pressed()
        dx = dy = 0.0
        move_speed = self.player.speed * self.sensitivity
        if keys[pygame.K_w]:
            dy -= move_speed * dt
        if keys[pygame.K_s]:
            dy += move_speed * dt
        if keys[pygame.K_a]:
            dx -= move_speed * dt
        if keys[pygame.K_d]:
            dx += move_speed * dt

        self.player.x = max(20, min(SCREEN_WIDTH - 20, self.player.x + dx))
        self.player.y = max(20, min(SCREEN_HEIGHT - 20, self.player.y + dy))

        self.player.update_reload()
        for bot in self.bots:
            if not bot.is_dead:
                self.update_bot(bot, dt)
                bot.update_reload()

        for bullet in self.bullets:
            bullet.update(dt)
        self.bullets = [b for b in self.bullets if b.alive]

        self.handle_collisions()
        self.damage_texts = [d for d in self.damage_texts if d.is_alive()]

        if self.wave_cooldown > 0:
            self.wave_cooldown = max(0.0, self.wave_cooldown - dt)

        if self.wave_cooldown == 0 and all(bot.is_dead for bot in self.bots):
            self.level += 1
            self.start_level()

    def update_bot(self, bot, dt):
        target = self.choose_bot_target(bot)
        if target is None:
            return

        angle = math.atan2(target.y - bot.y, target.x - bot.x)
        bot.x += math.cos(angle) * bot.speed * self.bot_speed_modifier() * dt
        bot.y += math.sin(angle) * bot.speed * self.bot_speed_modifier() * dt

        if random.random() < self.bot_shoot_rate():
            distance = math.hypot(target.x - bot.x, target.y - bot.y)
            if distance < 800:
                bot.shoot(target.x, target.y, self.bullets, accuracy_spread=self.bot_accuracy_spread())

    def choose_bot_target(self, bot):
        candidates = []
        if not self.player.is_dead:
            candidates.append(self.player)
        for other in self.bots:
            if other is not bot and not other.is_dead:
                candidates.append(other)

        if not candidates:
            return None

        if len(candidates) == 1:
            return candidates[0]

        if random.random() < 0.75:
            return min(candidates, key=lambda c: math.hypot(c.x - bot.x, c.y - bot.y))

        return random.choice(candidates)

    def handle_collisions(self):
        for bullet in self.bullets:
            if not bullet.alive:
                continue
            if bullet.owner is not self.player and not self.player.is_dead:
                if math.hypot(bullet.x - self.player.x, bullet.y - self.player.y) < self.player.radius + 5:
                    self.player.take_damage(bullet.damage, self.damage_texts)
                    bullet.alive = False
            for bot in self.bots:
                if bullet.owner is not bot and not bot.is_dead:
                    if math.hypot(bullet.x - bot.x, bullet.y - bot.y) < bot.radius + 5:
                        bot.take_damage(bullet.damage, self.damage_texts)
                        bullet.alive = False
        for pickup in list(self.pickups):
            if math.hypot(pickup.x - self.player.x, pickup.y - self.player.y) < pickup.radius + self.player.radius:
                free_slot = next((idx for idx, item in enumerate(self.player.inventory) if item is None), 0)
                self.player.inventory[free_slot] = pickup.name
                self.player.current_slot = free_slot
                self.pickups.remove(pickup)
                self.damage_texts.append(DamageText(f"Picked up {pickup.name}", self.player.x, self.player.y - 40))

    def draw(self):
        if self.state == "SETTINGS":
            self.draw_settings()
            return

        screen.fill(COLOR_BG)
        for pickup in self.pickups:
            pickup.draw(screen)
        for bot in self.bots:
            bot.draw(screen)
        for bullet in self.bullets:
            bullet.draw(screen)
        self.player.draw(screen)
        for dmg in self.damage_texts:
            dmg.draw(screen)
        self.draw_hud()
        pygame.display.flip()

    def draw_settings(self):
        screen.fill((20, 20, 20))
        title = FONT_LARGE.render("Settings", True, COLOR_TEXT)
        screen.blit(title, ((SCREEN_WIDTH - title.get_width()) / 2, SCREEN_HEIGHT / 2 - 140))

        option_values = [
            f"{int(self.master_volume * 100)}%",
            f"{self.sensitivity:.1f}x",
            "On" if self.fullscreen else "Off"
        ]

        for idx, option in enumerate(self.settings_items):
            is_selected = idx == self.settings_selected
            prefix = "> " if is_selected else "  "
            label_text = f"{prefix}{option}: {option_values[idx]}"
            label_color = (255, 255, 128) if is_selected else COLOR_TEXT
            option_label = FONT_MAIN.render(label_text, True, label_color)
            screen.blit(option_label, (SCREEN_WIDTH / 2 - 180, SCREEN_HEIGHT / 2 - 40 + idx * 32))

        prompt = FONT_MAIN.render("Up/Down = choose, Left/Right = change", True, COLOR_TEXT)
        toggle_hint = FONT_MAIN.render("Enter/F = toggle fullscreen", True, COLOR_TEXT)
        screen.blit(prompt, ((SCREEN_WIDTH - prompt.get_width()) / 2, SCREEN_HEIGHT / 2 + 80))
        screen.blit(toggle_hint, ((SCREEN_WIDTH - toggle_hint.get_width()) / 2, SCREEN_HEIGHT / 2 + 110))
        pygame.display.flip()

    def draw_hud(self):
        labels = [
            FONT_MAIN.render(f"Health: {int(self.player.health)}", True, COLOR_TEXT),
            FONT_MAIN.render(f"Weapon: {self.player.active_weapon_name()}", True, COLOR_TEXT),
            FONT_MAIN.render(f"Ammo: {self.player.current_mag}", True, COLOR_TEXT),
            FONT_MAIN.render(f"Level: {self.level}", True, COLOR_TEXT),
            FONT_MAIN.render("WASD move, Mouse1 shoot, R reload, 1-3 switch", True, COLOR_TEXT)
        ]
        for idx, label in enumerate(labels):
            screen.blit(label, (10, 10 + idx * 22))

    def run(self):
        while True:
            dt = clock.tick(TARGET_FPS) / 1000.0
            self.handle_events()
            self.update(dt)
            self.draw()


if __name__ == "__main__":
    GameEngine().run()

# -------------------------------------------------------------------
# GAME CLASSES
# -------------------------------------------------------------------

class DamageText:
    def __init__(self, text, x, y):
        self.text = str(text)
        self.x = x
        self.y = y
        self.lifetime = 1.0
        self.spawn_time = time.time()

    def is_alive(self):
        return time.time() - self.spawn_time < self.lifetime

    def draw(self, surface):
        alpha = int(255 * (1 - (time.time() - self.spawn_time) / self.lifetime))
        text_surf = FONT_MAIN.render(self.text, True, COLOR_DAMAGE)
        text_surf.set_alpha(max(0, alpha))
        surface.blit(text_surf, (self.x, self.y - int((time.time() - self.spawn_time) * 30)))


class Bullet:
    def __init__(self, x, y, angle, damage, is_headshot, owner):
        self.x = x
        self.y = y
        self.speed = 1200
        self.vx = math.cos(angle) * self.speed
        self.vy = math.sin(angle) * self.speed
        self.damage = damage
        self.is_headshot = is_headshot
        self.owner = owner
        self.alive = True

    def update(self, dt):
        self.x += self.vx * dt
        self.y += self.vy * dt
        if self.x < 0 or self.x > SCREEN_WIDTH or self.y < 0 or self.y > SCREEN_HEIGHT:
            self.alive = False

    def draw(self, surface):
        pygame.draw.circle(surface, (255, 255, 0), (int(self.x), int(self.y)), 4)


class GunPickup:
    def __init__(self, x, y, name):
        self.x = x
        self.y = y
        self.name = name
        self.radius = 15

    def draw(self, surface):
        color = WEAPON_STATS[self.name]["color"]
        pygame.draw.circle(surface, color, (self.x, self.y), self.radius)
        label = FONT_MAIN.render(self.name, True, (255, 255, 255))
        surface.blit(label, (self.x - 20, self.y - 25))


class Character:
    def __init__(self, x, y, name="Bot", is_player=False):
        self.x = x
        self.y = y
        self.name = name
        self.is_player = is_player
        self.radius = 20
        self.head_radius = 8
        self.speed = 200

        self.health = 100
        self.max_health = 100
        self.lives = 3
        self.is_dead = False

        self.inventory = ["Pistol", None, None, None, None]
        self.current_slot = 0
        self.current_mag = WEAPON_STATS["Pistol"]["mag_size"]
        self.last_shot_time = 0
        self.reloading = False
        self.reload_start_time = 0

    def active_weapon_name(self):
        return self.inventory[self.current_slot]

    def get_weapon_data(self):
        w_name = self.active_weapon_name()
        return WEAPON_STATS.get(w_name, None)

    def reload(self):
        w_data = self.get_weapon_data()
        if w_data and not self.reloading and self.current_mag < w_data["mag_size"]:
            self.reloading = True
            self.reload_start_time = time.time()

    def update_reload(self):
        if self.reloading:
            w_data = self.get_weapon_data()
            if w_data and (time.time() - self.reload_start_time >= w_data["reload_time"]):
                self.current_mag = w_data["mag_size"]
                self.reloading = False

    def shoot(self, target_x, target_y, bullets_list, accuracy_spread=0.0):
        """
        accuracy_spread: Radians of random offset added to shot angle (0.0 = perfect aim).
        """
        if self.is_dead or self.reloading:
            return

        w_data = self.get_weapon_data()
        if not w_data:
            return

        now = time.time()
        fire_delay = 1.0 / w_data["fire_rate"]
        
        if now - self.last_shot_time >= fire_delay:
            if self.current_mag > 0:
                self.current_mag -= 1
                self.last_shot_time = now

                # Base angle toward target
                dx = target_x - self.x
                dy = target_y - self.y
                base_angle = math.atan2(dy, dx)

                # Add inaccuracy spread offset for bots
                spread_offset = random.uniform(-accuracy_spread, accuracy_spread)
                final_angle = base_angle + spread_offset

                is_headshot = random.random() < 0.15
                damage = w_data["head_damage"] if is_headshot else w_data["body_damage"]

                bullets_list.append(Bullet(self.x, self.y, final_angle, damage, is_headshot, self))

                if self.current_mag == 0:
                    self.reload()

    def take_damage(self, amount, damage_texts):
        if self.is_dead:
            return
        self.health -= amount
        damage_texts.append(DamageText(amount, self.x, self.y - 30))

        if self.health <= 0:
            self.lives -= 1
            if self.lives > 0:
                self.health = 100
                self.x = random.randint(100, SCREEN_WIDTH - 100)
                self.y = random.randint(100, SCREEN_HEIGHT - 100)
            else:
                self.is_dead = True

    def draw(self, surface):
        if self.is_dead:
            return

        color = COLOR_PLAYER if self.is_player else COLOR_BOT
        pygame.draw.circle(surface, color, (int(self.x), int(self.y)), self.radius)
        pygame.draw.circle(surface, (255, 220, 177), (int(self.x), int(self.y)), self.head_radius)

        # Health Bar
        bar_w = 40
        bar_h = 6
        hp_percent = max(0, self.health / self.max_health)
        pygame.draw.rect(surface, (50, 50, 50), (self.x - bar_w/2, self.y - 35, bar_w, bar_h))
        pygame.draw.rect(surface, COLOR_HP_BAR, (self.x - bar_w/2, self.y - 35, bar_w * hp_percent, bar_h))

        name_surf = FONT_MAIN.render(f"{self.name} ({self.lives} L)", True, COLOR_TEXT)
        surface.blit(name_surf, (self.x - name_surf.get_width()/2, self.y - 55))


# -------------------------------------------------------------------
# GAME ENGINE
# -------------------------------------------------------------------
class GameEngine:
    def __init__(self):
        self.state = "LOBBY"
        self.coins = 0
        self.player_name = "Player1"

        self.player = None
        self.bots = []
        self.bullets = []
        self.pickups = []
        self.damage_texts = []
        self.victory_