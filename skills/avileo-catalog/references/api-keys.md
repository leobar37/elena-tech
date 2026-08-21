# Avileo API key boundaries

Choose credentials by operation, never by convenience.

| Prefix | Audience | Allowed here |
| --- | --- | --- |
| `avileo_pk_` | Public storefront/browser client | No. It is not a dashboard, tenant-agent, inventory, or admin credential. |
| `avileo_sk_` | Secret tenant Agent Key for registered `tienda` catalog automation | Yes, only when injected as `AVILEO_AGENT_API_KEY`. |
| `sak_` | Cross-product System Admin operator credential | No. It is not tenant-scoped and must never be sent to the catalog CLI. |

The catalog CLI reads the secret only from its host environment. Never place it
in a command argument, a JSON file, a URL, an issue, output, or a log.

An Agent Key grants only its issued actions. Default catalog keys can read the
business, catalog, assets, and inventory; they can write catalog and assets.
Inventory adjustments require the separately granted `inventory:write` action.
A successful prior operation is not proof that a later action is authorized.
