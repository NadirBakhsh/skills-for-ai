# AI Coding Rules — Frontend (React)

Rules for **Cursor**, **Claude Code**, and **ChatGPT** when building React frontends with the [recommended stack](./README.md).

Copy this file into:
- **Cursor**: `.cursor/rules/frontend-react.mdc` (see template below)
- **Claude Code**: `CLAUDE.md` or `.claude/rules/`
- **ChatGPT**: Project instructions or custom GPT knowledge

---

## Hard rules (never break)

- Use **React 19** only.
- Use **TypeScript strict mode**.
- **Functional components only.** Never use class components.
- Use **TanStack Query** for server state.
- Use **TanStack Form** + **Zod** for forms and validation.
- Follow **ESLint** and **Prettier** without disabling rules — no `eslint-disable`, no `@ts-ignore`, no `@ts-expect-error` unless the user explicitly approves with a comment explaining why.
- Install packages with **pnpm** only (`pnpm add`, `pnpm add -D`). Never edit `package.json` dependencies manually.

---

## React patterns

- Prefer **custom hooks** over duplicated logic.
- **Avoid unnecessary `useEffect`.** Derive state from props/query data; use event handlers for user actions; reserve `useEffect` for true side effects (subscriptions, imperative DOM, third-party widgets).
- **Avoid unnecessary `useMemo` / `useCallback`.** Add only when profiling shows a real cost or a child requires referential stability (e.g. memoized list items).
- Use **early returns** for loading, error, and empty states — keep the happy path unindented at the bottom.
- Keep components **under 200 lines**. Split into subcomponents or hooks when larger.
- Write **reusable, strongly typed** components. Export explicit prop types (`type FooProps = { ... }`). No `any`.
- Default to **Server Components** on Next.js. Add `'use client'` only when the file needs hooks, browser APIs, or event listeners.

---

## Architecture

Follow **feature-based architecture** — colocate everything a feature needs:

```
src/
├── features/
│   └── billing/
│       ├── components/
│       ├── hooks/
│       ├── api/
│       ├── schemas/      # Zod
│       └── types/
├── components/           # shared UI (buttons, layout shells)
├── hooks/                # shared hooks
├── lib/                  # utils, api client, query client
└── app/                  # Next.js routes (thin — import from features/)
```

- **Routes/pages stay thin** — compose feature components; no business logic in `page.tsx`.
- **One feature folder per domain** (auth, billing, dashboard, settings).
- Shared code lives in `components/`, `hooks/`, `lib/` — not inside unrelated features.
- Server state in **TanStack Query** (`features/*/api/`). Never duplicate fetch logic in `useEffect` + `useState`.
- Form schemas in **Zod** (`features/*/schemas/`) — reuse for API validation where possible.

---

## State rules

| State type | Tool | Example |
|------------|------|---------|
| Server / async | TanStack Query | Lists, detail views, mutations |
| Forms | TanStack Form + Zod | Login, checkout, settings |
| Local UI | `useState` / `useReducer` | Toggle, open/close, input before submit |
| Cross-component UI | Zustand (sparingly) | Global modal, sidebar |
| URL | nuqs / search params | Filters, tabs, pagination |

Never put server-fetched data in Zustand or Context.

---

## Component checklist

Before finishing a component, verify:

- [ ] Typed props; no implicit `any`
- [ ] Early returns for loading / error / empty
- [ ] Under 200 lines (or split)
- [ ] No class components
- [ ] No unnecessary `useEffect`, `useMemo`, `useCallback`
- [ ] Server data via TanStack Query, not manual fetch in `useEffect`
- [ ] Forms use TanStack Form + Zod schema
- [ ] Passes ESLint and Prettier with zero disabled rules

---

## Cursor rule template

Save as `.cursor/rules/frontend-react.mdc`:

```markdown
---
description: React 19 frontend standards — TypeScript strict, TanStack Query/Form, feature architecture
globs: "**/*.{ts,tsx}"
alwaysApply: false
---

# Frontend React Rules

- React 19 only. TypeScript strict. Functional components only — never class components.
- TanStack Query for server state. TanStack Form + Zod for forms.
- Feature-based architecture. Components under 200 lines. Early returns.
- Prefer custom hooks. Avoid unnecessary useEffect, useMemo, useCallback.
- Reusable, strongly typed components. No any.
- ESLint + Prettier — never disable rules. pnpm for all package installs.
- Next.js: Server Components by default; 'use client' only when required.
```

---

## One-line prompt prefix

Paste at the start of any frontend task:

```
Follow skills-for-ai Recommended Frontend Stack: React 19, TS strict, functional components only, TanStack Query (server state), TanStack Form + Zod (forms), feature-based architecture, components <200 lines, early returns, minimal useEffect/useMemo/useCallback, ESLint/Prettier with no rule disables, pnpm only.
```
