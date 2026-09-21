---
name: go-project-store-skill
description: Store layer patterns for Go projects — Driver interface, Store wrapper with caching, migration system, and model conventions. Use when adding models, implementing drivers, or creating migrations.
activation: /go-project-store-skill
license: MIT
metadata:
  version: 1.1.0
  author: pix
  tags: [go, store, driver, cache, migration, database, crud, sqlite, mysql, postgres]
  created: 2026-09-21
  last_reviewed: 2026-09-21
  review_interval_days: 90
  dependencies:
    - go
    - sqlite
    - mysql
    - postgres
provenance:
  maintainer: pix
  version: 1.1.0
  created: 2026-09-21
  last_reviewed: 2026-09-21
  review_interval_days: 90
  source_references:
    - references/code-patterns.md
---

# /go-project-store-skill

## When to use

Activate this skill when:

- Adding a new model/table to the store layer (new struct, CRUD methods, migration)
- Implementing a new database driver (sqlite, mysql, postgres)
- Creating or modifying migration files (LATEST.sql or incremental)
- Defining or extending the `Driver` interface in `driver.go`
- Working with cache wrappers in `store.go`
- Debugging store-layer compilation errors after proto changes

Do NOT use for:
- Server-side HTTP/RPC handlers (use go-project-server-skill)
- Proto message definitions (use go-project-proto-skill)
- CLI setup or startup flow (use go-project-main-skill)

## Prerequisites

**Order matters**: Proto → Store → Server

```bash
# 1. Create proto directory and definitions FIRST (see go-project-proto-skill)
mkdir -p proto/api/v1
touch proto/buf.yaml
touch proto/buf.gen.yaml

# 2. Define services in proto/api/v1/*_service.proto
#    Store models map to proto messages

# 3. Generate code
cd proto && buf generate

# 4. Create store layer (store/driver.go defines interfaces)
#    Proto messages reference store models

# 5. Create server layer (uses both proto and store)
```

## Directory Structure

```
store/
├── store.go              # Store wrapper (STRUCTURAL)
├── driver.go             # Driver interface (STRUCTURAL)
├── cache.go              # Cache implementation (STRUCTURAL)
├── migrator.go           # Migration system (STRUCTURAL)
├── model.go              # Base types (STRUCTURAL)
├── {model}.go            # Model definitions + Store methods (BUSINESS)
├── db/
│   ├── db.go             # Driver factory (STRUCTURAL)
│   ├── sqlite/           # SQLite CRUD (BUSINESS)
│   ├── mysql/            # MySQL CRUD (BUSINESS)
│   └── postgres/         # PostgreSQL CRUD (BUSINESS)
├── migration/            # SQL migrations (BUSINESS)
│   ├── sqlite/
│   │   ├── LATEST.sql    # Full schema
│   │   └── 0.22/
│   │       └── 00__add_table.sql
│   ├── mysql/
│   └── postgres/
└── seed/                 # Demo data (BUSINESS)
```

## Workflows

### Workflow: Add a New Model

1. **Define proto message** in `proto/api/v1/{model}_service.proto`
2. **Generate Go code**: `cd proto && buf generate`
3. **Add model struct** to `store/{model}.go` (with `ID int32`, `CreatedTs int64`, `RowStatus`, and business fields)
4. **Add Update/Find/Delete structs** — Update uses `*string`/`*int64` pointer fields for optional updates; Find uses pointers + `Limit *int`
5. **Add to Driver interface** in `store/driver.go` — Create, Update, List, Delete methods
6. **Add Store methods** with cache operations in `store/{model}.go`
7. **Add cache field** to `Store` struct in `store/store.go`
8. **Implement driver methods** in `store/db/sqlite/{model}.go` (and mysql/postgres)
9. **Create migration** — add table to `LATEST.sql` AND create incremental `0.XX/00__add_{model}.sql`
10. **Verify**: `go build ./...` && `go vet ./...`

### Workflow: Create a Migration

1. **Determine version** — check current `LATEST.sql` version, increment
2. **Create directory** `store/migration/{driver}/{version}/`
3. **Create SQL file** `00__description.sql` with DDL statements
4. **Update LATEST.sql** — add the new table/column to the full schema
5. **Test on all drivers**: `go test ./store/db/sqlite/...` (and mysql/postgres if available)

### Workflow: Implement a New Driver

1. **Create directory** `store/db/{driver}/`
2. **Implement Driver interface** — all methods from `store/driver.go`
3. **Register in factory** — add case to `store/db/db.go`
4. **Handle driver-specific SQL** — use build tags or switch for dialect differences (e.g., `AUTOINCREMENT` vs `AUTO_INCREMENT`)
5. **Copy migration files** — create `store/migration/{driver}/` with driver-specific LATEST.sql

## Examples

### Example 1: Add Memo Model with Full CRUD

**Goal**: Add a `Memo` model with create, read, update, delete, and list operations.

