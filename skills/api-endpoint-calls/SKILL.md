---
name: api-endpoint-calls
description: >-
  Structures typed API endpoint calls with a shared HTTP client, pure API
  functions, query-key factories, and TanStack Query v5 hooks for React web
  (Next.js/Vite) and React Native/Expo. Use when creating or reviewing
  services/, api/, endpoints, axios/fetch clients, useQuery/useMutation hooks,
  cache invalidation, or server-state data fetching.
---

# API Endpoint Calls

Structures the API/services layer so endpoint calls are typed, testable, and cache-safe — for React web and React Native/Expo alike. Complements [`tanstack-query-best-practices`](../../.agents/skills/tanstack-query-best-practices/SKILL.md) (query/cache mechanics); this skill governs **endpoint wiring, layering, and service conventions**.

## Layering (always)

```
lib/apiClient.ts        → HTTP only: base client, auth header, refresh, typed get/post/patch/del
services/endpoints.ts   → path constants/builders only — no fetch calls
services/<domain>.ts    → pure async API functions that throw on failure
<domain>Keys factory     → hierarchical array query keys
hooks (colocated or hooks/) → useQuery / useMutation / invalidation only
```

Screens/components own navigation and toasts. API functions stay pure — no `router.push`, no `useToast` inside a fetch function.

## Hard rules

1. **Throw on failure.** In `queryFn`/`mutationFn`, throw when the response is an error or `{ success: false }`. Never `catch` and silently return empty/default data — that hides outages from `isError`/`error`.
2. **Query key factories only.** Never invalidate with a raw literal like `['supplier', 'orders']` scattered across files. Define one factory per domain and import it everywhere.
3. **Invalidate the domain root** after mutations (`orderKeys.all`), not a single leaf key — list, detail, and counts all share the prefix and get refreshed together.
4. **Include filters/params in the key** for lists and infinite queries (`orderKeys.list(filters)`), so different filter combinations don't collide in cache.
5. **Use `enabled`** to gate queries that need auth, an id, or another query's result.
6. **Clear or remove queries on logout** (`queryClient.clear()` or targeted `removeQueries`) so the next user never sees stale cached data.
7. **Set `staleTime` by data volatility** — volatile lists (orders, deliveries) ~30s; stable data (catalog, profile) can rely on the QueryClient default (~5 min). Don't use one blanket value for everything.
8. **Prefer one counts/summary endpoint** over N parallel `limit: 1` requests via `useQueries` — cheaper and keeps counts consistent with the list.
9. **Use `select`** to map the raw API envelope to the shape components need, instead of duplicating mapping logic in every consumer.
10. **Infinite queries** always define both `initialPageParam` and `getNextPageParam`.

## Platform deltas

| Concern | Web (Next.js/Vite) | Mobile (Expo/React Native) |
|---|---|---|
| `refetchOnWindowFocus` | Keep default (`true`) or per-app choice | Set `false` — no browser window focus event |
| Token storage | Memory + httpOnly cookie, or secure storage per project | Memory + `expo-secure-store`/Keychain; attach `Bearer` via request interceptor |
| Network status | Browser `online`/`offline` events | Use TanStack Query's `onlineManager` with `@react-native-community/netinfo` if offline handling matters |
| Everything else | Same | Same — endpoints map, pure API functions, key factories, hooks, invalidation rules apply identically |

Detect the platform from the project (presence of `expo`/`react-native` vs `next`/`vite` in `package.json`) and apply the matching `QueryClient` defaults from [reference.md](reference.md).

## Workflows

### Adding a new endpoint

1. Add the path to `services/endpoints.ts` (string or builder function — no logic beyond the path).
2. Write a pure async function in `services/<domain>.ts` that calls the shared `api` client and throws on failure.
3. Add/extend the domain's key factory (`all`, `lists`, `list(filters)`, `details`, `detail(id)`).
4. Wrap it in `useQuery`/`useInfiniteQuery` (reads) or `useMutation` (writes) using the templates in [reference.md](reference.md).
5. For mutations, invalidate the domain root (and any cross-domain keys genuinely affected) in `onSuccess`.

### Reviewing an existing services layer

Check each file against this list and flag violations by severity:

- [ ] **Critical** — errors swallowed and replaced with empty success data instead of thrown
- [ ] **Critical** — raw string/array keys used for invalidation instead of the domain's key factory
- [ ] **Critical** — mutation invalidates only a partial subset of affected keys (e.g. list but not counts)
- [ ] Suggestion — list/detail/counts fetched as separate uncoordinated caches instead of sharing a key prefix
- [ ] Suggestion — API function imports router/toast/navigation (UI concern leaking into the data layer)
- [ ] Suggestion — `staleTime` missing or copy-pasted uniformly regardless of data volatility
- [ ] Nice to have — response mapping duplicated in components instead of centralized via `select`
- [ ] Nice to have — counts computed via multiple `limit: 1` requests instead of one summary endpoint

Report findings as 🔴 Critical / 🟡 Suggestion / 🟢 Nice to have, citing the exact file and line, and propose the concrete fix (usually a template from [reference.md](reference.md)).

## Additional resources

- Copy-paste templates (HTTP client, endpoints, key factory, query/mutation hooks, QueryClient defaults): [reference.md](reference.md)
- Concrete good/bad snippets: [examples.md](examples.md)
