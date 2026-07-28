# Reference — NestJS Backend Endpoints Response Templates

NestJS-native plumbing for the same wire envelope used in [`express-backend-endpoints-response`](../express-backend-endpoints-response/reference.md).

## `common/dto/api-response.dto.ts`

```ts
export class ApiResponse<T> {
  public readonly success: boolean;

  constructor(
    public readonly statusCode: number,
    public readonly data: T,
    public readonly message: string = "Success",
  ) {
    this.success = statusCode < 400;
  }
}
```

## `common/exceptions/api.exception.ts`

```ts
import { HttpException } from "@nestjs/common";
import type { FieldError } from "../types/field-error.type.js";

export class ApiException extends HttpException {
  constructor(
    statusCode: number,
    message: string = "Something went wrong",
    errors: FieldError[] = [],
  ) {
    super(
      {
        statusCode,
        data: null,
        message,
        success: false,
        errors,
      },
      statusCode,
    );
  }
}
```

## `common/types/field-error.type.ts`

```ts
export type FieldError = Record<string, string>;
```

## `common/filters/all-exceptions.filter.ts`

The **only** place errors become HTTP responses. Normalizes `ApiException`, Nest `HttpException`, and unknown errors into the envelope.

```ts
import {
  ArgumentsHost,
  Catch,
  ExceptionFilter,
  HttpException,
  HttpStatus,
  Logger,
} from "@nestjs/common";
import type { Response } from "express";
import { ApiException } from "../exceptions/api.exception.js";
import type { FieldError } from "../types/field-error.type.js";

@Catch()
export class AllExceptionsFilter implements ExceptionFilter {
  private readonly logger = new Logger(AllExceptionsFilter.name);

  catch(exception: unknown, host: ArgumentsHost) {
    const ctx = host.switchToHttp();
    const response = ctx.getResponse<Response>();
    const request = ctx.getRequest<Request>();

    const { statusCode, message, errors } = this.normalize(exception);

    const body = {
      statusCode,
      data: null,
      message,
      success: false,
      errors,
      ...(process.env.NODE_ENV === "development" && exception instanceof Error
        ? { stack: exception.stack }
        : {}),
    };

    this.logger.error(
      `${request.method} ${request.url} → ${statusCode}: ${message}`,
    );

    response.status(statusCode).json(body);
  }

  private normalize(exception: unknown): {
    statusCode: number;
    message: string;
    errors: FieldError[];
  } {
    // Already our envelope shape (thrown by ApiException or ValidationPipe factory).
    if (exception instanceof ApiException) {
      const res = exception.getResponse() as {
        statusCode: number;
        message: string;
        errors: FieldError[];
      };
      return {
        statusCode: exception.getStatus(),
        message: res.message,
        errors: res.errors ?? [],
      };
    }

    // Other Nest HttpExceptions (UnauthorizedException, NotFoundException, etc.).
    if (exception instanceof HttpException) {
      const status = exception.getStatus();
      const res = exception.getResponse();

      if (typeof res === "object" && res !== null && "statusCode" in res) {
        const envelope = res as {
          statusCode: number;
          message: string;
          errors?: FieldError[];
        };
        return {
          statusCode: envelope.statusCode ?? status,
          message: envelope.message ?? exception.message,
          errors: envelope.errors ?? [],
        };
      }

      const message =
        typeof res === "string"
          ? res
          : ((res as { message?: string | string[] }).message ?? exception.message);

      return {
        statusCode: status,
        message: Array.isArray(message) ? message.join(", ") : String(message),
        errors: [],
      };
    }

    // Prisma duplicate key → 409.
    // if (exception instanceof Prisma.PrismaClientKnownRequestError && exception.code === "P2002") {
    //   return { statusCode: 409, message: "Resource already exists", errors: [] };
    // }

    // Mongoose validation/cast → 400.
    // if (exception instanceof mongoose.Error) {
    //   return { statusCode: 400, message: exception.message, errors: [] };
    // }

    return {
      statusCode: HttpStatus.INTERNAL_SERVER_ERROR,
      message: "Something went wrong",
      errors: [],
    };
  }
}
```

## `common/interceptors/transform.interceptor.ts`

Wraps every successful controller return in the envelope.

```ts
import {
  CallHandler,
  ExecutionContext,
  Injectable,
  NestInterceptor,
} from "@nestjs/common";
import type { Response } from "express";
import { Observable, map } from "rxjs";
import { ApiResponse } from "../dto/api-response.dto.js";

/** Controllers return this shape; the interceptor adds statusCode + success. */
export type ControllerPayload<T = unknown> = {
  data: T;
  message: string;
};

@Injectable()
export class TransformInterceptor<T>
  implements NestInterceptor<T, ApiResponse<T>>
{
  intercept(context: ExecutionContext, next: CallHandler): Observable<ApiResponse<T>> {
    const response = context.switchToHttp().getResponse<Response>();

    return next.handle().pipe(
      map((payload) => {
        // Already wrapped (e.g. explicit ApiResponse return).
        if (payload instanceof ApiResponse) {
          return payload;
        }

        // Standard controller return: { data, message }.
        if (
          payload &&
          typeof payload === "object" &&
          "data" in payload &&
          "message" in payload
        ) {
          const { data, message } = payload as ControllerPayload<T>;
          return new ApiResponse(response.statusCode, data, message);
        }

        // Fallback — prefer always returning { data, message } from controllers.
        return new ApiResponse(response.statusCode, payload as T, "Success");
      }),
    );
  }
}
```

