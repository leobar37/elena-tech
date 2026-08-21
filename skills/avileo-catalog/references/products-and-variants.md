# Products and variants

Use `catalog product get` before product updates or deactivation. Create/update
payloads come from `--from <json-file>`; run first without `--apply`, explain
the local validation/diff, and require explicit approval before applying.

Use `catalog variant list` before reordering. A reorder must name the complete,
duplicate-free current variant order. Product and variant galleries accept only
asset IDs, never source URLs.

Useful reads:

```sh
npx --yes @tarileo/avileo-catalog-cli@0.1.0 catalog product get <PRODUCT_ID> --json
npx --yes @tarileo/avileo-catalog-cli@0.1.0 catalog variant list <PRODUCT_ID> --json
```

Do not hard-delete products or variants. Use the supported deactivate command
only after the user explicitly approves the identified resource.
