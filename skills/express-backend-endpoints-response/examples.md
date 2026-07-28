# Examples — Good vs Bad Response Bodies

Concrete pairs for each hard rule. "Bad" snippets are real patterns found in the wild.

## 1. Ad-hoc JSON vs the envelope

**Bad** — every endpoint invents its own shape; clients need per-endpoint parsing:

```ts
// GET /todos/:id
res.json({ todo });

// POST /todos — different endpoint, different shape
res.json({ ok: true, result: created, msg: "done" });
```

**Good** — one envelope everywhere:

```ts
return res
  .status(200)
  .json(new ApiResponse(200, todo, "Todo fetched successfully"));

return res
  .status(201)
  .json(new ApiResponse(201, created, "Todo created successfully"));
```

## 2. Inline error responses / swallowed errors vs thrown `ApiError`

**Bad** — try/catch in the controller responds directly, with the wrong code and a leaked internal message; worse, some branches return success-shaped fallbacks:

```ts
export const getTodoById = async (req, res) => {
  try {
    const todo = await Todo.findById(req.params.todoId);
    if (!todo) {
      return res.status(500).json({ error: "not found" }); // wrong code, ad-hoc shape
    }
    res.json(todo);
  } catch (e) {
    res.json({ todo: null }); // 200 + empty data — the client thinks this succeeded
  }
};
```

**Good** — throw with a precise code; `asyncHandler` + the central error middleware produce the envelope:

```ts
export const getTodoById = asyncHandler(async (req, res) => {
  const todo = await Todo.findById(req.params.todoId);

  if (!todo) {
    throw new ApiError(404, "Todo does not exist");
  }

  return res
    .status(200)
    .json(new ApiResponse(200, todo, "Todo fetched successfully"));
});
```

The client receives:

```json
{
  "statusCode": 404,
  "data": null,
  "message": "Todo does not exist",
  "success": false,
  "errors": []
}
```

## 3. Concatenated validation string vs field-level `errors[]`

**Bad** — one string; the frontend cannot highlight the offending inputs:

```json
{
  "statusCode": 400,
  "message": "Email is invalid, Password must be at least 8 characters",
  "success": false
}
```

**Good** — 422 with one object per field:

```json
{
  "statusCode": 422,
  "data": null,
  "message": "Received data is not valid",
  "success": false,
  "errors": [
    { "email": "Email is invalid" },
    { "password": "Password must be at least 8 characters" }
  ]
}
```

## 4. Bare array vs pagination payload

**Bad** — a bare array; adding pagination later breaks every consumer:

```json
{
  "statusCode": 200,
  "data": [{ "_id": "65a1...", "title": "Buy milk" }],
  "message": "Todos fetched successfully",
  "success": true
}
```

**Good** — standard pagination payload from day one, even when there's one page:

```json
{
  "statusCode": 200,
  "data": {
    "page": 1,
    "limit": 10,
    "totalPages": 1,
    "previousPage": false,
    "nextPage": false,
    "totalItems": 1,
    "currentPageItems": 1,
    "data": [{ "_id": "65a1...", "title": "Buy milk" }]
  },
  "message": "Todos fetched successfully",
  "success": true
}
```

## 5. Leaking secrets and internals vs sanitized data

**Bad** — the raw user document goes straight into the response, and the catch branch leaks driver internals:

```ts
const user = await User.findOne({ email });
res.json(user);
// → { "_id": "...", "email": "...", "password": "$2b$10$N9qo8uLO...",
//     "refreshToken": "eyJhbGciOi...", "__v": 0 }

// elsewhere:
res.status(500).json({ error: err.message });
// → "E11000 duplicate key error collection: prod.users index: email_1"
```

**Good** — explicit exclusion before the data reaches the envelope; duplicates become a clean 409:

```ts
const existing = await User.findOne({ email });
if (existing) {
  throw new ApiError(409, "User with this email already exists");
}

const user = await User.findById(created._id).select("-password -refreshToken");

return res
  .status(201)
  .json(new ApiResponse(201, { user }, "User registered successfully"));
```

## 6. Hand-set `success` vs derived

**Bad** — `success: true` hand-written on an error path during a copy-paste:

```ts
res.status(404).json({ statusCode: 404, data: null, message: "Not found", success: true });
```

**Good** — `ApiResponse`/`ApiError` derive it from the status code; it can never disagree:

```ts
new ApiResponse(200, data, "OK").success;   // true  (derived: 200 < 400)
new ApiError(404, "Not found").success;      // false (always)
```

## 7. Stack traces in production vs development-only

**Bad** — production clients (and attackers) see file paths and framework internals:

```json
{
  "message": "Cannot read properties of null (reading 'owner')",
  "stack": "TypeError: Cannot read properties of null...\n  at /var/app/src/controllers/todo.controllers.js:42:19"
}
```

**Good** — the error middleware includes `stack` only when `NODE_ENV === "development"`, logs the full error server-side, and production receives:

```json
{
  "statusCode": 500,
  "data": null,
  "message": "Something went wrong",
  "success": false,
  "errors": []
}
```
