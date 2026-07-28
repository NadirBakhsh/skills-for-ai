---
name: nest-backend-endpoints-response
description: >-
  Designs production-grade NestJS API endpoints with a single consistent
  response envelope (ApiResponse/ApiException), a global exception filter,
  a transform interceptor, ValidationPipe field-level errors, correct HTTP
  status codes, and a standard pagination payload. Use when creating or
  reviewing NestJS controllers, services, DTOs, exception filters, response
  interceptors, REST endpoints, or API error formats. Same wire contract as
  express-backend-endpoints-response; NestJS-native plumbing.
---

# NestJS Backend Endpoints Response

Governs **what a NestJS API endpoint returns** — the response body contract, error format, status codes, and pagination shape — so every endpoint speaks one predictable language that frontends can consume blindly. Complements [`api-endpoint-calls`](../api-endpoint-calls/SKILL.md) (client side) and mirrors the envelope from [`express-backend-endpoints-response`](../express-backend-endpoints-response/SKILL.md) with NestJS-native filters, interceptors, and pipes.

## Response envelope contract (non-negotiable)

Every endpoint — success or failure — returns the same envelope. No exceptions, no per-endpoint shapes.

**Success** (via global `TransformInterceptor` wrapping controller return values):

```json
{
  "statusCode": 200,
  "data": { "id": "65a1...", "title": "Buy milk", "isComplete": false },
  "message": "Todo fetched successfully",
  "success": true
}
```

**Error** (via `throw new ApiException(statusCode, message, errors)` → caught by the global exception filter):

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
src/common/
  dto/api-response.dto.ts          → ApiResponse class
  exceptions/api.exception.ts      → ApiException (extends HttpException)
  filters/all-exceptions.filter.ts → the ONLY place errors become HTTP responses
  interceptors/transform.interceptor.ts → wraps every success response in the envelope
src/<domain>/
  <domain>.controller.ts           → route handlers; return { data, message } or throw ApiException
  <domain>.service.ts              → business logic; never touches Response/Request
  dto/create-<domain>.dto.ts       → class-validator decorators only
  dto/update-<domain>.dto.ts
  <domain>.module.ts
main.ts                            → register global filter, interceptor, ValidationPipe
```

Controllers never call `response.json(...)` with ad-hoc shapes. Services never import `@Res()` unless streaming files — default to returning data and let the interceptor wrap it.

## Hard rules

1. **Every success response goes through the `TransformInterceptor`.** Controllers return `{ data, message }` (or `ApiResponse` directly); the interceptor adds `statusCode` and derived `success`. Never bypass with `@Res()` for ordinary JSON endpoints.
2. **Throw `ApiException`, don't respond on error.** Inside controllers/services, `throw new ApiException(404, "Todo does not exist")` — never `response.status(500).json(...)`. The global exception filter is the only place errors become HTTP responses.
3. **Register global plumbing once in `main.ts`.** `AllExceptionsFilter`, `TransformInterceptor`, and `ValidationPipe` with a custom `exceptionFactory` — not per-controller copies.
4. **Use precise status codes.** `@HttpCode(201)` for creates; 200 read/update/delete, 400 malformed, 401 not authenticated, 403 not allowed, 404 missing, 409 conflict, 422 validation. Never blanket-500 known failures. Full table in [reference.md](reference.md).
5. **`success` is derived from the status code** (`statusCode < 400`), never passed in or hand-set.
6. **Validation errors are field-level via `ValidationPipe`.** Configure `exceptionFactory` to throw `ApiException(422, ..., errors[])` — never Nest's default `{ message: string[], error: "Bad Request" }` shape.
7. **List endpoints always return the standard pagination payload** inside `data`: `{ page, limit, totalPages, previousPage, nextPage, totalItems, currentPageItems, data }`. Never a bare array.
8. **Every response carries a human-readable `message`**, and error responses carry `data: null` explicitly.
9. **Never leak internals.** Use response DTOs / `@Exclude()` / `ClassSerializerInterceptor` to strip `password`, `refreshToken`, and other secrets. Prisma/Mongoose/TypeORM driver errors are normalized in the filter — never forwarded raw.
10. **Unknown errors are normalized** in the global filter (DB errors → 400/409, everything else → 500 with a generic message). Log the full error server-side.

## Workflows

### Adding a new endpoint

1. Create DTOs in `dto/` with `class-validator` decorators (`@IsString()`, `@IsEmail()`, etc.).
2. Add the handler in `<domain>.controller.ts`:
   - Apply `@HttpCode(201)` on creates; use `@Get()`, `@Post()`, `@Patch()`, `@Delete()` decorators.
   - Inject the service; keep the controller thin.
   - Guard clauses: `throw new ApiException(404, "...")` / `409` / `403` as needed.
   - Happy path: `return { data: result, message: "Todo created successfully" }`.
3. Implement business logic in `<domain>.service.ts` — return entities or throw `ApiException`; no HTTP concerns.
4. For list endpoints, build the pagination payload in the service and return it as `data`.
5. Map entities to response DTOs (or apply `@Exclude()`) before they reach `data`.

Templates for every step are in [reference.md](reference.md).

### Reviewing existing endpoints

Check each controller/service against this list and flag violations by severity:

- [ ] **Critical** — `@Res()` used to send ad-hoc JSON instead of the envelope + interceptor
- [ ] **Critical** — errors handled inline (`try/catch` + manual response) instead of throwing `ApiException`
- [ ] **Critical** — no global exception filter or transform interceptor registered
- [ ] **Critical** — sensitive fields (`password`, `refreshToken`, tokens) leaking into response bodies
- [ ] Suggestion — default Nest `ValidationPipe` error shape instead of field-level `errors[]` with 422
- [ ] Suggestion — wrong status codes (200 for creates, 500 for not-found, missing `@HttpCode(201)`)
- [ ] Suggestion — list endpoint returns a bare array instead of the pagination payload
- [ ] Suggestion — business logic in the controller instead of the service
- [ ] Nice to have — missing or non-descriptive `message` ("OK", "Success")
- [ ] Nice to have — stack traces exposed outside development

Report findings as 🔴 Critical / 🟡 Suggestion / 🟢 Nice to have, citing the exact file and line, and propose the concrete fix (usually a template from [reference.md](reference.md)).

## Additional resources

- Copy-paste templates (ApiResponse, ApiException, global filter, transform interceptor, ValidationPipe, pagination, status-code table, full CRUD module): [reference.md](reference.md)
- Concrete good/bad NestJS response-body snippets: [examples.md](examples.md)
