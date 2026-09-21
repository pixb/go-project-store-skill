# go-project-store-skill

Store layer patterns for Go projects: Driver interface, Store wrapper with
caching, migration system, and model definition conventions. Follow the
Driver→Store→Server layering order when building new features.

## Activation

Invoke with `/go-project-store-skill` or when:

- Adding a new model to the store layer (new table, new CRUD methods)
- Implementing a new database driver (sqlite, mysql, postgres)
- Creating or modifying migration files
- Defining store interfaces in `driver.go`
- Working with cache wrappers in `store.go`

## What this skill covers

- Driver interface pattern (CRUD signatures, lifecycle methods)
- Store wrapper with in-memory caching
- Model definition conventions (Update/Find/Delete structs, pointer fields)
- Migration system (LATEST.sql + incremental, embed directives)
- Driver factory pattern (`db/db.go`)
- Relationship between proto definitions and store models

## How to use this file

This is the cross-tool companion file (AAIF format). The full reference —
directory structure, code examples, migration flow, and key patterns — lives
in [SKILL.md](SKILL.md). Read SKILL.md and follow it; treat this file as the
pointer, not the instructions.
