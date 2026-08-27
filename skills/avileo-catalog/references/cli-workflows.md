# Complete CLI workflows

Use the pinned `@tarileo/avileo-catalog-cli@0.3.0` release and request `--json`
on every invocation. Replace angle-bracket values only with identifiers returned
by an earlier read. Never invent IDs or put a secret in a command or input file.

Run the business context gate first. A successful envelope has `ok: true`; a
failure has `ok: false` and a safe error code. Stop on authentication,
permission, mode, validation, or conflict errors. Retryable remote failures use
exit code `2`; non-retryable authorization or validation failures use exit code
`3`.

## Read commands

Use reads before mutations and keep list queries bounded.

```sh
npx --yes @tarileo/avileo-catalog-cli@0.3.0 business show --json
npx --yes @tarileo/avileo-catalog-cli@0.3.0 catalog product list --limit 25 --offset 0 --json
npx --yes @tarileo/avileo-catalog-cli@0.3.0 catalog product get <PRODUCT_ID> --json
npx --yes @tarileo/avileo-catalog-cli@0.3.0 catalog variant list <PRODUCT_ID> --json
npx --yes @tarileo/avileo-catalog-cli@0.3.0 catalog variant get <VARIANT_ID> --json
npx --yes @tarileo/avileo-catalog-cli@0.3.0 catalog category list --json
npx --yes @tarileo/avileo-catalog-cli@0.3.0 catalog category get <CATEGORY_ID> --json
npx --yes @tarileo/avileo-catalog-cli@0.3.0 catalog tag list --json
npx --yes @tarileo/avileo-catalog-cli@0.3.0 catalog tag get <CATEGORY_ID> --json
npx --yes @tarileo/avileo-catalog-cli@0.3.0 catalog asset list --limit 25 --offset 0 --json
npx --yes @tarileo/avileo-catalog-cli@0.3.0 catalog asset get <ASSET_ID> --json
npx --yes @tarileo/avileo-catalog-cli@0.3.0 catalog inventory get <VARIANT_ID> --json
npx --yes @tarileo/avileo-catalog-cli@0.3.0 catalog inventory movements <VARIANT_ID> --limit 25 --offset 0 --json
```

## JSON input rules

- Represent money, quantity, and stock values as JSON strings such as `"12.50"`
  or `"-2"`, never JSON numbers.
- Use only IDs returned by the CLI. Gallery fields accept at most eight unique
  asset IDs.
- Direct product and variant payloads never include a tenant ID, key, source
  URL, or retry identifier. Unknown fields are rejected.
- Store temporary payloads outside source control and omit supplier-sensitive
  data that the operation does not need.

## Products and variants

A product may include its initial variants:

```json
{
  "name": "Café de origen",
  "unit": "unidad",
  "basePrice": "18.00",
  "description": "Café tostado para venta minorista.",
  "imageAssetIds": ["<ASSET_ID>"],
  "variants": [
    {
      "name": "250 g",
      "unitQuantity": "250",
      "price": "18.00",
      "costPrice": "11.50"
    }
  ]
}
```

Preview, describe the exact result, obtain explicit approval, then apply:

```sh
npx --yes @tarileo/avileo-catalog-cli@0.3.0 catalog product create --from product.json --json
npx --yes @tarileo/avileo-catalog-cli@0.3.0 catalog product create --from product.json --apply --json
npx --yes @tarileo/avileo-catalog-cli@0.3.0 catalog product update <PRODUCT_ID> --from update.json --json
npx --yes @tarileo/avileo-catalog-cli@0.3.0 catalog product update <PRODUCT_ID> --from update.json --apply --json
npx --yes @tarileo/avileo-catalog-cli@0.3.0 catalog product deactivate <PRODUCT_ID> --json
npx --yes @tarileo/avileo-catalog-cli@0.3.0 catalog product deactivate <PRODUCT_ID> --apply --json
npx --yes @tarileo/avileo-catalog-cli@0.3.0 catalog variant create <PRODUCT_ID> --from variant.json --json
npx --yes @tarileo/avileo-catalog-cli@0.3.0 catalog variant create <PRODUCT_ID> --from variant.json --apply --json
npx --yes @tarileo/avileo-catalog-cli@0.3.0 catalog variant reorder <PRODUCT_ID> --from reorder.json --json
npx --yes @tarileo/avileo-catalog-cli@0.3.0 catalog variant reorder <PRODUCT_ID> --from reorder.json --apply --json
```

