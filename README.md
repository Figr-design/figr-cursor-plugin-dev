# Figr - Cursor marketplace (dev)

Dev/staging variant of the Figr Cursor plugin. MCP → `https://dev-mcp.figr.design/mcp`.

**Live repo:** https://github.com/Figr-design/figr-cursor-plugin-dev (public)

## Install (local)

```bash
git clone https://github.com/Figr-design/figr-cursor-plugin-dev.git
ln -s "$PWD/figr-cursor-plugin-dev/figr-dev" ~/.cursor/plugins/local/figr-dev
```

Reload Cursor. Use this for staging FE (`dev-mcp`), not production.

## Layout

```text
.cursor-plugin/marketplace.json   # marketplace name: figr-dev
figr-dev/                         # plugin name: figr-dev
  .cursor-plugin/plugin.json
  mcp.json                        # → https://dev-mcp.figr.design/mcp
  rules/figr-mcp.mdc
  skills/figr-mcp/
```
