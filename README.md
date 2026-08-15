# battle-royal

<img width="1293" height="759" alt="Screenshot 2026-08-15 102554" src="https://github.com/user-attachments/assets/8459bbe0-62ee-4639-8624-e33116866f28" />

# 2D Battle Royale

A 2D battle royale-style game built with **Python and Pygame**. Fight against AI-controlled bots, collect different weapons, survive multiple levels, and improve your chances as the difficulty increases.

## 🎮 Features

* **2D top-down combat**
* AI-controlled enemy bots
* Multiple weapons:

  * Pistol
  * Assault Rifle
  * Sniper
* Different weapon damage and fire rates
* Ammo and reload system
* Headshot chance
* Health bars and multiple lives
* Weapon pickups around the map
* Increasing difficulty across levels
* Bot accuracy and movement improve as levels increase
* Damage numbers that appear when characters are hit
* Fullscreen mode
* Adjustable volume and mouse sensitivity
* Multiple resolution options, including:
 
  * 720p
  * 1080p
  * 1440p
  * 4K

## 🕹️ Controls

| Key / Input           | Action                          |
| --------------------- | ------------------------------- |
| **W A S D**           | Move                            |
| **Left Mouse Button** | Shoot                           |
| **R**                 | Reload                          |
| **1 - 3**             | Switch weapons                  |
| **ESC**               | Open/close settings             |
| **Arrow Keys**        | Navigate settings               |
| **Left/Right Arrow**  | Change settings                 |
| **F**                 | Toggle fullscreen               |
| **Enter**             | Toggle fullscreen when selected |

## 🔫 Weapons

The game currently includes three weapons with different statistics:

### Pistol

* Magazine: 30 rounds
* Body damage: 23
* Headshot damage: 41
* Reload time: 3.5 seconds

### Assault Rifle

* Magazine: 32 rounds
* Body damage: 30
* Headshot damage: 55
* Reload time: 4.5 seconds

### Sniper

* Magazine: 1 round
* Body damage: 95
* Headshot damage: 150
* Reload time: 5 seconds

## 🤖 Bot AI

Bots automatically search for targets and move toward them. They can target either the player or other bots.

As the level increases, bots become more challenging through:

* Better accuracy
* Faster movement
* Increased shooting frequency

This creates progressively harder waves as the player advances.

## 📈 Level System

The game starts at **Level 1**.

After all bots in a wave are defeated, the next level begins. Higher levels increase the number of bots and weapon pickups while also making the AI more difficult.

The player can also receive better weapons as they progress:

* **Level 3:** Assault Rifle bonus
* **Level 6:** Sniper bonus

## ❤️ Health & Lives

Characters start with **100 health** and **3 lives**.

When a character's health reaches zero, they lose a life. If they still have lives remaining, they respawn at a random location. Once all lives are gone, the character is eliminated.

## ⚙️ Settings

The game includes a settings menu where the player can adjust:

* Master volume
* Sensitivity
* Fullscreen mode

The game also supports several predefined resolutions, from **720p up to 4K**.

## 🛠️ Requirements

You need:

* Python 3
* Pygame

Install Pygame with:

```bash
pip install pygame
```

## ▶️ How to Run

1. Install Python 3.
2. Install Pygame.
3. Save the game as a `.py` file.
4. Open a terminal in the folder containing the file.
5. Run:

```bash
python your_game_file.py
```

## 📦 Main Libraries

The project uses several built-in Python modules along with Pygame:

* `pygame` — graphics, input, display, and game loop
* `math` — movement, angles, and distance calculations
* `random` — bot behavior, spawning, weapon pickups, and randomness
* `sys` — program/system controls
* `time` — shooting cooldowns, reload timers, and damage-text animations

## 🚀 Project Structure

The game is organized around several main classes:

* `DamageText` — displays temporary damage and pickup messages
* `Bullet` — controls bullet movement and damage information
* `GunPickup` — handles weapons that appear on the map
* `Character` — controls the player and bots
* `GameEngine` — manages the main game, levels, AI, collisions, settings, and rendering

## 📝 Notes

This project is a standalone Pygame game and does not require an internet connection or external game engine.

The code is currently structured as a single Python file, making it easy to run and experiment with while developing new features.

## 🔮 Possible Future Improvements

Some features that could be added later include:

* Better maps and environments
* More weapons
* More advanced bot AI
* Sound effects and music
* Main menu and lobby system
* More game modes
* Multiplayer support
* Weapon animations
* More detailed graphics
* Loot rarity system
* Armor/shield system
* Leaderboards

---

**Built with Python + Pygame 🎮**
