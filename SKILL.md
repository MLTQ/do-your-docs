---
name: do-your-docs
description: Aggressive code modularization with companion documentation. Use when working on any codebase where maintainability matters. Triggers on code creation, modification, refactoring, or when asked about code organization. Every code file gets a companion .md file documenting intent, rationale, and contracts. Review docs before touching code; update docs after every change.
---

# Do Your Docs

Maintain aggressive modularity and companion documentation for every code file.

## Core Principles

1. **One responsibility per file** - Split aggressively. When in doubt, split.
2. **Every code file has a companion `.md`** - Same name, same directory (e.g., `parser.rs` → `parser.md`)
3. **Docs before code** - Read the `.md` before modifying any file
4. **Docs after code** - Update the `.md` after any modification

## Companion Doc Structure

Keep docs **concise** (~50-100 lines max). Use this template:

```markdown
# {filename}

## Purpose
Why this file exists. Original intent. Current role. 2-3 sentences max.

## Components

### `function_name` / `StructName` / `ClassName`
- **Does**: One-line description
- **Interacts with**: `other_fn` in `other_file.rs`, `SomeStruct` in `types.rs`
- **Rationale**: Why it exists or why it's shaped this way (only if non-obvious)

(Repeat for each public component. Skip trivial helpers.)

## Contracts

Who depends on this and how:

| Dependent | Expects | Breaking changes |
|-----------|---------|------------------|
| `main.rs` | `init()` returns `Result<Config>` | Return type, error variants |
| `api.rs`  | `Handler` implements `Send + Sync` | Removing trait impl |

## Notes
Optional. Gotchas, tech debt, future plans. Delete section if empty.
```

## Workflow

### Before modifying code:
1. `view {filename}.md` - Read companion doc
2. Understand purpose, contracts, and interactions
3. Identify what docs will need updating

### After modifying code:
1. Update **Purpose** if intent changed
2. Update **Components** for any added/removed/changed functions
3. Update **Contracts** if external expectations changed
4. If file was split → create new `.md` files for new files

### When creating new files:
1. Create the code file
2. Immediately create companion `.md`
3. Fill in Purpose and Components
4. Add to Contracts table of files that will depend on it

## Splitting Heuristics

Split a file when:
- It has multiple distinct responsibilities (primary signal)
- You find yourself scrolling to understand it
- Two features could evolve independently
- Testing requires mocking unrelated parts
- The file is large enough that you'd hesitate to read it top-to-bottom

Resist splitting when:
- The components are genuinely coupled (splitting would just scatter one concept)
- A class is large but cohesive (700 lines of one well-defined thing > 7 files of fragments)

When splitting:
- Each new file gets its own `.md`
- Update the original `.md` to note what was extracted
- Update Contracts tables in affected files

## Example

For a file `src/network/client.rs`:

```markdown
# client.rs

## Purpose
HTTP client wrapper with retry logic and auth handling. Exists to centralize all external API calls behind a consistent interface.

## Components

### `Client`
- **Does**: Holds connection pool and auth state
- **Interacts with**: `Config` from `../config.rs`, `AuthToken` from `auth.rs`

### `Client::get`, `Client::post`
- **Does**: HTTP methods with automatic retry and auth refresh
- **Interacts with**: `RetryPolicy` from `retry.rs`
- **Rationale**: Retry logic here (not in callers) to ensure consistency

### `ClientError`
- **Does**: Unified error type for all client failures
- **Interacts with**: Converted from `reqwest::Error`, `AuthError`

## Contracts

| Dependent | Expects | Breaking changes |
|-----------|---------|------------------|
| `api/posts.rs` | `Client::get` returns `Result<Response, ClientError>` | Return type |
| `api/users.rs` | `Client` is `Clone` | Removing Clone derive |
| `main.rs` | `Client::new(config)` constructs client | Constructor signature |

## Notes
- Connection pool size tuned for our rate limits (see config)
- TODO: Add circuit breaker for cascade failure prevention
```

## Maintenance

When joining a codebase with this skill:
1. Run `find . -name "*.rs" -o -name "*.py" -o -name "*.ts"` (etc.) to list code files
2. Check which lack companion `.md` files
3. Create missing docs by reading the code and inferring intent

Prioritize documenting files you're about to modify.
