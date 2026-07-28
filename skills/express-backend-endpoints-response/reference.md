# Reference — Backend Endpoints Response Templates

TypeScript-first, Express-based. The envelope itself is framework-agnostic — see [Framework adaptation](#framework-adaptation) at the end for Fastify/Hono/NestJS.

## `utils/ApiResponse.ts`

```ts
export class ApiResponse<T> {
  public readonly success: boolean;

  constructor(
    public readonly statusCode: number,
    public readonly data: T,
    public readonly message: string = "Success"
  ) {
    // Derived, never passed in — statusCode and success can never disagree.
    this.success = statusCode < 400;
  }
}
```

## `utils/ApiError.ts`

```ts
export type FieldError = Record<string, string>;

export class ApiError extends Error {
  public readonly data: null = null;
  public readonly success: false = false;

  constructor(
    public readonly statusCode: number,
    message: string = "Something went wrong",
    public readonly errors: FieldError[] = [],
    stack?: string
  ) {
    super(message);
    if (stack) {
      this.stack = stack;
    } else {
      Error.captureStackTrace(this, this.constructor);
    }
  }
}
```

## `utils/asyncHandler.ts`

```ts
import type { NextFunction, Request, RequestHandler, Response } from "express";

export const asyncHandler =
  (
    handler: (req: Request, res: Response, next: NextFunction) => Promise<unknown>
  ): RequestHandler =>
  (req, res, next) => {
    Promise.resolve(handler(req, res, next)).catch(next);
  };
```

## `middlewares/error.middlewares.ts`

The **only** place errors become HTTP responses. Normalizes unknown errors, strips stacks outside development, logs everything server-side.

```ts
import type { ErrorRequestHandler } from "express";
import mongoose from "mongoose";
import { logger } from "../logger/index.js";
import { ApiError } from "../utils/ApiError.js";

export const errorHandler: ErrorRequestHandler = (err, req, res, next) => {
  let error = err instanceof ApiError ? err : normalizeUnknownError(err);

  const response = {
    statusCode: error.statusCode,
    data: null,
    message: error.message,
    success: false,
    errors: error.errors,
    // Stack traces are for developers only — never expose in production.
    ...(process.env.NODE_ENV === "development" ? { stack: error.stack } : {}),
  };

  logger.error(`${req.method} ${req.originalUrl} → ${error.statusCode}: ${error.message}`);

  return res.status(error.statusCode).json(response);
};

function normalizeUnknownError(err: unknown): ApiError {
  // Mongoose validation/cast errors are client mistakes → 400.
  if (err instanceof mongoose.Error) {
    return new ApiError(400, err.message, [], err.stack);
  }
  // Prisma known request errors (duplicate key etc.) → 409/400 as appropriate.
  // if (err instanceof Prisma.PrismaClientKnownRequestError) {
  //   return new ApiError(err.code === "P2002" ? 409 : 400, "Database request failed", [], err.stack);
  // }
  const message = err instanceof Error ? err.message : "Something went wrong";
  const stack = err instanceof Error ? err.stack : undefined;
  // Generic message would be safer here if messages may contain internals:
  return new ApiError(500, message, [], stack);
}
```

Mount it **last** in `app.ts`, after all routes:

```ts
app.use(errorHandler);
```

## `validators/validate.ts`

### express-validator variant

```ts
import type { RequestHandler } from "express";
import { validationResult } from "express-validator";
import { ApiError } from "../utils/ApiError.js";

export const validate: RequestHandler = (req, _res, next) => {
  const errors = validationResult(req);
  if (errors.isEmpty()) return next();

  const extractedErrors = errors
    .array()
    .map((err) => ({ [err.type === "field" ? err.path : "unknown"]: err.msg }));

  // 422 Unprocessable Entity — well-formed request, invalid content.
  throw new ApiError(422, "Received data is not valid", extractedErrors);
};
```

Usage in a route: `router.post("/", createTodoValidator(), validate, createTodo)`.

### Zod variant

```ts
import type { RequestHandler } from "express";
import type { ZodType } from "zod";
import { ApiError } from "../utils/ApiError.js";

export const validateBody =
  (schema: ZodType): RequestHandler =>
  (req, _res, next) => {
    const result = schema.safeParse(req.body);
    if (result.success) {
      req.body = result.data;
      return next();
    }
    const extractedErrors = result.error.issues.map((issue) => ({
      [issue.path.join(".") || "body"]: issue.message,
    }));
    throw new ApiError(422, "Received data is not valid", extractedErrors);
  };
```

## Pagination

### Payload shape (always this, for every list endpoint)

```json
{
  "statusCode": 200,
  "data": {
    "page": 2,
    "limit": 10,
    "totalPages": 5,
    "previousPage": true,
    "nextPage": true,
    "totalItems": 48,
    "currentPageItems": 10,
    "data": [ { "...": "..." } ]
  },
  "message": "Todos fetched successfully",
  "success": true
}
```

### In-memory helper (small/aggregated datasets)

```ts
export const getPaginatedPayload = <T>(dataArray: T[], page: number, limit: number) => {
  const startPosition = (page - 1) * limit;
  const totalItems = dataArray.length;
  const totalPages = Math.ceil(totalItems / limit);
  const items = dataArray.slice(startPosition, startPosition + limit);

  return {
    page,
    limit,
    totalPages,
    previousPage: page > 1,
    nextPage: page < totalPages,
    totalItems,
    currentPageItems: items.length,
    data: items,
  };
};
```

### Database-level (mongoose-aggregate-paginate-v2)

```ts
export const getMongoosePaginationOptions = ({
  page = 1,
  limit = 10,
  customLabels = {},
}: {
  page?: number;
  limit?: number;
  customLabels?: Record<string, string>;
}) => ({
  page: Math.max(page, 1),
  limit: Math.max(limit, 1),
  pagination: true,
  customLabels: {
    pagingCounter: "serialNumberStartFrom",
    ...customLabels,
  },
});
```

For SQL/Prisma, run `count()` and `findMany({ skip, take })` in a transaction and map into the same payload shape — the wire format stays identical regardless of the database.

## Status-code decision table

| Situation | Code | Notes |
|---|---|---|
| Read / update / toggle / delete succeeded | 200 | Delete returns the deleted resource in `data` |
| Resource created | 201 | Never 200 for creates |
| Accepted for async processing | 202 | Include a status/tracking reference in `data` |
| Malformed request (bad JSON, missing required param, bad ObjectId) | 400 | |
| Not authenticated (missing/expired/invalid token) | 401 | "Who are you?" |
| Authenticated but not allowed (not owner, wrong role) | 403 | "I know who you are — you can't do this" |
| Resource does not exist | 404 | Also for ids the user must not know exist |
| Duplicate / conflict (email taken, already a member) | 409 | |
| Well-formed but semantically invalid input | 422 | Always with field-level `errors[]` |
| Rate limited | 429 | |
| Unexpected server failure | 500 | Generic message; details go to logs only |

## Full CRUD controller template

```ts
import { Todo } from "../models/todo.models.js";
import { ApiError } from "../utils/ApiError.js";
import { ApiResponse } from "../utils/ApiResponse.js";
import { asyncHandler } from "../utils/asyncHandler.js";
import { getPaginatedPayload } from "../utils/helpers.js";

export const createTodo = asyncHandler(async (req, res) => {
  const { title, description } = req.body;

  const todo = await Todo.create({ title, description, owner: req.user._id });

  return res
    .status(201)
    .json(new ApiResponse(201, todo, "Todo created successfully"));
});

export const getTodoById = asyncHandler(async (req, res) => {
  const todo = await Todo.findById(req.params.todoId);

  if (!todo) {
    throw new ApiError(404, "Todo does not exist");
  }

  return res
    .status(200)
    .json(new ApiResponse(200, todo, "Todo fetched successfully"));
});

export const getAllTodos = asyncHandler(async (req, res) => {
  const page = Math.max(Number(req.query.page) || 1, 1);
  const limit = Math.max(Number(req.query.limit) || 10, 1);
  const query = typeof req.query.query === "string" ? req.query.query.trim() : "";

  const todos = await Todo.find(
    query ? { title: { $regex: query, $options: "i" } } : {}
  ).sort({ updatedAt: -1 });

  const payload = getPaginatedPayload(todos, page, limit);

  return res
    .status(200)
    .json(new ApiResponse(200, payload, "Todos fetched successfully"));
});

export const updateTodo = asyncHandler(async (req, res) => {
  const { title, description } = req.body;

  const todo = await Todo.findByIdAndUpdate(
    req.params.todoId,
    { $set: { title, description } },
    { new: true }
  );

  if (!todo) {
    throw new ApiError(404, "Todo does not exist");
  }

  return res
    .status(200)
    .json(new ApiResponse(200, todo, "Todo updated successfully"));
});

export const deleteTodo = asyncHandler(async (req, res) => {
  const todo = await Todo.findByIdAndDelete(req.params.todoId);

  if (!todo) {
    throw new ApiError(404, "Todo does not exist");
  }

  return res
    .status(200)
    .json(new ApiResponse(200, { deletedTodo: todo }, "Todo deleted successfully"));
});
```

Sensitive-data example (auth domain) — always exclude secrets before they reach `data`:

```ts
const user = await User.findById(userId).select("-password -refreshToken -emailVerificationToken");
return res.status(200).json(new ApiResponse(200, user, "User fetched successfully"));
```

## Framework adaptation

The envelope, status-code rules, pagination payload, and field-level `errors[]` are wire-format decisions — they apply unchanged everywhere. Only the plumbing differs:

- **Fastify** — no `asyncHandler` needed (async handlers are native). Throw `ApiError` and translate in `fastify.setErrorHandler((err, req, reply) => ...)`, building the same response object.
- **Hono** — throw `ApiError` and translate in `app.onError((err, c) => c.json(response, statusCode))`.
- **NestJS** — implement a global exception filter (`@Catch()`) that maps `HttpException`/`ApiError` into the envelope, and a response interceptor that wraps controller return values in `ApiResponse`.
- **Next.js route handlers** — no middleware chain; wrap each handler body in a shared `try/catch` helper that returns `NextResponse.json(envelope, { status })` for both branches.