## `main.ts` — register global plumbing once

```ts
import { ValidationPipe } from "@nestjs/common";
import { NestFactory } from "@nestjs/core";
import { AppModule } from "./app.module.js";
import { ApiException } from "./common/exceptions/api.exception.js";
import { AllExceptionsFilter } from "./common/filters/all-exceptions.filter.js";
import { TransformInterceptor } from "./common/interceptors/transform.interceptor.js";

async function bootstrap() {
  const app = await NestFactory.create(AppModule);

  app.useGlobalFilters(new AllExceptionsFilter());
  app.useGlobalInterceptors(new TransformInterceptor());

  app.useGlobalPipes(
    new ValidationPipe({
      whitelist: true,
      forbidNonWhitelisted: true,
      transform: true,
      exceptionFactory: (errors) => {
        const extractedErrors = errors.map((err) => ({
          [err.property]:
            err.constraints
              ? Object.values(err.constraints)[0]
              : "Invalid value",
        }));
        return new ApiException(422, "Received data is not valid", extractedErrors);
      },
    }),
  );

  await app.listen(3000);
}
bootstrap();
```

Optional: add `ClassSerializerInterceptor` globally to strip `@Exclude()` fields from entities.

```ts
import { ClassSerializerInterceptor } from "@nestjs/common";
import { Reflector } from "@nestjs/core";

app.useGlobalInterceptors(
  new TransformInterceptor(),
  new ClassSerializerInterceptor(app.get(Reflector)),
);
```

## DTO validation example

```ts
// todos/dto/create-todo.dto.ts
import { IsNotEmpty, IsOptional, IsString, MaxLength } from "class-validator";

export class CreateTodoDto {
  @IsString()
  @IsNotEmpty()
  @MaxLength(120)
  title!: string;

  @IsString()
  @IsOptional()
  @MaxLength(500)
  description?: string;
}
```

```ts
// todos/dto/list-todos-query.dto.ts
import { Type } from "class-transformer";
import { IsInt, IsOptional, IsString, Min } from "class-validator";

export class ListTodosQueryDto {
  @Type(() => Number)
  @IsInt()
  @Min(1)
  @IsOptional()
  page: number = 1;

  @Type(() => Number)
  @IsInt()
  @Min(1)
  @IsOptional()
  limit: number = 10;

  @IsString()
  @IsOptional()
  query?: string;
}
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
    "data": [{ "id": "...", "title": "..." }]
  },
  "message": "Todos fetched successfully",
  "success": true
}
```

### Helper (service layer)

```ts
export const getPaginatedPayload = <T>(items: T[], page: number, limit: number) => {
  const startPosition = (page - 1) * limit;
  const totalItems = items.length;
  const totalPages = Math.ceil(totalItems / limit);
  const data = items.slice(startPosition, startPosition + limit);

  return {
    page,
    limit,
    totalPages,
    previousPage: page > 1,
    nextPage: page < totalPages,
    totalItems,
    currentPageItems: data.length,
    data,
  };
};
```

### Prisma example (database-level)

```ts
async findAll(query: ListTodosQueryDto) {
  const page = Math.max(query.page, 1);
  const limit = Math.max(query.limit, 10);
  const skip = (page - 1) * limit;

  const where = query.query
    ? { title: { contains: query.query, mode: "insensitive" as const } }
    : {};

  const [items, totalItems] = await this.prisma.$transaction([
    this.prisma.todo.findMany({ where, skip, take: limit, orderBy: { updatedAt: "desc" } }),
    this.prisma.todo.count({ where }),
  ]);

  const totalPages = Math.ceil(totalItems / limit);

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
}
```

## Status-code decision table

| Situation | Code | NestJS notes |
|---|---|---|
| Read / update / toggle / delete succeeded | 200 | Default `@Get()` / `@Patch()` / `@Delete()` |
| Resource created | 201 | `@HttpCode(201)` on `@Post()` |
| Accepted for async processing | 202 | `@HttpCode(202)` |
| Malformed request | 400 | `BadRequestException` or `ApiException(400, ...)` |
| Not authenticated | 401 | `UnauthorizedException` or `@UseGuards(JwtAuthGuard)` |
| Authenticated but not allowed | 403 | `ForbiddenException` or `ApiException(403, ...)` |
| Resource does not exist | 404 | `throw new ApiException(404, "Todo does not exist")` |
| Duplicate / conflict | 409 | `throw new ApiException(409, "Email already registered")` |
| Validation failure | 422 | `ValidationPipe` `exceptionFactory` → `ApiException` |
| Rate limited | 429 | `@Throttle()` + `ApiException(429, ...)` |
| Unexpected server failure | 500 | Filter normalizes unknown errors |

