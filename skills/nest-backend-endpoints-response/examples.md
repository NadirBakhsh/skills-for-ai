# Examples — Good vs Bad NestJS Response Bodies

Concrete pairs for each hard rule. "Bad" snippets are real patterns found in NestJS projects.

## 1. `@Res()` ad-hoc JSON vs interceptor envelope

**Bad** — bypasses the global interceptor; every endpoint invents its own shape:

```ts
@Get(":id")
findOne(@Param("id") id: string, @Res() res: Response) {
  const todo = await this.todosService.findOne(id);
  return res.status(200).json({ todo }); // ad-hoc shape, no envelope
}
```

**Good** — return `{ data, message }`; the `TransformInterceptor` wraps it:

```ts
@Get(":id")
findOne(@Param("id") id: string) {
  return this.todosService.findOne(id);
  // service returns: { data: todo, message: "Todo fetched successfully" }
}
```

## 2. try/catch in controller vs thrown `ApiException`

**Bad** — inline error handling with wrong status and ad-hoc body:

```ts
@Get(":id")
async findOne(@Param("id") id: string) {
  try {
    const todo = await this.prisma.todo.findUnique({ where: { id } });
    if (!todo) {
      return { error: "not found" }; // 200 + wrong shape — client thinks success
    }
    return todo; // bare entity, no message
  } catch {
    throw new InternalServerErrorException(err.message); // leaks internals
  }
}
```

**Good** — service throws `ApiException`; filter produces the envelope:

```ts
// todos.service.ts
async findOne(id: string) {
  const todo = await this.prisma.todo.findUnique({ where: { id } });
  if (!todo) {
    throw new ApiException(404, "Todo does not exist");
  }
  return { data: todo, message: "Todo fetched successfully" };
}
```

Client receives:

```json
{
  "statusCode": 404,
  "data": null,
  "message": "Todo does not exist",
  "success": false,
  "errors": []
}
```

## 3. Default ValidationPipe shape vs field-level `errors[]`

**Bad** — Nest's default validation response; frontends cannot highlight fields:

```json
{
  "statusCode": 400,
  "message": ["email must be an email", "password must be longer than 8 characters"],
  "error": "Bad Request"
}
```

**Good** — custom `exceptionFactory` in `main.ts` throws `ApiException(422, ...)`:

```json
{
  "statusCode": 422,
  "data": null,
  "message": "Received data is not valid",
  "success": false,
  "errors": [
    { "email": "email must be an email" },
    { "password": "password must be longer than 8 characters" }
  ]
}
```

## 4. Bare array vs pagination payload

**Bad** — controller returns entities directly from the service:

```ts
@Get()
findAll() {
  return this.prisma.todo.findMany(); // interceptor wraps bare array as data
}
```

Resulting `data` is a flat array — pagination added later breaks every consumer.

**Good** — service returns the standard pagination object inside `data`:

```ts
async findAll(query: ListTodosQueryDto) {
  const payload = getPaginatedPayload(todos, query.page, query.limit);
  return { data: payload, message: "Todos fetched successfully" };
}
```

## 5. Leaking entity secrets vs response DTOs

**Bad** — Prisma entity returned as-is:

```ts
async register(dto: RegisterDto) {
  const user = await this.prisma.user.create({ data: dto });
  return { data: user, message: "User registered" };
  // → password hash and refreshToken in the response body
}
```

**Good** — map to a response DTO with `@Exclude()`, or select only safe fields:

```ts
async register(dto: RegisterDto) {
  const existing = await this.prisma.user.findUnique({ where: { email: dto.email } });
  if (existing) {
    throw new ApiException(409, "User with this email already exists");
  }

  const user = await this.prisma.user.create({
    data: dto,
    select: { id: true, email: true, username: true, createdAt: true },
  });

  return { data: { user }, message: "User registered successfully" };
}
```

## 6. Missing `@HttpCode(201)` on create

**Bad** — POST returns 200 by default; `success: true` with wrong semantics:

```ts
@Post()
create(@Body() dto: CreateTodoDto) {
  return this.todosService.create(dto); // HTTP 200 — should be 201
}
```

**Good**:

```ts
@Post()
@HttpCode(201)
create(@Body() dto: CreateTodoDto) {
  return this.todosService.create(dto);
}
```

The interceptor reads `response.statusCode` (201) and sets `statusCode: 201, success: true`.

## 7. Per-controller exception handling vs global filter

**Bad** — duplicated error formatting in every controller:

```ts
@Get(":id")
findOne(@Param("id") id: string) {
  try {
    return this.todosService.findOne(id);
  } catch (e) {
    if (e instanceof NotFoundException) {
      return { statusCode: 404, message: e.message, success: false };
    }
    return { statusCode: 500, message: "Error", success: false };
  }
}
```

**Good** — one global `AllExceptionsFilter` registered in `main.ts`; controllers and services only throw:

```ts
// main.ts
app.useGlobalFilters(new AllExceptionsFilter());

// service — just throw
throw new ApiException(404, "Todo does not exist");
```

## 8. Stack traces in production vs development-only

**Bad** — enabling Nest's built-in exception details in production:

```ts
const app = await NestFactory.create(AppModule, {
  logger: ["error", "warn"],
  // abortOnError defaults can expose stack in some setups
});
// No filter — unhandled errors return raw stack traces
```

**Good** — `AllExceptionsFilter` includes `stack` only when `NODE_ENV === "development"` and logs server-side in all environments. Production clients receive:

```json
{
  "statusCode": 500,
  "data": null,
  "message": "Something went wrong",
  "success": false,
  "errors": []
}
```
