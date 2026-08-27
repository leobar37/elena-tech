# Elena Tech Agent Resources

Public, installable resources for agents that interact with Elena Tech products.
This repository is a generated mirror. The normative source lives in
`apps/avileo/agent-skills` in the Elena Tech monorepo.

## Skills

| Skill                | Purpose                                                             | Credential boundary                                      |
| -------------------- | ------------------------------------------------------------------- | -------------------------------------------------------- |
| `avileo-key-routing` | Chooses the correct Avileo credential boundary before an operation. | Does not execute privileged operations.                  |
| `avileo-catalog`     | Safely inspects and manages a registered Avileo `tienda` catalog.   | Requires injected `AVILEO_AGENT_API_KEY` (`avileo_sk_`). |

Install the catalog skill for Codex, Claude Code, or another compatible host:

```sh
npx skills add leobar37/elena-tech --skill avileo-catalog
```

## Credential policy

| Credential   | Intended use                                                                   | Public skill behavior                                                                                                         |
| ------------ | ------------------------------------------------------------------------------ | ----------------------------------------------------------------------------------------------------------------------------- |
| `avileo_pk_` | Browser-facing public storefront SDK access.                                   | Never use it for dashboard, catalog-agent, or admin automation.                                                               |
| `avileo_sk_` | Secret, tenant-scoped catalog agent access for registered `tienda` businesses. | Use only via injected `AVILEO_AGENT_API_KEY` and the official catalog CLI.                                                    |
| `sak_`       | Cross-product System Admin operator access.                                    | Never supply it to tenant catalog tooling. No public execution skill exists until a secret-safe System Admin CLI is released. |

Secrets never belong in this repository, command arguments, examples, input
files, or logs.

## Distribution

Do not edit generated skill files here. The `Sync Avileo Agent Skills` workflow
exports the normative source and opens a reviewable pull request when drift is
found.
