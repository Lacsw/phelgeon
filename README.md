# phelgeon

A procedural, turn-based dungeon crawler for the terminal — written in [Phel](https://phel-lang.org/).

Three floors. Procedurally generated rooms. Multi-enemy combat with abilities. Hidden traps. Named bosses with backstory. A single immutable game state flowing through pure functions.

```
┌──────────────────────────────────────────────────────────────────────┐
│ [Floor 1]  The Goblin Warren                                         │
│ ━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━ │
│                                                                      │
│ A damp cave tunnel stretches before you. Bones crunch underfoot.     │
│ Two foes bar your way:                                               │
│   ― Cave rat (twitching whiskers and yellow teeth)                   │
│   ― Orc (scarred, snarling, all muscle)                              │
│                                                                      │
│ ──────────────────────────────────────────────────────────────────── │
│                                                                      │
│ CAVE RAT    HP: ████████  9/9                                        │
│ ORC         HP: ████████  36/36                                      │
│ YOU         HP: ███████░  87/100                                     │
│                                                                      │
│ ──────────────────────────────────────────────────────────────────── │
│                                                                      │
│ > [a]ttack  [f]lee  [u]se potion  [b]omb                             │
│                                                                      │
│ ― Acid sprays from a hidden vent — 13 damage.                        │
│ ― The Orc strikes with fury for 22 damage!                           │
│ ― You found a bomb.                                                  │
└──────────────────────────────────────────────────────────────────────┘
```

## What's in it

- **Three procedurally generated floors** of 6–8 rooms each, connected as a small graph (some branching, some loops).
- **Six enemy types** with abilities — Goblin, Cave rat (vanilla), Skeleton (`tough`, may halve damage taken), Orc (`fury`, may double damage), Dark mage (`caster`, fixed-damage spell), Wraith (`phase`, ignores defense).
- **Multi-enemy combat** — combat rooms can have 1 or 2 foes; you fight the front one until it falls, then the next.
- **Three named bosses** with backstory — Skarn the Bone-Crowned, Mother Rust the Iron Wyrm, the Cinderking. One is randomly chosen for floor 3 each run.
- **Hidden traps** — 18% of combat / treasure / exit rooms are trapped. Spike traps, dart vents, acid sprays, loose stones.
- **Five item types** — health potion, iron sword (+5 attack), leather armor (+3 defense), gold, bomb (instant-kill in combat), scroll of healing (full restore).
- **Critical hits**, flee chance, dynamic difficulty per floor (×1.0 → ×1.4 → ×1.8 stat scaling).
- **Confirmation before quit** so a stray `q` doesn't end your run.
- **ANSI-coloured frame in a Unicode box**, re-rendered in place each turn (alternate-screen buffer; doesn't pollute your shell scrollback).

## Install

```bash
composer install
```

Requires PHP 8.3+.

## Run

```bash
composer start
```

Or directly:

```bash
./vendor/bin/phel run phelgeon\\main
```

Single-keystroke input — no enter needed for any command.

## Controls

| Mode      | Keys                                                                      |
|-----------|---------------------------------------------------------------------------|
| Intro     | any key to begin                                                          |
| Explore   | `n` `s` `e` `w` move · `i` inventory · `q` quit (asks first)              |
| Descend   | `y` confirm · `n` stay (when standing on the stairs)                      |
| Combat    | `a` attack · `f` flee (50%) · `u` potion · `b` bomb                       |
| Inventory | `1` use potion · `2` use heal scroll · `b` back                           |
| End screen| any key to exit                                                           |

The current room's exits show `(back)` on whichever direction you came from.

## Code layout

```
src/phelgeon/
  main.phel      terminal lifecycle + prompt loop (the ONLY file with I/O)
  core.phel      pure game engine — state transitions, combat, procgen
  content.phel   constants, enemy / loot / description pools, lore
  render.phel    pure ANSI frame builders
docs/superpowers/    spec + plan + brainstorming notes (gitignored)
```

State is a single immutable map. Every action is a pure `state → state` function. Randomness is isolated to a few `rand-*` wrappers in `core.phel`. The only place that does I/O — terminal setup, line input, ANSI emit — is `main.phel`.

## Design notes

- **Alternate screen buffer**: the game uses `\e[?1049h` so its frames don't end up in your shell scrollback. On exit it returns you to the shell exactly as you left it.
- **Raw input mode**: `stty -icanon -echo` while playing, restored via `register_shutdown_function` so Ctrl-C or normal exit always leave the terminal usable.
- **No `phel-cli-gui`** for input/output — its lazy init flips stdin to non-blocking, which clashes with turn-based reads. `main.phel` emits the four ANSI escapes it needs (clear, home, hide-cursor, show-cursor) directly.
- **All Phel idioms are vanilla** — `rand-int` (note: Phel's is *inclusive* on both ends), `for :reduce`, `case`, threading macros, structs as plain maps.

## License

MIT.