An update may include only changed fields. Nullable fields can be cleared with
`null`; an empty `imageAssetIds` clears a gallery. A reorder payload must contain
the complete current variant permutation:

```json
{ "variantIds": ["<VARIANT_ID_A>", "<VARIANT_ID_B>"] }
```

## Categories and tags

Category payloads use `name` and optional `color`. Tag payloads require both.
Read the target and preserve assignment guards before update or deletion.

```sh
npx --yes @tarileo/avileo-catalog-cli@0.3.0 catalog category create --from category.json --json
npx --yes @tarileo/avileo-catalog-cli@0.3.0 catalog category create --from category.json --apply --json
npx --yes @tarileo/avileo-catalog-cli@0.3.0 catalog tag update <TAG_ID> --from tag.json --json
npx --yes @tarileo/avileo-catalog-cli@0.3.0 catalog tag update <TAG_ID> --from tag.json --apply --json
```

## Images

Validate a local upload before applying it. For a remote image, put only its
public HTTPS URL in `asset-url.json`. Do not include credentials or local paths.

```json
{ "url": "<PUBLIC_HTTPS_IMAGE_URL>" }
```

```sh
npx --yes @tarileo/avileo-catalog-cli@0.3.0 catalog asset upload --file ./product.png --json
npx --yes @tarileo/avileo-catalog-cli@0.3.0 catalog asset upload --file ./product.png --apply --json
npx --yes @tarileo/avileo-catalog-cli@0.3.0 catalog asset import-url --from asset-url.json --json
npx --yes @tarileo/avileo-catalog-cli@0.3.0 catalog asset import-url --from asset-url.json --apply --json
```

Use the returned asset ID in a later product or variant payload. Never treat an
imported URL as a product identity.

## Inventory

Inventory writes require `inventory:write`. Read the current snapshot first.
An adjustment applies a signed delta; a set protects against stale writes with
`expectedQuantity`.

```json
{ "quantity": "-2", "reason": "Conteo físico" }
```

```json
{
  "quantity": "24",
  "expectedQuantity": "26",
  "reason": "Conteo físico confirmado"
}
```

```sh
npx --yes @tarileo/avileo-catalog-cli@0.3.0 catalog inventory adjust <VARIANT_ID> --from adjust.json --json
npx --yes @tarileo/avileo-catalog-cli@0.3.0 catalog inventory adjust <VARIANT_ID> --from adjust.json --apply --json
npx --yes @tarileo/avileo-catalog-cli@0.3.0 catalog inventory set <VARIANT_ID> --from set.json --json
npx --yes @tarileo/avileo-catalog-cli@0.3.0 catalog inventory set <VARIANT_ID> --from set.json --apply --json
```

## Durable bulk import

Use at most 100 structured products per preview. Products match automatically
only by an explicit product ID or stored external reference, never merely by
similar name or SKU.

```sh
npx --yes @tarileo/avileo-catalog-cli@0.3.0 catalog import preview --from import.json --json
npx --yes @tarileo/avileo-catalog-cli@0.3.0 catalog import apply <IMPORT_ID> --json
npx --yes @tarileo/avileo-catalog-cli@0.3.0 catalog import apply <IMPORT_ID> --apply --json
npx --yes @tarileo/avileo-catalog-cli@0.3.0 catalog import status <IMPORT_ID> --json
```

Review and approve the returned durable preview before apply. Use exactly its
`importId`; target changes after preview become conflicts instead of silent
updates.
