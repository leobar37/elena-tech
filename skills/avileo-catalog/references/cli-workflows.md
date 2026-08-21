# Complete CLI workflows

All commands use the pinned CLI and return one JSON envelope. Replace angle-bracket
values with IDs returned by an earlier read; never invent IDs or put a secret in
the command.

```sh
npx --yes @tarileo/avileo-catalog-cli@0.1.0 <COMMAND> --json
```

Run `business show` first. A successful envelope has `ok: true`; a failure has
`ok: false` and a safe error code. Stop on authentication, permission, mode,
validation, or conflict errors. Retryable remote failures use exit code `2`;
non-retryable authorization or validation failures use exit code `3`.

## Read commands

Use reads before mutations and keep list queries bounded.

```sh
# Catalog products
npx --yes @tarileo/avileo-catalog-cli@0.1.0 catalog product list --limit 25 --offset 0 --json
npx --yes @tarileo/avileo-catalog-cli@0.1.0 catalog product get <PRODUCT_ID> --json

# Variants, categories, and tags
npx --yes @tarileo/avileo-catalog-cli@0.1.0 catalog variant list <PRODUCT_ID> --json
npx --yes @tarileo/avileo-catalog-cli@0.1.0 catalog variant get <VARIANT_ID> --json
npx --yes @tarileo/avileo-catalog-cli@0.1.0 catalog category list --json
npx --yes @tarileo/avileo-catalog-cli@0.1.0 catalog category get <CATEGORY_ID> --json
npx --yes @tarileo/avileo-catalog-cli@0.1.0 catalog tag list --json
npx --yes @tarileo/avileo-catalog-cli@0.1.0 catalog tag get <TAG_ID> --json

# Assets and inventory
npx --yes @tarileo/avileo-catalog-cli@0.1.0 catalog asset list --limit 25 --offset 0 --json
npx --yes @tarileo/avileo-catalog-cli@0.1.0 catalog asset get <ASSET_ID> --json
npx --yes @tarileo/avileo-catalog-cli@0.1.0 catalog inventory get <VARIANT_ID> --json
npx --yes @tarileo/avileo-catalog-cli@0.1.0 catalog inventory movements <VARIANT_ID> --limit 25 --offset 0 --json
```

## JSON input conventions

- Every money, quantity, and stock value is a JSON **string**, for example
  `"12.50"` or `"-2"`; never a JSON number.
- IDs are UUIDs returned by the CLI. Gallery fields contain only asset IDs, at
  most eight, with no duplicates.
- Unknown fields are rejected. Direct product/variant payloads never contain a
  tenant ID, key, source URL, or retry identifier.
- Save a payload outside source control and avoid sensitive product supplier
  data when it is not needed.

### Create a product with variants

