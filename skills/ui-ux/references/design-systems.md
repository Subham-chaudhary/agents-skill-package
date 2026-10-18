# Community design systems

| System | Best for | Next.js notes |
|---|---|---|
| **shadcn/ui** (default) | Most apps: SaaS, dashboards, landing pages | Copy-in components built on Radix and Tailwind. `bunx shadcn@latest init`, then `bunx shadcn@latest add button`. Components live in `src/components/ui/`. |
| **IBM Carbon** (`@carbon/react`) | Enterprise, admin and data-dense tools | Uses Sass, and many components need `'use client'`. Add the `@carbon/styles` setup. |
| **Material UI** (`@mui/material`) | Apps that want Material Design, or teams that know MUI | Use `@mui/material-nextjs` for App Router style integration. |
| **Mantine** (`@mantine/core`) | Feature-rich apps that need many ready components and hooks | Needs the `MantineProvider` in the root layout (a client component). |
| **Chakra UI** (`@chakra-ui/react`) | Fast, themeable product UIs | Provider in the root layout (client). |
| **HeroUI** (`@heroui/react`) | Polished consumer UIs on Tailwind | Tailwind plugin setup. |
| **Ant Design** (`antd`) | Back-office and CRUD-heavy admin panels | Use `@ant-design/nextjs-registry` for SSR styles. |
| **Fluent UI** (`@fluentui/react-components`) | Microsoft-style and Microsoft 365-adjacent apps | SSR needs the renderer setup from its docs. |
| **Radix Themes** (`@radix-ui/themes`) | Clean, accessible apps without Tailwind | Import the CSS once in the root layout. |
| **GitHub Primer** (`@primer/react`) | Developer tools | |

## Headless (bring your own styles)

Radix Primitives, React Aria (Adobe) and Headless UI. Use these when the user's own design needs accessible behavior without imposed styles.

## Rules

- Prefer systems with Tailwind or CSS-variable theming. They have the least runtime cost with Server Components.
- Runtime CSS-in-JS systems (MUI, Chakra, Ant Design) need their provider or registry setup. Follow the installed version's Next.js guide.
- Icons: `lucide-react` with shadcn/ui; otherwise use the system's own icon set.
