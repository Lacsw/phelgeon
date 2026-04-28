# phelgeon

CLI roguelike dungeon crawler written in **Phel** (Lisp on PHP).

## Layout

- `src/phelgeon/main.phel` — entry point (`phelgeon\main` namespace), thin launcher
- `src/phelgeon/core.phel` — all game logic (`phelgeon\core` namespace), pure functions
- `phel-config.php` — Flat layout (`src/`)
- Run: `composer start` → `./vendor/bin/phel run src/phelgeon/main.phel`

## Code style (per design)

- State is a single immutable map flowing through pure functions
- No globals; every action returns a new state
- I/O lives only in `main.phel`; `core.phel` is pure

## Phel reference

We've vendored the upstream Phel project's `.claude/` directory for reference:

- `.claude/rules/phel.md` — Phel language conventions (kebab-case, `defn-`, docstring metadata, etc.)
- `.claude/skills/phel-patterns/SKILL.md` — idioms, threading macros, struct patterns
- `.claude/skills/phel-repl/SKILL.md` — how to evaluate Phel expressions for verification (note: replace `./bin/phel` with `./vendor/bin/phel` in our setup)

The other upstream skills/agents (compiler-guide, release, changelog-keeper, etc.) target Phel-compiler maintainers, not us. Ignore unless relevant.

The original upstream `CLAUDE.md` and `AGENTS.md` are archived at `.claude/UPSTREAM_PHEL_*.md` for reference but no longer auto-load.

## Hooks

The vendored `.claude/settings.json` keeps two PostToolUse/PreToolUse hooks (`format-php.sh`, `protect-files.sh`) — both are no-ops in this project (no `.php` source files; protected paths don't exist here).
