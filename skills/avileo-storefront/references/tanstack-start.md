# TanStack Start storefront

Use the current official scaffold:

```sh
npx @tanstack/cli@latest create
```

Choose TanStack Start, the project's package manager, React Query, TypeScript,
and Tailwind only when the product needs it. Then install the pinned Avileo SDK:

```sh
npm install @tarileo/avileo-sdk@0.3.0 @tanstack/react-query
```

## Environment

```env
VITE_AVILEO_API_URL=https://api.avileo.com
VITE_AVILEO_CLIENT_KEY=avileo_pk_...
```

The Client Key is public but must still come from environment configuration so
it can be rotated without editing source. Never add an Agent Key to the project.
Do not let local development silently target production: default dev to the
local/Tailscale Avileo backend and require an explicit production API URL.

## Shared SDK client

```ts
import { createAvileo } from '@tarileo/avileo-sdk';

const baseUrl = (import.meta.env.VITE_AVILEO_API_URL || 'http://localhost:5201').replace(
  /\/+$/,
  '',
);
const clientKey = import.meta.env.VITE_AVILEO_CLIENT_KEY || '';

export const avileo = createAvileo({ baseUrl, clientKey });
export const avileoProviderConfig = { baseUrl, clientKey } as const;
```

Use `avileo.currentBusiness.catalog.list(...)` in route loaders or helpers.
Resolve relative Avileo asset paths against the configured API origin, not the
storefront origin.

## React root

```tsx
import { AvileoProvider } from '@tarileo/avileo-sdk/react';

export function RootProviders({ children, queryClient }) {
  return (
    <AvileoProvider {...avileoProviderConfig} queryClient={queryClient}>
      {children}
    </AvileoProvider>
  );
}
```

Inside the provider, prefer `useBusiness`, `useCatalog`, `useCart`, and
`useCheckout`. Keep one QueryClient and one AvileoProvider for the app.

## Optional storefront assistant

Only enable it when requested:

```tsx
<AvileoProvider
  {...avileoProviderConfig}
  queryClient={queryClient}
  presence
  features={{ assistant: true }}
>
  {children}
  <StorefrontAssistantHost />
</AvileoProvider>
```

Use the SDK's `StorefrontAssistantHost`; do not compose a second AI provider or
use the private Agent SDK in the browser. Add a purchase adapter when assistant
actions must update the app's cart/checkout.

## Reference architecture

`apps/stuff/elite-coffee` demonstrates TanStack Start SSR, React Query,
`createAvileo`, `AvileoProvider`, catalog, guest cart, checkout, assistant, and
Cloudflare Workers deployment. Copy only the generic architecture.
