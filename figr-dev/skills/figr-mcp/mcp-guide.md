# Figr MCP — agent guide

Figr is an AI design canvas. Designers build UI in conversation; the **Figr MCP** exposes that project as a **read-only virtual filesystem** so coding agents can implement the real product.

**Endpoint:** whatever Figr MCP HTTP URL the client has configured (production default: `https://mcp.figr.design/mcp`).

This guide is the source of truth for Claude Code, Cursor, VS Code, Codex, and other MCP clients.

---

## What you get

| Capability | Tool(s) | Notes |
|---|---|---|
| Bind a project | `set_project` | Pass a Figr project URL. Call this first. |
| Bind a design system | `set_design_system` | Org design-system URL when not in a project. |
| Explore structure | `ls`, `tree` | Start at `/`. Full live project tree. |
| Read files | `read` | Source, queries, `board.json`, `project.json`. |
| Search | `grep` | JS RegExp per line (e.g. `Button\|export`). Prefer scoping `path`. |
| File metadata | `stat` | Size, type, Figr-specific fields. |
| Design + chat context | `get_design_context` | Code **plus** the conversation that shaped it. Prefer over bare `read` when implementing. |
| Connectivity | `ping` | Smoke check. |

MCP is **read-only**. You never write back to Figr over these tools. Implementation happens in the user’s local repo.

---

## Project modes

After `set_project`, the server reports the mode.

### Deep Agent (primary)

Full app-style trees (React/TS screens, components, CSS). Typical VFS:

```text
/
├── <prototype-folder>/   # e.g. /login-prototype/
│   └── src/
│       ├── screens/
│       ├── components/
│       ├── App.tsx
│       └── …
├── queries/              # AI turns that produced the work
├── design-system/        # when a DS is attached
├── project.json
└── canvas/board.json     # spatial board layout
```

### Artifact projects

Older / alternate shape:

```text
/
├── artifacts/          # ComponentName::vN.ext
├── queries/
├── project.json
└── flow.md
```

---

## URLs

| Form | Example |
|---|---|
| Production | `https://app.figr.design/{orgId}/{projectId}` |
| Local / short | `http://localhost:5173/projects/{projectId}` |
| Node focus | `...?boardNode={nodeKey}` |
| Artifact focus | `...?selectedNodes=Name::v1` |
| Design system | `https://app.figr.design/{orgId}/design-system/{designSystemId}` |

`projectId` is a UUID (second path segment on org-scoped URLs).

---

## Recommended workflow

1. **`set_project`** with the user’s Figr URL (include `?boardNode=` when they care about one node).
2. **`tree` or `ls /`** — see prototypes + queries.
3. If `selectedBoardNode` is present, start there (`folderPrefix` / `filePath`).
4. **`get_design_context`** on the main screen/component paths (not only `read`).
5. **`grep`** scoped to that folder for symbols, routes, tokens.
6. **`ls /queries`** when you need the product/design rationale.
7. Implement in the **user’s codebase**, matching structure, tokens, and behavior from Figr — do not paste a disconnected sandbox.

Optional: `queryId` on tools = conversation-turn file snapshot.

---

## What the code is

Deep Agent folders are **real product UI source**, usually:

- **React + TypeScript** (screens, components, routes)
- **CSS / Tailwind** (often `index.css`, `tailwind.config.js`)
- App shell: `App.tsx`, `routes.tsx`, `src/screens/*`, `src/components/*`

Treat it as the design’s implementation reference:

- Layout, spacing, typography, color tokens
- Component API and composition
- Interaction affordances (buttons, forms, navigation)
- Naming the designer already chose

It is **not** necessarily production-complete (may lack real APIs, auth, tests). Your job is to port intent into the user’s stack cleanly.

Artifact projects expose versioned design artifacts (JSX/HTML/etc.) under `/artifacts` instead of a full app tree.

---

## How to extract / implement properly

1. **Orient** — `tree` the project; note entry routes and primary screens.
2. **Prefer `get_design_context`** for screens you will build — you get code + the chat that justified choices.
3. **Map, don’t dump** — map Figr screens → user’s routes/features; reuse their design system if they have one; use `/design-system` when attached.
4. **Match visual system** — colors, radii, type scale, spacing from Figr CSS/tokens before inventing new ones.
5. **Preserve structure where useful** — keep screen/component boundaries unless the user’s architecture requires a different split.
6. **Wire to real data** — replace mock state with their APIs; keep UX states (loading/empty/error) if present in the design.
7. **Use queries** — `/queries/qN/prompt.md` and history when “why is it like this?” matters.
8. **Scope grep** — e.g. `path: "/login-prototype"` so search stays on the surface you care about.
9. **Never fabricate** paths or file contents that MCP did not return.

---

## Anti-patterns

- Implementing from memory of a screenshot instead of MCP file contents
- Copying Figr folder layout blindly into an incompatible monorepo
- Ignoring `/design-system` when the project has one attached
- Using storage paths with `/deepagent/` instead of VFS paths from `ls` / `tree`

---

## Starter prompts (for humans)

**Canvas + node**

```text
Use the Figr MCP to understand this canvas, focusing on this node in particular.
Explore the project, then help me implement it in this repo.
<FIGR_URL_WITH_BOARD_NODE>
```

**Whole canvas**

```text
Use the Figr MCP to understand this canvas.
Explore the project, then help me implement it in this repo.
<FIGR_PROJECT_URL>
```

**Design system**

```text
Use the Figr MCP to load this design system and pull tokens, guidelines, and config from it.
<FIGR_DESIGN_SYSTEM_URL>
```
