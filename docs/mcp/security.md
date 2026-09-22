# MCP Security

Treat tool invocation as an untrusted boundary.

## Controls

- Authenticate the MCP client.
- Authorize each tool independently.
- Enforce tenant/user context server-side.
- Validate every argument.
- Reject unexpected fields where appropriate.
- Prevent SSRF/path traversal/injection in tool parameters.
- Do not trust model-provided user IDs or roles.
- Restrict network egress.
- Keep secrets outside model-visible context.
- Audit sensitive actions.

The model selecting a tool is **not** proof that the caller is authorized to execute it.