Proto definition exists → `buf generate` → create `store/memo.go` with struct + Update/Find/Delete structs → add to `store/driver.go` interface → add Store methods with cache → add `memoCache` to `Store` struct → implement `store/db/sqlite/memo.go` → create migration → `go build ./...`

### Example 2: Create SQLite Migration for New Column

**Goal**: Add `description` column to `user` table.

Create `store/migration/sqlite/0.23/01__add_user_description.sql`:
```sql
ALTER TABLE user ADD COLUMN description TEXT NOT NULL DEFAULT '';
```
Update `LATEST.sql` to include `description` in CREATE TABLE → update `User` struct and `UpdateUser` struct → `go build ./...`

### Example 3: Fresh Install vs Upgrade

- **Fresh install**: `preMigrate()` → `IsInitialized()` false → apply LATEST.sql → `migrateProd()` applies all incremental files
- **Upgrade** (at v0.22): `preMigrate()` skip → `migrateProd()` applies only files > 0.22
- **Demo mode**: Apply LATEST.sql → `seed()` loads seed/*.sql

### Example 4: Cache Invalidation on Update

```go
func (s *Store) UpdateUser(ctx context.Context, update *UpdateUser) (*User, error) {
    user, err := s.driver.UpdateUser(ctx, update)
    if err != nil { return nil, err }
    s.userCache.Delete(ctx, string(user.ID))  // invalidate
    s.userCache.Set(ctx, string(user.ID), user) // re-populate
    return user, nil
}
```

Delete + set avoids a cache miss on the next read within the same request.

### Example 5: Implement Postgres Driver for Memo

Create `store/db/postgres/memo.go` — use `$1, $2` parameterized queries instead of `?` → handle `RETURNING` differences → create `store/migration/postgres/LATEST.sql` with Postgres-compatible DDL (e.g., `SERIAL PRIMARY KEY`, `EXTRACT(EPOCH FROM NOW())` for timestamps).

## Quick Reference

| File | Type | Purpose |
|------|------|---------|
| `store.go` | Structural | Cache wrapper, lifecycle |
| `driver.go` | Structural | Interface for CRUD operations |
| `cache.go` | Structural | In-memory cache |
| `migrator.go` | Structural | Migration system |
| `{model}.go` | Business | Model + Store methods |
| `db/db.go` | Structural | Driver factory |
| `migration/driver/LATEST.sql` | Business | Complete schema |

## Key Patterns

| Pattern | Description |
|---------|-------------|
| **Caching** | Store wraps Driver, adds cache ops |
| **Dynamic Updates** | Update structs use pointers |
| **Timestamps** | Unix (`int64`), not `time.Time` |
| **RETURNING** | Get generated IDs |
| **Migration Flow** | preMigrate → migrateProd → seed |
| **Atomic Migrations** | Single transaction |

## Errors

| Error | Cause | Fix |
|-------|-------|-----|
| `undefined: store.User` | Proto not generated | Run `buf generate` in proto/ |
| `cannot use nil as *string` | Missing pointer in Update struct | Use `&value` or define pointer field |
| `migration failed` | SQL syntax error in .sql file | Check driver-specific DDL syntax |
| `cache miss after set` | Wrong cache key format | Ensure key matches `string(model.ID)` |
| `embed: no such file` | embed.FS path wrong | Check path relative to file with `//go:embed` |
| `compile error: missing method` | Driver doesn't implement all interface methods | Add missing methods to driver |

## Gotchas

- Proto definitions MUST exist before Store layer code — store models map to proto messages and the build will fail if protos are missing.
- The `mode` profile field (`modeProd` vs `modeDemo`) controls whether `Migrate()` runs incremental SQL or seeds demo data. Confusing these causes data loss.
- `LATEST.sql` is the canonical full schema for fresh installs. Incremental migrations in `0.XX/` folders are applied on top. Never edit `LATEST.sql` after release — only add new incremental files.
- Update structs use `*string` / `*int64` pointer fields for optional updates. Passing a zero value is NOT the same as omitting it — use pointers to distinguish "set to empty" from "don't change".
- Cache keys are stringified IDs. If your model uses a composite key, you must build a cache key string that encodes all parts, or cache lookups will silently return wrong results.
- `embed.FS` paths are relative to the file containing the `//go:embed` directive. If you move `migrator.go`, the embedded paths break at compile time, not runtime.

## Keywords

store, driver, cache, migration, crud, database, sqlite, mysql, postgres, go, golang, model, interface, wrapper, embed, sql, schema, transaction, row_status, timestamp, protobuf, buf

## Related Skills

- go-project-main-skill — CLI and startup flow
- go-project-server-skill — Server implementation
- go-project-proto-skill — Protocol buffer definitions
- go-project-conventions-skill — Code conventions

## References

- Read `references/code-patterns.md` for detailed Driver interface, Store wrapper, Model definition, and Migration code examples
