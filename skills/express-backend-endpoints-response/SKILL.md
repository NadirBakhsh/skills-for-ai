---
name: backend-endpoints-response
description: >-
  Designs production-grade backend API endpoints with a single consistent
  response envelope (ApiResponse/ApiError), central error handling, field-level
  validation errors, correct HTTP status codes, and a standard pagination
  payload. Use when creating or reviewing Express/Node controllers, routes,
  REST endpoints, error-handler middleware, response bodies, or API error
  formats. Modeled on the FreeAPI (hiteshchoudhary/apihub) production patterns.
---

# Backend Endpoints Response

Governs **what an API endpoint returns** — the response body contract, error format, status codes, and pagination shape — so every endpoint in the backend speaks one predictable language that frontends can consume blindly. Complements [`api-endpoint-calls`](../api-endpoint-calls/SKILL.md), which governs the client side consuming these responses.

## Response envelope contract (non-negotiable)

Every endpoint — success or failure — returns the same envelope. No exceptions, no per-endpoint shapes.

**Success** (via `new ApiResponse(statusCode, data, message)`):

```json
{
  "statusCode": 200,
  "data": { "_id": "65a1...", "title": "Buy milk", "isComplete": false },
  "message": "Todo fetched successfully",
  "success": true
}
```

**Error** (via `throw new ApiError(statusCode, message, errors)` → caught by the central error middleware):

```json
{
  "statusCode": 422,
  "data": null,
  "message": "Received data is not valid",
  "success": false,
  "errors": [{ "email": "Email is invalid" }, { "password": "Password must be at least 8 characters" }]
}
```

- `success` is **derived** (`statusCode < 400`), never hand-set.
- `data` is always present: the payload on success, `null` on error (never `{}`, never omitted).
- `message` is always a human-readable sentence suitable for a toast.
- `errors` is an array of field-level objects — only on error responses.
- `stack` appears **only** when `NODE_ENV === "development"`.

## Layering (always)

```
routes/<domain>.routes.ts     → paths + middleware chains only (auth, validators, validate) — no logic
validators/<domain>.validators.ts → validation chains / Zod schemas only
validators/validate.ts        → one middleware: collects validation errors, throws ApiError(422, ..., errors[])
controllers/<domain>.controllers.ts → business logic, wrapped in asyncHandler; returns ApiResponse or throws ApiError
middlewares/error.middlewares.ts → the ONLY place errors become HTTP responses
utils/ApiResponse.ts, ApiError.ts, asyncHandler.ts → shared envelope primitives
```

Controllers never build ad-hoc JSON and never call `res.status(...).json({...})` with a hand-rolled object. Services/models never touch `res` at all.

## Hard rules

1. **Every response goes through `ApiResponse` / `ApiError`.** Never raw `res.json({ ... })` — one endpoint deviating breaks every client that trusts the envelope.
2. **Throw, don't respond, on error.** Inside controllers, `throw new ApiError(404, "Todo does not exist")` — never `res.status(500).json(...)`. The central error middleware is the only place errors become HTTP responses, so logging, stack-stripping, and shape stay consistent.
3. **Wrap every async controller in `asyncHandler`.** An unwrapped rejected promise bypasses the error middleware and crashes or hangs the request.
4. **Use precise status codes.** 200 read/update/delete, 201 create, 400 malformed request, 401 not authenticated, 403 authenticated but not allowed, 404 resource missing, 409 conflict/duplicate (e.g. email already registered), 422 validation failure. Never blanket-500 known failures. Full decision table in [reference.md](reference.md).
5. **`success` is derived from the status code** (`statusCode < 400`), never passed in or hand-set — the two can never disagree.
6. **Validation errors are field-level.** Return `errors: [{ field: "message" }, ...]` with 422, never one concatenated string — frontends need per-field errors to highlight inputs.
7. **List endpoints always return the standard pagination payload**: `{ page, limit, totalPages, previousPage, nextPage, totalItems, currentPageItems, data }`. Never a bare array — adding pagination later is a breaking change.
8. **Every response carries a human-readable `message`**, and error responses carry `data: null` explicitly.
9. **Never leak internals.** No raw DB/driver error messages, no `password`, `refreshToken`, or other sensitive fields in `data` — use explicit field selection/exclusion (`.select("-password -refreshToken")` or DTO mapping). Stack traces only in development; log the full error server-side.
10. **Unknown errors are normalized**, not passed through: the error middleware converts non-`ApiError` errors into the envelope (DB/validation errors → 400, everything else → 500 with a generic message).

## Workflows

### Adding a new endpoint

1. Add the route in `routes/<domain>.routes.ts` with its middleware chain: `router.post("/", verifyJWT, createTodoValidator(), validate, createTodo)`.
2. Write the validation chains (or Zod schema) in `validators/<domain>.validators.ts`; the shared `validate` middleware converts failures into `ApiError(422, ..., errors[])`.
3. Write the controller in `controllers/<domain>.controllers.ts`, wrapped in `asyncHandler`:
   - Guard clauses `throw new ApiError(...)` with the precise status code (404 missing, 409 duplicate, 403 forbidden).
   - Happy path ends with `return res.status(code).json(new ApiResponse(code, data, message))` — 201 for creates, 200 otherwise.
4. For list endpoints, use the pagination helper and return the standard pagination payload as `data`.
5. Sanitize the payload: select/exclude sensitive fields before they reach `data`.

Templates for every step are in [reference.md](reference.md).

### Reviewing existing endpoints

Check each route/controller against this list and flag violations by severity:

- [ ] **Critical** — raw `res.json({...})` with an ad-hoc shape instead of the `ApiResponse` envelope
- [ ] **Critical** — errors handled inline (`res.status(500).json(...)` or try/catch that returns success-shaped fallback data) instead of throwing `ApiError`
- [ ] **Critical** — async controller not wrapped in `asyncHandler` (rejections bypass the error middleware)
- [ ] **Critical** — sensitive fields (`password`, `refreshToken`, tokens, internal DB errors) leaking into response bodies
- [ ] Suggestion — wrong or lazy status codes (200 for creates, 500 for not-found, 400 where 422/409/403 is precise)
- [ ] Suggestion — validation failures returned as a single string instead of field-level `errors[]`
- [ ] Suggestion — list endpoint returns a bare array instead of the pagination payload
- [ ] Nice to have — missing or non-descriptive `message` ("OK", "Error")
- [ ] Nice to have — stack traces exposed outside development

Report findings as 🔴 Critical / 🟡 Suggestion / 🟢 Nice to have, citing the exact file and line, and propose the concrete fix (usually a template from [reference.md](reference.md)).

## Additional resources

- Copy-paste templates (ApiResponse, ApiError, asyncHandler, error middleware, validate middleware, pagination, status-code table, full CRUD controller): [reference.md](reference.md)
- Concrete good/bad response-body snippets: [examples.md](examples.md)
