# go-project-store-skill

Store layer patterns for Go projects: Driver interface, Store wrapper with caching, migration system, and model definition conventions.

## Installation

### For opencode

Copy or symlink the skill directory into your opencode skills folder:

```bash
# Option A: Symlink (recommended — stays updated)
ln -s /path/to/go-project-store-skill ~/.config/opencode/skills/go-project-store-skill

# Option B: Copy
cp -r /path/to/go-project-store-skill ~/.config/opencode/skills/
```

### For Claude Code

```bash
# Copy to Claude Code skills directory
cp -r /path/to/go-project-store-skill ~/.claude/skills/
```

### For Cursor / Windsurf / Trae

Copy the `SKILL.md` and `AGENTS.md` files to your project's `.cursor/skills/` or equivalent directory.

## Usage

Invoke the skill when working on store-layer code:

```
/go-project-store-skill
```

The skill activates automatically when you:
- Add a new model/table to the store layer
- Implement a new database driver
- Create or modify migration files
- Define or extend the Driver interface

## What This Skill Covers

| Topic | Description |
|-------|-------------|
| Driver interface | CRUD method signatures and lifecycle methods |
| Store wrapper | In-memory caching layer over the driver |
| Model structs | Update/Find/Delete patterns with pointer fields |
| Migrations | LATEST.sql for fresh installs, incremental for upgrades |
| Driver factory | Multi-driver support (sqlite, mysql, postgres) |
| Cache invalidation | Keeping cache consistent on writes |

## Prerequisites

- Go project with `store/` directory structure
- Proto definitions (use go-project-proto-skill first)
- SQLite, MySQL, or PostgreSQL driver

## Related Skills

- `go-project-main-skill` — CLI and startup flow
- `go-project-server-skill` — Server implementation
- `go-project-proto-skill` — Protocol buffer definitions
- `go-project-conventions-skill` — Code conventions

## License

MIT
