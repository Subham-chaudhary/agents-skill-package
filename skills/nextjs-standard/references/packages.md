# Packages

First check whether Next.js or the web platform already does the job (`next/image`, `next/font`, `fetch`, `FormData`, `Intl`, `URL`, `crypto.randomUUID`).

## Standard choices

| Need | Package | Note |
|---|---|---|
| Validation | `zod` | At every boundary: forms, route bodies, env, webhooks |
| Styling | `tailwindcss`, `clsx`, `tailwind-merge` | Installed by `create-next-app` |
| UI primitives | `shadcn/ui` (via `bunx shadcn@latest`) | Default design system, see `skills/ui-ux` |
| Icons | `lucide-react` | |
| Theme (dark mode) | `next-themes` | |
| Toasts | `sonner` | |
| Forms (complex) | `react-hook-form`, `@hookform/resolvers` | Simple forms: a Server Action + `useActionState` |
| Client cache (only if needed) | `@tanstack/react-query` | Prefer server fetching |
| Client state (only if needed) | `zustand` | Prefer URL state and server state |
| Dates | `date-fns` | |
| Charts | `recharts` | |
| Email | `resend`, `react-email` | |
| Database / auth / payments | see `skills/integrations` | Supabase by default |
| Server-only guard | `server-only` | |
| Unit tests | `bun test`, `@testing-library/react` | |
| E2E tests | `@playwright/test` | |
| Lint / format | `eslint`, `eslint-config-next`, `prettier`, `prettier-plugin-tailwindcss` | Or Biome if the project already uses it |

## Vetting a package not on this list

All of these must hold:

1. It is widely used (check weekly downloads), had a release in the last 12 months, and has an active repo.
2. It ships TypeScript types and supports the installed React and Next.js versions.
3. The license is permissive: MIT, Apache-2.0, BSD or ISC.
4. The name is spelled exactly like the official one, with no typosquats, and it does not need install scripts (`trustedDependencies`).
5. It is not alpha, beta, canary or experimental.

If it fails any of these, propose the community-standard alternative. If the user still wants it, pin the exact version and record a `decisions` entry.
