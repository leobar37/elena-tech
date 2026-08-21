# Elena Tech Agent Resources

Public, installable resources for agents that interact with Elena Tech products.

## Skills

| Skill | Purpose | Credential boundary |
| --- | --- | --- |
| `avileo-key-routing` | Chooses the correct Avileo credential boundary before an operation. | Does not execute privileged operations. |
| `avileo-catalog` | Safely inspects and manages a registered Avileo `tienda` catalog. | Requires injected `AVILEO_AGENT_API_KEY` (`avileo_sk_`). |

Install a skill:

```sh
npx skills add leobar37/elena-tech --skill avileo-catalog
```

## Credential policy

| Credential | Intended use | Public skill behavior |
| --- | --- | --- |
| `avileo_pk_` | Browser-facing public storefront SDK access. | Never use it for dashboard, catalog-agent, or admin automation. |
| `avileo_sk_` | Secret, tenant-scoped catalog agent access for registered `tienda` businesses. | Use only via injected `AVILEO_AGENT_API_KEY` and the official catalog CLI. |
| `sak_` | Cross-product System Admin operator access. | Never supply it to tenant catalog tooling. No public execution skill exists until a secret-safe System Admin CLI is released. |

Secrets never belong in this repository, command arguments, examples, input files, or logs.

## Repository shape

```text
skills/                 # Canonical, host-neutral Agent Skills
products/<product>/     # Product-owned future resources, such as model profiles
shared/                 # Future cross-product resources
integrations/           # Future host packaging, including Claude Plugin metadata
```

`skills/` remains the only canonical copy of each skill. A future Claude Plugin must package or reference these files rather than duplicate them.
