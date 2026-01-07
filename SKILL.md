---
name: lsp-code-analysis
description: Semantic codebase intelligence via LSP. Essential for building an accurate mental model of unfamiliar codebases, safely navigating complex dependencies, and performing precise refactoring. Provides compiler-level accuracy for code understanding that surpasses simple text-based search.
license: LICENSE
---

# LSP Code Analysis

## Overview

Use `lsp-cli` for semantic code navigation and analysis via Language Server Protocol (LSP).

LSP provides **compiler-accurate** code understanding that surpasses text-based search:

- **Semantic navigation**: Jump to definitions, find references, locate implementations - understands code structure, not just text patterns.
- **Language-aware**: Distinguishes between variables, functions, classes, types - eliminates false positives from text search.
- **Cross-file intelligence**: Trace dependencies, refactor safely across entire codebase - knows what imports what.
- **Type-aware**: Get precise type information, signatures, documentation - without reading implementation code.

**When to use**: Exploring unfamiliar code, refactoring, debugging, understanding dependencies. **Prefer over grep/text search** when you need to understand how code works, not just where text appears.

## Prerequisites

```bash
uv tool install lsp-cli
```

## Locating Symbols

Most commands use a unified **Locate String** syntax via the `-L` or `--locate` option.

**Syntax**: `<file_path>[:<scope>][@<find>]`

- **Scope**: Narrows search area. Supports line (`:42`), range (`:10,20`), or symbol path (`:User.login`).
- **Find**: Text pattern within scope. Whitespace-insensitive; matches even if code formatting differs. Use `@text` or `@snippet<|>with_marker` for exact cursor positioning.
- **Marker**: `<|>` indicates the exact position for symbol resolution (e.g., `user.<|>name`).

**Examples**:

- `main.py:42` - Exactly line 42 in `main.py`.
- `app.py:User.login` - The `login` method of class `User` in `app.py`.
- `utils.py@process_data<|>` - Position at the symbol `process_data` in `utils.py`.
- `api.py:10,50@config` - The string `config` within lines 10-50 of `api.py`.

Use `lsp locate <string>` to verify a location before running other commands.

## Commands

All commands support `-h` or `--help`. Command aliases: `def` (definition), `ref` (reference), `sym` (symbol).

### Outline: File Structure

**MUST** use before reading files to get structural overview. **SHOULD** prefer `outline` over full `read` for non-essential code.

```bash
# Main symbols (classes, functions, methods)
lsp outline <file_path>

# All symbols (includes variables, parameters)
lsp outline <file_path> --all
```

### Definition: Navigate to Source

Jump to symbol definitions. **RECOMMENDED** for verifying function signatures without reading full implementation.

```bash
# By locate string (alias: lsp def)
lsp definition -L "models.py:User.get_id"

# Declaration instead of definition
lsp definition -L "models.py:25" --decl

# Type definition
lsp definition -L "models.py:30" --type
```

### Reference: Find All Usages

**REQUIRED** before refactoring or deleting code. Use `--impl` for finding implementations in abstract codebases.

```bash
# Find references (alias: lsp ref)
lsp reference -L "main.py:MyClass.run@logger"

# Find implementations
lsp reference -L "api.py@IDataProvider" --impl

# More context lines
lsp reference -L "app.py:10@TestClass" --context-lines 5
```

### Hover: Get Documentation

**SHOULD** prefer over `read` for understanding API contracts. Returns docstrings and type signatures.

```bash
# By line
lsp hover -L "main.py:42"

# By text search
lsp hover -L "models.py@process_data<|>"
```

### Search: Global Symbol Search

**RECOMMENDED** when symbol location is unknown. Use `--kind` to filter results.

```bash
# Search symbols (defaults to current directory)
lsp search "MyClassName"

# Specific workspace
lsp search "UserModel" --workspace /path/to/project

# Filter by kind
lsp search "init" --kind function --kind method

# Limit results
lsp search "Config" --max-items 10
```

### Rename: Safe Refactoring

Rename a symbol workspace-wide. Defaults to preview mode.

```bash
# Preview changes
lsp rename new_name -L "models.py:OldName"

# Execute changes (requires --execute)
lsp rename new_name -L "models.py:OldName" --execute

# Apply using a specific rename ID from a previous preview
lsp rename new_name -L "models.py:OldName" --execute --id rename_abc123
```

### Symbol: Local Symbol Info

Get precise coordinate info for a symbol. **MAY** be used to anchor subsequent `hover` or `definition` calls.

```bash
# By line (alias: lsp sym)
lsp symbol -L "main.py:15"

# By text search
lsp symbol -L "utils.py@UserClass<|>"
```

### Server: Manage Background Servers

Background manager starts automatically. Manual control is **OPTIONAL**.

```bash
# List running servers
lsp server list

# Start server
lsp server start <path>

# Stop server
lsp server stop <path>
```

## Best Practices

### General Workflows

#### Understanding Unfamiliar Code

**RECOMMENDED** sequence for exploring new codebases:

1. **Start with outline** - Get file structure without reading implementation.
2. **Inspect signatures** - Use `hover` to understand API contracts.
3. **Navigate dependencies** - Follow `definition` chains.
4. **Map usage** - Find where code is called with `reference`.

#### Refactoring Preparation

**REQUIRED** steps before modifying code:

1. **Find all references** - Identify impact scope.
2. **Check implementations** - For interfaces/abstract classes using `--impl`.
3. **Verify type definitions** - Understand type propagation with `--type`.
4. **Preview Rename** - Use `lsp rename` to see workspace-wide impact.

#### Debugging Unknown Behavior

1. **Search for the symbol** - Locate where it's defined with `lsp search`.
2. **Find the definition** - Verify implementation.
3. **Trace all callers** - See where it's invoked using `lsp reference`.

### Performance Tips

- **Use `outline` aggressively** - Avoid reading entire files when possible.
- **Leverage symbol paths** - More precise than line numbers for nested structures.
- **Use `--max-items`** - Limit results in large codebases.
- **Prefer `hover` over `definition`** - For understanding without navigating.
- **Verify with `locate`** - If a command fails, use `lsp locate` to debug the target.

### Common Patterns

#### Finding Interface Implementations

```bash
# Step 1: Locate interface definition
lsp search "IUserService" --kind interface

# Step 2: Find all implementations
lsp reference -L "src/interfaces.py:IUserService" --impl
```

#### Tracing Data Flow

```bash
# Step 1: Find where data is created
lsp search "UserDTO" --kind class

# Step 2: Find where it's used
lsp reference -L "models.py:UserDTO"

# Step 3: Check transformations
lsp hover -L "transform.py:map_to_dto"
```

#### Understanding Type Hierarchies

```bash
# Step 1: Get class outline
lsp outline models.py

# Step 2: Find subclasses (references to base)
lsp reference -L "models.py:BaseModel"

# Step 3: Check type definitions
lsp definition -L "models.py:BaseModel" --type
```

### Domain-Specific Guides

For specialized scenarios, see:

- **Frontend**: [bp_frontend.md](references/bp_frontend.md)
- **Backend**: [bp_backend.md](references/bp_backend.md)
