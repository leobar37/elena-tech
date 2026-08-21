# Images and galleries

Read an existing asset before associating it. Product and variant mutations use
only tenant-owned asset IDs in their gallery fields.

Remote-image import accepts public HTTPS image sources through the official CLI.
Do not use private, credential-bearing, or local-network sources. The server
validates redirects, destination safety, size, declared type, magic bytes, and
image decoding before producing a reusable asset.

For a local file, use the catalog asset upload command. For a remote source, use
`catalog asset import-url` first without `--apply` if the command supports a
local validation result; otherwise explain the intended reusable asset and ask
for explicit approval before the mutating invocation.
