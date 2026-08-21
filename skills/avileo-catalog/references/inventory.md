# Inventory

Inventory reads require `inventory:read`. Adjustments and sets additionally
require `inventory:write`; stop if `business show` does not report it.

Before a correction, read the current inventory and explain the target quantity,
reason, and effect. Use the mutation without `--apply` first where supported;
then require explicit approval before applying.

Use an adjustment for an audited delta and a set only when the user confirms the
expected current quantity. Never infer an expected quantity from a stale list.
