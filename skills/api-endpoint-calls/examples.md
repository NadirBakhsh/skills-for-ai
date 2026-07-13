# Good vs Bad Examples

Concrete snippets illustrating the rules in [SKILL.md](SKILL.md), based on patterns observed in a real Expo/React Native services layer review.

## 1. Query key factory

**Good** — one factory, hierarchical, imported everywhere:

```ts
export const supplierCustomerKeys = {
  all: ["supplier-customers"] as const,
  lists: () => [...supplierCustomerKeys.all, "list"] as const,
  list: () => [...supplierCustomerKeys.lists()] as const,
  details: () => [...supplierCustomerKeys.all, "detail"] as const,
  detail: (id: string) => [...supplierCustomerKeys.details(), id] as const,
}
```

**Bad** — raw literals duplicated across mutation files, prone to typos and drift:

```ts
// in one file
qc.invalidateQueries({ queryKey: ['supplier', 'orders'] })
// in another file, meant to be the same domain
qc.invalidateQueries({ queryKey: ['supplier-orders'] }) // typo — never matches, cache never invalidates
```

## 2. Throwing vs swallowing errors

**Bad** — error is logged and hidden; the query reports success with empty data:

```ts
async function fetchOrdersPage(params) {
  try {
    const res = await apiClient.get(endpoints.orders.list, { params })
    if (res.data?.success) return mapPage(res.data)
  } catch (e) {
    console.error('Failed to fetch orders', e)
  }
  return { orders: [], total: 0 } // isError is false, UI shows "no orders" instead of an error state
}
```

**Good** — failure surfaces through `isError`/`error`, retry works correctly:

```ts
async function fetchOrdersPage(params) {
  const res = await api.get<OrdersListResponse>(endpoints.orders.list, { params })
  if (!res.success) throw new Error(res.message ?? "Failed to load orders")
  return mapPage(res)
}
```

## 3. Invalidation completeness

**Bad** — mutation only invalidates the list, missing the counts that share the same domain:

```ts
onSuccess: () => {
  qc.invalidateQueries({ queryKey: ["orders", "list"] })
  // counts key ["orders", "counts"] never refreshed — filter tabs show stale numbers
}
```

**Good** — invalidate the domain root so list, detail, and counts all refresh together:

```ts
onSuccess: () => {
  qc.invalidateQueries({ queryKey: orderKeys.all }) // covers list(), counts(), detail(id)
}
```

## 4. Pure API functions vs UI-coupled API functions

**Bad** — the "service" mutation reaches into routing and toast, making it impossible to reuse or test the API call on its own:

```ts
export const useCompleteProfile = () => {
  const router = useRouter()
  return useMutation({
    mutationFn: (payload) => api.post(endpoints.auth.completeProfile, payload),
    onSuccess: () => router.replace("/success"), // fine here — this is the hook layer
  })
}

// but the pure function itself must not do this:
export async function completeProfileApi(payload) {
  const res = await api.post(endpoints.auth.completeProfile, payload)
  router.replace("/success") // wrong — navigation has no place in the data layer
  return res
}
```

**Good** — navigation/toast live only in the hook's `onSuccess`/`onError`, never inside the plain async function that performs the request.

## 5. Counts: one endpoint vs N parallel requests

**Bad** — five separate network requests just to render filter tab counts:

```ts
useQueries({
  queries: [
    { queryKey: [...keys, "all"], queryFn: () => getTotal({}) },
    { queryKey: [...keys, "pending"], queryFn: () => getTotal({ status: "pending" }) },
    { queryKey: [...keys, "completed"], queryFn: () => getTotal({ status: "completed" }) },
    { queryKey: [...keys, "cancelled"], queryFn: () => getTotal({ status: "cancelled" }) },
    { queryKey: [...keys, "unassigned"], queryFn: () => getTotal({ unassignedOnly: true }) },
  ],
})
```

**Good** — one summary endpoint returns all counts in a single round trip:

```ts
useQuery({
  queryKey: orderKeys.counts(),
  queryFn: () => api.get<OrderCounts>(endpoints.orders.counts),
  staleTime: 30_000,
})
```

## 6. Infinite queries

**Good** — both required options present, page size derived from loaded count vs total:

```ts
useInfiniteQuery({
  queryKey: orderKeys.list(filters),
  queryFn: ({ pageParam }) => fetchOrdersPage({ ...filters, page: pageParam }),
  initialPageParam: 1,
  getNextPageParam: (lastPage, allPages) => {
    const loaded = allPages.reduce((n, p) => n + p.orders.length, 0)
    return loaded < lastPage.total ? lastPage.page + 1 : undefined
  },
})
```

**Bad** — missing `getNextPageParam` guard causes infinite re-fetching of the last page once all data is loaded, or `initialPageParam` omitted (required in TanStack Query v5, causes a type/runtime error).
