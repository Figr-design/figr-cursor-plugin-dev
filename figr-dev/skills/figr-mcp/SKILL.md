---
name: figr-mcp
description: Use Figr MCP to load a Figr canvas or design system, explore UI source, and implement it in the user's repo. Use when the user pastes a Figr URL, asks to implement from Figr, or mentions Figr MCP.
---

# Figr MCP

Connect to Figr over MCP, read the design surface, implement in **this** codebase.

Full reference: [mcp-guide.md](mcp-guide.md)

## Quick path

1. `set_project` with the Figr URL (`?boardNode=` if they named a node).
2. `tree` / `ls /` — explore the live project tree.
3. If `selectedBoardNode` is set, start from its `folderPrefix` / `filePath`.
4. Prefer `get_design_context` on screens you will build (code + design chat).
5. `grep` scoped to the folder you care about; use `/queries` for rationale; use `/design-system` if present.
6. Port into the user's stack — match tokens/layout/behavior; wire real data; do not dump a disconnected copy.

## Rules

- MCP is read-only.
- Prefer VFS paths from `ls` / `tree` (e.g. `/login-prototype`), never `/deepagent/...` storage prefixes.
- `grep` patterns are JS RegExp (e.g. `Login|Button`).