Save this as a local JSON file, for example `product.json`:

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
    },
    {
      "name": "500 g",
      "unitQuantity": "500",
      "price": "34.00",
      "costPrice": "21.00"
    }
  ]
}
```

Preview locally first, explain the result, obtain approval, then apply:

```sh
npx --yes @tarileo/avileo-catalog-cli@0.1.0 catalog product create --from product.json --json
npx --yes @tarileo/avileo-catalog-cli@0.1.0 catalog product create --from product.json --apply --json
```

### Update or deactivate a product

An update payload can include only changed fields. `categoryId` and
`description` may be `null` to clear them; use an empty `imageAssetIds` array
to clear a gallery.

```json
{
  "basePrice": "19.00",
  "imageAssetIds": ["<ASSET_ID>"]
}
```

```sh
npx --yes @tarileo/avileo-catalog-cli@0.1.0 catalog product update <PRODUCT_ID> --from update.json --json
npx --yes @tarileo/avileo-catalog-cli@0.1.0 catalog product update <PRODUCT_ID> --from update.json --apply --json
npx --yes @tarileo/avileo-catalog-cli@0.1.0 catalog product deactivate <PRODUCT_ID> --json
npx --yes @tarileo/avileo-catalog-cli@0.1.0 catalog product deactivate <PRODUCT_ID> --apply --json
```

## Variants, categories, and tags

Variant create/update payloads support `name`, optional `sku`, `unitQuantity`,
`price`, optional `costPrice`, `isActive`, and `imageAssetIds`. A variant update
can set `sku` to `null` to clear it.

```sh
npx --yes @tarileo/avileo-catalog-cli@0.1.0 catalog variant create <PRODUCT_ID> --from variant.json --json
npx --yes @tarileo/avileo-catalog-cli@0.1.0 catalog variant create <PRODUCT_ID> --from variant.json --apply --json
npx --yes @tarileo/avileo-catalog-cli@0.1.0 catalog variant reorder <PRODUCT_ID> --from reorder.json --apply --json
```

`reorder.json` contains the full current permutation only:

```json
{ "variantIds": ["<VARIANT_ID_A>", "<VARIANT_ID_B>"] }
```

Category payloads use `name` and optional `color`. Tag payloads require `name`
and `color`. Create/update/delete always follows the same local-preview then
explicit-approval pattern:

```sh
npx --yes @tarileo/avileo-catalog-cli@0.1.0 catalog category create --from category.json --apply --json
npx --yes @tarileo/avileo-catalog-cli@0.1.0 catalog tag update <TAG_ID> --from tag.json --apply --json
```

## Assets

For a local image, validate its path before uploading:

```sh
npx --yes @tarileo/avileo-catalog-cli@0.1.0 catalog asset upload --file ./product.png --json
npx --yes @tarileo/avileo-catalog-cli@0.1.0 catalog asset upload --file ./product.png --apply --json
```

For a remote image, place a public HTTPS image source in `asset-url.json`.
Do not place credentials, local paths, or a raw API key in this file:

```json
{ "url": "<PUBLIC_HTTPS_IMAGE_URL>" }
```

```sh
npx --yes @tarileo/avileo-catalog-cli@0.1.0 catalog asset import-url --from asset-url.json --json
npx --yes @tarileo/avileo-catalog-cli@0.1.0 catalog asset import-url --from asset-url.json --apply --json
```

The server rejects unsafe network destinations, over-sized or invalid images,
and non-image content. Do not convert an imported URL into a product identity;
use the returned asset ID in a later product or variant payload.

## Inventory

Inventory writes require the separately issued `inventory:write` action.
Always read current inventory before preparing either payload.

`adjust.json` applies a signed delta:

```json
{ "quantity": "-2", "reason": "Conteo físico" }
```

`set.json` protects against stale writes:

```json
{
  "quantity": "24",
  "expectedQuantity": "26",
  "reason": "Conteo físico confirmado"
}
```

```sh
npx --yes @tarileo/avileo-catalog-cli@0.1.0 catalog inventory adjust <VARIANT_ID> --from adjust.json --json
npx --yes @tarileo/avileo-catalog-cli@0.1.0 catalog inventory adjust <VARIANT_ID> --from adjust.json --apply --json
npx --yes @tarileo/avileo-catalog-cli@0.1.0 catalog inventory set <VARIANT_ID> --from set.json --apply --json
```

## Durable bulk import

Use at most 100 structured products per preview. Products automatically match
only an explicit product ID or a stored external reference; names and SKU values
are not automatic upsert keys.

```json
{
  "source": "supplier-export",
  "products": [
    {
      "externalRef": { "source": "supplier-export", "externalId": "coffee-250" },
      "name": "Café de origen",
      "unit": "unidad",
      "basePrice": "18.00",
      "variants": [
        { "name": "250 g", "unitQuantity": "250", "price": "18.00" }
      ]
    }
  ]
}
```

```sh
npx --yes @tarileo/avileo-catalog-cli@0.1.0 catalog import preview --from import.json --json
npx --yes @tarileo/avileo-catalog-cli@0.1.0 catalog import apply <IMPORT_ID> --json
npx --yes @tarileo/avileo-catalog-cli@0.1.0 catalog import apply <IMPORT_ID> --apply --json
npx --yes @tarileo/avileo-catalog-cli@0.1.0 catalog import status <IMPORT_ID> --json
```

The preview creates neither products nor remote assets. Apply only the returned
`importId`; never replace it with a local guess. Target changes after preview
become conflicts instead of silent updates.
