# phelgeon

Turn-based, room-graph CLI dungeon crawler written in **Phel** (Lisp on PHP). See `README.md` for player-facing docs.

## File layout

```
src/phelgeon/
  main.phel      terminal lifecycle + prompt loop. ONLY file with I/O.
  core.phel      pure game engine — state, procgen, combat, step.
  content.phel   pure data — constants, enemies, loot, descriptions, lore, intro.
  render.phel    pure ANSI frame builders.
phel-config.php  flat layout, mainNamespace = phelgeon\main.
docs/superpowers/specs/  brainstorming spec for the redesign (gitignored).
docs/superpowers/plans/  implementation plan (gitignored).
```

`composer start` runs `phel run phelgeon\\main`. The `(main)` form at the bottom of `main.phel` is the entrypoint.

## Code style

- State is a single immutable map flowing through pure functions. No globals.
- Every action is `state → state`. Mutation only via `assoc` / `update` etc.
- `core.phel`, `content.phel`, `render.phel` are 100% pure. `main.phel` is the only file allowed to call `php/fwrite`, `php/fgets`, `php/shell_exec`, `php/register_shutdown_function`.
- Randomness is wrapped in a few `rand-*` helpers at the top of `core.phel`. Don't sprinkle `(rand-int …)` calls elsewhere — keeps the impure surface small.
- No tests in v1 (deliberate — pure-function design is easy to test later if we want).

## State shape (top-level keys)

```clojure
{:player    {:hp :max-hp :attack :defense :inventory [] :gold
             :kills :biggest-hit :potions-used}
 :floor     1
 :floor-name "..."
 :rooms     {<id> {:type :exits :description :enemies :loot :cleared :trap}}
 :current   <room-id>
 :visited   #{<room-ids>}
 :log       []                 ; last 3 strings
 :mode      :intro|:explore|:combat|:inventory|:gameover|:victory|:quit
 :combat    nil | {:enemies [...]}
 :from-dir  nil | direction back to previous room
 :pending-descend bool
 :pending-quit    bool}
```

## Phel idioms / gotchas — read this first

- **`rand-int n` is INCLUSIVE** (Phel uses `php/random_int(0, n)`). Use `(rand-int (dec n))` for 0..n-1 indexing. We learned this by hitting nil-arithmetic crashes.
- **`take-last n coll` throws "Cannot slice" when `n > (count coll)`.** Use `drop`/`take` arithmetic instead. The `log` helper in `core.phel` shows the pattern.
- **Don't use `phel-cli-gui` from `main.phel`.** Its lazy `get-gui` (triggered by `clear-screen`, `hide-cursor`, etc.) calls `stream_set_blocking(STDIN, false)`. With non-blocking stdin, `php/fgets` returns `false` immediately and the game auto-quits — including under `composer start`. We emit the ANSI escapes directly.
- **`phel build` evaluates top-level forms** in compiled namespaces. The `(main)` call at the bottom of `main.phel` will *run the game* during build. Output looks weird but exit code is fine; just be aware when adding new top-level expressions.
- **Clear the cache after Phel-source changes** if behaviour seems stale: `./vendor/bin/phel cache:clear`. Phel keeps a hash-keyed compiled cache that occasionally lags behind source edits.
- **`php/empty` doesn't work** — `empty` is a PHP language construct, not a function. Use `(zero? (count v))` or check `nil?`.
- **`(name kw)` and `#{...}`-as-predicate** both work; the codebase uses them.
- **Top-level `(def)` of vectors / maps with side-effecting expressions is fine** — they're evaluated once at namespace load.

## Build / verify

| What | Command |
|---|---|
| Compile-check all namespaces | `./vendor/bin/phel build` |
| Force fresh build | `./vendor/bin/phel cache:clear && ./vendor/bin/phel build` |
| Smoke-run with piped input | `printf 'q' \| ./vendor/bin/phel run phelgeon\\main` |
| Inspect runtime errors | `cat /tmp/phel-error.log` (default error log) |
| Strip ANSI for diff-able output | `LC_ALL=C sed 's/\x1b[^a-zA-Z]*[a-zA-Z]//g'` |

In raw input mode each game key is a single byte — pipe `q` (no newline) to quit immediately. EOF on stdin is treated as "quit cleanly" by `main.phel`.

## What to update where

- **New gameplay tunable** (HP, damage, weights) → `content.phel` only.
- **New room or enemy type** → `content.phel` (data) + `core.phel` (build/handle) + maybe `render.phel` (display).
- **New action** (key + effect) → `parse-input` mapping in `core.phel` + a handler + `combat-prompt` / `explore-prompt` in `render.phel`.
- **New visual element** → `render.phel` only (it's pure).
- **Terminal behaviour** (escape sequences, lifecycle) → `main.phel` only.

## Phel reference

The vendored upstream Phel `.claude/` directory has reference material we keep:

- `.claude/rules/phel.md` — Phel language conventions (kebab-case, `defn-`, docstring metadata, etc.).
- `.claude/skills/phel-patterns/SKILL.md` — idioms, threading macros, struct patterns.
- `.claude/skills/phel-repl/SKILL.md` — REPL evaluation. Replace `./bin/phel` with `./vendor/bin/phel` in our setup.

Other upstream skills/agents (compiler-guide, release, changelog-keeper, etc.) target Phel-compiler maintainers, not us. Ignore unless relevant. The original upstream `CLAUDE.md` and `AGENTS.md` are archived at `.claude/UPSTREAM_PHEL_*.md`.

## Hooks

The vendored `.claude/settings.json` keeps two PostToolUse/PreToolUse hooks (`format-php.sh`, `protect-files.sh`) — both are no-ops in this project (no `.php` source files; protected paths don't exist here).
