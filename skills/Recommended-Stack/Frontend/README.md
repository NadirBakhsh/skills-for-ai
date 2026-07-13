# Recommended Frontend Stack

Default stack for new React projects. Agents should use these choices unless the user specifies otherwise.

## Framework decision

| Project type | Use | Why |
|--------------|-----|-----|
| Marketing site, SaaS, e-commerce, SEO, auth, API routes | **Next.js 16+** (App Router) | SSR/RSC, file-based routing, built-in optimizations |
| SPA, admin dashboard, internal tool, embed | **Vite + React 19** | Faster dev loop, simpler deploy, no server required |
| Heavy scroll/3D landing pages | **Next.js 16+** | Pairs with RSC + Suspense stepper and static generation |

**Default when unsure:** Next.js 16+ with App Router.

---

## Core (always)

| Layer | Choice | Notes |
|-------|--------|-------|
| Language | **TypeScript** (strict) | Non-negotiable for maintainability |
| UI library | **React 19** | Server Components when on Next.js |
| Package manager | **pnpm** | Fast, strict dependency resolution |
| Styling | **Tailwind CSS** (v4) | Utility-first; use design tokens via CSS variables |
| Validation | **Zod** | Shared schemas for forms, API responses, env vars |
| Lint / format | **ESLint 9** + **Prettier** | Flat config; `eslint-config-next` on Next.js projects |

---

## UI & components

| Layer | Choice | When to pick |
|-------|--------|--------------|
| Component library | **shadcn/ui** | Default — copy-paste Radix primitives, full Tailwind control, great DX |
| Alternative | **React Aria Components** | Headless a11y-first UI; custom design systems, no pre-built chrome |
| Icons | **Lucide React** | Default with shadcn; consistent SVG icons |
| Fonts | **next/font** (Next.js) or **@fontsource** (Vite) | Self-hosted, no layout shift |
| Charts | **Recharts** or **Tremor** | Dashboards and data viz inside shadcn layouts |

**Rule:** Pick **one** component approach per project — don't mix shadcn and React Aria patterns in the same surface.

---

## Data & state

| Layer | Choice | Role |
|-------|--------|------|
| Server/async state | **TanStack Query v5** | Fetching, caching, mutations, optimistic updates |
| Forms | **TanStack Form** + **Zod** | Type-safe forms with schema validation |
| Client UI state | **Zustand** or **React `useState`/`useReducer`** | Zustand only when state crosses many components |
| URL state | **nuqs** (Next.js) or **React Router** search params (Vite) | Filters, tabs, pagination in the URL |

**Rule:** Server data lives in TanStack Query — not in global client stores.

---

## Routing

| Framework | Routing |
|-----------|---------|
| Next.js 16+ | App Router (`app/`), Server Components by default, `'use client'` only when needed |
| Vite | **React Router v7** |

---

## Animation (pick by need)

| Need | Skill / library | Use when |
|------|-----------------|----------|
| UI micro-interactions, page transitions, gestures | **Motion** (`motion`) + `motion-dev-animations` skill | Buttons, modals, layout, hero entrances |
| Scroll-scrub, pinning, parallax, scrollytelling | **GSAP + ScrollTrigger** + `gsap-scrolltrigger` skill | Landing pages, narrative scroll experiences |
| Sequential async steps (confirm flows) | `progress-stepper-rsc-suspense` skill | Next.js RSC + Suspense + `use()` steppers |

**Rule:** Motion for component-level UI; GSAP for scroll-driven scenes. Don't load both on every page — code-split by route.

---

## Testing

| Layer | Choice | Scope |
|-------|--------|-------|
| Unit / component | **Vitest** + **Testing Library** | Hooks, utils, isolated components |
| E2E | **Playwright** | Critical user flows, auth, checkout |
| Visual (optional) | Playwright screenshots or Chromatic | Regression on key pages |

---

## Developer experience

| Tool | Purpose |
|------|---------|
| **TypeScript** strict mode | Catch errors at compile time |
| **ESLint** + **Prettier** | Consistent style; run on save |
| **pnpm** | `pnpm add`, `pnpm add -D` — never edit `package.json` deps manually |
| **Turbopack** | Default dev bundler on Next.js 16+ |

---

## Optional add-ons

| Need | Add |
|------|-----|
| Auth | Better Auth, Clerk, or NextAuth (match backend choice) |
| i18n | next-intl (Next.js) or react-i18next (Vite) |
| Tables | TanStack Table |
| Drag and drop | @dnd-kit/core |
| SEO | Next.js Metadata API + `sitemap.ts` / `robots.ts` |
| Analytics | Vercel Analytics, Plausible, or PostHog |
| Design guidance | `ui-ux-pro-max` skill for layout, a11y, typography |

---

## Quick start commands

### Next.js (default)

```bash
pnpm create next-app@latest my-app --typescript --tailwind --eslint --app --src-dir
cd my-app
pnpm add @tanstack/react-query @tanstack/react-form zod motion
pnpm dlx shadcn@latest init
```

### Vite SPA

```bash
pnpm create vite@latest my-app --template react-ts
cd my-app
pnpm add react-router @tanstack/react-query @tanstack/react-form zod motion tailwindcss @tailwindcss/vite
pnpm dlx shadcn@latest init
```

---

## Stack summary (copy-paste)

```
React 19 · TypeScript · pnpm
Next.js 16+ (default) or Vite + React Router
Tailwind CSS v4 · shadcn/ui (default) or React Aria Components
TanStack Query · TanStack Form · Zod
Motion (UI) · GSAP ScrollTrigger (scroll scenes)
ESLint · Prettier · Vitest · Playwright
```

---

## AI coding rules

Mandatory rules for **Cursor**, **Claude Code**, and **ChatGPT** when implementing this stack.

→ Full rules (copy-paste ready): **[AI-CODING-RULES.md](./AI-CODING-RULES.md)**

### Hard rules

- Use **React 19** only.
- Use **TypeScript strict mode**.
- **Functional components only.** Never use class components.
- Use **TanStack Query** for server state.
- Use **TanStack Form** + **Zod** for forms.
- Follow **feature-based architecture** (`src/features/<domain>/`).
- Write **reusable, strongly typed** components.
- Use **early returns** for loading, error, and empty states.
- Keep components **under 200 lines**.
- Follow **ESLint** and **Prettier** without disabling rules.

### React discipline

- Prefer **custom hooks** over duplicated logic.
- **Avoid unnecessary `useEffect`** — derive state, use event handlers, reserve effects for real side effects only.
- **Avoid unnecessary `useMemo` / `useCallback`** — add only when profiling or referential stability requires it.
- **Server Components by default** on Next.js; `'use client'` only when hooks or browser APIs are needed.