## Full CRUD module template

```ts
// todos/todos.controller.ts
import {
  Body,
  Controller,
  Delete,
  Get,
  HttpCode,
  Param,
  Patch,
  Post,
  Query,
} from "@nestjs/common";
import { CreateTodoDto } from "./dto/create-todo.dto.js";
import { ListTodosQueryDto } from "./dto/list-todos-query.dto.js";
import { UpdateTodoDto } from "./dto/update-todo.dto.js";
import { TodosService } from "./todos.service.js";

@Controller("todos")
export class TodosController {
  constructor(private readonly todosService: TodosService) {}

  @Post()
  @HttpCode(201)
  create(@Body() dto: CreateTodoDto) {
    return this.todosService.create(dto);
  }

  @Get()
  findAll(@Query() query: ListTodosQueryDto) {
    return this.todosService.findAll(query);
  }

  @Get(":id")
  findOne(@Param("id") id: string) {
    return this.todosService.findOne(id);
  }

  @Patch(":id")
  update(@Param("id") id: string, @Body() dto: UpdateTodoDto) {
    return this.todosService.update(id, dto);
  }

  @Delete(":id")
  remove(@Param("id") id: string) {
    return this.todosService.remove(id);
  }
}
```

```ts
// todos/todos.service.ts
import { Injectable } from "@nestjs/common";
import { ApiException } from "../common/exceptions/api.exception.js";
import { getPaginatedPayload } from "../common/utils/pagination.util.js";
import { PrismaService } from "../prisma/prisma.service.js";
import type { CreateTodoDto } from "./dto/create-todo.dto.js";
import type { ListTodosQueryDto } from "./dto/list-todos-query.dto.js";
import type { UpdateTodoDto } from "./dto/update-todo.dto.js";

@Injectable()
export class TodosService {
  constructor(private readonly prisma: PrismaService) {}

  async create(dto: CreateTodoDto) {
    const todo = await this.prisma.todo.create({ data: dto });
    return { data: todo, message: "Todo created successfully" };
  }

  async findOne(id: string) {
    const todo = await this.prisma.todo.findUnique({ where: { id } });
    if (!todo) {
      throw new ApiException(404, "Todo does not exist");
    }
    return { data: todo, message: "Todo fetched successfully" };
  }

  async findAll(query: ListTodosQueryDto) {
    const page = Math.max(query.page, 1);
    const limit = Math.max(query.limit, 10);
    const where = query.query
      ? { title: { contains: query.query, mode: "insensitive" as const } }
      : {};

    const todos = await this.prisma.todo.findMany({
      where,
      orderBy: { updatedAt: "desc" },
    });

    const payload = getPaginatedPayload(todos, page, limit);
    return { data: payload, message: "Todos fetched successfully" };
  }

  async update(id: string, dto: UpdateTodoDto) {
    try {
      const todo = await this.prisma.todo.update({ where: { id }, data: dto });
      return { data: todo, message: "Todo updated successfully" };
    } catch {
      throw new ApiException(404, "Todo does not exist");
    }
  }

  async remove(id: string) {
    try {
      const deletedTodo = await this.prisma.todo.delete({ where: { id } });
      return { data: { deletedTodo }, message: "Todo deleted successfully" };
    } catch {
      throw new ApiException(404, "Todo does not exist");
    }
  }
}
```

### Response DTO with `@Exclude()` (auth domain)

```ts
import { Exclude } from "class-transformer";

export class UserResponseDto {
  id!: string;
  email!: string;
  username!: string;

  @Exclude()
  password!: string;

  @Exclude()
  refreshToken!: string;
}
```

Return mapped DTOs from the service, or use `plainToInstance(UserResponseDto, user)` before placing in `data`.

## Express vs NestJS plumbing

| Concern | Express skill | NestJS skill |
|---|---|---|
| Success wrapping | Manual `new ApiResponse(...)` in controller | `TransformInterceptor` wraps `{ data, message }` |
| Error handling | `errorHandler` middleware | `AllExceptionsFilter` |
| Async safety | `asyncHandler` wrapper | Native — not needed |
| Validation | `validate` middleware + express-validator/Zod | Global `ValidationPipe` + DTOs |
| Status codes | `res.status(201).json(...)` | `@HttpCode(201)` + interceptor reads `response.statusCode` |
| Auth | Route middleware (`verifyJWT`) | `@UseGuards(JwtAuthGuard)` |

The **wire envelope is identical** — only the plumbing differs.
