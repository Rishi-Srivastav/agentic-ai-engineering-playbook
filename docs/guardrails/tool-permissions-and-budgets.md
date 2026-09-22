# Tool Permissions and Budgets

Use a policy matrix:

| Tool class | Default | Approval |
|---|---|---|
| Read-only internal | allow if authorized | no |
| External API write | restricted | often |
| Financial/security action | deny by default | yes |
| Bulk/export | restricted | yes |

Also enforce:

- max iterations
- max tool calls
- max input/output tokens
- max wall-clock time
- max estimated spend
- rate limits
