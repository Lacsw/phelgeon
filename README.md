# phelgeon

A procedural, turn-based dungeon crawler for the terminal — written in [Phel](https://phel-lang.org/) (a Lisp on PHP).

```
[Floor 1]  The Goblin Warren
━━━━━━━━━━━━━━━━━━━━━━━━━━━━

A damp cave tunnel stretches before you. Bones crunch
underfoot. A goblin sharpens its blade in the corner.

GOBLIN  HP: ████░░░░  8/15
YOU     HP: ████████  80/100

Exits: north, east

> [a]ttack  [f]lee  [u]se potion

― You strike the goblin for 7 damage
― The goblin hits you for 4 damage
```

## What's in it

- Three procedurally generated floors of 6–8 rooms each.
- Random encounters: goblins, skeletons, orcs, dark mages, and a final boss on floor 3.
- Loot: health potions, iron swords (+attack), leather armor (+defense), gold.
- Turn-based combat with crits, flee, and item use.
- ANSI-coloured frame, re-rendered each turn.

## Install

```bash
composer install
```

Requires PHP 8.3+.

## Run

```bash
./vendor/bin/phel run phelgeon\\main
```

Or via composer:

```bash
composer start
```

## Controls

| Mode | Keys |
|---|---|
| Explore | `n` `s` `e` `w` to move, `i` inventory, `q` quit |
| Combat | `a` attack, `f` flee, `u` use potion |
| Inventory | `1`–`N` to use, `b` back |

## Code layout

```
src/phelgeon/
  main.phel      terminal setup + prompt loop (only file with I/O)
  core.phel      pure game engine — state transitions
  content.phel   constants, enemy/loot/description pools
  render.phel    ANSI frame builders
```

State is a single immutable map; every action is a pure `state → state` function.
