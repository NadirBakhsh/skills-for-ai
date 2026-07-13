# Reference — Progress Stepper Pattern

## Why RSC + Suspense + use()

Traditional approach: `await` all steps on the server → page is blank until everything finishes.

This pattern:
1. Server page **starts** promises immediately (work begins on the server).
2. HTML streams to the client with Suspense fallbacks visible right away.
3. Each step resolves independently as its promise settles.
4. Sequential chaining ensures step 2 doesn't start until step 1 completes — but the UI for step 1 can show "done" while step 2 still shows "in progress".

## How `unsafe_createSequentialProcesses` works

Given processes `[f0, f1, f2]`:

```
p0 = f0(undefined)
p1 = p0.then(f1)
p2 = p1.then(f2)
return [p0, p1, p2]
```

- `p0` resolves when `f0` finishes.
- `p1` resolves when `f0` → `f1` chain finishes (step 2's icon/title wait for this).
- `p2` resolves when the full chain finishes.

Each `StepComponent` receives its own promise. Step 1's UI updates when `p0` resolves; step 2 waits on `p1`, etc.

## Why `Asyncable` is a separate component

`use()` must be called inside a component that is a child of `<Suspense>`. Wrapping `use(work)` in `Asyncable` keeps `StepComponent` itself from suspending at the top level and allows multiple Suspense boundaries (icon + title) to share the same promise.

## Naming: `unsafe_` prefix

The helper uses loose `any` typing and assumes processes are pure sequential pipes. The `unsafe_` prefix signals:
- No runtime validation of process arity or return types.
- Errors in an early step propagate to all downstream promises.
- Not suitable for branching/conditional step graphs without modification.

## Common Pitfalls

| Pitfall | Fix |
|---------|-----|
| `await`ing promises in `page.tsx` | Remove awaits; pass promises to client |
| Putting `use()` in a Server Component | Move to `'use client'` `Asyncable` |
| Steps run in parallel when they should be sequential | Use `unsafe_createSequentialProcesses` |
| Whole page shows one spinner | Add per-step `<Suspense fallback={...}>` |
| Step 2 starts before step 1 finishes | Chain via `.then()` in sequential helper |

## Extending the Pattern

- **Show step result text**: return value from `use(work)` instead of discarding it.
- **Error handling**: wrap `Asyncable` in an error boundary; rejections propagate to the nearest boundary.
- **Skip a step**: conditionally omit `StepComponent` and adjust the process array.
- **Parallel steps**: pass independent promises (no chaining) to steps that don't depend on each other.

## Next.js 16+ Notes

- App Router Server Components support passing promises to Client Components (React 19 `use` API).
- Ensure React 19+ / Next.js 16+ for stable `use()` with promises in client components.
- Tailwind classes in the example are optional; the pattern is UI-framework agnostic.
