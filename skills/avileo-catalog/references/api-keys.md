# Avileo API key boundaries

Choose credentials by operation, never by convenience.

| Prefix       | Audience                                    | Boundary                                                        |
| ------------ | ------------------------------------------- | --------------------------------------------------------------- |
| `avileo_pk_` | Public catalog, storefront, browser and SDK | Publishable. It cannot administer Avileo or create credentials. |
| `avileo_sk_` | Trusted server-side agent for one business  | Secret. Inject only as `AVILEO_AGENT_API_KEY`.                  |
| `sak_`       | Cross-product System Admin operator         | Never use it with the tenant Agent API or catalog CLI.          |

The Agent Key has one visible mode:

- `read`: inspect the business, profile, catalog, assets and inventory;
- `mutate`: includes reads and may propose administrative changes.

A `mutate` Agent Key does not execute immediately. It creates an exact,
tenant-scoped proposal, the human approves it in Avileo, and the agent applies
that approved payload. This includes creating an `avileo_pk_` Client Key for a
web project. The public Client Key may then be written to the web project's
`.env`; the secret Agent Key must never be written there or exposed to the
browser.

The CLI reads the Agent Key only from its host environment or protected local
credential store. Never place it in argv, JSON input, URLs, issues, output or
logs. Changing a key between `read` and `mutate` updates the same credential and
does not rotate its secret.
