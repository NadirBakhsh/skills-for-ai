# Reference Templates

Copy-paste templates for the API endpoint calls layering described in [SKILL.md](SKILL.md). Works for both React web and React Native/Expo — the only platform-specific pieces are marked below.

## 1. Typed HTTP client (`lib/apiClient.ts`)

```ts
import axios, { AxiosRequestConfig } from "axios"

const BASE_URL = process.env.NEXT_PUBLIC_API_URL ?? process.env.EXPO_PUBLIC_API_URL

export const apiClient = axios.create({
  baseURL: BASE_URL,
  timeout: 10_000,
  headers: { "Content-Type": "application/json" },
})

let _token: string | null = null
export const setToken = (token: string) => { _token = token }
export const clearToken = () => { _token = null }

apiClient.interceptors.request.use((config) => {
  if (_token) config.headers.Authorization = `Bearer ${_token}`
  return config
})

// 401 handling (silent refresh, then logout on repeat failure) goes here.
// Keep it in this file only — never duplicate auth logic in service files.

const get = <T>(url: string, config?: AxiosRequestConfig): Promise<T> =>
  apiClient.get<T>(url, config).then((r) => r.data)

const post = <T>(url: string, data?: unknown, config?: AxiosRequestConfig): Promise<T> =>
  apiClient.post<T>(url, data, config).then((r) => r.data)

const patch = <T>(url: string, data?: unknown, config?: AxiosRequestConfig): Promise<T> =>
  apiClient.patch<T>(url, data, config).then((r) => r.data)

const del = <T>(url: string, config?: AxiosRequestConfig): Promise<T> =>
  apiClient.delete<T>(url, config).then((r) => r.data)

export const api = { get, post, patch, del }
```

## 2. Endpoints map (`services/endpoints.ts`)

Path constants and builders only — no fetch logic, no imports from `apiClient`.

```ts
export const endpoints = {
  orders: {
    list: "/orders",
    create: "/orders",
    detail: (id: string) => `/orders/${id}`,
    update: (id: string) => `/orders/${id}`,
  },
}
```

## 3. Pure API functions (`services/order.ts`)

Throw on failure. No router, no toast, no navigation.

```ts
import { api } from "@/lib/apiClient"
import { endpoints } from "@/services/endpoints"

export type Order = { id: string; status: string /* ... */ }

type OrdersListResponse = { success: boolean; message?: string; orders?: Order[]; total?: number }

export async function fetchOrdersPage(page: number, limit = 20): Promise<{ orders: Order[]; total: number }> {
  const raw = await api.get<OrdersListResponse>(endpoints.orders.list, { params: { page, limit } })
  if (!raw.success || !raw.orders) {
    throw new Error(raw.message ?? "Failed to load orders")
  }
  return { orders: raw.orders, total: raw.total ?? raw.orders.length }
}

export async function createOrder(input: unknown): Promise<Order> {
  const raw = await api.post<{ success: boolean; message?: string; order?: Order }>(
    endpoints.orders.create,
    input,
  )
  if (!raw.success || !raw.order) throw new Error(raw.message ?? "Failed to create order")
  return raw.order
}
```

## 4. Query key factory

One factory per domain. Every hook and every `invalidateQueries` call imports from here — never a raw array literal.

```ts
export const orderKeys = {
  all: ["orders"] as const,
  lists: () => [...orderKeys.all, "list"] as const,
  list: (filters: Record<string, unknown>) => [...orderKeys.lists(), filters] as const,
  counts: () => [...orderKeys.all, "counts"] as const,
  details: () => [...orderKeys.all, "detail"] as const,
  detail: (id: string) => [...orderKeys.details(), id] as const,
}
```

## 5. Query hooks (TanStack Query v5)

```ts
import { useQuery, useInfiniteQuery, queryOptions, infiniteQueryOptions } from "@tanstack/react-query"
import { fetchOrdersPage } from "@/services/order"
import { orderKeys } from "@/services/order-keys"

export const orderQueries = {
  list: (filters: Record<string, unknown>) =>
    infiniteQueryOptions({
      queryKey: orderKeys.list(filters),
      queryFn: ({ pageParam }) => fetchOrdersPage(pageParam, 20),
      initialPageParam: 1,
      getNextPageParam: (lastPage, allPages) => {
        const loaded = allPages.reduce((n, p) => n + p.orders.length, 0)
        return loaded < lastPage.total ? allPages.length + 1 : undefined
      },
      staleTime: 30_000, // volatile domain — see SKILL.md rule 7
    }),
  detail: (id: string) =>
    queryOptions({
      queryKey: orderKeys.detail(id),
      queryFn: () => fetchOrder(id),
      enabled: !!id,
    }),
}

// Usage in a component:
const { data } = useInfiniteQuery(orderQueries.list({ status: "pending" }))
```

## 6. Mutation hook with root invalidation

```ts
import { useMutation, useQueryClient } from "@tanstack/react-query"
import { createOrder } from "@/services/order"
import { orderKeys } from "@/services/order-keys"

export function useCreateOrder() {
  const qc = useQueryClient()
  return useMutation({
    mutationFn: createOrder,
    onSuccess: () => {
      // Invalidate the domain root — covers list, counts, and details together.
      qc.invalidateQueries({ queryKey: orderKeys.all })
    },
  })
}
```

Cross-domain side effects (e.g. an order mutation also affecting a wallet balance) invalidate the other domain's root the same way — `qc.invalidateQueries({ queryKey: walletKeys.all })` — never a hand-typed literal.

## 7. QueryClient defaults

**Web:**

```ts
import { QueryClient } from "@tanstack/react-query"

export const queryClient = new QueryClient({
  defaultOptions: {
    queries: {
      retry: 2,
      staleTime: 1000 * 60, // 1 min default; override per-query per rule 7
      gcTime: 1000 * 60 * 10,
      refetchOnWindowFocus: true,
    },
    mutations: { retry: 0 },
  },
})
```

**React Native / Expo:**

```ts
import { QueryClient } from "@tanstack/react-query"

export const queryClient = new QueryClient({
  defaultOptions: {
    queries: {
      retry: 2,
      staleTime: 1000 * 60 * 5,
      gcTime: 1000 * 60 * 10,
      refetchOnWindowFocus: false, // no window-focus event on native
    },
    mutations: { retry: 0 },
  },
})
```

## 8. Logout cleanup

```ts
function onLogout(qc: QueryClient) {
  clearToken()
  qc.clear() // drop all cached server state so the next user starts clean
}
```
