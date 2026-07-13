---
name: progress-stepper-rsc-suspense
description: >-
  Builds a sequential progress stepper in Next.js 16+ using React Server
  Components, client-side Suspense, and the use() hook. Use when implementing
  multi-step async workflows, confirmation pages, onboarding wizards, or
  timeline UIs where each step unlocks after the previous one completes.
---

# Progress Stepper (RSC + Suspense)

## Architecture

```
Server page (async)
  └─ creates Promise[] via unsafe_createSequentialProcesses
       └─ passes each promise to client StepComponent
            └─ Suspense + use(promise) resolves per-step UI
```

| Layer | File | Role |
|-------|------|------|
| Server | `page.tsx` | Starts async work as promises; never `await`s them for the stepper UX |
| Utility | `sequential.ts` | Chains promises so step N starts only after step N−1 resolves |
| Client | `components.tsx` | Renders each step with `Suspense` + `use(work)` |

## Quick Start

### 1. Chain sequential processes (server utility)

```ts
export function unsafe_createSequentialProcesses<T extends any[], R>(
  ...processes: [(arg?: any) => Promise<T[0]>, ...((arg: any) => Promise<any>)[]]
): Promise<R>[] {
  return processes.reduce((acc, process, index) => {
    if (index === 0) return [process(undefined)];
    return [...acc, acc[acc.length - 1].then(process)];
  }, [] as Promise<any>[]);
}
```

- First process receives `undefined`; each subsequent process receives the previous result.
- Returns an array of promises — one per step — all started from the server page.

### 2. Server page — kick off work, pass promises down

```tsx
// app/confirm/page.tsx — Server Component (no 'use client')
import { StepComponent } from './components';
import { unsafe_createSequentialProcesses } from './sequential';

export default async function ConfirmPage({ params }: { params: { id: string } }) {
  const [first, second, third] = unsafe_createSequentialProcesses(
    () => firstProcess(params.id),
    secondProcess,
    thirdProcess
  );

  return (
    <ol className="relative border-l">
      <StepComponent title="Verify Payment" description="..." work={first} isLast={false} />
      <StepComponent title="Generate PDF" description="..." work={second} isLast={false} />
      <StepComponent title="Upload to S3" description="..." work={third} isLast={true} />
    </ol>
  );
}
```

**Do not** `await` the promises on the server page — that blocks the entire page. Pass them to client components and let Suspense handle per-step resolution.

### 3. Client step component — Suspense + use()

```tsx
'use client';

import { Suspense, use } from 'react';

export type Step = { title: string; description: string; work: Promise<any> };

const Asyncable = ({ work, children }: { work: Promise<any>; children: React.ReactNode }) => {
  use(work);
  return <>{children}</>;
};

export function StepComponent({ title, description, work, isLast }: Step & { isLast: boolean }) {
  return (
    <li className={`ml-6 ${isLast ? '' : 'mb-10'} relative`}>
      <span className="absolute -left-4 ...">
        <Suspense fallback={<StepIcon status="in-progress" />}>
          <Asyncable work={work}><StepIcon status="done" /></Asyncable>
        </Suspense>
      </span>
      <Suspense fallback={<Title disabled>{title}</Title>}>
        <Asyncable work={work}><Title>{title}</Title></Asyncable>
      </Suspense>
      <p className="text-sm text-gray-600">{description}</p>
    </li>
  );
}
```

- `use(work)` suspends until the promise resolves; paired `<Suspense>` shows loading UI.
- Use **separate** Suspense boundaries for icon and title so each can update independently.
- `Title` with `disabled` prop styles the in-progress state (e.g. gray text).

## Rules

1. **`page.tsx` = Server Component** — async functions, no `'use client'`.
2. **`components.tsx` = Client Component** — required for `use()` and per-step Suspense.
3. **Pass promises, don't await** on the server page for stepper rendering.
4. **Chain with `unsafe_createSequentialProcesses`** when steps depend on prior results.
5. **One `work` promise per step** — reuse the same promise in multiple Suspense children (icon + title) within a step; React deduplicates the suspend.
6. **Mark the last step** with `isLast={true}` to control spacing/styling.

## When to Use vs. Alternatives

| Scenario | Use this pattern |
|----------|-----------------|
| Sequential server-side tasks with per-step visual feedback | ✅ |
| Independent parallel tasks | ❌ — start separate promises without chaining |
| Client-only fetch after mount | ❌ — use `useEffect` + local state instead |
| Need error boundary per step | Add `<ErrorBoundary>` around each `Asyncable` |

## Checklist

- [ ] Server page is async and does not await step promises
- [ ] `unsafe_createSequentialProcesses` chains dependent steps
- [ ] Client `StepComponent` uses `'use client'`, `Suspense`, and `use(work)`
- [ ] Each step has distinct loading (fallback) and done states
- [ ] `isLast` set correctly on the final step

## Additional Resources

- Full working example: [examples.md](examples.md)
- Pattern rationale and pitfalls: [reference.md](reference.md)
