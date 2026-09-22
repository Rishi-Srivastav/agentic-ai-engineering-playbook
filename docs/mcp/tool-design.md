# MCP Tool Design

## Good tool

`get_customer_order(order_id)` is preferable to `do_customer_stuff(input)`.

Each tool should define:

- name
- purpose
- input schema
- output schema
- authorization requirement
- timeout
- side-effect classification
- idempotency behavior
- error taxonomy

For destructive operations, separate preview from execute:

```text
preview_refund(...) → proposed change
approve_refund(...) → explicit authorization
execute_refund(...) → side effect
```
