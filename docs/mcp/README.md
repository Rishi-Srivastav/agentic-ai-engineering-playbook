# MCP Server Implementation Guidelines

Model Context Protocol (MCP) is a protocol for exposing capabilities such as tools, resources, and prompts to AI applications.

## Design goals

- Narrow tools with one clear responsibility.
- Stable, machine-readable schemas.
- Explicit authorization.
- Bounded output sizes.
- Deterministic errors.
- Request correlation and auditability.
- Safe defaults.

## Server surface

```text
MCP Server
 ├── Tools      → actions/functions
 ├── Resources  → readable context/data
 └── Prompts    → reusable interaction templates
```

Treat an MCP server like a production API, not a convenience script.
